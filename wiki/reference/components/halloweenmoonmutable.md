---
title: "halloweenmoonmutable"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "万圣月突变组件：为生物／植物／物品登记突变目标，用 Mutate 在原地生成新实体、迁移血量与容器归属并移除旧实体。"
aliases: ["月突变", "万圣月突变", "lunar mutation", "halloweenmoonmutable", "Mutate"]
tags: [component, creature, item]
dst_build: "steam-buildid:26230496"
sources: ["components/halloweenmoonmutable.lua", "components/halloweenpotionmoon.lua", "components/crop.lua", "prefabs/spider.lua", "prefabs/plant_normal.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# halloweenmoonmutable

## 快速判断

- 职责与适用：给实体登记"被月光药水／月突变触发后变成什么"，`Mutate` 负责生成新实体、转移血量、移交容器或推受击事件并移除旧实体（[组件全文](../../../raw/current/scripts/components/halloweenmoonmutable.lua#L28-L83)）。
- 优先使用：宿主设 `SetPrefabMutated`（支持值或函数）后由突变源调用 `Mutate`；原版突变源是 [halloweenpotionmoon](../../../raw/current/scripts/components/halloweenpotionmoon.lua#L15-L19)（另见 [[reference/components/halloweenpotionmoon]]）。
- 重要副作用：构造加 `halloweenmoonmutable` 标签、移除时撤销（[ctor](../../../raw/current/scripts/components/halloweenmoonmutable.lua#L9)）；突变成功推 `onhalloweenmoonmutate`（[触发](../../../raw/current/scripts/components/halloweenmoonmutable.lua#L59)），植物侧有监听者（[plant_normal.lua:275](../../../raw/current/scripts/prefabs/plant_normal.lua#L275）。
- 修改边界：仅服务端可用（SpawnPrefab／Remove）；宿主已死亡时 `Mutate` 直接返回 nil（[守卫](../../../raw/current/scripts/components/halloweenmoonmutable.lua#L29-L31)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SetPrefabMutated(prefab)` | `prefab: string\|function(inst)->string` | 登记突变目标；`Mutate` 内经 `FunctionOrValue` 求值（[求值](../../../raw/current/scripts/components/halloweenmoonmutable.lua#L45)）。 |
| `SetOnMutateFn(fn)` | `fn(inst, transformed_inst\|nil)` | 突变回调；override 分支下 new_inst 为 nil（[调用](../../../raw/current/scripts/components/halloweenmoonmutable.lua#L38-L39)）。 |
| `SetConversionOverrideFn(fn)` | `fn(inst) -> transformed_inst, container` | 完全接管转换：不再 Spawn／不删旧实体，容器移交由 fn 自理（[分支](../../../raw/current/scripts/components/halloweenmoonmutable.lua#L35-L43)）。 |
| `Mutate(overrideprefab)` | `overrideprefab: string\|nil -> entity\|nil` | 原地生成新实体并 `SetPosition`；双方都有 health 时按百分比迁移（[迁移](../../../raw/current/scripts/components/halloweenmoonmutable.lua#L52-L54)）；有 inventoryitem 容器则 `GiveItem` 新实体，否则给新实体推 `attacked`（damage=0）（[分支](../../../raw/current/scripts/components/halloweenmoonmutable.lua#L70-L75)）。 |

## 典型接入

### 月光药水突变（原版参照）

- 前提与选择：目标在 master 端挂组件并设突变目标，如蜘蛛 `SetPrefabMutated("spider_moon")`（([spider.lua:776-778](../../../raw/current/scripts/prefabs/spider.lua#L776-L778)）。
- 生效与撤销：药水 `Use` 调目标 `Mutate()` 后自毁；成功返回新实体、失败（死亡／无目标 prefab）返回 nil（([halloweenpotionmoon.lua:15-19](../../../raw/current/scripts/components/halloweenpotionmoon.lua#L15-L19)）。
- 原版依据：蜘蛛用 `SetOnMutateFn` 转移跟随领导者（[HalloweenMoonMutate](../../../raw/current/scripts/prefabs/spider.lua#L399-L408)）；作物生长阶段会移除该组件（([crop.lua:274-275](../../../raw/current/scripts/components/crop.lua#L274-L275)）。

```lua
-- 服务端：给自定义生物接月突变（挂在 master postinit）
inst:AddComponent("halloweenmoonmutable")
inst.components.halloweenmoonmutable:SetPrefabMutated("spider_moon") -- 目标 prefab
inst.components.halloweenmoonmutable:SetOnMutateFn(function(inst, new_inst)
    if new_inst ~= nil then
        -- 迁移跟随等宿主状态
    end
end)
```

- 调整与验证：`Mutate` 前无需自检死亡（组件内已守卫）；突变目标可传函数按实例决定。

## 常见陷阱与限制

- **叠堆物品只产出一个**：`stackable` 叠堆突变时移除整堆、只生成单个新实体（[移除](../../../raw/current/scripts/components/halloweenmoonmutable.lua#L64-L68)）。
- **override 分支不删旧实体**：`SetConversionOverrideFn` 路径只推事件与回调，旧实体的移除和新实体的落位完全交给 fn（[分支](../../../raw/current/scripts/components/halloweenmoonmutable.lua#L35-L43)）。
- **容器判定按 inventoryitem**：旧实体在容器／背包里时新实体由 `GiveItem` 放回原容器（[注释与实现](../../../raw/current/scripts/components/halloweenmoonmutable.lua#L61-L72)）；在地上时则对新实体推 `attacked`，用于触发威胁反应而非伤害。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；覆盖组件全文、halloweenpotionmoon 调用链、spider 宿主与回调、crop 移除时机、plant_normal 事件监听。
- 结构检查：本批 validate_page、validate_source_links 及示例 lua 语法探测（见批级记录）。
- 游戏运行：未运行。
