---
title: "火焰失败、月化结束与暗影火爆裂（spell_fire_throw 族）"
created: "2026-11-01"
updated: "2026-11-01"
type: prefab
summary: "火杖投射失败、沃姆伍德月化宠物结束、施火预留和薇洛暗影火命中表现的四个通用表注册。"
aliases: ["fire_fail_fx", "wormwood_lunar_transformation_finish", "spell_fire_throw", "willow_shadow_fire_explode", "火焰失败", "月化宠物结束", "薇洛暗影火爆裂"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["fx.lua", "prefabs/fx.lua", "prefabs/staff_projectile.lua", "prefabs/wormwood_carrat.lua", "prefabs/wormwood_fruitdragon.lua", "prefabs/wormwood_lightflier.lua", "prefabs/willow_ember.lua", "prefabs/willow_flame_fx.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 火焰失败、月化结束与暗影火爆裂（spell_fire_throw 族）

## 快速判断

- 四个动画 FX 分属火杖命中失败、月化宠物结束与薇洛暗影火等路径；施法与伤害由生成者执行。通用代理与清理见 [[reference/mechanisms/generic-fx-factory#关键流程|工厂流程]]。
- 简中 `.po` 定向检索四注册名无显示名键；中文为效果描述，非官方译名。

## 注册与构造

均用通用表工厂，未列 sound 或 fn 表示该字段缺省。

| 注册名 | bank / build / anim | 差异与证据 |
|---|---|---|
| `fire_fail_fx` | `fire_fail_fx` / 同 bank / `fx` | sound `dontstarve/common/fireOut`；fn 为 `flame01` 设 bloom 和光照覆盖 1。[条目](../../../raw/current/scripts/fx.lua#L3150-L3158) |
| `wormwood_lunar_transformation_finish` | `fx_moon_tea` / `moon_tea_fx` / `puff` | bloom；sound `meta2/wormwood/animation_dropdown`；fn `FinalOffset1`。[条目](../../../raw/current/scripts/fx.lua#L3178-L3184)、[函数](../../../raw/current/scripts/fx.lua#L1-L3) |
| `spell_fire_throw` | `fire_geyser` / `fire_geyser_fx` / `pre` | 无声、无 fn。[条目](../../../raw/current/scripts/fx.lua#L3234-L3238) |
| `willow_shadow_fire_explode` | `deer_fire_charge` / 同 bank / `blast` | sound `dontstarve/common/deathpoof`；黑色 tint，fn 缩放 1.5。[条目](../../../raw/current/scripts/fx.lua#L3241-L3249) |

## 关联机制导航

| 生成者／关系 | 条件及依据 |
|---|---|
| 火杖投射物 | `OnHitFire` 对有效目标且目标没有正在燃烧的 burnable 时，水平坐标取目标近弹体一侧并加随机偏移，世界 y 随机 0.1～0.3；回调末尾移除弹体。[命中](../../../raw/current/scripts/prefabs/staff_projectile.lua#L25-L45)、[接线](../../../raw/current/scripts/prefabs/staff_projectile.lua#L98-L105) |
| 月化宠物 | 胡萝卜鼠、火龙果龙蝇、球状光虫结束生命时分别释放 carrot、dragonfruit、lightbulb，在原位置生成月化结束 FX 并移除自身；定时及宠物主人攻击可触发结束。[鼠](../../../raw/current/scripts/prefabs/wormwood_carrat.lua#L38-L65)、[龙蝇](../../../raw/current/scripts/prefabs/wormwood_fruitdragon.lua#L37-L61)、[光虫](../../../raw/current/scripts/prefabs/wormwood_lightflier.lua#L29-L45) |
| 薇洛暗影火 | 寻到目标并进入近距离后由 source 的 combat 执行攻击；若火焰攻击后仍有效，在目标位置落爆裂 FX。[命中](../../../raw/current/scripts/prefabs/willow_flame_fx.lua#L121-L140) |
| 施火名 | `willow_ember` 的 prefabs 表声明 `spell_fire_throw`，声明不等于生成。[声明](../../../raw/current/scripts/prefabs/willow_ember.lua#L13-L20) |

## 常见陷阱与限制

- 在 scripts 全域检索 `spell_fire_throw`、`spell_fire`、`fire_throw`：除表项及依赖声明，未发现该注册名活跃生成点；名称拆分或模组运行时调用仍有未覆盖可能。[条目](../../../raw/current/scripts/fx.lua#L3234-L3238)
- `willow_shadow_fire_explode` 的 `tint = Vector3(0, 0, 0, 0.6)` 在通用工厂读取 xyz，alpha 来自另一个 `tintalpha` 字段的默认值 1；不能据第四个构造参数断言透明度 0.6。[条目](../../../raw/current/scripts/fx.lua#L3241-L3249)、[消费](../../../raw/current/scripts/prefabs/fx.lua#L69-L72)

## 验证记录

- 源码审阅：2026-11-01，Steam buildid 26335394；核对四注册字段、工厂与上述消费者，在 scripts 树检索完整注册名和 `spell_fire`、`fire_throw`。未核验动画资源内部、其他模组动态调用。
- 游戏运行：未运行。

返回 [[reference/prefabs/fx-table-index|通用表 FX 索引]]。
