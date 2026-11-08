---
title: "沃比零食（woby_treat）"
created: "2026-11-09"
updated: "2026-11-09"
type: prefab
summary: "沃尔特技能配方（1 份怪物肉干 → 2 份）产出的宠物零食：MEAT 主食材加 MONSTER 副食材、生命 −1 与理智 −5、金值 0、完全没有鲜度组件；quickeat 让吃它的动作走短状态、quickfeed 只缩短 `FEED` 投喂，`pet_treat` 让沃比吃到时给喂食者发事件（小沃比 `treatwoby` 带自身、大沃比 `praisewoby` 不带；事件≠真说话），饥饿收益按 prefab 名乘 3，鸟笼则把它列进拒收表。"
aliases: ["woby_treat", "沃比零食", "Woby Snacks", "沃比点心", "宠物零食"]
tags: [prefab, item, food]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/woby_treat.lua", "prefabs/oceanfish.lua", "components/wisecracker.lua", "prefabs/wobybig.lua", "prefabs/wobysmall.lua", "prefabs/birdcage.lua", "recipes.lua", "components/edible.lua", "components/eater.lua", "components/tradable.lua", "actions.lua", "componentactions.lua", "stategraphs/SGwilson.lua", "tuning.lua", "screens/redux/scrapbookdata.lua", "languages/chinese_s.po"]
size_class: simple
reliability: verified
validation: source-reviewed
status: active
---

# 沃比零食（woby_treat）

## 快速判断

