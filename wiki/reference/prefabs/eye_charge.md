---
title: "外围载体：眼弹、诱鱼区、黏着块与速爆弹（eye_charge 等四注册）"
created: "2026-11-04"
updated: "2026-11-04"
type: prefab
summary: "四个外围玩法载体的原版实现：眼睛炮塔眼弹、鱼食诱鱼区、恶液小黏块与绝望螨速爆弹；逐注册核验生成点、父挂、端与清理，并分清伤害／减速／吸附结算与 FX 表现。"
aliases: ["eye_charge", "chum_aoe", "gelblob_small_fx", "fused_shadeling_quickfuse_bomb", "眼弹", "眼睛炮塔弹体", "诱鱼区", "鱼食诱鱼区", "恶液小黏块", "黏着块", "绝望螨速爆弹", "速爆弹", "外围载体", "外围玩法载体"]
tags: [prefab, fx, server-client]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/eye_charge.lua", "prefabs/eyeturret.lua", "prefabs/chum.lua", "prefabs/chum_aoe.lua", "prefabs/gelblob.lua", "prefabs/gelblob_attach_fx.lua", "prefabs/messagebottle.lua", "prefabs/slingshotammo.lua", "stategraphs/SGshrouden.lua", "prefabs/fused_shadeling_bomb.lua", "stategraphs/SGfused_shadeling.lua", "components/projectile.lua", "components/schoolspawner.lua", "components/timer.lua", "components/groundshadowhandler.lua", "components/entitytracker.lua", "components/weapon.lua", "components/combat.lua", "brains/oceanfishbrain.lua", "entityscript.lua", "tuning.lua", "prefablist.lua", "languages/chinese_s.po"]
size_class: complex
reliability: verified
validation: source-reviewed
status: active
---

# 外围载体：眼弹、诱鱼区、黏着块与速爆弹（eye_charge 等四注册）

## 快速判断

