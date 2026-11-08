---
title: "洞穴蝙蝠翅膀与裸露鼻孔（batwing、batwing_cooked、batnose、batnose_cooked）"
created: "2026-11-09"
updated: "2026-11-09"
type: prefab
summary: "meats.lua 的四个蝙蝠来源肉品注册：洞穴蝙蝠翅膀与裸露鼻孔的标签、三维、两档干制 build、雪人装饰与漂浮差异，及其熟态对照和洞穴蝙蝠／裸鼹蝠两种掉落来源。"
aliases: ["batwing", "batwing_cooked", "batnose", "batnose_cooked", "洞穴蝙蝠翅膀", "熟蝙蝠翅膀", "裸露鼻孔", "炭烤鼻孔", "蝙蝠翅膀"]
tags: [prefab, item, food]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/meats.lua", "actions.lua", "components/snowmandecoratable.lua", "screens/redux/snowmandecoratingscreen.lua", "components/edible.lua", "components/perishable.lua", "components/stackable.lua", "components/dryable.lua", "components/cookable.lua", "components/floater.lua", "components/bait.lua", "components/eater.lua", "components/dryingrack.lua", "components/snowmandecor.lua", "componentactions.lua", "containers.lua", "standardcomponents.lua", "prefabs/bat.lua", "prefabs/molebat.lua", "prefabs/catcoon.lua", "cooking.lua", "recipes.lua", "tuning.lua", "languages/chinese_s.po", "screens/redux/scrapbookdata.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 洞穴蝙蝠翅膀与裸露鼻孔（batwing、batwing_cooked、batnose、batnose_cooked）

## 快速判断

