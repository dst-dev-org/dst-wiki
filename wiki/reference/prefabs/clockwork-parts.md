---
title: 齿轮与电子元件（gears、transistor）
created: "2026-11-10"
updated: "2026-11-10"
type: prefab
summary: 两种独立注册的机械零件：齿轮可堆叠 40、是 WX-78 专属食物（GEARS 食性 60/75/50）、可修发条废料与废料墙、还能喂给发条生物建立友好；电子元件可堆叠 10、不可食不可燃，主要作制造材料，也能当神龛供品交存。本页对照两者构造、掉落来源与三类消费（配方扣料、神龛供品交存并保留实体、装饰；两者都不参与猪王交易），并写清修理与友好化的实际门禁。
aliases: ["gears", "transistor", "齿轮", "电子元件", "Gears", "Electrical Doodad", "发条零件", "机械零件"]
tags: [prefab, item]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/gears.lua", "prefabs/transistor.lua", "prefabs/clockwork_common.lua", "prefabs/alterguardian_phase4_lunarrift.lua", "prefabs/rubble.lua", "prefabs/smashables.lua", "prefabs/wx78_possessedbody.lua", "stategraphs/SGwilson.lua", "prefabs/knight.lua", "prefabs/bishop.lua", "prefabs/rook.lua", "prefabs/chessjunk.lua", "prefabs/junk_pile_common.lua", "prefabs/mound.lua", "prefabs/minotaur.lua", "prefabs/daywalker2.lua", "prefabs/eyeofterror.lua", "prefabs/wagboss_robot.lua", "prefabs/wagdrone_common.lua", "prefabs/tumbleweed.lua", "prefabs/monkeyisland_portal_debris.lua", "prefabs/catcoon.lua", "prefabs/inspectaclesbox.lua", "prefabs/walls.lua", "prefabs/wx78.lua", "prefabs/yoth_knightshrine.lua", "prefabs/altar_prototyper.lua", "components/repairer.lua", "components/repairable.lua", "components/useabletargeteditem.lua", "components/clockworktracker.lua", "components/upgrademoduleowner.lua", "components/edible.lua", "components/eater.lua", "components/snowmandecoratable.lua", "components/stackable.lua", "standardcomponents.lua", "componentactions.lua", "constants.lua", "tuning.lua", "recipes.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 齿轮与电子元件（gears、transistor）

## 快速判断

