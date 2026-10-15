---
title: "machine"
created: "2026-10-20"
updated: "2026-10-28"
type: component
summary: "机器开关状态、回调、交互标签和短冷却；区分候选筛选、直接开关与宿主实际工作。"
aliases: ["灭火器开关冷却改为一秒", "机器开关", "开关重复回调", "machine"]
tags: [component, callback, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/machine.lua", "prefabs/firesuppressor.lua", "componentactions.lua", "actions.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# machine

## 快速判断

- 职责与适用：提供 ison、enabled、短交互冷却及开／关回调；探测、耗燃料、动画由宿主实现。原灭火器将回调接到 firedetector／fueled／SG。[组件](../../../raw/current/scripts/components/machine.lua#L17-L33) [宿主开机](../../../raw/current/scripts/prefabs/firesuppressor.lua#L100-L109)
- 优先使用：修改按钮重复操作间隔调 cooldowntime（秒）；实际开关用 TurnOn／TurnOff，并由调用入口校验场景。不能只改 ison 期待停止燃料或探测。[冷却](../../../raw/current/scripts/components/machine.lua#L67-L82) [开关](../../../raw/current/scripts/components/machine.lua#L86-L111)
- 重要副作用：每次调用都启动冷却、调回调、最后改状态并发 machineturnedon／machineturnedoff；不去重同状态调用，回调内读到的 ison 可能仍为旧值。[开机](../../../raw/current/scripts/components/machine.lua#L86-L92) [关机](../../../raw/current/scripts/components/machine.lua#L105-L111)
- 修改边界：服务端；组件方法不复查 enabled、燃料、冷却或放地条件，动作执行也仅判组件／当前状态。候选标签门禁不是服务端权限保证。[动作](../../../raw/current/scripts/actions.lua#L3285-L3302) [候选](../../../raw/current/scripts/componentactions.lua#L526-L541)

## 关键接口

| 接口／字段 | 条件、作用与证据 |
|---|---|
| `turnonfn`／`turnofffn` | fn(inst)，返回值被忽略；冷却已启动，状态与事件随后更新。返回 false 不撤销开关。[顺序](../../../raw/current/scripts/components/machine.lua#L86-L111) |
| `IsOn()`／`ison` | 默认 false；属性维护 turnedon 标签，直接赋值不执行工作回调或发开关事件。[标签](../../../raw/current/scripts/components/machine.lua#L1-L3) [方法](../../../raw/current/scripts/components/machine.lua#L114-L115) |
| `enabled`／`CanInteract()` | 默认 true；CanInteract 只检查 fueldepleted、持有但未装备的可装备物，以及 enabled==true，不检查冷却／groundonly，也不调用 caninteractfn。[实现](../../../raw/current/scripts/components/machine.lua#L95-L102) |
| `cooldowntime`／`StartCooldown()` | 默认 3 秒；正数时刷新任务，≤0 不启动新任务，**不会取消已有冷却**。结束只清 oncooldown，任务引用留存。[默认](../../../raw/current/scripts/components/machine.lua#L17-L26) [任务](../../../raw/current/scripts/components/machine.lua#L63-L75) |
| `StopCooldown()` | 有任务时取消、清引用与 oncooldown；不是改变开关状态。[方法](../../../raw/current/scripts/components/machine.lua#L78-L82) |
| `SetGroundOnlyMachine(bool)` | 维护 groundonlymachine 标签；候选检查持有／漂浮，不由 TurnOn 自动拒绝。两处候选条件亦有差异，不能机械等同。[属性](../../../raw/current/scripts/components/machine.lua#L13-L14) [场景候选](../../../raw/current/scripts/componentactions.lua#L526-L541) [另一候选](../../../raw/current/scripts/componentactions.lua#L2758-L2770) |
| `OnSave()`／`OnLoad(data)` | 只存 ison；加载先赋保存值，再调用 TurnOn／TurnOff，重放回调、事件和新冷却，不保存剩余秒数或 enabled。[存读](../../../raw/current/scripts/components/machine.lua#L50-L59) |

移除组件会取消冷却并清 turnedon／cooldown／groundonlymachine，却未清 enabled，也未调用 TurnOff；自定义宿主若需要停工作须另管清理。[移除](../../../raw/current/scripts/components/machine.lua#L36-L42)

## 典型接入

### 原灭火器开关交互间隔改为一秒

目标模组 `modmain.lua`，沿用原资源、燃料和状态机；原宿主配置 0.5 秒，本例只改后续冷却参数，不更换开／关回调。灭火器紧急模式、补燃料／耗尽也会直接调用 machine，不承诺一秒节流所有自动状态变化。[安装](../../../raw/current/scripts/prefabs/firesuppressor.lua#L335-L344) [自动调用](../../../raw/current/scripts/prefabs/firesuppressor.lua#L112-L151)

```lua
AddPrefabPostInit("firesuppressor", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    if inst.components.machine ~= nil then
        inst.components.machine.cooldowntime = 1
    end
end)
```

单位为秒，只影响此实例随后 StartCooldown，不修改 firedetector 的搜索周期、喷射频率或燃料消耗。已经排队的任务不因字段赋值改截止时刻；本例不主动 StopCooldown，保留当前交互状态。[任务](../../../raw/current/scripts/components/machine.lua#L67-L82)

参数不单独存档，加载工厂重新应用；停用并重载后恢复原工厂值。OnLoad 还会重放开关，扩展回调中的奖励／生成物必须自行避免重复。燃料机制另见 [[reference/components/fueled]]。

## 常见陷阱与限制

- **设 caninteractfn 就覆盖资格**：原灭火器确有该赋值，但当前 Machine:CanInteract 不读它，TURNON／TURNOFF 也不调用该方法。要加执行校验应修改真实被调用的入口并核验其他调用者。[赋值](../../../raw/current/scripts/prefabs/firesuppressor.lua#L335-L339) [组件](../../../raw/current/scripts/components/machine.lua#L95-L102) [动作](../../../raw/current/scripts/actions.lua#L3285-L3302)
- **enabled=false／冷却会阻止直接 TurnOn**：只是部分候选条件，直接调用仍执行；服务端自动流程与玩家操作要分别设计。[候选](../../../raw/current/scripts/componentactions.lua#L526-L541) [执行](../../../raw/current/scripts/components/machine.lua#L86-L92)
- **反复 TurnOff 是无副作用停机**：仍重复调工作回调、刷新冷却并发事件；需要幂等入口先查 IsOn，不能把读取字段当宿主工作状态的完整证明。[关机](../../../raw/current/scripts/components/machine.lua#L105-L115)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：actions.lua（R-01）、componentactions.lua（R-04）对照 build-26148036 更新报告逐条核验，锚点重定位 3 处、结论修订 0 处：TURNON 执行（actions.lua:3275-3292）、场景候选（componentactions.lua:526-541）与 INVENTORY 级候选（componentactions.lua:2746-2758）逐行无变化；未覆盖：运行验证。

- 源码审阅：2026-10-20，Steam buildid 25643519；覆盖完整 machine、两处候选、开关动作与灭火器安装／工作回调／自动调用。未全面审校紧急探测、SG、中断、所有机器宿主或客户端预测。
- 结构检查：本批格式、来源、链接／源码锚点、大小与登记；完整参数例经已有 Full Moon Lua 5.1 syntax-only，结果记本轮报告。
- 游戏运行：未运行；快速重复操作、燃料耗尽／补充、紧急模式、存读重放与模组冲突待实测。