- 职责与适用：四个注册共享 `meats.lua` 的 `common()` 工厂，都是可拾取食物；两个生肉注册各走「干制」与「烤制」两条转换。`batwing` 与 `batnose` 是两种不同来源的生肉，熟态共享同一组三维；同页对照可看清干制 build、雪人装饰与漂浮三处差异。[翅膀两态工厂](../../../raw/current/scripts/prefabs/meats.lua#L600-L634)、[鼻孔两态工厂](../../../raw/current/scripts/prefabs/meats.lua#L733-L776)
- 优先使用：改数值看[数值与获得](#数值与获得)；查两者差异看[两件生肉对照](#两件生肉对照)；干制显示异常看[关联机制导航](#关联机制导航)的 `dryable` 行。
- 重要副作用：`batwing` 额外挂 `snowmandecor`，能拿去装饰雪人；`batnose` 在肉架上挂着的 build 与干好后的 build 不是同一个（`meat_rack_food_tot` → `meat_rack_food`），干出来的产物是小肉干。[雪人装饰](../../../raw/current/scripts/prefabs/meats.lua#L617-L617)、[干制数据](../../../raw/current/scripts/prefabs/meats.lua#L734-L740)
- 修改边界：只讲这四个注册与它们的两条转换；洞穴蝙蝠、裸鼹蝠的行为与掉落逻辑只记获得关系；晾肉架本体、雪人本体各归其页。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `batwing` | `batwing()` 调 `common("batwing", "batwing", "raw", {"batwing", "catfood", "rawmeat"}, {product="smallmeat_dried", time=DRY_MED}, {product="batwing_cooked"})`；**端分界之前**先写漂浮，之后才判 `ismastersim` | 标签：`meat`＋`batwing`／`catfood`／`rawmeat`＋`dryable`／`lureplant_bait`＋`cookable`；主模拟挂 `edible`（`FOODTYPE.MEAT`）、`bait`、`inspectable`、`inventoryitem`、`stackable`、`tradable`、`perishable`、`dryable`、`cookable`，并额外 `AddComponent("snowmandecor")`。[构造](../../../raw/current/scripts/prefabs/meats.lua#L600-L620) |
| `batwing_cooked` | `batwing_cooked()` 调 `common("batwing", "batwing", "cooked", {"batwing"})`，`dryable`／`cookable` 都传 `nil` | 只保留 `meat`＋`batwing`；无 `snowmandecor`、无干制与烤制入口；`sanityvalue` 未赋值，沿用默认 0。[熟态](../../../raw/current/scripts/prefabs/meats.lua#L622-L634) |
| `batnose` | `batnose()` 调 `common("batnose", "batnose", "raw", BATNOSE_TAGS, BATNOSE_DRYABLE_DATA, BATNOSE_COOKABLE_DATA)`；同样在端分界前写漂浮 | `BATNOSE_TAGS` 只有 `catfood`／`rawmeat`，既没有 `batwing` 也没有 `batnose`；干制数据带 `build`／`dried_build`；无 `snowmandecor`。[标签](../../../raw/current/scripts/prefabs/meats.lua#L733-L733)、[构造](../../../raw/current/scripts/prefabs/meats.lua#L742-L759) |
| `batnose_cooked` | `batnose_cooked()` 调 `common("batnose", "batnose", "cooked")`——`tags` 也省略 | 只有共用的 `meat` 标签；无额外组件入口；漂浮仍写 `med`／0.7。[熟态](../../../raw/current/scripts/prefabs/meats.lua#L761-L776) |

四件的注册与预载分组：`batwing` 与 `batnose` 共用同一份 `batwingprefabs`（含 `batwing_cooked`／`meat_dried`／`spoiled_food`），两个熟态都无预载表；`batnose` 用独立的 `batnose_assets`（`anim/batnose.zip`）。[返回表](../../../raw/current/scripts/prefabs/meats.lua#L910-L911)、[batnose 注册](../../../raw/current/scripts/prefabs/meats.lua#L927-L928)、[预载表](../../../raw/current/scripts/prefabs/meats.lua#L80-L85)、[资源](../../../raw/current/scripts/prefabs/meats.lua#L22-L25)

`common()` 的共用部分与同文件其它肉品一致：写 bank／build／anim、加 `meat` 标签、`MakeInventoryFloatable(inst)`（默认 `small`、无偏移）、`SetPristine`、非主模拟返回；主模拟统一挂 `edible`（`ismeat=true`、`FOODTYPE.MEAT`）、`bait`、`inspectable`、`inventoryitem`、`stackable`（默认 20）、`tradable`（金值 1）、`perishable`（腐坏→`spoiled_food`），并按传参补 `dryable`／`cookable`。[工厂](../../../raw/current/scripts/prefabs/meats.lua#L138-L222)、[漂浮默认](../../../raw/current/scripts/standardcomponents.lua#L1772-L1791)、[bait 组件](../../../raw/current/scripts/components/bait.lua#L21-L29)

### 名称核验

`batwing`＝**洞穴蝙蝠翅膀**（Batilisk Wing）、`batwing_cooked`＝**熟蝙蝠翅膀**、`batnose`＝**裸露鼻孔**（Naked Nostrils）、`batnose_cooked`＝**炭烤鼻孔**（Charred Nostrils）；四个键各命中一次，注册名与 `NAMES` 键同名。[BATWING](../../../raw/current/scripts/languages/chinese_s.po#L339184-L339187)、[BATWING_COOKED](../../../raw/current/scripts/languages/chinese_s.po#L339189-L339192)、[BATNOSE](../../../raw/current/scripts/languages/chinese_s.po#L339114-L339117)、[BATNOSE_COOKED](../../../raw/current/scripts/languages/chinese_s.po#L339125-L339127)

## 数值与获得

时间基准：`seg_time`＝30 秒、`total_day_time`＝480 秒，下面天数按 480 秒/天换算。[基准](../../../raw/current/scripts/tuning.lua#L19-L20)

| 注册 | 饥饿／生命／理智 | 鲜度 | 堆叠 | 漂浮 | 转换 |
|---|---|---|---|---|---|
| `batwing` | 12.5／3／−10 | `PERISH_FAST`＝6 天＝2880 秒 | 20 | `med`，偏移 0.02、缩放 0.8 | 干制 `DRY_MED`＝2 天＝960 秒 → `smallmeat_dried`；烤制 → `batwing_cooked` |
| `batwing_cooked` | 18.75／8／0 | `PERISH_MED`＝10 天＝4800 秒 | 20 | 未设，沿用默认 `small`／无偏移／1 | 无 |
| `batnose` | 12.5／3／−10 | `PERISH_FAST`＝2880 秒 | 20 | `med`，无偏移、缩放 0.7 | 干制 `DRY_MED`＝960 秒 → `smallmeat_dried`（挂架 build `meat_rack_food_tot`）；烤制 → `batnose_cooked` |
| `batnose_cooked` | 18.75／8／0 | `PERISH_MED`＝4800 秒 | 20 | `med`，无偏移、缩放 0.7 | 无 |

数值来源：[batwing](../../../raw/current/scripts/prefabs/meats.lua#L611-L615)、[batwing_cooked](../../../raw/current/scripts/prefabs/meats.lua#L629-L631)、[batnose](../../../raw/current/scripts/prefabs/meats.lua#L752-L756)、[batnose_cooked](../../../raw/current/scripts/prefabs/meats.lua#L771-L773)、[常量](../../../raw/current/scripts/tuning.lua#L2021-L2023)、[热量](../../../raw/current/scripts/tuning.lua#L2058-L2060)、[理智](../../../raw/current/scripts/tuning.lua#L2033-L2034)、[鲜度](../../../raw/current/scripts/tuning.lua#L2043-L2051)、[干制](../../../raw/current/scripts/tuning.lua#L2053-L2056)、[堆叠](../../../raw/current/scripts/tuning.lua#L82-L82)。采集册对应行同值：`batwing` 标 `snowmandecor=true`，四件 `stacksize=20`。[采集册](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L130-L131)、[batnose](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L116-L117)

### 两件生肉对照

| 项目 | `batwing` | `batnose` |
|---|---|---|
| 标签 | `batwing`＋`catfood`＋`rawmeat` | 仅 `catfood`＋`rawmeat` |
| 干制产物与时长 | `smallmeat_dried`，`DRY_MED` | 同左 |
| 干制 build | 未传，肉架显示回落默认 build | `build=meat_rack_food_tot`（挂架时）、`dried_build=meat_rack_food`（干好后） |
| 雪人装饰 | 有 `snowmandecor`，可对雪人使用装饰动作 | 无 |
| 漂浮 | `med`／0.02／0.8 | `med`／无偏移／0.7 |
| 熟态 | `batwing_cooked`（保留 `batwing` 标签） | `batnose_cooked`（无额外标签） |

干制 build 的取用位置：挂架时用 `GetBuildFile()`、完成时用 `GetDriedBuildFile()`（缺省回落到 `GetBuildFile()`），所以 `batnose` 挂着是 `meat_rack_food_tot`、干好换成 `meat_rack_food`；其余三件都回落 `DEFAULT_MEAT_BUILD`。[取用](../../../raw/current/scripts/components/dryingrack.lua#L62-L76)、[完成](../../../raw/current/scripts/components/dryingrack.lua#L198-L224)、[组件](../../../raw/current/scripts/components/dryable.lua#L32-L46)

### 获得路径

| 注册 | 已核验来源 | 产出与证据 |
|---|---|---|
| `batwing` | ①洞穴蝙蝠（`bat`）共享掉落表 0.25，酸液变体 0.5；②浣猫（`catcoon`）礼物池第 5 档含 `batwing` | ①[bat](../../../raw/current/scripts/prefabs/bat.lua#L22-L34)、②[礼物档](../../../raw/current/scripts/prefabs/catcoon.lua#L159-L161) |
| `batnose` | 裸鼹蝠（`molebat`）随机战利品：`monstermeat` 权重 2、`batnose` 权重 1，`numrandomloot=1`，即约 1/3 | [随机战利品](../../../raw/current/scripts/prefabs/molebat.lua#L304-L308) |
| `batwing_cooked`／`batnose_cooked` | ①各自 `cookable.product` 指向：在带 `cooker` 标签的实体（营火、火坑、龙鳞火炉（`dragonflyfurnace`，文件 `prefabs/dragonfurnace.lua`）等）上烤；烹饪锅只有 `stewer` 标签与组件，不提供烤制；②掉落者已带 `burnt` 标签，或正在燃烧且（无 `fueled` 组件或 `burnable.ignorefuel` 为真）时，`DropLoot` 会把 `batwing`／`batnose` 换成对应熟态 | ①[batwing](../../../raw/current/scripts/prefabs/meats.lua#L601-L601)、[batnose](../../../raw/current/scripts/prefabs/meats.lua#L741-L741)、[烤制门槛](../../../raw/current/scripts/componentactions.lua#L1096-L1104)、[烹饪锅](../../../raw/current/scripts/prefabs/cookpot.lua#L290-L309)；②[门禁](../../../raw/current/scripts/components/lootdropper.lua#L415-L421)、[换名规则](../../../raw/current/scripts/components/lootdropper.lua#L424-L431) |

用途补充：烹饪锅把 `batwing` 记为 `meat=0.5`、`batnose` 记为 `meat=0.5`；配方里 `batwing` 用于蝙蝠棒（3 个）、唱片（1 个）与万圣实验根（1 个），`batnose` 用于 WX 尖啸模块（1 个）。[食材值](../../../raw/current/scripts/cooking.lua#L108-L109)、[蝙蝠棒](../../../raw/current/scripts/recipes.lua#L481-L481)、[唱片](../../../raw/current/scripts/recipes.lua#L1048-L1048)、[实验根](../../../raw/current/scripts/recipes.lua#L1421-L1421)、[模块](../../../raw/current/scripts/recipes.lua#L779-L779)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：edible | 写三维与 `FOODTYPE.MEAT`；`ismeat` 一并置真 | 仅主模拟；两种生肉理智为负，赋值即加 `badfood`（不能喂给其他玩家），熟态改回 18.75／8／0 后不再加 | [[reference/components/edible#关键接口]]；[batwing](../../../raw/current/scripts/prefabs/meats.lua#L611-L613)、[batnose](../../../raw/current/scripts/prefabs/meats.lua#L752-L754)、[`badfood`](../../../raw/current/scripts/components/edible.lua#L1-L7)、[喂食门槛](../../../raw/current/scripts/actions.lua#L2406-L2410) |
| 组件：perishable | 生肉 2880 秒、熟肉 4800 秒，腐坏替换 `spoiled_food` | 仅主模拟；标签在 0.5／0.2 百分比处切 | [[reference/components/perishable#关键接口]]；[batwing](../../../raw/current/scripts/prefabs/meats.lua#L615-L615)、[batwing_cooked](../../../raw/current/scripts/prefabs/meats.lua#L631-L631) |
| 组件：dryable | `product=smallmeat_dried`、`time=DRY_MED`；`batnose` 另带两段 build | 仅生肉；组件在主模拟、`dryable` 标签在两端 | [[reference/components/dryable#关键接口]]；[接线](../../../raw/current/scripts/prefabs/meats.lua#L734-L740)、[肉架接收](../../../raw/current/scripts/containers.lua#L234-L244) |
| 组件：cookable | `product` 指向各自熟态；`Cook()` 把鲜度折半补回产物；同一标签还决定能否进盐盒，故两种生肉可、两个熟态不可 | 仅生肉；烤后不再可烤 | [[reference/components/cookable#关键接口]]；[接线](../../../raw/current/scripts/prefabs/meats.lua#L601-L601)、[折算](../../../raw/current/scripts/components/cookable.lua#L28-L36)、[盐盒](../../../raw/current/scripts/containers.lua#L1049-L1055) |
| 组件：snowmandecor | 仅 `batwing` 挂；提供「装饰雪人」动作，条件是目标有 `snowmandecoratable` 且带 `heavy`、非 `waxedplant`，并且 `doer.replica.inventory` 不在 `IsHeavyLifting()`——玩家搬运重物时不会出现装饰动作（堆叠分支同样排除搬运重物） | 组件本身只有 `inst` 字段，行为在 `componentactions` 里 | [组件](../../../raw/current/scripts/components/snowmandecor.lua#L1-L7)、[物品分支](../../../raw/current/scripts/componentactions.lua#L1745-L1756)、[堆叠分支](../../../raw/current/scripts/componentactions.lua#L1758-L1766) |
| 组件：floater | `batwing`／`batnose`／`batnose_cooked` 都在端分界之前写 `SetSize`／`SetVerticalOffset`／`SetScale`，客户端副本同样带这些参数；`batwing_cooked` 一处都不写 | `batwing` 0.02／0.8、`batnose` 只改 0.7；熟态里 `batwing_cooked` 未设、`batnose_cooked` 沿用 0.7 | [[reference/components/floater#关键接口]]；[挂点](../../../raw/current/scripts/components/floater.lua#L107-L117)、[batwing](../../../raw/current/scripts/prefabs/meats.lua#L603-L605)、[batnose](../../../raw/current/scripts/prefabs/meats.lua#L745-L746) |
| 组件：stackable／tradable | 堆叠沿用默认 20；金值 `GOLD_VALUES.MEAT`＝1 | 仅主模拟；四件都未覆盖 | [[reference/components/stackable#关键接口]]、[[reference/components/tradable#关键接口]]；[堆叠默认](../../../raw/current/scripts/components/stackable.lua#L28-L28)、[金值](../../../raw/current/scripts/tuning.lua#L1946-L1952) |
| 标签：`catfood`／`rawmeat` | 浣猫按 `catfood` 接受物品；`rawmeat` 让 `eatsrawmeat` 吃者跳过负生命／理智结算 | 标签在两端；熟态都不再带（`batwing_cooked` 只留 `batwing`） | [浣猫](../../../raw/current/scripts/prefabs/catcoon.lua#L273-L279)、[吃者](../../../raw/current/scripts/components/eater.lua#L166-L170) |
| 关联 Prefab：`smallmeat_dried`／`spoiled_food` | 干制产物与腐坏替换物 | 主模拟到点生成 | [干制](../../../raw/current/scripts/prefabs/meats.lua#L601-L601)、[腐坏](../../../raw/current/scripts/prefabs/meats.lua#L199-L199) |
| 资源：`anim/batwing.zip`／`anim/batnose.zip` | 翅膀两态共用 `batwing` bank／build；鼻孔两态共用 `batnose` bank／build | 原版资源，不随页分发 | [assets](../../../raw/current/scripts/prefabs/meats.lua#L12-L12)、[鼻孔 assets](../../../raw/current/scripts/prefabs/meats.lua#L22-L25) |

## 常见陷阱与限制

- **误区：`batwing` 干出来是 `meat_dried`。** 它的预载表确实写着 `meat_dried`，但 `dryable.product` 是 `smallmeat_dried`；预载声明只是引用列表，不决定产物。[预载表](../../../raw/current/scripts/prefabs/meats.lua#L80-L85)、[产物](../../../raw/current/scripts/prefabs/meats.lua#L601-L601)
- **边界：`batnose` 挂架与干好是两个 build。** 挂架显示 `meat_rack_food_tot`，完成品换成 `meat_rack_food`（与小肉干同 build）；不要据此认为产物 prefab 也变了。[干制数据](../../../raw/current/scripts/prefabs/meats.lua#L734-L740)、[完成回调](../../../raw/current/scripts/components/dryingrack.lua#L198-L224)
- **边界：只有生翅膀能装饰雪人。** `batwing_cooked` 与两种鼻孔都没有 `snowmandecor`。[batwing](../../../raw/current/scripts/prefabs/meats.lua#L617-L617)、[熟态](../../../raw/current/scripts/prefabs/meats.lua#L622-L634)
- **边界：两个熟态并不等同。** 三维相同，但 `batwing_cooked` 保留 `batwing` 标签、漂浮参数未设；`batnose_cooked` 只有 `meat` 标签、漂浮沿用 `med`／0.7。[batwing_cooked](../../../raw/current/scripts/prefabs/meats.lua#L622-L634)、[batnose_cooked](../../../raw/current/scripts/prefabs/meats.lua#L761-L776)
- **边界：不要只给熟态补 `snowmandecor` 就以为能装饰雪人。** 装饰数据的唯一来源是 `snowmandecoratable.lua` 的本地表 `temp`，加载时写成 `ITEM_DATA[hash(prefab)]`，表里只有生 `batwing`；客户端拖动按 `GetItemData(hash(obj.prefab))` 查表，查不到就不生成拖拽物，服务端提交再按同一张表校验并按名扣件，所以只加组件最多让右键出现动作，实际摆放与扣件都会失败；原版没有向外注入这张表的入口。[表与构建](../../../raw/current/scripts/components/snowmandecoratable.lua#L3-L49)、[客户端查表](../../../raw/current/scripts/screens/redux/snowmandecoratingscreen.lua#L293-L295)、[服务端校验](../../../raw/current/scripts/components/snowmandecoratable.lua#L244-L271)
- **未核验**：本页未运行游戏；`molebat` 的 1/3 只按权重推导，未实测掉落分布；雪人装饰动作的界面表现未实测。

## 验证记录

- 源码审阅：2026-11-09，Steam buildid 26474873。核对 `meats.lua` 的 `common()`、四个工厂与返回表；`edible`／`perishable`／`stackable`／`dryable`／`cookable`／`floater`／`bait`／`eater`／`snowmandecor`／`snowmandecoratable`（`ITEM_DATA` 构建、客户端 `GetItemData`、服务端 `_ValidateAndAppendDecorData`）与 `componentactions` 的装饰分支、`snowmandecoratingscreen` 的查表；`dryingrack` 的 build 取用与完成回调、`containers.lua` 的肉架接收条件；`bat`／`molebat`／`catcoon` 的掉落与礼物表；`cooking.lua` 与 `recipes.lua` 的用途；`tuning` 的鲜度／干制／重伤／热量／理智／堆叠／金值常量；`scrapbookdata` 交叉核对。
  复审修订（同日）：补核盐盒的 `cookable` 门槛。
  复检修订（同日，第三轮）：删除「给熟态补 `snowmandecor`」的接入示例，改记为边界——装饰数据只认 `snowmandecoratable.lua` 的 `ITEM_DATA`，熟态无条目，客户端拖动与服务端提交两处都会失败。
  复检修订（同日，第二轮）：两个熟态的烤制入口改为「带 `cooker` 标签的实体」（烹饪锅不提供烤制），补 `DropLoot` 燃烧换名来源。
  复检修订（同日，第四轮）：`dragonfurnace` 的注册名改为 `dragonflyfurnace`；装饰动作条件补 `doer.replica.inventory` 不在 `IsHeavyLifting()`（搬运重物时不出现装饰动作，堆叠分支同样排除）。
  名称核验：`languages/chinese_s.po` 的 `NAMES.BATWING`＝洞穴蝙蝠翅膀、`BATWING_COOKED`＝熟蝙蝠翅膀、`BATNOSE`＝裸露鼻孔、`BATNOSE_COOKED`＝炭烤鼻孔；四个注册名与 `NAMES` 键同名。[po 锚点](#名称核验)
  未覆盖：模组注入与动态选名；雪人本体的装饰槽位与保存逻辑；原版是否在别处提供 `ITEM_DATA` 的注入点（本页只在 `snowmandecoratable.lua` 与装饰屏内检索）；`bat` 之外的蝙蝠皮肤／活动变体战利品表。
- 游戏运行：未运行。天数与三维均为 tuning 推导（`total_day_time`＝480 秒）。
- 结构检查：`validate_page.py` 通过（standard 档内）；`validate_source_links.py --strict` 0 错 0 警。

## 相关链接

- 可食与鲜度：[[reference/components/edible]]、[[reference/components/perishable]]
- 两条转换：[[reference/components/dryable]]、[[reference/components/cookable]]
- 漂浮与堆叠：[[reference/components/floater]]、[[reference/components/stackable]]

返回 [[reference/prefabs/index|Prefab 索引]]。
