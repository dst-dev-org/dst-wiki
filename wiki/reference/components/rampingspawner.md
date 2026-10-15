---
title: "rampingspawner"
created: 2026-10-26
updated: "2026-10-28"
type: component
summary: "维护递增波次的待生成配额、子体追踪和波间任务，由宿主 Brain／状态图决定实际逐只生成。"
aliases: ["rampingspawner", "龙蝇吐熔岩虫", "递增波次生成", "生成配额与暂停"]
tags: [component, lifecycle, callback, ai]
dst_build: "steam-buildid:26230496"
sources: ["components/rampingspawner.lua", "prefabs/dragonfly.lua", "brains/dragonflybrain.lua", "stategraphs/SGdragonfly.lua", "mathutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# rampingspawner

## 快速判断

- 职责与适用：管理波次配额、已生成实体集合和波间等待；组件本身没有自动逐只出怪的更新循环。原版龙蝇由 Brain 判断配额，再经状态图动画调用 `SpawnEntity`。[rampingspawner.lua:3-26](../../../raw/current/scripts/components/rampingspawner.lua#L3-L26) [dragonflybrain.lua:44-60](../../../raw/current/scripts/brains/dragonflybrain.lua#L44-L60) [SGdragonfly.lua:675-688](../../../raw/current/scripts/stategraphs/SGdragonfly.lua#L675-L688)
- 优先使用：需要“宿主动画逐只生成、耗尽配额后延迟加下一波”时参照龙蝇整条接线；只需定时产物时先看 [[reference/components/periodicspawner]]，不要只装本组件便期待自动刷怪。[rampingspawner.lua:93-109](../../../raw/current/scripts/components/rampingspawner.lua#L93-L109)
- 重要副作用：`Stop` 只停活跃标记和波间任务；`Reset` 只清待生成配额并解除子体追踪。两者都不删除已生成实体，`Reset` 也不重置累计波次。[rampingspawner.lua:128-149](../../../raw/current/scripts/components/rampingspawner.lua#L128-L149)
- 修改边界：组件没有主模拟断言或网络副本，原版在龙蝇客户端提前返回后安装；宿主应在服务端执行生成、事件和存读协作。[dragonfly.lua:490-521](../../../raw/current/scripts/prefabs/dragonfly.lua#L490-L521)

## 关键接口

| 接口／字段 | 签名或已核验约束 | 条件、作用与证据 |
|---|---|---|
| `spawn_prefab`、`min_wave/max_wave/waves_to_max/wave_time` | 默认 `lavae`、4／10／6／30秒 | 普通公开字段，无 setter 校验；`waves_to_max` 用于除法，不能设0。`max_wave` 是插值端点，不是硬容量。[rampingspawner.lua:6-18](../../../raw/current/scripts/components/rampingspawner.lua#L6-L18) [rampingspawner.lua:65-72](../../../raw/current/scripts/components/rampingspawner.lua#L65-L72) |
| `GetCurrentWave()`／`GetWaveSize()`／`DoWave()` | 返回待生成数／当前波大小；后者无返回 | `DoWave` 先加1累计波次，再把计算出的波大小加到待生成数；不检查是否活跃，不直接生成实体。[rampingspawner.lua:61-72](../../../raw/current/scripts/components/rampingspawner.lua#L61-L72) |
| `Start()`／`IsActive()` | 启动／查询 `spawning_on` | 已活跃时 Start 无动作；否则立即 DoWave，再置活跃并调用 `onstartfn(inst)`。启动不排逐只生成任务。[rampingspawner.lua:111-126](../../../raw/current/scripts/components/rampingspawner.lua#L111-L126) |
| `Stop()` | 仅已活跃时生效 | 置不活跃、取消波间 `SpawnTask`，调用 `onstopfn(inst)`；保留 `current_wave`、`wave_num` 和子体追踪。[rampingspawner.lua:128-142](../../../raw/current/scripts/components/rampingspawner.lua#L128-L142) |
| `SpawnEntity()` | 生成 `spawn_prefab`；无显式返回 | 不检查活跃、配额或存活容量；追踪后设置位置朝向、配额减1，宿主收到 `rampingspawner_spawn`，数据 `{newent=spawn}`。预制件必须有效且具备所用接口。[rampingspawner.lua:93-109](../../../raw/current/scripts/components/rampingspawner.lua#L93-L109) |
| `getspawnposfn(inst)`／`getspawnrotfn(inst)` | 返回带 `Get()` 的位置／旋转数值 | 可选；缺省用宿主位置和旋转。组件不检查地形、安全距离或坐标有效性。[rampingspawner.lua:74-86](../../../raw/current/scripts/components/rampingspawner.lua#L74-L86) |
| `TrackSpawn(spawn)`／`StopTrackingSpawn(spawn)` | 集合登记／撤销 | 幂等维护 `spawns`、`num_spawns`；监听子体 `inevitabledeath`、`death`、`onremove`。移除只撤追踪，死亡路径另通知宿主 `rampingspawner_death`，数据 `{remaining_spawns=num_spawns}`。[rampingspawner.lua:36-59](../../../raw/current/scripts/components/rampingspawner.lua#L36-L59) |
| `Reset()`／`OnRemoveFromEntity()` | 清配额与追踪／删除组件钩子 | Reset 不停止、不撤波间任务、不清 `wave_num`；删除组件先撤任务再 Reset，但不调用 Stop 或 `onstopfn`。[rampingspawner.lua:28-34](../../../raw/current/scripts/components/rampingspawner.lua#L28-L34) [rampingspawner.lua:144-149](../../../raw/current/scripts/components/rampingspawner.lua#L144-L149) |

## 关键流程

### 波次大小、待生成数与波间等待

`wave_num` 初始0，波大小为 `floor(Lerp(min_wave,max_wave,wave_num/waves_to_max))`。`Start` 先递增波次，因此默认第一次启动加5个配额，第6波加10个，第7波可加11个；`Lerp` 不钳制参数，不能把 `max_wave=10` 当后续永远不超过10。改变参数影响以后计算，不回算已经加入的配额。[rampingspawner.lua:10-15](../../../raw/current/scripts/components/rampingspawner.lua#L10-L15) [rampingspawner.lua:61-72](../../../raw/current/scripts/components/rampingspawner.lua#L61-L72) [mathutil.lua:14-16](../../../raw/current/scripts/mathutil.lua#L14-L16)

每次 `SpawnEntity` 将 `current_wave` 减1；减后不大于0且仍活跃，才把一个 `wave_time` 秒任务挂在宿主上。任务触发仅清任务句柄并 DoWave，实际出怪仍等宿主再次调用。这里不等待 `num_spawns` 清零；`num_spawns` 是被追踪的在外数量，并非并发容量门槛。[rampingspawner.lua:88-109](../../../raw/current/scripts/components/rampingspawner.lua#L88-L109)

### 保存、恢复与退出

`OnSave` 保存 `current_wave`、`wave_num`、`spawning_on`，以及被追踪子体 GUID 并返回引用列表；不保存波间任务剩余时间、生成配置或回调。`OnLoad` 先恢复配额和累计波次，若存档活跃则调用 Start：这会再递增波次并追加一波配额，不是原样恢复活跃标记或剩余计时。`LoadPostPass` 仅对仍能在 `ents` 找到的实体重新 TrackSpawn。[rampingspawner.lua:151-190](../../../raw/current/scripts/components/rampingspawner.lua#L151-L190)

需要“停调度且清待生成配额”时使用 Stop 后 Reset；若想连累计增长一并重置，还需宿主明确管理 `wave_num`。对实体的删除／撤退要另行实现；组件移除也不会运行 `onstopfn`，不能依赖它撤销开始回调加上的外部效果。[rampingspawner.lua:28-59](../../../raw/current/scripts/components/rampingspawner.lua#L28-L59) [rampingspawner.lua:128-149](../../../raw/current/scripts/components/rampingspawner.lua#L128-L149)

## 典型接入

### 原版龙蝇：血量节点启动，Brain 选池，动画生成

这是服务端已有组件的原版参照，不是独立可运行 demo。龙蝇在血量节点回调中先发变形请求再 Start；80%、50%、20% 共用该回调，血量跨线本身的触发语义见 [[reference/components/healthtrigger]]。开始／停止回调分别加上和移除1.4倍的 `spawning` 移速来源。[dragonfly.lua:416-438](../../../raw/current/scripts/prefabs/dragonfly.lua#L416-L438) [dragonfly.lua:556-571](../../../raw/current/scripts/prefabs/dragonfly.lua#L556-L571)

Brain 只看 `GetCurrentWave()>0` 决定是否进入吐虫分支，从出生点附近查 `lava` 标签位置、飞往目标并推 `spawnlavae`；它没有再用 `IsActive()` 作门槛。SG 收到事件并在非忙碌且未死时进入 `lavae`，动画第59帧经 `brain:OnSpawnLavae()` 调用组件 SpawnEntity。原版位置回调取龙蝇朝向前方约1.7、y偏移-0.3的位置。[dragonflybrain.lua:43-86](../../../raw/current/scripts/brains/dragonflybrain.lua#L43-L86) [SGdragonfly.lua:59-66](../../../raw/current/scripts/stategraphs/SGdragonfly.lua#L59-L66) [SGdragonfly.lua:661-707](../../../raw/current/scripts/stategraphs/SGdragonfly.lua#L661-L707) [dragonfly.lua:336-343](../../../raw/current/scripts/prefabs/dragonfly.lua#L336-L343)

生成事件由宿主把新熔岩虫关联到母体并锁定目标。死亡事件在待生成配额与剩余追踪数都不大于0时，执行 Stop、Reset并请求火焰变形；普通 `onremove` 路径只撤追踪，不发同样的死亡通知。龙蝇 SoftReset 也配对 Stop／Reset，而已生成虫的撤退由宿主另设 `reset` 处理。[dragonfly.lua:221-255](../../../raw/current/scripts/prefabs/dragonfly.lua#L221-L255) [dragonfly.lua:345-366](../../../raw/current/scripts/prefabs/dragonfly.lua#L345-L366) [事件接线 dragonfly.lua:592-597](../../../raw/current/scripts/prefabs/dragonfly.lua#L592-L597)

## 常见陷阱与限制

- **Start 就会定时自动出怪：** Start 只加配额；必须给宿主接完整的判断、行为和实际 SpawnEntity 调用，组件头注释也明确要求接 Brain。[rampingspawner.lua:1](../../../raw/current/scripts/components/rampingspawner.lua#L1 [rampingspawner.lua:115-126](../../../raw/current/scripts/components/rampingspawner.lua#L115-L126)
- **Stop 能暂停一切出怪：** Stop 保留待生成数，而龙蝇 Brain 只查配额；已进入动画的调用也不由组件拦截。需要完整停止时还应处理宿主行为、动画和 Reset 边界。[rampingspawner.lua:93-142](../../../raw/current/scripts/components/rampingspawner.lua#L93-L142) [dragonflybrain.lua:44-60](../../../raw/current/scripts/brains/dragonflybrain.lua#L44-L60)
- **Reset 是从第一波重开：** 它不清 `wave_num`，也不取消已有波间任务；仅 Reset 后旧任务仍可能追加下一波，停止后再次 Start 也会在保留配额上叠加新波。[rampingspawner.lua:88-91](../../../raw/current/scripts/components/rampingspawner.lua#L88-L91) [rampingspawner.lua:115-149](../../../raw/current/scripts/components/rampingspawner.lua#L115-L149)
- **读档精确续播波次：** 组件恢复活跃存档会追加一波；原版龙蝇另有宿主战斗恢复逻辑，保存为玩家战斗中时会延后请求重置，不能单凭组件存档字段推导龙蝇整场战斗续接结果。[rampingspawner.lua:171-190](../../../raw/current/scripts/components/rampingspawner.lua#L171-L190) [dragonfly.lua:386-408](../../../raw/current/scripts/prefabs/dragonfly.lua#L386-L408)

## 验证记录

- 2026-10-26，`steam-buildid:26148036`：审阅组件全文、Lerp、龙蝇服务端安装／配置／阈值启动／事件与复位／宿主存读分支、Brain 吐虫分支与 SG 对应事件／完整 lavae 状态，覆盖配额、波次增长、实际生成、退出清理和组件存读。
- 游戏运行：未运行；未验证打断动画、读档回调顺序、熔岩虫实际撤退完成或模组替换实体兼容性。未展开整场龙蝇战斗及子体 AI。
- 结构检查：`validate_page.py` 通过；两页合并执行 `validate_source_links.py --strict`，94 个行号引用、0错误／0警告。本页无新增 Lua 示例；机械检查不代替游戏测试。

- 版本沿用：2026-10-28 原始哈希等价批量沿用-70（26148036→26230496，本页 sources 所列文件原始字节全部未变），页内未决项零标记、符号反查无命中、严格链接通过；批次依据见 [[updates/build-26230496/index#wiki-沿用记录]]。
相关：[[reference/components/healthtrigger]] · [[reference/components/childspawner]]
