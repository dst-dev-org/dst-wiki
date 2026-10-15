---
title: "quagmire_recipebook"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "熔炉（暴食）活动菜谱发现广播队列：主模拟把新菜谱排队写入网络字段，客户端解码后推 quagmire_recipediscovered 事件供菜谱书模块消费。"
aliases: ["暴食菜谱", "quagmire_recipebook 组件", "菜谱发现广播", "quagmire 菜谱"]
tags: [component, network, server-client]
size_class: simple
dst_build: "steam-buildid:26230496"
sources: ["components/quagmire_recipebook.lua", "prefabs/quagmire_network.lua", "quagmire_recipebook.lua", "main.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# quagmire_recipebook

## 快速判断

- 职责与适用：挂在活动网络实体 `quagmire_network`（`persists = false`、CLASSIFIED）上的单向广播组件：主模拟收 `ms_quagmirerecipediscovered` 入队，每 3 秒广播一条菜谱发现；客户端解码推世界事件 `quagmire_recipediscovered`。[安装](../../../raw/current/scripts/prefabs/quagmire_network.lua#L54) [事件](../../../raw/current/scripts/components/quagmire_recipebook.lua#L124-L130)
- 优先使用：客户端消费走世界事件（product/dish/station/overcooked/ingredients），`station` 已转回 "pot"/"oven"/"grill"，`ingredients` 转回 prefab 名数组。[推送](../../../raw/current/scripts/components/quagmire_recipebook.lua#L124-L130) [名称表](../../../raw/current/scripts/components/quagmire_recipebook.lua#L41-L56)
- 重要副作用：与 [[reference/components/quagmire_recipeprices]] 同实体协同，共用 klump 资源加载分支（`QUAGMIRE_USE_KLUMP` 时按 `_klumpkey` 加载菜品贴图与名称串）。[资源](../../../raw/current/scripts/components/quagmire_recipebook.lua#L104-L111)
- 修改边界：无公开方法、无存读钩子；仅暴食活动世界有意义。注意与脚本根目录同名模块 `quagmire_recipebook.lua`（`TheRecipeBook`，main.lua 载入）区分——模块是事件消费方，组件是生产方。[模块载入](../../../raw/current/scripts/main.lua#L417)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| 主模拟事件 `ms_quagmirerecipediscovered` | 监听 TheWorld | 入参 `data.recipe`：product、dish、station（pot/oven/grill）、overcooked、ingredients（≤4 个 prefab 名）；转 ID 入队，空闲立即开跑。[处理](../../../raw/current/scripts/components/quagmire_recipebook.lua#L159-L176) |
| 客户端世界事件 `quagmire_recipediscovered` | TheWorld 上 | `_recipename` dirty 后解码推送；`ingredients` 里 ID 0／未知名自动剔除。[推送](../../../raw/current/scripts/components/quagmire_recipebook.lua#L113-L130) |
| 网络字段组 | `net_string`/`net_tinybyte`/`net_bool`/`net_smallbyte` | `_recipename`（带 `recipedirty`）、`_klumpkey`、`_dish`、`_station`、`_overcooked`、`_ingredients[1..4]`（`MAX_INGREDIENTS = 4`）。[定义](../../../raw/current/scripts/components/quagmire_recipebook.lua#L76-L84) |
| 消费者 | 原版模块 | 根目录 `quagmire_recipebook.lua` 同时监听本事件与估价事件，写入 `TheRecipeBook`。[监听](../../../raw/current/scripts/quagmire_recipebook.lua#L234-L235) |

## 常见陷阱与限制

- **组件与模块同名：** `require("quagmire_recipebook")` 拿到的是脚本根目录的客户端菜谱数据模块，不是本组件；模组想读发现历史应走模块或事件，别试图 require 组件。[载入点](../../../raw/current/scripts/main.lua#L417)
- **广播节拍与估价组件一致：** 队列每 3 秒一条、串行；客户端延迟 3 秒本地清 `_recipename` 显示但不推空事件。[队列](../../../raw/current/scripts/components/quagmire_recipebook.lua#L135-L153)
- **ingredients 只有 4 槽：** 入队保留全部原料，但广播时 `ProcessQueue` 只按 i=1..4 写网络字段，第 5 个起被静默丢弃。[上限](../../../raw/current/scripts/components/quagmire_recipebook.lua#L56) [截断点](../../../raw/current/scripts/components/quagmire_recipebook.lua#L144-L146)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（208 行），核验 quagmire_network 安装、main.lua 模块载入与根模块事件监听。
- 结构检查：2026-10-28 validate_page.py 通过（本页）；validate_source_links.py --strict 本批 16 页通过。
- 游戏运行：未运行。

相关：[[reference/components/quagmire_recipeprices]]
