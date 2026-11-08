---
title: "基础粮菜四作物的生熟两态（carrot、corn、pumpkin、potato 与各自 _cooked）"
created: "2026-11-08"
updated: "2026-11-08"
type: prefab
summary: "veggies.lua 同一工厂产出的基础粮菜八注册：MakeVegStats 的 16 位参数、MakeVeggie 的导出集合与主模拟端分界，四键生熟两态的三维／新鲜度／堆叠／漂浮差异，以及农田收获、野生胡萝卜、巨型作物与活动掉落等获得路径。"
aliases: ["carrot", "carrot_cooked", "corn", "corn_cooked", "pumpkin", "pumpkin_cooked", "potato", "potato_cooked", "胡萝卜", "烤胡萝卜", "玉米", "爆米花", "南瓜", "烤南瓜", "土豆", "烤土豆", "基础粮菜", "农田作物"]
tags: [prefab, item, food]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/veggies.lua", "prefabs/farm_plants.lua", "prefabs/farm_plant_defs.lua", "prefabs/carrot.lua", "prefabs/seeds.lua", "prefabs/pigking.lua", "prefabs/armor_carrotlure.lua", "prefabs/carrat.lua", "prefabs/halloweenpotion_moon.lua", "prefabs/wormwood_carrat.lua", "brains/cozy_bunnymanbrain.lua", "prefabs/bunnyman.lua", "components/perishable.lua", "components/edible.lua", "components/cookable.lua", "components/stackable.lua", "components/bait.lua", "components/tradable.lua", "components/weighable.lua", "components/repairer.lua", "components/repairable.lua", "components/activatable.lua", "components/halloweenmoonmutable.lua", "components/halloweenpotionmoon.lua", "components/pickable.lua", "components/snowmandecor.lua", "components/regrowthmanager.lua", "entityscript.lua", "standardcomponents.lua", "tuning.lua", "constants.lua", "actions.lua", "componentactions.lua", "simutil.lua", "gamelogic.lua", "map/layouts.lua", "map/forest_map.lua", "map/rooms/forest/blockers.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
size_class: complex
---

# 基础粮菜四作物的生熟两态（carrot、corn、pumpkin、potato 与各自 _cooked）

## 快速判断

