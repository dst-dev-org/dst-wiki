---
title: "可种植的树种（pinecone、twiggy_nut、palmcone_seed、marblebean 与四个放置器）"
created: "2026-11-10"
updated: "2026-11-10"
type: prefab
summary: "prefabs/pinecone.lua 的 addcone 工厂产出松果、多枝树种、棕榈松果树芽三个树种，加上独立文件的 marblebean：四条 DEPLOYMODE.PLANT 种植门禁、四个放置预览、共用 sapling_fn 的树苗计时与长成的常青树／多枝树／棕榈松果树／大理石灌木，以及常青树、棕榈松果树、大理石灌木、浣猫、海盗战利品与猴子岛传送门等来源。"
aliases: ["pinecone", "twiggy_nut", "palmcone_seed", "marblebean", "pinecone_placer", "twiggy_nut_placer", "palmcone_seed_placer", "marblebean_placer", "松果", "多枝树种", "棕榈松果树芽", "大理石豌豆", "种树", "树种子", "种松果"]
tags: [prefab, item]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/pinecone.lua", "prefabs/marblebean.lua", "prefabs/palmcone_scale.lua", "prefabs/planted_tree.lua", "prefabs/evergreens.lua", "prefabs/palmconetree.lua", "prefabs/marbleshrub.lua", "prefabs/catcoon.lua", "prefabs/kitcoonden.lua", "prefabs/monkeyisland_portal.lua", "prefabs/pigking.lua", "components/piratespawner.lua", "prefabs/wormwood.lua", "prefabs/waxed_plants.lua", "prefabs/waxed_plant_common.lua", "components/deployable.lua", "components/inventoryitem_replica.lua", "components/inventory.lua", "components/map.lua", "components/stackable.lua", "components/growable.lua", "components/playercontroller.lua", "components/tradable.lua", "prefabutil.lua", "tilemanager.lua", "tiledefs.lua", "constants.lua", "recipes.lua", "componentutil.lua", "actions.lua", "standardcomponents.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 可种植的树种（pinecone、twiggy_nut、palmcone_seed、marblebean 与四个放置器）

## 快速判断

