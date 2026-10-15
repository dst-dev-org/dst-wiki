---
title: "boatleak"
created: "2026-10-21"
updated: "2026-10-28"
type: component
summary: "服务端单个船体漏点实体的状态机：小/中漏、被压住、补丁修复与水草补丁到期，供 hullhealth 与补漏动作复用。"
aliases: ["船体漏点", "船的破洞", "补漏", "boatleak"]
tags: [component, lifecycle, state-machine]
dst_build: "steam-buildid:26230496"
sources: ["components/boatleak.lua", "prefabs/boat_leak.lua", "components/hullhealth.lua", "components/boatpatch.lua", "components/repairable.lua", "prefabs/boat.lua", "prefabs/boatpatch.lua", "prefabs/sewing_tape.lua", "prefabs/treegrowthsolution.lua", "actions.lua", "componentactions.lua", "tuning.lua", "entityscript.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# boatleak

## 快速判断

- 职责与适用：管**一个** `boat_leak` 漏点实体的表现与状态（小漏／中漏、被压住、补丁修好、水草补丁到期），并向外抛“漏了／修好了”回调。作用对象是独立实体而不是船；船体 HP 与“何时该漏”归 [[reference/components/hullhealth]]，本组件不扣血、不选点。[构造](../../../raw/current/scripts/components/boatleak.lua#L13-L25)
- 优先使用：修漏用 `Repair(doer, patch_item)`（会消耗补丁）；换状态用 `SetState(state)`；读状态读 `current_state`／`has_leaks`；水草补丁倒计时用 `SetRepairedTime(秒)`。[Repair](../../../raw/current/scripts/components/boatleak.lua#L62-L94)
- 重要副作用：切 `boat_leak`／`boat_repaired_patch` tag、换 build 与动画、开关循环水声、维护 `has_leaks`；进入小/中漏或修好时调用 prefab 挂的 `onsprungleak`／`onrepairedleak`。[ChangeToRepaired](../../../raw/current/scripts/components/boatleak.lua#L96-L121)
- 修改边界：仅服务端（组件在 prefab 的 `ismastersim` 之后添加）；`SetState` 遇同名状态直接返回，不能靠重复调用重放特效；碰撞产生的固定漏点由 hullhealth 存档，本组件只存 `isdynamic` 的动态漏点。

## 关键接口

| 接口／字段 | 已核验类型／单位 | 条件、作用与证据 |
|---|---|---|
| `current_state` | 字符串或 nil | 状态机当前值，初值 nil；`OnSave`、hullhealth 结算与 UNPATCH 都读它。[SetState 尾部](../../../raw/current/scripts/components/boatleak.lua#L260-L261) |
| `has_leaks` | 布尔，默认 false | 只要进过 leak 态就为 true，修复时置 false；brains 用 `has_leaks ~= false` 找漏点，语义是“这实体在漏”。[L116](../../../raw/current/scripts/components/boatleak.lua#L116)、[L175](../../../raw/current/scripts/components/boatleak.lua#L175) |
| `isdynamic` | 属性，赋值即写 `inst.persists` | 构造里赋 false；boat 生成动态漏点后赋 true，实体才进存档。属性 setter 见 [L9-L25](../../../raw/current/scripts/components/boatleak.lua#L9-L25) |
| `leak_build`／`leak_build_override` | build 名；默认 `"boat_leak_build"` | `SetBoat` 从船的 `leak_build`／`leak_build_override` 覆盖（远古船用独立 override），影响漏点贴图。[SetBoat](../../../raw/current/scripts/components/boatleak.lua#L263-L273) |
| `Repair(doer, patch_item)` | 返回 true／false | 目标必须带 `boat_leak` tag；先让船的 `repairable` 吃补丁（回血），没吃成就自己删物品；再按 `boatpatch:GetPatchType()` 决定修复态，0.4 秒后切过去。[L62-L94](../../../raw/current/scripts/components/boatleak.lua#L62-L94) |
| `SetState(state, skip_open)` | 无返回 | 状态机唯一切换入口；取消水草到期任务、换动画与 tag；`skip_open` 跳过开场动画，原版无调用者。[L144-L261](../../../raw/current/scripts/components/boatleak.lua#L144-L261) |
| `SetPlugged(setting)` | 布尔 | 只在 `small_leak`／`med_leak` 与对应 `_plugged` 之间切换；其它状态 no-op。[L135-L142](../../../raw/current/scripts/components/boatleak.lua#L135-L142) |
| `SetRepairedTime(time)`／`GetRemainingRepairedTime()` | 秒；剩余秒或 nil | 只有水草补丁用；到期任务会被 `SetState` 取消，`LongUpdate` 用它做时间跳跃补偿。[L123-L133](../../../raw/current/scripts/components/boatleak.lua#L123-L133) |

### 状态机

- 状态集合：`small_leak`、`med_leak` 各有 `_plugged` 变体；修复态 `repaired`（木补丁）、`repaired_tape`（缝纫胶带）、`repaired_kelp`（水草，会到期）、`repaired_treegrowth`（`treegrowthsolution`，播完动画自删）。plug 映射表只登记小/中漏。[映射](../../../raw/current/scripts/components/boatleak.lua#L1-L7)
- 进入 leak 态：摘 `boat_repaired_patch`、加 `boat_leak` tag，换 build 与 `boat_leak` bank、BillBoard 朝向、`LAYER_WORLD`，播循环水声并调 `onsprungleak`。[L154-L179](../../../raw/current/scripts/components/boatleak.lua#L154-L179)
- 被压住：塞住态播 `plugged_spray` 循环音并生成 `splash`；踩住的判定在 prefab 的 `FindLeakBlocker` 里（`updatelooper` 驱动，半径 0.4）。[prefab 阻塞](../../../raw/current/scripts/prefabs/boat_leak.lua#L113-L159)
- 修复态：摘 `boat_leak` tag、加 `boat_repaired_patch`、换 `boat_repair` bank、排序 3、`OnGround` + `LAYER_BACKGROUND`、停循环音、`has_leaks = false`。[L96-L121](../../../raw/current/scripts/components/boatleak.lua#L96-L121)
- 水草到期：`SetRepairedTime` 挂一次性任务 → 播 `pre_timeout` → 动画长度 + 2 帧后生成 `repaired_kelp_timeout_fx` 并回 `med_leak`。[L33-L60](../../../raw/current/scripts/components/boatleak.lua#L33-L60)
- 存档只存 `isdynamic` 的漏点，且把 plugged 态归一成未塞状态；固定漏点走 hullhealth 自己的表。[OnSave](../../../raw/current/scripts/components/boatleak.lua#L285-L294)

## 典型接入

### 原版宿主：漏点从哪来

- 唯一宿主：prefab `boat_leak` 在 master sim 段 `AddComponent("boatleak")`，随即把 `onsprungleak`／`onrepairedleak` 换成 prefab 内实现（可检查、潮湿、NOCLICK）。[宿主](../../../raw/current/scripts/prefabs/boat_leak.lua#L212-L246)
- 固定漏点：hullhealth 的 6 个点达到阈值时 `SpawnPrefab("boat_leak")` → 定位 → `SetBoat` → `SetState(small/med)`，存进 `leak_indicators`，由 hullhealth 存档。[生成](../../../raw/current/scripts/components/hullhealth.lua#L110-L134)
- 动态漏点：船收到 `spawnnewboatleak`（炮弹、锚、三叉戟、蟹王等）时生成，置 `isDynamic = true` 后进 `leak_indicators_dynamic`。[宿主监听](../../../raw/current/scripts/prefabs/boat.lua#L188-L213)
- 补漏入口：动作 `REPAIR_LEAK` 只在“手持带 `boat_patch` tag 的物品 + 目标带 `boat_leak` tag”时出现；`UNPATCH`（W.O.B.O.T. 机械手）撕下补丁、按回血量扣船血并把漏点打回 `small_leak`。[动作](../../../raw/current/scripts/actions.lua#L4916-L4920)、[出现条件](../../../raw/current/scripts/componentactions.lua#L1070-L1074)、[撕补丁](../../../raw/current/scripts/actions.lua#L5376-L5411)

### 模组：在漏点出现／修好时接手

- 前提与选择：改的是漏点实体而不是船；用 `AddPrefabPostInit("boat_leak")` 包住 prefab 已挂的回调，覆盖会丢掉检查与潮湿逻辑。
- 端与资源：服务端；先判 `TheWorld.ismastersim`；不需要新资源。
- 生效与撤销：只影响之后生成的漏点；删掉 postinit 并重载即恢复。在场漏点保留已包的回调直到销毁。
- 单位：`SetRepairedTime` 用秒；水草补丁按漏点大小取 0.75 天（小）／0.5 天（中）。[tuning](../../../raw/current/scripts/tuning.lua#L8255-L8256)

```lua
-- modmain.lua：漏点出现/修好时通知，保留原版回调
AddPrefabPostInit("boat_leak", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    local boatleak = inst.components.boatleak
    if boatleak == nil then
        return
    end

    local sprung, repaired = boatleak.onsprungleak, boatleak.onrepairedleak
    boatleak.onsprungleak = function(leak, state)
        if sprung ~= nil then sprung(leak, state) end
        print("[leak] sprung", state, tostring(leak:GetCurrentPlatform() ~= nil))
    end
    boatleak.onrepairedleak = function(leak)
        if repaired ~= nil then repaired(leak) end
        print("[leak] repaired", tostring(leak:HasTag("boat_repaired_patch")))
    end
end)
```

- 调整与验证：示例只做 Lua 5.1 语法检查，未在游戏内执行；回调时机见 `SetState` 与 `ChangeToRepaired` 的行锚点。

## 常见陷阱与限制

- 同名状态重复调用无效果：`SetState` 在 `state == current_state` 时直接 return，想“刷新”同等级漏点不会重播特效、也不会重触发 `onsprungleak`。[L145](../../../raw/current/scripts/components/boatleak.lua#L145)
- 名字有两套：`boatleak` 既是组件名也是实体 tag，漏点和已修好的补丁实体都带它；而“正在漏”的标记是另一个 tag `boat_leak`，修好后换成 `boat_repaired_patch`。按 `boat_leak` 找会漏掉补丁实体，按 `boatleak` 找则两者都命中。[prefab tag](../../../raw/current/scripts/prefabs/boat_leak.lua#L223)
- 自制补丁类型会落空：`Repair` 把 `patch_type` 拼成 `repaired_<type>`，但 `SetState` 只写了四个修复分支；新类型不换动画、不摘 `boat_leak` tag，而 `current_state` 变成未知串 —— hullhealth 按“非小/中漏”记 0 伤害，实体却仍在漏。[拼接](../../../raw/current/scripts/components/boatleak.lua#L87-L88)、[分支](../../../raw/current/scripts/components/boatleak.lua#L234-L258)
- 补丁的消耗与读取顺序：先调船的 `repairable:Repair`（材料匹配且船未满血才吃物品并回血），没吃成才由 boatleak 删物品；之后才读 `boatpatch:GetPatchType()`。`Remove()` 不清 `components`，所以同帧能读到，但这条顺序不能依赖。[Repair](../../../raw/current/scripts/components/boatleak.lua#L62-L94)、[满血短路](../../../raw/current/scripts/components/repairable.lua#L106-L115)
- 陆地或防漏船上的漏点会自己消失：prefab 下一帧 `checkforleakimmune` 打警告并 `Remove`，所以给 `leakproof` 船加动态漏点等于白刷。[检查](../../../raw/current/scripts/prefabs/boat_leak.lua#L47-L54)
- 动态漏点表只增不减：`leak_indicators_dynamic` 有 5 处 `table.insert`、0 处删除，实体销毁后留下无效引用；`UpdateHealth` 用 `IsValid()` 跳过（不报错），但表会随游戏时长变长。[插入](../../../raw/current/scripts/prefabs/boat.lua#L198)、[消费](../../../raw/current/scripts/components/hullhealth.lua#L68-L95)
- `SetBoat` 必须在首次 `SetState` 之前：它才把船的 `leak_build`／`leak_build_override` 抄进来，晚调会先用默认 `boat_leak_build` 播一帧。[L263-L273](../../../raw/current/scripts/components/boatleak.lua#L263-L273)
- 读档晚一帧：`OnLoad` 把 `SetBoat`／`SetState` 推迟到 task 0（要等平台关系归位），找不到平台则直接 `Remove`。[OnLoad](../../../raw/current/scripts/components/boatleak.lua#L296-L321)

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 锚点重定位 3 处（actions.lua REPAIR_LEAK／UNPATCH、tuning.lua 海带时限）；结论修订：无，仅锚点迁移。依据：枢纽 diff 审阅＋变更清单命中 actions.lua、componentactions.lua、tuning.lua，其余 sources 文件未变更。
- 源码审阅：2026-10-21，steam-buildid:25643519。覆盖构造与 `isdynamic` 属性、状态机全部分支、`Repair`、`ChangeToRepaired`、`SetRepairedTime`／到期、`SetPlugged`、`SetBoat`、`IsFinishedSpawning`、`OnSave`／`OnLoad`／`LongUpdate`；核对唯一宿主 prefab `boat_leak`、hullhealth 固定点生成、boat 的 `spawnnewboatleak` 动态生成、`REPAIR_LEAK`／`UNPATCH` 与 `repairable` 交互；未覆盖表现层与多人同步。
- 游戏运行：未运行；本页结论均来自源码静态核验，不含手感与数值实测。
- 结构检查：`validate_page.py` 通过（0 issue；105 行、12228 字节，低于 180 行／14 KiB）；`validate_source_links.py --strict` 通过（36 处引用、0 error／0 warning）；正文 Lua 示例另存 `output/group110-boatleak.lua`，`parser_probe --syntax-only` 通过（只查 Lua 5.1 语法，未执行）。

相关：[[reference/components/hullhealth]]、[[reference/components/boatpatch]]、[[reference/components/boatphysics]]。
