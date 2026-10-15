---
title: "lunarsupernovaburning"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "月耀超新星灼烧：目标被超新星点着后的 DOT 状态，多光源汇聚、按脉冲扣血并叠加眩晕与染色。"
aliases: ["超新星灼烧", "lunarsupernovaburning", "月耀燃烧"]
tags: [component, energy]
dst_build: "steam-buildid:26230496"
sources: ["components/lunarsupernovaburning.lua", "stategraphs/SGalterguardian_phase4_lunarrift.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# lunarsupernovaburning

## 快速判断

- 职责与适用：月耀守卫者四阶段的超新星命中后由状态图动态加到目标上（[加组件](../../../raw/current/scripts/stategraphs/SGalterguardian_phase4_lunarrift.lua#L237-L240)），每个攻击中的守卫都是一个"源"。
- 优先使用：不需要主动接入；想复现类似机制可参照其"源列表 + 场地校验 + 脉冲伤害"结构。
- 重要副作用：灼烧期间反复 `MaximizeGrogginess`（[眩晕](../../../raw/current/scripts/components/lunarsupernovaburning.lua#L142-L144)），并经 [[reference/components/health]] 登记月灼来源供死亡结算。
- 修改边界：全部服务器逻辑；源失效、目标出圈或进入豁免即自移除组件。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `AddSource(source)` | 源实体 | 登记一个灼烧光源并生成对应 FX；同源幂等（[实现](../../../raw/current/scripts/components/lunarsupernovaburning.lua#L35-L49)）。 |
| `OnUpdate(dt)` | 自动周期 | 校验源（需 sg 标签 `supernovaburning`）、场地与遮挡，按源数结算伤害（[主循环](../../../raw/current/scripts/components/lunarsupernovaburning.lua#L51-L153)）。 |
| `GetFxSize()` | → `size, radius` | 按物理半径/标签分 small/med/large，骑乘取坐骑（[实现](../../../raw/current/scripts/components/lunarsupernovaburning.lua#L23-L33)）。 |
| `sources` | `{源实体: FX}` | 当前生效源表，组件自管（[字段](../../../raw/current/scripts/components/lunarsupernovaburning.lua#L5)）。 |
| 自移除条件 | 内部检查 | 无源、limbo、`notarget`、sg `flight`/`invisible`/`noattack`、无 health 或已死、不可被攻击（[守卫](../../../raw/current/scripts/components/lunarsupernovaburning.lua#L95-L109)）。 |

## 典型接入

### 把目标交给一个灼烧源

- 前提与选择：服务器侧；源实体必须处在带 `supernovaburning` 状态标签的状态里，否则 FX 在下个周期被清（[校验](../../../raw/current/scripts/components/lunarsupernovaburning.lua#L58-L61)）。
- 生效与撤销：伤害首跳立即、之后每 12 tick 一跳（[脉冲](../../../raw/current/scripts/components/lunarsupernovaburning.lua#L129-L133)）；所有源消失组件即自拆。
- 原版依据：状态图技能周期扫描并对命中者加组件与 `AddSource`（[命中](../../../raw/current/scripts/stategraphs/SGalterguardian_phase4_lunarrift.lua#L222-L241)）。

```lua
-- 服务器：源为处于 supernovaburning 状态的守卫者实体
inst:AddComponent("lunarsupernovaburning")
inst.components.lunarsupernovaburning:AddSource(lunarrift_guardian)
```

- 调整与验证：DPS 取 `TUNING.ALTERGUARDIAN_LUNAR_SUPERNOVA_LUNAR_BURN_DPS`（[调参](../../../raw/current/scripts/tuning.lua#L8717)），实际扣血经 `CalcLunarBurnTickDamage` 按源数叠加（[结算](../../../raw/current/scripts/components/lunarsupernovaburning.lua#L120-L121)）。

## 常见陷阱与限制

- 场地规则硬编码：竞技场开启时只灼烧场内源，场外模式按 `WagBossUtil.SupernovaNoArenaRangeSq` 距离判定（[场地](../../../raw/current/scripts/components/lunarsupernovaburning.lua#L64-L70)），跨场景复用需替换这些检查。
- 首跳与后续跳的 `DoDelta` 参数不同（是否计为脉冲伤害，[差异](../../../raw/current/scripts/components/lunarsupernovaburning.lua#L115-L121)），统计伤害来源时注意 `alterguardian_phase4_lunarrift` 这一 afflicter 名。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，覆盖组件全文、SGalterguardian_phase4_lunarrift 命中链与相关 tuning；未运行游戏。
- 结构检查：validate_page.py、validate_source_links.py 与 Lua 语法探针通过。
