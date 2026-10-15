---
title: "shatterfx"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "按 levels 配置播放冻结破碎特效的分级动画，由 freezable 生成的 shatter 特效实体驱动，服务端经 net 变量下发等级。"
aliases: ["shatterfx", "碎冰特效层级", "冻结破碎特效", "shatter 等级"]
tags: [component, network-sync]
size_class: simple
dst_build: "steam-buildid:26230496"
sources: ["components/shatterfx.lua", "prefabs/shatter.lua", "components/freezable.lua", "standardcomponents.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# shatterfx

## 快速判断

- 职责与适用：特效实体（`shatter`）上的动画分级驱动器：宿主写入 `levels` 表后，`SetLevel(n)` 播放对应分级动画；等级随冻结程度由 freezable 决定。[实现](../../../raw/current/scripts/components/shatterfx.lua#L7-L18)
- 优先使用：不直接挂到角色上；角色经 freezable 的 `AddShatterFX("shatter", offset, sym)` 声明特效，解冻时 SpawnShatterFX 生成实体并对它 SetLevel。[生成链](../../../raw/current/scripts/components/freezable.lua#L96-L113)
- 重要副作用：shatter prefab 在服务端覆写组件的 SetLevel 改为设置 net 变量 `_level`，客户端监听 leveldirty 后用组件原版 SetLevel 播动画；两端同名组件、行为分叉。[服务端覆写](../../../raw/current/scripts/prefabs/shatter.lua#L81-L85)
- 修改边界：`persists = false`、`inst.persists` 不入存档；FX 实体播完 animover 自删，服务端 1 秒兜底移除。[生命周期](../../../raw/current/scripts/prefabs/shatter.lua#L87-L88)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `levels` | `{ {anim=...[, pre=...]}, ... }` | 宿主必须先注入分级表（shatter 用 tiny..huge 五级）；SetLevel 只在 `levels[level]` 存在且与当前等级不同时切换。[注入](../../../raw/current/scripts/prefabs/shatter.lua#L6-L13) [消费](../../../raw/current/scripts/components/shatterfx.lua#L8-L9) |
| `SetLevel(level)` | `level: number` -> `nil` | 先 `math.min(level, #levels)` 夹取；首次且该级有 `pre` 时先播 pre 再 Push 循环 anim，否则直接 `PlayAnimation(anim, true)` 循环。[实现](../../../raw/current/scripts/components/shatterfx.lua#L7-L18) |
| `level` | `number|nil` | 当前已播放等级；nil 表示尚未播过（决定 pre 分支）。[实现](../../../raw/current/scripts/components/shatterfx.lua#L3-L4) |

## 典型接入

### 复刻 shatter 的服务端等级同步（原版参照）

- 前提与选择：shatter prefab 是双端实体：客户端在 `leveldirty` 里延迟一帧生成代理 FX，把 `levels` 注入组件后 `SetLevel(proxy._level:value())`；服务端挂组件后用闭包覆写 SetLevel 直接 `inst._level:set(level)`。[客户端](../../../raw/current/scripts/prefabs/shatter.lua#L36-L55) [服务端](../../../raw/current/scripts/prefabs/shatter.lua#L81-L85)
- 生效与撤销：freezable 在 `Unfreeze` 等时机 SpawnShatterFX，按 `SetShatterFXLevel` 预设的 `fxlevel` 调 SetLevel；小型/微型冻僵角色经 standardcomponents 的 Make*FreezableCharacter 预设 1～2 级。[触发](../../../raw/current/scripts/components/freezable.lua#L288-L291) [预设](../../../raw/current/scripts/standardcomponents.lua#L328-L333)
- 原版依据：等级数值语义由生成方约定（freezable 的 fxlevel），组件只负责"等级→动画"映射与 pre 段播放。

## 常见陷阱与限制

- **以为 pinnable 的碎裂也走本组件：** pinnable 有同名 `SpawnShatterFX(ratio)` 方法，但实现是按剩余比例从 splash 列表直接 SpawnPrefab，不经过 shatterfx 组件；两者只是命名相近。[辨析](../../../raw/current/scripts/components/pinnable.lua#L81-L89)
- **没注入 levels 就 SetLevel：** `math.min(level, #self.levels)` 对空表得 0，`levels[0]` 为 nil，组件静默不播；首次分支还要求 `levels[level].pre` 判空，动画名错误同样无表现。[实现](../../../raw/current/scripts/components/shatterfx.lua#L8-L15)
- **在服务端期待本地播放动画：** 服务端实例的 SetLevel 已被 shatter prefab 覆写为写 net 变量，不会操作 AnimState；动画播放只发生在客户端代理 FX 上。[覆写](../../../raw/current/scripts/prefabs/shatter.lua#L83-L85)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（20 行）与 shatter prefab，核对 freezable 的 AddShatterFX/SetShatterFXLevel/SpawnShatterFX 链路、Unfreeze 触发点、standardcomponents 预设及 pinnable 同名方法辨析。未审阅 freezable 冻结状态机全貌（见 [[reference/components/freezable]]）。
- 结构检查：2026-10-28 validate_page.py 通过（shatterfx.md）；validate_source_links.py --strict 本页全部源码锚点通过；本页无完整 Lua 示例。
- 游戏运行：未运行；分级动画与网络同步未实测。
