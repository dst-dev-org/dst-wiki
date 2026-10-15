---
title: "浆果与多汁浆果的生熟两态（berries、berries_juicy 与各自 _cooked）"
created: "2026-11-08"
updated: "2026-11-08"
type: prefab
summary: "veggies.lua 的 4 个无种子浆果注册：生熟三维、6／3 天与 2／1 天保质、BERRY 次级食性、可作一次性海钓拟饵，以及三种浆果丛的采摘、撞落与采后再生差异。"
aliases: ["berries", "berries_cooked", "berries_juicy", "berries_juicy_cooked", "浆果", "烤浆果", "多汁浆果", "烤多汁浆果", "浆果丛", "多汁浆果丛"]
tags: [prefab, item, food]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/veggies.lua", "prefabs/berrybush.lua", "prefabs/tumbleweed.lua", "prefabs/oceanfishingrod.lua", "prefabs/oceanfish.lua", "prefabs/oceanfishdef.lua", "prefabs/smallbird.lua", "containers.lua", "cooking.lua", "standardcomponents.lua", "components/pickable.lua", "components/edible.lua", "components/eater.lua", "components/perishable.lua", "components/stackable.lua", "components/cookable.lua", "components/bait.lua", "components/oceanfishingtackle.lua", "components/oceanfishingrod.lua", "components/snowmandecoratable.lua", "stategraphs/SGwilson.lua", "map/forest_map.lua", "map/customize.lua", "tuning.lua", "constants.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 浆果与多汁浆果的生熟两态（berries、berries_juicy 与各自 _cooked）

## 快速判断

