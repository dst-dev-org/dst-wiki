---
title: "震地尘土（groundpound_fx）"
created: "2026-10-31"
updated: "2026-11-01"
type: prefab
summary: "两个通用表注册承载震地落点尘土（默认熊獾版与伍迪海狸换装版，同一 bearger_ground_fx 资源）：groundpounder 组件按环点在陆地非码头处落 (x,0,z)，fxRings 控制外圈是否出尘；中心冲击圈是独立工厂 groundpoundring_fx 不在本页。"
aliases: ["groundpound_fx", "werebeaver_groundpound_fx", "震地尘土", "震地特效", "熊獾震地", "海狸震地"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["fx.lua", "components/groundpounder.lua", "prefabs/woodie.lua", "prefabs/bearger.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 震地尘土（groundpound_fx）

## 快速判断

- 职责与适用：groundpounder 震地时沿环点铺开的落点尘土，两注册共用熊獾资源、仅音效不同（默认／伍迪海狸）。工厂协议见 [[reference/mechanisms/generic-fx-factory|通用 FX 工厂]]。[条目](../../../raw/current/scripts/fx.lua#L866-L878)
- 优先使用：查"震地尘土／震地特效"到本页；组件配置读 [[reference/components/groundpounder|groundpounder]]。
- 重要副作用：fx 自带 dustpoof／woodie ground_fx 声；逐环延迟任务按环点生成。[条目](../../../raw/current/scripts/fx.lua#L866-L878)
- 修改边界：只落在陆地且非码头的环点（y 压平 0）；水面震地不出尘。[门禁](../../../raw/current/scripts/components/groundpounder.lua#L284-L290)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `groundpound_fx` | 通用表条目，无 fn | bank/build=`bearger_ground_fx`，anim=`idle`，sound=`bearger/dustpoof`；groundpounder 组件默认值。[条目](../../../raw/current/scripts/fx.lua#L866-L871) |
| `werebeaver_groundpound_fx` | 同上协议 | 同 bank/build，sound=`meta2/woodie/ground_fx`；woodie 海狸模式覆写组件字段。[条目](../../../raw/current/scripts/fx.lua#L873-L878) |

两名在简中 .po 均无 `STRINGS.NAMES` 键；标题中文为已核验的效果描述，非官方译名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 工厂协议 | 代理＋一帧本地表现，落点式 | 组件字段 `groundpoundfx` 决定注册名 | [[reference/mechanisms/generic-fx-factory]] |
| 生成入口：GroundPound | `GroundPound(pt)`：先在中心落独立工厂 `groundpoundring_fx`（默认 fx 时按命中环数自动缩放），再逐环延迟调 DestroyPoints／DestroyRing，各环 `spawnfx = i <= (fxRings or numRings)` | 服务端；usePointMode 为废弃兼容路径 | [入口](../../../raw/current/scripts/components/groundpounder.lua#L307-L324)、[逐环派发](../../../raw/current/scripts/components/groundpounder.lua#L326-L351) |
| 环点落尘 | DestroyPoints／DestroyRing 在每个环点检查 `IsLandTileAtPoint` 且非 `IsDockAtPoint` 后，落 `groundpoundfx` 于 `(v.x, 0, v.z)` | 服务端；水面／码头环点跳过 | [点模式](../../../raw/current/scripts/components/groundpounder.lua#L161-L163)、[环模式](../../../raw/current/scripts/components/groundpounder.lua#L284-L290) |
| 换名覆写 | woodie 海狸模式把 `groundpoundfx` 覆写为 `werebeaver_groundpound_fx`（同时配 damageRings／destructionRings／workefficiency 等） | 服务端；wereworker 接线 | [覆写](../../../raw/current/scripts/prefabs/woodie.lua#L824-L830) |
| 默认宿主 | bearger 依赖声明两注册并走默认尘土与中心圈 | 服务端 | [声明](../../../raw/current/scripts/prefabs/bearger.lua#L24-L24) |
| 资源 | 共用 bank/build=`bearger_ground_fx`（熊獾资源复用），anim=`idle` | 动画时长未核验 | [条目](../../../raw/current/scripts/fx.lua#L866-L878) |

## 典型接入

### 模组震地复用尘土

- 前提与选择：服务端宿主挂 groundpounder 组件；`GroundPound` 是完整震地入口——默认前 `damageRings`（2）环经宿主 `combat` 结算（缺 combat 组件会在伤害分支报错）、默认前 `destructionRings`（3）环走工作破坏、默认前 `platformPushingRings`（2）环推船，**不是纯尘土表现**。仅要表现时把三类环数清零或自写落尘循环。
- 生效与撤销：逐环延迟落点动画，animover 自删；外圈是否出尘由 `fxRings` 控制。
- 原版依据：[组件字段](../../../raw/current/scripts/components/groundpounder.lua#L20-L21)、[海狸覆写](../../../raw/current/scripts/prefabs/woodie.lua#L830-L830)、[伤害分支](../../../raw/current/scripts/components/groundpounder.lua#L114-L135)。

```lua
-- 服务端；宿主需已挂 combat（默认伤害环经 combat 结算）
inst:AddComponent("groundpounder")
inst.components.groundpounder:UseRingMode()                     -- 默认 PointMode 已废弃，显式选环模式
inst.components.groundpounder.groundpoundfx = "groundpound_fx"  -- 默认已是该值，可换注册名
inst.components.groundpounder.fxRings = 2                       -- 只前 2 环出尘
-- 仅要尘土表现时清零结算与推船环（否则同时执行伤害／破坏／推船）：
inst.components.groundpounder.damageRings = 0
inst.components.groundpounder.destructionRings = 0
inst.components.groundpounder.platformPushingRings = 0    -- 默认 2，会对船上目标 boatphysics:ApplyForce
inst.components.groundpounder:GroundPound(pt)
```

- 调整与验证：中心冲击圈另由 `groundpoundringfx` 字段控制（默认 `groundpoundring_fx`）；换尘土注册不影响圈。

## 常见陷阱与限制

- **边界：两名同资源不同声**：bank/build 都是 `bearger_ground_fx`，差异只有 sound 字段与组件覆写来源；按资源判"同一表现"会漏掉海狸换名路径。[条目](../../../raw/current/scripts/fx.lua#L866-L878)
- **边界：尘土与中心圈是两套 fx**：`groundpound_fx` 沿环点铺开，`groundpoundring_fx` 是独立工厂的中心圈（命中环数自动缩放）；配置震地表现时两者分开调。[入口](../../../raw/current/scripts/components/groundpounder.lua#L307-L324)
- **边界：环点门禁**：非陆地（水面）或码头上的环点不出尘，但伤害与推船等结算照常进行——表现缺失不代表震地未生效。[门禁](../../../raw/current/scripts/components/groundpounder.lua#L284-L290)
- **边界：fxRings 缺省为全部环**：`fxRings or numRings`——不设 fxRings 时每一环都出尘；大半径震地会铺满整个环带。[派发](../../../raw/current/scripts/components/groundpounder.lua#L326-L351)

## 验证记录

- 源码审阅：2026-10-31，Steam buildid 26230496。两注册配置、groundpounder 入口与逐环派发（fxRings 缺省）、环点陆地／码头门禁、woodie 海狸覆写、bearger 依赖声明核验。未覆盖：groundpoundring_fx 工厂内部、bearger／其他宿主的完整震地调用面、动画资源内容、晚加入时序。
- 游戏运行：未运行；震地视觉未实测。

返回 [[reference/prefabs/fx-table-world-index|通用表 FX·环境与物件]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-1）。
