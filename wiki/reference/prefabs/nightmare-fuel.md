---
title: "噩梦燃料与纯粹恐惧（nightmarefuel、horrorfuel）"
created: "2026-11-10"
updated: "2026-11-10"
type: prefab
summary: "两个暗影燃料注册共用同一套物品骨架：噩梦燃料 180 秒燃料值、50 次耐久修理、LOW 暗影插槽；纯粹恐惧翻倍到 360 秒与 100 次、MEDIUM 插槽，另带沃特暗影法术与 60 秒 WX78 满充能。页面覆盖掉落源与概率、燃料/修理/施法/充能四条消费链的结算点，以及两者标签与品质的差异。"
aliases: ["nightmarefuel", "horrorfuel", "噩梦燃料", "纯粹恐惧", "恐怖燃料", "暗影燃料", "燃料与修理材料"]
tags: [prefab, item]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/nightmarefuel.lua", "prefabs/horrorfuel.lua", "prefabs/wx78_common.lua", "prefabs/wx78_shadow_fuel_debuff.lua", "prefabs/wurt.lua", "prefabs/skilltree_wurt.lua", "prefabs/nightmarecreature.lua", "prefabs/shadowcreature.lua", "prefabs/shrouden.lua", "prefabs/shadowthrall_hands.lua", "prefabs/shadowthrall_horns.lua", "prefabs/shadowthrall_mouth.lua", "prefabs/shadowthrall_wings.lua", "prefabs/shadowthrall_centipede.lua", "prefabs/chest_mimic.lua", "prefabs/daywalker.lua", "prefabs/fused_shadeling.lua", "prefabs/nightlight.lua", "prefabs/thurible.lua", "prefabs/amulet.lua", "prefabs/waxwelljournal.lua", "prefabs/pocketwatch_weapon.lua", "prefabs/armor_skeleton.lua", "prefabs/winona_battery_low.lua", "components/fueled.lua", "components/lootdropper.lua", "components/repairer.lua", "components/repairable.lua", "components/finiteuses.lua", "components/spellcaster.lua", "components/upgrademoduleowner.lua", "componentutil.lua", "standardcomponents.lua", "componentactions.lua", "stategraphs/SGwilson.lua", "recipes.lua", "tuning.lua", "constants.lua", "languages/chinese_s.po", "screens/redux/scrapbookdata.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 噩梦燃料与纯粹恐惧（nightmarefuel、horrorfuel）

## 快速判断

