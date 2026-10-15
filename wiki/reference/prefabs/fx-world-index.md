---
title: "FX · 天气、地形与生态痕迹"
created: "2026-11-04"
updated: "2026-11-07"
type: index
tags: [overview]
---

# FX · 天气、地形与生态痕迹

降雨、月暴、地陷、地面残留、树林与世界生态。

## 天气、闪电与月暴

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/lightning|闪电与雷声表现]] | 五注册：天气闪电、无动画雷声、远近雷声与月暴变体；阻挡、月暴与真实结算分工 | `thunder`、`thunder_close`、`thunder_far`、`moonstorm_lightning` / active |
| [[reference/prefabs/rain|天气降水表现]] | 七注册：雨雪花粉与洞穴降水，管理器驱动与池化回收 | `snow`、`pollen`、`lunarhail`、`lunarhaildrop`、`caverain`、`caveacidrain` / active |
| [[reference/prefabs/acidraindrop|酸雨落点烟与蘑菇树酸烟]] | 两注册：落点回池与酸滴符号循环烟 | acidsmoke_endless / active |
| [[reference/prefabs/acidsmoke_fx|酸雨烟雾特效]] | 酸雨侵蚀烟雾；组件调档与休眠清理 | — / active |
| [[reference/prefabs/raindrop|落点水花与呼气粒子]] | 2 注册：池化雨滴落点（animover 回池）与霜息呼气粒子 | `raindrop`、`frostbreath` / active |
| [[reference/prefabs/moonstorm_ground_lightning_fx|月风暴地面闪电]] | 本地月风暴闪电；链生节点与距离门禁 | — / active |
| [[reference/prefabs/moonstorm_glass_fx|月之装置破碎与月暴玻璃爆裂]] | 4 注册：月之装置破碎及月暴玻璃地面／空中碎裂 | moon_device_break_stage2、moon_device_break_stage3、moonstorm_glass_ground_fx / active |

## 地陷、热泉与落石预警

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/sinkhole_spawn_fx|地陷尘爆与落石预警族（sinkhole_spawn_fx 等三族）]] | 7 注册：地陷尘爆三名（蚁狮陷坑塌陷／修复、巨兽落坑、翻地补尘——farm_plow 被接管不出）、预警小尘三名（表面端袭击倒计时）、cavein_debris（洞穴端警告碎屑，非地震落石）；全落点式纯表现 | sinkhole_spawn_fx_2、sinkhole_spawn_fx_3、sinkhole_warn_fx_1、sinkhole_warn_fx_2、sinkhole_warn_fx_3、cavein_debris、地陷尘爆、落石预警、洞穴落石碎屑 / active |
| [[reference/prefabs/cavein_dust_low|落石尘与拾荒疯猪掉落物碎裂]] | 9 注册：cavein 高低两档尘（成阵/砸地触发链）＋daywalker2 七个掉落物碎裂（动态拼名、scraphat 掉落变体读档怪癖） | `cavein_dust_low`、`cavein_dust_high`、`daywalker2_object_break_fx` 等 / active |
| [[reference/prefabs/cavehole_flick|洞穴弹飞、重置烟与表情效果]] | 8 注册：洞穴出口、宝箱重置、鼹鼠与表情特效 | slurper_respawn、pandorachest_reset、cavehole_flick_warn、mole_move_fx、emote_fx、tears、spawn_fx_ocean_static / active |
| [[reference/prefabs/sand_puff|沙尘与暗影烟团（sand_puff）]] | 九个烟团；懒惰掠夺者拾取、懒人魔杖起落烟对、玫瑰眼镜燃料呈现、表演瞬移、拟态消散 | sand_puff_large_front、sand_puff_large_back、shadow_puff、shadow_puff_solid、shadow_puff_solid_large、shadow_puff_large_front、shadow_puff_large_back、itemmimic_puff、烟团、暗影烟团、传送烟 / active |
| [[reference/prefabs/crater_steam_fx1|温泉蒸汽（crater_steam_fx1）]] | 温泉两组编号蒸汽 9 名；浴弹状态选 crater、平时 slow，玻璃化停汽 | crater_steam_fx2、crater_steam_fx3、crater_steam_fx4、slow_steam_fx1～5、温泉蒸汽、月岛温泉蒸汽 / active |
| [[reference/prefabs/cave_vent_ground_fx|喷气孔地面特效]] | 喷气孔红晕；可存档，原版生成点未接线 | — / active |
| [[reference/prefabs/fumarole_ember|喷气孔与查理仪式圈特效]] | 4 注册：喷气孔余烬与烹饪、查理圈出现 | fumarole_cook_fx、charlie_circle_spawn_fx、charlie_circle_spawn_ground_fx / active |

