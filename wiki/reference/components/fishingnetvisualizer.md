---
title: fishingnetvisualizer
created: "2026-10-21"
updated: "2026-10-28"
type: component
summary: "渔网的表现与捕获执行实体：抛收网运动、动画事件、溅水特效，以及把附近可拾取物卷入网中再撒回投掷者脚边。"
aliases: ["渔网表现", "渔网视觉", "投网动画", "网捕获", "fishingnetvisualizer"]
tags: [component, state-machine, network-sync]
dst_build: "steam-buildid:26230496"
sources: ["components/fishingnetvisualizer.lua", "components/fishingnet.lua", "prefabs/fishingnetvisualizer.lua", "prefabs/fishingnetvisualizerfx.lua", "prefabs/fishingnet.lua", "stategraphs/SGfishingnetvisualizer.lua", "stategraphs/SGwilson.lua", "stategraphs/SGantchovies.lua", "components/fishschool.lua", "prefabs/antchovies.lua", "prefabs/boatpatch.lua", "prefabs/player_common.lua", "tuning.lua", "vecutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# fishingnetvisualizer

## 快速判断

- 职责与适用：由 `fishingnet:CastNet` 生成的一次性服务端表现实体组件，独占抛网／开网／收网／撒落的运动与计时，并把“网里装了什么”落成真实物品；不含动作选择、耐久与物品定义。[fishingnet.lua:6-9](../../../raw/current/scripts/components/fishingnet.lua#L6-L9)
- 优先使用：克隆 `prefabs/fishingnetvisualizer.lua` 的表现配置，`SpawnPrefab("fishingnetvisualizer")` 后只调 `BeginCast(doer, x, z)`，其余交给 `SGfishingnetvisualizer` 每帧驱动；原版唯一入口就是 `CastNet`。[prefabs/fishingnetvisualizer.lua:45](../../../raw/current/scripts/prefabs/fishingnetvisualizer.lua#L45)、[SGfishingnetvisualizer.lua:86](../../../raw/current/scripts/stategraphs/SGfishingnetvisualizer.lua#L86
- 重要副作用：改写投掷者的 `AnimState` 手臂显隐；把半径 2 内所有 `inventoryitem` 实体 `RemoveFromScene`，收网时再 `ReturnToScene` 并抛落；向半径 4 内所有实体广播 `on_pre_net`。[:31-34](../../../raw/current/scripts/components/fishingnetvisualizer.lua#L31-L34)、[:100-114](../../../raw/current/scripts/components/fishingnetvisualizer.lua#L100-L114)
- 修改边界：纯服务端；宿主 prefab 全程没有 `entity:AddNetwork()`，客户端既无该实体也无 SG 推进；组件不监听 `onremove`，无 `OnSave`／`OnLoad`，也不做超时兜底。[prefab:11-35](../../../raw/current/scripts/prefabs/fishingnetvisualizer.lua#L11-L35)

## 关键接口

本节起单独的 `[:行号]` 链接默认指 `components/fishingnetvisualizer.lua`；跨文件引用在方括号内标文件短名。

### 已核验字段与默认值（构造时写入，可被模组直接改写）

| 字段 | 默认值 | 含义与消费点 |
|---|---|---|
| `velocity` / `retrieve_velocity` | `10` / `12`（世界单位每秒） | 去程沿 `dir` 前进、回程朝投掷者的水平速度。[:37-49](../../../raw/current/scripts/components/fishingnetvisualizer.lua#L37-L49) |
| `collect_radius` | `2` | 捕获半径；`on_pre_net` 半径为其加 `TUNING.MAX_FISH_SCHOOL_SIZE`（=4）。[:102](../../../raw/current/scripts/components/fishingnetvisualizer.lua#L102)、[tuning.lua:4042](../../../raw/current/scripts/tuning.lua#L4042 |
| `collect_velocity` / `max_captured_entity_collect_distance` | `4` / `0.25` | 开网阶段拉向网心的速度与单个物品的最大位移上限。[:79-94](../../../raw/current/scripts/components/fishingnetvisualizer.lua#L79-L94) |
| `retrieve_distance` / `distance_to_play_open_anim` | `1.5` / `1.65` | 距投掷者小于前者即判定收网完成；剩余距离进入后者推一次 `play_throw_pst`。[:200-204](../../../raw/current/scripts/components/fishingnetvisualizer.lua#L200-L204)、[:51-54](../../../raw/current/scripts/components/fishingnetvisualizer.lua#L51-L54) |
| `captured_entities` / `captured_entities_collect_distance` | 数组／按实体索引的表 | 捕获清单与各自已拉近距离；只增不重置，靠一次性收网消化。[:11-12](../../../raw/current/scripts/components/fishingnetvisualizer.lua#L11-L12) |

### 方法

| 接口 | 作用与证据 |
|---|---|
| `BeginCast(thrower, target_x, target_z)` | 把网置到投掷者 `(x, 0, z)`，按 0.5 起始偏移算 `distance_remaining`／`total_distance` 与归一化方向，改手臂显隐并 `FacePoint`；要求 `thrower` 有 `AnimState`（玩家）。[:16-35](../../../raw/current/scripts/components/fishingnetvisualizer.lua#L16-L35) |
| `UpdateWhenMovingToTarget(dt)` | 沿 `dir` 推进 `velocity*dt`；到点向自身推 `begin_opening`；用绝对 y 摆抛物线。[:37-66](../../../raw/current/scripts/components/fishingnetvisualizer.lua#L37-L66) |
| `CalculateY(x, x_span, scale)` | `(x² - (x_span/2)²) * -scale`：两端为 0，中点最高。[:68-71](../../../raw/current/scripts/components/fishingnetvisualizer.lua#L68-L71) |
| `BeginOpening()` | 半径 4 内广播 `on_pre_net(self.inst)`，再在半径 2 内收集带 `inventoryitem` 的实体（排除自身）。[:100-114](../../../raw/current/scripts/components/fishingnetvisualizer.lua#L100-L114) |
| `UpdateWhenOpening(dt)` | 逐帧把捕获物拉向网心：有 `Physics` 走 `TeleportOffset`，否则 `Transform:OffsetPosition`。[:73-98](../../../raw/current/scripts/components/fishingnetvisualizer.lua#L73-L98) |
| `BeginRetrieving()` | 向投掷者推 `begin_retrieving`，并把所有有效捕获物 `RemoveFromScene`。[:156-165](../../../raw/current/scripts/components/fishingnetvisualizer.lua#L156-L165) |
| `UpdateWhenRetrieving(dt)` | 朝投掷者飞回；到位或进入 `retrieve_distance` 时向自身推 `begin_final_pickup`，并记下反向单位向量供撒落。[:186-209](../../../raw/current/scripts/components/fishingnetvisualizer.lua#L186-L209) |
| `DropItem(item, last_dir_x, last_dir_z, idx)` | 第 `idx` 个物品延迟 `idx*0.25 + 0.15` 秒 `ReturnToScene`、推 `on_release_from_net`，落到投掷者侧后方约 1 单位、高 0.65 处并给 `(0,-0.25,0)` 初速。[:116-154](../../../raw/current/scripts/components/fishingnetvisualizer.lua#L116-L154) |
| `BeginFinalPickup()` | 向投掷者推 `begin_final_pickup`，逐个 `DropItem`，恢复手臂显隐，最后 `inst:Remove()`。[:167-184](../../../raw/current/scripts/components/fishingnetvisualizer.lua#L167-L184) |

### 与 fishingnet 的分工与事件契约

`fishingnet` 只做“物品 → 目标点 → 生成表现实体”，本组件接管此后全部流程，两者无反向调用、不共用字段：改捕获规则只动本组件，改耐久／动作条件不必动它。

| 事件 | 推送方 → 接收方 | 时机与作用 |
|---|---|---|
| `play_throw_pst` | 本组件 → 宿主 SG `casting` | 剩余距离 ≤ 1.65 时播 `throw_pst`，只推一次。[:51-54](../../../raw/current/scripts/components/fishingnetvisualizer.lua#L51-L54)、[SG:17-19](../../../raw/current/scripts/stategraphs/SGfishingnetvisualizer.lua#L17-L19) |
| `begin_opening` | 本组件 → 宿主 SG `casting` | 抵达目标后切 `opening`。[:46-46](../../../raw/current/scripts/components/fishingnetvisualizer.lua#L46 |
| `begin_retrieving` | 本组件 → 投掷者 | 投掷者 SG 切 `cast_net_retrieving`；当前仅 `SGwilson` 有此状态。[SGwilson.lua:13906-13908](../../../raw/current/scripts/stategraphs/SGwilson.lua#L13906-L13908) |
| `begin_final_pickup` | 本组件 → 宿主 SG `retrieving` | 切 `final_pickup`。[:201-201](../../../raw/current/scripts/components/fishingnetvisualizer.lua#L201)、[SG:74-74](../../../raw/current/scripts/stategraphs/SGfishingnetvisualizer.lua#L74 |
| `begin_final_pickup` | 本组件 → 投掷者 | 投掷者 SG 切 `cast_net_release`。[:169-169](../../../raw/current/scripts/components/fishingnetvisualizer.lua#L169)、[SGwilson:13940-13942](../../../raw/current/scripts/stategraphs/SGwilson.lua#L13940-L13942) |
| `on_pre_net` | 本组件 → 半径 4 内所有实体 | 鱼群据此换算成鱼（`fishschool` 是本组件之外唯一的原版监听者）。[fishschool.lua:11](../../../raw/current/scripts/components/fishschool.lua#L11 |
| `on_release_from_net` | 本组件 → 每个撒落物品 | 原版仅 `SGantchovies` 监听，但该 SG 在 `antchovies` prefab 中被注释掉，实际无消费者。[SGantchovies.lua:28](../../../raw/current/scripts/stategraphs/SGantchovies.lua#L28)、[prefabs/antchovies.lua:37](../../../raw/current/scripts/prefabs/antchovies.lua#L37 |

## 典型接入

### 原版链路：投网 → 开网 → 收网 → 撒落

- 前提与选择：宿主由 `SpawnPrefab("fishingnetvisualizer")` 创建；prefab 在主机端设置 SG（默认状态 `casting`）与组件，并带 `NOCLICK`、`ignorewalkableplatforms`、`SetFourFaced`、排序 5、3×2 地面阴影。[:14-40](../../../raw/current/scripts/prefabs/fishingnetvisualizer.lua#L14-L40)
- 生效与撤销：`casting` 播 `throw_pre`+`throw_loop` 并每帧调 `UpdateWhenMovingToTarget`；`opening` 进入时 `BeginOpening` 并生成 `fishingnetvisualizerfx` 溅水，`animover` 切 `retrieving`；`retrieving` 先 `BeginRetrieving`、转向投掷者、播 `pull_loop`，到位切 `final_pickup` 撒落并自删。撤销只有这一条路径，无超时或投掷者失效兜底。[:8-83](../../../raw/current/scripts/stategraphs/SGfishingnetvisualizer.lua#L8-L83)
- 单位：位置为世界坐标；`velocity` 类为世界单位每秒，半径／距离类常量为世界单位；掉落间隔 0.25 秒是方法内字面量，不可配置。[:116-154](../../../raw/current/scripts/components/fishingnetvisualizer.lua#L116-L154)
- 原版依据：鱼群在 `on_pre_net` 里按 `fish_level` 在网的位置生成 `antchovies` 并把鱼量清零；这些新实体带 `inventoryitem`，会被紧随其后的半径 2 查询收进网。[fishschool:29-44](../../../raw/current/scripts/components/fishschool.lua#L29-L44)、[antchovies:35-35](../../../raw/current/scripts/prefabs/antchovies.lua#L35

### 模组复用：克隆表现链时的最小改动点

- 前提与选择：想“看得见”必须自建联网 prefab（对照 `fishingnetvisualizerfx` 有 `AddNetwork()`）或改成客户端 FX；直接复用原 prefab 只会得到主机端实体。
- 生效与撤销：调用方要保证流程能走到 `BeginFinalPickup`——`thrower` 有效且中途不被移除，否则捕获物留在场景外；复用前建议在宿主 `onremove` 里补一条把 `captured_entities` 逐个 `ReturnToScene` 的清理钩子（原版没有）。
- 原版依据：`fishingnetvisualizerfx` 播 `hit` 后 `animover` 自删，可作为一次性 FX 的最小参考。[prefabs/fishingnetvisualizerfx.lua:16-28](../../../raw/current/scripts/prefabs/fishingnetvisualizerfx.lua#L16-L28)

## 常见陷阱与限制

- 强制删除会吞物品：`BeginRetrieving` 已把捕获物 `RemoveFromScene`，只有 `BeginFinalPickup` 的 `DropItem` 才 `ReturnToScene`；组件不监听宿主 `onremove`，网被 `inst:Remove()`、投掷者消失或流程卡住时物品停在场景外。[:159-164](../../../raw/current/scripts/components/fishingnetvisualizer.lua#L159-L164)、[:167-184](../../../raw/current/scripts/components/fishingnetvisualizer.lua#L167-L184)
- 投掷者失效即报错：`UpdateWhenRetrieving` 每帧读 `self.thrower.Transform`，`BeginFinalPickup` 读 `self.thrower.AnimState` 复原手臂；`thrower` 为 `nil` 或已移除会抛错并中断收网。[:188-188](../../../raw/current/scripts/components/fishingnetvisualizer.lua#L188)、[:180-181](../../../raw/current/scripts/components/fishingnetvisualizer.lua#L180-L181)
- 抛物线用绝对高度：`BeginCast` 把网 y 置 0，此后每帧用 `CalculateY` 的绝对值覆盖 y，从不读投掷者 y；顶点为 `(total/2)² * scale`，长距离投掷会抬得很高，船上／高台起飞与落点会偏离视觉基线。[:18-19](../../../raw/current/scripts/components/fishingnetvisualizer.lua#L18-L19)、[:60-65](../../../raw/current/scripts/components/fishingnetvisualizer.lua#L60-L65)、[:198-198](../../../raw/current/scripts/components/fishingnetvisualizer.lua#L198
- 近距投掷会截断 `throw_pst`：抵达与 `distance_remaining <= 1.65` 可能同帧命中，代码先推 `begin_opening` 再推 `play_throw_pst`，`casting` 刚播 `throw_pst` 就切 `opening`。[:46-54](../../../raw/current/scripts/components/fishingnetvisualizer.lua#L46-L54)、[SG:17-21](../../../raw/current/scripts/stategraphs/SGfishingnetvisualizer.lua#L17-L21)
- 捕获范围写死：半径 2、只收 `inventoryitem`，船、建筑、鱼群本体都不进网；鱼群要靠 `on_pre_net` 自行换算。扩大范围需改 `collect_radius` 并同步 `on_pre_net` 半径的加法。[:102-113](../../../raw/current/scripts/components/fishingnetvisualizer.lua#L102-L113)
- 原版内容未完成：prefab 无网络组件，Scrapbook 与配方条目被注释，`player_common.lua` 注明 “Unfinished, bad file. For unused fishingnet”；`prefabs/boatpatch.lua` 只把它列入 prefab 依赖却从未生成。不要把该链当稳定接口。[player_common.lua:2008](../../../raw/current/scripts/prefabs/player_common.lua#L2008)、[prefabs/boatpatch.lua:14](../../../raw/current/scripts/prefabs/boatpatch.lua#L14)、[prefab:11-35](../../../raw/current/scripts/prefabs/fishingnetvisualizer.lua#L11-L35)

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 处理范围：锚点重定位 4 处（SGwilson 钓网段 +88、player_common −12、tuning +14，区间文本逐段比对一致，显示标签同步）；结论修订：无（钓网两版一致，与 player-hubs 地图散点结论相符）。
- 依据：player-hubs hunk 地图＋变更清单命中 stategraphs/SGwilson.lua、prefabs/player_common.lua、tuning.lua。

- 源码审阅：2026-10-21，`steam-buildid:25643519`。通读 `components/fishingnetvisualizer.lua` 全 211 行，并核验 `components/fishingnet.lua`、`prefabs/fishingnetvisualizer.lua`、`prefabs/fishingnetvisualizerfx.lua`、`prefabs/fishingnet.lua`、`SGfishingnetvisualizer.lua`（全 86 行）、`SGwilson.lua`（`cast_net*`）、`SGantchovies.lua`、`components/fishschool.lua`、`prefabs/antchovies.lua`、`prefabs/boatpatch.lua`、`player_common.lua`、`tuning.lua`、`vecutil.lua`。主源码 SHA256 `a5e52a0c34866cf987db764d7d58ee160b554e00cb53a14215e2c813ffca01f9`。
- 游戏运行：未运行；未覆盖真实投网手感、多人同步与投掷者中途下线。
- 结构检查：`validate_page.py` 通过，0 error；`validate_source_links.py --strict` 通过，本文 52 处行锚点、47 个唯一目标、12 个源文件，0 error／0 warning。本页不含完整 Lua 片段，未做语法探针。

相关：[[reference/components/fishingnet]]、[[reference/components/fishingrod]]、[[reference/components/fishable]]
