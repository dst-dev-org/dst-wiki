---
title: "旧鱼族：鱼与鳗鱼（fish、fish_cooked、eel、eel_cooked）"
created: "2026-11-09"
updated: "2026-11-09"
type: prefab
summary: "fish／fish_cooked 与 eel／eel_cooked 四件旧鱼物品：共用工厂下的标签、三维、鲜度、堆叠与交易值，干成小鱼干、烤成熟鱼，`fish` 已从采集册废弃而 `eel` 仍由洞穴池塘的活鳗鱼产出，以及与海鱼、与 `fish` 标签消费端的边界。"
aliases: ["fish", "fish_cooked", "eel", "eel_cooked", "鱼", "熟鱼", "鳗鱼", "熟鳗鱼", "旧鱼", "池塘鱼"]
tags: [prefab, item, food]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/fish.lua", "prefabs/eel.lua", "prefabs/pondfish.lua", "prefabs/pond.lua", "prefablist.lua", "prefabs/forest.lua", "scrapbook_prefabs.lua", "prefabs/spoiledfood.lua", "prefabs/meatrack2.lua", "cooking.lua", "tuning.lua", "components/edible.lua", "components/perishable.lua", "components/stackable.lua", "components/dryable.lua", "components/cookable.lua", "components/tradable.lua", "components/floater.lua", "components/bait.lua", "components/dryingrack.lua", "components/lootdropper.lua", "components/fishable.lua", "components/fishingrod.lua", "stategraphs/SGwilson.lua", "containers.lua", "componentactions.lua", "standardcomponents.lua", "prefabs/pigking.lua", "prefabs/wurt.lua", "prefabs/merm.lua", "prefabs/mermking.lua", "brains/otterbrain.lua", "screens/redux/scrapbookdata.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 旧鱼族：鱼与鳗鱼（fish、fish_cooked、eel、eel_cooked）

## 快速判断

