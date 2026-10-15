---
title: "playertargetindicator"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "玩家屏外指示器组件：客户端把出屏玩家经 HUD:AddTargetIndicator 显示为目标标记；本快照内无任何挂载点。"
aliases: ["玩家目标指示器", "屏外玩家标记", "playertargetindicator"]
tags: [component, ui]
dst_build: "steam-buildid:26230496"
sources: ["components/playertargetindicator.lua", "components/hudindicatable.lua", "components/hudindicatablemanager.lua", "prefabs/player_common.lua", "screens/playerhud.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# playertargetindicator

## 快速判断

- 职责与适用：客户端组件，每帧维护"出屏玩家"列表：把可追踪且刚离开视锥的玩家加入 `HUD:AddTargetIndicator`，回到屏内或不可追踪时移除（[OnUpdate](../../../raw/current/scripts/components/playertargetindicator.lua#L53-L87)）。
- 优先使用：**本快照内没有可复用的接入方式**——全库 grep（2026-10-28）仅命中组件自身文件，无 `AddComponent("playertargetindicator")` 或组件读取者；相关工作机制见 [[reference/components/hudindicatablemanager]] 与 [[reference/components/hudindicatable]]。
- 重要副作用：监听 TheWorld 的 `unregister_hudindicatable` 事件清理已注销玩家（[监听](../../../raw/current/scripts/components/playertargetindicator.lua#L25)）；事件由 hudindicatable 注销时推送（[推送方](../../../raw/current/scripts/components/hudindicatable.lua#L21)）。
- 修改边界：`OnRemoveFromEntity` 移除全部指示器并置空列表（[清理](../../../raw/current/scripts/components/playertargetindicator.lua#L30-L39)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `OnUpdate()` | `-> 无` | 三段式：先剔除应移除的指示器，再从 `TheWorld.components.hudindicatablemanager.items` 挑选新目标，最后用 `entity:FrustumCheck()` 记录本帧屏内玩家（[实现](../../../raw/current/scripts/components/playertargetindicator.lua#L53-L87)）。 |
| `ShouldShowIndicator(target)` | `-> bool` | 要求 `target.components.hudindicatable:ShouldTrack(self.inst)` 且上一帧在屏内（[实现](../../../raw/current/scripts/components/playertargetindicator.lua#L43-L47)）。 |
| `ShouldRemoveIndicator(target)` | `-> bool` | 目标不再可追踪即移除（[实现](../../../raw/current/scripts/components/playertargetindicator.lua#L49-L51)）。 |
| `offScreenPlayers` | `table`（实体数组） | 当前已显示指示器的玩家集合（[字段](../../../raw/current/scripts/components/playertargetindicator.lua#L17)）。 |

## 典型接入

### 依赖链路（源码事实，非可用案例）

- 前提与选择：依赖 `inst.HUD`（客户端 PlayerHud 的 `AddTargetIndicator/RemoveTargetIndicator`，[playerhud.lua:1729](../../../raw/current/scripts/screens/playerhud.lua#L1729)、[1749](../../../raw/current/scripts/screens/playerhud.lua#L1749)）与世界上的 hudindicatablemanager（客户端挂载，[world.lua:560](../../../raw/current/scripts/prefabs/world.lua#L560）；玩家侧 hudindicatable 已挂载并配好追踪函数（([player_common.lua:2587-2588](../../../raw/current/scripts/prefabs/player_common.lua#L2587-L2588)）。
- 生效与撤销：组件自身 `StartUpdatingComponent` 驱动（[ctor](../../../raw/current/scripts/components/playertargetindicator.lua#L27)）。

```text
（无可核验的接入示例：本快照无任何 AddComponent 调用者，不演示挂载。）
```

- 调整与验证：构造内注释掉的 `max_range = TUNING.MAX_INDICATOR_RANGE * 1.5` 与玩家侧追踪函数使用同一常量（[player_common.lua:2306](../../../raw/current/scripts/prefabs/player_common.lua#L2306），可视为配套参数。

## 常见陷阱与限制

- **无原版消费者**：`AddComponent("playertargetindicator")` 与 `components.playertargetindicator` 在本快照 scripts 树零命中，组件从不被挂载；其逻辑更接近 hudindicatorwatcher 的同类实现（[同类](../../../raw/current/scripts/components/hudindicatorwatcher.lua#L29-L40)），复用需求应优先看该组件。
- **OnUpdate 依赖 world 组件存在**：挑选目标的分支整体包在 `if TheWorld.components.hudindicatablemanager then` 内，缺登记表时只清理不新增（[分支](../../../raw/current/scripts/components/playertargetindicator.lua#L72-L85)）。
- **ShouldTrack 视角是自身**：`hudindicatable:ShouldTrack(self.inst)` 传入的是组件宿主（观察者），与玩家侧 hudindicatable 的 viewer 语义一致（[实现](../../../raw/current/scripts/components/playertargetindicator.lua#L45)）。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；覆盖组件全文、hudindicatable/hudindicatablemanager/playerhud 协作点、玩家侧 hudindicatable 挂载；挂载方式与启用条件未知。
- 待核验问题：该组件是否在特定分支／模式（或历史版本）挂载；本快照内无证据。
- 结构检查：本批 validate_page、validate_source_links 及示例 lua 语法探测（见批级记录）。
- 游戏运行：未运行。
