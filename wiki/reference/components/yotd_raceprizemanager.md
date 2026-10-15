---
title: "yotd_raceprizemanager"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "龙年（YotD）赛船活动世界组件：管理当日龙舟赛奖品额度与检查点注册表，活动关闭时奖品归零、新周期恢复。"
aliases: ["龙舟赛奖品", "yotd奖品管理", "yotd_raceprizemanager"]
tags: [component]
dst_build: "steam-buildid:26230496"
size_class: simple
sources: ["components/yotd_raceprizemanager.lua", "prefabs/cave.lua", "prefabs/forest.lua", "prefabs/boatrace_start.lua", "prefabs/boatrace_checkpoint.lua", "prefabs/yotd_dragonshrine.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# yotd_raceprizemanager

## 快速判断

- 职责与适用：挂在 `TheWorld` 上的服务端组件，记录龙舟赛"今天还有没有奖品"（初始 1 份）与已布置的检查点集合；赛点、龙 shrine 靠它决定表现与发奖。[变量](../../../raw/current/scripts/components/yotd_raceprizemanager.lua#L21-L22)
- 优先使用：查询 `HasPrizeAvailable()`，扣减 `PrizeGiven()`，检查点 `RegisterCheckpoint`／`UnregisterCheckpoint`／`GetCheckpoints()`。[公共方法](../../../raw/current/scripts/components/yotd_raceprizemanager.lua#L64-L88)
- 重要副作用：状态变化都推世界事件 `yotd_ratraceprizechange`；跨天（`WatchWorldState("cycles")`）在活动进行中把奖品恢复为 1。[跨天恢复](../../../raw/current/scripts/components/yotd_raceprizemanager.lua#L32-L39)
- 修改边界：构造断言 mastersim；组件只记额度与检查点，比赛流程全在 boatrace_start/checkpoint prefab；初始化 0 秒任务按活动开关决定 0 或 1。[断言](../../../raw/current/scripts/components/yotd_raceprizemanager.lua#L7)、[初始化](../../../raw/current/scripts/components/yotd_raceprizemanager.lua#L45-L50)

## 关键接口

| 接口／字段 | 已核验类型／单位 | 条件、作用与证据 |
|---|---|---|
| `HasPrizeAvailable()` | 布尔 | `_prize > 0`；龙 shrine 与赛点动画据此切换（idol/PRIZE 显示）。[L64-L66](../../../raw/current/scripts/components/yotd_raceprizemanager.lua#L64-L66)、[shrine](../../../raw/current/scripts/prefabs/yotd_dragonshrine.lua#L272-L279) |
| `PrizeGiven()` | 无返回 | `_prize - 1` 并推 `yotd_ratraceprizechange`；赛船发完奖品即调，当天剩余次数递减。[L68-L71](../../../raw/current/scripts/components/yotd_raceprizemanager.lua#L68-L71)、[赛点调用](../../../raw/current/scripts/prefabs/boatrace_start.lua#L616-L618) |
| `RegisterCheckpoint(cp)`／`UnregisterCheckpoint(cp)` | 无返回 | 以实体为 key 维护集合；赛点布设/移除时登记，供起跑线收集赛道。[L74-L84](../../../raw/current/scripts/components/yotd_raceprizemanager.lua#L74-L84)、[checkpoint 侧](../../../raw/current/scripts/prefabs/boatrace_checkpoint.lua#L151-L162) |
| `GetCheckpoints()` | 浅拷贝表 | 返回副本避免外部改坏注册表。[L86-L88](../../../raw/current/scripts/components/yotd_raceprizemanager.lua#L86-L88) |
| `updateprize`（内部，cycles 监听） | 无返回 | 仅当 `IsSpecialEventActive(SPECIAL_EVENTS.YOTD)` 且 `_prize < 1` 时拉回 1 并推事件——活动关闭后跨天不恢复。[L32-L39](../../../raw/current/scripts/components/yotd_raceprizemanager.lua#L32-L39)、[监听](../../../raw/current/scripts/components/yotd_raceprizemanager.lua#L54) |
| `OnSave`／`LoadPostPass` | 存读 | 只存 `_prize`；LoadPostPass 恢复后必推 `yotd_ratraceprizechange` 刷新表现；检查点注册表不存档，靠实体各自重登。[L94-L108](../../../raw/current/scripts/components/yotd_raceprizemanager.lua#L94-L108) |

## 典型接入

### 原版宿主与消费者

- 挂载：cave 与 forest 两个世界都 `AddComponent("yotd_raceprizemanager")`（与 yotc/yotb 同批），即地表与洞穴各自独立一份。[cave](../../../raw/current/scripts/prefabs/cave.lua#L543)、[forest](../../../raw/current/scripts/prefabs/forest.lua#L695)
- 赛点：激活起跑线时若 `HasPrizeAvailable()` 则布设 `{total=3},{total=2},{total=1}` 三档奖品；冲线发完调 `PrizeGiven()`；空额度时起跑线不显示 PRIZE。[布奖](../../../raw/current/scripts/prefabs/boatrace_start.lua#L502-L506)、[表现刷新](../../../raw/current/scripts/prefabs/boatrace_start.lua#L916-L921)
- 龙 shrine：监听 `yotd_ratraceprizechange` 切换神像形态。[shrine](../../../raw/current/scripts/prefabs/yotd_dragonshrine.lua#L272-L279)

### 模组：接入活动状态

- 前提与选择：服务端读取；活动开关注入点也可改 `_prize`（组件无 setter，直接赋值私有 local 不可行——外部只能经事件与方法交互）。
- 生效与撤销：监听 `yotd_ratraceprizechange` 响应奖品增减；不需要清理。

```lua
-- modmain.lua：奖品耗尽时提示（服务端）
AddPrefabPostInit("world", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    inst:ListenForEvent("yotd_ratraceprizechange", function(world)
        local m = world.components.yotd_raceprizemanager
        if m ~= nil and not m:HasPrizeAvailable() then
            print("[yotd] no race prize left today")
        end
    end)
end)
```

- 调整与验证：示例仅语法检查未运行；`_prize` 语义是"剩余可发奖的次数"而非奖品物品数，实际物品由赛点按 `{total=N}` 生成。

## 常见陷阱与限制

- 初始化走 0 秒任务而非构造期判定：构造后立刻读 `HasPrizeAvailable()` 可能拿到未修正的初值 1（活动关闭世界要等下帧才归 0）。[L45-L50](../../../raw/current/scripts/components/yotd_raceprizemanager.lua#L45-L50)
- 跨天恢复只把 `_prize` 拉回 1，不叠加上一周期未用次数；活动在关闭状态下的跨天不会恢复（要重新开启活动后等下一个周期）。[L32-L39](../../../raw/current/scripts/components/yotd_raceprizemanager.lua#L32-L39)
- 检查点注册表不进存档：读档后由各 `boatrace_checkpoint` 重新 Register；一个世界一份表意味着洞穴与地表的检查点互不相通。[LoadPostPass](../../../raw/current/scripts/components/yotd_raceprizemanager.lua#L100-L108)

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。覆盖组件全文（事件、存读、debug 串）；核对 cave/forest 挂载、boatrace_start 的布奖/扣减/表现、boatrace_checkpoint 的注册与注销、yotd_dragonshrine 表现；未覆盖 SPECIAL_EVENTS.YOTD 的活动排期配置。
- 游戏运行：未运行；本页结论均来自源码静态核验。
- 结构检查：`validate_page.py`、`validate_source_links.py --strict` 与示例 Lua 语法检查通过（见批记录）。
