---
title: "地皮物品（turf_*，本版 40 个注册）"
created: "2026-11-10"
updated: "2026-11-10"
type: prefab
summary: "turfs.lua 从 GroundTiles.turf 逐个生成的 40 个地皮物品：注册名与地块键的对应、TURF 部署模式与 grid 预览、只能铺在 DIRT 的落点门禁，以及配方／挖起两条来源。"
aliases: ["turf_road","turf_cotl_brick","turf_pebblebeach","turf_monkey_ground","turf_shellbeach","turf_marsh","turf_rocky","turf_savanna","turf_forest","turf_grass","turf_deciduous","turf_desertdirt","turf_cave","turf_fungus","turf_fungus_red","turf_fungus_green","turf_fungus_moon","turf_sinkhole","turf_underrock","turf_vent","turf_mud","turf_archive","turf_vault","turf_ruinsbrick_glow","turf_ruinsbrick","turf_ruinstiles_glow","turf_ruinstiles","turf_ruinstrim_glow","turf_ruinstrim","turf_meteor","turf_dragonfly","turf_woodfloor","turf_cotl_gold","turf_checkerfloor","turf_mosaic_grey","turf_mosaic_red","turf_mosaic_blue","turf_carpetfloor2","turf_carpetfloor","turf_beard_rug", "地皮", "草皮", "铺地皮", "turf", "地皮物品", "森林地皮", "卵石路", "棋盘地板"]
tags: [prefab, item]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/turfs.lua", "prefabs/gridplacer.lua", "prefabs/pitchfork.lua", "worldtiledefs.lua", "tilemanager.lua", "tiledefs.lua", "componentutil.lua", "components/terraformer.lua", "components/deployable.lua", "components/inventoryitem_replica.lua", "components/placer.lua", "components/map.lua", "components/stackable.lua", "components/trap.lua", "standardcomponents.lua", "constants.lua", "tuning.lua", "strings.lua", "recipes.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 地皮物品（turf_*，本版 40 个注册）

## 快速判断

