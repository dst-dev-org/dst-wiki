---
title: "ocean"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "洋流方向原型组件：持有一个随机角度与速度并可输出方向向量，原版无任何挂载点，属于未接入的死代码。"
aliases: ["洋流原型", "ocean 组件", "水流方向"]
tags: [component, mechanism]
size_class: simple
dst_build: "steam-buildid:26230496"
sources: ["components/ocean.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# ocean

## 快速判断

- 职责与适用：仅在实体上维护 `currentAngle`（构造时随机取 45°×0..7 + 22.5°，即 8 个 45° 扇区的中值）与 `currentSpeed`（恒 1），并能把两者转成方向向量；`OnUpdate` 为空函数。[字段](../../../raw/current/scripts/components/ocean.lua#L6-L11)
- 优先使用：没有。全库 grep 确认既无 `AddComponent("ocean")`，也无任何 `components.ocean` 读取者；与海洋相关的实际机制由 oceanfishable、oceanfishingrod、boatphysics 等组件承担。此组件是洋流方向的原型残留，仅作参考阅读。
- 重要副作用：构造不调用 `StartUpdatingComponent`，即使挂载也不会有任何每帧行为；`set_ocean_angle` 只在构造时执行一次。[构造](../../../raw/current/scripts/components/ocean.lua#L6-L11)
- 修改边界：无存读、无标签、无事件、无网络变量；数据完全私有，改角度/速度需直接改字段（原版连 setter 都未提供）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `currentAngle` | `number`（度，构造时随机 22.5+45k） | 由 `set_ocean_angle` 在构造时赋值；`GetCurrentAngle` 读取。[随机规则](../../../raw/current/scripts/components/ocean.lua#L2-L4) |
| `currentSpeed` | `number`（默认 1） | `GetCurrentSpeed` 读取；无任何写入点。[实现](../../../raw/current/scripts/components/ocean.lua#L20-L22) |
| `GetCurrentVec3()` | `-> x, 0, z` | `speed * cos(angle*DEGREES), 0, speed * sin(angle*DEGREES)`，角度按度转弧度。[实现](../../../raw/current/scripts/components/ocean.lua#L24-L26) |
| `OnUpdate(dt)` | `dt: number -> nil` | 空实现；且构造未启动更新，不外力调用即永不执行。[实现](../../../raw/current/scripts/components/ocean.lua#L13-L14) |

## 常见陷阱与限制

- **与海洋 prefab/海洋组件族混淆：** 名为 `ocean` 的组件与 `ocean` 地块、`oceanfishable`/`oceantrawler` 等组件无任何关联；引用时注意 `components.ocean` 在原版从未被使用过。想实现洋流需自行启动更新并补全逻辑。
- **期待角度持续变化：** `set_ocean_angle` 仅构造时调用一次；`OnUpdate` 为空，即使 `StartUpdatingComponent` 也不会改角度，"随机洋流"的动态部分从未实现。[空更新](../../../raw/current/scripts/components/ocean.lua#L13-L14)
- **角度单位：** `currentAngle` 是角度制，`GetCurrentVec3` 内部才乘 `DEGREES` 转弧度；直接拿 `currentAngle` 参与 `sin/cos` 会错一个单位。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（28 行）；全库 grep（`AddComponent("ocean")`、`components.ocean` 词边界）确认无消费者；核对 `ocean*` 组件族命名以区分。
- 结构检查：2026-10-28 validate_page.py 通过（本页）；validate_source_links.py --strict 通过。
- 游戏运行：未运行；组件从未被原版挂载，无可运行场景。
