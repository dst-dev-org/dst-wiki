---
title: "薇洛暗焰喷射布焰代理（flamethrower_fx）"
created: "2026-10-29"
updated: "2026-11-08"
type: prefab
summary: "以太余烬施放 lunar_fire 技能时的引导布焰代理：无视觉的 CLASSIFIED 实体携带 weapon／planardamage／shadow_aligned 加成，四条周期流把会结算伤害的地面火焰（warg_mutated_breath_fx）铺成扇面。"
aliases: ["flamethrower_fx", "暗焰喷射", "薇洛喷火", "lunar_fire", "月焰喷射", "willow lunar fire"]
tags: [prefab, fx, server-client]
size_class: simple
dst_build: "steam-buildid:26474873"
sources: ["prefabs/flamethrower_fx.lua", "prefabs/willow_ember.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 薇洛暗焰喷射布焰代理（flamethrower_fx）

## 快速判断

- 职责与适用：以太余烬（官方译名，[NAMES](../../../raw/current/scripts/languages/chinese_s.po#L351395-L351397)）的 lunar_fire 技能引导期间，挂在玩家身上的隐形代理：每 10 帧在 3/5/7/9 四档距离布下**会结算伤害的** [[reference/prefabs/warg_mutated_breath_fx|warg_mutated_breath_fx 地面火焰]]，本实体提供 weapon／planardamage 作为结算伤害源（[布焰](../../../raw/current/scripts/prefabs/flamethrower_fx.lua#L7-L43)）。
- 优先使用：查火焰怎么造成伤害／灼烬怎么生成 → 命中结算在 warg 页的 OnUpdateHitbox；查怎么开始/停止引导 → 本页 willow_ember 小节。
- 重要副作用：伤害路径是"目标 combat 查找 → owner（本代理）weapon 结算"（FlameTargetDamage），带 [[reference/components/planardamage|planardamage]] 30 与对 shadow_aligned 1.1 倍加成；停止引导后代理延迟 1 秒移除等余焰结算（[武器](../../../raw/current/scripts/prefabs/flamethrower_fx.lua#L111-L118)、[KillFX](../../../raw/current/scripts/prefabs/flamethrower_fx.lua#L68-L79)）。
- 修改边界：本实体无 AnimState，专服客户端都只见它布下的焰；`persists=false` 不存档；骑乘时不可施放。[无视觉](../../../raw/current/scripts/prefabs/flamethrower_fx.lua#L82-L89)、[骑乘限制](../../../raw/current/scripts/prefabs/willow_ember.lua#L434-L435)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `flamethrower_fx` | fn：Transform＋SoundEmitter＋Network、标签 CLASSIFIED＋pseudorangedweapon → SetPristine → 主模拟：flame_pool/ember_pool/angle、共享 `targets` 表、4 条 `DoPeriodicTask(10帧, SpawnBreathFX, 延迟 0/3/6/9 帧, dist 3/5/7/9, targets)` → weapon(20)／planardamage(30)／damagetypebonus(shadow_aligned ×1.1) → 暴露 SetFlamethrowerAttacker／KillFX、`persists=false` | 单一注册；声明依赖 warg_mutated_breath_fx／warg_mutated_ember_fx（ember 池由焰的 SpawnEmbers 消耗）。[工厂](../../../raw/current/scripts/prefabs/flamethrower_fx.lua#L81-L127)、[依赖](../../../raw/current/scripts/prefabs/flamethrower_fx.lua#L1-L5)、[注册](../../../raw/current/scripts/prefabs/flamethrower_fx.lua#L129) |

注册名无 NAMES 条目；标题"暗焰喷射"为已核验技能描述措辞，非官方译名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 生成者：willow_ember TryLunarFire | 施放时生成本代理并 `SetParent(玩家)`＋`SetFlamethrowerAttacker(玩家)`；5 秒 `EndLunarFire` → channelcaster:StopChanneling；监听玩家 `stopchannelcast` 立即 KillFX；冷却 `lunar_fire` 13 秒 | 前置：背包有 `lunarfire` 栈、未在引导中、未骑乘（CANT_SPELL_MOUNTED） | [施法函数](../../../raw/current/scripts/prefabs/willow_ember.lua#L431-L441)、[TryLunarFire](../../../raw/current/scripts/prefabs/willow_ember.lua#L327-L358)、[冷却](../../../raw/current/scripts/tuning.lua#L8132) |
| 布焰参数 | scale 1.4+rand×0.25（dist<6 再 ×1.2，dist>7 按 (dist-7)/6 放大）；fadeoption dist<6 `nofade`、≤7 `latefade`、否则 nil；落点＝母实体朝向 × dist＋随机散布 dist/20 | 四档距离共用一个 `targets` 表，同轮攻击共享多段判定 | [SpawnBreathFX](../../../raw/current/scripts/prefabs/flamethrower_fx.lua#L23-L42)、[targets 共享](../../../raw/current/scripts/prefabs/flamethrower_fx.lua#L101-L109) |
| 音效 | 首次布焰播 `fireAddFuel`＋`rifts3/mutated_varg/blast_lp`（"loop" 通道循环）；KillFX 播 `blast_pst`，6 帧后 KillSound | 声音在代理自身（非专服才可听） | [起音](../../../raw/current/scripts/prefabs/flamethrower_fx.lua#L9-L15)、[收音](../../../raw/current/scripts/prefabs/flamethrower_fx.lua#L77-L78) |
| 对象池 | flame_pool 供 SpawnBreathFX 复用焰实体；ember_pool 供焰实体的 SpawnEmbers 复用灼烬；KillFX／OnRemoveEntity 清池 | 仅主模拟 | [取池](../../../raw/current/scripts/prefabs/flamethrower_fx.lua#L17-L21)、[清池](../../../raw/current/scripts/prefabs/flamethrower_fx.lua#L49-L62) |
| 数值：tuning | 伤害 20／位面 30／shadow_aligned 加成 1.1／引导 5 秒／冷却 13 秒 | 单位为每击伤害／秒 | [数值](../../../raw/current/scripts/tuning.lua#L8107-L8115) |

## 典型接入

### 借用"代理武器＋周期布焰"的引导技能骨架

- 前提与选择：主模拟 `SpawnPrefab("flamethrower_fx")` 后 `SetParent(doer)`＋`SetFlamethrowerAttacker(结算归属)`；布焰节奏由工厂内 4 条周期任务驱动，无需额外调用。
- 生效与撤销：结束必须走 `KillFX()`（取消任务、清池、延迟 1 秒 Remove——注释明言延迟是因为余焰还引用本实体结算武器伤害）；直接 Remove 会掐断正在飞行/结算的焰的 owner 引用。
- 原版依据：willow_ember 的施法→引导→stopchannelcast→KillFX 链即官方用法。

```lua
local fx = SpawnPrefab("flamethrower_fx")
fx.entity:SetParent(doer.entity)
fx:SetFlamethrowerAttacker(doer)   -- 伤害归属
-- 引导结束：
fx:KillFX()                        -- 不要直接 fx:Remove()
```

- 调整与验证：距离档／周期写死在工厂任务表（[tasks](../../../raw/current/scripts/prefabs/flamethrower_fx.lua#L103-L109)）；伤害走 warg 页 OnUpdateHitbox 的 owner weapon 路径，改本实体 weapon 数值即改每击伤害。

## 常见陷阱与限制

- **误区：本体带火焰动画**：工厂只有 Transform＋SoundEmitter＋Network，视觉全由它生成的 warg_mutated_breath_fx 承担；想改外观要改焰工厂而非本页（[工厂分界](../../../raw/current/scripts/prefabs/flamethrower_fx.lua#L82-L89)）。
- **误区：KillFX 后立即消失**：代理延迟 1 秒才 Remove，期间仍可被余焰当作武器引用；把 owner 引用存到别处需自判有效性（[延迟注释](../../../raw/current/scripts/prefabs/flamethrower_fx.lua#L74-L75)）。
- **边界：焰的目标组沿用 warg 页规则**：本代理由玩家生成且非 PVP 判定按 attacker 展开——PVE 目标组排除 player（薇洛焰不烧玩家），PVP 开启时此组额外排除 wall（其他共同排除标签仍生效）；三组标签的完整定义见 [[reference/prefabs/warg_mutated_breath_fx#常见陷阱与限制|warg 页陷阱]]。
- **边界：骑乘与栈数**：骑乘施放直接失败返回 CANT_SPELL_MOUNTED；`lunarfire` 栈不足同样拒绝（[检查](../../../raw/current/scripts/prefabs/willow_ember.lua#L431-L437)）。

## 验证记录

- 源码审阅：2026-10-30，build 26230496；工厂全文（布焰参数、池、KillFX、组件数值）、willow_ember 的施法与引导链、tuning 数值、以太余烬译名已核验；warg_mutated_breath_fx 的命中结算沿用 [[reference/prefabs/warg_mutated_breath_fx#验证记录|warg 页已核验记录]]；channelcaster 组件内部、客户端对 CLASSIFIED 实体的可见性细节未核验。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、label-check 通过（结果见本批交付报告）；本批依据见[[updates/build-26335394/index#wiki-沿用记录]]（批次-2）。

返回 [[reference/prefabs/fx-character-index|角色技能、施法与读书]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-08 build 26474873：本页随版本迁移升版（锚点迁移 2 处），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-10）。
