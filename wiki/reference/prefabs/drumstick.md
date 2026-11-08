---
title: "鸟腿与炸鸟腿（drumstick、drumstick_cooked）"
created: "2026-11-09"
updated: "2026-11-09"
type: prefab
summary: "meats.lua 的 drumstick 与 drumstick_cooked：共用 common() 工厂下的标签、烹饪锅食材值、三维／鲜度／堆叠，干制成 smallmeat_dried 与烤制成炸鸟腿两条转换，以及火鸡、秃鹫、企鹅、麋鹿鹅的掉落来源。"
aliases: ["drumstick", "drumstick_cooked", "鸟腿", "炸鸟腿"]
tags: [prefab, item, food]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/meats.lua", "actions.lua", "componentactions.lua", "components/edible.lua", "components/perishable.lua", "components/stackable.lua", "components/dryable.lua", "components/cookable.lua", "components/floater.lua", "components/bait.lua", "components/eater.lua", "components/dryingrack.lua", "containers.lua", "standardcomponents.lua", "prefabs/birds.lua", "prefabs/perd.lua", "prefabs/buzzard.lua", "prefabs/moose.lua", "prefabs/mossling.lua", "prefabs/penguin.lua", "prefabs/catcoon.lua", "prefabs/oceanfish.lua", "cooking.lua", "tuning.lua", "languages/chinese_s.po", "screens/redux/scrapbookdata.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 鸟腿与炸鸟腿（drumstick、drumstick_cooked）

## 快速判断

