---
title: "wobycourier"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "Woby 快递坐标组件：按分片保存 Woby 记住的箱子坐标，经 classified netvar 同步客户端并驱动地图标记。"
aliases: ["沃比快递", "Woby快递箱", "wobycourier"]
tags: [component, network-sync, item]
dst_build: "steam-buildid:26230496"
sources: ["components/wobycourier.lua", "prefabs/walter.lua", "prefabs/woby_commands_classified.lua", "constants.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# wobycourier

## 快速判断

- 职责与适用：挂在沃尔特（walter，[挂接](../../../raw/current/scripts/prefabs/walter.lua#L1198)）身上，按 `TheShard:GetShardId()` 为键保存 Woby（宠物）记住的箱子坐标，并同步到 `woby_commands_classified` 网络变量供小地图标记（[同步](../../../raw/current/scripts/components/wobycourier.lua#L9-L27)）。
- 优先使用：写坐标用 `StoreXZ(x, z)`（自带陆地校验），清除用 `ClearXZ`；不直接改 `positions` 表。
- 重要副作用：同步时先 `set_local` 再 `set`（本机立即生效 + 网络广播），并按哨兵值表示"无箱子"（[写法](../../../raw/current/scripts/components/wobycourier.lua#L13-L21)）。
- 修改边界：`positions` 覆盖所有分片并在 OnSave/OnLoad 全量保留，但本组件只写当前分片键（[存读](../../../raw/current/scripts/components/wobycourier.lua#L58-L77)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `CanStoreXZ(x, z)` | `-> bool` | 取该点地砖（优先 undertile 底层地砖），要求 `TileGroupManager:IsLandTile`（[实现](../../../raw/current/scripts/components/wobycourier.lua#L29-L35)）。 |
| `StoreXZ(x, z)` | `-> bool` | 校验通过则写 `positions[shardid]` 并 `NetworkLocation`；失败返回 false（[实现](../../../raw/current/scripts/components/wobycourier.lua#L37-L47)）。 |
| `ClearXZ()` | `-> bool` | 清除当前分片坐标并同步；无记录返回 false（[实现](../../../raw/current/scripts/components/wobycourier.lua#L49-L56)）。 |
| `NetworkLocation()` | `-> 无` | 把当前分片坐标（或哨兵 `WOBYCOURIER_NO_CHEST_COORD = -32767`，[常量](../../../raw/current/scripts/constants.lua#L2931)）写入 `woby_commands_classified.chest_posx/z`；`inst == ThePlayer` 时推 `updatewobycourierchesticon`（[实现](../../../raw/current/scripts/components/wobycourier.lua#L9-L27)）。 |

## 典型接入

### 记箱子命令与地图标记（原版参照）

- 前提与选择：Woby 的 `REMEMBERCHEST` 命令要求沃尔特已激活 `walter_camp_wobycourier` 技能，随后调宿主的 `StoreXZ`（([woby_commands_classified.lua:392-396](../../../raw/current/scripts/prefabs/woby_commands_classified.lua#L392-L396)）。
- 生效与撤销：classified 挂接／换宠物时统一 `NetworkLocation`（[woby_commands_classified.lua:215](../../../raw/current/scripts/prefabs/woby_commands_classified.lua#L215）；客户端监听 `updatewobycourierchesticon` 生成 `wobycourier_marker` 地图标记（[walter.lua:1159](../../../raw/current/scripts/prefabs/walter.lua#L1159）。

```lua
-- 服务端：记住某点（等价 REMEMBERCHEST 命令内的关键调用，walter_inst 为沃尔特实体）
local ok = walter_inst.components.wobycourier:StoreXZ(x, z)
if not ok then
    -- 非陆地瓦格，命令侧会推 chest_pos_failed
end
```

- 调整与验证：坐标为世界坐标（x, z）；y 轴不参与。

## 常见陷阱与限制

- **哨兵值判断无箱子**：`-32767` 远超正常地图范围，读 netvar 的客户端代码需显式比对哨兵（[常量](../../../raw/current/scripts/constants.lua#L2931)、[写入](../../../raw/current/scripts/components/wobycourier.lua#L18-L21)）。
- **只写本分片**：`StoreXZ`/`ClearXZ` 只操作 `positions[TheShard:GetShardId()]`；跨分片的旧记录靠 OnSave/OnLoad 原样搬运（[存读](../../../raw/current/scripts/components/wobycourier.lua#L58-L77)）。
- **图标事件仅限主机玩家**：`updatewobycourierchesticon` 只在 `inst == ThePlayer` 时推（专用服务器上纯客户端依赖 netvar 脏事件，[分支](../../../raw/current/scripts/components/wobycourier.lua#L23-L25)）。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；覆盖组件全文、walter 挂接与标记生成、woby_commands_classified 两处调用、哨兵常量。
- 结构检查：本批 validate_page、validate_source_links 及示例 lua 语法探测（见批级记录）。
- 游戏运行：未运行。
