---
title: "stunnable"
created: "2026-10-21"
updated: "2026-10-28"
type: component
summary: "在滚动时间窗累计生命变化，超过阈值后发出眩晕事件，由龙蝇状态图完成倒地与起身。"
aliases: ["stunnable", "伤害窗口眩晕", "龙蝇倒地", "连续伤害阈值", "stunned", "stun_finished"]
tags: [component, event, state-machine, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/stunnable.lua", "components/health.lua", "prefabs/dragonfly.lua", "stategraphs/SGdragonfly.lua", "entityscript.lua", "tuning.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# stunnable

## 快速判断

- 职责与适用：挂在受击宿主上，监听 `healthdelta`，累计最近时间窗的生命变化绝对值；严格超过阈值时触发眩晕。原版安装与配置参照是主模拟端的龙蝇。[stunnable.lua:1](../../../raw/current/scripts/components/stunnable.lua#L1 [dragonfly.lua:490](../../../raw/current/scripts/prefabs/dragonfly.lua#L490
- 优先使用：调整**已有龙蝇宿主**的参数，保留 `SGdragonfly` 处理 `stunned`／`stun_finished` 的倒地链；组件不直接停止移动、停止脑或播放动画。[SGdragonfly.lua:35](../../../raw/current/scripts/stategraphs/SGdragonfly.lua#L35
- 重要副作用：每次 `Stun()` 清累计、从触发时刻起计冷却、提高下次阈值、立即发 `stunned`，延时发 `stun_finished`。没有 `stunned` 布尔状态或公共解除方法。[stunnable.lua:15](../../../raw/current/scripts/components/stunnable.lua#L15
- 修改边界：组件没有通用免疫测试、SG、动作入口或保存／移除清理接口。需要按来源禁止移动用 [[reference/components/rooted]]，玩家黏液束缚用 [[reference/components/pinnable]]；不要向任意对象加组件就期望倒地。

## 关键接口

| 接口／字段 | 参数、返回或单位 | 条件、作用与证据 |
|---|---|---|
| `TakeDamage(damage)` | 数值；无返回值 | 冷却中立即返回；否则按当前 `GetTime()` 为键记录 `math.abs(damage)`，窗口总和 **`>`** 当前阈值才 `Stun()`。不检查正负／攻击者／死亡。[stunnable.lua:23](../../../raw/current/scripts/components/stunnable.lua#L23 |
| `GetDamageInPeriod()` | 返回数值总和 | 仅保留 `记录时刻 + stun_period > 当前时刻` 的项，并删除其余项；清理是调用驱动，没有周期任务。[stunnable.lua:33](../../../raw/current/scripts/components/stunnable.lua#L33 |
| `Stun()` | 无参数／返回值 | 直接触发，不自行检查冷却或资格；增加阈值后发事件。重复直接调用会创建多个未保存句柄的结束任务。[stunnable.lua:15](../../../raw/current/scripts/components/stunnable.lua#L15 |
| `stun_threshold`／`stun_resist` | 数值；组件默认 1000／150 | 当前触发阈值／每次触发的阈值增量；龙蝇初始化为 1250／250。[stunnable.lua:5](../../../raw/current/scripts/components/stunnable.lua#L5 [dragonfly.lua:580](../../../raw/current/scripts/prefabs/dragonfly.lua#L580 [tuning.lua:2620](../../../raw/current/scripts/tuning.lua#L2620 |
| `stun_period` | 游戏秒；默认 5 | 滚动累计窗口，不是眩晕时长。[stunnable.lua:6](../../../raw/current/scripts/components/stunnable.lua#L6 |
| `stun_duration`／`stun_cooldown` | 游戏秒；默认 10／60 | 前者是结束事件延时，后者从触发时刻起禁止 `TakeDamage` 入账；两者互相独立。[stunnable.lua:7](../../../raw/current/scripts/components/stunnable.lua#L7 [stunnable.lua:15](../../../raw/current/scripts/components/stunnable.lua#L15 |
| `damage`／`valid_stun_time` | 时间键表／允许入账的最早时刻 | 同一时间键写入会覆盖旧值，不是每次相加；初始冷却截止为 0。[stunnable.lua:4](../../../raw/current/scripts/components/stunnable.lua#L4 [stunnable.lua:26](../../../raw/current/scripts/components/stunnable.lua#L26 |

`healthdelta.amount` 来自 `Health:DoDelta` 的 amount；它在重定向、无敌、吸收／修改与单次伤害限制处理后发送，并非必然等于血量实际减少值。组件取绝对值，因此**治疗的正 amount 也会入账**；直接 `TakeDamage` 与自动事件混用还会重复记录或覆盖同时间键。[health.lua:613](../../../raw/current/scripts/components/health.lua#L613 [health.lua:640](../../../raw/current/scripts/components/health.lua#L640 [stunnable.lua:12](../../../raw/current/scripts/components/stunnable.lua#L12

龙蝇 `stunned` 处理器只检查未死亡就进 `knockdown`；该状态停止 locomotor、播放倒地动画、启动鳞片掉落相关伤害跟踪，再转 `knockdown_idle`。`stun_finished` 只在当前有 `grounded` 且未死亡时处理：通常进起身，睡眠与触电有各自转接。组件的持续秒数不等于保证锁住同一个 SG 状态到那一刻。[SGdragonfly.lua:35](../../../raw/current/scripts/stategraphs/SGdragonfly.lua#L35 [SGdragonfly.lua:280](../../../raw/current/scripts/stategraphs/SGdragonfly.lua#L280 [SGdragonfly.lua:388](../../../raw/current/scripts/stategraphs/SGdragonfly.lua#L388 [SGdragonfly.lua:924](../../../raw/current/scripts/stategraphs/SGdragonfly.lua#L924

## 典型接入

### 调整原龙蝇倒地的持续时间与再次触发间隔

以下为已启用模组的完整 `modmain.lua`，直接加载；无需新增 `PrefabFiles`、动画或其他资源，沿用龙蝇原工厂、网络、SG 和动画。`AddPrefabPostInit` 是模组环境函数，游戏全局用 `GLOBAL`。[modutil.lua:593](../../../raw/current/scripts/modutil.lua#L593 [dragonfly.lua:448](../../../raw/current/scripts/prefabs/dragonfly.lua#L448 [dragonfly.lua:517](../../../raw/current/scripts/prefabs/dragonfly.lua#L517

```lua
local STUN_SECONDS = 6
local COOLDOWN_SECONDS = 45

AddPrefabPostInit("dragonfly", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    local stunnable = inst.components.stunnable
    if stunnable ~= nil then
        stunnable.stun_duration = STUN_SECONDS
        stunnable.stun_cooldown = COOLDOWN_SECONDS
    end
end)
```

替换点为两个非负游戏秒值；`45` 从触发开始计，不是从起身再加 45 秒。代码保留原初始阈值、窗口、抗性增量及伤害入口；以后触发使用新配置，已排定的结束任务不会随字段修改重排。实例重新构造时再次应用；卸载后新实例恢复原版参数。[stunnable.lua:15](../../../raw/current/scripts/components/stunnable.lua#L15

本例刻意保留阈值／窗口：龙蝇脱战软重置会把这两项写回 `TUNING.DRAGONFLY_STUN`／`DRAGONFLY_STUN_PERIOD`；若需求是自定义长期阈值，必须同时设计与这条原重置链的协作，不能只改初始化一次。[dragonfly.lua:245](../../../raw/current/scripts/prefabs/dragonfly.lua#L245

## 常见陷阱与限制

- **阈值相等就应倒地**：判定是严格大于；窗口边界相等也已过期。同一游戏时间戳的多次变化被后写覆盖，不能把该表当精确的逐击累加器。[stunnable.lua:23](../../../raw/current/scripts/components/stunnable.lua#L23 [stunnable.lua:39](../../../raw/current/scripts/components/stunnable.lua#L39
- **默认只统计敌人的真实扣血**：没有攻击者、原因、正负或 overtime 筛选。正向治疗、非战斗健康变化也可能累计；生命无敌先阻止 `healthdelta` 的路径与直接调用 `Stun()` 是不同入口。[stunnable.lua:12](../../../raw/current/scripts/components/stunnable.lua#L12 [health.lua:623](../../../raw/current/scripts/components/health.lua#L623
- **直接 `Stun()` 遵守冷却／免疫**：冷却只在 `TakeDamage` 检查。SG 的死亡／grounded 等资格是消费者规则，不是组件通用免疫；频繁强制触发可能使旧结束事件提前结束后一次倒地。[stunnable.lua:15](../../../raw/current/scripts/components/stunnable.lua#L15 [SGdragonfly.lua:41](../../../raw/current/scripts/stategraphs/SGdragonfly.lua#L41
- **移除组件即取消监听／结束任务**：构造使用捕获 self 的匿名 `healthdelta` 监听，没有 `OnRemoveFromEntity`，也未保留任务句柄；单独 `RemoveComponent` 不会替它注销监听或取消延时事件。实体整体删除会清所有事件与任务。不要把动态移除当安全解除接口。[stunnable.lua:12](../../../raw/current/scripts/components/stunnable.lua#L12 [entityscript.lua:670](../../../raw/current/scripts/entityscript.lua#L670 [entityscript.lua:1730](../../../raw/current/scripts/entityscript.lua#L1730
- **期待累计、抗性、冷却和倒地进度自动存档**：组件没有 `OnSave`／`OnLoad`；龙蝇工厂保存交战标志，并在载入交战存档后重置战斗，不保存这些字段。自定义保存方案需一起处理剩余结束任务和宿主 SG，不能只存一个布尔值。[stunnable.lua:1](../../../raw/current/scripts/components/stunnable.lua#L1 [dragonfly.lua:386](../../../raw/current/scripts/prefabs/dragonfly.lua#L386

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 处理范围：锚点重定位 2 处（entityscript／tuning 行号迁移）；结论修订：无，仅锚点迁移。
- 依据：全局枢纽 hunk 地图（R-08）＋变更清单命中 entityscript.lua、tuning.lua。

- 源码审阅：2026-10-21，Steam buildid 25643519；完整阅读 `components/stunnable.lua`，核验 healthdelta 生产、龙蝇服务端安装／配置／软重置／保存、SG 倒地与恢复（含睡眠／触电分支）、组件移除与实体任务清理。原始脚本只读。
- 示例范围：仅配置实际原宿主，沿用其事件消费者；完整 `modmain.lua` 经 Lua 5.1 语法检查。页面元数据／大小／本地链接与源码行锚点检查通过。
- 游戏运行：未运行。阈值临界、多次同时间戳变化、治疗入账、冷却交界、重复强制触发、死亡／睡眠／触电、读档与联机表现仍需实测；未设计其他宿主的通用眩晕协议。
