---
title: "worldsettingstimer"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "实体级命名计时器，按 maxtime 比例存档并支持世界设置缩放（暂停/恢复/变速），供刷怪间隔、再生时间等随世界难度调整。"
aliases: ["世界设置计时器", "worldsettingstimer", "设置缩放计时"]
tags: [component, callback]
dst_build: "steam-buildid:26230496"
sources: ["components/worldsettingstimer.lua", "worldsettingsutil.lua", "prefabs/world.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# worldsettingstimer

## 快速判断

- 职责与适用：给单个实体管理若干具名计时器，倒计时存档时按 `timeleft/maxtime` 比例保存（[OnSave](../../../raw/current/scripts/components/worldsettingstimer.lua#L176-L190)），配合世界设置改动重载后自动缩放；典型用途是 spawn/regen 延迟。
- 优先使用：一般经 `worldsettingsutil.lua` 的包装函数按世界设置自动挂载与启停（[包装](../../../raw/current/scripts/worldsettingsutil.lua#L89-L111)）；手动用时必须先 `AddTimer` 再 `StartTimer`（[校验](../../../raw/current/scripts/components/worldsettingstimer.lua#L62-L69)）。
- 重要副作用：计时完成先 `StopTimer` 再触发注册 callback，无 callback 时推 `timerdone` 事件（[完成](../../../raw/current/scripts/components/worldsettingstimer.lua#L53-L60)）。
- 修改边界：`enabled=false` 的计时器视同暂停（[IsPaused](../../../raw/current/scripts/components/worldsettingstimer.lua#L98-L100)）；`LongUpdate` 跳过 `externallongupdate`、暂停中及 `blocklongupdate` 的计时器（[跳秒](../../../raw/current/scripts/components/worldsettingstimer.lua#L205-L214)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `AddTimer(name, maxtime, enabled, callback, externallongupdate)` | name: string | 登记计时器定义；重名忽略；若存档有待恢复数据立即按比例 `StartTimer`（[实现](../../../raw/current/scripts/components/worldsettingstimer.lua#L7-L27)）。 |
| `StartTimer(name, time, paused, initialtime_override, blocklongupdate)` | time: 秒 | 启动倒计时，内部 `DoTaskInTime`；活跃中重复启动直接拒绝（[实现](../../../raw/current/scripts/components/worldsettingstimer.lua#L62-L80)）。 |
| `StopTimer(name)` | → `nil` | 清 timeleft 并取消任务（[实现](../../../raw/current/scripts/components/worldsettingstimer.lua#L82-L96)）。 |
| `ActiveTimerExists(name)` | → `bool` | 计时器已定义且 timeleft 非 nil，即"正在倒计时"（[实现](../../../raw/current/scripts/components/worldsettingstimer.lua#L49-L51)）；原版最常用的查询。 |
| `PauseTimer(name, blocklongupdate)` / `ResumeTimer(name)` | → `nil`/`bool` | 暂停前先 `GetTimeLeft` 固化剩余时间；恢复仅在 enabled 时重建任务（[实现](../../../raw/current/scripts/components/worldsettingstimer.lua#L102-L130)）。 |
| `GetTimeLeft(name)` / `SetTimeLeft(name, time)` | → `number or nil` | 未暂停时按 `end_time - GetTime()` 实时刷新；Set 对暂停中只改记录，否则暂停-改-恢复（[实现](../../../raw/current/scripts/components/worldsettingstimer.lua#L132-L151)）。 |
| `SetMaxTime(name, time)` | time: 秒 | 改 maxtime 并按新旧比例缩放 timeleft（[实现](../../../raw/current/scripts/components/worldsettingstimer.lua#L153-L167)）。 |
| `GetTimeElapsed(name)` | → `number or nil` | `initial_time - timeleft`（[实现](../../../raw/current/scripts/components/worldsettingstimer.lua#L169-L174)）。 |

## 典型接入

### 随世界设置缩放的再生延迟

- 前提与选择：把间隔定义成 maxtime，`worldsettingsutil` 依据 `WORLDSETTINGS` 前缀自动挂组件并启停，无需模组自己处理难度缩放。
- 生效与撤销：计时结束触发 callback 或 `timerdone` 事件；`OnLoad` 时若组件定义尚未 `AddTimer`，数据暂存 `saved_timers` 等登记后恢复（[恢复](../../../raw/current/scripts/components/worldsettingstimer.lua#L22-L26)）。
- 原版依据：世界实体自身挂载（[world](../../../raw/current/scripts/prefabs/world.lua#L580)）；草丛/盐堆再生（[grass](../../../raw/current/scripts/prefabs/grass.lua#L165)、[saltstack](../../../raw/current/scripts/prefabs/saltstack.lua#L296)）、蟹王/龙蝇/蚁狮刷怪（[crabking_spawner](../../../raw/current/scripts/prefabs/crabking_spawner.lua#L64)）等 10 处；`childspawner`/`pickable` 经 worldsettingsutil 接入 spawn/regen 期（[childspawner 集成](../../../raw/current/scripts/worldsettingsutil.lua#L39-L65)）。

```lua
-- 服务端实体上：定义 1 天再生计时并启动（enabled=true）
inst:AddComponent("worldsettingstimer")
inst.components.worldsettingstimer:AddTimer("myregen", TUNING.TOTAL_DAY_TIME, true, function(inst) print("done") end)
inst.components.worldsettingstimer:StartTimer("myregen", TUNING.TOTAL_DAY_TIME)
-- 查询：
inst.components.worldsettingstimer:ActiveTimerExists("myregen") -- true
```

- 调整与验证：time 单位为游戏秒并参与 LongUpdate 跳秒；监听 `timerdone` 事件时事件数据为 `{ name = 计时器名 }`（[推事件](../../../raw/current/scripts/components/worldsettingstimer.lua#L58)）。

## 常见陷阱与限制

- `AddTimer` 未先行时 `StartTimer` 只打印并返回，不报错（[打印](../../../raw/current/scripts/components/worldsettingstimer.lua#L63-L65)），静默失效易被忽略。
- 暂停态写入 `SetTimeLeft` 只更新记录不重建任务，恢复后才会按新值倒计时（[分支](../../../raw/current/scripts/components/worldsettingstimer.lua#L144-L149)）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，覆盖组件全文、worldsettingsutil 接入函数与 world/grass/saltstack/crabking_spawner 等挂载点；未运行游戏。
- 结构检查：validate_page.py、validate_source_links.py 与 Lua 语法探针通过。
