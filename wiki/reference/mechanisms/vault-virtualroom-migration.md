---
title: "vault→virtualroom 迁移（build 26148036）"
created: "2026-10-24"
updated: "2026-10-28"
type: mechanism
summary: "build 26148036 把金库（vault）专用判定重构为通用虚拟房间体系：map 新查询入口、暗影手/指南针/守卫等消费者的接入方式、vault_teleport 泛化与存档 5.25 自动迁移，附模组迁移判断。"
aliases: ["金库虚拟房间迁移", "vaultroommanager 替换", "vault virtualroom migration"]
tags: [mechanism, worldgen, data, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/map.lua", "components/shadowhandspawner.lua", "prefabs/vault_compass.lua", "stategraphs/SGvault_pillar_guard.lua", "map/static_layouts/vault_lobby.lua", "savefileupgrades.lua", "constants.lua", "prefabs/cave.lua", "prefabs/vault_pillar_guard.lua", "brains/vault_pillar_guardbrain.lua", "stategraphs/SGwilson.lua", "prefabs/archive_chandelier.lua", "prefabs/shadowcreature.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# vault→virtualroom 迁移（build 26148036）

## 快速判断

- 职责与适用：本页汇总本 build 将 vault 专用组件族（更新报告 P-122/P-123 已核验条目记旧名 `vaultroommanager`/`vaultroom`/`vault_floor_helper` 及 `ms_register_vault_*` 事件）替换为通用 virtualroom 体系（[[reference/components/virtualroommanager]]／[[reference/components/virtualroomset]]／[[reference/components/virtualroomteleporter]]）的影响面与模组迁移判断；旧组件与旧事件在当前源码已无定义（仅存档升级器仍读旧档字段）。[弃用声明](../../../raw/current/scripts/savefileupgrades.lua#L1423-L1434)
- 优先使用：判点在不在金库/大厅继续用 `Map:IsPointInVaultRoom` 等包装（语义=对应房间集）；新内容用 `VIRTUALROOMSETS` 常量＋房间集接口。[包装](../../../raw/current/scripts/components/map.lua#L1565-L1573)、[常量](../../../raw/current/scripts/constants.lua#L3049-L3053)
- 重要副作用：金库/竞技场身份现在由房间集注册表驱动——房间集未声明或未设原点时点查询恒 false；雨/酸雨/地震等环境查询也挂上了这些判定。[查询](../../../raw/current/scripts/components/map.lua#L925-L994)
- 修改边界：挂旧 vault 事件、读旧组件字段的模组必须迁移；只调 `IsPointInVaultRoom` 系包装的模组接口不变。tuning.lua 无 VIRTUALROOM 专属键（定向查证无匹配），调参不涉本迁移。

## 关键接口

### map.lua 查询入口（当前实现）

| 查询 | 语义与条件 | 证据 |
|---|---|---|
| `IsPointInVaultRoom(x,y,z)`／`IsPointInVaultLobby`／`IsPointInAnyVault` | 分别 = `IsPointInVirtualRoomSet(VAULT)`／`(LOBBYVAULT)`／二者之或；`IsPointInVirtualRoomSet` 为引擎侧实现（Lua 无定义） | [map.lua](../../../raw/current/scripts/components/map.lua#L1565-L1573) |
| `IsPointInOrAdjacentToAnyVault(x,y,z)` | 邻接含 VAULT 地砖（`WORLD_TILES.VAULT`）或位于金库房间 | [map.lua](../../../raw/current/scripts/components/map.lua#L1577-L1586) |
| `IsPointInCharlieBossArena(x,y,z)` | ATRIUM 房间集且**不在大厅**（`IsVirtualRoomSetInLobby` 为假）才算竞技场 | [map.lua](../../../raw/current/scripts/components/map.lua#L1588-L1591)、[中心点](../../../raw/current/scripts/components/map.lua#L1593-L1601) |
| `CanPointHaveRain(x,y,z)` | 竞技场内恒 false，其余按拓扑标签 `norain` | [map.lua](../../../raw/current/scripts/components/map.lua#L955-L963) |
| `CanPointHaveAcidRain(x,y,z)` | 以 CanPointHaveRain 为前提，且 `IsPointInAnyVault` 内 false | [map.lua](../../../raw/current/scripts/components/map.lua#L925-L949) |
| `CanPointHaveQuaker(x,y,z)`／`CanPointCauseQuaker` | 前者排除金库与竞技场；后者仅排除竞技场（爆炸推进地震计时用） | [map.lua](../../../raw/current/scripts/components/map.lua#L969-L994) |

### 常量与事件

- `MAX_VIRTUALROOMSETS=32`、`VIRTUALROOMSETS{VAULT,LOBBYVAULT,ATRIUM}`、`VIRTUALROOMCONTEXT{NONE,MARKER,TELEPORTER}`、`VIRTUALROOMDIRECTIONS_INDEX{N,E,S,W,IN,OUT}`、`VIRTUALROOMLOBBY="virtualroomlobby"`、`VIRTUALROOM_BOUNDINGBOXES`（三个房间集的包围盒/mask）。[constants.lua](../../../raw/current/scripts/constants.lua#L3048-L3121)
- 实体登记统一为世界事件 `ms_register_virtualroom_entity`／`ms_unregister_virtualroom_entity`（data 含 inst/roomsetname/context/onlyoneprefab），接收方为 virtualroommanager；原版标记、传送器、出口均按此注册。[登记入口](../../../raw/current/scripts/prefabs/vault_markers.lua#L91-L96)、[出口用例](../../../raw/current/scripts/prefabs/vault_lobby_exit.lua#L405-L413)

## 关键流程

| 入口／条件 | 当前步骤与结果 | 后续／中断／清理及证据 |
|---|---|---|
| 洞穴世界装配：`worldstaticlayouts:RegisterLayout(VAULT, CreateLayout_Vault)`（组件详见 [[reference/components/worldstaticlayouts]]，cave.lua master_postinit） | 在档案传送门附近按螺旋扫描放置 Vault_Lobby／Vault_Vault 静态布局，位置过滤器 `CreateVRSPositionFilter` 经 manager `WillBoundingBoxCollideAt` 排除与其他房间集包围盒重叠的点位 | 布局落成后其中的 vaultmarker_* 实体各自声明/登记房间集；[装配](../../../raw/current/scripts/prefabs/cave.lua#L487-L515)、[过滤器](../../../raw/current/scripts/maputil.lua#L432-L448) |
| 大厅布局接线（vault_lobby.lua，Tiled 导出） | 布局放置 `vaultmarker_lobby_center`（LOBBYVAULT 宿主）、`vault_lobby_exit`、`vault_teleporter`；传送器对象带 `data.virtualroomteleporter.direction/shuffleddirection="IN"、roomsetname="VAULT"` 属性，经 static_layout 点分解析进实体数据、由组件 OnLoad 恢复 | 本条在更新报告中此前为待审线索，本次已读源码转正；[布局对象](../../../raw/current/scripts/map/static_layouts/vault_lobby.lua#L331-L344)、[点分解析](../../../raw/current/scripts/map/static_layout.lua#L93-L110) |
| 暗影手改期 `ShouldRescheduleForVirtualRooms`（SpawnHand 前置） | 玩家在 VAULT 房间集且当前房为 `puzzle2`（熄灯房自带暗影手），或在 ATRIUM 房间集且不在大厅（查理竞技场有专属控制器）→ 改期不生成 | 本条更新报告此前未标核验，本次转正；[实现](../../../raw/current/scripts/components/shadowhandspawner.lua#L106-L128)、[调用点](../../../raw/current/scripts/components/shadowhandspawner.lua#L129-L139) |
| 金库指南针指向（vault_compass 周期任务 OnUpdateDirection） | 在 VAULT 内：按当前房 `GetClosestDirectionFromRoomToRoom(roomname,"key1")` 找方向并朝对应 TELEPORTER 上下文传送器转头，找不到播 `idle_marker_fail`；在 LOBBYVAULT 内指向 `vaultmarker_lobby_to_vault`；在外指向 archive_portal（近距 success） | 本条更新报告此前未标核验，本次转正；GetStatus 同判 KEYROOM/NOTVAULT；[指向](../../../raw/current/scripts/prefabs/vault_compass.lua#L21-L80)、[状态](../../../raw/current/scripts/prefabs/vault_compass.lua#L123-L138) |
| 存档升级 5.25（`Replacing vaultroommanager with virtualroommanager`） | 洞穴档读到 `persistdata.vaultroommanager` 时：`spawnedlayouts` 移交 worldstaticlayouts；按版本表把旧房间序号翻成房名，房间数据改写到 `vaultmarker_vault_center` 实体的 `data.virtualroomset`（含 resetting/version=1/currentroomname/roomsavedata/customdata.prngseed/repairedlinks/originalfloortiles 默认瓦序） | 无旧组件数据则跳过；旧组件版本号与 virtualroomset version 不是一套体系（注释明示）；[升级器](../../../raw/current/scripts/savefileupgrades.lua#L1423-L1434)、[版本表](../../../raw/current/scripts/savefileupgrades.lua#L1447-L1487)、[实体改写](../../../raw/current/scripts/savefileupgrades.lua#L1496-L1530) |

## SGvault_pillar_guard 与守卫金库边界（待审线索转正）

更新报告 detail-sg-brains 行记 SGvault_pillar_guard "+49/−191 内部重构"——本次核对当前源码：该状态图文件内**没有任何 virtualroom 引用**（grep 仅命中 `TUNING.VAULT_PILLAR_GUARD_*` 调参键），其重构内容是 AOE 攻击迁移到共享 aoeutil，与虚拟房间体系无关。石柱守卫的金库判定实际在 prefab 与 brain 中经 `Map:IsPointInVaultRoom`（即 VAULT 房间集包装）接入：出生/活动区域、攻击与跳跃目标的金库一致性判断、离开金库的处理。[守卫 prefab 判定](../../../raw/current/scripts/prefabs/vault_pillar_guard.lua#L216-L324)、[离库处理](../../../raw/current/scripts/prefabs/vault_pillar_guard.lua#L720)、[brain 同侧判定](../../../raw/current/scripts/brains/vault_pillar_guardbrain.lua#L24-L50)

## 典型接入

### 模组迁移判断（按挂接方式分列）

- **监听旧 `ms_register_vault_*`／读 `vaultroommanager` 字段**：必须迁移。旧事件/组件已删除；登记改推 `ms_register_virtualroom_entity`，房间集状态改读 virtualroomset（vault 中心实体档内 `data.virtualroomset`）。[升级器改写](../../../raw/current/scripts/savefileupgrades.lua#L1496-L1530)
- **只调 `Map:IsPointInVaultRoom/IsPointInVaultLobby/IsPointInAnyVault`**：接口保留，语义随房间集（VAULT 房间集未注册/未设原点时返回 false，例如布局放置失败的世界）；要覆盖查理竞技场需另用 `IsPointInCharlieBossArena`。[包装](../../../raw/current/scripts/components/map.lua#L1565-L1586)
- **调 `CanPointHaveRain/CanPointHaveAcidRain/CanPointHaveQuaker/CanPointCauseQuaker`**：金库/大厅/竞技场内返回值已并入房间集条件（见上表）；依赖旧返回值的区域逻辑需按新条件复核（map.lua 更新报告 C-23 行为已核验条目，本次以当前源码复核一致）。
- **消费 `vault_teleport` 事件**：事件本身是通用玩家传送状态入口（SGwilson `vault_teleport` 状态，onplayerpending 进场、t=1.3 onplayerready、fxprefab），virtualroomset 投票传送与 vault_teleporter IN/OUT 直达都走它；虫洞配对传送的 [[reference/components/teleporter]] 不涉本迁移（该页已核验备注）。[状态](../../../raw/current/scripts/stategraphs/SGwilson.lua#L20181-L20225)、[序列推送](../../../raw/current/scripts/components/virtualroomset.lua#L317-L331)、[直达推送](../../../raw/current/scripts/prefabs/vault_teleporter.lua#L191-L203)
- **金库精神光环**：`vault_sanity_adjuster` 系列 prefab 仍注册且仍在 vaultroom_defs 布局中生成（key1/decon1 等房间），经 [[reference/components/sanityaura]] 生效，未随本迁移变化。[生成点](../../../raw/current/scripts/prefabs/vaultroom_defs.lua#L1154-L1157)

其余已核验的迁移消费者（供同类改造参照）：archive_chandelier 供电改 `GetPlayersInfoForVirtualRoomSetName(VAULT)`＋`ms_virtualroomset_playerschanged` 事件（[实现](../../../raw/current/scripts/prefabs/archive_chandelier.lua#L281-L288)）；shadowcreature 在 VAULT key1 房清空战利品（[实现](../../../raw/current/scripts/prefabs/shadowcreature.lua#L183-L189)）；lightsout_trial/charlie_boss/shrouden 等取玩家改 `GetPlayersInfoForVirtualRoomSetName`（[助手](../../../raw/current/scripts/componentutil.lua#L2443-L2453)）。

## 常见陷阱与限制

- **房间集未就绪时点查询恒 false**：`IsPointInVirtualRoomSet` 依赖房间集已声明且已设原点；布局放置失败或标记实体缺失的世界里，旧"tile 即金库"直觉不再成立。[装配失败分支](../../../raw/current/scripts/prefabs/cave.lua#L487-L515)
- **竞技场判定含"非大厅"条件**：ATRIUM 房间集在大厅态时 `IsPointInCharlieBossArena` 为 false；只查 `IsPointInVirtualRoomSet(ATRIUM,...)` 会把大厅误判进竞技场。[实现](../../../raw/current/scripts/components/map.lua#L1588-L1591)
- **旧房间序号与房名两套体系**：存档升级器显式注明旧 vaultroommanager 版本号不等于 virtualroomset version；模组自管旧档数据不能按新 version 直读。[注释](../../../raw/current/scripts/savefileupgrades.lua#L1427-L1430)
- **SGvault_pillar_guard 不是 virtualroom 接入点**：见上文专节；给守卫加金库相关行为应改 vault_pillar_guard prefab/brain 的 `IsPointInVaultRoom` 调用处，而非状态图。

## 验证记录

- 源码审阅：2026-10-24，Steam buildid 26148036；逐条核对 map.lua 查询、shadowhandspawner/vault_compass/SGvault_pillar_guard/vault_lobby 四条待审线索（均已转正，其中 SG 一条结论为"与 virtualroom 无关，判定在 prefab/brain"）、vault_teleport 泛化、vault_sanity_adjuster 仍注册、savefileupgrades 5.25 升级链与 cave.lua 装配链；map.lua C-23 与 teleporter/sanityaura 页的已核验条目仅复用不重抄。未检项：引擎侧 IsPointInVirtualRoomSet 原生实现、旧版快照 diff 复算、游戏运行。
- 游戏运行：未运行；旧档经 5.25 升级的实际结果与竞技场环境查询表现未实测。
- 结构检查：本批执行格式、源码锚点检查；本页无完整 Lua 示例，未做语法探测。

- 2026-10-28 build 26230496：map.lua 新增 GetVirtualRoomSetNameAtPoint 与 constants.lua 新增 VRS 哈希表使包装/常量锚点下移（本页语义未受影响：IsPointInWalledOffVRS 为新增全局见报告 R-3）；ATRIUM 路径注册回调新增 lobby 豁免（报告 P-15）。
相关：[[reference/components/virtualroommanager]]、[[reference/components/virtualroomset]]、[[reference/components/virtualroomteleporter]]、[[reference/components/teleporter]]、[[reference/components/sanityaura]]
