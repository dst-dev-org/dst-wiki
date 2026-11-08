---
title: "黄油（butter）"
created: "2026-11-09"
updated: "2026-11-09"
type: prefab
summary: "蝴蝶随机掉落里那 0.1 权重换来的黄油：三维 +40／+25、40 天鲜度、堆叠 40、没有显式标签（只有组件自动加的 `edible_GENERIC` 与鲜度三态）；炖锅侧记 fat=1 与 dairy=1，既是华夫饼与龙虾正餐点名的原料，也能顶冰淇淋与蝙蝠鼻帽的乳制品位，同时会把蔬菜煎蛋与牛饲料排除掉。"
aliases: ["butter", "黄油"]
tags: [prefab, item, food]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/butter.lua", "prefabs/butterfly.lua", "components/edible.lua", "components/perishable.lua", "components/stackable.lua", "components/lootdropper.lua", "components/container.lua", "cooking.lua", "preparedfoods.lua", "preparednonfoods.lua", "containers.lua", "standardcomponents.lua", "componentactions.lua", "prefabs/spoiledfood.lua", "prefablist.lua", "tuning.lua", "languages/chinese_s.po", "screens/redux/scrapbookdata.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 黄油（butter）

## 快速判断

- 职责与适用：`butter` 是可拾取、可食用的物品（简中实显 **黄油**），单文件单注册，依赖表只声明 `spoiled_food`。它没有专属机制，价值几乎全在"炖锅里的 `fat`／`dairy`"与两个点名菜上。[构造](../../../raw/current/scripts/prefabs/butter.lua#L11-L52)
- 优先使用：数值／鲜度看[数值](#数值)；怎么来、概率多少看[获得途径](#获得途径)；能做什么菜、会把什么菜顶掉看[烹饪用途](#烹饪用途)。
- 重要副作用：炖锅判定里它同时给 `fat = 1` 与 `dairy = 1`，因此既能让华夫饼与龙虾正餐成立，也能顶掉冰淇淋与蝙蝠鼻帽的乳制品位；反过来，`fat`／`dairy` 会把蔬菜煎蛋与牛饲料排除。
- 修改边界：只覆盖这一件物品。掉落它的蝴蝶归 [[reference/prefabs/butterfly|蝴蝶页]]，共用同一次随机抽取的蝶翅归 [[reference/prefabs/butterfly-wings|蝶翅页]]；具体菜谱的完整竞争关系归炖锅机制页。

## 注册与构造

`prefablist.lua` 以文件名 `"butter"` 登记，文件只 `return Prefab("butter", ...)`。[登记](../../../raw/current/scripts/prefablist.lua#L190)、[导出](../../../raw/current/scripts/prefabs/butter.lua#L54)

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `butter` | `fn()`：SetBank／SetBuild `butter`、播 `idle`；**没有**任何 `AddTag`；`MakeInventoryFloatable(inst)` 用默认参数（`small`、无偏移、无缩放）；`SetPristine()` 后进主模拟 | 主模拟：`stackable`（显式 40）→ `inspectable` → `inventoryitem`／`tradable` → `edible`（`HEALING_LARGE`／`CALORIES_MED`，不写理智）→ `perishable`（`PERISH_SUPERSLOW`→`spoiled_food`）→ `MakeHauntableLaunchAndPerish`。[构造](../../../raw/current/scripts/prefabs/butter.lua#L11-L52) |

- 本批六件物品（蜂蜜、蜂王浆、两种蝶翅、黄油、乳白物）里，黄油是唯一**没有显式标签**的一件：`honeyed`、`catfood`、`cattoy`、`cookable`、`dryable` 都不加，所以它既不吸引熊獾、也不是浣猫玩具、还不能烤或晾；运行期仍会自动带 `edible_GENERIC`（`edible` 的食性标签）与 `fresh`／`stale`／`spoiled`（`perishable` 的鲜度标签）。[全文](../../../raw/current/scripts/prefabs/butter.lua#L11-L52)、[食性标签](../../../raw/current/scripts/components/edible.lua#L9-L17)、[鲜度标签](../../../raw/current/scripts/components/perishable.lua#L1-L27)
- 浮空参数走默认：`MakeInventoryFloatable` 的 `size` 为空时取 `"small"`，不传 offset／scale 就不设。[默认值](../../../raw/current/scripts/standardcomponents.lua#L1772-L1791)
- `edible` 只写了生命与饥饿，`foodtype` 落构造默认 `FOODTYPE.GENERIC`；生命为正、理智为 0，不会被打 `badfood`。[默认食性](../../../raw/current/scripts/components/edible.lua#L19-L24)、[badfood 判定](../../../raw/current/scripts/components/edible.lua#L1-L7)

### 名称核验

`NAMES.BUTTER`＝黄油；与 `BUTTERFLY`（蝴蝶）、`BUTTERFLYMUFFIN`（蝴蝶松饼）在名称表里相邻但互不指代。[BUTTER](../../../raw/current/scripts/languages/chinese_s.po#L340019-L340022)

## 数值

1 天 = 480 秒。[时间基准](../../../raw/current/scripts/tuning.lua#L19-L20)

| 项 | 值 | 常量与证据 |
|---|---|---|
| 生命 | +40 | `HEALING_LARGE`[赋值](../../../raw/current/scripts/prefabs/butter.lua#L41) |
| 饥饿 | +25 | `CALORIES_MED`[赋值](../../../raw/current/scripts/prefabs/butter.lua#L42) |
| 理智 | 0（未写） | 组件默认[默认值](../../../raw/current/scripts/components/edible.lua#L19-L24) |
| 食性 | GENERIC（未写） | 同上 |
| 鲜度 | 40 天＝19200 秒 | `PERISH_SUPERSLOW`[赋值](../../../raw/current/scripts/prefabs/butter.lua#L45)、[常量](../../../raw/current/scripts/tuning.lua#L2051) |
| 腐坏产物 | `spoiled_food` | [赋值](../../../raw/current/scripts/prefabs/butter.lua#L47)、[注册](../../../raw/current/scripts/prefabs/spoiledfood.lua#L263) |
| 堆叠 | 40 | `STACK_SIZE_SMALLITEM`[赋值](../../../raw/current/scripts/prefabs/butter.lua#L32-L33)、[常量](../../../raw/current/scripts/tuning.lua#L83) |

- 图鉴记录与上表一致（`hungervalue` 25／`healthvalue` 40／`sanityvalue` 0／`perishable` 19200／`stacksize` 40），可作交叉核对。[图鉴](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L246)
- 腐坏时 `perishable:Perish()` 生成 `spoiled_food`，按原叠数同步后交给持有者的原槽位；容器按 `itemtestfn` 拒收时掉在原地。[替换](../../../raw/current/scripts/components/perishable.lua#L301-L312)、[入槽校验](../../../raw/current/scripts/components/container.lua#L295)、[拒收落地](../../../raw/current/scripts/components/container.lua#L560-L566)

## 获得途径

| 来源 | 产出 | 条件 | 证据 |
|---|---|---|---|
| 蝴蝶 `butterfly` | 权重 0.1，与蝶翅权重 5 共用**一次**抽取（`numrandomloot = 1`），实际约 1.96% | 击杀；掉落者不是"已烧毁／燃烧中"状态 | [配置](../../../raw/current/scripts/prefabs/butterfly.lua#L158-L162)、[抽取](../../../raw/current/scripts/components/lootdropper.lua#L99-L112) |

- 检索范围内除 `butter.lua` 自己的 bank／build 外，只有 `butterfly.lua` 引用 `"butter"`，没有第二个生成点；也没有任何 `Recipe2` 用黄油当材料，它是纯掉落物。
- 抓在背包里的蝴蝶 3 天后饿死（活体 perish 计时，非饥饿）时走的是**同一张随机表**（`GenerateLoot` 里同样只抽 1 件），所以饿死也可能掉黄油，且可能因容器塞不下而落在地上；细节见[[reference/prefabs/butterfly]]的随身计时。
- **烧死的蝴蝶不给黄油**：掉落者已带 `burnt`，或燃烧中且（无 `fueled` 组件或 `burnable.ignorefuel` 为真）时按 `DropLoot` 改写；黄油没有 `butter_cooked`／`cookedbutter` 变体、也不在 `BURNED_LOOT_OVERRIDES` 里，最终换成 `ash`。[改写链](../../../raw/current/scripts/components/lootdropper.lua#L415-L442)、[覆盖表](../../../raw/current/scripts/tuning.lua#L4511-L4530)

## 烹饪用途

`cooking.lua` 给黄油登记两个标签：`fat = 1`、`dairy = 1`。[食材登记](../../../raw/current/scripts/cooking.lua#L133)

炖锅按**格**计：普通锅、便携锅与档案锅共用 `params.cookpot`，`acceptsstacks = false`、4 个投料格，所以两份黄油各占一格、`fat`／`dairy` 各累计 2；`GetIngredientValues` 逐格把标签值相加。[锅参数](../../../raw/current/scripts/containers.lua#L348-L371)、[三种锅共用](../../../raw/current/scripts/containers.lua#L389-L390)、[累加](../../../raw/current/scripts/cooking.lua#L217-L231)

| 菜 | 黄油的作用 | 判定 | 证据 |
|---|---|---|---|
| 华夫饼 `waffles` | **点名原料** | `names.butter and (浆果类) and tags.egg`；优先级 10 | [判定](../../../raw/current/scripts/preparedfoods.lua#L294-L303) |
| 龙虾正餐 `lobsterdinner` | **点名原料** | `names.wobster_sheller_land and names.butter and tags.meat >= 1 and tags.fish >= 1 and not tags.frozen` | [判定](../../../raw/current/scripts/preparedfoods.lua#L691-L700) |
| 冰淇淋 `icecream` | 提供 `dairy` | `tags.frozen and tags.dairy and tags.sweetener` 且无肉／菜／蛋／不可食 | [判定](../../../raw/current/scripts/preparedfoods.lua#L379-L387) |
| 蝙蝠鼻帽 `batnosehat` | 提供 `dairy` | `names.batnose and names.kelp and tags.dairy >= 1`；炖锅出装备 | [判定](../../../raw/current/scripts/preparednonfoods.lua#L7-L27) |
| 蔬菜煎蛋 `veggieomlet` | **反作用**：`not tags.dairy` | 放了黄油就不再是这道菜 | [判定](../../../raw/current/scripts/preparedfoods.lua#L1005-L1007) |
| 牛饲料 `beefalofeed` | **反作用**：`not tags.fat and not tags.dairy` | 同上 | [判定](../../../raw/current/scripts/preparedfoods.lua#L1034-L1037) |
| 甜茶 `sweettea` | **反作用**：`not tags.fat and not tags.dairy` | 同上 | [判定](../../../raw/current/scripts/preparedfoods.lua#L879-L881) |

- 两个"点名"判定都读 `names.butter`（食材名）；投料按格进行——`acceptsstacks = false` 时整叠不能一次放进，要拆成单件逐格投，每格 1 件，两份各占一格，`fat`／`dairy` 按格累加为 2。[登记](../../../raw/current/scripts/cooking.lua#L133)、[锅参数](../../../raw/current/scripts/containers.lua#L369)、[累加规则](../../../raw/current/scripts/cooking.lua#L217-L231)
- 黄油没有 `cookable`，不能像鱼肉那样上火烤成"熟黄油"；它的所有去向只有生吃、下炖锅、腐坏。[组件段](../../../raw/current/scripts/prefabs/butter.lua#L32-L49)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`edible` | 三维与默认 `GENERIC` 食性 | 主模拟 | [[reference/components/edible]]；[赋值](../../../raw/current/scripts/prefabs/butter.lua#L40-L42) |
| 组件：`perishable` | 40 天，腐坏成 `spoiled_food` | 主模拟 | [[reference/components/perishable]]；[赋值](../../../raw/current/scripts/prefabs/butter.lua#L44-L47) |
| 组件：`stackable` | 显式 40 | 主模拟 | [[reference/components/stackable]]；[赋值](../../../raw/current/scripts/prefabs/butter.lua#L32-L33) |
| 组件：`floater` | 走默认 `small` | pristine 前调用 | [[reference/components/floater]]；[调用](../../../raw/current/scripts/prefabs/butter.lua#L24) |
| 组件：`hauntable` | `MakeHauntableLaunchAndPerish`；实际只抛飞（腐坏分支被注释） | 主模拟 | [[reference/components/hauntable]]；[注释段](../../../raw/current/scripts/standardcomponents.lua#L1311-L1337) |
| 数据：`cooking.lua` | `fat = 1`、`dairy = 1` | 加载期注册 | [登记](../../../raw/current/scripts/cooking.lua#L133) |
| 关联 Prefab：`butterfly` | 击杀或饿死共用同一随机表（各抽 1 件） | 随机表 1 件 | [[reference/prefabs/butterfly]]；[配置](../../../raw/current/scripts/prefabs/butterfly.lua#L160) |
| 关联 Prefab：`butterflywings` | 与黄油共用同一次抽取，二者互斥 | 见蝶翅页 | [[reference/prefabs/butterfly-wings]]；[抽取](../../../raw/current/scripts/components/lootdropper.lua#L99-L112) |
| 关联 Prefab：`waffles`／`lobsterdinner` | 点名用黄油的成品 | 炖锅 | Wiki 待建；[华夫饼](../../../raw/current/scripts/preparedfoods.lua#L294-L303)、[龙虾正餐](../../../raw/current/scripts/preparedfoods.lua#L691-L700) |
| 资源：`anim/butter.zip` | 物品动画包 | 只核验声明，未验证资源内容 | [assets](../../../raw/current/scripts/prefabs/butter.lua#L1-L4) |

### 共享规则接入（S1–S7）

- **S1 烤制门禁**：不适用——黄油没有 `cookable`，物品侧门禁不成立，不能放到带 `cooker` 的实体上烤；门禁本身为：目标带 `cooker`、未 `fueldepleted`，`dangerouscooker` 需 `doer:HasTag("expertchef")`，骑乘时目标须归自己所有（[门禁](../../../raw/current/scripts/componentactions.lua#L1096-L1105)）。
- **S2 燃烧掉落**：接入点在来源侧——黄油由蝴蝶掉落；蝴蝶已 `burnt` 或正在燃烧且（`fueled == nil` 或 `ignorefuel`）时，没有 `butter_cooked`／`cookedbutter`，最终换成 `ash`。[改写链](../../../raw/current/scripts/components/lootdropper.lua#L415-L442)
- **S3 晾晒**：不适用——黄油没有 `dryable` 组件与标签。
- **S4 烹饪登记**：`fat = 1`、`dairy = 1`，登记未传第三参，不派生 `butter_cooked` 条目。[登记](../../../raw/current/scripts/cooking.lua#L133)
- **S5 堆叠与标签**：显式 `STACK_SIZE_SMALLITEM`＝40；**显式标签为空**（`honeyed`／`catfood`／`cattoy`／`cookable`／`dryable` 都不加），运行期由组件加 `edible_GENERIC` 与 `fresh`／`stale`／`spoiled`。[堆叠](../../../raw/current/scripts/prefabs/butter.lua#L32-L33)、[edible](../../../raw/current/scripts/components/edible.lua#L9-L17)、[perishable](../../../raw/current/scripts/components/perishable.lua#L1-L27)
- **S6 腐坏替换**：替换成 `spoiled_food`。[替换](../../../raw/current/scripts/components/perishable.lua#L277-L319)
- **S7 索引**：索引由主代理统一登记，本页不改 `index.md`。

## 常见陷阱与限制

- **误区：`0.1` 是 10% 掉落率**。它是权重，与蝶翅的 5 一起归一化成约 1.96%，且两者互斥（一次只抽 1 件）。[抽取](../../../raw/current/scripts/components/lootdropper.lua#L99-L112)
- **误区：有 `dairy` 的食材可以随便加**。蔬菜煎蛋 `veggieomlet`、牛饲料、甜茶都显式排除 `dairy`／`fat`，加黄油会把它们做成别的菜。[蔬菜煎蛋](../../../raw/current/scripts/preparedfoods.lua#L1005-L1007)、[牛饲料](../../../raw/current/scripts/preparedfoods.lua#L1034-L1037)、[甜茶](../../../raw/current/scripts/preparedfoods.lua#L879-L881)
- **边界：黄油不是浣猫玩具、也不吸引熊獾**。它没有显式标签，`honeyed`／`cattoy`／`catfood` 都不带（自动带的 `edible_GENERIC`／鲜度三态不是这三类）。[全文](../../../raw/current/scripts/prefabs/butter.lua#L11-L52)
- **边界：普通击杀或饿死都可能抽到黄油**；只有符合燃烧改写条件时（掉落者已 `burnt`，或燃烧中且未被 `fueled` 阻止）掉落才被改写成灰烬。[改写链](../../../raw/current/scripts/components/lootdropper.lua#L415-L442)
- **边界：`MakeHauntableLaunchAndPerish` 实际不腐坏**，作祟只会把它抛飞。[注释段](../../../raw/current/scripts/standardcomponents.lua#L1311-L1337)
- **未核验**：华夫饼与龙虾正餐以外的优先级竞争（如与 `waffles` 同分的菜）、黄油在交易表里的 `goldvalue`。

## 验证记录

- 源码审阅：2026-11-09，Steam buildid 26474873。核对 `prefabs/butter.lua` 全文与 `prefablist.lua` 登记；`prefabs/butterfly.lua` 的随机掉落；`components/edible.lua`／`components/perishable.lua`／`components/stackable.lua`／`components/lootdropper.lua` 的相关分支；`standardcomponents.lua` 的 `MakeInventoryFloatable` 默认值与 `MakeHauntableLaunchAndPerish`；`cooking.lua` 的食材登记与标签累加；`preparedfoods.lua` 的华夫饼／龙虾正餐／冰淇淋／蔬菜煎蛋／牛饲料／甜茶；`preparednonfoods.lua` 的蝙蝠鼻帽；`componentactions.lua` 的烤制门禁；`tuning.lua` 的常量与 `BURNED_LOOT_OVERRIDES`；`chinese_s.po` 与 `scrapbookdata.lua` 的名称与记录。
  未覆盖：炖锅优先级竞争全表、`tradable` 的 goldvalue、炖锅以外的消耗方。
- 游戏运行：未运行。三维、天数与概率由 `tuning` 常量与权重表推导。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 通过（见交付回报）。

## 相关链接

- 来源：[[reference/prefabs/butterfly|蝴蝶]]
- 共用抽取：[[reference/prefabs/butterfly-wings|蝴蝶翅膀与月蛾翅膀]]
- 食材与鲜度：[[reference/components/edible]]、[[reference/components/perishable]]
- 同批页面：[[reference/prefabs/honey|蜂蜜与蜂王浆]]、[[reference/prefabs/milkywhites|乳白物]]

返回 [[reference/prefabs/index|Prefab 索引]]。
