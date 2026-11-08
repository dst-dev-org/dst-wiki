---
title: "浆果丛与香蕉丛的移植件（dug_berrybush、dug_berrybush2、dug_berrybush_juicy、dug_bananabush）"
created: "2026-11-08"
updated: "2026-11-11"
type: prefab
summary: "berrybush.lua 的三丛与 bananabush.lua 一丛：采摘产物与再生、施肥／枯萎／冬季与雪覆盖的分工、挖起产出与回种后 pickable:OnTransplant 的恢复条件，以及 8 个挖出件注册的数据差异。"
aliases: ["dug_berrybush", "dug_berrybush_placer", "dug_berrybush2", "dug_berrybush2_placer", "dug_berrybush_juicy", "dug_berrybush_juicy_placer", "dug_bananabush", "dug_bananabush_placer", "berrybush", "berrybush2", "berrybush_juicy", "bananabush", "浆果丛", "多汁浆果丛", "香蕉丛", "移植浆果丛"]
tags: [prefab, item]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/berrybush.lua", "prefabs/bananabush.lua", "prefabs/plantables.lua", "prefabs/marsh_bush.lua", "prefabs/waxed_plant_common.lua", "prefabutil.lua", "components/pickable.lua", "components/witherable.lua", "components/growable.lua", "components/deployable.lua", "components/floater.lua", "components/inventoryitem_replica.lua", "components/stackable.lua", "components/lunarthrall_plantspawner.lua", "components/map.lua", "standardcomponents.lua", "actions.lua", "componentactions.lua", "tuning.lua", "constants.lua", "strings.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 浆果丛与香蕉丛的移植件（dug_berrybush、dug_berrybush2、dug_berrybush_juicy、dug_bananabush）

## 快速判断