- 职责与适用：`prefabs/turfs.lua` 遍历 `GroundTiles.turf` 的每个键，用 `make_turf(tile, data)` 生成 1 个 `turf_<data.name>` 物品；本版共 40 个注册。注册名取自数据表的 `name` 而不是地块键，两者不一定相同（如 `SCALE` → `turf_dragonfly`、`CHECKER` → `turf_checkerfloor`）。[工厂](../../../raw/current/scripts/prefabs/turfs.lua#L8-L17)、[注册循环](../../../raw/current/scripts/prefabs/turfs.lua#L83-L87)、[名称拼接](../../../raw/current/scripts/prefabs/turfs.lua#L80)
- 优先使用：查某块地皮对应哪个物品、哪个科技看[注册与构造](#注册与构造)；改铺设条件与预览看[部署：TURF 模式与 grid 预览](#部署turf-模式与-grid-预览)；改物品侧堆叠、燃料与诱饵看[物品侧接线](#物品侧接线)；改获得方式看[来源](#来源)。
- 重要副作用：落点不合格时动作在 `CanDeploy` 就返回失败、不进回调也不扣件；落点合格后 `ondeploy` 才 `SetTile` 改地块、再 `stackable:Get():Remove()` 扣 1 张，扣件前不读任何结果。两类要分开：前者是“放置失败”，后者是“动作成功但最终地块未被校验”。`Map:SetTile` 的包装只转调引擎并推 `onterraform` 事件，不返回成功值。[部署回调](../../../raw/current/scripts/prefabs/turfs.lua#L9-L17)、[地块判定](../../../raw/current/scripts/components/map.lua#L162-L164)、[事件](../../../raw/current/scripts/components/map.lua#L3-L8)
- 修改边界：本页只讲 `turfs.lua` 生成的物品。`dock_kit`、`wagpunk_floor_kit` 同样带 `groundtile` 标签，但由各自工厂提供，不在这 40 个注册内。

## 注册与构造

`ValidateTurfDef` 给两项缺省值：`anim` 缺省取 `name`，`bank_build` 缺省取字符串 `"turf"`；`bank_override`／`build_override`／`animzip_override`／`invicon_override` 四个字段本版 40 条都没有使用，源码注释说明它们是给模组用的细调项。[校验](../../../raw/current/scripts/tilemanager.lua#L117-L124)、[登记](../../../raw/current/scripts/tilemanager.lua#L205-L209)

- 资产是 `anim/<animzip_override or bank_build>.zip` 与 `INV_IMAGE turf_<invicon_override or name>`；物品图像按注册名取，所以 `turf_dragonfly` 的图与地面贴图不同名。[资产](../../../raw/current/scripts/prefabs/turfs.lua#L75-L80)
- `inst.tile = tile` 存的是地块数字 id（`WORLD_TILES.*`），不是字符串；`groundtile` 与 `molebait` 两个标签都在 `SetPristine` 之前加。[字段与标签](../../../raw/current/scripts/prefabs/turfs.lua#L37-L44)
- `scrapbook_deps = {}` 表示图鉴不列依赖；`scrapbook_specialinfo = "TURF"` 对应文案 "Can be placed on dirt ground to change the type of ground."。[图鉴](../../../raw/current/scripts/prefabs/turfs.lua#L26-L44)、[文案](../../../raw/current/scripts/strings.lua#L19003)

下表 40 行是本版 `GroundTiles.turf` 的全部键。`配方 L` 列为 `recipes.lua` 中该条起始行，`tile L` 列为 `tiledefs.lua` 中该地块 `AddTile` 的起始行。

| 地块键 | 注册名 | 简中名 | 科技 | 配方 L | tile L |
|---|---|---|---|---|---|
| `ROAD` | `turf_road` | 卵石路 | SCIENCE_TWO | 692 | 369 |
| `COTL_BRICK` | `turf_cotl_brick` | 砖地板 | LOST | 1025 | 393 |
| `PEBBLEBEACH` | `turf_pebblebeach` | 岩石海滩地皮 | TURFCRAFTING_TWO | 712 | 418 |
| `MONKEY_GROUND` | `turf_monkey_ground` | 月亮码头海滩地皮 | LOST | 705 | 441 |
| `SHELLBEACH` | `turf_shellbeach` | 贝壳海滩地皮 | LOST | 704 | 465 |
| `MARSH` | `turf_marsh` | 沼泽地皮 | MASHTURFCRAFTING_TWO | 721 | 488 |
| `ROCKY` | `turf_rocky` | 岩石地皮 | TURFCRAFTING_TWO | 713 | 552 |
| `SAVANNA` | `turf_savanna` | 热带草原地皮 | TURFCRAFTING_TWO | 709 | 575 |
| `FOREST` | `turf_forest` | 森林地皮 | TURFCRAFTING_TWO | 707 | 597 |
| `GRASS` | `turf_grass` | 长草地皮 | TURFCRAFTING_TWO | 708 | 619 |
| `DECIDUOUS` | `turf_deciduous` | 落叶林地皮 | TURFCRAFTING_TWO | 710 | 661 |
| `DESERT_DIRT` | `turf_desertdirt` | 沙漠地皮 | TURFCRAFTING_TWO | 711 | 683 |
| `CAVE` | `turf_cave` | 鸟粪地皮 | TURFCRAFTING_TWO | 714 | 706 |
| `FUNGUS` | `turf_fungus` | 真菌地皮 | TURFCRAFTING_TWO | 718 | 728 |
| `FUNGUSRED` | `turf_fungus_red` | 真菌地皮 | TURFCRAFTING_TWO | 719 | 750 |
| `FUNGUSGREEN` | `turf_fungus_green` | 真菌地皮 | TURFCRAFTING_TWO | 720 | 772 |
| `FUNGUSMOON` | `turf_fungus_moon` | 变异真菌地皮 | CELESTIAL_THREE | 843 | 794 |
| `SINKHOLE` | `turf_sinkhole` | 黏滑地皮 | TURFCRAFTING_TWO | 716 | 817 |
| `UNDERROCK` | `turf_underrock` | 洞穴岩石地皮 | TURFCRAFTING_TWO | 715 | 839 |
| `VENT` | `turf_vent` | 洞穴喷气孔地皮 | TURFCRAFTING_TWO | 733 | 863 |
| `MUD` | `turf_mud` | 泥泞地皮 | TURFCRAFTING_TWO | 717 | 887 |
| `ARCHIVE` | `turf_archive` | 远古石刻 | LOST | 723 | 909 |
| `VAULT_CLEAN` | `turf_vault` | 远古圣殿石刻 | LOST | 724 | 953 |
| `BRICK_GLOW` | `turf_ruinsbrick_glow` | 仿远古地面 | LOST | 727 | 976 |
| `BRICK` | `turf_ruinsbrick` | 远古地面 | LOST | 726 | 1000 |
| `TILES_GLOW` | `turf_ruinstiles_glow` | 仿远古瓷砖 | LOST | 729 | 1024 |
| `TILES` | `turf_ruinstiles` | 远古瓷砖 | LOST | 728 | 1048 |
| `TRIM_GLOW` | `turf_ruinstrim_glow` | 仿远古砖雕 | LOST | 731 | 1072 |
| `TRIM` | `turf_ruinstrim` | 远古砖雕 | LOST | 730 | 1096 |
| `METEOR` | `turf_meteor` | 月球环形山地皮 | CELESTIAL_THREE | 842 | 1120 |
| `SCALE` | `turf_dragonfly` | 龙鳞地板 | SCIENCE_TWO | 696 | 1253 |
| `WOODFLOOR` | `turf_woodfloor` | 木地板 | SCIENCE_TWO | 693 | 1279 |
| `COTL_GOLD` | `turf_cotl_gold` | 黄金地板 | LOST | 1024 | 1305 |
| `CHECKER` | `turf_checkerfloor` | 棋盘地板 | SCIENCE_TWO | 694 | 1330 |
| `MOSAIC_GREY` | `turf_mosaic_grey` | 灰色马赛克地板 | SCIENCE_TWO | 700 | 1356 |
| `MOSAIC_RED` | `turf_mosaic_red` | 红色马赛克地板 | SCIENCE_TWO | 701 | 1381 |
| `MOSAIC_BLUE` | `turf_mosaic_blue` | 蓝色马赛克地板 | SCIENCE_TWO | 702 | 1406 |
| `CARPET2` | `turf_carpetfloor2` | 茂盛地毯 | SCIENCE_TWO | 699 | 1432 |
| `CARPET` | `turf_carpetfloor` | 地毯地板 | SCIENCE_TWO | 695 | 1458 |
| `BEARD_RUG` | `turf_beard_rug` | 胡须地毯 | SCIENCE_TWO | 697 | 1501 |

- 资产包只有 9 种：缺省 `turf`，以及 `turf_cotl`（两块 COTL 地皮）、`turf_moon`（岩石海滩、月球环形山）、`turf_monkey_ground`、`turf_shellbeach`、`turf_fungus_moon`、`turf_archives`（远古石刻）、`turf_drama`（六块远古砖瓦边、三块马赛克与茂盛地毯）、`turf_beard`。[数据项](../../../raw/current/scripts/tiledefs.lua#L369-L1120)、[数据项](../../../raw/current/scripts/tiledefs.lua#L1375-L1525)
- 13 条显式写了 `anim`（如 `DESERT_DIRT` 用 `dirt`、`UNDERROCK` 用 `rock`、`VENT` 用 `fumarole`、`CHECKER` 用 `checker`、`CARPET` 用 `carpet`、`BEARD_RUG` 用 `turf_beard`），以覆写值为准；其余 27 条由 `ValidateTurfDef` 把 `anim` 缺省设为数据项 `name`，其中 **11 条与地块键不同名**：`SCALE`→`dragonfly`、`VAULT_CLEAN`→`vault`、`BRICK`／`BRICK_GLOW`／`TILES`／`TILES_GLOW`／`TRIM`／`TRIM_GLOW`→`ruins*` 系列，`FUNGUSRED`／`FUNGUSGREEN`／`FUNGUSMOON`→`fungus_*`。[缺省](../../../raw/current/scripts/tilemanager.lua#L117-L121)

## 部署：TURF 模式与 grid 预览

- **模式**：`SetDeployMode(DEPLOYMODE.TURF)`（枚举值 3），并 `SetUseGridPlacer(true)`；`turfs.lua` 的依赖表因此只写 `"gridplacer"`，没有各自的 `turf_<name>_placer`。[模式](../../../raw/current/scripts/prefabs/turfs.lua#L66-L69)、[依赖](../../../raw/current/scripts/prefabs/turfs.lua#L3-L6)、[枚举](../../../raw/current/scripts/constants.lua#L2214-L2226)
- **预览实体**：客户端取 placer 名时先看 `usegridplacer`，为真直接返回 `"gridplacer"`；该实体是 `CLASSIFIED`／`NOCLICK`／`placer` 的非联网预览，`persists = false`、`snap_to_tile = true`，并通过 `oncanbuild`／`oncannotbuild` 切换可建着色（`SetMultColour(1, 1, 1, 1)`，原色）与不可建着色（`SetMultColour(.75, .25, .25, 1)`，染红）。[取名](../../../raw/current/scripts/components/inventoryitem_replica.lua#L360-L368)、[gridplacer](../../../raw/current/scripts/prefabs/gridplacer.lua#L16-L43)
- **落点门禁**：TURF 分支只调 `Map:CanPlaceTurfAtPoint(pt:Get())`，其实现是 `GetTileAtPoint(...) == WORLD_TILES.DIRT`——只能铺在“被挖过的泥土”地块上，既不做 `IsDeployPointClear` 间距检查，也不看 `DEPLOYSPACING`。[门禁](../../../raw/current/scripts/components/deployable.lua#L110-L111)、[实现](../../../raw/current/scripts/components/map.lua#L162-L164)
- **共用门禁**：`IsDeployable` 仍先行，骑乘时只允许 `complexprojectile`，手持浮标时只允许 `boatbuilder` 标签，地皮两者都不满足，所以骑乘或手持浮标时不能铺。[执行](../../../raw/current/scripts/components/deployable.lua#L89-L102)
- **执行**：`ondeploy` 播 `dontstarve/wilson/dig`、把落点换成地块坐标、`SetTile(x, y, tile)`，随后**无条件**取 1 张移除；它没有返回值，Lua 也不读 `SetTile` 的结果（引擎改了没有、改成了什么，脚本侧不校验）。`DEPLOY` 动作先把物品从栏里取出再调 `Deploy`，只有 `Deploy` 返回失败才 `GiveItem` 放回；`CanDeploy` 不通过时根本进不了 `ondeploy`。[部署回调](../../../raw/current/scripts/prefabs/turfs.lua#L9-L17)、[动作](../../../raw/current/scripts/actions.lua#L1608-L1638)、[包装](../../../raw/current/scripts/components/map.lua#L3-L8)
- **客户端一致**：客户端在没有 `deployable` 组件时走 `inventoryitem_replica` 的同类分支，同样调用 `Map:CanPlaceTurfAtPoint`，所以预览的红绿与服务器判定用同一规则。[副本分支](../../../raw/current/scripts/components/inventoryitem_replica.lua#L327-L340)

## 物品侧接线

| 项目 | 取值与行为 | 证据 |
|---|---|---|
| 堆叠 | `TUNING.STACK_SIZE_MEDITEM`＝20 | [堆叠](../../../raw/current/scripts/prefabs/turfs.lua#L52-L53)、[常量](../../../raw/current/scripts/tuning.lua#L81-L82) |
| 燃料 | `fuelvalue = TUNING.TINY_FUEL`＝7.5 秒 | [燃料](../../../raw/current/scripts/prefabs/turfs.lua#L60-L61)、[常量](../../../raw/current/scripts/tuning.lua#L1673-L1674) |
| 可燃 | `MakeMediumBurnable(inst, TUNING.MED_BURNTIME)`，燃尽 15 秒 | [可燃](../../../raw/current/scripts/prefabs/turfs.lua#L62)、[常量](../../../raw/current/scripts/tuning.lua#L1682-L1683) |
| 蔓延 | `MakeSmallPropagator` | [蔓延](../../../raw/current/scripts/prefabs/turfs.lua#L63) |
| 作祟 | `MakeHauntableLaunchAndIgnite`：点火分支在 `#HAUNTFIX` 下停用，实际只 `Launch` 并返回 `false` | [调用](../../../raw/current/scripts/prefabs/turfs.lua#L64)、[实现](../../../raw/current/scripts/standardcomponents.lua#L1191-L1217) |
| 诱饵 | `bait` 组件 + `molebait` 标签：鼹鼠会把它当诱饵叼走，陷阱弹起时也会吃掉附近带 `bait` 的东西 | [诱饵](../../../raw/current/scripts/prefabs/turfs.lua#L39-L40)、[鼹鼠](../../../raw/current/scripts/brains/molebrain.lua#L56-L73)、[陷阱](../../../raw/current/scripts/components/trap.lua#L264-L267) |
| 浮水 | `MakeInventoryFloatable(inst, "med", nil, 0.65)` | [浮水](../../../raw/current/scripts/prefabs/turfs.lua#L42) |
| 拾取音效 | `inst.pickupsound = data.pickupsound`（`rock`／`cloth`／`squidgy` 等） | [字段](../../../raw/current/scripts/prefabs/turfs.lua#L30) |
| 动作文案 | `groundtile` 标签让 `DEPLOY.strfn` 返回 `"GROUNDTILE"` | [分支](../../../raw/current/scripts/actions.lua#L1644-L1650)、[文案](../../../raw/current/scripts/strings.lua#L560) |
| 外部消费者 | 蚁狮帽的容器过滤只收带 `groundtile` 且有 `tile` 字段的物品；`placer` 对 TURF 模式与 `groundtile` 一律禁用轴对齐放置 | [容器](../../../raw/current/scripts/containers.lua#L1283)、[放置器](../../../raw/current/scripts/components/placer.lua#L165-L184) |

## 来源

- **配方**：40 条**同名** `Recipe2("turf_…")`（配方名与注册名一致的那 40 条），科技分布是 `SCIENCE_TWO` 10 条、`TURFCRAFTING_TWO` 15 条、`MASHTURFCRAFTING_TWO` 1 条、`CELESTIAL_THREE` 2 条、`TECH.LOST` 12 条，每条都给 4 张。[配方段](../../../raw/current/scripts/recipes.lua#L692-L733)、[天体两条](../../../raw/current/scripts/recipes.lua#L842-L843)、[COTL 两条](../../../raw/current/scripts/recipes.lua#L1024-L1025)
- **别名产物与 nounlock**：同名配方之外还有 `wurt_turf_marsh`（`TECH.NONE`、`builder_tag = "merm_builder"`、`product = "turf_marsh"`、给 4 张），所以 `turf_marsh` 有两条配方；天体那两条带 `nounlock`（不进蓝图解锁体系、符合科技即可制作），见下一条。[沃特](../../../raw/current/scripts/recipes.lua#L109-L110)
- **制作站**：`TURFCRAFTING_TWO` 与 `MASHTURFCRAFTING_TWO` 都由土地夯实器（`turfcraftingstation`，Terra Firma Tamper）提供，它的 `prototyper.trees` 同时给两棵树 2 级。[制作站](../../../raw/current/scripts/prefabs/turfcraftingstation.lua#L129-L135)、[树](../../../raw/current/scripts/tuning.lua#L1429-L1432)、[科技](../../../raw/current/scripts/constants.lua#L1305-L1307)
- **`TECH.LOST`**：值是 `MAGIC／SCIENCE／ANCIENT 各 10 级`，任何常规制作站都提供不了；这 12 条不会因为站在某个制作站旁而解锁。[常量](../../../raw/current/scripts/constants.lua#L1324)
- **蓝图**：蓝图 prefab 由工厂按配方动态登记——`blueprint.lua` 遍历 `AllRecipes`，对 `CanBlueprintSpecificRecipe` 为真（有正科技等级、非 `nounlock`、非 `builder_tag`）的配方补一个 `<recipe>_blueprint`。因此 `turf_shellbeach_blueprint` 在本版**是真实注册**（`turf_shellbeach` 为 `TECH.LOST`、无 `nounlock`、无 `builder_tag`），同时它还是隐士蟹商店配方 L934 的产物（3 个空瓶）；`turf_meteor`／`turf_fungus_moon` 因 `nounlock` 被排除，`TURF_METEOR_BLUEPRINT` 只有名称键、没有对应 prefab。配方与成本细节见 [[reference/prefabs/shell-materials]]。[筛选](../../../raw/current/scripts/prefabs/blueprint.lua#L62-L74)、[登记](../../../raw/current/scripts/prefabs/blueprint.lua#L237-L244)、[商店配方](../../../raw/current/scripts/recipes.lua#L934)、[蓝图名](../../../raw/current/scripts/languages/chinese_s.po#L351055-L351057)
- **挖起**：带 `terraformer` 的工具先过门禁——`plow` 为真走 `CanPlowAtPoint`，否则走 `CanTerraformAtPoint`（免疫地块、非陆地、`terraform_extra_spacing` 内的阻挡都返回 false）；通过后按 `self.turf`、`undertile:GetTileUnderneath`、`WORLD_TILES.DIRT` 的顺序选新地块并 `SetTile`。最后按回调分两路：设了 `onterraformfn` 就只走自定义回调（`farm_plow` 是这条，`terraformer.turf = FARMING_SOIL`，不产地皮物品）；没有回调才 `HandleDugGround`，按被挖地块的 turf 定义反推 `turf_<name>`、继承地面湿度，地块没有 turf 定义则只生成 `sinkhole_spawn_fx_N`。[门禁](../../../raw/current/scripts/components/terraformer.lua#L12-L26)、[选地与分流](../../../raw/current/scripts/components/terraformer.lua#L28-L32)、[产物](../../../raw/current/scripts/componentutil.lua#L31-L45)、[叉子](../../../raw/current/scripts/prefabs/pitchfork.lua#L73-L79)、[犁](../../../raw/current/scripts/prefabs/farm_plow.lua#L230-L233)
- **活动地皮**：两块 COTL 主题地皮（`turf_cotl_gold`、`turf_cotl_brick`）共用 `turf_cotl` 动画包，配方是 `TECH.LOST`；本版 `recipes.lua` 没有给这两条加活动开关字段，实际是否在制作菜单可见取决于菜单过滤等其它环节，未核验。

### 不产生物品的地块

本版 `tiledefs.lua` 共 78 次 `AddTile` 调用，其中 67 次带地面定义、40 次带 turf 定义，剩下 27 块地块**没有** `turf_*` 物品：8 块海洋、5 块 Gorge 地形、`LUNAR_MARSH`、`SHADOW_MARSH`、`DIRT`、`VAULT`、`MONKEY_DOCK`、`OCEAN_ICE`、`CHARLIE_VINE`、`ROPE_BRIDGE`、`WAGSTAFF_FLOOR`、`QUAGMIRE_SOIL`、`FARMING_SOIL`、`LAVAARENA_TRIM`／`LAVAARENA_FLOOR`、`RIFT_MOON`。[枚举](../../../raw/current/scripts/tiledefs.lua#L96-L1643)

- `STRINGS.NAMES.TURF_DIRT`（“泥土地皮”）这个键存在，但 `GROUND.DIRT` 没有 turf 定义，所以**没有** `turf_dirt` 物品；`DIRT` 正是铺地皮时要踩的那块地。[键](../../../raw/current/scripts/languages/chinese_s.po#L350919-L350922)、[地块](../../../raw/current/scripts/tiledefs.lua#L641-L659)
- `STRINGS.NAMES.TURF_METEOR_BLUEPRINT`（环形山地皮蓝图）与 `STRINGS.NAMES.TURF_SHELLBEACH_BLUEPRINT`（贝壳地皮蓝图）都是蓝图名、不是地皮物品；差别在于**贝壳蓝图是真实注册的 prefab**，环形山蓝图因 `nounlock` 被排除、只剩名称键（见[来源](#来源)）。[键](../../../raw/current/scripts/languages/chinese_s.po#L350970-L350972)、[键](../../../raw/current/scripts/languages/chinese_s.po#L351055-L351057)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：deployable | TURF 模式、`ondeploy`、`usegridplacer` | 主模拟设值，replica 同步 | [[reference/components/deployable#关键接口]]；[接线](../../../raw/current/scripts/prefabs/turfs.lua#L66-L69) |
| 组件：placer | 由 `gridplacer` 承担预览，`snap_to_tile` 对齐地块 | 客户端，非联网 | [[reference/components/placer#关键接口]]；[gridplacer](../../../raw/current/scripts/prefabs/gridplacer.lua#L16-L43) |
| 组件：stackable | 上限 20 | 主模拟 | [[reference/components/stackable#关键接口]]；[堆叠](../../../raw/current/scripts/prefabs/turfs.lua#L52-L53) |
| 组件：fuel／burnable／propagator | 燃料 7.5 秒、燃尽 15 秒、小范围蔓延 | 主模拟 | [[reference/components/fuel#关键接口]]；[组件段](../../../raw/current/scripts/prefabs/turfs.lua#L60-L64) |
| 组件：bait | 与 `molebait` 标签配合成为诱饵 | 主模拟 | [[reference/components/bait#关键接口]]；[诱饵](../../../raw/current/scripts/prefabs/turfs.lua#L39-L40) |
| 组件：terraformer | 叉子挖起地块时反推出同名 turf 物品；`farm_plow` 走自定义回调不产 | 世界态工具，服务端 | [[reference/components/terraformer#关键接口]]；[挖起](../../../raw/current/scripts/components/terraformer.lua#L12-L32) |
| API／数据：`Map:SetTile`／`GetTileCoordsAtPoint` | 部署时改地块并推 `onterraform` | 服务端；客户端走同一判定函数 | [包装](../../../raw/current/scripts/components/map.lua#L3-L8) |
| API／数据：`TileManager.AddTile` | 地块定义里第 6 个参数即 turf 定义，决定是否生成物品 | 启动期注册，模组可加 | [登记](../../../raw/current/scripts/tilemanager.lua#L205-L209)、[表](../../../raw/current/scripts/worldtiledefs.lua#L204-L219) |
| 关联 Prefab：`turfcraftingstation` | TURFCRAFTING／MASHTURFCRAFTING 两棵树的来源 | 建筑，世界设施 | [制作站](../../../raw/current/scripts/prefabs/turfcraftingstation.lua#L129-L135) |
| 关联 Prefab：`pitchfork` | 挖起地块并产出同名 turf 物品 | 工具 | [[reference/prefabs/tools-field-index]]；[接线](../../../raw/current/scripts/prefabs/pitchfork.lua#L73-L79) |
| 关联 Prefab：`turf_shellbeach_blueprint` | 贝壳海滩地皮的图纸来源 | 蓝图物品 | [[reference/prefabs/shell-materials]]；[配方](../../../raw/current/scripts/recipes.lua#L934) |
| 资源：anim 包 | 9 种 `turf*` 动画包，按 `bank_build` 取 | 客户端段加载 | [资产](../../../raw/current/scripts/prefabs/turfs.lua#L75-L79) |

## 典型接入

### 把“只能铺在 DIRT”放宽到任意可种植陆地

- 前提与选择：服务端＋客户端 `modmain.lua`。判定在引擎 `Map` 类的方法上，**不是挂载型组件**：世界 prefab 用 `require "components/map"` 给 `Map` 加方法、实体侧走 `inst.entity:AddMap()`，本版 Lua 没有 `AddComponent("map")` 入口，所以 `AddComponentPostInit("map", ...)` 不会执行。也不能在 modmain 顶层就读方法再包装：`ModManager:LoadMods()` 先跑 modmain，`components/map.lua` 要等世界 prefab 加载时才定义该方法，冷启动时顶层包装会被随后的原版定义覆盖（捕获值也可能为空）。改在**世界 prefab 的 postinit**（`AddPrefabPostInitAny` 筛 `TheWorld`）里安装，见下一条。[modmain 时机](../../../raw/current/scripts/mods.lua#L539-L545)、[世界 require](../../../raw/current/scripts/prefabs/world.lua#L1-L2)、[引擎构造](../../../raw/current/scripts/prefabs/world.lua#L447-L452)、[原方法](../../../raw/current/scripts/components/map.lua#L162-L164)
- 生效与撤销：改在**世界 prefab 的 postinit** 里安装——`AddPrefabPostInitAny` 的回调由 `mainfunctions` 的 `SpawnPrefab` 包装在工厂返回后逐个调用，此时 `TheWorld` 已赋值（`world.lua` 在工厂内 `TheWorld = inst`）且已 `AddMap()`，不依赖主模拟。它跟着世界 prefab 的生成走：宿主／专服是本地开局或读档，远程加入的客户端则在 `TheNet:StartClient` 之后另起客端世界；而 `ModManager:SimPostInit` 全脚本只有 `gamelogic` 一处调用点（本地初始化流程），覆盖不到远程加入的客端。已铺下的地块不回滚。[世界构造](../../../raw/current/scripts/prefabs/world.lua#L421-L426)、[工厂内 AddMap](../../../raw/current/scripts/prefabs/world.lua#L447-L452)、[postinit 调用点](../../../raw/current/scripts/mainfunctions.lua#L409-L411)、[SimPostInit 唯一调用点](../../../raw/current/scripts/gamelogic.lua#L957-L958)、[远程加入](../../../raw/current/scripts/gamelogic.lua#L1198)
- 全局影响：除 TURF 部署的两处判定外，蚁狮帽的 `AutoTerraformer:DoTerraform` 也读这个方法。整条分支要先满足“槽内第 1 格是带 `tile` 的地皮、且与目标格**当前地块不同**”（相同会先 `return`）；满足后再看 `CanPlaceTurfAtPoint`——为真就**扣掉槽内那张、直接 `SetTile` 铺上并 `FinishTerraforming`**，不再走挖起分支。所以放宽后要盯的是“地皮与目标格不同、原版拒绝、包装放行”的落点。[服务端](../../../raw/current/scripts/components/deployable.lua#L110-L111)、[客户端](../../../raw/current/scripts/components/inventoryitem_replica.lua#L327-L340)、[蚁狮帽](../../../raw/current/scripts/components/autoterraformer.lua#L57-L77)、[挂载](../../../raw/current/scripts/prefabs/hats.lua#L2433-L2437)
- 原版依据：原实现只认 `DIRT`，见[部署：TURF 模式与 grid 预览](#部署turf-模式与-grid-预览)；`CanPlantAtPoint` 的门槛是“陆地且不是 `GROUND_HARD`”，比原规则宽但仍是陆地。[原实现](../../../raw/current/scripts/components/map.lua#L162-L164)、[可种植](../../../raw/current/scripts/components/map.lua#L167-L175)

```lua
-- modmain.lua：世界对象就绪后包装一次；宿主、专服与远程加入的客端都会走到
local function WrapCanPlaceTurfAtPoint(inst)
    if inst ~= GLOBAL.TheWorld or GLOBAL.Map._wiki_turf_relaxed then
        return
    end
    GLOBAL.Map._wiki_turf_relaxed = true

    local CanPlaceTurfAtPoint = GLOBAL.Map.CanPlaceTurfAtPoint
    GLOBAL.Map.CanPlaceTurfAtPoint = function(self, x, y, z)
        return CanPlaceTurfAtPoint(self, x, y, z) or self:CanPlantAtPoint(x, y, z)
    end
end

AddPrefabPostInitAny(WrapCanPlaceTurfAtPoint)
```

- 验收分开两条路径：**宿主／专服**——世界由本地开局或读档构造，`AddPrefabPostInitAny` 在工厂返回后执行，包装生效；**远程加入的客户端**——`TheNet:StartClient` 之后客端自行装载世界，同样走 prefab 工厂与 postinit-any，包装也生效。两条路径上服务端动作都是 `ACTIONS.DEPLOY.fn` → `Deployable:CanDeploy` → `TheWorld.Map:CanPlaceTurfAtPoint`；客端预览是 `replica.inventoryitem:CanDeploy` 的 TURF 分支调同一个方法。四种落点：① `DIRT` → 原 true／包装 true；② 普通可种植陆地（非 `DIRT`、非硬地）→ 原 false／包装 true（`CanPlantAtPoint` 为真）；③ 硬地（`GROUND_HARD`，地板／岩石类）→ 两者 false；④ 水面 → 两者 false（`IsLandTile` 为假）。蚁狮帽另验“槽内地皮与目标格不同且原版拒绝、包装放行”的落点：包装后走扣槽内铺上分支而不是挖起。[执行](../../../raw/current/scripts/components/deployable.lua#L104-L113)、[预览](../../../raw/current/scripts/components/inventoryitem_replica.lua#L327-L340)、[硬地与陆地](../../../raw/current/scripts/components/map.lua#L167-L175)
- 调整与验证：参数沿用 `(x, y, z)`、返回布尔；只想放开到某几块地块就把 `CanPlantAtPoint` 换成白名单（如 `self:GetTileAtPoint(x, y, z) == GLOBAL.WORLD_TILES.FOREST`）。只装在服务端时客户端预览仍按原规则变红。

## 常见陷阱与限制

- **误区：“地皮可以铺在任何空地上”**：`CanPlaceTurfAtPoint` 只认 `WORLD_TILES.DIRT`，所以要先挖掉原地皮才铺得上；这也解释了为什么 `DIRT` 自己反而没有地皮物品。[判定](../../../raw/current/scripts/components/map.lua#L162-L164)
- **误区：“TURF 模式会按 `DEPLOYSPACING` 校验间距”**：该分支只调 `CanPlaceTurfAtPoint`，没有 `IsDeployPointClear`；地皮也没有 `SetDeploySpacing` 调用。[分支](../../../raw/current/scripts/components/deployable.lua#L110-L111)
- **边界：扣件在地块改动之后，且不核实地块结果**：`SetTile` 之后立刻取 1 张移除，既不检查引擎是否真的改了地块，也没有“改地失败就退件”的分支；因此不能靠“放置失败”来避免消耗（能拦住消耗的只有 `CanDeploy` 的落点判定）。[回调](../../../raw/current/scripts/prefabs/turfs.lua#L9-L17)
- **误区：“有 `STRINGS.NAMES.TURF_*` 就一定对应一个地皮物品”**：`TURF_DIRT` 没有地块 turf 定义、`TURF_METEOR_BLUEPRINT` 因 `nounlock` 没有具体蓝图注册；但 `TURF_SHELLBEACH_BLUEPRINT` 是真实注册的蓝图 prefab，不能一并否定。地皮物品的判定标准应是 `GroundTiles.turf` 是否有该地块键。[TURF_DIRT](../../../raw/current/scripts/languages/chinese_s.po#L350919-L350922)、[环形山蓝图](../../../raw/current/scripts/languages/chinese_s.po#L350970-L350972)、[贝壳蓝图](../../../raw/current/scripts/languages/chinese_s.po#L351055-L351057)
- **边界：骑乘或手持浮标时不能铺**：这是所有部署物共用的 `IsDeployable` 前置，地皮没有例外标签。[前置](../../../raw/current/scripts/components/deployable.lua#L89-L102)
- **未核验**：两块 COTL 地皮是否受活动开关限制、`turf_*` 在炼金／拆解类配方中的再利用（如 `endtable`、`hermithouse_construction3` 把地皮当原料）的完整清单与本页编号未逐条覆盖。

## 验证记录

- 源码审阅：2026-11-10，Steam buildid 26474873。逐行读完 `turfs.lua`（工厂、资产、组件段、部署段、循环）与 `gridplacer.lua` 的公共预览工厂；用注释感知的括号解析遍历 `tiledefs.lua` 的 78 次 `AddTile`，按第 6 个参数枚举出 40 个 turf 键并逐条抓取 `name`／`anim`／`bank_build`，再与 `GroundTiles.turf` 的登记路径、`HandleDugGround`、`Terraformer:Terraform`、`Map:CanPlaceTurfAtPoint`、`deployable` 的 TURF 分支、`inventoryitem_replica` 的 placer 取名与 TURF 预览分支对照。
- 名称核验（`inspect_prefab_names.py`，只认 `languages/chinese_s.po` 的 msgstr）：40 个 `STRINGS.NAMES.TURF_*` 逐一取值，均落在本版 `chinese_s.po` 的 L350870–L351082 区段，无 fuzzy 条目。[ARCHIVE](../../../raw/current/scripts/languages/chinese_s.po#L350870-L350872)、[FOREST](../../../raw/current/scripts/languages/chinese_s.po#L350930-L350932)、[WOODFLOOR](../../../raw/current/scripts/languages/chinese_s.po#L351080-L351082)
- 游戏运行：未运行。秒数为 tuning 推导（`seg_time = 30`，`TINY_FUEL`＝7.5 秒、`MED_BURNTIME`＝15 秒）；数量与科技分布来自 `recipes.lua` 逐条抓取。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 结果见本批报告。

## 相关链接

- 材料对照：[[reference/prefabs/shell-materials]]（贝壳海滩地皮图纸）、[[reference/prefabs/basic-mineral-materials]]（石头、燧石等配方原料）
- 组件：[[reference/components/deployable]]、[[reference/components/placer]]、[[reference/components/terraformer]]、[[reference/components/bait]]、[[reference/components/stackable]]

返回 [[reference/prefabs/index|Prefab 索引]]。
