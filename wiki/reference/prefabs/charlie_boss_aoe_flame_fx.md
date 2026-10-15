---
title: "藏影女王 Boss 战表现特效族（charlie_boss_aoe_flame_fx 等）"
created: "2026-11-04"
updated: "2026-11-04"
type: prefab
summary: "藏影女王战的七个注册：召唤落焰、minion 命中火花、反射弹与弹体命中；三个玩法载体的伤害由 weapon／projectile／AOEUtil 结算，四类 FX 只负责表现。"
aliases: ["charlie_boss_aoe_flame_fx", "charlie_boss_minion_hit_fx", "charlie_boss_projectile", "charlie_boss_reflect_projectile_fx", "charlie_boss_projectile_hit_fx", "charlie_boss_minion1", "charlie_boss_minion2", "藏影女王 Boss 战特效", "查理 Boss 战斗表现", "群聚恐怖"]
tags: [prefab, fx, server-client]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/charlie_boss.lua", "prefabs/charlie_boss_minions.lua", "prefabs/charlie_boss_projectiles.lua", "stategraphs/SGcharlie_boss.lua", "aoeutil.lua", "components/combat.lua", "languages/chinese_s.po"]
size_class: complex
reliability: verified
validation: source-reviewed
status: active
---

# 藏影女王 Boss 战表现特效族（charlie_boss_aoe_flame_fx 等）

## 快速判断

