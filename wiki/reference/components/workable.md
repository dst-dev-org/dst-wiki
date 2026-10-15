---
title: "workable"
created: "2026-10-12"
updated: "2026-10-28"
type: component
summary: "让目标响应砍、挖矿或锤等工作动作，管理剩余工作量、反弹与完成回调。"
aliases: ["可挖矿目标", "采集工作量", "锤拆建筑", "禁用工作动作", "workable"]
tags: [component, mechanism, callback, loot, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/workable.lua", "actions.lua", "prefabs/rocks.lua", "prefabs/treasurechest.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# workable

## 快速判断

- 职责与适用：工作目标的动作、剩余工作量及完成通知；工具动作／效率归 [[reference/components/tool]]，耐久归 [[reference/components/finiteuses]]。[actions.lua:1693](../../../raw/current/scripts/actions.lua#L1693
- 优先使用：`SetWorkAction` 匹配动作，`SetWorkLeft` 改工作量；禁用用 `SetWorkable(false)`，完成后掉落／移除由原版回调或自己的回调负责。[workable.lua:65](../../../raw/current/scripts/components/workable.lua#L65 [workable.lua:79](../../../raw/current/scripts/components/workable.lua#L79 [rocks.lua:168](../../../raw/current/scripts/prefabs/rocks.lua#L168
- 重要副作用：SetWorkLeft 会重新启用，且至少设 1；工作过程中先 onwork，再判断是否完成，onwork 可改变结果。[workable.lua:83](../../../raw/current/scripts/components/workable.lua#L83 [workable.lua:151](../../../raw/current/scripts/components/workable.lua#L151
- 修改边界：服务端配置；“工作量”不等于鼠标次数，工具效率、人物倍率、目标倍率和反弹都参与。[actions.lua:1703](../../../raw/current/scripts/actions.lua#L1703 [workable.lua:189](../../../raw/current/scripts/components/workable.lua#L189

## 关键接口

| 接口／字段 | 参数或返回 | 条件、作用与证据 |
|---|---|---|
| `SetWorkAction(act)` | ACTIONS.CHOP／MINE／HAMMER 等 | 属性回调维护动作 id 的 `_workable` 标签；修改目标动作不会给工具增加能力。[workable.lua:16](../../../raw/current/scripts/components/workable.lua#L16 [workable.lua:65](../../../raw/current/scripts/components/workable.lua#L65 |
| `SetWorkLeft(work)`／`SetMaxWork(work)` | 数值；最低 1 | 前者启用并设剩余量，maxwork>0 时封顶；后者只设上限，不重置剩余量。[workable.lua:83](../../../raw/current/scripts/components/workable.lua#L83 [workable.lua:102](../../../raw/current/scripts/components/workable.lua#L102 |
| `SetWorkable(able)`／`CanBeWorked()` | boolean | 控制是否可工作；禁用时 GetWorkLeft 返回 0，内部量仍保留。[workable.lua:79](../../../raw/current/scripts/components/workable.lua#L79 |
| `SetOnWorkCallback(fn)` | `(inst, worker, workleft, numworks)` | 单槽替换；收到已结算量，可能在最终一次自行掉落／删目标。[workable.lua:151](../../../raw/current/scripts/components/workable.lua#L151 [rocks.lua:168](../../../raw/current/scripts/prefabs/rocks.lua#L168 |
| `SetOnFinishCallback(fn)` | `(inst, worker)` | onwork 后量≤0 才调用；不自动掉落或删除。[workable.lua:155](../../../raw/current/scripts/components/workable.lua#L155 |
| `WorkedBy(worker, numworks?)`／`Destroy(worker)` | 实体 worker；默认量 1 | 前者先反弹判定再工作；Destroy 传当前量，仍受反弹／倍率影响。worker 不能省略，内部访问其组件／事件。[workable.lua:73](../../../raw/current/scripts/components/workable.lua#L73 [workable.lua:123](../../../raw/current/scripts/components/workable.lua#L123 |
| `savestate`／`SetOnLoadFn(fn)` | 默认 false；`(inst, data)` | true 才保存剩余量与上限；加载量后回调，不保存动作、禁用标志或回调。[workable.lua:106](../../../raw/current/scripts/components/workable.lua#L106 |

### 工作顺序与原版完成位置

工作先应用目标 multiplier；正工作量减至末尾，剩余≤1 时任意正工作量会完成。随后记录 lastworker，向 worker 发 `working`、目标发 `worked`，调用 onwork，之后才判定完成、调用 onfinish 并发 `workfinished`／`finishedwork`。传入 0 也可能通知与回调，不应把它当纯查询。[workable.lua:130](../../../raw/current/scripts/components/workable.lua#L130

普通岩石在 **onwork** 的最终分支 DropLoot 并 Remove；木箱在 **onfinish** 掉落材料、处理 container 后 Remove。追加完成逻辑前先确认已有位置，避免重复掉落或漏掉内容物。[rocks.lua:168](../../../raw/current/scripts/prefabs/rocks.lua#L168 [rocks.lua:293](../../../raw/current/scripts/prefabs/rocks.lua#L293 [treasurechest.lua:53](../../../raw/current/scripts/prefabs/treasurechest.lua#L53

## 典型接入

### 将普通岩石的初始挖矿工作量设为四

目标模组 `modmain.lua`；对象是已注册的 `rock1`，沿用 MINE、资源、破碎动画和掉落回调，只修改初始量。工厂与注册见 [rocks.lua:293](../../../raw/current/scripts/prefabs/rocks.lua#L293)、[rocks.lua:315](../../../raw/current/scripts/prefabs/rocks.lua#L315)、[rocks.lua:524](../../../raw/current/scripts/prefabs/rocks.lua#L524)，PostInit 见 [modutil.lua:593](../../../raw/current/scripts/modutil.lua#L593)。

```lua
AddPrefabPostInit("rock1", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local workable = inst.components.workable
    if workable ~= nil then
        workable:SetWorkLeft(4)
    end
end)
```

数字单位为工作量，普通基础效率 1 且没有其他倍率时才对应四次工作；动画阈值仍按原版 ROCKS_MINE，未一起调整。本例不启用 savestate，读档后按工厂／PostInit 恢复初始量，停用后按原版初始化。不要直接套到已有阶段／升级／保存工作进度的对象；其 OnLoad 可覆盖此值。[workable.lua:106](../../../raw/current/scripts/components/workable.lua#L106 [rocks.lua:183](../../../raw/current/scripts/prefabs/rocks.lua#L183 [rocks.lua:234](../../../raw/current/scripts/prefabs/rocks.lua#L234

## 常见陷阱与限制

- **SetWorkLeft(0) 禁用或立即拆除**：它至少设 1 且启用；禁用用 SetWorkable(false)，实际销毁走有 worker 的工作／目标原版完成链。[workable.lua:83](../../../raw/current/scripts/components/workable.lua#L83
- **覆盖回调后原版清理仍会执行**：两个 setter 都替换单槽。先保留原函数，确认是否已掉落／删实体，再设计包装；容器内容物不由 workable 自动处理。[workable.lua:173](../../../raw/current/scripts/components/workable.lua#L173 [treasurechest.lua:53](../../../raw/current/scripts/prefabs/treasurechest.lua#L53
- **加组件就等于新目标已能交互**：仍需匹配工具动作、双端可见标签及原版动作条件。本页修改已可交互目标，不提供完整新 Prefab 工厂。[actions.lua:1693](../../../raw/current/scripts/actions.lua#L1693

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 漂移文件 actions.lua 对照 build-26148036 更新报告（R-01、R-04）逐条核验，锚点重定位 3 处、结论修订 0 处（DoToolWork 与 CHOP／MINE／HAMMER／DIG 四动作未变；本次新增 USEITEM.spellcaster 处理器按 `castonworkable` 标签对 CHOP/DIG/HAMMER/MINE_workable 目标给 CASTSPELL，是 spellcaster 侧新消费，不改变 workable 组件行为）；未覆盖：运行验证。

- 源码审阅：2026-10-12，Steam buildid 25643519；覆盖动作／标签、量／反弹、回调顺序与存档，普通岩石／木箱完成模式及发布案例。新动作、全部客户端动作收集和所有目标保存协作未覆盖。
- 结构检查：元数据、证据行、链接／章节、大小、登记通过；发布 Lua 与岩石工作＋掉落隔离接入经 Full Moon 3.0.0／Lua 5.1 语法及原字节往返检查。
- 游戏运行：未运行；工作次数、动画阈值、读档和联机／其他模组倍率待实测。

相关：[[reference/components/tool#关键接口]]；[[reference/components/lootdropper#典型接入]]；[[reference/components/burnable#常见陷阱与限制]]；[[reference/components/pickable#关键接口|采收与再生]]；[[reference/components/growable#关键接口|阶段变化]]；[[reference/components/workmultiplier#关键流程|执行者基础量、倍率与反弹顺序]]。
