---
title: "halloweenpotionmoon"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "万圣节月之药水的使用执行：对带 halloweenmoonmutable 的目标触发变异，回调宿主效果并无条件消耗药水。"
aliases: ["halloweenpotionmoon", "月之药水", "万圣节变形药水"]
tags: [component, item, actions]
size_class: simple
dst_build: "steam-buildid:26230496"
sources: ["components/halloweenpotionmoon.lua", "prefabs/halloweenpotion_moon.lua", "actions.lua", "componentactions.lua", "components/halloweenmoonmutable.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# halloweenpotionmoon

## 快速判断

- 职责与适用：挂在月之药水物品上；HALLOWEENMOONMUTATE 动作执行时调 `Use(doer, target)`，对带 `halloweenmoonmutable` 组件的目标做变异，把结果交给 onusefn 回调。[Use](../../../raw/current/scripts/components/halloweenpotionmoon.lua#L11-L30)
- 优先使用：想让物品"对目标施放变异"就走现成动作路由——componentactions 的 point 候选只要求目标没有 `DECOR` 标签，动作层再排除燃烧／冻结目标。[候选](../../../raw/current/scripts/componentactions.lua#L1340-L1343)
- 重要副作用：`Use` 结束时无条件消耗药水（stackable 取一件删除，否则整瓶 Remove），与目标是否真的发生变异无关。[消耗](../../../raw/current/scripts/components/halloweenpotionmoon.lua#L25-L29)
- 修改边界：组件不做变异本身（在 halloweenmoonmutable），也不判断目标合法性（动作层已做）；变异结果与失败提示都在 onusefn。[变异侧](../../../raw/current/scripts/components/halloweenmoonmutable.lua#L28-L35)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SetOnUseFn(fn)` | `fn(inst, doer, target, success, transformed_inst, container)` | 宿主效果回调；`transformed_inst` 为变异新实体（转换覆写或无效果时为 nil），`container` 为目标所在容器（可能为 nil）。[定义](../../../raw/current/scripts/components/halloweenpotionmoon.lua#L7-L9) [回调实参](../../../raw/current/scripts/components/halloweenpotionmoon.lua#L21-L23) |
| `Use(doer, target)` | `-> 无` | 目标有 halloweenmoonmutable 才调 `Mutate()` 并置 success；随后调 onusefn，最后无条件消耗自身。[实现](../../../raw/current/scripts/components/halloweenpotionmoon.lua#L11-L30) |

## 典型接入

### 月之药水的原版效果装配

- 前提与选择：prefab 在 mastersim 分支挂组件并设 onusefn；原版回调负责生成 `halloween_moonpuff` 特效，失败时向使用者推 `on_halloweenmoonpotion_failed`、向目标推零伤害 `attacked`。[装配](../../../raw/current/scripts/prefabs/halloweenpotion_moon.lua#L63-L64) [原版回调](../../../raw/current/scripts/prefabs/halloweenpotion_moon.lua#L13-L26)
- 生效与撤销：动作层（actions.lua）先拒绝无目标／燃烧／阴燃／冻结的目标，随后才调 Use；药水一旦进入 Use 必被消耗。[动作](../../../raw/current/scripts/actions.lua#L4601-L4611)

```lua
-- 服务端：仿 halloweenpotion_moon 的装配
inst:AddComponent("halloweenpotionmoon")
inst.components.halloweenpotionmoon:SetOnUseFn(function(inst, doer, target, success, transformed_inst, container)
    -- success=false 表示目标不可变异；transformed_inst 为新实体或 nil
end)
```

- 调整与验证：目标侧需要 `halloweenmoonmutable` 组件（原版宿主包括鸟、蝴蝶、兔子、蔬菜、花等）；变异细节（转换表／覆写）在该组件配置。[变异侧](../../../raw/current/scripts/components/halloweenmoonmutable.lua#L28-L35)

## 常见陷阱与限制

- **以为无效使用不消耗：** 对没有 halloweenmoonmutable 的目标用 `Use` 也会走完 onusefn 并删药水；想省材料要在动作候选层拦截，而不是改 Use。[消耗](../../../raw/current/scripts/components/halloweenpotionmoon.lua#L25-L29)
- **在 stackable 物品上直接 Remove：** Use 已处理整组药水只删一件（`stackable:Get():Remove()`），宿主回调里不要再删 inst，否则整组消失。[分支](../../../raw/current/scripts/components/halloweenpotionmoon.lua#L25-L29)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（32 行），核对 halloweenpotion_moon 装配与回调、HALLOWEENMOONMUTATE 动作与候选、halloweenmoonmutable:Mutate 的返回语义。未逐项审阅各可变异目标的转换表。
- 结构检查：2026-10-28 validate_page.py 通过；validate_source_links.py --strict 锚点通过（见批级记录）。
- 游戏运行：未运行；变异结果未实测。

## 相关链接

- [[reference/components/halloweenmoonmutable]]：月突变目标端：原地换实体与迁血量。