- 职责与适用：单一注册 `woby_treat`，物品态食物，原版里是沃尔特（Walter）专属的宠物零食——喂给沃比（Woby）时饥饿收益乘 3 并向喂食者发 `praisewoby` 事件（是否真说话还要过距离与冷却），也能吃但会掉血掉理智。简中名 **沃比零食**（`NAMES.WOBY_TREAT`，英文 "Woby Snacks"）。[注册](../../../raw/current/scripts/prefabs/woby_treat.lua#L57-L57)、[名称](../../../raw/current/scripts/languages/chinese_s.po#L352370-L352372)
- 优先使用：数值看[数值](#数值)；想知道每个标签到底被哪一行消费看[标签与消费端](#标签与消费端)；配方与沃比收益看[来源与去向](#来源与去向)。
- 重要副作用：`edible.healthvalue` 与 `sanityvalue` 都是负值，`edible` 会在赋值时自动给物品加 `badfood` 标签，收窄了玩家间互喂的候选；物品**没有** `perishable` 组件，放多久都不会坏。[负值与 badfood](../../../raw/current/scripts/components/edible.lua#L1-L7)、[构造](../../../raw/current/scripts/prefabs/woby_treat.lua#L33-L54)
- 修改边界：本页只负责 `woby_treat` 自身；沃比本体的饥饿与形态切换归 `prefabs/wobybig.lua`／`prefabs/wobysmall.lua`（Wiki 待建），沃尔特技能树归源码 `recipes.lua`。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `woby_treat` | `fn()`：`CreateEntity` → bank／build `woby_treat`、播 `idle` → 一次性加 5 个标签（`meat`／`quickeat`／`monstermeat`／`quickfeed`／`pet_treat`，`SetPristine` 前、两端都有）→ `MakeInventoryFloatable` → `SetPristine`；**非主模拟在此返回** | 物品态：`inspectable`、`inventoryitem`、`stackable`、`edible`、`tradable`，可燃＋传热＋可作祟；没有任何鲜度／烹饪组件。[工厂](../../../raw/current/scripts/prefabs/woby_treat.lua#L6-L55)、[主模拟分界](../../../raw/current/scripts/prefabs/woby_treat.lua#L27-L33) |

## 数值

时间基准：`seg_time`＝30 秒、`total_day_time`＝480 秒。[基准](../../../raw/current/scripts/tuning.lua#L19-L20)

| 项 | 值 | 定义处／结算处 |
|---|---|---|
| 饥饿 | `CALORIES_TINY`＝9.375 | [赋值](../../../raw/current/scripts/prefabs/woby_treat.lua#L44-L44)、[常量](../../../raw/current/scripts/tuning.lua#L2058-L2058)、[结算](../../../raw/current/scripts/components/eater.lua#L224-L224) |
| 生命 | `-HEALING_TINY`＝−1 | [赋值](../../../raw/current/scripts/prefabs/woby_treat.lua#L43-L43)、[常量](../../../raw/current/scripts/tuning.lua#L2021-L2021)、[结算](../../../raw/current/scripts/components/eater.lua#L213-L213) |
| 理智 | `-SANITY_TINY`＝−5 | [赋值](../../../raw/current/scripts/prefabs/woby_treat.lua#L45-L45)、[常量](../../../raw/current/scripts/tuning.lua#L2033-L2033)、[结算](../../../raw/current/scripts/components/eater.lua#L234-L234) |
| 食物类型 | 主 `MEAT`、副 `MONSTER` | [赋值](../../../raw/current/scripts/prefabs/woby_treat.lua#L41-L42)、[标签同步](../../../raw/current/scripts/components/edible.lua#L9-L16) |
| 堆叠 | `STACK_SIZE_SMALLITEM`＝40 | [赋值](../../../raw/current/scripts/prefabs/woby_treat.lua#L36-L37)、[常量](../../../raw/current/scripts/tuning.lua#L83-L83) |
| 金值 | **显式写 0**（不是走默认） | [赋值](../../../raw/current/scripts/prefabs/woby_treat.lua#L47-L48)、[默认值对照](../../../raw/current/scripts/components/tradable.lua#L1-L3) |
| 鲜度 | 无 `perishable` 组件，永不腐坏 | [全文构造段](../../../raw/current/scripts/prefabs/woby_treat.lua#L33-L54) |
| 燃烧 | `MakeSmallBurnable(inst, TUNING.SMALL_BURNTIME)`＝`seg_time×0.25`＝7.5 秒 | [接线](../../../raw/current/scripts/prefabs/woby_treat.lua#L50-L50)、[常量](../../../raw/current/scripts/tuning.lua#L1681-L1681) |

- **负值会自己挂上 `badfood`**：`edible` 的 `healthvalue`／`sanityvalue` 赋值回调在任一为负时 `AddTag("badfood")`，所以这块零食一构造完就带 `badfood`（不写在 prefab 里）。[回调](../../../raw/current/scripts/components/edible.lua#L1-L7)、[赋值](../../../raw/current/scripts/prefabs/woby_treat.lua#L43-L45)
- **`ismeat = true` 是冗余字段**：`prefabs/oceanfish.lua` 有一句自注说它没被任何地方使用，本页检索也没找到读取它的判定行；真正参与判定的是 `edible_MEAT`／`edible_MONSTER` 标签。[自注](../../../raw/current/scripts/prefabs/oceanfish.lua#L518-L519)、[赋值](../../../raw/current/scripts/prefabs/woby_treat.lua#L40-L40)、[判定](../../../raw/current/scripts/components/eater.lua#L288-L302)

## 标签与消费端

| 标签 | 消费端判定行 | 实际效果与条件 |
|---|---|---|
| `quickeat` | 玩家进食时选状态：`obj:HasTag("quickeat") and "quickeat"`；被投喂时同理 | 吃到它播短动作 `quickeat`；没有这个标签时，`MEAT` 食物会走更长的 `eat` 状态——所以这个标签是**真正生效**的差别。[玩家](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1250-L1255)、[被喂](../../../raw/current/scripts/actions.lua#L2436-L2441) |
| `quickfeed` | 只挂在 `ACTIONS.FEED` 的 ActionHandler 上：带标签时目标在场景里返回 `"give"`、骑乘时 `"domediumaction"`、否则 `"doshortaction"`，不带标签一律 `"dolongaction"` | 只缩短喂生物／宠物的动作；**喂玩家走 `FEEDPLAYER` → `"give"`**，与 quickfeed 无关。[FEED](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1368-L1381)、[FEEDPLAYER](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1277-L1277) |
| `pet_treat` | ① `ACTIONS.FEED.strfn` 把它显示成 `"TREAT"`；② 沃比 `OnEat` 里 `food:HasTag("pet_treat")` 后向喂食者推事件：小沃比 `treatwoby`（带自身）、大沃比 `praisewoby`（**不带自身**） | 但事件≠真的说话：`wisecracker` 的 `talktowoby` 还要过 `IsNear(woby, range)` 与冷却，`woby` 为 nil 时 `IsNear` 恒 false；带参数时 `treatwoby` 4 格／6 秒，`praisewoby` 8 格／30 秒＋8 秒全局。[动作串](../../../raw/current/scripts/actions.lua#L3764-L3766)、[小沃比](../../../raw/current/scripts/prefabs/wobysmall.lua#L657-L661)、[大沃比](../../../raw/current/scripts/prefabs/wobybig.lua#L901-L905)、[说话判定](../../../raw/current/scripts/components/wisecracker.lua#L409-L425) |
| `monstermeat` | ① 喂食候选与 `FEEDPLAYER` 的安全判定：`target:HasTag("strongstomach") and inst:HasTag("monstermeat")`；② 结算侧 `eater:DoFoodEffects` 对"强胃＋`monstermeat`"返回 false，使生命与理智的负值分支被跳过 | ① 让"硬喂给强胃目标"成立；② 强胃吃它时 −1 生命／−5 理智**都不结算**，只剩 9.375 饥饿；配合 `badfood`，普通玩家间默认喂不了。[FEEDPLAYER](../../../raw/current/scripts/componentactions.lua#L1164-L1171)、[服务端复核](../../../raw/current/scripts/actions.lua#L2406-L2410)、[DoFoodEffects](../../../raw/current/scripts/components/eater.lua#L166-L170)、[生命分支](../../../raw/current/scripts/components/eater.lua#L211-L213)、[理智分支](../../../raw/current/scripts/components/eater.lua#L233-L235) |
| `meat` | 本页检索未见任何 `HasTag("meat")` 判定行（`cooking.lua` 里的 `meat` 是食材标签常量） | 未核验：无消费端；起作用的是 `foodtype = MEAT` 带来的 `edible_MEAT`。[检索](../../../raw/current/scripts/prefabs/woby_treat.lua#L19-L19) |

`secondaryfoodtype` 也会加标签：`edible` 的 `foodtype`／`secondaryfoodtype` 共用同一个赋值回调，所以同时带 `edible_MEAT`／`edible_MONSTER`，喂食候选按它与目标的 `<type>_eater` 配对。[回调](../../../raw/current/scripts/components/edible.lua#L9-L16)、[配对](../../../raw/current/scripts/componentactions.lua#L1184-L1189)、[CanEat](../../../raw/current/scripts/components/eater.lua#L288-L302)

## 来源与去向

| 环节 | 内容 | 证据 |
|---|---|---|
| 配方 | `Recipe2("woby_treat", {Ingredient("monstermeat_dried", 1)}, TECH.NONE, {builder_skill="walter_camp_wobytreat", no_deconstruction=true, numtogive=2})`：1 份怪物肉干 → 2 份，`TECH.NONE`（无科技门槛），但需要沃尔特技能 `walter_camp_wobytreat` | [配方](../../../raw/current/scripts/recipes.lua#L248-L248) |
| 喂沃比 | 沃比（大小两态各一份实现）的 `CustomFoodStatsMod` 按 **prefab 名**判定：`food.prefab == "woby_treat"` 且 `hunger_delta > 0` 时把饥饿收益乘 3，其余不变；带的 `pet_treat` 只影响表扬事件，不参与倍率 | [大沃比](../../../raw/current/scripts/prefabs/wobybig.lua#L349-L354)、[小沃比](../../../raw/current/scripts/prefabs/wobysmall.lua#L516-L521) |
| 鸟笼 | `invalid_foods` 表里直接列了 `woby_treat`，即使它是 `FOODTYPE.MEAT`（本会因食性接收）也会被拒 | [拒收表](../../../raw/current/scripts/prefabs/birdcage.lua#L30-L39)、[判定](../../../raw/current/scripts/prefabs/birdcage.lua#L160-L170) |
| 烹饪锅 | 本页检索范围内 `cooking.lua` 没有 `AddIngredientValues` 注册它，因此不把它当锅食材 | [检索范围](../../../raw/current/scripts/prefabs/woby_treat.lua#L33-L54) |

采集册数值（`hungervalue=9.375`、`healthvalue=-1`、`sanityvalue=-5`、`stacksize=40`）与源码一致。[采集册](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L1859-L1859)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`edible` | 写 MEAT＋MONSTER、9.375／−1／−5；负值自动加 `badfood` | 仅主模拟 | [[reference/components/edible]]；[接线](../../../raw/current/scripts/prefabs/woby_treat.lua#L39-L45) |
| 组件：`eater`（消费端） | `TestFood` 只按 `edible_<type>` 标签匹配；`EAT` 结算分别读生命／饥饿／理智 | 任意进食者 | [[reference/components/eater]]；[匹配](../../../raw/current/scripts/components/eater.lua#L288-L302)、[结算](../../../raw/current/scripts/components/eater.lua#L213-L234) |
| 组件：`stackable`／`tradable` | 40 叠；金值 0 | 主模拟 | [[reference/components/stackable]]、[[reference/components/tradable]]；[接线](../../../raw/current/scripts/prefabs/woby_treat.lua#L36-L48) |
| 组件：`burnable`／`propagator`／`hauntable` | 可燃 7.5 秒、可传热；`MakeHauntableLaunchAndIgnite` 在本 build 只剩 Launch（ignite 分支被注释） | 主模拟 | [[reference/components/burnable]]、[[reference/components/hauntable]]；[接线](../../../raw/current/scripts/prefabs/woby_treat.lua#L50-L52)、[helper](../../../raw/current/scripts/standardcomponents.lua#L1191-L1217) |
| 关联 Prefab：`wobybig`／`wobysmall` | 沃比大小两态各自实现"吃到零食乘 3 饥饿＋向喂食者发 `praisewoby` 事件（实际说话受距离与冷却限制）" | 服务端；只在喂食链触发 | [大](../../../raw/current/scripts/prefabs/wobybig.lua#L349-L354)、[小](../../../raw/current/scripts/prefabs/wobysmall.lua#L516-L521) |
| 关联 Prefab：`monstermeat_dried` | 配方的唯一材料；`monstermeat_dried` 本身是晾肉架产物 | 配方消耗 | [[reference/prefabs/meat]]（晾制链）；[配方](../../../raw/current/scripts/recipes.lua#L248-L248) |
| 资源：`woby_treat.zip` | bank／build 同名，动画 `idle` | 原版资源，不随页分发 | [assets](../../../raw/current/scripts/prefabs/woby_treat.lua#L1-L4) |

## 常见陷阱与限制

- **误区：它是"怪物肉"，所以谁都能喂给谁。** 有 `monstermeat` 标签，但负值会自动带上 `badfood`，普通玩家间要 `strongstomach` 目标或 PVP 才喂得动；反过来强胃吃它时 `DoFoodEffects` 返回 false，−1 生命与 −5 理智不结算，只有饥饿照算。[badfood](../../../raw/current/scripts/components/edible.lua#L1-L7)、[安全判定](../../../raw/current/scripts/componentactions.lua#L1164-L1171)、[DoFoodEffects](../../../raw/current/scripts/components/eater.lua#L166-L170)、[生命分支](../../../raw/current/scripts/components/eater.lua#L211-L213)
- **误区：沃比零食会坏。** 构造段没有任何 `perishable` 或 `StartPerishing`，它没有鲜度，放箱子多久都一样。[构造](../../../raw/current/scripts/prefabs/woby_treat.lua#L33-L54)
- **边界：乘 3 按 prefab 名判定，换 `pet_treat` 标签无效。** 要让别的物品也享受同一加成，只能改成同名 prefab 或改沃比侧实现。[大沃比](../../../raw/current/scripts/prefabs/wobybig.lua#L349-L354)
- **边界：鸟笼拒收。** 它是 MEAT 食物，本来满足鸟笼的接收条件，但 `invalid_foods` 把它单独列掉。[接收条件](../../../raw/current/scripts/prefabs/birdcage.lua#L160-L170)
- **边界：`…Ignite` 名不副实。** helper 里点火分支被注释，本 build 只 `Launch`。[helper](../../../raw/current/scripts/standardcomponents.lua#L1191-L1217)
- **未核验**：未运行游戏；`meat` 标签无消费端（见上表），不声称其有玩法作用。

## 验证记录

- 源码审阅：2026-11-09，Steam buildid 26474873。通读 `prefabs/woby_treat.lua` 全文（57 行）；核对 `edible` 的负值回调与标签同步、`eater` 的 `TestFood`／`CanEat` 与三维结算、`tradable` 默认金值；消费端核对 `quickeat`／`quickfeed`／`FEEDPLAYER` 分支、`FEED` 状态选择与安全判定、喂食候选、`CustomFoodStatsMod` 与 `OnEat`、`wisecracker` 说话判定、`birdcage` 接收与拒收、`recipes` 配方；`tuning` 常量与 `scrapbookdata` 一致。
- 名称核验：`languages/chinese_s.po` 的 `NAMES.WOBY_TREAT`＝沃比零食。
- 未覆盖：沃尔特技能树解锁条件、沃比形态切换阈值、给自定义宠物复制"乘 3"的写法。
- 游戏运行：未运行；数值均为源码推导。
- 修订：2026-11-09，初检 P2（补 `monstermeat` 的结算侧消费端 `DoFoodEffects`）——已按源代码更正，见「标签与消费端」与「常见陷阱」。
- 修订：2026-11-09，外部复检 P2（`quickfeed` 只作用于 FEED、`FEEDPLAYER` 走 give；`treatwoby`／`praisewoby` 参数不同，事件≠实际说话）——已按源代码更正，见摘要、「标签与消费端」「来源与去向」。
- 结构检查：见交付回执。

## 相关链接

- 可食与结算：[[reference/components/edible]]、[[reference/components/eater]]、[[reference/components/stackable]]
- 交易与作祟：[[reference/components/tradable]]、[[reference/components/hauntable]]、[[reference/components/burnable]]
- 同批小件：[[reference/prefabs/bird-egg]]、[[reference/prefabs/crumbs]]

返回 [[reference/prefabs/special-food-index|节庆、角色与特殊食品索引]]。
