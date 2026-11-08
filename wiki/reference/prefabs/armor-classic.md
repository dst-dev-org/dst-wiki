---
title: "传统身体护甲族（armorgrass、armorwood 等 10 注册）"
created: "2026-11-07"
updated: "2026-11-08"
type: prefab
summary: "prefabs 目录下 10 个独立 BODY 槽装备文件的逐件工厂与机制：7 件带 armor 组件的传统护甲（草/木/大理石/铥矿/蜗壳/暗夜/嶙峋）与 3 件同槽功能装备（饥饿腰带、骨甲护盾、胡萝卜外套），覆盖耐久吸收推导值、专属机制（壳态清仇恨、骨甲充能免伤、引兔随从）、配方与掉落、官方名与消费端标签，供模组参照原版做身体装备。"
aliases: ["armorgrass", "草甲", "armorwood", "木甲", "armormarble", "大理石甲", "armorruins", "铥矿甲", "armorslurper", "饥饿腰带", "armorsnurtleshell", "蜗壳护甲", "armor_sanity", "暗夜甲", "armorskeleton", "骨头盔甲", "armor_rocky", "嶙峋甲壳", "armor_carrotlure", "胡萝卜外套", "body armor", "BODY 槽护甲", "身体护甲"]
tags: [prefab, item, equipment]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/armor_grass.lua", "prefabs/armor_wood.lua", "prefabs/armor_marble.lua", "prefabs/armor_ruins.lua", "prefabs/armor_slurper.lua", "prefabs/armor_snurtleshell.lua", "prefabs/armor_sanity.lua", "prefabs/armor_skeleton.lua", "prefabs/armor_rocky.lua", "prefabs/armor_carrotlure.lua", "tuning.lua", "recipes.lua", "languages/chinese_s.po", "components/armor.lua", "components/resistance.lua", "components/fueled.lua", "components/fuel.lua", "components/sewing.lua", "components/hunger.lua", "components/equippable.lua", "components/inventory.lua", "components/combat.lua", "components/rabbitkingmanager.lua", "componentutil.lua", "stategraphs/SGwilson.lua", "stategraphs/SGwagdrone_rolling.lua", "prefabs/slurtle.lua", "prefabs/rocky_boss.lua", "prefabs/bunnyman.lua", "prefabs/rabbitking.lua", "prefabs/skilltree_wortox.lua", "prefabs/altar_prototyper.lua", "prefabskins.lua", "widgets/itemtile.lua", "prefabs/sewingkit.lua", "actions.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: complex
---

# 传统身体护甲族（armorgrass、armorwood 等 10 注册）

## 快速判断

