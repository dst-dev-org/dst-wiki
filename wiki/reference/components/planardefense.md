---
title: "planardefense"
created: "2026-10-11"
updated: "2026-10-28"
type: "component"
summary: "为装备或受击实体设置位面防御，区分普通护甲吸收和特殊伤害抵消。"
aliases: ["位面防御", "位面减伤", "装备位面抗性", "planardefense"]
tags: ["component", "equipment", "mechanism"]
dst_build: "steam-buildid:26230496"
sources: ["components/combat.lua", "components/inventory.lua", "components/planardefense.lua", "components/spdamageutil.lua", "modutil.lua", "prefabs/armor_voidcloth.lua"]
reliability: "verified"
validation: "source-reviewed"
status: "active"
---

# planardefense

## 快速判断

- 职责与适用：提供 planar 特殊防御数值；人物装备通过 inventory 消费，受击实体自身也可走 combat 的 ApplySpDefense。[components/spdamageutil.lua:27](../../../raw/current/scripts/components/spdamageutil.lua#L27 [components/inventory.lua:455](../../../raw/current/scripts/components/inventory.lua#L455 [components/combat.lua:665](../../../raw/current/scripts/components/combat.lua#L665
- 优先使用：SetBaseDefense；临时加值／乘数用有来源的 AddBonus／AddMultiplier。[components/planardefense.lua:10](../../../raw/current/scripts/components/planardefense.lua#L10
- 重要副作用：公式为基础 × 乘数 + 加值；装备特殊防御分摊还可计入其 armor 磨损。[components/planardefense.lua:18](../../../raw/current/scripts/components/planardefense.lua#L18 [components/inventory.lua:481](../../../raw/current/scripts/components/inventory.lua#L481
- 修改边界：它不设置普通护甲吸收，不因处在物品栏就进入装备防御链。[components/inventory.lua:461](../../../raw/current/scripts/components/inventory.lua#L461

## 关键接口

| 接口／字段 | 约定 | 作用与证据 |
|---|---|---|
| `SetBaseDefense(defense)`／`GetDefense()` | number／number；默认基础 0 | 提供数值，不自行抵消攻击。[components/planardefense.lua:3](../../../raw/current/scripts/components/planardefense.lua#L3 |
| `AddMultiplier(src, mult, key?)`／`RemoveMultiplier(src, key?)` | 来源、乘数、可选键 | 默认乘数总值 1；撤销要匹配来源／键。[components/planardefense.lua:24](../../../raw/current/scripts/components/planardefense.lua#L24 |
| `AddBonus(src, bonus, key?)`／`RemoveBonus(src, key?)` | 来源、绝对加值、可选键 | 默认加值 0；动态状态不由本组件自动存档。[components/planardefense.lua:38](../../../raw/current/scripts/components/planardefense.lua#L38 |

## 典型接入

### 给原版木甲增加 10 点基础位面防御

以下放目标模组 `modmain.lua`；复用 armorwood 资源和装备逻辑，参照虚空袍的独立位面防御组件。[prefabs/armor_voidcloth.lua:126](../../../raw/current/scripts/prefabs/armor_voidcloth.lua#L126 [components/inventory.lua:462](../../../raw/current/scripts/components/inventory.lua#L462 [modutil.lua:593](../../../raw/current/scripts/modutil.lua#L593

```lua
AddPrefabPostInit("armorwood", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    if inst.components.planardefense == nil then inst:AddComponent("planardefense") end
    inst.components.planardefense:SetBaseDefense(10)
end)
```

装备时参加特殊防御；卸下退出装备链。与 armor 组合后被抵消的特殊伤害仍可能消耗护甲耐久，不能描述为完全免磨损。停用后新实例恢复原定义，无新增资源。

## 常见陷阱与限制

- **位面防御为百分比**：这里是数值防御，普通 armor.absorb_percent 才是普通吸收比例。[components/spdamageutil.lua:102](../../../raw/current/scripts/components/spdamageutil.lua#L102 [components/inventory.lua:433](../../../raw/current/scripts/components/inventory.lua#L433
- **给物品加防御就所有者免伤**：默认人物路径只查看 equipslots；携带 buff 应明确修改受击实体并配清理。[components/inventory.lua:461](../../../raw/current/scripts/components/inventory.lua#L461

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；相关调用与示例仍适用，本轮仅重定位证据（锚点 2 处），未运行游戏。

- 源码依据复核：2026-10-17，Steam buildid 25643519；核对范围：来源乘加公式、装备防御及 armor 磨损、木甲位面防御示例，依据见上文对应锚点；未覆盖所有受击宿主与其他特殊防御组合。
- 历次结构／语法检查：元数据、源码路径／行号、链接／章节、标签、大小和导航登记检查通过；本页 Lua 代码经 Full Moon 3.0.0／Lua 5.1 语法及原字节往返检查。
- 游戏运行：未运行；案例的多人交互、存档重载和其他模组兼容仍需游戏验证。

- 版本沿用：2026-10-28 原始哈希等价批量沿用-70（26148036→26230496，本页 sources 所列文件原始字节全部未变），页内未决项零标记、符号反查无命中、严格链接通过；批次依据见 [[updates/build-26230496/index#wiki-沿用记录]]。
相关：[[reference/components/armor]]；[[reference/components/planardamage]]。
