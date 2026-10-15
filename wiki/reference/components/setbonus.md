---
title: "setbonus"
created: "2026-10-21"
updated: "2026-10-28"
type: component
summary: "原版头部与身体同名套装匹配、逐件启停回调；扩展套装效果须成对施加和撤销。"
aliases: ["同套头盔护甲触发额外效果", "套装加成与卸装撤销", "setbonus"]
tags: [component, equipment, callback, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/setbonus.lua", "components/inventory.lua", "constants.lua", "prefabs/armor_lunarplant.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# setbonus

## 快速判断

- 职责与适用：原库存按 HEAD／BODY 两件的 setname 相等判套装，再通知已装备的同名套装件；不是任意装备数量计数器。[匹配与遍历](../../../raw/current/scripts/components/setbonus.lua#L52-L75)
- 优先使用：扩展原套装包 onenabledfn／ondisabledfn，效果成对增删；穿卸装备让库存调用 UpdateSetBonus，不直接改 enabled。[转换回调](../../../raw/current/scripts/components/setbonus.lua#L1-L21) [装备入口](../../../raw/current/scripts/components/inventory.lua#L1382-L1383)
- 重要副作用：先改 enabled 再调回调，重复同状态不重放；头身成立时可启用其他装备槽里同 setname 的组件。[辅助方法](../../../raw/current/scripts/components/setbonus.lua#L1-L21) [遍历](../../../raw/current/scripts/components/setbonus.lua#L67-L75)
- 修改边界：服务端；默认匹配固定头／身，新增栏位模组须另核验自己的库存改写。组件不自动生成数值效果或网络字段。[实现](../../../raw/current/scripts/components/setbonus.lua#L25-L75)

## 关键接口

| 接口／字段 | 条件、作用与证据 |
|---|---|
| `SetSetName(name)` | 断言 EQUIPMENTSETNAMES[string.upper(name)] 存在，再原样存 name；比较时区分大小写，优先用常量值。默认空字符串不代表已注册套装。[方法](../../../raw/current/scripts/components/setbonus.lua#L25-L41) [常量](../../../raw/current/scripts/constants.lua#L2205-L2212) |
| `SetOnEnabledFn(fn)`／`SetOnDisabledFn(fn)` | 设置 fn(item)，不传 owner；原月亮植物甲启停各自增删 damagetyperesist 的 setbonus 键。[setter](../../../raw/current/scripts/components/setbonus.lua#L44-L50) [原效果](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L49-L55) |
| `UpdateSetBonus(inventory,isequipping)` | 头身匹配则逐装备启用同名且尚未启用者；不匹配则逐装备禁用。isequipping=false 还单独禁用已离槽的自身。[实现](../../../raw/current/scripts/components/setbonus.lua#L52-L75) |
| `IsEnabled(setname?)` | 返回当前启用且可选名字匹配；传名字时同样检查常量，不会重算库存。[实现](../../../raw/current/scripts/components/setbonus.lua#L79-L85) |

穿上由 inventory:Equip 调用更新；卸下先清 equipslots，再更新并禁用被卸件。组件被移除时，若原本启用，先禁用自身，再尝试通过 inventoryitem.owner 的库存重算。组件无独立保存钩子；不要把写 enabled 当作可存读的效果协议。[穿上](../../../raw/current/scripts/components/inventory.lua#L1382-L1383) [卸下](../../../raw/current/scripts/components/inventory.lua#L1240-L1244) [移除](../../../raw/current/scripts/components/setbonus.lua#L87-L98)

## 典型接入

### 原月亮植物甲套装启用时，额外叠加 0.9 倍月亮类别受伤贡献

目标模组 `modmain.lua`，只扩展原 armor_lunarplant；资源、套装名、基础抗性和原回调保留，不新增装备注册。原工厂已有 setbonus 和 damagetyperesist；头身搭配仍按原规则。[原安装](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L220-L226) [注册名](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L436)

```lua
AddPrefabPostInit("armor_lunarplant", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local bonus = inst.components.setbonus
    if bonus == nil or inst.components.damagetyperesist == nil then return end
    local enable = bonus.onenabledfn
    local disable = bonus.ondisabledfn
    bonus:SetOnEnabledFn(function(item)
        if enable ~= nil then enable(item) end
        item.components.damagetyperesist:AddResist(
            "lunar_aligned", item, 0.9, "my_mod_set")
    end)
    bonus:SetOnDisabledFn(function(item)
        item.components.damagetyperesist:RemoveResist(
            "lunar_aligned", item, "my_mod_set")
        if disable ~= nil then disable(item) end
    end)
end)
```

0.9 是额外剩余伤害倍率贡献，与原抗性相乘，不把最终总减伤设为 10%。具体标签匹配和库存消费见 [[reference/components/damagetyperesist#关键接口]]。用自定义键避免覆盖原 "setbonus" 键。

本次实例在正常套装启用时施加，脱离原套装时撤销；组件移除走上文清理。只去掉这段代码不会热撤已存在的贡献，停用并完整重载后不再安装扩展。未提供动态改套装名、异常库存改写或新多栏套装方案。

## 常见陷阱与限制

- **新增套装仅传任意字符串**：setter 有常量断言，大小写转换仅用于检查，存值比较仍精确匹配；沿已定义值扩展，新增注册另核验。[名字](../../../raw/current/scripts/components/setbonus.lua#L39-L41) [匹配](../../../raw/current/scripts/components/setbonus.lua#L63)
- **任意两件同名装备就能触发**：默认仅 HEAD／BODY 作门禁，匹配后才遍历其他槽；手／背包组合不自动替代头身。[门禁](../../../raw/current/scripts/components/setbonus.lua#L52-L70)
- **enabled=true 就有防御／伤害**：实际效果在宿主回调；直接赋值会跳过施加，也可能让后续启用不再回调。包装要保留原效果并成对清理。[转换](../../../raw/current/scripts/components/setbonus.lua#L4-L20)

## 验证记录

- 增量复查：2026-10-25，Steam buildid 26148036；锚点重定位（armor_lunarplant +2~+6 段：47–53→49–55、214–220→220–226、430→436；constants +4 段：2201–2208→2205–2212），setbonus/inventory 无变化；P-07 改 OnHit_Vines/OnBlocked/OnHuskBlocked 触发门控，不涉 OnEnabled/OnDisabledSetBonus 与本页示例链，结论维持。
- 源码审阅：2026-10-21，Steam buildid 25643519；覆盖完整组件、原库存穿卸调用、套装常量、月亮植物甲安装／原启停效果与注册；倍率消费者见关联页。未审校全部套装效果、多栏模组、异常动态替换或同步 UI。
- 结构检查：本批格式、来源、链接／锚点、大小与登记；完整回调配对例经已有 Full Moon Lua 5.1 syntax-only。
- 游戏运行：未运行；换装顺序、破损／修复、存读、删除套装件和第三方装备栏待实测。
