---
title: "通用种子与烤种子（seeds、seeds_cooked 与放置预览 seeds_placer）"
created: "2026-11-08"
updated: "2026-11-08"
type: prefab
summary: "prefabs/seeds.lua 的三件注册：seeds 是可烤、可钓、可当诱饵的随机种子，现代种植先生成 farm_plant_randomseed，出苗替换为作物或杂草，识别可提前定种；seeds_cooked 可食、可当雪人装饰，不可播种或作拟饵；seeds_placer 用 tile_outline 预览可耕地块。"
aliases: ["seeds", "seeds_cooked", "seeds_placer", "种子", "烤种子", "随机种子", "通用种子"]
tags: [prefab, item, food]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/seeds.lua", "prefabs/veggies.lua", "prefabs/farm_plants.lua", "prefabs/farm_plant_defs.lua", "prefabs/weed_defs.lua", "prefabs/birdcage.lua", "prefabs/birds.lua", "prefabs/fruitfly.lua", "prefabs/tumbleweed.lua", "prefabs/bundle.lua", "components/farmplantable.lua", "components/plantable.lua", "components/grower.lua", "components/cookable.lua", "components/cooker.lua", "components/edible.lua", "components/perishable.lua", "components/stackable.lua", "components/bait.lua", "components/floater.lua", "components/oceanfishingtackle.lua", "components/deployable.lua", "components/inventoryitem_replica.lua", "containers.lua", "prefabutil.lua", "util.lua", "standardcomponents.lua", "actions.lua", "componentactions.lua", "tuning.lua", "constants.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 通用种子与烤种子（seeds、seeds_cooked 与放置预览 seeds_placer）

## 快速判断

