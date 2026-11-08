---
title: "火腺与熟火腺（mitegland、mitegland_cooked）"
created: "2026-11-09"
updated: "2026-11-09"
type: prefab
summary: "洞穴螨虫掉落的火腺与熟火腺：三维、鲜度不随熟制延长、cattoy／rawmeat 标签与食性差异、烤制与烹饪食材登记；以及吃生火腺给非生肉食者挂的 mitegland_debuff——温度修正量 +60（加到体温目标值上）、长 60 秒、首次 5 秒后每 10 秒在 idle 时播一句台词、死亡即停、重吃只刷新时长。"
aliases: ["mitegland", "mitegland_cooked", "火腺", "熟火腺", "Heat Gland"]
tags: [prefab, item, food]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/meats.lua", "prefablist.lua", "components/edible.lua", "components/eater.lua", "components/stackable.lua", "components/debuff.lua", "components/debuffable.lua", "components/temperature.lua", "components/timer.lua", "components/talker.lua", "entityscript.lua", "prefabs/cave_vent_mite.lua", "prefabs/cave_vent_mite_spawner.lua", "components/lootdropper.lua", "prefabs/firenettles.lua", "cooking.lua", "recipes.lua", "prefabs/catcoon.lua", "prefabs/webber.lua", "brains/catcoonbrain.lua", "speech_wilson.lua", "tuning.lua", "languages/chinese_s.po", "screens/redux/scrapbookdata.lua", "scrapbook_prefabs.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 火腺与熟火腺（mitegland、mitegland_cooked）

## 快速判断

