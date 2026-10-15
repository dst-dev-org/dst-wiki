---
title: "golfclub"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "高尔夫球杆组件：对 golfable 目标登记瞄准占用，配合 reticule 蓄力并在挥杆命中时传递方向与速度。"
aliases: ["高尔夫球杆", "golfclub"]
tags: [component, item, actions]
dst_build: "steam-buildid:26230496"
sources: ["components/golfclub.lua", "prefabs/carnival_golfclub.lua", "components/golfclub_reticule.lua", "actions.lua", "stategraphs/SGwilson.lua", "prefabs/carnivalgame_golf_tee.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# golfclub

## 快速判断

- 职责与适用：挂在高尔夫球杆物品上（原版 prefab `carnivalgame_golfclub`，[挂接](../../../raw/current/scripts/prefabs/carnival_golfclub.lua#L73)），维护"当前正在瞄准哪个 golfable 目标"的独占占用，并把挥杆结果交给 [golfable](../../../raw/current/scripts/components/golfable.lua#L8) 侧。
- 优先使用：瞄准由 `GOLF_START_AIMING` 动作启动（([actions.lua:7064-7072](../../../raw/current/scripts/actions.lua#L7064-L7072)），挥杆计时由 SGwilson 攻击状态驱动（[调用](../../../raw/current/scripts/stategraphs/SGwilson.lua#L28654-L28655)）。
- 重要副作用：命中后推 `golfclub_onswinghit` 事件（携带 golfable、doer、speed），球座以此结算击球（[事件](../../../raw/current/scripts/components/golfclub.lua#L81)）。
- 修改边界：组件移除／宿主删除时自动 `StopAiming` 释放占用（[别名](../../../raw/current/scripts/components/golfclub.lua#L86-L87)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `StartAiming(doer, target)` | `-> bool, reason?` | 要求目标有 `golfable` 组件、有效且不在 limbo；被占用返回 `false, "INUSE"`；成功登记占用、设 reticule 目标并调 `onstartaimingfn`（[实现](../../../raw/current/scripts/components/golfclub.lua#L26-L48)）。 |
| `StopAiming()` | `-> 无` | 释放占用（`golfable:OnUnoccupied`）、清 reticule、调 `onstopaimingfn`（[实现](../../../raw/current/scripts/components/golfclub.lua#L50-L64)）。 |
| `IsAiming()` / `GetTarget()` | `-> bool / entity\|nil` | 以 `self.golfable` 是否为空判断（[实现](../../../raw/current/scripts/components/golfclub.lua#L18-L24)）。 |
| `OnStartSwing(doer)` | `-> number` | 返回蓄力倍率：有 reticule 时取 `CalculateChargingScale()` 并清目标，否则 1；由 SGwilson 挥杆状态调用（[实现](../../../raw/current/scripts/components/golfclub.lua#L67-L74)）。 |
| `OnSwingHit(doer, speed)` | `-> 无` | 调 `golfable:OnHit(doer, self, dir, speed)`、推 `golfclub_onswinghit`，随后总是 `StopAiming`（[实现](../../../raw/current/scripts/components/golfclub.lua#L77-L84)）。 |

## 典型接入

### 瞄准→蓄力→命中链路（原版参照）

- 前提与选择：动作层要求球杆已装备且由 doer 手持才允许 StartAiming（([actions.lua:7064-7072](../../../raw/current/scripts/actions.lua#L7064-L7072)）；目标侧约定见 [golfable 组件](../../../raw/current/scripts/components/golfable.lua#L1-L10)。
- 生效与撤销：SGwilson 在瞄准中断/切换时调 `StopAiming`（([SGwilson.lua:28577-28578](../../../raw/current/scripts/stategraphs/SGwilson.lua#L28577-L28578)）；两种挥杆路径都会到达 `OnSwingHit`（[普通](../../../raw/current/scripts/stategraphs/SGwilson.lua#L28669)、[满蓄力](../../../raw/current/scripts/stategraphs/SGwilson.lua#L28721)）。

```lua
-- 服务端：动作层等价调用（doer 已装备球杆）
local club = doer.components.inventory:GetEquippedItem(EQUIPSLOTS.HANDS)
local ok, reason = club.components.golfclub:StartAiming(doer, target)
if not ok then
    -- reason == "INUSE" 表示目标正被其他人瞄准
end
```

- 调整与验证：蓄力倍率来自 `golfclub_reticule` 组件（同杆挂载），速度上限 `TUNING.GOLF_MAX_SWING_SPEED`（[满蓄力路径](../../../raw/current/scripts/stategraphs/SGwilson.lua#L28721)）。

## 常见陷阱与限制

- **占用是组件级独占**：同一时刻一杆只瞄一个目标，一个目标也只能被一杆占用（INUSE），多杆并发瞄准会互相失败（[实现](../../../raw/current/scripts/components/golfclub.lua#L31-L33)）。
- **OnSwingHit 不校验命中对象**：doer 与 `self.golfable` 都非 nil 就调 OnHit；未瞄准时挥杆静默无效果并仅执行 StopAiming（[实现](../../../raw/current/scripts/components/golfclub.lua#L77-L84)）。
- **文件名与 prefab 名不一致**：源码文件 `carnival_golfclub.lua` 对应 prefab `"carnivalgame_golfclub"`，AddPrefabPostInit 时注意用 prefab 名（[文件尾](../../../raw/current/scripts/prefabs/carnival_golfclub.lua#L80)）。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；覆盖组件全文、carnival_golfclub 宿主、GOLF_START/STOP_AIMING 动作、SGwilson 五处调用、球座事件监听。
- 结构检查：本批 validate_page、validate_source_links 及示例 lua 语法探测（见批级记录）。
- 游戏运行：未运行。
