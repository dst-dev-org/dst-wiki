---
title: "暗影碎布（voidcloth）"
created: "2026-11-10"
updated: "2026-11-10"
type: prefab
summary: "暗影碎布的单一定义：从裂影仆从与暗影憎恶掉落；作为纯合成材料进入暗影术基座的 10 条消耗侧配方；说明它没有燃料、交易与修理件，也没有自定义标签，堆叠 20，以及漂浮参数与装备侧的虚空修补套件关系。"
aliases: ["voidcloth", "暗影碎布", "虚空布", "虚布", "Dark Tatters", "暗影布"]
tags: [prefab, item]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/voidcloth.lua", "prefabs/shadowthrall_hands.lua", "prefabs/shadowthrall_horns.lua", "prefabs/shadowthrall_mouth.lua", "prefabs/shadowthrall_wings.lua", "prefabs/shrouden.lua", "stategraphs/SGshrouden.lua", "prefabs/shadow_forge.lua", "prefabs/forge_repair_kits.lua", "prefabs/hats.lua", "components/lootdropper.lua", "components/builder.lua", "componentutil.lua", "standardcomponents.lua", "recipes.lua", "recipe.lua", "tuning.lua", "constants.lua", "languages/chinese_s.po", "screens/redux/scrapbookdata.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# 暗影碎布（voidcloth）

## 快速判断

