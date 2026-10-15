---
title: "areaaware"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "区域感知组件：跟踪实体所在的地图拓扑节点（区域 id/类型/tag），区域变化时广播 changearea，并可监视特定地块类型进出事件。"
aliases: ["区域感知", "changearea", "areaaware"]
tags: [component, worldgen]
dst_build: "steam-buildid:26230496"
sources: ["components/areaaware.lua", "prefabs/player_common.lua", "prefabs/monkey.lua", "prefabs/powdermonkey.lua", "prefabs/stalker.lua", "prefabs/wx78_possessedbody.lua", "prefabs/wurt.lua", "components/sandstorms.lua", "components/moonstorms.lua", "components/dynamicmusic.lua", "components/playerspawner.lua", "components/nightmareclock.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# areaaware

## 快速判断

- 职责与适用：把"我在哪个拓扑区域"缓存成本组件状态：`current_area`（节点索引）与 `current_area_data`（id/type/center/poly/tags），供 AI 与环境系统查询；位置由每帧距离阈值驱动检测。[构造](../../../raw/current/scripts/components/areaaware.lua#L1-L14)
- 优先使用：查询 `GetCurrentArea()`／`CurrentlyInTag(tag)`；强制刷新 `UpdatePosition(x, y, z)`；调检测距离 `SetUpdateDist(dist)`；地块监视 `StartWatchingTile(tile_id)`。[查询](../../../raw/current/scripts/components/areaaware.lua#L85-L91)
- 重要副作用：区域变化推 `changearea`（大量系统监听该事件名）；被监视地块的进出推 `on_<TILE>_tile` 事件（真值=进入）。[changearea](../../../raw/current/scripts/components/areaaware.lua#L59)、[tile 事件](../../../raw/current/scripts/components/areaaware.lua#L62-L70)
- 修改边界：节点数据来自 `TheWorld.Map:FindVisualNodeAtPoint` 与 `TheWorld.topology`，离图/未加载区域 `current_area_data` 为 nil；不存档，状态即时重建。[节点数据](../../../raw/current/scripts/components/areaaware.lua#L50-L57)

## 关键接口

| 接口／字段 | 已核验类型／单位 | 条件、作用与证据 |
|---|---|---|
| `UpdatePosition(x, y, z)` | 无返回 | 找到节点索引后与 `current_area` 比对，变化才重建 `current_area_data` 并推 `changearea`；随后处理地块监视表。 OnUpdate 阈值触发时会以 y=0 调用。[L45-L71](../../../raw/current/scripts/components/areaaware.lua#L45-L71) |
| `current_area_data` | `{id, type, center, poly, tags}` 或 nil | id 取 `TheWorld.topology.ids[node_index]`；tags 是节点拓扑 tag（如 `Nightmare`、`Atrium`）。[L50-L57](../../../raw/current/scripts/components/areaaware.lua#L50-L57) |
| `CurrentlyInTag(tag)` | 布尔 | `current_area_data.tags` 里 `table.contains` 判断；沙暴、月风暴、梦魇循环等按它判定实体入区。[L89-L91](../../../raw/current/scripts/components/areaaware.lua#L89-L91)、[沙暴](../../../raw/current/scripts/components/sandstorms.lua#L128-L135) |
| `SetUpdateDist(dist)` | 距离单位 | 每帧移动超过 `dist` 才重查区域；玩家设 0.45（`updatedistsq = dist*dist`）。[L81-L83](../../../raw/current/scripts/components/areaaware.lua#L81-L83)、[玩家配置](../../../raw/current/scripts/prefabs/player_common.lua#L2494-L2495) |
| `StartWatchingTile(tile_id)`／`StopWatchingTile(tile_id)` | 无返回 | 注册目标地块（`WORLD_TILES.*`），进出时推 `on_<INVERTED_WORLD_TILES 名>_tile`；全表清空才撤销更新分支。[L113-L126](../../../raw/current/scripts/components/areaaware.lua#L113-L126) |
| `lastpt` | Vector3 | 上次检测点；月风暴/沙暴深度公式直接读它算"入区深度"。[L5](../../../raw/current/scripts/components/areaaware.lua#L5)、[moonstorms](../../../raw/current/scripts/components/moonstorms.lua#L77-L78) |
| `StartCheckingPosition(checkinterval)` | 无返回 | 周期任务版检测；**原版零调用者**（死接口），且未传间隔时 `self.checkinterval` 为 nil 会报错。[L109-L111](../../../raw/current/scripts/components/areaaware.lua#L109-L111) |

## 生命周期

- 构造即 `StartUpdatingComponent`；`done_embark_movement`（跳上下船动作结束）时强制刷新一次，弥补平台移动期间节点判定滞后。[L9-L12](../../../raw/current/scripts/components/areaaware.lua#L9-L12)
- `OnUpdate`：距 `lastpt` 的 xz 平面距离平方超过 `updatedistsq` 才调 `UpdatePosition`。[L73-L79](../../../raw/current/scripts/components/areaaware.lua#L73-L79)
- `OnRemoveFromEntity`：停更新、摘登船监听；`_TestArea`（L21-L43）为无调用者的死方法（map.lua 另有同名逻辑的独立实现）。[L16-L19](../../../raw/current/scripts/components/areaaware.lua#L16-L19)

## 典型接入

### 原版宿主：谁挂它、读什么

- 挂载：玩家（[player_common](../../../raw/current/scripts/prefabs/player_common.lua#L2494-L2495)，并额外监视 RIFT_MOON/LUNAR_MARSH/OCEAN_ICE 三地块（[L2664-L2666](../../../raw/current/scripts/prefabs/player_common.lua#L2664-L2666)））；梦魇猴（[monkey](../../../raw/current/scripts/prefabs/monkey.lua#L483)，按 `CurrentlyInTag("Nightmare")` 变身（[L369](../../../raw/current/scripts/prefabs/monkey.lua#L369)））；粉末猴（[powdermonkey](../../../raw/current/scripts/prefabs/powdermonkey.lua#L318)）、远古织影者（[stalker](../../../raw/current/scripts/prefabs/stalker.lua#L1764)，brain 按 `GetCurrentArea` 找墓地区域）、WX-78 占据体（[wx78_possessedbody](../../../raw/current/scripts/prefabs/wx78_possessedbody.lua#L646-L647)，监视海冰供"滑脚"判定）。
- 读取面：梦魇循环（[nightmareclock](../../../raw/current/scripts/components/nightmareclock.lua#L70-L71)）、动态音乐（[dynamicmusic](../../../raw/current/scripts/components/dynamicmusic.lua#L271-L277)）、沙暴/月风暴深度（[sandstorms](../../../raw/current/scripts/components/sandstorms.lua#L99-L100)）、出生点重定位（[playerspawner](../../../raw/current/scripts/components/playerspawner.lua#L268-L269)）等。
- Wurt 技能联动：沼泽抗湿技能按 `StartWatchingTile(WORLD_TILES.SHADOW_MARSH)` + `on_LUNAR_MARSH_tile`/`on_SHADOW_MARSH_tile` 事件启停效果。[wurt](../../../raw/current/scripts/prefabs/wurt.lua#L605-L615)

### 模组：让 NPC 感知区域

- 前提与选择：服务端移动实体即可；需要区域 tag 查询的（如"进梦魇区变身"）用 `CurrentlyInTag`，需要地块判定的用 `StartWatchingTile` + 实体事件。
- 生效与撤销：组件构造即开始更新；`changearea` 无数据（nil payload）也会在离开已加载节点时推送，监听方需容错。

```lua
-- 服务端 NPC：进入梦魇区域时标记
inst:AddComponent("areaaware")
inst:ListenForEvent("changearea", function(inst, data)
    local innightmare = data ~= nil and data.tags ~= nil and table.contains(data.tags, "Nightmare")
    if innightmare then
        inst:AddTag("nightmare_zone")
    else
        inst:RemoveTag("nightmare_zone")
    end
end)
```

- 调整与验证：示例仅语法检查未运行；`data` 即 `current_area_data`，`tags` 为拓扑 tag 字符串数组；频繁变区域时事件量与刷新距离成正比，可调大 `SetUpdateDist`。

## 常见陷阱与限制

- y 坐标被忽略：OnUpdate 用 `UpdatePosition(x, 0, z)`，高空/洞穴层差异不影响判定；但船体移动不触发 `OnUpdate` 阈值（lastpt 只在重查后更新），依赖 `done_embark_movement` 补一次。[L76](../../../raw/current/scripts/components/areaaware.lua#L76)、[L11-L12](../../../raw/current/scripts/components/areaaware.lua#L11-L12)
- 节点为空时 `current_area_data` 是 nil 且 `current_area` 记 0：`CurrentlyInTag` 静默返回 false，监听 `changearea` 收到的 data 也是 nil，别当作"在 0 号区域"。[L47-L57](../../../raw/current/scripts/components/areaaware.lua#L47-L57)
- 拓扑 tag ≠ 地块类型：`Nightmare`/`Atrium` 等来自世界生成拓扑；瓦片判定（海冰、影沼）必须走 `StartWatchingTile`，两套机制不可互换。[L62-L70](../../../raw/current/scripts/components/areaaware.lua#L62-L70)
- 检测是轮询式：实体瞬移超过阈值才重查；想即时响应用 `UpdatePosition` 手动刷（playerspawner 落地即如此）。[playerspawner](../../../raw/current/scripts/components/playerspawner.lua#L268-L269)

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。覆盖 areaaware.lua 全文；核对五个宿主挂载、changearea 的多系统监听（grottowarmanager/caveweather/playervision/quaker/stormwatcher 等 grep 确认监听存在）与沙暴/月风暴/梦魇/出生点读取点、wurt 与玩家地块监视；未覆盖拓扑节点数据的 worldgen 生成过程。
- 游戏运行：未运行；本页结论均来自源码静态核验。
- 结构检查：`validate_page.py`、`validate_source_links.py --strict` 与示例 Lua 语法检查通过（见批记录）。

相关：[[reference/components/charliearenawatcher]]。
