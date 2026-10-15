---
title: "hudindicatable"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "把实体注册进客户端 HUD 指示器系统：延迟登记到 hudindicatablemanager、按 shouldtrackfn 过滤，移除时注销；仅非专用端有意义。"
aliases: ["hudindicatable", "HUD 指示器注册", "离屏目标指示"]
tags: [component, ui, callback]
size_class: simple
dst_build: "steam-buildid:26230496"
sources: ["components/hudindicatable.lua", "components/hudindicatablemanager.lua", "prefabs/world.lua", "prefabs/wagstaff_npc.lua", "components/playertargetindicator.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# hudindicatable

## 快速判断

- 职责与适用：挂在需要"屏幕边缘指示器"的实体上（非专用端）；构造后下一帧把本体登记进 `TheWorld.components.hudindicatablemanager`，是否跟踪由 `shouldtrackfn(inst, viewer)` 决定。[组件全文](../../../raw/current/scripts/components/hudindicatable.lua#L1-L8)
- 优先使用：跟踪条件用 `SetShouldTrackFunction(fn)` 注入；默认恒真。管理器是世界级客户端组件（world prefab 仅在非专用端挂载 hudindicatablemanager）。[注入](../../../raw/current/scripts/components/hudindicatable.lua#L10-L12) [管理器挂载](../../../raw/current/scripts/prefabs/world.lua#L558-L562)
- 重要副作用：本体 `onremove` 时注销并推 `unregister_hudindicatable` 世界事件，订阅方（如 playertargetindicator）据此清理指示器。[注销](../../../raw/current/scripts/components/hudindicatable.lua#L18-L23)
- 修改边界：组件名是 `hudindicatable`（源码拼写，无第二个 i 前的 r），别按 indicatorable 猜；注册/注销只在 manager 组件存在时执行，专用服务器上为空操作。[存在守卫](../../../raw/current/scripts/components/hudindicatable.lua#L19)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SetShouldTrackFunction(fn)` | `fn(inst, viewer) -> boolean` | 替换默认恒真函数；读取方 playertargetindicator 每帧用 `ShouldTrack(viewer)` 过滤目标。[注入](../../../raw/current/scripts/components/hudindicatable.lua#L10-L16) [读取](../../../raw/current/scripts/components/playertargetindicator.lua#L45) |
| 延迟注册 | 构造内 `DoTaskInTime(0)` | 下一帧调 `RegisterWithWorldComponent`，保证世界组件先就绪；manager 按实体 GUID 去重登记。[注册](../../../raw/current/scripts/components/hudindicatable.lua#L4) [manager 登记](../../../raw/current/scripts/components/hudindicatablemanager.lua#L44-L48) |
| `UnRegisterWithWorldComponent()` | `-> 无` | 从 manager 移除并推 `unregister_hudindicatable`；onremove 与 OnRemoveFromEntity 双路径调用。[实现](../../../raw/current/scripts/components/hudindicatable.lua#L18-L23) |

## 典型接入

### 给 NPC／实体加屏幕指示（原版装配参照）

- 前提与选择：宿主在非专用端分支挂组件并注入跟踪函数——wagstaff_npc 两处（地面与地下版本）、moonstorm_static 的漫游体、player_common 均为此模式。[wagstaff](../../../raw/current/scripts/prefabs/wagstaff_npc.lua#L468-L471) [moonstorm](../../../raw/current/scripts/prefabs/moonstorm_static.lua#L507-L510)
- 生效与撤销：挂上即登记；实体删除或组件移除自动注销。指示器的绘制由 hudindicatorwatcher／playertargetindicator 消费 manager 的 items 表完成，本组件不画任何东西。[消费侧](../../../raw/current/scripts/components/playertargetindicator.lua#L72-L73)

```lua
-- prefab 内：仅非专用端挂载（参照 wagstaff_npc）
if not GLOBAL.TheNet:IsDedicated() then
    inst:AddComponent("hudindicatable")
    inst.components.hudindicatable:SetShouldTrackFunction(function(inst, viewer)
        return true
    end)
end
```

- 调整与验证：shouldtrackfn 的返回值决定离屏箭头是否出现；manager 断言只能存在于客户端（`assert(not TheNet:IsDedicated(), ...)`），服务端挂本组件不会报错但永远空转。[manager 断言](../../../raw/current/scripts/components/hudindicatablemanager.lua#L7)

## 常见陷阱与限制

- **在专用服务端指望它工作：** hudindicatablemanager 仅在非专用端挂载；专用端注册守卫让一切为空操作，指示器逻辑整体属于客户端显示层。[管理器挂载](../../../raw/current/scripts/prefabs/world.lua#L558-L562)
- **同步依赖注册时机：** 注册在构造后一帧（DoTaskInTime(0)），若实体在同一帧内被删除，onremove 监听会先注销未注册项——UnRegisterItem 对不存在项安全，但不要在构造同帧读 manager.items 找它。[时序](../../../raw/current/scripts/components/hudindicatable.lua#L4-L5)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（35 行），核对 world 的 manager 挂载条件、manager 登记／注销、wagstaff_npc／moonstorm_static／player_common 装配、playertargetindicator 读取。未审阅 hudindicatorwatcher 的绘制循环细节。
- 结构检查：2026-10-28 validate_page.py 通过；validate_source_links.py --strict 锚点通过（见批级记录）。
- 游戏运行：未运行；指示器显示未实测。
