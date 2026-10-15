---
title: "beaverness"
created: "2026-10-28"
updated: "2026-10-29"
type: component
summary: "伍迪海狸值遗留组件：饥饿式消耗条 + 变身事件；当前版本全库无安装点，已被 wereness 取代。"
aliases: ["海狸值", "伍迪变身条（旧）", "beaverness"]
tags: [component]
dst_build: "steam-buildid:26230496"
sources: ["components/beaverness.lua", "prefabs/woodie.lua", "prefabs/player_classified.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# beaverness

## 快速判断

- 职责与适用：设计上是伍迪的海狸值计量条（消耗→0 触发 startstarving 并按饥饿速率扣血，[扣血](../../../raw/current/scripts/components/beaverness.lua#L32-L38)），但当前快照全库 grep 无任何 `AddComponent("beaverness")` 或字段读取者——是未被使用的遗留代码。
- 优先使用：实现伍迪式变身条请改用 `wereness` 组件（伍迪实装于 [woodie.lua](../../../raw/current/scripts/prefabs/woodie.lua#L1822)），不要挂本组件。
- 重要副作用：若被安装，`DoDelta` 会推 `beavernessdelta`、`startstarving`/`stopstarving` 事件（[推送](../../../raw/current/scripts/components/beaverness.lua#L58-L77)）。
- 修改边界：`current` setter 会写 `player_classified.currentbeaverness`，而该 netvar 在 player_classified.lua 中并未定义——真实玩家上触发断言/空索引错误（[写点](../../../raw/current/scripts/components/beaverness.lua#L1-L6)），进一步证明其不可直接启用。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `max` / `current` | `number`，默认均 100 | 有 player_classified 时 `max` 被 `makereadonly` 锁定（[构造](../../../raw/current/scripts/components/beaverness.lua#L14-L21)）。 |
| `DoDelta(delta, overtime)` | `(number, boolean)` | 钳 0~max 并推 `beavernessdelta`（oldpercent/newpercent/overtime）；跨 0 边界推 start/stopstarving（[实现](../../../raw/current/scripts/components/beaverness.lua#L58-L77)）。 |
| `StartTimeEffect(dt, delta_b)` | `(number, number)` | 周期任务按 `time_effect_multiplier` 消耗；归零时按 `hunger.hurtrate` 扣血（[实现](../../../raw/current/scripts/components/beaverness.lua#L40-L45)）。 |
| `StopTimeEffect()` / `SetTimeEffectMultiplier(m)` | 无参 / `number` | 停任务；倍率默认 1（[实现](../../../raw/current/scripts/components/beaverness.lua#L47-L56)）。 |
| `GetPercent()` / `SetPercent(percent, overtime)` | → `number` / `(number, boolean)` | `current/max` 读写（[实现](../../../raw/current/scripts/components/beaverness.lua#L79-L86)）。 |
| `OnSave` / `OnLoad` | `current` 字段 | 持久化当前值（[存读](../../../raw/current/scripts/components/beaverness.lua#L88-L100)）。 |

## 典型接入

### 无原版消费者，仅作考古参照

- 全库 grep 结论（2026-10-28，steam-buildid:26230496）：`beaverness` 仅在组件文件自身出现；`beavernessdelta` 事件、`beaverness` 标签均无外部监听/读取者；`currentbeaverness` netvar 全库也只此一处引用。
- 伍迪现行变身实现使用 `wereness` 组件；`currentwereness` 是 `player_classified` 上的 net_byte 网络变量，不是独立实体，客户端通过它读取变身值比例。[组件挂载](../../../raw/current/scripts/prefabs/woodie.lua#L1822)、[网络变量定义](../../../raw/current/scripts/prefabs/player_classified.lua#L1393-L1397)、[比例读取](../../../raw/current/scripts/prefabs/woodie.lua#L583-L591)
- 若确需复用其"消耗→饥饿扣血"模式，应复制改造为自定义组件并自建网络同步，不直接 AddComponent。

## 常见陷阱与限制

- 直接 `AddComponent("beaverness")` 到玩家会埋雷：`current` 一旦变化即触发对未定义 netvar `player_classified.currentbeaverness` 的 `:set` 调用报错（[写点](../../../raw/current/scripts/components/beaverness.lua#L1-L6)）；非玩家实体则无此问题但事件仍无消费者。
- `DoDelta` 的 start/stopstarving 推送被 `hunger:IsStarving()` 门控（[门控](../../../raw/current/scripts/components/beaverness.lua#L66)），宿主必须同时有 hunger 组件，否则直接报错。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，覆盖组件全文；对组件名、标签、事件名、netvar 名做全库 grep 确认无消费者；比对 woodie.lua 的 wereness 实装；未运行游戏。
- 术语补核：2026-10-29，同一 build，定向核验 currentwereness 的 net_byte 定义与伍迪比例读取，纠正字段／同步实体混称；未扩大为整页重审。
- 结构检查：validate_page.py、validate_source_links.py 与 Lua 语法探针通过。
