---
title: "wx78_taserbuildup"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "WX-78 电击技能积攒组件：受击/格挡充能、闲置漏电，满值或主动释放触发电爆，按模块数动态挂载与调参。"
aliases: ["WX78电击积攒", "wx78_taserbuildup", "泰瑟积攒", "电爆积攒"]
tags: [component, energy]
dst_build: "steam-buildid:26230496"
sources: ["components/wx78_taserbuildup.lua", "wx78_moduledefs.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# wx78_taserbuildup

## 快速判断

- 职责与适用：WX-78"电击一类"技能的积攒层：监听宿主 `attacked`/`blocked` 事件按增益率充能，无伤害输入经过漏电缓冲后按速率衰减（[监听](../../../raw/current/scripts/components/wx78_taserbuildup.lua#L17-L19)、[衰减](../../../raw/current/scripts/components/wx78_taserbuildup.lua#L226-L232)）。
- 优先使用：不在 prefab 静态挂载——技能模块激活时动态 `AddComponent`，关闭时经 `ReleaseBuildup` 回调移除（[激活](../../../raw/current/scripts/wx78_moduledefs.lua#L691-L693)、[移除](../../../raw/current/scripts/wx78_moduledefs.lua#L684-L688)）。
- 重要副作用：积攒≥50 与满值时用 talker 播报台词；满值立即自动 `ReleaseBuildup`（[播报](../../../raw/current/scripts/components/wx78_taserbuildup.lua#L47-L56)）；爆炸给宿主推 `electrocute` 并临时无视绝缘（[爆炸](../../../raw/current/scripts/components/wx78_taserbuildup.lua#L148-L157)）。
- 修改边界：释放要求积攒比例 ≥0.25，且已有引爆倒计时中再调只更新回调（[释放](../../../raw/current/scripts/components/wx78_taserbuildup.lua#L128-L142)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SetCurrentBuildup(value)` / `GetBuildupPercent()` | 数值，clamp 到 [0,max] | 写入并管理更新启停与阈值播报（[实现](../../../raw/current/scripts/components/wx78_taserbuildup.lua#L43-L71)）。 |
| `DoDelta(delta)` | delta: number | 正值刷新 last_buildup_time；内部走 SetCurrentBuildup（[实现](../../../raw/current/scripts/components/wx78_taserbuildup.lua#L184-L189)）。 |
| `ReleaseBuildup(time, cb)` | time 缺省 3 秒 | 比例 ≥0.25 时令 FX 闪光并设定引爆倒计时；返回是否受理（[实现](../../../raw/current/scripts/components/wx78_taserbuildup.lua#L128-L142)）。 |
| `SetBuildupGainRate(rate)` / `SetBuildupDrainRate(rate)` / `SetBlastDamage(d)` / `SetBlastRadius(r)` | number | 增益乘数、漏电速率、爆炸伤害与半径，模块数变化时由 moduledefs 重算（[调参](../../../raw/current/scripts/wx78_moduledefs.lua#L667-L678)）。 |
| `SetDetonateTimer(time, cb)` | 秒 | 直接设定引爆倒计时与回调（[实现](../../../raw/current/scripts/components/wx78_taserbuildup.lua#L105-L110)）。 |
| `OnSave()` / `OnLoad(data)` | 组件存档钩子 | 仅保存 `current`（[实现](../../../raw/current/scripts/components/wx78_taserbuildup.lua#L235-L241)）。 |

## 典型接入

### 乘上技能模块的参数体系

- 前提与选择：组件默认值来自 `TUNING.SKILLS.WX78.TASER_*`（[构造](../../../raw/current/scripts/components/wx78_taserbuildup.lua#L5-L15)）；原版挂载完全由技能树驱动，模组仿制应同样动态挂载。
- 生效与撤销：受击增益前有 `IsEquipmentOnAttackedOrBlocked` 判定（对应装备才充能，[判定](../../../raw/current/scripts/components/wx78_taserbuildup.lua#L173-L182)）；`OnRemoveFromEntity` 反注册事件并移除 FX（[清理](../../../raw/current/scripts/components/wx78_taserbuildup.lua#L24-L31)）。
- 原版依据：每模块提升增益乘数、漏电基率与爆炸伤害/半径（[moduledefs](../../../raw/current/scripts/wx78_moduledefs.lua#L667-L678)）；技能停用时 `ReleaseBuildup(1, ...)` 强制快速引爆并在回调里移除组件（[停用](../../../raw/current/scripts/wx78_moduledefs.lua#L700-L705)）。

```lua
-- 服务端，wx 为 WX-78 实体（技能激活路径的仿写）
if wx._taser_modules > 0 and not wx.components.wx78_taserbuildup then
    wx:AddComponent("wx78_taserbuildup")
end
wx.components.wx78_taserbuildup:DoDelta(20)          -- 手动充能
wx.components.wx78_taserbuildup:ReleaseBuildup(1)     -- 1 秒后电爆
```

- 调整与验证：爆炸伤害/半径按当前比例缩放（`fx:Explode(damage*perc, radius*perc)`，[缩放](../../../raw/current/scripts/components/wx78_taserbuildup.lua#L150)）；特效实体 `wx78_taser_projectile_fx` 由组件自管，勿手工增删（[FX](../../../raw/current/scripts/components/wx78_taserbuildup.lua#L120-L126)）。

## 常见陷阱与限制

- 组件只服务 WX-78 形态假设：爆炸路径读 `self.inst.components.inventory` 与 `sg`（[爆炸](../../../raw/current/scripts/components/wx78_taserbuildup.lua#L153-L157)），挂到无 inventory 的实体会报错。
- 漏电只在 `OnUpdate` 中发生（有积攒才 StartUpdatingComponent）；`OnSave` 不存 detonate 倒计时，存读交界处的 pending 引爆会丢失（[存档](../../../raw/current/scripts/components/wx78_taserbuildup.lua#L235-L237)）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，覆盖组件全文与 wx78_moduledefs 激活/停用/调参链、TUNING.SKILLS.WX78 常量；未运行游戏。
- 结构检查：validate_page.py、validate_source_links.py 与 Lua 语法探针通过。
