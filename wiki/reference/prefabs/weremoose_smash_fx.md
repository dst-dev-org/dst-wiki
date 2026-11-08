---
title: "伍迪麋鹿形态重击特效（weremoose_smash_fx）"
created: "2026-10-29"
updated: "2026-11-08"
type: prefab
summary: "伍迪麋鹿形态三连击终结拍地动画：前景／背景双层、本地预测副本隐藏网络副本，无玩法组件，伤害由宿主攻击状态结算。"
aliases: ["weremoose_smash_fx", "麋鹿重击特效", "人形麋鹿拍地特效", "moose slam fx"]
tags: [prefab, fx, server-client]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/weremoose_smash_fx.lua", "stategraphs/SGwilson.lua", "stategraphs/SGwilson_client.lua", "prefabs/skilltree_woodie.lua", "prefabs/woodie.lua", "tuning.lua", "languages/chinese_s.po"]
size_class: simple
reliability: verified
validation: source-reviewed
status: active
---

# 伍迪麋鹿形态重击特效（weremoose_smash_fx）

## 快速判断

- 职责与适用：Woodie（官方译名"伍迪"，[NAMES](../../../raw/current/scripts/languages/chinese_s.po#L351765-L351767)）麋鹿形态（"Weremoose"无直接 NAMES 条目，标题中文为已核验效果描述，非官方译名）三连击终结拍地的冲击波动画，bank `weremoose`／build `weremoose_build` 播 `moose_slam_fx_front`；不是伤害来源。[工厂](../../../raw/current/scripts/prefabs/weremoose_smash_fx.lua#L37-L74)
- 优先使用：想要"宿主攻击帧驱动的双层次冲击波＋本地预测去重"参照选本页；伤害链路见下文宿主条目。
- 重要副作用：`_owner` netvar 在客户端触发隐藏——本地玩家预测已自建同名 FX 时隐藏服务器副本，避免双影。[预测去重](../../../raw/current/scripts/prefabs/weremoose_smash_fx.lua#L30-L35)
- 修改边界：联网实体；背景层副本仅非专服生成；animover 自删只在主模拟。[构造](../../../raw/current/scripts/prefabs/weremoose_smash_fx.lua#L55-L73)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `weremoose_smash_fx` | fn：Transform＋AnimState＋Network → bank `weremoose`／build `weremoose_build` 播 `moose_slam_fx_front`（FinalOffset 3）、四面向 → FX 标签、`_owner` net_entity → 非专服 DoTaskInTime(0) 生成背景层子实体 → SetPristine → 客户端监听 ownerdirty → 主模拟 animover 自删、`persists=false` | 单注册；背景层子实体为无 Network 本地实体，播 `moose_slam_fx_back`（FinalOffset -1）并对齐首帧。[工厂](../../../raw/current/scripts/prefabs/weremoose_smash_fx.lua#L37-L74)、[背景层](../../../raw/current/scripts/prefabs/weremoose_smash_fx.lua#L6-L28) |

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 生成者：SGwilson "attack" | 第 7 帧在宿主脚下生成并按宿主朝向旋转，`_owner:set(inst)` | 仅 `ismoosesmash` 分支：麋鹿形态第三段 `punch_b` 且有 `weremoosecombo` 标签时播 `moose_slam` | [生成](../../../raw/current/scripts/stategraphs/SGwilson.lua#L11022-L11029)、[组合拳门槛](../../../raw/current/scripts/stategraphs/SGwilson.lua#L10940-L10956) |
| 生成者：SGwilson_client "attack" | 客户端第 7 帧本地生成同名 FX（预测／表现路径），不设 `_owner` | 本地玩家移动预测时执行；远端玩家的客户端表现未核验 | [客户端生成](../../../raw/current/scripts/stategraphs/SGwilson_client.lua#L5263-L5270) |
| 宿主伤害：AOE 结算 | 与 FX 无读写关系：宿主以自身前方 1 距离、半径 2（+3 padding）查 `_combat`（排除 wall/companion 等），临时把 defaultdamage 设为 `SKILLS.WOODIE.MOOSE_SMASH_DAMAGE` 逐个 `DoAttack` | 目标带 planarentity 时临时 AddBonus 位面 80 再移除；命中后恢复 `WEREMOOSE_DAMAGE` | [AOE](../../../raw/current/scripts/stategraphs/SGwilson.lua#L11031-L11059)、[伤害常量](../../../raw/current/scripts/tuning.lua#L7119-L7120) |
| 技能门槛：weremoosecombo | Woodie 技能树 curse 组技能激活时 AddTag，决定第三段是否变为重击 | [技能](../../../raw/current/scripts/prefabs/skilltree_woodie.lua#L277-L287) | 依赖声明 [woodie.lua](../../../raw/current/scripts/prefabs/woodie.lua#L72) |
| 资源：weremoose_attacks.zip | bank `weremoose`、build `weremoose_build`；循环参数未显式传，实际是否重复未核验 | [assets](../../../raw/current/scripts/prefabs/weremoose_smash_fx.lua#L1-L4) | 内容未解包 |

## 常见陷阱与限制

- **误区：FX 造成伤害或震屏**：工厂无 combat；`PushMooseSmashShake` 与 AOE 伤害都是宿主攻击状态的调用，FX 只是同帧表现。[宿主结算](../../../raw/current/scripts/stategraphs/SGwilson.lua#L11018-L11059)
- **边界：预测去重只针对本地玩家**：ownerdirty 里仅 `_owner == ThePlayer` 时隐藏服务器副本；位面加成用 AddBonus／RemoveBonus 包裹单次 DoAttack，未命中不会误留加成。[去重](../../../raw/current/scripts/prefabs/weremoose_smash_fx.lua#L30-L35)、[加成](../../../raw/current/scripts/stategraphs/SGwilson.lua#L11050-L11054)
- **边界：背景层只对齐首帧**：AddBackFX 生成时取父层当前帧，之后两层各自独立播放，不做持续同步。[对齐](../../../raw/current/scripts/prefabs/weremoose_smash_fx.lua#L24-L25)

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496；76 行工厂全文、单注册、SGwilson／SGwilson_client 生成点、宿主 AOE 与位面加成、技能门槛、译名检索已核验；远端玩家客户端是否本地生成、晚加入表现、动画资源内容未核验。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、label-check 通过（结果见本批交付报告）。

返回 [[reference/prefabs/fx-creature-index|生物、Boss 与遭遇演出]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-1）。
- 2026-11-08 build 26474873：本页随版本迁移升版（锚点迁移 1 处），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-12）。
