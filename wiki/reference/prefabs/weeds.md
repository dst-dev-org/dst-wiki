---
title: "杂草产物三注册（必忘我 forgetmelots、犁地草 tillweed 与火荨麻叶 firenettles）"
created: "2026-11-08"
updated: "2026-11-08"
type: prefab
summary: "三种农田杂草的产物与背后植株：可食三维、毒叶 debuff、鲜度、堆叠、燃料与干制差异，杂草的随机株抽取、季节播种与蔓延、挖除再生和焚烧链条，以及 firenettle_toxin 的温度修正（定时只播台词）。"
aliases: ["forgetmelots", "tillweed", "firenettles", "必忘我", "犁地草", "火荨麻叶", "杂草", "weed_forgetmelots", "weed_tillweed", "weed_firenettle", "firenettle_toxin"]
tags: [prefab, item, food]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/forgetmelots.lua", "prefabs/tillweed.lua", "prefabs/firenettles.lua", "prefabs/weed_defs.lua", "prefabs/weed_plants.lua", "prefabs/farm_plants.lua", "prefabs/seeds.lua", "prefabs/driedplants_defs.lua", "components/farming_manager.lua", "components/edible.lua", "components/fueled.lua", "components/perishable.lua", "components/stackable.lua", "components/pickable.lua", "components/workable.lua", "components/dryable.lua", "components/dryingrack.lua", "components/deployable.lua", "components/growable.lua", "components/lootdropper.lua", "components/playerprox.lua", "components/temperature.lua", "components/debuff.lua", "components/debuffable.lua", "entityscript.lua", "standardcomponents.lua", "tuning.lua", "constants.lua", "cooking.lua", "preparedfoods.lua", "recipes.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 杂草产物三注册（必忘我 forgetmelots、犁地草 tillweed 与火荨麻叶 firenettles）

## 快速判断

