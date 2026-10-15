---
title: "forestpetrification：森林石化周期"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "世界级森林石化：冷却若干年后从随机一棵可石化树洪泛搜索相连林区，区内树少于阈值时整体石化，再进入下一轮冷却。"
aliases: [森林石化, 石化周期, forestpetrification, 树木石化事件]
tags: [component, worldgen, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/forestpetrification.lua", "components/petrifiable.lua", "prefabs/forest.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# forestpetrification（森林石化周期）

## 快速判断

- 职责与适用：`forest` 世界挂载的服务端组件（[forest.lua:668](../../../raw/current/scripts/prefabs/forest.lua#L668）。世界事件 `ms_registerpetrifiable`（由 [[reference/components/petrifiable]] 组件注册，[petrifiable.lua:16](../../../raw/current/scripts/components/petrifiable.lua#L16）维护全图可石化树清单；冷却到期后 BFS 搜索一片林区，若区内树数低于全图跟踪量的 20% 则整片石化（[石化执行](../../../raw/current/scripts/components/forestpetrification.lua#L152-L166)）。
- 优先使用：要调周期 → 改 `TUNING.PETRIFICATION_CYCLE.MIN_YEARS/MAX_YEARS`（0.6~0.9 年，[调谐](../../../raw/current/scripts/tuning.lua#L3250-L3254)）；要立即触发一次搜索 → `FindForest()`。
- 重要副作用：石化后在世界中心点生成 `petrify_announce` 公告实体；石化走 `petrifiable:Petrify(false)`（非玩家凿击来源）（[调用](../../../raw/current/scripts/components/forestpetrification.lua#L160)）。
- 修改边界：构造断言 [mastersim](../../../raw/current/scripts/components/forestpetrification.lua#L7)；搜索每帧预算 `MAX_WORK`(10) 个区块，洪泛跨度上限 36×36（[常量](../../../raw/current/scripts/components/forestpetrification.lua#L17-L23)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `FindForest()` | 无参 | 停冷却并立即开始一次森林搜索（[实现](../../../raw/current/scripts/components/forestpetrification.lua#L271-L274)）。 |
| `OnPostInit()` | 载入后调用 | 无冷却且未在搜索时按 `RandomizeYearPart(MIN, MAX)` 起冷却（[实现](../../../raw/current/scripts/components/forestpetrification.lua#L207-L211)）。 |
| `OnUpdate()` | 组件更新 | 每帧消费 `MAX_WORK` 预算做 BFS；搜完若 `0 < _numfound < 0.2×#_tracked` 则石化并进入半周期冷却（0.3~0.45 年），否则重试（至多 5 次）或进半冷却（[主循环](../../../raw/current/scripts/components/forestpetrification.lua#L217-L242)）。 |
| `LongUpdate(dt)` | 秒 | 跳时间时把冷却按 `TOTAL_DAY_TIME` 折算天数连续推进并触发搜索（[LongUpdate](../../../raw/current/scripts/components/forestpetrification.lua#L244-L265)）。 |
| `OnSave/OnLoad/LoadPostPass` | 存读钩子 | 存冷却天数（搜索中存 0）；读回 ≥1 起冷却，=0 则重开搜索（[存读](../../../raw/current/scripts/components/forestpetrification.lua#L280-L300)）。 |

搜索实现（[CheckSector](../../../raw/current/scripts/components/forestpetrification.lua#L85-L115)）：以随机一棵跟踪树为起点，按 4×4√2 单位网格向四邻扩展（跨度超过 36² 停扩），统计带 `petrifiable` 标签的实体去重计数；冷却由 `ms_cyclecomplete`（天循环完成）递减计数（[冷却](../../../raw/current/scripts/components/forestpetrification.lua#L143-L150)）。

## 典型接入

### 让自定义树参与石化

- 前提与选择：树实体服务端挂 [[reference/components/petrifiable]] 组件即自动进入跟踪（注册由该组件完成，本组件只听世界事件）。
- 生效与撤销：树被移除时自动移出跟踪表（swap-remove，[StopTracking](../../../raw/current/scripts/components/forestpetrification.lua#L168-L177)）；石化是单向替换，由 `petrifiable` 内部处理。

```lua
-- 服务端，树 prefab 的 master postinit 内
inst:AddComponent("petrifiable")  -- 注册进森林石化与石化的世界跟踪
```

- 调整与验证：石化比例阈值 0.2 写死在组件常量 `PETRIFICATION_THRESHOLD`（[常量](../../../raw/current/scripts/components/forestpetrification.lua#L21)）；冷却"年"按四季长度折算天数（[折算](../../../raw/current/scripts/components/forestpetrification.lua#L49-L59)）。

## 常见陷阱与限制

- **"区内树少"是相对全图而言**：`_numfound < 0.2 × #_tracked`，全图树极少时几乎所有搜索都会通过并石化；全图树极多时很难触发（[判定](../../../raw/current/scripts/components/forestpetrification.lua#L224)）。
- **搜索失败重试 5 次**：起点随机，若每次都搜不到低密度林区则放弃本轮进入半冷却（[重试](../../../raw/current/scripts/components/forestpetrification.lua#L228-L235)）。
- **冷却计时按天循环事件**：依赖世界 `ms_cyclecomplete`；`LongUpdate` 跳时间会自行补扣天数，不丢周期（[补扣](../../../raw/current/scripts/components/forestpetrification.lua#L250-L264)）。
- **跨度限制**：洪泛直径超过 72 单位（36×36 的平方距离）不再扩展，超大森林一次只石化一片（[跨度](../../../raw/current/scripts/components/forestpetrification.lua#L108)）。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496（steam-buildid:26230496）；覆盖 `components/forestpetrification.lua` 全部 317 行、`components/petrifiable.lua` 注册行、forest 挂载行、tuning PETRIFICATION_CYCLE。未覆盖：`petrifiable:Petrify` 的替换实现、`petrify_announce` 表现。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 通过；lua 示例通过 parser_probe 语法检查。
- 游戏运行：未运行。
