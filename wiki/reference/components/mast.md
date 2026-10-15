---
title: "mast（桅杆）"
created: 2026-10-21
updated: "2026-10-28"
type: component
summary: "船用桅杆：绑定船只、向 boatphysics 注册帆力与最大速度，并管理收帆/升帆的连续状态"
aliases: ["桅杆", "帆力", "收帆", "升帆", "mast"]
tags: [component, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/mast.lua", "components/boatphysics.lua", "prefabs/mast.lua", "tuning.lua", "actions.lua", "componentactions.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# mast（桅杆）

## 快速判断

- 职责与适用：船用桅杆组件。它把桅杆实体登记到所在船的 `boatphysics.masts`，向船提供帆力 `sail_force` 与速度上限 `max_velocity`；帆的收/放是带进度的连续过程，而不是布尔开关。[构造](../../../raw/current/scripts/components/mast.lua#L19-L45)
- 链上位置：船速链的“帆”端——`mast:CalcSailForce()` 与 `mast:CalcMaxVelocity()` 由 `boatphysics:OnUpdate` 汇总，再交给 `ApplySailForce`/`GetMaxVelocity`；“舵”端转向由 steeringwheel/boatphysics 负责，“阻力”端由 boatdrag 负责，本页不重复。
- 优先使用：自定义桅杆只需在服务端给实体 `AddComponent("mast")`，随后直接改 `sail_force`、`max_velocity` 或调用 `SetSailForce`；不要改 `boatphysics` 的汇总逻辑。[帆力设置](../../../raw/current/scripts/components/mast.lua#L82-L84)
- 重要副作用：构造时立即 `SpawnPrefab("rudder")` 并挂为子实体，下一帧才用 `GetCurrentPlatform()` 绑定船；绑定成功会注册进 `boatphysics:AddMast`、移除 `rotatableobject` 标签；船 `onremove`/`death` 会自动解绑。[绑定](../../../raw/current/scripts/components/mast.lua#L108-L128)
- 修改边界：仅服务端有逻辑；实体必须位于船上（平台为空时 `SetBoat(nil)`）。移除组件前应显式 `SetBoat(nil)`，因为 `OnRemoveFromEntity` 只取消建船任务，不做解绑。[移除回调](../../../raw/current/scripts/components/mast.lua#L52-L57)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SetBoat(boat)` | `boat` 为船实体或 `nil` | 先 `boatphysics:RemoveMast(self)` 并移除旧船的 `onremove`/`death` 监听，再登记新船、把桅杆转到船朝向、加/移 `rotatableobject`；重复传同一船直接返回。[L108-L128](../../../raw/current/scripts/components/mast.lua#L108-L128) |
| `sail_force` / `SetSailForce(set)` | 数值，默认 `TUNING.BOAT.MAST.BASIC.SAIL_FORCE` = 0.6 | 只保存数值，实际使用在 `CalcSailForce`；马瘤鲣桅杆在 prefab 中直接写 1.3。[默认值](../../../raw/current/scripts/components/mast.lua#L21-L29) |
| `max_velocity` | 数值，默认 2.5 | 桅杆自身的速度上限；马瘤鲣为 4。与 `boatphysics.max_velocity`（全局倍率）相乘后还受 `MAX_ALLOWED_VELOCITY` 限制。[TUNING](../../../raw/current/scripts/tuning.lua#L4245-L4258) |
| `CalcSailForce()` | 返回数值 | 普通模式且已升帆：`sail_force * (1 - furlunits/furlunits_max)`；反向部署（`inverted`）在未升帆或过渡中按 `furlunits` 比例给力；其余为 0。[L86-L93](../../../raw/current/scripts/components/mast.lua#L86-L93) |
| `CalcMaxVelocity()` | 返回数值 | 与帆力同一比例，乘 `max_velocity`；`boatphysics:GetMaxVelocity` 会把多条桅杆升序排序后按 1、0.7、0.49… 递减权重相加（先加较小值）。[L95-L102](../../../raw/current/scripts/components/mast.lua#L95-L102) |
| `AddSailFurler(doer, strength)` / `RemoveSailFurler(doer)` | `strength` 为每秒收帆单位 | 登记拉帆者并置 `is_sail_transitioning`，从而启用 `OnUpdate`；`furlunits_max = 4.5`、无人时 `autounfurlunits = 8` 每秒回落。[L149-L161](../../../raw/current/scripts/components/mast.lua#L149-L161) |
| `OnUpdate(dt)` | 每帧，仅在过渡时 | 有人收帆则累加 `dt * 拉帆强度`，达到 95% 调 `SailFurled`；无人则按 8/s 回落，到 0 调 `SailUnfurled`；同时刷新 `open_pst` 动画百分比。[L234-L259](../../../raw/current/scripts/components/mast.lua#L234-L259) |
| `UnfurlSail()` / `SailFurled()` / `SailUnfurled()` / `CloseSail()` | 无参 | `UnfurlSail` 播 `knot_release` 进入过渡；`SailFurled` 把 `furlunits` 置满、`is_sail_raised=false`；`SailUnfurled` 反之；`CloseSail` 按 `inverted` 选一个终态。[L173-L223](../../../raw/current/scripts/components/mast.lua#L173-L223) |
| `SetReveseDeploy(set)`（原文拼写） | 布尔 | 反向部署：`furlunits=0`、`is_sail_raised=set`、`inverted=set`；马瘤鲣桅杆用它。[L76-L80](../../../raw/current/scripts/components/mast.lua#L76-L80) |
| `SetRudderDirection(x, z)` | 世界坐标方向分量 | 让桅杆 `FacePoint` 到该方向；`boatphysics` 每帧对每条桅杆调用，船体转向也由同一组 `rudder_direction_*` 驱动。[L229-L232](../../../raw/current/scripts/components/mast.lua#L229-L232) |

## 典型接入

### 原版桅杆宿主（mast / mast_malbatross / mast_yotd）

- 前提与选择：目标是 `scripts/prefabs/mast.lua`。服务端 `fn_pst` 在 `SetPristine` 之后执行 `AddComponent("mast")`；普通、马瘤鲣、YOTD 三个 prefab 共用这段接入。[AddComponent](../../../raw/current/scripts/prefabs/mast.lua#L371-L402)
- 加载与端：prefab 文件由游戏 prefab 加载器加载；`fn_pre` 建 Transform/AnimState/Light/SoundEmitter/Network，`fn_pst` 仅在 `TheWorld.ismastersim` 为真时执行，客户端只拿到表现实体。[主机注册](../../../raw/current/scripts/prefabs/mast.lua#L404-L431)
- 注册名与资源：`mast`（船用普通桅杆）、`mast_malbatross`、`mast_yotd` 分别注册；资源为对应 `anim/*.zip` 与迷你地图图标，物品形态是 `mast_item` 等部署套件。[注册](../../../raw/current/scripts/prefabs/mast.lua#L613-L623)
- 单位与差异：普通/`mast_yotd` 用 `sail_force=0.6`、`max_velocity=2.5`；`mast_malbatross` 在 `fn_pst` 后覆盖为 `SAIL_FORCE=1.3`、`MAX_VELOCITY=4`，并调用 `SetReveseDeploy(true)`，因此其帆力方向与普通桅杆相反。[马瘤鲣覆盖](../../../raw/current/scripts/prefabs/mast.lua#L456-L462)
- 生效与撤销：桅杆部署到船上后，构造函数下一帧自动绑定；玩家通过 `RAISE_SAIL`/`LOWER_SAIL_BOOST` 推动收放，动作是否出现由 `sailraised`/`saillowered`/`sail_transitioning` 标签决定。[动作条件](../../../raw/current/scripts/componentactions.lua#L590-L604)
- 拆除：锤子拆掉前若 `mast.boat ~= nil` 会 `SetBoat(nil)`，再 `Remove()`；船死亡时 `OnDeath` 置 `boat_death` 并解绑，实体移除时按有无船播放沉没/坍塌特效。[拆解](../../../raw/current/scripts/prefabs/mast.lua#L73-L85) [死亡](../../../raw/current/scripts/components/mast.lua#L141-L147)

服务端最小挂接骨架（放在已 `SetPristine` 的自定义 prefab `fn_pst` 内；不是完整 prefab，部署/物理/动画需另备）：

```lua
inst:AddComponent("mast")
inst.components.mast.sail_force = 0.6
inst.components.mast.max_velocity = 2.5
```

- 调整与验证：字段是最终值而非倍率；两条桅杆并非简单相加（升序后按 0.7 递减权重），改 `max_velocity` 才能改变该桅杆贡献的上限。[汇总](../../../raw/current/scripts/components/boatphysics.lua#L344-L384) 本页源码审阅范围见“验证记录”，未做游戏内运行。

## 常见陷阱与限制

- `SetVelocityMod(set)` 只写 `self.max_velocity_mod`，快照内没有任何消费者读取桅杆的该字段；船速倍率由 `boatdrag.max_velocity_mod` 提供。改桅杆限速请用 `max_velocity`。[写入点](../../../raw/current/scripts/components/mast.lua#L104-L106)
- 状态命名与注释容易反读：`SailFurled` 的注释写着 “sail is raised”，实际把 `is_sail_raised` 置 `false`、`furlunits` 置满（收帆）；`UnfurlSail` 注释是 “lowering sail”。判断应以 `CalcSailForce()==0` 和 `sailraised`/`saillowered` 标签为准，不要按函数名推断。[状态](../../../raw/current/scripts/components/mast.lua#L173-L214)
- 绑定只在构造后一帧发生：实体构造时不在船上（陆地、Limbo、刚生成未落平台）会得到 `boat=nil`，之后即使搬上船也不会自动重绑；需要显式 `SetBoat(boat)`。构造后 `boatphysics:AddMast` 是否被调用可直接从 `boatphysics.masts` 核对。[延迟绑定](../../../raw/current/scripts/components/mast.lua#L41-L44)
- `RemoveComponent("mast")` 走 `OnRemoveFromEntity`，只取消 `_setup_boat_task`，不会从 `boatphysics.masts` 移除，也不会移除船的 `onremove`/`death` 监听；应先 `SetBoat(nil)` 再移除组件。[移除回调](../../../raw/current/scripts/components/mast.lua#L52-L57)
- 构造无条件 `SpawnPrefab("rudder")` 并 `SetRudder` 挂到实体；`SetRudder` 每次调用都会往 `highlightchildren` 追加而不清理旧舵，重复调用会残留子实体。把 mast 组件挂到非桅杆实体时也会多出一个舵。[构造](../../../raw/current/scripts/components/mast.lua#L39 [SetRudder](../../../raw/current/scripts/components/mast.lua#L130-L139)
- 收帆进度不是线性时间：`furlunits_max=4.5`、`autounfurlunits=8`，且 `GetCurrentFurlUnits` 只统计正在播放 `pull_small_loop` 或处于 `pull_big_loop` 前 8 帧的拉帆者；换动画会暂停进度。[进度计算](../../../raw/current/scripts/components/mast.lua#L163-L171)

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 锚点重定位 1 处（tuning.lua BOAT.MAST 数值块）；结论修订：无，仅锚点迁移。依据：枢纽 diff 审阅＋变更清单命中 componentactions.lua、tuning.lua（命中区域逐行核对均未漂移）。
- 源码审阅：2026-10-21，`steam-buildid:25643519`；完整阅读 `components/mast.lua`，并核验 `components/boatphysics.lua` 的 `AddMast`/`GetMaxVelocity`/`ApplySailForce`/转向段、`prefabs/mast.lua` 三个宿主与注册、`tuning.lua` 的 `BOAT.MAST`、`actions.lua` 与 `componentactions.lua` 的升/收帆入口。覆盖构造/绑定/帆力/限速/收帆/清理；未逐条审阅灯笼与避雷针升级分支。
- 游戏运行：未运行；本页所有结论为源码审阅，未验证多人延迟、动画事件时序或实际航速手感。
- 结构检查：`validate_page.py` 通过（0 问题）；`validate_source_links.py --strict` 通过：本页 29 个 Lua 引用，0 error / 0 warning；本页 Lua 片段经 `parser_probe.exe --syntax-only` 通过（0 错误）。

相关页面：[[reference/components/boatphysics]]、[[reference/components/steeringwheel]]、[[reference/components/boatdrag]]。
