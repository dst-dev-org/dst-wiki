---
title: "远古戍卫塔挥击与砸地特效（vault_pillar_guard_swipe_fx）"
created: "2026-10-29"
updated: "2026-11-08"
type: prefab
summary: "远古戍卫塔（Ruinous Vaults 守卫）的攻击表现：挥击地面动画与砸地双层动画（网络壳＋非专服本地子实体同步动画时间）；无玩法组件，伤害由宿主 SG 的 AOE 结算。"
aliases: ["vault_pillar_guard_swipe_fx", "vault_pillar_guard_smash_fx", "vault_pillar_guard_fx", "远古戍卫塔特效", "戍卫塔砸地特效", "pillar guard fx"]
tags: [prefab, fx, server-client]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/vault_pillar_guard_fx.lua", "stategraphs/SGvault_pillar_guard.lua", "prefabs/vault_pillar_guard.lua", "tuning.lua", "languages/chinese_s.po"]
size_class: simple
reliability: verified
validation: source-reviewed
status: active
---

# 远古戍卫塔挥击与砸地特效（vault_pillar_guard_swipe_fx）

## 快速判断

- 职责与适用：远古戍卫塔（官方译名，[NAMES](../../../raw/current/scripts/languages/chinese_s.po#L350625-L350627)）的攻击表现：`swipe_fx` 是挥击地面动画（attack1／attack2 复用、反手换 atk2），`smash_fx` 是砸地双层动画（网络壳播地面层，非专服再父挂本地子实体播前景层并同步动画时间）。全族无玩法组件，伤害与拆家由宿主 SG 的 AOE 结算。[注册](../../../raw/current/scripts/prefabs/vault_pillar_guard_fx.lua#L115-L116)、[宿主 AOE](../../../raw/current/scripts/stategraphs/SGvault_pillar_guard.lua#L219-L224)
- 优先使用：查"戍卫塔攻击动画／双层砸地表现"选本页；宿主攻击节奏见 SGvault_pillar_guard。
- 重要副作用：仅表现；smash 的本地子实体在客户端用 updatelooper 把动画时间对齐父实体，补偿网络时序差。[同步](../../../raw/current/scripts/prefabs/vault_pillar_guard_fx.lua#L43-L49)
- 修改边界：swipe 的 `atk1` 未显式传循环参数，播完 animover 即 Remove；smash 网络壳同样 animover 自删，子实体随壳移除。[swipe 自删](../../../raw/current/scripts/prefabs/vault_pillar_guard_fx.lua#L37-L40)、[smash 自删](../../../raw/current/scripts/prefabs/vault_pillar_guard_fx.lua#L109-L110)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `vault_pillar_guard_swipe_fx` | swipe_fn：Transform＋AnimState＋Network → FX／NOCLICK、四面向 → bank/build `vault_pillar_guard_fx` 播 `atk1`、OnGround 背景层 SortOrder 3 → SetPristine → 主模拟 Reverse（播 atk2）、persists=false、animover Remove | 与同批熊獾／疯猪挥击同构；无音效。[工厂](../../../raw/current/scripts/prefabs/vault_pillar_guard_fx.lua#L10-L41) |
| `vault_pillar_guard_smash_fx` | smash_fn：Transform＋AnimState＋Network → 播 `atk_smash_ground`（OnGround 背景层）→ 非专服再父挂 smash_CreateFx 本地子实体（播 `atk_smash`、FinalOffset 2、非网络）→ SetPristine → 主模拟 persists=false、animover Remove | 子实体只在非专服生成；客户端（非主机）给子实体加 updatelooper 同步动画时间。[壳](../../../raw/current/scripts/prefabs/vault_pillar_guard_fx.lua#L80-L113)、[子实体](../../../raw/current/scripts/prefabs/vault_pillar_guard_fx.lua#L51-L78) |

两注册名均无 NAMES 中文条目；标题"远古戍卫塔"为官方译名，"砸地"为已核验效果描述。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| swipe 生成：attack1／attack2 | SpawnSwipeFX 父挂宿主、前向偏移 1，attack2 传 reverse；注释：提前 3 帧生成防网络丢帧 | 同态 FrameEvent(29) 的 DoPunchAOE（180° 扇形、半径 4、击退）用宿主 combat＋AOEUtil 结算伤害 | [生成函数](../../../raw/current/scripts/stategraphs/SGvault_pillar_guard.lua#L234-L242)、[attack1](../../../raw/current/scripts/stategraphs/SGvault_pillar_guard.lua#L595-L599)、[attack2 反手](../../../raw/current/scripts/stategraphs/SGvault_pillar_guard.lua#L693-L695)、[DoPunchAOE](../../../raw/current/scripts/stategraphs/SGvault_pillar_guard.lua#L213-L216) |
| smash 生成：attack3 跃砸 | 第 43 帧 SpawnSmashFx（注释：故意晚一帧），第 45-47 帧连续三次 DoSmashAOE（半径 4，Work＋Dig＋Attack＋TossItems） | smash 网络壳父挂宿主；子实体时间同步仅客户端需要 | [跃砸帧事件](../../../raw/current/scripts/stategraphs/SGvault_pillar_guard.lua#L867-L885)、[DoSmashAOE](../../../raw/current/scripts/stategraphs/SGvault_pillar_guard.lua#L219-L224) |
| 子实体时间同步 | smash_PostUpdate 把子实体动画时间设为父实体当前动画时间，然后移除自身 updatelooper（只对齐一次） | 仅非主模拟（纯客户端）分支挂 updatelooper；主机直接自然播放 | [同步逻辑](../../../raw/current/scripts/prefabs/vault_pillar_guard_fx.lua#L43-L49)、[挂载分支](../../../raw/current/scripts/prefabs/vault_pillar_guard_fx.lua#L70-L75) |
| 回收 | 各攻击态退出时 KillSwipeOrSmashFx 兜底 Remove | 主模拟；`persists=false` 不入存档 | [兜底](../../../raw/current/scripts/stategraphs/SGvault_pillar_guard.lua#L250-L256)、[调用点](../../../raw/current/scripts/stategraphs/SGvault_pillar_guard.lua#L621) |
| 伤害参考值 | 宿主 combat 默认伤害 150（TUNING.VAULT_PILLAR_GUARD_DAMAGE），FX 不参与 | 数值属宿主，列作量级参考 | [combat 设值](../../../raw/current/scripts/prefabs/vault_pillar_guard.lua#L590)、[数值](../../../raw/current/scripts/tuning.lua#L9518) |
| 依赖声明 | vault_pillar_guard prefab 依赖表声明两注册名 | 声明保证资源加载，不代表即时生成 | [依赖表](../../../raw/current/scripts/prefabs/vault_pillar_guard.lua#L30-L31) |

## 典型接入

"前景＋地面双层砸地"表现可仿 smash 结构：网络壳播贴地层，非专服父挂非网络子实体播前景层；纯客户端需要表现与宿主动画对齐时，用 updatelooper 首帧同步 `SetTime(父动画时间)` 后即拆。[双层结构](../../../raw/current/scripts/prefabs/vault_pillar_guard_fx.lua#L99-L101)、[同步](../../../raw/current/scripts/prefabs/vault_pillar_guard_fx.lua#L43-L49)

## 常见陷阱与限制

- **误区：FX 参与伤害或拆家**：Work＋Dig＋Attack＋TossItems 全在宿主 SG 的 DoSmashAOE；FX 无 workable／combat。[AOE](../../../raw/current/scripts/stategraphs/SGvault_pillar_guard.lua#L219-L224)
- **边界：子实体的同步只对齐一次**：smash_PostUpdate 设完时间就 RemoveComponent("updatelooper")，不是逐帧跟随；宿主动画中途被打断时子实体不会再次对齐。[单次同步](../../../raw/current/scripts/prefabs/vault_pillar_guard_fx.lua#L43-L49)
- **边界：专服没有前景层**：子实体生成被 `TheNet:IsDedicated()` 挡住，专用服务器只有地面层网络壳（且不渲染）；这是有意的负载取舍。[专服分支](../../../raw/current/scripts/prefabs/vault_pillar_guard_fx.lua#L99-L101)
- **边界：Reverse 必须成对提供**：attack2 传 reverse 会直接调 FX 的 Reverse；换用自定义 FX 时需保留 atk1/atk2 约定。[调用](../../../raw/current/scripts/stategraphs/SGvault_pillar_guard.lua#L236-L241)

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496；117 行工厂全文（两工厂、子实体同步）、SGvault_pillar_guard 三攻击态生成点与 AOE 参数、宿主依赖表、tuning 数值、译名检索已核验；动画实际帧数、spin／jump 等其余攻击是否复用本 FX 未核验（本文件内仅 attack1／2／3 三处生成点）。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、label-check 通过（结果见本批交付报告）。

返回 [[reference/prefabs/fx-creature-index|生物、Boss 与遭遇演出]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-2）。
- 2026-11-08 build 26474873：本页随版本迁移升版（锚点迁移 1 处），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-12）。