- 职责与适用：两个注册都是可拾取食物，且都由 `meats.lua` 的 `common()` 生成；生腿 `drumstick` 带 `catfood`／`rawmeat`／`drumstick` 标签，能上晾肉架干成 `smallmeat_dried`、能烤成 `drumstick_cooked`；熟态不再可烤也不可干制。[生腿构造](../../../raw/current/scripts/prefabs/meats.lua#L465-L481)、[熟态构造](../../../raw/current/scripts/prefabs/meats.lua#L483-L498)
- 优先使用：改三维／鲜度看[数值与获得](#数值与获得)；改干制产物或时长看[关联机制导航](#关联机制导航)的 `dryable` 行；改掉落看[获得路径](#获得路径)。
- 重要副作用：干制不是就地改名——到点后原物被 `Remove`、另生成干制产物并继承湿度（[完成回调](../../../raw/current/scripts/components/dryingrack.lua#L198-L224)）；烤制会把新鲜度按 `1-(1-p)*0.5` 折算给产物（[Cook](../../../raw/current/scripts/components/cookable.lua#L18-L38)）。
- 修改边界：只讲这两个注册的构造、数值与获得；晾肉架本体（`meatrack2`）、鸟类行为与捕鸟、以及同文件的其它肉品各归其页。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `drumstick` | `drumstick()` 调 `common("drumstick", "drumstick", "raw", {"drumstick", "catfood", "rawmeat"}, {product="smallmeat_dried", time=DRY_FAST}, {product="drumstick_cooked"})`；`common` 内先做动画／物理／标签／漂浮，`SetPristine` 之后非主模拟直接返回 | 生腿：除共用的 `meat` 标签外另有三个标签，因 `dryable.product` 非空再补 `dryable` 与 `lureplant_bait`，因 `cookable` 非空补 `cookable`；主模拟再挂 `edible`（`ismeat=true`、`FOODTYPE.MEAT`）、`bait`、`inspectable`、`inventoryitem`、`stackable`、`tradable`、`perishable`、`dryable`、`cookable`，最后 `MakeHauntableLaunchAndPerish`。[构造](../../../raw/current/scripts/prefabs/meats.lua#L465-L481)、[共用工厂](../../../raw/current/scripts/prefabs/meats.lua#L138-L222) |
| `drumstick_cooked` | `drumstick_cooked()` 调 `common("drumstick", "drumstick", "cooked", {"drumstick"})`；`dryable` 与 `cookable` 都传 `nil` | 只保留 `meat`＋`drumstick` 标签，因此没有 `dryable`／`cookable` 标签也没有同名组件：不能再烤、不能上架；`sanityvalue` 未赋值，沿用 `edible` 默认 0。[熟态](../../../raw/current/scripts/prefabs/meats.lua#L483-L498)、[默认三维](../../../raw/current/scripts/components/edible.lua#L19-L44) |

`common()` 中与两件都有关的固定动作：写 bank／build／anim 与 `scrapbook_anim`；加 `meat` 标签；`MakeInventoryFloatable(inst)` 只给默认 `small` 尺寸、无偏移无缩放；`SetPristine` 后非主模拟返回。主模拟统一挂 `edible`（`ismeat = true`、`foodtype = FOODTYPE.MEAT`）、`bait`、`inspectable`、`inventoryitem`、`stackable`（不改 `maxsize`，沿用组件默认 20）、`tradable`（`goldvalue = TUNING.GOLD_VALUES.MEAT`＝1）、`perishable`（先 `PERISH_FAST`、`StartPerishing`、腐坏替换 `spoiled_food`），只有传参才补 `dryable`／`cookable`。[工厂主体](../../../raw/current/scripts/prefabs/meats.lua#L138-L222)、[漂浮默认](../../../raw/current/scripts/standardcomponents.lua#L1772-L1791)、[堆叠默认](../../../raw/current/scripts/components/stackable.lua#L23-L34)

`dryable`／`cookable` 的标签在 `SetPristine` **之前**加，组件在**主模拟分支**里加：客户端只有标签、没有组件。动作判定本身也只查标签——肉架槽位测试看 `dryable`，烤制动作处理器看 `target:HasTag("cookable")`。[标签段](../../../raw/current/scripts/prefabs/meats.lua#L161-L172)、[组件段](../../../raw/current/scripts/prefabs/meats.lua#L201-L212)、[肉架](../../../raw/current/scripts/containers.lua#L234-L244)、[烤制](../../../raw/current/scripts/componentactions.lua#L1113-L1118)

### 名称核验

`drumstick` 的简中实显是**鸟腿**（英文 Drumstick），`drumstick_cooked` 是**炸鸟腿**（Fried Drumstick）；两个键各命中一次，注册名与 `STRINGS.NAMES` 键同名。[DRUMSTICK](../../../raw/current/scripts/languages/chinese_s.po#L342289-L342292)、[DRUMSTICK_COOKED](../../../raw/current/scripts/languages/chinese_s.po#L342294-L342297)

## 数值与获得

时间基准：`seg_time`＝30 秒、`total_day_time`＝480 秒、`perish_warp`＝1，所以鲜度与干制时长都按 480 秒/天换算。[基准](../../../raw/current/scripts/tuning.lua#L19-L20)、[`perish_warp`](../../../raw/current/scripts/tuning.lua#L47-L47)

| 注册 | 饥饿／生命／理智 | 鲜度 | 堆叠 | 漂浮 | 转换 |
|---|---|---|---|---|---|
| `drumstick` | 12.5／0／−10 | `PERISH_FAST`＝6 天＝2880 秒 | 20 | `small`，偏移 0.2（仅服务端副本） | 干制 `DRY_FAST`＝1 天＝480 秒 → `smallmeat_dried`；烤制 → `drumstick_cooked` |
| `drumstick_cooked` | 12.5／1／0 | `PERISH_MED`＝10 天＝4800 秒 | 20 | `small`，偏移 0.15、缩放 0.85（仅服务端副本） | 无（无 `dryable`／`cookable`） |

数值来源：[生腿数值](../../../raw/current/scripts/prefabs/meats.lua#L472-L478)、[熟态数值](../../../raw/current/scripts/prefabs/meats.lua#L490-L495)、[常量](../../../raw/current/scripts/tuning.lua#L2021-L2023)、[热量](../../../raw/current/scripts/tuning.lua#L2058-L2060)、[理智](../../../raw/current/scripts/tuning.lua#L2033-L2034)、[鲜度](../../../raw/current/scripts/tuning.lua#L2043-L2051)、[干制](../../../raw/current/scripts/tuning.lua#L2053-L2056)、[堆叠](../../../raw/current/scripts/tuning.lua#L82-L82)。采集册同项写作 `stacksize=20`、`hungervalue=12.5`、`healthvalue=0/1`、`sanityvalue=-10/0`、`perishable=2880/4800`，与源码一致。[采集册](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L429-L430)

晾肉架只接收带 `dryable` 标签的物品（判定写在容器槽位测试里），所以生腿能上架、炸鸟腿会被拒。主模拟另有两条例外（物品刚生成 `GetTimeAlive() == 0`、或从本架原位挪回），它们只影响能否入槽：`dryingrack` 只有在物品带 `dryable` 组件时才真正开始计时，缺组件时只显示不入计时。[肉架接收条件](../../../raw/current/scripts/containers.lua#L234-L244)、[例外](../../../raw/current/scripts/containers.lua#L236-L243)、[计时条件](../../../raw/current/scripts/components/dryingrack.lua#L236-L270)

漂浮参数由各注册自己在工厂里写：生腿只改垂直偏移 0.2，熟态改 0.15 并缩放 0.85。但这两处写在主模拟早退（`if not TheWorld.ismastersim then return end`）之后，而 `floater` 的 `size`／`vert_offset`／`xscale` 都是普通字段、没有 netvar，所以只有服务端副本生效；客户端副本仍是默认 `small`／0／1 渲染。[生腿](../../../raw/current/scripts/prefabs/meats.lua#L468-L478)、[熟态](../../../raw/current/scripts/prefabs/meats.lua#L486-L495)、[无 netvar](../../../raw/current/scripts/components/floater.lua#L30-L45)、[入参写入](../../../raw/current/scripts/components/floater.lua#L52-L66)、[特效挂点](../../../raw/current/scripts/components/floater.lua#L107-L117)

### 获得路径

| 注册 | 已核验来源 | 产出与证据 |
|---|---|---|
| `drumstick` | ①火鸡（`perd`）战利品固定两份；②秃鹫（`buzzard`）共享掉落表 1.00；③`moose`（麋鹿鹅，显示名键为 `NAMES.MOOSE1`／`MOOSE2`）两份 1.00；④麋鹿鹅幼崽（`mossling`）1.00；⑤企鸥（`penguin`）0.1 | ①[perd](../../../raw/current/scripts/prefabs/perd.lua#L17-L21)、②[buzzard](../../../raw/current/scripts/prefabs/buzzard.lua#L19-L24)、③[moose](../../../raw/current/scripts/prefabs/moose.lua#L26-L35)、④[mossling](../../../raw/current/scripts/prefabs/mossling.lua#L22-L25)、⑤[penguin](../../../raw/current/scripts/prefabs/penguin.lua#L40-L40) |
| `drumstick_cooked` | ①`cookable.product` 指向：在带 `cooker` 标签的实体（营火、火坑、龙鳞火炉（`dragonflyfurnace`，文件 `prefabs/dragonfurnace.lua`）等）上烤；烹饪锅只有 `stewer` 标签与组件，不提供烤制；②掉落者已带 `burnt` 标签，或正在燃烧且（无 `fueled` 组件或 `burnable.ignorefuel` 为真）时，`DropLoot` 会把 `drumstick` 换成 `drumstick_cooked` | ①[接线](../../../raw/current/scripts/prefabs/meats.lua#L466-L466)、[烤制门槛](../../../raw/current/scripts/componentactions.lua#L1096-L1104)、[烹饪锅](../../../raw/current/scripts/prefabs/cookpot.lua#L290-L309)；②[门禁](../../../raw/current/scripts/components/lootdropper.lua#L415-L421)、[换名规则](../../../raw/current/scripts/components/lootdropper.lua#L424-L431) |

普通鸟（乌鸦、红雀、金丝雀等）的随机战利品是 `smallmeat` 与对应羽毛，不是鸟腿：`makebird` 里 `AddRandomLoot` 写的是 `feather_<name>` 与 `smallmeat`。[鸟类战利品](../../../raw/current/scripts/prefabs/birds.lua#L319-L327)

用途补充：鸟腿在烹饪锅登记 `meat = 0.5`；同一行的 `cancook` 会按名字派生一份 `drumstick_cooked` 的食材条目（`meat = 0.5`＋`precook = 1`），而烹饪锅槽位只判 `cooking.IsCookingIngredient(prefab)`，所以炸鸟腿能进锅、只算半份肉，只是不能再回炉烤。[食材值](../../../raw/current/scripts/cooking.lua#L108-L108)、[派生规则](../../../raw/current/scripts/cooking.lua#L46-L65)、[槽位判定](../../../raw/current/scripts/containers.lua#L373-L375)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：edible | 写饥饿 12.5、生命 0、理智 −10（熟态 12.5／1／0）与 `FOODTYPE.MEAT`；理智为负，赋值即加 `badfood`，因而不能喂给其他玩家（PVP 等例外） | 仅主模拟；`foodtype` 变化会同步 `edible_MEAT` 标签 | [[reference/components/edible#关键接口]]；[赋值](../../../raw/current/scripts/prefabs/meats.lua#L472-L474)、[`badfood`](../../../raw/current/scripts/components/edible.lua#L1-L7)、[喂食门槛](../../../raw/current/scripts/actions.lua#L2406-L2410) |
| 组件：perishable | `SetPerishTime`＋`StartPerishing`；腐坏替换成 `spoiled_food` | 生腿 2880 秒、熟态 4800 秒；新鲜／陈旧／腐烂按百分比 0.5／0.2 切标签 | [[reference/components/perishable#关键接口]]；[鲜度](../../../raw/current/scripts/prefabs/meats.lua#L476-L476)、[阈值](../../../raw/current/scripts/tuning.lua#L1647-L1648) |
| 组件：dryable | `product=smallmeat_dried`、`time=DRY_FAST`；`build`／`dried_build` 未传，肉架显示回落默认 build | 仅生腿；组件在主模拟，标签在两端 | [[reference/components/dryable#关键接口]]；[接线](../../../raw/current/scripts/prefabs/meats.lua#L466-L466)、[默认 build](../../../raw/current/scripts/components/dryingrack.lua#L62-L76) |
| 组件：cookable | `product=drumstick_cooked`；`Cook()` 生成产物并把鲜度折半补回；同一标签还决定能否进盐盒（要求 fresh／stale／spoiled＋`cookable`），故生腿可、炸鸟腿不可 | 仅生腿；烤后换成 `cooked` 动画 | [[reference/components/cookable#关键接口]]；[接线](../../../raw/current/scripts/prefabs/meats.lua#L466-L466)、[折算](../../../raw/current/scripts/components/cookable.lua#L28-L36)、[盐盒](../../../raw/current/scripts/containers.lua#L1049-L1055) |
| 组件：stackable | 未改 `maxsize`，沿用 20（`STACK_SIZE_MEDITEM`） | 两端由 replica 同步堆叠数 | [[reference/components/stackable#关键接口]]；[默认值](../../../raw/current/scripts/components/stackable.lua#L28-L28)、[常量](../../../raw/current/scripts/tuning.lua#L82-L82) |
| 组件：tradable | `goldvalue = GOLD_VALUES.MEAT`＝1 | 主模拟；熟态同样为 1（未覆盖） | [[reference/components/tradable#关键接口]]；[`GOLD_VALUES`](../../../raw/current/scripts/tuning.lua#L1946-L1952) |
| 组件：bait／hauntable | `bait` 让物品能进陷阱当饵；`MakeHauntableLaunchAndPerish` 的作祟只抛飞 | 作祟的加速腐烂分支被注释，实际只 `Launch` | [[reference/components/bait#关键接口]]；[bait](../../../raw/current/scripts/prefabs/meats.lua#L186-L186)、[作祟](../../../raw/current/scripts/standardcomponents.lua#L1311-L1337) |
| 标签：`drumstick`／`catfood`／`rawmeat` | `catfood` 让浣猫把物品当猫粮接受；`rawmeat` 让 `eatsrawmeat` 的吃者不结算负的生命／理智；`drumstick` 在快照内无 `HasTag` 消费者 | 浣猫判定在客户端与服务端都可跑；`DoFoodEffects` 只在结算负值时使用 | [浣猫](../../../raw/current/scripts/prefabs/catcoon.lua#L273-L279)、[吃者](../../../raw/current/scripts/components/eater.lua#L166-L170)、[结算](../../../raw/current/scripts/components/eater.lua#L211-L242) |
| 关联 Prefab：`smallmeat_dried` | 干制产物；未写进 `drumstickprefabs` 预载列表，但与鸟腿同在一个文件里返回，实际不缺 prefab | 肉架到点后生成 | [干制产物](../../../raw/current/scripts/prefabs/meats.lua#L466-L466)、[预载列表](../../../raw/current/scripts/prefabs/meats.lua#L60-L64)、[同文件注册](../../../raw/current/scripts/prefabs/meats.lua#L907-L907) |
| 资源：`anim/drumstick.zip` | 生／熟两种动画共用同一 bank 与 build，只切 `raw`／`cooked` 动画 | 原版资源，不随页分发 | [assets](../../../raw/current/scripts/prefabs/meats.lua#L7-L7) |

## 典型接入

### 把鸟腿的干制产物换成肉干

- 前提与选择：服务端 `modmain.lua` 的 postinit。要改的是 `dryable` 组件的产物与时长，而不是 `TUNING.DRY_FAST`（后者会连带改所有用它的物品，例如 `smallmeat`、`monstermeat`）。
- 生效与撤销：只影响之后在主模拟上完成干制的鸟腿；已经干完的产物不变。卸下模组后回到 `smallmeat_dried`。产物是在完成回调里按 `GetProduct()` 现取，所以挂架之后再改也生效。
- 原版依据：`common()` 用 `dryable.product`／`time` 初始化组件；完成回调 `OnDoneDrying` 到点才 `SpawnPrefab(product)`。[接线](../../../raw/current/scripts/prefabs/meats.lua#L201-L207)、[完成回调](../../../raw/current/scripts/components/dryingrack.lua#L198-L224)

```lua
-- modmain.lua：鸟腿改干成肉干，时长改成 DRY_MED
AddPrefabPostInit("drumstick", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    local dryable = inst.components.dryable
    if dryable ~= nil then
        dryable:SetProduct("meat_dried")
        dryable:SetDryTime(GLOBAL.TUNING.DRY_MED)
    end
end)
```

- 调整与验证：`DRY_FAST`＝480 秒、`DRY_MED`＝960 秒；实测看架子上到点生成的是 `meat_dried` 还是 `smallmeat_dried`，不要只看组件字段。

## 常见陷阱与限制

- **误区：杀普通鸟能拿鸟腿。** `makebird` 的随机战利品是羽毛与 `smallmeat`，鸟腿来自火鸡、秃鹫、企鹅、麋鹿鹅父子。[鸟类战利品](../../../raw/current/scripts/prefabs/birds.lua#L319-L327)
- **边界：炸鸟腿不能再烤、也不能上架干制。** 它连 `cookable`／`dryable` 标签都没有，肉架的槽位测试会直接拒绝。主模拟的「刚生成」例外最多让它落进槽位，因为缺 `dryable` 组件也不会开始计时（[例外](../../../raw/current/scripts/containers.lua#L236-L243)、[计时条件](../../../raw/current/scripts/components/dryingrack.lua#L236-L270)）。[熟态](../../../raw/current/scripts/prefabs/meats.lua#L484-L484)、[接收条件](../../../raw/current/scripts/containers.lua#L234-L244)
- **误区：`edible.ismeat` 会改变谁能吃。** 全快照 grep `ismeat` 共 15 行：11 处赋值、1 条「似乎没被用到」的注释，另有 3 行是 `meatrack2` 的同名**局部变量**（`local ismeat = foodtype == FOODTYPE.MEAT`），没有按该字段读取的分支；食性由 `foodtype`／`edible_MEAT` 标签与吃者的 diet 决定。[写入](../../../raw/current/scripts/prefabs/meats.lua#L183-L184)、[注释](../../../raw/current/scripts/prefabs/oceanfish.lua#L518-L519)
- **边界：`MakeHauntableLaunchAndPerish` 不会让食物更快腐烂。** 函数名里的 perish 分支被 `#HAUNTFIX` 注释掉，作祟只抛飞并给 `HAUNT_TINY`。[作祟](../../../raw/current/scripts/standardcomponents.lua#L1324-L1332)
- **未核验**：本页未运行游戏；干制链路只按源码推导，未实测肉架在多雨／酸性环境下的暂停行为。

## 验证记录

- 源码审阅：2026-11-09，Steam buildid 26474873。核对 `meats.lua` 的 `common()` 与 `drumstick`／`drumstick_cooked` 两个工厂及返回表；`edible`／`perishable`／`stackable`／`dryable`／`cookable`／`floater`／`bait`／`eater` 的相关分支；`dryingrack` 的接收与完成回调、`containers.lua` 的 `meatrack.itemtestfn`；`perd`／`buzzard`／`moose`／`mossling`／`penguin`／`birds`／`catcoon` 的掉落与接受条件；`cooking.lua` 的食材值；`tuning` 的鲜度／干制／重伤／热量／理智／堆叠／金值常量；`scrapbookdata` 交叉核对。
  复审修订（同日）：按独立复审补核 `cooking.lua` 的 `cancook` 派生与 `IsCookingIngredient`、`floater` 无 netvar 的端差异、`dryingrack` 的计时条件、盐盒的 `cookable` 门槛，并修正 `ismeat` 的写入计数（11 处赋值＋1 条注释＋3 行局部变量）。
  复检修订（同日，第二轮）：把烤制入口改为「带 `cooker` 标签的实体」并说明烹饪锅不提供烤制（`componentactions.lua` L1096-L1104、`cookpot.lua` L290-L309），补 `DropLoot` 的燃烧换名来源（`lootdropper.lua` L415-L431）。
  复检修订（同日，第四轮）：`dragonfurnace` 的注册名实为 `dragonflyfurnace`（文件 `prefabs/dragonfurnace.lua` L238），烤制入口处按「中文名（注册名，文件）」改写。
  名称核验：`languages/chinese_s.po` 的 `NAMES.DRUMSTICK`＝鸟腿、`NAMES.DRUMSTICK_COOKED`＝炸鸟腿；两个注册名与 `NAMES` 键同名，未额外查动态选名。[po 锚点](#名称核验)
  未覆盖：`MOOSE1`／`MOOSE2` 一类的显示名切换（`moose` 无 `NAMES.MOOSE` 键）、模组注入与活动脚本；肉架皮肤与多格架体行为。
- 游戏运行：未运行。天数、秒数与三维均为 tuning 推导（`total_day_time`＝480 秒）。
- 结构检查：`validate_page.py` 通过（standard 档内）；`validate_source_links.py --strict` 0 错 0 警。

## 相关链接

- 可食与鲜度：[[reference/components/edible]]、[[reference/components/perishable]]
- 两条转换：[[reference/components/dryable]]、[[reference/components/cookable]]、[[reference/components/stackable]]
- 携带与交易：[[reference/components/tradable]]、[[reference/components/bait]]

返回 [[reference/prefabs/index|Prefab 索引]]。
