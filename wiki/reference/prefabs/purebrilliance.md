---
title: "纯粹辉煌（purebrilliance）"
created: "2026-11-10"
updated: "2026-11-10"
type: prefab
summary: "月系高阶材料 purebrilliance 与它的符号跟随件 purebrilliance_symbol_fx：堆叠 40、可交易、带施法组件（仅沃特月之誓约技能可用），主消费是弹弓弹药、薇诺娜的宝石发电机与月之锻造配方。"
aliases: ["purebrilliance", "纯粹辉煌", "Pure Brilliance", "纯辉", "辉煌碎片"]
tags: [prefab, item]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/purebrilliance.lua", "components/spellcaster.lua", "prefabs/skilltree_wurt.lua", "componentactions.lua", "prefabs/winona_battery_high.lua", "prefabs/birdcage.lua", "prefabs/birds_mutant_rift.lua", "prefabs/lunarrift_crystals.lua", "prefabs/gestalt_guard_evolved.lua", "prefabs/alterguardian_phase1.lua", "prefabs/alterguardian_phase4_lunarrift.lua", "components/lunaralterguardianspawner.lua", "prefabs/bearger.lua", "prefabs/deerclops.lua", "prefabs/warg.lua", "prefabs/slingshotammo.lua", "recipes.lua", "constants.lua", "tuning.lua", "components/stackable.lua", "components/tradable.lua", "prefabs/pigking.lua", "prefabs/staff_common.lua", "prefabs/moonglass.lua", "screens/redux/scrapbookdata.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# 纯粹辉煌（purebrilliance）

## 快速判断

