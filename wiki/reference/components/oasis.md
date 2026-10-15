---
title: "oasis"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "绿洲范围标记组件：按半径判定实体是否在绿洲内，并给出进入程度的 0~1 接近度，供沙尘暴强度计算消减。"
aliases: ["绿洲范围", "oasis", "绿洲接近度"]
tags: [component, mechanism]
size_class: simple
dst_build: "steam-buildid:26230496"
sources: ["components/oasis.lua", "prefabs/oasislake.lua", "components/sandstorms.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# oasis

## 快速判断

- 职责与适用：挂在世界生成的水塘实体（oasislake）上，作为"绿洲"几何标记；消费方是 sandstorms 组件的沙尘暴强度计算——绿洲内无沙暴、边缘渐弱。[职责](../../../raw/current/scripts/components/oasis.lua#L1-L8)
- 优先使用：需要"某区域免受沙尘暴"时参照原版，把实体注册为绿洲：设置 `radius` 并 `TheWorld:PushEvent("ms_registeroasis", inst)`。[原版挂载](../../../raw/current/scripts/prefabs/oasislake.lua#L254-L255)
- 重要副作用：组件无存读、无事件监听，纯被动查询；注册通过 `ms_registeroasis` 事件进入 sandstorms 的 `_oases` 表。[注册监听](../../../raw/current/scripts/components/sandstorms.lua#L69)
- 修改边界：不修改天气本身；radius 默认 1，需显式覆写为 `TUNING.SANDSTORM_OASIS_RADIUS`（=1）才与原版一致。[取值](../../../raw/current/scripts/tuning.lua#L3329-L3330)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `radius` | `number`（默认 1） | 判定圆半径；原版挂载后立即覆写为 `TUNING.SANDSTORM_OASIS_RADIUS`。[赋值](../../../raw/current/scripts/prefabs/oasislake.lua#L255) |
| `IsEntityInOasis(ent)` | `-> bool` | `ent:IsNear(self.inst, self.radius)`，圆内即真。[实现](../../../raw/current/scripts/components/oasis.lua#L6-L8) |
| `GetProximityLevel(ent, range)` | `-> number [0,1]` | 圆内返回 1；圆外 `range` 内线性衰减到 0，超出为 0。原版 `range = TUNING.SANDSTORM_FULLY_ENTERED_DEPTH`（=20）。[实现](../../../raw/current/scripts/components/oasis.lua#L13-L22) |

## 典型接入

### 复刻"安全区免疫沙尘暴"的查询（原版参照）

- 前提与选择：服务端；宿主需是可被 `IsNear`/距离查询的实体。sandstorms 每次计算遍历已注册绿洲，取最大接近度。[CalcOasisLevel](../../../raw/current/scripts/components/sandstorms.lua#L114-L121)
- 生效与撤销：注册后实体 `onremove` 时 sandstorms 自动从 `_oases` 移除；组件本身无注销接口，撤销依赖宿主实体被移除。[注册/注销](../../../raw/current/scripts/components/sandstorms.lua#L55-L59)
- 原版依据：oasislake 在 master postinit 挂组件、设半径并注册；sandstorms 的 `IsInOasis` 用 `IsEntityInOasis` 做布尔判定。[IsInOasis](../../../raw/current/scripts/components/sandstorms.lua#L104-L109)

```lua
-- 服务端：把自定义实体注册为绿洲（等价原版 oasislake 做法）
inst:AddComponent("oasis")
inst.components.oasis.radius = TUNING.SANDSTORM_OASIS_RADIUS
TheWorld:PushEvent("ms_registeroasis", inst)
```

- 调整与验证：radius 单位与 `IsNear` 相同（世界单位）；range 参数只影响圆外衰减宽度，不影响圆内判定。

## 常见陷阱与限制

- **忘记推送 ms_registeroasis：** 只挂组件不注册，sandstorms 的 `_oases` 表为空，接近度永远为 0，沙暴照常。[注册链](../../../raw/current/scripts/components/sandstorms.lua#L55-L59)
- **以为组件驱动天气：** 组件只提供几何查询；真正减沙暴的是 sandstorms 消费 `GetProximityLevel` 的结果，单独调用组件不会改变任何天气状态。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（24 行），核对 oasislake.lua 挂载与注册、sandstorms.lua 两处消费、tuning.lua 两个常量；grep 确认消费者仅 oasislake 与 sandstorms。
- 结构检查：2026-10-28 validate_page.py 通过（本页）；validate_source_links.py --strict 通过；Lua 示例经 parser_probe --syntax-only 解析通过。
- 游戏运行：未运行；注册自定义绿洲对沙暴的实际消减未实测。
