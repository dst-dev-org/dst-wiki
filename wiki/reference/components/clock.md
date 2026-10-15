---
title: "clock"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "世界时钟组件：以 16 段昼夜相位驱动 day/dusk/night 与月相循环，net 变量全网同步并按 tick 推 clocksegs/cycles/phase/clocktick 等事件。"
aliases: ["世界时钟", "昼夜相位", "clock 组件"]
tags: [component, network-sync, lifecycle]
dst_build: "steam-buildid:26230496"
size_class: standard
sources: ["components/clock.lua", "prefabs/world_network.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# clock

## 快速判断

- 职责与适用：挂在 world_network 实体上（prefabs/world_network.lua:57），主 sim 推进相位并同步，客户端本地插值、段末等待服务端校正（:317-321 注释）。
- 优先使用：读状态走 `TheWorld.state`（由本组件 net 变量支撑）；改相位/段配置 → 世界事件 `ms_setphase`/`ms_nextphase`/`ms_nextcycle`/`ms_setclocksegs`（:296-299）；改月相 → `ms_setmoonphase`/`ms_lockmoonphase`/`ms_setmoonphasestyle`（:300-302）。
- 重要副作用：每 tick 推 `clocktick`（含 phase/timeinphase/time，:412-422）；相位段配置变化推 `clocksegschanged`，新周期推 `ms_cyclecomplete` 与 `cycleschanged`（:351-355、:390-393）。
- 修改边界：总段数恒 16（`NUM_SEGS`，:11、:189 断言）；段数调整由 seasons 组件负责（:355 注释），本组件只执行换相位。

## 关键接口

| 接口/字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `GetTimeUntilPhase` | `(phase) -> number` | 沿相位段累加剩余时间到目标相位；同相位返回 0（:100-122）。 |
| `AddMoonPhaseStyle` | `(style) -> nil` | 追加月相外观名（default/alter_active/glassed_*，:54-61）；uiclock 的 `_moon_builds` 需同步（:53 注释）。 |
| net 变量组 | `_segs[1..3]` net_smallbyte、`_cycles` net_ushortint、`_phase`/`_moonphase`/`_moonphasestyle` net_tinybyte、`_mooniswaxing` net_bool、两个 net_float | dirty 事件名 segsdirty/cyclesdirty/phasedirty/moonphasedirty/moonphasestyledirty（:85-95）；moonphase 与 waxing 共用 `moonphasedirty`（:91-92）。 |
| `OnUpdate`／`OnStaticUpdate`／`LongUpdate` | `(dt) -> nil` | 三者同一函数（:442-443）；主 sim 段切换时 set，客户端 `set_local` 并封顶（:322-379）。 |
| `OnSave`/`OnLoad` | 仅主 sim（:449、:467） | 存段数、周期、相位、`mooomphasecycle` 与相位内时间；旧档按周期推算月相（:491-496）。 |
| `Dump`／`GetDebugString` | 调试 | :510-533。 |

## 关键流程：相位推进与月相循环

1. 时间推进：`remainingtimeinphase` 递减；主 sim 跨段时 set 网络值，客户端只 `set_local` 并等待 `ms_simunpaused`/段同步强制校正（:322-341、:254-257）。
2. 相位切换：仅主 shard（`_ismastershard`）执行——按段配置循环切 day/dusk/night，回到 day 时周期 +1、推 `ms_cyclecomplete`、推进月相表 `MOON_PHASE_CYCLES`（11 项：new×1、quarter/half/threequarter/full 各 3 再对称退相，:30-50、:342-370）。
3. 副 shard 同步：非主 shard 主 sim 监听 `secondary_clockupdate` 接收主 shard 全量数据（:259-270、:305-308）；主 shard 每 tick 推 `master_clockupdate`（:424-439）。
4. 事件汇总：dirty 标志在 OnUpdate 末尾统一推 `clocksegschanged`/`cycleschanged`/`phasechanged`/`moonphasechanged`(deprecated)/`moonphasechanged2`/`moonphasestylechanged`（:381-410）。
5. 段配置：`ms_setclocksegs` 校验总和 16 并保持当天时刻不变换算新相位（:178-209）；默认 `DAY_SEGS_DEFAULT` 等 TUNING，非法总和回退全 day（:133-146）。

## 典型接入

### 让世界立即进入下一阶段（控制台/脚本同款）

- 前提与选择：世界事件仅在主 sim 生效（监听在 `_ismastersim` 分支，:294-309）；modded 客户端勿直接改 net 变量。
- 生效与撤销：`ms_setphase` 重置相位并满额剩余时间（:211-219）；`ms_nextphase` 把剩余时间清 0 触发自然推进（:221-224）；无撤销，需自存原段配置再用 `ms_setclocksegs` 恢复。
- 原版依据：debugkeys.lua:262-264。

```lua
if TheWorld.ismastersim then
    TheWorld:PushEvent("ms_nextphase")
end
```

- 调整与验证：`ms_setclocksegs` 传表须给足 16 段（`segs = {day = 10, dusk = 4, night = 2}`），否则断言失败（:189）。

## 常见陷阱与限制

- 客户端不能推进相位：非主 sim 在段末钳制等待服务端（:337-341、:376-378）；改 `TheWorld.state` 之类的读取端没有意义。
- `_moonphaselocked` 不落存档（:75 注释），加锁状态重启即失。
- `moonphasechanged` 已废弃但仍推（:400-403 注释）；新代码应监听 `moonphasechanged2`（含 waxing）。
- 月相推进发生在新周期开始而非 waxing/waning 切换处（:361 注释）；相位长短由 seasons 调整（:355），两者职责不同页互链。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，通读组件全部 539 行；挂载 world_network.lua:57、事件源 debugkeys.lua:262-264、debugcommands.lua:406/:412、shard 协作 shard_clock.lua:13 同步注释逐行核对。
- 游戏运行：未运行。

## 相关链接

- [[reference/components/seasons]]
- [[reference/components/shard_clock]]
