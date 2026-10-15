---
title: "农田作物专用种子（14 个 <作物>_seeds）"
created: "2026-11-08"
updated: "2026-11-08"
type: prefab
summary: "veggies.lua 的 fn_seeds 为 14 个可种作物各产出一件专用种子：共用 farm_plant_seeds 动画与 SEEDS 食性、40 堆叠与 40 天保质、烤制后统一变成通用 seeds_cooked，既是陷阱诱饵也是海钓拟饵，播种经 farmplantable 与 Wormwood 自定义部署两条链。"
aliases: ["carrot_seeds", "corn_seeds", "pumpkin_seeds", "eggplant_seeds", "durian_seeds", "pomegranate_seeds", "dragonfruit_seeds", "watermelon_seeds", "tomato_seeds", "potato_seeds", "asparagus_seeds", "onion_seeds", "garlic_seeds", "pepper_seeds", "椭圆形种子", "簇状种子", "尖种子", "漩涡种子", "脆籽荚", "风刮来的种子", "球茎状种子", "方形种子", "带刺的种子", "毛茸茸的种子", "筒状种子", "尖形种子", "种子荚", "块状种子", "胡萝卜种子", "玉米种子", "南瓜种子", "茄子种子", "榴莲种子", "石榴种子", "火龙果种子", "西瓜种子", "番茄种子", "土豆种子", "芦笋种子", "洋葱种子", "大蒜种子", "辣椒种子", "农田种子", "作物种子"]
tags: [prefab, item, food]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/veggies.lua", "prefabs/farm_plant_defs.lua", "prefabs/farm_plants.lua", "prefabs/farm_soil.lua", "prefabs/seeds.lua", "prefabs/birdcage.lua", "prefabs/fruitfly.lua", "prefabs/catcoon.lua", "prefabs/bundle.lua", "prefabs/wormwood.lua", "prefabs/oceanfishingrod.lua", "components/farmplantable.lua", "components/farmtiller.lua", "components/deployable.lua", "components/plantable.lua", "components/grower.lua", "components/cookable.lua", "components/cooker.lua", "components/edible.lua", "components/perishable.lua", "components/stackable.lua", "components/bait.lua", "components/trap.lua", "components/oceanfishingtackle.lua", "components/inventoryitem_replica.lua", "actions.lua", "componentactions.lua", "containers.lua", "standardcomponents.lua", "util.lua", "entityscript.lua", "tuning.lua", "constants.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 农田作物专用种子（14 个 <作物>_seeds）

## 快速判断

