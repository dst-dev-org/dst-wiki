---
title: "yotc_raceprizemanager"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "卡拉特赛跑裁判与奖品池：按起点线登记赛程与检查点，全员完赛发幸运金块，每日限发一次并广播奖品状态。"
aliases: ["卡拉特赛跑管理", "赛跑奖品", "yotc_raceprizemanager"]
tags: [component, state-machine]
dst_build: "steam-buildid:26230496"
sources: ["components/yotc_raceprizemanager.lua", "prefabs/cave.lua", "prefabs/forest.lua", "components/yotc_racecompetitor.lua", "actions.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# yotc_raceprizemanager

## 快速判断

- 职责与适用：挂在世界实体（地表 [forest](../../../raw/current/scripts/prefabs/forest.lua#L696) 与洞穴 [cave](../../../raw/current/scripts/prefabs/cave.lua#L544) 都装，断言仅主机 [L7](../../../raw/current/scripts/components/yotc_raceprizemanager.lua#L7)），按起点线实体分桶管理多场卡拉特赛：报名、开跑、检查点、名次与奖品。
- 优先使用：查奖品可用 `HasPrizeAvailable()`（[L87-L89](../../../raw/current/scripts/components/yotc_raceprizemanager.lua#L87-L89)）；赛程查询用 `GetRaceByRacer`／`GetRaceById`／`IsRaceUnderway`（[L150-L172](../../../raw/current/scripts/components/yotc_raceprizemanager.lua#L150-L172)）。
- 重要副作用：发奖日推 `yotc_ratraceprizechange` 世界事件（神龛据此刷新，[消费](../../../raw/current/scripts/prefabs/yotc_carratshrine.lua#L206)）；比赛音乐状态写参赛者 `_musicstate` net 变量（[L124-L136](../../../raw/current/scripts/components/yotc_raceprizemanager.lua#L124-L136)）。
- 修改边界：奖品每世界每天一次（`_prize` 存上次发奖天数，[L38-L59](../../../raw/current/scripts/components/yotc_raceprizemanager.lua#L38-L59)）；存档只存 `prize_date`（[L255-L271](../../../raw/current/scripts/components/yotc_raceprizemanager.lua#L255-L271)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `RegisterRacer(racer, start_line)` | → `bool, old_racer` | 只收有 `yotc_trainer` 实体关联或幽灵鼠的参赛者；同训鼠人重复报名返回旧鼠供调用方换鼠（[L91-L122](../../../raw/current/scripts/components/yotc_raceprizemanager.lua#L91-L122)；[调用](../../../raw/current/scripts/prefabs/carrat.lua#L393)）。 |
| `BeginRace(start_line)` | 起点线实体 | 置 `race_begun`、推 `yotc_racebegun`，开 4s 周期主题音乐任务（[L138-L148](../../../raw/current/scripts/components/yotc_raceprizemanager.lua#L138-L148)）。 |
| `RegisterCheckpoint(racer, checkpoint)` | 双实体 | 记录已通过的检查点，终赛逐一推 `yotc_race_over`（[L217-L222](../../../raw/current/scripts/components/yotc_raceprizemanager.lua#L217-L222)、[L182-L189](../../../raw/current/scripts/components/yotc_raceprizemanager.lua#L182-L189)；[消费](../../../raw/current/scripts/components/yotc_racecompetitor.lua#L178-L179)）。 |
| `RacerFinishedRace(racer, distance)` | → `bool` 是否第一名 | 首个完赛者记 `results.first_place` 与距离；全员完赛或仅剩者已完赛即 `GivePrizes`＋`EndOfRace`（[L224-L244](../../../raw/current/scripts/components/yotc_raceprizemanager.lua#L224-L244)；[消费](../../../raw/current/scripts/components/yotc_racecompetitor.lua#L245-L246)）。 |
| `GivePrizes(race)` | race 数据表 | 奖品可用时按距离与完赛率算第一名奖励（≥2 金块），安慰奖 1 金块，经 `yotc_racecompetitor:OnAllRacersFinished` 发放（[L38-L59](../../../raw/current/scripts/components/yotc_raceprizemanager.lua#L38-L59)）。 |
| `RemoveRacer(racer)` | racer 实体 | 退出／死亡／失联时清音乐、减报名数，末鼠触发终赛（[L195-L215](../../../raw/current/scripts/components/yotc_raceprizemanager.lua#L195-L215)；[消费](../../../raw/current/scripts/components/yotc_racecompetitor.lua#L34-L35)）。 |

## 奖品规则（关键流程）

1. 可用判定：`_prize < TheWorld.state.cycles`（今天还没发过）；发奖后 `_prize = cycles`（[L38-L59](../../../raw/current/scripts/components/yotc_raceprizemanager.lua#L38-L59)、[L87-L89](../../../raw/current/scripts/components/yotc_raceprizemanager.lua#L87-L89)）。
2. 数额：`distancebonus = floor(距离/(CHECKPOINT_FIND_DIST-2))` 封顶 `YOTC_RACE_MAX_DISTANCE_REWARDS`，`multiplier = 完赛数/报名数`，`first_prize = max(2, floor(multiplier×(distancebonus+3)))`，安慰奖 1（[L42-L47](../../../raw/current/scripts/components/yotc_raceprizemanager.lua#L42-L47)）。
3. 状态广播：跨天到发奖日（WatchWorldState cycles）推 `yotc_ratraceprizechange`；读档 retrofit 旧 0/1 值后再推一次（[L64-L68](../../../raw/current/scripts/components/yotc_raceprizemanager.lua#L64-L68)、[L261-L271](../../../raw/current/scripts/components/yotc_raceprizemanager.lua#L261-L271)）。

## 常见陷阱与限制

- **`IsFirstPlaceRacer` 是坏掉的死代码**：条件 `race.results == nil and race.results.first_place == racer` 在 results 为 nil 时必然索引报错、results 存在时恒返回 false，且快照内无调用者；需要判第一名应比对 `race.results.first_place`（[L246-L249](../../../raw/current/scripts/components/yotc_raceprizemanager.lua#L246-L249)；全库 grep 无调用）。
- **比赛状态不存档**：`_races`／检查点／results 全在内存，重开世界即清空；只有奖品日期持久化，比赛中重启会"白跑"。
- **起点线实体是赛程键**：`_races` 以起点线实体为键，起点线被移除后其 race 记录需靠 EndOfRace 链清理；直接删起点线实体会留下悬挂数据（[L94-L102](../../../raw/current/scripts/components/yotc_raceprizemanager.lua#L94-L102)）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。覆盖组件全文、两处装配行、yotc_racecompetitor 五处消费、actions.lua START_CARRAT_RACE、yotc_carrat_race_start 与 yotc_carratshrine 调用行；未覆盖 yotd（龙舟）同名组件差异与起点线 prefab 全流程。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、Lua 语法探测通过（见批级记录）。

相关：[[reference/components/yotc_racestart]]（起点开关）、[[reference/components/yotd_raceprizemanager]]（龙舟同名管理器）。
