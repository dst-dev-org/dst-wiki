---
title: "蜜脾（honeycomb）"
created: "2026-11-10"
updated: "2026-11-10"
type: prefab
summary: "单注册物品蜜脾：蜂窝与杀人蜂蜂窝固定表各 3 蜜＋1 脾、蜂王共享概率表 1.00＋0.50 两条独立判定、蜂王冠解构按护甲耐久折算，加上蜂箱与蜂蜡配方、honeyed 标签、无 edible 却仍能进炖锅的甜味剂登记，以及默认 20 叠。"
aliases: ["honeycomb", "蜜脾", "蜂巢", "蜂窝产出", "蜂蜡材料", "蜂箱材料"]
tags: [prefab, item]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/honeycomb.lua", "prefabs/beehive.lua", "prefabs/wasphive.lua", "prefabs/beequeen.lua", "prefabs/beequeenhive.lua", "prefabs/hats.lua", "prefabs/staff.lua", "prefabs/staff_common.lua", "prefabs/bearger.lua", "brains/beargerbrain.lua", "components/stackable.lua", "components/lootdropper.lua", "components/tradable.lua", "components/eater.lua", "components/workable.lua", "components/armor.lua", "standardcomponents.lua", "cooking.lua", "containers.lua", "recipes.lua", "tuning.lua", "prefabs/pigking.lua", "languages/chinese_s.po", "screens/redux/scrapbookdata.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: standard
---

# 蜜脾（honeycomb）

## 快速判断

