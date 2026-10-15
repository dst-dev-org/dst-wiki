---
title: "生成烟雾（spawn_fx 族）"
created: "2026-10-30"
updated: "2026-11-01"
type: prefab
summary: "通用表五注册的出生／离场烟雾：tiny/small/medium 三档、暂停敏感的 medium_static 与高位 small_high，由玩家进出、宠物出生、YOTB 舞台与牛／沃比消散调用；tiny 当前无生成点。"
aliases: ["spawn_fx_tiny", "spawn_fx_small", "spawn_fx_medium", "spawn_fx_medium_static", "spawn_fx_small_high", "生成烟雾", "出生烟雾", "传送门烟雾", "spawn 烟", "出生特效", "生成特效"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/fx.lua", "fx.lua", "components/playerspawner.lua", "prefabs/player_common.lua", "prefabs/world.lua", "components/yotb_stager.lua", "prefabs/beefalo.lua", "prefabs/wobysmall.lua", "prefabs/wobybig.lua", "prefabs/walter.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 生成烟雾（spawn_fx 族）

## 快速判断

- 职责与适用：实体出生／离场时的烟雾纯表现：三档常规、一个暂停敏感静态版、一个高位版；无玩法组件。协议见 [[reference/mechanisms/generic-fx-factory|通用 FX 工厂]]。[条目](../../../raw/current/scripts/fx.lua#L580-L644)
- 优先使用：查"出生烟／离场烟"选本页；玩家进出世界的装载暂停边界读下表 playerspawner 行与 [[reference/mechanisms/generic-fx-factory#关键流程|机制页暂停时钟]]。
- 重要副作用：无；tiny／small／medium 带 FinalOffset 1，`small_high` 不带。[条目](../../../raw/current/scripts/fx.lua#L580-L644)
- 修改边界：落点或父挂一次性表现；动画是否循环未核验。[代理回收](../../../raw/current/scripts/prefabs/fx.lua#L156-L157)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `spawn_fx_tiny`／`spawn_fx_small`／`spawn_fx_medium` | 通用表条目 → 加载器代理＋一帧后本地实体，fn=FinalOffset1 | bank=`spawn_fx`，build=`puff_spawning`，anim 逐名 tiny／small／medium，音效 `spawnportal_spawnplayer`。[三条](../../../raw/current/scripts/fx.lua#L580-L602) |
| `spawn_fx_medium_static` | 同 medium＋`update_while_paused = true` | 配置与 medium 全同；任务改走静态调度、动画暂停中仍播。[条目](../../../raw/current/scripts/fx.lua#L604-L611) |
| `spawn_fx_small_high` | 同 small 常规协议 | anim=`small_high`，**无 fn**（不设 FinalOffset），供飞行宠物出生用。[条目](../../../raw/current/scripts/fx.lua#L639-L644) |

定向检索简中 .po 未发现这 5 个注册名的 `STRINGS.NAMES` 键；标题中文为已核验的效果描述，非官方译名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 工厂协议 | 通用表条目；`medium_static` 是 `update_while_paused` 的原版用例 | 全族同协议 | [[reference/mechanisms/generic-fx-factory]]；[条目](../../../raw/current/scripts/fx.lua#L604-L611) |
| 生成者：玩家进出 | 离场在玩家位置播 `spawn_fx_medium_static`（可 fxoverride）；装载进场时生成同名静态版并**父挂玩家**，6 帧后显形淡入 | 服务端；静态版任务在装载暂停期间仍推进 | [离场](../../../raw/current/scripts/components/playerspawner.lua#L140-L147)、[进场](../../../raw/current/scripts/components/playerspawner.lua#L283-L296) |
| 生成者：宠物出生 | 玩家宠物生成一帧后 `DoEffects`：飞行宠物 `spawn_fx_small_high`，否则 `spawn_fx_small`；`pet.no_spawn_fx` 可关 | 服务端；迁移延迟一帧防错位 | [DoEffects](../../../raw/current/scripts/prefabs/player_common.lua#L1508-L1516) |
| 生成者：YOTB 舞台 | 移除参赛牛／训练师与奖品收集时点 medium／small | 服务端；年度活动舞台组件 | [移除](../../../raw/current/scripts/components/yotb_stager.lua#L266-L285) |
| 生成者：牛与沃比消散 | 皮牛 OnDespawnRequest 播 medium 后淡出；沃比存 `spawnfx` 字段（small／medium），消散时代理**父挂自身**再淡出移除 | 服务端；沃尔特召唤同样读该字段 | [皮牛](../../../raw/current/scripts/prefabs/beefalo.lua#L904-L911)、[字段](../../../raw/current/scripts/prefabs/wobysmall.lua#L846-L846)、[消散](../../../raw/current/scripts/prefabs/wobysmall.lua#L380-L383)、[沃尔特](../../../raw/current/scripts/prefabs/walter.lua#L742-L743) |
| 依赖声明 | world.lua 世界预载 tiny／small／small_high／medium／medium_static | 大／超大两条在注释中，未注册 | [预载](../../../raw/current/scripts/prefabs/world.lua#L199-L206) |
| 资源 | bank=`spawn_fx`，build=`puff_spawning` | 动画时长与循环性未核验 | [条目](../../../raw/current/scripts/fx.lua#L580-L644) |

## 典型接入

### 借用出生／离场烟雾

- 前提与选择：普通出现／消失用 small／medium；装载暂停期间（读档、迁移）需要表现可见时用 `spawn_fx_medium_static`；高处（飞行生物）用 small_high。
- 生效与撤销：落点一次性动画，animover 自删；父挂式（沃比／玩家进场）代理随宿主走。
- 原版依据：[玩家离场](../../../raw/current/scripts/components/playerspawner.lua#L140-L147)。

```lua
local fx = SpawnPrefab("spawn_fx_small")
fx.Transform:SetPosition(x, y, z)
-- 需要暂停期可见：SpawnPrefab("spawn_fx_medium_static")，任务走静态调度
```

- 调整与验证：`update_while_paused` 只切换任务调度与 `AnimateWhilePaused`，不改动画内容；非装载场景滥用会让烟雾在菜单暂停时继续播完（[静态分支](../../../raw/current/scripts/prefabs/fx.lua#L131-L135)）。

## 常见陷阱与限制

- **边界：`spawn_fx_tiny` 当前无生成点**：全库检索仅有 world.lua 依赖声明，未找到任何 `SpawnPrefab("spawn_fx_tiny")`；引用它不会自动有原版触发场景。[预载](../../../raw/current/scripts/prefabs/world.lua#L205-L205)
- **误区：large／huge 已注册**：两条目整段被注释，加载器不产出；需要大号烟雾只能自建或换 medium。[负例](../../../raw/current/scripts/fx.lua#L624-L637)
- **边界：small_high 没有 FinalOffset**：它是族内唯一不设 fn 的条目，显示层级与三档常规不同；别按"small 的变高版"推断它继承了排序。[条目](../../../raw/current/scripts/fx.lua#L639-L644)
- **边界：父挂到将消散的宿主**：沃比消散与玩家进场都把代理挂到宿主实体；本地表现随宿主移除如何清理属引擎父子行为，本页未核验，引用时自行确认时机。[消散](../../../raw/current/scripts/prefabs/wobysmall.lua#L380-L383)
- **边界：动画是否循环未核验**：条目仅 `PlayAnimation` 一次，未显式传循环参数，实际是否重复由动画数据决定。[播放](../../../raw/current/scripts/prefabs/fx.lua#L63-L65)

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496。五条目配置（含两条注释负例）、加载器协议、玩家进出／宠物出生／YOTB／皮牛／沃比／沃尔特生成点逐条核验，tiny 无生成点经全库检索确认；.po 名称键定向核验无结果。未覆盖：`spawn_fx_ocean_static`（同表相邻条目，本批不覆盖）、动画资源内容、晚加入时序。
- 游戏运行：未运行；烟雾在装载暂停中的实际可见性未实测。

返回 [[reference/prefabs/fx-table-world-index|通用表 FX·环境与物件]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-2）。
