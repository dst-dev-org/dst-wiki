---
title: "boatai"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "原型期船用 AI 组件：每帧扫描附近升帆的船并模仿其航向，原版无任何挂载点，属于未接入的死代码。"
aliases: ["船 AI", "boatai", "船只跟随原型"]
tags: [component, ai]
size_class: simple
dst_build: "steam-buildid:26230496"
sources: ["components/boatai.lua", "components/mast.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# boatai

## 快速判断

- 职责与适用：构造即 `StartUpdatingComponent` 开启每帧更新；OnUpdate 半径 200 扫描实体，发现升帆的船时把己方帆升起并把桅杆风向改为朝目标船下风处。[OnUpdate](../../../raw/current/scripts/components/boatai.lua#L8-L23)
- 优先使用：无原版使用场景。全库 grep 确认 `boatai` 仅出现在本组件文件内，没有任何 `AddComponent("boatai")`、prefab 或 brain 引用——原版船只 AI 不经此组件。把它当"参考实现"读，不要当可用功能挂。
- 重要副作用：每帧一次 `TheSim:FindEntities(..., 200)` 无任何 tag 过滤，是全实体扫描，开销显著；且直接改写己方 `hull.mast` 的 `wind_direction`。[扫描](../../../raw/current/scripts/components/boatai.lua#L11)
- 修改边界：硬依赖 `self.inst.components.hull.mast.components.mast` 链——宿主必须有 hull 且已装桅杆，否则命中目标时直接报错。[硬依赖](../../../raw/current/scripts/components/boatai.lua#L16-L20)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| 构造 | `Class(function(self, inst))` | 立即 `StartUpdatingComponent(self)`，无配置项、无开关。[实现](../../../raw/current/scripts/components/boatai.lua#L1-L5) |
| `OnUpdate(dt)` | `dt: number -> nil` | 只处理 `v.components.mast ~= nil and mast.is_sail_raised` 的实体；对每个目标重复改己方风向，多目标时以后处理的为准。[实现](../../../raw/current/scripts/components/boatai.lua#L13-L22) |
| 依赖字段 `mast.is_sail_raised` | `boolean`（mast 组件） | 目标筛选条件；mast 组件默认 false，升帆时置位。[字段](../../../raw/current/scripts/components/mast.lua#L21) |

## 常见陷阱与限制

- **以为它是可用的跟随/护航 AI：** 逻辑只改风向朝目标船的"下风 10 单位"方向，不处理靠岸、碰撞、掉队，也没有停止条件；实现停留在原型阶段，原版从未接线。[完整逻辑](../../../raw/current/scripts/components/boatai.lua#L8-L23)
- **在无桅杆船上挂载：** OnUpdate 命中升帆目标时访问 `components.hull.mast.components.mast`，`hull.mast` 为 nil 即抛错；挂载前必须保证 hull 与 mast 都就位。
- **误当成性能安全的被动组件：** 构造就开更新循环，不存在"挂上但不生效"的状态；不想要扫描开销就必须 RemoveComponent 或 StopUpdatingComponent。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（25 行）与 mast.lua 相关字段；全库 grep `boatai`（含大小写不敏感）确认无原版消费者、无 prefab/brain/行动引用。
- 结构检查：2026-10-28 validate_page.py 通过（本页）；validate_source_links.py --strict 通过。
- 游戏运行：未运行；组件行为仅在源码层核验，未在游戏内复现。

相关：[[reference/components/hull]]、[[reference/components/mast]]