- 职责与适用：`prefabs/purebrilliance.lua` 一次返回两个注册——物品 `purebrilliance`（官方简中"纯粹辉煌"）与纯视觉件 `purebrilliance_symbol_fx`。物品带 `spellcaster`（月系少见）：堆叠 40、可交易、可作祟、可施法。[注册](../../../raw/current/scripts/prefabs/purebrilliance.lua#L128-L129)、[构造](../../../raw/current/scripts/prefabs/purebrilliance.lua#L51-L99)
- 优先使用：查弹药／法术／发电机怎么吃它看[弹药、法术与电池](#弹药法术与电池)；查来路看[获取路径](#获取路径)；查"为什么别人按不了施法"看[常见陷阱与限制](#常见陷阱与限制)。
- 重要副作用：施法走 `SPELLTYPES.WURT_LUNAR`，`CanCast` 先要求施法者带 `wurt_lunar_spelluser`——只由沃特技能树给出，其它角色连 `CAST` 候选都拿不到；成功施法消耗 1 个。[组件](../../../raw/current/scripts/components/spellcaster.lua#L152-L157)、[技能标签](../../../raw/current/scripts/prefabs/skilltree_wurt.lua#L424)、[消耗](../../../raw/current/scripts/prefabs/purebrilliance.lua#L36-L49)
- 修改边界：弹药的命中标记归 [[reference/prefabs/ranged-weapons]] 与 [[reference/prefabs/slingshotammo_purebrilliance_debuff_fx]]；发电机本体 `winona_battery_high`（Wiki 待建）；月亮碎片两态归 [[reference/prefabs/moonglass-materials]]。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `purebrilliance` | `fn()`：实体 → Transform／AnimState／Network → `MakeInventoryPhysics` → `CommonSetupAnims(inst, "idle")`（bank／build 同名、bloom）→ 加 `purebrilliance`／`mermbuffcast` → `pickupsound="gem"` → `MakeInventoryFloatable(inst, "small", .1)` → `SetPristine` → 主模拟随机帧＋挂件 | 物品形态；`stackable.maxsize = TUNING.STACK_SIZE_SMALLITEM`＝40；`prefabs` 表声明 `wurt_merm_planar`／两个 castfx，本文件不生成。[构造](../../../raw/current/scripts/prefabs/purebrilliance.lua#L51-L99)、[依赖](../../../raw/current/scripts/prefabs/purebrilliance.lua#L7-L12) |
| `purebrilliance_symbol_fx` | `symbolfxfn()`：实体 → Transform／AnimState／**Follower**／Network → `CommonSetupAnims(inst, "idle_sparkle")` → 加 `FX`／`NOCLICK` → `SetPristine` → 主模拟随机帧、`persists = false` | 纯 FX，无 inventoryitem／堆叠；由发电机宝石座 `SetGem` 时生成并 `FollowSymbol`；**关联复用**（工厂与槽位生命周期主属 [[reference/prefabs/winona_battery_high_shatterfx]]）。[工厂](../../../raw/current/scripts/prefabs/purebrilliance.lua#L102-L126)、[使用方](../../../raw/current/scripts/prefabs/winona_battery_high.lua#L268-L274) |

- **主模拟分界**：两工厂都在 `SetPristine` 后按 `ismastersim` 提前返回；组件、随机帧与 `lightcolour`／`fxprefab`／`castsound` 只在主模拟段。[端分界](../../../raw/current/scripts/prefabs/purebrilliance.lua#L66-L93)
- **堆叠是显式赋值**：40 与 `stackable` 默认的 20 不同，采集册也记 `stacksize=40`。[赋值](../../../raw/current/scripts/prefabs/purebrilliance.lua#L81-L82)、[常量](../../../raw/current/scripts/tuning.lua#L83)、[默认值](../../../raw/current/scripts/components/stackable.lua#L23-L28)、[采集册](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L1195)

### 名称核验

`STRINGS.NAMES.PUREBRILLIANCE` 的 `msgstr` 是"纯粹辉煌"，没有改名调用。[名称键](../../../raw/current/scripts/languages/chinese_s.po#L347384-L347387)

## 获取路径

| 来源 | 产出与参数 | 证据 |
|---|---|---|
| 亮喙鸟（`mutatedbird`）＋鸟笼 | 喂满 3 个注能月亮碎片后置位；**只有那一份仍是注能碎片**的消化才产出 1 个并清位。冷却 480 秒，冷却期拒收注能碎片 | [置位](../../../raw/current/scripts/prefabs/birdcage.lua#L206-L212)、[那一份](../../../raw/current/scripts/prefabs/birdcage.lua#L112-L118)、[拒收](../../../raw/current/scripts/prefabs/birdcage.lua#L172-L177)、[阈值](../../../raw/current/scripts/tuning.lua#L8837)、[冷却](../../../raw/current/scripts/prefabs/birds_mutant_rift.lua#L93-L95) |
| 裂隙水晶（小／大） | 小 `{"purebrilliance", 0.75}`；大 1.00＋0.75 | [小](../../../raw/current/scripts/prefabs/lunarrift_crystals.lua#L234-L237)、[大](../../../raw/current/scripts/prefabs/lunarrift_crystals.lua#L195-L199) |
| 敌意虚影（`gestalt_guard_evolved`） | 三套表：1.0＋0.2；1.0＋0.5＋0.2；1.0＋0.8＋0.5＋0.2 | [三张表](../../../raw/current/scripts/prefabs/gestalt_guard_evolved.lua#L32-L49) |
| 天体后裔／仇灵 | 后裔 5×1.0＋0.7＋0.3；仇灵（`alterguardian_phase1_lunarrift`）2×1.0，由大裂隙刷新器生成 | [后裔](../../../raw/current/scripts/prefabs/alterguardian_phase4_lunarrift.lua#L46-L60)、[仇灵表](../../../raw/current/scripts/prefabs/alterguardian_phase1.lua#L49-L71)、[挂表](../../../raw/current/scripts/prefabs/alterguardian_phase1.lua#L626)、[刷新器](../../../raw/current/scripts/components/lunaralterguardianspawner.lua#L58-L70) |
| 变异熊獾／巨鹿／座狼 | 三张表各 1.0＋0.75 | [熊獾](../../../raw/current/scripts/prefabs/bearger.lua#L90-L99)、[巨鹿](../../../raw/current/scripts/prefabs/deerclops.lua#L343-L354)、[座狼](../../../raw/current/scripts/prefabs/warg.lua#L143-L160) |
| 威尔逊嬗变 | `moonglass_charged`×3 → 1 个；反向 1 个 → **2 个**注能碎片 | [正向](../../../raw/current/scripts/recipes.lua#L1499)、[反向](../../../raw/current/scripts/recipes.lua#L1500) |

- **掉落表读法**：每条 `{prefab, chance}` 独立 roll，`chance>=1.0` 必掉，其余过 `GetChance`；"1.0＋0.7＋0.3"期望 2 个。[结算](../../../raw/current/scripts/components/lootdropper.lua#L186-L227)、[GetChance](../../../raw/current/scripts/components/lootdropper.lua#L173-L184)
- **嬗变不对称**：3 换 1，反向 1 换 2（`numtogive=2`），往返净消耗 1 个；两条都 `TECH.NONE`＋技能 `wilson_allegiance_lunar`。[L1499-L1500](../../../raw/current/scripts/recipes.lua#L1499-L1500)

## 弹药、法术与电池

| 消费者 | 机制 | 门禁与条件 | 证据 |
|---|---|---|---|
| `slingshotammo_purebrilliance` | 1 个 → 20 发；命中挂 `ammo_purebrilliance_mark`＋`slingshotammo_purebrilliance_debuff` | `TECH.LUNARFORGING_TWO`＋沃尔特 `walter_ammo_lunar`；**无 `nounlock`**：首次在月之锻造台旁学会，之后离站可做、技能仍生效；`walter_ammo_efficiency` 时 ×1.5＝30 | [配方](../../../raw/current/scripts/recipes.lua#L229)、[加成](../../../raw/current/scripts/recipes.lua#L199-L204)、[标记](../../../raw/current/scripts/prefabs/slingshotammo.lua#L845) |
| `spellcaster`（`WURT_LUNAR`） | 遍历 `doer.components.leader.followers`，给带 `lunarminion` 且未死的随从加 `wurt_merm_planar`，并在 `.3*math.random()` 秒内随机延迟推 `cheer`；随后消耗 1 个 | `canuseontargets`／`canusefrominventory`／`canonlyuseonlocomotorspvp`；施法者须为 `wurt_lunar_spelluser`；`mermbuffcast` 只决定动作（`SGwilson` 映射 `mermbuffcastspell`） | [法术体](../../../raw/current/scripts/prefabs/purebrilliance.lua#L36-L49)、[接线](../../../raw/current/scripts/prefabs/purebrilliance.lua#L84-L89)、[门禁](../../../raw/current/scripts/prefabs/skilltree_wurt.lua#L424)、[SG](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1312) |
| 薇诺娜的宝石发电机 | `SetGem` 生成 symbol fx 并 `_brilliance_level + 1`；`RefreshEnergyFX` 开能量、`m2` 色相 `0.3083`／饱和 `0.65`；取出时 `- 1` | 放入者须已激活 `winona_lunar_2`，否则 `NOGENERATORSKILL`；跟随件 `persists=false` | [SetGem](../../../raw/current/scripts/prefabs/winona_battery_high.lua#L268-L274)、[UnsetGem](../../../raw/current/scripts/prefabs/winona_battery_high.lua#L302-L303)、[能量](../../../raw/current/scripts/prefabs/winona_battery_high.lua#L223-L235)、[门禁](../../../raw/current/scripts/prefabs/winona_battery_high.lua#L1152-L1164) |
| 配方材料 | 涉及记录 22 条：19 条 `Recipe2` 消耗（含反向嬗变）＋1 条正向产出＋2 条 `DeconstructRecipe` 回收 | 锻造台系 `TECH.LUNARFORGING_TWO`＋`station_tag="lunar_forge"`＋`nounlock`（跳过解锁、每次站旁） | [站点配方](../../../raw/current/scripts/recipes.lua#L848-L859)、[建造包](../../../raw/current/scripts/recipes.lua#L840) |

- **施法链两道过滤**：`componentactions` 先剔除 `HasTag(v.."_spellcaster") and not doer:HasTag(v.."_spelluser")`，`CanCast` 再查 `spelltype.."_spelluser"`；只改物品标签绕不过第二道。[候选过滤](../../../raw/current/scripts/componentactions.lua#L1781)、[CanCast](../../../raw/current/scripts/components/spellcaster.lua#L152-L157)

## 堆叠与标签边界

| 属性 | 本物品 | 消费者／证据 |
|---|---|---|
| 堆叠上限 | 40（`STACK_SIZE_SMALLITEM`） | [赋值](../../../raw/current/scripts/prefabs/purebrilliance.lua#L81-L82)、[常量](../../../raw/current/scripts/tuning.lua#L83) |
| 游戏标签 | `purebrilliance`、`mermbuffcast`；前者读取端本版未找到（未核验，不作过滤依据） | [标签](../../../raw/current/scripts/prefabs/purebrilliance.lua#L61-L62) |
| 可食用 | 否，没有 `edible` | [构造](../../../raw/current/scripts/prefabs/purebrilliance.lua#L77-L96) |
| 可交易 | 有 `tradable`，`goldvalue` 默认 0，猪王不收 | [默认值](../../../raw/current/scripts/components/tradable.lua#L1-L7)、[门槛](../../../raw/current/scripts/prefabs/pigking.lua#L575) |
| 拾取音 | `gem` | [赋值](../../../raw/current/scripts/prefabs/purebrilliance.lua#L64) |

- **不算宝石但被当宝石处理**：`staff_common.lua` 的 gem 音效组把它与六色宝石并列。[音效分组](../../../raw/current/scripts/prefabs/staff_common.lua#L566-L567)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`spellcaster`／`stackable`／`inventoryitem`／`tradable`／`hauntable` | 施法三回调、40 叠、背包与交易、作祟抛飞 | 仅主模拟；施法者需 `wurt_lunar_spelluser` | [[reference/components/spellcaster]]、[[reference/components/stackable]]；[接线](../../../raw/current/scripts/prefabs/purebrilliance.lua#L77-L89)、[helper](../../../raw/current/scripts/standardcomponents.lua#L1053-L1068) |
| 关联 Prefab：`purebrilliance_symbol_fx` | 第二注册，关联复用，供发电机符号跟随 | 仅 `SetGem` 时生成；槽位生命周期归 FX 页 | [[reference/prefabs/winona_battery_high_shatterfx]]；[工厂](../../../raw/current/scripts/prefabs/purebrilliance.lua#L102-L126) |
| 关联 Prefab：`wurt_merm_planar`／两个 castfx | `prefabs` 表声明为依赖，本文件不生成 | 施法时由 debuff／FX 取用 | [依赖](../../../raw/current/scripts/prefabs/purebrilliance.lua#L7-L12)（Wiki 待建） |
| 关联 Prefab：`slingshotammo_purebrilliance` | 弹药产物，命中挂同名 debuff | 需锻造台二级＋沃尔特技能 | [[reference/prefabs/ranged-weapons]]、[[reference/prefabs/slingshotammo_purebrilliance_debuff_fx]] |
| 关联 Prefab：`winona_battery_high` | 宝石座消费，进 brilliance 模式 | 仅该设施；放入者需 `winona_lunar_2` | [源码](../../../raw/current/scripts/prefabs/winona_battery_high.lua#L268-L274)（Wiki 待建） |
| 资源：`anim/purebrilliance.zip`、`INV_IMAGE` | 物品与 FX 共用 bank／build | 原版资源，不随页分发 | [assets](../../../raw/current/scripts/prefabs/purebrilliance.lua#L1-L5) |

## 常见陷阱与限制

- **误区：带 `mermbuffcast` 就能施法。** 门禁是 `wurt_lunar_spelluser`（沃特 `wurt_lunar_allegiance_1`）；`canonlyuseonlocomotorspvp` 还要求目标是带 `locomotor` 的非队友或 PVP 开启。[候选过滤](../../../raw/current/scripts/componentactions.lua#L1781)、[CanCast](../../../raw/current/scripts/components/spellcaster.lua#L190-L199)、[技能](../../../raw/current/scripts/prefabs/skilltree_wurt.lua#L424)
- **误区：宝石座来者不拒。** 还查角色技能：未激活 `winona_lunar_2` 者返回 `NOGENERATORSKILL`。[门禁](../../../raw/current/scripts/prefabs/winona_battery_high.lua#L1152-L1164)
- **边界：施法消耗 1 个。** 有堆叠走 `stackable:Get():Remove()`，否则 `inst:Remove()`。[消耗](../../../raw/current/scripts/prefabs/purebrilliance.lua#L44-L48)
- **边界：`RIFT_BIRD_BRILLIANCE_CHANCE = 0.33` 不是掉率。** 只有定义、无读取端；输出由"3 个碎片置位＋那一份仍是注能碎片"决定。[定义](../../../raw/current/scripts/tuning.lua#L8852)、[结算](../../../raw/current/scripts/prefabs/birdcage.lua#L206-L212)
- **边界：与 `moonglass_charged` 不同物。** 后者带 `perishable`（720 秒，耗尽变 `moonglass`），纯粹辉煌无 `edible`、无新鲜度。[腐坏](../../../raw/current/scripts/prefabs/moonglass.lua#L139-L143)、[常量](../../../raw/current/scripts/tuning.lua#L6249)
- **未核验**：未运行游戏。"那一份注能碎片的消化必掉 1 个"按 `DigestFood` 顺序推导；冷却期拒收与 brilliance 能量效果未实测。

## 验证记录

- 源码审阅：2026-11-10，Steam buildid 26474873。`purebrilliance.lua` 全文、`spellcaster.lua` 与 `_spellcaster` 过滤、`skilltree_wurt.lua` 标签、`birdcage.lua` 置位／消化／拒收、`birds_mutant_rift.lua` 注册与冷却、`lunarrift_crystals.lua` 两表、`gestalt_guard_evolved.lua`、`alterguardian_phase1`／`phase4` 与 `lunaralterguardianspawner.lua`、`bearger`／`deerclops`／`warg`、`winona_battery_high.lua`、`recipes.lua` 22 条与嬗变、`lootdropper.lua`、`tuning.lua`、`constants.lua`、`tradable.lua`、`pigking.lua`、`scrapbookdata.lua`、`chinese_s.po`。
- 未覆盖：同名 debuff 层数与伤害（归弹药页）；发电机充能曲线；沃特技能树其余节点。
- 模板的"典型接入"节省略：施法链被角色技能标签锁死，无可复用接入点。
- 游戏运行：未运行；秒数按 `total_day_time`＝480 秒换算。
- 复审修订：2026-11-10，P1 嬗变产出、P2 获取源／发电机门禁／亮喙鸟名、P3 消化限定；R10 记录口径、R14 FX 归属；R12 弹药与站内配方分开。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 见交付回报。

## 相关链接

- 月系装备与弹药：[[reference/prefabs/ranged-weapons]]、[[reference/prefabs/armor-rift]]、[[reference/prefabs/staffs-more]]、[[reference/prefabs/throwables]]、[[reference/prefabs/lunarplant-tools]]
- 配套材料：[[reference/prefabs/lunarplant-husk]]、[[reference/prefabs/moonglass-materials]]、[[reference/prefabs/moonrocknugget]]
- 施法：[[reference/components/spellcaster]]、[[reference/prefabs/wurt_casting_fx]]

返回 [[reference/prefabs/index|Prefab 索引]]。
