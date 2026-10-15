---
title: "作物挖掘烟（dirt_puff）"
created: "2026-10-30"
updated: "2026-11-01"
type: prefab
summary: "农场作物按阶段配置的挖掘烟：farm_plants 的 dig_fx 字段在挖除时于植株位置落点生成；与 small_puff 同 build 不同注册（多 FinalOffset、无声）。"
aliases: ["dirt_puff", "作物挖掘烟", "挖除烟", "农田挖掘特效"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/fx.lua", "fx.lua", "prefabs/farm_plants.lua", "prefabs/carnivalgame_feedchicks.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 作物挖掘烟（dirt_puff）

## 快速判断

- 职责与适用：农场作物被挖除（铲除植株）时的烟团表现，单注册；由 farm_plants 各阶段数据的 `dig_fx` 字段驱动，另有鸦年华喂鸡小游戏直接落点。协议见 [[reference/mechanisms/generic-fx-factory|通用 FX 工厂]]。[条目](../../../raw/current/scripts/fx.lua#L313-L319)
- 优先使用：查"作物挖除烟／dirt_puff"选本页；与 [[reference/prefabs/small_puff|small_puff]]（变身烟）是同 build 的两个注册，语义按消费者走。
- 重要副作用：无音效——数据条目的 sound 行被注释（small_puff 同 build 有死亡扑扑声）。[条目](../../../raw/current/scripts/fx.lua#L313-L319)
- 修改边界：植株位置一次性落点（不父挂）；动画是否重复未核验。[落点](../../../raw/current/scripts/prefabs/farm_plants.lua#L76-L77)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `dirt_puff` | 通用表条目 → 加载器代理＋一帧后本地实体，fn=FinalOffset1 | bank=`small_puff`、build=`smoke_puff_small`、anim=`puff`；sound 行注释。[条目](../../../raw/current/scripts/fx.lua#L313-L319) |

注册名在简中 .po 无 `STRINGS.NAMES` 键；标题中文为已核验的效果描述，非官方译名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 工厂协议 | 通用表条目，fn=FinalOffset1 | 单注册 | [[reference/mechanisms/generic-fx-factory]]；[条目](../../../raw/current/scripts/fx.lua#L313-L319) |
| 生成者：作物挖除 | farm_plants 各阶段数据带 `dig_fx = "dirt_puff"`，挖除处理在**植株位置**生成（多阶段条目），prefabs 表按阶段数据收集依赖 | 服务端；阶段字段决定是否生成 | [生成](../../../raw/current/scripts/prefabs/farm_plants.lua#L76-L77)、[依赖收集](../../../raw/current/scripts/prefabs/farm_plants.lua#L787-L788) |
| 生成者：喂鸡小游戏收尾 | `carnivalgame_feedchicks` 的 `RemoveGameItems` 在**清理场上剩余食物**时逐个在食物位置落点（非投喂命中路径） | 服务端 | [清理落点](../../../raw/current/scripts/prefabs/carnivalgame_feedchicks.lua#L184-L191) |
| 资源 | bank=`small_puff`、build=`smoke_puff_small`——与 [[reference/prefabs/small_puff|small_puff]] 同 build | 动画时长与循环性未核验 | [条目](../../../raw/current/scripts/fx.lua#L313-L319) |

## 典型接入

### 复用挖掘烟

- 前提与选择：给自定义作物阶段配 `dig_fx = "dirt_puff"` 即接入 farm_plants 链路；任意位置复用直接 Spawn。
- 生效与撤销：落点一次性动画，animover 自删。
- 原版依据：[dig_fx 生成](../../../raw/current/scripts/prefabs/farm_plants.lua#L76-L77)。

```lua
local fx = SpawnPrefab("dirt_puff")
fx.Transform:SetPosition(plant.Transform:GetWorldPosition())
```

- 调整与验证：要无声烟选 dirt_puff，要有 deathpoof 声选 [[reference/prefabs/small_puff|small_puff]]——两者 build 相同但注册独立。

## 常见陷阱与限制

- **误区：与 small_puff 同一表现**：同 bank 同 build 同 anim，但 dirt_puff 多 `FinalOffset 1`、无声、生成路径完全不同（作物挖除 vs 变身）；按语义选注册。[对照](../../../raw/current/scripts/fx.lua#L193-L198)
- **边界：无声是注释掉的**：sound 行保留在数据里但被注释；恢复声音即取消注释（改数据表）。[条目](../../../raw/current/scripts/fx.lua#L313-L319)
- **边界：dig_fx 按阶段配置**：不是所有阶段都带 dig_fx（barren 等阶段字段为 nil 时不生成）；复刻时按 stage_data 判断。[字段](../../../raw/current/scripts/prefabs/farm_plants.lua#L76-L77)
- **边界：动画是否重复未核验**：条目仅 `PlayAnimation` 一次，未显式传循环参数，实际是否重复由动画数据决定。[播放](../../../raw/current/scripts/prefabs/fx.lua#L63-L65)

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496。单注册配置（含注释音效）、加载器协议、farm_plants dig_fx 生成与依赖收集、喂鸡小游戏直落点、与 small_puff 的同 build 辨析核验；.po 名称键定向核验无结果。未覆盖：`smoke_puff_small` 动画资源内容、farm_plants 全阶段清单、晚加入时序。
- 游戏运行：未运行；烟团视觉未实测。

返回 [[reference/prefabs/fx-table-world-index|通用表 FX·环境与物件]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-2）。
