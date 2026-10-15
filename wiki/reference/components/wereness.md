---
title: "wereness"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "Woodie 变身值计量组件：0-100 的值按负速率自动消耗驱动变形/还原，数值与消耗速率经 player_classified 同步客户端，变化统一推 werenessdelta 事件。"
aliases: ["变身值", "伍迪变形计量", "wereness"]
tags: [component, network-sync]
dst_build: "steam-buildid:26230496"
size_class: simple
sources: ["components/wereness.lua", "prefabs/woodie.lua", "prefabs/player_classified.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# wereness

## 快速判断

- 职责与适用：仅 Woodie 挂载（prefabs/woodie.lua:1822）；max 固定 100（components/wereness.lua:21），客户端实例上 max 经 makereadonly 保护（:28-30）；current/rate 写入时同步 `player_classified.currentwereness`/`werenessdrainrate`（:1-13、:34-36）。
- 优先使用：`SetDrainRate(负值)`+`StartDraining()` 启动自然消耗；`DoDelta`/`SetPercent` 改值；`SetWereMode`/`GetWereMode` 记录当前变形来源。
- 重要副作用：每次 DoDelta 推 `"werenessdelta" {oldpercent, newpercent, overtime}`（:73）；Woodie（woodie.lua:1479）、player_classified（prefabs/player_classified.lua:1179、:403 转发给玩家）与状态栏监听。
- 修改边界：current 归 0 自动 StopDraining（:69-71）；OnSave 仅 >0 时保存 current 与 mode，OnLoad 恢复并自动重启消耗（:91-105）。

## 关键接口

| 接口/字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `DoDelta` | `(delta, overtime) -> nil` | clamp 到 [0,max]；overtime=true 表示自然消耗区间。:64-74 |
| `StartDraining`/`StopDraining` | `() -> nil` | current>0 才启动组件更新；OnUpdate 按 `rate*dt` 消耗（overtime=true）。:50-62、:85-89 |
| `SetDrainRate` | `(rate) -> nil` | 原版经 `CalculateWerenessDrainRate` 计算（woodie.lua:630、:731 等）；同步侧断言 rate∈[-10,0]（:8-13）。 |
| `SetPercent`/`GetPercent` | `(percent, overtime)` / `-> number` | 按 max 换算；SetPercent 内部走 DoDelta(0) 以触发同步与事件。:76-83 |
| `SetWereMode`/`GetWereMode` | `(weremode\|nil)` | 原版存 "fullmoon" 或 weremode id（woodie.lua:712-727）。:38-44 |

## 典型接入

### 原版参照：变身决策在 Woodie 侧

- 前提与选择：wereness 只管数值与消耗；何时变形/还原、速度惩罚等全在 woodie.lua（werenessdelta 监听 :1479、载入特例 :1624 附近）。控制台调参入口 consolecommands.lua:443-449。
- 生效与撤销：载入时 current>0 自动 StartDraining，无需手动续。

## 常见陷阱与限制

- rate 同步断言 `-10 <= rate <= 0`（:10），正速率会 assert 失败——原版语义是只减不增的计量条。
- 变形态下其他系统（如变身计时）不走本组件，勿把 wereness 当通用 timer 用。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，通读组件全部 111 行，核验挂载（woodie.lua:1822）、classified 同步链（player_classified.lua:403/1179）与控制台入口；Woodie 侧 drain rate 计算与事件消费抽验 6 处。
- 游戏运行：未运行。
