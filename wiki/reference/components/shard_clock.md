---
title: "shard_clock"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "多分片世界的时间同步桥：主分片把 clock 组件状态写入网络变量，次分片收包后转推 secondary_clockupdate 喂给本分片 clock。"
aliases: ["分片时钟同步", "shard_clock", "跨分片时间"]
tags: [component, network-sync]
dst_build: "steam-buildid:26230496"
size_class: simple
sources: ["components/shard_clock.lua", "components/clock.lua", "prefabs/shard_network.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# shard_clock

## 快速判断

- 职责与适用：只在挂于 `TheWorld.shard` 实体上时工作，把主分片的昼夜／月相状态跨分片同步给次分片；无任何公开方法，纯事件＋网络变量桥。[初始化](../../../raw/current/scripts/components/shard_clock.lua#L105-L111)
- 优先使用：模组不应直接调它；改时间仍走本分片 `clock` 的 `ms_*` 事件，次分片读时间读本分片 `clock`（它消费同步来的数据）。[clock 侧消费](../../../raw/current/scripts/components/clock.lua#L307)
- 重要副作用：维护 7 组网络变量（3 段 segs、cycles、phase、moonphase、mooniswaxing、total/remaining time），值变化触发 `clockdirty`；纯 remaining 时间微调用 `set_local` 不触发脏事件。[变量声明](../../../raw/current/scripts/components/shard_clock.lua#L27-L36)、[L77-L81](../../../raw/current/scripts/components/shard_clock.lua#L77-L81)
- 修改边界：构造即断言 `TheWorld.ismastersim`，客户端不存在；不存档（宿主实体 `persists = false`），状态每 tick 由主分片重推。[断言](../../../raw/current/scripts/components/shard_clock.lua#L7)、[宿主](../../../raw/current/scripts/prefabs/shard_network.lua#L25-L32)

## 关键接口

| 接口／字段 | 已核验类型／单位 | 条件、作用与证据 |
|---|---|---|
| `_segs[1..3]` | net_smallbyte ×3 | 昼夜相位内的 seg 进度；`NUM_PHASES = 3`，注释要求与 clock.lua `PHASE_NAMES` 保持同步。[L13](../../../raw/current/scripts/components/shard_clock.lua#L13)、[L28-L30](../../../raw/current/scripts/components/shard_clock.lua#L28-L30) |
| `_cycles`／`_phase`／`_moonphase`／`_mooniswaxing` | net_ushortint／net_tinybyte ×2／net_bool | 天数、昼夜相位、月相及盈亏；任一 `set` 触发 `clockdirty`。[L31-L34](../../../raw/current/scripts/components/shard_clock.lua#L31-L34) |
| `_totaltimeinphase`／`_remainingtimeinphase` | net_float ×2 | 相位总时长与剩余秒；仅 remaining 会在"其余全无变化"时走 `set_local` 静默更新。[L35-L36](../../../raw/current/scripts/components/shard_clock.lua#L35-L36)、[L77-L81](../../../raw/current/scripts/components/shard_clock.lua#L77-L81) |
| `OnClockUpdate`（内部） | 事件处理 | 主分片：收 `master_clockupdate`（clock.lua 每 tick 推，[L438](../../../raw/current/scripts/components/clock.lua#L438)）逐项比对后写网络变量。[L42-L82](../../../raw/current/scripts/components/shard_clock.lua#L42-L82) |
| `OnClockDirty`（内部） | 事件处理 | 次分片：收 `clockdirty` 组装 data 表并 `TheWorld:PushEvent("secondary_clockupdate", data)`。[L84-L99](../../../raw/current/scripts/components/shard_clock.lua#L84-L99) |

## 数据流

- 主分片：`clock` 组件 OnUpdate 组装 `{segs, cycles, moonphase, mooniswaxing, phase, totaltimeinphase, remainingtimeinphase}` 推 `master_clockupdate` → 本组件写网络变量。[数据组装](../../../raw/current/scripts/components/clock.lua#L425-L438)
- 次分片：网络变量同步到次分片后触发 `clockdirty` → 本组件推 `secondary_clockupdate` → 次分片 `clock` 监听它并据此驱动本分片昼夜。[监听](../../../raw/current/scripts/components/shard_clock.lua#L110)、[clock 接收](../../../raw/current/scripts/components/clock.lua#L307)
- 注册在初始化按 `_ismastershard` 二选一完成，生命周期与 shard 实体一致，无清理钩子。[L105-L111](../../../raw/current/scripts/components/shard_clock.lua#L105-L111)

## 常见陷阱与限制

- 只桥接不裁判：两个分片各有一份 `clock`，修改时间必须发 `ms_setphase` 等 `ms_*` 事件让主分片 `clock` 生效，直接改 shard_clock 的网络变量会被下一 tick 主分片数据覆盖。[clock 事件注册](../../../raw/current/scripts/components/clock.lua#L296-L305)
- `NUM_PHASES = 3` 是硬编码常量，与 clock.lua 的 `PHASE_NAMES` 数量人工同步；改相位数量需两处同时改。[L13](../../../raw/current/scripts/components/shard_clock.lua#L13)
- 组件整页无 OnSave/OnLoad/OnRemoveFromEntity；宿主 shard 实体 `persists = false`，重启后由主分片重新推送，不需要也不应给它补存读。[宿主](../../../raw/current/scripts/prefabs/shard_network.lua#L25-L32)

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。覆盖文件全文（变量声明、两个事件处理分支与初始化）及 clock.lua 的推收两端、shard_network 挂载段；未覆盖网络层底层同步机制与专用服务器表现。
- 游戏运行：未运行；本页结论均来自源码静态核验。
- 结构检查：`validate_page.py`、`validate_source_links.py --strict` 通过（见批记录）。

相关：[[reference/components/sandstormwatcher]]。
