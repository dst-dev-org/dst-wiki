---
title: "steeringwheel（船舵）"
created: 2026-10-21
updated: "2026-10-28"
type: component
summary: "船舵：记录驾驶者绑定与占用状态，转向输入由 steeringwheeluser 转成船的目标舵向"
aliases: ["船舵", "方向盘", "驾驶", "steeringwheel"]
tags: [component, network-sync]
dst_build: "steam-buildid:26230496"
sources: ["components/steeringwheel.lua", "prefabs/steeringwheel.lua", "components/steeringwheeluser.lua", "components/boatphysics.lua", "components/playercontroller.lua", "prefabs/player_common.lua", "actions.lua", "componentactions.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# steeringwheel（船舵）

## 快速判断

- 职责与适用：船舵组件只维护“谁在驾驶”这一绑定：记录 `sailor`、加/移 `occupied` 标签、在开始/停止时调用外部回调；它不计算转向角，也不直接给船加速。[构造与标签](../../../raw/current/scripts/components/steeringwheel.lua#L1-L10)
- 链上位置：船—帆—舵—阻力链的“舵”端。船舵提供可交互目标，`steeringwheeluser` 把玩家输入写成船的 `target_rudder_direction`，`boatphysics` 再把它积分成 `rudder_direction` 并旋转船体/桅杆；帆力与阻力分别由 mast、boatdrag 提供。
- 优先使用：要让玩家驾驶，走原版链路——船舵 `AddComponent("steeringwheel")`，玩家侧 `steeringwheeluser:SetSteeringWheel(wheel)`，交互入口用 `STEER_BOAT` 动作；不要只调 `StartSteering` 就算完成接入。[绑定入口](../../../raw/current/scripts/components/steeringwheeluser.lua#L24-L73)
- 重要副作用：`StartSteering` 会给舵加 `occupied` 并监听驾驶者的 `onremove`；`StopSteering` 移除标签并回调 `onstopfn`。舵被移除/烧毁/锤掉时会清掉驾驶者手上的轮引用，避免玩家停在驾驶状态。[移除清理](../../../raw/current/scripts/components/steeringwheel.lua#L44-L52)
- 修改边界：服务端逻辑；舵必须放在船上，`steeringwheeluser` 在绑定时缓存 `GetCurrentPlatform()`，不在船上时转向请求直接返回 `false`。客户端只负责发 `SteerBoat` RPC，不运行本组件。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `StartSteering(sailor)` | `sailor` 为玩家实体 | 设 `self.sailor`、监听该玩家的 `onremove`、加 `occupied`，最后调用 `onstartfn(self.inst, sailor)`；不校验玩家是否已有其他舵。[L21-L30](../../../raw/current/scripts/components/steeringwheel.lua#L21-L30) |
| `StopSteering()` | 无参 | 移除 `occupied`、调用 `onstopfn(self.inst, self.sailor)`、清空 `sailor`；`self.sailor == nil` 时仍会执行回调（回调收到 `nil`）。[L32-L42](../../../raw/current/scripts/components/steeringwheel.lua#L32-L42) |
| `SetOnStartSteeringFn(fn)` / `SetOnStopSteeringFn(fn)` | `fn(inst, sailor)` | 原版用来隐藏/显示轮盘符号、给驾驶者加轮盘皮肤覆盖；不设置则没有表现处理。[L13-L19](../../../raw/current/scripts/components/steeringwheel.lua#L13-L19) |
| `sailor` | 玩家实体或 `nil` | 只读用途：`boatphysics:GetRudderTurnSpeed` 在船半径内找 `steeringwheel.sailor`，据此判断 `master_crewman` 的转向加成。[L416-L441](../../../raw/current/scripts/components/boatphysics.lua#L416-L441) |
| `onsailorremoved` | 内部回调 | 驾驶者被移除时，若仍是当前 `sailor` 就 `StopSteering()`；这是组件对玩家消失的唯一自动兜底。[L9-L9](../../../raw/current/scripts/components/steeringwheel.lua#L9 |
| `OnRemoveFromEntity()` | 引擎回调 | 有驾驶者时优先走 `steeringwheeluser:SetSteeringWheel(nil)`，让玩家侧完整退出；没有 user 组件才直接 `StopSteering()`。[L44-L52](../../../raw/current/scripts/components/steeringwheel.lua#L44-L52) |
| `GetDebugString()` | 返回字符串 | 仅调试显示 `Sailor: <实体>`；不参与逻辑。[L54-L56](../../../raw/current/scripts/components/steeringwheel.lua#L54-L56) |

## 典型接入

### 原版船舵宿主（steeringwheel / yotd_steeringwheel）

- 前提与选择：目标文件 `scripts/prefabs/steeringwheel.lua`。服务端 `common_fn` 在 `SetPristine` 之后 `AddComponent("steeringwheel")`，随即挂上开始/停止回调；YOTD 变体只换 build，逻辑相同。[组件接入](../../../raw/current/scripts/prefabs/steeringwheel.lua#L120-L126)
- 加载与端：`common_fn` 先建 Transform/AnimState/SoundEmitter/Network；`TheWorld.ismastersim` 为假时在第 116-118 行返回，客户端只有表现实体，组件与回调只在服务端。[端判断](../../../raw/current/scripts/prefabs/steeringwheel.lua#L115-L124)
- 注册名与资源：`steeringwheel` 与 `yotd_steeringwheel`；资源为 `anim/boat_wheel.zip`、`anim/seafarer_wheel.zip`、`anim/yotd_steeringwheel.zip` 及物品图标；部署套件是 `steeringwheel_item`/`yotd_steeringwheel_item`，带 `boat_accessory` 标签、`DEPLOYSPACING.MEDIUM`。[注册](../../../raw/current/scripts/prefabs/steeringwheel.lua#L159-L170)
- 生效：舵部署在船上后，玩家对其执行 `STEER_BOAT`；动作要求目标有 `steeringwheel` 组件、`sailor == nil`、未燃烧，且玩家有 `steeringwheeluser`。[动作实现](../../../raw/current/scripts/actions.lua#L4928-L4937) 动作候选还要求舵未带 `occupied`、未带 `fire` 标签。[候选条件](../../../raw/current/scripts/componentactions.lua#L831-L835)
- 转向输入：玩家每帧由 `PlayerController:DoBoatSteering` 取方向；服务端调 `steeringwheeluser:SteerInDir`，客户端发 `RPC.SteerBoat`。`SteerInDir` 只写船的 `target_rudder_direction`，并据角度差决定是否播放转向动画/推 `set_heading`。[输入](../../../raw/current/scripts/components/playercontroller.lua#L4164-L4181) [写入目标舵向](../../../raw/current/scripts/components/steeringwheeluser.lua#L87-L101)
- 撤销：再次交互/离开驾驶位走 `STOP_STEERING_BOAT` → `SetSteeringWheel(nil)`；舵被锤掉、烧毁或移除时，prefab 与组件的清理会解绑驾驶者。`steeringwheeluser:SetSteeringWheel(nil)` 还会把目标舵向对齐当前舵向，避免松手后继续转。[停止动作](../../../raw/current/scripts/actions.lua#L4947-L4952) [舵烧毁](../../../raw/current/scripts/prefabs/steeringwheel.lua#L61-L77)

服务端最小挂接骨架（放在已 `SetPristine` 的自定义舵 prefab `fn_pst` 内；不是完整 prefab，部署/物理/动画需另备）：

```lua
local steeringwheel = inst:AddComponent("steeringwheel")
steeringwheel:SetOnStartSteeringFn(on_start_steering)
steeringwheel:SetOnStopSteeringFn(on_stop_steering)
```

- 调整与验证：`steeringwheel` 标签由组件构造时添加，`boatphysics` 用它搜索船上正在使用的舵（判断 `master_crewman` 转向加成）；自定义舵漏加该标签会影响该判定。本页源码审阅范围见“验证记录”，未做游戏内运行。

## 常见陷阱与限制

- `StopSteering` 第 34 行调用的是 `ListenForEvent("onremove", self.onsailorremoved, self.sailor)`，而不是 `RemoveEventCallback`；原意应是移除监听。结果是旧驾驶者的 `onremove` 回调不会被清理；同一玩家反复上/下舵后，该回调可能仍被注册并在移除时再次触发 `onstopfn`。接入时不要把 `StopSteering` 当作已完全清场。[L32-L42](../../../raw/current/scripts/components/steeringwheel.lua#L32-L42)
- 只调 `StartSteering(player)` 不会让玩家真正“在驾驶”：`steeringwheeluser.steering_wheel` 没被设置，输入链不会写目标舵向，`StopSteering` 也不会走玩家状态图退出。原版由 `SetSteeringWheel` 同时调 `StartSteering`。[双向绑定](../../../raw/current/scripts/components/steeringwheeluser.lua#L52-L65)
- `STEER_BOAT` 的动作候选只看“没有 `occupied`”，执行时还会检查 `sailor == nil`；若模组手写 `sailor` 却不加 `occupied`，其他玩家会看到动作但执行失败；只加 `occupied` 而不在停止时移除，舵会一直不可用。[占用标签](../../../raw/current/scripts/components/steeringwheel.lua#L25
- 舵不在船上时不会报错：`steeringwheeluser.boat = GetCurrentPlatform()` 为 `nil`，`Steer` 直接返回 `false`，玩家表现为“转舵无反应”；把舵放在陆地/码头边缘最容易触发。[平台缓存](../../../raw/current/scripts/components/steeringwheeluser.lua#L52 [Steer](../../../raw/current/scripts/components/steeringwheeluser.lua#L75-L85)
- 转向不是客户端预测：客户端只发 `RPC.SteerBoat`，服务端 `boatphysics` 才积分 `rudder_direction` 并旋转船与桅杆；在客户端直接改 `steeringwheel.sailor` 或目标方向不会同步。[RPC](../../../raw/current/scripts/components/playercontroller.lua#L4174-L4180) [积分](../../../raw/current/scripts/components/boatphysics.lua#L506-L536)
- 驾驶者状态图丢失 `is_using_steering_wheel` 标签时，`OnUpdate` 会自动 `SetSteeringWheel(nil)` 并停止更新；这是状态机中断的兜底，不是舵组件主动检测。[OnUpdate](../../../raw/current/scripts/components/steeringwheeluser.lua#L107-L120)

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 锚点重定位 2 处（actions.lua STEER_BOAT／STOP_STEERING_BOAT 执行函数）；结论修订：无，仅锚点迁移。依据：枢纽 diff 审阅＋变更清单命中 actions.lua、componentactions.lua、components/playercontroller.lua（命中区域逐行核对均未漂移）。
- 源码审阅：2026-10-21，`steam-buildid:25643519`；完整阅读 `components/steeringwheel.lua`，并核验 `prefabs/steeringwheel.lua` 两个宿主与注册、`components/steeringwheeluser.lua` 的绑定/输入、`components/boatphysics.lua` 的舵向积分与船员加成、`components/playercontroller.lua` 的 RPC 分支、`actions.lua`/`componentactions.lua` 的动作入口、`prefabs/player_common.lua` 的 `steeringwheeluser` 挂载。未审阅所有状态图中 `STEER_BOAT` 的进入/退出细节。
- 游戏运行：未运行；未验证多人同时抢舵、延迟下 RPC 转向、烧毁瞬间的客户端表现。
- 结构检查：`validate_page.py` 通过（0 问题）；`validate_source_links.py --strict` 通过：本页 27 个 Lua 引用，0 error / 0 warning；本页 Lua 片段经 `parser_probe.exe --syntax-only` 通过（0 错误）。

相关页面：[[reference/components/boatphysics]]、[[reference/components/mast]]、[[reference/components/boatdrag]]。

## 相关链接

- [[reference/components/steeringwheeluser]]：玩家操舵端：登舵、转舵与断舵回落。
