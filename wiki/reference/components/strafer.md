---
title: "strafer"
created: "2026-10-26"
updated: "2026-10-28"
type: component
summary: "为本地活动玩家采集独立瞄准朝向，经 locomotor 与 RPC 同步，区分输入瞄准、移动方向和服务端 strafing 状态。"
aliases: ["边走边瞄准", "侧移瞄准朝向", "StrafeFacing", "strafer"]
tags: [component, lifecycle, network-sync, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/strafer.lua", "prefabs/player_common.lua", "components/playercontroller.lua", "components/locomotor.lua", "prefabs/player_classified.lua", "networkclientrpc.lua", "input.lua", "entityscript.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# strafer

## 快速判断

- 职责与适用：活动玩家的本地输入桥接，读取鼠标／手柄瞄准方向，交 locomotor 调整朝向及运动相对方向；不是自动向侧面移动的 AI 或独立位移组件。[strafer.lua:51](../../../raw/current/scripts/components/strafer.lua#L51 [locomotor.lua:705](../../../raw/current/scripts/components/locomotor.lua#L705
- 优先使用：沿用原版活动玩家安装与 locomotor:StartStrafing／StopStrafing 协议；构造缓存 playercontroller，须在它建立之后安装。[strafer.lua:24](../../../raw/current/scripts/components/strafer.lua#L24 [player_common.lua:1001](../../../raw/current/scripts/prefabs/player_common.lua#L1001
- 重要副作用：客户端有预测 locomotor 时本地改朝向；没有时仍可将变化的方向发 StrafeFacing RPC。IsAiming 只表示本组件已进入输入更新，不保证当前输入有效或人物不忙。[strafer.lua:1](../../../raw/current/scripts/components/strafer.lua#L1 [strafer.lua:51](../../../raw/current/scripts/components/strafer.lua#L51
- 修改边界：原版只给 ThePlayer 的活动组件组添加它，可运行于本地主机或远程客户端；不应无条件加到每个服务端玩家／NPC 上，因为它读取 TheInput／TheCamera。[player_common.lua:722](../../../raw/current/scripts/prefabs/player_common.lua#L722 [player_common.lua:1036](../../../raw/current/scripts/prefabs/player_common.lua#L1036

## 关键接口

| 接口／字段 | 参数或返回 | 条件、作用与证据 |
|---|---|---|
| `IsAiming()` | 布尔值，构造时 false | 返回 aiming；startstrafing 且已缓存 playercontroller 才置 true 并开始组件更新，stopstrafing 清除。不是服务端是否 strafing 的总状态查询。[strafer.lua:1](../../../raw/current/scripts/components/strafer.lua#L1 [strafer.lua:39](../../../raw/current/scripts/components/strafer.lua#L39 |
| `playercontroller`／`ismastersim` | 构造时缓存 | controller 后加或替换不会自动刷新该引用；locomotor 则每次取 components，因为关闭移动预测的客户端可以没有它。[strafer.lua:24](../../../raw/current/scripts/components/strafer.lua#L24 |
| `OnUpdate(dt)` | 框架调用，无显式返回 | 控制器启用或处于允许有限操作的 HUD 阻挡模式时才继续，另要求 SG 不带 busy；失败时清 lastdir 并返回，不自动停止 aiming。[strafer.lua:51](../../../raw/current/scripts/components/strafer.lua#L51 [playercontroller.lua:490](../../../raw/current/scripts/components/playercontroller.lua#L490 |
| `lastdir` | 最近发送方向或 nil | 仅非主模拟端在发送 RPC 时更新；按数值是否变化判断是否发送，没有固定频率或角度量化。停止及输入被控制器／busy 门禁阻断时清空。[strafer.lua:12](../../../raw/current/scripts/components/strafer.lua#L12 [strafer.lua:51](../../../raw/current/scripts/components/strafer.lua#L51 |

## 关键流程

### 活动玩家安装与启停事件

OnSetOwner 先按所属端建立 playercontroller，再在 inst==ThePlayer 且需要建立 HUD 时调用 AddActivePlayerComponents，其中安装 strafer。取消活动 HUD 时 UnregisterHUD 调 RemoveActivePlayerComponents，移除本组件。不能在普通 Prefab 构造早期抢先安装后，指望将来新建的 controller 被自动发现。[player_common.lua:1001](../../../raw/current/scripts/prefabs/player_common.lua#L1001 [player_common.lua:722](../../../raw/current/scripts/prefabs/player_common.lua#L722 [player_common.lua:781](../../../raw/current/scripts/prefabs/player_common.lua#L781

服务端 locomotor:StartStrafing 设置自身 strafedir、classified.isstrafing=true，并发 startstrafing；StopStrafing 反向清除并发 stopstrafing。客户端 classified 的 isstrafingdirty 再把事件转发给父玩家。本组件构造也检查当前净变量，已在 strafing 时立即走开始逻辑，补上先发生状态、后安装组件的情况。[locomotor.lua:681](../../../raw/current/scripts/components/locomotor.lua#L681 [player_classified.lua:655](../../../raw/current/scripts/prefabs/player_classified.lua#L655 [player_classified.lua:1623](../../../raw/current/scripts/prefabs/player_classified.lua#L1623 [strafer.lua:24](../../../raw/current/scripts/components/strafer.lua#L24

本地收到开始事件，有缓存 controller 才开启 aiming／更新；非主模拟端只要当前有 locomotor，还会 SetStrafing(true)。结束事件停止输入更新、清 lastdir，客户端把现有 locomotor:SetStrafing(false)。重复事件不会叠加组件更新任务；组件本身没有公开的 Start／Stop 方法。[strafer.lua:1](../../../raw/current/scripts/components/strafer.lua#L1

### 输入方向、预测与权威接收

- **手柄**：读取四个 VIRTUAL_CONTROL_STRAFE 轴；任一轴绝对值达到 CONTROLLER_DEADZONE_RADIUS 才根据摄像机 RightVec／DownVec 换算世界方向。它不是沿人物当前朝向直接解释摇杆。[strafer.lua:58](../../../raw/current/scripts/components/strafer.lua#L58
- **鼠标**：用 GetWorldXZWithHeight(1) 取得点，计算人物朝向该点的角度。无有效点／无有效手柄方向时不更新朝向，也不发新方向。[strafer.lua:68](../../../raw/current/scripts/components/strafer.lua#L68
- **应用**：有 locomotor 则立即 OnStrafeFacingChanged(dir)；非主模拟且方向不同于 lastdir 时发送 RPC.StrafeFacing。本地主机直接改自己的 locomotor，不经这次 RPC。[strafer.lua:74](../../../raw/current/scripts/components/strafer.lua#L74

RPC 接收端检查 dir 为合法 number、玩家有 locomotor 且 SG 不 busy，再调用 OnStrafeFacingChanged；后者仅在 strafedir 已设置且显示朝向真的改变时生效。它转人物 Transform，并按保留的移动方向重算相对 MotorVel；SetMoveDir 在 strafing 时维护的是移动方向，不直接用它覆盖瞄准朝向。实际移动速度和移动意图仍由 locomotor 其他入口负责。[networkclientrpc.lua:480](../../../raw/current/scripts/networkclientrpc.lua#L480 [locomotor.lua:697](../../../raw/current/scripts/components/locomotor.lua#L697 [locomotor.lua:721](../../../raw/current/scripts/components/locomotor.lua#L721

关闭客户端预测时可能没有本地 locomotor，本组件仍采集与发送方向；重新启用预测创建 locomotor 时，player_common 会按 classified.isstrafing 设置 strafing 状态。不能把“没有本地 locomotor”解释为本地不再瞄准。[player_common.lua:877](../../../raw/current/scripts/prefabs/player_common.lua#L877 [strafer.lua:74](../../../raw/current/scripts/components/strafer.lua#L74

## 典型接入

### 原版物品持续施法：状态由施法回调驱动

player_common 给 channelcaster 配 OnStartChannelCastingItem／OnStopChannelCastingItem。开始回调在物品明确关闭 channelcastable.strafing 时跳过，否则在调整相关移动倍率后 locomotor:StartStrafing；结束回调恢复倍率并 StopStrafing。这说明 strafer 承接的是宿主已启用的独立瞄准流程，不是负责开始施法、减速或检查物品资源的组件。[player_common.lua:300](../../../raw/current/scripts/prefabs/player_common.lua#L300 [player_common.lua:2855](../../../raw/current/scripts/prefabs/player_common.lua#L2855

Input 的虚拟控制解析还读取 strafer:IsAiming：例如经典控制方案中，瞄准时避免库存导航抢占方向控制；法术轮盘打开时又可整体阻断虚拟方向键。故 HUD 模式放行 OnUpdate，也不保证每种输入一定提供有效方向。[input.lua:335](../../../raw/current/scripts/input.lua#L335 [input.lua:351](../../../raw/current/scripts/input.lua#L351

## 常见陷阱与限制

- **安装后不响应 startstrafing**：若构造时 controller 不存在，缓存仍为 nil，开始事件不会进入 aiming；应沿活动玩家的安装顺序，不通过反复发事件补救。[strafer.lua:1](../../../raw/current/scripts/components/strafer.lua#L1 [player_common.lua:1019](../../../raw/current/scripts/prefabs/player_common.lua#L1019
- **删除本地组件等于服务端停止 strafing**：移除钩子只撤事件监听，并清客户端当前 locomotor 的 strafing；宿主 RemoveComponent 负责停组件更新。它不调用服务端 StopStrafing 或清 classified 净变量。[strafer.lua:43](../../../raw/current/scripts/components/strafer.lua#L43 [entityscript.lua:670](../../../raw/current/scripts/entityscript.lua#L670
- **aiming／lastdir 跨存档保留**：本组件没有 OnSave／OnLoad，缓存与输入状态由活动玩家安装时重建，并读取当时 classified 状态；不是持久化瞄准方向记录。[strafer.lua:24](../../../raw/current/scripts/components/strafer.lua#L24

## 验证记录

- 源码审阅：2026-10-26，Steam buildid 26148036；覆盖组件全文、活动玩家安装／移除、持续施法启停、controller 门禁、locomotor 朝向／运动方向分离、classified 事件、StrafeFacing RPC、预测重建与输入优先级示例。未展开所有输入方案、持续施法内部、完整网络验证或引擎运动预测。
- 结构检查：validate_page 及源码引用 strict 检查通过；无新增完整 Lua 示例。机械检查不代替游戏运行。
- 游戏运行：未运行；本地主机／远程客户端、预测切换、手柄死区、HUD／busy 阻挡及卸载重建待实测。

相关：[[reference/components/locomotor]]。
