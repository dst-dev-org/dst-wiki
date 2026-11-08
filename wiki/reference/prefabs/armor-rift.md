---
title: "裂隙三护甲族（armordreadstone、armor_voidcloth、armor_lunarplant）"
created: "2026-11-07"
updated: "2026-11-08"
type: prefab
summary: "裂隙版本三件 BODY 槽护甲与荆棘茄甲：planardefense＋damagetyperesist＋setbonus 新系装配骨架之上，绝望石理智驱动自愈、虚空长袍负光环免疫与破损双态、亮茄甲位面反伤与 Wormwood 根须/荆棘协作，覆盖配方（绝望石蓝图/双熔炉）与暗影/月亮阵营交互，供模组参照原版做带阵营与自愈机制的护甲。"
aliases: ["armordreadstone", "绝望石盔甲", "armor_voidcloth", "虚空长袍", "armor_lunarplant", "亮茄盔甲", "armor_lunarplant_husk", "荆棘茄甲", "恐惧石甲", "暗影碎布甲", "brightshade armor", "voidcloth armor", "dreadstone armor", "裂隙护甲", "月亮裂缝护甲"]
tags: [prefab, item, equipment]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/armor_dreadstone.lua", "prefabs/armor_voidcloth.lua", "prefabs/armor_lunarplant.lua", "prefabs/hats.lua", "prefabs/daywalker.lua", "prefabs/forge_repair_kits.lua", "prefabs/wormwood_vined_debuff.lua", "prefabs/bramblefx.lua", "prefabs/skilltree_wortox.lua", "prefabs/skilltree_wormwood.lua", "prefabs/waxwell.lua", "prefabs/wanda.lua", "prefabs/gestalt.lua", "prefabs/gestalt_guard.lua", "prefabs/gestalt_guard_evolved.lua", "prefabs/shadow_forge.lua", "prefabs/lunar_forge.lua", "prefabs/tree_rocks.lua", "prefabs/armor_skeleton.lua", "components/armor.lua", "components/setbonus.lua", "components/damagetyperesist.lua", "components/planardefense.lua", "components/damagereflect.lua", "components/forgerepairable.lua", "components/forgerepair.lua", "components/resistance.lua", "components/sanity.lua", "components/inventory.lua", "components/combat.lua", "components/equippable.lua", "components/shadowlevel.lua", "standardcomponents.lua", "componentutil.lua", "stategraphs/SGwagdrone_rolling.lua", "stategraphs/SGshadowwaxwell.lua", "util/sourcemodifierlist.lua", "tuning.lua", "recipes.lua", "constants.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
size_class: complex
---

# 裂隙三护甲族（armordreadstone、armor_voidcloth、armor_lunarplant）

## 快速判断

