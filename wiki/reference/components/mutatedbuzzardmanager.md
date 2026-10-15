---
title: "mutatedbuzzardmanager"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "变异秃鹫群世界管理器：向 migrationmanager 登记秃鹫 gestalt 迁徙类型，把种群附到玩家头顶盘旋，按月相推进并管理尸体保留与弃置。"
aliases: ["变异秃鹫", "秃鹫迁徙", "mutatedbuzzardmanager 组件"]
tags: [component, ai]
dst_build: "steam-buildid:26230496"
size_class: standard
sources: ["components/mutatedbuzzardmanager.lua", "prefabs/forest.lua", "prefabs/buzzard.lua", "components/mutatedbuzzardcircler.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# mutatedbuzzardmanager

## 快速判断

- 职责与适用：仅主机的 forest 世界组件（:17 断言；prefabs/forest.lua:742）；与 corpsepersistmanager、migrationmanager 协作（:9-13 头注释）——注册 `MIGRATION_TYPES.MUTATED_BUZZARD_GESTALT` 迁徙类型（constants.lua:2984）。
- 优先使用：登记新秃鹫实例 → 世界事件 `ms_registermutatedbuzzard`（:298，buzzard.lua:511-512 推）；查"弃置中" → `GetDropBuzzards()`（:420-422，mutatedbuzzardcircler.lua:272 消费）。
- 重要副作用：玩家被附身后每 `TOTAL_DAY_TIME` 清一次跟踪并把种群迁去下节点、清盘旋影（:424-440）；死亡与超燃事件写 node 吸引计时（:230-262）。
- 修改边界：每玩家盘旋影上限 10（`MUTATEDBUZZARD_MAX_SHADOWS`，:37）；月球传送门不活跃即进入弃置（`RIFT_INACTIVE`，:264-267），弃置源列表 `DROP_BUZZARDS_REASONS.WINTER` 定义但本组件内未使用（:39-43）。

## 关键接口

| 接口/字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SetDropBuzzardsSource`/`GetDropBuzzards` | `(source, boolval) -> nil`／`() -> boolean` | SourceModifierList 布尔源；置真后每 3s 对种群全体执行弃置（:488-497，`UPDATE_DROP_BUZZARD_SECONDS`=3）。:416-422 |
| `TrackPopulationOnPlayer`/`ClearPopulationTracking` | `(player, population) -> nil` | 跟踪一天；清理时迁移种群并 `KillShadow` 全部影（:424-440、:175-181）。 |
| 迁徙类型配置（局部） | `CreateMigrationType` | 5 节点路径、群上限 10、基础迁徙时间 `MUTATEDBUZZARD_MIGRATE_TIME_BASE`(=4 seg)+var(=2 seg)（tuning.lua:9055-9056）；非夜晚 ×`RIFT_BIRD_MIGRATE_TIME_NON_NIGHT_FACTOR`(=1/1.5)、按 `moonphase` 乘 `RIFT_BIRD_MIGRATE_TIME_MOONPHASE_FACTOR`（tuning.lua:9044/:9053）；跟玩家的群不迁徙（:341-349）。 |
| `CorpsePersistFn`（局部） | `(corpse) -> boolean` | 海上、小虫、buzzard 自身不保留；节点内该种群 >0 或 25 距内（`MUTATEDBUZZARD_CORPSE_RANGE`，tuning.lua:9058）有活秃鹫/影则保留（:377-393）。 |
| `OnSave`/`OnLoad` | 常规存取 | 只存 `megaflare_nodes`/`death_nodes` 计时（:547-570）；秃鹫与影不入本组件存档。 |
| 影/秃鹫关联（局部） | `SpawnBuzzardShadow`/互删回调 | `circlingbuzzard_lunar` 影与 `buzzard.shadow` 双向绑定，onremove 互清（:98-130）。 |

## 关键流程：附身与弃置

1. 附身：每 0.5s（`UPDATE_TIME_SECONDS`，:23）扫描玩家迁徙节点——未跟群时找节点内首个无影种群 `TrackPopulationOnPlayer` 并生成盘旋影；已跟群时节点一致且相邻 ≤1 才跟随迁移，否则视为失效清理（:499-540）。
2. 弃置：`OnRiftStateUpdate` 按月球传送门活跃性置源（:264-267）；弃置期间每 3s `ForEachEntityInMigration` 逐体执行——有影让影 `DropBuzzard`，无影找同节点玩家接手，否则移除实体（:148-163）。
3. 吸引计时：`megaflare_detonated`（一天，tuning.lua:9066）与 `entity_death` 有尸（半天，tuning.lua:9068）写节点表并随 UpdateTimers 衰减，供路径决策 TODO 使用（:313-318 注释声明尚未接入）。
4. 尸体保留：OnPostInit 向 corpsepersistmanager 注册 `mutatedbuzzard_corpse_persist` 源（:406-409）。

## 典型接入

### 查询/扩展弃置状态（mod 侧）

- 前提与选择：仅 forest 主机存在，调用前判 nil；`SetDropBuzzardsSource` 是公开接口，可加自定义源（注意源未移除则永不恢复）。
- 生效与撤销：弃置是持续状态，`SourceModifierList` 任意源为真即弃置；撤销必须按同名 source 置 false。
- 原版依据：components/mutatedbuzzardmanager.lua:264-267、:416-422。

```lua
if TheWorld.ismastersim then
    local mgr = TheWorld.components.mutatedbuzzardmanager
    if mgr then
        mgr:SetDropBuzzardsSource("my_reason", true)  -- 开始弃置
        -- mgr:SetDropBuzzardsSource("my_reason", false) -- 结束
    end
end
```

- 调整与验证：弃置速率固定 3s 一轮不可调；`GetDebugString` 返回空串（:580-582），调试用 `Debug_GetPlayerData`（:576-578）。

## 常见陷阱与限制

- 路径决策是占位实现：`UpdatePopulationMigrationPath` 注释明言"随机选邻节点，不好！"，megaflare/death 吸引表虽在衰减但未参与决策（:313-318）——调优需重写该回调。
- `ms_registermutatedbuzzard` 只在组件存在时推（buzzard.lua:511）：组件判 nil 后注册事件才挂上，晚于秃鹫生成不会补注册。
- 种群附身按"首个无影种群"分配：`FilterPopulationFn` 跳过已有影的个体（:132-135），多玩家同节点会分到同一种群。
- 影实体与 buzzard 双向引用：任意一方外部强制移除时靠 onremove 回调解链（:98-115），绕过事件直接置 nil 会留悬挂引用。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，通读组件全部 589 行；挂载 forest.lua:742，消费方 buzzard.lua:509-512、mutatedbuzzardcircler.lua:64/:272、TUNING tuning.lua:9039-9068、MIGRATION_TYPES constants.lua:2984 逐行核对。
- 游戏运行：未运行。

## 相关链接

- [[reference/components/mutatedbuzzardcircler]]
- [[reference/components/corpsepersistmanager]]
