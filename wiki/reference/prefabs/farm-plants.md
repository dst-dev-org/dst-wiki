---
title: "农田植株（15 个 farm_plant_<作物> 注册）"
created: "2026-11-08"
updated: "2026-11-09"
type: prefab
summary: "farm_plants.lua 按 PLANT_DEFS 循环注册的 15 个世界植株：growable 六阶段与两阶段随机种、farmplantstress 七类压力与每阶段检查点、farmplanttendable 照料、成熟收获的产量分档与巨型判定、腐烂再生、铲除与烧毁，以及 PLANT_DEFS 给出的耗水／养分／季节／家族差异。"
aliases: ["farm_plant_carrot", "farm_plant_corn", "farm_plant_potato", "farm_plant_tomato", "farm_plant_asparagus", "farm_plant_eggplant", "farm_plant_pumpkin", "farm_plant_watermelon", "farm_plant_dragonfruit", "farm_plant_durian", "farm_plant_garlic", "farm_plant_onion", "farm_plant_pepper", "farm_plant_pomegranate", "farm_plant_randomseed", "农田植株", "作物植株", "胡萝卜植株", "南瓜植株", "种下的种子", "随机种子植株", "巨型作物植株"]
tags: [prefab]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/farm_plants.lua", "prefabs/farm_plant_defs.lua", "prefabs/veggies.lua", "prefabs/seeds.lua", "prefabs/farm_soil.lua", "components/farmplantable.lua", "components/farmplantstress.lua", "components/farmplanttendable.lua", "components/farmsoildrinker.lua", "components/growable.lua", "components/pickable.lua", "components/workable.lua", "components/lootdropper.lua", "components/farming_manager.lua", "actions.lua", "standardcomponents.lua", "mainfunctions.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 农田植株（15 个 farm_plant_<作物> 注册）

## 快速判断

