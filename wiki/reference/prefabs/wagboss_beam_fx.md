---
title: "W.A.R.B.O.T. 轨道光束与机械 Boss 战伴随特效族（wagboss_beam_fx 等 4 注册）"
created: "2026-11-05"
updated: "2026-11-05"
type: prefab
summary: "Wagstaff 机械 Boss 战的 4 个伴随注册：轨道打击光束（FX＋Light＋combat/planardamage 的混合载体，伤害由光束实体自身结算）、机械腿超新星灼烧阻挡的地面喷射标记（弹射落地的是腿本体，本 FX 在阻挡时才出现）、终场演出里被拽走的瓦格斯塔夫联网说话体（自带本地白剪影子层）与其伴随的 containment 装置（官方名'月亮精华提取器'，混合载体）。"
aliases: ["wagboss_beam_fx", "wagboss_robot_leg_fx", "wagstaff_npc_finale_fx", "alterguardian_contained", "轨道打击光束", "机械腿灼烧标记", "瓦格斯塔夫剪影", "月亮精华提取器", "战争瓦器人", "W.A.R.B.O.T.", "颗粒状传输", "机械Boss战特效"]
tags: [prefab, fx, creature, server-client]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/wagboss_beam.lua", "prefabs/wagboss_robot_leg.lua", "prefabs/wagstaff_npc.lua", "stategraphs/SGwagboss_robot.lua", "stategraphs/SGalterguardian_phase4_lunarrift.lua", "stategraphs/SGwagstaff_npc.lua", "components/lunarsupernovablocker.lua", "prefabs/wagboss_util.lua", "prefabs/alterguardian_phase3dead.lua", "components/wagpunk_arena_manager.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# W.A.R.B.O.T. 轨道光束与机械 Boss 战伴随特效族（wagboss_beam_fx 等 4 注册）

## 快速判断

