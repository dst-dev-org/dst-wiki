---
title: "wx78_abilitycooldowns"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "WX-78 技能冷却组件：以哈希为键管理 wx78_abilitycooldown 冷却实体，服务端重启／停止，双端查询并驱动 UI。"
aliases: ["WX78技能冷却", "技能冷却", "wx78_abilitycooldowns"]
tags: [component, network, callback]
dst_build: "steam-buildid:26230496"
sources: ["components/wx78_abilitycooldowns.lua", "prefabs/wx78.lua", "prefabs/wx78_abilitycooldown.lua", "stategraphs/SGwx78_common.lua", "prefabs/wx78_common.lua", "widgets/upgrademodulesdisplay.lua", "prefabs/wx78_shadow_fuel_debuff.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# wx78_abilitycooldowns

## 快速判断

- 职责与适用：挂在 WX-78（[挂接](../../../raw/current/scripts/prefabs/wx78.lua#L715)）与被附身躯体（[wx78_possessedbody.lua:656](../../../raw/current/scripts/prefabs/wx78_possessedbody.lua#L656）上，登记每个技能的冷却实体（wx78_abilitycooldown prefab），支撑 shielding/screech/shadow_energy 等冷却的存续与 UI 显示。
- 优先使用：服务端 `RestartAbilityCooldown(abilityname, duration)` 幂等重启；查询用 `IsInCooldown`／`GetAbilityCooldownPercent`（双端可用）。
- 重要副作用：注册／移除都推 `wx78_abilitycooldowns_update` 事件（data.abilityname），UI 监听它刷新（[推送](../../../raw/current/scripts/components/wx78_abilitycooldowns.lua#L43)、[监听](../../../raw/current/scripts/widgets/upgrademodulesdisplay.lua#L145)）。
- 修改边界：冷却实体 removal 自动注销并再推事件（[回调](../../../raw/current/scripts/components/wx78_abilitycooldowns.lua#L6-L16)）；重启／停止有 `ismastersim` 守卫。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `IsInCooldown(abilityname)` | `abilityname: string\|hash -> bool` | 键为 `GetHash(abilityname)`（字符串转哈希数值）（[实现](../../../raw/current/scripts/components/wx78_abilitycooldowns.lua#L19-L28)）。 |
| `GetAbilityCooldownPercent(abilityname)` | `-> number\|nil` | 返回冷却实体 `GetPercent()`，无冷却返回 nil（[实现](../../../raw/current/scripts/components/wx78_abilitycooldowns.lua#L30-L33)）。 |
| `RegisterAbilityCooldown(cd)` | `cd: entity -> 无` | 校验 `cd:GetAbilityName()` 非 0 且不重复，注册并监听其 onremove（[实现](../../../raw/current/scripts/components/wx78_abilitycooldowns.lua#L35-L46)）。 |
| `RestartAbilityCooldown(abilityname, duration)` | 服务端 | 已有冷却则原地重启，否则 Spawn `"wx78_abilitycooldown"`、SetParent、SetClassifiedTarget 后 `InitAbilityCooldown(hash, duration)`（[实现](../../../raw/current/scripts/components/wx78_abilitycooldowns.lua#L51-L66)）。 |
| `StopAbilityCooldown(abilityname)` | 服务端 | 对应冷却实体 `Remove()`，经 onremove 回调注销（[实现](../../../raw/current/scripts/components/wx78_abilitycooldowns.lua#L68-L76)）。 |

## 典型接入

### 原版使用点（参照）

- 前提与选择：技能状态机在进入冷却时重启（shielding：([SGwx78_common.lua:154-156](../../../raw/current/scripts/stategraphs/SGwx78_common.lua#L154-L156)；screech：([SGwx78_common.lua:1100-1101](../../../raw/current/scripts/stategraphs/SGwx78_common.lua#L1100-L1101)）；能力判定在 wx78_common 查询（[146](../../../raw/current/scripts/prefabs/wx78_common.lua#L146)、[158](../../../raw/current/scripts/prefabs/wx78_common.lua#L158)）。
- 生效与撤销：shadow_energy 由暗影燃料 debuff 借道本组件做网络同步（restart/stop，([wx78_shadow_fuel_debuff.lua:22-23](../../../raw/current/scripts/prefabs/wx78_shadow_fuel_debuff.lua#L22-L23)、[48-49](../../../raw/current/scripts/prefabs/wx78_shadow_fuel_debuff.lua#L48-L49)）。

```lua
-- 服务端：给 WX-78 的自定义模块加冷却
local wx = inst.components.wx78_abilitycooldowns
wx:RestartAbilityCooldown("my_module", 30)            -- 30 秒冷却
if wx:IsInCooldown("my_module") then                  -- 双端可查
    -- 冷却中
end
```

- 调整与验证：冷却实体 prefab 自带 net classified（`SetClassifiedTarget` 指向宿主），客户端读百分比走 replica；`GetLength/GetPercent` 在 ([wx78_abilitycooldown.lua:7-15](../../../raw/current/scripts/prefabs/wx78_abilitycooldown.lua#L7-L15)。

## 常见陷阱与限制

- **键是哈希数值不是字符串**：内部 `cooldowns` 以 `hash(abilityname)` 为键；直接遍历或索引要用 GetHash 转换（UI 也先转换再读，([upgrademodulesdisplay.lua:663-672](../../../raw/current/scripts/widgets/upgrademodulesdisplay.lua#L663-L672)）。
- **abilityname==0 只打印不拒绝**：注册非法名仅输出日志且不注册，调用方拿不到错误返回（[实现](../../../raw/current/scripts/components/wx78_abilitycooldowns.lua#L37-L40)）。
- **专用 UI 直读内部表**：upgrademodulesdisplay 直接 `pairs(cooldowns)` 绕过封装（[直读](../../../raw/current/scripts/widgets/upgrademodulesdisplay.lua#L663)），改动组件内部结构会破坏该 widget。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；覆盖组件全文、两个挂接点、SG 与 wx78_common 查询、UI 监听与直读、暗影燃料借道、冷却实体 prefab。
- 结构检查：本批 validate_page、validate_source_links 及示例 lua 语法探测（见批级记录）。
- 游戏运行：未运行。
