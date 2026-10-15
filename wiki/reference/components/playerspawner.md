---
title: "playerspawner：玩家出生点与上下线调度"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "世界级玩家出生组件：登记出生点与迁移门户，处理新角色落点（fixed/scatter）、读档落地、出生保护、下线淡出与跨分片迁移。"
aliases: [玩家出生点, 出生管理, playerspawner, 重生点管理]
tags: [component, server-client, network-sync]
dst_build: "steam-buildid:26230496"
sources: ["components/playerspawner.lua", "prefabs/world.lua", "networking.lua", "mainfunctions.lua", "simutil.lua", "prefabs/player_common.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# playerspawner（玩家出生点与上下线调度）

## 快速判断

- 职责与适用：所有世界（含洞穴）都挂载的服务端世界组件（[world.lua:589](../../../raw/current/scripts/prefabs/world.lua#L589)，断言 [mastersim](../../../raw/current/scripts/components/playerspawner.lua#L7)）。经 `ms_registerspawnpoint` 收集出生点、`ms_registermigrationportal` 收集分片门户；新角色经 `SpawnAtNextLocation` 落地，读档与跨分片迁移经 `SpawnAtLocation(isloading)` 落地（[落地](../../../raw/current/scripts/components/playerspawner.lua#L238-L300)）。
- 优先使用：找任一出生点坐标 → `GetAnySpawnPoint()`（等价 `GetNextSpawnPosition`，被 simutil/ocean_util/活动组件广泛调用，[登记名](../../../raw/current/scripts/components/playerspawner.lua#L302)）；驱赶玩家下线 → 推 `ms_playerdespawn*` 事件而非直接调用。
- 重要副作用：黑暗且 4 单位内无 `spawnlight` 时自动生成 `spawnlight_multiplayer` 照明；按世界生成覆盖与怪物密度可给新号加 `spawnprotectionbuff`（[保护](../../../raw/current/scripts/components/playerspawner.lua#L109-L128)）。
- 修改边界：`fixed` 模式优先 master 出生点并轮换；`scatter` 按 `easing.inQuart` 偏前随机；出生点耗尽时 open/used 两表互换循环使用（[选点](../../../raw/current/scripts/components/playerspawner.lua#L44-L79)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SpawnAtNextLocation(inst, player)` | 世界实体、玩家实体 | 取下一点并 `SpawnAtLocation`；新角色入口由 `networking.lua` 调用（[新角色](../../../raw/current/scripts/networking.lua#L269)）。 |
| `SpawnAtLocation(inst, player, x, y, z, isloading)` | 坐标、布尔 | 迁移解析（非本分片来源则落到迁移门户并搬 `migrationpets`）→Teleport→`areaaware:UpdatePosition`→补光→出生保护→按 isloading/scatter 播传送门 FX 淡入，否则推 `ms_newplayercharacterspawned`（[实现](../../../raw/current/scripts/components/playerspawner.lua#L238-L300)）。 |
| `IsPlayersInitialSpawn(player)` | 玩家 → 布尔 | 按 userid 查 `_players_spawned` 表（落过地即记），player_common 用它只给首降补资源（[判定](../../../raw/current/scripts/components/playerspawner.lua#L304-L306)、[消费](../../../raw/current/scripts/prefabs/player_common.lua#L1483)）。 |
| `GetAnySpawnPoint` | 等于 `GetNextSpawnPosition` | 无出生点打印并返回原点；fixed 无 master 点回退第一个（[取点](../../../raw/current/scripts/components/playerspawner.lua#L44-L79)）。 |
| `OnSave/OnLoad` | 存读钩子 | 只存 `_players_spawned`（[存读](../../../raw/current/scripts/components/playerspawner.lua#L313-L321)）。 |

事件接线（[注册段](../../../raw/current/scripts/components/playerspawner.lua#L211-L216)）：`ms_playerdespawn`/`ms_playerdespawnanddelete`/`ms_playerdespawnandmigrate` → 淡出后 `PlayerRemove`（保存或删会话并跨分片续迁，[下线](../../../raw/current/scripts/components/playerspawner.lua#L81-L104)、[淡出](../../../raw/current/scripts/components/playerspawner.lua#L134-L149)）；`ms_setspawnmode` → fixed/scatter；出生点带 `master` 标记者设为主点（[登记](../../../raw/current/scripts/components/playerspawner.lua#L178-L189)）。读档落地由 `mainfunctions.lua` 以 `isloading=true` 调 `SpawnAtLocation`（[读档](../../../raw/current/scripts/mainfunctions.lua#L2120)）。

## 典型接入

### 查询出生点与首降判定

- 前提与选择：服务端、世界组件已挂载；查询型调用无副作用（但 `GetNextSpawnPosition` 在 fixed 模式会消耗/轮换表项）。
- 生效与撤销：出生保护在读档落地（isloading）时跳过世界覆盖检查（[isloading 分支](../../../raw/current/scripts/components/playerspawner.lua#L113-L114)），拓扑覆盖 `spawnprotection` 可设 always/never。

```lua
-- 服务端：取一个出生点坐标并判断玩家是否首降
local x, y, z = TheWorld.components.playerspawner:GetAnySpawnPoint()
if TheWorld.components.playerspawner:IsPlayersInitialSpawn(player) then
    -- 该 userid 从未在本世界落过地
end
```

- 调整与验证：保护判定半径 16~32 分档数怪物与建筑（[阈值](../../../raw/current/scripts/components/playerspawner.lua#L120-L124)）；dev 分支下保护直接关闭（[源注](../../../raw/current/scripts/components/playerspawner.lua#L110-L112)）。

## 常见陷阱与限制

- **GetAnySpawnPoint 有轮换副作用**：它就是选点函数本体，fixed 模式每次调用会移动主点表项；高频调用会打乱出生顺序（[别名](../../../raw/current/scripts/components/playerspawner.lua#L302)、[轮换](../../../raw/current/scripts/components/playerspawner.lua#L62-L68)）。
- **出生保护不看存档进度以外的时间**：`cycles <= 1` 直接关闭保护（[条件](../../../raw/current/scripts/components/playerspawner.lua#L119)），与玩家年龄无关。
- **迁移门户必须带 worldmigrator 组件**：注册即断言，非迁移实体不能注册（[断言](../../../raw/current/scripts/components/playerspawner.lua#L198)）；组件移除时清空全局 ShardPortals（[清理](../../../raw/current/scripts/components/playerspawner.lua#L222-L226)）。
- **淡出期间玩家标记 `_despawning`**：重复触发 despawn 事件无防护，逻辑须避免重入（[标记](../../../raw/current/scripts/components/playerspawner.lua#L135)）。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496（steam-buildid:26230496）；覆盖 `components/playerspawner.lua` 全部 327 行、world.lua 挂载行、networking/mainfunctions 的两个入口、simutil/ocean_util/componentutil 的 GetAnySpawnPoint 消费、player_common 首降判定。未覆盖：`spawnprotectionbuff` debuff 内容、迁移门户 worldmigrator 全流程。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 通过；lua 示例通过 parser_probe 语法检查。
- 游戏运行：未运行。
