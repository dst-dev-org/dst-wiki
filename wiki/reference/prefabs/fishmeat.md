---
title: "鱼肉两链六注册（生鱼肉 fishmeat 与小鱼块 fishmeat_small）"
created: "2026-11-09"
updated: "2026-11-09"
type: prefab
summary: "大小两条鱼肉链的生／熟／干六件对照：三维、鲜度（生品 3 天，腐坏成鱼专用 spoiled_fish／spoiled_fish_small）、标签与堆叠，烤制继承鲜度、晾晒时长与架子贴图 build，以及海洋鱼与一角鲸／岩石大白鲨掉落、烹饪食材与沃特随身理智等消费面。"
aliases: ["fishmeat", "fishmeat_cooked", "fishmeat_dried", "fishmeat_small", "fishmeat_small_cooked", "fishmeat_small_dried", "生鱼肉", "鱼排", "鱼肉干", "小鱼块", "熟小鱼块", "小鱼干", "鱼肉"]
tags: [prefab, item, food]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/meats.lua", "prefablist.lua", "components/edible.lua", "components/perishable.lua", "components/dryable.lua", "components/dryingrack.lua", "components/cookable.lua", "components/stackable.lua", "prefabs/meatrack2.lua", "prefabs/spoiledfood.lua", "prefabs/oceanfishdef.lua", "prefabs/gnarwail.lua", "prefabs/shark.lua", "prefabs/lightcrab.lua", "prefabs/fish.lua", "prefabs/eel.lua", "prefabs/pondfish.lua", "prefabs/pond.lua", "prefabs/mermhouse.lua", "components/container.lua", "components/piratespawner.lua", "components/lootdropper.lua", "components/eater.lua", "cooking.lua", "prefabs/wurt.lua", "components/itemaffinity.lua", "prefabs/catcoon.lua", "prefabs/lureplant.lua", "componentactions.lua", "actions.lua", "tuning.lua", "languages/chinese_s.po", "screens/redux/scrapbookdata.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 鱼肉两链六注册（生鱼肉 fishmeat 与小鱼块 fishmeat_small）

## 快速判断

