---
title: "wx78_shield"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "WX-78 电荷护盾：经 health.deltamodifierfn 拦截伤害，越过穿透阈值完全免疫，配合充能源按秒回充并同步 wx78_classified。"
aliases: ["WX78护盾", "wx78_shield", "电荷护盾", "威拓护盾"]
tags: [component, energy]
dst_build: "steam-buildid:26230496"
sources: ["components/wx78_shield.lua", "prefabs/wx78.lua", "wx78_moduledefs.lua", "prefabs/wx78_possessedbody.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# wx78_shield

## 快速判断

- 职责与适用：给 WX-78（及 wx78_possessedbody）提供可充能伤害护盾；原版把 `health.deltamodifierfn` 指向 `RedirectToWxShield`，由组件 `OnTakeDamage` 决定实际入血量（[挂载](../../../raw/current/scripts/prefabs/wx78.lua#L788-L791)、[重定向](../../../raw/current/scripts/prefabs/wx78.lua#L508-L510)）。
- 优先使用：改上限用 `SetMax`，改当前值用 `SetCurrent`/`DoDelta`，充能源用 `AddChargeSource`/`RemoveChargeSource`（[接口](../../../raw/current/scripts/components/wx78_shield.lua#L72-L90)）。
- 重要副作用：`SetCurrent` 推 `wxshielddelta` 事件并按阈值切换生成护盾特效（[事件与特效](../../../raw/current/scripts/components/wx78_shield.lua#L92-L127)）；四个字段变化经 `wx78_classified` 同步客户端（[netvar](../../../raw/current/scripts/components/wx78_shield.lua#L3-L26)）。
- 修改边界：`amount >= 0`、`overtime`、`ignore_absorb` 或无 afflicter 时直接放行不吸收（[放行](../../../raw/current/scripts/components/wx78_shield.lua#L189-L191)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `OnTakeDamage(amount, overtime, cause, ignore_invincible, afflicter, ignore_absorb)` | amount 为负伤害值 | 达到穿透阈值（默认 15，[字段](../../../raw/current/scripts/components/wx78_shield.lua#L35)）时全吸收并返回 0；未达阈值按剩余电量表减伤，返回值即入血量（[实现](../../../raw/current/scripts/components/wx78_shield.lua#L188-L203)）。 |
| `Impenetrable()` | → `bool` | `currentshield >= penetrationthreshold`（[实现](../../../raw/current/scripts/components/wx78_shield.lua#L149-L151)）。 |
| `SetMax(amount)` / `GetMax()` | amount > 0 | 断言上限为正；`SetMax` 后 `DoDelta(0)` 触发一次同步（[实现](../../../raw/current/scripts/components/wx78_shield.lua#L86-L90)）。 |
| `SetCurrent(amount)` / `GetCurrent()` / `GetPercent()` / `SetPercent(p)` | 数值 | 写入即 clamp 并推事件；`SetPercent` 按上限换算（[实现](../../../raw/current/scripts/components/wx78_shield.lua#L92-L147)）。 |
| `AddChargeSource(source, amount, reason)` / `RemoveChargeSource(source, reason)` / `HasChargeSource()` | amount: 每秒回充量 | 加法 SourceModifierList；首个来源启动组件更新，全移除即停（[实现](../../../raw/current/scripts/components/wx78_shield.lua#L56-L84)）。 |
| `GetCanShieldCharge()` | → `bool` | 充能须：有更新、未满、6 秒内未交战且未受伤（[实现](../../../raw/current/scripts/components/wx78_shield.lua#L158-L186)）。 |

## 典型接入

### 模组电路给 WX-78 持续回充护盾

- 前提与选择：WX-78 已默认挂载（上限 1、当前 0），模组模块技能应像原版蜂王冠电路一样在技能激活时 `SetMax` 与 `AddChargeSource`。
- 生效与撤销：充能源键用稳定字符串，技能停用时配对 `RemoveChargeSource`；`OnUpdate` 每帧按 `chargegenerationsources:Get()*dt` 回充（[回充](../../../raw/current/scripts/components/wx78_shield.lua#L207-L219)）。
- 原版依据：蜜蜂电路按模块数设置上限与每秒回充（[bee 电路](../../../raw/current/scripts/wx78_moduledefs.lua#L1115-L1132)）；备用机体模块在换装时保存/恢复当前电量（[backupbody](../../../raw/current/scripts/prefabs/wx78_backupbody.lua#L301-L302)）；控制台 `SetPercent` 调试（[console](../../../raw/current/scripts/consolecommands.lua#L436-L437)）。

```lua
-- 服务端，wx 为 WX-78 玩家实体
wx.components.wx78_shield:SetMax(50)
wx.components.wx78_shield:AddChargeSource(wx, 2, "MYMOD_CIRCUIT") -- 每秒回充 2 点
-- 停用：
wx.components.wx78_shield:RemoveChargeSource(wx, "MYMOD_CIRCUIT")
```

- 调整与验证：护盾体感阈值默认 15，原版把上限设为 1 时护盾"完全不穿透"不成立——达到 15 才 `Impenetrable`；监听 `wxshielddelta` 可做 UI（事件数据含 old/newpercent、maxshield、penetrationthreshold，[数据](../../../raw/current/scripts/components/wx78_shield.lua#L99-L104)）。

## 常见陷阱与限制

- 默认上限是 100 但原版挂载即 `SetMax(1)`（[挂载](../../../raw/current/scripts/prefabs/wx78.lua#L789-L790)），按 0~1 百分比体系运行；直接写大数值会脱离原版 UI 假设。
- `OnTakeDamage` 的返回值交给 `health.deltamodifierfn` 消费：想接入非 WX-78 实体需自己仿写重定向钩子，组件不会自动拦截伤害（原版接线见 [wx78.lua](../../../raw/current/scripts/prefabs/wx78.lua#L791)）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，覆盖组件全文与 wx78、wx78_possessedbody 挂载、wx78_moduledefs 蜜蜂电路、wx78_backupbody、consolecommands 消费点；未运行游戏。
- 结构检查：validate_page.py、validate_source_links.py 与 Lua 语法探针通过。
