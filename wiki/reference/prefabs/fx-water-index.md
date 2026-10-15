---
title: "FX · 水面、航海与船件"
created: "2026-11-07"
updated: "2026-11-07"
type: index
tags: [overview]
---

# FX · 水面、航海与船件

水花、冲滩、海面波纹、船尾航迹、船体破坏与海冰。

## 入水、出水与漂浮

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/splash|落水水花（splash）]] | 五种水花；floater 冷热分支、青蛙回巢、splash_ocean 活跃生成点与雪花溅 | hot_splash、frogsplash、splash_ocean、splash_snow_fx、水花、热水花、青蛙水花、海面水花、雪花溅 / active |
| [[reference/prefabs/splash_sink|沉没与单点水花（splash_sink 族）]] | 三注册：SinkEntity OCEAN 档协议与实体离场点、漩涡传送门入口／出口双落蓝染花、pigshrine 三触发泥花 | oceanwhirlportal_splash、mudpuddle_splash、沉没水花、漩涡传送门水花、泥坑水花 / active |
| [[reference/prefabs/splash_green|海面绿花与冲滩水洼（splash_green 族）]] | 四注册：三档按体型选名（GetCombatFxSize 映射）、生物／玩家冲滩差异、malbatross 死代码；washashore 水洼为地面层伴生 | splash_green_small、splash_green_large、washashore_puddle_fx、绿色水花、海面绿花、冲滩水洼 / active |
| [[reference/prefabs/ocean_splash_med1|海面水花特效族（ocean_splash_med1）]] | 8 注册：海鱼破水、浮标、暗影傀儡等水花与游泳尾流 | ocean_splash_med2、ocean_splash_small1、ocean_splash_small2、ocean_splash_ripple1、ocean_splash_ripple2、ocean_splash_swim1、ocean_splash_swim2、海面水花、游泳水花、水面涟漪、海面涟漪 / active |
| [[reference/prefabs/float_fx_front|漂浮物前后水环]] | 本地前后漂浮水环；宿主移除或侵蚀淡出 | `float_fx_back`、`float_fx` / active |
| [[reference/prefabs/player_float_hop_water_fx|玩家出水水环]] | 玩家出水水环；对帧、短暂跟随与解绑 | `player_hotspring_water_fx`、`player_float_fx` / active |
| [[reference/prefabs/flotsam_puddle|冲滩浮渣伴生水洼与拾取碎裂（flotsam_puddle 族）]] | 两注册：钓获漂浮物冲滩后父挂伴生水洼（缩放随机朝向）、拾取时碎裂动画；水中分支与绞盘打捞不出水洼 | flotsam_break、浮渣水洼、冲滩水洼、浮渣碎裂 / active |
| [[reference/prefabs/merm_splash|鱼人水花（merm_splash 族）]] | 三注册六代码生成点：王诞生 buff 态生成语句（当前快照未找到世界事件转发链）、月变异变、王诞与守卫召唤／回巢出退场，全部自身落点；音效由生成方播放，条目无 sound | merm_king_splash、merm_spawn_fx、鱼人水花、鱼人王水花、生成水花 / active |