- 职责与适用：蜜脾是**唯一一个注册**的普通物品（简中实显 **蜜脾**），本体的玩法挂载只有浮空、可拾取、可作祟抛飞与默认 20 叠的堆叠（另有一个 `goldvalue`＝0 的 `tradable`）；它**没有** `edible`、`perishable`、`cookable`，不能吃、不会腐坏、不能上火烤。[构造](../../../raw/current/scripts/prefabs/honeycomb.lua#L15-L40)
- 优先使用：查"这块脾从哪来"看[获得途径](#获得途径)，四处产出的接线与概率逐条分开；查"脾能造什么、能不能下锅"看[用途与消耗](#用途与消耗)；查标签、堆叠与显示名看[注册与构造](#注册与构造)。
- 重要副作用：带 `honeyed` 标签，但熊獾的食物筛选还要过 `eater:CanEat`（要求 `edible` 组件），所以**蜜脾不会被熊獾当食物捡走**；反过来它能进炖锅，因为锅的入锅判定是按 `prefab` 名查烹饪食材表，不查 `edible`。[标签](../../../raw/current/scripts/prefabs/honeycomb.lua#L19)、[熊獾筛选](../../../raw/current/scripts/brains/beargerbrain.lua#L51-L60)、[入锅判定](../../../raw/current/scripts/containers.lua#L373-L375)
- 修改边界：蜂窝／杀人蜂蜂窝／蜂王／蜂王巢的本体行为归 [[reference/prefabs/honey]] 与后续批次，蜂王冠归装备页；本页只写蜜脾自身与"谁产出它"的接口（S5）。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `honeycomb` | `fn()`：实体 → Transform／AnimState／Network → `MakeInventoryPhysics` → bank／build `honeycomb` 播 `idle` → 标签 → `MakeInventoryFloatable("med", nil, 0.78)` → `SetPristine()` → 主模拟挂 `tradable`／`inspectable`／`inventoryitem`／`stackable` → `MakeHauntableLaunch` | 唯一注册，没有皮肤子注册、没有变体。`assets` 只有 `anim/honeycomb.zip`。[构造](../../../raw/current/scripts/prefabs/honeycomb.lua#L1-L43)、[注册](../../../raw/current/scripts/prefabs/honeycomb.lua#L45) |

- **pristine 之前的只有标签与浮空**：`honeyed` 在 `SetPristine()` 前加，客户端也读得到；`floater` 与动画同样在公共端。[标签](../../../raw/current/scripts/prefabs/honeycomb.lua#L19-L23)
- **`edible` 是注释掉的，不是挂上又被删**：源码里 1 行 `AddComponent("edible")` 与 2 行数值赋值都以 `--` 开头，成品种没有该组件，也没有 `edible_*` 标签。[注释段](../../../raw/current/scripts/prefabs/honeycomb.lua#L29-L31)
- **堆叠 20 是组件默认值，不是显式写的**：本文件没有 `SetStackSize` 也没有 `maxsize` 赋值，落 `stackable` 构造默认 `TUNING.STACK_SIZE_MEDITEM`＝20；同族蜂蜜与蜂王浆是显式 40，不能互套（S7）。[组件默认](../../../raw/current/scripts/components/stackable.lua#L23-L28)、[常量](../../../raw/current/scripts/tuning.lua#L81-L83)、[图鉴交叉核对](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L711)
- **`tradable` 挂了但值是 0**：`tradable` 构造默认 `goldvalue = 0`，而猪王只接受 `goldvalue > 0`，所以蜜脾不能换金块。[默认值](../../../raw/current/scripts/components/tradable.lua#L1-L7)、[接收判定](../../../raw/current/scripts/prefabs/pigking.lua#L605)
- **作祟只抛飞**：`MakeHauntableLaunch` 的成功分支是 `Launch`＋落地下沉，没有碎裂或转化分支。[helper](../../../raw/current/scripts/standardcomponents.lua#L1053-L1069)

### 名称核验

`NAMES.HONEYCOMB`＝**蜜脾**，是简中实显名（S1）；本 prefab 没有 `SetPrefabName`／`SetPrefabNameOverride`／`nameoverride`，注册名即检索名。[po 名称键](../../../raw/current/scripts/languages/chinese_s.po#L343944-L343947)、[注册名](../../../raw/current/scripts/prefabs/honeycomb.lua#L45)

## 获得途径

四处主线产出的接线方式并不同：两处是**固定 `SetLoot` 列表**，一处是**共享概率表逐条独立判定**，一处是**解构配方按耐久折算**。

| 来源 | 产出 | 接线与条件 | 证据 |
|---|---|---|---|
| 蜂窝 `beehive` | 3 `honey`＋1 `honeycomb` | 死亡回调 `OnKilled` 里 `DropLoot`；掉落表是 `SetLoot({"honey","honey","honey","honeycomb"})`，四项各生成 1 个 | [掉落表](../../../raw/current/scripts/prefabs/beehive.lua#L204-L205)、[死亡](../../../raw/current/scripts/prefabs/beehive.lua#L77-L86) |
| 杀人蜂蜂窝 `wasphive` | 3 `honey`＋1 `honeycomb` | 同一张固定表，死亡时 `DropLoot`；po 显示名是**杀人蜂蜂窝** | [掉落表](../../../raw/current/scripts/prefabs/wasphive.lua#L149-L150)、[死亡](../../../raw/current/scripts/prefabs/wasphive.lua#L22-L31) |
| 蜂王 `beequeen` | 1 条 1.00＋1 条 0.50，期望 **1.5**（推导，无幸运修正的基准） | `SetChanceLootTable('beequeen')` 读共享表；两条 `honeycomb` 分属两个表项，各自独立掷点：1.00 走必中分支，0.50 先过 `GetChance`（击杀者幸运）再与随机值比较 | [战利品表](../../../raw/current/scripts/prefabs/beequeen.lua#L24-L43)、[接线](../../../raw/current/scripts/prefabs/beequeen.lua#L412) |
| 蜂王冠 `hivehat` 解构 | 拆解表原料 4 脾＋3 蜜＋2 蜂王浆＋4 蜜蜂，各材料按护甲耐久比例**独立**折算并向上取整 | `DeconstructRecipe` 写返回表；`destroystructure` 用 `armor:GetPercent() / numtogive` 当比例，每项 `math.max(1, math.ceil(amount × 比例))`——80% 耐久蜜脾仍是 `ceil(3.2)=4`，75% 起降为 3 | [拆解表](../../../raw/current/scripts/recipes.lua#L1673)、[比例](../../../raw/current/scripts/prefabs/staff_common.lua#L644-L649)、[生成](../../../raw/current/scripts/prefabs/staff_common.lua#L663-L674)、[法杖接线](../../../raw/current/scripts/prefabs/staff_common.lua#L770-L776)、[注册](../../../raw/current/scripts/prefabs/staff.lua#L307) |

- **`SetLoot` 会清掉随机段**：赋值即清 `chanceloot`／`randomloot`／`numrandomloot`（不清 `chanceloottable`），所以蜂窝那 4 件是"表里逐项各 1 个"，不是"随机抽 1 个"。[语义](../../../raw/current/scripts/components/lootdropper.lua#L30-L35)、[生成循环](../../../raw/current/scripts/components/lootdropper.lua#L213-L225)
- **概率表逐条独立，小于 1 的条目先过幸运**：1.00 直接入表（必中分支），其余先 `GetChance(chance)` 再与随机值比较——击杀者有幸运时按 `LuckFormulas.LootDropperChance` 改算，所以 50% 是**无幸运修正的基准**；蜂王那两条因此是"必得 1 个，再掷一次 50%"。[掷点](../../../raw/current/scripts/components/lootdropper.lua#L213-L225)、[幸运解析](../../../raw/current/scripts/components/lootdropper.lua#L173-L184)、[共享表读取](../../../raw/current/scripts/components/lootdropper.lua#L21-L28)
- **解构按耐久向上取整，不存在「满耐久才 4 个」**：生成量是 `max(1, ceil(amount × 比例))`，80% 耐久的冠照样给 4 个蜜脾、75% 才降为 3 个；「至少 1 个」只对仍然存在的冠成立——护甲耐久归零会 `Remove()`，0% 冠不是可施法的正常案例。[下限](../../../raw/current/scripts/prefabs/staff_common.lua#L667-L672)、[耗尽移除](../../../raw/current/scripts/components/armor.lua#L101-L112)
- **蜂王冠本体**：`MakeHat("hive")`，`armor:InitCondition(ARMOR_HIVEHAT, ARMOR_HIVEHAT_ABSORPTION)`，解构比例就取这个 `armor` 的百分比。[构造](../../../raw/current/scripts/prefabs/hats.lua#L2070-L2081)、[注册](../../../raw/current/scripts/prefabs/hats.lua#L7887)
- **补充产出口（机制详见 honey 页）**：长成态蜂王巢 `beequeenhivegrown` 被锤击时，`workleft` 取**本击扣减后**的剩余量（记作 workleft_after；首锤 16→15），`lootscale = workleft / BEEQUEEN_SPAWN_MAX_WORK`（16），`rnd = math.random() / lootscale`，`rnd < .01` 才给 1 个 `honeycomb`——通用式为 **概率＝0.01 × workleft_after / 16**（workleft_after 为本击扣减后的剩余量，基准 16）；工作量从 16 起、本击扣 1 时，首锤（16→15）＝**0.9375%**（示例，推导）。workleft_after 越低本击概率越低，归零时 `lootscale=0` 直接取 `rnd=1` 不给。该片段排在"召唤蜂王"判定之后，召唤成功会提前 `return`；执行者须存活、带 `player` 且非 `playerghost`。[执行者门禁](../../../raw/current/scripts/prefabs/beequeenhive.lua#L188-L190)、[扣减顺序](../../../raw/current/scripts/components/workable.lua#L130-L153)、[掉落分支](../../../raw/current/scripts/prefabs/beequeenhive.lua#L192-L213)、[工作设定](../../../raw/current/scripts/prefabs/beequeenhive.lua#L492-L497)、[常量](../../../raw/current/scripts/tuning.lua#L2575-L2576)、[归属](../../../raw/current/scripts/prefabs/beequeenhive.lua#L573-L574)

## 用途与消耗

| 去向 | 投入 | 条件 | 证据 |
|---|---|---|---|
| 蜂箱 `beebox` | 木板 2＋蜜脾 1＋蜜蜂 4 | `TECH.SCIENCE_TWO`，`placer="beebox_placer"`、`min_spacing=2.5` | [配方](../../../raw/current/scripts/recipes.lua#L450) |
| 蜂蜡 `beeswax` | 蜜脾 1 | `TECH.SCIENCE_TWO`，一次 1 块蜂蜡 | [配方](../../../raw/current/scripts/recipes.lua#L464) |
| 炖锅食材 | 1 份＝`sweetener = 1` | 加载期登记；锅按名字收料，**不需要 `edible`** | [登记](../../../raw/current/scripts/cooking.lua#L97)、[入锅判定](../../../raw/current/scripts/containers.lua#L373-L375)、[判定实现](../../../raw/current/scripts/cooking.lua#L185-L187) |

- **能下锅但没有实体可烤**：`AddIngredientValues({"honey","honeycomb"}, {sweetener=1}, true)` 的第三参 `true` 会派生一个 `honeycomb_cooked` 食材条目（带 `precook`），但游戏里没有这个 prefab，物品也没有 `cookable`，所以"烤蜜脾"这条路不存在。[派生](../../../raw/current/scripts/cooking.lua#L46-L72)
- **甜度按份累加**：1 份蜜脾只给 1 点甜度，够不到太妃糖的 ≥3；具体菜谱判定归 [[reference/prefabs/honey#用途与消耗]]（S2）。
- **没有其它加工路径**：无 `cookable`、无 `dryable`、无 `fuel`，所以除了上述两条配方与炖锅填料，剩下只有堆叠、搬运与被作祟抛飞。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`stackable` | 默认 `STACK_SIZE_MEDITEM`＝20，未显式赋值 | 主模拟 | [[reference/components/stackable]]；[挂载](../../../raw/current/scripts/prefabs/honeycomb.lua#L38)、[默认值](../../../raw/current/scripts/components/stackable.lua#L23-L28) |
| 组件：`inventoryitem` | 可拾取、可落地 | 主模拟 | [[reference/components/inventoryitem]]；[挂载](../../../raw/current/scripts/prefabs/honeycomb.lua#L37) |
| 组件：`tradable` | 挂载但不赋值 → `goldvalue = 0`，猪王不收 | 主模拟 | [[reference/components/tradable]]；[挂载](../../../raw/current/scripts/prefabs/honeycomb.lua#L33)、[默认值](../../../raw/current/scripts/components/tradable.lua#L1-L7)、[接收判定](../../../raw/current/scripts/prefabs/pigking.lua#L605) |
| 组件：`hauntable` | `MakeHauntableLaunch`：作祟成功只抛飞 | 主模拟 | [[reference/components/hauntable]]；[挂载](../../../raw/current/scripts/prefabs/honeycomb.lua#L40) |
| 标签：`honeyed` | 消费者只有熊獾（`HoneyedItem`、取食优先、容器目标与 SG 判定）；且取食还要过 `CanEat`（需 `edible`），蜜脾不通过 | 运行期读取 | [标签](../../../raw/current/scripts/prefabs/honeycomb.lua#L19)、[判定](../../../raw/current/scripts/prefabs/bearger.lua#L107-L109)、[取食链](../../../raw/current/scripts/brains/beargerbrain.lua#L51-L60)、[`TestFood` 门禁](../../../raw/current/scripts/components/eater.lua#L288-L302) |
| 数据：`cooking.lua` 食材表 | 蜜脾登记 `sweetener = 1`，并派生不存在的 `honeycomb_cooked` 条目 | 加载期；锅与便携锅共用 | [登记](../../../raw/current/scripts/cooking.lua#L97)、[判定](../../../raw/current/scripts/cooking.lua#L185-L187) |
| 关联 Prefab：`honey`／`beeswax`／`beebox` | 同批产出物与两条配方产物 | 由 `lootdropper`／配方生成 | [[reference/prefabs/honey]]；[蜂蜡配方](../../../raw/current/scripts/recipes.lua#L464) |
| 关联 Prefab：`beehive`／`wasphive`／`beequeen`／`beequeenhivegrown` | 四个宿主侧产出口 | 死亡或锤击触发 | [[reference/prefabs/honey]]（宿主本体行为）；[蜂窝](../../../raw/current/scripts/prefabs/beehive.lua#L204-L205) |
| 关联 Prefab：`hivehat` | 蜂王冠解构回流 | 解构法杖对携带配方实体施法 | [拆解表](../../../raw/current/scripts/recipes.lua#L1673)、[绿杖](../../../raw/current/scripts/prefabs/staff_common.lua#L770-L776) |
| 资源：`anim/honeycomb.zip` | bank／build `honeycomb`、动画 `idle` | 原版资源，不随页分发 | [资产](../../../raw/current/scripts/prefabs/honeycomb.lua#L1-L4) |

## 常见陷阱与限制

- **误区：蜜脾是"蜂蜜类食物"，能直接吃或喂熊。** 本文件把 `edible` 整段注释掉，成品种没有该组件与 `edible_*` 标签；熊獾的 `honeyed` 优先分支前面还有 `eater:CanEat`，后者要求 `food.components.edible ~= nil`，所以蜜脾不会被当作食物目标。[注释段](../../../raw/current/scripts/prefabs/honeycomb.lua#L29-L31)、[CanEat](../../../raw/current/scripts/components/eater.lua#L288-L302)、[取食链](../../../raw/current/scripts/brains/beargerbrain.lua#L51-L60)
- **边界：默认 20 叠，不是 40。** 没有 `SetStackSize` 就落组件默认值；同族的蜂蜜／蜂王浆是显式 40，图鉴也把蜜脾记成 20 叠，可作交叉核对。[默认值](../../../raw/current/scripts/components/stackable.lua#L23-L28)、[图鉴](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L711)
- **已核对：honey 页已同步（本轮勘误已合入）。** 该页现为蜂窝 3 蜜＋1 脾、蜂王冠拆解，不再需要按本页更正；两份记录一致。[掉落表](../../../raw/current/scripts/prefabs/beehive.lua#L205)、[HIVEHAT](../../../raw/current/scripts/languages/chinese_s.po#L343930-L343932)
- **边界：烧毁的蜂窝掉灰烬，不掉蜜脾。** 蜂窝与杀人蜂蜂窝都带 `hive` 标签，`DropLoot` 在掉落者已烧毁／正在燃烧时走燃烧改写链，换成 `ash`；完整改写顺序见 [[reference/prefabs/honey#常见陷阱与限制]]，本页不重复展开。[hive 标签](../../../raw/current/scripts/prefabs/beehive.lua#L168-L171)、[杀人蜂蜂窝标签](../../../raw/current/scripts/prefabs/wasphive.lua#L114-L115)
- **边界：蜂王巢的蜜脾概率随剩余工作量下降，首锤不到 1%。** 通用式为 **概率＝0.01 × workleft_after / 16**（workleft_after 取本击扣减后的剩余量）；工作量从 16 起、本击扣 1 时，首锤（16→15）＝**0.9375%**（示例，推导）。workleft_after 越低本击概率越低，归零时不给；召唤成功提前返回，执行者须是存活的非幽灵玩家。[门禁](../../../raw/current/scripts/prefabs/beequeenhive.lua#L188-L190)、[分支](../../../raw/current/scripts/prefabs/beequeenhive.lua#L192-L213)
- **未核验**：未运行游戏。蜂王 1.5 个的期望（无幸运基准）、蜂王巢 0.01×workleft_after/16（工作量从 16 起、本击扣 1 时首锤 0.9375%）、解构按耐久折算后的实际个数都是源码推导；缺游戏内掉落采样这一步。

## 验证记录

- 源码审阅：2026-11-10，Steam buildid 26474873。逐行读 `prefabs/honeycomb.lua`（全 45 行）；`prefabs/beehive.lua` 的掉落、死亡与标签段，`prefabs/wasphive.lua` 的掉落与死亡段，`prefabs/beequeen.lua` 的共享表与接线，`prefabs/beequeenhive.lua` 的 `OnWorked`／`workable`／两个注册；`prefabs/hats.lua` 的 `hive` 帽构造与注册；`prefabs/staff.lua`／`staff_common.lua` 的解构比例与生成；`components/stackable.lua`、`lootdropper.lua`、`tradable.lua`、`eater.lua`、`standardcomponents.lua` 的相关函数；`cooking.lua` 的食材登记与 `IsCookingIngredient`；`containers.lua` 的 `cookpot.itemtestfn`；`recipes.lua` 的蜂箱／蜂蜡／蜂王冠拆解三行；`tuning.lua` 的堆叠与蜂王巢常量；`chinese_s.po` 的名称键；`scrapbookdata.lua` 的蜜脾记录。
  未覆盖：蜂王与蜂巢的完整战斗／生成逻辑、其它事件场景（如场景宝箱）里的蜜脾摆放、模组对 `honeyed` 的复用。
- 游戏运行：未运行；上文的"期望 1.5 个""1%×lootscale""每项至少 1 个"均为源码推导。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 结果见交付回报。

## 相关链接

- 蜂蜜与蜂王浆（同批产出、宿主行为与燃烧改写）：[[reference/prefabs/honey]]
- 物品组件：[[reference/components/stackable]]、[[reference/components/lootdropper]]、[[reference/components/tradable]]、[[reference/components/hauntable]]
- 同批页面：[[reference/prefabs/houndstooth|犬牙]]、[[reference/prefabs/stinger|针刺]]、[[reference/prefabs/tentaclespots|触手皮]]、[[reference/prefabs/walrus-tusk|海象牙]]

返回 [[reference/prefabs/index|Prefab 索引]]。
