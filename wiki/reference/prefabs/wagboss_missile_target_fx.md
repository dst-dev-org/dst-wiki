---
title: "WAGboss 导弹落点与无人机电束电球（5 注册）"
created: "2026-11-05"
updated: "2026-11-08"
type: prefab
summary: "WAGboss 阵营外围五注册：热追踪导弹的客户端落点环、黄莺／动能场扩展器的电束、黄莺与电刑机的电球弹体；两类混合载体自带 combat，伤害归属与选取规则各不相同。"
aliases: ["wagboss_missile_target_fx", "wagdrone_laserwire_fx", "wagpunk_cagewall_fx", "wagdrone_projectile_fx", "wx78_drone_zap_projectile_fx", "热追踪导弹落点", "黄莺电束", "电刑机电球", "动能场扩展器电束"]
tags: [prefab, fx, server-client]
size_class: complex
dst_build: "steam-buildid:26474873"
sources: ["prefabs/wagboss_missile.lua", "prefabs/wagdrone_laserwire.lua", "prefabs/wagdrone_projectile.lua", "prefabs/wagdrone_rolling.lua", "prefabs/wagdrone_common.lua", "components/combat.lua", "stategraphs/SGwagdrone_flying.lua", "stategraphs/SGwx78_drone_zap.lua", "components/wagpunk_arena_manager.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# WAGboss 导弹落点与无人机电束电球（5 注册）

## 快速判断

- 职责与适用：本页收的是 WAGboss／WAGpunk 外围的 **5 个注册**，分三套表现：`wagboss_missile_target_fx` 是热追踪导弹（`wagboss_missile`，官方名"热追踪导弹"）的落点标记，实体只做网络同步、可见圆环由客户端本地创建；`wagdrone_laserwire_fx`／`wagpunk_cagewall_fx` 是黄莺的滚动形态（`wagdrone_rolling`）之间的电束与"动能场扩展器"（`wagpunk_cagewall`）笼墙电束；`wagdrone_projectile_fx`／`wx78_drone_zap_projectile_fx` 是黄莺与电刑机（`wx78_drone_zap`）的电球弹体。[导弹](../../../raw/current/scripts/prefabs/wagboss_missile.lua#L900-L900)、[电束](../../../raw/current/scripts/prefabs/wagdrone_laserwire.lua#L356-L357)、[电球](../../../raw/current/scripts/prefabs/wagdrone_projectile.lua#L308-L309)
- 优先使用：查"热追踪导弹落点圈"看第一套；查"黄莺／笼墙电束的伤害与客户端分段"看第二套；查"黄莺／电刑机落雷球"看第三套。三套都只在主模拟结算伤害，客户端只有本地视觉。[伤害](#伤害归属与数值)
- 重要副作用：四个混合注册都**以 FX 实体自身作 attacker**（`inst.components.combat:DoAttack`），不是以无人机为 attacker；电束按 beam 分段扫描并自带 1 秒/目标冷却，电球按半径 1.5 结算 18 帧窗口；30 是默认基础伤害，非免疫目标还要乘 1.5＋湿度电系倍率（干燥 45、满湿 75），电免疫分支先把默认伤害降到 15 再结算，两者都会被推 `electrocute`。`wagboss_missile_target_fx` 自身无伤害。[电束扫描](../../../raw/current/scripts/prefabs/wagdrone_laserwire.lua#L136-L177)、[电球命中](../../../raw/current/scripts/prefabs/wagdrone_projectile.lua#L71-L139)
- 修改边界：只覆盖 5 个注册的工厂、端分工、生成点、伤害归属与清理；热追踪导弹本体（`wagboss_missile`）的飞行／AOE／落点选择、黄莺与电刑机的 AI／技能、`wagpunk_cagewall` 实体本身不展开。
- 名称：`WAGBOSS_MISSILE`＝"热追踪导弹"、`WAGDRONE_FLYING`＝"黄莺"、`WX78_DRONE_ZAP`＝"电刑机"、`WAGPUNK_CAGEWALL`＝"动能场扩展器"（[po](../../../raw/current/scripts/languages/chinese_s.po#L351379-L351382)、[po](../../../raw/current/scripts/languages/chinese_s.po#L351424-L351427)、[po](../../../raw/current/scripts/languages/chinese_s.po#L352679-L352682)、[po](../../../raw/current/scripts/languages/chinese_s.po#L351454-L351457)）；5 个 FX 注册名均无 `STRINGS.NAMES` 键，标题中的中文是已核验效果描述。

## 注册与构造

| 注册名 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `wagboss_missile_target_fx` | `targetfn()`：Transform／Network → `CLASSIFIED` → `net_float x0/z0`＋`net_tinybyte arc` → 非专服挂 `updatelooper`（`Target_OnWallUpdate`）→ `SetPristine`；客户端监听 `tweendirty`；主模拟导出 `StartTweenFromXZ`、`persists=false`。 | **无 AnimState**；可见圆环不是它本体，而是客户端 `CreateTargetRing` 造的本地 `FX`／`NOCLICK` 实体；专服不建环。[工厂](../../../raw/current/scripts/prefabs/wagboss_missile.lua#L864-L895)、[本地环](../../../raw/current/scripts/prefabs/wagboss_missile.lua#L730-L756) |
| `wagdrone_laserwire_fx` | `fn()`：Transform／Network → `CLASSIFIED`＋`notarget` → `net_byte len/rot` → `SetPrefabNameOverride("wagdrone_rolling")`、`CanMouseThrough` → `SetPristine`；客户端监听 `beamdirty`；主模拟加 `combat`（默认 30、`ignorehitrange`、`KeepTargetFn=false`）、导出 `SetBeam`、`OnEntitySleep=ClearSegs`／`OnEntityWake=RefreshSegs`、`persists=false`。 | 宿主实体本身无 `FX`／`NOCLICK`；电束分段是客户端本地实体；伤害由宿主 `combat` 结算。[工厂](../../../raw/current/scripts/prefabs/wagdrone_laserwire.lua#L263-L299) |
| `wagpunk_cagewall_fx` | `fn_cagewall()`：同构 → `SetPrefabNameOverride("wagpunk_cagewall")`；主模拟同款 combat 与 `SetBeam_CageWall`；客户端用 `follow_marker_cage` 与 `OnBeamDirty_CageWall`。 | 与激光束共用 netvar 通道名；**快照内无任何生成点**（诚实缺口，见下）。[工厂](../../../raw/current/scripts/prefabs/wagdrone_laserwire.lua#L321-L354) |
| `wagdrone_projectile_fx` | `MakeProjectile`：Transform／AnimState／SoundEmitter／Light／Network／Follower／Physics（mass 1、sphere .5）→ bank/build `wagdrone_projectile`、`projectile_loop` 循环、bloom、scale .65、灯初始关闭 → `showbase` net_bool → `FX`＋`NOCLICK`＋`notarget` → common（`SetPrefabNameOverride("wagdrone_flying")`）→ `SetPristine`；客户端监听 `showbasedirty`；主模拟 `updatelooper`＋`combat`＋`AttachTo`／`Launch`、`persists=false`。 | 电球本体是联网混合载体；命中后落地的 `crackle_projection` 是本地子 FX。[共享工厂](../../../raw/current/scripts/prefabs/wagdrone_projectile.lua#L167-L232)、[黄莺差异](../../../raw/current/scripts/prefabs/wagdrone_projectile.lua#L236-L250) |
| `wx78_drone_zap_projectile_fx` | 同 `MakeProjectile`；common 覆写 `SetPrefabNameOverride("wx78_drone_zap")`；master 设 `damage`＝`SKILLS.WX78.ZAPDRONE_DAMAGE`、`_FindTargets`／`_CheckTarget`／`_OnAttackedTarget`、`_actiondata` 与 `updatelooper`。 | 与黄莺电球共用工厂，差异在**目标筛选（PVP）、友军排除、目标分享与仇恨**；生成时由 SG 写 `caster`。[WX78 差异](../../../raw/current/scripts/prefabs/wagdrone_projectile.lua#L254-L304) |

## 一、热追踪导弹的落点标记

`wagboss_missile` 在 `Launch` 时生成标记并挂到目标（[生成](../../../raw/current/scripts/prefabs/wagboss_missile.lua#L379-L424)）：

```lua
inst.ring = SpawnPrefab("wagboss_missile_target_fx")
inst.ring.Transform:SetRotation((id % 3) * 120)
inst.ring:StartTweenFromXZ(x + 3 * math.cos(theta), z - 3 * math.sin(theta))
```

- 目标是实体时：`ring.entity:SetParent(target.entity)`，并监听目标 `onremove` 与 `wagboss_missile_target_detonated`；目标消失时解除父挂并落回 `targetpos`。目标是坐标时直接放目标点。[挂接](../../../raw/current/scripts/prefabs/wagboss_missile.lua#L407-L413)
- 客户端 `Target_OnWallUpdate` 首次调用才创建本地圆环；之后做 0.5 秒 tween（带弧线）、落点 ping、闪烁／淡出；圆环 `target_pre` 接 `target_loop`、`OnGroundFixed`、背景下层、排序 3、初始 scale 0.85／alpha 0.5。[tween 与环](../../../raw/current/scripts/prefabs/wagboss_missile.lua#L773-L841)
- 标记自身**没有 combat、没有伤害**；它只是把 `x0/z0/arc` 同步给客户端画环。导弹移除时 `OnRemoveEntity` 删除 `inst.ring`。[清理](../../../raw/current/scripts/prefabs/wagboss_missile.lua#L482-L488)
- 专服不创建 `updatelooper`，因此专服上只有网络壳、没有圆环；客户端各自本地建环。[专服分支](../../../raw/current/scripts/prefabs/wagboss_missile.lua#L876-L880)

## 二、黄莺与笼墙的电束

### 宿主与客户端分段

两个注册共用一套"宿主 netvar＋客户端本地分段"的结构（[共享视觉](../../../raw/current/scripts/prefabs/wagdrone_laserwire.lua#L9-L79)）：

- 宿主只加 `CLASSIFIED`＋`notarget`，用 `len`／`rot` 两个 `net_byte` 描述电束长度与角度；`SetBeam(len, rot)` 写 netvar 并强制 dirty（[SetBeam](../../../raw/current/scripts/prefabs/wagdrone_laserwire.lua#L245-L253)）。
- 客户端收到 `beamdirty` 后 `ClearSegs`＋`RefreshSegs`：按 `len:value()/255*15`、`rot:value()/255*360` 重建若干段，每段是本地 `FX`＋`NOCLICK` 实体，父挂宿主，用 `Follower:FollowSymbol(seg.GUID, "marker")` 抬起，播放随机 `beam_1..4`，并叠一层极暗的影子层。[分段](../../../raw/current/scripts/prefabs/wagdrone_laserwire.lua#L32-L79)、[刷新](../../../raw/current/scripts/prefabs/wagdrone_laserwire.lua#L179-L201)
- 专服不建分段；`OnEntitySleep` 清分段与目标任务、`OnEntityWake` 重建（[休眠](../../../raw/current/scripts/prefabs/wagdrone_laserwire.lua#L108-L124)、[挂接](../../../raw/current/scripts/prefabs/wagdrone_laserwire.lua#L291-L294)）。
- **采样缓存不随 `SetBeam` 重算：** `RefreshSegs` 只在 `inst.targetx == nil` 时按当前实体坐标、长度与角度建采样点；`ClearSegs` 只删视觉段与任务、不清 `targetx/targetz`；`SetBeam` 重建视觉与冷却，但伤害仍按旧采样线段。原版用法是连接时配置一次、断开即删；动态移动／转向应重建实体，不能只重复 `SetBeam`。[刷新](../../../raw/current/scripts/prefabs/wagdrone_laserwire.lua#L179-L220)、[清理](../../../raw/current/scripts/prefabs/wagdrone_laserwire.lua#L108-L124)、[SetBeam](../../../raw/current/scripts/prefabs/wagdrone_laserwire.lua#L245-L253)
- 笼墙变体只把动画换成 `follow_marker_cage`，并把死亡通告名覆写为 `wagpunk_cagewall`；目标筛选与伤害同激光束。[笼墙](../../../raw/current/scripts/prefabs/wagdrone_laserwire.lua#L303-L354)

### 伤害扫描

主模拟沿电束按 4 单位间隔布目标点，周期任务每 1 秒扫一次；一旦本帧有目标进入"快速"档，就把周期切到 2 帧（[扫描](../../../raw/current/scripts/prefabs/wagdrone_laserwire.lua#L136-L177)、[布点](../../../raw/current/scripts/prefabs/wagdrone_laserwire.lua#L207-L236)）：

- 每个目标点用 `WagdroneCommon.FindShockTargets(x, z, 6)` 找 `_combat` 目标，排除 `INLIMBO`／`flight`／`invisible`／`notarget`／`noattack`／ghost／shadow 系／`wagdrone`／`wagboss` 等。[筛选](../../../raw/current/scripts/prefabs/wagdrone_common.lua#L3-L10)
- 再要求目标到电束线段的距离小于 `0.1 + 物理半径`（`DistPointToSegmentXYSq`），才结算。[命中条件](../../../raw/current/scripts/prefabs/wagdrone_laserwire.lua#L141-L150)
- 电免疫目标：`SetDefaultDamage(30 × 0.5)` 后 `DoAttack(v, nil, nil, "electric")`，再恢复 30；`DoAttack` 对电免疫跳过电系倍率，实际按 15 结算。非免疫目标同样 `DoAttack(..., "electric")`，但 combat 会再乘 `ELECTRIC_DAMAGE_MULT(1.5)＋ELECTRIC_WET_DAMAGE_MULT(1)×湿度`：干燥 45、满湿 75。两者都会 `PushEventImmediate("electrocute")`（注释说明即使电免疫的玩家仍有电击状态）。[结算](../../../raw/current/scripts/prefabs/wagdrone_laserwire.lua#L151-L160)、[电系倍率](../../../raw/current/scripts/components/combat.lua#L1195-L1213)
- `targets[v]` 记录 `t + 1` 秒，冷却内不重复打同一目标；`DoAttack` 用宿主 `combat`，因此 **attacker 是电束 FX 自身**，不是黄莺。[冷却](../../../raw/current/scripts/prefabs/wagdrone_laserwire.lua#L162-L163)、[combat 归属](../../../raw/current/scripts/prefabs/wagdrone_laserwire.lua#L286-L289)

### 生成点与笼墙缺口

- `wagdrone_laserwire_fx` 唯一生成点：黄莺滚动形态 `ConnectBeams` 在两台距离 ≥4 的 `wagdrone_rolling` 之间取中点生成，`SetBeam(距离, 角度)`；断开时 `DisconnectBeams` 负责 `Remove`。[生成](../../../raw/current/scripts/prefabs/wagdrone_rolling.lua#L71-L100)
- `wagpunk_cagewall_fx` **全快照无 `SpawnPrefab`**；竞技场生成的是不带 `_fx` 的 `wagpunk_cagewall`（`components/wagpunk_arena_manager.lua:412`），两者不是同一注册。因此本注册的"笼墙电束"只有工厂与战斗逻辑，没有实际生成链，按诚实缺口记录。[竞技场](../../../raw/current/scripts/components/wagpunk_arena_manager.lua#L405-L415)

## 三、黄莺与电刑机的电球

### 附着、发射与落地

两个注册共用 `MakeProjectile`（[工厂](../../../raw/current/scripts/prefabs/wagdrone_projectile.lua#L167-L232)），生命周期由 SG 驱动：

| 阶段 | 行为 | 证据 |
|---|---|---|
| 附着 | SG 生成后 `AttachTo(drone)`：父挂无人机、physics 关闭、`Follower:FollowSymbol(parent.GUID, "light")`、播充电声 | [AttachTo](../../../raw/current/scripts/prefabs/wagdrone_projectile.lua#L155-L161) |
| 发射 | `Launch(x,y,z)`：停止跟随、解除父挂、physics 开启、传送到落点上方、`SetMotorVel(0,-15,0)` 下落、开灯、挂 `OnUpdate` | [Launch](../../../raw/current/scripts/prefabs/wagdrone_projectile.lua#L141-L153) |
| 落地 | `y < 0.1`：停止、贴地、`crackle_hit`、`showbase=true`、初始化 `targets`、进入伤害循环；18 帧后 `DisableHits`，动画长度＋2 帧后 `Remove` | [落地](../../../raw/current/scripts/prefabs/wagdrone_projectile.lua#L71-L101) |
| 本地底座 | 客户端在 `showbase` dirty 时创建 `crackle_projection` 本地子 FX（`FX`＋`NOCLICK`、`OnGround`、背景层），动画结束自删；专服不建 | [OnShowBase](../../../raw/current/scripts/prefabs/wagdrone_projectile.lua#L23-L60) |

生成点：黄莺在 `SGwagdrone_flying` 攻击状态第 12 帧生成并 `AttachTo`、第 34 帧 `Launch`；电刑机在 `SGwx78_drone_zap` 第 0 帧生成、写 `caster = owner`、第 22 帧 `Launch`。[黄莺](../../../raw/current/scripts/stategraphs/SGwagdrone_flying.lua#L675-L690)、[电刑机](../../../raw/current/scripts/stategraphs/SGwx78_drone_zap.lua#L514-L530)

### 伤害循环

落地后每帧以落点为中心找目标（[实现](../../../raw/current/scripts/prefabs/wagdrone_projectile.lua#L104-L139)）：

- 搜索半径 `1.5 + 3`，精确命中半径 `1.5 + 目标物理半径`；要求目标未在 `targets`、有效、非 limbo、未死、有 combat、`CanTarget`，若变体提供 `_CheckTarget` 还要满足它。
- 电免疫目标：`SetDefaultDamage(damage × insulated_mult)` 后 `DoAttack(v, nil, nil, "electric")`，再恢复；免疫分支跳过电系倍率，按 15 结算。非免疫直接 `DoAttack(..., "electric")`，再由 combat 乘 1.5＋湿度（干燥 45、满湿 75）。两者都推 `PushEventImmediate("electrocute")`。
- 命中后 `targets[v] = true`；`_OnAttackedTarget` 回调（仅电刑机有）用于分享目标／拉仇恨。
- 伤害归属同样是**弹体 FX 自身 combat**；`caster`（电刑机）只参与目标筛选、友军排除与仇恨，不是 damage attacker。[combat 挂载](../../../raw/current/scripts/prefabs/wagdrone_projectile.lua#L213-L220)

两变体差异：

| 维度 | `wagdrone_projectile_fx`（黄莺） | `wx78_drone_zap_projectile_fx`（电刑机） |
|---|---|---|
| 基础伤害 | `WAGDRONE_FLYING_DAMAGE`＝30 | `SKILLS.WX78.ZAPDRONE_DAMAGE`＝30 |
| 电免疫倍率 | `WAGDRONE_FLYING_INSULATED_DAMAGE_MULT`＝0.5 | `ZAPDRONE_INSULATED_DAMAGE_MULT`＝0.5 |
| 目标筛选 | `FindShockTargets`：`_combat`，排除 shadow／wagdrone／wagboss 等 | `_combat`；PVP 排除表含 companion／wall 等，非 PVP 额外排除 player；`_CheckTarget` 再排除施法者友军 |
| 目标分享／仇恨 | 无 | 20 范围内把 `ATTACK` 放进 `_actiondata` 给 caster；15 范围内给目标推 0 伤害 `attacked` 事件 |
| 名称覆写 | `wagdrone_flying`（黄莺） | `wx78_drone_zap`（电刑机） |

（[调值](../../../raw/current/scripts/tuning.lua#L8672-L8677)、[WX78 调值](../../../raw/current/scripts/tuning.lua#L7542-L7545)、[筛选](../../../raw/current/scripts/prefabs/wagdrone_projectile.lua#L262-L283)）

## 伤害归属与数值

| 注册 | 基础伤害／电系输入 | attacker | 命中上限／窗口 |
|---|---:|---|---|
| `wagboss_missile_target_fx` | 0 | — | 仅是标记；导弹本体的 AOE 另算 |
| `wagdrone_laserwire_fx` | 30；非免疫干燥 45、满湿 75；电免疫 15 | 电束 FX 自身 | 每目标 1 秒一次；沿束分段扫描 |
| `wagpunk_cagewall_fx` | 30；非免疫干燥 45、满湿 75；电免疫 15 | 笼墙电束 FX 自身 | 同左；无生成点 |
| `wagdrone_projectile_fx` | 30；非免疫干燥 45、满湿 75；电免疫 15 | 电球 FX 自身 | 每目标一次；18 帧伤害窗口 |
| `wx78_drone_zap_projectile_fx` | 30；非免疫干燥 45、满湿 75；电免疫 15 | 电球 FX 自身 | 每目标一次；18 帧窗口；caster 只做筛选／仇恨 |

`DoAttack` 的 attacker 取 `self.inst`（FX 实体），不是父无人机；因此击杀归属、经验或掉落若按 attacker 判定，会落在 FX 而不是黄莺／电刑机上。[combat 调用](../../../raw/current/scripts/prefabs/wagdrone_projectile.lua#L124-L128)、[电束调用](../../../raw/current/scripts/prefabs/wagdrone_laserwire.lua#L151-L157)

## 关联机制导航

| 类型／对象 | 本注册如何使用 | 变体、端与条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| `Prefab：wagboss_missile` | 标记的生成者与清理者；落点由导弹 `Launch` 决定 | 目标实体时父挂、目标移除时落回旧点 | [生成](../../../raw/current/scripts/prefabs/wagboss_missile.lua#L379-L424) |
| `Prefab：wagdrone_rolling` | 电束的生成者；两台滚动黄莺之间连接 | 距离 ≥4 才生成；断开即删 | [生成](../../../raw/current/scripts/prefabs/wagdrone_rolling.lua#L71-L100) |
| `Prefab：wagdrone_flying`／`wx78_drone_zap` | 电球的附着宿主与发射者 | SG 帧事件驱动；电刑机写 `caster` | [黄莺 SG](../../../raw/current/scripts/stategraphs/SGwagdrone_flying.lua#L675-L690)、[电刑机 SG](../../../raw/current/scripts/stategraphs/SGwx78_drone_zap.lua#L514-L530) |
| `Prefab：wagpunk_cagewall` | 笼墙本体；与 `_fx` 注册不是同一实体 | 竞技场生成的是无 `_fx` 的本体 | [竞技场](../../../raw/current/scripts/components/wagpunk_arena_manager.lua#L405-L415) |
| `组件：combat` | 四个混合注册的伤害结算；`ignorehitrange` 常开、`KeepTargetFn=false` | 只有主模拟挂；FX 自身作 attacker | [电束 combat](../../../raw/current/scripts/prefabs/wagdrone_laserwire.lua#L286-L289)、[电球 combat](../../../raw/current/scripts/prefabs/wagdrone_projectile.lua#L213-L220) |
| `工具：WagdroneCommon.FindShockTargets` | 电束与黄莺电球的共享目标筛选 | 排除 shadow／wagdrone／wagboss 等 | [实现](../../../raw/current/scripts/prefabs/wagdrone_common.lua#L3-L10) |
| 关联页面：`shadow_despawn` | 同为"混合 FX 自带 combat、attacker 归属 FX"的写法对照 | 那边是位面光束；本页是电系 | [[reference/prefabs/shadow_despawn|暗影消散、舞台影手与暗影憎恶光束]] |

## 典型接入

### 手动拉一条电束

- 前提与选择：只在主模拟调用；`SetBeam` 写 netvar，客户端自动重建分段。电束没有寿命，调用方必须自己 `Remove`（原版由 `DisconnectBeams` 负责）。
- 生效与撤销：`SetBeam(len, rot)` 后立即开始扫描；要停就 `fx:Remove()`，或让它休眠触发 `ClearSegs`。
- 原版依据：[SetBeam](../../../raw/current/scripts/prefabs/wagdrone_laserwire.lua#L245-L253)、[生成](../../../raw/current/scripts/prefabs/wagdrone_rolling.lua#L78-L89)。

```lua
local fx = SpawnPrefab("wagdrone_laserwire_fx")
fx.Transform:SetPosition(mid_x, 0, mid_z)
fx:SetBeam(length, angle) -- 主模拟；客户端经 beamdirty 重建分段
-- 结束或断开时：fx:Remove()
```

- 调整与验证：长度上限 15（net_byte 映射）、角度按 360 映射；目标筛选与伤害在工厂里写死，要改需复制工厂。示例只做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **边界：`wagpunk_cagewall_fx` 没有生成点。** 全快照只有注册；竞技场生成的是不带 `_fx` 的 `wagpunk_cagewall`。需要笼墙电束时必须自己生成 `_fx` 并调它的 `SetBeam`（导出的是 `SetBeam_CageWall`，外部仍按 `SetBeam` 调用）。[缺口](#生成点与笼墙缺口)
- **边界：电束宿主没有 `FX` 标签。** `CLASSIFIED`＋`notarget` 的宿主负责网络与伤害，`FX` 标签在客户端本地分段上；按 FX 标签筛选宿主会漏。[工厂](../../../raw/current/scripts/prefabs/wagdrone_laserwire.lua#L269-L270)
- **边界：电免疫目标仍会被电击。** 电免疫分支把默认伤害降到 15（不再乘电系倍率），不是免伤；非免疫目标按 1.5＋湿度倍率放大。两类目标都会收到 `PushEventImmediate("electrocute")`；注释明确玩家即使电免疫也有电击状态。[电束](../../../raw/current/scripts/prefabs/wagdrone_laserwire.lua#L151-L160)、[电球](../../../raw/current/scripts/prefabs/wagdrone_projectile.lua#L122-L131)
- **边界：电束的 `targets` 冷却表在重建分段时重置。** `RefreshSegs` 每次新建 `targets` 表；`SetBeam` 改长度／角度会重建，等于清掉 1 秒冷却。频繁改束可能重复打同一目标。[重建](../../../raw/current/scripts/prefabs/wagdrone_laserwire.lua#L225-L236)
- **边界：电球落地后先进入 18 帧伤害窗口，再按动画长度＋2 帧删除。** 若动画短于 18 帧，实体先删、`DisableHits` 随实体取消；实际可命中时长取两者较小值（未运行核验）。[落地](../../../raw/current/scripts/prefabs/wagdrone_projectile.lua#L71-L84)
- **边界：电刑机非 PVP 不打玩家。** 非 PVP 的排除表额外含 `player`；PVP 才把玩家纳入候选，且 `_CheckTarget` 仍排除 caster 友军。[筛选](../../../raw/current/scripts/prefabs/wagdrone_projectile.lua#L259-L271)
- **边界：落点标记的圆环是客户端本地实体。** 专服没有圆环；标记实体无 AnimState，不要按"有动画的 FX"给它挂组件。[本地环](../../../raw/current/scripts/prefabs/wagboss_missile.lua#L730-L756)
- **误区：伤害是无人机的。** 四个混合注册的 `DoAttack` 都在 FX 自身 combat 上执行，attacker 记 FX；需要归属无人机时不能直接复用这套工厂。[归属](#伤害归属与数值)

## 验证记录

- 源码审阅：2026-11-05，steam-buildid:26335394（实读 `raw/current/BUILD.txt`）。完整读取 `prefabs/wagdrone_laserwire.lua`（356 行）、`prefabs/wagdrone_projectile.lua`（309 行）；按需读取 `prefabs/wagboss_missile.lua`（Launch L379-L424、环 L703-L862、targetfn L864-L898、清理 L482-L488）、`prefabs/wagdrone_rolling.lua`（L50-L105）、`prefabs/wagdrone_common.lua`（L3-L10）、`stategraphs/SGwagdrone_flying.lua`（L675-L690）、`stategraphs/SGwx78_drone_zap.lua`（L514-L530）、`components/wagpunk_arena_manager.lua`（L405-L415）、`components/combat.lua`（电系倍率 L1195-L1213）、`tuning.lua`（L659-L660、L7541-L7544、L8671-L8676）。全域反查：`wagboss_missile_target_fx` 仅 `wagboss_missile.lua:392` 生成；`wagdrone_laserwire_fx` 仅 `wagdrone_rolling.lua:83` 生成；`wagpunk_cagewall_fx` **0 生成**；`wagdrone_projectile_fx` 仅 `SGwagdrone_flying.lua:680`；`wx78_drone_zap_projectile_fx` 仅 `SGwx78_drone_zap.lua:518`。台账分诊结论（组 49 的 5 注册、1 纯 FX＋4 混合、生成点与缺口）复核属实；补充实核：落点环为客户端本地实体、电束宿主无 FX 标签、电束冷却表随重建重置而采样缓存不重置、电球 attacker 为 FX 自身、电刑机非 PVP 排除玩家；四注册基础伤害 30 经 electric 倍率后非免疫干燥 45／满湿 75，电免疫分支 15。
- 名称核验：`languages/chinese_s.po` 定向检索——`WAGBOSS_MISSILE`＝"热追踪导弹"（[L351379-L351382](../../../raw/current/scripts/languages/chinese_s.po#L351379-L351382)）、`WAGDRONE_FLYING`＝"黄莺"（[L351424-L351427](../../../raw/current/scripts/languages/chinese_s.po#L351424-L351427)）、`WX78_DRONE_ZAP`＝"电刑机"（[L352679-L352682](../../../raw/current/scripts/languages/chinese_s.po#L352679-L352682)）、`WAGPUNK_CAGEWALL`＝"动能场扩展器"（[L351454-L351457](../../../raw/current/scripts/languages/chinese_s.po#L351454-L351457)）；5 个 FX 注册名无 `STRINGS.NAMES` 键。
- 分档说明：`size_class: complex`——5 注册跨 3 文件，包含"网络标记＋客户端本地环""宿主 netvar＋客户端分段＋服务端扫描""共享弹体工厂＋两套目标筛选"三条独立生命周期与两套伤害归属，需要整体说明；不是按字节升档。
- 未覆盖：热追踪导弹本体飞行／AOE／落点选择、黄莺与电刑机 AI 与技能全貌、`wagpunk_cagewall` 本体、电球动画实际帧长与 18 帧窗口的相对关系、电束分段动画包内容；未运行游戏。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 结果见交付记录。
- 游戏运行：未运行；`validation` 保持 `source-reviewed`。

## 相关链接

- [[reference/prefabs/shadow_despawn|暗影消散、舞台影手与暗影憎恶光束]]（混合 FX 自带 combat 的对照页）
- 返回 [[reference/prefabs/fx-combat-index|攻击、弹道与落点]]（WAGboss／无人机）
- 2026-11-08 build 26474873：本页随版本迁移升版（锚点迁移 2 处，tuning.lua 常量段 +1 行；CRLF 页按原字节格式宿主编辑），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-01）。
