---
title: "天体英雄激光、灼烧与地面特效（alterguardian_laserscorch 等）"
created: "2026-11-04"
updated: "2026-11-04"
type: prefab
summary: "天体英雄与天体后裔的八项外围表现注册：激光焦痕／烟尾／命中加色、裂隙与超新星灼烧叠层、三阶段陷阱脉冲地圈、四阶段砸地环与喷发；按工厂、端与三种收尾协议分组核验。"
aliases: ["alterguardian_laserscorch", "alterguardian_lasertrail", "alterguardian_laserhit", "alterguardian_lunar_fissure_burn_fx", "alterguardian_lunar_supernova_burn_fx", "alterguardian_phase3trapgroundfx", "alterguardian_phase4_lunarrift_slam_fx", "alterguardian_phase4_lunarrift_erupt_fx", "天体英雄激光、灼烧与地面特效", "天体英雄与天体后裔的激光、灼烧与地面特效", "天体英雄激光焦痕", "天体英雄灼烧地面特效", "天体后裔砸地特效"]
tags: [prefab, fx, server-client]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/alterguardian_laser.lua", "prefabs/alterguardian_lunar_fissures.lua", "prefabs/alterguardian_phase3trap.lua", "prefabs/alterguardian_phase4_lunarrift.lua", "prefabs/alterguardian_phase3.lua", "prefabs/alterguardian_phase3circle.lua", "components/lunarfissureburning.lua", "components/lunarsupernovaburning.lua", "stategraphs/SGalterguardian_phase3.lua", "stategraphs/SGalterguardian_phase4_lunarrift.lua", "stategraphs/SGdaywalker2.lua", "netvars.lua", "entityscript.lua", "languages/chinese_s.po"]
size_class: complex
reliability: verified
validation: source-reviewed
status: active
---

# 天体英雄激光、灼烧与地面特效（alterguardian_laserscorch 等）

## 快速判断

