---
title: "followerherder：切换现有随从字段并触发驱赶回调"
created: 2026-10-26
updated: "2026-10-28"
type: component
summary: "为蜘蛛哨类物品提供使用资格、随从 hostile 字段切换、耐久消耗与使用后回调。"
aliases: [followerherder, 蜘蛛哨, 驱赶随从, 随从 hostile 字段]
tags: [component, callback, item, actions, server-client]
dst_build: "steam-buildid:26230496"
sources: [components/followerherder.lua, prefabs/spider_whistle.lua, actions.lua, componentactions.lua]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# followerherder：切换现有随从字段并触发驱赶回调

## 快速判断

- 职责与适用：挂在驱赶物品上，保存物品自己的 `hostile` 开关；使用时给 `leader.followers` 表键所指的每个实体写普通字段 `hostile`，不是 `components.follower.hostile`。组件不建立或解除跟随关系。[followerherder.lua:1-4](../../../raw/current/scripts/components/followerherder.lua#L1-L4)、[followerherder.lua:29-42](../../../raw/current/scripts/components/followerherder.lua#L29-L42)
- 优先使用：参照蜘蛛哨的“资格检查 → 执行 → 额外召集效果”链路；额外效果用 `SetOnHerdFn` 配置，随从登记仍由 [[reference/components/leader]]／[[reference/components/follower]] 管理。[actions.lua:5650-5658](../../../raw/current/scripts/actions.lua#L5650-L5658)、[spider_whistle.lua:24-47](../../../raw/current/scripts/prefabs/spider_whistle.lua#L24-L47)
- 重要副作用：每次 `Herd` 先翻转开关、遍历全部普通随从，再消耗可选 `finiteuses`，最后调用回调；没有蜘蛛标签过滤。[followerherder.lua:29-42](../../../raw/current/scripts/components/followerherder.lua#L29-L42)
- 修改边界：原版在物品服务端分支添加组件。这里只保证写入字段，不保证任意随从会改变 AI、战斗目标或阵营；具体行为须由消费者支持。[spider_whistle.lua:70-82](../../../raw/current/scripts/prefabs/spider_whistle.lua#L70-L82)、[followerherder.lua:29-33](../../../raw/current/scripts/components/followerherder.lua#L29-L33)

## 关键接口

| 接口／字段 | 参数、返回／默认值 | 条件、作用与证据 |
|---|---|---|
| `SetCanHerdFn(fn)`、`CanHerd(leader)` | 回调 `fn(物品inst, 使用者)`，返回可用值和原因 | 未设置回调则返回 `true`；回调首返回值为假时透传 `false, reason`。组件不自行检查使用者是否有 `leader`。[followerherder.lua:6-8](../../../raw/current/scripts/components/followerherder.lua#L6-L8)、[followerherder.lua:18-27](../../../raw/current/scripts/components/followerherder.lua#L18-L27) |
| `SetOnHerdFn(fn)` | 回调 `fn(物品inst, 使用者)` | 内部保存为 `onherfn`；在耐久消耗之后执行，返回值不参与结果判定。[followerherder.lua:10-12](../../../raw/current/scripts/components/followerherder.lua#L10-L12)、[followerherder.lua:35-41](../../../raw/current/scripts/components/followerherder.lua#L35-L41) |
| `SetUseAmount(use_amount)` | 耐久消耗量 | 仅赋值；有 `finiteuses` 时调用 `Use(use_amount or 1)`，没有该组件则跳过消耗。[followerherder.lua:14-16](../../../raw/current/scripts/components/followerherder.lua#L14-L16)、[followerherder.lua:35-37](../../../raw/current/scripts/components/followerherder.lua#L35-L37) |
| `Herd(leader)`、`hostile` | 无显式返回；开关默认 `true` | 第一次调用翻为 `false`；直接访问 `leader.components.leader.followers`，写全部表键实体，随后消耗耐久和回调。[followerherder.lua:1-4](../../../raw/current/scripts/components/followerherder.lua#L1-L4)、[followerherder.lua:29-42](../../../raw/current/scripts/components/followerherder.lua#L29-L42) |

## 典型接入

### 蜘蛛哨：动作检查与额外召集分工

`spider_whistle` 的工厂在服务端配置 `followerherder` 的两类回调和 `TUNING.SPIDER_WHISTLE_USE_AMOUNT`；同时添加 `finiteuses`，其耗尽回调移除物品。注册名为 `spider_whistle`。[spider_whistle.lua:49-92](../../../raw/current/scripts/prefabs/spider_whistle.lua#L49-L92)

库存动作收集器只对带 `spiderwhisperer` 标签的使用者提供 `HERD_FOLLOWERS`；执行动作时先 `CanHerd(act.doer)`，失败返回原因，成功才 `Herd(act.doer)`。蜘蛛哨自己的 `CanHerd` 也检查该标签，否则返回 `WEBBERONLY`。这些资格约束来自动作与宿主回调，不是组件默认行为。[componentactions.lua:2570](../../../raw/current/scripts/componentactions.lua#L2570)、[componentactions.lua:2724-2728](../../../raw/current/scripts/componentactions.lua#L2724-L2728)、[actions.lua:5650-5658](../../../raw/current/scripts/actions.lua#L5650-L5658)、[spider_whistle.lua:12-18](../../../raw/current/scripts/prefabs/spider_whistle.lua#L12-L18)

宿主 `OnHerd` 在使用者附近按 `TUNING.SPIDER_WHISTLE_RANGE` 查找蜘蛛巢／茧，符合有巢内子体且提供 `SummonChildren` 的条件才调用召出；再查附近蜘蛛，排除蛛后、尸体及 `NOCLICK`，对睡着者唤醒并添加 `spider_summoned_buff`。最后给普通随从集合中带 `spider` 标签者添加 `spider_whistle_buff`。组件自身的字段切换没有这个标签与距离过滤。[spider_whistle.lua:20-47](../../../raw/current/scripts/prefabs/spider_whistle.lua#L20-L47)

## 常见陷阱与限制

- **直接调用绕过资格检查**：`Herd` 不调用 `CanHerd`，也不判空 `leader` 组件。自定义入口须自行检查资格及所需随从集合；动作入口已提供前一层检查。[followerherder.lua:18-42](../../../raw/current/scripts/components/followerherder.lua#L18-L42)、[actions.lua:5650-5658](../../../raw/current/scripts/actions.lua#L5650-L5658)
- **开关属于物品**：它按物品上次值取反并覆盖当前集合，不根据每个随从原值切换；新加入的随从也不会自动继承该值。不要把它理解为已同步的玩家统一命令状态。[followerherder.lua:1-4](../../../raw/current/scripts/components/followerherder.lua#L1-L4)、[followerherder.lua:29-33](../../../raw/current/scripts/components/followerherder.lua#L29-L33)
- **存读及撤销需要外部设计**：组件没有 `OnSave`／`OnLoad`、移除清理或恢复随从旧字段的逻辑；重建时开关回到 `true`，移除物品不会由本组件回滚先前写入。`SetOnHerdFn` 是单槽赋值，覆盖会替换原蜘蛛哨效果。[followerherder.lua:1-44](../../../raw/current/scripts/components/followerherder.lua#L1-L44)

## 验证记录

- 源码审阅：2026-10-26，Steam buildid 26148036；覆盖组件全部实现、蜘蛛哨完整工厂与回调、库存动作收集及动作执行顺序。简单档范围为单物品使用链，无独立成员生命周期。
- 游戏运行：未运行；未核验任意生物对 `hostile` 字段的 AI 消费、两种 buff 内部效果、蜘蛛巢 `SummonChildren` 内部及耗尽时序表现。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 通过；无新增 Lua 示例，未运行语法探针。

相关：[[reference/components/leader]]、[[reference/components/follower]]、[[reference/components/finiteuses]]。