- 职责与适用：`prefabs/fish.lua` 与 `prefabs/eel.lua` 各用一个局部工厂注册生／熟两件，四件都是可拾取食物（`FOODTYPE.MEAT`、`ismeat = true`）且都带 `bait`；两件生品能上晾肉架、也能在带 `cooker` 的实体上烤。生鱼干成 `fishmeat_small_dried`、烤成 `fish_cooked`；生鳗鱼同样干成 `fishmeat_small_dried`、烤成 `eel_cooked`。[鱼注册](../../../raw/current/scripts/prefabs/fish.lua#L168-L174)、[鳗鱼注册](../../../raw/current/scripts/prefabs/eel.lua#L139-L145)
- 优先使用：改数值看[数值与获得](#数值与获得)；改干制产物看[消费端](#消费端)的 `dryable` 行；判断「这条鱼现在还能不能拿到」看[旧鱼族现状与海鱼分界](#旧鱼族现状与海鱼分界)。
- 重要副作用：`fish` 与 `eel` 的腐坏产物**不同**——`fish` 走通用 `spoiled_food`，`eel` 走鱼专用 `spoiled_fish`；干制完成只继承湿度、不继承鲜度（[完成回调](../../../raw/current/scripts/components/dryingrack.lua#L198-L224)）；烤制把新鲜度折成 `1-(1-p)*0.5`（[Cook](../../../raw/current/scripts/components/cookable.lua#L18-L37)）。
- 修改边界：本页只写这四件物品与它们的产出关系。池塘／钓竿设施只给获得入口（[[reference/prefabs/fishing-tools]]）；活鱼 `pondfish`／`pondeel` 见 [[reference/prefabs/freshwater-fish]]，海洋鱼见 [[reference/prefabs/ocean-fish]]；鱼人本体仍待建（源码入口 `prefabs/merm.lua`）；`fishmeat*` 鱼肉链已有页，见 [[reference/prefabs/fishmeat]]（源码入口 `prefabs/meats.lua`）。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `fish` | `fish("fish", "fish01")` 内的 `makefish` 返回 raw／cooked 两函数；`rawfn` 先调 `commonfn("fish01", "idle", false, true, true)`，再在主模拟写三维、改鲜度并挂踢水动画 | 生鱼：`meat`／`catfood`／`pondfish` 三个标签；因两个后置参数为真再补 `dryable`／`cookable` 标签。主模拟依次挂 `edible`、`stackable`（40）、`bait`、`perishable`（`spoiled_food`）、`dryable`、`cookable`、`inspectable`、`inventoryitem`、`MakeHauntableLaunchAndPerish`、`tradable`（1），并先写 `inst.build = "fish01"`。[生鱼](../../../raw/current/scripts/prefabs/fish.lua#L113-L137)、[共用段](../../../raw/current/scripts/prefabs/fish.lua#L22-L98) |
| `fish_cooked` | `cookedfn(build)` 调 `commonfn(build, "cooked")`，`dryable`／`cookable` 缺省 | 只保留 `meat`／`catfood`／`pondfish`：不能再烤、不能上架；主模拟另把 `floater` 改成偏移 0.2、缩放 0.75——这段在早退之后，只有服务端副本生效。[熟鱼](../../../raw/current/scripts/prefabs/fish.lua#L139-L154) |
| `eel` | `eel("eel", "eel01")` 同构；`rawfn` 调 `commonfn("eel01", "idle", true, true, true)`，`MakeInventoryFloatable(inst, "med", 0.05, {1.25, 0.6, 1.25})` 写在 `commonfn` 之外、主模拟早退之前，所以两端都生效 | 生鳗鱼：`meat`／`catfood`／`fishmeat` 三个标签（**没有** `fish`／`pondfish`）；挂载顺序 `edible`、`stackable`（40）、`bait`、`perishable`（`spoiled_fish`）、`dryable`、`cookable`、`MakeHauntableLaunchAndPerish`、`inspectable`、`inventoryitem`、再补一次 `MakeHauntableLaunchAndPerish`、`tradable`（`GOLD_VALUES.RAREMEAT`＝5）；作祟 helper 在共用段被调两次（第 78、84 行），第二次覆盖同一回调，行为无差。[生鳗](../../../raw/current/scripts/prefabs/eel.lua#L93-L110)、[共用段](../../../raw/current/scripts/prefabs/eel.lua#L18-L91) |
| `eel_cooked` | `cookedfn(build)` 调 `commonfn(build, "cooked")`，两个后置参数缺省，再 `MakeInventoryFloatable(inst, "small", 0.1, 0.82)`（同样两端生效） | 只保留三个标签：不能再烤、不能上架；三维与鲜度按熟品档重写。[熟鳗](../../../raw/current/scripts/prefabs/eel.lua#L112-L126) |

两个 `commonfn` 的共同顺序：`AddTransform`／`AddAnimState`／`AddNetwork` → `MakeInventoryPhysics` → bank 与 build（`fish`／`eel`，不是 `fish01`／`eel01`）→ 播 `idle`（鱼不循环、鳗鱼循环）→ 标签 → 条件标签 → `SetPristine` → 主模拟挂件。差别在音效与漂浮：`fish` 的共用段多一个 `AddSoundEmitter`，`rawfn` 用它在落水／上岸时播 `fishingpole_fishland`；`eel` 没有音效组件，也不播这些声音。[鱼的音效](../../../raw/current/scripts/prefabs/fish.lua#L24)、[落水声](../../../raw/current/scripts/prefabs/fish.lua#L100-L111)。`inst.build` 存的是 `fish01`／`eel01`，注释说它给 SGwilson 用：钓竿把中鱼事件连 `build` 一起发出去，`catchfish` 状态用它替换手里的 `fish01` 图样。[鱼的自赋值](../../../raw/current/scripts/prefabs/fish.lua#L59)、[鳗鱼的自赋值](../../../raw/current/scripts/prefabs/eel.lua#L50)、[事件源](../../../raw/current/scripts/components/fishingrod.lua#L181-L182)、[消费](../../../raw/current/scripts/stategraphs/SGwilson.lua#L6325-L6347)

「生鱼／生鳗鱼」是主模拟里再改一次鲜度的写法：`commonfn` 先设 `PERISH_FAST` 并 `StartPerishing`，`rawfn` 随后又 `SetPerishTime(PERISH_SUPERFAST)`；`SetPerishTime` 会重置剩余时间，所以最终生效的是 3 天。[鱼的第二次设置](../../../raw/current/scripts/prefabs/fish.lua#L70-L73)、[鳗鱼的第二次设置](../../../raw/current/scripts/prefabs/eel.lua#L61-L64)、[重置语义](../../../raw/current/scripts/components/perishable.lua#L188-L194)

### 名称核验

`fish` 的简中实显是**鱼**（Fish），`fish_cooked` 是**熟鱼**（Cooked Fish），`eel` 是**鳗鱼**（Eel），`eel_cooked` 是**熟鳗鱼**（Cooked Eel）；四个键各命中一次，注册名与 `STRINGS.NAMES` 键同名。[FISH](../../../raw/current/scripts/languages/chinese_s.po#L342825-L342827)、[FISH_COOKED](../../../raw/current/scripts/languages/chinese_s.po#L342885-L342887)、[EEL](../../../raw/current/scripts/languages/chinese_s.po#L342440-L342442)、[EEL_COOKED](../../../raw/current/scripts/languages/chinese_s.po#L342445-L342447)

## 数值与获得

时间基准：`seg_time`＝30 秒、`total_day_time`＝480 秒、`perish_warp`＝1，鲜度与干制都按 480 秒／天换算。[基准](../../../raw/current/scripts/tuning.lua#L19-L20)、[`perish_warp`](../../../raw/current/scripts/tuning.lua#L47)

| 注册 | 饥饿／生命／理智 | 鲜度 | 堆叠 | 漂浮 | 交易值 | 转换 |
|---|---|---|---|---|---|---|
| `fish` | 12.5／1／0 | `PERISH_SUPERFAST`＝3 天＝1440 秒 | 40 | `small`，零偏移（两端） | 1 | 干制 `DRY_FAST`＝1 天＝480 秒 → `fishmeat_small_dried`；烤制 → `fish_cooked`；腐坏 → `spoiled_food` |
| `fish_cooked` | 12.5／1／0 | `PERISH_FAST`＝6 天＝2880 秒 | 40 | `small`，偏移 0.2、缩放 0.75（仅服务端副本） | 1 | 无（无 `dryable`／`cookable`） |
| `eel` | 9.375／3／0 | 3 天＝1440 秒 | 40 | `med`、偏移 0.05、缩放 `{1.25, 0.6, 1.25}`（两端） | 5 | 干制 1 天 → `fishmeat_small_dried`；烤制 → `eel_cooked`；腐坏 → `spoiled_fish` |
| `eel_cooked` | 12.5／8／0 | 6 天＝2880 秒 | 40 | `small`、偏移 0.1、缩放 0.82（两端） | 5 | 无 |

数值来源：[生鱼](../../../raw/current/scripts/prefabs/fish.lua#L120-L122)、[熟鱼](../../../raw/current/scripts/prefabs/fish.lua#L146-L148)、[生鳗](../../../raw/current/scripts/prefabs/eel.lua#L101-L103)、[熟鳗](../../../raw/current/scripts/prefabs/eel.lua#L121-L123)、[常量](../../../raw/current/scripts/tuning.lua#L2021-L2030)、[热量](../../../raw/current/scripts/tuning.lua#L2058-L2065)、[鲜度](../../../raw/current/scripts/tuning.lua#L2043-L2051)、[干制](../../../raw/current/scripts/tuning.lua#L2053-L2056)、[交易值](../../../raw/current/scripts/tuning.lua#L1946-L1952)、[堆叠](../../../raw/current/scripts/tuning.lua#L82-L83)。采集册只收了 `eel` 两件（`fish` 两件被注释），其条目写作 `stacksize=40`、`hungervalue=9.375/12.5`、`healthvalue=3/8`、`perishable=1440/2880`，与源码一致。[采集册](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L457-L458)

### 来源

| 注册 | 已核验来源 | 说明与证据 |
|---|---|---|
| `eel` | 洞穴池塘钓上的活鳗鱼 `pondeel`：被击杀时掉落、在背包里腐坏时替换，两条都产出 `eel` | 数据表 `loot` 与 `perish_product` 都写 `eel`；洞穴池塘是唯一挂 `pondeel` 的鱼群。[数据](../../../raw/current/scripts/prefabs/pondfish.lua#L178-L190)、[鱼群](../../../raw/current/scripts/prefabs/pond.lua#L480) |
| `eel_cooked` | ①`cookable.product` 指向：在带 `cooker` 且未 `fueldepleted` 的实体上烤生鳗鱼（`dangerouscooker` 须 `doer:HasTag("expertchef")`；骑乘时只有烹饪器属于自己才可烤——本页走食材对烹饪器的 `cookable` 分支，骑乘门禁看目标，另一条 `cooker` 分支才检查食材归属，[cookable 分支](../../../raw/current/scripts/componentactions.lua#L1096-L1105)、[cooker 分支](../../../raw/current/scripts/componentactions.lua#L1113-L1126)）；②活鳗鱼 `pondeel` 自己的 `cookable_product` 直接指向 `eel_cooked`，不经过生鳗鱼；③掉落者满足燃烧条件时 `DropLoot` 把 `eel` 换成 `eel_cooked` | ②是「钓到就下锅」的捷径；③对活鳗鱼不可达（它没有 `burnable`）。[①](../../../raw/current/scripts/prefabs/eel.lua#L73-L76)、[②](../../../raw/current/scripts/prefabs/pondfish.lua#L184)、[③](../../../raw/current/scripts/components/lootdropper.lua#L424-L431)、[烤制门槛](../../../raw/current/scripts/componentactions.lua#L1096-L1105) |
| `fish` | 快照内**没有**活体来源：地表池塘的鱼群挂的是 `pondfish`，它的 `loot` 与 `perish_product` 是 `fishmeat_small`，不是 `fish` | 全 scripts 检索 `SpawnPrefab("fish")` 与 `SpawnPrefab("fish_cooked")` 均 0 命中。[地表鱼群](../../../raw/current/scripts/prefabs/pond.lua#L329)、[地表鱼数据](../../../raw/current/scripts/prefabs/pondfish.lua#L161-L176) |
| `fish_cooked` | 只剩 `cookable.product` 这一条接线；再没有产出 `fish` 的路径，所以正常玩法里也拿不到 | 同上 |

钓上来的活鱼是实体（有 `inventoryitem`、`weighable`、`lootdropper`），不是本页四件物品；`fishable` 在中鱼时 `SpawnPrefab` 的是鱼群里登记的 prefab，并用 `AddChild` 把它挂在池塘下面。[中鱼](../../../raw/current/scripts/components/fishable.lua#L58-L80)、[池塘登记](../../../raw/current/scripts/prefabs/pond.lua#L363)

### 消费端

- 烹饪锅：`fish` 记 `meat = 1`、`fish = 1`；`eel` 记 `meat = 0.5`、`fish = 1`；两者都 `cancook`，因此各派生一份 `fish_cooked`／`eel_cooked` 的食材条目（原值加 `precook = 1`），熟鱼、熟鳗鱼都能进锅但不能回炉。活鱼另有一套：`pondeel` 是 `meat = 0.5`、`fish = 1` 且 `cancook`，`pondfish` 是 `meat = 0.5`、`fish = 0.5` 且**不** `cancook`。[食材值](../../../raw/current/scripts/cooking.lua#L111-L115)、[派生](../../../raw/current/scripts/cooking.lua#L46-L65)、[槽位判定](../../../raw/current/scripts/containers.lua#L373-L375)
- 晾肉架：四件里只有两件生品带 `dryable`，产物同为 `fishmeat_small_dried`、时长 `DRY_FAST`，且都显式写了 `SetDriedBuildFile("meat_rack_food_tot")`——架面用鱼干专属 build，不像[[reference/prefabs/froglegs|蛙腿]]那样回落到 `meat_rack_food`。[鱼的接线](../../../raw/current/scripts/prefabs/fish.lua#L75-L80)、[鳗鱼的接线](../../../raw/current/scripts/prefabs/eel.lua#L66-L71)、[默认 build](../../../raw/current/scripts/components/dryingrack.lua#L62-L76)
- 交易：`fish`／`fish_cooked` 是 `GOLD_VALUES.MEAT`＝1，`eel`／`eel_cooked` 是 `GOLD_VALUES.RAREMEAT`＝5，四件都过猪王的 `goldvalue > 0` 门槛（与蛙腿的 0 不同）；但 `AcceptTest` 会先拒带 `merm` 标签的交付者（`pig_token` 除外；沃特带该标签，[标签](../../../raw/current/scripts/prefabs/wurt.lua#L405)），沃特交付时这四件都不换金。[赋值](../../../raw/current/scripts/prefabs/eel.lua#L86-L88)、[接收判定](../../../raw/current/scripts/prefabs/pigking.lua#L598-L606)
- 腐坏产物：`eel`／`eel_cooked` 走 `spoiled_fish`，`fish`／`fish_cooked` 走通用的 `spoiled_food`；两者别混写。锤 `spoiled_fish` 时每消耗一件**固定**掉 1 份 `spoiled_food` 与 1 块 `boneshard`——它的战利品是 `SetLoot({"spoiled_food", "boneshard"})`，`numrandomloot = 1` 但 `randomloot` 为空：`GenerateLoot` 的随机段取不到东西，固定段两件照给（[战利品](../../../raw/current/scripts/prefabs/spoiledfood.lua#L29)、[初始化](../../../raw/current/scripts/prefabs/spoiledfood.lua#L224-L230)、[锤击](../../../raw/current/scripts/prefabs/spoiledfood.lua#L33-L52)、[固定段](../../../raw/current/scripts/components/lootdropper.lua#L236-L240)）。真正「二选一」的是 `spoiled_fish_small`：两条 `AddRandomLoot` 加 `numrandomloot = 1`（[小鱼腐坏](../../../raw/current/scripts/prefabs/spoiledfood.lua#L247-L253)）。[鳗鱼](../../../raw/current/scripts/prefabs/eel.lua#L61-L64)、[鱼](../../../raw/current/scripts/prefabs/fish.lua#L70-L73)、[`spoiled_fish` 注册](../../../raw/current/scripts/prefabs/spoiledfood.lua#L263-L265)

## 旧鱼族现状与海鱼分界

- **`fish`／`fish_cooked` 已废弃。** 采集册清单把它们注释掉并标了 `Deprecated prefab`，而 `eel`／`eel_cooked` 仍在清单里；两者仍由 `PREFABFILES` 加载，地表世界 prefab 列表也留着 `fish`，注释写明是给模组兼容用——所以「还能加载」不等于「还能获得」。[废弃](../../../raw/current/scripts/scrapbook_prefabs.lua#L385)、[熟鱼废弃](../../../raw/current/scripts/scrapbook_prefabs.lua#L842)、[鳗鱼在册](../../../raw/current/scripts/scrapbook_prefabs.lua#L340)、[文件清单](../../../raw/current/scripts/prefablist.lua#L438)、[地表预载](../../../raw/current/scripts/prefabs/forest.lua#L327)
- **本页四件都没有 `fish` 标签。** `fish` 带的是 `pondfish`，`eel` 带的是 `fishmeat`；而 `fish` 标签的消费端覆盖沃特的鱼保鲜（`WURT_FISH_PRESERVER_RATE`＝1/4）、鱼人欢呼与鱼人王接受等。所以这些奖励与减免**不作用于**本页物品，而作用于活鱼 `pondfish`／`pondeel` 与海洋鱼。[鱼的标签](../../../raw/current/scripts/prefabs/fish.lua#L35-L37)、[鳗鱼的标签](../../../raw/current/scripts/prefabs/eel.lua#L31-L33)、[沃特](../../../raw/current/scripts/prefabs/wurt.lua#L370-L372)、[鱼人](../../../raw/current/scripts/prefabs/merm.lua#L342)、[鱼人王](../../../raw/current/scripts/prefabs/mermking.lua#L209)
- **与海鱼的分界。** 海鱼是 `oceanfish.lua` 一族（`oceanfish_*` 与它们的 `_inv` 背包形态），走海钓竿／拖网，食材值按 `oceanfishdef` 逐条登记；本页四件是池塘时代的旧淡水鱼物品。鱼肉两链见 [[reference/prefabs/fishmeat]]（B07 的海鱼页只覆盖 `oceanfish_*` 实体），本页只保留「干制产物指向 `fishmeat_small_dried`」这一层关系。[海鱼食材登记](../../../raw/current/scripts/cooking.lua#L118-L123)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：edible | 四件都写 `ismeat = true`、`FOODTYPE.MEAT` 与三维；理智一律 0 | 仅主模拟 | [[reference/components/edible#关键接口]]；[鱼](../../../raw/current/scripts/prefabs/fish.lua#L61-L63)、[鳗鱼](../../../raw/current/scripts/prefabs/eel.lua#L52-L54) |
| 组件：perishable | `SetPerishTime` 加 `StartPerishing`；腐坏替换鱼用 `spoiled_food`、鳗鱼用 `spoiled_fish` | 生品 1440 秒、熟品 2880 秒；生品在 `rawfn` 里改第二次 | [[reference/components/perishable#关键接口]]；[鱼](../../../raw/current/scripts/prefabs/fish.lua#L70-L73)、[鳗鱼](../../../raw/current/scripts/prefabs/eel.lua#L61-L64) |
| 组件：dryable | `product=fishmeat_small_dried`、`time=DRY_FAST`、`dried_buildfile=meat_rack_food_tot` | 仅两件生品；标签两端、组件主模拟 | [[reference/components/dryable#关键接口]]；[鱼](../../../raw/current/scripts/prefabs/fish.lua#L75-L80)、[鳗鱼](../../../raw/current/scripts/prefabs/eel.lua#L66-L71) |
| 组件：cookable | `product` 分别是 `fish_cooked`／`eel_cooked`；`Cook()` 折算鲜度 | 仅两件生品 | [[reference/components/cookable#关键接口]]；[鱼](../../../raw/current/scripts/prefabs/fish.lua#L82-L85)、[鳗鱼](../../../raw/current/scripts/prefabs/eel.lua#L73-L76) |
| 组件：stackable | 四件都写 `maxsize = STACK_SIZE_SMALLITEM`＝40 | 两端由 replica 同步 | [[reference/components/stackable#关键接口]]；[鱼](../../../raw/current/scripts/prefabs/fish.lua#L65-L66)、[鳗鱼](../../../raw/current/scripts/prefabs/eel.lua#L56-L57) |
| 组件：tradable | 鱼 1、鳗鱼 5（`GOLD_VALUES.RAREMEAT`） | 主模拟；四件都过猪王门槛 | [[reference/components/tradable#关键接口]]；[鱼](../../../raw/current/scripts/prefabs/fish.lua#L93-L95)、[鳗鱼](../../../raw/current/scripts/prefabs/eel.lua#L86-L88) |
| 组件：floater | 鱼默认、熟鱼改偏移／缩放、鳗鱼 `med` 加三轴缩放 | 鱼熟态改参在主模拟之后（仅服务端）；鳗鱼两态在主模拟之前（两端） | [[reference/components/floater#关键接口]]；[鱼熟态](../../../raw/current/scripts/prefabs/fish.lua#L150-L151)、[鳗鱼生](../../../raw/current/scripts/prefabs/eel.lua#L95) |
| 组件：bait／hauntable | `bait` 让鱼能当陷阱饵；`MakeHauntableLaunchAndPerish` 作祟只抛飞 | 加速腐烂分支被注释；鳗鱼共用段调了两次 | [[reference/components/bait#关键接口]]；[作祟](../../../raw/current/scripts/standardcomponents.lua#L1311-L1337)、[重复调用](../../../raw/current/scripts/prefabs/eel.lua#L84) |
| 标签：`pondfish`／`fishmeat` | `pondfish` 在快照内没有 `HasTag("pondfish")` 消费者；`fishmeat` 被 `otter` 的 Brain 当 `NOT_TOSSABLE_TAGS` 读取，带该标签的物品不会被它抛着玩 | 两端 | [检索范围](#验证记录)、[水獭](../../../raw/current/scripts/brains/otterbrain.lua#L262-L271) |
| 关联 Prefab：`fishmeat_small_dried`／`spoiled_food`／`spoiled_fish` | 干制产物与两种腐坏产物 | 到点由 `OnDoneDrying` 与 `Perishable:Perish` 生成 | [[reference/prefabs/fishmeat]]；[干制](../../../raw/current/scripts/components/dryingrack.lua#L198-L224) |
| 资源：`anim/fish.zip`／`anim/eel.zip` | bank 与 build 分别是 `fish`／`eel`，四件共用两套资源 | 原版资源，不随页分发 | [鱼 assets](../../../raw/current/scripts/prefabs/fish.lua#L1-L5)、[鳗鱼 assets](../../../raw/current/scripts/prefabs/eel.lua#L1-L5) |

## 典型接入

### 让鳗鱼也算「鱼」，吃沃特的鱼保鲜

- 前提与选择：服务端 `modmain.lua` 的 postinit。沃特的鱼保鲜是 `preserver` 的速率函数按 `HasTag("fish")` 判定的，补标签即可，不必改 `perishable`。
- 生效与撤销：标签随 prefab 构造重加，旧存档读入也生效；卸模组后回到常规腐坏速率。
- 重要副作用：`fish` 标签同时被鱼人欢呼与鱼人王接受读取，补标签会让鳗鱼在这些地方也被当成鱼；只想改腐坏速率就别走这条路，改成给 `perishable` 单独加倍率。
- 原版依据：速率函数返回 `nil` 时 `preserver` 回落到 1，返回数值时按该倍数结算。[速率](../../../raw/current/scripts/prefabs/wurt.lua#L370-L372)、[`or 1`](../../../raw/current/scripts/components/preserver.lua#L12-L14)、[常量](../../../raw/current/scripts/tuning.lua#L4689)

```lua
-- modmain.lua：让鳗鱼带 fish 标签（沃特的鱼保鲜按 1/4 速率腐坏）
AddPrefabPostInit("eel", function(inst)
    inst:AddTag("fish")
end)
```

- 调整与验证：`WURT_FISH_PRESERVER_RATE`＝1/4；实测把一条鳗鱼放沃特背包里等一天，看剩余鲜度是否约为常规的四分之一，并确认鱼人王的接受列表也一起变了。

## 常见陷阱与限制

- **误区：池塘钓上来的就是 `fish`。** 地表池塘挂的是 `pondfish`（淡水鱼），它的掉落与腐坏产物都是 `fishmeat_small`；洞穴池塘挂 `pondeel`（活鳗鱼），才产出 `eel`。[池塘](../../../raw/current/scripts/prefabs/pond.lua#L329)、[洞穴池塘](../../../raw/current/scripts/prefabs/pond.lua#L480)、[淡水鱼数据](../../../raw/current/scripts/prefabs/pondfish.lua#L161-L176)
- **误区：四件的腐坏产物都是 `spoiled_food`。** 只有 `fish` 两件是；`eel` 两件是 `spoiled_fish`。这条差异写在各自的 `commonfn` 里，不是共用默认值。[鱼](../../../raw/current/scripts/prefabs/fish.lua#L73)、[鳗鱼](../../../raw/current/scripts/prefabs/eel.lua#L64)
- **边界：熟鱼／熟鳗鱼不能再烤、不能上架。** 二者都没有 `cookable`／`dryable` 标签，动作层与肉架槽位测试都会拒绝；进锅可以，但只算 `precook` 食材。[槽位测试](../../../raw/current/scripts/containers.lua#L234-L244)、[烤制门槛](../../../raw/current/scripts/componentactions.lua#L1096-L1105)
- **边界：晾晒暂停与烧架。** 正常晾晒时容器腐坏速率 0；**酸雨**（`isinacid`）先命中酸雨分支、恒为 `1 + rate`（与是否暂停无关）；只有**普通雨**暂停时才回落到 1（正常腐坏）；架子被烧时 `InstantDry` 立即出产物、不继承湿度与鲜度。[速率](../../../raw/current/scripts/components/dryingrack.lua#L26-L38)、[InstantDry](../../../raw/current/scripts/components/dryingrack.lua#L376-L391)、[完成回调](../../../raw/current/scripts/components/dryingrack.lua#L198-L224)
- **边界：鱼干不走默认架面 build。** 两件生鱼显式写了 `meat_rack_food_tot`，而 `dryable` 的通用默认是 `meat_rack_food`；复刻时漏掉这行会显示成肉干贴图。[接线](../../../raw/current/scripts/prefabs/fish.lua#L79)、[默认](../../../raw/current/scripts/components/dryingrack.lua#L62-L76)
- **未核验**：本页未运行游戏；「`fish` 无活体来源」只按源码检索与采集册注释判断，未穷举模组与调试命令路径。

## 验证记录

- 源码审阅：2026-11-09，Steam buildid 26474873。通读 `prefabs/fish.lua` 与 `prefabs/eel.lua` 的 `commonfn`、两个生／熟工厂与两个注册工厂；核对 `prefabs/pondfish.lua` 的两张数据表与 `prefabs/pond.lua` 的三处 `AddFish`；`components/fishable.lua` 的中鱼流程；`edible`／`perishable`／`stackable`／`dryable`／`cookable`／`tradable`／`floater`／`bait` 分支与 `dryingrack`、`preserver`；`cooking.lua` 的食材值与 `cancook` 派生；`scrapbook_prefabs.lua`、`prefablist.lua`、`prefabs/forest.lua` 的废弃与预载状态；`wurt`／`merm`／`mermking` 的 `fish` 标签消费；`scrapbookdata` 交叉核对。
  检索范围：全 `scripts` 的 `*.lua` 中 `SpawnPrefab("fish")`、`SpawnPrefab("fish_cooked")` 均 0 命中；`HasTag("pondfish")` 0 命中；`HasTag("fish")` 命中 7 处，全部只按标签判定。
  名称核验：`languages/chinese_s.po` 的 `NAMES.FISH`＝鱼、`NAMES.FISH_COOKED`＝熟鱼、`NAMES.EEL`＝鳗鱼、`NAMES.EEL_COOKED`＝熟鳗鱼。
  未覆盖：鱼人与池塘本体；海鱼完整数值见 [[reference/prefabs/ocean-fish]]；模组注入与调试命令生成路径。`fishmeat*` 两链的数值见 [[reference/prefabs/fishmeat]]。
- 游戏运行：未运行。天数、秒数与三维均为 tuning 推导（`total_day_time`＝480 秒）。
- 结构检查：`validate_page.py` 通过（standard 档内）；`validate_source_links.py --strict` 0 错 0 警。

## 相关链接

- 可食与鲜度：[[reference/components/edible]]、[[reference/components/perishable]]
- 两条转换：[[reference/components/dryable]]、[[reference/components/cookable]]、[[reference/components/stackable]]
- 获得设施：[[reference/prefabs/fishing-tools]]、[[reference/components/fishable]]
- 邻近产物链：[[reference/prefabs/fishmeat]]、[[reference/prefabs/smallmeat]]

返回 [[reference/prefabs/index|Prefab 索引]]。