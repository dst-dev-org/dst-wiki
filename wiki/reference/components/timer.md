---
title: "timer"
created: "2026-10-12"
updated: "2026-10-28"
type: component
summary: "用具名秒数倒计时触发完成事件，支持刷新、暂停与保存，明确取消和离线推进边界。"
aliases: ["具名倒计时", "定时关闭箱子", "刷新计时器", "计时器暂停", "timer"]
tags: [component, mechanism, event, lifecycle, post-init, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/timer.lua", "components/container.lua", "prefabs/treasurechest.lua", "prefabs/foodbuffs.lua", "entityscript.lua", "mainfunctions.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# timer

## 快速判断

- 职责与适用：实体上的多个具名倒计时，以秒为单位；到点发宿主 `timerdone`。不自带周期效果、技能资格或 Buff 管理；技能充能用 [[reference/components/rechargeable]]，限时效果组合 [[reference/components/debuff]]。[timer.lua:31](../../../raw/current/scripts/components/timer.lua#L31
- 优先使用：首次 StartTimer，刷新选 StopTimer → StartTimer；只改现有剩余用 SetTimeLeft，保留原暂停状态。[timer.lua:36](../../../raw/current/scripts/components/timer.lua#L36 [timer.lua:104](../../../raw/current/scripts/components/timer.lua#L104
- 重要副作用：同名再次 StartTimer 被拒绝；完成时先删计时记录，再发事件，监听器可重启同名计时。[timer.lua:31](../../../raw/current/scripts/components/timer.lua#L31 [timer.lua:37](../../../raw/current/scripts/components/timer.lua#L37
- 修改边界：本页服务端计时；组件没有网络同步或客户端倒计时展示。只需不保存的单次任务可直接用 DoTaskInTime；有保存需求用本组件并确保读档工厂仍添加它。[timer.lua:122](../../../raw/current/scripts/components/timer.lua#L122 [entityscript.lua:1547](../../../raw/current/scripts/entityscript.lua#L1547

## 关键接口

| 接口 | 参数或返回 | 条件、作用与证据 |
|---|---|---|
| `StartTimer(name, time, paused?, initialtime_override?)` | 唯一名称、秒数；无成功返回值 | 建任务与记录，可初始暂停；不校验／截断负 time，接入传非负秒数，name 用模组唯一字符串。[timer.lua:36](../../../raw/current/scripts/components/timer.lua#L36 |
| `TimerExists(name)`／`IsPaused(name)` | boolean | 暂停仍存在；无该名称时分别 false／false。[timer.lua:27](../../../raw/current/scripts/components/timer.lua#L27 [timer.lua:68](../../../raw/current/scripts/components/timer.lua#L68 |
| `StopTimer(name)` | 缺名称也可调用 | 取消任务、删记录；不发 timerdone。[timer.lua:56](../../../raw/current/scripts/components/timer.lua#L56 |
| `PauseTimer(name)`／`ResumeTimer(name)` | Resume 真正恢复返回 true，否则 nil | Pause 保留剩余并取消任务；Resume 仅处理暂停项。[timer.lua:72](../../../raw/current/scripts/components/timer.lua#L72 [timer.lua:84](../../../raw/current/scripts/components/timer.lua#L84 |
| `GetTimeLeft(name)`／`SetTimeLeft(name, time)` | 秒数或 nil／无返回 | getter 的运行项可能短暂为负；setter 仅改已有项并截断到零，暂停项不恢复，运行项重排任务。[timer.lua:95](../../../raw/current/scripts/components/timer.lua#L95 [timer.lua:104](../../../raw/current/scripts/components/timer.lua#L104 |
| `GetTimeElapsed(name)` | 秒数或 nil | 初始时长减当前剩余；SetTimeLeft 不改 initial_time，延长后可能为负，不是独立累计运行秒数。[timer.lua:116](../../../raw/current/scripts/components/timer.lua#L116 |

## 生命周期

OnSave 记剩余、暂停、初始时长；OnLoad 对保存的同名项 Stop 后重建，不清理保存表之外的工厂预设项。宿主先建工厂／PostInit，再加载组件数据；不要在读档后的其他回调无条件重启全部时长。[timer.lua:122](../../../raw/current/scripts/components/timer.lua#L122 [timer.lua:135](../../../raw/current/scripts/components/timer.lua#L135 [mainfunctions.lua:388](../../../raw/current/scripts/mainfunctions.lua#L388 [mainfunctions.lua:503](../../../raw/current/scripts/mainfunctions.lua#L503

LongUpdate(dt) 对**所有项，包括暂停项**扣 dt，再通过 SetTimeLeft 截断；运行项重排任务，不在该循环同步发完成事件。主动移除 timer 组件会取消任务；宿主 Remove 则取消其待执行任务，不发 timerdone，玩法清理不能只依赖完成事件。[timer.lua:144](../../../raw/current/scripts/components/timer.lua#L144 [timer.lua:6](../../../raw/current/scripts/components/timer.lua#L6 [entityscript.lua:1741](../../../raw/current/scripts/entityscript.lua#L1741

## 典型接入

### 木箱打开十秒后自动关闭并刷新倒计时

目标模组 `modmain.lua`；只改原 `treasurechest`，复用资源与开关动画。监听 onopen，不替换 container 的原 onopenfn；每个新开箱者触发 onopen，刷新整箱共用十秒，最后一人关闭时取消。[container.lua:656](../../../raw/current/scripts/components/container.lua#L656 [container.lua:719](../../../raw/current/scripts/components/container.lua#L719 [treasurechest.lua:150](../../../raw/current/scripts/prefabs/treasurechest.lua#L150 [treasurechest.lua:766](../../../raw/current/scripts/prefabs/treasurechest.lua#L766 [modutil.lua:593](../../../raw/current/scripts/modutil.lua#L593

```lua
local TIMER_NAME = "my_mod_chest_autoclose"
local CLOSE_DELAY = 10

AddPrefabPostInit("treasurechest", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    if inst.components.container == nil then return end
    if inst.components.timer == nil then inst:AddComponent("timer") end

    inst:ListenForEvent("onopen", function(chest)
        local timer = chest.components.timer
        timer:StopTimer(TIMER_NAME)
        timer:StartTimer(TIMER_NAME, CLOSE_DELAY)
    end)
    inst:ListenForEvent("onclose", function(chest)
        local container = chest.components.container
        if container == nil or not container:IsOpen() then
            chest.components.timer:StopTimer(TIMER_NAME)
        end
    end)
    inst:ListenForEvent("timerdone", function(chest, data)
        if data ~= nil and data.name == TIMER_NAME then
            local container = chest.components.container
            if container ~= nil then container:Close() end
        end
    end)
end)
```

Close() 无 doer 时遍历所有开箱者，沿用正常界面／回调／事件清理；不是只关触发者。计时名称替换为模组唯一键，10 为游戏秒；同一玩家已经打开时再次 Open 不等于新开箱事件。[container.lua:674](../../../raw/current/scripts/components/container.lua#L674 [container.lua:617](../../../raw/current/scripts/components/container.lua#L617

未到点的计时可保存；案例不靠计时器保存开箱 UI，恢复后当前无人打开时 Close 无开箱者可关。停用功能应移除此键及本功能监听，不删整个 timer；箱子销毁后宿主任务取消。刷新写法也见原食物 Buff 的扩展回调，限时效果的撤销逻辑读 [[reference/components/debuff#三十秒加速效果并成对撤销]]。[timer.lua:122](../../../raw/current/scripts/components/timer.lua#L122 [foodbuffs.lua:163](../../../raw/current/scripts/prefabs/foodbuffs.lua#L163

## 常见陷阱与限制

- **反复 Start 当刷新**：已有名称只打印提示并返回，剩余不变；完整重计时 Stop → Start，单纯调整剩余用 SetTimeLeft。[timer.lua:37](../../../raw/current/scripts/components/timer.lua#L37
- **暂停等于不受任何推进**：普通任务被取消，但 LongUpdate 仍扣暂停项；需要自定义冻结策略时核验调用者，不承诺暂停能抵御所有离线推进。[timer.lua:72](../../../raw/current/scripts/components/timer.lua#L72 [timer.lua:144](../../../raw/current/scripts/components/timer.lua#L144
- **零秒等于立即回调、取消等于完成**：SetTimeLeft(0) 排任务，不直接发事件；Stop／移组件不发完成。监听检查 data.name，避免处理其他功能的 timerdone。[timer.lua:104](../../../raw/current/scripts/components/timer.lua#L104 [timer.lua:31](../../../raw/current/scripts/components/timer.lua#L31
- **TransferComponent 是移动或覆盖**：目的地须已有 timer；仅尝试复制，旧项仍在，目标同名项被 Start 拒绝。迁移前定义冲突与旧实体清理策略。[timer.lua:150](../../../raw/current/scripts/components/timer.lua#L150

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 处理范围：锚点重定位 4 处（entityscript／mainfunctions 行号迁移）；结论修订：无，仅锚点迁移。
- 依据：全局枢纽 hunk 地图（R-08/R-09）＋变更清单命中 entityscript.lua、mainfunctions.lua。

- 源码审阅：2026-10-12，Steam buildid 25643519；覆盖上表、完成／取消顺序、保存／暂停／LongUpdate／复制与箱子消费者及发布案例。未审校所有宿主追赶时机、世界设置计时器或引擎调度细节。
- 结构检查：元数据、证据行、链接／章节、大小与登记；发布 Lua 经 Full Moon 3.0.0／Lua 5.1 语法和原字节往返检查。
- 游戏运行：未运行；箱子多开者／读档／销毁、暂停追赶与其他模组同事件协作待实测。

相关：[[reference/components/container#关键接口]]；[[reference/components/debuffable#保存恢复与移除]]。
