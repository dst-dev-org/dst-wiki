---
title: "moosespawner"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "春季驼鹿(Moose)生成调度世界组件：进入春季且过了新手保护期后，按密度抽样驼鹿巢并经软/硬两条路径生成 moose 与 mooseegg。"
aliases: ["驼鹿生成调度", "春季驼鹿刷新", "moosespawner"]
tags: [component, creature]
dst_build: "steam-buildid:26230496"
size_class: simple
sources: ["components/moosespawner.lua", "prefabs/forest.lua", "prefabs/mooseegg.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# moosespawner

## 快速判断

- 职责与适用：挂 forest 世界（prefabs/forest.lua:657），仅服务器（assert，components/moosespawner.lua:12）；`WatchWorldState("isspring")` 在进入春季且 `TheWorld.state.cycles > TUNING.NO_BOSS_TIME`（26，tuning.lua:3041）时初始化巢（:100-106）。
- 优先使用：改生成密度→`TUNING.MOOSE_DENSITY`（0.5，tuning.lua:6015，即 50% 巢会被启用）；组件无 OnSave，季节抽样不持久化。
- 重要副作用：`DoSoftSpawn`/`DoHardSpawn` 直接 SpawnPrefab `moose`/`mooseegg`；给巢置 `mooseIncoming` 与 `CallMoose` timer（:84-87）。
- 修改边界：生成位置由巢/蛋坐标决定；`OverrideAttackDensity` 为已废弃空函数（:47-49）。

## 关键接口

| 接口/字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `InitializeNests` | `() -> nil` | 遍历 Ents 找 `moose_nesting_ground`（:31-41），按密度 `PickSome` 抽样并逐巢 `StartTimer("CallMoose", SEG_TIME*random(8,24))`。:89-98 |
| `DoSoftSpawn` | `(nest) -> nil` | 巢上方 30 高度生成 moose 进 `glide` 状态，moose 上 `WantsToLayEgg` timer；由蛋的 playerprox 玩家靠近触发（prefabs/mooseegg.lua:175-178、:221）。:51-62 |
| `DoHardSpawn` | `(nest) -> nil` | 生成 mooseegg+相邻 moose 并互 `TrackEntity`、`egg:InitEgg()`；巢休眠整段 CallMoose 后由 mooseegg 定时器触发（mooseegg.lua:167-172）。:64-82 |
| `InitializeNest` | `(nest) -> nil` | 单巢启用：置 timer 与 mooseIncoming。:84-87 |

## 典型接入

### 原版参照：巢→蛋/亲鸟链路

- 前提与选择：组件仅被 mooseegg.lua:171、:178 两处外部调用，mod 不直调组件；干预入口是巢的 timer 数据与 `TUNING.MOOSE_DENSITY`。
- 生效与撤销：抽样只在春季切换瞬间执行一次；世界重载后由下次春季重建 `_seasonalnests`，巢上的 timer/mooseIncoming 随 moose_nesting_ground 自身存档保留（mooseegg.lua:180 起有 nest_onsave）。
- 不提供代码（无已核验的直调需求）。

## 常见陷阱与限制

- `FindNests` 全量遍历 Ents 找 prefab（:31-41），开销集中在春季切换一帧。
- 密度抽样结果 `_seasonalnests` 不入存档，"哪些巢今年有驼鹿"在重载后不保证一致（组件无 OnSave/OnLoad）。
- `cycles > TUNING.NO_BOSS_TIME` 表示新开档首春不生成驼鹿（:101）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，通读组件全部 108 行，核验挂载（forest.lua:657）与仅有的两个外部调用点（mooseegg.lua:171/178）；MOOSE_DENSITY、NO_BOSS_TIME 取值经 tuning.lua 核对。
- 游戏运行：未运行。
