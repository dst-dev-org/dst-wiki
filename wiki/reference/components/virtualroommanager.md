---
title: "virtualroommanager"
created: "2026-10-24"
updated: "2026-10-28"
type: component
summary: "世界宿主组件：登记/删除虚拟房间集并维护数量上限与包围盒碰撞检查，经世界事件维护房间集实体登记表，按 tick 驱动各房间集的玩家检测、重置与空场旋转。"
aliases: ["虚拟房间管理器", "virtualroommanager"]
tags: [component, lifecycle, callback, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/virtualroommanager.lua", "components/virtualroomset.lua", "constants.lua", "prefabs/world.lua", "maputil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# virtualroommanager

## 快速判断

- 职责与适用：服务端世界组件，一个世界一份；管理 virtualroomset（单房间集）的声明/删除/遍历、房间集实体登记表与 tick 调度，自身不定义房间内容。宿主为 world prefab。[挂载](../../../raw/current/scripts/prefabs/world.lua#L598)、[构造断言](../../../raw/current/scripts/components/virtualroommanager.lua#L1-L8)
- 优先使用：声明房间集走宿主侧 `virtualroomset:DeclareVirtualRoomSetName`（内部转 `DeclareVirtualRoomSet`），不直接调本组件声明；查房间集用 `GetVirtualRoomSet(roomsetname)`，查登记实体用 `GetVirtualRoomEntities(roomsetname, context)`。[声明转发](../../../raw/current/scripts/components/virtualroomset.lua#L88-L106)、[登记查询](../../../raw/current/scripts/components/virtualroommanager.lua#L121-L128)
- 重要副作用：首个房间集声明即 `StartUpdatingComponent`，最后一个删除时停更；实体登记表不存档，靠 `ms_register/unregister_virtualroom_entity` 事件在加载后重建。[启停更新](../../../raw/current/scripts/components/virtualroommanager.lua#L130-L149)、[登记表注释](../../../raw/current/scripts/components/virtualroommanager.lua#L16-L18)
- 修改边界：仅 mastersim（构造即断言）；房间数量上限 32 为引擎常量；本组件无 OnSave/OnLoad，持久化在各个 virtualroomset 组件上。[断言](../../../raw/current/scripts/components/virtualroommanager.lua#L3-L4)、[上限](../../../raw/current/scripts/constants.lua#L3048)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `DeclareVirtualRoomSet(virtualroomset, roomsetname)` | 名称重复或数量达上限时 assert | 首个声明启动组件更新；`roomsetindex >= MAX_VIRTUALROOMSETS`（32，引擎常量）断言失败；双键存储（名称与序号）并同步 `map:DeclareVirtualRoomSet`（引擎侧）。[实现](../../../raw/current/scripts/components/virtualroommanager.lua#L130-L149)、[上限常量](../../../raw/current/scripts/constants.lua#L3048) |
| `DeleteVirtualRoomSet(roomsetname)` | 不存在时静默返回 | 先 `SetRoom(0)` 回大厅，再压缩序号列表；数量归零停更并同步 `map:DeleteVirtualRoomSet`。由 `ms_virtualroomset_removed` 事件触发。[实现](../../../raw/current/scripts/components/virtualroommanager.lua#L151-L173) |
| `GetVirtualRoomSet(roomsetname)` | -> virtualroomset 或 nil | 名称键查询；消费者遍布 vault_compass、shadowhandspawner 等。[实现](../../../raw/current/scripts/components/virtualroommanager.lua#L126-L128) |
| `GetVirtualRoomEntities(roomsetname, context)` | -> 实体列表或 nil | 按房间集名＋context（`VIRTUALROOMCONTEXT.NONE/MARKER/TELEPORTER`）查登记表；表不存档，实体自注册。[实现](../../../raw/current/scripts/components/virtualroommanager.lua#L121-L124)、[context 常量](../../../raw/current/scripts/constants.lua#L3061-L3065) |
| `OnRegisterVirtualRoomEntity(data)` | data = {inst, roomsetname, context, onlyoneprefab} | `onlyoneprefab=true` 时先移除同 prefab 旧实体（期间置 `ignoreonvirtualroomentitieschanged` 防重入）；postinit 前的变更进 pending 队列。[实现](../../../raw/current/scripts/components/virtualroommanager.lua#L62-L95)、[pending 队列](../../../raw/current/scripts/components/virtualroommanager.lua#L41-L60) |
| `OnUnregisterVirtualRoomEntity(data)` | 同上减 onlyoneprefab | `table.removearrayvalue` 后空列表逐级清键并通知房间集。[实现](../../../raw/current/scripts/components/virtualroommanager.lua#L97-L119) |
| `CheckForBoundingBoxCollisions()` | 矩形相交即 assert | 全对比较各房间集包围盒（原点 tile＋`VIRTUALROOM_BOUNDINGBOXES` 偏移），重叠时断言失败（mask 精查为 FIXME 未实现）；由 `ms_virtualroomset_originset` 触发。[实现](../../../raw/current/scripts/components/virtualroommanager.lua#L175-L200) |
| `WillBoundingBoxCollideAt(boundingbox, tx, ty)` | -> boolean | 判定给定包围盒放在 (tx,ty) 是否与任一已声明房间集重叠；原版消费者为静态布局放置过滤器（[[reference/components/worldstaticlayouts]] 装配链）。[实现](../../../raw/current/scripts/components/virtualroommanager.lua#L202-L224)、[消费者](../../../raw/current/scripts/maputil.lua#L432-L448) |
| `StartRoomVoteForRoomSetBy(player, roomsetname, virtualroomteleporter)`／`StopRoomVoteForRoomSetBy` | 转发至房间集 | 仅做名称→房间集解析后转 `StartRoomVoteBy/StopRoomVoteBy`；投票语义见 [[reference/components/virtualroomset]]。[实现](../../../raw/current/scripts/components/virtualroommanager.lua#L226-L238) |
| `ForEachVirtualRoomSet(fn, ...)` | fn 返回真值即中断遍历 | 按序号遍历；玩家重生定位用它找玩家所在房间集（`FindSafePlayerPointFrom`）。[实现](../../../raw/current/scripts/components/virtualroommanager.lua#L240-L247)、[重生消费者](../../../raw/current/scripts/mainfunctions.lua#L2105-L2121) |
| `OnPostInit()` | 置 `postinited` | 逐房间集调 `OnPostInit` 并冲刷 pending 变更队列。[实现](../../../raw/current/scripts/components/virtualroommanager.lua#L249-L256) |
| `UPDATE_TICK_TIME`／`UPDATE_ROTATE_ROOMS_COOLDOWN_TICKS_COUNT` | 1（秒）／10（tick） | `OnUpdate` 累加 dt 超过 1 秒才 `OnTick`；无人时旋转冷却倒数 10 tick。[常量](../../../raw/current/scripts/components/virtualroommanager.lua#L20-L23) |

## 生命周期

- tick 链（`OnTick`，[实现](../../../raw/current/scripts/components/virtualroommanager.lua#L279-L298)）：每 tick 先收集 `AllPlayers` 位置并逐房间集 `CheckForPlayers`（成员增减与逐玩家回调），再 `HandleResets`（无人且标记重置时回大厅清存档数据）；有玩家时 `FinishRotatingRooms` 复位空场旋转，无玩家时冷却倒数到 0 才 `RotateRoomsTick` 轮转房间。
- 事件接线（[实现](../../../raw/current/scripts/components/virtualroommanager.lua#L25-L37)）：本组件在世界实体上监听 `ms_virtualroomset_removed`（→删除）、`ms_virtualroomset_originset`（→碰撞检查）、`ms_register/unregister_virtualroom_entity`（→登记表）。这些事件由 virtualroomset 宿主与登记实体推送，发送方在 [[reference/components/virtualroomset]]。
- 存档边界：本组件无 OnSave/OnLoad；`version` 字段存在但未被保存流程读取。房间集的版本、房间名、房间内实体存档均在各 virtualroomset 组件的 OnSave/OnLoad。[房间集存档](../../../raw/current/scripts/components/virtualroomset.lua#L1209-L1250)

## 典型接入

### 把自定义实体登记进现有房间集（如 VAULT）的上下文表

前提与选择：目标模组 `modmain.lua`；需要让房间集回调（如标记校验、到达传送器刷新）感知到自定义实体时，仿照原版标记实体的注册模式推送登记事件；房间集必须已由世界声明（VAULT 由 vaultmarker 宿主声明）。[原版模式](../../../raw/current/scripts/prefabs/vault_markers.lua#L91-L96)

```lua
AddPrefabPostInit("myprefab", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    inst:ListenForEvent("onremove", function()
        GLOBAL.TheWorld:PushEvent("ms_unregister_virtualroom_entity", {
            inst = inst, roomsetname = GLOBAL.VIRTUALROOMSETS.VAULT,
            context = GLOBAL.VIRTUALROOMCONTEXT.MARKER,
        })
    end)
    GLOBAL.TheWorld:PushEvent("ms_register_virtualroom_entity", {
        inst = inst, roomsetname = GLOBAL.VIRTUALROOMSETS.VAULT,
        context = GLOBAL.VIRTUALROOMCONTEXT.MARKER,
    })
end)
```

生效与撤销：登记仅影响服务端内存表与房间集 `OnVirtualRoomEntitiesChanged` 回调，不存档——实体移除时必须成对注销（原版靠 `onremove` 监听）；`onlyoneprefab` 未传时不做同 prefab 去重。调整 context 决定落入哪张子表（`GetVirtualRoomEntities` 第二参）。[注销路径](../../../raw/current/scripts/components/virtualroommanager.lua#L97-L119)

- 调整与验证：本例仅接入既有房间集；新建房间集需宿主实体挂 [[reference/components/virtualroomset]] 并在 `VIRTUALROOM_BOUNDINGBOXES` 声明包围盒。未做运行验证。

## 常见陷阱与限制

- **登记表不存档**：`virtualroomsetentities` 明确注释不保存，靠实体加载后自推送事件重建；直接向表写字典在重载后丢失。[注释](../../../raw/current/scripts/components/virtualroommanager.lua#L16-L18)
- **房间集数量上限是引擎常量**：`MAX_VIRTUALROOMSETS = 32` 注明须与引擎实现同步，Lua 侧只断言不扩容。[常量](../../../raw/current/scripts/constants.lua#L3048)
- **声明期 assert 均为硬失败**：重名、超量、包围盒矩形相交（origin 设置时全对检查）都会直接断言崩世界；模组声明的房间集名与包围盒要避开原版三个（VAULT/LOBBYVAULT/ATRIUM）。[碰撞检查](../../../raw/current/scripts/components/virtualroommanager.lua#L175-L200)、[常量表](../../../raw/current/scripts/constants.lua#L3081-L3118)
- **manager 无持久化**：把房间集状态写到 manager 的 OnSave 预期不会生效——它没有该方法；状态归属各房间集组件。[对照](../../../raw/current/scripts/components/virtualroomset.lua#L1209-L1219)

## 验证记录

- 源码审阅：2026-10-24，Steam buildid 26148036；覆盖 virtualroommanager.lua 全文（声明/删除、登记表、碰撞检查、tick 调度）及 constants.lua 常量、world.lua 挂载、maputil.lua 放置过滤器、mainfunctions.lua 重生消费者；未审校引擎侧 map:DeclareVirtualRoomSet 等原生实现。
- 游戏运行：未运行；多房间集碰撞、空场旋转节律与存档往返未实测。
- 结构检查：本批执行格式、源码锚点与完整示例 Lua 语法检查；结果见交付报告。

- 2026-10-28 build 26230496：constants.lua VIRTUALROOMSETS 后新增哈希表块（R-4），VIRTUALROOMCONTEXT 锚点下移至 :3061-3065；常量内容未变。
相关：[[reference/components/virtualroomset]]（单房间集与房间内容）、[[reference/components/virtualroomteleporter]]（传送点）、[[reference/mechanisms/vault-virtualroom-migration]]（vault 迁移影响面）
