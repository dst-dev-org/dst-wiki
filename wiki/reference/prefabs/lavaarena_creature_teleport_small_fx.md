---
title: "熔炉生物传送出生特效（lavaarena_creature_teleport_small_fx）"
created: "2026-10-29"
updated: "2026-11-01"
type: prefab
summary: "熔炉（Forge）生物登场传送与场地刷怪桩的表现层：联网代理壳＋非专服本地动画实体，音量按同帧实例数衰减；同文件 6 个注册名（teleport small/medium＋spawnerdecor 4 种）；生成调用在 lavaarena 事件数据包，脚本树内仅依赖声明。"
aliases: ["lavaarena_creature_teleport_small_fx", "lavaarena_creature_teleport_medium_fx", "lavaarena_spawnerdecor_fx_small", "lavaarena_spawnerdecor_fx_1", "lavaarena_spawnerdecor_fx_2", "lavaarena_spawnerdecor_fx_3", "lavaarena_creature_spawn_fx", "熔炉传送特效", "熔炉出生特效"]
tags: [prefab, fx, server-client]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/lavaarena_creature_spawn_fx.lua", "fx.lua", "prefabs/lavaarena_boarrior.lua", "prefabs/lavaarena_spawner.lua", "prefablist.lua", "entityscript.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 熔炉生物传送出生特效（lavaarena_creature_teleport_small_fx）

## 快速判断

