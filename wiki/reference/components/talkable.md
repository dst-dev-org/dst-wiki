---
title: "talkable"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "可对话目标数据组件：TALKTO 动作成功的前提；候选生成却由 maxwellnottalking 标签驱动，原版脚本无 AddComponent 消费者。"
aliases: ["可对话组件", "TALKTO 对话目标", "talkable"]
tags: [component, actions]
dst_build: "steam-buildid:26230496"
sources: ["components/talkable.lua", "actions.lua", "componentactions.lua", "components/maxwelltalker.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# talkable

## 快速判断

- 职责与适用：保存对话状态的两个字段（`conversation`、`conv_index`），无方法、无更新逻辑。[组件](../../../raw/current/scripts/components/talkable.lua#L1-L4)
- 优先使用：无——全脚本字面量检索不存在 `AddComponent("talkable")`；TALKTO 动作执行以 `targ.components.talkable` 存在为成功前提（[动作](../../../raw/current/scripts/actions.lua#L1532-L1543)）。
- 重要副作用：组件本体无副作用；TALKTO 候选生成看 `maxwellnottalking` 标签，由 maxwelltalker 组件维护（说话时移除、说完加回），与 talkable 无关（[SCENE 候选](../../../raw/current/scripts/componentactions.lua#L872-L874)、[INVENTORY 候选](../../../raw/current/scripts/componentactions.lua#L2996-L2998)、[标签来源](../../../raw/current/scripts/components/maxwelltalker.lua#L1-L12)）。
- 修改边界：无存读钩子、无清理钩子；字段无任何读写者。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `conversation` | 默认 nil | 全脚本无读写者（字面量检索）。[构造](../../../raw/current/scripts/components/talkable.lua#L3) |
| `conv_index` | 默认 1（数字） | 全脚本无读写者（字面量检索）。[构造](../../../raw/current/scripts/components/talkable.lua#L4) |

## 常见陷阱与限制

- **原版无消费者**：按本快照检索，talkable 与 maxwelltalker 两个组件都没有原版 AddComponent 宿主；TALKTO 候选靠 maxwelltalker 的标签，但动作执行要求目标带 talkable 才返回 true——对话闭环在脚本层不成立，组件疑似为未接线的遗留接口。
- **TALKTO.fn 的空引用路径**：动作目标可回退到 `act.invobject`（[第 1533 行](../../../raw/current/scripts/actions.lua#L1533)），但后续对话分支读的是 `act.target.components.maxwelltalker`（[第 1537 行](../../../raw/current/scripts/actions.lua#L1537)）；若经 invobject 发起且 `act.target` 为 nil，该行会直接索引空值。引用此动作时按 target 路径使用。
- **动作成功不代表有话说**：目标有 talkable 但无 maxwelltalker 时，fn 跳过对话分支仍返回 true（[动作](../../../raw/current/scripts/actions.lua#L1537-L1543)）——"可对话"与"会说话"由两个组件分别承担。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；覆盖组件全文、TALKTO 定义与执行、SCENE／INVENTORY 两处候选、maxwelltalker 标签维护，并以字面量检索确认 `talkable`／`maxwellnottalking`／`maxwelltalker` 无其他引用。未覆盖：引擎侧行为（如有）、运行验证。
- 结构检查：本批 validate_page、validate_source_links。
- 游戏运行：未运行。