- 职责与适用：`prefabs/veggies.lua` 的 `MakeVeggie` 为 `VEGGIES` 每项生成生／熟两件物品；本页覆盖 `berries`、`berries_juicy` 及各自 `_cooked` 共 4 个可食物品注册，来自 `SEEDLESS` 表内的两个 `VEGGIES` 键；因此没有种子、巨型与称重导出。[表项](../../../raw/current/scripts/prefabs/veggies.lua#L85-L101)、[SEEDLESS](../../../raw/current/scripts/prefabs/veggies.lua#L160-L168)、[注册循环](../../../raw/current/scripts/prefabs/veggies.lua#L1102-L1108)
- 优先使用：端分界、生熟公共组件集合、`cookable` 接线与堆叠／新鲜度通则见 [[reference/prefabs/farm-crops-staple#共用工厂与生熟两态]]；本页只写四键差异、获得路径与拟饵字段。三维与保质见[生熟两态数值](#生熟两态数值)。
- 重要副作用：两键都写 `secondaryfoodtype = FOODTYPE.BERRY`，生熟都带 `edible_BERRY` 次级标签；生件带 `oceanfishing_lure` tag 与 `oceanfishingtackle`，可装进海钓杆鱼饵格，中鱼后按 `single_use` 销毁；`berries_juicy` 采摘是抛出 3 颗而不是一叠入包。[写入](../../../raw/current/scripts/prefabs/veggies.lua#L595-L600)、[拟饵 tag](../../../raw/current/scripts/prefabs/veggies.lua#L585-L587)、[抛落](../../../raw/current/scripts/prefabs/berrybush.lua#L353-L360)
- 修改边界：不含种子注册、巨型／打蜡变体与农田生长，也不含烹饪锅食谱；浆果丛实体的施肥、枯萎与雪覆盖只写与采摘有关的分支。

## 注册与构造

4 件由 `MakeVeggie(name, not SEEDLESS[name])` 产出：两键各调一次、各得两件 Prefab（本体与熟态），导出顺序与公共骨架见 [[reference/prefabs/farm-crops-staple#导出集合与端分界]]。[导出段](../../../raw/current/scripts/prefabs/veggies.lua#L1084-L1099)

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `berries` | `fn`：bank／build＝`berries`、播 `idle`、`pickupsound`、tag `cookable`、浮水 `{"med", nil, 0.7}`、`lure_data` 存在时加 `oceanfishing_lure` tag → `SetPristine` → 主模拟 | 因在 `SEEDLESS` 内，跳过 `weighable` 与 `weighable_OVERSIZEDVEGGIES` tag；主模拟再挂 `edible`／`perishable`／`stackable`／`bait`／`cookable`／`oceanfishingtackle`／`snowmandecor`。[公共端](../../../raw/current/scripts/prefabs/veggies.lua#L553-L587)、[SEEDLESS 分支](../../../raw/current/scripts/prefabs/veggies.lua#L567-L570) |
| `berries_cooked` | `fn_cooked`：同 bank／build 播 `cooked`、设 `scrapbook_anim`、浮水 `{"med", nil, 0.65}` → `SetPristine` → 主模拟 | 没有 `cookable`／`weighable`／`oceanfishingtackle`／`snowmandecor`，堆叠恒写 40。[熟工厂](../../../raw/current/scripts/prefabs/veggies.lua#L707-L777) |
| `berries_juicy` | `fn`，公共端与 `berries` 同构，拟饵符号不同 | 生件浮水与 `berries` 同为 `{"med", nil, 0.7}`；`lure_data.symbol = "hook_juiceberries"`；`cooked_float_settings` 为空，故熟件回落到默认浮水（两键的浮水差异只在熟件）。[表项](../../../raw/current/scripts/prefabs/veggies.lua#L94-L101)、[默认浮水](../../../raw/current/scripts/standardcomponents.lua#L1772-L1784) |
| `berries_juicy_cooked` | `fn_cooked` | `PERISH_ONE_DAY`＝1 天，是 `VEGGIES` 表内唯一用该档的食物；饥饿 18.75。[表项](../../../raw/current/scripts/prefabs/veggies.lua#L95) |

### 生熟两态数值

一天 `total_day_time`＝480 秒（`seg_time`=30×16），`perish_warp`=1，故天数与秒数可直接换算；饥饿基准 `calories_per_day`=75。[基准](../../../raw/current/scripts/tuning.lua#L19-L20)、[perish_warp](../../../raw/current/scripts/tuning.lua#L47)、[热量](../../../raw/current/scripts/tuning.lua#L41)、[PERISH_*](../../../raw/current/scripts/tuning.lua#L2043-L2051)、[CALORIES_*](../../../raw/current/scripts/tuning.lua#L2058-L2062)、[HEALING_*](../../../raw/current/scripts/tuning.lua#L2021-L2023)

| 注册 | 饥饿 | 生命 | 理智 | 新鲜度 | 堆叠上限 |
|---|---:|---:|---:|---|---:|
| `berries` | 9.375（`CALORIES_TINY`） | 0 | 0 | 6 天（2880 秒，`PERISH_FAST`） | 40 |
| `berries_cooked` | 12.5（`CALORIES_SMALL`） | 1（`HEALING_TINY`） | 0 | 3 天（1440 秒，`PERISH_SUPERFAST`） | 40 |
| `berries_juicy` | 12.5（`CALORIES_SMALL`） | 1（`HEALING_TINY`） | 0 | 2 天（960 秒，`PERISH_TWO_DAY`） | 40 |
| `berries_juicy_cooked` | 18.75（`CALORIES_MEDSMALL`） | 3（`HEALING_SMALL`） | 0 | 1 天（480 秒，`PERISH_ONE_DAY`） | 40 |

四键堆叠同为 40：生件走工厂的显式赋值（不在 `pumpkin`／`eggplant`／`durian`／`watermelon` 例外名单内），熟件恒写 `TUNING.STACK_SIZE_SMALLITEM`。[生件堆叠](../../../raw/current/scripts/prefabs/veggies.lua#L613-L619)、[熟件堆叠](../../../raw/current/scripts/prefabs/veggies.lua#L752-L753)、[档位](../../../raw/current/scripts/tuning.lua#L81-L83)

### 获得与入栏

| 注册 | 已核验获得路径 | 产出与交付 |
|---|---|---|
| `berries` | ①`berrybush`／`berrybush2` 采摘 ②挖起成熟的浆果丛 ③风滚草战利品 | ①采摘者有库存才尝试入包，否则落在地上 ②成熟时 1 颗＋`dug_<prefab>`（`berrybush2` 掉 `dug_berrybush2`）；枯萎或枯死才只给 2 根 `twigs`（摘空的丛仍给挖掘产物） ③每次开 3 件按权重抽取，`berries` 权重 1，不是必得。[采摘接线](../../../raw/current/scripts/prefabs/berrybush.lua#L342-L351)、[挖起](../../../raw/current/scripts/prefabs/berrybush.lua#L170-L198)、[风滚草表](../../../raw/current/scripts/prefabs/tumbleweed.lua#L173-L174)、[抽取](../../../raw/current/scripts/prefabs/tumbleweed.lua#L213-L245) |
| `berries_juicy` | ①`berrybush_juicy` 撞落采摘 ②挖起成熟的多汁浆果丛 | ①3 颗独立抛出（`dropheight`=3.5）②最多 3 颗＋`dug_berrybush_juicy`。[接线](../../../raw/current/scripts/prefabs/berrybush.lua#L353-L366)、[抛落分支](../../../raw/current/scripts/components/pickable.lua#L460-L470) |
| 两件 `_cooked` | 只能由生件烤出 | `cookable.product = name.."_cooked"`，已损失的新鲜度按半价传给熟件，见[[reference/prefabs/farm-crops-staple#共用工厂与生熟两态]]。[接线](../../../raw/current/scripts/prefabs/veggies.lua#L664-L665) |

两条入栏链不同：普通浆果丛不设 `droppicked`，产物走 `SpawnPrefab` 后 `inventory:GiveItem`（采摘者无库存时落地）；多汁浆果丛设 `droppicked`，改走 `lootdropper:SpawnLootPrefab` 逐个抛落，这也是同一处 `numtoharvest` 产生「3 个实体」而非「一叠 3」的原因。[交付分支](../../../raw/current/scripts/components/pickable.lua#L450-L518)、[SetUp 第三参](../../../raw/current/scripts/components/pickable.lua#L222-L228)

- **再生**：`pickable:SetUp` 的再生时长为 `BERRY_REGROW_TIME`（3 天）与 `BERRY_JUICY_REGROW_TIME`（9 天）；`getregentimefn` 再按「已耗轮次×0.5 天 + 0～2 天随机」加码，而轮次只有移植过的丛才会累积。`cycles_left` 同样只在 `transplanted` 时扣，野生丛不会因采摘耗尽；再生时长在春季再乘 `SPRING_GROWTH_MODIFIER`＝0.75。[普通再生](../../../raw/current/scripts/prefabs/berrybush.lua#L110-L121)、[多汁再生](../../../raw/current/scripts/prefabs/berrybush.lua#L123-L134)、[扣减条件](../../../raw/current/scripts/components/pickable.lua#L535-L539)、[常量](../../../raw/current/scripts/tuning.lua#L1774-L1782)
- **撞取**：`berries_juicy` 的 `jostlepick = true` 让动作走 `dojostleaction`（撞树而非点摘），`onworked_juicy` 在还没挖完（工作回调中途）时也可直接补一次 `Pick`。[接线](../../../raw/current/scripts/prefabs/berrybush.lua#L353-L366)、[动作](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1133)、[补摘](../../../raw/current/scripts/prefabs/berrybush.lua#L158-L168)
- **火鸡**：采摘后按 `PERD_SPAWNCHANCE`＝0.1（YOTG 活动期 0.3，均为幸运修正前的基值）在 3～6 秒后生成 `perd` 并设其为家；带 `berrythief` tag 的采摘者或丛上的 `_noperd` 标记不触发。[概率](../../../raw/current/scripts/prefabs/berrybush.lua#L87-L108)、[常量](../../../raw/current/scripts/tuning.lua#L546)

### BERRY 次级食性

- 来源：`VEGGIES` 第 13 位 `secondary_foodtype`，两键都传 `FOODTYPE.BERRY`，生熟两态读同一表项。[表项](../../../raw/current/scripts/prefabs/veggies.lua#L85-L101)、[生写入](../../../raw/current/scripts/prefabs/veggies.lua#L600)、[熟写入](../../../raw/current/scripts/prefabs/veggies.lua#L750)
- 标签：`edible` 的属性写入器对 `secondaryfoodtype` 追加 `edible_BERRY`，主食性仍是 `edible_VEGGIE`；写入器还断言主食性与次要食性不能相同。[写入器](../../../raw/current/scripts/components/edible.lua#L9-L17)
- 消费者：`eater:TestFood` 按 `edible_<类型>` 标签判定，饮食表含 `FOODTYPE.BERRY` 或 `FOODGROUP.BERRIES_AND_SEEDS`（展开为 `SEEDS`＋`BERRY`）的吃者因此能进食。[判定](../../../raw/current/scripts/components/eater.lua#L288-L302)、[组](../../../raw/current/scripts/constants.lua#L2065-L2073)
- 原版实例：`smallbird` 与未配 `diet` 的海钓鱼用 `BERRIES_AND_SEEDS` 组；唯一直接配 `DIET.BERRY` 的是冬季海钓鱼 `oceanfish_medium_8`，其诱饵偏好里 `berry` 权重 2.0。[小鸟](../../../raw/current/scripts/prefabs/smallbird.lua#L422)、[海钓默认](../../../raw/current/scripts/prefabs/oceanfish.lua#L354-L360)、[BERRY 饮食](../../../raw/current/scripts/prefabs/oceanfishdef.lua#L156-L161)、[冬季鱼](../../../raw/current/scripts/prefabs/oceanfishdef.lua#L1014-L1015)、[偏好](../../../raw/current/scripts/tuning.lua#L331)
- 常量注释写明该食性是「hack for smallbird；浆果其实属于 veggie」，所以它不改变主食性，也不参与烹饪锅分类。[注释](../../../raw/current/scripts/constants.lua#L2036)

### 海钓拟饵字段

- 字段：`{lure_data = TUNING.OCEANFISHING_LURE.BERRY, single_use = true, build = "oceanfishing_lure_mis", symbol = "hook_berries"}`；多汁浆果同构，只换 `symbol = "hook_juiceberries"`。[表项](../../../raw/current/scripts/prefabs/veggies.lua#L85-L101)
- 装配：公共端按 `lure_data ~= nil` 加 `oceanfishing_lure` tag 与资源 `anim/oceanfishing_lure_mis.zip`；主模拟 `oceanfishingtackle:SetupLure(VEGGIES[name].lure_data)`。[tag 与资源](../../../raw/current/scripts/prefabs/veggies.lua#L397-L399)、[装配](../../../raw/current/scripts/prefabs/veggies.lua#L667-L670)
- 数据：`OCEANFISHING_LURE.BERRY`＝`{charm = 0.3, reel_charm = -0.3, radius = 3.0, style = "berry", dist_max = 1}`。[常量](../../../raw/current/scripts/tuning.lua#L298-L300)
- 使用：鱼饵只能进海钓杆第 2 格（容器 `itemtestfn` 要求 `oceanfishing_lure` tag）；抛竿时杆用 `build`／`symbol` 覆盖线饵外观；中鱼后 `single_use` 使该物品从杆容器中删除。[槽位](../../../raw/current/scripts/containers.lua#L1342-L1346)、[外观](../../../raw/current/scripts/components/oceanfishingrod.lua#L177-L179)、[消耗](../../../raw/current/scripts/prefabs/oceanfishingrod.lua#L118-L136)
- 边界：只有生件有这套接线；熟件既无 `oceanfishingtackle` 也无 tag，不能当拟饵。组件接口与一次性语义见 [[reference/components/oceanfishingtackle#关键接口]]。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：edible | 写三维、主食性 `VEGGIE` 与 `secondary_foodtype`；赋值即重跑 `badfood` 与 `edible_*` 标签 | 生熟各一套；主模拟 | [[reference/components/edible#关键接口]]；[生](../../../raw/current/scripts/prefabs/veggies.lua#L595-L600)、[熟](../../../raw/current/scripts/prefabs/veggies.lua#L745-L750) |
| 组件：perishable | `SetPerishTime`＋`StartPerishing`，腐坏替换 `spoiled_food`；上限按秒计 | 生用 `perishtime`、熟用 `cooked_perishtime` | [[reference/components/perishable#关键接口]]；[生](../../../raw/current/scripts/prefabs/veggies.lua#L602-L605)、[熟](../../../raw/current/scripts/prefabs/veggies.lua#L740-L743) |
| 组件：stackable | 生熟都写 40；上限同步到 replica | 四键同档 | [[reference/components/stackable#关键接口]]；[生](../../../raw/current/scripts/prefabs/veggies.lua#L613-L619)、[熟](../../../raw/current/scripts/prefabs/veggies.lua#L752-L753) |
| 组件：cookable | 仅生件：`product = name.."_cooked"` | 生件、主模拟 | [[reference/components/cookable#关键接口]]；[接线](../../../raw/current/scripts/prefabs/veggies.lua#L664-L665) |
| 组件：bait | 无参数挂载，只提供陷阱诱饵的回调 | 生熟都有 | [[reference/components/bait#关键接口]]；[生](../../../raw/current/scripts/prefabs/veggies.lua#L657)、[熟](../../../raw/current/scripts/prefabs/veggies.lua#L765) |
| 组件：oceanfishingtackle | `SetupLure` 写 `lure_data` 与整份 `lure_setup` | 仅生件（含 `lure_data` 的两键） | [[reference/components/oceanfishingtackle#关键接口]]；[装配](../../../raw/current/scripts/prefabs/veggies.lua#L667-L670) |
| 组件：snowmandecor | 空标记组件；`snowmandecoratable` 表按 prefab 给出 `snowman_decor` 动画（bank／build 与物品同名） | 仅生件（两键都有） | [[reference/components/snowmandecor]]；[挂载](../../../raw/current/scripts/prefabs/veggies.lua#L679-L681)、[装饰映射](../../../raw/current/scripts/components/snowmandecoratable.lua#L9-L10) |
| 关联 Prefab：`berrybush`／`berrybush2`／`berrybush_juicy` | 三件植株共用 `createbush` 工厂，产物分别绑定 `berries` 与 `berries_juicy` | 世界实体；森林与洞穴都把它列为可生成资源 | [工厂与依赖](../../../raw/current/scripts/prefabs/berrybush.lua#L228-L241)、[三件注册](../../../raw/current/scripts/prefabs/berrybush.lua#L368-L370)、[资源展开](../../../raw/current/scripts/map/forest_map.lua#L187)、[世界归属](../../../raw/current/scripts/map/customize.lua#L357) |
| API／数据：烹饪食材 | 锅里 `berries`／`berries_juicy` 都是 `fruit=0.5`，`cancook` 让熟件继承 | 生熟同表 | [食材表](../../../raw/current/scripts/cooking.lua#L92-L93) |
| 资源：anim 包 | `anim/berries.zip`／`anim/berries_juicy.zip` 内含 `idle` 与 `cooked` 两个动画名；拟饵另用 `anim/oceanfishing_lure_mis.zip` | 四件注册共用 | [资源段](../../../raw/current/scripts/prefabs/veggies.lua#L391-L399) |

## 典型接入

### 抬高多汁浆果堆叠并延长烤制态的保质期

- 前提与选择：服务端 `modmain.lua`；两件是独立注册，必须分别 postinit，不能回灌 `VEGGIES` 表（表已在 prefab 文件加载时消费）。选 `SetNewMaxPerishTime` 而不是 `SetPerishTime`，因为前者按当前百分比换算、不会把已在背包里的物品回鲜。
- 生效与撤销：postinit 在每次构造（含读档重建）后执行；停用模组即恢复工厂原值。改 `stackable.maxsize` 会经 `onmaxsize` 同步到 replica。
- 原版依据：数值见[生熟两态数值](#生熟两态数值)；上限换算见 [SetNewMaxPerishTime](../../../raw/current/scripts/components/perishable.lua#L204-L208) 与 [GetPercent](../../../raw/current/scripts/components/perishable.lua#L215-L221)。

```lua
-- modmain.lua：多汁浆果堆到 60（STACK_SIZE_TINYITEM），烤多汁浆果保质期改回 3 天
local function retune(prefab, maxsize, perishtime)
    AddPrefabPostInit(prefab, function(inst)
        if not GLOBAL.TheWorld.ismastersim then
            return
        end
        if inst.components.stackable ~= nil and maxsize ~= nil then
            inst.components.stackable.maxsize = maxsize
        end
        if inst.components.perishable ~= nil and perishtime ~= nil then
            inst.components.perishable:SetNewMaxPerishTime(perishtime)
        end
    end)
end

retune("berries_juicy", 60, nil)
retune("berries_juicy_cooked", nil, 3 * GLOBAL.TUNING.TOTAL_DAY_TIME)
```

- 调整与验证：单位分别是个与秒；`3 * TOTAL_DAY_TIME` 等于 `PERISH_SUPERFAST` 的 1440 秒。`maxsize` 只影响此后构造的实例；读档实例先跑 postinit 设新上限，再由 `Perishable:OnLoad` 写回存档里的**旧剩余秒数**（绝对秒数，不按比例缩放），所以显示比例是「旧剩余÷新上限」且最多 1。

## 常见陷阱与限制

- **误区：浆果类生件默认只堆 20**：`stackable` 构造默认确为 `STACK_SIZE_MEDITEM`＝20，但生成器对四键都显式写 40，只有 `pumpkin`／`eggplant`／`durian`／`watermelon` 才落回默认。[例外名单](../../../raw/current/scripts/prefabs/veggies.lua#L613-L619)、[默认](../../../raw/current/scripts/components/stackable.lua#L23-L28)
- **误区：`berries_juicy` 一次摘到「一叠 3」**：当 `droppicked` 为真时走 `SpawnLootPrefab` 逐个抛 3 个独立实体；只有非抛落分支才会 `SetStackSize(numtoharvest)`。[两条分支](../../../raw/current/scripts/components/pickable.lua#L460-L502)
- **边界：野生浆果丛不会因采满而枯死**：`cycles_left` 只在 `transplanted` 时递减，所以未移植的丛可无限采摘；重新种下的丛由 `ontransplantfn` 立刻 `MakeBarren`，需施肥把 `cycles_left` 重置回 `max_cycles` 才恢复。[扣减](../../../raw/current/scripts/components/pickable.lua#L535-L539)、[移植回调](../../../raw/current/scripts/prefabs/berrybush.lua#L200-L204)、[施肥重置](../../../raw/current/scripts/components/pickable.lua#L274-L290)
- **边界：熟件不是「浆果二代」**：熟件没有 `cookable`、`weighable`、拟饵组件与 tag，只剩三维、保质、堆叠与 `bait`／`tradable`。[熟件主模拟段](../../../raw/current/scripts/prefabs/veggies.lua#L740-L776)
- **边界：`edible_BERRY` 不是主分类**：主食性仍为 `VEGGIE`，锅里的身份也按 `fruit=0.5` 算；把 BERRY 当「另一类食物」会同时漏掉 veggie 侧的判定。[写入器](../../../raw/current/scripts/components/edible.lua#L9-L17)、[食材表](../../../raw/current/scripts/cooking.lua#L92-L93)
- **未核验**：浆果丛在 quagmire 模式的 `event_server_data("quagmire", "prefabs/berrybush")` 分支（事件文件不在本快照）、打蜡／皮肤变体的外观链。[钩子](../../../raw/current/scripts/prefabs/berrybush.lua#L328-L330)

## 验证记录

- 源码审阅：2026-11-08，Steam buildid 26335394。核对 `veggies.lua` 全文（`MakeVegStats` 位序、两键表项、`fn`／`fn_cooked`、拟饵与资源分支、导出与注册段、`SEEDLESS`）、`berrybush.lua` 全文（三件注册、`normal_postinit`／`juicy_postinit`、挖起与火鸡分支）、`pickable` 的 `SetUp`／`Pick`／`SpawnProductLoot`、`edible` 的标签写入器与腐坏结算、`eater` 的 `TestFood`、`stackable`／`perishable`／`cookable`／`bait` 的相关分支、`oceanfishingtackle`／`oceanfishingrod` 的拟饵读取与消耗、`containers` 的鱼饵槽判定、`oceanfish`／`oceanfishdef`／`smallbird` 的饮食、`tumbleweed` 的战利品抽取、`cooking` 的食材表、`snowmandecoratable` 的装饰映射、`tuning` 的基准与 `PERISH_*`／`CALORIES_*`／`HEALING_*`／`BERRY_*` 再生常量、`constants` 的 `FOODTYPE`，以及简中 4 个名称键。
  名称核验：`NAMES.BERRIES`＝浆果、`BERRIES_COOKED`＝烤浆果、`BERRIES_JUICY`＝多汁浆果、`BERRIES_JUICY_COOKED`＝烤多汁浆果。[BERRIES](../../../raw/current/scripts/languages/chinese_s.po#L339395-L339397)、[BERRIES_COOKED](../../../raw/current/scripts/languages/chinese_s.po#L339400-L339402)、[BERRIES_JUICY](../../../raw/current/scripts/languages/chinese_s.po#L339405-L339407)、[BERRIES_JUICY_COOKED](../../../raw/current/scripts/languages/chinese_s.po#L339410-L339412)
  相关实体同批核对：`berrybush`／`berrybush2`＝浆果丛、`berrybush_juicy`＝多汁浆果丛、`perd`＝火鸡。[BERRYBUSH](../../../raw/current/scripts/languages/chinese_s.po#L339415-L339417)、[BERRYBUSH2](../../../raw/current/scripts/languages/chinese_s.po#L339420-L339422)、[BERRYBUSH_JUICY](../../../raw/current/scripts/languages/chinese_s.po#L339425-L339427)、[PERD](../../../raw/current/scripts/languages/chinese_s.po#L346800-L346802)
  未覆盖：浆果丛的皮肤／打蜡外观、施肥与雪覆盖的完整行为、火鸡 `perd` 的后续行为、quagmire 事件数据；生成点检索只覆盖 `pickable` 产品、挖起掉落与风滚草表，未穷举动态选名与模组注入。
- 游戏运行：未运行。三维、天数与秒数均为 tuning 推导（`CALORIES_TINY`＝75/8＝9.375、`PERISH_FAST`＝6×480 秒），撞取与抛落是代码路径推导。
- 结构检查：`validate_page.py` 通过（standard 档内）；`validate_source_links.py --strict` 0 错 0 警。

## 相关链接

- 同一工厂的其它类：[[reference/prefabs/farm-crops-staple]]（共用工厂与生熟两态）、[[reference/prefabs/farm-crops-fruit]]、[[reference/prefabs/farm-crops-spice]]
- 采摘与食用：[[reference/components/pickable]]、[[reference/components/edible]]、[[reference/components/eater]]
- 海钓与装饰：[[reference/components/oceanfishingtackle]]、[[reference/components/snowmandecor]]

返回 [[reference/prefabs/index|Prefab 索引]]。
