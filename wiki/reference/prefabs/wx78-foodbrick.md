---
title: "营养砖（wx78_foodbrick）"
created: "2026-11-09"
updated: "2026-11-09"
type: prefab
summary: "WX-78 单消化电路吃掉 5 份腐烂食物后烤出的砖：干态是 RAW、玩家与 WX-78 的食性都不含 RAW，所以吃不了，但能当 90 秒燃料与 `inedible` 锅料；浇湿后在同一实体上就地改成 GENERIC 食物＋肥料＋可部署，干湿两态不可叠，湿砖放进凝胶团储藏会停表。"
aliases: ["wx78_foodbrick", "营养砖", "受潮的营养砖", "Nutribrick", "消化电路产物", "WX78 砖"]
tags: [prefab, item, food]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/wx78_foodbrick.lua", "prefabs/wx78.lua", "constants.lua", "prefabs/fertilizer_nutrient_defs.lua", "prefabs/gelblob_storage.lua", "stategraphs/SGwx78_common.lua", "wx78_moduledefs.lua", "cooking.lua", "components/edible.lua", "components/eater.lua", "components/perishable.lua", "components/fertilizer.lua", "components/deployable.lua", "components/inventoryitem.lua", "components/inventoryitemmoisture.lua", "components/inventoryitemholder.lua", "components/wateryprotection.lua", "components/fueled.lua", "components/grower.lua", "components/pickable.lua", "components/crop.lua", "components/tradable.lua", "components/inventoryitem_replica.lua", "componentutil.lua", "entityscript.lua", "preparedfoods.lua", "componentactions.lua", "actions.lua", "standardcomponents.lua", "tuning.lua", "screens/redux/scrapbookdata.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 营养砖（wx78_foodbrick）

## 快速判断

