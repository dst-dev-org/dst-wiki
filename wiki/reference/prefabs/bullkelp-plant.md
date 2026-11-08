---
title: "公牛海带（bullkelp_plant）、水下叶片与冲岸根（bullkelp_beachedroot）"
created: "2026-11-08"
updated: "2026-11-09"
type: prefab
summary: "海面植株 bullkelp_plant 及其两个附属注册：pickable 的 3 天再生与空株随机帧、海面燃烧与作祟、被推上岸后的防抖判定与「有叶先摘、无叶只换茎」；水下叶片子实体；布景冲岸根 bullkelp_beachedroot 的两条拾取转化路径与月岛／隐士岛布点。"
aliases: ["bullkelp_plant", "bullkelp_plant_leaves", "bullkelp_beachedroot", "公牛海带", "海带丛", "冲岸的海带", "公牛海带叶片", "Bull Kelp"]
tags: [prefab]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/bullkelp_plant.lua", "prefabs/bullkelp_beached.lua", "prefabs/bullkelp_root.lua", "prefabs/books.lua", "components/pickable.lua", "components/map.lua", "standardcomponents.lua", "entityscript.lua", "simutil.lua", "tuning.lua", "constants.lua", "map/forest_map.lua", "map/terrain.lua", "map/rooms/forest/terrain_moonisland.lua", "map/rooms/forest/terrain_ocean.lua", "map/static_layouts/hermitcrab_01.lua", "map/static_layouts/retrofit_moonisland_large.lua", "map/layouts.lua", "components/retrofitforestmap_anr.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 公牛海带（bullkelp_plant）、水下叶片与冲岸根（bullkelp_beachedroot）

## 快速判断

