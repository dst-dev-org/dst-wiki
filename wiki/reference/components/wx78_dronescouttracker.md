---
title: "wx78_dronescouttracker"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "WX-78 侦察无人机登记组件：维护无人机集合、分片存取，技能停用时释放全部无人机。"
aliases: ["侦察无人机追踪", "无人机登记", "wx78_dronescouttracker"]
tags: [component, network-sync, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/wx78_dronescouttracker.lua", "prefabs/wx78.lua", "prefabs/wx78_drone_scout.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# wx78_dronescouttracker

## 快速判断

- 职责与适用：挂在 WX-78 上（[挂接](../../../raw/current/scripts/prefabs/wx78.lua#L793-L795)），登记该玩家名下的侦察无人机（wx78_drone_scout），维护双向事件通知与跨分片存档（[全文](../../../raw/current/scripts/components/wx78_dronescouttracker.lua#L1-L7)）。
- 优先使用：无人机生成／加载时由无人机侧调宿主 `StartTracking(inst)`（[调用](../../../raw/current/scripts/prefabs/wx78_drone_scout.lua#L222-L223)）；宿主侧技能停用调 `ReleaseAllDrones`。
- 重要副作用：登记／注销向无人机推 `ms_dronescout_tracked/untracked`，宿主删除时向全部无人机推 `ms_dronescout_despawn`（[事件](../../../raw/current/scripts/components/wx78_dronescouttracker.lua#L30)、[despawn](../../../raw/current/scripts/components/wx78_dronescouttracker.lua#L9-L13)）；无人机侧有对应监听（([wx78_drone_scout.lua:325-327](../../../raw/current/scripts/prefabs/wx78_drone_scout.lua#L325-L327)）。
- 修改边界：无人机实体删除自动注销（onremove 监听，[注册](../../../raw/current/scripts/components/wx78_dronescouttracker.lua#L26)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `StartTracking(drone)` / `StopTracking(drone)` | `-> 无` | 集合登记／注销并推送事件、触发 `onstarttrackingfn/onstoptrackingfn`（[实现](../../../raw/current/scripts/components/wx78_dronescouttracker.lua#L23-L43)）。 |
| `SetOnStartTrackingFn(fn)` / `SetOnStopTrackingFn(fn)` | `fn(inst, drone)` | 宿主回调；WX-78 用它做无人机上限管理与视觉（[挂接](../../../raw/current/scripts/prefabs/wx78.lua#L794-L795)）。 |
| `ReleaseAllDrones()` | `-> 无` | 全部注销并向每台无人机推 `ms_dronescout_despawn`、清 `data`；技能停用与技能树未激活时调用（[实现](../../../raw/current/scripts/components/wx78_dronescouttracker.lua#L45-L51)、[调用](../../../raw/current/scripts/prefabs/wx78.lua#L572-L584)）。 |
| `OnSave` / `OnLoad` | 存读钩子 | 本分片无人机存实体记录（`GetSaveRecord`），按分片 ID 键控；其他分片数据原样保留到 `self.data` 不生成实体（[实现](../../../raw/current/scripts/components/wx78_dronescouttracker.lua#L53-L91)）。 |

## 典型接入

### 技能生命周期联动（原版参照）

- 前提与选择：`wx78_scoutdrone_1` 技能停用（OnDeactivateSkill）或技能树初始化未激活时释放全部无人机（([wx78.lua:572-584](../../../raw/current/scripts/prefabs/wx78.lua#L572-L584)）。
- 生效与撤销：无人机存档里带 builder 引用，加载时回登记（([wx78_drone_scout.lua:222-223](../../../raw/current/scripts/prefabs/wx78_drone_scout.lua#L222-L223)）。

```lua
-- 服务端：宿主侧等价调用（WX-78 master postinit 已挂组件）
local tracker = inst.components.wx78_dronescouttracker
tracker:SetOnStartTrackingFn(function(inst, drone)
    -- 无人机开始跟随
end)
tracker:SetOnStopTrackingFn(function(inst, drone)
    -- 无人机停止跟随
end)
```

- 调整与验证：无人机实体自身负责生成／回收，本组件只管登记与通知。

## 常见陷阱与限制

- **跨分片数据不落地**：OnLoad 只生成当前分片的无人机，其他分片记录存 `self.data` 等切分片时随存档带过去（[实现](../../../raw/current/scripts/components/wx78_dronescouttracker.lua#L73-L91)）。
- **无数据时不写存档键**：OnSave 无无人机且无跨分片数据时返回 nil，组件在存档中整键省略（[返回](../../../raw/current/scripts/components/wx78_dronescouttracker.lua#L70)）。
- **OnRemoveEntity 与 ReleaseAllDrones 不同**：宿主被删除时只推 despawn 不逐个 StopTracking（避免改遍历中的集合）；主动释放才走 ReleaseAllDrones（[对比](../../../raw/current/scripts/components/wx78_dronescouttracker.lua#L9-L13)）。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；覆盖组件全文、WX-78 挂接与技能两处释放、无人机侧登记与事件监听。
- 结构检查：本批 validate_page、validate_source_links 及示例 lua 语法探测（见批级记录）。
- 游戏运行：未运行。
