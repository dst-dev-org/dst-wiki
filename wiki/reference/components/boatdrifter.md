---
title: "boatdrifter"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "船只漂移组件：维护船的移动／漂移状态，延迟停用船物理并管理唤醒测试，避免静止船持续模拟。"
aliases: ["船漂移", "boatdrifter"]
tags: [component, state-machine, callback]
dst_build: "steam-buildid:26230496"
sources: ["components/boatdrifter.lua", "prefabs/boat.lua", "components/boatphysics.lua", "components/walkableplatform.lua", "prefabs/yotd_boats.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# boatdrifter

## 快速判断

- 职责与适用：挂在船上（[boat.lua:579](../../../raw/current/scripts/prefabs/boat.lua#L579)、龙舟 [yotd_boats.lua:461](../../../raw/current/scripts/prefabs/yotd_boats.lua#L461），聚合"是否在移动／漂移"两个状态位，据此开关船物理与 PhysicsWaker 唤醒测试（[全文](../../../raw/current/scripts/components/boatdrifter.lua#L1-L8)）。
- 优先使用：不直接调本组件开关节奏——`OnStartMoving/OnStopMoving` 由 boatphysics 调（[调用](../../../raw/current/scripts/components/boatphysics.lua#L579-L589)），`OnStartDrifting/OnStopDrifting` 由 walkableplatform 调（[调用](../../../raw/current/scripts/components/walkableplatform.lua#L227-L232)）；睡眠／唤醒回调由引擎触发。
- 重要副作用：停止移动是延迟 1 秒的定时任务（`BOAT_PHYSICS_SLEEP_DELAY = 1`），期间任意启动会取消任务（[实现](../../../raw/current/scripts/components/boatdrifter.lua#L28-L41)）。
- 修改边界：`inst:StartBoatPhysics/StopBoatPhysics` 是宿主注入的方法（[注入](../../../raw/current/scripts/prefabs/boat.lua#L588-L589)），复用组件必须自行提供同签名方法。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `IsMoving()` / `IsDrifting()` | `-> bool` | IsDrifting = 移动中且漂移中（[实现](../../../raw/current/scripts/components/boatdrifter.lua#L20-L26)）。 |
| `StartBoatPhysics()` / `StopBoatPhysics()` | `-> 无` | 启动立即生效；停止排 1 秒延迟任务，任务期间再启动会先取消（[实现](../../../raw/current/scripts/components/boatdrifter.lua#L28-L41)）。 |
| `StartWakeTests()` / `StopWakeTests()` | `-> 无` | 转发 `inst.PhysicsWaker`（引擎接口）的唤醒测试开关（[转发](../../../raw/current/scripts/components/boatdrifter.lua#L43-L49)）。 |
| `OnStartMoving` / `OnStopMoving` | `-> 无` | 由 boatphysics 移动起止时调用；停止时若实体在睡眠区则排停止任务（[实现](../../../raw/current/scripts/components/boatdrifter.lua#L63-L77)）。 |
| `OnStartDrifting` / `OnStopDrifting` | `-> 无` | 由 walkableplatform 漂移起止时调用，联动唤醒测试（[实现](../../../raw/current/scripts/components/boatdrifter.lua#L51-L61)）。 |
| `OnEntitySleep` / `OnEntityWake` | `-> 无` | 静止入睡时停物理，唤醒时启动（[实现](../../../raw/current/scripts/components/boatdrifter.lua#L79-L87)）。 |

## 典型接入

### 自定义船的最小接法（原版参照）

- 前提与选择：与 `boatphysics`、`walkableplatform` 同挂；宿主需提供 `inst.StartBoatPhysics/StopBoatPhysics`（原版为 boat prefab 上的局部函数，[定义](../../../raw/current/scripts/prefabs/boat.lua#L335-L341)）。
- 生效与撤销：移动／漂移状态完全由协作组件驱动，本组件不自判速度；移除实体时取消任务并停唤醒测试（[清理](../../../raw/current/scripts/components/boatdrifter.lua#L10-L18)）。

```lua
-- 服务端：自定义船宿主（与原版 boat.lua 等价的最小骨架）
local function StartBoatPhysics(inst)
    -- 原版实现见 boat.lua:341
end

local function StopBoatPhysics(inst)
    -- 原版实现见 boat.lua:335
end

inst.StartBoatPhysics = StartBoatPhysics
inst.StopBoatPhysics = StopBoatPhysics
inst:AddComponent("boatdrifter")
```

- 调整与验证：`BOAT_PHYSICS_SLEEP_DELAY` 是文件顶部局部常量，改延迟需改组件源码或自写任务。

## 常见陷阱与限制

- **宿主方法缺失即报错**：`StartBoatPhysics` 直接调 `self.inst:StartBoatPhysics()`（[实现](../../../raw/current/scripts/components/boatdrifter.lua#L33)），自定义宿主没注入同名方法会在首次移动时报 nil 调用。
- **状态不自算**：is_moving／is_drifting 只在协作组件回调里翻转，直接读组件前确认船确实挂了 boatphysics／walkableplatform，否则状态恒 false。
- **清理对等**：`OnRemoveEntity` 与 `OnRemoveFromEntity` 都指向同一清理（[别名](../../../raw/current/scripts/components/boatdrifter.lua#L18)），动态 RemoveComponent 也会停掉唤醒测试。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；覆盖组件全文、boat／yotd_boats 挂接、boatphysics 与 walkableplatform 的全部调用点、宿主方法注入。
- 结构检查：本批 validate_page、validate_source_links 及示例 lua 语法探测（见批级记录）。
- 游戏运行：未运行。
