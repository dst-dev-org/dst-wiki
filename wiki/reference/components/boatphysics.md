---
title: "boatphysics"
created: "2026-10-21"
updated: "2026-10-28"
type: component
summary: "服务端船体运动：把帆、桨、锚、磁力与碰撞合成为速度、舵向和每帧物理位移。"
aliases: ["船的推进与转向", "船速与舵向", "锚与桅杆增删", "boatphysics"]
tags: [component, lifecycle, state-machine, server-client, network-sync]
dst_build: "steam-buildid:26230496"
sources: ["components/boatphysics.lua", "components/boatdrag.lua", "components/anchor.lua", "components/mast.lua", "components/boatmagnet.lua", "components/hullhealth.lua", "components/walkableplatform.lua", "components/steeringwheeluser.lua", "components/oar.lua", "prefabs/boat.lua", "prefabs/yotd_boats.lua", "prefabs/boatrace_seastack.lua", "prefabs/anchor.lua", "prefabs/mast.lua", "prefabs/seastack.lua", "stategraphs/SGboat.lua", "tuning.lua", "modutil.lua", "mainfunctions.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# boatphysics

## 快速判断

- 职责与适用：服务端船体运动组件，维护世界坐标速度 `velocity_x/velocity_z`、当前／目标舵向，把帆力、划桨冲量、磁力、锚阻力与碰撞反弹合成后每帧写入 `Physics:SetMotorVel`；作用对象是已有 Physics 圆柱的水上平台；原版船形宿主（`boat`／`dragonboat_*`）另配 walkableplatform，`boatrace_seastack*` 这类浮标宿主则不需要。[构造](../../../raw/current/scripts/components/boatphysics.lua#L148-L187)、[OnUpdate](../../../raw/current/scripts/components/boatphysics.lua#L506-L638)
- 优先使用：不要直接改速度。帆挂 `AddMast`（对方提供 `CalcSailForce`/`CalcMaxVelocity`），划桨用 `ApplyRowForce`，锚／绞盘用 `AddBoatDrag` 并由 `boatdrag` 字段描述阻力，一次性外力用 `ApplyForce`，急停用 `AddEmergencyBrakeSource`。[mast:SetBoat](../../../raw/current/scripts/components/mast.lua#L108-L128)、[oar:Row](../../../raw/current/scripts/components/oar.lua#L11-L33)、[boatdrag](../../../raw/current/scripts/components/boatdrag.lua#L1-L14)
- 重要副作用：构造即 `StartUpdating` 并注册永久碰撞回调；`stopturning` 在容差内每帧都发；碰撞给对方推 `on_collide`／`hit_boat`；速度超过 1 时加 `scarytoprey` tag；服务端还会回写 `doplatformcamerazoom` 网络布尔。[构造](../../../raw/current/scripts/components/boatphysics.lua#L179-L187)、[:540-543](../../../raw/current/scripts/components/boatphysics.lua#L540-L543)、[:577-620](../../../raw/current/scripts/components/boatphysics.lua#L577-L620)
- 修改边界：只在服务端存在（三个宿主都在 `ismastersim` 之后添加）；客户端拿不到该组件，只能看到物理复制、tag 与 `doplatformcamerazoom`；运行期不要移除它（见陷阱）。

## 关键接口

| 接口／字段 | 已核验类型／单位 | 条件、作用与证据 |
|---|---|---|
| `velocity_x`／`velocity_z` | 世界坐标速度，单位/秒 | 所有推力的累加点；`DoApplyForce` 统一 clamp 到 `MAX_ALLOWED_VELOCITY`=10。[:299-308](../../../raw/current/scripts/components/boatphysics.lua#L299-L308)、[:453-470](../../../raw/current/scripts/components/boatphysics.lua#L453-L470) |
| `max_velocity` | 无量纲倍率，默认 1 | 只是 `GetMaxVelocity` 的乘数之一，不是速度上限；多个碰撞 prefab 反用它当伤害分母。[:154](../../../raw/current/scripts/components/boatphysics.lua#L154)、[:344-384](../../../raw/current/scripts/components/boatphysics.lua#L344-L384) |
| `turn_acc` | 弧度/秒²，默认 `PI` | 舵向角速度每帧按它逼近目标速度。[:171-172](../../../raw/current/scripts/components/boatphysics.lua#L171-L172)、[:524-529](../../../raw/current/scripts/components/boatphysics.lua#L524-L529) |
| `GetMaxVelocity()` | 速度上限 | 帆、磁铁各自的最大值升序排列后依次按 1／0.7／0.49… 加权叠加，再乘 `max_velocity` 与各 `boatdrag.max_velocity_mod`，最后 clamp 到 10。[:344-384](../../../raw/current/scripts/components/boatphysics.lua#L344-L384) |
| `GetRudderTurnSpeed()` | 弧度/秒 | 按当前速度分档硬编码 0.1975／0.255／0.37／0.48／0.6；舵手带 `master_crewman` 时 ×1.5。[:416-441](../../../raw/current/scripts/components/boatphysics.lua#L416-L441) |
| `SetTargetRudderDirection(x,z)`／`GetTargetRudderDirection()` | 单位方向向量 | 只设目标，由 OnUpdate 逐帧逼近；驾驶者组件在转向输入时调用。[:244-255](../../../raw/current/scripts/components/boatphysics.lua#L244-L255)、[SteerInDir](../../../raw/current/scripts/components/steeringwheeluser.lua#L87-L99) |
| `AddMast(mast)`／`RemoveMast(mast)` | 以实体为 key 的集合 | 需要 `CalcSailForce`、`CalcMaxVelocity`、`SetRudderDirection`、`CloseSail`；`mast:SetBoat` 自动增删。[:257-263](../../../raw/current/scripts/components/boatphysics.lua#L257-L263) |
| `AddBoatDrag(inst)`／`RemoveBoatDrag(inst)` | 需要 `components.boatdrag` | 读 `drag`／`max_velocity_mod`／`forcedampening`／`sailforcemodifier`，并监听该实体的 `onremove`／`death`／`onburnt` 自动移除。[:222-242](../../../raw/current/scripts/components/boatphysics.lua#L222-L242) |
| `AddMagnet(magnet)`／`RemoveMagnet(magnet)` | 需要 `PairedBeacon`、`CalcMagnetForce` | 与拖拽同构的集合；boatmagnet 自动增删。[:265-271](../../../raw/current/scripts/components/boatphysics.lua#L265-L271)、[boatmagnet:SetBoat](../../../raw/current/scripts/components/boatmagnet.lua#L84-L100) |
| `ApplyRowForce(x,z,force,max_velocity)` | 冲量（一次速度增量） | 划桨入口；沿速度方向的增益随速度衰减，达到 `max_velocity` 后该方向推进归零，侧向分量仍按基础阻尼生效。[:310-338](../../../raw/current/scripts/components/boatphysics.lua#L310-L338) |
| `ApplyForce(x,z,force)` | 冲量 | 乘 `GetForceDampening()` 后叠加；总速度仍受 10 上限约束。[:289-297](../../../raw/current/scripts/components/boatphysics.lua#L289-L297)、[:340-342](../../../raw/current/scripts/components/boatphysics.lua#L340-L342) |
| `GetBoatDrag`／`GetTotalAnchorDrag`／`ApplyDrag` | 减速度（单位/秒²） | 基础 `inCubic(0.2→1.5, 上限 10)` 加锚的 `outExpo(累计 drag)`；ApplyDrag 把结果乘 dt。[:386-406](../../../raw/current/scripts/components/boatphysics.lua#L386-L406)、[:453-470](../../../raw/current/scripts/components/boatphysics.lua#L453-L470) |
| `AddEmergencyBrakeSource(source)`／`RemoveEmergencyBrakeSource` | 字符串来源 key | 置 `halting`、`Physics:SetMass(0)`；来源集合清空才恢复 `TUNING.BOAT.MASS`。[:640-652](../../../raw/current/scripts/components/boatphysics.lua#L640-L652) |
| `SetHalting(halt)` | 布尔 | 供 walkableplatform 用：关所有帆并加来源 `boatinbadstate`；玩法层不要直接调用。[:654-664](../../../raw/current/scripts/components/boatphysics.lua#L654-L664)、[walkableplatform](../../../raw/current/scripts/components/walkableplatform.lua#L186-L192) |
| `OnSave()`／`OnLoad(data)` | 存档表 | 只存目标舵向两分量、`boat_rotation_offset`、速度两分量；读档把当前舵向也设成目标。[:189-212](../../../raw/current/scripts/components/boatphysics.lua#L189-L212) |

### 状态与几何

- 舵向是单位向量：`rudder_direction_*` 为当前，`target_rudder_direction_*` 为目标；夹角小于 PI/32 时停止转动，`turn_vel` 按 `turn_acc` 加速或减速；两轴差都小于 0.1 时另发 `stopturning`。[OnUpdate](../../../raw/current/scripts/components/boatphysics.lua#L506-L543)
- `velocity_x/z` 按世界方向累加，写入物理前用 `Transform:GetRotation()` 旋转成 motor vel；直接推速度要用世界方向，不是船的本地朝向。[:631-635](../../../raw/current/scripts/components/boatphysics.lua#L631-L635)
- `boat_rotation_offset` 只在 `SetCanSteeringRotate(true)` 时记录；本快照没有原版调用者，`steering_rotate` 恒为 false，此时 OnUpdate 转的是各桅杆模型而不是船体。[SetCanSteeringRotate](../../../raw/current/scripts/components/boatphysics.lua#L443-L451)、[:622-629](../../../raw/current/scripts/components/boatphysics.lua#L622-L629)
- `was_moving`、`has_speed`、`turn_vel` 是运行期缓存，不存档；`has_speed` 对应 `scarytoprey` tag 的增删。

## 关键流程

### 每帧合成顺序

转舵 → 汇总帆力（乘锚的 `sailforcemodifier`）→ 磁力 → 阻力与帆力（有锚且有帆时先帆后阻力，否则先阻力后帆）→ 起停事件与 `scarytoprey` → 1 秒节流的镜头缩放 → 舵向写回桅杆 → 旋转成 motor vel 并设 `boat_movement` 音效速度。[OnUpdate](../../../raw/current/scripts/components/boatphysics.lua#L506-L638)

### 碰撞与沉没

- 只处理 `other == TheWorld`、带 `BLOCKER` tag 或带 boatphysics 的对象；同一 GUID 4 tick 内去重；法线 x/z 都为 0 时判为无效碰撞，直接 `health:Kill()`（无 health 则 `Remove()`），以防 NaN 存档。[OnCollide](../../../raw/current/scripts/components/boatphysics.lua#L7-L47)
- 普通碰撞按相对速度与法线夹角算反弹并加到自身速度；对方收到 `on_collide` 与 `hit_boat`，本船只在对方不是船时收到自己的 `on_collide`，船对船由对方回调补上。伤害与漏水由 hullhealth 消费，不在本组件。[碰撞派发](../../../raw/current/scripts/components/boatphysics.lua#L77-L133)、[hullhealth](../../../raw/current/scripts/components/hullhealth.lua#L3-L8)
- 沉没不由本组件驱动：health 死亡后本组件只 KillSound；SGboat 走 `ready_to_snap`→`snapping`（onpresink）→`popping`（sinkloot 后 Remove）。walkableplatform 发现船压在陆地视觉格上会直接 Kill health。[SGboat](../../../raw/current/scripts/stategraphs/SGboat.lua#L37-L132)

## 典型接入

### 案例一：在既有船上调推进与转向（服务端 postinit）

- 前提与选择：目标文件是模组 `modmain.lua`；`AddPrefabPostInit` 的函数在 prefab 工厂函数（含 master postinit）之后、`entity_spawned` 之前执行，此时 boatphysics 已存在。[modutil.lua:592-598](../../../raw/current/scripts/modutil.lua#L592-L598)、[mainfunctions.lua:387-411](../../../raw/current/scripts/mainfunctions.lua#L387-L411)
- 端与资源：只在服务端生效，客户端实体上 `inst.components.boatphysics` 为 nil；不需要新资源与动画。
- 生效与撤销：影响之后生成的每条 `boat`，不改已存在实例；撤销即删除该 postinit 并重载模组／重启世界。
- 调整与验证：`max_velocity` 是倍率，`turn_acc` 是弧度/秒²；提高 `max_velocity` 会同时缩小部分碰撞 prefab 的伤害（它们用它当分母）。`rudder_turn_speed` 无效，见陷阱。原版推进链参照：帆 `mast:SetBoat`→`AddMast`、锚 `anchor:SetIsAnchorLowered`→`AddBoatDrag`、桨 `Oar:Row`→`ApplyRowForce`。[anchor](../../../raw/current/scripts/components/anchor.lua#L145-L157)

```lua
-- modmain.lua
AddPrefabPostInit("boat", function(inst)
    if not TheWorld.ismastersim then
        return
    end
    local boatphysics = inst.components.boatphysics
    if boatphysics == nil then
        return
    end
    boatphysics.max_velocity = 1.2
    boatphysics.turn_acc = PI / 2
end)
```

### 案例二：不带 walkableplatform 的浮动障碍物（boatrace_seastack 参照）

- 前提与选择：`boatrace_seastack` 与 `boatrace_seastack_monkey` 加 waterphysics（`restitution` 1.0+`BOATRACE_SEASTACK_EXTRA_RESTITUTION`=1.5）、boatphysics、health、combat、workable、inspectable（前者另有 lootdropper），没有 walkableplatform、hullhealth 或 healthsyncer；说明 boatphysics 可以当"会被撞动的浮标"而不必是船。[boatrace_seastack](../../../raw/current/scripts/prefabs/boatrace_seastack.lua#L85-L163)
- 必要宿主条件：Physics 圆柱 + `COLLISION.OBSTACLES` 组 + 含 `WORLD`／`OBSTACLES` 的 mask；被撞方自己会收到 `on_collide`（对方非船时本组件直接推，船对船由对方回调补发），宿主仍需自己监听消费；没有 SGboat 时要自己处理死亡移除，因为不存在沉没流程。
- 生效与撤销：反弹由 boatphysics 负责，伤害／掉落由宿主自己的 `on_collide` 处理器负责；撤销时移除宿主实体，不要单独 `RemoveComponent("boatphysics")`。
- 端：均在 `ismastersim` 之后添加，客户端只有物理复制。

## 常见陷阱与限制

- 调 `rudder_turn_speed` 无效：构造时赋 `TUNING.BOAT.RUDDER_TURN_SPEED`，但 `GetRudderTurnSpeed` 用按速度分档的硬编码值，全脚本没有其它读取点。[字段](../../../raw/current/scripts/components/boatphysics.lua#L155)、[分档](../../../raw/current/scripts/components/boatphysics.lua#L416-L441)
- `max_velocity` 不是「最高速度」：它只是 `GetMaxVelocity` 的倍率，而 seastack／oceantree／waterplant／wobster_den 等把它当伤害分母，改它会连带改碰撞伤害。[boatphysics](../../../raw/current/scripts/components/boatphysics.lua#L377)、[seastack](../../../raw/current/scripts/prefabs/seastack.lua#L82-L90)
- 运行期移除组件会留下悬空回调：没有 `OnRemoveFromEntity`，构造时的 `Physics:SetCollisionCallback(OnCollide)` 与 `ListenForEvent("death", on_death)` 都不撤销；此后任何一次碰撞会访问 `nil._recent_collisions`，死亡会调用 `nil:OnDeath()`。[构造](../../../raw/current/scripts/components/boatphysics.lua#L181-L186)、[:26-31](../../../raw/current/scripts/components/boatphysics.lua#L26-L31)、[:144-146](../../../raw/current/scripts/components/boatphysics.lua#L144-L146)
- `stopturning` 不是边沿事件：两轴差都小于 0.1 时每帧都发，监听方要自己幂等。[boatphysics](../../../raw/current/scripts/components/boatphysics.lua#L540-L543)
- 垂直碰撞会直接毁船：法线 x/z 均为 0 被当作无效碰撞，Kill 或 Remove 并打印一行日志；这是防 NaN 存档的保底，不是可配置行为。[boatphysics](../../../raw/current/scripts/components/boatphysics.lua#L14-L24)
- 宿主需要补 `sounds.damage`：只要 `inst.sounds` 非空，碰撞就访问 `inst.sounds.damage`；`doplatformcamerazoom` 则是可选，缺失时只跳过镜头缩放。[boatphysics](../../../raw/current/scripts/components/boatphysics.lua#L126-L129)、[:607-620](../../../raw/current/scripts/components/boatphysics.lua#L607-L620)
- 存档不覆盖附属件：帆／锚／磁铁不在 OnSave 内，靠各自实体存档；`was_moving`、`has_speed`、`turn_vel` 不存，读档首帧前依赖它们的消费者（如 anchor 初始化）会读到 nil。[boatphysics](../../../raw/current/scripts/components/boatphysics.lua#L189-L212)、[anchor](../../../raw/current/scripts/components/anchor.lua#L40)

## 验证记录

- 版本增量复核 26148036（2026-10-25）：锚点重定位 1 处（mainfunctions SpawnPrefabFromSim 段 +26）；无结论修订，仅锚点迁移——boatphysics 及全部协作组件／prefab／SG 本版未变。依据：全局枢纽 diff 审阅（mainfunctions R-09 改动在错误界面／落点保护／世界标签）＋变更清单命中 tuning.lua、mainfunctions.lua（tuning 本版 BOAT 参数未变且本页无行锚点）。
- 源码审阅：2026-10-21，steam-buildid:25643519。覆盖构造、OnUpdate、OnCollide、OnSave／OnLoad、增删与推力接口，三个真实宿主的 AddComponent 位置与注册名（`boat`、`dragonboat_body`／`dragonboat_shadowboat`、`boatrace_seastack`／`boatrace_seastack_monkey`），以及与 hullhealth、walkableplatform、anchor、mast、oar、boatdrag、boatmagnet 的调用边界；未逐方法翻译，未覆盖渲染／音效资源层。
- 游戏运行：未运行（本页结论均来自源码静态核验，不含手感实测）。
- 结构检查：`validate_page.py` 通过（0 issue，行数与字节均在 200 行／16 KiB 上限内）；`validate_source_links.py --strict` 通过（57 处源码引用、51 个目标、0 error／0 warning）；正文 modmain 示例另存 296 字节，`parser_probe --syntax-only` 通过。

## 相关链接

- [[reference/components/waterphysics]]：船对船反弹推力系数。
- [[reference/components/boatdrifter]]：船漂移状态机。
- [[reference/components/anchor]]：船锚状态机与 boatdrag 联动。
