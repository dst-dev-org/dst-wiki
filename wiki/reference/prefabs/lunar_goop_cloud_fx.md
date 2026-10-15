---
title: "牧者黏液雾云（lunar_goop_cloud_fx）"
created: "2026-10-29"
updated: "2026-11-01"
type: prefab
summary: "月牧者（Grazer）周身悬浮的雾气粒子实体：联网纯粒子发射器，父级 IsCloudEnabled 关断发射，催眠玩法在宿主侧，本实体只做视觉。"
aliases: ["lunar_goop_cloud_fx", "牧者雾云", "月牧者雾气", "lunar goop cloud"]
tags: [prefab, fx, server-client]
size_class: simple
dst_build: "steam-buildid:26335394"
sources: ["prefabs/lunar_goop_cloud_fx.lua", "prefabs/lunar_grazer.lua", "stategraphs/SGlunar_grazer.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 牧者黏液雾云（lunar_goop_cloud_fx）

## 快速判断

- 职责与适用：月牧者（官方译名"牧者"，[NAMES](../../../raw/current/scripts/languages/chinese_s.po#L344304-L344307)）身上的持续雾气**粒子发射器实体**：联网、单发射器烟粒子，挂在牧者实体下随其移动；工厂无任何玩法组件，催眠效果是宿主牧者的独立周期任务，不经过本实体。[工厂](../../../raw/current/scripts/prefabs/lunar_goop_cloud_fx.lua#L63-L117)
- 优先使用：查"牧者雾气／月精华云视觉"选本页；要复用"被父级开关的跟随粒子"可借鉴本页发射器结构；催眠数值与豁免查宿主牧者源码（本批未建页）。
- 重要副作用：发射器每帧回调按父级 `IsCloudEnabled()` 门控——父级没有该方法或返回真才发射；专用服务器提前 return，完全不装粒子（也无自移除任务，寿命随父级）。[门控](../../../raw/current/scripts/prefabs/lunar_goop_cloud_fx.lua#L108-L113)、[专用分界](../../../raw/current/scripts/prefabs/lunar_goop_cloud_fx.lua#L76-L78)
- 修改边界：persists=false 不入存档；粒子参数（1.5 秒寿命、50 上限、球域 2.5、reveal 着色器）写死在工厂。[粒子配置](../../../raw/current/scripts/prefabs/lunar_goop_cloud_fx.lua#L85-L96)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `lunar_goop_cloud_fx` | fn：Transform＋Network＋FX 标签 → SetPristine → persists=false → 专用服务器提前 return → 非 InitEnvelope 注册色/缩放包 → VFXEffect 单发射器（烟粒子）→ EmitterManager 每帧回调：父级未关云才按帧数补发粒子 | 单一注册；无自移除任务，作为牧者的常驻子实体存在。[工厂](../../../raw/current/scripts/prefabs/lunar_goop_cloud_fx.lua#L63-L117)、[注册](../../../raw/current/scripts/prefabs/lunar_goop_cloud_fx.lua#L119) |

牧者本体官方译名"牧者"（NAMES.LUNAR_GRAZER，[译名](../../../raw/current/scripts/languages/chinese_s.po#L344304-L344307)）；本 FX 注册名无 NAMES 条目，标题"黏液雾云"为已核验效果描述，非官方译名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 宿主：lunar_grazer | 牧者初始化时生成并 `SetParent` 到自身（`inst.cloud`），实体随宿主移动、宿主移除时一并消失 | 主模拟生成；客户端经网络拿到同一父子关系 | [挂载](../../../raw/current/scripts/prefabs/lunar_grazer.lua#L429-L430) |
| 开关协议：IsCloudEnabled | 发射回调读父级 `IsCloudEnabled()`；宿主用 net_bool `_cloudenabled` 承载，状态图各状态成对 EnableCloud(false/true)，本 FX 只据此停发粒子；宿主 EnableCloud 还会启停自己的 cloudtask | 客户端可读（net_bool 同步）；本实体不监听事件、每帧轮询父级方法 | [门控](../../../raw/current/scripts/prefabs/lunar_goop_cloud_fx.lua#L108-L113)、[宿主定义](../../../raw/current/scripts/prefabs/lunar_grazer.lua#L416-L418)、[状态图开关](../../../raw/current/scripts/stategraphs/SGlunar_grazer.lua#L126-L142) |
| 催眠玩法（在宿主，不在本实体） | 牧者 cloudtask 每 1 秒在 CLOUD_RADIUS 2.5 内对睡眠者 AddGrogginess(1, 6)；云开启才跑（EnableCloud 控制 Start/StopCloudTask） | 主模拟；玩家经 grogginess、生物经 sleeper 分支；0.5 秒保护窗防连触 | [DoCloudTask](../../../raw/current/scripts/prefabs/lunar_grazer.lua#L197-L225)、[开关接线](../../../raw/current/scripts/prefabs/lunar_grazer.lua#L235-L245)、[数值](../../../raw/current/scripts/tuning.lua#L7646-L7647) |
| 资源 | fx/animsmoke.tex＋shaders/vfx_particle_reveal.ksh，色/缩放包 `lunar_goop_cloud_*envelope_smoke` 首次生成时注册 | 客户端；声明与使用已核验 | [资源与包](../../../raw/current/scripts/prefabs/lunar_goop_cloud_fx.lua#L1-L12)、[注册包](../../../raw/current/scripts/prefabs/lunar_goop_cloud_fx.lua#L20-L42) |

## 典型接入

### 复用"跟随宿主、可开关的粒子云"

- 前提与选择：生成后 `SetParent` 到宿主；宿主暴露 `IsCloudEnabled` 方法即接入门控协议，不想被门控就什么都不加（回调对无方法的父级默认发射）。
- 生效与撤销：粒子发射随父级开关启停；实体不自移除，宿主移除时子实体随之消失（persists=false 不入存档）。
- 原版依据：牧者是唯一生成者，状态图用成对 EnableCloud 控制视觉与催眠任务同步启停。[挂载](../../../raw/current/scripts/prefabs/lunar_grazer.lua#L429-L430)、[开关](../../../raw/current/scripts/prefabs/lunar_grazer.lua#L235-L245)

```lua
local cloud = SpawnPrefab("lunar_goop_cloud_fx")
cloud.entity:SetParent(host.entity)          -- 宿主需提供 IsCloudEnabled 才可关断
-- 关断：host._cloudenabled:set(false)（牧者协议）或自行实现 IsCloudEnabled
```

- 调整与验证：专用服务器上无粒子属正常（[提前 return](../../../raw/current/scripts/prefabs/lunar_goop_cloud_fx.lua#L76-L78)）；改粒子节奏改 `SMOKE_MAX_LIFETIME` 与每帧补发逻辑（[发射器](../../../raw/current/scripts/prefabs/lunar_goop_cloud_fx.lua#L100-L114)）。

## 常见陷阱与限制

- **误区：实体名带 cloud 就会催眠**：本实体只有粒子发射器；催眠在牧者的 cloudtask，沿实体找玩法会扑空。要自定义催眠请读宿主 DoCloudTask 的分支结构。[DoCloudTask](../../../raw/current/scripts/prefabs/lunar_grazer.lua#L197-L225)
- **边界：门控是每帧轮询父级方法，不是事件**：父级切 `IsCloudEnabled` 的同一帧即生效（回调里现读），但父级没有该方法时视为常开；挂到不认识协议的宿主上无法关断。[门控](../../../raw/current/scripts/prefabs/lunar_goop_cloud_fx.lua#L108-L113)
- **边界：专用服务器与客户端都跑 fn，但表现不同**：专用提前 return 无粒子也无发射器；联网实体靠 AddNetwork 同步存在性，客户端各自本地发粒子，无逐粒子同步。[分界](../../../raw/current/scripts/prefabs/lunar_goop_cloud_fx.lua#L63-L78)
- **边界：无自移除路径**：工厂不安排 Remove，脱离父级（如手动 SetParent(nil)）会常驻场景；复用时注意。[工厂](../../../raw/current/scripts/prefabs/lunar_goop_cloud_fx.lua#L63-L117)

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496；120 行工厂全文、lunar_grazer 挂载与 cloudtask／EnableCloud 协议、SGlunar_grazer 开关调用样例、tuning 催眠数值已核验；粒子视觉、云关闭时的表现、牧者整页机制未核验（属牧者页面范围）。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、label-check 通过（结果见本批交付报告）。

返回 [[reference/prefabs/fx-world-index|特效 Prefab 索引·环境与世界]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-2）。
