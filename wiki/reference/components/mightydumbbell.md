---
title: "mightydumbbell"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "哑铃锻炼负载组件：按使用者 mightiness 阶层选效率档驱动涨力与 finiteuses 消耗，锻炼期挂 lifting tag，投掷命中按攻击效率涨力。"
aliases: ["哑铃锻炼", "举哑铃", "mightydumbbell"]
tags: [component, equipment]
dst_build: "steam-buildid:26230496"
size_class: simple
sources: ["components/mightydumbbell.lua", "prefabs/dumbbells.lua", "components/dumbbelllifter.lua", "componentactions.lua", "stategraphs/SGwilson.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# mightydumbbell

## 快速判断

- 职责与适用：挂 dumbbell 系列物品（prefabs/dumbbells.lua:619-621 附 SetConsumption/SetEfficiency）；使用者必须是带 `strongman` tag 且有 mightiness 组件的玩家（components/mightydumbbell.lua:8-10；strongman/mightiness 仅 Wolfgang 挂载，prefabs/wolfgang.lua:667-669）。
- 优先使用：锻炼由玩家侧 dumbbelllifter 组件驱动（components/dumbbelllifter.lua:19、:25、:34）；装备态动作注册见 componentactions.lua:2389（场景）与 :2808（已装备）。
- 重要副作用：StartWorkout 给哑铃 AddTag `"lifting"`（:63）；CheckEfficiency 顺带 `mightiness:SetRateScale`（:36）；DoWorkout 消耗 finiteuses，耗尽自动 StopWorkout（:97-104）。
- 修改边界：仅服务器调用（状态图/组件内）；效率映射表只含 LOW/MED/HIGH 三键（:17-22）。

## 关键接口

| 接口/字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `StartWorkout`/`StopWorkout` | `(doer) -> nil` | 置 strongman、挂/卸 lifting tag；Stop 恢复 `RATE_SCALE.NEUTRAL`。:62-72 |
| `CheckEfficiency` | `() -> number` | 按 strongman 的 mightiness.current 与 `TUNING.WIMPY_THRESHOLD`/`MIGHTY_THRESHOLD` 比较，选 efficiency_wimpy/normal/mighty 并同步 rate scale；未开始锻炼返回 0。:24-42 |
| `DoWorkout` | `(doer) -> boolean` | `mightiness:DoDelta(效率值)` + `finiteuses:Use(consumption)`；耐久归零返回 false。:92-108 |
| `DoAttackWorkout` | `(doer) -> nil` | 投掷命中时 `CheckAttackEfficiency * TUNING.DUMBBELL_EFFICIENCY_ATTCK_SCALE` 涨力（prefabs/dumbbells.lua:103、:281）。:84-90 |
| `SetConsumption`/`SetEfficiency` | 注入接口 | 由 dumbbells.lua:620-621 按 gem 档位注入。:74-82 |

## 典型接入

### 原版参照：举重全链路

- 前提与选择：SGwilson.lua:2822 检查装备物带组件才进入 use_dumbbell 流程；动作层 LIFT_DUMBBELL/STOP_LIFT_DUMBBELL 依赖 lifting tag 区分（componentactions.lua:2389-2396、:2808-2815）。
- 生效与撤销：每次 Lift→DoWorkout 涨力并扣耐久，StopLifting 由 dumbbelllifter 统一卸 tag（dumbbelllifter.lua:26-33）。
- 无 mod 接入案例，不提供代码。

## 常见陷阱与限制

- `StopWorkout` 不判 `self.strongman` 空即访问其 mightiness（:70-71），未开始锻炼时调用会报错；DoWorkout 耗尽路径以无参 `StopWorkout()` 调用（:101），doer 形参未用。
- `IsWorkingOut` 内含调试 print（:13），调用会刷服务器日志。
- 自定义效率值必须命中 `efficiency_rate_scale` 的三键之一，否则 SetRateScale 收到 nil（:36）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，通读组件全部 110 行，核验 dumbbelllifter 三调用点、dumbbells 注入与投掷调用、componentactions 两处动作注册、SGwilson 装备检查与 wolfgang.lua:667-669 的 strongman/mightiness 挂载。
- 游戏运行：未运行。

## 相关链接

- 玩家侧驱动：[[reference/components/dumbbelllifter]]；耐久消耗：[[reference/components/finiteuses]]。
