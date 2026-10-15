---
title: "附身座狼喷焰与灼烬（warg_mutated_breath_fx）"
created: "2026-10-29"
updated: "2026-11-01"
type: prefab
summary: "会结算伤害的地面火焰：主模拟 combat／planardamage 逐帧 AOE、超冷降温，灼烬可结冰打滑；同文件另有 warg_mutated_ember_fx，薇洛暗焰喷射与突变秃鹫火墙复用同一工厂。"
aliases: ["warg_mutated_breath_fx", "warg_mutated_ember_fx", "附身座狼喷焰", "座狼喷火特效", "突变座狼火墙", "冰烬", "warg flame fx"]
tags: [prefab, fx, server-client]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/warg_mutated_fx.lua", "stategraphs/SGwarg.lua", "stategraphs/SGbuzzard.lua", "prefabs/flamethrower_fx.lua", "prefabs/willow_ember.lua", "prefabs/warg.lua", "prefabs/buzzard.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 附身座狼喷焰与灼烬（warg_mutated_breath_fx）

## 快速判断

- 职责与适用：附身座狼（官方译名，[NAMES](../../../raw/current/scripts/languages/chinese_s.po#L345459-L345462)）喷焰吐息落地的**会攻击的地面火焰**：联网 FX 壳＋主模拟 combat／planardamage／updatelooper，由生成方指定的 attacker 经 `combat:DoAttack` 结算伤害并施加超冷；不是纯视觉。同文件注册 `warg_mutated_ember_fx`（地面灼烬，可结冰打滑）。[注册](../../../raw/current/scripts/prefabs/warg_mutated_fx.lua#L595-L596)、[伤害](../../../raw/current/scripts/prefabs/warg_mutated_fx.lua#L46-L113)
- 优先使用：查"地面持续火焰／火墙／冰面打滑"选本页；三个原版生成者（座狼、秃鹫、薇洛暗焰喷射）见关联机制导航。
- 重要副作用：判伤条件为 hit_tick+10<tick（高焰 +15），逐帧更新时相邻命中至少隔 11／16 tick；温度高于 -2 时才降到 `MUTATED_WARG_COLDFIRE_TEMPERATURE(-2)`；秃鹫路径灼烬带 slipperyfeettarget 打滑。[超冷与命中](../../../raw/current/scripts/prefabs/warg_mutated_fx.lua#L88-L99)、[打滑](../../../raw/current/scripts/prefabs/warg_mutated_fx.lua#L524-L546)
- 修改边界：伤害结算在主模拟；亮度渐变（brightness netvar）双端消费、灰烬 fizzle 小特效仅非专服；实体 `persists=false` 且成对池化，存档不保存。[端分界](../../../raw/current/scripts/prefabs/warg_mutated_fx.lua#L304-L317)、[fizzle 专服跳过](../../../raw/current/scripts/prefabs/warg_mutated_fx.lua#L425-L427)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `warg_mutated_breath_fx` | fn：Transform＋AnimState＋SoundEmitter＋Network → bank/build `warg_mutated_breath_fx` 播 `flame1_pre`、bloom → brightness net_tinybyte（0-7）→ FX／NOCLICK、updatelooper → SetPristine → 客户端监听 brightnessdirty 早退 → 主模拟 combat（默认 20）／planardamage（30）、animqueueover 回池、暴露 SetFXOwner／RestartFX／ConfigureDamage／ExtendFx → 追加 `flame1_loop`（显式循环） | 动画与寿命按 RestartFX 参数重配；`flame{n}_pre→_loop(true)→KillFX 播 _pst`。[工厂](../../../raw/current/scripts/prefabs/warg_mutated_fx.lua#L287-L337)、[RestartFX](../../../raw/current/scripts/prefabs/warg_mutated_fx.lua#L224-L268) |
| `warg_mutated_ember_fx` | emberfn：Transform＋AnimState＋Network → 同 bank/build、播 `ember1_ground`（显式循环）、OnGround 背景层 SortOrder 3、隐藏 `iced` 层 → fade／fizzle net_smallbyte → updatelooper → SetPristine → 客户端监听 fadedirty 早退 → 主模拟 `persists=false`、暴露 SetFXOwner／RestartFX／KillFX／SetIced | 无 combat；`SetIced` 显示 iced 层并挂 slipperyfeettarget（仅 iced 路径）。[工厂](../../../raw/current/scripts/prefabs/warg_mutated_fx.lua#L548-L591)、[SetIced](../../../raw/current/scripts/prefabs/warg_mutated_fx.lua#L538-L546) |

两个注册名均无 NAMES 条目；标题中文为已核验效果描述，非官方译名。

## 关联机制导航

### 喷焰伤害与宿主（warg_mutated_breath_fx）

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 伤害结算：OnUpdateHitbox | 逐帧以自身为圆心以 `0.9×scale+3` 粗查 `_combat`，实际命中距离严格小于 `0.9×scale+目标物理半径`（padding 3 不参与判伤）；combat 取 `attacker` 的，`owner` 带 weapon 组件时（flamethrower_fx）把 owner 当武器传入 DoAttack；`ConfigureDamage` 后临时把 FX 伤害换进 attacker 的 combat 再还原 | 每目标按 `hit_tick+10<tick`（tallflame +15）限频；`targets` 表由同一轮攻击共享，跨焰防重复 | [结算](../../../raw/current/scripts/prefabs/warg_mutated_fx.lua#L46-L113)、[限频帧](../../../raw/current/scripts/prefabs/warg_mutated_fx.lua#L39-L44)、[ConfigureDamage](../../../raw/current/scripts/prefabs/warg_mutated_fx.lua#L277-L281) |
| 目标筛选：RegisterFindTags | notplayer 路径排除 lunar_aligned；PVP 排除 wall；PVE 排除 player——即座狼焰可烧玩家、薇洛焰 PVE 不烧玩家 | 按 `attacker` 是否玩家与 PVP 设置选三组之一，注册后缓存 | [三组标签](../../../raw/current/scripts/prefabs/warg_mutated_fx.lua#L13-L37) |
| 生成者：SGwarg flamethrower_pre/loop/pst | 附身座狼（`canflamethrower`）扇形 -45°~45° 多距离布焰（`SpawnBreathFX`，dist 4-9，nofade/latefade 按距离）；近距空档由 SG 自身 `DoFlamethrowerAOE` 补刀；灼烬由 `SpawnCloseEmberFX` 直接生成 ember（nofade） | 座狼用自身 combat（默认伤害 SG 已设 20）＋超冷；冷却 8s+随机 2s | [布焰](../../../raw/current/scripts/stategraphs/SGwarg.lua#L61-L88)、[近距 AOE](../../../raw/current/scripts/stategraphs/SGwarg.lua#L98-L136)、[攻击态](../../../raw/current/scripts/stategraphs/SGwarg.lua#L851-L941)、[灼烬](../../../raw/current/scripts/stategraphs/SGwarg.lua#L38-L59)、[门槛与冷却](../../../raw/current/scripts/stategraphs/SGwarg.lua#L10-L18) |
| 生成者：SGbuzzard flamethrower_pre/loop | 突变秃鹫低身喷出火墙：RestartFX 传 `tallflame=true`（`_tall` 后缀动画＋`fire_ground_LP` 声）、`icedember=true`（灼烬结冰）、缩放 0.85-1；落点附近已有高焰则 `ExtendFx` 续时不再堆新焰 | 伤害经 ConfigureDamage 换成 15／位面 10；火焰寿命 2.5-3s | [秃鹫布焰](../../../raw/current/scripts/stategraphs/SGbuzzard.lua#L49-L80)、[循环态](../../../raw/current/scripts/stategraphs/SGbuzzard.lua#L640-L671)、[续时](../../../raw/current/scripts/prefabs/warg_mutated_fx.lua#L270-L275) |
| 消费者：月焰消退烟团 | fx.lua 的 12 个 `lunarflame_puff_*` 注册复用本 bank／build 作进食灭火烟团，纯表现无伤害 | 水晶冠秃鹫进食燃烧尸体时按尸体体型生成；协议是通用表代理，非本页实体工厂 | [[reference/prefabs/lunarflame_puff_small|月焰消退烟团]]、[数据](../../../raw/current/scripts/fx.lua#L4136-L4157) |
| 生成者：willow_ember TryLunarFire → flamethrower_fx | 薇洛"暗焰喷射"引导技能：CLASSIFIED 代理实体挂 weapon（20）／planardamage（30）／shadow_aligned 加成，挂到玩家身上；4 条 DoPeriodicTask（10 帧）流在 dist 3/5/7/9 布焰，`SetFXOwner(fx实体, 玩家)` → OnUpdateHitbox 用玩家 combat＋owner 武器结算 | 停止引导时 KillFX；代理延迟 1 秒移除等余焰结算完 | [TryLunarFire](../../../raw/current/scripts/prefabs/willow_ember.lua#L327-L358)、[代理武器](../../../raw/current/scripts/prefabs/flamethrower_fx.lua#L97-L118)、[代理布焰](../../../raw/current/scripts/prefabs/flamethrower_fx.lua#L7-L43)、[延迟移除](../../../raw/current/scripts/prefabs/flamethrower_fx.lua#L68-L79) |

### 灼烬与结冰打滑（warg_mutated_ember_fx）

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 灼烬生命周期 | 焰生成 2 帧后 SpawnEmbers（scale×1.1，同 fadeoption）；淡出曲线由 fade netvar 驱动（0-63），第 8 帧非专服生成 fizzle 烟花 | iced 焰的烬 KillFX 延迟 11-12s 才开始淡出；fade≤18 且 1 距离内打滑 | [SpawnEmbers](../../../raw/current/scripts/prefabs/warg_mutated_fx.lua#L185-L206)、[淡出](../../../raw/current/scripts/prefabs/warg_mutated_fx.lua#L415-L442)、[iced 延迟](../../../raw/current/scripts/prefabs/warg_mutated_fx.lua#L516-L522)、[[reference/components/slipperyfeettarget#关键接口]] |

### 共享对象池与参数

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 对象池 | animqueueover 时回 `owner.flame_pool`／淡出后回 `ember_pool`，无池则 Remove；宿主在变更体初始化池 | 座狼 [warg.lua](../../../raw/current/scripts/prefabs/warg.lua#L1003-L1005)、秃鹫 [buzzard.lua](../../../raw/current/scripts/prefabs/buzzard.lua#L490-L492)、代理 [flamethrower_fx.lua](../../../raw/current/scripts/prefabs/flamethrower_fx.lua#L97-L98) | 仅主模拟；[回池](../../../raw/current/scripts/prefabs/warg_mutated_fx.lua#L147-L159) |
| 数值：tuning | 焰伤害 20／位面 30（座狼默认）、秃鹫 15／10、超冷 -2、高焰寿命 2.5+rand×0.5、iced 烬 11+rand×1、打滑率 20、薇洛武器 20／位面 30 | [座狼](../../../raw/current/scripts/tuning.lua#L7944-L7947)、[秃鹫](../../../raw/current/scripts/tuning.lua#L9062-L9063)、[寿命与烬](../../../raw/current/scripts/tuning.lua#L9077-L9082)、[薇洛](../../../raw/current/scripts/tuning.lua#L8113-L8114) | 单位为每击伤害／温度值／秒 |

## 典型接入

### 复用"带伤害的地面火焰"

- 前提与选择：主模拟生成后按序调用 `SetFXOwner(owner, attacker)` → 需要自定义伤害时 `ConfigureDamage(伤害, 位面)` → `RestartFX(scale, fadeoption, targets, tallflame, icedember)`；owner 是池宿主，attacker 是结算伤害的 combat 持有者。调用 ConfigureDamage 后，实际选用的 combat.inst 还必须有 planardamage，源码直接读取其 GetBaseDamage，没有判空保护。
- 生效与撤销：RestartFX 重播 pre→loop、重排寿命任务；fadeoption `"nofade"/"latefade"/nil` 控制亮度起点；`targets` 传共享表可让多焰共享多段判定。回收靠 animqueueover 回池，宿主 OnRemoveEntity 清池。
- 原版依据：三个生成者的参数组合即三种官方形态——座狼（普通焰＋SG 补刀）、秃鹫（高焰＋冰烬＋续时合并）、薇洛（代理武器结算）。

```lua
-- 主模拟；myhost 为有效池宿主，myattacker 为有效伤害归属。
local combat = myattacker ~= nil and myattacker:IsValid()
    and myattacker.components.combat or nil
assert(combat ~= nil and combat.inst.components.planardamage ~= nil,
    "ConfigureDamage requires attacker combat and planardamage")
local fx = SpawnPrefab("warg_mutated_breath_fx")
fx:SetFXOwner(myhost, myattacker)              -- 池宿主与伤害归属
fx:ConfigureDamage(15, 10)                     -- 可选：改写本次伤害
fx:RestartFX(1.0, "nofade", {}, false, false)  -- 缩放/淡出/共享targets/高焰/冰烬
```

- 调整与验证：伤害路径在 `OnUpdateHitbox`（[工厂](../../../raw/current/scripts/prefabs/warg_mutated_fx.lua#L46-L113)）；attacker 无效或缺 combat 时选择 FX 自身 combat；但 owner≠attacker 且 owner 无效会先直接 return，不保证产生回退伤害。ConfigureDamage 的外部 attacker 路径须满足上述 planardamage 前提。

## 常见陷阱与限制

- **误区：名字带 fx 就是纯视觉**：主模拟挂 combat／planardamage 并逐帧 DoAttack，是真实伤害源；击杀归属 attacker。[结算](../../../raw/current/scripts/prefabs/warg_mutated_fx.lua#L46-L113)
- **误区：伤害读 FX 自身组件就完事**：座狼路径结算用 attacker（座狼）的 combat，FX 组件值只是 ConfigureDamage 的源与 attacker 失效时的回退；薇洛路径伤害来自 owner 代理的 weapon。[武器路径](../../../raw/current/scripts/prefabs/warg_mutated_fx.lua#L49-L70)
- **误区：多焰叠加会多次结算**：同一轮攻击共享 `targets` 表，每目标每 tick 一次超冷、仅在 hit_tick+10／15<tick 时再次伤害；独立生成而不共享表才有叠加。[共享表](../../../raw/current/scripts/prefabs/warg_mutated_fx.lua#L83-L99)
- **边界：PVP／PVE 目标集不同**：PVE 组排除 player 与 wall（薇洛焰不烧玩家），PVP 在共同排除标签之外排 wall；notplayer（怪物）焰排除 lunar_aligned 而可烧玩家。[三组标签](../../../raw/current/scripts/prefabs/warg_mutated_fx.lua#L17-L36)
- **边界：秃鹫路径才打滑**：slipperyfeettarget 只在 `SetIced` 后挂载；座狼／薇洛的烬是纯视觉＋淡出，无玩法组件。[SetIced](../../../raw/current/scripts/prefabs/warg_mutated_fx.lua#L538-L546)

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496；597 行工厂全文（焰与烬两工厂、命中结算、淡出、池化、iced 路径）、SGwarg／SGbuzzard 攻击态、willow_ember TryLunarFire 与 flamethrower_fx 代理全程、池初始化、tuning 数值、译名检索已核验；动画资源内容、`flame*_loop` 之外动画的实际帧数、晚加入表现、客户端 targets 表复制的显示差异未核验。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、label-check 通过（结果见本批交付报告）。

返回 [[reference/prefabs/fx-creature-index|特效 Prefab 索引·生物与Boss]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-2）。
