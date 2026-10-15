---
title: "lunarfissureburning"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "月裂灼烧组件：给站在激活月裂上的实体动态挂载，持续扣血（骑乘时坐骑同扣）、周期性眩晕并显示灼烧特效，离开裂隙 1 秒后自毁。"
aliases: ["月裂灼烧", "裂隙燃烧", "lunarfissureburning"]
tags: [component]
dst_build: "steam-buildid:26230496"
size_class: simple
sources: ["components/lunarfissureburning.lua", "prefabs/alterguardian_lunar_fissures.lua", "prefabs/wagboss_util.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# lunarfissureburning

## 快速判断

- 职责与适用：动态挂载组件——裂隙实体每 0.5s 扫描 AOE 内实体后 `AddComponent`（prefabs/alterguardian_lunar_fissures.lua:53-58、:65-69）；构造即 StartUpdatingComponent（components/lunarfissureburning.lua:8）。
- 优先使用：无需手动调用；OnUpdate 按 `WagBossUtil.HasFissure`（tile 注册表，prefabs/wagboss_util.lua:7、:81）与豁免条件（InLimbo、notarget/flying、sg flight/invisible/noattack、死亡、不可被攻击）判定灼烧（:60-67）。
- 重要副作用：DPS=`TUNING.ALTERGUARDIAN_LUNAR_FISSURE_LUNAR_BURN_DPS`（25，tuning.lua:8715），`health:DoDelta(-dmg, not pulse, "alterguardian_phase4_lunarrift")`，坐骑存活时同扣（:72-78、:88-94）；`RegisterLunarBurnSource`/注销（:80、:103）；grogginess:MaximizeGrogginess（:96-98）。
- 修改边界：离开灼烧区 `cleartime` 累计 >1s 后组件自我 RemoveComponent（:106-111）；OnRemoveFromEntity 注销灼烧源并移除 fx（:12-19）。

## 关键接口

| 接口/字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `OnUpdate` | `(dt) -> nil` | 首个命中 tick 与之后每 12 tick（`lastlunarburnpulsetick`）为 pulse（overtime=false），其余 overtime；cleartime 三态管理。:58-112 |
| `SetFxEnabled` | `(enable) -> nil` | 生成/复用 `alterguardian_lunar_fissure_burn_fx` 子实体，按物理半径（≥1.5/≥0.9）与 epic/largecreature tag 选 large/med/small；缺 colouradder 时自动补挂并 PushColour。:21-56 |
| `cleartime` | `number\|nil` | nil=正在灼烧；离开后累计，>1s 自毁。:5、:100-111 |

## 典型接入

### 原版参照：裂隙 AOE 动态挂载

- 前提与选择：`TheSim:RegisterFindTags` 预注册 `{ "_health" }` 且排除 FX/飞行/无敌等（alterguardian_lunar_fissures.lua:45-51），再以 `CircleTouchesSquare` 判定实体与裂隙格接触（:30、:57-58）。
- 生效与撤销：组件生死完全由位置驱动，mod 无法"保留"一个离线灼烧实例。
- 无 mod 接入案例，不提供代码。

## 常见陷阱与限制

- 伤害走专用 lunar burn 通道（RegisterLunarBurnSource），常规火焰免疫/护甲减免不适用，需远离裂隙或利用豁免 tag（flying/notarget 等）。
- 同帧内 mount 与本体各结算一次 `CalcLunarBurnTickDamage`（prefabs/wagboss_util.lua:114），双人骑乘生物承伤翻倍是设计行为。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，通读组件全部 114 行，核验唯一挂载方（alterguardian_lunar_fissures.lua:53-69）、WagBossUtil 辅助（:7/:81/:114/:88）与 DPS 常量（tuning.lua:8715）。
- 游戏运行：未运行。

## 相关链接

- 同体系掩体标记：[[reference/components/lunarsupernovablocker]]。
