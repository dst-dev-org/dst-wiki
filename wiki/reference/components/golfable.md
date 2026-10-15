---
title: "golfable"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "让实体可被高尔夫球杆瞄准与击打：占用互斥标签 + 命中转向/施速 + 外力接口。"
aliases: ["可击打高尔夫球", "高尔夫目标", "golfable"]
tags: [component, item, actions]
dst_build: "steam-buildid:26230496"
sources: ["components/golfable.lua", "components/golfclub.lua", "prefabs/carnivalgame_golfball.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# golfable

## 快速判断

- 职责与适用：装在高尔夫球（或自定义"球"）上，与 `golfclub` 组件配对：球杆瞄准时占用、挥杆命中时按方向速度给物理冲量（[命中](../../../raw/current/scripts/components/golfable.lua#L65-L74)）。
- 优先使用：动作侧由 `componentactions` 按 `golfable` + 非 `golfable_occupied` 标签给出 `GOLF_START_AIMING`（[动作](../../../raw/current/scripts/componentactions.lua#L2354-L2357)），装组件即可被瞄准。
- 重要副作用：`golfclub` setter 自动增删 `golfable_occupied` 标签（[标签同步](../../../raw/current/scripts/components/golfable.lua#L1-L3)），`OnHit` 只对当前占用的球杆生效。
- 修改边界：组件自带 `enterlimbo` 监听与移除钩子，球进 limbo/组件被移除都会 `StopAiming`（[入 limbo](../../../raw/current/scripts/components/golfable.lua#L5-L10)、[清理](../../../raw/current/scripts/components/golfable.lua#L82-L95)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `golfclub` | golfclub 组件引用 or nil | 当前占用的球杆；被 setter 监听同步标签（[构造](../../../raw/current/scripts/components/golfable.lua#L12-L27)）。 |
| `SetOnOccupiedFn(fn)` / `SetOnUnoccupiedFn(fn)` | `fn(inst, doer, club_inst)` / `fn(inst, club_inst)` | 占用变化回调；原版球注释掉了这两个回调（[调用点](../../../raw/current/scripts/prefabs/carnivalgame_golfball.lua#L298-L300)）。 |
| `SetOnHitFn(fn)` | `fn(inst, doer, club_inst, dir, speed)` | 命中回调；原版球用它驱动动画/摩擦（[注册](../../../raw/current/scripts/prefabs/carnivalgame_golfball.lua#L301)）。 |
| `IsOccupied()` | → `boolean` | `golfclub ~= nil`。 |
| `OnOccupied(doer, golfclub)` | 由 golfclub 调用 | assert 未被占用，记引用并回调（[实现](../../../raw/current/scripts/components/golfable.lua#L46-L53)）。 |
| `OnHit(doer, golfclub, dir, speed)` | 由 golfclub 调用 | 仅当前占用球杆生效：设朝向并 `Physics:SetVel(speed*cosθ, 0, -speed*sinθ)`（[实现](../../../raw/current/scripts/components/golfable.lua#L65-L74)）。 |
| `OnExternalPhysics(doer, dir, speed)` | 外力入口 | 旋球器等非球杆来源推力，`onhitfn` 收 club=nil（[实现](../../../raw/current/scripts/components/golfable.lua#L76-L80)）。 |

## 典型接入

### 自定义可击打球体

- 前提与选择：宿主需要 `Physics`；pristine 前加 `golfable` tag 供动作过滤（构造已自动加，[标签](../../../raw/current/scripts/components/golfable.lua#L20)）。
- 生效与撤销：装上即纳入 `golfclub` 搜索（[搜索](../../../raw/current/scripts/components/golfclub.lua#L27-L36)）；命中链在 `golfclub` 挥杆动画节点推 `golfclub_onswinghit` 事件（[命中链](../../../raw/current/scripts/components/golfclub.lua#L78-L81)）。
- 原版依据：carnivalgame_golfball 安装并只挂 `OnHitFn`（[安装](../../../raw/current/scripts/prefabs/carnivalgame_golfball.lua#L298-L301)）；洞口传送台用 `OnExternalPhysics` 弹球（[调用](../../../raw/current/scripts/prefabs/carnivalgame_golf_tee.lua#L133)）。

```lua
-- 球类 prefab 的 master_postinit 内
inst:AddComponent("golfable")
inst.components.golfable:SetOnHitFn(function(inst, doer, club, dir, speed)
    -- dir 为角度（度），speed 为初速；物理冲量已由组件施加
end)
```

- 调整与验证：速度单位与 `TUNING.GOLF_MAX_SWING_SPEED` 同量纲；占用期间其他球杆瞄准会被 `IsOccupied` 拒绝。

## 常见陷阱与限制

- `OnHit` 的 assert 与守卫都绑定 `self.golfclub == golfclub`：外部直接调 `OnHit` 传别的球杆会静默不生效，外力请走 `OnExternalPhysics`。
- 移除标签的手工操作不需要：`OnRemoveFromEntity` 只移除 `golfable`，`golfable_occupied` 依赖 `StopAiming` 间接清掉（[注释](../../../raw/current/scripts/components/golfable.lua#L88)）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，覆盖组件全文、golfclub 配对链、carnivalgame_golfball/carnivalgame_golf_tee/componentactions 消费点；未运行游戏。
- 结构检查：validate_page.py、validate_source_links.py 与 Lua 语法探针通过。

相关：[golfclub](golfclub.md)、[golfpropitem](golfpropitem.md)
