---
title: "虚布回旋镖与皮弗娄牛复活雷光（voidcloth_boomerang_launch_fx 等）"
created: "2026-11-01"
updated: "2026-11-08"
type: prefab
summary: "虚布回旋镖发射与命中、皮弗娄牛复活雷光的三个通用表特效及各自生成入口。"
aliases: ["voidcloth_boomerang_launch_fx", "voidcloth_boomerang_impact_fx", "beefalo_reviving_lightning_fx", "虚布回旋镖特效", "牛复活雷光"]
tags: [prefab, fx]
dst_build: "steam-buildid:26474873"
sources: ["fx.lua", "prefabs/voidcloth_boomerang.lua", "prefabs/beef_bell.lua", "prefabs/beefalo.lua", "stategraphs/SGBeefalo.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 虚布回旋镖与皮弗娄牛复活雷光（voidcloth_boomerang_launch_fx 等）

## 快速判断

- 三名是通用表动画表现；回旋镖命中与返回由投射物处理，牛的复活、无敌及动作由牛本体处理。公共代理、端与清理见 [[reference/mechanisms/generic-fx-factory#关键流程]]。
- 中文为依据条目和调用语境核验的效果描述，非官方译名；简中 `languages/chinese_s.po` 定向检索三个注册名的显示名键未命中。

## 注册与构造

| 注册名 | bank／build；anim | 其他字段及证据 |
|---|---|---|
| `voidcloth_boomerang_launch_fx` | `boomerang_voidcloth`／同左；`launch_fx` | tintalpha 0.85；fn 设置贴地、GROUND 层、排序 1、FinalOffset 1；无 sound。[条目](../../../raw/current/scripts/fx.lua#L3657-L3667) |
| `voidcloth_boomerang_impact_fx` | `boomerang_voidcloth`／同左；`impact_fx` | 1.3 倍三轴 transform、tintalpha 0.85、fn=FinalOffset1；无 sound。[条目](../../../raw/current/scripts/fx.lua#L3670-L3676)、[helper](../../../raw/current/scripts/fx.lua#L1-L3) |
| `beefalo_reviving_lightning_fx` | `beefalo`／`beefalo_revive`；`revive_lightning` | bloom；fn 设置 LightOverride 0.7 与 FinalOffset 1；无 sound。[条目](../../../raw/current/scripts/fx.lua#L3679-L3688) |

## 关联机制导航

| 生成者／触发 | 条件、落点与证据 |
|---|---|
| 回旋镖投射物 `Projectile_OnThrown` | attacker 非空且有效时在目标相对攻击者的相邻位置生成 launch 代理，并朝向目标调整旋转；其他武器攻击与投射物登记由调用方处理。[回调](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L442-L468)、[绑定](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L541-L543) |
| 回旋镖投射物 `Projectile_OnHit` | 先调用返回投掷者；目标有效时按目标物理半径与投射物旋转计算偏移，在目标高度生成 impact，不父挂。`OnMiss` 返回但不生成该命中特效。[命中与未命中](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L361-L380) |
| 牛复活状态 | 牛 `OnRevived` 进入 `revive` 状态；状态进入时 `SpawnChild` 雷光并设置本体无敌、唤醒，复活声由牛自身播放。牛铃回调调用该入口。[牛铃](../../../raw/current/scripts/prefabs/beef_bell.lua#L271-L280)、[牛入口](../../../raw/current/scripts/prefabs/beefalo.lua#L705-L707)、[状态](../../../raw/current/scripts/stategraphs/SGBeefalo.lua#L660-L679) |

## 常见陷阱与限制

- 雷光子代理与动画表现不负责复活或赋无敌；声音在状态本体，不是注册条目的 sound。
- 回旋镖未命中分支只调用返回，不应把 impact 当成每次返回都出现的效果。[未命中](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L378-L380)

## 验证记录

- 源码审阅：2026-11-01，Steam buildid 26335394。逐项核对三条 bank/build/anim/sound/fn 及附加字段；在 `raw/current/scripts` 全树检索完整注册名和 `voidcloth_boomerang`、`beefalo_reviving` 前缀以检查动态选名，区分依赖声明与生成。动画素材与模组调用未核验。
- 游戏运行：未运行。

返回 [[reference/prefabs/fx-combat-index|攻击、弹道与落点]]。
- 2026-11-08 build 26474873：本页随版本迁移升版（机械沿用；引用区间未受本次源码行号位移影响），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-07）。