- 职责与适用：`farm_plants.lua` 的 `MakePlant` 为 `PLANT_DEFS` 每个键产出一株世界植株，14 个作物加 `randomseed` 共 15 个 `farm_plant_<键>`。植株不是物品：收获、铲除、烧毁统一走 `lootdropper`。[工厂循环](../../../raw/current/scripts/prefabs/farm_plants.lua#L930-L937)、[键表](../../../raw/current/scripts/prefabs/farm_plant_defs.lua#L5-L19)
- 优先使用：改阶段时长看[阶段链与生长结算](#阶段链与生长结算)；改压力与照料看[七类压力与照料](#七类压力与照料)；改产量与巨型看[收获、腐烂与清除](#收获腐烂与清除)；逐作物差异看下文的[差异表](#逐作物差异表)。机制细节归 [[reference/components/farming-manager]]、[[reference/components/farmplantable]]、[[reference/components/farmplantstress]]，本页只记植株侧。
- 重要副作用：成熟株被采摘即删除植株，并在 `FARMING_SOIL` 地块补一个 `farm_soil` 再推 `breaksoil`，地块回到不可种植的塌陷态；不采摘则 `full → rotten` 腐烂，普通株腐烂后再循环回 `sprout`，巨型株停止生长。[OnPicked](../../../raw/current/scripts/prefabs/farm_plants.lua#L287-L301)、[rotten](../../../raw/current/scripts/prefabs/farm_plants.lua#L537-L566)
- 修改边界：14 种作物物品与 14 种专用种子的数值／获得见 [[reference/prefabs/farm-crop-seeds]]、[[reference/prefabs/farm-crops-staple]]、[[reference/prefabs/farm-crops-fruit]]、[[reference/prefabs/farm-crops-spice]]；巨型三件与敲碎掉种见 [[reference/prefabs/giant-crops]]；通用 `seeds`／`seeds_cooked`／`seeds_placer` 见 [[reference/prefabs/generic-seeds]]。

## 注册与构造

`PLANT_DEFS` 的 15 个键各对应一个注册；带 `data_only` 的键被跳过，其余每键调用一次 `MakePlant`。[循环](../../../raw/current/scripts/prefabs/farm_plants.lua#L930-L937)

| 注册名 | 简中实显 | build／bank 差异 |
|---|---|---|
| `farm_plant_carrot` | 胡萝卜植株 | 默认同名 |
| `farm_plant_corn` | 玉米杆 | build `farm_plant_corn_build`，bank `farm_plant_pumpkin` |
| `farm_plant_potato` | 土豆植株 | 默认同名 |
| `farm_plant_tomato` | 番茄植株 | 默认同名 |
| `farm_plant_asparagus` | 芦笋蕨 | 默认同名 |
| `farm_plant_eggplant` | 茄子杆 | build `farm_plant_eggplant_build`，bank `farm_plant_potato` |
| `farm_plant_pumpkin` | 南瓜植株 | 默认同名 |
| `farm_plant_watermelon` | 西瓜植株 | build `farm_plant_watermelon_build`，bank `farm_plant_tomato` |
| `farm_plant_dragonfruit` | 火龙果藤 | build `farm_plant_dragonfruit_build`，bank `farm_plant_potato` |
| `farm_plant_durian` | 榴莲藤 | build `farm_plant_durian_build`，bank `farm_plant_pepper` |
| `farm_plant_garlic` | 大蒜植株 | 默认同名 |
| `farm_plant_onion` | 洋葱植株 | build `farm_plant_onion_build`，bank `farm_plant_pumpkin` |
| `farm_plant_pepper` | 辣椒植株 | 默认同名 |
| `farm_plant_pomegranate` | 石榴枝 | build `farm_plant_pomegranate_build`，bank `farm_plant_potato` |
| `farm_plant_randomseed` | 种下的种子 | build／bank 都是 `farm_soil` |

build／bank 缺省时由 `data.prefab` 补齐为 `farm_plant_<键>`。[缺省填充](../../../raw/current/scripts/prefabs/farm_plant_defs.lua#L254-L258)、[键差异](../../../raw/current/scripts/prefabs/farm_plant_defs.lua#L5-L19)

### 共同构造与端分界

| 阶段 | 关键内容 | 证据 |
|---|---|---|
| 资源与依赖 | `anim/<bank>.zip`＋`anim/<build>.zip`＋`farm_soil.zip`，脚本依赖 `farm_plant_defs`／`weed_defs`；prefabs 依赖按 `plant_def` 追加产物、巨型件、种子与各阶段 `dig_fx` | [assets](../../../raw/current/scripts/prefabs/farm_plants.lua#L767-L790) |
| 公共端外形 | bank／build、`OverrideSymbol("soil01","farm_soil","soil01")`、`SetDeploySmartRadius(0.5)`、物理半径 `FARM_PLANT_PHYSICS_RADIUS`＝0.5 | [外形](../../../raw/current/scripts/prefabs/farm_plants.lua#L800-L805) |
| 公共端 tag | `plantedsoil`／`farm_plant`／`lunarplant_target`／`plant`／`plantresearchable`／`farmplantstress`／`tendable_farmplant`，另有 `plant_type_tag` 与 `israndomseed` | [tags](../../../raw/current/scripts/prefabs/farm_plants.lua#L807-L819) |
| 网络 | `_research_stage` 用 `net_tinybyte`，`plantregistrykey = product`，`displaynamefn = GetDisplayName`；`inspectable.nameoverride = "FARM_PLANT"` 只是检查文本前缀 | [netvar](../../../raw/current/scripts/prefabs/farm_plants.lua#L821-L826)、[检查文本](../../../raw/current/scripts/prefabs/farm_plants.lua#L843-L849) |
| 端分界 | `SetPristine()` 后 `if not TheWorld.ismastersim then return inst end`，此后组件全在主模拟 | [分界](../../../raw/current/scripts/prefabs/farm_plants.lua#L830-L834) |
| 主模拟组件 | `inspectable`→`plantresearchable`→`farmplantstress`（7 类）→`farmsoildrinker`→`farmplanttendable`→`hauntable`(`HAUNT_TINY`)→`lootdropper`→`growable`→`workable`→`knownlocations`／`herdmember`(`domesticplantherd`)→可燃／可蔓延→`MakeWaxablePlant` | [挂载](../../../raw/current/scripts/prefabs/farm_plants.lua#L843-L902) |

- 名称来自引擎默认：`SpawnPrefab` 返回后调 `SetPrefabName`，用 `STRINGS.NAMES[upper(prefab)]` 填 `name`；`GetDisplayName` 只在未识别／腐烂等状态覆盖它。[默认命名](../../../raw/current/scripts/mainfunctions.lua#L387-L392)、[显示名](../../../raw/current/scripts/prefabs/farm_plants.lua#L648-L674)
- 15 个注册各有唯一 `NAMES` 键：`FARM_PLANT_CARROT`＝胡萝卜植株、`FARM_PLANT_CORN`＝玉米杆、`FARM_PLANT_RANDOMSEED`＝种下的种子等；植株另有拼名键 `FARM_PLANT_SEED`＝「种下的{seed}」与状态键 `FARM_PLANT_UNKNOWN`／`FARM_PLANT_ROTTEN`。[po 段](../../../raw/current/scripts/languages/chinese_s.po#L342545-L342632)

## 阶段链与生长结算

普通作物用 `GROWTH_STAGES` 六段，`loopstages = true` 且 `loopstages_start = 2`，所以 `rotten` 之后回到 `sprout` 而不是 `seed`。[阶段表](../../../raw/current/scripts/prefabs/farm_plants.lua#L438-L567)、[循环参数](../../../raw/current/scripts/prefabs/farm_plants.lua#L876-L885)、[推进顺序](../../../raw/current/scripts/components/growable.lua#L121-L151)

| 阶段 | 时长函数与基准区间 | 进入时做什么 |
|---|---|---|
| `seed` 种子 | `GetGerminationTime`：360–480 秒（12–16×`SEG_TIME`），好季节 ×0.5 | 加 `planted_seed`、播 `sow`／`sow_idle` |
| `sprout` 发芽 | `GetGrowTime`：960–1680 秒 | 去 `planted_seed`、压力检查点、播 `grow_sprout` |
| `small` 小 | `GetGrowTime`：576–1008 秒 | 压力检查点 |
| `med` 中 | `GetGrowTime`：384–672 秒 | 压力检查点 |
| `full` 成熟 | `GetSpoilTime`：1920 秒（巨型 2880 秒）×`long_life` | 压力检查点并结算最终等级、判定巨型、开启 `pickable`、推 `ms_fruitflyspawneractive` |
| `rotten` 腐烂 | `GetSelfRegrowTime`：1920–2400 秒 | 压力检查点后 `Reset`、推 `ms_oncroppedrotted`、加 `farm_plant_killjoy` |

- 生长三段线性插值：`CalcGrowTime(检查点压力点数, 8, min, max)` = `min + (max-min)×点数/8 + rand×(max-min)/8`，好季节再 ×0.5；`num_steps` 取 `num_stressors + 1`＝8。[GetGrowTime](../../../raw/current/scripts/prefabs/farm_plants.lua#L377-L384)、[插值](../../../raw/current/scripts/prefabs/farm_plants.lua#L370-L375)
- 成熟期与腐烂期都不吃压力；`long_life` 只在成熟期乘（`GetSpoilTime` ×`TUNING.FARM_PLANT_LONG_LIFE_MULT`＝1.5），`GetSelfRegrowTime` 不乘。[GetSpoilTime](../../../raw/current/scripts/prefabs/farm_plants.lua#L386-L392)、[倍率](../../../raw/current/scripts/tuning.lua#L5796)
- 停长条件：夜间且 `DAYLIGHT_SEARCH_RANGE`＝30 内没有 `daylight`／`lightsource`（有效半径按 0.7 折算）就 `Pause`，白天 `Resume`；`full` 段 `night_grow = true` 不受此限；燃烧时同样暂停。夜间每 5 秒复检一次；读档时 `OnLoadPostPass` 先 `Pause`，再由 `OnIsDark` 按光照续长。[IsTooDarkToGrow](../../../raw/current/scripts/prefabs/farm_plants.lua#L30-L65)、[night_grow](../../../raw/current/scripts/prefabs/farm_plants.lua#L514-L535)、[读档暂停](../../../raw/current/scripts/prefabs/farm_plants.lua#L761-L765)

## 七类压力与照料

压力组件在构造时注册 7 类，`num_stressors`＝7；判定函数同时是描述文本的来源。[注册](../../../raw/current/scripts/prefabs/farm_plants.lua#L854-L861)

| 类别 | 判定 | 触发条件 | 对应操作 |
|---|---|---|---|
| `family` 家族 | `FamilyStressTest` | 半径 `FARM_PLANT_SAME_FAMILY_RADIUS`＝4 内同 `plant_type_tag` 少于 `FARM_PLANT_SAME_FAMILY_MIN`＝4 株（含自身） | 同种作物密植到 4 株 |
| `season` 季节 | `SeasonStressTest` | `plant_def.good_seasons[当前季节]` 为假 | 按季节表轮作 |
| `moisture` 水分 | `MoistureStressTest` | 含水时间占比 `CalcPercentTimeHydrated()` 低于 `FARM_PLANT_DROUGHT_TOLERANCE`＝0.10 | 浇水、等雨、冰／雪球融水 |
| `nutrients` 养分 | `NutrientsStressTest`（[扣肥](../../../raw/current/scripts/components/farming_manager.lua#L400-L455)） | 脚下地块对应养分格现有量小于 `nutrient_consumption` 的需求 | 堆肥、轮作回肥 |
| `overcrowding` 拥挤 | `OvercrowdingStressTest` | 同一地块格 `farm_plant` 超过 `FARM_PANT_OVERCROWDING_MAX_PLANTS`＝10 | 拉开株距 |
| `killjoys` 扫兴 | `KillJoyStressTest` | 半径 `FARM_PLANT_KILLJOY_RADIUS`＝6 内带 `farm_plant_killjoy`（腐烂株）超过 `FARM_PLANT_KILLJOY_TOLERANCE`＝0 | 清掉腐烂株 |
| `happiness` 幸福 | `HappinessStressTest` | 直接读当前值；默认「有压力」，只有照料把它置假 | 每阶段照料一次 |

以上判定与常量：[压力函数](../../../raw/current/scripts/prefabs/farm_plants.lua#L169-L221)、[常量](../../../raw/current/scripts/tuning.lua#L5798-L5814)

- 检查点：`sprout`／`small`／`med`／`full` 的 `pregrowfn` 各跑一次 `MakeStressCheckpoint`（`rotten` 也跑一次但随即 `Reset`，不进最终等级），逐类 `SetStressed` 后 `MakeCheckpoint` 把本轮「有压力」的类数累加进 `stress_points`，并把每类复位为有压力、等下一次重评；`full` 那次额外 `CalcFinalStressState`。[检查点](../../../raw/current/scripts/prefabs/farm_plants.lua#L223-L258)
- 最终等级只看累计点数：≤1 `NONE`（允许犯一次错）、≤6 `LOW`、≤11 `MODERATE`、其余 `HIGH`。[分级](../../../raw/current/scripts/components/farmplantstress.lua#L72-L80)
- 照料链路：照料动作是 `ACTIONS.INTERACT_WITH`（`strfn` 只对 `farm_plant` 给名），它的 `fn` 调 `farmplanttendable:TendTo(doer)`；`ontendto` 只清 `happiness` 并返回真，`TendTo` 随即把 `tendable` 置假，下一阶段 `fn` 的 `SetTendable(stage_data.tendable)` 再开放（`seed`–`med` 为真，`full`／`rotten` 为假）。每个阶段最多照料一次，正好覆盖下一个检查点。[动作](../../../raw/current/scripts/actions.lua#L1547-L1563)、[TendTo](../../../raw/current/scripts/components/farmplanttendable.lua#L26-L31)、[阶段 tendable](../../../raw/current/scripts/prefabs/farm_plants.lua#L438-L512)
- **「照料加速生长」没有直接倍率**：照料降低的是检查点压力点数，而下一段时长按该点数线性插值，0 点仍在首个 1/8 带宽内随机；想让植株长快只有少压力、好季节和缩短 `plant_def.grow_time` 三条路。[GetGrowTime](../../../raw/current/scripts/prefabs/farm_plants.lua#L377-L384)

## 收获、腐烂与清除

- 可采开关：`full` 进阶段时 `MakePickable(inst, true)`——`pickable:SetUp(nil)`、`use_lootdropper_for_product = true`、`remove_when_picked = true`，采摘音实际恒为 `pickup_plants`（调用点从不传 `product`，`harvest_berries` 分支是死代码）；非腐烂株获得 `fruitflyspawner`，`full` 进阶段时推 `ms_fruitflyspawneractive`。[MakePickable](../../../raw/current/scripts/prefabs/farm_plants.lua#L303-L322)、[full](../../../raw/current/scripts/prefabs/farm_plants.lua#L520-L530)
- 采摘时 `pickable` 用 `lootdropper:GenerateLoot` 结算，摘／铲／烧共用 `SetupLoot` 的基础表；烧毁时这张表还会被再转换一次（见[陷阱](#常见陷阱与限制)）。[SpawnProductLoot](../../../raw/current/scripts/components/pickable.lua#L450-L490)

| 结局 | 战利品表 | 证据 |
|---|---|---|
| 巨型成熟株 | `{<作物>_oversized}` | [SetupLoot](../../../raw/current/scripts/prefabs/farm_plants.lua#L344-L362)、[lootsetupfn](../../../raw/current/scripts/components/lootdropper.lua#L26-L39) |
| `NONE`／`LOW` | `{产物, 种子, 种子}` | 同上 |
| `MODERATE` | `{产物, 种子}` | 同上 |
| `HIGH` | `{产物}` | 同上 |
| 腐烂普通株 | `{spoiled_food}` | 同上 |
| 腐烂巨型株 | `loot_oversized_rot`＝3×`spoiled_food`＋1×种子＋2×`fruitfly` | [字段](../../../raw/current/scripts/prefabs/farm_plant_defs.lua#L270) |

- 巨型不是掷骰：`full` 的 `pregrowfn` 里 `is_oversized = force_oversized or (not no_oversized and 最终压力 == NONE)`；`no_oversized` 在法术催熟后与「从腐烂株循环回来」的植株上被置真。[判定](../../../raw/current/scripts/prefabs/farm_plants.lua#L514-L518)、[催熟](../../../raw/current/scripts/prefabs/farm_plants.lua#L582-L596)、[腐烂循环](../../../raw/current/scripts/prefabs/farm_plants.lua#L551-L558)
- 腐烂分支：`full` 计时到期进 `rotten`，加 `farm_plant_killjoy` 与 `pickable_harvest_str`，外观换 `rot`／`rot_oversized`；普通株再次 `StartGrowing` 回 `sprout`，巨型株 `StopGrowing` 永久停在腐烂态。[rotten](../../../raw/current/scripts/prefabs/farm_plants.lua#L537-L566)
- 铲除：`workable` 用 `ACTIONS.DIG`、工作量 1；`dig_up` 先 `DropLoot`，按当前阶段 `dig_fx`（都是 `dirt_puff`）生成特效后删除，并对非 `plantkin` 的作业者触发 `farm_plant_defender` 的 `defend_farm_plant` 求援。[workable](../../../raw/current/scripts/prefabs/farm_plants.lua#L887-L890)、[设工作量](../../../raw/current/scripts/components/workable.lua#L83-L90)、[dig_up](../../../raw/current/scripts/prefabs/farm_plants.lua#L67-L82)
- 烧毁：除 `fireproof`（只有 `dragonfruit`）外都挂 `MakeSmallBurnable`＋`MakeSmallPropagator`；`onignite` 记幸福压力并重算暂停，`onburnt` 的落地细节见[陷阱](#常见陷阱与限制)。[可燃分支](../../../raw/current/scripts/prefabs/farm_plants.lua#L896-L902)、[onburnt](../../../raw/current/scripts/prefabs/farm_plants.lua#L84-L91)

### 花土接线

- 种下：种子侧 `FarmPlantable:Plant` 要求目标 `HasTag("soil")`，先 `target:Remove()`（耕地被吃掉）再 `SpawnPrefab("farm_plant_<作物>")`，推 `on_planted {doer, seed, in_soil = true}` 与世界事件 `itemplanted`。[Plant](../../../raw/current/scripts/components/farmplantable.lua#L7-L30)
- 植株侧只消费 `doer`：`plantkin`（沃姆伍德）种下时置 `long_life = true`，成熟期再乘 1.5；`in_soil` 在农田植株上没有消费者。[on_planted](../../../raw/current/scripts/prefabs/farm_plants.lua#L693-L697)
- 收获后：若脚下是 `WORLD_TILES.FARMING_SOIL`，`OnPicked` 生成 `farm_soil` 并推 `breaksoil`，`OnBreak` 把它变成 `NOCLICK`／`NOBLOCK` 的背景塌陷态，耕地不复原。[OnPicked](../../../raw/current/scripts/prefabs/farm_plants.lua#L287-L301)、[OnBreak](../../../raw/current/scripts/prefabs/farm_soil.lua#L22-L31)

## 逐作物差异表

时间基准 `SEG_TIME`＝30 秒、`TOTAL_DAY_TIME`＝480 秒；14 种作物的 `grow_time` 由同一组参数 `MakeGrowTimes(12–16×SEG_TIME, 4–7×TOTAL_DAY_TIME)` 生成，数值完全一致，差异在耗水、养分、季节与特性上。[基准](../../../raw/current/scripts/tuning.lua#L19-L20)、[生成器](../../../raw/current/scripts/prefabs/farm_plant_defs.lua#L21-L38)

| 段 | 计算 | 秒（设计区间） |
|---|---|---|
| 种子期 `seed` | 12–16×`SEG_TIME` 均匀随机 | 360–480 |
| 成长期三段 `sprout`／`small`／`med` | 4–7 天按 0.5／0.3／0.2 拆分 | 960–1680／576–1008／384–672，合计 1920–3360 |
| 成熟期 `full` | 4 天 | 1920（巨型 2880） |
| 枯萎期 `rotten` | 4–5 天 | 1920–2400 |

- 好季节 ×0.5 只作用于发芽与 `sprout`／`small`／`med`，成熟与腐烂不乘；压力点数把三段推进到设计区间，0 点仍在首个 1/8 带宽内、7 点落在最后一个 1/8 带宽内，都不是定值。[设季减半](../../../raw/current/scripts/prefabs/farm_plants.lua#L364-L384)
- 养分记号 `S`／`M`／`L` 即 `FARM_PLANT_CONSUME_NUTRIENT_LOW`＝2／`MED`＝4／`HIGH`＝8，向量按「格 1／格 2／格 3」排列，0 表示不耗且该格参与回补。[常量](../../../raw/current/scripts/tuning.lua#L5798-L5800)、[回补表](../../../raw/current/scripts/prefabs/farm_plant_defs.lua#L126-L131)
- 耗水档换算：`LOW`＝−0.0075/秒（100 点约 13333 秒≈27.8 天）、`MED`＝−0.02/秒（≈10.4 天）、`HIGH`＝−0.035/秒（≈6.0 天）；只有 `growable` 正在生长时才耗水。[耗水常量](../../../raw/current/scripts/tuning.lua#L5802-L5804)、[取率](../../../raw/current/scripts/prefabs/farm_plants.lua#L121-L123)、[湿干计时](../../../raw/current/scripts/components/farmsoildrinker.lua#L32-L39)
- 家族归属：每株的 `plant_type_tag` 恒为 `farm_plant_<作物>`，所以「家族」等于同种作物，阈值 4 株、半径 4；下表不再逐行重复。[家族字段](../../../raw/current/scripts/prefabs/farm_plant_defs.lua#L265-L274)

| 注册名 | 简中 | 耗水 | 养分 1／2／3 | 好季节 | 特性 |
|---|---|---|---|---|---|
| `farm_plant_carrot` | 胡萝卜植株 | `LOW` | `M`／0／0 | 秋、冬、春 | — |
| `farm_plant_corn` | 玉米杆 | `LOW` | 0／`M`／0 | 秋、春、夏 | — |
| `farm_plant_potato` | 土豆植株 | `LOW` | 0／0／`M` | 秋、冬、春 | — |
| `farm_plant_tomato` | 番茄植株 | `HIGH` | `S`／`S`／0 | 秋、春、夏 | — |
| `farm_plant_asparagus` | 芦笋蕨 | `LOW` | 0／`M`／0 | 冬、春 | — |
| `farm_plant_eggplant` | 茄子杆 | `MED` | 0／0／`M` | 秋、春 | — |
| `farm_plant_pumpkin` | 南瓜植株 | `MED` | `M`／0／0 | 秋、冬 | `iscarpable` |
| `farm_plant_watermelon` | 西瓜植株 | `HIGH` | 0／`S`／`S` | 春、夏 | — |
| `farm_plant_dragonfruit` | 火龙果藤 | `MED` | 0／0／`L` | 春、夏 | `fireproof`（不可燃） |
| `farm_plant_durian` | 榴莲藤 | `LOW` | 0／`L`／0 | 春 | — |
| `farm_plant_garlic` | 大蒜植株 | `LOW` | 0／`L`／0 | 四季 | — |
| `farm_plant_onion` | 洋葱植株 | `MED` | `L`／0／0 | 秋、春、夏 | — |
| `farm_plant_pepper` | 辣椒植株 | `LOW` | 0／0／`L` | 秋、夏 | — |
| `farm_plant_pomegranate` | 石榴枝 | `MED` | `L`／0／0 | 春、夏 | — |

以上逐作物数值：[耗水](../../../raw/current/scripts/prefabs/farm_plant_defs.lua#L60-L80)、[季节](../../../raw/current/scripts/prefabs/farm_plant_defs.lua#L83-L100)、[养分](../../../raw/current/scripts/prefabs/farm_plant_defs.lua#L102-L124)、[特性](../../../raw/current/scripts/prefabs/farm_plant_defs.lua#L153-L155)

- `iscarpable` 只在作物物品侧被消费：`veggies.lua` 用它给 `pumpkin_oversized` 加 `pumpkincarvable` 与两条雕刻特效依赖，植株构造里没有消费者；好季节则同时决定该段时长 ×0.5 与 `season` 压力判定，表里缺的季节就是坏季节。[消费](../../../raw/current/scripts/prefabs/veggies.lua#L439-L441)、[挂载](../../../raw/current/scripts/prefabs/veggies.lua#L853-L857)、[两处读取](../../../raw/current/scripts/prefabs/farm_plants.lua#L364-L382)

## farm_plant_randomseed 的独立接线

- 字段：`PLANT_DEFS.randomseed` 是 `is_randomseed = true`，于是 `seed = "seeds"`、`plant_type_tag = "farm_plant_randomseed"`、`family_min_count = 0`（无家族压力），也没有 `product`／`product_oversized`。[分支](../../../raw/current/scripts/prefabs/farm_plant_defs.lua#L260-L263)
- 只有两阶段：`GROWTH_STAGES_RANDOMSEED` = `seed`（播种动画）→ `sprout`（`ReplaceWithPlant`），没有 `small`／`med`／`full`／`rotten`，也不跑压力检查点与巨型判定；`SetupLoot` 只在 `full`／`rotten` 赋表，随机株两段都不命中，`loot` 为空。[两阶段](../../../raw/current/scripts/prefabs/farm_plants.lua#L412-L436)、[SetupLoot](../../../raw/current/scripts/prefabs/farm_plants.lua#L344-L362)
- 品种决定：`pickfarmplant` 先按 `FARM_PLANT_RANDOMSEED_WEED_CHANCE`＝0.20 抽杂草（`weed_defs.weighted_seed_table`），否则按 `VEGGIES[k].seed_weight` 加权，好季节 ×`SEED_WEIGHT_SEASON_MOD`＝2，返回 `farm_plant_<作物>`。[pickfarmplant](../../../raw/current/scripts/prefabs/farm_plants.lua#L134-L149)、[权重](../../../raw/current/scripts/tuning.lua#L102-L103)
- 替换分两支：抽到作物（`plant.plant_def ~= nil`）时，生成目标株后置 `no_oversized = true`、复制 `long_life`、`CopyFrom` 两个状态、`growable:DoGrowth()` 并覆盖贴图；抽到杂草时只 `SpawnPrefab`、定位与替换，不复制状态、不置 `no_oversized`。[替换](../../../raw/current/scripts/prefabs/farm_plants.lua#L151-L167)
- 识别早知：带 `farmplantidentifier` 的玩家可用 `BeIdentified` 提前锁定 `_identified_plant_type`（存读档，`ReplaceWithPlant` 优先用它），描述函数只对该 tag 玩家列出品种名。[识别](../../../raw/current/scripts/prefabs/farm_plants.lua#L700-L717)、[存档](../../../raw/current/scripts/prefabs/farm_plants.lua#L721-L759)
- 种下入口：通用 `seeds` 的播种目标返回 `"farm_plant_randomseed"`，沃姆伍德直接部署也生成它并推 `on_planted {in_soil = false}`。[seeds](../../../raw/current/scripts/prefabs/seeds.lua#L60-L62)、[部署](../../../raw/current/scripts/prefabs/seeds.lua#L131-L138)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`growable` | 六阶段表、`loopstages`／`loopstages_start = 2`、`growoffscreen`、`magicgrowable`、`domagicgrowthfn`；`DoGrowth` 顺序是 `pregrowfn → SetStage(fn) → StartGrowing` | 仅主模拟；夜间与燃烧 `Pause` | [[reference/components/growable#关键接口]]；[配置](../../../raw/current/scripts/prefabs/farm_plants.lua#L876-L885) |
| 组件：`farmplantstress` | 7 类压力、检查点累计、最终分级；随存档保存 | 主模拟 | [[reference/components/farmplantstress#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/farm_plants.lua#L854-L861) |
| 组件：`farmsoildrinker` | 湿／干计时、`CalcPercentTimeHydrated` 供水压力；`getdrinkratefn` 只在生长时给耗水率 | 主模拟；向 `farming_manager` 注册 | [[reference/components/farmsoildrinker#关键接口]]；[接线](../../../raw/current/scripts/prefabs/farm_plants.lua#L863-L865) |
| 组件：`farmplanttendable` | `ACTIONS.INTERACT_WITH` 入口；成功后关闭并清 happiness | `seed`–`med` 可照料 | [[reference/components/farmplanttendable#关键接口]]；[接线](../../../raw/current/scripts/prefabs/farm_plants.lua#L867-L868) |
| 组件：`pickable` | 成熟／腐烂态可采，`remove_when_picked` 直接删株 | `full`／`rotten` | [[reference/components/pickable#关键接口]]；[开关](../../../raw/current/scripts/prefabs/farm_plants.lua#L303-L322) |
| 组件：`workable` | `ACTIONS.DIG`、工作量 1 | 全阶段 | [[reference/components/workable#关键接口]]；[配置](../../../raw/current/scripts/prefabs/farm_plants.lua#L887-L890) |
| 组件：`lootdropper` | `lootsetupfn = SetupLoot`，摘／铲／烧同表 | 主模拟 | [[reference/components/lootdropper#关键接口]]；[接线](../../../raw/current/scripts/prefabs/farm_plants.lua#L873-L874) |
| 组件：`waxable` | `MakeWaxablePlant` 让植株可打蜡成蜡制标本 | 主模拟；腐烂株受 `waxable` 生命周期影响 | [helper](../../../raw/current/scripts/standardcomponents.lua#L1888-L1892)、[调用](../../../raw/current/scripts/prefabs/farm_plants.lua#L910) |
| API：`farming_manager` | `CycleNutrientsAtPoint` 扣／回养分、`AddSoilMoistureAtPoint` 补水、`IsSoilMoistAtPoint` 判湿 | 世界组件，按地块格 | [[reference/components/farming-manager#关键接口]]；[调用](../../../raw/current/scripts/prefabs/farm_plants.lua#L211-L217) |
| 关联 Prefab：`farm_soil` | 种植时被消耗，收获后补一块再 `breaksoil` | 仅 `FARMING_SOIL` 地块 | [OnPicked](../../../raw/current/scripts/prefabs/farm_plants.lua#L287-L293) |
| 关联 Prefab：`seeds`／`<作物>_seeds` | 种下入口与 `plant_def.seed` 战利品 | 见 [[reference/prefabs/farm-crop-seeds]]、[[reference/prefabs/generic-seeds]] | [种子字段](../../../raw/current/scripts/prefabs/farm_plant_defs.lua#L265-L267) |
| 资源：anim | `anim/farm_plant_<bank>.zip`＋`anim/farm_plant_<build>.zip`＋`farm_soil.zip` | 14 键按 bank 分组共用 | [assets](../../../raw/current/scripts/prefabs/farm_plants.lua#L768-L775) |

## 典型接入

### 只改某一种作物的成熟期（腐烂倒计时）

- 前提与选择：服务端 `modmain.lua` 的 `AddPrefabPostInit`。选择改 `plant_def.grow_time.full` 而不是 `TUNING`：`GetSpoilTime` 每读一次都从 `inst.plant_def` 取，而 `plant_def` 就是 `PLANT_DEFS[键]` 那张**共享表**，所以一次改动会作用于该作物所有实例（含旧存档里尚未进入成熟期的植株）。
- 生效与撤销：只影响「之后才进入 `full`」的植株；已经起算的成熟期计时器不会重算。撤掉模组、重进世界后表回到原值，但已按新值起算的计时器会带着剩余时间走完。巨型株走 `grow_time.oversized`，本示例不动它。
- 原版依据：`GetSpoilTime` 读 `plant_def.grow_time.full`（巨型读 `.oversized`）再乘 `long_life` 倍率。[GetSpoilTime](../../../raw/current/scripts/prefabs/farm_plants.lua#L386-L388)

```lua
-- modmain.lua：把南瓜植株的成熟期从 4 天延到 8 天（巨型仍 6 天）
local CROP = "pumpkin"
local DEFAULT_FULL = 4 * GLOBAL.TUNING.TOTAL_DAY_TIME

AddPrefabPostInit("farm_plant_" .. CROP, function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    local def = inst.plant_def
    if def ~= nil and def.grow_time ~= nil and def.grow_time.full == DEFAULT_FULL then
        def.grow_time.full = 8 * GLOBAL.TUNING.TOTAL_DAY_TIME
    end
end)
```

- 调整与验证：单位是秒；验证看 `full` 之后的 `growable.targettime` 与 `GetTime()` 差值；只改单株要换 `inst.plant_def` 副本。

## 常见陷阱与限制

- **误区：压力按株计。** 实际按「类别数 × 检查点次数」累计：4 个检查点、每点最多 7 类，`NONE` 要求全程累计 ≤1（只容一次失误），所以巨型是压力管理的产物，不是运气。[分级](../../../raw/current/scripts/components/farmplantstress.lua#L72-L80)
- **边界：不照料会用光唯一的容错。** `happiness` 默认算有压力，只有照料（或某些回调）把它置假；漏一次即 1 点，正好用尽 `NONE` 的 ≤1 额度，漏两次就掉到 `LOW`。[检查点](../../../raw/current/scripts/prefabs/farm_plants.lua#L223-L258)
- **边界：收获即毁株，没有「再生长」。** `remove_when_picked = true`，摘完原地只剩塌陷的 `farm_soil`；真正的循环只有 `rotten → sprout`，且巨型株不进这一循环。[MakePickable](../../../raw/current/scripts/prefabs/farm_plants.lua#L303-L322)
- **边界：催熟的植株被永久封掉巨型。** 法术催熟会置 `no_oversized`，从腐烂循环回来的植株同样置真，之后再完美也长不成巨型。[催熟](../../../raw/current/scripts/prefabs/farm_plants.lua#L582-L596)
- **边界：烧掉的是「转换后」的那份，未成熟株只有灰。** 免燃只给 `dragonfruit`；非免燃株 `onburnt` 先落 `ash` 再 `DropLoot` 再删除，此时仍在燃烧，`lootdropper` 逐件换 `SetupLoot` 的表：可烤换 `_cooked`（胡萝卜→烤胡萝卜）、种子按 `BURNED_LOOT_OVERRIDES` 换 `seeds_cooked`、其余落灰（受控燃烧保留原注册）；未成熟株没有表。[不可燃](../../../raw/current/scripts/prefabs/farm_plant_defs.lua#L155)、[onburnt](../../../raw/current/scripts/prefabs/farm_plants.lua#L84-L91)、[转换](../../../raw/current/scripts/components/lootdropper.lua#L415-L441)
- **边界：`iscarpable` 在植株侧不消费。** 它只让 `pumpkin_oversized` 带 `pumpkincarvable`；想「给植株加雕刻」得自己挂组件。[消费](../../../raw/current/scripts/prefabs/veggies.lua#L853-L857)
- **未核验**：`farm_plant_killjoy` 在腐烂株上的完整表现（果蝇生成概率、铲除时的求援）只核对了分支入口，未做运行验证；植物图鉴 UI（`plantregistrywidget`）不在本页。

## 验证记录

- 源码审阅：2026-11-08，Steam buildid 26474873（2026-11-09 复检生长计时与随机株两分支）。核对 `farm_plants.lua` 全文（15 注册循环、六阶段与两阶段随机种、7 类压力与检查点、`SetupLoot`、pickable／workable／burn 分支、`ReplaceWithPlant` 与存档字段）、`farm_plant_defs.lua` 全表（`MakeGrowTimes`、耗水、季节、养分、特性、末段填充）、`farmplantable`／`farmplantstress`／`farmplanttendable`／`farmsoildrinker`／`growable`／`pickable`／`workable`／`lootdropper`／`farming_manager`／`farm_soil` 的相关分支、`seeds.lua` 的随机种入口、`veggies.lua` 的 `iscarpable` 消费、`mainfunctions.lua` 的默认命名，以及 `tuning.lua` 的 `SEG_TIME`／`TOTAL_DAY_TIME`／`FARM_PLANT_*`／`FARM_PANT_OVERCROWDING_MAX_PLANTS`／`FARM_PLANT_LONG_LIFE_MULT`。
  名称核验：15 个注册在 `languages/chinese_s.po` 各有唯一 `NAMES` 键，简中名见[逐作物差异表](#逐作物差异表)。[po 段](../../../raw/current/scripts/languages/chinese_s.po#L342545-L342632)
  未覆盖：14 种作物物品与专用种子数值（见链接页）、巨型物品的雕刻与称重、`weed_plants` 的杂草株、图鉴 UI 与存档迁移。
- 游戏运行：未运行。秒数、天数与耗水天数均为 tuning 推导（`SEG_TIME`＝30、`TOTAL_DAY_TIME`＝480、水分上限 100）。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 均通过；Lua 示例人工核对语法。

## 相关链接

- 种子与播种：[[reference/prefabs/farm-crop-seeds]]、[[reference/prefabs/generic-seeds]]
- 作物物品与巨型：[[reference/prefabs/farm-crops-staple]]、[[reference/prefabs/farm-crops-fruit]]、[[reference/prefabs/farm-crops-spice]]、[[reference/prefabs/giant-crops]]
- 机制页：[[reference/components/farming-manager]]、[[reference/components/farmplantable]]、[[reference/components/farmplantstress]]、[[reference/components/farmplanttendable]]、[[reference/components/growable]]
- 农具：[[reference/prefabs/farm-hoe]]

返回 [[reference/prefabs/index|Prefab 索引]]。
