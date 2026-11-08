---
title: "兔绒（manrabbit_tail）"
created: "2026-11-10"
updated: "2026-11-10"
type: prefab
summary: "单注册物品兔绒：普通兔人三选一掉落、吸血鬼兔与兔王召唤兔人的额外来源，作为 HORRIBLE 食物、40 叠、金值 2，并用于悬蛛变身涂鸦、毛皮铺盖与兔屋。"
aliases: ["manrabbit_tail", "兔绒", "兔人尾巴", "兔人掉落", "毛皮铺盖材料"]
tags: [prefab, item, food, loot, recipe]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/manrabbit_tail.lua", "prefabs/bunnyman.lua", "prefabs/cozy_bunnyman.lua", "prefabs/rabbit.lua", "prefabs/rabbitking.lua", "prefabs/squidherd.lua", "components/edible.lua", "components/tradable.lua", "prefabs/hats.lua", "prefabs/pigskin.lua", "actions.lua", "standardcomponents.lua", "recipes.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# 兔绒（manrabbit_tail）

## 快速判断

- 职责与适用：`prefabs/manrabbit_tail.lua` 单注册的可拾取物品“兔绒”（S1）。它不是普通兔子的掉落，而是兔人（bunnyman）普通死亡三选一的一支（S2），另有吸血鬼兔与兔王召唤兔人的变体来源。[注册](../../../raw/current/scripts/prefabs/manrabbit_tail.lua#L6-L51)
- 优先使用：查“兔人为什么掉兔绒”看[获取](#获取)；查能不能吃、值多少看[注册与构造](#注册与构造)；查用途看[配方与消费者](#配方与消费者)。
- 重要副作用：可食用，食物类型为 `HORRIBLE`；10 饥饿／10 生命／0 理智来自 `edible` 默认值，数值、食性与禁忌主章节见 [[reference/prefabs/cave-drops]]、[[reference/components/edible]]、[[reference/components/eater]]；40 叠、金值 2、猫玩具标签，没有燃料与新鲜度。[食用](../../../raw/current/scripts/prefabs/manrabbit_tail.lua#L43-L46)、[默认值](../../../raw/current/scripts/components/edible.lua#L21-L24)
- 修改边界：兔人、兔王与兔屋只写产出／消耗本物品的分支；兔人完整行为归 [[reference/prefabs/farm-crops-staple]]，兔王与兔屋本体未在本页展开。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 属性与证据 |
|---|---|---|
| `manrabbit_tail` | `fn()`：实体→Transform／AnimState／Network→`MakeInventoryPhysics`→bank／build `manrabbit_tail`、播 `idle`→`pickupsound="cloth"`→`AddTag("cattoy")`→`MakeInventoryFloatable("small", 0.1, 0.85)`→`SetPristine`；主模拟挂 `stackable`／`inspectable`／`MakeSmallBurnable`／`MakeSmallPropagator`／`MakeHauntableLaunchAndIgnite`／`tradable`／`inventoryitem`／`edible`。[工厂](../../../raw/current/scripts/prefabs/manrabbit_tail.lua#L6-L49) | 单注册物品；40 叠；可燃（`MakeSmallBurnable` 未传时间，默认 10 秒）；无 `fuel`、无 `perishable`；`tradable.goldvalue` 显式设为 `GOLD_VALUES.MEAT×2`＝2；`edible.foodtype=FOODTYPE.HORRIBLE`。[堆叠](../../../raw/current/scripts/prefabs/manrabbit_tail.lua#L31-L32)、[金值](../../../raw/current/scripts/prefabs/manrabbit_tail.lua#L40-L41)、[食用](../../../raw/current/scripts/prefabs/manrabbit_tail.lua#L45-L46) |

- 堆叠 40 来自 [`STACK_SIZE_SMALLITEM`](../../../raw/current/scripts/tuning.lua#L81-L84)；金值 1 来自 [`GOLD_VALUES.MEAT`](../../../raw/current/scripts/tuning.lua#L1946-L1953)，乘 2 后为 2。
- 名称核验：`STRINGS.NAMES.MANRABBIT_TAIL` 为 **兔绒**，英文 msgid 是 “Bunny Puff” 而不是 “Manrabbit Tail”（S1）。[po](../../../raw/current/scripts/languages/chinese_s.po#L345004-L345007)

## 获取

| 来源 | 触发与数量 | 证据 |
|---|---|---|
| 普通兔人 `bunnyman` | `AddRandomLoot`：`carrot` 3／`meat` 3／`manrabbit_tail` 2，`numrandomloot=1` → 只出 1 件，兔绒占 2/8＝25%。 | [分支](../../../raw/current/scripts/prefabs/bunnyman.lua#L303-L318)、[加权抽取](../../../raw/current/scripts/components/lootdropper.lua#L99-L112) |
| 兔人 beardlord／强制 beardlord | `SetLoot({beardhair, beardhair, monstermeat})`，强制版再加 1 噩梦燃料；两分支都不带兔绒。 | [表](../../../raw/current/scripts/prefabs/bunnyman.lua#L41-L42)、[分支](../../../raw/current/scripts/prefabs/bunnyman.lua#L303-L318)、[固定表结算](../../../raw/current/scripts/components/lootdropper.lua#L236-L240) |
| 吸血鬼兔（万圣夜） | 同一 `bunnyman` 实体转成 vampire 形态：`SetLootSetupFn(nil)`＋`SetLoot({monstermeat, monstermeat, manrabbit_tail})`，兔绒 1 件保底；另加三颗万圣糖、1 个南瓜炸弹保底、0.15 南瓜炸弹与 0.1 装饰。 | [掉落表](../../../raw/current/scripts/prefabs/bunnyman.lua#L412-L414)、[接线](../../../raw/current/scripts/prefabs/bunnyman.lua#L466-L475)、[概率结算](../../../raw/current/scripts/components/lootdropper.lua#L202-L211) |
| 兔王召唤的 `rabbitkingminion_bunnyman` | 5 项权重池：`beardhair` 3／`monstermeat` 3／`carrot` 3／`meat` 3／`manrabbit_tail` 4，总权重 16，`numrandomloot=1` → 4/16＝25%。 | [池](../../../raw/current/scripts/prefabs/rabbitking.lua#L462-L468)、[加权抽取](../../../raw/current/scripts/components/lootdropper.lua#L99-L112) |
| `rabbitkingspear` 拆解 | 拆解表给兔绒 1＋胡须 3＋绳索 1。 | [拆解](../../../raw/current/scripts/recipes.lua#L1619-L1619) |

- **活动变体也走同一池。** `cozy_bunnyman` 的 `LootSetupFunction` 仍是 `carrot` 3／`meat` 3／`manrabbit_tail` 2、`numrandomloot=1`；它是独立注册，但兔绒概率仍是 25%。[cosy](../../../raw/current/scripts/prefabs/cozy_bunnyman.lua#L289-L294)
- **普通兔子 `rabbit` 不掉兔绒。** 它的战利品表是 `rabbitloot = { "smallmeat" }`；兔绒只挂在兔人一侧。[兔子表](../../../raw/current/scripts/prefabs/rabbit.lua#L45-L45)、[普通分支](../../../raw/current/scripts/prefabs/rabbit.lua#L90-L94)
- **beardlord 不是兔绒来源。** 兔人变 beardlord 后走固定三件表，兔绒反而被 `SetLoot` 清掉；不要把“所有兔人形态”都算成 25%。[清空](../../../raw/current/scripts/components/lootdropper.lua#L30-L35)、[固定表](../../../raw/current/scripts/prefabs/bunnyman.lua#L41-L42)

## 配方与消费者

| 产物／用途 | 材料（本物品数量） | 证据 |
|---|---|---|
| `mutator_dropper`（悬蛛变身涂鸦） | 兔绒 1＋怪物肉 1＋蜘蛛丝 1，蜘蛛制作技能 1 | [配方](../../../raw/current/scripts/recipes.lua#L330-L330) |
| `bedroll_furry`（毛皮铺盖） | 兔绒 2＋草席 1，科学 2 档 | [配方](../../../raw/current/scripts/recipes.lua#L423-L423) |
| `rabbithouse`（兔屋） | 兔绒 4＋木板 4＋胡萝卜 10，科学 2 档 | [配方](../../../raw/current/scripts/recipes.lua#L583-L583) |
| `rabbitkingspear` 拆解 | 兔绒 1（另回胡须 3、绳索 1） | [拆解](../../../raw/current/scripts/recipes.lua#L1619-L1619) |
| 眼面具 `eyemaskhat` 喂食修甲 | 自带 `eater`（`EYEMASK_DIET`＝OMNI＋HORRIBLE）与 `fedbyall`；FEED 走 `eater:Eat`，兔绒（同属 HORRIBLE 的[猪皮](../../../raw/current/scripts/prefabs/pigskin.lua#L41-L42)同理）的 `health`／`hunger` 分别乘 4.0／1.75 后相加修甲（推导；受护甲上限截断） | [diet](../../../raw/current/scripts/prefabs/hats.lua#L10-L10)、[fedbyall](../../../raw/current/scripts/prefabs/hats.lua#L2321-L2322)、[配置](../../../raw/current/scripts/prefabs/hats.lua#L2353-L2357)、[结算](../../../raw/current/scripts/prefabs/hats.lua#L2329-L2332)、[FEED](../../../raw/current/scripts/actions.lua#L3768-L3791)；[[reference/prefabs/hats-armor]] |
| 交易 | `tradable.goldvalue=2`，供按金值收货的交易端读取 | [赋值](../../../raw/current/scripts/prefabs/manrabbit_tail.lua#L40-L41)、[默认金值表](../../../raw/current/scripts/tuning.lua#L1946-L1953) |

- **消费端简中名按 po**：兔王棍、兔屋、悬蛛变身涂鸦、毛皮铺盖、蜘蛛丝。[兔王棍](../../../raw/current/scripts/languages/chinese_s.po#L348449-L348452)、[兔屋](../../../raw/current/scripts/languages/chinese_s.po#L348429-L348432)、[悬蛛](../../../raw/current/scripts/languages/chinese_s.po#L346094-L346097)、[毛皮铺盖](../../../raw/current/scripts/languages/chinese_s.po#L339239-L339242)、[蜘蛛丝](../../../raw/current/scripts/languages/chinese_s.po#L349384-L349387)

## 关联机制导航

| 类型／对象 | 本页如何使用 | 条件、端与证据 |
|---|---|---|
| 组件：`lootdropper` | 兔人用 `AddRandomLoot` 建 3/3/2 池；beardlord 与吸血鬼兔改用 `SetLoot` 固定表 | `SetLoot` 清空随机表；`numrandomloot=1` 只抽一次 | [清空](../../../raw/current/scripts/components/lootdropper.lua#L30-L35)、[抽一次](../../../raw/current/scripts/components/lootdropper.lua#L186-L200)；[[reference/components/lootdropper]] |
| 组件：`edible` | 可食用；只显式设 `foodtype`，三维用默认值 | 进食端还会看角色禁忌与食物类型 | [挂载](../../../raw/current/scripts/prefabs/manrabbit_tail.lua#L43-L46)；[[reference/components/edible]] |
| 组件：`tradable` | 金值 2 | 只有金值 >0 的交易端才会收 | [赋值](../../../raw/current/scripts/prefabs/manrabbit_tail.lua#L40-L41)；[[reference/components/tradable]] |
| 组件：`stackable`／`burnable` | 40 叠；默认 10 秒可燃，烧毁成灰 | 物品态 | [堆叠](../../../raw/current/scripts/prefabs/manrabbit_tail.lua#L31-L32)、[可燃](../../../raw/current/scripts/standardcomponents.lua#L164-L174)；[[reference/components/stackable]] |
| 关联 Prefab：`bunnyman`／`rabbitkingminion_bunnyman` | 两个实际掉落方；兔王 minion 是兔王召唤物，不是另一个物品 | 见“获取”表 | [[reference/prefabs/farm-crops-staple]]、[minion 注册](../../../raw/current/scripts/prefabs/rabbitking.lua#L611-L611) |

## 常见陷阱与限制

- **误区：兔绒是“兔子”掉的。** 普通兔子只掉小肉；兔绒来自兔人（bunnyman）、吸血鬼兔与兔王召唤的兔人。把 rabbit 与 bunnyman 混写会得到完全错误的 25%。[兔子](../../../raw/current/scripts/prefabs/rabbit.lua#L45-L45)、[兔人](../../../raw/current/scripts/prefabs/bunnyman.lua#L313-L316)
- **边界：3／3／2 是权重，不是三个概率。** `numrandomloot=1`，每次只抽一件；加起来 8 份，兔绒 2/8。不要写成“37.5% 胡萝卜、37.5% 肉、25% 兔绒，另加兔绒”。[池](../../../raw/current/scripts/prefabs/bunnyman.lua#L313-L316)、[抽取](../../../raw/current/scripts/components/lootdropper.lua#L99-L112)
- **边界：beardlord 与强制 beardlord 不掉兔绒。** 它们用 `SetLoot` 固定给胡须与怪物肉；只有普通、吸血鬼与兔王 minion 三条支路有兔绒。[固定表](../../../raw/current/scripts/prefabs/bunnyman.lua#L41-L42)
- **边界：吸血鬼兔不是独立注册。** 它仍是 `bunnyman` 实体；除 `SetPrefabNameOverride("vampire_bunny")` 与掉落表外，还改 brain／build／tags／health／damage／locomotor／sanityaura，并禁用 trader、清空 follower leader。按注册名统计不要把它当第二只兔人。[形态转换](../../../raw/current/scripts/prefabs/bunnyman.lua#L412-L484)
- **边界：没有新鲜度与燃料。** 它不会腐坏，也不能当燃料；可燃只表示会被烧成灰。[工厂](../../../raw/current/scripts/prefabs/manrabbit_tail.lua#L31-L49)
- **误区：`squidherd.lua` 里那段生成兔绒的代码是来源。** 它位于 Lua 注释块内，不执行；不能据此把鱿鱼群算成兔绒来源。[注释块](../../../raw/current/scripts/prefabs/squidherd.lua#L93-L101)

## 验证记录

- 源码审阅：2026-11-10，Steam buildid 26474873。通读 `prefabs/manrabbit_tail.lua` 全文、`prefabs/bunnyman.lua` 的普通／beardlord／吸血鬼三条掉落链、`prefabs/rabbitking.lua` 的 minion 池与注册、`prefabs/rabbit.lua` 的普通兔子表、`prefabs/squidherd.lua` 的注释块、`components/edible.lua`／`tradable.lua`／`lootdropper.lua`、`prefabs/hats.lua` 的 eyemask eater／fedbyall／修甲、`actions.lua` 的 FEED、`standardcomponents.lua`、`recipes.lua` 的三条配方与一条拆解、`tuning.lua` 的堆叠／金值、`chinese_s.po` 的名称。
- 游戏运行：未运行。未核验：吸血鬼兔在非万圣夜的实际转换条件、兔王 minion 的实际召唤数量、不同角色对 `HORRIBLE` 食物的接受差异。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 见交付回报。

## 相关链接

- 兔人掉落与胡萝卜：[[reference/prefabs/farm-crops-staple]]
- 普通兔子与小肉：[[reference/prefabs/smallmeat]]
- 胡须与兔王棍拆解：[[reference/prefabs/beardhair]]
- 兔屋与毛皮铺盖：页面 Wiki 待建
- 返回 [[reference/prefabs/index|Prefab 索引]]
