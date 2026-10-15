---
title: "planardamage"
created: "2026-10-11"
updated: "2026-10-28"
type: "component"
summary: "给武器或攻击者提供独立位面伤害，配置来源修饰并区分普通伤害倍率。"
aliases: ["位面伤害", "位面攻击", "伤害加成来源", "planardamage"]
tags: ["component", "equipment", "mechanism"]
dst_build: "steam-buildid:26230496"
sources: ["components/combat.lua", "components/planardamage.lua", "components/spdamageutil.lua", "components/weapon.lua", "modutil.lua", "prefabs/sword_lunarplant.lua"]
reliability: "verified"
validation: "source-reviewed"
status: "active"
---

# planardamage

## 快速判断

- 职责与适用：提供 planar 特殊伤害数值；武器 GetDamage 与攻击者 combat 会收集，组件本身不主动造成伤害。[components/spdamageutil.lua:23](../../../raw/current/scripts/components/spdamageutil.lua#L23 [components/combat.lua:891](../../../raw/current/scripts/components/combat.lua#L891
- 优先使用：SetBaseDamage；可撤销效果用 AddBonus／AddMultiplier 与相应 Remove。[components/planardamage.lua:10](../../../raw/current/scripts/components/planardamage.lua#L10
- 重要副作用：公式为基础值 × 来源乘数 + 来源加值；武器和攻击者可共同提供特殊伤害。[components/planardamage.lua:18](../../../raw/current/scripts/components/planardamage.lua#L18 [components/combat.lua:898](../../../raw/current/scripts/components/combat.lua#L898
- 修改边界：普通伤害倍率不是全部应用到位面伤害；防御由 [[reference/components/planardefense]] 与特殊伤害路径处理。[components/combat.lua:948](../../../raw/current/scripts/components/combat.lua#L948

## 关键接口

| 接口／字段 | 约定 | 作用与证据 |
|---|---|---|
| `SetBaseDamage(damage)`／`GetDamage()` | number／number；默认基础 0 | 设置基础／取得计算后数值；只有正特殊伤害进入 CollectSpDamage。[components/planardamage.lua:3](../../../raw/current/scripts/components/planardamage.lua#L3 [components/spdamageutil.lua:57](../../../raw/current/scripts/components/spdamageutil.lua#L57 |
| `AddMultiplier(src, mult, key?)`／`RemoveMultiplier(src, key?)` | 来源、乘数、可选键 | 默认乘数总值 1；显式匹配来源／键用于撤销。[components/planardamage.lua:24](../../../raw/current/scripts/components/planardamage.lua#L24 |
| `AddBonus(src, bonus, key?)`／`RemoveBonus(src, key?)` | 来源、绝对加值、可选键 | 默认加值 0，位于乘法后；动态存档恢复需业务重建，本组件无 OnSave。[components/planardamage.lua:38](../../../raw/current/scripts/components/planardamage.lua#L38 |

## 典型接入

### 给原版长矛增加 10 点基础位面伤害

目标模组 `modmain.lua`；沿用 spear 的普通攻击与资源。位面组合原版参照为剑的 planardamage 初始化，消费由 weapon:GetDamage。[prefabs/sword_lunarplant.lua:259](../../../raw/current/scripts/prefabs/sword_lunarplant.lua#L259 [components/weapon.lua:94](../../../raw/current/scripts/components/weapon.lua#L94 [modutil.lua:593](../../../raw/current/scripts/modutil.lua#L593

```lua
AddPrefabPostInit("spear", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    if inst.components.planardamage == nil then inst:AddComponent("planardamage") end
    inst.components.planardamage:SetBaseDamage(10)
end)
```

影响使用此长矛经过普通武器攻击链的基础特殊伤害，仍保留原普通伤害和耐久；不使物品在背包里主动伤人。停用后新实例恢复原定义；没有新增特效资源。

## 常见陷阱与限制

- **加在任意携带物品就人物自动增伤**：消费入口收集武器与攻击者，不遍历所有携带物品；携带 buff 应明确给实际攻击者加来源修饰并撤销。[components/combat.lua:889](../../../raw/current/scripts/components/combat.lua#L889
- **伤害翻倍一定翻倍位面值**：combat 对普通和特殊伤害倍率分开处理，不能用普通 multiplier 推断位面输出。[components/combat.lua:938](../../../raw/current/scripts/components/combat.lua#L938 [components/combat.lua:948](../../../raw/current/scripts/components/combat.lua#L948

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；相关调用与示例仍适用，本轮仅重定位证据（锚点 6 处），未运行游戏。

- 源码依据复核：2026-10-17，Steam buildid 25643519；核对范围：来源乘加公式、武器／攻击者收集与普通倍率分工、长矛位面伤害示例，依据见上文对应锚点；未覆盖全部特殊伤害类型与战斗修饰组合。
- 历次结构／语法检查：元数据、源码路径／行号、链接／章节、标签、大小和导航登记检查通过；本页 Lua 代码经 Full Moon 3.0.0／Lua 5.1 语法及原字节往返检查。
- 游戏运行：未运行；案例的多人交互、存档重载和其他模组兼容仍需游戏验证。

- 版本沿用：2026-10-28 原始哈希等价批量沿用-70（26148036→26230496，本页 sources 所列文件原始字节全部未变），页内未决项零标记、符号反查无命中、严格链接通过；批次依据见 [[updates/build-26230496/index#wiki-沿用记录]]。
相关：[[reference/components/weapon]]；[[reference/components/planardefense]]。
