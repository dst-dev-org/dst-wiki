---
title: "兔子卷、月饼、月冻与泡芙串（yotr_food1、yotr_food2、yotr_food3、yotr_food4）"
created: "2026-11-09"
updated: "2026-11-09"
type: prefab
size_class: standard
summary: "兔人神龛（yotr_rabbitshrine）的四个幸运金块供品：同工厂四注册、全部 GOODIES 且 PERISH_SLOW（15 天），数值分别是 CALORIES_HUGE／HEALING_HUGE／SANITY_LARGE 的整倍或 1/2、1/4；3、4 另有 AddOverrideBuild(yotr_food_water)，但本文件只登记了 yotr_food.zip。"
aliases: ["yotr_food1", "yotr_food2", "yotr_food3", "yotr_food4", "兔子卷", "月饼", "月冻", "泡芙串", "Bunny Roll", "Moon Cake", "Moon Jelly", "Skewered Puffs", "兔年供品", "兔人神龛供品"]
tags: [prefab, item, food]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/yotr_food.lua", "prefabs/yotr_rabbitshrine.lua", "prefabs/wx78.lua", "componentactions.lua", "recipes.lua", "recipes_filter.lua", "constants.lua", "tuning.lua", "recipe.lua", "widgets/craftslot.lua", "components/edible.lua", "components/perishable.lua", "components/tradable.lua", "components/floater.lua", "standardcomponents.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 兔子卷、月饼、月冻与泡芙串（yotr_food1、yotr_food2、yotr_food3、yotr_food4）

## 快速判断

