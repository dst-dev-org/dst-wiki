---
title: "旺达怀表族（pocketwatch_heal、pocketwatch_warp 等 11 注册）"
created: "2026-11-06"
updated: "2026-11-08"
type: prefab
summary: "旺达专属时间机制的工厂族：共享工厂 common_fn 注册的五只功能怀表（回春/复活/倒走/溯源/裂缝）、警钟武器、时间碎片、钟表匠工具、时间裂缝出入口与跨分片复活媒介，覆盖施法门禁、冷却、配方与拆解。"
aliases: ["pocketwatch_heal", "pocketwatch_revive", "pocketwatch_revive_reviver", "pocketwatch_warp", "pocketwatch_recall", "pocketwatch_weapon", "pocketwatch_parts", "pocketwatch_dismantler", "pocketwatch_portal", "pocketwatch_portal_entrance", "pocketwatch_portal_exit", "不老表", "第二次机会表", "倒走表", "溯源表", "裂缝表", "警钟", "时间碎片", "钟表匠工具", "时间裂缝", "旺达怀表", "怀表族", "Ageless Watch", "Second Chance Watch", "Backstep Watch", "Backtrek Watch", "Rift Watch", "Alarming Clock", "Time Pieces", "Clockmaker's Tools", "Time Rift"]
tags: [prefab, item, equipment]
size_class: complex
dst_build: "steam-buildid:26474873"
sources: ["prefabs/pocketwatch.lua", "prefabs/pocketwatch_common.lua", "prefabs/pocketwatch_weapon.lua", "prefabs/pocketwatch_parts.lua", "prefabs/pocketwatch_portal.lua", "prefabs/pocketwatch_dismantler.lua", "components/pocketwatch.lua", "components/pocketwatch_dismantler.lua", "components/recallmark.lua", "components/rechargeable.lua", "components/positionalwarp.lua", "components/equippable.lua", "components/weapon.lua", "components/oldager.lua", "components/builder.lua", "components/lootdropper.lua", "recipe.lua", "prefabs/wanda.lua", "prefabs/player_common_extensions.lua", "recipes.lua", "prefabs/staffcastfx.lua", "constants.lua", "actions.lua", "componentactions.lua", "stategraphs/SGwilson.lua", "fx.lua", "stringutil.lua", "tuning.lua", "strings.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 旺达怀表族（pocketwatch_heal、pocketwatch_warp 等 11 注册）

## 快速判断

- 职责与适用：六个 pocketwatch 文件共注册 16 个 Prefab。`prefabs/pocketwatch_common.lua` 导出共享工厂 `common_fn`（不是 Prefab 注册），五只功能怀表 `pocketwatch_heal`（不老表，回春）、`pocketwatch_revive`（第二次机会表，复活）、`pocketwatch_warp`（倒走表，瞬移回历史点）、`pocketwatch_recall`（溯源表，标记/传送）、`pocketwatch_portal`（裂缝表，开门）由它组装；另有武器 `pocketwatch_weapon`（警钟）、材料 `pocketwatch_parts`（时间碎片）、工具 `pocketwatch_dismantler`（钟表匠工具）、临时传送门 `pocketwatch_portal_entrance`／`pocketwatch_portal_exit`（时间裂缝）与跨分片复活媒介 `pocketwatch_revive_reviver`。主要对象类别：施法物品（item）＋手持装备（equipment，仅警钟）。同文件的另 5 个 FX 注册（`pocketwatch_warp_marker`、`pocketwatch_recall_marker`、`pocketwatch_portal_entrance_overlay`／`_underlay`、`pocketwatch_portal_exit_fx`）由 [[reference/prefabs/pocketwatch_warp_marker|怀表与传送门附属表现 FX 族]] 主责，警钟环绕微粒见 [[reference/prefabs/pocketwatch_weapon_fx]]。
- 优先使用：改某只表效果／冷却／伤害查[注册与构造](#注册与构造)与[数值与冷却](#数值与冷却)；让其他角色用上怀表读[典型接入](#让非旺达角色使用怀表)；改配方或拆解产物读[配方与获取](#配方与获取)；自定义"充能施法物品"的装配原型读 [[reference/components/pocketwatch#典型接入]] 与本页共享工厂。
- 重要副作用：施法先过 `pocketwatch.inactive` 充能门禁，只有成功分支才 `rechargeable:Discharge` 进冷却（首次标记与 SHARD_UNAVAILABLE 等失败原因不进冷却）（[组件门禁](../../../raw/current/scripts/components/pocketwatch.lua#L21-L32)）；治疗表同时回春旺达年龄；复活表以幽灵玩家为目标、复活时移除其旧骨架；倒走表瞬移期间无敌；溯源/裂缝标记可跨分片，裂缝表施放后原地替换为继承了标记的溯源表；拆解工具回收配方材料但**不返还噩梦燃料**。
- 修改边界：施法动作只对带 `pocketwatchcaster` 的角色可见，配方／拆解只对 `clockmaker` 开放，两标签原版只在 wanda.lua 授予（[门禁](#角色门禁)）；治疗表的 `DoCastSpell` 无守卫解引用 `doer.components.oldager`，普通角色拿到标签后施放治疗表会报错（[陷阱](#常见陷阱与限制)）；本页不展开 FX 工厂内部与警钟粒子发射参数。

## 注册与构造

共享工厂 `PocketWatchCommon.common_fn(bank, build, DoCastSpell, cast_from_inventory, tags, common_postinit, master_postinit)`（[pocketwatch_common.lua#L31-L93](../../../raw/current/scripts/prefabs/pocketwatch_common.lua#L31-L93)）：实体组件＋`MakeInventoryPhysics` → bank/build＋`idle` → `scrapbook_deps` → `MakeInventoryFloatable("small", 0.05, {1.2, 0.75, 1.2})` → 公共端标签 `pocketwatch`、`cattoy`（`cast_from_inventory=true` 另加 `pocketwatch_castfrominventory`；`tags` 逐个追加）→ `common_postinit` → `SetPristine`（客户端早退）→ 主模拟依次 `lootdropper` → `inventoryitem`（`SetOnDroppedFn`）→ `rechargeable` → `pocketwatch`（注入 `DoCastSpell`）→ `inspectable`（充能中检视返回 RECHARGING）→ `MakeHauntableLaunch` → `master_postinit`。

充能门禁统一在共享回调里：`pocketwatch.inactive` 初始 true，充满回调置 true 并播回 `idle`，放电回调置 false（[OnCharged／OnDischarged](../../../raw/current/scripts/prefabs/pocketwatch_common.lua#L12-L24)）；组件的 `CanCast`／`CastSpell` 都要求 `inactive` 才执行 `DoCastSpell`（[组件](../../../raw/current/scripts/components/pocketwatch.lua#L21-L32)），字段变化同步增删 `pocketwatch_inactive` 实体标签（[oninactive](../../../raw/current/scripts/components/pocketwatch.lua#L2-L4)），componentactions 靠该标签过滤动作。掉落时按剩余充能比例播 `cooldown_long`／`cooldown_short` 进度动画（[OnDropped](../../../raw/current/scripts/prefabs/pocketwatch_common.lua#L2-L10)）。

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `pocketwatch_heal`（不老表） | `healfn` → `common_fn("pocketwatch","pocketwatch_marble",Heal_DoCastSpell, true, MOUNTED_CAST_TAGS)` | 施法＝`oldager:StopDamageOverTime()`＋`health:DoDelta(20,true,"pocketwatch_heal")`＋heal_fx（骑乘换 `_mount` 版）＋Discharge；`castfxcolour` 暖白；可骑乘施法。[施法](../../../raw/current/scripts/prefabs/pocketwatch.lua#L29-L41)、[工厂](../../../raw/current/scripts/prefabs/pocketwatch.lua#L45-L55) |
| `pocketwatch_revive`（第二次机会表） | `revivefn` → `common_fn("pocketwatch","pocketwatch_wood",Revive_DoCastSpell, false, MOUNTED_CAST_TAGS)` | 不带 `pocketwatch_castfrominventory`：对幽灵玩家点目标施法；`GetActionVerb_CAST_POCKETWATCH="REVIVER"`、`pocketwatch_CanTarget`＝`CanCastFn`（目标须 playerghost 且未在复活中）；监听 `activateresurrection` 移除同 userid 玩家骨架；自定义作祟回调。[施法与目标](../../../raw/current/scripts/prefabs/pocketwatch.lua#L113-L133)、[工厂](../../../raw/current/scripts/prefabs/pocketwatch.lua#L146-L165) |
| `pocketwatch_revive_reviver`（同显示名"第二次机会表"） | 手写 `revive_reviverfn`，不经共享工厂：Transform＋Network → 主模拟 `inventoryitem(SetSinks)` → 监听 `activateresurrection` 自移除 → `DoTaskInTime(0)` 跑 `ReviveOwner` | 跨分片复活媒介：对"死亡分片≠当前分片"的幽灵施放时塞进其背包；到死亡分片后对幽灵主人推 `respawnfromghost(source=自身)`，主人不是幽灵则自删。[媒介工厂](../../../raw/current/scripts/prefabs/pocketwatch.lua#L78-L109)、[授予](../../../raw/current/scripts/prefabs/pocketwatch.lua#L118-L133) |
| `pocketwatch_warp`（倒走表） | `warpfn` → `common_fn("pocketwatch","pocketwatch_warp",Warp_DoCastSpell, true, {"pocketwatch_warp","pocketwatch_warp_casting"})` | 施法＝瞬移到持有者 `positionalwarp` 历史点（无历史/禁传送区返回失败原因）；`pocketwatch_warp_casting` 让 SG 走 warpback 专用链；入包/掉落向主人推 `show_warp_marker`／`hide_warp_marker` 事件驱动落点标记显示。[施法](../../../raw/current/scripts/prefabs/pocketwatch.lua#L368-L382)、[事件](../../../raw/current/scripts/prefabs/pocketwatch.lua#L386-L416) |
| `pocketwatch_recall`（溯源表） | `recallfn` → `common_fn("pocketwatch","pocketwatch_recall",Recall_DoCastSpell, true, {"pocketwatch_warp_casting","gemsocket"})`＋`MakeRecallMarkable` | 未标记＝记录施法者位置（recallmark＋音效＋延迟台词）；已标记＝warpback 到标记点（可跨分片，目标分片不可用报 SHARD_UNAVAILABLE）；`trader` 只收紫宝石，收到即变形为 `pocketwatch_portal`（继承标记，替换原槽位）；`OnBuiltFn` 给建造者学习裂缝表配方。[施法](../../../raw/current/scripts/prefabs/pocketwatch.lua#L226-L247)、[宝石交易](../../../raw/current/scripts/prefabs/pocketwatch.lua#L249-L275)、[工厂](../../../raw/current/scripts/prefabs/pocketwatch.lua#L288-L306) |
| `pocketwatch_portal`（裂缝表） | `fn` → `common_fn("pocketwatch","pocketwatch_portal",DoCastSpell, true)`＋`MakeRecallMarkable`；`inst.onPreBuilt` | 未标记＝同溯源表记录；已标记＝在附近找落点生成 `pocketwatch_portal_entrance`＋出口（跨分片写迁移目标），本体替换为只复制标记、并立即 `Discharge(480)` 的新 `pocketwatch_recall`；另一条“溯源表喂紫宝石→裂缝表”的变形才走 `onPreBuilt`，从被消耗的溯源表复制标记与（未满的）充能进度。[施法](../../../raw/current/scripts/prefabs/pocketwatch_portal.lua#L42-L90)、[PreBuilt](../../../raw/current/scripts/prefabs/pocketwatch_portal.lua#L97-L109) |
| `pocketwatch_portal_entrance`（时间裂缝） | 手写 `portal_entrance_fn`：Transform/AnimState/SoundEmitter/Light/Network＋两个 netvar（`_lightframe`/`_islighton`） | 临时传送设施：10 秒 timer 自关（`closeportal`）；`teleporter` 与出口配对（跨分片 `MigrationTarget`）；`OnActivate` 对有 sanity 且无 `pocketwatchcaster`／`nowormholesanityloss` 标签者扣 20 理智；主模拟父挂生成 overlay/underlay（FX 页主责）。[入口工厂](../../../raw/current/scripts/prefabs/pocketwatch_portal.lua#L237-L304)、[SpawnExit](../../../raw/current/scripts/prefabs/pocketwatch_portal.lua#L205-L220) |
| `pocketwatch_portal_exit`（同显示名"时间裂缝"） | 手写 `portal_exit_fn`：仅 Transform，非联网 `CLASSIFIED`，`persists=false` | 出口侧 `teleporter`：`trynooffset=true`、`offset=3`、到达态覆写 `pocketwatch_portal_land`；`OnDoneTeleporting` 重置到达者 `positionalwarp` 并延迟台词（施法者 1.25s/其他 2.5s）。[出口工厂](../../../raw/current/scripts/prefabs/pocketwatch_portal.lua#L405-L435)、[到达回调](../../../raw/current/scripts/prefabs/pocketwatch_portal.lua#L393-L403) |
| `pocketwatch_weapon`（警钟） | 手写 `fn`（bank/build `pocketwatch_weapon`），`net_event` 播放加燃料音 | 本族唯一装备：`equippable`（`restrictedtag="pocketwatchcaster"`，槽位用组件默认 HANDS）；`weapon` 伤害按燃料两档切换并增删 `shadow_item` 标签；`fueled` NIGHTMARE 型 720；攻击距离 `WHIP_RANGE`；无 finiteuses，磨损在 onattack 里手动扣燃料。[工厂](../../../raw/current/scripts/prefabs/pocketwatch_weapon.lua#L121-L185)、[燃料切档](../../../raw/current/scripts/prefabs/pocketwatch_weapon.lua#L82-L96) |
| `pocketwatch_parts`（时间碎片） | 手写 `fn`（bank/build `pocketwatch_parts`），带心跳音效任务 | 材料＋诱饵：`stackable`（上限 20）＋`bait`，标签 `molebait`、`cattoy`；掉落/唤醒播心跳循环音，拾取停止；无 `pocketwatch` 标签（不可被拆解）。[工厂](../../../raw/current/scripts/prefabs/pocketwatch_parts.lua#L46-L91) |
| `pocketwatch_dismantler`（钟表匠工具） | 手写 `fn`（bank/build `pocketwatch_dismantler`），`scrapbook_specialinfo="POCKETWATCHDISMANTLER"` | 拆解工具：`pocketwatch_dismantler` 组件对 `pocketwatch` 标签目标回收配方材料（除噩梦燃料）并丢 `brokentool`；可燃（SMALL_BURNTIME）可作祟。[工厂](../../../raw/current/scripts/prefabs/pocketwatch_dismantler.lua#L13-L50)、[拆解](../../../raw/current/scripts/components/pocketwatch_dismantler.lua#L5-L42) |

注册尾（[pocketwatch.lua#L423-L429](../../../raw/current/scripts/prefabs/pocketwatch.lua#L423-L429)、[pocketwatch_portal.lua#L509-L514](../../../raw/current/scripts/prefabs/pocketwatch_portal.lua#L509-L514)、[pocketwatch_weapon.lua#L189](../../../raw/current/scripts/prefabs/pocketwatch_weapon.lua#L189)、[pocketwatch_parts.lua#L93](../../../raw/current/scripts/prefabs/pocketwatch_parts.lua#L93)、[pocketwatch_dismantler.lua#L50](../../../raw/current/scripts/prefabs/pocketwatch_dismantler.lua#L50)）：6 文件合计 16 注册，其中 5 个 FX 注册归 [[reference/prefabs/pocketwatch_warp_marker]]。无 quagmire 分支、skinprefabs.lua 中无本族皮肤注册（grep 无结果）。

名称键经简中 .po `msgctxt` 逐个核验：`POCKETWATCH_HEAL`=不老表（Ageless Watch）、`POCKETWATCH_REVIVE`／`POCKETWATCH_REVIVE_REVIVER`=第二次机会表（Second Chance Watch）、`POCKETWATCH_WARP`=倒走表（Backstep Watch）、`POCKETWATCH_RECALL`=溯源表（Backtrek Watch）、`POCKETWATCH_PORTAL`／`POCKETWATCH_PORTAL_ENTRANCE`／`POCKETWATCH_PORTAL_EXIT`=裂缝表／时间裂缝（Rift Watch／Time Rift）、`POCKETWATCH_WEAPON`=警钟（Alarming Clock）、`POCKETWATCH_PARTS`=时间碎片（Time Pieces）、`POCKETWATCH_DISMANTLER`=钟表匠工具（Clockmaker's Tools）。[NAMES 键区](../../../raw/current/scripts/languages/chinese_s.po#L347045-L347097)

### 数值与冷却

（tuning 键＋实际值；`seg_time`=30、`total_day_time`=480、`wilson_attack`=34×倍率 1，[tuning.lua#L19-L37](../../../raw/current/scripts/tuning.lua#L19-L37)）

| 项 | 键／来源 | 实际值 |
|---|---|---|
| 治疗量 | `POCKETWATCH_HEAL_HEALING` | 20 生命（[tuning.lua#L5011](../../../raw/current/scripts/tuning.lua#L5011)） |
| 治疗回春换算 | `OLDAGE_HEALTH_SCALE`=60/150 | 20×0.4＝回春 8 年（有效治疗原因 `pocketwatch_heal`；[原因注册](../../../raw/current/scripts/prefabs/wanda.lua#L397)、[换算](../../../raw/current/scripts/components/oldager.lua#L45-L48)、[tuning.lua#L2740](../../../raw/current/scripts/tuning.lua#L2740)） |
| 治疗冷却 | `POCKETWATCH_HEAL_COOLDOWN`=`seg_time`×4 | 120 秒（[tuning.lua#L5010](../../../raw/current/scripts/tuning.lua#L5010)） |
| 复活冷却 | `POCKETWATCH_REVIVE_COOLDOWN`=`total_day_time`×0.5 | 240 秒（[tuning.lua#L5013](../../../raw/current/scripts/tuning.lua#L5013)） |
| 倒走冷却 | `POCKETWATCH_WARP_COOLDOWN` | 2 秒（[tuning.lua#L5014](../../../raw/current/scripts/tuning.lua#L5014)） |
| 溯源/裂缝充能 | `POCKETWATCH_RECALL_COOLDOWN`=`total_day_time` | 480 秒（[tuning.lua#L5015](../../../raw/current/scripts/tuning.lua#L5015)） |
| 警钟伤害（有燃料） | `POCKETWATCH_SHADOW_DAMAGE`=`wilson_attack`×2.4 | 81.6 生命（[tuning.lua#L5007](../../../raw/current/scripts/tuning.lua#L5007)） |
| 警钟伤害（无燃料） | `POCKETWATCH_DEPLETED_DAMAGE`=`wilson_attack`×0.8 | 27.2 生命（[tuning.lua#L5008](../../../raw/current/scripts/tuning.lua#L5008)） |
| 警钟攻击距离／燃料 | `WHIP_RANGE`＝2；`4×LARGE_FUEL`＝720；每击 `-TINY_FUEL`＝7.5 | 充能态约 96 击（720/7.5，源码推导；[燃料初始化](../../../raw/current/scripts/prefabs/pocketwatch_weapon.lua#L170-L174)、[每击扣减](../../../raw/current/scripts/prefabs/pocketwatch_weapon.lua#L63-L65)、[tuning.lua#L423](../../../raw/current/scripts/tuning.lua#L423)、[L1673](../../../raw/current/scripts/tuning.lua#L1673)、[L1677](../../../raw/current/scripts/tuning.lua#L1677)） |
| 倒走步数（旺达年龄三档） | `WANDA_WARP_DIST_OLD/NORMAL/YOUNG`＝2/4/8 | 历史点间距≥2 单位、0.1 秒采样、环形 60 点（[档位接线](../../../raw/current/scripts/prefabs/wanda.lua#L86-L153)、[tuning.lua#L5036-L5038](../../../raw/current/scripts/tuning.lua#L5036-L5038)、[历史机制](../../../raw/current/scripts/components/positionalwarp.lua#L12-L21)） |
| 裂缝入口存续／乘门理智 | 入口 timer `closeportal`=10 秒；有 sanity 且无 `pocketwatchcaster`／`nowormholesanityloss` 标签者扣 `SANITY_MEDLARGE`=20 | [timer](../../../raw/current/scripts/prefabs/pocketwatch_portal.lua#L283-L285)、[理智](../../../raw/current/scripts/prefabs/pocketwatch_portal.lua#L222-L230)、[tuning.lua#L2036](../../../raw/current/scripts/tuning.lua#L2036) |
| 其他 | 时间碎片堆叠 `STACK_SIZE_MEDITEM`=20；工具燃烧 `SMALL_BURNTIME`=`seg_time`×0.25=7.5 秒 | [stackable](../../../raw/current/scripts/prefabs/pocketwatch_parts.lua#L79-L80)、[tuning.lua#L82](../../../raw/current/scripts/tuning.lua#L82)、[tuning.lua#L1681](../../../raw/current/scripts/tuning.lua#L1681) |

### 配方与获取

全部配方带 `builder_tag="clockmaker"`（[recipes.lua#L360-L369](../../../raw/current/scripts/recipes.lua#L360-L369)）；`no_deconstruction=pocketwatch_nodecon`＝冷却中（无 `pocketwatch_inactive` 标签）禁止拆解（[谓词](../../../raw/current/scripts/recipes.lua#L360)）。

| 注册名 | 配方 | 科技 | 备注 |
|---|---|---|---|
| `pocketwatch_dismantler` | 金块 1＋燧石 1＋树枝 3 | `TECH.NONE` | 拆解产物＝目标配方材料**除噩梦燃料**＋`brokentool`（[Dismantle](../../../raw/current/scripts/components/pocketwatch_dismantler.lua#L16-L42)） |
| `pocketwatch_parts` | 铥矿碎片 8＋噩梦燃料 2＋**携带**钟表匠工具 | `TECH.NONE` | 配料数量 0＝`HasIngredients` 按 1 检查持有、`GetIngredients` 只收集 amount>0 不消耗（[检查](../../../raw/current/scripts/components/builder.lua#L929-L933)、[收集](../../../raw/current/scripts/components/builder.lua#L502-L513)、[配料构造](../../../raw/current/scripts/recipe.lua#L5-L22)） |
| `pocketwatch_heal` | 时间碎片 1＋大理石 2＋红宝石 1 | `TECH.NONE` | 冷却中禁拆 |
| `pocketwatch_revive` | 时间碎片 1＋活木 2＋骨片 4 | `TECH.NONE` | 冷却中禁拆 |
| `pocketwatch_warp` | 时间碎片 1＋金块 2 | `TECH.NONE` | 冷却中禁拆 |
| `pocketwatch_recall` | 时间碎片 2＋金块 2＋海象长牙 1 | `TECH.MAGIC_TWO` | 冷却中禁拆 |
| `pocketwatch_portal` | 溯源表 1（`deconstruct=true` 配料）＋紫宝石 1，`actionstr="SOCKET"` | `TECH.MAGIC_TWO` | 另一条获取路径：对溯源表喂紫宝石（trader）直接变形；`deconstruct=true` 使拆解裂缝表时按溯源表配方展开材料而非掉溯源表本体（[lootdropper 展开](../../../raw/current/scripts/components/lootdropper.lua#L117-L126)） |
| `pocketwatch_weapon` | 时间碎片 3＋大理石 4＋噩梦燃料 8 | `TECH.MAGIC_THREE` | 冷却中禁拆 |

`pocketwatch_revive_reviver` 无配方，仅由复活施法在跨分片场景生成。

### 角色门禁

- `pocketwatchcaster`：仅 wanda.lua `common_postinit` 授予（[wanda.lua#L371-L374](../../../raw/current/scripts/prefabs/wanda.lua#L371-L374)）。消费端：`ACTIONS.CAST_POCKETWATCH.fn`（[actions.lua#L3146-L3151](../../../raw/current/scripts/actions.lua#L3146-L3151)）、点目标/物品栏两条动作过滤器（[componentactions.lua#L1560-L1566](../../../raw/current/scripts/componentactions.lua#L1560-L1566)、[L2848-L2856](../../../raw/current/scripts/componentactions.lua#L2848-L2856)）、警钟 `equippable.restrictedtag`（[装备限制](../../../raw/current/scripts/components/equippable.lua#L159-L161)）、入口乘门理智豁免与出口台词时差、复活表作祟自复活、溯源表检视分支（[RecallMarkable_GetStatus](../../../raw/current/scripts/prefabs/pocketwatch_common.lua#L137-L141)）、到达态 `pocketwatch_portal_land` 对非施法者转 fallout（[SGwilson.lua#L20817-L20820](../../../raw/current/scripts/stategraphs/SGwilson.lua#L20817-L20820)）。
- `clockmaker`：同处授予。消费端：八条配方 `builder_tag`、拆解动作过滤器（[componentactions.lua#L1568-L1572](../../../raw/current/scripts/componentactions.lua#L1568-L1572)）与 `CanDismantle`（[组件](../../../raw/current/scripts/components/pocketwatch_dismantler.lua#L5-L14)）、独立制作栏页签 `CLOCKMAKER`（owner_tag 挂该标签，[constants.lua#L1578](../../../raw/current/scripts/constants.lua#L1578)）。
- 普通角色可以拾取、持有怀表（无装备限制、无拾取门禁），但看不到施法动作、不能制作/拆解、不能装备警钟；旺达死亡时身上怀表 `keepondeath=true`，唯独 `pocketwatch_revive` 例外掉落以便幽灵作祟自复活（[wanda.lua#L308-L309](../../../raw/current/scripts/prefabs/wanda.lua#L308-L309)）。骑乘坐骑施法需目标表带 `pocketwatch_mountedcast` 标签（`MOUNTED_CAST_TAGS`；复活表不带 castfrominventory 也可骑乘点选施法），过滤器同时检查（[componentactions.lua#L1563-L1565](../../../raw/current/scripts/componentactions.lua#L1563-L1565)）。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：pocketwatch | `inactive` 充能门禁＋`DoCastSpell` 分发；revive 表注入 `CanCastFn` | 五只功能怀表＋裂缝表，主模拟；`pocketwatch_inactive` 标签供客户端过滤 | [[reference/components/pocketwatch#关键接口]]；[组件](../../../raw/current/scripts/components/pocketwatch.lua#L21-L32) |
| 组件：rechargeable | `Discharge(chargetime)` 置 0 后按 chargetime 回充；OnCharged/OnDischarged 切 `inactive` | 五只功能表（含裂缝表）；`chargetimemod` 支持加速充能 | [[reference/components/rechargeable#关键接口]]；[Discharge](../../../raw/current/scripts/components/rechargeable.lua#L129-L132) |
| 组件：recallmark | 记录/复制标记点与分片 id，存档跟随 | 仅溯源表＋裂缝表（`MakeRecallMarkable`），主模拟 | [[reference/components/recallmark#关键接口]]；[MarkPosition](../../../raw/current/scripts/components/recallmark.lua#L7-L26) |
| 组件：trader | 只收紫宝石 → `Recall_OnGemGiven` 变形为裂缝表 | 仅溯源表 | [[reference/components/trader#关键接口]]；[onaccept](../../../raw/current/scripts/prefabs/pocketwatch.lua#L260-L275) |
| 组件：equippable／weapon／fueled | 警钟装备（restrictedtag）、两档伤害、NIGHTMARE 燃料；onattack 手动扣燃料（无 finiteuses） | 仅警钟；`percentusedchange` 驱动标签与伤害切换 | [[reference/components/equippable]]、[[reference/components/weapon#关键接口]]、[[reference/components/fueled#关键接口]]；[OnFuelChanged](../../../raw/current/scripts/prefabs/pocketwatch_weapon.lua#L82-L96) |
| 组件：teleporter | 入口配对出口/迁移目标；出口覆写到达态并重置旺达历史 | entrance/exit 专属 | [[reference/components/teleporter#生命周期]]；[SpawnExit](../../../raw/current/scripts/prefabs/pocketwatch_portal.lua#L205-L220) |
| 组件：pocketwatch_dismantler | `CanDismantle`（目标冷却中拒绝 ONCOOLDOWN＋doer 须 clockmaker）→ `Dismantle` 回收配方材料 | 仅钟表匠工具持有；作用于 `pocketwatch` 标签目标（parts 不带该标签不可拆） | [[reference/components/pocketwatch-dismantler#关键接口]]；[组件](../../../raw/current/scripts/components/pocketwatch_dismantler.lua#L5-L42) |
| 组件：stackable／bait／burnable | 时间碎片堆叠 20 与鼹鼠诱饵；钟表匠工具小型可燃 | parts／dismantler | [[reference/components/inventoryitem#关键接口]]；[parts 组件](../../../raw/current/scripts/prefabs/pocketwatch_parts.lua#L72-L80)、[burnable](../../../raw/current/scripts/prefabs/pocketwatch_dismantler.lua#L43-L45) |
| 宿主组件：positionalwarp／oldager | 旺达实体的移动历史（倒走表数据源）与年龄（治疗表回春目标）；`AddValidHealingCause("pocketwatch_heal")` | 旺达专属挂载；普通角色没有，治疗表对其施放会崩（见陷阱） | [[reference/components/positionalwarp#关键接口]]、[[reference/components/oldager#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/wanda.lua#L396-L401) |
| SG：SGwilson `pocketwatch_cast` 等 | 动作路由：`recall_unmarked`→dolongaction、`pocketwatch_warp_casting`→warpback 三态链、portal→`pocketwatch_openportal`，其余→`pocketwatch_cast`（25 帧执行动作、8 帧生成 cast_fx） | 施法者端；warpback 链 4 帧起无敌并关影子，>30 单位拉黑屏；`pocketwatch_portal_land` 非施法者转 fallout | [路由](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1336-L1344)、[pocketwatch_cast](../../../raw/current/scripts/stategraphs/SGwilson.lua#L20459-L20550)、[warpback](../../../raw/current/scripts/stategraphs/SGwilson.lua#L20595-L20680) |
| API／动作：`CAST_POCKETWATCH`／`DISMANTLE_POCKETWATCH` | RMB 优先级 -1、mount_valid；verb 按 `GetActionVerb_CAST_POCKETWATCH` 取（REVIVER=复活、RECALL_MARK=在空间中设定时间，RECALL/WARP 已注释回退 GENERIC=激活） | 服务端结算；动作字符串缺键回退 GENERIC（[getmodifiedstring](../../../raw/current/scripts/stringutil.lua#L11-L16)） | [动作声明](../../../raw/current/scripts/actions.lua#L456)、[fn](../../../raw/current/scripts/actions.lua#L3146-L3151)、[DISMANTLE fn](../../../raw/current/scripts/actions.lua#L5687-L5694)、[verb 键](../../../raw/current/scripts/strings.lua#L1020-L1028)、[.po 动词](../../../raw/current/scripts/languages/chinese_s.po#L514-L527) |
| API／数据：复活链 `respawnfromghost` | 复活表施法/作祟推事件 → `start_rewindtime_revive`/直接回魂，先按 `last_death_position` 传送或跨分片迁移再复活；媒体为 revive/reviver 时进 `rewindtime_rebirth` 并在媒体上推 `activateresurrection` | 旺达复活专用路径（[OnRespawnFromGhost 分支](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L595-L606)）；reviver 到达后补推事件（[ReviveOwner](../../../raw/current/scripts/prefabs/pocketwatch.lua#L78-L87)） | [DoActualRez](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L313-L410)、[DoMoveToRezPosition](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L540-L563)、[OnRespawnFromGhost](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L569-L611) |
| 关联 Prefab：施法 FX | `pocketwatch_cast_fx(_mount)`／`pocketwatch_warpback_fx`／`_out_fx` 由 staffcastfx.lua 的 MakeStaffFX 注册；`pocketwatch_heal_fx(_mount)`／`pocketwatch_ground_fx` 在 fx.lua 通用表 | SG 与施法函数按骑乘/场景生成；本文件只声明依赖 | [staffcastfx 注册](../../../raw/current/scripts/prefabs/staffcastfx.lua#L69-L72)、[fx.lua heal/ground](../../../raw/current/scripts/fx.lua#L2570-L2590)、[[reference/prefabs/pocketwatch_weapon_fx|警钟微粒]] |
| 关联 Prefab：`brokentool` | 拆解产物与复活表被作祟自毁的残骸 | 生成后独立存在 | [拆解](../../../raw/current/scripts/components/pocketwatch_dismantler.lua#L29)、[作祟自毁](../../../raw/current/scripts/prefabs/pocketwatch.lua#L135-L144) |
| 资源：anim | 怀表五 build（pocketwatch/pocketwatch_marble/_recall/_warp/_wood 共用 bank `pocketwatch`）、pocketwatch_weapon、pocketwatch_parts、pocketwatch_dismantler、pocketwatch_portal_fx、pocketwatch_warp_marker | 声明与播放点已核验；包内容未解包 | [怀表 assets](../../../raw/current/scripts/prefabs/pocketwatch.lua#L2-L11)、[weapon](../../../raw/current/scripts/prefabs/pocketwatch_weapon.lua#L2-L5)、[parts](../../../raw/current/scripts/prefabs/pocketwatch_parts.lua#L1-L4)、[portal](../../../raw/current/scripts/prefabs/pocketwatch_portal.lua#L4-L10) |

## 典型接入

### 让非旺达角色使用怀表

- 前提与选择：服务端 `modmain.lua`。门禁是三条链的组合：`pocketwatchcaster`（施法动作＋警钟装备）、`clockmaker`（配方＋拆解）、以及治疗表对 `oldager` 的无守卫调用。原版把前两个标签只给旺达；给其他角色补标签即可开放，但治疗表必须额外包一层 `DoCastSpell`——`Heal_DoCastSpell` 直接调 `doer.components.oldager:StopDamageOverTime()`（[施法](../../../raw/current/scripts/prefabs/pocketwatch.lua#L29-L41)），没有该组件会报错，而给角色挂 `oldager` 组件会让其开始老化（构造即 `StartUpdatingComponent`，[oldager.lua#L22](../../../raw/current/scripts/components/oldager.lua#L22)），不能作为修复手段。
- 生效与撤销：`AddPrefabPostInit` 对指定角色加标签，作用于其实体全程；`DoCastSpell` 包装只影响新施法调用，撤销模组即恢复原行为。复活表以幽灵为目标、倒走表依赖 `positionalwarp` 历史，普通角色拿到标签后倒走表会先在 `Warp_DoCastSpell` 第一行解引用 `positionalwarp` 报错——示例必须同时给倒走表加守卫，或不用倒走表。
- 原版依据：门禁消费端见[角色门禁](#角色门禁)；动作过滤器还要求表处于充能完成状态（`pocketwatch_inactive` 标签）。

```lua
-- modmain.lua：允许威尔逊使用怀表（施法＋制作＋拆解），并修掉治疗表的 oldager 崩溃
AddPrefabPostInit("wilson", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    inst:AddTag("pocketwatchcaster")
    inst:AddTag("clockmaker")
end)

AddPrefabPostInit("pocketwatch_warp", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    local pocketwatch = inst.components.pocketwatch
    local old_cast = pocketwatch.DoCastSpell
    pocketwatch.DoCastSpell = function(inst, doer, target, pos)
        if doer ~= nil and doer.components.positionalwarp == nil then
            return false, "WARP_NO_POINTS_LEFT" -- 无历史组件按无历史处理
        end
        return old_cast(inst, doer, target, pos)
    end
end)

AddPrefabPostInit("pocketwatch_heal", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    local pocketwatch = inst.components.pocketwatch
    local old_cast = pocketwatch.DoCastSpell
    pocketwatch.DoCastSpell = function(inst, doer, target, pos)
        if doer ~= nil and doer.components.oldager == nil then
            -- 非旺达：跳过回春调用，仅保留治疗与冷却
            local health = doer.components.health
            if health ~= nil and not health:IsDead() then
                health:DoDelta(TUNING.POCKETWATCH_HEAL_HEALING, true, inst.prefab)
                if inst.components.rechargeable ~= nil then
                    inst.components.rechargeable:Discharge(TUNING.POCKETWATCH_HEAL_COOLDOWN)
                end
                return true
            end
            return false
        end
        return old_cast(inst, doer, target, pos)
    end
end)
```

- 调整与验证：标签在主模拟添加即可让客户端动作过滤器生效（`pocketwatch_inactive`／`pocketwatchcaster` 都走标签复制）；modmain 环境只直挂 `TUNING` 等白名单，`TheWorld` 须经 `GLOBAL.` 前缀取（mods.lua CreateEnvironment 无 `__index=_G`）。未做游戏运行验证。

### 调整冷却与武器数值

- 前提与选择：服务端 `modmain.lua` 顶层。冷却在每次施法时读 `TUNING.POCKETWATCH_*_COOLDOWN` 传入 `Discharge`；警钟伤害只在**构造时**（按满燃料档）与 `percentusedchange` 事件里写入组件，已有实例不会重读 TUNING。
- 生效与撤销：冷却对已有实例下次施法生效；警钟伤害对已有实例不追溯——要换档须触发一次燃料变化（或直接 `SetDamage`）。撤销模组后恢复原值，存档中的进行中冷却不会被追溯。

```lua
-- modmain.lua：治疗表冷却 120 秒改 60 秒，警钟无燃料伤害 27.2 改 34
TUNING.POCKETWATCH_HEAL_COOLDOWN = 60
TUNING.POCKETWATCH_DEPLETED_DAMAGE = 34
```

- 调整与验证：单位为秒／生命值；`TUNING` 在 modmain 白名单内可直接写，运行时全局须 `GLOBAL.` 前缀的接入写法参照 [[reference/prefabs/ruins-bat#典型接入]]。更多装配入口（自定义怀表施法）见 [[reference/components/pocketwatch#典型接入]]。

## 常见陷阱与限制

- **误区：怀表是手持装备**：共享工厂不挂 `equippable`，施法走物品栏 RMB（`pocketwatch_castfrominventory`）或对目标点选，装备槽相关的 postinit 对怀表无效；本族唯一装备是警钟（HANDS 槽）。[工厂](../../../raw/current/scripts/prefabs/pocketwatch_common.lua#L31-L93)、[警钟 equippable](../../../raw/current/scripts/prefabs/pocketwatch_weapon.lua#L159-L163)
- **陷阱：给普通角色开放后施放治疗表／倒走表崩溃**：`Heal_DoCastSpell` 无守卫调用 `doer.components.oldager`、`Warp_DoCastSpell` 首行无守卫调用 `doer.components.positionalwarp:GetHistoryPosition`，两组件都只挂旺达；给其他角色加 `pocketwatchcaster` 后必须按[典型接入](#让非旺达角色使用怀表)分别包装两条 `DoCastSpell`。[施法](../../../raw/current/scripts/prefabs/pocketwatch.lua#L29-L41)、[组件挂载](../../../raw/current/scripts/prefabs/wanda.lua#L396)
- **边界：`pocketwatch_inactive` 标签语义与命名相反**：标签在 `inactive==true`（已充满、可用）时添加，冷却中反而没有该标签；动作过滤、拆解禁用（`pocketwatch_nodecon`）都按"无标签＝不可用"判断。[oninactive](../../../raw/current/scripts/components/pocketwatch.lua#L2-L4)、[禁拆谓词](../../../raw/current/scripts/recipes.lua#L360)
- **边界：复活表目标只能是幽灵玩家，不是骨架**：`Revive_CanTarget` 要求 `playerghost` 且未在复活中；复活成功后按玩家 `last_death_position` 回魂（同分片传送、跨分片迁移），并移除现场同 userid 的旧骨架（掉落其物品）。scrapbook 文案提到的"对骨架使用"与当前代码门禁不符，以代码为准。[目标检查](../../../raw/current/scripts/prefabs/pocketwatch.lua#L113-L116)、[骨架移除](../../../raw/current/scripts/prefabs/pocketwatch.lua#L59-L71)、[回魂链](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L569-L611)
- **边界：倒走表历史是旺达实体的**：`positionalwarp` 只挂在旺达身上，0.1 秒采样、移动超过 2 单位才记录、环形 60 点；掉线/重生后 `Reset`。其他角色被开放施法时，须先按[典型接入](#让非旺达角色使用怀表)给倒走表加守卫（或挂 `positionalwarp`），才会得到 `WARP_NO_POINTS_LEFT` 失败；否则会在 `GetHistoryPosition` 处报错。倒走会回退历史指针（连续施放逐步回溯更远），溯源表则带 `reset_warp=true` 清空历史。[采样与回退](../../../raw/current/scripts/components/positionalwarp.lua#L99-L144)、[两种回退语义](../../../raw/current/scripts/prefabs/wanda.lua#L339-L348)
- **边界：裂缝表替换而非保留**：开门施法成功后本体被 Remove，同槽位换成只复制标记、并立即 `Discharge(480)` 的新溯源表（不继承充能进度）；入口 10 秒自关，跨分片走 `ms_playerdespawnandmigrate` 迁移而不是实体传送。[施法](../../../raw/current/scripts/prefabs/pocketwatch_portal.lua#L42-L90)、[迁移分支](../../../raw/current/scripts/prefabs/pocketwatch_portal.lua#L205-L212)
- **边界：拆解回收不含噩梦燃料**：`Dismantle` 取目标配方全量材料后过滤 `nightmarefuel`，额外在施法者脚下丢 `brokentool`；目标带 `itemmimic` 时不拆解而是变敌；目标冷却中拒绝（ONCOOLDOWN）。[Dismantle](../../../raw/current/scripts/components/pocketwatch_dismantler.lua#L16-L42)、[CanDismantle](../../../raw/current/scripts/components/pocketwatch_dismantler.lua#L5-L14)
- **边界：警钟空燃料后不消耗也不播暗影音**：onattack 只在燃料非空时扣 `TINY_FUEL`；伤害档位、`shadow_item` 标签与微粒 FX 都由 `percentusedchange` 事件统一切换，加载时延迟一帧对齐一次。[onattack](../../../raw/current/scripts/prefabs/pocketwatch_weapon.lua#L63-L75)、[事件对齐](../../../raw/current/scripts/prefabs/pocketwatch_weapon.lua#L176-L180)

## 验证记录

- 源码审阅：2026-11-06，Steam buildid 26335394（`raw/current/BUILD.txt`）。全文核验 6 个 pocketwatch 文件，逐一列出 16 个注册并区分共享工厂/专属 postinit；核验 `pocketwatch`／`recallmark`／`rechargeable`／`positionalwarp`／`pocketwatch_dismantler`／`oldager` 组件、`equippable` 默认 HANDS 槽与 `restrictedtag` 检查、`weapon:OnAttack` 顺序；追完 5 条施法链（治疗/复活/倒走/溯源/裂缝）到 SG 状态与 `player_common_extensions` 复活迁移链；核验 `CAST_POCKETWATCH`/`DISMANTLE_POCKETWATCH` 声明、fn 与 componentactions 过滤器、配方八条与 `builder_tag="clockmaker"`、0 数量配料与 `deconstruct` 配料的 builder/lootdropper 消费、tuning 各键实际值、`chinese_s.po` 11 个 NAMES 键与动作动词键。确认：blinkstaff 组件未挂载本族任何 Prefab（瞬移走 positionalwarp＋SG warpback，回溯治疗走 oldager）；无 quagmire 分支、无皮肤注册。
- 游戏运行：未运行；`validation` 保持 `source-reviewed`。未覆盖：实机施法手感、跨分片迁移时序、充能动画播放、警钟实战伤害（81.6/27.2 与 96 击为源码推导）。
- 结构检查：validate_page.py 通过（203 行／38802 字节，complex 档 480/49152）；validate_source_links.py --strict 通过（129 条源码行引用、29 个源文件，0 error 0 warning，初跑 4 条 source-end-blank 警告已按范围收窄消除）；两个 lua 围栏经 parser_probe --syntax-only（Lua 5.1）通过。
- 分档说明：本页用 `size_class: complex`——11 个主注册共享一个工厂但各自接不同施法链（SG 状态、跨分片迁移、复活媒介生命周期），需要整体对照保留；不按源码文件总量扩额。

相关页面：[[reference/prefabs/pocketwatch_warp_marker|怀表与传送门附属表现 FX 族]]（同文件的 5 个 FX 注册）、[[reference/prefabs/pocketwatch_weapon_fx|警钟暗影微粒特效]]、[[reference/components/pocketwatch|pocketwatch 组件]]。

返回 [[reference/prefabs/staffs-character-index|法杖与角色道具]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-08 build 26474873：本页随版本迁移升版（机械沿用；引用区间未受本次源码行号位移影响），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-07）。
