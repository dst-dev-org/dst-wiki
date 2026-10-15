---
title: "anchor"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "船锚结构状态机：多人合力起锚／自动落锚的进度推进，联动 boatdrag 与船物理启停，供 RAISE/LOWER_ANCHOR 动作驱动。"
aliases: ["船锚", "起锚", "anchor组件"]
tags: [component, state-machine]
dst_build: "steam-buildid:26230496"
sources: ["components/anchor.lua", "prefabs/anchor.lua", "actions.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# anchor

## 快速判断

- 职责与适用：挂在船锚建筑 `anchor`（[装配](../../../raw/current/scripts/prefabs/anchor.lua#L110)），管理"起锚进行中 / 已落下 / 已升起"三态与 raiseunits 进度；对象是船的移动限制与拖曳阻力。
- 优先使用：玩家动作入口只有两个——RAISE_ANCHOR 调 `AddAnchorRaiser(doer)`，LOWER_ANCHOR 调 `StartLoweringAnchor()`（[动作](../../../raw/current/scripts/actions.lua#L4885-L4891)）；程序查状态用 `GetBoat()`／标签 `anchor_raised`／`anchor_lowered`。
- 重要副作用：`is_anchor_lowered` 变化时在船上增删 [[reference/components/boatdrag]]（[L145-L157](../../../raw/current/scripts/components/anchor.lua#L145-L157)）；锚定状态下船不动，起锚完成推 `anchor_raised` 后船可开。
- 修改边界：构造后 1 帧自动绑当前平台为船（[L71-L73](../../../raw/current/scripts/components/anchor.lua#L71-L73)）；读档在 `LoadPostPass` 恢复并直达对应状态（[L117-L138](../../../raw/current/scripts/components/anchor.lua#L117-L138)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `AddAnchorRaiser(doer)` / `RemoveAnchorRaiser(doer)` | → `bool` | 多人合力：每人速度＝`expertsailor:GetAnchorRaisingSpeed()` × 船员头衔倍率，累进 `currentraiseunits`；进过渡态加 `anchor_transitioning` 标签（[L181-L215](../../../raw/current/scripts/components/anchor.lua#L181-L215)）。 |
| `StartLoweringAnchor()` / `StartRaisingAnchor()` | → `bool` | 已在位／已烧焦返回 false；否则进过渡并推 `lowering_anchor`／`raising_anchor` 实体事件（[L159-L179](../../../raw/current/scripts/components/anchor.lua#L159-L179)）。 |
| `SetIsAnchorLowered(is_lowered)` | bool | 状态机落位由 SGanchor 调用；同时管理船上 boatdrag（[L145-L157](../../../raw/current/scripts/components/anchor.lua#L145-L157)、[SG 调用](../../../raw/current/scripts/stategraphs/SGanchor.lua#L3-L11)）。 |
| `GetCurrentDepth()` | → 数字 | 锚链"深度"：默认 0.1，海图瓦片有 `ocean_depth` 时查 `TUNING.ANCHOR_DEPTH_TIMES`（[L95-L107](../../../raw/current/scripts/components/anchor.lua#L95-L107)；[表](../../../raw/current/scripts/tuning.lua#L4599)）。 |
| `OnUpdate(dt)` | 过渡态更新 | 有人拉：`raiseunits -= dt×currentraiseunits`；无人：`+dt×3` 自动落；动画倍速 `0.2+0.3×人数`；到 0 起锚完成、到深度落锚完成（[L234-L266](../../../raw/current/scripts/components/anchor.lua#L234-L266)）。 |
| `OnSave` / `OnLoad` | 仅 `raiseunits` | 其余状态由 LoadPostPass 按深度重推（[L86-L115](../../../raw/current/scripts/components/anchor.lua#L86-L115)）。 |

## 状态与事件（关键流程）

1. 字段监听：三个字段带赋值监听——`is_anchor_lowered` 同步 `anchor_raised`／`anchor_lowered` 标签；另两者按"船是否在动"互斥启停组件更新（[L1-L29](../../../raw/current/scripts/components/anchor.lua#L1-L29)、[监听表](../../../raw/current/scripts/components/anchor.lua#L77-L79)）。
2. 船绑定：监听 `onremove`／`boat_stop_moving`／`boat_start_moving` 维护 `is_boat_moving`；船动时过渡态不启停更新（[L32-L45](../../../raw/current/scripts/components/anchor.lua#L32-L45)）。
3. 完成回调：`AnchorRaised`／`AnchorLowered` 清过渡态与标签、复位动画倍速、逐个移除拉锚者并推 `anchor_raised`／`anchor_lowered` 实体事件（[L217-L232](../../../raw/current/scripts/components/anchor.lua#L217-L232)）。
4. 外部消费：玩家起锚状态在离开时调 `RemoveAnchorRaiser`（[SGwilson](../../../raw/current/scripts/stategraphs/SGwilson.lua#L12989)）；海浪之手等只对 `anchor_lowered` 且非过渡的锚出手（[例](../../../raw/current/scripts/brains/waveyjoneshandbrain.lua#L23)）。

## 常见陷阱与限制

- **动作只加人，进度在 OnUpdate**：`AddAnchorRaiser` 只登记拉锚者与速度，锚是否动完全由过渡态 OnUpdate 推进；直接改 `raiseunits` 而不进过渡态不会触发完成回调（[L238-L259](../../../raw/current/scripts/components/anchor.lua#L238-L259)）。
- **陆地与船上行为分叉**：无船（`boat == nil`）且深度达标时落锚直达 `lowered_land` 状态；深度的 0.1 兜底意味着非海洋瓦片几乎立即落定（[L126-L135](../../../raw/current/scripts/components/anchor.lua#L126-L135)）。
- **源码拼写遗留**：内部字段 `self.rasing`（应为 raising）拼写错误但无消费者，不影响行为（[L191](../../../raw/current/scripts/components/anchor.lua#L191)）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。覆盖组件全文、anchor prefab 装配与锤毁段、RAISE/LOWER_ANCHOR 动作与候选行、SGanchor／SGwilson 消费行与 ANCHOR_DEPTH_TIMES 表头；未覆盖 boatphysics 的 AddBoatDrag 内部与 client 预测表现。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、Lua 语法探测通过（见批级记录）。

相关：[[reference/components/boatdrag]]（锚定阻力）、[[reference/components/boatphysics]]（船体物理）。