- 职责与适用：覆盖 `charlie_boss`（官方译名「藏影女王」，[NAMES](../../../raw/current/scripts/languages/chinese_s.po#L340814-L340822)）战中三个文件注册的七个对象：四个纯表现 FX（`charlie_boss_aoe_flame_fx`、`charlie_boss_minion_hit_fx`、`charlie_boss_reflect_projectile_fx`、`charlie_boss_projectile_hit_fx`）与三个有表现的玩法载体（`charlie_boss_minion1`／`2`、`charlie_boss_projectile`；后者官方名「群聚恐怖」）。玩法载体的伤害与可工作目标由 weapon、projectile、AOEUtil 结算，命中事件由 combat 推出，FX 只负责画面；反射 FX 的 `animover` 还是 Boss 侧生成弹体的时点。[三文件注册](../../../raw/current/scripts/prefabs/charlie_boss_minions.lua#L227-L235)、[boss 侧注册](../../../raw/current/scripts/prefabs/charlie_boss.lua#L1057-L1058)、[弹体注册](../../../raw/current/scripts/prefabs/charlie_boss_projectiles.lua#L398-L400)
- 优先使用：查「召唤 minion 的落焰／minion 命中的火花／反射弹生成时机／弹体命中表现」从下方注册与构造、关联机制导航进入；只查主体行为时看 `charlie_boss` 本体（Wiki 待建）或 `charlie_boss_runner`，不把本页当 Boss 行为页。
- 重要副作用：`charlie_boss_minion1`／`2` 与 `charlie_boss_projectile` 在主模拟挂 weapon／projectile；前者每帧用 Boss 的 tagset 做 `AOEUtil.Attack`（半径 2.4＋目标物理半径），生成时另以 `AOEUtil.Work(inst, 2.4, ...)` 处理一次可工作目标——该 helper 查询半径为 2.9，当前调用没有额外 arc／hitbox 窄相；后者 `Detonate` 时做半径 1.2 的 AOE。三只 minion 共用 Boss SG 的 `targets` 表，整轮每个目标只结算一次。`charlie_boss_aoe_flame_fx` 名字含 `aoe` 但不结算伤害。[minion AOE](../../../raw/current/scripts/prefabs/charlie_boss_minions.lua#L56-L62)、[弹体 AOE](../../../raw/current/scripts/prefabs/charlie_boss_projectiles.lua#L269-L279)
- 修改边界：本页七个注册都没有主实体父挂；只有 `charlie_boss_projectile` 在非专服创建两个未注册的内部视觉子实体（rotator 父挂弹体、looper 父挂 rotator）。Boss 本体、`charlie_boss_runner`、`charlie_boss_vines` 等玩法不写入本页。本族注册中只有弹体有官方名「群聚恐怖」，其余（含四个 FX）均无 NAMES 键，标题中的效果描述不是官方译名；实际动画帧长与演出未运行验证。[内部子实体](../../../raw/current/scripts/prefabs/charlie_boss_projectiles.lua#L49-L100)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `charlie_boss_aoe_flame_fx` | `fxfn`：Transform＋AnimState＋Network → bank／build／anim → SetPristine → 主模拟分支。 | 纯 FX：无 SoundEmitter、无 weapon／combat／AOE；bank `charlie_boss`／build `charlie_boss_basic`／anim `aoe_flame_fx`，`SetFinalOffset(-1)` 作地面层，`fx_red_particle` 灯覆盖；FX／NOCLICK。服务端 `animover` 自删、`persists=false`，客户端只接收复制。[工厂](../../../raw/current/scripts/prefabs/charlie_boss.lua#L1028-L1055)、[注册](../../../raw/current/scripts/prefabs/charlie_boss.lua#L1057-L1058) |
| `charlie_boss_minion_hit_fx` | `hitfxfn`：Transform＋AnimState＋Network → bank／build／anim → SetPristine → 主模拟分支。 | 纯 FX；assets 独立（`shadow_insanity1_basic.zip`）；bank `shadowcreature1`／anim `charlie_atk_fx`、FinalOffset 7、`shadow_dust_parts` 灯覆盖；服务端 `animover` 自删、`persists=false`。生成时按受害者位置与 `GetCombatFxSize` 设 Y 偏移、`clamp(r/1.5,0.5,1)` 缩放与随机镜像。[工厂](../../../raw/current/scripts/prefabs/charlie_boss_minions.lua#L174-L200)、[生成](../../../raw/current/scripts/prefabs/charlie_boss_minions.lua#L38-L44)、[注册](../../../raw/current/scripts/prefabs/charlie_boss_minions.lua#L229) |
| `charlie_boss_minion1` | `MakeMinion` 返回的工厂＋`minion_data["charlie_boss_minion1"]`；`InitMinion` 由 SG 调用。 | 有表现的玩法载体：四面向、anim `charlie_atk`、白／红 shadowcreature body 覆盖；FX／NOCLICK／weapon／projectile 标签；主模拟 updatelooper＋weapon（`TUNING.CHARLIE_BOSS_MINION_DAMAGE`）＋projectile；jumpout 2 帧、impact 17 帧、impact 后 3 帧结束命中；speedmult 0.7。[工厂](../../../raw/current/scripts/prefabs/charlie_boss_minions.lua#L104-L170)、[数据](../../../raw/current/scripts/prefabs/charlie_boss_minions.lua#L204-L214) |
| `charlie_boss_minion2` | 同一 `MakeMinion` 循环注册；`minion_data["charlie_boss_minion2"]`。 | 与 minion1 职责相同；六面向、bank `shadowcreature2`／build `shadow_insanity2_basic`；jumpout 3 帧、impact 13 帧；speedmult 1。[数据](../../../raw/current/scripts/prefabs/charlie_boss_minions.lua#L216-L224)、[循环注册](../../../raw/current/scripts/prefabs/charlie_boss_minions.lua#L231-L233) |
| `charlie_boss_projectile` | `fn`：Transform＋SoundEmitter＋Network（主实体无 AnimState）→ 物理／标签／netvar → 主模拟分支；非专服另建视觉子实体。 | 有表现的玩法载体／联网壳：CLASSIFIED／pseudoprojectile；质量 1、球半径 0.5；`tilt` net_smallbyte 同步；主模拟 weapon（`TUNING.CHARLIE_BOSS_PROJECTILE_DAMAGE`）＋projectile；暴露 `Launch`／`Detonate`；`OnEntitySleep` 自删、`persists=false`。官方名「群聚恐怖」。[工厂](../../../raw/current/scripts/prefabs/charlie_boss_projectiles.lua#L281-L331)、[名称](../../../raw/current/scripts/languages/chinese_s.po#L340819-L340822) |
| `charlie_boss_reflect_projectile_fx` | `reflectfxfn`：Transform＋AnimState＋Network → 八面向 → bank／build／anim → SetPristine → 主模拟分支。 | 纯 FX＋生成时点：bank `charlie_boss`／build `charlie_boss_actions`／anim `air_ripple_block`、`air_ripple_parts_red` 灯覆盖；FX／NOCLICK。服务端不用 `animover` 自删，而是 `动画长度+2帧` 定时移除（注释说明 `animover` 被 Boss 用于生成弹体）、`persists=false`。[工厂](../../../raw/current/scripts/prefabs/charlie_boss_projectiles.lua#L335-L363)、[定时移除](../../../raw/current/scripts/prefabs/charlie_boss_projectiles.lua#L358-L359) |
| `charlie_boss_projectile_hit_fx` | `hitfxfn`：Transform＋AnimState＋SoundEmitter＋Network → bank／build／anim → SetPristine → 主模拟分支。 | 纯 FX；bank `charlie_boss`／build `charlie_boss_actions`／anim `air_ripple_atk_impact`；FinalOffset 7、`fx_red_particle` 灯覆盖；打击声由生成函数 `SpawnHitFxAtXYZ` 播（`rifts8/charlie/projectile_impact`）；服务端 `animover` 自删、`persists=false`。[工厂](../../../raw/current/scripts/prefabs/charlie_boss_projectiles.lua#L367-L394)、[生成与声](../../../raw/current/scripts/prefabs/charlie_boss_projectiles.lua#L41-L47) |

范围辨析：`charlie_boss`（本体，[注册](../../../raw/current/scripts/prefabs/charlie_boss.lua#L1057-L1058)）与 `charlie_boss_runner` 是 Boss 本体玩法，不在本页展开；`CreateMissileLoop`／`CreateRotator`（[L49](../../../raw/current/scripts/prefabs/charlie_boss_projectiles.lua#L49-L69)、[L71](../../../raw/current/scripts/prefabs/charlie_boss_projectiles.lua#L71-L100)）未注册，只在 `charlie_boss_projectile` 的非专服表现中作为内部子实体说明。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 生成者：`SGcharlie_boss` `attack` | 第 33／37／40 帧先后 `SpawnMinion`（距离 3／2／2，偏移 0／±4，速度 24／20／20）；每次都在该 minion 的生成点同时生成 `charlie_boss_aoe_flame_fx`。 | 主模拟；Boss 攻击状态。 | [状态](../../../raw/current/scripts/stategraphs/SGcharlie_boss.lua#L640-L642)、[三次生成](../../../raw/current/scripts/stategraphs/SGcharlie_boss.lua#L702-L715)、[SpawnMinion](../../../raw/current/scripts/stategraphs/SGcharlie_boss.lua#L91-L109) |
| 发射与落地：minion `InitMinion`／`OnJumpOut`／`OnImpact` | `InitMinion` 记录 speed／caster／targets；`OnJumpOut` 在 jumpout 帧设 `Physics:SetMotorVel(speed×speedmult,0,0)`；`OnImpact` 停物理并在当前世界坐标生成 `charlie_boss_aoe_flame_fx`；伤害不在此结算。 | 主模拟；minion1 2／17 帧、minion2 3／13 帧。 | [InitMinion](../../../raw/current/scripts/prefabs/charlie_boss_minions.lua#L82-L102)、[OnJumpOut](../../../raw/current/scripts/prefabs/charlie_boss_minions.lua#L64-L68)、[OnImpact](../../../raw/current/scripts/prefabs/charlie_boss_minions.lua#L70-L75)、[数据](../../../raw/current/scripts/prefabs/charlie_boss_minions.lua#L204-L225) |
| 生成者：minion `OnAttackOther` | 监听 Boss 的 `onattackother`（`data.projectile == minion`）；命中时在受害者位置生成 `charlie_boss_minion_hit_fx`，按 `GetCombatFxSize` 设 Y 偏移、缩放和随机镜像；同一回调对 pinnable 目标推 knockback（非 FX 实体行为）。 | 主模拟；命中瞬间；事件由 `combat:DoAttack` 携带 target／weapon／projectile 推出。 | [回调](../../../raw/current/scripts/prefabs/charlie_boss_minions.lua#L26-L54)、[监听](../../../raw/current/scripts/prefabs/charlie_boss_minions.lua#L91-L96)、[事件](../../../raw/current/scripts/components/combat.lua#L1166)、[[reference/components/combat#伤害与事件顺序]] |
| 生成者：Boss `SpawnReflectProjectileAtXYZ` | 在反射点生成 `charlie_boss_reflect_projectile_fx` 并按 rot 朝向；该 FX 的 `animover` 被此函数监听：先移除 FX，再 `SpawnPrefab("charlie_boss_projectile")` 并 `Launch`（目标为实体或兜底点）。 | 主模拟；反射路径（`OnBlocked`／`TryGenericDeflectable`／复杂投射物）与 `taunt` 的 `air_cast` 弹幕都走此入口。 | [生成函数](../../../raw/current/scripts/prefabs/charlie_boss.lua#L205-L221)、[复杂投射物反射](../../../raw/current/scripts/prefabs/charlie_boss.lua#L301-L333)、[taunt 发射](../../../raw/current/scripts/stategraphs/SGcharlie_boss.lua#L452)、[环布偏移](../../../raw/current/scripts/stategraphs/SGcharlie_boss.lua#L472-L503) |
| 生成者：弹体 `Despawn` | 在弹体位置生成 `charlie_boss_projectile_hit_fx`（y 取 `max(0, y-0.2)`，随机镜像），补播 `rifts8/charlie/projectile_impact`，再移除弹体；由生命周期任务、施法者失效／死亡、落地或命中 `Detonate` 触发。 | 主模拟；弹体消失时。 | [SpawnHitFxAtXYZ](../../../raw/current/scripts/prefabs/charlie_boss_projectiles.lua#L41-L47)、[Despawn](../../../raw/current/scripts/prefabs/charlie_boss_projectiles.lua#L131-L134)、[生命周期](../../../raw/current/scripts/prefabs/charlie_boss_projectiles.lua#L260-L263) |
| 弹体飞行与命中 | `UpdateFlightPath` 每帧刷新目标位置；目标在阴影中累计 0.3 秒丢锁；y<0.5，或在低空时与目标水平距离小于 `0.5+目标物理半径`，即 `Detonate`；施法者失效／死亡立即 `Despawn`。 | 主模拟；飞行期间。 | [UpdateFlightPath](../../../raw/current/scripts/prefabs/charlie_boss_projectiles.lua#L142-L237)、[Detonate](../../../raw/current/scripts/prefabs/charlie_boss_projectiles.lua#L269-L279) |
| 结算：weapon／projectile／AOEUtil | minion 与弹体都在主模拟挂 weapon＋projectile；`AOEUtil.Attack` 最终以 `caster.components.combat:DoAttack(v, weapon, projectile)` 结算。minion 每帧攻击半径 2.4＋目标物理半径（`targets` 去重），弹体 `Detonate` 半径 1.2；tagset 由 Boss 构建并排除 shadow 系。 | 主模拟；minion 飞行期间与弹体引爆时；伤害数取自各自的 weapon 伤害常量。 | [minion 挂载](../../../raw/current/scripts/prefabs/charlie_boss_minions.lua#L148-L154)、[minion AOE](../../../raw/current/scripts/prefabs/charlie_boss_minions.lua#L56-L62)、[弹体挂载](../../../raw/current/scripts/prefabs/charlie_boss_projectiles.lua#L318-L322)、[AOE 结算](../../../raw/current/scripts/aoeutil.lua#L129)、[tagset](../../../raw/current/scripts/prefabs/charlie_boss.lua#L44-L53)、[[reference/components/weapon#关键接口]]、[[reference/components/projectile#关键接口]] |
| 结算：AOEUtil.Work | `InitMinion` 生成时以 `AOEUtil.Work(inst, 2.4, targets, caster)` 处理一次 CHOP／HAMMER／MINE，helper 实际查询半径为 2.9；当前调用没有额外 arc／hitbox，不再按 2.4 窄相筛选。处理过的目标记入同一 `targets` 表。 | 主模拟；仅初始化一次。 | [调用](../../../raw/current/scripts/prefabs/charlie_boss_minions.lua#L98-L100)、[可工作目标处理](../../../raw/current/scripts/aoeutil.lua#L221-L250) |
| 同步／客户端：`tilt` netvar 与内部子实体 | `charlie_boss_projectile` 用 net_smallbyte 同步倾角；非专服在 `InitializeVisualFx` 建 rotator（父挂弹体）与 looper（父挂 rotator，FollowSymbol），两者不联网、`persists=false`；客户端按 `tiltdirty` 更新动画帧。 | 非专服（含主机）／客户端表现；专服不建这两个子实体。 | [netvar](../../../raw/current/scripts/prefabs/charlie_boss_projectiles.lua#L295-L302)、[CreateRotator](../../../raw/current/scripts/prefabs/charlie_boss_projectiles.lua#L71-L100)、[InitializeVisualFx](../../../raw/current/scripts/prefabs/charlie_boss_projectiles.lua#L116-L129) |
| 清理 | 纯 FX 在服务端 `animover` 自删；reflect FX 用 `动画长度+2帧` 定时删；minion 在 `animover` 与 `resetboss` 监听时移除；弹体在生命周期到期、`resetboss`、`OnEntitySleep` 或 `Detonate`→`Despawn` 时移除；七个注册均 `persists=false`。 | 主模拟；客户端随网络移除。 | [aoe 清理](../../../raw/current/scripts/prefabs/charlie_boss.lua#L1047-L1052)、[minion 清理](../../../raw/current/scripts/prefabs/charlie_boss_minions.lua#L156-L161)、[reflect 清理](../../../raw/current/scripts/prefabs/charlie_boss_projectiles.lua#L358-L360)、[弹体清理](../../../raw/current/scripts/prefabs/charlie_boss_projectiles.lua#L326-L328)、[hit fx 清理](../../../raw/current/scripts/prefabs/charlie_boss_projectiles.lua#L390-L391) |
| 资源 | 动画 bank／build：`charlie_boss`／`charlie_boss_basic`（aoe flame）；`shadowcreature1`／`shadow_insanity1_basic`（minion hit、minion1）；`shadowcreature2`／`shadow_insanity2_basic`（minion2）；`charlie_boss`／`charlie_boss_actions`（弹体子实体、reflect、projectile hit）。音效由生成方播放。 | 声明见各文件 assets；未核验资源实际帧数。 | [boss assets](../../../raw/current/scripts/prefabs/charlie_boss.lua#L1-L8)、[minion assets](../../../raw/current/scripts/prefabs/charlie_boss_minions.lua#L1-L4)、[projectile assets](../../../raw/current/scripts/prefabs/charlie_boss_projectiles.lua#L3-L6) |
| 关联 Prefab：`charlie_boss` | 依赖列表声明本族 5 个名字；Boss 自身是生成者，本体玩法不在本页。 | 主模拟；本体页 Wiki 待建。 | [依赖](../../../raw/current/scripts/prefabs/charlie_boss.lua#L10-L18)、[注册](../../../raw/current/scripts/prefabs/charlie_boss.lua#L1057-L1058) |

## 常见陷阱与限制

- **误区：名字带 `_aoe_` 就结算伤害**：`charlie_boss_aoe_flame_fx` 无 weapon／combat／AOE 组件，只在 minion 生成点与落点 `SetPosition`；实际伤害在 minion 的 weapon＋`AOEUtil.Attack`。[工厂](../../../raw/current/scripts/prefabs/charlie_boss.lua#L1028-L1055)
- **边界：反射 FX 的 `animover` 是生成弹体的时点**：Boss 侧在 `SpawnReflectProjectileAtXYZ` 中监听它；FX 自身反而不用 `animover` 自删，而用 `动画长度+2帧`，避免与生成监听竞争。提前 `Remove` 会让弹体不生成；替换动画时需确认仍会派发 `animover`。[监听](../../../raw/current/scripts/prefabs/charlie_boss.lua#L210-L220)、[自删](../../../raw/current/scripts/prefabs/charlie_boss_projectiles.lua#L358-L359)
- **误区：minion 是纯视觉**：`charlie_boss_minion1`／`2` 有 weapon／projectile 与每帧 2.4 AOE，还能做 work；`charlie_boss_minion_hit_fx` 只是 minion 命中目标时的表现。三只 minion 共用 SG 的 `targets` 表（[SG](../../../raw/current/scripts/stategraphs/SGcharlie_boss.lua#L702-L715)），整轮每个目标只结算一次；复制时拆表会叠加伤害。
- **边界：弹体主实体没有动画**：`charlie_boss_projectile` 主实体只有 Transform／SoundEmitter／Network／Physics；可见弹体是非专服的 rotator，`fx.AnimState` 不适用于主实体，rotator／looper 不联网，不能按注册名生成或清理。[主实体](../../../raw/current/scripts/prefabs/charlie_boss_projectiles.lua#L281-L302)
- **边界：弹体命中表现也在自然消失时出现**：寿命到期、施法者失效／死亡、落地都会走 `Despawn` 生成 `charlie_boss_projectile_hit_fx`；`OnEntitySleep` 则直接 `Remove`，不生成 FX。[Despawn](../../../raw/current/scripts/prefabs/charlie_boss_projectiles.lua#L131-L134)、[OnEntitySleep](../../../raw/current/scripts/prefabs/charlie_boss_projectiles.lua#L326-L328)
- **名称边界**：除 `charlie_boss_projectile`（群聚恐怖）与宿主 Boss（藏影女王）外，本族其余名称（含 FX 与两个 minion）均为效果描述或注册名，非官方译名。

## 验证记录

- 源码审阅：2026-11-04，Steam buildid 26335394（实读 `raw/current/BUILD.txt`）。本页覆盖 7 个注册、3 个玩法载体与 4 个纯 FX、两种端与多条清理路径，需整体保留，按 complex 档。逐项核验七个注册的工厂、端、生成点、父挂与清理；递归检索 `raw/current/scripts` 的七个注册名，另对 2 个非 UTF-8 文件按字节检查无命中；`charlie_boss*.lua` 无 `event_server_data` 调用，工厂自足，无事件包缺口；`languages/chinese_s.po` 名称键已核验。未覆盖：Boss 本体与 `charlie_boss_runner` 玩法、资源动画帧长与视觉演出、游戏运行、各组件的完整实现。
- 游戏运行：未运行。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 均通过（结果见交付报告）。

相关页面：[[reference/prefabs/alterguardian_spintrail_fx|天体英雄与月脉冲特效]]（同类 Boss 战表现与生成方分工）。

返回 [[reference/prefabs/fx-creature-index|生物、Boss 与遭遇演出]]。