- 职责与适用：单一注册 `wx78_foodbrick`，物品态，原版里是 WX-78 的**消化电路**（digestion，GAMMA 槽、2 格）把吃下去的腐烂食物攒够数后，在 `wx_bake` 状态里掉出来的产物；干态是 `RAW`，玩家默认食性 `FOODGROUP.OMNI` 与 WX-78 的 `{OMNI, GEARS}` 都不含 RAW，所以连同 EAT 候选都出不来、也不能施肥；湿态是 `GENERIC`，才既是食物又是肥料。[注册](../../../raw/current/scripts/prefabs/wx78_foodbrick.lua#L176-L176)、[掉落点](../../../raw/current/scripts/stategraphs/SGwx78_common.lua#L1143-L1151)
- 优先使用：想知道干湿各自改了什么看[干湿两态](#干湿两态就地改字段)；数值看[数值表](#数值)；谁生成、能去哪看[来源与去向](#来源与去向)；肥料与部署看[关联机制导航](#关联机制导航)的 `fertilizer` 与 `deployable` 行。
- 重要副作用：湿态是**同一实体上就地改字段**（改 `edible` 的三维与 `foodtype`、按需增删 `fertilizer`／`deployable` 组件与 `tile_deploy` 标签、切换物品图与拾取音），不是替换成另一个 prefab；`CanStackWithFn` 比双方 `IsWet()`，所以**干砖与湿砖叠不到一起**。[改字段](../../../raw/current/scripts/prefabs/wx78_foodbrick.lua#L9-L56)、[不可叠](../../../raw/current/scripts/prefabs/wx78_foodbrick.lua#L90-L92)
- 修改边界：本页只负责这个注册与两态切换；WX-78 电路与 `wx_bake` 状态机归 `wx78_moduledefs.lua` 与 `SGWX78Common`（Wiki 待建），肥料在农田侧的结算归 [[reference/components/fertilizer]]，湿度本体归 [[reference/components/inventoryitemmoisture]]。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `wx78_foodbrick` | `fn()`：`CreateEntity` → bank／build `wx78_foodbrick`、播 `idle_dry` → 加标签 `fertilizerresearchable`／`canpourwateron` → `MakeInventoryFloatable`＋`MakeDeployableFertilizerPristine` → 写 `stackable_CanStackWithFn`／`itemtile_Refresh`／`wet_prefix`／`GetFertilizerKey` → `SetPristine`；**客户端只挂 `wetnesschange` 监听就返回**，其余组件全在主模拟 | 初始为**干态**：`inspectable`＋`getstatus`、`inventoryitem`（并给 `inventoryitemmoisture` 设 `SetOnMoistureDeltaCallback`）、`stackable`、`tradable`、`fuel`、`edible`、`perishable`、`fertilizerresearchable`，可燃、传热、可作祟。[工厂](../../../raw/current/scripts/prefabs/wx78_foodbrick.lua#L108-L174)、[客户端分界](../../../raw/current/scripts/prefabs/wx78_foodbrick.lua#L134-L139) |

构造时就固定的四项：`MakeDeployableFertilizerPristine` 给**两端**都加 `deployable`／`tile_deploy` 标签并写 `_custom_candeploy_fn`（只认可耕土壤）与 `overridedeployplacername`；`wet_prefix` 取 `STRINGS.WET_PREFIX.WX78_FOODBRICK`，湿态显示名由它拼出；`GetFertilizerKey` 直接返回 `inst.prefab`，供植物登记册按注册名记肥料；`COMMON_ItemTileRefresh` 只是按湿度推 `show_spoilage`／`hide_spoilage`。[pristine helper](../../../raw/current/scripts/standardcomponents.lua#L1812-L1821)、[四项固定](../../../raw/current/scripts/prefabs/wx78_foodbrick.lua#L126-L132)

### 名称核验

简中实显 `NAMES.WX78_FOODBRICK` ＝ **营养砖**（英文 "Nutribrick"）；湿态前缀 `WET_PREFIX.WX78_FOODBRICK` ＝ **受潮的**，所以湿砖显示为"受潮的营养砖"——`GetName` 在该实体 `GetIsWet()` 为真时用 `wet_prefix` 拼形容词。[名称](../../../raw/current/scripts/languages/chinese_s.po#L352690-L352692)、[湿前缀](../../../raw/current/scripts/languages/chinese_s.po#L445450-L445452)、[拼名](../../../raw/current/scripts/entityscript.lua#L711-L714)

## 干湿两态：就地改字段

切换只有一个入口：`inventoryitemmoisture` 的 `OnMoistureDeltaCallback` 读 `inventoryitem:IsWet()`，真则 `MakeWet`、假则 `MakeDry`；湿度每次变化（下雨、泡水、浇水、读档回调）都会走一遍，所以字段始终跟着湿度走。[入口](../../../raw/current/scripts/prefabs/wx78_foodbrick.lua#L58-L76)

| 字段／对象 | 构造即干（`fn`） | 湿（`MakeWet`） | 再干（`MakeDry`） |
|---|---|---|---|
| `edible.healthvalue` | `HEALING_TINY`＝1 | 1 | 1 |
| `edible.hungervalue` | **0** | `CALORIES_MEDSMALL`＝18.75 | `CALORIES_TINY`＝9.375 |
| `edible.sanityvalue` | 0 | 0 | 0 |
| `edible.foodtype` | `RAW` | `GENERIC` | `RAW` |
| `fertilizer` 组件 | 无 | 无则添加（见下） | `RemoveComponent` |
| `deployable` 组件 | 无 | 无则添加 | `RemoveComponent`（组件自己清 `deployable` 标签与 deploymode） |
| `tile_deploy` 标签 | 有（pristine helper） | 再 `AddTag` 一次 | `RemoveTag` |
| `perishable` 计时 | **没起表** | 在凝胶团储藏里 `StopPerishing`，否则 `StartPerishing` | `StopPerishing` |
| 物品图／拾取音 | 原图、`vegetation_firm` | `wx78_foodbrick_wet`、`squidgy`、推 `show_spoilage` | 原图、`vegetation_firm`、推 `hide_spoilage` |

- **按回调历史分四种状态，只有湿态可食**：① 刚构造＝`RAW`／饥饿 0；② 首次低湿回调（可能从未湿过）＝`RAW`／9.375；③ 湿＝`GENERIC`／18.75；④ 湿后再干＝`RAW`／9.375。①②④ 三个 `RAW` 状态都吃不了——EAT 候选按 `edible_<type>` 与 `<group>_eater` 配对，玩家只有 `OMNI_eater`（默认 `{FOODGROUP.OMNI}`），而 OMNI 的 `types` 不含 `RAW`。[状态表](#干湿两态就地改字段)、[OMNI](../../../raw/current/scripts/constants.lua#L2049-L2063)、[默认食性](../../../raw/current/scripts/components/eater.lua#L22-L24)、[WX-78 食性](../../../raw/current/scripts/prefabs/wx78.lua#L753-L755)、[EAT 候选](../../../raw/current/scripts/componentactions.lua#L2672-L2689)
- **干砖不走鲜度表**：`fn` 只 `SetPerishTime`＋`SetOnPerishFn`，**没有** `StartPerishing`，而 `SetPerishTime` 仅在已有 `updatetask` 时才重启，所以刚烤出来的干砖不计时；湿态才起表，再干立刻停表。[构造](../../../raw/current/scripts/prefabs/wx78_foodbrick.lua#L162-L164)、[SetPerishTime](../../../raw/current/scripts/components/perishable.lua#L188-L194)
- **湿砖在凝胶团储藏里停表**：`MakeWet` 先看 `inst.parent`，`parent.prefab == "gelblob_storage"` 就用 `StopPerishing`。这个 `parent` 是 `inventoryitemholder:GiveItem` 里 `self.inst:AddChild(item)` 建立的引擎父子关系；储藏本体在 `OnFoodGiven` 里也会 `StopPerishing` 并在取出时按 `_wasperishing` 决定是否重启。[分支](../../../raw/current/scripts/prefabs/wx78_foodbrick.lua#L16-L21)、[亲子关系](../../../raw/current/scripts/components/inventoryitemholder.lua#L139-L151)、[储藏停表](../../../raw/current/scripts/prefabs/gelblob_storage.lua#L83-L86)、[取出恢复](../../../raw/current/scripts/prefabs/gelblob_storage.lua#L125-L129)
- **湿才起效的肥料**：`MakeWet` 只在 `fertilizer` 组件不存在时添加，并一次性写 `fertilizervalue`／`soil_cycles`／`withered_cycles`／`SetNutrients`，再 `AddTag("tile_deploy")` 与 `MakeDeployableFertilizer(inst)`；`MakeDry` 移 `fertilizer`／`deployable` 两个组件并 `RemoveTag("tile_deploy")`。部署标签分三态：刚构造的砖有 pristine helper 加的 `deployable`／`tile_deploy` 但无组件；**首次低湿回调（可能从未湿过）**时 `deployable` 组件本就不存在，组件不存在时 `RemoveComponent` 跳过该代码块、不触发 `OnRemoveFromEntity`，只被 `MakeDry` 摘掉 `tile_deploy`，所以 `deployable` 标签仍在；湿过再干时组件存在，`RemoveComponent` 才触发 `OnRemoveFromEntity`，置 deploymode 为 `NONE` 并 `RemoveTag("deployable")`，两个标签都不剩。[湿态挂件](../../../raw/current/scripts/prefabs/wx78_foodbrick.lua#L28-L37)、[干态拆除](../../../raw/current/scripts/prefabs/wx78_foodbrick.lua#L53-L55)、[清理](../../../raw/current/scripts/components/deployable.lua#L55-L62)、[RemoveComponent](../../../raw/current/scripts/entityscript.lua#L670-L680)
- **动画只在跨态时补一次**：`_waswet` 记录上一次状态，首次变湿 `PushAnimation("idle_wet")`、首次变干 `PushAnimation("idle_dry")`，同态重复回调不会重播。[动画](../../../raw/current/scripts/prefabs/wx78_foodbrick.lua#L58-L76)
- **读档会重放一次切换**：`inventoryitemmoisture:OnLoad` 读完存档湿度后回调 `onmoisturedeltacallback`，所以存档里的湿砖读档后字段与湿度一致，不依赖存档里的 `edible` 值。[OnLoad](../../../raw/current/scripts/components/inventoryitemmoisture.lua#L251-L259)

### 湿度怎么来

`iswet` 有阈值与滞后：`MAX_WETNESS`＝100，`MOISTURE_WET_THRESHOLD`＝35 起湿、`MOISTURE_DRY_THRESHOLD`＝15 才转干，两者之间保持原状。[常量](../../../raw/current/scripts/tuning.lua#L88-L88)、[阈值](../../../raw/current/scripts/tuning.lua#L2930-L2931)、[滞后](../../../raw/current/scripts/components/inventoryitemmoisture.lua#L150-L164)

| 途径 | 链 | 证据 |
|---|---|---|
| 淋雨／泡水 | 无 owner 时目标湿度取世界 `wetness`（下雨且无 `rainimmunity`），漂在水上直接取 `MAX_WETNESS`；`UpdateMoisture` 每秒向目标靠拢（涨 0.5/秒、落 1/秒），每次改动都回调 | [取目标](../../../raw/current/scripts/components/inventoryitemmoisture.lua#L183-L227)、[靠拢](../../../raw/current/scripts/components/inventoryitemmoisture.lua#L229-L240) |
| 玩家浇水 | 砖带 `canpourwateron` → 手持带 `wateryprotection` 的物品出 `POUR_WATER` 候选 → `SpreadProtection` → `ApplyProtectionToEntity` 按 `addwetness` 调 `DoDeltaMoistureToEntity` → `inventoryitem:AddMoisture` → `inventoryitemmoisture:DoDelta` | [候选](../../../raw/current/scripts/componentactions.lua#L2529-L2533)、[动作](../../../raw/current/scripts/actions.lua#L5416-L5434)、[扩散](../../../raw/current/scripts/components/wateryprotection.lua#L21-L41)、[入库](../../../raw/current/scripts/componentutil.lua#L2399-L2408)、[AddMoisture](../../../raw/current/scripts/components/inventoryitem.lua#L172-L176) |

## 数值

时间基准：`seg_time`＝30 秒、`day_time`＝`seg_time × 10`＝300 秒、`total_day_time`＝480 秒、`perish_warp`＝1。[基准](../../../raw/current/scripts/tuning.lua#L19-L29)、[`perish_warp`](../../../raw/current/scripts/tuning.lua#L47-L47)

| 项 | 干（构造） | 湿 | 定义处／结算处 |
|---|---|---|---|
| 饥饿／生命／理智 | 0／1／0（`RAW`，**不可食**） | 18.75／1／0（`GENERIC`，可食） | 干：[构造](../../../raw/current/scripts/prefabs/wx78_foodbrick.lua#L156-L160)；湿：[MakeWet](../../../raw/current/scripts/prefabs/wx78_foodbrick.lua#L23-L26)；常量：[HEALING_TINY](../../../raw/current/scripts/tuning.lua#L2021-L2021)、[热量档](../../../raw/current/scripts/tuning.lua#L2058-L2060)；结算：[eater 取生命](../../../raw/current/scripts/components/eater.lua#L213-L213)、[取饥饿](../../../raw/current/scripts/components/eater.lua#L224-L224) |
| 堆叠 | 40 | 40 | [赋值](../../../raw/current/scripts/prefabs/wx78_foodbrick.lua#L148-L149)、[常量](../../../raw/current/scripts/tuning.lua#L83-L83) |
| 燃料 | `MED_LARGE_FUEL`＝`seg_time×3`＝90 秒 | 90×`WET_FUEL_PENALTY`(0.75)＝67.5 秒 | [赋值](../../../raw/current/scripts/prefabs/wx78_foodbrick.lua#L153-L154)、[常量](../../../raw/current/scripts/tuning.lua#L1676-L1676)、[湿惩罚](../../../raw/current/scripts/tuning.lua#L1671-L1671)、[结算](../../../raw/current/scripts/components/fueled.lua#L205-L212) |
| 肥料值 | 无（干砖没有 `fertilizer`） | `WX78_FOODBRICK_FERTILIZE`＝`day_time×0.75`＝225 | [赋值](../../../raw/current/scripts/prefabs/wx78_foodbrick.lua#L30-L30)、[常量](../../../raw/current/scripts/tuning.lua#L9450-L9450)、[结算](../../../raw/current/scripts/components/crop.lua#L96-L96) |
| 土壤循环 | 无 | `WX78_FOODBRICK_SOILCYCLES`＝8 | [赋值](../../../raw/current/scripts/prefabs/wx78_foodbrick.lua#L31-L31)、[常量](../../../raw/current/scripts/tuning.lua#L9451-L9451)、[结算](../../../raw/current/scripts/components/grower.lua#L77-L77) |
| 枯萎循环 | 无 | `WX78_FOODBRICK_WITHEREDCYCLES`＝0.5 | [赋值](../../../raw/current/scripts/prefabs/wx78_foodbrick.lua#L32-L32)、[常量](../../../raw/current/scripts/tuning.lua#L9452-L9452)、[结算](../../../raw/current/scripts/components/pickable.lua#L271-L271) |
| 养分三元组 | 无 | `WX78_FOODBRICK_NUTRIENTS`＝`{0, 32, 0}` | [赋值](../../../raw/current/scripts/prefabs/wx78_foodbrick.lua#L33-L33)、[常量](../../../raw/current/scripts/tuning.lua#L5878-L5878)、[索引含义](../../../raw/current/scripts/tuning.lua#L5854-L5856)、[结算](../../../raw/current/scripts/standardcomponents.lua#L1797-L1802) |
| 鲜度 | 无计时起点（未 `StartPerishing`） | `PERISH_ONE_DAY`＝1 天＝480 秒，到点 `Remove` | [赋值](../../../raw/current/scripts/prefabs/wx78_foodbrick.lua#L162-L164)、[常量](../../../raw/current/scripts/tuning.lua#L2043-L2043) |
| 交易值 | 未赋值 → `tradable` 默认 0 | 同 | [挂件](../../../raw/current/scripts/prefabs/wx78_foodbrick.lua#L151-L151)、[默认](../../../raw/current/scripts/components/tradable.lua#L1-L3) |

养分三元组按 `FORMULA_NUTRIENTS_INDEX`＝1、`COMPOST_NUTRIENTS_INDEX`＝2、`MANURE_NUTRIENTS_INDEX`＝3 排列，所以 `{0, 32, 0}` 是**只有 32 点堆肥养分**；部署时 `fertilizer_ondeploy` 把三元组交给 `farming_manager:AddTileNutrients` 再 `OnApplied` 消耗一份。[索引](../../../raw/current/scripts/tuning.lua#L5854-L5856)、[部署结算](../../../raw/current/scripts/standardcomponents.lua#L1797-L1802)、[消耗](../../../raw/current/scripts/components/fertilizer.lua#L30-L48)

采集册记的是构造态：`hungervalue=0`、`healthvalue=1`、`sanityvalue=0`、`foodtype="RAW"`、`stacksize=40`，与 `fn` 完全一致；它不区分湿态。[采集册](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L1887-L1887)

## 来源与去向

| 环节 | 内容 | 证据 |
|---|---|---|
| 生成条件 | WX-78 的 `digestion` 电路（GAMMA 槽、2 格）激活时给 `eater:SetSpoiledProcessor(true)` 并监听 `oneat`；每吃一份能被处理的腐烂食物 `_num_spoiledfood_eaten` 加 1，达到 `WX78_DIGESTION_SPOILED_NEEDED - (_digestion_modules - 1)`（单电路 5 份，每多一块电路少 1 份）就清计数并 `PushEventImmediate("queue_post_eat_state", { post_eat_state = "wx_bake" })` | [电路与计数](../../../raw/current/scripts/wx78_moduledefs.lua#L1453-L1483)、[基础值](../../../raw/current/scripts/tuning.lua#L9431-L9431)、[2 格与预载](../../../raw/current/scripts/wx78_moduledefs.lua#L1497-L1507) |
| 生成点 | `SGWX78Common.AddWX78BakeState` 的 `wx_bake` 状态：第 46 帧（约 46/30 秒）在角色位置 `SpawnPrefab("wx78_foodbrick")`，骑乘时抬高 2.5、否则 0.25，并给随机水平＋向上初速 | [状态](../../../raw/current/scripts/stategraphs/SGwx78_common.lua#L1121-L1170) |
| 作锅料 | `AddIngredientValues({"wx78_foodbrick"}, {inedible = 1})`：能被放进烹饪锅，但带 `inedible` 标记，能不能落进某道菜取决于该配方的 `test` 是否排斥 `inedible`：`dragonpie` 只要求 `(names.dragonfruit or names.dragonfruit_cooked) and not tags.meat`，所以 1 火龙果＋3 块砖能出火龙果派（可复现样本）；写 `not tags.inedible` 的配方则会排除它。本页未逐个核对全部配方 | [注册](../../../raw/current/scripts/cooking.lua#L173-L173)、[dragonpie](../../../raw/current/scripts/preparedfoods.lua#L128-L131)、[排除例](../../../raw/current/scripts/preparedfoods.lua#L187-L187) |
| 作肥料 | 湿砖可部署，`MakeDeployableFertilizer` 设 `DEPLOYMODE.CUSTOM`、`ondeploy = fertilizer_ondeploy`、`SetUseGridPlacer(false)`、`keep_in_inventory_on_deploy = true`；位置只认 `IsFarmableSoilAtPoint` | [部署组件](../../../raw/current/scripts/standardcomponents.lua#L1812-L1831) |
| 作燃料 | 干湿都能丢进火堆：`fuel.fuelvalue` 再乘 `wetmult`，`fuel_obj:GetIsWet()` 为真时取 `WET_FUEL_PENALTY`＝0.75（90 秒 → 67.5 秒） | [燃料](../../../raw/current/scripts/prefabs/wx78_foodbrick.lua#L153-L154)、[湿惩罚与结算](../../../raw/current/scripts/components/fueled.lua#L205-L212)、[常量](../../../raw/current/scripts/tuning.lua#L1671-L1671) |
| 作肥料研究 | `fertilizerresearchable:SetResearchFn` 返回 `GetFertilizerKey()`（即 `inst.prefab`），植物登记册按这个键记"已知肥料" | [接线](../../../raw/current/scripts/prefabs/wx78_foodbrick.lua#L100-L106)、[候选判定](../../../raw/current/scripts/componentactions.lua#L65-L69) |

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`inventoryitemmoisture` | 两态切换的唯一触发源：`SetOnMoistureDeltaCallback(OnMoistureDeltaCallback)` | 主模拟；由 `inventoryitem` 作为扩展组件挂上 | [[reference/components/inventoryitemmoisture]]；[接线](../../../raw/current/scripts/prefabs/wx78_foodbrick.lua#L144-L146) |
| 组件：`edible` | 干 `RAW`／饥饿 0（刚构造）或 9.375（回调后），都不可食；湿 `GENERIC`／18.75（可食） | 主模拟；`foodtype` 改值会同步 `edible_RAW`／`edible_GENERIC` 标签，而 RAW 不在 `OMNI` 的 `types` 里 | [[reference/components/edible]]；[构造](../../../raw/current/scripts/prefabs/wx78_foodbrick.lua#L156-L160)、[标签同步](../../../raw/current/scripts/components/edible.lua#L9-L16) |
| 组件：`perishable` | 干砖不起表，湿砖起表，凝胶团储藏内停表；到点 `Remove` | 主模拟；`PERISH_ONE_DAY` | [[reference/components/perishable]]；[接线](../../../raw/current/scripts/prefabs/wx78_foodbrick.lua#L162-L164) |
| 组件：`fertilizer` | 湿态添加并写 225／8／0.5／`{0,32,0}`，干态移除 | 仅湿态；`fertilizer` 组件构造时自动加 `fertilizer` 标签 | [[reference/components/fertilizer]]；[添加](../../../raw/current/scripts/prefabs/wx78_foodbrick.lua#L28-L37)、[移除](../../../raw/current/scripts/prefabs/wx78_foodbrick.lua#L53-L53) |
| 组件：`deployable` | 湿态由 `MakeDeployableFertilizer` 挂上，自定义部署到农田 | 仅湿态；两端另有 `_custom_candeploy_fn` 供客户端预判 | [[reference/components/deployable]]；[部署组件](../../../raw/current/scripts/standardcomponents.lua#L1823-L1831) |
| 组件：`fuel` | `MED_LARGE_FUEL`＝90 秒 | 主模拟 | [[reference/components/fuel]]；[接线](../../../raw/current/scripts/prefabs/wx78_foodbrick.lua#L153-L154) |
| 组件：`fertilizerresearchable` | 研究函数返回 `inst.prefab`，供植物登记册记肥料 | 主模拟 | [[reference/components/fertilizerresearchable]]；[接线](../../../raw/current/scripts/prefabs/wx78_foodbrick.lua#L166-L167) |
| 组件：`stackable`／`tradable` | 40 叠；交易值默认 0 | 主模拟；`stackable_CanStackWithFn` 让干湿不可叠 | [[reference/components/stackable]]、[[reference/components/tradable]]；[接线](../../../raw/current/scripts/prefabs/wx78_foodbrick.lua#L148-L151)、[不可叠](../../../raw/current/scripts/prefabs/wx78_foodbrick.lua#L90-L92) |
| 标签：`canpourwateron` | 让 `POUR_WATER` 候选把砖当有效目标 | 两端都有 | [候选](../../../raw/current/scripts/componentactions.lua#L2529-L2533) |
| 标签：`tile_deploy` | 客户端部署候选按它选 `DEPLOY_TILEARRIVE` | 构造时即有；湿态重复添加，干态被移除 | [候选](../../../raw/current/scripts/componentactions.lua#L2090-L2092) |
| SG：`wx_bake` | 只用于生成，砖自身不带 SG | WX-78 服务端状态 | `SGWX78Common`（Wiki 待建）；[状态](../../../raw/current/scripts/stategraphs/SGwx78_common.lua#L1121-L1170) |
| 关联 Prefab：`gelblob_storage` | 湿砖放进去（`parent.prefab` 判定）停表；储藏只收 `edible_GENERIC/MEAT/VEGGIE/SEEDS/BERRY/RAW/GOODIES/MONSTER` 这类带 `edible_*` 标签的物品 | 服务端；干砖 `edible_RAW` 也在接收范围内 | [停表分支](../../../raw/current/scripts/prefabs/wx78_foodbrick.lua#L16-L21)、[接收标签](../../../raw/current/scripts/prefabs/gelblob_storage.lua#L15-L31) |
| 资源：`wx78_foodbrick.zip`／`wx78_foodbrick_wet` | bank／build 同名，另需湿态物品图 | 原版资源，不随页分发 | [assets](../../../raw/current/scripts/prefabs/wx78_foodbrick.lua#L1-L5) |

## 典型接入

### 让湿砖不再因腐烂消失

- 前提与选择：服务端 `modmain.lua` 的 postinit。砖的 `SetOnPerishFn(inst.Remove)` 是"到点删除"，想改成别的处置就在这里换回调；别去动 `TUNING.PERISH_ONE_DAY`。
- 生效与撤销：只在湿态计时，所以回调被调用就说明"湿着放够 1 天（480 秒，按 `perishable` 倍率折算）"。注意回调是在 `Perish()` 已经 `Cancel` 掉 `updatetask` 之后执行的，而 `SetPercent` 只在 `updatetask` 非空时才重启计时，所以回调里必须自己 `StartPerishing()`，否则鲜度回满但永久停表；去掉这层包装即恢复删除。
- 原版依据：回调接线见[赋值](../../../raw/current/scripts/prefabs/wx78_foodbrick.lua#L162-L164)，`Perish()` 的执行顺序见[Perish](../../../raw/current/scripts/components/perishable.lua#L277-L291)。

```lua
-- modmain.lua：营养砖到点不再自删，改成把鲜度回满并继续走表
AddPrefabPostInit("wx78_foodbrick", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    local perishable = inst.components.perishable
    if perishable ~= nil then
        perishable:SetOnPerishFn(function(brick)
            local brick_perishable = brick.components.perishable
            brick_perishable:SetPercent(1)
            brick_perishable:StartPerishing()
        end)
    end
end)
```

- 调整与验证：`SetPercent` 只把剩余时间按总时长重算，计时任务是否重启取决于 `updatetask` 是否为 nil[SetPercent](../../../raw/current/scripts/components/perishable.lua#L223-L233)、[Perish 先取消任务](../../../raw/current/scripts/components/perishable.lua#L277-L285)；回调里任务已被取消，所以示例显式 `StartPerishing()`。验证看湿砖放一天后的鲜度条是否回满且继续走表、以及干砖是否仍然不计时。

## 常见陷阱与限制

- **误区：干砖与湿砖是两件物品。** 只有一个注册，两态是同一实体上的字段与组件增删；因此不能靠 `Prefab` 名区分子类型，要判 `inventoryitem:IsWet()`。[工厂](../../../raw/current/scripts/prefabs/wx78_foodbrick.lua#L9-L56)
- **误区：干砖毕竟是食物，总吃得下去。** `fn` 里 `foodtype` 是 `RAW`，玩家默认 `FOODGROUP.OMNI` 不含它、WX-78 的 `{OMNI, GEARS}` 也不含，所以干砖根本出不来 EAT 候选，更谈不上"0 饥饿但能吃"；它也没有 `fertilizer` 组件。要顶饿或施肥必须先弄湿。[OMNI](../../../raw/current/scripts/constants.lua#L2049-L2063)、[EAT 候选](../../../raw/current/scripts/componentactions.lua#L2672-L2689)[构造](../../../raw/current/scripts/prefabs/wx78_foodbrick.lua#L156-L164)
- **边界：干砖只要湿度有任何变动就会切到 `MakeDry`。** `inventoryitemmoisture` 每次改湿度都调 `onmoisturedeltacallback`（`SetMoisture`／`InheritMoisture`／`MakeMoistureAtLeast` 三条路径都调），`IsWet()` 为假就进 `MakeDry`：饥饿从 `fn` 的 0 变成 `CALORIES_TINY`＝9.375，并移除 `tile_deploy` 标签。所以雨里 0→0.5、还没到 35 的起湿阈值，也会改到字段；"构造即干"与"湿过又干"不是仅有的两种状态。本页未核验改字段后客户端部署候选的实际表现。[回调](../../../raw/current/scripts/components/inventoryitemmoisture.lua#L150-L163)、[异同](../../../raw/current/scripts/prefabs/wx78_foodbrick.lua#L40-L56)、[构造标签](../../../raw/current/scripts/standardcomponents.lua#L1816-L1821)
- **边界：三种干砖都部署不了，但标签状态不同。** `MakeDeployableFertilizerPristine` 在构造时给两端加 `deployable`／`tile_deploy` 并写 `_custom_candeploy_fn`；刚构造的砖两个标签都在、没有组件，deploymode 是 `NONE`，`CanDeploy` 因此返回 false。首次低湿回调（可能从未湿过）会走 `MakeDry`：它 `RemoveTag("tile_deploy")`，而 `deployable` 组件本就不存在，组件不存在时 `RemoveComponent` 跳过该代码块、不触发 `OnRemoveFromEntity`，所以 `deployable` 标签仍在。湿过再干时组件存在，`RemoveComponent` 才触发 `OnRemoveFromEntity`，置 deploymode 为 `NONE`、清 restrictedtag 并 `RemoveTag("deployable")`——三种状态的标签集合都不同，但都出不了部署动作。[候选](../../../raw/current/scripts/componentactions.lua#L2090-L2092)、[replica 判定](../../../raw/current/scripts/components/inventoryitem_replica.lua#L327-L341)、[清理](../../../raw/current/scripts/components/deployable.lua#L55-L62)
- **边界：湿砖放进普通容器不会停表。** 停表只认 `gelblob_storage` 这一个 `parent.prefab`（源码里带 `-- HACK :P` 注释）；背包、箱子只让湿砖继续按 `perishable` 倍率走。[分支](../../../raw/current/scripts/prefabs/wx78_foodbrick.lua#L16-L21)
- **未核验**：未运行游戏。湿润阈值、每秒靠拢速率与 `perishable` 倍率均为源码推导；能匹配哪些"需要 `inedible`"的锅配方、以及 `queue_post_eat_state` 在骑乘／忙碌时的排队行为未逐一核对。

## 验证记录

- 源码审阅：2026-11-09，Steam buildid 26474873。通读 `prefabs/wx78_foodbrick.lua` 全文（176 行）、`prefabs/fertilizer_nutrient_defs.lua` 与 `components/fertilizer.lua` 全文；核对 `components/inventoryitemmoisture.lua` 的目标湿度／回调／`OnLoad`、`components/inventoryitem.lua` 的 `AddMoisture`／`IsWet`、`components/wateryprotection.lua` 与 `componentutil.lua` 的加水链、`componentactions.lua` 的 `POUR_WATER`／`deployable`／`fertilizerresearchable` 候选、`actions.lua` 的 `ACTIONS.POUR_WATER.fn`、`standardcomponents.lua` 的 `MakeDeployableFertilizer*`／`MakeHauntableLaunchAndIgnite`；来源侧核对 `wx78_moduledefs.lua` 的消化电路与计数、`stategraphs/SGwx78_common.lua` 的 `wx_bake`；去向侧核对 `cooking.lua`、`prefabs/gelblob_storage.lua`、`components/inventoryitemholder.lua`、`components/grower.lua`／`pickable.lua`／`crop.lua` 的肥料结算、`components/fueled.lua` 的燃料结算；`tuning.lua` 的时间／热量／鲜度／燃料／养分常量与 `scrapbookdata.lua` 的构造态数值交叉一致。
- 名称核验：`languages/chinese_s.po` 的 `NAMES.WX78_FOODBRICK`＝营养砖、`WET_PREFIX.WX78_FOODBRICK`＝受潮的。
- 未覆盖：消化电路的技能加成细节、`depleted` 电路条表现、湿砖在酸雨中的额外腐烂、以及模组改 `edible` 后 `badfood` 标签的联动。
- 游戏运行：未运行；秒数与倍数均为源码推导（`day_time`＝300 秒）。
- 修订：2026-11-09，初检 P1（`Perish` 回调里必须显式 `StartPerishing`）、P2（湿砖燃料 67.5 秒；干砖任何湿度变动都会 `MakeDry`）——已按源代码更正，见「典型接入」「数值」「来源与去向」「常见陷阱」。
- 修订：2026-11-09，外部复检 P1（干砖 RAW 不在 OMNI 食性内、根本不可食，拆四态）、P1（锅料按配方是否排斥 inedible 而定，补火龙果派）、P2（`RemoveComponent` 连带清 `deployable` 标签；"吃 5 份"限定单电路）——已按源代码更正。
- 修订：2026-11-09，外部复检 P2（部署标签三态统一口径，改写 `RemoveComponent` 的代码细节表述）——已改。
- 结构检查：见交付回执。

## 相关链接

- 湿度与物品：[[reference/components/inventoryitemmoisture]]、[[reference/components/inventoryitem]]、[[reference/components/stackable]]
- 食物与鲜度：[[reference/components/edible]]、[[reference/components/eater]]、[[reference/components/perishable]]
- 肥料与部署：[[reference/components/fertilizer]]、[[reference/components/deployable]]、[[reference/components/fertilizerresearchable]]
- 同批小件：[[reference/prefabs/crumbs]]

返回 [[reference/prefabs/special-food-index|节庆、角色与特殊食品索引]]。
