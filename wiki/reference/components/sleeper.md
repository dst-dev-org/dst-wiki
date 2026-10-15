---
title: "sleeper"
created: "2026-10-17"
updated: "2026-10-28"
type: component
summary: "控制生物的日常睡眠测试、催眠累积与醒来，协同 Brain 和状态图；区别离屏休眠和玩家床具睡眠。"
aliases: ["生物睡眠","催眠生物","夜行生物","睡眠抗性","sleeper"]
tags: [component, creature, lifecycle, callback, ai]
dst_build: "steam-buildid:26230496"
sources: ["components/sleeper.lua","prefabs/rabbit.lua","prefabs/pigman.lua","stategraphs/SGrabbit.lua","stategraphs/SGpig.lua","stategraphs/commonstates.lua","entityscript.lua","scheduler.lua","mainfunctions.lua","modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# sleeper

## 快速判断

- 职责与适用：生物端睡眠状态、睡／醒测试与 sleepiness；原版兔子已有组件、Brain 和睡眠 SG。玩家昏睡见 [[reference/components/grogginess]]，床具见 [[reference/components/sleepingbag]]。
- 优先使用：日程用 SetNocturnal 或睡／醒测试；催眠用 AddSleepiness；主动唤醒 WakeUp。见[接口与证据](#关键接口)。
- 重要副作用：GoToSleep 停 Brain、清战斗目标并发 gotosleep；WakeUp 重启 Brain、发 onwakeup。组件与动画状态须配合，见[生命周期](#生命周期)。
- 修改边界：服务端；实体离屏的 OnEntitySleep 只是停止测试，不等于玩法入睡。增加组件不自动补全 SG／动画。

## 关键接口

| 接口／字段 | 参数或返回 | 条件、作用与证据 |
|---|---|---|
| `SetSleepTest(fn)`／`SetWakeTest(fn, time?)` | `(inst) -> truthy`；首次延迟秒数 | 单回调替换并重排测试。默认周期 4 秒，实际每次重建取 ±0.5 秒扰动；time 只在已睡时用于首次唤醒检查。[sleeper.lua:13](../../../raw/current/scripts/components/sleeper.lua#L13)、[:179](../../../raw/current/scripts/components/sleeper.lua#L179)、[:201](../../../raw/current/scripts/components/sleeper.lua#L201)、[:310](../../../raw/current/scripts/components/sleeper.lua#L310) |
| `SetNocturnal(bool)` | true 白天睡，false 恢复默认 | 替换两个测试函数，不自行重排已有任务；默认夜间规则与洞穴光照、cavedweller 的昼夜规则不同。[sleeper.lua:94](../../../raw/current/scripts/components/sleeper.lua#L94)、[:113](../../../raw/current/scripts/components/sleeper.lua#L113)、[:127](../../../raw/current/scripts/components/sleeper.lua#L127) |
| `SetResistance(value)`／`AddSleepiness(amount, sleeptime?)` | 门槛／累积量、首次检查延迟 | 大于门槛或已睡时立即请求 GoToSleep；恰好相等则延迟 resistance 秒请求，未保存这项任务句柄；较低时启动每 wearofftime 秒减 1 的任务。[sleeper.lua:162](../../../raw/current/scripts/components/sleeper.lua#L162)、[:197](../../../raw/current/scripts/components/sleeper.lua#L197)、[:248](../../../raw/current/scripts/components/sleeper.lua#L248) |
| `GoToSleep(sleeptime?)` | 秒数或 nil；无成功返回 | 可见、未死才生效；停止 Brain、清目标、按 nosleep 标签决定是否停止 locomotor。只有首次入睡发 gotosleep，重复调用仍重排唤醒测试。[sleeper.lua:279](../../../raw/current/scripts/components/sleeper.lua#L279 |
| `WakeUp()` | 无参数 | 清 hibernate；已睡且未死才清状态／sleepiness、重启 Brain、发 onwakeup 并重建睡眠测试。未睡时不替你撤销恰好门槛的延迟入睡请求。[sleeper.lua:323](../../../raw/current/scripts/components/sleeper.lua#L323 |
| `IsAsleep()`／`IsInDeepSleep()` | boolean | 前者读组件状态；后者是已睡且 sleepiness > 0。默认唤醒测试本身没有统一排除深睡，自定义生物测试须自行消费此状态。[sleeper.lua:103](../../../raw/current/scripts/components/sleeper.lua#L103)、[:209](../../../raw/current/scripts/components/sleeper.lua#L209) |
| `diminishingreturns`／`SetExtraResist(value)` | 可选开关／额外数值 | 首次入睡可累加，影响首次唤醒检查延迟倍率，最低 0.2；不改变 AddSleepiness 比较的基础 resistance。extraresist 每 30 秒减 0.1。[sleeper.lua:263](../../../raw/current/scripts/components/sleeper.lua#L263)、[:275](../../../raw/current/scripts/components/sleeper.lua#L275)、[:299](../../../raw/current/scripts/components/sleeper.lua#L299) |

默认睡眠检查排除近家、战斗目标、燃烧、冻结、组队攻击和 busy；默认醒来检查关注战斗、火、冻结等及昼夜。换成自定义测试就替换这些判定，不能只写时间而默认保留全部安全门禁。[sleeper.lua:74](../../../raw/current/scripts/components/sleeper.lua#L74)、[:86](../../../raw/current/scripts/components/sleeper.lua#L86)

## 生命周期

生物发给自身的 gotosleep 由 CommonHandlers.OnSleep 切到 sleep／sleeping；CommonStates.AddSleepStates 处理动画与 onwakeup，wake 入口再校正组件 WakeUp。兔子与猪人实际接入了这些处理，其他 SG 需独立核验。[commonstates.lua:28-33](../../../raw/current/scripts/stategraphs/commonstates.lua#L28-L33)、[:1191](../../../raw/current/scripts/stategraphs/commonstates.lua#L1191)、[:1204](../../../raw/current/scripts/stategraphs/commonstates.lua#L1204)、[SGrabbit.lua:11](../../../raw/current/scripts/stategraphs/SGrabbit.lua#L11)、[:312](../../../raw/current/scripts/stategraphs/SGrabbit.lua#L312)

受 attacked／onignite／firedamage 会调用 WakeUp；移组件只取消测试／衰减任务和监听、清标签，**不主动重启 Brain 或退出睡眠 SG**，主动移除前先走适用的唤醒链。离屏／唤回钩子分别 StopTesting／StartTesting，由宿主显式派发。[sleeper.lua:1](../../../raw/current/scripts/components/sleeper.lua#L1)、[:34](../../../raw/current/scripts/components/sleeper.lua#L34)、[:40](../../../raw/current/scripts/components/sleeper.lua#L40)、[:189](../../../raw/current/scripts/components/sleeper.lua#L189)、[mainfunctions.lua:696](../../../raw/current/scripts/mainfunctions.lua#L696)、[:725](../../../raw/current/scripts/mainfunctions.lua#L725)

只保存正 extraresist，不保存睡眠状态、sleepiness、测试函数或日程配置；初始化时重施配置，不能据此承诺存档后继续原剩余睡眠。[sleeper.lua:337](../../../raw/current/scripts/components/sleeper.lua#L337

## 典型接入

### 让原版兔子白天睡、夜间活动

目标模组 `modmain.lua`；复用 rabbit 原版资源／Brain／SGrabbit，只改默认睡眠日程，不改移动或觅食。兔子工厂在服务端已有 sleeper 和 watchlight；本例用其默认／夜行测试，不覆盖自定义角色的专用测试。[rabbit.lua:374](../../../raw/current/scripts/prefabs/rabbit.lua#L374)、[:423](../../../raw/current/scripts/prefabs/rabbit.lua#L423)、[:452](../../../raw/current/scripts/prefabs/rabbit.lua#L452)、[modutil.lua:593](../../../raw/current/scripts/modutil.lua#L593

```lua
AddPrefabPostInit("rabbit", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local sleeper = inst.components.sleeper
    if sleeper ~= nil then
        sleeper:SetNocturnal(true)
        sleeper:StartTesting()
    end
end)
```

配置影响初始化后的兔子，由周期测试决定转换，不是马上强睡；战斗、火、冻结等仍由所选测试判断。停用后新实例恢复原版；运行中撤销可用 SetNocturnal(false) 后 StartTesting，不能保证恢复其他模组之前的自定义测试。猪人不宜直接套用：普通／守卫／变身会重设自己的测试。[sleeper.lua:113](../../../raw/current/scripts/components/sleeper.lua#L113)、[:201](../../../raw/current/scripts/components/sleeper.lua#L201)、[pigman.lua:257](../../../raw/current/scripts/prefabs/pigman.lua#L257)、[:324](../../../raw/current/scripts/prefabs/pigman.lua#L324)、[:840](../../../raw/current/scripts/prefabs/pigman.lua#L840)

## 常见陷阱与限制

- **GoToSleep(10) 恰好十秒后醒**：10 是首次测试延迟，还可乘收益递减倍率；是否醒仍由 waketestfn／hibernate 决定，后续按扰动周期继续测试。调度器的 initialdelay 不替换 period。[sleeper.lua:148](../../../raw/current/scripts/components/sleeper.lua#L148)、[:306](../../../raw/current/scripts/components/sleeper.lua#L306)、[:310](../../../raw/current/scripts/components/sleeper.lua#L310)、[scheduler.lua:295](../../../raw/current/scripts/scheduler.lua#L295
- **StopTesting 就免疫催眠**：只取消当前睡／醒测试，AddSleepiness 和恰好门槛的延迟任务仍可请求入睡；不能当完整免疫开关。[sleeper.lua:62](../../../raw/current/scripts/components/sleeper.lua#L62)、[:248](../../../raw/current/scripts/components/sleeper.lua#L248)
- **OnEntitySleep 是睡觉回调**：它是实体离屏休眠钩子；玩法睡觉查 IsAsleep 与 gotosleep，不把两条生命周期混用。见[生命周期](#生命周期)。
- **直接移组件就完成醒来**：移除清理不等同 WakeUp，Brain／SG 可能仍处旧状态；自定义宿主还需处理正在等待的延迟请求。[sleeper.lua:40](../../../raw/current/scripts/components/sleeper.lua#L40)、[:242](../../../raw/current/scripts/components/sleeper.lua#L242)

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 处理范围：锚点重定位 6 处（commonstates／mainfunctions／rabbit 行号迁移，区间文本一致）；结论修订：无，仅锚点迁移。
- 依据：全局枢纽 hunk 地图（R-09）＋变更清单命中 mainfunctions.lua、stategraphs/commonstates.lua、prefabs/rabbit.lua、prefabs/pigman.lua。

- 源码审阅：2026-10-17，Steam buildid 25643519；覆盖睡／醒测试、门槛等值分支、调度延迟、Brain／目标与 SG 事件、离屏／移除／保存及兔子接入。未审校全部 Boss、洞穴／季节特例、所有宿主自定义测试或催眠攻击来源。
- 结构／语法：页面格式、来源、大小、链接与唯一登记通过，源码锚点 strict 检查无问题；本页完整 modmain 示例经 Full Moon 3.0.0／Lua 5.1 syntax-only 通过，未检查字节往返或运行效果。
- 游戏运行：未运行；昼夜切换、离屏、延迟催眠、受击唤醒、读档及其他模组测试覆盖待测。

- 2026-10-28 build 26230496：commonstates onsleep 守卫改写（SB-1）——electrocute/knockback 期间不可入睡（除非 caninterrupt）；路由行为本身未变。
相关：[[reference/components/grogginess]]；[[reference/components/locomotor]]；[[reference/components/combat]]；[[reference/components/follower]]。
