---
title: "raindomewatcher"
created: "2026-10-21"
updated: "2026-10-28"
type: component
summary: "在本地激活玩家实体上每帧查询雨棚，维护 underdome 状态并发出 enterraindome／underraindomes／exitraindome，供天气音效、HUD 与滤镜读取。"
aliases: ["雨棚监听", "玩家雨棚状态", "是否在雨棚下", "raindomewatcher"]
tags: [component, lifecycle, callback, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/raindomewatcher.lua", "components/raindome.lua", "prefabs/player_common.lua", "components/weather.lua", "components/caveweather.lua", "screens/playerhud.lua", "widgets/raindomeover.lua", "components/colourcube.lua", "componentutil.lua", "prefabs/wilson.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# raindomewatcher

## 快速判断

- 职责与适用：玩家侧的**状态传感器**，只回答“本地激活玩家当前是否在任一活动雨棚内”，不授予免疫、不改半径、不参与存档。[构造](../../../raw/current/scripts/components/raindomewatcher.lua#L1-L5) [更新](../../../raw/current/scripts/components/raindomewatcher.lua#L11-L24)
- 优先使用：轮询用 `IsUnderRainDome()`；需要“进入／离开”边沿时监听 `enterraindome`／`exitraindome`；需要每帧雨棚列表时监听 `underraindomes`。[接口](../../../raw/current/scripts/components/raindomewatcher.lua#L7-L9) [事件](../../../raw/current/scripts/components/raindomewatcher.lua#L14-L23)
- 重要副作用：`underraindomes` 在棚内**每帧**推送，payload 是当帧新建的雨棚实体数组；监听者要轻量，不能在里面做重查询。[L19](../../../raw/current/scripts/components/raindomewatcher.lua#L19)
- 修改边界：只加在 `ThePlayer`（本地激活玩家）上，随激活／反激活增删；其它服务端玩家没有这份组件，不能拿它做服务端权威判定。[宿主](../../../raw/current/scripts/prefabs/player_common.lua#L722-L730) [激活条件](../../../raw/current/scripts/prefabs/player_common.lua#L1036-L1041)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `underdome` | boolean | 构造时为 `false`；每次 `OnUpdate` 按查询结果翻转，是 `IsUnderRainDome()` 与边沿事件的唯一状态。[L3](../../../raw/current/scripts/components/raindomewatcher.lua#L3) [L15-L22](../../../raw/current/scripts/components/raindomewatcher.lua#L15-L22) |
| `IsUnderRainDome()` | boolean | 读 `underdome`；本帧状态，下一次更新前不保证与位置同步。[L7-L9](../../../raw/current/scripts/components/raindomewatcher.lua#L7-L9) |
| `OnUpdate(dt)` | 内部 | 每帧取玩家 x/z，调全局 `GetRainDomesAtXZ`；有棚则必要时发 `enterraindome` 并每帧发 `underraindomes`，无棚且此前在棚内则发 `exitraindome`。[L11-L24](../../../raw/current/scripts/components/raindomewatcher.lua#L11-L24) |
| 事件 `enterraindome` | 无参，玩家实体上 | 从“不在棚内”变为“在棚内”的那一帧，先于同帧 `underraindomes`。[L15-L18](../../../raw/current/scripts/components/raindomewatcher.lua#L15-L18) |
| 事件 `underraindomes` | `(player, domes)` | 在棚内的每一帧都发；`domes` 是 `GetRainDomesAtXZ` 的返回数组，含所有覆盖玩家的活动棚。[L19](../../../raw/current/scripts/components/raindomewatcher.lua#L19) |
| 事件 `exitraindome` | 无参，玩家实体上 | 从“在棚内”变为“不在棚内”时发一次；组件被移除时**不会**补发。[L20-L23](../../../raw/current/scripts/components/raindomewatcher.lua#L20-L23) |
| 全局查询依赖 | `GetRainDomesAtXZ(x, z)` | 由 `components/raindome.lua` 定义、`componentutil` 提前加载；返回空表即不在棚内。[查询](../../../raw/current/scripts/components/raindome.lua#L29-L43) [加载](../../../raw/current/scripts/componentutil.lua#L1) |

### 状态开关与事件顺序

- 没有公开的 `Enable`／`Disable`；所谓“开关”就是 `underdome` 的布尔翻转：0 个棚 → `false`，≥1 个棚 → `true`。[L14-L22](../../../raw/current/scripts/components/raindomewatcher.lua#L14-L22)
- 边沿顺序固定为：进入帧先 `enterraindome`，再 `underraindomes`；离开帧只发 `exitraindome`，该帧不会再发 `underraindomes`。[L15-L23](../../../raw/current/scripts/components/raindomewatcher.lua#L15-L23)
- 事件是 `PushEvent` 本地派发，不跨网络；服务端玩家实体上的推送不会通过网络复制到远程客户端，客户端 UI 必须在客户端实体上监听。[本地事件](../../../raw/current/scripts/components/raindomewatcher.lua#L17-L22)
- **每帧查询依赖客户端活动半径**：`GetRainDomesAtXZ` 的客户端 `_maxsize` 由 `_activeradiusdirty` 维护，查询结果要等网络半径到达后才完整。[客户端维护](../../../raw/current/scripts/components/raindome.lua#L69-L78)

### 谁写、谁读、何时撤

| 环节 | 位置与触发 | 结果 |
|---|---|---|
| 写 | 构造 | `underdome=false`，立即 `StartUpdatingComponent`，无初始查询。[L1-L5](../../../raw/current/scripts/components/raindomewatcher.lua#L1-L5) |
| 写 | 每帧 `OnUpdate` | 覆写 `underdome`，按边沿推送三个事件之一。[L11-L24](../../../raw/current/scripts/components/raindomewatcher.lua#L11-L24) |
| 读 | `weather`／`caveweather` 天气音效；`playerhud` 雨滴特效；`colourcube` 鱼眼；`widgets/raindomeover` 罩层 | 方法轮询与事件监听两种读法并存。[天气](../../../raw/current/scripts/components/weather.lua#L973-L982) [洞穴天气](../../../raw/current/scripts/components/caveweather.lua#L655-L663) [HUD](../../../raw/current/scripts/screens/playerhud.lua#L143-L149) [罩层](../../../raw/current/scripts/widgets/raindomeover.lua#L24-L38) [滤镜](../../../raw/current/scripts/components/colourcube.lua#L322-L328) |
| 撤 | `RemoveActivePlayerComponents` | 玩家反激活／HUD 注销时 `RemoveComponent("raindomewatcher")`，没有 `OnRemoveFromEntity` 清理；不补发 `exitraindome`。[移除](../../../raw/current/scripts/prefabs/player_common.lua#L732-L738) [调用](../../../raw/current/scripts/prefabs/player_common.lua#L1203-L1205) |

## 典型接入

### 原版宿主：本地激活玩家

- 文件与端：组件没有独立 prefab 注册名，挂在所有玩家角色共用的 `prefabs/player_common.lua` 上（如 `wilson`）；`AddActivePlayerComponents` 只在 `inst == ThePlayer` 时调用，因此是本地激活玩家组件，不是每个服务端玩家都有的常驻组件。[装配](../../../raw/current/scripts/prefabs/player_common.lua#L722-L730) [调用点](../../../raw/current/scripts/prefabs/player_common.lua#L1001-L1041) [角色注册](../../../raw/current/scripts/prefabs/wilson.lua#L170)
- 生效：组件添加后从下一帧开始更新；出生、传送、棚开关都通过下一帧查询体现，构造时没有即时判定。[L1-L5](../../../raw/current/scripts/components/raindomewatcher.lua#L1-L5)
- 撤销：注销 HUD 与玩家离开两条路径都会 `RemoveActivePlayerComponents`；组件没有存档、没有网络状态，重新激活时重建为 `false`。[L732-L738](../../../raw/current/scripts/prefabs/player_common.lua#L732-L738) [离开](../../../raw/current/scripts/prefabs/player_common.lua#L1203-L1205)
- 原版消费者：天气组件用它把雨声压到 0.1 并播放屏障音；HUD 用它抑制雨滴溅射；罩层与滤镜分别用事件和方法。[天气](../../../raw/current/scripts/components/weather.lua#L978-L982) [罩层](../../../raw/current/scripts/widgets/raindomeover.lua#L24-L38)

### 模组读取“我在雨棚下吗”

- 前提与选择：客户端 UI 用事件（进入／离开），逐帧逻辑用 `IsUnderRainDome()`；不要为了读状态自己 `AddComponent("raindomewatcher")`，它属于激活玩家生命周期，重复添加也不会多一份监听。[接口](../../../raw/current/scripts/components/raindomewatcher.lua#L7-L9)
- 生效与撤销：监听 `ThePlayer` 上的事件即可；玩家反激活时组件被移除且不发 `exitraindome`，自定义 UI 要在 `playerdeactivated` 或 HUD 销毁路径自行收尾。
- 服务端注意：专用服务器不经过这条本地激活路径；服务端要判定请直接调 `IsUnderRainDomeAtXZ(x, z)`，而不是找这个组件。[查询](../../../raw/current/scripts/components/raindome.lua#L45-L59)
- 原版依据：`colourcube` 在玩家激活时注册 `enterraindome`／`exitraindome`，反激活时移除回调，可作为客户端监听生命周期的参照。[L322-L347](../../../raw/current/scripts/components/colourcube.lua#L322-L347)

## 常见陷阱与限制

- **每帧事件不是状态变化事件**：`underraindomes` 在棚内每帧都发，只有 `enterraindome`／`exitraindome` 是边沿；把 `underraindomes` 当进入事件会反复触发。[L14-L23](../../../raw/current/scripts/components/raindomewatcher.lua#L14-L23)
- **payload 不能长期缓存**：`domes` 是当帧新建的数组，下一帧会被替换；需要跨帧用时保存实体并检查 `IsValid()`，不要保存数组等它更新。[L13](../../../raw/current/scripts/components/raindomewatcher.lua#L13)
- **移除无退出事件**：组件没有 `OnRemoveFromEntity`，`RemoveComponent` 也不发 `exitraindome`；依赖离开事件收尾的 UI 会在反激活路径留下状态。[L1-L5](../../../raw/current/scripts/components/raindomewatcher.lua#L1-L5) [移除](../../../raw/current/scripts/prefabs/player_common.lua#L732-L738)
- **不是服务端权威**：只加给 `ThePlayer`，服务端其它玩家没有该组件；用 `components.raindomewatcher` 判断所有玩家会漏。[宿主条件](../../../raw/current/scripts/prefabs/player_common.lua#L1036-L1041)
- **事件不跨网络**：`PushEvent` 是本地派发；服务端判定与客户端表现要各写一份，不能靠一个端的事件驱动另一个端。[L17-L22](../../../raw/current/scripts/components/raindomewatcher.lua#L17-L22)
- **首帧延迟**：`underdome` 初始为 `false`，进入雨棚后要等下一次 `OnUpdate` 才变真；在同一帧里先查状态再移动会读到旧值。[L3](../../../raw/current/scripts/components/raindomewatcher.lua#L3) [L11-L14](../../../raw/current/scripts/components/raindomewatcher.lua#L11-L14)
- **空棚时仍每帧更新**：即使全图没有活动雨棚，组件也每帧调用查询；查询在 `_maxsize <= 0` 时提前返回，但更新循环本身不会停。[L4](../../../raw/current/scripts/components/raindomewatcher.lua#L4) [早退](../../../raw/current/scripts/components/raindome.lua#L29-L31)

## 验证记录

- 增量复查：2026-10-25，Steam buildid 26148036；锚点重定位（caveweather +15 段、playerhud +1 段、wilson −1 段），raindomewatcher 消费块本身未变；C-08 的 rainfx 门控加在锚点上方不涉屏障音逻辑，R-05/P-137/P-83 变更区不涉本页语义，结论维持。
- 源码审阅：2026-10-21，Steam buildid 25643519；覆盖构造与更新循环、`underdome` 翻转与三个事件的顺序、玩家本地激活宿主的添加／移除与角色 prefab 注册、天气与洞穴音效、HUD 雨滴、罩层与滤镜四个消费者，以及 `GetRainDomesAtXZ` 的依赖来源。
- 未运行游戏；未覆盖：多人联机下客户端与服务端实体的实际事件时序、反激活／无缝换人时的 HUD 清理、多个雨棚重叠时的表现。
- 本页无完整 Lua 示例，未执行 Lua 语法检查。
- 结构检查：`validate_page.py` 无 issue（86 行，任务上限 180 行／14 KiB）；`validate_source_links.py --strict` 0 error／0 warning（50 处源码引用）。

相关：[[reference/components/raindome]]；[[reference/components/rainimmunity]]。
