---
title: "virtualroomteleporter"
created: "2026-10-24"
updated: "2026-10-28"
type: component
summary: "虚拟房间传送点组件：保存方向/洗牌方向/房间集名等协议字段，管理投票者集合并转发房间集投票，向宿主提供 OnDepart/OnArrive/OnForceRegisterEntity 回调与传送落点覆盖。"
aliases: ["虚拟房间传送器", "virtualroomteleporter"]
tags: [component, callback, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/virtualroomteleporter.lua", "components/virtualroomset.lua", "constants.lua", "prefabs/vault_teleporter.lua", "prefabs/charliearena_teleporter.lua", "stategraphs/SGwilson.lua", "map/static_layouts/vault_lobby.lua", "map/static_layout.lua", "componentactions.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# virtualroomteleporter

## 快速判断

- 职责与适用：服务端组件，挂在传送门类实体上，代表通向某房间集（`roomsetname`）一个方向（`direction`）的传送点；投票聚合、传送序列与房间切换由 [[reference/components/virtualroomset]] 执行，本组件只存协议字段＋转发。[构造](../../../raw/current/scripts/components/virtualroomteleporter.lua#L1-L14)
- 优先使用：宿主构造期用 `SetRoomSetName`＋`SetDirection`（必要时 `SetShuffledDirection`/`SetTeleportFXPrefab`/`SetTeleportDestinationPositionOverride`）完成接线；玩家交互走 `StartRoomVote/StopRoomVote`，IN/OUT 直达传送由宿主自行推 `vault_teleport` 事件。[原版接线](../../../raw/current/scripts/prefabs/vault_teleporter.lua#L568-L576)、[交互分流](../../../raw/current/scripts/prefabs/vault_teleporter.lua#L182-L207)
- 重要副作用：构造即加 `virtualroomteleporter` 标签——componentactions 对带此标签的容器排除 JUMPIN、右键走 TELEPORT；移除组件/实体时清空全部投票者。[标签影响](../../../raw/current/scripts/componentactions.lua#L882-L885)、[清理](../../../raw/current/scripts/components/virtualroomteleporter.lua#L22-L29)
- 修改边界：仅 mastersim，且要求世界已挂 virtualroommanager；方向的网络同步由宿主 netvar 承担（vault_teleporter.dircode），组件字段本身不上网。[断言](../../../raw/current/scripts/components/virtualroomteleporter.lua#L4-L7)、[netvar](../../../raw/current/scripts/prefabs/vault_teleporter.lua#L517-L520)

## 关键接口

### 协议字段（setter/getter 对）

| 字段 | 存档行为与约束 | 条件、作用与证据 |
|---|---|---|
| `direction`／`shuffleddirection` | OnSave 存字符串名（`VIRTUALROOMDIRECTIONS_INDEX`：N/E/S/W/IN/OUT），OnLoad 经 `VIRTUALROOMDIRECTIONS` 反查 | 真实朝向与洗牌显示朝向分离：shuffleddirection 驱动实体旋转表现，direction 驱动链接选择。[存取](../../../raw/current/scripts/components/virtualroomteleporter.lua#L61-L75)、[OnSave/OnLoad](../../../raw/current/scripts/components/virtualroomteleporter.lua#L155-L176)、[旋转消费](../../../raw/current/scripts/prefabs/vault_teleporter.lua#L406-L424)、[常量](../../../raw/current/scripts/constants.lua#L3066-L3077) |
| `roomsetname` | OnSave/OnLoad 直存字符串 | `GetVirtualRoomSet` 据此查 manager；查不到时 `GetTargetRoomName` 回退 `VIRTUALROOMLOBBY`。[实现](../../../raw/current/scripts/components/virtualroomteleporter.lua#L77-L99) |
| `temporarytargetroomname` | 不存档（运行期字段） | `GetTargetRoomName` 优先返回它，越过链接图。[实现](../../../raw/current/scripts/components/virtualroomteleporter.lua#L93-L95)、[L101-L123](../../../raw/current/scripts/components/virtualroomteleporter.lua#L101-L123) |
| `tpfxprefab` | 不存档（运行期字段） | 传送序列把它传给玩家 `vault_teleport` 事件的 `fxprefab`，SG 状态进场即 SpawnPrefab；charliearena 设 "atrium_portal_fx"。[setter](../../../raw/current/scripts/components/virtualroomteleporter.lua#L85-L91)、[协议消费](../../../raw/current/scripts/components/virtualroomset.lua#L317-L331)、[SG 状态](../../../raw/current/scripts/stategraphs/SGwilson.lua#L20181-L20203)、[用例](../../../raw/current/scripts/prefabs/charliearena_teleporter.lua#L319-L327) |

### 投票、回调与落点

| 接口 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `StartRoomVote(player)`／`StopRoomVote(player)` | voters[player]=true/nil 后转发房间集 | 经 manager 的 `StartRoomVoteForRoomSetBy/StopRoomVoteForRoomSetBy` 落到对应 virtualroomset；全员同选一个传送点才触发切换（见 [[reference/components/virtualroomset]] 的 CheckRoomVotes）。[实现](../../../raw/current/scripts/components/virtualroomteleporter.lua#L145-L153) |
| `ClearAllVoters()`／`OnRemoveFromEntity()`／`OnRemoveEntity()` | 移除标签并逐一 StopRoomVote | 组件卸载与实体移除都先清投票，避免房间集计数悬挂。[实现](../../../raw/current/scripts/components/virtualroomteleporter.lua#L16-L29) |
| `SetOnDepart(fn)`／`SetOnArrive(fn)`／`SetOnForceRegisterEntity(fn)` | fn(inst[, virtualroomset]) | 调用对 `OnDepart(virtualroomset)`（传送序列开始）、`OnArrive()`（宿主自行在到达路径调用）、`OnForceRegisterEntity()`（vault_teleporter 用它把懒注册提前到 OnLoad）。[回调对](../../../raw/current/scripts/components/virtualroomteleporter.lua#L31-L59)、[OnArrive 消费](../../../raw/current/scripts/prefabs/vaultroom_defs.lua#L107-L118)、[强制注册用例](../../../raw/current/scripts/prefabs/vault_teleporter.lua#L298-L307) |
| `GetTargetRoomName()` | -> 房名 | 顺序：无房间集→大厅；有临时目标→临时目标；当前房 links[direction]→linkedroom；无 direction→大厅。原版当前无外部消费者。[实现](../../../raw/current/scripts/components/virtualroomteleporter.lua#L101-L123) |
| `GetTeleportDestinationPosition(ent)` | -> x,y,z 或 nil,nil,nil | 先试 `destinationoverridefn(inst, ent)`，否则回实体自身位置；无房间集直接三 nil。vault 版 override 按链接图/标记解析目标点。[实现](../../../raw/current/scripts/components/virtualroomteleporter.lua#L125-L143)、[vault override](../../../raw/current/scripts/prefabs/vault_teleporter.lua#L89-L130) |

## 典型接入

### 经 Tiled 静态布局属性初始化传送点字段

前提与选择：静态布局（Tiled 导出 lua）中的对象属性以 `data.` 前缀进入实体存档数据——`data.virtualroomteleporter.*` 点分键被 static_layout 解析为嵌套表，实体生成（SpawnSaveRecord）后由组件 OnLoad 按字符串方向名恢复，与 OnSave 协议一致。[属性样例](../../../raw/current/scripts/map/static_layouts/vault_lobby.lua#L331-L344)、[点分解析](../../../raw/current/scripts/map/static_layout.lua#L93-L110)

```lua
-- map/static_layouts/my_layout.lua 内的对象条目（节选自 vault_lobby 模式）
local myteleporter = {
  type = "vault_teleporter",
  shape = "rectangle", x = 288, y = 96, width = 0, height = 0,
  properties = {
    ["data.virtualroomteleporter.direction"] = "IN",
    ["data.virtualroomteleporter.shuffleddirection"] = "IN",
    ["data.virtualroomteleporter.roomsetname"] = "VAULT",
  }
}
```

生效与撤销：该通道等价于一次组件 OnLoad，方向值必须是 `VIRTUALROOMDIRECTIONS_INDEX` 中的字符串名；`tpfxprefab`、`temporarytargetroomname` 不在 OnSave 字段内，不能经此配置，需宿主代码设置。运行期改字段用对应 setter，无广播。

- 调整与验证：代码侧最小接线见 vault_teleporter 构造（SetRoomSetName＋三回调＋落点 override）；本例为布局数据说明，未运行验证。

## 常见陷阱与限制

- **组件是服务端 only，方向显示另走 netvar**：客户端看不到组件字段；vault_teleporter 用 `dircode` net_tinybyte 同步真实方向给基座表现，模组自建传送点的客户端表现需自担。[断言](../../../raw/current/scripts/components/virtualroomteleporter.lua#L4-L5)、[netvar](../../../raw/current/scripts/prefabs/vault_teleporter.lua#L517-L520)
- **标签副作用**：加组件即在实体上加 `virtualroomteleporter` 标签（模板建议宿主 pristine 期自带），该标签改变 componentactions 的动作选择（不 JUMPIN、右键 TELEPORT）；卸组件移除标签并清投票。[标签](../../../raw/current/scripts/components/virtualroomteleporter.lua#L12-L13)、[动作分流](../../../raw/current/scripts/componentactions.lua#L882-L885)
- **OnSave 只存三字段**：direction/shuffleddirection/roomsetname；`voters`、`destinationoverridefn`、`tpfxprefab`、`temporarytargetroomname` 重载后为空，宿主需在构造/OnLoad 重建。[OnSave](../../../raw/current/scripts/components/virtualroomteleporter.lua#L155-L162)
- **OnArrive 不在传送序列内自动调用**：序列只调 OnDepart；OnArrive 由宿主按到达位置自行判定调用（vault 用 DoOnArriveTeleporters_HACK 按落点匹配传送器）。[序列侧](../../../raw/current/scripts/components/virtualroomset.lua#L314-L316)、[HACK](../../../raw/current/scripts/prefabs/vaultroom_defs.lua#L107-L118)

## 验证记录

- 源码审阅：2026-10-24，Steam buildid 26148036；覆盖 virtualroomteleporter.lua 全文与 vault_teleporter/charliearena_teleporter 接线、SGwilson vault_teleport 状态协议、static_layout 属性管线、componentactions 标签分流；`GetTargetRoomName` 经全库 grep 确认原版无消费者。
- 游戏运行：未运行；多玩家投票、IN/OUT 直达传送与客户端表现未实测。
- 结构检查：本批执行格式、源码锚点检查；页面内 Lua 片段为布局数据节选（包装为局部表后 Lua 5.1 语法解析通过，主代理验收补验）。

相关：[[reference/components/virtualroomset]]、[[reference/components/virtualroommanager]]、[[reference/components/teleporter]]（虫洞配对传送，另一套体系）、[[reference/mechanisms/vault-virtualroom-migration]]
