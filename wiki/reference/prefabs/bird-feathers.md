---
title: "四色鸟羽（feather_crow、feather_robin、feather_robin_winter、feather_canary）"
created: "2026-11-10"
updated: "2026-11-10"
type: prefab
summary: "一个工厂注册的四色鸟羽：鸦黑／红／蓝／金丝雀黄在属性上完全一致（40 叠、7.5 秒燃料、可装饰雪人），差异只落在掉落鸟种与配方消费上——16 条制作配方＋1 条拆解，表按颜色展开为 18 行；另有一个金值为 0 因而不能卖给猪王的 tradable 组件。"
aliases: ["feather_crow", "feather_robin", "feather_robin_winter", "feather_canary", "黑色羽毛", "红色羽毛", "蓝色羽毛", "黄色羽毛", "鸟羽", "鸟羽毛", "乌鸦羽毛", "金丝雀羽毛"]
tags: [prefab, item]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/feathers.lua", "prefabs/birds.lua", "prefabs/buzzard.lua", "prefabs/penguin.lua", "prefabs/tumbleweed.lua", "prefabs/leafpile.lua", "prefabs/catcoon.lua", "prefabs/yot_catcoonshrine.lua", "stategraphs/SGcanarypoisoned.lua", "components/lootdropper.lua", "components/fueled.lua", "components/tradable.lua", "components/snowmandecoratable.lua", "components/hauntable.lua", "components/floater.lua", "prefabs/pigking.lua", "standardcomponents.lua", "actions.lua", "brains/catcoonbrain.lua", "tuning.lua", "recipes.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
size_class: standard
---

# 四色鸟羽（feather_crow、feather_robin、feather_robin_winter、feather_canary）

## 快速判断