- 职责与适用：`nightmarefuel`（噩梦燃料）与 `horrorfuel`（纯粹恐惧）是两个独立注册、共用一套物品骨架的暗影燃料：在半透明 `idle_loop` 图形上挂 `inventoryitem`＋`stackable`＋`inspectable`＋`fuel`＋`repairer`，因此两者**同时是燃料与耐久修理材料**；差异集中在燃料值、修理值、插槽品质，以及 `horrorfuel` 才有的沃特施法与长充能。[噩梦燃料组件段](../../../raw/current/scripts/prefabs/nightmarefuel.lua#L78-L93)、[纯粹恐惧组件段](../../../raw/current/scripts/prefabs/horrorfuel.lua#L231-L253)
- 优先使用：查“这东西能塞进哪台设备”看[消费链](#消费链)；查“打谁掉几个”看[获取](#获取)；只关心 WX78 充能看[WX78 充能](#wx78-充能)，只关心沃特施法看[沃特施法](#沃特施法)。
- 重要副作用：两者都带 `waterproofer` 标签但 `SetEffectiveness(0)`，**没有任何防潮效果**；修理只对 `repairable.repairmaterial = MATERIALS.NIGHTMARE` 的目标生效，全库只有懒人护符一个目标。[防潮标签](../../../raw/current/scripts/prefabs/nightmarefuel.lua#L52-L55)、[无效果的防潮](../../../raw/current/scripts/prefabs/nightmarefuel.lua#L88-L89)、[修理材质](../../../raw/current/scripts/prefabs/nightmarefuel.lua#L84-L86)
- 修改边界：燃料值与修理值都不在 `TUNING` 里单独定义，而是 `TUNING.LARGE_FUEL` 与 `TUNING.NIGHTMAREFUEL_FINITEUSESREPAIRVALUE` 的原值／两倍；掉落概率归各生物页，设备侧判定归 [[reference/components/fueled]]、[[reference/components/repairable]]。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `nightmarefuel`（噩梦燃料） | 独立 `fn()`：实体→Transform/AnimState/Network→`MakeInventoryPhysics`→bank/build `nightmarefuel`＋`idle_loop`→`SetPristine`→主模拟挂组件 | 无 `prefabs` 依赖表；`fuelvalue = TUNING.LARGE_FUEL`、`finiteusesrepairvalue = 50`、`socketable` 品质 `LOW`；数组资源只有 `anim/nightmarefuel.zip`。[工厂](../../../raw/current/scripts/prefabs/nightmarefuel.lua#L37-L96)、[注册](../../../raw/current/scripts/prefabs/nightmarefuel.lua#L98) |
| `horrorfuel`（纯粹恐惧） | 同骨架＋客户端 `CreateCore()` 子实体（专用服务器跳过）＋主模拟 `spellcaster` | `fuelvalue = TUNING.LARGE_FUEL * 2`、`finiteusesrepairvalue = 50 * 2`、品质 `MEDIUM`；`prefabs` 表声明 `wurt_merm_planar`／`horrorfuel_castfx`／`horrorfuel_castfx_mount`。[工厂](../../../raw/current/scripts/prefabs/horrorfuel.lua#L178-L260)、[注册](../../../raw/current/scripts/prefabs/horrorfuel.lua#L262) |

两条工厂共用的骨架（按执行顺序）：

- **表现**：都是 `SetBank`/`SetBuild`＝`nightmarefuel`／`horrorfuel`、`PlayAnimation("idle_loop", true)`、`SetMultColour(1, 1, 1, 0.5)`（半透明）、`UsePointFiltering(true)`；主模拟再把当前帧随机到 `math.random(GetCurrentAnimationNumFrames()) - 1`，纯表现、不参与判定。[nightmarefuel 表现](../../../raw/current/scripts/prefabs/nightmarefuel.lua#L46-L50)、[horrorfuel 表现](../../../raw/current/scripts/prefabs/horrorfuel.lua#L187-L191)、[随机帧](../../../raw/current/scripts/prefabs/horrorfuel.lua#L229)
- **漂浮与作祟**：`MakeInventoryFloatable` 走默认小号漂浮参数；`MakeHauntableLaunch` 只抛飞、不损坏，实现里顺手把 `inventoryitem.is_landed` 清掉。[漂浮](../../../raw/current/scripts/prefabs/nightmarefuel.lua#L55)、[作祟](../../../raw/current/scripts/prefabs/nightmarefuel.lua#L91)、[作祟实现](../../../raw/current/scripts/standardcomponents.lua#L1053-L1069)
- **堆叠**：`stackable.maxsize = TUNING.STACK_SIZE_SMALLITEM`，即 40；采集册同值为 `stacksize=40`。[赋值](../../../raw/current/scripts/prefabs/nightmarefuel.lua#L78-L79)、[常量](../../../raw/current/scripts/tuning.lua#L83)、[采集册](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L1011)
- **Pristine 前的客户端钩子**：`inst.UseableTargetedItem_ValidTarget`（WX78 消耗目标判定）与 `MakeItemSocketable_Client(inst, SOCKETNAMES.SHADOW)` 都在 `SetPristine` 之前接线，因为 socketable 客户端 helper 自己会挂 hook。[接线](../../../raw/current/scripts/prefabs/nightmarefuel.lua#L57-L61)
- **服务端顺序固定**：`useabletargeteditem` → `WX78Common.MakeItemSocketable` → 插槽品质 → `stackable` → `inspectable` → `fuel` → `repairer` → `waterproofer` → `MakeHauntableLaunch` → `inventoryitem`；`horrorfuel` 末尾再补 `spellcaster`。[噩梦燃料](../../../raw/current/scripts/prefabs/nightmarefuel.lua#L70-L93)、[纯粹恐惧](../../../raw/current/scripts/prefabs/horrorfuel.lua#L223-L253)

两者逐项差异（同页对照）：

| 项目 | `nightmarefuel` | `horrorfuel` | 证据 |
|---|---|---|---|
| 燃料值 | `TUNING.LARGE_FUEL`＝180 秒 | ×2＝360 秒 | [180](../../../raw/current/scripts/prefabs/nightmarefuel.lua#L81-L83)、[360](../../../raw/current/scripts/prefabs/horrorfuel.lua#L234-L236) |
| 修理量 | `finiteusesrepairvalue = 50` | ×2＝100 | [50](../../../raw/current/scripts/prefabs/nightmarefuel.lua#L84-L86)、[100](../../../raw/current/scripts/prefabs/horrorfuel.lua#L237-L239) |
| 暗影插槽品质 | `SOCKETQUALITY.LOW` | `SOCKETQUALITY.MEDIUM` | [LOW](../../../raw/current/scripts/prefabs/nightmarefuel.lua#L73-L74)、[MEDIUM](../../../raw/current/scripts/prefabs/horrorfuel.lua#L226-L227) |
| WX78 充能时长 | 20 秒 | 60 秒 | [20](../../../raw/current/scripts/prefabs/nightmarefuel.lua#L12)、[60](../../../raw/current/scripts/prefabs/horrorfuel.lua#L153) |
| 施法 | 无 | `SPELLTYPES.WURT_SHADOW` | [施法组件](../../../raw/current/scripts/prefabs/horrorfuel.lua#L248-L253) |
| 额外标签 | 无 | `purehorror`、`mermbuffcast` | [标签](../../../raw/current/scripts/prefabs/horrorfuel.lua#L193-L196) |
| 客户端光效 | 无 | `core` 子实体（理智越低越亮） | [CreateCore](../../../raw/current/scripts/prefabs/horrorfuel.lua#L102-L126) |

`horrorfuel` 的 `core` 是**不联网的本地 FX 子实体**：`persists = false`、`AddTag("FX")`、`entity:SetParent(inst.entity)`，并挂 `highlightchildren`；亮度和本地玩家理智挂钩——`CalcTargetLightOverride` 在 `replica.sanity:IsInsanityMode()` 且理智百分比 `k < 0.6` 时返回 `(1 - k/0.6)^2`，再由 1 秒周期任务以 `targetlight * .1 + currentlight * .9` 平滑。[核心](../../../raw/current/scripts/prefabs/horrorfuel.lua#L102-L126)、[亮度公式](../../../raw/current/scripts/prefabs/horrorfuel.lua#L19-L37)、[唤醒/睡眠](../../../raw/current/scripts/prefabs/horrorfuel.lua#L75-L100)

`mermbuffcast` 这个标签只影响玩家侧的施法动画：`SGwilson` 的 `CASTSPELL` 分支按它选 `mermbuffcastspell`，与效果结算无关。[动画分支](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1303-L1314)

### 名称核验

两个注册的显示名直接取自己的名字键，没有 `nameoverride`：`NAMES.NIGHTMAREFUEL`＝**噩梦燃料**、`NAMES.HORRORFUEL`＝**纯粹恐惧**（英文名 `Pure Horror`，不是“恐怖燃料”）。[噩梦燃料名称](../../../raw/current/scripts/languages/chinese_s.po#L346149-L346152)、[纯粹恐惧名称](../../../raw/current/scripts/languages/chinese_s.po#L343964-L343967)

## 获取

掉落表用 `SetSharedLootTable` 登记、由 `SetChanceLootTable` 挂到生物上；每条独立 roll——`chance >= 1.0` 直接入列，小于 1 的先过 `GetChance`，按死亡归因实体（`health.causeofdeath`）或最后工作者（`workable.lastworker`）的**正向幸运**抬高，无幸运／负幸运回落基础值。所以表里的数字是**无幸运修正的基础概率**而不是数量，同一 prefab 列多次就分别判定；“1.00＋1.00＋0.5”是基础期望 2.5 件、不是保底 3 件。[生成](../../../raw/current/scripts/components/lootdropper.lua#L163-L227)、[幸运公式](../../../raw/current/scripts/componentutil.lua#L2092-L2097)

| 来源 | 掉噩梦燃料 | 掉纯粹恐惧 | 条件与证据 |
|---|---|---|---|
| 地面暗影生物 `crawlinghorror`／`terrorbeak`（`shadow_creature` 表） | 1.00＋0.50 | — | 疯狂时出现；`persists = false`。[表](../../../raw/current/scripts/prefabs/shadowcreature.lua#L147-L151)、[接线](../../../raw/current/scripts/prefabs/shadowcreature.lua#L333-L334) |
| 暗影生物通用表 `nightmare_creature`（`crawlingnightmare`／`nightmarebeak`） | 1.00＋0.50 | — | 由 `MakeShadowCreature` 统一接线。[表](../../../raw/current/scripts/prefabs/nightmarecreature.lua#L46-L50)、[接线](../../../raw/current/scripts/prefabs/nightmarecreature.lua#L256-L257) |
| 废墟暗影生物 `ruinsnightmare` | 1.00＋1.00＋0.50＋0.25 | — | 无裂隙时的默认表。[表](../../../raw/current/scripts/prefabs/nightmarecreature.lua#L123-L129)、[接线](../../../raw/current/scripts/prefabs/nightmarecreature.lua#L319) |
| 同上，暗影门户激活时换 `ruinsnightmare_rifts` | 1.00＋0.67 | 1.00＋1.00＋0.50 | 条件是 `riftspawner:IsShadowPortalActive()`；并顺手加 `planarentity`／`planardamage`、提速。[表](../../../raw/current/scripts/prefabs/nightmarecreature.lua#L131-L138)、[切换](../../../raw/current/scripts/prefabs/nightmarecreature.lua#L141-L157) |
| 暗影憎恶 `shrouden` | 1.00＋1.00＋0.75 | 1.00×5＋0.75 | 死亡表；另有 `shrouden2` 表专掉绝望石／暗影碎布。[表](../../../raw/current/scripts/prefabs/shrouden.lua#L32-L44)、[接线](../../../raw/current/scripts/prefabs/shrouden.lua#L1434) |
| 裂影仆从 `shadowthrall_horns`／`_mouth` | 1.00＋1.00＋0.67 | 1.00＋0.50 | 受控裂隙刷出的三件套之一。[horns 表](../../../raw/current/scripts/prefabs/shadowthrall_horns.lua#L17-L27)、[mouth 表](../../../raw/current/scripts/prefabs/shadowthrall_mouth.lua#L16-L26) |
| 裂影仆从 `shadowthrall_wings` | 1.00＋0.67 | 1.00＋1.00＋0.33 | 同上，概率分布不同。[表](../../../raw/current/scripts/prefabs/shadowthrall_wings.lua#L18-L27) |
| 裂影仆从 `shadowthrall_hands` | 1.00＋1.00＋0.67 | 1.00＋0.50 | 同表还掉 3×1.00＋0.33 暗影碎布。[表](../../../raw/current/scripts/prefabs/shadowthrall_hands.lua#L17-L28) |
| 地下裂影蜈蚣 `shadowthrall_centipede_head` | 1.00＋1.00＋0.50＋0.25＋0.25 | 1.00＋1.00＋0.25 | 同时掉 3 段绝望石（1.00／0.66／0.34）。[表](../../../raw/current/scripts/prefabs/shadowthrall_centipede.lua#L23-L37) |
| 华丽箱子 `chest_mimic` | — | 1.00×3＋0.75＋0.50×2 | 打开箱子触发的拟态形态。[表](../../../raw/current/scripts/prefabs/chest_mimic.lua#L208-L215) |
| 梦魇疯猪 `daywalker` | 0.50 | 1.00×4＋0.50 | 另有“未从监狱放出”时换 `BONUS_PILLAR_LOOT` 的开关。[表](../../../raw/current/scripts/prefabs/daywalker.lua#L29-L37)、[开关](../../../raw/current/scripts/prefabs/daywalker.lua#L1101-L1102) |
| 熔合暗影 `fused_shadeling` | — | 1.00＋1.00＋0.75＋0.50 | 裂隙侧小怪。[表](../../../raw/current/scripts/prefabs/fused_shadeling.lua#L23-L28) |

另外两条不看脸的来源：

- **合成**：`Recipe2("nightmarefuel", {Ingredient("petals_evil", 4)}, TECH.MAGIC_TWO)`——4 个恶花瓣换 1 个噩梦燃料，无角色限制。[配方](../../../raw/current/scripts/recipes.lua#L467)
- **嬗变三角**（均需威尔逊 `wilson_allegiance_shadow` 技能、`TECH.NONE`）：1 绝望石→2 纯粹恐惧、3 纯粹恐惧→1 绝望石、1 纯粹恐惧→2 噩梦燃料。[三条](../../../raw/current/scripts/recipes.lua#L1495-L1497)

## 消费链

### 燃料

`FUELTYPE.NIGHTMARE` 是独立燃料类型，不是 `BURNABLE`；设备侧 `Fueled:CanAcceptFuelItem` 只比较 `fuel.fueltype == self.fueltype or fuel.fueltype == self.secondaryfueltype`，所以**营火一类默认 `BURNABLE` 的燃烧设备收不下它们**。[类型常量](../../../raw/current/scripts/constants.lua#L1995-L2005)、[接受判定](../../../raw/current/scripts/components/fueled.lua#L169-L175)、[默认燃料类型](../../../raw/current/scripts/components/fueled.lua#L94-L95)

已核验的 `FUELTYPE.NIGHTMARE` 消费者共 6 个 `fueled` 设备，加 1 个带门槛的发电机（放置态与物品态两个注册）：

| 设备（注册名） | 容量（秒） | 与单块燃料的关系 | 证据 |
|---|---|---|---|
| 暗夜灯 `nightlight` | `(night_time + dusk_time) * 3`＝540 | 3 块噩梦燃料填满（推导：540 ÷ 180） | [fueltype](../../../raw/current/scripts/prefabs/nightlight.lua#L137-L140)、[容量](../../../raw/current/scripts/tuning.lua#L1710) |
| 暗影香炉 `thurible` | `(night_time + dusk_time) * 3`＝540 | 同上 | [fueltype](../../../raw/current/scripts/prefabs/thurible.lua#L280)、[容量](../../../raw/current/scripts/tuning.lua#L3416) |
| 骨头盔甲 `armorskeleton` | `4 * TUNING.LARGE_FUEL`＝720 | 4 块 | [初始化](../../../raw/current/scripts/prefabs/armor_skeleton.lua#L233-L238) |
| 魔光护符 `yellowamulet` | `TUNING.YELLOWAMULET_FUEL`＝`total_day_time`＝480 | 2.67 块（推导：480 ÷ 180） | [fueltype](../../../raw/current/scripts/prefabs/amulet.lua#L584-L589)、[容量](../../../raw/current/scripts/tuning.lua#L207) |
| 暗影秘典 `waxwelljournal` | `TUNING.LARGE_FUEL * 4`＝720 | 4 块 | [fueltype](../../../raw/current/scripts/prefabs/waxwelljournal.lua#L683)、[容量](../../../raw/current/scripts/prefabs/waxwelljournal.lua#L686) |
| 警钟 `pocketwatch_weapon` | `4 * TUNING.LARGE_FUEL`＝720 | 4 块 | [fueltype](../../../raw/current/scripts/prefabs/pocketwatch_weapon.lua#L171)、[容量](../../../raw/current/scripts/prefabs/pocketwatch_weapon.lua#L172) |
| 薇诺娜的发电机 `winona_battery_low`／`winona_battery_low_item` | 次级燃料类型（主类型 `CHEMICAL`） | 两个注册同一套 `CanAddFuelItem`＋`CalcFuelMultiplier`：有技能才收，且值只按 **0.5 倍** | [放置态](../../../raw/current/scripts/prefabs/winona_battery_low.lua#L1106-L1114)、[物品态与注册](../../../raw/current/scripts/prefabs/winona_battery_low.lua#L1134-L1136)、[门槛](../../../raw/current/scripts/prefabs/winona_battery_low.lua#L462-L467)、[0.5 倍](../../../raw/current/scripts/prefabs/winona_battery_low.lua#L579-L581) |

结算链（谁乘谁）：`fuelvalue × self.bonusmult × multfn结果 × wetmult × masterymult`，其中潮湿燃料乘 `TUNING.WET_FUEL_PENALTY`＝0.75；发电机那条 `multfn` 就是上表的 `CalcFuelMultiplier`。[结算](../../../raw/current/scripts/components/fueled.lua#L197-L230)、[潮湿惩罚](../../../raw/current/scripts/tuning.lua#L1671)

### 修理

`repairer` 组件挂上后按数值给自己打标签：`finiteusesrepairvalue > 0` 打 `finiteuses_<material>`；`MATERIALS.NIGHTMARE`＝`"nightmare"`，所以是 `finiteuses_nightmare`。[标签生成](../../../raw/current/scripts/components/repairer.lua#L8-L24)、[材质常量](../../../raw/current/scripts/constants.lua#L1956)

- 候选：目标必须带 `repairable_nightmare`，且（有 `finiteuses_nightmare` 且目标带 `finiteusesrepairable`）才给 `REPAIR`；目标侧的 `finiteusesrepairable` 由 `finiteuses` 在每次耐久变化时按“未满”刷新。[候选](../../../raw/current/scripts/componentactions.lua#L1688-L1697)、[目标侧刷新](../../../raw/current/scripts/components/finiteuses.lua#L1-L8)
- 结算：`finiteuses:Repair(finiteusesrepairvalue)`，命中后从堆叠扣 1；目标满耐久时直接返回 false 不消耗。[结算](../../../raw/current/scripts/components/repairable.lua#L135-L142)、[消耗](../../../raw/current/scripts/components/repairable.lua#L172-L176)
- 全库唯一目标是懒人护符 `orangeamulet`：`repairable.repairmaterial = MATERIALS.NIGHTMARE`、总耐久 `ORANGEAMULET_USES`＝225；每块请求 +`50`（`finiteuses:Repair` 按上限截断）。`ceil(225 ÷ 50)`＝5 只是纯算术：护符耐久耗尽会在 `SetOnFinished(inst.Remove)` 里被移除，不存在“0 耐久待修”的实例；按还有 25 点的实例算，4 块即补满（推导）。[目标](../../../raw/current/scripts/prefabs/amulet.lua#L556-L564)、[耗尽即移除](../../../raw/current/scripts/prefabs/amulet.lua#L556-L559)、[总耐久](../../../raw/current/scripts/tuning.lua#L214)
- `horrorfuel` 同一标签、同一目标，一次 +100。

### WX78 充能

`ValidTargetToConsumeAsWX78` 要求目标（或持有者）有 `socketholder` 且最高品质的 `SOCKETNAMES.SHADOW` 插槽高于 `SOCKETQUALITY.NONE`，并只允许目标是自己或 `nil`；满足时客户端把 `GetUseItemOnVerb` 返回为 `"CONSUME"`，服务端在 `OnUseAsWX78` 里给目标挂 `wx78_shadow_fuel_debuff` 并按堆叠扣 1。[目标判定](../../../raw/current/scripts/prefabs/nightmarefuel.lua#L25-L35)、[动作候选](../../../raw/current/scripts/componentactions.lua#L1900-L1913)、[使用](../../../raw/current/scripts/prefabs/nightmarefuel.lua#L13-L23)

那个 `debuff` 实际是限时满充能：`buff_OnAttached` 调 `upgrademoduleowner:SetOverrideFullCharge(true)`，并把 `shadow_energy` 冷却按 `data.duration` 重排；`GetChargeLevel` 在 override 期间直接返回 `GetMaxChargeLevel()`，到点 `buff_OnDetached` 再关掉。[附加](../../../raw/current/scripts/prefabs/wx78_shadow_fuel_debuff.lua#L11-L32)、[分离](../../../raw/current/scripts/prefabs/wx78_shadow_fuel_debuff.lua#L44-L53)、[读取点](../../../raw/current/scripts/components/upgrademoduleowner.lua#L361-L376)、[时长](../../../raw/current/scripts/tuning.lua#L7585-L7586)

### 沃特施法

只有 `horrorfuel` 挂 `spellcaster`：`SetSpellType(SPELLTYPES.WURT_SHADOW)`、`canuseontargets`、`canusefrominventory`、`canonlyuseonlocomotorspvp`，颜色 `{102/255, 16/255, 16/255}`、音效 `meta4/casting/shadow`。[接线](../../../raw/current/scripts/prefabs/horrorfuel.lua#L248-L257)

三道门都要过：

1. **施法者技能**：`spelltype` 非空时，`CanCast` 要求施法者带 `<spelltype>_spelluser`；该标签由沃特技能 `wurt_shadow_allegiance_1` 的 `onactivate` 打上，即 `wurt_shadow_spelluser`。[门](../../../raw/current/scripts/components/spellcaster.lua#L152-L157)、[技能](../../../raw/current/scripts/prefabs/skilltree_wurt.lua#L375-L377)
2. **目标**：`canonlyuseonlocomotorspvp` 要求目标有 `locomotor`，且“目标是施法者自己／开了 PVP／目标不是玩家而施法者是玩家”三者任一。[判定](../../../raw/current/scripts/components/spellcaster.lua#L190-L199)
3. **动作候选**：物品带 `castonlocomotorspvp`（由 `canonlyuseonlocomotorspvp` 在 `oncancast` 里打的标签）；若施法者没有 `wurt_shadow_spelluser`，候选端也不给动作。[候选](../../../raw/current/scripts/componentactions.lua#L1780-L1784)、[标签来源](../../../raw/current/scripts/components/spellcaster.lua#L16-L18)

效果作用在**施法者自己的随从**而不是目标身上：`Wurt_MermSpellFn` 遍历 `doer.components.leader.followers`，给每个带 `shadowminion` 且未死的随从挂 `wurt_merm_planar` 并推 `cheer`，然后从堆叠扣 1。[施法函数](../../../raw/current/scripts/prefabs/horrorfuel.lua#L134-L147)、[欢呼](../../../raw/current/scripts/prefabs/horrorfuel.lua#L128-L132)

`wurt_merm_planar` 的数值：`planardamage:SetBaseDamage(TUNING.WURT_PLANAR_BUFF_DAMAGE)`＝20，持续 `TUNING.WURT_PLANAR_TIME_BUFF_LASTS`＝`8 * seg_time`＝240 秒（0.5 天）；`shadowminion` 分支额外生成 `wurt_shadow_merm_planar_fx` 并把 `planarbuffed` 置 true。[附加](../../../raw/current/scripts/prefabs/wurt.lua#L712-L747)、[数值](../../../raw/current/scripts/tuning.lua#L8332-L8334)

### 配方消费者

两个注册在配方侧的数量：`nightmarefuel` 49 条（42 条 `Recipe2` ＋ 7 条 `DeconstructRecipe`），`horrorfuel` 21 条（20＋1）；同一装备只列一次，装备侧效果归各自页面。

| 分组 | 消耗 | 代表与指针 |
|---|---|---|
| 暗影与魔法装备／设施 | 噩梦燃料 | `nightsword`、`armor_sanity`、`amulet`／`purpleamulet`、`firestaff`／`telestaff`／`telebase`、`nightlight`、`researchlab3`、`magician_chest`、`book_sleep`、`waxwelljournal`。[例](../../../raw/current/scripts/recipes.lua#L483)、[例](../../../raw/current/scripts/recipes.lua#L388) |
| 远古／废墟系 | 噩梦燃料 | 四色护符与三法杖的远古门槛、`nightmare_timepiece`、`ruinshat`／`armorruins`／`ruins_bat`、`turf_ruins*`、`slingshotammo_thulecite`、`slingshot_frame_gems`。[例](../../../raw/current/scripts/recipes.lua#L807-L809) |
| 角色专属 | 噩梦燃料 | 旺达 `pocketwatch_parts`／`pocketwatch_weapon`、温蒂 `abigail_flower`、伍迪 `leif_idol`、WX78 `wx78module_maxsanity` 与两种 `wx78_shadowdrone_*`、麦斯威尔四式暗影仆从。[例](../../../raw/current/scripts/recipes.lua#L297-L300)、[例](../../../raw/current/scripts/recipes.lua#L790-L791) |
| 裂隙与暗影术（纯粹恐惧主场） | 两者 | 虚空四件＋`voidcloth_kit`、`beeswax_spray`、`shadow_battleaxe`／`saddle_shadow`／`shadow_beef_bell`、`armordreadstone`／`dreadstonehat`、`punchingbag_shadow`、沃特 `wurt_swampitem_shadow`、温蒂 `ghostlyelixir_shadow`、女武神 `battlesong_shadowaligned`、沃尔特 `slingshotammo_horrorfuel`。[例](../../../raw/current/scripts/recipes.lua#L866-L875) |
| 拆解回收 | 两者 | 噩梦燃料 7 条（`opalstaff`、`shadow_forge`、`gelblob_storage`、`armorskeleton`、`skeletonhat`、`thurible`、`shieldofterror`），纯粹恐惧 1 条（`shadow_forge`）。[拆解段](../../../raw/current/scripts/recipes.lua#L1661-L1664) |

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`fuel` | `fueltype = FUELTYPE.NIGHTMARE`、`fuelvalue` 180／360 | 主模拟；不参与 `BURNABLE` 设备 | [[reference/components/fuel]]；[赋值](../../../raw/current/scripts/prefabs/nightmarefuel.lua#L81-L83) |
| 组件：`repairer` | `MATERIALS.NIGHTMARE`＋`finiteusesrepairvalue` 50／100 | 主模拟；打 `finiteuses_nightmare` 标签 | [[reference/components/repairer]]；[赋值](../../../raw/current/scripts/prefabs/nightmarefuel.lua#L84-L86) |
| 组件：`stackable` | `maxsize = STACK_SIZE_SMALLITEM`（40） | 主模拟；两者同值 | [[reference/components/stackable]]；[赋值](../../../raw/current/scripts/prefabs/nightmarefuel.lua#L78-L79) |
| 组件：`socketable` | `MakeItemSocketable_Client(..., SOCKETNAMES.SHADOW)`＋`WX78Common.MakeItemSocketable`；品质 LOW／MEDIUM | 客户端 hook＋服务端组件；插槽来自 WX78 的 `socketholder` | [客户端](../../../raw/current/scripts/prefabs/nightmarefuel.lua#L58-L59)、[服务端](../../../raw/current/scripts/prefabs/nightmarefuel.lua#L73-L74)、[helper](../../../raw/current/scripts/prefabs/wx78_common.lua#L589-L590) |
| 组件：`useabletargeteditem` | 只有 WX78 满足插槽条件时给 `CONSUME`；消耗 1 个加 20／60 秒满充能 | 主模拟；`target == doer or target == nil` | [接线](../../../raw/current/scripts/prefabs/horrorfuel.lua#L223-L224)、[debuff](../../../raw/current/scripts/prefabs/wx78_shadow_fuel_debuff.lua#L11-L32) |
| 组件：`spellcaster` | 仅 `horrorfuel`：`SPELLTYPES.WURT_SHADOW`＋三个 flag | 主模拟；施法者需 `wurt_shadow_spelluser` | [[reference/components/spellcaster]]；[接线](../../../raw/current/scripts/prefabs/horrorfuel.lua#L248-L253) |
| 组件：`waterproofer` | `SetEffectiveness(0)`，标签在 pristine 前加 | 两端；无任何防潮效果 | [标签](../../../raw/current/scripts/prefabs/nightmarefuel.lua#L52-L55)、[赋值](../../../raw/current/scripts/prefabs/nightmarefuel.lua#L88-L89) |
| 关联 Prefab：`wx78_shadow_fuel_debuff` | WX78 充能载体，非联网实体、`CLASSIFIED` | 服务端；`buff_OnTimerDone` 到点自停 | [注册](../../../raw/current/scripts/prefabs/wx78_shadow_fuel_debuff.lua#L61-L88) |
| 关联 Prefab：`wurt_merm_planar` | 鱼人位面伤害 buff，8 秒×30 后停 | 服务端；`shadowminion` 分支带暗影 FX | [注册](../../../raw/current/scripts/prefabs/wurt.lua#L877-L879) |
| 关联 Prefab：`horrorfuel_castfx`／`horrorfuel_castfx_mount` | 施法特效与坐骑形态特效 | 由 `spellcaster` 的 `fxprefab`／坐骑表取 | [声明](../../../raw/current/scripts/prefabs/horrorfuel.lua#L8-L13) |
| 资源：`anim/nightmarefuel.zip`／`anim/horrorfuel.zip` | 各自的 bank／build；采集册动画单独取 `scrapbook` 帧 | 原版资源，不随页分发 | [asset](../../../raw/current/scripts/prefabs/nightmarefuel.lua#L3-L6)、[asset](../../../raw/current/scripts/prefabs/horrorfuel.lua#L3-L6) |

## 常见陷阱与限制

- **误区：噩梦燃料能往营火里丢。** 营火侧 `fueled` 没有改 `fueltype`，保持默认 `BURNABLE`；物品是 `NIGHTMARE`，`CanAcceptFuelItem` 直接失配——要烧只能找上表那 6 台设备。[默认](../../../raw/current/scripts/components/fueled.lua#L94-L95)、[判定](../../../raw/current/scripts/components/fueled.lua#L169-L175)
- **边界：薇诺娜的发电机是“0.5 倍”而不是“不收”。** 它把 `NIGHTMARE` 设成 `secondaryfueltype`，`CanAddFuelItem` 只对带 `winona_shadow_1`（噩梦燃料）或 `winona_shadow_2`（纯粹恐惧）技能的工程师放行，且 `CalcFuelMultiplier` 返回 0.5，实际入账 90／180 秒。[次级类型](../../../raw/current/scripts/prefabs/winona_battery_low.lua#L920-L928)、[技能门槛](../../../raw/current/scripts/prefabs/winona_battery_low.lua#L462-L467)、[0.5](../../../raw/current/scripts/prefabs/winona_battery_low.lua#L579-L581)
- **边界：`waterproofer` 标签不等于防潮。** 标签在 pristine 阶段就加，组件却在主模拟才 `SetEffectiveness(0)`；燃料本身照常变湿，潮湿时按 0.75 结算燃料值。[标签](../../../raw/current/scripts/prefabs/nightmarefuel.lua#L52-L55)、[湿度结算](../../../raw/current/scripts/components/fueled.lua#L205-L212)
- **边界：`purehorror` 标签没有消费端。** 全库检索只有 `horrorfuel.lua` 一处 `AddTag("purehorror")`，没有任何判定读它；需要区分“纯粹恐惧这件物品”时用 `prefab` 比较（发电机就是这么写的），不要指望这个标签。[生产处](../../../raw/current/scripts/prefabs/horrorfuel.lua#L195)、[按 prefab 判定](../../../raw/current/scripts/prefabs/winona_battery_low.lua#L477)
- **边界：施法“对目标”只是动作门槛。** 真正吃到 buff 的是施法者的 `shadowminion` 随从，目标只是 `canonlyuseonlocomotorspvp` 的可选对象；没有随从时施法同样扣燃料。[函数](../../../raw/current/scripts/prefabs/horrorfuel.lua#L134-L147)
- **边界：修理只在未满时生效且精确匹配材质。** `Repairable:Repair` 先比 `repairmaterial` 再落数值，满耐久返回 false 不消耗物品；把噩梦燃料拿去修别的东西不会有任何结果。[比对](../../../raw/current/scripts/components/repairable.lua#L153-L165)
- **未核验**：未运行游戏。上述概率、秒数与推导值全部来自源码；多人环境下 `horrorfuel` 的 `core` 光效只跟随本地玩家（`_player` 单例），分屏与专用服务器表现未验证。

## 验证记录

- 源码审阅：2026-11-10，Steam buildid 26474873。通读 `prefabs/nightmarefuel.lua`、`prefabs/horrorfuel.lua` 全文；`prefabs/wx78_common.lua` 的 `MakeItemSocketable` 出口、`prefabs/wx78_shadow_fuel_debuff.lua` 全文、`prefabs/wurt.lua` 的 `wurt_merm_planar` 定义与 `prefabs/skilltree_wurt.lua` 的技能挂标签段；`components/fueled.lua`（`CanAcceptFuelItem`／`TakeFuelItem`）、`components/repairer.lua`、`components/repairable.lua`、`components/finiteuses.lua`、`components/spellcaster.lua`、`components/upgrademoduleowner.lua`、`componentactions.lua` 的 `repairer`／`useabletargeteditem`／`spellcaster` 候选段；6 个 `fueled` 消费者与 `winona_battery_low` 的燃料段；`prefabs/nightmarecreature.lua`、`shadowcreature.lua`、`shrouden.lua`、四个 `shadowthrall_*`、`shadowthrall_centipede.lua`、`chest_mimic.lua`、`daywalker.lua`、`fused_shadeling.lua` 的掉落表；`recipes.lua` 的配方与嬗变、`tuning.lua` 的 `LARGE_FUEL`／修理值／两个 debuff 时长／各设备容量、`constants.lua` 的 `MATERIALS`／`FUELTYPE`／`SPELLTYPES`；名称取 `chinese_s.po`，堆叠与燃料值用 `scrapbookdata.lua` 交叉核对（`nightmarefuel` 40／180、`horrorfuel` 40／360）。
  未覆盖：`horrorfuel_castfx` 与 `_mount` 两个 FX 的独立行为、`purehorror` 标签在模组侧的用途、其它 `NIGHTMARE` 燃料设备（`waxwelljournal`／`pocketwatch_weapon`）的容量与消耗曲线（归各自页面）、`charlie_boss` 的掉落表。
- 游戏运行：未运行。燃料秒数、修理次数、容量换算与概率均为源码推导（`seg_time`＝30、`total_day_time`＝480）。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 见交付回报。

## 相关链接

- 装备侧消费者：[[reference/prefabs/amulets]]（懒人护符／魔光护符）、[[reference/prefabs/hats-armor]]、[[reference/prefabs/armor-rift]]、[[reference/prefabs/nightsword]]、[[reference/prefabs/shadow-battleaxe]]
- 组件：[[reference/components/fuel]]、[[reference/components/repairer]]、[[reference/components/repairable]]、[[reference/components/stackable]]、[[reference/components/spellcaster]]
- 同批材料页：[[reference/prefabs/dreadstone]]、[[reference/prefabs/voidcloth]]

返回 [[reference/prefabs/index|Prefab 索引]]。
