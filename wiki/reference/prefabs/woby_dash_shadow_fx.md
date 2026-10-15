---
title: "沃比暗影冲刺特效（woby_dash_shadow_fx）"
created: "2026-10-29"
updated: "2026-11-01"
type: prefab
summary: "沃比（Walter 技能）暗影冲刺的传送残影与剪影：起点／落点大影与跟随途中小影两注册，无玩法组件，传送与无敌帧由玩家 SG 状态完成。"
aliases: ["woby_dash_shadow_fx", "woby_dash_silhouette_fx", "沃比暗影冲刺特效", "woby 传送残影", "shadow dash fx"]
tags: [prefab, fx, server-client]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/woby_shadow_fx.lua", "stategraphs/SGwilson.lua", "prefabs/wobybig.lua", "tuning.lua", "languages/chinese_s.po"]
size_class: simple
reliability: verified
validation: source-reviewed
status: active
---

# 沃比暗影冲刺特效（woby_dash_shadow_fx）

## 快速判断

- 职责与适用：Walter 骑沃比（官方译名"沃比"，[NAMES](../../../raw/current/scripts/languages/chinese_s.po#L351729-L351732)）触发"暗影沃比"（官方技能名，[SKILLTREE](../../../raw/current/scripts/languages/chinese_s.po#L366169-L366172)）冲刺时的传送残影组：起点／落点的大影、途中跟随的小影（`woby_dash_shadow_fx`）与玩家身上的剪影（`woby_dash_silhouette_fx`）；不是伤害来源。[工厂](../../../raw/current/scripts/prefabs/woby_shadow_fx.lua#L34-L61)
- 优先使用：想要"传送类残影＋服务端向宿主追踪的 FX"参照选本页；传送落点与无敌帧见下文玩家状态。
- 重要副作用：仅途中小影通过 `SetFxOwner` 挂 OnWallUpdate 每帧向宿主移动 `dt×3`（考虑服务器暂停与时间缩放）；起点／落点影保持原地不追踪。[追踪](../../../raw/current/scripts/prefabs/woby_shadow_fx.lua#L6-L32)
- 修改边界：两个注册都是有 Network 的联网实体，追踪逻辑只在主模拟（SetFxOwner 定义于主模拟段），客户端跟随靠复制；FX 自身无伤害／冻结等玩法组件。[构造](../../../raw/current/scripts/prefabs/woby_shadow_fx.lua#L49-L60)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `woby_dash_shadow_fx` | fn：Transform＋AnimState＋SoundEmitter＋Network → bank/build `woby_shadow_fx` 播 `woby_teleport_fx_big` → FX／NOCLICK → SetPristine → 客户端早退 → 主模拟 animover 自删、`persists=false`、暴露 SetFxOwner | 单注册可被 SG 播 `woby_teleport_fx_small1/2` 覆盖动画；SoundEmitter 的音效由 SG 状态调用。[工厂](../../../raw/current/scripts/prefabs/woby_shadow_fx.lua#L34-L61)、[注册](../../../raw/current/scripts/prefabs/woby_shadow_fx.lua#L89) |
| `woby_dash_silhouette_fx` | silhouttefn：Transform＋AnimState＋Network（无 SoundEmitter）→ 六面向、播 `tp_silhoutte` → FX 标签 → SetPristine → 客户端早退 → 主模拟仅 `persists=false` | 作为玩家子实体使用，由状态 onexit 显式 Remove，不监听 animover。[工厂](../../../raw/current/scripts/prefabs/woby_shadow_fx.lua#L63-L87)、[注册](../../../raw/current/scripts/prefabs/woby_shadow_fx.lua#L90) |

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 生成者：SGwilson "dash_woby_shadow" | onenter：玩家 alpha 置 0＋无敌帧＋关操作、剪影挂到玩家身上、起点大影 SetFrame(3) 播 `shadow_dash_out`；第 2 帧选落点传送后生成落点大影播 `shadow_dash_in` | 进入条件：`dash_woby_pre` 第 3 帧骑乘沃比＋`walter_woby_shadow` 技能激活＋ShouldShadowDash() 指令 | [进入门槛](../../../raw/current/scripts/stategraphs/SGwilson.lua#L25289-L25294)、[起点影与剪影](../../../raw/current/scripts/stategraphs/SGwilson.lua#L25379-L25387)、[落点影](../../../raw/current/scripts/stategraphs/SGwilson.lua#L25451-L25454) |
| 途中小影 | 第 2 帧另生成一个小影（`woby_teleport_fx_small1/2` 随机）并 `SetFxOwner(inst)` 追踪玩家 | 仅这一枚追踪；同帧还执行 7-12.5 距离落点搜索（失败回退 6.5-0.5）并 Teleport | [小影与落点搜索](../../../raw/current/scripts/stategraphs/SGwilson.lua#L25397-L25448) |
| 传送与无敌帧 | 与 FX 无读写关系：状态期间 `SetInvincible(true)`、playercontroller 禁用、物理收起；onexit 恢复并 Remove 剪影 | 普通冲刺走 `dash_woby` 状态，不生成任何残影 | [状态](../../../raw/current/scripts/stategraphs/SGwilson.lua#L25362-L25487)、[恢复](../../../raw/current/scripts/stategraphs/SGwilson.lua#L25461-L25474) |
| 消耗：wobybig OnDash | 坐骑监听 `ondash_woby`（shadow 事件带 `shadow=true`）扣饥饿：暗影冲刺 4、普通 2，耐力修正 0.8 | 坐骑本身无 FX 逻辑 | [消耗](../../../raw/current/scripts/prefabs/wobybig.lua#L485-L489)、[tuning](../../../raw/current/scripts/tuning.lua#L7343-L7354)、[依赖](../../../raw/current/scripts/prefabs/wobybig.lua#L39-L40) |
| 资源：woby_shadow_fx.zip | bank／build 同名；`woby_teleport_fx_big` 未显式传循环参数，实际是否重复未核验 | [assets](../../../raw/current/scripts/prefabs/woby_shadow_fx.lua#L1-L4) | 内容未解包 |

## 常见陷阱与限制

- **误区：残影造成传送**：传送（Teleport）、无敌帧与操作禁用全部在玩家状态里完成，FX 不读写玩家组件；复用残影不会产生位移。[状态](../../../raw/current/scripts/stategraphs/SGwilson.lua#L25420-L25448)
- **误区：SetFxOwner 适用于所有残影**：SG 对起点／落点影显式注释"don't track owner, keep fx stationary"，只有途中小影追踪；对静止影调 SetFxOwner 会改变原版表现。[注释](../../../raw/current/scripts/stategraphs/SGwilson.lua#L25385-L25386)
- **边界：追踪在服务端步进**：OnWallUpdate 调 `TheSim:GetTimeScale()` 与服务器暂停判断，仅主模拟注册；专服照常步进，客户端不重复模拟。[追踪实现](../../../raw/current/scripts/prefabs/woby_shadow_fx.lua#L6-L26)

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496；91 行工厂全文、两注册、SGwilson dash_woby_pre／dash_woby_shadow 全程（门槛、三枚残影、落点搜索、传送与恢复）、坐骑饥饿消耗、译名检索已核验；`tp_silhoutte` 动画时长、晚加入表现、落点搜索的地图细节未核验。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、label-check 通过（结果见本批交付报告）。

返回 [[reference/prefabs/fx-creature-index|特效 Prefab 索引·生物与Boss]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-1）。