- 职责与适用：`prefabs/seeds.lua` 一次返回三件注册——`seeds`（通用种子，简中「种子」）、`seeds_cooked`（烤种子）、`seeds_placer`（部署预览）。经 `PLANTSOIL` 播种或 Wormwood 直接部署都会先生成 `farm_plant_randomseed`。出苗时替换为作物或杂草，识别可提前锁定结果。旧农田走 `plantable.product`。可借鉴：一件物品同时承担播种、烤制、拟饵与「随机结果延迟抽取」。[导出](../../../raw/current/scripts/prefabs/seeds.lua#L216-L218)、[随机株两段生长](../../../raw/current/scripts/prefabs/farm_plants.lua#L412-L436)
- 优先使用：改可食与保质看[数值、保质与烤制](#数值保质与烤制)；改「种出什么」看[随机种植与随机株](#随机种植与随机株)；作物专用种子看 [[reference/prefabs/farm-crop-seeds]]。
- 重要副作用：`PLANTSOIL` 播种会把脚下的 `farm_soil` 换成 `farm_plant_randomseed`（耕地被消耗；Wormwood 直接部署无需预先锄地，但同样会 `CollapseSoilAtPoint` 处理附近土堆）；随机株的 `lootdropper` 没有赋表，挖掉或烧掉**不返还种子**。[Plant](../../../raw/current/scripts/components/farmplantable.lua#L7-L30)、[SetupLoot](../../../raw/current/scripts/prefabs/farm_plants.lua#L344-L362)
- 修改边界：14 种作物专用种子、农田压力与再生、`farm_plant_randomseed` 之外的植株行为不在本页；`seeds_placer` 只写接线，预览用图标资源不属本页。

## 注册与构造

### 三件注册

| 注册名 | 工厂 | 差异与证据 |
|---|---|---|
| `seeds` | `common("idle", true, true)` → `raw()` | 两个 `true` 分别开启「可烤」与 `oceanfishing_lure` tag；`raw()` 再补 `bait`／`farmplantable`／`plantable`／`deployable`／`oceanfishingtackle`。[raw](../../../raw/current/scripts/prefabs/seeds.lua#L145-L176) |
| `seeds_cooked` | `common("cooked")` → `cooked()` | 没有 `cookable`／`bait`／`oceanfishingtackle`／`farmplantable`／`deployable`；只剩可食、保质、堆叠与 `snowmandecor`。[cooked](../../../raw/current/scripts/prefabs/seeds.lua#L178-L192) |
| `seeds_placer` | `MakePlacer("seeds_placer", "farm_soil", "farm_soil", "till_idle", …)` | `placer.onupdatetransform` 用 `tile_outline` 在可耕地块中心显示预览；`seeds` 没有 `overridedeployplacername`，靠默认 `<prefab>_placer` 正好命中它。[定义](../../../raw/current/scripts/prefabs/seeds.lua#L194-L214)、[签名](../../../raw/current/scripts/prefabutil.lua#L1-L1)、[取名](../../../raw/current/scripts/components/inventoryitem_replica.lua#L360-L369) |

### common() 的两端分界

| 位置 | 内容 | 证据 |
|---|---|---|
| 公共端外形 | bank／build 都是 `seeds`，动画名由参数给（`idle`／`cooked`），`SetRayTestOnBB(true)`，`pickupsound = "vegetation_firm"` | [外形](../../../raw/current/scripts/prefabs/seeds.lua#L73-L78) |
| 公共端 tag | `cookable` 为真才加 `deployedplant`／`deployedfarmplant`／`cookable`；`oceanfishing_lure` 为真才加同名 tag；随后统一 `MakeInventoryFloatable(inst)`（无参 → size `"small"`） | [tag 与漂浮](../../../raw/current/scripts/prefabs/seeds.lua#L80-L92) |
| 端分界 | `SetPristine()` 后 `if not TheWorld.ismastersim then return inst end`，组件全部只装在主模拟 | [分界](../../../raw/current/scripts/prefabs/seeds.lua#L94-L98) |
| 主模拟组件 | `inventoryitem` → `stackable` 40 → `edible`（`FOODTYPE.SEEDS`）→ 可选 `cookable`（`product = "seeds_cooked"`）→ `tradable`／`inspectable` | [组件](../../../raw/current/scripts/prefabs/seeds.lua#L100-L113) |
| 主模拟其余 | `MakeSmallBurnable(inst, SMALL_BURNTIME)`／`MakeSmallPropagator`／`MakeHauntableLaunchAndPerish` → `perishable` 40 天＋腐坏换 `spoiled_food` → `snowmandecor` → 图鉴裁剪 `scrapbook_removedeps`（6 个 `SEEDLESS` 作物） | [其余](../../../raw/current/scripts/prefabs/seeds.lua#L115-L126)、[裁剪表](../../../raw/current/scripts/prefabs/seeds.lua#L19-L27) |

- `raw()` 追加：`edible.healthvalue = 0`、`hungervalue = TUNING.CALORIES_TINY / 2`、`bait`、`farmplantable.plant = pickfarmplant`（**函数**）、`oceanfishingtackle` 的 SEED 拟饵、`deployable`（`DEPLOYMODE.CUSTOM`／`restrictedtag = "plantkin"`）、`plantable`（`growtime = TUNING.SEEDS_GROW_TIME`＝1800 秒／`product = pickproduct`）。[raw 段](../../../raw/current/scripts/prefabs/seeds.lua#L148-L173)、[SEEDS_GROW_TIME](../../../raw/current/scripts/tuning.lua#L1614)
- `cooked()` 追加：`inst.components.floater:SetScale(0.8)`（在端分界之前，双端都执行）、`healthvalue = TUNING.HEALING_TINY`＝1、`hungervalue = TUNING.CALORIES_TINY / 2`、`perishable:SetPerishTime(TUNING.PERISH_MED)` 覆盖 `common` 的 40 天。[cooked 段](../../../raw/current/scripts/prefabs/seeds.lua#L178-L192)、[SetScale](../../../raw/current/scripts/components/floater.lua#L68-L68)

### 数值、保质与烤制

时间基准 `total_day_time`＝`seg_time`(30)×16＝480 秒、`perish_warp`＝1，饥饿基准 `calories_per_day`＝75。[基准](../../../raw/current/scripts/tuning.lua#L19-L20)、[系数](../../../raw/current/scripts/tuning.lua#L41-L47)

| 项 | `seeds` | `seeds_cooked` | 说明与证据 |
|---|---|---|---|
| 饥饿 | **4.6875** | **4.6875** | 都是 `CALORIES_TINY / 2`＝(75/8)/2；作物专用种子是 9.375。[定义](../../../raw/current/scripts/tuning.lua#L2058)、[赋值](../../../raw/current/scripts/prefabs/seeds.lua#L154-L155) |
| 生命 | **0** | **1** | 烤后才给 `HEALING_TINY`。[常量](../../../raw/current/scripts/tuning.lua#L2021)、[赋值](../../../raw/current/scripts/prefabs/seeds.lua#L187-L188) |
| 理智 | 0 | 0 | 两者都未赋值，`edible` 默认 0，不会触发 `badfood`。[默认](../../../raw/current/scripts/components/edible.lua#L19-L25) |
| 食性 | `FOODTYPE.SEEDS` | `FOODTYPE.SEEDS` | 赋值即加 `edible_SEEDS` tag。[常量](../../../raw/current/scripts/constants.lua#L2026-L2035)、[hook](../../../raw/current/scripts/components/edible.lua#L9-L17) |
| 保质 | 40 天（19200 秒） | 10 天（4800 秒） | `PERISH_SUPERSLOW` 被 `PERISH_MED` 覆盖。[常量](../../../raw/current/scripts/tuning.lua#L2043-L2051)、[覆盖](../../../raw/current/scripts/prefabs/seeds.lua#L189) |
| 堆叠 | 40 | 40 | `STACK_SIZE_SMALLITEM`，与组件默认 20 不同。[档位](../../../raw/current/scripts/tuning.lua#L81-L83)、[赋值](../../../raw/current/scripts/prefabs/seeds.lua#L101-L102)、[默认](../../../raw/current/scripts/components/stackable.lua#L23-L29) |
| 燃烧 | 7.5 秒 | 7.5 秒 | 显式传 `TUNING.SMALL_BURNTIME`（`seg_time*.25`）。[常量](../../../raw/current/scripts/tuning.lua#L1681)、[helper](../../../raw/current/scripts/standardcomponents.lua#L164-L174) |
| 漂浮 | `"small"` | `"small"`＋`SetScale(0.8)` | 缩放写在 `cooked()` 里，双端生效。[helper](../../../raw/current/scripts/standardcomponents.lua#L1772-L1791) |
| 其它 | 可烤、可钓、可作诱饵、可播种 | 全无 | 见[三件注册](#三件注册) |

烤制链：`seeds` 的 `cookable.product = "seeds_cooked"`，`Cook` 生成熟件后把「已损失的新鲜度打五折」写入（`1 - (1-p)*0.5`），所以一包陈旧种子烤出来也按熟件自己的 10 天上限折算；`ACTIONS.COOK` 对堆叠只取 1 件，一叠 40 个要点 40 次。14 种作物专用种子也指向同一件 `seeds_cooked`，烤种子的「来源作物」不可追溯。[接线](../../../raw/current/scripts/prefabs/seeds.lua#L107-L110)、[Cook](../../../raw/current/scripts/components/cookable.lua#L18-L38)、[取件](../../../raw/current/scripts/actions.lua#L2067-L2073)

### 随机种植与随机株

- `farmplantable.plant` 存的是**函数** `pickfarmplant`，`FarmPlantable:Plant` 用 `FunctionOrValue(self.plant, self.inst)` 以种子实例调用，返回常量 `"farm_plant_randomseed"`；作物专用种子这里存的是字符串。[Plant](../../../raw/current/scripts/components/farmplantable.lua#L7-L21)、[函数](../../../raw/current/scripts/prefabs/seeds.lua#L60-L62)、[定义](../../../raw/current/scripts/util.lua#L1718-L1724)
- 该植株由 `PLANT_DEFS.randomseed` 生成：`build`／`bank` 都是 `farm_soil`，`seed = "seeds"`、`plant_type_tag = "farm_plant_randomseed"`、`family_min_count = 0`、`nutrient_consumption = {0,0,0}`、`is_randomseed = true`；没有 `product`／`product_oversized`／`loot_oversized_rot`。[定义](../../../raw/current/scripts/prefabs/farm_plant_defs.lua#L5-L5)、[随机分支](../../../raw/current/scripts/prefabs/farm_plant_defs.lua#L260-L263)、[零消耗](../../../raw/current/scripts/prefabs/farm_plant_defs.lua#L107-L107)
- 生长只有两段：`seed` 段发芽 `12–16 × SEG_TIME`＝360–480 秒，秋／春（`good_seasons`）减半为 180–240 秒；`sprout` 段的 `fn` 直接 `ReplaceWithPlant`，不再进入 `small`／`med`／`full`。[发芽时长](../../../raw/current/scripts/prefabs/farm_plant_defs.lua#L40-L40)、[阶段表](../../../raw/current/scripts/prefabs/farm_plants.lua#L412-L436)、[发芽计算](../../../raw/current/scripts/prefabs/farm_plants.lua#L364-L368)
- `ReplaceWithPlant`：优先用已识别的注册名，否则由此文件的 `pickfarmplant` 抽取，再 `SpawnPrefab`；复制 `farmsoildrinker`／`farmplantstress`、置 `no_oversized = true`、`long_life` 继承、`DoGrowth()` 与贴图覆盖都只在 `plant.plant_def ~= nil` 内执行，抽到杂草时只做生成、定位与替换。[替换](../../../raw/current/scripts/prefabs/farm_plants.lua#L151-L167)
- 抽取规则：先掷 `FARM_PLANT_RANDOMSEED_WEED_CHANCE`＝0.20 决定出杂草（按 `WEIGHTED_SEED_TABLE`：`weed_forgetmelots` 5、另三种各 0.5）；否则按 `VEGGIES[k].seed_weight × (PLANT_DEFS[k].good_seasons[season] and SEED_WEIGHT_SEASON_MOD or 1)` 抽作物。[抽取](../../../raw/current/scripts/prefabs/farm_plants.lua#L134-L149)、[杂草权重](../../../raw/current/scripts/prefabs/weed_defs.lua#L43-L46)、[权重表](../../../raw/current/scripts/prefabs/weed_defs.lua#L336-L341)
- 权重：`SEED_CHANCE_COMMON` 2.5（胡萝卜／玉米／番茄／土豆）、`UNCOMMON` 1（南瓜／茄子／西瓜／芦笋）、`RARE` 0.5（榴莲／石榴／火龙果／洋葱／大蒜／辣椒），14 项合计 17；`SEEDLESS` 的 6 项为 0，永不抽出；`SEED_WEIGHT_SEASON_MOD`＝2。[常量](../../../raw/current/scripts/tuning.lua#L97-L102)、[每键权重](../../../raw/current/scripts/prefabs/veggies.lua#L39-L155)
- 识别：带 `farmplantidentifier` 的玩家查看随机株时会经 `RandomSeedDescriptionFn` 调 `inst:BeIdentified(viewer)`，提前定种并推 `idplantseed`、去掉 `israndomseed` tag；本快照里 `BeIdentified` 只有这一处调用。存档里的 `identified_plant_type` 读档后同样去 tag。[定种](../../../raw/current/scripts/prefabs/farm_plants.lua#L700-L717)、[读档](../../../raw/current/scripts/prefabs/farm_plants.lua#L754-L757)
- 掉落：`lootdropper.lootsetupfn = SetupLoot`，而 `SetupLoot` 只在 `farm_plant_killjoy`（腐烂株）或 `pickable ~= nil`（成熟株）时赋表；随机株两段都不命中，`loot` 保持初始空表，挖掉与烧掉都不返还种子。[两段分支](../../../raw/current/scripts/prefabs/farm_plants.lua#L344-L362)、[挂载](../../../raw/current/scripts/prefabs/farm_plants.lua#L873-L874)

## 获得路径

| 路径 | 产出与条件 | 证据 |
|---|---|---|
| 鸟笼喂食 | 只适用于普通鸟：喂 `seeds` 或这批 14 种作物专用种子时 `DigestFood` 拼 `<prefab>_seeds` 找不到注册，落到 `FEED_BIRD_POOP_CHANCE`＝0.33 的 `guano`（幸运修正前的基值）；肉质另给蛋。`bird_mutant` 对非肉食物改掉 `spoiled_food`，`bird_mutant_rift` 只收月亮碎片类。可喂判定先过食性／注册，再排除 `invalid_foods`（蛋与 `woby_treat`） | [消化](../../../raw/current/scripts/prefabs/birdcage.lua#L132-L147)、[可喂判定](../../../raw/current/scripts/prefabs/birdcage.lua#L155-L181)、[拒收表](../../../raw/current/scripts/prefabs/birdcage.lua#L30-L39)、[概率](../../../raw/current/scripts/tuning.lua#L9350) |
| 小果蝇 | 死亡掉落 10% `seeds` | [战利品](../../../raw/current/scripts/prefabs/fruitfly.lua#L292-L295)、[挂载](../../../raw/current/scripts/prefabs/fruitfly.lua#L348-L349) |
| 风滚草 | 权重 `2` 的条目（表中干草与树枝各 40），每次风滚草按权重抽 3 件 | [条目](../../../raw/current/scripts/prefabs/tumbleweed.lua#L147-L156)、[抽取](../../../raw/current/scripts/prefabs/tumbleweed.lua#L213-L227) |
| 活动种子包 | `yotc_seedpacket` 的 `lootfn` 固定 2 件 `seeds`＋1 件 set1 随机种子 | [lootfn](../../../raw/current/scripts/prefabs/bundle.lua#L529-L537)、[注册](../../../raw/current/scripts/prefabs/bundle.lua#L693-L695) |
| 鸟类新手掉落 | 靠近低龄玩家的鸟会掉 `seeds`：`ChooseSeeds` 非冬季才返回，按玩家年龄与 `BIRD_DROP_SEEDS_CHANCE_*` 判定 | [ChooseSeeds](../../../raw/current/scripts/prefabs/birds.lua#L63-L65)、[分支](../../../raw/current/scripts/prefabs/birds.lua#L87-L103) |

- 种子袋 `seedpouch` 属收纳用途：`itemtestfn` 收 `seeds`、名字含 `_seeds` 的件与 `treeseed`，不产出种子，见[过滤](../../../raw/current/scripts/containers.lua#L1731-L1735)。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：cookable | `product = "seeds_cooked"`，只有 `seeds` 有 | 生件；客户端靠 pristine tag 识别 | [[reference/components/cookable#关键接口]]；[接线](../../../raw/current/scripts/prefabs/seeds.lua#L107-L110) |
| 组件：edible | `FOODTYPE.SEEDS`，饥饿 4.6875／生命 0 或 1 | 主模拟；赋值即打 `edible_SEEDS` | [[reference/components/edible#关键接口]]；[赋值](../../../raw/current/scripts/prefabs/seeds.lua#L154-L155) |
| 组件：perishable | 40 天（`seeds`）／10 天（`seeds_cooked`），腐坏换 `spoiled_food` | 主模拟；整叠按原叠数替换 | [[reference/components/perishable#关键接口]]；[赋值](../../../raw/current/scripts/prefabs/seeds.lua#L119-L122)、[Perish](../../../raw/current/scripts/components/perishable.lua#L293-L302) |
| 组件：stackable | 显式 40 | 三件里两件都有 | [[reference/components/stackable#关键接口]]；[赋值](../../../raw/current/scripts/prefabs/seeds.lua#L101-L102) |
| 组件：farmplantable | `plant` 存**函数** `pickfarmplant`，播种时求值 | 仅 `seeds`；主模拟 | [[reference/components/farmplantable#关键接口]]；[赋值](../../../raw/current/scripts/prefabs/seeds.lua#L159-L160) |
| 组件：deployable | CUSTOM 模式＋`_custom_candeploy_fn`，`restrictedtag = "plantkin"`，`ondeploy` 生成随机株 | 仅 Wormwood 能直接部署 | [[reference/components/deployable#关键接口]]；[接线](../../../raw/current/scripts/prefabs/seeds.lua#L148-L168)、[门禁](../../../raw/current/scripts/components/deployable.lua#L89-L102) |
| 组件：plantable（已废弃） | `growtime`＝1800 秒、`product = pickproduct`（按权重返回作物名），供旧农田 `grower` 读 | 仅旧农田 `slow_farmplot`／`fast_farmplot` 路径 | [[reference/components/plantable#关键接口]]；[赋值](../../../raw/current/scripts/prefabs/seeds.lua#L170-L173)、[消费者](../../../raw/current/scripts/components/grower.lua#L110-L131) |
| 组件：bait／oceanfishingtackle | 陷阱诱饵与海钓拟饵；拟饵为 `TUNING.OCEANFISHING_LURE.SEED`（charm 0.2／radius 3.0／dist_max 1） | 仅 `seeds`；竿 2 号槽按 `oceanfishing_lure` tag 收 | [[reference/components/bait#关键接口]]、[[reference/components/oceanfishingtackle#关键接口]]；[接线](../../../raw/current/scripts/prefabs/seeds.lua#L162-L163)、[参数](../../../raw/current/scripts/tuning.lua#L298-L299)、[竿槽](../../../raw/current/scripts/containers.lua#L1342-L1346) |
| 组件：snowmandecor | 空标记组件，行为在 `componentactions` 按 tag 接入雪人装饰 | 两件都有；14 种作物种子没有 | [[reference/components/snowmandecor]]；[挂载](../../../raw/current/scripts/prefabs/seeds.lua#L124) |
| 关联 Prefab：`farm_plant_randomseed` | 播种产物；两段生长后自我替换成真植株 | 由 `PLANT_DEFS.randomseed` 生成 | [两段生长](../../../raw/current/scripts/prefabs/farm_plants.lua#L412-L436) |
| 关联 Prefab：`seeds_placer` | 部署预览，`overridedeployplacername` 缺失时按 `<prefab>_placer` 命中 | 作物种子显式指向同一件 | [定义](../../../raw/current/scripts/prefabs/seeds.lua#L205-L214) |
| 关联 Prefab：`seeds_cooked` | 烤制产物 | 见[数值、保质与烤制](#数值保质与烤制) | [接线](../../../raw/current/scripts/prefabs/seeds.lua#L109) |
| API／数据：`VEGGIES.seed_weight`／`WEED_DEFS` | 随机株抽取用的权重来源 | 主模拟；活动季乘 `SEED_WEIGHT_SEASON_MOD` | [农田抽取](../../../raw/current/scripts/prefabs/farm_plants.lua#L134-L149) |
| 资源：anim | `anim/seeds.zip`＋`anim/oceanfishing_lure_mis.zip` | 两件物品共用 `seeds` bank／build | [assets](../../../raw/current/scripts/prefabs/seeds.lua#L4-L8) |

## 典型接入

### 把随机种子固定成指定作物（只改 `PLANTSOIL` 路径）

- 前提与选择：服务端 `modmain.lua`；`farmplantable.plant` 允许函数，`Plant` 会用 `FunctionOrValue(self.plant, self.inst)` 调用它，因此不必改动 `farm_plant_randomseed` 本体。该改法只影响 `PLANTSOIL` 播种：Wormwood 直接部署走 `OnDeploy` 里硬编码的 `SpawnPrefab("farm_plant_randomseed")`，旧农田读 `plantable.product`，两条路径都不受这个字段影响。
- 生效与撤销：postinit 只影响之后构造的种子；停用模组即恢复随机。已种下的随机株不会被追改。
- 原版依据：默认值是 `pickfarmplant`（返回常量 `farm_plant_randomseed`），未提前识别时才在出苗的 `ReplaceWithPlant` 中抽取。[默认](../../../raw/current/scripts/prefabs/seeds.lua#L60-L62)、[替换](../../../raw/current/scripts/prefabs/farm_plants.lua#L151-L167)

```lua
-- modmain.lua：让通用种子固定长出胡萝卜植株（原版是 farm_plant_randomseed）
AddPrefabPostInit("seeds", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    local farmplantable = inst.components.farmplantable
    if farmplantable ~= nil then
        farmplantable.plant = function(seed)
            return "farm_plant_carrot"
        end
    end
end)
```

- 调整与验证：把返回值换成任一同族植株注册名（`farm_plant_<作物>`）即可；目标植株必须已注册，否则 `SpawnPrefab` 返回 nil、随后的 `plant.Transform` 会直接报错。该做法同时绕开 `farm_plant_randomseed`，因此也不再走 20% 杂草分支。

## 常见陷阱与限制

- **误区：随机株能挖回种子**：`SetupLoot` 对随机株既没有 killjoy 分支也没有 `pickable`，`lootdropper` 保持空表；想回收只能等它出苗后按作物处理。[SetupLoot](../../../raw/current/scripts/prefabs/farm_plants.lua#L344-L362)
- **边界：`pickproduct` 的 `or 1` 对 0 权重不生效**：`v.seed_weight or 1` 中 `0` 在 Lua 里为真，所以 6 个 `SEEDLESS` 项权重是 0、不会被抽到（只有 `nil` 才回落 1，本表没有 `nil`）；该函数只服务已废弃的 `plantable`／旧农田。[实现](../../../raw/current/scripts/prefabs/seeds.lua#L43-L58)
- **边界：`seeds_cooked` 除了吃与装饰雪人没有别的用途**：没有 `cookable`、`farmplantable`、`deployable`、`bait` 与 `oceanfishing_lure` tag，既不能种也不能钓。[common](../../../raw/current/scripts/prefabs/seeds.lua#L64-L129)
- **边界：通用种子不是「弱化版作物种子」**：饥饿只有 4.6875（作物种子 9.375）、生命 0（作物种子 0.5），但两者烤制后都是同一件 `seeds_cooked`（生命 1）。[对照](../../../raw/current/scripts/prefabs/veggies.lua#L501-L502)
- **未核验**：`israndomseed` tag 在本快照只出现于 `farm_plants.lua` 的加／减三处，未找到其它消费者（检索范围：整个 `scripts` 根），UI 是否读取未核验；`seeds` 的物品图标来源与 `seeds_placer` 的资源文件也不在本页范围。

## 验证记录

- 源码审阅：2026-11-08，Steam buildid 26335394。核对 `seeds.lua` 全文（`common`／`raw`／`cooked`／`OnDeploy`／`can_plant_seed`／`pickproduct`／`pickfarmplant`／`seeds_placer` 与三个返回值）、`veggies.lua` 的 `VEGGIES` 权重与对照数值、`farm_plants.lua` 的 `pickfarmplant`／`ReplaceWithPlant`／`GROWTH_STAGES_RANDOMSEED`／`SetupLoot`／定种与读档、`farm_plant_defs.lua` 的 `randomseed` 字段、`weed_defs.lua` 的权重表、`birdcage.lua` 的 `DigestFood`／`ShouldAcceptItem`／`invalid_foods`、`birds.lua` 的 `ChooseSeeds`、`fruitfly.lua` 的小果蝇战利品、`tumbleweed.lua` 的权重与三件抽取、`bundle.lua` 的 `yotc_seedpacket`、`farmplantable`／`plantable`／`grower`／`cookable`／`cooker`／`edible`／`perishable`／`stackable`／`bait`／`floater`／`oceanfishingtackle`／`deployable` 的相关分支、`actions` 的 `PLANTSOIL`／`COOK`、`componentactions` 的候选、`containers` 的种子袋与海钓竿过滤、`prefabutil` 的 `MakePlacer` 签名、`util` 的 `FunctionOrValue`、`standardcomponents` 两个 helper、`tuning` 的基准与 `CALORIES_*`／`PERISH_*`／`SEED_CHANCE_*`／`SEED_WEIGHT_SEASON_MOD`／`FARM_PLANT_RANDOMSEED_WEED_CHANCE`／`SMALL_BURNTIME`／`SEEDS_GROW_TIME`、`constants` 的 `FOODTYPE`。
  名称核验：`NAMES.SEEDS`=种子、`NAMES.SEEDS_COOKED`=烤种子；植物侧拼名用 `NAMES.FARM_PLANT_SEED`=「种下的{seed}」。两件物品都没有 `displaynamefn`，显示名按注册名大写取值。[SEEDS](../../../raw/current/scripts/languages/chinese_s.po#L349085-L349087)、[SEEDS_COOKED](../../../raw/current/scripts/languages/chinese_s.po#L349090-L349092)、[FARM_PLANT_SEED](../../../raw/current/scripts/languages/chinese_s.po#L342615-L342617)
  未覆盖：`seeds_placer` 的图标与资源表现、鸟类新手掉落的完整幸运分支、`farm_plant_randomseed` 之外的田间生长与压力、14 种作物专用种子（见 [[reference/prefabs/farm-crop-seeds]]）。
- 游戏运行：未运行。4.6875／1／19200 秒／4800 秒／7.5 秒与 20% 杂草率均为常量与工厂推导，非运行测量。
- 结构检查：`validate_page.py` 通过；`validate_source_links.py --strict` 0 错 0 警；Lua 示例经 `parser_probe --syntax-only` 解析通过。

## 相关链接

- 作物专用种子（14 个 `<作物>_seeds`）：[[reference/prefabs/farm-crop-seeds]]
- 同一工厂的作物本体：[[reference/prefabs/farm-crops-staple]]、[[reference/prefabs/farm-crops-fruit]]、[[reference/prefabs/farm-crops-spice]]
- 种植与生长：[[reference/components/farmplantable]]、[[reference/components/crop]]、[[reference/components/farming-manager]]

返回 [[reference/prefabs/index|Prefab 索引]]。