- 职责与适用：熔炉（The Forge）生物登场传送与竞技场刷怪桩的**表现层 FX**：同文件 6 个注册名——传送动画 `lavaarena_creature_teleport_small_fx`／`_medium_fx`（anim `spawn_small`／`spawn_medium`）与摔跤火花装饰 `lavaarena_spawnerdecor_fx_small`／`_1`／`_2`／`_3`；联网代理壳＋非专服本地动画实体＋出生音效按同帧实例数衰减。[注册](../../../raw/current/scripts/prefabs/lavaarena_creature_spawn_fx.lua#L186-L191)
- 优先使用：查"熔炉刷怪传送／竞技场火花装饰"选本页；生成时机由事件数据决定（见下），本页只保证工厂行为面。
- 重要副作用：主模拟延迟 0-0.2 秒随机补生成 `lavaarena_creature_teleport_smoke_fx_1..3`（注册在 fx.lua 通用表，非本文件）；音量随同页实例数从 1 衰减到 0.5（teleport 满 10 个、decor 满 18 个）。[烟](../../../raw/current/scripts/prefabs/lavaarena_creature_spawn_fx.lua#L72-L74)、[调度](../../../raw/current/scripts/prefabs/lavaarena_creature_spawn_fx.lua#L100)、[音量](../../../raw/current/scripts/prefabs/lavaarena_creature_spawn_fx.lua#L36-L42)
- 修改边界：**脚本树内没有生成调用**——熔炉 prefab 的 master_postinit 全部来自 `event_server_data("lavaarena", ...)` 事件数据包（本快照不含）；in-tree 只有熔炉 prefab 的依赖声明。[事件数据加载](../../../raw/current/scripts/entityscript.lua#L77-L87)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `lavaarena_creature_teleport_small_fx` | makespawnfx(1, name, "spawn_small")：Transform＋Network＋FX → 非专服0 延迟任务中 PlayAnim（bank/build `lavaarena_creature_teleport`、bloom、播 `lava_arena/spawn` 按实例页 1 计数衰减、animover 自删）＋RegisterInstance → SetPristine → 客户端早退 → 主模拟 0-0.2s 后 SpawnSmokeFx＋persists=false＋1 秒自清 | 依赖声明 smoke fx 1-3。[工厂](../../../raw/current/scripts/prefabs/lavaarena_creature_spawn_fx.lua#L76-L109)、[注册](../../../raw/current/scripts/prefabs/lavaarena_creature_spawn_fx.lua#L186) |
| `lavaarena_creature_teleport_medium_fx` | 同工厂（实例页 1），anim `spawn_medium` | 中大型熔炉生物用。[注册](../../../raw/current/scripts/prefabs/lavaarena_creature_spawn_fx.lua#L187) |
| `lavaarena_spawnerdecor_fx_small` | makespawndecorfx(2, name, {spark_small_pre, spark_small_loop, spark_small_loop, spark_small_pst})：同壳非专服 PlayDecor（bank/build `lavaarena_wrestling_sparks`、DECOR＋NOCLICK 标签、可随父级、首段 PlayAnimation 其余 PushAnimation 显式 false、animqueueover 自删）＋RegisterInstance 页 2 → 主模拟 persists=false＋1 秒自清 | 无出生烟、无 spawn 声（用 `spawner_decor` 声）。[工厂](../../../raw/current/scripts/prefabs/lavaarena_creature_spawn_fx.lua#L151-L182)、[PlayDecor](../../../raw/current/scripts/prefabs/lavaarena_creature_spawn_fx.lua#L113-L149)、[注册](../../../raw/current/scripts/prefabs/lavaarena_creature_spawn_fx.lua#L188) |
| `lavaarena_spawnerdecor_fx_1`／`_2`／`_3` | 同工厂（实例页 3），anims `{spark_N_pre, spark_N_pst}`（N＝1/2/3） | 三种火花样式。[注册](../../../raw/current/scripts/prefabs/lavaarena_creature_spawn_fx.lua#L189-L191) |

页名取首个注册名。六个注册名均无 NAMES 条目，标题中文为已核验效果描述，非官方译名。注意 prefablist 登记的 "lavaarena_creature_spawn_fx"（[L636](../../../raw/current/scripts/prefablist.lua#L636)）是文件名条目，不是注册名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 实例计数与音量 | 文件局部 `instance_count[3]`（按 instance_page 分页）；RegisterInstance 计数、0.1 秒后自动减一（OnRemoveEntity 提前减）；ScaleVolume：count≥max→0.5，否则 1−(count/max)²×0.5 | 只在非专服分支计数（各模拟进程各自维护）；teleport max 10、decor max 18 | [计数](../../../raw/current/scripts/prefabs/lavaarena_creature_spawn_fx.lua#L20-L32)、[ScaleVolume](../../../raw/current/scripts/prefabs/lavaarena_creature_spawn_fx.lua#L36-L42)、[max 实参](../../../raw/current/scripts/prefabs/lavaarena_creature_spawn_fx.lua#L67) |
| 出生烟 | 主模拟 0-0.2 秒随机延迟 SpawnSmokeFx → smoke_fx_1..3 随机一个 | smoke fx 是 fx.lua MakeFx 通用表条目（bank/build `lavaarena_creature_teleport_smoke_fx`、anim smoke_N），非本文件注册 | [SpawnSmokeFx](../../../raw/current/scripts/prefabs/lavaarena_creature_spawn_fx.lua#L72-L74)、[调度](../../../raw/current/scripts/prefabs/lavaarena_creature_spawn_fx.lua#L100)、[fx.lua 注册](../../../raw/current/scripts/fx.lua#L908-L922) |
| 代理-本地实体模式 | 网络壳＋非专服0 延迟任务中 playfx（SetFromProxy 定位）播动画与声音；专服只有壳（1 秒自清），注释明言 dedicated 无需本地 fx | 与 collapse 系同构 | [makespawnfx](../../../raw/current/scripts/prefabs/lavaarena_creature_spawn_fx.lua#L76-L109)、[PlayAnim](../../../raw/current/scripts/prefabs/lavaarena_creature_spawn_fx.lua#L46-L70) |
| 消费者（事件数据） | 9 个熔炉 prefab 依赖声明：boarrior/beetletaur/peghook/trails 用 medium，boaron/turtillus/snapper/rhinodrill 用 small，spawner 用 spawnerdecor 4 种；事件数据包内的实际 SpawnPrefab 调用不可见，不能据依赖声明断定其全部生成点 | master_postinit 经 `event_server_data("lavaarena", ...)` 接线 | [boarrior 依赖](../../../raw/current/scripts/prefabs/lavaarena_boarrior.lua#L17)、[spawner 依赖](../../../raw/current/scripts/prefabs/lavaarena_spawner.lua#L9-L12)、[事件接线](../../../raw/current/scripts/prefabs/lavaarena_boarrior.lua#L65)、[加载实现](../../../raw/current/scripts/entityscript.lua#L77-L87) |
| 资源 | anim/lavaarena_creature_teleport.zip、anim/lavaarena_wrestling_sparks.zip；音效 `dontstarve/common/lava_arena/spawn` 与 `/spawner_decor` | 声明与使用已核验，包内容未解包 | [声明](../../../raw/current/scripts/prefabs/lavaarena_creature_spawn_fx.lua#L1-L9) |

## 典型接入

### 复用"带音量衰减的成批出场特效"

- 前提与选择：主模拟 SpawnPrefab 对应注册名后摆位即可；动画与声音由非专服各端自建。
- 生效与撤销：壳 1 秒自清、动画 animover／animqueueover 自删；出生烟延迟随机出现，别把它当即时报点。
- 原版依据：in-tree 无调用方（生成在事件数据包），以上为工厂行为面。

```lua
local fx = SpawnPrefab("lavaarena_creature_teleport_medium_fx")
fx.Transform:SetPosition(x, y, z)
```

- 调整与验证：同帧大量生成时音量自动衰减，但计数是各端独立的文件局部表，跨端不同步。

## 常见陷阱与限制

- **边界："lavaarena_creature_spawn_fx" 不是注册名**：文件名与 prefablist 登记名都是文件条目，`SpawnPrefab("lavaarena_creature_spawn_fx")` 会失败；实际 6 个注册名见注册表（[注册行](../../../raw/current/scripts/prefabs/lavaarena_creature_spawn_fx.lua#L186-L191)）。
- **边界：脚本树内查不到生成调用**：熔炉 prefab 接入 `lavaarena_event_server` 事件数据包，本快照不含；in-tree 依赖声明只能证明加载关系，实际生成点未核验。
- **边界：音量衰减按端独立**：instance_count 是各模拟进程的文件局部表，专服不计数；客户端与服务端听到的音量可能不同。[计数](../../../raw/current/scripts/prefabs/lavaarena_creature_spawn_fx.lua#L20-L32)
- **边界：动画是否重复未核验**：teleport 动画单参播放（未显式传循环参数）；decor 后续段 PushAnimation 显式 false，设计预期播完自删，别依赖循环。

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496；192 行工厂全文（两工厂、实例计数、音量、出生烟、decor 队列、端分界）、fx.lua smoke 注册、9 个消费者依赖声明与事件接线、prefablist 登记名、NAMES 检索（无命中）已核验；事件数据包内实际调用点与参数、anim zip 内容、晚加入时序未核验。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、label-check 通过（结果见本批交付报告）。

返回 [[reference/prefabs/fx-event-index|节庆、小游戏与事件模式]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-1）。
