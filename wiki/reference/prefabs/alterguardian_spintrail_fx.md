---
title: "天体英雄与月脉冲特效（alterguardian_spintrail_fx 等）"
created: "2026-11-01"
updated: "2026-11-01"
type: prefab
summary: "天体英雄轨迹与尖刺、月装置破裂、陷阱结束及月脉冲的七个通用表表现。"
aliases: ["alterguardian_spike_breakfx", "alterguardian_spintrail_fx", "alterguardian_phase1fallfx", "alterguardian_phase3trappst", "moon_geyser_explode", "moonpulse_fx", "moonpulse2_fx", "天体英雄轨迹", "月脉冲特效"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["fx.lua", "simutil.lua", "prefabs/alterguardian_phase2spike.lua", "prefabs/alterguardian_phase3trap.lua", "prefabs/moon_device.lua", "prefabs/moonpulse.lua", "stategraphs/SGalterguardian_phase1.lua", "stategraphs/SGalterguardian_phase2.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 天体英雄与月脉冲特效（alterguardian_spintrail_fx 等）

## 快速判断

- 七个通用表注册名覆盖天体英雄战斗与月装置演出：尖刺受阻、旋转轨迹、装置破裂和两种月脉冲。伤害、刷怪及屏幕后处理由生成方负责。工厂代理与各端本地表现的共同协议见 [[reference/mechanisms/generic-fx-factory#关键流程]]。
- 中文为按条目及调用语境核验的效果描述，非官方译名；`languages/chinese_s.po` 定向检索七名无显示名键。实际动画外观与时长未作游戏验证。

## 注册与构造

下表均由通用工厂注册；未列的 `sound`、`fn` 字段未设置，bank/build 相同时合写。

| 注册名 | bank / build；anim | 差异与证据 |
|---|---|---|
| `alterguardian_spike_breakfx` | `alterguardian_spike`；`spike_pst` | 无 sound、fn。[条目](../../../raw/current/scripts/fx.lua#L2386-L2389) |
| `alterguardian_spintrail_fx` | `alterguardian_sinkhole`；`pre` | 无 sound；`animqueue = true`；fn 设置地面背景层、八向，追加循环 `idle`，60 帧后启动 60 帧侵蚀。[条目](../../../raw/current/scripts/fx.lua#L2392-L2405)、[地面设置](../../../raw/current/scripts/fx.lua#L24-L27) |
| `alterguardian_phase1fallfx` | `alterguardian_spawn_death`；`fall_pre` | 无 sound、fn。[条目](../../../raw/current/scripts/fx.lua#L2446-L2449) |
| `moon_geyser_explode` | `moon_altar_geyser` / `moon_geyser`；`explode` | 无 sound、fn。[条目](../../../raw/current/scripts/fx.lua#L2452-L2455) |
| `moonpulse_fx` | `moon_altar_geyser` / `moon_geyser`；`moonpulse` | 无 sound；fn 设置 `shaders/anim.ksh` 泛光。[条目](../../../raw/current/scripts/fx.lua#L2458-L2464) |
| `moonpulse2_fx` | 同上；`moonpulse2` | 无 sound；fn 同上。[条目](../../../raw/current/scripts/fx.lua#L2467-L2473) |
| `alterguardian_phase3trappst` | `alterguardian_meteor`；`meteor_pst` | sound `turnoftides/common/together/moon_glass/mine`，无 fn。[条目](../../../raw/current/scripts/fx.lua#L2476-L2480) |

## 关联机制导航

| 生成者／触发 | 落点、端与关联 | 证据 |
|---|---|---|
| 二阶段尖刺轨迹 `emerge` → `try_spawn_spike` | 对候选点先 `DoAttack`；命中时在该点生成 `alterguardian_spike_breakfx`，否则生成尖刺实体。两侧额外候选点须可通行。主模拟任务触发，FX 不父挂。 | [分支](../../../raw/current/scripts/prefabs/alterguardian_phase2spike.lua#L76-L85)、[展开](../../../raw/current/scripts/prefabs/alterguardian_phase2spike.lua#L91-L120)、[任务安装](../../../raw/current/scripts/prefabs/alterguardian_phase2spike.lua#L232-L249) |
| 一阶段 SG `spawn_landfx` | 多个状态的时间轴调用，在 Boss 坐标落 `alterguardian_spintrail_fx` 与 `mining_moonglass_fx`；例如一处第 1 帧、另一处第 18 帧。 | [生成](../../../raw/current/scripts/stategraphs/SGalterguardian_phase1.lua#L233-L236)、[例一](../../../raw/current/scripts/stategraphs/SGalterguardian_phase1.lua#L399-L404)、[例二](../../../raw/current/scripts/stategraphs/SGalterguardian_phase1.lua#L485-L492) |
| 二阶段 SG `spawn_spintrail` | 旋转状态更新中按时间窗口周期调用，在 Boss 当前位置生成同一轨迹和月玻璃效果；伤害由 SG 另行处理。 | [生成](../../../raw/current/scripts/stategraphs/SGalterguardian_phase2.lua#L49-L53)、[更新一](../../../raw/current/scripts/stategraphs/SGalterguardian_phase2.lua#L311-L318)、[更新二](../../../raw/current/scripts/stategraphs/SGalterguardian_phase2.lua#L436-L443) |
| 月装置 `breaksequence` | 装置位置生成 `alterguardian_phase1fallfx`；9 帧后 `break_device` 执行三阶段破裂，其中 stage1 在 x/z、y=0 生成 `moon_geyser_explode`，再延迟 1 帧生成 Boss。 | [落下与延时](../../../raw/current/scripts/prefabs/moon_device.lua#L329-L334)、[破裂与 Boss](../../../raw/current/scripts/prefabs/moon_device.lua#L320-L325)、[爆发落点](../../../raw/current/scripts/prefabs/moon_device.lua#L289-L295) |
| 三阶段陷阱移除 | 主模拟 `onremove` 清理存留的 pulse FX、掉落战利品，并在陷阱位置生成 `alterguardian_phase3trappst`；寿命计时也是移除的入口之一。 | [回调](../../../raw/current/scripts/prefabs/alterguardian_phase3trap.lua#L262-L277)、[监听](../../../raw/current/scripts/prefabs/alterguardian_phase3trap.lua#L318-L325) |
| `moonpulse` 本地效果 | `moonpulse_spawner` 非专服侧创建无网络 `moonpulse`；三阶段进入 0.18 秒后分别生成小、小、大波，即 `moonpulse_fx` 两次和 `moonpulse2_fx` 一次，落在 pulse 的 x/z、y=0。屏幕闪光、镜头摇动与声音由 `moonpulse` 另管。 | [端与创建](../../../raw/current/scripts/prefabs/moonpulse.lua#L190-L204)、[spawner](../../../raw/current/scripts/prefabs/moonpulse.lua#L223-L240)、[三阶段](../../../raw/current/scripts/prefabs/moonpulse.lua#L78-L100)、[小波](../../../raw/current/scripts/prefabs/moonpulse.lua#L44-L55)、[大波](../../../raw/current/scripts/prefabs/moonpulse.lua#L58-L75) |

## 常见陷阱与限制

- 轨迹的 `idle` 是循环动画，不能把普通一次性动画结束回收当作其唯一寿命：条目 60 帧后调用 `ErodeAway`，helper 随时间侵蚀并移除本地实体。[helper](../../../raw/current/scripts/simutil.lua#L575-L597)；代理与本地实体的区别见 [[reference/mechanisms/generic-fx-factory#常见陷阱与限制]]。
- `moonpulse_fx`／`moonpulse2_fx` 表项没有音效；wave1／wave2 声音从 `TheFocalPoint` 播放。月脉冲生成点在各非专服的本地控制器，不能概括为全部由主模拟生成。
- SG 与尖刺、陷阱、装置各自产生的伤害、掉落和 Boss 不由本页 FX 执行。FX 均按上表落点生成，未设置父挂。

## 验证记录

- 源码审阅：2026-11-01，Steam buildid 26335394；逐项核验表项 bank/build/anim/sound/fn 与轨迹 animqueue。递归检索 `raw/current/scripts` 中七个完整注册名与 `alterguardian_spike`、`alterguardian_spintrail`、`alterguardian_phase1fall`、`alterguardian_phase3trap`、`moon_geyser`、`moonpulse` 前缀，检查动态拼名；命中调用按上表列出，依赖声明不计生成。未检查资源动画帧或游戏演出。
- 游戏运行：未运行。

返回 [[reference/prefabs/fx-table-index|通用表 FX 索引]]。
