---
title: "focalpoint"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "客户端相机焦点组件：按优先级与距离在多个焦点源间选择并偏移跟随相机，供过场与特写镜头使用。"
aliases: ["相机焦点", "镜头聚焦", "特写镜头", "focalpoint"]
tags: [component, callback]
dst_build: "steam-buildid:26230496"
size_class: standard
sources: ["components/focalpoint.lua", "prefabs/focalpoint.lua", "cameras/followcamera.lua", "components/charliecutscene.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# focalpoint

## 快速判断

- 职责与适用：仅客户端（源码首注 components/focalpoint.lua:1-2）。单例 `TheFocalPoint`（main.lua:330）在玩家激活时挂到玩家 entity 下（prefabs/focalpoint.lua:22-25），followcamera 每帧调 `CameraUpdate(dt)`（cameras/followcamera.lua:431-432）。
- 优先使用：临时特写→`StartFocusSource(...)` 成对 `StopFocusSource`；需要完整相机行为→传 updater 表（协议见下）。
- 重要副作用：焦点生效期间直接改 `TheCamera` 偏移（默认每帧加 y+1.5，:95-103）；焦点更替或结束时 `Reset` 回默认相机（:11-18）。
- 修改边界：source 实体移除自动清其全部条目（onremove 监听，:7、:28）；`PushTempFocus` 已废弃只打日志（:77-80）。

## 关键接口

| 接口/字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `StartFocusSource` | `(source, id, target, minrange, maxrange, priority, updater?, offset?, redirecttarget?) -> nil` | 同 source+id 重复调用改参数；`target` 缺省用 source；完成后立即 `CameraUpdate(0)`。:23-46 |
| `StopFocusSource` | `(source, id?) -> nil` | id 省略清该 source 全部条目；清的是当前焦点时 `Reset(true)` 不瞬移。:48-66 |
| `CameraUpdate` | `(dt) -> nil` | 选优先级最高、同优先级取最近且在 maxrange 内的焦点；`updater.IsEnabled` 可否决；失效 target 进清理队列。:105-153 |
| `RemoveAllFocusSources` | `(no_snap?) -> nil` | 清空全部并 Reset。:68-75 |
| `FocalPoint_CalcBaseOffset` | `global (dt, params, parent, dist_sq) -> Vector3` | 距离在 min..max 间线性插值的基础偏移；可复用做自定义 updater（消费例 components/charliecutscene.lua:42）。:83-93 |
| updater 协议 | `{IsEnabled?, ActiveFn?, UpdateFn?}` | `IsEnabled(parent, params, source)` 门控、`ActiveFn(params, parent, dist_sq)` 首次激活、`UpdateFn(dt, params, parent, dist_sq)` 每帧；缺省用内置 UpdateFocus。:117、:141-146 |

## 典型接入

### 客户端临时聚焦某目标

- 前提与选择：仅客户端 mod 逻辑；参照 charliecutscene.lua:70-72 的成对调用模式。
- 生效与撤销：source 建议传宿主实体，实体移除自动清理；id 用于同 source 多路焦点。

```lua
-- 客户端：把相机轻微拉向 target，结束后恢复
local function FocusCamera(source, target)
    TheFocalPoint.components.focalpoint:StartFocusSource(
        source, "my_focus", target, 5, 12, 4)
end

local function UnfocusCamera(source)
    TheFocalPoint.components.focalpoint:StopFocusSource(source, "my_focus")
end
```

- 调整与验证：minrange/maxrange 是距离插值区间（超 maxrange 无偏移）；priority 数字越大越优先。

## 常见陷阱与限制

- `CameraUpdate` 清理失效 target 的调用传参与意图不符（`unpack(toremove)` 把条目表当 source 传入，:127-129），该路径实际不会移除失效条目，依赖 source 的 onremove 兜底；mod 侧应自行 `StopFocusSource`。
- 焦点切换时对旧焦点只 `Reset(true)`，无"结束"回调语义（:132-139 注释）；需要收尾时在自定义 updater 外自行管理。
- `Reset` 直接 `TheCamera:SetDefault()`（:14），源码注释承认与 player_classified 相机设置存在冲突（:13）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，通读组件全部 167 行与 prefabs/focalpoint.lua；核验驱动点（followcamera.lua:431-432）、单例创建（main.lua:330）与 updater 消费例（charliecutscene.lua:42、:70-72）。
- 游戏运行：未运行。