- 职责与适用：`prefabs/voidcloth.lua` 只注册一个 `voidcloth`，是纯材料页——`idle` 单帧图形＋`inventoryitem`＋`stackable`＋`inspectable`，再加一个“作祟只抛飞”的 `hauntable`；没有 `fuel`、`tradable`、`repairer`，也没有任何 `AddTag`。[组件段](../../../raw/current/scripts/prefabs/voidcloth.lua#L27-L33)
- 优先使用：查“打谁掉”看[获取](#获取)；查“配方各要几个布”看[配方消费者](#配方消费者)；查为什么配方灰着看[暗影术基座](#配方消费者)。
- 重要副作用：它只被当**材料消耗**，不进任何燃料槽、也不当修理件；装备侧的“虚空修补套件”修的是成品装备，不消耗本物品以外的通道。[无燃料与修理件](../../../raw/current/scripts/prefabs/voidcloth.lua#L27-L33)、[套件](../../../raw/current/scripts/prefabs/forge_repair_kits.lua#L28-L57)
- 修改边界：护甲／帽子／镰刀／回旋镖各自的行为与耐久归对应页面；本页只写这一个注册的构造、掉落与配方数量。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `voidcloth`（暗影碎布） | 独立 `fn()`：实体→Transform/AnimState/Network→`MakeInventoryPhysics`→bank/build `voidcloth`＋`idle`（单帧）→漂浮→`SetPristine`→主模拟挂组件 | 资源只有 `anim/voidcloth.zip`；无 `prefabs` 依赖表；无 `SetMultColour`（不透明，与两件暗影燃料相反）。[工厂](../../../raw/current/scripts/prefabs/voidcloth.lua#L6-L36)、[注册](../../../raw/current/scripts/prefabs/voidcloth.lua#L38) |

构造细节（按执行顺序）：

- **图形**：`SetBank`/`SetBuild` 都是 `voidcloth`，`PlayAnimation("idle")`，没开循环。[表现](../../../raw/current/scripts/prefabs/voidcloth.lua#L15-L17)
- **漂浮**：`MakeInventoryFloatable(inst, "large", nil, 0.6)`——大号漂浮体、偏移取 helper 默认（`nil` 时不设）、缩放 0.6；helper 只设 `floater` 字段，不加标签。[漂浮](../../../raw/current/scripts/prefabs/voidcloth.lua#L19)、[helper](../../../raw/current/scripts/standardcomponents.lua#L1772-L1791)
- **堆叠**：`TUNING.STACK_SIZE_MEDITEM`＝20，比噩梦燃料／绝望石的 40 少一半；采集册同为 20。[赋值](../../../raw/current/scripts/prefabs/voidcloth.lua#L30-L31)、[常量](../../../raw/current/scripts/tuning.lua#L82)、[采集册](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L1669)
- **组件顺序**：`inspectable` → `inventoryitem` → `stackable` → `MakeHauntableLaunch`；`inventoryitem` 排在 `stackable` 之后，与噩梦燃料的顺序相反，但两者都不依赖对方先存在。[顺序](../../../raw/current/scripts/prefabs/voidcloth.lua#L27-L33)
- **没有的东西**：无 `tradable`（不能卖）、无 `fuel`（不是燃料）、无 `repairer`（`MATERIALS` 表里也没有 `VOIDCLOTH`，它只出现在 `FORGEMATERIALS`）。[constants](../../../raw/current/scripts/constants.lua#L1940-L1967)

### 名称

显示名取自己的名字键，无 `nameoverride`：`NAMES.VOIDCLOTH` 英文是 `Dark Tatters`、简中＝**暗影碎布**；带“虚空”字样的中文名属于成品装备（虚空长袍／虚空风帽），既有页里常见的“虚布”是简写而非 `NAMES` 里的名字。[名称](../../../raw/current/scripts/languages/chinese_s.po#L351340-L351342)、[装备名](../../../raw/current/scripts/languages/chinese_s.po#L338860-L338862)

## 获取

掉落表里的数字是**基础概率**：`chanceloot` 逐条 roll，`chance >= 1.0` 直接入列，小于 1 的先过 `GetChance`——按死亡归因实体或最后工作者的**正向**幸运抬高，无幸运／负幸运回落基础值。所以“1.00×3＋0.33”在无幸运修正是基础期望 3.33 块；那条 0.33 在有幸运时**高于** 33%（R11）。[生成](../../../raw/current/scripts/components/lootdropper.lua#L163-L227)、[幸运公式](../../../raw/current/scripts/componentutil.lua#L2092-L2097)

| 来源 | 掉暗影碎布 | 同表其它产出 | 证据 |
|---|---|---|---|
| 裂影仆从 `shadowthrall_hands` | 1.00＋1.00＋1.00＋0.33 | 纯粹恐惧 1.00＋0.50、噩梦燃料 1.00＋1.00＋0.67 | [表](../../../raw/current/scripts/prefabs/shadowthrall_hands.lua#L17-L28) |
| 裂影仆从 `shadowthrall_horns` | 1.00＋1.00＋1.00＋0.33 | 纯粹恐惧 1.00＋0.50、噩梦燃料 1.00＋1.00＋0.67 | [表](../../../raw/current/scripts/prefabs/shadowthrall_horns.lua#L17-L27) |
| 裂影仆从 `shadowthrall_mouth` | 1.00＋1.00＋1.00＋0.33 | 同 horns | [表](../../../raw/current/scripts/prefabs/shadowthrall_mouth.lua#L16-L26) |
| 裂影仆从 `shadowthrall_wings` | 1.00＋1.00＋0.67 | 纯粹恐惧 1.00＋1.00＋0.33、噩梦燃料 1.00＋0.67 | [表](../../../raw/current/scripts/prefabs/shadowthrall_wings.lua#L18-L27) |
| 暗影憎恶 `shrouden` 的 `shrouden2` 表 | 1.00×4＋0.75＋0.50 | 绝望石 1.00×4＋0.75＋0.50、素描 1.00 | [表](../../../raw/current/scripts/prefabs/shrouden.lua#L46-L63)、[接线](../../../raw/current/scripts/stategraphs/SGshrouden.lua#L982-L986) |

四只裂影仆从都是受控裂隙由 `shadowthrallmanager` 刷出的守卫；无其它生物表掉这块布，也没有合成或嬗变入口（`recipes.lua` 里 `voidcloth` 只出现在产物配方与 `Ingredients` 消耗侧）。

## 配方消费者

10 条消耗侧配方都要 `TECH.SHADOWFORGING_TWO`＋`station_tag = "shadow_forge"`，科技来自**暗影术基座**（`prototyper.trees = TUNING.PROTOTYPER_TREES.SHADOW_FORGE`，该树给 `SHADOWFORGING = 2`）。但**能否离站制作取决于 `nounlock`**（R12）：其中 9 条标了 `nounlock = true`，`Builder:UnlockRecipe` 对它们直接跳过，不会写进已学列表，只能在基座旁做；唯一例外是 `slingshot_handle_voidcloth`，它只有 `force_hint`＋`station_tag`＋沃尔特技能，可在站旁首次解锁后离站制作。[基座](../../../raw/current/scripts/prefabs/shadow_forge.lua#L124-L152)、[科技树](../../../raw/current/scripts/tuning.lua#L1452-L1453)、[九条 nounlock](../../../raw/current/scripts/recipes.lua#L866-L875)、[握把](../../../raw/current/scripts/recipes.lua#L245)、[unlock 跳过](../../../raw/current/scripts/components/builder.lua#L463-L473)

| 产物 | 消耗暗影碎布 | 其它材料 | 证据 |
|---|---|---|---|
| `armor_voidcloth`（虚空长袍） | 4 | ＋纯粹恐惧 4 | [配方](../../../raw/current/scripts/recipes.lua#L866) |
| `voidclothhat`（虚空风帽） | 2 | ＋纯粹恐惧 4 | [配方](../../../raw/current/scripts/recipes.lua#L867) |
| `voidcloth_boomerang` | 2 | ＋纯粹恐惧 4 | [配方](../../../raw/current/scripts/recipes.lua#L870) |
| `voidcloth_umbrella` | 1 | ＋纯粹恐惧 5 | [配方](../../../raw/current/scripts/recipes.lua#L868) |
| `voidcloth_scythe` | 1 | ＋纯粹恐惧 3 | [配方](../../../raw/current/scripts/recipes.lua#L869) |
| `voidcloth_kit`（虚空修补套件） | 1 | ＋纯粹恐惧 1 | [配方](../../../raw/current/scripts/recipes.lua#L871) |
| `slingshot_handle_voidcloth` | 1 | 无其它材料，Walter 技能 | [配方](../../../raw/current/scripts/recipes.lua#L245) |
| `shadow_beef_bell`／`shadow_battleaxe`／`saddle_shadow` | 2／2／5 | 三者都要注入暗影之心＋纯粹恐惧＋绝望石 | [配方](../../../raw/current/scripts/recipes.lua#L873-L875) |

成品侧的机制不在此展开：护甲归 [[reference/prefabs/armor-rift|裂隙三护甲族]]、风帽归 [[reference/prefabs/hats-armor|防护与战斗帽族]]、回旋镖归 [[reference/prefabs/voidcloth_boomerang]]、暗影槌归 [[reference/prefabs/shadow-battleaxe]]、镰刀与伞归各自页面。

**套件不是本物品的修理件**：虚空修补套件由 `MakeKit("voidcloth_kit", FORGEMATERIALS.VOIDCLOTH)` 生成，走 `forgerepairable`＋`forgerepair` 这套独立材质，装备侧用 `MakeForgeRepairable(inst, FORGEMATERIALS.VOIDCLOTH, ...)` 接上；与本页的 `repairer` 无关。[套件](../../../raw/current/scripts/prefabs/forge_repair_kits.lua#L28-L57)、[装备侧](../../../raw/current/scripts/prefabs/hats.lua#L4144)、[材质](../../../raw/current/scripts/constants.lua#L1962-L1967)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`stackable` | `maxsize = STACK_SIZE_MEDITEM`（20） | 主模拟 | [[reference/components/stackable]]；[赋值](../../../raw/current/scripts/prefabs/voidcloth.lua#L30-L31) |
| 组件：`inventoryitem`／`floater` | 可拾取；`large` 漂浮体、缩放 0.6 | 两端；漂浮在 pristine 前构造 | [[reference/components/inventoryitem]]；[漂浮](../../../raw/current/scripts/prefabs/voidcloth.lua#L19) |
| 组件：`hauntable` | `MakeHauntableLaunch`：作祟只抛飞，不改状态 | 主模拟；落地清 `is_landed` | [接线](../../../raw/current/scripts/prefabs/voidcloth.lua#L33)、[实现](../../../raw/current/scripts/standardcomponents.lua#L1053-L1069) |
| 关联 Prefab：`shadow_forge` | 全部配方的制作台：带 `shadow_forge` 标签的 `prototyper` | 建造后常驻；同一标签也用于配方 UI 过滤 | [基座](../../../raw/current/scripts/prefabs/shadow_forge.lua#L124-L152)、[station_tag 语义](../../../raw/current/scripts/recipe.lua#L159) |
| 关联 Prefab：`voidcloth_kit` | 成品装备的修理通道（`forgerepairable`），不是本物品的修理件 | 主模拟；每件装备按套件材质判定 | [套件](../../../raw/current/scripts/prefabs/forge_repair_kits.lua#L28-L57) |
| 关联 Prefab：`shadowthrall_hands`／`_horns`／`_mouth`／`_wings` | 主要掉落源，四张表都含本物品 | 世界内生物；由 `shadowthrallmanager` 在受控裂隙刷出 | [表](../../../raw/current/scripts/prefabs/shadowthrall_hands.lua#L17-L24) |
| 资源：`anim/voidcloth.zip` | 本物品自己的 bank／build | 原版资源，不随页分发 | [asset](../../../raw/current/scripts/prefabs/voidcloth.lua#L1-L4) |

## 常见陷阱与限制

- **误区：暗影碎布可以当燃料或当修理件。** 它只有 `inspectable`／`inventoryitem`／`stackable`／`hauntable`，没有 `fuel` 也没有 `repairer`；`MATERIALS` 表里同样没有 `VOIDCLOTH`。[组件段](../../../raw/current/scripts/prefabs/voidcloth.lua#L27-L33)、[材质表](../../../raw/current/scripts/constants.lua#L1940-L1967)
- **边界：配方“看得见”与“能做”不是同一件事。** `TECH.SHADOWFORGING_TWO` 要暗影术基座的科技等级 2；`station_tag` 只决定该配方在制作站筛选里的归类，不代替科技门槛；九条 `nounlock` 配方还额外受“不能离站”限制。[技术等级](../../../raw/current/scripts/constants.lua#L1334-L1335)、[基座科技](../../../raw/current/scripts/prefabs/shadow_forge.lua#L148-L152)
- **边界：堆叠只有 20。** 同为暗影系材料，噩梦燃料与绝望石是 `STACK_SIZE_SMALLITEM`＝40，本物品是 `STACK_SIZE_MEDITEM`＝20，搬运时占用格数不同。[赋值](../../../raw/current/scripts/prefabs/voidcloth.lua#L30-L31)、[常量](../../../raw/current/scripts/tuning.lua#L82-L83)
- **边界：名称不要按英文直译。** `Dark Tatters` 的官方简中是“暗影碎布”；“虚空”是 `armor_voidcloth`／`voidclothhat` 等成品的中文名用词，混用会搜不到本物品。[名称](../../../raw/current/scripts/languages/chinese_s.po#L351340-L351342)
- **未核验**：未运行游戏。掉落概率与配方数量来自源码；裂影仆从的实际刷新节奏（`shadowthrallmanager` 的找裂隙与事件条件）未实测。

## 验证记录

- 源码审阅：2026-11-10，Steam buildid 26474873。通读 `prefabs/voidcloth.lua` 全文；四个 `shadowthrall_*` 的掉落表与 `SetChanceLootTable` 接线；`prefabs/shrouden.lua` 的 `shrouden2` 表与 `stategraphs/SGshrouden.lua` 的掉落调用；`prefabs/shadow_forge.lua` 的标签与 `prototyper`；`prefabs/forge_repair_kits.lua` 的 `MakeKit("voidcloth_kit", FORGEMATERIALS.VOIDCLOTH)`；`prefabs/hats.lua` 的 `MakeForgeRepairable` 调用；`recipes.lua` 全部含 `voidcloth` 的配方；`recipe.lua` 的 `station_tag` 注释；`tuning.lua` 的 `STACK_SIZE_MEDITEM`、`PROTOTYPER_TREES.SHADOW_FORGE`；`constants.lua` 的 `MATERIALS`／`FORGEMATERIALS`／`TECH.SHADOWFORGING_TWO`；名称取 `chinese_s.po`，堆叠用 `scrapbookdata.lua` 交叉核对（`stacksize=20`）。
  未覆盖：`forgerepair` 的完整修理流程（归装备页与 [[reference/components/forgerepairable]]）、暗影术基座的建造与拆除流程、模组注入。
- 游戏运行：未运行。概率、堆叠与配方数量均为源码读取值，未实测。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 见交付回报。

## 相关链接

- 装备与配方产物：[[reference/prefabs/armor-rift]]（虚空长袍）、[[reference/prefabs/hats-armor]]（虚空风帽）、[[reference/prefabs/voidcloth_boomerang]]、[[reference/prefabs/shadow-battleaxe]]
- 原料页：[[reference/prefabs/nightmare-fuel]]（纯粹恐惧）、[[reference/prefabs/dreadstone]]
- 组件：[[reference/components/stackable]]、[[reference/components/inventoryitem]]

返回 [[reference/prefabs/index|Prefab 索引]]。
