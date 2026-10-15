---
title: "恐怖之眼降临传送门（eyeofterror_arrive_fx）"
created: "2026-10-29"
updated: "2026-11-01"
type: prefab
summary: "恐怖之眼与双子魔眼入场时的传送门特效：arrive 动画＋半径 0→5 的脉冲灯光（netvar 驱动逐帧插值）＋入场音，120 帧后自动收灯、animover 自删。"
aliases: ["eyeofterror_arrive_fx", "恐怖之眼传送门", "降临传送门", "眼睛传送门特效", "eye of terror portal fx"]
tags: [prefab, fx, server-client]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/eyeofterror_arrive_fx.lua", "stategraphs/SGeyeofterror.lua", "prefabs/eyeofterror.lua", "languages/chinese_s.po"]
size_class: simple
reliability: verified
validation: source-reviewed
status: active
---

# 恐怖之眼降临传送门（eyeofterror_arrive_fx）

## 快速判断

- 职责与适用：恐怖之眼（[NAMES](../../../raw/current/scripts/languages/chinese_s.po#L341914-L341917)）与双子魔眼（激光眼／魔焰眼，[1](../../../raw/current/scripts/languages/chinese_s.po#L350505-L350507)、[2](../../../raw/current/scripts/languages/chinese_s.po#L350510-L350512)）共用 SG 的 arrive 状态入场时，落在 Boss 位置的传送门光效＋灯光脉冲。无伤害组件。[工厂](../../../raw/current/scripts/prefabs/eyeofterror_arrive_fx.lua#L77-L121)
- 优先使用：查"Boss 入场传送门／脉冲灯光"选本页；生成入口见关联机制导航。
- 重要副作用：灯光半径经 `_lightframe`／`_islighton` 两个 netvar 驱动逐帧插值（0↔5），主模拟 120 帧后关灯，`arrive` 动画播完 animover 自删；`arrive_portal` 音效由主模拟 0 帧任务播放。[灯光开关](../../../raw/current/scripts/prefabs/eyeofterror_arrive_fx.lua#L101-L105)、[收尾](../../../raw/current/scripts/prefabs/eyeofterror_arrive_fx.lua#L116-L118)
- 修改边界：灯光半径插值任务双端各跑一份（服务端直接调、客户端监听 dirty）；色彩呼吸 tween 只在非专服。`persists=false` 不入存档。[OnLightDirty](../../../raw/current/scripts/prefabs/eyeofterror_arrive_fx.lua#L48-L65)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `eyeofterror_arrive_fx` | fn：Transform＋AnimState＋SoundEmitter＋Light＋Network → Light 初始化（radius 0、intensity 0.8、falloff 0.7、`EnableClientModulation(true)`）→ bank/build `eyeofterror_portal` 播 `arrive`、bloom、FinalOffset -1、FX 标签 → `net_smallbyte _lightframe`＋`net_bool _islighton`（dirty 事件 `lightdirty`）、初始置亮并直接调 OnLightDirty → SetPristine → 客户端监听 lightdirty 后早退 → 主模拟 0 帧播音、120 帧后 TurnLightOff、animover Remove → `persists=false` | 单注册；默认亮起，不需要外部再调接口。[工厂](../../../raw/current/scripts/prefabs/eyeofterror_arrive_fx.lua#L77-L121)、[注册](../../../raw/current/scripts/prefabs/eyeofterror_arrive_fx.lua#L123) |

灯光行为：亮起时 `_lightframe` 每帧 +1 到上限 10（半径＝5×帧/10），熄灭时反向减到 0；颜色按每帧正弦三通道呼吸（非专服 tween 任务）。[插值](../../../raw/current/scripts/prefabs/eyeofterror_arrive_fx.lua#L13-L31)、[颜色](../../../raw/current/scripts/prefabs/eyeofterror_arrive_fx.lua#L33-L46)

本 FX 无 NAMES 中文条目；标题"降临传送门"为已核验效果描述，非官方译名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 生成者：SGeyeofterror arrive 状态 | 恐怖之眼与双子魔眼进入 `arrive` 状态时生成于 Boss 位置；Boss 该状态 122 帧后才解除无敌，特效覆盖入场过程 | 双子1 直接 GoToState("arrive")；双子2 走 `arrive_delay` 10 帧后 push "arrive" 事件再进状态 | [生成](../../../raw/current/scripts/stategraphs/SGeyeofterror.lua#L810-L821)、[事件入口](../../../raw/current/scripts/stategraphs/SGeyeofterror.lua#L95-L96)、[双子生成](../../../raw/current/scripts/prefabs/eyeofterror.lua#L795-L810) |
| netvar／dirty：`_lightframe`／`_islighton` | 主模拟改值后双端经 `lightdirty` 启动／续跑每帧插值任务；任务完成后自动 Cancel | 客户端只有监听 dirty 的 OnLightDirty；颜色 tween 任务仅非专服 | [声明](../../../raw/current/scripts/prefabs/eyeofterror_arrive_fx.lua#L101-L105)、[客户端分支](../../../raw/current/scripts/prefabs/eyeofterror_arrive_fx.lua#L107-L112) |
| 资源：eyeofterror_portal.zip | bank/build 同名，仅 `arrive` 一段动画（未显式传循环参数） | bloom 走 `shaders/anim.ksh`；音效 `terraria1/eyeofterror/arrive_portal`（Terraria 联动音库） | [动画与 bloom](../../../raw/current/scripts/prefabs/eyeofterror_arrive_fx.lua#L93-L97)、[音效](../../../raw/current/scripts/prefabs/eyeofterror_arrive_fx.lua#L73-L75) |

## 典型接入

复用为"从天而降／传送入场"的光门：生成即自动亮灯、播动画，无需外部参数；需要提前熄灯可在自有工厂复用局部函数 `TurnLightOff` 的逻辑（主模拟设 `_islighton=false` 并调用 OnLightDirty）；原实体没有暴露 TurnLightOff 方法，动画结束自删不可阻拦，除非移除 animover 监听。[工厂](../../../raw/current/scripts/prefabs/eyeofterror_arrive_fx.lua#L77-L121)

```lua
local fx = SpawnPrefab("eyeofterror_arrive_fx")
fx.Transform:SetPosition(x, y, z)
```

- 调整与验证：灯光强度／半径常量在文件头（[MAX_LIGHT_RADIUS 5](../../../raw/current/scripts/prefabs/eyeofterror_arrive_fx.lua#L9-L10)）；总寿命由 `arrive` 动画长度决定，与主模拟 120 帧关灯相互独立。

## 常见陷阱与限制

- **边界：灯光同步粒度是帧**：`_lightframe` 是 net_smallbyte，主模拟每次 dirty 后双端各自跑每帧任务插值到目标；晚加入玩家从当前值继续而非重放全过程（未逐项核验首帧表现）。[插值任务](../../../raw/current/scripts/prefabs/eyeofterror_arrive_fx.lua#L48-L65)
- **边界：颜色呼吸只在客户端**：OnUpdateLightColour 的正弦调色任务仅在非专服创建；专服无视觉但灯光半径插值仍在跑。[分支](../../../raw/current/scripts/prefabs/eyeofterror_arrive_fx.lua#L54-L64)
- **边界：寿命由动画决定**：主模拟仅监听 animover，不设兜底 Remove 任务；若动画资源缺失会留下光柱实体（`persists=false`，不随存档恢复）。[收尾](../../../raw/current/scripts/prefabs/eyeofterror_arrive_fx.lua#L116-L118)
- **误区：特效是传送门本体**：它只是 Boss 入场的覆盖特效，不承载 Boss 出现逻辑；Boss 出现由 SG 状态与 twinmanager 驱动（完整召唤链未在本页核验）。[生成点](../../../raw/current/scripts/stategraphs/SGeyeofterror.lua#L813-L818)

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496；124 行工厂全文（灯光插值／颜色 tween／双端分界／收尾）、SGeyeofterror arrive 与 arrive_delay 状态、双子生成调用、译名检索已核验；恐怖之眼本体完整召唤链（Terrarium 侧）、晚加入首帧表现未核验。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、label-check 通过（结果见本批交付报告）。

返回 [[reference/prefabs/fx-creature-index|生物、Boss 与遭遇演出]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-2）。
