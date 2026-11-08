---
title: "暗夜剑（nightsword）"
created: "2026-11-06"
updated: "2026-11-08"
type: prefab
summary: "原版暗影操控器武器 nightsword：HANDS 槽近战武器，伤害 68、耐久 100 仅攻击磨损，装备期持续扣理智（魔法系，Maxwell 免疫），影子等级 2 供 Maxwell 影像随从增伤；耗尽整实体移除，5 皮肤各配装备跟随攻击爆发 FX。"
aliases: ["nightsword", "暗夜剑", "Dark Sword", "暗影武器"]
tags: [prefab, item, equipment]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/nightsword.lua", "prefabs/nightsword_cotl_fx.lua", "prefabs/shadowchesspieces.lua", "prefabs/skinprefabs.lua", "prefabs/magicprototyper.lua", "prefabs/waxwell.lua", "prefabs/wanda.lua", "prefabs/walter.lua", "prefabs/gestalt_guard.lua", "prefabs/hats.lua", "components/weapon.lua", "components/finiteuses.lua", "components/equippable.lua", "components/inspectable.lua", "components/inventoryitem.lua", "components/sanity.lua", "components/shadowlevel.lua", "components/combat.lua", "stategraphs/SGwilson.lua", "stategraphs/SGwilson_client.lua", "stategraphs/SGshadowwaxwell.lua", "standardcomponents.lua", "prefabskin.lua", "tuning.lua", "constants.lua", "recipes.lua", "recipes_filter.lua", "map/boons.lua", "worldgen_main.lua", "wx78_moduledefs.lua", "scenarios/chest_insanity.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 暗夜剑（nightsword）

## 快速判断

