---
title: "age"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "实体存活时长组件：记录跨存档的秒龄并换算天数，玩家挂载后周期上报天数成就与统计，生成类组件按玩家龄调参。"
aliases: ["实体年龄", "存活天数", "age 组件"]
tags: [component, lifecycle]
dst_build: "steam-buildid:26230496"
size_class: simple
sources: ["components/age.lua", "prefabs/player_common.lua", "prefabs/crabking.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# age

## 快速判断

- 职责与适用：`GetAge()` = saved_age + (last_pause_time or GetTime()) - spawntime - paused_time（components/age.lua:51-53）；玩家全员挂载（prefabs/player_common.lua:2803），每 `SYNC_PERIOD`=10s 与 setowner 时 syncage（age.lua:1、:21-36）。
- 优先使用：`GetAgeInDays()`（整天）/`GetDisplayAgeInDays()`（+1，展示口径）；hounded、piratespawner、spooked 等按玩家龄加权（components/hounded.lua:115、components/spooked.lua:71）。
- 重要副作用：syncage 在展示天数变化时发 survive_20/35/55/70 成就、`NotifyPlayerProgress("days")` 与 `Network:SetPlayerAge`（age.lua:3-19）。
- 修改边界：PauseAging/ResumeAging 冻结计时（玩家变鬼等，prefabs/player_common_extensions.lua:226、:274、:651）；LongUpdate（睡眠跳时）直接累加 saved_age（:94-101）。

## 关键接口

| 接口/字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `GetAge` | `() -> number` | 秒龄，含存档累计与暂停扣减。:51-53 |
| `GetAgeInDays`/`GetDisplayAgeInDays` | `() -> number` | 前者向下取整；后者 +1（"第 N 天"口径）。:55-61 |
| `PauseAging`/`ResumeAging` | `() -> nil` | last_pause_time 标记；恢复时把暂停时长记入 paused_time 并重启同步任务。:63-76 |
| `OnSave`/`OnLoad` | 常规存取 | 保存总龄与 ispaused；载入按需重启同步。:78-84、:103-113 |
| `CancelPeriodicSync`/`RestartPeriodicSync` | `() -> nil` | 同步任务管理；Cancel 会立即 syncage 一次。:38-49 |

## 典型接入

### 挂载核验：crabking 线索的真实落点

- crabking.lua 的 age 实际挂在法术生成器实体而非 boss 本体：`crabking_geyserspawner`（geyserfn，prefabs/crabking.lua:1860 挂载 :1876，喷发计时按 age Remap，:1753、:1970）与未启用的 `crabking_feeze`（freezefn，:2029 挂载 :2045，注册行 :2140 被注释）。
- 玩家之外的挂载极少；读龄方（hounded/spooked/worldoverseer 等）均先判玩家存在。

## 常见陷阱与限制

- 天数按 `GetTime()` 累计而非世界 cycles，两者不同步（evergreens.lua:531 即以 cycles 兜底再取 age）。
- 成就/统计上报依赖 `AwardPlayerAchievement`/`NotifyPlayerProgress` 等玩家服务接口（:5-18），非玩家实体挂载后勿复用 syncage。
- `GetDebugString` 以 0.5 天为界切换单位（:86-92），仅调试用。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，通读组件全部 115 行；玩家挂载（player_common.lua:2803）与 crabking.lua:1753/1860/1876/1908/1970/2029/2045/2140 逐行核对，确认线索指向的是生成器实体；读龄消费方抽验 hounded.lua:115/143、spooked.lua:71。
- 游戏运行：未运行。
