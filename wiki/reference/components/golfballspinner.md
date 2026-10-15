---
title: "golfballspinner"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "嘉年华旋盘组件：按周期对周围 golfable 实体施加切向推力，让球绕盘旋转。"
aliases: ["高尔夫旋盘", "旋球器", "golfballspinner"]
tags: [component]
dst_build: "steam-buildid:26230496"
sources: ["components/golfballspinner.lua", "prefabs/carnivalgame_golfprops.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# golfballspinner

## 快速判断

- 职责与适用：装在嘉年华旋盘道具上（顺/逆时针两变体，[安装](../../../raw/current/scripts/prefabs/carnivalgame_golfprops.lua#L309-L311)、[逆时针](../../../raw/current/scripts/prefabs/carnivalgame_golfprops.lua#L331-L334)），每帧对半径内贴地的 `golfable` 实体施加切向加速度（[施力](../../../raw/current/scripts/components/golfballspinner.lua#L53-L89)）。
- 优先使用：做旋转平台/风场类推力时参照其"周期定速"模型：`accel = 2π * dist / period * dt`。
- 重要副作用：推力同时回调 `golfable:OnExternalPhysics`，触发球的 `onhitfn`（club=nil）（[外力链](../../../raw/current/scripts/components/golfballspinner.lua#L86)）。
- 修改边界：`SetEnabled(false)` 或实体睡眠都会停更新，唤醒自动恢复（[睡眠](../../../raw/current/scripts/components/golfballspinner.lua#L95-L105)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SetEnabled(enabled)` | `boolean` | 切换每帧更新；实体睡眠时只记状态不启动（[实现](../../../raw/current/scripts/components/golfballspinner.lua#L17-L28)）。 |
| `SetRadius(radius)` / `GetRadius()` | `number`，默认 1 | `FindEntities` 搜索半径（[使用](../../../raw/current/scripts/components/golfballspinner.lua#L56)）。 |
| `SetRadialStrength(radialstrength)` / `GetRadialStrength()` | `number`，默认 3 | 原版旋盘设 5；但该值在 `SpinGolfBalls` 内未被引用，当前版本无实际物理效果（[设值](../../../raw/current/scripts/components/golfballspinner.lua#L38-L44)，全函数无引用）。 |
| `SetIsCounterClockwise(ccw)` | `boolean` | 反转切向方向与朝向角（[反转](../../../raw/current/scripts/components/golfballspinner.lua#L75-L78)）。 |
| `period` | `number`，默认 2（秒） | 角速度周期：球公转一圈的时间，直接决定加速度大小（[公式](../../../raw/current/scripts/components/golfballspinner.lua#L80)）。 |

## 典型接入

### 自制旋转推力场

- 前提与选择：目标实体需有 `golfable` 组件（搜索按该 tag）；宿主需 `Transform`。
- 生效与撤销：`SetEnabled(true)` 后逐帧生效，组件移除/停用自然停止，无清理钩子需求（`OnRemoveFromEntity` 为空，[实现](../../../raw/current/scripts/components/golfballspinner.lua#L11-L13)）。
- 原版依据：旋盘在游戏激活时 `SetEnabled(true)` 并播转盘音效，停玩/收摊时关闭（[开关](../../../raw/current/scripts/prefabs/carnivalgame_golfprops.lua#L269-L284)）；被旋球击中判定在球的 `OnHit` 内查 `doer.components.golfballspinner`（[消费](../../../raw/current/scripts/prefabs/carnivalgame_golfball.lua#L130-L135)）。

```lua
-- 旋转平台 master_postinit 内
inst:AddComponent("golfballspinner")
inst.components.golfballspinner:SetRadius(3)
inst.components.golfballspinner.period = 2 -- 秒/圈，越小推得越快
inst.components.golfballspinner:SetEnabled(true)
```

- 调整与验证：推力施加的是速度增量（`Physics:GetVelocity` 后叠加 `SetVel`），非直接定位；高度 ≥0.1 的实体被跳过（[过滤](../../../raw/current/scripts/components/golfballspinner.lua#L59)）。

## 常见陷阱与限制

- `radialstrength` 是死配置：改它不会改变推力，实际强度只由 `period` 与距离决定（已核验 `SpinGolfBalls` 全函数，[实现](../../../raw/current/scripts/components/golfballspinner.lua#L53-L89)）。
- 圆心重合的球会取随机方向弹出（[特判](../../../raw/current/scripts/components/golfballspinner.lua#L62-L67)），批量生成时注意初始位置错开。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，覆盖组件全文与 carnivalgame_golfprops/carnivalgame_golfball 消费链；未运行游戏。
- 结构检查：validate_page.py、validate_source_links.py 与 Lua 语法探针通过。

相关：[golfable](golfable.md)、[golfclub](golfclub.md)
