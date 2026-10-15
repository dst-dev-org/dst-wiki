---
title: "souleater"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "食魂者能力：EAT 灵魂物品时取出单件、推 oneatsoul 事件并回调宿主效果，随后删除灵魂；原版唯一宿主是沃拓克斯。"
aliases: ["souleater", "食魂者", "食魂组件"]
tags: [component, actions, callback]
size_class: simple
dst_build: "steam-buildid:26230496"
sources: ["components/souleater.lua", "prefabs/wortox.lua", "actions.lua", "componentactions.lua", "stategraphs/SGwilson.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# souleater

## 快速判断

- 职责与适用：挂在会吃灵魂的角色／生物上；构造即加 `souleater` 标签（V2C 建议在 pristine 前加），供双端动作候选判定。[构造](../../../raw/current/scripts/components/souleater.lua#L6-L7)
- 优先使用：灵魂物品的进食路由已在原版完成——EAT 动作发现目标是 soul 物品且进食者有本组件时调 `EatSoul(obj)`；自定义宿主只需挂组件，不必改动作。[EAT 分支](../../../raw/current/scripts/actions.lua#L804-L805)
- 重要副作用：`EatSoul` 会把成组灵魂拆成单件（stackable:Get()）、推 `oneatsoul` 事件、调 onusefn 同类的 oneatsoulfn，最后删除灵魂实体；事件与回调里删除灵魂是预期行为。[实现](../../../raw/current/scripts/components/souleater.lua#L14-L29)
- 修改边界：组件不提供灵魂数量、回饱食度等效果——全部在宿主回调；灵魂物品侧要求见 [soul](../../../raw/current/scripts/components/soul.lua)（已有 Wiki 页 [[reference/components/soul]]）。[宿主回调示例](../../../raw/current/scripts/prefabs/wortox.lua#L545-L558)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SetOnEatSoulFn(fn)` | `fn(inst, soul)` | 宿主效果回调；soul 已拆为单件实体。wortox 用它回饱食度并按 inclination 扣理智。[定义](../../../raw/current/scripts/components/souleater.lua#L10-L12) [原版回调](../../../raw/current/scripts/prefabs/wortox.lua#L545-L558) |
| `EatSoul(soul)` | `-> boolean` | 无 `soul` 组件返回 false；否则发 `oneatsoul {soul}`、调回调、`soul:IsValid()` 才 Remove（回调可能已删），返回 true。[实现](../../../raw/current/scripts/components/souleater.lua#L14-L29) |
| `souleater` 标签 | 构造时添加 | 动作候选（对自身使用／背包内食用）与 SG 进食校验都按标签或组件判定；prefab 应在 pristine 前显式加。[标签](../../../raw/current/scripts/components/souleater.lua#L6-L7) [wortox pristine](../../../raw/current/scripts/prefabs/wortox.lua#L698-L699) |

## 典型接入

### 让自定义角色吃灵魂（原版装配参照）

- 前提与选择：标签在 pristine 前加、组件在 mastersim 分支挂；进食者无需其他组件。[装配](../../../raw/current/scripts/prefabs/wortox.lua#L777-L778)
- 生效与撤销：进食链由 EAT 动作与 SGwilson 食用状态驱动——SG 校验发现 soul 物品而无 souleater 时推 `wonteatfood` 并中止；有组件则在动画帧里调 `EatSoul`。[SG 校验](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1229-L1232) [执行](../../../raw/current/scripts/stategraphs/SGwilson.lua#L6462-L6463)

```lua
-- prefab 内：pristine 前加标签
inst:AddTag("souleater")
-- mastersim 分支：
inst:AddComponent("souleater")
inst.components.souleater:SetOnEatSoulFn(function(inst, soul)
    -- soul 为单件灵魂实体；组件随后负责删除
end)
```

- 调整与验证：非玩家生物也能吃灵魂——hermitcrab／WX-78 附身体的状态图在动画帧按组件存在与否分支调用 `EatSoul`，均为判空守卫后调用。[hermitcrab](../../../raw/current/scripts/stategraphs/SGhermitcrab.lua#L1847-L1848) [possessedbody](../../../raw/current/scripts/stategraphs/SGwx78_possessedbody.lua#L2177-L2178)

## 常见陷阱与限制

- **只加组件不加标签：** 服务端 EatSoul 能跑，但动作候选按 `souleater` 标签过滤（componentactions 两处），客户端看不到"吃"选项；标签必须在 pristine 前加。[候选](../../../raw/current/scripts/componentactions.lua#L2936)
- **在 onusefn 式回调里假设 soul 仍有效：** 组件在回调后才删灵魂，但回调本身允许删除；`EatSoul` 用 `soul:IsValid()` 兜底，回调里不要二次 PushEvent oneatsoul 或重复计数。[兜底](../../../raw/current/scripts/components/souleater.lua#L26-L28)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（32 行），核对 wortox 装配与回调、EAT 动作分支、两处 componentactions 候选、SGwilson 校验与执行、SGhermitcrab／SGwx78_possessedbody 判空调用。未审阅灵魂掉落与 souljar 收集链路（见 soul 页）。
- 结构检查：2026-10-28 validate_page.py 通过；validate_source_links.py --strict 锚点通过（见批级记录）。
- 游戏运行：未运行。
