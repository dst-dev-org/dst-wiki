---
title: "铥矿与铥矿碎片（thulecite、thulecite_pieces）"
created: "2026-11-10"
updated: "2026-11-10"
type: prefab
summary: "远古遗迹的两种矿产材料：铥矿（thulecite）与铥矿碎片（thulecite_pieces）分属两个注册、两套数值——碎片可 6 换 1 合成铥矿；两者同为 ELEMENTAL 可食物、都能修远古建筑（数值差 6 倍）、能当陷阱诱饵、会被鼹鼠偷，堆叠 20／40。"
aliases: ["thulecite", "thulecite_pieces", "铥矿", "铥矿碎片", "图勒石", "图勒碎片", "Thulecite", "Thulecite Fragments", "远古材料", "铥矿合成"]
tags: [prefab, item]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/thulecite.lua", "prefabs/thulecite_pieces.lua", "recipes.lua", "constants.lua", "tuning.lua", "components/edible.lua", "components/stackable.lua", "components/repairer.lua", "components/repairable.lua", "components/health.lua", "components/builder.lua", "componentutil.lua", "recipe.lua", "components/bait.lua", "components/tradable.lua", "components/lootdropper.lua", "components/trap.lua", "components/inventoryitem.lua", "standardcomponents.lua", "componentactions.lua", "prefabs/walls.lua", "prefabs/altar_prototyper.lua", "prefabs/atrium_statue.lua", "prefabs/minotaur.lua", "prefabs/vault_pillar_guard.lua", "prefabs/statueruins.lua", "prefabs/eyeturret.lua", "stategraphs/SGeyeturret.lua", "prefabs/dustmothden.lua", "prefabs/bishop.lua", "prefabs/knight.lua", "prefabs/rook.lua", "prefabs/archive_props.lua", "prefabs/smashables.lua", "prefabs/chessjunk.lua", "prefabs/antlion.lua", "prefabs/compostingbin.lua", "prefabs/pigking.lua", "prefabs/mole.lua", "prefabs/slurtle.lua", "prefabs/rocky.lua", "brains/molebrain.lua", "screens/redux/scrapbookdata.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 铥矿与铥矿碎片（thulecite、thulecite_pieces）

## 快速判断

