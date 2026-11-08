---
title: 伏特羊角（lightninggoathorn）
created: "2026-11-10"
updated: "2026-11-10"
type: prefab
summary: 单文件注册的可堆叠羊角材料：堆叠走 STACK_SIZE_LARGEITEM＝10、没有 edible／fuel／tradable／burnable，作祟只弹飞；伏特羊两张掉落表各按基础 0.25 单条掷（仅受击杀者正向幸运提高），普通燃烧死亡与烧毁尸体会被改写成灰烬、受控燃烧保留原物；用途是暗夜灯、龙卷风法杖、薇格弗德奔雷矛（技能门禁）与 Warly 便携锅的伏特羊肉冻。
aliases: ["lightninggoathorn", "伏特羊角", "Volt Goat Horn", "闪电羊角", "电羊角"]
tags: [prefab, item]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/lightninggoathorn.lua", "prefabs/lightninggoat.lua", "prefabs/warly.lua", "prefabs/skilltree_willow.lua", "components/lootdropper.lua", "components/stackable.lua", "components/burnable.lua", "components/builder.lua", "components/health.lua", "components/deathloothandler.lua", "components/corpsepersistmanager.lua", "entityscript.lua", "componentutil.lua", "prefabs/corpses.lua", "prefabs/corpses_defs.lua", "stategraphs/SGlightninggoat.lua", "stategraphs/commonstates.lua", "standardcomponents.lua", "tuning.lua", "cooking.lua", "preparedfoods_warly.lua", "containers.lua", "recipes.lua", "constants.lua", "recipe.lua", "prefabs/skilltree_wathgrithr.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 伏特羊角（lightninggoathorn）

## 快速判断

