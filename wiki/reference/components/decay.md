---
title: decay
created: "2026-10-23"
updated: "2026-10-28"
type: component
summary: "识别旧式数值衰减线程及其 num 分支疑点，避免将它当成可直接复用的食物腐败或燃料组件。"
aliases: ["decay", "SetTimeDelta", "旧式数值衰减", "spentfuel", "addfuel"]
tags: [component, lifecycle, event]
dst_build: "steam-buildid:26230496"
sources: ["components/decay.lua", "scheduler.lua", "entityscript.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# decay

## 快速判断

- 职责与适用：保存一个独立的 currenthealth 数值，支持手动变化与线程周期变化，并发送 `spentfuel`／`addfuel` 事件；本页用于识别这份实现，不把字段名等同于 [[reference/components/health]] 的人物生命。[构造:1](../../../raw/current/scripts/components/decay.lua#L1)、[变化:8](../../../raw/current/scripts/components/decay.lua#L8)
- 优先使用：先按真实需求查 [[reference/components/fueled]]、[[reference/components/perishable]] 或 [[reference/components/timer]]；不要因“衰减”一词直接挂本组件。当前快照未定位到定义外的明确消费者，见下方搜索边界。
- 重要副作用：从正值降至非正时发 spentfuel 并归零；超过 maxhealth 时发 addfuel，却不裁到上限。[阈值与事件:14](../../../raw/current/scripts/components/decay.lua#L14)
- 修改边界：`SetTimeDelta` 的 num 真值分支读取未在该函数作用域定义的 `self.decayrate`，存在明确源码疑点。没有游戏运行核验，不提供带 num 的可用计时示例。[循环:30](../../../raw/current/scripts/components/decay.lua#L30)

## 关键接口

| 接口／字段 | 已核验约定 | 条件、作用与证据 |
|---|---|---|
| `maxhealth`、`currenthealth`、`decayrate` | 默认 100、100、1 | 普通 Lua 字段；组件没有配置 setter、replica 或 netvar。decayrate 只在下述有疑点的 num 分支被引用。[构造:1](../../../raw/current/scripts/components/decay.lua#L1)、[分支:35](../../../raw/current/scripts/components/decay.lua#L35) |
| `DoDelta(amount)` | amount 为数值，无返回值 | 加到 currenthealth；无伤害吸收、死亡处理或 delta 参数事件。旧值正且新值非正时 PushEvent(spentfuel)，其后 currenthealth=0。[实现:8](../../../raw/current/scripts/components/decay.lua#L8) |
| `SetTimeDelta(amount, pause, num)` | pause 必须可与 0 比较；num 可省略 | 先 KillThread 旧句柄，仅 pause>0 才新建线程；线程绑定宿主 GUID。每个未 paused 的循环先 DoDelta，再处理 num，最后 Sleep(pause)。[入口:49](../../../raw/current/scripts/components/decay.lua#L49)、[循环:30](../../../raw/current/scripts/components/decay.lua#L30) |
| `paused` | 默认未赋值，即 nil；真值暂停变化 | 仅跳过 DoDelta／num，循环仍 Sleep；没有 Pause／Resume 方法，不是暂停并保存剩余等待时间。[判断:32](../../../raw/current/scripts/components/decay.lua#L32)、[等待:43](../../../raw/current/scripts/components/decay.lua#L43) |
| `deltatask` | 保存 StartThread 返回的任务 | 新线程正常返回后置 nil；仅杀旧线程而不新建时不清句柄。StartThread／KillThread 由调度器实现。[入口:51](../../../raw/current/scripts/components/decay.lua#L51)、[StartThread:387](../../../raw/current/scripts/scheduler.lua#L387)、[KillThread:366](../../../raw/current/scripts/scheduler.lua#L366) |

## 典型接入

### 识别旧脚本，而非生成未经验证的衰减骨架

本页不提供独立计时 demo。若模组已有 `components.decay`，先确认它是否真指当前原版模块、是否覆写了线程逻辑、是否有事件监听与生命周期清理；不能把源码中的疑点自行“修正”后称作原版行为。

在当前 `raw/current/scripts/` 的 `.lua` 内定向搜索双／单引号 `AddComponent("decay")`、`components.decay` 与 `SetTimeDelta(`：未发现定义外的这些字面调用。搜索到的 timer 名称 `"decay"`、动画 `"decay"` 不足以证明挂载本组件；这不排除动态字符串挂载、额外加载路径或外部模组使用。

组件本身没有端判定、添加依赖、存档接口或移组件清理；如确需兼容，应先在目标模组核验实际宿主与服务端调用，而不能由这份孤立定义推导它已有客户端同步和存档恢复。[完整构造与接口起点:1](../../../raw/current/scripts/components/decay.lua#L1)、[最终接口:49](../../../raw/current/scripts/components/decay.lua#L49)

## 常见陷阱与限制

- **传 num 限定次数**：局部函数参数名是 `health`，但扣次数写成 `num = num - self.decayrate`；没有局部 self 定义或形参，常规环境中将访问错误。它发生在一次 DoDelta 之后，不能假定出错时尚未改变数值。`num=0` 在 Lua 也是真值，不能绕开该分支。[delta:30](../../../raw/current/scripts/components/decay.lua#L30)
- **设置 decayrate 就能按速率衰减**：每次变化只使用传入的 amount；decayrate 不用于 amount 或等待时间。即使避开 num，也不能把它解读为每秒自动扣除的速率配置。[变化调用:33](../../../raw/current/scripts/components/decay.lua#L33)、[等待:43](../../../raw/current/scripts/components/decay.lua#L43)
- **addfuel 表示“补了燃料”**：事件判断仅是变化后 currenthealth>maxhealth；未限制本次 amount 为正，未限制旧值在上限以下，也不钳制上限。因此超过上限期间的后续变化仍可能再发 addfuel。[判断:21](../../../raw/current/scripts/components/decay.lua#L21)
- **spentfuel 每次到零都发**：只有旧值>0时发；已经为零再扣不会重复发。事件先发、再将负值归零，监听器当时可能读到未归零的 currenthealth。[顺序:14](../../../raw/current/scripts/components/decay.lua#L14)
- **移组件等于停线程**：这里没有 OnRemoveFromEntity；线程按宿主 GUID 绑定，宿主的 `KillTasks()` 才会调用 `KillThreadsWithID`。不要把移组件与宿主任务清理混为一谈；兼容逻辑须自行核验并负责停止其创建的线程。[线程绑定:57](../../../raw/current/scripts/components/decay.lua#L57)、[宿主 KillTasks:1091](../../../raw/current/scripts/entityscript.lua#L1091)
- **pause<=0 是完整复位**：它可终止旧线程但不把 deltatask 置 nil，也不重置 currenthealth／paused。后续检查字段非 nil 不能据此判定正在运行。[实现:49](../../../raw/current/scripts/components/decay.lua#L49)

## 验证记录

- 版本增量复核 26148036（2026-10-25）：锚点重定位 1 处（entityscript KillTasks +4）；无结论修订，仅锚点迁移——decay.lua、scheduler.lua 本版未变，num 分支疑点原样保留。依据：全局枢纽 diff 审阅（R-08 改动在湿润查询与距离函数，不涉任务清理）＋变更清单命中 entityscript.lua。
- 源码审阅：2026-10-23，Steam buildid 25643519；完整审阅 decay.lua 的 63 行，核对事件边界、num 自由变量疑点、暂停／停止顺序，以及调度器 StartThread、KillThread 和宿主 KillTasks。消费者搜索范围与未覆盖项见“典型接入”；不据无字面命中断言整个游戏或外部模组无用途。
- 结构检查：本页元数据、大小、Wiki／Markdown 链接与源码锚点检查通过；没有新增完整 Lua 示例，导航登记由本批整合检查。
- 游戏运行：未运行；num 分支未作运行复现，线程调度时序、报错后的句柄状态及旧模组兼容未验证。
