---
title: "walkingplank"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "船踏板控制端：维护登板玩家占用，用 immediate 事件驱动伸缩/登板/弃船状态图，与玩家侧 walkingplankuser 配对。"
aliases: ["船踏板", "walkingplank", "登船踏板"]
tags: [component]
dst_build: "steam-buildid:26230496"
sources: ["components/walkingplank.lua", "prefabs/walkingplank.lua", "actions.lua", "componentactions.lua", "components/walkingplankuser.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# walkingplank

## 快速判断

- 职责与适用：挂在踏板实体（walkingplank、walkingplank_grass、walkingplank_yotd、walkingplank_ancient）上；记住当前登板玩家 `doer`，伸缩/登板/弃船都以 `PushEventImmediate` 同步推事件给状态图执行（[组件全文](../../../raw/current/scripts/components/walkingplank.lua#L1-L50)、[宿主](../../../raw/current/scripts/prefabs/walkingplank.lua#L41-L56)）。
- 优先使用：玩家与踏板的配对靠玩家侧 [[reference/components/walkingplankuser]]——MountPlank 成功后调 `SetCurrentPlank`，AbandonShip 调 `Dismount`（[MountPlank](../../../raw/current/scripts/components/walkingplank.lua#L21-L32)、[AbandonShip](../../../raw/current/scripts/components/walkingplank.lua#L39-L48)）。
- 重要副作用：MountPlank 直接 `Physics:Teleport` 把玩家传到踏板位置（[传送](../../../raw/current/scripts/components/walkingplank.lua#L27)）；已占用时拒绝第二次登板（返回 false）。
- 修改边界：动作候选按 `interactable`、`plank_extended`、`on_walkable_plank` 标签生成（[候选](../../../raw/current/scripts/componentactions.lua#L937-L957)）；组件只推事件，伸缩动画与检测都在 SGwalkingplank 内。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `Extend()` / `Retract()` | `-> 无` | 分别推 `start_extending`/`start_retracting`（immediate）。[实现](../../../raw/current/scripts/components/walkingplank.lua#L13-L19) |
| `MountPlank(doer)` | `doer: entity -> bool` | `doer` 已被占用返回 false；否则记录 doer、传送玩家、推 `start_mounting`、调 `walkingplankuser:SetCurrentPlank`。[实现](../../../raw/current/scripts/components/walkingplank.lua#L21-L32) |
| `StopMounting()` | `-> 无` | 清 `doer` 并推 `stop_mounting`；玩家侧 Dismount 会调用。[实现](../../../raw/current/scripts/components/walkingplank.lua#L34-L37) |
| `AbandonShip(doer)` | `doer: entity -> bool` | 仅当 `doer == self.doer`；调 `walkingplankuser:Dismount()` 后推 `start_abandoning`。[实现](../../../raw/current/scripts/components/walkingplank.lua#L39-L48) |
| `doer` | `entity\|nil` | 当前登板玩家；构造中仅注释声明（[字段](../../../raw/current/scripts/components/walkingplank.lua#L4)），不参与存档。 |

## 典型接入

### 玩家登板链路（原版参照）

- 前提与选择：MOUNT_PLANK 要求踏板 `interactable` 且 `plank_extended`（[候选](../../../raw/current/scripts/componentactions.lua#L951-L956)）；ABANDON_SHIP 仅在玩家带 `on_walkable_plank` 标签时出现（[候选](../../../raw/current/scripts/componentactions.lua#L938-L941)）。
- 生效与撤销：动作执行集中在 ([actions.lua:4893-4912](../../../raw/current/scripts/actions.lua#L4893-L4912)（MOUNT_PLANK/ABANDON_SHIP/EXTEND_PLANK/RETRACT_PLANK）；登板后玩家退出踏板状态时 SG 侧兜底强制 Dismount（见 walkingplankuser 页）。

```lua
-- 服务端：动作链路等价调用（一般勿手动调用，会绕过占用检查外的 SG 状态）
plank.components.walkingplank:Extend()
plank.components.walkingplank:MountPlank(doer)
```

- 调整与验证：`PushEventImmediate` 为同步派发，事件处理立即执行完才返回；自定义踏板需同时满足标签（pristine 内加 `walkingplank`，[宿主](../../../raw/current/scripts/prefabs/walkingplank.lua#L20)）与 SGwalkingplank 状态图。

## 常见陷阱与限制

- **DISMOUNT_PLANK 是死路径**：动作函数调用 `walkingplank:DismountPlank(act.doer)`（[调用](../../../raw/current/scripts/actions.lua#L4897-L4900)），但全库无 `DismountPlank` 定义，也没有候选生成器插入 DISMOUNT_PLANK（仅状态图 ActionHandler 引用）；复用该动作会直接报错，离板走 AbandonShip 或 SG 兜底。
- **占用不做归属校验（登板）**：MountPlank 只查 `self.doer ~= nil`，不校验旧 doer 是否还活着；旧 doer 异常残留时会一直拒绝登板，需依赖 SG/宿主清理。
- **组件不管水上位置**：踏板与船体的装配、可站区域由 walkableplatform/船体侧处理，本组件只负责踏板自身的占用与事件。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；覆盖组件全文、四个踏板宿主、动作候选与执行、walkingplankuser 配对点、DISMOUNT_PLANK 全库检索。
- 结构检查：本批 validate_page、validate_source_links 通过（见批级记录）。
- 游戏运行：未运行。

相关：[[reference/components/walkingplankuser]]、[[reference/components/walkableplatform]]