- 职责与适用：两个各自独立文件、各自 `Prefab()` 注册的物品——齿轮（`gears`）与电子元件（`transistor`），没有共享工厂。齿轮是发条系零件兼 WX-78 的食物与修理材料；电子元件是不可食、不可燃的纯制造材料。可借鉴：同族物品"可食但只有特定角色吃得到"的双层门禁、`repairer`／`repairable` 的四类回复通道（health／work／perish／finiteuses）、以及 `useabletargeteditem` 的"对目标使用一次"接线。[gears 注册](../../../raw/current/scripts/prefabs/gears.lua#L102)、[transistor 注册](../../../raw/current/scripts/prefabs/transistor.lua#L38)
- 优先使用：调堆叠／可食／修理数值查[数值与参数](#数值与参数)；让别的角色也能吃齿轮查[典型接入](#典型接入)；修理目标与友好化的判定见[关联机制导航](#关联机制导航)。
- 重要副作用：齿轮的 `edible` 只是"登记可食"，实际入口还要求角色带 `GEARS_eater` 标签（只有 WX-78 与其附身机体用 `SetDiet` 加）。`useabletargeteditem` **预检**失败不消耗；但"目标已记住该 leader 且跟随上限已满"之外的路径即使加不进 tracker，也会照常回满血、唤醒并扣 1 个（同时推 `ms_maxclockworks`，目标不跟随）。电子元件挂 `MakeHauntableLaunch`，作祟只会把它弹飞。
- 修改边界：本页不覆盖发条生物的 Brain／SG 主体行为（只写 `TryBefriendChess` 的门禁与效果）；不覆盖 `chessjunk` 拆解后的发条生成流程。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `gears` | 单文件单注册 `Prefab("gears", fn, assets)`。公共端 bank／build `gears`、`pickupsound="metal"`、`molebait`、`MakeInventoryFloatable("med", nil, 0.7)`，并在 `SetPristine` 前把 `UseableTargetedItem_ValidTarget` 挂成实体方法（客户端也要用）。 | 主模拟组件：`stackable`(40)、`inspectable`、`inventoryitem`、`bait`、`edible`(GEARS)、`repairer`(GEARS，work 1／health 75)、`useabletargeteditem`(`OnUsedOnChess`)、`snowmandecor`、`MakeHauntableLaunchAndSmash`。[组件段](../../../raw/current/scripts/prefabs/gears.lua#L72-L97)、[客户端方法](../../../raw/current/scripts/prefabs/gears.lua#L64) |
| `transistor` | 单文件单注册 `Prefab("transistor", fn, assets)`。公共端 bank／build `transistor`＋`MakeInventoryFloatable(inst)`（无参→small 档、无偏移缩放），`SetPristine` 后主模拟。 | 组件仅 `stackable`(`STACK_SIZE_LARGEITEM`＝10)、`inspectable`、`inventoryitem`、`MakeHauntableLaunch`；无 `edible`／`fuel`／`tradable`／`bait`／`snowmandecor`／`repairer`。[组件段](../../../raw/current/scripts/prefabs/transistor.lua#L27-L33) |

### 名称与实显

简中 `.po` 键：`NAMES.GEARS`＝齿轮（Gears）、`NAMES.TRANSISTOR`＝电子元件（英文名是 `Electrical Doodad`，不是"晶体管"直译）。[GEARS](../../../raw/current/scripts/languages/chinese_s.po#L343140-L343142)、[TRANSISTOR](../../../raw/current/scripts/languages/chinese_s.po#L350525-L350527)

## 数值与参数

tuning 基准：`HEALING_HUGE`＝60、`CALORIES_HUGE`＝`calories_per_day`＝75、`SANITY_HUGE`＝50、`REPAIR_GEARS_WORK`＝1、`REPAIR_GEARS_HEALTH`＝75；堆叠 `SMALLITEM`＝40／`LARGEITEM`＝10。[可食值](../../../raw/current/scripts/tuning.lua#L2028-L2063)、[修理值](../../../raw/current/scripts/tuning.lua#L2107-L2108)、[堆叠](../../../raw/current/scripts/tuning.lua#L81-L83)

| 注册名 | 堆叠 | 可食（foodtype） | 治疗／饥饿／理智 | 修理材料／回复量 | 交易 | 雪人装饰 | 作祟效果 |
|---|---|---|---|---|---|---|---|
| `gears` | 40 | GEARS | 60／75／50 | `gears`：work 1、health 75（无 finiteuses／perish 值） | 无 `tradable` | 有表项，无 `canflip` | 仅弹飞 |
| `transistor` | 10 | 无 | — | 无 | 无 `tradable` | 无表项 | 仅弹飞 |

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| `组件：edible`＝GEARS | 三维直接取 tuning 的大值；`foodtype` 决定物品标签 `edible_GEARS` | 主模拟挂载；食性与数值见上表 | [[reference/components/edible#关键接口]]；[gears edible](../../../raw/current/scripts/prefabs/gears.lua#L81-L85) |
| `API／数据：eater 食性门禁` | 玩家要吃必须同时满足"物品带 `edible_GEARS`"与"角色带 `GEARS_eater`"；默认 `FOODGROUP.OMNI` 的六类不含 GEARS。本版只有两处 `SetDiet({FOODGROUP.OMNI, FOODTYPE.GEARS}, …)`：WX-78 本体与其附身机体 | 服务端加标签、客户端读标签生成 `EAT` 动作；`Eater:SetDiet` 经 `oncaneat` 增删 `<TYPE>_eater` 标签；`SetCanEatGears` 已标废弃且全 scripts 无调用 | [[reference/components/eater#关键接口]]；[默认 OMNI](../../../raw/current/scripts/constants.lua#L2049-L2062)、[WX 食性](../../../raw/current/scripts/prefabs/wx78.lua#L753-L757)、[附身机体](../../../raw/current/scripts/prefabs/wx78_possessedbody.lua#L601-L603)、[EAT 门禁](../../../raw/current/scripts/componentactions.lua#L2683-L2687)、[废弃接口](../../../raw/current/scripts/components/eater.lua#L349-L357) |
| `组件：repairer` | `repairmaterial=MATERIALS.GEARS`（值是**小写** `"gears"`）、`workrepairvalue=1`、`healthrepairvalue=75`；组件按值拼 `work_<材料>`／`health_<材料>`，即 `work_gears`／`health_gears` | 主模拟；`REPAIR` 是右键动作，目标要带 `repairable_gears` 与 `workrepairable`／`healthrepairable` 才出动作 | [[reference/components/repairer#关键接口]]；[材料值](../../../raw/current/scripts/constants.lua#L1940-L1947)、[挂载](../../../raw/current/scripts/prefabs/gears.lua#L87-L90)、[标签生成](../../../raw/current/scripts/components/repairer.lua#L8-L24)、[动作门禁](../../../raw/current/scripts/componentactions.lua#L1688-L1695) |
| `组件：repairable` | 判定目标侧：材料必须相同；回复按 `health → work → perish → finiteuses` 顺序取第一个可用分支 | `health` 已满时直接返回 false（不再试 work）；`work` 分支要求 `workable.workable` 且 `workleft < maxwork` | [[reference/components/repairable#关键接口]]；[回复分流](../../../raw/current/scripts/components/repairable.lua#L106-L146)、[消耗物品](../../../raw/current/scripts/components/repairable.lua#L148-L183) |
| `组件：useabletargeteditem` | `SetOnUseFn(OnUsedOnChess)`：对带 `TryBefriendChess` 的目标使用，成功则回满血、唤醒、扣 1 个齿轮，并用 `DoStaticTaskInTime(0, ResetInUse)` 解锁。`TryBefriendChess` 在前置条件不满足（目标已死／已有 leader／施与者非法或幽灵／缺 `leader` 组件／在小游戏）、或"已记住该 leader 且 tracker 加不进"时返回 false；未记住且已达上限仍返回 true（扣 1 并推 `ms_maxclockworks`，目标不跟随） | 客户端可用实体方法 `UseableTargetedItem_ValidTarget` 预判；`USEITEMON` 动作在目标附近生成 | [[reference/components/useabletargeteditem#关键接口]]；[使用回调](../../../raw/current/scripts/prefabs/gears.lua#L18-L34)、[有效目标](../../../raw/current/scripts/prefabs/gears.lua#L36-L42)、[动作门禁](../../../raw/current/scripts/componentactions.lua#L1900-L1914) |
| `API／数据：clockwork_common.TryBefriendChess` | 目标侧实装：要求未死、无现任 leader、施与者存活且非幽灵、有 `leader` 组件、不在小游戏；再由 `clockworktracker` 判上限 | 成功条件还包含"已记住该 leader 时不得超上限"；超限则清 leader 并推 `ms_maxclockworks` | Wiki 待建；[TryBefriendChess](../../../raw/current/scripts/prefabs/clockwork_common.lua#L231-L269)、[上限判定](../../../raw/current/scripts/components/clockworktracker.lua#L87-L105) |
| `API／数据：CanDoerBefriendTarget` | 两条放行条件：目标带 `befriendable_clockwork`，或施与者是带 `chess` 模块的 WX-78 且目标带 `chess` 标签 | `befriendable_clockwork` 只由发条三兄弟 SG 的 stunned／shock 状态加（`sgTrySetBefriendable`）；`GetModuleTypeCount("chess")` 只数仍在充能范围内的 `wx78module_chess` | Wiki 待建；[判定](../../../raw/current/scripts/prefabs/gears.lua#L10-L16)、[状态加标签](../../../raw/current/scripts/prefabs/clockwork_common.lua#L279-L288)、[模块计数](../../../raw/current/scripts/components/upgrademoduleowner.lua#L114-L127) |
| `组件：stackable`／`组件：bait`／`标签 molebait` | 齿轮堆叠 40，并带 `bait` 组件与 `molebait` 标签（可当陷阱诱饵／被鼹鼠偷）；电子元件堆叠 10，`bait` 组件与 `molebait` 标签都没有 | 主模拟 | [[reference/components/stackable#关键接口]]、[[reference/components/bait#关键接口]]；[gears 标签](../../../raw/current/scripts/prefabs/gears.lua#L60)、[transistor 组件段](../../../raw/current/scripts/prefabs/transistor.lua#L27-L33) |
| `组件：snowmandecor` | 仅齿轮有 `snowmandecor` 组件与 `ITEM_DATA` 表项（无 `canflip`）；电子元件既无组件也无表项 | 主模拟／表项按注册名 hash | Wiki 待建；[挂载](../../../raw/current/scripts/prefabs/gears.lua#L95)、[表项](../../../raw/current/scripts/components/snowmandecoratable.lua#L24) |
| `组件：hauntable` | 齿轮 `MakeHauntableLaunchAndSmash`（砸碎段被 `#HAUNTFIX` 整段注释，只剩 Launch）；电子元件 `MakeHauntableLaunch`（本就只有 Launch） | 两者实际都只弹飞，并把 `inventoryitem.is_landed` 置 false 重新落地；电子元件没有砸碎或点燃分支 | [[reference/components/hauntable#按效果选工厂]]；[LaunchAndSmash](../../../raw/current/scripts/standardcomponents.lua#L1071-L1105)、[Launch](../../../raw/current/scripts/standardcomponents.lua#L1053-L1069) |
| `关联 Prefab：chessjunk`／`wall_scrap` | 齿轮的可修目标：发条废料堆（`repairable_gears`，work 类）与废料墙（`repairable_gears`，有 health 也有 workable） | 服务器 | [chessjunk](../../../raw/current/scripts/prefabs/chessjunk.lua#L211-L213)、[walls 数据](../../../raw/current/scripts/prefabs/walls.lua#L324-L327)、[废料墙数据](../../../raw/current/scripts/prefabs/walls.lua#L418-L429) |
| `关联 Prefab：yoth_knightshrine` | 齿轮／电子元件都可当供品放进骑士神龛：收下后**保留实体**、移出场景，作为原型器开启条件（不扣料）。换供品时把上一件送回场景；清空／点燃／烧毁时才删除（点燃或烧毁时若仍持供品，掉 1 个灰烬；同一件只结算一次） | 服务器；`able_to_accept_test` 只认齿轮、`trinket_6`、电子元件；`trader.deleteitemonaccept=false` | [收取配置](../../../raw/current/scripts/prefabs/yoth_knightshrine.lua#L183-L189)、[交存与退回](../../../raw/current/scripts/prefabs/yoth_knightshrine.lua#L90-L129)、[清空分支](../../../raw/current/scripts/prefabs/yoth_knightshrine.lua#L156-L170)、[烧毁删除](../../../raw/current/scripts/prefabs/yoth_knightshrine.lua#L193-L205) |

## 获得链

**齿轮**——击杀发条三兄弟是稳定来源，其余多为概率或拆解：

| 来源 | 产出 | 触发／条件 | 证据 |
|---|---|---|---|
| 发条骑士／主教／车 | 普通与鎏金变体 2 个；噩梦变体里骑士／车各 1 个，噩梦主教**不掉**齿轮 | 共享掉落表逐条按概率 1.0 入列；主教另掉紫宝石，鎏金骑士另掉马掌／红袋 | [knight](../../../raw/current/scripts/prefabs/knight.lua#L59-L63)、[knight_nightmare](../../../raw/current/scripts/prefabs/knight.lua#L65-L70)、[knight_gilded](../../../raw/current/scripts/prefabs/knight.lua#L72-L80)、[bishop](../../../raw/current/scripts/prefabs/bishop.lua#L41-L46)、[bishop_nightmare](../../../raw/current/scripts/prefabs/bishop.lua#L48-L53)、[rook](../../../raw/current/scripts/prefabs/rook.lua#L37-L41)、[rook_nightmare](../../../raw/current/scripts/prefabs/rook.lua#L43-L48) |
| 双恐怖之眼（`twinofterror1`／`twinofterror2`，两个独立注册、各自挂同名表） | 每只 3–5 个；两只都击杀合计 6–10 个 | 每只表内 3 条 1.0＋2 条 0.5，逐条独立判定；单眼 `eyeofterror` 的表没有齿轮 | [注册](../../../raw/current/scripts/prefabs/eyeofterror.lua#L984-L987)、[twin1 表](../../../raw/current/scripts/prefabs/eyeofterror.lua#L543-L564)、[twin2 表](../../../raw/current/scripts/prefabs/eyeofterror.lua#L615-L634)、[死亡结算](../../../raw/current/scripts/stategraphs/SGeyeofterror.lua#L967-L977) |
| 天体后裔（`alterguardian_phase4_lunarrift`） | 1–2 个 | 共享掉落表：1 条 1.0＋1 条 0.5 | [掉落表](../../../raw/current/scripts/prefabs/alterguardian_phase4_lunarrift.lua#L65-L66) |
| 废料机器人（`wagboss_robot`） | 0.5 概率 1 个 | 共享掉落表 | [掉落表](../../../raw/current/scripts/prefabs/wagboss_robot.lua#L25-L31) |
| 废料无人机（`wagdrone_common`） | 1 个＋0.333 再 1 个 | 被拆（`OnWorked`）时按表掉落 | [掉落表](../../../raw/current/scripts/prefabs/wagdrone_common.lua#L125-L133) |
| 发条废料堆（`chessjunk`） | 修好后 10% 掉 1 个 | `OnPlayerRepaired` 里 `AddChanceLoot("gears", .1)` 后 `DropLoot` | [修好回调](../../../raw/current/scripts/prefabs/chessjunk.lua#L80-L87) |
| 垃圾堆（`junk_pile_common`） | 约 3.125%（权重 1／合计 32） | 挖掘时按权重抽一件 | [权重表](../../../raw/current/scripts/prefabs/junk_pile_common.lua#L14-L25) |
| 墓碑堆（`mound`） | 约 3.8% | 50% 先抽小玩具，否则在 5 项权重表里抽，齿轮权重 1／合计 13 | [挖开回调](../../../raw/current/scripts/prefabs/mound.lua#L70-L82)、[权重表](../../../raw/current/scripts/prefabs/mound.lua#L33-L40) |
| 远古守护者宝箱（`minotaur` 生成 `minotaurchest`） | 3–6 个 | 开箱战利品表条目，非击杀直接掉落 | [chest_loot](../../../raw/current/scripts/prefabs/minotaur.lua#L64-L75)、[装箱](../../../raw/current/scripts/prefabs/minotaur.lua#L613-L632) |
| 拾荒者（`daywalker2`） | 0.5 概率 1 个 | 共享掉落表 | [掉落表](../../../raw/current/scripts/prefabs/daywalker2.lua#L46-L48) |
| 滚草（`tumbleweed`） | 权重 1，占每次抽取的约 1%，整株抽 3 次 | `MakeLoot` 按权重抽 3 次（可重复） | [权重表](../../../raw/current/scripts/prefabs/tumbleweed.lua#L175-L190)、[抽取](../../../raw/current/scripts/prefabs/tumbleweed.lua#L213-L245) |
| 猴岛传送门残骸（`monkeyisland_portal_debris`） | 0.25 概率 1 个 | 锤碎时 `DropLoot` | [掉落表](../../../raw/current/scripts/prefabs/monkeyisland_portal_debris.lua#L14-L19) |
| 废墟／可砸物杂项（`rubble`、`smashables`） | 很低（随机掉落表里 `gears` 权重 0.01） | `rubble` 走 `numrandomloot=1` 的权重池；`smashables` 还要先过 1% 的 `chancerandomloot` | [rubble](../../../raw/current/scripts/prefabs/rubble.lua#L62-L71)、[smashables](../../../raw/current/scripts/prefabs/smashables.lua#L283-L290) |
| 骑士神龛（`yoth_knightshrine`） | 交存：供品被收下并保留实体（移出场景、不扣料），开启原型器；换供品时上一件送回场景，清空／烧毁才删除 | 供品判定；`deleteitemonaccept=false` | [供品判定](../../../raw/current/scripts/prefabs/yoth_knightshrine.lua#L145-L154)、[交存](../../../raw/current/scripts/prefabs/yoth_knightshrine.lua#L108-L129)、[退回](../../../raw/current/scripts/prefabs/yoth_knightshrine.lua#L90-L106) |

**电子元件**——有保底来源：双恐怖之眼（`twinofterror1`／`twinofterror2`，每只至少 2 个）与检查眼镜补给箱 2（`inspectaclesbox2`，至少 1 个）；其余靠拆发条／废料与概率：

| 来源 | 产出 | 触发／条件 | 证据 |
|---|---|---|---|
| 双恐怖之眼（`twinofterror1`／`twinofterror2`） | 每只 2–3 个（**至少 2 个必掉**）；两只合计 4–6 个 | 两只各自 `SetChanceLootTable(data.chanceloottable)` 挂同名表：2 条 1.0 必得＋1 条 0.75 | [挂表](../../../raw/current/scripts/prefabs/eyeofterror.lua#L386-L387)、[注册](../../../raw/current/scripts/prefabs/eyeofterror.lua#L984-L987)、[twin1 表](../../../raw/current/scripts/prefabs/eyeofterror.lua#L547-L558)、[twin2 表](../../../raw/current/scripts/prefabs/eyeofterror.lua#L617-L628)、[概率结算](../../../raw/current/scripts/components/lootdropper.lua#L213-L224) |
| 废料机器人（`wagboss_robot`） | 0.5 概率 1 个 | 共享掉落表 | [掉落表](../../../raw/current/scripts/prefabs/wagboss_robot.lua#L25-L31) |
| 废料无人机（`wagdrone_common`） | 0.667 概率 1 个 | 被拆时按表掉落 | [掉落表](../../../raw/current/scripts/prefabs/wagdrone_common.lua#L125-L133) |
| 检查眼镜补给箱 1（`inspectaclesbox`） | 0–2 个（75%＋50%，可能一个都不出） | 共享表 `inspectaclesbox` | [表](../../../raw/current/scripts/prefabs/inspectaclesbox.lua#L12-L20) |
| 检查眼镜补给箱 2（`inspectaclesbox2`） | 1–2 个（100%＋75%，至少 1 个） | 共享表 `inspectaclesbox2` | [表](../../../raw/current/scripts/prefabs/inspectaclesbox.lua#L21-L28) |
| 垃圾堆（`junk_pile_common`） | 约 3.125% | 与齿轮同表 | [权重表](../../../raw/current/scripts/prefabs/junk_pile_common.lua#L14-L25) |
| 浣猫友善礼物（`catcoon`） | 1 个 | 友善时按 tier 抽，tier 7 含 `transistor` | [table](../../../raw/current/scripts/prefabs/catcoon.lua#L182-L188)、[抽取](../../../raw/current/scripts/prefabs/catcoon.lua#L265-L273) |
| 骑士神龛（`yoth_knightshrine`） | 交存（保留实体、不扣料） | 供品判定；换供品送回上一件，清空／烧毁删除 | [供品判定](../../../raw/current/scripts/prefabs/yoth_knightshrine.lua#L145-L154)、[交存](../../../raw/current/scripts/prefabs/yoth_knightshrine.lua#L108-L129) |

## 用途与配方

- **齿轮**（`recipes.lua`）：冰箱 icebox×1、灭火器 firesuppressor×2、冰包 icepack×1、龙卷风法杖 staff_tornado×1、船转向套件 boat_rotator_kit×1、留声机 phonograph×1、WX 模块（movespeed2×1、chess×1）、WX 备用机体 wx78_backupbody×1；恐惧护盾 shieldofterror 与船转向器拆解也回收齿轮。[icebox](../../../raw/current/scripts/recipes.lua#L563)、[firesuppressor](../../../raw/current/scripts/recipes.lua#L569)、[icepack](../../../raw/current/scripts/recipes.lua#L518)、[staff_tornado](../../../raw/current/scripts/recipes.lua#L490)、[wx78module_chess](../../../raw/current/scripts/recipes.lua#L777)、[拆解回收](../../../raw/current/scripts/recipes.lua#L1679)
- **电子元件**（主要入口，非全表）：研究院 2（researchlab2×2）、冰包×1、灭火器×2、冷火坑 coldfirepit×2、鼹鼠帽 molehat×2、植物登记帽 plantregistryhat×1、暗夜灯 nightstick×2、书本站 book_research_station×1、钓具台 tacklestation×1、薇诺娜系列（电池×2、遥控器×1、储物机器人×4、传送伞×4、传送板×6、检查眼镜×2）、WX 扫描器与三种无人机、暗影收割／削弱无人机各 1、沃利便携搅拌机×2、月装置一期×2、船磁铁套件×1、留声机×2、无线电台 w_radio×2、万圣实验台 madscience_lab×2、弹弓废料框×1、废料帽／甲各×3、废料零件包×1。[portableblender](../../../raw/current/scripts/recipes.lua#L98)、[researchlab2](../../../raw/current/scripts/recipes.lua#L383)、[winona_teleport_pad](../../../raw/current/scripts/recipes.lua#L321)、[暗影无人机](../../../raw/current/scripts/recipes.lua#L790-L791)、[废料帽／甲／零件包](../../../raw/current/scripts/recipes.lua#L861-L863)、[弹弓废料框](../../../raw/current/scripts/recipes.lua#L238)
- **供品（神龛）**：与配方扣料不同，供品是"交存"——放进神龛后实体保留、移出场景并开启原型器；换供品时上一件送回场景，清空／点燃／烧毁才删除。出灰按回调分别看：`OnIgnite` 时若仍持供品，先掉 1 个灰烬、再由 `MakeEmpty` 删除；`OnBurnt` 只在届时仍持供品时掉 1 个灰烬——正常燃烧已先经点燃清空，同一件供品不会二次出灰。[交存](../../../raw/current/scripts/prefabs/yoth_knightshrine.lua#L108-L129)、[清空](../../../raw/current/scripts/prefabs/yoth_knightshrine.lua#L156-L170)、[点燃](../../../raw/current/scripts/prefabs/yoth_knightshrine.lua#L207-L214)、[烧毁](../../../raw/current/scripts/prefabs/yoth_knightshrine.lua#L193-L205)
- **交易与装饰**：两者都没有 `tradable` 组件，不进入猪王交易；只有齿轮能在雪人上装饰。[雪人表项](../../../raw/current/scripts/components/snowmandecoratable.lua#L24)

## 典型接入

### 让自定义角色也能吃齿轮

- 前提与选择：服务端 `modmain.lua`；用 `SetDiet` 在既有食性上追加 `FOODTYPE.GEARS`，不要只加 `edible` 或直接加标签——`EAT` 动作与 `PrefersToEat` 都走本人 `caneat`／`preferseating`。
- 生效与撤销：`AddPlayerPostInit` 在每个玩家实体构造时执行（含读档）；移除模组即恢复原食性。同一处若同时设置 `preferseating`，两参数都要含 GEARS。
- 原版依据：WX-78 即用同一写法追加 GEARS；`SetDiet` 会经 `oncaneat` 维护 `GEARS_eater` 标签。[WX 写法](../../../raw/current/scripts/prefabs/wx78.lua#L753-L757)、[SetDiet](../../../raw/current/scripts/components/eater.lua#L45-L48)

```lua
-- modmain.lua：给某角色开放齿轮（保留原 OMNI 食性）
AddPlayerPostInit(function(inst)
    if GLOBAL.TheWorld.ismastersim and inst.prefab == "my_character"
        and inst.components.eater ~= nil then
        inst.components.eater:SetDiet(
            { GLOBAL.FOODGROUP.OMNI, GLOBAL.FOODTYPE.GEARS },
            { GLOBAL.FOODGROUP.OMNI, GLOBAL.FOODTYPE.GEARS })
    end
end)
```

- 调整与验证：食性只改"能不能吃"，三维收益仍取 `gears` 的 `edible`；验证时看右键菜单是否出现 `EAT`（客户端读 `GEARS_eater` 标签），再看饥饿／理智实际变化。本例为源码推导，未运行游戏。

## 常见陷阱与限制

- **边界：齿轮的修理回复量分两条，不能混写**——同一份齿轮对"只有 workable"的目标按 `workrepairvalue`=1 回复，对有 health 的目标先走 health 分支回复 75；`repairable` 的顺序是 health→work→perish→finiteuses，所以对同时有 health 和 workable 的目标（如废料墙），只要 health 未满就永远不会走 work 分支，health 满了则整次修理被拒（即使 workleft 有损）。[回复分流](../../../raw/current/scripts/components/repairable.lua#L106-L146)
- **边界：`REPAIR` 是右键动作**——`componentactions` 里 `repairer` 分支先判 `right`，再逐材料比对 `repairable_<材料>` 与 `work_/health_` 标签；左键点击不会出修理。[动作门禁](../../../raw/current/scripts/componentactions.lua#L1679-L1698)
- **误区：喂齿轮"失败一律不消耗"**——分两层：`UseableTargetedItem_ValidTarget`／`CanDoerBefriendTarget` 预检不过时不出动作、不消耗；进入 `TryBefriendChess` 后，前置条件不满足（`clockwork_common.lua#L232-L237`：目标已死／已有 leader／施与者非法或幽灵／缺 `leader` 组件／在小游戏）会落到 #L268 的 `return false`（不消耗）；此外只有"已记住该 leader 且 `clockworktracker` 加不进"会提前 `return false`。未记住的路径即使 `CanAddClockwork` 为假，也会 `SetLeader(nil)`＋推 `ms_maxclockworks`，随后仍 `return true` → 齿轮照常回满目标血、唤醒、扣 1 个。[使用回调](../../../raw/current/scripts/prefabs/gears.lua#L18-L34)、[TryBefriendChess](../../../raw/current/scripts/prefabs/clockwork_common.lua#L231-L269)
- **边界：成功使用后要手动解锁，最后一个要判空**——`StartUsingItem` 成功后 `inuse_targeted=true`，齿轮靠 `DoStaticTaskInTime(0, ResetInUse)` 调 `StopUsingItem` 解锁。`inst` 是被使用的齿轮本体而非 target：`stackable:Get()` 在只剩 1 个时返回 `self.inst` 本身，`Remove()` 后 `inst:IsValid()` 为假，于是跳过解锁（避免对已移除实体调用）。[解锁段](../../../raw/current/scripts/prefabs/gears.lua#L26-L30)、[Get 与末枚分支](../../../raw/current/scripts/components/stackable.lua#L109-L153)
- **误区：齿轮作祟会砸碎**——`MakeHauntableLaunchAndSmash` 的砸碎段整段被 `#HAUNTFIX` 注释，实际只弹飞；电子元件用的是 `MakeHauntableLaunch`，本就没有砸碎或点燃分支。[LaunchAndSmash](../../../raw/current/scripts/standardcomponents.lua#L1071-L1105)、[Launch](../../../raw/current/scripts/standardcomponents.lua#L1053-L1069)
- **边界：`GEARS_eater` 标签不是常量**——它由 `Eater:SetDiet` 的 `oncaneat` 动态增删；直接 `AddTag("GEARS_eater")` 只骗过 `EAT` 动作判定，不改 `preferseating`／`caneat`，实吃时 `SGwilson` 的 EAT 处理仍用 `PrefersToEat` 拒绝。用 `SetDiet` 才是正路。[标签维护](../../../raw/current/scripts/components/eater.lua#L1-L24)、[PrefersToEat](../../../raw/current/scripts/components/eater.lua#L304-L318)、[EAT 拦截](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1236-L1240)
- **边界：`molebait` 只在齿轮上**——电子元件没有 `molebait` 也没有 `bait`，不会被鼹鼠偷、也不能进陷阱诱饵槽。[gears 标签](../../../raw/current/scripts/prefabs/gears.lua#L60)

## 验证记录

- 源码审阅：2026-11-10，Steam buildid 26474873。核验范围：两份 prefab 全文与组件接线；`edible`／`eater` 双层食性门禁与 EAT 动作条件；`repairer` 标签生成、`repairable` 回复分流与右键动作；`useabletargeteditem` 的成功／失败路径与 `clockworktracker` 上限；`CanDoerBefriendTarget` 两条放行条件与 `befriendable_clockwork` 的 SG 来源；齿轮与电子元件的主要掉落条目（发条三兄弟各变体、双恐怖之眼（`twinofterror1`／`twinofterror2`）两套表、天体后裔、废料机器人与无人机、废料堆、垃圾堆、墓碑堆、守护者宝箱、拾荒者、滚草、猴岛残骸、废墟／可砸物、检查眼镜箱两套表、浣猫、骑士神龛的供品交存／退回／删除）；`recipes.lua` 消费行；简中名称键。未覆盖：`chessjunk` 修复完成后的发条生成流程；`minotaurchest` 的开箱槽位随机上限细节；发条三兄弟各变体的掉落表选择条件；`GetModuleTypeCount` 的充能扣减顺序。
- 游戏运行：未运行；概率、回复量与消耗数量均来自源码推导。
- 结构检查：`validate_page.py` 与 `validate_source_links.py` 见交付回报。

相关：[[reference/prefabs/boneshard]]（同批骨材）、[[reference/prefabs/moonrocknugget]]（同批月岩）、[[reference/prefabs/icepack]]（配方同时消耗齿轮与电子元件）、[[reference/prefabs/staffs-more]]（龙卷风法杖等法杖族配方与装备）。

返回 [[reference/prefabs/crafting-materials-index|通用材料与加工中间件（暂行索引）]]、[[reference/prefabs/index|Prefab 索引]]。
