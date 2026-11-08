---
title: "桦栗果与巨石枝种子（acorn、acorn_cooked、tree_rock_seed 与两个放置器）"
created: "2026-11-08"
updated: "2026-11-09"
type: prefab
summary: "acorn.lua 与 tree_rock_seed.lua 的五个注册：三种可食物品的三维／鲜度／堆叠，PLANT 与 CUSTOM 两条种植校验、放置预览与树苗生长链，以及桦栗树、巨石枝树、巨大洞穴蠕虫掉落等获得路径。"
aliases: ["acorn", "acorn_cooked", "acorn_placer", "tree_rock_seed", "tree_rock_seed_placer", "桦栗果", "烤桦栗果", "巨石枝种子"]
tags: [prefab, item, food]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/acorn.lua", "prefabs/tree_rock_seed.lua", "prefabs/planted_tree.lua", "prefabs/deciduoustrees.lua", "prefabs/tree_rocks.lua", "prefabs/leafpile.lua", "prefabs/birchnutdrake.lua", "prefabs/catcoon.lua", "prefabs/spoiledfood.lua", "prefabs/worm_boss.lua", "prefabs/winter_treestand.lua", "components/deployable.lua", "components/inventoryitem_replica.lua", "components/playercontroller.lua", "components/workable.lua", "components/edible.lua", "components/perishable.lua", "components/stackable.lua", "components/cookable.lua", "components/timer.lua", "components/winter_treeseed.lua", "prefabutil.lua", "standardcomponents.lua", "components/map.lua", "tilemanager.lua", "tiledefs.lua", "tuning.lua", "constants.lua", "actions.lua", "componentactions.lua", "cooking.lua", "recipes.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 桦栗果与巨石枝种子（acorn、acorn_cooked、tree_rock_seed 与两个放置器）

## 快速判断