- 职责与适用：本页覆盖两个工厂的四个可种物品——`addcone` 一次产出的 `pinecone`（松果）、`twiggy_nut`（多枝树种）、`palmcone_seed`（棕榈松果树芽），三件共用同一个 `fn`；`marblebean`（大理石豌豆）在自己的文件里以独立 `fn` 注册。四件都走 `DEPLOYMODE.PLANT`，种下后各自生成 `pinecone_sapling`／`twiggy_nut_sapling`／`palmcone_sapling`／`marblebean_sapling`。[addcone](../../../raw/current/scripts/prefabs/pinecone.lua#L48-L133)、[三注册](../../../raw/current/scripts/prefabs/pinecone.lua#L135-L137)、[marblebean](../../../raw/current/scripts/prefabs/marblebean.lua#L22-L63)
- 优先使用：改物品自身数值看[数值、堆叠与可燃](#数值堆叠与可燃)；改「能不能种、种在哪」看[种植门禁与落点](#种植门禁与落点)；改长成什么、挖回什么看[树苗与成树](#树苗与成树)；补来源看[获得路径](#获得路径)。
- 重要副作用：种下只消耗堆叠里的 1 件；三个 `addcone` 树种在半径 16 内按基础概率 0.33（严格 <5）／0.15 让树精沉睡，大理石豌豆没有这段；`marblebean` 没有 `deployedplant` 标签，连 `itemplanted` 都不推，沃姆伍德的种植回理智因此不触发。[树精安抚](../../../raw/current/scripts/prefabs/pinecone.lua#L18-L37)、[部署事件](../../../raw/current/scripts/components/deployable.lua#L132-L147)、[大理石豆标签](../../../raw/current/scripts/prefabs/marblebean.lua#L36-L38)、[沃姆伍德](../../../raw/current/scripts/prefabs/wormwood.lua#L120-L134)
- 修改边界：成树的砍伐／采掘、再生、烧毁与石化归各自页面（见[相关链接](#相关链接)）；树苗工厂 `sapling_fn` 只列与本批相关的差异；`palmcone_scale`（棕榈松果树鳞片）是**不可种**的燃料与材料，本页只作对照与误区。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `pinecone` | `addcone("pinecone","pinecone_sapling","pinecone","pinecone","idle_planted","winter_tree")`：公共段 → `SetPristine` → 主模拟 | 树苗目标 `pinecone_sapling`、冬季树架目标 `winter_tree`；显示名 松果。[注册行](../../../raw/current/scripts/prefabs/pinecone.lua#L135-L135) |
| `twiggy_nut` | 同 `addcone`：目标 `twiggy_nut_sapling`、冬季目标 `winter_twiggytree` | 同一 `fn`，只换 bank／build／动画，行为与 `pinecone` 逐项相同；显示名 多枝树种。[注册行](../../../raw/current/scripts/prefabs/pinecone.lua#L136-L136) |
| `palmcone_seed` | 同 `addcone`：目标 `palmcone_sapling`、冬季目标 `winter_palmconetree` | 显示名 **棕榈松果树芽**（不是「种子」）；与 `palmcone_scale` 不是同一件物品。[注册行](../../../raw/current/scripts/prefabs/pinecone.lua#L137-L137) |
| `marblebean` | 独立 `fn`：无 `deployedplant`／`fuel`／`burnable`；`ondeploy` 自建树苗 | 显示名 **大理石豌豆**（不是「大理石豆」）；tag `cattoy`／`molebait`／`treeseed`，`SetSinks(true)`，`MakeHauntableLaunch`。[构造](../../../raw/current/scripts/prefabs/marblebean.lua#L30-L59) |
| `pinecone_placer`／`twiggy_nut_placer`／`palmcone_seed_placer` | `addcone` 尾部统一 `MakePlacer(name.."_placer", bank, build, anim)` | 与实物同 bank／build，播 `idle_planted`；三件参数结构相同。[注册](../../../raw/current/scripts/prefabs/pinecone.lua#L131-L132) |
| `marblebean_placer` | `MakePlacer("marblebean_placer","marblebean","marblebean","idle_planted")` | 与 `pinecone` 族同构；四个 placer 都只有 `idle_planted` 一个动画。[注册](../../../raw/current/scripts/prefabs/marblebean.lua#L62-L63) |

### addcone 的两端分界

| 位置 | 内容 | 证据 |
|---|---|---|
| 公共端外形 | `Transform`／`AnimState`／`SoundEmitter`／`Network`，`MakeInventoryPhysics`，bank＝build＝参数，播 `idle` | [公共段](../../../raw/current/scripts/prefabs/pinecone.lua#L65-L77) |
| 公共端 tag 与漂浮 | `deployedplant`＋`cattoy`＋`treeseed`；`MakeInventoryFloatable(inst,"small",0.05,0.9)`；`scrapbook_specialinfo="PLANTABLE"` | [标签](../../../raw/current/scripts/prefabs/pinecone.lua#L79-L85) |
| 端分界 | `SetPristine()` 之后 `if not TheWorld.ismastersim then return inst end`；`_spawn_prefab` 因此只在主模拟赋值 | [分界](../../../raw/current/scripts/prefabs/pinecone.lua#L87-L93) |
| 主模拟组件 | `tradable`（未设 `goldvalue`）、`stackable.maxsize=TUNING.STACK_SIZE_SMALLITEM`、`inspectable`、`fuel`（`SMALL_FUEL`）、`MakeSmallBurnable(TUNING.SMALL_BURNTIME)`＋`MakeSmallPropagator`、`inventoryitem`、`MakeHauntableLaunchAndIgnite`、`deployable`（PLANT）、`forcecompostable.brown` | [主模拟](../../../raw/current/scripts/prefabs/pinecone.lua#L95-L117) |
| 冬季树架 | 只有传入 `winter_tree` 的注册挂 `winter_treeseed` 并 `SetTree`；本例三注册都有 | [接线](../../../raw/current/scripts/prefabs/pinecone.lua#L119-L123) |
| 存档升级遗留 | `OnLoad` 只处理 `data.growtime`：有值就 `plant`；本文件没有 `OnSave` 写入该字段 | [OnLoad](../../../raw/current/scripts/prefabs/pinecone.lua#L40-L44)、[赋值](../../../raw/current/scripts/prefabs/pinecone.lua#L126-L126) |

### 名称核验

四件物品各有唯一 `NAMES` 键：`PINECONE`＝松果、`TWIGGY_NUT`＝多枝树种、`PALMCONE_SEED`＝棕榈松果树芽、`MARBLEBEAN`＝大理石豌豆；四个树苗键分别是 `PINECONE_SAPLING`＝常青树苗、`TWIGGY_NUT_SAPLING`＝多枝树苗、`PALMCONE_SAPLING`＝棕榈松果树苗、`MARBLEBEAN_SAPLING`＝大理石芽。作为对照，`PALMCONE_SCALE`＝棕榈松果树鳞片——那是另一件不可种物品。[PINECONE](../../../raw/current/scripts/languages/chinese_s.po#L346960-L346962)、[TWIGGY_NUT](../../../raw/current/scripts/languages/chinese_s.po#L351115-L351117)、[PALMCONE_SEED](../../../raw/current/scripts/languages/chinese_s.po#L346725-L346727)、[PALMCONE_SAPLING](../../../raw/current/scripts/languages/chinese_s.po#L346715-L346717)、[MARBLEBEAN](../../../raw/current/scripts/languages/chinese_s.po#L345025-L345027)、[MARBLEBEAN_SAPLING](../../../raw/current/scripts/languages/chinese_s.po#L345029-L345032)、[PALMCONE_SCALE](../../../raw/current/scripts/languages/chinese_s.po#L346720-L346722)

## 数值、堆叠与可燃

时间基准：`seg_time`＝30 秒、`day_time`＝300 秒、`total_day_time`＝480 秒；燃料与燃烧时长用 `seg_time`，生长用 `day_time`，不能互换。[时间基准](../../../raw/current/scripts/tuning.lua#L19-L27)

| 注册 | 堆叠上限 | 燃料（`fuel`） | 可燃 | 漂浮 | 其它 |
|---|---|---|---|---|---|
| `pinecone`／`twiggy_nut`／`palmcone_seed` | 40（`STACK_SIZE_SMALLITEM`） | `SMALL_FUEL`＝15 | `SMALL_BURNTIME`＝7.5 秒＋小号蔓延 | `small`，垂直偏移 0.05／0.9 | `tradable` 存在但 `goldvalue` 为 0，猪王不收 |
| `marblebean` | 40 | 无 | **不可燃**（没有 burnable） | **无 floater**（全文没有调用 `MakeInventoryFloatable`） | `molebait`（鼹鼠饵）、`SetSinks(true)` 落水下沉 |
| `palmcone_scale`（对照，不可种） | 20（组件默认 20，未改写） | `MED_FUEL`＝45 | `MED_BURNTIME`＝15 秒 | `med`，0.05／0.68 | 无 `treeseed`、无 `deployable`、无 `forcecompostable` |

- 常量：`STACK_SIZE_SMALLITEM`＝40、`STACK_SIZE_MEDITEM`＝20、`SMALL_FUEL`＝15、`MED_FUEL`＝45、`SMALL_BURNTIME`＝7.5、`MED_BURNTIME`＝15。[堆叠](../../../raw/current/scripts/tuning.lua#L81-L83)、[燃料与燃烧](../../../raw/current/scripts/tuning.lua#L1674-L1683)
- `stackable` 的默认上限是 `TUNING.STACK_SIZE_MEDITEM`＝20，`palmcone_scale` 只 `AddComponent("stackable")` 就落在这个默认值上；其余三件显式写成 40。[默认](../../../raw/current/scripts/components/stackable.lua#L23-L34)、[palmcone_scale](../../../raw/current/scripts/prefabs/palmcone_scale.lua#L27-L35)
- `tradable` 的 `goldvalue` 默认 0，猪王只收 `goldvalue > 0`（或万圣夜糖果）；三个 `addcone` 树种都挂了 `tradable` 且没写值，`marblebean` 则**没有** `tradable`——四件都不能换金块。[默认](../../../raw/current/scripts/components/tradable.lua#L1-L7)、[猪王门禁](../../../raw/current/scripts/prefabs/pigking.lua#L572-L580)

## 种植门禁与落点

部署分三段：`deployable:CanDeploy` → `Deploy` → 物品的 `ondeploy`；`Deploy` 在 `deployedplant` 标签存在时向世界推 `itemplanted`。`CanDeploy` 的第一关是 `IsDeployable`（两类模式共有）：`restrictedtag` 不匹配直接拒绝；骑乘时只有带 `complexprojectile` 的物品能部署；手持浮标时只有 `boatbuilder` 能部署。[CanDeploy／Deploy](../../../raw/current/scripts/components/deployable.lua#L132-L147)、[IsDeployable](../../../raw/current/scripts/components/deployable.lua#L89-L102)

- 四件都是 `DEPLOYMODE.PLANT`（值 4）→ `Map:CanDeployPlantAtPoint`：先 `CanPlantAtPoint`（陆地且不在 `GROUND_HARD`），再 `IsDeployPointClear`——它先拒废铁战车竞技场禁建带（即使周围没有实体也拒），再按间距扫实体，跳过不可见／带 `placer`／有父级的非 `childdeployblocker` 对象。[PLANT 分支](../../../raw/current/scripts/components/deployable.lua#L112-L113)、[地块判定](../../../raw/current/scripts/components/map.lua#L167-L175)、[禁建带](../../../raw/current/scripts/components/map.lua#L263-L266)、[实体扫描](../../../raw/current/scripts/components/map.lua#L268-L282)
- `GROUND_HARD` 由 `tiledefs.lua` 里逐块 `hard = true` 登记，含道路（`cobblestone`）、岩地（`rocky`）、洞穴岩层、喷发口等共 30 余项；因此四件**都不能**种在硬质地面上。[登记](../../../raw/current/scripts/tilemanager.lua#L171-L173)、[道路](../../../raw/current/scripts/tiledefs.lua#L374-L380)、[岩地](../../../raw/current/scripts/tiledefs.lua#L557-L563)
- 间距取部署件的 `replica.inventoryitem:DeploySpacingRadius()`：四件都没调 `SetDeploySpacing`，间距档位是 `DEPLOYSPACING.DEFAULT`（值为 0，半径取 `DEPLOYSPACING_RADIUS[DEPLOYSPACING.DEFAULT]`＝2）。邻居调过 `SetDeploySmartRadius` 时，`IsNearOther` 把要求抬到 `邻居半径 + 1`；树苗智能半径是 1，故两株苗要隔 ≥2。[间距常量](../../../raw/current/scripts/constants.lua#L2236-L2263)、[智能半径](../../../raw/current/scripts/components/map.lua#L244-L261)、[树苗半径](../../../raw/current/scripts/prefabs/planted_tree.lua#L125-L125)
- 本批两条路径的差异：`DEPLOYMODE.DEFAULT`（`CanDeployAtPoint`）用 `mouseover` 筛选鼠标下实体，`PLANT`（本页四件）不读它；`mouseover` 也参与 WATER／CUSTOM 分支，但只在预览阶段拿得到——服务端 `ACTIONS.DEPLOY` 与 `Deploy` 复查都传 `nil`，客户端 placer 的 `testfn` 才把鼠标实体传给副本。
- 放置预览由客户端按 `<物品 prefab>_placer` 取；placer 带 `CLASSIFIED`／`NOCLICK`／`placer` 标签、`persists=false`，每帧用该物品 replica 的 `CanDeploy` 判色，PLANT 分支在客户端同样走 `CanDeployPlantAtPoint`。[服务端传 nil](../../../raw/current/scripts/actions.lua#L1607-L1613)、[Deploy 复查](../../../raw/current/scripts/components/deployable.lua#L132-L139)、[预览 testfn](../../../raw/current/scripts/components/playercontroller.lua#L2906-L2911)、[取名](../../../raw/current/scripts/components/inventoryitem_replica.lua#L360-L369)、[replica 校验](../../../raw/current/scripts/components/inventoryitem_replica.lua#L327-L353)、[MakePlacer](../../../raw/current/scripts/prefabutil.lua#L1-L19)

### 两条 ondeploy（四个物品）

| 注册 | 行为 | 证据 |
|---|---|---|
| `pinecone`／`twiggy_nut`／`palmcone_seed` | `stackable:Get()` → `Physics:Teleport(pt)` → `plant`：`SpawnPrefab(inst._spawn_prefab or "pinecone_sapling")`、`StartGrowing()`、播 `plant_tree`、删原件 → 再遍历半径 16 内的 `leif` 做安抚抽签 | [ondeploy](../../../raw/current/scripts/prefabs/pinecone.lua#L12-L38)、[plant](../../../raw/current/scripts/prefabs/pinecone.lua#L3-L9) |
| `marblebean` | 直接 `SpawnPrefab("marblebean_sapling")`＋`StartGrowing`＋播声＋`inst:Remove()`；不读 `deployer`、无安抚 | [ondeploy](../../../raw/current/scripts/prefabs/marblebean.lua#L13-L20) |

- 消耗：部署动作先在容器里 `RemoveItem` 拆出 **1 件**（`stackable:Get()`）再 `Deploy`，所以整叠只少一件；三个 `addcone` 种子的 `ondeploy` 里又 `stackable:Get()` 一次，对这件单体是空操作，`marblebean` 的 `ondeploy` 没有这一步。[拆件](../../../raw/current/scripts/components/inventory.lua#L1390-L1403)、[动作](../../../raw/current/scripts/actions.lua#L1625-L1638)
- 安抚抽签：近档距离是 `dist² < 5²`（**严格小于 5**），取基础概率 0.33；其余落在半径 16 内的取基础概率 0.15。两者都只是 `TryLuckRoll(deployer, chance, LuckFormulas.LeifChill)` 的**基础**输入，实际概率还会被操作者的运气修正；抽中后还要目标带 `sleeper` 组件才会 `GoToSleep(1000)`（1000 是睡眠接口的时长参数，内部再乘睡眠倍率）并给 `pacify_forest` 成就；失败分支只播一次 `taunt_VO`。[抽签](../../../raw/current/scripts/prefabs/pinecone.lua#L19-L37)、[幸运式](../../../raw/current/scripts/componentutil.lua#L2237-L2251)、[常量](../../../raw/current/scripts/tuning.lua#L1060-L1064)

## 树苗与成树

四个树苗同由 `planted_tree.lua` 的 `sapling_fn` 产出：`timer` 的 `"grow"` 计时到点后 `SpawnPrefab(growprefab)`、调 `tree:growfromseed()`、删自己。[工厂](../../../raw/current/scripts/prefabs/planted_tree.lua#L72-L99)、[四条注册](../../../raw/current/scripts/prefabs/planted_tree.lua#L183-L189)

| 树种 | 生成的树苗 | 生长计时 | 长成 | 挖起掉落 |
|---|---|---|---|---|
| `pinecone` | `pinecone_sapling`（tag `evergreen`） | `growtimes` 为 `nil` → `PINECONE_GROWTIME.base`＝0.75`day_time`＝225 秒，±75 秒 | `evergreen_short`，`growfromseed` 压到阶段 1 | `twigs`（默认表，注册未传 `overrideloot`）[注册](../../../raw/current/scripts/prefabs/planted_tree.lua#L183-L183)、[growfromseed](../../../raw/current/scripts/prefabs/evergreens.lua#L571-L576) |
| `twiggy_nut` | `twiggy_nut_sapling`（tag `twiggytree`） | 同上（225±75 秒） | `twiggy_short` | `twigs`[注册](../../../raw/current/scripts/prefabs/planted_tree.lua#L186-L186) |
| `palmcone_seed` | `palmcone_sapling`（tag `palmconetree`） | 同上 | `palmconetree_short` | `twigs`[注册](../../../raw/current/scripts/prefabs/planted_tree.lua#L189-L189)、[growfromseed](../../../raw/current/scripts/prefabs/palmconetree.lua#L426-L431) |
| `marblebean` | `marblebean_sapling`（tag `marbleshrub`，`fireproof=true`） | 同上 | `marbleshrub_short`（随机外形与颜色） | `marblebean`（`{"marblebean"}`）[注册](../../../raw/current/scripts/prefabs/planted_tree.lua#L187-L187)、[growfromseed](../../../raw/current/scripts/prefabs/marbleshrub.lua#L99-L109) |

- 计时口径：`startgrowing` 只在没有 `"grow"` 计时器时起表；点火时 `stopgrowing` 只调 `StopTimer("grow")`，**不保存剩余时长**，熄灭后重新 `startgrowing` 会再抽一次 `GetRandomWithVariance`，等于整段重来（防火的大理石芽没有这条监听）。[起表](../../../raw/current/scripts/prefabs/planted_tree.lua#L86-L93)、[燃烧监听](../../../raw/current/scripts/prefabs/planted_tree.lua#L164-L173)、[常量](../../../raw/current/scripts/tuning.lua#L802-L802)
- `growfromseed` 的门：`evergreen` 侧压阶段 1 并 `SetStage`；`palmconetree` 侧 `SetStage(1)`＋播 `grow_seed_to_short`；`marbleshrub` 侧随机 1–3 号外形、随机灰度、`SetStage(1)`＋`grow_seed_to_short`。[常青树](../../../raw/current/scripts/prefabs/evergreens.lua#L571-L576)、[棕榈松果树](../../../raw/current/scripts/prefabs/palmconetree.lua#L426-L431)、[大理石灌木](../../../raw/current/scripts/prefabs/marbleshrub.lua#L99-L109)、[SetStage 触发阶段 fn](../../../raw/current/scripts/components/growable.lua#L207-L219)
- 长成后各自的行为不在本页：常青树见 [[reference/prefabs/evergreen-trees]]；棕榈松果树与大理石灌木的砍伐／采掘、再生与烧毁只有源码入口（Wiki 待建）。

## 获得路径

| 注册 | 已核验来源 | 产出与证据 |
|---|---|---|
| `pinecone` | ①砍倒普通常青树：`normal_loot` 1 个、`tall_loot` 2 个，老树阶段整表换成 1 个；②烧毁株的自然掉落与「先砍」各再掷一次 10%（仅 `drop_pinecones` 为真的 build）；③浣猫礼物池（tier 7／8 与 friendGift tier 2）；④小浣猫托儿所（kitcoonden）捉迷藏的「非 YOT」奖励池（权重 3） | ①[loot 表](../../../raw/current/scripts/prefabs/evergreens.lua#L66-L70)、[老树](../../../raw/current/scripts/prefabs/evergreens.lua#L315-L329)、②[烧毁 10%](../../../raw/current/scripts/prefabs/evergreens.lua#L184-L191)、③[猫礼物](../../../raw/current/scripts/prefabs/catcoon.lua#L106-L117)、[friendGift](../../../raw/current/scripts/prefabs/catcoon.lua#L130-L137)、④[池](../../../raw/current/scripts/tuning.lua#L5300-L5311)、[发放](../../../raw/current/scripts/prefabs/kitcoonden.lua#L287-L293) |
| `twiggy_nut` | ①多枝树：`normal_loot` 1 个、`tall_loot` 2 个（多枝树 `drop_pinecones=false`，烧毁不补）；②小浣猫托儿所（kitcoonden）捉迷藏「非 YOT」奖励池（权重 3，注释说明这是不靠世界生成替换也能拿到多枝树的渠道） | ①[loot 表](../../../raw/current/scripts/prefabs/evergreens.lua#L94-L97)、②[池](../../../raw/current/scripts/tuning.lua#L5300-L5311)、[发放](../../../raw/current/scripts/prefabs/kitcoonden.lua#L287-L293) |
| `palmcone_seed` | ①仅 `tall` 棕榈松果树掉落（矮树／中树都不掉，共享表里 1 个，权重 1.0）；②猴子岛传送门的周期吐物池：权重 5（总 43）；15 秒是**每次吐物后重排**的计时（睡眠 `PauseTimer`／醒来 `ResumeTimer`），掷普通池前先清理失效／超 80 距离旧记录并要求 `#_loot < 10`，超限只走 FX 备用；开关 `MONKEYISLAND_PORTAL_ENABLED`／世界设置；③事件清单另有 2 个 | ①[表](../../../raw/current/scripts/prefabs/palmconetree.lua#L63-L71)、②[权重表](../../../raw/current/scripts/prefabs/monkeyisland_portal.lua#L35-L46)、[清理与抽池](../../../raw/current/scripts/prefabs/monkeyisland_portal.lua#L105-L115)、[上限与 FX 备用](../../../raw/current/scripts/prefabs/monkeyisland_portal.lua#L184-L194)、[重排计时](../../../raw/current/scripts/prefabs/monkeyisland_portal.lua#L344-L351)、[睡眠暂停](../../../raw/current/scripts/prefabs/monkeyisland_portal.lua#L330-L339)、[常量](../../../raw/current/scripts/tuning.lua#L6687-L6695)、③[事件清单](../../../raw/current/scripts/prefabs/monkeyisland_portal.lua#L220-L233) |
| `marblebean` | ①采掘大理石灌木：`normal` 的抽签基础概率 0.75（经 `TryLuckRoll` 带操作者运气修正），成功只给 1 个 `marble`、失败才多给 1 个 `marblebean`；`tall` 基础概率 0.5，成功给 2 个 `marble`、失败给 `marble`＋`marblebean`；`short` 只给 `marble`；②配方 `marblebean`：1 块 `marble`，`TECH.SCIENCE_TWO` | ①[三档表](../../../raw/current/scripts/prefabs/marbleshrub.lua#L20-L56)、②[配方](../../../raw/current/scripts/recipes.lua#L465-L465) |
| `palmcone_scale`（对照） | ①`tall` 棕榈松果树 2 个；②海盗宝箱生成时先塞 2–4 个 | ①[表](../../../raw/current/scripts/prefabs/palmconetree.lua#L63-L71)、②[宝箱战利品](../../../raw/current/scripts/components/piratespawner.lua#L553-L566) |

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`deployable` | 决定种植模式与 `ondeploy`；`onmode`／`onspacing` 同步 replica | 四件都是 PLANT；仅主模拟持有组件 | [[reference/components/deployable#关键接口]]；[构造](../../../raw/current/scripts/components/deployable.lua#L31-L53) |
| 组件：`stackable` | 堆叠上限 40（`palmcone_scale` 20）；部署时被拆出 1 件 | 主模拟与 replica 各一份计数 | [[reference/components/stackable#关键接口]]；[默认](../../../raw/current/scripts/components/stackable.lua#L23-L34) |
| 组件：`fuel`／`burnable`／`propagator` | 当燃料、可燃、蔓延；烧尽走 `DefaultBurntFn` | `marblebean` 三者都没有；其余三件小号燃烧 7.5 秒 | [[reference/components/burnable#关键接口]]、[[reference/components/fuel]]；[helper](../../../raw/current/scripts/standardcomponents.lua#L164-L174)、[烧尽](../../../raw/current/scripts/standardcomponents.lua#L18-L44) |
| 组件：`tradable` | 只挂组件、`goldvalue` 为 0 | 只有三个 `addcone` 树种有；`marblebean` 没有——四件都不能换金块 | [[reference/components/tradable]]；[默认](../../../raw/current/scripts/components/tradable.lua#L1-L7) |
| 组件：`winter_treeseed` | 三个 `addcone` 注册各写一个冬季树 prefab，供冬季树架种 | `marblebean` 没有 | [[reference/components/winter_treeseed#关键接口]]；[接线](../../../raw/current/scripts/prefabs/pinecone.lua#L119-L123) |
| 组件：`forcecompostable` | `brown=true`，作堆肥棕色料 | 仅三个 `addcone` 注册 | [[reference/components/forcecompostable]]；[接线](../../../raw/current/scripts/prefabs/pinecone.lua#L116-L117) |
| 组件：`hauntable` | 三个 `addcone` 树种用 `MakeHauntableLaunchAndIgnite`；`marblebean` 用 `MakeHauntableLaunch` | 前者：弹飞成立、点燃分支被 `#HAUNTFIX` 注释掉，命中弹飞时仍**返回 false**；后者：弹飞成立并**返回 true**，本来就没有点燃分支 | [[reference/components/hauntable#关键接口]]；[LaunchAndIgnite](../../../raw/current/scripts/standardcomponents.lua#L1191-L1215)、[Launch](../../../raw/current/scripts/standardcomponents.lua#L1053-L1069) |
| 关联 Prefab：四个树苗 | 种下后生成；长成后删自己 | 主模拟生成；共用 `sapling_fn` | [注册](../../../raw/current/scripts/prefabs/planted_tree.lua#L183-L189)（Wiki 待建） |
| 关联 Prefab：`evergreen_short` | 松果长成的第一形态 | 主模拟；`growfromseed` 后进入四阶段 | [[reference/prefabs/evergreen-trees]] |
| 关联 Prefab：`palmconetree_short`／`marbleshrub_short` | 棕榈松果树／大理石灌木的第一形态 | 主模拟 | [palmconetree](../../../raw/current/scripts/prefabs/palmconetree.lua#L417-L430)、[marbleshrub](../../../raw/current/scripts/prefabs/marbleshrub.lua#L155-L232)（Wiki 待建） |
| 关联 Prefab：`palmcone_scale` | 同树的另一件掉落物；材料，不可种 | 只有 `tall` 树与海盗战利品 | [构造](../../../raw/current/scripts/prefabs/palmcone_scale.lua#L1-L45)（Wiki 待建） |
| 关联 Prefab：四个蜡封树苗（`pinecone_sapling_waxed` 等） | 蜡封喷雾把已种下的树苗换成 `<_sapling>_waxed`，蜡封件自身行为不在本页 | 需蜡喷雾；`MakeWaxablePlant` 在主模拟挂载 | [注册表](../../../raw/current/scripts/prefabs/waxed_plants.lua#L693-L699)、[换件](../../../raw/current/scripts/prefabs/waxed_plant_common.lua#L543-L556)、[挂载](../../../raw/current/scripts/prefabs/planted_tree.lua#L175-L175) |
| 资源：`anim/pinecone.zip`／`twiggy_nut.zip`／`palmcone_seed.zip`／`marblebean.zip` | 物品、树苗与 placer 共用动画包 | 原版资源，不随页分发 | [pinecone assets](../../../raw/current/scripts/prefabs/pinecone.lua#L49-L55)、[marblebean assets](../../../raw/current/scripts/prefabs/marblebean.lua#L3-L6) |

## 典型接入

### 让松果种下后生成自定义树苗

- 前提与选择：服务端 `modmain.lua` 的 prefab postinit。`plant` 只读 `_spawn_prefab`，改它不碰模式、堆叠或掉落；改 `PINECONE_GROWTIME` 只改计时、不换产物。自定义苗要实现 `StartGrowing`（起 `"grow"` 计时）、`Transform:SetPosition` 与 `SoundEmitter`——`plant` 会无条件播 `dontstarve/wilson/plant_tree`，缺组件直接报错。
- 生效与撤销：对**通常存档**只影响之后新部署的松果；字段在主模拟构造时赋 `"pinecone_sapling"`。但 `pinecone` 还有一条只为旧存档升级保留的 `OnLoad` 分支：若旧存档的该物品带着 `data.growtime`，读档时同样走 `plant`，于是也会用**改后的** `_spawn_prefab` 立即长出新苗（原物品随即删除）。
- 原版依据：`plant` 的顺序是 `SpawnPrefab(inst._spawn_prefab or "pinecone_sapling")` → `StartGrowing()` → `SetPosition(原物品位置)` → 播 `plant_tree` → `inst:Remove()`；形参 `growtime` 在函数体里**没有被使用**。[plant](../../../raw/current/scripts/prefabs/pinecone.lua#L3-L9)、[赋值](../../../raw/current/scripts/prefabs/pinecone.lua#L93-L93)

```lua
-- modmain.lua：松果改种自定义树苗（该 prefab 需自带 StartGrowing）
AddPrefabPostInit("pinecone", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    inst._spawn_prefab = "mytree_sapling"
end)
```

- 调整与验证：`mytree_sapling` 要按 `planted_tree.lua` 的树苗实现 `StartGrowing`（内部给 `timer` 起 `"grow"`）并在到点时换树，同时保留 `Transform` 与 `SoundEmitter`；验证分两路——新种下的松果是否生成该 prefab 并可被点燃／熄灭（注意熄灭会重抽整段），以及带 `growtime` 的旧存档物品读档后是否也换成新 prefab。

## 常见陷阱与限制

- **误区：`palmcone_scale`（棕榈松果树鳞片）能种。** 它只有 `inventoryitem`／`stackable`／`tradable`／`fuel`／小号可燃，没有 `deployable`、没有 `treeseed` 标签，也没有 placer；可种的棕榈种子是同树的 `palmcone_seed`（棕榈松果树芽）。[对照](../../../raw/current/scripts/prefabs/palmcone_scale.lua#L27-L42)、[种子](../../../raw/current/scripts/prefabs/pinecone.lua#L137-L137)
- **误区：`marblebean` 叫「大理石豆」。** 同版简中实显是 **大理石豌豆**，树苗是 **大理石芽**。[po](../../../raw/current/scripts/languages/chinese_s.po#L345024-L345032)
- **边界：硬地上种不了。** 四件同走 PLANT，`GROUND_HARD`（道路、岩地、洞穴岩层、喷发口等）一律拒绝；要种在硬地得换 CUSTOM 或 ANYWHERE 模式，本页未提供该改法。[PLANT](../../../raw/current/scripts/components/map.lua#L167-L175)
- **边界：种一次只少一件。** 拆分发生在 `Inventory:RemoveItem`；三个 `addcone` 种子在 `ondeploy` 里再 `stackable:Get()` 一次（对单体是空操作），`marblebean` 直接 `inst:Remove()`。[拆件](../../../raw/current/scripts/components/inventory.lua#L1397-L1403)
- **边界：烧尽只出灰，不退种子。** 三件可燃种子都走 `DefaultBurntFn`：非海上生成 `ash`（堆叠取 `min(ash 上限, 原堆叠)`），海上什么都不产。[烧尽](../../../raw/current/scripts/standardcomponents.lua#L31-L44)
- **边界：大理石豌豆种下不推 `itemplanted`。** 它没有 `deployedplant` 标签，`Deployable:Deploy` 的 `isplant` 判假，事件不推。[事件](../../../raw/current/scripts/components/deployable.lua#L137-L145)、[标签](../../../raw/current/scripts/prefabs/marblebean.lua#L36-L38)
- **边界：作祟不会点燃这些种子，但两种作祟的返回值不同。** 三个 `addcone` 种子用 `MakeHauntableLaunchAndIgnite`：弹飞会执行，点燃分支被 `#HAUNTFIX` 注释掉，弹飞后仍 `return false`；`marblebean` 用 `MakeHauntableLaunch`：弹飞并 `return true`，没有点燃分支可注释。[LaunchAndIgnite](../../../raw/current/scripts/standardcomponents.lua#L1204-L1215)、[Launch](../../../raw/current/scripts/standardcomponents.lua#L1053-L1069)
- **未核验**：`pinecone` 的 `OnLoad` 分支在正常存档中不可达（本文件无 `OnSave` 写 `growtime`）；四个 `sapling_fn` 树苗的蜡封件（`<_sapling>_waxed`）行为不在本页；`marbleshrub` 战利品抽签的实际掉率未实测（源码按 1−0.75／1−0.5 推）。

## 验证记录

- 源码审阅：2026-11-10，Steam buildid 26474873。通读 `pinecone.lua`、`palmcone_scale.lua`、`marblebean.lua` 全文；核对 `planted_tree.lua` 的 `sapling_fn`／`growtree`／`digup` 与四条注册，`evergreens.lua`／`palmconetree.lua`／`marbleshrub.lua` 的 `growfromseed` 与掉落表，`catcoon.lua`／`kitcoonden.lua`／`monkeyisland_portal.lua`／`piratespawner.lua`／`pigking.lua` 的礼物池、奖励池、周期吐物、宝箱战利品与交易门禁，以及 `deployable`／`map`／`tilemanager`／`tiledefs`／`inventoryitem_replica`／`inventory`／`stackable`／`tradable`／`growable`／`standardcomponents` 的相关分支和 `tuning`／`constants` 常量。
  名称核验：`NAMES.PINECONE`＝松果、`TWIGGY_NUT`＝多枝树种、`PALMCONE_SEED`＝棕榈松果树芽、`MARBLEBEAN`＝大理石豌豆；树苗 `PINECONE_SAPLING`＝常青树苗、`TWIGGY_NUT_SAPLING`＝多枝树苗、`PALMCONE_SAPLING`＝棕榈松果树苗、`MARBLEBEAN_SAPLING`＝大理石芽；对照 `PALMCONE_SCALE`＝棕榈松果树鳞片。四个 placer 无 `NAMES` 键。
  未覆盖：成树的砍伐／采掘与再生、蜡封树苗交互、活动皮肤、实际游戏运行。
- 游戏运行：未运行。秒数按 `seg_time`＝30、`day_time`＝300、`total_day_time`＝480 推算；掉率为源码常量推导。
- 结构检查：`validate_page.py` 通过（standard 档内）；`validate_source_links.py --strict` 0 错 0 警（计数见校验输出与交付回报）。Lua 示例用 `luaparser` 4.2.0 解析通过（非游戏运行时）。

## 相关链接

- 同族树：[[reference/prefabs/evergreen-trees]]、[[reference/prefabs/deciduous-trees]]、[[reference/prefabs/tree-rocks]]
- 另一种可种种子：[[reference/prefabs/acorn]]
- 树苗与蜡封：[[reference/components/winter_treeseed]]、[[reference/components/deployable]]、[[reference/components/growable]]
- 材料去向：[[reference/prefabs/crafting-materials-index]]

返回 [[reference/prefabs/index|Prefab 索引]]。