- 职责与适用：三个注册分属两个文件。`prefabs/bullkelp_plant.lua` 导出海面植株 `bullkelp_plant` 与水下叶片 `bullkelp_plant_leaves`，`prefabs/bullkelp_beached.lua` 导出冲岸根 `bullkelp_beachedroot`。植株是海上可采植物（`pickable`＋小地图＋水面波纹），不是可入包物品；冲岸根是**布景形态**的可拾取实体，拾取时整体换成两件物品。[植株与叶片导出](../../../raw/current/scripts/prefabs/bullkelp_plant.lua#L150-L151)、[冲岸根注册](../../../raw/current/scripts/prefabs/bullkelp_beached.lua#L88-L88)
- 优先使用：改再生时长看[采摘与再生](#采摘与再生)；理解「漂上岸会变成什么」看[冲岸判定与后果](#冲岸判定与后果)；只是要海带叶／茎的数值与用途，直接去 [[reference/prefabs/kelp]]、[[reference/prefabs/bullkelp_root]]，本页不重复。
- 重要副作用：冲岸时先尝试 `pickable:Pick(TheWorld)`（通过门禁就掉 1 片海带叶在地上），随后**无条件**移除植株并在原位生成 `bullkelp_root`（可拾取武器，40 天保质）；冲岸根被拾取或入包时同样换成 `kelp`＋`bullkelp_root` 各 1（须拿到接收容器）并继承湿度。[冲岸](../../../raw/current/scripts/prefabs/bullkelp_plant.lua#L42-L54)、[替换](../../../raw/current/scripts/prefabs/bullkelp_beached.lua#L15-L35)
- 修改边界：海带叶三态（生／熟／干的数值、干制、修补材质）见 [[reference/prefabs/kelp]]；公牛海带茎的武器面、折断与种回水面见 [[reference/prefabs/bullkelp_root]]；海带补丁 `boatpatch_kelp` 见 [[reference/prefabs/boat-patches]]。本页只写植株、水下叶片与冲岸根三件。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `bullkelp_plant` | `fn`：entity 的 transform／anim／sound／minimap／network → `MiniMapEntity:SetIcon("bullkelp_plant.png")` → `MakeInventoryPhysics(inst, nil, 0.7)` → `SetDeploySmartRadius(DEPLOYSPACING_RADIUS[DEPLOYSPACING.MEDIUM] / 2)` → bank／build `bullkelp`＋`PlayAnimation("idle", true)` → `SetPristine` → 主模拟：生成叶片子实体、随机起始帧、随机色调、pickable、inspectable、燃烧与作祟、碰撞回调 | 海上植物（无 `inventoryitem`、无装备／食物职责）；`kelp` tag 在 `SetPristine` 前加，双端可见。[工厂](../../../raw/current/scripts/prefabs/bullkelp_plant.lua#L63-L123)、[小地图与物理](../../../raw/current/scripts/prefabs/bullkelp_plant.lua#L72-L82) |
| `bullkelp_plant_leaves` | `underwaterleafsfn`：transform／anim／network → bank／build `bullkelp_underwater`＋`idle` 循环 → `SetLayer(LAYER_BACKGROUND)` → `AddTag("DECOR")` → `SetPristine` → 主模拟只写 `persists = false` | 纯表现子实体，无组件、无回调用；服务端建、带 `AddNetwork` 故客户端也有同步副本。[叶片工厂](../../../raw/current/scripts/prefabs/bullkelp_plant.lua#L125-L148) |
| `bullkelp_beachedroot` | `fn`（bullkelp_beached.lua）：transform／anim／network → `MakeInventoryPhysics(inst)` → bank／build `bullkelp`＋`PlayAnimation("dropped_beached")` → `MakeInventoryFloatable` → `SetPrefabNameOverride("BULLKELP_PLANT")` → `SetPristine` → 主模拟：inspectable、inventoryitem（两条替换回调）、可燃可蔓延、作祟 | 布景用的「冲上岸的海带」，名字显示为「公牛海带」；自身不带 `perishable`，价值全在拾取瞬间的那次替换。[工厂](../../../raw/current/scripts/prefabs/bullkelp_beached.lua#L51-L86) |

构造细节按顺序落位：[资源与依赖](../../../raw/current/scripts/prefabs/bullkelp_plant.lua#L1-L17) 里已声明 `kelp`、`bullkelp_root`、`bullkelp_plant_leaves` 三个依赖；`SetFinalOffset(1)` 只调绘制层级；`AddDefaultRippleSymbols(inst, true, false)` 把 `water_ripple` 符号换成水面波纹、不加重影，所以它浮在海上时自带波纹。[渲染与波纹](../../../raw/current/scripts/prefabs/bullkelp_plant.lua#L77-L82)、[波纹 helper](../../../raw/current/scripts/standardcomponents.lua#L1762-L1770)

两处数值含义：`MakeInventoryPhysics(inst, nil, 0.7)` 里第二参省略用默认质量 1、第三参是碰撞球半径 0.7（碰撞组 `COLLISION.ITEMS`）；`SetDeploySmartRadius(1 / 2)` 声明本实体的占位半径 0.5，部署判定时与附近调用过同一接口的实体按双方半径相加算间距，注释指明「公牛海带茎部署间距的一半」。[物品物理](../../../raw/current/scripts/standardcomponents.lua#L371-L386)、[间距表与说明](../../../raw/current/scripts/constants.lua#L2248-L2263)、[调用](../../../raw/current/scripts/prefabs/bullkelp_plant.lua#L74-L75)

名称：`STRINGS.NAMES.BULLKELP_PLANT`＝公牛海带（Bull Kelp），冲岸根用 `SetPrefabNameOverride("BULLKELP_PLANT")` 直接借这条译文，玩家看到的名字与植株相同；`GetBasicDisplayName` 优先读 `nameoverride` 查 `STRINGS.NAMES`。[名称键](../../../raw/current/scripts/languages/chinese_s.po#L339975-L339977)、[nameoverride 消费](../../../raw/current/scripts/entityscript.lua#L687-L692)

### 采摘与再生

- 接线：`pickable:SetUp("kelp", TUNING.BULLKELP_REGROW_TIME)`，只传产物与再生时长：第三参是 `numtoharvest`（每次产量），轮次走 `max_cycles`／`cycles_left`，本处都没设；`picksound = "turnoftides/common/together/water/harvest_plant"`；三个回调分别负责采摘、再生、空株三套动画。[接线](../../../raw/current/scripts/prefabs/bullkelp_plant.lua#L104-L109)、[SetUp](../../../raw/current/scripts/components/pickable.lua#L222-L228)
- 时长：`BULLKELP_REGROW_TIME = total_day_time * 3`，1 天＝`seg_time`(30)×16＝480 秒，故 3 天＝1440 秒；春季 `SpringGrowthMod` 把再生时长乘 `SPRING_GROWTH_MODIFIER`＝0.75，即 1080 秒（2.25 天）。[常量](../../../raw/current/scripts/tuning.lua#L3798-L3798)、[时间基准](../../../raw/current/scripts/tuning.lua#L19-L20)、[春季修正](../../../raw/current/scripts/simutil.lua#L563-L565)、[修正值](../../../raw/current/scripts/tuning.lua#L2284-L2284)
- 采摘：`Pick` 先走单产物分支生成 1 个 `kelp`（继承世界湿度；`picker = TheWorld` 时没有 inventory，物品就落在植株坐标），再调 `onpickedfn`、置 `canbepicked = false`，按 `baseregentime` 起再生任务。[Pick](../../../raw/current/scripts/components/pickable.lua#L535-L583)、[单产物落地](../../../raw/current/scripts/components/pickable.lua#L493-L513)
- 动画：采摘播 `picking` 再推 `picked`（循环），再生播 `grow` 再推 `idle`；两个函数都同时对植株与 `inst.underwater` 叶片执行，所以水上水下同一节奏。[采摘动画](../../../raw/current/scripts/prefabs/bullkelp_plant.lua#L19-L24)、[再生动画](../../../raw/current/scripts/prefabs/bullkelp_plant.lua#L26-L31)
- 空株帧：`makeemptyfn` 把两套动画都停在 `picked` 的**随机帧**（`math.random(GetCurrentAnimationNumFrames()) - 1`），因此每株被采空后的「秃」程度不一样；同一函数也是读档与 `MakeEmpty`（例如种下空株）时的统一入口。[空株随机帧](../../../raw/current/scripts/prefabs/bullkelp_plant.lua#L33-L40)、[MakeEmpty](../../../raw/current/scripts/components/pickable.lua#L426-L448)
- 没有 `remove_when_picked` 也没有 `max_cycles`：植株在原地循环，不枯萎、不计轮次。

### 燃烧、作祟与碰撞

- 燃烧：`MakeSmallBurnable(inst, TUNING.SMALL_FUEL)` 把**燃料值常量**当燃烧时长传入，实际烧 `SMALL_FUEL`＝`seg_time`×0.5＝15 秒（不是 `SMALL_BURNTIME`）；`MakeSmallPropagator` 给引燃点 5～10、传播半径 3～5、传播时造成伤害。[接线](../../../raw/current/scripts/prefabs/bullkelp_plant.lua#L114-L116)、[小燃可燃](../../../raw/current/scripts/standardcomponents.lua#L164-L174)、[传播工厂](../../../raw/current/scripts/standardcomponents.lua#L200-L213)、[常量](../../../raw/current/scripts/tuning.lua#L1674-L1674)
- 烧完：默认 `DefaultBurntFn` 在**非海面**点位生成 1 份 `ash` 再 `Remove`；公牛海带长在海面，`IsOceanAtPoint` 为真，所以烧毁后连灰都不留。[烧毁](../../../raw/current/scripts/standardcomponents.lua#L18-L44)
- 作祟：`MakeHauntableIgnite` 在当前构建的点火分支整段被注释，只装 `hauntable` 组件并设冷却（`HAUNT_COOLDOWN_MEDIUM`＝5 秒），作祟回调恒返回 false；对本植株作祟不会点火、不产生产物。[工厂](../../../raw/current/scripts/standardcomponents.lua#L1173-L1189)、[冷却常量](../../../raw/current/scripts/tuning.lua#L3045-L3045)
- 碰撞：`Physics:SetCollisionCallback(OnCollide)` 是「被外力推上岸」的唯一入口。碰撞回调调用极频繁，`OnCollide` 只用 `inst._checkgroundtask` 做一次性防抖，排 `1 + math.random()` 秒后的 `CheckBeached`；构造时也直接排一次同函数，覆盖「生成时就已在陆地／平台」的情况。[登记](../../../raw/current/scripts/prefabs/bullkelp_plant.lua#L119-L120)、[防抖](../../../raw/current/scripts/prefabs/bullkelp_plant.lua#L56-L61)

### 冲岸判定与后果

`CheckBeached` 的判定是**或**关系，两个条件任一成立即算冲岸：站在某个平台上（`inst:GetCurrentPlatform() ~= nil`，木船甲板属此类），或该点已是可见地面（`TheWorld.Map:IsVisualGroundAtPoint(x, y, z)`，含地图的 overhang 部分）。[判定](../../../raw/current/scripts/prefabs/bullkelp_plant.lua#L42-L54)、[overhang 说明](../../../raw/current/scripts/components/map.lua#L10-L10)

命中后按固定顺序做三件事：

1. `pickable:Pick(TheWorld)`——以世界为采摘者走一次正常采摘，只有 `canbepicked`／`caninteractwith` 都通过（且未 `stuck`）才掉 1 片 `kelp` 在地上；采空状态下这一次调用什么也不产。
2. `inst:Remove()`——植株本体（含叶片子实体）消失，不留残株。
3. `SpawnPrefab("bullkelp_root")` 并 `SetPosition` 到同一坐标——生成的是**公牛海带茎**，不是 `bullkelp_beachedroot`。

为什么不是冲岸根，源码注释写得很直白：这次是靠外力冲上岸的，不该伪装成布景用的冲岸根，而应发「会过期的东西」——`bullkelp_root` 自带 `perishable`（40 天），`bullkelp_beachedroot` 没有。[注释原句](../../../raw/current/scripts/prefabs/bullkelp_plant.lua#L43-L43)、[茎的保质](../../../raw/current/scripts/tuning.lua#L2051-L2051)

玩家可见后果：船撞上藤丛、或海带被浪推到岸／甲板后 1～2 秒内，水面那株消失，地上多出 1 片可捡的海带叶（若当时未采空）与 1 根可拾取的公牛海带茎；叶子躺在地上会按地面腐坏速率老化，茎则开始 40 天保质倒计时。若植株当时已被采空，只剩茎。

### 水下叶片（bullkelp_plant_leaves）

- 归属：`inst.underwater = SpawnPrefab("bullkelp_plant_leaves")` 后 `entity:SetParent(inst.entity)`，只登记**引擎父子**（不是 Lua `children`），坐标归零随父实体移动。[生成与父级](../../../raw/current/scripts/prefabs/bullkelp_plant.lua#L92-L94)
- 表现：`DECOR` tag＋`LAYER_BACKGROUND` 层，画在水面之下；`persists = false` 不写存档，读档时植株构造会重新生成一份并重新随机起始帧。[工厂](../../../raw/current/scripts/prefabs/bullkelp_plant.lua#L125-L148)、[随机帧](../../../raw/current/scripts/prefabs/bullkelp_plant.lua#L97-L99)
- 动画同步：叶片自身没有任何回调，全部由植株的 `onpickedfn`／`onregenfn`／`makeemptyfn` 直接调 `inst.underwater.AnimState`，因此两层永远同帧；要改水下表现只能改这三个函数。[三套回调](../../../raw/current/scripts/prefabs/bullkelp_plant.lua#L19-L40)

### 冲岸根（bullkelp_beachedroot）

- 定位：这是**布景件**，只由世界生成与静态布景放置（见下），不会由植株冲岸产生；它可被拾取、可燃、可作祟，但没有 `perishable` 也没有 `pickable`。[工厂](../../../raw/current/scripts/prefabs/bullkelp_beached.lua#L51-L86)
- 拾取路径：`inventoryitem:SetOnPickupFn(onpickup)` → `ReplaceOnPickup`：先读自身的 `GetMoisture()`／`IsWet()`，`inst:Remove()`，再在拿到容器时生成 `kelp` 与 `bullkelp_root` 各 1，两者都 `InheritMoisture`，最后 `container:GiveItem` 逐件入包；回调返回 true（实体已被移除）。[替换](../../../raw/current/scripts/prefabs/bullkelp_beached.lua#L15-L35)、[拾取](../../../raw/current/scripts/prefabs/bullkelp_beached.lua#L37-L40)
- 入包兜底：`SetOnPutInInventoryFn(onputininventory)` 处理「跳过 `OnPickup` 直接进容器」的路径（源码举例：Woby 拾取时没有 inventory 组件），容器取 `owner.components.container or owner.components.inventory`；已经走过 `OnPickup` 的实体此时已被移除，不会重复给件；两件的生成都在 `if container then` 内，容器为空时实体已被移除、什么也不掉。[兜底](../../../raw/current/scripts/prefabs/bullkelp_beached.lua#L42-L49)
- 燃烧与作祟：`MakeMediumBurnable(inst, TUNING.LARGE_BURNTIME)`＝30 秒＋`MakeSmallPropagator`＋`MakeHauntableIgnite`（同样是被注释过的点火分支，实际只挂组件）。[接线](../../../raw/current/scripts/prefabs/bullkelp_beached.lua#L80-L83)、[时长常量](../../../raw/current/scripts/tuning.lua#L1683-L1683)
- 世界里谁放它：见[布点出处](#布点出处)。

### 布点出处

| 来源 | 内容与证据 |
|---|---|
| 世界生成设置 | 地表世界设置 `moon_bullkelp` 翻译成 `{"bullkelp_beachedroot"}`（月岛海带）；同一张表里 `ocean_bullkelp` 对应海面的 `bullkelp_plant`。[设置表](../../../raw/current/scripts/map/forest_map.lua#L235-L235)、[海面项](../../../raw/current/scripts/map/forest_map.lua#L242-L242) |
| 月岛地形房间 | `MoonIsland_IslandShard`：散布率 0.17、`bullkelp_beachedroot` 权重 1.5；`MoonIsland_Beach`：散布率 0.18、权重 1.25。[碎岛](../../../raw/current/scripts/map/rooms/forest/terrain_moonisland.lua#L2-L29)、[海滩](../../../raw/current/scripts/map/rooms/forest/terrain_moonisland.lua#L31-L55) |
| 地块白名单 | `bullkelp_beachedroot` 只允许落在 ROAD／WOODFLOOR／SCALE／CARPET／CHECKER／ROCKY／DIRT／DESERT_DIRT／METEOR／MONKEY_DOCK 这些地块上。[白名单](../../../raw/current/scripts/map/terrain.lua#L122-L122) |
| 隐士岛布景 | `hermitcrab_01` 静态布景固定放 8 处（`type = "bullkelp_beachedroot"`，坐标为布景内绝对坐标）。[第 1 处](../../../raw/current/scripts/map/static_layouts/hermitcrab_01.lua#L421-L429) |
| 老存档补丁 | `retrofit_moonisland_large`（另有 medium／small）为已生成的月岛补放同类布景件；海面另有 `TurnOfTidesRetrofitting_PopulateOcean` 给旧档 `OCEAN_COASTAL` 按 0.01／权重 2 补 `bullkelp_plant`。[补丁布景](../../../raw/current/scripts/map/static_layouts/retrofit_moonisland_large.lua#L4841-L4849)、[海面补种](../../../raw/current/scripts/components/retrofitforestmap_anr.lua#L163-L171) |
| 海面植株 | `OceanCoastal` 房间：散布率 0.01，`bullkelp_plant` 权重 6，另按 `BullkelpFarmSmall`×6／`BullkelpFarmMedium`×3 放固定海带田布局，布局的 `kelp_area` 另按 0.9 概率补 1 株。[海面房间](../../../raw/current/scripts/map/rooms/forest/terrain_ocean.lua#L15-L33)、[海带田布局](../../../raw/current/scripts/map/layouts.lua#L1074-L1086) |

本页只给出处与权重，房间／布局内部流程归世界生成主题。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：pickable | `SetUp("kelp", 3 天)`＋三个动画回调；无轮次、无 `remove_when_picked` | 主模拟；客户端靠 `pickable` tag 判断可交互 | [[reference/components/pickable#关键接口]]；[接线](../../../raw/current/scripts/prefabs/bullkelp_plant.lua#L104-L109) |
| 组件：inspectable | 植株与冲岸根各挂一个空配置的 inspectable | 主模拟；检视文案走公共 `DESCRIBE` 表 | [植株](../../../raw/current/scripts/prefabs/bullkelp_plant.lua#L111-L111)、[冲岸根](../../../raw/current/scripts/prefabs/bullkelp_beached.lua#L74-L74) |
| 组件：inventoryitem | 仅冲岸根：拾取／入包两条替换回调 | 主模拟；客户端只有 `inventoryitem_replica` 可用 | [接线](../../../raw/current/scripts/prefabs/bullkelp_beached.lua#L76-L78) |
| 组件：burnable／propagator | 植株 15 秒、冲岸根 30 秒；两者都会蔓延 | 主模拟 | [[reference/components/burnable#关键接口]]、[[reference/components/propagator#关键接口]]；[植株](../../../raw/current/scripts/prefabs/bullkelp_plant.lua#L114-L115)、[冲岸根](../../../raw/current/scripts/prefabs/bullkelp_beached.lua#L80-L81) |
| 组件：hauntable | 两件都 `MakeHauntableIgnite`，点火分支被注释，只剩冷却与恒 false 的回调 | 主模拟；冷却 5 秒 | [[reference/components/hauntable#关键接口]]；[工厂](../../../raw/current/scripts/standardcomponents.lua#L1173-L1189) |
| 组件：floater | 仅冲岸根：`MakeInventoryFloatable` 默认参数 | 公共端构造（`SetPristine` 前），双端都有 floater | [[reference/components/floater#关键接口]]；[调用](../../../raw/current/scripts/prefabs/bullkelp_beached.lua#L64-L64) |
| helper：`MakeInventoryPhysics`／`AddDefaultRippleSymbols`／`SetDeploySmartRadius` | 碰撞球 0.7、水面波纹符号、占位半径 0.5 | 公共端构造 | [物理](../../../raw/current/scripts/standardcomponents.lua#L371-L386)、[波纹](../../../raw/current/scripts/standardcomponents.lua#L1762-L1770)、[间距](../../../raw/current/scripts/constants.lua#L2248-L2263) |
| API／数据：`kelp` tag | 在 `SetPristine` 前加，双端可见；园艺学书本的候选表把 `kelp` 列入「可施法一族」但同表排除 `pickable`，所以只有被采空、tag 已摘掉的植株会被选中，效果归书本页面 | 双端可见，实际命中的是采空态 | [加标签](../../../raw/current/scripts/prefabs/bullkelp_plant.lua#L84-L84)、[候选表](../../../raw/current/scripts/prefabs/books.lua#L22-L23)、[筛选](../../../raw/current/scripts/prefabs/books.lua#L179-L186) |
| 关联 Prefab：`bullkelp_root` | 冲岸产物；反向 `ondeploy` 生成植株并 `MakeEmpty`（种下即空株，等 3 天再生） | 主模拟，种下即空 | [[reference/prefabs/bullkelp_root#获取途径]]；[部署](../../../raw/current/scripts/prefabs/bullkelp_root.lua#L53-L64) |
| 关联 Prefab：`kelp` | 采摘产物与冲岸掉落 | 主模拟生成 | [[reference/prefabs/kelp]]；[产物](../../../raw/current/scripts/prefabs/bullkelp_plant.lua#L106-L106) |
| 资源：anim | `anim/bullkelp.zip`（bank／build `bullkelp`，`idle`／`picking`／`picked`／`grow`／`dropped_beached`）、`anim/bullkelp_underwater.zip` | 只声明，不验证包内符号 | [植株资源](../../../raw/current/scripts/prefabs/bullkelp_plant.lua#L1-L5)、[叶片资源](../../../raw/current/scripts/prefabs/bullkelp_plant.lua#L7-L10) |

## 典型接入

### 把海带丛的再生周期从 3 天改成 5 天

- 前提与选择：服务端 `modmain.lua`。两条路一起用：改 `TUNING` 只对**注册后新构造**的实例生效（`fn` 构造时读表），已存在的植株要在 postinit 里改 `pickable.baseregentime`；`Pick` 每次都用 `baseregentime` 重算下次再生，所以改字段对之后每次采摘都有效。[读取点](../../../raw/current/scripts/prefabs/bullkelp_plant.lua#L104-L106)、[重算](../../../raw/current/scripts/components/pickable.lua#L559-L573)
- 生效与撤销：`TUNING` 改动停用模组即恢复；已排定的再生任务不会被回溯重排，只影响之后新起的那一次。春季仍会再乘 0.75。[春季修正](../../../raw/current/scripts/simutil.lua#L563-L565)
- 原版依据：默认值与时基见[采摘与再生](#采摘与再生)。

```lua
-- modmain.lua：海带丛采摘后 5 天再生（春季 3.75 天）
TUNING.BULLKELP_REGROW_TIME = GLOBAL.TUNING.TOTAL_DAY_TIME * 5

AddPrefabPostInit("bullkelp_plant", function(inst)
    if not GLOBAL.TheWorld.ismastersim or inst.components.pickable == nil then
        return
    end
    inst.components.pickable.baseregentime = GLOBAL.TUNING.BULLKELP_REGROW_TIME
end)
```

- 调整与验证：单位是秒，天数换算按 1 天＝480 秒；示例不改产物、不改冲岸逻辑（`CheckBeached` 是文件内 local，无法从模组替换，要改冲岸行为只能整体替换工厂）。未运行游戏验证。

## 常见陷阱与限制

- **误区：冲岸会生成 `bullkelp_beachedroot`**：植株冲岸生成的是 `bullkelp_root`（有保质期的茎，显示名「公牛海带茎」），冲岸根只是世界生成／布景摆件（借 `SetPrefabNameOverride` 显示成「公牛海带」）；两者不是一件东西，冲岸时茎必然落地，叶只在 `Pick` 门禁全过时才多掉 1 片，而冲岸根要等被拾取、且拿到接收容器后才换成「kelp＋茎」各 1。[冲岸](../../../raw/current/scripts/prefabs/bullkelp_plant.lua#L51-L52)、[注释](../../../raw/current/scripts/prefabs/bullkelp_plant.lua#L43-L43)
- **边界：冲岸最多掉 1 片叶，且要过门禁**：`Pick(TheWorld)` 与玩家采摘走同一套判定，采空、`caninteractwith` 为假或 `stuck` 时都不掉叶，叶落在植株坐标上而不是玩家背包里；茎不走这条门禁，照旧生成。[Pick](../../../raw/current/scripts/components/pickable.lua#L535-L536)
- **边界：烧海带丛拿不到灰**：`DefaultBurntFn` 只在不属于海面的点位生成 `ash`，海面上的植株烧完直接消失。[烧毁](../../../raw/current/scripts/standardcomponents.lua#L33-L41)
- **边界：燃耗常数被当燃烧时长用**：`MakeSmallBurnable(inst, TUNING.SMALL_FUEL)` 传的是燃料值 15，落在「燃烧时长」参数位，所以是烧 15 秒；照抄时别以为默认 10 秒。[helper 签名](../../../raw/current/scripts/standardcomponents.lua#L164-L168)
- **边界：`MakeHauntableIgnite` 不点火**：该名字在当前构建名不副实，点火段整段注释，作祟只设冷却；想让作祟生效要自己 `SetOnHauntFn`。[工厂](../../../raw/current/scripts/standardcomponents.lua#L1176-L1188)
- **未核验**：叶片子实体的 `persists = false` 与引擎父子在客户端侧的清理时序未运行验证；月岛房间的实际落点密度只按散布率与权重推导，未统计实测。

## 验证记录

- 源码审阅：2026-11-08，Steam buildid 26474873（2026-11-09 复检冲岸掉叶门禁与冲岸根容器条件）。核对 `bullkelp_plant.lua` 全文（三套回调、`CheckBeached` 与注释、`OnCollide` 防抖、`fn` 的构造顺序与数值、`underwaterleafsfn`、导出）、`bullkelp_beached.lua` 全文（替换两条路径、名称重定向、燃烧与作祟）、`bullkelp_root.lua` 的 `ondeploy`、`pickable` 的 `SetUp`／`Pick`／`MakeEmpty`／单产物分支、`standardcomponents` 的 `DefaultBurntFn`／三个可燃工厂／`MakeHauntableIgnite`／`MakeInventoryPhysics`／`AddDefaultRippleSymbols`、`constants` 的部署间距表、`simutil` 的 `SpringGrowthMod`、`entityscript` 的显示名回退、`map/forest_map` 的设置翻译、月岛与海面房间、`map/terrain` 白名单、隐士岛与 retrofit 布景。
- 名称核验：`STRINGS.NAMES.BULLKELP_PLANT`＝公牛海带、`BULLKELP_ROOT`＝公牛海带茎；冲岸根不设独立译名，靠 `SetPrefabNameOverride("BULLKELP_PLANT")` 显示为「公牛海带」。[BULLKELP_PLANT](../../../raw/current/scripts/languages/chinese_s.po#L339975-L339977)、[BULLKELP_ROOT](../../../raw/current/scripts/languages/chinese_s.po#L339980-L339982)
- 未覆盖：海带丛被炮弹／三叉戟／投石机击毁的逐源分支归 [[reference/prefabs/bullkelp_root#获取途径]]，本页只写冲岸；月岛地形与 retrofit 的内部流程未展开。
- 游戏运行：未运行。天数、秒数与燃烧时长均为源码推导。
- 复审直修（2026-11-08）：`OceanCoastal` 散布率 0.005→0.01（0.005 实为相邻 `OceanCoastalShore`）、补海带田 `kelp_area` 0.9 与旧档海面补种、floater 端归属、`SetUp` 第三参说明、冲岸根替换的容器门禁；未运行游戏。
- 结构检查：`validate_page.py`（`--wiki-root` 指向 `K/wiki`）通过（standard 档内）；`validate_source_links.py` 0 错 0 警。

## 相关链接

- 产物与武器：[[reference/prefabs/kelp]]、[[reference/prefabs/bullkelp_root]]、[[reference/prefabs/boat-patches]]
- 采摘与世界再生：[[reference/components/pickable]]、[[reference/components/regrowthmanager]]
- 世界生成主题：[[reference/components/retrofitforestmap_anr]]

返回 [[reference/prefabs/index|Prefab 索引]]。