## 海面、海树与水上生物

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/wave_med|海面波浪与浪花]] | 七注册：网络浪实体与客户端波光／岸浪；推船、加湿与本地生成 | `wave_splash`、`wave_shimmer`、`wave_shimmer_med`、`wave_shimmer_deep`、`wave_shimmer_flood`、`wave_shore` / active |
| [[reference/prefabs/oceantree_ripples_short|海树波纹、根须、倒伏与落叶表现]] | 12 注册：三档涟漪／根影、倒伏树干、柱体涟漪／根影／落叶与大树干涟漪／根影 | `oceantree_ripples_normal`、`oceantree_falling`、`watertree_pillar_ripples` 等 / active |
| [[reference/prefabs/waterplant_pollen_fx|海草花粉与诱饵云]] | 2 注册：海草喷云表现与落海诱饵云（混合：chum 标签引鱼＋周期投喂 chumpiece） | `waterplant_pollen_fx`、`waterplant_pollen_fx_ocean` / active |
| [[reference/prefabs/waterplant_destroy|暗影手与海上水植特效]] | 5 注册：暗影手、水流弹与刺球命中、水植升级、桅杆避雷针 | shadowhand_fx、waterstreak_burst、waterplant_burr_burst、mastupgrade_lightningrod_fx / active |
| [[reference/prefabs/hermitcrab_fx_small|寄居蟹隐士迁移、传送门与涟漪特效]] | 9 注册：隐士迁移、岩树、传送门、导弹与涟漪 | hermitcrab_fx_med、hermitcrab_fx_tall、missile_explosion_fx、tree_rock_chop、tree_rock_fall、vault_portal_fx、atrium_portal_fx、boss_ripple_fx / active |

## 航迹、船件、海冰与龙舟

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/boat_water_fx|船尾航迹水花]] | 船尾本地水花；漂移减速与双重自删 | — / active |
| [[reference/prefabs/boat_grass_erode|草／海带船侵蚀与垫料特效（boat_grass_erode 族）]] | 7 注册：沉没侵蚀四名成对同点（_water 18 帧淡出）、草／海带垫料屑（漏点 17 事件源＋钻船按 material 分名）、水草补丁到期断裂（动态拼名唯一名） | boat_grass_erode_water、boat_otterden_erode、boat_otterden_erode_water、fx_grass_boat_fluff、fx_kelp_boat_fluff、repaired_kelp_timeout_fx、船体侵蚀、垫料碎屑、草船沉没、水草补丁到期 / active |
| [[reference/prefabs/fx_boat_crackle|海洋平台破坏开裂与爆裂声画（fx_boat_crackle 族）]] | 5 注册：船沉 crackle/pop、码头拆除预警对、海冰 crackle（pop 在组外 fx_ice_pop）；boat_crackle 字段选名辨析与声音分层 | fx_boat_pop、fx_dock_crackle、fx_dock_pop、fx_ice_crackle、开裂预警、码头拆除声画、海冰破碎、船沉爆裂 / active |
| [[reference/prefabs/boat_mast_sink_fx|船件沉没与受击特效（boat_mast_sink_fx 族）]] | 八注册三系统：桅杆沉没随宿主 mast.sink_fx 选名（正常船沉 boat_death 出；单纯快速拆船与玩家锤拆不出）、防撞条按 bumpertype 动态拼名（碰撞吸收＋锤击＋降级出，升级不出；death 状态自身不出但致死碰撞可先出 hit）、小龙舟灯气球离场 | boat_malbatross_mast_sink_fx、mast_yotd_sink_fx、boat_bumper_hit_kelp、boat_bumper_hit_shell、boat_bumper_hit_crabking、boat_bumper_hit_yotd、miniboatlantern_loseballoon、桅杆沉没特效、防撞条受击特效、气球丢失特效 / active |
| [[reference/prefabs/ice_crack_grid_fx|海冰网格裂纹]] | 海冰瓦片裂纹；管理器铺设、查重与清除 | — / active |
| [[reference/prefabs/boatrace_start_flag|龙舟竞赛旗帜与烟花]] | 4 注册：名次旗/冠军烟花/检查点缎带（符号 override、退场协议统一）＋起跑浮标（dockjammer 禁修码头、无显式移除） | `boatrace_start_flag`、`boatrace_fireworks`、`boatrace_checkpoint_flag`、`boatrace_start_bobber` / active |

## 关联入口

天气雨滴、陆地和落石见 [[reference/prefabs/fx-world-index|天气、地形与生态痕迹]]。

返回 [[reference/prefabs/fx-index|FX 主题总索引]]、[[reference/prefabs/index|Prefab 索引]]。
