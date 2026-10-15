---
title: "boattrail"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "按移动距离在船尾生成 boat_water_fx 水花并交给 boattrailmover 漂移，休眠自动停更；客户端可见 FX 组件。"
aliases: ["船尾水花", "航行尾迹", "boattrail"]
tags: [component, callback]
dst_build: "steam-buildid:26230496"
sources: ["components/boattrail.lua", "prefabs/boat.lua", "prefabs/wave.lua", "prefabs/yotd_boats.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# boattrail

## 快速判断

- 职责与适用：宿主（船、浪）移动时每隔 `effect_spawn_rate` 距离在行进反方向生成一个 `boat_water_fx`；纯表现层，非专用服务器才安装（[boat 安装守卫](../../../raw/current/scripts/prefabs/boat.lua#L436-L448)）。
- 优先使用：船类 prefab 在 `not TheNet:IsDedicated()` 分支内 AddComponent 即可；FX 漂移由 [[reference/components/boattrailmover]] 承担。
- 重要副作用：监听 `entitysleep`／`entitywake` 自动停更/续更，唤醒时重置累计距离与上次坐标（[监听](../../../raw/current/scripts/components/boattrail.lua#L25-L26)）。
- 修改边界：不写存档、无网络变量；特效生命期由 boat_water_fx 自身管理，组件不回收已生成的水花。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `effect_spawn_rate` | `number`，默认 `1` | 每移动多少单位距离生成一个水花；浪类宿主调成 2.5。[字段](../../../raw/current/scripts/components/boattrail.lua#L20) [浪配置](../../../raw/current/scripts/prefabs/wave.lua#L124) |
| `radius`／`scale_x`／`scale_z` | `number`，默认 `3`／`1`／`1` | 水花相对行进方向的后置偏移与 xy 缩放；冰船按船半径比例放大。[字段](../../../raw/current/scripts/components/boattrail.lua#L21-L23) [冰船调整](../../../raw/current/scripts/prefabs/boat.lua#L1329-L1335) |
| `SpawnEffectPrefab(x, y, z, dir_x, dir_z)` | `number` ×5 | 生成水花、随机三选一动画并调 `boattrailmover:Setup(dir, 0, 0.5, -0.125)`。[实现](../../../raw/current/scripts/components/boattrail.lua#L30-L42) |
| `OnUpdate(dt)` | 组件更新循环 | 累计位移（单帧超过 rate 按上限计），超过阈值即生成水花并扣减；记录最后位置与方向。[实现](../../../raw/current/scripts/components/boattrail.lua#L44-L68) |

## 典型接入

### 自定义船体加尾迹

- 前提与选择：船 prefab 的 common 侧（非专用服务器分支）安装；需要与 boatringdata 等常规船组件并存，无顺序要求。
- 生效与撤销：唤醒后开始累计移动；宿主休眠自动停更，组件随宿主移除；水花本身独立消亡。
- 原版依据：主船（[boat](../../../raw/current/scripts/prefabs/boat.lua#L447)）、浪（[wave](../../../raw/current/scripts/prefabs/wave.lua#L123-L127)）、周年活动船（[yotd_boats](../../../raw/current/scripts/prefabs/yotd_boats.lua#L195)）。

```lua
-- 自定义船 prefab 的 common 侧（非专用服务器分支）
if not TheNet:IsDedicated() then
    inst:AddComponent("boattrail")
    inst.components.boattrail.effect_spawn_rate = 1.5
    inst.components.boattrail.radius = 2.5
end
```

- 调整与验证：radius 单位与世界坐标一致；scale_x/scale_z 直接传给 FX Transform 缩放。船速极快时单帧最多按 rate 记 1 个水花（min 截断），不会爆量。

## 常见陷阱与限制

- boat.lua 冰船调整里写了 `boattrail.scale_y = scale`（[boat](../../../raw/current/scripts/prefabs/boat.lua#L1333)），但组件只读 scale_x/scale_z，`scale_y` 是无效字段；接入方不要效仿。
- 组件在 `OnUpdate` 首帧才初始化 last_x/last_z（[实现](../../../raw/current/scripts/components/boattrail.lua#L47-L51)），唤醒后第一帧不生成水花，属正常行为。

## 相关链接

- [[reference/components/boattrailmover]]：水花漂移与减速。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，覆盖全部接口与 boat／wave／yotd_boats 消费链；未运行游戏。
- 结构检查：validate_page.py、validate_source_links.py 与 Lua 语法探针通过。