- 职责与适用：`MakeVeggie(name, has_seeds)` 在 `has_seeds` 为真时额外导出一件 `<name>_seeds`，工厂是独立的 `fn_seeds`。本页覆盖 14 件（名单见[十四注册与名称](#十四注册与名称)）。可借鉴：一件物品兼具播种、烤制、诱饵、拟饵四职。[种子工厂](../../../raw/current/scripts/prefabs/veggies.lua#L453-L537)、[SEEDLESS](../../../raw/current/scripts/prefabs/veggies.lua#L160-L168)
- 优先使用：改数值看[种子数值与堆叠](#种子数值与堆叠)；改「种下去长成什么」看[播种与种植路径](#播种与种植路径)；烤制与拟饵看[烤制与钓具](#烤制与钓具)。共用工厂的 `MakeVegStats` 位序、导出顺序与端分界见 [[reference/prefabs/farm-crops-staple#共用工厂与生熟两态]]，本页不重复。
- 重要副作用：14 种种子烤制后都是同一件 `seeds_cooked`，作物身份不保留；`PLANTSOIL` 先移除目标土堆，再生成植株，动作仅在 `Plant` 正常返回失败时尝试交回种子。[烤制](#烤制与钓具)、[Plant](../../../raw/current/scripts/components/farmplantable.lua#L7-L25)、[动作回交](../../../raw/current/scripts/actions.lua#L4727-L4745)
- 修改边界：通用种子 `seeds`／`seeds_cooked`／`seeds_placer` 见 [[reference/prefabs/generic-seeds]]；6 个 `SEEDLESS` 作物不产种子；`_oversized` 三件见 [[reference/prefabs/giant-crops]]；田间生长、压力与再生见 [[reference/components/crop]]、[[reference/components/farming-manager]]。本页只讲物品形态与其直接接线。

## 注册与构造

注册循环对 `VEGGIES` 每项调 `MakeVeggie(name, not SEEDLESS[name])`，14 个可种作物的导出序列首位是 `<name>_seeds`；`SEEDLESS` 的 6 项（`berries`、`cave_banana`、`cactus_meat`、`berries_juicy`、`fig`、`kelp`）没有种子与巨型导出，其中 `kelp` 另导出干制态 `kelp_dried`。[循环](../../../raw/current/scripts/prefabs/veggies.lua#L1102-L1110)、[导出段](../../../raw/current/scripts/prefabs/veggies.lua#L1084-L1099)

### 十四注册与名称

播种产物由 `farmplantable.plant` 写死为 `"farm_plant_"..name`；简中名分两档：未进植物图鉴时按外形命名，学会种子与植株名后才显示作物名（见[名称与显示](#名称与显示)）。

| 注册名 | 简中显示名（未识别 → 识别后） | 播种产物 |
|---|---|---|
| `carrot_seeds` | [椭圆形种子](../../../raw/current/scripts/languages/chinese_s.po#L340680-L340682) → 胡萝卜种子 | `farm_plant_carrot` |
| `corn_seeds` | [簇状种子](../../../raw/current/scripts/languages/chinese_s.po#L341710-L341712) → 玉米种子 | `farm_plant_corn` |
| `pumpkin_seeds` | [尖种子](../../../raw/current/scripts/languages/chinese_s.po#L347365-L347367) → 南瓜种子 | `farm_plant_pumpkin` |
| `eggplant_seeds` | [漩涡种子](../../../raw/current/scripts/languages/chinese_s.po#L342470-L342472) → 茄子种子 | `farm_plant_eggplant` |
| `durian_seeds` | [脆籽荚](../../../raw/current/scripts/languages/chinese_s.po#L342415-L342417) → 榴莲种子 | `farm_plant_durian` |
| `pomegranate_seeds` | [风刮来的种子](../../../raw/current/scripts/languages/chinese_s.po#L347140-L347142) → 石榴种子 | `farm_plant_pomegranate` |
| `dragonfruit_seeds` | [球茎状种子](../../../raw/current/scripts/languages/chinese_s.po#L342235-L342237) → 火龙果种子 | `farm_plant_dragonfruit` |
| `watermelon_seeds` | [方形种子](../../../raw/current/scripts/languages/chinese_s.po#L351830-L351832) → 西瓜种子 | `farm_plant_watermelon` |
| `tomato_seeds` | [带刺的种子](../../../raw/current/scripts/languages/chinese_s.po#L350465-L350467) → 番茄种子 | `farm_plant_tomato` |
| `potato_seeds` | [毛茸茸的种子](../../../raw/current/scripts/languages/chinese_s.po#L347260-L347262) → 土豆种子 | `farm_plant_potato` |
| `asparagus_seeds` | [筒状种子](../../../raw/current/scripts/languages/chinese_s.po#L338915-L338917) → 芦笋种子 | `farm_plant_asparagus` |
| `onion_seeds` | [尖形种子](../../../raw/current/scripts/languages/chinese_s.po#L346660-L346662) → 洋葱种子 | `farm_plant_onion` |
| `garlic_seeds` | [种子荚](../../../raw/current/scripts/languages/chinese_s.po#L343130-L343132) → 大蒜种子 | `farm_plant_garlic` |
| `pepper_seeds` | [块状种子](../../../raw/current/scripts/languages/chinese_s.po#L346795-L346797) → 辣椒种子 | `farm_plant_pepper` |

### fn_seeds 的构造顺序

| 阶段 | 关键内容 | 证据 |
|---|---|---|
| 公共端外形 | bank／build 都是 `farm_plant_seeds`，`PlayAnimation(name)`（动画名＝作物名），`SetRayTestOnBB(true)`，`scrapbook_anim = name`，`pickupsound = "vegetation_firm"` | [外形](../../../raw/current/scripts/prefabs/veggies.lua#L462-L468) |
| 公共端 tag | `cookable`、`deployedplant`、`deployedfarmplant`、`oceanfishing_lure` 四个 tag 在 `SetPristine` 前加好，客户端据此识别可烤／可钓 | [tags](../../../raw/current/scripts/prefabs/veggies.lua#L470-L474) |
| 公共端接线 | `overridedeployplacername = "seeds_placer"`（否则默认名会拼成 `<prefab>_placer`）、`plant_def = PLANT_DEFS[name]`、`displaynamefn = Seed_GetDisplayName`、`_custom_candeploy_fn = can_plant_seed`、`MakeInventoryFloatable(inst)` 无参 | [接线](../../../raw/current/scripts/prefabs/veggies.lua#L476-L483)、[放置器名](../../../raw/current/scripts/components/inventoryitem_replica.lua#L360-L369) |
| 端分界 | `SetPristine()` 后 `if not TheWorld.ismastersim then return inst end`，此后全部只在主模拟执行 | [分界](../../../raw/current/scripts/prefabs/veggies.lua#L485-L489) |
| 主模拟组件 | `edible`（`FOODTYPE.SEEDS`）→ `stackable` → `tradable` → `inspectable` → `inventoryitem` | [组件](../../../raw/current/scripts/prefabs/veggies.lua#L491-L499) |
| 主模拟数值 | `healthvalue = TUNING.HEALING_TINY / 2`、`hungervalue = TUNING.CALORIES_TINY`；`perishable` 40 天＋`StartPerishing`，腐坏换 `spoiled_food`；`cookable.product = "seeds_cooked"` | [数值与保质](../../../raw/current/scripts/prefabs/veggies.lua#L501-L510) |
| 主模拟其余 | `bait`、`farmplantable.plant`、已废弃的 `plantable`、`deployable`（CUSTOM／`restrictedtag = "plantkin"`）、`oceanfishingtackle` 拟饵、`MakeSmallBurnable`／`MakeSmallPropagator`／`MakeHauntableLaunchAndPerish` | [其余](../../../raw/current/scripts/prefabs/veggies.lua#L512-L534) |

### 种子数值与堆叠

时间基准 `total_day_time`＝`seg_time`(30)×16＝480 秒、`perish_warp`＝1，`calories_per_day`＝75，故天与秒可直接换算；燃烧与漂浮走 helper 默认值，见[关联机制导航](#关联机制导航)。[基准](../../../raw/current/scripts/tuning.lua#L19-L20)、[热量与下滑系数](../../../raw/current/scripts/tuning.lua#L41-L47)

| 项 | 值 | 来源与说明 |
|---|---|---|
| 饥饿 | `CALORIES_TINY`＝`calories_per_day/8`＝75/8＝**9.375** | [常量](../../../raw/current/scripts/tuning.lua#L2058)、[赋值](../../../raw/current/scripts/prefabs/veggies.lua#L502) |
| 生命 | `HEALING_TINY / 2`＝1/2＝**0.5** | [常量](../../../raw/current/scripts/tuning.lua#L2021)、[赋值](../../../raw/current/scripts/prefabs/veggies.lua#L501) |
| 理智 | 未赋值 → `edible` 默认 **0**；健康与理智都非负，不会被 `badfood` 判定命中 | [默认](../../../raw/current/scripts/components/edible.lua#L19-L25)、[判定](../../../raw/current/scripts/components/edible.lua#L1-L7) |
| 食性 | `FOODTYPE.SEEDS` → `edible_SEEDS` tag | [常量](../../../raw/current/scripts/constants.lua#L2026-L2035)、[赋值](../../../raw/current/scripts/prefabs/veggies.lua#L492)、[hook](../../../raw/current/scripts/components/edible.lua#L9-L17) |
| 堆叠 | `STACK_SIZE_SMALLITEM`＝**40**（组件默认是 `STACK_SIZE_MEDITEM`＝20） | [档位](../../../raw/current/scripts/tuning.lua#L81-L83)、[赋值](../../../raw/current/scripts/prefabs/veggies.lua#L494-L495)、[默认](../../../raw/current/scripts/components/stackable.lua#L23-L29) |
| 保质 | `PERISH_SUPERSLOW`＝40×480＝**19200 秒**（40 天） | [常量](../../../raw/current/scripts/tuning.lua#L2051)、[赋值](../../../raw/current/scripts/prefabs/veggies.lua#L505-L506) |
| 腐坏 | 换成 `spoiled_food`，并按原叠数生成同样的堆（超过上限才 `SetIgnoreMaxSize`） | [Perish](../../../raw/current/scripts/components/perishable.lua#L293-L302) |
| 抽取权重 | `seed_weight` 取 `MakeVegStats` 第 1 位（`COMMON` 2.5／`UNCOMMON` 1／`RARE` 0.5），只决定随机种子长成哪种作物，不改本件种子掉落 | [常量](../../../raw/current/scripts/tuning.lua#L97-L102) |

### 名称与显示

`Seed_GetDisplayName` 用 `plant_def.product`（作物名）作为图鉴键：`ThePlantRegistry:KnowsSeed` 与 `KnowsPlantName` 同时为真时返回 `STRINGS.NAMES["KNOWN_"..upper(prefab)]`，否则返回 `nil`；此时 `GetBasicDisplayName` 回落到实体自带的 `self.name`，而 `self.name` 在 `SetPrefabName` 时已由 `STRINGS.NAMES[upper(prefab)]` 填好。[显示名函数](../../../raw/current/scripts/prefabs/veggies.lua#L266-L272)、[回落](../../../raw/current/scripts/entityscript.lua#L687-L692)、[命名](../../../raw/current/scripts/entityscript.lua#L840-L844)

14 个 `KNOWN_*_SEEDS` 键集中在简中 `.po` 的 `KNOWN_` 段（同段另有 `KNOWN_TURNIP_SEEDS`），值即「作物名＋种子」；植株侧另用 `STRINGS.NAMES.FARM_PLANT_SEED`＝「种下的{seed}」拼名。[KNOWN 段](../../../raw/current/scripts/languages/chinese_s.po#L344345-L344417)、[植株拼名](../../../raw/current/scripts/prefabs/farm_plants.lua#L663-L668)、[FARM_PLANT_SEED](../../../raw/current/scripts/languages/chinese_s.po#L342615-L342617)

### 获得与掉种

| 路径 | 产出 | 证据 |
|---|---|---|
| 农田成熟收获 | 按最终压力：`NONE`／`LOW` 给产物×1＋种子×2，`MODERATE` 给产物×1＋种子×1，`HIGH` 只给产物；巨型植株只给 `_oversized`，非巨型腐烂株只给 `spoiled_food` | [SetupLoot](../../../raw/current/scripts/prefabs/farm_plants.lua#L344-L362) |
| 巨型敲碎 | `{作物, 作物, 种子, 种子, 75% 作物／否则种子}` 共 5 件 | [掉落表](../../../raw/current/scripts/prefabs/veggies.lua#L231-L235) |
| 巨型腐烂件 | `loot_oversized_rot`＝3×`spoiled_food`＋1×种子＋2×`fruitfly` | [字段](../../../raw/current/scripts/prefabs/farm_plant_defs.lua#L270) |
| 笼中鸟 | 只对普通鸟：喂任一「存在 `<prefab>_seeds` 注册」的可食物（含 14 种作物）→ 掉 1 件该种子；肉质另给蛋（`bird_mutant` 对非肉食物改掉 `spoiled_food`，`bird_mutant_rift` 只收月亮碎片类） | [DigestFood](../../../raw/current/scripts/prefabs/birdcage.lua#L106-L147) |
| 果蝇王尸体 | 场上**没有**友好果蝇时掉 1×`fruitflyfruit`；**已有**友好果蝇时改为 4 轮随机作物种子（每轮 1.0 必掉＋0.25 追加） | [战利品](../../../raw/current/scripts/prefabs/fruitfly.lua#L110-L120) |
| 浣猫（`catcoon`）礼物 | 建立随从关系后按 tier 抽：tier1 为 `carrot`／`corn`／`potato`／`tomato` 种子，tier4 为其余 10 种 | [礼物表](../../../raw/current/scripts/prefabs/catcoon.lua#L121-L157)、[抽取](../../../raw/current/scripts/prefabs/catcoon.lua#L265-L271) |
| 活动种子包 | `yotc_seedpacket` 家族 `lootfn` 抽 set1／set2；`carnival_seedpacket` 固定 4 件 `corn_seeds`，幸运再＋1 | [包内容](../../../raw/current/scripts/prefabs/bundle.lua#L442-L461)、[lootfn](../../../raw/current/scripts/prefabs/bundle.lua#L529-L582) |

## 播种与种植路径

三条链都从「种子＋可耕土地」出发，但前置、产物与门禁不同。

| 链 | 前置与门禁 | 产物 | 证据 |
|---|---|---|---|
| `farmplantable`（主路径） | 目标需带 `soil` tag（`farm_soil`，由锄头 `farmtiller:Till` 在 `CanTillSoilAtPoint` 处生成）；`componentactions` 只在 `target:HasTag("soil") and not target:HasTag("NOCLICK")` 时给 `ACTIONS.PLANTSOIL` | `FarmPlantable:Plant` 取 `FunctionOrValue(self.plant)`（[定义](../../../raw/current/scripts/util.lua#L1718-L1724)），**先 `target:Remove()`（耕地被吃掉；`plant_prefab` 为 nil 时不进这一步）**，再 `SpawnPrefab("farm_plant_<name>")`、`PushEvent("on_planted", {doer, seed, in_soil = true})` 并推 `itemplanted` | [Till](../../../raw/current/scripts/components/farmtiller.lua#L5-L16)、[候选](../../../raw/current/scripts/componentactions.lua#L1235-L1239)、[函数](../../../raw/current/scripts/actions.lua#L4727-L4745)、[Plant](../../../raw/current/scripts/components/farmplantable.lua#L7-L30) |
| Wormwood 部署（`DEPLOY`） | `deployable` 是 `DEPLOYMODE.CUSTOM`，`_custom_candeploy_fn` 只查 `CanTillSoilAtPoint`（**无需预先锄地**，不要求目标已有 `farm_soil`）；`restrictedtag = "plantkin"`，快照内只有 `wormwood` 加该 tag | `OnDeploy` 直接 `SpawnPrefab(farmplantable.plant)`、`PushEvent("on_planted", {in_soil = false})`、`CollapseSoilAtPoint`（把点附近 `soil` 实体推成 `collapsesoil`／`breaksoil`，即处理土堆）、移除种子；`deployedplant` tag 让 `Deployable:Deploy` 追加 `itemplanted` | [接线](../../../raw/current/scripts/prefabs/veggies.lua#L523-L526)、[门禁](../../../raw/current/scripts/components/deployable.lua#L89-L102)、[部署](../../../raw/current/scripts/components/deployable.lua#L132-L147)、[OnDeploy](../../../raw/current/scripts/prefabs/veggies.lua#L187-L202)、[tag](../../../raw/current/scripts/prefabs/wormwood.lua#L754) |
| 旧农田（`plantable`，已废弃） | `plantable` 只有 `growtime`／`product` 两个字段、没有行为；消费者是旧农田 `slow_farmplot`／`fast_farmplot` 的 `grower:PlantItem` | `crop:StartGrowing(product, growtime×growrate)` 长出 `plant_normal`；`product = name` 是**作物本体名**，不是 `farm_plant_<name>` | [组件](../../../raw/current/scripts/components/plantable.lua#L1-L7)、[赋值](../../../raw/current/scripts/prefabs/veggies.lua#L517-L520)、[消费者](../../../raw/current/scripts/components/grower.lua#L110-L131)、[动作](../../../raw/current/scripts/actions.lua#L2632-L2649) |

- 播种成功才移除种子；`ACTIONS.PLANTSOIL.fn` 在 `Plant` 返回假时尝试用 `GiveItem` 交回种子。
- 植株侧 `on_planted` 只读 `doer`：`doer` 带 `plantkin`（Wormwood）时置 `long_life`，据 `GetSpoilTime` 乘 `FARM_PLANT_LONG_LIFE_MULT` 延长成熟株的保鲜窗口；再生时长由 `GetSelfRegrowTime` 单独计算，不吃该倍率。`in_soil` 字段在农田植株上**没有消费者**，只有 `weed_plants` 用它。[监听](../../../raw/current/scripts/prefabs/farm_plants.lua#L693-L697)、[时长](../../../raw/current/scripts/prefabs/farm_plants.lua#L386-L388)

## 烤制与钓具

- **烤制**：14 件种子都设 `cookable.product = "seeds_cooked"`，烤出来是同一件通用烤种子，作物身份不保留；`Cook` 在生成熟件后把「已损失的新鲜度打五折」写入（`1 - (1-p)*0.5`，按熟件自己的上限算），客户端只靠公共端 `cookable` tag 识别可烤。[接线](../../../raw/current/scripts/prefabs/veggies.lua#L509-L510)、[Cook](../../../raw/current/scripts/components/cookable.lua#L18-L38)
- **一次一件**：`ACTIONS.COOK` 对堆叠只取 1 件（`stackable:Get()`），一叠 40 个要点 40 次；`Cooker:CookItem` 成功后移除原件。[取件](../../../raw/current/scripts/actions.lua#L2067-L2073)、[CookItem](../../../raw/current/scripts/components/cooker.lua#L25-L49)
- **陷阱诱饵**：`bait` 无参挂载，`Trap:SetBait` 只要求 `item.components.bait`；陷阱被拿／移除时 `bait.lua` 的 `onremove`／`onpickup` 清掉 `trap` 指针。种子没有额外诱饵标签，适用性由目标吃不吃决定。[挂载](../../../raw/current/scripts/prefabs/veggies.lua#L512)、[SetBait](../../../raw/current/scripts/components/trap.lua#L412-L424)、[清理](../../../raw/current/scripts/components/bait.lua#L1-L36)
- **海钓拟饵**：`oceanfishingtackle:SetupLure` 写 `lure_data` 与 `lure_setup`；`TUNING.OCEANFISHING_LURE.SEED` 为 `charm` 0.2、`reel_charm` −0.3、`radius` 3.0、`dist_max` 1、`style` `"seed"`。竿的容器只接受带 `oceanfishing_lure` tag 的物品进 2 号槽且 `acceptsstacks = false`（一次一件）；`single_use = true`，勾到东西时该槽对象被 `Remove()`。[SetupLure](../../../raw/current/scripts/prefabs/veggies.lua#L528-L529)、[参数](../../../raw/current/scripts/tuning.lua#L298-L299)、[竿槽](../../../raw/current/scripts/containers.lua#L1342-L1346)、[消耗](../../../raw/current/scripts/prefabs/oceanfishingrod.lua#L118-L136)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：farmplantable | `plant` 存播种目标，`Plant` 以 `soil` tag 为门禁并用 `FunctionOrValue` 解析 | 主模拟；14 键都是字符串 | [[reference/components/farmplantable#关键接口]]；[赋值](../../../raw/current/scripts/prefabs/veggies.lua#L514-L515) |
| 组件：deployable | CUSTOM 模式＋自定义判定，`restrictedtag = "plantkin"`，`ondeploy = OnDeploy` | 仅 Wormwood 能过 `IsDeployable` | [[reference/components/deployable#关键接口]]；[门禁](../../../raw/current/scripts/components/deployable.lua#L89-L102) |
| 组件：cookable | `product = "seeds_cooked"`，`Cook` 结算新鲜度 | 生件；客户端靠 pristine tag | [[reference/components/cookable#关键接口]]；[接线](../../../raw/current/scripts/prefabs/veggies.lua#L509-L510) |
| 组件：edible | `FOODTYPE.SEEDS`，生命 0.5／饥饿 9.375／理智 0 | 主模拟；赋值即打 `edible_SEEDS` | [[reference/components/edible#关键接口]]；[赋值](../../../raw/current/scripts/prefabs/veggies.lua#L501-L502) |
| 组件：perishable | 40 天＋`StartPerishing`，腐坏换 `spoiled_food` | 主模拟；整叠按原叠数替换 | [[reference/components/perishable#关键接口]]；[赋值](../../../raw/current/scripts/prefabs/veggies.lua#L504-L507) |
| 组件：stackable | 显式 40，覆盖组件默认 20 | 14 键一致 | [[reference/components/stackable#关键接口]]；[赋值](../../../raw/current/scripts/prefabs/veggies.lua#L494-L495) |
| 组件：inventoryitem／inspectable／floater | 入栏与拾取落地；`displaynamefn` 走图鉴；`MakeInventoryFloatable` 无参（size `"small"`） | `pickupsound` 只在公共端设一次 | [[reference/components/inventoryitem#关键接口]]、[[reference/components/inspectable#关键接口]]、[[reference/components/floater#关键接口]]；[接线](../../../raw/current/scripts/prefabs/veggies.lua#L478-L483) |
| 组件：bait／oceanfishingtackle | 陷阱诱饵与海钓拟饵两用 | 拟饵数据写死，不读 `VEGGIES` 表 | [[reference/components/bait#关键接口]]、[[reference/components/oceanfishingtackle#关键接口]]；[接线](../../../raw/current/scripts/prefabs/veggies.lua#L528-L529) |
| 组件：burnable／propagator／hauntable | `MakeSmallBurnable`（默认 10 秒）＋`MakeSmallPropagator`＋`MakeHauntableLaunchAndPerish` | 主模拟 | [[reference/components/burnable#关键接口]]、[[reference/components/hauntable#关键接口]]；[helper](../../../raw/current/scripts/standardcomponents.lua#L164-L174) |
| API／数据：`PLANT_DEFS` | 种子读 `product`（图鉴键）与 `plantregistryinfo`；植株侧由它填 `seed`／`product`／`loot_oversized_rot` | 主模拟 | [填充](../../../raw/current/scripts/prefabs/farm_plant_defs.lua#L254-L270) |
| 关联 Prefab：`farm_plant_<name>` | 播种产物；`MakePlant` 的 `prefabs` 依赖含 `plant_def.seed`，形成种子↔植株互相引用 | 14 键各一 | [依赖](../../../raw/current/scripts/prefabs/farm_plants.lua#L777-L790) |
| 关联 Prefab：`seeds_cooked` | 烤制产物，14 键共用一个注册 | 见 [[reference/prefabs/generic-seeds]] | [接线](../../../raw/current/scripts/prefabs/veggies.lua#L510) |
| 资源：anim | `anim/seeds.zip`＋`anim/farm_plant_seeds.zip`，动画名即作物名 | 14 键共用；工厂未声明 `INV_IMAGE` | [assets](../../../raw/current/scripts/prefabs/veggies.lua#L170-L180) |

## 典型接入

### 统一 14 件种子的堆叠与保质

- 前提与选择：服务端 `modmain.lua`；`fn_seeds` 是单一工厂，改数值要对每个注册分别 `AddPrefabPostInit`，不能只改 `VEGGIES` 表（表里没有种子字段）。postinit 运行时物品满鲜，`SetNewMaxPerishTime` 与 `SetPerishTime` 结果相同；要按比例改已有剩余才必须用前者。
- 生效与撤销：postinit 在每次构造时运行，旧存档里的种子读档时按存档剩余秒数走 `Perishable:OnLoad`；停用模组即恢复 40 天与 40 堆叠。[OnLoad](../../../raw/current/scripts/components/perishable.lua#L337-L349)
- 原版依据：`stackable` 显式 40、`perishable` 用 `PERISH_SUPERSLOW`，都在 `SetPristine` 之后的主模拟段。[赋值](../../../raw/current/scripts/prefabs/veggies.lua#L494-L506)

```lua
-- modmain.lua：14 件作物种子统一 40 堆叠与 80 天保质（原 40 天）
local SEED_NAMES = {
    "carrot_seeds", "corn_seeds", "pumpkin_seeds", "eggplant_seeds",
    "durian_seeds", "pomegranate_seeds", "dragonfruit_seeds", "watermelon_seeds",
    "tomato_seeds", "potato_seeds", "asparagus_seeds", "onion_seeds",
    "garlic_seeds", "pepper_seeds",
}

for _, name in ipairs(SEED_NAMES) do
    AddPrefabPostInit(name, function(inst)
        if not GLOBAL.TheWorld.ismastersim then
            return
        end
        local stackable = inst.components.stackable
        if stackable ~= nil then
            stackable.maxsize = GLOBAL.TUNING.STACK_SIZE_SMALLITEM
        end
        local perishable = inst.components.perishable
        if perishable ~= nil then
            perishable:SetNewMaxPerishTime(2 * GLOBAL.TUNING.PERISH_SUPERSLOW)
        end
    end)
end
```

- 调整与验证：单位是个与秒；`maxsize` 影响新构造实例，`SetNewMaxPerishTime` 保留当前百分比（此处仍是满鲜）。`farmplantable.plant` 改成函数只对 `PLANTSOIL` 路径生效（`Plant` 用 `FunctionOrValue` 解析）；Wormwood 直接部署时 `OnDeploy` 把该字段原样交给 `SpawnPrefab`，函数会报错，需同步改部署回调。

## 常见陷阱与限制

- **边界：种子烤了就不再是种子**：14 件种子的 `cookable.product` 都指向 `seeds_cooked`，烤制会丢掉作物身份；想留种只能不烤。[接线](../../../raw/current/scripts/prefabs/veggies.lua#L509-L510)
- **误区：任何角色都能右键种下**：`deployable.restrictedtag = "plantkin"`，`IsDeployable` 对无该 tag 的角色返回假；除 Wormwood 外只能走 `farmplantable`／`PLANTSOIL`。[门禁](../../../raw/current/scripts/components/deployable.lua#L89-L102)
- **边界：`plantable` 与 `farmplantable` 是两条链**：前者 `product = name`（作物本体名，配旧农田 `grower`），后者 `plant = "farm_plant_"..name`（植株）；改错字段不改变播种结果。[赋值对照](../../../raw/current/scripts/prefabs/veggies.lua#L514-L520)
- **边界：播种会消耗耕地**：`FarmPlantable:Plant` 先 `target:Remove()` 再生成植株，`on_planted` 里的 `in_soil = true` 在农田植株上没有消费者（只有杂草用）。[Plant](../../../raw/current/scripts/components/farmplantable.lua#L7-L30)、[监听](../../../raw/current/scripts/prefabs/farm_plants.lua#L693-L697)
- **边界：喂种子给笼中鸟换不出种子**：`DigestFood` 会拼 `<prefab>_seeds`，`carrot_seeds` 拼出 `carrot_seeds_seeds` 不存在，只落到 `FEED_BIRD_POOP_CHANCE`＝0.33（幸运修正前基值）的鸟粪分支；喂**作物本体**才得到该作物种子。这条路径只对普通鸟成立，变异鸟对非肉食物直接给 `spoiled_food`。[DigestFood](../../../raw/current/scripts/prefabs/birdcage.lua#L132-L147)
- **未核验**：`quagmire` 分支只在生食物工厂 `fn` 内（`event_server_data("quagmire", "prefabs/veggies").master_postinit(inst)`），`fn_seeds` 没有这条调用；该节日数据不在本快照内，种子在 Gorge 下是否有额外处理未核验。[钩子](../../../raw/current/scripts/prefabs/veggies.lua#L683-L685)

## 验证记录

- 源码审阅：2026-11-08，Steam buildid 26335394。核对 `veggies.lua` 全文（`VEGGIES`／`SEEDLESS`、`fn_seeds` 全段、`Seed_GetDisplayName`、`OnDeploy`、`can_plant_seed`、巨型掉落表、导出循环）、`farm_plant_defs` 的字段填充与 `loot_oversized_rot`、`farm_plants` 的 `SetupLoot`／`on_planted`／`MakePlant`、`farm_soil`／`farmtiller`、`birdcage` 的 `DigestFood`、`fruitfly` 的 `pickseed` 与果蝇王战利品、`catcoon` 礼物表、`bundle` 种子包、`farmplantable`／`deployable`／`plantable`／`grower`／`cookable`／`cooker`／`edible`／`perishable`／`stackable`／`bait`／`trap`／`oceanfishingtackle`／`inventoryitem_replica`／`entityscript` 的相关分支、`actions` 的 `PLANTSOIL`／`DEPLOY`／`PLANT`／`COOK`、`componentactions` 候选、`containers` 的海钓竿过滤、`tuning` 的基准与 `CALORIES_*`／`PERISH_*`／`SEED_CHANCE_*`／`SEEDS_GROW_TIME`、`constants` 的 `FOODTYPE`、`util` 的 `FunctionOrValue`、简中 14 个种子名与 `KNOWN_*_SEEDS`。
  名称核验：`CARROT_SEEDS`=椭圆形种子、`CORN_SEEDS`=簇状种子、`PUMPKIN_SEEDS`=尖种子、`EGGPLANT_SEEDS`=漩涡种子、`DURIAN_SEEDS`=脆籽荚、`POMEGRANATE_SEEDS`=风刮来的种子、`DRAGONFRUIT_SEEDS`=球茎状种子、`WATERMELON_SEEDS`=方形种子、`TOMATO_SEEDS`=带刺的种子、`POTATO_SEEDS`=毛茸茸的种子、`ASPARAGUS_SEEDS`=筒状种子、`ONION_SEEDS`=尖形种子、`GARLIC_SEEDS`=种子荚、`PEPPER_SEEDS`=块状种子；识别后为「作物名＋种子」（见[十四注册与名称](#十四注册与名称)表内锚点）。
  未覆盖：鼹鼠丘／蝙蝠洞、南瓜炸弹等其它生成点只抽查未穷举；`_oversized*` 与打蜡、田间压力与再生、`quagmire` 节日数据、`seeds`／`seeds_cooked` 本体不在本页。
- 游戏运行：未运行。9.375／0.5／19200 秒与 10 秒燃烧均为 tuning 与组件默认推导（`CALORIES_TINY`＝75/8、`PERISH_SUPERSLOW`＝40×480），非运行测量。
- 结构检查：`validate_page.py` 通过；`validate_source_links.py --strict` 0 错 0 警；Lua 示例经 `parser_probe --syntax-only` 解析通过。

## 相关链接

- 同一工厂的生熟两态与共用工厂：[[reference/prefabs/farm-crops-staple]]、[[reference/prefabs/farm-crops-fruit]]、[[reference/prefabs/farm-crops-spice]]
- 通用种子与烤种子：[[reference/prefabs/generic-seeds]]
- 巨型形态与打蜡：[[reference/prefabs/giant-crops]]
- 种植与生长：[[reference/components/farmplantable]]、[[reference/components/crop]]、[[reference/components/farming-manager]]

返回 [[reference/prefabs/index|Prefab 索引]]。
