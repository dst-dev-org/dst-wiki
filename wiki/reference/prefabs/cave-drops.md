---
title: 洞穴掉落：蘑菇皮（shroom_skin）与啜食者皮（slurper_pelt）
created: "2026-11-10"
updated: "2026-11-10"
type: prefab
summary: 两件洞穴生物产物的收集口径：蘑菇皮堆叠 10、不可食不可燃，靠毒菌蟾蜍按血量阈值 70%／40% 的阶段抛出（悲惨版另有 20%）与击杀表（普通 1 张、悲惨 2 张）获得；啜食者皮堆叠 40、edible 只设 HORRIBLE（三维仍是默认 10/10/0）、交易金值 1、可燃且烧尽成灰，来源是啜食者基础 0.5 掉落（仅正向幸运提高）。本页同时写清蘑菇灯／睡袋炸弹／饥饿腰带／WX-78 饥饿模块四道配方与蓝图门禁。
aliases: ["shroom_skin", "蘑菇皮", "Shroom Skin", "slurper_pelt", "啜食者皮", "Slurper Pelt", "slurperpelt", "洞穴生物掉落"]
tags: [prefab, item]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/shroom_skin.lua", "prefabs/slurperpelt.lua", "prefabs/toadstool.lua", "prefabs/slurper.lua", "prefabs/corpses_defs.lua", "prefabs/blueprint.lua", "componentutil.lua", "prefabs/pigking.lua", "prefabs/pigman.lua", "prefabs/sketch.lua", "prefabs/lightbulb.lua", "prefabs/ash.lua", "components/edible.lua", "components/eater.lua", "components/tradable.lua", "components/stackable.lua", "components/burnable.lua", "components/healthtrigger.lua", "components/health.lua", "components/lootdropper.lua", "components/teacher.lua", "components/builder.lua", "stategraphs/SGslurper.lua", "standardcomponents.lua", "physics.lua", "cooking.lua", "containers.lua", "recipes.lua", "constants.lua", "tuning.lua", "wx78_moduledefs.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 洞穴掉落：蘑菇皮（shroom_skin）与啜食者皮（slurper_pelt）

## 快速判断

