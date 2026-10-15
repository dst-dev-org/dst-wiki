---
title: "winch"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "绞盘起锚机的收放线状态机：按瓦片海洋深度换算目标深度，以可调速度增减 line_length 并在触底/收满时回调与推事件；供 winch prefab 的 SG、UNLOAD_WINCH 动作与打捞流程驱动。"
aliases: ["winch", "绞盘", "起锚机", "收放线状态机"]
tags: [component, state-machine, callback, equipment]
dst_build: "steam-buildid:26230496"
sources: ["components/winch.lua", "prefabs/winch.lua", "actions.lua", "stategraphs/SGwinch.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# winch

## 快速判断

- 职责与适用：挂在 winch 实体上的状态机组件；`line_length` 从 0（收满）涨到 `GetCurrentDepth()`（放到底），速度为 `dt * raising_speed`／`dt * lowering_speed`，两个方向各在启动与到位时回调并推事件。[状态字段](../../../raw/current/scripts/components/winch.lua#L13-L35) [推进循环](../../../raw/current/scripts/components/winch.lua#L203-L219)
- 优先使用：想驱动绞盘 → `StartRaising()`／`StartLowering()`，到位回调与打捞入口用 `SetUnloadFn` 等 setter 注入（原版由 SGwinch 与 UNLOAD_WINCH 动作调用）。[动作消费](../../../raw/current/scripts/actions.lua#L5556-L5559)
- 重要副作用：`winch_ready` 字段变更自动同步 `winch_ready` tag（供动作过滤与 SG 使用）；`StartDepthTesting`／`StopDepthTesting` 控制组件更新开停。[tag 同步](../../../raw/current/scripts/components/winch.lua#L1-L11) [深度测试](../../../raw/current/scripts/components/winch.lua#L191-L201)
- 修改边界：深度来源是所在瓦片的 `ocean_depth` 类别乘 `TUNING.ANCHOR_DEPTH_TIMES`，可用 `SetOverrideGetCurrentDepthFn` 整体接管；组件随宿主存档恢复全部数值字段但**不**恢复回调函数，载入后必须重新 setter。[深度换算](../../../raw/current/scripts/components/winch.lua#L130-L145) [存档](../../../raw/current/scripts/components/winch.lua#L51-L64)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| 构造 | `(inst)` | `winch_ready = true`（tag 监听器）、`line_length = 0`、`is_static = true`、`raising_speed = 1`、`lowering_speed = .5`；原版宿主按是否抓物把 raising_speed 覆写为 1.8/1.1（`BOAT_WINCH.RAISING_SPEED_*`）。([winch.lua:13-35](../../../raw/current/scripts/components/winch.lua#L13-L35) [调谐](../../../raw/current/scripts/tuning.lua#L5493-L5502) |
| `GetCurrentDepth()` | 返回 number | `overridegetcurrentdepthfn` 优先；否则查瓦片 `GetTileInfo(tile).ocean_depth` × `TUNING.ANCHOR_DEPTH_TIMES`（LAND 0／SHALLOW 2／BASIC 6／DEEP 8／VERY_DEEP 10），无瓦片信息返回 0。([winch.lua:130-145](../../../raw/current/scripts/components/winch.lua#L130-L145) [深度表](../../../raw/current/scripts/tuning.lua#L4599-L4605) |
| `StartRaising(loading_in)`／`StartLowering(loading_in)` | 无参或载入标记 | 置 `is_static = false`、`is_raising` 并 `StartUpdatingComponent`；调 `onstartraisingfn`／`onstartloweringfn`，推 `start_raising_winch`／`start_lowering_winch`。([winch.lua:147-166](../../../raw/current/scripts/components/winch.lua#L147-L166) |
| `FullyRaised()`／`FullyLowered()` | 无参 | 收满清零线长，放到底把线长锁定为当前深度；各自停更新、调 `onfullyraisedfn`／`onfullyloweredfn`、推 `winch_fully_raised`／`winch_fully_lowered`。([winch.lua:168-189](../../../raw/current/scripts/components/winch.lua#L168-L189) |
| `OnUpdate(dt)` | 组件更新 | 收线时 `line_length` 递减至 0 触发 FullyRaised；放线时递增至深度触发 FullyLowered。([winch.lua:203-219](../../../raw/current/scripts/components/winch.lua#L203-L219) |
| `SetOnFullyLoweredFn` 等 6 个 setter | `(fn)` | `onfullyloweredfn`、`onfullyraisedfn`、`onstartraisingfn`、`onstartloweringfn`、`overridegetcurrentdepthfn`、`unloadfn`；`unloadfn` 由 `ACTIONS.UNLOAD_WINCH` 直接调用（打捞 [[reference/components/winchtarget]] 的货物）。([winch.lua:106-128](../../../raw/current/scripts/components/winch.lua#L106-L128) ([actions.lua:5556-5559](../../../raw/current/scripts/actions.lua#L5556-L5559) |
| `OnSave`／`OnLoad` | 存档钩子 | 持久化 `line_length`、`is_raising`、`is_static`、`winch_ready` 与两个速度；载入只还原数值，不动回调。([winch.lua:51-96](../../../raw/current/scripts/components/winch.lua#L51-L96) |

## 典型接入

### 给自定义起重设备复用收放线状态机

- 前提与选择：宿主需要 Transform（深度查瓦片）与 SG/事件接收侧；纯"按深度分类推进度"的设备可直接用，想自定义深度公式用 `SetOverrideGetCurrentDepthFn`。
- 生效与撤销：`StartLowering` 到 `FullyLowered`（或 `StartRaising` 到 `FullyRaised`）为一个行程；期间宿主侧逻辑（SG 动画、打捞判定）读 `line_length`／`is_raising` 联动。
- 原版依据：宿主 winch prefab 在平台放置/拆除时用 `is_static`＋`line_length` 判断恢复行程（([prefabs/winch.lua:426-437](../../../raw/current/scripts/prefabs/winch.lua#L426-L437)），SG 到点调 `FullyRaised`（[SGwinch.lua:84](../../../raw/current/scripts/stategraphs/SGwinch.lua#L84）。

```lua
-- 宿主 prefab master_postinit：
inst:AddComponent("winch")
inst.components.winch:SetUnloadFn(function(inst2)
    -- 与 winchtarget 打捞对接，返回是否成功
    return false
end)
inst.components.winch:SetOnFullyLoweredFn(function(inst2)
    -- 放到底：此处开始打捞判定
end)
-- 需要开始收线（如动画状态进入收拉）：
inst.components.winch:StartRaising()
```

- 调整与验证：速度单位是"深度单位/秒"，放线默认 0.5、收线默认 1（原版宿主会覆写为 `BOAT_WINCH.RAISING_SPEED_FAST/SLOW`）；`GetCurrentDepth` 只在瓦片带 `ocean_depth` 类别时非 0，陆地上恒 0。

## 常见陷阱与限制

- 回调不随存档恢复：`OnSave` 只存数值，`onfullyloweredfn` 等函数引用载入后为 nil（([winch.lua:86-96](../../../raw/current/scripts/components/winch.lua#L86-L96)），必须在宿主 OnLoad 里重新 setter，否则到位时既无回调也无事件以外的联动。
- 深度与线长解耦：放到底时 `line_length` 被写成当时的 `GetCurrentDepth()`，若随后船移到更深/更浅瓦片，线长不会自动重算，需重新触发 `StartLowering`/`StartRaising`（([winch.lua:180-189](../../../raw/current/scripts/components/winch.lua#L180-L189)）。
- 组件更新由 `is_static` 把守：直接改 `line_length` 而不走 `StartDepthTesting` 不会推进事件（([winch.lua:191-201](../../../raw/current/scripts/components/winch.lua#L191-L201)）；外部强制同步用 `FullyRaised`／`FullyLowered` 代替手改字段。
- `SetLoweringSpeedMultiplier`／`SetRaisingSpeedMultiplier` 名为 multiplier 实为直接覆写速度值（([winch.lua:98-104](../../../raw/current/scripts/components/winch.lua#L98-L104)），不是乘算。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，通读组件全文；核验 prefabs/winch.lua 挂载与恢复逻辑、SGwinch 调用、UNLOAD_WINCH 动作与 ANCHOR_DEPTH_TIMES／BOAT_WINCH 调谐。
- 游戏运行：未运行。
- 结构检查：未执行。

相关链接：[[reference/components/winchtarget]]、[[reference/components/mast]]、[[reference/components/hull]]
