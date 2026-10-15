---
title: "questowner"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "通用接/弃任务宿主组件：以 questing/questcomplete 状态与同步 tag 支撑 BEGIN_QUEST/ABANDON_QUEST 右键动作，任务行为完全由宿主回调决定。"
aliases: ["任务宿主", "接受任务", "放弃任务", "questowner"]
tags: [component, lifecycle]
dst_build: "steam-buildid:26230496"
size_class: simple
sources: ["components/questowner.lua", "prefabs/smallghost.lua", "prefabs/ticoon.lua", "actions.lua", "componentactions.lua", "brains/ticoonbrain.lua", "stategraphs/SGticoon.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# questowner

## 快速判断

- 职责与适用：记录 questing/questcomplete 布尔并同步 `questing`/`questcomplete` tag（components/questowner.lua:1-14、:27-30）；动作层对带组件实体右键按 tag 给 ACTIONS.BEGIN_QUEST 或 ACTIONS.ABANDON_QUEST（componentactions.lua:716-722）。
- 优先使用：宿主挂组件后注册回调；`CanBeginFn`/`CanAbandonFn` 是直接赋值字段（prefabs/smallghost.lua:503、:505），begin/abandon 回调返回 `(bool, message)`。
- 重要副作用：BeginQuest 重置 questcomplete=false（:59）；CompleteQuest 置 questing=false/questcomplete=true 并调 on_complete_quest（:86-92）。
- 修改边界：OnSave/OnLoad 保存两个状态（:94-109）；OnRemoveFromEntity 清 tag（:44-47）。

## 关键接口

| 接口/字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `BeginQuest` | `(doer) -> bool, message` | 调 `on_begin_quest(inst, doer)`，其首返回值存为 questing；questcomplete 先清 false。:57-64 |
| `AbandonQuest` | `(doer) -> bool\|nil, message` | 非 questing 返回 nil；有 on_abandon_quest 以其返回 bool 决定是否置 questing=false，无回调直接成功。:70-84 |
| `CompleteQuest` | `() -> nil` | 宿主自行调用（brains/ticoonbrain.lua:86 到达目标后）；触发 on_complete_quest。:86-92 |
| `CanBeginQuest`/`CanAbandonQuest` | `(doer) -> bool` | 无 CanBeginFn 默认 true（:49-55）；无 CanAbandonFn 默认 true（:66-68）。 |

## 典型接入

### 原版参照：小幽灵与猫猫任务

- 前提与选择：smallghost.lua:502-506 注册 begin/abandon（无 complete）；ticoon.lua:293-298 四回调齐全并由建造者触发 BeginQuest（ticoon.lua:344）；SGticoon.lua:121 按 questcomplete tag 分支状态。
- 生效与撤销：动作实现见 actions.lua:5222-5240——BEGIN_QUEST 先 CanBegin 再 Begin，ABANDON_QUEST 结果取 `(success ~= false)`。

## 常见陷阱与限制

- 回调字段命名不对称：`SetOnBeginQuest` 存 `on_begin_quest`，但 `CanBeginQuest` 读的是直赋字段 `CanBeginFn`（:50-54）——漏赋 CanBeginFn 意味着任何玩家都能开始任务。
- BeginQuest 的 questing 取回调首返回值：回调返回 false/nil 即"未接任务"，message 透传给动作层提示。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，通读组件全部 111 行，核验全部消费点：smallghost.lua:502-506、ticoon.lua:293-298/:344、ticoonbrain.lua:77/:86、SGticoon.lua:121、actions.lua:5222-5240、componentactions.lua:716-722。
- 游戏运行：未运行。