- 职责与适用：`prefabs/veggies.lua` 的 `MakeVeggie` 工厂为 `VEGGIES` 表每一项生成「生 + 熟」两件可食物品。本页覆盖 `carrot`、`corn`、`pumpkin`、`potato` 及各自 `_cooked` 共 8 个注册。可借鉴：同一数据表驱动的生熟双注册。[VEGGIES 表](../../../raw/current/scripts/prefabs/veggies.lua#L39-L155)、[注册循环](../../../raw/current/scripts/prefabs/veggies.lua#L1102-L1110)
- 优先使用：改三维／保质／堆叠先看[生熟两态数值](#生熟两态数值)；寿命与腐坏产物查 [[reference/components/perishable#关键接口]]、烹饪产物查 [[reference/components/cookable#关键接口]]；巨型形态与打蜡查 [[reference/prefabs/giant-crops]]。
- 重要副作用：生土豆设 `health=-3`、`sanity=-5`，`edible` 赋值即打 `badfood` 标签；生南瓜保质期按万圣夜活动期取 20 天、否则 10 天，加载时定死。[badfood](../../../raw/current/scripts/components/edible.lua#L1-L7)、[南瓜分支](../../../raw/current/scripts/prefabs/veggies.lua#L56-L58)
- 修改边界：只讲物品形态的构造、数值与获得；`_seeds`（[[reference/prefabs/farm-crop-seeds|作物种子]]）、三件 `_oversized*`、田间生长与压力、烹饪配方不在本页。

## 注册与构造

8 个注册由 `MakeVeggie(name, not SEEDLESS[name])` 产出：四键各调一次、各得六件 Prefab（种子、三件巨型、本体、熟态），其中只有本体与熟态进本页。[SEEDLESS](../../../raw/current/scripts/prefabs/veggies.lua#L160-L168)

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `carrot` | `fn`：公共端 bank／build＝`carrot`、`PlayAnimation("idle")`、tag `cookable`／`weighable_OVERSIZEDVEGGIES`、浮水 `{"med",0.05,0.8}` → `SetPristine` → 主模拟逐项挂组件 | 唯一有专属接线：`repairer`（`MATERIALS.CARROT`＋0.25 保鲜修复）、`halloweenmoonmutable`（`carrat`）、`snowmandecor`、`timer`／`activatable`／`Spin`／`SetOnPutInInventoryFn`。堆叠 40。[浮水与 tag](../../../raw/current/scripts/prefabs/veggies.lua#L578-L583)、[专属组件](../../../raw/current/scripts/prefabs/veggies.lua#L607-L611)、[旋转接线](../../../raw/current/scripts/prefabs/veggies.lua#L689-L702) |
| `carrot_cooked` | `fn_cooked`：同名 bank／build、`PlayAnimation("cooked")`、浮水 `{"small",0.1,nil}`、`scrapbook_anim="cooked"` → `SetPristine` → 主模拟 | 只有 perishable／edible／stackable／inspectable／inventoryitem／burnable＋propagator／bait／tradable／hauntable；无 cookable、无 weighable、无上述专属接线。[熟态工厂](../../../raw/current/scripts/prefabs/veggies.lua#L707-L777) |
| `corn` | `fn`：无任何名字分支，走默认浮水 `MakeInventoryFloatable(inst)` | 组件集与 `carrot` 相同减去四项专属；堆叠 40。熟态饥饿反而低于生（25→12.5）。[数值行](../../../raw/current/scripts/prefabs/veggies.lua#L53-L54)、[默认浮水](../../../raw/current/scripts/standardcomponents.lua#L1772-L1774) |
| `corn_cooked` | `fn_cooked` | 12.5／3／0，保质 15 天；简中名「爆米花」。 |
| `pumpkin` | `fn`：无专属组件；工厂按 `PLANT_DEFS.pumpkin.iscarvable` 只在巨型件上追加雕刻依赖 | 生鲜期按活动期分支（见[南瓜的活动期分支](#南瓜的活动期分支)）；stackable **不设** maxsize 而落到组件默认 20。[堆叠例外](../../../raw/current/scripts/prefabs/veggies.lua#L613-L619)、[`iscarvable`](../../../raw/current/scripts/prefabs/farm_plant_defs.lua#L8-L9) |
| `pumpkin_cooked` | `fn_cooked` | 37.5／8／0，保质 6 天，堆叠 40，浮水 `{"small",0.1,nil}`——生熟堆叠不同档。 |
| `potato` | `fn`：无专属组件；浮水 `{nil,0.05,0.65}`（size 省略→`"small"`） | 生吃负收益→ `badfood`；堆叠 40。[数值行](../../../raw/current/scripts/prefabs/veggies.lua#L128-L130) |
| `potato_cooked` | `fn_cooked` | 25／20／0——四键里熟后提升最大的一件；保质 6 天。 |

### 获得与入栏

| 注册 | 已核验获得路径 | 产出与证据 |
|---|---|---|
| `carrot` | ①农田成熟收获 ②野生 `carrot_planted` 采摘 ③`carrot_oversized` 锤碎 ④普通兔人死亡随机掉落 ⑤兔人小游戏、`wormwood_carrat` 结束掉落 | ④三选一（`carrot`／`meat`／`manrabbit_tail` 权重 3／3／2）[兔人掉落](../../../raw/current/scripts/prefabs/bunnyman.lua#L313-L316)；⑤见 [cozy_bunnymanbrain](../../../raw/current/scripts/brains/cozy_bunnymanbrain.lua#L423-L438)、[wormwood_carrat](../../../raw/current/scripts/prefabs/wormwood_carrat.lua#L38-L44)；`carrat` 掉落表不含 `carrot` [carrat](../../../raw/current/scripts/prefabs/carrat.lua#L52-L56) |
| `corn` | ①农田成熟收获 ②`corn_oversized` 锤碎 | 无野生植株：检索范围内未见其他生成点（范围见[验证记录](#验证记录)） |
| `pumpkin` | ①农田成熟收获 ②`pumpkin_oversized` 锤碎 ③`FisherPig` 布局有一处 `unknown_fruit` 摆放位；该布局只见定义、现役房间引用未证实，**放置未核验** | ③[池](../../../raw/current/scripts/map/layouts.lua#L324-L324)、[摆放](../../../raw/current/scripts/map/layouts.lua#L340-L340)、[引用仅注释](../../../raw/current/scripts/map/rooms/forest/blockers.lua#L171-L171) |
| `potato` | ①农田成熟收获 ②`potato_oversized` 锤碎 ③沃利默认开局带 2 个 | ③[开局物品](../../../raw/current/scripts/tuning.lua#L3466-L3466)；无野生植株，同 `corn` |

三条入栏链不同：**农田成熟**走 `pickable` 的 lootdropper，逐件 `inventory:GiveItem(item, nil, pt)`，收不下才落地（[product 交付](../../../raw/current/scripts/components/pickable.lua#L463-L491)、[农田接线](../../../raw/current/scripts/prefabs/farm_plants.lua#L303-L322)）；**野生胡萝卜**走单产物分支 `SpawnPrefab("carrot")`＋`GiveItem`，植株 `remove_when_picked`（[野生注册](../../../raw/current/scripts/prefabs/carrot.lua#L35-L40)、[单产物](../../../raw/current/scripts/components/pickable.lua#L493-L512)）；**巨型作物**是 `workable(HAMMER)` 完成回调 `DropLoot`＋`Remove`，先落地再拾取（[锤碎](../../../raw/current/scripts/prefabs/veggies.lua#L221-L224)）。

掉种子看路径：农田按压力给 0～2 包（低／无压力 `{作物, 种子×2}`、中等 `{作物, 种子}`、高压仅作物、非巨型腐烂株只给 `spoiled_food`，[SetupLoot](../../../raw/current/scripts/prefabs/farm_plants.lua#L344-L362)）；巨型锤碎固定 2 本体＋2 包种子，第 5 件 75% 再给本体、否则给种子（[掉落表](../../../raw/current/scripts/prefabs/veggies.lua#L231-L235)）。`<name>_seeds` 归种子注册。

## 共用工厂与生熟两态

有种子（可种）的 14 个基础名共用一套工厂：`VEGGIES` 是 20 项 `MakeVegStats(...)` 调用的数据表（另 6 项无种子，见 `SEEDLESS`），注册循环对每项调 `MakeVeggie(name, not SEEDLESS[name])` 并摊平返回值。[数据表](../../../raw/current/scripts/prefabs/veggies.lua#L39-L155)、[循环](../../../raw/current/scripts/prefabs/veggies.lua#L1102-L1110)

### MakeVegStats 的参数位序

16 位全按位置传参，返回表的键序与形参序不同（下表以形参序为准）；缺省位留 `nil`，本页四键只用到前 11 位与第 14、16 位。[定义](../../../raw/current/scripts/prefabs/veggies.lua#L4-L23)

| 位 | 形参／含义 |
|---|---|
| 1 | `seedweight`——随机种子加权抽取的权重（取值 `SEED_CHANCE_COMMON`／`UNCOMMON`／`RARE`，消费方在种子侧）；[seeds](../../../raw/current/scripts/prefabs/seeds.lua#L43-L58) |
| 2–5 | `hunger`、`health`、`perish_time`、`sanity`——生四维 |
| 6–9 | `cooked_hunger`、`cooked_health`、`cooked_perish_time`、`cooked_sanity`——熟四维 |
| 10–11 | `float_settings`、`cooked_float_settings`——`{size, offset, scale}` 三元组，直接展开给 `MakeInventoryFloatable` |
| 12–16 | 条件件：`dryable`（非 `nil` 才注册 `_dried`）、`secondary_foodtype`（写 `edible.secondaryfoodtype`）、`halloweenmoonmutable_settings`（`{prefab, onmutatefn}`）、`lure_data`（海钓拟饵）、`issnowmandecor`（挂 `snowmandecor`）。本页四键只用到第 14、16 位，且都只给 `carrot` |

### 导出集合与端分界

一次 `MakeVeggie` 最多返回 7 个 Prefab，顺序为：`<name>_seeds` → `<name>_oversized` → `<name>_oversized_waxed` → `<name>_oversized_rotten`（以上四件仅 `has_seeds` 为真时插入）→ `<name>_dried`（仅 `VEGGIES[name].dryable ~= nil`）→ `<name>` → `<name>_cooked`（后两件无条件）。[导出段](../../../raw/current/scripts/prefabs/veggies.lua#L1084-L1099)

- `_seeds` 走独立的 `fn_seeds`：`farm_plant_seeds` bank／build、`edible` 食性 `SEEDS`、`stackable` 40、`perishable` ＝`PERISH_SUPERSLOW`、`cookable` 指向 `seeds_cooked`、`farmplantable.plant = "farm_plant_<name>"`。[种子工厂](../../../raw/current/scripts/prefabs/veggies.lua#L453-L537)
- `_dried` 只用 `dryable.build` 的动画名 `dried_<name>`；20 项里只有 `kelp` 带 `dryable`，故全族只有 `kelp_dried`，本页四键没有干制态。[干制工厂](../../../raw/current/scripts/prefabs/veggies.lua#L779-L833)
- 资产同样按条件拼：`anim/<name>.zip`＋`INV_IMAGE <name>` 必在，熟态另加 `INV_IMAGE <name>_cooked`，`_oversized_rot` 图标无条件追加，`has_seeds` 时再加 `oceanfishing_lure_mis.zip`，`_seeds` 共用 `seeds.zip`＋`farm_plant_seeds.zip`。[`assets`](../../../raw/current/scripts/prefabs/veggies.lua#L391-L451)

**端分界**在 `inst.entity:SetPristine()`：之前是双端都能读的公共端（实体／动画／物理、`pickupsound`、`cookable` 等优化用 tag、浮水组件），紧随其后的 `if not TheWorld.ismastersim then return inst end` 把客户端整体短路；此后所有 `AddComponent`、数值写入与事件监听只在主模拟执行。[生端分界](../../../raw/current/scripts/prefabs/veggies.lua#L589-L591)、[熟端分界](../../../raw/current/scripts/prefabs/veggies.lua#L734-L736)

### 生熟两态数值

时间基准 `total_day_time`＝480 秒（`seg_time`=30×16）、`perish_warp`=1，故天与秒可直接换算；饥饿基准 `calories_per_day`=75。[基准](../../../raw/current/scripts/tuning.lua#L20-L20)、[perish_warp](../../../raw/current/scripts/tuning.lua#L47-L47)、[PERISH_*](../../../raw/current/scripts/tuning.lua#L2043-L2051)、[热量与治疗](../../../raw/current/scripts/tuning.lua#L2058-L2062)

| 注册 | 饥饿／生命／理智 | 新鲜度 | 堆叠上限 |
|---|---|---|---|
| `carrot` | 12.5／1／0（`CALORIES_SMALL`／`HEALING_TINY`） | 10 天（4800 秒） | 40 |
| `carrot_cooked` | 12.5／3／0 | 6 天（2880 秒） | 40 |
| `corn` | 25／3／0（`CALORIES_MED`） | 10 天（4800 秒） | 40 |
| `corn_cooked` | 12.5／3／0（`CALORIES_SMALL`） | 15 天（7200 秒，`PERISH_SLOW`） | 40 |
| `pumpkin` | 37.5／3／0（`CALORIES_LARGE`） | 活动期 20 天（9600 秒）／否则 10 天（4800 秒） | 未设→组件默认 20 |
| `pumpkin_cooked` | 37.5／8／0（`HEALING_MEDSMALL`） | 6 天（2880 秒） | 40 |
| `potato` | 12.5／**−3**／**−5**（`−HEALING_SMALL`／`−SANITY_TINY`） | 10 天（4800 秒） | 40 |
| `potato_cooked` | 25／20／0（`CALORIES_MED`／`HEALING_MED`） | 6 天（2880 秒） | 40 |

浮水参数见[注册与构造](#注册与构造)。堆叠来源两条：`stackable` 构造时 `maxsize = TUNING.STACK_SIZE_MEDITEM`（20），生成器显式赋 `TUNING.STACK_SIZE_SMALLITEM`（40）；生成器对 `pumpkin`／`eggplant`／`durian`／`watermelon` 跳过赋值，熟态一律 40。[默认](../../../raw/current/scripts/components/stackable.lua#L23-L28)、[档位](../../../raw/current/scripts/tuning.lua#L81-L83)、[生成器例外](../../../raw/current/scripts/prefabs/veggies.lua#L613-L619)、[熟态](../../../raw/current/scripts/prefabs/veggies.lua#L752-L753)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：edible | 写 `healthvalue`／`hungervalue`／`sanityvalue`，食性 `FOODTYPE.VEGGIE`；赋值即触发 `badfood` 判定与 `edible_VEGGIE` 标签 | 生熟各一套；主模拟 | [[reference/components/edible#关键接口]]；[生](../../../raw/current/scripts/prefabs/veggies.lua#L595-L600)、[熟](../../../raw/current/scripts/prefabs/veggies.lua#L745-L750) |
| 组件：perishable | `SetPerishTime`＋`StartPerishing`，腐坏替换 `spoiled_food`；`fresh`／`stale`／`spoiled` 三标签由百分比切换 | 生用 `perishtime`、熟用 `cooked_perishtime` | [[reference/components/perishable#关键接口]]；[生](../../../raw/current/scripts/prefabs/veggies.lua#L602-L605)、[熟](../../../raw/current/scripts/prefabs/veggies.lua#L740-L743)、[标签](../../../raw/current/scripts/components/perishable.lua#L1-L27) |
| 组件：cookable | `product = name.."_cooked"`；`Cook` 生成熟件并把「已损失的新鲜度打五折」传给熟件（按熟件上限重算） | 仅生件；客户端只靠 `cookable` tag 识别 | [[reference/components/cookable#关键接口]]；[接线](../../../raw/current/scripts/prefabs/veggies.lua#L664-L665)、[Cook](../../../raw/current/scripts/components/cookable.lua#L18-L38) |
| 组件：stackable | 见[生熟两态数值](#生熟两态数值)；上限同步到 replica | 生熟不同档 | [[reference/components/stackable#关键接口]]；[生成器例外](../../../raw/current/scripts/prefabs/veggies.lua#L613-L619) |
| 组件：inventoryitem | 入栏与拾取落地；`pickupsound="vegetation_firm"` 只在生件公共端设置 | 生件；熟件未设 | [[reference/components/inventoryitem#关键接口]]；[`pickupsound`](../../../raw/current/scripts/prefabs/veggies.lua#L557-L557) |
| 组件：bait／tradable | `bait` 无参数挂载（被陷阱当诱饵时的搬运／被吃回调）；`tradable` 仅挂载、`goldvalue` 保持默认 0，故猪王拒收 | 生熟都有 | [[reference/components/bait#关键接口]]、[[reference/components/tradable#关键接口]]；[生](../../../raw/current/scripts/prefabs/veggies.lua#L657-L660)、[熟](../../../raw/current/scripts/prefabs/veggies.lua#L765-L768)、[猪王判定](../../../raw/current/scripts/prefabs/pigking.lua#L598-L606) |
| 组件：weighable | `type = TROPHYSCALE_TYPES.OVERSIZEDVEGGIES` 且不设重量；公共端预置 `weighable_OVERSIZEDVEGGIES` tag | 仅生件、且非 `SEEDLESS`；熟件无 | [[reference/components/weighable#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/veggies.lua#L645-L651)、[tag](../../../raw/current/scripts/prefabs/veggies.lua#L567-L570) |
| 组件：repairer | `repairmaterial = MATERIALS.CARROT`、`perishrepairpercent = 0.25` → 打上 `freshen_carrot` tag | 仅 `carrot`；修复对象见陷阱节 | [[reference/components/repairer#关键接口]]；[接线](../../../raw/current/scripts/prefabs/veggies.lua#L607-L611)、[tag 规则](../../../raw/current/scripts/components/repairer.lua#L8-L24) |
| 组件：burnable／hauntable | `MakeSmallBurnable`＋`MakeSmallPropagator` 可燃可蔓延、无 `fuel`；`MakeHauntableLaunchAndPerish` 只弹飞（腐坏分支已注释） | 生熟都有；主模拟 | [[reference/components/burnable#关键接口]]、[[reference/components/hauntable#关键接口]]；[可燃](../../../raw/current/scripts/standardcomponents.lua#L164-L174)、[作祟](../../../raw/current/scripts/standardcomponents.lua#L1311-L1337) |
| 组件：halloweenmoonmutable | `prefab_mutated = "carrat"`，被月亮精华液（`halloweenpotion_moon`）选中时整件替换成生物 | 仅 `carrot`；主模拟 | [[reference/components/halloweenmoonmutable#关键接口]]；[接线](../../../raw/current/scripts/prefabs/veggies.lua#L672-L677)、[Mutate](../../../raw/current/scripts/components/halloweenmoonmutable.lua#L28-L78) |
| 组件：timer／activatable | `Spin` 计时器与 `ACTIVATE` 入口，入包停转 | 仅 `carrot` | [[reference/components/timer#关键接口]]、[[reference/components/activatable#关键接口]]；[接线](../../../raw/current/scripts/prefabs/veggies.lua#L689-L702) |
| 组件：snowmandecor | 空标记组件，行为在 `componentactions` 里按 tag 接入雪人装饰 | 仅 `carrot`（本页四键） | [[reference/components/snowmandecor]]；[挂载](../../../raw/current/scripts/prefabs/veggies.lua#L679-L681) |
| 组件：floater | 经 `MakeInventoryFloatable` 的 `size`／`offset`／`scale` 三参控制水面漂浮 | 生熟各一套参数 | [[reference/components/floater#关键接口]]；[helper](../../../raw/current/scripts/standardcomponents.lua#L1772-L1791) |
| API／数据：`plant_def` | 农田侧取自 `PLANT_DEFS[name]` 的 `product`／`product_oversized`／`seed`／`build`，决定收获与巨型产物 | 主模拟；世界生成与农田共用 | [默认填充](../../../raw/current/scripts/prefabs/farm_plant_defs.lua#L254-L270)、[产品字段](../../../raw/current/scripts/prefabs/farm_plant_defs.lua#L265-L267) |
| 关联 Prefab：`carrot_planted` | 野生胡萝卜植株；`pickable:SetUp("carrot", 10)` 的 10 秒因 `remove_when_picked` 不生效，摘后由再生管理器按 `CARROT_REGROWTH_TIME`（`day_time`×20＝6000 秒）补种 | 森林／洞穴房间与世界生成摆放 | [carrot.lua](../../../raw/current/scripts/prefabs/carrot.lua#L35-L40)、[再生常量](../../../raw/current/scripts/tuning.lua#L3169-L3169)、[生成表](../../../raw/current/scripts/map/forest_map.lua#L186-L186) |
| 关联 Prefab：`carrot_spinner` | 旋转结束后附在胡萝卜上的特效；工厂把它写进 `prefabs` 依赖 | 仅 `carrot` | [依赖](../../../raw/current/scripts/prefabs/veggies.lua#L424-L426)、[生成](../../../raw/current/scripts/prefabs/veggies.lua#L377-L379) |
| 资源：anim 包 | `anim/<name>.zip` 内含 `idle`（生）与 `cooked`（熟）两个动画名；生件 bank／build＝`name`，熟件沿用同一 bank／build 只换动画名 | 8 个注册共用同两个动画名 | [动画设置](../../../raw/current/scripts/prefabs/veggies.lua#L553-L555) |

### 南瓜的活动期分支

`pumpkin` 的生鲜期是一个三元表达式：`IsSpecialEventActive(SPECIAL_EVENTS.HALLOWED_NIGHTS) and TUNING.PERISH_PRESERVED or TUNING.PERISH_MED`。它在 **`veggies.lua` 被 `LoadPrefabFile` 执行、`VEGGIES` 表构造的那一刻**求值一次：`IsSpecialEventActive` 读 `WORLD_SPECIAL_EVENT`／`WORLD_EXTRA_EVENTS`；这两个全局在 `PopulateWorld` 里由 `ApplySpecialEvent`／`ApplyExtraEvent` 按存档覆盖项改写（`default` 不改写；新进程为 `constants.lua` 的 `HALLOWED_NIGHTS`），但 prefab 文件在此前的 `LoadAssets` 已重载，构造时读到的是加载前的全局值：新进程即上述默认值；同进程内切换世界时读到什么未核验。[分支](../../../raw/current/scripts/prefabs/veggies.lua#L56-L56)、[判定](../../../raw/current/scripts/constants.lua#L1095-L1097)、[默认值](../../../raw/current/scripts/constants.lua#L858-L859)、[应用](../../../raw/current/scripts/simutil.lua#L635-L647)、[装载顺序](../../../raw/current/scripts/gamelogic.lua#L1041-L1042)、[PopulateWorld](../../../raw/current/scripts/gamelogic.lua#L364-L373)

该值在**每次加载世界重跑 prefab 文件时重新求值**；每个实例构造时用 `perishable:SetPerishTime(VEGGIES[name].perishtime)` 把当时数值抄进 `perishtime`／`perishremainingtime`，故切换活动事件不改已生成实例，也不改下一次加载前的取值。[抄写](../../../raw/current/scripts/prefabs/veggies.lua#L602-L605)、[`SetPerishTime`](../../../raw/current/scripts/components/perishable.lua#L188-L194)

### 胡萝卜的旋转、变异与修补

- **旋转**：生 `carrot` 在公共端加 `SoundEmitter` 并定义 `GetActivateVerb → "SPIN"`；主模拟把 `inst.Spin` 指到 `Carrot_StartSpinning`，再加 `timer` 与 `activatable`（`OnActivate` 调 `inst:Spin()`、`quickaction=true`），并注册 `timerdone`。玩家对地上的胡萝卜执行 `ACTIONS.ACTIVATE`（动作文本由 `GetActivateVerb` 给出）即开转：播 `spin_pre`／`spin_loop`、起 `spin` 计时器、循环音效；计时到点改八向朝向＋随机角度、播 `spin_pst`、生成并附着 `carrot_spinner`，同时把 `activatable.inactive` 复位。[函数组](../../../raw/current/scripts/prefabs/veggies.lua#L333-L387)、[接线](../../../raw/current/scripts/prefabs/veggies.lua#L689-L702)、[动作](../../../raw/current/scripts/actions.lua#L3095-L3113)、[候选](../../../raw/current/scripts/componentactions.lua#L142-L155)
- **谁能用**：除玩家外，兔人小游戏会直接调用 `carrot:Spin(5)`（`cozy_bunnyman` 的丢胡萝卜环节），可见 `Spin` 是可被其它实体复用的方法而非仅限动作。[调用](../../../raw/current/scripts/brains/cozy_bunnymanbrain.lua#L423-L444)
- **入包即停**：`inventoryitem:SetOnPutInInventoryFn(Carrot_StopSpinning)`——进任何容器／物品栏就清朝向、回 `idle`、停计时器、`KillAllSounds` 并让 `activatable` 重新可用；旋转途中入包同样不再残留 `spin_lp` 音效。[停转](../../../raw/current/scripts/prefabs/veggies.lua#L345-L361)、[挂接](../../../raw/current/scripts/prefabs/veggies.lua#L699-L699)
- **变异**：`halloweenmoonmutable` 让 `carrot` 成为可被月亮精华液转化的目标。`halloweenpotion_moon` 的 `Use` 找到带该组件的目标后调 `Mutate()`，后者 `SpawnPrefab("carrat")`、有容器时只对它调 `GiveItem`（不查返回值；`carrat` 有 `inventoryitem`，可入槽），无容器且标志开启时才推 `attacked`，并移除原件；`carrat`（胡萝卜鼠）是生物（掉落 `plantmeat` 100%、`carrot_seeds` 33%），所以被变异的胡萝卜不再回到物品形态。[药水](../../../raw/current/scripts/components/halloweenpotionmoon.lua#L11-L30)、[Mutate](../../../raw/current/scripts/components/halloweenmoonmutable.lua#L28-L78)、[掉落](../../../raw/current/scripts/prefabs/carrat.lua#L52-L56)
- **修补**：`repairer` 只声明 `repairmaterial = MATERIALS.CARROT`（`"carrot"`）与 `perishrepairpercent = 0.25`，没有生命／耐久修复值，因此它只能修「以 `MATERIALS.CARROT` 为材料、且靠 `perishable` 判损耗」的可修物；`prefabs/` 内的 `repairmaterial` 赋值中唯一匹配的是胡萝卜诱饵护甲 `armor_carrotlure`。每次成功修补按 `perishable:SetPercent(GetPercent() + 0.25)` 回鲜，且只在剩余新鲜度低于 95% 时才算「需要修」。[材料常量](../../../raw/current/scripts/constants.lua#L1944-L1951)、[可修物](../../../raw/current/scripts/prefabs/armor_carrotlure.lua#L114-L116)、[结算](../../../raw/current/scripts/components/repairable.lua#L106-L133)、[门槛](../../../raw/current/scripts/components/repairable.lua#L90-L104)

## 典型接入

### 统一堆叠上限并分别设定保质期

- 前提与选择：服务端 `modmain.lua`；两件是独立注册，必须分别 postinit，不能靠改数据表回灌。postinit 时物品满鲜，`SetNewMaxPerishTime` 与 `SetPerishTime` 结果相同；`SetNewMaxPerishTime` 保留比例要作用于已有剩余。
- 生效与撤销：postinit 在每次构造时运行；停用模组即恢复工厂原值。读档时 postinit 先设新上限，随后 `Perishable:OnLoad` 照抄存档剩余秒数；`GetPercent` 读数时在 1 封顶。[OnLoad](../../../raw/current/scripts/components/perishable.lua#L337-L349)、[GetPercent](../../../raw/current/scripts/components/perishable.lua#L215-L221)
- 原版依据：工厂在构造末尾才 `StartPerishing`，故 postinit 改上限时物品仍满鲜；两态的原始值见[生熟两态数值](#生熟两态数值)。[`SetNewMaxPerishTime`](../../../raw/current/scripts/components/perishable.lua#L204-L208)

```lua
-- modmain.lua：南瓜两态统一堆叠上限，保质期分别设 10／6 天
local function retune(prefab, maxsize, perishtime)
    AddPrefabPostInit(prefab, function(inst)
        if not GLOBAL.TheWorld.ismastersim then
            return
        end
        if inst.components.stackable ~= nil then
            inst.components.stackable.maxsize = maxsize
        end
        if inst.components.perishable ~= nil then
            inst.components.perishable:SetNewMaxPerishTime(perishtime)
        end
    end)
end

retune("pumpkin", GLOBAL.TUNING.STACK_SIZE_SMALLITEM, GLOBAL.TUNING.PERISH_MED)
retune("pumpkin_cooked", GLOBAL.TUNING.STACK_SIZE_SMALLITEM, GLOBAL.TUNING.PERISH_FAST)
```

- 调整与验证：单位为个与秒；改 `maxsize` 影响新构造实例。示例把堆叠统一到 40（生南瓜原为默认 20），保质期分别设为 10／6 天——活动期生南瓜的 20 天也会被改成 10 天。

## 常见陷阱与限制

- **边界：熟态不是生鲜态的全面升级**：`corn_cooked` 饥饿（12.5）低于 `corn`（25），`pumpkin_cooked` 与生南瓜持平，只有 `potato_cooked` 由负转正。[玉米](../../../raw/current/scripts/prefabs/veggies.lua#L53-L54)、[南瓜](../../../raw/current/scripts/prefabs/veggies.lua#L56-L58)、[土豆](../../../raw/current/scripts/prefabs/veggies.lua#L128-L130)
- **边界：生南瓜的 20 不是显式配置**：生成器只对 `pumpkin`／`eggplant`／`durian`／`watermelon` 跳过 `maxsize` 赋值，值来自 `stackable` 构造默认；改组件默认会连带改这四件生作物。[例外](../../../raw/current/scripts/prefabs/veggies.lua#L613-L619)、[默认](../../../raw/current/scripts/components/stackable.lua#L23-L28)
- **边界：活动期分支不追溯**：分支在 prefab 文件加载（`LoadAssets` 重跑）时求值并抄进每个实例；活动事件切换后旧物品仍按旧值腐坏；新建物品取当前工厂表里已算好的值，只有重跑 prefab 文件才会按当时活动重新求值。见[南瓜的活动期分支](#南瓜的活动期分支)。
- **误区：挂 `tradable` 就会被猪王收**：四键都没有 `goldvalue`／`halloweencandyvalue`，猪王 `AcceptTest` 要求 `goldvalue > 0`（或节日糖果、`pig_token`），交付会被拒。[默认 0](../../../raw/current/scripts/components/tradable.lua#L1-L7)、[判定](../../../raw/current/scripts/prefabs/pigking.lua#L598-L606)
- **边界：`perishable:SetPerishTime` 会把剩余时间拉回满值**（同时写 `perishtime` 与 `perishremainingtime`）；对已经在世界里的物品调用等于「回鲜」。要按比例改上限用 `SetNewMaxPerishTime`，要加时间用 `AddTime`。[实现](../../../raw/current/scripts/components/perishable.lua#L188-L194)、[`AddTime`](../../../raw/current/scripts/components/perishable.lua#L180-L186)
- **边界：腐坏对三维分别结算**：普通吃者的默认饥饿倍率为陈旧 `.667`／腐烂 `.5`；负生命／理智跳过腐坏折扣，腐烂时原本非负的理智改为 `-SANITY_SMALL`（−10）；若 `CanProcessSpoiledItem` 为真则把负值钳 0。[折扣](../../../raw/current/scripts/components/edible.lua#L96-L119)、[腐烂理智](../../../raw/current/scripts/components/edible.lua#L83-L85)、[钳零](../../../raw/current/scripts/components/edible.lua#L127-L128)
- **未核验**：quagmire 模式对所有蔬菜都会调用 `event_server_data("quagmire", "prefabs/veggies").master_postinit(inst)`，但该节日数据不在本快照的 `scripts` 根内，四键在 quagmire 下是否有额外行为未核验。[钩子](../../../raw/current/scripts/prefabs/veggies.lua#L683-L685)、[`event_server_data`](../../../raw/current/scripts/entityscript.lua#L77-L77)

## 验证记录

- 源码审阅：2026-11-08，Steam buildid 26335394。核对 `veggies.lua` 全文（`MakeVegStats` 位序、`VEGGIES` 四键、`fn`／`fn_cooked`、`fn_seeds`／`fn_dried`、导出与注册段）、`farm_plants.lua` 的 `MakePickable`／`SetupLoot`、`farm_plant_defs.lua` 的产物与种子填充、`carrot.lua` 全文，以及 `perishable`／`edible`／`cookable`／`stackable`／`bait`／`tradable`／`weighable`／`repairer`／`repairable`／`activatable`／`halloweenmoonmutable`／`pickable`／`regrowthmanager`／`inventory`／`eater` 的相关分支；`standardcomponents` 三个 helper、`tuning` 的基准与 `PERISH_*`／`CALORIES_*` 等常量、`constants` 的 `MATERIALS.CARROT`／`FOODTYPE`／`SPECIAL_EVENTS`、`actions`／`componentactions` 的 ACTIVATE、`simutil`／`gamelogic` 的活动事件覆盖与加载顺序、`pigking` 的 `AcceptTest`、`armor_carrotlure`、`bunnyman` 与 `carrat` 掉落、`halloweenpotion_moon`／`halloweenmoonmutable` 变异链、`cozy_bunnymanbrain`／`wormwood_carrat` 的胡萝卜环节、`map/layouts` 与 `map/forest_map` 的生成落点、简中 8 个名称键。
  名称核验：`NAMES.CARROT`=胡萝卜、`CARROT_COOKED`=烤胡萝卜、`CORN`=玉米、`CORN_COOKED`=爆米花、`PUMPKIN`=南瓜、`PUMPKIN_COOKED`=烤南瓜、`POTATO`=土豆、`POTATO_COOKED`=烤土豆。[CARROT](../../../raw/current/scripts/languages/chinese_s.po#L340655-L340657)、[CARROT_COOKED](../../../raw/current/scripts/languages/chinese_s.po#L340660-L340662)、[CORN](../../../raw/current/scripts/languages/chinese_s.po#L341690-L341692)、[CORN_COOKED](../../../raw/current/scripts/languages/chinese_s.po#L341695-L341697)、[PUMPKIN](../../../raw/current/scripts/languages/chinese_s.po#L347300-L347302)、[PUMPKIN_COOKED](../../../raw/current/scripts/languages/chinese_s.po#L347345-L347347)、[POTATO](../../../raw/current/scripts/languages/chinese_s.po#L347225-L347227)、[POTATO_COOKED](../../../raw/current/scripts/languages/chinese_s.po#L347245-L347247)
  未覆盖：`corn`／`potato`／`pumpkin` 的生成点检索只覆盖 `SpawnPrefab("<name>")`、`SetLoot`／`SetChanceLootTable` 直接引用、世界生成布局与房间表，未穷举动态选名、模组注入与活动脚本；`carrot_spinner` 表现、quagmire 节日数据、`_seeds` 与三件巨型变体不在本页。
- 游戏运行：未运行。三维、天数与秒数均为 tuning 推导（`CALORIES_SMALL`=12.5、`PERISH_MED`=10×480 秒），非运行测量。
- 分档：8 注册（4 键生熟）且承载 `MakeVegStats` 16 位参数、导出集合与南瓜／胡萝卜分支等跨文件协作，需按族整体说明，取 `complex` 档（480 行／64 KiB）。
- 结构检查：`validate_page.py` 通过（186 行／32,694 字节，standard 档内）；`validate_source_links.py --strict` 0 错 0 警（125 处锚点）。

## 相关链接

- 同一工厂的另外两类：[[reference/prefabs/farm-crops-fruit]]（蔬果 6 键及熟态）、[[reference/prefabs/farm-crops-spice]]（辛香 4 键及熟态）
- 巨型形态与打蜡：[[reference/prefabs/giant-crops]]
- 种植与生长：[[reference/components/crop]]、[[reference/components/farmplantable]]、[[reference/components/farming-manager]]

返回 [[reference/prefabs/index|Prefab 索引]]。
- 2026-11-08 build 26474873：本页随版本迁移升版（机械沿用；引用区间未受本次源码行号位移影响），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-07）。
