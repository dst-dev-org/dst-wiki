---
title: "树木石化特效（petrified_trunk_break_fx）"
created: "2026-10-29"
updated: "2026-11-01"
type: prefab
summary: "树木石化与石化树桩碎裂的五注册特效族：petrified_trunk_break_fx 桩碎裂，petrified_tree_fx_short/normal/tall/old 四段石化落尘；代理＋本地动画，经 _darken netvar 继承树木染色。"
aliases: ["petrified_trunk_break_fx", "petrified_tree_fx_short", "petrified_tree_fx_normal", "petrified_tree_fx_tall", "petrified_tree_fx_old", "petrify_fx", "石化特效", "石化树碎裂", "petrified tree fx"]
tags: [prefab, fx, server-client]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/petrify_fx.lua", "prefabs/evergreens.lua", "components/petrifiable.lua", "components/forestpetrification.lua", "prefabs/gargoyles.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 树木石化特效（petrified_trunk_break_fx）

## 快速判断

- 职责与适用：常青树／尖刺树石化变身与石化树桩破碎的表现层：四个 `petrified_tree_fx_*` 是石化落尘（树变岩石时扬起），`petrified_trunk_break_fx` 是树桩被石化事件命中时的碎裂。无玩法组件；染色经 `_darken` netvar 从树木继承。**文件名 petrify_fx.lua 不是注册名**。[五注册](../../../raw/current/scripts/prefabs/petrify_fx.lua#L83-L87)
- 优先使用：查"石化变身特效／岩石散落"选本页；触发链与生成者见关联机制导航。
- 重要副作用：纯表现；主模拟仅存活 1 秒，本地动画实体非专服 0 帧延迟创建、animover 自删；本地实体带音效（石化落地声）。[代理工厂](../../../raw/current/scripts/prefabs/petrify_fx.lua#L49-L81)
- 修改边界：代理仅 Transform＋Network；`_darken` 是 net_tinybyte（0-7 灰度），消费者用 `InheritColour` 传 RGB。`persists=false`。[染色编码](../../../raw/current/scripts/prefabs/petrify_fx.lua#L11-L18)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `petrified_trunk_break_fx` | makefx("petrified_trunk_break","break_apart","post_stump")：Transform＋Network → FX 标签 → 非专服 0 帧后 PlayFX：bank/build `petrified_trunk_break` 播 `break_apart`、MultColour 由 `_darken` 反序列化、FinalOffset 1、播 `petrified/post_stump` 音、animover 自删 → SetPristine → 客户端早退 → 主模拟暴露 InheritColour、1 秒后 Remove | 树桩路径专用；消费者按树桩尺寸再缩放 0.75／1.2。[注册](../../../raw/current/scripts/prefabs/petrify_fx.lua#L83)、[消费缩放](../../../raw/current/scripts/prefabs/evergreens.lua#L780-L787) |
| `petrified_tree_fx_short`／`_normal`／`_tall`／`_old` | makefx("petrified_tree_fx","rock_scatter<后缀>","post") 循环注册：同框架，bank/build `petrified_tree_fx`、anim `rock_scatter_short/normal/tall/old`、音效 `petrified/post` | 与树成长期一一对应，由 evergreens 按阶段下标选择；后缀拼写与树阶段名不完全对齐（normal 对 2 阶段）。[循环注册](../../../raw/current/scripts/prefabs/petrify_fx.lua#L84-L86)、[阶段表](../../../raw/current/scripts/prefabs/evergreens.lua#L730-L735) |

本族均无 NAMES 中文条目；标题"树木石化特效"为已核验效果描述，非官方译名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 生成者：evergreens dopetrify | 树播完石化预动画后替换为岩石，非 instant 时于原位 Spawn 对应阶段 `petrified_tree_fx_*` 并 `InheritColour(r,g,b)` 继承染色 | worldgen 填充（POPULATING）走 instant 不出特效 | [dopetrify](../../../raw/current/scripts/prefabs/evergreens.lua#L737-L754)、[fx 生成](../../../raw/current/scripts/prefabs/evergreens.lua#L749-L752) |
| 生成者：evergreens onpetrifiedfn_evergreen | 树是树桩或无石化动画的阶段时，直接 Spawn `petrified_trunk_break_fx`：按 stump_short／stump_tall 缩放 0.75／1.2 并继承染色 | 由 petrifiable 组件回调触发；非 POPULATING 才出特效 | [树桩分支](../../../raw/current/scripts/prefabs/evergreens.lua#L763-L788)、[组件装配](../../../raw/current/scripts/prefabs/evergreens.lua#L918-L919) |
| [[reference/components/petrifiable#关键接口]] | 触发源头：Petrify 链式传导（半径 8、延迟 0.6-1.8 秒）后执行 onPetrifiedFn | 仅 evergreens 系树木装配；链式传导让石化成片发生 | [链式传导](../../../raw/current/scripts/components/petrifiable.lua#L1-L51) |
| [[reference/components/forestpetrification#关键接口]] | 世界组件批量把注册树木推向石化（剩余树量低于阈值时），最终落到 dopetrify／树桩分支 | 仅主模拟世界组件 | [Petrify 调用](../../../raw/current/scripts/components/forestpetrification.lua#L154-L162)、[阈值](../../../raw/current/scripts/components/forestpetrification.lua#L17-L23) |
| 资源：petrified_trunk_break.zip／petrified_tree_fx.zip | 两 build 各自 bank 同名；本地实体带 SoundEmitter 播石化音 | 与 gargoyle（石像）系列**不是同一工厂**：石像 FX 用 sculpture_fx bank 自带注册 | [石像独立工厂](../../../raw/current/scripts/prefabs/gargoyles.lua#L240-L246) |

## 典型接入

复用"继承染色的落尘代理"：生成后调 `fx:InheritColour(r, g, b)` 把源实体当前染色折算进 `_darken`，本地动画会自动套灰度；不调则用默认 7（不压暗）。染色编码把 RGB 均值编码到 0-7，再解码为 0.5~1 的灰度（更暗的输入会被截到 0.5），只能近似灰度，彩色染色会失真。[编码](../../../raw/current/scripts/prefabs/petrify_fx.lua#L11-L13)

```lua
local r, g, b = sourcetree.AnimState:GetMultColour()
local fx = SpawnPrefab("petrified_tree_fx_normal")
fx.Transform:SetPosition(x, y, z)
fx:InheritColour(r, g, b)
```

- 调整与验证：动画／音效名在注册处传参（[L83-L86](../../../raw/current/scripts/prefabs/petrify_fx.lua#L83-L86)）；主模拟 1 秒兜底删除独立于本地动画时长。[兜底](../../../raw/current/scripts/prefabs/petrify_fx.lua#L76-L77)

## 常见陷阱与限制

- **误区：census 名 petrify_fx 是注册名**：本文件实际注册 `petrified_trunk_break_fx` 与四个 `petrified_tree_fx_*`，`petrify_fx` 只是文件名／模块名，不能 Spawn。[返回](../../../raw/current/scripts/prefabs/petrify_fx.lua#L83-L87)
- **边界：石像（gargoyle）石化不走本页**：gargoyles.lua 自带同构代理工厂（sculpture_fx bank、petrified_tree_fx 仅作 build），其 `gargoyle_*` 系列是独立注册；改石化石像表现别改错文件。[独立工厂](../../../raw/current/scripts/prefabs/gargoyles.lua#L240-L246)
- **边界：视觉与音效专服均无**：本地动画实体由非专服创建，专服只有 1 秒空代理；音效也只在本地实体上播放。[分支](../../../raw/current/scripts/prefabs/petrify_fx.lua#L58-L63)
- **边界：阶段名错位**：STAGE_PETRIFY_FX 下标 1-4 依次是 short／normal／tall／old，而树阶段动画名是 petrify_short／petrify_normal／petrify_tall／petrify_old；改动成长阶段数时两表要同步。[两表](../../../raw/current/scripts/prefabs/evergreens.lua#L730-L735)、[阶段动画](../../../raw/current/scripts/prefabs/evergreens.lua#L756-L761)
- **边界：worldgen 无特效**：POPULATING 期间石化走 instant 分支，只换岩石不出特效；自建世界填充逻辑可复用该约定。[instant](../../../raw/current/scripts/prefabs/evergreens.lua#L766-L768)

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496；88 行工厂全文（五注册、代理＋本地双层、染色编码）、evergreens 两生成点与阶段表、petrifiable 链式传导、forestpetrification 阈值调用、译名检索已核验；gargoyle 完整生成链、石化事件的世界触发节奏、本地动画实际帧数未核验。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、label-check 通过（结果见本批交付报告）。

返回 [[reference/prefabs/fx-creature-index|特效 Prefab 索引·生物与Boss]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页 sources 所引文件原始字节本轮全部未变（哈希等价批量沿用，无语义修订），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-3）。
