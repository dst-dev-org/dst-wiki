---
title: "sleepingbag"
created: "2026-10-17"
updated: "2026-10-28"
type: component
summary: "床具侧占用、睡眠参数与唤醒回调；睡袋和帐篷共用组件，实际恢复由使用者与状态图完成。"
aliases: ["睡袋","帐篷回血","床具睡眠","午睡凉棚","sleepingbag"]
tags: [component, lifecycle, callback, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/sleepingbag.lua","components/sleepingbaguser.lua","prefabs/bedroll.lua","prefabs/tent.lua","actions.lua","componentactions.lua","stategraphs/SGwilson.lua","prefabs/player_common.lua","components/moisture.lua","components/temperature.lua","tuning.lua","modutil.lua","entityscript.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# sleepingbag

## 快速判断

- 职责与适用：挂在床具，保存唯一 sleeper、恢复参数及睡／醒回调；睡袋／帐篷都用它。人物端计算见 [[reference/components/sleepingbaguser]]，催眠不是本组件职责。
- 优先使用：改床具的每 tick 参数；正常使用走 SLEEPIN 动作和既有 SG，DoSleep 不替代整条动作校验。见[接口](#关键接口)与[占用流程](#占用与唤醒)。
- 重要副作用：睡／醒会互写床具 sleeper 与人物 sleepingbag，更新 hassleeper 及人物睡眠治疗标志；醒来还可扣床具耐久。见[占用与唤醒](#占用与唤醒)。
- 修改边界：服务端；动画、时段／危险／饥饿校验由实际动作／SG／Prefab 分担，只加组件不能构成新床具。示例复用原版帐篷。

## 关键接口

| 接口／字段 | 参数或约定 | 条件、作用与证据 |
|---|---|---|
| `DoSleep(doer)`／`InUse()` | 使用者实体／boolean | 仅检查自己无 sleeper、doer.sleepingbag 为 nil；直接访问 doer.components.sleepingbaguser。无成功返回，不判距离／危险／时段／生命。[sleepingbag.lua:61](../../../raw/current/scripts/components/sleepingbag.lua#L61)、[:88](../../../raw/current/scripts/components/sleepingbag.lua#L88) |
| `DoWakeUp(nostatechange?)` | boolean? | 仅双向引用匹配时清关联、取消使用者任务，再调用 onwake；true 跳过使用者的 wakeup 状态切换，不跳过回调／耐久。[sleepingbag.lua:74](../../../raw/current/scripts/components/sleepingbag.lua#L74)、[sleepingbaguser.lua:53](../../../raw/current/scripts/components/sleepingbaguser.lua#L53 |
| `tick_period`、`hunger_tick`、`health_tick`、`sanity_tick` | 周期秒数／每 tick 增量 | 默认 1 秒、-1 饥饿、+1 生命、+1 精神；乘人物各自倍率后使用。改周期会改变同增量的每秒速率，不是四个每秒值。[sleepingbag.lua:29](../../../raw/current/scripts/components/sleepingbag.lua#L29)、[tuning.lua:3073](../../../raw/current/scripts/tuning.lua#L3073)、[sleepingbaguser.lua:79](../../../raw/current/scripts/components/sleepingbaguser.lua#L79 |
| `healthsleep` | 默认 true | 允许非挨饿 tick 治疗；属性回调更新当前人物 player_classified.issleephealing。false 不会停止精神／饥饿结算。[sleepingbag.lua:1](../../../raw/current/scripts/components/sleepingbag.lua#L1)、[:21](../../../raw/current/scripts/components/sleepingbag.lua#L21)、[sleepingbaguser.lua:97](../../../raw/current/scripts/components/sleepingbaguser.lua#L97 |
| `SetSleepPhase(phase)`／`GetSleepPhase()` | 默认 "night" | 使用者在世界 phase **变化时**对比并醒来；不负责初始动作时段、SG 的 day／siestahut 判断，不据此实现全天睡眠。[sleepingbag.lua:49](../../../raw/current/scripts/components/sleepingbag.lua#L49)、[sleepingbaguser.lua:32](../../../raw/current/scripts/components/sleepingbaguser.lua#L32)、[SGwilson.lua:12516](../../../raw/current/scripts/stategraphs/SGwilson.lua#L12516 |
| `onsleep`／`onwake` | `(bed, doer)`／`(bed, doer, nostatechange)` | 单字段回调；前者在建立关系／启动 tick 后，后者在取消任务／清占用后调用。已有回调不可覆盖掉原动画／耗用逻辑。[sleepingbag.lua:61](../../../raw/current/scripts/components/sleepingbag.lua#L61)、[:74](../../../raw/current/scripts/components/sleepingbag.lua#L74) |
| `SetTemperatureTickFn(fn)` | `(bed, doer)` | 每 SleepTick 由人物端调用，组件不自动消费 sleep_temp_min／max；原版毛皮睡袋的回调才读取上下限。[sleepingbaguser.lua:101](../../../raw/current/scripts/components/sleepingbaguser.lua#L101)、[bedroll.lua:34](../../../raw/current/scripts/prefabs/bedroll.lua#L34 |
| `dryingrate`／`ambient_temp` | 非负湿度点／秒或 nil；环境温度或 nil | 分别由人物 moisture 的专用速率分支、temperature 的环境计算消费，**不在 SleepTick 里执行**。干燥率设 0 仍选专用分支；详见 [[reference/components/moisture#持续速率的组成]]。[moisture.lua:304](../../../raw/current/scripts/components/moisture.lua#L304)、[:345](../../../raw/current/scripts/components/moisture.lua#L345)、[temperature.lua:341](../../../raw/current/scripts/components/temperature.lua#L341 |

## 占用与唤醒

SLEEPIN 根据 invobject／target 选床具后调用 DoSleep；组件先写 bed.sleeper、人物 sleepingbag，再调用 sleepingbaguser:DoSleep 启动任务，最后 onsleep。hassleeper 和 issleephealing 随属性变化更新。组件不播放睡眠动画；SGwilson 的 bedroll／tent 动作状态另负责初始门禁、动作帧、隐藏／无敌与睡眠状态标记。[actions.lua:2699](../../../raw/current/scripts/actions.lua#L2699)、[sleepingbag.lua:7](../../../raw/current/scripts/components/sleepingbag.lua#L7)、[:61](../../../raw/current/scripts/components/sleepingbag.lua#L61)、[SGwilson.lua:12403](../../../raw/current/scripts/stategraphs/SGwilson.lua#L12403)、[:12509](../../../raw/current/scripts/stategraphs/SGwilson.lua#L12509)

正常清理从床具 DoWakeUp 进入；人物单独取消 tick 不会完成双向解除。SG 状态中断及人物离开会调 DoWakeUp(true)，床具移组件／删实体也复用唤醒；主动移组件仍应先完成唤醒并检查回调前提：宿主先将 components.sleepingbag 清空才执行移除钩子，原回调若再次读取该组件便不安全；不要手写占用字段跳过退出。[sleepingbag.lua:74](../../../raw/current/scripts/components/sleepingbag.lua#L74)、[:92](../../../raw/current/scripts/components/sleepingbag.lua#L92)、[SGwilson.lua:12492](../../../raw/current/scripts/stategraphs/SGwilson.lua#L12492)、[player_common.lua:1417](../../../raw/current/scripts/prefabs/player_common.lua#L1417)、[tent.lua:93](../../../raw/current/scripts/prefabs/tent.lua#L93)、[entityscript.lua:670](../../../raw/current/scripts/entityscript.lua#L670

## 典型接入

### 让原版帐篷每次睡眠 tick 多恢复两点生命

目标模组 `modmain.lua`；只改 tent，保留原动画／资源、建造与睡／醒回调。原帐篷每 tick 生命值为默认的两倍，示例在工厂完成后加 2，不重设其他三维或周期。[tent.lua:182](../../../raw/current/scripts/prefabs/tent.lua#L182)、[:205](../../../raw/current/scripts/prefabs/tent.lua#L205)、[:240](../../../raw/current/scripts/prefabs/tent.lua#L240)、[modutil.lua:593](../../../raw/current/scripts/modutil.lua#L593

```lua
AddPrefabPostInit("tent", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local bag = inst.components.sleepingbag
    if bag ~= nil then
        bag.health_tick = bag.health_tick + 2
    end
end)
```

原版周期 1 秒、人物 health_bonus_mult 为 1 时，每次由 2 改为 4；实际治疗仍受人物倍率、挨饿、生命组件和上限等门禁。影响所有经此入口初始化的 tent，不影响 siestahut／睡袋；停用后新实例恢复原值，组件不保存这些配置。本例不新建床具或承诺任意角色能睡。[sleepingbaguser.lua:83](../../../raw/current/scripts/components/sleepingbaguser.lua#L83)、[:97](../../../raw/current/scripts/components/sleepingbaguser.lua#L97)

原版不同模式只作选型：
- 毛皮睡袋在 SLEEPIN 动作消费 finiteuses，醒来只在无次数时移除；草席醒来消耗一件。不是所有床具都在 DoWakeUp 内主动 Use。[bedroll.lua:11](../../../raw/current/scripts/prefabs/bedroll.lua#L11)、[:114](../../../raw/current/scripts/prefabs/bedroll.lua#L114)
- 帐篷／凉棚在 onwake 调 Use；凉棚另带 siestahut 标签、日间 SG 条件、SetSleepPhase("day") 和降温回调，不能只改一个 phase 字段复刻。[tent.lua:93](../../../raw/current/scripts/prefabs/tent.lua#L93)、[:125](../../../raw/current/scripts/prefabs/tent.lua#L125)、[:222](../../../raw/current/scripts/prefabs/tent.lua#L222)

## 常见陷阱与限制

- **直接 DoSleep 就经过全部校验**：组件只判双向空闲，动作选择还查玩家／insomniac／占用等，SG 再查危险／饥饿／时段。自建入口须明确复用哪条路径，不能把 ACTIONS.SLEEPIN 返回 true 当入睡确认。[componentactions.lua:794](../../../raw/current/scripts/componentactions.lua#L794)、[actions.lua:2699](../../../raw/current/scripts/actions.lua#L2699)、[SGwilson.lua:12410](../../../raw/current/scripts/stategraphs/SGwilson.lua#L12410
- **DoWakeUp(true) 不消耗耐久**：true 只控制使用者是否转 SG，仍执行 onwake；帐篷依然扣次。见[接口](#关键接口)。
- **写 sleep_temp_min 自动恒温**：上下限是具体 Prefab 回调的输入，未安装回调不会自动处理；ambient_temp 则由另一条自然换热路径消费。[bedroll.lua:34](../../../raw/current/scripts/prefabs/bedroll.lua#L34)、[:119](../../../raw/current/scripts/prefabs/bedroll.lua#L119)
- **改 tick_period 立刻重排当前睡眠**：任务在 DoSleep 时创建，之后直接改字段不重建旧周期；每次增量参数则在 SleepTick 当下读取。[sleepingbaguser.lua:40](../../../raw/current/scripts/components/sleepingbaguser.lua#L40)、[:83](../../../raw/current/scripts/components/sleepingbaguser.lua#L83)

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 处理范围：锚点重定位 8 处（actions／SGwilson／tuning 行号迁移，区间文本逐段比对一致）；结论修订：无，仅锚点迁移。
- 依据：枢纽 diff 审阅（actions.lua／componentactions.lua hunk 地图）＋变更清单命中 actions.lua、componentactions.lua、stategraphs/SGwilson.lua、prefabs/player_common.lua、entityscript.lua、tuning.lua。

- 源码审阅：2026-10-17，Steam buildid 25643519；覆盖床具占用／唤醒、回调时序、每 tick 单位、人物消费者、原床具动作／SG、干燥／温度分工及帐篷增益接入。未审校蜘蛛巢、便携帐篷、所有床具变体或自建资源／网络工程。
- 结构／语法：页面格式、来源、大小、链接与唯一登记通过，源码锚点 strict 检查无问题；本页完整 modmain 示例经 Full Moon 3.0.0／Lua 5.1 syntax-only 通过，未检查字节往返或运行效果。
- 游戏运行：未运行；占用竞争、起身／销毁扣次、退出重连、三维恢复、昼夜和多人兼容待测。

相关：[[reference/components/sleepingbaguser]]；[[reference/components/finiteuses]]；[[reference/components/health]]；[[reference/components/temperature]]。
