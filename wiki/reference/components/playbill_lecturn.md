---
title: "playbill_lecturn"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "查理戏台剧本架组件：持有当前 playbill 物品，接受 GIVE 换本（旧本弹出、新本上座），驱动 writeable 剧本文本与戏台 AddPlay。"
aliases: ["剧本架", "剧本讲台", "playbill_lecturn"]
tags: [component, building]
dst_build: "steam-buildid:26230496"
size_class: simple
sources: ["components/playbill_lecturn.lua", "prefabs/charlie_lecturn.lua", "prefabs/charlie_stage.lua", "components/stageactingprop.lua", "actions.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# playbill_lecturn

## 快速判断

- 职责与适用：挂 charlie_lecturn（prefabs/charlie_lecturn.lua:90-91 附 onstageset 回调）；构造即 AddTag `playbill_lecturn`（components/playbill_lecturn.lua:8），动作层以该 tag 允许把 playbill 物品 GIVE 给讲台（actions.lua:2296-2297，位于 ACTIONS.GIVE.fn，:2283）。
- 优先使用：换本核心 `SwapPlayBill(playbill, doer)`；`SetStage(stage)` 由戏台绑定（prefabs/charlie_stage.lua:283）或存档 postpass（charlie_lecturn.lua:43-48）。
- 重要副作用：UpdateText 拼当前幕剧本+演员表写入 `writeable:SetText` 并推 `"text_changed"`（:30-44）；换本时旧本 `FlingItem` 弹出、新本 book_build 覆盖 AnimState（:56-69）。
- 修改边界：OnSave 记 playbill GUID，LoadPostPass 延迟一帧换本（注释要求 stage 已先载入，:97-110）；OnRemoveEntity 与 OnRemoveFromEntity 都只移除 tag（:11-14）。

## 关键接口

| 接口/字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SwapPlayBill` | `(playbill, doer\|nil) -> nil` | doer 在场时重置剧本起始幕并从其背包移除；有 stage 时向 `stageactingprop:AddPlay` 递交 costumes/scripts/act 数据并播 switch 动画。:46-85 |
| `ChangeAct` | `(next_act) -> nil` | 戏台演完切幕后同步（components/stageactingprop.lua:398 FinishAct 调用）并刷新文本。:23-28 |
| `SetStage` | `(stage) -> nil` | 绑定戏台实体，触发 onstageset 回调。:16-21 |
| `UpdateText` | `() -> nil` | 读 `playbill` 组件的 scripts/costumes 数据结构拼文本，依赖宿主 writeable。:30-44 |

## 典型接入

### 原版参照：换本与演出协作

- 前提与选择：宿主需带 writeable/lootdropper/AnimState（charlie_lecturn.lua:89-91 同步挂载）；物品侧需 playbill 组件提供 scripts、costumes、starting_act、book_build。
- 生效与撤销：载入链 LoadPostPass→延迟一帧 SwapPlayBill（无 doer，不弹旧本）保证 stage 先恢复（:102-110）。
- 无 mod 接入案例，不提供代码。

## 常见陷阱与限制

- UpdateText 直接解引用 `script.playbill`/`script.cast`/`pb.costumes[cast_member].name`（:34-39），剧本数据缺字段会报错。
- 换本把旧 playbill 弹在地上而非入包，`doer` 为 nil 时新本直接落位（:46-55）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，通读组件全部 112 行，核验挂载（charlie_lecturn.lua:72/90-91）、GIVE 动作分支（actions.lua:2283/2296-2297）、戏台协作（charlie_stage.lua:283/287、stageactingprop.lua:398）。
- 游戏运行：未运行。

## 相关链接

- 数据来源物品组件：[[reference/components/playbill]]；文本写入：[[reference/components/writeable]]。
