---
title: "牧者黏液拖尾（lunar_goop_trail_fx）"
created: "2026-10-29"
updated: "2026-11-01"
type: prefab
summary: "月牧者走位留下的黏液地面拖尾：三段动画状态机（pre→主体→pst），时长与缩放由生成方传入，宿主用对象池回收，支持 Dissipate 提前淡出。"
aliases: ["lunar_goop_trail_fx", "牧者拖尾", "黏液拖尾", "lunar goop trail"]
tags: [prefab, fx, server-client]
size_class: simple
dst_build: "steam-buildid:26335394"
sources: ["prefabs/lunar_goop_trail_fx.lua", "prefabs/lunar_grazer.lua", "stategraphs/SGlunar_grazer.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 牧者黏液拖尾（lunar_goop_trail_fx）

## 快速判断

- 职责与适用：月牧者（官方译名"牧者"，[NAMES](../../../raw/current/scripts/languages/chinese_s.po#L344304-L344307)）移动／施法时落地的**黏液地面拖尾动画**：联网动画实体，7 支 `trail{n}` 变体，生命周期全由动画段与生成方参数驱动，无玩法组件。[工厂](../../../raw/current/scripts/prefabs/lunar_goop_trail_fx.lua#L46-L81)
- 优先使用：查"地面拖尾／可池化的三段式动画 FX"选本页；生成节流与池在宿主牧者源码（本批未建页）。
- 重要副作用：三段状态机：`SetVariation` 播 `trail{n}_pre` → animover 转主体 `trail{n}`（未显式传循环参数）并按 `duration` 延时 → `OnStartFade` 播 `_pst`（0.5 倍速）→ pst 播完走 `onfinished()` 回调（牧者路径回池）或 Remove。[状态机](../../../raw/current/scripts/prefabs/lunar_goop_trail_fx.lua#L6-L35)
- 修改边界：主模拟驱动状态机（SetPristine 后），客户端经网络同步动画；persists=false；`Dissipate()` 是提前淡出入口——取消 pending 延时直接进 pst，已开始淡出则只置 duration=0。[Dissipate](../../../raw/current/scripts/prefabs/lunar_goop_trail_fx.lua#L37-L44)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `lunar_goop_trail_fx` | fn：Transform＋AnimState（bank/build `lunar_goop_trail`、播 `trail1_pre`、MultColour 透明 0.4、LightOverride 0.1、bloom、世界背景层 SortOrder 3）→ FX／NOCLICK 标签 → SetPristine → 主模拟：trailname/duration 默认值、animover 监听、暴露 SetVariation／Dissipate、persists=false | 单一注册；变体号、缩放、时长全部由 SetVariation 注入，工厂默认 `trail1`＋1 秒。[工厂](../../../raw/current/scripts/prefabs/lunar_goop_trail_fx.lua#L46-L81)、[注册](../../../raw/current/scripts/prefabs/lunar_goop_trail_fx.lua#L83) |

牧者本体官方译名"牧者"（NAMES.LUNAR_GRAZER，[译名](../../../raw/current/scripts/languages/chinese_s.po#L344304-L344307)）；本 FX 注册名无 NAMES 条目，标题"黏液拖尾"为已核验效果描述，非官方译名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 生成者：lunar_grazer SpawnTrail | 从宿主 `inst.trails` 轮换取变体号（1-7），池里有就取出复用（ReturnToScene）否则 SpawnPrefab 并设 `onfinished`；定位后调 `SetVariation(variation, scale, duration)`，记录为 `last_trail` | 主模拟；牧者本体生成 | [SpawnTrail](../../../raw/current/scripts/prefabs/lunar_grazer.lua#L34-L55)、[onfinished 接线](../../../raw/current/scripts/prefabs/lunar_grazer.lua#L511-L512) |
| 状态图触发：SGlunar_grazer | 各状态帧事件调 `inst:SpawnTrail(scale, duration)`：idle 走位 0.2-0.5 秒短拖尾、移动态 10-11 秒长拖尾；中断/退场时对 `last_trail` 调 `Dissipate()` | duration=0 时立即转 pst；参数由各状态写死 | [短拖尾](../../../raw/current/scripts/stategraphs/SGlunar_grazer.lua#L93)、[长拖尾](../../../raw/current/scripts/stategraphs/SGlunar_grazer.lua#L690)、[Dissipate](../../../raw/current/scripts/stategraphs/SGlunar_grazer.lua#L358-L360) |
| 回收：TRAIL_POOL | pst 播完触发 `onfinished`→RecycleTrail：还有被跟踪实体就 RemoveFromScene 入池，否则 Remove；最后一个被跟踪牧者停止跟踪时清空整池 | 主模拟；池是模块级共享列表，以 TRACKED_ENTS 跟踪宿主 | [RecycleTrail](../../../raw/current/scripts/prefabs/lunar_grazer.lua#L22-L32)、[清池](../../../raw/current/scripts/prefabs/lunar_grazer.lua#L61-L69) |
| 资源 | anim/lunar_goop_trail.zip（bank/build 同名，含 `trail1..7` 的 pre/主体/pst 三段） | 声明与使用已核验，包内容未解包 | [资源声明](../../../raw/current/scripts/prefabs/lunar_goop_trail_fx.lua#L1-L4)、[变体名拼装](../../../raw/current/scripts/prefabs/lunar_goop_trail_fx.lua#L26-L35) |

## 典型接入

### 复用"三段式可池化拖尾"

- 前提与选择：SpawnPrefab 后必须调 `SetVariation(rand, scale, duration)`（1≤rand≤7），不调用也会按默认 trail1／duration=1 推进；SetVariation 用于覆写变体、缩放与时长；要拿"淡出完成"时机就先设 `onfinished` 再 SetVariation。
- 生效与撤销：duration 单位秒，pre 播完并开始主体段时就启动 duration 计时，计时到点切 pst；`Dissipate()` 可随时提前淡出，重复调用安全（任务已取消时只改 duration）。
- 原版依据：牧者用帧事件批量铺拖尾并用池复用；没有池的模组让 `onfinished` 留空走默认 Remove 即可。[状态机](../../../raw/current/scripts/prefabs/lunar_goop_trail_fx.lua#L6-L44)、[池](../../../raw/current/scripts/prefabs/lunar_grazer.lua#L22-L32)

```lua
local fx = SpawnPrefab("lunar_goop_trail_fx")
fx.onfinished = function(inst) inst:Remove() end -- 可省：默认路径就是 Remove
fx.Transform:SetPosition(x, 0, z)
fx:SetVariation(3, 1.0, 5)   -- trail3、原尺寸、主体段停留 5 秒
-- 需要提前淡出时：fx:Dissipate()
```

- 调整与验证：淡出段固定 0.5 倍速播放（[OnStartFade](../../../raw/current/scripts/prefabs/lunar_goop_trail_fx.lua#L6-L10)）；`SetVariation` 会取消上一次 pending 延时，复用实体换参数即重开一轮（[SetVariation](../../../raw/current/scripts/prefabs/lunar_goop_trail_fx.lua#L26-L35)）。

## 常见陷阱与限制

- **误区：主体动画会自己循环拖住实体**：`trail{n}` 未显式传循环参数，主体段播一次即停；淡出由 duration 延时任务驱动，不是动画循环。实际画面是否重复未核验，但实体推进不依赖它。[状态机](../../../raw/current/scripts/prefabs/lunar_goop_trail_fx.lua#L12-L24)
- **边界：onfinished 缺省即 Remove，入池必须先设回调**：牧者把回收逻辑全放在 `onfinished`；复用方不设回调时实体直接消失，池协议失效。[OnAnimOver](../../../raw/current/scripts/prefabs/lunar_goop_trail_fx.lua#L17-L23)、[接线](../../../raw/current/scripts/prefabs/lunar_grazer.lua#L511-L512)
- **边界：SetVariation 的 rand 是动画号不是随机种子**：拼成 `trail"..tostring(rand)`；传 0 或大于变体数的值依赖资源里存在对应段，工厂不校验。[拼装](../../../raw/current/scripts/prefabs/lunar_goop_trail_fx.lua#L26-L35)
- **边界：池实体复用会重播 pre**：RemoveFromScene 后 ReturnToScene 再 SetVariation 会重播 pre，但没有重置 OnStartFade 设置的 0.5 倍播放速率；不能保证复用后恢复初次播放速度。对"残留旧画面"有要求时需自己清场。[SpawnTrail](../../../raw/current/scripts/prefabs/lunar_grazer.lua#L36-L43)

## 验证记录

- 源码审阅：2026-10-30，build 26230496；84 行工厂全文、lunar_grazer 的 SpawnTrail／RecycleTrail／池与 onfinished 接线、SGlunar_grazer 的 SpawnTrail 与 Dissipate 调用点已核验；动画包内容、长拖尾（10-11 秒）状态实际名称、晚加入表现未核验。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、label-check 通过（结果见本批交付报告）；本批依据见[[updates/build-26335394/index#wiki-沿用记录]]（批次-2）。

返回 [[reference/prefabs/fx-world-index|特效 Prefab 索引·环境与世界]]、[[reference/prefabs/index|Prefab 索引]]。
