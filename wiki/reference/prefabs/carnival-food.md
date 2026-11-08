---
title: "玉米泥（carnivalfood_corntea）"
created: "2026-11-09"
updated: "2026-11-09"
type: prefab
size_class: simple
summary: "狂欢节奖品摊位用 12 张奖票换的饮料：VEGGIE 9.375／0／5、−40 度 15 秒冷食、3 天腐坏成 spoiled_food，也是小乌鸦（carnival_crowkid）接受的两种零食之一（换 3–8 张奖票＋1 鸦年华代币）。"
aliases: ["carnivalfood_corntea", "玉米泥", "Corny Slush", "狂欢节玉米泥", "小乌鸦零食"]
tags: [prefab, item, food]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/carnival_food.lua", "stategraphs/SGwilson.lua", "modutil.lua", "mainfunctions.lua", "components/trader.lua", "recipes.lua", "recipes_filter.lua", "constants.lua", "tuning.lua", "recipe.lua", "prefabs/carnival_prizebooth.lua", "prefabs/carnival_crowkid.lua", "stategraphs/SGcarnival_crowkid.lua", "components/edible.lua", "components/perishable.lua", "components/tradable.lua", "components/builder.lua", "widgets/craftslot.lua", "standardcomponents.lua", "actions.lua", "prefabs/warly.lua", "components/eater.lua", "scrapbook_prefabs.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 玉米泥（carnivalfood_corntea）

## 快速判断

