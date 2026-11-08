---
title: "贡品烤肉、八宝泥馅饼与鱼头串（yotp_food1、yotp_food2、yotp_food3）"
created: "2026-11-09"
updated: "2026-11-09"
type: prefab
size_class: standard
summary: "猪神龛（pigshrine）的三个幸运金块供品：贡品烤肉 150／12／5、鱼头串 75／6／1 各 15 天腐坏，八宝泥馅饼 150／0／0 却是 HORRIBLE 且没有 perishable；三件的标签循环因 data.tags 从未赋值而是死分支，鱼头串另有浮水 override build。"
aliases: ["yotp_food1", "yotp_food2", "yotp_food3", "贡品烤肉", "八宝泥馅饼", "鱼头串", "Tribute Roast", "Eight Treasure Mud Pie", "Fish Heads on a Stick", "猪神龛供品", "猪年食物"]
tags: [prefab, item, food]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/yotp_food.lua", "prefabs/pigshrine.lua", "prefabs/player_common.lua", "prefabs/pigman.lua", "componentactions.lua", "recipes.lua", "recipes_filter.lua", "constants.lua", "tuning.lua", "recipe.lua", "widgets/craftslot.lua", "debugcommands.lua", "components/edible.lua", "components/perishable.lua", "components/tradable.lua", "components/floater.lua", "standardcomponents.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 贡品烤肉、八宝泥馅饼与鱼头串（yotp_food1、yotp_food2、yotp_food3）

## 快速判断

