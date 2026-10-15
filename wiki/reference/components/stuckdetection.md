---
title: "stuckdetection"
created: "2026-10-26"
updated: "2026-10-28"
type: component
summary: "按调用时的相邻水平位移维护卡住计时，供 Brain 自行决定攻击等应对；不自动采样、寻路或脱困。"
aliases: ["生物卡住检测", "小位移卡住计时", "IsStuck", "stuckdetection"]
tags: [component, ai, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/stuckdetection.lua", "util.lua", "prefabs/dragonfly.lua", "brains/dragonflybrain.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# stuckdetection

## 快速判断

- 职责与适用：调用 IsStuck 时采样 x/z，用相邻样本间的小位移是否持续够久作判断；没有 OnUpdate、自动事件或脱困动作。[stuckdetection.lua:1](../../../raw/current/scripts/components/stuckdetection.lua#L1
- 优先使用：由实际行为按需要调用 IsStuck，并在不应计入卡住的行为期间 Reset；原版龙蝇在吐幼虫阶段据此决定是否尝试攻击。[dragonflybrain.lua:71](../../../raw/current/scripts/brains/dragonflybrain.lua#L71
- 重要副作用：每次 IsStuck 都更新 lastx／lastz，小步累计移动很远也未必重置计时；它不是距初始点持续小于 0.05 的判定。[stuckdetection.lua:16](../../../raw/current/scripts/components/stuckdetection.lua#L16
- 修改边界：只提供空间／时钟采样，无内置服务端断言；原版参照装在主模拟生物上，阈值取世界距离与 GetTime 秒，非像素、地皮数或帧数。[dragonfly.lua:492](../../../raw/current/scripts/prefabs/dragonfly.lua#L492 [stuckdetection.lua:16](../../../raw/current/scripts/components/stuckdetection.lua#L16

## 关键接口

| 接口／字段 | 参数或返回 | 条件、作用与证据 |
|---|---|---|
| `SetTimeToStuck(t)`／`timetostuck` | 秒；默认 2 | 直接改时长，不重置 starttime，也不校验／钳制参数；已开始的窗口按新时长判断。[stuckdetection.lua:3](../../../raw/current/scripts/components/stuckdetection.lua#L3 [stuckdetection.lua:12](../../../raw/current/scripts/components/stuckdetection.lua#L12 |
| `IsStuck()` | 布尔值，同时采样 | 首次初始化并返回 false；与上次采样距离严格大于 0.05 才重置；否则更新坐标后，以 starttime+timetostuck 严格小于当前时间判 true。[stuckdetection.lua:16](../../../raw/current/scripts/components/stuckdetection.lua#L16 |
| `GetRemainingTime()` | 秒差；未初始化返回 −1 | 只读窗口终点减 GetTime，不查新坐标、不采样、不封底。已经到期可为 0 或负数，负值不能单独区分“未初始化”和“已过期”。[stuckdetection.lua:28](../../../raw/current/scripts/components/stuckdetection.lua#L28 |
| `Reset()` | 无参数 | 仅在 starttime 已存在时把坐标与时间改成当前值；首次 IsStuck 前调用无效果，不会主动启动窗口。[stuckdetection.lua:36](../../../raw/current/scripts/components/stuckdetection.lua#L36 |

## 关键流程

距离阈值固定为 0.05²，调用 util.distsq(lastx,lastz,x,z) 的四数值分支，只比较水平位置，忽略 y。距离恰等于 0.05 时不会重置；时长恰到 timetostuck 时也仍为 false，必须超过窗口终点。[stuckdetection.lua:1](../../../raw/current/scripts/components/stuckdetection.lua#L1 [stuckdetection.lua:16](../../../raw/current/scripts/components/stuckdetection.lua#L16 [util.lua:555](../../../raw/current/scripts/util.lua#L555

这是相邻采样，不是累计路程：例如每次采样只移动 0.04，last 坐标会跟着更新，但时间不重置，累计走得远仍可能判卡住；反过来单次超过阈值就重新计时。因此调用频率、行为是否暂停采样及何时 Reset 都是结果的一部分。GetRemainingTime 仅反映上次采样／Reset 留下的时钟，不能代替对当前位置的 IsStuck。[stuckdetection.lua:16](../../../raw/current/scripts/components/stuckdetection.lua#L16

组件没有 OnSave／OnLoad／LongUpdate、定时任务、OnRemoveFromEntity 或“已卡住”事件；重建时从未初始化开始，离线期间不做位移补算。停止调用也不会自动 Reset，后续再次采样若只发生小位移，仍会使用旧 starttime；应由当前行为决定何时重置。[stuckdetection.lua:3](../../../raw/current/scripts/components/stuckdetection.lua#L3

## 典型接入

### 原版龙蝇：吐幼虫路径受阻时尝试攻击

`dragonfly` 在服务端安装 stuckdetection、SGdragonfly 与 Brain，配置时长 2 秒。Brain 的 Spawn Lavae 分支一边 Leash 到所选位置，一边循环执行检测：SG busy 且不带 hit 时 Reset；其他情况下 IsStuck 为真且 combat 不在冷却才 TryAttack。组件只给判断，攻击选择与执行由 Brain／combat／SG 负责，不保证“卡住两秒就传送或解围”。[dragonfly.lua:516](../../../raw/current/scripts/prefabs/dragonfly.lua#L516 [dragonfly.lua:578](../../../raw/current/scripts/prefabs/dragonfly.lua#L578 [dragonfly.lua:624](../../../raw/current/scripts/prefabs/dragonfly.lua#L624 [dragonflybrain.lua:71](../../../raw/current/scripts/brains/dragonflybrain.lua#L71

这里 Reset 的意义是避免正常忙碌动作消耗卡住窗口；它并不让从未初始化的组件立刻开始计时，也不将卡住状态永久禁用。其他生物若采用不同采样节奏，不能直接照搬“2 秒”解释成同样的实际移动判定。[stuckdetection.lua:36](../../../raw/current/scripts/components/stuckdetection.lua#L36 [dragonflybrain.lua:78](../../../raw/current/scripts/brains/dragonflybrain.lua#L78

## 常见陷阱与限制

- **Reset 后不用 IsStuck 也会自动计时／触发**：Reset 未初始化时不生效，组件也无自动更新；GetRemainingTime 本身不初始化或产生应对动作。[stuckdetection.lua:28](../../../raw/current/scripts/components/stuckdetection.lua#L28 [stuckdetection.lua:36](../../../raw/current/scripts/components/stuckdetection.lua#L36
- **计时器只看离起点多远／角色是否移动**：它逐次更新 last 坐标，只比较本次水平位移；小步移动、只改高度及不同采样频率均不能按累计移动解释。[stuckdetection.lua:16](../../../raw/current/scripts/components/stuckdetection.lua#L16

## 验证记录

- 源码审阅：2026-10-26，Steam buildid 26148036；覆盖组件全文、水平距离分支、龙蝇主模拟安装与吐幼虫 Brain 的 Reset／IsStuck／TryAttack 消费。未展开其他生物／路线跟随消费者或完整攻击／寻路执行。
- 结构检查：validate_page 及源码引用 strict 检查通过；无新增完整 Lua 示例。机械检查不代替游戏运行。
- 游戏运行：未运行；阈值相等、小步采样、长时间停止采样、未初始化 Reset 与龙蝇受阻行为待实测。

相关：[[reference/components/locomotor]]；龙蝇吐幼虫分工见 [[reference/components/rampingspawner]]。
