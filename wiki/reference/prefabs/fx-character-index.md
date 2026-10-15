---
title: "FX · 角色技能、施法与读书"
created: "2026-11-07"
updated: "2026-11-07"
type: index
tags: [overview]
---

# FX · 角色技能、施法与读书

角色法术、读书、技能反馈、灵魂消耗与薇洛控火。

## 施法、读书与工具提示

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/staffcastfx|施法与怀表回溯表现]] | 八注册：骑乘选名、父挂染色与状态清理 | `staffcastfx_mount`、`cointosscastfx`、`cointosscastfx_mount`、`pocketwatch_cast_fx`、`pocketwatch_cast_fx_mount`、`pocketwatch_warpback_fx`、`pocketwatch_warpbackout_fx` / active |
| [[reference/prefabs/staff_castinglight|施法光源]] | 两注册：施法／护符／怀表状态的光晕，五次幂包络与 2 帧延迟移除 | `staff_castinglight_small` / active |
| [[reference/prefabs/book_fx|读书施法书本特效]] | 六种读书表现；骑乘选型与施法失败反馈 | `book_fx_mount`、`waxwell_book_fx`、`waxwell_book_fx_mount`、`waxwell_shadow_book_fx`、`waxwell_shadow_book_fx_mount` / active |
| [[reference/prefabs/fx_book_moon|读书施法特效族（fx_book_moon 等）]] | 18 注册：读者落点与书本按受效对象两条生成链，骑乘回落与 fish 特殊工厂 | fx_book_moon_mount、fx_book_research_station、fx_book_research_station_mount、fx_book_temperature、fx_book_temperature_mount、fx_book_bees、fx_book_fire、fx_book_fire_mount、fx_book_light、fx_book_light_upgraded、fx_book_birds、fx_book_birds_mount、fx_book_rain、fx_book_rain_mount、fx_book_sleep、fx_book_sleep_mount、fx_book_fish、读书特效、施法特效 / active |
| [[reference/prefabs/fx_book_mount|书页覆盖与骑乘挂载特效族]] | 12 注册：六种读书效果的前后层与骑乘变体 | `fx_tentacles_under_book`、`fx_lightning_over_book` 等 / active |
| [[reference/prefabs/spell_fire_throw|火杖、月化宠物与薇洛火特效]] | 4 注册：火杖失败、宠物结束、暗影火爆裂与未见生成的施火条目 | fire_fail_fx、wormwood_lunar_transformation_finish、willow_shadow_fire_explode / active |
| [[reference/prefabs/dr_warm_loop_1|探测杖距离环与 WX 音乐特效]] | 6 注册：探测杖距离选环、音乐电路音符；旧蒸汽无生成点 | dr_warm_loop_2、dr_warmer_loop、dr_hot_loop、wx78_heat_steam、wx78_musicbox_fx / active |

## 角色技能、灵魂与火焰

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/wortox_eat_soul_fx|进食灵魂特效]] | 灵魂进食动画；骑乘切换与状态清理 | — / active |
| [[reference/prefabs/wortox_portal_fx|灵魂跳跃出入场特效]] | 灵魂跳跃出入场；宿主位移与特效分工 | `wortox_portal_jumpin_fx`、`wortox_portal_jumpout_fx`、`soulhop` / active |
| [[reference/prefabs/wortox_soul_in_fx|灵魂入体特效]] | 灵魂入体提示；目标同步与客户端染色 | — / active |
| [[reference/prefabs/wortox_soul_spawn_fx|沃拓克斯传送、灵魂与分身特效]] | 7 注册：复活传送、灵魂、分身与抵抗 | wortox_teleport_reviver_top、wortox_teleport_reviver_bottom、wortox_decoy_explode_fx、wortox_decoy_fizzle_fx、wortox_decoy_expire_fx、wortox_resist_fx / active |
| [[reference/prefabs/wortox_soulecho_buff_fx|沃拓克斯技能与装备表现族]] | 6 注册：灵魂回响加速层、灵魂过载警告、强抢袋三层（载体/动画/粒子）、双尾心跟随体 | `wortox_soulecho_buff_fx`、`wortox_overloading_fx`、`wortox_nabbag_body` 等 / active |
| [[reference/prefabs/wurt_casting_fx|沃特施法特效]] | 四种鱼人施法动画；骑乘拼名与上下分层 | `purebrilliance_castfx`、`purebrilliance_castfx_mount`、`horrorfuel_castfx`、`horrorfuel_castfx_mount` / active |
| [[reference/prefabs/wurt_terraformer_fx_shadow|沃特沼泽改造、水盾与暗影鱼人特效]] | 8 注册：改造器、湿度抵伤与暗影鱼人 | wurt_swamp_terraform_fx、wurt_terraformer_fx_lunar、wurt_water_splash_1、wurt_water_splash_2、wurt_water_splash_3、shadow_merm_spawn_poof_fx、shadow_merm_smacked_poof_fx / active |
| [[reference/prefabs/wurt_shadow_merm_planar_fx|沃特位面增益与投泥带表现]] | 5 注册：暗影持续跟随层、月影一次性 pre/pst、投泥弹壳（改泥代理辨析）与两个 chargedfx（清单外并入） | `wurt_shadow_merm_planar_fx`、`wurt_lunar_merm_planar_fx`、`wurt_terraform_projectile` 等 / active |
| [[reference/prefabs/willow_shadow_flame|薇洛暗焰]] | 薇洛三种火焰；索敌伤害与循环清理边界 | `willow_throw_flame`、`willow_frenzy` / active |
| [[reference/prefabs/flamethrower_fx|薇洛暗焰喷射布焰代理]] | 薇洛喷焰代理；周期布焰与延迟武器清理 | `lunar_fire` / active |
| [[reference/prefabs/channel_absorb_fire_fx|薇洛打火机吸火引导特效]] | 打火机引导吸火表现；KillFX 与一次性标记 | `channel_absorb_fire`、`channel_absorb_smoulder`、`channel_absorb_embers` / active |
| [[reference/prefabs/wormwood_plant_fx|沃姆伍德开花植物特效]] | 开花小植物；变体轮换与玩家驻留条件 | — / active |
| [[reference/prefabs/wx78_big_spark|WX78 外围表现族]] | 4 注册：大火花（代理无 AnimState 闪白不可达）、照明电路灯点链、扫描锥 AddChild、备份底盘渲染层（混合） | `wx78_big_spark`、`wx78_lightbeam`、`wx78_scanner_fx`、`wx78_backupbody_inventory` / active |
| [[reference/prefabs/witchbroom_fx|扫帚轨迹与沃比驮架与教练buff]] | 3 注册：隐形壳＋本地池化轨迹（6/s 节拍勘误）、沃比驮架双层（大沃比专属）、教练伤害buff纯表现层 | `witchbroom_fx`、`woby_rack_swap_fx`、`wolfgang_coach_buff_fx` / active |

返回 [[reference/prefabs/fx-index|FX 主题总索引]]、[[reference/prefabs/index|Prefab 索引]]。
