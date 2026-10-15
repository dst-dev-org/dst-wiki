---
title: "坠落云与线条（fallingswish_clouds 等）"
created: "2026-11-01"
updated: "2026-11-01"
type: prefab
summary: "虚空坠落的普通云、快速云和线条三个通用表表现，以及实体沉没路径的选名。"
aliases: ["fallingswish_clouds", "fallingswish_clouds_fast", "fallingswish_lines", "坠落云", "坠落线条", "虚空坠落特效"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["fx.lua", "stategraphs/commonstates.lua", "ocean_util.lua", "prefabs/oceanwhirlbigportal.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 坠落云与线条（fallingswish_clouds 等）

## 快速判断

- 三名是通用表动画，普通云与线条可配对用于虚空落下，快速云用于 `abyss_drop` 状态；落下传送及伤害等状态不由 FX 自身实现。代理和本地实体的分工见 [[reference/mechanisms/generic-fx-factory#关键流程]]。
- 标题中文为按条目与调用语境核验的效果描述，非官方译名；`languages/chinese_s.po` 按三名检索无显示名键。

## 注册与构造

均用 bank `fallingswish`、build `falling_swish`，无 sound 字段。共同加载器见 [[reference/mechanisms/generic-fx-factory#关键接口]]。

| 注册名 | anim／fn 差异与证据 |
|---|---|
| `fallingswish_clouds` | anim `clouds`；fn `FinalOffset1`。[条目](../../../raw/current/scripts/fx.lua#L3558-L3562)、[函数](../../../raw/current/scripts/fx.lua#L1-L3) |
| `fallingswish_clouds_fast` | anim `clouds_fast`；fn `FinalOffset1`。[条目](../../../raw/current/scripts/fx.lua#L3565-L3569) |
| `fallingswish_lines` | anim `lines`；fn 设 FinalOffset 1 与 `LAYER_BELOW_GROUND`。[条目](../../../raw/current/scripts/fx.lua#L3572-L3579) |

## 关联机制导航

| 生成者／触发 | 选名及落点 | 证据 |
|---|---|---|
| `CommonStates` 虚空下坠 | `DoVoidFall` 在 `skip_vfx` 未置位时于实体坐标同时生成普通云与线条；状态按动画或跳过分支调用。 | [生成](../../../raw/current/scripts/stategraphs/commonstates.lua#L2291-L2295)、[状态分支](../../../raw/current/scripts/stategraphs/commonstates.lua#L2338-L2357) |
| `CommonStates` 的 `abyss_drop` | 进入状态停移动与 Brain 后在实体坐标生成快速云。 | [进入](../../../raw/current/scripts/stategraphs/commonstates.lua#L2403-L2426) |
| `SinkEntity` 和旋涡 | `GetSinkEntityFXPrefabs` 先按 OceanTile 选水花，再对 `TileGroupManager:IsImpassableTile` 选 VOID 数组；`SinkEntity` 遍历生成普通云及线条，旋涡入口也用该选择器。地图外和其他瓦片回退旧海洋特效。 | [选择](../../../raw/current/scripts/ocean_util.lua#L188-L207)、[沉没](../../../raw/current/scripts/ocean_util.lua#L228-L232)、[旋涡](../../../raw/current/scripts/prefabs/oceanwhirlbigportal.lua#L209-L217) |

## 常见陷阱与限制

- 三名均无条目 sound；屏幕、角色状态和声音的其他行为需查具体调用方。上述生成用落点坐标，不父挂；通用回收及服务端／本地边界见 [[reference/mechanisms/generic-fx-factory#常见陷阱与限制]]。

## 验证记录

- 源码审阅：2026-11-01，Steam buildid 26335394。逐条核验 bank/build/anim/sound/fn；递归检索 `raw/current/scripts` 三个完整名及 `fallingswish` 前缀，追查 VOID 数组的消费者。未验证动画包与游戏中表现。
- 游戏运行：未运行。

返回 [[reference/prefabs/fx-table-index|通用表 FX 索引]]。
