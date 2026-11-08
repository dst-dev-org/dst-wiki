---
title: "冬季盛宴菜肴（berrysauce、bibingka、cabbagerolls、festivefish、gravy、latkes、lutefisk、mulleddrink、panettone、pavlova、pickledherring、polishcookie、pumpkinpie、roastturkey、stuffing、sweetpotato、tamales、tourtiere）"
created: "2026-11-09"
updated: "2026-11-09"
type: prefab
summary: "十八道冬宴菜共用 MakeFood 工厂与一张 cooktime 数据表：物品无 edible 与 perishable，180 份耐久只能被冬宴餐桌按 1/3 秒一口消耗（末份归零后仍在槽里停留 0–0.5 秒才清掉），烤制则走烤炉两段各乘 cooktime 的科学流程。"
aliases: ["berrysauce", "bibingka", "cabbagerolls", "festivefish", "gravy", "latkes", "lutefisk", "mulleddrink", "panettone", "pavlova", "pickledherring", "polishcookie", "pumpkinpie", "roastturkey", "stuffing", "sweetpotato", "tamales", "tourtiere", "快乐浆果酱", "比宾卡米糕", "白菜卷", "节庆鱼料理", "好肉汁", "土豆饼", "苏打鱼", "香料潘趣酒", "托尼甜面包", "巴甫洛娃蛋糕", "腌鲱鱼", "波兰饼干", "南瓜派", "烤火鸡", "烤火鸡面包馅", "红薯焗饭", "塔马利", "饕餮馅饼", "冬宴菜", "冬季盛宴菜肴"]
tags: [prefab, item, recipe]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/wintersfeastcookedfoods.lua", "wintersfeastcookedfoods.lua", "prefabs/wintersfeastoven.lua", "components/madsciencelab.lua", "prefabs/table_winters_feast.lua", "components/wintersfeasttable.lua", "components/feasts.lua", "components/finiteuses.lua", "componentactions.lua", "recipes.lua", "recipes_filter.lua", "constants.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 冬季盛宴菜肴（berrysauce、bibingka、cabbagerolls、festivefish、gravy、latkes、lutefisk、mulleddrink、panettone、pavlova、pickledherring、polishcookie、pumpkinpie、roastturkey、stuffing、sweetpotato、tamales、tourtiere）

## 快速判断