- 职责与适用：两件洞穴生物的产物，各自单文件注册、没有共享工厂。蘑菇皮（`shroom_skin`）是毒菌蟾蜍的产物——阶段切换时抛出的实物＋击杀表里的必掉条目；啜食者皮（`slurper_pelt`）是啜食者的基础 0.5 掉落，自身有 `edible`（HORRIBLE）、`tradable`（1 金块）与可燃。[蘑菇皮注册](../../../raw/current/scripts/prefabs/shroom_skin.lua#L39-L39)、[啜食者皮注册](../../../raw/current/scripts/prefabs/slurperpelt.lua#L46-L46)
- 优先使用：查蘑菇皮怎么来（阶段抛出与击杀表分开）看[获得：蘑菇皮](#获得蘑菇皮毒菌蟾蜍)；查啜食者皮掉落与食性／交易看[获得：啜食者皮](#获得啜食者皮啜食者)；查它们能造什么看[用途与配方](#用途与配方)。
- 重要副作用：阶段抛出的蘑菇皮由 `SpawnPrefab`＋`LaunchAt` 直接生成，**不经过 lootdropper**，所以燃烧状态不影响它；击杀表里的蘑菇皮只在**普通燃烧**死亡时被改写成灰烬，受控燃烧（Willow 控火）下保留原物。啜食者皮自己可被点燃，烧尽时整叠转成等量灰烬。[抛出](../../../raw/current/scripts/prefabs/toadstool.lua#L695-L698)、[改写](../../../raw/current/scripts/components/lootdropper.lua#L415-L445)
- 修改边界：本页不覆盖毒菌蟾蜍的战斗（孢子炸弹、蘑菇炸弹、砸地）与啜食者的行为／活体帽子；蘑菇灯、睡袋炸弹、饥饿腰带、WX-78 模块的本体各自归已有页或待建页。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `shroom_skin` | 单文件单注册 `Prefab("shroom_skin", fn, assets)`。公共端：Transform／AnimState／SoundEmitter／Network → `MakeInventoryPhysics` → bank／build `shroom_skin` → `idle` → `MakeInventoryFloatable(inst, "large", nil, 0.6)` → `SetPristine`。 | 主模拟只有 4 项：`inspectable`、`inventoryitem`、`stackable`（`STACK_SIZE_LARGEITEM`）、`MakeHauntableLaunch`。没有 `edible`／`fuel`／`burnable`／`tradable`。[fn](../../../raw/current/scripts/prefabs/shroom_skin.lua#L6-L37) |
| `slurper_pelt` | 单文件单注册，**文件名是 `slurperpelt.lua`，注册名是 `slurper_pelt`**。公共端：Transform／AnimState／Network → `MakeInventoryPhysics` → bank／build `slurper_pelt` → `idle` → `MakeInventoryFloatable(inst)` → `SetPristine`。 | 主模拟：`stackable`（`STACK_SIZE_SMALLITEM`）、`inspectable`、`MakeSmallBurnable`＋`MakeSmallPropagator`、`tradable`（`GOLD_VALUES.MEAT`）、`inventoryitem`、`edible`（只设 `foodtype = FOODTYPE.HORRIBLE`）。没有 `hauntable`。[fn](../../../raw/current/scripts/prefabs/slurperpelt.lua#L6-L44) |

### 名称与实显

- `STRINGS.NAMES.SHROOM_SKIN`＝**蘑菇皮**（英文 Shroom Skin）。[名称](../../../raw/current/scripts/languages/chinese_s.po#L349354-L349357)
- `STRINGS.NAMES.SLURPER_PELT`＝**啜食者皮**（英文 Slurper Pelt）。[名称](../../../raw/current/scripts/languages/chinese_s.po#L349634-L349637)

两件物品都没有 `nameoverride`，物品栏与检视按注册名取 `NAMES` 键。台账／候选清单里写的 `slurperpelt` 是文件名，**不是注册名**；检索时两个写法都会命中，正文与 aliases 以 `slurper_pelt` 为准。

## 数值与参数

| 项 | 蘑菇皮 | 啜食者皮 | 证据 |
|---|---|---|---|
| 堆叠上限 | 10（`STACK_SIZE_LARGEITEM`） | 40（`STACK_SIZE_SMALLITEM`） | [蘑菇皮](../../../raw/current/scripts/prefabs/shroom_skin.lua#L31-L32)、[皮](../../../raw/current/scripts/prefabs/slurperpelt.lua#L27-L28)、[常量](../../../raw/current/scripts/tuning.lua#L81-L83) |
| 可食 | 无 | 有：HORRIBLE，三维取默认 10／10／0 | [皮](../../../raw/current/scripts/prefabs/slurperpelt.lua#L40-L41)、[默认值](../../../raw/current/scripts/components/edible.lua#L19-L23) |
| 交易金值 | 无 `tradable`（`GIVE` 动作不出现） | 1（`GOLD_VALUES.MEAT`） | [皮](../../../raw/current/scripts/prefabs/slurperpelt.lua#L35-L36)、[常量](../../../raw/current/scripts/tuning.lua#L1946-L1952) |
| 可燃 | 无 | `MakeSmallBurnable`（默认 10 秒）＋`MakeSmallPropagator`；烧尽转灰烬 | [皮](../../../raw/current/scripts/prefabs/slurperpelt.lua#L32-L33)、[烧尽](../../../raw/current/scripts/standardcomponents.lua#L164-L174) |
| 作祟 | `MakeHauntableLaunch`（只弹飞） | 没有 `hauntable`，不能被作祟 | [蘑菇皮](../../../raw/current/scripts/prefabs/shroom_skin.lua#L34-L34)、[全文](../../../raw/current/scripts/prefabs/slurperpelt.lua#L6-L44) |
| 浮水 | `large`、缩放 0.6 | 无参 → `small` | [蘑菇皮](../../../raw/current/scripts/prefabs/shroom_skin.lua#L20-L20)、[皮](../../../raw/current/scripts/prefabs/slurperpelt.lua#L19-L19) |

### edible 默认值与 HORRIBLE 食性

`components/edible.lua` 构造时就把 `healthvalue = 10`、`hungervalue = 10`、`sanityvalue = 0` 写死；`slurper_pelt` 只改 `foodtype`，所以吃下它的实际基础值是 **10 生命／10 饥饿／0 理智**，且它没有 `perishable`，不会因腐败再打折。[构造默认值](../../../raw/current/scripts/components/edible.lua#L19-L23)、[三个 getter](../../../raw/current/scripts/components/edible.lua#L72-L143)

“有 `edible`”不等于“玩家能吃”：食用资格由食用者侧的 `caneat` 决定，`Eater:TestFood` 只匹配物品上的 `edible_<类型>` 标签。玩家 `eater` 的默认 `caneat` 是 `{ FOODGROUP.OMNI }`，而 `OMNI` 只含 MEAT／VEGGIE／INSECT／SEEDS／GENERIC／GOODIES 六类，**不含 HORRIBLE**；因此默认玩家吃不到它。把 HORRIBLE 放进 diet 的生物（猪人、蜘蛛、猎犬等）能吃，喂猪人时 HORRIBLE 与 MEAT 走同一分支：保持跟随并按饥饿值折算忠诚时间。[默认食性](../../../raw/current/scripts/components/eater.lua#L18-L24)、[判定](../../../raw/current/scripts/components/eater.lua#L288-L302)、[OMNI 组成](../../../raw/current/scripts/constants.lua#L2051-L2062)、[猪人收食](../../../raw/current/scripts/prefabs/pigman.lua#L62-L78)、[猪人食性](../../../raw/current/scripts/prefabs/pigman.lua#L746-L746)、[跟随与忠诚](../../../raw/current/scripts/prefabs/pigman.lua#L98-L111)

### 交易金值

`slurper_pelt` 的 `tradable.goldvalue = TUNING.GOLD_VALUES.MEAT`，常量表里 `MEAT = 1`。猪王收到 `goldvalue > 0` 的物品时进入 `cointoss`，按 `goldvalue` 的次数各生成一个金块；`GIVE` 动作本身只对带 `tradable` 的物品注册，所以蘑菇皮这类无 `tradable` 的物品根本不会出现交付选项。[设置](../../../raw/current/scripts/prefabs/slurperpelt.lua#L35-L36)、[金值表](../../../raw/current/scripts/tuning.lua#L1946-L1952)、[发放](../../../raw/current/scripts/prefabs/pigking.lua#L59-L78)、[接受判定](../../../raw/current/scripts/prefabs/pigking.lua#L598-L606)、[GIVE 注册](../../../raw/current/scripts/componentactions.lua#L1860-L1868)

### 可燃与烧毁

`MakeSmallBurnable(inst)` 给啜食者皮挂 `burnable`：默认燃烧 10 秒、点燃时 `persists = false`；烧尽时走 `DefaultBurntFn`——在位置生成一个 `ash`（叠数取 `min(灰烬上限 40, 原叠数)`）再移除本体，海上位置不生成。它不是“烧掉耐久”而是整件消失；也没有 `fuel`，不能当燃料添进火堆。[工厂](../../../raw/current/scripts/standardcomponents.lua#L164-L174)、[烧尽](../../../raw/current/scripts/standardcomponents.lua#L18-L44)、[灰烬上限](../../../raw/current/scripts/prefabs/ash.lua#L70-L70)

## 获得：蘑菇皮（毒菌蟾蜍）

### 阶段抛出（与击杀表分开）

毒菌蟾蜍用 `healthtrigger` 按血量百分比登记回调；`healthtrigger` 只在一次 `healthdelta` **跨过**阈值时触发，同一击跨多个阈值时按由高到低依次回调。回调里再检查当前血量，决定这一档是否真的抛出：

| 形态 | 阈值 | 回调与附加条件 | 结果 | 证据 |
|---|---|---|---|---|
| 毒菌蟾蜍（普通） | 70% | `EnterPhase2Trigger`：当前血量仍 > 40% | 抛 1 张 | [触发](../../../raw/current/scripts/prefabs/toadstool.lua#L700-L708)、[登记](../../../raw/current/scripts/prefabs/toadstool.lua#L901-L902) |
| 毒菌蟾蜍（普通） | 40% | `EnterPhase3Trigger`：未死亡 | 抛 1 张 | [触发](../../../raw/current/scripts/prefabs/toadstool.lua#L710-L718)、[登记](../../../raw/current/scripts/prefabs/toadstool.lua#L1004-L1004) |
| 悲惨的毒菌蟾蜍（`toadstool_dark`） | 70% | 共用 `EnterPhase2Trigger`：当前 > 40% | 抛 1 张 | [登记](../../../raw/current/scripts/prefabs/toadstool.lua#L901-L902) |
| 悲惨的毒菌蟾蜍 | 40% | `EnterPhase3TriggerDark`：当前 > 20% | 抛 1 张 | [触发](../../../raw/current/scripts/prefabs/toadstool.lua#L720-L728)、[登记](../../../raw/current/scripts/prefabs/toadstool.lua#L1029-L1029) |
| 悲惨的毒菌蟾蜍 | 20% | `EnterPhase4TriggerDark`：未死亡 | 抛 1 张 | [触发](../../../raw/current/scripts/prefabs/toadstool.lua#L730-L738)、[登记](../../../raw/current/scripts/prefabs/toadstool.lua#L1031-L1031) |

阈值是 `PHASE2_HEALTH = .7`、`PHASE3_HEALTH = .4`、`PHASE4_HEALTH = .2` 三个血量百分比常量。[常量](../../../raw/current/scripts/prefabs/toadstool.lua#L676-L678)、[触发语义](../../../raw/current/scripts/components/healthtrigger.lua#L25-L38)、[百分比载荷](../../../raw/current/scripts/components/health.lua#L642-L642)

抛出方式：`DropShroomSkin` 先 `SpawnPrefab("shroom_skin")`，再 `LaunchAt(..., inst, player, 1, 4, 2)`；其中 `player` 是取最近玩家的那行被注释后留下的 `nil`，所以 `LaunchAt` 的目标为 nil、走无目标分支（内部用 `TheCamera:GetDownVec()` 取基准方向），投掷参数为 `speedmult=1`、`startheight=4`、`startradius=2`。它**不写 `lootdropper`**，与击杀表完全独立。[抛出](../../../raw/current/scripts/prefabs/toadstool.lua#L695-L698)、[LaunchAt](../../../raw/current/scripts/physics.lua#L114-L137)

### 击杀表

| 表 | 蘑菇皮条目 | 其余要点 | 证据 |
|---|---|---|---|
| `toadstool`（普通） | 1 条 `1.00`（必掉 1 张） | 3 条 `meat` 1.00＋0.5＋0.25、`froglegs` 1.00、毒菌蟾蜍雕塑草图（`chesspiece_toadstool_sketch`）1.00、三色蘑菇各 1 条 1.00、各另有 2 条 0.33 | [表](../../../raw/current/scripts/prefabs/toadstool.lua#L88-L111)、[挂表](../../../raw/current/scripts/prefabs/toadstool.lua#L1008-L1008) |
| `toadstool_dark`（悲惨） | 2 条 `1.00`（必掉 2 张） | 另有毒菌蟾蜍雕塑草图 1.00、`mushroom_light2_blueprint` 与 `sleepbomb_blueprint` 各 1.00 | [表](../../../raw/current/scripts/prefabs/toadstool.lua#L113-L140)、[挂表](../../../raw/current/scripts/prefabs/toadstool.lua#L1035-L1035) |

击杀表在死亡时经 `DropDeathLoot` 结算：没有化尸就直接 `DropLoot`，化尸则把掷好的名单存进 `toadstoolcorpse`。**普通燃烧死亡**时每条都会过 `DropLoot` 的改写分支——蘑菇皮没有覆盖项、没有熟态变体、蟾蜍不是建筑，于是换成 `ash`；**受控燃烧**（Willow 控火点燃的蟾蜍或尸体）下该分支保留原物，蘑菇皮正常落地且不会冒烟（它没有 `burnable`）；阶段抛出的那几张完全不受影响（见上节）。[死亡结算](../../../raw/current/scripts/entityscript.lua#L2181-L2192)、[改写](../../../raw/current/scripts/components/lootdropper.lua#L415-L445)、[受控分支](../../../raw/current/scripts/components/lootdropper.lua#L436-L437)、[尸体定义](../../../raw/current/scripts/prefabs/corpses_defs.lua#L1107-L1118)、[名单转入尸体](../../../raw/current/scripts/componentutil.lua#L1571-L1579)

### 蓝图

蘑菇灯与睡袋炸弹的配方是 `TECH.LOST`，只能靠蓝图学：

- 普通形态的 `AddSpecialLoot` 必给一顶蘑菇帽蓝图，并按 `math.random() < .33` 给 `mushroom_light2_blueprint`，否则给 `mushroom_light_blueprint`，另加 2–3 个孢子。[普通额外掉落](../../../raw/current/scripts/prefabs/toadstool.lua#L68-L77)、[孢子](../../../raw/current/scripts/prefabs/toadstool.lua#L60-L66)
- 悲惨形态的 `AddSpecialLootDark` 给两顶蘑菇帽蓝图＋孢子；`sleepbomb_blueprint` 与 `mushroom_light2_blueprint` 由它的掉落表直接必掉。[悲惨额外掉落](../../../raw/current/scripts/prefabs/toadstool.lua#L79-L86)

击杀表里的 `chesspiece_toadstool_sketch` 是**雕塑草图**：条目名取 `NAMES.SKETCH` 的 `{item}草图` 模板＋`NAMES.CHESSPIECE_TOADSTOOL_BUILDER`＝“毒菌蟾蜍雕塑”，与蘑菇帽蓝图无关；蘑菇帽蓝图只由 `AddSpecialLoot`／`AddSpecialLootDark` 发放。[草图条目](../../../raw/current/scripts/prefabs/sketch.lua#L26-L26)、[草图命名](../../../raw/current/scripts/prefabs/sketch.lua#L158-L176)、[模板](../../../raw/current/scripts/languages/chinese_s.po#L349434-L349437)、[雕塑名](../../../raw/current/scripts/languages/chinese_s.po#L341394-L341397)

## 获得：啜食者皮（啜食者）

| 表 | 条目（概率） | 说明 | 证据 |
|---|---|---|---|
| `slurper` | `lightbulb` 1.00×2、`slurper_pelt` 0.5 | 灯泡必掉 2 个；皮是单条 0.5 **基础**概率（仅正向幸运提高） | [表](../../../raw/current/scripts/prefabs/slurper.lua#L20-L25)、[挂表](../../../raw/current/scripts/prefabs/slurper.lua#L320-L321)、[幸运](../../../raw/current/scripts/components/lootdropper.lua#L173-L184) |

啜食者的死亡状态没有走 `DropDeathLoot`，而是在死亡动画第 60 帧直接 `lootdropper:DropLoot(...)`；它也没有 `corpse` 状态，因此不存在化尸缓存。**普通燃烧死亡**时整表照 `DropLoot` 改写——皮与灯泡都会变成 `ash`；**受控燃烧**时两者都保留原物，且两者都自带 `burnable`，都会被 `StartWildfire()` 点燃冒烟。[死亡](../../../raw/current/scripts/stategraphs/SGslurper.lua#L246-L267)、[受控保留](../../../raw/current/scripts/components/lootdropper.lua#L436-L437)、[受控来源](../../../raw/current/scripts/components/burnable.lua#L352-L367)、[掉落入火](../../../raw/current/scripts/components/lootdropper.lua#L404-L407)、[灯泡可燃](../../../raw/current/scripts/prefabs/lightbulb.lua#L60-L61)

另外两点容易混：活着的啜食者本体是 `EQUIPSLOTS.HEAD` 的可装备物（会吸饥饿），和掉落的“皮”是两件东西；拿扫描仪扫啜食者给的是 `maxhunger` 扫描数据（3 点），与皮无关。[活体帽子](../../../raw/current/scripts/prefabs/slurper.lua#L297-L300)、[扫描数据](../../../raw/current/scripts/wx78_moduledefs.lua#L884-L895)

## 用途与配方

| 产物 | 配方 | 科技／门禁 | 证据 |
|---|---|---|---|
| 蘑菇灯 `mushroom_light` | 蘑菇皮 1＋肥料 1（`deconstruct=true`） | `TECH.LOST`＋蓝图；带 `mushroom_light_placer` | [配方](../../../raw/current/scripts/recipes.lua#L436-L436) |
| 菌伞灯 `mushroom_light2` | 蘑菇皮 1＋肥料 1＋木板 1 | `TECH.LOST`＋蓝图；带 `mushroom_light2_placer` | [配方](../../../raw/current/scripts/recipes.lua#L437-L437) |
| 睡袋炸弹 `sleepbomb` | 蘑菇皮 1＋中毒金丝雀 1 | `TECH.LOST`＋蓝图；一次产 4 | [配方](../../../raw/current/scripts/recipes.lua#L484-L484) |
| 饥饿腰带 `armorslurper` | 啜食者皮 6＋绳子 2＋噩梦燃料 2 | `TECH.MAGIC_THREE` | [配方](../../../raw/current/scripts/recipes.lua#L521-L521) |
| WX-78 饥饿模块 `wx78module_maxhunger` | 扫描数据 3＋啜食者皮 1＋`wx78module_maxhunger1` 1 | `TECH.ROBOTMODULECRAFT_ONE`＋`builder_tag = "upgrademoduleowner"`（只有 WX-78 一族） | [配方](../../../raw/current/scripts/recipes.lua#L756-L756)、[模块定义](../../../raw/current/scripts/wx78_moduledefs.lua#L884-L895) |

`TECH.LOST` 的实际值是 `{ MAGIC = 10, SCIENCE = 10, ANCIENT = 10 }`，任何原型机都到不了这个等级；配方只能靠 `<配方名>_blueprint` 学习。蓝图 prefab 由 `blueprint.lua` 为“至少有一条科技 > 0、且没有 `nounlock` 或 `builder_tag`”的配方自动生成，等级里出现 10 的会被判成稀有蓝图；学习走 `teacher:Teach` → `builder:UnlockRecipe` 把配方加进玩家的已学列表。[LOST 值](../../../raw/current/scripts/constants.lua#L1324-L1324)、[自动生成](../../../raw/current/scripts/prefabs/blueprint.lua#L237-L244)、[稀有判定](../../../raw/current/scripts/prefabs/blueprint.lua#L205-L235)、[教学](../../../raw/current/scripts/components/teacher.lua#L10-L28)、[写入已学](../../../raw/current/scripts/components/builder.lua#L463-L473)

这些成品的本体不在本页：睡袋炸弹的投掷与睡眠云见 [[reference/prefabs/throwables]]，材料金丝雀见 [[reference/prefabs/canary-poisoned]]，饥饿腰带的燃料与自毁见 [[reference/prefabs/armor-classic]]。蘑菇灯／菌伞灯本体（结构、灯芯过滤）**Wiki 待建**，灯具与灯芯关系可先看 [[reference/prefabs/lightbulb]]；WX-78 模块页 **Wiki 待建**（源码入口 `prefabs/wx78_modules.lua`）。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| `组件：stackable` | 蘑菇皮 10、皮 40，均由注册文件显式写 `maxsize` | 主模拟 | [[reference/components/stackable#关键接口]]；[蘑菇皮](../../../raw/current/scripts/prefabs/shroom_skin.lua#L31-L32) |
| `组件：edible`／`eater` | 皮只设 HORRIBLE；能否吃由食用者 `caneat` 决定 | 主模拟；玩家默认 OMNI 不含 HORRIBLE | [[reference/components/edible#关键接口]]、[[reference/components/eater#关键接口]]；[皮](../../../raw/current/scripts/prefabs/slurperpelt.lua#L40-L41) |
| `组件：tradable` | 皮的金值 1，交给猪王换金块 | 主模拟；`GIVE` 只对带 `tradable` 的物品出现 | [[reference/components/tradable#关键接口]]；[设置](../../../raw/current/scripts/prefabs/slurperpelt.lua#L35-L36) |
| `组件：burnable`／`propagator` | 皮可燃、会引燃周围；烧尽成灰 | 主模拟；默认 burn time 10 秒 | [[reference/components/burnable#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/slurperpelt.lua#L32-L33) |
| `组件：healthtrigger`（宿主侧） | 蟾蜍按血量百分比登记阶段回调，回调里再抛蘑菇皮 | 服务端；跨阈值才触发，一击跨多档按高→低 | [[reference/components/healthtrigger#关键接口]]；[登记](../../../raw/current/scripts/prefabs/toadstool.lua#L901-L902) |
| `组件：lootdropper`（宿主侧） | 蟾蜍两张击杀表、啜食者一张表；燃烧死亡改写 | 服务端；受控燃烧保留原物 | [[reference/components/lootdropper#关键接口]]；[蟾蜍表](../../../raw/current/scripts/prefabs/toadstool.lua#L88-L140) |
| `组件：builder`／`teacher` | 蓝图学习把 `TECH.LOST` 配方加入已学列表 | 服务端教学、`builder_replica` 客户端过滤 | [[reference/components/builder#关键接口]]、[[reference/components/teacher#关键接口]]；[教学](../../../raw/current/scripts/components/teacher.lua#L10-L28) |
| `关联 Prefab：toadstool／toadstool_dark` | 蘑菇皮的宿主；普通与悲惨两套掉率与阶段 | 洞穴实体；悲惨版多一档 20% 阶段 | Wiki 待建；[注册](../../../raw/current/scripts/prefabs/toadstool.lua#L1044-L1045) |
| `关联 Prefab：slurper` | 啜食者皮的宿主；另掉 2 个灯泡 | 洞穴实体；活体本身可作头部装备 | Wiki 待建；[注册](../../../raw/current/scripts/prefabs/slurper.lua#L419-L421) |
| `关联 Prefab：sleepbomb／armorslurper／wx78module_maxhunger` | 本页两件材料的成品 | 见“用途与配方”的门禁 | [[reference/prefabs/throwables]]、[[reference/prefabs/armor-classic]]；模块页待建 |
| `资源：anim/shroom_skin.zip／anim/slurper_pelt.zip` | bank／build 与注册名同名，播 `idle` | 原版自带；不分发资源 | [蘑菇皮资产](../../../raw/current/scripts/prefabs/shroom_skin.lua#L1-L4)、[皮资产](../../../raw/current/scripts/prefabs/slurperpelt.lua#L1-L4) |

### 共享规则接入

- **edible 默认值与 HORRIBLE 食性（本页为主章节）**：默认三维 10／10／0 来自 `components/edible.lua` 构造；啜食者皮只改 `foodtype`，所以三维保持默认，而能否入口取决于食用者 `caneat` 里有没有 HORRIBLE（默认玩家没有）。其他页引用这条时，请保留“默认玩家不可食、可喂给 HORRIBLE 食性生物”这层条件。
- **tradable 金值（本页为主章节）**：`GOLD_VALUES.MEAT = 1`，猪王按 `goldvalue` 次数发金块；无 `tradable` 的物品连 `GIVE` 动作都不会出现。
- **燃烧掉落改写**（主章节在 [[reference/prefabs/lightninggoat]]，不引跨批编号）：本页的接入差异有三处——①蟾蜍阶段抛出的蘑菇皮不经过 `DropLoot`，燃烧状态与它无关；②受控燃烧下击杀表条目保留原物，蘑菇皮不会变灰；③啜食者死亡直接 `DropLoot`，没有化尸缓存，所以不存在“尸体二次改写”这条路径。[宿主侧规则](../../../raw/current/scripts/prefabs/lightninggoat.lua#L21-L34)、[受控分支](../../../raw/current/scripts/components/lootdropper.lua#L436-L437)
- **堆叠口径**：这两件都是**显式赋值**（10／40），与骨头碎片那种“只挂组件、取默认 20”不同；改数值要改注册文件，不是改 `stackable` 默认。

## 常见陷阱与限制

- **误区：蘑菇皮只有击杀毒菌蟾蜍才掉。** 阶段抛出才是更早、更稳的来源：普通形态血量阈值 70%／40%、各阶段满足回调条件时各 1 张（最多 2 张），悲惨形态另有 20% 档（最多 3 张），再加击杀表的 1／2 张。[阶段](../../../raw/current/scripts/prefabs/toadstool.lua#L700-L738)
- **误区：一击把蟾蜍打到低血量，低档阶段的皮会漏发。** 同一击跨多档时按高→低回调，高档回调先看到“当前血量已低于下一档”就跳过，由真正跨过的那一档补发：普通形态 71%→39% 的一击**只抛 1 张**（40% 档补发）；如果这一击直接打死（`IsDead` 为真），则所有阶段回调都不抛——致死一击不产阶段皮。[条件](../../../raw/current/scripts/prefabs/toadstool.lua#L700-L738)、[触发顺序](../../../raw/current/scripts/components/healthtrigger.lua#L25-L38)
- **误区：燃烧死亡只是把肉烤熟。** 普通燃烧时击杀表里没有熟态变体的条目都会变成 `ash`，蘑菇皮与啜食者皮都在此列；受控燃烧除外——两者保留原物，且皮与灯泡都会因自身 `burnable` 冒烟。阶段抛出的蘑菇皮完全不走这条分支，因为它不经过 `lootdropper`。[改写](../../../raw/current/scripts/components/lootdropper.lua#L418-L442)
- **误区：有 `edible` 就能吃。** 啜食者皮是 HORRIBLE，默认玩家的 `FOODGROUP.OMNI` 不含这一类；它也不能进锅（锅料判定只查 `cooking.ingredients`，而 `slurper_pelt` 在 `cooking.lua` 全文 0 命中），没有 `cookable` 也不能烤。[默认食性](../../../raw/current/scripts/components/eater.lua#L18-L24)、[OMNI](../../../raw/current/scripts/constants.lua#L2051-L2062)、[锅料判定](../../../raw/current/scripts/cooking.lua#L185-L187)
- **边界：啜食者皮点燃等于销毁。** `MakeSmallBurnable` 烧尽后整件转灰烬（叠数受灰烬上限 40 约束），不是掉耐久；它也没有 `fuel` 组件，不能当燃料。[工厂](../../../raw/current/scripts/standardcomponents.lua#L164-L174)、[烧尽](../../../raw/current/scripts/standardcomponents.lua#L18-L44)
- **边界：`TECH.LOST` 不等于不能造。** 它表示“科技等级到不了”，蘑菇灯与睡袋炸弹要靠毒菌蟾蜍掉落的蓝图学；`TECH.LOST` 配方在自动生成蓝图时被判成稀有蓝图。[LOST 值](../../../raw/current/scripts/constants.lua#L1324-L1324)、[生成](../../../raw/current/scripts/prefabs/blueprint.lua#L237-L244)
- **未核验**：阶段抛出的落点与拾取体验、蟾蜍化尸与尸体烧毁的实际频率、喂猪人的实际忠诚时长都只按源码推导，未运行游戏。

## 验证记录

- 源码审阅：2026-11-10，Steam buildid 26474873。核验范围：`prefabs/shroom_skin.lua`、`prefabs/slurperpelt.lua` 全文（注册名、组件段、堆叠、可燃、edible、tradable）；`prefabs/toadstool.lua` 的两张击杀表、`AddSpecialLoot`／`AddSpecialLootDark`、四个阶段回调与 `DropShroomSkin`、`healthtrigger` 登记；`prefabs/slurper.lua` 与 `SGslurper` 的掉落与死亡结算；`components/edible` 默认值、`eater` 的 `TestFood` 与 OMNI 组成；`tradable`／`pigking`／`pigman` 的交易与喂食分支；`burnable`／`standardcomponents` 的烧尽转灰；`blueprint`／`teacher`／`builder` 的蓝图生成与学习；`recipes.lua` 五条配方与 `TECH.LOST`；两件物品的简中名称键。
- 未覆盖：毒菌蟾蜍与啜食者的战斗、阶段技能与生成调度；蘑菇灯／菌伞灯、睡袋炸弹、饥饿腰带、WX-78 模块的本体行为（链到已有页或标待建）；运行表现。
- 游戏运行：未运行；掉率、阶段条件与配方门禁均为源码推导。
- 结构检查：见交付回报。

## 相关链接

- 燃烧改写主章节：[[reference/prefabs/lightninggoat]]（B06 伏特羊页，不是本页两件材料的宿主）；同批关联：[[reference/prefabs/goatmilk]]
- 成品：[[reference/prefabs/throwables]]、[[reference/prefabs/canary-poisoned]]、[[reference/prefabs/armor-classic]]、[[reference/prefabs/lightbulb]]
- 同类材料：[[reference/prefabs/boneshard]]、[[reference/prefabs/lightninggoathorn]]、[[reference/prefabs/clockwork-parts]]、[[reference/prefabs/beefalowool]]
- 组件：[[reference/components/lootdropper]]、[[reference/components/healthtrigger]]、[[reference/components/edible]]、[[reference/components/tradable]]、[[reference/components/burnable]]、[[reference/components/builder]]

返回 [[reference/prefabs/index|原版 Prefab 索引]]。
