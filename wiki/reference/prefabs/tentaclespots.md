---
title: "触手皮（tentaclespots）"
created: "2026-11-10"
updated: "2026-11-10"
type: prefab
summary: "单注册物品触手皮：触手 0.2 与大触手 0.40 的独立判定、变异鱼人 0.25 的追加表，40 叠与 15 秒燃料，鞭子等九条配方，以及可燃烧／传热但作祟不当真点燃的边界。"
aliases: ["tentaclespots", "触手皮", "触手斑点", "鞭子材料", "雨衣材料"]
tags: [prefab, item]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/tentaclespots.lua", "prefabs/tentacle.lua", "prefabs/tentacle_pillar.lua", "prefabs/merm.lua", "prefabs/tree_rocks.lua", "prefabs/tree_rock_data.lua", "util.lua", "components/stackable.lua", "components/fuel.lua", "components/fueled.lua", "components/lootdropper.lua", "standardcomponents.lua", "recipes.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# 触手皮（tentaclespots）

## 快速判断

- 职责与适用：触手皮是**唯一一个注册**的可堆叠材料（简中实显 **触手皮**，40 叠），本体是可拾取、可作燃料（基础 15 秒）、可燃烧并传热的物品，没有食用、鲜度或交易组件。[构造](../../../raw/current/scripts/prefabs/tentaclespots.lua#L15-L41)
- 优先使用：查触手与大触手各多少概率看[掉落来源](#掉落来源)；查鞭子、雨衣等配方看[用途与消耗](#用途与消耗)；查燃料值与燃烧行为看[注册与构造](#注册与构造)。
- 重要副作用：**触手类两条掉落**（触手、大触手）都在**共享概率表**里与触手尖刺同表，逐条独立判定——同一只触手可以同时掉尖刺与触手皮；**变异鱼人追加**是另一条独立概率表，**巨石枝**的藤蔓战利品则是地形权重抽取。[触手表](../../../raw/current/scripts/prefabs/tentacle.lua#L15-L21)、[大触手表](../../../raw/current/scripts/prefabs/tentacle_pillar.lua#L20-L26)、[鱼人表](../../../raw/current/scripts/prefabs/merm.lua#L86-L89)、[藤蔓表](../../../raw/current/scripts/prefabs/tree_rock_data.lua#L35-L40)
- 修改边界：触手／大触手的战斗与位移不展开；鞭子等成品归各自页面（S5）。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `tentaclespots` | `fn()`：实体 → Transform／AnimState／Network → `MakeInventoryPhysics` → bank／build `tentaclespots`、播 `idle` → 浮空 `("med", nil, 0.72)` → `SetPristine()` → 主模拟挂 `inspectable`／`inventoryitem`／`stackable`（40）／`fuel`（`SMALL_FUEL`） → `MakeSmallBurnable(SMALL_BURNTIME)` → `MakeSmallPropagator` → `MakeHauntableLaunchAndIgnite` | 唯一注册，无变体；`assets` 只有 `anim/tentaclespots.zip`。[工厂](../../../raw/current/scripts/prefabs/tentaclespots.lua#L6-L41) |

- **堆叠显式 40**：`stackable.maxsize = TUNING.STACK_SIZE_SMALLITEM`，不落组件默认的 20（S7）。[赋值](../../../raw/current/scripts/prefabs/tentaclespots.lua#L30-L31)、[常量](../../../raw/current/scripts/tuning.lua#L81-L83)
- **燃料值是 15 秒**：`fuel.fuelvalue = TUNING.SMALL_FUEL`，而 `SMALL_FUEL = seg_time × 0.5 = 30 × 0.5`；实际入炉还要乘 `bonusmult × mult × 湿燃料惩罚 × 技能倍率`。[赋值](../../../raw/current/scripts/prefabs/tentaclespots.lua#L33-L34)、[常量](../../../raw/current/scripts/tuning.lua#L1673-L1678)、[结算](../../../raw/current/scripts/components/fueled.lua#L205-L212)
- **可燃烧且会传热**：`MakeSmallBurnable` 用 `SMALL_BURNTIME`＝**7.5 秒**；`MakeSmallPropagator` 打开 `acceptsheat` 并把闪点接到 `DefaultIgniteFn`。[燃烧](../../../raw/current/scripts/standardcomponents.lua#L164-L174)、[常量](../../../raw/current/scripts/tuning.lua#L1680-L1683)、[传热](../../../raw/current/scripts/standardcomponents.lua#L200-L203)
- **烧完变灰烬**：`DefaultBurntFn` 生成 1 个 `ash` 并把叠数抄过去（受灰烬上限约束）。[回调](../../../raw/current/scripts/standardcomponents.lua#L18-L44)
- **作祟名字带 Ignite，实际只抛飞**：`MakeHauntableLaunchAndIgnite` 的点燃分支整段被 `#HAUNTFIX` 注释。[helper](../../../raw/current/scripts/standardcomponents.lua#L1191-L1217)

### 名称核验

`NAMES.TENTACLESPOTS`＝**触手皮**，是简中实显名（S1）；没有改名调用。[po 名称键](../../../raw/current/scripts/languages/chinese_s.po#L350334-L350337)

## 掉落来源

| 宿主 | 触手皮条目 | 同表其它产出 | 证据 |
|---|---|---|---|
| 触手 `tentacle` | 1 条 0.2 | 怪物肉 1.0×2、触手尖刺 0.5 | [表](../../../raw/current/scripts/prefabs/tentacle.lua#L15-L21)、[接线](../../../raw/current/scripts/prefabs/tentacle.lua#L124-L125) |
| 大触手 `tentacle_pillar` | 1 条 0.40 | 触手尖刺 0.50、沼泽地皮 0.25、石头 1.00 | [表](../../../raw/current/scripts/prefabs/tentacle_pillar.lua#L20-L26)、[接线](../../../raw/current/scripts/prefabs/tentacle_pillar.lua#L369-L370)、[死亡掉落](../../../raw/current/scripts/prefabs/tentacle_pillar.lua#L224-L230) |
| 变异鱼人 `merm_lunar`／变异鱼人守卫 `mermguard_lunar` | 追加 1 条 0.25 | 基础固定表由 `common_master` 先设，月变体再加概率表，两段并存；两个注册共用同一张表 | [追加表](../../../raw/current/scripts/prefabs/merm.lua#L86-L89)、[基础表](../../../raw/current/scripts/prefabs/merm.lua#L1468-L1489)、[接线](../../../raw/current/scripts/prefabs/merm.lua#L1727)、[守卫接线](../../../raw/current/scripts/prefabs/merm.lua#L1770)、[两个注册](../../../raw/current/scripts/prefabs/merm.lua#L1782-L1787) |
| 巨石枝（`tree_rocks.lua` 的 `tree_rock*`） | 地形表权重 2 | 生成后首帧按所在拓扑表抽 5 次（有放回，权重归一），砍倒时只把已抽好的 5 件全部落地；MARSH 与 MUD_MARSH 两张表都含该权重 | [[reference/prefabs/tree-rocks]]；[首帧抽取](../../../raw/current/scripts/prefabs/tree_rocks.lua#L789)、[砍倒只落地](../../../raw/current/scripts/prefabs/tree_rocks.lua#L619-L622)、[读档沿用](../../../raw/current/scripts/prefabs/tree_rocks.lua#L683-L684)、[两张地形表](../../../raw/current/scripts/prefabs/tree_rock_data.lua#L35-L40)、[MUD_MARSH 表](../../../raw/current/scripts/prefabs/tree_rock_data.lua#L221-L227)、[5 次抽取](../../../raw/current/scripts/prefabs/tree_rocks.lua#L67)、[按地形取表](../../../raw/current/scripts/prefabs/tree_rocks.lua#L102-L128)、[砍倒生成](../../../raw/current/scripts/prefabs/tree_rocks.lua#L615-L625)、[有放回](../../../raw/current/scripts/util.lua#L746-L763) |

- **0.2／0.40 是单条概率，不是"必掉其中一件"**：共享表每条各掷一次，所以触手可能同时给尖刺和皮，也可能两样都不给；触手尖刺侧的同表概率见 [[reference/prefabs/tentaclespike]]（S2）。[掷点](../../../raw/current/scripts/components/lootdropper.lua#L213-L225)、[共享表](../../../raw/current/scripts/components/lootdropper.lua#L21-L24)
- **大触手掉落从空中落下**：死亡回调传 `Vector3(x, 20, z)`，且只在未被 `overtaken` 时掉落。[OnDeath](../../../raw/current/scripts/prefabs/tentacle_pillar.lua#L224-L230)
- **推导**：同一只触手"至少掉 1 张皮"的概率是 20%，大触手 40%；两者都过同一个 `GetChance`，击杀者有幸运值时按 `LuckFormulas.LootDropperChance` 改算，所以上面的数字是无幸运基准。

## 用途与消耗

| 去向 | 消耗 | 条件与产物 | 证据 |
|---|---|---|---|
| 鞭子 `whip` | 猫尾 3＋触手皮 1 | `TECH.SCIENCE_TWO`；成品机制见 [[reference/prefabs/whip]] | [配方](../../../raw/current/scripts/recipes.lua#L479) |
| 羽毛帽 `featherhat` | 黑鸟羽 3＋红鸟羽 2＋触手皮 2 | `TECH.SCIENCE_TWO` | [配方](../../../raw/current/scripts/recipes.lua#L544) |
| 雨衣 `raincoat` | 触手皮 2＋绳索 2＋犬骨 2 | `TECH.SCIENCE_ONE` | [配方](../../../raw/current/scripts/recipes.lua#L546) |
| 触手书 `book_tentacles` | 纸莎草 2＋触手皮 1 | `TECH.SCIENCE_THREE`、书匠 | [配方](../../../raw/current/scripts/recipes.lua#L283) |
| 鱼人瞭望塔 | 木板 5＋触手皮 1＋长矛 2 | `TECH.SCIENCE_ONE`、鱼人建造，限沼泽地皮 | [配方](../../../raw/current/scripts/recipes.lua#L109) |
| 升级贡品罐 | 木板 3＋芦苇 3＋触手皮 1 | 沃特技能 `wurt_civ_1_2` | [配方](../../../raw/current/scripts/recipes.lua#L119) |
| 升级鱼人军械库／工具棚 | 各含触手皮 1 | 沃特技能 `wurt_civ_3_2`／`wurt_civ_2_2` | [军械库](../../../raw/current/scripts/recipes.lua#L121)、[工具棚](../../../raw/current/scripts/recipes.lua#L123) |
| 弹弓触手皮筋 | 猫尾 1＋触手皮 1 | 沃尔特弹弓皮筋技能 | [配方](../../../raw/current/scripts/recipes.lua#L236) |

- **燃料用途要按秒算**：1 张皮入火堆只加 15 秒基础燃料，低于同族的 `MED_FUEL`（45 秒）与 `LARGE_FUEL`（180 秒）；给火堆续命时不能按"中等燃料"估。[常量](../../../raw/current/scripts/tuning.lua#L1673-L1678)、[入炉结算](../../../raw/current/scripts/components/fueled.lua#L209-L212)
- **没有其它加工路径**：无 `edible`／`perishable`／`cookable`／`dryable`，所以除了配方、燃料与燃烧，剩下的去向只有堆叠与被作祟抛飞。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`stackable` | 显式 40 | 主模拟 | [[reference/components/stackable]]；[赋值](../../../raw/current/scripts/prefabs/tentaclespots.lua#L30-L31) |
| 组件：`fuel` | `fuelvalue = SMALL_FUEL`＝15 秒，类型默认 `BURNABLE` | 主模拟；入炉时由 `fueled` 读值 | [[reference/components/fuel]]；[赋值](../../../raw/current/scripts/prefabs/tentaclespots.lua#L33-L34)、[默认类型](../../../raw/current/scripts/components/fuel.lua#L10-L14) |
| 组件：`burnable` | 7.5 秒烧完，掉同叠数灰烬 | 主模拟 | [[reference/components/burnable]]；[挂载](../../../raw/current/scripts/prefabs/tentaclespots.lua#L36) |
| 组件：`propagator` | 接受邻近热量、闪点点燃 | 主模拟 | [[reference/components/propagator]]；[挂载](../../../raw/current/scripts/prefabs/tentaclespots.lua#L37) |
| 组件：`hauntable` | 作祟只抛飞（Ignite 分支被注释） | 主模拟 | [[reference/components/hauntable]]；[挂载](../../../raw/current/scripts/prefabs/tentaclespots.lua#L38) |
| 宿主：`tentacle`／`tentacle_pillar` | 0.2／0.40 两条独立判定 | 服务端死亡 `DropLoot` | [触手](../../../raw/current/scripts/prefabs/tentacle.lua#L124-L125)、[大触手](../../../raw/current/scripts/prefabs/tentacle_pillar.lua#L369-L370) |
| 关联 Prefab：`tentaclespike` | 同表另一件产出，机制归尖刺页 | 与触手皮同表不同条目 | [[reference/prefabs/tentaclespike]]；[表](../../../raw/current/scripts/prefabs/tentacle.lua#L15-L21) |
| 关联 Prefab：`tree_rock*`（巨石枝系列） | 砍倒藤蔓战利品里按地形权重给触手皮 | 服务端砍倒时结算 | [[reference/prefabs/tree-rocks]]；[砍倒生成](../../../raw/current/scripts/prefabs/tree_rocks.lua#L615-L625) |
| 关联 Prefab：`whip` | 配方产物 | `TECH.SCIENCE_TWO` | [[reference/prefabs/whip]]；[配方](../../../raw/current/scripts/recipes.lua#L479) |
| 资源：`anim/tentaclespots.zip` | bank／build `tentaclespots` | 原版资源，不随页分发 | [资产](../../../raw/current/scripts/prefabs/tentaclespots.lua#L1-L4) |

## 常见陷阱与限制

- **误区：触手皮是"触手必掉"。** 它是共享概率表里的一条 0.2（大触手 0.40），与尖刺各自独立掷；同一次死亡可以两件都掉，也可以都不掉。[表](../../../raw/current/scripts/prefabs/tentacle.lua#L15-L21)、[掷点](../../../raw/current/scripts/components/lootdropper.lua#L213-L225)
- **边界：大触手的掉落从高空落下。** `DropLoot(Vector3(x, 20, z))` 会先落地再散开，拾取位置不是触手根部。[OnDeath](../../../raw/current/scripts/prefabs/tentacle_pillar.lua#L224-L230)
- **边界：1 张皮只有 15 秒燃料。** `SMALL_FUEL` 不是"小堆"的量词，而是 `seg_time × 0.5` 的秒数；还要再乘火堆倍率与湿燃料惩罚。[常量](../../../raw/current/scripts/tuning.lua#L1673-L1678)、[结算](../../../raw/current/scripts/components/fueled.lua#L205-L212)
- **边界：作祟不会点燃它。** 函数名带 `Ignite`，但点燃段在标准库里被注释，只剩抛飞。[注释段](../../../raw/current/scripts/standardcomponents.lua#L1204-L1213)
- **边界：烧掉一叠得到同叠数灰烬。** 不是"烧掉 1 个"，`DefaultBurntFn` 会把原叠数抄给灰烬。[回调](../../../raw/current/scripts/standardcomponents.lua#L31-L43)
- **未核验**：未运行游戏。0.2／0.40 与 15 秒都是源码常量与掷点推导；触手刷新密度与实战掉落频率未核验。

## 验证记录

- 源码审阅：2026-11-10，Steam buildid 26474873。逐行读 `tentaclespots.lua`（全 43 行）；`tentacle.lua` 与 `tentacle_pillar.lua` 的共享表、接线与死亡掉落；`merm.lua` 的月光追加表；`stackable`／`fuel`／`fueled`／`lootdropper`／`standardcomponents` 的燃料与燃烧函数；`recipes.lua` 的九条配方；`tuning.lua` 的堆叠、燃料与燃烧常量；`chinese_s.po` 名称键。
  未覆盖：触手与大触手的战斗、位移与刷新；鞭子等成品的机制（归各自页）；鱼人月变体的触发条件；巨石枝所在拓扑与地块分布。
- 游戏运行：未运行；概率、燃料秒数与叠数为源码推导。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 结果见交付回报。

## 相关链接

- 同表产物与配方成品：[[reference/prefabs/tentaclespike]]、[[reference/prefabs/whip]]
- 组件：[[reference/components/stackable]]、[[reference/components/fuel]]、[[reference/components/burnable]]
- 同批页面：[[reference/prefabs/honeycomb|蜜脾]]、[[reference/prefabs/houndstooth|犬牙]]、[[reference/prefabs/stinger|针刺]]、[[reference/prefabs/walrus-tusk|海象牙]]

返回 [[reference/prefabs/index|Prefab 索引]]。
