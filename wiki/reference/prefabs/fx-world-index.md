---
title: 特效 Prefab 索引·环境与世界
created: "2026-11-04"
updated: "2026-11-05"
type: index
tags: [overview]
---

# 特效 Prefab 索引·环境与世界

环境、天气、光源与宿主附属类特效知识页，按主题分组；生物／Boss 战特效见 [[reference/prefabs/fx-creature-index|特效 Prefab 索引·生物与Boss]]，角色技能、装备与杂项见 [[reference/prefabs/fx-index|特效 Prefab 索引]]。返回 [[reference/prefabs/index|Prefab 索引]]。

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/diseaseflies|病害蝇群表现]] | 病害组件生成的蝇群代理、本地动画与循环次数 | 病害蝇群 / active |
| [[reference/prefabs/grottopool_sfx|洞穴水池声音发射器]] | 本地声音载体、控制器配对及淡变 | 洞穴水池声音 / active |
| [[reference/prefabs/explode_small|小爆裂、换装烟与火药爆裂]] | 十注册：爆裂、换装烟、火药及爆竹 | explode_reskin、reskin_tool_brush_explode_fx、reskin_tool_bouquet_explode_fx、reskin_tool_wand_explode_fx、reskin_tool_toilet_explode_fx、explode_small_slurtle、explode_small_slurtlehole、explode_firecrackers、slingshotammo_gunpowder_explode / active |
| [[reference/prefabs/acidraindrop|酸雨落点烟与蘑菇树酸烟]] | 两注册：落点回池与酸滴符号循环烟 | acidsmoke_endless / active |
| [[reference/prefabs/abysspillar_fx|深渊石柱坍塌与重建特效]] | 坍塌与形成、实体柱恢复及组追踪 | 深渊石柱特效 / active |
| [[reference/prefabs/lightning|闪电与雷声表现]] | 五注册：天气闪电、无动画雷声、远近雷声与月暴变体；阻挡、月暴与真实结算分工 | `thunder`、`thunder_close`、`thunder_far`、`moonstorm_lightning` / active |
| [[reference/prefabs/rain|天气降水表现]] | 七注册：雨雪花粉与洞穴降水，管理器驱动与池化回收 | `snow`、`pollen`、`lunarhail`、`lunarhaildrop`、`caverain`、`caveacidrain` / active |
| [[reference/prefabs/wave_med|海面波浪与浪花]] | 七注册：网络浪实体与客户端波光／岸浪；推船、加湿与本地生成 | `wave_splash`、`wave_shimmer`、`wave_shimmer_med`、`wave_shimmer_deep`、`wave_shimmer_flood`、`wave_shore` / active |
| [[reference/prefabs/fire|火焰、火光与烟雾特效族]] | 15 注册：firefx 等级表、heater 热度与宿主燃烧链 | `campfirefire`、`character_fire`、`floatinglanternfire`、`thurible_smoke` 等 / active |
| [[reference/prefabs/cavelight_small|光源实体（洞穴、切斯特、读书、出生与中庭）]] | 13 注册：洞穴相位光、变身光柱、读书光与中庭灯 | `cavelight`、`cavelightmoon`、`chesterlight`、`booklight`、`spawnlight_multiplayer`、`atrium_light` / active |
| [[reference/prefabs/fence_gate_anim|宿主挂接的附属表现实体]] | 12 注册：门动画、椅背层、茶铺前层、垃圾堆侧片／蓝图、灯链与水草底座 | `wood_chair_back`、`hermitcrab_teashop_front`、`yots_lantern_light_chain` 等 / active |
| [[reference/prefabs/groundpoundring_fx|冲击环、爆点与碎块表现特效]] | 6 注册：震地环、火环、地刺爆裂、炸弹爆炸、碎块与眼睛炮塔命中音 | `firering_fx`、`ground_chunks_breaking`、`eye_charge_hit` 等 / active |
| [[reference/prefabs/ice_puddle|宿主附属的贴地与贴水 FX 族]] | 4 注册：冰面水洼、融冰水花、硝石晶簇与浮标涟漪；端／父挂／清理各异 | `ice_splash`、`nitre_formation`、`oceanfishinghook_ripple` / active |
| [[reference/prefabs/mushtree_tall_webbed_burntfx|生物残骸与植物／菌类次级表现]] | 6 注册：牧者残骸、藤蔓节、烧尽菌树、焦灌木侵蚀、梦魇地裂与大血滴 | `lunar_grazer_debris`、`nightmaregrowth_crack`、`minotaur_blood_big` 等 / active |
| [[reference/prefabs/madscience_lab_goop|实验台三层特效与粒子杂项]] | 6 注册：实验台 goop／fire／glow、森林 Mist 雾、防滑砂 buff 粒子与小猫躲藏 | `mist`、`nonslipgrit_buff_fx`、`kitcoon_hide_fx` 等 / active |
| [[reference/prefabs/moon_device_pillar|月装置支柱、顶盖、一阶段破碎与天体裂缝裂纹层]] | 4 注册：月亮虹吸器阶段支柱／顶盖／首次破碎与按月相显隐的裂纹层 | `moon_device_top`、`moon_fissure_fx` 等 / active |
| [[reference/prefabs/crabking_claw_shadow|地面贴片与投影阴影 FX 族]] | 6 注册：巨钳水下阴影／挥击弧、冰圈真实冻结、焦痕淡出、柱基座／裂口 | `crabking_claw_swipe_fx`、`deerclopseyeball_sentryward_fx`、`fused_shadeling_bomb_scorch` 等 / active |
| [[reference/prefabs/king_cane_fx|宿主持有与跟随的附属 FX 族]] | 7 注册：国王魔杖宝石/腐蚀、风铃装饰、牧者岩核、亮茄背层、月装置连接件、传送门掉落跟随与哈奇音乐地光 | `hermithouse_ornament_fx`、`moon_altar_link_contained`、`hutch_music_light_fx` 等 / active |
| [[reference/prefabs/wormlight|灯具与生物光源的附属光／FX 实体]] | 16 注册：蠕虫灯、矿灯、船灯、红灯笼、女王火炬、哈奇音乐地光与生物光 | `lanternlight`、`miniboatlanternlight`、`slurperlight`、`squideyelight` 等 / active |
| [[reference/prefabs/eye_charge|有表现的玩法载体（眼弹、诱鱼区、黏块、速爆弹）]] | 4 注册：炮塔眼弹、落海诱鱼区、可装瓶小黏块与速爆弹；伤害/定身/吸附归属分清 | `chum_aoe`、`gelblob_small_fx`、`fused_shadeling_quickfuse_bomb` / active |
| [[reference/prefabs/grasspartfx|小型一次性外围 FX 族]] | 8 注册：草屑、脆弱玫瑰、黑暗痕迹、鱼饵碎块、恶液吐物、鹿挣脱/生角与渔网溅水 | `charlierose`、`charlieresidue`、`deer_unshackle_fx` 等 / active |
| [[reference/prefabs/gestalt_trail|宿主附着与跟随的外围 FX 族]] | 6 注册：虚影拖尾、埋地垃圾层、换装跟随层、恶液背层、暗影切斯特旋涡与雪球发射器光晕 | `daywalker2_buried_fx`、`gestalt_cage_swap_fx`、`shadow_chester_swirl_fx` 等 / active |
| [[reference/prefabs/impact|命中与落点伤害表现及传送门投影 FX 族]] | 7 注册：命中贴片、吹箭弹道、垃圾球两段、裂隙改造者爆炸与两个传送门投影 | `houndstooth_proj`、`junkball_fall_fx`、`lunarrift_portal_shadow` 等 / active |
| [[reference/prefabs/ruins_cavein_obstacle_rubble_fx|废墟落石碎屑与实验室吊舱皮肤 FX]] | 2 注册：落石柱碎屑与研究台吊舱吸附皮肤 | `ruins_cavein_obstacle_rubble_fx`、`researchlab2_pod_fx` / active |
| [[reference/prefabs/raindrop|落点水花与呼气粒子]] | 2 注册：池化雨滴落点（animover 回池）与霜息呼气粒子 | `raindrop`、`frostbreath` / active |
| [[reference/prefabs/damp_trail|地面痕迹与瞬时提示的外围 FX 族]] | 3 注册：复活骸骨地面拖尾、肥料便便云与森林石化公告音 | `damp_trail`、`poopcloud`、`petrify_announce` / active |
| [[reference/prefabs/carnival_sparkle_bush|皮肤火星与随身连缀 FX 族]] | 4 注册：鸦年华皮肤火星、胡萝卜旋转残影、迷你风扇叶片与小蛾子发光 buff | `carrot_spinner`、`fan_wheel`、`critterbuff_lunarmoth` 等 / active |
| [[reference/prefabs/oceantree_ripples_short|海树波纹、根须、倒伏与落叶表现]] | 10 注册：三档涟漪／根影、倒伏树干与柱体涟漪／根影／落叶 | `oceantree_ripples_normal`、`oceantree_falling` 等 / active |
| [[reference/prefabs/pocketwatch_warp_marker|怀表与传送门附属表现 FX 族]] | 5 注册：warp／recall 标记与传送门 overlay／underlay／exit FX | `pocketwatch_recall_marker`、`pocketwatch_portal_exit_fx` 等 / active |
| [[reference/prefabs/sleepcloud|睡眠云与孢子云]] | 5 注册：范围催眠云（含月版）与范围伤害/腐败云，各带 7 片 overlay | `sleepcloud_lunar`、`sporecloud_overlay` 等 / active |
| [[reference/prefabs/staff_lunarplant_fx|装备与物品持握跟随层 FX 族]] | 6 注册：亮茄魔杖／剑刃、暗影香炉、弹弓部件、雪人帽与滚雪球拖影 | `sword_lunarplant_blade_fx`、`thuriblebody`、`snowmanhat_fx` 等 / active |
| [[reference/prefabs/shadow_trap|暗影陷阱与减速表现]] | 2 注册：法术陷阱本体与目标减速／恐慌贴片 | `shadow_trap_debuff_fx` / active |
| [[reference/prefabs/scorchedground|地面焦痕、跟随换装与附着状态 FX 族]] | 4 注册：焦痕贴片、暗影战斧跟随、思魂坛月瓣与孢子炸弹附着 | `shadow_battleaxe_fx`、`sisturn_moon_petal_fx`、`sporebomb` 等 / active |
| [[reference/prefabs/fx_book_mount|书页覆盖与骑乘挂载特效族]] | 12 注册：六种读书效果的前后层与骑乘变体 | `fx_tentacles_under_book`、`fx_lightning_over_book` 等 / active |
| [[reference/prefabs/goosplash|黏液飞溅变体族]] | 9 注册：八种定号飞溅与随机基名；脚本层无生成点（事件数据未覆盖） | `goosplash1`～`goosplash8`、`goosplash` / active |
| [[reference/prefabs/nightmarelightfx|暗影光源、幻灵头与盘旋秃鹫 FX 族]] | 8 注册：梦魇灯座/裂缝父挂动画、虚影头跟随与秃鹫盘旋影子 | `gestalt_head`、`circlingbuzzard_lunar` 等 / active |
| [[reference/prefabs/boat_water_fx|船尾航迹水花]] | 船尾本地水花；漂移减速与双重自删 | — / active |
| [[reference/prefabs/cave_vent_ground_fx|喷气孔地面特效]] | 喷气孔红晕；可存档，原版生成点未接线 | — / active |
| [[reference/prefabs/ice_crack_grid_fx|海冰网格裂纹]] | 海冰瓦片裂纹；管理器铺设、查重与清除 | — / active |
| [[reference/prefabs/lunar_goop_cloud_fx|牧者黏液雾云]] | 牧者雾云纯视觉；父级开关与粒子配置 | — / active |
| [[reference/prefabs/lunar_goop_trail_fx|牧者黏液拖尾]] | 牧者三段拖尾；duration、提前淡出与共享池 | — / active |
| [[reference/prefabs/lunarrift_crystal_spawn_fx|裂隙晶体生成烟尘]] | 晶体生成烟尘；专服无内建自删 | — / active |
| [[reference/prefabs/miasma_cloud|瘴气]] | 瘴气主实体有玩法；源登记、火消退与视觉变体 | `miasma`、`miasma_cloud_visual`、`miasma_cloud_fx`、`miasma_ember_fx`、`miasma_cloud_fx_thin`、`miasma_cloud_fx_border`、`miasma_cloud_arenabordervisual` / active |
| [[reference/prefabs/superjump_fx|超级跳落点特效]] | 超级跳落点表现；目标定位、碎屑与音效 | `superjump_debris`、`player_superjump`、`superjump` / active |
| [[reference/prefabs/pumpkincarving_swap_fx|南瓜雕刻换装代理]] | UGC 数据代理；南瓜切口／雪人装饰贴符号 | `snowmandecorating_swap_fx`、`ugc_swap_fx` / active |
| [[reference/prefabs/collapse_big|建筑倒塌特效]] | 建筑倒塌大小两款；SetMaterial 选择破碎声 | `collapse_small`、`structure_collapse_fx` / active |
| [[reference/prefabs/ghost_transform_overlay_fx|幽灵复活转生光效]] | 复活叠层动画；闪光时间线与队列退场 | — / active |
| [[reference/prefabs/lavaarena_creature_teleport_small_fx|熔炉生物传送出生特效]] | 熔炉六种出场表现；音量衰减与出生烟 | `lavaarena_creature_teleport_medium_fx`、`lavaarena_spawnerdecor_fx_small`、`lavaarena_spawnerdecor_fx_1`、`lavaarena_spawnerdecor_fx_2`、`lavaarena_spawnerdecor_fx_3`、`lavaarena_creature_spawn_fx` / active |
| [[reference/prefabs/cannon_aoe_range_fx|船炮瞄准范围圈特效]] | 本地船炮／球杆瞄准圈；调用方负责清理 | `cannon_reticule_fx` / active |
| [[reference/prefabs/rope_bridge_fx|绳桥与藤桥桥面表现]] | 3 注册：管理器按瓦片生成的桥面贴片（可通行面是瓦片非 FX；绳桥双绳本地重建、藤桥装饰子体、创建即排定 18 秒拆除） | `rope_bridge_fx`、`vine_bridge_fx`、`vine_bridge_decor_fx` / active |
| [[reference/prefabs/fossilspike_base|化石骨刺地面基座层]] | 2 注册：Stalker 两代尖刺的引擎父挂基座（本体混合结构辨析、两代收尾不对称） | `fossilspike_base`、`fossilspike2_base` / active |
| [[reference/prefabs/lunarrift_terraformer_visual|裂隙地皮改造视觉层与河灯影层]] | 2 注册：隐形管理器的 23 秒阶段演出层（earlyexit 门控）与天灯引擎父挂影层（成对动画同步） | `lunarrift_terraformer_visual`、`floatinglantern_shadow` / active |
| [[reference/prefabs/shatter|一次性碎裂与爆裂表现族]] | 8 注册：冻结碎裂等级壳＋本地动画双端代理、蛛网溅落（无生成点诚实缺口）、贝壳小生物、木牌碎裂、南瓜炸弹确定性 PRNG 碎块、风滚草破碎、工具损坏代理、亮茄爆裂 | `shatter`、`splash_spiderweb`、`brokentool` 等 / active |
| [[reference/prefabs/cavein_dust_low|落石尘与拾荒疯猪掉落物碎裂]] | 9 注册：cavein 高低两档尘（成阵/砸地触发链）＋daywalker2 七个掉落物碎裂（动态拼名、scraphat 掉落变体读档怪癖） | `cavein_dust_low`、`cavein_dust_high`、`daywalker2_object_break_fx` 等 / active |
| [[reference/prefabs/terrariumchest_fx|显眼箱子微光与传送塔沙柱]] | 2 注册：恐怖之眼门禁的宝箱双层微光（fx 引用即 SHIMMER 标记）与传送沙柱（唯一带 ObstaclePhysics 的 FX，KillFX＋35 帧兜底） | `terrariumchest_fx`、`townportalsandcoffin_fx` / active |

