---
title: "migrationmanager"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "世界级种群迁移组件：按拓扑节点建迁移地图，让注册的迁徙生物种群沿节点路径周期移动；变异秃鹫群是唯一原版使用者。"
aliases: ["种群迁移管理", "生物迁徙", "migrationmanager"]
tags: [component, creature]
dst_build: "steam-buildid:26230496"
sources: ["components/migrationmanager.lua", "components/mutatedbuzzardmanager.lua", "prefabs/buzzard.lua", "stategraphs/SGbuzzard.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# migrationmanager

## 快速判断

- 职责与适用：世界级组件（仅主端），装配于地表世界（[forest](../../../raw/current/scripts/prefabs/forest.lua#L741)）；把世界拓扑约简成"迁移节点图"，为各迁移类型维护种群组，按计时器把整组在节点间移动。注意与分片迁移无关（文件头注释声明，[L5-L11](../../../raw/current/scripts/components/migrationmanager.lua#L5-L11)）。
- 优先使用：注册自己的迁徙生物时调 `CreateMigrationType` 注册类型，再 `EnterMigration(type, ent)` 收编实体（[L254-L257](../../../raw/current/scripts/components/migrationmanager.lua#L254-L257)、[L413-L422](../../../raw/current/scripts/components/migrationmanager.lua#L413-L422)）；原版参照 [mutatedbuzzardmanager 的注册](../../../raw/current/scripts/components/mutatedbuzzardmanager.lua#L309-L349)。
- 重要副作用：入组实体被 `RemoveFromScene`（不可见、不可交互），出组才 `ReturnToScene`（[L194-L213](../../../raw/current/scripts/components/migrationmanager.lua#L194-L213)）；组件在 `OnPostInit` 后常驻 1/3 秒节拍更新（[L245-L248](../../../raw/current/scripts/components/migrationmanager.lua#L245-L248)、[L510-L525](../../../raw/current/scripts/components/migrationmanager.lua#L510-L525)）。
- 修改边界：迁移地图只在首次 `OnPostInit` 从 `TheWorld.topology` 构建一次（[L163-L178](../../../raw/current/scripts/components/migrationmanager.lua#L163-L178)）；组件不生成生物本体，何时把实体送进迁移系统由调用方决定。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `CreateMigrationType(migration_data)` | data 字段见下 | 注册类型并建种群容器；必须发生在读档 `OnLoad` 之前，否则存档种群无处落（[L254-L257](../../../raw/current/scripts/components/migrationmanager.lua#L254-L257)、[OnLoad 假设](../../../raw/current/scripts/components/migrationmanager.lua#L551-L570)）。 |
| `migration_data` 必需字段 | type, num_path_nodes, `GetMigrateTime()`, `GetMigrateTimeMult()`, `GetMaxGroupPopulation(population)` | 计时重置值、倒计时倍率（夜/月相调频）、组容量；可选 `CanPopulationGroupMigrate` 与 `UpdatePopulationMigrationPath` 覆盖默认随机扩路（[结构注释](../../../raw/current/scripts/components/migrationmanager.lua#L43-L76)、[默认扩路](../../../raw/current/scripts/components/migrationmanager.lua#L467-L475)）。 |
| `EnterMigration(migrator_type, ent)` / `EnterMigrationInNode(type, ent, node)` | — | 实体定位当前节点，加入未满组或新建组；**定位失败直接 `ent:Remove()`**（[L408-L422](../../../raw/current/scripts/components/migrationmanager.lua#L408-L422)）。 |
| `AddEntityToPopulationGroup(uid, ent)` / `RemoveEntityFromPopulationGroup(ent)` | → bool | 直接搬组；Remove 会 `ReturnToScene` 并清 `migrationmanager_groupuid`，组空即清组（[L194-L213](../../../raw/current/scripts/components/migrationmanager.lua#L194-L213)、[L380-L406](../../../raw/current/scripts/components/migrationmanager.lua#L380-L406)）。 |
| `GetMigrationNodeAtPoint(x[,y],z)` / `GetMigrationNodeAtInst(inst)` | → 节点名或 nil | `Map:FindNodeAtPoint` → topology id → `layout_id or task_id`，支持 `(x,z)`／Vector3（[L270-L284](../../../raw/current/scripts/components/migrationmanager.lua#L270-L284)）。 |
| `GetDistanceNodeToNode(n1, n2)` | → 步数或 nil | 构图时预计算的邻接跳数缓存（[L151-L178](../../../raw/current/scripts/components/migrationmanager.lua#L151-L178)、[L286-L291](../../../raw/current/scripts/components/migrationmanager.lua#L286-L291)）。 |
| `GetFirstPopulationGroupInNode(type, node, filterfn)` / `GetPopulationGroup(uid)` | → uid／group, type | 按节点找可入组；uid 全类型共用自增序号（[L302-L309](../../../raw/current/scripts/components/migrationmanager.lua#L302-L309)、[L293-L300](../../../raw/current/scripts/components/migrationmanager.lua#L293-L300)）。 |
| `GetPopulationForNodeAtPoint(type, x, y, z)` | → 数量 | 统计该节点上所有组实体数（[L337-L347](../../../raw/current/scripts/components/migrationmanager.lua#L337-L347)）。 |
| `ShouldPopulationGroupMigrate(type_or_data, population)` | → bool | 优先回调 `CanPopulationGroupMigrate`，缺省判非空组（[L353-L361](../../../raw/current/scripts/components/migrationmanager.lua#L353-L361)）。 |
| `ForEachEntityInMigration(type, cb)` | → cb 是否提前终止 | 遍历该类型全部实体，cb 返回真停止（[L320-L335](../../../raw/current/scripts/components/migrationmanager.lua#L320-L335)）。 |

## 关键流程

### 迁移更新（1/3 秒节拍）

1. 刷新每个在线玩家的当前迁移节点（`_activeplayers`，ms_playerjoined／left 维护，[L219-L239](../../../raw/current/scripts/components/migrationmanager.lua#L219-L239)、[L481-L485](../../../raw/current/scripts/components/migrationmanager.lua#L481-L485)）。
2. 对每个种群：迁移路径不足 `num_path_nodes` 就补一节（默认随机邻接）；`migrate_timer <= 0` 且可迁移时弹出到路径下一节点并重置计时；计时只在可迁移时按 `dt * GetMigrateTimeMult()` 倒扣（[L487-L506](../../../raw/current/scripts/components/migrationmanager.lua#L487-L506)、[L448-L465](../../../raw/current/scripts/components/migrationmanager.lua#L448-L465)）。
3. `recently_visited_nodes` 环形记录最近 `num_path_nodes` 个节点供路径回调避让（[L430-L446](../../../raw/current/scripts/components/migrationmanager.lua#L430-L446)）。

### 存档

OnSave 存各类型种群（uid、实体 GUID 列表、data）与 `lastuid`；LoadPostPass 按 GUID 重新注册实体回组（[L531-L587](../../../raw/current/scripts/components/migrationmanager.lua#L531-L587)）。

## 常见陷阱与限制

- **`GetEntityFromMigrationNode` 不按节点过滤**：实现遍历该类型所有组取第一个实体并立即出组，`node` 参数未参与筛选（[L311-L318](../../../raw/current/scripts/components/migrationmanager.lua#L311-L318)）；要按节点取实体请自查 `ForEachEntityInMigration`。
- **无效位置直接删实体**：`EnterMigration` 对定位不到节点的实体执行 `Remove()`（源码 TODO 自认粗糙，[L413-L418](../../../raw/current/scripts/components/migrationmanager.lua#L413-L418)）；先确认实体在有效地形再收编。
- **迁移中的实体在场景外**：种群移动只改组与节点数据，实体不可见；原版靠 buzzard 的睡眠计时再 `EnterMigration`（[buzzard 消费](../../../raw/current/scripts/prefabs/buzzard.lua#L310-L326)），死亡／出土路径见 [SGbuzzard](../../../raw/current/scripts/stategraphs/SGbuzzard.lua#L95)、[corpses_defs](../../../raw/current/scripts/prefabs/corpses_defs.lua#L312)；类型键为 `MIGRATION_TYPES.MUTATED_BUZZARD_GESTALT`（[constants](../../../raw/current/scripts/constants.lua#L2982-L2984)）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。覆盖组件全文、mutatedbuzzardmanager 注册段、buzzard／SGbuzzard／corpses_defs 调用点、forest 装配行；未覆盖 mutatedbuzzardmanager 的 rift/巨型耀斑驱动细节。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、Lua 语法探测通过（见批级记录）。

相关：[[reference/components/worldmigrator]]（分片门户迁移，另一套系统）。
