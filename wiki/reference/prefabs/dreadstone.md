---
title: "绝望石（dreadstone）"
created: "2026-11-10"
updated: "2026-11-10"
type: prefab
summary: "绝望石的单一定义：从裂隙矿苗、梦魇疯猪柱子与几类暗影怪物获取；作为材料进入裂隙装备与建造计划（单次最多 40 块）；作为修理材料向绝望石墙请求回复 80 生命（按上限截断），并带一组被 health 分支短路的 work 修理值。页面说明它不是燃料、只有 repairer 生成的两个材质标签，以及堆叠 40 与漂浮参数。"
aliases: ["dreadstone", "绝望石", "恐惧石", "绝望石矿苗", "dreadstone_stack", "暗影石"]
tags: [prefab, item]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/dreadstone.lua", "prefabs/nightmarefissure.lua", "prefabs/daywalker_pillar.lua", "prefabs/shadowthrall_centipede.lua", "prefabs/shrouden.lua", "prefabs/walls.lua", "components/lootdropper.lua", "components/floater.lua", "components/repairer.lua", "components/repairable.lua", "components/health.lua", "componentutil.lua", "components/tradable.lua", "components/shadowthrallmanager.lua", "componentactions.lua", "standardcomponents.lua", "stategraphs/SGshrouden.lua", "recipes.lua", "tuning.lua", "constants.lua", "languages/chinese_s.po", "screens/redux/scrapbookdata.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# 绝望石（dreadstone）

## 快速判断