- 职责与适用：四个文件、八个注册，全部是纯表现实体，不带伤害、生命或灼烧组件：三阶段激光的焦痕／烟尾／命中闪光（[注册](../../../raw/current/scripts/prefabs/alterguardian_laser.lua#L487-L491)）、裂隙与超新星的灼烧叠层（[注册](../../../raw/current/scripts/prefabs/alterguardian_lunar_fissures.lua#L341-L343)）、三阶段陷阱的地面脉冲圈（[注册](../../../raw/current/scripts/prefabs/alterguardian_phase3trap.lua#L359-L361)）、四阶段砸地的烟雾和喷发（[注册](../../../raw/current/scripts/prefabs/alterguardian_phase4_lunarrift.lua#L1232-L1234)）。激光族也被拾荒疯猪炮击复用（[调用方](../../../raw/current/scripts/stategraphs/SGdaywalker2.lua#L453-L486)）。
- 优先使用：查焦痕、烟尾或命中加色看“激光族”；查目标身上的裂隙／超新星燃烧层看“灼烧叠层”；查陷阱脉冲地圈、四阶段砸地环或喷发看各自地面表。生成者与触发条件集中在“关联机制导航”。
- 重要副作用：清理分三类——焦痕靠 netvar 淡出并由服务端移除、烟尾／砸地／喷发靠定时或 `animover`、灼烧与陷阱地圈**没有自删任务**，分别由组件或陷阱宿主 Hide／Remove（[焦痕收尾](../../../raw/current/scripts/prefabs/alterguardian_laser.lua#L327-L337)、[组件清理](../../../raw/current/scripts/components/lunarfissureburning.lua#L12-L19)、[陷阱清理](../../../raw/current/scripts/prefabs/alterguardian_phase3trap.lua#L267-L278)）。所有工厂都设 `persists=false`，但它不等于自动清理。
- 修改边界：`alterguardian_laserhit` 是非联网、无 Transform／AnimState 的目标附加闪光（[工厂](../../../raw/current/scripts/prefabs/alterguardian_laser.lua#L474-L485)）；焦痕用 net_byte 初值加两端本地递减（[netvar](../../../raw/current/scripts/prefabs/alterguardian_laser.lua#L356-L360)）；砸地的地面环是非联网子实体且专服不建（[入口](../../../raw/current/scripts/prefabs/alterguardian_phase4_lunarrift.lua#L1175-L1178)）。生成者、宿主和具体玩法实体不在本页展开。

## 注册与构造

下表按文件分组；这些是独立 Prefab 工厂（不是 `fx.lua` 通用表条目），八个工厂都不挂玩法组件。

### 激光族（prefabs/alterguardian_laser.lua）

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `alterguardian_laserscorch` | `scorchfn`：Transform＋AnimState＋Network → bank/build `burntground` 播 `idle`，OnGround／背景层／SortOrder 3 → NOCLICK／FX → net_byte `alterguardian_laserscorch._fade`（脏事件 `fadedirty`）初值 25（20 蓝＋5 淡出）→ `DoPeriodicTask(0, Scorch_OnUpdateFade)` 与首次 `Scorch_OnFadeDirty` → SetPristine → 客户端监听 `fadedirty` 后早退 → 主模拟随机旋转、`persists=false`。 | 单枚落点焦痕；无音效；激光链可用 `Trigger` 的 `scorchscale` 覆盖缩放，直接生成时默认 1。[工厂](../../../raw/current/scripts/prefabs/alterguardian_laser.lua#L339-L374)、[淡出协议](../../../raw/current/scripts/prefabs/alterguardian_laser.lua#L305-L337)、[注册](../../../raw/current/scripts/prefabs/alterguardian_laser.lua#L489) |
| `alterguardian_lasertrail` | `trailfn`：Transform＋AnimState＋Network → FX／NOCLICK → bank/build `lavaarena_staff_smoke_fx` 播 `idle`，`SetAddColour(0,0,1,0)`＋`SetMultColour(0,0,1,1)`＋bloom → SetPristine → 客户端早退 → 主模拟 `persists=false`、动画长度＋2 帧后 Remove，并暴露 `FastForward`。 | 蓝烟尾；`FastForward(pct)` 取消旧任务、`SetTime(len*pct)` 后按剩余时长重排，pct 夹到 0-1。[工厂](../../../raw/current/scripts/prefabs/alterguardian_laser.lua#L386-L414)、[FastForward](../../../raw/current/scripts/prefabs/alterguardian_laser.lua#L376-L384)、[注册](../../../raw/current/scripts/prefabs/alterguardian_laser.lua#L490) |
| `alterguardian_laserhit` | `hitfn`：只 CreateEntity → `CLASSIFIED` 标签；无 Transform／AnimState／Network，注释明示非联网 → `persists=false` → 暴露 `SetTarget`，同时以 0 帧任务兜底 Remove。 | 不落点、不显示自身；绑定目标后按 `bloomer` 推泛光或直写 AnimState；`flash` 初值 0.8-1.2，逐帧减 0.075，低于 0.3 弹掉泛光，归零 Remove；`OnRemoveHit` 在目标未挂 `colouradder`／`bloomer` 时还原直写效果。[工厂](../../../raw/current/scripts/prefabs/alterguardian_laser.lua#L474-L485)、[绑定](../../../raw/current/scripts/prefabs/alterguardian_laser.lua#L455-L472)、[逐帧](../../../raw/current/scripts/prefabs/alterguardian_laser.lua#L431-L453)、[还原](../../../raw/current/scripts/prefabs/alterguardian_laser.lua#L416-L429)、[注册](../../../raw/current/scripts/prefabs/alterguardian_laser.lua#L491) |

### 灼烧叠层（prefabs/alterguardian_lunar_fissures.lua）

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `alterguardian_lunar_fissure_burn_fx` | `fxfn`：Transform＋AnimState＋Network → DECOR／NOCLICK → bank/build `wagboss_lunar_blast` 播循环 `fissure_hit_small`，bloom、LightOverride 0.3、FinalOffset 3 → SetPristine → 客户端早退 → 主模拟挂 `SetFxSize`、`persists=false`。 | 单枚叠层，由组件父挂目标；`SetFxSize` 在 `fissure_hit_small/med/large` 间切循环动画，不缩放实体；无 netvar、无自删。[工厂](../../../raw/current/scripts/prefabs/alterguardian_lunar_fissures.lua#L275-L302)、[尺寸](../../../raw/current/scripts/prefabs/alterguardian_lunar_fissures.lua#L268-L273)、[资源](../../../raw/current/scripts/prefabs/alterguardian_lunar_fissures.lua#L9-L12)、[注册](../../../raw/current/scripts/prefabs/alterguardian_lunar_fissures.lua#L342) |
| `alterguardian_lunar_supernova_burn_fx` | `supernovafxfn`：Transform＋AnimState＋Network → FX／NOCLICK → 同 bank/build 播循环 `supernova_hit_small`，bloom、LightOverride 0.3，**无 FinalOffset** → SetPristine → 客户端早退 → 主模拟 `SetFxSize`＋`persists=false`。 | 每个灼烧源一枚、不父挂；由组件按目标半径摆到目标朝源方向的边缘，遮挡时 Hide；同样三档尺寸、无自删。[工厂](../../../raw/current/scripts/prefabs/alterguardian_lunar_fissures.lua#L313-L339)、[尺寸](../../../raw/current/scripts/prefabs/alterguardian_lunar_fissures.lua#L306-L311)、[注册](../../../raw/current/scripts/prefabs/alterguardian_lunar_fissures.lua#L343) |

### 三阶段陷阱（prefabs/alterguardian_phase3trap.lua）

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `alterguardian_phase3trapgroundfx` | `groundfx_fn`：Transform＋AnimState＋Network → CLASSIFIED／DECOR／NOCLICK → bank/build `alterguardian_meteor` 播 `meteorground_pre`，OnGround／背景层／SortOrder 3 → SetPristine → 客户端早退 → 主模拟 `persists=false`。 | 脉冲地圈；无自删、无 netvar；陷阱懒生成后复用（首次 Spawn，之后 Show），每次先播 pre 再 Push 循环 loop；`finish_pulse` 播 pst 并在其时长后 Hide，陷阱移除时一并 Remove。[工厂](../../../raw/current/scripts/prefabs/alterguardian_phase3trap.lua#L330-L357)、[播放](../../../raw/current/scripts/prefabs/alterguardian_phase3trap.lua#L205-L227)、[收尾](../../../raw/current/scripts/prefabs/alterguardian_phase3trap.lua#L190-L202)、[移除](../../../raw/current/scripts/prefabs/alterguardian_phase3trap.lua#L267-L278)、[注册](../../../raw/current/scripts/prefabs/alterguardian_phase3trap.lua#L361) |

### 四阶段砸地（prefabs/alterguardian_phase4_lunarrift.lua）

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `alterguardian_phase4_lunarrift_slam_fx` | `slamfxfn`：Transform＋AnimState＋Network → bank/build `bomb_lunarplant` 播 `used`，`Hide("bomb")`、`OverrideSymbol("sleepcloud_pre","sleepcloud","sleepcloud_pre")`，bloom、MultColour alpha 0.2、AddColour、点采样、缩放 2、LightOverride 0.5 → FX／NOCLICK → 非专服建非联网子实体 `slamfx_CreateGroundFx`（bank/build `wagboss_robot` 播 `atk_ground_projection`，OnGround／背景层、alpha 0.2、缩放 5/3.3）并 SetParent → SetPristine → 客户端把子环动画时间对准主层一次并自摘 → 主模拟 animover Remove、`persists=false`。 | 烟雾主层＋地面投影环；专服不建环；子环不联网，只在本端存在；主层 animover Remove 不显式移除子环，引擎级联未核验。[主工厂](../../../raw/current/scripts/prefabs/alterguardian_phase4_lunarrift.lua#L1153-L1193)、[子环](../../../raw/current/scripts/prefabs/alterguardian_phase4_lunarrift.lua#L1119-L1146)、[客户端对时](../../../raw/current/scripts/prefabs/alterguardian_phase4_lunarrift.lua#L1148-L1151)、[资源](../../../raw/current/scripts/prefabs/alterguardian_phase4_lunarrift.lua#L14-L19)、[注册](../../../raw/current/scripts/prefabs/alterguardian_phase4_lunarrift.lua#L1233) |
| `alterguardian_phase4_lunarrift_erupt_fx` | `eruptfxfn`：Transform＋AnimState＋Network → bank/build `wagboss_lunar_blast` 播 `blast_01`，bloom、MultColour alpha 0.4、LightOverride 0.5 → FX／NOCLICK → SetPristine → 客户端早退 → 主模拟 50% 改播 `blast_02`，animover Remove、`persists=false`。 | 单发喷发、无地面朝向；与灼烧层共用 `wagboss_lunar_blast` 资源。[工厂](../../../raw/current/scripts/prefabs/alterguardian_phase4_lunarrift.lua#L1197-L1228)、[资源](../../../raw/current/scripts/prefabs/alterguardian_phase4_lunarrift.lua#L21-L24)、[注册](../../../raw/current/scripts/prefabs/alterguardian_phase4_lunarrift.lua#L1234) |

八个注册名都没有对应的 `STRINGS.NAMES.*` 键（`languages/chinese_s.po` 全文件定向检索）；标题与表中的“焦痕／烟尾／命中闪光／灼烧叠层／脉冲地圈／砸地环／喷发”均为已核验效果描述，不是官方译名。宿主官方名：三阶段 `ALTERGUARDIAN_PHASE3`＝天体英雄（[po](../../../raw/current/scripts/languages/chinese_s.po#L338559-L338562)），四阶段 `ALTERGUARDIAN_PHASE4_LUNARRIFT`＝天体后裔（[po](../../../raw/current/scripts/languages/chinese_s.po#L338579-L338582)）。

### 端与收尾对照

| 注册名 | 端与可见范围 | 清理与寿命 |
|---|---|---|
| `alterguardian_laserscorch` | 联网；主模拟生成并 `set` 初值，之后两端各自 `set_local` 递减，客户端按本地计时淡出。 | 服务端从 25 递减到 1 时 Remove；客户端在值降到 1 时把本地值置 0 并隐藏。[递减](../../../raw/current/scripts/prefabs/alterguardian_laser.lua#L327-L337) |
| `alterguardian_lasertrail` | 联网；主模拟生成，客户端早退；动画状态随联网实体表现。 | 动画长度＋2 帧后 Remove；`FastForward` 会取消并重排该任务。[定时](../../../raw/current/scripts/prefabs/alterguardian_laser.lua#L409)、[重排](../../../raw/current/scripts/prefabs/alterguardian_laser.lua#L383) |
| `alterguardian_laserhit` | **非联网**，只在主模拟创建和执行；无空间位置。 | 闪光衰减到 0 时 Remove；未被 `SetTarget` 时由 0 帧任务 Remove；移除时按目标未挂的 `colouradder`／`bloomer` 分别还原直写效果。[兜底](../../../raw/current/scripts/prefabs/alterguardian_laser.lua#L482)、[逐帧](../../../raw/current/scripts/prefabs/alterguardian_laser.lua#L431-L453) |
| `alterguardian_lunar_fissure_burn_fx` | 联网；主模拟生成并父挂目标，客户端拿到实体与父级。 | **无自删**；离开灼烧时 Hide，组件移除时 Remove。[组件](../../../raw/current/scripts/components/lunarfissureburning.lua#L12-L56) |
| `alterguardian_lunar_supernova_burn_fx` | 联网；主模拟按灼烧源生成并逐源摆位。 | **无自删**；源失效或出圈时 Remove，遮挡时 Hide，组件移除时全部 Remove。[源循环](../../../raw/current/scripts/components/lunarsupernovaburning.lua#L58-L90)、[组件移除](../../../raw/current/scripts/components/lunarsupernovaburning.lua#L18-L20) |
| `alterguardian_phase3trapgroundfx` | 联网；主模拟生成并驱动 pre／loop／pst，客户端早退。 | **无自删**；陷阱 `finish_pulse` 播 pst 后 Hide；陷阱移除时 Remove。[收尾](../../../raw/current/scripts/prefabs/alterguardian_phase3trap.lua#L190-L202)、[移除](../../../raw/current/scripts/prefabs/alterguardian_phase3trap.lua#L267-L278) |
| `alterguardian_phase4_lunarrift_slam_fx` | 主层联网；地面环是非联网子实体，`TheNet:IsDedicated()` 为真时不建。客户端把环对时一次后自摘 updatelooper。 | 主层 `animover` Remove；环只 `SetParent`，代码未显式 Remove，宿主删除时的引擎级联未核验。[主层](../../../raw/current/scripts/prefabs/alterguardian_phase4_lunarrift.lua#L1189-L1190) |
| `alterguardian_phase4_lunarrift_erupt_fx` | 联网；主模拟生成并决定 50% 变体。 | `animover` Remove。[清理](../../../raw/current/scripts/prefabs/alterguardian_phase4_lunarrift.lua#L1224-L1225) |

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 生成者：三阶段 SG `atk_beam` | 第 35 帧调用三次 `SpawnBeam`；`SpawnBeam` 沿 i=0..10 共 11 格生成（首格为 `alterguardian_laserempty`，只有伤害、无 AnimState／Light，DoDamage 走 else 分支 2 帧后自删；其余为 `alterguardian_laser`），循环后另补 2 道，带 `caster` 和 `Trigger`；第二次 blast 传 `skipscorch=true`，只跳过焦痕，烟尾仍生成。 | 主模拟；空激光只结算伤害、不产生视觉；首击 10 条可视激光＋1 条空激光，循环后再补 2 道。 | [状态时间轴](../../../raw/current/scripts/stategraphs/SGalterguardian_phase3.lua#L689-L717)、[SpawnBeam](../../../raw/current/scripts/stategraphs/SGalterguardian_phase3.lua#L61-L140)、[第二击](../../../raw/current/scripts/stategraphs/SGalterguardian_phase3.lua#L118-L126) |
| 生成者：三阶段 SG `atk_sweep` | 第 37 帧调用 1-3 次 `SpawnSweep`；按距离分 10+ 段生成同名激光，第二 blast 同样 `skipscorch`。 | 主模拟；扫射段数随距离增加。 | [状态时间轴](../../../raw/current/scripts/stategraphs/SGalterguardian_phase3.lua#L808-L823)、[SpawnSweep](../../../raw/current/scripts/stategraphs/SGalterguardian_phase3.lua#L146-L216) |
| 生成者：拾荒疯猪炮击 SG | `SpawnLaser`／`SpawnLaserHitOnly` 是唯一向 `Trigger` 传 `scale`、`scorchscale`、`hitscale` 与击退参数的调用方；`scorchscale < 0.2` 时跳过焦痕；第 5 帧先给自身生成 `laserhit`，再生成六道激光。 | 主模拟；属混合玩法复用，本页只核验其表现子实体。 | [炮击](../../../raw/current/scripts/stategraphs/SGdaywalker2.lua#L453-L486)、[时间轴](../../../raw/current/scripts/stategraphs/SGdaywalker2.lua#L1456-L1472) |
| 生成者：三阶段召唤圈 | `start_summon_circle` 生成 `alterguardian_phase3circle`；它每 3 帧在圆周点生成 `lasertrail`，这是烟尾唯一的非激光生成点。 | 主模拟；与召唤警示光柱并存，光柱见兄弟页。 | [start_summon_circle](../../../raw/current/scripts/stategraphs/SGalterguardian_phase3.lua#L222-L234)、[圆周生成](../../../raw/current/scripts/prefabs/alterguardian_phase3circle.lua#L28-L38)、[任务](../../../raw/current/scripts/prefabs/alterguardian_phase3circle.lua#L56) |
| 灼烧：`lunarfissureburning` | 裂隙每 0.5 秒扫描并把组件挂到接触目标；组件首次启用时生成一枚 `alterguardian_lunar_fissure_burn_fx` 并父挂，按目标物理半径／标签选 small／med／large。 | 主模拟；离开裂隙 Hide，组件移除时 Remove。玩法细节见组件页。 | [挂载](../../../raw/current/scripts/prefabs/alterguardian_lunar_fissures.lua#L53-L59)、[组件生成与尺寸](../../../raw/current/scripts/components/lunarfissureburning.lua#L21-L56)、[[reference/components/lunarfissureburning]] |
| 灼烧：`lunarsupernovaburning` | 四阶段命中扫描给目标挂组件并 `AddSource(inst)`；每个源生成一枚 `alterguardian_lunar_supernova_burn_fx`，组件按目标半径把它摆到目标朝源方向的边缘。 | 主模拟；遮挡 Hide，源失效／出圈 Remove。玩法细节见组件页。 | [命中与加源](../../../raw/current/scripts/stategraphs/SGalterguardian_phase4_lunarrift.lua#L226-L241)、[AddSource](../../../raw/current/scripts/components/lunarsupernovaburning.lua#L35-L49)、[摆位](../../../raw/current/scripts/components/lunarsupernovaburning.lua#L58-L90)、[[reference/components/lunarsupernovaburning]] |
| 生成者：三阶段陷阱 | 本体 `SpawnTrapProjectile` 生成 `alterguardian_phase3trapprojectile`，其 `animover` 再生成 `alterguardian_phase3trap`；陷阱 timer 的 `start_pulse` 懒生成／复用地面圈，`finish_pulse` 收尾，`onremove` 清理。 | 主模拟；陷阱的落点伤害、眩晕、可挖与战利品属玩法，不在本页展开。 | [本体投放](../../../raw/current/scripts/prefabs/alterguardian_phase3.lua#L140-L153)、[投射物收尾](../../../raw/current/scripts/prefabs/alterguardian_phase3trap.lua#L88-L99)、[脉冲](../../../raw/current/scripts/prefabs/alterguardian_phase3trap.lua#L205-L227)、[移除](../../../raw/current/scripts/prefabs/alterguardian_phase3trap.lua#L267-L278) |
| 生成者：四阶段 `slam` | 第 42 帧生成两枚 `slam_fx`（Boss 两侧 r=3、±45°）；第 74 帧围绕两个落点各 4 枚 `erupt_fx`（r=3.25）；第 75 帧再各生成一枚 `slam_fx`。 | 主模拟；第 76 帧的第二段 AOE／震动由 SG 处理，不在本页。 | [状态与时间轴](../../../raw/current/scripts/stategraphs/SGalterguardian_phase4_lunarrift.lua#L614-L726) |
| 资源 | `burntground.zip`、`lavaarena_staff_smoke_fx.zip`、`wagboss_lunar_blast.zip`、`alterguardian_meteor.zip`、`bomb_lunarplant.zip`＋`sleepcloud.zip`＋`wagboss_robot.zip`。 | 声明与使用已核验；包内动画帧数和视觉内容未解包。 | [焦痕资源](../../../raw/current/scripts/prefabs/alterguardian_laser.lua#L6-L14)、[灼烧资源](../../../raw/current/scripts/prefabs/alterguardian_lunar_fissures.lua#L9-L12)、[砸地资源](../../../raw/current/scripts/prefabs/alterguardian_phase4_lunarrift.lua#L14-L24) |

## 与玩法实体及兄弟页的分工

- 玩法实体：`alterguardian_laser`／`alterguardian_laserempty` 的激光伤害、可工作物破坏、温度／理智与船只漏水等（[DoDamage](../../../raw/current/scripts/prefabs/alterguardian_laser.lua#L39-L226)）、`alterguardian_lunar_fissures` 的裂隙登记与 AOE 挂载（[扫描](../../../raw/current/scripts/prefabs/alterguardian_lunar_fissures.lua#L37-L62)）、`alterguardian_phase3trapprojectile`／`alterguardian_phase3trap` 的落点伤害、眩晕、可挖与战利品（[陷阱工厂](../../../raw/current/scripts/prefabs/alterguardian_phase3trap.lua#L280-L328)）、`alterguardian_phase4_lunarrift` 本体的战斗与状态均属玩法，不在本页展开；本页只覆盖它们的表现子实体。
- 兄弟页：旋转轨迹、尖刺破裂、月脉冲与同文件的 `alterguardian_phase3trappst` 见 [[reference/prefabs/alterguardian_spintrail_fx]]；召唤警示光柱见 [[reference/prefabs/alterguardian_summon_fx]]。本页不复述这些工厂与收尾，只处理 `alterguardian_phase3trapgroundfx` 及其余七项。

## 典型接入

### 落点焦痕

- 前提与选择：主模拟生成；落点需要时先按原版用 `TheWorld.Map:IsPassableAtPoint` 校验；要缩放时按原版在生成后 `SetScale`（激光链中由 `Trigger` 的 `scorchscale` 参数传入）。
- 生效与撤销：实体自身从 25 递减，服务端减到 1 即 Remove；无需额外清理，也不应把它当伤害源。
- 原版依据：激光命中链在可通行点生成焦痕（[命中表现](../../../raw/current/scripts/prefabs/alterguardian_laser.lua#L57-L63)），焦痕淡出协议见[收尾](../../../raw/current/scripts/prefabs/alterguardian_laser.lua#L327-L337)。

```lua
-- 主模拟：在可通行落点生成焦痕，实体自行淡出并移除
local scorch = SpawnPrefab("alterguardian_laserscorch")
scorch.Transform:SetPosition(x, 0, z)
scorch.AnimState:SetScale(1.2, 1.2) -- 可选；原版 daywalker2 按炮击档位传入
```

- 调整与验证：缩放只改视觉，不改变 25 tick 的寿命或淡出曲线；不生成光、声音或伤害。未运行验证实际视觉。

### 目标身上的裂隙灼烧层

- 前提与选择：主模拟；目标是已存在的宿主实体。原版由 `lunarfissureburning` 组件持有并父挂，单枚复用；若要复现，最好由自己的状态／组件管理生命周期，而不是在伤害回调里反复 Spawn。
- 生效与撤销：`SetFxSize` 只切换循环动画名（small／med／large），不缩放实体；离开灼烧时组件 Hide，组件移除时 Remove；实体本身没有自删任务。
- 原版依据：组件首次启用时生成并父挂（[生成](../../../raw/current/scripts/components/lunarfissureburning.lua#L21-L38)），尺寸映射见[选择](../../../raw/current/scripts/components/lunarfissureburning.lua#L43-L50)。

```lua
-- 主模拟：把裂隙灼烧层挂到目标身上；结束时由控制方 Remove
local fx = SpawnPrefab("alterguardian_lunar_fissure_burn_fx")
fx.entity:SetParent(target.entity)
fx:SetFxSize("med") -- 仅主模拟挂载；small / med / large
-- fx 没有自删任务，离开灼烧或目标移除时要显式 Remove/Hide
```

- 调整与验证：`alterguardian_lunar_supernova_burn_fx` 虽然同 build，但由 `lunarsupernovaburning` 按“每源一枚、目标边缘摆位”管理，不能照搬单枚父挂模式；两枚 FX 都不造成灼烧伤害，伤害在组件。

## 常见陷阱与限制

- **误区：这些 FX 会自己造成灼烧或伤害**：八个工厂都没有 combat／health／lunar burn 组件；裂隙与超新星的扣血在 `lunarfissureburning`／`lunarsupernovaburning`，激光伤害在 `alterguardian_laser`。[焦痕工厂](../../../raw/current/scripts/prefabs/alterguardian_laser.lua#L339-L374)、[灼烧工厂](../../../raw/current/scripts/prefabs/alterguardian_lunar_fissures.lua#L275-L339)
- **边界：`persists=false` 只保证不入存档，不等于自动消失**：裂隙灼烧层、超新星灼烧层、陷阱地圈都没有自删任务，分别依赖组件或陷阱宿主 Hide／Remove；单独 Spawn 后不管会一直留到场景卸载。[组件清理](../../../raw/current/scripts/components/lunarfissureburning.lua#L12-L19)、[陷阱清理](../../../raw/current/scripts/prefabs/alterguardian_phase3trap.lua#L267-L278)
- **边界：`alterguardian_laserhit` 不能摆位**：它没有 Transform／AnimState／Network，只接受 `SetTarget`；生成后不绑定目标会被 0 帧任务直接移除。目标通常还需要 AnimState（有 `colouradder`／`bloomer` 时走组件通道），不能把它当普通落点 FX 用。[工厂](../../../raw/current/scripts/prefabs/alterguardian_laser.lua#L474-L485)、[绑定](../../../raw/current/scripts/prefabs/alterguardian_laser.lua#L455-L472)
- **边界：焦痕 netvar 不逐帧同步**：服务端只在创建时 `set` 初值，之后两端用 `set_local` 各自递减；改淡出语义或需要中途重同步时应显式调用 `set`，否则客户端只按本地节奏估算。[初值](../../../raw/current/scripts/prefabs/alterguardian_laser.lua#L356-L360)、[netvar 语义](../../../raw/current/scripts/netvars.lua#L52-L67)
- **边界：客户端不能调用主模拟方法**：`FastForward`、`SetFxSize` 在客户端早退之后才赋值；`alterguardian_laserhit` 本身非联网，只应由主模拟的激光／疯猪生成，`SetTarget` 也不应在客户端调用。`slam_fx` 的地面环则是非联网子实体，专服不建，跨端逻辑不能假设它存在。[FastForward 赋值](../../../raw/current/scripts/prefabs/alterguardian_laser.lua#L411)、[SetFxSize 赋值](../../../raw/current/scripts/prefabs/alterguardian_lunar_fissures.lua#L298)、[环入口](../../../raw/current/scripts/prefabs/alterguardian_phase4_lunarrift.lua#L1175-L1178)

## 验证记录

- 源码审阅：2026-11-04，Steam buildid 26335394。逐注册核验八个注册的工厂链、AnimState／bank／build／anim、符号覆盖、缩放、标签、netvar、端与清理，并追到生成者：`alterguardian_laser.lua` 全文、`alterguardian_lunar_fissures.lua` 全文、`alterguardian_phase3trap.lua` 全文、`alterguardian_phase4_lunarrift.lua` 的 FX 工厂段（L1119-L1234）；另核 `lunarfissureburning`／`lunarsupernovaburning` 组件、三阶段与四阶段 SG、`SGdaywalker2` 炮击、`alterguardian_phase3circle`。以 `raw/current/scripts` 全目录 Lua 文件反查八个完整注册名及 `alterguardian_laser`、`phase3trapground`、`lunar_fissure_burn`、`lunar_supernova_burn`、`phase4_lunarrift_slam`、`phase4_lunarrift_erupt` 前缀；命中调用均列于上表，`prefabs`／依赖数组不计生成。`languages/chinese_s.po` 全文件检索八个 `STRINGS.NAMES.*` 无命中；宿主官方名按 `ALTERGUARDIAN_PHASE3`／`ALTERGUARDIAN_PHASE4_LUNARRIFT` 条目核对。各动画包内容、实际帧数、晚加入表现未核验。
- 分档说明：覆盖 4 个文件、8 个注册、三种收尾协议与联网／非联网／专服差异，需要整体对照保留，故 `size_class: complex`。
- 游戏运行：未运行。
- 结构检查：validate_page 通过（141 行／27578 字节，complex 档 400 行／32768 字节）；validate_source_links --strict 通过（97 条行引用，0 error／0 warning）；两段 Lua 示例均通过 `parser_probe.exe --syntax-only`。
- 未覆盖：各动画包的实际帧数与视觉、客户端动画同步细节、`SGdaywalker2` 完整炮击战斗链、三阶段陷阱与四阶段 Boss 的玩法机制（归各自页面）。

返回 [[reference/prefabs/fx-creature-index|生物、Boss 与遭遇演出]]。
