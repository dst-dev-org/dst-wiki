---
title: "moonstorms"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "月暴节点登记与查询：维护地图拓扑节点级月暴范围，经 net 数组同步到各端，供地图、生成与玩家减速等系统共用。"
aliases: ["月暴节点", "月暴范围查询", "moonstorms"]
tags: [component, network, event]
dst_build: "steam-buildid:26230496"
size_class: standard
sources: ["components/moonstorms.lua", "prefabs/forest_network.lua", "components/moonstormmanager.lua", "components/map.lua", "components/birdspawner.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# moonstorms

## 快速判断

- 职责与适用：挂在 `TheWorld.net` 上的节点登记器（网络世界 prefab 构造，prefabs/forest_network.lua:18），记录哪些地图拓扑节点处于月暴并向所有端同步；服务器侧的月暴推进由 moonstormmanager 组件驱动。
- 优先使用：判断某点是否在月暴→`IsXZInMoonstorm(x, z)`（服务器与客户端都经 `TheWorld.net.components.moonstorms` 读取，components/map.lua:1066-1067）；按实体查询→`GetMoonstormLevel(ent)`。
- 重要副作用：节点变化时 set net 变量 `moonstorm.moonstorm_nodes`，并推 `ms_stormchanged`、`ms_moonstormwindowover` 事件（components/moonstorms.lua:125-128、:132）。
- 修改边界：节点表与地图标记（`moonstormmarker_big`）随调用维护，`ClearMoonstormNodes` 一并清空（:135-142）；组件无自持任务。

## 关键接口

| 接口/字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `IsXZInMoonstorm` / `IsPointInMoonstorm` | `((x, z) \| (pt)) -> boolean` | 按坐标 `Map:GetNodeIdAtPoint` 查活动节点表。:88-100 |
| `IsInMoonstorm` | `(ent) -> boolean` | 需实体带 `areaaware` 且当前节点在表中。:82-86 |
| `GetMoonstormLevel` | `(ent) -> number 0..1` | 月暴内时按拓扑边到点距离/`TUNING.SANDSTORM_FULLY_ENTERED_DEPTH` 计算，否则 0。:52-71、:102-107 |
| `AddMoonstormNodes` | `(node_indices, firstnode) -> nil` | 标记节点、在每个节点中心生成 `moonstormmarker_big`、同步 net 变量并推事件。:109-128 |
| `StopMoonstorm` | `(is_relocating) -> nil` | 清节点；`ms_stormchanged.setting` 取决于是否迁移中。:130-133 |
| `ClearMoonstormNodes` | `() -> nil` | 清空节点表、net 变量与全部地图标记。:135-142 |
| `GetMoonstormNodes` / `GetMoonstormCenter` | `() -> table` / `() -> Point\|nil` | 节点表（node_index→true）；有节点时返回质心。:144-159 |
| `convertlist` / `_moonstorm_nodes` | `(data) -> number[]` / `net_ushortarray` | true 表转索引数组；net 变量 dirty 事件转发为 `moonstorm_nodes_dirty_relay`。:21-25、:30-39 |

## 典型接入

### 查询坐标是否处于月暴（任一端）

- 前提与选择：组件在 `TheWorld.net` 上，客户端可直接读；节点表是服务器数据经 net 数组同步的快照。
- 生效与撤销：无月暴活动时组件仍存在但节点表为空。

```lua
local function IsMoonstormAt(x, z)
    local moonstorms = TheWorld.net ~= nil and TheWorld.net.components.moonstorms or nil
    return moonstorms ~= nil and moonstorms:IsXZInMoonstorm(x, z) or false
end
```

- 调整与验证：服务器开/关月暴走 `ms_startthemoonstorms`/`ms_stopthemoonstorms` 事件（moonstormmanager.lua:265-266 监听，moon_altar_link.lua:28 为原版触发点），不要直接改节点表。

## 常见陷阱与限制

- `CalcMoonstormLevel` 依赖 `areaaware` 的 `lastpt`/`current_area`，实体无该组件直接得 0（:74-80）。
- `AddMoonstormNodes` 传单个数字而非表时，内部 `node_indice` 拼写错误会产生全局变量并随后的 ipairs 报错（:110-114）；原版调用始终传表（moonstormmanager.lua:381）。
- 读侧消费者：birdspawner 抑制月暴内鸟生成（components/birdspawner.lua:470）、moonstormwatcher 玩家减速入口（components/moonstormwatcher.lua:9-11）、moonstormlightningmanager 直读 net 变量（components/moonstormlightningmanager.lua:10）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，通读组件全部 165 行；核验挂载（forest_network.lua:18）、服务器驱动（moonstormmanager.lua:380-381、:265-266）与读侧（map.lua:1066-1067、birdspawner.lua:470）。
- 游戏运行：未运行。

## 相关链接

- 玩家侧响应：[[reference/components/moonstormwatcher]]；闪电：[[reference/components/moonstormlightningmanager]]；静电捕捉：[[reference/components/moonstormstaticcatcher]]、[[reference/components/moonstormstaticcapturable]]；区域感知：[[reference/components/areaaware]]。