| [[reference/prefabs/waterplant_pollen_fx|海草花粉与诱饵云]] | 2 注册：海草喷云表现与落海诱饵云（混合：chum 标签引鱼＋周期投喂 chumpiece） | `waterplant_pollen_fx`、`waterplant_pollen_fx_ocean` / active |
| [[reference/prefabs/trap_fumarole_burn_fx|喷气孔陷阱与藤蔓底座]] | 2 注册：DECOR 标签异例（无 FX）、组件复用同实体、藤蔓底座复用影柱 bank（宿主 DespawnTrap 先挪位再 KillFX） | `trap_fumarole_burn_fx`、`trap_vines_base_fx` / active |
| [[reference/prefabs/boatrace_start_flag|龙舟竞赛旗帜与烟花]] | 4 注册：名次旗/冠军烟花/检查点缎带（符号 override、退场协议统一）＋起跑浮标（dockjammer 禁修码头、无显式移除） | `boatrace_start_flag`、`boatrace_fireworks`、`boatrace_checkpoint_flag`、`boatrace_start_bobber` / active |
| [[reference/prefabs/vault_compass_visual|宝库设施表现]] | 4 注册：罗盘指针三层链、净化器雾效、地面纹样（18 处生成、未置 persists 反例）、宝库火炬冷火三档（无 heater） | `vault_compass_visual`、`vault_decon_mister_fx`、`vault_ground_pattern_fx`、`vault_torch_flame` / active |
| [[reference/prefabs/mushtree_tall_burntfx|蘑菇树焦壳与绽放焦壳]] | 7 注册：动态拼名全展开（tall/medium/small×普通/绽放＋月树）；烧尽与读档两条触发、双兜底清理 | `mushtree_tall_burntfx`、`mushtree_tall_bloom_burntfx`、`mushtree_moon_burntfx` 等 / active |
| [[reference/prefabs/moon_altar_break|月坛破碎演出]] | 3 注册：月亮装置 9.5 秒破碎链清场生成（锤毁不生成）；crown/claw 为动画库名与祭坛 prefab 名错位辨析 | `moon_altar_break`、`moon_altar_claw_break`、`moon_altar_crown_break` / active |
| [[reference/prefabs/attackfx_handpillow_petals|枕头命中碎屑与作物挖起]] | 7 注册：四材质枕头碎屑（动态拼名、伤害 0 击退分流的混合载体）＋三档作物挖起尘（small_fx 两处清单外拖尾生成） | `attackfx_handpillow_petals`、`plant_dug_small_fx` 等 / active |
| [[reference/prefabs/burntground|焦地渐隐与蜂蜜轨迹]] | 4 注册：焦地 netvar 64 tick 五日渐隐（persists 未关闭反例）、蜂蜜轨迹混合载体（FX 实体逐帧推减速） | `burntground`、`burntground_faded`、`honey_trail`、`honey_trail_shrouden_fx` / active |
| [[reference/prefabs/archive_ambient_sfx|远古档案机关声画与基座]] | 5 注册：警戒光球无 AnimState、双层贴地面贴片、环境声区域世界生成铺设、探测仪底座持久实体（arhivepoweron 源码拼写陷阱） | `archive_security_pulse_sfx`、`archive_switch_pad`、`archive_switch_base`、`archive_ambient_sfx`、`archive_resonator_base` / active |
| [[reference/prefabs/atrium_gate_activatedfx|远古大门声画三兄弟]] | 4 注册：activatedfx 两态切换（KillFX 兜底）＋makesfx 同构三兄弟（16.5 秒脉冲、爆炸、地穴战线；地区门控辨析） | `atrium_gate_activatedfx`、`atrium_gate_pulsesfx`、`atrium_gate_explodesfx`、`grotto_war_sfx` / active |
| [[reference/prefabs/balloon_held_child|气球持握两层与彩带表现]] | 4 注册：联网代理＋非联网贴手客户对象（ballon 通道拼写陷阱）、彩带云混合载体（逐秒 AddDebuff、未置 persists 反例）、彩带气球 | `balloon_held_child`、`balloon_held_child_client`、`balloonparty_confetti_cloud`、`balloonparty_confetti_balloon` / active |
| [[reference/prefabs/positronbeam_back|月亮石充能光束]] | 3 注册：back/front/pulse 三件套（满月充能链、KillFX 双语义、StopFX 笔误陷阱） | `positronbeam_back`、`positronbeam_front`、`positronpulse` / active |
| [[reference/prefabs/terrarium_fx|盒中泰拉召唤光柱]] | 1 注册：夜间召唤光柱，激活／循环／退场三段，随宿主父挂，10 帧后移除 | `terrarium_fx` / active |
| [[reference/prefabs/yotr_fightring_torch|枕头大战坑与兔人火炬]] | 4 注册＋placer：擂台、套装、钟与 8 支事件驱动火炬；火炬非持久 FX 壳 | `yotr_fightring_torch`、`yotr_fightring` 等 / active |
返回 [[reference/prefabs/index|Prefab 索引]]。
