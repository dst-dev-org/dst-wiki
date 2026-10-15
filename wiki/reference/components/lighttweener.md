---
title: "lighttweener"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "灯光参数补间：对实体 Light 的半径/强度/衰减/颜色做时长插值，结束应用目标值并回调，用于哨兵水晶开关灯等渐变。"
aliases: ["灯光补间", "lighttweener"]
tags: [component, callback]
dst_build: "steam-buildid:26230496"
sources: ["components/lighttweener.lua", "prefabs/deerclopseyeball_sentryward.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# lighttweener

## 快速判断

- 职责与适用：给带 Light 实体的实体做一次性灯光渐变：`StartTween` 记录当前值与目标值，组件更新期间逐通道 Lerp，到时 `EndTween` 应用目标值并触发回调（[实现](../../../raw/current/scripts/components/lighttweener.lua#L66-L103)）。
- 优先使用：开关灯渐变原版消费者是独眼鹿哨兵水晶（[deerclopseyeball_sentryward](../../../raw/current/scripts/prefabs/deerclopseyeball_sentryward.lua#L438)，开/关灯各一次 StartTween，[L320](../../../raw/current/scripts/prefabs/deerclopseyeball_sentryward.lua#L320)）；也能给动态生成的碎片实体临时挂用（lunarhailmanager.lua:220-221，该管理组件已标注弃用）。
- 重要副作用：`time=0` 时跳过插值立即 EndTween（[L98-L102](../../../raw/current/scripts/components/lighttweener.lua#L98-L102)）；组件更新由 `StartUpdatingComponent` 驱动，结束自动停。
- 修改边界：纯表现层、无存读钩子——补间中途存档重载会停在初始状态，需要宿主按状态重放 StartTween；组件不负责 Light 的 Enable 开关（原版由宿主处理，[L316-L318](../../../raw/current/scripts/prefabs/deerclopseyeball_sentryward.lua#L316-L318)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `StartTween(light, rad, intensity, falloff, colour, time, callback)` | `(Light, number, number, number, table, number, fn) -> void` | light 未设且从未设置时打印并放弃；初始值从 light 读取、读不到用目标值兜底；`colour` 为长度 ≥3 的数组；time>0 启动更新否则立即结束。[实现](../../../raw/current/scripts/components/lighttweener.lua#L66-L103) |
| `EndTween` | `-> void` | 全部通道直接设为目标值、停更新、`tweening=false` 并回调 `callback(inst, light)`。[实现](../../../raw/current/scripts/components/lighttweener.lua#L28-L57) |
| `OnUpdate(dt)` | 组件更新 | `t = timepassed/time`，对 radius/intensity/falloff/colour 逐通道 Lerp（未提供的通道跳过），完成后 EndTween。[实现](../../../raw/current/scripts/components/lighttweener.lua#L105-L137) |
| `tweening` | bool | 是否在补间中；StartTween 置 true、EndTween 置 false。[字段](../../../raw/current/scripts/components/lighttweener.lua#L24) |

## 典型接入

### 开关灯渐变

- 前提与选择：实体已有 Light（`inst.entity:AddLight()`）；双端表现时组件需双端添加（哨兵水晶在公共 postinit 挂组件，net 事件驱动双端 StartTween）。
- 生效与撤销：每次 StartTween 覆盖上一次补间；关灯渐变把目标半径/强度设 0 并在回调里 `Light:Enable(false)`（原版 TurnLightOffCallback，[L341](../../../raw/current/scripts/prefabs/deerclopseyeball_sentryward.lua#L341)）。

```lua
inst:AddComponent("lighttweener")
inst.components.lighttweener:StartTween(inst.Light, 4, 0.9, 0.5, {1, 0.9, 0.7}, 0.3, function(inst_)
    inst_.Light:Enable(false)
end)
```

- 调整与验证：`colour` 用 `{r, g, b}`（<3 个元素视为未提供，[L59-L64](../../../raw/current/scripts/components/lighttweener.lua#L59-L64)）；`nil` 的参数通道不参与插值，适合只改半径的呼吸效果。

## 常见陷阱与限制

- **无持久化与恢复**：补间状态（timepassed/初值）不落盘，宿主要在 OnLoad 后按当前状态重新 StartTween 一次收尾。
- **回调在 EndTween 内同步触发**：time=0 时回调在 StartTween 调用栈里直接执行，回调里再 StartTween 要注意顺序。
- **light 换绑无清理**：组件保存的 light 引用直到下次 StartTween 才更新；宿主替换 Light 实例后必须重新 StartTween，否则改的是旧灯。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；组件全文、deerclopseyeball_sentryward 挂接与开关灯调用、lunarhailmanager 用点（grep 核验）。
- 游戏运行：未运行。
