---
title: "月之装置破碎与月暴玻璃爆裂（moonstorm_glass_fx 族）"
created: "2026-11-01"
updated: "2026-11-01"
type: prefab
summary: "月之装置二、三阶段破碎与月暴玻璃爆裂的四个通用表动画注册。"
aliases: ["moon_device_break_stage2", "moon_device_break_stage3", "moonstorm_glass_ground_fx", "moonstorm_glass_fx", "月之装置破碎", "月暴玻璃爆裂"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["fx.lua", "prefabs/fx.lua", "prefabs/moon_device.lua", "prefabs/moonstorm_glass.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 月之装置破碎与月暴玻璃爆裂（moonstorm_glass_fx 族）

## 快速判断

- 四注册覆盖装置破碎两阶段和月暴玻璃爆裂两层；伤害、生成天体英雄和实体移除由生成者处理。代理与本地动画清理见 [[reference/mechanisms/generic-fx-factory#关键流程|通用 FX 工厂流程]]。
- 以上注册名在简中 `.po` 未找到显示名键；中文为效果描述，非官方译名。

## 注册与构造

共同由通用表工厂构造；以下 sound 未列即无 sound，fn 未列即无 fn。

| 注册名 | bank / build / anim | 差异与证据 |
|---|---|---|
| `moon_device_break_stage2` | `moon_device_break` / 同 bank / `stage2_break` | fn 设八方向。[条目](../../../raw/current/scripts/fx.lua#L2408-L2414) |
| `moon_device_break_stage3` | `moon_device_break` / 同 bank / `stage3_break` | 无 fn。[条目](../../../raw/current/scripts/fx.lua#L2417-L2421) |
| `moonstorm_glass_ground_fx` | `moonglass_charged` / `moonglass_charged_tile` / `explosion` | fn 为 `GroundOrientation`，贴地背景层。[条目](../../../raw/current/scripts/fx.lua#L2423-L2427)、[函数](../../../raw/current/scripts/fx.lua#L24-L27) |
| `moonstorm_glass_fx` | `moonglass_charged` / `moonglass_charged_tile` / `crack_fx` | 无 fn。[条目](../../../raw/current/scripts/fx.lua#L2430-L2434) |

## 关联机制导航

| 生成者 | 条件、落点和证据 |
|---|---|
| 月之装置 | `breaksequence` 第 9 帧 `break_device`：三阶段 FX 在装置位置，二阶段对每个柱子在其位置与旋转生成；柱子与顶端下一帧移除，同次序另调用独立的 stage1，再于下一帧生成天体英雄。[破碎](../../../raw/current/scripts/prefabs/moon_device.lua#L190-L204)、[序列](../../../raw/current/scripts/prefabs/moon_device.lua#L311-L335) |
| 月暴玻璃 | `explode` 的 crack 动画结束时在自身位置同点生成两个 FX 并播放声响；搜索半径 4 内目标并对符合条件的 combat 对象造成 30 伤害，然后移除玻璃。破碎音效来自玻璃，非这两条目。[爆裂](../../../raw/current/scripts/prefabs/moonstorm_glass.lua#L23-L59) |
| 通用工厂 | 非持久代理和本地显示实体的端分界及动画结束清理 | [[reference/mechanisms/generic-fx-factory#关键流程|工厂流程]] |

## 常见陷阱与限制

- `moon_device_break_stage1` 是同源文件里的独立注册，不是本页通用表四名之一。[注册](../../../raw/current/scripts/prefabs/moon_device.lua#L643-L643)
- 玻璃定时结束与已消解存档加载均可调用 `explode`；挖矿用完 work 的路径另行掉落并移除，不能混为爆裂。[触发](../../../raw/current/scripts/prefabs/moonstorm_glass.lua#L63-L96)

## 验证记录

- 源码审阅：2026-11-01，Steam buildid 26335394；核对四个表项及构造、装置破碎和玻璃爆裂生成链；全 scripts 检索 `moon_device_break_stage`、`moonstorm_glass_fx`、`moonstorm_glass_ground_fx`，未将声明视为生成。未核验动画资源内部及模组生成。
- 游戏运行：未运行。

返回 [[reference/prefabs/fx-world-index|天气、地形与生态痕迹]]。
