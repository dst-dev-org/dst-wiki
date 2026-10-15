---
title: "crabkingspawner"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "森林世界实体上的蟹王生成兜底：读档后如果蟹王生成器丢失，按旧存档数据重建 crabking_spawner。"
aliases: ["蟹王生成器恢复", "世界级刷怪兜底", "crabkingspawner"]
tags: [component, lifecycle]
dst_build: "steam-buildid:26230496"
size_class: simple
sources: ["components/crabkingspawner.lua", "prefabs/forest.lua", "prefabs/crabking_spawner.lua", "prefabs/crabking.lua", "map/static_layouts/crabking.lua", "map/rooms/forest/terrain_ocean.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# crabkingspawner

## 快速判断

- 职责与适用：挂在森林（`forest` 地图 prefab）世界实体上的服务端组件，只有一个 `LoadPostPass` 存读钩子：读档后发现世界里没有 `crabking_spawner` 实体时，按存档数据在旧坐标重建生成器。正常生成器的来源是世界生成（静态布局放置），本组件是兜底。[宿主](../../../raw/current/scripts/prefabs/forest.lua#L678)
- 优先使用：一般不需要直接使用；它是世界级后勤组件，属于原版存档兼容路径。
- 重要副作用：重建时把 `childspawner.childreninside` 清 0（不立即内置蟹王），若有 `timetorespawn` 数据则用 `worldsettingstimer:StartTimer("regen_crabking", ...)` 恢复重生计时（[LoadPostPass](../../../raw/current/scripts/components/crabkingspawner.lua#L20-L31)）。
- 修改边界：构造即断言 mastersim（[L7](../../../raw/current/scripts/components/crabkingspawner.lua#L7)）；无 OnSave／OnLoad，`LoadPostPass` 只读不写；不影响已有的 `crabking_spawner`（有实体就不重建）。

## 关键接口

| 接口／字段 | 已核验类型 | 条件、作用与证据 |
|---|---|---|
| `LoadPostPass(newents, data)` | 无返回 | 存读后置钩子；`data.crabkingx`／`data.crabkingz` 决定重建坐标，`data.timetorespawn` 可选恢复 `regen_crabking` 计时。[L20-L31](../../../raw/current/scripts/components/crabkingspawner.lua#L20-L31) |
| （重建目标）`crabking_spawner` | prefab | 由 childspawner（childname `crabking`）与 `regen_crabking` 计时器驱动刷新。[生成器实现](../../../raw/current/scripts/prefabs/crabking_spawner.lua#L43-L57) |

## 常见陷阱与限制

- 本快照 scripts 内没有任何代码写入 `crabkingx`／`crabkingz`／`timetorespawn`（全库 grep 仅命中组件自身）：这批数据只可能来自旧版本存档，属遗留兼容路径；新存档走世界生成的 `crabking_spawner`（[静态布局](../../../raw/current/scripts/map/static_layouts/crabking.lua#L269)、[地形房间 required_prefabs](../../../raw/current/scripts/map/rooms/forest/terrain_ocean.lua#L38)）。因此不要指望给组件喂 `OnSave` 数据就能自定义坐标——它没有保存端。
- 世界里已存在任一带 `crabking_spawner` tag 的实体时不会重建（[L21](../../../raw/current/scripts/components/crabkingspawner.lua#L21)）；另一处兜底在蟹王本体 prefab：找不到生成器时会现场补一个（[crabking.lua#L704-L705](../../../raw/current/scripts/prefabs/crabking.lua#L704-L705)），两条路径并存。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。完整阅读 components/crabkingspawner.lua（37 行）与 prefabs/crabking_spawner.lua，核对宿主 forest.lua 的 AddComponent、世界生成放置点、crabking.lua 的兜底生成；grep 确认存档数据字段在 scripts 内无写入者。未覆盖引擎侧 LoadPostPass 数据装配细节。
- 游戏运行：未运行。
- 结构检查：validate_page.py 通过（0 issue；42 行、3859 字节，simple 档内）；validate_source_links.py --strict 通过（9 处源码引用，0 error／0 warning）；本页无完整 Lua 代码块。
