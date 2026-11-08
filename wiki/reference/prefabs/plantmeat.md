---
title: "叶肉与熟叶肉（plantmeat、plantmeat_cooked）"
created: "2026-11-09"
updated: "2026-11-09"
type: prefab
summary: "meats.lua 的 plantmeat 与 plantmeat_cooked：共用 common() 下仍是 MEAT 食性的植物来源肉，显式带的 lureplant_bait 标签，食人花诱饵与两种小型海鱼、胡萝卜鼠的获得／烤制关系，以及它为何不能上晾肉架干制。"
aliases: ["plantmeat", "plantmeat_cooked", "叶肉", "熟叶肉", "食人花叶肉"]
tags: [prefab, item, food]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/meats.lua", "prefabs/oceanfishdef.lua", "prefabs/oceanfish.lua", "prefabs/carrat.lua", "actions.lua", "components/edible.lua", "components/perishable.lua", "components/stackable.lua", "components/cookable.lua", "components/floater.lua", "components/bait.lua", "components/eater.lua", "components/dryable.lua", "containers.lua", "standardcomponents.lua", "prefabs/lureplant.lua", "components/shelf.lua", "components/thief.lua", "components/digester.lua", "cooking.lua", "recipes.lua", "tuning.lua", "languages/chinese_s.po", "screens/redux/scrapbookdata.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 叶肉与熟叶肉（plantmeat、plantmeat_cooked）

## 快速判断

