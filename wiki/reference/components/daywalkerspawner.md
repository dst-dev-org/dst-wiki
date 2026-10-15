---
title: "daywalkerspawner"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "洞穴 daywalker（挖掘者）生成器：仅在跨分片侧记录「当前被囚于洞穴」时按天数择地造竞技场，清场并围柱。"
aliases: ["洞穴挖掘者生成", "daywalkerspawner"]
tags: [component, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/daywalkerspawner.lua", "prefabs/cave.lua", "components/shard_daywalkerspawner.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# daywalkerspawner

## 快速判断

- 职责与适用：挂在洞穴世界实体（[安装](../../../raw/current/scripts/prefabs/cave.lua#L552)，[断言](../../../raw/current/scripts/components/daywalkerspawner.lua#L7)），生成 `daywalker` 及其三根石柱竞技场；与 [[reference/components/shard_daywalkerspawner]]、地表 [[reference/components/forestdaywalkerspawner]] 构成三点协作。
- 优先使用：模组一般不直接调；生成点登记用 `ms_registerdaywalkerspawningground` 世界事件（[监听](../../../raw/current/scripts/components/daywalkerspawner.lua#L16)，[推事件](../../../raw/current/scripts/prefabs/daywalkerspawningground.lua#L17)）。
- 重要副作用：选定竞技场清场——2.5 瓦片半径内 structure／植物／可工作目标被销毁或移除（[L83-L112](../../../raw/current/scripts/components/daywalkerspawner.lua#L83-L112)）。
- 修改边界：整链受 `TUNING.SPAWN_DAYWALKER` 门控（[L231-L239](../../../raw/current/scripts/components/daywalkerspawner.lua#L231-L239)）；`daywalker` 在场时不重复生成（[L194-L197](../../../raw/current/scripts/components/daywalkerspawner.lua#L194-L197)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `TryToRegisterSpawningPoint(spawnpoint)` | → `bool` | 去重登记生成点；`RegisterDayWalkerSpawningPoint` 注释说明存在供模组直接调（[L24-L37](../../../raw/current/scripts/components/daywalkerspawner.lua#L24-L37)）。 |
| `TryToSpawnDayWalkerArena()` | → daywalker 或 nil | 打乱候选→`FindBestSpawningPoint`→瓦片中心造场；失败返回 nil 且不清空计数（[L125-L192](../../../raw/current/scripts/components/daywalkerspawner.lua#L125-L192)）。 |
| `IncrementPowerLevel()` / `GetPowerLevel()` | 上限 2 | daywalker 被击败后 +1；快照内无读取者，`GetPowerLevel` 属预留接口（[L60-L66](../../../raw/current/scripts/components/daywalkerspawner.lua#L60-L66)）。 |
| `OnPostInit()` | 无参 | world 通用分发调用；`SPAWN_DAYWALKER` 开启时监听 cycles 并对"剩余天数为 0"立即尝试一次（[L231-L239](../../../raw/current/scripts/components/daywalkerspawner.lua#L231-L239)）。 |
| `OnSave` / `OnLoad` / `LoadPostPass` | days_to_spawn／power_level／daywalker_GUID 引用 | 天数按 `TUNING.DAYWALKER_RESPAWN_DAYS_COUNT(10)` 封顶；读档重挂 daywalker 监听（[L241-L276](../../../raw/current/scripts/components/daywalkerspawner.lua#L241-L276)）。 |

## 生成条件与流程（关键流程）

1. 跨分片门控：每次昼夜切换（`OnDayChange`）先问 [[reference/components/shard_daywalkerspawner]]——只有其记录的位置名等于 `"cavejail"`（daywalker 当前被囚于洞穴）才继续倒计时（[L199-L202](../../../raw/current/scripts/components/daywalkerspawner.lua#L199-L202)；地表侧对称见 [[reference/components/forestdaywalkerspawner]]）。
2. 天数倒计时：`days_to_spawn > 0` 先扣减返回；归 0 才尝试生成，成功后重置为 10 天（[L205-L218](../../../raw/current/scripts/components/daywalkerspawner.lua#L205-L218)；[常量](../../../raw/current/scripts/tuning.lua#L6913-L6915)）。
3. 择点：3×3 瓦片全在地面＋35 半径无玩家＋无蚁狮陷坑＋优先无建筑者；全不理想退化为建筑最少者，再退化为 5/10/15 环找可走偏移（[L125-L180](../../../raw/current/scripts/components/daywalkerspawner.lua#L125-L180)）。
4. 造场：清场规则区分 structure（可工作则 Destroy 否则移除）／植物树木（移除）／NPC_workable（保留）；随后生成 3 根 `daywalker_pillar` 围住 daywalker（[L83-L123](../../../raw/current/scripts/components/daywalkerspawner.lua#L83-L123)）。
5. 战败记录：监听 daywalker onremove，`defeated` 则 `IncrementPowerLevel` 并 `Shard_SyncBossDefeated("daywalker")` 广播跨分片（[L220-L229](../../../raw/current/scripts/components/daywalkerspawner.lua#L220-L229)）。

## 常见陷阱与限制

- **生成点候选来自世界生成**：`daywalkerspawningground` 是地图生成摆件；全无候选时 `FindBestSpawningPoint` 返回 nil，本周期直接放弃且不重试直到下次昼夜（[L130-L131](../../../raw/current/scripts/components/daywalkerspawner.lua#L130-L131)）。
- **读档先 OnLoad 后 LoadPostPass**：`LoadPostPass` 直接取 `data.daywalker_GUID`，依赖存档引用解析；daywalker 未随档恢复时监听丢失（[L268-L276](../../../raw/current/scripts/components/daywalkerspawner.lua#L268-L276)）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。覆盖组件全文、cave 装配行、shard_daywalkerspawner 的 cavejail 位置名、daywalkerspawningground 事件两端、DAYWALKER 调 tuning 段；未覆盖 daywalker prefab 内部与 pillar 行为。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、Lua 语法探测通过（见批级记录）。

相关：[[reference/components/shard_daywalkerspawner]]、[[reference/components/forestdaywalkerspawner]]。
