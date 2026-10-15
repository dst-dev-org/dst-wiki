---
title: "steeringwheeluser"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "玩家操舵组件：登舵占位、把玩家钉在舵轮上、按指向改船舵方向并驱动转向动画，断舵/移除自动回落。"
aliases: ["操舵", "舵轮玩家", "steeringwheeluser"]
tags: [component]
dst_build: "steam-buildid:26230496"
sources: ["components/steeringwheeluser.lua", "components/steeringwheel.lua", "components/playercontroller.lua", "prefabs/player_common.lua", "prefabs/steeringwheel.lua", "stategraphs/SGwilson.lua", "actions.lua", "networkclientrpc.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# steeringwheeluser

## 快速判断

- 职责与适用：玩家角色侧的操舵能力，与舵轮（steeringwheel 组件）成对工作；所有船体动力学都转交给 `boatphysics` 的舵向接口，本组件只管"人-舵"绑定与输入转译。[构造](../../../raw/current/scripts/components/steeringwheeluser.lua#L1-L22)
- 优先使用：登舵/离舵用 `SetSteeringWheel(wheel 或 nil)`，按点位转向用 `Steer(pos_x, pos_z)`，按方向向量转向用 `SteerInDir(dir_x, dir_z)`，查所在船用 `GetBoat()`。[SetSteeringWheel](../../../raw/current/scripts/components/steeringwheeluser.lua#L24-L73)
- 重要副作用：登舵加 `steeringboat` tag、把玩家 `Physics:Teleport` 到舵轮并持续钉位；转向差异超容差时置 `should_play_left_turn_anim` 并推 `set_heading` 让 SG 播左/右舵动画。[SteerInDir](../../../raw/current/scripts/components/steeringwheeluser.lua#L87-L101)
- 修改边界：服务端组件；舵轮被移除、SG 失去 `is_using_steering_wheel` tag 都会自动 `SetSteeringWheel(nil)`；离舵保留当前舵向（不回中）。[OnUpdate](../../../raw/current/scripts/components/steeringwheeluser.lua#L107-L120)

## 关键接口

| 接口／字段 | 已核验类型／单位 | 条件、作用与证据 |
|---|---|---|
| `SetSteeringWheel(wheel)` | 无返回 | 幂等；换舵先完整清理旧舵（停更新、摘 tag、解监听、旧舵 `StopSteering`、SG 转 `stop_steering`）；新舵则钉位并 `StartSteering(inst)`，同时监听船的 `stopturning` 与 `onremove`。[L24-L73](../../../raw/current/scripts/components/steeringwheeluser.lua#L24-L73) |
| `Steer(pos_x, pos_z)` | 布尔 | 无船返回 false；否则把"船指向目标点"的方向归一化后走 `SteerInDir`，`SET_HEADING` 动作入口。[L75-L85](../../../raw/current/scripts/components/steeringwheeluser.lua#L75-L85)、[动作](../../../raw/current/scripts/actions.lua#L4939-L4945) |
| `SteerInDir(dir_x, dir_z)` | 无返回 | `boatphysics:SetTargetRudderDirection`；当前舵向与目标各轴差 < 0.1 时不推动画事件；叉积 `rz*dx - rx*dz > 0` 判左转。[L87-L101](../../../raw/current/scripts/components/steeringwheeluser.lua#L87-L101) |
| `GetBoat()` | 实体或 nil | `inst:GetCurrentPlatform()` 快捷方式。[L103-L105](../../../raw/current/scripts/components/steeringwheeluser.lua#L103-L105) |
| `should_play_left_turn_anim` | 布尔 | SG `steer_boat` 系状态据此选 `steer_left_*`／`steer_right_*` 动画。[读取](../../../raw/current/scripts/stategraphs/SGwilson.lua#L13063-L13067) |
| `wheel_remove_callback`（内部） | 无返回 | 舵轮实体移除：停更新、摘 `steeringboat` tag、旧舵 `StopSteering`、推 `stop_steering_boat`。[L5-L15](../../../raw/current/scripts/components/steeringwheeluser.lua#L5-L15) |

## 输入链路

- 动作：`STEER_BOAT`（目标舵轮空闲、未燃烧）→ `SetSteeringWheel(target)`；`SET_HEADING` → `Steer`；`STOP_STEERING_BOAT` → `SetSteeringWheel(nil)`。[动作组](../../../raw/current/scripts/actions.lua#L4928-L4952)
- 本地/远端：master 直接 `SteerInDir`，客户端经 RPC `SteerBoat` 发方向给服务端执行。[playercontroller](../../../raw/current/scripts/components/playercontroller.lua#L4173-L4180)、[RPC](../../../raw/current/scripts/networkclientrpc.lua#L508-L518)
- 解绑来源：舵轮组件在 `StopSteering` 时反调 `sailor:SetSteeringWheel(nil)`（组件与 prefab 两处），舵轮实体移除走回调。[组件侧](../../../raw/current/scripts/components/steeringwheel.lua#L46-L48)、[prefab 侧](../../../raw/current/scripts/prefabs/steeringwheel.lua#L68)
- 挂载：player_common master 段统一 `AddComponent("steeringwheeluser")`。[挂载](../../../raw/current/scripts/prefabs/player_common.lua#L2842)

## 典型接入

### 模组：监听操舵行为

- 前提与选择：服务端；监听玩家实体事件而非包装组件方法即可覆盖登/离舵（`stop_steering_boat`、`playerstopturning`、`set_heading`）。
- 生效与撤销：不改变组件状态；想强制玩家离舵调 `SetSteeringWheel(nil)`（会触发 SG 收尾与舵向保留）。[L67-L72](../../../raw/current/scripts/components/steeringwheeluser.lua#L67-L72)

```lua
-- modmain.lua：记录上舵/离舵（服务端）
AddPlayerPostInit(function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    inst:ListenForEvent("stop_steering_boat", function(inst)
        print("left the wheel:", inst:GetDisplayName())
    end)
end)
```

- 调整与验证：示例仅语法检查未运行；`set_heading` 只在转向幅度超容差时推送，缓慢修正不触发。

## 常见陷阱与限制

- 钉位靠组件 `OnUpdate` 每帧 `Transform:SetPosition(舵轮位置)`：SG 打断（如被传送、攻击）时由 `HasStateTag("is_using_steering_wheel")` 失败触发离舵，若模组改状态图标签需保留该 tag。[OnUpdate 守卫](../../../raw/current/scripts/components/steeringwheeluser.lua#L113-L117)
- 离舵不回中船舵：`SetSteeringWheel(nil)` 把当前舵向读出后原样设回目标方向，船会保持转向惯性继续转；要回中需另行调 `boatphysics`。[L67-L72](../../../raw/current/scripts/components/steeringwheeluser.lua#L67-L72)
- `SetSteeringWheel` 假定 `inst.sg` 存在：仅玩家/有 SG 实体可用；给无状态图实体挂组件再登舵会报错。[L37-L40](../../../raw/current/scripts/components/steeringwheeluser.lua#L37-L40)
- 服务器传送玩家到舵轮用 `Physics:Teleport`，登舵瞬间玩家实体位置跳变；客户端表现依赖 SG `steer_boat` 状态里的无脸朝向处理。[L58](../../../raw/current/scripts/components/steeringwheeluser.lua#L58)、[SG 状态](../../../raw/current/scripts/stategraphs/SGwilson.lua#L13054-L13069)

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。覆盖 steeringwheeluser.lua 全文（含回调与 OnUpdate 守卫）；核对 steeringwheel 组件/prefab 的反向解绑、三个动作、RPC 与 playercontroller 双端输入、SGwilson 的动画读取与挂载点；未覆盖 boatphysics 的舵向→船体姿态结算。
- 游戏运行：未运行；本页结论均来自源码静态核验。
- 结构检查：`validate_page.py`、`validate_source_links.py --strict` 与示例 Lua 语法检查通过（见批记录）。

相关：[[reference/components/steeringwheel]]、[[reference/components/boatphysics]]。
