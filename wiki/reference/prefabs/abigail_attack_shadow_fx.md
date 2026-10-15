---
title: "温蒂、阿比盖尔与纯辉标记特效（abigail_attack_shadow_fx 等）"
created: "2026-11-01"
updated: "2026-11-01"
type: prefab
summary: "温蒂理智光环增益、阿比盖尔暗影与启迪攻击、纯辉标记命中的七个通用表特效。"
aliases: ["wendy_sanityaura_buff_on_fx", "wendy_sanityaura_buff_off_fx", "abigail_attack_shadow_fx", "abigail_rising_twinkles_fx", "abigail_gestalt_hit_fx", "abigail_shadow_buff_fx", "purebrilliance_mark_hit_fx", "温蒂理智光环特效", "阿比盖尔暗影特效", "纯辉标记命中"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["fx.lua", "prefabs/wendy.lua", "prefabs/abigail.lua", "stategraphs/SGabigail.lua", "prefabs/slingshotammo_debuffs.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 温蒂、阿比盖尔与纯辉标记特效（abigail_attack_shadow_fx 等）

## 快速判断

- 七名分属理智光环、阿比盖尔技能与纯辉标记命中表现；状态、理智修正和伤害由生成者结算。公共代理、端与清理见 [[reference/mechanisms/generic-fx-factory#关键流程]]。
- 中文为依据条目和调用语境核验的效果描述，非官方译名；简中 `languages/chinese_s.po` 定向检索七个注册名的显示名键未命中。

## 注册与构造

| 注册名 | bank／build；anim | 其他字段及证据 |
|---|---|---|
| `wendy_sanityaura_buff_on_fx` | `wendy_sanityaura_buff_fx`／同左；`on` | 无 sound、fn。[条目](../../../raw/current/scripts/fx.lua#L3778-L3781) |
| `wendy_sanityaura_buff_off_fx` | 同上；`off` | 无 sound、fn。[条目](../../../raw/current/scripts/fx.lua#L3785-L3788) |
| `abigail_attack_shadow_fx` | `abigail_attack_fx`／同左；`attack3_ground_pre` | animqueue；fn 依次追加 loop、pst，染黑、贴地、GROUND 层、排序与偏移 1；sound 为注释，实际无 sound。[条目](../../../raw/current/scripts/fx.lua#L3792-L3808) |
| `abigail_rising_twinkles_fx` | bank／build／anim 均为 `abigail_rising_twinkles` | 无 sound、fn。[条目](../../../raw/current/scripts/fx.lua#L3811-L3814) |
| `abigail_gestalt_hit_fx` | `abigail_meta5_fx`／同左；`ghostalt_atk` | bloom；无 sound、fn。[条目](../../../raw/current/scripts/fx.lua#L3818-L3822) |
| `abigail_shadow_buff_fx` | `abigail_meta5_fx`／同左；`sacrifice_boost` | 无 sound、fn。[条目](../../../raw/current/scripts/fx.lua#L3825-L3828) |
| `purebrilliance_mark_hit_fx` | `slingshotammo_purebrilliance_mark_fx`／同左；`fx_hit` | fn 随机 1.2～1.4 缩放、可翻转，FinalOffset 7；无 sound。[条目](../../../raw/current/scripts/fx.lua#L3831-L3840) |

## 关联机制导航

| 生成者／关系 | 条件及证据 |
|---|---|
| 温蒂理智光环 | `sanityauraadjuster` 回调在骨灰盒登记系统激活时收集 25 范围玩家；新进名单、有 sanity 的玩家生成 on 代理并 AddChild，离开名单则 off；玩家自身播放声音，修正由 sanity 组件维护。[选人和切换](../../../raw/current/scripts/prefabs/wendy.lua#L101-L149)、[组件绑定](../../../raw/current/scripts/prefabs/wendy.lua#L373-L374) |
| 阿比盖尔暗影增益 | 温蒂 `murdered` 事件经 `wendy_shadow_3`、幽灵存在及不在 INLIMBO 门禁调用 `DoShadowBurstBuff`；后者自身位置生成黑色攻击动画、父挂 shadow buff 表现，并操作 murder buff。[门禁](../../../raw/current/scripts/prefabs/wendy.lua#L246-L251)、[绑定](../../../raw/current/scripts/prefabs/wendy.lua#L411-L411)、[生成与增益](../../../raw/current/scripts/prefabs/abigail.lua#L590-L604) |
| 阿比盖尔额外生命 | `AddBonusHealth` 在当前 bonus 低于计算上限时增加数值并父挂 twinkles；否则不生成。全 `raw/current/scripts` 按 `AddBonusHealth` 检索仅见定义、注释与实体导出，未找到原版调用；不排除动态或外部调用。[函数](../../../raw/current/scripts/prefabs/abigail.lua#L633-L641)、[导出](../../../raw/current/scripts/prefabs/abigail.lua#L949-L949) |
| 启迪形态命中 | SG 两种攻击路径在目标可被攻击且满足距离条件后，只有目标 combat 的 hiteffectsymbol 存在时才对子目标父挂 gestalt hit；攻击及闪白在状态图处理。[第一路径](../../../raw/current/scripts/stategraphs/SGabigail.lua#L959-L975)、[另一攻击](../../../raw/current/scripts/stategraphs/SGabigail.lua#L1064-L1076) |
| 纯辉弹弓标记 | 已附着目标受到非重定向位面攻击且通过生命和 combat 条件时追加伤害；排除攻击者带 `attacktriggereddebuff` 标签或攻击者就是该 debuff 的情况。攻击者存在才生成命中 FX，按目标半径及攻击者方向落点，世界 y 随机 0～1。[门禁与生成调用](../../../raw/current/scripts/prefabs/slingshotammo_debuffs.lua#L142-L177)、[落点](../../../raw/current/scripts/prefabs/slingshotammo_debuffs.lua#L107-L136) |

## 常见陷阱与限制

- 命中动画不保证每次启迪攻击都出现：目标没有 `hiteffectsymbol` 时不会走该表现分支。
- `purebrilliance_mark_hit_fx` 不是阿比盖尔专属；它属于纯辉弹弓标记的受击链，和前述六名仅按本批表项归页。

## 验证记录

- 源码审阅：2026-11-01，Steam buildid 26335394。逐项核验七条 bank/build/anim/sound/fn；`raw/current/scripts` 全树检索七个完整名及 `wendy_sanityaura`、`abigail_`、`purebrilliance_mark` 前缀，并区分依赖声明与生成。未核验动画素材、模组动态调用或导出函数的外部接线。
- 游戏运行：未运行。

返回 [[reference/prefabs/fx-table-character-index|通用表 FX·角色与装备补充]]。