- 职责与适用：`prefabs/yotr_food.lua` 用一张四项 `foodinfo` 表加 `MakeFood(num)` 循环注册四个兔年供品，全部是 `GOODIES`、全部有 15 天鲜度；本页适合查"同工厂多注册、数值按整倍／分倍分配"的写法与浮水 override 的资源缺口。[注册](../../../raw/current/scripts/prefabs/yotr_food.lua#L122-L127)
- 优先使用：查四件差异先看[注册与构造](#注册与构造)与[数值与鲜度](#数值与鲜度)；与猪年三件（yotp）的对照看[常见陷阱](#常见陷阱与限制)。
- 重要副作用：**四件都能腐坏**（`PERISH_SLOW`＝15 天＝7200 秒，构造即开始），与 yotp 只有两件有鲜度不同；四件又**都不挂 `burnable`／`propagator`**，不能点燃。[数据表](../../../raw/current/scripts/prefabs/yotr_food.lua#L6-L40)、[公共段](../../../raw/current/scripts/prefabs/yotr_food.lua#L86-L114)
- 修改边界：兔人神龛 `yotr_rabbitshrine` 本体与幸运金块来源不展开；本页只负责四个注册的工厂、数值、鲜度、浮水与商店入口（[工厂](../../../raw/current/scripts/prefabs/yotr_food.lua#L42-L120)）。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `yotr_food1` | `MakeFood(1)`：`data=foodinfo[1]`，`Prefab("yotr_food1", fn, assets)`（L42-L45、L119） | 兔子卷：GOODIES；饥饿 `CALORIES_HUGE`＝75、生命 `HEALING_HUGE/4`＝15、理智 `SANITY_LARGE/4`＝8.25；`floater={"med", 0.9, false}`（L8-L15） |
| `yotr_food2` | 同工厂 `MakeFood(2)`；注册名为 `yotr_food2` | 月饼：GOODIES；饥饿 `CALORIES_HUGE/4`＝18.75、生命 `HEALING_HUGE`＝60、理智 `SANITY_LARGE/4`＝8.25（L16-L23） |
| `yotr_food3` | 同工厂 `MakeFood(3)` | 月冻：GOODIES；饥饿 75/4＝18.75、生命 60/4＝15、理智 `SANITY_LARGE`＝33；`floater={"med", {0.8, 0.5, 0.8}, true}`→按轴缩放并 `AddOverrideBuild("yotr_food_water")`（L24-L31、L70-L78） |
| `yotr_food4` | 同工厂 `MakeFood(4)` | 泡芙串：GOODIES；饥饿 75/2＝37.5、生命 60/2＝30、理智 33/2＝16.5；`floater` 第三位同为 `true`（L32-L39、L70-L78） |

四件共用同一 `fn`：bank／build 固定 `yotr_food`、动画 `"food"..num`（L55-L59）；两端先加 `pre-preparedfood`（L66）；`MakeInventoryFloatable` 后按键覆盖 size／scale（L68-L78）；`SetPristine` 后主模拟才加 `edible`／`perishable`／`stackable` 40／`tradable`／`inspectable`／`inventoryitem`（L80-L106），最后只有 `MakeHauntableLaunch`（L114）。`assets` 只声明 `anim/yotr_food.zip`（L1-L4）。

### 标签循环同样是死分支

`foodinfo` 四项都没有 `tags` 字段，L61-L65 的 `if data.tags ~= nil then … AddTag` 不会执行。工厂显式加的玩法标签只有 `pre-preparedfood`（L66）与 `edible_GOODIES`（`edible` 在 `foodtype` 赋值时挂上，[食性标签](../../../raw/current/scripts/components/edible.lua#L9-L17)）；`perishable` 另挂 `fresh`／`stale`／`spoiled`（[状态标签](../../../raw/current/scripts/components/perishable.lua#L3-L22)）。

### 浮水 override 与资源声明

第三位 `true` 的两件（3、4）在公共端执行 `inst.AnimState:AddOverrideBuild("yotr_food_water")`（L76），主模拟再监听 `floater_startfloating`／`floater_stopfloating`，分别播 `food3_float`／`food4_float` 与原动画（L108-L112）；事件由 `floater` 在落水与离水时推（[开始](../../../raw/current/scripts/components/floater.lua#L146-L176)、[结束](../../../raw/current/scripts/components/floater.lua#L214-L222)）。**但 `assets` 只声明 `anim/yotr_food.zip`**：`yotr_food_water` 在 `scripts/` 内只出现在这一行 `AddOverrideBuild`，没有 `Asset` 声明——本文件未登记该 build，加载来源未核验。对照：同族的 `yotp_food.lua` 在自己的 `assets` 里声明了 `yotp_food_water.zip`（L3-L4）。

### 名称核验

简中：`NAMES.YOTR_FOOD1`＝**兔子卷**、`NAMES.YOTR_FOOD2`＝**月饼**、`NAMES.YOTR_FOOD3`＝**月冻**、`NAMES.YOTR_FOOD4`＝**泡芙串**。

| 注册 | 英文 msgid | 简中 msgstr | 证据 |
|---|---|---|---|
| `yotr_food1` | Bunny Roll | 兔子卷 | [po](../../../raw/current/scripts/languages/chinese_s.po#L353084-L353087) |
| `yotr_food2` | Moon Cake | 月饼 | [po](../../../raw/current/scripts/languages/chinese_s.po#L353089-L353092) |
| `yotr_food3` | Moon Jelly | 月冻 | [po](../../../raw/current/scripts/languages/chinese_s.po#L353094-L353097) |
| `yotr_food4` | Skewered Puffs | 泡芙串 | [po](../../../raw/current/scripts/languages/chinese_s.po#L353099-L353102) |

## 数值与鲜度

时间基准：`seg_time`＝30 秒、`total_day_time`＝30×16＝480 秒、`perish_warp`＝1。[基准](../../../raw/current/scripts/tuning.lua#L19-L20)、[`perish_warp`](../../../raw/current/scripts/tuning.lua#L47-L47)

鲜度口径：下表 7200 秒是 modifier=1 的定义值（手持或普通容器内），天数按 480 秒／天换算；`perishable:Update` 在无 owner 时取 `PERISH_GROUND_MULT`＝1.5（约 4800 秒），owner 带 `fridge` 且未冻结时取 `PERISH_FRIDGE_MULT`＝0.5（约 14400 秒），有 `preserver` 组件时优先取它。以上落地秒数按常温、干燥、无酸雨、默认倍率计；落地且 `isacidraining`、无 `rainimmunity`、`CanPointHaveAcidRain` 同时成立时另有额外折旧，潮湿 ×1.3；`ambient_temperature < 0`（未冻结）乘 `PERISH_WINTER_MULT`（0.75），`ambient_temperature > OVERHEAT_TEMP`（70）乘 `PERISH_SUMMER_MULT`（1.25）。剩余比例挂 `fresh`／`stale`／`spoiled` 标签供 `edible` 判衰减。[落地](../../../raw/current/scripts/components/perishable.lua#L97-L97)、[冰箱／preserver](../../../raw/current/scripts/components/perishable.lua#L80-L86)、[酸雨／倍率](../../../raw/current/scripts/components/perishable.lua#L97-L125)、[标签](../../../raw/current/scripts/components/perishable.lua#L3-L22)、[结算](../../../raw/current/scripts/components/perishable.lua#L129-L133)、[倍率](../../../raw/current/scripts/tuning.lua#L2079-L2088)

| 注册 | 饥饿（公式） | 生命（公式） | 理智（公式） | 鲜度 |
|---|---:|---:|---:|---|
| `yotr_food1` | 75（`CALORIES_HUGE`×1） | 15（`HEALING_HUGE` 60÷4） | 8.25（`SANITY_LARGE` 33÷4） | `PERISH_SLOW`＝15 天＝7200 秒 → `spoiled_food` |
| `yotr_food2` | 18.75（75÷4） | 60（60×1） | 8.25（33÷4） | 同上 |
| `yotr_food3` | 18.75（75÷4） | 15（60÷4） | 33（33×1） | 同上 |
| `yotr_food4` | 37.5（75÷2） | 30（60÷2） | 16.5（33÷2） | 同上 |

"谁除几"：四件都从同一组三个常量取值——`CALORIES_HUGE`＝75、`HEALING_HUGE`＝60、`SANITY_LARGE`＝33；1／2／3 号各偏一项、其余两项 ÷4；4 号三项统一 ÷2。定义处见[热量档](../../../raw/current/scripts/tuning.lua#L2058-L2065)、[治疗档](../../../raw/current/scripts/tuning.lua#L2021-L2030)、[理智档](../../../raw/current/scripts/tuning.lua#L2032-L2038)；结算处是 `edible` 的三个 getter（[饥饿](../../../raw/current/scripts/components/edible.lua#L118)、[生命](../../../raw/current/scripts/components/edible.lua#L142)、[理智](../../../raw/current/scripts/components/edible.lua#L93)），赋值在工厂 L86-L92。

- **四件全部有 `perishable`**：`perishtime=PERISH_SLOW` 均非空（L13、L21、L29、L37），所以都走 `AddComponent("perishable")`、`SetPerishTime`、`StartPerishing`、`onperishreplacement="spoiled_food"`（L94-L99）。15 天＝7200 秒（定义值；落地约 4800、冰箱约 14400 秒，见上），[计时](../../../raw/current/scripts/components/perishable.lua#L262-L271)、[替换](../../../raw/current/scripts/components/perishable.lua#L293-L304)。
- **腐坏从构造开始**：与同族一致，没有"进背包才计时"或"落地停表"的分支；鲜度进入 stale／spoiled 后 `GetHunger`／`GetHealth` 会乘衰减系数（WX-78 默认 `ignoresspoilage` 例外，见[典型接入](#典型接入)）（[饥饿](../../../raw/current/scripts/components/edible.lua#L118)）。
- **四件都是 GOODIES**：默认玩家 `caneat={FOODGROUP.OMNI}`，OMNI 含 `GOODIES`，所以四件对默认玩家都可食（与 yotp 的 HORRIBLE 那件不同）。[默认食性](../../../raw/current/scripts/components/eater.lua#L18-L24)、[OMNI](../../../raw/current/scripts/constants.lua#L2049-L2063)、[EAT 候选](../../../raw/current/scripts/componentactions.lua#L2672-L2688)
- **堆叠与交易**：统一 40 叠；`tradable` 未写 `goldvalue`，默认 0（[接线](../../../raw/current/scripts/prefabs/yotr_food.lua#L101-L106)、[默认值](../../../raw/current/scripts/components/tradable.lua#L1-L7)）。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`edible` | 四件各写 `foodtype=GOODIES` 与三项数值；温度字段落到默认 0 | 仅主模拟；`foodtype` 赋值同步 `edible_GOODIES` 标签 | [[reference/components/edible]]；[赋值](../../../raw/current/scripts/prefabs/yotr_food.lua#L86-L92)、[食性标签](../../../raw/current/scripts/components/edible.lua#L9-L17) |
| 组件：`perishable` | 四件全部 `PERISH_SLOW`＋`StartPerishing`＋替换 `spoiled_food` | 主模拟；无逐件开关 | [[reference/components/perishable]]；[接线](../../../raw/current/scripts/prefabs/yotr_food.lua#L94-L99) |
| 组件：`floater` | 四件都挂；3／4 为按键缩放并加 `yotr_food_water` override | 公共端挂组件与 override build；事件监听只在主模拟 | [[reference/components/floater]]；[接线](../../../raw/current/scripts/prefabs/yotr_food.lua#L68-L78)、[监听](../../../raw/current/scripts/prefabs/yotr_food.lua#L108-L112) |
| 组件：`stackable`／`tradable`／`inspectable`／`inventoryitem` | 40 叠；交易值默认 0；可检视、可入栏 | 仅主模拟 | [[reference/components/stackable]]、[[reference/components/tradable]]；[接线](../../../raw/current/scripts/prefabs/yotr_food.lua#L101-L106) |
| 缺失：`burnable`／`propagator` | 全文件只有 `MakeHauntableLaunch`，没有 `MakeSmallBurnable`／`MakeSmallPropagator` | 四件一致；不能点燃、不传火 | [公共段](../../../raw/current/scripts/prefabs/yotr_food.lua#L108-L114) |
| API：`Recipe2("yotr_food1")` | 5 张 `lucky_goldnugget`，`TECH.RABBITOFFERING_THREE`，`nounlock`，`actionstr="PERDOFFERING"` | 兔人神龛的供品配方 | [配方](../../../raw/current/scripts/recipes.lua#L1123-L1123)、[TECH](../../../raw/current/scripts/constants.lua#L1279-L1279) |
| API：`Recipe2("yotr_food2")` | 5 张；同 TECH 与选项 | 同上 | [配方](../../../raw/current/scripts/recipes.lua#L1124-L1124) |
| API：`Recipe2("yotr_food3")` | 5 张；同 TECH 与选项 | 同上 | [配方](../../../raw/current/scripts/recipes.lua#L1125-L1125) |
| API：`Recipe2("yotr_food4")` | 5 张；同 TECH 与选项 | 同上 | [配方](../../../raw/current/scripts/recipes.lua#L1126-L1126) |
| 站：`yotr_rabbitshrine` | `prototyper.trees = TUNING.PROTOTYPER_TREES.RABBITSHRINE`；该树给 `RABBITOFFERING=3` 与 `PERDOFFERING=1` | 服务端、神龛实体；空龛无 prototyper（先放胡萝卜，见陷阱） | [[reference/components/prototyper]]；[接线](../../../raw/current/scripts/prefabs/yotr_rabbitshrine.lua#L190-L199)、[科技树](../../../raw/current/scripts/tuning.lua#L1349-L1352)、[站动作串](../../../raw/current/scripts/recipes.lua#L44-L44)、[别名](../../../raw/current/scripts/recipes.lua#L55-L55) |
| 资源：`anim/yotr_food.zip` | bank／build `yotr_food`；未声明 `yotr_food_water` | 原版资源，不随页分发 | [assets](../../../raw/current/scripts/prefabs/yotr_food.lua#L1-L4) |

## 典型接入

### 单独调整月饼的治疗

- 前提与选择：服务端 `modmain.lua` 的 `AddPrefabPostInit`；只改 `edible.healthvalue`，不动 `HEALING_HUGE`（后者会连带所有用该档的食物）。
- 生效与撤销：`GetHealth` 在进食时读 `healthvalue`，但 `AddPrefabPostInit` 只在此后构造的实例上执行，本进程已存在的实体不会回炉、需另行遍历改；普通食客在鲜度进入 stale／spoiled 后仍乘衰减系数（WX-78 默认 `SetIgnoresSpoilage(true)`，不吃该衰减，[wx78](../../../raw/current/scripts/prefabs/wx78.lua#L753-L756)）。卸模组后回到 60。
- 原版依据：四件的数值只在 `foodinfo` 表里定义一次，工厂逐项赋值（L86-L92）。

```lua
-- modmain.lua：把月饼的治疗从 60 改成 40（HEALING_LARGE）
AddPrefabPostInit("yotr_food2", function(inst)
    if GLOBAL.TheWorld.ismastersim and inst.components.edible ~= nil then
        inst.components.edible.healthvalue = GLOBAL.TUNING.HEALING_LARGE
    end
end)
```

- 调整与验证：`HEALING_LARGE`＝40（[档位](../../../raw/current/scripts/tuning.lua#L2027-L2027)）；验证看进食后生命增量与鲜度衰减，不要只读组件字段。本页未运行游戏。

## 常见陷阱与限制

- **与 yotp 的差异一：鲜度不是整组一致。** yotr 四件**全部**有 `perishable`（15 天）；yotp 只有 1／3 有、2 无。详见 [[reference/prefabs/yotp-food#注册与构造]]。[yotr 数据](../../../raw/current/scripts/prefabs/yotr_food.lua#L6-L40)、[yotp 条件](../../../raw/current/scripts/prefabs/yotp_food.lua#L66-L71)
- **与 yotp 的差异二：都不燃。** 本文件没有 `MakeSmallBurnable`／`MakeSmallPropagator`，四件都不能点燃；同族的玉米泥（carnival_food）反而会烧。yotp 三件同样不烧。[本文件公共段](../../../raw/current/scripts/prefabs/yotr_food.lua#L108-L114)、[玉米泥](../../../raw/current/scripts/prefabs/carnival_food.lua#L65-L66)
- **误区：`yotr_food_water` 已登记。** 本文件只在 `AddOverrideBuild` 里引用它，`assets` 只有 `anim/yotr_food.zip`；`scripts/` 内检索不到第二处声明，加载来源未核验。别把 yotp 的"已声明 `yotp_food_water.zip`"类推过来。[引用](../../../raw/current/scripts/prefabs/yotr_food.lua#L76-L76)、[assets](../../../raw/current/scripts/prefabs/yotr_food.lua#L1-L4)、[yotp 对照](../../../raw/current/scripts/prefabs/yotp_food.lua#L1-L5)
- **误区：标签循环给四件加过标签。** `data.tags` 从未赋值，L61-L65 是死分支；工厂显式加的标签只有 `pre-preparedfood`＋`edible_GOODIES`；此外 `perishable` 另挂 `fresh`／`stale`／`spoiled`（[状态标签](../../../raw/current/scripts/components/perishable.lua#L3-L22)）。[循环](../../../raw/current/scripts/prefabs/yotr_food.lua#L61-L66)
- **边界：`PERISH_SLOW` 是 15 天，不是 3 天。** 四件都按 7200 秒（定义值；落地约 4800、冰箱约 14400 秒）计；到点替换 `spoiled_food` 并继承堆叠。[常量](../../../raw/current/scripts/tuning.lua#L2049-L2049)、[替换](../../../raw/current/scripts/components/perishable.lua#L293-L304)
- **边界：供品配方不是徒手合成。** 四条都是 `Recipe2`＋`nounlock`，仍需 `CanPrototypeRecipe` 的 `TECH.RABBITOFFERING_THREE`，由 `yotr_rabbitshrine` 的科技树提供；`recipes_filter.lua` 把四件登记进 `CRAFTING_FILTERS.CRAFTING_STATION.recipes`（L398 起，四件在 L723-L726），那是 UI 筛选、不是可获得性门禁。[配方](../../../raw/current/scripts/recipes.lua#L1123-L1126)、[`nounlock` 定义](../../../raw/current/scripts/recipe.lua#L124-L131)、[UI 门禁](../../../raw/current/scripts/widgets/craftslot.lua#L168-L171)、[筛选表](../../../raw/current/scripts/recipes_filter.lua#L398-L398)、[条目](../../../raw/current/scripts/recipes_filter.lua#L723-L726)
- **边界：空兔人神龛不是原型机且只收胡萝卜。** `onbuilt` 调 `MakeEmpty`（[接线](../../../raw/current/scripts/prefabs/yotr_rabbitshrine.lua#L417-L417)）：移除 `prototyper`、挂 `trader`（[空态](../../../raw/current/scripts/prefabs/yotr_rabbitshrine.lua#L261-L271)）；投入 `carrot` 后才 `SetOffering`→`MakePrototyper`（[收供品](../../../raw/current/scripts/prefabs/yotr_rabbitshrine.lua#L240-L249)）。
- **未核验**：未运行游戏；`yotr_food_water` 的实际加载来源、客户端是否看到 `food3_float`／`food4_float`、以及采集册是否收录四件均未核验。

## 验证记录

- 源码审阅：2026-11-09，Steam buildid 26474873。通读 `prefabs/yotr_food.lua`（四项数据、工厂、浮水分支、循环）；`recipes.lua` 的 L1123-L1126、L44、L55；`recipes_filter.lua` 的 L723-L726；`constants.lua` 的 `RABBITOFFERING_THREE` 与 `FOODGROUP`；`tuning.lua` 的 RABBITSHRINE 树与热量／治疗／理智／鲜度／堆叠常量；`yotr_rabbitshrine.lua` 的 `MakePrototyper`；`recipe.lua`／`craftslot.lua` 的 `nounlock` 语义；`edible`／`perishable`／`floater`／`tradable`／`standardcomponents` 的结算与 helper；`chinese_s.po` 名称；全 `scripts/` 检索 `yotr_food_water` 的结果。
- 名称核验：四个 `NAMES.YOTR_FOOD*` 键与注册名同名。
- 游戏运行：未运行；数值、秒数与资源声明结论均为源码推导／检索。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 见交付回执。
- 修订记录：2026-11-09 按内检与三轮外检修订：空龛／只收胡萝卜、标签范围／措辞、鲜度倍率／温湿／酸雨、postinit 作用域、WX-78 例外。

## 相关链接

- 同族节庆供品：[[reference/prefabs/yotp-food]]、[[reference/prefabs/carnival-food]]
- 食物组件：[[reference/components/edible]]、[[reference/components/perishable]]、[[reference/components/floater]]
- 金块与神龛：[[reference/prefabs/basic-mineral-materials]]（`lucky_goldnugget`）、[[reference/components/prototyper]]

返回 [[reference/prefabs/special-food-index|节庆、角色与特殊食品索引]]。
