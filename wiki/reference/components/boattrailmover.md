---
title: "boattrailmover"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "船尾水花 FX 的直线漂移驱动：按行进方向、初速与减速度逐帧移动特效实体，由 boattrail 生成时装配。"
aliases: ["boattrailmover", "船尾水花移动器", "航迹特效漂移"]
tags: [component, callback]
size_class: simple
dst_build: "steam-buildid:26230496"
sources: ["components/boattrailmover.lua", "components/boattrail.lua", "prefabs/boat_water_fx.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# boattrailmover

## 快速判断

- 职责与适用：纯客户端可见的 FX 位移组件，挂在水花特效实体上；`Setup` 记录方向／初速／加速度后启动逐帧更新，`OnUpdate` 让特效沿行进方向漂移并按加速度减速。[Setup](../../../raw/current/scripts/components/boattrailmover.lua#L7-L18)
- 优先使用：船航行尾迹由 boattrail 组件生成 `boat_water_fx` 并调用 `Setup(dir_x, dir_z, 0.5, -0.125)`；复用现成 FX 时直接沿用这组参数。[调用方](../../../raw/current/scripts/components/boattrail.lua#L39-L41)
- 重要副作用：`Setup` 会按方向取反算舵角并设置实体朝向（`-rudder_angle + 90`），随后 StartUpdatingComponent；不手动调用 StopUpdating，停止依赖宿主删除。[朝向](../../../raw/current/scripts/components/boattrailmover.lua#L10-L17)
- 修改边界：宿主 `boat_water_fx` 播完动画或 80 帧后自我移除，组件随之消亡；不写存档、无网络变量。[宿主自清](../../../raw/current/scripts/prefabs/boat_water_fx.lua#L27-L29)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `Setup(dir_x, dir_z, velocity, acceleration)` | `-> 无` | 方向取反存储；`velocity`、`acceleration` 原样记录；设置朝向后启动更新。原版参数 `(dir_x, dir_z, 0.5, -0.125)`，负加速度即逐渐减速。[实现](../../../raw/current/scripts/components/boattrailmover.lua#L7-L18) |
| `OnUpdate(dt)` | `-> 无` | `velocity = velocity + dt * acceleration`，再按 `dir * velocity * dt` 累加位移；组件不自行停止。[实现](../../../raw/current/scripts/components/boattrailmover.lua#L20-L30) |
| `track_boat_time` | `number`（默认 0.4） | 构造赋初值、每帧递减，但全库无读取者——疑似遗留的死字段，不要依赖其语义。[定义](../../../raw/current/scripts/components/boattrailmover.lua#L4) |

## 典型接入

### 复用船尾水花做自定义航迹（原版参照）

- 前提与选择：服务端生成 `boat_water_fx`（FX 本体含组件），需要漂移时传行进方向；不漂移的静态水花不调 Setup 即可。[FX 定义](../../../raw/current/scripts/prefabs/boat_water_fx.lua#L27-L29)
- 生效与撤销：Setup 即刻开始移动；FX 由 animover／80 帧任务自删，无需手动清理。

```lua
-- 服务端：生成水花并让它沿行进方向漂移（原版参数）
local fx = SpawnPrefab("boat_water_fx")
fx.Transform:SetPosition(x, y, z)
fx.components.boattrailmover:Setup(dir_x, dir_z, 0.5, -0.125)
```

- 调整与验证：`velocity` 单位约等于每秒位移量、`acceleration` 为每秒增量；想要加速尾迹可传正加速度，但需自行确认 FX 生命周期足够长。

## 常见陷阱与限制

- **以为方向参数是"特效朝向"：** `Setup` 存的是取反后的方向并据此转实体角度，FX 视觉朝向与位移方向由同一组值推导；自行另设 Transform 旋转会被 Setup 覆盖。[朝向计算](../../../raw/current/scripts/components/boattrailmover.lua#L10-L15)
- **依赖 track_boat_time 判断寿命：** 该字段只减不读，组件不会因它停止；FX 的实际寿命由宿主 prefab 的动画与定时器决定。[宿主自清](../../../raw/current/scripts/prefabs/boat_water_fx.lua#L29-L30)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（31 行），核对 boattrail 生成调用、boat_water_fx 挂载与自清路径，grep 确认 `track_boat_time` 无读取者。
- 结构检查：2026-10-28 validate_page.py 通过；validate_source_links.py --strict 锚点通过（见批级记录）。
- 游戏运行：未运行。

## 相关链接

- [[reference/components/boattrail]]：按移动距离生成船尾水花 FX 的来源组件。
