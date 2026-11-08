---
title: "蝴蝶翅膀与月蛾翅膀（butterflywings、moonbutterflywings）"
created: "2026-11-09"
updated: "2026-11-09"
type: prefab
summary: "两件蝶翅的对照：VEGGIE 食性、三维（月蛾翅膀多 +15 理智）、6 天鲜度、堆叠 40 与 cattoy 标签；炖锅只记 decoration=2（蝴蝶松饼必须另配素食），以及温蒂召唤、绷带、竞速鞍与月书等配方与两件掉落来源。"
aliases: ["butterflywings", "moonbutterflywings", "蝴蝶翅膀", "月蛾翅膀", "蝶翅"]
tags: [prefab, item, food]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/butterflywings.lua", "prefabs/moonbutterflywings.lua", "prefabs/butterfly.lua", "prefabs/moonbutterfly.lua", "prefabs/tumbleweed.lua", "components/edible.lua", "components/perishable.lua", "components/stackable.lua", "components/lootdropper.lua", "components/container.lua", "cooking.lua", "preparedfoods.lua", "recipes.lua", "prefabs/catcoon.lua", "brains/catcoonbrain.lua", "actions.lua", "standardcomponents.lua", "componentactions.lua", "prefablist.lua", "tuning.lua", "languages/chinese_s.po", "screens/redux/scrapbookdata.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 蝴蝶翅膀与月蛾翅膀（butterflywings、moonbutterflywings）

## 快速判断

