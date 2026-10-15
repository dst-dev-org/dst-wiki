---
title: "touchstonetracker"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "重生石使用记录组件：按会话（分片）记录玩家用过的重生石 ID，供 CanUseTouchStone 判定并随角色交换迁移。"
aliases: ["重生石记录", "重生石追踪", "touchstonetracker"]
tags: [component, data, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/touchstonetracker.lua", "prefabs/player_common.lua", "prefabs/player_classified.lua", "prefabs/resurrectionstone.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# touchstonetracker

## 快速判断

- 职责与适用：挂在玩家实体上（[挂接](../../../raw/current/scripts/prefabs/player_common.lua#L2832)），监听 `usedtouchstone` 事件，把当前会话用过的重生石 ID 记入 `used`，并把其他会话的旧记录保存在 `used_foreign`（[构造](../../../raw/current/scripts/components/touchstonetracker.lua#L18-L23)）。
- 优先使用：判定走 `IsUsed(touchstone)`；玩家侧统一入口是 `inst.CanUseTouchStone`（[指派](../../../raw/current/scripts/prefabs/player_common.lua#L2275)）。
- 重要副作用：新使用即同步到 `player_classified`（`net_smallbytearray "touchstonetracker.used"`，[netvar](../../../raw/current/scripts/prefabs/player_classified.lua#L1604)、[写入](../../../raw/current/scripts/prefabs/player_classified.lua#L225)），客户端据此做本地判定。
- 修改边界：组件移除时清空 classified 记录并注销监听（[清理](../../../raw/current/scripts/components/touchstonetracker.lua#L25-L28)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `IsUsed(touchstone)` | `-> bool` | 按重生石 ID 查当前会话记录（[实现](../../../raw/current/scripts/components/touchstonetracker.lua#L38-L40)）。 |
| `used` / `used_foreign` | `table`（ID->true）/ `table`（会话ID->ID数组） | `used` 服务判定，`used_foreign` 仅随存档保留（[字段](../../../raw/current/scripts/components/touchstonetracker.lua#L20-L21)）。 |
| `OnSave` / `OnLoad` | 存读钩子 | 以 `TheWorld.meta.session_identifier` 为键分组保存；加载时当前会话走正式记录路径，其余进 `used_foreign`（[实现](../../../raw/current/scripts/components/touchstonetracker.lua#L42-L80)）。 |
| `TransferComponent(newinst)` | `newinst: entity -> 无` | 把 `used` 逐条重放到新实体并把 `used_foreign` 直接移交；由角色交换的组件搬运循环统一调用（[实现](../../../raw/current/scripts/components/touchstonetracker.lua#L82-L88)、[循环](../../../raw/current/scripts/prefabs/player_common.lua#L2330-L2333)）。 |

## 典型接入

### 使用判定链路（原版参照）

- 前提与选择：重生石使用后向玩家推 `usedtouchstone`（data 为重生石实体，[resurrectionstone.lua:105](../../../raw/current/scripts/prefabs/resurrectionstone.lua#L105），组件转成 ID 记账（[事件处理](../../../raw/current/scripts/components/touchstonetracker.lua#L14-L16)）。
- 生效与撤销：服务端 `CanUseTouchStone` 查组件，客户端查 classified 数组（[双端分支](../../../raw/current/scripts/prefabs/player_common.lua#L194-L198)）。

```lua
-- 服务端：判定玩家能否使用某重生石
local function CanUseTouchStone(inst, touchstone)
    if inst.components.touchstonetracker ~= nil then
        return not inst.components.touchstonetracker:IsUsed(touchstone)
    end
    -- 客户端回退：查 player_classified.touchstonetrackerused 数组
    return touchstone.GetTouchStoneID ~= nil
        and not table.contains(inst.player_classified.touchstonetrackerused:value(), touchstone:GetTouchStoneID())
end
```

- 调整与验证：ID 来自重生石的 `GetTouchStoneID`（ID<=0 不记账，[守卫](../../../raw/current/scripts/components/touchstonetracker.lua#L2)）。

## 常见陷阱与限制

- **IsUsed 不看 used_foreign**：其他会话（旧存档期）的记录不影响判定，只随存档往返搬运；跨会话"永久已用"需要自己另外处理（[字段语义](../../../raw/current/scripts/components/touchstonetracker.lua#L20-L21)）。
- **组件注释明示未网络化**：玩家侧注释"不想把 touchstonetracker 做成网络组件"，客户端读的是 classified 快照而非组件本身（[player_common.lua:2275](../../../raw/current/scripts/prefabs/player_common.lua#L2275）。
- **分类挂载**：所有玩家默认挂载（[player_common.lua:2832](../../../raw/current/scripts/prefabs/player_common.lua#L2832），无需游戏模式开关。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；覆盖组件全文、玩家挂接与 CanUseTouchStone 双端分支、classified netvar、重生石事件推送、角色交换搬运循环。
- 结构检查：本批 validate_page、validate_source_links 及示例 lua 语法探测（见批级记录）。
- 游戏运行：未运行。