- 职责与适用：`prefabs/carnival_food.lua` 的 `food_defs` 只有一项，构造出**一个注册** `carnivalfood_corntea`（简中"玉米泥"），是可堆叠、会腐坏、可点燃的食物饮料；页面适合查"活动摊位用奖票换食物"与"NPC 收零食"两类接线。[注册](../../../raw/current/scripts/prefabs/carnival_food.lua#L81-L81)
- 优先使用：查数值、温度与鲜度看[注册与构造](#注册与构造)和[关联机制导航](#关联机制导航)；查怎么获得看 `Recipe2` 行；查小乌鸦的零食换奖看[关联机制导航](#关联机制导航)的 `carnival_crowkid` 行。
- 重要副作用：**腐坏从构造那一刻开始**，3 天＝1440 秒（定义值；倍率见[鲜度口径](#鲜度口径)）后替换成 `spoiled_food`；吃下给 −40 度、15 秒的冷食修正。[计时](../../../raw/current/scripts/prefabs/carnival_food.lua#L51-L56)
- 修改边界：奖票 `carnival_prizeticket` 的产出、奖品摊位货架与小乌鸦的活动 AI 不展开；本页只负责该注册的工厂、数值、标签、配方与两条消费链（[工厂](../../../raw/current/scripts/prefabs/carnival_food.lua#L7-L82)）。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `carnivalfood_corntea` | `MakeFood(def)` 返回 `fn`（L7-L82），外层 `Prefab(def.name, fn, assets)`（L81）；`food_defs` 只此一项（L2-L5），循环 L84-L89 | 动画 bank／build `carnival_food`、anim `corn_tea`（L18-L21）；两端标签 `fooddrink`＋`pre-preparedfood`（L23-L28）；`floater` size `small`／scale 0.9（L30-L35）；主模拟挂 `edible`（VEGGIE 9.375／0／5，−40 度×15 秒）、`perishable`（3 天→`spoiled_food`）、`stackable` 40、`tradable`、`inspectable`、`inventoryitem`、`MakeSmallBurnable` 7.5 秒＋`MakeSmallPropagator`、`MakeHauntableLaunch`（L43-L68）。[工厂](../../../raw/current/scripts/prefabs/carnival_food.lua#L7-L82) |

`food_defs` 没有第二形态；`assets` 只声明 `anim/carnival_food.zip`（L73-L76），bank 与 build 都取 `def.art`。`scrapbook_anim` 与播放动画同为 `corn_tea`（L20-L21）。

### 鲜度口径

基准：`seg_time`＝30 秒、`total_day_time`＝480 秒、`perish_warp`＝1，天数按 480 秒／天；1440 秒是 modifier=1 的定义值（手持／普通容器）。`perishable:Update` 无 owner 时乘 `PERISH_GROUND_MULT`（1.5，约 960 秒），`fridge` 且未冻结时乘 `PERISH_FRIDGE_MULT`（0.5，约 2880 秒），`preserver` 优先于 `fridge`。落地秒数按常温／干燥／无酸雨／默认倍率；落地且 `isacidraining`、无 `rainimmunity`、`CanPointHaveAcidRain` 同时成立时另有额外折旧，潮湿 ×1.3；`ambient_temperature < 0`（未冻结）乘 `PERISH_WINTER_MULT`（0.75），`ambient_temperature > OVERHEAT_TEMP`（70）乘 `PERISH_SUMMER_MULT`（1.25）。剩余比例挂 `fresh`／`stale`／`spoiled` 标签供 `edible` 判衰减。[基准](../../../raw/current/scripts/tuning.lua#L19-L20)、[冰箱](../../../raw/current/scripts/components/perishable.lua#L80-L86)、[落地／酸雨](../../../raw/current/scripts/components/perishable.lua#L97-L125)、[标签](../../../raw/current/scripts/components/perishable.lua#L3-L22)、[结算](../../../raw/current/scripts/components/perishable.lua#L129-L133)、[倍率](../../../raw/current/scripts/tuning.lua#L2079-L2088)

### 名称核验

简中 `STRINGS.NAMES.CARNIVALFOOD_CORNTEA`＝**玉米泥**（msgid `Corny Slush`），注册名与 `NAMES` 键同名。[名称](../../../raw/current/scripts/languages/chinese_s.po#L340264-L340267)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`edible` | 写 `foodtype=VEGGIE`、hunger 9.375、health 0、sanity 5、`temperaturedelta=-40`、`temperatureduration=15` | 仅主模拟；`foodtype` 赋值同时挂 `edible_VEGGIE` 标签 | [[reference/components/edible]]；[赋值](../../../raw/current/scripts/prefabs/carnival_food.lua#L43-L49)、[饥饿结算](../../../raw/current/scripts/components/edible.lua#L118)、[生命](../../../raw/current/scripts/components/edible.lua#L142)、[理智](../../../raw/current/scripts/components/edible.lua#L93)、[温度](../../../raw/current/scripts/components/edible.lua#L202-L207)、[温度档](../../../raw/current/scripts/tuning.lua#L2326-L2332)、[热量档](../../../raw/current/scripts/tuning.lua#L2058-L2058)、[理智档](../../../raw/current/scripts/tuning.lua#L2032-L2038) |
| 组件：`perishable` | `SetPerishTime(PERISH_SUPERFAST)`＋`StartPerishing()`＋`onperishreplacement="spoiled_food"` | 构造即开始，没有 StopPerishing／进背包开关 | [[reference/components/perishable]]；[接线](../../../raw/current/scripts/prefabs/carnival_food.lua#L51-L56)、[计时](../../../raw/current/scripts/components/perishable.lua#L262-L271)、[替换](../../../raw/current/scripts/components/perishable.lua#L293-L304) |
| 组件：`stackable` | `maxsize = TUNING.STACK_SIZE_SMALLITEM`＝40 | 主模拟 | [[reference/components/stackable]]；[接线](../../../raw/current/scripts/prefabs/carnival_food.lua#L58-L59)、[常量](../../../raw/current/scripts/tuning.lua#L83-L83) |
| 组件：`burnable`／`propagator` | `MakeSmallBurnable(SMALL_BURNTIME)`＝7.5 秒；`MakeSmallPropagator` 传热引燃（flashpoint 5–10、range 3–5） | 仅主模拟；同族 yotp／yotr 不挂 | [[reference/components/burnable]]；[接线](../../../raw/current/scripts/prefabs/carnival_food.lua#L65-L66)、[helper](../../../raw/current/scripts/standardcomponents.lua#L164-L174)、[传播参数](../../../raw/current/scripts/standardcomponents.lua#L200-L213)、[常量](../../../raw/current/scripts/tuning.lua#L1681-L1681) |
| 组件：`tradable` | 只 `AddComponent("tradable")`，未写 `goldvalue`，保持默认 0 | 主模拟；`goldvalue` 默认 0，不是按金值交易的物品 | [[reference/components/tradable]]；[接线](../../../raw/current/scripts/prefabs/carnival_food.lua#L61-L61)、[默认值](../../../raw/current/scripts/components/tradable.lua#L1-L7) |
| 标签：`fooddrink` | 让 EAT 的 `strfn` 返回 `DRINK`（界面／动作串），不改变动作类型 | 两端，`SetPristine` 之前加 | [数据](../../../raw/current/scripts/prefabs/carnival_food.lua#L4-L4)、[消费](../../../raw/current/scripts/actions.lua#L787-L793) |
| 标签：`pre-preparedfood` | 沃利 `SetPrefersEatingTag("pre-preparedfood")`，`PrefersToEat` 对不在偏好标签里的食物返回 false | 两端；只对带该偏好的进食者生效 | [加标签](../../../raw/current/scripts/prefabs/carnival_food.lua#L28-L28)、[沃利](../../../raw/current/scripts/prefabs/warly.lua#L35-L38)、[判定](../../../raw/current/scripts/components/eater.lua#L304-L318)、[EAT 入口](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1228-L1240) |
| API：`Recipe2("carnivalfood_corntea")` | 12 张奖票（`carnival_prizeticket`）；`TECH.CARNIVAL_PRIZESHOP_ONE`；`nounlock`、`no_deconstruction`、`actionstr="CARNIVAL_PRIZESHOP"`、`sg_state="give"` | 奖品摊位入口，不是世界掉落 | [配方](../../../raw/current/scripts/recipes.lua#L1195-L1195)、[TECH](../../../raw/current/scripts/constants.lua#L1285-L1285)、[配方页签定义](../../../raw/current/scripts/constants.lua#L1552-L1552)、[站科技树](../../../raw/current/scripts/tuning.lua#L1373-L1375)、[站接线](../../../raw/current/scripts/prefabs/carnival_prizebooth.lua#L131-L135)、[筛选表](../../../raw/current/scripts/recipes_filter.lua#L398-L398)、[条目](../../../raw/current/scripts/recipes_filter.lua#L793-L793) |
| 关联 Prefab：`carnival_crowkid` | `trader:SetAcceptTest` 只认 `corn_cooked` 或 `carnivalfood_corntea`；`OnGetItemFromPlayer` 先写 `has_snack`＋剩余鲜度×0.5 计时、切 `give_reward`，再 `item:Remove()`；奖励抛 3＋ceil(random×5) 张奖票＋1 鸦年华代币 | 服务端；物品只比 prefab 名、要求 `has_snack == nil`；`AbleToAccept` 要 enabled、未死、未睡、非busy | [接受](../../../raw/current/scripts/prefabs/carnival_crowkid.lua#L29-L40)、[门禁](../../../raw/current/scripts/components/trader.lua#L103-L121)、[接线](../../../raw/current/scripts/prefabs/carnival_crowkid.lua#L126-L130)、[奖励](../../../raw/current/scripts/stategraphs/SGcarnival_crowkid.lua#L16-L38) |
| 关联 Prefab：`spoiled_food` | 腐坏替换目标；替换时把原堆叠数复制给产物 | 到点由 `perishable:Perish` 生成 | [替换](../../../raw/current/scripts/components/perishable.lua#L293-L304) |
| 资源：`anim/carnival_food.zip` | bank／build `carnival_food`、anim `corn_tea` | 原版资源，不随页分发 | [assets](../../../raw/current/scripts/prefabs/carnival_food.lua#L18-L21)、[声明](../../../raw/current/scripts/prefabs/carnival_food.lua#L73-L76) |

## 典型接入

### 延长玉米泥鲜度

- 前提与选择：服务端 `modmain.lua` 的 `AddPrefabPostInit`；改实例的 `perishable`，不改 `TUNING.PERISH_SUPERFAST`（波及其它）。
- 生效与撤销：`AddPrefabPostInit` 按 prefab 名登记（[登记](../../../raw/current/scripts/modutil.lua#L593-L599)），每次 `SpawnPrefab` 构造（含读档重建）都会调用（[调用](../../../raw/current/scripts/mainfunctions.lua#L394-L407)）：`SetPerishTime` 先把总时长与剩余都设为新值；读档时 `perishable:OnLoad` 再用存档 `time` 覆盖 `perishremainingtime`（[读档](../../../raw/current/scripts/components/perishable.lua#L340-L346)），新生成是满的新总时长，读档实例的总时长变新、剩余时间保留存档值、百分比按新总时长重算。卸模组后新构造回 `PERISH_SUPERFAST`。
- 原版依据：鲜度接线在工厂主模拟段，没有第二处覆盖。

```lua
-- modmain.lua：把玉米泥的鲜度从 PERISH_SUPERFAST（3 天）延到 PERISH_SLOW（15 天）
AddPrefabPostInit("carnivalfood_corntea", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    local perishable = inst.components.perishable
    if perishable ~= nil then
        perishable:SetPerishTime(GLOBAL.TUNING.PERISH_SLOW)
    end
end)
```

- 调整与验证：`PERISH_SLOW`＝15 天＝15×480＝7200 秒（[档位](../../../raw/current/scripts/tuning.lua#L2049-L2049)）；验证看新生成与读档重建实例的鲜度百分比，不要只读组件字段。本页未运行游戏。

## 常见陷阱与限制

- **误区：玉米泥是世界掉落或烹饪锅产物。** 可核验的生产入口只有奖品摊位的 `Recipe2`（12 张奖票 `carnival_prizeticket`）；`carnival_popcorn` 是另一条配方且 `product="corn_cooked"`，两件不能混。[配方](../../../raw/current/scripts/recipes.lua#L1193-L1195)
- **误区：`nounlock=true` 等于免费或不需要站点。** 它只免去"已解锁／知道"的知识（`self.nounlock` 与 UI 的 `knows or recipe.nounlock`），配方受 `CanPrototypeRecipe(recipe.level, tech trees)` 与材料数量约束；本配方要 12 张票，科技树由 `carnival_prizebooth` 的 `prototyper` 提供。[定义](../../../raw/current/scripts/recipe.lua#L124-L131)、[UI 门禁](../../../raw/current/scripts/widgets/craftslot.lua#L168-L171)、[站科技树](../../../raw/current/scripts/prefabs/carnival_prizebooth.lua#L131-L135)
- **边界：工厂显式标签只有两个＋食性标签。** 构造函数加 `fooddrink` 与 `pre-preparedfood`；`edible_VEGGIE` 由 `edible.foodtype` 赋值挂上；`perishable` 另挂 `fresh`／`stale`／`spoiled`（见[鲜度口径](#鲜度口径)）。[标签](../../../raw/current/scripts/prefabs/carnival_food.lua#L23-L28)、[食性标签](../../../raw/current/scripts/components/edible.lua#L9-L17)
- **边界：腐坏从构造开始，3 天＝1440 秒（定义值；倍率见[鲜度口径](#鲜度口径)）。** `StartPerishing` 就在构造段，没有进背包才计时的分支；到点用 `SpawnPrefab("spoiled_food")` 替换并继承堆叠。[计时](../../../raw/current/scripts/prefabs/carnival_food.lua#L51-L56)、[基准](../../../raw/current/scripts/tuning.lua#L19-L20)、[鲜度档](../../../raw/current/scripts/tuning.lua#L2043-L2046)、[替换](../../../raw/current/scripts/components/perishable.lua#L293-L304)
- **边界：−40 度是"体温修正量×时长"，不是立刻降 40 度。** 只有 `temperaturedelta` 与 `temperatureduration` 都非 0、且 `chill<1` 时才走 `SetTemperatureInBelly`；辣椒粉等香料还会再乘倍率。[结算](../../../raw/current/scripts/components/edible.lua#L202-L207)、[常量](../../../raw/current/scripts/tuning.lua#L2326-L2332)
- **边界：小乌鸦只收两种零食。** `AcceptTest` 的物品侧只比对 prefab 名、不看新鲜度，但要求它当前没有 `has_snack`；快过期仍能换奖励（同正文 `trader` enabled、未睡、非busy），但 `has_snack` 冷却＝`perishremainingtime×0.5`，新鲜时 1440×0.5＝720 秒（推导）。[接受](../../../raw/current/scripts/prefabs/carnival_crowkid.lua#L29-L40)
- **未核验**：未运行游戏；采集册名单里 `carnivalfood_corntea` 被注释、实际是否显示未核验；`sg_state="give"` 在建造者状态机里的具体表现与配方在非狂欢节期间的可见性未核验。

## 验证记录

- 源码审阅：2026-11-09，build 26474873。通读 `prefabs/carnival_food.lua`；`recipes.lua`／`recipes_filter.lua`；`constants.lua` 的 `TECH` 与 `CARNIVAL_PRIZESHOP` 页签；`tuning.lua` 的时间、鲜度、热量、理智、温度、堆叠、燃烧常量；`carnival_prizebooth.lua` 的 `prototyper`；`carnival_crowkid.lua` 与 `SGcarnival_crowkid.lua`；`edible`／`perishable`／`tradable`／`standardcomponents` 结算；`recipe`／`craftslot`／`builder` 的 `nounlock`；`actions.lua` 的 `DRINK`；`warly`／`eater`；`chinese_s.po`；`scrapbook_prefabs.lua`。
- 名称核验：`NAMES.CARNIVALFOOD_CORNTEA`＝玉米泥，msgid `Corny Slush`。
- 游戏运行：未运行；天数、秒数与温度结算均为源码推导。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 见交付回执。
- 修订记录：2026-11-09 按内外检修订：鲜度倍率／标签／温湿酸雨、乌鸦顺序／门禁、读档。

## 相关链接

- 同族节庆供品：[[reference/prefabs/yotp-food]]、[[reference/prefabs/yotr-food]]
- 食物组件：[[reference/components/edible]]、[[reference/components/perishable]]、[[reference/components/stackable]]
- 另一条零食来源：[[reference/prefabs/farm-crops-staple]]（`corn_cooked`／爆米花）
- 站与配方语义：[[reference/components/prototyper]]、[[reference/components/builder]]

返回 [[reference/prefabs/special-food-index|节庆、角色与特殊食品索引]]。
