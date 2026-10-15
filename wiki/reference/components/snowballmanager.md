---
title: "snowballmanager"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "冬宴（Winter's Feast）雪球世界宿主组件：降雪且地表积雪时按 DataGrid 分区限密生成 snowball_item，降雪停止或积雪消融时逐只移除，登记表随世界存档恢复。"
aliases: ["snowballmanager", "雪球管理器", "冬宴雪球生成", "snowball_item 调度"]
tags: [component, lifecycle, callback]
dst_build: "steam-buildid:26230496"
sources: ["components/snowballmanager.lua", "prefabs/forest.lua", "tuning.lua", "prefabs/world_network.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# snowballmanager

## 快速判断

- 职责与适用：挂在 forest 世界实体的服务端组件（构造断言 mastersim）；冬宴活动（`SPECIAL_EVENTS.WINTERS_FEAST`）且 `issnowing`＋`issnowcovered` 时启用，按 `SNOWBALLMANAGER_SPACING`(10 瓦) 分区限密（每区 `SNOWBALLMANAGER_DENSITY`(2) 只）生成雪球；条件不满足时逐只移除并停更。[构造与状态](../../../raw/current/scripts/components/snowballmanager.lua#L4-L16) [启停门控](../../../raw/current/scripts/components/snowballmanager.lua#L132-L147)
- 优先使用：让自定义雪球纳入限密与融化豁免 → `RegisterSnowball(snowball)`；主动补雪球 → `TryToCreateSnowballAnywhere()`／`TryToCreateSnowballForEachPlayer()`。[注册](../../../raw/current/scripts/components/snowballmanager.lua#L54-L73)
- 重要副作用：登记中的雪球经 [[reference/components/snowballmelting]] 暂停融化，注销时恢复；生成点强制 `MAX_PHYSICS_RADIUS` 内无实体；移除时若非休眠补 `snowball_shatter_fx`。[融化联动](../../../raw/current/scripts/components/snowballmanager.lua#L49-L52) [破碎特效](../../../raw/current/scripts/components/snowballmanager.lua#L177-L186)
- 修改边界：仅 forest 挂载（cave 无）；构造时捕获四个 `SNOWBALLMANAGER_*` 调谐为局部量，之后改 `TUNING` 不生效；网格尺寸依赖 `worldmapsetsize` 事件初始化。[挂载](../../../raw/current/scripts/prefabs/forest.lua#L688 [调谐捕获](../../../raw/current/scripts/components/snowballmanager.lua#L9-L12)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| 构造 | `(inst)` | `snowballs`（实体→格索引）、`snowballscount`、`enabled`、`accumulator`；`DoTaskInTime(0)` 前 `snowballgrid` 为 nil，靠 `worldmapsetsize` 事件建 DataGrid。([snowballmanager.lua:19-28](../../../raw/current/scripts/components/snowballmanager.lua#L19-L28) |
| `RegisterSnowball(snowball)` | 雪球实体 | 按位置定格、格计数自增；监听 `onremove`／`onputininventory` 自动注销；有 `snowballmelting` 则 `StopMelting()`。([snowballmanager.lua:54-73](../../../raw/current/scripts/components/snowballmanager.lua#L54-L73) |
| `UnregisterSnowball(snowball)` | 雪球实体 | 格计数自减、清零删格数据；恢复融化。([snowballmanager.lua:37-53](../../../raw/current/scripts/components/snowballmanager.lua#L37-L53) |
| `NoSnowballTest(map, x, y, z)` | 选点过滤器 | 仅陆地瓦且无地面覆盖禁铺层；`MAX_PHYSICS_RADIUS` 内无实体；所在分区计数未满密度。([snowballmanager.lua:75-92](../../../raw/current/scripts/components/snowballmanager.lua#L75-L92) |
| `TryToCreateSnowballAtPoint(x, y, z, skip)` | 返回 boolean | 过滤后生成 `snowball_item`（播 `spawn`＋`ground_small` 动画）并登记。([snowballmanager.lua:94-105](../../../raw/current/scripts/components/snowballmanager.lua#L94-L105) |
| `TryToCreateSnowballAnywhere()`／`TryToCreateSnowballForEachPlayer()` | 无参 | 全图 `FindRandomPointWithFilter(50, ...)` 随机补 1 只；后者对不在平台上的玩家在相机可视距离环带内最多 10 次采样（有意向玩家聚密）。([snowballmanager.lua:107-130](../../../raw/current/scripts/components/snowballmanager.lua#L107-L130) |
| `SetEnabled(enabled)`／`TryToEnable()` | 返回 boolean | 开启时启动更新；`TryToEnable` 校验冬宴活动与 `issnowing and issnowcovered`。([snowballmanager.lua:132-147](../../../raw/current/scripts/components/snowballmanager.lua#L132-L147) |
| 世界状态联动 | WatchWorldState | `issnowing`／`issnowcovered` 变化都触发 `TryToEnable`；`OnPostInit` 用当前世界状态做首次判定（由 ([world_network.lua:5-10](../../../raw/current/scripts/prefabs/world_network.lua#L5-L10) 的组件 OnPostInit 循环调用）。([snowballmanager.lua:149-170](../../../raw/current/scripts/components/snowballmanager.lua#L149-L170) |
| `OnSave`／`LoadPostPass` | 存档钩子 | 存登记雪球 GUID 列表；载入逐个重登记并在有余量时重启更新。([snowballmanager.lua:201-225](../../../raw/current/scripts/components/snowballmanager.lua#L201-L225) |

## 典型接入

### 复用"活动＋世界状态门控的限量地表生成"

- 前提与选择：与 [[reference/components/leafpilemanager]] 同构（DataGrid 分区限密、活动门控、随玩家撒点）；季节雪球场景直接参照该页与本页，世界类型须为 forest。
- 生效与撤销：启用期间每 `SNOWBALLMANAGER_SECONDS_PER_SPAWN`(5) 秒一轮"全图 1 只＋每玩家 1 只"；停用后每 0.5 秒移除一只未积雪覆盖的登记雪球直至清空自停更新。
- 原版依据：调谐键 ([tuning.lua:8573-8576](../../../raw/current/scripts/tuning.lua#L8573-L8576)；`OnPostInit` 首判机制见上表。

```lua
-- 服务端：把一个自定义雪球实体纳入管理（限密计数＋暂停融化）
if TheWorld.components.snowballmanager ~= nil then
    TheWorld.components.snowballmanager:RegisterSnowball(snowball)
end
-- 服务端：在指定点补一只雪球（跳过选点过滤的用法见 skip 参数）
-- TheWorld.components.snowballmanager:TryToCreateSnowballAtPoint(x, y, z)
```

- 调整与验证：密度／间隔都是构造期快照，需覆盖 `TUNING.SNOWBALLMANAGER_*` 后再构造世界组件；`GetDebugString` 未实现，用 `snowballscount` 自查。

## 常见陷阱与限制

- 双条件门控：只下雪不积雪（`issnowcovered` 为假）不会生成，但已生成的雪球在积雪覆盖时也不清除（([snowballmanager.lua:174-186](../../../raw/current/scripts/components/snowballmanager.lua#L174-L186) 只在非积雪分支移除）。
- 被捡起即出管：`onputininventory` 触发注销并恢复融化（([snowballmanager.lua:48-52](../../../raw/current/scripts/components/snowballmanager.lua#L48-L52)），放进容器的雪球回到世界后需重新登记才免融。
- `LoadPostPass` 而非 `OnLoad`：依赖实体重建后的引用，直接模仿时注意钩子选择（([snowballmanager.lua:213](../../../raw/current/scripts/components/snowballmanager.lua#L213）。
- 网格初始化依赖 `worldmapsetsize` 事件：在事件前调用 `RegisterSnowball` 会因 `snowballgrid` 为 nil 报错（([snowballmanager.lua:20-27](../../../raw/current/scripts/components/snowballmanager.lua#L20-L27)）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，通读组件全文；核验 forest.lua 挂载、SNOWBALLMANAGER_* 调谐、world_network 的 OnPostInit 派发与 snowballmelting 联动。
- 游戏运行：未运行。
- 结构检查：未执行。

相关链接：[[reference/components/leafpilemanager]]、[[reference/components/snowballmelting]]
