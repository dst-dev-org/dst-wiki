---
title: "病害蝇群表现（diseaseflies）"
created: "2026-11-02"
updated: "2026-11-02"
type: prefab
summary: "病害组件发病时按邻近发病实体概率生成的蝇群：联网代理同步循环次数，各非专用端创建本地声画。"
aliases: ["diseaseflies", "病害蝇群", "病害苍蝇特效"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/diseaseflies.lua", "components/diseaseable.lua", "prefabs/grass.lua", "prefabs/grassgekko.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 病害蝇群表现（diseaseflies）

## 快速判断

- `diseaseflies` 是独立工厂注册的蝇群声画，疾病状态、传播和重排由 [[reference/components/diseaseable|diseaseable 组件]] 处理；它与通用表的 `disease_puff` 是不同注册和工厂。[工厂](../../../raw/current/scripts/prefabs/diseaseflies.lua#L44-L77)、[[reference/prefabs/bee_poof_big#关联机制导航|病害表现辨析]]
- 在所选 scripts 中，组件 `DoFX` 有实际生成代码，`grass.lua` 的数组只是依赖声明；全树检索未发现原版 `AddComponent("diseaseable")` 挂载调用，因此不能把这条生成代码声称为当前原版世界中已观察到的活跃流程。外部模组仍可使用组件。[生成](../../../raw/current/scripts/components/diseaseable.lua#L117-L138)、[声明](../../../raw/current/scripts/prefabs/grass.lua#L17-L24)
- 标题与别名是根据工厂和用途核验的中文效果描述，非官方译名；简中 `languages/chinese_s.po` 定向检索 `STRINGS.NAMES.DISEASEFLIES` 未命中。

## 注册与构造

| 注册名／对象 | 构造及端 | 差异与证据 |
|---|---|---|
| `diseaseflies` 网络代理 | 加 Transform、Network、`FX`／`NOCLICK`，`net_tinybyte` 循环次数初始 5；非专用端排 0 秒任务创建本地表现，主模拟设不存档并在 2 秒后移除代理，`SetLoops` 只在主模拟导出。 | [代理](../../../raw/current/scripts/prefabs/diseaseflies.lua#L40-L74)、[注册](../../../raw/current/scripts/prefabs/diseaseflies.lua#L77-L77) |
| 本地蝇群 | 独立非联网实体加 Transform、AnimState、SoundEmitter、`FX`／`NOCLICK`，禁休眠且不存档；代理有父级时本地实体挂同一父级，否则创建时从代理取变换。 | `flies` bank／build 播 `swarm_pre`，按同步值压入对应次数的 `swarm_loop`，再播 `swarm_pst`；层偏移 1，循环声音 `dontstarve/common/flies` 音量 0.5；`animqueueover` 移除本地实体。[构造与清理](../../../raw/current/scripts/prefabs/diseaseflies.lua#L6-L38) |

工厂声明 `anim/flies.zip`；未解包确认素材帧数或视觉内容。[资源](../../../raw/current/scripts/prefabs/diseaseflies.lua#L1-L4)

## 关联机制导航

| 对象／调用 | 条件与作用 | 证据 |
|---|---|---|
| diseaseable: `Disease` → `DoFX` | 首次发病时重置旧任务、设发病状态并以 `math.random()` 秒延迟执行 `DoFX`；组件与宿主负责疾病状态及传播。 | [发病](../../../raw/current/scripts/components/diseaseable.lua#L154-L166)、[[reference/components/diseaseable#关键接口]] |
| `DoFX` 概率与选名 | 以宿主为中心检索 8 半径 `diseased` 实体，除自身外仅计有组件且 `_lastfx < GetTime()` 的实体；以 `1/num` 概率生成 `diseaseflies` 并父挂宿主，随机传 3～7 次循环。每次执行后按 `loops * 100 * FRAMES + 5 + math.random() * 3` 重排；未生成时 loops 为 0。 | [计数、生成和重排](../../../raw/current/scripts/components/diseaseable.lua#L117-L138) |
| 组件移除 | `OnRemoveFromEntity` 取消待执行的 FX 与传播、延迟和预警任务；已经创建的代理、本地动画按各自实体生命周期处理。 | [取消任务](../../../raw/current/scripts/components/diseaseable.lua#L83-L102)、[工厂](../../../raw/current/scripts/prefabs/diseaseflies.lua#L27-L38) |
| 资源／依赖 | grass 依赖列表含本注册名；grassgekko 的病害资源声明保留且注释说明供恢复病害的模组使用，均不等于添加组件。 | [依赖](../../../raw/current/scripts/prefabs/grass.lua#L17-L24)、[资源注释](../../../raw/current/scripts/prefabs/grassgekko.lua#L1-L7) |

## 常见陷阱与限制

- 代理的 2 秒移除与本地实体的 `animqueueover` 是两条不同生命周期，不能推断本地蝇群必在 2 秒结束；动画时长以及网络延迟、晚加入的实际表现未在游戏核验。
- `SetLoops` 修改代理的同步值；本地表现创建时读取该值并压入动画队列。脚本未提供在本地队列生成后重新构建队列的逻辑，不能保证事后修改立即重排已有表现。[读取](../../../raw/current/scripts/prefabs/diseaseflies.lua#L27-L33)、[设置](../../../raw/current/scripts/prefabs/diseaseflies.lua#L40-L42)

## 验证记录

- 源码审阅：2026-11-02，Steam buildid 26335394；核对 `diseaseflies` 注册、工厂、同步字段、端分界、挂接与清理；对 `raw/current/scripts` 全树检索注册名、`diseaseflies` 前缀和 `AddComponent("diseaseable")`，区分组件生成与依赖声明。未验证资源内容、模组动态挂载和运行时网络表现。
- 游戏运行：未运行。

返回 [[reference/prefabs/fx-world-index|特效 Prefab 索引·环境与世界]]。
