---
title: "沃拓克斯传送、灵魂与分身特效（wortox_soul_spawn_fx 等）"
created: "2026-11-01"
updated: "2026-11-01"
type: prefab
summary: "沃拓克斯复活传送上下动画、灵魂生成、分身爆裂／消退与抵抗表现的七种通用表注册。"
aliases: ["wortox_teleport_reviver_top", "wortox_teleport_reviver_bottom", "wortox_soul_spawn_fx", "wortox_decoy_explode_fx", "wortox_decoy_fizzle_fx", "wortox_decoy_expire_fx", "wortox_resist_fx", "沃拓克斯灵魂特效", "灵魂分身特效", "沃拓克斯抵抗特效"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["fx.lua", "stategraphs/SGwilson.lua", "stategraphs/SGwortox_decoy.lua", "prefabs/wortox_reviver.lua", "prefabs/wortox.lua", "prefabs/panflute.lua", "prefabs/skilltree_wortox.lua", "prefabs/hats.lua", "prefabs/armor_lunarplant.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 沃拓克斯传送、灵魂与分身特效（wortox_soul_spawn_fx 等）

## 快速判断

- 七名均为通用表表现；传送、灵魂实体、分身伤害和抵抗判定在生成方。代理、本地动画及回收见 [[reference/mechanisms/generic-fx-factory#关键流程]]。
- 效果中文为根据动画和调用语境核验的描述，非官方译名；七个注册名的 NAMES 键未命中，角色 Wortox 官方简中名为沃拓克斯。[角色名称](../../../raw/current/scripts/languages/chinese_s.po#L352480-L352482)

## 注册与构造

除 resist 外 bank／build 都为 `wortox_teleport_reviver_fx`。表中未列的 sound 字段与 fn 均未设置。

| 注册名 | anim 与字段 | 证据 |
|---|---|---|
| `wortox_teleport_reviver_top` | `reviver_teleport`；sound `meta5/wortox/ttheart_in_f31`，fn `FinalOffset1` | [条目](../../../raw/current/scripts/fx.lua#L3582-L3587) |
| `wortox_teleport_reviver_bottom` | `reviver_jumpout`；无 sound，fn `FinalOffset1` | [条目](../../../raw/current/scripts/fx.lua#L3590-L3594) |
| `wortox_soul_spawn_fx` | `soul_spawn_poof`；无 sound 字段，fn 添加声音组件并播 `dontstarve/characters/wortox/soul/hop_out` | [条目](../../../raw/current/scripts/fx.lua#L3597-L3604) |
| `wortox_decoy_explode_fx` | `decoy_laughsplode`；`nofaced`，fn 播 hop_out 并设层偏移 1 | [条目](../../../raw/current/scripts/fx.lua#L3607-L3616) |
| `wortox_decoy_fizzle_fx` | `decoy_deathfade`；`nofaced`，fn 同上 | [条目](../../../raw/current/scripts/fx.lua#L3619-L3628) |
| `wortox_decoy_expire_fx` | `decoy_expirefade`；`nofaced`，fn 同上 | [条目](../../../raw/current/scripts/fx.lua#L3631-L3640) |
| `wortox_resist_fx` | bank／build `planar_resist_fx`，anim `deflect`；sound hop_out；fn 随机缩放和翻转并设颜色 | [条目](../../../raw/current/scripts/fx.lua#L3643-L3656) |

## 关联机制导航

| 生成者 | 触发、落点及证据 |
|---|---|
| 复活传送状态 | 物品施法需 caster 与 linkeditem 的 owner；owner 为自己走 selfuse，owner 缺失或两点间禁止传送时失败。仅传送到其他 owner 的许可分支进入 `wortox_teleport_reviver`，旧位置生成 top；pst 按有效平台及偏移或目标坐标传送，在新位置生成 bottom。无敌和控制器恢复由状态负责。[入口](../../../raw/current/scripts/stategraphs/SGwilson.lua#L16784-L16801)、[到达](../../../raw/current/scripts/stategraphs/SGwilson.lua#L16830-L16857)、[物品接线](../../../raw/current/scripts/prefabs/wortox_reviver.lua#L83-L137) |
| 排箫召灵魂 | 完成演奏且激活 `wortox_panflute_soulcaller` 时延迟 52 帧开始；每次 `SummonSoul` 同点生成 FX 和独立灵魂实体。[召唤](../../../raw/current/scripts/prefabs/panflute.lua#L58-L64)、[调度](../../../raw/current/scripts/prefabs/panflute.lua#L66-L88) |
| 分身荆棘 | `DoThorns_decoy` 要求目标有效可见、通过 combat 标签筛选和 `CanTarget`，再通过 `SoulDamageTest` 才在目标位置生成 soul FX；伤害由随后 combat 处理。[命中](../../../raw/current/scripts/prefabs/wortox.lua#L1420-L1453) |
| 分身死亡 | death 按 `decoyexplodes` 分爆炸与 fizzle 状态；各在第 18 帧落点生对应 FX、第 22 帧进入结算。fizzle 状态依 `expired` 或随机结果选 expire／fizzle。[分支](../../../raw/current/scripts/stategraphs/SGwortox_decoy.lua#L48-L59)、[爆炸](../../../raw/current/scripts/stategraphs/SGwortox_decoy.lua#L62-L92)、[消退](../../../raw/current/scripts/stategraphs/SGwortox_decoy.lua#L99-L132) |
| 月亮装备抵抗 | `OnResistDamage` 在持有人或物品附近按攻击者方向或随机方向生成 resist；y 为随机值，抵抗资格由 `ShouldResistFn` 与装备组件处理。帽与护甲的依赖声明本身不是生成。[抵抗资格](../../../raw/current/scripts/prefabs/skilltree_wortox.lua#L113-L126)、[表现](../../../raw/current/scripts/prefabs/skilltree_wortox.lua#L127-L146)、[帽声明](../../../raw/current/scripts/prefabs/hats.lua#L6922-L6922)、[护甲声明](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L17-L17) |

## 常见陷阱与限制

- `wortox_soul_spawn_fx` 是视觉声效，`wortox_soul_spawn` 才是召唤的灵魂实体；不要把两者当同一注册。
- 分身 fizzle 与 expire 是不同动画但共用声音和显示偏移；不要从 FX 推断爆炸伤害。

## 验证记录

- 源码审阅：2026-11-01，Steam buildid 26335394。逐项核对注册字段，在 `raw/current/scripts` 全树检索七个完整名称及 `wortox_decoy_`／`wortox_teleport_reviver_` 等前缀，区分依赖声明与生成。未解包动画资源，未核验其他模组调用。
- 游戏运行：未运行。

返回 [[reference/prefabs/fx-table-character-index|通用表 FX·角色与装备补充]]。
