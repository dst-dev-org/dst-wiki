---
title: firedetector：火情扫描与灭火器调度
created: 2026-10-22
updated: "2026-10-28"
type: component
summary: 调整灭火器的目标扫描与紧急响应，区分发现目标、发射雪球和实际保护效果。
aliases: [firedetector, 灭火器扫描, 只灭火不浇植物, 紧急灭火]
tags: [component, building, callback, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: [components/firedetector.lua, prefabs/firesuppressor.lua, stategraphs/SGfiresuppressor.lua, prefabs/waterprojectiles.lua, components/wateryprotection.lua, components/machine.lua, tuning.lua]
reliability: verified
validation: source-reviewed
status: active
---

# firedetector：火情扫描与灭火器调度

## 快速判断

- 职责与适用：按范围选火焰、阴燃或可枯萎植物，把目标**位置**交给回调；组件本身不灭火、不生成雪球、不扣燃料。[扫描与回调](../../../raw/current/scripts/components/firedetector.lua#L119-L170)
- 优先使用：只修改原版灭火器的选靶条件时，调整 `firedetector` 字段并保留宿主回调；原版注册名是 `firesuppressor`。[组件组装](../../../raw/current/scripts/prefabs/firesuppressor.lua#L351-L364)、[注册](../../../raw/current/scripts/prefabs/firesuppressor.lua#L488-L490)
- 重要副作用：同一目标被选中后抑制重复选择 2 秒；这是扫描冷却，不代表已命中或成功灭火。[登记](../../../raw/current/scripts/components/firedetector.lua#L108-L114)
- 修改边界：原版组件在主模拟端添加。扫描、SG、燃料和保护效果分别协作，不要用一个新回调覆盖整条效果链。[端边界](../../../raw/current/scripts/prefabs/firesuppressor.lua#L317-L321)、[燃料与启停](../../../raw/current/scripts/prefabs/firesuppressor.lua#L85-L110)

## 关键接口

| 接口／字段 | 参数或约束 | 条件、作用与证据 |
|---|---|---|
| `range`、`detectPeriod` | 距离、秒；默认来自 TUNING | 本快照默认 15、1；周期任务创建时读取周期，之后改周期须重新激活；范围在每次扫描读取。[初始化](../../../raw/current/scripts/components/firedetector.lua#L15-L39)、[默认值](../../../raw/current/scripts/tuning.lua#L2934-L2935)、[扫描与激活](../../../raw/current/scripts/components/firedetector.lua#L139-L170) |
| `fireOnly` | 真值启用；默认未赋值 | 普通模式候选由 `witherable/fire/smolder` 缩为 `fire/smolder`；不会改变紧急模式的筛选。[候选标签](../../../raw/current/scripts/components/firedetector.lua#L1-L5)、[普通扫描](../../../raw/current/scripts/components/firedetector.lua#L139-L165) || `SetOnFindFireFn(fn)` | `fn(inst, targetpos)` | 回调收到位置，不是目标实体；普通扫描优先燃烧、阴燃、可枯萎、可恢复植物。植物保护只排除枯萎／复壮分支，不阻止前面的燃烧／阴燃选择。[设置](../../../raw/current/scripts/components/firedetector.lua#L52-L54)、[评分](../../../raw/current/scripts/components/firedetector.lua#L119-L165) |
| `Activate(randomizedStartTime)` | 可选真值 | 取消旧模式并创建普通扫描；首次延迟 13 帧，可再随机延迟一个周期。[实现](../../../raw/current/scripts/components/firedetector.lua#L167-L170) |
| `ActivateEmergencyMode(randomizedStartTime)` | 可选真值 | 先进入监视/预警模式，监听候选的 `onburnt/onremove`；达到计数、预警时长且仍有火才转为紧急发射扫描。[升级条件](../../../raw/current/scripts/components/firedetector.lua#L215-L265)、[监视与激活](../../../raw/current/scripts/components/firedetector.lua#L268-L315) |
| `DetectFire()` | 无参数 | 仅已有扫描任务、且处于普通或已升级紧急模式时强制扫描；绕过 SG `busy` 门禁，不绕过目标冷却。[实现](../../../raw/current/scripts/components/firedetector.lua#L382-L390) |
| `Deactivate()` | 无参数 | 普通/已升级紧急模式会取消任务；仍在预警监视模式时只清预警计数，保留监视任务。[实现](../../../raw/current/scripts/components/firedetector.lua#L365-L377) |

### 扫描、发射与效果的分工

普通扫描跳过 `busy` SG；候选排除 `FX/NOCLICK/DECOR/INLIMBO/burnt/player/monster/shadow_fire`，26148036 起另排除 `gestaltflame`（发光火焰不进灭火器候选）。遇到燃烧目标立即选中，不保证比较所有燃烧目标；其他类型按分值选择。[筛选与评分](../../../raw/current/scripts/components/firedetector.lua#L119-L165)

原版 `OnFindFire` 在宿主睡眠时延迟调用宿主 `wateryprotection`；醒着时发 `putoutfire`，SG 只在机器开启且允许的状态接收，随后发射雪球并调用 `DetectFire()` 接续扫描。[宿主回调](../../../raw/current/scripts/prefabs/firesuppressor.lua#L43-L53)、[事件门禁](../../../raw/current/scripts/stategraphs/SGfiresuppressor.lua#L1-L11)、[发射节奏](../../../raw/current/scripts/stategraphs/SGfiresuppressor.lua#L224-L233)

`LaunchProjectile` 生成 `snowball` 并用 `complexprojectile:Launch` 投射；雪球命中才扩散 `wateryprotection` 并移除。它负责熄火、降温、抗枯萎和加冷冻值；`firedetector` 不直接执行这些效果。原版雪球没有配置正的 `addwetness`，不能把“保护植物”理解成增加潮湿或农田水分。[投射](../../../raw/current/scripts/prefabs/firesuppressor.lua#L25-L41)、[命中](../../../raw/current/scripts/prefabs/waterprojectiles.lua#L95-L99)、[雪球配置](../../../raw/current/scripts/prefabs/waterprojectiles.lua#L170-L192)、[效果实现](../../../raw/current/scripts/components/wateryprotection.lua#L21-L56)

### 启停、保存与清理

宿主开启时激活普通扫描并开始消耗燃料；关闭时停止消耗，有燃料则重新布防紧急模式。因此“关机”仍可能在紧急火情时自动开机；燃料耗尽通过 `machine:TurnOff()` 关闭。[启停](../../../raw/current/scripts/prefabs/firesuppressor.lua#L85-L123)、[耗尽](../../../raw/current/scripts/prefabs/firesuppressor.lua#L144-L146)

组件没有 `OnSave/OnLoad`；宿主由 `machine` 保存开关并在加载时调用启停回调，不能据此认为扫描目标、预警历史和冷却被保存。移除组件会取消扫描、紧急倒计时、目标事件监听与两秒任务；模式切换的 `Cancel` 不清 `detectedItems`，短时间内仍可能跳过此前目标。[机器保存恢复](../../../raw/current/scripts/components/machine.lua#L50-L61)、[取消与移除](../../../raw/current/scripts/components/firedetector.lua#L78-L106)

## 典型接入

### 原版灭火器只主动选择火焰与阴燃目标

在独立模组的完整 `modmain.lua` 放下列代码；只影响 `firesuppressor`，复用原版模型、资源、燃料、警报、SG 和雪球回调，无需新增资源。扫描候选收窄，植物若自身已带火焰/阴燃标签仍可入选，雪球范围效果也仍可能保护附近植物。

```lua
AddPrefabPostInit("firesuppressor", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    local detector = inst.components.firedetector
    if detector ~= nil then
        detector.fireOnly = true
    end
end)
```

这是主模拟端的运行配置，不替换原回调、不重建任务；宿主启停仍按原版执行。改回 `nil/false` 可恢复普通候选；删除代码并重新载入世界也会恢复默认。此例不关闭紧急布防，不改变雪球范围和效果。[字段消费](../../../raw/current/scripts/components/firedetector.lua#L139-L165)、[原版接入](../../../raw/current/scripts/prefabs/firesuppressor.lua#L351-L364)

## 常见陷阱与限制

- 直接调用 `DetectFire()` 不会启动停用组件，也不会在仅布防状态发射；需要由宿主启用正确模式。[调用门禁](../../../raw/current/scripts/components/firedetector.lua#L382-L390)
- 覆盖发现回调却不保留 `putoutfire`/睡眠路径，会出现“扫描正常但没有雪球或保护”；修改选靶优先改字段，改效果须分别考虑醒着的雪球和睡眠的宿主保护。[回调分支](../../../raw/current/scripts/prefabs/firesuppressor.lua#L43-L53)
- `Deactivate()` 不等于任意状态的彻底取消。若要删除自定义组件，走组件移除清理；不要把保留布防任务误判为泄漏。[停止分支](../../../raw/current/scripts/components/firedetector.lua#L365-L377)、[清理](../../../raw/current/scripts/components/firedetector.lua#L101-L106)
- `onendemergency(inst, level)` 的调用发生在 `Cancel` 把 `emergencyLevel` 清零之后，不能用这个参数恢复退出前等级。[取消](../../../raw/current/scripts/components/firedetector.lua#L78-L99)、[结束回调](../../../raw/current/scripts/components/firedetector.lua#L175-L180)

## 验证记录

- 版本增量复核 26148036（2026-10-25）：锚点重定位 1 处（tuning FIRE_DETECTOR_PERIOD/RANGE +14）；结论修订 1 处：NOTAGS 候选排除表新增 `gestaltflame`（firedetector.lua:1，发光火焰不再被灭火器扫描选中）。FIRE_DETECTOR_PERIOD=1／RANGE=15 值未变（tuning-hunk-map 5 改值键不含）。依据：tuning-hunk-map 键级审阅＋变更清单命中 components/firedetector.lua（本版仅 NOTAGS 一行改动）、tuning.lua；firesuppressor／SG／雪球链未变。
- 源码审阅：2026-10-22，Steam buildid 25643519；覆盖普通扫描排序、两秒抑制、紧急模式关键门禁、启停清理、宿主注册、SG/雪球/保护链与上述 `fireOnly` 接入；未逐项展开全部紧急调级接口。
- 游戏运行：未运行；未实测大火数量、睡眠状态、断燃料或联机表现。
- 结构与语法：本页通过页面格式和严格源码链接检查；完整 `modmain.lua` 通过 Lua 5.1 语法检查。检查不验证游戏效果。
