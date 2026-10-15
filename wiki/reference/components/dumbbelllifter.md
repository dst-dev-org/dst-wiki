---
title: "dumbbelllifter"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "沃尔夫冈专属的哑铃举起状态组件：登记正在用的哑铃、驱动 mightydumbbell 训练并用 tag 供动画与动作判断。"
aliases: ["哑铃举起", "健身组件", "dumbbelllifter"]
tags: [component, actions]
dst_build: "steam-buildid:26230496"
size_class: simple
sources: ["components/dumbbelllifter.lua", "actions.lua", "stategraphs/SGwilson.lua", "prefabs/wolfgang.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# dumbbelllifter

## 快速判断

- 职责与适用：角色侧举哑铃的会话组件：持有 `self.dumbbell` 引用，起／停时调用哑铃的 `mightydumbbell:StartWorkout`／`StopWorkout`，并维护 `liftingdumbbell` tag。原版只有沃尔夫冈携带（与 mightiness 同批添加，[wolfgang.lua#L668](../../../raw/current/scripts/prefabs/wolfgang.lua#L668)）。
- 优先使用：入口是 `ACTIONS.LIFT_DUMBBELL`（动作层调 `CanLift`＋`StartLifting`，[actions.lua#L5703-L5719](../../../raw/current/scripts/actions.lua#L5703-L5719)）；SG 在训练循环里用 `Lift()` 逐次执行训练。
- 重要副作用：`StartLifting`／`StopLifting` 分别加／删 `liftingdumbbell` tag（[L20](../../../raw/current/scripts/components/dumbbelllifter.lua#L20)、[L29](../../../raw/current/scripts/components/dumbbelllifter.lua#L29)）；`Lift()` 失败会把 `self.dumbbell` 置 nil（哑铃失效自清理，[L32-L41](../../../raw/current/scripts/components/dumbbelllifter.lua#L32-L41)）。
- 修改边界：不存档（无 OnSave／OnLoad）；直接索引 `dumbbell.components.mightydumbbell`，宿主必须是带该组件的哑铃；只有沃尔夫冈有组件，其他角色动作层直接失败。

## 关键接口

| 接口／字段 | 已核验类型 | 条件、作用与证据 |
|---|---|---|
| `CanLift(dumbbell)` | 恒 true | 基础实现无门槛，留给子类／宿主覆盖；动作层在此之前还有 itemmimic 检查。[L5-L7](../../../raw/current/scripts/components/dumbbelllifter.lua#L5-L7)、[动作层](../../../raw/current/scripts/actions.lua#L5703-L5719) |
| `IsLiftingAny()`／`IsLifting(d)` | boolean | 判断 `self.dumbbell`；`LIFT_DUMBBELL` 动作 / SG 用它防重入（[SGwilson.lua#L1662](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1662)）。[L9-L15](../../../raw/current/scripts/components/dumbbelllifter.lua#L9-L15) |
| `StartLifting(dumbbell)` | 无返回 | 记录哑铃、`mightydumbbell:StartWorkout(self.inst)`、加 tag。[L17-L21](../../../raw/current/scripts/components/dumbbelllifter.lua#L17-L21) |
| `StopLifting()` | 无返回 | 有哑铃时 `StopWorkout` 并清引用；无论有无都删 tag。[L23-L30](../../../raw/current/scripts/components/dumbbelllifter.lua#L23-L30) |
| `Lift()` | boolean 或 nil | 哑铃有效时 `mightydumbbell:DoWorkout(self.inst)` 决定 true／false（false 时清引用）；无哑铃时隐式返回 nil。[L32-L41](../../../raw/current/scripts/components/dumbbelllifter.lua#L32-L41) |

## 典型接入

### 原版训练链

- 举起：LIFT_DUMBBELL 动作（mimic 检查→CanLift→StartLifting）后 SG 进入训练状态（[SGwilson.lua#L2823](../../../raw/current/scripts/stategraphs/SGwilson.lua#L2823)）；训练循环节点里 `Lift()` 成功且 mightiness 未满就继续（[SGwilson.lua#L8284-L8286](../../../raw/current/scripts/stategraphs/SGwilson.lua#L8284-L8286)）。
- 中断：SG 在受击、换装、死亡等分支调 `StopLifting()`（[SGwilson.lua#L2213](../../../raw/current/scripts/stategraphs/SGwilson.lua#L2213)、[L8177](../../../raw/current/scripts/stategraphs/SGwilson.lua#L8177)）；训练结束读取 `self.dumbbell` 决定后续（[SGwilson.lua#L8152](../../../raw/current/scripts/stategraphs/SGwilson.lua#L8152)）。

## 常见陷阱与限制

- `Lift()` 的失败语义是三态：true（练了）、false（哑铃说练不了，引用已清）、nil（没有哑铃）。消费方一律按真值判断；自己扩展时不要把 nil 当 false 处理后还继续用 `self.dumbbell`。
- `StopLifting` 无条件删 tag：在非举起状态调用也安全，但会把别人依赖的 `liftingdumbbell` tag 顺手删掉；重复起停按"先 Stop 再 Start"写。
- 组件不同步：`self.dumbbell` 与训练进度都在服务端；客户端 SG 只跟动画，不要在客户端读组件字段。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。完整阅读 components/dumbbelllifter.lua（43 行），核对沃尔夫冈添加点、LIFT_DUMBBELL 动作与 SGwilson 训练／中断分支。未覆盖 mightydumbbell 组件的训练数值。
- 游戏运行：未运行。
- 结构检查：validate_page.py 通过（0 issue；53 行、4911 字节，simple 档内）；validate_source_links.py --strict 通过（17 处源码引用，0 error／0 warning）；本页无完整 Lua 代码块。
