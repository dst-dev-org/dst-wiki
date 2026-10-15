---
title: "独眼巨鹿冰刺（deerclops_icespike_fx）"
created: "2026-10-29"
updated: "2026-11-01"
type: prefab
summary: "独眼巨鹿攻击的地面冰刺动画：按宿主朝向四面向、对象池复用，无玩法组件，伤害由宿主 SG 的 AOE 单独结算。"
aliases: ["deerclops_icespike_fx", "独眼巨鹿冰刺", "冰刺特效", "ice spike"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/deerclops_icespike_fx.lua", "stategraphs/SGdeerclops.lua", "prefabs/deerclops.lua", "languages/chinese_s.po"]
size_class: simple
reliability: verified
validation: source-reviewed
status: active
---

# 独眼巨鹿冰刺（deerclops_icespike_fx）

## 快速判断

- 职责与适用：独眼巨鹿（官方译名，[NAMES](../../../raw/current/scripts/languages/chinese_s.po#L341480-L341482)）攻击时在目标区域刺出的地面冰刺动画，`spike1-4` 与更大号的 `spike_big1-4` 由生成方按落点信息选择；不是会攻击的实体。[动画选择](../../../raw/current/scripts/prefabs/deerclops_icespike_fx.lua#L19-L22)
- 优先使用：想要"地面刺出物＋对象池复用＋四面向"参照选本页；落点分布与伤害链路见下文生成者条目。
- 重要副作用：自身无组件、无伤害；实际伤害与冻结击退由宿主 SG 的 `DoIceSpikeAOE` 用独眼巨鹿自己的 combat 结算，冰刺只是同帧的表现层。[AOE 结算](../../../raw/current/scripts/stategraphs/SGdeerclops.lua#L41-L67)
- 修改边界：联网实体（Transform＋AnimState＋SoundEmitter＋Network），动画选择与回池监听只在主模拟；客户端副本接受服务端场景进出／移除与动画同步，本地不参与池决策。[构造](../../../raw/current/scripts/prefabs/deerclops_icespike_fx.lua#L24-L53)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `deerclops_icespike_fx` | fn：Transform＋AnimState＋SoundEmitter＋Network → bank/build `deerclops_icespike` 播 `spike1` → FX／NOCLICK 标签、`SetFourFaced` → SetPristine → 客户端早退 → 主模拟监听 `animover` 回池、`persists=false`、暴露 SetFXOwner／RestartFX | 单注册；`spike_big` 变体只是动画名差异，不改实体结构。[工厂](../../../raw/current/scripts/prefabs/deerclops_icespike_fx.lua#L24-L53)、[注册](../../../raw/current/scripts/prefabs/deerclops_icespike_fx.lua#L55) |

冰刺无直接 NAMES 翻译条目；标题中文部分是已核验的效果描述，非官方译名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 生成者：SGdeerclops "attack" | DoSpawnIceSpike 从宿主 `icespike_pool` 取出或生成，按落点 `info.big/info.variation` 重播动画、按 `rot` 旋转，`shouldsfx` 时在冰刺自己的 SoundEmitter 上播 `ice_small` | `attack` 状态（SG 为 deerclops 与 mutateddeerclops 共用）timeline 调 SpawnIceFx 铺落点 | [DoSpawnIceSpike](../../../raw/current/scripts/stategraphs/SGdeerclops.lua#L72-L88)、[铺落点](../../../raw/current/scripts/stategraphs/SGdeerclops.lua#L891)、[状态](../../../raw/current/scripts/stategraphs/SGdeerclops.lua#L855-L891) |
| 外部伤害：DoIceSpikeAOE | 与冰刺无关的伤害链：以落点为圆心 `ICESPIKE_RADIUS(1)+3` 找 `_combat` 实体，宿主 `combat:DoAttack` 结算；已冻结目标（hasknockback）另推 `knockback` 事件 | 每根冰刺独立延迟结算（`hitdelay`）；同一 `data.targets` 表防重复命中 | [AOE](../../../raw/current/scripts/stategraphs/SGdeerclops.lua#L41-L67)、[击退](../../../raw/current/scripts/stategraphs/SGdeerclops.lua#L50-L58) |
| 对象池：icespike_pool | `animover` 时回 `owner.icespike_pool`（RestartFX 里 ReturnToScene 复活），无池才 Remove | 池挂在宿主身上，`OnRemoveEntity` 清池；仅主模拟 | [回池](../../../raw/current/scripts/prefabs/deerclops_icespike_fx.lua#L6-L13)、[宿主池](../../../raw/current/scripts/prefabs/deerclops.lua#L415) |
| 资源：deerclops_icespike.zip | bank／build 同名；循环参数未显式传，动画播完靠 animover 事件驱动 | [assets](../../../raw/current/scripts/prefabs/deerclops_icespike_fx.lua#L1-L4) | 声明即全部依赖，内容未解包 |

## 典型接入

### 复用"池化刺出物"模式

- 前提与选择：宿主持有池表并在移除时清空；FX 用 `SetFXOwner` 记宿主，`RestartFX(big, variation)` 复位动画后 ReturnToScene。
- 生效与撤销：`animover` 是唯一回收信号；宿主侧不主动 Remove 在场冰刺，靠动画自然播完。同一帧多根冰刺由宿主的 `data.targets` 共享防重复，不由 FX 判定。
- 原版依据：生成时落点与旋转由 SG 计算（[DoSpawnIceSpike](../../../raw/current/scripts/stategraphs/SGdeerclops.lua#L72-L88)），FX 只接受结果。

## 常见陷阱与限制

- **误区：冰刺本身造成伤害或击退**：工厂无 combat／freezable 等玩法组件；击退事件由宿主 SG 在 AOE 里对已冻结目标推送，与 FX 实体无关。[结算](../../../raw/current/scripts/stategraphs/SGdeerclops.lua#L54-L58)
- **边界：客户端副本不回池**：`animover` 监听与池逻辑都在主模拟段；客户端接受服务端 RemoveFromScene／ReturnToScene／Remove 对应状态同步，不在本地维护池。[端分界](../../../raw/current/scripts/prefabs/deerclops_icespike_fx.lua#L43-L50)

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496；56 行工厂全文、单注册、SGdeerclops 生成与 AOE 结算、宿主池初始化已核验；`spike_big` 的落点选择规则（SpawnIceFx 的 info 表分布）只确认存在、未逐项展开；晚加入／客户端动画同步未核验。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、label-check 通过（结果见本批交付报告）。

返回 [[reference/prefabs/fx-creature-index|特效 Prefab 索引·生物与Boss]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-2）。