- 职责与适用：`prefabs/thulecite.lua` 注册 `thulecite`（官方简中"铥矿"）、`prefabs/thulecite_pieces.lua` 注册 `thulecite_pieces`（"铥矿碎片"）。两者都是 item，但**不是同一工厂**：`thulecite` 有 `edible`、修复值按整块计（工作 1.5／血 100），`thulecite_pieces` 的修复值只有铥矿的 1/6、堆叠是 2 倍。[铥矿注册](../../../raw/current/scripts/prefabs/thulecite.lua#L54)、[碎片注册](../../../raw/current/scripts/prefabs/thulecite_pieces.lua#L55)
- 优先使用：查一块铥矿能修多少血、能修什么看[修复链](#修复链)；查去哪挖、掉多少看[获取](#获取)；查"能不能吃、能不能喂、会不会被偷"看[食用、堆叠与标签](#食用堆叠与标签)；查配方分组看[配方消费者](#配方消费者)。
- 重要副作用：两者都带 `molebait` 标签并有 `bait` 组件——地上的铥矿会被鼹鼠偷走，也能充当陷阱诱饵；`inventoryitem:SetSinks(true)` 让它们落水沉底；`MakeHauntableLaunchAndSmash` **在本版只实现抛飞，砸碎分支被整段注释**。[标签](../../../raw/current/scripts/prefabs/thulecite.lua#L22)、[诱饵组件](../../../raw/current/scripts/prefabs/thulecite.lua#L49)、[沉底](../../../raw/current/scripts/prefabs/thulecite.lua#L43)、[作祟 helper](../../../raw/current/scripts/standardcomponents.lua#L1071-L1104)
- 修改边界：远古科技配方成本归 `recipes.lua`（本页只给分组与代表值）；铥矿甲／铥矿皇冠／铥矿棒／眼睛炮塔等产物本体各有页面；远古戍卫塔的掉落只列材料，不展开其战斗。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `thulecite` | `fn()`：实体 → Transform／AnimState／**SoundEmitter**／Network → `MakeInventoryPhysics` → bank／build `thulecite` 播 `anim` → `pickupsound = "rock"` → `AddTag("molebait")` → `SetPristine` → 客户端返回 → 主模拟：`repairer` → `edible` → `tradable` → `inspectable` → `inventoryitem`＋`SetSinks(true)` → `stackable`（不设 maxsize） → `MakeHauntableLaunchAndSmash` → `bait` | 修复值 work 1.5／health 100；可食 `FOODTYPE.ELEMENTAL`、饥饿 3；堆叠取默认 20（`STACK_SIZE_MEDITEM`）。[工厂](../../../raw/current/scripts/prefabs/thulecite.lua#L6-L52) |
| `thulecite_pieces` | 同一骨架：`SoundEmitter`、`pickupsound="rock"`、`molebait` 标签、`SetPristine` → 主模拟：`edible` → `tradable` → `inspectable` → `inventoryitem`＋`SetSinks(true)` → `bait` → `repairer` → `stackable`（显式 40）→ `MakeHauntableLaunchAndSmash` | 修复值 `1.5/6`＝0.25 work、`100/6`≈16.67 health；饥饿 1；`maxsize = STACK_SIZE_SMALLITEM`＝40。[工厂](../../../raw/current/scripts/prefabs/thulecite_pieces.lua#L6-L53) |

- **数值定义**：`REPAIR_THULECITE_WORK = 1.5`、`REPAIR_THULECITE_HEALTH = 100`、`REPAIR_THULECITE_PIECES_WORK = 1.5/6`、`REPAIR_THULECITE_PIECES_HEALTH = 100/6`；碎片正好是铥矿的六分之一，与"6 碎片合成 1 铥矿"同比例。[常量](../../../raw/current/scripts/tuning.lua#L2111-L2115)、[合成](../../../raw/current/scripts/recipes.lua#L796)
- **食用数值一半靠默认值**：`edible` 构造默认 `healthvalue = 10`、`sanityvalue = 0`；两个 prefab 只覆盖 `hungervalue`（3／1）与 `foodtype`，所以回血 10 来自默认而非本文件。采集册记的 `healthvalue=10` 与此一致。[默认值](../../../raw/current/scripts/components/edible.lua#L19-L24)、[scrapbookdata](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L1512-L1513)
- **堆叠差异不是笔误**：`thulecite` 完全不写 `maxsize`，取 `stackable` 默认 20；`thulecite_pieces` 写死 40。采集册分别记 20 与 40。[组件默认](../../../raw/current/scripts/components/stackable.lua#L23-L28)、[碎片赋值](../../../raw/current/scripts/prefabs/thulecite_pieces.lua#L47-L48)
- **材质字符串**：`MATERIALS.THULECITE = "thulecite"`，与注册名同字面，所以 `repairer` 打出的标签是 `work_thulecite`／`health_thulecite`，与修复目标的 `repairable_thulecite` 按字符串比对。[常量](../../../raw/current/scripts/constants.lua#L1945)、[标签生成](../../../raw/current/scripts/components/repairer.lua#L8-L24)

### 名称核验

`STRINGS.NAMES.THULECITE`＝`Thulecite`／"铥矿"，`STRINGS.NAMES.THULECITE_PIECES`＝`Thulecite Fragments`／"铥矿碎片"；两个 prefab 都没有改名调用，注册名与显示名一一对应。"图勒石／图勒碎片"是俗称，不是游戏内显示名。[两个名称键](../../../raw/current/scripts/languages/chinese_s.po#L350374-L350387)

## 获取

### 碎片：远古遗迹的通用产出

| 来源 | 参数 | 证据 |
|---|---|---|
| 远古雕像 `atrium_statue` | `thulecite_pieces` 两条 1.00＋两条 0.50，期望 3 个；同表还有 `thulecite` 1.00 与 0.25 | [掉落表](../../../raw/current/scripts/prefabs/atrium_statue.lua#L16-L24) |
| 远古戍卫塔 `vault_pillar_guard` | `thulecite_pieces` 1／0.6667／0.3333（期望 2 个）；`thulecite` 1／1／0.5（期望 2.5 个） | [掉落表](../../../raw/current/scripts/prefabs/vault_pillar_guard.lua#L59-L66) |
| 大号华丽箱子 `minotaurchest` | 碎片 12–36、铥矿 8–14，但宝箱只从 9 条战利品里抽 `math.random(max_loots-2, max_loots)` 条，**不保证出铥矿** | [战利品表](../../../raw/current/scripts/prefabs/minotaur.lua#L64-L75)、[抽取](../../../raw/current/scripts/prefabs/minotaur.lua#L623-L639) |
| 铥墙（`ruins`／`ruins_2` 墙） | 被砸毁时掉 `thulecite_pieces`，`maxloots = 2` | [墙体数据](../../../raw/current/scripts/prefabs/walls.lua#L392-L393) |
| 眼睛炮塔 `eyeturret` | 碎片 1.00＋0.75＋0.50＋0.25（期望 2.5 个），同表铥矿 1.00＋0.75（期望 1.75 块） | 死亡状态 `DropLoot`。[掉落表](../../../raw/current/scripts/prefabs/eyeturret.lua#L16-L24)、[触发](../../../raw/current/scripts/stategraphs/SGeyeturret.lua#L41-L51) |
| 尘蛾巢 `dustmothden` | 碎片 1＋1＋1＋0.75＋0.5＋0.25＋0.25（期望 4.75 个），同表铥矿 0.01 | 挖完 `DropLoot`。[掉落表](../../../raw/current/scripts/prefabs/dustmothden.lua#L13-L23)、[接线](../../../raw/current/scripts/prefabs/dustmothden.lua#L157-L158) |
| 梦魇发条三件 `bishop_nightmare`／`knight_nightmare`／`rook_nightmare` | 各 0.5 块碎片（另各 0.6 噩梦燃料） | 三者是独立注册的废墟变体，各自在 `nightmare_master_postinit` 里挂表。[主教](../../../raw/current/scripts/prefabs/bishop.lua#L48-L53)、[骑士](../../../raw/current/scripts/prefabs/knight.lua#L65-L70)、[战车](../../../raw/current/scripts/prefabs/rook.lua#L43-L48) |

- **结算语义（R11）**：`SetSharedLootTable` 每条 `{prefab, chance}` 独立 roll；`chance >= 1.0` 直接入列，小于 1 的先过 `GetChance`——用死亡归因实体（`health.causeofdeath`）或最后工作者（`workable.lastworker`）的**正向**幸运按 `chance × (3 - 6/(luck+3))` 抬高，无幸运／负幸运回落基础值。因此 `Σ chance` 只是**无幸运修正**的基础期望，不是保底；有幸运时逐条用修正后概率再求和。[生成](../../../raw/current/scripts/components/lootdropper.lua#L163-L227)、[幸运公式](../../../raw/current/scripts/componentutil.lua#L2092-L2097)、[接线](../../../raw/current/scripts/componentutil.lua#L2151)
- 废墟花瓶（`smashables`）走 `AddRandomLoot` 加权池，`thulecite` 权重 0.1，且 `numrandomloot = 1`＋`chancerandomloot = 0.05`——只有 5% 概率掉 1 条随机物。[花瓶](../../../raw/current/scripts/prefabs/smashables.lua#L269-L281)
- 棋子垃圾（`chessjunk`）的 `chess_junk` 是**概率表**：`thulecite` 0.01，与其它条目各自独立 roll，不设 `numrandomloot`，所以一次敲碎可能同时出多条。[表](../../../raw/current/scripts/prefabs/chessjunk.lua#L30-L42)、[挂表](../../../raw/current/scripts/prefabs/chessjunk.lua#L197-L198)
- 带宝石的废墟雕像 `statueruins`（无独立 NAMES 键）用 `SetLoot({ "thulecite", gem })` 保证 1 铥矿，再 `AddChanceLoot("thulecite", .05)` 追加一次 5%；**无宝石**版本走 `statue_ruins_no_gem` 表，铥矿 1.00＋0.05、另必掉 1 噩梦燃料。[带宝石](../../../raw/current/scripts/prefabs/statueruins.lua#L194-L199)、[无宝石](../../../raw/current/scripts/prefabs/statueruins.lua#L53-L58)、[接线](../../../raw/current/scripts/prefabs/statueruins.lua#L330)

### 铥矿：主要靠碎片合成

- 远古科技二级、`nounlock = true`（不需要蓝图）：`Recipe2("thulecite", { Ingredient("thulecite_pieces", 6) }, TECH.ANCIENT_TWO, { nounlock = true })`——6 碎片 → 1 铥矿。[合成](../../../raw/current/scripts/recipes.lua#L796)
- 整块掉落常见这几处：远古雕像 1.00＋0.25、远古戍卫塔 1／1／0.5、眼睛炮塔 1.00＋0.75、档案馆月石雕像 1.00（另加 0.05 月石）、无宝石废墟雕像 1.00＋0.05、锤碎破损祭坛 1.00＋1.00、大号华丽箱子 8–14（非保底），以及花瓶与棋子垃圾的低权重随机；尘蛾巢与梦魇发条只给 0.01／0。[雕像](../../../raw/current/scripts/prefabs/atrium_statue.lua#L16-L24)、[柱卫](../../../raw/current/scripts/prefabs/vault_pillar_guard.lua#L59-L66)、[炮塔](../../../raw/current/scripts/prefabs/eyeturret.lua#L16-L24)、[档案馆雕像](../../../raw/current/scripts/prefabs/archive_props.lua#L60-L65)、[无宝石雕像](../../../raw/current/scripts/prefabs/statueruins.lua#L53-L58)、[祭坛](../../../raw/current/scripts/prefabs/altar_prototyper.lua#L53-L60)

## 配方消费者

共 35 条 `recipes.lua` 记录读这两种材料：23 条要铥矿、12 条要碎片。按门槛分组如下（"四／二级"指 `TECH.ANCIENT_FOUR`／`TECH.ANCIENT_TWO`）：

| 分组 | 条数 | 代表与单件成本 | 证据 |
|---|---:|---|---|
| 远古科技二级 | 4 | 铥墙 `wall_ruins_item` 1 出 6、铥矿徽章 2、魔光护符 2、建造护符 2 | [L797-L798](../../../raw/current/scripts/recipes.lua#L797-L798)、[L800-L801](../../../raw/current/scripts/recipes.lua#L800-L801) |
| 远古科技四级 | 6 | 铥矿甲 6、铥矿皇冠 4、铥矿棒 4、眼睛炮塔 5、懒人护符 2、多用斧镐 2 | [L799-L810](../../../raw/current/scripts/recipes.lua#L799-L810) |
| 失落科技 `TECH.LOST` | 3 | 土地夯实器 1、铥矿捕虫网 2、星象探测仪 1 | [L390-L411](../../../raw/current/scripts/recipes.lua#L390-L411) |
| 角色技能（`TECH.NONE`） | 3 | 宝石哑铃／火铃／冰铃各 2 | [L258-L262](../../../raw/current/scripts/recipes.lua#L258-L262) |
| 弹弓／工作站 | 2 | 铥矿弹弓框 2（`ancient_station`）、火花柜 2（WAGPUNK 二级） | [L244](../../../raw/current/scripts/recipes.lua#L244)、[L1021](../../../raw/current/scripts/recipes.lua#L1021) |
| 圣殿建造与拆解 | 5 | 戍卫塔基址 `vault_pillar_guard_constr` 8；拆解眼睛炮塔 5、铥墙 1、星象探测仪 1、远古之塔 8 | [L1551](../../../raw/current/scripts/recipes.lua#L1551)、[L1681](../../../raw/current/scripts/recipes.lua#L1681) |
| 碎片：远古科技二级 | 2 | 6 碎片合成铥矿、高级耕作先驱帽 4 | [L796](../../../raw/current/scripts/recipes.lua#L796)、[L806](../../../raw/current/scripts/recipes.lua#L806) |
| 碎片：弹弓与怀表 | 2 | 诅咒弹药 1＋噩梦燃料 1 出 20 发（`ancient_station`）、时间碎片 8 | [L226](../../../raw/current/scripts/recipes.lua#L226)、[L363](../../../raw/current/scripts/recipes.lua#L363) |
| 碎片：地块与圣殿遗物 | 4 | 远古石刻／远古圣殿石刻各 1 出 4、仿制圣殿碗 1、仿制圣殿花瓶 1（均 `TECH.LOST`） | [L723-L745](../../../raw/current/scripts/recipes.lua#L723-L745) |
| 碎片：拆解 | 4 | 路标导航仪 1、戍卫塔核心 1–2、三号核心 1 | [L1684-L1687](../../../raw/current/scripts/recipes.lua#L1684-L1687) |

- **诅咒弹药（`slingshotammo_thulecite`）是"1 换 20"**：`numtogive=20`，再经 `calc_slingshotammo_numtogive` 在沃尔特 `walter_ammo_efficiency` 下变 30；`no_deconstruction=true`、`allowautopick=true`。[配方](../../../raw/current/scripts/recipes.lua#L226)、[加成](../../../raw/current/scripts/recipes.lua#L199-L204)
- **站点筛选与配方记忆是两件事（R12）**：诅咒弹药与铥矿弹弓框只写 `station_tag="ancient_station"`＋`force_hint`、**没有** `nounlock`，所以它们可被正常学会——在站旁首次解锁后离站仍能做，`station_tag` 仅把配方归入制作站的筛选。反过来，远古那批配方（L797-L810）标了 `nounlock`，`Builder:UnlockRecipe` 对它们直接跳过，普通解锁不会写入，只能在提供对应科技的站旁制作。[弹药](../../../raw/current/scripts/recipes.lua#L226)、[框](../../../raw/current/scripts/recipes.lua#L244)、[筛选语义](../../../raw/current/scripts/recipe.lua#L159)、[unlock 跳过](../../../raw/current/scripts/components/builder.lua#L463-L473)、[已学判定](../../../raw/current/scripts/components/builder.lua#L883-L907)

## 修复链

修复是"修复材料 `repairer` → 被修目标 `repairable`"两段：目标先比 `repairmaterial` 是否**完全相等**，再按目标自身有无 `health`／`workable` 选一个分支结算，最后消耗 1 个材料。[比对](../../../raw/current/scripts/components/repairable.lua#L148-L162)、[分支](../../../raw/current/scripts/components/repairable.lua#L106-L146)、[消耗](../../../raw/current/scripts/components/repairable.lua#L172-L176)

| 目标 | 走哪条分支 | 一块铥矿的结算 | 证据 |
|---|---|---|---|
| 碎掉的远古祭坛 `ancient_altar_broken` | `workable`（无 `health`）→ `SetWorkLeft(workleft + 1.5)` | prefab 自身默认 `SetWorkLeft(1)`／`SetMaxWork(9+1)`，即 1/10；1＋6×1.5＝10，恰好 **6 块**。玩家把完整祭坛锤碎时 `complete_onhammered` 生成破损件后把工作量改写成 9，再补 1 块即可（9＋1.5 ≥ 10） | [repairable 材质](../../../raw/current/scripts/prefabs/altar_prototyper.lua#L452-L454)、[broken_fn 的 1/10](../../../raw/current/scripts/prefabs/altar_prototyper.lua#L465-L470)、[锤落后的 9](../../../raw/current/scripts/prefabs/altar_prototyper.lua#L327-L337)、[修复完成回调](../../../raw/current/scripts/prefabs/altar_prototyper.lua#L406-L420) |
| 远古墙（`wall_ruins`／`wall_ruins_2`） | `health` → `DoDelta(100)` | 100 是**请求值**：`Health:SetVal` 会把结果截到上限，残血墙只补到满、不会溢出（满血时更早一步就 return false）。同材质墙物品 `wall_ruins_item` 自己的 `repairer` 是 `maxhealth / 6`＝800/6≈133.33，同样是请求值 | [墙的 repairable](../../../raw/current/scripts/prefabs/walls.lua#L321-L331)、[墙物品的 repairer](../../../raw/current/scripts/prefabs/walls.lua#L223-L225)、[血量截断](../../../raw/current/scripts/components/health.lua#L565-L574)、[墙血常量](../../../raw/current/scripts/tuning.lua#L2153) |
| 任意带 `repairable` 且材质为 `thulecite` 的目标 | 有 `health` 走血、有 `workable` 走工作量 | 碎片同流程，数值为 100/6 与 1.5/6 | [碎片 repairer](../../../raw/current/scripts/prefabs/thulecite_pieces.lua#L42-L45) |

- **满血时修复会失败而不是溢出**：`address_repair_amount` 先看 `health`，`GetPercent() >= 1` 直接 `return false`，不会继续尝试 `workable` 分支。目标同时有血与工作量时，血量满即无法用铥矿补工作量。[分支顺序](../../../raw/current/scripts/components/repairable.lua#L106-L124)
- **6 碎片≈1 铥矿**：因为碎片两项修复值都定义为铥矿的 1/6，6 个碎片的修复量刚好等于 1 块铥矿；这也是合成配方 6:1 的数感来源（推导）。[常量](../../../raw/current/scripts/tuning.lua#L2111-L2115)

## 食用、堆叠与标签

| 属性 | `thulecite` | `thulecite_pieces` | 消费者／证据 |
|---|---|---|---|
| 堆叠 | 20（默认 `STACK_SIZE_MEDITEM`） | 40（`STACK_SIZE_SMALLITEM`） | [常量](../../../raw/current/scripts/tuning.lua#L82-L83) |
| 食用 | `FOODTYPE.ELEMENTAL`、饥饿 3、回血 10（默认）、理智 0 | 同类型、饥饿 1、回血 10、理智 0 | [默认值](../../../raw/current/scripts/components/edible.lua#L19-L24) |
| 标签 | `molebait` | `molebait` | [铥矿](../../../raw/current/scripts/prefabs/thulecite.lua#L22)、[碎片](../../../raw/current/scripts/prefabs/thulecite_pieces.lua#L22) |
| 入水 | `SetSinks(true)` 沉底 | 同 | [沉没判定](../../../raw/current/scripts/components/inventoryitem.lua#L541-L546) |
| 交易 | 有 `tradable`、`goldvalue` 默认 0 | 同 | [默认值](../../../raw/current/scripts/components/tradable.lua#L1-L7)、[猪王门槛](../../../raw/current/scripts/prefabs/pigking.lua#L575) |

- **谁能吃元素类食物**：`mole`（鼹鼠）`SetDiet({FOODTYPE.ELEMENTAL})`、`slurtle`（蛞蝓龟）、`dustmoth`（尘蛾）、`rocky`（石虾）与 `rocky_boss` 都按 `ELEMENTAL` 设食性，石虾还额外用 `item.components.edible.foodtype == FOODTYPE.ELEMENTAL` 判断"能不能喂"。[鼹鼠](../../../raw/current/scripts/prefabs/mole.lua#L229)、[蛞蝓](../../../raw/current/scripts/prefabs/slurtle.lua#L150)、[石虾](../../../raw/current/scripts/prefabs/rocky.lua#L215-L220)
- **不能进堆肥桶**：`edible` 设了 `FOODTYPE.ELEMENTAL` 就会打上 `edible_ELEMENTAL` 标签；`componentactions` 生成 `ADDCOMPOSTABLE` 时排除该标签，堆肥桶自己的 `DONT_ACCEPT_FOODTYPES` 里也列了 `ELEMENTAL`。[标签生成](../../../raw/current/scripts/components/edible.lua#L9-L17)、[候选过滤](../../../raw/current/scripts/componentactions.lua#L1210-L1216)、[拒收表](../../../raw/current/scripts/prefabs/compostingbin.lua#L16-L22)
- **会被鼹鼠偷**：`molebrain` 的 `TAKEBAIT_MUST_TAGS = { "molebait" }`，用 `STEALMOLEBAIT` 动作捡走地上的铥矿；`target.components.inventoryitem:IsHeld()` 时不偷（拿在手里安全）。[行为](../../../raw/current/scripts/brains/molebrain.lua#L64-L81)
- **能当陷阱诱饵**：`trap` 的 `BAIT_TAGS` 是 `molebait` 加所有 `edible_<FOODTYPE>`；判定时对鼹鼠看 `molebait`，对其他目标看 `eater:CanEat(v)`。铥矿两个标签都满足。[诱饵标签](../../../raw/current/scripts/components/trap.lua#L264-L267)、[判定](../../../raw/current/scripts/components/trap.lua#L320-L331)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`repairer` | 提供 `repairmaterial`／`workrepairvalue`／`healthrepairvalue`，并自动打 `work_thulecite`／`health_thulecite` 标签 | 仅主模拟 | [[reference/components/repairer]]；[接线](../../../raw/current/scripts/prefabs/thulecite.lua#L30-L33) |
| 组件：`repairable` | 本物品不挂；它是**修复材料**，被修方才有 | 目标侧 | [[reference/components/repairable]]；[比对](../../../raw/current/scripts/components/repairable.lua#L148-L162) |
| 组件：`edible` | `FOODTYPE.ELEMENTAL`、饥饿 3／1；回血与理智取默认 | 仅主模拟 | [[reference/components/edible]]；[接线](../../../raw/current/scripts/prefabs/thulecite.lua#L35-L37) |
| 组件：`bait` | 地面物品可作陷阱诱饵；被拾取或移除时通知陷阱 | 仅主模拟 | [组件](../../../raw/current/scripts/components/bait.lua#L7-L36) |
| 组件：`inventoryitem`／`stackable`／`tradable` | 背包、堆叠 20／40、可交易（goldvalue 0） | 仅主模拟 | [[reference/components/inventoryitem]]、[[reference/components/stackable]] |
| 标签：`molebait` | 鼹鼠偷取与陷阱诱饵的共用标签 | 两注册都有 | [鼹鼠行为](../../../raw/current/scripts/brains/molebrain.lua#L64-L81) |
| 组件：`hauntable` | `MakeHauntableLaunchAndSmash`：只抛飞，砸碎分支已注释 | 仅主模拟 | [helper](../../../raw/current/scripts/standardcomponents.lua#L1071-L1104) |
| 数据：`recipes.lua` | 35 条配方／拆解记录，见[配方消费者](#配方消费者) | 按 `TECH` 分组 | [合成](../../../raw/current/scripts/recipes.lua#L796) |
| 关联 Prefab：`ancient_altar_broken`／`wall_ruins` | 两个主要修复目标 | 需对应 `repairable` | [祭坛](../../../raw/current/scripts/prefabs/altar_prototyper.lua#L443-L454)、[墙](../../../raw/current/scripts/prefabs/walls.lua#L315-L331)（Wiki 待建） |
| 资源：`anim/thulecite.zip`／`anim/thulecite_pieces.zip` | 各自 bank／build，播 `anim` | 原版资源，不随页分发 | [铥矿](../../../raw/current/scripts/prefabs/thulecite.lua#L1-L4)、[碎片](../../../raw/current/scripts/prefabs/thulecite_pieces.lua#L1-L4) |

## 典型接入

### 让自定义结构接受铥矿修复

- 前提与选择：被修对象必须有 `repairable`，且 `repairmaterial` 与修复材料的 `repairer.repairmaterial` **字符串相等**；铥矿系的正确值是 `MATERIALS.THULECITE`。原版 `walls.lua` 的 `ruins` 墙与 `altar_prototyper.lua` 的破损祭坛都是这个写法。[墙](../../../raw/current/scripts/prefabs/walls.lua#L321-L331)、[祭坛](../../../raw/current/scripts/prefabs/altar_prototyper.lua#L452-L454)
- 生效与撤销：只在主模拟段加组件；`onrepaired` 在材料被消耗**之后**触发，可在此换实体或播音效，但不要假设材料还在。
- 原版依据：分支顺序与消耗点见[修复链](#修复链)。

```lua
-- 放在被修对象的 prefab 文件里；fn() 内须在 TheWorld.ismastersim 判断之后调用 MakeRepairable(inst)
local function OnRepaired(inst, doer, repair_item)
    -- 材料已在此前被消耗；这里做换实体、播音效一类收尾
end

local function MakeRepairable(inst)
    inst:AddComponent("repairable")
    inst.components.repairable.repairmaterial = MATERIALS.THULECITE
    inst.components.repairable.onrepaired = OnRepaired
    return inst
end

return MakeRepairable
```

- 调整与验证：`MakeRepairable` 只在服务端执行（客户端段提前 `return inst`）；目标若有 `health`，一次修复只加 `repairer.healthrepairvalue`，满血时调用直接失败；没有任何回调能改这个顺序。验证时先造一个半血目标再用手上的铥矿试一次即可。未实测。

## 常见陷阱与限制

- **误区：`thulecite` 与 `thulecite_pieces` 可以互相顶替。** 修复值差 6 倍，堆叠差 2 倍，配方也分开：`thulecite` 的 23 条与碎片的 12 条互不通用（`Ingredient` 名写死）。[配方总表](#配方消费者)
- **误区：`MakeHauntableLaunchAndSmash` 会砸碎铥矿。** 该 helper 的 smash 分支整段被注释（`#HAUNTFIX`），实际只有 `launch_chance` 的抛飞。[helper](../../../raw/current/scripts/standardcomponents.lua#L1071-L1104)
- **边界：满血目标不能"顺手修一下工作量"。** `health` 分支优先且满血即返回失败，不会落到 `workable`。[分支](../../../raw/current/scripts/components/repairable.lua#L106-L124)
- **边界：祭坛的两个初值不同。** 世界生成／重载的破损祭坛 `workleft = 1`（需 6 块铥矿），被敲碎的祭坛 `workleft = 9`（再补 1 块），差值来自 `complete_onhammered` 的 `SetWorkLeft(TUNING.ANCIENT_ALTAR_BROKEN_WORK)`。[两处](../../../raw/current/scripts/prefabs/altar_prototyper.lua#L327-L337)、[broken 段](../../../raw/current/scripts/prefabs/altar_prototyper.lua#L465-L467)
- **边界：铥矿不是猪王的货币，也不是蚁狮的贡品。** `tradable` 有但 `goldvalue` 默认 0：猪王收不收看它的金值门槛，蚁狮的 `AcceptTest` 看的是 `tradable.rocktribute > 0`（铥矿系两者都没有），`goldvalue` 只在决定回礼时用到。[猪王](../../../raw/current/scripts/prefabs/pigking.lua#L575)、[蚁狮验收](../../../raw/current/scripts/prefabs/antlion.lua#L115-L119)、[回礼](../../../raw/current/scripts/prefabs/antlion.lua#L138)
- **未核验**：未运行游戏。掉落期望值按 `Σ chance` 推导；"6 块修好破损祭坛"按 `workable` 的初值与 1.5/次推导，未实测修复动画与打断；`slurtle`／`dustmoth` 食性只核到 `SetDiet` 行，未追其取食动作的完整链路。

## 验证记录

- 源码审阅：2026-11-10，Steam buildid 26474873。通读 `prefabs/thulecite.lua`（54 行）与 `prefabs/thulecite_pieces.lua`（55 行）；`components/repairer.lua`、`repairable.lua`、`edible.lua`（默认值段与 `foodtype` setter）、`stackable.lua`（默认 maxsize）、`bait.lua`、`tradable.lua`、`lootdropper.lua`（`AddChanceLoot`／`GenerateLoot`）、`trap.lua` 的诱饵判定、`standardcomponents.lua` 的 `MakeHauntableLaunchAndSmash`；`prefabs/walls.lua` 的墙体数据、物品 `repairer` 与已放置 `repairable`；`prefabs/altar_prototyper.lua` 的 `broken_fn`／`complete_onhammered`／`broken_onrepaired`；`prefabs/atrium_statue.lua`、`minotaur.lua`（含宝箱抽取）、`vault_pillar_guard.lua`、`statueruins.lua`、`smashables.lua`、`chessjunk.lua`、`compostingbin.lua`、`pigking.lua`；`recipes.lua` 全部 35 条相关行；`tuning.lua` 堆叠与四个修复常量、`RUINSWALL_HEALTH`；`componentactions.lua` 的堆肥桶过滤；`scrapbookdata.lua`、`chinese_s.po` 名称键。
- 未覆盖：`tree_rock_data.lua` 的石化树矿脉权重（只作定位，不引用）；`cave_entrance.lua` 的铥矿掉落语境；远古科技台之外的其它 `prototyper` 交互。
- 游戏运行：未运行；涉及天数／秒数不适用（无计时器），血量与工作量为绝对值。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 见交付回报。

## 相关链接

- 远古装备与设施：[[reference/prefabs/armor-classic]]、[[reference/prefabs/ruins-bat]]、[[reference/prefabs/multitool_axe_pickaxe]]、[[reference/prefabs/basic-mineral-materials]]
- 弹药与工具：[[reference/prefabs/ranged-weapons]]、[[reference/prefabs/tools-misc]]
- 组件：[[reference/components/repairer]]、[[reference/components/repairable]]、[[reference/components/edible]]、[[reference/components/inventoryitem]]
- 同批材料：[[reference/prefabs/purebrilliance]]、[[reference/prefabs/lunarplant-husk]]

返回 [[reference/prefabs/index|Prefab 索引]]。
