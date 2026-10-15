---
title: "hallucinations"
created: "2026-10-28"
updated: "2026-10-28"
type: component
size_class: simple
summary: "世界级幻觉组件：按本地玩家理智在黑暗处定时生成窥视眼、暗影窥伺者与暗影惊走者，纯客户端表现。"
aliases: ["幻觉", "hallucinations", "低理智幻影", "暗影幻觉"]
tags: [component]
dst_build: "steam-buildid:26230496"
sources: ["components/hallucinations.lua", "prefabs/cave.lua", "prefabs/forest.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# hallucinations

## 快速判断

- 职责与适用：挂在 TheWorld 的非 dedicated 端组件（[cave](../../../raw/current/scripts/prefabs/cave.lua#L346)、[forest](../../../raw/current/scripts/prefabs/forest.lua#L631)，均在 `not TheNet:IsDedicated()` 块内），以 `_player.replica.sanity` 为输入，定时在玩家附近生成三类幻觉实体。
- 优先使用：无需调用——`playeractivated`/`playerdeactivated` 自动启停（[注册](../../../raw/current/scripts/components/hallucinations.lua#L235-L236)）；要自定义幻觉需改 `HALLUCINATION_TYPES` 表。
- 重要副作用：`isnight` 且非满月且玩家无夜视时启动 `nightonly` 类幻觉，白天只跑 `shadowskittish`（[夜间开关](../../../raw/current/scripts/components/hallucinations.lua#L186-L192)）。
- 修改边界：仅表现层生成实体，不伤害玩家；组件无公开方法（除 `GetDebugString`，[调试](../../../raw/current/scripts/components/hallucinations.lua#L242-L248)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `HALLUCINATION_TYPES`（组件内局部表） | `{interval, variance, initial_variance, nightonly, spawnfn}` | 三类：creepyeyes（5±2.5s，夜）、shadowwatcher（30±15s，夜）、shadowskittish（10±5s，全时段）（[定义](../../../raw/current/scripts/components/hallucinations.lua#L11-L36)）。 |
| 各 `spawnfn(inst, hallucination)` | 内部函数 | 各自按理智阈值（0.6/0.5/0.8）与数量上限分叉：过亮重试、过多延后、够暗才生成（[creepyeyes](../../../raw/current/scripts/components/hallucinations.lua#L74-L102)、[watcher](../../../raw/current/scripts/components/hallucinations.lua#L107-L136)、[skittish](../../../raw/current/scripts/components/hallucinations.lua#L138-L163)）。 |
| `GetDebugString()` | → `string` | 打印各类型当前存活数（[实现](../../../raw/current/scripts/components/hallucinations.lua#L242-L248)）。 |

## 典型接入

### 理解生成条件（无外部接入面）

- 前提与选择：世界实体在非 dedicated 端挂载；本地玩家激活后监听其 `nightvision` 事件判断黑暗可见性（[玩家切换](../../../raw/current/scripts/components/hallucinations.lua#L194-L207)）。
- 生效与撤销：生成即 `StartTracking` 计数并监听 `onremove`（[计数](../../../raw/current/scripts/components/hallucinations.lua#L62-L72)）；玩家登出 `Stop()` 取消全部任务（[停止](../../../raw/current/scripts/components/hallucinations.lua#L173-L180)）。
- 原版依据：creepyeyes 要求生成点光照 ≤0.05（[光照](../../../raw/current/scripts/components/hallucinations.lua#L88-L89)）；shadowwatcher 找 60 单位内的火并取其周围暗处（[找火](../../../raw/current/scripts/components/hallucinations.lua#L116-L122)）；shadowskittish 避开女王火炬（[火炬](../../../raw/current/scripts/components/hallucinations.lua#L151-L153)）。

```lua
-- 组件自驱，无调用代码；模组增删幻觉类型的思路（服务端不适用，见限制）：
-- 修改 components/hallucinations.lua 的 HALLUCINATION_TYPES 表，spawnfn 形如：
spawnfn = function(inst, hallucination)
    local sanity = _player.replica.sanity:IsInsanityMode() and _player.replica.sanity:GetPercent() or 1
    -- 低于阈值、低于数量上限时 SpawnPrefab 并 StartTracking
end
```

- 调整与验证：理智读取用 replica（客户端组件）；生成节流靠 `interval+variance*random()`（[调度](../../../raw/current/scripts/components/hallucinations.lua#L54-L60)）。

## 常见陷阱与限制

- 该组件只存在于客户端/非 dedicated 世界实体：服务端改 `HALLUCINATION_TYPES` 不会生效，模组想替换需走客户端文件级覆盖并自担联机差异（[挂载端](../../../raw/current/scripts/prefabs/forest.lua#L628-L634)）。
- `shadowwatcher` 的找火依赖 `_fueltags`（由 FUELTYPE 生成，排除 USAGE，[燃料标签](../../../raw/current/scripts/components/hallucinations.lua#L228-L232)），新增燃料类型实体会自动纳入搜索。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，覆盖组件全文与 cave/forest 挂载上下文；未运行游戏。
- 结构检查：validate_page.py、validate_source_links.py 与 Lua 语法探针通过。