## 地表痕迹、植物与云雾

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/ice_puddle|宿主附属的贴地与贴水 FX 族]] | 4 注册：冰面水洼、融冰水花、硝石晶簇与浮标涟漪；端／父挂／清理各异 | `ice_splash`、`nitre_formation`、`oceanfishinghook_ripple` / active |
| [[reference/prefabs/damp_trail|地面痕迹与瞬时提示的外围 FX 族]] | 3 注册：复活骸骨地面拖尾、肥料便便云与森林石化公告音 | `damp_trail`、`poopcloud`、`petrify_announce` / active |
| [[reference/prefabs/scorchedground|地面焦痕、跟随换装与附着状态 FX 族]] | 4 注册：焦痕贴片、暗影战斧跟随、思魂坛月瓣与孢子炸弹附着 | `shadow_battleaxe_fx`、`sisturn_moon_petal_fx`、`sporebomb` 等 / active |
| [[reference/prefabs/burntground|焦地渐隐与蜂蜜轨迹]] | 4 注册：焦地 netvar 64 tick 五日渐隐（persists 未关闭反例）、蜂蜜轨迹混合载体（FX 实体逐帧推减速） | `burntground`、`burntground_faded`、`honey_trail`、`honey_trail_shrouden_fx` / active |
| [[reference/prefabs/grasspartfx|小型一次性外围 FX 族]] | 8 注册：草屑、脆弱玫瑰、黑暗痕迹、鱼饵碎块、恶液吐物、鹿挣脱/生角与渔网溅水 | `charlierose`、`charlieresidue`、`deer_unshackle_fx` 等 / active |
| [[reference/prefabs/shadow_glob_fx|暗影地面痕特效]] | 暗影地面痕；去重标签与海面涟漪 | — / active |
| [[reference/prefabs/leafpile_spawn_fx|落叶堆生成与碎裂特效（leafpile_spawn_fx）]] | 6 注册：放置与生长聚叶、瓦片失效／敲碎／清扫碎裂，红黄换 build 拼名；休眠门仅盖放置与碎裂，衰减／烧毁不出 fx | leafpile_shatter_fx、leafpile_red_spawn_fx、leafpile_red_shatter_fx、leafpile_yellow_spawn_fx、leafpile_yellow_shatter_fx、落叶堆特效、叶堆碎裂特效 / active |
| [[reference/prefabs/sugarwood_leaf_fx|特殊树叶与死亡烟]] | 15 注册：树叶、催长与死亡烟；糖木事件包生成点缺口 | die_fx、pine_needles_chop、purple_leaves、purple_leaves_chop、sugarwood_leaf_fx_chop、sugarwood_leaf_withered_fx、sugarwood_leaf_withered_fx_chop、tree_petal_fx_chop、treegrowthsolution_use_fx、oceantree_leaf_fx_fall、oceantree_leaf_fx_chop、palmcone_leaf_fx_tall、palmcone_leaf_fx_normal、palmcone_leaf_fx_short / active |
| [[reference/prefabs/green_leaves|树叶飘落与砍伐（green_leaves）]] | 四色八注册两动画；落叶树摇摆／砍伐、越冬树落叶与鸦年华广场树分支 | red_leaves、orange_leaves、yellow_leaves、green_leaves_chop、red_leaves_chop、orange_leaves_chop、yellow_leaves_chop、落叶、树叶飘落 / active |
| [[reference/prefabs/miasma_cloud|瘴气]] | 瘴气主实体有玩法；源登记、火消退与视觉变体 | `miasma`、`miasma_cloud_visual`、`miasma_cloud_fx`、`miasma_ember_fx`、`miasma_cloud_fx_thin`、`miasma_cloud_fx_border`、`miasma_cloud_arenabordervisual` / active |
| [[reference/prefabs/lunar_goop_cloud_fx|牧者黏液雾云]] | 牧者雾云纯视觉；父级开关与粒子配置 | — / active |
| [[reference/prefabs/lunar_goop_trail_fx|牧者黏液拖尾]] | 牧者三段拖尾；duration、提前淡出与共享池 | — / active |

## 关联入口

海面、航迹与水花见 [[reference/prefabs/fx-water-index|水面、航海与船件]]，光源、火焰与声音代理见 [[reference/prefabs/fx-light-index|灯光、火焰与音效代理]]。雨滴与无雨区的版本专题见 [[reference/mechanisms/rain-fx-gating|雨滴 FX 与无雨区门控（build 26148036）]]。

返回 [[reference/prefabs/fx-index|FX 主题总索引]]、[[reference/prefabs/index|Prefab 索引]]。