- 职责与适用：`prefabs/yotp_food.lua` 用一张 `foodinfo` 表加 `MakeFood(num)` 循环注册三件猪年供品，都在猪神龛（`pigshrine`）的 `Recipe2` 里用幸运金块换；本页适合查"同工厂多注册但属性／鲜度／标签不对称"的写法。[注册](../../../raw/current/scripts/prefabs/yotp_food.lua#L96-L101)
- 优先使用：先看[注册与构造](#注册与构造)的三行差异，再看[数值与鲜度](#数值与鲜度)的公式；鲜度为何不是整组一致见[常见陷阱](#常见陷阱与限制)。
- 重要副作用：**只有 `yotp_food1`／`yotp_food3` 有 `perishable`**（`PERISH_SLOW`＝15 天）；`yotp_food2` 的 `perishtime` 为空，没有腐坏组件。`yotp_food2` 的食性是 `HORRIBLE`，默认玩家食性 `FOODGROUP.OMNI` 不含它。[数据表](../../../raw/current/scripts/prefabs/yotp_food.lua#L7-L12)、[玩家食性](../../../raw/current/scripts/components/eater.lua#L18-L24)
- 修改边界：猪神龛本体与奖励逻辑、幸运金块来源不展开；本页只负责三个注册的工厂、数值、鲜度、浮水表现与商店入口（[工厂](../../../raw/current/scripts/prefabs/yotp_food.lua#L14-L94)）。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `yotp_food1` | `MakeFood(1)`：`data=foodinfo[1]`，`Prefab("yotp_food1", fn, assets)`（L14-L16、L93） | 贡品烤肉：MEAT；饥饿 `CALORIES_SUPERHUGE`＝150、生命 `HEALING_SMALL*4`＝12、理智 `SANITY_TINY`＝5；`perishtime=PERISH_SLOW`；`floater={"med", 0.9, false}`（L9、L42-L50） |
| `yotp_food2` | 同工厂 `MakeFood(2)`；`Prefab("yotp_food2", …)` | 八宝泥馅饼：HORRIBLE；150／0／0；**无 `perishtime` 字段**，条件不成立所以不加 `perishable`；**无 `floater` 字段**，只保留 `MakeInventoryFloatable` 默认 small／1.0（L10、L40-L50、L66-L71） |
| `yotp_food3` | 同工厂 `MakeFood(3)`；`Prefab("yotp_food3", …)` | 鱼头串：MEAT；`CALORIES_HUGE`＝75、`HEALING_SMALL*2`＝6、`SANITY_SUPERTINY`＝1；`PERISH_SLOW`；`floater={"med", {0.8, 0.5, 0.8}, true}`→按轴缩放并 `AddOverrideBuild("yotp_food_water")`（L11、L42-L50、L80-L84） |

三件共用同一 `fn` 骨架：bank／build 固定 `yotp_food`、动画 `"food"..num`（L27-L31）；两端先加 `pre-preparedfood`（L38）；`MakeInventoryFloatable`→按 `data.floater` 覆盖（L40-L50）；`SetPristine` 后主模拟才加 `edible`／`perishable`／`stackable` 40／`tradable`／`inspectable`／`inventoryitem`（L52-L78），最后 `MakeHauntableLaunch`（L86）。`assets` 同时声明 `yotp_food.zip` 与 `yotp_food_water.zip`（L1-L5）。

### 标签循环是死分支

`foodinfo` 的三项都没有 `tags` 字段，所以 L33-L37 的 `if data.tags ~= nil then … AddTag` 循环一次也不会执行。三件工厂显式加的玩法标签只有 `pre-preparedfood`（L38）＋ `edible_MEAT`／`edible_HORRIBLE`（`edible` 在 `foodtype` 赋值时挂上，[食性标签](../../../raw/current/scripts/components/edible.lua#L9-L17)）；`perishable` 另挂 `fresh`／`stale`／`spoiled` 状态标签（[状态标签](../../../raw/current/scripts/components/perishable.lua#L3-L22)）。不要写成"每件带 `data.tags` 里某个标签"。

### 浮水表现（第三位 true）

`yotp_food3` 的第三位 `true` 走两条：公共端 `AddOverrideBuild("yotp_food_water")`（L47-L49）；主模拟再注册 `floater_startfloating`／`floater_stopfloating`，分别播 `food3_float` 与 `food3`（L80-L84）。事件只由主模拟的 `floater:OnLandedServer`／`OnNoLongerLandedServer` 推（[开始](../../../raw/current/scripts/components/floater.lua#L146-L176)、[结束](../../../raw/current/scripts/components/floater.lua#L214-L222)），客户端 `OnLandedClient` 不推（[客户端分支](../../../raw/current/scripts/components/floater.lua#L178-L194)）。`debugcommands.lua` 的 `d_weirdfloaters` 也把 `yotp_food3` 列入按轴缩放测试名单（L836-L861，条目在 L847）。客户端是否同步看到该动画未核验。

### 名称核验

简中：`NAMES.YOTP_FOOD1`＝**贡品烤肉**、`NAMES.YOTP_FOOD2`＝**八宝泥馅饼**、`NAMES.YOTP_FOOD3`＝**鱼头串**。

| 注册 | 英文 msgid | 简中 msgstr | 证据 |
|---|---|---|---|
| `yotp_food1` | Tribute Roast | 贡品烤肉 | [po](../../../raw/current/scripts/languages/chinese_s.po#L353029-L353032) |
| `yotp_food2` | Eight Treasure Mud Pie | 八宝泥馅饼 | [po](../../../raw/current/scripts/languages/chinese_s.po#L353034-L353037) |
| `yotp_food3` | Fish Heads on a Stick | 鱼头串 | [po](../../../raw/current/scripts/languages/chinese_s.po#L353039-L353042) |

## 数值与鲜度

时间基准：`seg_time`＝30 秒、`total_day_time`＝30×16＝480 秒、`perish_warp`＝1。[基准](../../../raw/current/scripts/tuning.lua#L19-L20)、[`perish_warp`](../../../raw/current/scripts/tuning.lua#L47-L47)

鲜度口径：下表 7200 秒是 modifier=1 的定义值（手持或普通容器内），天数按 480 秒／天换算；`perishable:Update` 在无 owner 时取 `PERISH_GROUND_MULT`＝1.5（约 4800 秒），owner 带 `fridge` 且未冻结时取 `PERISH_FRIDGE_MULT`＝0.5（约 14400 秒），有 `preserver` 组件时优先取它。以上落地秒数按常温、干燥、无酸雨、默认倍率计；落地且 `isacidraining`、无 `rainimmunity`、`CanPointHaveAcidRain` 同时成立时另有额外折旧，潮湿 ×1.3；`ambient_temperature < 0`（未冻结）乘 `PERISH_WINTER_MULT`（0.75），`ambient_temperature > OVERHEAT_TEMP`（70）乘 `PERISH_SUMMER_MULT`（1.25）。剩余比例挂 `fresh`／`stale`／`spoiled` 标签供 `edible` 判衰减。[落地](../../../raw/current/scripts/components/perishable.lua#L97-L97)、[冰箱／preserver](../../../raw/current/scripts/components/perishable.lua#L80-L86)、[酸雨／倍率](../../../raw/current/scripts/components/perishable.lua#L97-L125)、[标签](../../../raw/current/scripts/components/perishable.lua#L3-L22)、[结算](../../../raw/current/scripts/components/perishable.lua#L129-L133)、[倍率](../../../raw/current/scripts/tuning.lua#L2079-L2088)

| 注册 | 食性 | 饥饿 | 生命 | 理智 | 鲜度 | 浮水尺寸／缩放 |
|---|---|---:|---:|---:|---|---|
| `yotp_food1` | MEAT | 150（`CALORIES_SUPERHUGE`＝75×2） | 12（`HEALING_SMALL` 3×4） | 5（`SANITY_TINY`） | `PERISH_SLOW`＝15 天＝7200 秒 → `spoiled_food` | med／0.9 |
| `yotp_food2` | HORRIBLE | 150（75×2） | 0 | 0 | 无 `perishable`，不会腐坏 | 默认 small／1.0 |
| `yotp_food3` | MEAT | 75（`CALORIES_HUGE`） | 6（3×2） | 1（`SANITY_SUPERTINY`） | `PERISH_SLOW`＝15 天＝7200 秒 → `spoiled_food` | med／{0.8, 0.5, 0.8} |

定义与结算：饥饿／生命／理智在工厂里分别写入 `edible.hungervalue`／`healthvalue`／`sanityvalue`（[赋值](../../../raw/current/scripts/prefabs/yotp_food.lua#L58-L64)），实际进食时由 `edible:GetHunger`／`GetHealth`／`GetSanity` 返回（[饥饿](../../../raw/current/scripts/components/edible.lua#L118)、[生命](../../../raw/current/scripts/components/edible.lua#L142)、[理智](../../../raw/current/scripts/components/edible.lua#L93)）；常量值见 [`calories_per_day`](../../../raw/current/scripts/tuning.lua#L41-L41)、[热量档](../../../raw/current/scripts/tuning.lua#L2058-L2065)、[治疗档](../../../raw/current/scripts/tuning.lua#L2021-L2030)、[理智档](../../../raw/current/scripts/tuning.lua#L2032-L2038)、[鲜度档](../../../raw/current/scripts/tuning.lua#L2043-L2051)。堆叠统一 `STACK_SIZE_SMALLITEM`＝40（[接线](../../../raw/current/scripts/prefabs/yotp_food.lua#L73-L74)、[常量](../../../raw/current/scripts/tuning.lua#L83-L83)）。

- **鲜度只在 1／3 上存在**：`if data.perishtime ~= nil and data.perishtime > 0` 才 `AddComponent("perishable")`、`SetPerishTime`、`StartPerishing`、`onperishreplacement="spoiled_food"`（L66-L71）。`yotp_food2` 没有该字段，整段跳过。
- **腐坏从构造开始**：`StartPerishing` 就在构造段里，没有"进背包才计时"或"落地停表"分支；15 天＝7200 秒（定义值；落地约 4800、冰箱约 14400 秒，见上）。[计时实现](../../../raw/current/scripts/components/perishable.lua#L262-L271)、[替换实现](../../../raw/current/scripts/components/perishable.lua#L293-L304)
- **无 `perishable` 就没有 stale／spoiled 倍率**：`GetHunger`／`GetHealth` 的衰减分支只在 `self.inst.components.perishable ~= nil` 时进入（[饥饿](../../../raw/current/scripts/components/edible.lua#L118)），所以 `yotp_food2` 无论放多久都按面板值结算。
- **`tradable` 只挂组件**：未写 `goldvalue`，默认 0（[接线](../../../raw/current/scripts/prefabs/yotp_food.lua#L76-L78)、[默认值](../../../raw/current/scripts/components/tradable.lua#L1-L7)）。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`edible` | 三件各自写 `foodtype`／hunger／health／sanity；`temperaturedelta`／`temperatureduration` 都落到默认 0 | 仅主模拟；`foodtype` 赋值同步 `edible_*` 标签 | [[reference/components/edible]]；[赋值](../../../raw/current/scripts/prefabs/yotp_food.lua#L58-L64)、[食性标签](../../../raw/current/scripts/components/edible.lua#L9-L17) |
| 组件：`perishable` | 仅 1／3：`PERISH_SLOW`＋`StartPerishing`＋替换 `spoiled_food` | 主模拟；2 没有该组件 | [[reference/components/perishable]]；[接线](../../../raw/current/scripts/prefabs/yotp_food.lua#L66-L71) |
| 组件：`floater` | 三件都由 `MakeInventoryFloatable` 挂上；1／3 覆盖 size／scale，3 另加 override build 与动画监听 | 公共端挂组件；override build 在 `SetPristine` 前；事件监听只在主模拟 | [[reference/components/floater]]；[接线](../../../raw/current/scripts/prefabs/yotp_food.lua#L40-L50)、[监听](../../../raw/current/scripts/prefabs/yotp_food.lua#L80-L84) |
| 组件：`stackable`／`tradable`／`inspectable`／`inventoryitem` | 40 叠；交易值默认 0；可检视、可入栏 | 仅主模拟 | [[reference/components/stackable]]、[[reference/components/tradable]]；[接线](../../../raw/current/scripts/prefabs/yotp_food.lua#L73-L78) |
| 食性消费者：默认玩家 `eater` | 玩家默认 `caneat={FOODGROUP.OMNI}`，OMNI 不含 `HORRIBLE`，所以 `yotp_food2` 对默认玩家不可食 | 玩家构造只 `AddComponent("eater")`，不改食性；EAT 候选只按 `edible_*` 与 `*_eater` 标签匹配 | [[reference/components/eater]]；[默认食性](../../../raw/current/scripts/components/eater.lua#L18-L24)、[OMNI 组成](../../../raw/current/scripts/constants.lua#L2049-L2063)、[玩家构造](../../../raw/current/scripts/prefabs/player_common.lua#L2794-L2796)、[EAT 候选](../../../raw/current/scripts/componentactions.lua#L2672-L2688) |
| 食性消费者：猪人 | `DIET={FOODGROUP.OMNI, FOODTYPE.HORRIBLE, FOODTYPE.RAW}`，能吃 HORRIBLE | 猪人自己的 prefab | [猪人食性](../../../raw/current/scripts/prefabs/pigman.lua#L746-L746) |
| API：`Recipe2("yotp_food3")` | 4 张 `lucky_goldnugget`，`TECH.PIGOFFERING_THREE`，`nounlock`，`actionstr="PERDOFFERING"` | 猪神龛的供品配方 | [配方](../../../raw/current/scripts/recipes.lua#L1181-L1181)、[TECH](../../../raw/current/scripts/constants.lua#L1275-L1275) |
| API：`Recipe2("yotp_food1")` | 6 张；同 TECH 与选项 | 同上 | [配方](../../../raw/current/scripts/recipes.lua#L1182-L1182) |
| API：`Recipe2("yotp_food2")` | 1 张；同 TECH 与选项 | 同上 | [配方](../../../raw/current/scripts/recipes.lua#L1183-L1183) |
| 站：`pigshrine` | `prototyper.trees = TUNING.PROTOTYPER_TREES.PIGSHRINE`；该树给 `PIGOFFERING=3` 与 `PERDOFFERING=1` | 服务端、神龛实体；空龛无 prototyper（先放肉，见陷阱） | [[reference/components/prototyper]]；[接线](../../../raw/current/scripts/prefabs/pigshrine.lua#L35-L44)、[科技树](../../../raw/current/scripts/tuning.lua#L1329-L1332)、[站动作串](../../../raw/current/scripts/recipes.lua#L44-L44)、[别名](../../../raw/current/scripts/recipes.lua#L51-L51) |
| 资源：`anim/yotp_food.zip`、`anim/yotp_food_water.zip` | bank／build `yotp_food`；water build 供第三件的 override | 原版资源，不随页分发 | [assets](../../../raw/current/scripts/prefabs/yotp_food.lua#L1-L5) |

## 典型接入

### 把八宝泥馅饼改成玩家可食

- 前提与选择：服务端 `modmain.lua`；改 `edible.foodtype` 而不是改 `FOODGROUP.OMNI`（后者会影响所有默认食性角色）。
- 生效与撤销：`foodtype` 的 setter 会先摘掉旧的 `edible_HORRIBLE` 标签、再加 `edible_MEAT`（[setter](../../../raw/current/scripts/components/edible.lua#L9-L17)），但 `AddPrefabPostInit` 只在构造时执行，所以只影响此后新构造的实例；本进程已存在的实体不会回炉，需另行遍历改。卸模组后新构造回到 HORRIBLE。
- 原版依据：三件的食性都来自 `foodinfo` 表，没有第二处覆盖。

```lua
-- modmain.lua：把三件供品的食性都改成 MEAT（会同步替换 edible_ 标签）
local FOODS = { "yotp_food1", "yotp_food2", "yotp_food3" }
for _, name in ipairs(FOODS) do
    AddPrefabPostInit(name, function(inst)
        if GLOBAL.TheWorld.ismastersim and inst.components.edible ~= nil then
            inst.components.edible.foodtype = GLOBAL.FOODTYPE.MEAT
        end
    end)
end
```

- 调整与验证：只改 `foodtype` 不会重算饥饿／生命／理智；要一起改就继续写 `hungervalue` 等字段。验证看 EAT 动作是否出现、以及 `edible_MEAT` 标签是否替换成功。本页未运行游戏。

## 常见陷阱与限制

- **误区：三件都是 15 天腐坏。** 只有 `yotp_food1`／`yotp_food3` 有 `perishtime`；`yotp_food2` 没有 `perishable`，不能整组写"三天／十五天腐烂"，也不能给它写"腐烂后变 spoiled_food"。[数据](../../../raw/current/scripts/prefabs/yotp_food.lua#L9-L11)、[条件](../../../raw/current/scripts/prefabs/yotp_food.lua#L66-L71)
- **误区：标签循环会给每件加 `data.tags` 里的标签。** `foodinfo` 从未写 `tags`，L33-L37 是死分支；工厂显式加的标签只有 `pre-preparedfood`＋`edible_*`；此外 `perishable` 会另挂 `fresh`／`stale`／`spoiled`（三件里只有 1／3 有 `perishable`，2 没有；[条件](../../../raw/current/scripts/prefabs/yotp_food.lua#L66-L71)、[状态标签](../../../raw/current/scripts/components/perishable.lua#L3-L22)）。[循环](../../../raw/current/scripts/prefabs/yotp_food.lua#L33-L38)、[食性标签](../../../raw/current/scripts/components/edible.lua#L9-L17)
- **误区：八宝泥馅饼是玩家食物。** 它的食性是 `HORRIBLE`，默认玩家 `caneat={FOODGROUP.OMNI}` 不含该类型，玩家侧不出现 EAT；能吃它的是把 `HORRIBLE` 加进食性的生物（如猪人）。[数据](../../../raw/current/scripts/prefabs/yotp_food.lua#L10-L10)、[默认食性](../../../raw/current/scripts/components/eater.lua#L18-L24)、[OMNI](../../../raw/current/scripts/constants.lua#L2049-L2063)、[EAT 候选](../../../raw/current/scripts/componentactions.lua#L2672-L2688)、[猪人](../../../raw/current/scripts/prefabs/pigman.lua#L746-L746)
- **边界：`floater` 缺字段不等于没有浮水。** `yotp_food2` 仍由 `MakeInventoryFloatable` 挂了 `floater`，只是没覆盖 size／scale，取默认 `small` 与 1.0。[helper](../../../raw/current/scripts/standardcomponents.lua#L1772-L1791)、[默认缩放](../../../raw/current/scripts/components/floater.lua#L30-L34)、[跳过覆盖](../../../raw/current/scripts/prefabs/yotp_food.lua#L42-L50)
- **边界：override build 两端都执行，只有浮水动画监听在主模拟。** `AddOverrideBuild("yotp_food_water")` 在 `SetPristine` 之前的公共端（L47-L49），两端都跑；`floater_startfloating`／`stopfloating` 监听在 `ismastersim` 分支之后（L80-L84），只有主模拟注册。客户端是否看到 `food3_float` 未核验。[build](../../../raw/current/scripts/prefabs/yotp_food.lua#L47-L49)、[监听](../../../raw/current/scripts/prefabs/yotp_food.lua#L80-L84)、[事件](../../../raw/current/scripts/components/floater.lua#L146-L176)
- **边界：供品配方不是徒手合成。** 三条 `Recipe2` 都标 `nounlock`（不需要已解锁知识），但仍要 `CanPrototypeRecipe` 通过 `TECH.PIGOFFERING_THREE`，而该等级由 `pigshrine` 的科技树提供（`PIGOFFERING=3`），其他年度神龛不提供 `PIGOFFERING`；`recipes_filter.lua` 另把三条登记进制造菜单的 `CRAFTING_FILTERS.CRAFTING_STATION.recipes`（L398 起，三条在 L768-L770），那是 UI 筛选、不是可获得性门禁。[配方](../../../raw/current/scripts/recipes.lua#L1181-L1183)、[`nounlock` 定义](../../../raw/current/scripts/recipe.lua#L124-L131)、[UI 门禁](../../../raw/current/scripts/widgets/craftslot.lua#L168-L171)、[神龛树](../../../raw/current/scripts/tuning.lua#L1329-L1332)、[筛选表](../../../raw/current/scripts/recipes_filter.lua#L398-L398)、[条目](../../../raw/current/scripts/recipes_filter.lua#L768-L770)
- **边界：空猪神龛不是原型机。** 建成时 `onbuilt` 调 `MakeEmpty`（[接线](../../../raw/current/scripts/prefabs/pigshrine.lua#L278-L278)）：挂 `trader`、`RemoveComponent("prototyper")`（[空态](../../../raw/current/scripts/prefabs/pigshrine.lua#L111-L123)）；投入 `meat`／`meat_dried`／`cookedmeat` 后 `SetOffering` 才 `MakePrototyper`（[收供品](../../../raw/current/scripts/prefabs/pigshrine.lua#L90-L101)），所以要先放肉才有 `PIGOFFERING` 页签。
- **未核验**：未运行游戏；`yotp_food3` 的 override 动画在客户端是否可见、猪神龛在活动外的可获得性、以及采集册是否收录三件均未核验。

## 验证记录

- 源码审阅：2026-11-09，Steam buildid 26474873。通读 `prefabs/yotp_food.lua`（数据表、工厂、浮水分支、循环与导出注释）；`recipes.lua` 的 L1181-L1183 与 L44/L51 站点定义；`recipes_filter.lua` 的 L768-L770；`constants.lua` 的 `PIGOFFERING_THREE` 与 `FOODGROUP`；`tuning.lua` 的 PIGSHRINE 树与热量／治疗／理智／鲜度／堆叠常量；`pigshrine.lua` 的 `MakePrototyper`；`recipe.lua`／`craftslot.lua` 的 `nounlock` 语义；`debugcommands.lua` 的浮水测试名单；`edible`／`perishable`／`floater`／`tradable`／`standardcomponents` 的结算与 helper；`eater`／`player_common`／`pigman` 的食性；`chinese_s.po` 名称。
- 名称核验：三个 `NAMES.YOTP_FOOD*` 键与注册名同名。
- 游戏运行：未运行；数值、秒数与食性判定均为源码推导。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 见交付回执。
- 修订记录：2026-11-09 按内检与三轮外检修订：空龛／收供品、override 端、标签范围／措辞、鲜度倍率／温湿／酸雨、postinit 作用域。

## 相关链接

- 同族节庆供品：[[reference/prefabs/carnival-food]]、[[reference/prefabs/yotr-food]]
- 食物组件：[[reference/components/edible]]、[[reference/components/perishable]]、[[reference/components/floater]]
- 金块与神龛：[[reference/prefabs/basic-mineral-materials]]（`lucky_goldnugget`）、[[reference/components/prototyper]]

返回 [[reference/prefabs/special-food-index|节庆、角色与特殊食品索引]]。
