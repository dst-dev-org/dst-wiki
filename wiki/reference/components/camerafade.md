---
title: "camerafade"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "镜头遮挡淡化：按实体与摄像机距离、可选中心符号屏幕距离与高度，逐帧计算 alpha 并 OverrideMultColour，让贴近镜头的实体半透明。"
aliases: ["镜头淡化", "camerafade", "靠近镜头透明"]
tags: [component, callback]
dst_build: "steam-buildid:26230496"
sources: ["components/camerafade.lua", "prefabs/floatinglantern.lua", "prefabs/floatinglanternfire.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# camerafade

## 快速判断

- 职责与适用：挂在会遮挡镜头的实体（原版 [floatinglantern](../../../raw/current/scripts/prefabs/floatinglantern.lua#L27-L41)）；构造即 `Enable(true)` 并用 WallUpdate 逐帧把 alpha 经 `AnimState:OverrideMultColour(1,1,1,alpha)` 应用到实体动画（[实现](../../../raw/current/scripts/components/camerafade.lua#L82-L139)）。
- 优先使用：需要"挡镜头时变半透明"的自定义实体（灯笼、大型飞行物）按原版组合调 `SetUp` + `SetUpCenterFade` + `SetLerpToHeight`。
- 重要副作用：主机上无玩家或玩家距离 ≥30（距离平方 ≥900）时恒不透明并按需停止更新（[L85-L93](../../../raw/current/scripts/components/camerafade.lua#L85-L93)）；实体睡眠停更、唤醒恢复（[L64-L80](../../../raw/current/scripts/components/camerafade.lua#L64-L80)）。
- 修改边界：客户端可见性由宿主控制——原版用 net_bool `enablecamerafade` 的 dirty 事件双端驱动组件的加/移除与 Enable（[驱动](../../../raw/current/scripts/prefabs/floatinglantern.lua#L27-L41)）；组件自身无网络变量、无存读钩子。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `Enable(enable, instant)` | `(bool, 可选 bool) -> void` | 开启后实体醒着即启动 WallUpdate；关闭仅停止接受淡化，alpha 回 1 后才停更，`instant=true` 立即复位颜色并停。[实现](../../../raw/current/scripts/components/camerafade.lua#L26-L42) |
| `SetUp(range, fadetodist)` | `(number, number) -> void` | 俯视点积距离在 `fadetodist..range` 间线性淡化，默认 25/5。[实现](../../../raw/current/scripts/components/camerafade.lua#L44-L47) |
| `SetUpCenterFade(symbol, min_dist_sq, dist_sq, min_fade)` | `(string, number, number, number) -> void` | 以 AnimState 符号与镜头中心的屏幕距离做第二重淡化，按 `RESOLUTION_X/Y` 缩放（原版 150²/350²、下限 0.1）。[实现](../../../raw/current/scripts/components/camerafade.lua#L49-L54) |
| `SetLerpToHeight(height)` | `number -> void` | 实体越高于该值淡化越弱（按 y/height 插值）。[实现](../../../raw/current/scripts/components/camerafade.lua#L56-L58) |
| `GetCurrentAlpha` | `-> number(0..1)` | 当前 alpha；原版灯笼火焰用它同步透明度（[消费](../../../raw/current/scripts/prefabs/floatinglanternfire.lua#L27-L31)）。[实现](../../../raw/current/scripts/components/camerafade.lua#L60-L62) |
| `OnWallUpdate(dt)` | WallUpdate | 综合三重系数：中心符号屏幕距离、摄像机俯视距离、高度插值；alpha 平滑（差 <1/255 直接取值，否则 0.95/0.05 混合）。[实现](../../../raw/current/scripts/components/camerafade.lua#L82-L139) |

## 典型接入

### 给悬浮实体加镜头淡化

- 前提与选择：组件加在实体公共段（双端各算各的 alpha）；宿主用 net 变量决定启用时机（原版 [floatinglantern.lua](../../../raw/current/scripts/prefabs/floatinglantern.lua#L29-L41) 的写法可照抄）。

```lua
-- 客户端/非专用端：配置镜头淡化参数
inst:AddComponent("camerafade")
inst.components.camerafade:SetUp(10, 5)
inst.components.camerafade:SetUpCenterFade("glow", 150 * 150, 350 * 350, 0.1)
inst.components.camerafade:SetLerpToHeight(4)
```

- 调整与验证：淡化结果只影响 AnimState（OverrideMultColour 是覆盖式），实体上的子实体/火焰要自己读 `GetCurrentAlpha` 同步（见 floatinglanternfire 的 OnPostUpdate）。

## 常见陷阱与限制

- **构造即启用**：不调用任何配置也会有默认淡化（range=25/fadetodist=5，[L6-L7](../../../raw/current/scripts/components/camerafade.lua#L6-L7)），只想按需启用要用 `Enable(false, true)` 先关。
- **中心淡化依赖符号存在**：`GetSymbolPosition` 失败时该重系数按 1 处理（[L104-L119](../../../raw/current/scripts/components/camerafade.lua#L104-L119)），动画隐藏帧期间中心淡化会暂时失效。
- **非 instant 关闭有尾延迟**：`Enable(false)` 后组件会继续更新到 alpha 回 1（[L135-L138](../../../raw/current/scripts/components/camerafade.lua#L135-L138)），移除实体前若要立刻复位需 `Enable(false, true)`（OnRemoveFromEntity 即此做法）。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；组件全文、floatinglantern 驱动与参数、floatinglanternfire 的 alpha 消费点。
- 游戏运行：未运行。
