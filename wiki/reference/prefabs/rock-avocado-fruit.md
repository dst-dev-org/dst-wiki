---
title: "石果与多肉植物（成熟石果、熟石果、发芽的石果与两个多肉注册）"
created: "2026-11-08"
updated: "2026-11-08"
type: prefab
summary: "成熟／熟／发芽石果与两个同名多肉注册的构造、三维与鲜度；石果丛 4 阶段、外壳逐次采掘的 65/1/34 产出、发芽石果的种植链，以及多肉两态的入栏边界。"
aliases: ["rock_avocado_fruit_ripe", "rock_avocado_fruit_ripe_cooked", "rock_avocado_fruit_sprout", "succulent_plant", "succulent_picked", "rock_avocado_fruit", "rock_avocado_bush", "成熟石果", "熟石果", "发芽的石果", "多肉植物", "石果", "石果灌木丛"]
tags: [prefab, item, food]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/rock_avocado_fruit.lua", "prefabs/rock_avocado_bush.lua", "prefabs/succulent_plant.lua", "prefabs/oasislake.lua", "prefabs/plantables.lua", "components/pickable.lua", "components/workable.lua", "components/growable.lua", "components/edible.lua", "components/perishable.lua", "components/stackable.lua", "components/deployable.lua", "components/dryable.lua", "components/dryer.lua", "components/witherable.lua", "components/timer.lua", "stategraphs/SGwilson.lua", "prefabutil.lua", "standardcomponents.lua", "tuning.lua", "constants.lua", "cooking.lua", "recipes.lua", "map/static_layouts/oasis.lua", "map/rooms/forest/terrain_moonisland.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 石果与多肉植物（成熟石果、熟石果、发芽的石果与两个多肉注册）

## 快速判断

