---
title: "walkableplatformmanager"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "可行走平台管理器：世界级 UID 注册表与每帧平台更新驱动，保证平台 UID 递增不复用。"
aliases: ["平台管理器", "平台UID管理", "walkableplatformmanager"]
tags: [component, server-client, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/walkableplatformmanager.lua", "components/walkableplatform.lua", "prefabs/world.lua", "update.lua", "gamelogic.lua", "mainfunctions.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# walkableplatformmanager

## 快速判断

- 职责与适用：挂在世界实体上（[挂接](../../../raw/current/scripts/prefabs/world.lua#L563)，所有世界），维护两套集合：`walkable_platform_uids`（UID→平台，持久化语义）与 `walkable_platforms`（当前活跃集合），并驱动每帧平台成员更新（[全文](../../../raw/current/scripts/components/walkableplatformmanager.lua#L1-L6)）。平台侧组件见 [[reference/components/walkableplatform]]。
- 优先使用：平台组件在 OnLoad/构造时 `RegisterPlatform`、移除时 `UnregisterPlatform`、进出活跃态 `AddPlatform/RemovePlatform`（[调用](../../../raw/current/scripts/components/walkableplatform.lua#L48-L74)）；管理器一般不直接操作。
- 重要副作用：`GetNewUID` 严格递增，注释明示**绝不回收 UID**——玩家存档持有 UID，平台可能在玩家离线期间被销毁（[注释](../../../raw/current/scripts/components/walkableplatformmanager.lua#L8-L13)）。
- 修改边界：主模拟端跑完整更新循环；客户端只对 ThePlayer 做 `TestForPlatform`（[分支](../../../raw/current/scripts/components/walkableplatformmanager.lua#L56-L79)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `GetNewUID()` | `-> number` | `lastuid` 自增返回；`lastuid` 随世界存档保存（[实现](../../../raw/current/scripts/components/walkableplatformmanager.lua#L10-L13)、[存读](../../../raw/current/scripts/components/walkableplatformmanager.lua#L81-L89)）。 |
| `RegisterPlatform(platform)` | `-> 无` | 无 UID 则分配；旧档 UID 大于 lastuid 时（历史无管理期）把 lastuid 跳到 `uid + 10000` 防撞；重复 UID 只警告（[实现](../../../raw/current/scripts/components/walkableplatformmanager.lua#L26-L42)）。 |
| `UnregisterPlatform(platform)` | `-> 无` | 校验 UID 与平台匹配后才清除；不匹配／无 UID 打印警告（[实现](../../../raw/current/scripts/components/walkableplatformmanager.lua#L15-L24)）。 |
| `GetPlatformWithUID(uid)` | `-> entity\|nil` | 按 UID 查平台；玩家存档还原定位用（([mainfunctions.lua:452-454](../../../raw/current/scripts/mainfunctions.lua#L452-L454)）。 |
| `PostUpdate(dt)` | `dt: number -> 无` | master：逐平台 `SetEntitiesOnPlatform` → 玩家 `TestForPlatform` → `CommitPlayersOnPlatform`；客户端仅 ThePlayer 检测（[实现](../../../raw/current/scripts/components/walkableplatformmanager.lua#L56-L79)）。 |

## 典型接入

### 每帧驱动链路（原版参照）

- 前提与选择：`PostUpdate` 由物理墙更新阶段的 `PostPhysicsWallUpdate` 每帧调用（([update.lua:149-154](../../../raw/current/scripts/update.lua#L149-L154)）；世界加载后 gamelogic 先跑一次 `PostUpdate(0)`（([gamelogic.lua:622-623](../../../raw/current/scripts/gamelogic.lua#L622-L623)）。
- 生效与撤销：平台挂载即注册（walkableplatform 组件 OnLoad），组件移除时注销（([walkableplatform.lua:48-74](../../../raw/current/scripts/components/walkableplatform.lua#L48-L74)）。

```lua
-- 世界端：查询某 UID 对应平台（等价存档还原逻辑）
local platform = TheWorld.components.walkableplatformmanager:GetPlatformWithUID(saved_uid)
if platform ~= nil then
    -- 命中已注册平台
end
```

- 调整与验证：UID 持久化只存 `lastuid` 一个数，平台↔UID 映射由平台自身存档恢复。

## 常见陷阱与限制

- **失效平台清理写错字段名（源码缺陷）**：master 循环里无效实体的清除写的是 `self.walkableplatform[k] = nil`，而真实集合叫 `self.walkable_platforms`——失效平台实际不会被剔除，只残留到下次重启（[缺陷点](../../../raw/current/scripts/components/walkableplatformmanager.lua#L58-L64)）。
- **不要复用 UID**：手动构造 `walkableplatform.uid` 可能与已存档 UID 冲突；必须经 `GetNewUID` 分配（[注释](../../../raw/current/scripts/components/walkableplatformmanager.lua#L8-L9)）。
- **PostUpdate 遍历顺序不保证**：成员集合用无序 pairs，跨平台逻辑不要依赖遍历顺序（[实现](../../../raw/current/scripts/components/walkableplatformmanager.lua#L57-L64)）。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；覆盖组件全文、walkableplatform 四个调用点、update/gamelogic/mainfunctions 三处驱动与查询、世界挂接。
- 结构检查：本批 validate_page、validate_source_links 及示例 lua 语法探测（见批级记录）。
- 游戏运行：未运行。

## 相关链接

- [[reference/components/walkableplatformplayer]]：玩家侧平台跟随：登离船登记与镜头。