- 职责与适用：本页覆盖两个文件返回的五个注册——可食种子 `acorn`、其烤制态 `acorn_cooked`、巨石枝种子 `tree_rock_seed`，以及只负责放置预览的 `acorn_placer`、`tree_rock_seed_placer`。`acorn` 走 `DEPLOYMODE.PLANT`，`tree_rock_seed` 走 `DEPLOYMODE.CUSTOM`，这是两者最大的行为差异。[acorn 注册](../../../raw/current/scripts/prefabs/acorn.lua#L193-L195)、[tree_rock_seed 注册](../../../raw/current/scripts/prefabs/tree_rock_seed.lua#L106-L107)
- 优先使用：要改食物数值查[数值与获得](#数值与获得)；要改种植校验或预览查[种植、放置预览与生长](#种植放置预览与生长)；冬季树架用桦栗果另有一套入口，见[关联机制导航](#关联机制导航)。
- 重要副作用：`acorn`／`tree_rock_seed` 种下时会消耗堆叠中的一件并把原物品删除；桦栗果种下时还会安抚半径 30 内第一个可见的怪物桦栗树；两种树苗被挖起时默认只掉树枝，不退种子。[消耗与删件](../../../raw/current/scripts/prefabs/acorn.lua#L30-L35)、[挖苗掉落](../../../raw/current/scripts/prefabs/planted_tree.lua#L101-L103)
- 修改边界：只讲这两个物品的构造、种植与获得；宿主树（`deciduoustree`、`tree_rock1/2`）的生长、砍伐、再生与掉落细节只作关系说明（见 [[reference/prefabs/deciduous-trees]]、[[reference/prefabs/tree-rocks]]），不展开世界生成或树苗页。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `acorn` | `fn()`：bank／build 均为 `acorn`、播 `idle`；公共端加 `deployedplant`／`icebox_valid`／`cattoy`／`show_spoilage`／`treeseed`／`cookable` 标签与 `MakeInventoryFloatable("small", 0.15)`，随后 `SetPristine` → 主模拟 | 主模拟依次挂 `cookable`（product=`acorn_cooked`）、`tradable`、`perishable`（`PERISH_PRESERVED`→`spoiled_food`）、`edible`（`CALORIES_TINY`／`HEALING_TINY`／`FOODTYPE.RAW`）、`stackable`（40）、`inspectable`、`burnable`／`propagator`、`inventoryitem`、`deployable`（PLANT）、`winter_treeseed`（`winter_deciduoustree`）、`forcecompostable`，最后 `MakeHauntableLaunchAndIgnite`；另设 `OnLoad`。[构造段](../../../raw/current/scripts/prefabs/acorn.lua#L67-L137)、[组件段](../../../raw/current/scripts/prefabs/acorn.lua#L98-L133) |
| `acorn_cooked` | `cooked()`：沿用 `acorn` bank／build，播 `cooked`；`MakeInventoryFloatable("small", 0.1)`；公共端不再加 `treeseed`／`cookable` 等标签 | 只挂 `edible`（食性 `SEEDS`，`secondaryfoodtype=ROUGHAGE`，仍是 `CALORIES_TINY`／`HEALING_TINY`）、`tradable`、`perishable`（`PERISH_FAST`）、`stackable`（40）、`inspectable`、`burnable`／`propagator`、`inventoryitem`、`forcecompostable`、`MakeHauntableLaunch`；无 `cookable`、无 `deployable`、无 `winter_treeseed`。[熟态构造](../../../raw/current/scripts/prefabs/acorn.lua#L140-L190)、[食性](../../../raw/current/scripts/prefabs/acorn.lua#L162-L166) |
| `acorn_placer` | `MakePlacer("acorn_placer","acorn","acorn","idle_planted")`；返回独立 Prefab | 仅放置预览：`CLASSIFIED`／`NOCLICK`／`placer` 标签、`persists=false`、无网络，播 `idle_planted`；不是实体物品。[工厂](../../../raw/current/scripts/prefabutil.lua#L1-L64)、[注册](../../../raw/current/scripts/prefabs/acorn.lua#L193-L195) |
| `tree_rock_seed` | `fn()`：bank／build 均为 `tree_rock_seed`、播 `idle`；公共端加 `deployedplant`／`icebox_valid`／`cattoy`／`show_spoilage`／`treeseed`，`MakeInventoryFloatable("small", 0.15)`，在 `SetPristine` **之前**写 `_custom_candeploy_fn` | 主模拟挂 `inspectable`／`inventoryitem`／`tradable`／`perishable`（`PERISH_MED`）／`edible`（RAW）／`stackable`（40）／`deployable`（CUSTOM，spacing=`PLACER_DEFAULT`）／`forcecompostable`／`burnable`／`propagator`／`MakeHauntableLaunchAndIgnite`；`winter_treeseed` 两行被注释掉，故没有冬季树架入口。[构造段](../../../raw/current/scripts/prefabs/tree_rock_seed.lua#L41-L103)、[组件段](../../../raw/current/scripts/prefabs/tree_rock_seed.lua#L70-L101) |
| `tree_rock_seed_placer` | `MakePlacer("tree_rock_seed_placer","tree_rock_seed","tree_rock_seed","idle_planted")` | 与实物同 bank／build／动画名；仅预览，无库存与持久化。[注册](../../../raw/current/scripts/prefabs/tree_rock_seed.lua#L106-L107) |

### 名称核验

五个注册中，`acorn` 的简中实显是**桦栗果**，不是橡果；`acorn_cooked` 是**烤桦栗果**；`tree_rock_seed` 是**巨石枝种子**。两个 placer 在名称定位器中未命中 `NAMES` 键，标题中的注册名即源码名。宿主与产物名另核：`TREE_ROCK`＝巨石枝、`acorn_sapling`＝桦栗树树苗、`tree_rock_sapling`＝巨石枝树苗。[ACORN](../../../raw/current/scripts/languages/chinese_s.po#L338495-L338497)、[ACORN_COOKED](../../../raw/current/scripts/languages/chinese_s.po#L338500-L338502)、[TREE_ROCK_SEED](../../../raw/current/scripts/languages/chinese_s.po#L350585-L350587)、[TREE_ROCK](../../../raw/current/scripts/languages/chinese_s.po#L350575-L350577)、[ACORN_SAPLING](../../../raw/current/scripts/languages/chinese_s.po#L338505-L338507)、[TREE_ROCK_SAPLING](../../../raw/current/scripts/languages/chinese_s.po#L350580-L350582)

## 种植、放置预览与生长

### 两条部署校验

部署动作先走物品 `deployable:CanDeploy`；成功后 `Deploy` 调 `ondeploy`，并在 `deployedplant` 标签存在时向世界推 `itemplanted`。[CanDeploy／Deploy](../../../raw/current/scripts/components/deployable.lua#L104-L146)

| 注册 | 模式与校验 | 结果 |
|---|---|---|
| `acorn` | `DEPLOYMODE.PLANT`（值 4）→ `TheWorld.Map:CanDeployPlantAtPoint` | 要求 `CanPlantAtPoint`：地块属于陆地且不在 `GROUND_HARD`，再要求放置点周围空位；间距用 `DEPLOYSPACING_RADIUS[DEPLOYSPACING.DEFAULT]`＝2。[PLANT 分支](../../../raw/current/scripts/components/deployable.lua#L113)、[地块判定](../../../raw/current/scripts/components/map.lua#L167-L174)、[硬地清单](../../../raw/current/scripts/tiledefs.lua#L563-L563) |
| `tree_rock_seed` | `DEPLOYMODE.CUSTOM`（值 7）→ 物品自带的 `_custom_candeploy_fn` | 只查陆地、非临时地块与放置点空位，**不查** `GROUND_HARD`，因此能种在道路／岩石等硬质地面上；间距用 `DEPLOYSPACING_RADIUS[DEPLOYSPACING.PLACER_DEFAULT]`＝3.2。[CUSTOM 分支](../../../raw/current/scripts/components/deployable.lua#L124-L128)、[自定义校验](../../../raw/current/scripts/prefabs/tree_rock_seed.lua#L24-L38)、[间距常量](../../../raw/current/scripts/constants.lua#L2254-L2262) |

`GROUND_HARD` 由 `tiledefs.lua` 的 `hard = true` 逐块登记，含 `ROAD`、`ROCKY`、`UNDERROCK`、`VENT` 等；`CanPlantAtPoint` 用 `not GROUND_HARD[tile]` 拒绝它们。[登记表](../../../raw/current/scripts/tilemanager.lua#L171-L173)、[道路](../../../raw/current/scripts/tiledefs.lua#L380-L380)、[岩地](../../../raw/current/scripts/tiledefs.lua#L563-L563)、[洞穴岩层](../../../raw/current/scripts/tiledefs.lua#L850-L850)、[喷发口](../../../raw/current/scripts/tiledefs.lua#L874-L874)

### 放置预览

客户端按 `<物品 prefab>_placer`（或 `overridedeployplacername`）生成预览实体；`placer.testfn` 每帧调用该物品 replica 的 `CanDeploy`，后者在主模拟上直接转给 `deployable:CanDeploy`、在客户端按 `deploymode` 走同名地图校验。[名字约定](../../../raw/current/scripts/components/inventoryitem_replica.lua#L360-L368)、[生成与 testfn](../../../raw/current/scripts/components/playercontroller.lua#L2893-L2911)、[replica 校验](../../../raw/current/scripts/components/inventoryitem_replica.lua#L327-L337)

两个 placer 都只提供 `idle_planted` 动画：`acorn_placer` 用 `acorn` bank／build，`tree_rock_seed_placer` 用 `tree_rock_seed` bank／build；预览实体的 `persists=false` 且带 `NOCLICK`，所以不能拾取、不存档。[MakePlacer](../../../raw/current/scripts/prefabutil.lua#L1-L64)

### 种下之后

| 种子 | 生成的树苗 | 实际生长计时 | 长成的树 |
|---|---|---|---|
| `acorn` | `acorn_sapling`（`startgrowing` 在构造时已启动 timer） | 树苗的 `inst.growtimes` 为 `nil`，回落到 `TUNING.PINECONE_GROWTIME`：`0.75*day_time`±`0.25*day_time`＝225±75 秒（150–300 秒） | `deciduoustree`，生成后调用 `growfromseed()`；树苗默认挖掉只给 `twigs`。[回退](../../../raw/current/scripts/prefabs/planted_tree.lua#L86-L92)、[长树](../../../raw/current/scripts/prefabs/planted_tree.lua#L72-L79)、[注册](../../../raw/current/scripts/prefabs/planted_tree.lua#L185-L185) |
| `tree_rock_seed` | `tree_rock_sapling`（tag `treerock`） | 该注册显式传 `TUNING.TREE_ROCK.SAPLING_GROW_TIME`：`4*day_time`±`1*day_time`＝1200±300 秒（900–1500 秒） | 从 `{"tree_rock1_short","tree_rock2_short"}` 随机一个，长出后同样 `growfromseed()`；挖掉默认给 `twigs`。[注册行](../../../raw/current/scripts/prefabs/planted_tree.lua#L190-L190)、[常量](../../../raw/current/scripts/tuning.lua#L8926-L8926) |

- **acorn 的 `ACORN_GROWTIME` 没有参与生长**：`ondeploy` 里虽然算了 `GetRandomWithVariance(TUNING.ACORN_GROWTIME.base, ...)` 并传给局部 `plant(pt, growtime)`，但 `plant` 的第二个形参从未被使用，实际只调 `sapling:StartGrowing()`；全快照内除定义与该处计算外没有 `ACORN_GROWTIME` 消费者。[计算](../../../raw/current/scripts/prefabs/acorn.lua#L34-L35)、[未用形参](../../../raw/current/scripts/prefabs/acorn.lua#L16-L21)、[常量](../../../raw/current/scripts/tuning.lua#L814-L814)
- **读档分支缺写入方**：`acorn` 的 `OnLoad` 只有拿到 `data.growtime` 才 `plant(inst, data.growtime)`，但该 prefab 没有 `OnSave` 写入 `growtime`；且该分支不像 `ondeploy` 那样先删除种子，一旦被外部写入会同时留下种子和树苗。[OnLoad](../../../raw/current/scripts/prefabs/acorn.lua#L61-L64)
- **树苗不是 `growable`**：两种树苗都用 `timer` 的 `"grow"` 计时器与 `timerdone` 长大，不挂 `growable`；`growable` 属于宿主树自己的生长。挖苗走 `workable(ACTIONS.DIG)` 的完成回调 `digup`。[计时器与挖苗](../../../raw/current/scripts/prefabs/planted_tree.lua#L150-L162)、[构造段](../../../raw/current/scripts/prefabs/planted_tree.lua#L106-L177)
- **安抚怪物树**：`acorn` 种下后以 `{"birchnut","monster"}` 为必须标签、排除 `stump`／`burnt`／FX 等，在半径 `TUNING.DECID_MONSTER_ACORN_CHILL_RADIUS`＝30 内找第一个可见目标；命中后取消其 `monster_start_task`，并在 0–3 秒后调用 `StopMonster`（前提是 `ent.monster` 为真且未在燃烧、未成桩）。[安抚段](../../../raw/current/scripts/prefabs/acorn.lua#L28-L58)、[半径](../../../raw/current/scripts/tuning.lua#L2922-L2922)、[StopMonster](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L874-L896)

## 数值与获得

时间基准：`seg_time`＝30 秒、`day_time`＝300 秒（10 个白天段）、`total_day_time`＝480 秒；生长常量用 `day_time`，鲜度常量用 `total_day_time`，两者不能互换。[时间基准](../../../raw/current/scripts/tuning.lua#L19-L27)、[`perish_warp`](../../../raw/current/scripts/tuning.lua#L47-L47)

| 注册 | 饥饿／生命／理智 | 食性 | 新鲜度 | 堆叠 | 漂浮 |
|---|---|---|---|---|---|
| `acorn` | 9.375／1／0（`CALORIES_TINY`／`HEALING_TINY`） | `FOODTYPE.RAW` | `PERISH_PRESERVED`＝20 天＝9600 秒 | 40 | `small`，垂直偏移 0.15 |
| `acorn_cooked` | 9.375／1／0（同名常量） | `FOODTYPE.SEEDS`＋次要 `ROUGHAGE` | `PERISH_FAST`＝6 天＝2880 秒 | 40 | `small`，垂直偏移 0.1 |
| `tree_rock_seed` | 9.375／1／0 | `FOODTYPE.RAW` | `PERISH_MED`＝10 天＝4800 秒 | 40 | `small`，垂直偏移 0.15 |

三者的 `sanityvalue` 都未显式赋值，沿用 `edible` 构造默认 0；生命与饥饿均为正，故不会触发 `badfood` 标签。堆叠上限都写作 `TUNING.STACK_SIZE_SMALLITEM`＝40。[默认三维](../../../raw/current/scripts/components/edible.lua#L19-L44)、[`badfood` 判定](../../../raw/current/scripts/components/edible.lua#L1-L7)、[堆叠常量](../../../raw/current/scripts/tuning.lua#L83-L83)

### 获得路径

| 注册 | 已核验来源 | 产出与证据 |
|---|---|---|
| `acorn` | ①砍倒桦栗树（`deciduoustree`）：普通／红／橙／黄四种 build 的 `tall_loot` 都含 1 个 `acorn`；②第 3 阶段且 `leaf_state=="colorful"` 的树倒下时额外 `SpawnLootPrefab("acorn")`；③毒桦栗 build 的 `normal_loot`／`short_loot`／`tall_loot` 直接含 1–3 个；④烧毁的树有 0.1 概率 | ①[builds](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L48-L54)、②[额外掉落](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L603-L617)、③[毒树表](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L100-L109)、④[烧毁表](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L668-L669) |
| `acorn`（补充） | ⑤`birchnutdrake`（桦栗果精）的随机掉落 0.4；⑥落叶堆 `leafpile` 的 DEBRIS 表 0.2；⑦浣猫（`catcoon`）礼物池含 `acorn` | ⑤[随机掉表](../../../raw/current/scripts/prefabs/birchnutdrake.lua#L114-L117)、⑥[落叶堆](../../../raw/current/scripts/prefabs/leafpile.lua#L98-L101)、⑦[猫礼物](../../../raw/current/scripts/prefabs/catcoon.lua#L116-L117) |
| `acorn_cooked` | 仅由 `cookable.product` 指向：把 `acorn` 放上烹饪锅／营火烤制；检索范围内未见世界生成或独立掉落 | [接线](../../../raw/current/scripts/prefabs/acorn.lua#L98-L99) |
| `tree_rock_seed` | ①巨石枝树采掘耗尽时掉 `tree_rock1_mine` 表（种子权重 1.00）；②崩落破坏时掉 `tree_rock1_mine_break` 表（同为 1.00）；③巨大洞穴蠕虫（`worm_boss`）的普通掉落表给权重 2；④烧毁态的石块会让该项按 `BURNED_LOOT_OVERRIDES` 变成 `ash` | ①[表](../../../raw/current/scripts/prefabs/tree_rocks.lua#L41-L51)、[耗尽掉落](../../../raw/current/scripts/prefabs/tree_rocks.lua#L397-L402)、②[崩落表](../../../raw/current/scripts/prefabs/tree_rocks.lua#L548-L549)、③[虫王表](../../../raw/current/scripts/prefabs/worm_boss.lua#L96-L106) |

用途补充：`acorn` 在 `cooking.lua` 登记 `seed=1`，并可作冬季树架的种子——`winter_treeseed` 组件指向 `winter_deciduoustree`，对 `winter_treestand` 执行 `PLANT` 会推 `plantwintertreeseed`，由树架生成冬季桦栗树；`tree_rock_seed` 的同名组件被注释，故没有该入口。另有两条配方用到桦栗果：落叶林地皮（`turf_deciduous`，1 树枝＋1 桦栗果）与饕餮馅饼（`wintercooking_tourtiere`，1 冬季盛宴燃料＋1 桦栗果＋1 松果）。[料理食材](../../../raw/current/scripts/cooking.lua#L143-L144)、[冬季树架生成](../../../raw/current/scripts/prefabs/winter_treestand.lua#L22-L29)、[动作分发](../../../raw/current/scripts/actions.lua#L2632-L2646)、[组件动作](../../../raw/current/scripts/componentactions.lua#L2019-L2022)、[草皮配方](../../../raw/current/scripts/recipes.lua#L710-L710)、[大餐配方](../../../raw/current/scripts/recipes.lua#L1447-L1447)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：deployable | 决定种植模式、间距、`CanDeploy` 与 `ondeploy`；`onmode`／`onspacing` 同步 replica | `acorn`＝PLANT；`tree_rock_seed`＝CUSTOM＋PLACER_DEFAULT；仅主模拟持有组件，客户端用 replica 校验 | [[reference/components/deployable#关键接口]]；[组件](../../../raw/current/scripts/components/deployable.lua#L31-L82)、[acorn 接线](../../../raw/current/scripts/prefabs/acorn.lua#L123-L125) |
| 组件：inventoryitem／replica | 入栏、落地、图片名与 deploy 查询；`GetDeployPlacerName` 决定预览 prefab | 三件都有；均未调用 `SetSinks` | [[reference/components/inventoryitem#关键接口]]；[replica 部署](../../../raw/current/scripts/components/inventoryitem_replica.lua#L327-L368) |
| 组件：edible | 写饥饿／生命／食性；正数不触发 `badfood` | `acorn`／`tree_rock_seed` 为 RAW；`acorn_cooked` 为 SEEDS＋ROUGHAGE | [[reference/components/edible#关键接口]]；[默认值](../../../raw/current/scripts/components/edible.lua#L19-L44) |
| 组件：perishable | `SetPerishTime`＋`StartPerishing`，腐坏替换 `spoiled_food`；鲜度标签在阈值处切换 | 20／6／10 天三档；仅主模拟 | [[reference/components/perishable#关键接口]]；[`SetPerishTime`](../../../raw/current/scripts/components/perishable.lua#L188-L194)、[标签](../../../raw/current/scripts/components/perishable.lua#L1-L27) |
| 组件：cookable | `product="acorn_cooked"`，客户端只靠 tag 识别 | 仅 `acorn` | [[reference/components/cookable#关键接口]]；[接线](../../../raw/current/scripts/prefabs/acorn.lua#L98-L99) |
| 组件：stackable | 40 上限；`ondeploy` 用 `stackable:Get()` 只取一件 | 三件都有；组件默认 20 被显式覆盖为 40 | [[reference/components/stackable#关键接口]]；[默认](../../../raw/current/scripts/components/stackable.lua#L23-L34)、[构造](../../../raw/current/scripts/prefabs/acorn.lua#L113-L114) |
| 组件：timer／workable（树苗侧） | 树苗用 `timer("grow")` 长大，用 `workable(DIG)` 挖起；不是本页物品自身的组件 | 仅 `acorn_sapling`／`tree_rock_sapling` | [[reference/components/timer#关键接口]]、[[reference/components/workable#关键接口]]；[树苗构造](../../../raw/current/scripts/prefabs/planted_tree.lua#L146-L162) |
| 组件：winter_treeseed | 仅 `acorn` 挂载并设 `winter_deciduoustree`；给冬季树架提供树 prefab | `tree_rock_seed` 的挂载被注释 | [[reference/components/winter_treeseed#关键接口]]；[acorn](../../../raw/current/scripts/prefabs/acorn.lua#L127-L128)、[注销](../../../raw/current/scripts/prefabs/tree_rock_seed.lua#L93-L94) |
| 组件：burnable／propagator／hauntable | 三件都可燃、可蔓延；`MakeHauntableLaunchAndIgnite` 或 `MakeHauntableLaunch` 决定作祟效果 | 熟态只弹飞；生种子也只抛飞（点火分支已注释），树苗的 `MakeHauntableIgnite` 同样被注释，作祟都不会点燃 | [[reference/components/burnable#关键接口]]、[[reference/components/hauntable#关键接口]]；[可燃](../../../raw/current/scripts/standardcomponents.lua#L164-L173)、[作祟](../../../raw/current/scripts/standardcomponents.lua#L1191-L1217) |
| 组件：forcecompostable | `brown=true`，作为堆肥棕色材料 | 三件都有 | [[reference/components/forcecompostable]]；[acorn](../../../raw/current/scripts/prefabs/acorn.lua#L130-L131) |
| 关联 Prefab：`acorn_sapling` | 种下后生成；tag `deciduoustree`，长成 `deciduoustree` | 仅 `acorn` 的 deploy；主模拟生成 | 树见 [[reference/prefabs/deciduous-trees]]；[注册](../../../raw/current/scripts/prefabs/planted_tree.lua#L185-L185) |
| 关联 Prefab：`tree_rock_sapling` | 种下后生成；tag `treerock`，长成 `tree_rock1_short`／`tree_rock2_short` | 仅 `tree_rock_seed` 的 deploy | 树见 [[reference/prefabs/tree-rocks]]；[注册](../../../raw/current/scripts/prefabs/planted_tree.lua#L190-L190) |
| 关联 Prefab：`spoiled_food` | 三个可腐坏物品的 `onperishreplacement` | 主模拟腐坏时生成 | Wiki 待建；[定义](../../../raw/current/scripts/prefabs/spoiledfood.lua#L263-L263)、[acorn 接线](../../../raw/current/scripts/prefabs/acorn.lua#L103-L106) |
| 资源：`anim/acorn.zip`／`anim/tree_rock_seed.zip` | 物品、树苗与 placer 共用的动画包；熟态只换 `cooked` 动画 | 原版资源，不随页分发 | [acorn assets](../../../raw/current/scripts/prefabs/acorn.lua#L3-L6)、[种子 assets](../../../raw/current/scripts/prefabs/tree_rock_seed.lua#L3-L6) |

## 典型接入

### 只改桦栗树苗的生长时间

- 前提与选择：这是服务端 `modmain.lua` 的 postinit。不能改 `TUNING.ACORN_GROWTIME`——它不参与实际计时；也不能全局改 `PINECONE_GROWTIME`，否则松果一起变。树苗构造时已启动计时器，所以 postinit 里要停掉再按新 `growtimes` 重启。
- 生效与撤销：只影响新种下的 `acorn_sapling`。已存档树苗读档时由计时器恢复 `"grow"` 的剩余时间，晚于 postinit，所以示例的翻倍不追溯已存在的树苗；停用模组后新建树苗回到 `PINECONE_GROWTIME`。`inst.growtimes` 的字段由 `startgrowing` 读取，单位是秒。
- 原版依据：`StartGrowing` 在无 `growtimes` 时才回落到 `PINECONE_GROWTIME`；`startgrowing` 在已有 `"grow"` 计时器时直接返回。[回退](../../../raw/current/scripts/prefabs/planted_tree.lua#L86-L92)、[构造时启动](../../../raw/current/scripts/prefabs/planted_tree.lua#L146-L152)

```lua
-- modmain.lua：把桦栗树苗的生长时间改成 2 倍（不影响松果）
AddPrefabPostInit("acorn_sapling", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    inst.growtimes = {
        base = 2 * GLOBAL.TUNING.PINECONE_GROWTIME.base,
        random = 2 * GLOBAL.TUNING.PINECONE_GROWTIME.random,
    }
    if inst.components.timer ~= nil then
        inst.components.timer:StopTimer("grow")
        inst:StartGrowing()
    end
end)
```

- 调整与验证：示例把基准从 225 秒改成 450 秒、随机量从 75 秒改成 150 秒；实测时看 `timer` 的 `"grow"` 剩余与长出的树 prefab，而不是看 `ACORN_GROWTIME`。

## 常见陷阱与限制

- **误区：改 `ACORN_GROWTIME` 会改桦栗树苗。** 实际计时读的是 `PINECONE_GROWTIME`；`ACORN_GROWTIME` 在 `plant` 中被丢弃。要只改桦栗树，用上一节的 `growtimes`。
- **边界：树苗挖掉不退种子。** 两种树苗的 `lootdropper` 都用 `overrideloot or {"twigs"}`，两个注册都没有传 `overrideloot`，所以挖苗只得树枝。[默认掉落](../../../raw/current/scripts/prefabs/planted_tree.lua#L156-L157)
- **边界：`acorn` 不能种在硬质地面上。** PLANT 模式的 `CanPlantAtPoint` 拒绝 `GROUND_HARD`（道路、岩地、洞穴岩层、喷发口等）；`tree_rock_seed` 的 CUSTOM 校验只查陆地与临时地块，反而可以。[PLANT](../../../raw/current/scripts/components/map.lua#L167-L174)、[CUSTOM](../../../raw/current/scripts/prefabs/tree_rock_seed.lua#L24-L38)
- **边界：烤过的桦栗果不能种。** `acorn_cooked` 没有 `deployable`、`treeseed` 标签与 `cookable`；它只是食物。[熟态构造](../../../raw/current/scripts/prefabs/acorn.lua#L140-L190)
- **边界：巨石枝种子的可燃掉落被改写成灰。** `BURNED_LOOT_OVERRIDES` 把 `tree_rock_seed` 映射为 `ash`，注释说明巨石枝的战利品受保护、不应被烧掉；这不等同于种子自身有防火属性。[覆盖表](../../../raw/current/scripts/tuning.lua#L4511-L4529)
- **未核验**：`acorn` 的 `OnLoad` 分支在正常存档流程中不可达（本文件无 `OnSave` 写入 `growtime`）；若模组外部写该字段，才会触发，且不会移除种子。未运行游戏验证该异常路径。

## 验证记录

- 源码审阅：2026-11-08，Steam buildid 26335394。核对 `acorn.lua`、`tree_rock_seed.lua` 全文，`planted_tree.lua` 的 `sapling_fn`／`growtree`／`startgrowing`／`digup` 与两条注册，`deciduoustrees.lua` 的 builds／倒下额外桦栗果／烧毁表，`tree_rocks.lua` 的采掘表与耗尽／崩落分支，`worm_boss.lua` 掉落表，`winter_treestand.lua` 的 `plantwintertreeseed`，以及 `deployable`／`inventoryitem_replica`／`playercontroller`／`map`／`tiledefs`／`edible`／`perishable`／`stackable`／`cookable`／`winter_treeseed`／`workable` 的相关分支；`tuning` 的 `ACORN_GROWTIME`／`PINECONE_GROWTIME`／`TREE_ROCK.SAPLING_GROW_TIME`／`PERISH_*`／`CALORIES_TINY`／`HEALING_TINY`／`STACK_SIZE_SMALLITEM`／`DECID_MONSTER_ACORN_CHILL_RADIUS`；`cooking` 与 `recipes` 的用途。
  名称核验：`languages/chinese_s.po` 的 `NAMES.ACORN`＝桦栗果、`ACORN_COOKED`＝烤桦栗果、`TREE_ROCK_SEED`＝巨石枝种子、`TREE_ROCK`＝巨石枝、`ACORN_SAPLING`＝桦栗树树苗、`TREE_ROCK_SAPLING`＝巨石枝树苗；三个物品注册各有唯一 `NAMES` 键；两个 placer 未命中 `NAMES` 键。[po 锚点](#名称核验)
  未覆盖：模组注入、动态选名与活动脚本；宿主树的砍伐／再生与树苗页；`acorn` 的异常 `OnLoad` 路径未运行验证；`worm_boss` 掉落表只核对权重，未核验实际战斗掉落概率。
- 游戏运行：未运行。三维、天数、秒数与生长区间均为 tuning 推导（`day_time`＝300 秒、`total_day_time`＝480 秒）。
- 结构检查：`validate_page.py` 通过（standard 档内）；`validate_source_links.py --strict` 0 错 0 警。Lua 示例已单独做语法解析。
- 复核：2026-11-08，Steam buildid 26474873。核对本页引用源在 26335394→26474873 间的差异（`tuning.lua`、`plantables.lua` 各插入 1 行，`SGwilson.lua` 仅换声效），锚点按新树重定位、结论未变；上一条 26335394 审阅记录保留作历史。

## 相关链接

- 种植与部署：[[reference/components/deployable]]、[[reference/components/placer]]、[[reference/components/inventoryitem]]
- 可食与鲜度：[[reference/components/edible]]、[[reference/components/perishable]]、[[reference/components/cookable]]
- 冬季树架：[[reference/components/winter_treeseed]]
- 同批的石果与多肉：[[reference/prefabs/rock-avocado-fruit]]

返回 [[reference/prefabs/index|Prefab 索引]]。
