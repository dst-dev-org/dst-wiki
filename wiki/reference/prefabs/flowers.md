---
title: "花（flower）、邪恶花（flower_evil）、荧光花（flower_cave）与枯萎的花（flower_withered）"
created: "2026-11-08"
updated: "2026-11-09"
type: prefab
summary: "花丛十变体与 1% 玫瑰、采摘即消失与世界再生、作祟转邪恶花、洞穴白天的缺光枯萎；邪恶花的产物与三个来源；荧光花六注册（单／双／三株与三项枯萎版）的产光状态机、产物与再生差；以及同一工厂返回的 flower_rose 与 planted_flower。"
aliases: ["flower", "flower_evil", "flower_cave", "flower_cave_double", "flower_cave_triple", "flower_cave_withered", "flower_cave_double_withered", "flower_cave_triple_withered", "flower_withered", "flower_rose", "planted_flower", "花", "花丛", "邪恶花", "荧光花", "枯萎的花", "枯萎的荧光花", "玫瑰"]
tags: [prefab]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/flower.lua", "prefabs/flower_evil.lua", "prefabs/flower_cave.lua", "prefabs/flower_withered.lua", "prefabs/butterfly.lua", "prefabs/stalker.lua", "prefabs/gravestone.lua", "stategraphs/SGwilson.lua", "components/pickable.lua", "components/roseinspectable.lua", "components/regrowthmanager.lua", "components/worldstate.lua", "standardcomponents.lua", "entityscript.lua", "tuning.lua", "map/forest_map.lua", "map/customize.lua", "map/terrain.lua", "map/rooms/forest/terrain_grass.lua", "map/rooms/cave/bluemush.lua", "map/static_layouts/stagehandgarden.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 花（flower）、邪恶花（flower_evil）、荧光花（flower_cave）与枯萎的花（flower_withered）

## 快速判断

