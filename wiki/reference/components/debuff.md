---
title: "debuff"
created: "2026-10-12"
updated: "2026-10-28"
type: component
summary: "定义 Buff 实体的施加、重复施加与撤销，用计时器和来源修改器组装可恢复的限时效果。"
aliases: ["自定义Buff", "三十秒加速", "限时加速效果", "Buff撤销", "debuff"]
tags: [component, mechanism, callback, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/debuff.lua", "components/debuffable.lua", "components/timer.lua", "components/locomotor.lua", "prefabs/foodbuffs.lua", "prefabs/healthregenbuff.lua", "entityscript.lua", "mainfunctions.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# debuff

## 快速判断

- 职责与适用：挂在**效果实体**上，回调实现正／负效果；目标端集合用 [[reference/components/debuffable]]，不要把 debuff 挂到人物上当效果管理器。[debuff.lua:35](../../../raw/current/scripts/components/debuff.lua#L35 [debuffable.lua:72](../../../raw/current/scripts/components/debuffable.lua#L72
- 优先使用：工厂设置 Attached／Extended／Detached 回调；通过目标 AddDebuff 接入，到期用效果的 debuff:Stop()。[debuff.lua:12](../../../raw/current/scripts/components/debuff.lua#L12 [debuff.lua:28](../../../raw/current/scripts/components/debuff.lua#L28
- 重要副作用：同键只进 Extended，不自动刷新、叠加或重建；Detached 前 name／target 已清空，使用回调传入的 target。有效 Detached 不会被管理器自动补 Remove。[debuff.lua:44](../../../raw/current/scripts/components/debuff.lua#L44 [debuffable.lua:95](../../../raw/current/scripts/components/debuffable.lua#L95 [debuffable.lua:110](../../../raw/current/scripts/components/debuffable.lua#L110
- 修改边界：本页是服务端隐藏机制实体，无动画／网络实体／状态图标；计时、死亡清理、来源撤销和显示均须按需实现，不由组件自动提供。[foodbuffs.lua:150](../../../raw/current/scripts/prefabs/foodbuffs.lua#L150 [foodbuffs.lua:182](../../../raw/current/scripts/prefabs/foodbuffs.lua#L182

## 关键接口

| 接口／字段 | 参数或返回 | 条件、作用与证据 |
|---|---|---|
| `SetAttachedFn(fn)` | `(effect, target, followsymbol, followoffset, data, buffer)` | 单回调；管理器先登记，再设 name／target 并调用。SetParent／效果应用由回调负责。[debuff.lua:35](../../../raw/current/scripts/components/debuff.lua#L35 |
| `SetExtendedFn(fn)` | 同上 | 已有键再次 AddDebuff 调用；是否刷新时间、更新强度由此回调明确规定。[debuff.lua:53](../../../raw/current/scripts/components/debuff.lua#L53 |
| `SetDetachedFn(fn)` | `(effect, target)` | 撤销来源／监听，再删效果或处理退出动画；target 已不能从组件字段取回。[debuff.lua:44](../../../raw/current/scripts/components/debuff.lua#L44 [foodbuffs.lua:173](../../../raw/current/scripts/prefabs/foodbuffs.lua#L173 |
| `SetChangeFollowSymbolFn(fn)` | `(effect, target, symbol, offset)` | 接管理器定位参数变化；不自动挂动画符号。[debuff.lua:60](../../../raw/current/scripts/components/debuff.lua#L60 |
| `Stop()` | 无返回 | 已附着时请求目标 RemoveDebuff(name)；未附着无动作，不直接删除自身。[debuff.lua:28](../../../raw/current/scripts/components/debuff.lua#L28 |
| `name`／`target`／`keepondespawn` | 管理键／目标实体或 nil／可选标志 | 前两项由管理器维护；keepondespawn 构造器未赋值，真值供 RemoveOnDespawn 跳过。不是自动跨分片／重选正确性的保证。[debuff.lua:1](../../../raw/current/scripts/components/debuff.lua#L1 [debuffable.lua:39](../../../raw/current/scripts/components/debuffable.lua#L39 |

## 典型接入

### 三十秒加速效果并成对撤销

新增目标模组 `scripts/prefabs/my_mod_speedbuff.lua`，完整工厂如下。无额外美术资源；在 `modmain.lua` 注册与施加的方法见 [[reference/components/debuffable#吃胡萝卜施加或刷新加速效果]]，本工厂不能单独启用玩法。Prefab 文件沿用游戏全局函数，modmain 的 GLOBAL 写法不搬进工厂。沿用原 foodbuffs 的服务端隐藏实体／计时／死亡停止，机制用 [[reference/components/locomotor#关键接口]] 的独立来源倍率。[mainfunctions.lua:171](../../../raw/current/scripts/mainfunctions.lua#L171 [foodbuffs.lua:150](../../../raw/current/scripts/prefabs/foodbuffs.lua#L150 [foodbuffs.lua:182](../../../raw/current/scripts/prefabs/foodbuffs.lua#L182 [locomotor.lua:490](../../../raw/current/scripts/components/locomotor.lua#L490

```lua
local DURATION = 30
local SPEED_MULT = 1.2
local MODIFIER_KEY = "my_mod_speed"
local TIMER_NAME = "my_mod_speed_over"

local function StopEffect(inst)
    inst.components.debuff:Stop()
end

local function OnAttached(inst, target)
    if target.components.locomotor == nil then
        StopEffect(inst)
        return
    end
    inst.entity:SetParent(target.entity)
    inst.Transform:SetPosition(0, 0, 0)
    inst:ListenForEvent("death", function() StopEffect(inst) end, target)
    inst:ListenForEvent("onremove", function() StopEffect(inst) end, target)
    target.components.locomotor:SetExternalSpeedMultiplier(
        inst, MODIFIER_KEY, SPEED_MULT)
end

local function OnExtended(inst)
    local timer = inst.components.timer
    timer:StopTimer(TIMER_NAME)
    timer:StartTimer(TIMER_NAME, DURATION)
end

local function OnDetached(inst, target)
    if target ~= nil and target.components.locomotor ~= nil then
        target.components.locomotor:RemoveExternalSpeedMultiplier(
            inst, MODIFIER_KEY)
    end
    inst:Remove()
end

local function fn()
    local inst = CreateEntity()
    if not TheWorld.ismastersim then
        inst:DoTaskInTime(0, inst.Remove)
        return inst
    end
    inst.entity:AddTransform()
    inst.entity:Hide()
    inst.persists = false
    inst:AddTag("CLASSIFIED")

    inst:AddComponent("debuff")
    inst.components.debuff:SetAttachedFn(OnAttached)
    inst.components.debuff:SetExtendedFn(OnExtended)
    inst.components.debuff:SetDetachedFn(OnDetached)
    inst.components.debuff.keepondespawn = true

    inst:AddComponent("timer")
    inst.components.timer:StartTimer(TIMER_NAME, DURATION)
    inst:ListenForEvent("timerdone", function(effect, data)
        if data ~= nil and data.name == TIMER_NAME then StopEffect(effect) end
    end)
    return inst
end

return Prefab("my_mod_speedbuff", fn)
```

30 是游戏秒，1.2 是外部速度倍率，和其他来源相乘；同管理键重复施加只刷新到三十秒，不累计倍数／时长。更换文件／Prefab／管理键时同步注册与施加页。到期、目标死亡／删除或主动 RemoveDebuff 都撤销本来源；实例删除同时清理它注册的事件监听，其他效果不受这对来源／键的显式撤销影响。[locomotor.lua:480](../../../raw/current/scripts/components/locomotor.lua#L480 [locomotor.lua:515](../../../raw/current/scripts/components/locomotor.lua#L515 [entityscript.lua:1741](../../../raw/current/scripts/entityscript.lua#L1741

计时只在工厂首次创建时启动，**不在 Attached 重置**：恢复效果先 SpawnSaveRecord → SetPersistData → timer OnLoad，随后管理器 AttachTo 重施来源，因此保留保存剩余。keepondespawn=true 使普通玩家清理跳过此效果；目标管理器负责保存它，persists=false 不等于这条保存链不保存。跨分片、重选与客户端预测另行实测。[mainfunctions.lua:464](../../../raw/current/scripts/mainfunctions.lua#L464 [mainfunctions.lua:503](../../../raw/current/scripts/mainfunctions.lua#L503 [debuffable.lua:126](../../../raw/current/scripts/components/debuffable.lua#L126 [debuffable.lua:139](../../../raw/current/scripts/components/debuffable.lua#L139 [timer.lua:135](../../../raw/current/scripts/components/timer.lua#L135

### 原版模式选择

foodbuffs 的 buff_attack 等是来源化属性＋到期撤销，Extended 明确重计时；healthregenbuff 另建周期回血任务，Extend 还取消／重排周期。要“持续回血”，到期 timer 只负责终止，周期收益另查 [[reference/components/health#关键接口]]，不要把一个到期事件当周期循环。[foodbuffs.lua:163](../../../raw/current/scripts/prefabs/foodbuffs.lua#L163 [foodbuffs.lua:219](../../../raw/current/scripts/prefabs/foodbuffs.lua#L219 [healthregenbuff.lua:11](../../../raw/current/scripts/prefabs/healthregenbuff.lua#L11 [healthregenbuff.lua:26](../../../raw/current/scripts/prefabs/healthregenbuff.lua#L26

## 常见陷阱与限制

- **只写 Attached，不写 Extended／Detached**：重吃不刷新，撤销后效果实体／属性可能残留。为自己的工厂成对实现；修改原效果先保留旧回调，setter 是单槽。[debuff.lua:12](../../../raw/current/scripts/components/debuff.lua#L12 [debuff.lua:53](../../../raw/current/scripts/components/debuff.lua#L53
- **直接 effect:Remove 当撤销**：管理器只移集合并通知，不调 Detached。本例 locomotor 有 source 删除兜底，但其他属性不保证；正常结束走 Stop／目标 RemoveDebuff，移除 debuff 组件前也先 Stop。[debuffable.lua:77](../../../raw/current/scripts/components/debuffable.lua#L77 [locomotor.lua:501](../../../raw/current/scripts/components/locomotor.lua#L501
- **在 Attached 从 data 恢复必要状态**：管理器恢复时不传原施加 data／buffer；可变强度需由效果自己的 OnSave／OnLoad 保存，再附着，不只保存回调输入。恢复顺序和完整边界见 [[reference/components/debuffable#保存恢复与移除]]。[debuffable.lua:139](../../../raw/current/scripts/components/debuffable.lua#L139
- **倍率无效仍称施加成功**：目标需要 locomotor；本例缺组件立即停止。骑乘、幽灵、模式重建或预测是否实际消费此来源需查对应消费者，不能把工厂语法通过当运行正确。[locomotor.lua:178](../../../raw/current/scripts/components/locomotor.lua#L178

## 验证记录

- 版本增量复核 26148036（2026-10-25）：锚点重定位 4 处（mainfunctions LoadPrefabFile／SpawnSaveRecord／SetPersistData +26、entityscript onremove +3）；无结论修订，仅锚点迁移——debuff.lua、debuffable.lua、timer.lua、foodbuffs.lua、healthregenbuff.lua 未变；locomotor.lua 本版改动在 PreviewAction/PushAction/OnUpdate（845+），本页引用的 178-515 行未变。依据：全局枢纽 diff 审阅＋变更清单命中 locomotor.lua、entityscript.lua、mainfunctions.lua。
- 源码审阅：2026-10-12，Steam buildid 25643519；覆盖回调／管理协作、隐藏原版工厂、到期／重复施加／撤销、来源速度和保存恢复及发布工厂。不代表全 Buff 类型、动画符号或数据／实体引用保存均已审校。
- 结构检查：元数据、证据行、链接／章节、大小与登记；发布工厂及施加文件分别经 Full Moon 3.0.0／Lua 5.1 语法和原字节往返检查。
- 游戏运行：未运行；首次／刷新／到期／死亡撤销、存档剩余、退出重连、玩家预测／骑乘、多人和其他模组待实测。

相关：[[reference/components/timer#关键接口]]；[[reference/components/debuffable#关键接口]]。