- 职责与适用：两件都是可拾取、可食用的肉类物品，由 `common()` 工厂（`prefabs/meats.lua`）构造，与鱼肉类同一套组装；差别在于**生品挂了一个吃下回调**，会给「不会生吃肉的食用者」加一个只动温度修正量的 debuff。同文件表尾还导出一个非物品实体 `mitegland_debuff`（debuff 本体），本页只作关联实体说明，不单独建页、也不把它当本页主体的别名。
- 优先使用：三维／鲜度看[两件对照](#两件对照)；吃下会发生什么、怎么结束看[生火腺的灼烧 debuff](#生火腺的灼烧-debuff)；从哪来看[获得与消耗](#获得与消耗)；改数值或改 debuff 前先看[常见陷阱](#常见陷阱与限制)。
- 重要副作用：吃生火腺时，**没有 `eatsrawmeat` 的食用者**（缺 `eater` 组件也算）会被 `AddDebuff("mitegland_debuff","mitegland_debuff")`；debuff 本体**不直接**扣生命／饥饿／理智，只给目标挂一项**温度修正量 +60**（`Temperature:SetModifier`，加到体温目标值上）、时长 60 秒，并在首次 5 秒后每 10 秒于 `idle` 状态播一句台词；该修正量会把体温目标值抬高 60，一旦顶过过热阈值，掉血由温度组件结算（见[生火腺的灼烧 debuff](#生火腺的灼烧-debuff)）。熟火腺没有这个回调。
- 修改边界：不覆盖宿主生物 `cave_vent_mite` 的实体行为与刷新条件（宿主页归属待定），只写掉落关系；不展开 `fumarole` 工具／陷阱配方内部；不覆盖其它 debuff 与 `firenettle_toxin` 本体的植株侧（见 [[reference/prefabs/weeds#火荨麻叶的毒素链]]）。

## 注册与构造

三行都在 `prefabs/meats.lua` 的表尾导出里：`mitegland`、`mitegland_debuff` 与 `mitegland_cooked` 连续排列（**熟火腺是末行，不是 debuff**）。文件本身由 `prefablist.lua` 的 `"meats"` 整文件加载。[文件注册](../../../raw/current/scripts/prefablist.lua#L739)、[三行导出](../../../raw/current/scripts/prefabs/meats.lua#L929-L931)

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `mitegland` | `common("mite_gland","mite_gland","raw", MITEGLAND_TAGS, nil, MITEGLAND_COOKABLE_DATA)` → `ismastersim` 之后写三维、`SetOnEatenFn`、改鲜度 | 第 5 个参数是 `nil`：**没有干制数据**，因此不带 `dryable` 标签与组件；标签表是 `{"cattoy","rawmeat"}`。[定义](../../../raw/current/scripts/prefabs/meats.lua#L784-L801) |
| `mitegland_cooked` | `common("mite_gland","mite_gland","cooked")` → 主模拟只写三维与鲜度 | 无回调、无标签表；三维上调、理智负值收窄，鲜度不变。[定义](../../../raw/current/scripts/prefabs/meats.lua#L803-L816) |
| `mitegland_debuff` | 独立工厂 `mitegland_debuff_fn`，不挂到任何物品上 | 服务端实体：客户端一进来就 `DoTaskInTime(0, Remove)`；本体 `Hide`、带 `CLASSIFIED`、不联网。[工厂](../../../raw/current/scripts/prefabs/meats.lua#L865-L895) |

- 两件物品的共同接线来自 `common()`：pristine 前设 bank／build／动画、加基础 `meat` 标签与传入标签、`cookable` 非空时加 `cookable` 标签；主模拟挂 `edible`（`ismeat`、`foodtype = FOODTYPE.MEAT`）／`bait`／`inspectable`／`inventoryitem`＋`stackable`（默认 20）／`tradable`／`perishable`，再按参数挂 `dryable`／`cookable`。[共同工厂](../../../raw/current/scripts/prefabs/meats.lua#L138-L222)
- 干制缺席的连锁后果：`common()` 只在 `dryable.product` 非空时加 `dryable` 标签，但**只要 `dryable` 参数非空就加 `lureplant_bait`**；火腺传 `nil`，所以两件都**没有** `lureplant_bait`、也没有 `dryable` 标签。[分支](../../../raw/current/scripts/prefabs/meats.lua#L161-L172)
- 资源：`anim/mite_gland.zip`，两件共用同一 bank／build（`mite_gland`）只换动画 `raw`／`cooked`。[assets](../../../raw/current/scripts/prefabs/meats.lua#L27-L30)

## 两件对照

1 天 = 16 × 30 = 480 秒。[时间基准](../../../raw/current/scripts/tuning.lua#L19-L20)

| 注册 | 生命 | 饥饿 | 理智 | 鲜度 | 堆叠 | 标签（除 `meat`） | `badfood` |
|---|---:|---:|---:|---|---:|---|---|
| `mitegland` | +8（`HEALING_MEDSMALL`） | +18.75（`CALORIES_MEDSMALL`） | **−10**（`-SANITY_SMALL`） | 6 天（`PERISH_FAST`） | 20 | `cattoy`／`rawmeat`／`cookable` | 有 |
| `mitegland_cooked` | +20（`HEALING_MED`） | +25（`CALORIES_MED`） | **−5**（`-SANITY_TINY`） | 6 天（`PERISH_FAST`） | 20 | （无） | 有 |

- 「熟制不延长保质期」是本页与多数肉类的差异点：两件都显式设 `PERISH_FAST`，熟火腺只改三维，不换鲜度档。[生品](../../../raw/current/scripts/prefabs/meats.lua#L798)、[熟品](../../../raw/current/scripts/prefabs/meats.lua#L813)
- 两件的理智都是负值，`edible` 的 `oncheckbadfood` 因此给它们打上 `badfood`（负理智即触发，不需要生命为负）。[判定](../../../raw/current/scripts/components/edible.lua#L1-L7)
- 常量：`HEALING_MEDSMALL/MED`＝8／20，`SANITY_SMALL/TINY`＝10／5，`CALORIES_MEDSMALL/MED`＝18.75／25，`PERISH_FAST`＝6 天，堆叠默认 `STACK_SIZE_MEDITEM`＝20。[数值常量](../../../raw/current/scripts/tuning.lua#L2021-L2061)、[堆叠](../../../raw/current/scripts/tuning.lua#L81-L85)
- 与图鉴记录一致（`healthvalue` 8／20、`hungervalue` 18.75／25、`sanityvalue` −10／−5、`perishable` 2880／2880），可作交叉核对。[图鉴数据](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L904-L905)、[图鉴登记](../../../raw/current/scripts/scrapbook_prefabs.lua#L2061-L2062)
- 烹饪口径：`cooking.lua` 给 `mitegland` 登记 `meat .5, monster .5` 且带 `cancook`（无 `candry`），于是 `mitegland_cooked` 自动登记为食材并附加 `precook = 1`；这既解释了「熟火腺能下锅」，也与「生品没有干制组件」互相印证。[食材](../../../raw/current/scripts/cooking.lua#L175)、[展开规则](../../../raw/current/scripts/cooking.lua#L45-L72)

## 生火腺的灼烧 debuff

```text
吃 mitegland
  ├─ Eater:Eat 先结算生命／饥饿／理智
  └─ Edible:OnEaten → oneaten(inst, eater)
       └─ not eater.components.eater.eatsrawmeat ?
            └─ AddDebuff("mitegland_debuff", "mitegland_debuff")
                 ├─ 已有同名 → Extend：计时器重置为 60 秒
                 └─ 新建 → attach：SetParent + 温度修正量 +60 + 每 10 秒台词（首次 5 秒）
                      └─ 60 秒到 / 目标死亡 → Stop → detach：撤温度修正 + 播恢复台词 + 移除实体
```

| 环节 | 实际行为 | 证据 |
|---|---|---|
| 入口 | `edible:SetOnEatenFn(mitegland_oneaten)`；`Edible:OnEaten` **内部**先跑 `oneaten`，随后才处理体感温度与推 `oneaten` 事件——此时三维已在 `Eater:Eat` 里结算完毕 | [挂载](../../../raw/current/scripts/prefabs/meats.lua#L796)、[调用顺序](../../../raw/current/scripts/components/edible.lua#L183-L214) |
| 判定 | `if not (eater.components.eater ~= nil and eater.components.eater.eatsrawmeat)`：**没有 `eater` 组件**、或 `eatsrawmeat` 为假，都会进 `AddDebuff` | [回调](../../../raw/current/scripts/prefabs/meats.lua#L778-L782) |
| `eatsrawmeat` 从哪来 | `Eater:SetCanEatRawMeat(true)` 置位并加同名标签；蜘蛛、韦伯、洞穴螨虫、盾之恐怖等带该位 | [置位](../../../raw/current/scripts/components/eater.lua#L103-L111)、[韦伯](../../../raw/current/scripts/prefabs/webber.lua#L123) |
| `AddDebuff` 门禁 | 目标没有 `debuffable` 时自动补组件；仅在 `DebuffsEnabled()` 且目标**不是死亡／幽灵**时真正挂上，否则静默返回假（无报错、无副作用） | [AddDebuff](../../../raw/current/scripts/entityscript.lua#L2140-L2154) |
| 实体形态 | 客户端分支立刻 `Remove`；本体 `Hide()`＋`CLASSIFIED` 标签＋不联网，只有 `debuff` 与 `timer` 组件，`keepondespawn = true` | [工厂](../../../raw/current/scripts/prefabs/meats.lua#L865-L895) |
| attach | `entity:SetParent(target.entity)`、位置归零；监听目标 `death` → `debuff:Stop()`；有 `temperature` 时 `SetModifier("mitegland_debuff", FIRE_NETTLE_TOXIN_TEMP_MODIFIER)`；起 `DoPeriodicTask(10, ..., 5, target)` | [OnAttached](../../../raw/current/scripts/prefabs/meats.lua#L828-L841) |
| 台词节拍 | 每 10 秒（首次 5 秒）检查：目标有 `talker`、有 `health` 且未死、且带 `idle` 标签时播 `ANNOUNCE_MITEGLAND_DEBUFF` | [tick](../../../raw/current/scripts/prefabs/meats.lua#L822-L826)、[台词](../../../raw/current/scripts/speech_wilson.lua#L1257-L1262) |
| 时长与重置 | `FIRE_NETTLE_TOXIN_DURATION = seg_time * 2`＝60 秒；`OnExtended` 先 `StopTimer` 再 `StartTimer` 到满时长——重复吃只刷新，不叠加 | [常量](../../../raw/current/scripts/tuning.lua#L5825-L5826)、[重置](../../../raw/current/scripts/prefabs/meats.lua#L854-L857) |
| 结束 | 计时器名 `mitedebuff_timer`，`timerdone` → `debuff:Stop()` → 移除 debuff → `OnDetached` 撤温度修正、播 `ANNOUNCE_MITEGLAND_DEBUFF_DONE`、`inst:Remove()` | [计时与到期](../../../raw/current/scripts/prefabs/meats.lua#L859-L863)、[OnDetached](../../../raw/current/scripts/prefabs/meats.lua#L843-L852) |
| 存档 | `debuffable:OnSave` 把每个 debuff 实体存成 save record，`OnLoad` 重新生成并注册；`timer` 自带 `timeleft` 存取 | [存取](../../../raw/current/scripts/components/debuffable.lua#L126-L150)、[计时器存取](../../../raw/current/scripts/components/timer.lua#L122-L142) |

- 改的是**温度修正量、不是体温本身**：`Temperature:SetModifier` 按名字记账，`totalmodifiers` 只在算 `delta` 时加进体温目标值（`delta = 环境 + totalmodifiers - current`），所以体温是逐步趋近「环境 +60」而不是瞬移；同名同值直接返回、`RemoveModifier` 按名字扣回，`mitegland_debuff` 期间再吃一个不会把 +60 叠成 +120，只会把到期时间推回满。[SetModifier](../../../raw/current/scripts/components/temperature.lua#L198-L229)
- 与火荨麻叶互不覆盖：植株／毒叶用的是另一个键 `"firenettle_toxin"`（同一组 +60／60 秒常量），两个一起中就是 +120；同名键才互相覆盖。[火荨麻叶](../../../raw/current/scripts/prefabs/firenettles.lua#L86-L124)、[常量](../../../raw/current/scripts/tuning.lua#L5825-L5826)
- 这条链**没有直接伤害**：tick 只调 `talker:Say`，本体只写 `temperature`，不直接动 `health`／`hunger`／`sanity`。但 +60 是加算的温度修正量，会计入体温目标值（`delta = 环境 + totalmodifiers - current`），体温被顶到 `overheattemp`（默认 `OVERHEAT_TEMP`＝70）之上时，温度组件每帧按 `overheathurtrate or hurtrate` 扣血、原因记 `"hot"`——是否越线取决于环境温度与当前体温，**不是必然**。[tick](../../../raw/current/scripts/prefabs/meats.lua#L822-L826)、[修正量进目标值](../../../raw/current/scripts/components/temperature.lua#L367)、[过热掉血](../../../raw/current/scripts/components/temperature.lua#L516-L523)、[阈值](../../../raw/current/scripts/tuning.lua#L2301)
- 默认过热速率是 `hurtrate`＝`WILSON_HEALTH / FREEZING_KILL_TIME`＝150／120≈1.25 点/秒（temperature.lua#L31），但结算读 `overheathurtrate or hurtrate`，而该值可被角色覆盖：薇洛在 `willow.lua` 里 `SetOverheatHurtRate(WILSON_HEALTH / WILLOW_OVERHEAT_KILL_TIME)`，`WILLOW_OVERHEAT_KILL_TIME`＝180 → 约 0.83 点/秒。[默认速率](../../../raw/current/scripts/tuning.lua#L753)、[setter](../../../raw/current/scripts/components/temperature.lua#L67-L69)、[薇洛覆盖](../../../raw/current/scripts/prefabs/willow.lua#L176)、[薇洛常量](../../../raw/current/scripts/tuning.lua#L3772)

## 获得与消耗

只写产出与消耗关系；`cave_vent_mite` 的实体行为、刷新节奏与世界生成归属另页处理（宿主页待定）。

| 来源／去向 | 产出或消耗 | 条件 | 证据 |
|---|---|---|---|
| `cave_vent_mite`（洞穴螨虫） | **基础**随机掉落二选一：`monstermeat` 或 `mitegland`，各权重 1 | `numrandomloot = 1`，基础抽取概率约 50% 出火腺；若它死亡时正在燃烧或已烧毁，`DropLoot` 会把抽中的火腺改判为 `mitegland_cooked`（`PrefabExists(v.."_cooked")` 分支），所以「烧着打死」可能直接给熟火腺 | [掉落](../../../raw/current/scripts/prefabs/cave_vent_mite.lua#L184-L187)、[权重语义](../../../raw/current/scripts/components/lootdropper.lua#L41-L54)、[烧毁改判](../../../raw/current/scripts/components/lootdropper.lua#L415-L431) |
| `cave_vent_mite_spawner` | 按 `childspawner` 周期放出螨虫；`TUNING.CAVE_MITE_ENABLED` 关掉时初始存量为 0 | 生成点属于洞穴 vents 房间，归世界生成页 | [生成器](../../../raw/current/scripts/prefabs/cave_vent_mite_spawner.lua#L100-L133) |
| 烤制 | `mitegland` → `mitegland_cooked`（`cookable.product`） | 需要 `cookable` 标签与 `cooker` 目标 | [赋值](../../../raw/current/scripts/prefabs/meats.lua#L785) |
| 配方消耗 | `fumaroleaxe`／`fumarolepickaxe`／`fumaroleshovel`／`fumarolehammer`／`fumarole_farm_hoe` 各 2 个；`trap_fumarole` 2 个；`healingsalve_fumarole` 1 个 | 均 `TECH.SCIENCE_TWO` | [配方](../../../raw/current/scripts/recipes.lua#L1077-L1084) |
| 猫浣熊 | 带 `cattoy` 即可被接收与玩耍（`PLAY_TAGS` 含 `cattoy`）；本件**没有** `cattoy` 组件 | 与 `catfood` 分支无关 | [接收](../../../raw/current/scripts/prefabs/catcoon.lua#L273-L279)、[玩耍目标](../../../raw/current/scripts/brains/catcoonbrain.lua#L39-L58) |

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`edible` | 三维、`foodtype`、`badfood`、`SetOnEatenFn` | 仅主模拟；`Eater:Eat` **先结算**三维（生命／饥饿／理智）再调 `Edible:OnEaten`，`oneaten` 是 `OnEaten` 内最先执行的回调（食用者自身的 `oneatfn` 更早一步）；本页 debuff 只动温度修正量，与结算顺序无交互 | [[reference/components/edible#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/meats.lua#L793-L796)、[结算在前](../../../raw/current/scripts/components/eater.lua#L248-L256)、[OnEaten 在后](../../../raw/current/scripts/components/eater.lua#L270-L275)、[oneaten 最先行](../../../raw/current/scripts/components/edible.lua#L183-L186) |
| 组件：`perishable` | 两件都 6 天，腐坏落 `spoiled_food`（继承 `common` 默认） | 主模拟；`StartPerishing()` 在 `common` 内 | [[reference/components/perishable#关键接口]]；[默认值](../../../raw/current/scripts/prefabs/meats.lua#L196-L199) |
| 组件：`cookable` | 生品写 `product = "mitegland_cooked"` | 主模拟；熟品无组件 | [[reference/components/cookable#关键接口]]；[数据](../../../raw/current/scripts/prefabs/meats.lua#L785) |
| 组件：`stackable` | 不写 `maxsize`，默认 20 | 主模拟 | [[reference/components/stackable#关键接口]]；[默认值](../../../raw/current/scripts/components/stackable.lua#L26-L28) |
| 组件：`debuff`／`debuffable` | 回调三件套（attach／detach／extend）、`keepondespawn`、注册与存档 | 仅服务端；客户端实体立刻移除 | [[reference/components/debuff#关键接口]]；[接线](../../../raw/current/scripts/prefabs/meats.lua#L883-L890) |
| 组件：`temperature` | 加算温度修正 `"mitegland_debuff"` | 目标有该组件才生效（玩家、多数生物） | [[reference/components/temperature#关键接口]]；[设置](../../../raw/current/scripts/prefabs/meats.lua#L836-L838) |
| 组件：`timer` | 单一计时器 `mitedebuff_timer`，到期回调 | 主模拟；存取剩余时间 | [[reference/components/timer#关键接口]]；[计时](../../../raw/current/scripts/prefabs/meats.lua#L889-L892) |
| 组件：`talker` | 只用于 `Say` 两句台词 | 需要 `talker` 且目标 `idle` | [[reference/components/talker#关键接口]]；[台词](../../../raw/current/scripts/prefabs/meats.lua#L822-L826) |
| 数据：`TUNING.FIRE_NETTLE_TOXIN_*` | 复用火荨麻叶的两个常量（+60／60 秒） | 改常量会同时改两条链 | [常量](../../../raw/current/scripts/tuning.lua#L5825-L5826) |
| 数据：`cooking.lua` | `meat .5, monster .5`＋`precook` 展开 | 加载期注册 | [食材](../../../raw/current/scripts/cooking.lua#L175) |
| 关联 Prefab：`cave_vent_mite` | 唯一原版产出源 | 掉落关系，不含实体行为 | [掉落](../../../raw/current/scripts/prefabs/cave_vent_mite.lua#L184-L187) |
| 关联 Prefab：`firenettle_toxin` | 同常量、不同键的另一条毒素链 | 植株侧触发，物品侧是叶片 | [[reference/prefabs/weeds#火荨麻叶的毒素链]] |
| 资源：`mite_gland.zip` | 两件物品动画 | 只核验声明，未验证资源内容 | [assets](../../../raw/current/scripts/prefabs/meats.lua#L27-L30) |

## 常见陷阱与限制

- **误区：把火腺 debuff 当持续 DoT**：debuff 本体不直接扣血，tick 也只播台词；但它挂的是温度修正量、会把体温目标值抬高，**体温越过过热阈值同样会掉血**，所以既不能按固定 DoT 记伤害数字，也不能当纯装饰忽略——在高温环境或已接近过热时吃生火腺，伤害来自温度组件而不是 debuff 自身。[tick](../../../raw/current/scripts/prefabs/meats.lua#L822-L826)、[过热掉血](../../../raw/current/scripts/components/temperature.lua#L516-L523)
- **误区：烤过就更耐放**：两件都是 `PERISH_FAST`＝6 天，熟制只抬三维；与多数肉类「熟制延长一档」不同。[熟品](../../../raw/current/scripts/prefabs/meats.lua#L813)
- **边界：判定是「没有 `eatsrawmeat`」，不是「是不是玩家」**：没有 `eater` 组件的食用者同样会进 `AddDebuff`；韦伯、蜘蛛一类带该位的不会挂。[回调](../../../raw/current/scripts/prefabs/meats.lua#L778-L782)
- **边界：同名 debuff 只刷新不叠加**：`SetModifier` 同名同值直接返回，`OnExtended` 只把计时器重置为满；连续吃 3 个还是同一项 +60 修正量、60 秒。[SetModifier](../../../raw/current/scripts/components/temperature.lua#L206-L211)、[重置](../../../raw/current/scripts/prefabs/meats.lua#L854-L857)
- **边界：与火荨麻叶可叠加但键不同**：`mitegland_debuff` 与 `firenettle_toxin` 各记一笔，最多 +120；把两者当同一个 buff 会漏算。[常量共用](../../../raw/current/scripts/tuning.lua#L5825-L5826)
- **边界：台词有双重前置条件**：本页 tick 只在目标带 `idle` 标签、有 `talker`、未死亡时调用；`Talker:Say` 自身还会因 `TheWorld.speechdisabled`、`ignoring`、目标死亡或睡眠而静默。跑动、战斗中或刚读档时没有台词，不代表 debuff 不在。[tick](../../../raw/current/scripts/prefabs/meats.lua#L822-L826)、[Say 门禁](../../../raw/current/scripts/components/talker.lua#L267-L296)
- **边界：死亡会提前清掉**：attach 时监听 `death` 直接 `Stop`，detach 里还会再播一次「恢复」台词（若未死）。[监听](../../../raw/current/scripts/prefabs/meats.lua#L832-L834)
- **误区：`cattoy` 等于挂了 `cattoy` 组件**：本件只有标签，猫浣熊按标签接收与玩耍；组件的 `ShouldBypassLastAirTime` 分支对它不生效。[组件判定](../../../raw/current/scripts/brains/catcoonbrain.lua#L42-L49)
- **未核验**：`CAVE_MITE_*` 各常量的具体秒数、裂隙开启条件对生成点的约束、`keepondespawn` 与读档组合的实测节拍、`mitegland_debuff` 的图鉴／成就关联。

## 验证记录

- 源码审阅：2026-11-09，Steam buildid 26474873。核对 `prefabs/meats.lua` 的 `common()`、`mitegland`／`mitegland_cooked`／`mitegland_debuff` 三个工厂与表尾导出行；`edible`／`perishable`／`cookable`／`stackable`／`eater`／`debuff`／`debuffable`／`temperature`／`timer`／`talker`／`entityscript` 的相关分支；`cave_vent_mite` 的掉落与 `cave_vent_mite_spawner` 的放怪接线；`lootdropper` 的随机掉落语义；`cooking.lua` 与 `recipes.lua` 的消耗面；`catcoon`／`catcoonbrain` 的 `cattoy` 消费；`firenettles` 的同常量对照；`tuning` 的三维／鲜度／毒素常量。
  名称核验：`NAMES.MITEGLAND`＝火腺、`NAMES.MITEGLAND_COOKED`＝熟火腺；`mitegland_debuff` 无独立 `NAMES` 条目，玩家可见文本走角色台词汇总 `STRINGS.CHARACTERS.*.ANNOUNCE_MITEGLAND_DEBUFF(_DONE)`，本页以威尔逊条目为例。[NAMES.MITEGLAND](../../../raw/current/scripts/languages/chinese_s.po#L345489-L345497)、[台词](../../../raw/current/scripts/speech_wilson.lua#L1257-L1262)
  未覆盖：宿主生物实体行为与刷新常量、裂隙条件、`fumarole` 配方内部、读档实测。
- 游戏运行：未运行。三维、天数与秒数均由 `tuning` 常量推导。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 通过（见交付回报）。

## 相关链接

- 食用与 debuff 机制：[[reference/components/edible]]、[[reference/components/debuff]]、[[reference/components/debuffable]]、[[reference/components/temperature]]、[[reference/components/timer]]、[[reference/components/talker]]
- 同常量、不同键的毒素链：[[reference/prefabs/weeds#火荨麻叶的毒素链]]
- 同工厂的鱼肉类（同批新页）：[[reference/prefabs/fishmeat]]

返回 [[reference/prefabs/index|Prefab 索引]]。