- 职责与适用：两个注册都由 `meats.lua` 的 `common()` 生成，是可拾取食物；`plantmeat` 显式带 `lureplant_bait` 与 `rawmeat` 标签、可烤成 `plantmeat_cooked`，但**不可干制**（`dryable` 传 `nil`）。[生叶肉构造](../../../raw/current/scripts/prefabs/meats.lua#L636-L650)、[熟叶肉构造](../../../raw/current/scripts/prefabs/meats.lua#L652-L664)
- 优先使用：要判断能不能上肉架看[为什么不可干制](#为什么不可干制)；改数值看[数值与获得](#数值与获得)；查它与食人花的关系看[获得路径](#获得路径)。
- 重要副作用：`common()` 把 `foodtype` 写死 `FOODTYPE.MEAT`，所以它是「植物来源但按肉结算」的食物；熟态把理智从 −10 拉回 0。[食性](../../../raw/current/scripts/prefabs/meats.lua#L182-L184)、[熟态](../../../raw/current/scripts/prefabs/meats.lua#L659-L661)
- 修改边界：只讲这两个注册；食人花本体（诱饵展示、消化、小兵与战斗）归其自身主题页，本页只记获得与诱饵关系。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `plantmeat` | `plantmeat()` 调 `common("plant_meat", "plant_meat", "raw", {"lureplant_bait", "rawmeat"}, nil, {product="plantmeat_cooked"})`：按 `common(bank, build, anim, tags, dryable, cookable)` 的形参顺序，`tags` 之后第 5 个实参 `nil` 是 `dryable`，第 6 个实参才是 `cookable` | 标签：`meat`＋`lureplant_bait`／`rawmeat`＋`cookable`；因 `dryable` 为 `nil`，`common()` 不会补 `dryable` 与 `lureplant_bait`，这两个标签里只有 `lureplant_bait` 是显式写死在 `tags` 表里的；主模拟挂 `edible`（`ismeat=true`、`MEAT`）、`bait`、`inspectable`、`inventoryitem`、`stackable`、`tradable`、`perishable`、`cookable`，无 `dryable`。[构造](../../../raw/current/scripts/prefabs/meats.lua#L636-L650)、[标签分支](../../../raw/current/scripts/prefabs/meats.lua#L161-L172) |
| `plantmeat_cooked` | `plantmeat_cooked()` 调 `common("plant_meat", "plant_meat", "cooked")`：`tags`、`dryable`、`cookable` 全缺省 | 只剩共用 `meat` 标签；没有 `lureplant_bait`，因此熟叶肉不再被食人花当作诱饵；无 `dryable`／`cookable` 入口。[熟态](../../../raw/current/scripts/prefabs/meats.lua#L652-L664) |

`common()` 的共用部分：写 bank／build／anim、加 `meat` 标签、`MakeInventoryFloatable(inst)`（默认 `small`、无偏移；这一步在 `SetPristine()` **之前**，因此两端都会挂上 `floater`，客户端副本就是这套默认值）、`SetPristine`、非主模拟返回；主模拟统一挂 `edible`（`ismeat=true`、`FOODTYPE.MEAT`）、`bait`、`inspectable`、`inventoryitem`、`stackable`（默认 20）、`tradable`（金值 1）、`perishable`（腐坏→`spoiled_food`）。[工厂](../../../raw/current/scripts/prefabs/meats.lua#L138-L222)、[漂浮默认](../../../raw/current/scripts/standardcomponents.lua#L1772-L1791)

### 名称核验

`plantmeat` 的简中实显是**叶肉**（Leafy Meat），`plantmeat_cooked` 是**熟叶肉**（Cooked Leafy Meat）；两个键各命中一次，注册名与 `STRINGS.NAMES` 键同名。[PLANTMEAT](../../../raw/current/scripts/languages/chinese_s.po#L346984-L346987)、[PLANTMEAT_COOKED](../../../raw/current/scripts/languages/chinese_s.po#L346989-L346992)

注意源码里 prefab 名是 `plantmeat`，而 bank／build 是 `plant_meat`（下划线）；两者不能混用。[构造](../../../raw/current/scripts/prefabs/meats.lua#L637-L637)、[资源](../../../raw/current/scripts/prefabs/meats.lua#L13-L13)

## 数值与获得

时间基准：`seg_time`＝30 秒、`total_day_time`＝480 秒，下面天数按 480 秒/天换算。[基准](../../../raw/current/scripts/tuning.lua#L19-L20)

| 注册 | 饥饿／生命／理智 | 鲜度 | 堆叠 | 漂浮 | 转换 |
|---|---|---|---|---|---|
| `plantmeat` | 12.5／0／−10 | `PERISH_FAST`＝6 天＝2880 秒 | 20 | 未设，默认 `small`／无偏移／1 | 烤制 → `plantmeat_cooked`；无干制 |
| `plantmeat_cooked` | 18.75／1／0 | `PERISH_MED`＝10 天＝4800 秒 | 20 | 未设，默认 `small` | 无 |

数值来源：[生叶肉](../../../raw/current/scripts/prefabs/meats.lua#L643-L647)、[熟叶肉](../../../raw/current/scripts/prefabs/meats.lua#L659-L661)、[常量](../../../raw/current/scripts/tuning.lua#L2021-L2023)、[热量](../../../raw/current/scripts/tuning.lua#L2058-L2060)、[理智](../../../raw/current/scripts/tuning.lua#L2033-L2034)、[鲜度](../../../raw/current/scripts/tuning.lua#L2043-L2051)、[堆叠](../../../raw/current/scripts/tuning.lua#L82-L82)。采集册同值：`hungervalue=12.5/18.75`、`healthvalue=0/1`、`sanityvalue=-10/0`、`perishable=2880/4800`。[采集册](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L1129-L1130)

两件都没有写 `floater` 参数，所以漂浮特效按组件默认的 `small` 尺寸、零偏移、1 倍缩放渲染。[漂浮参数](../../../raw/current/scripts/components/floater.lua#L178-L194)

### 为什么不可干制

三处缺一不可，缺任何一处都上不了肉架：

1. `dryable` 参数为 `nil`，所以 `common()` 不会 `AddComponent("dryable")`。[组件段](../../../raw/current/scripts/prefabs/meats.lua#L201-L207)
2. 同一分支也不会补 `dryable` 标签（只有 `dryable.product` 非空时才加）。[标签段](../../../raw/current/scripts/prefabs/meats.lua#L161-L167)
3. 晾肉架的接收判定只看 `dryable` 标签，缺标签直接被拒；主模拟另有两条入槽例外（刚生成的物品、或从本架原位挪回），但它们不改变结论，因为缺组件时 `dryingrack` 不会开始计时。[接收条件](../../../raw/current/scripts/containers.lua#L234-L244)、[例外](../../../raw/current/scripts/containers.lua#L236-L243)、[计时条件](../../../raw/current/scripts/components/dryingrack.lua#L236-L270)

所以叶肉上不了肉架：它既没有组件也没有标签，槽位测试直接返回假；即使靠主模拟的「刚生成」例外落进槽位，缺 `dryable` 组件也不会开始计时。[构造](../../../raw/current/scripts/prefabs/meats.lua#L637-L637)

### 获得路径

| 注册 | 已核验来源 | 产出与证据 |
|---|---|---|
| `plantmeat` | ①食人花（`lureplant`）展示诱饵时，若自己的物品栏里没有带 `lureplant_bait` 的东西，且 `numminions * 2 >= maxminions`（小兵达到上限的一半；上限为 0 时也成立，源码注释特意保留这一旧行为），就 `SpawnPrefab("plantmeat")` 放进物品栏，再挂成可被拿走的货架物；②落叶比目鱼与花朵金枪鱼（`oceanfish_small_6`／`oceanfish_small_7`）的 `_inv` 形态把 `loot` 设成 `{"plantmeat"}`，宰杀时掉落 | ①[选择诱饵](../../../raw/current/scripts/prefabs/lureplant.lua#L178-L196)、[挂架](../../../raw/current/scripts/prefabs/lureplant.lua#L43-L54)；②[掉落表](../../../raw/current/scripts/prefabs/oceanfishdef.lua#L116-L116)、[秋季鱼](../../../raw/current/scripts/prefabs/oceanfishdef.lua#L417-L451)、[春季鱼](../../../raw/current/scripts/prefabs/oceanfishdef.lua#L462-L496)、[`_inv` 掉落](../../../raw/current/scripts/prefabs/oceanfish.lua#L496-L499) |
| `plantmeat_cooked` | 由各生品的 `cookable.product`／`cooking_product` 指向：①叶肉 `plantmeat`；②胡萝卜鼠（`carrat`）；③上述两种小鱼的 `_inv`（`cooking_product = plantmeat_cooked`） | ①[叶肉](../../../raw/current/scripts/prefabs/meats.lua#L637-L637)、②[carrat](../../../raw/current/scripts/prefabs/carrat.lua#L481-L483)、③[烤制产物](../../../raw/current/scripts/prefabs/oceanfishdef.lua#L146-L146) |

拿取路径：`TryRevealBait` 把 `shelf.cantakeitem` 置真、`shelf.itemonshelf` 指向这份叶肉；玩家取走时 `Shelf:TakeItem` 调 `ontakeitemfn`（此处绑定为 `OnPicked`）并把物品交给取用者，食人花随之收起诱饵。另有 `onitemstolen` 监听：被小偷（`thief` 组件）偷走时，只有带 `lureplant_bait` 标签的物品才触发同一收起逻辑。[shelf](../../../raw/current/scripts/components/shelf.lua#L42-L66)、[绑定](../../../raw/current/scripts/prefabs/lureplant.lua#L373-L375)、[偷窃事件](../../../raw/current/scripts/components/thief.lua#L33-L33)、[收起](../../../raw/current/scripts/prefabs/lureplant.lua#L130-L135)

用途补充：烹饪锅把叶肉与熟叶肉都记为 `meat=1`；沃姆伍德的技能树配方 `wormwood_lureplant` 需要 5 份叶肉。[食材值](../../../raw/current/scripts/cooking.lua#L157-L157)、[配方](../../../raw/current/scripts/recipes.lua#L348-L348)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：edible | 写三维与 `FOODTYPE.MEAT`；生叶肉理智 −10、熟叶肉 0 | 仅主模拟；`foodtype` 变化同步 `edible_MEAT` 标签；生叶肉因理智为负加 `badfood`，不能喂给其他玩家 | [[reference/components/edible#关键接口]]；[生](../../../raw/current/scripts/prefabs/meats.lua#L643-L645)、[熟](../../../raw/current/scripts/prefabs/meats.lua#L659-L660)、[`badfood`](../../../raw/current/scripts/components/edible.lua#L1-L7)、[喂食门槛](../../../raw/current/scripts/actions.lua#L2406-L2410) |
| 组件：perishable | 2880／4800 秒，腐坏替换 `spoiled_food` | 仅主模拟；按 0.5／0.2 百分比切新鲜度标签 | [[reference/components/perishable#关键接口]]；[生](../../../raw/current/scripts/prefabs/meats.lua#L647-L647)、[熟](../../../raw/current/scripts/prefabs/meats.lua#L661-L661) |
| 组件：cookable | 仅生叶肉有，`product=plantmeat_cooked`；`Cook()` 把鲜度折半补回产物；同一标签还决定能否进盐盒，故生叶肉可、熟叶肉不可 | 仅生叶肉；熟态不再可烤 | [[reference/components/cookable#关键接口]]；[接线](../../../raw/current/scripts/prefabs/meats.lua#L637-L637)、[折算](../../../raw/current/scripts/components/cookable.lua#L28-L36)、[盐盒](../../../raw/current/scripts/containers.lua#L1049-L1055) |
| 组件：dryable | **未挂载**：无组件、无标签，肉架拒收 | 两端一致，都因缺标签上不了架 | [[reference/components/dryable#关键接口]]；[接收条件](../../../raw/current/scripts/containers.lua#L234-L244) |
| 组件：bait | `common()` 统一挂，供陷阱当饵 | 仅主模拟；没被陷阱引用时 `IsFree()` 为真 | [[reference/components/bait#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/meats.lua#L186-L186)、[状态](../../../raw/current/scripts/components/bait.lua#L42-L44) |
| 标签：`lureplant_bait` | 食人花选诱饵时把它当可用诱饵；被拿走则收起 | 仅生叶肉有；熟叶肉失去该标签 | [选择](../../../raw/current/scripts/prefabs/lureplant.lua#L180-L186)、[收起](../../../raw/current/scripts/prefabs/lureplant.lua#L130-L135) |
| 标签：`rawmeat` | 让 `eatsrawmeat` 吃者跳过负生命／理智结算 | 生叶肉有，熟态无 | [吃者](../../../raw/current/scripts/components/eater.lua#L166-L170)、[结算](../../../raw/current/scripts/components/eater.lua#L211-L242) |
| 关联 Prefab：`lureplant` | 叶肉的产出方与使用者；`digester` 每 20 秒按 prefab 名扣 1 件库存。`CanDigest` 只是候选筛选：展示中的堆（`shelf.itemonshelf`）在堆叠 ≤5 时**不提供候选**、>5 才进候选；入选后 `ConsumeByName` 按 prefab 名从前往后扣件，同名另一叠入选时仍可能扣到这堆展示物 | 仅主模拟；诱饵被收回时不再可拿 | [消化](../../../raw/current/scripts/components/digester.lua#L14-L39)、[`CanDigest`](../../../raw/current/scripts/prefabs/lureplant.lua#L207-L213)、[`ConsumeByName`](../../../raw/current/scripts/components/inventory.lua#L1772-L1785) |
| 关联 Prefab：`spoiled_food` | 腐坏替换物 | 主模拟到点生成 | [腐坏](../../../raw/current/scripts/prefabs/meats.lua#L199-L199) |
| 资源：`anim/plant_meat.zip` | 生／熟共用 `plant_meat` bank／build，只切 `raw`／`cooked` 动画 | 原版资源，不随页分发 | [assets](../../../raw/current/scripts/prefabs/meats.lua#L13-L13) |

## 典型接入

### 让叶肉也能上晾肉架

- 前提与选择：服务端 `modmain.lua` 的 postinit。要同时补服务端组件与两端的 `dryable` 标签：肉架判定看标签，实际干制看组件。[接收条件](../../../raw/current/scripts/containers.lua#L234-L244)、[组件段](../../../raw/current/scripts/prefabs/meats.lua#L201-L207)
- 生效与撤销：只影响新生成的叶肉；已经存在的物品需要重新进世界。卸下模组后组件与标签都不再添加，已干制产物不受影响。
- 原版依据：`common()` 在 `SetPristine` 之前加 `dryable` 标签、在主模拟分支加组件；postinit 晚于这两步，所以这里显式补标签（两端）再补组件（仅服务端）。

```lua
-- modmain.lua：给叶肉补上干制能力（产物为小肉干，时长 DRY_MED）
AddPrefabPostInit("plantmeat", function(inst)
    inst:AddTag("dryable") -- 肉架只按这个标签决定能否接收
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    if inst.components.dryable == nil then
        local dryable = inst:AddComponent("dryable")
        dryable:SetProduct("smallmeat_dried")
        dryable:SetDryTime(GLOBAL.TUNING.DRY_MED)
    end
end)
```

- 调整与验证：`DRY_MED`＝960 秒；若想让熟叶肉也能干制，需要另对 `plantmeat_cooked` 做同样处理。实测看能否把叶肉拖进肉架槽位、到点是否生成 `smallmeat_dried`；本页未运行验证该补丁。

## 常见陷阱与限制

- **误区：叶肉可以晾成肉干。** 它没有 `dryable` 组件也没有标签，肉架提交会被拒（主模拟的两条入槽例外见 [`itemtestfn`](../../../raw/current/scripts/containers.lua#L236-L243)，但不带组件也不会开始计时）。[构造](../../../raw/current/scripts/prefabs/meats.lua#L637-L637)、[接收条件](../../../raw/current/scripts/containers.lua#L234-L244)
- **误区：植物来源的肉按素食结算。** `common()` 无条件写 `foodtype = FOODTYPE.MEAT`，烹饪锅也按 `meat=1` 计；吃者按肉处理。[食性](../../../raw/current/scripts/prefabs/meats.lua#L182-L184)、[食材值](../../../raw/current/scripts/cooking.lua#L157-L157)
- **边界：`lureplant_bait` 来自 `tags` 表，不是 `common()` 的自动补标签。** 自动补 `lureplant_bait` 只要求第 5 参 `dryable` 非空；`dryable.product` 非空才决定 `dryable` 标签与组件。叶肉第 5 参传的是 `nil`，两条自动分支都不触发，两者容易混淆。[自动补标签](../../../raw/current/scripts/prefabs/meats.lua#L161-L167)、[组件分支](../../../raw/current/scripts/prefabs/meats.lua#L201-L212)、[显式标签](../../../raw/current/scripts/prefabs/meats.lua#L637-L637)
- **边界：熟叶肉不再是诱饵。** `plantmeat_cooked` 的 `tags` 缺省，只剩 `meat`。[熟态](../../../raw/current/scripts/prefabs/meats.lua#L653-L653)
- **未核验**：本页未运行游戏；食人花诱饵的展示时机与小兵数量门槛只按源码推导，未实测拿取后的收起表现。

## 验证记录

- 源码审阅：2026-11-09，Steam buildid 26474873。核对 `meats.lua` 的 `common()`、`plantmeat`／`plantmeat_cooked` 两个工厂与返回表；`edible`／`perishable`／`stackable`／`cookable`／`floater`／`bait`／`eater`／`dryable` 的相关分支；`containers.lua` 的肉架接收条件；`lureplant` 的诱饵选择、货架挂载与拿取回调；`shelf`／`digester` 的拿取与消化；`cooking.lua` 与 `recipes.lua` 的用途；`tuning` 的鲜度／干制／热量／理智／堆叠常量；`scrapbookdata` 交叉核对。
  复审修订（同日）：补核两种小型海鱼 `_inv` 的宰杀掉落与 `cooking_product`、`carrat` 的 `cookable.product`、肉架入槽例外与计时条件、盐盒门槛。
  复检修订（同日，第二轮）：修正 `common()` 形参序号（第 5 参 `dryable`、第 6 参 `cookable`），补 `MakeInventoryFloatable` 在 `SetPristine` 前于两端挂载，改写 `CanDigest` 的堆叠 ≤5 门槛。
  名称核验：`languages/chinese_s.po` 的 `NAMES.PLANTMEAT`＝叶肉、`NAMES.PLANTMEAT_COOKED`＝熟叶肉；两个注册名与 `NAMES` 键同名。[po 锚点](#名称核验)
  未覆盖：食人花本体的小兵、战斗与休眠机制（只核验与叶肉相关的诱饵与消化部分）；模组注入与动态选名。
- 游戏运行：未运行。天数与三维均为 tuning 推导（`total_day_time`＝480 秒）。
- 结构检查：`validate_page.py` 通过（standard 档内）；`validate_source_links.py --strict` 0 错 0 警。

## 相关链接

- 可食与鲜度：[[reference/components/edible]]、[[reference/components/perishable]]
- 转换与拒收：[[reference/components/cookable]]、[[reference/components/dryable]]
- 诱饵与携带：[[reference/components/bait]]、[[reference/components/stackable]]

返回 [[reference/prefabs/index|Prefab 索引]]。
