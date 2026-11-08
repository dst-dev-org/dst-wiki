---
title: "香蕉、无花果与仙人掌肉的六注册（cave_banana、fig、cactus_meat 及各自 _cooked）"
created: "2026-11-08"
updated: "2026-11-09"
type: prefab
summary: "veggies.lua 的 6 个无种子采集类食物注册：香蕉的 monkeyqueenbribe 进贡标签、无花果的 BERRY 次级食性与海钓拟饵、仙人掌肉生熟的正负值与夏季开花，以及三种植株／藤的采摘链。"
aliases: ["cave_banana", "cave_banana_cooked", "fig", "fig_cooked", "cactus_meat", "cactus_meat_cooked", "香蕉", "熟香蕉", "无花果", "熟无花果", "仙人掌肉", "熟仙人掌肉", "采集类水果"]
tags: [prefab, item, food]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/veggies.lua", "prefabs/cave_banana_tree.lua", "prefabs/bananabush.lua", "prefabs/oceanvine.lua", "prefabs/oceantree_pillar.lua", "prefabs/cactus.lua", "prefabs/monkeyqueen.lua", "prefabs/monkeyisland_portal.lua", "prefabs/oceanfishingrod.lua", "brains/powdermonkeybrain.lua", "map/layouts.lua", "map/forest_map.lua", "actions.lua", "componentactions.lua", "cooking.lua", "standardcomponents.lua", "containers.lua", "components/pickable.lua", "components/edible.lua", "components/eater.lua", "components/perishable.lua", "components/stackable.lua", "components/cookable.lua", "components/bait.lua", "components/oceanfishingtackle.lua", "components/oceanfishingrod.lua", "components/regrowthmanager.lua", "stategraphs/SGBeefalo.lua", "tuning.lua", "constants.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 香蕉、无花果与仙人掌肉的六注册（cave_banana、fig、cactus_meat 及各自 _cooked）

## 快速判断

