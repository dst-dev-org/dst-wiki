---
title: "debugger"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "实体级线段调试渲染器：按 key 记录端点与颜色，每帧 Flush 画线；当前版本无任何挂载者，唯一引用是 knownlocations 内的注释代码。"
aliases: ["debugger", "调试画线", "DebugRender 调试器"]
tags: [component]
size_class: simple
dst_build: "steam-buildid:26230496"
sources: ["components/debugger.lua", "components/knownlocations.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# debugger

## 快速判断

- 职责与适用：给实体加 `DebugRender` 引擎句柄，维护 `debuggerdraws[key] = {p1, p2, c}` 线段表并每帧重绘；用于开发期可视化两点连线（如位置关系、路径）。[绘制循环](../../../raw/current/scripts/components/debugger.lua#L69-L81)
- 优先使用：**当前快照内无活跃消费者**——全库 grep 无 `AddComponent("debugger")`、无 `components.debugger` 有效引用；`knownlocations.lua:7-17` 的消费代码整体被注释（按 6 色循环画已知位置连线），仅能说明设计意图。[注释消费](../../../raw/current/scripts/components/knownlocations.lua#L7-L17)
- 重要副作用：构造即 `StartUpdatingComponent` 常驻每帧 Flush；`SetRenderLoop(true)` 开启渲染循环。[构造](../../../raw/current/scripts/components/debugger.lua#L17-L19)
- 修改边界：`SetZ(val)` 有明显笔误——忽略参数、总是写回构造默认 `self.z = 0.5`；调试绘制仅本地可见，不影响网络与存档。[SetZ 笔误](../../../raw/current/scripts/components/debugger.lua#L65-L67)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SetOrigin(key, x, y)` / `SetTarget(key, x, y)` | key: 任意字符串 | 分别写线段起点/终点；同 key 复用同一线段。[实现](../../../raw/current/scripts/components/debugger.lua#L22-L34) |
| `SetColour(key, r, g, b, a)` | 四个 number（0–1） | 写线段颜色；未设置时 OnUpdate 用默认绿色 `{r=0,g=1,b=0,a=1}`。[默认色](../../../raw/current/scripts/components/debugger.lua#L73) |
| `SetAll(key, origin, tar, colour)` | origin/tar: `{x,y}` 或 `{x,z}` 表；colour 同色表 | 一次写全套；含 z 的表自动按 x/z 平面取值（俯视坐标直传）。三个参数均可传 nil 跳过。[实现](../../../raw/current/scripts/components/debugger.lua#L43-L63) |
| `debuggerdraws` | table：`key -> {p1, p2, c}` | 线段注册表，OnUpdate 全量重绘；p1 缺省 (0,0)、p2 缺省 (100,100)。[绘制](../../../raw/current/scripts/components/debugger.lua#L72-L79) |
| `SetZ(val)` | `-> nil` | 引擎句柄 z 值设置；参数被忽略（恒 `self.debugger:SetZ(self.z)`，0.5）。[实现](../../../raw/current/scripts/components/debugger.lua#L65-L67) |

## 典型接入

### 开发期画线（按设计意图还原）

- 前提与选择：任意实体可挂；因无原版活跃消费者，以下用法按注释代码与组件逻辑推得，需自行游戏验证。注释代码的用法是按 key 重写端点实现跟踪线（含按 6 色循环的配色示例）。[注释用法](../../../raw/current/scripts/components/knownlocations.lua#L16-L17)
- 生效与撤销：线段随 key 增改、组件移除随实体消失；无生命周期清理接口，不再需要时须自行清 `debuggerdraws`。

```lua
-- 开发期可视化（未接线组件；自行验证）
inst:AddComponent("debugger")
local dbg = inst.components.debugger
dbg:SetColour("myline", 1, 0, 0, 1)
dbg:SetAll("myline", inst:GetPosition(), target:GetPosition(), nil)
-- 每帧更新端点即可保持跟踪
```

- 调整与验证：绘制 z 固定 0.5（`SetZ` 参数无效），贴地线建议直接传世界坐标的 x/z 分量。

## 常见陷阱与限制

- **别指望 key 生命周期管理：** 组件没有 Remove/Clear 接口，`debuggerdraws` 只增不减（注释里的示例代码也是覆盖式写）；长期挂载会累积线段。[无清理](../../../raw/current/scripts/components/debugger.lua#L22-L41)
- **SetZ 参数无效：** 想改绘制高度需直接改 `self.z` 后调 `SetZ()`（无参），或直接操作 `inst.entity:AddDebugRender()` 返回的句柄；原样传参会被忽略。[笔误位置](../../../raw/current/scripts/components/debugger.lua#L65-L67)
- **knownlocations 的消费是注释：** 引用任务提示"knownlocations.lua 内消费"，实际该段（含 AddComponent 与 SetAll 调用）全部处于 `--` 注释状态，不构成运行时依赖；验证结论时勿引用为活跃消费者。[注释证据](../../../raw/current/scripts/components/knownlocations.lua#L7-L17)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（83 行）；全库 grep（`components.debugger`、`AddComponent("debugger"`、`"debugger"`）确认无活跃消费者，仅 `knownlocations.lua:7-17` 注释引用；`DEBUGGER_ENABLED` 等引擎调试器为另一体系（`main.lua:88`）。
- 结构检查：2026-10-28 validate_page.py 通过；validate_source_links.py --strict 锚点通过（见批级记录）。
- 游戏运行：未运行。
