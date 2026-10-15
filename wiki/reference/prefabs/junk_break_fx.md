---
title: "退化、破裂与改造特效（junk_break_fx 等）"
created: "2026-11-01"
updated: "2026-11-01"
type: prefab
summary: "草船与冰碎片、海冰爆裂、围栏与地皮调整、废料、箱子升级、打蜡和束缚藤破裂的 11 个通用表表现。"
aliases: ["degrade_fx_grass", "degrade_fx_ice", "fx_ice_pop", "fence_rotator_fx", "turf_smoke_fx", "pull_smoke_fx", "junk_break_fx", "chestupgrade_stacksize_fx", "chestupgrade_stacksize_taller_fx", "beeswax_spray_fx", "goo_vines_break_fx", "废料破碎", "箱子升级特效", "打蜡喷雾", "冰块退化"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["fx.lua", "simutil.lua", "components/hullhealth.lua", "components/oceanicemanager.lua", "components/fencerotator.lua", "components/pinnable.lua", "prefabs/boat.lua", "prefabs/hats.lua", "prefabs/fence.lua", "prefabs/junk_pile.lua", "prefabs/junk_pile_big.lua", "prefabs/junk_pile_common.lua", "stategraphs/SGdaywalker2.lua", "prefabs/treasurechest.lua", "prefabs/dragonfly_chest.lua", "prefabs/waxed_plant_common.lua", "prefabs/snowman.lua", "prefabs/charlie_boss_vines.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 退化、破裂与改造特效（junk_break_fx 等）

## 快速判断

- 本页 11 名是通用表动画表现；造船、地皮调整、升级、打蜡和束缚玩法归各自调用方。工厂代理与本地实体的关系见 [[reference/mechanisms/generic-fx-factory#关键流程]]。
- 中文为按表项与调用语境核验的效果描述，非官方译名；`languages/chinese_s.po` 对 11 名定向检索无显示名键。动画包外观与时长未运行核验。

## 注册与构造

表中 bank/build 相同时合写；未列的 sound 或 fn 字段未设置。完整音效路径保留在证据所指条目中。

| 注册名 | bank / build；anim | 字段差异与证据 |
|---|---|---|
| `degrade_fx_grass` | `boat_grass`；`degrade_fx1` | 无声；animqueue、nofaced；fn 在地下层播随机 1～3 循环，缩放 0.5，180 帧后用 60 帧侵蚀。[条目](../../../raw/current/scripts/fx.lua#L2747-L2762) |
| `degrade_fx_ice` | `ice_debris`；`degrade_fx1` | 无声；animqueue、nofaced；fn 在地下层播随机 1～4 循环，约 10±2 秒后用 3 秒侵蚀。[条目](../../../raw/current/scripts/fx.lua#L3255-L3268) |
| `fx_ice_pop` | `fx_dock_crackleandpop`；`pop` | 声 `dontstarve_DLC001/common/iceboulder_smash`；无 fn。[条目](../../../raw/current/scripts/fx.lua#L3273-L3277) |
| `fence_rotator_fx` | 同名；`idle` | 声 `wickerbottom_rework/fence_rotator/use`；fn `FinalOffset1`。[条目](../../../raw/current/scripts/fx.lua#L3089-L3094) |
| `turf_smoke_fx` | 同名；`fx` | 声 `meta4/turfraiser_helm/raise_turf`；无 fn。[条目](../../../raw/current/scripts/fx.lua#L3098-L3102) |
| `pull_smoke_fx` | 同名；`fx` | 声 `lunarhail_event/gestalt/moonshard_smoke_pull`；fn 地面背景层。[条目](../../../raw/current/scripts/fx.lua#L3105-L3110)、[函数](../../../raw/current/scripts/fx.lua#L24-L27) |
| `junk_break_fx` | `scrapball`；`scrap_destruction_1` | 声 `qol1/daywalker_scrappy/pile_destroy`；fn 随机选择 1～3 动画及镜像，设 FinalOffset 1。[条目](../../../raw/current/scripts/fx.lua#L3306-L3321) |
| `chestupgrade_stacksize_fx` | `cavein_dust_fx`；`dust_low` | 声 `qol1/chest_upgrade/poof`；fn 在第 6 帧另播 `wolfgang1/mightygym/item_removed`，设层并逐帧改变颜色和透明度。[条目](../../../raw/current/scripts/fx.lua#L3324-L3353) |
| `chestupgrade_stacksize_taller_fx` | 同上 | 同声与渐变，fn 另将动画纵向缩放为 1.3。[条目](../../../raw/current/scripts/fx.lua#L3356-L3386) |
| `beeswax_spray_fx` | `fx_plant_spray`；`play_fx` | 声 `qol1/wax_spray/effect`；fn FinalOffset 3、动画缩放 1.3。[条目](../../../raw/current/scripts/fx.lua#L3293-L3303) |
| `goo_vines_break_fx` | `charlie_boss_vines`；`vine_break` | 声 `rifts/lunarthrall/vine_death`；fn FinalOffset 2、LightOverride 1。[条目](../../../raw/current/scripts/fx.lua#L4034-L4043) |

`ErodeAway` 的参数单位为秒，完成后移除传入的本地实体，不等于一秒后移除的网络代理。[helper](../../../raw/current/scripts/simutil.lua#L575-L597)

## 关联机制导航

| 生成者／触发 | 选名、落点与证据 |
|---|---|
| 草船退化 | 草船设 `hullhealth.degradefx`；组件退化更新按概率在船周边 x/z、y=0 生成该名。[配置](../../../raw/current/scripts/prefabs/boat.lua#L842-L845)、[选名](../../../raw/current/scripts/components/hullhealth.lua#L224-L234)、[触发](../../../raw/current/scripts/components/hullhealth.lua#L240-L246) |
| 冰船／海冰 | 冰船沉没碎片和 oceanicemanager 碎片生成 `degrade_fx_ice`；海冰销毁时退化碎片与 `fx_ice_pop` 均要求非 silent。预警 crackle 另见 [[reference/prefabs/fx_boat_crackle#触发窗口与生成者]]。[冰船](../../../raw/current/scripts/prefabs/boat.lua#L1377-L1382)、[另一碎片](../../../raw/current/scripts/prefabs/boat.lua#L1422-L1427)、[海冰](../../../raw/current/scripts/components/oceanicemanager.lua#L65-L71)、[销毁](../../../raw/current/scripts/components/oceanicemanager.lua#L362-L378) |
| 围栏旋转、蚁狮帽、天体帽 | `fencerotator` 在目标旋转后落 `fence_rotator_fx`；蚁狮帽地皮改造结束在瓦片中心落 `turf_smoke_fx`；天体帽种植过程找到月玻璃后在帽位置落 `pull_smoke_fx`。[旋转](../../../raw/current/scripts/components/fencerotator.lua#L14-L19)、[地皮](../../../raw/current/scripts/prefabs/hats.lua#L2395-L2397)、[吸取](../../../raw/current/scripts/prefabs/hats.lua#L3273-L3280) |
| 废料堆与废料围栏 | 多处使用 `junk_break_fx`；普通堆的零工作量 junkmob 互动和 workleft 耗尽是不同生成路径。大型堆拾取须对应 daywalker 状态，或通过 `CanSpawnFromJunk` 且成功生成，不能把每次拾取视为必出；落点与代理缩放各异。[围栏](../../../raw/current/scripts/prefabs/fence.lua#L367-L372)、[普通堆](../../../raw/current/scripts/prefabs/junk_pile.lua#L78-L115)、[大型堆](../../../raw/current/scripts/prefabs/junk_pile_big.lua#L292-L330)、[大型堆三](../../../raw/current/scripts/prefabs/junk_pile_big.lua#L409-L409)、[大型堆四](../../../raw/current/scripts/prefabs/junk_pile_big.lua#L514-L525)、[公共函数](../../../raw/current/scripts/prefabs/junk_pile_common.lua#L56-L56)、[SG](../../../raw/current/scripts/stategraphs/SGdaywalker2.lua#L418-L418) |
| 箱子升级 | 普通箱与龙鳞箱的 `numupgrades == 1` 且由物品升级而非读档时，分别在箱位生成普通版及 taller；箱本体把视觉变化延迟 6 帧，与条目中的第二声同步。[普通箱](../../../raw/current/scripts/prefabs/treasurechest.lua#L312-L328)、[龙鳞箱](../../../raw/current/scripts/prefabs/dragonfly_chest.lua#L150-L166) |
| 打蜡 | 蜡处理公共函数在给定 x/z、y=0 落 `beeswax_spray_fx`，植物转换成功后调用；雪人打蜡在本体位置生成，但推动中、被持有或小雪人未叠放时提前返回。实际状态修改由调用方完成。[植物 FX](../../../raw/current/scripts/prefabs/waxed_plant_common.lua#L213-L220)、[转换](../../../raw/current/scripts/prefabs/waxed_plant_common.lua#L570-L583)、[雪人](../../../raw/current/scripts/prefabs/snowman.lua#L342-L354) |
| 查莉藤束缚 | 藤把 `goo_vines_break_fx` 列表传给目标 pinnable；组件在束缚受击或解除时从该列表选名、生成并作为目标子实体，玩法由 pinnable 管理。[列表](../../../raw/current/scripts/prefabs/charlie_boss_vines.lua#L10-L12)、[传入](../../../raw/current/scripts/prefabs/charlie_boss_vines.lua#L319-L325)、[生成](../../../raw/current/scripts/components/pinnable.lua#L81-L88)、[受击](../../../raw/current/scripts/components/pinnable.lua#L37-L40)、[解除](../../../raw/current/scripts/components/pinnable.lua#L163-L166) |

## 常见陷阱与限制

- 退化条目用循环动画并自行安排侵蚀清理，不应假定普通一次性动画事件保证其寿命。其代理与本地实体独立；位置和父挂的共同边界见 [[reference/mechanisms/generic-fx-factory#常见陷阱与限制]]。
- 箱子条目声与 fn 中延时声是两次播放；升级数据与改造逻辑不在 FX 中。不同生成方对代理的变换不应直接当成本地实体持续跟随的证明。

## 验证记录

- 源码审阅：2026-11-01，Steam buildid 26335394；逐项核验 11 个表项和随机动画、sound、fn；递归检索 `raw/current/scripts` 完整名与 `degradefx`、`chestupgrade_stacksize`、`splashfxlist` 等前缀／字段，追到组件动态消费及声明。未核验动画素材内容、事件显示及游戏运行。
- 游戏运行：未运行。

返回 [[reference/prefabs/fx-table-world-index|通用表 FX 索引]]。