- 职责与适用：三件都是农田杂草的**产物**（可食物品），与各自植株 `weed_forgetmelots`／`weed_tillweed`／`weed_firenettle` 成对：植株是 `WEED_DEFS` 数据表的一项，由同一个 `MakeWeed` 工厂构造，长到 `full`（必忘我随后会进 `bolting`，那一步 `MakePickable(inst, false)` 撤掉可摘）才可摘，摘出来的就是本页物品。可借鉴：数据表 ＋ 单工厂的族写法，产物只由 `product` 字段一处决定。[产物字段](../../../raw/current/scripts/prefabs/weed_defs.lua#L48-L51)、[工厂](../../../raw/current/scripts/prefabs/weed_plants.lua#L459-L600)
- 优先使用：数值与鲜度看[数值](#数值)；毒素链看[火荨麻叶的毒素链](#火荨麻叶的毒素链)；杂草怎么冒出来看[生成与增殖](#生成与增殖)。随机株的抽取概率已在 [[reference/prefabs/generic-seeds#随机种植与随机株]] 写过，本页只引结论。
- 重要副作用：三件都是 `FOODTYPE.VEGGIE`；`firenettles` 生命 −3／理智 −5，非 `plantkin` 吃下会尝试施加 `firenettle_toxin`（`AddDebuff` 自身还要过 `DebuffsEnabled()`、目标未死且非幽灵）；`forgetmelots` 是唯一登记了食材值的生品（`decoration = 1`），`tillweed`／`firenettles` 生品不能下锅。[吃下接线](../../../raw/current/scripts/prefabs/firenettles.lua#L13-L17)、[食材值](../../../raw/current/scripts/cooking.lua#L159-L171)
- 修改边界：不含 `weed_ivy`（刺针旋花）；不含晾肉架内部流程；不含作物与巨型作物页；`tillweedsalve`／隐士蟹茶等成品只给出处。

## 注册与构造

三件物品各自一个文件、各自一个 `Prefab(...)`；对应的植株另由 `weed_plants.lua` 的注册循环逐个 `MakeWeed` 产出。三个非物品注册分散在同批物品文件里：`firenettles.lua` 的 `firenettle_toxin`，以及 `forgetmelots.lua` 的 `sweettea_buff` 与 `weed_forgetmelots_respawner`。

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `forgetmelots` | `fn`：bank／build 都 `forgetmelots`、`PlayAnimation("idle")`、浮水 `("med", 0.0, 0.7)`、pristine 前加 `cattoy`／`vasedecoration` → 主模拟挂 `stackable`／`tradable`／`inspectable`／`inventoryitem`／`vasedecoration`／`edible`／`perishable`／`dryable`／`fuel`，再 `MakeSmallBurnable`＋`MakeSmallPropagator`＋`MakeHauntableLaunchAndPerish` | 三件里唯一带 `vasedecoration` 的一件；同文件另导出甜茶 buff 与「必忘我再生体」两个服务端实体。[物品工厂](../../../raw/current/scripts/prefabs/forgetmelots.lua#L13-L71)、[三个导出](../../../raw/current/scripts/prefabs/forgetmelots.lua#L194-L196) |
| `tillweed` | `fn`：bank／build 都 `tillweed`、`PlayAnimation("idle")`、浮水 `("med", 0.05, 0.68)`、pristine 前只加 `cattoy` → 主模拟挂 `inventoryitem`／`stackable`／`edible`／`inspectable`／`tradable`／`fuel`／`perishable`／`dryable`，燃耗与传播后接 `MakeHauntableLaunchAndIgnite` | 作祟走 `MakeHauntableLaunchAndIgnite`（点燃分支被注释，实际只抛飞且返回假）；没有 `vasedecoration`。[工厂](../../../raw/current/scripts/prefabs/tillweed.lua#L13-L71) |
| `firenettles` | `fn`：bank／build 都 `firenettles`、`PlayAnimation("idle")`、浮水 `("med", 0.0, 0.7)`、无 tag 附加 → 主模拟挂 `stackable`／`tradable`／`inspectable`／`inventoryitem`／`edible`（`SetOnEatenFn(oneaten)`）／`perishable`／`fuel`／`dryable`，燃耗后接 `MakeHauntableLaunchAndPerish` | 唯一带「吃下附加 debuff」的一件；同文件导出 `firenettle_toxin`。[工厂](../../../raw/current/scripts/prefabs/firenettles.lua#L19-L77)、[毒素导出](../../../raw/current/scripts/prefabs/firenettles.lua#L174-L175) |

### 数值

1 天 = `TUNING.TOTAL_DAY_TIME` = 16 × 30 = 480 秒。[基准](../../../raw/current/scripts/tuning.lua#L19-L20)

| 注册 | 饥饿 | 生命 | 理智 | 食性 | 鲜度 | 堆叠 | 燃料值 | 燃烧 | 浮水 | 干制产物 |
|---|---|---|---|---|---|---|---|---|---|---|
| `forgetmelots` | 0 | 0 | **+1**（`SANITY_SUPERTINY`） | `VEGGIE` | 3 天（`PERISH_SUPERFAST`） | 40 | 7.5 秒（`TINY_FUEL`） | 10 秒（helper 默认） | med | `forgetmelots_dried` |
| `tillweed` | 0 | **+1**（`HEALING_TINY`） | 0 | `VEGGIE` | 6 天（`PERISH_FAST`） | 40 | 7.5 秒 | 10 秒 | med | `tillweed_dried` |
| `firenettles` | 0 | **−3**（`-HEALING_SMALL`） | **−5**（`-SANITY_TINY`） | `VEGGIE` | 3 天 | 40 | 15 秒（`SMALL_FUEL`） | 10 秒 | med | `firenettles_dried` |

- 赋值：`forgetmelots` 只写 `sanityvalue`，生命／饥饿留组件默认 0；`tillweed` 只写 `healthvalue`；`firenettles` 三项全写并挂 `oneaten`。[必忘我](../../../raw/current/scripts/prefabs/forgetmelots.lua#L45-L49)、[犁地草](../../../raw/current/scripts/prefabs/tillweed.lua#L43-L46)、[火荨麻叶](../../../raw/current/scripts/prefabs/firenettles.lua#L50-L55)
- `firenettles` 的两个负值让 `edible` 的属性 setter 打上 `badfood`；`forgetmelots`／`tillweed` 不带。[判定](../../../raw/current/scripts/components/edible.lua#L1-L7)
- 常量：`SANITY_SUPERTINY`=1、`HEALING_TINY`=1、`HEALING_SMALL`=3、`SANITY_TINY`=5、`PERISH_SUPERFAST`=3 天、`PERISH_FAST`=6 天、`DRY_FAST`=1 天、`STACK_SIZE_SMALLITEM`=40、`TINY_FUEL`=7.5、`SMALL_FUEL`=15。[生命与理智](../../../raw/current/scripts/tuning.lua#L2021-L2033)、[鲜度与干制](../../../raw/current/scripts/tuning.lua#L2045-L2055)、[燃料](../../../raw/current/scripts/tuning.lua#L1673-L1674)、[堆叠](../../../raw/current/scripts/tuning.lua#L83)
- 三件的 `MakeSmallBurnable`／`MakeSmallPropagator` 都不传时长，落 helper 默认 10 秒（不是 `SMALL_BURNTIME`＝7.5 秒）；`fuel` 是「投火堆的燃料值」，与「在地上被点燃」是两条链。[helper](../../../raw/current/scripts/standardcomponents.lua#L164-L174)、[取料](../../../raw/current/scripts/components/fueled.lua#L198-L214)
- 干品改写收益：`forgetmelots_dried` 理智 +5、`tillweed_dried` 生命 +8、`firenettles_dried` 仍是 −3／−5 且**保留**吃下挂毒素的回调。[干品表](../../../raw/current/scripts/prefabs/driedplants_defs.lua#L24-L47)
- 烹饪口径：`forgetmelots` 登记 `decoration = 1` 可下锅（甜茶判定读 `names.forgetmelots` 或 `names.forgetmelots_dried`）；`tillweed`／`firenettles` 生品无食材值，只有干品进 `driedleaves` 组（`decoration = 1, dried = 1`）。[食材](../../../raw/current/scripts/cooking.lua#L161-L171)、[甜茶](../../../raw/current/scripts/preparedfoods.lua#L879-L902)

### 生成与增殖

| 机制 | 条件与数值 | 证据 |
|---|---|---|
| 随机种子株 | 播种 `farm_plant_randomseed` 后发芽段掷 `FARM_PLANT_RANDOMSEED_WEED_CHANCE`＝0.20 出杂草，否则按作物权重抽；杂草权重来自 `seed_weight`：必忘我 5、犁地草与火荨麻 0.5（同表还有 `weed_ivy` 0.5，不在本页范围） | [[reference/prefabs/generic-seeds#随机种植与随机株]]；[抽取](../../../raw/current/scripts/prefabs/farm_plants.lua#L134-L149)、[权重](../../../raw/current/scripts/prefabs/weed_defs.lua#L43-L51)、[权重表](../../../raw/current/scripts/prefabs/weed_defs.lua#L336-L341) |
| 季节播种 | `farming_manager` 换季时对**已有营养格的农田格**按 `SEASONAL_WEED_SPAWN_CAHNCE`（秋 0.05／春 0.15）掷点，把命中格的时间摊在剩余季节的前 25%，逐 `SEG_TIME` 到点生成；落点优先顶掉同格 `soil` 实体，否则塌陷可耕地 | [换季](../../../raw/current/scripts/components/farming_manager.lua#L274-L293)、[生成](../../../raw/current/scripts/components/farming_manager.lua#L241-L272)、[推进](../../../raw/current/scripts/components/farming_manager.lua#L332-L352)、[常量](../../../raw/current/scripts/tuning.lua#L5816-L5820) |
| 自身蔓延 | 计时只在 `spread.stage` 阶段跑：必忘我 `bolting`、犁地草与火荨麻 `full`；周期 `time_min + random × time_var`（必忘我 3–5 天，另两种 6–8 天）。落点先找 `tilled_dist` 内的 `soil` 实体（用掉它），否则在 `ground_dist`～`ground_dist + ground_dist_var` 内找可耕地；要求 `tooclose_dist` 内无同种 | [配置](../../../raw/current/scripts/prefabs/weed_defs.lua#L38-L41)、[蔓延](../../../raw/current/scripts/prefabs/weed_plants.lua#L283-L334)、[计时](../../../raw/current/scripts/prefabs/weed_plants.lua#L164-L172) |
| 挖除再生（仅必忘我） | 只在 `bolting` 阶段被挖时判定：数 12 半径内 `weed_forgetmelots_respawner` 数量 n，`chance = 1 − (n+1)/4`，再按幸运修正；命中则原位留一个再生体，2–5 天后（`FORGETMELOTS_RESPAWNER_MIN`＋随机×`VAR`）在该点或 2–3 格内可耕点塌陷土壤并生成新的 `weed_forgetmelots` | [判定](../../../raw/current/scripts/prefabs/weed_defs.lua#L83-L99)、[再生体](../../../raw/current/scripts/prefabs/forgetmelots.lua#L77-L127)、[常量](../../../raw/current/scripts/tuning.lua#L5822-L5823) |
| 采摘后重长 | 成株才有 `pickable`（`SetUp(nil)`＋`use_lootdropper_for_product`，产物固定为 `{product}`）；摘完 `onpickedfn` 把 growable 的 stage 拨回 1 并继续生长，**植株不消失** | [可摘装配](../../../raw/current/scripts/prefabs/weed_plants.lua#L133-L145)、[摘后复位](../../../raw/current/scripts/prefabs/weed_plants.lua#L124-L131)、[产物表](../../../raw/current/scripts/prefabs/weed_plants.lua#L104-L109) |
| 挖除与焚烧 | `workable`：`ACTIONS.DIG`、`SetWorkLeft(1)`，完成回调先 `DropLoot()`、再 `ondigup`、最后 `Remove`；燃烧则先生成 `ash`、同样 `DropLoot()` 后移除，起火期间暂停生长、熄灭后恢复 | [挖除](../../../raw/current/scripts/prefabs/weed_plants.lua#L53-L79)、[焚烧](../../../raw/current/scripts/prefabs/weed_plants.lua#L81-L98)、[挂载](../../../raw/current/scripts/prefabs/weed_plants.lua#L571-L582) |
| 作物压力 | 杂草构造时带 `farm_plant_killjoy`，会让半径 6 内作物的「killjoys」压力项成立（容差 `FARM_PLANT_KILLJOY_TOLERANCE`＝0）；植株另有 `weed`／`farm_plant`／`plantresearchable` 等 tag | [压力测试](../../../raw/current/scripts/prefabs/farm_plants.lua#L169-L176)、[常量](../../../raw/current/scripts/tuning.lua#L5808-L5809)、[tag](../../../raw/current/scripts/prefabs/weed_plants.lua#L503-L519) |

- 三件都**没有专属种子注册**：本快照内检索不到 `forgetmelots_seeds`／`tillweed_seeds`／`firenettles_seeds`，`WEED_DEFS` 的 `seed_weight` 只用于加权抽取，不是可播种物品；玩家侧唯一的「种」是通用 `seeds` 经随机株转成杂草，播种与部署门禁见 [[reference/prefabs/generic-seeds]]。[`farmplantable`](../../../raw/current/scripts/prefabs/seeds.lua#L159-L168)

## 火荨麻叶的毒素链

- 入口两条，出口同一个 prefab：吃 `firenettles`（或 `firenettles_dried`）时非 `plantkin` 调 `AddDebuff("firenettle_toxin", "firenettle_toxin")`，实际是否挂上由 `AddDebuff` 的门禁决定（`DebuffsEnabled()` 且目标未死、非幽灵）；走近成株 `weed_firenettle` 时由 `playerprox` 触发，先 `GetAttacked(..., WEED_FIRENETTLE_DAMAGE)`，只有攻击结算返回真才挂同名字 debuff，随后植株把自身 stage 拨回 1 重长。[吃下](../../../raw/current/scripts/prefabs/firenettles.lua#L13-L17)、[植株碰撞](../../../raw/current/scripts/prefabs/weed_defs.lua#L104-L137)、[干品](../../../raw/current/scripts/prefabs/driedplants_defs.lua#L24-L32)
- 植株侧门禁：成株且 `playerprox` 距离 0.5–2.5、`Schedule(5 * FRAMES)`；正在燃烧、植株湿了、目标带 `plantkin`（沃姆伍德等）都打不动。[挂载](../../../raw/current/scripts/prefabs/weed_defs.lua#L127-L137)、[判定](../../../raw/current/scripts/prefabs/weed_defs.lua#L104-L125)、[距离](../../../raw/current/scripts/components/playerprox.lua#L165-L195)
- debuff 本体：`keepondespawn = true`；attach 时把 `firenettle_toxin` 温度修正设为 `FIRE_NETTLE_TOXIN_TEMP_MODIFIER`＝60，并每 10 秒对「活着且 idle」的目标播一次 `ANNOUNCE_FIRENETTLE_TOXIN`；到期任务 `FIRE_NETTLE_TOXIN_DURATION`＝60 秒，命中即 `debuff:Stop()`，detach 时移除温度修正并播「恢复」台词。[接线](../../../raw/current/scripts/prefabs/firenettles.lua#L86-L111)、[计时](../../../raw/current/scripts/prefabs/firenettles.lua#L113-L124)、[常量](../../../raw/current/scripts/tuning.lua#L5825-L5827)
- 重复吃／重复触发不会叠层：同名 debuff 已存在时 `AddDebuff` 走 `Extend`，本件的 `SetExtendedFn` 只取消旧到期任务、按满时长重排。[`AddDebuff`](../../../raw/current/scripts/entityscript.lua#L2140-L2154)、[扩展分支](../../../raw/current/scripts/components/debuffable.lua#L95-L105)、[`Extend`](../../../raw/current/scripts/components/debuff.lua#L53-L57)、[重排](../../../raw/current/scripts/prefabs/firenettles.lua#L119-L124)
- 存档只存剩余秒数：`OnSave` 写 `remaining = GetTaskRemaining(inst.task)`，`OnLoad` 复位到期任务；玩家死亡时 debuff 自停。[存取](../../../raw/current/scripts/prefabs/firenettles.lua#L126-L139)、[死亡监听](../../../raw/current/scripts/prefabs/firenettles.lua#L90-L92)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`edible` | 三维、食性、`SetOnEatenFn` | 主模拟；`oneaten` 在 `OnEaten` 里先跑 | [[reference/components/edible#关键接口]]；[`OnEaten`](../../../raw/current/scripts/components/edible.lua#L183-L186) |
| 组件：`perishable` | 3 天／6 天两档，腐坏换 `spoiled_food` | 主模拟；三件都是 `StartPerishing()` | [[reference/components/perishable#关键接口]]；[接线](../../../raw/current/scripts/prefabs/tillweed.lua#L54-L57) |
| 组件：`stackable` | 显式 40 | 主模拟；组件默认 20 | [[reference/components/stackable#关键接口]]；[赋值](../../../raw/current/scripts/prefabs/firenettles.lua#L43-L44) |
| 组件：`dryable` | 三件都有，产物各自 `*_dried` | 主模拟；客户端只靠 pristine tag | [[reference/components/dryable#关键接口]]；[接线](../../../raw/current/scripts/prefabs/firenettles.lua#L65-L69) |
| 组件：`pickable`（植株侧） | 成株才装，`use_lootdropper_for_product` | 主模拟；摘后不删植株 | [[reference/components/pickable#关键接口]]；[装配](../../../raw/current/scripts/prefabs/weed_plants.lua#L133-L145) |
| 组件：`workable`（植株侧） | `ACTIONS.DIG`、1 次 | 主模拟；完成回调 `dig_up` | [[reference/components/workable#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/weed_plants.lua#L571-L574) |
| 组件：`growable`（植株侧） | 四段阶段表（small／med／full／bolting） | 主模拟；`growoffscreen` | [[reference/components/growable#关键接口]]；[阶段表](../../../raw/current/scripts/prefabs/weed_plants.lua#L180-L279) |
| helper：`MakeSmallBurnable`／`MakeHauntableLaunchAndPerish`／`MakeHauntableLaunchAndIgnite` | 物品与植株的燃耗、作祟差异 | 主模拟；作祟反应随 helper 不同 | [燃耗](../../../raw/current/scripts/standardcomponents.lua#L164-L174)、[抛飞](../../../raw/current/scripts/standardcomponents.lua#L1311-L1337) |
| 关联 Prefab：`firenettle_toxin` | 毒素 debuff 本体，`CLASSIFIED` 隐藏实体 | 仅主模拟；`OnSave`／`OnLoad` 自管 | [工厂](../../../raw/current/scripts/prefabs/firenettles.lua#L141-L171) |
| 关联 Prefab：`sweettea_buff` | 甜茶 buff：每 2 秒 +1 理智、共 60 秒 | 仅主模拟；`keepondespawn` | [工厂](../../../raw/current/scripts/prefabs/forgetmelots.lua#L162-L192) |
| 关联 Prefab：`weed_forgetmelots_respawner` | 挖掉 `bolting` 期必忘我后的再生体 | 仅主模拟；非联网实体 | [工厂](../../../raw/current/scripts/prefabs/forgetmelots.lua#L105-L127) |
| 关联 Prefab：`farm_soil_debris` | 犁地草成株在地上撒的杂物（上限 `WEED_TILLWEED_MAX_DEBRIS`＝5，间隔 0.75–1 天） | 世界实体；`weed_tillweed.OnMakeFullFn` 起停计时 | [接线](../../../raw/current/scripts/prefabs/weed_defs.lua#L142-L170)、[常量](../../../raw/current/scripts/tuning.lua#L5829-L5831) |
| 数据：`WEED_DEFS` | 生成时长、蔓延、权重、产物、耗水与耗肥的唯一来源 | 加载即构造；模组可改表 | [表](../../../raw/current/scripts/prefabs/weed_defs.lua#L7-L68) |
| 数据：`farming_manager` | 季节播种与营养格 | 世界组件；仅主模拟 | [换季](../../../raw/current/scripts/components/farming_manager.lua#L274-L293) |

## 典型接入

### 让未成熟杂草也能挖出产物

- 前提与选择：服务端 `modmain.lua`。杂草的挖除与采摘都走 `lootdropper`，而 `lootsetupfn` 每次 `GenerateLoot` 都会重跑，所以直接换掉 setup 函数即可覆盖「挖」与「摘」两条出口，不用改工厂。[时机](../../../raw/current/scripts/components/lootdropper.lua#L186-L191)
- 生效与撤销：postinit 在每次构造时运行，只影响新生成的植株；停用模组恢复原表。原版 `SetupLoot` 只在有 `pickable` 时给产物，成株前会留下空表。[原版](../../../raw/current/scripts/prefabs/weed_plants.lua#L104-L109)
- 原版依据：产物字段与摘后复位见[生成与增殖](#生成与增殖)。

```lua
-- modmain.lua：犁地草无论长到哪一段，被挖或被摘都给 1 个犁地草
AddPrefabPostInit("weed_tillweed", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    local lootdropper = inst.components.lootdropper
    if lootdropper ~= nil then
        lootdropper:SetLootSetupFn(function(dropper)
            dropper:SetLoot({ "tillweed" })
        end)
    end
end)
```

- 调整与验证：单位为件；替换的是整张产物表，原版「成株只给 1 个」也要在这里显式写回。未跑游戏验证。

## 常见陷阱与限制

- **边界：摘杂草不会毁掉杂草**：成株被摘后 stage 拨回 1 继续长，只有挖／烧／再生体替代才会消失；按「一次性资源」配复采周期会算错。[复位](../../../raw/current/scripts/prefabs/weed_plants.lua#L124-L131)
- **边界：默认燃烧 10 秒**：三件与植株都不传时长，落 `MakeSmallBurnable` 的默认 10 秒；把 `SMALL_BURNTIME` 当默认会算错。[helper](../../../raw/current/scripts/standardcomponents.lua#L164-L174)
- **误区：`MakeHauntableLaunchAndPerish` 会扣新鲜度**：该 helper 的 perish 分支整段被注释，实际只做抛飞；不得引用它解释腐坏加速。[实现](../../../raw/current/scripts/standardcomponents.lua#L1324-L1332)
- **误区：`MakeHauntableLaunchAndIgnite` 会点火**：该 helper 的点燃分支整段被注释，且末尾无条件 `return false`，所以犁地草的作祟只抛飞、不上报成功。[实现](../../../raw/current/scripts/standardcomponents.lua#L1191-L1217)
- **边界：毒素免疫面不止「植物人」**：植株判定还排除燃烧中与湿透的自身；物品侧只排除 `plantkin`，非植物人都会被尝试施加，但目标已死／是幽灵或 debuff 被禁用时不生效。毒素本体只改体温（`FIRE_NETTLE_TOXIN_TEMP_MODIFIER`）并每 10 秒播一次台词，伤害来自植株接触时的单次结算，不是 debuff 持续掉血。[植株判定](../../../raw/current/scripts/prefabs/weed_defs.lua#L104-L106)
- **边界：再生体只看 `bolting`**：必忘我在 `full`（还能摘）被挖不会留再生体，`bolting` 才判定；同区再生体越多概率越低（`1 − (n+1)/4`，半径内已有 3 个即 0）。[判定](../../../raw/current/scripts/prefabs/weed_defs.lua#L89-L99)
- **未核验**：`WEED_DEFS.plantregistryinfo`／图鉴页、`beeswax_spray` 与 `waxed_plants` 对杂草的打蜡分支、`weed_firenettle` 的 `trapdamage` tag 消费端未逐条展开。

## 验证记录

- 源码审阅：2026-11-08，Steam buildid 26335394。核对 `forgetmelots.lua`／`tillweed.lua`／`firenettles.lua` 全文（含三个附带注册）、`weed_defs.lua` 的数据表与自定义行为、`weed_plants.lua` 的阶段表／蔓延／挖除／焚烧／产物装配、`farm_plants.lua` 的随机株抽取与 killjoy 压力、`farming_manager` 的季节播种、`seeds.lua` 的播种与部署链、`edible`／`perishable`／`stackable`／`pickable`／`workable`／`dryable`／`growable`／`lootdropper`／`playerprox`／`temperature`／`debuff`／`debuffable`／`entityscript` 的相关分支、`standardcomponents` 三个 helper、`tuning` 常量。
  名称核验：`NAMES.FORGETMELOTS`＝必忘我、`TILLWEED`＝犁地草、`FIRENETTLES`＝火荨麻叶；植株键 `WEED_FORGETMELOTS`＝必忘我、`WEED_TILLWEED`＝犁地草、`WEED_FIRENETTLE`＝火荨麻（植株世界内另有 `inspectable.nameoverride = "FARM_PLANT"`＋`displaynamefn`，未识别时显示 `FARM_PLANT_UNKNOWN`）。[FORGETMELOTS](../../../raw/current/scripts/languages/chinese_s.po#L342955-L342957)、[TILLWEED](../../../raw/current/scripts/languages/chinese_s.po#L350405-L350407)、[FIRENETTLES](../../../raw/current/scripts/languages/chinese_s.po#L342795-L342797)、[WEED_*](../../../raw/current/scripts/languages/chinese_s.po#L351935-L351952)、[植株名](../../../raw/current/scripts/prefabs/weed_plants.lua#L412-L425)、[nameoverride](../../../raw/current/scripts/prefabs/weed_plants.lua#L541-L543)
  未覆盖：`weed_ivy` 全文、图鉴与打蜡分支、`beeswax_spray` 的阻止效果、种子袋／隐士蟹茶等成品的内部机制；生成点检索只覆盖上表三条链与掉落表，未穷举动态选名与模组注入。
- 游戏运行：未运行。三维、天数与秒数均由 `tuning` 常量推导，非运行测量。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 通过；页内 Lua 片段经 `parser_probe.exe --syntax-only` 通过。
- 复核：2026-11-08，Steam buildid 26474873。核对本页引用源在 26335394→26474873 间的差异（`tuning.lua`、`plantables.lua` 各插入 1 行，`SGwilson.lua` 仅换声效），锚点按新树重定位、结论未变；上一条 26335394 审阅记录保留作历史。

## 相关链接

- 随机株与通用种子：[[reference/prefabs/generic-seeds]]
- 杂草会压的作物压力：[[reference/components/farmplantstress]]、[[reference/components/farming-manager]]
- 花瓣族（同样可干制、可燃烧的小件）：[[reference/prefabs/petals]]
- 干制消费端：[[reference/components/dryable]]、[[reference/components/dryingrack]]

返回 [[reference/prefabs/index|Prefab 索引]]。