- 职责与适用：三份独立 prefab 文件共 **4 个 BODY 槽注册**（其中 `armor_lunarplant_husk`＝荆棘茄甲是独立注册的 Wormwood 专属变身态）——共享「armor＋planardefense＋damagetyperesist＋setbonus」新系护甲骨架，各自带专属机制：绝望石盔甲理智驱动自愈、虚空长袍负理智光环免疫＋破损双态、亮茄盔甲位面反伤＋Wormwood 根须协作。传统 armor 护甲（草甲/铥矿甲等 10 注册）归 [[reference/prefabs/armor-classic|传统身体护甲族]]；三顶同族帽的帽子侧机制归 [[reference/prefabs/hats-armor|防护与战斗帽族]]。
- 优先使用：查某件的耐久/吸收/位面防御 → [注册与构造](#注册与构造)；查自愈速率/破损修复/反伤数值 → [典型接入](#典型接入)对应小节；查标签（`shadow_item`/`hardarmor`/`gestaltprotection`）实际效果 → [消费端标签](#消费端标签)；自己写新系护甲按组件骨架进入。
- 重要副作用：armor 耐久按 `Inventory:ApplyDamage` 分摊到该件的吸收量扣减（非原始来伤）并推 `armordamaged`（推导见 [[reference/components/armor|armor 组件]]）；虚空长袍装备期免疫负理智光环但会惹月暴守卫主动攻击；亮茄甲对攻击者反射位面伤害；荆棘茄甲仅 Wormwood 可穿。
- 修改边界：本页只写护甲本体与直接触发链；装备跟随光效（`armor_voidcloth_fx`、`armor_lunarplant_glow_fx`、`armor_lunarplant_husk_glow_fx`）归 [[reference/prefabs/armor_lunarplant_glow_fx|装备与帽饰跟随光效]]；帽子侧套装增伤/防雷归 hats-armor 页；虚空风帽武器叠层机制属帽子侧，本页不展开。数值区分配置值与推导值：联机耐久乘数 `multiplayer_armor_durability_modifier = 0.7`、`wilson_health = 150`（[tuning.lua#L33-L38](../../../raw/current/scripts/tuning.lua#L33-L38)）。

## 注册与构造

三文件返回关系：[armor_dreadstone.lua#L151](../../../raw/current/scripts/prefabs/armor_dreadstone.lua#L151)、[armor_voidcloth.lua#L279-L280](../../../raw/current/scripts/prefabs/armor_voidcloth.lua#L279-L280)、[armor_lunarplant.lua#L436-L440](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L436-L440)。注意 `armor_dreadstone.lua` 的注册名**无连字符**（`armordreadstone`），配方与蓝图键同名；`armor_lunarplant` 与 `armor_lunarplant_husk` 共用 `commonfn(build, common_postinit, master_postinit)` 工厂（[commonfn](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L170-L237)），破损/修复双态共用 `OnBroken`/`OnRepaired`（[voidcloth](../../../raw/current/scripts/prefabs/armor_voidcloth.lua#L67-L85)、[lunarplant](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L134-L152)）。

| 注册名（官方名） | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `armordreadstone`（绝望石盔甲） | 独立 `fn()`：骨架＋标签 → `SetPristine` → 主模拟端挂组件 | 标签 `dreadstone`/`hardarmor`/`shadow_item`/`shadowlevel`（[L101-L106](../../../raw/current/scripts/prefabs/armor_dreadstone.lua#L101-L106)）；armor `InitCondition(840, 0.9)`（[L124](../../../raw/current/scripts/prefabs/armor_dreadstone.lua#L124)）＋`ontakedamage`（[L125](../../../raw/current/scripts/prefabs/armor_dreadstone.lua#L125)）启动自愈；planar 5；`damagetyperesist` 暗影 0.9；`shadowlevel` 2；`setbonus` DREADSTONE；`dapperfn`（[L132](../../../raw/current/scripts/prefabs/armor_dreadstone.lua#L132)）＋`is_magic_dapperness = true`（[L133](../../../raw/current/scripts/prefabs/armor_dreadstone.lua#L133)）；**无 forgerepairable**——自愈即修复通道 |
| `armor_voidcloth`（虚空长袍） | 独立 `fn()`：`SetupEquippable` 可拆装（破损态 `RemoveComponent("equippable")`、修复重建，[L57-L62](../../../raw/current/scripts/prefabs/armor_voidcloth.lua#L57-L62)） | 标签 `cloth`/`shadow_item`/`show_broken_ui`/`shadowlevel`（[L100-L105](../../../raw/current/scripts/prefabs/armor_voidcloth.lua#L100-L105)）；armor `InitCondition(830, 0.8)`（[L122-L123](../../../raw/current/scripts/prefabs/armor_voidcloth.lua#L122-L123)）＋planar 10；暗影抗 0.9；`shadowlevel` 3；套装 VOIDCLOTH 启用加暗影抗（[L10-L16](../../../raw/current/scripts/prefabs/armor_voidcloth.lua#L10-L16)）；`MakeForgeRepairable(FORGEMATERIALS.VOIDCLOTH)`（[L141](../../../raw/current/scripts/prefabs/armor_voidcloth.lua#L141)）；装备期免疫负理智光环（[L34-L36](../../../raw/current/scripts/prefabs/armor_voidcloth.lua#L34-L36)） |
| `armor_lunarplant`（亮茄盔甲） | `commonfn("armor_lunarplant", nil, master_postinit)`（[L245-L247](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L245-L247)）：master 段装 `damagereflect` 反伤并接 Wortox 月抗 | 标签 `lunarplant`/`gestaltprotection`/`show_broken_ui`（[L179-L181](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L179-L181)）；armor `InitCondition(830, 0.8)`（[L208-L209](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L208-L209)）＋planar 10；月亮抗 0.9；套装 LUNARPLANT 启用加月亮抗（[L49-L55](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L49-L55)）；`damagereflect` 位面反射（[L214-L216](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L214-L216)）；`MakeForgeRepairable(FORGEMATERIALS.LUNARPLANT)`（[L228](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L228)） |
| `armor_lunarplant_husk`（荆棘茄甲） | `commonfn("armor_lunarplant_husk", husk_common_postinit, husk_master_postinit)`（[L303-L305](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L303-L305)）：换 build＋加 `bramble_resistant` 标签＋`restrictedtag = "plantkin"`（[L291-L301](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L291-L301)） | 仅 Wormwood 可装备（restrictedtag）；被击/格挡触发荆棘尖刺环（`bramblefx_armor_upgrade`）；Wormwood 技能 `wormwood_armor_bramble` 下主动攻击 3 连击也触发；armor/planar/月亮抗/套装与本体同源（commonfn 共享）；配方由亮茄盔甲＋荆棘外壳合成 |

FX 注册 3 个（`armor_voidcloth_fx`、`armor_lunarplant_glow_fx`、`armor_lunarplant_husk_glow_fx`，[voidcloth fxfn](../../../raw/current/scripts/prefabs/armor_voidcloth.lua#L250-L277)、[MakeGlow](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L402-L434)）随本体 onequip 生成、unequip 移除，工厂与端边界归 [[reference/prefabs/armor_lunarplant_glow_fx|装备与帽饰跟随光效]]。

## 关联机制导航

### 组件

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| `组件：armor` | `InitCondition(耐久, 吸收)`；armordreadstone 另设 `ontakedamage` 回调驱动自愈；`Repair`（自愈）与 `SetKeepOnFinished`/`SetOnFinished`（破损链经 MakeForgeRepairable） | 三件全主模拟端挂载；破损双态在耐久归零时触发 | [[reference/components/armor]]；[InitCondition](../../../raw/current/scripts/components/armor.lua#L39-L43)、[TakeDamage](../../../raw/current/scripts/components/armor.lua#L156-L164)、[Repair](../../../raw/current/scripts/components/armor.lua#L166-L171) |
| `组件：planardefense` | `SetBaseDefense` 固定位面减伤（绝望石 5、虚空/亮茄 10） | 三件全挂；被位面伤害（spdamage）结算消费 | [[reference/components/planardefense]]；[GetDefense](../../../raw/current/scripts/components/planardefense.lua#L18-L20) |
| `组件：damagetyperesist` | `AddResist("shadow_aligned"/"lunar_aligned", inst, 0.9)` 基础抗性；套装启用经回调以 key `"setbonus"` 叠加 `sqrt(0.75)/0.9 ≈ 0.962`（[tuning#L1898](../../../raw/current/scripts/tuning.lua#L1898)、[L1910](../../../raw/current/scripts/tuning.lua#L1910)） | 同 src 不同 key 两笔相乘（[SetModifier key 协议](../../../raw/current/scripts/util/sourcemodifierlist.lua#L60-L80)）：套装期单件抗变 0.9×0.962≈0.866，头＋身两件 → 0.866²=0.75（承伤倍率 0.75：该阵营伤害按 75% 结算，即**减免 25%**）；消费端为玩家 `Inventory:ApplyDamage` | [[reference/components/damagetyperesist]]；[AddResist](../../../raw/current/scripts/components/damagetyperesist.lua#L11-L18)、[GetResist](../../../raw/current/scripts/components/damagetyperesist.lua#L37-L47)、[ApplyDamage 乘区](../../../raw/current/scripts/components/inventory.lua#L424-L425) |
| `组件：setbonus` | `SetSetName(EQUIPMENTSETNAMES.DREADSTONE/VOIDCLOTH/LUNARPLANT)`＋启用/禁用回调 | 启用判定要求 HEAD+BODY 两件同名（[同名判定](../../../raw/current/scripts/components/setbonus.lua#L52-L64)）；单件不生效（hats-armor 页已核，互链复引）；DREADSTONE 套装回调走自愈倍率 1.5×与 dapperness 减半 | [[reference/components/setbonus]]；[UpdateSetBonus](../../../raw/current/scripts/components/setbonus.lua#L67-L76)、[EQUIPMENTSETNAMES](../../../raw/current/scripts/constants.lua#L2205-L2211) |
| `组件：equippable` | BODY 槽；armordreadstone `dapperfn`＋`is_magic_dapperness`；husk `restrictedtag = "plantkin"`；`GetDapperness` 中 `dapperfn` 返回值**整体替换**静态 `dapperness`（不是叠加） | 三件全主模拟端；husk 限制仅对玩家生效（IsRestricted） | [[reference/components/equippable]]；[GetDapperness](../../../raw/current/scripts/components/equippable.lua#L196-L205)、[IsRestricted](../../../raw/current/scripts/components/equippable.lua#L147-L158) |
| `组件：damagereflect` | `SetReflectDamageFn` 返回 0 近战＋spdamage planar（对 `shadow_aligned` 攻击者 20，否则 10）；`onreflectdamage` 事件生成 `hitsparks_reflect_fx` | 仅 armor_lunarplant（commonfn，husk 继承）；反射经 combat 的装备槽遍历生效 | [[reference/components/damagereflect]]；[ReflectDamageFn](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L154-L161)、[combat 反射收集](../../../raw/current/scripts/components/combat.lua#L966-L1003)、[事件推送](../../../raw/current/scripts/components/combat.lua#L1238) |
| `组件：shadowlevel` | `SetDefaultLevel`：armordreadstone 2（T2）、armor_voidcloth 3（T3）；亮茄系无此组件（月亮阵营） | 等级进麦斯威尔影随从伤害加成（`SHADOWWAXWELL_PROTECTOR_DAMAGE + level × 加成`） | [[reference/components/shadowlevel]]；[SGshadowwaxwell 消费](../../../raw/current/scripts/stategraphs/SGshadowwaxwell.lua#L147-L160)、[T2/T3 常量](../../../raw/current/scripts/tuning.lua#L2870-L2880) |
| `组件：sanity`（持有者侧） | armor_voidcloth onequip 对持有者 `neg_aura_modifiers:SetModifier(inst, 0)` → 负理智光环 ×0；onunequip `RemoveModifier` | 仅 armor_voidcloth；只乘负光环，不影响正光环与其他理智项 | [[reference/components/sanity]]；[GetAuraMultipliers](../../../raw/current/scripts/components/sanity.lua#L484-L486)、[负光环应用](../../../raw/current/scripts/components/sanity.lua#L570-L571)、[默认乘法](../../../raw/current/scripts/util/sourcemodifierlist.lua#L21)、[multiply 实现](../../../raw/current/scripts/util/sourcemodifierlist.lua#L25-L27) |
| `组件：forgerepairable`／`forgerepair` | 修补套件（`voidcloth_kit`/`lunarplant_kit`）对受损件 `SetPercent(1)` 一次修满并消耗 1 层；套件只有 `forgerepair` 组件＋对应 material | armor_voidcloth/armor_lunarplant/husk；armordreadstone 无此链（自愈）；破损态装备先被 unequip | [[reference/components/forgerepairable]]、[[reference/components/forgerepair]]；[MakeForgeRepairable](../../../raw/current/scripts/standardcomponents.lua#L1851-L1892)、[Repair 匹配材料](../../../raw/current/scripts/components/forgerepairable.lua#L58-L73)、[OnRepair 回满](../../../raw/current/scripts/components/forgerepair.lua#L24-L49)、[kit 注册](../../../raw/current/scripts/prefabs/forge_repair_kits.lua#L56-L58) |
| `组件：resistance`（Wortox 月阵营） | armor_lunarplant master_postinit 调 `skilltree_defs` 的 `SetupLunarResists` 加 resistance（抗 `_combat`/explosive/quakedebris/lunarhaildebris/caveindebris/trapdamage/rock_tree） | 四道门禁同时成立才免伤：装备中＋持有者激活 `wortox_allegiance_lunar`＋存在 `owner.finishportalhoptask`＋`TryToPortalHop(1, false)` 成功（即至少 1 灵魂、且参与正常跳跃计数/结算）——不是常驻免伤；成功后 `Inventory:ApplyDamage` 直接返回 0 伤害（免伤也不耗耐久）。husk 的 `husk_master_postinit` 不接这条 | [SetupLunarResists](../../../raw/current/scripts/prefabs/skilltree_wortox.lua#L157-L164)、[抗性表](../../../raw/current/scripts/prefabs/skilltree_wortox.lua#L148-L155)、[ShouldResistFn](../../../raw/current/scripts/prefabs/skilltree_wortox.lua#L113-L125)、[ApplyDamage 免伤](../../../raw/current/scripts/components/inventory.lua#L416-L421)；与 armor_skeleton 共表（[同步注释](../../../raw/current/scripts/prefabs/skilltree_wortox.lua#L148)） |

### 事件与数据

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| `事件："blocked"`（持有者） | armordreadstone 播受击音；armor_lunarplant/husk 播仙人掌音并作为荆棘/根须触发源之一 | 完全格挡时由 combat 推送（含武器/stimuli data） | [blocked 推送](../../../raw/current/scripts/components/combat.lua#L730)、[dreadstone OnBlocked](../../../raw/current/scripts/prefabs/armor_dreadstone.lua#L6-L8)、[lunarplant OnBlocked](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L43-L47) |
| `事件："attacked"/"blocked" → OnHit_Vines` | armor_lunarplant 对 plantkin 持有者监听两事件：攻击者挂 `wormwood_vined_debuff`（rooted 定身 6 秒＋10 伤害），需持有者激活技能 `wormwood_allegiance_lunar_plant_gear_1`；排除自己攻击/重定向/无 locomotor/已死 | 仅 Wormwood（`plantkin`）；技能定义见 [skilltree_wormwood#L174](../../../raw/current/scripts/prefabs/skilltree_wormwood.lua#L174) | [OnHit_Vines](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L26-L41)、[debuff 定身](../../../raw/current/scripts/prefabs/wormwood_vined_debuff.lua#L12-L43)、[时长/伤害调值](../../../raw/current/scripts/tuning.lua#L7846-L7847) |
| `事件："onattackother" → OnAttackOther` | husk 对 plantkin 监听持有者攻击：技能 `wormwood_armor_bramble` 下累计 3 击触发 `DoThorns`；本体 armor_lunarplant 无此链 | 仅 armor_lunarplant_husk；技能定义见 [skilltree_wormwood#L373](../../../raw/current/scripts/prefabs/skilltree_wormwood.lua#L373) | [OnAttackOther](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L270-L283)、[DoThorns](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L255-L268)、[命中数调值](../../../raw/current/scripts/tuning.lua#L7849) |
| `数据：armor 命中音效标签` | `lunarplant`/`dreadstone`/`cloth` 标签被 `GetArmorImpactSound` 选中对应受击音 | 标签在 three 文件构造段写入 | [标签选音](../../../raw/current/scripts/componentutil.lua#L1624-L1644) |
| `数据：TUNING 调值` | 全部耐久/吸收/抗性/反射/自愈常量集中在 tuning 的裂隙三段 | 推导值见[注册与构造](#注册与构造)与[典型接入](#典型接入) | [dreadstone 段](../../../raw/current/scripts/tuning.lua#L1877-L1888)、[lunarplant 段](../../../raw/current/scripts/tuning.lua#L1891-L1900)、[voidcloth 段](../../../raw/current/scripts/tuning.lua#L1903-L1914) |

### 消费端标签

| 标签 | 写入者 | 消费端与效果 | 证据 |
|---|---|---|---|
| `shadow_item` | armordreadstone、armor_voidcloth | 月暴守卫 `GetLevelForTarget` 对装备 shadow_item 的目标直接判 3 级＝主动攻击（无视理智）；麦斯威尔负 dapperness ×0（`WAXWELL_SHADOW_ITEM_RESISTANCE = 0`）；旺达按年龄 ×1/0.33/0 | [gestalt_guard](../../../raw/current/scripts/prefabs/gestalt_guard.lua#L62-L64)、[evolved](../../../raw/current/scripts/prefabs/gestalt_guard_evolved.lua#L115-L117)、[waxwell](../../../raw/current/scripts/prefabs/waxwell.lua#L179-L184)、[wanda](../../../raw/current/scripts/prefabs/wanda.lua#L202-L208)、[调值](../../../raw/current/scripts/tuning.lua#L2853) |
| `hardarmor` | armordreadstone | Wagdrone 滚动冲撞走减免分支；滚石（tree_rock）对戴 hardarmor 帽者反弹（本标签在身体甲的该消费未核） | [SGwagdrone_rolling#L324](../../../raw/current/scripts/stategraphs/SGwagdrone_rolling.lua#L324)、[tree_rocks 判定](../../../raw/current/scripts/prefabs/tree_rocks.lua#L505-L507) |
| `gestaltprotection` | armor_lunarplant（及 husk） | `IsEntityGestaltProtected` → 月暴 Gestalt 不追踪/守卫不攻击不监视 | [判定函数](../../../raw/current/scripts/componentutil.lua#L1996-L2000)、[gestalt Retarget](../../../raw/current/scripts/prefabs/gestalt.lua#L79-L81)、[守卫双检查](../../../raw/current/scripts/prefabs/gestalt_guard.lua#L101-L112) |
| `bramble_resistant` | armor_lunarplant_husk | bramblefx 尖刺环跳过 `EquipHasTag("bramble_resistant")` 实体（husk 穿戴者免疫其他荆棘反伤） | [bramblefx 过滤](../../../raw/current/scripts/prefabs/bramblefx.lua#L20-L21)、[写入](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L292) |
| `broken`／`show_broken_ui` | armor_voidcloth、armor_lunarplant（及 husk）破损态 | broken 标签随耐久归零挂、修复摘；`show_broken_ui` 让物品栏显示破损耐久条 | [OnBroken](../../../raw/current/scripts/prefabs/armor_voidcloth.lua#L67-L75)、[itemtile#L101](../../../raw/current/scripts/widgets/itemtile.lua#L101)、[itemtile#L601](../../../raw/current/scripts/widgets/itemtile.lua#L601) |
| `shadowlevel`（标签） | armordreadstone、armor_voidcloth | 与 `shadowmagic` 组合在物品描述追加影子等级条目；客户端 pristine 态即加 | [描述消费](../../../raw/current/scripts/stringutil.lua#L343-L344)、[写入](../../../raw/current/scripts/prefabs/armor_dreadstone.lua#L106) |

### 关联 Prefab 与资源

| 类型／对象 | 关系 | 触发或加载条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| `关联 Prefab：dreadstonehat/lunarplanthat/voidclothhat` | 同套装帽（帽子侧机制、虚空风帽武器叠层归 hats-armor 页）；GetSetBonusEquip 直接按 `dreadstonehat` prefab 判套装 | 头＋身同名 setbonus 才启用 | [[reference/prefabs/hats-armor]]；[GetSetBonusEquip](../../../raw/current/scripts/prefabs/armor_dreadstone.lua#L10-L13) |
| `关联 Prefab：voidcloth_kit/lunarplant_kit`（虚空/亮茄修补套件） | 修复材料：`forgerepair` 组件带对应 material，一次修满耗 1 层 | 对受损（含破损态）本体使用 REPAIR 动作 | [kit 工厂](../../../raw/current/scripts/prefabs/forge_repair_kits.lua#L5-L58)、[REPAIR 分支](../../../raw/current/scripts/actions.lua#L1000-L1003) |
| `关联 Prefab：armor_bramble` | husk 配方材料（亮茄盔甲＋荆棘外壳合成）；荆棘反伤原型（armor_bramble 自身机制归 FX 页） | Wormwood 技能 `wormwood_allegiance_lunar_plant_gear_1` 门禁 | [[reference/prefabs/bramblefx_armor]]；[配方](../../../raw/current/scripts/recipes.lua#L352) |
| `关联 Prefab：armordreadstone_blueprint/dreadstonehat_blueprint` | 绝望石装备蓝图：Rift 4 Boss daywalker 掉落表各 100%，蓝图学习后解锁 TECH.LOST 配方 | daywalker 战利品 | [掉落表](../../../raw/current/scripts/prefabs/daywalker.lua#L36-L43)、[prefabs 依赖](../../../raw/current/scripts/prefabs/daywalker.lua#L18-L22) |
| `关联 Prefab：shadow_forge/lunar_forge` | 双熔炉制作站：station_tag 与 `armor_voidcloth`/`armor_lunarplant` 配方绑定 | 建站后可造，`nounlock=true` 不入常规解锁 | [shadow_forge 标签](../../../raw/current/scripts/prefabs/shadow_forge.lua#L124-L128)、[lunar_forge 标签](../../../raw/current/scripts/prefabs/lunar_forge.lua#L128-L132) |
| `关联 Prefab：wormwood_vined_debuff/wortox_resist_fx/hitsparks_reflect_fx/bramblefx_armor_upgrade` | 护甲触发链生成物：根须 debuff、Wortox 免伤特效、反伤火花、husk 荆棘尖刺环 | 对应事件触发；FX 行为归各自页 | [生成点](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L39)、[L166](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L163-L168)、[L263](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L255-L268)、[[reference/prefabs/bramblefx_armor]] |
| `资源：anim/armor_dreadstone.zip` 等 3 包 | 各甲 bank/build 同名动画；husk 另用 `armor_lunarplant_husk.zip` | 构造声明；破损态播 `broken` 动画 | [assets](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L6-L10)、[SWAP_DATA_BROKEN](../../../raw/current/scripts/prefabs/armor_voidcloth.lua#L64-L65) |

## 典型接入

### 绝望石式理智驱动自愈装备

- 前提与选择：给自定义 BODY 甲接「越疯回越快」通道时，复用 `armor.ontakedamage` 回调＋`DoPeriodicTask` 而非每帧改 condition；伤害事件只负责起任务，实际恢复放在周期任务里并判断理智模式。
- 生效与撤销：受击起任务（[OnTakeDamage](../../../raw/current/scripts/prefabs/armor_dreadstone.lua#L70-L77)）；`DoRegen` 每秒执行，仅在持有者 `IsInsanityMode()`（非月暴模式，[sanity#L163-L165](../../../raw/current/scripts/components/sanity.lua#L163-L165)）时 `Repair(maxcondition × rate × setbonus)`，回满自动取消任务（[DoRegen](../../../raw/current/scripts/prefabs/armor_dreadstone.lua#L15-L25)）；卸甲停任务（[onunequip](../../../raw/current/scripts/prefabs/armor_dreadstone.lua#L58-L68)）。速率 `1 / Lerp(1/MAXRATE, 1/MINRATE, 理智%)`：理智 0% 约 0.00111/秒（约 900 秒修满 840 点）、100% 约 0.000667/秒（约 1500 秒），DREADSTONE 套装 ×1.5（约 600–1000 秒）；`CRAZINESS_MED = -100/day_time ≈ -0.333 理智/秒`（[tuning#L2225](../../../raw/current/scripts/tuning.lua#L2225)）在 regen 进行中经 dapperfn 持续扣理智。
- 原版依据：dreadstonehat 同型自愈在 [hats.lua#L3705-L3812](../../../raw/current/scripts/prefabs/hats.lua#L3705-L3812)；套装回调不额外加自愈通道，只把倍率乘进同一公式。

```lua
-- modmain.lua：自定义护甲复刻绝望石理智自愈（数值可替换）
-- mod 环境（mods.lua CreateEnvironment）无 __index=_G：
-- TheWorld/EQUIPSLOTS/SpawnPrefab 等游戏全局一律 GLOBAL. 前缀；TUNING 由环境直接提供。
AddPrefabPostInit("mymetalarmor", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end

    local function stopregen(inst_)
        if inst_.regentask ~= nil then
            inst_.regentask:Cancel()
            inst_.regentask = nil
        end
    end

    local function startregen(inst_, owner)
        if inst_.regentask ~= nil or owner == nil or owner.components.sanity == nil then
            return
        end
        inst_.regentask = inst_:DoPeriodicTask(1, function(i)
            local o = i.components.inventoryitem.owner
            -- 关键：放回背包/箱子后 inventoryitem.owner 仍在，必须再判"已装备"
            if i.components.equippable == nil or not i.components.equippable:IsEquipped() then
                stopregen(i)
                return
            end
            if o ~= nil and o.components.sanity ~= nil and o.components.sanity:IsInsanityMode() then
                i.components.armor:Repair(i.components.armor.maxcondition * 0.001)
            end
            if not i.components.armor:IsDamaged() then
                stopregen(i)   -- 装满即停
            end
        end)
    end

    -- 原版 ontakedamage 只是"起任务"的入口
    inst.components.armor.ontakedamage = function(inst_)
        startregen(inst_, inst_.components.inventoryitem.owner)
    end

    -- 原版 onequip 也会启动（穿上已受损的件就回），unequip 成对取消
    local onequip_orig = inst.components.equippable.onequipfn
    inst.components.equippable:SetOnEquip(function(inst_, owner, from_ground)
        if onequip_orig ~= nil then onequip_orig(inst_, owner, from_ground) end
        if owner ~= nil and owner.components.sanity ~= nil and inst_.components.armor:IsDamaged() then
            startregen(inst_, owner)
        end
    end)
    local onunequip_orig = inst.components.equippable.onunequipfn
    inst.components.equippable:SetOnUnequip(function(inst_, owner)
        if onunequip_orig ~= nil then onunequip_orig(inst_, owner) end
        stopregen(inst_)
    end)
end)
```

- 调整与验证：替换点为 prefab 名与速率常量；`ontakedamage` 签名是 `(inst, amount)`，收到的是 `Armor:TakeDamage` 已乘损耗倍率、**已由 `Inventory:ApplyDamage` 分摊到该件**的损耗量（不是原始来伤，结算关系见 [[reference/components/armor]]）。装备态判定不能省：只读 `inventoryitem.owner` 的话，甲回到人物背包后 owner 仍在、周期任务会继续修；留在别的容器时任务也不会因失去合格持有者自动取消。示例只做语法检查，未在游戏内执行。

### 破损双态＋修补套件（forge repairable）

- 前提与选择：需要「耐久归零不消失、改破损态、用专属材料修满」的装备时，用 `MakeForgeRepairable(inst, FORGEMATERIALS.X, OnBroken, OnRepaired)`（[standardcomponents#L1851-L1892](../../../raw/current/scripts/standardcomponents.lua#L1851-L1892)）：它自动 `SetKeepOnFinished(true)`＋耐久归零 unequip 并调 OnBroken。
- 生效与撤销：OnBroken `RemoveComponent("equippable")`、播 `broken` 动画、挂 `broken` 标签与 `BROKEN_FORGEDITEM` 名称覆写（[voidcloth OnBroken](../../../raw/current/scripts/prefabs/armor_voidcloth.lua#L67-L75)，检查名"它破损了，但是我可以把它修好。"）；OnRepaired 反向重建（[L77-L85](../../../raw/current/scripts/prefabs/armor_voidcloth.lua#L77-L85)）。修复动作要求材料带 `forgerepair` 组件且 material 一致——即 `voidcloth_kit`/`lunarplant_kit`，不是针线包；`OnRepair` 对 armor 直接 `SetPercent(1)` 一次修满并消耗套件 1 层（[forgerepair#L24-L49](../../../raw/current/scripts/components/forgerepair.lua#L24-L49)）。套件配方：1 纯粹辉煌＋1 亮茄外壳、1 纯粹恐惧＋1 暗影碎布（[recipes#L855](../../../raw/current/scripts/recipes.lua#L855)、[L871](../../../raw/current/scripts/recipes.lua#L871)）。
- 原版依据：husk 与本体共用同一 `OnBroken`/`OnRepaired`，`SetupEquippable` 重建时还原 restrictedtag（[lunarplant#L120-L129](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L120-L129)）——自定义 restrictedtag 装备必须照抄这一还原：`SetupEquippable` 只在 `inst._equippable_restrictedtag ~= nil` 时写回 `restrictedtag`，自写修复漏了这行，修好后**限制就消失、谁都能穿**（不是"玩家装不上"）。

### 亮茄甲的 Wormwood 技能门禁触发链

- 前提与选择：onequip 时**只查 `plantkin`** 决定要不要挂监听，技能 `IsActivated(...)` 留到事件触发时再判——别写成"装备时查技能、不满足就不监听"，那样中途点亮技能不会生效；监听挂 owner、卸甲解绑，不要在 onactivateskill 事件里动态挂甲。
- 生效与撤销：本体 armor_lunarplant 对 plantkin 挂 `attacked`/`blocked` 监听（[onequip#L71-L80](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L71-L80)）→ `OnHit_Vines` 检查技能 `wormwood_allegiance_lunar_plant_gear_1` 后给攻击者挂定身 debuff；husk 则是受击/格挡直接 `DoThorns`（0.3 秒冷却防连锁）＋技能 `wormwood_armor_bramble` 下累计 3 次 `onattackother` 触发（未满 3 不清零，门禁失败或释放尖刺才清零）（[DoThorns/OnAttackOther](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L255-L283)）；两链共用 `ShouldProcOnAttackedOrBlocked` 排除自身装备伤害与重定向（[componentutil#L114-L122](../../../raw/current/scripts/componentutil.lua#L114-L122)）。
- 原版依据：荆棘环实体 `bramblefx_armor_upgrade` 的伤害 `ARMORBRAMBLE_DMG = wilson_attack/1.5 ≈ 22.7`＋位面 5（[bramblefx#L124](../../../raw/current/scripts/prefabs/bramblefx.lua#L124)、[调值](../../../raw/current/scripts/tuning.lua#L4562-L4566)）；fx 实体行为归荆棘尖刺反伤页（`[[reference/prefabs/bramblefx_armor]]`）。

## 常见陷阱与限制

- **误区：注册名是 `armor_dreadstone`。** 文件叫 `armor_dreadstone.lua`，但 `Prefab` 注册名无连字符（[armor_dreadstone.lua#L151](../../../raw/current/scripts/prefabs/armor_dreadstone.lua#L151)）；`AddPrefabPostInit("armor_dreadstone", ...)` 会静默不生效，`recipes.lua` 的键也是 `armordreadstone`（[L502](../../../raw/current/scripts/recipes.lua#L502)）。另两件带连字符，命名不统一是原版事实。
- **`Armor:TakeDamage` 自己不做分摊，拿到什么扣什么。** 它只做 `condition - damage_amount`（先乘 `conditionlossmultipliers`）并回调 `ontakedamage`；分摊发生在调用方 `Inventory:ApplyDamage`（总吸收量按各件吸收率占比分配，另加弱点与特殊伤害）。所以复用自愈/破损链时不要在 `TakeDamage` 里二次扣减，也不要以为它会重新乘吸收率。[TakeDamage](../../../raw/current/scripts/components/armor.lua#L156-L164)、[耐久分摊](../../../raw/current/scripts/components/inventory.lua#L408-L500)
- **套装单件不生效。** 头＋身两件同名 setbonus 才启用回调（[同名判定](../../../raw/current/scripts/components/setbonus.lua#L52-L64)）；DREADSTONE 套装只改两处读取：`CalcDapperness` 经 `GetSetBonusEquip` 检测帽子（含借用帽子的 regentask 判断）并把 CRAZINESS_MED 减半、`DoRegen` 的 `IsEnabled` 把自愈倍率乘 1.5；虚空/亮茄的阵营抗性叠加也只在套装启用后追加（hats-armor 页互链复引）。
- **自愈依赖普通疯狂模式。** `IsInsanityMode()` 在月暴（Lunacy）模式下为 false，自愈与 CRAZINESS_MED 掉理智同时停；月暴世界穿戴绝望石甲没有自我修复。IsInsanityMode 判定见 sanity 组件页。
- **shadow_item 是双刃标签。** 同一件虚空长袍既免疫负理智光环又让月暴守卫无条件主动攻击（等级 3 判定优先于理智检查）；做"暗影阵营护甲不惹月暴"的自定义装备不能只抄负光环一行。见[消费端标签](#消费端标签)。
- **修补套件不是 sewing kit。** `forgerepairable` 匹配的是材料的 `forgerepair` 组件，针线包（repairer 组件）走另一条 REPAIR 分支，对这三件无效；且修复无比例、直接回满，不需要写数值换算。[REPAIR 分支](../../../raw/current/scripts/actions.lua#L1000-L1003)
- **dapperfn 是整体替换。** `GetDapperness` 里 `dapperness = self.dapperfn(inst, owner)`（[equippable#L203-L205](../../../raw/current/scripts/components/equippable.lua#L196-L205)）覆盖静态 `dapperness` 字段；给 dreadstone 式装备补"常驻 dapperness"要走别的通道。
- **`is_magic_dapperness` 只在角色侧的 `get_equippable_dappernessfn` 里被读。** equippable.lua 自身只留注释（[L45](../../../raw/current/scripts/components/equippable.lua#L45)），实际消费者是 Walter（[walter.lua#L754-L760](../../../raw/current/scripts/prefabs/walter.lua#L754-L760)）：只有 `is_magic_dapperness` 的装备才给他算理智；同批还有 Wanda、Waxwell、WX78 模块走同一钩子。别把它当"无用的装饰字段"。
- **Wortox 免伤链是"窗口"而非"常驻"。** resistance 免伤要求"装备中＋激活 `wortox_allegiance_lunar`＋`finishportalhoptask ~= nil`＋`TryToPortalHop(1, false)` 成功"四道门禁同时成立（[ShouldResistFn](../../../raw/current/scripts/prefabs/skilltree_wortox.lua#L113-L125)、[跳跃结算](../../../raw/current/scripts/prefabs/wortox.lua#L614-L650)），所以不是永久免伤。注意它**只借跳跃的记账、不产生位移**：`TryToPortalHop` 先查灵魂数，再把这次计入 `_freesoulhop_counter`／`_soulhop_cost`；未达 `GetHopsPerSoul()` 时这次记为免费、起 `finishportalhoptask`（到期才 `ConsumeByName` 扣灵魂、清计数、摘 soulecho），否则立刻 `FinishPortalHop()` 结算。真正的位移是另一条链：`ACTIONS.BLINK` → `portal_jumpin` 状态（[BLINK](../../../raw/current/scripts/actions.lua#L3586-L3591)），`FinishPortalHop` 本身也不移动玩家。荆棘茄甲的 `husk_master_postinit` 不接这条。该组件是 master_postinit 动态加的，客户端 replica 上没有，不要在客户端读取。
- **husk 的 `_onblocked` 同时接 attacked。** 命名误导：husk 工厂把同一回调挂到 `attacked` 与 `blocked` 两事件（[onequip#L71-L74](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L71-L80)），被击和格挡都会触发尖刺环，移植时别按函数名只当格挡反伤处理。
- **modmain 全局前缀。** mod 环境（[mods.lua#L258-L317](../../../raw/current/scripts/mods.lua#L258-L317)）无 `__index=_G`，`TheWorld`/`EQUIPSLOTS`/`SpawnPrefab` 一律 `GLOBAL.` 前缀，`TUNING` 直用；见上方示例注释。

## 验证记录

- 源码审阅：2026-11-07，steam-buildid:26335394（实读 `raw/current/BUILD.txt`）。全文实读三个护甲 prefab 文件（armor_dreadstone.lua 151 行、armor_voidcloth.lua 280 行、armor_lunarplant.lua 440 行）并逐组件段锚定；组件侧实读 armor/setbonus/damagetyperesist/planardefense/damagereflect/forgerepairable/forgerepair/resistance/sanity（neg_aura 段）/inventory（ApplyDamage 全函数）/combat（blocked 与 reflect 段）/equippable（GetDapperness、IsRestricted）与 standardcomponents（MakeForgeRepairable）、componentutil（ShouldProc、IsEntityGestaltProtected、命中音效）、sourcemodifierlist（默认乘法与 key 协议）；消费端按标签/事件定向实读 gestalt/gestalt_guard/gestalt_guard_evolved/waxwell/wanda/SGwagdrone_rolling/SGshadowwaxwell/tree_rocks/forge_repair_kits/wormwood_vined_debuff/bramblefx/daywalker/shadow_forge/lunar_forge/skilltree_wortox/skilltree_wormwood（两技能节点行）。FX 内部（跟随帧/皮肤 hash）与 kit 栈上限调值未展开，归 FX 页与源码入口。
- 名称核验：`languages/chinese_s.po`（Python UTF-8 按 msgctxt 逐键读取）：ARMORDREADSTONE＝绝望石盔甲（[L338787](../../../raw/current/scripts/languages/chinese_s.po#L338787)）、ARMOR_LUNARPLANT＝亮茄盔甲（[L338842](../../../raw/current/scripts/languages/chinese_s.po#L338842)）、ARMOR_LUNARPLANT_HUSK＝荆棘茄甲（[L338847](../../../raw/current/scripts/languages/chinese_s.po#L338847)）、ARMOR_VOIDCLOTH＝虚空长袍（[L338862](../../../raw/current/scripts/languages/chinese_s.po#L338862)）；配方材料与套件：DREADSTONE＝绝望石（L342262）、HORRORFUEL＝纯粹恐惧（L343967）、PUREBRILLIANCE＝纯粹辉煌（L347387）、LUNARPLANT_HUSK＝亮茄外壳（L344872）、VOIDCLOTH＝暗影碎布（L351342）、LUNARPLANT_KIT＝亮茄修补套件（L344877）、VOIDCLOTH_KIT＝虚空修补套件（L351357）；检查名 BROKEN_FORGEDITEM＝"它破损了，但是我可以把它修好。"（L9962）；册页条目 ARMORDREADSTONE"这件铠甲会缓慢自我修复。"（L360892）、ARMORLUNARPLANT"将伤害反射给敌人，对暗影阵营的敌人伤害更大。"（L360897）、VOIDCLOTHARMOR"穿上这件可怕的长袍会让你不在乎周围的一切有多么可怕。"（L363317）。
- 配方核验：`recipes.lua` 逐行——armordreadstone L502（6 绝望石＋4 纯粹恐惧，TECH.LOST＝三系 10 级不可达科技 [constants#L1324](../../../raw/current/scripts/constants.lua#L1324)，蓝图经 daywalker 掉落）、armor_lunarplant L848（4 纯粹辉煌＋4 亮茄外壳，LUNARFORGING_TWO，station_tag lunar_forge，nounlock）、armor_voidcloth L866（4 纯粹恐惧＋4 暗影碎布，SHADOWFORGING_TWO，station_tag shadow_forge，nounlock）、armor_lunarplant_husk L352（1 亮茄盔甲＋1 荆棘外壳，TECH.NONE，builder_skill wormwood_allegiance_lunar_plant_gear_1）、修补套件 L855/L871；蓝图掉落 daywalker.lua#L39-L41（100% 两张）。
- 数值核验：840＝150×8×0.7、830＝150×6×0.7＋200（＋20×10 注释＝20 次挡 10 位面伤害，[tuning#L1891-L1892](../../../raw/current/scripts/tuning.lua#L1891-L1892)）；吸收乘数 `multiplayer_armor_absorption_modifier = 1`（0.75 注释未启用）；`CRAZINESS_MED`/`WAXWELL_SHADOW_ITEM_RESISTANCE`/`WANDA_SHADOW_RESISTANCE_*`/`WORMWOOD_ROOT_TIME`/`ROOT_DAMAGE`/`HITCOUNT`/`ARMORBRAMBLE_DMG`（wilson_attack=34）逐条锚定；自愈速率 Lerp 端点按配置式推得 0.000667–0.001111/秒（源码推导值，未实测）。
- 消费端反查：`shadow_item` 全仓 grep → 月暴守卫/麦斯威尔/旺达三消费点（ pocketswatch/SGwilson 的武器侧命中不涉及本页三件）；`hardarmor` → SGwagdrone_rolling 与 tree_rocks；`gestaltprotection` → componentutil＋gestalt/gestalt_guard；`bramble_resistant` → bramblefx 过滤表；`broken`/`show_broken_ui` → itemtile 与 entityscript；`cloth`/`dreadstone`/`lunarplant` → GetArmorImpactSound；`forgerepairable_voidcloth/_lunarplant` 动态标签 → forgerepairable/forgerepair 组件；`onreflectdamage` → combat.lua 唯一推送点。
- 分档说明：`size_class: complex`——4 注册共享工厂但双态（broken/repaired）、三套专属机制（自愈/双态修复/反伤＋技能链）、两条阵营交互（月暴敌我、Wortox 免伤）与两类获取体系（蓝图/熔炉）须整体对照，超出 standard 单页容量。
- 结构检查：2026-11-07 第六轮修订后 `validate_page.py`（含 `--check-indexes`）通过（201 行／41562 字节，complex 档上限 480 行／65536 字节，0 issue）；`validate_source_links.py --strict` 通过（144 条引用、43 个源文件，0 error 0 warning）；Lua 示例经 `parser_probe --syntax-only` 解析通过。
- 游戏运行：未运行；`validation` 保持 `source-reviewed`。

## 相关链接

- [[reference/prefabs/hats-armor|防护与战斗帽族]]（同套装三顶帽：dreadstonehat 自愈/dapperfn、lunarplanthat、voidclothhat 武器叠层，帽子侧全量机制）
- [[reference/prefabs/armor-classic|传统身体护甲族]]（armor 组件传统用法：resistance/weakness/sewing 修理通道对照）
- [[reference/prefabs/armor_lunarplant_glow_fx|装备与帽饰跟随光效]]（本族 3 个 FX 注册的工厂与端边界）
- [[reference/prefabs/bramblefx_armor|荆棘尖刺反伤]]（bramblefx_armor_upgrade 实体行为与 armor_bramble 原型）
- [[reference/components/armor|armor 组件]]、[[reference/components/setbonus|setbonus 组件]]、[[reference/components/damagetyperesist|damagetyperesist 组件]]、[[reference/components/damagereflect|damagereflect 组件]]、[[reference/components/planardefense|planardefense 组件]]、[[reference/components/shadowlevel|shadowlevel 组件]]、[[reference/components/forgerepairable|forgerepairable 组件]]（本页各机制的组件主页面）

返回 [[reference/prefabs/wearables-index|穿戴防具与饰品]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-08 build 26474873：本页随版本迁移升版（锚点迁移 2 处），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-10）。
