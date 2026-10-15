---
title: "vanish_on_sleep"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "实体休眠后延时消失：睡满 10 秒执行 vanishfn 并删除实体，唤醒取消计时；原版用于海盗船清场回收。"
aliases: ["vanish_on_sleep", "休眠消失", "睡眠移除"]
tags: [component, lifecycle, callback]
size_class: simple
dst_build: "steam-buildid:26230496"
sources: ["components/vanish_on_sleep.lua", "components/piratespawner.lua", "components/boatcrew.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# vanish_on_sleep

## 快速判断

- 职责与适用：挂在"离屏就该收拾掉"的临时实体上；实体进入休眠（OnEntitySleep）后起 10 秒任务，到期执行 `vanishfn` 再 `inst:Remove()`。[组件全文](../../../raw/current/scripts/components/vanish_on_sleep.lua#L11-L32)
- 优先使用：删除前的清理逻辑写在 `vanishfn` 字段（组件不初始化，由宿主赋值）；海盗船用它把船上物品送回 stash、清海盗、杀乘员。[宿主赋值](../../../raw/current/scripts/components/piratespawner.lua#L115-L119)
- 重要副作用：唤醒（OnEntityWake）取消未到期的任务；`OnRemoveFromEntity` 复用同一取消逻辑，防止已删除实体上的任务再触发。[取消](../../../raw/current/scripts/components/vanish_on_sleep.lua#L17-L25)
- 修改边界：依赖实体休眠系统（远端实体才睡）；`vanishfn` 内部自行保证可清理性（原版回调先判 walkableplatform 存在）。原版唯一 AddComponent 消费者是 piratespawner。[回调守卫](../../../raw/current/scripts/components/piratespawner.lua#L93-L95)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `vanishfn` | `fn(inst)`（宿主赋值） | vanish 前调用的清理回调；nil 时直接删除实体。[实现](../../../raw/current/scripts/components/vanish_on_sleep.lua#L27-L32) |
| `OnEntitySleep` | 实体休眠回调 | `vanish_task` 为空才起 `DoTaskInTime(10, DoVanish)`，重复休眠不会叠加任务。[实现](../../../raw/current/scripts/components/vanish_on_sleep.lua#L11-L15) |
| `OnEntityWake` | 实体唤醒回调 | 取消并清空 `vanish_task`；同一逻辑赋给 OnRemoveFromEntity。[实现](../../../raw/current/scripts/components/vanish_on_sleep.lua#L17-L25) |
| `vanish()` | `-> 无` | 执行 vanishfn 后 `inst:Remove()`；由延时任务触发。[实现](../../../raw/current/scripts/components/vanish_on_sleep.lua#L27-L32) |

## 典型接入

### 海盗船的休眠清场（原版参照）

- 前提与选择：piratespawner 的 `SetPirateBoat` 给无主海盗船挂本组件并赋 `vanishfn = OnPirateBoatVanish`——回调把船上物品送 stash、删除海盗、杀死带血量乘员后删船。[挂载](../../../raw/current/scripts/components/piratespawner.lua#L115-L119) [清场回调](../../../raw/current/scripts/components/piratespawner.lua#L93-L107)
- 生效与撤销：boatcrew 在船长移除或船员清空时 `RemoveComponent("vanish_on_sleep")`——有"归属"的船不再自动消失，这是取消消失的正式途径。[撤除](../../../raw/current/scripts/components/boatcrew.lua#L193) [撤除二](../../../raw/current/scripts/components/boatcrew.lua#L249)

```lua
-- 服务端：临时实体休眠后清理再删除
inst:AddComponent("vanish_on_sleep")
inst.components.vanish_on_sleep.vanishfn = function(inst)
    -- 送回物品／生成替代物等清理
end
```

- 调整与验证：延时固定 10 秒（组件内硬编码），不可配置；`vanishfn` 在删除前执行，回调里不需要再调 Remove。

## 常见陷阱与限制

- **指望玩家附近实体也消失：** 实体在视距内不会休眠，OnEntitySleep 不触发；该组件只清"离屏"实体，常驻实体需另配管理器轮询。[休眠前提](../../../raw/current/scripts/components/vanish_on_sleep.lua#L11-L15)
- **回调里做不可重复的假设：** 10 秒内实体被唤醒再休眠会重新计时，vanishfn 的执行时机不保证距挂载多久；清理逻辑必须幂等且以当前实体状态为准。[取消机制](../../../raw/current/scripts/components/vanish_on_sleep.lua#L17-L22)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（34 行），核对 piratespawner 挂载与清场回调、boatcrew 两处撤除；grep 全库确认无其他 AddComponent／字段消费者。类名为 `Vanish_on_sleep`（下划线混合大小写，保持原文）。
- 结构检查：2026-10-28 validate_page.py 通过；validate_source_links.py --strict 锚点通过（见批级记录）。
- 游戏运行：未运行。
