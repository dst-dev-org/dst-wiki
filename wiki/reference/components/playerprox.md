---
title: "playerprox"
created: "2026-10-17"
updated: "2026-10-28"
type: component
summary: "按近远两个阈值轮询玩家接近／离开，选择整体开关、逐人或锁定目标模式。"
aliases: ["玩家靠近触发", "玩家离开范围", "接近检测", "playerprox"]
tags: [component, callback, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/playerprox.lua", "simutil.lua", "entityscript.lua", "prefabs/wormhole.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# playerprox

## 快速判断

- 职责与适用：定期检测玩家距离并在状态切换时回调；不是每帧范围光环。[默认参数](../../../raw/current/scripts/components/playerprox.lua#L127-L143)
- 优先使用：整体开关用默认 `TargetModes.AnyPlayer`，逐人进出用 `AllPlayers`；近阈值小、远阈值大形成滞回。[模式差异](#关键接口)
- 重要副作用：默认远离回调只有宿主参数；AllPlayers 模式才逐人传回目标，不能共用“离开者必存在”的假设。[AnyPlayer](../../../raw/current/scripts/components/playerprox.lua#L44-L60)、[AllPlayers](../../../raw/current/scripts/components/playerprox.lua#L9-L42)
- 修改边界：组件可用于客户端 Prefab，本页服务端示例不意味着组件自动同步；SpecificPlayer 路径仅测距离，不能假定生死／可见过滤适用于所有模式。[SetTarget](../../../raw/current/scripts/components/playerprox.lua#L234-L244)、[指定目标](../../../raw/current/scripts/components/playerprox.lua#L62-L76)

## 关键接口

| 接口／模式 | 条件、作用与证据 |
|---|---|
| `SetDist(near, far)` | 地图距离，非平方值；只赋值，无范围检查，建议 `0 < near < far`。[实现](../../../raw/current/scripts/components/playerprox.lua#L177-L180) |
| `SetOnPlayerNear(fn)`／`SetOnPlayerFar(fn)` | 覆盖回调，不叠加；需要保留原行为时包裹原函数。[设置](../../../raw/current/scripts/components/playerprox.lua#L165-L171) |
| `AnyPlayer` | 首名进入 near 时一次 near；全部离开 far 时一次 far，期间不逐人通知。[实现](../../../raw/current/scripts/components/playerprox.lua#L44-L60) |
| `AllPlayers` | 每名进入者 near；已记录者离开 far／失去过滤资格时 far，但已无效者直接丢弃，不保证对应 far。[实现](../../../raw/current/scripts/components/playerprox.lua#L9-L42) |
| `SpecificPlayer` | `SetTargetMode(mode, target)`，必须给非 nil 目标；只对该目标做 `IsNear`。[实现](../../../raw/current/scripts/components/playerprox.lua#L62-L76)、[断言](../../../raw/current/scripts/components/playerprox.lua#L224-L232) |
| `LockOnPlayer`／`LockAndKeepPlayer` | 前者进范围锁定、离开释放；后者首次锁定后切 SpecificPlayer，继续跟踪同一目标。[实现](../../../raw/current/scripts/components/playerprox.lua#L78-L113) |
| `SetPlayerAliveMode(mode)` | `AliveModes.AliveOnly=true`、`DeadOnly=false`、`DeadOrAlive=nil`；默认全部。[枚举](../../../raw/current/scripts/components/playerprox.lua#L145-L159) |
| `Schedule(seconds)`／`ForceUpdate()`／`Stop()` | 默认周期 `10 * FRAMES`；Schedule 重排任务，ForceUpdate 仅已有 task 时执行，Stop 不清 isclose 或补发 far。[实现](../../../raw/current/scripts/components/playerprox.lua#L190-L209) |
| `IsPlayerClose()` | 返回上次检测状态，不是实时空间查询。[实现](../../../raw/current/scripts/components/playerprox.lua#L173-L175) |

Any／All 模式通过 simutil 遍历可见玩家，生死过滤包含鬼魂判断，距离使用 x/z 平面平方值；等于范围边界不算内部。[筛选](../../../raw/current/scripts/simutil.lua#L143-L171)、[平面距离](../../../raw/current/scripts/entityscript.lua#L1435-L1442)

## 典型接入

### 虫洞更早响应活着的玩家，所有人远离后关闭

目标模组 `modmain.lua`，沿用原版虫洞资源、SG 及 near／far 回调，仅改为近 6、远 8，并忽略鬼魂。原版回调还要求传送有效／不忙碌，不保证进范围必开或离开立即关。[原版开关](../../../raw/current/scripts/prefabs/wormhole.lua#L61-L71)、[挂载](../../../raw/current/scripts/prefabs/wormhole.lua#L148-L168)、[Hook](../../../raw/current/scripts/modutil.lua#L593-L599)

```lua
AddPrefabPostInit("wormhole", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    local prox = inst.components.playerprox
    if prox ~= nil then
        prox:SetDist(6, 8)
        prox:SetPlayerAliveMode(prox.AliveModes.AliveOnly)
    end
end)
```

默认 AnyPlayer 适合虫洞整体开关；不要为了“多人”直接切 AllPlayers 后复用原 far：某一人离开就可能请求关闭，仍有别人附近。无需新动画；停用模组后恢复原构造配置，既有开关状态由后续轮询与原版 SG 决定。

## 常见陷阱与限制

- **离屏不是持续检测：** OnEntitySleep 先 ForceUpdate 再 Stop；唤醒重排并更新，卸载／移除取消任务。需要离屏持续业务应另设计，不把距离回调当通用计时器。[生命周期](../../../raw/current/scripts/components/playerprox.lua#L211-L222)
- **切模式不清旧状态：** SetTargetMode 不重置 isclose／closeplayers；运行中切模式须处理旧状态，不能期待补发匹配的 far。[实现](../../../raw/current/scripts/components/playerprox.lua#L224-L244)
- **丢目标恢复有代码边界：** OnTargetLeft 检查 `initialtargetmode`，而 SetTargetMode 写的是 `originaltargetmode`；本实现不能据此承诺锁定模式在目标移除后自动恢复。SpecificPlayer 应明确安排重新选择，避免稍后恢复轮询时目标为 nil。[丢失路径](../../../raw/current/scripts/components/playerprox.lua#L115-L125)、[赋值](../../../raw/current/scripts/components/playerprox.lua#L224-L232)
- **AliveMode 不是统一门禁：** SpecificPlayer／锁定后的距离分支不重新应用生死过滤；AllPlayers 的无效目标也不会发 far，逐人资源清理要补目标移除路径。[模式实现](../../../raw/current/scripts/components/playerprox.lua#L9-L113)

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 处理范围：锚点重定位 1 处（entityscript GetDistanceSqToPoint 等价改写 math2d.DistSq，x/z 平面距离语义不变；GetDistanceSqToInst 移除 assert 不影响本页结论）；结论修订：无，仅锚点迁移。
- 依据：全局枢纽 hunk 地图（R-08）＋变更清单命中 entityscript.lua、simutil.lua。

- 源码审阅：2026-10-17，Steam buildid 25643519；覆盖五种模式、参数／过滤、任务生命周期与虫洞现有回调，特别保留目标模式字段不一致的边界。未据头注释“四种模式”省略实际 AllPlayers。
- 结构与语法：本批执行格式、登记、链接／源码锚点检查及完整示例 Lua 5.1 语法检查；结果见本批交付报告。
- 游戏运行：未运行；多人先后进出、死亡／鬼魂、目标离场、离屏与动态切模式待实测。