- 职责与适用：`prefabs/nightsword.lua` 顶层 `fn` 直接注册的单注册可拾取手持武器（物品＋装备）：weapon 伤害＋finiteuses 耐久＋equippable HANDS 槽，特色是 `equippable.dapperness`（装备期持续扣理智，标记魔法系）与 shadowlevel 影子等级 2；无 tool、无 onattack 定制、同文件无 quagmire 分支。[源文件](../../../raw/current/scripts/prefabs/nightsword.lua#L28-L87)
- 优先使用：复刻"伤害＋耐久＋理智代价"手持武器按[注册与构造](#注册与构造)组装；改伤害／耐久／理智惩罚查数值表与[典型接入](#典型接入)；从零造手持武器查 [[cookbook/equipment/hand]]；无耐久/耐久型/吸血型对照 [[reference/prefabs/hambat]]／[[reference/prefabs/spear]]／[[reference/prefabs/batbat]]。
- 重要副作用：装备期间按 `TUNING.CRAZINESS_MED` 每秒约 −0.333 理智（普通角色；Maxwell 免疫、Wanda 按年龄减免，见导航）；耐久耗尽 `SetOnFinished(inst.Remove)` 整个实体移除；影子等级 2 被 Maxwell 影像随从增伤消费。[dapperness 消费](../../../raw/current/scripts/components/sanity.lua#L513-L523)、[耗尽移除](../../../raw/current/scripts/prefabs/nightsword.lua#L69)
- 修改边界：标签与 floater 挂公共端、组件挂主模拟（[分界](../../../raw/current/scripts/prefabs/nightsword.lua#L55-L59)）；皮肤事件在装备回调内、皮肤 FX 由 prefabskin 事件接线；dapperness 无独立组件（是 equippable 字段，消费链按角色覆盖）；不覆盖皮肤贴图内容。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `nightsword` | 无包装工厂：`fn()` 直接构造，无 postinit | 物品＋装备；皮肤经 `nightsword_init_fn`（带 equipped 监听，见[皮肤](#皮肤装备跟随-fx)）。[注册](../../../raw/current/scripts/prefabs/nightsword.lua#L89)、[皮肤 init](../../../raw/current/scripts/prefabskin.lua#L3238-L3253) |

共同初始化（[nightsword.lua#L28-L87](../../../raw/current/scripts/prefabs/nightsword.lua#L28-L87)）：实体组件＋`MakeInventoryPhysics` → `SetBank("nightmaresword")`＋`SetBuild("nightmaresword")`＋`PlayAnimation("idle")` → 公共端标签 `shadow_item`／`shadow`／`sharp`／`weapon`（[pristine 镜像注释](../../../raw/current/scripts/prefabs/nightsword.lua#L46-L47)）／`shadowlevel`（[pristine 镜像注释](../../../raw/current/scripts/prefabs/nightsword.lua#L49-L50)）→ `MakeInventoryFloatable("med", 0.05, {1.0, 0.4, 1.0}, true, -17.5, {sym_build = "swap_nightmaresword", bank = "nightmaresword"})` → `SetPristine`（非主模拟返回）→ 主模拟依次 weapon → finiteuses → inspectable → inventoryitem → equippable → shadowlevel → `MakeHauntableLaunch`。装备槽不逐 prefab 设置，用组件默认 HANDS。[equippable 默认](../../../raw/current/scripts/components/equippable.lua#L29)、[EQUIPSLOTS.HANDS](../../../raw/current/scripts/constants.lua#L656)

名称键经简中 .po `msgctxt` 核验：`STRINGS.NAMES.NIGHTSWORD`＝暗夜剑（Dark Sword）。[.po](../../../raw/current/scripts/languages/chinese_s.po#L346179-L346182)

数值（tuning 键＋实际值；`wilson_attack`＝34×倍率 1，[tuning.lua#L31-L37](../../../raw/current/scripts/tuning.lua#L31-L37)）：

| 项 | 键 | 实际值 |
|---|---|---|
| 武器伤害 | `TUNING.NIGHTSWORD_DAMAGE`＝`wilson_attack`×2 | 68 生命值（源码推导，[tuning.lua#L347](../../../raw/current/scripts/tuning.lua#L347)） |
| 耐久上限 | `TUNING.NIGHTSWORD_USES` | 100（[tuning.lua#L174](../../../raw/current/scripts/tuning.lua#L174)） |
| 每次攻击磨损 | weapon `attackwear`（从未赋值） | 1 耐久点/次，且为唯一消耗路径（无 tool、无 `SetConsumption`；[OnAttack 结算](../../../raw/current/scripts/components/weapon.lua#L111-L120)） |
| 装备理智变化 | `TUNING.CRAZINESS_MED`＝−100/`day_time`（＝−100/300） | ≈−0.333 理智/秒（源码推导；`is_magic_dapperness = true`，[tuning.lua#L2225](../../../raw/current/scripts/tuning.lua#L2225)、[接线](../../../raw/current/scripts/prefabs/nightsword.lua#L78-L79)） |
| 影子等级 | `TUNING.NIGHTSWORD_SHADOW_LEVEL` | 2（[tuning.lua#L2869](../../../raw/current/scripts/tuning.lua#L2869)） |

获取途径（均在快照内核验）：

| 途径 | 内容与证据 |
|---|---|
| 配方：噩梦燃料 5＋活木头 1，`TECH.MAGIC_THREE`（＝{MAGIC=3}，暗影操控器 researchlab3 档） | [配方](../../../raw/current/scripts/recipes.lua#L483)、[TECH](../../../raw/current/scripts/constants.lua#L1250)、[暗影操控器树](../../../raw/current/scripts/tuning.lua#L1276-L1278)、[researchlab3](../../../raw/current/scripts/prefabs/magicprototyper.lua#L219) |
| 影子棋子掉落：shadow_rook／shadow_knight／shadow_bishop 共用 `lootsetfn`，等级 ≥3 时追加 nightsword＋armor_sanity＋shadowheart＋噩梦燃料 | [lootsetfn](../../../raw/current/scripts/prefabs/shadowchesspieces.lua#L47-L68)、[挂接](../../../raw/current/scripts/prefabs/shadowchesspieces.lua#L402)、[三注册](../../../raw/current/scripts/prefabs/shadowchesspieces.lua#L552-L554) |
| 世界生成：Rare 组 `Level4Boon` 的 `item_area` 从 8 件物品随机抽 1（含 nightsword）；Rare 98% 概率被跳过 | [boons](../../../raw/current/scripts/map/boons.lua#L163-L166)、[Rare 跳过](../../../raw/current/scripts/worldgen_main.lua#L271)、[数量循环](../../../raw/current/scripts/worldgen_main.lua#L368-L372) |
| scenario 陷阱箱：`chest_insanity` 的 loot 含 nightsword×1（开箱触发陷阱时经 `AddChestItems` 装入）；仅对应 scenario 启用时生效 | [nightsword 条目](../../../raw/current/scripts/scenarios/chest_insanity.lua#L9-L11)、[装箱](../../../raw/current/scripts/scenarios/chest_insanity.lua#L29) |

配方过滤归 crafting 菜单 WEAPONS 与 MAGIC 两类。[WEAPONS](../../../raw/current/scripts/recipes_filter.lua#L1035)、[MAGIC](../../../raw/current/scripts/recipes_filter.lua#L1452)

配方原料中文名经 .po 核验：Nightmare Fuel=噩梦燃料（[L346150-L346152](../../../raw/current/scripts/languages/chinese_s.po#L346150-L346152)）、Living Log=活木头（[L344755-L344757](../../../raw/current/scripts/languages/chinese_s.po#L344755-L344757)）。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：weapon | 仅 `SetDamage(NIGHTSWORD_DAMAGE)`；无 onattack／射程／投射 | 主模拟；`OnAttack` 推 `weapononattack` 事件后按 `(attackwear or 1)×multipliers` 扣耐久 | [[reference/components/weapon#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/nightsword.lua#L61-L62)、[OnAttack](../../../raw/current/scripts/components/weapon.lua#L105-L121) |
| 组件：finiteuses | 上限＝当前＝100；`SetOnFinished(inst.Remove)`；未设 `SetConsumption` | 主模拟；耗尽先加 `usesdepleted` 标签再调回调 | [[reference/components/finiteuses#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/nightsword.lua#L66-L69)、[SetUses](../../../raw/current/scripts/components/finiteuses.lua#L64-L79) |
| 组件：equippable | `SetOnEquip`／`SetOnUnequip`：覆盖 `swap_object`、ARM_carry／ARM_normal 显隐、皮肤事件；`dapperness = CRAZINESS_MED`＋`is_magic_dapperness = true` | 主模拟；槽默认 HANDS；dapperness 在每次理智 Recalc 时读取（见下条） | [[reference/components/equippable#关键接口]]；[接线](../../../raw/current/scripts/prefabs/nightsword.lua#L75-L79)、[onequip](../../../raw/current/scripts/prefabs/nightsword.lua#L7-L17) |
| 数据：dapperness 消费链 | 无独立 dapperness 组件：sanity `Recalc` 遍历装备槽逐件取 `GetDapperness`；`Wet_Item_Dapperness` 额外叠加湿装备修正 | 普通角色默认路径；Walter 仅认 `is_magic_dapperness`（本剑受影响）；Maxwell 对 `shadow_item` 负值 ×0；Wanda 按年龄 ×1／0.33／0；WX-78 理智电路只放大正值 | [[reference/components/sanity#关键接口]]；[Recalc](../../../raw/current/scripts/components/sanity.lua#L513-L523)、[Walter](../../../raw/current/scripts/prefabs/walter.lua#L754-L759)、[Maxwell](../../../raw/current/scripts/prefabs/waxwell.lua#L179-L183)、[Wanda](../../../raw/current/scripts/prefabs/wanda.lua#L201-L213)、[WX-78](../../../raw/current/scripts/wx78_moduledefs.lua#L224-L244) |
| 组件：shadowlevel | `SetDefaultLevel(TUNING.NIGHTSWORD_SHADOW_LEVEL)`＝2 | 主模拟；pristine 加同名标签镜像 | [[reference/components/shadowlevel#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/nightsword.lua#L81-L82) |
| 组件：inspectable／inventoryitem | 检视与拾取 | 主模拟 | [[reference/components/inspectable#关键接口]]、[[reference/components/inventoryitem#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/nightsword.lua#L71-L73) |
| 组件：floater | 公共端 `MakeInventoryFloatable("med", 0.05, {1.0, 0.4, 1.0}, true, -17.5, swap_data)`，浮水换 swap_nightmaresword | 公共端；`SetBankSwapOnFloat(true, -17.5, swap_data)` | [[reference/components/floater#关键接口]]；[调用](../../../raw/current/scripts/prefabs/nightsword.lua#L52-L53)、[helper](../../../raw/current/scripts/standardcomponents.lua#L1772-L1790) |
| 组件：hauntable | `MakeHauntableLaunch`：被作祟按 `HAUNT_CHANCE_ALWAYS` 弹飞 | 主模拟 | [[reference/components/hauntable#按效果选工厂]]；[调用](../../../raw/current/scripts/prefabs/nightsword.lua#L84)、[helper](../../../raw/current/scripts/standardcomponents.lua#L1053-L1069) |
| API／数据：`sharp` 标签 | `Combat:GetImpactSound` 按武器 `sharp` 切命中音效档 | 命中穿甲结算时读取 | [[reference/components/combat#关键接口]]；[消费](../../../raw/current/scripts/components/combat.lua#L745-L747) |
| API／数据：`shadow` 标签 | 装备态攻击呼啸音换 `dontstarve/wilson/attack_nightsword`（服务端 SG 与客户端预测各一处） | 持握攻击时读取 | [服务端](../../../raw/current/scripts/stategraphs/SGwilson.lua#L10921)、[客户端](../../../raw/current/scripts/stategraphs/SGwilson_client.lua#L5160) |
| API／数据：`shadow_item` 标签 | 消费端：Maxwell／Wanda 理智减免（见上）；Wanda 装备 shadow_item 武器时攻击伤害乘年龄系数（老×1.75／中×1.2／青×1，经 `customdamagemultfn`，作用于武器伤害 68）＋命中影子 FX；gestalt_guard 对装备者仇恨升到 3 级；voidcloth 套装 buff 条件之一（另需 `planardamage` 组件，本剑不满足） | 各自消费时机判定 | [Wanda 乘数](../../../raw/current/scripts/prefabs/wanda.lua#L271-L285)、[乘数与 FX 接线](../../../raw/current/scripts/prefabs/wanda.lua#L431-L432)、[Wanda FX](../../../raw/current/scripts/prefabs/wanda.lua#L287-L305)、[gestalt_guard](../../../raw/current/scripts/prefabs/gestalt_guard.lua#L63)、[voidcloth 条件](../../../raw/current/scripts/prefabs/hats.lua#L4006-L4011) |
| 关联 Prefab：shadowwaxwell 系影像随从（Wiki 待建） | shadowlevel 消费：protector 形态伤害＝20＋4×领导者装备 shadowlevel 总和（本剑装备态贡献 2）；Maxwell 装备时按 level 播报 | Maxwell 专属；只统计装备槽内物品 | [protector 增伤](../../../raw/current/scripts/stategraphs/SGshadowwaxwell.lua#L144-L160)、[20＋4/级](../../../raw/current/scripts/tuning.lua#L2805-L2806)、[装备播报](../../../raw/current/scripts/prefabs/waxwell.lua#L225-L241) |
| 关联 Prefab：shadow_rook／shadow_knight／shadow_bishop | 等级 ≥3 时 `lootsetfn` 把 nightsword 加入掉落 | 影子棋子被击杀时，主模拟 | [掉落分支](../../../raw/current/scripts/prefabs/shadowchesspieces.lua#L59-L61) |
| 皮肤 | 5 种 Elegant／Woven 皮肤，各配独立 `*_fx` 攻击爆发 FX；中文名与接线见下小节 | 装备态跟随宿主 | 见[皮肤小节](#皮肤装备跟随-fx) |
| 资源：anim／swap 动画包 | nightmaresword.zip（bank＋build）／swap_nightmaresword.zip（持握符号） | 静态声明 | [声明](../../../raw/current/scripts/prefabs/nightsword.lua#L1-L5) |

## 典型接入

### 调整暗夜剑的伤害、耐久与理智惩罚

- 前提与选择：服务端 `modmain.lua`；nightsword 无专属机制回调，`AddPrefabPostInit` 直接改组件字段。dapperness 不是组件而是 equippable 字段，直接赋值即可。
- 生效与撤销：postinit 在实体构造（含读档重建）时执行；dapperness 每次理智 Recalc 现读现算，装备态立即生效；耐久当前值仍按存档 `OnLoad` 恢复。停用模组后新实例恢复原上限，存档中的 `uses` 仍可能高于原上限。
- 原版依据：伤害在 weapon、耐久在 finiteuses、理智在 equippable.dapperness，三条独立接线（见[关联机制导航](#关联机制导航)）。

```lua
-- modmain.lua：暗夜剑伤害 80、耐久 150，基础 dapperness 设为 0（湿装备仍叠加湿度修正）
AddPrefabPostInit("nightsword", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    inst.components.weapon:SetDamage(80)
    inst.components.finiteuses:SetMaxUses(150)
    inst.components.finiteuses:SetUses(150)
    inst.components.equippable.dapperness = 0 -- 只清基础值；装备潮湿时 GetDapperness 仍加 WET_ITEM_DAPPERNESS
    -- 影子等级若要调整：inst.components.shadowlevel:SetDefaultLevel(2)
end)
```

- 调整与验证：伤害单位为生命值/击、耐久为点/次、dapperness 为理智/秒（负值扣减）。把 `dapperness` 设为 0 只清基础值；装备潮湿时 `GetDapperness` 仍叠加 `WET_ITEM_DAPPERNESS`，无法完全免除湿装惩罚。三项都在构造时读 `TUNING` 一次性赋入组件字段：`TUNING.NIGHTSWORD_DAMAGE`／`NIGHTSWORD_USES`／`CRAZINESS_MED` 运行时改动只影响新实例，已有实例需 postinit 改字段或等读档重建；sanity `Recalc` 每次读的是 equippable 字段，不再查 `TUNING`。未运行验证。

## 皮肤（装备跟随 FX）

皮肤接线与常见手持物一致：`GetSkinBuild()` 非空时装备／卸装推 `equipskinneditem`／`unequipskinneditem`，持握贴图用 `OverrideItemSkinSymbol`（[onequip](../../../raw/current/scripts/prefabs/nightsword.lua#L7-L17)）。已注册 5 种皮肤，`nightsword_init_fn` 在主模拟额外监听 equipped／unequipped／onremove：装备时生成该皮肤的 `fx_prefab` 实体、`FollowSymbol` 挂到持有者 `swap_object` 符号（偏移常量 `NIGHTSWORD_FX_OFFSETS`），卸装／移除时删除；FX 在宿主攻击动画 `atk` 0.13 秒处一次性爆发粒子（[init](../../../raw/current/scripts/prefabskin.lua#L3220-L3253)、[常量](../../../raw/current/scripts/constants.lua#L2944)）。各 FX 构造细节见互链 FX 页（已建 5 页）；`nightsword_curve_fx` 全文已核验，其余 4 个 FX 文件未逐行审阅。

| 皮肤（官方简中名） | fx_prefab | 注册与名称证据 |
|---|---|---|
| 咩咩之刃（nightsword_cotl，CotL 联动） | `nightsword_cotl_fx` | [[reference/prefabs/nightsword_cotl_fx]]；[注册](../../../raw/current/scripts/prefabs/skinprefabs.lua#L8667-L8678)、[名](../../../raw/current/scripts/languages/chinese_s.po#L401360-L401362) |
| 扭曲匕首（nightsword_curve） | `nightsword_curve_fx` | [[reference/prefabs/nightsword_curve_fx]]；[注册](../../../raw/current/scripts/prefabs/skinprefabs.lua#L8680-L8691)、[名](../../../raw/current/scripts/languages/chinese_s.po#L401365-L401367) |
| 魔光剑（nightsword_lightsbane） | `nightsword_lightsbane_fx` | [[reference/prefabs/nightsword_lightsbane_fx]]；[注册](../../../raw/current/scripts/prefabs/skinprefabs.lua#L8693-L8704)、[名](../../../raw/current/scripts/languages/chinese_s.po#L401370-L401372) |
| 腐败刀锋（nightsword_sharp） | `nightsword_sharp_fx` | [[reference/prefabs/nightsword_sharp_fx]]；[注册](../../../raw/current/scripts/prefabs/skinprefabs.lua#L8706-L8717)、[名](../../../raw/current/scripts/languages/chinese_s.po#L401375-L401377) |
| 诅咒之剑（nightsword_wizard） | `nightsword_wizard_fx` | [[reference/prefabs/nightsword_wizard_fx]]；[注册](../../../raw/current/scripts/prefabs/skinprefabs.lua#L8719-L8730)、[名](../../../raw/current/scripts/languages/chinese_s.po#L401380-L401382) |

## 常见陷阱与限制

- **误区：理智惩罚随时随地生效**：dapperness 只被 sanity `Recalc` 在装备槽内读取（遍历 `inventory.equipslots`），放背包／容器／地面无任何理智效果。[消费](../../../raw/current/scripts/components/sanity.lua#L513-L523)
- **边界：理智惩罚按角色覆盖**：Maxwell 对 `shadow_item` 负 dapperness ×0（`WAXWELL_SHADOW_ITEM_RESISTANCE`＝0，[tuning.lua#L2853](../../../raw/current/scripts/tuning.lua#L2853)）即完全免疫；Wanda 老年 ×0／中年 ×0.33／青年 ×1；Walter 只认魔法系 dapperness，本剑受影响；其余角色走默认路径。[Maxwell 端](../../../raw/current/scripts/prefabs/waxwell.lua#L179-L183)
- **边界：耐久耗尽即整个实体 Remove**：`SetOnFinished(inst.Remove)`，无残骸掉落；`OnSave` 只在当前数≠上限时存 `uses`。[回调](../../../raw/current/scripts/prefabs/nightsword.lua#L69)、[存档](../../../raw/current/scripts/components/finiteuses.lua#L48-L52)
- **边界：影子等级无普适加成**：`shadowlevel` 在快照内只被 Maxwell 影像随从增伤、装备播报与 `shadowmagic` 角色检视描述附加播报消费（[stringutil.lua#L343](../../../raw/current/scripts/stringutil.lua#L343)），普通角色装备无效果，不要当通用暗影伤害加成。[消费](../../../raw/current/scripts/stategraphs/SGshadowwaxwell.lua#L154-L159)
- **边界：不触发暗影碎布（voidcloth）套装 buff**：套装条件要求 HANDS 槽武器带 `shadow_item` 标签**且**有 `planardamage` 组件，nightsword 无 planardamage，不满足。[条件](../../../raw/current/scripts/prefabs/hats.lua#L4006-L4011)
- **边界：皮肤 FX 是"装备跟随"型而非"命中挂目标"型**：FX 装备时挂宿主 `swap_object`，随攻击动画自爆发；与 batbat 那类 `SKIN_FX_PREFAB` 命中 FX（挂目标）是两套协议，改哪把武器就查哪套接线。[接线](../../../raw/current/scripts/prefabskin.lua#L3220-L3229)
- **边界：`weapon`／`shadowlevel` 标签是公共端手打的 pristine 镜像**（客户端优化用）；同文件无 `GetServerGameMode` 调用，quagmire 下组件照常挂载。[注释](../../../raw/current/scripts/prefabs/nightsword.lua#L46-L50)

## 验证记录

- 源码审阅：2026-11-07（第三轮检视修订：复核 dapperness 湿装叠加，示例改为基础值归零）。早前 2026-11-06，Steam buildid 26335394（raw/current/BUILD.txt）。核验 prefabs/nightsword.lua 全文（fn 组装、onequip／onunequip 皮肤事件、五组件接线、floater／MakeHauntableLaunch 参数）；weapon／finiteuses／equippable／inspectable／inventoryitem／shadowlevel 关键方法；sanity Recalc 的装备 dapperness 消费与四角色覆盖（Walter／Waxwell／Wanda／WX-78 电路，含各自接线行）；shadowlevel 消费（SGshadowwaxwell protector 增伤、waxwell 装备播报、stringutil shadowmagic 检视附加）；`sharp`（GetImpactSound）／`shadow`（SGwilson 攻击音＋客户端）／`shadow_item`（Wanda 乘数与 FX、gestalt_guard 仇恨、voidcloth 套装条件）消费端；tuning 五键与 `wilson_attack`／`day_time` 基值；配方行、`TECH.MAGIC_THREE` 与暗影操控器（researchlab3）科技树；影子棋子掉落分支与三注册；Level4Boon 与 worldgen 放置链；`chest_insanity` scenario loot；5 皮肤注册、官方中文名与 prefabskin 事件接线（`nightsword_cotl_fx` 全文已核验，其余 4 个 FX 文件未逐行审阅，另有 5 个已建 FX 页互链）；recipes_filter 两分类。名称键以 chinese_s.po `msgctxt` 定向核验。同日独立复审修正三处：demo 补 `GLOBAL.TheWorld` 前缀、dapperness 运行时改 `TUNING` 只影响新实例、Wanda 影子武器伤害为乘数而非改写。
- 游戏运行：未运行。未覆盖：实机伤害与理智消耗速率、皮肤实装外观与粒子观感、影子棋子掉落实机概率与等级获取路径、boon 实际落点与频率；68／−0.333／20＋4×2 均为源码推导，非运行测量。
- 结构检查：validate_page.py 0 error（128 行／21743 字节，standard 档）与 validate_source_links.py --strict（88 条源码引用、29 个源文件，0 error 0 warning）实际执行通过；一个 lua 围栏语法完整。

返回 [[reference/prefabs/weapons-combat-index|武器与战斗]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-08 build 26474873：本页随版本迁移升版（机械沿用；引用区间未受本次源码行号位移影响），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-07）。
