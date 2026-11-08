---
title: "阿比盖尔之花召唤、收回与升级特效（abigailsummonfx 等）"
created: "2026-11-02"
updated: "2026-11-08"
type: prefab
summary: "阿比盖尔之花召唤与收回按骑乘选用的四种动画，以及幽灵等级变化和惊吓复用的花形动画。"
aliases: ["abigailsummonfx", "abigailsummonfx_mount", "abigailunsummonfx", "abigailunsummonfx_mount", "abigaillevelupfx", "阿比盖尔召唤特效", "阿比盖尔收回特效", "阿比盖尔升级特效"]
tags: [prefab, fx]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/abigail_flower.lua", "prefabs/abigail.lua", "prefabs/wendy.lua", "prefabs/ghostcommand_defs.lua", "stategraphs/SGwilson.lua", "stategraphs/SGabigail.lua", "actions.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 阿比盖尔之花召唤、收回与升级特效（abigailsummonfx 等）

## 快速判断

- 五个注册由阿比盖尔之花文件的 `MakeSummonFX` 包装工厂创建联网动画实体；召唤／收回各有骑乘变体，等级变化与惊吓共用 `abigaillevelupfx`。这是专属 Prefab 工厂，召唤和惊吓玩法由动作、绑定组件及状态图负责。[工厂与注册](../../../raw/current/scripts/prefabs/abigail_flower.lua#L415-L461)
- 阿比盖尔及阿比盖尔之花是简中官方名称；标题及别名中的效果词是源码核验的描述，非官方译名。五个 FX 注册名未命中 `.po` 的 `STRINGS.NAMES` 键。[角色与花](../../../raw/current/scripts/languages/chinese_s.po#L338464-L338472)

## 注册与构造

五种工厂均加 Transform、AnimState、Network 与 `FX` 标签；anim 与 bank 同值，偏移 1，无工厂音效；客户端在 `SetPristine` 后返回，主模拟设 `persists = false` 并在 `animover` 移除该联网实体。骑乘版六朝向，其余四朝向；动画帧数和视觉内容未解包验证。[工厂](../../../raw/current/scripts/prefabs/abigail_flower.lua#L415-L454)

| 注册名 | bank／anim | build、朝向与资源 |
|---|---|---|
| `abigailsummonfx` | `wendy_channel_flower` | `wendy_channel_flower`，四朝向，覆盖 `flower` 符号为 `abigail_flower_rework`。[注册](../../../raw/current/scripts/prefabs/abigail_flower.lua#L457-L457) |
| `abigailsummonfx_mount` | `wendy_mount_channel_flower` | 同上 build／符号覆盖，六朝向。[注册](../../../raw/current/scripts/prefabs/abigail_flower.lua#L458-L458) |
| `abigailunsummonfx` | `wendy_recall_flower` | `abigail_flower_rework`，四朝向。[注册](../../../raw/current/scripts/prefabs/abigail_flower.lua#L459-L459) |
| `abigailunsummonfx_mount` | `wendy_mount_recall_flower` | 同上 build，六朝向。[注册](../../../raw/current/scripts/prefabs/abigail_flower.lua#L460-L460) |
| `abigaillevelupfx` | `abigail_flower_change` | 同上 build，四朝向。[注册](../../../raw/current/scripts/prefabs/abigail_flower.lua#L461-L461) |

资源表声明对应花与四种施法动画 zip、`abigail_flower_change.zip`；资源声明不是对素材内容或可分发性的验证。[声明](../../../raw/current/scripts/prefabs/abigail_flower.lua#L395-L413)

## 关联机制导航

| 生成者／入口 | 条件与清理 | 证据 |
|---|---|---|
| 玩家召唤状态 | 阿比盖尔之花的 CASTSUMMON 进入 `summon_abigail`；51 帧按 rider 骑乘状态选召唤名并父挂玩家，花有 skin build 时覆写 FX 的 flower 符号。62 帧动作成功后状态清引用，由 FX 的 animover 清理；失败时 69 帧移除仍保存的 FX，提前退出也移除仍保存者。 | [动作分派](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1353-L1359)、[生成与动作](../../../raw/current/scripts/stategraphs/SGwilson.lua#L9644-L9679)、[退出](../../../raw/current/scripts/stategraphs/SGwilson.lua#L9701-L9706) |
| 玩家收回状态 | CASTUNSUMMON 进入 `unsummon_abigail`；25 帧按骑乘选收回名并父挂玩家，皮肤覆盖同样按花的 skin build 判断；26 帧执行动作，成功清引用，失败 32 帧移除仍保存的 FX，退出时也清理仍保存者。 | [生成与动作](../../../raw/current/scripts/stategraphs/SGwilson.lua#L9741-L9765)、[失败及退出](../../../raw/current/scripts/stategraphs/SGwilson.lua#L9771-L9798) |
| 绑定等级变化 | 阿比盖尔收到绑定等级变化时，非 limbo、level 大于 1、状态不 busy 且未死亡才进入 `ghostlybond_levelup`；15 帧生成 `abigaillevelupfx` 并父挂自身，有 skin build 时覆盖 flower 符号。声音由阿比盖尔在 14 帧播放。 | [事件及门禁](../../../raw/current/scripts/prefabs/abigail.lua#L352-L357)、[监听](../../../raw/current/scripts/prefabs/abigail.lua#L454-L455)、[状态](../../../raw/current/scripts/stategraphs/SGabigail.lua#L428-L452) |
| 惊吓指令 | `wendy_ghostcommand_3` 的惊吓指令通过 `DoGhostSpell` 检查冷却、绑定幽灵及其生命后转入 `scare`；15 帧同样父挂 `abigaillevelupfx`，同时推 `do_ghost_scare` 事件。 | [指令](../../../raw/current/scripts/prefabs/ghostcommand_defs.lua#L124-L152)、[选名](../../../raw/current/scripts/prefabs/ghostcommand_defs.lua#L162-L164)、[选项](../../../raw/current/scripts/prefabs/ghostcommand_defs.lua#L259-L270)、[状态](../../../raw/current/scripts/stategraphs/SGabigail.lua#L809-L831) |
| 依赖与玩法 | 温蒂列四个召唤／收回名，阿比盖尔列升级名，声明只保证依赖而不是生成；实际召唤／收回动作调用 `ghostlybond:Summon`／`Recall`。 | [温蒂依赖](../../../raw/current/scripts/prefabs/wendy.lua#L29-L32)、[阿比盖尔依赖](../../../raw/current/scripts/prefabs/abigail.lua#L28-L30)、[动作](../../../raw/current/scripts/actions.lua#L3687-L3697)；[[reference/components/ghostlybond]] |

## 常见陷阱与限制

- 召唤动画先于 `PerformBufferedAction`：看到 FX 不证明召唤或收回成功；状态失败与打断路径可提前移除实体。
- 此族直接创建带 AnimState 的联网实体；不要把通用表 FX 工厂的“网络代理另建本地动画”套用到这里。声音由相应状态播放，不在五项工厂内。[构造](../../../raw/current/scripts/prefabs/abigail_flower.lua#L415-L454)
- 玩家召唤／收回及阿比盖尔等级／惊吓状态在 `SpawnPrefab` 后直接访问返回值的 `.entity`，未做 nil 检查；若生成失败，会在父挂处报错，不能依赖后续动作失败清理处理此情况。[玩家](../../../raw/current/scripts/stategraphs/SGwilson.lua#L9645-L9646)、[收回](../../../raw/current/scripts/stategraphs/SGwilson.lua#L9743-L9744)、[阿比盖尔](../../../raw/current/scripts/stategraphs/SGabigail.lua#L446-L447)、[惊吓](../../../raw/current/scripts/stategraphs/SGabigail.lua#L822-L823)
- `abigaillevelupfx` 同时被等级变化与惊吓状态复用；它与 [[reference/prefabs/abigail_attack_fx|阿比盖尔攻击特效]] 是不同注册和工厂。

## 验证记录

- 源码审阅：2026-11-02，Steam buildid 26335394；逐项检查五个注册、工厂、资源、端及上述动作与状态。对 `raw/current/scripts` 全树检索 `abigailsummonfx`、`abigailunsummonfx`、`abigaillevelupfx` 及 `summonfx`／`unsummonfx` 前缀，区分状态生成与依赖声明；动画素材内部与外部模组调用未核验。
- 游戏运行：未运行。

返回 [[reference/prefabs/fx-transition-index|变身、召唤、传送与退场]]。
- 2026-11-08 build 26474873：本页随版本迁移升版（机械沿用；引用区间未受本次源码行号位移影响），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-03）。
