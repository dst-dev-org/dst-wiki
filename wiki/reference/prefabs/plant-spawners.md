---
title: "可放置的植物与巢卵（lureplantbulb、spidereggsack）"
created: "2026-11-10"
updated: "2026-11-10"
type: prefab
summary: "两件放下就生成世界实体的物品：食人花种子 lureplantbulb（文件 lureplant_bulb.lua，注册名无下划线）放下生成食人花 lureplant，先在 spawn 状态播 grow（退出该状态时推 freshspawn）再进 hibernate，960 秒后才出芽；蜘蛛卵 spidereggsack 放下生成 1 级蜘蛛巢 spiderden，巢内立刻满 3 只、首个放员不等周期（计时置零），之后每 5±0.5 秒一次尝试（还要过开关／库存／离屏排队等门禁）。两者都不设 SetDeployMode，落在 DEPLOYMODE.DEFAULT（一般落点条件：可通行＋空位＋IsDeployable），不查 GROUND_HARD；堆叠 10、燃料 180 秒、可燃 30 秒，来源分别是杀死食人花／沃姆伍德技能配方与蜘蛛配方、3 级蜘蛛巢、蜘蛛女王与韦伯初始物品。"
aliases: ["lureplantbulb", "lureplantbulb_placer", "spidereggsack", "spidereggsack_placer", "lureplant", "spiderden", "食人花种子", "蜘蛛卵", "食人花", "蜘蛛巢", "眼球草"]
tags: [prefab, item]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/lureplant_bulb.lua", "prefabs/lureplant.lua", "stategraphs/SGlureplant.lua", "prefabs/spidereggsack.lua", "prefabs/spiderden.lua", "prefabs/spiderqueen.lua", "prefabs/skilltree_wormwood.lua", "prefabs/hermitcrab.lua", "components/deployable.lua", "components/map.lua", "components/playercontroller.lua", "components/childspawner.lua", "components/tradable.lua", "brains/rabbitbrain.lua", "brains/carratbrain.lua", "standardcomponents.lua", "prefabutil.lua", "recipe.lua", "recipes.lua", "components/builder_replica.lua", "prefabs/pigking.lua", "constants.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 可放置的植物与巢卵（lureplantbulb、spidereggsack）

## 快速判断

