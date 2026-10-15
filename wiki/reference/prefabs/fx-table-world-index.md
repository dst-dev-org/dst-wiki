---
title: 通用表 FX·环境与物件
created: "2026-11-01"
updated: "2026-11-01"
type: index
tags: [overview]
---

# 通用表 FX·环境与物件

`fx.lua` 通用注册表中环境、物品与世界表现的同族页；角色、战斗与技能表现的各族见 [[reference/prefabs/fx-table-index|通用表 FX 索引]]。公共工厂协议只在 [[reference/mechanisms/generic-fx-factory|通用 FX 工厂]] 维护一份，各页链接不重复。返回 [[reference/prefabs/index|Prefab 索引]]。

通用表 504 个注册名均已有族页。

## 环境与物件（44 族 223 注册）

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/hermitcrab_fx_small|寄居蟹隐士迁移、传送门与涟漪特效]] | 9 注册：隐士迁移、岩树、传送门、导弹与涟漪 | hermitcrab_fx_med、hermitcrab_fx_tall、missile_explosion_fx、tree_rock_chop、tree_rock_fall、vault_portal_fx、atrium_portal_fx、boss_ripple_fx / active |
| [[reference/prefabs/fumarole_ember|喷气孔与查莉仪式圈特效]] | 4 注册：喷气孔余烬与烹饪、查莉圈出现 | fumarole_cook_fx、charlie_circle_spawn_fx、charlie_circle_spawn_ground_fx / active |
| [[reference/prefabs/cavehole_flick|洞穴弹飞、重置烟与表情效果]] | 8 注册：洞穴出口、宝箱重置、鼹鼠与表情特效 | slurper_respawn、pandorachest_reset、cavehole_flick_warn、mole_move_fx、emote_fx、tears、spawn_fx_ocean_static / active |
| [[reference/prefabs/junk_break_fx|退化、破裂与改造特效]] | 11 注册：草船与冰碎片、海冰、围栏、地皮、废料、箱子、打蜡与藤条 | degrade_fx_grass、degrade_fx_ice、fx_ice_pop、fence_rotator_fx、turf_smoke_fx、pull_smoke_fx、chestupgrade_stacksize_fx、chestupgrade_stacksize_taller_fx、beeswax_spray_fx、goo_vines_break_fx / active |
| [[reference/prefabs/sugarwood_leaf_fx|特殊树叶与死亡烟]] | 15 注册：树叶、催长与死亡烟；糖木事件包生成点缺口 | die_fx、pine_needles_chop、purple_leaves、purple_leaves_chop、sugarwood_leaf_fx_chop、sugarwood_leaf_withered_fx、sugarwood_leaf_withered_fx_chop、tree_petal_fx_chop、treegrowthsolution_use_fx、oceantree_leaf_fx_fall、oceantree_leaf_fx_chop、palmcone_leaf_fx_tall、palmcone_leaf_fx_normal、palmcone_leaf_fx_short / active |
| [[reference/prefabs/moonstorm_glass_fx|月之装置破碎与月暴玻璃爆裂]] | 4 注册：月之装置破碎及月暴玻璃地面／空中碎裂 | moon_device_break_stage2、moon_device_break_stage3、moonstorm_glass_ground_fx / active |
| [[reference/prefabs/icespike_fx_1|月矿采掘、旧冰刺与雕像转变]] | 9 注册：月矿采掘、四旧冰刺与两层雕像转变；旧冰刺无活跃生成点 | mining_moonglass_fx、mining_charged_moonglass_fx、mining_crystal_fx、icespike_fx_2、icespike_fx_3、icespike_fx_4、statue_transition、statue_transition_2 / active |
| [[reference/prefabs/messagebottle_break_fx|消息瓶与歌贝特效]] | 4 注册：消息瓶破碎与歌贝按平台材质选落点；浮动特效未找到生成点 | messagebottle_bob_fx、singingshell_creature_rockfx、singingshell_creature_woodfx / active |
| [[reference/prefabs/waterplant_destroy|暗影手与海上水植特效]] | 5 注册：暗影手、水流弹与刺球命中、水植升级、桅杆避雷针 | shadowhand_fx、waterstreak_burst、waterplant_burr_burst、mastupgrade_lightningrod_fx / active |
| [[reference/prefabs/moon_altar_link_fx|月坛连接光效]] | 1 注册：月坛步进生成与非专用服月脉冲本地光点 | 月坛连接光效、月脉冲光点 / active |
| [[reference/prefabs/farm_plant_happy|作物表情与活动彩纸]] | 4 注册：作物压力、杂草照料与枕头战出界；牛年彩纸未找到生成点 | farm_plant_unhappy、yotb_confetti、pillowfight_confetti_fx / active |
| [[reference/prefabs/winters_feast_depletefood|冬季盛宴餐桌食物枯竭特效（winters_feast_depletefood）]] | 2 注册：开餐过程每名进食者 30% 概率延迟随机闪粒（1~5 号动画＋bloom）、食物耐久耗尽终局 burst（条目无声，终局音效由餐桌自播）；仅餐桌盛宴回调一处生成，落点不父挂 | winters_feast_food_depleted、冬季盛宴食物枯竭特效、盛宴餐桌特效、食物耗尽特效 / active |
| [[reference/prefabs/sinkhole_spawn_fx|地陷尘爆与落石预警族（sinkhole_spawn_fx 等三族）]] | 7 注册：地陷尘爆三名（蚁狮陷坑塌陷／修复、巨兽落坑、翻地补尘——farm_plow 被接管不出）、预警小尘三名（表面端袭击倒计时）、cavein_debris（洞穴端警告碎屑，非地震落石）；全落点式纯表现 | sinkhole_spawn_fx_2、sinkhole_spawn_fx_3、sinkhole_warn_fx_1、sinkhole_warn_fx_2、sinkhole_warn_fx_3、cavein_debris、地陷尘爆、落石预警、洞穴落石碎屑 / active |
| [[reference/prefabs/glass_fx|玻璃化、灰烬消散与睡眠炸弹爆裂（glass_fx 等三族）]] | 三条互不相关单消费链：glass_fx 沙晶点燃 22 帧后起爆、animover 才换玻璃产物；erode_ash 化石尖刺到期消散（moonspider 仅声明无生成）；sleepbomb_burst 睡眠炸弹命中爆裂（睡眠在 sleepcloud） | glass_fx、erode_ash、sleepbomb_burst、玻璃化特效、灰烬消散、睡眠炸弹爆裂 / active |
| [[reference/prefabs/ink_puddle_land|墨水与胆汁洼地（ink_puddle、bile_puddle 族）]] | 四注册：墨水／胆汁弹命中按 IsOnOcean 分海陆两档贴地洼底动画；溅起与洼底同点各自生成，上墨与 AOE 在投射物 onhit | ink_puddle_water、bile_puddle_land、bile_puddle_water、墨水洼、胆汁洼、命中洼地 / active |
| [[reference/prefabs/flotsam_puddle|冲滩浮渣伴生水洼与拾取碎裂（flotsam_puddle 族）]] | 两注册：钓获漂浮物冲滩后父挂伴生水洼（缩放随机朝向）、拾取时碎裂动画；水中分支与绞盘打捞不出水洼 | flotsam_break、浮渣水洼、冲滩水洼、浮渣碎裂 / active |
| [[reference/prefabs/boat_grass_erode|草／海带船侵蚀与垫料特效（boat_grass_erode 族）]] | 7 注册：沉没侵蚀四名成对同点（_water 18 帧淡出）、草／海带垫料屑（漏点 17 事件源＋钻船按 material 分名）、水草补丁到期断裂（动态拼名唯一名） | boat_grass_erode_water、boat_otterden_erode、boat_otterden_erode_water、fx_grass_boat_fluff、fx_kelp_boat_fluff、repaired_kelp_timeout_fx、船体侵蚀、垫料碎屑、草船沉没、水草补丁到期 / active |
| [[reference/prefabs/fx_boat_crackle|海洋平台破坏开裂与爆裂声画（fx_boat_crackle 族）]] | 5 注册：船沉 crackle/pop、码头拆除预警对、海冰 crackle（pop 在组外 fx_ice_pop）；boat_crackle 字段选名辨析与声音分层 | fx_boat_pop、fx_dock_crackle、fx_dock_pop、fx_ice_crackle、开裂预警、码头拆除声画、海冰破碎、船沉爆裂 / active |
| [[reference/prefabs/boat_mast_sink_fx|船件沉没与受击特效（boat_mast_sink_fx 族）]] | 八注册三系统：桅杆沉没随宿主 mast.sink_fx 选名（正常船沉 boat_death 出；单纯快速拆船与玩家锤拆不出）、防撞条按 bumpertype 动态拼名（碰撞吸收＋锤击＋降级出，升级不出；death 状态自身不出但致死碰撞可先出 hit）、小龙舟灯气球离场 | boat_malbatross_mast_sink_fx、mast_yotd_sink_fx、boat_bumper_hit_kelp、boat_bumper_hit_shell、boat_bumper_hit_crabking、boat_bumper_hit_yotd、miniboatlantern_loseballoon、桅杆沉没特效、防撞条受击特效、气球丢失特效 / active |
| [[reference/prefabs/splash_green|海面绿花与冲滩水洼（splash_green 族）]] | 四注册：三档按体型选名（GetCombatFxSize 映射）、生物／玩家冲滩差异、malbatross 死代码；washashore 水洼为地面层伴生 | splash_green_small、splash_green_large、washashore_puddle_fx、绿色水花、海面绿花、冲滩水洼 / active |
| [[reference/prefabs/splash_sink|沉没与单点水花（splash_sink 族）]] | 三注册：SinkEntity OCEAN 档协议与实体离场点、漩涡传送门入口／出口双落蓝染花、pigshrine 三触发泥花 | oceanwhirlportal_splash、mudpuddle_splash、沉没水花、漩涡传送门水花、泥坑水花 / active |
| [[reference/prefabs/crater_steam_fx1|温泉蒸汽（crater_steam_fx1）]] | 温泉两组编号蒸汽 9 名；浴弹状态选 crater、平时 slow，玻璃化停汽 | crater_steam_fx2、crater_steam_fx3、crater_steam_fx4、slow_steam_fx1～5、温泉蒸汽、月岛温泉蒸汽 / active |
| [[reference/prefabs/splash|落水水花（splash）]] | 五种水花；floater 冷热分支、青蛙回巢、splash_ocean 活跃生成点与雪花溅 | hot_splash、frogsplash、splash_ocean、splash_snow_fx、水花、热水花、青蛙水花、海面水花、雪花溅 / active |
| [[reference/prefabs/sand_puff|沙尘与暗影烟团（sand_puff）]] | 九个烟团；懒惰掠夺者拾取、懒人魔杖起落烟对、玫瑰眼镜燃料呈现、表演瞬移、拟态消散 | sand_puff_large_front、sand_puff_large_back、shadow_puff、shadow_puff_solid、shadow_puff_solid_large、shadow_puff_large_front、shadow_puff_large_back、itemmimic_puff、烟团、暗影烟团、传送烟 / active |
| [[reference/prefabs/green_leaves|树叶飘落与砍伐（green_leaves）]] | 四色八注册两动画；落叶树摇摆／砍伐、越冬树落叶与鸦年华广场树分支 | red_leaves、orange_leaves、yellow_leaves、green_leaves_chop、red_leaves_chop、orange_leaves_chop、yellow_leaves_chop、落叶、树叶飘落 / active |
| [[reference/prefabs/spawn_fx_small|生成烟雾（spawn_fx_small）]] | 出生离场烟雾五注册；玩家进出的 medium_static 暂停时钟与 tiny 无生成点 | spawn_fx_tiny、spawn_fx_medium、spawn_fx_medium_static、spawn_fx_small_high、生成烟雾、出生烟雾 / active |
| [[reference/prefabs/ink_splash|墨水弹溅落（ink_splash）]] | 墨水弹命中溅落；洼地按海陆分流与 inkable 上墨在投射物链路 | 墨水溅落、墨水弹特效 / active |
| [[reference/prefabs/bile_splash|胆汁弹溅落（bile_splash）]] | 胆汁弹命中溅落；半径 2 AOE 伤害与洼地分流由投射物结算 | 胆汁溅落、胆汁弹特效 / active |
| [[reference/prefabs/waterballoon_splash|水球溅落（waterballoon_splash）]] | 水球投射物命中与水球堆建成两路径；加湿／建造者沾湿在调用方，条目无 fn | 水球溅落、水球水花 / active |
| [[reference/prefabs/spat_splat_fx|唾沫溅斑（spat_splat_fx）]] | 五注册溅斑；spat 弹命中落斑（y=0）＋pinnable 默认碎裂列表四段量级（入参 ratio 被内部覆盖） | spat_splash_fx_full、spat_splash_fx_med、spat_splash_fx_low、spat_splash_fx_melted、唾沫特效、碎裂唾沫 / active |
| [[reference/prefabs/icing_splat_fx|糖霜溅斑（icing_splat_fx）]] | 五注册姜饼黏液弹溅斑；双命中分支落斑、pinnable:Stick 传 icing 专属列表（full→melted 顺序） | icing_splash_fx_full、icing_splash_fx_med、icing_splash_fx_low、icing_splash_fx_melted、糖霜特效、姜饼溅斑 / active |
| [[reference/prefabs/dirt_puff|作物挖掘烟（dirt_puff）]] | 单注册作物挖除烟；farm_plants 按 dig_fx 字段生成，与 small_puff 同 build 不同注册（无声多 FinalOffset） | 作物挖掘烟、挖除烟 / active |
| [[reference/prefabs/shovel_dirt|铲土飞尘（shovel_dirt）]] | 单注册铲挖飞尘；海狸啃咬 DIG 分支与冬牧兔人动作帧／5% 概率分支落点，woodie 仅声明 | 铲土飞尘、挖土特效 / active |
| [[reference/prefabs/mining_fx|采石碎屑（mining_fx）]] | 两注册采矿碎屑；PlayMiningFX 按标签选名（frozen→冰、月晶水晶→组外），玩家啃咬／猎犬月兔直落点等十余处 | mining_ice_fx、采矿碎屑、冰屑特效 / active |
| [[reference/prefabs/groundpound_fx|震地尘土（groundpound_fx）]] | 震地落点尘土 2 名：groundpounder 按环点在陆地非码头落 (x,0,z)、fxRings 控制外圈、海狸模式覆写换名 | werebeaver_groundpound_fx、震地尘土、震地特效 / active |
| [[reference/prefabs/lunarflame_puff_small|月焰消退烟团（lunarflame_puff_small）]] | 灭火烟团 12 注册（尺寸×高度）；水晶冠秃鹫进食燃烧尸体生成，复用附身座狼喷焰资源 | lunarflame_puff_tiny、lunarflame_puff_med、lunarflame_puff_large、lunarflame_puff_tiny_low、lunarflame_puff_tiny_high、lunarflame_puff_small_low、lunarflame_puff_small_high、lunarflame_puff_med_low、lunarflame_puff_med_high、lunarflame_puff_large_low、lunarflame_puff_large_high、月焰烟团 / active |
| [[reference/prefabs/leafpile_spawn_fx|落叶堆生成与碎裂特效（leafpile_spawn_fx）]] | 6 注册：放置与生长聚叶、瓦片失效／敲碎／清扫碎裂，红黄换 build 拼名；休眠门仅盖放置与碎裂，衰减／烧毁不出 fx | leafpile_shatter_fx、leafpile_red_spawn_fx、leafpile_red_shatter_fx、leafpile_yellow_spawn_fx、leafpile_yellow_shatter_fx、落叶堆特效、叶堆碎裂特效 / active |
| [[reference/prefabs/halloween_firepuff_1|万圣夜火焰绒与月绒烟（halloween_firepuff_1）]] | 7 注册：药水与 livingtree_root 被营火族取作燃料起焰烟（blueflame 分冷热、三选一、随火苗符号）＋月药水与月玻璃再生的月绒落点烟；纯表现 | halloween_firepuff_2、halloween_firepuff_3、halloween_firepuff_cold_1、halloween_firepuff_cold_2、halloween_firepuff_cold_3、halloween_moonpuff、万圣夜火焰绒、冷焰绒、月绒烟、药水燃料烟 / active |
| [[reference/prefabs/pumpkincarving_shatter_fx|南瓜雕刻碎裂动画（pumpkincarving_shatter_fx）]] | 雕刻定稿碎裂 1 名：pumpkincarvable／pumpkinhatcarvable 校验通过后落点，与换装代理是不同注册 | 南瓜雕刻碎裂、雕刻碎裂特效 / active |
| [[reference/prefabs/snowball_shatter_fx|雪球碎裂动画（snowball_shatter_fx）]] | 雪球原地消失碎裂 1 名：融尽、冬宴清场、雪人锤碎三路径；投掷命中是 splash_snow_fx | 雪球碎裂、雪球消失特效 / active |
| [[reference/prefabs/sanity_raise|影岩升降标记动画（sanity_raise）]] | 两注册黑石升降标记；影柱计时、梦魇岩三态、破损祭坛交互与场景脚本落点，名称不代表理智增减 | sanity_lower、理智升降动画、影柱升起特效、梦魇岩升降 / active |
| [[reference/prefabs/lightning_rod_fx|避雷针电弧动画（lightning_rod_fx）]] | 单注册充电电弧标记；dozap 三触发（雷击充电／日终／取电）落点生成，zap 声在避雷针本体 | 避雷针电弧、避雷针特效、避雷针动画 / active |
| [[reference/prefabs/bundle_unwrap|开包特效（bundle_unwrap）]] | 开包展开烟 17 名：bundle.lua 按 inventoryimage 拼名，UNWRAP 动作战利品落点生成；龙舟赛对 yotd 直落点复用，quagmire 两名仅依赖声明 | gift_unwrap、redpouch_unwrap、redpouch_yotp_unwrap、redpouch_yotc_unwrap、redpouch_yotb_unwrap、redpouch_yot_catcoon_unwrap、redpouch_yotr_unwrap、redpouch_yotd_unwrap、redpouch_yoth_unwrap、yotc_seedpacket_unwrap、yotc_seedpacket_rare_unwrap、carnival_seedpacket_unwrap、wetpouch_unwrap、hermit_bundle_unwrap、quagmire_seedpacket_unwrap、quagmire_crate_unwrap、礼物开包、红包特效 / active |
| [[reference/prefabs/balloon_pop_body|气球爆裂（balloon_pop_body）]] | 两注册按装备槽选头部／身体变体（未装备选名 nil 不生成）；骑乘仅解除激活 | balloon_pop_head、气球爆裂、气球爆炸特效 / active |