- 职责与适用：本页覆盖两个文件的五个指定注册——`rock_avocado_fruit_ripe`、`rock_avocado_fruit_ripe_cooked`、`rock_avocado_fruit_sprout`、`succulent_plant` 与 `succulent_picked`（后两者简中同名“多肉植物”）。另补记 `rock_avocado_fruit` 与 `rock_avocado_bush` 两个世界／中间态注册。[注册段](../../../raw/current/scripts/prefabs/rock_avocado_fruit.lua#L432-L437)、[多肉注册](../../../raw/current/scripts/prefabs/succulent_plant.lua#L145-L146)
- 优先使用：查三维／鲜度看[数值与获得](#数值与获得)；查石果从丛到果、再采矿分裂的链路看[世界丛、采集中间态与种植链](#世界丛采集中间态与种植链)；查同名多肉的入栏差异直接看注册表与陷阱节。
- 重要副作用：石果灌木丛一次采摘产出 **1 叠 3 个** `rock_avocado_fruit`，不是 3 个独立实体；移栽后立即进入 barren，必须施肥才恢复。[三果一叠](../../../raw/current/scripts/prefabs/rock_avocado_bush.lua#L287-L300)、[移栽](../../../raw/current/scripts/prefabs/rock_avocado_bush.lua#L215-L217)
- 修改边界：只讲这五个物品注册及必要的来源宿主；石果灌木丛的完整世界生成、盆栽多肉 `succulent_potted`、干多肉 `succulent_picked_dried` 的后续用途只作关系说明。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `rock_avocado_fruit_ripe` | `rock_avocado_fruit_ripe()`：bank `rock_avo_fruit_master`／build `rock_avocado_fruit_build`，播 `idle_split_open`；加 `cookable`／`molebait` 标签、`MakeInventoryPhysics`、默认 `MakeInventoryFloatable`，然后 `SetPristine` → 主模拟 | 主模拟：`scrapbook_anim`、`edible`（`HEALING_TINY`／`CALORIES_SMALL`／0／`VEGGIE`）、`bait`、`perishable`（`PERISH_SUPERFAST`→`spoiled_food`）、`stackable`（40）、`inspectable`、`inventoryitem`（图像 `rock_avocado_fruit_ripe`）、`tradable`、`cookable`（→`rock_avocado_fruit_ripe_cooked`）、`burnable`／`propagator`、`MakeHauntableLaunchAndPerish`。[构造](../../../raw/current/scripts/prefabs/rock_avocado_fruit.lua#L170-L229)、[三维](../../../raw/current/scripts/prefabs/rock_avocado_fruit.lua#L198-L209) |
| `rock_avocado_fruit_ripe_cooked` | `rock_avocado_fruit_ripe_cooked()`：同 bank／build，播 `cooked`；默认浮水；`SetPristine` → 主模拟 | 主模拟：`edible`（`HEALING_SMALL`／`CALORIES_SMALL`／0／`VEGGIE`）、`perishable`（`PERISH_TWO_DAY`）、`stackable`（40）、`inspectable`、`inventoryitem`（图像 `..._cooked`）、`tradable`、`burnable`／`propagator`、`MakeHauntableLaunchAndPerish`；无 `cookable`、无 `bait`。[构造](../../../raw/current/scripts/prefabs/rock_avocado_fruit.lua#L232-L281) |
| `rock_avocado_fruit_sprout` | `rock_avocado_fruit_sprout()`：bank `rock_avo_fruit_master`／build `rock_avocado_fruit_build`，播 `idle_seed`；`MakeInventoryFloatable("small", nil, 0.4)`；加 `deployedplant` 标签；`SetPristine` → 主模拟 | 主模拟：`stackable`（40）、`inspectable`、`inventoryitem`（图像 `rock_avocado_fruit_sprout`）、`tradable`、`deployable`（PLANT，`on_deploy_fn`）、`burnable`／`propagator`、`MakeHauntableLaunchAndIgnite`；**没有** `edible`、`perishable`。[构造](../../../raw/current/scripts/prefabs/rock_avocado_fruit.lua#L383-L429)、[部署接线](../../../raw/current/scripts/prefabs/rock_avocado_fruit.lua#L420-L422) |
| `succulent_plant` | `plantfn()`：bank／build `succulent`，播 `idle`；只加 `succulent` 标签；`SetPristine` → 主模拟 | 主模拟：`inspectable`、`pickable`（`SetUp("succulent_picked")`、`remove_when_picked=true`、`quickpick=true`）、`burnable`／`propagator`、`MakeHauntableIgnite`、`OnSave`／`OnLoad` 保存 `plantid`；没有 `inventoryitem`、没有 `deployable`、没有 `growable`。[世界态构造](../../../raw/current/scripts/prefabs/succulent_plant.lua#L43-L81)、[采摘接线](../../../raw/current/scripts/prefabs/succulent_plant.lua#L62-L73) |
| `succulent_picked` | `invfn()`：bank／build `succulent_picked`，播 `idle`；加 `cattoy`／`dryable` 标签、`MakeInventoryFloatable("med", nil, 0.8)`；`SetPristine` → 主模拟 | 主模拟：`stackable`（40）、`tradable`、`inspectable`、`fuel`（`TINY_FUEL`）、`burnable`（`TINY_BURNTIME`）／`propagator`、`inventoryitem`、`edible`（0 饥饿／`HEALING_TINY`／`VEGGIE`）、`perishable`（`PERISH_FAST`）、`dryable`（→`succulent_picked_dried`，`DRY_FAST`）、`MakeHauntableLaunchAndIgnite`。[物品态构造](../../../raw/current/scripts/prefabs/succulent_plant.lua#L84-L142) |

### 名称核验

`rock_avocado_fruit_ripe`＝成熟石果、`rock_avocado_fruit_ripe_cooked`＝熟石果、`rock_avocado_fruit_sprout`＝发芽的石果；`succulent_plant` 与 `succulent_picked` 两个键的 msgstr 都是**多肉植物**，这是同名不同形态的根源。世界宿主 `rock_avocado_fruit`＝石果、`rock_avocado_bush`＝石果灌木丛、`dug_rock_avocado_bush`＝石果灌木丛；`rock_avocado_fruit_sprout_sapling` 没有独立 `NAMES` 键，靠 `SetPrefabNameOverride("ROCK_AVOCADO_FRUIT_SPROUT")` 显示为发芽的石果。[RIPE](../../../raw/current/scripts/languages/chinese_s.po#L348715-L348717)、[COOKED](../../../raw/current/scripts/languages/chinese_s.po#L348720-L348722)、[SPROUT](../../../raw/current/scripts/languages/chinese_s.po#L348725-L348727)、[SUCCULENT_PLANT](../../../raw/current/scripts/languages/chinese_s.po#L350115-L350117)、[SUCCULENT_PICKED](../../../raw/current/scripts/languages/chinese_s.po#L350105-L350107)、[FRUIT](../../../raw/current/scripts/languages/chinese_s.po#L348710-L348712)、[BUSH](../../../raw/current/scripts/languages/chinese_s.po#L348705-L348707)、[DUG_BUSH](../../../raw/current/scripts/languages/chinese_s.po#L342340-L342342)

## 世界丛、采集中间态与种植链

### 石果灌木丛（世界态）

`rock_avocado_bush` 是四阶段 `growable`：`stage_1`（空，`EMPTY` 2*day_time±2*seg_time）、`stage_2`（`PREPICK` 6*seg_time±2*seg_time）、`stage_3`（可采，`PICK` 3*day_time±2*seg_time）、`stage_4`（`CRUMBLE` 1*day_time±2*seg_time），`loopstages=true` 循环，且 `springgrowth=true`。[阶段表](../../../raw/current/scripts/prefabs/rock_avocado_bush.lua#L89-L114)、[循环设置](../../../raw/current/scripts/prefabs/rock_avocado_bush.lua#L306-L312)

- 只有 `stage_3` 的 `set_stage3` 把产品设为 `rock_avocado_fruit` 并 `Regen()`；`stage_1`／`stage_2` 清空产品并置 `canbepicked=false`，`stage_4` 也清空产品，但不可采时会 `Regen()` 保持可点。[stage3](../../../raw/current/scripts/prefabs/rock_avocado_bush.lua#L57-L63)、[stage1／2／4](../../../raw/current/scripts/prefabs/rock_avocado_bush.lua#L33-L78)
- `pickable.numtoharvest = 3`，产品走单 prefab 分支：生成一个 `rock_avocado_fruit` 后 `SetStackSize(3)`，所以是一次 1 叠 3 个；摘后 `onpickedfn` 把 growable 设回 `stage_1`。[数量](../../../raw/current/scripts/prefabs/rock_avocado_bush.lua#L287-L300)、[单产物叠加](../../../raw/current/scripts/components/pickable.lua#L493-L501)、[摘后阶段](../../../raw/current/scripts/prefabs/rock_avocado_bush.lua#L165-L181)
- 野生丛的 `transplanted=false`，`Pick` 不扣 `cycles_left`；移栽时 `plantables` 调 `pickable:OnTransplant()`，`ontransplantfn` 立即 `MakeBarren()`，所以移栽后是枯死态，需施肥恢复；`Fertilize` 把 `cycles_left` 重置为 `max_cycles`（3）并 `MakeEmpty` 重启生长。[扣次数条件](../../../raw/current/scripts/components/pickable.lua#L535-L539)、[移栽回调](../../../raw/current/scripts/prefabs/plantables.lua#L17-L30)、[barren](../../../raw/current/scripts/components/pickable.lua#L259-L290)
- 挖起：`workable(DIG)` 完成回调先看 `witherable:IsWithered()` 或 `pickable:IsBarren()`——两者任一为真只掉 2 个 `twigs`，**不返还灌木丛**；否则 `stage_3` 时额外喷 3 个石果，再给 `dug_rock_avocado_bush`，最后删除本体。[挖起](../../../raw/current/scripts/prefabs/rock_avocado_bush.lua#L142-L162)
- 着火的丛在 `stage_3` 被烧毁时会先喷 3 个石果，再 `StopGrowing` 并走 `DefaultBurntFn`；注释说明石果本身是“rock hard”，不会被烧掉。[烧毁](../../../raw/current/scripts/prefabs/rock_avocado_bush.lua#L124-L135)
- 世界生成在月岛房间的 `distributeprefabs` 里（`rock_avocado_bush` 权重 0.1／1.0）；移栽件由 `plantables` 的 `dug_rock_avocado_bush` 条目部署回 `rock_avocado_bush`。[月岛](../../../raw/current/scripts/map/rooms/forest/terrain_moonisland.lua#L82-L89)、[可移栽条目](../../../raw/current/scripts/prefabs/plantables.lua#L162-L168)

### 石果外壳（采集中间态）

`rock_avocado_fruit` 是“石果外壳”，也是唯一能直接入栏的石果前体：`stackable` 40、`inventoryitem` 把图像换成 `rock_avocado_fruit_rockhard` 并 `SetSinks(true)`（会沉水），同时挂 `workable(MINE)`；`workleft = ROCK_FRUIT_MINES * stacksize`，并在 `stacksizechange` 时同步。[构造](../../../raw/current/scripts/prefabs/rock_avocado_fruit.lua#L114-L167)、[沉水与图像](../../../raw/current/scripts/prefabs/rock_avocado_fruit.lua#L138-L145)

- 采掘回调是 `SetOnWorkCallback(on_mine)`，不是完成回调：每命中一次就按 `ceil(workdone / ROCK_FRUIT_MINES)` 取果、生成战利品，再从堆叠里 `Get(n)` 后删除；`ROCK_FRUIT_MINES=1` 时普通 1 点工作量消耗 1 个外壳，结果还会 clamp 到当前堆叠数（多段工作或调高 `ROCK_FRUIT_MINES` 时按 `ceil` 合并结算）。[挂接](../../../raw/current/scripts/prefabs/rock_avocado_fruit.lua#L149-L153)、[采掘主体](../../../raw/current/scripts/prefabs/rock_avocado_fruit.lua#L45-L98)
- 每次采掘独立掷一次：`RIPE_CHANCE=0.65` → 成熟石果；`SEED_CHANCE=0.01` 且按累计概率 `0.65+0.01` 判断 → 发芽的石果；剩余 0.34 → `rocks`。[概率](../../../raw/current/scripts/prefabs/rock_avocado_fruit.lua#L60-L72)、[常量](../../../raw/current/scripts/tuning.lua#L3800-L3809)
- 它自己的 `edible` 只显式设 `foodtype=ELEMENTAL` 与 `hungervalue=2`，`healthvalue` 未赋值，沿用组件默认 **10**，`sanityvalue` 默认 0；所以不能按“石头”推断它为零收益。[edible 默认值](../../../raw/current/scripts/components/edible.lua#L19-L44)、[外壳接线](../../../raw/current/scripts/prefabs/rock_avocado_fruit.lua#L155-L158)

### 发芽石果的种植链

`rock_avocado_fruit_sprout` 走 `deployable` 的 PLANT 模式；`on_deploy_fn` 从堆叠取一件、删除原物品，在落点 `SpawnPrefab("rock_avocado_fruit_sprout_sapling")` 并播放种树音效。[部署回调](../../../raw/current/scripts/prefabs/rock_avocado_fruit.lua#L374-L381)、[PLANT 模式](../../../raw/current/scripts/prefabs/rock_avocado_fruit.lua#L420-L422)

- 树苗用 `timer("grow")`，时长 `TUNING.ROCK_FRUIT_SPROUT_GROWTIME = 5*day_time`＝1500 秒；计时器到点后播 `seed_growth`，`animover` 时删除树苗并在原位 `SpawnPrefab("rock_avocado_bush")`。[计时器](../../../raw/current/scripts/prefabs/rock_avocado_fruit.lua#L299-L311)、[长成](../../../raw/current/scripts/prefabs/rock_avocado_fruit.lua#L289-L296)、[常量](../../../raw/current/scripts/tuning.lua#L3810-L3810)
- 树苗被点燃时 `StopTimer("grow")` 直接删除计时，熄灭时 `StartTimer` 新建、从满 1500 秒重新计（不是暂停后续算）；被 `DIG` 时只掉回 1 个 `rock_avocado_fruit_sprout` 并删除，不返还外壳。[燃烧接线](../../../raw/current/scripts/prefabs/rock_avocado_fruit.lua#L362-L369)、[挖起](../../../raw/current/scripts/prefabs/rock_avocado_fruit.lua#L284-L286)、[掉落表](../../../raw/current/scripts/prefabs/rock_avocado_fruit.lua#L351-L357)
- 树苗固定 `SetDeploySmartRadius(DEPLOYSPACING_RADIUS[DEPLOYSPACING.DEFAULT] / 2)`＝1，并 `SetPrefabNameOverride("ROCK_AVOCADO_FRUIT_SPROUT")`，所以世界上的树苗与物品同名“发芽的石果”。[树苗初始化](../../../raw/current/scripts/prefabs/rock_avocado_fruit.lua#L320-L342)
- 长出的 `rock_avocado_bush` 在构造时 `SetStage(math.random(1, 4))`，因此新丛可能是空丛、预采态或直接成熟；不保证种下即结果。[随机阶段](../../../raw/current/scripts/prefabs/rock_avocado_bush.lua#L306-L312)

### 多肉的世界态与物品态

`succulent_plant` 只在世界侧存在：`pickable:SetUp("succulent_picked")`、`remove_when_picked=true`、`quickpick=true`，没有 `inventoryitem` 也没有 `deployable`，所以它**不能进物品栏、不能移栽**；摘取动作走 `quickpick` 的短动作，产物直接进采摘者物品栏（或落地），摘完整株 `Remove()`。[采摘构造](../../../raw/current/scripts/prefabs/succulent_plant.lua#L62-L73)、[remove_when_picked](../../../raw/current/scripts/components/pickable.lua#L577-L579)、[短动作](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1133-L1135)

`succulent_picked` 是唯一的物品态：可堆叠 40、可入栏、可干制（`succulent_picked_dried`，`DRY_FAST`＝480 秒）或作燃料（`TINY_FUEL`＝7.5）；它没有 `deployable`，不能把多肉直接种回地里，盆栽多肉是另一个配方产物 `succulent_potted`（2 个多肉＋1 个石砖，科学二本）。[物品态](../../../raw/current/scripts/prefabs/succulent_plant.lua#L116-L138)、[盆栽配方](../../../raw/current/scripts/recipes.lua#L736-L736)

世界多肉由绿洲湖生成：湖实体在半径 `SUCCULENT_RANGE=15` 内补到 `MAX_SUCCULENTS=18` 株，落点至少离水 `WATER_RADIUS+0.5`，且 2 格内无实体；`oasis.lua` 静态布局把 `oasislake` 放在绿洲。[生成](../../../raw/current/scripts/prefabs/oasislake.lua#L55-L77)、[布局](../../../raw/current/scripts/map/static_layouts/oasis.lua#L88-L95)

## 数值与获得

时间基准：`seg_time`＝30 秒、`day_time`＝300 秒、`total_day_time`＝480 秒；生长常量用 `day_time`，鲜度常量用 `total_day_time`，换算差 1.6 倍，不能把“天”直接套到生长常量上。[基准](../../../raw/current/scripts/tuning.lua#L19-L27)、[鲜度表](../../../raw/current/scripts/tuning.lua#L2043-L2051)

| 注册 | 饥饿／生命／理智 | 食性 | 新鲜度 | 堆叠 | 漂浮／沉水 |
|---|---|---|---|---|---|
| `rock_avocado_fruit_ripe` | 12.5／1／0（`CALORIES_SMALL`／`HEALING_TINY`） | `VEGGIE` | `PERISH_SUPERFAST`＝3 天＝1440 秒 | 40 | `small` 浮水 |
| `rock_avocado_fruit_ripe_cooked` | 12.5／3／0（`HEALING_SMALL`） | `VEGGIE` | `PERISH_TWO_DAY`＝2 天＝960 秒 | 40 | `small` 浮水 |
| `rock_avocado_fruit_sprout` | —（无 `edible`） | — | —（无 `perishable`） | 40 | `small`，scale 0.4 |
| `succulent_picked` | 0／1／0（`HEALING_TINY`） | `VEGGIE` | `PERISH_FAST`＝6 天＝2880 秒 | 40 | `med`，scale 0.8 |
| `rock_avocado_fruit`（外壳） | 2／默认 10／0 | `ELEMENTAL` | 无 | 40 | 沉水（`SetSinks(true)`） |

`rock_avocado_fruit_ripe` 的 3 天名叫 `PERISH_SUPERFAST`，比熟态的 2 天更长；熟制提升的是生命（1→3），不是保质。外壳的生命 10 来自 `edible` 构造默认，不是显式配置。[常量](../../../raw/current/scripts/tuning.lua#L2043-L2051)、[外壳默认](../../../raw/current/scripts/components/edible.lua#L19-L44)

### 获得路径

| 注册 | 已核验来源 | 产出与证据 |
|---|---|---|
| `rock_avocado_fruit` | ①石果灌木丛 `stage_3` 采摘，1 叠 3 个；②丛着火时 `stage_3` 喷 3 个；③挖起 `stage_3` 的丛时喷 3 个 | ①[产品](../../../raw/current/scripts/prefabs/rock_avocado_bush.lua#L287-L300)、②[烧毁](../../../raw/current/scripts/prefabs/rock_avocado_bush.lua#L124-L135)、③[挖起](../../../raw/current/scripts/prefabs/rock_avocado_bush.lua#L142-L162) |
| `rock_avocado_fruit_ripe` | 采掘外壳时 65% 掷中；外壳本身来自上述三条 | [概率](../../../raw/current/scripts/prefabs/rock_avocado_fruit.lua#L60-L72) |
| `rock_avocado_fruit_ripe_cooked` | 仅 `cookable.product`：把成熟石果烤制 | [接线](../../../raw/current/scripts/prefabs/rock_avocado_fruit.lua#L221-L222) |
| `rock_avocado_fruit_sprout` | ①采掘外壳时 1% 掷中；②挖起已种下的发芽石果树苗 | ①[概率](../../../raw/current/scripts/prefabs/rock_avocado_fruit.lua#L60-L72)、②[树苗掉落](../../../raw/current/scripts/prefabs/rock_avocado_fruit.lua#L351-L357) |
| `succulent_plant` | 绿洲湖生成，最多 18 株；检索范围内未见第二种生成点 | [生成](../../../raw/current/scripts/prefabs/oasislake.lua#L55-L77) |
| `succulent_picked` | 摘取世界多肉；另可作盆栽配方原料、干制与燃料 | [采摘](../../../raw/current/scripts/prefabs/succulent_plant.lua#L62-L73)、[干制](../../../raw/current/scripts/prefabs/succulent_plant.lua#L134-L138)、[燃料](../../../raw/current/scripts/prefabs/succulent_plant.lua#L116-L117) |

料理用途：`rock_avocado_fruit_ripe` 在 `cooking.lua` 登记 `veggie=1`（`cancook=true` 同时生成熟态条目）；外壳、发芽石果与两个多肉没有 `AddIngredientValues` 登记，不能按“食物”假设可下锅。[食材登记](../../../raw/current/scripts/cooking.lua#L140-L140)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：pickable | 丛：product／numtoharvest／阶段切换／移栽与施肥；多肉：一次性采摘 | 丛 `transplanted` 才扣 cycles；多肉 `remove_when_picked` 无再生 | [[reference/components/pickable#关键接口]]；[丛接线](../../../raw/current/scripts/prefabs/rock_avocado_bush.lua#L287-L312)、[多肉接线](../../../raw/current/scripts/prefabs/succulent_plant.lua#L62-L68) |
| 组件：growable | 丛的四阶段循环、春季倍率、施肥后的重启 | 只挂世界丛；树苗用 timer 而非 growable | [[reference/components/growable#关键接口]]；[阶段表](../../../raw/current/scripts/prefabs/rock_avocado_bush.lua#L89-L114)、[启动](../../../raw/current/scripts/prefabs/rock_avocado_bush.lua#L306-L312) |
| 组件：workable | 丛 `DIG`、外壳 `MINE`；都是每次 work 回调而非完成回调 | 丛挖起 1 次；外壳按堆叠数给 workleft | [[reference/components/workable#关键接口]]；[丛](../../../raw/current/scripts/prefabs/rock_avocado_bush.lua#L280-L285)、[外壳](../../../raw/current/scripts/prefabs/rock_avocado_fruit.lua#L149-L153) |
| 组件：deployable | 发芽石果 PLANT 部署；多肉没有部署 | 仅发芽石果；主模拟执行 `on_deploy_fn` | [[reference/components/deployable#关键接口]]；[接线](../../../raw/current/scripts/prefabs/rock_avocado_fruit.lua#L420-L422) |
| 组件：edible | 成熟／熟／多肉／外壳的饥饿、生命与食性 | 外壳默认生命 10；发芽石果无 edible | [[reference/components/edible#关键接口]]；[熟](../../../raw/current/scripts/prefabs/rock_avocado_fruit.lua#L198-L202)、[多肉](../../../raw/current/scripts/prefabs/succulent_plant.lua#L124-L127) |
| 组件：perishable | 成熟／熟／多肉腐坏→`spoiled_food`；外壳与发芽石果无 | 1440／960／2880 秒 | [[reference/components/perishable#关键接口]]；[`SetPerishTime`](../../../raw/current/scripts/components/perishable.lua#L188-L194) |
| 组件：stackable | 五个物品 40；外壳堆叠变化同步 workleft | 全部主模拟 | [[reference/components/stackable#关键接口]]；[外壳同步](../../../raw/current/scripts/prefabs/rock_avocado_fruit.lua#L108-L112) |
| 组件：dryable／fuel | 多肉干制为 `succulent_picked_dried`（1 天）、作燃料 7.5 | 仅物品态 | [[reference/components/dryable#关键接口]]、[[reference/components/fuel#关键接口]]；[干制](../../../raw/current/scripts/prefabs/succulent_plant.lua#L134-L138) |
| 组件：timer（树苗侧） | 发芽石果树苗用 `"grow"` 计时器，5*day_time | 点燃 `StopTimer` 删除计时、熄灭 `StartTimer` 从满 1500 秒重计；DIG 返还 | [[reference/components/timer#关键接口]]；[计时器](../../../raw/current/scripts/prefabs/rock_avocado_fruit.lua#L299-L311) |
| 组件：witherable／burnable | 丛可枯萎、可燃；烧毁按阶段给果 | 世界丛；主模拟 | [[reference/components/witherable#关键接口]]、[[reference/components/burnable#关键接口]]；[烧毁](../../../raw/current/scripts/prefabs/rock_avocado_bush.lua#L124-L135) |
| 关联 Prefab：`rock_avocado_bush` | 丛由世界生成、发芽石果长成或挖起件部署 | 世界态；不在本页注册表 | Wiki 待建；[注册](../../../raw/current/scripts/prefabs/rock_avocado_bush.lua#L328-L328)、[部署](../../../raw/current/scripts/prefabs/plantables.lua#L17-L30) |
| 关联 Prefab：`dug_rock_avocado_bush`／`rock_avocado_fruit_sprout_sapling` | 前者是移栽件，后者是发芽石果种下后的世界形态 | 均由 `plantables`／`on_deploy_fn` 生成 | Wiki 待建；[移栽件](../../../raw/current/scripts/prefabs/plantables.lua#L162-L168)、[树苗](../../../raw/current/scripts/prefabs/rock_avocado_fruit.lua#L378-L379) |
| 资源：`anim/rock_avocado_fruit.zip`／`anim/rock_avocado_fruit_build.zip`／`anim/rock_avocado.zip` | 果、树苗、丛与图标共用 | 原版资源，不随页分发 | [果 assets](../../../raw/current/scripts/prefabs/rock_avocado_fruit.lua#L3-L36)、[丛 assets](../../../raw/current/scripts/prefabs/rock_avocado_bush.lua#L1-L7) |

## 典型接入

### 让石果采掘只出成熟石果

- 前提与选择：服务端 `modmain.lua` 直接改 `TUNING.ROCK_FRUIT_LOOT`。`on_mine` 每次采掘都重新读这个表，所以改后对新采掘立即生效；不需要 postinit 每个外壳实例。
- 生效与撤销：只影响之后的外壳采掘；已存在的成熟／发芽石果不受影响。停用模组后新采掘回到默认。别把 `SEED_CHANCE` 只设 0 而保留 `RIPE_CHANCE<1` 就以为不会掉岩石——剩余概率仍归 `rocks`。
- 原版依据：判定按 `loot_roll < RIPE_CHANCE`，否则 `< RIPE_CHANCE+SEED_CHANCE` 给发芽，否则给 `rocks`。[判定](../../../raw/current/scripts/prefabs/rock_avocado_fruit.lua#L60-L72)

```lua
-- modmain.lua：每次采掘都产成熟石果，不再掉发芽石果或岩石
GLOBAL.TUNING.ROCK_FRUIT_LOOT.RIPE_CHANCE = 1
GLOBAL.TUNING.ROCK_FRUIT_LOOT.SEED_CHANCE = 0
```

- 调整与验证：若要保留 1% 发芽率，把 `RIPE_CHANCE` 设为 0.99、`SEED_CHANCE` 保持 0.01；采掘一次看掉落 prefab，注意 `ROCK_FRUIT_MINES=1`，普通 1 点工作量消耗一个外壳（结算按 `ceil(workdone/ROCK_FRUIT_MINES)` 并 clamp 到堆叠）。

## 常见陷阱与限制

- **误区：两个“多肉植物”都能入栏。** `succulent_plant` 没有 `inventoryitem`、没有 `deployable`，摘取即整株删除；只有 `succulent_picked` 能进物品栏、堆叠、干制或做盆栽。两者同名，检索时看注册名而不是看中文名。[世界态](../../../raw/current/scripts/prefabs/succulent_plant.lua#L43-L81)、[物品态](../../../raw/current/scripts/prefabs/succulent_plant.lua#L84-L142)
- **边界：发芽石果不是食物。** 它没有 `edible` 也没有 `perishable`，三维与鲜度都不适用；唯一用途是种下或挖回。[构造](../../../raw/current/scripts/prefabs/rock_avocado_fruit.lua#L383-L429)
- **边界：熟石果比成熟石果烂得快。** 生命从 1 升到 3，但鲜度从 3 天降到 2 天；不要按“熟制更耐放”推断。[常量](../../../raw/current/scripts/tuning.lua#L2043-L2051)
- **边界：移栽石果丛先枯死。** `plantables` 部署后立即 `OnTransplant`→`MakeBarren`，必须施肥；施肥把 `cycles_left` 重置为 3，之后每摘一次扣 1，扣到 0 再枯死。枯死或枯萎时挖起只给 2 个树枝，灌木丛本体不返还。[移栽](../../../raw/current/scripts/prefabs/plantables.lua#L22-L24)、[施肥](../../../raw/current/scripts/components/pickable.lua#L274-L290)、[挖起损失](../../../raw/current/scripts/prefabs/rock_avocado_bush.lua#L142-L149)
- **边界：采摘不重置 growable 计时。** `onpickedfn` 只 `SetStage(1)`；stage3 的计时器仍在跑，到点后进入 stage2，再过 `PREPICK` 的 180±60 秒才回到 stage3。下一次成熟不是从采摘时刻固定重算 900 秒。[摘后阶段](../../../raw/current/scripts/prefabs/rock_avocado_bush.lua#L165-L181)、[阶段表](../../../raw/current/scripts/prefabs/rock_avocado_bush.lua#L89-L114)
- **边界：外壳的 10 点生命是默认值。** `rock_avocado_fruit` 只显式设 `hungervalue=2` 与 `foodtype=ELEMENTAL`；`healthvalue` 沿用 `edible` 构造默认 10。改组件默认会连带改它。[默认](../../../raw/current/scripts/components/edible.lua#L19-L44)、[外壳](../../../raw/current/scripts/prefabs/rock_avocado_fruit.lua#L155-L158)
- **边界：外壳是逐次采掘而不是“敲碎后结算”。** `SetOnWorkCallback` 在普通 1 点工作量后扣 1 个并掷战利品（`ceil(workdone/ROCK_FRUIT_MINES)`，再 clamp 到堆叠）；堆叠 40 时 workleft 也是 40，不是 1。[回调](../../../raw/current/scripts/prefabs/rock_avocado_fruit.lua#L149-L153)

## 验证记录

- 源码审阅：2026-11-08，Steam buildid 26335394。核对 `rock_avocado_fruit.lua` 全文（外壳、成熟、熟、发芽、树苗、placer 与五个返回注册）、`rock_avocado_bush.lua` 全文（阶段、采摘、挖起、烧毁、移栽、初始随机阶段）、`succulent_plant.lua` 全文、`oasislake.lua` 的生成段、`plantables.lua` 的 `ondeploy` 与 `rock_avocado_bush` 条目，以及 `pickable`／`workable`／`growable`／`edible`／`perishable`／`stackable`／`deployable`／`dryable`／`dryer`／`witherable`／`timer`／`burnable` 的相关分支；`tuning` 的 `ROCK_FRUIT_*`／`PERISH_*`／`CALORIES_SMALL`／`HEALING_*`／`STACK_SIZE_SMALLITEM`；`recipes` 的盆栽配方、`cooking` 的 `veggie=1` 登记、月岛与绿洲布局。
  名称核验：`languages/chinese_s.po` 的 `NAMES.ROCK_AVOCADO_FRUIT_RIPE`＝成熟石果、`..._RIPE_COOKED`＝熟石果、`..._SPROUT`＝发芽的石果、`SUCCULENT_PLANT`＝多肉植物、`SUCCULENT_PICKED`＝多肉植物；补核 `ROCK_AVOCADO_FRUIT`＝石果、`ROCK_AVOCADO_BUSH`／`DUG_ROCK_AVOCADO_BUSH`＝石果灌木丛；`rock_avocado_fruit_sprout_sapling` 无独立 `NAMES` 键，走 `SetPrefabNameOverride`。[po 锚点](#名称核验)
  未覆盖：`succulent_potted`、`succulent_picked_dried` 与冬青茶等下游；石果丛世界生成的完整概率与房间权重；模组动态选名；`rock_avocado_fruit` 的爆炸/作祟表现未运行验证。
- 游戏运行：未运行。三维、天数、秒数与阶段区间均为 tuning 推导（`day_time`＝300 秒、`total_day_time`＝480 秒）。
- 结构检查：`validate_page.py` 通过（standard 档内）；`validate_source_links.py --strict` 0 错 0 警。Lua 示例已单独做语法解析。
- 复核：2026-11-08，Steam buildid 26474873。核对本页引用源在 26335394→26474873 间的差异（`tuning.lua`、`plantables.lua` 各插入 1 行，`SGwilson.lua` 仅换声效），锚点按新树重定位、结论未变；上一条 26335394 审阅记录保留作历史。

## 相关链接

- 采掘与采摘：[[reference/components/pickable]]、[[reference/components/workable]]、[[reference/components/growable]]
- 可食与鲜度：[[reference/components/edible]]、[[reference/components/perishable]]、[[reference/components/stackable]]
- 干制与燃料：[[reference/components/dryable]]、[[reference/components/fuel]]
- 同批的桦栗果与巨石枝种子：[[reference/prefabs/acorn]]

返回 [[reference/prefabs/index|Prefab 索引]]。
