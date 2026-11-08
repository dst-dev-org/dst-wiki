---
title: "胡须（beardhair）"
created: "2026-11-10"
updated: "2026-11-10"
type: prefab
summary: "单注册物品胡须：兔子 beardling 形态、梦魇猴与伯尼空燃料腐坏的掉落，40 叠、45 秒燃料与雪人贴饰，以及伯尼、肉块雕像、地毯、冬宴与炼金转化等消费者。"
aliases: ["beardhair", "胡须", "兔人胡须", "雪人装饰胡须", "燃料胡须"]
tags: [prefab, item, loot, recipe]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/beardhair.lua", "prefabs/rabbit.lua", "prefabs/monkey.lua", "prefabs/bunnyman.lua", "prefabs/bernie_inactive.lua", "prefabs/wilson.lua", "prefabs/tumbleweed.lua", "prefabs/rabbitking.lua", "components/lootdropper.lua", "components/beard.lua", "components/snowmandecor.lua", "components/snowmandecoratable.lua", "componentactions.lua", "componentutil.lua", "standardcomponents.lua", "recipes.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# 胡须（beardhair）

## 快速判断

- 职责与适用：`prefabs/beardhair.lua` 单注册的可拾取物品“胡须”（S1）；它同时是燃料与雪人贴饰，主要获取端是兔子 beardling 形态、梦魇猴与伯尼腐坏，主要消费端是伯尼、肉块雕像、地毯、冬宴与炼金转化。[注册](../../../raw/current/scripts/prefabs/beardhair.lua#L6-L47)
- 优先使用：查“这只兔子为什么掉胡须”看[获取](#获取)；查堆叠／燃料／雪人装饰看[注册与构造](#注册与构造)；查能做什么看[配方与消费者](#配方与消费者)。
- 重要副作用：兔子 beardling 池是“三选一”而不是三件都掉；伯尼腐坏固定给胡须、牛毛、蜘蛛丝各 1（S2）；物品本身 40 叠、45 秒燃料、15 秒可燃，烧毁成灰。[池](../../../raw/current/scripts/prefabs/rabbit.lua#L96-L104)、[伯尼](../../../raw/current/scripts/prefabs/bernie_inactive.lua#L28-L37)
- 修改边界：兔子／兔人／伯尼只写产出本物品的分支，完整宿主行为归 [[reference/prefabs/smallmeat]]、[[reference/prefabs/farm-crops-staple]]、[[reference/prefabs/character-gear-extra]]；雪人装饰会话归 [[reference/components/snowmandecoratable]]。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 属性与证据 |
|---|---|---|
| `beardhair` | `fn()`：实体→Transform／AnimState→`MakeInventoryPhysics`→bank／build `beardhair`、播 `idle`→`AddNetwork`→`MakeInventoryFloatable("med", nil, 0.75)`→`SetPristine`；主模拟再挂 `inspectable`／`inventoryitem`／`stackable`／`fuel`／`snowmandecor`／`MakeSmallBurnable`／`MakeSmallPropagator`／`MakeHauntableLaunchAndIgnite`。[工厂](../../../raw/current/scripts/prefabs/beardhair.lua#L6-L45) | 单注册物品；`stackable.maxsize = TUNING.STACK_SIZE_SMALLITEM`＝40；`fuel.fuelvalue = TUNING.MED_FUEL`＝`seg_time×1.5`＝45 秒；`MakeSmallBurnable(inst, TUNING.MED_BURNTIME)`＝15 秒；`snowmandecor` 是空标记组件，外观表在 `snowmandecoratable.lua` 的 `beardhair` 条目（bank／build／anim＋`canflip`）。[堆叠](../../../raw/current/scripts/prefabs/beardhair.lua#L31-L32)、[燃料](../../../raw/current/scripts/prefabs/beardhair.lua#L34-L35)、[燃烧](../../../raw/current/scripts/prefabs/beardhair.lua#L39-L40)、[雪人表](../../../raw/current/scripts/components/snowmandecoratable.lua#L5-L9) |

- 40 叠来自 [`STACK_SIZE_SMALLITEM`](../../../raw/current/scripts/tuning.lua#L81-L84)，工厂只是把它赋给 `maxsize`；45 秒燃料来自 [`MED_FUEL`](../../../raw/current/scripts/tuning.lua#L1673-L1678)；15 秒可燃来自 [`MED_BURNTIME`](../../../raw/current/scripts/tuning.lua#L1680-L1683)。
- `beardhair` 没有 `edible`、`tradable`、`perishable`：不能吃、没有金值、不会腐坏。`MakeInventoryFloatable` 的 `"med"`／0.75 只影响落水漂浮表现。[工厂](../../../raw/current/scripts/prefabs/beardhair.lua#L18-L22)
- 名称核验：`STRINGS.NAMES.BEARDHAIR` 为 **胡须**（S1）。[po](../../../raw/current/scripts/languages/chinese_s.po#L339209-L339212)

## 获取

| 来源 | 触发与数量 | 证据 |
|---|---|---|
| 兔子 `rabbit` 的 beardling 战利品（S2） | 击杀者疯狂：`SetLoot(nil)` 后重建 `beardhair` 0.5／`monstermeat` 1／`nightmarefuel` 1，`numrandomloot=1`，只出 1 件，胡须占 0.5／2.5＝20%。强制噩梦：先 `SetLoot({nightmarefuel})` 保底 1 个，再以 0.5 为基础概率由 `TryLuckRoll` 按幸运修正追加 1 件同池物品，胡须仍只占该追加件的 20%。 | [池](../../../raw/current/scripts/prefabs/rabbit.lua#L96-L104)、[强制](../../../raw/current/scripts/prefabs/rabbit.lua#L106-L116)、[分流](../../../raw/current/scripts/prefabs/rabbit.lua#L283-L291)、[加权抽取](../../../raw/current/scripts/components/lootdropper.lua#L99-L112)、[幸运判定](../../../raw/current/scripts/componentutil.lua#L2237-L2251) |
| 梦魇猴 `monkey` | 正常猴 `SetLoot({smallmeat, cave_banana})` 且清空 chance table；进入梦魇形态后 `SetChanceLootTable("monkey")`，共享表逐项独立：`smallmeat` 1.0／`cave_banana` 1.0／`beardhair` 1.0／`nightmarefuel` 0.5；强制噩梦另 `SetLoot({nightmarefuel})` 保底 1 个。 | [表](../../../raw/current/scripts/prefabs/monkey.lua#L25-L31)、[正常](../../../raw/current/scripts/prefabs/monkey.lua#L264-L274)、[梦魇切换](../../../raw/current/scripts/prefabs/monkey.lua#L305-L312)、[调用](../../../raw/current/scripts/prefabs/monkey.lua#L327-L329)、[世界梦魇](../../../raw/current/scripts/prefabs/monkey.lua#L371-L372)、[读档](../../../raw/current/scripts/prefabs/monkey.lua#L395-L401)、[强制](../../../raw/current/scripts/prefabs/monkey.lua#L359-L363) |
| 伯尼 `bernie_inactive` 空燃料腐坏 | 空燃料且未持有，落地 3 总日＝1440 秒后 `dodecay` 固定生成 `beardhair`＋`beefalowool`＋`silk` 各 1，然后移除伯尼；拿起即取消腐坏。 | [掉落](../../../raw/current/scripts/prefabs/bernie_inactive.lua#L28-L43)、[触发](../../../raw/current/scripts/prefabs/bernie_inactive.lua#L125-L143)、[落地](../../../raw/current/scripts/prefabs/bernie_inactive.lua#L167-L173)、[时长](../../../raw/current/scripts/tuning.lua#L3755-L3755) |
| 兔人 `bunnyman` 的 beardlord | 击杀者疯狂时 `SetLoot({beardhair, beardhair, monstermeat})`：2 胡须＋1 怪物肉保底；强制噩梦版再加 1 噩梦燃料。 | [表](../../../raw/current/scripts/prefabs/bunnyman.lua#L41-L42)、[分支](../../../raw/current/scripts/prefabs/bunnyman.lua#L303-L318)、[固定表结算](../../../raw/current/scripts/components/lootdropper.lua#L236-L240) |
| 兔王召唤的 `rabbitkingminion_bunnyman` | 5 项权重池中 `beardhair` 权重 3、总权重 16，`numrandomloot=1` → 3/16＝18.75%。 | [池](../../../raw/current/scripts/prefabs/rabbitking.lua#L462-L468)、[加权抽取](../../../raw/current/scripts/components/lootdropper.lua#L99-L112) |
| 威尔逊剃须 | `beard.prize = "beardhair"`；`Beard:Shave` 每掉 1 个 bit 生成 1 个奖品；胡须等级 bits 为 1／3／9。 | [奖品](../../../raw/current/scripts/prefabs/wilson.lua#L152-L158)、[剃须](../../../raw/current/scripts/components/beard.lua#L144-L155)、[bits 定义](../../../raw/current/scripts/tuning.lua#L7614-L7618)、[赋值](../../../raw/current/scripts/prefabs/wilson.lua#L109-L129) |
| 风滚草 | 开包按权重抽 3 件；`beardhair` 权重 0.02（权重不是 2%），`beefalowool` 权重 1。 | [权重表](../../../raw/current/scripts/prefabs/tumbleweed.lua#L171-L173)、[抽 3 件](../../../raw/current/scripts/prefabs/tumbleweed.lua#L218-L244) |

- **兔子外观与战利品不是同一判定**：服务端按击杀者理智决定是否走 beardling 池；客户端另按观察者理智把兔子贴图覆盖成 `beard_monster`，所以“看见的是胡须兔”不等于这只兔子一定按 beardling 池结算。[战利品](../../../raw/current/scripts/prefabs/rabbit.lua#L283-L291)、[客户端覆盖](../../../raw/current/scripts/prefabs/rabbit.lua#L360-L364)
- **伯尼不是随机池**：`dodecay` 连续三次 `SpawnLootPrefab`，胡须、牛毛、蜘蛛丝各 1 件，没有概率判定；只有“空燃料且未持有”与 1440 秒计时两个门禁。[掉落](../../../raw/current/scripts/prefabs/bernie_inactive.lua#L28-L36)

## 配方与消费者

| 产物／用途 | 材料（本物品数量） | 证据 |
|---|---|---|
| `bernie_inactive`（伯尼） | 胡须 2＋牛毛 2＋蜘蛛丝 2，限 Willow（`builder_tag="pyromaniac"`） | [配方](../../../raw/current/scripts/recipes.lua#L94-L94) |
| `resurrectionstatue`（肉块雕像） | 胡须 4＋木板 4＋角色生命值代价 | [配方](../../../raw/current/scripts/recipes.lua#L580-L580) |
| `turf_beard_rug`（胡须地毯） | 胡须 2＋采下的草 2，一次产出 4 | [配方](../../../raw/current/scripts/recipes.lua#L697-L697) |
| `wintercooking_stuffing`（烤火鸡面包馅） | 胡须 2＋节日欢愉 1 | [配方](../../../raw/current/scripts/recipes.lua#L1444-L1444) |
| `transmute_beardhair` 与反向转化 | 牛毛 2→胡须 1；胡须 2→牛毛 1，均需 Wilson 炼金技能 9 | [正向](../../../raw/current/scripts/recipes.lua#L1488-L1488)、[反向](../../../raw/current/scripts/recipes.lua#L1489-L1489) |
| `rabbitkingspear` 拆解 | 兔王棍拆回胡须 3＋兔绒 1＋绳索 1 | [拆解表](../../../raw/current/scripts/recipes.lua#L1619-L1619) |
| 雪人贴饰（非配方） | 装饰会话按 `beardhair` 的 `ITEM_DATA` 表项摆放并消耗背包中的胡须 | [表项](../../../raw/current/scripts/components/snowmandecoratable.lua#L5-L9)、[校验与扣除](../../../raw/current/scripts/components/snowmandecoratable.lua#L244-L297) |

- **消费端简中名按 po**：肉块雕像、烤火鸡面包馅、节日欢愉（`wintersfeastfuel`）、采下的草（`cutgrass`）、兔王棍、蜘蛛丝。[雕像](../../../raw/current/scripts/languages/chinese_s.po#L348614-L348617)、[面包馅](../../../raw/current/scripts/languages/chinese_s.po#L352184-L352187)、[节日欢愉](../../../raw/current/scripts/languages/chinese_s.po#L352214-L352217)、[采下的草](../../../raw/current/scripts/languages/chinese_s.po#L341944-L341947)、[兔王棍](../../../raw/current/scripts/languages/chinese_s.po#L348449-L348452)、[蜘蛛丝](../../../raw/current/scripts/languages/chinese_s.po#L349384-L349387)

## 关联机制导航

| 类型／对象 | 本页如何使用 | 条件、端与证据 |
|---|---|---|
| 组件：`lootdropper` | 兔子／兔人用 `SetLoot`／`AddRandomLoot` 建表，死亡时 `GenerateLoot` 结算 | 仅服务端；`SetLoot` 会清空 `randomloot`、`numrandomloot`、`chanceloot`（`chanceloottable` 不在此列） | [清空](../../../raw/current/scripts/components/lootdropper.lua#L30-L35)、[生成](../../../raw/current/scripts/components/lootdropper.lua#L186-L200)；[[reference/components/lootdropper]] |
| 组件：`beard` | 威尔逊的奖品是胡须；剃须按 `bits` 差生成 | 玩家角色，需持剃刀类工具 | [Shave](../../../raw/current/scripts/components/beard.lua#L107-L166)；[[reference/components/beard]] |
| 组件：`snowmandecor`＋表 `snowmandecoratable` | 物品侧只挂空标记；雪人侧按哈希表取 bank／build／anim 并消耗物品 | 仅雪人装饰会话；目标需 `heavy` 且非 `waxedplant` | [物品侧](../../../raw/current/scripts/prefabs/beardhair.lua#L37-L37)、[动作供给](../../../raw/current/scripts/componentactions.lua#L1745-L1756)、[表](../../../raw/current/scripts/components/snowmandecoratable.lua#L5-L9)；[[reference/components/snowmandecoratable]] |
| 组件：`fuel`＋`burnable` | 45 秒燃料；15 秒烧完，走 `DefaultBurntFn` 成灰 | 物品态；投入火源点燃，作祟只弹飞 | [燃料](../../../raw/current/scripts/prefabs/beardhair.lua#L34-L35)、[可燃](../../../raw/current/scripts/prefabs/beardhair.lua#L39-L40)、[烧毁](../../../raw/current/scripts/standardcomponents.lua#L164-L174)、[作祟实现](../../../raw/current/scripts/standardcomponents.lua#L1191-L1217) |
| 关联 Prefab：`rabbit`／`monkey`／`bunnyman`／`bernie_inactive`／`rabbitkingminion_bunnyman` | 五个实体宿主；本页只写产出胡须的分支（表中另有威尔逊与风滚草） | 见“获取”表 | [[reference/prefabs/smallmeat]]、[[reference/prefabs/farm-crops-staple]]、[[reference/prefabs/character-gear-extra]] |

## 常见陷阱与限制

- **误区：普通兔子也会掉胡须。** 普通兔子 `SetRabbitLoot` 只给 `smallmeat`；只有强制噩梦或击杀者疯狂时才切到 beardling 池。[普通表](../../../raw/current/scripts/prefabs/rabbit.lua#L90-L94)、[分流](../../../raw/current/scripts/prefabs/rabbit.lua#L283-L291)
- **误区：beardling 池“三样都掉”。** `numrandomloot=1`，三选一只出一件；强制噩梦版也只有“保底噩梦燃料＋一次追加判定”，不是保底胡须。[池](../../../raw/current/scripts/prefabs/rabbit.lua#L96-L116)
- **边界：`SetLoot` 是覆盖不是追加。** 它同时清空 `randomloot`／`numrandomloot` 与 `chanceloot`；看到后面连续 `AddRandomLoot`，不能理解成“在旧表上再加”。共享概率表走 `SetChanceLootTable`，是另一条线。[清空](../../../raw/current/scripts/components/lootdropper.lua#L30-L35)
- **边界：伯尼腐坏只发生在空燃料且不持有。** 拿在手上或放进背包会 `stopdecay`；重新落地且仍为空燃料才重新计时。[落地](../../../raw/current/scripts/prefabs/bernie_inactive.lua#L162-L173)、[取消](../../../raw/current/scripts/prefabs/bernie_inactive.lua#L45-L50)
- **边界：`snowmandecor` 本身没有接口。** 它只是让物品获得装饰资格的标记；外观与消耗都在 `snowmandecoratable` 的 `ITEM_DATA` 与会话校验里。[组件](../../../raw/current/scripts/components/snowmandecor.lua#L1-L7)、[消耗](../../../raw/current/scripts/components/snowmandecoratable.lua#L244-L297)
- **边界：威尔逊的 1／3／9 是 `bits` 等级，不是一次剃须的掉落数。** `Beard:Shave` 掉的是 `oldbits−newbits`；跨级剃须会先回调到下一级并 `break`，不能把长胡须直接当 9 个胡须。[Shave](../../../raw/current/scripts/components/beard.lua#L118-L155)、[等级](../../../raw/current/scripts/tuning.lua#L7614-L7618)
- **边界：风滚草的 `chance` 是权重。** 表里 0.02 不是 2%，而是与切草 40、树枝 40 等同一权重池里的 0.02；每次只抽 3 件。[表](../../../raw/current/scripts/prefabs/tumbleweed.lua#L147-L184)

## 验证记录

- 源码审阅：2026-11-10，Steam buildid 26474873。通读 `prefabs/beardhair.lua` 全文、`prefabs/rabbit.lua` 的 beardling 池与战利品分流、`prefabs/monkey.lua` 的梦魇猴共享表与形态切换、`prefabs/bunnyman.lua` 的 beardlord 表、`prefabs/bernie_inactive.lua` 的腐坏链、`prefabs/wilson.lua` 的胡须奖品、`prefabs/rabbitking.lua` 的 minion 池、`prefabs/tumbleweed.lua` 的权重与抽 3 件、`components/lootdropper.lua` 的清空与加权抽取、`components/beard.lua` 的按 bit 掉奖、`components/snowmandecoratable.lua` 的装饰扣除、`standardcomponents.lua` 的默认烧毁、`recipes.lua` 的七条消费、`tuning.lua` 的堆叠／燃料／燃烧／时间基数、`chinese_s.po` 的名称。
- 游戏运行：未运行。未核验：雪人装饰在客户端／专用服务器上的显示差异、风滚草实际开包频率。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 见交付回报。

## 相关链接

- 兔子与 beardling 池：[[reference/prefabs/smallmeat]]
- 兔人三选一与胡萝卜：[[reference/prefabs/farm-crops-staple]]
- 伯尼两态与腐坏：[[reference/prefabs/character-gear-extra]]
- 雪人装饰组件：[[reference/components/snowmandecoratable]]
- 同族材料与炼金转化：[[reference/prefabs/beefalowool]]
- 返回 [[reference/prefabs/index|Prefab 索引]]
