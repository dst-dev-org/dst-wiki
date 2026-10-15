---
title: "battery"
created: "2026-10-22"
updated: "2026-10-28"
type: component
summary: "为取电动作提供电源可用性、部分充电倍率与成功后的消费回调；不自行保存或输送电量。"
aliases: ["取电源", "避雷针取电", "电源消费回调", "battery"]
tags: [component, callback, energy, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/battery.lua", "components/batteryuser.lua", "prefabs/lightningrod.lua", "prefabs/winona_battery_low.lua", "actions.lua", "componentactions.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# battery

## 快速判断

- 职责与适用：挂在被取电的电源实体上，委托宿主管可用性、部分供电和扣费；接收端用 [[reference/components/batteryuser]]。组件没有电量字段、存读或周期供电任务。[实现](../../../raw/current/scripts/components/battery.lua#L1-L49)
- 优先使用：正常取电走 `batteryuser:ChargeFrom(source)`；`battery:OnUsed` 只是消费回调，不会替接收者充电。[调用顺序](../../../raw/current/scripts/components/batteryuser.lua#L31-L48)
- 重要副作用：接收回调成功后才调用电源 `OnUsed`，实际放电／扣燃料由宿主完成；无可用性回调默认允许，无消费回调不扣费。[默认行为](../../../raw/current/scripts/components/battery.lua#L35-L47)
- 修改边界：原避雷针与低级发电机在主模拟端挂组件；`battery` 标签用于发现动作，不证明此刻有电。薇诺娜设备持续供电另走 `circuitnode` 周期任务。[宿主](../../../raw/current/scripts/prefabs/lightningrod.lua#L173-L194) [动作发现](../../../raw/current/scripts/componentactions.lua#L179-L183) [电路链](../../../raw/current/scripts/prefabs/winona_battery_low.lua#L88-L109)

## 关键接口

| 接口／字段 | 签名与约束 | 条件、作用与证据 |
|---|---|---|
| `SetCanBeUsedFn(fn)` | `fn(source,user,mult) → success,reason` | `CanBeUsed(user,mult)` 原样返回回调结果；未设置时返回 true。`user` 是实际接收实体，可能是物品。[实现](../../../raw/current/scripts/components/battery.lua#L16-L18) [检查](../../../raw/current/scripts/components/battery.lua#L35-L41) [动作选取](../../../raw/current/scripts/actions.lua#L5858-L5874) |
| `SetOnUsedFn(fn)` | `fn(source,user,mult)` | `OnUsed(user,mult)` 调用它；返回值被忽略，不可用它拒绝已完成的充电。[实现](../../../raw/current/scripts/components/battery.lua#L20-L22) [消费](../../../raw/current/scripts/components/battery.lua#L43-L47) |
| `SetResolvePartialChargeMultFn(fn)` | `fn(source,user,mult) → mult` | 接收者给出倍率且允许部分充电时才调用；应返回数值，未设置／返回 nil 或 false 时沿用传入倍率。[实现](../../../raw/current/scripts/components/battery.lua#L24-L33) [条件](../../../raw/current/scripts/components/batteryuser.lua#L31-L36) |
| `OnRemoveFromEntity()` | 移除组件时 | 只删 `battery` 标签；宿主的燃料、任务、光效和存读由宿主维护。[实现](../../../raw/current/scripts/components/battery.lua#L12-L14) |

`mult` 是协议倍率，不是统一的秒数或电量格数；nil 与数值走宿主不同分支，Lua 的 0 仍为真。动作服务端先验证 target 的 `battery`，优先选择库存物品的 `batteryuser`，否则选择执行者。[动作](../../../raw/current/scripts/actions.lua#L5858-L5874) [完整协议](../../../raw/current/scripts/components/batteryuser.lua#L31-L48)

### 原版宿主与消费边界

- 源文件 `prefabs/lightningrod.lua` 实际注册 `lightning_rod`。其可用电量是随三天衰减的一份电荷，并非三次取电；无倍率成功使用会完全放电，数值倍率可扣部分电荷。放空会取消电弧任务、停止日期监听并关灯，保存由 prefab 的 OnSave／OnLoad 负责。[注册](../../../raw/current/scripts/prefabs/lightningrod.lua#L208) [检查／消费](../../../raw/current/scripts/prefabs/lightningrod.lua#L100-L132) [放电](../../../raw/current/scripts/prefabs/lightningrod.lua#L42-L63) [保存](../../../raw/current/scripts/prefabs/lightningrod.lua#L81-L92)
- `winona_battery_low` 同时挂 `battery` 和 `circuitnode`：取电按宿主 `CanBeUsedAsBattery → UseAsBattery → ConsumeBatteryAmount` 扣费，倍率可缩放；持续电路供电则每 0.5 秒遍历节点调用 `AddBatteryPower`。改 `battery` 回调不会替代电路任务。[挂载](../../../raw/current/scripts/prefabs/winona_battery_low.lua#L935-L946) [取电](../../../raw/current/scripts/prefabs/winona_battery_low.lua#L200-L241) [周期供电](../../../raw/current/scripts/prefabs/winona_battery_low.lua#L88-L109) [注册](../../../raw/current/scripts/prefabs/winona_battery_low.lua#L1134)

## 典型接入

### 仅禁止夜棍从避雷针取电，保留其他接收者与原放电逻辑

目标为模组根 `modmain.lua`，由模组入口加载；不增加资源，沿用真实宿主 `lightning_rod`。只包装服务端可用性检查，原 `onused` 和部分倍率处理保留；物品取电时 `user` 就是夜棍，不能当作持有者。[原挂载](../../../raw/current/scripts/prefabs/lightningrod.lua#L191-L194) [动作选取](../../../raw/current/scripts/actions.lua#L5863-L5873)

```lua
AddPrefabPostInit("lightning_rod", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local battery = inst.components.battery
    if battery == nil then return end
    local oldcanbeused = battery.canbeused
    battery:SetCanBeUsedFn(function(source, user, mult)
        if user ~= nil and user.prefab == "nightstick" then
            return false
        end
        if oldcanbeused ~= nil then
            return oldcanbeused(source, user, mult)
        end
        return true
    end)
end)
```

`nightstick` 是需替换时修改的接收物品注册名；失败不提供专用台词，不删除原交互提示。WX-78 等其他接收者仍走原检查／扣费，禁止只覆盖此电源，不影响夜棍从其他电源取电。停用模组并重载会恢复原检查；不改存档电量、不新增任务或监听，无额外清理对象。未做游戏运行验证。

## 常见陷阱与限制

- **只加组件就得到有限电池**：默认始终可用且不消费，必须由宿主配置检查和消费；不要直接调用 `OnUsed` 代替完整接收流程。[默认行为](../../../raw/current/scripts/components/battery.lua#L35-L47) [完整流程](../../../raw/current/scripts/components/batteryuser.lua#L31-L48)
- **在消费回调返回 false 撤销充电**：接收端已先完成变化，`OnUsed` 的结果不参与判定。提前在 `CanBeUsed` 拒绝，或由接收回调返回 false；组件没有事务回滚。[流程](../../../raw/current/scripts/components/batteryuser.lua#L36-L47)
- **移除 battery 就停止薇诺娜电路**：组件只清标签；低级发电机的燃料耗尽会另行 `StopBattery`，烧毁分支另行断开电路。电路生命周期不能由此组件代管。[组件移除](../../../raw/current/scripts/components/battery.lua#L12-L14) [燃料耗尽](../../../raw/current/scripts/prefabs/winona_battery_low.lua#L434-L441) [烧毁](../../../raw/current/scripts/prefabs/winona_battery_low.lua#L395-L411)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 中变更文件 actions.lua、componentactions.lua（battery.lua／batteryuser.lua／lightningrod.lua／winona_battery_low.lua 本版未变）对照 build-26148036 更新报告核验：CHARGE_FROM 定义与 fn、SCENE.battery／EQUIPPED.batteryuser 候选均无语义变化，无 nomagiccast／禁长按涉及；锚点重定位 3 处、结论无修订；未覆盖：运行验证。
- 源码审阅：2026-10-22，Steam buildid 25643519；覆盖组件全部实现、接收顺序、CHARGE_FROM、动作标签发现、避雷针注册／电荷／存读／放电清理，以及低级发电机取电与电路任务分工。未全面审校薇诺娜技能、燃料层级与高级发电机。
- 结构检查：本页格式、文件大小、来源与本地源码锚点；本页完整 `modmain.lua` 示例经 Full Moon Lua 5.1 syntax-only 检查。
- 游戏运行：未运行；需实测夜棍拒绝、WX-78 正常取电、无电失败、提示与多模组回调组合。