- 职责与适用：六件都是可拾取的肉食物品，两条链各三态（生／熟／干），全部由同一个 `common()` 工厂（`prefabs/meats.lua`）构造：链间只差 bank／build／动画、标签表与干制数据，链内三态只差初始化赋值。查「吃多少／放几天／能不能晾或烤」先看本页。[工厂](../../../raw/current/scripts/prefabs/meats.lua#L138-L222)
- 与常规肉链的关键差异：生品 **3 天**（常规生肉 6 天）、腐坏产物鱼专用 **`spoiled_fish`／`spoiled_fish_small`**、干品 build 为 **`meat_rack_food_tot`**、小鱼块生品无 `rawmeat`。见[与常规肉链的差异](#与常规肉链的差异)。
- 优先使用：三维／鲜度／堆叠看[六件对照](#六件对照)；熟制与干制看[两条转换链](#两条转换链)；从哪来、能下什么锅看[获得与消费](#获得与消费)；动手前先看[常见陷阱](#常见陷阱与限制)。
- 重要副作用：烤制继承鲜度（腐坏量减半）、晾晒整件替换为干品并留在原格；干品带 `lureplant_bait`、无 `dryable`／`cookable`，不能再晾也不能烤（详见[陷阱](#常见陷阱与限制)）。
- 修改边界：只覆盖这六件物品。世界态鱼（`oceanfish_*`、`pondfish`／`fish`、`eel`）的实体行为与获得流程见 [[reference/prefabs/ocean-fish]] 与 [[reference/prefabs/freshwater-fish]]（旧鱼见 [[reference/prefabs/pond-fish]]），本页只留产出关系；`spoiled_fish*` 的肥料／锤击／烹饪细节只给出口。

## 注册与构造

`prefablist.lua` 的 `PREFABFILES` 里列了 `"meats"`，整文件加载后由表尾 `return Prefab(...)` 逐个注册；六件在同一个 return 里；只有两个生品行传了 `prefabs` 依赖表（`fishmeat_small` 与 `fishmeat` 各声明熟品、干品与腐坏产物），其余四行不传。[文件注册](../../../raw/current/scripts/prefablist.lua#L739)、[六件导出](../../../raw/current/scripts/prefabs/meats.lua#L914-L919)、[依赖表](../../../raw/current/scripts/prefabs/meats.lua#L66-L78)

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `fishmeat` | `common("fishmeat","fishmeat","raw",{"fishmeat","catfood","rawmeat"}, dryable, cookable)` → `ismastersim` 之后写三维、改鲜度与腐坏产物 | 大鱼生品；生品里唯一同时带 `fishmeat`＋`catfood`＋`rawmeat` 的一件。[定义](../../../raw/current/scripts/prefabs/meats.lua#L552-L566) |
| `fishmeat_cooked` | `common("fishmeat","fishmeat","cooked",{"fishmeat"})`，无干制／烤制数据 | 与生品同 build、换 `cooked` 动画；三维不变，只拉长鲜度。[定义](../../../raw/current/scripts/prefabs/meats.lua#L568-L582) |
| `fishmeat_dried` | `common("meat_rack_food_tot","meat_rack_food_tot","dried_fishmeat",nil,{isdried=true})` | 干品改用架子 build；只在主模拟写三维与鲜度。[定义](../../../raw/current/scripts/prefabs/meats.lua#L584-L598) |
| `fishmeat_small` | `common("fishmeat_small","fishmeat_small","raw",{"fishmeat","catfood"}, dryable, cookable)` | 小鱼块生品；**无 `rawmeat`**，干制时长比大鱼短一半。[定义](../../../raw/current/scripts/prefabs/meats.lua#L500-L514) |
| `fishmeat_small_cooked` | `common("fishmeat_small","fishmeat_small","cooked",{"fishmeat"})` | 三维与生品相同，鲜度翻倍；浮水偏移 0.2、缩放 0.75。[定义](../../../raw/current/scripts/prefabs/meats.lua#L516-L533) |
| `fishmeat_small_dried` | `common("meat_rack_food_tot","meat_rack_food_tot","dried_fishmeat_small",nil,{isdried=true})` | **小鱼链内**三维最高的一档（生命 +15、理智 +20；大鱼干是 +30／+15）。[定义](../../../raw/current/scripts/prefabs/meats.lua#L535-L550) |

`common()` 里与这六件相关的接线，按执行顺序：

- pristine 之前（服务端与客户端都执行）：`SetBank`／`SetBuild`／`PlayAnimation` 并记 `scrapbook_anim`；加基础 `meat` 标签再逐个追加传入标签；`dryable ~= nil` 时**只有 `dryable.product` 非空才加 `dryable` 标签**，但无论如何都加 `lureplant_bait`；`cookable ~= nil` 时加 `cookable` 标签；`MakeInventoryFloatable`。[pristine 前](../../../raw/current/scripts/prefabs/meats.lua#L145-L176)
- 主模拟：`edible`（`ismeat = true`、`foodtype = FOODTYPE.MEAT`）→ `bait` → `inspectable` → `inventoryitem`＋`stackable`（不写 `maxsize`，落组件默认 20）→ `tradable`（`GOLD_VALUES.MEAT`）→ `perishable`（先给默认 `PERISH_FAST`、`StartPerishing()`、`onperishreplacement = "spoiled_food"`，随后各变体覆盖）→ 可选 `dryable`（`SetProduct`／`SetDryTime`／`SetBuildFile`／`SetDriedBuildFile`）→ 可选 `cookable`（写 `product`）→ `MakeHauntableLaunchAndPerish` 并监听 `spawnedfromhaunt` 抛飞。[主模拟接线](../../../raw/current/scripts/prefabs/meats.lua#L182-L221)、[堆叠默认值](../../../raw/current/scripts/components/stackable.lua#L23-L29)
- 干品的 `{ isdried = true }` 落在 `common()` 的**第 5 个参数**（`dryable`）位置，而 `common()` 只读 `dryable.product`：该字段在本文件内没有任何读取点，实际效果是「加了 `lureplant_bait`、没有 `dryable` 标签与组件」。[干品调用](../../../raw/current/scripts/prefabs/meats.lua#L536)、[读取点](../../../raw/current/scripts/prefabs/meats.lua#L161-L172)
- 资源：两条链共用 `anim/fishmeat.zip`、`anim/fishmeat_small.zip` 与 `anim/meat_rack_food_tot.zip`（干品）。[assets](../../../raw/current/scripts/prefabs/meats.lua#L1-L15)

## 六件对照

1 天 = 16 × 30 = 480 秒（`seg_time = 30`、`perish_warp = 1`）。[时间基准](../../../raw/current/scripts/tuning.lua#L19-L20)、[倍率](../../../raw/current/scripts/tuning.lua#L41-L47)

| 注册 | 生命 | 饥饿 | 理智 | 鲜度 | 腐坏产物 | 堆叠 | 额外标签（除 `meat`） |
|---|---:|---:|---:|---|---|---:|---|
| `fishmeat` | +8（`HEALING_MEDSMALL`） | +25（`CALORIES_MED`） | 0（未写，组件默认） | 3 天（`PERISH_SUPERFAST`） | `spoiled_fish` | 20 | `fishmeat`／`catfood`／`rawmeat`／`dryable`／`cookable`／`lureplant_bait` |
| `fishmeat_cooked` | +8 | +25 | 0 | 6 天（`PERISH_FAST`） | `spoiled_fish` | 20 | `fishmeat` |
| `fishmeat_dried` | +30（`HEALING_MEDLARGE`） | +25 | +15（`SANITY_MED`） | 15 天（`PERISH_SLOW`） | `spoiled_fish` | 20 | `lureplant_bait` |
| `fishmeat_small` | +1（`HEALING_TINY`） | +12.5（`CALORIES_SMALL`） | 0 | 3 天 | `spoiled_fish_small` | 20 | `fishmeat`／`catfood`／`dryable`／`cookable`／`lureplant_bait` |
| `fishmeat_small_cooked` | +1 | +12.5 | 0 | 6 天 | `spoiled_fish_small` | 20 | `fishmeat` |
| `fishmeat_small_dried` | +15（`HEALING_MOREMEDSMALL`） | +12.5 | +20（`SANITY_MEDLARGE`） | 15 天 | `spoiled_fish_small` | 20 | `lureplant_bait` |

- 「理智 0」只属于四件生／熟品：它们的初始化没写 `sanityvalue`，落 `edible` 组件默认 0；两件干品是**显式赋值**——`fishmeat_dried` +15（`SANITY_MED`）、`fishmeat_small_dried` +20（`SANITY_MEDLARGE`）。`oncheckbadfood` 只在 `healthvalue < 0` 或 `sanityvalue < 0` 时打 `badfood`，本页六件都不带。[组件默认值](../../../raw/current/scripts/components/edible.lua#L19-L24)、[大鱼干理智](../../../raw/current/scripts/prefabs/meats.lua#L593)、[小鱼干理智](../../../raw/current/scripts/prefabs/meats.lua#L544)、[badfood 判定](../../../raw/current/scripts/components/edible.lua#L1-L7)
- 常量：三维 1／8／15／30、饥饿 12.5／25、理智 15／20；鲜度 3／6／15 天、干制 0.5／1 天、堆叠 20。[数值](../../../raw/current/scripts/tuning.lua#L2021-L2061)、[堆叠](../../../raw/current/scripts/tuning.lua#L81-L85)
- 腐坏发生在整叠单位上：`Perishable:Perish()` 生成 `onperishreplacement`、按原堆叠数（必要时临时放开上限）同步，并把原槽位传给 `holder:GiveItem(goop, slot)`——**优先**放回原持有者的原格；容器仍按 `itemtestfn`／`CanTakeItemInSlot` 检查，拒收时走 `GiveItem` 失败分支（`drop_on_fail` 默认真），落到持有者所在位置。[替换](../../../raw/current/scripts/components/perishable.lua#L293-L320)、[GiveItem 失败分支](../../../raw/current/scripts/components/container.lua#L557-L565)
- 生品那层 `rawmeat` 差异只进 `DoFoodEffects`：带 `eatsrawmeat` 的食用者吃带 `rawmeat` 的食物时，**负生命与负理智**的结算被跳过（`Eater:Eat` 分别用 `healthvalue >= 0 or DoFoodEffects` 与 `sanityvalue >= 0 or DoFoodEffects` 两道门），**饥饿从不受该门禁**；本页两个生品三维非负，实际不产生数值差，但「大鱼有、小鱼没有」会影响按该标签判定的第三方代码。[门禁](../../../raw/current/scripts/components/eater.lua#L211-L212)、[理智门禁](../../../raw/current/scripts/components/eater.lua#L233-L234)、[饥饿无门禁](../../../raw/current/scripts/components/eater.lua#L223-L224)、[判定函数](../../../raw/current/scripts/components/eater.lua#L166-L170)

## 两条转换链

```text
fishmeat        ──cook──▶ fishmeat_cooked        ──▶ 无 cookable，止步
   │  └─dry(1 天)──▶ fishmeat_dried
   └─perish(3 天)──▶ spoiled_fish
fishmeat_small  ──cook──▶ fishmeat_small_cooked
   │  └─dry(0.5 天)──▶ fishmeat_small_dried
   └─perish(3 天)──▶ spoiled_fish_small
```

- 烤制：`cookable.product` 指向熟品，`Cookable:Cook` 生成产物并把熟品鲜度设为 `1 - (1 - 生品百分比) * 0.5`，即**只保留一半腐坏量**；熟品自身没有 `cookable` 组件，不能再烤一次。[Cook 实现](../../../raw/current/scripts/components/cookable.lua#L18-L38)、[生品赋值](../../../raw/current/scripts/prefabs/meats.lua#L553)
- 可烤门禁（入口一：**玩家拿食材点烹饪器**，`componentactions.lua` 的 `cookable` 分支）：物品要有 `cookable` 标签；目标要有 `cooker` 标签且未 `fueldepleted`；目标带 `dangerouscooker` 时还要求 `doer` 带 `expertchef`；`doer` 骑乘时检查的是**目标（烹饪器）**是否归 `doer` 所有。另有入口二「拿烹饪器点食材」的 `cooker` 分支，门禁方向相反（骑乘看**食材**归属，且食材须被持有或可拾取），两者不要混。整叠时只取 1 个出来烤。[cookable 分支](../../../raw/current/scripts/componentactions.lua#L1096-L1105)、[cooker 分支](../../../raw/current/scripts/componentactions.lua#L1113-L1126)、[取单个](../../../raw/current/scripts/actions.lua#L2065-L2068)
- 晾晒：`dryable` 的 `product`／`time`／`build`／`dried_build` 四处一起决定挂架表现。两条链的 `build` 与 `dried_build` **都是** `meat_rack_food_tot`：生品上架时用 `GetBuildFile()`、烘干完成时用 `GetDriedBuildFile()`，显示时以「物品 prefab 名」作为 symbol 名去覆盖架子槽位。[干制数据](../../../raw/current/scripts/prefabs/meats.lua#L501)、[取 build](../../../raw/current/scripts/components/dryingrack.lua#L198-L220)、[显示](../../../raw/current/scripts/prefabs/meatrack2.lua#L133-L153)、[getter](../../../raw/current/scripts/components/dryable.lua#L32-L46)
- 干制时长：`fishmeat_small` 用 `DRY_VERYFAST`＝0.5 天（240 秒），`fishmeat` 用 `DRY_FAST`＝1 天（480 秒）。[时长常量](../../../raw/current/scripts/tuning.lua#L2053-L2056)、[完成回调](../../../raw/current/scripts/components/dryingrack.lua#L198-L211)
- 干品刷新了外观与保质：`bank`／`build` 换成 `meat_rack_food_tot`、动画 `dried_fishmeat(_small)`，鲜度 15 天，三维按上表抬升。[干品构造](../../../raw/current/scripts/prefabs/meats.lua#L585)、[小鱼干构造](../../../raw/current/scripts/prefabs/meats.lua#L536)
- `meatrack2` 的 `prefabs` 依赖表里列了四件生／干鱼肉，用于架子 build 的 symbol 换图；**依赖声明不等于生成**，实际生成点是 `dryingrack` 的 `OnDoneDrying`。[依赖](../../../raw/current/scripts/prefabs/meatrack2.lua#L47-L50)、[生成点](../../../raw/current/scripts/components/dryingrack.lua#L198-L211)

## 与常规肉链的差异

同为 `common()` 工厂产物，鱼链与常规肉三链（`meat`／`smallmeat`／`monstermeat`）差别集中在六处；三链并**不共享**鲜度档，`monstermeat` 必须单列：

| 项 | 鱼肉两链 | `meat`／`smallmeat` | `monstermeat` | 证据 |
|---|---|---|---|---|
| 生品鲜度 | 3 天（`PERISH_SUPERFAST`） | 6 天（`PERISH_FAST`） | 6 天（`PERISH_FAST`） | [鱼](../../../raw/current/scripts/prefabs/meats.lua#L562)、[大肉](../../../raw/current/scripts/prefabs/meats.lua#L394)、[怪物肉](../../../raw/current/scripts/prefabs/meats.lua#L294) |
| 熟品鲜度 | 6 天（`PERISH_FAST`） | 10 天（`PERISH_MED`） | **15 天**（`PERISH_SLOW`） | [鱼](../../../raw/current/scripts/prefabs/meats.lua#L578)、[大肉](../../../raw/current/scripts/prefabs/meats.lua#L352)、[熟怪物肉](../../../raw/current/scripts/prefabs/meats.lua#L320) |
| 干品鲜度 | 15 天（`PERISH_SLOW`） | 20 天（`PERISH_PRESERVED`） | 20 天（`PERISH_PRESERVED`） | [鱼](../../../raw/current/scripts/prefabs/meats.lua#L594)、[大肉干](../../../raw/current/scripts/prefabs/meats.lua#L376)、[怪物肉干](../../../raw/current/scripts/prefabs/meats.lua#L338) |
| 腐坏产物 | `spoiled_fish`／`spoiled_fish_small` | `spoiled_food` | `spoiled_food` | [鱼](../../../raw/current/scripts/prefabs/meats.lua#L563)、[常规](../../../raw/current/scripts/prefabs/meats.lua#L199) |
| 干品 build | `meat_rack_food_tot`（`dried_fishmeat*`） | `meat_rack_food`（`idle_dried_*`） | `meat_rack_food`（`idle_dried_*`） | [鱼](../../../raw/current/scripts/prefabs/meats.lua#L585)、[大肉干](../../../raw/current/scripts/prefabs/meats.lua#L367) |
| 生品标签 | 大鱼 `fishmeat`＋`catfood`＋`rawmeat`；小鱼块 `fishmeat`＋`catfood`（无 `rawmeat`） | `catfood`＋`rawmeat` | `monstermeat`＋`selfstacker`（**无** `catfood`／`rawmeat`） | [鱼](../../../raw/current/scripts/prefabs/meats.lua#L553)、[小肉](../../../raw/current/scripts/prefabs/meats.lua#L408)、[怪物肉](../../../raw/current/scripts/prefabs/meats.lua#L284) |

- 净效果（只对 `meat`／`smallmeat` 成立）：同样「生→熟→干」走一遍，鱼肉的每一档都比这两链**短一档**（3↔6、6↔10、15↔20 天），干鱼只有 15 天而不是 20 天；把这两链的保质期抄给鱼会高估约一倍。**怪物肉是例外**：生品同为 6 天，但 `cookedmonstermeat` 是 15 天（与鱼干同档、比熟鱼长 9 天），不能套「常规熟肉一律 10 天」。[常量](../../../raw/current/scripts/tuning.lua#L2043-L2056)、[熟怪物肉](../../../raw/current/scripts/prefabs/meats.lua#L320)
- 腐坏分流：鱼腐坏成 `spoiled_fish*`（肥料值与 `spoiled_food` 不同），不是 `spoiled_food`。两者锤开方式也不同：`spoiled_fish` 用 `SetLoot({spoiled_food, boneshard})`，**固定给两条**（`SetLoot` 会清空 `randomloot`／`numrandomloot`，之后再写 `numrandomloot = 1` 也无随机表可取）；`spoiled_fish_small` 用 `AddRandomLoot` 两条各权重 1＋`numrandomloot = 1`，**只随机其中一条**。[固定两条](../../../raw/current/scripts/prefabs/spoiledfood.lua#L29)、[大鱼装载](../../../raw/current/scripts/prefabs/spoiledfood.lua#L224-L235)、[随机一条](../../../raw/current/scripts/prefabs/spoiledfood.lua#L247-L261)、[SetLoot 语义](../../../raw/current/scripts/components/lootdropper.lua#L30-L35)

## 获得与消费

只列产出／消耗关系；世界态行为（刷新、行为树、钓法）见 [[reference/prefabs/ocean-fish]] 与 [[reference/prefabs/freshwater-fish]]。

| 来源／去向 | 产出或消耗 | 条件 | 证据 |
|---|---|---|---|
| 海洋鱼 `oceanfish_*` | 每条鱼各自引用四张表：`loot`（宰杀掉落）／`heavy_loot`（重鱼掉落）／`perish_product`（腐坏替代）／`cooking_product`（**火源烤制产物**，接 `_inv` 鱼的 `cookable.product`，**不是炖锅**）。**不能按体型一概而论**：默认小型组给 `fishmeat_small`、中型组给 `fishmeat`，但 `oceanfish_small_8` 的四项分别是 `LOOT.SMALL_COOKED`（宰杀即 `fishmeat_small_cooked`）／`HEAVY_LOOT.SMALL_COOKED`（重鱼 `fishmeat_cooked`）／`COOKING_PRODUCT.SMALL`（上火烤 `fishmeat_small_cooked`）／`PERISH.SMALL`（腐坏才回生 `fishmeat_small`）；另有 `small_6`／`small_7` 的 `PLANTMEAT` 组（给 `plantmeat`）等例外 | 按该鱼实际引用的表与状态（宰杀／重鱼／腐坏／上火烤）取值；完整鱼种表归 B07 | [四张表](../../../raw/current/scripts/prefabs/oceanfishdef.lua#L106-L147)、[small_8](../../../raw/current/scripts/prefabs/oceanfishdef.lua#L539-L542)、[PLANTMEAT 组](../../../raw/current/scripts/prefabs/oceanfishdef.lua#L449-L452)、[cooking_product 接 cookable](../../../raw/current/scripts/prefabs/oceanfish.lua#L527-L531) |
| `gnarwail`（一角鲸） | 4 个 `fishmeat`；角未被敲掉时掉落表另加 1 根 `gnarwail_horn` | 击杀掉落，两种掉落表按角是否损坏切换 | [掉落表](../../../raw/current/scripts/prefabs/gnarwail.lua#L29-L31)、[切换](../../../raw/current/scripts/prefabs/gnarwail.lua#L387-L397) |
| `shark`（岩石大白鲨） | 3–5 个 `fishmeat` | 概率表 1.0／1.0／1.0／0.5／0.25 | [掉落表](../../../raw/current/scripts/prefabs/shark.lua#L26-L33) |
| `lightcrab`（发光蟹） | 25% `fishmeat_small` | 与灯泡／壳片共用一次随机掉落 | [掉落](../../../raw/current/scripts/prefabs/lightcrab.lua#L103-L107) |
| 淡水鱼（`pondfish`，地表池塘现役钓获） | 宰杀（MURDER）→`fishmeat_small`，腐坏也换成 `fishmeat_small`；本体可上火烤→`fishmeat_small_cooked`（`cookable_product`）。**本体没有 `dryable`**，不能直接上晾肉架：要干制得先拿到 `fishmeat_small`，再走[两条转换链](#两条转换链) | 宰杀走 `loot` 表＋`GenerateLoot`，不经 `DropLoot`；`pond.lua` 的 `fishable:AddFish("pondfish")` 才是现役池塘鱼 | [数据表](../../../raw/current/scripts/prefabs/pondfish.lua#L161-L176)、[组件无 dryable](../../../raw/current/scripts/prefabs/pondfish.lua#L116-L138)、[MURDER 取表](../../../raw/current/scripts/actions.lua#L3185-L3196)、[钓获登记](../../../raw/current/scripts/prefabs/pond.lua#L329) |
| 鳗鱼（`eel`） | 本体挂 `dryable`：产物 `fishmeat_small_dried`、`DRY_FAST`、`dried_build = meat_rack_food_tot`；另有 `cookable`→`eel_cooked` | 是否挂 dryable 由构造参数决定，`eel` 传的是 true | [干制接线](../../../raw/current/scripts/prefabs/eel.lua#L66-L71)、[构造](../../../raw/current/scripts/prefabs/eel.lua#L94) |
| 「鱼」（`fish`，**已废弃**） | 定义仍在：`dryable`→`fishmeat_small_dried`、`cookable`→`fish_cooked`、腐坏→`spoiled_food`；但快照内检索不到 `SpawnPrefab("fish")`，采集册把 `fish`／`fish_cooked` 注释为 Deprecated，只剩 `prefablist` 里的加载与模组兼容 | **不要**把它当现役池塘产物；现役是 `pondfish` | [定义](../../../raw/current/scripts/prefabs/fish.lua#L70-L85)、[废弃标注](../../../raw/current/scripts/scrapbook_prefabs.lua#L385)、[同族标注](../../../raw/current/scripts/scrapbook_prefabs.lua#L842)、[加载](../../../raw/current/scripts/prefablist.lua#L438) |
| 海盗藏匿处 `pirate_stash` | 0.5 概率进入该段，3–6 次各 0.3 概率塞 1 个 `fishmeat_dried` | 海盗刷新时生成 | [战利品](../../../raw/current/scripts/components/piratespawner.lua#L584-L590) |
| 烹饪锅食材 | `fishmeat`＝`meat 1, fish 1`；`fishmeat_small`＝`meat .5, fish .5`；带 `cancook`／`candry` 展开 | 熟品另记 `precook = 1`、干品另记 `dried = 1` | [食材值](../../../raw/current/scripts/cooking.lua#L116-L117)、[展开规则](../../../raw/current/scripts/cooking.lua#L45-L72) |
| 沃特随身判定 | 背包里有带 `fishmeat` 标签的物品时理智**按速率流失** `-DAPPERNESS_MED_LARGE`≈0.074 点/秒（生／熟鱼都带，干鱼不带） | `itemaffinity` 取最高优先级一条写进 `sanity.externalmodifiers`（每秒速率，不是一次性扣点） | [沃特](../../../raw/current/scripts/prefabs/wurt.lua#L655-L659)、[取用](../../../raw/current/scripts/components/itemaffinity.lua#L38-L51)、[速率合并](../../../raw/current/scripts/components/sanity.lua#L590)、[常量](../../../raw/current/scripts/tuning.lua#L2215-L2216) |

- 关于 `BURNED_LOOT_OVERRIDES` 里的 `pondfish = "fishmeat_small_cooked"`：它只在 `DropLoot` 改写「即将掉出的物品列表」时命中，因此要分入口看——
  - **宰杀 `pondfish`**：`MURDER` 用 `GenerateLoot`＋逐件 `SpawnPrefab`（actions.lua#L3185-L3196），不经过 `DropLoot`，覆盖表不参与，得生 `fishmeat_small`。
  - **锤掉烧毁的 `mermhouse`（漏雨的小屋，天然鱼人屋）**：其固定 `loot` 表含 `pondfish`，锤击回调直接 `lootdropper:DropLoot()`，建筑带 `burnt` 时该项被覆盖表换成 `fishmeat_small_cooked`（lootdropper.lua#L415-L431）。
  - `pondfish` 自身没有 `burnable`（点不着），所以「把淡水鱼点着再烧出熟小鱼块」仍不成立，也不掉灰烬。[覆盖表](../../../raw/current/scripts/tuning.lua#L4527)、[MURDER](../../../raw/current/scripts/actions.lua#L3185-L3196)、[小屋战利品](../../../raw/current/scripts/prefabs/mermhouse.lua#L21-L26)、[战利品装载](../../../raw/current/scripts/prefabs/mermhouse.lua#L330)、[锤击掉落](../../../raw/current/scripts/prefabs/mermhouse.lua#L100-L104)、[DropLoot 改写](../../../raw/current/scripts/components/lootdropper.lua#L415-L431)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`edible` | 三维、`foodtype = MEAT`、`ismeat`、`badfood` 判定 | 主模拟；`oneaten` 未挂 | [[reference/components/edible#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/meats.lua#L182-L184) |
| 组件：`perishable` | 三档鲜度、`onperishreplacement` 指向鱼专用腐坏体 | 主模拟；`StartPerishing()` 已在 `common` 打开 | [[reference/components/perishable#关键接口]]；[赋值](../../../raw/current/scripts/prefabs/meats.lua#L196-L199) |
| 组件：`dryable` | 生品两件挂载；干品**不挂** | 主模拟；客户端靠 pristine 的 `dryable` 标签 | [[reference/components/dryable#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/meats.lua#L201-L207) |
| 组件：`cookable` | 生品两件挂载，写 `product` | 主模拟；客户端靠 `cookable` 标签 | [[reference/components/cookable#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/meats.lua#L209-L212) |
| 组件：`stackable` | 不写 `maxsize`，默认 20 | 主模拟统一挂载 | [[reference/components/stackable#关键接口]]；[默认值](../../../raw/current/scripts/components/stackable.lua#L26-L28) |
| 组件：`bait`／`inspectable`／`inventoryitem`／`tradable` | 统一挂载，`goldvalue` 取肉价 | 主模拟 | [接线](../../../raw/current/scripts/prefabs/meats.lua#L186-L194) |
| 数据：`cooking.lua`（炖锅食材） | 食材标签与 `precook`／`dried` 展开；与上表 `cooking_product`（火源烤制）是两条不同链 | 加载期注册；干／熟品共用生品标签 | [食材](../../../raw/current/scripts/cooking.lua#L116-L117) |
| 标签：`catfood` | 猫浣熊把带 `catfood` 的物品当可收下的东西 | 运行时行为归猫浣熊页 | [接收判定](../../../raw/current/scripts/prefabs/catcoon.lua#L273-L279) |
| 标签：`lureplant_bait` | 食人花用它当诱饵、被摘走会收起 | 生品两件与干品两件带，熟品两件不带（熟品只传 tags，第 5／6 参为空） | [选取](../../../raw/current/scripts/prefabs/lureplant.lua#L178-L196)、[分支](../../../raw/current/scripts/prefabs/meats.lua#L161-L172) |
| 关联 Prefab：`spoiled_fish`／`spoiled_fish_small` | 腐坏产物；可当肥料、可锤开 | 本页只给入口 | [注册](../../../raw/current/scripts/prefabs/spoiledfood.lua#L263-L265) |
| 关联 Prefab：`meatrack2` 家族 | 晾晒宿主，按 `build`／`dried_build` 换图 | 架子本体另页 | [[reference/components/dryingrack]]；[显示](../../../raw/current/scripts/prefabs/meatrack2.lua#L133-L153) |
| 资源：`fishmeat.zip`／`fishmeat_small.zip`／`meat_rack_food_tot.zip` | 物品动画与架子 symbol 来源 | 只核验声明，未验证资源内容 | [assets](../../../raw/current/scripts/prefabs/meats.lua#L1-L15) |

## 常见陷阱与限制

- **误区：鱼肉也是「6 天生鲜」**：生品是 `PERISH_SUPERFAST`＝3 天，短于常规生肉一半；照常规肉排库存会把「还剩一点」算成「还能放三天」。[赋值](../../../raw/current/scripts/prefabs/meats.lua#L562)
- **边界：腐坏产物分大小两件**：`fishmeat*` → `spoiled_fish`，`fishmeat_small*` → `spoiled_fish_small`，都不会变成 `spoiled_food`；依赖 `spoiled_food` 的配方不能指望鱼腐坏。[小鱼](../../../raw/current/scripts/prefabs/meats.lua#L511)、[大鱼](../../../raw/current/scripts/prefabs/meats.lua#L563)
- **边界：干鱼不能再晾、也不能上火烤**：干品只带 `lureplant_bait`，没有 `dryable`／`cookable` 组件与标签（`isdried` 在 `common()` 里没有读取点）。熟品两件同理没有 `cookable`／`dryable`／`lureplant_bait`：它们只传 4 个参数，第 5／6 参为空，最终标签只有 `meat`＋`fishmeat`。[干品分支](../../../raw/current/scripts/prefabs/meats.lua#L161-L172)、[熟鱼调用](../../../raw/current/scripts/prefabs/meats.lua#L569)
- **边界：烤一次只处理 1 个并折损鲜度**：整叠烤制时只取 1 个，熟品鲜度＝`1 - (1 - 生品比例) × 0.5`，90% 新鲜的生鱼只烤出 95% 新鲜的熟鱼，不是满鲜。[Cook](../../../raw/current/scripts/components/cookable.lua#L22-L35)、[取单个](../../../raw/current/scripts/actions.lua#L2065-L2068)
- **边界：架子显示按 prefab 名找 symbol**：晾晒与成品都用 `meat_rack_food_tot` 这一个 build，靠「prefab 名＝symbol 名」换图；模组新增鱼干若沿用该 build 却没有同名 symbol，会显示成空槽或错图。资源内是否含该 symbol 未在本页验证。[显示](../../../raw/current/scripts/prefabs/meatrack2.lua#L148)、[取 build](../../../raw/current/scripts/components/dryingrack.lua#L205)
- **边界：沃特带鱼掉理智、带鱼干不掉**：判定读 `fishmeat` 标签，干品没有该标签；生／熟鱼在背包里都是**每秒** −0.074 理智（速率修正，随携带时长累积，不是一次性 −20）。[沃特](../../../raw/current/scripts/prefabs/wurt.lua#L658)、[熟品标签](../../../raw/current/scripts/prefabs/meats.lua#L569)、[速率](../../../raw/current/scripts/components/sanity.lua#L590)
- **未核验**：`bait` 组件的消费端、18 条海洋鱼各自四张表的完整映射与权重、`oceanfishdef.cooker_ingredient_value` 分支、`spoiled_fish*` 的肥料数值与锤击次数。

## 验证记录

- 源码审阅：2026-11-09，Steam buildid 26474873。核对 `prefabs/meats.lua` 的 `common()`、六个变体、表尾导出、依赖表与 assets；`prefablist.lua` 的 `"meats"` 登记；`edible`／`perishable`／`dryable`／`dryingrack`／`cookable`／`stackable`／`eater`／`itemaffinity`／`lootdropper`／`piratespawner` 的相关分支；`meatrack2` 的显示函数与依赖表；`spoiledfood` 的鱼腐坏体；`oceanfishdef`／`gnarwail`／`shark`／`lightcrab`／`fish`／`eel`／`pondfish` 的掉落与干制接线、`pond.lua` 的钓获登记、`pondfish` 的组件清单（确认无 `dryable`）；`mermhouse` 的战利品表与锤击 `DropLoot` 路径；`container` 的 `GiveItem` 拒收分支；`cooking.lua` 的食材注册与展开；`wurt` 的随身判定；`componentactions`／`actions` 的烤制门禁与 `MURDER` 取表；`tuning` 的三维／鲜度／干制／堆叠常量与 `BURNED_LOOT_OVERRIDES`；`scrapbookdata`／`scrapbook_prefabs` 的图鉴记录与废弃标注。
  名称核验：`NAMES.FISHMEAT`＝生鱼肉、`FISHMEAT_COOKED`＝鱼排、`FISHMEAT_DRIED`＝鱼肉干、`FISHMEAT_SMALL`＝小鱼块、`FISHMEAT_SMALL_COOKED`＝熟小鱼块、`FISHMEAT_SMALL_DRIED`＝小鱼干；另核 `FISH`＝鱼、`PONDFISH`＝淡水鱼、`MERMHOUSE`＝漏雨的小屋。[NAMES.FISHMEAT 起](../../../raw/current/scripts/languages/chinese_s.po#L342839-L342867)、[FISH](../../../raw/current/scripts/languages/chinese_s.po#L342825-L342827)、[PONDFISH](../../../raw/current/scripts/languages/chinese_s.po#L347155-L347157)、[MERMHOUSE](../../../raw/current/scripts/languages/chinese_s.po#L345335-L345337)
  未覆盖：世界态鱼实体、`bait` 消费端、海洋鱼四张表在 18 条鱼上的完整映射与权重、`spoiled_fish*` 完整机制、图鉴解锁键。
- 游戏运行：未运行。三维、天数与秒数均由 `tuning` 常量推导。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 通过（见交付回报）。

## 相关链接

- 可食与鲜度：[[reference/components/edible]]、[[reference/components/perishable]]、[[reference/components/stackable]]
- 熟制与干制：[[reference/components/cookable]]、[[reference/components/dryable]]、[[reference/components/dryingrack]]
- 同源工厂的常规肉链：[[reference/prefabs/meat|肉与共用工厂]]、[[reference/prefabs/smallmeat|小肉链]]、[[reference/prefabs/monstermeat|怪物肉链]]
- 同批新页（火腺，同源工厂）：[[reference/prefabs/mitegland]]
- 火荨麻毒素的温度修正（与火腺 debuff 共用常量）：[[reference/prefabs/weeds#火荨麻叶的毒素链]]

返回 [[reference/prefabs/index|Prefab 索引]]。
