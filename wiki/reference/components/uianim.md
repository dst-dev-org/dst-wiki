---
title: "uianim"
created: "2026-10-28"
updated: "2026-10-28"
type: component
size_class: simple
summary: "UI widget 补间动画组件：为控件提供缩放、位移、旋转、着色四类 outCubic 补间，挂墙钟更新不随服务器暂停。"
aliases: ["UI 动画", "uianim", "界面补间"]
tags: [component]
dst_build: "steam-buildid:26230496"
sources: ["components/uianim.lua", "widgets/widget.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# uianim

## 快速判断

- 职责与适用：给 UI 实体（widget）做 `ScaleTo`/`MoveTo`/`RotateTo`/`TintTo` 补间，全部 `easing.outCubic`、墙钟驱动（`StartWallUpdatingComponent`，[实现](../../../raw/current/scripts/components/uianim.lua#L148-L233)）；纯客户端 UI 用途。
- 优先使用：控件上直接调 `Widget:MoveTo` 等封装（[封装](../../../raw/current/scripts/widgets/widget.lua#L167-L209)）；Widget 构造时已自动 `AddComponent("uianim")`（[自动挂载](../../../raw/current/scripts/widgets/widget.lua#L20)）。
- 重要副作用：`update_while_paused` 默认 true，服务器暂停时仍走 `OnWallUpdate`（[分支](../../../raw/current/scripts/components/uianim.lua#L154)）；每类补间各自保存 start/dest/duration 状态字段。
- 修改边界：`TintTo` 要求控件实现 `SetTint`，否则直接返回（[守卫](../../../raw/current/scripts/components/uianim.lua#L40-L42)）；全部补间完成即自动 `StopWallUpdatingComponent`（[停止](../../../raw/current/scripts/components/uianim.lua#L230-L232)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `TintTo(start, dest, duration, whendone)` | start/dest 为含 r,g,b,a 的表 | 每帧 `widget:SetTint`，结束触发 whendone（[实现](../../../raw/current/scripts/components/uianim.lua#L39-L51)）。 |
| `ScaleTo(start, dest, duration, whendone)` | 数值 | 保留当前各轴符号（负缩放不翻转，[实现](../../../raw/current/scripts/components/uianim.lua#L81-L93)）；重复调用先 `FinishCurrentScale`。 |
| `MoveTo(start, dest, duration, whendone)` | dest 为坐标表 | 置 `UITransform` 位置；旧 `pos_whendone` 会被立即调用（[实现](../../../raw/current/scripts/components/uianim.lua#L103-L117)）。 |
| `RotateTo(start, dest, duration, whendone, infinite)` | 数值；infinite 可选 | infinite 时按帧累加 `rot_dest` 无限旋转（[实现](../../../raw/current/scripts/components/uianim.lua#L127-L145)）。 |
| `CancelTintTo/CancelScaleTo/CancelMoveTo/CancelRotateTo(run_complete_fn)` | → `nil` | 取消进行中补间；传 true 且有挂起回调时执行之（[实现](../../../raw/current/scripts/components/uianim.lua#L31-L37)）。 |
| `UpdateWhilePaused(bool)` | → `nil` | 切换暂停时是否继续动画，默认 true（[实现](../../../raw/current/scripts/components/uianim.lua#L9-L11)）。 |

## 典型接入

### 滑入 / 弹出控件

- 前提与选择：屏幕类代码持有的是 Widget 实例，经其转发调用即可，不需要手动挂组件。
- 生效与撤销：补间立即开始；结束时回调 `whendone` 并停止更新。
- 原版依据：乌鸦小游戏屏幕用 `MoveTo` 移动 UI 并覆写 `OnWallUpdate`（[参照](../../../raw/current/scripts/screens/crowgamescreen.lua#L180-L185)）；交易界面直接置 `pos_t = nil` 强停位移（[参照](../../../raw/current/scripts/screens/tradescreen.lua#L1212)）。

```lua
-- self 是某个 Widget
self.inst.components.uianim:MoveTo(startpos, destpos, 0.3, function() print("done") end)
-- 需要提前终止并触发回调时：
self.inst.components.uianim:CancelMoveTo(true)
```

- 调整与验证：duration 单位为秒（墙钟）；`itemtile.lua` 还直接读 `pos_t > 0` 判断"位移进行中"（[参照](../../../raw/current/scripts/widgets/itemtile.lua#L248)），模组可复用同一状态字段。

## 常见陷阱与限制

- `UIAnim` 类并未定义 `ForceStartWallUpdating`/`ForceStopWallUpdating`，而 `Widget:ForceStartWallUpdating` 会转发调用（[封装](../../../raw/current/scripts/widgets/widget.lua#L212-L230)），frontend.lua 保存指示器走该路径（[调用点](../../../raw/current/scripts/frontend.lua#L261)）；触发时是对 nil 方法的调用错误，复用这两个封装前需自行核验。
- `MoveTo` 重复调用会立即触发上一次的 `pos_whendone` 再替换（[顺序](../../../raw/current/scripts/components/uianim.lua#L109-L112)），不要把首次完成回调当作"新动画完成"。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，覆盖组件全文与 widget.lua 封装、crowgamescreen/tradescreen/itemtile 调用点；未运行游戏，ForceStartWallUpdating 路径未实测。
- 结构检查：validate_page.py、validate_source_links.py 与 Lua 语法探针通过。