- 职责与适用：单文件单注册的库存材料，唯一来源是伏特羊——未充能表与充能表各有一条 `0.25` 的羊角；物品自身只挂 `stackable`（10）、`inspectable`、`inventoryitem` 与 `MakeHauntableLaunch`。可借鉴：同一个掉落物在两套宿主状态表里共用同一条概率项，以及“能进锅、但不能直接吃”的材料登记法。[注册](../../../raw/current/scripts/prefabs/lightninggoathorn.lua#L38)
- 优先使用：查羊角怎么掉、什么情况下变灰看[获得](#获得)；查它能造什么、卡在哪道门禁看[用途与配方](#用途与配方)；宿主侧的充能／放电与奶见 [[reference/prefabs/lightninggoat]]。
- 重要副作用：`0.25` 是每次死亡独立掷一次的基础机会项，不是保底（实际概率按击杀者正向幸运提高）。掉落者带 `burnt`、或正在燃烧且（`fueled == nil` 或 `burnable.ignorefuel`）时，掉落表逐条走改写链：覆盖表 → 熟态变体 → 受控燃烧保留 → 其余 `ash`；羊角无覆盖项也无熟态，普通燃烧下变 `ash`，受控燃烧（Willow 控火）下保留原物落地，且因自身没有 `burnable` 不会冒烟。[幸运修正](../../../raw/current/scripts/components/lootdropper.lua#L173-L184)、[概率表](../../../raw/current/scripts/components/lootdropper.lua#L213-L227)、[改写](../../../raw/current/scripts/components/lootdropper.lua#L415-L445)
- 修改边界：本页不覆盖伏特羊的雷击改道、充能递减与群体行为（归 [[reference/prefabs/lightninggoat]]），也不覆盖暗夜灯／龙卷风法杖／奔雷矛本体；两种洞穴材料的掉落见 [[reference/prefabs/cave-drops]]。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `lightninggoathorn` | 单文件单注册 `Prefab("lightninggoathorn", fn, assets)`，无包装工厂。公共端：`CreateEntity` → Transform／AnimState／Network → `MakeInventoryPhysics` → bank／build 都是 `lightning_goat_horn` → `PlayAnimation("idle")` → `MakeInventoryFloatable(inst)`（无参 → `small` 档） → `SetPristine`，非主模拟直接返回。 | 主模拟只有 4 项：`stackable`（`maxsize = TUNING.STACK_SIZE_LARGEITEM`）、`inspectable`、`inventoryitem`、`MakeHauntableLaunch`。没有 `edible`／`fuel`／`burnable`／`tradable`／`cookable`／`bait`。[fn](../../../raw/current/scripts/prefabs/lightninggoathorn.lua#L6-L36) |

### 名称与实显

简中 `STRINGS.NAMES.LIGHTNINGGOATHORN`＝**伏特羊角**（英文 Volt Goat Horn）。物品栏与检视都按注册名取这个键；本 prefab 没有设 `nameoverride`，显示名不依赖检视覆盖。[名称](../../../raw/current/scripts/languages/chinese_s.po#L344739-L344742)

“闪电羊角”“电羊角”只是描述性别名，正文主称按简中实显写“伏特羊角”。

## 数值与参数

| 项 | 实际值 | 证据 |
|---|---|---|
| 堆叠上限 | 10（`STACK_SIZE_LARGEITEM`；文件显式赋值，不是组件默认 20） | [设置](../../../raw/current/scripts/prefabs/lightninggoathorn.lua#L27-L28)、[常量](../../../raw/current/scripts/tuning.lua#L81-L83) |
| 浮水 | 无参 `MakeInventoryFloatable` → 尺寸 `small`、无额外偏移／缩放 | [调用](../../../raw/current/scripts/prefabs/lightninggoathorn.lua#L19)、[helper](../../../raw/current/scripts/standardcomponents.lua#L1772-L1782) |
| 作祟 | `MakeHauntableLaunch`：必弹飞并把 `inventoryitem.is_landed` 置 false；没有砸碎或点燃段 | [调用](../../../raw/current/scripts/prefabs/lightninggoathorn.lua#L33)、[helper](../../../raw/current/scripts/standardcomponents.lua#L1053-L1069) |
| 可食／燃料／交易／可燃 | 都没有。不能吃、不能当燃料；没有 `tradable` 所以 `GIVE` 动作不会出现在猪王身上；自身不会被点燃 | [全文](../../../raw/current/scripts/prefabs/lightninggoathorn.lua#L6-L36)、[GIVE 门禁](../../../raw/current/scripts/componentactions.lua#L1860-L1868) |
| 锅料登记 | `cooking.lua` 把 `lightninggoathorn` 登记成 `inedible=1` 的锅料：能放进烹饪锅原料格，但只贡献 `inedible` 权重 | [登记](../../../raw/current/scripts/cooking.lua#L135-L135)、[锅的过滤](../../../raw/current/scripts/containers.lua#L373-L375) |

## 获得

宿主的两张表都在 `prefabs/lightninggoat.lua` 顶部用 `SetSharedLootTable` 注册，羊角在两张表里都是同一条 `0.25`：

| 表名 | 条目（概率） | 羊角口径 | 证据 |
|---|---|---|---|
| `lightninggoat`（未充能） | `meat` 1.00×2、`lightninggoathorn` 0.25 | 单条独立掷，基础概率 0.25（基础期望 0.25 个；实际按击杀者幸运修正） | [表](../../../raw/current/scripts/prefabs/lightninggoat.lua#L21-L26)、[幸运](../../../raw/current/scripts/components/lootdropper.lua#L173-L184) |
| `chargedlightninggoat`（充能） | `meat` 1.00×2、`goatmilk` 1.00、`lightninggoathorn` 0.25 | 同上；多出的奶是另一条 1.00 | [表](../../../raw/current/scripts/prefabs/lightninggoat.lua#L28-L34) |

两表的切换（`setcharged`／`discharge` 里的 `SetChanceLootTable`）归 [[reference/prefabs/lightninggoat]]，本页只接掉落结果。[切换](../../../raw/current/scripts/prefabs/lightninggoat.lua#L92-L119)

结算分五种情形。单次死亡只生成一次掉落，直接掉落与转存是互斥路径：前者由 `DropLoot(pt)` 生成，后者由 `DropDeathLoot` 显式 `GenerateLoot` 后转存；尸体提交把处理后的名单**显式**传给 `DropLoot`，不再生成。

| 情形 | 掷表位置与结算 | 燃烧改写是否生效 |
|---|---|---|
| 直接掉落（非化尸，常态） | 死亡状态 `onenter` 调 `DropDeathLoot`；非化尸分支只传**位置**，`DropLoot(pt)` 内部 `GenerateLoot()` 现掷，`0.25` 在这一刻判定 | 这次调用时带 `burnt` 或正在燃烧（`fueled == nil` 或 `ignorefuel`）才改写：普通燃烧 → `ash`，受控燃烧保留原物 |
| 转存（化尸） | 化尸分支显式 `GenerateLoot()` 后 `StoreLoot`，名单进 `deathloothandler`，随 `lightninggoatcorpse` 走 | 转存时不判定，结果由尸体后续结算决定 |
| 尸体燃尽 | 尸体 `OnBurnt` → `DropCorpseLoot`，把已按肉量处理的名单显式传给 `DropLoot` | 此刻仍在燃烧：普通燃烧把羊角换 `ash`，受控燃烧保留原物 |
| 尸体熄灭（未变异、无 `_skip_extinguish_fade`／`no_destroy_on_burn` 保护标记） | `Extinguish` 先清 `burning` 与 `controlled_burn`，再回调 `OnExtinguish` → `DropCorpseLoot`；变异中或带保护标记时跳过这次普通熄灭掉落 | 改写门禁已不成立 ⇒ 不改写，原物落地 |
| 啃空／侵蚀 | 侵蚀先卸掉 `burnable` 再 `DropCorpseLoot` | 同上，不改写 |

伏特羊默认不走化尸，所以常规路径只有第一行：直接掉落＋当次燃烧门禁。[死亡入口](../../../raw/current/scripts/entityscript.lua#L2181-L2192)、[DropLoot 生成](../../../raw/current/scripts/components/lootdropper.lua#L415-L421)、[转存实现](../../../raw/current/scripts/components/deathloothandler.lua#L11-L15)、[尸体提交掉落](../../../raw/current/scripts/prefabs/corpses.lua#L489-L496)、[肉量处理](../../../raw/current/scripts/prefabs/corpses.lua#L465-L487)、[燃尽回调](../../../raw/current/scripts/prefabs/corpses.lua#L38-L43)、[熄灭顺序](../../../raw/current/scripts/components/burnable.lua#L465-L493)、[熄灭回调](../../../raw/current/scripts/prefabs/corpses.lua#L45-L52)、[侵蚀前卸可燃](../../../raw/current/scripts/prefabs/corpses.lua#L158-L173)

概率语义与幸运：`chance >= 1.0` 的条目必出；其余逐条 `math.random() <= GetChance(chance)`，`GetChance` 用击杀者（`health.causeofdeath`）的幸运值套 `LuckFormulas.LootDropperChance`——只处理正向幸运（`luck > 0` 时 ×`3 − 6/(luck+3)`，上限 ×3），`luck <= 0` 返回基础概率。羊角基础概率 0.25，只有正向幸运才会抬高。[概率语义](../../../raw/current/scripts/components/lootdropper.lua#L213-L227)、[幸运公式](../../../raw/current/scripts/componentutil.lua#L2151-L2151)、[公式实现](../../../raw/current/scripts/componentutil.lua#L2092-L2097)

改写链顺序（只在门禁成立时走）：覆盖表 → `<名>_cooked`／`cooked<名>` → **受控燃烧保留** → 其余非建筑、非树／石类换 `ash`。羊角无覆盖项也无熟态，所以普通燃烧下换 `ash`；受控燃烧（`GetControlledBurn()` 非空）保留原物，且因没有 `burnable` 不会被 `StartWildfire` 点燃冒烟。两件 `meat` 在受控判断之前就换成熟态 `cookedmeat`。[条件与顺序](../../../raw/current/scripts/components/lootdropper.lua#L418-L442)、[受控分支](../../../raw/current/scripts/components/lootdropper.lua#L436-L437)、[受控来源](../../../raw/current/scripts/components/burnable.lua#L352-L367)、[控火技能](../../../raw/current/scripts/prefabs/skilltree_willow.lua#L60-L63)、[掉落入火](../../../raw/current/scripts/components/lootdropper.lua#L404-L407)、[覆盖表](../../../raw/current/scripts/tuning.lua#L4511-L4530)

成尸资格按顺序判：①状态图有 `corpse` 且 `sg.mem.nocorpse` 未置；②`forcecorpse`；③之后才用“正在燃烧”阻止其余分支（持久保留、月变）。原版没有 `forcecorpse`，伏特羊的 SG 也没接月变、未置 `nocorpse`，所以默认路径只有“未燃烧＋变异秃鹫保留”一种；`IsMutating()`、`_skip_extinguish_fade`、`no_destroy_on_burn` 只在尸体回调上跳过普通熄灭掉落，不是伏特羊的常规入口。[状态图前置](../../../raw/current/scripts/componentutil.lua#L1472-L1475)、[资格顺序](../../../raw/current/scripts/componentutil.lua#L1546-L1561)、[SG 挂尸体状态](../../../raw/current/scripts/stategraphs/SGlightninggoat.lua#L305-L305)、[转移](../../../raw/current/scripts/componentutil.lua#L1571-L1579)

因此“羊角从击杀里稳定获得”不成立：直接掉落时单次击杀的获得量是 0 或 1；普通燃烧死亡（含烧毁尸体）改成灰烬拿不到，受控燃烧击杀／烧尸保留原物；尸体阶段不会再掷一次。

## 用途与配方

| 产物 | 配方（羊角位置） | 门禁 | 证据 |
|---|---|---|---|
| 暗夜灯 `nightstick` | 羊角 1＋电子元件 2＋硝石 2 | `TECH.SCIENCE_TWO`（科学 2 级，炼金引擎） | [配方](../../../raw/current/scripts/recipes.lua#L482-L482)、[科技键](../../../raw/current/scripts/constants.lua#L1245-L1246)、[炼金引擎](../../../raw/current/scripts/prefabs/scienceprototyper.lua#L300-L300) |
| 龙卷风法杖 `staff_tornado` | 羊角 1＋鹅毛 10＋齿轮 1 | `TECH.SCIENCE_TWO` | [配方](../../../raw/current/scripts/recipes.lua#L490-L490) |
| 奔雷矛 `spear_wathgrithr_lightning` | 羊角 2＋树枝 2＋牛毛 1 | `TECH.NONE`，但带 `builder_skill = "wathgrithr_arsenal_spear_3"`：技能树节点激活后才显示／可造 | [配方](../../../raw/current/scripts/recipes.lua#L192-L192)、[字段](../../../raw/current/scripts/recipe.lua#L137-L137)、[技能检查](../../../raw/current/scripts/components/builder.lua#L876-L880)、[技能节点](../../../raw/current/scripts/prefabs/skilltree_wathgrithr.lua#L232-L243) |
| 伏特羊肉冻 `voltgoatjelly` | 便携锅配方：`names.lightninggoathorn` 且甜味剂 ≥2 且无肉 | 该菜只登记在 `portablecookpot`；便携锅本身是 Warly 专属（`masterchef`） | [菜谱](../../../raw/current/scripts/preparedfoods_warly.lua#L29-L49)、[只进便携锅](../../../raw/current/scripts/cooking.lua#L200-L203)、[锅的配方门禁](../../../raw/current/scripts/recipes.lua#L97-L97)、[masterchef 标签](../../../raw/current/scripts/prefabs/warly.lua#L18-L18) |

烹饪的三层门禁与骨头碎片同型：羊角能被锅接（`IsCookingIngredient` 只查锅料登记），但没有 `edible` 所以不能直接吃，也没有 `cookable` 所以不能在篝火上烤。[锅的过滤](../../../raw/current/scripts/containers.lua#L373-L375)、[烤制门禁](../../../raw/current/scripts/components/cooker.lua#L12-L18)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| `组件：stackable` | 承载 10 上限的合并栈；`SetStackSize` 推 `stacksizechange` | 主模拟；上限显式写成 `STACK_SIZE_LARGEITEM` | [[reference/components/stackable#关键接口]]；[设置](../../../raw/current/scripts/prefabs/lightninggoathorn.lua#L27-L28) |
| `组件：hauntable` | `MakeHauntableLaunch` 只弹飞 | 主模拟；命中概率默认 `HAUNT_CHANCE_ALWAYS` | [[reference/components/hauntable#关键接口]]；[helper](../../../raw/current/scripts/standardcomponents.lua#L1053-L1069) |
| `组件：lootdropper`（宿主侧） | 宿主用两张 `SetSharedLootTable` 表，死亡时照表掷 | 服务端；燃烧死亡时经 `DropLoot` 改写（受控燃烧保留原物） | [[reference/components/lootdropper#关键接口]]；[宿主表](../../../raw/current/scripts/prefabs/lightninggoat.lua#L21-L34) |
| `组件：deathloothandler`／尸体 | 化尸时保存显式 `GenerateLoot` 得到的名单，随 `lightninggoatcorpse` 转移 | 服务端；需 `CanEntityBecomeCorpse` 成立 | [[reference/components/deathloothandler]]、[[reference/components/corpsepersistmanager]]；[尸体定义](../../../raw/current/scripts/prefabs/corpses_defs.lua#L829-L843)、[转存](../../../raw/current/scripts/entityscript.lua#L2181-L2192) |
| `API／数据：cooking.ingredients` | 登记为 `inedible=1` 锅料，供锅的 `itemtestfn` 放行 | 模块加载期注册 | [[reference/components/edible]]；[登记](../../../raw/current/scripts/cooking.lua#L135-L135) |
| `API／数据：builder_skill` | 奔雷矛配方要求技能树节点 `wathgrithr_arsenal_spear_3` | 服务端 `KnowsRecipe`／`CanLearn` 双侧检查 | [[reference/components/builder#关键接口]]；[检查](../../../raw/current/scripts/components/builder.lua#L876-L880) |
| `关联 Prefab：lightninggoat` | 两张表的宿主；充能／放电切换表 | 服务端；雷击／电击条件见宿主页 | [[reference/prefabs/lightninggoat#掉落表]]；[表](../../../raw/current/scripts/prefabs/lightninggoat.lua#L21-L34) |
| `关联 Prefab：goatmilk` | 充能表的另一条产物 | 充能期间死亡且未燃烧改写 | [[reference/prefabs/goatmilk]]；[表](../../../raw/current/scripts/prefabs/lightninggoat.lua#L28-L34) |
| `关联 Prefab：lightninggoatherd／lightninggoatcorpse` | 宿主的预载依赖；尸体承接化尸掉落 | 声明不等于实际生成 | [依赖](../../../raw/current/scripts/prefabs/lightninggoat.lua#L10-L17)、[尸体](../../../raw/current/scripts/prefabs/corpses_defs.lua#L829-L843) |
| `资源：anim/lightning_goat_horn.zip` | bank／build 同名，播 `idle` | 原版自带；不分发资源 | [资产表](../../../raw/current/scripts/prefabs/lightninggoathorn.lua#L1-L4) |

### 共享规则接入

- **燃烧掉落改写**（不引跨批编号）：主章节与完整推导在 [[reference/prefabs/lightninggoat]]（该页登记了伏特羊的可燃烧构造与 `DropDeathLoot` 分支）；本页只接“羊角没有熟态变体、非受控燃烧 → `ash`”这一项，并补尸体结算的三种入口（燃尽／熄灭／侵蚀）：三者都把同一份名单再交给 `DropLoot`，只有燃尽时燃烧改写门禁仍成立，熄灭与侵蚀都不改写。
- **堆叠与作祟差异**（本页为主章节）：羊角堆叠 10、作祟只弹飞；啜食者皮堆叠 40 且**没有** `hauntable`；蘑菇皮堆叠 10、作祟也只弹飞。三者的差异来自各自文件显式写的 `maxsize` 与是否调 `MakeHauntable*`，不是组件默认值。[对照](../../../raw/current/scripts/prefabs/slurperpelt.lua#L27-L36)、[蘑菇皮](../../../raw/current/scripts/prefabs/shroom_skin.lua#L31-L34)

## 常见陷阱与限制

- **误区：击杀伏特羊必掉羊角。** 两张表里羊角都是 `0.25`，每次死亡独立掷一次；1.00 的条目只有 `meat`（充能表另有 `goatmilk`）。[表](../../../raw/current/scripts/prefabs/lightninggoat.lua#L21-L34)
- **误区：烧死只少一瓶奶。** 普通燃烧／已燃死亡会改写整张掉落表，羊角同样变 `ash`；但受控燃烧（Willow 控火）下没有熟态的条目会保留原物，只有命中熟态分支的 `meat` 仍换 `cookedmeat`。[改写](../../../raw/current/scripts/components/lootdropper.lua#L418-L442)
- **边界：受控燃烧只“保留”，不保证“冒烟”。** 掉落物要自身带 `burnable` 才会在受控燃烧死亡时被 `StartWildfire()` 点燃；羊角没有 `burnable`，因此受控燃烧下只保留原名落地，不会冒烟（带 `burnable` 的啜食者皮、灯泡才会冒烟，见 [[reference/prefabs/cave-drops]]）。[条件](../../../raw/current/scripts/components/lootdropper.lua#L404-L407)
- **边界：死亡不一定化尸。** 常态死亡直接 `DropLoot`；只有 `CanEntityBecomeCorpse` 成立（顺序：状态图有 `corpse` 且 `sg.mem.nocorpse` 未置 → `forcecorpse` → 才用燃烧阻止其余分支；原版没有 `forcecorpse`，所以等价于“未燃烧”，再加变异秃鹫保留）时才在死亡时显式掷表并转存。化尸后尸体被吃空／侵蚀不改写，但**普通燃烧烧毁尸体仍会把羊角换成灰烬**（受控燃烧烧尸保留原物，见上）。[化尸](../../../raw/current/scripts/componentutil.lua#L1546-L1561)、[尸体烧毁](../../../raw/current/scripts/prefabs/corpses.lua#L38-L43)
- **边界：堆叠 10 是本文件写的。** 它在 `prefabs/lightninggoathorn.lua` 里显式赋 `STACK_SIZE_LARGEITEM`；该常量是 10，而组件构造默认是 `STACK_SIZE_MEDITEM`＝20。[设置](../../../raw/current/scripts/prefabs/lightninggoathorn.lua#L27-L28)、[默认值](../../../raw/current/scripts/components/stackable.lua#L23-L34)
- **边界：能进锅不等于能吃，也不等于能烤。** 羊角只是锅料登记（`inedible=1`），没有 `edible`／`cookable`；伏特羊肉冻还只登记在便携锅，普通锅不会出这道菜。[登记](../../../raw/current/scripts/cooking.lua#L135-L135)、[便携锅菜谱](../../../raw/current/scripts/cooking.lua#L200-L203)
- **误区：`TECH.NONE` 的奔雷矛人人可造。** 它还要求 `builder_skill` 节点已激活，且要薇格弗德；`TECH.NONE` 只表示不依赖科技机器。[配方](../../../raw/current/scripts/recipes.lua#L192-L192)、[技能检查](../../../raw/current/scripts/components/builder.lua#L876-L880)
- **边界：作祟只弹飞。** `MakeHauntableLaunch` 本就没有砸碎／点燃段；本 build 的 `LaunchAndSmash`／`LaunchAndIgnite` 附加段也被整段注释，不能按 helper 名推断会破坏或起火。[Launch](../../../raw/current/scripts/standardcomponents.lua#L1053-L1069)、[LaunchAndSmash](../../../raw/current/scripts/standardcomponents.lua#L1071-L1105)、[LaunchAndIgnite](../../../raw/current/scripts/standardcomponents.lua#L1191-L1217)
- **未核验**：化尸在变异秃鹫区域的触发频率、烧死与烧尸体的实际掉落体验都只按源码推导，未运行游戏。

## 验证记录

- 源码审阅：2026-11-10，Steam buildid 26474873。核验范围：`prefabs/lightninggoathorn.lua` 全文（注册、公共端、四个组件、堆叠常量、作祟）；`prefabs/lightninggoat.lua` 的两张共享表与 `SetChanceLootTable` 切换；`lootdropper` 的概率判定与燃烧改写分支、`BURNED_LOOT_OVERRIDES`；`entityscript`／`health`／`componentutil`／`corpses` 的死亡、化尸与尸体烧毁路径；`recipes.lua` 的三条配方与 `builder_skill` 检查、技能树节点；`cooking.lua` 的锅料登记与便携锅菜谱、`containers`／`cooker` 的锅与烤制门禁；简中名称键。
- 未覆盖：伏特羊充能／放电与雷击改道（归宿主页）、暗夜灯／龙卷风法杖／奔雷矛本体、伏特羊肉冻的实战收益、尸体变异分支。
- 游戏运行：未运行；概率、改写与配方结果均为源码推导。
- 结构检查：见交付回报。

## 相关链接

- 宿主与奶：[[reference/prefabs/lightninggoat]]、[[reference/prefabs/goatmilk]]
- 三个成品：[[reference/prefabs/nightstick]]、[[reference/prefabs/staffs-more]]、[[reference/prefabs/event-weapons]]
- 同批材料：[[reference/prefabs/cave-drops]]、[[reference/prefabs/beefalowool]]、[[reference/prefabs/clockwork-parts]]、[[reference/prefabs/boneshard]]
- 组件：[[reference/components/lootdropper]]、[[reference/components/stackable]]、[[reference/components/hauntable]]、[[reference/components/deathloothandler]]

返回 [[reference/prefabs/index|原版 Prefab 索引]]。
