---
title: "lunarhailbuildup"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "月雹堆积组件：月雹天气下物体逐渐堆积，堆满后进入可敲除状态并掉落月亮玻璃，随 MakeSnowCovered 自动接入。"
aliases: ["月雹堆积", "lunarhailbuildup", "月亮玻璃清理"]
tags: [component]
dst_build: "steam-buildid:26230496"
sources: ["components/lunarhailbuildup.lua", "standardcomponents.lua", "actions.lua", "prefabs/rock_ice.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# lunarhailbuildup

## 快速判断

- 职责与适用：可工作实体（建筑、冰石等）的月雹天气响应：监听 `islunarhailing` 世界状态，堆积期周期累加 `buildupcurrent`，满值转可工作（敲/铲）状态（[驱动](../../../raw/current/scripts/components/lunarhailbuildup.lua#L11-L14)、[状态机](../../../raw/current/scripts/components/lunarhailbuildup.lua#L195-L219)）。
- 优先使用：一般经 `MakeSnowCovered`→`MakeLunarHailBuildup` 间接挂载（[接入](../../../raw/current/scripts/standardcomponents.lua#L842-L855)、[包装](../../../raw/current/scripts/standardcomponents.lua#L888-L893)），再按体型 `SetLunarHailBuildupAmountSmall/Medium/Large` 配工作量。
- 重要副作用：进入可工作态打 `LunarBuildup` 标签并监听 `worked` 事件（被砸毁也结算奖励，[桥接](../../../raw/current/scripts/components/lunarhailbuildup.lua#L165-L173)）；推 `lunarhailbuildupdelta`/`lunarhailbuildupworkablestatechanged`/`lunarhailbuildupworked` 事件（[事件](../../../raw/current/scripts/components/lunarhailbuildup.lua#L139-L142)）。
- 修改边界：`rainimmunity` 组件或 `SetIgnoreLunarHailTicks(true)` 可免疫堆积 tick（[免疫](../../../raw/current/scripts/components/lunarhailbuildup.lua#L75-L78)、[开关](../../../raw/current/scripts/components/lunarhailbuildup.lua#L54-L71)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `IsBuildupWorkable()` | → `bool` | `workleft > 0`，即处于可清除状态（[实现](../../../raw/current/scripts/components/lunarhailbuildup.lua#L33-L35)）。 |
| `DoWorkToRemoveBuildup(workcount, doer)` | workcount: number | 扣减工作量，清零时掉落奖励并复位；给 doer 推 `workinglunarhailbuildup`（[实现](../../../raw/current/scripts/components/lunarhailbuildup.lua#L129-L143)）。 |
| `DoAllRemainingWorkToRemoveBuildup(doer)` | → `nil` | 一次清空（投石机弹丸用，[消费](../../../raw/current/scripts/prefabs/winona_catapult_projectile.lua#L186-L187)）。 |
| `SetTotalWorkAmount(n)` / `SetMoonGlassAmount(n)` | number | 配置总工作量与掉落月亮玻璃数量（[实现](../../../raw/current/scripts/components/lunarhailbuildup.lua#L37-L44)）。 |
| `SetBuildupPercent(percent)` / `GetBuildupPercent()` | 0~1 | 堆积比例读写（[实现](../../../raw/current/scripts/components/lunarhailbuildup.lua#L46-L52)）。 |
| `DropRewards(mult, doer)` | mult: number or nil | 掉 `floor(moonglassamount*mult)` 个月亮玻璃，充能概率随 buildupcurrent 插值（[实现](../../../raw/current/scripts/components/lunarhailbuildup.lua#L151-L162)）。 |
| `OnSave()` / `OnLoad(data)` | 组件存档钩子 | 保存 workleft 与 buildupcurrent（[实现](../../../raw/current/scripts/components/lunarhailbuildup.lua#L223-L244)）。 |

## 典型接入

### 让建筑参与月雹堆积与清理

- 前提与选择：实体须可被工作组件处理（原版动作 REMOVELUNARBUILDUP 校验目标组件与工具，[动作](../../../raw/current/scripts/actions.lua#L6630-L6645)）；`MakeSnowCovered` 已含挂载，勿重复调 `MakeLunarHailBuildup`。
- 生效与撤销：开始下雹启动堆积任务、停雹启动衰减任务（[切换](../../../raw/current/scripts/components/lunarhailbuildup.lua#L108-L125)）；`RemoveLunarHailBuildup` 移除组件并还原雪层符号（[移除](../../../raw/current/scripts/standardcomponents.lua#L895-L899)）。
- 原版依据：冰石按生长阶段切换 Small/Medium/Large 工作量并在干涸时移除组件（[rock_ice](../../../raw/current/scripts/prefabs/rock_ice.lua#L139-L151)）；玩家清理动作与 SGwilson 状态引用 `IsBuildupWorkable`（[SG](../../../raw/current/scripts/stategraphs/SGwilson.lua#L5216-L5217)）。

```lua
-- 服务端实体（mastersim），例如自定义雕像：
MakeSnowCovered(inst)                          -- 内部自动 MakeLunarHailBuildup
inst.components.lunarhailbuildup:SetTotalWorkAmount(TUNING.LUNARHAIL_BUILDUP_TOTAL_WORK_AMOUNT_SMALL)
inst.components.lunarhailbuildup:SetMoonGlassAmount(TUNING.LUNARHAIL_BUILDUP_MOONGLASS_AMOUNT_SMALL)
```

- 调整与验证：堆积/衰减速率与时长全在 `TUNING.LUNARHAIL_*`；掉落物向上弹开距离取实体物理半径（[弹射](../../../raw/current/scripts/components/lunarhailbuildup.lua#L153-L160)）。

## 常见陷阱与限制

- 堆满才可清除：`buildupcurrent < max` 时 `workleft` 为 0，REMOVELUNARBUILDUP 会因 `IsBuildupWorkable()` 为假而失败（[动作校验](../../../raw/current/scripts/actions.lua#L6631-L6634)）。
- 被工作组件直接砸毁（`worked` 事件且 workleft==0）走 `OnWorked_Bridge` 掉落并乘销毁系数，但不复位堆积（组件将随实体销毁）；手动清理路径才复位重积（[对照](../../../raw/current/scripts/components/lunarhailbuildup.lua#L165-L173)）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，覆盖组件全文、standardcomponents 接入函数族、actions REMOVELUNARBUILDUP、rock_ice/SGwilson/SGwx78_common/winona_catapult_projectile 消费点；未运行游戏。
- 结构检查：validate_page.py、validate_source_links.py 与 Lua 语法探针通过。