- 职责与适用：四个文件，运行时清单里花族共 11 个注册（`flower` 侧 3、`flower_evil` 1、`flower_cave` 6、`flower_withered` 1）。`prefabs/flower.lua` 产 `flower`（地表花丛）与 `flower_rose`；`flower_evil.lua` 产 `flower_evil`；`flower_cave.lua` 产 `flower_cave`／`flower_cave_double`／`flower_cave_triple` 与三项 `_withered` 变体；`flower_withered.lua` 产枯萎的花。全部是 `pickable` 世界植物，产物各不相同（花瓣／深色花瓣／荧光球／割草／腐坏食物）。[花丛导出](../../../raw/current/scripts/prefabs/flower.lua#L261-L263)、[荧光花六注册](../../../raw/current/scripts/prefabs/flower_cave.lua#L556-L562)、[枯萎的花](../../../raw/current/scripts/prefabs/flower_withered.lua#L58-L58)
- 优先使用：问「采了会怎样、多久再长」看[采摘、消失与再生](#采摘消失与再生)；问「冬天／下雪会不会死」看[冬季、雪与枯萎](#冬季雪与枯萎)；问「怎么变成邪恶花」看[邪恶花的来源与转化](#邪恶花的来源与转化)；问「荧光花怎么亮」看[荧光花六注册与产光](#荧光花六注册与产光)。
- 重要副作用：花丛被采摘即**整株移除**（`remove_when_picked`），重生靠世界再生管理器按位置补种，不是原地长回来；对花丛作祟有一半概率把它换成 `flower_evil`；洞穴里的花丛在「洞穴白天」若照不到日光会被换成 `flower_withered`。[采摘即移除](../../../raw/current/scripts/components/pickable.lua#L577-L579)、[作祟换株](../../../raw/current/scripts/standardcomponents.lua#L1255-L1270)、[缺光枯萎](../../../raw/current/scripts/prefabs/flower.lua#L79-L99)
- 修改边界：花瓣与深色花瓣、`cactus_flower` 的数值与干制见 [[reference/prefabs/petals]]；`lightbulb` 属照明族（本批未写），本页只在荧光花产物处给关系；`charlierose`（查理玫瑰物品）是另一个注册，本页只写「花丛如何产出它」。本页只写花丛本体与四个族的状态。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `flower` | `plainfn` → `commonfn()`：bank／build `flowers`、`SetRayTestOnBB(true)`、`SetDeploySmartRadius(DEPLOYSPACING_RADIUS[LESS] / 2)`、tag `flower`／`cattoy`、`net_bool flower._isrose` → `SetPristine` → 主模拟：inspectable、pickable、可燃烧、再生登记、roseinspectable、洞穴缺光监听、作祟换株、`halloweenmoonmutable`（突变目标 `moonbutterfly_sapling`）、抽变体 | 地表花丛；`POPULATING`（世界生成阶段）为真时跳过抽变体，交给读档时的 `OnLoad`。[公共工厂](../../../raw/current/scripts/prefabs/flower.lua#L138-L206)、[抽变体条件](../../../raw/current/scripts/prefabs/flower.lua#L205-L206) |
| `flower_evil` | `fn`（flower_evil.lua）：bank／build `flowers_evil`、tag 仅 `flower`、`pickupsound="vegetation_grassy"` → `SetPristine` → 主模拟：inspectable、`sanityaura`、pickable、小燃可燃＋小传播、作祟 | 独立注册，不与花丛共用工厂；无世界再生登记。[工厂](../../../raw/current/scripts/prefabs/flower_evil.lua#L35-L88) |
| `flower_cave` | `commonfn("bulb_plant_single", "bulb_plant_single", lightparams_single)`：Light＋LightWatcher、三个 netvar（`_lighttime`／`_lightframe`／`_islighton`）、`SetPrefabNameOverride("flower_cave")` → 主模拟：timer、pickable、lootdropper、中燃可燃＋小传播、万圣节可变、再生登记、作祟 | 洞穴荧光花最小档：光半径 3、产物 `lightbulb`、再生 3 天。[单株](../../../raw/current/scripts/prefabs/flower_cave.lua#L395-L425) |
| `flower_cave_double` | 同 `commonfn`，bank／build `bulb_plant_double`、光半径 4.5 | 产量 2、再生 ×1.5；`scrapbook_proxy="flower_cave"`。[双株](../../../raw/current/scripts/prefabs/flower_cave.lua#L427-L446) |
| `flower_cave_triple` | 同 `commonfn`，bank／build `bulb_plant_triple`、光半径 4.5 | 产量 3、再生 ×2。[三株](../../../raw/current/scripts/prefabs/flower_cave.lua#L448-L467) |
| `flower_cave_withered` | `commonfn(..., is_withered = true)`：build `bulb_plant_single_withered_build`、光色改暗橙、`SetPrefabNameOverride("flower_cave_withered")` | 产物变成 `spoiled_food`、光弱（半径 2、强度 .2）、不进世界再生。[枯萎单株](../../../raw/current/scripts/prefabs/flower_cave.lua#L489-L512) |
| `flower_cave_double_withered`／`flower_cave_triple_withered` | 同上，build 分别加 `_withered_build`；光半径 2.5 | 产物仍 `spoiled_food`，产量 2／3。[枯萎双株](../../../raw/current/scripts/prefabs/flower_cave.lua#L521-L533)、[枯萎三株](../../../raw/current/scripts/prefabs/flower_cave.lua#L542-L554) |
| `flower_withered` | `fn`：bank／build `withered_flowers`、三个动画变体 `wf1`～`wf3`、无 tag → 主模拟：inspectable、pickable、小燃可燃＋小传播 | 唯一产物是 `cutgrass` 的枯萎花。[工厂](../../../raw/current/scripts/prefabs/flower_withered.lua#L17-L56) |

名称：`STRINGS.NAMES.FLOWER`＝花、`FLOWER_EVIL`＝邪恶花、`FLOWER_CAVE`＝荧光花、`FLOWER_CAVE_WITHERED`＝枯萎的荧光花、`FLOWER_WITHERED`＝枯萎的花。`flower_cave_double`／`_triple` 无独立键，靠 `SetPrefabNameOverride` 落到「荧光花」；`flower_cave_*_withered` 中的 double／triple 同理落到「枯萎的荧光花」。[FLOWER](../../../raw/current/scripts/languages/chinese_s.po#L342905-L342907)、[FLOWER_CAVE](../../../raw/current/scripts/languages/chinese_s.po#L342920-L342922)、[FLOWER_CAVE_WITHERED](../../../raw/current/scripts/languages/chinese_s.po#L342925-L342927)、[FLOWER_EVIL](../../../raw/current/scripts/languages/chinese_s.po#L342930-L342932)、[FLOWER_WITHERED](../../../raw/current/scripts/languages/chinese_s.po#L342935-L342937)、[重定向取值](../../../raw/current/scripts/entityscript.lua#L687-L692)

### 花丛的十个变体与 1% 玫瑰

- 变体不是多 build，而是**同一 build 下的 10 个动画**：`names = {"f1"…"f10"}`，`setflowertype` 等概率抽一个并 `PlayAnimation`；另有 `ROSE_CHANCE = 0.01` 抽到 `rose`，抽中时加 `thorny` tag 并把 `net_bool flower._isrose` 置真。[名单与概率](../../../raw/current/scripts/prefabs/flower.lua#L16-L40)
- 存读档：`OnSave` 只存 `anim` 与 `planted`；`OnLoad` 用它们复原（`data.anim` 为空就重新随机），所以变体不会读档走样。[存档](../../../raw/current/scripts/prefabs/flower.lua#L42-L50)
- 客户端的 `_isrose` 只服务图鉴：`OnIsRoseDirty` 把 `scrapbook_proxy` 设成 `flower_rose` 或清空。[图鉴代理](../../../raw/current/scripts/prefabs/flower.lua#L107-L109)
- 检视：`inspectable.getstatus` 只在玫瑰变体返回 `"ROSE"`。[状态](../../../raw/current/scripts/prefabs/flower.lua#L71-L73)

### 采摘、消失与再生

- 接线：`pickable:SetUp("petals", 10)`＋`onpickedfn`＋`remove_when_picked = true`＋`quickpick = true`＋`wildfirestarter = true`。`quickpick` 让玩家用 `doshortaction` 而不是长动作；`wildfirestarter` 让 `Pickable:IsWildfireStarter` 返回真，成为野火系统的候选火源（带 `withered` tag 的植株同样算）。[接线](../../../raw/current/scripts/prefabs/flower.lua#L170-L176)、[判定](../../../raw/current/scripts/components/pickable.lua#L125-L127)、[动作选择](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1133-L1138)
- 产物与附带：单产物分支生成 1 份 `petals`；采摘者若有 `sanity` 且不带 `plantkin`，理智 +5；玫瑰变体再按 `combat`／`bramble_resistant`／`shadowminion` 三道门禁刺 1 点（`ROSE_DAMAGE`）。[采摘回调](../../../raw/current/scripts/prefabs/flower.lua#L52-L69)、[伤害常量](../../../raw/current/scripts/tuning.lua#L765-L766)、[基准血量](../../../raw/current/scripts/tuning.lua#L38-L38)
- 世界事件：每次采摘都推 `plantkilled`（带 `doer` 与 `pos`），注释说明别处也会推同一事件。[事件](../../../raw/current/scripts/prefabs/flower.lua#L68-L68)
- 消失：`Pick` 仍按 `SetUp` 的 10 秒排一次 `regen` 任务，但函数末尾 `remove_when_picked` 直接 `Remove` 整株，任务随实体移除被取消，**不会原地长回来**。[排程](../../../raw/current/scripts/components/pickable.lua#L559-L573)、[移除](../../../raw/current/scripts/components/pickable.lua#L577-L579)
- 再生：`DoTaskInTime(0, CheckForPlanted)` → 非 `planted` 的花丛挂 `AddToRegrowthManager`，被移除时向世界再生管理器登记；`flower` 的条目是 `FLOWER_REGROWTH_TIME`＝30 秒，倍率函数在「正在下雨、夜晚、冬季、地面湿度 ≤1（已干）、有雪」任一成立时返回 0，全不成立才按 1 累积，春季再 ×2。[登记](../../../raw/current/scripts/prefabs/flower.lua#L101-L105)、[条目与倍率](../../../raw/current/scripts/components/regrowthmanager.lua#L154-L159)、[常量](../../../raw/current/scripts/tuning.lua#L3171-L3172)
- 补种落点：再生不是回到原点，而是在记录点周围抖动（半径 20）、要求同地块且 `EXCLUDE_RADIUS` 内无其他实体、`BASE_RADIUS` 内无结构／墙，否则重排。[补种](../../../raw/current/scripts/components/regrowthmanager.lua#L83-L100)

### 冬季、雪与枯萎

- 花丛**没有雪覆盖接线**：`flower.lua` 全文（`flower`／`flower_rose`／`planted_flower` 共用 `commonfn`）未调用 `MakeSnowCovered`／`MakeSnowCoveredPristine`，冬季不会出现积雪外观。
- 花丛也**不挂 `witherable`**，冬季不会枯萎；冬季的实际影响只有两条：世界再生倍率函数返回 0（不再补种），以及花丛本身照常可采。[倍率](../../../raw/current/scripts/components/regrowthmanager.lua#L154-L159)
- 真正的「花 → 枯萎的花」只有一条路径：`TheWorld:HasTag("cave")` 时才注册 `WatchWorldState("iscaveday", OnIsCaveDay)`，洞穴白天开始后 5～10 秒调 `DieInDarkness`——在 30 半径内找同时带 `daylight` 与 `lightsource` 的实体，若其光照计算半径 ×0.7 覆盖不到花丛，就 `Remove` 自己并在原位生成 `flower_withered`。[洞穴监听](../../../raw/current/scripts/prefabs/flower.lua#L199-L201)、[缺光判定](../../../raw/current/scripts/prefabs/flower.lua#L79-L99)、[搜索半径](../../../raw/current/scripts/tuning.lua#L5756-L5756)
- 洞穴相位与地表共用同一套变量：地表世界的 `OnPhaseChanged` 也会调 `OnCavePhaseChanged`，所以 `iscaveday` 在地表就等于白天。[相位](../../../raw/current/scripts/components/worldstate.lua#L63-L80)

### 邪恶花的来源与转化

- 与普通花的差异：独立 build／bank `flowers_evil`、8 个变体（`f1`～`f8`）、产物 `petals_evil`、自带 `sanityaura`＝`-SANITYAURA_SMALL`（100/(30×8)≈0.417/秒）、采摘扣 5 理智（Wendy 的 `wendy_gravestone_1` 技能激活时改为 +5）。[组件](../../../raw/current/scripts/prefabs/flower_evil.lua#L63-L74)、[采摘理智](../../../raw/current/scripts/prefabs/flower_evil.lua#L25-L33)、[光环常量](../../../raw/current/scripts/tuning.lua#L2207-L2207)
- 来源一（作祟）：花丛挂 `MakeHauntableChangePrefab(inst, "flower_evil")`，默认概率 `HAUNT_CHANCE_HALF`＝0.5；命中时在原地生成 `small_puff` 与 `flower_evil`，旧株 `persists=false`、`Hide()` 后下一帧移除。[接线](../../../raw/current/scripts/prefabs/flower.lua#L203-L203)、[换株实现](../../../raw/current/scripts/standardcomponents.lua#L1219-L1270)、[概率常量](../../../raw/current/scripts/tuning.lua#L3051-L3051)
- 来源二（世界生成）：地表世界设置 `flowers` 同时翻译成 `flower` 与 `flower_evil`；邪恶花丛房间 `EvilFlowerPatch` 另有 `flower_evil = 2` 的散布，棋盘与大理石一类布景／布局里也有大量静态摆放。[设置表](../../../raw/current/scripts/map/forest_map.lua#L263-L263)、[房间散布](../../../raw/current/scripts/map/rooms/forest/terrain_grass.lua#L88-L101)
- 来源三（Wendy 墓园技能）：`try_evil_flower` 在非冬季、且半径 3 内的 `flower` tag 实体少于 `WENDYSKILL_GRAVESTONE_EVILFLOWERCOUNT` 时，在坟边 1.5 半径随机角生成 1 株 `flower_evil` 并配 `attune_out_fx`。[坟边生成](../../../raw/current/scripts/prefabs/gravestone.lua#L88-L102)
- 会不会转化或消失：它没有 `AddToRegrowthManager`，摘掉／烧掉后不会世界补种；作祟用的是 `MakeHauntableIgnite`，点火分支被注释，作祟不会把它变成别的株。[燃烧与作祟](../../../raw/current/scripts/prefabs/flower_evil.lua#L78-L81)、[作祟工厂](../../../raw/current/scripts/standardcomponents.lua#L1173-L1189)

### 荧光花六注册与产光

- 六注册共享 `commonfn(bank, build, light_params, is_withered)`：Light＋LightWatcher（阈值 .075／.05）、灯色 `{237,237,209}`（枯萎版 `{201,93,10}`）、三个 netvar 驱动客户端逐帧调光、`SetPrefabNameOverride` 决定显示名。[公共构造](../../../raw/current/scripts/prefabs/flower_cave.lua#L268-L307)
- 产光状态机：状态枚举 `ON`／`CHARGED`／`RECHARGING`，每个状态对应「过渡动画＋循环动画」；点亮时长 `FLOWER_CAVE_LIGHT_TIME`＝90 秒＋4～8 秒过渡＋最多 10 秒随机，充电 `FLOWER_CAVE_RECHARGE_TIME`＝110 秒＋过渡；枯萎版改用 `FLOWER_CAVE_WITHERED_LIGHT_TIME`＝20 秒与 `_RECHARGE_TIME`＝200 秒。[状态与动画](../../../raw/current/scripts/prefabs/flower_cave.lua#L29-L41)、[点亮](../../../raw/current/scripts/prefabs/flower_cave.lua#L146-L164)、[熄灭](../../../raw/current/scripts/prefabs/flower_cave.lua#L133-L144)、[常量](../../../raw/current/scripts/tuning.lua#L1769-L1771)、[枯萎常量](../../../raw/current/scripts/tuning.lua#L8976-L8977)
- 产物：正常三档都是 `lightbulb`，`FLOWER_CAVE_REGROW_TIME`＝3 天；双株 ×1.5 且一次 2 个、三株 ×2 且一次 3 个（第三个参数就是 `numtoharvest`）。**枯萎三档的产物是 `spoiled_food`**，不是荧光球。[单株](../../../raw/current/scripts/prefabs/flower_cave.lua#L419-L419)、[双株](../../../raw/current/scripts/prefabs/flower_cave.lua#L443-L443)、[三株](../../../raw/current/scripts/prefabs/flower_cave.lua#L464-L464)、[枯萎产物](../../../raw/current/scripts/prefabs/flower_cave.lua#L506-L506)
- 采摘：先 `ForceOff` 并停掉 `turnoff`／`recharge` 两块计时器，播 `pickup_lightbulb` 音；正常版播 `picking`→`picked`（`pickable:IsBarren()` 为真、即枯死时改播 `idle_dead`）；枯萎版播 `picked_wilt`、`persists=false`，动画结束即移除。[采摘](../../../raw/current/scripts/prefabs/flower_cave.lua#L197-L212)
- 可燃与作祟：`MakeMediumBurnable`（默认 20 秒）＋`MakeSmallPropagator`＋`MakeHauntableIgnite`（点火分支被注释）。[接线](../../../raw/current/scripts/prefabs/flower_cave.lua#L336-L337)、[作祟](../../../raw/current/scripts/prefabs/flower_cave.lua#L361-L361)
- 再生与形态转换：正常三档进世界再生管理器（`FLOWER_CAVE_REGROWTH_TIME`＝5 天，倍率 1）；非枯萎版还挂 `halloweenmoonmutable`，被月亮事件改写成 `lightflier_flower`；枯萎版两者都没有。[再生登记](../../../raw/current/scripts/prefabs/flower_cave.lua#L339-L345)、[再生条目](../../../raw/current/scripts/components/regrowthmanager.lua#L168-L173)、[再生常量](../../../raw/current/scripts/tuning.lua#L3175-L3176)
- 洞穴生成：`world=cave` 的设置项 `flower_cave` 翻译成三个 prefab，并另有 `flower_cave_regrowth` 开关；地块白名单只允许 FUNGUSMOON 与 MONKEY_DOCK；房间散布如 `bluemush`（0.1／0.05／0.05）。[设置](../../../raw/current/scripts/map/customize.lua#L354-L354)、[设置翻译](../../../raw/current/scripts/map/forest_map.lua#L210-L210)、[再生开关映射](../../../raw/current/scripts/map/customize.lua#L804-L807)、[地块白名单](../../../raw/current/scripts/map/terrain.lua#L124-L124)、[房间](../../../raw/current/scripts/map/rooms/cave/bluemush.lua#L14-L21)

### 枯萎的花的来源

- 产生条件只有一条：上一节[冬季、雪与枯萎](#冬季雪与枯萎)里的 `DieInDarkness`；全 scripts 只有 `flower.lua` 一处 `SpawnPrefab("flower_withered")`。没有「冬天整片枯」的全局逻辑，也不是烧毁产物。
- 表现：3 个动画变体 `wf1`～`wf3`，随机取一个；无任何 tag（因此不会被「植物」类筛选命中）。[变体](../../../raw/current/scripts/prefabs/flower_withered.lua#L24-L37)
- 能否捡起或恢复：它挂 `pickable:SetUp("cutgrass", 10)` 且 `remove_when_picked`／`quickpick`／`wildfirestarter`，摘一次得 1 份 `cutgrass` 后整株消失；没有 `witherable`、没有 `growable`、没有世界再生登记，所以**不能施肥复原、摘掉也不再生**。[采摘](../../../raw/current/scripts/prefabs/flower_withered.lua#L41-L46)
- 与花丛的关系：它是花丛在洞穴缺光时的**终态替换**（旧株移除、新株生成），不是花丛的一个状态；反过来没有任何路径把它变回花丛。

### flower_rose 与 planted_flower（附属入口）

- 两者都走 `commonfn`，随后 `SetPrefabName("flower")`：把 `inst.prefab`、引擎 prefab 名与显示名一并改成 `flower`，所以玩家看到的名字都是「花」，按 prefab 名做判断的代码看到的也是 `flower`，不能靠名字区分这两个注册；同理 `AddPrefabPostInit("flower", ...)` 也会命中它们（见[典型接入](#让花丛采完不整株消失)）。[改名 API](../../../raw/current/scripts/entityscript.lua#L840-L844)
- `flower_rose`：用 `setflowertype(inst, "rose")` 固定成玫瑰变体，另写 `scrapbook_anim="rose"`、`scrapbook_damage=ROSE_DAMAGE`、`scrapbook_proxy="flower_rose"` 与 `DoRoseBounceAnim`；已核验的生成入口是查理（`stalker` 的离场死亡与 `SGstalker` 的死亡帧都留花）、薇诺娜 `winona_charlie_2` 技能在玩家死亡被查理拉回时生成的一次性玫瑰（`death_vinesave_pst` 中 `persists=false`、`NOCLICK`、播 `rose_bounce`）与舞台布景 `stagehandgarden` 的 5 处固定摆放。它不是可制造物品，也没有独立物品形态。[工厂](../../../raw/current/scripts/prefabs/flower.lua#L225-L245)、[查理留花](../../../raw/current/scripts/prefabs/stalker.lua#L1046-L1055)、[技能玫瑰](../../../raw/current/scripts/stategraphs/SGwilson.lua#L4099-L4105)、[布景](../../../raw/current/scripts/map/static_layouts/stagehandgarden.lua#L69-L69)
- `planted_flower`：`planted = true`（`commonfn(true)` 传参，主模拟再补写一次），所以 `CheckForPlanted` 不会把它登记进世界再生管理器，摘掉后不再生；唯一入口是放生蝴蝶——`butterfly` 的 `OnDeploy` 生成 `planted_flower` 并推 `growfrombutterfly`。[工厂](../../../raw/current/scripts/prefabs/flower.lua#L247-L259)、[放生](../../../raw/current/scripts/prefabs/butterfly.lua#L54-L58)
- 花丛本体的玫瑰变体（1%）与 `flower_rose` 注册是两回事：前者只是 `flower` 的一个动画变体＋`thorny` tag，后者是独立注册实体。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：pickable | 花丛／邪恶花／枯萎花用 10 秒再生＋`remove_when_picked`；荧光花用 3 天（×1.5／×2）与 `max_cycles=20` | 主模拟 | [[reference/components/pickable#关键接口]]；[花丛](../../../raw/current/scripts/prefabs/flower.lua#L170-L176)、[荧光花](../../../raw/current/scripts/prefabs/flower_cave.lua#L323-L330) |
| 组件：burnable／propagator | 花丛与邪恶花小燃可燃（默认 10 秒）、枯萎花同；荧光花中燃可燃（20 秒）；全部配小传播 | 主模拟 | [[reference/components/burnable#关键接口]]、[[reference/components/propagator#关键接口]]；[花丛](../../../raw/current/scripts/prefabs/flower.lua#L184-L188)、[荧光花](../../../raw/current/scripts/prefabs/flower_cave.lua#L336-L337) |
| 组件：hauntable | 花丛 `MakeHauntableChangePrefab("flower_evil")`（50% 换株）；邪恶花与荧光花用 `MakeHauntableIgnite`（点火段已注释，等于无效果） | 主模拟 | [[reference/components/hauntable#关键接口]]；[换株](../../../raw/current/scripts/standardcomponents.lua#L1255-L1270)、[无效点火](../../../raw/current/scripts/standardcomponents.lua#L1173-L1189) |
| 组件：sanityaura | 仅邪恶花：`-SANITYAURA_SMALL` | 主模拟 | [[reference/components/sanityaura#关键接口]]；[接线](../../../raw/current/scripts/prefabs/flower_evil.lua#L65-L66) |
| 组件：roseinspectable | 仅花丛：把「残留物生成→改玫瑰、残留物激活→给 `charlierose` 并移除花丛」三件事挂上，能否生成由 `winona_charlie_2` 技能判定 | 主模拟；客户端无 | [[reference/components/roseinspectable]]；[接线](../../../raw/current/scripts/prefabs/flower.lua#L193-L197)、[回调](../../../raw/current/scripts/prefabs/flower.lua#L113-L134) |
| 组件：timer／light／lightwatcher | 仅荧光花：定时点亮／充电与逐帧调光 | 双端有 Light，逻辑在主模拟 | [构造](../../../raw/current/scripts/prefabs/flower_cave.lua#L279-L303) |
| API／数据：世界再生管理器 | 花丛 30 秒、荧光花三档 5 天；邪恶花与枯萎花不登记 | 主模拟 | [[reference/components/regrowthmanager#原版注册与其季节倍率]]；[登记](../../../raw/current/scripts/components/regrowthmanager.lua#L154-L173) |
| API／数据：`WorldState.iscaveday` | 花丛在洞穴世界监听洞穴白天并触发缺光枯萎 | 仅 `TheWorld:HasTag("cave")` | [[reference/components/worldstate#关键接口]]；[监听](../../../raw/current/scripts/prefabs/flower.lua#L199-L201)、[相位](../../../raw/current/scripts/components/worldstate.lua#L63-L70) |
| 关联 Prefab：`petals`／`petals_evil` | 花丛与邪恶花的产物 | 主模拟生成 | [[reference/prefabs/petals#数值]]；[花丛产物](../../../raw/current/scripts/prefabs/flower.lua#L172-L172)、[邪恶花产物](../../../raw/current/scripts/prefabs/flower_evil.lua#L68-L74) |
| 关联 Prefab：`lightbulb` | 荧光花产物，物品侧见 [[reference/prefabs/lightbulb]] | 主模拟生成 | [单株产物](../../../raw/current/scripts/prefabs/flower_cave.lua#L419-L419) |
| 关联 Prefab：`charlierose` | 花丛残留物被激活时塞进背包并移除花丛 | 主模拟；需 `winona_charlie_2` 技能 | 源码入口（Wiki 待建）：[激活](../../../raw/current/scripts/prefabs/flower.lua#L125-L134) |
| 关联 Prefab：`flower_evil`／`flower_withered`／`planted_flower`／`flower_rose` | 花丛的三个转化／附属产物 | 见[邪恶花的来源与转化](#邪恶花的来源与转化)与本页「flower_rose 与 planted_flower」一节 | [换株](../../../raw/current/scripts/prefabs/flower.lua#L203-L203)、[枯萎替换](../../../raw/current/scripts/prefabs/flower.lua#L79-L93) |
| 资源：anim | `flowers.zip`（bank `flowers`）、`flowers_evil.zip`、`bulb_plant_single/double/triple.zip`（枯萎版另加 `_withered_build`）、`withered_flowers.zip` | 静态声明 | [花丛](../../../raw/current/scripts/prefabs/flower.lua#L1-L4)、[荧光花](../../../raw/current/scripts/prefabs/flower_cave.lua#L1-L17)、[枯萎花](../../../raw/current/scripts/prefabs/flower_withered.lua#L1-L4) |

## 典型接入

### 让花丛采完不整株消失

- 前提与选择：服务端 `modmain.lua`。只改 `pickable.remove_when_picked`：置 false 后 `Pick` 不会再移除实体，于是 `SetUp` 时写下的 10 秒再生开始生效；花丛会留在原地，10 秒后自行恢复可采。[移除分支](../../../raw/current/scripts/components/pickable.lua#L577-L579)、[再生排程](../../../raw/current/scripts/components/pickable.lua#L559-L573)
- 生效与撤销：postinit 每次构造运行。`flower_rose` 与 `planted_flower` 的工厂都调用 `inst:SetPrefabName("flower")`，而生成器是在 `fn` 返回后按 `inst.prefab` 取 postinit 表，所以这条 postinit 对 `flower`／`flower_rose`／`planted_flower` **三者都命中**（`flower_rose` 还会再跑一次按自己注册名登记的条目）。`flower_evil`／`flower_cave_*`／`flower_withered` 是别的注册名，不受影响。此刻 `inst.prefab` 已被改成 `flower`，服务端 postinit 要区分变体可读工厂写下的字段：玫瑰（含花丛 1% 抽中的变体）是 `inst.animname == "rose"`，放生蝴蝶种下的是 `inst.planted == true`（后者只在主模拟写；世界生成阶段尚未抽变体，`animname` 可能为空）。停用模组即恢复。[改名](../../../raw/current/scripts/entityscript.lua#L840-L844)、[生成器按 inst.prefab 取表](../../../raw/current/scripts/mainfunctions.lua#L387-L407)、[两工厂](../../../raw/current/scripts/prefabs/flower.lua#L225-L259)
- 原版依据：原版为什么「采完就没」见[采摘、消失与再生](#采摘消失与再生)。

```lua
-- modmain.lua：地表花丛改为原地 10 秒再生，不再整株消失
AddPrefabPostInit("flower", function(inst)
    if not GLOBAL.TheWorld.ismastersim or inst.components.pickable == nil then
        return
    end
    inst.components.pickable.remove_when_picked = false
end)
```

- 调整与验证：单位为秒；`flower` 没有 `onregenfn`／`makeemptyfn`，所以采空期间仍是满花外观、只是不可采，想有区分要自己补动画回调；春季该时长再乘 0.75。示例不改产物与理智收益。未运行游戏验证。

## 常见陷阱与限制

- **误区：花丛有雪覆盖、冬天会枯**：本构建的花丛没有 `MakeSnowCovered`、也没有 `witherable`，冬季既不积雪也不枯萎，只是世界再生停止；「枯萎的花」只从洞穴缺光路径产生。[再生倍率](../../../raw/current/scripts/components/regrowthmanager.lua#L154-L159)、[缺光转换](../../../raw/current/scripts/prefabs/flower.lua#L79-L93)
- **误区：`SetUp("petals", 10)` 的 10 秒是花丛的再生周期**：`remove_when_picked` 让整株在采摘时就被移除，这个计时器随即作废；真实回归依赖世界再生管理器的 30 秒累积（且要满足白天、非冬季、不下雨而地面仍湿，即 `wetness > 1`）。[移除](../../../raw/current/scripts/components/pickable.lua#L577-L579)、[倍率](../../../raw/current/scripts/components/regrowthmanager.lua#L154-L159)
- **边界：邪恶花不会自己回来**：它不登记世界再生，摘掉或烧掉就少一株；唯一的「新增」途径是花丛被作祟换株、世界生成布点与 Wendy 墓园技能。[组件集](../../../raw/current/scripts/prefabs/flower_evil.lua#L63-L81)
- **边界：荧光花的灯与采摘状态互锁**：点亮途中被采摘会 `ForceOff` 并停表；枯萎版被采后 `persists=false` 且播完 `picked_wilt` 就移除，所以枯萎荧光花是消耗品而不是可循环资源。[采摘](../../../raw/current/scripts/prefabs/flower_cave.lua#L197-L212)
- **边界：荧光花的 `max_cycles=20` 在原生内容里不会耗尽**：`Pick` 只在 `transplanted` 为真时扣 `cycles_left`，而荧光花没有移植物品形态，所以它不会变 `barren`。[扣减条件](../../../raw/current/scripts/components/pickable.lua#L537-L539)、[接线](../../../raw/current/scripts/prefabs/flower_cave.lua#L329-L330)
- **边界：`flower_rose`／`planted_flower` 的 prefab 名会被改掉**：它们构造时 `SetPrefabName("flower")`，之后按 `inst.prefab` 判定的逻辑看到的是 `flower`；模组若想区分这两个形态，要读 `inst.planted` 或 `DoRoseBounceAnim` 之类的标记，不能靠 prefab 名。[改名](../../../raw/current/scripts/entityscript.lua#L840-L844)
- **未核验**：`flower` 的 `cattoy` tag、`halloweenmoonmutable` 的月亮改写、荧光花的逐帧调光曲线均只做代码审阅，未运行验证；洞穴白天与地表相位的实际耦合只在 `OnPhaseChanged` 里核到，跨分片表现未测。

## 验证记录

- 源码审阅：2026-11-08，Steam buildid 26474873（2026-11-09 复检 postinit 影响范围与花族十一注册）。核对 `flower.lua` 全文（变体抽签、存读档、采摘回调、缺光枯萎、再生登记、玫瑰残留物、作祟换株、三个导出工厂）、`flower_evil.lua` 全文、`flower_cave.lua` 的公共构造／光状态机／六注册差异／采摘与再生、`flower_withered.lua` 全文、`butterfly.lua` 的放生入口、`stalker.lua` 与 `SGwilson.lua` 的玫瑰生成、`gravestone.lua` 的坟边邪恶花、`pickable` 的 `SetUp`／`Pick`／`Regen`／`MakeEmpty`、`regrowthmanager` 的花与荧光花条目、`worldstate` 的相位变量、`standardcomponents` 的换株／作祟／燃烧工厂、`entityscript` 的改名与显示名回退、`tuning` 与 `map` 的相关常量与布点。
- 名称核验：`FLOWER`＝花、`FLOWER_EVIL`＝邪恶花、`FLOWER_CAVE`＝荧光花、`FLOWER_CAVE_WITHERED`＝枯萎的荧光花、`FLOWER_WITHERED`＝枯萎的花；`FLOWER_CAVE_DOUBLE`／`_TRIPLE`／`FLOWER_ROSE`／`PLANTED_FLOWER` 在简中 po 无条目，靠 `SetPrefabNameOverride`／`SetPrefabName` 借用其他键。[名称证据](../../../raw/current/scripts/languages/chinese_s.po#L342905-L342937)
- 未覆盖：`petals`／`petals_evil`／`lightbulb`／`charlierose` 的数值与去向；`roseinspectable` 与 `charlierose` 的完整技能链；`halloweenmoonmutable` 的触发条件。
- 游戏运行：未运行。再生秒数、光时长与概率均为源码推导。
- 结构检查：见交付回报。

## 相关链接

- 产物：[[reference/prefabs/petals]]、[[reference/prefabs/tropical-gathered-fruits]]
- 采摘与再生：[[reference/components/pickable]]、[[reference/components/regrowthmanager]]、[[reference/components/worldstate]]
- 相邻主题：[[reference/prefabs/kelp]]（另一种「采后消失＋世界再生」的植物）、[[reference/prefabs/mushrooms]]

返回 [[reference/prefabs/index|Prefab 索引]]。
