---
title: "incrementalproducer"
created: "2026-10-24"
updated: "2026-10-28"
type: component
summary: "按当前数量和时间条件建立待生产预算，再由宿主行为逐次消费；参考蜘蛛女王的产仔调用链。"
aliases: ["incrementalproducer", "分批产仔预算", "蜘蛛女王产仔", "CanProduce有副作用", "toproduce"]
tags: [component, creature, callback, ai, state-machine, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/incrementalproducer.lua", "prefabs/spiderqueen.lua", "brains/spiderqueenbrain.lua", "stategraphs/SGspiderqueen.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# incrementalproducer

## 快速判断

- 职责与适用：管理 `toproduce` 待处理预算，达到时间／数量条件时重新装填，每次产物回调消耗一份；不自带更新、任务、收获动作或产物实体。自动累积可收取产物优先查 [[reference/components/harvestable]]。[完整状态](../../../raw/current/scripts/components/incrementalproducer.lua#L2-L16) [预算与消费](../../../raw/current/scripts/components/incrementalproducer.lua#L20-L49)
- 优先使用：宿主先提供 `countfn`／`producefn`，配置上限、增量与间隔；由 Brain／业务入口查询，再由合适动画帧 `TryProduce()`。原版蜘蛛女王按此链路连接。[工厂配置](../../../raw/current/scripts/prefabs/spiderqueen.lua#L201-L206) [Brain入口](../../../raw/current/scripts/brains/spiderqueenbrain.lua#L12-L15) [SG执行](../../../raw/current/scripts/stategraphs/SGspiderqueen.lua#L201-L204)
- 重要副作用：`CanProduce()` **不是纯查询**，可能重算上限／增量并覆盖预算；条件不满足时保留旧预算。`DoProduce()` 不重验资格，不读取回调返回值，回调正常返回即预算减一并记录时间。[查询](../../../raw/current/scripts/components/incrementalproducer.lua#L20-L36) [执行](../../../raw/current/scripts/components/incrementalproducer.lua#L45-L49)
- 修改边界：原版只在服务端宿主安装，生产内容、计数和行为门禁由宿主负责；组件无 OnSave／OnLoad、更新或移除钩子，单加组件不会定时产生物品／生物。[服务端分界](../../../raw/current/scripts/prefabs/spiderqueen.lua#L147-L151) [挂载](../../../raw/current/scripts/prefabs/spiderqueen.lua#L201-L206) [完整组件](../../../raw/current/scripts/components/incrementalproducer.lua#L2-L57)

## 关键接口

| 接口／字段 | 参数、初值与边界 |
|---|---|
| `countfn(inst)` | 必须提供，CanProduce 和 GetDebugString 直接调用；返回当前数量。组件不维护实际存量。[读取](../../../raw/current/scripts/components/incrementalproducer.lua#L29-L33) [调试读取](../../../raw/current/scripts/components/incrementalproducer.lua#L53-L54) |
| `maxcount`／`maxcountfn(inst)` | 默认 0／nil；有函数时每次 CanProduce 都用其结果覆盖 maxcount。只在装填新预算时比较数量，不是持续硬上限。[字段](../../../raw/current/scripts/components/incrementalproducer.lua#L6-L8) [重算](../../../raw/current/scripts/components/incrementalproducer.lua#L20-L33) |
| `increment`／`incrementfn(inst)` | 默认 1／nil；CanProduce 先调用函数更新 increment，允许装填时又调用一次，并以剩余容量夹取。回调应稳定且无一次性消费副作用，结果由调用方保证有效数值。[定义](../../../raw/current/scripts/components/incrementalproducer.lua#L10-L12) [两次调用](../../../raw/current/scripts/components/incrementalproducer.lua#L25-L33) |
| `incrementdelay`／`lastproduction` | 默认 1 秒／0；严格 `GetTime() > lastproduction + incrementdelay` 才可重新装填。不是两件产物之间的强制间隔，初值也不是宿主创建时刻。[初值](../../../raw/current/scripts/components/incrementalproducer.lua#L12-L16) [资格](../../../raw/current/scripts/components/incrementalproducer.lua#L31-L36) |
| `toproduce` | 默认 0；装填时为 min(剩余容量, 本次增量)，是**覆盖**而非与旧预算相加。装填条件不满足不会清零旧值。[实现](../../../raw/current/scripts/components/incrementalproducer.lua#L31-L36) |
| `CanProduce()`／`TryProduce()` | 前者返回当前预算是否大于 0；后者先调用前者，通过才 DoProduce，无成功返回约定。[实现](../../../raw/current/scripts/components/incrementalproducer.lua#L20-L42) |
| `producefn(inst)`／`DoProduce()` | 有回调就先调用，再预算减 1（不低于 0）、lastproduction=GetTime；无回调则什么都不变。直接 DoProduce 即使预算为 0 也可调用回调。[执行](../../../raw/current/scripts/components/incrementalproducer.lua#L45-L49) |

### 预算与间隔不是同一层限制

CanProduce 每次先重算动态配置和当前数量。若时间已过且数量低于上限，重新设置预算；否则仅返回**原有**预算是否仍为正。因此数量已经达标或尚未经过 delay，仍可能因旧预算而返回 true；反复查询也可能重填而非仅观察预算。需要“每次产出前绝不超额／必须等间隔”的新机制，须在宿主定义明确的执行门禁，不能直接套用此组件的名字。[完整判定](../../../raw/current/scripts/components/incrementalproducer.lua#L20-L36)

## 典型接入

### 按蜘蛛女王的计数、Brain 与动画帧分工

这是原版接入指引，不是完整新生物 demo；自定义宿主仍需自己的实体、资源、Brain／SG 与产物回调。

1. **服务端配置：** 蜘蛛女王挂组件并连接 BabyCount、MakeBaby、MaxBabies、AdditionalBabies，间隔为 `TUNING.SPIDERQUEEN_GIVEBIRTHPERIOD`，当前 10 秒；另配置 leader、SG 与 Brain。[端界](../../../raw/current/scripts/prefabs/spiderqueen.lua#L147-L151) [组件接线](../../../raw/current/scripts/prefabs/spiderqueen.lua#L201-L206) [配套](../../../raw/current/scripts/prefabs/spiderqueen.lua#L216-L224) [常量](../../../raw/current/scripts/tuning.lua#L2007)
2. **数量／批次：** 当前数量读 leader.numfollowers，不是组件自己的计数；动态上限与增量按附近带 player、无 playerghost 标签的数量计算，分别为 `RoundBiasedDown((人数*20)^(1/1.4))` 和 `RoundBiasedUp(人数*.5)`。范围当前为 20，关系登记复用 [[reference/components/leader]]。[计数](../../../raw/current/scripts/prefabs/spiderqueen.lua#L73-L74) [动态计算](../../../raw/current/scripts/prefabs/spiderqueen.lua#L99-L111) [范围](../../../raw/current/scripts/tuning.lua#L2010)
3. **触发／执行：** Brain 额外要求宿主存活时间大于 5 秒且 SG 非 busy，然后检查 CanProduce、进入 poop_pre；poop_loop 第 10 帧 TryProduce，动画结束再 CanProduce 决定继续循环还是收尾。动作节奏由 SG 提供，不是组件自己排定时器。[Brain门禁](../../../raw/current/scripts/brains/spiderqueenbrain.lua#L12-L15) [入场](../../../raw/current/scripts/brains/spiderqueenbrain.lua#L60-L61) [循环执行与判断](../../../raw/current/scripts/stategraphs/SGspiderqueen.lua#L196-L215)
4. **实际产物：** MakeBaby 根据战斗目标和幸运抽取选蜘蛛类型，经 lootdropper 生成、放到母体后方并设为随从；生成失败也正常返回，组件仍扣预算、更新时间。不将“消费一次预算”记作“成功生成一只”。[产物回调](../../../raw/current/scripts/prefabs/spiderqueen.lua#L77-L96) [预算消费](../../../raw/current/scripts/components/incrementalproducer.lua#L45-L49)

改变原版批次策略应接入自己负责的动态回调与 Brain／SG 门禁；仅设 maxcount／increment 会被已有函数覆盖。只想调整新预算的等待条件，可在原版服务端初始化后改 incrementdelay，但不承诺同步改变每只出生的动画节奏或清掉旧预算。[原配置](../../../raw/current/scripts/prefabs/spiderqueen.lua#L201-L206) [重算](../../../raw/current/scripts/components/incrementalproducer.lua#L20-L36)

## 常见陷阱与限制

- **把 CanProduce 当无副作用 UI 查询：** 它写 maxcount、increment、toproduce，incrementfn 在同次调用中还可能执行两次；不要在回调内消耗资源或依赖“调用一次就是一轮生产”。[实现](../../../raw/current/scripts/components/incrementalproducer.lua#L20-L36)
- **用回调 false／nil 表示失败就不扣预算：** 组件不读返回值，只要回调正常返回就更新预算与时间；需要可靠成功状态时由宿主另设协议，不把 TryProduce 的无返回值当成功。[执行](../../../raw/current/scripts/components/incrementalproducer.lua#L39-L49)
- **直接 DoProduce 能保证容量：** 它不检查 countfn、间隔或预算，空预算也可调用生产函数；常规调用沿 TryProduce，但旧预算的限制仍须按上文理解。[执行](../../../raw/current/scripts/components/incrementalproducer.lua#L45-L49)
- **缺 countfn 或 producefn 只是关闭生产：** 前者被直接调用会报错；后者让预算不消耗。若宿主 SG 按 CanProduce 续循环，配置缺口会导致行为不结束，不能用省回调当停用开关。[读取](../../../raw/current/scripts/components/incrementalproducer.lua#L29-L36) [执行](../../../raw/current/scripts/components/incrementalproducer.lua#L45-L49) [SG续轮](../../../raw/current/scripts/stategraphs/SGspiderqueen.lua#L210-L215)
- **自动保存／移除任务：** 此组件没有自己的任务、更新、存档或清理钩子；间隔、预算与 lastproduction 不由它保存。实际行为由 Brain／SG 驱动，动态移除前先处理这些调用方，不能仅删组件后保证整套产仔停止。[完整定义](../../../raw/current/scripts/components/incrementalproducer.lua#L2-L57) [原版调用](../../../raw/current/scripts/brains/spiderqueenbrain.lua#L12-L15)

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 处理范围：锚点重定位 6 处（spiderqueen 工厂区 +2 行移、tuning +9，区间文本逐段比对一致）；结论修订：无（SPIDERQUEEN_GIVEBIRTHPERIOD／RANGE 键值未变）。
- 依据：tuning 键级 hunk 地图＋变更清单命中 tuning.lua、prefabs/spiderqueen.lua。

- 源码审阅：2026-10-24，Steam buildid 25643519；完整组件、蜘蛛女王工厂／产物与动态配置、Brain 入口和 SG 生产循环、两项 tuning。未全面审阅蜘蛛类型幸运公式、leader 全生命周期或自定义产物体系。
- 结构检查：本页格式、来源、本地链接及源码引用严格检查通过；无完整 Lua 示例，导航由本批统一登记后检查。
- 游戏运行：未运行；多人批次数量、跨存读、生成失败、动画中断与动态换组件未实测。
