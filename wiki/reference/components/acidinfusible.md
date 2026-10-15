---
title: "acidinfusible"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "酸雨灌注组件：酸雨且无雨 immunity 时自动灌注生物，按来源键统一加成伤害/承伤/移速并挂烟雾特效。"
aliases: ["酸液灌注", "acidinfusible", "酸雨强化", "灌注状态"]
tags: [component]
dst_build: "steam-buildid:26230496"
sources: ["components/acidinfusible.lua", "prefabs/bat_boss.lua", "prefabs/spider.lua", "prefabs/monkey.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# acidinfusible

## 快速判断

- 职责与适用：给生物挂"被酸雨灌注"的状态机：监听 `isacidraining` 世界状态与实体 `gainrainimmunity`/`loserainimmunity` 事件，自动进入/退出灌注（[驱动](../../../raw/current/scripts/components/acidinfusible.lua#L43-L48)、[判定](../../../raw/current/scripts/components/acidinfusible.lua#L181-L190)）。
- 优先使用：用 `SetMultipliers(TUNING.ACID_INFUSION_MULT.*)` 一次配齐三系加成；回调经 `SetOnInfuseFn`/`SetOnUninfuseFn`（[配置](../../../raw/current/scripts/components/acidinfusible.lua#L102-L115)）。
- 重要副作用：灌注时向 combat 的 external 乘区、locomotor 的外部速度乘区写键 `acidinfusion`，退出时按同键清除（[键](../../../raw/current/scripts/components/acidinfusible.lua#L1)、[OnUninfuse](../../../raw/current/scripts/components/acidinfusible.lua#L164-L179)）。
- 修改边界：`userainimmunity=true`（默认）且实体带 [[reference/components/rainimmunity]] 组件时不会灌注；`OnRemoveFromEntity` 反注册全部监听（[清理](../../../raw/current/scripts/components/acidinfusible.lua#L52-L60)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `IsInfused()` | → `bool` | 当前灌注状态（[实现](../../../raw/current/scripts/components/acidinfusible.lua#L134-L136)）。 |
| `SetDamageMultiplier(n)` / `SetDamageTakenMultiplier(n)` / `SetSpeedMultiplier(n)` | n: number or nil | 仅灌注中即时生效；nil 表示清除该系加成（[实现](../../../raw/current/scripts/components/acidinfusible.lua#L63-L95)）。 |
| `SetMultipliers(tuning)` | tuning: `{DAMAGE, DAMAGE_TAKEN, SPEED}` | 一次设置三系；传 nil 用 CLEARED_VALUES 全清（[实现](../../../raw/current/scripts/components/acidinfusible.lua#L97-L107)）。 |
| `SetFXLevel(level)` | level: number | `acidsmoke_fx` 子实体等级 1~4（[实现](../../../raw/current/scripts/components/acidinfusible.lua#L124-L130)、[生成](../../../raw/current/scripts/components/acidinfusible.lua#L194-L202)）。 |
| `SetUseRainImmunity(bool)` | bool | 切换是否尊重雨免疫，切换即重算状态（[实现](../../../raw/current/scripts/components/acidinfusible.lua#L117-L122)）。 |
| `SetOnInfuseFn(fn)` / `SetOnUninfuseFn(fn)` | fn(inst) | 状态切换回调（[实现](../../../raw/current/scripts/components/acidinfusible.lua#L109-L115)）。 |

## 典型接入

### 让自定义生物受酸雨强化

- 前提与选择：mastersim 挂载即可，世界状态监听组件自建；需要 combat/locomotor 才有对应加成路径（[分支](../../../raw/current/scripts/components/acidinfusible.lua#L141-L153)）。
- 生效与撤销：酸雨开始即灌注、结束自动退出并清加成/特效；实体休眠时杀特效、唤醒时按状态重生（[睡眠](../../../raw/current/scripts/components/acidinfusible.lua#L220-L228)）。
- 原版依据：蝙蝠 boss 用 3 级特效与专属乘数、回调切音效光环（[bat_boss](../../../raw/current/scripts/prefabs/bat_boss.lua#L556-L561)）；蜘蛛喷吐者 STRONGER、猴子 WEAKER（[spider](../../../raw/current/scripts/prefabs/spider.lua#L715-L717)、[monkey](../../../raw/current/scripts/prefabs/monkey.lua#L485-L487)）；脑层普遍以 `IsInfused()` 分叉行为（[batbrain](../../../raw/current/scripts/brains/batbrain.lua#L45)）。

```lua
inst:AddComponent("acidinfusible")
inst.components.acidinfusible:SetFXLevel(2)
inst.components.acidinfusible:SetMultipliers(TUNING.ACID_INFUSION_MULT.WEAKER)
inst.components.acidinfusible:SetOnInfuseFn(function(inst) inst.AnimState:SetMultColour(0.6, 1, 0.6, 1) end)
```

- 调整与验证：乘数单位是倍率（1=不变）；灌注状态本身不存档，重载后靠 `DoTaskInTime(0)` 的初始化按当前天气重算（[初始化](../../../raw/current/scripts/components/acidinfusible.lua#L3-L9)）。

## 常见陷阱与限制

- 加成键固定为 `acidinfusion`：同实体其他系统再用 `RemoveExternalSpeedMultiplier` 等同键会互相顶掉（[键](../../../raw/current/scripts/components/acidinfusible.lua#L1)）。
- 未配置 `damagemult` 等字段时灌注只出特效与回调，不加成；`TUNING.ACID_INFUSION_MULT` 的键名是 DAMAGE/DAMAGE_TAKEN/SPEED，大小写敏感（[取值](../../../raw/current/scripts/components/acidinfusible.lua#L102-L107)）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，覆盖组件全文与 bat_boss、spider、monkey、bishop、cave_vent_mite 挂载及 batbrain/bat_bossbrain/spiderbrain 读取点；未运行游戏。
- 结构检查：validate_page.py、validate_source_links.py 与 Lua 语法探针通过。