- 职责与适用：`prefabs/veggies.lua` 的 `MakeVeggie` 为 `VEGGIES` 每项生成生／熟两件物品；本页覆盖 `cave_banana`、`fig`、`cactus_meat` 及各自 `_cooked` 共 6 个可食物品注册。三键都在 `SEEDLESS` 名单内，没有种子、巨型与称重导出，只能靠采集获得。[表项](../../../raw/current/scripts/prefabs/veggies.lua#L41-L43)、[SEEDLESS](../../../raw/current/scripts/prefabs/veggies.lua#L160-L168)、[注册循环](../../../raw/current/scripts/prefabs/veggies.lua#L1102-L1108)
- 优先使用：端分界、生熟公共组件集合、`cookable` 接线与堆叠／新鲜度通则见 [[reference/prefabs/farm-crops-staple#共用工厂与生熟两态]]；BERRY 次级食性的机制与消费端见 [[reference/prefabs/wild-berries#berry-次级食性]]。数值见[生熟两态数值](#生熟两态数值)。
- 重要副作用：香蕉两态都带 `monkeyqueenbribe` 进贡标签；无花果写 `secondaryfoodtype = FOODTYPE.BERRY` 并可当一次性海钓拟饵；生仙人掌肉的负数让它带 `badfood`，熟后反转为 +1 生命／+15 理智；采摘仙人掌在满足门禁时会被刺 6 点基础伤害（需 `combat`，排除 `bramble_resistant` 装备者与 `shadowminion`）。[额外标签](../../../raw/current/scripts/prefabs/veggies.lua#L157-L158)、[拟饵](../../../raw/current/scripts/prefabs/veggies.lua#L103-L109)、[badfood](../../../raw/current/scripts/components/edible.lua#L1-L7)、[刺伤](../../../raw/current/scripts/prefabs/cactus.lua#L18-L27)
- 修改边界：不含烹饪锅菜谱与配方消耗、`cactus_flower`（另一注册，数值见 [[reference/prefabs/petals]]）、各种植株本体行为（香蕉丛见 [[reference/prefabs/berry-bushes]]、苔藓藤条见 [[reference/prefabs/oceanvine]]、仙人掌见 [[reference/prefabs/cactus]]），以及 `_seeds`／巨型变体（三键都没有）。

## 注册与构造

6 件由 `MakeVeggie(name, not SEEDLESS[name])` 产出，每个键各得本体与熟态两件 Prefab，导出顺序见 [[reference/prefabs/farm-crops-staple#导出集合与端分界]]。[导出段](../../../raw/current/scripts/prefabs/veggies.lua#L1084-L1099)

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `cave_banana` | `fn`：bank／build＝`cave_banana`、播 `idle`、tag `cookable`、浮水 `{"small", 0.05, 0.9}`、写 `extra_tags_fresh` → `SetPristine` → 主模拟 | 唯一带额外标签的键（`monkeyqueenbribe`）；无 `lure_data`、无 `snowmandecor`、无 `weighable`。[表项](../../../raw/current/scripts/prefabs/veggies.lua#L41-L43)、[标签循环](../../../raw/current/scripts/prefabs/veggies.lua#L572-L576) |
| `cave_banana_cooked` | `fn_cooked`：同 bank／build 播 `cooked`、浮水 `{"med", nil, 0.75}`、写 `extra_tags_cooked` | 没有 `cookable`／拟饵组件，堆叠恒 40；进贡标签与生件同名。[熟件标签](../../../raw/current/scripts/prefabs/veggies.lua#L721-L725) |
| `fig` | `fn`：浮水 `{"med", nil, 0.7}`、`lure_data` 存在时加 `oceanfishing_lure` tag → `SetPristine` → 主模拟 | 写 BERRY 次级食性；无 `snowmandecor`。[表项](../../../raw/current/scripts/prefabs/veggies.lua#L103-L109)、[拟饵 tag](../../../raw/current/scripts/prefabs/veggies.lua#L585-L587) |
| `fig_cooked` | `fn_cooked`：浮水 `{"med", nil, 0.65}` | 仍写同一 BERRY 次级食性；无拟饵组件与 tag。[熟件写入](../../../raw/current/scripts/prefabs/veggies.lua#L745-L750) |
| `cactus_meat` | `fn`：不传浮水参数（回落 `MakeInventoryFloatable` 默认）、无额外标签 → `SetPristine` → 主模拟 | 生吃负生命／负理智 → 赋值即打 `badfood`；无拟饵。[表项](../../../raw/current/scripts/prefabs/veggies.lua#L111-L112)、[默认浮水](../../../raw/current/scripts/standardcomponents.lua#L1772-L1784) |
| `cactus_meat_cooked` | `fn_cooked`：同样不传浮水参数 | 生熟都用 `PERISH_MED`（10 天）；熟后理智 +15。[表项](../../../raw/current/scripts/prefabs/veggies.lua#L112) |

### 生熟两态数值

一天 `total_day_time`＝480 秒（`seg_time`=30×16），`perish_warp`=1，故天数与秒数可直接换算；饥饿基准 `calories_per_day`=75。[基准](../../../raw/current/scripts/tuning.lua#L19-L20)、[perish_warp](../../../raw/current/scripts/tuning.lua#L47)、[热量](../../../raw/current/scripts/tuning.lua#L41)、[PERISH_*](../../../raw/current/scripts/tuning.lua#L2043-L2051)、[CALORIES_*](../../../raw/current/scripts/tuning.lua#L2058-L2062)、[HEALING_*／SANITY_*](../../../raw/current/scripts/tuning.lua#L2021-L2035)

| 注册 | 饥饿 | 生命 | 理智 | 新鲜度 | 堆叠上限 |
|---|---:|---:|---:|---|---:|
| `cave_banana` | 12.5（`CALORIES_SMALL`） | 1（`HEALING_TINY`） | 0 | 10 天（4800 秒，`PERISH_MED`） | 40 |
| `cave_banana_cooked` | 12.5（`CALORIES_SMALL`） | 3（`HEALING_SMALL`） | 0 | 6 天（2880 秒，`PERISH_FAST`） | 40 |
| `fig` | 12.5（`CALORIES_SMALL`） | 0 | 0 | 6 天（2880 秒，`PERISH_FAST`） | 40 |
| `fig_cooked` | 18.75（`CALORIES_MEDSMALL`） | 1（`HEALING_TINY`） | 0 | 3 天（1440 秒，`PERISH_SUPERFAST`） | 40 |
| `cactus_meat` | 12.5（`CALORIES_SMALL`） | −3（`-HEALING_SMALL`） | −5（`-SANITY_TINY`） | 10 天（4800 秒，`PERISH_MED`） | 40 |
| `cactus_meat_cooked` | 12.5（`CALORIES_SMALL`） | 1（`HEALING_TINY`） | 15（`SANITY_MED`） | 10 天（4800 秒，`PERISH_MED`） | 40 |

六键堆叠都是 40：三键不在 `pumpkin`／`eggplant`／`durian`／`watermelon` 例外名单内，生件走显式赋值，熟件恒写 `TUNING.STACK_SIZE_SMALLITEM`。[生件堆叠](../../../raw/current/scripts/prefabs/veggies.lua#L613-L619)、[熟件堆叠](../../../raw/current/scripts/prefabs/veggies.lua#L752-L753)、[档位](../../../raw/current/scripts/tuning.lua#L81-L83)

### 获得路径

| 注册 | 已核验获得路径 | 关键参数与证据 |
|---|---|---|
| `cave_banana` | ①`cave_banana_tree`（洞穴香蕉树）采摘 ②砍倒该树时若仍可采再掉 1 个 ③`bananabush`（香蕉丛）长到第 4 阶段后采摘 ④猴岛布局把 1 丛 `bananabush` 加入候选表（候选保证，非每次实际放置）；非自然传送门（`monkeyisland_portal`）的战利品含 `dug_bananabush` | ①`CAVE_BANANA_GROW_TIME`＝4 天 ②先掉 `log`＋2 `twigs` [采摘](../../../raw/current/scripts/prefabs/cave_banana_tree.lua#L146-L158)、[砍倒](../../../raw/current/scripts/prefabs/cave_banana_tree.lua#L56-L74)；③④见下 |
| `fig` | `oceanvine`（苔藓藤条）采摘 | 每次 1 个，`OCEANVINE_REGROW_TIME`＝2.5 天；`max_cycles` 为空而 `cycles_left`＝1 [接线](../../../raw/current/scripts/prefabs/oceanvine.lua#L305-L314) |
| `cactus_meat` | `cactus`／`oasis_cactus` 采摘 | `CACTUS_REGROW_TIME`＝4 天；采摘者满足门禁时被刺 `CACTUS_DAMAGE`＝6（需 `combat`，排除刺抗装备与 `shadowminion`）；有采摘者且植株当前带花（`has_flower` 由夏季再生／唤醒置真）时另给 `cactus_flower` [接线](../../../raw/current/scripts/prefabs/cactus.lua#L104-L111)、[采摘](../../../raw/current/scripts/prefabs/cactus.lua#L18-L43) |

- **香蕉丛**：`bananabush` 的 `pickable:SetUp("cave_banana")` 不带再生时长，再生靠 `growable` 的 4 段（每段 1 个 `TOTAL_DAY_TIME`）；`canbepicked` 初始为 false，只有长到 `tall` 时 `set_big` 里才 `Regen()`，采摘后回到第 1 段重长。轮次为 `BANANABUSH_CYCLES`＝4 加 `math.random(BANANABUSH_CYCLES_VAR)` 的 1～3，即 5～7 次，但 `cycles_left` 只在 `transplanted` 时扣，野生丛不消耗额度；移植会先 `MakeBarren`，施肥经 `OnRegen` 直接把 `growable` 阶段设为 4。野生丛从空到 `tall` 要过三个计时段，春季每段再乘 `SPRING_GROWTH_MODIFIER`＝0.75。[阶段表](../../../raw/current/scripts/prefabs/bananabush.lua#L51-L76)、[接线](../../../raw/current/scripts/prefabs/bananabush.lua#L177-L200)、[采摘](../../../raw/current/scripts/prefabs/bananabush.lua#L99-L114)、[常量](../../../raw/current/scripts/tuning.lua#L6741-L6742)
- **猴岛布局与传送门**：`monkeyisland_prefabs_area` 把 1 丛 `bananabush` 加入候选表（候选保证，不等于每次实际放置），非自然传送门的战利品表另有 `dug_bananabush`（权重由 `MONKEYISLAND_PORTAL_BANANABUSHWEIGHT` 决定，默认 5.0）。[布局](../../../raw/current/scripts/map/layouts.lua#L72-L94)、[战利品表](../../../raw/current/scripts/prefabs/monkeyisland_portal.lua#L35-L46)、[权重](../../../raw/current/scripts/tuning.lua#L6691)
- **苔藓藤条**：`oceanvine`（苔藓藤条）挂在海树柱 `oceantree_pillar` 周围；柱边不足 1 条藤时补生成 1～2 条并播落下动画（补藤由撞击后的调度触发，新藤先设为空株），藤被采摘后切到空株动画并移除 `inspectable`。掉落逻辑在烧毁／落地路径另有分支（本页不展开）。[补藤](../../../raw/current/scripts/prefabs/oceantree_pillar.lua#L248-L262)、[生成](../../../raw/current/scripts/prefabs/oceantree_pillar.lua#L234-L245)、[表现](../../../raw/current/scripts/prefabs/oceanvine.lua#L205-L237)
- **仙人掌与开花**：`has_flower` 只在夏季的 `onregenfn` 与 `OnEntityWake` 中置真，采摘时满足门禁才结算 `CACTUS_DAMAGE`（需 `combat`，排除刺抗装备与 `shadowminion`）；`cactus_flower` 不受该门禁影响——有采摘者且植株带花时追加 1 朵交给采摘者（无背包时落地）；`oasis_cactus` 共用同一工厂。[开花](../../../raw/current/scripts/prefabs/cactus.lua#L45-L56)、[唤醒](../../../raw/current/scripts/prefabs/cactus.lua#L64-L69)、[采摘](../../../raw/current/scripts/prefabs/cactus.lua#L18-L43)、[受伤门槛](../../../raw/current/scripts/prefabs/cactus.lua#L24-L27)
- **再生**：`cactus`／`oasis_cactus`／`cave_banana_tree` 都注册在世界再生管理器里（基础 20 天与 5 天，仙人掌另有夏季 ×2、冬季 ×0、下雨 ×1.2 倍率），被清掉后由该管理器补种；触发与落点细节见 [[reference/components/regrowthmanager#原版注册与其季节倍率]]。[注册](../../../raw/current/scripts/components/regrowthmanager.lua#L182-L195)

### 香蕉的 monkeyqueenbribe 标签

- 写入：`VEGGIES.cave_banana.extra_tags_fresh` 与 `.extra_tags_cooked` 各给一个 `{"monkeyqueenbribe"}`；生件在 `SetPristine` 前加、熟件同样在 `SetPristine` 前加，因此双端都可见。[赋值](../../../raw/current/scripts/prefabs/veggies.lua#L157-L158)、[生件](../../../raw/current/scripts/prefabs/veggies.lua#L572-L576)、[熟件](../../../raw/current/scripts/prefabs/veggies.lua#L721-L725)
- 消费端一：月亮码头女王（`monkeyqueen`）的 `accept_trade_test` 只认带该 tag 的物件，作为 `trader:SetAcceptTest` 注册；被收下时 `on_accept_item` 播 `getitem` 状态并直接 `item:Remove()`。[判定](../../../raw/current/scripts/prefabs/monkeyqueen.lua#L45-L47)、[接线](../../../raw/current/scripts/prefabs/monkeyqueen.lua#L172-L176)、[收下](../../../raw/current/scripts/prefabs/monkeyqueen.lua#L19-L22)
- 消费端二：火药猴（`powder_monkey`）的行为在背包里 `FindItem(item_is_monkeyqueen_bribe)` 找带该 tag 的物件，找到就向女王发 `ACTIONS.GIVE`。[判定](../../../raw/current/scripts/brains/powdermonkeybrain.lua#L467-L469)、[递送](../../../raw/current/scripts/brains/powdermonkeybrain.lua#L471-L489)
- 边界：这是「可进贡物」标记，不改变食物分类，也不等于 `tradable` 的收买条件；三键都没有 `goldvalue`，猪王仍拒收（见[[reference/prefabs/farm-crops-staple#常见陷阱与限制]]）。全快照只有这两个注册带该 tag。

### 无花果的 BERRY 次级食性与拟饵

- 次级食性：`secondary_foodtype = FOODTYPE.BERRY` 由 `VEGGIES.fig` 第 13 位传入，生熟都写，于是实体同时带 `edible_VEGGIE` 与 `edible_BERRY`；判定与消费者与浆果一致，见 [[reference/prefabs/wild-berries#berry-次级食性]]。[写入](../../../raw/current/scripts/prefabs/veggies.lua#L600)、[标签常量](../../../raw/current/scripts/constants.lua#L2036)
- 拟饵字段：`{lure_data = TUNING.OCEANFISHING_LURE.BERRY, single_use = true, build = "oceanfishing_lure_mis", symbol = "hook_fig"}`，即外观换成 `hook_fig`，效果参数仍用 `BERRY` 档。[表项](../../../raw/current/scripts/prefabs/veggies.lua#L103-L109)
- 档位差异：`OCEANFISHING_LURE.BERRY`＝`{charm = 0.3, radius = 3.0, dist_max = 1}`，而表内另有 `OCEANFISHING_LURE.FIG`＝`{charm = 0.5, radius = 4.0, dist_max = 1}`；本快照 `scripts/` 内检索不到对 `FIG` 档的引用，无花果并未使用这一档。[两档](../../../raw/current/scripts/tuning.lua#L298-L300)
- 使用与消耗：鱼饵只能进海钓杆第 2 格，中鱼后按 `single_use` 删除；组件侧语义见 [[reference/components/oceanfishingtackle#关键接口]]。[槽位](../../../raw/current/scripts/containers.lua#L1342-L1346)、[消耗](../../../raw/current/scripts/prefabs/oceanfishingrod.lua#L118-L136)

### 仙人掌肉的生熟正负

- 数值符号：生件写 `-TUNING.HEALING_SMALL`（−3）与 `-TUNING.SANITY_TINY`（−5），熟件改写 `TUNING.HEALING_TINY`（+1）与 `TUNING.SANITY_MED`（+15）；两者保质同为 `PERISH_MED`＝10 天，熟制既不延长也不缩短。[表项](../../../raw/current/scripts/prefabs/veggies.lua#L111-L112)、[常量](../../../raw/current/scripts/tuning.lua#L2021-L2035)、[PERISH_MED](../../../raw/current/scripts/tuning.lua#L2048)
- `badfood` 的产生：`edible` 的 `healthvalue`／`sanityvalue` 属性写入器在任一为负时加 tag、否则移除，所以生件带 tag、熟件自动清除。[写入器](../../../raw/current/scripts/components/edible.lua#L1-L7)
- 实际作用面：`badfood` 只参与非 PvP 的喂玩家路径（`FEEDPLAYER` 候选与喂食安全判定）。牛的状态机不看这个标签，自己读食物的 `hungervalue`／`healthvalue` 是否为负。`FEED`（喂生物）与 `FEEDPLAYER`（喂玩家）的候选判断也是两条，玩家自己吃不受限制——`ACTIONS.EAT.fn` 只转调 `eater:Eat`。[喂食安全](../../../raw/current/scripts/actions.lua#L2397-L2411)、[喂食候选](../../../raw/current/scripts/componentactions.lua#L1184-L1197)、[自己吃](../../../raw/current/scripts/actions.lua#L799-L810)、[牛](../../../raw/current/scripts/stategraphs/SGBeefalo.lua#L503-L525)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：edible | 写三维、主食性 `VEGGIE`，无花果另写 `secondary_foodtype` | 生熟各一套；主模拟 | [[reference/components/edible#关键接口]]；[生](../../../raw/current/scripts/prefabs/veggies.lua#L595-L600)、[熟](../../../raw/current/scripts/prefabs/veggies.lua#L745-L750) |
| 组件：perishable | `SetPerishTime`＋`StartPerishing`，腐坏替换 `spoiled_food` | 生用 `perishtime`、熟用 `cooked_perishtime` | [[reference/components/perishable#关键接口]]；[生](../../../raw/current/scripts/prefabs/veggies.lua#L602-L605)、[熟](../../../raw/current/scripts/prefabs/veggies.lua#L740-L743) |
| 组件：stackable | 六键都写 40 | 六键同档 | [[reference/components/stackable#关键接口]]；[生](../../../raw/current/scripts/prefabs/veggies.lua#L613-L619)、[熟](../../../raw/current/scripts/prefabs/veggies.lua#L752-L753) |
| 组件：cookable | 仅生件：`product = name.."_cooked"` | 生件、主模拟 | [[reference/components/cookable#关键接口]]；[接线](../../../raw/current/scripts/prefabs/veggies.lua#L664-L665) |
| 组件：bait | 无参数挂载，只提供陷阱诱饵回调 | 6 件都有 | [[reference/components/bait#关键接口]]；[生](../../../raw/current/scripts/prefabs/veggies.lua#L657)、[熟](../../../raw/current/scripts/prefabs/veggies.lua#L765) |
| 组件：oceanfishingtackle | 仅 `fig` 生件：`SetupLure` 写 `hook_fig` 外观 | 生件 | [[reference/components/oceanfishingtackle#关键接口]]；[装配](../../../raw/current/scripts/prefabs/veggies.lua#L667-L670) |
| 组件：trader | 月亮码头女王的 `trader` 以 `monkeyqueenbribe` 为接受条件，收下即移除物件 | 仅香蕉两态触发 | [[reference/components/trader#关键接口]]；[接线](../../../raw/current/scripts/prefabs/monkeyqueen.lua#L172-L176) |
| API／数据：烹饪食材 | `cave_banana` 是 `fruit=1`；`fig` 是 `fruit=0.5`；`cactus_meat` 是 `veggie=1`；`cancook` 让熟件继承 | 生熟同表 | [香蕉](../../../raw/current/scripts/cooking.lua#L86-L87)、[无花果](../../../raw/current/scripts/cooking.lua#L94)、[仙人掌肉](../../../raw/current/scripts/cooking.lua#L139) |
| 关联 Prefab：`cave_banana_tree` | 树形态来源：采摘 1 个；砍倒时若仍可采再掉 1 个，随后变树桩 | 洞穴房间与世界生成表 | [采摘](../../../raw/current/scripts/prefabs/cave_banana_tree.lua#L146-L158)、[导出](../../../raw/current/scripts/prefabs/cave_banana_tree.lua#L313-L315)、[资源展开](../../../raw/current/scripts/map/forest_map.lua#L219) |
| 关联 Prefab：`bananabush` | 丛形态来源：`growable` 4 段后采摘 | 猴岛布局与移植 | [接线](../../../raw/current/scripts/prefabs/bananabush.lua#L177-L200) |
| 关联 Prefab：`oceanvine` | 唯一可采到 `fig` 的实体 | 海树柱周围 | [接线](../../../raw/current/scripts/prefabs/oceanvine.lua#L305-L314)、[导出](../../../raw/current/scripts/prefabs/oceanvine.lua#L334-L335) |
| 关联 Prefab：`cactus`／`oasis_cactus` | 仙人掌肉来源；有采摘者且植株当前带花时追加 `cactus_flower`（[[reference/prefabs/petals]]） | 地面世界资源表 | [工厂](../../../raw/current/scripts/prefabs/cactus.lua#L74-L127)、[导出](../../../raw/current/scripts/prefabs/cactus.lua#L177-L179)、[资源展开](../../../raw/current/scripts/map/forest_map.lua#L193) |

## 典型接入

### 让生仙人掌肉不再带 badfood

- 前提与选择：服务端 `modmain.lua`；改的是实例上的 `edible` 字段。工厂在构造时才把 `VEGGIES` 表项抄进组件、postinit 又在抄值之后运行，所以在这里赋值能盖住抄来的值；熟件读的是另一组 `cooked_*` 键，不受影响。
- 生效与撤销：赋值会重跑 `oncheckbadfood`，负值消失即移除 `badfood`，只放开这道非 PvP 喂食限制（`spoiled`／`unsafefood`／食性等其它门禁仍适用）；停用模组即恢复工厂原值。
- 原版依据：负值来源与消费端见[仙人掌肉的生熟正负](#仙人掌肉的生熟正负)。

```lua
-- modmain.lua：把生仙人掌肉的两项负值归零，熟件的正收益不动
AddPrefabPostInit("cactus_meat", function(inst)
    if not GLOBAL.TheWorld.ismastersim or inst.components.edible == nil then
        return
    end
    inst.components.edible.healthvalue = 0
    inst.components.edible.sanityvalue = 0
end)
```

- 调整与验证：单位是生命与理智点；`badfood` 只看这两个字段的符号，归零只解除这一道喂食限制，`spoiled`（腐坏）、`unsafefood` 与食性仍会拦截。改 `IsFoodSafeToEat` 之类的判定会同时影响所有食物，影响面远大于只改这一件。

## 常见陷阱与限制

- **误区：`cave_banana` 的简中名是「洞穴香蕉」**：`NAMES.CAVE_BANANA` 的实际译文只有「香蕉」，注册名里的 `cave_` 只是区分洞穴来源；熟件为「熟香蕉」。[名称](../../../raw/current/scripts/languages/chinese_s.po#L340715-L340717)
- **误区：生仙人掌肉的负值会挡住玩家自己吃**：`ACTIONS.EAT.fn` 不做 `badfood` 检查，吃下去照样扣血扣理智。非 PvP 喂玩家才会检查这项标签，牛另读食物数值。见[仙人掌肉的生熟正负](#仙人掌肉的生熟正负)。
- **边界：熟制不一定更耐放**：`cactus_meat` 生熟同为 10 天，`cave_banana` 由 10 天降到 6 天，`fig` 由 6 天降到 3 天；按「烤过更耐放」配货会算错。[表项](../../../raw/current/scripts/prefabs/veggies.lua#L41-L43)
- **边界：香蕉丛不是计时再生**：`pickable` 侧没有再生时长，靠 `growable` 逐段生长；工厂把 `canbepicked` 显式置 false，只有长到 `tall` 时 `set_big` 里的 `Regen()` 才放开采摘。[接线](../../../raw/current/scripts/prefabs/bananabush.lua#L177-L200)
- **边界：无花果只有一个来源且不会耗尽**：`oceanvine` 用 `max_cycles = nil`＋`cycles_left = 1` 的写法，采摘后按 `OCEANVINE_REGROW_TIME` 重新结果；它没有 `dug_oceanvine` 移植形态（`scripts/` 内检索不到该注册），`cycles_left` 也不因采摘递减，因此不会枯死。[接线](../../../raw/current/scripts/prefabs/oceanvine.lua#L305-L314)
- **未核验**：非自然传送门的出现条件与战利品权重分布、`bananabush` 的火烧／枯萎分支；生成点检索只覆盖上述植株与布局、战利品表，未穷举动态选名与模组注入。

## 验证记录

- 源码审阅：2026-11-08，Steam buildid 26335394。核对 `veggies.lua` 全文（`MakeVegStats` 位序、三键表项、`extra_tags` 赋值、`fn`／`fn_cooked` 的标签与浮水分支、导出与注册段、`SEEDLESS`）、`cave_banana_tree.lua` 的采摘与砍倒、`bananabush.lua` 的阶段表与采摘、`oceanvine.lua` 的采摘与补藤、`oceantree_pillar.lua` 的补藤入口、`cactus.lua` 的采摘／开花／花注册、`monkeyqueen.lua` 与 `powdermonkeybrain.lua` 的进贡链、`monkeyisland_portal.lua` 与 `map/layouts.lua` 的来源、`regrowthmanager.lua` 的注册与倍率、`pickable`／`edible`／`eater`／`perishable`／`stackable`／`cookable`／`bait`／`oceanfishingtackle` 的相关分支、`actions` 的 `EAT` 与 `IsFoodSafeToEat`、`componentactions` 的喂食候选、`SGBeefalo` 的 `badfood` 状态、`cooking` 的食材表、`tuning` 的基准与相关常量、`constants` 的 `FOODTYPE`，以及简中 6 个名称键。
  名称核验：`NAMES.CAVE_BANANA`＝香蕉、`CAVE_BANANA_COOKED`＝熟香蕉、`FIG`＝无花果、`FIG_COOKED`＝熟无花果、`CACTUS_MEAT`＝仙人掌肉、`CACTUS_MEAT_COOKED`＝熟仙人掌肉。[CAVE_BANANA](../../../raw/current/scripts/languages/chinese_s.po#L340715-L340717)、[CAVE_BANANA_COOKED](../../../raw/current/scripts/languages/chinese_s.po#L340720-L340722)、[FIG](../../../raw/current/scripts/languages/chinese_s.po#L342750-L342752)、[FIG_COOKED](../../../raw/current/scripts/languages/chinese_s.po#L342765-L342767)、[CACTUS_MEAT](../../../raw/current/scripts/languages/chinese_s.po#L340065-L340067)、[CACTUS_MEAT_COOKED](../../../raw/current/scripts/languages/chinese_s.po#L340070-L340072)
  相关实体同批核对：`bananabush`＝香蕉丛、`cave_banana_tree`＝洞穴香蕉树、`oceanvine`＝苔藓藤条、`monkeyqueen`＝月亮码头女王、`powder_monkey`＝火药猴、`monkeyisland_portal`＝非自然传送门。[BANANABUSH](../../../raw/current/scripts/languages/chinese_s.po#L339020-L339022)、[CAVE_BANANA_TREE](../../../raw/current/scripts/languages/chinese_s.po#L340725-L340727)、[OCEANVINE](../../../raw/current/scripts/languages/chinese_s.po#L346580-L346582)、[MONKEYQUEEN](../../../raw/current/scripts/languages/chinese_s.po#L345565-L345567)、[POWDER_MONKEY](../../../raw/current/scripts/languages/chinese_s.po#L347275-L347277)、[MONKEYISLAND_PORTAL](../../../raw/current/scripts/languages/chinese_s.po#L345550-L345552)
  未覆盖：`cactus_flower` 与其花田表现、非自然传送门与月亮码头女王的交易产物、`bananabush` 的移植与火烧分支、`oceanvine` 的烧毁／落地掉落分支。
- 游戏运行：未运行。三维、天数与秒数均为 tuning 推导（`CALORIES_MEDSMALL`＝75/4＝18.75、`CACTUS_DAMAGE`＝150×0.04），进贡与采摘为代码路径推导。
- 结构检查：`validate_page.py` 通过（standard 档内）；`validate_source_links.py --strict` 0 错 0 警。

## 相关链接

- 同一工厂的其它类：[[reference/prefabs/farm-crops-staple]]（共用工厂与生熟两态）、[[reference/prefabs/wild-berries]]（BERRY 次级食性与拟饵）、[[reference/prefabs/farm-crops-fruit]]、[[reference/prefabs/farm-crops-spice]]
- 采集与食用：[[reference/components/pickable]]、[[reference/components/edible]]、[[reference/components/trader]]
- 世界再生：[[reference/components/regrowthmanager]]

返回 [[reference/prefabs/index|Prefab 索引]]。
- 2026-11-08 build 26474873：本页随版本迁移升版（锚点迁移 2 处），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-12）。
