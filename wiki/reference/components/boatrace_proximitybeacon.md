---
title: "boatrace_proximitybeacon"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "船赛检查点上的接近信标：带 tag 供 checker 搜索，并转发 boatrace_start／boatrace_finish 事件给宿主回调。"
aliases: ["船赛接近信标", "船赛检查点信标", "boatrace_proximitybeacon"]
tags: [component, callback]
dst_build: "steam-buildid:26230496"
size_class: simple
sources: ["components/boatrace_proximitybeacon.lua", "components/boatrace_proximitychecker.lua", "prefabs/boatrace_checkpoint_indicator.lua", "prefabs/boatrace_start.lua", "prefabs/boatrace_primemate.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# boatrace_proximitybeacon

## 快速判断

- 职责与适用：组件头注释自述"可配置在 boatrace_proximitychecker 旁边，用于标识信标进入 checker 范围"（[L1-L2](../../../raw/current/scripts/components/boatrace_proximitybeacon.lua#L1-L2)）。接近判定本身在 checker 侧（用 `boatrace_proximitybeacon` tag 做 FindEntities 过滤，[checker](../../../raw/current/scripts/components/boatrace_proximitychecker.lua#L8)）；本组件还负责把船赛开始／结束事件转成宿主回调。
- 优先使用：宿主加组件后 `SetBoatraceStartedFn`／`SetBoatraceFinishedFn` 接事件；需要"进入范围"语义时搭配 boatrace_proximitychecker 使用。
- 重要副作用：构造加 `boatrace_proximitybeacon` tag 并挂两个事件监听，`OnRemoveFromEntity` 摘除（[构造](../../../raw/current/scripts/components/boatrace_proximitybeacon.lua#L3-L23)、[清理](../../../raw/current/scripts/components/boatrace_proximitybeacon.lua#L25-L30)）。
- 修改边界：事件由 `boatrace_start` prefab 推给 `_beacons` 集合（即带本组件的检查点指示器）；组件不广播、不存档（无 OnSave／OnLoad），原版宿主 `boatrace_checkpoint_indicator` 本身 `persists = false`（[宿主](../../../raw/current/scripts/prefabs/boatrace_checkpoint_indicator.lua#L197-L201)）。

## 关键接口

| 接口／字段 | 已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SetBoatraceStartedFn(fn)` | `fn(inst, data)` 或 nil | 宿主实体收到 `boatrace_start` 事件时调用，data 为发起比赛的起点实体。[L32-L34](../../../raw/current/scripts/components/boatrace_proximitybeacon.lua#L32-L34)、[推事件](../../../raw/current/scripts/prefabs/boatrace_start.lua#L445) |
| `SetBoatraceFinishedFn(fn)` | `fn(inst, data)` 或 nil | 收到 `boatrace_finish` 时调用；组件取 `data.start`／`data.winner` 传入回调。[L36-L38](../../../raw/current/scripts/components/boatrace_proximitybeacon.lua#L36-L38)、[L17-L22](../../../raw/current/scripts/components/boatrace_proximitybeacon.lua#L17-L22)、[finish 数据](../../../raw/current/scripts/prefabs/boatrace_start.lua#L141-L149) |
| （tag）`boatrace_proximitybeacon` | 构造添加 | checker 与 primemate 的实体搜索 musttags。[checker](../../../raw/current/scripts/components/boatrace_proximitychecker.lua#L8)、[primemate](../../../raw/current/scripts/prefabs/boatrace_primemate.lua#L67) |

## 典型接入

### 原版检查点指示器

- 前提与选择：`boatrace_checkpoint_indicator` 是非持久实体，构建时加组件并立即设置两个回调（[L199-L201](../../../raw/current/scripts/prefabs/boatrace_checkpoint_indicator.lua#L199-L201)）；比赛发起方 `boatrace_start` 收集周围带 tag 的信标，开赛与结算时逐个推事件。
- 生效与撤销：比赛开始 → `boatrace_start` 事件（data=起点）；比赛结束 → `boatrace_finish`（data=`{start, winner}`，[L141-L149](../../../raw/current/scripts/prefabs/boatrace_start.lua#L141-L149)）；指示器随宿主移除时 `OnRemoveFromEntity` 摘监听。

## 常见陷阱与限制

- `boatrace_finish` 有两种推送：给 `_beacons` 的带 finish_data，给 `_checkpoints` 的不带 data（[boatrace_start.lua#L148](../../../raw/current/scripts/prefabs/boatrace_start.lua#L148)、[L156](../../../raw/current/scripts/prefabs/boatrace_start.lua#L156)）。而组件回调固定索引 `data.start`／`data.winner`（[L19](../../../raw/current/scripts/components/boatrace_proximitybeacon.lua#L19)）：宿主若同时出现在两个集合里，收到无 data 事件会直接报错；自定义宿主监听该事件时应做 nil 防御。
- 接近判定不在本组件：只有 tag 参与 FindEntities，"进入范围"的时机、半径都由 checker／boatrace_start 侧定义；把组件挂到移动实体上不会被自动追踪。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。完整阅读 components/boatrace_proximitybeacon.lua（40 行），核对 checker／primemate 的 tag 消费、checkpoint_indicator 宿主设置与 boatrace_start 的两处事件推送。未覆盖船赛计圈与名次逻辑。
- 游戏运行：未运行。
- 结构检查：validate_page.py 通过（0 issue；50 行、5100 字节，simple 档内）；validate_source_links.py --strict 通过（17 处源码引用，0 error／0 warning）；本页无完整 Lua 代码块。