- 职责与适用：`prefabs/feathers.lua` 用同一个 `makefeather(name)` 注册四个物品，注册名是 `"feather_"..name`——`feather_crow`（黑色羽毛）、`feather_robin`（红色羽毛）、`feather_robin_winter`（蓝色羽毛）、`feather_canary`（黄色羽毛）。四者共用一条工厂链、参数逐项相同，差别只在资源名与配方消费者。[注册](../../../raw/current/scripts/prefabs/feathers.lua#L59-L62)
- 优先使用：查"哪只鸟掉哪种"看[来源](#来源谁掉什么)；查"能做什么"看[消费者与配方计数](#消费者与配方计数)；查堆叠／燃料／装饰／交易值看[四条属性](#四条属性逐项同值)；只想改数值看[典型接入](#典型接入)。
- 重要副作用：四注册都带 `cattoy`（可喂给驯服的浣猫）与 `birdfeather`（猫年神龛的供奉判定）；都可燃料、可燃烧、可装饰雪人；都挂了 `tradable` 但**从未写 `goldvalue`**，默认 0，猪王门槛要求 `goldvalue > 0`，所以实际卖不掉。[标签](../../../raw/current/scripts/prefabs/feathers.lua#L23-L24)、[交易组件](../../../raw/current/scripts/prefabs/feathers.lua#L50-L50)、[默认金值](../../../raw/current/scripts/components/tradable.lua#L3-L3)、[猪王门槛](../../../raw/current/scripts/prefabs/pigking.lua#L605-L605)
- 修改边界：鸟本体（刷新、捕鸟、毒金丝雀形态）归 `birds.lua` 与 [[reference/prefabs/canary-poisoned]]；吹箭归 [[reference/prefabs/ranged-weapons]]（S2）；羽毛扇与龙卷风法杖归 [[reference/prefabs/cooling-fans]]、[[reference/prefabs/staffs-more]]（S2）；羽毛笔、羽毛帽本体各有归属，本页只列配方行。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `feather_crow` | `makefeather("crow")` → `assetname = "feather_"..name` → `Prefab(assetname, fn, assets)` | bank／build／anim 全按 `feather_crow` 拼；组件链与其余三色完全相同。[工厂](../../../raw/current/scripts/prefabs/feathers.lua#L1-L6)、[注册](../../../raw/current/scripts/prefabs/feathers.lua#L56-L57) |
| `feather_robin` | `makefeather("robin")` | 同上，仅资源名不同。[注册](../../../raw/current/scripts/prefabs/feathers.lua#L59-L60) |
| `feather_robin_winter` | `makefeather("robin_winter")` | 同上；名字里带下划线，bank 也是 `feather_robin_winter`。[注册](../../../raw/current/scripts/prefabs/feathers.lua#L61-L61) |
| `feather_canary` | `makefeather("canary")` | 同上；掉落权重只有 0.1，见[来源](#来源谁掉什么)。[注册](../../../raw/current/scripts/prefabs/feathers.lua#L62-L62) |

共用构造顺序（服务端）：实体→`MakeInventoryPhysics`→`SetBank`／`SetBuild`／播 `idle`→`pickupsound = "cloth"`→打 `cattoy`／`birdfeather`→`MakeInventoryFloatable(inst, "small", 0.05, 0.95)`→`SetPristine()`；主模拟再加 `stackable`（40）→`inspectable`→`MakeSmallBurnable(SMALL_BURNTIME)`→`MakeSmallPropagator`→`fuel`（`TINY_FUEL`）→`MakeHauntableLaunchAndIgnite`→`inventoryitem`（`nobounce`）→`tradable`→`snowmandecor`。[非主模拟段](../../../raw/current/scripts/prefabs/feathers.lua#L17-L32)、[主模拟段](../../../raw/current/scripts/prefabs/feathers.lua#L34-L54)

### 四条属性逐项同值

四个注册之间**没有任何数值差异**——`makefeather` 里没有一处读 `name` 去做分支（`name` 只用于拼资源名）。下面四项因此对四色同时成立：

| 属性 | 取值 | 定义处 | 结算处与条件 |
|---|---|---|---|
| 堆叠 | 40 | `maxsize = TUNING.STACK_SIZE_SMALLITEM`＝40。[定义](../../../raw/current/scripts/prefabs/feathers.lua#L34-L35)、[常量](../../../raw/current/scripts/tuning.lua#L83-L83) | `stackable.maxsize`，与同族 20 叠的麋鹿鹅／邪天翁羽毛不同 |
| 燃料 | 7.5 秒 | `TINY_FUEL = seg_time*.25`，`seg_time`＝30 秒。[定义](../../../raw/current/scripts/prefabs/feathers.lua#L42-L43)、[常量与基准](../../../raw/current/scripts/tuning.lua#L19-L19)、[常量](../../../raw/current/scripts/tuning.lua#L1673-L1673) | `fuelvalue = fuel.fuelvalue * bonusmult * mult * wetmult * masterymult`；湿的羽毛再乘 `WET_FUEL_PENALTY`＝0.75。[结算](../../../raw/current/scripts/components/fueled.lua#L197-L212)、[湿惩罚](../../../raw/current/scripts/tuning.lua#L1671-L1671) |
| 燃烧 | 7.5 秒 | `SMALL_BURNTIME = seg_time*.25`。[接线](../../../raw/current/scripts/prefabs/feathers.lua#L39-L40)、[常量](../../../raw/current/scripts/tuning.lua#L1681-L1681) | `MakeSmallBurnable` 把它写进 `burnable:SetBurnTime`。[helper](../../../raw/current/scripts/standardcomponents.lua#L164-L171) |
| 漂浮 | `small`／偏移 0.05／缩放 0.95 | [接线](../../../raw/current/scripts/prefabs/feathers.lua#L26-L26) | `floater:SetSize`／`SetVerticalOffset`／`SetScale`。[helper](../../../raw/current/scripts/standardcomponents.lua#L1772-L1782) |

名称核验（S1）：`NAMES.FEATHER_CROW`＝黑色羽毛、`FEATHER_ROBIN`＝红色羽毛、`FEATHER_ROBIN_WINTER`＝蓝色羽毛、`FEATHER_CANARY`＝黄色羽毛；四个 prefab 都没有 `SetPrefabName`／`nameoverride`，注册名与显示名一一对应。[CROW](../../../raw/current/scripts/languages/chinese_s.po#L342674-L342677)、[ROBIN](../../../raw/current/scripts/languages/chinese_s.po#L342679-L342682)、[ROBIN_WINTER](../../../raw/current/scripts/languages/chinese_s.po#L342684-L342687)、[CANARY](../../../raw/current/scripts/languages/chinese_s.po#L342669-L342672)

## 来源：谁掉什么

鸟类的掉落走 `lootdropper` 的**随机表**：`AddRandomLoot` 追加权重，`numrandomloot` 决定抽几次，`GenerateLoot` 里 `PickRandomLoot` 在 `math.random() * 总权重` 上做一次加权落点——所以权重比就是概率比，抽 1 次即"只出其中一件"。[抽表](../../../raw/current/scripts/components/lootdropper.lua#L99-L112)、[结算](../../../raw/current/scripts/components/lootdropper.lua#L186-L200)

| 宿主 | 接线 | 实际结果与条件 |
|---|---|---|
| 乌鸦 `crow`、红雀 `robin`、雪雀 `robin_winter` | `AddRandomLoot("feather_"..name, 1)` ＋ `AddRandomLoot("smallmeat", 1)`，`numrandomloot = 1` | 每次死亡只抽 1 件：羽毛 1/2＝**50%**，否则小块肉。[接线](../../../raw/current/scripts/prefabs/birds.lua#L319-L328)、[注册](../../../raw/current/scripts/prefabs/birds.lua#L436-L439) |
| 金丝雀 `canary` | 同一段代码，但羽毛权重写死 `.1` | 0.1/(0.1+1)＝**1/11≈9.09%**，不是"10%"；剩下的 10/11 出小块肉。[权重](../../../raw/current/scripts/prefabs/birds.lua#L324-L324) |
| 中毒金丝雀 `canary_poisoned` | 状态机直接 `SpawnLootPrefab("feather_canary")` | 爆毒掉 `math.random(2)`＝**1–2 根**；复原掉 `4 + math.random(2)`＝**5–6 根**。形态与条件见 [[reference/prefabs/canary-poisoned]]。[爆毒](../../../raw/current/scripts/stategraphs/SGcanarypoisoned.lua#L256-L266)、[复原](../../../raw/current/scripts/stategraphs/SGcanarypoisoned.lua#L348-L351) |
| 秃鹫 `buzzard` | `SetChanceLootTable('buzzard')`：`feather_crow` 0.33 为**独立判定** | 33%；同表 `drumstick` 1.00 必掉、`smallmeat` 0.33 独立，三者可同时出。月光秃鹫的 `buzzard_lunar` 表把这一行**注释掉了**，不掉羽毛。[表](../../../raw/current/scripts/prefabs/buzzard.lua#L19-L24)、[月光表](../../../raw/current/scripts/prefabs/buzzard.lua#L218-L223) |
| 企鹅 `penguin` | `SetChanceLootTable('penguin')`：`feather_crow` 0.2 | 每次死亡独立 20%，与 `smallmeat` 0.1、`drumstick` 0.1 各自判定。[表](../../../raw/current/scripts/prefabs/penguin.lua#L36-L41) |
| 海鹦 `puffin` | `no_feather = true` ＋ 自定义 `puffin_loot_setup` | 名字是 puffin，掉的却是 `feather_crow`（权重 1＋小块肉 1，抽 1＝50%）。[定制表](../../../raw/current/scripts/prefabs/birds.lua#L430-L434)、[注册](../../../raw/current/scripts/prefabs/birds.lua#L441-L441) |
| 风滚草 `tumbleweed` | 四色各一行 `chance = 0.33`，从全表权重做 **3 次有放回**抽取 | 每次抽取命中某色的概率＝`0.33 / 权重总和`（其余条目权重在 0.01～40 之间，总和随解锁态变化）。[权重行](../../../raw/current/scripts/prefabs/tumbleweed.lua#L166-L169)、[抽取](../../../raw/current/scripts/prefabs/tumbleweed.lua#L213-L245) |
| 落叶堆 `leafpile` | `DEBRIS` 表里 `feather_crow` 权重 0.1（该表总权重 0.9）；抽取次数从 1 起，叶堆每高一级再用 `LEAFPILE_ADDITIONAL_LOOT_PER_STAGE_CHANCE` 各试加 1 次，TRICK 分支固定 1 次 | 每一次抽命中黑羽的概率都是 0.1/0.9＝**1/9≈11.1%**；每抽都回到同一张表、且不移除上次选项，所以同一次开堆可出两根。已经有两次抽取时，至少一根＝1−(8/9)²≈**21.0%**（推导）。TREAT 分支不含羽毛。[表](../../../raw/current/scripts/prefabs/leafpile.lua#L97-L108)、[抽次数](../../../raw/current/scripts/prefabs/leafpile.lua#L174-L193)、[抽取循环](../../../raw/current/scripts/prefabs/leafpile.lua#L195-L211) |
| 浣猫 `catcoon` | `friendGiftPrefabs` 第 3 层列四色羽毛 | 这是浣猫**吐出的礼物**表，不是它掉落的；投喂绒球换礼物的机制不在本页。[礼物表](../../../raw/current/scripts/prefabs/catcoon.lua#L140-L144) |

## 消费者与配方计数

按四色原料名查 `recipes.lua`，只有 **16 条不同的 `Recipe2`** 和 **1 条 `DeconstructRecipe`**。下表按颜色展开成 18 行：多出的一行是 `featherhat`——同一条配方同时吃黑羽 3 与红羽 2，按颜色拆两次；最后一行 `yoth_knighthat` 是**拆解返还**红羽，方向与原料消费相反，不是制作材料。材料名与数量逐条取自 `recipes.lua`，不按记忆补。

| 羽毛 | 配方（注册名） | 用量与科技 | 证据 |
|---|---|---|---|
| `feather_crow` | `featherpencil` | 1（＋树枝 1、木炭 1），`TECH.SCIENCE_ONE` | [L409](../../../raw/current/scripts/recipes.lua#L409-L409) |
| `feather_crow` | `blowdart_sleep` | 1（＋芦苇 2、针刺 1），`TECH.SCIENCE_TWO` | [L488](../../../raw/current/scripts/recipes.lua#L488-L488) |
| `feather_crow` | `featherhat` | **3**（＋红色羽毛 2、触手皮 2），`TECH.SCIENCE_TWO` | [L544](../../../raw/current/scripts/recipes.lua#L544-L544) |
| `feather_crow` | `saddlehorn` | 1（＋树枝 2、骨片 2），`TECH.SCIENCE_TWO` | [L681](../../../raw/current/scripts/recipes.lua#L681-L681) |
| `feather_crow` | `oceanfishingbobber_crow` | 1，`TECH.FISHING_ONE` | [L647](../../../raw/current/scripts/recipes.lua#L647-L647) |
| `feather_robin` | `blowdart_fire` | 1（＋芦苇 2、木炭 1），`TECH.SCIENCE_TWO` | [L486](../../../raw/current/scripts/recipes.lua#L486-L486) |
| `feather_robin` | `featherhat` | 2 | [L544](../../../raw/current/scripts/recipes.lua#L544-L544) |
| `feather_robin` | `reflectivevest` | **3**（＋绳索 1、猪皮 2），`TECH.SCIENCE_ONE` | [L550](../../../raw/current/scripts/recipes.lua#L550-L550) |
| `feather_robin` | `oceanfishingbobber_robin` | 1 | [L648](../../../raw/current/scripts/recipes.lua#L648-L648) |
| `feather_robin` | `yoth_knighthat` 拆解 | 拆解返还 1 根（不是制作材料） | [L1741](../../../raw/current/scripts/recipes.lua#L1741-L1741) |
| `feather_robin_winter` | `blowdart_pipe` | 1（＋芦苇 2、犬牙 1），`TECH.SCIENCE_TWO` | [L485](../../../raw/current/scripts/recipes.lua#L485-L485) |
| `feather_robin_winter` | `saddle_wathgrithr` | **6**（＋金块 4、大理石 3），需技能 `wathgrithr_beefalo_saddle` | [L194](../../../raw/current/scripts/recipes.lua#L194-L194) |
| `feather_robin_winter` | `battlesong_container` | 2（＋木板 2、金块 3、蜂蜡 2），需技能 | [L195](../../../raw/current/scripts/recipes.lua#L195-L195) |
| `feather_robin_winter` | `oceanfishingbobber_robin_winter` | 1 | [L649](../../../raw/current/scripts/recipes.lua#L649-L649) |
| `feather_canary` | `blowdart_yellow` | 1（＋芦苇 2、金块 1），`TECH.SCIENCE_TWO` | [L487](../../../raw/current/scripts/recipes.lua#L487-L487) |
| `feather_canary` | `slingshotammo_scrapfeather` | 1（＋废料零件 1），一次产出 20 发，需沃尔特技能 | [L223](../../../raw/current/scripts/recipes.lua#L223-L223) |
| `feather_canary` | `polly_rogershat` | 1（＋猴子帽 1、黑旗 1），`TECH.LOST` | [L558](../../../raw/current/scripts/recipes.lua#L558-L558) |
| `feather_canary` | `oceanfishingbobber_canary` | 1 | [L650](../../../raw/current/scripts/recipes.lua#L650-L650) |

四条吹箭的完整消费者链路（伤害、弹道、命中效果）在 [[reference/prefabs/ranged-weapons]]（S2）；羽毛扇／龙卷风法杖在 [[reference/prefabs/cooling-fans]]、[[reference/prefabs/staffs-more]]（S2）——它们吃的是 `goose_feather`，不是这四色。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`stackable` | `maxsize = 40` | 主模拟；四色一致 | [接线](../../../raw/current/scripts/prefabs/feathers.lua#L34-L35) |
| 组件：`fuel` | `fuelvalue = TINY_FUEL`（7.5 秒） | 主模拟；`ADDFUEL` 交给 `fueled:TakeFuelItem` 结算，湿件 ×0.75 | [接线](../../../raw/current/scripts/prefabs/feathers.lua#L42-L43)、[动作](../../../raw/current/scripts/actions.lua#L2188-L2194) |
| 组件：`burnable` | `MakeSmallBurnable(inst, SMALL_BURNTIME)`，烧完变灰 | 主模拟 | [接线](../../../raw/current/scripts/prefabs/feathers.lua#L39-L40)、[helper](../../../raw/current/scripts/standardcomponents.lua#L164-L171) |
| 组件：`propagator` | `MakeSmallPropagator`：`flashpoint` 5–10、`propagaterange` 3–5、`heatoutput` 3–5 | 主模拟；靠近火源会被点燃 | [接线](../../../raw/current/scripts/prefabs/feathers.lua#L40-L40)、[helper](../../../raw/current/scripts/standardcomponents.lua#L200-L212) |
| 组件：`hauntable` | `MakeHauntableLaunchAndIgnite`，但点火分支整段被 `#HAUNTFIX` 注释掉 | 主模拟；实际只有"抛飞"一条 | [接线](../../../raw/current/scripts/prefabs/feathers.lua#L45-L45)、[实现](../../../raw/current/scripts/standardcomponents.lua#L1191-L1216) |
| 组件：`inventoryitem` | `nobounce = true` | 主模拟 | [接线](../../../raw/current/scripts/prefabs/feathers.lua#L47-L48) |
| 组件：`tradable` | 只 `AddComponent`，未设 `goldvalue` → 0 | 主模拟；猪王判定要求 > 0，故不可交易 | [接线](../../../raw/current/scripts/prefabs/feathers.lua#L50-L50)、[门槛](../../../raw/current/scripts/prefabs/pigking.lua#L605-L605) |
| 组件：`snowmandecor` | 仅标记组件；真正的动画数据在 `ITEM_DATA` | 主模拟；装饰时按 `hash(prefab)` 查表并从背包扣件 | [接线](../../../raw/current/scripts/prefabs/feathers.lua#L52-L52)、[数据](../../../raw/current/scripts/components/snowmandecoratable.lua#L18-L22)、[建表](../../../raw/current/scripts/components/snowmandecoratable.lua#L44-L49)、[扣件](../../../raw/current/scripts/components/snowmandecoratable.lua#L258-L264) |
| 组件：`floater` | `small`／0.05／0.95 | 主模拟；落水时加成湿度与降温由 `OnLandedServer` 处理 | [接线](../../../raw/current/scripts/prefabs/feathers.lua#L26-L26)、[落地](../../../raw/current/scripts/components/floater.lua#L146-L158) |
| 标签：`cattoy` | 浣猫接受带此标签的物品作玩具 | 两端均有；喂食换礼物 | [接受判定](../../../raw/current/scripts/prefabs/catcoon.lua#L273-L279)、[行为树](../../../raw/current/scripts/brains/catcoonbrain.lua#L44-L44) |
| 标签：`birdfeather` | 猫年神龛只收带此标签的供奉品 | 两端均有；全库仅此一处消费 | [供奉判定](../../../raw/current/scripts/prefabs/yot_catcoonshrine.lua#L85-L87) |
| 关联 Prefab：`featherpencil`／`blowdart_*`／`featherhat`／`oceanfishingbobber_*` | **16 条制作配方**消费这四色羽毛，另 **1 条拆解**（`yoth_knighthat`）**返还**红羽；表因羽毛帽一行吃两色而展开为 18 行 | 配方见上表 | [[reference/prefabs/ranged-weapons]]、[[reference/prefabs/hats-event]]、[拆解](../../../raw/current/scripts/recipes.lua#L1741-L1741) |
| 资源：`anim/feather_<name>.zip` | 每条注册只带自己那份动画包 | 原版资源，不随页分发 | [assets](../../../raw/current/scripts/prefabs/feathers.lua#L3-L6) |

## 典型接入

### 给四色羽毛补一个可交易金值

- 前提与选择：服务端 `modmain.lua` 的 postinit。原版四个 prefab 都挂了 `tradable` 却没写 `goldvalue`，默认 0 让猪王门槛落空；直接补这个字段比重写 prefab 更小。[默认值](../../../raw/current/scripts/components/tradable.lua#L3-L3)
- 生效与撤销：只作用于 postinit 时构造的实体——新生成与读档重建的羽毛都会走到；热重载前已存在的实体不会重跑，移除模组后新生成的羽毛回到 0。
- 原版依据：`pigking` 的接受判定是 `item.components.tradable.goldvalue > 0`，金值同时决定给出的金块数目循环次数。[接受](../../../raw/current/scripts/prefabs/pigking.lua#L605-L605)、[给金](../../../raw/current/scripts/prefabs/pigking.lua#L74-L74)

```lua
-- modmain.lua：四色羽毛原版 tradable.goldvalue 为 0，这里补 1 让猪王收
for _, prefab in ipairs({ "feather_crow", "feather_robin", "feather_robin_winter", "feather_canary" }) do
    AddPrefabPostInit(prefab, function(inst)
        if GLOBAL.TheWorld.ismastersim and inst.components.tradable ~= nil then
            inst.components.tradable.goldvalue = 1
        end
    end)
end
```

- 调整与验证：金值同时影响猪王给的金块数，想只"能卖"就给 1。本例未运行，只核对了 `tradable` 与 `pigking` 的判定顺序；实机需要把羽毛递给猪王看是否收下。

## 常见陷阱与限制

- **误区：四色羽毛属性不同。** 四个注册走同一个 `makefeather`，没有任何按 `name` 的分支；堆叠、燃料、燃烧时间、漂浮参数全部相同，差异只在资源名与配方。[工厂](../../../raw/current/scripts/prefabs/feathers.lua#L1-L57)
- **误区：挂了 `tradable` 就能卖给猪王。** `goldvalue` 默认 0，猪王要求 > 0；本页四个 prefab 都没有赋值。[默认](../../../raw/current/scripts/components/tradable.lua#L3-L3)、[判定](../../../raw/current/scripts/prefabs/pigking.lua#L605-L605)
- **误区：`MakeHauntableLaunchAndIgnite` 会让作祟点燃羽毛。** 这个 build 里点火段被 `#HAUNTFIX` 注释掉，只剩抛飞；抛飞结果与 `MakeHauntableLaunch` 相同，但那段注释把 `return true` 一起注掉了，回调恒返回 false，于是 `DoHaunt` 走失败分支——`haunted` 标志（连带 `haunted` 标签）与 shader／FX 仍由 else 分支置真，但不派发 `haunted` 事件、也不清 `hauntvalue`（`MakeHauntableLaunch` 成功时返回 true 并派发）。[被注掉的 return](../../../raw/current/scripts/standardcomponents.lua#L1213-L1215)、[成功分支](../../../raw/current/scripts/components/hauntable.lua#L88-L98)、[else 分支](../../../raw/current/scripts/components/hauntable.lua#L106-L112)、[对照](../../../raw/current/scripts/standardcomponents.lua#L1053-L1069)
- **边界：金丝雀羽毛不是 10%。** 随机表是"羽毛 0.1＋小块肉 1，抽 1 次"，权重归一后 1/11≈9.09%；`.1` 只是权重而不是概率。[权重](../../../raw/current/scripts/prefabs/birds.lua#L324-L324)、[抽表](../../../raw/current/scripts/components/lootdropper.lua#L99-L112)
- **边界：乌鸦羽毛不只来自乌鸦。** 秃鹫 0.33、企鹅 0.2、海鹦 50%（它自己的表却掉 `feather_crow`）、风滚草与落叶堆也在表内。[秃鹫](../../../raw/current/scripts/prefabs/buzzard.lua#L19-L24)、[企鹅](../../../raw/current/scripts/prefabs/penguin.lua#L36-L41)、[海鹦](../../../raw/current/scripts/prefabs/birds.lua#L430-L434)
- **边界：月光变体的鸟不掉羽毛。** 秃鹫的 `buzzard_lunar` 表把 `feather_crow` 一行注释掉了。[表](../../../raw/current/scripts/prefabs/buzzard.lua#L218-L223)
- **边界：羽毛不是食物、不腐坏、不可装备。** 组件链里没有 `edible`／`perishable`／`equippable`，标签也只有 `cattoy`／`birdfeather`，不带任何 foodtype。[组件链](../../../raw/current/scripts/prefabs/feathers.lua#L34-L54)、[标签](../../../raw/current/scripts/prefabs/feathers.lua#L23-L24)
- **未核验**：未运行游戏。浣猫把羽毛当玩具时的具体动画与冷却、雪人装饰界面在客户端的显示与扣件顺序，只按源码推导，未实测；tumbleweed 的权重总和随存档解锁态变化，未给单一概率。

## 验证记录

- 源码审阅：2026-11-10，Steam buildid 26474873。通读 `prefabs/feathers.lua` 全文（四注册、共用工厂、组件顺序）；`prefabs/birds.lua` 的 `makebird` 掉落段与海鹦定制表；`buzzard.lua`／`penguin.lua` 的两张 `SetChanceLootTable`；`tumbleweed.lua` 的权重表与 3 次抽取；`leafpile.lua` 的 DEBRIS 表与分支选择；`catcoon.lua` 的 `cattoy` 接受判定与礼物表；`yot_catcoonshrine.lua` 的 `birdfeather` 判定；`SGcanarypoisoned.lua` 的爆毒／复原掉落；`lootdropper.lua` 的 `PickRandomLoot`／`GenerateLoot`；`fueled.lua` 的 `TakeFuelItem`；`tradable.lua` 默认值与 `pigking.lua` 门槛；`snowmandecoratable.lua` 的 `ITEM_DATA` 与扣件；`standardcomponents.lua` 的四个 helper；`tuning.lua` 的 `STACK_SIZE_SMALLITEM`／`TINY_FUEL`／`SMALL_BURNTIME`／`WET_FUEL_PENALTY`；`recipes.lua` 的 16 条制作＋1 条拆解（按颜色展开 18 行）；`chinese_s.po` 四色名称。
  检索范围：`feather_` 与四个注册名的定向检索，覆盖 `recipes.lua`、`birds.lua`、`buzzard.lua`、`penguin.lua`、`tumbleweed.lua`、`leafpile.lua`、`catcoon.lua`、`yot_catcoonshrine.lua`、`SGcanarypoisoned.lua`、`snowmandecoratable.lua`；未覆盖：其它模组注入路径、装饰界面的客户端实现、浣猫礼物的层级概率。
- 游戏运行：未运行。所有百分比为源码权重推导（`math.random()` 端点与权重和），天数换算基准 `total_day_time`＝480 秒、`seg_time`＝30 秒。
- 复审修订：2026-11-10，R2 条目：P3-3（17 制作＋1 拆解）、P3-4（失败分支置 haunted 但不派发事件、不清 hauntvalue）、P3-6（补 4 个被引文件）、P3-7（补 `size_class: standard`）。
- 外部复检修订：2026-11-10，R05（16 制作＋1 拆解、颜色展开 18 行）、R06（落叶堆按抽次数给概率）。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 见交付回报。

## 相关链接

- 吹箭与弹弓弹药：[[reference/prefabs/ranged-weapons]]
- 羽毛扇：[[reference/prefabs/cooling-fans]]；龙卷风法杖：[[reference/prefabs/staffs-more]]
- 中毒金丝雀的羽毛掉落：[[reference/prefabs/canary-poisoned]]
- 羽毛帽：[[reference/prefabs/hats-event]]
- 同批：[[reference/prefabs/goose-feather]]、[[reference/prefabs/malbatross-feather]]

返回 [[reference/prefabs/index|Prefab 索引]]。