- 职责与适用：两件「放下就生成世界实体」的物品。`lureplantbulb`（食人花种子，文件 `prefabs/lureplant_bulb.lua`，注册名**不带下划线**）放下生成 `lureplant`（食人花）；`spidereggsack`（蜘蛛卵）放下生成 1 级 `spiderden`（蜘蛛巢）。两件都没有调 `SetDeployMode`，落在 `DEPLOYMODE.DEFAULT`：落点要可通行（陆地或船面）、空位够、并过 `IsDeployable` 前置，但**不查** `GROUND_HARD`；共用规则（`IsDeployable`、禁建带、mouseover 阶段）见 [[reference/prefabs/tree-seeds#种植门禁与落点]]。[球茎注册](../../../raw/current/scripts/prefabs/lureplant_bulb.lua#L58-L67)、[蛛卵注册](../../../raw/current/scripts/prefabs/spidereggsack.lua#L71-L85)、[默认模式](../../../raw/current/scripts/components/deployable.lua#L31-L36)
- 优先使用：改放下的产物看[放置门禁与产物](#放置门禁与产物)；看放下之后怎么长／怎么放蜘蛛看[产出物的后续](#产出物的后续)；补来源看[获得路径](#获得路径)。
- 重要副作用：两者放下时都调 `PreventCharacterCollisionsWithPlacedObjects`——先清掉与角色的碰撞，再由每 0.5 秒一次的任务按**最近角色的距离**把碰撞胶囊收缩到该距离；附近没有角色后恢复满半径并自行停掉任务（不是按玩家数量重算）。食人花放下进入 `spawn`（播 `grow`，**退出该状态时**推 `freshspawn`）→ `hibernate`，`FreshSpawn` 排 `LUREPLANT_HIBERNATE_TIME`＝960 秒后才 `emerge` 并启动 `minionspawner`；蜘蛛巢放下时巢内已有 3 只，首次放员不等周期（`StartSpawning` 把计时置零），之后每 5±0.5 秒一次尝试（需开关开启、巢内有库存、可离屏或清醒生成，否则排队）。[碰撞](../../../raw/current/scripts/standardcomponents.lua#L1694-L1726)、[球茎产物](../../../raw/current/scripts/prefabs/lureplant_bulb.lua#L9-L18)、[状态](../../../raw/current/scripts/stategraphs/SGlureplant.lua#L291-L318)、[FreshSpawn](../../../raw/current/scripts/prefabs/lureplant.lua#L137-L150)、[巢容量](../../../raw/current/scripts/prefabs/spiderden.lua#L142-L149)
- 修改边界：食人花与蜘蛛巢本体的战斗、战利品表、女王转化、眼球草行为只作关系说明；被它们生成／消耗的物品另归各页。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `lureplantbulb` | `fn`：`Transform`／`AnimState`／`SoundEmitter`／`Network` → `MakeInventoryPhysics` → bank／build `eyeplant_bulb`、播 `idle` → `SetPristine` → 主模拟 | 显示名 食人花种子；`pickupsound="vegetation_firm"`；**无** `cattoy`、**无** `tradable`；`MakeInventoryFloatable("small",0.05,0.95)`。[构造](../../../raw/current/scripts/prefabs/lureplant_bulb.lua#L20-L64) |
| `lureplantbulb_placer` | `MakePlacer("lureplantbulb_placer","eyeplant_trap","eyeplant_trap","idle_hidden")` | 预览用食人花的 bank／build，播藏起来的 `idle_hidden`；placer 带 `CLASSIFIED`／`NOCLICK`／`placer` 标签、`persists=false`。[注册](../../../raw/current/scripts/prefabs/lureplant_bulb.lua#L66-L67)、[MakePlacer](../../../raw/current/scripts/prefabutil.lua#L1-L19) |
| `spidereggsack` | `fn`：同骨架，bank／build `spider_egg_sac`、播 `idle`；tag `cattoy`；`scrapbook_specialinfo="PLANTABLE_ON"` | 显示名 蜘蛛卵；`MakeInventoryFloatable("med",0.05,{0.85,0.6,0.85})`；有 `tradable`（`goldvalue` 0）。[构造](../../../raw/current/scripts/prefabs/spidereggsack.lua#L28-L77) |
| `spidereggsack_placer` | `MakePlacer("spidereggsack_placer","spider_cocoon","spider_cocoon","cocoon_small",…,placer_postinit)` | `placer_postinit` 里 `HideSymbol("bedazzled_flare")`，预览不显示闪光装饰。[注册](../../../raw/current/scripts/prefabs/spidereggsack.lua#L79-L85) |

### 共同段与差异

| 项 | `lureplantbulb` | `spidereggsack` | 证据 |
|---|---|---|---|
| 堆叠 | `STACK_SIZE_LARGEITEM`＝10 | 同 | [球茎](../../../raw/current/scripts/prefabs/lureplant_bulb.lua#L44-L45)、[蛛卵](../../../raw/current/scripts/prefabs/spidereggsack.lua#L54-L55) |
| 燃料／可燃 | `LARGE_FUEL`＝180 秒；`MakeSmallBurnable(TUNING.LARGE_BURNTIME)`＝30 秒＋小号蔓延 | 同 | [球茎](../../../raw/current/scripts/prefabs/lureplant_bulb.lua#L48-L52)、[蛛卵](../../../raw/current/scripts/prefabs/spidereggsack.lua#L58-L62) |
| 交易 | 没有 `tradable` | 有 `tradable`，但 `goldvalue` 为 0→猪王不收 | [猪王](../../../raw/current/scripts/prefabs/pigking.lua#L572-L580) |
| 其它组件 | `inspectable`／`inventoryitem`／`MakeHauntableLaunchAndIgnite` | 另有 `SetOnPickupFn`（捡起放 `spider_egg_sack` 音效） | [球茎](../../../raw/current/scripts/prefabs/lureplant_bulb.lua#L46-L56)、[蛛卵](../../../raw/current/scripts/prefabs/spidereggsack.lua#L65-L69) |
| 部署模式 | 不设 → `DEPLOYMODE.DEFAULT` | 同 | [注释掉的 ANYWHERE](../../../raw/current/scripts/prefabs/lureplant_bulb.lua#L58-L61)、[蛛卵同](../../../raw/current/scripts/prefabs/spidereggsack.lua#L71-L74) |

### 名称核验

`NAMES.LUREPLANTBULB`＝食人花种子、`NAMES.LUREPLANT`＝食人花、`NAMES.SPIDEREGGSACK`＝蜘蛛卵、`NAMES.SPIDERDEN`＝蜘蛛巢（`spiderden_2`／`spiderden_3` 没有独立键，都靠 `SetPrefabName("spiderden")`）。两个 placer 无 `NAMES` 键。[食人花种子](../../../raw/current/scripts/languages/chinese_s.po#L344940-L344942)、[食人花](../../../raw/current/scripts/languages/chinese_s.po#L344935-L344937)、[蜘蛛卵](../../../raw/current/scripts/languages/chinese_s.po#L349835-L349837)、[蜘蛛巢](../../../raw/current/scripts/languages/chinese_s.po#L349825-L349827)、[改名](../../../raw/current/scripts/prefabs/spiderden.lua#L718-L718)

## 放置门禁与产物

### DEFAULT 分支的门禁

`DEPLOYMODE.DEFAULT` → `Map:CanDeployAtPoint(pt, inst, mouseover)`，三个条件同时成立：[分支](../../../raw/current/scripts/components/deployable.lua#L116-L117)、[实现](../../../raw/current/scripts/components/map.lua#L316-L321)

1. `mouseover == nil` 或鼠标指向的实体带 `player`／`walkableplatform`／`walkableperipheral` 之一；
2. `IsPassableAtPointWithPlatformRadiusBias(..., allow_water=false, exclude_boats=false, NO_BUILD_BORDER_RADIUS, ignore_land_overhang=true)`：陆地地块，或在可走平台（船）上；
3. `IsDeployPointClear`：按 `DEPLOYSPACING_RADIUS[DEPLOYSPACING.DEFAULT]`＝2 检查周围空位（`DEPLOYSPACING.DEFAULT` 的值是 0，2 出自半径表）。

这三条之外还有两处与 PLANT 共用的前置，完整证据与反例见 [[reference/prefabs/tree-seeds#种植门禁与落点]]：`CanDeploy` 先过 `IsDeployable`（骑乘或手持浮标时普通放置件过不去）；`IsDeployPointClear` 进入后先拒废铁战车竞技场的禁建带，再扫实体。

第 1 条的 `mouseover` 只在**预览阶段**拿得到：服务端 `ACTIONS.DEPLOY` 与 `Deploy` 内部复查都传 `nil`，客户端 placer 的 `testfn` 才把鼠标下的实体传给副本——所以「鼠标指着谁」不会成为服务端的放行依据。该参数同样参与 WATER／CUSTOM 分支，只是本页两件走 DEFAULT。

因此它们**不查 `GROUND_HARD`**，道路上也能放；但落点必须可通行（水里放不了，除非站在船上）并过上述前置。[可通行判定](../../../raw/current/scripts/components/map.lua#L44-L67)、[间距常量](../../../raw/current/scripts/constants.lua#L2236-L2263)、[IsDeployable](../../../raw/current/scripts/components/deployable.lua#L89-L102)、[禁建带](../../../raw/current/scripts/components/map.lua#L263-L266)、[预览传 mouseover](../../../raw/current/scripts/components/playercontroller.lua#L2906-L2911)

### 产物

| 注册 | `ondeploy` 实际动作 | 证据 |
|---|---|---|
| `lureplantbulb` | `SpawnPrefab("lureplant", inst.linked_skinname, inst.skin_id)`（皮肤随球茎传下去）→ 定位 → `stackable:Get():Remove()` 扣 1 → `PreventCharacterCollisionsWithPlacedObjects` → `lp.sg:GoToState("spawn")` → `AddTag("planted")` | [ondeploy](../../../raw/current/scripts/prefabs/lureplant_bulb.lua#L9-L18) |
| `spidereggsack` | 播 `spider_egg_sack` 音效 → `SpawnPrefab("spiderden")` → 定位 → `PreventCharacterCollisionsWithPlacedObjects` → 扣 1 | [ondeploy](../../../raw/current/scripts/prefabs/spidereggsack.lua#L14-L22) |

- 两个 `ondeploy` 都不读 `deployer`；扣件用 `stackable:Get():Remove()`，所以整叠只少一件。[[reference/prefabs/tree-seeds#种植门禁与落点]]
- `planted` 标签会存读档（`data.planted`）。它的实际消费端是寄居蟹：世界事件 `CHEVO_lureplantdied` 只在目标带 `planted` 时按「玩家种下的食人花」处理并说 `HERMITCRAB_PLANTED_LUREPLANT_DIED`，否则走普通检查。兔子／胡萝卜鼠的觅食判定虽然也排除 `planted`，但**还要求目标有 `bait` 组件**，食人花有 `planted` 但没有 `bait`，单是 `bait` 这一条就不成立，所以那层过滤对它不起作用。[读档](../../../raw/current/scripts/prefabs/lureplant.lua#L215-L227)、[寄居蟹](../../../raw/current/scripts/prefabs/hermitcrab.lua#L1382-L1399)、[兔子](../../../raw/current/scripts/brains/rabbitbrain.lua#L31-L37)、[胡萝卜鼠](../../../raw/current/scripts/brains/carratbrain.lua#L84-L89)

## 产出物的后续

### 食人花（lureplant）

1. `spawn` 状态：播 `grow`，带 `busy`／`hiding`／`noelectrocute`；动画结束转 `hibernate`，退出时推 `freshspawn`。[状态](../../../raw/current/scripts/stategraphs/SGlureplant.lua#L291-L318)
2. `FreshSpawn` 把 `task` 取消、停 `minionspawner`、清空眼球草，然后排 `LUREPLANT_HIBERNATE_TIME`＝`total_day_time`×2＝960 秒的 `WakeUp`。[FreshSpawn](../../../raw/current/scripts/prefabs/lureplant.lua#L137-L150)、[常量](../../../raw/current/scripts/tuning.lua#L1204-L1207)
3. `WakeUp` 在冬天只会再排一次 960 秒（继续睡）；否则开 `minionspawner`、进屋 `emerge`，并开始找饵（`showbait`）。[WakeUp](../../../raw/current/scripts/prefabs/lureplant.lua#L75-L88)、[冬季](../../../raw/current/scripts/prefabs/lureplant.lua#L255-L265)
4. 本体另有 `shelf`／`inventory`／`eater`（只吃 MEAT）／`digester`、300 生命、`MakeLargeBurnable`＋30 秒×`PLANTMOB_BURNTIME_MULT`。[构造](../../../raw/current/scripts/prefabs/lureplant.lua#L364-L417)

### 蜘蛛巢（spiderden）

- 放下的是 `MakeSpiderDenFn(1)`：`growable:SetStage(1)`＋`StartGrowing`，`SetStage` 把 `childspawner` 上限写成 `SPIDERDEN_SPIDERS[1]`＝3、生命写成 `SPIDERDEN_HEALTH[1]`＝250、应急半径 `SPIDERDEN_EMERGENCY_RADIUS[1]`＝10；`SetMaxChildren` 会把 `childreninside` 补到上限，所以巢里立刻是满的。[注册](../../../raw/current/scripts/prefabs/spiderden.lua#L838-L840)、[SetStage](../../../raw/current/scripts/prefabs/spiderden.lua#L142-L149)、[补满](../../../raw/current/scripts/components/childspawner.lua#L315-L321)
- 放员与补员是**两条独立的外部计时**（spiderden 调 `WorldSettings_ChildSpawner_SpawnPeriod`／`RegenPeriod`，把 `useexternaltimer` 置真），周期 5±0.5／90±9 秒（方差由 `Set*Period` 取 `period*0.1`，覆盖组件构造里的 2）：
  - **放员**：到点要求 `spawning`、未排队、`childreninside > 0`，再查 `CanSpawnOffscreenOrAwake()`（`spawnoffscreen or not IsAsleep`；不满足就 `QueueSpawnChild`，睡眠时积压、醒来补一次）。`StartSpawning` 会把首个外部计时置零，所以**首轮放员不等待 5±0.5 秒**；`OnFreeze` 与 `OnIsCaveDay` 只调 `StopSpawning`。
  - **补员**：到点要求 `regening` 且未达普通／应急容量上限，**与巢内库存无关——空巢也会补**。
  - **世界开关**：`SPIDERDEN_ENABLED` 写进 `worldsettingstimer` 的 `enabled`；为假时 `IsPaused` 直接为真、`ResumeTimer` 也不装任务，两条外部计时都不推进。另注意构造里 `childreninside = 0` 的赋值早于 `SetStage`，后者会按容量重新补满库存——不能据那次清零推断「开关关掉后巢内为空」。[接线](../../../raw/current/scripts/prefabs/spiderden.lua#L736-L752)、[常量](../../../raw/current/scripts/tuning.lua#L1108-L1119)
- 容量与生命：`SetStage` 写的是 `childspawner:SetMaxChildren(math.floor(SpringCombatMod(SPIDERDEN_SPIDERS[stage])))` 与 `health:SetMaxHealth(SPIDERDEN_HEALTH[stage])`。3 级默认上限 9；若那次 `SetStage` 发生在春季（构造或晋级当刻），上限是 `floor(9 × 1.33)`＝11。换季本身不会回头改已写好的上限，生命也不吃春季加成，仍是 1000。
- 三阶生长：1→2 级用 `SPIDERDEN_GROW_TIME[1]`×（1＋随机）＝2400–4800 秒，2→3 级同量级（这两段春季各为 1800–3600 秒）；3→女王改用 `SPIDERDEN_GROW_TIME_QUEEN`（不是 `GROW_TIME[3]`）×（1＋随机）＝6000–12000 秒，春季 4500–9000 秒。
- 女王门禁：到点后先过 `TUNING.SPAWN_SPIDERQUEEN`（关则回落 3 级）、`bedazzled`、睡袋正在使用、`IsNearPlayer(30)`（不在附近则改回 3 级并 60–120 秒后重试）；最后用半径 `QUEEN_RANGE_CHECK`＝60 内「巢＋女王」数量对 `QUEEN_CAP`＝4：**不足 4** 时 `should_duplicate=true`，女王作为额外个体生成、旧巢保留（此前已被 `SetStage(1)` 压回 1 级，之后继续长）；**达到或超过 4** 时仍生成女王，但旧巢被删除。[阶段表](../../../raw/current/scripts/prefabs/spiderden.lua#L568-L574)、[取时长](../../../raw/current/scripts/prefabs/spiderden.lua#L516-L524)、[常量](../../../raw/current/scripts/tuning.lua#L1108-L1112)
- 放在船上（`GetCurrentPlatform() ~= nil`）会 `StopGrowing`＋`StopRegen`＋地皮半径归零；这段在 `OnLoadPostPass` 里，新放置的巢也会在 0 秒后跑一次。[平台分支](../../../raw/current/scripts/prefabs/spiderden.lua#L608-L626)、[新放置](../../../raw/current/scripts/prefabs/spiderden.lua#L830-L832)

## 获得路径

| 注册 | 已核验来源 | 证据 |
|---|---|---|
| `lureplantbulb` | ①杀死食人花：它的 `lootdropper` 只有 `{"lureplantbulb"}`；②沃姆伍德技能配方 `wormwood_lureplant`：25 点生命＋2 个堆肥包＋5 个植物肉，`builder_skill="wormwood_lureplantbulbcrafting"`，`sg_state="form_bulb"`、`actionstr="GROW"` | ①[战利品](../../../raw/current/scripts/prefabs/lureplant.lua#L382-L389)、②[配方](../../../raw/current/scripts/recipes.lua#L348-L348)、[技能节点](../../../raw/current/scripts/prefabs/skilltree_wormwood.lua#L92-L100) |
| `spidereggsack` | ①配方：12 丝＋4 蜘蛛腺＋3 莎草纸，`TECH.NONE`，`builder_tag="spiderwhisperer"`（仅韦伯）；②3 级蜘蛛巢战利品表含 1 个；③蜘蛛女王战利品表含 1 个；④韦伯初始物品 | ①[配方](../../../raw/current/scripts/recipes.lua#L324-L324)、②[战利品表](../../../raw/current/scripts/prefabs/spiderden.lua#L61-L66)、③[女王表](../../../raw/current/scripts/prefabs/spiderqueen.lua#L22-L34)、④[初始物品](../../../raw/current/scripts/tuning.lua#L3448-L3462) |

- 两个配方都靠 `builder_tag`／`builder_skill` 限制人物；`builder_tag` 在 `builder_replica` 里按 `HasTag` 校验，`builder_skill` 按 `skilltreeupdater:IsActivated` 校验。[校验](../../../raw/current/scripts/components/builder_replica.lua#L305-L316)、[配方字段](../../../raw/current/scripts/recipe.lua#L136-L140)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`deployable` | 提供 `CanDeploy`／`ondeploy`；不设模式与间距 | 仅主模拟持有；模式 `DEPLOYMODE.DEFAULT`；间距档 `DEPLOYSPACING.DEFAULT`（半径取 `DEPLOYSPACING_RADIUS[DEFAULT]`＝2） | [[reference/components/deployable#关键接口]]；[构造](../../../raw/current/scripts/components/deployable.lua#L31-L53) |
| 组件：`stackable` | 上限 10，部署时拆 1 件 | 主模拟与 replica | [[reference/components/stackable#关键接口]]；[接线](../../../raw/current/scripts/prefabs/lureplant_bulb.lua#L44-L45) |
| 组件：`fuel`／`burnable` | 180 秒燃料（`LARGE_FUEL`）；30 秒小号可燃＋蔓延；烧尽走 `DefaultBurntFn` | 主模拟 | [[reference/components/fuel]]、[[reference/components/burnable#关键接口]]；[helper](../../../raw/current/scripts/standardcomponents.lua#L164-L198) |
| 组件：`childspawner`（产物侧） | 蜘蛛巢用它放／补蜘蛛 | 主模拟；平台上被 `StopRegen` | [[reference/components/childspawner#关键接口]]；[接线](../../../raw/current/scripts/prefabs/spiderden.lua#L736-L754) |
| 组件：`growable`（产物侧） | 食人花不用；蜘蛛巢三阶生长 | 主模拟 | [[reference/components/growable#关键接口]]；[阶段](../../../raw/current/scripts/prefabs/spiderden.lua#L568-L574) |
| SG／Brain：`SGlureplant` | `spawn`／`hibernate`／`idleout`／`showbait` 等状态；食人花行为 | 主模拟 | [spawn 状态](../../../raw/current/scripts/stategraphs/SGlureplant.lua#L291-L318)（Wiki 待建） |
| 关联 Prefab：`lureplant` | 球茎的产物；带 `planted` 与 `plantcreature`／`nolife` | 主模拟 | [构造](../../../raw/current/scripts/prefabs/lureplant.lua#L343-L350)（Wiki 待建） |
| 关联 Prefab：`spiderden`／`spiderden_2`／`spiderden_3` | 蛛卵的产物（恒为 1 级）；另有 2／3 级注册 | 主模拟；同一 `MakeSpiderDenFn` | [注册](../../../raw/current/scripts/prefabs/spiderden.lua#L838-L840)（Wiki 待建） |
| 关联 Prefab：`eyeplant`／`plantmeat` | 食人花的眼球草与诱饵产物 | 主模拟 | [assets／prefabs](../../../raw/current/scripts/prefabs/lureplant.lua#L9-L14)（Wiki 待建） |
| 资源：`anim/eyeplant_bulb.zip`／`anim/eyeplant_trap.zip`／`anim/spider_egg_sac.zip` | 物品、产物与 placer 共用动画包 | 原版资源，不随页分发 | [球茎 assets](../../../raw/current/scripts/prefabs/lureplant_bulb.lua#L3-L7)、[蛛卵 assets](../../../raw/current/scripts/prefabs/spidereggsack.lua#L3-L7) |

## 典型接入

### 让蜘蛛卵放下时直接生成 3 级巢

- 前提与选择：服务端 `modmain.lua` 的 prefab postinit，替换 `deployable.ondeploy`。选这个入口是因为产物名写死在 `ondeploy` 里，改 `spiderden` 的 postinit 只会影响已生成的巢。
- 生效与撤销：只影响之后放下的蜘蛛卵；三个等级共用 `MakeSpiderDenFn`，所以产物是 3 级档：生命 1000、生长起点 3 级，容量按放巢当刻的季节算（非春季 9、春季 11）。换回默认只需移除该 postinit。
- 原版依据：原版 `ondeploy` 就是「`SpawnPrefab("spiderden")`→定位→`PreventCharacterCollisionsWithPlacedObjects`→扣 1 件」四步；3 级注册是 `Prefab("spiderden_3", MakeSpiderDenFn(3), …)`。[原版](../../../raw/current/scripts/prefabs/spidereggsack.lua#L14-L22)、[注册](../../../raw/current/scripts/prefabs/spiderden.lua#L838-L840)

```lua
-- modmain.lua：蜘蛛卵放下后直接生成 3 级蜘蛛巢
AddPrefabPostInit("spidereggsack", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    local deployable = inst.components.deployable
    if deployable == nil then
        return
    end
    deployable.ondeploy = function(item, pt)
        local den = GLOBAL.SpawnPrefab("spiderden_3")
        if den ~= nil then
            den.Transform:SetPosition(pt:Get())
            GLOBAL.PreventCharacterCollisionsWithPlacedObjects(den)
            item.components.stackable:Get():Remove()
        end
    end
end)
```

- 调整与验证：验证看放下的巢是否立刻显示 3 级外观、生命 1000（`SPIDERDEN_HEALTH[3]`），并**按季节分开看容量**：非春季是 `SPIDERDEN_SPIDERS[3]`＝9，春季（`SpringCombatMod`）是 `floor(9 × 1.33)`＝11。不要在船上验证，平台分支会停在生长与补员。

## 常见陷阱与限制

- **误区：食人花种子放下就能吃人。** 它先播 `grow`，转 `hibernate` 后再等 960 秒（`total_day_time`×2），冬季还会继续顺延；期间不生成眼球草。[FreshSpawn](../../../raw/current/scripts/prefabs/lureplant.lua#L137-L150)、[常量](../../../raw/current/scripts/tuning.lua#L1204-L1207)
- **误区：蜘蛛卵放下得到的是「升级过的巢」。** 产物写死 `spawn("spiderden")`，恒为 1 级；升级只能靠巢自己的生长或升级物品。[产物](../../../raw/current/scripts/prefabs/spidereggsack.lua#L14-L22)
- **边界：可以在硬地上放，但不是「只要空位」。** 两件走 DEFAULT，不查 `GROUND_HARD`（道路、岩地都行），但仍要落点可通行、过 `IsDeployable`、且不在废铁战车竞技场禁建带内；骑乘或手持浮标时也放不下。[门禁](../../../raw/current/scripts/components/map.lua#L316-L321)
- **边界：烧尽只出灰，不返还物品。** 两件都用默认 `onburnt` → `DefaultBurntFn`：非海上生成 `ash`、堆叠取 `min(ash 上限, 原堆叠)`，海上不产灰。[烧尽](../../../raw/current/scripts/standardcomponents.lua#L18-L44)
- **边界：作祟不点燃。** 两件都用 `MakeHauntableLaunchAndIgnite`，点燃分支带 `#HAUNTFIX` 注释，当前只弹飞。[弹飞](../../../raw/current/scripts/standardcomponents.lua#L1204-L1215)
- **边界：蜘蛛巢在船上只停生长与补员。** `GetCurrentPlatform() ~= nil` 时 `OnLoadPostPass` 停生长、`StopRegen`（不再补员）、地皮半径归零，但**不额外停**放员：巢内已有的蜘蛛仍可能被放出，也仍受世界开关、冻结、睡眠排队与 investigate／紧急放员等事件影响——不是「必定按周期逐只放完」。[平台](../../../raw/current/scripts/prefabs/spiderden.lua#L608-L626)
- **边界：蜘蛛卵能交易但不值钱。** 它挂 `tradable` 却没写 `goldvalue`，猪王不收；食人花球茎连 `tradable` 都没有。[默认值](../../../raw/current/scripts/components/tradable.lua#L1-L7)
- **未核验**：游戏内运行；食人花战斗与眼球草行为；女王转化的实际触发；沃姆伍德技能解锁后的界面与实际产出；活动皮肤（`prefabskin.lua` 的 Plantera 皮肤只核到 `linked_skinname` 传递路径）。

## 验证记录

- 源码审阅：2026-11-10，Steam buildid 26474873。通读 `lureplant_bulb.lua`、`spidereggsack.lua` 全文；核对 `lureplant.lua` 的 `FreshSpawn`／`WakeUp`／`OnIsWinter`／组件段与战利品，`SGlureplant.lua` 的 `spawn`／`hibernate` 状态，`spiderden.lua` 的 `SetStage`／`LOOT_DATA`／`growth_stages`／`OnLoadPostPass`／三个注册，`spiderqueen.lua` 战利品表，`skilltree_wormwood.lua` 与 `recipes.lua` 的门禁，`brains/rabbitbrain.lua`／`brains/carratbrain.lua` 的饵判定，以及 `deployable`／`map`／`childspawner`／`growable`／`stackable`／`tradable`／`standardcomponents` 的相关分支与 `tuning`／`constants` 常量。
  名称核验：`NAMES.LUREPLANTBULB`＝食人花种子、`LUREPLANT`＝食人花、`SPIDEREGGSACK`＝蜘蛛卵、`SPIDERDEN`＝蜘蛛巢。
  未覆盖：食人花战斗与眼球草、女王转化实测、活动皮肤、游戏内运行。
- 游戏运行：未运行。秒数按 `seg_time`＝30、`day_time`＝300、`total_day_time`＝480 推算。
- 结构检查：`validate_page.py` 通过（standard 档内）；`validate_source_links.py --strict` 0 错 0 警（计数见校验输出与交付回报）。Lua 示例用 `luaparser` 4.2.0 解析通过（非游戏运行时）。

## 相关链接

- 种植类：[[reference/prefabs/tree-seeds]]、[[reference/prefabs/special-tree-planting]]、[[reference/prefabs/acorn]]
- 蜘蛛与巢：[[reference/prefabs/index|Prefab 索引]]
- 机制页：[[reference/components/deployable]]、[[reference/components/childspawner]]、[[reference/components/growable]]、[[reference/components/burnable]]

返回 [[reference/prefabs/index|Prefab 索引]]。
