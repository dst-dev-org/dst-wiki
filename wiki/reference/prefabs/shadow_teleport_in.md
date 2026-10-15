---
title: "暗影传送、孢子咳出与宝盒玩家特效（shadow_teleport_in 族）"
created: "2026-11-01"
updated: "2026-11-01"
type: prefab
summary: "暗影生物海陆交换的进出传送、蘑菇地精咳孢子与远古宝盒玩家特效的四个通用表注册。"
aliases: ["shadow_teleport_in", "shadow_teleport_out", "spore_moon_coughout", "archive_lockbox_player_fx", "暗影传送", "孢子咳出特效", "宝盒玩家特效"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["fx.lua", "prefabs/fx.lua", "prefabs/shadowcreature.lua", "prefabs/oceanshadowcreature.lua", "stategraphs/SGshadowcreature.lua", "stategraphs/SGoceanshadowcreature.lua", "stategraphs/SGmushgnome.lua", "prefabs/archive_lockbox.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 暗影传送、孢子咳出与宝盒玩家特效（shadow_teleport_in 族）

## 快速判断

- 四个动画 FX 分属三条生成链：暗影生物交换、蘑菇地精攻击和远古宝盒授予配方；游戏行为由各生成者处理。通用工厂的端与清理见 [[reference/mechanisms/generic-fx-factory#关键流程|通用 FX 工厂流程]]。
- 传送进、出为贴地背景层；宝盒特效父挂玩家；咳出特效落在攻击开始时记录的位置。各条目无 `sound` 字段。[条目](../../../raw/current/scripts/fx.lua#L2273-L2297)

## 注册与构造

均由通用表代理生成本地动画实体，具体流程见 [[reference/mechanisms/generic-fx-factory#关键流程|工厂流程]]；以下为条目字段，未列 sound 即无 sound。

| 注册名 | bank / build / anim | fn 与证据 |
|---|---|---|
| `shadow_teleport_in` | `shadow_teleport` / `shadow_teleport` / `portal_in` | `GroundOrientation`：贴地、背景层。[条目](../../../raw/current/scripts/fx.lua#L2273-L2277)、[函数](../../../raw/current/scripts/fx.lua#L24-L27) |
| `shadow_teleport_out` | `shadow_teleport` / `shadow_teleport` / `portal_out` | 同上。[条目](../../../raw/current/scripts/fx.lua#L2280-L2284) |
| `spore_moon_coughout` | `spore_moon` / `mushroom_spore_moon` / `pre_cough_out` | 无 fn。[条目](../../../raw/current/scripts/fx.lua#L2287-L2290) |
| `archive_lockbox_player_fx` | `archive_lockbox_player_fx` / 同 bank / `activation` | `FinalOffset1`。[条目](../../../raw/current/scripts/fx.lua#L2293-L2297)、[函数](../../../raw/current/scripts/fx.lua#L1-L3) |

在简中 `.po` 中定向检索注册名未命中显示名键；标题及别名中文是效果描述，非官方译名。

## 关联机制导航

| 生成者／对象 | 触发、落点与证据 |
|---|---|
| 陆上暗影生物 | `teleport_to_sea` 入状态和第 40 帧各在自身位置生成 `out`；动画队列结束执行海陆交换并移除旧实体。[状态](../../../raw/current/scripts/stategraphs/SGshadowcreature.lua#L278-L308)；交换成功后在海上新实体位置生成 `in`。[交换](../../../raw/current/scripts/prefabs/shadowcreature.lua#L200-L220) |
| 海上暗影生物 | `teleport_to_land` 第 42 帧在自身位置生成 `out`；队列结束交换并移除旧实体。[状态](../../../raw/current/scripts/stategraphs/SGoceanshadowcreature.lua#L345-L369)；成功后在陆上新实体位置生成 `in`。[交换](../../../raw/current/scripts/prefabs/oceanshadowcreature.lua#L291-L313) |
| 蘑菇地精 | `attack` 进入时保存位置，第 23 帧在该位置生成咳出 FX，第 25 帧起另生成孢子。[状态](../../../raw/current/scripts/stategraphs/SGmushgnome.lua#L182-L209) |
| 远古宝盒 | 开始播放 `activation` 后立即排入 `174/30` 秒后的 `teach`（不等 animover）；搜索 20 范围玩家，当 recipe 非空且玩家有 builder 时生成特效并 `AddChild`，配方蓝图与对白由宝盒处理。[触发](../../../raw/current/scripts/prefabs/archive_lockbox.lua#L102-L108)、[授予](../../../raw/current/scripts/prefabs/archive_lockbox.lua#L47-L96) |

## 常见陷阱与限制

- 海陆交换要求存在战斗目标且目标位于相应的海／陆视觉地面；目标缺失、地形不符，或搜索循环变量 `radius` 达到 30 时均不生成对应 `in`。循环每轮先把 `radius` 加 2，再把该轮 offset 累加到当前位置，30 不是实际搜索半径（累计位移可更大）。状态的动画队列结束后仍移除旧实体，因此 `out` 不证明交换成功。[陆到海](../../../raw/current/scripts/prefabs/shadowcreature.lua#L193-L220)、[海到陆](../../../raw/current/scripts/prefabs/oceanshadowcreature.lua#L286-L313)、[陆上状态](../../../raw/current/scripts/stategraphs/SGshadowcreature.lua#L302-L308)、[海上状态](../../../raw/current/scripts/stategraphs/SGoceanshadowcreature.lua#L364-L369)
- 宝盒授予特效不代表获得了新蓝图：已知配方也可能产生特效。[授予](../../../raw/current/scripts/prefabs/archive_lockbox.lua#L62-L96)

## 验证记录

- 源码审阅：2026-11-01，Steam buildid 26335394；核对四条注册、工厂及上述生成链；scripts 下按完整注册名、`shadow_teleport`、`spore_moon`、`archive_lockbox_player_fx` 检索，依赖声明不作为生成。第 7 轮复核明确交换搜索的 30 是循环变量上限而非实际搜索半径，宝盒计时从开始播放 `activation` 起算。未核验动画资源内部及其他模组动态生成。
- 游戏运行：未运行。

返回 [[reference/prefabs/fx-transition-index|变身、召唤、传送与退场]]。