- 职责与适用：`raw/current/scripts/prefabs/` 下 10 个身体（BODY 槽）装备文件，**逐文件独立工厂**（各自 `Prefab(...)` 注册、无共享装配骨架）。7 件带 armor 组件：草甲、木甲、大理石甲、铥矿甲、蜗壳护甲、暗夜甲、嶙峋甲壳；3 件同槽但**无 armor 组件**的功能装备：饥饿腰带（减速饥饿）、骨头盔甲（resistance 充能护盾）、胡萝卜外套（引兔随从）。头部防护帽族归 [[reference/prefabs/hats-armor|防护与战斗帽族]]，新系 planar 甲（bramble/dragonfly/dreadstone/lunarplant/voidcloth/wagpunk/yoth）不属本页。
- 优先使用：查某件的耐久/吸收/机制/获取 → [注册与构造](#注册与构造)；查标签消费端 → [消费端标签](#消费端标签)；写装备期数值修正或受击护盾 → [典型接入](#典型接入)。
- 重要副作用：armor 耐久按「`Inventory:ApplyDamage` 分摊到该件」的吸收量扣减（先乘伤害类型抗性、取各件最高有效吸收率算总吸收、再按各自吸收率占比分配，另加弱点与特殊伤害；最后 `Armor:TakeDamage` 才乘该件 `conditionlossmultipliers`）并推 `armordamaged`（[分摊](../../../raw/current/scripts/components/inventory.lua#L408-L499)、[扣耐久](../../../raw/current/scripts/components/armor.lua#L156-L164)）；`armorslurper` 燃料耗尽即自毁且不能用物品充能；`armorsnurtleshell` 壳态满吸收＋5 秒后清 20 格仇恨；`armor_sanity` 每次受击额外扣理智；`armor_rocky` 免击退但骑乘被撞时甲额外损耗。
- 修改边界：本页只覆盖 10 件本体；全部为 BODY 槽、组件主模拟挂载；皮肤仅 7 件本体甲有（`prefabskins.lua`，见[验证记录](#验证记录)），腰/壳/诱饵无皮肤；`quagmire_*.lua` 域 grep 无本页注册引用。数值区分配置值与推导值：联机耐久乘数 `multiplayer_armor_durability_modifier = 0.7`、吸收乘数当前为 1（[tuning.lua#L33-L38](../../../raw/current/scripts/tuning.lua#L33-L38)，wilson_health＝150），下表给出推导值。

## 注册与构造

10 个文件各自返回独立 `Prefab`，注册名见各行证据（均从 `return Prefab(...)` 行实读，与文件名并非一一对应）；10 件**全部**装 inventoryitem／inspectable／equippable 且 `equipslot = EQUIPSLOTS.BODY`，onequip 覆写 `swap_body`（或 `swap_body_tall`）符号并走 `GetSkinBuild` 皮肤协议；差别只在**玩法组件**：`armorslurper` 无 armor 只有 fueled，`armorskeleton` 用 resistance＋fueled＋cooldown，`armor_carrotlure` 用 perishable＋repairable＋leader。

| 注册名（官方名） | 耐久×吸收／定位 | 附加组件与专属机制 | 获取 | 证据 |
|---|---|---|---|---|
| `armorgrass`（草甲） | 157.5×0.6 | armor＋beaver 弱点 17（海狸变身额外磨甲）；fuel LARGE_FUEL＋小可燃＋小蔓延；`grass` 标签；blocked 音效监听 | 配方草10＋枝2（TECH.NONE） | [工厂](../../../raw/current/scripts/prefabs/armor_grass.lua#L68-L70)、[tuning#L1818-L1819](../../../raw/current/scripts/tuning.lua#L1818-L1819)、[recipes#L496](../../../raw/current/scripts/recipes.lua#L496) |
| `armorwood`（木甲） | 315×0.8 | 与草甲同构（beaver 弱点、可燃、blocked 音效），`wood` 标签 | 配方木8＋绳2（SCIENCE_ONE） | [工厂](../../../raw/current/scripts/prefabs/armor_wood.lua#L68-L70)、[recipes#L497](../../../raw/current/scripts/recipes.lua#L497) |
| `armormarble`（大理石甲） | 735×0.95 | 不可燃无燃料；`walkspeedmult = 0.7` 减速；`marble`/`heavyarmor`/`hardarmor` 标签 | 配方大理石6＋绳2（SCIENCE_TWO）；锤古 altar 加权随机 0.5 | [减速](../../../raw/current/scripts/prefabs/armor_marble.lua#L64-L69)、[recipes#L501](../../../raw/current/scripts/recipes.lua#L501)、[altar#L62-L97](../../../raw/current/scripts/prefabs/altar_prototyper.lua#L62-L97) |
| `armorruins`（铥矿甲） | 1260×0.9 | `dapperness = DAPPERNESS_MED`（magic，＋0.056 理智/秒）；`shadowlevel` 组件等级 2；`ruins`/`metal`/`hardarmor`/`shadowlevel` 标签；文件头 `require "prefabs/amulet"` 仅依赖声明 | 配方铥矿6＋噩梦燃料4（ANCIENT_FOUR，nounlock） | [构造](../../../raw/current/scripts/prefabs/armor_ruins.lua#L69-L81)、[recipes#L808](../../../raw/current/scripts/recipes.lua#L808) |
| `armorslurper`（饥饿腰带） | **无 armor 组件** | fueled（USAGE）3840 秒，穿戴期消耗、耗尽 `inst.Remove` 自毁；装备期持有者饥饿燃烧 ×0.6（`burnratemodifiers`）；dapperness SMALL（magic，＋0.033/秒）；shadowlevel 2；`fur`/`ruins`/`shadowlevel` 标签 | 配方啜食者皮6＋绳2＋噩梦燃料2（MAGIC_THREE） | [装备链](../../../raw/current/scripts/prefabs/armor_slurper.lua#L6-L13)、[fueled](../../../raw/current/scripts/prefabs/armor_slurper.lua#L70-L80)、[tuning#L1830](../../../raw/current/scripts/tuning.lua#L1830)、[tuning#L1748](../../../raw/current/scripts/tuning.lua#L1748)、[recipes#L521](../../../raw/current/scripts/recipes.lua#L521) |
| `armorsnurtleshell`（蜗壳护甲） | 735×0.6（壳态改 1） | useableitem 右键入玩家 SG `shell_enter`→`shell_idle`（`shell`/`hiding` 标签）：壳态 armor 吸收改满 1，退壳还原并 `StopUsingItem`；入壳 5 秒后把 20 格内以持有者为目标的实体清仇恨；换装符号为 `swap_body_tall`；与 `slurtlehat` 双向互认发成就 `snail_armour_set`；`shell`/`hardarmor` 标签 | 蜗牛龟（snurtle）75% 掉落 | [壳态协议](../../../raw/current/scripts/prefabs/armor_snurtleshell.lua#L10-L48)、[构造](../../../raw/current/scripts/prefabs/armor_snurtleshell.lua#L107-L118)、[SGwilson#L5656-L5702](../../../raw/current/scripts/stategraphs/SGwilson.lua#L5656-L5702)、[slurtle#L28-L31](../../../raw/current/scripts/prefabs/slurtle.lua#L28-L31) |
| `armor_sanity`（暗夜甲） | 525×0.95 | `armor.ontakedamage`：每次受击（全额伤害）×0.10 扣持有者理智；dapperness CRAZINESS_SMALL（magic，−0.167 理智/秒）；shadowlevel 2；`sanity`/`shadow_item`/`shadowlevel` 标签；scrapbook ARMORSANITY | 配方噩梦燃料5＋莎纸3（MAGIC_THREE）；锤古 altar 加权随机 0.5 | [扣理智](../../../raw/current/scripts/prefabs/armor_sanity.lua#L32-L41)、[构造](../../../raw/current/scripts/prefabs/armor_sanity.lua#L80-L93)、[recipes#L500](../../../raw/current/scripts/recipes.lua#L500)、[altar#L62-L97](../../../raw/current/scripts/prefabs/altar_prototyper.lua#L62-L97) |
| `armorskeleton`（骨头盔甲） | **无 armor 组件**；resistance 充能护盾 | fueled（NIGHTMARE，accepting）720 秒＝16 次抵挡（每次 −45）；cooldown 5 秒（装备首发 1 秒）；充能后登记 `_combat`/`explosive`/`quakedebris`/`lunarhaildebris`/`caveindebris`/`trapdamage`/`rock_tree` 七类标签，命中即全额免伤＋生成 `shadow_shield1-6` FX＋1/3 秒免计费窗口；shadowlevel 3（有燃料时，`SetLevelFn` 动态）；`fossil`/`shadowlevel` 标签 | 无配方；Deconstruct 骨片10＋噩梦燃料6 | [免伤链](../../../raw/current/scripts/prefabs/armor_skeleton.lua#L57-L98)、[充能](../../../raw/current/scripts/prefabs/armor_skeleton.lua#L123-L147)、[组件段](../../../raw/current/scripts/prefabs/armor_skeleton.lua#L229-L251)、[recipes#L1675](../../../raw/current/scripts/recipes.lua#L1675) |
| `armor_rocky`（嶙峋甲壳） | 1050×0.95 | 减速 0.7；`superheavyarmor`/`heavyarmor`/`hardarmor` 标签；监听 `knockbackblocked`：骑乘被撞时 armor 额外损耗 52.5；`inventoryitem:SetSinks(true)` 落水沉没；无皮肤外唯一有 `armor_rocky_spikey` | 岩石大师（rocky_boss）常规/硝石战利品表固定含本件；Deconstruct 石8＋燧石4 | [击退链](../../../raw/current/scripts/prefabs/armor_rocky.lua#L10-L15)、[构造](../../../raw/current/scripts/prefabs/armor_rocky.lua#L54-L77)、[rocky_boss#L58-L59](../../../raw/current/scripts/prefabs/rocky_boss.lua#L58-L59)、[recipes#L1689](../../../raw/current/scripts/recipes.lua#L1689) |
| `armor_carrotlure`（胡萝卜外套） | **无 armor 组件** | 装备期 1 秒周期：引 12 格内 `regular_bunnyman` 标签兔人（无首领者）作随从，上限 10，并给现有兔随从 +3 忠诚秒——**随从挂在持有者的 `owner.components.leader` 上**（搜索半径、容量、`GetFollowersByTag` 计数与加忠诚全走持有者侧；本件虽也 `AddComponent("leader")` 却不收编），进入装备时已有兔随从计入上限、不被替换；perishable 1500 秒后腐坏成 `spoiled_food`（腐坏即停引怪）；repairable（胡萝卜）；dapperness TINY（＋0.022/秒）；`leader` 组件挂本件；`show_spoilage`/`hidesmeats` 标签 | 兔王商店交易：胡萝卜8＋绳3（RABBITKINGSHOP_TWO，nounlock，action RABBITKINGSHOP）；Deconstruct 同材料 | [引怪](../../../raw/current/scripts/prefabs/armor_carrotlure.lua#L12-L52)、[组件段](../../../raw/current/scripts/prefabs/armor_carrotlure.lua#L108-L131)、[recipes#L991](../../../raw/current/scripts/recipes.lua#L991) |

推导注：157.5＝150×1.5×0.7、315＝150×3×0.7、735＝150×7×0.7、1260＝150×12×0.7、525＝150×5×0.7、1050＝150×10×0.7、52.5＝1050/20（[tuning.lua#L1916-L1919](../../../raw/current/scripts/tuning.lua#L1916-L1919)）、beaver 17＝wilson_attack 34×0.5（[tuning.lua#L4653](../../../raw/current/scripts/tuning.lua#L4653)）。tuning 里 **L1829 `ARMORSLURPER_ABSORPTION`（0.6）与 L1831 `ARMORSLURPER`（150×4×0.7＝420）两条护甲配置未被任何 prefab 使用**，夹在中间的 L1830 `ARMORSLURPER_SLOW_HUNGER`（0.6）才是腰带在用的减饥饿系数（[tuning.lua#L1829-L1831](../../../raw/current/scripts/tuning.lua#L1829-L1831)、[消费](../../../raw/current/scripts/prefabs/armor_slurper.lua#L8-L10)）——见[陷阱](#常见陷阱与限制)。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| `组件：armor` | `InitCondition(耐久, 吸收)`；`AddWeakness("beaver", 17)`（草/木）；`ontakedamage` 回调（暗夜甲扣理智）；`TakeDamage` 只扣 `ApplyDamage` 分摊到该件的量（另乘 `conditionlossmultipliers`）并推 `armordamaged` | 7 件主模拟挂载；皮肤不影响组件 | [[reference/components/armor]]；[TakeDamage](../../../raw/current/scripts/components/armor.lua#L156-L164)、[GetBonusDamage](../../../raw/current/scripts/components/armor.lua#L144-L154) |
| `组件：resistance` | 骨头盔甲唯一使用者：`AddResistance(tag)`×7 ＋ `SetShouldResistFn`（须装备且持有者无 `forcefield` 标签）＋ `SetOnResistDamageFn`（消耗＋FX＋冷却）；玩家 `Inventory:ApplyDamage` 装备槽循环里命中即 `ResistDamage` 并返回 0 伤害 | 仅 armorskeleton；Wortox 技能树 `LunarResists` 注明与本表同步（[skilltree_wortox#L148-L156](../../../raw/current/scripts/prefabs/skilltree_wortox.lua#L148-L156)） | [[reference/components/resistance]]；[HasResistance](../../../raw/current/scripts/components/resistance.lua#L16-L24)、[ApplyDamage 检查](../../../raw/current/scripts/components/inventory.lua#L408-L420) |
| `组件：fueled` | 两种耐久体系：腰带 USAGE（`accepting` 默认 false 不可物品充能，`needssewing` 标签走缝纫包 `fueled:DoDelta`，耗尽 `SetDepletedFn(inst.Remove)`）；骨头盔甲 NIGHTMARE（`accepting = true`，噩梦燃料可充，`SetTakeFuelFn` 充能即启动冷却） | 腰带穿戴期才消耗（equip StartConsuming / unequip StopConsuming）；骨甲冷却在装备/充燃料时启动 | [[reference/components/fueled]]；[CanAcceptFuelItem](../../../raw/current/scripts/components/fueled.lua#L169-L175)、[needssewing](../../../raw/current/scripts/components/fueled.lua#L11-L15)、[腰带耗尽](../../../raw/current/scripts/prefabs/armor_slurper.lua#L73) |
| `组件：equippable` | 全家 BODY 槽；`walkspeedmult` 0.7（大理石/嶙峋）；`dapperness`＋`is_magic_dapperness`（铥矿/腰带/暗夜/诱饵四件） | 装备期间生效、卸下还原 | [[reference/components/equippable]]；[减速写入](../../../raw/current/scripts/prefabs/armor_marble.lua#L69)、[dapperness](../../../raw/current/scripts/prefabs/armor_ruins.lua#L74-L75) |
| `组件：shadowlevel` | 铥矿/腰带/暗夜 2 级、骨甲 3 级（有燃料才计，`SetLevelFn` 动态返回） | 装备即生效 | [[reference/components/shadowlevel]]；[动态等级](../../../raw/current/scripts/prefabs/armor_skeleton.lua#L181-L183) |
| `组件：useableitem` | 蜗壳右键 `SetOnUseFn` → 持有者 SG `GoToState("shell_enter")`；`SetOnStopUseFn` 取消清仇恨任务 | 依赖玩家 SG 的 `shell_*` 状态（生物无此状态，见[陷阱](#常见陷阱与限制)） | [[reference/components/useableitem]]；[shell_enter](../../../raw/current/scripts/stategraphs/SGwilson.lua#L5656-L5692)、[shell_idle](../../../raw/current/scripts/stategraphs/SGwilson.lua#L5695-L5711) |
| `组件：perishable`／`repairable`／`leader` | 胡萝卜外套：腐坏计时→`spoiled_food`＋停引怪；胡萝卜修理；`leader` 组件挂本件，但随从实际加入**持有者**的 `owner.components.leader`（容量/计数/忠诚都在持有者侧） | 腐坏独立于穿戴（持续计时）；修理材料 `MATERIALS.CARROT` | [[reference/components/perishable]]、[[reference/components/repairable]]、[[reference/components/leader]]；[carrotlure 组件段](../../../raw/current/scripts/prefabs/armor_carrotlure.lua#L108-L131) |
| `组件：hunger` | 腰带装备期 `burnratemodifiers:SetModifier(inst, 0.6)`，饥饿燃烧率 ×0.6 | 以装备实体为 modifier key，卸下成对 `RemoveModifier` | [[reference/components/hunger]]；[燃烧结算](../../../raw/current/scripts/components/hunger.lua#L154) |
| `组件：cooldown` | 骨头盔甲：`cooldown_duration = 5`；`onchargedfn = OnChargedFn` 充能完成即登记 resistances | 装备/充燃料时 `StartCharging`（首发 1 秒） | [[reference/components/cooldown]]；[组件段](../../../raw/current/scripts/prefabs/armor_skeleton.lua#L246-L247) |
| `事件/数据："blocked"` | 各甲 onequip 监听持有者 `blocked` 播放受击音效；该事件只在一次伤害被**完全归零**时才推（`combat:GetAttacked` 剩余伤害为 0 走 else 分支） | 部分吸收的普通受击不触发（见[陷阱](#常见陷阱与限制)） | [blocked 推送](../../../raw/current/scripts/components/combat.lua#L730)、[归零判定](../../../raw/current/scripts/components/combat.lua#L653) |
| `事件/数据："knockbackblocked"` | 嶙峋甲监听：骑乘状态则 armor `TakeDamage(52.5)` | 消费端在 SGwilson `knockback` 事件：`EquipHasTag("superheavyarmor")` 时免击退并推此事件（possessedbody 同） | [SGwilson#L2024-L2026](../../../raw/current/scripts/stategraphs/SGwilson.lua#L2024-L2026)、[OnKnockbackBlocked](../../../raw/current/scripts/prefabs/armor_rocky.lua#L10-L15) |
| `Replica／同步：net_event playfuelsound` | 骨头盔甲加燃料音效跨端转发：客户端监听 `armorskeleton.playfuelsound`，专服以装备者为发声体 | 仅 armorskeleton；其余 9 件无专属 replica | [net_event](../../../raw/current/scripts/prefabs/armor_skeleton.lua#L208)、[客户端监听](../../../raw/current/scripts/prefabs/armor_skeleton.lua#L215-L219) |
| `关联 Prefab：shadow_shield1-6` | 骨头盔甲抵挡时按冷却权重选 3 变体之一挂在持有者身上（`SetParent`） | 1/3 秒（`SHIELD_DURATION = 10*FRAMES`）是**免伤窗口**，不是 FX 寿命；FX 自行存活 | [[reference/prefabs/shadow_shield1]]；[PickShield](../../../raw/current/scripts/prefabs/armor_skeleton.lua#L29-L47)、[OnResistDamage](../../../raw/current/scripts/prefabs/armor_skeleton.lua#L57-L72) |
| `关联 Prefab：spoiled_food` | 胡萝卜外套 `onperishreplacement` | 腐坏时替换本体 | [[reference/components/perishable]]；[写入](../../../raw/current/scripts/prefabs/armor_carrotlure.lua#L111) |
| `关联 Prefab：slurtlehat（背壳头盔）` | 蜗壳/帽双向互认：任一侧装备时检查另一槽发成就 `snail_armour_set` | 仅成就判定，无属性联动 | [[reference/prefabs/hats-armor]]；[帽侧](../../../raw/current/scripts/prefabs/hats.lua#L1237)、[壳侧](../../../raw/current/scripts/prefabs/armor_snurtleshell.lua#L63-L68) |
| `关联 Prefab：amulet` | armor_ruins.lua 文件头 `require`，本文件未使用其符号 | 仅加载声明 | [armor_ruins#L1](../../../raw/current/scripts/prefabs/armor_ruins.lua#L1) |
| `资源：anim/armor_*.zip` | 各件 `SetBank/SetBuild`；foleysound 逐件配置（grassarmour/logarmour/marblearmour/metalarmour/fur/shellarmour/nightarmour/bone/rockyboss_armor/cactus_armor） | 皮肤 build 经 `OverrideItemSkinSymbol` 协议 | 各文件 assets 段与 onequip |

### 消费端标签

| 标签 | 写入者 | 消费端与效果 | 证据 |
|---|---|---|---|
| `grass`/`wood`/`marble`/`metal`/`sanity`/`shell` | 草甲/木甲/大理石甲/铥矿甲/暗夜甲/蜗壳 | `GetArmorImpactSound` 命中音效选型（`straw_armour_`/`wood_armour_`/`marble_armour_`/`metal_armour_`/`sanity_armour_`/`shell_armour_`）；优先级 forcefield＞sanity＞lunarplant＞dreadstone＞metal＞marble＞shell＞wood＞grass＞fur＞cloth | [componentutil#L1625-L1641](../../../raw/current/scripts/componentutil.lua#L1625-L1641)、[调用](../../../raw/current/scripts/components/combat.lua#L740-L747) |
| `heavyarmor`/`superheavyarmor`/`hardarmor` | 大理石甲/嶙峋甲（heavy＋superheavy＋hard）、蜗壳（hard） | 击退改 `knockbacklanded`（heavyarmor）；完全免击退改 `hit` 态并推 `knockbackblocked`（superheavyarmor）；Wagdrone 滚动冲撞 `EquipHasTag("hardarmor")` 走减免分支 | [SGwilson#L2024-L2026](../../../raw/current/scripts/stategraphs/SGwilson.lua#L2024-L2026)、[SGwagdrone_rolling#L324](../../../raw/current/scripts/stategraphs/SGwagdrone_rolling.lua#L324) |
| `hidesmeats` | 胡萝卜外套 | `HasMeatInInventoryFor` 装备位判定直接返回 false：兔人不因携带肉仇恨、兔王状态判 PASSIVE | [componentutil#L771-L779](../../../raw/current/scripts/componentutil.lua#L771-L779)、[bunnyman#L265-L276](../../../raw/current/scripts/prefabs/bunnyman.lua#L265-L276)、[rabbitkingmanager#L431-L437](../../../raw/current/scripts/components/rabbitkingmanager.lua#L431-L437) |
| `show_spoilage` | 胡萝卜外套 | 物品格腐败条显示 | [itemtile#L735-L739](../../../raw/current/scripts/widgets/itemtile.lua#L735-L739) |
| `regular_bunnyman` | bunnyman（兔人本体写入） | 胡萝卜外套引怪的 MUST 标签 | [bunnyman#L527](../../../raw/current/scripts/prefabs/bunnyman.lua#L539)、[armor_carrotlure#L10-L11](../../../raw/current/scripts/prefabs/armor_carrotlure.lua#L10-L11) |
| `shadow_item` | 暗夜甲 | voidclothhat 的叠层判定读取（HANDS 槽物品须同时有 `shadow_item` 与 `planardamage`；与套装无关） | [hats.lua#L4008](../../../raw/current/scripts/prefabs/hats.lua#L4008) |
| `shadowlevel` | 铥矿/腰带/暗夜/骨甲（pristine 优化标签） | itemtile 装备阴影特效（持有者有 `shadowmagic` 标签时） | [itemtile#L831](../../../raw/current/scripts/widgets/itemtile.lua#L831) |
| `fossil` | 骨头盔甲 | scripts 域消费为物体命中音效选型（stone_）与 stalker 骨刺目标排除；无护甲玩法消费 | [componentutil#L1686](../../../raw/current/scripts/componentutil.lua#L1686)、[stalkerbrain#L74](../../../raw/current/scripts/brains/stalkerbrain.lua#L74) |

## 典型接入

### 装备期修正持有者数值（饥饿腰带模式）

- 前提与选择：modmain.lua 注册自定义 BODY 装备后，在 `equippable` 回调里给持有者组件挂 `SourceModifierList` 修正（饥饿燃烧、移速等），原版 `armorslurper`（饥饿修正）与 `armormarble`（`walkspeedmult`）是两种典型。
- 生效与撤销：modifier 以**装备实体**为 key，onunequip 必须成对 `RemoveModifier`，换装/死亡移除才不残留；腰带原版还成对处理 `SetOnEquipToModel`（试穿模型时同样停止生效）。
- 原版依据：[armor_slurper onequip/onunequip](../../../raw/current/scripts/prefabs/armor_slurper.lua#L6-L22)、[hunger 燃烧结算](../../../raw/current/scripts/components/hunger.lua#L154)。

```lua
-- modmain.lua：自定义 BODY 装备的饥饿减耗＋减速（饥饿腰带模式）
-- mod 环境无 __index=_G（mods.lua#L258-L317）：EQUIPSLOTS/TheWorld 等游戏全局用 GLOBAL. 前缀，TUNING 直用
AddPrefabPostInit("mybelt", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    inst:AddComponent("equippable")
    inst.components.equippable.equipslot = GLOBAL.EQUIPSLOTS.BODY
    inst.components.equippable.walkspeedmult = 0.9

    inst.components.equippable:SetOnEquip(function(inst, owner)
        if owner.components.hunger ~= nil then
            owner.components.hunger.burnratemodifiers:SetModifier(inst, 0.6)  -- 饥饿燃烧 ×0.6
        end
    end)
    inst.components.equippable:SetOnUnequip(function(inst, owner)
        if owner.components.hunger ~= nil then
            owner.components.hunger.burnratemodifiers:RemoveModifier(inst)    -- 成对移除，否则永久锁速
        end
    end)
end)
```

- 调整与验证：替换点为 prefab 名与修正值；`walkspeedmult` 是字段赋值（非 modifier），卸下自动还原；示例只做语法检查，未在游戏内执行。

### 受击全额免伤护盾（骨头盔甲模式）

- 前提与选择：给装备做「充能期挡一刀」时用 `resistance` 组件而非 armor 吸收：玩家 `Inventory:ApplyDamage` 在装备槽循环里先查 resistance，命中即整击归零并返回，不进 armor 结算（[inventory.lua#L408-L420](../../../raw/current/scripts/components/inventory.lua#L408-L420)）。
- 生效与撤销：`ShouldResistFn` 必须自检「已装备」——resistance 组件本身不感知装备状态；原版以 fueled（燃料）＋cooldown（冷却）控制频率，并在抵挡后 `SetOnResistDamageFn(nil)` 开 1/3 秒（`SHIELD_DURATION = 10*FRAMES`）窗口，窗口内后续命中静默归零不再计费。**窗口结束的恢复要写成无条件**——原版 `OnShieldOver` 不看装备态，装备态由 `ShouldResistFn` 判断；写成「还装备着才恢复」会在窗口内卸甲/转移后永久丢掉回调（仍免伤但不再触发）。**窗口结束 `OnShieldOver` 会把七类 `RemoveResistance` 全撤**、只复位回调，等 cooldown 充能完成 `OnChargedFn` 才重新登记——所以原版冷却期真的挡不住。`ShouldResistFn` 里的 `forcefield` 判断是**拒绝本件承担这一击**（改由别的装备或本体结算），不是"只跳过扣费"。
- 原版依据：[ShouldResistFn/OnResistDamage/OnChargedFn](../../../raw/current/scripts/prefabs/armor_skeleton.lua#L57-L98)、[组件段](../../../raw/current/scripts/prefabs/armor_skeleton.lua#L229-L251)。

```lua
-- modmain.lua：自定义 BODY 装备的骨甲式全额免伤（简化协议，无燃料/冷却）
AddPrefabPostInit("mybonemail", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    local RESIST_TAGS = { "_combat", "explosive", "quakedebris" }  -- 原版七类见 armor_skeleton.lua#L12-L21
    local function OnResistDamage(inst)
        inst.components.resistance:SetOnResistDamageFn(nil)        -- 开 1/3 秒窗口：后续命中归零但不重复触发
        -- 简化点：原版窗口结束（OnShieldOver）会 RemoveResistance 全部七类、冷却完再登记，
        -- 这里保留登记不动，所以下面这份实现「冷却期仍能挡」，不要当成原版复刻
        -- 窗口结束「无条件」恢复回调：不要在这里判断是否还装备着。
        -- 窗口内卸甲（或转移/入包）时条件为假，回调就再也不会恢复：之后照样免伤
        -- （resistance 登记还在），却不再触发本回调。装备态交给 ShouldResistFn 判断。
        inst:DoTaskInTime(10 * GLOBAL.FRAMES, function()
            inst.components.resistance:SetOnResistDamageFn(OnResistDamage)
        end)
        -- 原版此处还做：SpawnPrefab("shadow_shield"..n) 挂 owner、fueled:DoDelta(-TUNING.MED_FUEL)、cooldown:StartCharging()
    end
    inst:AddComponent("resistance")
    for _, tag in ipairs(RESIST_TAGS) do
        inst.components.resistance:AddResistance(tag)
    end
    inst.components.resistance:SetShouldResistFn(function(inst)
        local owner = inst.components.inventoryitem.owner
        return inst.components.equippable:IsEquipped() and owner ~= nil
            and not (owner.components.inventory ~= nil
                     and owner.components.inventory:EquipHasTag("forcefield"))  -- 原版语义：持有者已有 forcefield（如铥矿皇冠 proc）时本件不承担这一击，改由别处结算
    end)
    inst.components.resistance:SetOnResistDamageFn(OnResistDamage)
end)
```

- 调整与验证：`AddResistance` 的标签匹配 attacker/weapon 的 tag（`"_combat"` 等价于挡所有战斗实体）；结算点在服务端 `Inventory:ApplyDamage`，客户端无需同步组件。**原版与简化版的差别在"撤登记"**：原版窗口一结束就撤掉七类 resistance、冷却完才重加，简化版永久登记、只切回调（等于冷却期仍免伤）；要做真复刻就把 `OnShieldOver` 的循环和 `OnChargedFn` 的登记一起抄。示例只做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **误区：三个"armor"都护甲，而且都不挡伤害。** `armorslurper`/`armorskeleton`/`armor_carrotlure` 名字带 armor 但**都没有 armor 组件**：腰带与外套确实全额穿透（不吸收），耐久体系分别是饥饿燃料与腐坏时间；**骨头盔甲例外**——它走 `resistance` 组件，充能窗口内对七类标签的攻击整击归零（[免伤链](../../../raw/current/scripts/prefabs/armor_skeleton.lua#L57-L98)）。给它们调 `SetAbsorption` 无效；`TUNING.ARMORSLURPER`（420）与 `ARMORSLURPER_ABSORPTION`（0.6）也无消费者（[armor_slurper.lua 全文无 armor](../../../raw/current/scripts/prefabs/armor_slurper.lua#L68-L84)）。
- **armor 耐久扣的是分摊量，暗夜甲的理智也按这份损耗算。** `Inventory:ApplyDamage` 先把各件吸收率求和、按占比把总吸收量分到每件（＋弱点与特殊伤害），再对每件调 `Armor:TakeDamage`；后者先乘该件 `conditionlossmultipliers`、再扣 `condition`，并把**这份已折算的值**传给 `ontakedamage`。所以暗夜甲扣的理智＝（分摊到该件的量×损耗倍率）×0.10，不是原始来伤 ×0.10（[分摊](../../../raw/current/scripts/components/inventory.lua#L408-L500)、[TakeDamage](../../../raw/current/scripts/components/armor.lua#L156-L164)、[OnTakeDamage](../../../raw/current/scripts/prefabs/armor_sanity.lua#L32-L41)）。
- **`armorslurper` 耗尽即毁且不能充能。** `SetDepletedFn(inst.Remove)` 直接移除本体；`fueled.accepting` 默认 false（[fueled.lua#L93](../../../raw/current/scripts/components/fueled.lua#L93)），燃料物品全部被 `CanAcceptFuelItem` 拒绝，唯一恢复途径是缝纫包（`needssewing` 标签 → `sewing:DoSewing` → `fueled:DoDelta`，缝纫包每次 +`SEWINGKIT_REPAIR_VALUE`＝2400 秒，[sewing.lua#L6-L10](../../../raw/current/scripts/components/sewing.lua#L6-L10)、[sewingkit.lua#L40-L41](../../../raw/current/scripts/prefabs/sewingkit.lua#L40-L41)）。
- **六件传统甲的 `blocked` 音效在普通受击不会响。** `combat:GetAttacked` 只在伤害被完全归零时推 `blocked`（[L653](../../../raw/current/scripts/components/combat.lua#L653)、[L730](../../../raw/current/scripts/components/combat.lua#L730)）；平时听到的甲命中声是 `GetArmorImpactSound` 按标签选型，且它要求装备位**有 armor 组件**（`ArmorHasTag` 判 `components.armor ~= nil`，[inventory.lua#L376-L382](../../../raw/current/scripts/components/inventory.lua#L376-L382)）——饥饿腰带的 `fur` 标签因此不参与。
- **蜗壳壳态依赖玩家 SG。** `useableitem` 回调直接 `owner.sg:GoToState("shell_enter")`：只有玩家 SG 有 `shell_enter/shell_idle/shell_hit` 状态（[SGwilson#L5656-L5752](../../../raw/current/scripts/stategraphs/SGwilson.lua#L5656-L5752)）；给生物装备会在右键时因缺状态出错。壳态满吸收的还原挂在 owner `newstate` 监听上，直接读**当前 BODY 槽** armor——若在壳态边缘换甲，改的是新甲。
- **`armor_rocky` 免击退有代价。** `superheavyarmor` 免击退由 SG 层实现（推 `knockbackblocked`）；嶙峋甲在该回调里对**骑乘状态**的持有者额外 `TakeDamage(52.5)`——这次损耗独立于受击吸收，改耐久公式时要同时核对这 1/20 上限项（[OnKnockbackBlocked](../../../raw/current/scripts/prefabs/armor_rocky.lua#L10-L15)、[tuning.lua#L1919](../../../raw/current/scripts/tuning.lua#L1919)）。
- **胡萝卜外套的引怪与避肉都以装备位为界。** 引怪任务句柄挂装备实体（卸下/试穿模型/腐坏都 `DisableLure`）；`hidesmeats` 走 `EquipHasTag`——放背包不生效（[componentutil#L776](../../../raw/current/scripts/componentutil.lua#L776)）。引来的兔随从忠诚由外套每秒 +3 秒维持，卸甲后忠诚按 follower 原有规则自然耗尽。
- **时间基数勿混。** 腰带 3840 秒＝8×`total_day_time`(480)；外套腐坏 1500 秒＝5×`day_time`(300)≈3.125 个全天；骨甲 720 秒燃料与挡刀 −45 均为 fueled 秒数（[tuning.lua#L1748](../../../raw/current/scripts/tuning.lua#L1748)、[L8505](../../../raw/current/scripts/tuning.lua#L8506)）。
- **modmain 全局前缀。** mod 环境（[mods.lua#L258-L317](../../../raw/current/scripts/mods.lua#L258-L317)）无 `__index=_G`，`TheWorld`/`EQUIPSLOTS`/`FRAMES` 等一律 `GLOBAL.` 前缀，`TUNING` 直用；`AddPrefabPostInit` 等注册函数由 modutil 注入、无前缀。

## 验证记录

- 源码审阅：2026-11-07，steam-buildid:26335394。10 个 armor 文件全文实读（注册名、InitCondition/SetAbsorption、附加组件、专属回调逐行核对）；tuning/recipes 按键实读；SGwilson shell 三态、combat GetAttacked、inventory ApplyDamage、fueled/sewing/hunger/resistance 组件关键函数实读；消费端标签以 scripts 域 grep 反查（检索范围：scripts 全域 `HasTag`/`EquipHasTag`/`ArmorHasTag` 命中点）。
- 名称核验：`languages/chinese_s.po`（Python UTF-8 逐键读 msgctxt `STRINGS.NAMES.*`）：ARMORGRASS＝草甲（[L338790](../../../raw/current/scripts/languages/chinese_s.po#L338790)）、ARMORMARBLE＝大理石甲（L338795）、ARMORRUINS＝铥矿甲（L338800）、ARMORSKELETON＝骨头盔甲（L338805）、ARMORSLURPER＝饥饿腰带（L338810）、ARMORSNURTLESHELL＝蜗壳护甲（L338815）、ARMORWOOD＝木甲（L338825）、ARMOR_CARROTLURE＝胡萝卜外套（L338835）、ARMOR_ROCKY＝嶙峋甲壳（L338850）、ARMOR_SANITY＝暗夜甲（L338855）；辅助键 SNURTLE＝蜗牛龟（L349710）、SLURTLE＝蛞蝓龟（L349640）、SLURPER_PELT＝啜食者皮（L349635）、ROCKY_BOSS＝岩石大师（L348685）。
- 配方与获取核验：recipes.lua 逐行——Recipe2：armorgrass L496、armorwood L497、armor_sanity L500、armormarble L501、armorslurper L521、armorruins L808（nounlock）、rabbitkingshop_armor_carrotlure L991（product=armor_carrotlure）；DeconstructRecipe：armor_carrotlure L1615、armorskeleton L1675、armor_rocky L1689。掉落/发落实读：snurtle 75% 掉蜗壳（[slurtle#L28-L31](../../../raw/current/scripts/prefabs/slurtle.lua#L28-L31)）、rocky_boss LOOT/NITRELOOT 固定含嶙峋甲（[rocky_boss#L58-L59](../../../raw/current/scripts/prefabs/rocky_boss.lua#L58-L59)、[L109/L114](../../../raw/current/scripts/prefabs/rocky_boss.lua#L109)）、锤古 altar 加权随机含大理石甲/暗夜甲/蜗壳 0.5（[altar_prototyper#L62-L97](../../../raw/current/scripts/prefabs/altar_prototyper.lua#L62-L97)、[L167-L172](../../../raw/current/scripts/prefabs/altar_prototyper.lua#L167-L172)、[L334/L431/L440](../../../raw/current/scripts/prefabs/altar_prototyper.lua#L334)）。草甲另见 hound 相关表与地图 boon（[hound.lua#L884](../../../raw/current/scripts/prefabs/hound.lua#L884)、[boons.lua#L140](../../../raw/current/scripts/map/boons.lua#L140)），消费链未展开。
- 归属核对（对分派简报的勘误）：文件名实为 `armor_marble.lua`/`armor_ruins.lua`/`armor_slurper.lua`/`armor_snurtleshell.lua`/`armor_wood.lua`（简报写成 armormarble 等连写变体），注册名与简报一致；`armor_sanity` 为暗夜甲（Night Armor，非"梦魇护甲"译名）；`armorslurper` 无吸附机制（实为饥饿减速腰带）；`armor_sanity` 非理智护盾而是「护甲吸收之外按受击扣理智」；`armor_rocky` 无石化机制（实为减速重甲＋免击退）；`armor_carrotlure` 无兔子仇恨伪装组件（伪装效果经由 `hidesmeats` 标签实现），核心是引兔随从；`armorruins` 与升级版 ruins 系（hats 的 ruinshat proc）无代码关联，仅同名前缀。10 件均为装备本体（非 FX/非共享工厂）。
- 皮肤核验：`prefabskins.lua` 逐键——armorgrass 2（cloak/woven）、armorwood 5、armormarble 4、armorruins 6、armor_sanity 3、armorskeleton 3、armor_rocky 1（spikey）；armorslurper/armorsnurtleshell/armor_carrotlure 无皮肤（[prefabskins.lua#L84-L143](../../../raw/current/scripts/prefabskins.lua#L84-L143)）。皮肤 fn 细节（basic_init_fn 协议）未展开。
- 分档说明：`size_class: complex`——10 注册逐件独立工厂且机制异构：壳态 SG 状态机、骨甲 fueled＋cooldown＋resistance 三组件协作、腰带 fueled 不可充耐久体系、诱饵 leader 周期任务、击退标签消费链需整体对照；获取途径横跨配方/Boss 掉落/掉落表/商店交易/古 altar 四类，超出 standard 单页容量。
- 未核验／待办：皮肤 init/clear fn 细节与 `armor_wood_rw_gustree`（_rw 前缀）皮肤来由；草甲 hound/boons 掉落的实际触发场景；rocky_boss 两套 loot 表的阶段切换条件（仅核到 `SetLoot` 调用行，未追 boss 阶段状态机）；`snail_armour_set` 成就的客户端展示；骨甲 `rock_tree` 标签的具体攻击者（登记表内保留原样，未反查写入者）；兔王商店 TECH.RABBITKINGSHOP_TWO 的解锁/建造条件。
- 结构检查：2026-11-07 第六轮修订后 `validate_page.py`（含 `--check-indexes`）通过（187 行／37807 字节，complex 档上限 480 行／65536 字节，0 issue）；`validate_source_links.py --strict` 通过（116 条引用、37 个源文件，0 error 0 warning）；两段 Lua 示例经 `parser_probe --syntax-only` 解析通过。
- 游戏运行：未运行；`validation` 保持 `source-reviewed`。

## 相关链接

- [[reference/prefabs/hats-armor|防护与战斗帽族]]（HEAD 槽护甲与战斗帽；蜗壳与 slurtlehat 的成就在其注册表内互认）
- [[reference/components/armor|armor 组件]]（耐久/吸收/弱点/受击协议的主要页）
- [[reference/components/resistance|resistance 组件]] 与 [[reference/components/fueled|fueled 组件]]（骨甲护盾与两种燃料耐久体系）
- [[reference/prefabs/shadow_shield1|骨甲护盾 FX]]（armorskeleton 抵挡表现）
- [[reference/components/equippable|equippable 组件]]（BODY 槽/减速/dapperness 的主要页）

返回 [[reference/prefabs/wearables-index|穿戴防具与饰品]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-08 build 26474873：本页随版本迁移升版（锚点迁移 2 处），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-10）。
