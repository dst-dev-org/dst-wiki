---
title: "晶体独眼巨鹿冰圈特效组（deerclops_icelance_ping_fx 等同文件 4 注册）"
created: "2026-10-29"
updated: "2026-11-01"
type: prefab
summary: "晶体独眼巨鹿四件套：冰枪落点标记环、带灭火／加冷／降温／昏沉的冰圈、足迹冰环与燃烧附火；冰圈是玩法实体，ping 与 spikefire 为表现层。"
aliases: ["deerclops_icelance_ping_fx", "deerclops_impact_circle_fx", "deerclops_aura_circle_fx", "deerclops_spikefire_fx", "晶体独眼巨鹿冰圈", "冰枪落点标记", "ice lance fx", "mutated deerclops fx"]
tags: [prefab, fx, server-client]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/deerclops_mutated_fx.lua", "stategraphs/SGdeerclops.lua", "prefabs/deerclops.lua", "components/freezable.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 晶体独眼巨鹿冰圈特效组（deerclops_icelance_ping_fx 等同文件 4 注册）

## 快速判断

- 职责与适用：晶体独眼巨鹿（官方译名，[NAMES](../../../raw/current/scripts/languages/chinese_s.po#L345449-L345452)）的四类技能特效：`deerclops_icelance_ping_fx`（冰枪落点标记环，纯视觉）、`deerclops_impact_circle_fx`（冰枪落点冰圈，**带灭火／加冷／降温／昏沉**）、`deerclops_aura_circle_fx`（战斗中跟随足迹的冰环，玩法作用同冰圈）、`deerclops_spikefire_fx`（被点燃时附在冰刺上的火焰，纯视觉）。冰圈伤害外的四类作用在 `OnUpdateIceCircle`。[注册](../../../raw/current/scripts/prefabs/deerclops_mutated_fx.lua#L400-L403)、[冰圈作用](../../../raw/current/scripts/prefabs/deerclops_mutated_fx.lua#L220-L248)
- 优先使用：查"冰枪预警／地面冰环／灭火加冷光环／燃烧附火"选本页；伤害与冻结由宿主结算，见下文生成者条目。
- 重要副作用：冰圈每帧对范围内 `freezable`／`fire`／`smolder`／`inventoryitemtemperature` 实体：Extinguish（无 fueled 的 burnable）、AddColdness 0.1（freezelimit 0.7，不主动冻结）、温度高于 -2 时降到 `DEER_ICE_TEMPERATURE(-2)`、AddGrogginess 0.1。[作用明细](../../../raw/current/scripts/prefabs/deerclops_mutated_fx.lua#L225-L245)
- 修改边界：ping／impact／aura 为联网实体，主模拟跑冰圈作用；ping 的标记盘与 impact 的爆炸子实体仅非专服生成；spikefire 联网但作用全在宿主点燃流程。[端分界](../../../raw/current/scripts/prefabs/deerclops_mutated_fx.lua#L127-L133)、[爆炸子实体](../../../raw/current/scripts/prefabs/deerclops_mutated_fx.lua#L274-L277)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `deerclops_icelance_ping_fx` | pingfn：Transform＋AnimState＋Network → bank `deerclops`／build `deerclops_mutated` 播 `target_fx_pre`、OnGround 背景层 → fade net_tinybyte → 非专服生成标记盘子实体（`target_fx_ring`，随 fade 渐显渐隐）→ SetPristine → 客户端监听 fadedirty → 主模拟 animqueueover 推进、暴露 KillFX | 无组件；`pre→target_fx(循环)→pst` 三段由 fade 0=入 1=停 2=出 驱动。[工厂](../../../raw/current/scripts/prefabs/deerclops_mutated_fx.lua#L107-L147)、[fade 驱动](../../../raw/current/scripts/prefabs/deerclops_mutated_fx.lua#L74-L105) |
| `deerclops_impact_circle_fx` | impactfn：bank/build `deer_ice_circle` 播 `impact`、OnGround、scale 2.2 → 非专服挂爆炸子实体（`ice_impact`＋`break_iceblock` 音）→ 主模拟 updatelooper＋OnUpdateIceCircle、`freezelimit=0.7`、2 秒后 KillFX 播 `pst` | 落点固定单次释放；冰圈动画持续期约 2 秒。[工厂](../../../raw/current/scripts/prefabs/deerclops_mutated_fx.lua#L256-L294)、[KillFX](../../../raw/current/scripts/prefabs/deerclops_mutated_fx.lua#L250-L254) |
| `deerclops_aura_circle_fx` | aurafn：与 impact 同 bank/build／层／缩放 → 主模拟随机旋转、OnUpdateIceCircle、`freezelimit=0.7`、暴露 GrowFX／KillFX | `GrowFX` 播 `pre`（SetFrame 20）＋`ice_circle_LP` 循环声；`KillFX()` 侵蚀淡出 2 秒，`KillFX(true)` 播 `pst` 快速退场。[工厂](../../../raw/current/scripts/prefabs/deerclops_mutated_fx.lua#L321-L358)、[Grow/Kill](../../../raw/current/scripts/prefabs/deerclops_mutated_fx.lua#L303-L319) |
| `deerclops_spikefire_fx` | spikefirefn：bank/build `fire_large_character` 播 `pre_med`（SetFrame 30）、bloom、Follower、`SetRayTestOnBB(true)`、仅 FX 标签（无 NOCLICK）→ 主模拟追加 `loop_med`、暴露 KillFX | `KillFX` 播 `post_med_fast` 后自删；通过 Follower 跟随宿主符号。[工厂](../../../raw/current/scripts/prefabs/deerclops_mutated_fx.lua#L367-L396) |

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 生成者：SGdeerclops "icelance" | 第 30 帧按目标方向预测落点（距离 clamp 进 `ICELANCE_RANGE 6-14`）生成 ping；第 60 帧 KillFX ping、落点生成 impact、设伤害 200 并三次 `DoIceLanceAOE` | 进入条件：`hasicelance`（突变体专属）且冰眼未烧毁、目标在最小射程外或自身燃烧 | [ping](../../../raw/current/scripts/stategraphs/SGdeerclops.lua#L1130-L1134)、[impact 与 AOE](../../../raw/current/scripts/stategraphs/SGdeerclops.lua#L1144-L1163)、[状态](../../../raw/current/scripts/stategraphs/SGdeerclops.lua#L1054-L1201)、[门槛](../../../raw/current/scripts/stategraphs/SGdeerclops.lua#L293-L301) |
| 宿主伤害：DoIceLanceAOE | 与 FX 无读写：落点半径内 `combat:DoAttack`（每目标一次）；已冻结目标推 `knockback`；命中经宿主 OnHitOther 额外 AddColdness(`freezepower`，icelance 期间 99)——冻结来自这里而非冰圈 | [AOE](../../../raw/current/scripts/stategraphs/SGdeerclops.lua#L228-L249)、[OnHitOther](../../../raw/current/scripts/prefabs/deerclops.lua#L207-L215)、[freezepower](../../../raw/current/scripts/prefabs/deerclops.lua#L668-L673) | 数值 200／射程 [tuning](../../../raw/current/scripts/tuning.lua#L7926-L7927) |
| 生成者：SGdeerclops "icegrow" 与脚步 | `icegrow` 第 9 帧在脚下生成 aura 并 GrowFX；此后 `DeerclopsFootstep` 每步 KillFX 旧环再生成新环（移动时前方 1.5 距离）；`death`／`stagger_pre` 等以 KillFX(true) 快速清除 | `hasiceaura` 且有战斗目标才生成；[脚步](../../../raw/current/scripts/stategraphs/SGdeerclops.lua#L348-L364)、[生成](../../../raw/current/scripts/stategraphs/SGdeerclops.lua#L1243-L1254)、[死亡清除](../../../raw/current/scripts/stategraphs/SGdeerclops.lua#L833-L835)、[踉跄清除](../../../raw/current/scripts/stategraphs/SGdeerclops.lua#L1556-L1558) | impact 半径为 5.5；aura 经 GrowFX 从 0.25 开始，按源码 `r*0.98+5.5+0.02` 一次更新即到 5.765，随后不再增长 |
| 冰圈玩法作用：OnUpdateIceCircle | 灭火（burnable 且无 fueled）、加冷 0.1（coldness 低于 resistance×0.7 才加，`nofreeze=true` 不直接触发 Freeze，见 [[reference/components/freezable#关键流程]]）、高于 -2 时降温到 -2；未昏睡且 grog<0.1 时 AddGrogginess(0.1) | impact 与 aura 共用同一函数；`NOTAGS` 排除 ghost/limbo/deerclops/flight/invisible 与各类燃料标签 | [作用](../../../raw/current/scripts/prefabs/deerclops_mutated_fx.lua#L220-L248)、[nofreeze 语义](../../../raw/current/scripts/components/freezable.lua#L137-L158)、[数值](../../../raw/current/scripts/tuning.lua#L698-L699) |
| 生成者：deerclops.lua Mutated_OnIgnite | 巨鹿被点燃（非全冰封）时生成 spikefire，Follower 跟随 `swap_fire_0/1/2` 符号；`Mutated_OnExtinguish` KillFX | 火焰只是燃烧表现；燃烧逻辑在宿主 burnable | [点燃](../../../raw/current/scripts/prefabs/deerclops.lua#L307-L315)、[熄灭](../../../raw/current/scripts/prefabs/deerclops.lua#L318-L321)、[依赖](../../../raw/current/scripts/prefabs/deerclops.lua#L37-L41) |
| 资源 | ping/impact 用 `deerclops_mutated(_actions)` 与 `deer_ice_circle.zip`，spikefire 用 `fire_large_character.zip`；仅 ping 的 `target_fx` 循环显式传循环参数（[L89](../../../raw/current/scripts/prefabs/deerclops_mutated_fx.lua#L89)），其余 `target_fx_pre`／`impact`／`pre`／`loop_med` 等未显式传，实际是否重复未核验 | [assets](../../../raw/current/scripts/prefabs/deerclops_mutated_fx.lua#L3-L24) | 内容未解包 |

## 典型接入

### 复用"灭火加冷光环"

- 前提与选择：主模拟生成 impact/aura 后由 updatelooper 驱动，无需额外调用；想自定义强度改 `freezelimit`（影响加冷上限判断）与常量 `ICE_CIRCLE_RADIUS`。
- 生效与撤销：aura 需先调 `GrowFX()`（icegrow 状态就是这么做的）；快速退场用 `KillFX(true)`，自然退场用 `KillFX()`（侵蚀 2 秒，期间不再生效——updatelooper 回调已移除）。
- 原版依据：宿主每步换新环而旧环侵蚀淡出，避免视觉与判定同时断裂（[脚步](../../../raw/current/scripts/stategraphs/SGdeerclops.lua#L353-L354)）。

## 常见陷阱与限制

- **误区：冰圈把目标冻住**：`freezelimit=0.7` 时 `AddColdness(…, true)` 只在加冷前判断 coldness<resistance×0.7，每次加 0.1 可越过该阈值；nofreeze=true 不触发 Freeze；冻结来自冰枪命中的 OnHitOther（freezepower 99）。[nofreeze](../../../raw/current/scripts/components/freezable.lua#L152-L154)、[OnHitOther](../../../raw/current/scripts/prefabs/deerclops.lua#L210)
- **误区：ping 是落点伤害**：ping 只有标记环动画与 fade；伤害与击退在第 60 帧由宿主 AOE 结算，落点由 SG 预测而非 ping 决定。[结算](../../../raw/current/scripts/stategraphs/SGdeerclops.lua#L1144-L1163)
- **误区：spikefire 是地面火**：它是跟随宿主符号的附身火焰（Follower），不独立占地、无灭火／加冷作用；那些作用在 impact／aura 上。[附火](../../../raw/current/scripts/prefabs/deerclops_mutated_fx.lua#L367-L396)
- **边界：标记盘／爆炸子实体专服不存在**：两者都在 `not TheNet:IsDedicated()` 分支创建；玩法作用（OnUpdateIceCircle）不受影响。[端分界](../../../raw/current/scripts/prefabs/deerclops_mutated_fx.lua#L127-L133)、[爆炸](../../../raw/current/scripts/prefabs/deerclops_mutated_fx.lua#L274-L277)

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496；404 行工厂全文与 4 注册、SGdeerclops icelance／icegrow／脚步／清除全程、宿主 OnHitOther 冻结链、freezable nofreeze 语义、tuning 数值、译名检索已核验；`pre`／`impact` 动画时长、aura 在 laserbeam 等其余状态的消费、晚加入表现未核验。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、label-check 通过（结果见本批交付报告）。

返回 [[reference/prefabs/fx-creature-index|生物、Boss 与遭遇演出]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-2）。
