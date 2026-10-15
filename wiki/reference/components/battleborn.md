---
title: "battleborn"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "攻击积攒型回复（Wathgrithr 战吼）：按伤害占比积攒 battleborn 点数，超过阈值一次性回血/修甲/回 san，可经回调自定义触发效果。"
aliases: ["战吼积攒", "battleborn", "女武神回血"]
tags: [component, callback]
dst_build: "steam-buildid:26230496"
sources: ["components/battleborn.lua", "prefabs/wathgrithr.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# battleborn

## 快速判断

- 职责与适用：挂在可攻击的角色（原版 [wathgrithr](../../../raw/current/scripts/prefabs/wathgrithr.lua#L188-L193)），监听 `onattackother` 积攒点数；积攒超阈值时按配置回血、修复带 `battleborn_repairable` 标签的装备、回 san 并触发回调。
- 优先使用：角色攻击附带回复需求直接加组件配置即可；数值全部由 `Set*` 方法与 TUNING 覆盖（[配置点](../../../raw/current/scripts/prefabs/wathgrithr.lua#L189-L193)）。
- 重要副作用：监听 `onattackother` 与 `death`（[L26-L27](../../../raw/current/scripts/components/battleborn.lua#L26-L27)）；死亡清零积攒（[L129-L131](../../../raw/current/scripts/components/battleborn.lua#L129-L131)）。
- 修改边界：主机端组件，无存读钩子（点数不跨存档）；不判定伤害来源合法性时需自行给 `SetValidVictimFn`。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `OnAttack(data)` | onattackother 数据 -> void | 核心流程：`delta = clamp(受害者 defaultdamage × battleborn_bonus × 伤害占比, clamp_min, clamp_max)`；存量超 `battleborn_trigger_threshold` 时消耗并触发回复。[实现](../../../raw/current/scripts/components/battleborn.lua#L73-L127) |
| `SetBattlebornBonus(bonus)` | number -> void | 伤害收益系数，原版 Wathgrithr 为 `TUNING.WATHGRITHR_BATTLEBORN_BONUS`（0.25）。[实现](../../../raw/current/scripts/components/battleborn.lua#L49-L51) |
| `SetSanityEnabled/SetHealthEnabled` | bool -> void | 控制回复通道：血不满优先回血，满血改为修甲；sanity 通道独立。[实现](../../../raw/current/scripts/components/battleborn.lua#L53-L59) |
| `SetValidVictimFn(fn)` | `fn(victim) -> bool` | 过滤有效受击者；未设置时对所有目标结算。[实现](../../../raw/current/scripts/components/battleborn.lua#L69-L71) |
| `SetTriggerThreshold/SetDecayTime/SetStoreTime` | number -> void | 阈值、衰减期与保鲜期，默认取 `TUNING.BATTLEBORN_TRIGGER_THRESHOLD`（1）/ `DECAY_TIME`（5）/ `STORE_TIME`（3）。[实现](../../../raw/current/scripts/components/battleborn.lua#L33-L43) |
| `SetClampMin/SetClampMax` | number -> void | 单次 delta 夹取范围，默认 0.33～2.0。[实现](../../../raw/current/scripts/components/battleborn.lua#L61-L67) |
| `RepairEquipment` | `(item, delta)` | 暴露给模组的装备修复入口：修 `battleborn_repairable` 标签的受损 armor，倍率 `TUNING.BATTLEBORN_REPAIR_EQUIPMENT_MULT`（3.5）。[实现](../../../raw/current/scripts/components/battleborn.lua#L1-L8) |
| `SetOnTriggerFn(fn)` | `fn(inst, battleborn) -> void` | 阈值触发时的自定义效果回调。[实现](../../../raw/current/scripts/components/battleborn.lua#L45-L47) |

## 典型接入

### 给角色加"越战越勇"回复

- 前提与选择：主机端角色 master postinit 挂组件并配置；受害者需有 health/combat 组件供取值。

```lua
inst:AddComponent("battleborn")
inst.components.battleborn:SetBattlebornBonus(TUNING.WATHGRITHR_BATTLEBORN_BONUS)
inst.components.battleborn:SetSanityEnabled(true)
inst.components.battleborn:SetHealthEnabled(true)
inst.components.battleborn:SetValidVictimFn(inst.IsValidVictim)
inst.components.battleborn.allow_zero = false
```

- 调整与验证：阈值单位是"累计 delta"，默认 1；保鲜 3 秒后存量在 5 秒内按平方插值衰减到 0（[L90-L98](../../../raw/current/scripts/components/battleborn.lua#L90-L98)）。

## 常见陷阱与限制

- **allow_zero 默认 true**：0 伤害攻击也按 `clamp(0, 0.33, 2.0)` 计 0.33 点积攒（[L81-L87](../../../raw/current/scripts/components/battleborn.lua#L81-L87)）；Wathgrithr 显式关掉（[L193](../../../raw/current/scripts/prefabs/wathgrithr.lua#L193)），自定义角色要注意同样配置。
- **修甲需要标签**：装备必须带 `battleborn_repairable` 标签且 armor 已受损才会被修复，否则满血触发时无效果。
- **取值用受害者 defaultdamage**：delta 按"受害者默认伤害×占比"计，与实际打出的伤害（护甲减免后）无关。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；组件全文、wathgrithr 配置点、tuning.lua 相关常量（tuning.lua:4981-4987）。
- 游戏运行：未运行。