- 职责与适用：本页收 4 个文件各 1 个主注册——`eye_charge` 是眼睛炮塔（官方名，[po](../../../raw/current/scripts/languages/chinese_s.po#L342531-L342532)）的弹体；`chum_aoe` 是鱼食投掷物（官方名，[po](../../../raw/current/scripts/languages/chinese_s.po#L341566-L341567)）落海后的诱鱼区；`gelblob_small_fx` 是恶液（官方名，[po](../../../raw/current/scripts/languages/chinese_s.po#L343146-L343147)）的小黏块；`fused_shadeling_quickfuse_bomb` 是绝望螨（`fused_shadeling_bomb` 官方名，[po](../../../raw/current/scripts/languages/chinese_s.po#L343096-L343097)）的速爆弹。四项都不是单一纯 FX：分别承担伤害投射、鱼群生成、接触吸附与 AOE 玩法，FX 只是各自的一段表现。[chum_aoe 预载](../../../raw/current/scripts/prefablist.lua#L293) [eye_charge 预载](../../../raw/current/scripts/prefablist.lua#L400)
- 尺寸档：`size_class: complex`。4 文件 4 注册、4 套生成与清理模型，并含 combat、schoolspawner、AOE 三种结算归属，需要整体保留；不是按源码行数升档。
- 名称：四个注册名本身都没有 `STRINGS.NAMES` 键；`gelblob_small_fx` 与速爆弹只在构造时 `SetPrefabNameOverride` 借宿主显示名（恶液／绝望螨），实际 `prefab` 字段不变；标题中的眼弹、诱鱼区、黏着块、速爆弹是效果描述，不是官方译名。[名称覆盖](../../../raw/current/scripts/entityscript.lua#L846-L849) [小黏块](../../../raw/current/scripts/prefabs/gelblob.lua#L860) [速爆弹](../../../raw/current/scripts/prefabs/fused_shadeling_bomb.lua#L354)
- 优先使用：查“伤害／减速／吸附由谁结算”先看下方“结算归属”；查工厂、生成点、父挂与清理看“注册与构造”；纯表现复用分别看 death_fx、焦痕、命中音等已有页。
- 修改边界：四者端分界都在 `SetPristine` 后早退；主模拟才挂组件、任务与生成点。`eye_charge` 的伤害不在弹体，`chum_aoe` 不定身，`gelblob_small_fx` 不直接减速，速爆弹的伤害不在 `death_fx`。

### 结算归属

- `eye_charge`：伤害由眼睛炮塔 `combat` 读取隐藏武器伤害结算（`TUNING.EYETURRET_DAMAGE = 65`）；`projectile:Hit` 先 `combat:DoAttack`，弹体仍有效时之后才调 `OnHit` 生成命中音壳。弹体只负责飞行、命中位置与自删。[武器](../../../raw/current/scripts/prefabs/eyeturret.lua#L134-L152) [伤害值](../../../raw/current/scripts/tuning.lua#L2766-L2770) [Hit 顺序](../../../raw/current/scripts/components/projectile.lua#L236-L266)
- `chum_aoe`：不结算伤害、定身或减速；工厂没有 combat／health／locomotor／rooted，只有 timer、动画、声效与生成任务，位置固定在落点。它挂 `chum` 标签吸引鱼，并周期生成可食用的 `chumpiece`。[工厂](../../../raw/current/scripts/prefabs/chum_aoe.lua#L130-L181)
- `gelblob_small_fx`：自身不结算伤害或减速；接触判定在它的 `OnUpdateProximity`，吸附、`gelblobbed` 标签与减速由 `gelblob_attach_fx` 结算（普通玩家走 carefulwalker，骑乘／变形／生物走 locomotor 乘数）；小黏块只播挤压／装载／回吸或消退表现。[接触生成](../../../raw/current/scripts/prefabs/gelblob.lua#L146-L163) [吸附层](../../../raw/current/scripts/prefabs/gelblob_attach_fx.lua#L279-L308)
- `fused_shadeling_quickfuse_bomb`：伤害由同文件 `do_explosion_effect` 的 AOE 结算（50 物理＋20 位面、半径 3、排除 `shadow_aligned` 等），速爆弹只负责投掷物理、固定引信与爆炸时机；`fused_shadeling_bomb_death_fx` 与 `fused_shadeling_bomb_scorch` 只负责表现。[AOE](../../../raw/current/scripts/prefabs/fused_shadeling_bomb.lua#L93-L106) [数值](../../../raw/current/scripts/tuning.lua#L7682-L7684)

表现分工：`eye_charge` 播 `idle` 飞行，命中处只生成有声音、无 AnimState 的壳；`chum_aoe` 播 `fish_chum` 系列、维持 `spore_loop` 与地面层，`chumpiece` 无可视动画；`gelblob_small_fx` 播 `blob_idle_med`、挤压、splash／erode，吸附覆盖层另由 `gelblob_attach_fx` 播；速爆弹播 `ball_grow` 与地面投影，爆炸动画与音效在 `death_fx`，焦痕是独立贴地贴片。[弹体动画](../../../raw/current/scripts/prefabs/eye_charge.lua#L31-L35) [诱鱼区动画](../../../raw/current/scripts/prefabs/chum_aoe.lua#L138-L153) [小黏块动画](../../../raw/current/scripts/prefabs/gelblob.lua#L856-L858) [速爆弹动画](../../../raw/current/scripts/prefabs/fused_shadeling_bomb.lua#L350-L352)

## 注册与构造

下表只列本页四个主注册。同文件另有附属注册：`eye_charge_hit`（命中音效壳，见 [[reference/prefabs/groundpoundring_fx|冲击环与爆点表现页]]）、`chumpiece`、`gelblob` 本体及 `gelblob_back_fx`／`gelblob_item_fx`、`fused_shadeling_bomb` 本体及 death_fx／焦痕；本页只在直接相关处说明，不展开其余工厂。

| 注册名／变体 | 工厂与初始化分界 | 差异、生成与证据 |
|---|---|---|
| `eye_charge` | `fn`：Transform＋AnimState＋SoundEmitter＋Network；`MakeInventoryPhysics` 后 `RemovePhysicsColliders`、FourFaced、bank/build `eyeball_turret_attack` 播 `idle`；加 `projectile` 标签 → `SetPristine` → 客户端早退 → 主模拟 `persists=false`＋projectile（速度 30、非追踪、命中距离 2、`OnHit` 生成命中壳、Miss 自删、Thrown 后监听 `animover` 并在 0.3 秒后自删）。 | 生成点全树只有 `eyeturret` 的隐藏武器 `SetProjectile("eye_charge")`；无父挂，发射时定位在炮塔位置，命中后独立定位生成 `eye_charge_hit`。炮塔伤害 65、攻击距离 15，武器射程 15～19。[工厂](../../../raw/current/scripts/prefabs/eye_charge.lua#L20-L57) [弹体配置](../../../raw/current/scripts/prefabs/eye_charge.lua#L48-L54) [炮塔](../../../raw/current/scripts/prefabs/eyeturret.lua#L134-L152) [注册](../../../raw/current/scripts/prefabs/eye_charge.lua#L100-L101) |
| `chum_aoe` | `fn`：Transform＋AnimState＋SoundEmitter＋Network；bank/build `fish_chum` 播 `fish_chum_base_pre`→`fish_chum_base_idle` 循环，OnGround／`LAYER_BELOW_GROUND`／sort 3；`FX`／`NOCLICK`／`chum` 标签与 `spore_loop` 循环音 → `SetPristine` → 客户端早退 → 主模拟：剩余尝试 5、0.5 秒周期生成 chumpiece、timer `disperse` 20 秒、`timerdone`／`onremove` 监听、2.5＋随机 2.5 秒后尝试生成鱼群、`OnSave`／`OnLoad`。 | 生成点全树只有 `chum.lua` 的 `OnHit`：落海时生成 `chum_aoe`＋`SetThrower(attacker)`，非海面只落回 `chum`；无父挂，位置固定在落点。同文件另注册 `chumpiece`（edible、`secondaryfoodtype=MEAT`、无 AnimState、`persists=false`）。[工厂](../../../raw/current/scripts/prefabs/chum_aoe.lua#L130-L181) [常量](../../../raw/current/scripts/prefabs/chum_aoe.lua#L11-L23) [生成](../../../raw/current/scripts/prefabs/chum.lua#L19-L33) [注册](../../../raw/current/scripts/prefabs/chum_aoe.lua#L213-L214) |
| `gelblob_small_fx` | `smallfn`：Transform＋AnimState＋DynamicShadow＋SoundEmitter＋Network；影 2×1.5、物理 mass 10／friction 0／damping 5／CHARACTERS→WORLD／capsule .5,1、`canbebottled`、bank/build `gelblob` 播 `blob_idle_med` 循环、`SetPrefabNameOverride("gelblob")` → `SetPristine` → 客户端早退 → 主模拟随机帧、entitytracker＋timer、接触半径 .4／脱离 1、`_collectors` 与 `startlongaction` 监听、暴露 SetLifespan／ReleaseFromBottle／ReleaseFromAmmoAfflicted／Toss／KillFX／OnContactChanged。 | 4 个生成来源（5 处调用）：主恶液掉级时 `OnHealthDelta` 生成并 `TrackEntity("mainblob")`＋`Toss`；黏液瓶落地 `SetLifespan(TUNING.TOTAL_DAY_TIME)`＋`ReleaseFromBottle`；弹弓黏液弹 `SetLifespan(60)`＋`ReleaseFromAmmoAfflicted`；暗影憎恶死亡 SG 两帧共 7 个短寿命 `Toss`。自身无父挂；吸附层才由 `gelblob_attach_fx` 挂到目标。[工厂](../../../raw/current/scripts/prefabs/gelblob.lua#L835-L900) [主恶液生成](../../../raw/current/scripts/prefabs/gelblob.lua#L405-L408) [瓶装](../../../raw/current/scripts/prefabs/messagebottle.lua#L385-L400) [弹弓](../../../raw/current/scripts/prefabs/slingshotammo.lua#L665-L676) [憎恶](../../../raw/current/scripts/stategraphs/SGshrouden.lua#L1224-L1250) [注册](../../../raw/current/scripts/prefabs/gelblob.lua#L968) |
| `fused_shadeling_quickfuse_bomb` | `quickfuse_fn`：Transform＋AnimState＋SoundEmitter＋Network；`MakeInventoryPhysics(10,.2)`＋friction .3、groundshadowhandler 1.5×1.0、FourFaced、hostile／monster／notraptrigger／shadow／shadow_aligned、bank/build `fused_shadeling_bomb` 播 `ball_grow` 循环、`SetPrefabNameOverride("fused_shadeling_bomb")` → `SetPristine` → 客户端早退 → 主模拟 inspectable＋`DoTaskInTime(QUICKFUSE_TIME, do_quick_explode)`＋`persists=false`；到点先结算 AOE、生成 death_fx 与焦痕，再 `Remove`。 | 两个生成来源（本体 3 个＋SG 1 个）：主炸弹 `do_full_explode` 在 0.2 秒内分 3 次投出（速度 4＋随机 1、上抛 8＋随机 2、随机角±PI/6）；融合暗影（官方名「熔合暗影」，[po](../../../raw/current/scripts/languages/chinese_s.po#L343089-L343092)）的 `SGfused_shadeling` 在 teleport 第 14 帧投 1 个（速度 2.5＋随机 1、上抛 12）。无父挂；引信为固定 35 帧（`GROW_TIME` 70 帧的一半，约 1.17 秒），不是落地触发。[工厂](../../../raw/current/scripts/prefabs/fused_shadeling_bomb.lua#L328-L371) [快速投掷](../../../raw/current/scripts/prefabs/fused_shadeling_bomb.lua#L108-L119) [SG 生成](../../../raw/current/scripts/stategraphs/SGfused_shadeling.lua#L410-L424) [注册](../../../raw/current/scripts/prefabs/fused_shadeling_bomb.lua#L450-L453) |

## 关联机制导航

只列与本页四注册直接相关的工厂、生成者、组件与关联 Prefab；目标内部机制链接各主要页，不递归展开。

### 投射与诱鱼（eye_charge／chum_aoe）

| 类型／对象 | 本 Prefab 如何使用 | 端、条件与生效 | Wiki／源码入口及证据 |
|---|---|---|---|
| 生成者：`eyeturret` 隐藏武器 | `EquipWeapon` 建非联网武器实体，`SetDamage(combat.defaultdamage)`、`SetRange(attackrange, attackrange+4)`、`SetProjectile("eye_charge")`；炮塔 1 秒后装备，攻击时 `combat` 走 `LaunchProjectile`。 | 主模拟；`weapon.CanRangedAttack` 为真时由 combat 发射，不落近战伤害。 | [装备](../../../raw/current/scripts/prefabs/eyeturret.lua#L134-L152) [炮塔 combat](../../../raw/current/scripts/prefabs/eyeturret.lua#L279-L294)；宿主 Wiki 待建 |
| 组件：`projectile` | 工厂设速度／命中距离／非追踪，并挂 OnHit／OnMiss／OnThrown；`Hit` 先 `DoAttack`，弹体仍有效时再 `onhit`；`onhit` 不是净伤害确认。 | 主模拟；弹体只在 `SetPristine` 后加组件。 | [[reference/components/projectile#伤害与发射分工|projectile 伤害与发射分工]]；[Hit](../../../raw/current/scripts/components/projectile.lua#L236-L266) |
| 关联 Prefab：`eye_charge_hit` | 命中时在弹体位置生成，0.5 秒自删；无 AnimState，本地音效代理只播 `shotexplo`；Miss 不生成。 | 主模拟生成主壳；非专服在壳位置本地播声。 | [[reference/prefabs/groundpoundring_fx|冲击环、爆点与碎块表现特效]]；[OnHit](../../../raw/current/scripts/prefabs/eye_charge.lua#L7-L10) |
| 生成者：`chum` 投掷物 | `oceanthrowable` 的 complexprojectile 命中时：非海面落回 `chum`，海面生成 `splash_green` 与 `chum_aoe`，并 `SetThrower(attacker)`。 | 主模拟；仅海面落点生成诱鱼区。 | [OnHit](../../../raw/current/scripts/prefabs/chum.lua#L19-L33) [复杂弹](../../../raw/current/scripts/prefabs/chum.lua#L76-L81) |
| 组件：`timer` | 工厂 `StartTimer("disperse", 20)`，到期播 `fish_chum_base_pst`、摘 `chum` 标签、`persists=false`，2 秒后 Remove；`timerdone` 处理。 | 主模拟；timer 组件自行保存剩余时间。 | [[reference/components/timer#关键接口|timer]]；[定时](../../../raw/current/scripts/prefabs/chum_aoe.lua#L170-L174) [收尾](../../../raw/current/scripts/prefabs/chum_aoe.lua#L25-L38) [保存](../../../raw/current/scripts/components/timer.lua#L122-L140) |
| 组件：`schoolspawner`／鱼脑 | 首次在 2.5＋随机 2.5 秒后调 `schoolspawner:SpawnSchool`；附近 30 内 `oceanfish`＋`oceanfishable` 少于 15 才尝试，返回 0／nil 时每 0.5 秒重试；剩余次数初值 5，但 `retry = remaining <= 0 and false or retry` 在 retry 为 true 时仍为 true，当前表达式不能在耗尽时停止重试。鱼脑以 `chum` 标签、16 距离靠近诱鱼区。 | 主模拟；`thrower` 只作为 SpawnSchool 的 target 传入（影响伴随生物判定），不决定鱼群种类。 | [[reference/components/schoolspawner#典型接入|原版诱鱼剂直接生成并重试]]；[生成鱼群](../../../raw/current/scripts/prefabs/chum_aoe.lua#L49-L74) [SpawnSchool](../../../raw/current/scripts/components/schoolspawner.lua#L206-L260) [鱼脑](../../../raw/current/scripts/brains/oceanfishbrain.lua#L219-L220) |
| 关联 Prefab：`chumpiece` | 每 0.5 秒在海面落点半径 3 内生成，最多 7 个；被移除时若宿主仍 `persists` 立即补一个。可食用、无 AnimState、`persists=false`。 | 主模拟；仅落点为海面时生成。 | [周期生成](../../../raw/current/scripts/prefabs/chum_aoe.lua#L88-L105) [清理](../../../raw/current/scripts/prefabs/chum_aoe.lua#L76-L86) [工厂](../../../raw/current/scripts/prefabs/chum_aoe.lua#L184-L211) |
| 资源 | `anim/eyeball_turret_attack.zip`（弹体）与 `anim/fish_chum.zip`（诱鱼区）；命中壳无动画资源。 | 仅注册字段引用；未解包检查帧数与视觉。 | [弹体资产](../../../raw/current/scripts/prefabs/eye_charge.lua#L1-L5) [诱鱼区资产](../../../raw/current/scripts/prefabs/chum_aoe.lua#L1-L4) |

### 黏着块（gelblob_small_fx）

| 类型／对象 | 本 Prefab 如何使用 | 端、条件与生效 | Wiki／源码入口及证据 |
|---|---|---|---|
| 生成者：`gelblob` 主恶液 | 掉级时按损失等级生成若干块，设 `mainblob` 追踪并 `Toss`；块在 1 单位内回到仍存活的主恶液时 `Absorb` 并播 `blob_attach_middle_pst` 自删。 | 主模拟；`OnHealthDelta` 受伤后；主恶液死亡后改走 `KillFX`。 | [掉级生成](../../../raw/current/scripts/prefabs/gelblob.lua#L405-L408) [回吸](../../../raw/current/scripts/prefabs/gelblob.lua#L658-L693) |
| 生成者：黏液瓶 | `GelBlobBottle_OnHit` 在陆地／平台生成小黏块，`SetLifespan(TUNING.TOTAL_DAY_TIME)` 后 `ReleaseFromBottle`；空瓶装回时按实际 `prefab` 识别并删除。 | 主模拟；瓶装动作与消耗见 bottler 页。 | [[reference/components/bottler#典型接入|空瓶装入小黏液]]；[释放](../../../raw/current/scripts/prefabs/messagebottle.lua#L385-L400) [装回](../../../raw/current/scripts/prefabs/messagebottle.lua#L212-L228) |
| 生成者：`slingshotammo_gelblob` | 命中可通行点时生成小黏块，`SetLifespan(TUNING.SLINGSHOT_AMMO_GELBLOB_DURATION)`（60 秒）并 `ReleaseFromAmmoAfflicted`：摘 `canbebottled`、加 `NOCLICK`、`persists=false`、启动快速接触检测。 | 主模拟；不可通行点且 `TheWorld.has_ocean` 时改生成海面涟漪。 | [生成](../../../raw/current/scripts/prefabs/slingshotammo.lua#L665-L676) [时长](../../../raw/current/scripts/tuning.lua#L4862) |
| 生成者：`SGshrouden` 死亡 | 死亡帧 1 与帧 3 各取一半索引，共 7 个块，`Toss(dist, angle, 0.6*math.random(), true)`；短寿命结束后消退。 | 主模拟；暗影憎恶死亡状态。 | [死亡撒块](../../../raw/current/scripts/stategraphs/SGshrouden.lua#L1224-L1250) |
| 组件：`entitytracker`／`timer` | entitytracker 存 `"mainblob"` 供回吸；timer 存 `"lifespan"`，到期只把 `persists` 置 false（睡眠中直接 Remove），无 mainblob 的瓶／弹弓／憎恶路径随后由回吸循环转 `KillFX` 消退。 | 主模拟；`SetLifespan` 可被瓶／弹弓／憎恶三路复用。 | [[reference/components/entitytracker#关键接口|entitytracker]]、[[reference/components/timer#关键接口|timer]]；[生命周期](../../../raw/current/scripts/prefabs/gelblob.lua#L755-L768) |
| 关联 Prefab：`gelblob_attach_fx` | 接触圈内生成并 `SetupBlob(小黏块, 目标)`；吸附层挂到目标、加 `gelblobbed` 标签并结算减速；脱离／死亡／睡眠时 KillFX 或直接 Remove。 | 主模拟；普通玩家走 carefulwalker，骑乘／变形／生物走 locomotor 乘数；同目标多团经共享表防叠加。 | [[reference/prefabs/gelblob_attach_fx#关联机制导航|吸附层与减速归属]]；[接触生成](../../../raw/current/scripts/prefabs/gelblob.lua#L146-L163) [SetupBlob](../../../raw/current/scripts/prefabs/gelblob_attach_fx.lua#L279-L308) |
| 资源 | `anim/gelblob.zip`；小黏块、主恶液、背层与掉落层共用同一 bank／build。 | 仅注册字段引用；未解包检查动画。 | [资产](../../../raw/current/scripts/prefabs/gelblob.lua#L1-L4) |

### 速爆弹（fused_shadeling_quickfuse_bomb）

| 类型／对象 | 本 Prefab 如何使用 | 端、条件与生效 | Wiki／源码入口及证据 |
|---|---|---|---|
| 生成者：`fused_shadeling_bomb` 本体 | `do_full_explode` 先在本体位置结算一次 AOE，再分 3 次投出速爆弹，立即 `RemoveFromScene`、0.2 秒后 `Remove`；速爆弹各自按固定引信结算自己的 AOE。 | 主模拟；本体由 timer `start_explosion` 触发，本体 Wiki 待建。 | [完整引爆](../../../raw/current/scripts/prefabs/fused_shadeling_bomb.lua#L123-L140) [投掷](../../../raw/current/scripts/prefabs/fused_shadeling_bomb.lua#L108-L119) |
| 生成者：`SGfused_shadeling` teleport | teleport 第 14 帧按战斗目标朝向（无目标则随机）生成 1 个，`Physics:Teleport` 到自身位置后 `SetVel` 抛出，并在弹体上播 `bomb_spawn`。 | 主模拟；仅在 teleport 状态帧事件。 | [SG 帧事件](../../../raw/current/scripts/stategraphs/SGfused_shadeling.lua#L410-L424) |
| 结算：`do_explosion_effect` | 以 `_combat` 为 must tag、排除 `shadow_aligned`／`DECOR`／`INLIMBO`／`NOCLICK`／`FX`／`playerghost`，半径 3 内逐个 `combat:GetAttacked(inst, 50, nil, nil, {planar=20})`；同函数先生成 death_fx、后生成焦痕。 | 主模拟；本体与速爆弹共用该函数，伤害来源是当前弹体实体。 | [AOE 与排除标签](../../../raw/current/scripts/prefabs/fused_shadeling_bomb.lua#L93-L106) [数值](../../../raw/current/scripts/tuning.lua#L7682-L7684) |
| 组件：`groundshadowhandler` | 工厂加 1.5×1.0 地面投影；投影实体是非联网、CLASSIFIED/NOCLICK、随高度缩放，纯表现。 | 双端工厂都会添加（在 `SetPristine`/master 早退之前）；生成的 shadow 实体非联网。 | [[reference/components/groundshadowhandler|groundshadowhandler]]；[添加](../../../raw/current/scripts/prefabs/fused_shadeling_bomb.lua#L339-L340) |
| 关联 Prefab：death_fx／焦痕 | `do_explosion_effect` 生成 `fused_shadeling_bomb_death_fx`（动画＋爆炸音）与 `fused_shadeling_bomb_scorch`（贴地淡出焦痕）；两者都不结算伤害。 | 主模拟生成；death_fx 走 `animover` 自删，焦痕按 netvar 帧淡出。 | [[reference/prefabs/groundpoundring_fx|death_fx]]、[[reference/prefabs/crabking_claw_shadow|焦痕]]；[生成](../../../raw/current/scripts/prefabs/fused_shadeling_bomb.lua#L95-L106) [death_fx 音](../../../raw/current/scripts/prefabs/fused_shadeling_bomb.lua#L290-L293) |
| 资源 | `anim/fused_shadeling_bomb.zip`（bank／build 同名）；本体、death_fx 与速爆弹共用该资源，焦痕另用 `anim/burntground.zip`。 | 仅注册字段引用；未解包检查视觉。 | [资产](../../../raw/current/scripts/prefabs/fused_shadeling_bomb.lua#L1-L6) [焦痕资产](../../../raw/current/scripts/prefabs/fused_shadeling_bomb.lua#L374-L377) |

## 典型接入

### 调整眼弹飞行速度，不碰伤害

- 前提与选择：主模拟、已装眼睛炮塔；只想改飞行手感时用 `AddPrefabPostInit("eye_charge")`，不要改 `eyeturret` 的 combat 伤害。弹体每次发射都会新建，PostInit 对每个实例生效。
- 生效与撤销：`SetSpeed` 只改当前弹体速度；已在飞行的弹体在 `IsThrown()` 时立即重设马达速度，新弹体按新速度。移除模组后新弹恢复原版 30。
- 原版依据：速度由工厂 `SetSpeed(30)` 设定，伤害与命中距离分别在炮塔 combat 与 `SetHitDist(2)`；`Hit` 的伤害先于 `onhit`。[弹体配置](../../../raw/current/scripts/prefabs/eye_charge.lua#L48-L54) [Hit 顺序](../../../raw/current/scripts/components/projectile.lua#L236-L266)

```lua
-- modmain.lua：只改 eye_charge 飞行速度，不改伤害、命中距离或命中音
AddPrefabPostInit("eye_charge", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    if inst.components.projectile ~= nil then
        inst.components.projectile:SetSpeed(20)
    end
end)
```

- 调整与验证：20 是物理马达速度，不是伤害或射程；命中距离仍为 2，伤害仍由炮塔 combat 结算（默认 65）。未运行验证实际手感。

## 常见陷阱与限制

- **误区：命中音壳是伤害来源**：`eye_charge` 的伤害在 `projectile:Hit` 里由炮塔 combat 先结算，`OnHit` 只生成无 AnimState 的 `eye_charge_hit` 并在 0.5 秒后删除；Miss 完全不生成 FX。[Hit](../../../raw/current/scripts/components/projectile.lua#L236-L266) [命中壳](../../../raw/current/scripts/prefabs/eye_charge.lua#L74-L98)
- **边界：弹体还受动画寿命限制**：`OnThrown` 监听 `animover`，动画结束后 0.3 秒自删；工厂没有 `SetRange`，所以实际可飞距离同时受动画长度与命中／Miss 影响，动画长度未核验。[OnAnimOver](../../../raw/current/scripts/prefabs/eye_charge.lua#L12-L18)
- **误区：`chum_aoe` 会定身或减速**：它没有 combat／health／locomotor／rooted，位置固定只是因为没有 Physics；`timer` 是 20 秒寿命，不是定身计时器。鱼只是被 `chum` 标签吸引靠近。[工厂](../../../raw/current/scripts/prefabs/chum_aoe.lua#L130-L181) [鱼脑](../../../raw/current/scripts/brains/oceanfishbrain.lua#L219-L220)
- **误区：鱼群按周期反复生成**：`SpawnFishSchool` 只在初始 2.5＋随机 2.5 秒后启动一次，失败每 0.5 秒重试，但计数耗尽不会把 retry 置 false（`and false or retry` 仍返回 true），当前代码没有“最多 5 次”上限；成功令 retry=false 或实体删除才会结束该重试链。周期 0.5 秒的是 `chumpiece`，不是鱼群。[鱼群](../../../raw/current/scripts/prefabs/chum_aoe.lua#L49-L74) [chumpiece](../../../raw/current/scripts/prefabs/chum_aoe.lua#L88-L105)
- **边界：读档会经 `fn` 重建任务**：`OnSave`／`OnLoad` 只恢复剩余尝试次数（剩余为 0 时取消鱼群任务），但读档时 prefab `fn` 仍会重新执行，`chum_aoe.lua` 无条件重建 0.5 秒 `chumpiece` 周期任务与 `_spawn_fish_school_task`；因此读档后仍会补块，剩余尝试 >0 时还会再次尝试生成鱼群。实际时序未运行验证。[存读](../../../raw/current/scripts/prefabs/chum_aoe.lua#L111-L128) [任务重建](../../../raw/current/scripts/prefabs/chum_aoe.lua#L168-L176)
- **误区：小黏块自己减速**：它没有 combat／health，也不设速度乘数；接触判定只负责挑目标与生成 `gelblob_attach_fx`，吸附、`gelblobbed` 标签和减速都在吸附层完成。[接触生成](../../../raw/current/scripts/prefabs/gelblob.lua#L146-L163) [吸附层](../../../raw/current/scripts/prefabs/gelblob_attach_fx.lua#L237-L308)
- **边界：名称覆盖不改 `prefab`**：`SetPrefabNameOverride("gelblob")` 只影响显示名；装瓶回调仍按 `target.prefab == "gelblob_small_fx"` 识别，不能用名称覆盖代替实际注册名。[名称覆盖](../../../raw/current/scripts/entityscript.lua#L846-L849) [装回判断](../../../raw/current/scripts/prefabs/messagebottle.lua#L212-L228)
- **边界：睡眠时吸附层被直接 Remove**：小黏块睡眠走共享 `OnEntitySleep`，对已吸附 FX 直接 `Remove`，不播 KillFX 的 splash；减速仍由吸附层 `OnRemoveEntity` 注销。[睡眠](../../../raw/current/scripts/prefabs/gelblob.lua#L344-L353) [吸附层页面](../../../raw/current/scripts/prefabs/gelblob_attach_fx.lua#L338-L360)
- **误区：速爆弹落地才爆**：`DoTaskInTime(QUICKFUSE_TIME, do_quick_explode)` 是固定 35 帧引信，落地与否都在同一时刻爆炸；它没有落地回调或碰撞触发。[引信](../../../raw/current/scripts/prefabs/fused_shadeling_bomb.lua#L321-L328) [炸](../../../raw/current/scripts/prefabs/fused_shadeling_bomb.lua#L365)
- **边界：速爆弹不是可击杀怪**：工厂只加 inspectable 与 groundshadowhandler，没有 health／combat；`hostile`／`monster` 标签参与目标与分类筛选，但不等于可被击杀，AOE 也会排除 `shadow_aligned`。[标签与工厂](../../../raw/current/scripts/prefabs/fused_shadeling_bomb.lua#L344-L371) [AOE 排除](../../../raw/current/scripts/prefabs/fused_shadeling_bomb.lua#L93-L106)
- **边界：爆炸表现与伤害分成三个注册**：`death_fx` 播动画与爆炸音、焦痕贴地淡出，二者都不结算伤害；伤害只在 `do_explosion_effect`。主炸弹本体也调用同一函数，所以一次本体引爆共有本体 1 次＋速爆弹 3 次 AOE 结算。[完整引爆](../../../raw/current/scripts/prefabs/fused_shadeling_bomb.lua#L123-L140) [AOE](../../../raw/current/scripts/prefabs/fused_shadeling_bomb.lua#L93-L106)

## 验证记录

- 源码审阅：2026-11-04，Steam buildid 26335394。覆盖 4 个主文件全文与 4 个注册的工厂／端分界／生成点／父挂／清理；在 `raw/current/scripts` 全树按 `eye_charge`、`chum_aoe`、`gelblob_small_fx`、`fused_shadeling_quickfuse_bomb` 完整名反查，并补查 `chum`、`gelblob`、`fused_shadeling`、`messagebottle`、`slingshotammo`、`shrouden` 宿主与依赖；`prefablist.lua` 确认加载入口。名称按 `languages/chinese_s.po` 实际命中核验：眼睛炮塔／鱼食／恶液／绝望螨／熔合暗影；四个注册名本身无 NAMES 键。未覆盖：动画帧数与视觉、游戏运行、鱼群实际读档行为、`chumpiece` 的具体食用者、主炸弹本体 SG 的完整流程。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict 与 Lua 探针结果见交付报告；`size_class: complex` 为 4 文件 4 注册与三种结算归属保留。

返回 [[reference/prefabs/fx-combat-index|攻击、弹道与落点]]、[[reference/prefabs/index|Prefab 索引]]；其他外围表现见 [[reference/prefabs/fx-index|FX 主题总索引]]。