- 职责与适用：本页收拢机械 Boss 战相关 3 个工厂文件里的 **4 个注册**：`wagboss_beam_fx` 是 W.A.R.B.O.T.（战争瓦器人）轨道打击的光束——**混合载体**，FX 外观之下带 combat/planardamage 真实结算；`wagboss_robot_leg_fx` 是机械腿阻挡月相超新星时的地面喷射标记（纯本地子层伴随的联网壳）；`wagstaff_npc_finale_fx` 是终场演出中瓦格斯塔夫被天体拽走的联网说话体（本地白剪影子层负责"剪影"观感）；`alterguardian_contained` 是官方名"月亮精华提取器"的伴随装置——混合载体，随瓦格斯塔夫 NPC 的工作流出现、收集、erosion 进出场。[beam 注册](../../../raw/current/scripts/prefabs/wagboss_beam.lua#L492)、[leg_fx 注册](../../../raw/current/scripts/prefabs/wagboss_robot_leg.lua#L267)、[finale 注册](../../../raw/current/scripts/prefabs/wagstaff_npc.lua#L1767)、[contained 注册](../../../raw/current/scripts/prefabs/wagstaff_npc.lua#L1768)
- 本体辨析：`wagboss_robot`（W.A.R.B.O.T.，官方名见 [po](../../../raw/current/scripts/languages/chinese_s.po#L351384-L351387)）、`wagboss_robot_leg`（机械腿本体，落地后成 `lunarsupernovablocker` 阻挡物）、`wagstaff_npc`／`wagstaff_npc_pstboss`（瓦格斯塔夫 NPC）与 `alterguardian_phase4_lunarrift`（月相天体 Boss）是玩法实体，不在本页覆盖。
- 重要副作用：光束**有 6 秒硬寿命**（`DoTaskInTime(6, KillFx)`，[L481](../../../raw/current/scripts/prefabs/wagboss_beam.lua#L479-L481)）；机械腿 FX **无自删**，生命期挂在组件的 source 单槽映射上；`wagstaff_npc_finale_fx` 动画播完会推全局事件 `ms_wagboss_snatched_wagstaff`（[L1583-L1591](../../../raw/current/scripts/prefabs/wagstaff_npc.lua#L1583-L1591)）驱动竞技场管理器移除 NPC。
- 修改边界：只覆盖 4 注册的工厂、生成点、父子/跟随与清理；Boss 战状态机全貌、超新星伤害体系（lunarsupernovaburning）、提取器工作流剧情与竞技场管理器不展开。
- 名称：官方名 `ALTERGUARDIAN_CONTAINED`＝"月亮精华提取器"（[po](../../../raw/current/scripts/languages/chinese_s.po#L338529-L338532)）、宿主 `WAGSTAFF_NPC`＝"颗粒状传输"（[po](../../../raw/current/scripts/languages/chinese_s.po#L351514-L351517)）；`WAGBOSS_BEAM_FX`／`WAGBOSS_ROBOT_LEG_FX`／`WAGSTAFF_NPC_FINALE_FX` 无 `STRINGS.NAMES.*` 键，标题中文为已核验效果描述，**非官方译名**。

## 注册与构造

| 注册名 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `wagboss_beam_fx` | `fn`：Transform／AnimState／SoundEmitter／Light／Network；标签 `FX`＋`NOCLICK`；AnimState `beam_pre`→`beam_loop`＋泛光＋LightOverride 0.3；Light 蓝色（0.01,0.35,1）初始禁用；`animsync` net_bool；`SetPrefabNameOverride("wagboss_robot")`（死亡播报归属）；非专服造本地地面标记子体 | **混合载体**：主模拟加 combat（默认伤害 0）＋planardamage（[TUNING.WAGBOSS_BEAM_PLANAR_DAMAGE＝50](../../../raw/current/scripts/tuning.lua#L8692-L8694)），AOE 由光束自身每帧结算（见导航）。[工厂](../../../raw/current/scripts/prefabs/wagboss_beam.lua#L423-L488)、[结算组件](../../../raw/current/scripts/prefabs/wagboss_beam.lua#L472-L477) |
| `wagboss_robot_leg_fx` | `fxfn`：Transform／AnimState／Network；标签 `FX`＋`NOCLICK`；bank/build `wagboss_leg`、`ground_spray` 循环、贴地＋`LAYER_BACKGROUND` 排序 3＋泛光；非专服加本地子体 `CreateFx2`（`wagboss_lunar_blast/supernova_hit_large`，偏移 0.6）；主模拟仅 `persists=false` | **无自删、无净变量**：方向/显隐全由宿主组件每帧写入；子体与壳都是纯视觉。[工厂](../../../raw/current/scripts/prefabs/wagboss_robot_leg.lua#L228-L262)、[本地子体](../../../raw/current/scripts/prefabs/wagboss_robot_leg.lua#L206-L226) |
| `wagstaff_npc_finale_fx` | `finale_fn`：Transform／AnimState／SoundEmitter／Follower／Network；六向；bank `wilson`、build `wagstaff`、播 `wagstaff_finale`→`wagstaff_finale2`；遮 face/帽子符号并覆盖 `fx_white`；erosion 参数 -1.0（初始隐藏）；talker（chatter）＋npc_talker；非专服造本地剪影子体 | 联网体是"被拽走的瓦格斯塔夫"本体层（可说话、可 Materialize/Brighten）；**剪影观感来自本地子体** `finale_CreateSilhouette`（build `wagstaff_finale`、Hide bgwhite、FinalOffset 1）。[工厂](../../../raw/current/scripts/prefabs/wagstaff_npc.lua#L1593-L1639)、[剪影子体](../../../raw/current/scripts/prefabs/wagstaff_npc.lua#L1559-L1575) |
| `alterguardian_contained` | `alterguardian_containedfn`：Transform／AnimState／SoundEmitter／Light／Network；MakeInventoryPhysics(50,.5)；bank/build `alterguardian_contained`、`idle`、erosion -1.0；Light 橙色（255,179,107）初始禁用；主模拟 `persists=false`＋lootdropper＋inspectable＋`erode` 方法，听 `docollect` 事件 | **混合载体**：装置带掉落与光照（收集时亮灯、冬熟节掉 Boss 挂饰），可见状态全由动画与 erosion 事件驱动，无战斗组件。[工厂](../../../raw/current/scripts/prefabs/wagstaff_npc.lua#L1687-L1716)、[docollect](../../../raw/current/scripts/prefabs/wagstaff_npc.lua#L1676-L1685) |

## 关联机制导航

### wagboss_beam_fx 的伤害结算归属

光束生成于 Boss 的轨道打击：`signal_loop` 状态帧事件在 `canorbitalstrike` 且冷却（[24–30 秒](../../../raw/current/scripts/tuning.lua#L8701-L8701)）就绪时调 `TryOrbitalStrike`（[SGwagboss_robot L1907-L1911](../../../raw/current/scripts/stategraphs/SGwagboss_robot.lua#L1907-L1911)→[L395-L406](../../../raw/current/scripts/stategraphs/SGwagboss_robot.lua#L395-L406)）：从 `grouptargeter`/战斗目标里按竞技场内外范围（场外 [18 格](../../../raw/current/scripts/stategraphs/SGwagboss_robot.lua#L279-L279)）选目标，最多 `min(6, 目标数/2+1)` 个（[L337](../../../raw/current/scripts/stategraphs/SGwagboss_robot.lua#L337-L337)），每个目标一束，`TrackTarget` 让光束在 `beam_pre` 期间向目标滑移（[TrackTarget](../../../raw/current/scripts/prefabs/wagboss_beam.lua#L100-L120)）。

| 结算分支 | 行为（伤害归属） | 证据 |
|---|---|---|
| 可工作物（CHOP/DIG/HAMMER/MINE/NPC_workable） | `workable:Destroy(inst)`——**破坏归属记在光束实体**；树桩直接移除 | [L193-L201](../../../raw/current/scripts/prefabs/wagboss_beam.lua#L193-L201) |
| 可采集物 | `pickable:Pick(inst)`——采集归属光束 | [L202-L204](../../../raw/current/scripts/prefabs/wagboss_beam.lua#L202-L204) |
| brightmare 类（敌对方） | 回血 `WAGBOSS_BEAM_BRIGHTMARE_HEAL＝5` 并登记月灼源 | [L206-L211](../../../raw/current/scripts/prefabs/wagboss_beam.lua#L206-L211) |
| 有 health 目标·首帧 | `inst.components.combat:DoAttack(v)`——首击走光束的 combat，**0 物理＋50 平面伤害** | [L214-L215](../../../raw/current/scripts/prefabs/wagboss_beam.lua#L205-L234) |
| 有 health 目标·持续 | `health:DoDelta(-CalcLunarBurnTickDamage(v, 40 DPS))`——按帧 tick，每 12 tick 一次可见"脉冲"（overtime=false），并登记 `RegisterLunarBurnSource`；骑乘目标坐骑同步扣血；同时 `MaximizeGrogginess` | [L216-L230](../../../raw/current/scripts/prefabs/wagboss_beam.lua#L205-L234)、[伤害公式](../../../raw/current/scripts/prefabs/wagboss_util.lua#L114-L128) |
| 掉落物 | 停用物理后 `TossLaunch` 弹飞 | [L235-L241](../../../raw/current/scripts/prefabs/wagboss_beam.lua#L235-L241) |

距离驱动的白色加色按 `Remap` 计算并双向去重（epic×0.4、largecreature×0.6，[L268-L287](../../../raw/current/scripts/prefabs/wagboss_beam.lua#L268-L287)），退场时 0.75 秒淡出（[UpdateColouredFade](../../../raw/current/scripts/prefabs/wagboss_beam.lua#L313-L341)）。**伤害归属于光束，不归属于 Boss**：死亡播报经 `SetPrefabNameOverride` 记回 `wagboss_robot`（[L449](../../../raw/current/scripts/prefabs/wagboss_beam.lua#L449)）。

### wagboss_robot_leg_fx 的生成时机（Boss 战阶段）

机械腿不是普通落点装饰：W.A.R.B.O.T. 的 `death` 状态把腿本体弹射出去（[SGwagboss_robot L2041-L2043](../../../raw/current/scripts/stategraphs/SGwagboss_robot.lua#L2041-L2043)），腿落地后挂 `lunarsupernovablocker`（[MakeLandedAtXZ L43-L68](../../../raw/current/scripts/prefabs/wagboss_robot_leg.lua#L43-L68)；落地判定与 `foot_land` 落地音在 [L70-L106](../../../raw/current/scripts/prefabs/wagboss_robot_leg.lua#L70-L106)）。随后月相天体 Boss 施放超新星时，对范围内带 `lunarsupernovablocker` 的实体逐个 `AddSource`（[SGalterguardian_phase4_lunarrift L226-L228](../../../raw/current/scripts/stategraphs/SGalterguardian_phase4_lunarrift.lua#L198-L240)）；组件为**每个灼烧源**生成一枚 `wagboss_robot_leg_fx` 子体挂到腿上，按源方向 `atan2` 旋转、同点位则隐藏（[AddSource L27-L46](../../../raw/current/scripts/components/lunarsupernovablocker.lua#L27-L46)）。台账所记"落地喷雾"与源码不符：**落地只有音效，本 FX 是超新星灼烧期间的阻挡标记**。

### wagstaff_npc_finale_fx 与 alterguardian_contained 的载体链

- 终场：月相天体 Boss 的 `finale` 状态生成 finale_fx 并 `AttachToAlter`（父挂 Boss＋`FollowSymbol(player_follow)`，[SG L1563-L1564](../../../raw/current/scripts/stategraphs/SGalterguardian_phase4_lunarrift.lua#L1548-L1596)）；时间线驱动它说话、`Materialize`（erosion 显形 2 秒）、`Brighten`（加白 2 秒）（[Materialize/Brighten](../../../raw/current/scripts/prefabs/wagstaff_npc.lua#L1466-L1493)）。`animover` 时若 Boss 状态仍在等它，推 `ms_wagboss_snatched_wagstaff`，竞技场管理器监听后移除世界上的瓦格斯塔夫（[wagpunk_arena_manager L1905](../../../raw/current/scripts/components/wagpunk_arena_manager.lua#L1905-L1905)）。
- 客户端：`OnEntityReplicated` 复制父挂与跟随，剪影子体也挂到 Boss 上并双双登记 `highlightchildren`（[L1507-L1516](../../../raw/current/scripts/prefabs/wagstaff_npc.lua#L1507-L1534)、[AttachToAlter](../../../raw/current/scripts/prefabs/wagstaff_npc.lua#L1519-L1534)）；客户端 PostUpdate 把剪影动画对齐到本体 `wagstaff_finale2`（[L1551-L1556](../../../raw/current/scripts/prefabs/wagstaff_npc.lua#L1551-L1556)）。
- 提取器：瓦格斯塔夫 NPC（pstboss）经 `spawndevice` 事件在身旁 2 格生成装置并同步 erosion（[spawn_device L594-L605](../../../raw/current/scripts/prefabs/wagstaff_npc.lua#L594-L605)、[erode 同步 L580-L591](../../../raw/current/scripts/prefabs/wagstaff_npc.lua#L580-L591)）；推送点三处——重定位 [L699](../../../raw/current/scripts/prefabs/wagstaff_npc.lua#L671-L701)、读档 [L726](../../../raw/current/scripts/prefabs/wagstaff_npc.lua#L722-L731)、phase3 残骸处 [alterguardian_phase3dead L127-L136](../../../raw/current/scripts/prefabs/alterguardian_phase3dead.lua#L127-L136)。NPC 的 `capture_appearandwork` 超时后对目标推 `orbtaken`、对装置推 `docollect`（[SGwagstaff_npc L433-L442](../../../raw/current/scripts/stategraphs/SGwagstaff_npc.lua#L433-L442)）——装置亮灯播 `collect`，冬熟节额外弹出 `winter_ornament_boss_wagstaff`（[L1676-L1685](../../../raw/current/scripts/prefabs/wagstaff_npc.lua#L1676-L1685)）。

### 清理路径

| 注册 | 清理方式 | 边界与证据 |
|---|---|---|
| `wagboss_beam_fx` | 6 秒 `KillFx`：播 `beam_pst`＋地面标记 `ground_marker_pst`，`animover` 与 `OnEntitySleep` 双兜底自删，注销全部月灼源、停循环音（[KillFx L393-L419](../../../raw/current/scripts/prefabs/wagboss_beam.lua#L393-L419)） | `persists=false`；本地标记子体随壳级联；亮度淡出 0.75 秒独立收尾。 |
| `wagboss_robot_leg_fx` | 无自删：`RemoveSource` 时 `fx:Remove()`；腿本体被移除（Boss 消亡 `KillMe` 或被挖）则级联 | source 是组件上的映射表，源失效/离开范围/停止灼烧都会摘除（[RemoveSource](../../../raw/current/scripts/components/lunarsupernovablocker.lua#L57-L67)、[OnUpdate](../../../raw/current/scripts/components/lunarsupernovablocker.lua#L81-L106)）。 |
| `wagstaff_npc_finale_fx` | 无自删钩子：`finale_OnRemoveEntity` 摘剪影子体与 highlightchildren 登记；Boss 移除级联（[L1495-L1505](../../../raw/current/scripts/prefabs/wagstaff_npc.lua#L1495-L1505)） | `persists=false`；剪影子体仅本地存在。 |
| `alterguardian_contained` | 无显式 Remove：随瓦格斯塔夫 NPC 工作流反复重建（relocate/读档推送 `spawndevice`），erosion 进出场同步（[relocate_wagstaff](../../../raw/current/scripts/prefabs/wagstaff_npc.lua#L671-L701)） | `persists=false` 但由 NPC 侧任务续期；收集动画播完转 `close_idle` 待下一轮。 |

## 典型接入

### 复刻"伤害与视觉同体"的 Boss AOE 光束

- 前提与选择：需要 AOE 表现自带真实结算（破坏/采集/伤害记在 FX 而非 Boss）时，复刻 beam 模式：FX 实体挂 combat＋planardamage，`updatelooper` 每帧跑结算函数；死亡归属用 `SetPrefabNameOverride` 记回宿主。
- 生效与撤销：用 `DoTaskInTime` 设硬寿命收口，退出前显式 `UnregisterLunarBurnSource` 与 `PopColour`，否则染色与月灼源会残留到目标身上。
- 原版依据：[结算函数](../../../raw/current/scripts/prefabs/wagboss_beam.lua#L153-L311)、[KillFx](../../../raw/current/scripts/prefabs/wagboss_beam.lua#L393-L419)。

```lua
local fx = SpawnPrefab("wagboss_beam_fx")
fx:TrackTarget(target, x, z) -- beam_pre 期间向 target 滑移
```

- 调整与验证：替换点是注册名、伤害调值与结算标签集；`TrackTarget` 只在 `beam_pre` 生效，错过该动画窗口则光束停在原地。示例只做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **误区：光束伤害是 Boss 打的。** 首击与破坏都由光束实体的 combat/workable 结算（Boss 不在调用链里）；唯一的回写是死亡播报的名字覆盖。做"受伤来源"统计或战利品判定时要按光束实体处理。
- **边界：`inst.caster` 无赋值点。** 结算里的 `commander:IsSoldier` 豁免分支依赖 `inst.caster`，而全域 `grep -rn "wagboss_beam"` 仅命中注册/依赖/一处生成，**没有任何代码给 beam 写 caster**——士兵豁免实际不可达，复刻时要么补赋值要么删分支（[唯一读取点](../../../raw/current/scripts/prefabs/wagboss_beam.lua#L167-L167)）。
- **边界：机械腿 FX 不是落地喷雾。** 腿落地只播音效（[DoLandingSfx](../../../raw/current/scripts/prefabs/wagboss_robot_leg.lua#L70-L75)）；FX 在超新星灼烧且腿在范围内时才出现，方向背向灼烧源，同一腿对多个源会叠多层。
- **边界：finale 体是联网说话体，剪影才是本地层。** 直接 `SpawnPrefab("wagstaff_npc_finale_fx")` 而不 `AttachToAlter` 会得到一个悬空、erosion 隐藏的实体；专用服务器上没有剪影子体（[非专服分支](../../../raw/current/scripts/prefabs/wagstaff_npc.lua#L1632-L1635)）。
- **边界：beam 的地面标记只在非专服存在。** `CreateRing` 子体在专服为 nil，客户端靠 `animsync` 净变量 dirty 对齐标记动画时间（[L38-L68](../../../raw/current/scripts/prefabs/wagboss_beam.lua#L38-L68)）；专服只剩联网动画壳与伤害结算。
- **边界：contained 的 Light 平时是关的。** 只有 `docollect` 时点亮（[L1677-L1678](../../../raw/current/scripts/prefabs/wagstaff_npc.lua#L1676-L1685)）；把它当常亮路标用需自行 Enable，且 erosion 参数初始 -1.0（不可见），要显式调 `erode` 才会显形。

## 验证记录

- 源码审阅：2026-11-05，steam-buildid:26335394（实读 `raw/current/BUILD.txt`）。完整读取 `prefabs/wagboss_beam.lua`、`prefabs/wagboss_robot_leg.lua` 全文，`prefabs/wagstaff_npc.lua`（finale/contained 两工厂与 spawn_device/relocate 段；同文件其余注册 `wagstaff_npc`／`wagstaff_npc_pstboss`／`wagstaff_npc_mutations`／`wagstaff_npc_wagpunk`／`wagstaff_npc_wagpunk_arena`／`enable_lunar_rift_construction_container` 不在本页），按需读取 `SGwagboss_robot.lua`（轨道打击目标选取与 signal_loop/death 状态）、`SGalterguardian_phase4_lunarrift.lua`（supernova AOE 与 finale 状态）、`SGwagstaff_npc.lua` capture 段、`lunarsupernovablocker.lua` 全文、`wagboss_util.lua` 月灼段、`alterguardian_phase3dead.lua`、`wagpunk_arena_manager.lua` 监听行、`tuning.lua` 相关调值。对 4 个注册名以 `grep -rn "注册名" --include=*.lua` 在 `raw/current/scripts` 全域反查：`wagboss_beam_fx` 命中注册（wagboss_beam.lua:492）、依赖（wagboss_robot.lua:13）、生成（SGwagboss_robot.lua:404）——**生成点唯一**；`wagboss_robot_leg_fx` 命中注册（wagboss_robot_leg.lua:267）、依赖（:14）、生成（lunarsupernovablocker.lua:32）——**生成点唯一**；`wagstaff_npc_finale_fx` 命中注册（wagstaff_npc.lua:1767）、依赖（alterguardian_phase4_lunarrift.lua:33）、生成（SGalterguardian_phase4_lunarrift.lua:1563）——**生成点唯一**；`alterguardian_contained` 命中注册（wagstaff_npc.lua:1768）、依赖（:31）与生成（:595，经 `spawndevice` 事件的三处推送），其余命中为图鉴/皮肤数据。
- 名称核验：`languages/chinese_s.po` 定向检索——`ALTERGUARDIAN_CONTAINED`＝"月亮精华提取器"、`WAGBOSS_ROBOT`＝"战争瓦器人"、`WAGSTAFF_NPC`＝"颗粒状传输"（锚点见"快速判断"）；`WAGBOSS_BEAM_FX`／`WAGBOSS_ROBOT_LEG_FX`／`WAGSTAFF_NPC_FINALE_FX` 无 `STRINGS.NAMES.*` 键，标题使用效果描述。
- 分档说明：`size_class: standard`——4 注册两条 Boss 战链＋一条 NPC 工作流链，结算归属与生成时机各成小节；未选 complex 因超新星/竞技场体系另有展开空间，本页只记到 FX 触发深度。
- 未覆盖：`wagboss_robot` 战斗状态机全貌与 `wagboss_robot_leg` 本体的落地物理细节、超新星灼烧（lunarsupernovaburning）与月灼抗性数值体系、月相天体 Boss 其余状态、竞技场管理器与剧情流程、`wagboss_lunar_blast`/`wagboss_beam` 动画包内容。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 结果见本页交付记录（0 error 0 warning 后入库）。
- 游戏运行：未运行；`validation` 保持 `source-reviewed`。

## 相关链接

- [[reference/prefabs/rocky_snip_fx|岩甲 Boss 攻击特效与枯萎变体表现]]（同为 Boss 战 SG 生成 FX；对照"伤害留宿主"与"伤害在 FX"两种归属写法）
- [[reference/prefabs/deerclops_laserscorch|独眼巨鹿激光族]]（同为混合载体激光族，含焦痕/烟尾分层）
- [[reference/prefabs/charlie_boss_aoe_flame_fx|藏影女王 Boss 战表现特效族]]（同为 Boss 战多注册、混合载体伤害归属辨析）
- 返回 [[reference/prefabs/fx-creature-index|特效 Prefab 索引·生物与Boss]]（WAGboss 机械 Boss 战）
