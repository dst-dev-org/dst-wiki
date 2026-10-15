---
title: "armor"
created: "2026-10-11"
updated: "2026-10-28"
type: "component"
summary: "配置装备普通伤害吸收与护甲耐久，说明多件护甲、破损和位面伤害的分工。"
aliases: ["护甲耐久", "护甲吸收", "装备减伤", "护甲修复", "armor"]
tags: ["component", "equipment", "lifecycle", "callback"]
dst_build: "steam-buildid:26230496"
sources: ["components/armor.lua", "components/inventory.lua", "prefabs/armor_wood.lua"]
reliability: "verified"
validation: "source-reviewed"
status: "active"
---

# armor

## 快速判断

- 职责与适用：装备普通伤害吸收及耐久；在人物 inventory 装备槽中参与 ApplyDamage，携带不贡献吸收。[components/inventory.lua:408](../../../raw/current/scripts/components/inventory.lua#L408
- 优先使用：初始化 InitCondition(amount, absorb_percent)；改吸收用 SetAbsorption，修复用 Repair。[components/armor.lua:39](../../../raw/current/scripts/components/armor.lua#L39 [components/armor.lua:166](../../../raw/current/scripts/components/armor.lua#L166
- 重要副作用：SetCondition 低至 0 会回调 onfinished，并默认删除实体；保留空护甲需 SetKeepOnFinished(true)。[components/armor.lua:93](../../../raw/current/scripts/components/armor.lua#L93
- 修改边界：吸收率用 0～1 比例；多件护甲普通伤害取最高吸收比例并分摊耐久，不把比例直接相加。[components/inventory.lua:433](../../../raw/current/scripts/components/inventory.lua#L433

## 关键接口

| 接口／字段 | 约定 | 作用与证据 |
|---|---|---|
| `InitCondition(amount, absorb_percent)` | 正数耐久；0～1 比例推荐域 | 同设 maxcondition 和 condition；会补满，勿用作保留存档比例的日常修复。[components/armor.lua:39](../../../raw/current/scripts/components/armor.lua#L39 |
| `SetAbsorption(percent)` | 比例 | 不改当前／最大耐久；SetTags 与弱点可改变实际吸收／磨损。[components/armor.lua:85](../../../raw/current/scripts/components/armor.lua#L85 [components/armor.lua:126](../../../raw/current/scripts/components/armor.lua#L126 |
| `SetPercent(pct)`／`SetCondition(amount)` | 比例／绝对耐久 | 上限限制；indestructible 时 SetCondition 不做事。[components/armor.lua:89](../../../raw/current/scripts/components/armor.lua#L89 |
| `SetOnFinished(fn)`／`SetKeepOnFinished(flag?)` | `(inst)`／省略也开启保留 | 回调后默认删除；普通修复 Repair 增加 condition 并调用可选 onrepair。[components/armor.lua:31](../../../raw/current/scripts/components/armor.lua#L31 [components/armor.lua:110](../../../raw/current/scripts/components/armor.lua#L110 [components/armor.lua:166](../../../raw/current/scripts/components/armor.lua#L166 |
| `InitIndestructible(absorb_percent)` | 比例 | 标为不损耐久；随后 InitCondition 不会自动清此标志。[components/armor.lua:45](../../../raw/current/scripts/components/armor.lua#L45 |

（26148036 核验：SetCondition 现以 `math.clamp(amount, 0, maxcondition)` 钳制写入 [0, maxcondition]，负值／越界入参不再写入负耐久（旧版仅 `math.min` 限上限）——[components/armor.lua:98](../../../raw/current/scripts/components/armor.lua#L98）

## 典型接入

### 将原版木甲基础吸收率改为 90%

放目标模组 `modmain.lua`，复用 armorwood 资源与耐久，不重新 InitCondition。源码文件是 armor_wood.lua，注册名为 armorwood。[prefabs/armor_wood.lua:68](../../../raw/current/scripts/prefabs/armor_wood.lua#L68 [prefabs/armor_wood.lua:83](../../../raw/current/scripts/prefabs/armor_wood.lua#L83

```lua
AddPrefabPostInit("armorwood", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    if inst.components.armor ~= nil then
        inst.components.armor:SetAbsorption(0.9)
    end
end)
```

只在装备槽中参与普通伤害吸收；不保证所有伤害减免 90%，不改变位面防御。卸下后不再被装备链消费，停用后新实例恢复原吸收。

## 常见陷阱与限制

- **两件 80% 护甲等于 160% 防御**：普通伤害取最大吸收率；消耗按各件比例分摊，特殊伤害另处理。[components/inventory.lua:433](../../../raw/current/scripts/components/inventory.lua#L433
- **护甲自动吸收所有位面伤害**：特殊防御由 SpDamageUtil 取 planardefense；有 armor 的位面防御装备还可能损耐久。[components/inventory.lua:455](../../../raw/current/scripts/components/inventory.lua#L455
- **破损回调之后还能安全操作实体**：SetCondition 回调后可删除；事件 payload 也可能在归零／移除前发出，调用后核验 IsValid。[components/armor.lua:99](../../../raw/current/scripts/components/armor.lua#L99

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 的全部变更文件对照 build-26148036 更新报告逐条核验，锚点重定位 0 处、结论修订 1 处（SetCondition 钳制核验，见正文"26148036 核验"标注）；未覆盖：运行验证。

- 源码依据复核：2026-10-17，Steam buildid 25643519；核对范围：SetCondition 归零回调／删除、多甲取最大吸收并分摊耐久、木甲只改吸收率示例，依据见上文对应锚点；未覆盖全部特殊伤害组合与破损回调重入。
- 历次结构／语法检查：元数据、源码路径／行号、链接／章节、标签、大小和导航登记检查通过；本页 Lua 代码经 Full Moon 3.0.0／Lua 5.1 语法及原字节往返检查。
- 游戏运行：未运行；案例的多人交互、存档重载和其他模组兼容仍需游戏验证。

- 版本沿用：2026-10-28 原始哈希等价批量沿用-70（26148036→26230496，本页 sources 所列文件原始字节全部未变），页内未决项零标记、符号反查无命中、严格链接通过；批次依据见 [[updates/build-26230496/index#wiki-沿用记录]]。
相关：[[reference/components/combat#伤害与事件顺序|受击与护甲入口]]；[[reference/components/planardefense]]；[[reference/components/equippable]]；[[reference/components/trader#给物品投喂材料]]。
