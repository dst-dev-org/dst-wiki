---
title: "暗影手与海上水植特效（waterplant_destroy 等）"
created: "2026-11-01"
updated: "2026-11-01"
type: prefab
summary: "暗影手、水流与水植命中、海蚀柱升级和桅杆避雷针的五个通用表效果。"
aliases: ["shadowhand_fx", "waterstreak_burst", "waterplant_burr_burst", "waterplant_destroy", "mastupgrade_lightningrod_fx", "暗影手特效", "水植爆裂", "避雷针特效"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["fx.lua", "prefabs/waveyjones.lua", "prefabs/minotaur.lua", "prefabs/waterprojectiles.lua", "prefabs/waterplant_seed.lua", "prefabs/waterplant_rock.lua", "prefabs/seastack.lua", "prefabs/mastupgrade_lightningrod.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 暗影手与海上水植特效（waterplant_destroy 等）

## 快速判断

- 五名均为通用表动画表现，分别对应暗影手、水流弹、水植刺球、水植升级和桅杆避雷针；生成方负责命中、伤害或升级。代理、端与回收见 [[reference/mechanisms/generic-fx-factory#关键流程]]。
- 中文为按条目及调用语境核验的效果描述，非官方译名；`languages/chinese_s.po` 按五个注册名检索无显示名键。

## 注册与构造

共用通用工厂；五个表项均无 `fn`，未列 sound 者无 `sound` 字段。

| 注册名 | bank / build；anim | 其他字段与证据 |
|---|---|---|
| `shadowhand_fx` | 同名 bank / build；`idle` | 无 sound。[条目](../../../raw/current/scripts/fx.lua#L2239-L2242) |
| `waterstreak_burst` | `waterstreak` / `waterstreak`；`used` | `sixfaced = true`；sound `turnoftides/common/together/water/splash/small`。[条目](../../../raw/current/scripts/fx.lua#L2245-L2250) |
| `waterplant_burr_burst` | `barnacle_burr` / `barnacle_burr`；`used` | sound `dangerous_sea/creatures/water_plant/burr_burst`。[条目](../../../raw/current/scripts/fx.lua#L2253-L2257) |
| `waterplant_destroy` | `collapse` / `structure_collapse_fx`；`collapse_small` | sound `dangerous_sea/creatures/water_plant/grow`。[条目](../../../raw/current/scripts/fx.lua#L2260-L2264) |
| `mastupgrade_lightningrod_fx` | 同名 bank / build；`idle` | 无 sound。[条目](../../../raw/current/scripts/fx.lua#L2267-L2270) |

## 关联机制导航

| 生成者／触发 | 落点及关联 | 证据 |
|---|---|---|
| Wavey Jones 的 hand art | 主模拟每 2 秒排队，随机延迟最多 0.5 秒后在本体位置生成 `shadowhand_fx`，不父挂。`minotaur.lua` 中同名仅为依赖声明。 | [主模拟分界](../../../raw/current/scripts/prefabs/waveyjones.lua#L339-L341)、[生成与周期](../../../raw/current/scripts/prefabs/waveyjones.lua#L308-L313)、[周期安装](../../../raw/current/scripts/prefabs/waveyjones.lua#L360-L360)、[声明](../../../raw/current/scripts/prefabs/minotaur.lua#L30-L30) |
| 水流弹 `OnHitWaterstreak` | 在弹体坐标生成 `waterstreak_burst`；`IsPassableAtPoint` 为假时另生 splash，随后由弹体 wateryprotection 扩散效果并移除弹体。 | [命中](../../../raw/current/scripts/prefabs/waterprojectiles.lua#L339-L349) |
| 水植刺球两种命中回调 | 在刺球位置生成 `waterplant_burr_burst`，`IsPassableAtPoint` 为假时另生 splash；伤害由调用方 `do_bomb` 处理并移除弹体。 | [库存刺球](../../../raw/current/scripts/prefabs/waterplant_seed.lua#L111-L126)、[水植弹体](../../../raw/current/scripts/prefabs/waterplant_seed.lua#L228-L239) |
| `waterplant_rock` 与 `seastack` 升级 | 升级回调生成水植幼体和 `waterplant_destroy`，分别落在原物坐标或其 x/z、y=0，之后移除原物。 | [水植岩](../../../raw/current/scripts/prefabs/waterplant_rock.lua#L37-L52)、[海蚀柱](../../../raw/current/scripts/prefabs/seastack.lua#L61-L77) |
| 桅杆避雷针 `dozap` | 充能及充能期间定时调用，在本体位置生成 `mastupgrade_lightningrod_fx`；独立音效由生成方播放。 | [生成](../../../raw/current/scripts/prefabs/mastupgrade_lightningrod.lua#L34-L42)、[充能](../../../raw/current/scripts/prefabs/mastupgrade_lightningrod.lua#L68-L95) |

## 常见陷阱与限制

- `waterplant_destroy` 与 [[reference/prefabs/collapse_big#关联机制导航]] 中同素材效果是不同注册与工厂；相同动画不意味着调用同一 Prefab。
- 五种生成均为落点设置，不随生成者父挂；视觉效果不承担调用者的伤害、保护或升级。专服及动画回收边界见 [[reference/mechanisms/generic-fx-factory#常见陷阱与限制]]。

## 验证记录

- 源码审阅：2026-11-01，Steam buildid 26335394。逐项检查 bank/build/anim/sound/fn；在 `raw/current/scripts` 全域检索完整名及 `shadowhand`、`waterstreak`、`waterplant_burr`、`waterplant_destroy`、`mastupgrade_lightningrod` 前缀以查动态拼名，命中调用与声明按上表区分。未核验动画实际外观及其他版本。
- 游戏运行：未运行。

返回 [[reference/prefabs/fx-water-index|水面、航海与船件]]。