- 职责与适用：两件都是可拾取、可食用的物品（简中实显 **蝴蝶翅膀**／**月蛾翅膀**），各自独占一个 `prefabs/*.lua`、只导出一个注册、只依赖 `spoiled_food`。两者数值几乎相同，真正分道的是**理智 +15**、贴图与来源。[蝶翅](../../../raw/current/scripts/prefabs/butterflywings.lua#L11-L57)、[月蛾翅](../../../raw/current/scripts/prefabs/moonbutterflywings.lua#L11-L58)
- 优先使用：数值／鲜度／堆叠看[两件对照](#两件对照)；从哪来、能做哪些东西看[获得途径](#获得途径)与[烹饪与配方](#烹饪与配方)。
- 重要副作用：两件在炖锅里只贡献 `decoration` 2，**不贡献菜类**——蝴蝶松饼要求 `veggie >= 0.5`，必须另有胡萝卜等 `veggie` 食材（浆果只给 `fruit`，顶不上）；带 `cattoy` 标签，会被浣猫当玩具或礼物收下。
- 修改边界：蝴蝶与月蛾本体各归本批／B07 页面，本页只写两件物品与它们的产出、消耗关系；月蛾的变形链只给入口。

## 注册与构造

`prefablist.lua` 分别以 `"butterflywings"` 与 `"moonbutterflywings"` 登记，两个文件互不引用。[蝶翅登记](../../../raw/current/scripts/prefablist.lua#L192)、[月蛾翅登记](../../../raw/current/scripts/prefablist.lua#L795)

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `butterflywings` | `fn()`：bank／build `butterfly_wings`、播 `idle`；pristine 前写 `inst.pickupsound = "vegetation_firm"`、加 `cattoy`、`MakeInventoryFloatable("small", 0.03)` | 主模拟：`edible`（`VEGGIE`）→ `stackable` 40 → `inspectable`／`inventoryitem`／`tradable` → `perishable`（`PERISH_FAST`→`spoiled_food`）→ `MakeHauntableLaunchAndPerish`。[构造](../../../raw/current/scripts/prefabs/butterflywings.lua#L11-L57) |
| `moonbutterflywings` | 同上换 bank／build `moonbutterfly_wings`，同样 `cattoy` 与 `("small", 0.03)` | **不写** `pickupsound`；`edible.sanityvalue = TUNING.SANITY_MED`（+15），其余数值与蝶翅逐项相同。[构造](../../../raw/current/scripts/prefabs/moonbutterflywings.lua#L11-L58) |

- 两件都在 `SetPristine()` **之前**加 `cattoy`，所以客户端也认这个标签；`stackable` 显式写 `STACK_SIZE_SMALLITEM`＝40，不走组件默认的 20。[蝶翅标签](../../../raw/current/scripts/prefabs/butterflywings.lua#L26-L28)、[堆叠](../../../raw/current/scripts/prefabs/moonbutterflywings.lua#L40-L41)、[组件默认](../../../raw/current/scripts/components/stackable.lua#L23-L28)
- 两件都**没有** `cookable`、`dryable`、`bait` 组件：既不能上火烤，也不能上晾晒架，只能下炖锅或直接吃。[蝶翅组件段](../../../raw/current/scripts/prefabs/butterflywings.lua#L36-L52)

### 名称核验

`NAMES.BUTTERFLYWINGS`＝蝴蝶翅膀、`NAMES.MOONBUTTERFLYWINGS`＝月蛾翅膀；对应的生物是 `NAMES.BUTTERFLY`＝蝴蝶、`NAMES.MOONBUTTERFLY`＝月蛾。[BUTTERFLYWINGS](../../../raw/current/scripts/languages/chinese_s.po#L340034-L340037)、[MOONBUTTERFLYWINGS](../../../raw/current/scripts/languages/chinese_s.po#L345614-L345617)

## 两件对照

1 天 = 480 秒。[时间基准](../../../raw/current/scripts/tuning.lua#L19-L20)

| 注册 | 生命 | 饥饿 | 理智 | 食性 | 鲜度 | 腐坏产物 | 堆叠 | 漂浮 | 拾取音 |
|---|---:|---:|---:|---|---|---|---:|---|---|
| `butterflywings` | +8（`HEALING_MEDSMALL`） | +9.375（`CALORIES_TINY`） | 0（未写，组件默认） | `VEGGIE` | 6 天（`PERISH_FAST`） | `spoiled_food` | 40 | `small`，0.03 | `vegetation_firm` |
| `moonbutterflywings` | +8 | +9.375 | **+15**（`SANITY_MED`） | `VEGGIE` | 6 天 | `spoiled_food` | 40 | `small`，0.03 | 未写 |

- 常量：`HEALING_MEDSMALL`＝8、`CALORIES_TINY`＝9.375、`SANITY_MED`＝15、`PERISH_FAST`＝6 天、`STACK_SIZE_SMALLITEM`＝40。[数值](../../../raw/current/scripts/tuning.lua#L2021-L2061)、[堆叠](../../../raw/current/scripts/tuning.lua#L82-L83)
- 图鉴记录与上表一致（两件都是 `hungervalue` 9.375／`healthvalue` 8／`perishable` 2880 秒／`stacksize` 40），月蛾翅膀额外记 `sanityvalue` 15，可作交叉核对。[蝶翅](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L249)、[月蛾翅](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L949)
- 两件都显式写 `foodtype = FOODTYPE.VEGGIE`，因此带 `edible_VEGGIE` 标签；正生命、非负理智，不会被打 `badfood`。[食性](../../../raw/current/scripts/components/edible.lua#L9-L24)、[badfood 判定](../../../raw/current/scripts/components/edible.lua#L1-L7)
- 腐坏时 `perishable:Perish()` 生成 `spoiled_food`，按原叠数同步后交给持有者的原槽位；容器按 `itemtestfn` 拒收时掉在原地。[替换](../../../raw/current/scripts/components/perishable.lua#L301-L312)、[入槽校验](../../../raw/current/scripts/components/container.lua#L295)、[拒收落地](../../../raw/current/scripts/components/container.lua#L560-L566)

## 获得途径

| 来源 | 产出 | 条件 | 证据 |
|---|---|---|---|
| 蝴蝶 `butterfly` | 1 件 `butterflywings`（权重 5，与黄油 0.1 共用一次抽取，约 98.04%） | 击杀；`numrandomloot = 1` | [掉落表](../../../raw/current/scripts/prefabs/butterfly.lua#L158-L162)、[抽取](../../../raw/current/scripts/components/lootdropper.lua#L99-L112) |
| 月蛾 `moonbutterfly` | 恰好 1 件 `moonbutterflywings` | 击杀或场景内饿死；共享表里记 1.0，`chance >= 1.0` 直接给，不再掷点；燃烧死亡会被改写成 `ash` | [共享表](../../../raw/current/scripts/prefabs/moonbutterfly.lua#L17-L20)、[必给分支](../../../raw/current/scripts/components/lootdropper.lua#L213-L227) |
| 风滚草 `tumbleweed` | 权重 1 的 `butterflywings`，每次抽取约 1%（权重和约 100，另有按世界设置与棋类解锁追加的条目） | 打散时按权重表连抽 3 件；抽到什么都不影响实体本身 | [权重表](../../../raw/current/scripts/prefabs/tumbleweed.lua#L147-L184)、[抽 3 件](../../../raw/current/scripts/prefabs/tumbleweed.lua#L218-L245)、[权重和](../../../raw/current/scripts/prefabs/tumbleweed.lua#L213-L216) |

- 月蛾本体是蝴蝶被"月亮药水"变形后的形态，变形入口写在 [[reference/prefabs/butterfly]] 的关联导航里；月蛾的战斗与行为见 [[reference/prefabs/moonbutterfly]]，本页只取掉落关系。
- `butterflywings` 在检索范围内还有第二处来源：风滚草（`tumbleweed`）的打散掉落表里以权重 1 收录，权重和约 100，一次打散抽 3 件，单次打散命中约 3%。
- `moonbutterflywings` 只有月蛾一处来源，未见拆解或套装配方产出它们。

## 烹饪与配方

`cooking.lua` 只给两件登记了一个标签：`decoration = 2`，**没有** `veggie`、`meat`、`sweetener` 等可计数的食材类别。[食材登记](../../../raw/current/scripts/cooking.lua#L131-L132)

| 用途 | 消耗 | 关键条件 | 证据 |
|---|---|---|---|
| 蝴蝶松饼 `butterflymuffin` | 1 件蝶翅（两件任一） | `(names.butterflywings or names.moonbutterflywings) and not tags.meat and tags.veggie >= 0.5`；菜谱卡示例为蝶翅 1＋胡萝卜 2＋浆果 1 | [判定](../../../raw/current/scripts/preparedfoods.lua#L3-L16) |
| 温蒂召唤蝴蝶 `wendy_butterfly` | `butterflywings` 1＋`ghostflower` 2 | 温蒂技能 `wendy_ghostflower_butterfly`，产物是活的 `butterfly` | [配方](../../../raw/current/scripts/recipes.lua#L164) |
| 温蒂召唤月蛾 `wendy_moonbutterfly` | `moonbutterflywings` 1＋`ghostflower` 4 | 同一技能，产物是 `moonbutterfly` | [配方](../../../raw/current/scripts/recipes.lua#L165) |
| 蝶翅绷带 `bandage_butterflywings` | `butterflywings` 3 | 沃尔特技能 `walter_camp_firstaid` | [配方](../../../raw/current/scripts/recipes.lua#L249) |
| 竞速鞍 `saddle_race` | `butterflywings` **68**＋`livinglog` 2＋`silk` 4 | `TECH.SCIENCE_TWO`；这是检索范围内蝶翅消耗最大的一条 | [配方](../../../raw/current/scripts/recipes.lua#L684) |
| 波兰饼干 `wintercooking_polishcookie` | `butterflywings` 2＋`wintersfeastfuel` 1 | 冬季盛宴，`manufactured`＋`actionstr = "COOK"`（烤制类制造，不是炖锅） | [配方](../../../raw/current/scripts/recipes.lua#L1441) |
| 月之书 `book_moon` | `moonbutterflywings` 2＋`papyrus` 2＋`opalpreciousgem` 1 | `builder_tag = "bookbuilder"` | [配方](../../../raw/current/scripts/recipes.lua#L286) |
| 战斗歌·回理智 `battlesong_sanitygain` | `moonbutterflywings` 1＋`papyrus` 1＋`featherpencil` 1 | `builder_tag = "battlesinger"` | [配方](../../../raw/current/scripts/recipes.lua#L182) |
| 万圣节月亮实验 `halloween_experiment_moon` | `moonbutterflywings` 1＋`moon_tree_blossom` 1＋10 理智 | `MADSCIENCE_ONE`，产物是能把蝴蝶变月蛾的 `halloweenpotion_moon` | [配方](../../../raw/current/scripts/recipes.lua#L1420) |

- 蝴蝶松饼是本页唯一的"食材"用途：因为蝶翅只给 `decoration`，`veggie >= 0.5` 必须由另一样 `veggie` 食材提供（浆果只给 `fruit = .5`，不顶菜类）；把 4 件蝶翅直接丢锅不会出松饼。[食材标签](../../../raw/current/scripts/cooking.lua#L131-L132)、[判定](../../../raw/current/scripts/preparedfoods.lua#L5)
- 两件都带 `cattoy` 标签：浣猫的收货判定把 `cattoy`／`catfood`／`cattoyairborne` 都当可收下的东西，行为树也把这三类当玩具目标。[收货](../../../raw/current/scripts/prefabs/catcoon.lua#L273-L279)、[玩具目标](../../../raw/current/scripts/brains/catcoonbrain.lua#L24-L48)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`edible` | 三维与 `VEGGIE` 食性 | 主模拟；月蛾翅多一个正的 `sanityvalue` | [[reference/components/edible]]；[赋值](../../../raw/current/scripts/prefabs/moonbutterflywings.lua#L34-L38) |
| 组件：`perishable` | 6 天，腐坏成 `spoiled_food` | 主模拟 | [[reference/components/perishable]]；[赋值](../../../raw/current/scripts/prefabs/butterflywings.lua#L49-L52) |
| 组件：`stackable` | 显式 40 | 主模拟 | [[reference/components/stackable]]；[赋值](../../../raw/current/scripts/prefabs/butterflywings.lua#L41-L42) |
| 组件：`floater` | 水上漂浮外观，`small` 0.03 | 主模拟（`MakeInventoryFloatable` 在 pristine 前执行） | [[reference/components/floater]]；[调用](../../../raw/current/scripts/prefabs/butterflywings.lua#L28) |
| 组件：`hauntable` | `MakeHauntableLaunchAndPerish`；实际只抛飞（腐坏分支被注释） | 主模拟 | [[reference/components/hauntable]]；[注释段](../../../raw/current/scripts/standardcomponents.lua#L1311-L1337) |
| 标签：`cattoy` | 浣猫收货与玩具目标 | 运行期读取；两件只有标签、没有 `cattoy` 组件 | [收货](../../../raw/current/scripts/prefabs/catcoon.lua#L273-L279)、[玩玩具](../../../raw/current/scripts/actions.lua#L3882-L3894) |
| 数据：`cooking.lua` | `decoration = 2`，不提供 veggie | 加载期注册 | [登记](../../../raw/current/scripts/cooking.lua#L131-L132) |
| 关联 Prefab：`butterflymuffin` | 唯一消耗蝶翅的炖锅菜 | 需要另有素食 | Wiki 待建；[判定](../../../raw/current/scripts/preparedfoods.lua#L3-L16) |
| 关联 Prefab：`butterfly`／`moonbutterfly` | 掉落来源；后者也是温蒂配方与月亮实验的产物 | [[reference/prefabs/butterfly]]、[[reference/prefabs/moonbutterfly]] | [[reference/prefabs/butterfly]]；[月蛾](../../../raw/current/scripts/prefabs/moonbutterfly.lua#L104-L218) |
| 资源：`anim/butterfly_wings.zip`／`anim/moonbutterfly_wings.zip` | 物品动画包 | 只核验声明，未验证资源内容 | [蝶翅 assets](../../../raw/current/scripts/prefabs/butterflywings.lua#L1-L4)、[月蛾翅 assets](../../../raw/current/scripts/prefabs/moonbutterflywings.lua#L1-L4) |

### 共享规则接入（S1–S7）

- **S1 烤制门禁**：不适用——两件都没有 `cookable`，物品侧门禁不成立，不能上任何带 `cooker` 的实体；门禁本身为：目标带 `cooker`、未 `fueldepleted`，`dangerouscooker` 需 `doer:HasTag("expertchef")`，骑乘时目标须归自己所有（[门禁](../../../raw/current/scripts/componentactions.lua#L1096-L1105)）。
- **S2 燃烧掉落**：不适用——两件不是掉落者；来源蝴蝶被烧时的掉落改写见[[reference/prefabs/butterfly]]的"共享规则接入"。
- **S3 晾晒**：不适用——两件都没有 `dryable` 组件与标签。
- **S4 烹饪登记**：`decoration = 2`（两件同值）；登记未传第三参，不派生 `*_cooked` 食材条目。[登记](../../../raw/current/scripts/cooking.lua#L131-L132)
- **S5 堆叠与标签**：两件都显式 `STACK_SIZE_SMALLITEM`＝40；显式标签只有 `cattoy`（没有 `cattoy` 组件），运行期另有组件自动加的 `edible_VEGGIE` 与鲜度三态。[标签](../../../raw/current/scripts/prefabs/butterflywings.lua#L26-L28)、[堆叠](../../../raw/current/scripts/prefabs/moonbutterflywings.lua#L40-L41)
- **S6 腐坏替换**：两件都替换成 `spoiled_food`。[替换](../../../raw/current/scripts/components/perishable.lua#L277-L319)
- **S7 索引**：索引由主代理统一登记，本页不改 `index.md`。

## 常见陷阱与限制

- **误区：两件蝶翅是"素食食材"**。炖锅侧只记 `decoration = 2`，不提供 `veggie`；蝴蝶松饼的 0.5 菜类得另外凑。[登记](../../../raw/current/scripts/cooking.lua#L131-L132)
- **边界：蝶翅不能烤、不能晾**。两件都没有 `cookable`／`dryable` 组件与标签；烤制除了目标要带 `cooker`，物品自己也得带 `cookable`。[组件段](../../../raw/current/scripts/prefabs/butterflywings.lua#L36-L52)、[烤制门禁](../../../raw/current/scripts/componentactions.lua#L1096-L1105)
- **边界：两件的差别只有理智、贴图与拾取音**。三维其余项、鲜度、堆叠、浮空参数逐项相同，别把"月蛾翅更强"扩写成别的属性。[对照](#两件对照)
- **边界：`cattoy` 只是标签**。真正"被玩"的效果要看目标有没有 `cattoy` 组件（`CATPLAYGROUND` 先查组件，没有组件再走别的分支），蝶翅只有标签。[动作](../../../raw/current/scripts/actions.lua#L3882-L3894)
- **未核验**：月蛾本体行为（B07）、`butterflymuffin` 的完整优先级竞争、两件作为交易品的具体数值（`tradable` 只有组件，本页未查 goldvalue）。

## 验证记录

- 源码审阅：2026-11-09，Steam buildid 26474873。核对 `prefabs/butterflywings.lua` 与 `prefabs/moonbutterflywings.lua` 全文、`prefablist.lua` 两条登记；`prefabs/butterfly.lua` 的随机掉落、`prefabs/moonbutterfly.lua` 的共享表与外形；`components/edible.lua` 的默认值与 `badfood`、`components/perishable.lua` 的替换、`components/stackable.lua` 的默认值、`components/lootdropper.lua` 的加权与 `chance >= 1.0` 分支；`prefabs/tumbleweed.lua` 的掉落权重表与 3 件抽取；`cooking.lua` 的两条食材登记与标签累加；`preparedfoods.lua` 的蝴蝶松饼；`recipes.lua` 的 7 条相关配方；`prefabs/catcoon.lua`／`brains/catcoonbrain.lua`／`actions.lua` 的 `cattoy` 链路；`tuning.lua` 的数值；`chinese_s.po` 与 `scrapbookdata.lua` 的名称与记录。
  未覆盖：月蛾本体、蝴蝶松饼的优先级竞争、`tradable` 的 goldvalue、两件在礼物表中的权重。风滚草权重表已核（权重 1／权重和约 100／抽 3 件）。
- 游戏运行：未运行。天数与三维由 `tuning` 常量推导。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 通过（见交付回报）。

## 相关链接

- 来源生物：[[reference/prefabs/butterfly|蝴蝶]]
- 食材与鲜度：[[reference/components/edible]]、[[reference/components/perishable]]、[[reference/components/stackable]]
- 同批页面：[[reference/prefabs/butter|黄油]]、[[reference/prefabs/honey|蜂蜜与蜂王浆]]、[[reference/prefabs/milkywhites|乳白物]]

返回 [[reference/prefabs/index|Prefab 索引]]。
