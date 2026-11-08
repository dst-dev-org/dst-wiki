---
title: "亮茄外壳（lunarplant_husk）"
created: "2026-11-10"
updated: "2026-11-10"
type: prefab
summary: "致命亮茄死亡掉落的月系锻造材料：单一纯物品注册，堆叠 20、没有任何游戏标签、不可食用，全部消费来自月之锻造台配方与沃尔特弹弓弹药。"
aliases: ["lunarplant_husk", "亮茄外壳", "亮茄壳", "月植外壳", "Brightshade Husk", "亮茄材料", "亮茄弹药材料"]
tags: [prefab, item]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/lunarplant_husk.lua", "prefabs/lunarthrall_plant.lua", "recipes.lua", "tuning.lua", "components/stackable.lua", "standardcomponents.lua", "screens/redux/scrapbookdata.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# 亮茄外壳（lunarplant_husk）

## 快速判断

- 职责与适用：`prefabs/lunarplant_husk.lua` 只有一个注册 `lunarplant_husk`，官方简中"亮茄外壳"；对象类别是纯 item——能捡、能叠、能漂在水面，除 `inspectable`／`inventoryitem`／`stackable` 外没有别的玩法组件。[注册](../../../raw/current/scripts/prefabs/lunarplant_husk.lua#L38)、[构造](../../../raw/current/scripts/prefabs/lunarplant_husk.lua#L27-L33)
- 优先使用：查某条月之锻造配方要几个外壳看[月之锻造配方消费者](#月之锻造配方消费者)；查"打什么掉"看[获取：只有致命亮茄](#获取只有致命亮茄)；查能不能吃、能不能当诱饵、能不能烧看[堆叠与标签边界](#堆叠与标签边界)。
- 重要副作用：全文件只调用一次标签类 helper——`MakeHauntableLaunch`（挂 `hauntable`，作祟时被抛飞），**没有任何 `AddTag`**；也没有 `edible`／`fuel`／`tradable`／`repairer`。[全文件](../../../raw/current/scripts/prefabs/lunarplant_husk.lua#L6-L35)、[hauntable 挂载](../../../raw/current/scripts/standardcomponents.lua#L1053-L1055)
- 修改边界：掉落归致命亮茄 `lunarthrall_plant` 本体，配方成本归 `recipes.lua`，弹药数值归 [[reference/prefabs/ranged-weapons]]；致命亮茄的战斗、裂隙刷新与月之锻造台本体不在本页。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `lunarplant_husk` | `fn()`：实体 → Transform／AnimState／Network → `MakeInventoryPhysics` → bank／build `lunarplant_husk` 播 `idle` → `MakeInventoryFloatable(inst, "large", nil, 0.6)` → `SetPristine` → 客户端返回 → 主模拟挂 `inspectable`／`inventoryitem`／`stackable` → `MakeHauntableLaunch` | 唯一形态，没有世界态、投射态或变体；`stackable.maxsize = TUNING.STACK_SIZE_MEDITEM`。资源只有 `anim/lunarplant_husk.zip`。[工厂](../../../raw/current/scripts/prefabs/lunarplant_husk.lua#L6-L35)、[资源](../../../raw/current/scripts/prefabs/lunarplant_husk.lua#L1-L4) |

- **端分界**：`SetPristine` 之前只有物理、动画、浮水与网络；三个组件和 `hauntable` 都在 `if not TheWorld.ismastersim then return inst end` 之后，客户端实体没有这些组件。[主模拟分界](../../../raw/current/scripts/prefabs/lunarplant_husk.lua#L21-L33)
- **堆叠是显式赋值**：`inst.components.stackable.maxsize = TUNING.STACK_SIZE_MEDITEM`，常量 20；虽然 `stackable` 的构造默认值也是 20，但改默认值不会改本物品，只有改写法或改常量才会。[赋值](../../../raw/current/scripts/prefabs/lunarplant_husk.lua#L30-L31)、[常量](../../../raw/current/scripts/tuning.lua#L82)、[组件默认](../../../raw/current/scripts/components/stackable.lua#L23-L28)
- **采集册与源码一致**：`scrapbookdata` 记 `lunarplant_husk` 为 `type="item"`、`stacksize=20`、`build／bank="lunarplant_husk"`、`anim="idle"`。[scrapbookdata](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L807)

### 名称核验

`STRINGS.NAMES.LUNARPLANT_HUSK` 的 `msgid` 是 `Brightshade Husk`、`msgstr` 是"亮茄外壳"；本 prefab 没有 `SetPrefabName`、`SetPrefabNameOverride` 或 `nameoverride`，注册名与显示名一一对应。[名称键](../../../raw/current/scripts/languages/chinese_s.po#L344869-L344872)

## 获取：只有致命亮茄

致命亮茄 `lunarthrall_plant`（官方简中"致命亮茄"）的死亡掉落表是**基础名单**：`local loot = { "lunarplant_husk", "lunarplant_husk", "plantmeat", "plantmeat" }`，四条概率都是 100%；未被烧死时稳定落地 2 外壳＋2 植物肉，火烧状态见下条。[掉落表](../../../raw/current/scripts/prefabs/lunarthrall_plant.lua#L25-L30)、[挂载](../../../raw/current/scripts/prefabs/lunarthrall_plant.lua#L487-L488)

- **结算点**：`OnDeath` 先 `killvines()`，再 `inst.components.lootdropper:DropLoot()`，然后播 `death_<size>` 并在 `animover` 时自删。掉落发生在死亡瞬间，与敲碎、收割或采粉无关。[OnDeath](../../../raw/current/scripts/prefabs/lunarthrall_plant.lua#L196-L217)
- **火烧状态会换名**：宿主带 `burnable`（`MakeLargeBurnableCharacter`）与 `plant` 标签；`DropLoot` 时若带 `burnt`，或正在燃烧且没有 `fueled`／`burnable.ignorefuel`，就逐条换名——两项都不在 `TUNING.BURNED_LOOT_OVERRIDES` 里，`plantmeat` 命中 `plantmeat_cooked`（有熟态），`lunarplant_husk` 没有 `_cooked` 变体，受控火（`GetControlledBurn`）下保留原名（丢出时的 `StartWildfire` 只对有 `burnable` 的战利品生效，外壳没有，故不会闷烧）；非受控火则变 `ash`。`plant` 不在灰烬豁免的 `tree`／`boulder` 集合内，外壳同样转灰。[宿主可燃](../../../raw/current/scripts/prefabs/lunarthrall_plant.lua#L514-L516)、[plant 标签](../../../raw/current/scripts/prefabs/lunarthrall_plant.lua#L444)、[换名与门禁](../../../raw/current/scripts/components/lootdropper.lua#L414-L442)、[受控火](../../../raw/current/scripts/components/lootdropper.lua#L404-L407)、[熟肉注册](../../../raw/current/scripts/prefabs/meats.lua#L913)
- **本版检索范围**：按完整注册名 `"lunarplant_husk"` 检索（排除 `armor_lunarplant_husk` 皮肤族与本文件自身）只命中 `lunarthrall_plant.lua`、`recipes.lua`（10 条配方）、`scrapbookdata.lua`、`scrapbook_prefabs.lua`、`prefablist.lua` 与 `languages/strings.pot`（译名键），另 `skins_defs_data.lua#L6117` 一处注释；`item_blacklist.lua#L28-L29`、`prefabskin.lua#L1698-L1699`、`prefabskins.lua#L79-L82`、`recipes_filter.lua#L308`、`skin_assets.lua`、`skin_strings.lua`、`skinprefabs.lua` 命中的都是护甲皮肤族。没有第二张掉落表，也没有礼物、钓鱼或交易产出路径。

## 月之锻造配方消费者

`recipes.lua` 共 10 条配方吃外壳。前 9 条是月之锻造台的**站内配方**：`TECH.LUNARFORGING_TWO`＋`station_tag="lunar_forge"`＋`nounlock=true`，每次都要在月之锻造台旁制作；第 10 条弹药没有 `nounlock`，见下。

| 产物 | 外壳 | 同配方其它材料 | 证据 |
|---|---:|---|---|
| `armor_lunarplant`（亮茄盔甲） | 4 | `purebrilliance`×4 | [L848](../../../raw/current/scripts/recipes.lua#L848) |
| `lunarplanthat`（亮茄头盔） | 2 | `purebrilliance`×4 | [L849](../../../raw/current/scripts/recipes.lua#L849) |
| `bomb_lunarplant`（亮茄炸弹，一次 6 个） | 4 | `purebrilliance`×4、`moonglass_charged`×1 | [L850](../../../raw/current/scripts/recipes.lua#L850) |
| `staff_lunarplant`（亮茄魔杖） | 6 | `purebrilliance`×3 | [L851](../../../raw/current/scripts/recipes.lua#L851) |
| `sword_lunarplant`（亮茄剑） | 3 | `purebrilliance`×4 | [L852](../../../raw/current/scripts/recipes.lua#L852) |
| `pickaxe_lunarplant`（亮茄粉碎者） | 2 | `purebrilliance`×1 | [L853](../../../raw/current/scripts/recipes.lua#L853) |
| `shovel_lunarplant`（亮茄锄铲） | 2 | `purebrilliance`×1 | [L854](../../../raw/current/scripts/recipes.lua#L854) |
| `lunarplant_kit`（亮茄修补套件） | 1 | `purebrilliance`×1 | [L855](../../../raw/current/scripts/recipes.lua#L855) |
| `houndstooth_blowpipe`（嚎弹炮） | 3 | `security_pulse_cage_full`×1、`purebrilliance`×3、`moonrocknugget`×3 | [L859](../../../raw/current/scripts/recipes.lua#L859) |
| `slingshotammo_lunarplanthusk`（亮茄外壳弹药，一次 20 发） | 1 | 无 | [L228](../../../raw/current/scripts/recipes.lua#L228) |

- **弹药是唯一一条"1 换 20"，但它不是站内配方**：`numtogive=20`，再经 `override_numtogive_fn=calc_slingshotammo_numtogive`；沃尔特技能 `walter_ammo_efficiency` 生效时返回 `numtogive * 1.5`＝30。[配方](../../../raw/current/scripts/recipes.lua#L228)、[加成](../../../raw/current/scripts/recipes.lua#L199-L204)
- **弹药与锻造配方的门禁不同**：弹药只有 `station_tag="lunar_forge"`、没有 `nounlock`——第一次在月之锻造台旁、满足 `builder_skill="walter_ammo_lunar"` 才能做，学会后离站仍可制作，技能门禁继续生效。上面 9 条锻造配方都带 `nounlock=true`，`Builder:UnlockRecipe` 会跳过它们，不会被一次普通解锁记住，只能靠当次站点科技制作。[解锁跳过](../../../raw/current/scripts/components/builder.lua#L463-L473)、[技能与已学检查](../../../raw/current/scripts/components/builder.lua#L869-L888)、[锻造配方](../../../raw/current/scripts/recipes.lua#L848-L859)
- **按产出换算**：1 个外壳做成弹药是 20 发，做成亮茄盔甲只占 4 个外壳里的 1 个（推导，非实测）；两条配方的材料与产出都取自同版 `recipes.lua`。

## 堆叠与标签边界

| 属性 | 本物品 | 消费者／证据 |
|---|---|---|
| 堆叠上限 | 20（`STACK_SIZE_MEDITEM`） | `stackable` 赋值；[常量](../../../raw/current/scripts/tuning.lua#L82) |
| 游戏标签 | **无**（`AddTag` 零次调用） | [全文件](../../../raw/current/scripts/prefabs/lunarplant_husk.lua#L6-L35) |
| 可食用 | 否，没有 `edible` | 同上；对照 [[reference/prefabs/thulecite]] 的 `edible` |
| 可交易 | 否，没有 `tradable` | 同上 |
| 可作饵／被鼹鼠偷 | 否，没有 `bait`、没有 `molebait` | 同上 |
| 可修复别物 | 否，没有 `repairer` | 同上 |

- **不要把它当"月亮系材料都能吃"的证据**：同批发光材料 `thulecite` 有 `edible`（`FOODTYPE.ELEMENTAL`），本物品没有；采集册只记堆叠与动画，不记组件，不能反推。[采集册条目](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L807)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`inventoryitem`／`stackable` | 背包物品与堆叠 20 | 仅主模拟 | [[reference/components/inventoryitem]]；[接线](../../../raw/current/scripts/prefabs/lunarplant_husk.lua#L28-L31) |
| 组件：`hauntable` | `MakeHauntableLaunch`：作祟抛飞，概率 `HAUNT_CHANCE_ALWAYS`、值 `HAUNT_TINY` | 仅主模拟 | [helper](../../../raw/current/scripts/standardcomponents.lua#L1053-L1068) |
| 关联 Prefab：`lunarthrall_plant` | 唯一掉落来源（2 外壳＋2 植物肉） | 死亡时 `DropLoot` | [掉落表](../../../raw/current/scripts/prefabs/lunarthrall_plant.lua#L25-L30)（Wiki 待建） |
| 数据：`recipes.lua` | 9 条月之锻造配方＋1 条弹弓弹药 | 全部 `TECH.LUNARFORGING_TWO`，弹药另需技能 | 见[月之锻造配方消费者](#月之锻造配方消费者) |
| 资源：`anim/lunarplant_husk.zip` | bank／build 同名，播 `idle` | 原版资源，不随页分发 | [assets](../../../raw/current/scripts/prefabs/lunarplant_husk.lua#L1-L4) |

## 常见陷阱与限制

- **误区：`armor_lunarplant_husk` 是外壳的护甲形态。** 两者是不同注册：`armor_lunarplant_husk`（荆棘茄甲）在 `prefabs/armor_lunarplant.lua` 里由 `commonfn` 注册，走 BODY 装备槽，和本页的材料物品没有工厂关系。[[reference/prefabs/armor-rift]]、[材料注册](../../../raw/current/scripts/prefabs/lunarplant_husk.lua#L38)
- **边界：不可食用、不可燃烧、不可作饵。** 组件与标签清单见[堆叠与标签边界](#堆叠与标签边界)；需要"能喂给石虾"的月系材料时用的是 `thulecite`，不是外壳。
- **边界：掉落只发生在致命亮茄死亡。** 收割、敲碎、火烧都不产出；用 `lootdropper` 改表也得改到 `lunarthrall_plant` 上。
- **未核验**：未运行游戏。上表配方产出（含 20 发弹药与 6 个炸弹）按源码推导，未实测手搓产出与技能加成后的实际堆叠行为。

## 验证记录

- 源码审阅：2026-11-10，Steam buildid 26474873。通读 `prefabs/lunarplant_husk.lua` 全文 38 行；`prefabs/lunarthrall_plant.lua` 的掉落表、`lootdropper` 挂载、`OnDeath`、`plant` 标签与 `MakeLargeBurnableCharacter`；`components/lootdropper.lua` 的 `DropLoot` 换名与受控火分支；`prefabs/meats.lua` 的 `plantmeat_cooked` 注册；`recipes.lua` 中 10 条含 `lunarplant_husk` 的配方逐行核对，以及 `calc_slingshotammo_numtogive`；`tuning.lua` 的 `STACK_SIZE_MEDITEM`；`components/stackable.lua` 默认值与 `standardcomponents.lua` 的 `MakeHauntableLaunch`；`scrapbookdata.lua` 条目；`chinese_s.po` 名称键。
- 未覆盖：致命亮茄的战斗、刷新与月之锻造台本体；弹药的伤害与位面数值（归 [[reference/prefabs/ranged-weapons]]）；其它角色技能对月系材料的额外读取。
- 模板的"典型接入"节省略：本物品没有任何接入点（无组件回调、无事件、无施法入口），按模板"无已核验案例时删除本节"处理。
- 游戏运行：未运行；日期、build 与源码一致，天数／秒数不涉及。
- 复审修订：2026-11-10，P1 条目（检索范围）、R04（火烧换名链）；第二轮 R12（弹药与站内配方分开）。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 见交付回报。

## 相关链接

- 月之锻造与亮茄装备：[[reference/prefabs/armor-rift]]、[[reference/prefabs/ranged-weapons]]、[[reference/prefabs/throwables]]、[[reference/prefabs/staffs-more]]
- 配套材料：[[reference/prefabs/purebrilliance]]、[[reference/prefabs/moonglass-materials]]
- 组件：[[reference/components/inventoryitem]]

返回 [[reference/prefabs/index|Prefab 索引]]。