- 职责与适用：`prefabs/berrybush.lua` 的 `createbush(name, inspectname, berryname, master_postinit)` 产出 `berrybush`／`berrybush2`／`berrybush_juicy` 三丛，`prefabs/bananabush.lua` 单独产出 `bananabush`；挖出件都由 `prefabs/plantables.lua` 的 `make_plantable` 生成。本页负责 `dug_berrybush`、`dug_berrybush_placer`、`dug_berrybush2`、`dug_berrybush2_placer`、`dug_berrybush_juicy`、`dug_berrybush_juicy_placer`、`dug_bananabush`、`dug_bananabush_placer` 共 8 个注册；`berrybush`、`berrybush2`、`berrybush_juicy`、`bananabush` 四个世界态作来源。[丛工厂](../../../raw/current/scripts/prefabs/berrybush.lua#L228-L241)、[香蕉丛](../../../raw/current/scripts/prefabs/bananabush.lua#L150-L237)、[注册循环](../../../raw/current/scripts/prefabs/plantables.lua#L186-L191)
- 优先使用：要改采摘产物、再生时长与轮次看[采摘与再生](#采摘与再生)；要改香蕉丛的生长段与挖起看[香蕉丛的生长链](#香蕉丛的生长链)；公共厂、放置校验、燃料与堆叠等挖出件通用接线见 [[reference/prefabs/grass-saplings#公共厂与调用链]]，不在本页重复。
- 重要副作用：挖起成熟丛给「产物 ×N ＋ `dug_<prefab>`」，而高温枯萎（`withered`）或枯死（`barren`）时只给 2 根 `twigs` 且**不给**挖出件；回种时 `ondeploy` 调 `pickable:OnTransplant()`，浆果丛与香蕉丛都把它变成枯死态，必须先施肥才会重新结果。[挖起](../../../raw/current/scripts/prefabs/berrybush.lua#L170-L198)、[移植回调](../../../raw/current/scripts/prefabs/berrybush.lua#L200-L204)、[回种](../../../raw/current/scripts/prefabs/plantables.lua#L17-L36)
- 修改边界：`berries`／`berries_juicy` 的生熟数值与食用留在 [[reference/prefabs/wild-berries]]，`cave_banana` 的数值留在 [[reference/prefabs/tropical-gathered-fruits]]；蜡封挖出件归 B14／B43；`marsh_bush`、`monkeytail`、`rock_avocado_bush` 的挖出形态归 B14。

## 注册与构造

世界态与挖出件出自两套工厂，但挖出件的数据表就写在 `plantables.lua` 里，改数据表即可换 bank／build／间距与图鉴文案。

| 世界态注册名 | 工厂与初始化分界 | 变体差异与证据 |
|---|---|---|
| `berrybush` | `createbush("berrybush", "berrybush", "berries", normal_postinit)`：`fn` 先客户端段（标签、bank／build、`idle`、莓果符号、雪覆盖）→ `SetPristine` → 主模拟段（pickable／witherable／可燃／workable／inspectable／雪覆盖／蜡封）→ `master_postinit` | `inspectname` 与注册名相同，故不设 `inspectable.nameoverride`；图鉴提示 `NEEDFERTILIZER`。[注册](../../../raw/current/scripts/prefabs/berrybush.lua#L368-L370)、[主模拟段](../../../raw/current/scripts/prefabs/berrybush.lua#L284-L322) |
| `berrybush2` | 同一个 `fn`，只换名字与资源 | `inspectname = "berrybush"` ≠ 注册名，于是 `inspectable.nameoverride = "berrybush"`，检查台词借用普通浆果丛；其余行为与 `berrybush` 完全一致（同 `normal_postinit`）。[注册](../../../raw/current/scripts/prefabs/berrybush.lua#L368-L370)、[名字覆盖](../../../raw/current/scripts/prefabs/berrybush.lua#L310-L313) |
| `berrybush_juicy` | 同一个 `fn`，资源与 `juicy_postinit` 不同 | 产物 `berries_juicy`、每次 3 个、`jostlepick` 撞取、`droppicked` 抛落、`dropheight = 3.5`；`inspectname` 同名，无 nameoverride。[juicy 段](../../../raw/current/scripts/prefabs/berrybush.lua#L353-L366) |
| `bananabush` | 独立 `fn`：标签 `bananabush`／`plant`、小障碍碰撞、`bananabush` bank／build、`idle_small`、`scrapbook_anim = "idle_big"` | 产物 `cave_banana`，采摘由 `growable` 四段控制而不是 `pickable` 计时；`canbepicked` 初始为 false。[构造](../../../raw/current/scripts/prefabs/bananabush.lua#L150-L200) |

三个浆果丛共用同一份客户端段与主模拟段：标签 `bush`／`plant`／`renewable`／`lunarplant_target`／`witherable`，`MakeSnowCoveredPristine` 与 `scrapbook_specialinfo = "NEEDFERTILIZER"` 都在 `SetPristine` 前；主模拟挂 `witherable`、`MakeLargeBurnable`、`MakeMediumPropagator`、`AddHauntableCustomReaction`、`MakeSnowCovered`、`MakeNoGrowInWinter`、`MakeWaxablePlant`。[客户端段](../../../raw/current/scripts/prefabs/berrybush.lua#L254-L279)、[主模拟段](../../../raw/current/scripts/prefabs/berrybush.lua#L294-L320)

| 挖出件注册名（含 placer） | 数据项与初始化分界 | 差异与证据 |
|---|---|---|
| `dug_berrybush`／`dug_berrybush_placer` | `make_plantable` 统一路径，无需额外组件；placer 由 `MakePlacer(..., "berrybush", "berrybush", "dead")` 生成 | floater `{"med", 0.2, 0.95}`；无 `inspectoverride`（回落 `dug_berrybush`）；`PLANTABLE_FERTILIZE`；未设 `mediumspacing`。[数据项](../../../raw/current/scripts/prefabs/plantables.lua#L122-L124) |
| `dug_berrybush2`／`dug_berrybush2_placer` | 同上，bank／build 换 `berrybush2` | `inspectoverride = "dug_berrybush"`；floater `{"large", 0.2, 0.65}`。[数据项](../../../raw/current/scripts/prefabs/plantables.lua#L127-L130) |
| `dug_berrybush_juicy`／`dug_berrybush_juicy_placer` | 同上，bank／build 换 `berrybush_juicy` | `inspectoverride = "dug_berrybush"`；floater `{"large", 0.025, {0.65, 0.5, 0.65}}`，第三项是按轴缩放的表。[数据项](../../../raw/current/scripts/prefabs/plantables.lua#L133-L136)、[缩放分支](../../../raw/current/scripts/components/floater.lua#L68-L87) |
| `dug_bananabush`／`dug_bananabush_placer` | 同上，bank／build `bananabush` | floater `{"med", 0.2, 0.95}`；`PLANTABLE_FERTILIZE`；`data.anim = "dead"` 只影响 placer。[数据项](../../../raw/current/scripts/prefabs/plantables.lua#L170-L172) |

- 四件的 `scrapbook_specialinfo` 都是 `PLANTABLE_FERTILIZE`，因为图鉴分流把 `berrybush`／`berrybush2`／`berrybush_juicy`／`bananabush` 都列进第一组；`PLANTABLE` 那组只有 `sapling`／`sapling_moon`／`marsh_bush`。[分流](../../../raw/current/scripts/prefabs/plantables.lua#L62-L76)
- 四件都没写 `mediumspacing`，所以部署间距走 `DEPLOYSPACING.DEFAULT`（半径 2），比草丛／树苗的 MEDIUM（半径 1）大；世界态一侧它们同样 `SetDeploySmartRadius(DEPLOYSPACING_RADIUS[...] / 2)`，浆果丛与香蕉丛用的是 DEFAULT／2。[间距枚举](../../../raw/current/scripts/constants.lua#L2236-L2246)、[半径表](../../../raw/current/scripts/constants.lua#L2254-L2263)、[浆果丛](../../../raw/current/scripts/prefabs/berrybush.lua#L251)、[香蕉丛](../../../raw/current/scripts/prefabs/bananabush.lua#L158)
- `dug_berrybush2` 与 `dug_berrybush_juicy` 的 `inspectoverride` 都指向 `dug_berrybush`，只表示检查台词里的物品名借用它；HUD 名仍按各自注册名取 `NAMES.DUG_BERRYBUSH2`／`NAMES.DUG_BERRYBUSH_JUICY`，三者的简中名分别是“浆果丛／浆果丛／多汁浆果丛”。[取名链路](../../../raw/current/scripts/stringutil.lua#L375)、[名称核验](#验证记录)
- 三件浆果丛挖出件另有配方来路：Wormwood 的技能配方 `wormwood_berrybush`／`wormwood_berrybush2`／`wormwood_juicyberrybush` 分别直接产出 `dug_berrybush`／`dug_berrybush2`／`dug_berrybush_juicy`，不必先挖活丛。[配方](../../../raw/current/scripts/recipes.lua#L344-L346)

## 采摘与再生

| 丛 | pickable 产物 | 每次数量 | 基础再生 | 轮次上限 | 交付方式 |
|---|---|---:|---|---|---|
| `berrybush`／`berrybush2` | `berries` | 1 | 3 天（`BERRY_REGROW_TIME`） | 3 ＋ random(2)，即 4–5 | 有库存则尝试入包，否则落地 |
| `berrybush_juicy` | `berries_juicy` | 3 | 9 天（`BERRY_JUICY_REGROW_TIME`） | 3 ＋ random(2)，即 4–5 | `droppicked` 逐个抛落（`dropheight` 3.5） |
| `bananabush` | `cave_banana` | 1 | 无 `pickable` 计时，靠 `growable` 四段生长（空→小→中→高，到可采需 3 个计时段） | 4 ＋ random(3)，即 5–7 | 有库存则尝试入包，否则落地 |

- 再生算法：`getregentimefn_normal` 返回 `BERRY_REGROW_TIME + BERRY_REGROW_INCREASE × 已耗轮次 + BERRY_REGROW_VARIANCE × random()`，即 3 天 ＋ 0.5 天／轮 ＋ 0～2 天随机；`juicy` 同构但基数是 9 天。排程入口是 `Pickable:Pick`，最终值还会经 `SpringGrowthMod` 在春季乘 0.75。[普通](../../../raw/current/scripts/prefabs/berrybush.lua#L110-L121)、[多汁](../../../raw/current/scripts/prefabs/berrybush.lua#L123-L134)、[常量](../../../raw/current/scripts/tuning.lua#L1774-L1782)、[春季](../../../raw/current/scripts/tuning.lua#L2284)
- 轮次只在 `transplanted` 时递减：野生丛 `cycles_left` 不变，可无限采；移植过的丛每采一次扣 1，扣到 0 即 `IsBarren()` 并把 `barren` 标签挂上，丛进入枯死态。[扣减](../../../raw/current/scripts/components/pickable.lua#L535-L539)、[判定](../../../raw/current/scripts/components/pickable.lua#L378-L384)、[标签](../../../raw/current/scripts/components/pickable.lua#L9-L15)
- 采摘重置：`Pick` 先结算产物与回调，再把 `canbepicked` 置 false；未枯死且未暂停时按上面的时长排 `regen` 任务，所以“采完”只是把可采位翻成不可采并重新计时，不是删掉重长。[Pick](../../../raw/current/scripts/components/pickable.lua#L535-L583)
- 施肥：`Pickable:Fertilize` 把 `cycles_left` 重置回 `max_cycles`，随后在未高温枯萎（无 `withered`）时 `MakeEmpty()`（排一次再生），高温枯萎时改为 `ForceRejuvenate()`；`CanBeFertilized` 只在 `IsBarren()` 或带 `withered` 时为真，动作候选也只对带 `barren` 标签的目标开放。[施肥](../../../raw/current/scripts/components/pickable.lua#L259-L293)、[条件](../../../raw/current/scripts/components/pickable.lua#L251-L253)、[动作候选](../../../raw/current/scripts/componentactions.lua#L1241-L1254)、[动作执行](../../../raw/current/scripts/actions.lua#L1818-L1822)
- 高温枯萎（`withered`）：`witherable` 在高温且未淋雨时调 `pickable:MakeBarren()`（转枯死）并暂存 `restore_cycles`；恢复走 `DoPickableRejuvenate`，先把 `restore_cycles` 写回 `cycles_left`，写回后仍非 `barren` 才 `MakeEmpty()`。枯萎期间丛不可采、挖起只掉 `twigs`。[枯萎](../../../raw/current/scripts/components/witherable.lua#L59-L69)、[恢复](../../../raw/current/scripts/components/witherable.lua#L102-L117)
- 冬季：三丛都 `MakeNoGrowInWinter`，入冬 `pickable:Pause()`、出冬 `Resume()`；出冬不一定是春季（长度为 0 的季节会被跳过），只有出冬时恰为春季才把剩余等待时间乘 0.75。被暂停期间采摘不会排再生计时，丛会一直空着到出冬。[暂停与恢复](../../../raw/current/scripts/standardcomponents.lua#L822-L833)、[排程条件](../../../raw/current/scripts/components/pickable.lua#L559-L573)
- 雪覆盖：`MakeSnowCoveredPristine` 只预挂 `SnowCovered` 标签并隐藏 `snow` 符号，`MakeSnowCovered` 按 `TheWorld.state.issnowcovered` 显隐同名符号；它不参与是否可采的判断，也不改再生。三丛另有 `SetLunarHailBuildupAmountSmall`，属于月雹堆积的同一套表现层。[雪覆盖](../../../raw/current/scripts/standardcomponents.lua#L835-L855)、[挂载](../../../raw/current/scripts/prefabs/berrybush.lua#L275)、[挂载](../../../raw/current/scripts/prefabs/berrybush.lua#L316-L318)
- 与采摘绑定的额外分支：浆果丛采完后按 `PERD_SPAWNCHANCE`（活动期取 `YOTG_PERD_SPAWNCHANCE`）在 3～6 秒后生成火鸡；多汁丛还有“工作未完成时补一次 `Pick`”的撞取分支。[火鸡](../../../raw/current/scripts/prefabs/berrybush.lua#L75-L108)、[补摘](../../../raw/current/scripts/prefabs/berrybush.lua#L158-L168)
- 挖起：`dig_up_common` 先看 `withered` 或 `IsBarren()`——成立时只 `SpawnLootPrefab("twigs")` 两次；否则仍可采就把产物按 `numberries`（普通 1、多汁 3）逐个抛在 `dropheight` 偏移处，再掉 `"dug_"..inst.prefab` 并带上皮肤。[挖起](../../../raw/current/scripts/prefabs/berrybush.lua#L170-L198)
- 回种：`ondeploy` 先 `SpawnPrefab`，**只有返回非空**才取 1 个挖出件 `Remove()` 并调用 `pickable:OnTransplant()`（它先把 `transplanted` 置真再执行 `ontransplantfn`）；返回空则回调不改动该堆叠；但玩家 `DEPLOY` 动作已在进回调前从物品栏拆出 1 件，只有 `Deploy` 返回失败才 `GiveItem`，所以动作仍算成功并照推 `deployitem`／`itemplanted`，种植提示不能反推世界态真的生成。浆果丛的回调播 `dead`、隐藏莓果并 `MakeBarren()`，所以回种后的丛必须靠施肥把 `cycles_left` 填回 `max_cycles` 并触发 `MakeEmpty` 才会重新结果。[回种](../../../raw/current/scripts/prefabs/plantables.lua#L17-L36)、[移植入口](../../../raw/current/scripts/components/pickable.lua#L418-L424)、[浆果丛回调](../../../raw/current/scripts/prefabs/berrybush.lua#L200-L204)、[施肥](../../../raw/current/scripts/components/pickable.lua#L274-L292)

## 香蕉丛的生长链

- **世界态构成**：只有 `bananabush`／`plant` 两个标签（无 `renewable`、无 `lunarplant_target`、无 `bush`），`MakeSmallObstaclePhysics(inst, .1)`，`idle_small` 起播，图鉴动画固定 `idle_big`。[构造](../../../raw/current/scripts/prefabs/bananabush.lua#L150-L174)
- **pickable 只借位**：`pickable:SetUp("cave_banana")` 不传再生时长，`baseregentime` 为空，所以 `Pick` 不会排定时器；可采性由 `canbepicked` 与 `growable` 推动：初始 `canbepicked = false`，只有长到第 4 段 `set_big` 时才 `pickable:Regen()`。[pickable](../../../raw/current/scripts/prefabs/bananabush.lua#L177-L191)、[第四段](../../../raw/current/scripts/prefabs/bananabush.lua#L39-L45)
- **四段生长**：`empty`／`small`／`normal`／`tall` 每段 `TOTAL_DAY_TIME`（1 天），`loopstages = false`、`springgrowth = true`；`SetStage(1)` 后 `StartGrowing`，春季每段乘 0.75。野生丛从空到可采要过 3 个计时段（约 3 天）。[阶段表](../../../raw/current/scripts/prefabs/bananabush.lua#L51-L76)、[启动](../../../raw/current/scripts/prefabs/bananabush.lua#L194-L203)、[春季](../../../raw/current/scripts/tuning.lua#L2284)
- **采摘**：`OnPicked` 在枯死时播 `idle_to_dead` 并 `StopGrowing`，否则播 `picked`／`idle_empty`、`SetStage(1)` 再 `StartGrowing`；没有 `pickable` 计时，因此“再生”就是从第 1 段重长。[采摘](../../../raw/current/scripts/prefabs/bananabush.lua#L99-L114)
- **轮次**：`max_cycles = BANANABUSH_CYCLES + random(BANANABUSH_CYCLES_VAR)`，即 5–7；同样只在 `transplanted` 时递减，野生丛不耗尽。[轮次](../../../raw/current/scripts/prefabs/bananabush.lua#L180-L183)、[常量](../../../raw/current/scripts/tuning.lua#L6741-L6742)
- **挖起**：`OnDig` 与浆果丛同构——高温枯萎或枯死只掉 2 根 `twigs`，否则先按 `dropheight` 偏移掉 `cave_banana`（仍可采时）再掉 `dug_bananabush`。[挖起](../../../raw/current/scripts/prefabs/bananabush.lua#L78-L97)
- **回种**：`OnTransplant` 只调 `pickable:MakeBarren()`，但该丛注册了 `SetMakeBarrenFn(MakeBarren)`，于是连带把 `growable` 停在第 1 段、关掉 `magicgrowable` 并播 `dead`；施肥走 `Fertilize → MakeEmpty`，`MakeEmpty` 里重新 `SetStage(1)`＋`StartGrowing`＋打开 `magicgrowable`。`OnRegen` 另会在 `pickable:Regen` 时 `Resume()` 并把阶段补到 4。[移植](../../../raw/current/scripts/prefabs/bananabush.lua#L116-L141)、[换绑](../../../raw/current/scripts/prefabs/bananabush.lua#L185-L191)
- **读档补正**：`on_load` 在存档里已枯萎时强制 `ForceWither()`，避免 `pickable` 在读档后自行再生。[读档](../../../raw/current/scripts/prefabs/bananabush.lua#L143-L148)
- **边界**：`bananabush` 不挂 `MakeSnowCovered`，雪覆盖对它没有表现；冬季唯一的处理是 `MakeNoGrowInWinter` 对 `pickable` 的 `Pause`；它本来就没有 `pickable` 再生计时，而 `growable` 不参与这条暂停，冬季仍按 1 天／段推进。出冬（不一定是春季）`Resume()` 时因 `pause_time` 为空，若仍未长成且未枯死会走 `MakeEmpty` 退回第 1 段。[主模拟段](../../../raw/current/scripts/prefabs/bananabush.lua#L219-L231)、[暂停](../../../raw/current/scripts/standardcomponents.lua#L822-L833)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：pickable | 世界态定义产物、每次数量、再生与移植回调 | 主模拟；挖出件不挂 | [[reference/components/pickable#关键接口]]；[接线](../../../raw/current/scripts/prefabs/berrybush.lua#L286-L292) |
| 组件：witherable | 高温枯萎 → `MakeBarren`；恢复先写回轮次，仍非 `barren` 才 `MakeEmpty` | 四丛都有，主模拟 | [[reference/components/witherable#关键接口]]；[枯萎](../../../raw/current/scripts/components/witherable.lua#L59-L69) |
| 组件：growable | 香蕉丛四段生长，取代 `pickable` 计时 | 仅 `bananabush` | [[reference/components/growable#关键接口]]；[启动](../../../raw/current/scripts/prefabs/bananabush.lua#L194-L200) |
| 组件：deployable | 挖出件 PLANT 模式 + `ondeploy` 回种 | 挖出件，主模拟 | [[reference/components/deployable#关键接口]]；[公共厂](../../../raw/current/scripts/prefabs/plantables.lua#L99-L105) |
| 组件：stackable | 挖出件上限 10，放置取 1 个 | 挖出件 | [[reference/components/stackable#关键接口]]；[取 1 个](../../../raw/current/scripts/components/stackable.lua#L109-L120) |
| 组件：floater | `floater` 前三项决定浮水外观，第三项可给按轴缩放表 | 挖出件，客户端 | [[reference/components/floater]]；[缩放](../../../raw/current/scripts/components/floater.lua#L68-L87) |
| 组件：lunarthrall_plantspawner | 浆果丛带 `lunarplant_target`，落地与读档时补 herdmember | 世界组件存在时 | [[reference/components/lunarthrall_plantspawner#关键接口]]；[登记](../../../raw/current/scripts/components/lunarthrall_plantspawner.lua#L529-L533)、[读档](../../../raw/current/scripts/prefabs/berrybush.lua#L220-L226) |
| API／数据：`MakePlacer` | 四个 placer，动画取 `data.anim or "idle"`（本页都是 `dead`） | 客户端预览 | [[reference/components/placer#关键接口]]；[工厂](../../../raw/current/scripts/prefabutil.lua#L1-L65) |
| API／数据：`DEPLOYMODE`／`DEPLOYSPACING` | PLANT 模式；四件都用默认间距 | 主模拟设值，replica 同步 | [枚举](../../../raw/current/scripts/constants.lua#L2215-L2246) |
| 关联 Prefab：`berries`／`berries_juicy` | 三丛的采摘产物，数值与食用在对应页 | 物品 | [[reference/prefabs/wild-berries]]；[SetUp](../../../raw/current/scripts/prefabs/berrybush.lua#L343-L354) |
| 关联 Prefab：`cave_banana` | 香蕉丛唯一产物 | 物品 | [[reference/prefabs/tropical-gathered-fruits]]；[SetUp](../../../raw/current/scripts/prefabs/bananabush.lua#L186) |
| 关联 Prefab：`dug_<name>_waxed` | 蜡封挖出件，保留外观与 `savedata` | 本页只给指针，B14／B43 | [工厂](../../../raw/current/scripts/prefabs/waxed_plant_common.lua#L440-L539) |
| 资源：anim 包 | `anim/berrybush.zip`／`berrybush2.zip`／`berrybush_juicy.zip`／`bananabush.zip` | 世界态与 placer 共用 | [浆果丛资产](../../../raw/current/scripts/prefabs/berrybush.lua#L229-L233)、[香蕉丛资产](../../../raw/current/scripts/prefabs/bananabush.lua#L1-L5) |

## 典型接入

### 让回种后的浆果丛免施肥

- 前提与选择：服务端 `modmain.lua`。改世界态的 `pickable.ontransplantfn`：`ondeploy` 在 `SpawnPrefab` 之后才调 `OnTransplant()`，postinit 先于该调用执行，替换后放置的丛直接进入“空株 + 再生”，而不是“枯死 + 等施肥”。
- 生效与撤销：`AddPrefabPostInit` 在每次构造（含读档重建）后执行；停用模组即恢复。`OnTransplant` 仍会置 `transplanted = true`，所以此后每采一次扣 1 个轮次（初始 4–5），扣到 0 仍会枯死并需要施肥——免掉的只是“回种即枯死”。
- 原版依据：默认回调与 `MakeBarren`／`MakeEmpty` 的差别、以及施肥如何填回轮次见[采摘与再生](#采摘与再生)。

```lua
-- modmain.lua：回种后的三丛都不再枯死，保留满轮次并立刻开始再生
for _, prefab in ipairs({ "berrybush", "berrybush2", "berrybush_juicy" }) do
    AddPrefabPostInit(prefab, function(inst)
        if not GLOBAL.TheWorld.ismastersim or inst.components.pickable == nil then
            return
        end
        inst.components.pickable.ontransplantfn = function(plant)
            plant.components.pickable:MakeEmpty()
        end
    end)
end
```

- 调整与验证：单位是轮与秒；`MakeEmpty` 会用各自的 `getregentimefn` 排程（普通 3 天／多汁 9 天起），并让丛立刻显示空株。若想连轮次也不扣，得改 `Pick` 的 `transplanted` 分支（组件级 Hook），影响面会扩大到这个组件上的所有植物。

## 常见陷阱与限制

- **边界：高温枯萎或枯死的丛挖不出挖出件**：`dig_up_common` 在 `withered` 或 `IsBarren()` 时只给 2 根 `twigs`，想靠挖取留种就必须在丛还活着时挖（配方来路见[注册与构造](#注册与构造)）。[分支](../../../raw/current/scripts/prefabs/berrybush.lua#L170-L190)
- **边界：回种不是“立刻能吃”**：`OnTransplant` 之后 `cycles_left` 为 0，动作候选只对 `barren` 目标给 FERTILIZE，所以“种下去 → 施肥 → 等一次再生”是固定三段，不能省中间一步。[回调](../../../raw/current/scripts/prefabs/berrybush.lua#L200-L204)、[候选](../../../raw/current/scripts/componentactions.lua#L1241-L1254)
- **误区：野生浆果丛会越采越少**：轮次只在 `transplanted` 时扣，野生丛 `cycles_left` 始终是满值；施肥、枯萎恢复都不会改变这条规则。[扣减](../../../raw/current/scripts/components/pickable.lua#L535-L539)
- **边界：冬季采摘会把丛“冻结”在空株**：`Pause` 期间 `Pick` 跳过排程，直到出冬 `Resume()` 才补一次（出冬不一定是春季，只有恰逢春季才乘 0.75）；雪覆盖本身只切 `snow` 符号，不拦采摘、不改再生。[暂停](../../../raw/current/scripts/standardcomponents.lua#L822-L833)、[排程](../../../raw/current/scripts/components/pickable.lua#L559-L573)
- **边界：“枯萎只给树枝”不是所有灌木的规则**：同厂的 `marsh_bush` 挖起时无论状态都掉 `dug_marsh_bush`，只有 `twigs` 是额外补的可采产物；照浆果丛的挖起口径去推 `dug_marsh_bush` 会算错。该项目归 B14。[对照](../../../raw/current/scripts/prefabs/marsh_bush.lua#L28-L34)
- **边界：香蕉丛的再生不在 `pickable` 上**：`SetUp` 没给再生时长，采后靠 `growable` 从第 1 段重长；给 `pickable.regentime` 赋值不会让它提前结果。[接线](../../../raw/current/scripts/prefabs/bananabush.lua#L177-L191)、[重长](../../../raw/current/scripts/prefabs/bananabush.lua#L99-L114)
- **未核验**：quagmire 模式下 `berrybush` 的 `event_server_data` 分支、`SetLunarHailBuildupAmountSmall` 与月雹堆积的完整表现、蜡封挖出件的外观／存档细节（B14／B43）、`marsh_bush` 等 B14 页的对照结论。

## 验证记录

- 源码审阅：2026-11-08，Steam buildid 26474873（2026-11-09 复检回种门禁、冬季暂停口径与恢复分支）。逐行读完 `berrybush.lua`（`createbush`、客户端段、主模拟段、`normal_postinit`／`juicy_postinit`、采摘／再生／挖起／移植回调、三件注册）与 `bananabush.lua`（阶段表、pickable／growable 接线、OnPicked／OnDig／OnTransplant／MakeEmpty／MakeBarren／OnRegen、读档补正）；核验 `plantables.lua` 的四个数据项与图鉴分流、`pickable` 的 SetUp／Pick／Fertilize／MakeBarren／MakeEmpty／OnTransplant、`witherable` 的枯萎与恢复、`growable` 的阶段推进与春季倍率、`floater` 的缩放表分支、`deployable` 与 `map` 的落点判定、`standardcomponents` 的冬季暂停与雪覆盖 helper、FERTILIZE 的动作候选与执行、tuning 常量。

  名称核验（`inspect_prefab_names.py`，只认 `languages/chinese_s.po` 的 msgstr）：`BERRYBUSH`＝浆果丛、`BERRYBUSH2`＝浆果丛、`BERRYBUSH_JUICY`＝多汁浆果丛、`BANANABUSH`＝香蕉丛；挖出件 `DUG_BERRYBUSH`＝浆果丛、`DUG_BERRYBUSH2`＝浆果丛、`DUG_BERRYBUSH_JUICY`＝多汁浆果丛、`DUG_BANANABUSH`＝香蕉丛。[BERRYBUSH](../../../raw/current/scripts/languages/chinese_s.po#L339415-L339417)、[BERRYBUSH2](../../../raw/current/scripts/languages/chinese_s.po#L339420-L339422)、[BERRYBUSH_JUICY](../../../raw/current/scripts/languages/chinese_s.po#L339425-L339427)、[BANANABUSH](../../../raw/current/scripts/languages/chinese_s.po#L339020-L339022)、[DUG_BERRYBUSH](../../../raw/current/scripts/languages/chinese_s.po#L342305-L342307)、[DUG_BERRYBUSH2](../../../raw/current/scripts/languages/chinese_s.po#L342310-L342312)、[DUG_BERRYBUSH_JUICY](../../../raw/current/scripts/languages/chinese_s.po#L342315-L342317)、[DUG_BANANABUSH](../../../raw/current/scripts/languages/chinese_s.po#L342300-L342302)
- 游戏运行：未运行。天数与秒数为 tuning 推导（3 天＝1440 秒、9 天＝4320 秒、春季 ×0.75）；轮次区间由 `math.random` 上界推导（`BERRYBUSH_CYCLES`＋random(2) 得 4–5、`BANANABUSH_CYCLES`＋random(3) 得 5–7）。
- 结构检查：`validate_page.py`（`--wiki-root` 指向 `dst-wiki/wiki`）通过，standard 档内（行数／字节见校验输出）；`validate_source_links.py --strict` 0 错 0 警。

## 相关链接

- 产物数值：[[reference/prefabs/wild-berries]]（berries 与 berries_juicy）、[[reference/prefabs/tropical-gathered-fruits]]（cave_banana）
- 公共厂与放置：[[reference/prefabs/grass-saplings]]（`make_plantable`、placer、燃料与堆叠）
- 工具与施肥：[[reference/prefabs/farm-hoe]]、[[reference/prefabs/soil-amender]]、[[reference/components/fertilizer#关键接口]]
- 组件：[[reference/components/pickable]]、[[reference/components/witherable]]、[[reference/components/growable]]、[[reference/components/deployable]]

返回 [[reference/prefabs/index|Prefab 索引]]。
