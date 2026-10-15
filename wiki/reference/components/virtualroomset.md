---
title: "virtualroomset"
created: "2026-10-24"
updated: "2026-10-28"
type: component
size_class: complex
summary: "单个虚拟房间集实例：声明房间与链接图、维护当前房间与大厅切换（卸载/重装载实体与地砖批）、投票触发的传送序列、方向缓存与 Grue 免疫区，宿主经声明式 defs 提供房间布局数据。"
aliases: ["虚拟房间集", "virtualroomset"]
tags: [component, lifecycle, callback, worldgen]
dst_build: "steam-buildid:26230496"
sources: ["components/virtualroomset.lua", "components/virtualroommanager.lua", "components/virtualroomteleporter.lua", "constants.lua", "prefabs/vault_markers.lua", "prefabs/vaultroom_defs.lua", "prefabs/vault_teleporter.lua", "components/grue.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# virtualroomset

## 快速判断

- 职责与适用：服务端组件，一个实体一份，代表一个虚拟房间集（原版 VAULT/LOBBYVAULT/ATRIUM）；房间图声明、当前房间切换、玩家成员维护、投票与传送序列都在这里，世界侧调度见 [[reference/components/virtualroommanager]]。构造即要求世界已挂 manager。[构造](../../../raw/current/scripts/components/virtualroomset.lua#L16-L25)
- 整体保留范围（complex 档依据）：声明链（房间集名/房间/链接/免疫区）＋房间切换生命周期（HideRoom/ShowRoom/UnloadRoom/LoadRoom 的实体分类与存档）＋投票→传送序列（vault_teleport 协议）＋tick 协作（玩家检测/重置/旋转）四条主线需整体成文；地砖批与原点/包围盒仅记接口约束，defs 房间布局数据只记组织方式。
- 优先使用：声明顺序固定——先 `DeclareVirtualRoomSetName`（含包围盒与 manager 登记），再 `DeclareVirtualRoom`＋`LinkVirtualRooms*`，最后 `SetRoomDefinitions(defs)`（原版注明 Do last）。[声明](../../../raw/current/scripts/components/virtualroomset.lua#L88-L106)、[原版顺序](../../../raw/current/scripts/prefabs/vault_markers.lua#L288-L302)
- 重要副作用：`SetRoom` 切房会移除并按存档记录重建整间实体、批量重写地砖；`UnloadRoom` 在 POPULATING 保护下移除实体，靠 `staysthroughvirtualrooms` 等标签豁免。[切房](../../../raw/current/scripts/components/virtualroomset.lua#L700-L718)、[分类](../../../raw/current/scripts/components/virtualroomset.lua#L982-L1014)
- 修改边界：仅 mastersim；大厅（`currentroomindex==0`，房名 `VIRTUALROOMLOBBY`）是保留态不是房间；玩家成员判定依赖引擎侧 `map:IsPointInVirtualRoomSet`。[大厅常量](../../../raw/current/scripts/constants.lua#L3078)、[成员判定](../../../raw/current/scripts/components/virtualroomset.lua#L855-L884)

## 关键接口

### 声明链

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `DeclareVirtualRoomSetName(roomsetname)` | 必须已在 `VIRTUALROOM_BOUNDINGBOXES` 声明 | 取包围盒（min/max/mask，mask 尺寸须等于矩形面积），经 manager 登记拿 `roomsetindex`，同步引擎侧 boundingbox/mask；mask 可为 nil（实心矩形）。[实现](../../../raw/current/scripts/components/virtualroomset.lua#L88-L106)、[常量表](../../../raw/current/scripts/constants.lua#L3081-L3118) |
| `DeclareVirtualRoom(roomname)` | 保留名与重名均 assert | 追加 `VirtualRoom{roomname, roomindex, links}`，名称与序号双键存储；`VIRTUALROOMLOBBY` 为保留名。[实现](../../../raw/current/scripts/components/virtualroomset.lua#L646-L663) |
| `LinkVirtualRooms(from, fromdir, to, todir)` | link = {linkedroom, linkeddirection} | 单向写 from 房链接；`LinkVirtualRoomsBroken` 附 `broken=true`；`MakeLinkRigid(direction)` 标记该链接不参与洗牌（宿主 ShuffleRooms 读取）。[实现](../../../raw/current/scripts/components/virtualroomset.lua#L108-L136)、[洗牌消费](../../../raw/current/scripts/prefabs/vault_markers.lua#L160-L176) |
| `MakeRoomGrueImmuneArea(roomname)` | rooms[roomname].grueimmunearea=true | 玩家加入该房时向玩家推 `grueimmunearea` 事件 `{name, isimmune}`，grue 组件监听后加黑暗免死。[声明](../../../raw/current/scripts/components/virtualroomset.lua#L138-L140)、[推送](../../../raw/current/scripts/components/virtualroomset.lua#L779-L801)、[监听](../../../raw/current/scripts/components/grue.lua#L50) |
| `SetRoomDefinitions(defs)` | defs.DeleteLayout + defs.InitializeLayout 必须存在 | 传完立即执行两个强制项；defs 组织见下文"数据面"。[实现](../../../raw/current/scripts/components/virtualroomset.lua#L370-L374) |

### 查询与方向缓存

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `GetCurrentRoom()`／`GetCurrentRoomName()`／`IsCurrentRoomLobby()` | index 0 → nil／`VIRTUALROOMLOBBY`／true | 大厅不是 `rooms` 成员；`SetCurrentRoomIndex` 同步引擎侧 `SetVirtualRoomSetIsInLobby`。[实现](../../../raw/current/scripts/components/virtualroomset.lua#L142-L147)、[L206-L216](../../../raw/current/scripts/components/virtualroomset.lua#L206-L216)、[L695-L698](../../../raw/current/scripts/components/virtualroomset.lua#L695-L698) |
| `GetClosestDirectionFromRoomToRoom(from, to)` | -> 方向或 nil（同房 nil） | 惰性建 BFS 缓存 `closestdirectioncache[from][to]={dist,direction}`；`defs.internal.IsOtherRoomLinkBroken` 存在时断链不入缓存；原版消费者 vault_compass。[实现](../../../raw/current/scripts/components/virtualroomset.lua#L149-L204)、[消费者](../../../raw/current/scripts/prefabs/vault_compass.lua#L26-L28) |
| `InvalidateClosestDirectionCache()` | 清空缓存 | 修好断链后必须调用，否则旧方向继续生效（vault_teleporter 修理路径即如此）。[实现](../../../raw/current/scripts/components/virtualroomset.lua#L149-L151)、[消费者](../../../raw/current/scripts/prefabs/vault_teleporter.lua#L336-L344) |
| `GetPlayersInfo()` | -> players 表, numberplayers | 供投票与传送判定；通用助手 `GetPlayersInfoForVirtualRoomSetName` 也走这里。[实现](../../../raw/current/scripts/components/virtualroomset.lua#L914-L916)、[助手](../../../raw/current/scripts/componentutil.lua#L2443-L2453) |
| `IsEntInVirtualRoom(ent)`／`GetRoomLinks(roomname)`／`GetVirtualRoomEntities(context)` | -> bool／links 表／实体列表 | 前两者直读；后者转发 manager 登记表（context 见 [[reference/components/virtualroommanager]]）。[实现](../../../raw/current/scripts/components/virtualroomset.lua#L956-L959)、[L366-L368](../../../raw/current/scripts/components/virtualroomset.lua#L366-L368)、[L665-L667](../../../raw/current/scripts/components/virtualroomset.lua#L665-L667) |

### 投票与传送序列

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `StartRoomVoteBy(player, teleporter)`／`StopRoomVoteBy` | voters[player]=teleporter | 每次变更后 `CheckRoomVotes`。[实现](../../../raw/current/scripts/components/virtualroomset.lua#L348-L364) |
| `CheckRoomVotes()` | 全员同意且目标一致才触发 | `numbervoters >= numberplayers` 且所有 voter 选同一 teleporter 时，读其 `GetDirection/GetTeleportDestinationPosition/GetTeleportFXPrefab` 组装 `teleportingentsdata` 进入传送序列（[[reference/components/virtualroomteleporter]]）。[实现](../../../raw/current/scripts/components/virtualroomset.lua#L218-L253) |
| `TryStartTeleportSequence(teleportingentsdata)` | 有人时 true | data 含 `virtualroomteleporter/targetroomname/x/z/fxprefab`（可选 `state/onteleportcb/radius`）；逐玩家 `PushEventImmediate("vault_teleport", {onplayerpending, onplayerready, state, fxprefab})`，全部 ready（含 onremove 兜底）后 `SetRoom(target)`。[实现](../../../raw/current/scripts/components/virtualroomset.lua#L277-L334) |
| `TryTeleportSequenceImmediatelyWithNoPlayers(data)` | 无人且无进行序列时 true | 直接 OnDepart→onteleportcb→SetRoom，不推玩家事件；vault 无人回 mask1 房用它。[实现](../../../raw/current/scripts/components/virtualroomset.lua#L255-L275)、[消费者](../../../raw/current/scripts/prefabs/vault_markers.lua#L112-L129) |
| `CancelPendingTeleport()` | 清 pendingtps 与 onremove 监听 | `HideRoom` 先调它防悬挂。[实现](../../../raw/current/scripts/components/virtualroomset.lua#L336-L346)、[L669-L679](../../../raw/current/scripts/components/virtualroomset.lua#L669-L679) |
| `TeleportEntities(ents, x, y, z, r)` | 环形散布后 -> true | 随机角偏移均分圆周（r 可为函数），Physics:Teleport 或 Transform:SetPosition，末尾 `manager:UpdatePlayerPositions()` 立即刷新成员。[实现](../../../raw/current/scripts/components/virtualroomset.lua#L720-L747) |

### 原点、包围盒与地砖批

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SetOrigin(x,y,z)`／`SetOriginInTiles(tx,ty)` | 已有原点时先回大厅 | 原点入引擎侧并推 `ms_virtualroomset_originset`（触发 manager 碰撞检查）；tx 为 nil 清原点。[实现](../../../raw/current/scripts/components/virtualroomset.lua#L563-L588) |
| `RecalculateSaveRadius()`／`GetSaveRadius()` | 半径 = 角点距离×TILE_SCALE＋半格×√2＋0.1 | `UnloadRoom` 的 FindEntities 半径；无包围盒则 nil（不卸载）。[实现](../../../raw/current/scripts/components/virtualroomset.lua#L496-L524) |
| `NewFloorTileBatch()`／`SetFloorTileInBatch(dtx,dty,tile)`／`ApplyFloorTileBatch()` | batch 以 `VIRTUALROOM_DONOTCARE_TILE`（"x"）占位 | 出界或写 DONOTCARE 格均 assert；Apply 期间关 `InstantPhysicsRebuild` 一次重建并触发 `TempTile_HandleTileChange`；大厅回退 `originalfloortiles`。[实现](../../../raw/current/scripts/components/virtualroomset.lua#L422-L475)、[占位常量](../../../raw/current/scripts/constants.lua#L3080) |
| `ScanOriginalFloorTiles()` | 离开大厅前快照 | 按 mask 记录原版地砖供回大厅恢复。[实现](../../../raw/current/scripts/components/virtualroomset.lua#L376-L396) |

### 配置回调与禁制

- 全部经 `SetXxx` 写入 `self.config`：`OnHideRoom/OnShowRoom/OnPlayerAdded/OnPlayerRemoved/OnPlayersChanged/OnPostInit/OnVirtualRoomEntitiesChanged/OnTeleportedEntity/OnTeleportedAllEntities/OnPlayerTick/OnReset/FindSafePlayerPointFrom`，统一签名 `(inst, virtualroomset, ...)`（OnShowRoom 另带当前房名/传送数据，FindSafePlayerPointFrom 需返回坐标）。[setter 区](../../../raw/current/scripts/components/virtualroomset.lua#L606-L632)、[OnReset](../../../raw/current/scripts/components/virtualroomset.lua#L910-L912)
- 开关项：`SetDoNotRotateRooms(true)`（动态地形型房间集不轮转）、`SetKeepInventoryItemsthroughReset(true)`（重置时保留房间内 inventoryitem）、`SetTeleportingIntoLobbyProhibited/Out(true)`（写引擎侧，寻路过滤器读取）。[开关区](../../../raw/current/scripts/components/virtualroomset.lua#L918-L932)

## 关键流程

| 入口／条件 | 当前步骤与结果 | 后续／中断／清理及证据 |
|---|---|---|
| `SetRoom(name|index|nil)`（L700-L718） | 房号不变直接返回；离开大厅前 `ScanOriginalFloorTiles` → `HideRoom`（取消挂起传送、UnloadRoom 存房间档、OnHideRoom 回调）→ `SetCurrentRoomIndex` → `ShowRoom`；进大厅后清 `originalfloortiles` | 传送序列在 ShowRoom 后按 `teleportingentsdata` 调 `TeleportEntities`；[Hide/Show](../../../raw/current/scripts/components/virtualroomset.lua#L669-L693) |
| `UnloadRoom(save)`（L1019-L1141） | 按存档半径 FindEntities，`_GetEntUnloadAction` 三分类：`_SKIP`（有父级/挂 staysthroughvirtualrooms 标签/所有主链在房外）、`_KEEP`（玩家、irreplaceable、followsthroughvirtualrooms、迁移宠物、重置期保留物——除非 `forcedtosavethroughvirtualrooms`）、其余 `_SAVE` | `_SAVE` 实体先被容器强制掉落 irreplaceable/宠物，POPULATING 下 GetSaveRecord（剥 prefab 字段）后移除；CLASSIFIED 仅 `forcedtosavethroughvirtualrooms` 者随房保存；返回剩余实体表供传送。[分类](../../../raw/current/scripts/components/virtualroomset.lua#L982-L1014) |
| `ShowRoom`→`LoadRoom(roomname, roomsavedata)`（L1163-L1207） | 无档走 `LayoutNewRoom`（defs.layouts 的 ApplyFloorTiles/CreateRoomEntities，POPULATING 包裹）；有档 SpawnSaveRecord＋LoadPostPass 重建，再按 `world_time` 差值 LongUpdate 补算 | 房间档是房间集存档的一部分（`roomsavedata[roomname]`），重置时整表清空。[LayoutNewRoom](../../../raw/current/scripts/components/virtualroomset.lua#L1143-L1161) |
| manager tick → `CheckForPlayers`（L855-L884） | 逐玩家 `map:IsPointInVirtualRoomSet` 判成员：新进→`OnPlayerAdded`（drownable 传送点、grueimmunearea 事件、config 回调、世界事件 `ms_virtualroomset_playeradded`）；在场→`OnPlayerTick`；离开反向 | 成员变化收敛到 `OnPlayersChanged`（回调＋`ms_virtualroomset_playerschanged`）；[Added/Removed](../../../raw/current/scripts/components/virtualroomset.lua#L779-L820) |
| 无人 tick → `HandleResets`→`TryToReset`（L886-L908） | `resetting` 标记时回大厅、清全部 `roomsavedata`、OnReset 回调 | 标记来源：`FlagForReset`、OnLoad 版本不符、未知房名。[L886-L908](../../../raw/current/scripts/components/virtualroomset.lua#L886-L908) |
| 空场旋转 `RotateRoomsTick`（L934-L954） | manager 冷却到点且未禁旋转时房号 +1 循环（含 0 大厅）；有玩家即刻 `FinishRotatingRooms` 复位 | 复位基准 `roomindexbeforerotate` 随档保存；[L934-L954](../../../raw/current/scripts/components/virtualroomset.lua#L934-L954) |
| 移除宿主 `OnRemoveEntity`（L70-L75） | 推 `ms_virtualroomset_removed` | manager 收到后 `DeleteVirtualRoomSet`（回大厅并摘除）；[manager 侧](../../../raw/current/scripts/components/virtualroommanager.lua#L151-L173) |

## 数据面：defs 组织与扩展点

源码 1252 行中约七成是宿主提供的房间/布局数据，组件只消费其结构：`defs = {internal, layouts, defaultfloortiles}`——`layouts[roomname]` 为 `{ApplyFloorTiles(inst, vrs), CreateRoomEntities(inst, vrs)}`（模板注释见原文件头部），`internal` 放宿主自定义谓词（原版 vaultroom_defs 的 `IsLinkBroken/IsOtherRoomLinkBroken` 会被方向缓存与通电判定读取），`defaultfloortiles` 按包围盒 mask 展开（DONOTCARE 或默认地砖）。原版 vault 共 19 个布局房间，重复走廊用 `shallowcopy` 复用。[defs 模板](../../../raw/current/scripts/prefabs/vaultroom_defs.lua#L19-L31)、[默认地砖](../../../raw/current/scripts/prefabs/vaultroom_defs.lua#L35-L48)、[internal 谓词](../../../raw/current/scripts/prefabs/vaultroom_defs.lua#L69-L105)。扩展点即自定义 defs 表 + `SetRoomDefinitions`，大厅固定空布局（EMPTYLAYOUT＋originalfloortiles）。[大厅分支](../../../raw/current/scripts/components/virtualroomset.lua#L1143-L1149)

## 典型接入

### 用自定义 defs 建一个不旋转的两房间集

前提与选择：宿主实体 prefab（master 分支）挂本组件；新房间集名须先在 `VIRTUALROOMSETS`（模组侧可扩展同名常量表）与 `VIRTUALROOM_BOUNDINGBOXES` 声明，否则 DeclareVirtualRoomSetName 断言。仿 LOBBYVAULT 静态区域用法（`SetDoNotRotateRooms`＋空房间集）。[原版大厅宿主](../../../raw/current/scripts/prefabs/vault_markers.lua#L393-L397)

```lua
GLOBAL.VIRTUALROOMSETS.MYSET = "MYSET"
GLOBAL.VIRTUALROOM_BOUNDINGBOXES[GLOBAL.VIRTUALROOMSETS.MYSET] = {
    minx = -2, miny = -2, maxx = 2, maxy = 2, mask = nil,
}
local mydefs = { internal = {}, layouts = {} }
mydefs.DeleteLayout = function(vrs) end
mydefs.InitializeLayout = function(vrs)
    vrs:DeclareVirtualRoom("room_a")
    vrs:DeclareVirtualRoom("room_b")
    vrs:LinkVirtualRooms("room_a", GLOBAL.VIRTUALROOMDIRECTIONS.N, "room_b", GLOBAL.VIRTUALROOMDIRECTIONS.S)
end
AddPrefabPostInit("myroomsethost", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    local vrs = inst:AddComponent("virtualroomset")
    vrs:DeclareVirtualRoomSetName(GLOBAL.VIRTUALROOMSETS.MYSET)
    vrs:SetDoNotRotateRooms(true)
    vrs:SetRoomDefinitions(mydefs)
end)
```

生效与撤销：房间集登记进世界 manager 后参与 tick 与碰撞检查；移除宿主实体即触发 `ms_virtualroomset_removed` 摘除。defs 两个强制项缺一在 SetRoomDefinitions 即报错。`layouts` 留空表示所有房间仅切地砖不生成实体（`LayoutNewRoom` 只认 defs.layouts 中已声明的房名，缺失即 assert）。

- 调整与验证：mask 为 nil 即整矩形；自定义 `internal.IsOtherRoomLinkBroken` 可让断链影响方向缓存。本例为接口最小组装，未运行验证。

## 常见陷阱与限制

- **声明顺序与强断言**：房集名无包围盒、mask 尺寸不符、房名重名/保留名、SetFloorTileInBatch 出界或写 DONOTCARE 格、LoadRoom/LayoutNewRoom 未知房名都是 assert 崩溃，不是可捕获错误。[声明](../../../raw/current/scripts/components/virtualroomset.lua#L88-L106)、[批写](../../../raw/current/scripts/components/virtualroomset.lua#L431-L444)
- **切房会移除实体**：无 `staysthroughvirtualrooms`/`followsthroughvirtualrooms` 标签、非 irreplaceable、非玩家的房内实体在每次 SetRoom 被存档移除再重建；CLASSIFIED 默认不随房保存（`forcedtosavethroughvirtualrooms` 例外）。FX 类 prefab 应挂 `staysthroughvirtualrooms`。[分类](../../../raw/current/scripts/components/virtualroomset.lua#L982-L1014)、[原版用法](../../../raw/current/scripts/prefabs/vault_compass.lua#L201)
- **version 不匹配即整集重置**：OnLoad 时组件内 `self.version`（由 `SetVersion` 或存档数据设置）与存档版本不同直接 `FlagForReset`，下次无人时清空全部房间档；升级 defs 后旧档房间内容不保留。[实现](../../../raw/current/scripts/components/virtualroomset.lua#L489-L494)、[L1221-L1233](../../../raw/current/scripts/components/virtualroomset.lua#L1221-L1233)
- **传送事件名带 vault 前缀但属通用体系**：`vault_teleport` 是玩家 SG 通用传送状态入口（见 [[reference/components/virtualroomteleporter]]），非金库专用；监听该事件的自定义传送也会走 onplayerpending/onplayerready 协议。[推送](../../../raw/current/scripts/components/virtualroomset.lua#L317-L331)
- **方向缓存需手动失效**：修改链接或修复断链后不调 `InvalidateClosestDirectionCache`，GetClosestDirection* 继续返回旧方向。[缓存](../../../raw/current/scripts/components/virtualroomset.lua#L149-L204)
- **OnSave 字段集有限**：只存 `version/resetting/currentroomname/roomsavedata/originalfloortiles/roomindexbeforerotate`；`customdata` 由宿主经存档流恢复（vault_markers 依赖 vaultmarker 实体档里的 data.virtualroomset.customdata 通道），`scratchpad` 不存档。[OnSave](../../../raw/current/scripts/components/virtualroomset.lua#L1209-L1219)、[customdata/scratchpad](../../../raw/current/scripts/components/virtualroomset.lua#L35-L36)

## 验证记录

- 源码审阅：2026-10-24，Steam buildid 26148036；覆盖 virtualroomset.lua 全文（声明链、切换生命周期、实体卸载分类、投票/传送序列、tick 协作、存档），并核对 vault_markers/vault_teleporter/vault_compass/grue/componentutil 消费点与 constants 常量；vaultroom_defs 仅核组织结构与谓词，19 个房间的具体布局数据未逐项审校；引擎侧 map 系列原生函数未读实现。
- 游戏运行：未运行；切房实体往返、多玩家投票并发、旋转节律与存档升级路径未实测。
- 结构检查：本批执行格式、源码锚点与完整示例 Lua 语法检查；结果见交付报告。

相关：[[reference/components/virtualroommanager]]、[[reference/components/virtualroomteleporter]]、[[reference/mechanisms/vault-virtualroom-migration]]
