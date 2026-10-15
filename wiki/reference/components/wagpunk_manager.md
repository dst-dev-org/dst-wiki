---
title: "wagpunk_manager"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "世界级垃圾场事件组件：月裂开启后循环刷瓦斯塔夫机械与提示 NPC，把玩家引向垃圾场废料，支持 hint 计时与围栏布景一次性生成。"
aliases: ["瓦冈朋克管理", "wagpunk_manager"]
tags: [component, creature]
dst_build: "steam-buildid:26230496"
sources: ["components/wagpunk_manager.lua", "components/riftspawner.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# wagpunk_manager

## 快速判断

- 职责与适用：世界级组件（仅主端），装配于地表世界（[forest](../../../raw/current/scripts/prefabs/forest.lua#L714)）；月裂活动开启后循环投放 `wagstaff_machinery` 机械组与 `wagstaff_npc_wagpunk` 提示 NPC，并维护垃圾场布景围栏。
- 优先使用：模组无需主动驱动——由 [riftspawner.EnableLunarRifts 调 `Enable()`](../../../raw/current/scripts/components/riftspawner.lua#L188-L190)；调试用 `DebugForceSpawnMachine`／`DebugForceHint`（[L644-L656](../../../raw/current/scripts/components/wagpunk_manager.lua#L644-L656)）。
- 重要副作用：生成机械、笔记与 NPC 三类实体；机械增减靠世界事件 `wagstaff_machine_added`／`wagstaff_machine_destroyed` 汇报（[L75-L76](../../../raw/current/scripts/components/wagpunk_manager.lua#L75-L76)、[L37-L43](../../../raw/current/scripts/components/wagpunk_manager.lua#L37-L43)）。
- 修改边界：`OnRemoveFromEntity` 直接断言禁止移除（[L618-L620](../../../raw/current/scripts/components/wagpunk_manager.lua#L618-L620)）；不管理 wagboss 战斗本身（查询 wagboss_tracker 只用于停提示，[L156-L158](../../../raw/current/scripts/components/wagpunk_manager.lua#L156-L158)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `Enable()` / `Disable()` / `IsEnabled()` | — | 启动投放计时或清空状态； riftspawner 在月裂开启时 Enable、世界设置 "never" 时 Disable（[L116-L140](../../../raw/current/scripts/components/wagpunk_manager.lua#L116-L140)、[riftspawner](../../../raw/current/scripts/components/riftspawner.lua#L227-L235)）。 |
| `StartSpawnMachinesTimer(timeoverride)` / `StartHintTimer(timeoverride)` | 秒 | 投放计时 4 天 ±3 天随机；提示计时按 hintcount 在 1~15 天间 Lerp；两计时互斥并驱动组件更新（[L144-L168](../../../raw/current/scripts/components/wagpunk_manager.lua#L144-L168)、[tuning](../../../raw/current/scripts/tuning.lua#L8050-L8053)）。 |
| `SpawnMachines(force)` | — | 无机械（或强制）时选址、按需生成围栏与变异笔记、绕点放 3 台机械，然后转提示计时（[L506-L529](../../../raw/current/scripts/components/wagpunk_manager.lua#L506-L529)、[常量](../../../raw/current/scripts/components/wagpunk_manager.lua#L3-L4)）。 |
| `FindSpotForMachines()` | → pos, at_setpiece | 优先布景机械标记点（玩家在一屏内且"猪人当班"不成立时改为刷垃圾场 NPC 寻物线索）；否则随机拓扑节点且离上次位置 >300 距离、玩家视野外（[L333-L389](../../../raw/current/scripts/components/wagpunk_manager.lua#L333-L389)、[视野](../../../raw/current/scripts/components/wagpunk_manager.lua#L21-L25)）。 |
| `TryHinting()` | — | 随机在线玩家旁 15 距离、朝向机械 ±30 度放 `wagstaff_npc_wagpunk`；hintcount 上限 10，满或选址失败 30 秒后重试（[L411-L462](../../../raw/current/scripts/components/wagpunk_manager.lua#L411-L462)、[L3](../../../raw/current/scripts/components/wagpunk_manager.lua#L3)）。 |
| `SpawnWagstaff(pos, machinepos)` / `SpawnJunkWagstaff(pos, junkpos)` | → NPC 实体 | NPC 自带 `hunt_stage`/"expiretime" 计时，knownlocations 记目标点；注释保留给模组覆盖（[L170-L184](../../../raw/current/scripts/components/wagpunk_manager.lua#L170-L184)、[L391-L405](../../../raw/current/scripts/components/wagpunk_manager.lua#L391-L405)）。 |
| `GetBigJunk()` | → 实体或 nil | 注册的大垃圾堆（`ms_register_junk_pile_big` 事件，[L82](../../../raw/current/scripts/components/wagpunk_manager.lua#L82)、[L550-L556](../../../raw/current/scripts/components/wagpunk_manager.lua#L550-L556)）；forestdaywalkerspawner 借它定位 daywalker（[消费](../../../raw/current/scripts/components/forestdaywalkerspawner.lua#L18-L24)）。 |
| `OnUpdate(dt)`（含 `LongUpdate`） | — | 倒计时到点触发 SpawnMachines／TryHinting；wagboss 已击败则不再提示；两计时皆空即停更新（[L584-L614](../../../raw/current/scripts/components/wagpunk_manager.lua#L584-L614)）。 |
| OnSave/OnLoad | 计时、hintcount、currentnodeindex、spawnedfences | [L558-L580](../../../raw/current/scripts/components/wagpunk_manager.lua#L558-L580)。 |

## 关键流程

### 投放与引导循环

1. `Enable` → 投放计时 → `SpawnMachines`：布景点（机械标记 `ms_register_wagstaff_machinery`，[L81](../../../raw/current/scripts/components/wagpunk_manager.lua#L81)）会先按大垃圾堆到标记点角度旋转围栏布局再铺 `fence_junk`（一次性，[L291-L316](../../../raw/current/scripts/components/wagpunk_manager.lua#L291-L316)、[旋转映射](../../../raw/current/scripts/components/wagpunk_manager.lua#L245-L290)）。
2. 无变异笔记在场才补一张 `wagstaff_mutations_note`（[L519-L521](../../../raw/current/scripts/components/wagpunk_manager.lua#L519-L521)）。
3. 机械清空（全被摧毁）后重新进入投放计时（[L86-L94](../../../raw/current/scripts/components/wagpunk_manager.lua#L86-L94)）。
4. 世界创建/载入后 `DoTaskInTime(0)` 补一次布景围栏，仅一次（`spawnedfences`，[L83](../../../raw/current/scripts/components/wagpunk_manager.lua#L83)、[L532-L543](../../../raw/current/scripts/components/wagpunk_manager.lua#L532-L543)）。

## 常见陷阱与限制

- **玩家在场不直接刷**：布景点被玩家视野覆盖且无 junkwagpunk 世界计时时，改刷 NPC 并开 240~480 秒世界计时器 `junkwagpunk` 防重复（[L336-L352](../../../raw/current/scripts/components/wagpunk_manager.lua#L336-L352)）。
- **猪人占据布景点**：`IsWerepigInCharge` 查 shard_daywalkerspawner 地点与 forestdaywalkerspawner 是否有 daywalker，占据时走随机节点选址分支（[L319-L331](../../../raw/current/scripts/components/wagpunk_manager.lua#L319-L331)）。
- **围栏坐标是布景私有布局**：`fences` 表与旋转映射依赖 junk_yard1 布景的固定相对坐标，自建场景不能直接复用（[L186-L244](../../../raw/current/scripts/components/wagpunk_manager.lua#L186-L244)）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。覆盖组件全文、riftspawner Enable/Disable 两侧、forestdaywalkerspawner 消费点、tuning 段、forest 装配行；未覆盖 wagstaff_npc_wagpunk 与机械 prefab 内部状态机。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、Lua 语法探测通过（见批级记录）。
