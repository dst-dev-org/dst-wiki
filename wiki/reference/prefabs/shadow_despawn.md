---
title: "暗影消散、舞台影手与暗影憎恶光束的外围 FX 族（shadow_despawn 等）"
created: "2026-11-04"
updated: "2026-11-04"
type: prefab
summary: "四个暗影阵营外围表现注册：影人消散过渡、多刺舞台之手的影臂与影手、暗影憎恶的持续光束；差异集中在端、父挂、伤害归属与清理。"
aliases: ["shadow_despawn", "stageusher_attackarm", "stageusher_attackhand", "shrouden_optic_blast_fx", "影人消散", "暗影消散特效", "舞台影手", "舞台影臂", "多刺舞台之手影手", "暗影憎恶光束", "外围 FX 族"]
tags: [prefab, fx, server-client]
size_class: complex
dst_build: "steam-buildid:26335394"
sources: ["prefabs/shadowwaxwell.lua", "prefabs/stageusher.lua", "prefabs/shrouden_optic_blast.lua", "prefabs/shadow_trap.lua", "prefabs/rabbit.lua", "prefabs/bunnyman.lua", "prefabs/monkey.lua", "prefabs/clockwork_common.lua", "prefabs/daywalker_pillar.lua", "prefabs/moonbase.lua", "prefabs/terrarium.lua", "prefabs/quagmire_book_shadow.lua", "prefabs/shrouden.lua", "stategraphs/SGshadowwaxwell.lua", "stategraphs/SGstageusher.lua", "stategraphs/SGshrouden.lua", "stategraphs/SGshadow_trap.lua", "components/stretcher.lua", "components/combat.lua", "components/spdamageutil.lua", "aoeutil.lua", "entityscript.lua", "tuning.lua", "prefablist.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 暗影消散、舞台影手与暗影憎恶光束的外围 FX 族（shadow_despawn 等）

## 快速判断

- 职责与适用：三个文件里的四个外围注册，全部联网并 `SetPristine`，玩法组件只在主模拟。`shadow_despawn` 是影人／强制梦魇化过渡的纯 FX（`FX`／`NOCLICK`，播完自删）；`stageusher_attackarm` 是纯 FX 的拉伸影臂；`stageusher_attackhand` 是混合项（移动碰撞＋`GetAttacked` 伤害＋理智光环）；`shrouden_optic_blast_fx` 也是混合项（weapon＋planardamage＋Light＋Follower＋本地烟雾），不是纯视觉。[消散工厂](../../../raw/current/scripts/prefabs/shadowwaxwell.lua#L632-L661)、[影臂](../../../raw/current/scripts/prefabs/stageusher.lua#L518-L548)、[影手](../../../raw/current/scripts/prefabs/stageusher.lua#L462-L515)、[光束](../../../raw/current/scripts/prefabs/shrouden_optic_blast.lua#L411-L481)
- 优先使用：查“影人消散／强制梦魇化过渡”选 `shadow_despawn`；查“多刺舞台之手的影臂／影手”选另两名；查“暗影憎恶光束／位面光束”选 `shrouden_optic_blast_fx`。同批暗影秘典的玩法对象另见 [[reference/prefabs/shadow_pillar]]、[[reference/prefabs/shadow_trap]]；同帧结伴生成的地面痕见 [[reference/prefabs/shadow_glob_fx]]。
- 重要副作用：`stageusher_attackarm` 不自删、不自己播动画，动画与拉伸目标都由生成者 `stageusher` 写入；`stageusher_attackhand` 的 80 点伤害默认由宿主 `combat:DoAttack` 结算（无宿主时才直接 `GetAttacked`），并带 `-TUNING.SANITYAURA_MED` 光环；`shrouden_optic_blast_fx` 以武器身份参与施法者 `DoAttack`，直接伤害 0、位面 5，与施法者自身 35 位面叠加为结算前的 40，且每帧 AOE／Work／TossItems。[测试](../../../raw/current/scripts/prefabs/stageusher.lua#L393-L434)、[武器与位面](../../../raw/current/scripts/prefabs/shrouden_optic_blast.lua#L462-L467)、[结算](../../../raw/current/scripts/aoeutil.lua#L121-L129)
- 修改边界：只覆盖四个工厂、端分界、生成点、父挂、伤害归属与清理。`shadowwaxwell.lua` 同文件的 11 个 shadow 玩法注册只辨析不展开；宿主 AI、动画包实际帧数与客户端观感、专服与晚加入时序未核验。

## 注册与构造

四个注册都是 `Prefab(注册名, 工厂[, assets[, prefabs]])` 的直接返回，没有 `MakeX` 包装；`shadow_despawn` 与同文件 11 个玩法注册同批返回。`shadow_despawn` 刻意不走 `fx.lua` 通用工厂（源码注释：需与动画／实体移除紧密同步）；`shrouden_optic_blast_fx` 的 `weapon` 标签写进 Pristine 前置状态是源码注明的优化，真正的 `weapon` 组件只在主模拟添加。[返回](../../../raw/current/scripts/prefabs/shadowwaxwell.lua#L665-L678)、[注释](../../../raw/current/scripts/prefabs/shadowwaxwell.lua#L626-L626)、[Pristine 优化](../../../raw/current/scripts/prefabs/shrouden_optic_blast.lua#L439-L445)

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `shadow_despawn`（`prefabs/shadowwaxwell.lua`） | `despawn_fx_fn`：Transform／AnimState／SoundEmitter／Network → `FX`＋`NOCLICK` → bank／build `statue_ruins_fx`、播 `transform_nightmare`、`SetMultColour(1,1,1,.5)`、点采样 → `SetPristine` → 非主模拟早退；主模拟 0 帧播 `dontstarve/maxwell/shadowmax_despawn`、监听 `animover` 自删、`persists=false`。 | 无父挂、无 Follower、无玩法组件，也不做缩放或平台处理；同一动画在别处也作 `statue_transition` 用，但本注册有自己的音效并可被调用方挂到平台。[工厂](../../../raw/current/scripts/prefabs/shadowwaxwell.lua#L632-L661)、[平台说明](../../../raw/current/scripts/prefabs/monkey.lua#L237-L240) |
| `stageusher_attackarm`（`prefabs/stageusher.lua`） | `armfn`：Transform／AnimState／Network → `NOCLICK`＋`FX` → bank／build `stagehand_sts_arm`、`OnGround`、`LAYER_BACKGROUND`、排序 3 → `SetPristine` → 非主模拟早退；主模拟加 `stretcher`（resting 2.5、width ratio 0.1）、`persists=false`。 | 工厂**不播动画、不设拉伸目标、不清理**；`arm_loop1..3` 与 `SetStretchTarget(hand)` 由影手生成时写入，影手移除时经 `onremove` 连带删除。[工厂](../../../raw/current/scripts/prefabs/stageusher.lua#L518-L548)、[生成](../../../raw/current/scripts/prefabs/stageusher.lua#L286-L307) |
| `stageusher_attackhand`（`prefabs/stageusher.lua`） | `handfn`：Transform／AnimState／SoundEmitter／Network＋`MakeCharacterPhysics(10,.5)`＋`RemovePhysicsColliders` → `ignorewalkableplatforms`／`NOCLICK`／`shadowhand`／`shadow_aligned` → 同 bank／build、`hand_in`＋`hand_in_loop` → `SetPristine` → 非主模拟早退；主模拟加 `sanityaura`（`-TUNING.SANITYAURA_MED`）、owner／target 的 `onremove` 回调、导出 `SetOwner`／`SetCreepTarget`、`persists=false`。 | 混合：移动＋范围伤害检测＋光环。无 `combat`／`weapon` 组件，所以不能被战斗系统选为目标；伤害归属见“挂接、伤害归属与清理”。[工厂](../../../raw/current/scripts/prefabs/stageusher.lua#L462-L515) |
| `shrouden_optic_blast_fx`（`prefabs/shrouden_optic_blast.lua`） | `fn`：`can_offset_sort_pos` 首标签 → Transform／AnimState／SoundEmitter／Light／Follower／Network；红光 intensity .5／falloff .95／初始关闭 → bank／build `shrouden_blast`、`beam_pre`、alpha .8、`SetLightOverride(1)`、排序偏移 2 → `FX`＋`NOCLICK`＋`weapon`（Pristine 优化） → net_bool `smoke`／`wide`、`SetPrefabNameOverride("shrouden")`、`updatelooper` → `SetPristine`；客户端只监听两个 dirty 事件后早退；主模拟加 `weapon`（0）与 `planardamage`（40−35＝5）、22 帧循环音任务、`animover` 起 AOE、导出 `InitBlast`／`MakeWide`／`KillFx`、`persists=false`。 | 混合：持续光束＋AOE＋Work／Toss＋击退反馈＋Light＋烟雾子实体；`wide` 决定半径 5／2 与 wide 动画；烟雾是客户端本地非联网池化实体；`SetPrefabNameOverride("shrouden")` 注释为 death announce。[工厂](../../../raw/current/scripts/prefabs/shrouden_optic_blast.lua#L411-L481)、[烟雾](../../../raw/current/scripts/prefabs/shrouden_optic_blast.lua#L106-L133) |

名称：四个注册名在简中 `.po` 均无 `STRINGS.NAMES.*` 键；标题中文是已核验的效果描述，不是官方译名。实际命中的官方名都属于宿主或名称覆写：多刺舞台之手（`STRINGS.NAMES.STAGEUSHER`＝“Prickly Stagehand”，[po](../../../raw/current/scripts/languages/chinese_s.po#L349979-L349982)）、暗影憎恶（`STRINGS.NAMES.SHROUDEN`＝“Umbral Abomination”，光束用 `SetPrefabNameOverride("shrouden")` 指向它，[po](../../../raw/current/scripts/languages/chinese_s.po#L349359-L349362)）、暗影傀儡（`STRINGS.NAMES.SHADOWWAXWELL`，同文件 7 个 minion 覆写为它，[po](../../../raw/current/scripts/languages/chinese_s.po#L349239-L349242)）。覆写本身只改显示名与 `nameoverride`，不改变 prefab 实体。[覆写实现](../../../raw/current/scripts/entityscript.lua#L846-L850) 因此“暗影憎恶光束”是效果描述，不能当成该 FX 的官方名。

### 同文件的 11 个 shadow 单位注册（只辨析）

`shadowwaxwell.lua` 末尾还返回 7 个 `MakeMinion`（`shadowdancer`、`shadowworker`、`shadowprotector`、`shadowduelist`（注释 DEPRECATED、保留给模组）、`shadowlumber`、`shadowminer`、`shadowdigger`）与 4 个 `MakeBuilder` 代理（`shadowduelist_builder`、`shadowlumber_builder`、`shadowminer_builder`、`shadowdigger_builder`），合计 11 个玩法注册。7 个 minion 是玩法实体，统一 `SetPrefabNameOverride("shadowwaxwell")` 并挂 `SGshadowwaxwell`；4 个 builder 是只带 Transform 的非联网代理，0 帧兜底自删并把召唤交给 petleash，不是 FX。它们的召唤、配方与 AI 不在本页范围；本页只涉及其消散路径：`SGshadowwaxwell` 在死亡动画结束或 `quickdespawn` 时调用 `DoDespawnFX`。[注册表](../../../raw/current/scripts/prefabs/shadowwaxwell.lua#L665-L678)、[minion 工厂](../../../raw/current/scripts/prefabs/shadowwaxwell.lua#L439-L574)、[builder 工厂](../../../raw/current/scripts/prefabs/shadowwaxwell.lua#L596-L622)

## 关联机制导航

以下生成点按完整注册名在 `raw/current/scripts` 全域反查；依赖声明与实际生成分开列，未展开动态拼接名。

### `shadow_despawn` 的生成点与父挂

| 生成点 | 端、阶段与条件 | 父挂与位置 | 证据 |
|---|---|---|---|
| `SGshadowwaxwell` 的 `death`／`quickdespawn` | 主模拟；死亡动画 `animover`，或健康组件缺失时直接进 `quickdespawn`；同帧另生成一枚 `shadow_glob_fx`（缩放 ±1.3）。 | `shadow_despawn` 平台时父挂平台、平台 `onremove` 转世界坐标并解父；同帧的 `shadow_glob_fx` 也父挂平台但没有解父监听（随沉船消失），否则世界坐标。 | [DoDespawnFX](../../../raw/current/scripts/stategraphs/SGshadowwaxwell.lua#L32-L48)、[quickdespawn](../../../raw/current/scripts/stategraphs/SGshadowwaxwell.lua#L327-L337)、[death](../../../raw/current/scripts/stategraphs/SGshadowwaxwell.lua#L572-L577) |
| `shadow_trap:TryTrapTarget` | 主模拟；每个首次命中的可恐慌目标一枚（`targets` 去重），随后按目标类型推噩梦状态或 `Panic`。 | 平台父挂＋`onremove` 解父；否则世界坐标。 | [生成](../../../raw/current/scripts/prefabs/shadow_trap.lua#L150-L161)、[后续](../../../raw/current/scripts/prefabs/shadow_trap.lua#L162-L169) |
| `rabbit`／`bunnyman`／`monkey` 的强制梦魇化 | 主模拟；`forcenightmare` 定时到期或 `ms_forcenightmarestate`；`isnightmare=true`（影陷阱已生成）时跳过，避免重复。 | 平台父挂＋`onremove` 解父；否则世界坐标。 | [rabbit](../../../raw/current/scripts/prefabs/rabbit.lua#L50-L71)、[bunnyman](../../../raw/current/scripts/prefabs/bunnyman.lua#L49-L70)、[monkey](../../../raw/current/scripts/prefabs/monkey.lua#L237-L262)、[定时调用](../../../raw/current/scripts/prefabs/rabbit.lua#L157-L157)、[事件调用](../../../raw/current/scripts/prefabs/bunnyman.lua#L139-L139) |
| `clockwork_common:OnBecomeShadowChess` | 主模拟；发条怪变为暗影棋子时。 | 平台父挂＋`onremove` 解父；否则世界坐标（rook 会先把 y 抬高 1.5 再生成消散；同帧 `statue_transition_2` 仍在原 y）。 | [工厂](../../../raw/current/scripts/prefabs/clockwork_common.lua#L179-L201)、[调用](../../../raw/current/scripts/prefabs/clockwork_common.lua#L211-L211) |
| `daywalker_pillar` 的囚犯移除 | 主模拟；柱子仍 `persists` 且未睡眠时侵蚀收尾。 | 世界坐标，不父挂。 | [L723-L742](../../../raw/current/scripts/prefabs/daywalker_pillar.lua#L723-L742) |
| `moonbase` 的腐化王杖复原 | 主模拟；`mooncharge` 计时结束且持有腐化王杖。 | `(x, y + 2, z)` 世界坐标，不父挂。 | [L459-L466](../../../raw/current/scripts/prefabs/moonbase.lua#L459-L466) |
| `terrarium` 的猩红状态 | 主模拟；放入物品进入猩红、首领战结束退出猩红。 | 世界坐标，不父挂。 | [进入](../../../raw/current/scripts/prefabs/terrarium.lua#L407-L411)、[结束](../../../raw/current/scripts/prefabs/terrarium.lua#L295-L303) |
| `SGshadow_trap` 的 `dispell` | 主模拟；驱散状态。 | 父挂陷阱实体，未写位置与解父；陷阱 1 秒后 `Remove`。 | [L199-L226](../../../raw/current/scripts/stategraphs/SGshadow_trap.lua#L199-L226) |

### 影手、影臂与光束的生成点

| 生成点 | 涉及注册 | 端、阶段与条件 | 证据 |
|---|---|---|---|
| `SGstageusher` 的 `attack` 帧 15 | `stageusher_attackhand` | 主模拟；目标有效且 `StartAttackingTarget` 返回真时生成一枚；位置为宿主向目标 0.5。 | [状态](../../../raw/current/scripts/stategraphs/SGstageusher.lua#L147-L157)、[生成](../../../raw/current/scripts/prefabs/stageusher.lua#L72-L92) |
| `stageusher_attackhand:SetCreepTarget`／`start_new_creep` | `stageusher_attackarm` | 主模拟；初始 1 条，之后每次到达再 1 条，正常一轮 6 条；`arm_loop1..3` 轮播。 | [初始](../../../raw/current/scripts/prefabs/stageusher.lua#L436-L458)、[续爬](../../../raw/current/scripts/prefabs/stageusher.lua#L318-L336)、[步进](../../../raw/current/scripts/prefabs/stageusher.lua#L374-L384) |
| `SGshrouden` 的 `optic_blast_appear` | `shrouden_optic_blast_fx` | 主模拟；传送到目标位置后生成并 `InitBlast(inst)`；appear 动画结束时若 Boss 已死亡则先 `fx:Remove()`，未转入 loop 的退场也会 Remove。 | [生成](../../../raw/current/scripts/stategraphs/SGshrouden.lua#L2112-L2115)、[死亡分支](../../../raw/current/scripts/stategraphs/SGshrouden.lua#L2139-L2147)、[退场](../../../raw/current/scripts/stategraphs/SGshrouden.lua#L2152-L2166) |
| `SGshrouden` 的 `optic_blast_loop` | `shrouden_optic_blast_fx` | 主模拟；`data.fx` 无效时兜底再生成一枚，否则复用 appear 的实例。 | [L2188-L2193](../../../raw/current/scripts/stategraphs/SGshrouden.lua#L2188-L2193) |
| `SGshrouden` 的 `optic_blast_loop` 时间线＋`shrouden` 阶段 | 同一注册的 `MakeWide`／`KillFx` | 主模拟；帧 75 非 wide 直接 `KillFx`；可 wide 时 `MakeWide`；帧 120 `KillFx`；`canwideblast` 仅在 HP≤1/3 阶段为真；`resetboss` 直接 `Remove`。 | [时间线](../../../raw/current/scripts/stategraphs/SGshrouden.lua#L2261-L2288)、[阶段](../../../raw/current/scripts/prefabs/shrouden.lua#L583-L590)、[resetboss](../../../raw/current/scripts/prefabs/shrouden.lua#L665-L665) |

### 挂接、伤害归属与清理

| 注册名 | 父挂／跟随 | 伤害与玩法归属 | 清理路径与证据 |
|---|---|---|---|
| `shadow_despawn` | 无内建父挂；平台／世界坐标由每个生成点自己决定（见上表）。 | 无伤害、无光环，也不参与攻击结算。 | 主模拟 `animover` 自删，`persists=false`，无兜底；客户端依赖网络移除。[清理](../../../raw/current/scripts/prefabs/shadowwaxwell.lua#L651-L658) |
| `stageusher_attackarm` | 不 `SetParent`；`stretcher:SetStretchTarget(hand)` 逐帧朝向并拉伸（resting 2.5、宽比 0.1）。 | 纯 FX，无伤害、无光环；拉伸目标失效时组件自行停止更新。 | 自身无 `animover`、无 lifetime；影手 `onremove` 时删除；影手正常消散后连带收尾。[拉伸](../../../raw/current/scripts/components/stretcher.lua#L16-L50)、[删除监听](../../../raw/current/scripts/prefabs/stageusher.lua#L298-L299) |
| `stageusher_attackhand` | 不 `SetParent`，世界坐标移动；无 Follower。 | 正常路径走 `owner.components.combat:DoAttack(target)`（owner 已设 `ignorehitrange`），每 4 帧在半径 2 内检测，同目标 25 帧内不重复；无 owner 时 `target.components.combat:GetAttacked(hand, 80)`。光环 `-TUNING.SANITYAURA_MED`。 | 目标移除即停伤害并清 `updatelooper`；owner 移除即 `Remove`；到达步数耗尽后 `grab`→`hand_scare`→`animover` 自删，同时让影臂播 `arm_scare1..4`。[检测](../../../raw/current/scripts/prefabs/stageusher.lua#L393-L434)、[回调](../../../raw/current/scripts/prefabs/stageusher.lua#L496-L504)、[光环](../../../raw/current/scripts/prefabs/stageusher.lua#L492-L493)、[参数](../../../raw/current/scripts/tuning.lua#L6880-L6886) |
| `shrouden_optic_blast_fx` | `InitBlast` 里 `Follower:FollowSymbol(caster.GUID, "optic_blast_fx_follow")`；自身不 `SetParent`。 | 武器 0＋位面 5，经施法者 `combat:DoAttack` 与施法者自身 35 位面合并为结算前 40；施法者存活时每帧 `AOEUtil.Work／Attack／TossItems`，同目标 0.25 秒重复；`onattackother` 在 2 秒内累计 3 次即击退并伴随颜色闪烁；红光；烟雾为客户端本地。 | `KillFx` 睡眠中直接 `Remove`，否则播 pst 后 `animover` 自删；`resetboss` 直接 `Remove`；施法者死亡／失效时 AOE 更新被移除但实体保留到收尾；`persists=false`。[InitBlast](../../../raw/current/scripts/prefabs/shrouden_optic_blast.lua#L353-L367)、[AOE](../../../raw/current/scripts/prefabs/shrouden_optic_blast.lua#L200-L213)、[KillFx](../../../raw/current/scripts/prefabs/shrouden_optic_blast.lua#L303-L322) |

位面叠加的读法：`CalcDamage` 在带武器时先取武器伤害与武器 `planardamage`（5），再用 `SpDamageUtil.CollectSpDamage` 把攻击者自身的 `planardamage`（35）并入同一张位面表，得到 40；随后才经目标 `planardefense`／倍率与 PvP 修正。直接伤害取武器的 `SetDamage(0)`，原版施法者也没有另设 `combat.damagebonus`，所以物理部分为 0；自定义施法者若设了 `damagebonus` 会叠加到直接伤害上。[武器路径](../../../raw/current/scripts/components/combat.lua#L871-L964)、[位面定义](../../../raw/current/scripts/components/spdamageutil.lua#L23-L25)、[施法者位面](../../../raw/current/scripts/prefabs/shrouden.lua#L1409-L1410)、[数值](../../../raw/current/scripts/tuning.lua#L9719-L9734)

### 资源与关联 Prefab

| 类型／对象 | 本页如何使用 | 条件与边界 | Wiki／源码入口及证据 |
|---|---|---|---|
| 资源 | `anim/statue_ruins_fx.zip`（消散）、`anim/stagehand_sts_arm.zip`（影手／影臂共用）、`anim/shrouden_blast.zip`（光束及烟雾）；音效为 `dontstarve/maxwell/shadowmax_despawn`、`dontstarve/sanity/shadowhand_creep`、`rifts8/shrouden/opticblast_*`。 | 只核验声明与播放点，未解包帧数、循环标记与观感；不分发资源。 | [消散资源](../../../raw/current/scripts/prefabs/shadowwaxwell.lua#L1-L4)、[影手资源](../../../raw/current/scripts/prefabs/stageusher.lua#L10-L13)、[光束资源](../../../raw/current/scripts/prefabs/shrouden_optic_blast.lua#L1-L6) |
| 关联 FX：`shadow_glob_fx` | `SGshadowwaxwell` 的 `DoDespawnFX` 与消散同帧生成一枚，缩放 ±1.3；影陷阱也用它布地面痕。 | 该页维护本体、涟漪与去重标签；本页只记共同生成关系。 | [[reference/prefabs/shadow_glob_fx]]；[同帧生成](../../../raw/current/scripts/stategraphs/SGshadowwaxwell.lua#L32-L48) |
| 关联 Prefab：`shadow_pillar`／`shadow_trap` | 同属暗影秘典法术链：影柱是玩法柱体系，影陷阱会生成 `shadow_despawn` 与 `shadow_glob_fx`。 | 玩法、施法与定身归各自页面；本页只共用消散表现与生成点。 | [[reference/prefabs/shadow_pillar]]、[[reference/prefabs/shadow_trap]]；[影陷阱生成](../../../raw/current/scripts/prefabs/shadow_trap.lua#L150-L161) |
| 依赖与预载 | `stageusher.lua` 把两个影手／影臂写进 `prefabs` 表；`shrouden.lua` 预载 `shrouden_optic_blast_fx`；`stageusher` 进 `prefablist.lua`。此外 `quagmire_book_shadow.lua` 仅声明 `shadow_despawn` 而不生成。 | 依赖声明不等于生成；`prefablist.lua` 另有一条 `shrouden_optic_blast`（无 `_fx`），本快照没有同名注册。 | [影手依赖](../../../raw/current/scripts/prefabs/stageusher.lua#L15-L18)、[光束依赖](../../../raw/current/scripts/prefabs/shrouden.lua#L10-L13)、[宿主预载](../../../raw/current/scripts/prefablist.lua#L1224-L1224)、[陈旧条目](../../../raw/current/scripts/prefablist.lua#L1150-L1150)、[仅声明](../../../raw/current/scripts/prefabs/quagmire_book_shadow.lua#L7-L11) |

## 典型接入

### 在平台／陆地安全地放一枚影人消散

- 前提与选择：宿主是服务端实体，需要“影人消散／强制梦魇化”的过渡表现；`shadow_despawn` 是纯 FX，不携带玩法，原版在实体位置或状态图死亡帧生成。
- 生效与撤销：`animover` 自删，`persists=false`，不跨存档；平台上的实例按原版模式父挂平台，平台 `onremove` 时先写回世界坐标再解父，避免随沉船一起消失。
- 原版依据：[rabbit 的 DoShadowFx](../../../raw/current/scripts/prefabs/rabbit.lua#L50-L71) 与 [SGshadowwaxwell 的 DoDespawnFX](../../../raw/current/scripts/stategraphs/SGshadowwaxwell.lua#L32-L48)。

```lua
-- 主模拟：宿主位置放消散；平台上的实例随平台，平台消失时落回世界坐标
local function spawn_shadow_despawn(host)
    if not TheWorld.ismastersim then
        return
    end
    local x, y, z = host.Transform:GetWorldPosition()
    local fx = SpawnPrefab("shadow_despawn")
    local platform = host:GetCurrentPlatform()
    if platform ~= nil then
        fx.entity:SetParent(platform.entity)
        fx.Transform:SetPosition(platform.entity:WorldToLocalSpace(x, y, z))
        fx:ListenForEvent("onremove", function()
            fx.Transform:SetPosition(fx.Transform:GetWorldPosition())
            fx.entity:SetParent(nil)
        end, platform)
    else
        fx.Transform:SetPosition(x, y, z)
    end
    return fx
end
```

- 调整与验证：颜色、动画、音效与 0 帧音任务都由工厂固定，替换点只有位置与父挂；不要在客户端重复生成。示例只做语法检查，未在游戏内执行。

### 给自定义施法者挂一道暗影憎恶光束

- 前提与选择：需要“持续光束＋范围伤害＋击退反馈”时复用 `shrouden_optic_blast_fx`；施法者必须是有效、存活且带 `health`、`combat`、`GetAOEAttackTagSet()` 与 `optic_blast_fx_follow` 符号的实体。原版由 `SGshrouden` 的 appear／loop 状态生成并 `InitBlast(caster)`。
- 生效与撤销：`InitBlast` 立即读 `caster.components.health:IsDead()`，记录仍存活的施法者、按 `optic_blast_fx_follow` 符号跟随，并在 `resetboss` 时自删；实体自行从 `beam_pre` 推进到 `beam_basic_loop` 后开始 AOE。结束时必须调 `KillFx()`；`MakeWide()` 只在 `beam_basic_loop` 当前动画下生效。
- 原版依据：[SGshrouden 生成](../../../raw/current/scripts/stategraphs/SGshrouden.lua#L2112-L2115)、[InitBlast](../../../raw/current/scripts/prefabs/shrouden_optic_blast.lua#L353-L367)、[KillFx 调用](../../../raw/current/scripts/stategraphs/SGshrouden.lua#L2261-L2268)。

```lua
-- 主模拟：施法者需有效、存活，且有 health、combat、GetAOEAttackTagSet() 与 optic_blast_fx_follow 符号
local function start_optic_blast(caster)
    if not TheWorld.ismastersim then
        return
    end
    local fx = SpawnPrefab("shrouden_optic_blast_fx")
    fx.Transform:SetPosition(caster.Transform:GetWorldPosition())
    fx:InitBlast(caster)
    return fx
end

-- 结束：播 pst 后 animover 自删；施法者死亡或移除前主动调用
local function stop_optic_blast(fx)
    if fx ~= nil and fx:IsValid() then
        fx:KillFx()
    end
end
```

- 调整与验证：施法者缺 `health` 时 `InitBlast` 立即报错；缺 `combat` 时 `AOEUtil.Attack` 在 `ignorehitrange`／`CanTarget`／`DoAttack` 处报错；缺 `GetAOEAttackTagSet()` 时 `UpdateBeamAOE` 每次更新报错。没有 35 位面的施法者只能打出 5 位面；符号缺失时跟随行为未核验。示例只做语法检查，未在游戏内执行。[InitBlast](../../../raw/current/scripts/prefabs/shrouden_optic_blast.lua#L353-L367)、[AOEUtil.Attack](../../../raw/current/scripts/aoeutil.lua#L77-L78)、[结算](../../../raw/current/scripts/aoeutil.lua#L118-L129)

## 常见陷阱与限制

- **误区：`shrouden_optic_blast_fx` 名字带 fx 就只是视觉。** 它挂 `weapon`＋`planardamage`，每帧 AOE 攻击、`Work`（CHOP／HAMMER／MINE）与 `TossItems`，还会按受击次数把目标击退；复用前要按“玩法载体”评估。[AOE](../../../raw/current/scripts/prefabs/shrouden_optic_blast.lua#L200-L213)、[工作与投掷](../../../raw/current/scripts/aoeutil.lua#L158-L275)
- **边界：光束的 40 位面依赖施法者自身 35 位面。** 武器只提供 5；换到没有 `planardamage` 的施法者身上只剩 5，换到其他 Boss 上会按对方位面数值叠加。[武器与位面](../../../raw/current/scripts/prefabs/shrouden_optic_blast.lua#L462-L467)、[叠加读法](../../../raw/current/scripts/components/combat.lua#L891-L899)
- **边界：`MakeWide` 是静默失败。** 只有当前动画是 `beam_basic_loop` 时才切 wide；在 pre、pst 或已 wide 时调用不做任何事。原版还要求 `canwideblast`，仅在 HP≤1/3 阶段为真。[MakeWide](../../../raw/current/scripts/prefabs/shrouden_optic_blast.lua#L391-L407)、[阶段](../../../raw/current/scripts/prefabs/shrouden.lua#L583-L590)
- **边界：`stageusher_attackarm` 是不能独立使用的实体。** 工厂不播动画、不设 `stretcher` 目标、也不自删；单独 `SpawnPrefab` 会留下一个静止的 FX，只能靠读档（`persists=false`）或手动删除清理。必须由影手生成并设 `arm_loop1..3` 与拉伸目标。[工厂](../../../raw/current/scripts/prefabs/stageusher.lua#L518-L548)、[生成者的设置](../../../raw/current/scripts/prefabs/stageusher.lua#L292-L299)
- **边界：影手的 80 点伤害不属于影手自身。** 正常路径由 owner 的 `combat:DoAttack` 结算（含 owner 的倍率与标签）；只有 owner 缺失时才直接 `GetAttacked(hand, 80)`，此时死亡归因实体是没有 NAMES 键的 `stageusher_attackhand`。owner 移除会立刻删除影手并停掉伤害。[结算](../../../raw/current/scripts/prefabs/stageusher.lua#L426-L431)、[回调](../../../raw/current/scripts/prefabs/stageusher.lua#L496-L504)、[归因读取](../../../raw/current/scripts/components/combat.lua#L669-L678)
- **边界：影手能不能打到其他玩家，取决于原目标是不是玩家。** 目标为玩家时使用 `ARM_TARGETS_CANT`（允许其他玩家），否则用 `ARM_TARGETS_CANT_WITHPLAYER`（排除所有玩家）；两套都排除 `stageusher`、`FX`、`flying`、`ghost` 等，且要求目标带 `_combat`。[过滤表](../../../raw/current/scripts/prefabs/stageusher.lua#L393-L434)
- **边界：`shadow_despawn` 的平台处理没有统一实现。** rabbit／bunnyman／monkey／clockwork／影陷阱的命中目标都父挂平台并在 `onremove` 解父；`daywalker_pillar`、`moonbase`、`terrarium` 直接写世界坐标；`SGshadow_trap` 的 dispell 直接父挂陷阱且不写解父。移植时要按宿主平台语义自选，不能照抄一个调用点就假定通用。[父挂对照](../../../raw/current/scripts/prefabs/shadow_trap.lua#L150-L158)、[不父挂例](../../../raw/current/scripts/prefabs/moonbase.lua#L459-L466)
- **边界：`prefablist.lua` 的 `shrouden_optic_blast` 与注册名不一致。** 本快照全树没有 `shrouden_optic_blast` 注册，真正的预载来自 `shrouden.lua` 的 `prefabs` 表；该陈旧条目在启动时是被忽略还是报错未核验，模组按 `prefablist` 反查时不要据此判断存在性。[条目](../../../raw/current/scripts/prefablist.lua#L1150-L1150)、[真实依赖](../../../raw/current/scripts/prefabs/shrouden.lua#L10-L13)

## 验证记录

- 源码审阅：2026-11-04，steam-buildid:26335394（实读 `raw/current/BUILD.txt`）。核验三个工厂文件的相关段落（`shadowwaxwell.lua` 共 678 行、`stageusher.lua` 552 行、`shrouden_optic_blast.lua` 485 行）、`SGshadowwaxwell` 的消散与死亡状态、`SGstageusher` 的攻击状态、`SGshrouden` 的 appear／loop／pst 与 wide 分支、`shrouden.lua` 的阶段与 `resetboss`、`shadow_trap`／`SGshadow_trap` 的生成点，以及 `stretcher`、`combat`、`spdamageutil`、`aoeutil`、`entityscript`、`tuning`、`prefablist` 的相关接口。
- 反查：全 `raw/current/scripts` 以完整注册名 `grep -rn --include=*.lua`：`shadow_despawn` 28 处命中（11 处生成、8 处依赖声明、8 处注释、1 处自身注册），生成点分布见上表；`stageusher_attackarm` 3 处、`stageusher_attackhand` 3 处（均为依赖表、生成、注册）；`shrouden_optic_blast_fx` 4 处（依赖、注册、SG 两处生成），另有同文件两个 netvar 名以该名为前缀。`SGstageusher` 仅被 `stageusher` 设置；`SGshadowwaxwell` 是 7 个 minion 的共同状态图；未穷举运行时拼接名。
- 名称核验：`languages/chinese_s.po` 定向检索四个注册名均无 `STRINGS.NAMES.*` 键；宿主／覆写名命中 `STAGEUSHER`＝多刺舞台之手、`SHROUDEN`＝暗影憎恶、`SHADOWWAXWELL`＝暗影傀儡。标题使用效果描述，不是官方译名。
- 游戏运行：未运行；`validation` 保持 `source-reviewed`。
- 分档说明：覆盖 3 个宿主族（11 个 shadow 玩法注册的消散链、多刺舞台之手、暗影憎恶）、4 个注册、11 个 `shadow_despawn` 生成点与两套混合伤害结算，需整体对照保留，故 `size_class: complex`。
- 结构检查：`validate_page.py` 通过（0 issue；180 行、约 30.9 KiB，complex 档 400 行／32 KiB 内）；`validate_source_links.py --strict` 通过（101 处源码引用、0 error／0 warning）；两段 Lua 示例经 `parser_probe --syntax-only` 通过（只查语法、未执行，临时文件已删）。
- 未覆盖：三个动画包的实际帧数、循环标记与客户端观感；影手／影臂／光束在游戏内的命中与击退表现；`optic_blast_fx_follow` 符号在非原版施法者动画里的存在性；专服与晚加入的烟雾时序；陈旧 `prefablist` 条目的启动行为。

## 相关链接

- [[reference/prefabs/shadow_pillar|暗影囚牢·影柱法术体系]]（同属暗影秘典；对照“玩法柱＋贴片”与纯消散过渡）
- [[reference/prefabs/shadow_glob_fx|暗影地面痕特效]]（与消散同帧生成的地面痕；含海面涟漪）
- [[reference/prefabs/shadow_trap|暗影陷阱与减速表现]]（生成 `shadow_despawn` 的玩法陷阱）
- [[reference/mechanisms/generic-fx-factory|通用 FX 工厂（fx.lua 注册表）]]（`shadow_despawn` 之外的外围 FX 通用协议；本页四个注册均不走该表）
- [[reference/prefabs/fx-world-index|特效 Prefab 索引·环境与世界]]、[[reference/prefabs/fx-index|特效 Prefab 索引]]

返回 [[reference/prefabs/fx-shadow-index|暗影生物、裂隙生物与附着]]。
