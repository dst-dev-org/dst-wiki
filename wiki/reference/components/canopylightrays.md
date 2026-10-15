---
title: "canopylightrays"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "在树冠宿主周围按 4 单位网格随机铺 lightrays_canopy 光柱特效，用全局引用计数让重叠树冠共享光柱。"
aliases: ["树冠光柱", "光柱特效", "canopylightrays"]
tags: [component]
dst_build: "steam-buildid:26230496"
sources: ["components/canopylightrays.lua", "prefabs/watertree_pillar.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# canopylightrays

## 快速判断

- 职责与适用：世界树等大型树冠宿主安装后，自动在周围圆形范围内随机生成光柱 FX；原版消费者仅 watertree_pillar（[安装](../../../raw/current/scripts/prefabs/watertree_pillar.lua#L462-L463)）。
- 优先使用：想复用同款树冠光柱直接 AddComponent 并按需覆盖 `range`；效果预置 `lightrays_canopy`。
- 重要副作用：光柱实体登记在全局表 `Global_Lightrays`（键 `"x-z"`），多树冠重叠时引用计数共享、不重复生成（[注册表](../../../raw/current/scripts/components/canopylightrays.lua#L25-L26)）。
- 修改边界：组件构造后 `DoTaskInTime(0)` 自动铺放；移除组件或宿主时按计数回收光柱；不写存档、无网络变量。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `range` | `number`，默认 `math.floor(TUNING.SHADE_CANOPY_RANGE/4)`（tuning.lua:6334 为 28，即 7 格） | 以 4 单位为步长的方形枚举半径，圆形判定后落点对齐 `floor(坐标/4)*4+2` 网格。[构造](../../../raw/current/scripts/components/canopylightrays.lua#L9-L18) [落点](../../../raw/current/scripts/components/canopylightrays.lua#L33-L34) |
| `lightray_prefab` | `string`，默认 `"lightrays_canopy"` | 光柱预制体名，可在安装后替换。[构造](../../../raw/current/scripts/components/canopylightrays.lua#L13) |
| `SpawnLightrays()` | 无参数 | `math.random() < 0.1` 的格子生成光柱，占用格只累计 `refs`；重复调用会继续叠引用。[实现](../../../raw/current/scripts/components/canopylightrays.lua#L28-L50) |
| `DespawnLightrays()` | 无参数 | 按 `lightray_positions` 逐项减引用，减到 0 移除光柱实体并清全局表项。[实现](../../../raw/current/scripts/components/canopylightrays.lua#L52-L63) |

## 典型接入

### 大型树木挂树冠光柱

- 前提与选择：树 prefab 的 master_postinit 内安装；若树冠小于原版水杉，可调小 `range` 控制光柱密度与数量。
- 生效与撤销：安装后下一帧自动铺放；组件/宿主移除时 `OnRemoveEntity` 与 `OnRemoveFromEntity` 同走 DespawnLightrays 回收（[移除](../../../raw/current/scripts/components/canopylightrays.lua#L20-L23)）。
- 原版依据：watertree_pillar 安装并保持默认 range（[宿主](../../../raw/current/scripts/prefabs/watertree_pillar.lua#L462-L463)）。

```lua
-- 树 prefab 的 master_postinit 内
inst:AddComponent("canopylightrays")
inst.components.canopylightrays.range = math.floor(TUNING.SHADE_CANOPY_RANGE / 4)
```

- 调整与验证：落点始终在 4 单位网格上（对齐 +2 偏移），改 `lightray_prefab` 可换成自定 FX，但需自行保证该预制的 Transform 定位方式兼容。

## 常见陷阱与限制

- `DespawnLightrays` 不清空 `lightray_positions`（[实现](../../../raw/current/scripts/components/canopylightrays.lua#L52-L63)）：组件存活期间手动重复调用会重复减引用，把仍被其他树冠共享的光柱提前移除；正常只在移除钩子里走一次。
- 光柱是全局共享实体，模组不要直接 `Remove()` 拿到的光柱，应只增删自己的树冠组件让计数闭环。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，覆盖全部接口与 watertree_pillar 消费链、tuning 取值；未运行游戏。
- 结构检查：validate_page.py、validate_source_links.py 与 Lua 语法探针通过。
