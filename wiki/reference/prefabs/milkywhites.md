---
title: "乳白物（milkywhites）"
created: "2026-11-09"
updated: "2026-11-09"
type: prefab
summary: "恐怖之眼战利品里的乳白物：三维 +8／+12.5／−20 与由此产生的 badfood、10 天鲜度、堆叠 40、catfood 标签；炖锅侧只记 dairy=1（冰淇淋与蝙蝠鼻帽可用），另外是恐怖之眼小宠物建造与眼罩帽拆解的原料。"
aliases: ["milkywhites", "乳白物"]
tags: [prefab, item, food]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/milkywhites.lua", "prefabs/eyeofterror.lua", "prefabs/hats.lua", "prefabs/staff_common.lua", "components/edible.lua", "components/perishable.lua", "components/stackable.lua", "components/lootdropper.lua", "components/container.lua", "components/eater.lua", "cooking.lua", "preparedfoods.lua", "preparednonfoods.lua", "recipes.lua", "componentactions.lua", "standardcomponents.lua", "prefabs/catcoon.lua", "prefabs/spoiledfood.lua", "constants.lua", "prefablist.lua", "tuning.lua", "languages/chinese_s.po", "screens/redux/scrapbookdata.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 乳白物（milkywhites）

## 快速判断

- 职责与适用：`milkywhites` 是可拾取、可食用、也可腐坏的物品（简中实显 **乳白物**），单文件单注册，`prefabs` 依赖表为 `nil`。[构造](../../../raw/current/scripts/prefabs/milkywhites.lua#L9-L53)
- 优先使用：数值／鲜度看[数值](#数值)；掉落与概率看[获得途径](#获得途径)；能做什么、为什么喂不了人看[用途与边界](#用途与边界)。
- 重要副作用：食用 −20 理智，因此被 `edible` 打上 `badfood`：非 PVP 下不能把它喂给其他玩家（目标带 `possessedbody` 时另有直通分支）；炖锅侧只给 `dairy = 1`，是冰淇淋与蝙蝠鼻帽的乳制品位备选。
- 修改边界：只覆盖这一件物品与其消耗入口。恐怖之眼本体（`eyeofterror`）与其掉落表的行为归后续批次；眼罩帽 `eyemaskhat` 的装备机制归帽类页，本页只写拆解来源。

## 注册与构造

`prefablist.lua` 以文件名 `"milkywhites"` 登记，文件只导出 1 个注册。[登记](../../../raw/current/scripts/prefablist.lua#L759)、[导出](../../../raw/current/scripts/prefabs/milkywhites.lua#L55)

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `milkywhites` | `fn()`：SetBank／SetBuild `milkywhites`、播 `idle`；pristine **之前**加 `catfood` 标签、`MakeInventoryFloatable(inst)` 用默认参数（`small`）；`SetPristine()` 后进主模拟 | 主模拟：`edible`（`HEALING_MEDSMALL`／`CALORIES_SMALL`／`-SANITY_MEDLARGE`）→ `inspectable` → `inventoryitem` → `perishable`（`PERISH_MED`→`spoiled_food`）→ `stackable`（40）→ `tradable` → `MakeHauntableLaunchAndPerish`。[构造](../../../raw/current/scripts/prefabs/milkywhites.lua#L9-L53) |

- `catfood` 加在 pristine 之前，客户端也认；它没有 `honeyed`、`cattoy`、`cookable`、`dryable`，所以不吸引熊獾、不是浣猫玩具、不能烤也不能晾。[标签与组件](../../../raw/current/scripts/prefabs/milkywhites.lua#L22-L46)
- 该文件除动画包外还声明了 `Asset("INV_IMAGE", "milkywhites")`，即背包图标单独出图；本批其余四页的物品都没有这条声明。[assets](../../../raw/current/scripts/prefabs/milkywhites.lua#L1-L5)
- `edible` 未写 `foodtype`，落默认 `FOODTYPE.GENERIC`；`sanityvalue < 0` 会让 `oncheckbadfood` 打上 `badfood`。[默认食性](../../../raw/current/scripts/components/edible.lua#L19-L24)、[badfood 判定](../../../raw/current/scripts/components/edible.lua#L1-L7)

### 名称核验

`NAMES.MILKYWHITES`＝乳白物（不是"乳白液"之类），与产出方 `NAMES.EYEOFTERROR`（恐怖之眼）不是同一键。[MILKYWHITES](../../../raw/current/scripts/languages/chinese_s.po#L345434-L345437)

## 数值

1 天 = 480 秒。[时间基准](../../../raw/current/scripts/tuning.lua#L19-L20)

| 项 | 值 | 常量与证据 |
|---|---|---|
| 生命 | +8 | `HEALING_MEDSMALL`[赋值](../../../raw/current/scripts/prefabs/milkywhites.lua#L32) |
| 饥饿 | +12.5 | `CALORIES_SMALL`[赋值](../../../raw/current/scripts/prefabs/milkywhites.lua#L33) |
| 理智 | **−20** | `-SANITY_MEDLARGE`[赋值](../../../raw/current/scripts/prefabs/milkywhites.lua#L34)、[常量](../../../raw/current/scripts/tuning.lua#L2036) |
| 食性 | GENERIC（未写） | 组件默认[默认值](../../../raw/current/scripts/components/edible.lua#L19-L24) |
| 鲜度 | 10 天＝4800 秒 | `PERISH_MED`[赋值](../../../raw/current/scripts/prefabs/milkywhites.lua#L41)、[常量](../../../raw/current/scripts/tuning.lua#L2048) |
| 腐坏产物 | `spoiled_food` | [赋值](../../../raw/current/scripts/prefabs/milkywhites.lua#L43)、[注册](../../../raw/current/scripts/prefabs/spoiledfood.lua#L263) |
| 堆叠 | 40 | `STACK_SIZE_SMALLITEM`[赋值](../../../raw/current/scripts/prefabs/milkywhites.lua#L45-L46)、[常量](../../../raw/current/scripts/tuning.lua#L83) |

- 图鉴记录与上表一致（`hungervalue` 12.5／`healthvalue` 8／`sanityvalue` −20／`perishable` 4800／`stacksize` 40），可作交叉核对。[图鉴](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L894)
- 腐坏时 `perishable:Perish()` 生成 `spoiled_food`，按原叠数同步后交给持有者的原槽位；容器按 `itemtestfn` 拒收时掉在原地。[替换](../../../raw/current/scripts/components/perishable.lua#L301-L312)、[入槽校验](../../../raw/current/scripts/components/container.lua#L295)、[拒收落地](../../../raw/current/scripts/components/container.lua#L560-L566)
- 负理智只在 `sanityvalue < 0` 或 `healthvalue < 0` 时触发 `badfood`；正生命不会抵消这条判定。[判定](../../../raw/current/scripts/components/edible.lua#L1-L7)

## 获得途径

| 来源 | 产出 | 条件 | 证据 |
|---|---|---|---|
| 恐怖之眼 `eyeofterror` | 3 条记 1.00 ＋ 2 条记 0.50，期望 4 个 | `SetSharedLootTable`；每条独立判定，`chance >= 1.0` 直接给 | [战利品表](../../../raw/current/scripts/prefabs/eyeofterror.lua#L461-L474)、[必给分支](../../../raw/current/scripts/components/lootdropper.lua#L213-L227) |
| 眼罩帽 `eyemaskhat` 拆解 | 3 个 `milkywhites` ＋ 2 个 `monstermeat` | `DeconstructRecipe`；按护甲耐久比例折算，每项至少 1 个 | [拆解表](../../../raw/current/scripts/recipes.lua#L1678)、[折算](../../../raw/current/scripts/prefabs/staff_common.lua#L637-L649)、[生成](../../../raw/current/scripts/prefabs/staff_common.lua#L663-L674) |

- 恐怖之眼每次击杀掉 **3 个必给 + 2 个半概率**，不是固定的 5 个；按期望算 4 个，运气差时只有 3 个。[战利品表](../../../raw/current/scripts/prefabs/eyeofterror.lua#L461-L474)
- 眼罩帽本体是 `MakeHat("eyemask")`，护甲按橄榄球盔（`ARMOR_FOOTBALLHAT`）初始化，所以拆解收益随耐久下降。[注册](../../../raw/current/scripts/prefabs/hats.lua#L7903)、[构造](../../../raw/current/scripts/prefabs/hats.lua#L2343-L2367)
- 本页物品没有 `cookable`／`dryable`：不能烤成"熟乳白物"，也不能上晾晒架，所有去向只有生吃、下炖锅、腐坏、当材料。[组件段](../../../raw/current/scripts/prefabs/milkywhites.lua#L31-L50)

## 用途与边界

| 用途 | 消耗 | 关键条件 | 证据 |
|---|---|---|---|
| 冰淇淋 `icecream` | 提供 `dairy` | `tags.frozen and tags.dairy and tags.sweetener` 且无肉／菜／蛋／不可食 | [判定](../../../raw/current/scripts/preparedfoods.lua#L379-L387) |
| 蝙蝠鼻帽 `batnosehat` | 提供 `dairy >= 1` | `names.batnose and names.kelp and tags.dairy >= 1`；炖锅出装备 | [判定](../../../raw/current/scripts/preparednonfoods.lua#L7-L27) |
| 恐怖之眼小宠物 `critter_eyeofterror_builder` | 1 个 `milkywhites`＋1 个 `baconeggs` | `TECH.ORPHANAGE_ONE`，`actionstr = "ORPHANAGE"`（寄养所建造） | [配方](../../../raw/current/scripts/recipes.lua#L827) |
| 喂给浣猫 | 作为 `catfood` 被收下 | 浣猫收货判定认 `catfood` 标签 | [收货](../../../raw/current/scripts/prefabs/catcoon.lua#L273-L279) |

- 炖锅侧只有 `dairy = 1` 一个标签，没有 `fat`、也没有 `sweetener`，所以它单独不能满足任何"点名乳制品"以外的菜；冰淇淋还需要 `frozen` 与 `sweetener`。[食材登记](../../../raw/current/scripts/cooking.lua#L148)
- **边界：非 PVP 下喂不了别的玩家**。`FEEDPLAYER` 的生成条件里有一条"食物不带 `badfood`／`unsafefood`／`spoiled`"，乳白物带 `badfood`，只能自己吃、喂宠物或丢锅；但目标带 `possessedbody` 时另有一条直通分支、不受该限制。[喂食门禁](../../../raw/current/scripts/componentactions.lua#L1164-L1175)
- 浣猫侧只认标签，不认数值：喂它 `catfood` 与"它爱吃"是两件事，实际收益走 `eater` 的通用结算。[收货](../../../raw/current/scripts/prefabs/catcoon.lua#L273-L279)、[Eat](../../../raw/current/scripts/components/eater.lua#L198-L286)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`edible` | 三维（含 −20 理智）与默认 `GENERIC` 食性；因此带 `badfood` | 主模拟 | [[reference/components/edible]]；[赋值](../../../raw/current/scripts/prefabs/milkywhites.lua#L31-L34) |
| 组件：`perishable` | 10 天，腐坏成 `spoiled_food` | 主模拟 | [[reference/components/perishable]]；[赋值](../../../raw/current/scripts/prefabs/milkywhites.lua#L40-L43) |
| 组件：`stackable` | 显式 40 | 主模拟 | [[reference/components/stackable]]；[赋值](../../../raw/current/scripts/prefabs/milkywhites.lua#L45-L46) |
| 组件：`eater`（食用方） | 负理智照常结算；`badfood` 不阻止自己吃 | 主模拟；`DoFoodEffects` 返回"是否应用食物效果"，普通食用者恒真；只有强胃＋怪物肉、`eatsrawmeat`＋生肉或食物亲和命中时为假，才跳过理智 | [[reference/components/eater]]；[判定](../../../raw/current/scripts/components/eater.lua#L166-L170)、[理智门禁](../../../raw/current/scripts/components/eater.lua#L233-L242) |
| 组件：`hauntable` | `MakeHauntableLaunchAndPerish`；实际只抛飞（腐坏分支被注释） | 主模拟 | [[reference/components/hauntable]]；[注释段](../../../raw/current/scripts/standardcomponents.lua#L1311-L1337) |
| 标签：`catfood` | 浣猫收货判定 | 运行期读取 | [收货](../../../raw/current/scripts/prefabs/catcoon.lua#L273-L279) |
| 标签：`badfood` | 由负理智自动打上；影响喂食门禁 | 组件属性变更时重算 | [判定](../../../raw/current/scripts/components/edible.lua#L1-L7)、[喂食](../../../raw/current/scripts/componentactions.lua#L1164-L1171) |
| 数据：`cooking.lua` | `dairy = 1` | 加载期注册 | [登记](../../../raw/current/scripts/cooking.lua#L148) |
| 关联 Prefab：`eyeofterror` | 唯一掉落来源（本体归后续批次） | 击杀战利品表 | Wiki 待建；[战利品](../../../raw/current/scripts/prefabs/eyeofterror.lua#L461-L474) |
| 关联 Prefab：`eyemaskhat` | 拆解来源 | 拆解法杖；按耐久折算 | [拆解表](../../../raw/current/scripts/recipes.lua#L1678) |
| 关联 Prefab：`spoiled_food` | 腐坏产物 | 主模拟腐坏时生成 | Wiki 待建；[注册](../../../raw/current/scripts/prefabs/spoiledfood.lua#L263) |
| 资源：`anim/milkywhites.zip`、`INV_IMAGE milkywhites` | 物品动画与背包图标 | 只核验声明，未验证资源内容 | [assets](../../../raw/current/scripts/prefabs/milkywhites.lua#L1-L5) |

### 共享规则接入（S1–S7）

- **S1 烤制门禁**：不适用——乳白物没有 `cookable`，物品侧门禁不成立，不能上带 `cooker` 的实体；门禁本身为：目标带 `cooker`、未 `fueldepleted`，`dangerouscooker` 需 `doer:HasTag("expertchef")`，骑乘时目标须归自己所有（[门禁](../../../raw/current/scripts/componentactions.lua#L1096-L1105)）。
- **S2 燃烧掉落**：不适用——本页物品不是任何掉落表的掉落者；它的来源恐怖之眼是生物，被烧后的掉落改写归其本体页。
- **S3 晾晒**：不适用——没有 `dryable` 组件与标签。
- **S4 烹饪登记**：`dairy = 1`，登记未传第三参，不派生 `milkywhites_cooked` 条目。[登记](../../../raw/current/scripts/cooking.lua#L148)
- **S5 堆叠与标签**：显式 `STACK_SIZE_SMALLITEM`＝40；显式标签只有 `catfood`（另有由负理智自动打上的 `badfood`，以及组件自动加的 `edible_GENERIC` 与鲜度三态）。[标签](../../../raw/current/scripts/prefabs/milkywhites.lua#L24)、[堆叠](../../../raw/current/scripts/prefabs/milkywhites.lua#L45-L46)
- **S6 腐坏替换**：替换成 `spoiled_food`。[替换](../../../raw/current/scripts/components/perishable.lua#L277-L319)
- **S7 索引**：索引由主代理统一登记，本页不改 `index.md`。

## 常见陷阱与限制

- **误区：负理智会挡自己吃**。`badfood` 只影响他人喂食等门禁；自己吃照常结算 −20 理智。[判定](../../../raw/current/scripts/components/edible.lua#L1-L7)、[喂食门禁](../../../raw/current/scripts/componentactions.lua#L1164-L1171)
- **误区：一次必掉 5 个**。表里是 3 条 1.00 与 2 条 0.50，单次击杀只保证 3 个。[战利品表](../../../raw/current/scripts/prefabs/eyeofterror.lua#L461-L474)
- **边界：拆解眼罩帽的收益看耐久**。`destroystructure` 用护甲百分比算 `ingredient_percent`，每项取 `max(1, ceil(amount × percent))`，所以半耐久帽只剩约一半。[折算](../../../raw/current/scripts/prefabs/staff_common.lua#L644-L674)
- **边界：它不是"素食"**。`foodtype` 未写，落 `GENERIC`；按 `FOODGROUP` 判定的饮食限制（如 `VEGETARIAN` 含 `GENERIC`）与按 `VEGGIE` 判定的逻辑结果不同，别拿蝶翅的 `VEGGIE` 类推。[默认食性](../../../raw/current/scripts/components/edible.lua#L24)、[食物组](../../../raw/current/scripts/constants.lua#L2098-L2108)
- **未核验**：恐怖之眼本体的掉落触发与战斗流程；`eyemaskhat` 的装备数值与"喂帽子"机制；`critter_eyeofterror` 宠物的行为。

## 验证记录

- 源码审阅：2026-11-09，Steam buildid 26474873。核对 `prefabs/milkywhites.lua` 全文与 `prefablist.lua` 登记；`prefabs/eyeofterror.lua` 的共享战利品表；`prefabs/hats.lua` 的 `eyemask` 构造与注册；`prefabs/staff_common.lua` 的拆解折算；`components/edible.lua`／`components/perishable.lua`／`components/stackable.lua`／`components/lootdropper.lua`／`components/eater.lua` 的相关分支；`cooking.lua` 的食材登记；`preparedfoods.lua` 的冰淇淋；`preparednonfoods.lua` 的蝙蝠鼻帽；`recipes.lua` 的小宠物建造与眼罩帽拆解；`componentactions.lua` 的喂食门禁；`tuning.lua` 的常量；`constants.lua` 的食物组；`chinese_s.po` 与 `scrapbookdata.lua` 的名称与记录。
  未覆盖：恐怖之眼本体、眼罩帽装备机制、宠物行为、`tradable` 的 goldvalue。
- 游戏运行：未运行。三维、天数与掉落期望由 `tuning` 常量与概率表推导。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 通过（见交付回报）。

## 相关链接

- 食材与鲜度：[[reference/components/edible]]、[[reference/components/perishable]]、[[reference/components/eater]]
- 同批页面：[[reference/prefabs/butter|黄油]]、[[reference/prefabs/butterfly-wings|蝴蝶翅膀与月蛾翅膀]]、[[reference/prefabs/honey|蜂蜜与蜂王浆]]

返回 [[reference/prefabs/index|Prefab 索引]]。
