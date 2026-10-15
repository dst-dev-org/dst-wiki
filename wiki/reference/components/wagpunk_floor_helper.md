---
title: "wagpunk_floor_helper"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "挂世界 shard 网络实体(TheWorld.net)的瓦格纳竞技场几何同步组件：以 net 变量广播竞技场原点与屏障状态，提供三矩形拼合的范围/厚度查询。"
aliases: ["wagpunk 竞技场范围", "竞技场屏障同步", "wagpunk_floor_helper"]
tags: [component, network-sync]
dst_build: "steam-buildid:26230496"
size_class: simple
sources: ["components/wagpunk_floor_helper.lua", "prefabs/forest_network.lua", "components/map.lua", "prefabs/wagpunk_floor_kit.lua", "components/wagpunk_arena_manager.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# wagpunk_floor_helper

## 快速判断

- 职责与适用：挂 forest 网络实体（prefabs/forest_network.lua:20，所有端存在）；4 个 net 变量：`barrier_active`/`arena_active`（net_bool）、`arena_origin_x/z`（net_float）（components/wagpunk_floor_helper.lua:11-18）。
- 优先使用：任何端都应经 Map 包装器调用——`Map:IsPointInWagPunkArena`、`Map:IsPointInWagPunkArenaAndBarrierIsUp`、`Map:IsXZWithThicknessInArena(AndBarrierIsUp)`、`Map:GetWagPunkArenaCenterXZ`、`Map:IsWagPunkArenaBarrierUp`（components/map.lua:1496-1556）。
- 重要副作用：`TryToSetMarker` 记录竞技场中心标记并广播 origin/active；标记 onremove 时清零（:90-109）。
- 修改边界：竞技场形状固定为 hermitcrab_01 布局的三矩形拼合（横宽 7/竖高 7/中心方 6 个 TILE_SCALE，宽高由 7/6/5 派生，:20-23）；客户端只读 net 值。

## 关键接口

| 接口/字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `IsPointInArena` | `(x, y, z) -> boolean` | 仅 `arena_active` 为真有效，否则恒 false。:59-65 |
| `IsXZWithThicknessInArena` | `(x, z, thickness) -> boolean` | 边界带查询：正厚度取"带内带外差集"，用于屏障附近判定。:67-74 |
| `GetArenaOrigin` | `() -> x, z\|nil` | 读 origin net 值；未激活返回 nil。:76-82 |
| `IsBarrierUp` | `() -> boolean` | 读 `barrier_active`；boss 战状态由 wagpunk_arena_manager 设置并推 `ms_wagpunk_barrier_isactive`（components/wagpunk_arena_manager.lua:1477-1490）。:84-86 |
| `TryToSetMarker` | `(inst) -> nil` | 服务器：已有标记时移除新标记（冲突保护），否则记录并广播。:97-109 |

## 典型接入

### 原版参照：标记与屏障的写入方

- 前提与选择：wagpunk_floor_kit.lua:161、:170 让地板标记实体入世时 TryToSetMarker；arena_manager 只写 barrier_active；其余读取全走 Map 包装器。
- 生效与撤销：标记实体移除即清零 origin/active（:90-95），竞技场随之"不存在"。

## 常见陷阱与限制

- origin 用 net_float，注释自述若 arena 仅 tile 对齐本可用 ushort（:13）——直接消费 value() 时注意 float 精度。
- 三矩形判定以 origin 为中心对称，不含旋转；世界层无旋转补偿。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，通读组件全部 111 行，核验挂载（forest_network.lua:20）、Map 包装器组（map.lua:1496-1556）、标记写入（wagpunk_floor_kit.lua:161/170）与屏障写入（wagpunk_arena_manager.lua:1477-1490）。
- 游戏运行：未运行。
