---
title: "沃尔夫冈力量变身与远古守护者血滴（minotaur_blood1 等）"
created: "2026-11-01"
updated: "2026-11-08"
type: prefab
summary: "力量状态上升动画与远古守护者受伤阶段随机血滴的四个通用表注册及调用条件。"
aliases: ["wolfgang_mighty_fx", "minotaur_blood1", "minotaur_blood2", "minotaur_blood3", "力量变身特效", "远古守护者血滴", "牛头怪血滴"]
tags: [prefab, fx]
dst_build: "steam-buildid:26474873"
sources: ["fx.lua", "components/mightiness.lua", "stategraphs/SGwilson.lua", "prefabs/wolfgang.lua", "prefabs/minotaur.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 沃尔夫冈力量变身与远古守护者血滴（minotaur_blood1 等）

## 快速判断

- 四名是通用表动画：`wolfgang_mighty_fx` 属玩家 `powerup` 状态，三个 `minotaur_blood` 条目由远古守护者按数字随机选名。力量属性、目标选择和触手逻辑由生成者负责；公共代理与本地实体见 [[reference/mechanisms/generic-fx-factory#关键流程]]。
- 简中 `languages/chinese_s.po` 定向检索四个注册名的 `STRINGS.NAMES` 键未命中；标题中文是根据源码核验的效果描述，非官方译名。

## 注册与构造

| 注册名 | bank／build；anim | 字段与证据 |
|---|---|---|
| `wolfgang_mighty_fx` | `fx_wolfgang`／同左；`idle` | `nofaced`；无 sound，fn `FinalOffsetNegative1` 设偏移 -1。[条目](../../../raw/current/scripts/fx.lua#L2665-L2671)、[函数](../../../raw/current/scripts/fx.lua#L13-L15) |
| `minotaur_blood1`／`minotaur_blood2`／`minotaur_blood3` | `rook_rhino_blood_fx`／同左；依次为 `blood1`／`blood2`／`blood3` | 三名均 `nofaced`，声 `ancientguardian_rework/minotaur2/blood_splurt_small`；fn 设置透明度 0.5 并对本地动画设两朝向。[1](../../../raw/current/scripts/fx.lua#L2674-L2684)、[2](../../../raw/current/scripts/fx.lua#L2687-L2697)、[3](../../../raw/current/scripts/fx.lua#L2700-L2710) |

## 关联机制导航

| 生成者／关系 | 触发、落点及证据 |
|---|---|
| 力量变身 | `mightiness:BecomeState` 在非静默转换时按旧／新档推事件：进入 mighty 或从 wimpy 到 normal 可推 `powerup`。玩家状态图仅非 dead 时进入 `powerup`，状态进入时在当前位置生成 FX；属性和皮肤由组件处理。[档位](../../../raw/current/scripts/components/mightiness.lua#L19-L47)、[变档门禁](../../../raw/current/scripts/components/mightiness.lua#L262-L266)、[事件](../../../raw/current/scripts/components/mightiness.lua#L293-L309)、[状态](../../../raw/current/scripts/stategraphs/SGwilson.lua#L2220-L2229)、[生成](../../../raw/current/scripts/stategraphs/SGwilson.lua#L2802-L2810) |
| 远古守护者受击 | `OnAttacked` 在生命低于 0.6、攻击者不是 chess 且尚无任务时启动 0.2 秒周期检查；每次有效检查按当前生命比例计算 chance2，命中后调用 `SpawnShadowFX`。该函数用 `"minotaur_blood"..math.random(1,3)` 选名，在本体位置生成并随机转向，非父挂。[阈值](../../../raw/current/scripts/prefabs/minotaur.lua#L78-L78)、[受击启动](../../../raw/current/scripts/prefabs/minotaur.lua#L180-L195)、[周期判定](../../../raw/current/scripts/prefabs/minotaur.lua#L166-L175)、[生成](../../../raw/current/scripts/prefabs/minotaur.lua#L329-L338) |
| 大血滴区别 | `minotaur_blood_big` 是独立工厂注册，其受附近暗影触手数量限制，并非这三条通用表注册。[大血滴](../../../raw/current/scripts/prefabs/minotaur.lua#L306-L325)、[工厂](../../../raw/current/scripts/prefabs/minotaur.lua#L693-L733) |

## 常见陷阱与限制

- `SpawnShadowFX` 虽计算随机半径和 offset，生成坐标仍直接用 `pt.x, pt.y, pt.z`，不能把未使用的 offset 写成血滴随机散布。[函数](../../../raw/current/scripts/prefabs/minotaur.lua#L329-L338)
- `wolfgang_mighty_fx` 的生成入口是通用玩家状态 `powerup`，不能仅由注册名推断调用者一定是沃尔夫冈；`wolfgang.lua` 中声明仅是依赖。[状态](../../../raw/current/scripts/stategraphs/SGwilson.lua#L2802-L2810)、[声明](../../../raw/current/scripts/prefabs/wolfgang.lua#L39-L39)

## 验证记录

- 源码审阅：2026-11-01，Steam buildid 26335394；逐项核对四条 bank/build/anim/sound/fn 与朝向字段；全 `raw/current/scripts` 对完整注册名、`minotaur_blood` 字符串拼接、`SpawnShadowFX` 和玩家 `powerup` 状态定向反查。生成位置与实际使用的 offset 区分；未解包动画资源或验证其他模组调用。
- 游戏运行：未运行。

返回 [[reference/prefabs/fx-creature-index|生物、Boss 与遭遇演出]]。
- 2026-11-08 build 26474873：本页随版本迁移升版（机械沿用；引用区间未受本次源码行号位移影响），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-03）。