- 职责与适用：`prefabs/wintersfeastcookedfoods.lua` 用一份 `MakeFood` 注册 18 件冬宴菜，注册名直接取 `wintersfeastcookedfoods.foods` 的键；每件是一个以 `finiteuses` 记份数的物品，**没有 `edible` 组件**，不能直接吃，但可以上冬宴餐桌开餐，也能当陷阱诱饵（工厂挂了 `bait`）。[工厂](../../../raw/current/scripts/prefabs/wintersfeastcookedfoods.lua#L4-L73)、[注册循环](../../../raw/current/scripts/prefabs/wintersfeastcookedfoods.lua#L75-L78)
- 优先使用：查每道菜的 cooktime、名字与配方看[注册与构造](#注册与构造)；查 180 份与"一口"的换算看[份数与消耗](#份数与消耗)；查烤炉两段计时看[烤制时长](#烤制时长)；查那 18 个 `wintercooking_*` 看[假 prefab 与配方 product](#假-prefab-与配方-product)。
- 重要副作用：`SetOnFinished` 在工厂里被注释，注释写明"removed by the winters feast table"，所以耐久归零后由餐桌回调用 `item:Remove()` 清掉，而不是 `finiteuses` 自己删；物品同时没有 `perishable`，永远不会腐烂。[注释](../../../raw/current/scripts/prefabs/wintersfeastcookedfoods.lua#L51-L55)、[清桌](../../../raw/current/scripts/prefabs/table_winters_feast.lua#L169-L190)
- 修改边界：餐桌与盛宴 tick 归 [[reference/components/wintersfeasttable]]、[[reference/components/feasts]]；烤炉与科学流程归 `prefabs/wintersfeastoven.lua`（Wiki 待建）；枯竭特效归 [[reference/prefabs/winters_feast_depletefood]]；开餐后拿到的 buff 是 `wintersfeastbuff`（Wiki 待建）。

## 注册与构造

18 件同一工厂、同一 `fn`：assets 只有 `anim/food_winters_feast_2019.zip` 与 `Asset("INV_IMAGE", name)`；构造函数写 bank／build `food_winters_feast_2019`、播 `idle`、用 `OverrideSymbol("swap_food", …, name)` 换菜色；`inst.scrapbook_overridedata` 与 `scrapbook_specialinfo = "WINTERSFEASTCOOKEDFOODS"` 供采集册；浮水用默认 `MakeInventoryFloatable(inst)`（`floater` 覆盖项只在 `berrysauce` 里出现且被注释）。[构造](../../../raw/current/scripts/prefabs/wintersfeastcookedfoods.lua#L16-L49)

两端共有：`wintersfeastcookedfood` 标签（本 prefab 唯一显式加的标签）、`wet_prefix` 字段。只在主模拟挂的组件依次是 `finiteuses`（份数见下）→ `inspectable` → `inventoryitem` → `MakeSmallBurnable` → `MakeSmallPropagator` → `MakeHauntableLaunchAndPerish` → `bait` → `tradable`；**没有** `edible`、`perishable`、`stackable`、`cookable`。[标签](../../../raw/current/scripts/prefabs/wintersfeastcookedfoods.lua#L41-L45)、[挂件](../../../raw/current/scripts/prefabs/wintersfeastcookedfoods.lua#L51-L69)

| 注册名 | 中文名 | cooktime → 每段／两段合计 | 配方（除燃料 ×1 外的配料） |
|---|---|---|---|
| `berrysauce` | 快乐浆果酱[名](../../../raw/current/scripts/languages/chinese_s.po#L339432-L339432) | 0.8 → ≈8.0／≈16.0 秒[数据](../../../raw/current/scripts/wintersfeastcookedfoods.lua#L3-L10) | 蚊子血囊 ×2[配方](../../../raw/current/scripts/recipes.lua#L1430-L1430) |
| `bibingka` | 比宾卡米糕[名](../../../raw/current/scripts/languages/chinese_s.po#L339437-L339437) | 1 → ≈10.0／≈20.0 秒[数据](../../../raw/current/scripts/wintersfeastcookedfoods.lua#L11-L14) | 蕨叶 ×2[配方](../../../raw/current/scripts/recipes.lua#L1431-L1431) |
| `cabbagerolls` | 白菜卷[名](../../../raw/current/scripts/languages/chinese_s.po#L340052-L340052) | 0.8 → ≈8.0／≈16.0 秒[数据](../../../raw/current/scripts/wintersfeastcookedfoods.lua#L15-L18) | 采下的芦苇 ×2[配方](../../../raw/current/scripts/recipes.lua#L1432-L1432) |
| `festivefish` | 节庆鱼料理[名](../../../raw/current/scripts/languages/chinese_s.po#L342742-L342742) | 1 → ≈10.0／≈20.0 秒[数据](../../../raw/current/scripts/wintersfeastcookedfoods.lua#L19-L22) | 变质的小鱼块（`spoiled_fish_small`）×1＋花瓣 ×1[配方](../../../raw/current/scripts/recipes.lua#L1433-L1433) |
| `gravy` | 好肉汁[名](../../../raw/current/scripts/languages/chinese_s.po#L343427-L343427) | 1 → ≈10.0／≈20.0 秒[数据](../../../raw/current/scripts/wintersfeastcookedfoods.lua#L23-L26) | 腐烂物 ×1＋骨头碎片 ×1[配方](../../../raw/current/scripts/recipes.lua#L1434-L1434) |
| `latkes` | 土豆饼[名](../../../raw/current/scripts/languages/chinese_s.po#L344462-L344462) | 0.8 → ≈8.0／≈16.0 秒[数据](../../../raw/current/scripts/wintersfeastcookedfoods.lua#L27-L30) | 树枝 ×1＋采下的草 ×1[配方](../../../raw/current/scripts/recipes.lua#L1435-L1435) |
| `lutefisk` | 苏打鱼[名](../../../raw/current/scripts/languages/chinese_s.po#L344947-L344947) | 1.4 → ≈14.0／≈28.0 秒[数据](../../../raw/current/scripts/wintersfeastcookedfoods.lua#L31-L34) | 变质的鱼（`spoiled_fish`）×1＋浮木桩 ×1[配方](../../../raw/current/scripts/recipes.lua#L1436-L1436) |
| `mulleddrink` | 香料潘趣酒[名](../../../raw/current/scripts/languages/chinese_s.po#L345962-L345962) | 1 → ≈10.0／≈20.0 秒[数据](../../../raw/current/scripts/wintersfeastcookedfoods.lua#L35-L38) | 深色花瓣 ×1＋冰 ×1[配方](../../../raw/current/scripts/recipes.lua#L1437-L1437) |
| `panettone` | 托尼甜面包[名](../../../raw/current/scripts/languages/chinese_s.po#L346737-L346737) | 1 → ≈10.0／≈20.0 秒[数据](../../../raw/current/scripts/wintersfeastcookedfoods.lua#L39-L42) | 石果 ×2（配方强制岩石贴图）[配方](../../../raw/current/scripts/recipes.lua#L1438-L1438) |
| `pavlova` | 巴甫洛娃蛋糕[名](../../../raw/current/scripts/languages/chinese_s.po#L346752-L346752) | 1 → ≈10.0／≈20.0 秒[数据](../../../raw/current/scripts/wintersfeastcookedfoods.lua#L43-L46) | 月树花 ×2[配方](../../../raw/current/scripts/recipes.lua#L1439-L1439) |
| `pickledherring` | 腌鲱鱼[名](../../../raw/current/scripts/languages/chinese_s.po#L346862-L346862) | 1.2 → ≈12.0／≈24.0 秒[数据](../../../raw/current/scripts/wintersfeastcookedfoods.lua#L47-L50) | 燧石 ×1＋盐晶 ×1[配方](../../../raw/current/scripts/recipes.lua#L1440-L1440) |
| `polishcookie` | 波兰饼干[名](../../../raw/current/scripts/languages/chinese_s.po#L347107-L347107) | 1 → ≈10.0／≈20.0 秒[数据](../../../raw/current/scripts/wintersfeastcookedfoods.lua#L51-L54) | 蝴蝶翅膀 ×2[配方](../../../raw/current/scripts/recipes.lua#L1441-L1441) |
| `pumpkinpie` | 南瓜派[名](../../../raw/current/scripts/languages/chinese_s.po#L347337-L347337) | 1 → ≈10.0／≈20.0 秒[数据](../../../raw/current/scripts/wintersfeastcookedfoods.lua#L55-L58) | 灰烬 ×1＋脓鼻涕 ×1[配方](../../../raw/current/scripts/recipes.lua#L1442-L1442) |
| `roastturkey` | 烤火鸡[名](../../../raw/current/scripts/languages/chinese_s.po#L348647-L348647) | 1.2 → ≈12.0／≈24.0 秒[数据](../../../raw/current/scripts/wintersfeastcookedfoods.lua#L59-L62) | 木头 ×1＋木炭 ×1[配方](../../../raw/current/scripts/recipes.lua#L1443-L1443) |
| `stuffing` | 烤火鸡面包馅[名](../../../raw/current/scripts/languages/chinese_s.po#L350102-L350102) | 1 → ≈10.0／≈20.0 秒[数据](../../../raw/current/scripts/wintersfeastcookedfoods.lua#L63-L66) | 胡须 ×2[配方](../../../raw/current/scripts/recipes.lua#L1444-L1444) |
| `sweetpotato` | 红薯焗饭[名](../../../raw/current/scripts/languages/chinese_s.po#L350187-L350187) | 1 → ≈10.0／≈20.0 秒[数据](../../../raw/current/scripts/wintersfeastcookedfoods.lua#L67-L70) | 石头 ×2[配方](../../../raw/current/scripts/recipes.lua#L1445-L1445) |
| `tamales` | 塔马利[名](../../../raw/current/scripts/languages/chinese_s.po#L350262-L350262) | 1 → ≈10.0／≈20.0 秒[数据](../../../raw/current/scripts/wintersfeastcookedfoods.lua#L71-L74) | 针刺 ×2[配方](../../../raw/current/scripts/recipes.lua#L1446-L1446) |
| `tourtiere` | 饕餮馅饼[名](../../../raw/current/scripts/languages/chinese_s.po#L350492-L350492) | 1 → ≈10.0／≈20.0 秒[数据](../../../raw/current/scripts/wintersfeastcookedfoods.lua#L75-L78) | 桦栗果 ×1＋松果 ×1[配方](../../../raw/current/scripts/recipes.lua#L1447-L1447) |

十八行同名同构，差异只在 `cooktime`（0.8／1／1.2／1.4 四档）与名字；数据文件里除 `cooktime` 外，只有 `berrysauce` 留有一条被注释掉的 `uses`／`wet_prefix`／`floater` 覆盖项（其余 17 条只有 `cooktime`），所以 18 件的份数、漂浮与湿润前缀都走默认值。[数据表](../../../raw/current/scripts/wintersfeastcookedfoods.lua#L1-L85)

### 份数与消耗

- 定义：`uses = data.uses or TUNING.WINTERSFEASTBUFF.EATTIME`，而 `TUNING.WINTERSFEASTBUFF.EATTIME = 180`；结算：`SetMaxUses(uses)`＋`SetUses(uses)`，于是每件 180 份。[常量](../../../raw/current/scripts/tuning.lua#L5176-L5187)、[接线](../../../raw/current/scripts/prefabs/wintersfeastcookedfoods.lua#L51-L54)、[采集册同项](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L167-L167)
- "一口"不是玩家动作：世界组件 `feasts` 在有人开餐时每 `TICK_RATE = 1/3` 秒跑一次 `doFeastCheck`，对组内每张桌调 `wintersfeasttable:DepleteFood(#feasters)`，组件里再 `finiteuses:Use()`（默认 1 份）。[tick](../../../raw/current/scripts/components/feasts.lua#L24-L24)、[扣份](../../../raw/current/scripts/components/feasts.lua#L80-L98)、[Use](../../../raw/current/scripts/components/wintersfeasttable.lua#L65-L81)
- 由此推导：180 份 × 1/3 秒 = **每位桌上这道菜可供 60 秒连续开餐**；`DepleteFood` 收到的进食人数参数在组件里没有被使用，所以多人与单人耗速相同。**推导，未运行验证。**
- 归零分支：`GetUses() > 0` 时调 `ondepletefoodfn`（随机闪粒），否则调 `onfinishfoodfn`；餐桌的 `OnFinishFood` 先把菜品标 `persists = false`，再在 `0–0.5` 秒后的 `DoTaskInTime` 里 drop 并 `Remove`、置 `canfeast = false`／`cantakeitem = false`、生成 `winters_feast_food_depleted`——不是同步删除。[分支](../../../raw/current/scripts/components/wintersfeasttable.lua#L65-L81)、[清桌](../../../raw/current/scripts/prefabs/table_winters_feast.lua#L169-L190)

### 不可直接食用

- 18 件都没有 `edible` 组件，EAT 候选挂在 `componentactions` 的 `edible` 分支上，只有带 `edible_<类型>` 标签的物品才会被塞进 `ACTIONS.EAT`；这些菜只有 `wintersfeastcookedfood` 标签，所以玩家手里拿着也吃不了。[EAT 分支](../../../raw/current/scripts/componentactions.lua#L2647-L2689)、[标签](../../../raw/current/scripts/prefabs/wintersfeastcookedfoods.lua#L41-L41)
- 进桌：餐桌的 `trader` 用 `ItemTradeTest` 只接受带 `wintersfeastcookedfood` 或 `preparedfood` 标签的物品，投入后 `shelf:PutItemOnShelf`，只有 `wintersfeastcookedfood` 分支会把 `canfeast` 置 true。入口还要求桌上当前没有占位（`takeshelfitem` 未置）、`trader` 处于 enabled、桌子既没着火也没烧毁；另外这 18 件在工厂里挂了 `bait`，所以也能塞进陷阱当诱饵。[接收](../../../raw/current/scripts/prefabs/table_winters_feast.lua#L42-L53)、[判定](../../../raw/current/scripts/prefabs/table_winters_feast.lua#L201-L243)、[bait](../../../raw/current/scripts/prefabs/wintersfeastcookedfoods.lua#L65-L65)
- 效果不在这件物品上：开餐时每个进食者每 tick 都 `AddDebuff("wintersfeastbuff")` 再 `addeffectbonusfn(buff, 人数, 菜品种类数, 菜品数)`，菜只是被桌子减份与计数。[buff](../../../raw/current/scripts/components/feasts.lua#L57-L78)

### 烤制时长

- 基准：`WINTERS_FEAST_OVEN_BASE_COOK_TIME = night_time * .3333`，`night_time = seg_time * 2 = 60` 秒，所以基准 ≈19.998 秒；烤炉把它对半分给两段：`BASE_STAGE_COOK_TIME = TUNING.WINTERS_FEAST_OVEN_BASE_COOK_TIME / 2`（注释写明 2 是 `SCIENCE_STAGES` 的段数）。[基准](../../../raw/current/scripts/tuning.lua#L3638-L3638)、[分段](../../../raw/current/scripts/prefabs/wintersfeastoven.lua#L28-L33)
- 倍率：`SetOvenCookTimeMultiplier` 把 `inst.science_stages` 每段的 `v.time` 改写成 `BASE_STAGE_COOK_TIME * multiplier`；`multiplier` 来自 `GetWinterFoodCookTimeMultiplier(recipe.product)`，即"假 prefab → 真菜名 → 数据表 cooktime"，取不到时回落 1。[改写](../../../raw/current/scripts/prefabs/wintersfeastoven.lua#L94-L105)、[接线](../../../raw/current/scripts/prefabs/wintersfeastoven.lua#L122-L131)
- 结算：`madsciencelab:SetStage` 用 `self.stages[stage].time` 排下一个 `DoTaskInTime`，段数超过 `#self.stages` 时回调 `OnScienceWasMade`；表格里的"每段／两段合计"就是 `9.999 × cooktime` 与再乘 2。读档时 `onload` 会按已记下的 product 重新套一次倍率。[推进](../../../raw/current/scripts/components/madsciencelab.lua#L23-L51)、[读档](../../../raw/current/scripts/prefabs/wintersfeastoven.lua#L302-L310)

### 假 prefab 与配方 product

- 18 条配方 `Recipe2("wintercooking_<name>", …)` 的 `product` 就是那个假名，Tech 是 `TECH.WINTERSFEASTCOOKING_ONE`（等价于 `WINTERSFEASTCOOKING = 1` 的一级科技），`actionstr = "COOK"`、`nounlock`、`manufactured`；配料一律先放 `wintersfeastfuel` ×1。[配方](../../../raw/current/scripts/recipes.lua#L1430-L1447)、[科技](../../../raw/current/scripts/constants.lua#L1309-L1309)、[树](../../../raw/current/scripts/tuning.lua#L1393-L1395)
- 烤炉自己注册这 18 个假 prefab：`WINTERS_FEAST_COOKED_FOODS["wintercooking_"..v] = v` 建映射，注释写明是"add fake prefabs for all the experiments so the game doesn't log about non-existing prefabs due to recipes"；`dummy_fn` 只建一个无网实体，加 `CLASSIFIED` 标签、`persists = false`，主模拟下一帧自行 `Remove`。[映射](../../../raw/current/scripts/prefabs/wintersfeastoven.lua#L57-L63)、[dummy](../../../raw/current/scripts/prefabs/wintersfeastoven.lua#L413-L428)、[用途](../../../raw/current/scripts/prefabs/wintersfeastoven.lua#L435-L438)
- 因此假 prefab 从不落地：配方只借它当 product 名，烤完由 `OnScienceWasMade` 把 `wintercooking_*` 翻回真菜名，再用 `pickable:SetUp(真菜名, 1000000)` 把成品挂到炉口等玩家取。[翻名](../../../raw/current/scripts/prefabs/wintersfeastoven.lua#L234-L250)、[装盘](../../../raw/current/scripts/prefabs/wintersfeastoven.lua#L83-L92)
- `CanCookPrefab` 用 `string.find(prefab, "wintercooking_")` 截前缀再逐项比对 `FOOD_PREFABS`，是烤炉判定"这道是不是我该做的菜"的唯一入口。[判定](../../../raw/current/scripts/prefabs/wintersfeastoven.lua#L107-L120)、[名单](../../../raw/current/scripts/prefabs/wintersfeastoven.lua#L35-L55)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`finiteuses` | 180 份；`SetOnFinished` 未接，删件交给餐桌 | 仅主模拟；只在开餐 tick 里被扣 | [[reference/components/finiteuses]]；[接线](../../../raw/current/scripts/prefabs/wintersfeastcookedfoods.lua#L51-L55) |
| 组件：`inventoryitem／inspectable／tradable` | 可拾取、可检视、可交易 | 仅主模拟；没有 `stackable`，一件一格 | [[reference/components/inventoryitem]]、[[reference/components/tradable]]；[接线](../../../raw/current/scripts/prefabs/wintersfeastcookedfoods.lua#L57-L67) |
| 组件：`bait` | 可塞进陷阱当诱饵 | 仅主模拟 | [[reference/components/bait]]；[接线](../../../raw/current/scripts/prefabs/wintersfeastcookedfoods.lua#L65-L65) |
| 组件：`burnable／propagator／hauntable` | `MakeSmallBurnable`＋`MakeSmallPropagator` 可烧可引燃；`MakeHauntableLaunchAndPerish` 只加 `hauntable` 并抛出（perish 分支被注释） | 主模拟 | [[reference/components/burnable]]、[[reference/components/hauntable]]；[接线](../../../raw/current/scripts/prefabs/wintersfeastcookedfoods.lua#L61-L63) |
| 标签：`wintersfeastcookedfood` | 餐桌 `ItemTradeTest` 与 `feasts` 统计菜品都读它 | 两端都在 pristine 前加 | [[reference/components/wintersfeasttable]]；[标签](../../../raw/current/scripts/prefabs/wintersfeastcookedfoods.lua#L41-L41) |
| 组件：`wintersfeasttable`（消费者） | 桌上唯一菜品槽；`DepleteFood` 每 tick 扣 1 份并分派"还有剩／已归零"两个回调 | 服务端；桌上有带该标签的菜才 `canfeast` | [[reference/components/wintersfeasttable]]；[扣份](../../../raw/current/scripts/components/wintersfeasttable.lua#L65-L81) |
| 组件：`feasts`（消费者） | 世界组件：每 1/3 秒对每组每桌扣份，并给每名进食者续 `wintersfeastbuff` | 仅主模拟（组件开头 assert） | [[reference/components/feasts]]；[tick](../../../raw/current/scripts/components/feasts.lua#L80-L101) |
| 组件：`madsciencelab`（上游） | 烤炉用它的 `stages` 推进两段计时，结束时回调 `OnScienceWasMade` | 仅主模拟 | [[reference/components/madsciencelab]]；[接线](../../../raw/current/scripts/prefabs/wintersfeastoven.lua#L353-L357) |
| 关联 Prefab：`wintersfeastoven` | 唯一烤制设施，两段科学流程出菜 | 服务端；`TECH.WINTERS_FEAST` 建造 | 源码入口（Wiki 待建）：[工厂](../../../raw/current/scripts/prefabs/wintersfeastoven.lua#L312-L357) |
| 关联 Prefab：`table_winters_feast` | 扣冬宴份数的设施，投菜→开餐→清桌；末份归零后先置 `persists = false`、延迟 0–0.5 秒再 drop 并 `Remove` | 服务端；`TECH.WINTERS_FEAST` 建造 | 源码入口（Wiki 待建）：[投菜与开餐](../../../raw/current/scripts/prefabs/table_winters_feast.lua#L201-L243) |
| 关联 Prefab：`wintersfeastbuff` | 开餐期间挂在进食者身上的世界 buff，每人每 tick 续一次 | 主模拟；由 `feasts` 直接 `AddDebuff` | 源码入口（Wiki 待建）：[添加](../../../raw/current/scripts/components/feasts.lua#L73-L77) |
| 关联 Prefab：`winters_feast_food_depleted／winters_feast_depletefood` | 扣份与清桌时的一次性特效 | 主模拟生成 | [[reference/prefabs/winters_feast_depletefood]]；[清桌](../../../raw/current/scripts/prefabs/table_winters_feast.lua#L148-L190) |
| 关联 Prefab：`wintersfeastfuel` | 18 条配方的固定第一味材料 | 由配方消耗 | 源码入口（Wiki 待建）：[配方](../../../raw/current/scripts/recipes.lua#L1430-L1447) |
| 数据：`cooktime` | 每件菜的烤制倍率，只被烤炉读 | 仅服务端；缺省回落 1 | [读取](../../../raw/current/scripts/prefabs/wintersfeastoven.lua#L103-L105) |
| 资源：`anim/food_winters_feast_2019.zip` | 18 件共用一 bank／build，靠 `swap_food` 换菜色；另有 `INV_IMAGE` 同名图标 | 原版资源，不随页分发 | [声明](../../../raw/current/scripts/prefabs/wintersfeastcookedfoods.lua#L5-L9)、[覆盖](../../../raw/current/scripts/prefabs/wintersfeastcookedfoods.lua#L25-L29) |

## 典型接入

### 改一道菜的份数

- 前提与选择：服务端 `modmain.lua` 的 postinit。份数存在 `finiteuses` 实例上，改组件比改 `TUNING.WINTERSFEASTBUFF.EATTIME` 安全——后者是所有 18 件菜的份数默认值，一改就一起改掉 18 件的份数、从而改变可开餐秒数（节奏固定 1/3 秒，[tick](../../../raw/current/scripts/components/feasts.lua#L24-L24)）。
- 生效与撤销：`SetMaxUses` 只改分母，必须紧跟 `SetUses`；本页这 18 件没有 `SetOnFinished`，所以新份数只影响 `DepleteFood` 何时走 `GetUses() > 0` 分支，不改变"由餐桌删件"的收尾方式。
- 原版依据：接线在 `MakeFood` 的 `SetMaxUses／SetUses`，消耗侧在 `WintersFeastTable:DepleteFood`。[接线](../../../raw/current/scripts/prefabs/wintersfeastcookedfoods.lua#L51-L54)、[消耗](../../../raw/current/scripts/components/wintersfeasttable.lua#L65-L81)

```lua
-- modmain.lua：把烤火鸡从 180 份改成 60 份（仅服务端）
AddPrefabPostInit("roastturkey", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    local uses = inst.components.finiteuses
    if uses ~= nil then
        uses:SetMaxUses(60)
        uses:SetUses(60)
    end
end)
```

- 调整与验证：换成任意一个注册名即可逐件改；60 份按 1/3 秒一口推导为 20 秒开餐时间。**必须用本轮新烤或新生成的菜验证**：`finiteuses:OnSave` 只在 `current ~= total` 时写 `uses`，`OnLoad` 又直接 `SetUses(存档值)`，改过上限的旧菜会读成「剩 100／上限 60」这类当前值大于上限的错位。餐桌侧看 `getstatus` 进入 `HAS_FOOD`（`DISH_READY` 是烤炉的状态），开餐时看 `finiteuses:GetUses()` 每 1/3 秒减 1，清桌时看是否生成 `winters_feast_food_depleted`。[存档](../../../raw/current/scripts/components/finiteuses.lua#L48-L58)、[餐桌状态](../../../raw/current/scripts/prefabs/table_winters_feast.lua#L325-L330)、[烤炉状态](../../../raw/current/scripts/prefabs/wintersfeastoven.lua#L284-L288)

## 常见陷阱与限制

- **误区：冬宴菜可以像普通食物那样吃、会腐烂、能叠。** 工厂没有 `AddComponent("edible")`、没有 `AddComponent("perishable")`、没有 `stackable`；EAT 候选只认 `edible` 分支，所以它们不能直接吃，但仍可上冬宴餐桌或当陷阱诱饵，且永远不会变坏、一格一件。[组件表](../../../raw/current/scripts/prefabs/wintersfeastcookedfoods.lua#L51-L69)、[EAT 分支](../../../raw/current/scripts/componentactions.lua#L2647-L2689)
- **边界：180 不是"180 秒"，也不按人数分摊。** 它是 `finiteuses` 的份数，被 1/3 秒一次的 tick 每次扣 1，折合 60 秒；`DepleteFood(#feasters)` 的人数实参在组件内未被使用。推导结论，未运行验证。[常量](../../../raw/current/scripts/tuning.lua#L5182-L5182)、[tick](../../../raw/current/scripts/components/feasts.lua#L80-L98)、[Use](../../../raw/current/scripts/components/wintersfeasttable.lua#L65-L69)
- **边界：cooktime 是"每段"的倍率，不是总时长。** 两段各 `BASE_STAGE_COOK_TIME * cooktime`，所以总时长是 `2 * 9.999 * cooktime`；把 1.4 当成"28 秒基准"会差一倍。[改写](../../../raw/current/scripts/prefabs/wintersfeastoven.lua#L94-L101)、[推进](../../../raw/current/scripts/components/madsciencelab.lua#L23-L41)
- **边界：配方 product 是假 prefab。** 直接 `SpawnPrefab("wintercooking_lutefisk")` 得到的是 `dummy_fn` 的空壳（无网、`CLASSIFIED`、主模拟下一帧删除），不是菜；要菜得自己按去掉 `wintercooking_` 前缀求菜名（名单见同文件 `FOOD_PREFABS`）；`WINTERS_FEAST_COOKED_FOODS` 只是 `wintersfeastoven.lua:57` 的 local 表，模组拿不到，需自建映射。[dummy](../../../raw/current/scripts/prefabs/wintersfeastoven.lua#L413-L428)、[映射表](../../../raw/current/scripts/prefabs/wintersfeastoven.lua#L57-L63)
- **边界：普通食物进不了开餐流程，但被拒的环节不同。** 入口 `ItemTradeTest` 只放行带 `wintersfeastcookedfood` 或 `preparedfood` 的物品，所以腐烂物这类两样标签都没有的连上桌都做不到；锅菜与香料菜带 `preparedfood`，能过入口并真的上架，随后在 `OnGetShelfItem` 的"非 `wintersfeastcookedfood`"分支被 `RefuseFood` 弹回。[入口](../../../raw/current/scripts/prefabs/table_winters_feast.lua#L42-L53)、[上架后拒绝](../../../raw/current/scripts/prefabs/table_winters_feast.lua#L205-L241)
- **边界：末份归零不等于立刻下桌。** `OnFinishFood` 先把槽里那件标成 `persists = false`，再在 `0–0.5` 秒后 drop 并 `Remove`；这段延迟里 `feasts` 的 tick 仍读得到槽内的菜，把它算进菜品数、种类数与开餐增益，之后才真正清空。[清桌](../../../raw/current/scripts/prefabs/table_winters_feast.lua#L169-L190)、[统计](../../../raw/current/scripts/components/feasts.lua#L57-L78)
- **未核验**：未运行游戏。60 秒开餐时间、"每段 ≈8／10／12／14 秒"的实际观感、以及多桌同组时是否都同步扣份都按源码推导；`wintersfeastbuff` 本体的数值不在本页范围。

## 验证记录

- 修订记录（外部第四轮）：快速判断「只有 finiteuses」改为「以 finiteuses 记份数」、导航餐桌职责收窄为「扣冬宴份数的设施」；其余段落未动。
- 修订记录（外部第三轮）：陷阱「只能进餐桌」改为「不能直接吃，但仍可上餐桌或当诱饵」，与快速判断、`bait` 导航行统一；其余段落未动。
- 修订记录（外部第二轮）：快速判断改为「不能直接吃，但可上餐桌或作诱饵」、归零分支改写为两阶段（先 `persists = false`，0–0.5 秒后才清）；其余段落未动。
- 修订记录（外部复检）：接外部复检报告后按点名条目修订 P1-1（示例改用新菜＋餐桌状态改为 `HAS_FOOD`）、P2-2（补交易门禁与 `bait`）、P2-3（入口拒收与上架后弹回分开写）、P2-4（末份延迟清槽；摘要／导航／陷阱同步），其余段落未动。
- 修订记录：2026-11-09 第一轮独立检视后按点名条目修订（P1 配料物品名 2 处、P2 配料显示名 10 处（7 处点名＋3 处补充）＋local 映射表＋餐桌锚点、P3 覆盖项范围＋配方锚点起点＋EATTIME 表述），其余段落未动；其中 cutreeds／cutgrass／stinger 三处为检视补充条目，同批并入。
- 源码审阅：2026-11-09，Steam buildid 26474873。通读 `prefabs/wintersfeastcookedfoods.lua`（工厂、1 个标签、8 个组件）与数据表 `wintersfeastcookedfoods.lua`（18 项 `cooktime` 与注释掉的覆盖项）；`prefabs/wintersfeastoven.lua` 的 `BASE_STAGE_COOK_TIME`、`FOOD_PREFABS`、`WINTERS_FEAST_COOKED_FOODS`、`SetupDish`、`SetOvenCookTimeMultiplier`、`GetWinterFoodCookTimeMultiplier`、`CanCookPrefab`、`StartMakingScience`、`OnScienceWasMade`、`dummy_fn` 与注册列表；`components/madsciencelab` 的 `SetStage／StartMakingScience`；`prefabs/table_winters_feast.lua` 的 `ItemTradeTest`、`OnGetShelfItem`、`OnFinishFood`；`components/wintersfeasttable` 的 `DepleteFood` 与 `components/feasts` 的 `doFeastCheck／ApplyFeastBuff`；`componentactions` 的 EAT 分支；`recipes.lua` 的 18 条 `wintercooking_*` 与 `recipes_filter.lua` 的同名单；`constants` 的 `WINTERSFEASTCOOKING_ONE`、`tuning` 的 `WINTERSFEASTBUFF` 与 `WINTERS_FEAST_OVEN_BASE_COOK_TIME`；`languages/chinese_s.po` 的 18 个 `NAMES`。
- 名称核验：18 个键都是 `STRINGS.NAMES.<大写注册名>`（无下划线），逐条读 `msgstr`：快乐浆果酱／比宾卡米糕／白菜卷／节庆鱼料理／好肉汁／土豆饼／苏打鱼／香料潘趣酒／托尼甜面包／巴甫洛娃蛋糕／腌鲱鱼／波兰饼干／南瓜派／烤火鸡／烤火鸡面包馅／红薯焗饭／塔马利／饕餮馅饼。表格每行给了对应 `msgstr` 行锚点。
- 交叉核对：`screens/redux/scrapbookdata.lua` 同项写作 `finiteuses=180`、`burnable=true`、`subcat="wintersfeastfood"`，与源码一致。[采集册](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L167-L167)
- 未覆盖：`wintersfeastbuff` 的时长与增益曲线、烤炉 `pickable` 取菜动作、`wintersfeastfuel` 的来源、以及 `recipes_filter.lua` 的筛选用途。
- 游戏运行：未运行。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 见交付回报。

## 相关链接

- 消耗侧：[[reference/components/wintersfeasttable]]、[[reference/components/feasts]]
- 生产侧：[[reference/components/madsciencelab]]
- 特效与 buff：[[reference/prefabs/winters_feast_depletefood]]
- 共用材料：[[reference/prefabs/petals]]、[[reference/prefabs/moontree-blossom]]
- 同批寄居蟹茶：[[reference/prefabs/hermitcrab-tea]]

返回 [[reference/prefabs/special-food-index|节庆、角色与特殊食品索引]]。