- 职责与适用：`prefabs/dreadstone.lua` 只注册一个 `dreadstone`，是最小物品骨架——`idle` 单帧图形、`inventoryitem`＋`stackable`＋`inspectable`＋`tradable`＋`repairer`，没有 `fuel`／`waterproofer`／`edible`。[组件段](../../../raw/current/scripts/prefabs/dreadstone.lua#L29-L43)
- 优先使用：查“去哪挖”看[获取](#获取)；查“配方要几个”看[作为材料](#作为材料)；查“能不能修墙、修多少”看[作为修理材料](#作为修理材料)。
- 重要副作用：`repairer` 给它打上 `health_dreadstone`／`work_dreadstone` 两个标签，这是本物品**唯一的自定义标签**；修理材质不是 `dreadstone` 的目标在比对处失败。[标签生成](../../../raw/current/scripts/components/repairer.lua#L8-L24)、[材质比对](../../../raw/current/scripts/components/repairable.lua#L153-L156)
- 修改边界：矿苗、墙体与护甲的耐久机制归各自页面；燃料侧与 `tradable` 的误解见[常见陷阱](#常见陷阱与限制)。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `dreadstone`（绝望石） | 独立 `fn()`：实体→Transform/AnimState/Network→`MakeInventoryPhysics`→bank/build `dreadstone`＋`idle`（不循环）→`pickupsound = "rock"`→漂浮→`SetPristine`→主模拟挂组件 | 资源只有 `anim/dreadstone.zip`；无 `prefabs` 依赖表。[工厂](../../../raw/current/scripts/prefabs/dreadstone.lua#L6-L46)、[注册](../../../raw/current/scripts/prefabs/dreadstone.lua#L48) |

构造细节（按执行顺序）：

- **表现与漂浮**：bank/build 都是 `dreadstone`、播单帧 `idle`、`pickupsound = "rock"`；`MakeInventoryFloatable(inst, "med", .145, { .77, .75, .77 })` 的第 4 参是**三轴缩放**（0.77／0.75／0.77），不是染色，helper 不加额外标签。[表现](../../../raw/current/scripts/prefabs/dreadstone.lua#L15-L21)、[缩放语义](../../../raw/current/scripts/components/floater.lua#L68-L78)、[helper](../../../raw/current/scripts/standardcomponents.lua#L1772-L1791)
- **堆叠**：`TUNING.STACK_SIZE_SMALLITEM`＝40，采集册同值。[赋值](../../../raw/current/scripts/prefabs/dreadstone.lua#L31-L32)、[常量](../../../raw/current/scripts/tuning.lua#L83)、[采集册](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L424)
- **作祟**：`MakeHauntableLaunchAndSmash` 的砸碎分支被 `#HAUNTFIX` 注释掉，实际只有抛飞。[接线](../../../raw/current/scripts/prefabs/dreadstone.lua#L43)、[实现](../../../raw/current/scripts/standardcomponents.lua#L1071-L1096)
- **修理值**：`repairmaterial = MATERIALS.DREADSTONE`（`"dreadstone"`）、`healthrepairvalue = TUNING.REPAIR_DREADSTONE_HEALTH`＝80、`workrepairvalue = TUNING.REPAIR_DREADSTONE_WORK`＝4。[赋值](../../../raw/current/scripts/prefabs/dreadstone.lua#L38-L41)、[常量](../../../raw/current/scripts/tuning.lua#L2131-L2132)、[材质](../../../raw/current/scripts/constants.lua#L1957)

### 名称

显示名取名字键，无 `nameoverride`：`NAMES.DREADSTONE`＝**绝望石**；矿苗的显示名是另一条键 `NAMES.DREADSTONE_STACK`＝**绝望石矿苗**。[物品名](../../../raw/current/scripts/languages/chinese_s.po#L342260-L342262)、[矿苗名](../../../raw/current/scripts/languages/chinese_s.po#L342269-L342272)

## 获取

掉落表数字是**基础概率**：`chance >= 1.0` 直接入列，小于 1 的先过 `GetChance`（按死亡归因实体或最后工作者的正向幸运抬高，无／负幸运回落），同 prefab 列多次即多次判定，`Σchance` 只是无幸运修正期望；矿苗与墙体走 `SpawnLootPrefab` 直发、不 roll。[生成](../../../raw/current/scripts/components/lootdropper.lua#L163-L227)、[幸运公式](../../../raw/current/scripts/componentutil.lua#L2092-L2097)

| 来源 | 产出绝望石 | 条件与证据 |
|---|---|---|
| 裂隙矿苗（`fissure`／`fissure_lower` 上的 `dreadstone_stack` 显示态） | 挖完固定 3 个 | `MINE`、`FISSURE_DREADSTONE_WORK`＝6 点、`SetRequiresToughWork(true)`，完成后循环 3 次 `SpawnLootPrefab("dreadstone")`；480 秒是 `shadowthrallmanager` 的**管理器冷却**（`FISSURE_DREADSTONE_COOLDOWN`）；480 秒后回调执行时读当时的 `_fissure`，那时仍受控才放回（R13）。[挖完](../../../raw/current/scripts/prefabs/nightmarefissure.lua#L341-L358)、[工作量](../../../raw/current/scripts/prefabs/nightmarefissure.lua#L540-L552)、[管理器](../../../raw/current/scripts/components/shadowthrallmanager.lua#L470-L490) |
| 梦魇疯猪留下的柱子 `daywalker_pillar` | 1.00＋1.00＋1.00＋0.50 | `MINE`、`DAYWALKER_PILLAR_MINE`＝10 点；同表还有 0.50 大理石。[表](../../../raw/current/scripts/prefabs/daywalker_pillar.lua#L20-L27)、[可挖](../../../raw/current/scripts/prefabs/daywalker_pillar.lua#L710-L719) |
| 地下裂影蜈蚣 `shadowthrall_centipede_head` | 1.00＋0.66＋0.34 | 同表另掉纯粹恐惧 1.00＋1.00＋0.25、噩梦燃料 1.00＋1.00＋0.50＋0.25＋0.25。[表](../../../raw/current/scripts/prefabs/shadowthrall_centipede.lua#L23-L37) |
| 暗影憎恶 `shrouden` 的 `shrouden2` 表 | 1.00×4＋0.75＋0.50 | 同表另掉暗影碎布与一张素描。[表](../../../raw/current/scripts/prefabs/shrouden.lua#L46-L63)、[接线](../../../raw/current/scripts/stategraphs/SGshrouden.lua#L982-L986) |
| 绝望石墙被锤掉 | `max(1, floor(2 × 当前生命比例))`，即 1–2 块 | 墙体 `loot = "dreadstone"`、`maxloots = 2`，按锤落时的血量比例结算；这是**墙体回收**，与配方拆解 `DeconstructRecipe` 不是一回事。[公式](../../../raw/current/scripts/prefabs/walls.lua#L167-L173)、[数据行](../../../raw/current/scripts/prefabs/walls.lua#L406-L413) |

- **嬗变**：`Recipe2("transmute_horrorfuel", {Ingredient("dreadstone", 1)})` 用 1 块绝望石换 2 个纯粹恐惧；反向 `transmute_dreadstone` 是 3 个纯粹恐惧换 1 块。两者都需要威尔逊 `wilson_allegiance_shadow` 技能、`TECH.NONE`。[配方](../../../raw/current/scripts/recipes.lua#L1495-L1496)

## 作为材料

配方里的数量是“每次制作消耗”，`numtogive` 才是产出件数。

| 产物 | 消耗绝望石 | 其它材料与门槛 | 证据 |
|---|---|---|---|
| `armordreadstone`（绝望石盔甲） | 6 | ＋纯粹恐惧 4，`TECH.LOST` | [配方](../../../raw/current/scripts/recipes.lua#L502) |
| `dreadstonehat`（绝望石头盔） | 4 | ＋纯粹恐惧 4，`TECH.LOST` | [配方](../../../raw/current/scripts/recipes.lua#L503) |
| `wall_dreadstone_item` | 4 | 产出 4 件，`TECH.LOST` | [配方](../../../raw/current/scripts/recipes.lua#L601) |
| `support_pillar_dreadstone_scaffold` | 4 | ＋木板 2，带 placer 与放船检测 | [配方](../../../raw/current/scripts/recipes.lua#L576) |
| `slingshotammo_dreadstone` | 1 | 产出 20 发，Walter 幸运弹药技能 | [配方](../../../raw/current/scripts/recipes.lua#L222) |
| `gelblob_storage_kit` | 5 | ＋凝胶瓶 3，暗影术基座 | [配方](../../../raw/current/scripts/recipes.lua#L876) |
| `shadow_battleaxe`／`saddle_shadow`／`shadow_beef_bell` | 3／8／4 | 均需注入暗影之心＋纯粹恐惧＋暗影碎布 | [配方](../../../raw/current/scripts/recipes.lua#L873-L875) |
| `shadow_forge_kit` | 2 | ＋噩梦燃料 5＋纯粹恐惧 1 | [配方](../../../raw/current/scripts/recipes.lua#L811) |
| 建造计划 `CONSTRUCTION_PLANS` | 40／5 | `support_pillar_dreadstone` 40 块（映射到脚手架键）、`charlie_hand` 5 块；与 `Recipe2` 那套是两张表 | [40](../../../raw/current/scripts/recipes.lua#L1537)、[映射](../../../raw/current/scripts/recipes.lua#L1553-L1554)、[5](../../../raw/current/scripts/recipes.lua#L1533) |
| 拆解 `shadow_forge`／`gelblob_storage` | 2／5 | 拆解回收建造成本，不是新配方 | [拆解](../../../raw/current/scripts/recipes.lua#L1661-L1664) |

装备与建筑侧机制不在本页展开：护甲归 [[reference/prefabs/armor-rift|裂隙三护甲族]]、帽子归 [[reference/prefabs/hats-armor|防护与战斗帽族]]、墙体与柱子归各自页面。

## 作为修理材料

`repairer` 只在数值大于 0 时打标签，所以绝望石同时带 `health_dreadstone` 与 `work_dreadstone`；动作候选要求目标带 `repairable_dreadstone`，再按 `healthrepairable`／`workrepairable` 分支给 `REPAIR`。[标签](../../../raw/current/scripts/components/repairer.lua#L8-L24)、[候选](../../../raw/current/scripts/componentactions.lua#L1688-L1697)

- 全库唯一带 `repairable_dreadstone` 的是绝望石墙：墙体的 `repairmaterial` 直接取 `data.name`，该行 `name = MATERIALS.DREADSTONE`；墙的建造件 `wall_dreadstone_item` 自己也是同材质的修理件，`healthrepairvalue = REPAIR_DREADSTONE_HEALTH * 4`＝320。[墙赋值](../../../raw/current/scripts/prefabs/walls.lua#L324-L327)、[数据行](../../../raw/current/scripts/prefabs/walls.lua#L406-L413)、[建造件修理值](../../../raw/current/scripts/prefabs/walls.lua#L223-L225)
- 结算顺序是“先 health 后 work”：目标有 `health` 且修理件 `healthrepairvalue > 0` 时走 `health:DoDelta(80)` 并直接返回。80 是**请求值**，`Health:SetVal` 按上限截断，残血墙只补到满；`800 ÷ 80`＝10 只是“每次都完整加 80”的算术，不是保底块数（R08）。[结算](../../../raw/current/scripts/components/repairable.lua#L106-L124)、[截断](../../../raw/current/scripts/components/health.lua#L565-L574)、[墙血量](../../../raw/current/scripts/tuning.lua#L2159)
- 因此 `workrepairvalue`＝4 在这一材质上没有生效路径：墙虽然也挂 `HAMMER` 的 `workable`，但 health 分支先返回，`work_dreadstone` 标签不会命中。[workable](../../../raw/current/scripts/prefabs/walls.lua#L364-L368)
- 消耗方式：成功一次扣 1 块（走 `stackable:Get():Remove()`），满耐久直接返回 false 不扣。[消耗](../../../raw/current/scripts/components/repairable.lua#L164-L176)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`repairer` | `MATERIALS.DREADSTONE`＋health 80／work 4，打两个材质标签 | 主模拟；每次成功消耗 1 块 | [[reference/components/repairer]]；[赋值](../../../raw/current/scripts/prefabs/dreadstone.lua#L38-L41) |
| 组件：`stackable` | `maxsize = STACK_SIZE_SMALLITEM`（40） | 主模拟 | [[reference/components/stackable]]；[赋值](../../../raw/current/scripts/prefabs/dreadstone.lua#L31-L32) |
| 组件：`tradable` | 挂载但不设 `goldvalue`（默认 0） | 主模拟 | [[reference/components/tradable]]；[挂载](../../../raw/current/scripts/prefabs/dreadstone.lua#L29) |
| 组件：`inventoryitem`／`floater` | 可拾取；`med` 漂浮体、偏移 0.145、灰紫染色 | 两端；漂浮在 pristine 前构造 | [[reference/components/inventoryitem]]；[漂浮](../../../raw/current/scripts/prefabs/dreadstone.lua#L21) |
| 关联 Prefab：`fissure`／`fissure_lower` | 矿苗宿主：`AllowShadowThralls` 名单内的裂隙才有 `workable`＋`lootdropper`，并用 `SetPrefabNameOverride` 改成矿苗显示名 | 世界生成；`fissure_grottowar` 不在名单内 | [名单](../../../raw/current/scripts/prefabs/nightmarefissure.lua#L299-L302)、[显示名](../../../raw/current/scripts/prefabs/nightmarefissure.lua#L360-L368) |
| 关联 Prefab：`wall_dreadstone` | 唯一修理目标；锤拆按血量比例回吐 1–2 块 | 建造后常驻 | [墙体数据](../../../raw/current/scripts/prefabs/walls.lua#L406-L413) |
| 关联 Prefab：`daywalker_pillar` | 梦魇疯猪柱子，可挖 10 点 | 由梦魇疯猪事件生成 | [表](../../../raw/current/scripts/prefabs/daywalker_pillar.lua#L20-L27) |

## 常见陷阱与限制

- **误区：绝望石是暗影燃料。** 它没有 `fuel` 组件，采集册行也没有 `fueltype`／`fuelvalue`；燃料侧只有噩梦燃料与纯粹恐惧。[组件段](../../../raw/current/scripts/prefabs/dreadstone.lua#L29-L43)、[采集册](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L424)
- **边界：矿苗不是独立 prefab。** 它是裂隙在 `OnDreadstoneMineCooldown` 里把显示名改成矿苗键、并显示 `stack_*` 符号换出来的外观，实体始终是裂隙；挖完再清掉名字覆盖收起。[改名](../../../raw/current/scripts/prefabs/nightmarefissure.lua#L360-L368)、[收起](../../../raw/current/scripts/prefabs/nightmarefissure.lua#L341-L348)
- **边界：矿苗初始不可挖。** `workable:SetWorkable(false)` 是构造时的状态，只有 `OnDreadstoneMineCooldown`（再生到点、读档、或被 corruption 命中）才打开；`OnReleasedFromControl` 会播 `shadowrock_down`、隐藏矿苗并再次关掉。[构造](../../../raw/current/scripts/prefabs/nightmarefissure.lua#L540-L552)、[收起](../../../raw/current/scripts/prefabs/nightmarefissure.lua#L411-L433)
- **边界：`tradable` 不等于能卖。** 构造时挂了 `tradable` 但没设 `goldvalue`，取组件默认 0；别把“有 tradable 组件”当成有交易价值。[挂载](../../../raw/current/scripts/prefabs/dreadstone.lua#L29)、[默认](../../../raw/current/scripts/components/tradable.lua#L3)
- **未核验**：未运行游戏。矿苗出现的节奏（受 `shadowthrallmanager` 时序影响）只按源码描述，未实测。

## 验证记录

- 源码审阅：2026-11-10，Steam buildid 26474873。通读 `prefabs/dreadstone.lua`；核 `prefabs/nightmarefissure.lua` 的 `AllowShadowThralls`、矿苗三段与 `Make` 工厂，`components/shadowthrallmanager.lua` 的冷却回调，`prefabs/daywalker_pillar.lua`／`shadowthrall_centipede.lua`／`shrouden.lua`／`stategraphs/SGshrouden.lua` 的掉落，`prefabs/walls.lua` 的墙体数据与锤拆公式，`components/repairer.lua`／`repairable.lua`／`health.lua`／`componentactions.lua` 的修理链，`recipes.lua` 全部相关行，`tuning.lua`／`constants.lua` 的常量；名称取 `chinese_s.po`，堆叠用 `scrapbookdata.lua` 核对。
  未覆盖：墙体完整耐久流程、建造计划与拆解的触发时序、模组注入。
- 游戏运行：未运行。挖掘工作量、480 秒冷却与修理块数均为源码推导。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 见交付回报。

## 相关链接

- 装备与建筑消费者：[[reference/prefabs/armor-rift]]（绝望石盔甲）、[[reference/prefabs/hats-armor]]（绝望石头盔）、[[reference/prefabs/shadow-battleaxe]]
- 原料页：[[reference/prefabs/nightmare-fuel]]（噩梦燃料／纯粹恐惧）、[[reference/prefabs/voidcloth]]（暗影碎布）
- 组件：[[reference/components/repairer]]、[[reference/components/repairable]]、[[reference/components/stackable]]

返回 [[reference/prefabs/index|Prefab 索引]]。
