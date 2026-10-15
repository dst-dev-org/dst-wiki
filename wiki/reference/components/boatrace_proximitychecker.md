---
title: "boatrace_proximitychecker"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "船赛检查点接近检测：比赛期间周期扫描范围内的船赛信标，驻留 1.5 秒判命中并回调；起点与检查点共用。"
aliases: ["boatrace_proximitychecker", "船赛接近检测", "船赛检查点"]
tags: [component, callback]
size_class: standard
dst_build: "steam-buildid:26230496"
sources: ["components/boatrace_proximitychecker.lua", "prefabs/boatrace_checkpoint.lua", "prefabs/boatrace_start.lua", "components/boatrace_proximitybeacon.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# boatrace_proximitychecker

## 快速判断

- 职责与适用：与 [[reference/components/boatrace_proximitybeacon]] 成对使用：检查器在 `OnStartRace` 后每 5 帧扫描 `range`（默认 `TUNING.BOATRACE_DEFAULT_PROXIMITY`，6.0）内带 `boatrace_proximitybeacon` 标签的实体，驻留满 `found_delay`（1.5 秒）判为命中。[扫描](../../../raw/current/scripts/components/boatrace_proximitychecker.lua#L26-L47)
- 优先使用：命中业务写在 `on_found_beacon(inst, beacon)` 回调里，启停走 `OnStartRace`/`OnFinishRace`；原版在船赛起点（`boatrace_start.lua:882-883`）与检查点（`boatrace_checkpoint.lua:222-223`）各配一个回调。[起点配置](../../../raw/current/scripts/prefabs/boatrace_start.lua#L882-L883)
- 重要副作用：命中时向信标实体推 `found_by_boatrace_checker` 事件，并把该信标的记录时间改写为当前时间 + 20 游戏天，同一次比赛中不会重复触发。[防重触发](../../../raw/current/scripts/components/boatrace_proximitychecker.lua#L40-L45)
- 修改边界：扫描由 `OnStartRace` 显式开启——没有比赛调用就不会有任何检测；构造只加 `boatrace_proximitychecker` 标签不起任务。[启动点](../../../raw/current/scripts/components/boatrace_proximitychecker.lua#L63-L67)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `on_found_beacon` | `fn(inst, beacon)` 或 nil | 命中回调，宿主必须赋值才有业务效果。[调用点](../../../raw/current/scripts/components/boatrace_proximitychecker.lua#L43-L45) |
| `range` | number（默认 6.0） | 扫描半径，初值取 `TUNING.BOATRACE_DEFAULT_PROXIMITY`（`tuning.lua:8152`）。[默认值](../../../raw/current/scripts/components/boatrace_proximitychecker.lua#L9) |
| `found_delay` | number（默认 1.5） | 驻留判定时长（秒）；首见记录时间，之后某帧满足"已驻留 ≥1.5 秒"才命中。[判定实现](../../../raw/current/scripts/components/boatrace_proximitychecker.lua#L37-L41) |
| `proximity_check_must_flags` | 标签数组（默认 `{"boatrace_proximitybeacon"}`） | FindEntities 的 must 条件，可覆盖以改目标集。[默认值](../../../raw/current/scripts/components/boatrace_proximitychecker.lua#L8) |
| `OnStartRace()` | `-> nil` | 幂等开启 `DoPeriodicTask(5*FRAMES)` 扫描；起点在开赛计时结束时调（`boatrace_checkpoint.lua:118` 同），起点 prefab 在 `boatrace_start.lua:466` 亦调用。[实现](../../../raw/current/scripts/components/boatrace_proximitychecker.lua#L63-L67) |
| `OnFinishRace()` | `-> nil` | 取消扫描任务并清空两份记录表；当前脚本库内无调用者，比赛收尾靠实体移除。[实现](../../../raw/current/scripts/components/boatrace_proximitychecker.lua#L69-L81) |
| `found_by_boatrace_checker` | 实体事件（推给信标） | 命中通知，data 为检查器实体；信标侧可监听做反馈。[推送点](../../../raw/current/scripts/components/boatrace_proximitychecker.lua#L42) |

## 典型接入

### 自定义赛道检查点（原版参照）

- 前提与选择：目标（船/赛艇）需挂 boatrace_proximitybeacon（自动带标签），检查点实体挂本组件并赋 `on_found_beacon`；检查点在比赛开始时机调 `OnStartRace`。[检查点回调](../../../raw/current/scripts/prefabs/boatrace_checkpoint.lua#L222-L223)
- 生效与撤销：扫描开启后持续检测；命中记录驻留时间防止重复计分，`OnFinishRace` 或实体移除彻底清场。

```lua
-- 服务端：检查点实体接入
inst:AddComponent("boatrace_proximitychecker")
inst.components.boatrace_proximitychecker.on_found_beacon = function(inst, beacon)
    inst:PushEvent("beacon_reached_checkpoint", { beacon = beacon, checkpoint = inst })
end
-- 比赛开始时：
inst.components.boatrace_proximitychecker:OnStartRace()
```

- 调整与验证：命中节流固定为 20 游戏天，短周期复赛的检查点需自行在 `OnFinishRace` 清 `stored_beacons`（该方法本就负责清表，但需有调用时机）。[节流写入](../../../raw/current/scripts/components/boatrace_proximitychecker.lua#L41)

## 常见陷阱与限制

- **忘调 OnStartRace：** 组件不会自己开始扫描，比赛开始事件需宿主显式转发；未启动时一切回调静默不发生。[显式启动](../../../raw/current/scripts/components/boatrace_proximitychecker.lua#L63-L67)
- **OnFinishRace 无人调用：** 原版比赛结束靠移除起点/检查点实体，组件方法成了预留能力；长生命周期赛道复用时要自己安排清理时机，否则 20 天节流让信标"永远命中过"。
- **目标不是信标：** 覆盖 `proximity_check_must_flags` 后目标需有 `_activatefn` 之外的兼容性——命中逻辑只依赖实体引用与事件推送，但 `found_delay` 驻留语义是按移动目标设计的，静态目标首帧即开始计时。[驻留计时](../../../raw/current/scripts/components/boatrace_proximitychecker.lua#L34-L41)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（82 行），核对 boatrace_checkpoint.lua:118/222-223、boatrace_start.lua:466/882-883 的启停与回调配置、信标组件的标签与事件（`components/boatrace_proximitybeacon.lua`）；全库 grep 确认 `OnFinishRace` 无外部调用。
- 结构检查：2026-10-28 validate_page.py 通过；validate_source_links.py --strict 锚点通过（见批级记录）。
- 游戏运行：未运行。
