---
title: "hatchable：按热源偏好推进孵化状态"
created: 2026-10-26
updated: "2026-10-28"
type: component
summary: "为蛋维护破壳、舒适孵化与失败计时，由宿主回调完成换蛋、出雏和表现。"
aliases: [hatchable, 蛋孵化, 破壳计时, 孵化冷热偏好]
tags: [component, state-machine, callback, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: [components/hatchable.lua, prefabs/tallbirdegg.lua, prefabs/lavae_egg.lua]
reliability: verified
validation: source-reviewed
status: active
---

# hatchable：按热源偏好推进孵化状态

## 快速判断

- 职责与适用：维护蛋的 `state`、孵化进度和不适累计；以附近 heater 输出总和的正负及世界昼夜阶段判定条件，不直接读取蛋的温度。[hatchable.lua:106-152](../../../raw/current/scripts/components/hatchable.lua#L106-L152)
- 优先使用：需要“连续满足热源条件才破壳，破壳后舒适推进、不适失败”的蛋；用 `SetOnState` 接管状态结果。组件不会自行生成幼体。[hatchable.lua:34-61](../../../raw/current/scripts/components/hatchable.lua#L34-L61)、[hatchable.lua:154-189](../../../raw/current/scripts/components/hatchable.lua#L154-L189)
- 重要副作用：`hatch`／`dead` 会停止周期任务；状态回调可能由宿主替换或移除当前实体，存档加载也会重放非 `unhatched` 状态回调。[hatchable.lua:173-221](../../../raw/current/scripts/components/hatchable.lua#L173-L221)
- 修改边界：原版在服务端添加和配置组件，背包暂停与加载后修正由 prefab 负责；组件自身没有持有物品判定。[tallbirdegg.lua:200-230](../../../raw/current/scripts/prefabs/tallbirdegg.lua#L200-L230)、[tallbirdegg.lua:58-78](../../../raw/current/scripts/prefabs/tallbirdegg.lua#L58-L78)

## 关键接口

| 接口／字段 | 参数、默认值 | 条件、作用与证据 |
|---|---|---|
| `SetOnState(fn)`、`OnState(state, forcestateupdate)` | 回调 `fn(inst, state)` | 状态改变或强制更新才调用；先写 `self.state`，再通知。可用于动画、换 prefab 和最终出雏。[hatchable.lua:34-61](../../../raw/current/scripts/components/hatchable.lua#L34-L61) |
| `SetCrackTime(t)`、`SetHatchTime(t)`、`SetHatchFailTime(t)` | 以更新累计时间计；默认 10、600、60 秒 | 分别控制破壳、孵化进度阈值、不适失败阈值；setter 仅赋值，不校验或重置已有进度。[hatchable.lua:4-11](../../../raw/current/scripts/components/hatchable.lua#L4-L11)、[hatchable.lua:38-51](../../../raw/current/scripts/components/hatchable.lua#L38-L51) |
| `SetHeaterPrefs(day, dusk, night)`、`SetChillerPrefs(day, dusk, night)` | 每项 `true`／`false`／`nil` | 分别表示需要、排斥、不关心对应热源；默认加热偏好为 `false, nil, true`，制冷偏好全 `false`。[hatchable.lua:13-25](../../../raw/current/scripts/components/hatchable.lua#L13-L25)、[hatchable.lua:84-104](../../../raw/current/scripts/components/hatchable.lua#L84-L104) |
| `StartUpdating()`、`StopUpdating()` | 周期 `TUNING.HATCH_UPDATE_PERIOD` | start 防重入且拒绝 `dead`／`hatch`；stop 只取消周期任务，不清进度、不换状态。[hatchable.lua:68-82](../../../raw/current/scripts/components/hatchable.lua#L68-L82) |
| `Delay(time)` | 延迟秒数 | 置 `delay=true`，到时恢复；期间 `OnUpdate` 直接返回，不累计任何进度或不适。[hatchable.lua:63-66](../../../raw/current/scripts/components/hatchable.lua#L63-L66)、[hatchable.lua:108-111](../../../raw/current/scripts/components/hatchable.lua#L108-L111) |

## 关键流程

### 热源与状态推进

1. 在 `TUNING.HATCH_CAMPFIRE_RADIUS` 内查有 `HASHEATER`、无 `INLIMBO` 的实体。先调用 `heater:GetHeat(inst)`，再检查是否放热或吸热，合计有效返回值。总和大于零视为有加热，小于零视为有制冷，等于零两者皆无；多个来源可互相抵消。[hatchable.lua:106-132](../../../raw/current/scripts/components/hatchable.lua#L106-L132)
2. 按 `TheWorld.state.phase` 取两类偏好，逐项更新 `toohot`／`toocold`。例如需要加热却无正总和会过冷，排斥制冷却有负总和也会过冷。[hatchable.lua:98-152](../../../raw/current/scripts/components/hatchable.lua#L98-L152)
3. `unhatched` 仅在“存在且需要加热”或“存在且需要制冷”时累计 `progress`；否则清零。达到 `cracktime` 时清零并回调 `crack`，本轮立即返回。这一阶段不会累计不适或触发 `dead`。[hatchable.lua:154-165](../../../raw/current/scripts/components/hatchable.lua#L154-L165)
4. 非 `unhatched` 阶段按冷热标志进入 `uncomfy` 或 `comfy`；同为不适但冷热标志改变时也强制通知。舒适时先减少 `discomfort` 至零，归零的该轮即可增加 `progress`；不适时只增加 `discomfort`，保留原进度。分别达到 `hatchtime`／`hatchfailtime` 后停止更新并通知 `hatch`／`dead`。[hatchable.lua:167-189](../../../raw/current/scripts/components/hatchable.lua#L167-L189)

### 保存、恢复与暂停

`OnSave` 保存状态、进度、不适及冷热标志，不保存任务是否运行、`delay`、回调或配置时间。`OnLoad(data)` 有数据时恢复字段，缺失状态回退为 `comfy`；随后重放非 `unhatched` 状态回调，再尝试 `StartUpdating()`。因此“存前暂停”不由组件自动恢复。[hatchable.lua:192-222](../../../raw/current/scripts/components/hatchable.lua#L192-L222)

原版把背包暂停落实在 `inventoryitem` 的放下／放入回调，并在 prefab 的 `OnLoadPostPass` 中再次检查 `IsHeld()`，避免组件加载顺序导致背包中的蛋继续更新。撤销孵化更新用 `StopUpdating()`；组件没有专用移除清理钩子。[tallbirdegg.lua:58-78](../../../raw/current/scripts/prefabs/tallbirdegg.lua#L58-L78)、[tallbirdegg.lua:218-228](../../../raw/current/scripts/prefabs/tallbirdegg.lua#L218-L228)、[hatchable.lua:1-224](../../../raw/current/scripts/components/hatchable.lua#L1-L224)

## 典型接入

### 高脚鸟蛋：分阶段换实体并等待玩家靠近

服务端 `defaultfn` 配置三个孵化时间、日／昏／夜偏好 `false, nil, true`，注册回调并启动。`crack` 回调生成 `tallbirdegg_cracked` 后移除旧蛋；裂蛋构造把初始状态设为 `comfy`，配置 `playerprox` 的靠近回调。注册名分别为 `tallbirdegg`、`tallbirdegg_cracked`。[tallbirdegg.lua:104-126](../../../raw/current/scripts/prefabs/tallbirdegg.lua#L104-L126)、[tallbirdegg.lua:200-255](../../../raw/current/scripts/prefabs/tallbirdegg.lua#L200-L255)、[tallbirdegg.lua:284-286](../../../raw/current/scripts/prefabs/tallbirdegg.lua#L284-L286)

进入 `hatch` 仅表示组件计时完成：宿主 `CheckHatch` 还要求玩家靠近、蛋未被持有且位于可视地面，才生成 `smallbird` 并移除蛋。玩家靠近或放下蛋会重试检查，不能把 `state == "hatch"` 等同于幼体已经出现。[tallbirdegg.lua:24-47](../../../raw/current/scripts/prefabs/tallbirdegg.lua#L24-L47)、[tallbirdegg.lua:58-65](../../../raw/current/scripts/prefabs/tallbirdegg.lua#L58-L65)

### 岩浆虫卵：全阶段需要加热

`lavae_egg`／`lavae_egg_cracked` 共用服务端配置，把三个阶段的加热偏好均设为 `true`；裂蛋同样从 `comfy` 开始并等待玩家靠近。`hatch` 回调经过持有与距离检查后播放孵化动画，动画结束由宿主生成牙齿并尝试通过牙齿的 `petleash` 生成宠物，最后移除卵。[lavae_egg.lua:27-48](../../../raw/current/scripts/prefabs/lavae_egg.lua#L27-L48)、[lavae_egg.lua:58-65](../../../raw/current/scripts/prefabs/lavae_egg.lua#L58-L65)、[lavae_egg.lua:139-192](../../../raw/current/scripts/prefabs/lavae_egg.lua#L139-L192)

## 常见陷阱与限制

- 偏好全部设为 `nil` 不会让完整蛋自动破壳：`unhatched` 必须存在被需要的加热或制冷。若设计为直接孵化阶段，应由宿主明确初始化对应状态，原版裂蛋使用 `comfy`。[hatchable.lua:154-165](../../../raw/current/scripts/components/hatchable.lua#L154-L165)、[tallbirdegg.lua:237-250](../../../raw/current/scripts/prefabs/tallbirdegg.lua#L237-L250)
- `GetHeaterPref(phase)`／`GetChillerPref(phase)` 的形参没有参与索引，传入阶段不能查询指定时段；实现始终使用世界当前 `phase`。[hatchable.lua:98-104](../../../raw/current/scripts/components/hatchable.lua#L98-L104)
- 连续调用 `Delay` 不会取消先前的恢复任务，较早任务可提前解除较晚设置的延迟；不要把它当作可续期的单任务计时器。[hatchable.lua:63-66](../../../raw/current/scripts/components/hatchable.lua#L63-L66)

## 验证记录

- 源码审阅：2026-10-26，Steam buildid 26148036；覆盖组件全部实现、状态阈值与热源判定、任务暂停与存读，及高脚鸟蛋／岩浆虫卵的服务端构造、状态回调、玩家靠近与背包消费者。
- 游戏运行：未运行；未实测冷热来源组合、回调换实体时序和存读后的实际表现；未审阅各类 heater 的具体输出算法及宠物内部生成机制。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 通过，格式／大小无问题，源码链接无错误或警告；正文无新增 Lua 示例，未运行语法探针。

- 版本沿用：2026-10-28 原始哈希等价批量沿用-70（26148036→26230496，本页 sources 所列文件原始字节全部未变），页内未决项零标记、符号反查无命中、严格链接通过；批次依据见 [[updates/build-26230496/index#wiki-沿用记录]]。
相关：[[reference/components/heater]]、[[reference/components/inventoryitem]]、[[reference/components/playerprox]]。
