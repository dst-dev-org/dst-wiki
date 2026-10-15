---
title: "projectedeffects"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "用侵蚀（erosion）着色器参数把宿主做浮现/消散过渡：全息投影与仪式特效的通用渐隐组件。"
aliases: ["投影特效", "全息渐隐", "erosion 渐变", "projectedeffects"]
tags: [component]
dst_build: "steam-buildid:26230496"
sources: ["components/projectedeffects.lua", "prefabs/inspectaclesbox.lua", "prefabs/winona_holoitems.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# projectedeffects

## 快速判断

- 职责与适用：对宿主 `AnimState` 施加 `SetErosionParams` 并在 0↔1 间补间，实现"投影成形/消散"；无网络变量，表现层组件（[核心](../../../raw/current/scripts/components/projectedeffects.lua#L76-L103)）。
- 优先使用：需要整体显隐过渡（含 cutoff/intensity 曲线）时用它，而不是逐帧手写 alpha。
- 重要副作用：alpha 到 0/1 时回调 `ondecaycallback`/`onconstructcallback` 并自动停更新；消散到底常被用来 `inst.Remove`（[回调](../../../raw/current/scripts/prefabs/inspectaclesbox.lua#L129)）。
- 修改边界：`Decay(permanent)` 的 permanent 标记会让后续 `Construct()` 直接跳过（[守卫](../../../raw/current/scripts/components/projectedeffects.lua#L49-L52)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `MakeOpaque()` | 无参数 | alpha 置 1 并立即 `SetErosionParams(0, cutoffheight, intensity)`（[实现](../../../raw/current/scripts/components/projectedeffects.lua#L16-L20)）。 |
| `Construct()` / `Decay(permanent)` | 无参/`boolean` | 设目标 alpha 并按需 `StartUpdatingComponent`；到端点触发对应回调（[实现](../../../raw/current/scripts/components/projectedeffects.lua#L49-L66)）。 |
| `SetDecayTime(duration)` / `SetConstructTime(duration)` | `number`，下限 0.01 | 过渡时长，秒（[限制](../../../raw/current/scripts/components/projectedeffects.lua#L22-L28)）。 |
| `SetCutoffHeight(cutoffheight)` | `number`，0 会被压成 -0.01 | 侵蚀截断高度，控制"从哪条线开始显形"（[实现](../../../raw/current/scripts/components/projectedeffects.lua#L30-L35)）。 |
| `SetIntensity(intensity)` | `number`，必须 ≤ -0.01 | 投影着色器强度，非负不生效（[守卫](../../../raw/current/scripts/components/projectedeffects.lua#L37-L39)）。 |
| `LockDecay(locked)` / `SetPaused(paused)` | `boolean` | locked 使 alpha=0 时不触发回调也不停更新；paused 冻结补间（[实现](../../../raw/current/scripts/components/projectedeffects.lua#L68-L79)）。 |
| `ondecaycallback` / `onconstructcallback` | `function(inst)` | 经 `SetOnDecayCallback`/`SetOnConstructCallback` 设置（[设置器](../../../raw/current/scripts/components/projectedeffects.lua#L41-L47)）。 |

## 典型接入

### 全息投影浮现与消散移除

- 前提与选择：宿主需 `AnimState`；安装前先 `SetErosionParams(1, 20, -1)` 完全隐藏首帧（[预置](../../../raw/current/scripts/prefabs/inspectaclesbox.lua#L122)）。
- 生效与撤销：安装后调 `Construct` 显形；消散结束经 `ondecaycallback` 移除实体，链路闭环。
- 原版依据：inspectaclesbox 全息盒按状态机在 Construct/Decay/MakeOpaque 间切换（[消费](../../../raw/current/scripts/prefabs/inspectaclesbox.lua#L124-L129)、[状态切换](../../../raw/current/scripts/prefabs/inspectaclesbox.lua#L153-L208)）；Winona 全息物消散 2 秒后移除（[消费](../../../raw/current/scripts/prefabs/winona_holoitems.lua#L30-L36)）。

```lua
inst.AnimState:SetErosionParams(1, 20, -1) -- 首帧完全隐藏
local fx = inst:AddComponent("projectedeffects")
fx:SetDecayTime(2)
fx:SetConstructTime(1)
fx:SetIntensity(-1.0)
fx:SetOnDecayCallback(inst.Remove)
fx:Construct() -- 1 秒内浮现；之后 Decay() 即 2 秒消散并移除
```

- 调整与验证：cutoffheight 控制显形起点高度（负值自下而上）；alpha=0 时组件强制用 20 的高度完全隐藏（[隐藏](../../../raw/current/scripts/components/projectedeffects.lua#L1)、[应用](../../../raw/current/scripts/components/projectedeffects.lua#L88)）。

## 常见陷阱与限制

- `intensity` 传正值会被静默钳到 -0.01 但着色器可能不启动：保持负值原版习惯。
- `LockDecay(true)` 期间 alpha 会停在 0 附近继续空转更新，忘记解锁会持续耗帧（[空转](../../../raw/current/scripts/components/projectedeffects.lua#L90-L96)）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，覆盖组件全文与 inspectaclesbox/winona_holoitems 消费链；未运行游戏。
- 结构检查：validate_page.py、validate_source_links.py 与 Lua 语法探针通过。
