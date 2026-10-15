---
title: "abysspillargroup"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "深渊柱阵管理：为试炼实体集中生成、追踪、坍塌一批石柱，记录坍塌点位供整组复活与存读。"
aliases: ["深渊石柱组", "abysspillargroup", "柱阵管理"]
tags: [component, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/abysspillargroup.lua", "prefabs/abysspillar_trial.lua", "prefabs/lightsout_trial.lua", "prefabs/abysspillar.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# abysspillargroup

## 快速判断

- 职责与适用：挂在试炼管理实体上（如 `abysspillar_trial`、`lightsout_trial`），以回调注入的方式生成/坍塌柱子并统一跟踪，柱子侧经 `_abysspillargroup` 反查所属组。
- 优先使用：整组柱子的生命周期操作（生成、坍塌、复活、清除）全部经本组件，不要单独管理柱子实体。
- 重要副作用：柱子 `onremove` 触发时自动记下 XZ 点位并回调（[移除](../../../raw/current/scripts/components/abysspillargroup.lua#L26-L32)），`RespawnAllPillars` 在这些点位重生。
- 修改边界：生成/坍塌本体由 `SetSpawnAtXZFn`/`SetCollapseFn` 注入，组件只做编排与存读。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SetSpawnAtXZFn(fn)` / `SetCollapseFn(fn)` | `fn(inst, x, z, instant)` / `fn(inst, pillar)` | 注入生成与坍塌实现，返回柱实体（[注册](../../../raw/current/scripts/components/abysspillargroup.lua#L35-L41)）。 |
| `SetOnAddPillarFn(fn)` / `SetOnRemovePillarFn(fn)` | `fn(inst, pillar)` | 柱子进/出跟踪时的回调（[注册](../../../raw/current/scripts/components/abysspillargroup.lua#L43-L49)）。 |
| `StartTrackingPillar(pillar)` / `StopTrackingPillar(pillar)` | 柱实体 | 建/除断言过的 `_abysspillargroup` 反向引用并监听 `onremove`（[跟踪](../../../raw/current/scripts/components/abysspillargroup.lua#L51-L73)）。 |
| `SpawnPillarAtXZ(x, z, instant)` | 仅服务器语义 | 调 spawnfn 生成并纳入跟踪（[实现](../../../raw/current/scripts/components/abysspillargroup.lua#L75-L80)）。 |
| `AddPillarSpawnPointXZ(x, z)` / `RespawnAllPillars()` | 点位表 | 积累点位并在调用时全部重生（[实现](../../../raw/current/scripts/components/abysspillargroup.lua#L82-L90)）。 |
| `CollapseAllPillars()` / `Clear()` | 无参数 | 全部坍塌（走 collapsefn）／清空点位并删除所有柱子（[实现](../../../raw/current/scripts/components/abysspillargroup.lua#L92-L108)）。 |
| `HasPillars()` / `HasSpawnPoints()` | → `bool` | 试炼门控的常用查询（[实现](../../../raw/current/scripts/components/abysspillargroup.lua#L110-L116)）。 |
| `OnSave` / `OnLoad` / `LoadPostPass` | 存读钩子 | 存点位与柱 GUID；读档恢复点位、`LoadPostPass` 重新跟踪柱子（[存读](../../../raw/current/scripts/components/abysspillargroup.lua#L118-L145)）。 |

## 典型接入

### 重组一组可坍塌石柱

- 前提与选择：服务器侧；先注入 spawn/collapse 回调再生成，柱子 prefab 需接受被外部 `Remove`。
- 生效与撤销：柱子被移除即进点位表；`Clear` 是彻底撤销，`RespawnAllPillars` 只消耗点位表。
- 原版依据：`abysspillar_trial` 注册全部四个回调（[注册](../../../raw/current/scripts/prefabs/abysspillar_trial.lua#L513-L517)）；`lightsout_trial` 用点位表布阵（[布阵](../../../raw/current/scripts/prefabs/lightsout_trial.lua#L92-L94)）。

```lua
inst:AddComponent("abysspillargroup")
inst.components.abysspillargroup:SetSpawnAtXZFn(function(inst, x, z, instant)
    local pillar = SpawnPrefab("my_pillar")
    pillar.Transform:SetPosition(x, 0, z)
    return pillar
end)
inst.components.abysspillargroup:SetCollapseFn(function(inst, pillar)
    pillar:PushEvent("collapse") -- 柱子内部处理坍塌表现
end)
inst.components.abysspillargroup:AddPillarSpawnPointXZ(10, 20)
inst.components.abysspillargroup:RespawnAllPillars()
```

- 调整与验证：柱子换位后原版以"停跟踪旧柱、跟踪新柱"保持组内唯一（[换位](../../../raw/current/scripts/prefabs/abysspillar.lua#L772-L773)）。

## 常见陷阱与限制

- 柱子必须经组件跟踪：柱子侧逻辑依赖 `_abysspillargroup` 反查试炼（[反查](../../../raw/current/scripts/prefabs/abysspillar.lua#L293-L297)），绕过 `StartTrackingPillar` 直接 SpawnPrefab 会让柱子与试炼脱钩。
- `OnLoad` 直接取 `data.pts`（[读取](../../../raw/current/scripts/components/abysspillargroup.lua#L132-L134)）；`OnSave` 无点位且无柱时返回 nil，引擎不会带 data 调 `OnLoad`，属正常路径。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，覆盖组件全文与 abysspillar_trial、lightsout_trial、abysspillar 三处消费链；未运行游戏。
- 结构检查：validate_page.py、validate_source_links.py 与 Lua 语法探针通过。
