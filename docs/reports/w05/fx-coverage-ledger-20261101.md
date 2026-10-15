# FX 覆盖台账与任务检查点（2026-11-01）

Steam buildid：**26335394**（`raw/current/BUILD.txt` 实读）；知识仓 HEAD：e271fae（开工时，未提交工作区）。本文件是本轮"通用表＋专属＋外围"FX 扩库的逐注册覆盖台账与任务检查点；逐注册原始登记沿用 [fx-table-coverage-20261030.csv](fx-table-coverage-20261030.csv)（504 名，口径与来源行）与 [fx-other-candidates-20261030.csv](fx-other-candidates-20261030.csv)（外围 380 文件），本台账按族汇总状态，不复制来源行。

## 口径与方法

- **通用表**：`fx.lua` 加载器返回的注册名，504 个（453 静态＋51 动态展开），无重名。已覆盖判定＝注册名出现在既有 Prefab 页 frontmatter aliases **且**正文（扫描脚本 output/tmp/fx_ledger_scan.py，词边界匹配；`slow_steam_fx2/3/4` 三名正文以 `slow_steam_fx1`～`slow_steam_fx5` 简写承载，别名与变体表已核，计入已覆盖）。扫描结果：160 名别名＋正文双证、3 名别名＋简写、**341 名未覆盖**，与 PLAN"32 族 163 注册"一致。
- **专属工厂**：`prefabs/*_fx*.lua` 87 文件的 75 页已于 2026-10-30 交付（[[reference/prefabs/fx-index]]）；宿主文件内 `_fx` 注册名按既有决定留宿主页承载。本轮不重开，只在外围核对时引用。
- **外围候选**：380 文件为筛选线索；本轮任务=逐文件核验"确属特效注册名"后归入覆盖/排除，不把文件数当缺页数。核验前状态一律 pending，不预写排除。
- **状态值**：`covered-existing`（已有页覆盖）／`pending`（本轮待建）／`claimed`（他方认领）／`generated`（已生成待验）／`accepted`（已验收）／`excluded`（有证据排除）／`blocked`（阻塞，附原因）。
- 分族是按共享机制的**选题分组**，每族建页时仍须完整核验生成点、消费者与边界；族内页面命名以实际注册名落名，可按机制再分页。

## 已覆盖：通用表 163 名（32 页，covered-existing）

| 页面（fx-table-index 已登记） | 注册数 |
|---|---:|
| werebeaver_transform_fx（变身烟 8）＋weregoose_splash（10） | 18 |
| bundle_unwrap（17）＋oldager_become_younger_front_fx（6） | 23 |
| slingshot_ice_aoe_fx（4）＋groundpound_fx（2）＋spider_heal_fx（4） | 10 |
| crater_steam_fx1（9）＋splash（5）＋sand_puff（9）＋green_leaves（8）＋spawn_fx_small（5） | 36 |
| shock_fx（6）＋planar_hit_fx（2）＋slingshotammo_hitfx_rock（18）＋lunarflame_puff_small（12）＋wx78_shield_full（6） | 44 |
| shadow_shield1（6）＋sanity_raise（2）＋lightning_rod_fx（1） | 9 |
| ink_splash／bile_splash／waterballoon_splash／balloon_pop_body（2）／spat_splat_fx（5）／icing_splat_fx（5） | 15 |
| small_puff／charlie_snap（2）／dirt_puff／maxwell_smoke／shovel_dirt／mining_fx（2） | 7 |
| 合计 | **163** |

另有非通用表承载：electrocute_fx 页覆盖通用表 `shock_arc_fx`（已计入上表 shock_fx 族）、专属工厂 75 页见 fx-index。

## 待覆盖：通用表 341 名分族台账

| 族 | 主题 | 注册名 | 数 | 目标页（建议，可按机制调整） | 状态 |
|---|---|---|---:|---|---|
| F01 | 万圣夜落叶堆 spawn/shatter | leafpile_spawn_fx, leafpile_shatter_fx, leafpile_red_spawn_fx, leafpile_red_shatter_fx, leafpile_yellow_spawn_fx, leafpile_yellow_shatter_fx | 6 | leafpile_spawn_fx | accepted（批1） |
| F02 | 万圣夜火绒与月绒 | halloween_firepuff_1, halloween_firepuff_2, halloween_firepuff_3, halloween_firepuff_cold_1, halloween_firepuff_cold_2, halloween_firepuff_cold_3, halloween_moonpuff | 7 | halloween_firepuff_1 | accepted（批1） |
| F03 | 投掷物碎裂对 | pumpkincarving_shatter_fx, snowball_shatter_fx | 2 | pumpkincarving_shatter_fx ＋ snowball_shatter_fx（机制无关拆两页） | accepted（批1） |
| F04 | 幽灵灵药（温蒂／阿比盖尔，非伯尼） | ghostflower_spirit1_fx, ghostflower_spirit2_fx, ghostlyelixir_slowregen_fx, ghostlyelixir_fastregen_fx, ghostlyelixir_shield_fx, ghostlyelixir_attack_fx, ghostlyelixir_speed_fx, ghostlyelixir_retaliation_fx, ghostlyelixir_revive_fx, ghostlyelixir_shadow_fx, ghostlyelixir_lunar_fx, ghostlyelixir_slowregen_dripfx, ghostlyelixir_fastregen_dripfx, ghostlyelixir_shield_dripfx, ghostlyelixir_attack_dripfx, ghostlyelixir_speed_dripfx, ghostlyelixir_retaliation_dripfx, ghostlyelixir_revive_dripfx, ghostlyelixir_shadow_dripfx, ghostlyelixir_lunar_dripfx, ghostlyelixir_player_slowregen_fx, ghostlyelixir_player_fastregen_fx, ghostlyelixir_player_shield_fx, ghostlyelixir_player_attack_fx, ghostlyelixir_player_speed_fx, ghostlyelixir_player_retaliation_fx, ghostlyelixir_player_revive_fx, ghostlyelixir_player_shadow_fx, ghostlyelixir_player_lunar_fx, ghostlyelixir_player_slowregen_dripfx, ghostlyelixir_player_fastregen_dripfx, ghostlyelixir_player_shield_dripfx, ghostlyelixir_player_attack_dripfx, ghostlyelixir_player_speed_dripfx, ghostlyelixir_player_retaliation_dripfx, ghostlyelixir_player_revive_dripfx, ghostlyelixir_player_shadow_dripfx, ghostlyelixir_player_lunar_dripfx | 38 | ghostlyelixir_slowregen_fx（36）＋ghostflower_spirit1_fx（灵体 2 拆页，与灵药机制无关） | accepted（批2） |
| F05 | 战歌与武神之魂 | wathgrithr_spirit, battlesong_attach, battlesong_loop, battlesong_detach, battlesong_durability_fx, battlesong_healthgain_fx, battlesong_sanitygain_fx, battlesong_sanityaura_fx, battlesong_fireresistance_fx, battlesong_shadowaligned_fx, battlesong_lunaraligned_fx, battlesong_instant_electric_fx, battlesong_instant_taunt_fx, battlesong_instant_panic_fx | 14 | wathgrithr_spirit | accepted（批2） |
| F06 | 旺达怀表 | wanda_attack_pocketwatch_old_fx, wanda_attack_pocketwatch_normal_fx, wanda_attack_shadowweapon_old_fx, wanda_attack_shadowweapon_normal_fx, pocketwatch_heal_fx, pocketwatch_heal_fx_mount, pocketwatch_ground_fx | 7 | wanda_attack_pocketwatch_old_fx | accepted（批2） |
| F07 | 相位认同与牛变身 | attune_out_fx, attune_in_fx, attune_ghost_in_fx, beefalo_transform_fx | 4 | attune_out_fx（3）＋beefalo_transform_fx（拆页，驯化变身与调谐无关） | accepted（批3） |
| F08 | 海面水花 | ocean_splash_med1, ocean_splash_med2, ocean_splash_small1, ocean_splash_small2, ocean_splash_ripple1, ocean_splash_ripple2, ocean_splash_swim1, ocean_splash_swim2 | 8 | ocean_splash_med1 | accepted（批3） |
| F09 | 水面杂项水花 | mudpuddle_splash, slide_puff, splash_sink, washashore_puddle_fx, splash_green_small, splash_green, splash_green_large, round_puff_fx_sm, round_puff_fx_lg, round_puff_fx_hi, oceanwhirlportal_splash | 11 | splash_green（绿花＋水洼）＋splash_sink（沉没＋传送门＋泥花）＋round_puff_fx_sm（圆烟＋滑行烟） | accepted（批3） |
| F10 | 木屑钻屑 | wood_splinter_jump, wood_splinter_drill | 2 | round_puff_fx_sm（并入，jump 为无生成点诚实缺口） | accepted（批3） |
| F11 | 船体侵蚀与垫料 | boat_grass_erode, boat_grass_erode_water, boat_otterden_erode, boat_otterden_erode_water, fx_grass_boat_fluff, fx_kelp_boat_fluff, fx_dock_crackle, fx_dock_pop, fx_ice_crackle, fx_boat_crackle, fx_boat_pop, repaired_kelp_timeout_fx | 12 | boat_grass_erode（7）＋fx_boat_crackle（破坏声画 5，姊妹页） | accepted（批4） |
| F12 | 船件损坏 | boat_mast_sink_fx, boat_malbatross_mast_sink_fx, mast_yotd_sink_fx, boat_bumper_hit_kelp, boat_bumper_hit_shell, boat_bumper_hit_crabking, boat_bumper_hit_yotd, miniboatlantern_loseballoon | 8 | boat_mast_sink_fx | accepted（批4） |
| F13 | 船炮与迫击炮 | cannonball_used, mortarball_used, mortarball_used_wood, mortarball_used_ice | 4 | cannonball_used | accepted（批4） |
| F14 | 水洼与浮渣 | ink_puddle_land, ink_puddle_water, bile_puddle_land, bile_puddle_water, flotsam_puddle, flotsam_break | 6 | ink_puddle_land（4）＋flotsam_puddle（2，父挂伴生机制分页） | accepted（批5） |
| F15 | 鱼人水花 | merm_king_splash, merm_splash, merm_spawn_fx | 3 | merm_splash | accepted（批5） |
| F16 | 蜂蜜蜂群与病害 | honey_splash, bee_poof_big, bee_poof_small, disease_puff | 4 | bee_poof_big（disease_puff 无生成点诚实缺口：病害系统休眠、实际特效 diseaseflies） | accepted（批5） |
| F17 | 蟹王 | crab_king_bubble1, crab_king_bubble2, crab_king_bubble3, crab_king_waterspout, crab_king_shine, crab_king_icefx, crabking_ring_fx | 7 | crab_king_bubble1（icefx 无活跃生成点诚实缺口；炮击冒泡重排死链入页） | accepted（批5） |
| F18 | 天坑与塌陷 | sinkhole_spawn_fx_1, sinkhole_spawn_fx_2, sinkhole_spawn_fx_3, sinkhole_warn_fx_1, sinkhole_warn_fx_2, sinkhole_warn_fx_3, cavein_debris, glass_fx, erode_ash, sleepbomb_burst | 10 | sinkhole_spawn_fx（7）＋glass_fx（杂项三族归档页 3） | accepted（批6） |
| F19 | 暴食传送门与盐盘 | quagmire_portal_player_fx, quagmire_portal_playerdrip_fx, quagmire_portal_player_splash_fx, quagmire_salting_plate_fx, quagmire_salting_bowl_fx | 5 | quagmire_portal_player_fx（生成点在事件数据包，快照不含——诚实缺口入页） | accepted（批6） |
| F20 | 熔炉出场与火系 | lavaarena_portal_player_fx, lavaarena_player_revive_from_corpse_fx, ember_short_fx, lavaarena_creature_teleport_smoke_fx_1, lavaarena_creature_teleport_smoke_fx_2, lavaarena_creature_teleport_smoke_fx_3, shadowstrike_slash_fx, shadowstrike_slash2_fx, firesplash_fx, tauntfire_fx, attackfire_fx, vomitfire_fx | 12 | lavaarena_portal_player_fx（熔炉流程 6）＋firesplash_fx（蜻蜓龙火系＋暗影斩击 6，勘误非熔炉敌人） | accepted（批6） |
| F21 | 消息瓶与歌贝 | messagebottle_break_fx, messagebottle_bob_fx, singingshell_creature_rockfx, singingshell_creature_woodfx | 4 | messagebottle_break_fx | accepted（批8） |
| F22 | 暗影手与水植 | shadowhand_fx, waterstreak_burst, waterplant_burr_burst, waterplant_destroy, mastupgrade_lightningrod_fx | 5 | waterplant_destroy | accepted（批8） |
| F23 | 暗影传送与宝盒 | shadow_teleport_in, shadow_teleport_out, spore_moon_coughout, archive_lockbox_player_fx | 4 | shadow_teleport_in | accepted（批8） |
| F24 | 月坛连接 | moon_altar_link_fx | 1 | moon_altar_link_fx | accepted（批8） |
| F25 | 作物表情与彩纸 | farm_plant_happy, farm_plant_unhappy, yotb_confetti, pillowfight_confetti_fx | 4 | farm_plant_happy | accepted（批8） |
| F26 | 狂欢节 | carnival_confetti_fx, carnival_sparkle_fx, carnival_streamer_fx, carnival_unwrap_fx, carnival_unwrap_fx_s2, carnival_unwrap_fx_s3, carnivalgame_shooting_projectile_fx | 7 | carnival_confetti_fx | accepted（批7） |
| F47 | 冬季盛宴食物 | winters_feast_depletefood, winters_feast_food_depleted | 2 | winters_feast_depletefood | accepted（批7） |
| F48 | 读书施法 | fx_book_moon, fx_book_moon_mount, fx_book_research_station, fx_book_research_station_mount, fx_book_temperature, fx_book_temperature_mount, fx_book_bees, fx_book_fire, fx_book_fire_mount, fx_book_light, fx_book_light_upgraded, fx_book_birds, fx_book_birds_mount, fx_book_sleep, fx_book_sleep_mount, fx_book_rain, fx_book_rain_mount, fx_book_fish | 18 | fx_book_moon（complex 一页：路径 A/B 两条生成链 11 本 def 全列） | accepted（批7） |
| F27 | 天体英雄 | alterguardian_spike_breakfx, alterguardian_spintrail_fx, alterguardian_phase1fallfx, alterguardian_phase3trappst, moon_geyser_explode, moonpulse_fx, moonpulse2_fx | 7 | alterguardian_spintrail_fx | accepted（批9） |
| F28 | 月之装置与月暴玻璃 | moon_device_break_stage2, moon_device_break_stage3, moonstorm_glass_ground_fx, moonstorm_glass_fx | 4 | moonstorm_glass_fx | accepted（批9） |
| F29 | 月矿冰刺与雕像 | mining_moonglass_fx, mining_charged_moonglass_fx, mining_crystal_fx, icespike_fx_1, icespike_fx_2, icespike_fx_3, icespike_fx_4, statue_transition, statue_transition_2 | 9 | icespike_fx_1 | accepted（批9） |
| F30 | 树叶族补充 | die_fx, pine_needles_chop, purple_leaves, purple_leaves_chop, sugarwood_leaf_fx, sugarwood_leaf_fx_chop, sugarwood_leaf_withered_fx, sugarwood_leaf_withered_fx_chop, tree_petal_fx_chop, treegrowthsolution_use_fx, oceantree_leaf_fx_fall, oceantree_leaf_fx_chop, palmcone_leaf_fx_tall, palmcone_leaf_fx_normal, palmcone_leaf_fx_short | 15 | sugarwood_leaf_fx | accepted（批10） |
| F31 | 温度环与 WX78 | dr_warm_loop_1, dr_warm_loop_2, dr_warmer_loop, dr_hot_loop, wx78_heat_steam, wx78_musicbox_fx | 6 | dr_warm_loop_1 | accepted（批9） |
| F32 | 施法失败与月化与薇洛火 | fire_fail_fx, wormwood_lunar_transformation_finish, spell_fire_throw, willow_shadow_fire_explode | 4 | spell_fire_throw | accepted（批10） |
| F33 | 暗影 threat 与诅咒 | fused_shadeling_spawn_fx, dreadstone_spawn_fx, shadowthrall_parasite_attach_poof_fx, beef_bell_shadow_cursefx | 4 | dreadstone_spawn_fx | accepted（批10） |
| F34 | WAGpunk 蒸汽 | wagpunksteam_hat_up, wagpunksteam_hat_down, wagpunksteam_armor_up, wagpunksteam_armor_down | 4 | wagpunksteam_hat_up | accepted（批10） |
| F35 | 无人机与玩家力场 | wagdrone_rolling_collide_small_fx, wagdrone_rolling_collide_med_fx, elixir_player_forcefield | 3 | wagdrone_rolling_collide_small_fx | accepted（批10） |
| F36 | 退化破裂与改造 | degrade_fx_grass, degrade_fx_ice, fx_ice_pop, fence_rotator_fx, turf_smoke_fx, pull_smoke_fx, junk_break_fx, chestupgrade_stacksize_fx, chestupgrade_stacksize_taller_fx, beeswax_spray_fx, goo_vines_break_fx | 11 | junk_break_fx | accepted（批11） |
| F37 | 沃特改造与暗影鱼人 | wurt_swamp_terraform_fx, wurt_terraformer_fx_shadow, wurt_terraformer_fx_lunar, wurt_water_splash_1, wurt_water_splash_2, wurt_water_splash_3, shadow_merm_spawn_poof_fx, shadow_merm_smacked_poof_fx | 8 | wurt_terraformer_fx_shadow | accepted（批11） |
| F38 | 沃克斯传送与分身 | wortox_teleport_reviver_top, wortox_teleport_reviver_bottom, wortox_soul_spawn_fx, wortox_decoy_explode_fx, wortox_decoy_fizzle_fx, wortox_decoy_expire_fx, wortox_resist_fx | 7 | wortox_soul_spawn_fx | accepted（批11） |
| F39 | 虚布回旋镖与牛复活雷 | voidcloth_boomerang_launch_fx, voidcloth_boomerang_impact_fx, beefalo_reviving_lightning_fx | 3 | voidcloth_boomerang_launch_fx | accepted（批11） |
| F40 | 温蒂与阿比盖尔 | wendy_sanityaura_buff_on_fx, wendy_sanityaura_buff_off_fx, abigail_attack_shadow_fx, abigail_rising_twinkles_fx, abigail_gestalt_hit_fx, abigail_shadow_buff_fx, purebrilliance_mark_hit_fx | 7 | abigail_attack_shadow_fx | accepted（批11） |
| F41 | 坠落丝带 | fallingswish_clouds, fallingswish_clouds_fast, fallingswish_lines | 3 | fallingswish_clouds | accepted（批11） |
| F42 | 隐士蟹与传送门杂项 | hermitcrab_fx_small, hermitcrab_fx_med, hermitcrab_fx_tall, missile_explosion_fx, tree_rock_chop, tree_rock_fall, vault_portal_fx, atrium_portal_fx, boss_ripple_fx | 9 | hermitcrab_fx_small | accepted（批12） |
| F43 | 喷气孔与查莉圈 | fumarole_ember, fumarole_cook_fx, charlie_circle_spawn_fx, charlie_circle_spawn_ground_fx | 4 | fumarole_ember | accepted（批12） |
| F44 | 洞穴与地表杂项 | slurper_respawn, pandorachest_reset, cavehole_flick_warn, cavehole_flick, mole_move_fx, emote_fx, tears, spawn_fx_ocean_static | 8 | cavehole_flick | accepted（批12） |
| F45 | 猴群变形 | monkey_morphin_power_players_fx, monkey_de_morphin_fx, monkey_cursed_pre_fx, monkey_cursed_pst_fx, monkey_deform_pre_fx, monkey_deform_pst_fx | 6 | monkey_cursed_pre_fx | accepted（批12） |
| F46 | 沃尔夫冈与牛头怪血 | wolfgang_mighty_fx, minotaur_blood1, minotaur_blood2, minotaur_blood3 | 4 | minotaur_blood1 | accepted（批12） |
| 合计 | | | **341** | | |

## 外围候选（380 文件）核对进度

逐文件核验"确属特效注册名"，结论记此处：覆盖→并入族页；排除→附依据；阻塞→附原因。文件数不是注册名数；初查与已核验结果分开记录，初查不计入已完成文件。

已核验并闭环的文件（按批次；新页与既有宿主页分别注明）：

| 文件 | 注册名结论与依据 | 状态 |
|---|---|---|
| `prefabs/backpack.lua` | `backpack_swap_fx` 已由 `wiki/reference/prefabs/backpack.md` 的注册、皮肤与清理章节承载；`backpack` 是可装备容器主体而非独立 FX。工厂与注册见源码 L236–329。 | covered-existing：1 个 FX 注册，1 个主体不计 FX |
| `prefabs/heatrock.lua` | `heatrocklight` 已由 `wiki/reference/prefabs/heatrock.md` 的注册与构造章节承载，记录生成、启停和移除；`heatrock` 是暖石物品主体而非独立 FX。源码 L31–36、L78–92、L196–302。 | covered-existing：1 个 FX 注册，1 个主体不计 FX |
| `prefabs/sparks.lua` | `sparks`、`electrichitsparks`、`electrichitsparks_electricimmune` 已由 `wiki/reference/prefabs/sparks.md` 承载；本次回补 WX78 湿度更新的直接生成点。源码 L128–179、`prefabs/wx78.lua` L268–283。 | covered-existing：3 个 FX 注册；新增来源已 record |
| `prefabs/weaponsparks.lua` | `weaponsparks`、`weaponsparks_piercing`、`weaponsparks_thrusting`、`weaponsparks_bounce` 已由 `wiki/reference/prefabs/weaponsparks.md` 承载；Forge 事件数据包中的后置行为及生成点仍按页内缺口保留。源码 L163–166。 | covered-existing：4 个 FX 注册；事件包缺口保留 |
| `prefabs/abigail_flower.lua` | 五个召唤／收回／升级 FX 登记于 `wiki/reference/prefabs/abigailsummonfx.md`；`abigail_flower` 为物品主体。源码 L415–461。 | accepted（外围批 A）：5 个 FX 注册，1 个主体不计 FX |
| `prefabs/abysspillar.lua` | `abysspillar_fx` 登记于 `wiki/reference/prefabs/abysspillar_fx.md`，包含形成实体柱的玩法；`abysspillar` 是实体柱主体。源码 L804–844。 | accepted（外围批 A）：1 个混合 FX 注册，1 个主体不计 FX |
| `prefabs/acidraindrop.lua` | `acidraindrop`、`acidsmoke_endless` 登记于 `wiki/reference/prefabs/acidraindrop.md`；分别为池化落点烟和蘑菇树循环烟。源码 L34–100。 | accepted（外围批 A）：2 个 FX 注册 |
| `prefabs/diseaseflies.lua` | `diseaseflies` 登记于 `wiki/reference/prefabs/diseaseflies.md`；组件存在直接生成代码，快照内未发现原版字面挂载 `diseaseable`。源码 L1–77。 | accepted（外围批 B）：1 个 FX 注册 |
| `prefabs/explode_small.lua` | 十个爆裂、换装烟与火药注册登记于 `wiki/reference/prefabs/explode_small.md`；逐名见该页注册表。源码 L6–79、L155–164。 | accepted（外围批 B）：10 个 FX 注册 |
| `prefabs/grottopool_sfx.lua` | `grottopool_sfx` 登记于 `wiki/reference/prefabs/grottopool_sfx.md`，组件页保留调用关系；控制器未找到逐枚销毁回调的缺口入页。源码 L1–20。 | accepted（外围批 B）：1 个 FX 注册 |
| `prefabs/stafflight.lua` | `stafflight`、`staffcoldlight`、`staffcoldlightfx` 由既有宿主页 `stafflight.md`／`staffcoldlight.md` 的注册与构造节承载；`emberlight` 由新页 `emberlight.md` 覆盖，整文件 4 注册闭合。源码 L202–205。 | accepted（外围批 C）：4 个 FX 注册（3 既有＋1 新增） |
| `prefabs/lightning.lua` | `lightning`、`thunder` 由新页 `lightning.md` 覆盖，含工厂字段、端、真实结算与近距离雷声对照。源码 L59–129。 | accepted（外围批 C）：2 个 FX 注册 |
| `prefabs/staffcastfx.lua` | 八个施法／怀表回溯动画注册由新页 `staffcastfx.md` 逐名登记；`fx.lua` 同名 bank 辨析与客户端镜像入页。源码 L65–72。 | accepted（外围批 C）：8 个 FX 注册 |
| `prefabs/thunder_close.lua` | `thunder_close` 并入 `lightning.md` 的“远近雷声”节：网络壳＋世界屏闪＋随机 net_byte 本地声源，天气 L1116 唯一生成点。源码 L34–68。 | accepted（批 C 扩页）：1 个 FX 注册 |
| `prefabs/thunder_far.lua` | `thunder_far` 同上页；纯音效、2 秒移除、不推屏闪。源码 L15–41。 | accepted（批 C 扩页）：1 个 FX 注册 |
| `prefabs/moonstorm_lightning.lua` | `moonstorm_lightning` 并入 `lightning.md` 月暴变体节：动画／端、半径 4 月玻璃生成或充能、天气选名与月暴管理器两处生成点。源码 L75–116。 | accepted（批 C 扩页）：1 个混合 FX 注册 |

批 C 的三篇历史 claimed 已于 2026-11-04 逐页独立核验：保留原稿、修复 P2×3／P3×9 后验收，三页建页完成并 record（0 漂移）；`stafflight.lua` 整文件 4 注册按上表闭合。历史认领不再代表占用。

外围批 C 收口（2026-11-04）：6 文件、3 页、17 个 FX 注册（`stafflight.lua` 4／`lightning.lua` 2／`staffcastfx.lua` 8，另扩页并入 `thunder_close`、`thunder_far`、`moonstorm_lightning` 3 名）。独立复审 P1×0、P2×3（`lightning` 标题漏第五注册、`forest_network` 预载数写错、`emberlight` 用非官方名“极寒之星”）＋P3×9 已全部修复。8 文件页面／导航通过，strict 166 引用 0 错 0 警（跳过 2 个工具不支持链接），3 页 record 后 35 组 0 漂移；无 Lua 示例、未运行游戏。`fx-index` 达拆分阈值，已按主题拆出 `fx-world-index`（生物／Boss＋环境世界）并在父索引登记。

外围批 D（2026-11-04，机制族页第一组）：19 文件、4 页、64 个 FX 注册（reticule 家族 48／天气降水 7／海面波浪 7／施法光源 2）。文件与页面对应如下：

| 文件 | 注册名结论与依据 | 状态 |
|---|---|---|
| `prefabs/reticule.lua` | `reticule` 并入 `reticuleaoe.md`；默认瞄准圈注册与持续环入口。源码 L28。 | accepted（批 D）：1 个 FX 注册 |
| `prefabs/reticuleaoe.lua` | 31 个 AoE 圈／ping／target 注册并入同页；两套端模型、target FX 需 KillFX、声明≠生成与动态 ping 拼名入页。源码 L290–359。 | accepted（批 D）：31 个 FX 注册 |
| `prefabs/reticulearc.lua` | 2 名并入同页；弧形准星与 ping。源码 L30–66。 | accepted（批 D）：2 个 FX 注册 |
| `prefabs/reticulecharging.lua` | 1 名并入同页；`chargingreticule.pingprefab` 无人读取的陷阱入页。源码 L49。 | accepted（批 D）：1 个 FX 注册 |
| `prefabs/reticuleline.lua` | 3 名并入同页。源码 L89–91。 | accepted（批 D）：3 个 FX 注册 |
| `prefabs/reticulelong.lua` | 4 名并入同页。源码 L34–75。 | accepted（批 D）：4 个 FX 注册 |
| `prefabs/reticulemultitarget.lua` | 3 名并入同页；每帧子目标生成与 AddChild。源码 L158–160。 | accepted（批 D）：3 个 FX 注册 |
| `prefabs/golfclub_reticule.lua` | 3 名并入同页；高尔夫瞄准圈动态切换。源码 L53–104。 | accepted（批 D）：3 个 FX 注册 |
| `prefabs/rain.lua` | `rain` 并入 `rain.md`；天气驱动、雨棚门禁、池化 `raindrop`、初速率为 0。源码 L87–273。 | accepted（批 D）：1 个 FX 注册 |
| `prefabs/snow.lua` | `snow` 并入同页；冬宴换贴图与倍率。源码 L174。 | accepted（批 D）：1 个 FX 注册 |
| `prefabs/pollen.lua` | `pollen` 并入同页；水体／不可通行 tile 门禁与夏季降速。源码 L167。 | accepted（批 D）：1 个 FX 注册 |
| `prefabs/lunarhail.lua` | `lunarhail` 并入同页；海陆选名与喷溅池。源码 L263。 | accepted（批 D）：1 个 FX 注册 |
| `prefabs/lunarhaildrop.lua` | `lunarhaildrop` 并入同页；落点动画、回池与 delay 复用缺口。源码 L62。 | accepted（批 D）：1 个 FX 注册 |
| `prefabs/caverain.lua` | `caverain` 并入同页；洞穴天气驱动。源码 L274。 | accepted（批 D）：1 个 FX 注册 |
| `prefabs/caveacidrain.lua` | `caveacidrain` 并入同页；酸雨门禁与 `acidraindrop` 池；伤害在 `acidlevel`。源码 L274。 | accepted（批 D）：1 个 FX 注册 |
| `prefabs/wave.lua` | `wave_med`、`wave_splash` 并入 `wave_med.md`；碰撞、推船、加湿与 DoSplash。源码 L91–196。 | accepted（批 D）：2 个 FX 注册 |
| `prefabs/wave_shimmer.lua` | 4 名并入同页；客户端 wavemanager 生成，`wave_shimmer_flood` 无生成点。源码 L72–75。 | accepted（批 D）：4 个 FX 注册 |
| `prefabs/wave_shore.lua` | `wave_shore` 并入同页；SetAnim 选向与半格移位。源码 L70。 | accepted（批 D）：1 个 FX 注册 |
| `prefabs/staff_castinglight.lua` | 2 名由新页 `staff_castinglight.md` 覆盖；五次幂包络、六处状态调用与 2 帧延迟移除。源码 L78–79。 | accepted（批 D）：2 个 FX 注册 |

批次 D 验收（2026-11-04）：独立复审 P1×1（联网 target 被误写成“自然淡出”，实际只有服务端 `KillFX` 才反向淡出）＋P2×6（`LAYER_WORLD_BACKGROUND` 与 `LAYER_BACKGROUND` 混写、4 处生成点漏项、`SetChargeScale` 归属、把“scripts 内未见”断定为缺口、sources 缺 14 个消费文件）＋P3×12 已全部修复；`pollen` 的“水不生成”改为只排除 `IsImpassableTile`（海洋不在此组）。4 页页面与导航通过，strict 266 引用 0 错 0 警（reticule 112／rain 68／wave 67／staff 18），record 后 90 组 0 漂移；reticule 与 wave 各 1 段 Lua 语法通过；未运行游戏。`reticuleaoe` 按 complex 记录（178 行／约 31.9 KiB），`rain` 与 `wave_med` 压入 standard（分别约 16.0 KiB）。

外围批 E（2026-11-04，机制族页第二组）：30 文件、3 页、57 个 FX 注册（火焰烟雾 15／光源实体 13／威胁预警 29）。文件与页面对应如下（页面已生成并登记 fx-world-index，状态先记 generated，待独立复审后改 accepted）：

| 文件 | 注册名与结论 | 状态 |
|---|---|---|
| `prefabs/fire.lua` | `fire` 并入 `fire.md`；6 档 firefx 等级表＋heater 热度 30–240；非死代码：12 处 `AddBurnFX("fire")` 与 helper→`SpawnFX` 间接生成。 | accepted（批 E）：1 个混合 FX 注册 |
| `prefabs/campfirefire.lua` | `campfirefire`、`portable_campfirefire` 并入同页；4 档／3 档，quagmire 分支依赖缺失事件包。 | accepted（批 E）：2 个 FX 注册 |
| `prefabs/coldfirefire.lua` | `coldfirefire` 并入同页；吸热 heater，热度 −10…−40。 | accepted（批 E）：1 个 FX 注册 |
| `prefabs/nightlight_flame.lua` | `nightlight_flame` 并入同页；4 档、无 heater、压暗。 | accepted（批 E）：1 个 FX 注册 |
| `prefabs/pigtorch_flame.lua` | `pigtorch_flame` 并入同页；3 档、无 heater。 | accepted（批 E）：1 个 FX 注册 |
| `prefabs/nightstickfire.lua` | `nightstickfire` 并入同页；纯 Light FX，宿主装备生成／卸下删除。 | accepted（批 E）：1 个 FX 注册 |
| `prefabs/character_fire.lua` | `character_fire`、`character_fire_flicker` 并入同页；flicker 仅第 3 档有效，仅巨鹿替换 fxdata。 | accepted（批 E）：2 个 FX 注册 |
| `prefabs/firefx_light.lua` | `firefx_light` 并入同页；`firefx` 构造时生成并父挂的纯灯光子实体。 | accepted（批 E）：1 个 FX 注册 |
| `prefabs/floatinglanternfire.lua` | `floatinglanternfire` 并入同页；4 档＋非专服淡出＋灯位偏移。 | accepted（批 E）：1 个 FX 注册 |
| `prefabs/thurible_smoke.lua` | `thurible_smoke` 并入同页；独立 VFX 烟（非 firefx），专服跳过粒子。 | accepted（批 E）：1 个 FX 注册 |
| `prefabs/smoke_plant.lua` | `smoke_plant` 并入同页；闷烧指示，`StartWildfire` 生成、`StopSmoldering` 清理。 | accepted（批 E）：1 个 FX 注册 |
| `prefabs/firehits.lua` | `firehit` 并入同页；本地非联网命中体；`master_postinit` 在缺失事件包，快照无生成点。 | accepted（批 E）：1 个 FX 注册（事件包缺口） |
| `prefabs/eyeflame.lua` | `eyeflame` 并入同页；完全非联网、成对生成／删除。 | accepted（批 E）：1 个 FX 注册 |
| `prefabs/cavelight.lua` | `cavelight`、`_small`、`_tiny`、`_atrium` 并入 `cavelight_small.md`；四档 widthscale＋hideout（5±4 秒），fullmoon 档在洞穴不可达。 | accepted（批 E）：4 个 FX 注册 |
| `prefabs/cavelightmoon.lua` | `cavelightmoon`、`_small`、`_tiny` 并入同页；normal 兼鼹鼠蝠 spawner；读档满月初值缺口。 | accepted（批 E）：3 个 FX 注册 |
| `prefabs/chesterlight.lua` | `chesterlight` 并入同页；SGchester transition 唯一生成点，客户端不跑淡出。 | accepted（批 E）：1 个 FX 注册 |
| `prefabs/booklight.lua` | `booklight` 并入同页；`SetDuration`／淡出仅服务端，读档续时。 | accepted（批 E）：1 个 FX 注册 |
| `prefabs/spawnlight_multiplayer.lua` | `spawnlight_multiplayer` 并入同页；出生光＋保温，playerspawner 生成。 | accepted（批 E）：1 个 FX 注册 |
| `prefabs/atrium_light.lua` | `atrium_light`、`_back`、`_light` 并入同页；可供电设施＋两个子实体（`_light` 无 Light）。 | accepted（批 E）：3 个 FX 注册 |
| `prefabs/houndwarning.lua` | `houndwarning_lvl1`～`lvl4` 并入 `houndwarning_lvl1.md`；4 档距离、非联网、父挂焦点随机角。 | accepted（批 E）：4 个 FX 注册 |
| `prefabs/wormwarning.lua` | `wormwarning_lvl1`～`lvl4`＋`wormwarning_worm_boss` 并入同页。 | accepted（批 E）：5 个 FX 注册 |
| `prefabs/monsterwarningsounds.lua` | 12 个动态 `*warning_lvlN` 并入同页；联网代理＋客户端本地声源。 | accepted（批 E）：12 个 FX 注册 |
| `prefabs/piratewarningsound.lua` | `piratewarningsound` 并入同页；1 秒代理、15 拉回。 | accepted（批 E）：1 个 FX 注册 |
| `prefabs/yoth_knightwarningsound.lua` | `yothknightwarningsound` 并入同页；RANGE 注释与代码不符入页。 | accepted（批 E）：1 个 FX 注册 |
| `prefabs/meteorwarning.lua` | `meteorwarning` 并入同页；联网动画、由 shadowmeteor 爆炸删除。 | accepted（批 E）：1 个 FX 注册 |
| `prefabs/warningshadow.lua` | `warningshadow` 并入同页；联网动画，宿主缩放移除；lunarhailmanager 调用已弃用。 | accepted（批 E）：1 个 FX 注册 |
| `prefabs/shadowskittish.lua` | `shadowskittish` 并入同页；非联网、playerprox、5+10r 秒消失。 | accepted（批 E）：1 个 FX 注册 |
| `prefabs/shadowwatcher.lua` | `shadowwatcher` 并入同页；LightWatcher 进光即移除。 | accepted（批 E）：1 个 FX 注册 |
| `prefabs/creepyeyes.lua` | `creepyeyes` 并入同页；随机动画与 blink 周期。 | accepted（批 E）：1 个 FX 注册 |
| `prefabs/sounddebugicon.lua` | `sounddebugicon` 并入同页；调试用途，唯一生成点为 `debugsounds` 钩子。 | accepted（批 E）：1 个 FX 注册 |

批次 E 验收（2026-11-04）：独立复审 P1×1（`cavelight_small` 把 `STRINGS.NAMES.ATRIUM_LIGHT`＝「远古灯柱」误判为无官方名）＋P2×9（`fire` 生成点漏宿主与 sources 缺 12 文件；`cavelight` 漏 `areas.lights` 生成链；`houndwarning` 理智阈值顺序、暗影暗度/夜晚门控、洞穴挂载与 po sources）＋P3×12 已修。3 页页面／导航通过，strict 235 引用 0 错 0 警（fire 62／cavelight 64／houndwarning 106），record 后 104 组 0 漂移；未运行游戏。`fire` 18.6 KiB／`cavelight_small` 17.4 KiB 按 complex 记录（多工厂、多宿主与两套状态机），`houndwarning_lvl1` 30.9 KiB complex。

### 外围逐文件结论（只读分诊批 2，2026-11-04，30 文件）

方法同批 1；结论：covered-existing 3（atrium_light 家族已由批 E 光源页承载）、excluded 57、needs-page 39（26 文件）、blocked 1（bishop_charge_hit，唯一生成者 bishop_charge 已废弃且无生成点）。

| 文件 | 注册名与结论 | 状态 |
|---|---|---|
| `prefabs/alterguardian_hat_equipped.lua` | `alterguardian_hat_equipped`：仅 Transform/AnimState/Follower/Bloom＋FX/DECOR，无组件；`hats.lua` 前后层各生成一份。 | needs-page（1 个 FX） |
| `prefabs/alterguardian_laser.lua` | `alterguardian_laser`、`laserempty` 为玩法；`laserscorch`（地面焦痕 net_byte 淡出）、`lasertrail`（烟尾）、`laserhit`（非联网 CLASSIFIED 闪光）为 FX。 | excluded（2 玩法）＋needs-page（3 个 FX） |
| `prefabs/alterguardian_lunar_fissures.lua` | `alterguardian_lunar_fissures` 给踩踏实体加 `lunarfissureburning` 属玩法；`alterguardian_lunar_fissure_burn_fx`、`alterguardian_lunar_supernova_burn_fx` 为纯 Anim FX。 | excluded＋needs-page（2 个 FX） |
| `prefabs/alterguardian_phase1.lua` | `alterguardian_phase1`、`_lunarrift`、`_lunarrift_gestalt` 为 Boss（locomotor/health/combat/lootdropper/planarentity）；内部 PLANAR/跟随 FX 未注册。 | excluded（3 玩法） |
| `prefabs/alterguardian_phase2spike.lua` | `alterguardian_phase2spiketrail` 带 combat＋物理＋emerge 伤害；`alterguardian_phase2spike` 有 workable(MINE)/obstacle；内部 `alterguardian_spiketrail_fx` 非注册。 | excluded（混合＋玩法） |
| `prefabs/alterguardian_phase3trap.lua` | `alterguardian_phase3trapprojectile` 落地 AOE 伤害/破坏；`alterguardian_phase3trap` 有 workable/timer/lootdropper；`alterguardian_phase3trapgroundfx` 纯 CLASSIFIED/DECOR 地面动画。 | excluded＋needs-page（1 个 FX） |
| `prefabs/alterguardian_phase4_lunarrift.lua` | Boss 满配组件；`_slam_fx`（bomb_lunarplant bank＋sleepcloud 符号＋地面环）与 `_erupt_fx`（wagboss_lunar_blast）为 FX。 | excluded＋needs-page（2 个 FX） |
| `prefabs/alterguardianhatshard.lua` | `alterguardianhatshard` 为 container/inventoryitem/preserver；`alterguardianhatshard_symbol_fx` 仅 AnimState/Follower，由电池跟随生成。 | excluded＋needs-page（1 个 FX） |
| `prefabs/amulet.lua` | 六护符均 equippable/inventoryitem＋fueled/finiteuses/heater；`yellowamuletlight` 仅 Light＋FX。 | excluded＋needs-page（1 个 FX） |
| `prefabs/archive_props.lua` | 6 玩法＋3 技术实体（waypoint 埋点、rubble1/2 worldgen 占位）；`archive_security_pulse_sfx`（纯声＋绕圈 updatelooper）、`archive_switch_pad`／`archive_switch_base`（地面双层贴图）、`archive_ambient_sfx`（事件驱动循环声）为 FX。 | excluded＋needs-page（4 个 FX） |
| `prefabs/archive_resonator.lua` | `archive_resonator`（finiteuses/workable）、`_item`（deployable）、`_item_placer`（预览）为玩法；`archive_resonator_base` 仅 Light＋FX＋到期 timer。 | excluded＋needs-page（1 个 FX） |
| `prefabs/armor_lunarplant.lua` | 两护甲为玩法；`armor_lunarplant_glow_fx`、`armor_lunarplant_husk_glow_fx` 由 MakeGlow 生成，仅 colouraddersync＋highlightchild。 | excluded＋needs-page（2 个 FX） |
| `prefabs/armor_voidcloth.lua` | `armor_voidcloth` 为玩法；`armor_voidcloth_fx` 有 colouraddersync/autojiggle＋9 帧跟随子实体，无玩法。 | excluded＋needs-page（1 个 FX） |
| `prefabs/armor_wagpunk.lua` | `armorwagpunk` 为玩法；`armorwagpunk_fx` 仅 net_tinybyte level＋colouraddersync；`steamon_fxfn` 定义未注册（死代码）。 | excluded＋needs-page（1 个 FX） |
| `prefabs/atrium_gate_activatedfx.lua` | `atrium_gate_activatedfx`：Light＋AnimState＋SetFX/KillFX 状态切换，`atrium_gate.lua:162` 生成。 | needs-page（1 个 FX） |
| `prefabs/atrium_gate_pulsesfx.lua` | `atrium_gate_pulsesfx`、`atrium_gate_explodesfx`、`grotto_war_sfx` 同一 makesfx 工厂，只 CreateEntity＋SoundEmitter＋1 秒自毁。 | needs-page（3 个 FX，合 1 页） |
| `prefabs/atrium_light.lua` | `atrium_light` 为可供电设施（混合）；`_back`、`_light` 为贴图子实体；三者已由 `cavelight_small.md` 承载。 | covered-existing（3 个注册，批 E 光源页） |
| `prefabs/balloon_held_child.lua` | `balloon_held_child`（联网代理，仅 netvar/DECOR）与 `balloon_held_child_client`（SGballoonheld＋FollowSymbol，非联网）。 | needs-page（2 个 FX，合 1 页） |
| `prefabs/balloonparty.lua` | `balloonparty`、`balloonparty_buff` 为玩法/debuff；`balloonparty_confetti_cloud` 每 1 秒给附近玩家加 debuff（混合），`balloonparty_confetti_balloon` 仅物理＋动画（FX）。 | excluded＋needs-page（1 个纯 FX；混合另记） |
| `prefabs/bat_boss.lua` | `bat_boss`、`bat_boss_shadow` 为玩法；`bat_bosshat_fx`（DECOR＋colouraddersync＋chomp）与 `bat_boss_shadow_split_fx`（一次性 split＋唤醒声）为 FX。 | excluded＋needs-page（2 个 FX） |
| `prefabs/batbat.lua` | `batbat` 为玩法；`batbat_bats`、`batbat_fantasy_fx` 为 net_tinybyte variation 代理＋本地播放，皮肤映射在 `skinprefabs.lua`（快照内，非缺口）。 | excluded＋needs-page（2 个 FX） |
| `prefabs/battreefx.lua` | `battreefx`：SetClassifiedTarget 只让目标玩家看见，本地扇翅＋`batspooked` 覆盖层事件；`components/spooked.lua` 生成。 | needs-page（1 个 FX） |
| `prefabs/beargerfur_sack.lua` | 袋为 container/preserver；`beargerfur_sack_frost_fx` 仅贴图＋pre/loop/pst，开袋生成。 | excluded＋needs-page（1 个 FX） |
| `prefabs/bernie_big.lua` | 伯尼为 health/combat/locomotor/SG+brain；`bernie_big_fire` Light＋AnimState＋EndBernieFire 收尾。 | excluded＋needs-page（1 个 FX） |
| `prefabs/bishop.lua` | 3 玩法＋worldgen 生成器；`bishop_targeting_fx` Light＋dist 同步＋reticule 动画。 | excluded＋needs-page（1 个 FX） |
| `prefabs/bishop_charge.lua` | `bishop_charge`（已废弃 projectile，无生成点）、`bishop_charge_hit`（只由废弃 charge 生成）；`bishop_charge2_fx` 虽 tag FX，但 updatelooper 做 electric 伤害/electrocute（混合）。 | excluded（2 玩法/混合）＋blocked（charge_hit 无生成点） |
| `prefabs/boatrace_checkpoint.lua` | 主件／deploykit／placer 为玩法；`boatrace_checkpoint_flag` 仅 highlightchild＋贴图。 | excluded＋needs-page（1 个 FX） |
| `prefabs/boatrace_checkpoint_indicator.lua` | `boatrace_checkpoint_indicator` tag FX 但挂 proximitybeacon＋timer，参与赛道信标/名次/音乐事件。 | excluded（混合玩法） |
| `prefabs/boatrace_start.lua` | 主件控制器／kit／placer／bobber 为玩法或混合；`boatrace_start_flag`、`boatrace_fireworks` 仅贴图＋animover。 | excluded＋needs-page（2 个 FX） |
| `prefabs/bomb_lunarplant.lua` | 炸弹为 complexprojectile/weapon/explosive；`bomb_lunarplant_explode_fx` 仅贴图＋bloom＋sleepcloud 符号覆盖＋音效。 | excluded＋needs-page（1 个 FX） |

外围批 F（2026-11-04，宿主附属与远古设施组，生成中）：10 文件、3 页、27 个 FX 注册（宿主附属层 12／吊灯与石像鬼 8／熔炉化石碎裂 7）。分诊批 1／2 的 needs-page 项开始落地：

| 文件 | 注册名与结论 | 状态 |
|---|---|---|
| `prefabs/fence.lua` | `fence_gate_anim`、`quagmire_park_gate_anim` 并入 `fence_gate_anim.md`；`MakeWallAnim` 纯外观，宿主 SetParent＋highlightforward，门美术全在子实体。 | accepted（批 F）：2 个 FX 注册 |
| `prefabs/furniture_chairs.lua` | 4 个 `_back` 椅背层并入同页；`backfn` 无组件、SetParent 于椅并进 highlightchildren。 | accepted（批 F）：4 个 FX 注册 |
| `prefabs/hermitcrab_teashop.lua` | `hermitcrab_teashop_front` 并入同页；L449-482 纯 AnimState＋FX，宿主转发动画与燃烧态。 | accepted（批 F）：1 个 FX 注册 |
| `prefabs/junk_pile_big.lua` | `junk_pile_side`、`junk_pile_blueprint` 并入同页；侧片 4 向、蓝图 Follower＋animover 自删。 | accepted（批 F）：2 个 FX 注册 |
| `prefabs/lantern_posts.lua` | `yots_lantern_light_chain`、`hermitcrab_lightpost_light_chain` 并入同页；独立联网存档、不 SetParent，读档重挂。 | accepted（批 F）：2 个 FX 注册 |
| `prefabs/waterplant.lua` | `waterplant_base` 并入同页；服务端挂 freezable，染色与 bud 图层由宿主同步。 | accepted（批 F）：1 个 FX 注册 |
| `prefabs/archive_chandelier.lua` | `archive_chandelier`、`vault_chandelier`、`vault_crawler_chandelier` 并入 `archive_chandelier.md`；三相灯、`vault_crawler_chandelier` 兼生成 vault_crawler；`OnSpawnTask` 为死代码入页。 | accepted（批 F）：3 个 FX 注册 |
| `prefabs/gargoyles.lua` | 5 个 `gargoyle_*_fx` 并入同页；网络空壳＋非专服本地动画，仅 `OnPetrified` 生成。 | accepted（批 F）：5 个 FX 注册 |
| `prefabs/lavaarena_beetletaur.lua` | 4 个 `beetletaur_fossilized_break_fx_*` 并入 `lavaarena_beetletaur_fossilized_break_fx_right.md`；四向、side 隐藏与 `_alt` 中断变体。 | accepted（批 F）：4 个 FX 注册 |
| `prefabs/lavaarena_rhinodrill.lua` | 3 个 `rhinodrill_fossilized_break_fx*` 并入同页；六向、无覆盖/无 alt。生成点在缺失的 `components/fossilizable.lua` 与事件包，页面按缺口标注。 | accepted（批 F）：3 个 FX 注册 |

批次 F 验收（2026-11-04）：独立复审 P1×1（`archive_chandelier` 把 `archivemanager` 缺失时的 `nil` 短路写成“按断电”，源码实为按通电，与页内示例矛盾）＋P2×3（废料堆生成点把 `TRANSLATE_TO_PREFABS` 声明表当世界生成点、chandelier “掉落镜头”归属 SGvault_crawler、`CreateBreakFX` build 名写错）＋P3 若干已修。3 页页面／导航通过，strict 238 引用 0 错 0 警（fence 112／archive 76／fossilized 50），record 后 40 组 0 漂移；archive 与 fossilized 各 1 段 Lua 语法通过；未运行游戏。

外围批 G（2026-11-04，Boss 与装备表现组，生成中）：13 文件、3 页、22 个 FX 注册（装备跟随光效 7／天体英雄激光与灼烧 8／藏影女王战斗 7）。页面已生成并登记（装备页入 fx-index，其余入 fx-world-index），待独立复审与 record：

| 文件 | 注册名与结论 | 状态 |
|---|---|---|
| `prefabs/armor_lunarplant.lua` | `armor_lunarplant_glow_fx`、`armor_lunarplant_husk_glow_fx` 并入 `armor_lunarplant_glow_fx.md`；MakeGlow 控制器＋6 本地子帧，父挂 owner。 | accepted（批 G）：2 个 FX 注册 |
| `prefabs/armor_voidcloth.lua` | `armor_voidcloth_fx` 并入同页；9 子帧、autojiggle、皮肤 netvar。 | accepted（批 G）：1 个 FX 注册 |
| `prefabs/armor_wagpunk.lua` | `armorwagpunk_fx` 并入同页；level netvar、2–5 级 bank 同 05、`steamon_fxfn` 死代码。 | accepted（批 G）：1 个 FX 注册 |
| `prefabs/amulet.lua` | `yellowamuletlight` 并入同页；独立 Light、只 SetParent(owner)。 | accepted（批 G）：1 个 FX 注册 |
| `prefabs/alterguardian_hat_equipped.lua` | `alterguardian_hat_equipped` 并入同页；前/后两层、hair 跟随、animover 收尾。 | accepted（批 G）：1 个 FX 注册 |
| `prefabs/alterguardianhatshard.lua` | `alterguardianhatshard_symbol_fx` 并入同页；电池槽 gemN 符号跟随。 | accepted（批 G）：1 个 FX 注册 |
| `prefabs/alterguardian_laser.lua` | `alterguardian_laserscorch`／`lasertrail`／`laserhit` 并入 `alterguardian_laserscorch.md`；地面焦痕／烟尾／命中加色。 | accepted（批 G）：3 个 FX 注册 |
| `prefabs/alterguardian_lunar_fissures.lua` | 两个 burn_fx 并入同页；裂隙与超新星灼烧叠层。 | accepted（批 G）：2 个 FX 注册 |
| `prefabs/alterguardian_phase3trap.lua` | `alterguardian_phase3trapgroundfx` 并入同页；CLASSIFIED/DECOR 地面动画。 | accepted（批 G）：1 个 FX 注册 |
| `prefabs/alterguardian_phase4_lunarrift.lua` | `_slam_fx`、`_erupt_fx` 并入同页；砸地环与喷发。 | accepted（批 G）：2 个 FX 注册 |
| `prefabs/charlie_boss.lua` | `charlie_boss_aoe_flame_fx` 并入 `charlie_boss_aoe_flame_fx.md`；召唤落焰。 | accepted（批 G）：1 个 FX 注册 |
| `prefabs/charlie_boss_minions.lua` | `charlie_boss_minion_hit_fx`（FX）＋`minion1/2`（混合载体，weapon/projectile/AOE 结算）并入同页。 | accepted（批 G）：3 个注册（2 混合） |
| `prefabs/charlie_boss_projectiles.lua` | `charlie_boss_projectile`（混合）＋反射／命中两 FX 并入同页；内部 looper/rotator 非注册。 | accepted（批 G）：3 个注册（1 混合） |

批次 G 验收（2026-11-04）：独立复审 P1×0、P2×4（护甲页客户端子帧创建点、天体英雄空激光无视觉与 11+2 道、藏影女王 Despawn 路径与非专服主机建 rotator）＋P3 若干已修；3 页页面／导航通过、strict 276 引用 0 错 0 警（armor 108／alterguardian 97／charlie 71），record 后 31 组 0 漂移；未运行游戏。

外围批 H（2026-11-04，冲击／激光／贴地表现组，生成中）：11 文件、3 页、15 个 FX 注册（冲击与爆点 6／巨鹿激光 5／贴地贴水 4）。页面已生成并登记 fx-world-index，待独立复审与 record：

| 文件 | 注册名与结论 | 状态 |
|---|---|---|
| `prefabs/charliearena_spike.lua` | `charliearena_spike_explode_fx` 并入 `groundpoundring_fx.md`；纯 Anim/Network＋FX、animover 自删。 | accepted（批 H）：1 个 FX 注册 |
| `prefabs/fireringfx.lua` | `firering_fx` 并入同页；本地代理（OnGround/背景层/bloom），主壳 3 秒自删。 | accepted（批 H）：1 个 FX 注册 |
| `prefabs/groundpoundringfx.lua` | `groundpoundring_fx` 并入同页；本地代理＋net_bool fastforward。 | accepted（批 H）：1 个 FX 注册 |
| `prefabs/ground_chunks_breaking.lua` | `ground_chunks_breaking` 并入同页；本地代理＋1 秒自删，7 处生成。 | accepted（批 H）：1 个 FX 注册 |
| `prefabs/fused_shadeling_bomb.lua` | `fused_shadeling_bomb_death_fx` 并入同页；纯 Anim/Sound/Network、animover 自删。 | accepted（批 H）：1 个 FX 注册 |
| `prefabs/eye_charge.lua` | `eye_charge_hit` 并入同页；纯 FX、0.5 秒自删。`eye_charge` 本体为 projectile 玩法，仅辨析。 | accepted（批 H）：1 个 FX 注册 |
| `prefabs/deerclops_laser.lua` | 5 注册并入 `deerclops_laserscorch.md`：3 纯 FX＋2 混合载体（combat／Trigger，伤害归属入页）。 | accepted（批 H）：5 个注册（2 混合） |
| `prefabs/ice_puddle.lua` | `ice_puddle` 并入 `ice_puddle.md`；背景层、客户端 rock_ice 父挂、不自删。 | accepted（批 H）：1 个 FX 注册 |
| `prefabs/ice_splash.lua` | `ice_splash` 并入同页；animover 自删，rock_ice／sharkboi 生成。 | accepted（批 H）：1 个 FX 注册 |
| `prefabs/nitre_formation.lua` | `nitre_formation` 并入同页；客户端登记 pond.highlightchildren，pond 按世界数据父挂；官方名「硝石堆」入页。 | accepted（批 H）：1 个 FX 注册 |
| `prefabs/oceanfishingbobber.lua` | `oceanfishinghook_ripple` 并入同页；CLASSIFIED＋FX、非联网、挂浮标下；BOBBERS 表驱动 30 名不属本页。 | accepted（批 H）：1 个 FX 注册 |

批次 H 验收（2026-11-04）：独立复审 P1×0、P2×3（`groundpounder` 的 hitrings 只推外观而非命中半径、`willow_ember` 整体被误称死代码、酸雨停雨不会立即移除晶簇）＋P3 若干已修；3 页页面／导航通过、strict 192 引用 0 错 0 警（groundpound 80／deerclops 61／ice 51），record 后 38 组 0 漂移；deerclops 页保持 standard（16,363 B，余 21 B）；未运行游戏。

外围批 I（2026-11-04，残骸植物／实验台粒子／月装置组，生成中）：12 文件、3 页、16 个 FX 注册（残骸与植物 6／实验台与粒子 6／月装置 4）。页面已生成并登记 fx-world-index，待独立复审与 record：

| 文件 | 注册名与结论 | 状态 |
|---|---|---|
| `prefabs/lunar_grazer.lua` | `lunar_grazer_debris` 并入 `mushtree_tall_webbed_burntfx.md`；联网物理装饰、5 变体、随宿主清理。 | accepted（批 I）：1 个 FX 注册 |
| `prefabs/lunarthrall_plant.lua` | `lunarthrall_plant_vine` 并入同页；普通形态 fx，weak 形态可受伤；头端 removetail 清理。 | accepted（批 I）：1 个混合 FX 注册 |
| `prefabs/mushtree_webbed.lua` | `mushtree_tall_webbed_burntfx` 并入同页；双清理（animover＋时长兜底）。 | accepted（批 I）：1 个 FX 注册 |
| `prefabs/marsh_bush.lua` | `burnt_marsh_bush_erode` 并入同页；联网空壳＋本地 ashes 代理、专服不建。 | accepted（批 I）：1 个 FX 注册 |
| `prefabs/nightmaregrowth.lua` | `nightmaregrowth_crack` 并入同页；背景层装饰、读档恢复旋转、宿主移除清理。 | accepted（批 I）：1 个 FX 注册 |
| `prefabs/minotaur.lua` | `minotaur_blood_big` 并入同页；21 帧生成触手，伤害归 bigshadowtentacle。 | accepted（批 I）：1 个混合 FX 注册 |
| `prefabs/madscience_lab.lua` | `_goop`／`_fire`／`_glow` 并入 `madscience_lab_goop.md`；三层共用 bank 靠 Hide 分层。 | accepted（批 I）：3 个 FX 注册 |
| `prefabs/mistparticle.lua` | `mist` 并入同页；gamelogic 按 Mist 节点 poly 建 area emitter，非联网。 | accepted（批 I）：1 个 FX 注册 |
| `prefabs/nonslipgrit.lua` | `nonslipgrit_buff_fx` 并入同页；双 VFX 发射器按父级位移调发射量。 | accepted（批 I）：1 个 FX 注册 |
| `prefabs/kitcoon.lua` | `kitcoon_hide_fx` 并入同页；1 秒/animover 自删，OnHide_HideAndSeek 生成。 | accepted（批 I）：1 个 FX 注册 |
| `prefabs/moon_device.lua` | `moon_device_pillar`／`_top`（SetPrefabNameOverride）／`_break_stage1` 并入 `moon_device_pillar.md`。 | accepted（批 I）：3 个 FX 注册 |
| `prefabs/moon_fissure.lua` | `moon_fissure_fx` 并入同页；联网 DECOR，宿主 Show/Hide 与 transition 驱动。 | accepted（批 I）：1 个 FX 注册 |

外围批 L（2026-11-04，地面贴片／宿主挂接／灯具光源组，生成中）：22 文件、3 页、29 个 FX 注册（地面贴片 6／宿主挂接 7／灯具光源 16）。页面已生成并登记 fx-world-index，待独立复审与 record：

| 文件 | 注册名与结论 | 状态 |
|---|---|---|
| `prefabs/crabking_claw.lua` | `crabking_claw_shadow`、`crabking_claw_swipe_fx` 并入 `crabking_claw_shadow.md`；水下阴影与挥击弧。 | accepted（批 L）：2 个 FX 注册 |
| `prefabs/deerclopseyeball_sentryward.lua` | `deerclopseyeball_sentryward_fx` 并入同页；冰圈真实冻结/降温/催眠。 | accepted（批 L）：1 个混合 FX 注册 |
| `prefabs/fused_shadeling_bomb.lua` | `fused_shadeling_bomb_scorch` 并入同页；net_byte 淡出、缩 0.7。 | accepted（批 L）：1 个 FX 注册 |
| `prefabs/daywalker_pillar.lua` | `daywalker_pillar_base_fx`、`daywalker_pillar_hole` 并入同页；贴地装饰。 | accepted（批 L）：2 个 FX 注册 |
| `prefabs/king_cane.lua` | `king_cane_fx` 并入 `king_cane_fx.md`；宝石/腐蚀层。 | accepted（批 L）：1 个 FX 注册 |
| `prefabs/hermithouse_ornament.lua` | `hermithouse_ornament_fx` 并入同页；CloneAsFx 三宿主复用。 | accepted（批 L）：1 个 FX 注册 |
| `prefabs/lunar_grazer.lua` | `lunar_grazer_core_fx` 并入同页；Follower 跟 rock_cycle_follow。 | accepted（批 L）：1 个 FX 注册 |
| `prefabs/lunarthrall_plant.lua` | `lunarthrall_plant_back` 并入同页；`OnRemovedEntity` 拼写陷阱入页。 | accepted（批 L）：1 个 FX 注册 |
| `prefabs/moon_altar_link.lua` | `moon_altar_link_contained` 并入同页；moon_geyser build、阶段设置。 | accepted（批 L）：1 个 FX 注册 |
| `prefabs/monkeyisland_portal.lua` | `monkeyisland_portal_lootfollowfx` 并入同页；AddChild 挂 loot、自删。 | accepted（批 L）：1 个 FX 注册 |
| `prefabs/wormlight.lua` | 8 个 wormlight 注册并入 `wormlight.md`；spell 代理与三档光。 | accepted（批 L）：8 个 FX 注册 |
| `prefabs/mininglantern.lua` | `lanternlight` 并入同页；半径由提灯 fuelupdate 写。 | accepted（批 L）：1 个 FX 注册 |
| `prefabs/miniboatlantern.lua` | `miniboatlanternlight` 并入同页；双端闪烁。 | accepted（批 L）：1 个 FX 注册 |
| `prefabs/redlantern.lua` | `redlanternlight`、`redlanternbody` 并入同页；灯/体分离。 | accepted（批 L）：2 个 FX 注册 |
| `prefabs/queen_torch.lua` | `queen_torch_fx`、`queen_torchlight` 并入同页；技能亮度改半径。 | accepted（批 L）：2 个 FX 注册 |
| `prefabs/yotc_carrat_race_finish.lua` | `yotc_carrat_race_finish_light` 并入同页；赛后 3.5 秒关灯。 | accepted（批 L）：1 个 FX 注册 |
| `prefabs/yots_worm_lantern.lua` | `yots_worm_lantern_light` 并入同页；变身回收为缺口。 | accepted（批 L）：1 个 FX 注册 |
| `prefabs/slurper.lua` | `slurperlight` 并入同页；低/高档插值。 | accepted（批 L）：1 个 FX 注册 |
| `prefabs/squid.lua` | `squideyelight` 并入同页；FollowSymbol＋dimLight。 | accepted（批 L）：1 个 FX 注册 |
| `prefabs/hutch.lua` | `hutch_music_light_fx` 由 `wormlight.md` 承载；`king_cane_fx.md` 仅交叉引用。 | accepted（批 L）：1 个 FX 注册 |

批次 L 验收（2026-11-04）：独立复审 P1×0、P2×5（`king_cane_fx` 仍把交叉引用的哈奇光算作本页第七注册；背层帧同步因调用顺序为死代码；`soundpath` 缺失时 `idle_LP` 不响而非走默认音；背层 `OnRemovedEntity` 反登记不执行；`wormlight` 把代理结束后仍有光写反）＋P3 若干已修。3 页页面／导航通过，strict 321 引用 0 错 0 警（crabking 67／king_cane 104／wormlight 150），record 后 46 组 0 漂移；未运行游戏。

外围批 M（2026-11-04，附着跟随／小型一次性物件／有表现载体组，生成中）：15 文件、3 页、18 个 FX 注册（附着跟随 6／小型物件 8／载体 4）。页面已生成并登记 fx-world-index，待独立复审与 record：

| 文件 | 注册名与结论 | 状态 |
|---|---|---|
| `prefabs/gestalt.lua` | `gestalt_trail` 并入 `gestalt_trail.md`；OnGround/bloom、40 帧自删。 | accepted（批 M）：1 个 FX 注册 |
| `prefabs/daywalker2.lua` | `daywalker2_buried_fx` 并入同页；Follower/Network、八面朝。 | accepted（批 M）：1 个 FX 注册 |
| `prefabs/gestalt_cage.lua` | `gestalt_cage_swap_fx` 并入同页；AttachToOwner、八面朝。 | accepted（批 M）：1 个 FX 注册 |
| `prefabs/gelblob.lua` | `gelblob_back_fx` 并入同页；FinalOffset −7、隐藏 FRONT。 | accepted（批 M）：1 个 FX 注册 |
| `prefabs/chester.lua` | `shadow_chester_swirl_fx` 并入同页；CLASSIFIED、pre/loop/pst。 | accepted（批 M）：1 个 FX 注册 |
| `prefabs/firesuppressor.lua` | `firesuppressor_glow` 并入同页；net_bool 淡入。 | accepted（批 M）：1 个 FX 注册 |
| `prefabs/charlierose.lua` | `charlierose` 并入 `grasspartfx.md`；FX＋可进背包的燃料混合。 | accepted（批 M）：1 个混合 FX 注册 |
| `prefabs/charlieresidue.lua` | `charlieresidue` 并入同页；CLASSIFIED＋客户端 CreateFX。 | accepted（批 M）：1 个混合 FX 注册 |
| `prefabs/chum_aoe.lua` | `chumpiece` 并入同页；edible＋FX、无 AnimState。 | accepted（批 M）：1 个混合 FX 注册 |
| `prefabs/grass.lua` | `grasspartfx` 并入同页；animover 自删＋拾取音。 | accepted（批 M）：1 个 FX 注册 |
| `prefabs/gelblob.lua` | `gelblob_item_fx` 并入同页；KillFX 播 impact。 | accepted（批 M）：1 个 FX 注册 |
| `prefabs/deer.lua` | `deer_unshackle_fx`、`deer_growantler_fx` 并入同页；2 秒 ErodeAway。 | accepted（批 M）：2 个 FX 注册 |
| `prefabs/fishingnetvisualizerfx.lua` | `fishingnetvisualizerfx` 并入同页；animover 双端自删。 | accepted（批 M）：1 个 FX 注册 |
| `prefabs/eye_charge.lua` | `eye_charge` 并入 `eye_charge.md`；projectile 载体，伤害归炮塔 combat。 | accepted（批 M）：1 个混合 FX 注册 |
| `prefabs/chum_aoe.lua` | `chum_aoe` 并入同页；诱鱼区不结算伤害/定身。 | accepted（批 M）：1 个混合 FX 注册 |
| `prefabs/gelblob.lua` | `gelblob_small_fx` 并入同页；吸附/减速归 gelblob_attach_fx。 | accepted（批 M）：1 个混合 FX 注册 |
| `prefabs/fused_shadeling_bomb.lua` | `fused_shadeling_quickfuse_bomb` 并入同页；AOE 伤害归 do_explosion_effect。 | accepted（批 M）：1 个混合 FX 注册 |

外围批 N（2026-11-04，命中落点／传送门投影／碎屑皮肤组，生成中）：8 文件、2 页、9 个 FX 注册（命中与落点 7／碎屑与吊舱 2）。页面已生成并登记 fx-world-index，待独立复审与 record：

| 文件 | 注册名与结论 | 状态 |
|---|---|---|
| `prefabs/impact.lua` | `impact` 并入 `impact.md`；联网壳＋本地代理、0.5 秒自删、全 scripts 15 处调用。 | accepted（批 N）：1 个 FX 注册 |
| `prefabs/houndstooth_blowpipe.lua` | `houndstooth_proj` 并入同页；弹体自带 weapon 结算，尾迹独立非联网。 | accepted（批 N）：1 个混合 FX 注册 |
| `prefabs/junkball.lua` | `junkball_fx`（第一段纯表现）与 `junkball_fall_fx`（combat 落点）并入同页。 | accepted（批 N）：2 个 FX 注册 |
| `prefabs/lunarrift_terraformer_explosion.lua` | `lunarrift_terraformer_explosion` 并入同页；直接 `GetAttacked`、半地块对角线半径。 | accepted（批 N）：1 个混合 FX 注册 |
| `prefabs/lunarrift_portal.lua` | `lunarrift_portal_shadow` 并入同页；`ismasersim` 拼写陷阱三处标注。 | accepted（批 N）：1 个 FX 注册 |
| `prefabs/multiplayer_portal.lua` | `multiplayer_portal_moonrock_fx` 并入同页；非洞穴 master 生成并父挂。 | accepted（批 N）：1 个 FX 注册 |
| `prefabs/ruins_cavein_obstacle.lua` | `ruins_cavein_obstacle_rubble_fx` 并入 `ruins_cavein_obstacle_rubble_fx.md`；按宿主多色继承、animover 自删。 | accepted（批 N）：1 个 FX 注册 |
| `prefabs/scienceprototyper.lua` | `researchlab2_pod_fx` 并入同页；皮肤挂载、DECOR/NOCLICK、皮肤清除函数移除。 | accepted（批 N）：1 个 FX 注册 |

外围批 O（2026-11-04，皮肤火星／地面痕迹／粒子降水组，生成中）：9 文件、3 页、9 个 FX 注册（随身连缀与皮肤 4／地面痕迹 3／粒子降水 2）。页面已生成并登记 fx-world-index，待独立复审与 record：

| 文件 | 注册名与结论 | 状态 |
|---|---|---|
| `prefabs/carnival_sparkle_bush.lua` | `carnival_sparkle_bush` 并入 `carnival_sparkle_bush.md`；prefabskin 的 SKIN_FX_PREFAB 动态挂到浆果丛皮肤。 | accepted（批 O）：1 个 FX 注册 |
| `prefabs/carrot_spinner.lua` | `carrot_spinner` 并入同页；FX/NOCLICK＋timer 淡入淡出，veggies 生成。 | accepted（批 O）：1 个 FX 注册 |
| `prefabs/fan_wheel.lua` | `fan_wheel` 并入同页；非网络子实体跟 swap_object、net_bool 转动。 | accepted（批 O）：1 个 FX 注册 |
| `prefabs/critterbuff_lunarmoth.lua` | `critterbuff_lunarmoth` 并入同页；Transform＋Light＋net_smallbyte 闪烁。 | accepted（批 O）：1 个 FX 注册 |
| `prefabs/damp_trail.lua` | `damp_trail` 并入 `damp_trail.md`；SetVariation/Refresh、0 帧兜底。 | accepted（批 O）：1 个 FX 注册 |
| `prefabs/poopcloud.lua` | `poopcloud` 并入同页；网络壳＋本地代理、壳 1 秒删。 | accepted（批 O）：1 个 FX 注册 |
| `prefabs/petrify_announce.lua` | `petrify_announce` 并入同页；纯音效、20 距离归一化。 | accepted（批 O）：1 个 FX 注册 |
| `prefabs/raindrop.lua` | `raindrop` 并入 `raindrop.md`；非网络池化、GetPooledFx。 | accepted（批 O）：1 个 FX 注册 |
| `prefabs/frostbreath.lua` | `frostbreath` 并入同页；VFXEffect 单发射器，frostybreather/chester 生成。 | accepted（批 O）：1 个 FX 注册 |

外围批 P（2026-11-04，阿比盖尔子特效／海树表现／怀表传送门组，生成中）：8 文件、3 页、20 个 FX 注册（阿比盖尔 5／海树 10／怀表传送门 5）。页面已生成并登记 fx-world-index，待独立复审与 record：

| 文件 | 注册名与结论 | 状态 |
|---|---|---|
| `prefabs/abigail.lua` | `abigail_retaliation`、`abigail_vex_hit`、`abigail_vex_debuff`、`abigail_vex_shadow_debuff`、`abigail_murder_buff` 并入 `abigail_retaliation.md`；均为随本体返回的 FX 子注册。 | accepted（批 P）：5 个 FX 注册 |
| `prefabs/oceantree.lua` | `oceantree_ripples_{short,normal,tall}`、`oceantree_roots_{short,normal,tall}`、`oceantree_falling` 并入 `oceantree_ripples_short.md`。 | accepted（批 P）：7 个 FX 注册 |
| `prefabs/oceantree_pillar.lua` | `oceantree_pillar_ripples`、`oceantree_pillar_roots`、`oceantree_pillar_leaves` 并入同页。 | accepted（批 P）：3 个 FX 注册 |
| `prefabs/pocketwatch.lua` | `pocketwatch_warp_marker`、`pocketwatch_recall_marker` 并入 `pocketwatch_warp_marker.md`；回溯定位与召回标记。 | accepted（批 P）：2 个 FX 注册 |
| `prefabs/pocketwatch_portal.lua` | `pocketwatch_portal_entrance_overlay`、`_underlay`、`_exit_fx` 并入同页；传送门三层与落地 FX。 | accepted（批 P）：3 个 FX 注册 |

批次 P 验收（2026-11-04）：独立复审 P1×4（暗影怨咒方向写成“反伤”、怨咒只在突击命令、柱体 build 名、overlay/underlay 把 SetParent 当 AddChild）＋P2×6（玩家花冠链漏立即 20、×3 死代码理由、阿比盖尔自身 1/1.1 归一、murdered 来源、树桩 growable 证据、标记清理与补建冲突）＋P3 若干已修。`abigail_retaliation` 因增补内容改为 complex（17,457 B）；3 页页面／导航通过，strict 239 引用 0 错 0 警，record 后 35 组 0 漂移；未运行游戏。

外围批 Q（2026-11-04，影柱法术体系／梦魇鞍具组，生成中）：3 文件、2 页、6 个 FX 注册（影柱 4／鞍具 2）。页面已生成并登记 fx-world-index，待独立复审与 record：

| 文件 | 注册名与结论 | 状态 |
|---|---|---|
| `prefabs/shadow_pillar.lua` | `shadow_pillar`（玩法柱）＋`shadow_pillar_base_fx`／`_target`／`_spell` 并入 `shadow_pillar.md`；地面贴片、隐藏 root 辅助与 CLASSIFIED 扫描法术。 | accepted（批 Q）：4 个注册 |
| `prefabs/saddle.lua` | `saddle_shadow_fx` 并入 `saddle_shadow_fx.md`；网络代理 FX、本地跟随帧与三段改主链。 | accepted（批 Q）：1 个 FX 注册 |
| `prefabs/saddle_shadow_footprints.lua` | `saddle_shadow_footprint` 并入同页；地面脚印贴片、随机 offset/art、updatelooper。 | accepted（批 Q）：1 个 FX 注册 |

批次 Q 验收（2026-11-04）：独立复审 P1×0、P2×6（`shadow_pillar` 的 `t < dt` 条件、1.25 秒停扫描、重施驱散主路径、柱体/辅助目标读档差异；`saddle_shadow_fx` 的 DetachChild 实参错误与脚印池回收条件）＋P3 若干已修。2 页页面／导航通过，strict 125 引用 0 错 0 警，record 后 22 组 0 漂移；未运行游戏。

外围批 R（2026-11-04，睡眠云／装备持握层组）：7 文件、2 页、11 个 FX 注册（睡眠云与孢子云 5／装备持握层 6；表头旧写"4 文件"有误，以分节 7 行为准）。页面已生成并登记 fx-world-index，已独立复审与 record：

| 文件 | 注册名与结论 | 状态 |
|---|---|---|
| `prefabs/sleepcloud.lua` | `sleepcloud`／`_lunar`／`_overlay` 并入 `sleepcloud.md`；范围催眠、月版底座与 7 片 overlay。 | accepted（批 R）：3 个注册 |
| `prefabs/sporecloud.lua` | `sporecloud`／`_overlay` 并入同页；范围伤害/腐败与 overlay。 | accepted（批 R）：2 个注册 |
| `prefabs/staff_lunarplant.lua` | `staff_lunarplant_fx` 并入 `staff_lunarplant_fx.md`；Follower/highlightchild 持握层。 | accepted（批 R）：1 个 FX 注册 |
| `prefabs/sword_lunarplant.lua` | `sword_lunarplant_blade_fx` 并入同页；双层 blade。 | accepted（批 R）：1 个 FX 注册 |
| `prefabs/thurible.lua` | `thuriblebody` 并入同页；持握皮肤层。 | accepted（批 R）：1 个 FX 注册 |
| `prefabs/slingshot.lua` | `slingshotparts_fx` 并入同页；六层部件跟随。 | accepted（批 R）：1 个 FX 注册 |
| `prefabs/snowman.lua` | `snowmanhat_fx`、`snowball_rolling_fx` 并入同页。 | accepted（批 R）：2 个 FX 注册 |

批次 R 验收（2026-11-04）：独立复审 P1×2（`sleepcloud` 漏 `STRINGS.NAMES.SPORECLOUD`＝“孢子云”官方名与 po sources；`staff_lunarplant_fx` 把剑破损写成隐藏 blade，实际不隐藏）＋P2×5（sleepcloud 锚点起点、4 秒语义、countdownbeta 死代码、标签端归属；staff 剑浮空高亮属主）＋P3 若干已修。2 页页面／导航通过，strict 191 引用 0 错 0 警，record 后 35 组 0 漂移；`staff_lunarplant_fx` 保持 32,264 B（余 504 B）。

外围批 S（2026-11-04，影陷阱／暗影消散／散件贴地组）：8 文件、3 页、10 个 FX 注册（影陷阱 2／暗影消散 4／散件 4；表头旧写"9 文件"有误，以分节 8 行为准）。页面已生成并登记 fx-world-index，已独立复审与 record：

| 文件 | 注册名与结论 | 状态 |
|---|---|---|
| `prefabs/shadow_trap.lua` | `shadow_trap`（玩法）＋`shadow_trap_debuff_fx` 并入 `shadow_trap.md`；CLASSIFIED 减速贴片。 | accepted（批 S）：1 个 FX 注册 |
| `prefabs/shadowwaxwell.lua` | `shadow_despawn` 并入 `shadow_despawn.md`；11 处生成点、平台父挂。 | accepted（批 S）：1 个 FX 注册 |
| `prefabs/stageusher.lua` | `stageusher_attackarm`（FX）＋`_attackhand`（混合 80 伤害归 owner）并入同页。 | accepted（批 S）：2 个注册 |
| `prefabs/shrouden_optic_blast.lua` | `shrouden_optic_blast_fx` 并入同页；武器 0＋位面 5 与施法者 35 合并为 40。 | accepted（批 S）：1 个混合注册 |
| `prefabs/scorchedground.lua` | `scorchedground` 并入 `scorchedground.md`；静态布局放置、存读 anim/rotation。 | accepted（批 S）：1 个 FX 注册 |
| `prefabs/shadow_battleaxe.lua` | `shadow_battleaxe_fx` 并入同页；跟随帧层、双 netvar。 | accepted（批 S）：1 个 FX 注册 |
| `prefabs/sisturn.lua` | `sisturn_moon_petal_fx` 并入同页；loop/pst 状态机、AddChild。 | accepted（批 S）：1 个 FX 注册 |
| `prefabs/sporebomb.lua` | `sporebomb` 并入同页；附着 debuff、timer 引爆后生成 sporecloud。 | accepted（批 S）：1 个 FX 注册 |

批次 S 验收（2026-11-04）：独立复审 P1×0、P2×6（`shadow_trap` 漏 `scrapbook_prefabs` source、`NIGHTMARE_TIME` 常量名、撞船驱散分支；`shadow_despawn` 把 glob 也写成平台解父；`scorchedground` 反查“仅命中”不全、`ms_updatesisturnstate` 只锚移除分支）＋P3 若干已修；`shadow_trap` 压到 16,381 B（余 3 B）。3 页页面／导航通过，strict 237 引用 0 错 0 警，record 后 69 组 0 漂移；未运行游戏。

外围批 T（2026-11-04，书页挂载／黏液飞溅／暗影光源组）：3 页、5 文件、29 个 FX 注册（书页 12／goosplash 9／暗影光源 8）。页面已登记 fx-world-index；表头旧写"8 文件"有误，以实际 5 文件为准：

| 文件 | 注册名与结论 | 状态 |
|---|---|---|
| `prefabs/books.lua` | 12 个 `fx_*_over/under_book(+_mount)` 并入 `fx_book_mount.md`；动态拼名、SGwilson book2 唯一生成点。 | accepted（批 T）：12 个 FX 注册 |
| `prefabs/goosplash.lua` | 9 个 goosplash 变体并入 `goosplash.md`；脚本层无生成点（事件数据未覆盖）。 | accepted（批 T）：9 个 FX 注册 |
| `prefabs/nightmarelightfx.lua` | 4 个 nightmarelight 变体并入 `nightmarelightfx.md`；DECOR+NOCLICK、父挂。 | accepted（批 T）：4 个 FX 注册 |
| `prefabs/gestalt_head.lua` | `gestalt_head`／`gestalt_guard_head` 并入同页；引擎 Follower、无 Network。 | accepted（批 T）：2 个 FX 注册 |
| `prefabs/circlingbuzzard.lua` | `circlingbuzzard`／`_lunar` 并入同页；net_byte/net_bool 淡入淡出。 | accepted（批 T）：2 个 FX 注册 |

批次 T 验收（2026-11-04）：独立复审 P1×0、P2×0；逐页对源码核验 `MakeFX` 工厂与注册循环、`MakeBook` deps 仅声明、SGwilson book2 生成／帧跳过／44／50 帧结算时序／onexit 兜底、`goosplash` 9 变体 num→两层动画映射与 flip 不对称镜像、`nightmarelightfx` DECOR/NOCLICK 与 `fissure_grottowarfx` 的 `mouseparent="fissure"` 判定不命中、`gestalt_head` 非联网与宿主侧 `persists`、`circlingbuzzard` fade netvar 与 `KillShadow` 全部 6 处调用点（含 `mutatedbuzzardcircler` L47／133／219／253／327 与管理器 L178）；po 官方名锚点 6 组与无键声明复核成立；`fx_over`／`fx_under` 全库反查仅 2 文件、`"fx_"..` 拼名仅 books.lua 4 处、`goosplash` 全库 7 行 3 文件均与页内验证记录一致。3 页页面／导航通过，strict 206 引用 0 错 0 警；record 3 页后 check 0 漂移；2 段 Lua 语法通过；未运行游戏。

批次 T 复审计实（2026-11-05，接手会话）：主会话对三页做第二次逐锚点独立复审，前次验收成立——`fx_book_mount` 的工厂／注册循环／6 书目 def／SGwilson book2 状态链（`book` 态回背包→`book2` 生成、SetFrame L10137/L10143、44 帧 fn50 与 50 帧执行、earlycast fn19、onexit 双 Remove）与 `goosplash` 的 9 变体映射（`math.ceil(n/4)`、`4+variation` 溅花、flip 2/4 与 >2 不对称）、外部引用（netvars L7、entityscript L840-850、peghook L32-36/242/313-318、prefablist L517、`goosplash` 全库 7 行 3 文件）与 `nightmarelightfx` 的 DECOR 表达式（L26 注释与条件）、`fissure_grottowar` mouseparent 不匹配、`KillShadow` 5+1 调用点、`_num_circling_buzzards<10`（L37 常量 10）、HATGUARD_SCALE=0.4、po 六组官方名逐字命中全部复核属实。修复 P2×2：`fx_book_mount`／`nightmarelightfx` 验证记录引用旧尺寸预算（standard 200 行/16 KiB、complex 400 行/32768 字节），改按当前 write-format（240/24 与 480/48）；只改正文，sources 未变，基线 check 0 漂移维持。另查明旧预算引用波及 13 个历史已验收页（组件 5＋外围批 C～S 8），属规则变更后未回填的历史口径，按"历史页不批量迁移"不改动，最终报告如实记录。

索引拆分（2026-11-04）：原 `fx-world-index` 达 16.4 KiB 上限，按主题拆为 `fx-world-index`（环境与世界）与 `fx-creature-index`（生物与Boss），父索引 `prefabs/index.md` 两行登记；页面尾链暂保留指向 `fx-world-index`（其引言链到生物索引），后续统一改链。

批次 O 验收（2026-11-04）：独立复审 P1×0、P2×4（`damp_trail` 把已结束任务写成 `nil`、poopcloud 壳/代理消失方向写反；`carnival_sparkle_bush` “饥饿发光”条件写反、网络壳“只有 Transform”与宿主加 Follower 矛盾）＋P3 若干。3 页页面／导航通过，strict 174 引用 0 错 0 警（raindrop 59／damp 55／sparkle 60），record 后 48 组 0 漂移；未运行游戏。

批次 N 验收（2026-11-04）：独立复审 P1×0、P2×5（`impact` 代表链接计数、吹箭与回旋镖朝向、爆炸 ONEOF 过滤；碎屑页 `OnWork` 顺序、`charge_barrier` 机制描述）＋P3 若干已修。2 页页面／导航通过，strict 165 引用 0 错 0 警（impact 125／rubble 40），record 后 32 组 0 漂移；未运行游戏。

### 外围逐文件结论（只读分诊批 5，2026-11-04，30 文件）

结论：给定 30 条路径中 **15 条在本 buildid 不存在**（归档亦无），属候选清单路径/名称错误；其余 15 个文件仅 2 个真 FX 注册。45 个注册名（2 FX＋6 placer＋37 玩法）：covered-existing 1（`opalstaff`，实际在 `staff.lua`）、needs-page 2（`ruins_cavein_obstacle_rubble_fx`、`researchlab2_pod_fx`）、excluded 19、blocked 8（路径错误）。

| 文件 | 注册名（行号）与结论 | 状态 |
|---|---|---|
| `prefabs/opalstaff.lua` | 文件不存在；`opalstaff` 实为 `staff.lua:309`。 | covered-existing（`prefabs/opalstaff.md`） |
| `prefabs/panflute.lua` | `panflute`（instrument/inspectable/tool/finiteuses）。 | excluded（玩法） |
| `prefabs/paperlantern.lua` | 文件与同名注册均不存在。 | blocked（路径错误） |
| `prefabs/petals.lua` | `petals`（edible/fuel/perishable/stackable/upgrader/dryable）。 | excluded（玩法） |
| `prefabs/pig_house.lua` | 文件不存在；实为 `pighouse.lua:512`＋placer；内部 MakeWindow 为未注册 DECOR。 | excluded（玩法） |
| `prefabs/pigman.lua` | `pigman`／`pigguard`／`moonpig`（locomotor/health/combat/werebeast/trader）。 | excluded（玩法） |
| `prefabs/pigtorch.lua` | `pigtorch`／`pigtorch_fuel`；火焰已由 `fire.md` 覆盖。 | excluded（玩法＋covered FX） |
| `prefabs/pillow_fight.lua` | 文件不存在；实为 yotr_fightring 事件串＋动态名。 | blocked（动态名/路径错误） |
| `prefabs/playbill.lua` | 四个 playbill 注册（inventoryitem/furnituredecor/fuel）。 | excluded（玩法） |
| `prefabs/pond.lua` | `pond`／`pond_mos`／`pond_cave`。 | excluded（结构） |
| `prefabs/portable_sail.lua` | 文件与同名注册均不存在（近似 `mast`）。 | blocked（路径错误） |
| `prefabs/pottedfern.lua` | `pottedfern`＋placer。 | excluded（玩法） |
| `prefabs/powder_monkey.lua` | 文件不存在；实为 `powdermonkey.lua:356`。 | excluded（玩法） |
| `prefabs/pumpkin_lantern.lua` | `pumpkin_lantern`（Light＋combat/health/perishable）。 | excluded（玩法） |
| `prefabs/rabbit_house.lua` | 文件不存在；实为 `rabbithouse.lua:270`＋placer。 | excluded（玩法；皮肤 FX 已有页） |
| `prefabs/raincoat.lua` | `raincoat`。 | excluded（玩法） |
| `prefabs/raft.lua` | 文件与同名注册均不存在（船体 `boat`）。 | blocked（路径错误） |
| `prefabs/rainometer.lua` | `rainometer`＋placer。 | excluded（玩法） |
| `prefabs/reeds.lua` | `reeds`（pickable/inspectable/fuel）。 | excluded（玩法） |
| `prefabs/reflective_armor.lua` | 文件与同名注册均不存在（近似 `reflectivevest`）。 | blocked（路径错误） |
| `prefabs/reindeer.lua` | 文件与同名注册均不存在（近似 `deer`）。 | blocked（路径错误） |
| `prefabs/researchlab.lua` | 文件不存在；实为 `scienceprototyper.lua:299/300`＋placers＋`researchlab2_pod_fx`（DECOR/NOCLICK、persists=false、0 组件、唯一挂载 skinprefabs）。 | needs-page（1 个 FX） |
| `prefabs/rock_avocado.lua` | 文件不存在；实为 `rock_avocado_bush.lua:328`、`rock_avocado_fruit.lua:437` 等。 | excluded（玩法） |
| `prefabs/rock_light.lua` | `rock_light`（burnable/fueled/workable/sanityaura）。 | excluded（玩法；外部 `lavalight` 无页为关联缺口） |
| `prefabs/rocky.lua` | `rocky`。 | excluded（玩法） |
| `prefabs/rose.lua` | 文件不存在；`rose` 只是 `flower.lua` 的动画名。 | blocked（路径错误） |
| `prefabs/rotator.lua` | 文件不存在；实为 `boat_rotator`／`fence_rotator`。 | excluded（内部实体） |
| `prefabs/rowboat.lua` | 文件与同名注册均不存在（`boat`）。 | blocked（路径错误） |
| `prefabs/ruins_bat.lua` | `ruins_bat`。 | excluded（玩法） |
| `prefabs/ruins_cavein_obstacle.lua` | `ruins_cavein_obstacle`＋`ruins_cavein_obstacle_rubble_fx`（FX、break_small、animover→Remove、唯一生成点 L28）。 | needs-page（1 个 FX） |

### 外围逐文件结论（只读分诊批 6，2026-11-04，30 文件）

结论：82 注册（9 文件纯 FX／6 混合／14 玩法／0 未注册内部实体；4 处文件名与注册名不一致：`oceanshadowcreature`→`oceanhorror`、`fossil_spike`→`fossilspike`、`pocketwatch` 无本体、`rift_terraformer`→`lunarrift_terraformer_visual`）。covered-existing 0、excluded 19、needs-page 9、blocked 2（熔炉事件包）。

| 文件 | 注册名与结论 | 状态 |
|---|---|---|
| `prefabs/abigail.lua` | 主 `abigail` 玩法；`abigail_retaliation`／`_vex_hit`／`_vex_debuff`／`_vex_shadow_debuff`／`_murder_buff` 为 FX 子注册。 | excluded＋FX 子注册待合并 |
| `prefabs/carnival_sparkle_bush.lua` | `carnival_sparkle_bush` 纯 FX；prefabskin 的 SKIN_FX_PREFAB 动态挂到浆果丛皮肤。 | needs-page（1 个 FX） |
| `prefabs/carrot_spinner.lua` | `carrot_spinner` FX/NOCLICK＋timer/updatelooper 淡入淡出；veggies 生成。 | needs-page（1 个 FX） |
| `prefabs/charlie_boss_trial.lua` | 混合（FX 标签＋charliearenawatcher/temperatureoverrider/entitytracker）。 | excluded（混合玩法） |
| `prefabs/charlie_boss_vines.lua` | 混合（FX/NOCLICK 但 weapon/projectile/locomotor 会伤害）。 | excluded（混合玩法） |
| `prefabs/critterbuff_lunarmoth.lua` | `critterbuff_lunarmoth` 纯 FX（Transform＋Light＋net_smallbyte）；critters 生成。 | needs-page（1 个 FX） |
| `prefabs/damp_trail.lua` | `damp_trail` 纯 FX（SetVariation/Refresh、0 帧自删）；stalker 生成。 | needs-page（1 个 FX） |
| `prefabs/fan_wheel.lua` | `fan_wheel` 纯 FX（非网络子实体跟 swap_object、net_bool 转动）；minifan 生成。 | needs-page（1 个 FX） |
| `prefabs/fireball_projectile.lua` | 6 名；表现可核验，但玩法走缺失的 lavaarena 事件包。 | blocked（事件包） |
| `prefabs/fireballstaff.lua` | `fireballstaff` 玩法走缺失事件包；`fireball_cast_fx` 无生成点。 | blocked（事件包） |
| `prefabs/floatinglantern.lua` | 主物品玩法；`floatinglantern_shadow` 为 FX 子注册。 | excluded＋FX 子注册待合并 |
| `prefabs/fossil_spike.lua` | `fossilspike` 混合（combat 10 伤害）；`fossilspike_base` 为 FX。 | excluded＋FX 子注册待合并 |
| `prefabs/fossil_spike2.lua` | `fossilspike2` 混合；`fossilspike2_base` 为 FX。 | excluded＋FX 子注册待合并 |
| `prefabs/frostbreath.lua` | `frostbreath` 纯 VFXEffect 粒子；frostybreather/chester 生成。 | needs-page（1 个 FX） |
| `prefabs/oceanshadowcreature.lua` | 注册名为 `oceanhorror` 玩法；`_ripples`／`_attachpivot` 为 FX 子注册。 | excluded＋FX 子注册待合并 |
| `prefabs/oceantree.lua` | 6 树玩法＋7 FX 子注册（ripples/roots/falling）。 | excluded＋FX 子注册待合并 |
| `prefabs/oceantree_pillar.lua` | 树玩法＋3 FX 子注册（ripples/roots/leaves）。 | excluded＋FX 子注册待合并 |
| `prefabs/oceanwhirlbigportal.lua` | 两注册均为传送门玩法（wateryprotection/oceanwhirlportalphysics/worldmigrator）。 | excluded（玩法） |
| `prefabs/penguin_ice.lua` | 混合（slipperyfeettarget 玩法区域＋CreateIceFX 附属表现）。 | excluded（混合玩法） |
| `prefabs/petrify_announce.lua` | `petrify_announce` 纯音效 FX（无 AnimState、播后即删、20 距离归一化）。 | needs-page（1 个 FX） |
| `prefabs/pocketwatch.lua` | 7 名；5 玩法＋2 marker FX（warp/recall）。 | excluded＋FX 子注册待合并 |
| `prefabs/pocketwatch_portal.lua` | 3 玩法＋3 FX（overlay/underlay/exit_fx）。 | excluded＋FX 子注册待合并 |
| `prefabs/poopcloud.lua` | `poopcloud` 纯 FX（proxy＋animover）；compost/guano 生成。 | needs-page（1 个 FX） |
| `prefabs/propsign.lua` | `propsign` 玩法；`propsignshatterfx` 为 FX。 | excluded＋FX 子注册待合并 |
| `prefabs/pumpkin_bomb.lua` | `pumpkin_bomb` 玩法；`pumpkin_bomb_explode_fx` 为 FX；另有 2 个未注册内部实体。 | excluded＋FX 子注册待合并 |
| `prefabs/purebrilliance.lua` | `purebrilliance` 玩法；`purebrilliance_symbol_fx` 为 FX（winona_battery_high 生成）。 | excluded＋FX 子注册待合并 |
| `prefabs/raindrop.lua` | `raindrop` 纯 FX（非网络池化、GetPooledFx）；rain/caverain/lunarhail 生成。 | needs-page（1 个 FX） |
| `prefabs/rift_terraformer.lua` | 主为地皮改造逻辑（无 AnimState）；`lunarrift_terraformer_visual` 为 FX。 | excluded＋FX 子注册待合并 |
| `prefabs/rocky_boss.lua` | Boss 玩法；`rocky_snip_fx`／`rocky_dash_fx`／`rocky_boss_shadow` 为 FX 子注册。 | excluded＋FX 子注册待合并 |
| `prefabs/rope_bridge.lua` | kit 玩法＋placer；`rope_bridge_fx` 为 FX（ropebridgemanager 生成）。 | excluded＋FX 子注册待合并 |

### 外围逐文件结论（只读分诊批 7，2026-11-04，30 文件）

结论：30 条路径中 7 条不存在（`sinkholespawner`、`skullchest`、`snurtle`、`soul`、`spat_splash`、`spawnlight`、`spring_antlion`，已映射/归 blocked）；90 个注册名：needs-page 12 文件／30 注册（纯 FX 12 个）、covered-existing 2（`shadowthrall_projectile_fx`、`stafflight` 家族）、excluded 9、blocked 7。可合并建页 8 组。

| 文件 | 注册名与结论 | 状态 |
|---|---|---|
| `prefabs/saddle.lua` | 5 种鞍玩法＋`saddle_shadow_fx`（网络代理 FX、本地跟随帧 FollowSymbol）。 | needs-page（1 个 FX） |
| `prefabs/saddle_shadow_footprints.lua` | `saddle_shadow_footprint` 地面脚印贴片（FX/NOCLICK、OnGround 背景层、updatelooper）。 | needs-page（1 个 FX） |
| `prefabs/scorchedground.lua` | `scorchedground` 地面焦痕贴片（10 种随机 idle、存读 anim/rotation）。 | needs-page（1 个 FX） |
| `prefabs/shadow_battleaxe.lua` | 武器玩法＋`shadow_battleaxe_fx`（跟随符号、双 net 变量、highlightchild）。 | needs-page（1 个 FX） |
| `prefabs/shadow_pillar.lua` | 法术柱＋`shadow_pillar_base_fx`、`shadow_pillar_target`、`shadow_pillar_spell`（隐藏辅助/地面 FX/CLASSIFIED）。 | needs-page（3 个注册） |
| `prefabs/shadow_trap.lua` | 陷阱＋`shadow_trap_debuff_fx`（CLASSIFIED 规避悬停、FinalOffset）。 | needs-page（1 个 FX） |
| `prefabs/shadowchesspieces.lua` | 3 暗影棋子 Boss＋`shadow_bishop_fx`（feather_loop 一帧后 animover）。 | needs-page（1 个 FX） |
| `prefabs/shadowrift_portal.lua` | 裂隙传送门＋`shadowrift_portal_fx`（粒子 FX、PlayStage/Disappear）。 | needs-page（1 个 FX） |
| `prefabs/shadowthrall_mouth.lua` | 巨口怪＋`shadowthrall_mouth_dupe_fx`（colouradder/bloomer、animover）。 | needs-page（1 个 FX） |
| `prefabs/shadowthrall_parasite.lua` | 寄生体＋`shadowthrall_parasite_fx`（Tag 用大写 "fx" 的异例）。 | needs-page（1 个 FX） |
| `prefabs/shadowthrall_projectile_fx.lua` | 专属页已覆盖（带 complexprojectile 载伤）。 | covered-existing |
| `prefabs/sharkboi.lua` | Boss 玩法；挥击 FX 在 `sharkboi_fx.lua` 且已有页。 | excluded（玩法） |
| `prefabs/sharkboi_ice_hazard.lua` | 可采冰岩玩法（SetPrefabNameOverride rock_ice）；ice_splash 属既有 FX。 | excluded（玩法） |
| `prefabs/sisturn.lua` | 思魂坛结构＋`sisturn_moon_petal_fx`（lunar fx 状态机）。 | needs-page（1 个 FX） |
| `prefabs/sleepbomb.lua` | 投掷物玩法。 | excluded（玩法） |
| `prefabs/slingshotammo.lua` | 18 数据项→35 注册，全为物品/投射物；命中 FX 在 `fx.lua` 且已有页。 | excluded（玩法） |
| `prefabs/slurper.lua` | 生物玩法；`slurperlight` 已被 `wormlight.md` 收录。 | excluded＋covered FX |
| `prefabs/smallbird.lua` | 雏鸟/青年鸟玩法。 | excluded（玩法） |
| `prefabs/spiderden.lua` | 三级蜘蛛巢玩法。 | excluded（玩法） |
| `prefabs/spiderqueen.lua` | Boss 玩法。 | excluded（玩法） |
| `prefabs/sporebomb.lua` | `sporebomb` 附着型 debuff FX（FX 标签＋私有无网络光＋timer 引爆）。 | needs-page（1 个 FX） |
| `prefabs/staff.lua` | 9 法杖玩法；`yellowstaff`／`opalstaff` 已有页，其余非 FX。 | excluded（玩法） |
| `prefabs/stafflight.lua` | 4 名均有页（stafflight／staffcoldlight／staffcoldlightfx／emberlight）。 | covered-existing |
| `prefabs/snurtle.lua`（不存在） | 实为 `slurtle.lua:264` 注册的 `snurtle`，玩法。 | blocked（路径不存在） |
| `prefabs/sinkholespawner.lua`（不存在） | 无 prefab，仅组件；`components/sinkholespawner.md` 已覆盖。 | blocked（路径不存在）＋excluded |
| `prefabs/skullchest.lua`（不存在） | 无实体，仅 `containers.lua` UI 参数。 | blocked（路径不存在）＋excluded |
| `prefabs/soul.lua`（不存在） | 实为 `wortox_soul.lua`，玩法；`components/soul.md` 已有页。 | blocked（路径不存在） |
| `prefabs/spat_splash.lua`（不存在） | 实际 `fx.lua` 的 5 个 spat 注册，已有 `spat_splat_fx.md`。 | blocked（路径不存在）＋covered |
| `prefabs/spawnlight.lua`（不存在） | 实为 `spawnlight_multiplayer.lua`，已有 `cavelight_small.md` 覆盖。 | blocked（路径不存在）＋covered |
| `prefabs/spring_antlion.lua`（不存在） | 全库无注册/引用。 | blocked（路径不存在） |

### 外围逐文件结论（只读分诊批 8，2026-11-04，30 文件）

结论：95 注册（FX 35／混合 8／玩法 48／技术 4）；covered-existing 3（`slingshotaoefx` 三名同页，其中 2 名仅正文待补 aliases）、needs-page 39（32 FX＋7 混合）、excluded 53、blocked 0。5 名 0 生成点（`spear_gungnir_lungefx`、`spellmasterybuff`／`orb`／`orbs`、`splash_spiderweb`）按诚实缺口记录。命名陷阱：`singingshell` 不注册本体、`shrouden_optic_blast` 注册名带 `_fx`、`townportalsandcoffin_fx` 用 teleport_sand bank、`trap_fumarole` 行号 +3、`trap_vines_base_fx` 复用 shadow_pillar bank。

| 文件 | 注册名与结论 | 状态 |
|---|---|---|
| `prefabs/shadowwaxwell.lua` | `shadow_despawn` 为 FX；11 个 shadow 单位玩法；涟漪池内部实体。 | needs-page（1 FX）＋excluded |
| `prefabs/shatter.lua` | `shatter` 纯 FX（net_tinybyte 双端代理、1 秒自删）。 | needs-page（1 FX） |
| `prefabs/shrouden_optic_blast.lua` | `shrouden_optic_blast_fx` 混合（weapon＋planardamage 光束）。 | needs-page（1 混合） |
| `prefabs/singingshell.lua` | 3 玩法＋`singingshell_critterfx` FX（破壳动画）。 | needs-page（1 FX）＋excluded |
| `prefabs/sleepcloud.lua` | `sleepcloud`／`_lunar` 混合＋`sleepcloud_overlay` FX。 | needs-page（3） |
| `prefabs/slingshot.lua` | 5 玩法＋`slingshotparts_fx` FX（Follower 持握层）。 | needs-page（1 FX）＋excluded |
| `prefabs/slingshotammo_debuffs.lua` | 2 纯 FX＋1 混合（horrorfuel DoT 载体）＋1 玩法。 | needs-page（3）＋excluded |
| `prefabs/slingshotaoefx.lua` | 3 名已被 `slingshot_ice_aoe_fx.md` 正文覆盖（powerup 两名缺 aliases）。 | covered-existing |
| `prefabs/snowman.lua` | 1 玩法＋3 FX（帽子/碎块/滚雪球）。 | needs-page（3）＋excluded |
| `prefabs/spear_gungnir.lua` | 1 玩法＋`spear_gungnir_lungefx`（0 生成点，事件包缺口）。 | needs-page（1 FX） |
| `prefabs/spear_wathgrithr.lua` | 3 玩法＋2 FX（八向突刺光、装备跟随）。 | needs-page（2）＋excluded |
| `prefabs/spellmasterybuff.lua` | 1 混合＋2 FX（事件包缺失、0 生成点）。 | needs-page（3） |
| `prefabs/splash_spiderweb.lua` | `splash_spiderweb` 纯 FX（0 生成点，疑遗留）。 | needs-page（1 FX） |
| `prefabs/sporecloud.lua` | `sporecloud` 混合＋`sporecloud_overlay` FX。 | needs-page（2） |
| `prefabs/staff_lunarplant.lua` | 1 玩法＋`staff_lunarplant_fx`（Follower/highlightchild 持握层）。 | needs-page（1）＋excluded |
| `prefabs/stageusher.lua` | 1 玩法＋`stageusher_attackhand` 混合＋`_attackarm` FX。 | needs-page（2）＋excluded |
| `prefabs/sword_lunarplant.lua` | 1 玩法＋`sword_lunarplant_blade_fx`（双层 blade）。 | needs-page（1）＋excluded |
| `prefabs/thurible.lua` | 1 玩法＋`thuriblebody` FX（持握皮肤层）。 | needs-page（1）＋excluded |
| `prefabs/townportal.lua` | 1 玩法＋placer＋`townportalsandcoffin_fx`（teleport_sand bank）。 | needs-page（1 FX）＋excluded |
| `prefabs/trap_fumarole.lua` | 1 玩法＋placer＋`trap_fumarole_burn_fx`。 | needs-page（1 FX）＋excluded |
| `prefabs/trap_vines.lua` | 1 玩法＋`trap_vines_base_fx`（复用 shadow_pillar bank）。 | needs-page（1）＋excluded |
| `prefabs/treasurechest.lua` | 6 玩法＋placer＋`terrariumchest_fx`。 | needs-page（1 FX）＋excluded |
| `prefabs/tumbleweed.lua` | 1 玩法＋`tumbleweedbreakfx`（四向 break、双清理）。 | needs-page（1）＋excluded |
| `prefabs/vault_compass.lua` | 1 玩法＋marker＋`vault_compass_visual`（贴地 pointer）。 | needs-page（1 FX）＋excluded |
| `prefabs/vault_decon_mister.lua` | 1 玩法＋`vault_decon_mister_fx`（VFX 粒子）。 | needs-page（1）＋excluded |
| `prefabs/vault_ground_pattern.lua` | `vault_ground_pattern_fx` 纯 FX（15 处生成、存档 API）。 | needs-page（1 FX） |
| `prefabs/vault_key_activator.lua` | 6 玩法/技术，无 FX 注册。 | excluded |
| `prefabs/vault_key_trial.lua` | 混合控制器（FX 外壳＋entitytracker），非独立 FX。 | excluded |
| `prefabs/vault_torch.lua` | 1 玩法＋`vault_torch_flame`（coldfire 三档）。 | needs-page（1）＋excluded |
| `prefabs/vine_bridge.lua` | `vine_bridge_fx`／`_decor_fx` 纯 FX（贴地/子片）。 | needs-page（2 FX） |

### 外围逐文件结论（只读分诊批 9，2026-11-04，30 文件）

结论：30 文件全部存在；114 注册（含 6 placer、18 个 `wintercooking_*` 配方占位假 prefab）→ 有效实体 90；FX 31／混合 6／玩法与技术 53／placer 6／占位 18。covered-existing 0、excluded 53、needs-page 37（28 文件）、blocked 1（`woby_rack` 仅 prefablist 有名、全库无注册）。9 个合并建页组；`watertree_pillar_ripples`／`_roots` 建议并入既有海树柱页。

| 文件 | 注册名与结论 | 状态 |
|---|---|---|
| `prefabs/voidcloth_boomerang.lua` | 玩法＋`voidcloth_boomerang_fx`（FollowSymbol 持握层）＋proj 混合。 | needs-page（1 FX）＋excluded |
| `prefabs/voidcloth_scythe.lua` | 玩法＋`voidcloth_scythe_fx`（同款持握层）。 | needs-page（1 FX）＋excluded |
| `prefabs/voidcloth_umbrella.lua` | 玩法＋`voidcloth_umbrella_fx`；另 2 个未注册内部 FX（wave/dome）。 | needs-page（1 FX）＋excluded |
| `prefabs/wagboss_beam.lua` | `wagboss_beam_fx` 混合（FX＋Light＋combat/planardamage 真实结算）。 | needs-page（1 混合） |
| `prefabs/wagboss_robot_leg.lua` | 玩法障碍＋`wagboss_robot_leg_fx`（落地喷雾）。 | needs-page（1 FX）＋excluded |
| `prefabs/wagstaff_npc.lua` | 5 玩法＋2 混合（`_finale_fx` 剪影、`alterguardian_contained`）。 | needs-page（2 混合）＋excluded |
| `prefabs/waterplant_pollen.lua` | `waterplant_pollen_fx`（落海烟）＋`_ocean`（诱饵云，每 1 秒生成 chumpiece）。 | needs-page（2） |
| `prefabs/watertree_pillar.lua` | 玩法＋`_ripples`／`_roots`（同 bank，建议并入海树柱页）。 | needs-page（2，并入既有页） |
| `prefabs/wendy_resurrectiongrave.lua` | 玩法＋placer＋`abigail_gravestone_rebirth_fx`／`wendy_gravestone_rebirth_fx`（同一重生动画）。 | needs-page（2）＋excluded |
| `prefabs/winona_battery_high.lua` | 玩法＋placer＋`_shatterfx`（红宝石碎裂）。 | needs-page（1 FX）＋excluded |
| `prefabs/winona_battery_sparks.lua` | `winona_battery_sparks` FX（不用代理、8 处生成）。 | needs-page（1 FX） |
| `prefabs/winona_catapult_projectile.lua` | 混合投射物，0 个 FX 注册（表现全在未注册内部工厂）。 | excluded |
| `prefabs/winona_spotlight.lua` | 玩法＋placer＋`_head`（灯头/光柱）。 | needs-page（1 FX）＋excluded |
| `prefabs/winona_telebrella.lua` | 玩法＋`_swap_fx`（LED 持握层）。 | needs-page（1 FX）＋excluded |
| `prefabs/wintersfeastbuff.lua` | 玩法 debuff＋`wintersfeastbuff_fx`（随机帧、animover 自删）。 | needs-page（1 FX）＋excluded |
| `prefabs/wintersfeastoven.lua` | 玩法＋`_fire` FX＋placer＋18 个配方占位假 prefab。 | needs-page（1 FX）＋excluded |
| `prefabs/witchbroom.lua` | 玩法＋`witchbroom_fx`（10Hz 轨迹池）。 | needs-page（1 FX）＋excluded |
| `prefabs/woby_rack.lua` | `woby_rack_swap_fx` FX＋container widget；`woby_rack` 仅 prefablist 有名、全库无注册。 | needs-page（1）＋blocked（幽灵名） |
| `prefabs/wolfgang.lua` | 玩法＋`wolfgang_coach_buff` debuff＋`_coach_buff_fx`（Follower）。 | needs-page（1 FX）＋excluded |
| `prefabs/worm_boss.lua` | 10 玩法/部位＋`_dirt_ground_fx`（地面隆隆）。 | needs-page（1 FX）＋excluded |
| `prefabs/wortox.lua` | 玩法/debuff＋`soulecho_buff_fx`／`overloading_fx`；`wortox_panflute_buff_fx_fn` 为未注册死代码。 | needs-page（2 FX）＋excluded |
| `prefabs/wortox_nabbag.lua` | 玩法＋`_body` 混合＋`_body_fx`／`_body_soulfx`。 | needs-page（3）＋excluded |
| `prefabs/wortox_reviver.lua` | 玩法＋`wortox_reviver_body`（FollowSymbol）。 | needs-page（1 FX）＋excluded |
| `prefabs/wortox_soul.lua` | 玩法，0 个 FX 注册（尾部光效为内部实体）。 | excluded |
| `prefabs/wurt.lua` | 玩法＋`_shadow_merm_planar_fx`／`_lunar_merm_planar_fx`。 | needs-page（2 FX）＋excluded |
| `prefabs/wurt_terraform_item.lua` | 4 玩法＋2 个 chargedfx（同工厂仅 build 不同）＋projectile 混合。 | needs-page（2 FX）＋excluded |
| `prefabs/wx78_backupbody.lua` | 玩法＋placer＋`_inventory` 混合（机体渲染层）。 | needs-page（1 混合）＋excluded |
| `prefabs/wx78_big_spark.lua` | `wx78_big_spark` FX（代理无 AnimState，闪白路径不可达）。 | needs-page（1 FX） |
| `prefabs/wx78_lightbeam.lua` | `wx78_lightbeam` FX（无 AnimState、本地灯点链）。 | needs-page（1 FX） |
| `prefabs/wx78_scanner.lua` | 4 玩法＋`_fx`（扫描锥、AddChild 到目标）。 | needs-page（1 FX）＋excluded |

### 外围逐文件结论（只读分诊批 10，2026-11-04，30 文件）

结论：272 注册（FX 70／混合 9／玩法 163／技术 30）；covered-existing 0、excluded 191（10 文件整文件排除）、needs-page 75（18 文件）、blocked 6（2 个吹箭文件的事件包）。9 个合并建页组；陷阱：`mushtree` 焦壳动态拼名、`books` 的 deps≠生成、`burntground_faded` 用 `SetPrefabName`、`goosplash` prefs 依赖、`forcefieldfx` 属图勒护盾帽。

| 文件 | 注册名与结论 | 状态 |
|---|---|---|
| `prefabs/archive_lockbox.lua` | 2 玩法＋2 技术（空声源壳、worldgen 占位）。 | excluded |
| `prefabs/blowdart_lava.lua` | 1 玩法＋2 混合；postinit 在缺失 lavaarena 事件包。 | blocked |
| `prefabs/blowdart_lava2.lua` | 3 混合；同一事件包。 | blocked |
| `prefabs/books.lua` | 18 书玩法＋1 隐形任务体＋12 个书页 FX。 | excluded＋needs-page（12） |
| `prefabs/bramblefx.lua` | `_armor`／`_armor_upgrade`／`_trap` 混合（FX 直接结算伤害）。 | needs-page（3） |
| `prefabs/brokentool.lua` | `brokentool` FX（非联网代理、4 处生成）。 | needs-page（1） |
| `prefabs/burntground.lua` | `burntground`／`_faded` FX（64 tick×5 天、3 处生成）。 | needs-page（2） |
| `prefabs/carnivalgame_golfprops.lua` | 24 玩法＋24 placer。 | excluded |
| `prefabs/cavein_boulder.lua` | 1 玩法＋`cavein_dust_low`／`_high` FX。 | excluded＋needs-page（2） |
| `prefabs/charliearena_teleporter.lua` | 玩法；内部 base 未注册。 | excluded |
| `prefabs/circlingbuzzard.lua` | `circlingbuzzard`／`_lunar` FX。 | needs-page（2） |
| `prefabs/daywalker.lua` | Boss 玩法；眼焰/头为内部实体。 | excluded |
| `prefabs/daywalker2_items.lua` | 7 个掉落物 FX（1 带 scraphat）。 | needs-page（7） |
| `prefabs/fence_electric_field.lua` | 玩法；段 FX 未注册。 | excluded |
| `prefabs/forcefieldfx.lua` | `forcefieldfx`（图勒护盾帽生成）。 | needs-page（1） |
| `prefabs/gestalt_head.lua` | `gestalt_head`／`gestalt_guard_head` FX（3 处生成）。 | needs-page（2） |
| `prefabs/goosplash.lua` | `goosplash1-8`＋`goosplash` 共 9 FX；快照无生成点（诚实缺口）。 | needs-page（9） |
| `prefabs/hats.lua` | 85 帽玩法＋11 跟随/帽灯 FX＋1 技术。 | excluded＋needs-page（11） |
| `prefabs/honey_trail.lua` | `honey_trail`／`_shrouden_fx` 混合（每 tick 减速）。 | needs-page（2） |
| `prefabs/lighterfire_common.lua` | 工厂模块，0 注册（被 8 个文件 require）。 | excluded |
| `prefabs/moon_altar_break.lua` | `moon_altar_break`／`_claw_`／`_crown_` FX。 | needs-page（3） |
| `prefabs/moonrockseed.lua` | 玩法＋icon 技术。 | excluded |
| `prefabs/mushroombomb.lua` | 4 玩法；地面光晕内部未注册。 | excluded |
| `prefabs/mushtree.lua` | 6 玩法/树＋6 焦壳/Bloom FX（动态拼名）。 | excluded＋needs-page（6） |
| `prefabs/mushtree_moon.lua` | 2 玩法＋`_burntfx` 动态拼名。 | excluded＋needs-page（1） |
| `prefabs/nightmarelightfx.lua` | 4 个 nightmarelight 变体 FX。 | needs-page（4） |
| `prefabs/pillow_equipment.lua` | 8 玩法＋4 攻击碎屑 FX。 | excluded＋needs-page（4） |
| `prefabs/plant_normal.lua` | 2 玩法＋3 个 dig FX。 | excluded＋needs-page（3） |
| `prefabs/portableblender.lua` | 2 玩法＋placer。 | excluded |
| `prefabs/portablecookpot.lua` | 2 玩法＋placer。 | excluded |

批次 M 验收（2026-11-04）：独立复审 P1×1（`eye_charge` 把 `chum_aoe` 读档后的任务重建写成“不重建”）＋P2×6（`eye_charge` 官方名“熔合暗影”写错、`groundshadowhandler` 双端添加、`SetPrefabNameOverride` 锚点、`SGshrouden` 死亡撒块范围；`grasspartfx` 生角链顺序与渔网联网范围；`gestalt_trail` 把 `junk` 参数写成从 tracker 取）＋P3 若干已修。3 页页面／导航通过，strict 262 引用 0 错 0 警（gestalt 93／grasspartfx 68／eye 101），record 后 67 组 0 漂移；未运行游戏。

批次 I 验收（2026-11-04）：独立复审 P1×1（`moon_device_pillar` 把 `leveldirty` 引擎脏事件误判为不可用）＋P2×9（残骸页工厂总述过度概括／SetParent 级联断言／壳“不能父挂”；实验台页 mist 端、Hide 互斥范围、父挂级联、slipperyfeet 证据链；月装置页漏锤毁月坛与旧档改装生成点、漏裂纹层父挂）＋P3 若干已修。3 页页面／导航通过，strict 221 引用 0 错 0 警（remains 111／madscience 63／moon 47），record 后 42 组 0 漂移；未运行游戏。`moon_device_pillar` 压到 16,368 B（余 16 B，standard）。

### 外围逐文件结论（只读分诊批 4，2026-11-04，30 文件）

结论：125 注册（31 FX／5 混合／72 玩法／17 内部或占位）；covered-existing 1（`moon_altar_link_fx_spawner`，`moon_altar_link_fx.md` 正文专行）、excluded 88、needs-page 36、blocked 0。发现两处源码陷阱：`lunarrift_portal.lua` L728 `TheWorld.ismasersim` 拼写错误（master 分支不可达）；`lunarthrall_plant.lua` L84 误挂 `OnRemovedEntity`（高亮子实体不摘除）。

| 文件 | 注册名与结论 | 状态 |
|---|---|---|
| `prefabs/hermithouse_ornament.lua` | `hermithouse_ornament` 玩法物品；`hermithouse_ornament_fx`（FX＋Follower/Sound，CloneAsFx 注入 skin_build，隐士屋/洗衣绳/Wagstaff 三宿主复用）。 | needs-page（1 个 FX） |
| `prefabs/houndstooth_blowpipe.lua` | 物品玩法；`houndstooth_proj` 混合（weapon/planardamage/projectile＋Light），尾迹为独立非联网实体。 | needs-page（1 个混合） |
| `prefabs/hutch.lua` | `hutch` 玩法容器；`hutch_music_light_fx`（FX＋5 netvar＋本地子实体 SpawnFX，MUSIC 形态生成 3 个地面辉光）。 | needs-page（1 个 FX） |
| `prefabs/ice_puddle.lua` | `ice_puddle` 纯 FX（背景层、客户端 rock_ice 父挂）。 | needs-page（1 个 FX） |
| `prefabs/ice_splash.lua` | `ice_splash` 纯 FX（animover 自删，rock_ice 融化生成）。 | needs-page（1 个 FX） |
| `prefabs/impact.lua` | `impact` 纯 FX（联网壳＋本地代理 SetFromProxy、FinalOffset 3、0.5 秒自删，复用点≥8）。 | needs-page（1 个 FX） |
| `prefabs/junkball.lua` | `junkball_fx`（updatelooper＋影子）与 `junkball_fall_fx`（combat 结算 JUNK_FALL_DAMAGE 的混合）。 | needs-page（2 个） |
| `prefabs/king_cane.lua` | 物品玩法；`king_cane_fx`（highlightchild＋colouraddersync＋3 netvar，宝石符号覆盖/腐蚀分级）。 | needs-page（1 个 FX） |
| `prefabs/kitcoon.lua` | 9 只生物玩法＋hider_prop 非 FX；`kitcoon_hide_fx`（FX，1 秒/animover 自删）。 | excluded＋needs-page（1 个 FX） |
| `prefabs/lunar_grazer.lua` | 生物玩法；`lunar_grazer_core_fx`（Follower 跟 rock_cycle_follow）与 `lunar_grazer_debris`（DECOR＋物理、随机 5 变体）。 | excluded＋needs-page（2 个 FX） |
| `prefabs/lunarrift_portal.lua` | 结构玩法；`lunarrift_portal_shadow`（OnGroundFixed/背景层，随阶段缩放）；`ismasersim` 拼写陷阱入页。 | excluded＋needs-page（1 个 FX） |
| `prefabs/lunarrift_terraformer_explosion.lua` | `lunarrift_terraformer_explosion` 混合（FX＋combat 真实伤害、1 格对角、CANT_TAGS lunar_aligned）。 | needs-page（1 个混合） |
| `prefabs/lunarthrall_plant.lua` | 生物／vine_end 玩法；`lunarthrall_plant_back`（MiniMapEntity＋小写 fx 标签）与 `lunarthrall_plant_vine`（NOCLICK/colouradder/freezable/burnable＋SG）为表现；`OnRemovedEntity` 拼写陷阱入页。 | excluded＋needs-page（2 个） |
| `prefabs/madscience_lab.lua` | 结构／placer／6 实验占位非 FX；`_goop`、`_fire`、`_glow` 三层共用 bank 靠 Hide 分层。 | excluded＋needs-page（3 个 FX） |
| `prefabs/marsh_bush.lua` | 两 bush 玩法；`burnt_marsh_bush_erode`（联网空壳＋本地代理、bank ashes、0.5 秒自删）。 | excluded＋needs-page（1 个 FX） |
| `prefabs/miniboatlantern.lua` | 物品玩法；`miniboatlanternlight`（Light＋FX、客户端周期闪烁，随燃料生灭）。 | needs-page（1 个 FX） |
| `prefabs/mininglantern.lua` | 注册名 `lantern` 为物品玩法；`lanternlight`（Light＋FX、唤醒/睡眠声音，强度由宿主 fuelupdate 插值）。 | needs-page（1 个 FX） |
| `prefabs/minotaur.lua` | Boss／chestspawner 非本页；`minotaur_blood_big` 混合（21 帧后生成 bigshadowtentacle，按附近触手数限制）。 | excluded＋needs-page（1 个混合，建议并入 minotaur_blood1） |
| `prefabs/mistparticle.lua` | `mist` 纯 FX（VFXEffect＋emitter，gamelogic 按 Mist 节点 poly 建 area emitter）。 | needs-page（1 个 FX） |
| `prefabs/monkeyisland_portal.lua` | 结构玩法；`monkeyisland_portal_lootfollowfx`（NOBLOCK/NOCLICK、固定缩放加色、attach_light_fx 跟随）。 | excluded＋needs-page（1 个 FX，建议并入 monkeyisland_portal_fxloot） |
| `prefabs/moon_altar.lua` | 三祭坛玩法；两 marker 无动画组件、仅 map tag 记账；`moon_altar_link_fx_spawner` 已被 `moon_altar_link_fx.md` 正文专行覆盖。 | covered-existing（1）＋excluded（其余） |
| `prefabs/moon_altar_link.lua` | 结构玩法；`moon_altar_link_contained`（FX＋moon_geyser build＋_set_stage_fn，由 moon_device 生成）。 | needs-page（1 个 FX） |
| `prefabs/moon_device.lua` | 三阶段结构／placer／meteor_spawner 非 FX；`pillar`／`top`（SetPrefabNameOverride 冒充 moon_device）与 `break_stage1`（ErodeAway）为 FX。 | needs-page（3 个 FX） |
| `prefabs/moon_fissure.lua` | 两结构玩法；`moon_fissure_fx`（联网 DECOR，宿主 Show/Hide 与 transition 驱动）。 | excluded＋needs-page（1 个 FX） |
| `prefabs/multiplayer_portal.lua` | 三传送门玩法；`multiplayer_portal_moonrock_fx`（覆盖 portal_stone 符号，主模拟非洞穴生成并 SetParent）。 | excluded＋needs-page（1 个 FX） |
| `prefabs/mushtree_webbed.lua` | 玩法树；`mushtree_tall_webbed_burntfx`（FX、chop_burnt＋animover/时长兜底双自删）。 | excluded＋needs-page（1 个 FX） |
| `prefabs/nightmaregrowth.lua` | 结构／spawner／两 retrofit 数据占位；`nightmaregrowth_crack`（FX/NOCLICK/DECOR、背景层、随机旋转）。 | excluded＋needs-page（1 个 FX） |
| `prefabs/nitre_formation.lua` | `nitre_formation` 纯 FX（客户端登记 pond.highlightchildren，pond 按世界数据父挂并选 idle）。 | needs-page（1 个 FX） |
| `prefabs/nonslipgrit.lua` | 物品×2／pool／buff 非 FX；`nonslipgrit_buff_fx`（FX＋声音＋双 VFX 发射器，按父级位移调发射量）。 | excluded＋needs-page（1 个 FX） |
| `prefabs/oceanfishingbobber.lua` | `oceanfishinghook_ripple`（CLASSIFIED＋FX、非联网、客户端挂浮标下）；BOBBERS 表驱动 30 个玩法名不属本页。 | needs-page（1 个 FX） |

### 外围逐文件结论（只读分诊批 3，2026-11-04，30 文件）

结论：79 注册（35 FX／13 混合／31 玩法）；covered-existing 0、excluded 31、needs-page 42、blocked 6（熔炉事件包）、未注册内部实体 6 组；仅 30 名被他页正文提及。可合并建页 8 组（48 注册）。

| 文件 | 注册名与结论 | 状态 |
|---|---|---|
| `prefabs/charlie_boss.lua` | `charlie_boss` 为 Boss 本体；`charlie_boss_aoe_flame_fx`（fxfn 纯 FX/NOCLICK，animover 自删，生成点 minions L74／SG L108）。 | excluded＋needs-page（1 个 FX） |
| `prefabs/charlie_boss_minions.lua` | `charlie_boss_minion_hit_fx` 纯 FX；`charlie_boss_minion1/2`（MakeMinion）为带 weapon/projectile/AOE 的混合载体，FX L133。 | needs-page（3 个：1 FX＋2 混合） |
| `prefabs/charlie_boss_projectiles.lua` | `charlie_boss_projectile` 为联网混合载体（weapon/projectile）；`_reflect_projectile_fx`、`_projectile_hit_fx` 为纯 FX；内部 looper/rotator 非注册。 | needs-page（3 个：1 混合＋2 FX） |
| `prefabs/charlie_boss_runner.lua` | `charlie_boss_runner` 为玩法（spawnfader/locomotor/health/combat/brain）。 | excluded（玩法） |
| `prefabs/charliearena_spike.lua` | `charliearena_spike` 为障碍玩法；`charliearena_spike_explode_fx` 纯 Anim/Network＋FX，animover 自删。 | excluded＋needs-page（1 个 FX） |
| `prefabs/charlieresidue.lua` | `charlieresidue` 为混合（CLASSIFIED＋netvar＋inspectable/activatable，客户端 CreateFX 承担表现）。 | needs-page（1 个混合） |
| `prefabs/charlierose.lua` | `charlierose` 为混合（inventoryitem/fuel＋FX），进包转 rose、掉落碎裂音。 | needs-page（1 个混合） |
| `prefabs/chester.lua` | `chester` 为玩法（container/health/combat）；`shadow_chester_swirl_fx` 纯 Anim/Network＋CLASSIFIED 三段动画。 | excluded＋needs-page（1 个 FX） |
| `prefabs/chum_aoe.lua` | `chum_aoe` 混合（timer 定身＋周期鱼群＋FX）；`chumpiece` 混合（edible＋FX、无 AnimState）。 | needs-page（2 个混合） |
| `prefabs/crabking_claw.lua` | `crabking_claw` 为玩法；`crabking_claw_shadow`（六面朝＋LAYER_BELOW_GROUND）与 `_swipe_fx`（OnGround 背景层）为纯 FX。 | excluded＋needs-page（2 个 FX） |
| `prefabs/crabking_icewall.lua` | `crabking_icewall` 为障碍玩法（obstacle/combat/health）。 | excluded（玩法） |
| `prefabs/daywalker2.lua` | `daywalker2` 为玩法；`daywalker2_buried_fx` 纯 Anim/Follower/Network＋FX（junk 三处生成）。 | excluded＋needs-page（1 个 FX） |
| `prefabs/daywalker_pillar.lua` | `daywalker_pillar` 为玩法；`_base_fx` 与 `_hole` 为贴地装饰（后者无 FX 标签但 decor/NOCLICK）。 | excluded＋needs-page（2 个 FX） |
| `prefabs/deer.lua` | `deer`／`deer_red`／`deer_blue` 为玩法；`deer_unshackle_fx`、`deer_growantler_fx` 纯 FX、2 秒 ErodeAway。 | excluded＋needs-page（2 个 FX） |
| `prefabs/deerclops_laser.lua` | `deerclops_laser` 混合（Light＋combat＋伤害）；`laserempty` 无 AnimState 的玩法变体；`laserscorch`／`lasertrail`／`laserhit` 为 FX。 | needs-page（3 个 FX＋2 混合辨析） |
| `prefabs/deerclopseyeball_sentryward.lua` | 本体／kit／placer 为玩法；`deerclopseyeball_sentryward_fx` 混合（FX＋真实冻结/降温/催眠）。 | excluded＋needs-page（1 个混合 FX） |
| `prefabs/explosivehit.lua` | `explosivehit` 本地代理纯表现，但全 scripts 无生成点、master_postinit 取缺失事件包。 | blocked（事件包／无生成点） |
| `prefabs/eye_charge.lua` | `eye_charge` 混合（projectile 命中生成 hit）；`eye_charge_hit` 纯 FX、0.5 秒自删。 | needs-page（2 个） |
| `prefabs/fireringfx.lua` | `firering_fx` 本地代理（OnGround/背景层/bloom），主壳 3 秒自删；生成点 willow_ember／dragonfly 覆写。 | needs-page（1 个 FX） |
| `prefabs/firesuppressor.lua` | 本体／placer 玩法；`firesuppressor_glow` 纯 Anim/Follower/Network＋FX。 | excluded＋needs-page（1 个 FX） |
| `prefabs/fishingnetvisualizerfx.lua` | `fishingnetvisualizerfx` 纯 Anim/Sound/Network＋NOBLOCK/FX，animover 自删。 | needs-page（1 个 FX） |
| `prefabs/fused_shadeling_bomb.lua` | 本体玩法；`death_fx`／`scorch` 纯 FX；`quickfuse_bomb` 混合（落地即爆＋groundshadowhandler）。 | excluded＋needs-page（3 个） |
| `prefabs/gelblob.lua` | 本体／spawningground 玩法；`back_fx`／`item_fx` 纯 FX，`small_fx` 混合（吸附逻辑＋SetPrefabNameOverride）。 | excluded＋needs-page（3 个） |
| `prefabs/gestalt.lua` | `gestalt` 玩法；`gestalt_trail` 纯 FX（OnGround/bloom、40 帧自删）。 | excluded＋needs-page（1 个 FX） |
| `prefabs/gestalt_cage.lua` | cage/filled/placer 玩法；`gestalt_cage_swap_fx` 纯 Anim/Follower/Network＋FX、八面朝。 | excluded＋needs-page（1 个 FX） |
| `prefabs/grass.lua` | `grass`／`depleted_grass` 玩法；`grasspartfx` 纯 FX、animover 自删。 | excluded＋needs-page（1 个 FX） |
| `prefabs/ground_chunks_breaking.lua` | `ground_chunks_breaking` 本地代理（OnGround＋stone_drop 声），主壳 1 秒自删，7 处生成。 | needs-page（1 个 FX） |
| `prefabs/groundpoundringfx.lua` | `groundpoundring_fx` 本地代理＋net_bool fastforward，主壳 3 秒自删。 | needs-page（1 个 FX） |
| `prefabs/hammer_mjolnir.lua` | 本体玩法；`hammer_mjolnir_crackle`／`_cracklebase`／`_cracklehit`／`cracklehitfx` 均因事件包缺失且无生成点。 | excluded＋blocked（4 个 FX） |
| `prefabs/healingstaff.lua` | 本体玩法；`blossom_cast_fx` 事件包缺失、无生成点。 | excluded＋blocked（1 个 FX） |

### 外围逐文件结论（只读分诊批 1，2026-11-04，30 文件）

分诊方法：完整读取每个文件，`Prefab(` 穷举并展开 `MakePlacer`／`MakeDeployableKitItem`／`AddChair`／`makegargoyle`／`MakeMerm` 等包装与循环；在 `wiki/reference` 全量检索注册名的页名／别名／正文。结论：17 文件纯玩法（excluded）、11 文件含未覆盖 FX（needs-page）、3 文件因事件数据包缺失阻塞（blocked）、0 covered-existing。

| 文件 | 注册名与结论 | 状态 |
|---|---|---|
| `prefabs/boat.lua` | 22 名全为船体／碰撞体／物品与 placer（hull／hullhealth／boatphysics／walkableplatform／inventoryitem）；FX 如 boat_grass_erode、boat_water_fx、fx_boat_pop 在别处注册，本文件只有 SpawnPrefab 调用（L220、L797、L862）。 | excluded（玩法主体）：0 个 FX 注册 |
| `prefabs/bearger.lua` | `bearger`、`mutatedbearger` 为生物（combat／health／locomotor／eater／thief／shedder／planardamage）；熊獾挥击 FX 在 `bearger_swipe_fx` 注册。 | excluded（生物）：0 个 FX |
| `prefabs/deerclops.lua` | `deerclops`、`mutateddeerclops` 为生物；`deerclops_spikefire_fx` 仅 SpawnPrefab（L309），非本文件注册。 | excluded（生物）：0 个 FX |
| `prefabs/beequeenhive.lua` | `beequeenhive`、`beequeenhivegrown` 为蜂巢结构（workable／pointofinterest／entitytracker），成长体由 SG 与 timer 出蜂王（L139）。 | excluded（结构）：0 个 FX |
| `prefabs/lucy.lua` | `lucy` 为物品／武器（sentientaxe／possessedaxe／weapon／tool／equippable／inventoryitem）。 | excluded（物品）：0 个 FX |
| `prefabs/waterplant.lua` | `waterplant`（creature／combat／harvestable）与 `waterplant_spawner_rough`（无表现的技术生成点）不计；`waterplant_base` 仅 AnimState＋`FX` 标签＋freezable 碎裂表现（L557），唯一生成点为本体构造 SpawnPrefab 后 SetParent（L422）。 | needs-page（1 个 FX 注册，待并入宿主／族页） |
| `prefabs/wormwood_vined_debuff.lua` | `wormwood_vined_debuff` 为 debuff＋rooted＋每跳真实伤害（L40），机制由 `components/rooted` 页承载。 | excluded（debuff）：0 个 FX |
| `prefabs/atrium_gate.lua` | `atrium_gate` 为仪式控制器（trader／pickable／playerprox／charliecutscene）；脉冲／爆裂 FX 在别处注册。 | excluded（控制器）：0 个 FX |
| `prefabs/balatro_machine.lua` | `balatro_machine` 为小游戏机器（activatable／timer／lootdropper／talker）。 | excluded（玩法机器）：0 个 FX |
| `prefabs/archive_chandelier.lua` | `archive_chandelier`、`vault_chandelier`、`vault_crawler_chandelier` 同一 `MakeChandelier` 工厂：Transform＋AnimState＋Light＋`FX`／`NOCLICK`，仅 archive 档挂 playerprox 20/23 做灯光开关；vault_crawler_chandelier 兼作 vault_crawler 生成点（L486）。 | needs-page（3 个 FX 注册） |
| `prefabs/fence.lua` | 8 名玩法（墙／门／junk 墙／物品／placer）；`fence_gate_anim`、`quagmire_park_gate_anim` 为 `MakeWallAnim`（L763–810）的纯 AnimState＋`FX` 表现，由门本体依赖生成。 | needs-page（2 个 FX 注册） |
| `prefabs/furniture_chairs.lua` | 13 名玩法（sittable／workable／物品／placer）；4 个 `_back` 椅背层（`backfn` L195–229）只有 AnimState＋`FX`，SetParent 于椅并进 highlightchildren。 | needs-page（4 个 FX 注册） |
| `prefabs/gargoyles.lua` | 5 个石像为玩法（workable MINE／lootdropper／可复活）；5 个 `_fx`（`makefx` L278）为 `FX` 标签代理＋本地动画、1 秒自删，生成点在 Petrify／Reanimate 回调。 | needs-page（5 个 FX 注册） |
| `prefabs/grotto_pool_big.lua` | `grotto_pool_big` 为水源结构（watersource／inspectable）。 | excluded（结构）：0 个 FX |
| `prefabs/grotto_pool_small.lua` | `grotto_pool_small` 为水源结构（watersource／inspectable／birdblocker）。 | excluded（结构）：0 个 FX |
| `prefabs/hedgehound_bush.lua` | `hedgehound_bush` 为可激活灌木（health／combat／activatable），激活 SpawnPrefab("hedgehound")（L65）。 | excluded（玩法）：0 个 FX |
| `prefabs/hermitcrab_teashop.lua` | `hermitcrab_teashop`／`_placer` 为玩法（craftingstation／workable／playerprox／burnable）；`hermitcrab_teashop_front`（L449–482）只有 AnimState＋`FX`、`SetFinalOffset(3)`，主模拟 L373 生成。 | needs-page（1 个 FX 注册） |
| `prefabs/hermithotspring.lua` | 3 名玩法（bathingpool／bathbombable／heater／watersource＋施工点）。 | excluded（结构）：0 个 FX |
| `prefabs/horrorfuel.lua` | `horrorfuel` 为施法材料物品（fuel／spellcaster／socketable）；施法 FX 在 `wurt_casting_fx` 页。 | excluded（物品）：0 个 FX |
| `prefabs/inspectaclesbox.lua` | `inspectaclesbox`／`2` 为奖励箱（activatable／lootdropper／projectedeffects）。 | excluded（玩法）：0 个 FX |
| `prefabs/itemmimic_revealed.lua` | `itemmimic_revealed` 为生物；`itemmimic_revealed_shadow`（L227–285）为纯 VFXEffect＋EmitterManager、非联网、persists=false、AddChild 于本体。 | needs-page（1 个 FX 注册） |
| `prefabs/junk_pile_big.lua` | `junk_pile_big` 为玩法（workable／timer／lootdropper）；`junk_pile_side`（视觉侧片）与 `junk_pile_blueprint`（Follower＋`FX` 循环动画）无组件、persists=false、SetParent 于垃圾堆。 | needs-page（2 个 FX 注册） |
| `prefabs/lantern_posts.lua` | 5 名玩法（post：workable／container／preserver；item／placer）；两组 `*_light_chain`（`lightchainfn` L830–904）只有 Light＋`FX`＋updatelooper／entitytracker，由 post 的 lightchainprefab 依赖生成。 | needs-page（2 个 FX 注册） |
| `prefabs/lavaarena_beetletaur.lua` | `beetletaur` 为玩法（master_postinit 在事件数据包）；4 个 `beetletaur_fossilized_break_fx_*`（L265–303）只 Transform＋AnimState＋Network＋`FX`、animover 消散，工厂自足。 | needs-page（4 个 FX 注册） |
| `prefabs/lavaarena_blooms.lua` | 7 个治疗花视觉注册可核验，但 bloom_postinit／heal／sleep／buff 构造全在缺失的 `event_server_data("lavaarena",…)`；无法确认是否挂玩法组件。 | blocked（事件数据包缺口）：7 个候选待定 |
| `prefabs/lavaarena_crowdstand.lua` | stand_postinit／spectator_postinit 在缺失事件数据包；就地仅见本地 DECOR 子件；`lavaarena_groundtargetblocker` 无视觉。 | blocked（事件数据包缺口）：3 个候选待定 |
| `prefabs/lavaarena_rhinodrill.lua` | `rhinodrill`／`rhinodrill2` 为玩法（master_postinit 在数据包）；3 个 `rhinodrill_fossilized_break_fx*`（L194–228）为纯 `FX` 表现。 | needs-page（3 个 FX 注册） |
| `prefabs/lavaarena_spawner.lua` | `lavaarena_spawner` 的 master_postinit 在缺失事件数据包；就地只有 Transform＋AnimState＋Network 与未注册装饰齿。 | blocked（事件数据包缺口）：1 个候选待定 |
| `prefabs/lightsout_trial.lua` | `lightsout_trial` 为解谜控制器（vaulttorchgrid／abysspillargroup），驱动 shadowhand／abysspillar。 | excluded（控制器）：0 个 FX |
| `prefabs/merm.lua` | 6 名均为鱼人生物（combat／health／locomotor／follower／trader）；相关 FX 在 `lunarmerm_thorns_fx`、`merm_splash` 页。 | excluded（生物）：0 个 FX |

外围批 A：3 文件、3 页、8 个 FX 注册；独立复审修复酸雨遮挡与隐藏时序、深渊柱加载组条件及花特效生成失败边界。6 文件页面与导航通过、strict 101 引用 0 错 0 警（另跳过 1 个不支持链接）、3 页 record 后 17 组 0 漂移；无 Lua 示例、未运行游戏。新增结论不代表其余外围文件已完成。

外围批 B：3 文件、3 页、12 个 FX 注册；独立复审修正水声发射器旧池有效门禁。7 文件页面与导航通过、strict 116 引用 0 错 0 警（另跳过 1 个不支持链接）、3 页 record 后 24 组 0 漂移；无 Lua 示例、未运行游戏。

### 阶段交接（2026-11-05 待建清单复核）

本段与下方两张页组表维护当前续跑状态；批次验收段提供已完成记录，分诊表与旧交接数字仅作历史依据。

- 当前 51 页组全部落地，并有 U1～U17 验收记录；实际对应 50 个新页与 1 个增修页（组 18 对应两页，组 36／37 合并到 burntground，U15/U16/U17 各新增 3 个新页）。本次核对页面存在、frontmatter 归属与验收批次，不重做全文技术复审。
- 待建 **0 组、0 个计划注册**；`yotr_fightring_kit_placer` 已随组 51 落地。
- **收尾（2026-11-05）**：U15～U17 九页全部验收，51 页组全部 accepted；全库 `validate_page --check-indexes` 1146 页 0 failed、导航 0 问题，strict links 29,309 引用 0 error／27 条存量 warning（12 个既有页面，strict 未通过），source hashes 7,415 组比较 0 漂移。外部独立检视 P1×2／P2×7 已全部采纳并复核。最终报告见 [fx-final-report-20261105.md](fx-final-report-20261105.md)。spellmasterybuff（组 11，3 注册）曾误标 U4，已于 U17 建页；旧交接“43 组完成／剩 8 组 16 注册”作废。
- 候选分诊基数仍为 380＝187 既有闭环＋101 needs-page＋26 blocked＋66 excluded；101 是 U 批建页前的候选文件池，不是当前剩余。旧对账脚本不消费数字页组表或 U 批验收段，不能单独用其 remaining 判定待建。当前 51 页组已全部验收、无待建；本节只保留分诊基数。旧交接“110 注册”未经逐名累计复核，不沿用为完成总数。

以下四条保留 U 批建页前的分诊基数与历史处理背景，不作为当前剩余清单：

- 通用表 504／504 已覆盖并逐名对账，详见下方全量统一检查进度。外围 380 个候选文件中，**187 个已闭环**（按台账表体首次 accepted 去重归属：首表 16＝既有工具 4＋批 A 3＋批 B 3＋批 C 6；批 D 19／E 30／F 10／G 13／H 11／I 12／L 17／M 12／N 8／O 9／P 5／Q 3／R 7／S 8／T 5＝169；分诊节 2＝`moon_altar.lua`（唯一 FX 注册已由既有页承载、本体 excluded）与 `slingshotaoefx.lua`（3 名已有页）；L/M/P 的批表头 22/15/8 为旧误、表体实为 17/12/5 个新增去重文件），承载 **388 个外围 FX 注册**（既有页 29＋批 C 17＋批 D 64＋批 E 57＋批 F 27＋批 G 22＋批 H 15＋批 I 16＋批 L 29＋批 M 18＋批 N 9＋批 O 9＋批 P 20＋批 Q 6＋批 R 11＋批 S 10＋批 T 29；注册数按各批验收和，文件数按表体去重，两者口径不同）。另有 2 个表体 covered-existing 行不在候选清单内（`opalstaff.lua`→`staff.lua`、`shadowthrall_projectile_fx.lua` 已有专属页），不计入 380 口径。**剩余 193 个未闭环**（2026-11-05 用 `output/tmp/fx_remaining_classify.py` 按台账状态重算）：**88 个 needs-page**（含批 6 表述为"excluded＋FX 子注册待合并"的 10 文件：floatinglantern、fossil_spike、fossil_spike2、oceanshadowcreature、propsign、pumpkin_bomb、purebrilliance、rift_terraformer、rocky_boss、rope_bridge）、**11 个 blocked**（bishop_charge、blowdart_lava、blowdart_lava2、explosivehit、fireball_projectile、fireballstaff、hammer_mjolnir、healingstaff、lavaarena_blooms、lavaarena_crowdstand、lavaarena_spawner）、**40 个 excluded**、**54 个未提及候选已于 2026-11-05 全部分诊**（三节逐文件实核：blocked 15／excluded 26／needs-page 13 文件，见下方"未分诊 54 候选之实核"三节）。needs-page 文件总数相应从 88 增至 **101 文件**（+8 文件 17 注册见组 41-48、+3 文件 5 注册见组 49、+2 文件 5 注册见组 50/51），页组 40→**51**。旧阶段交接的"139 个已有结论"未拆分 needs-page／blocked／excluded，"约 20 组合并页"只覆盖批 3/6/8/9 代表组、漏掉批 1/2/7/10 的 needs-page（约 30 文件），均以本清单与本节下方建页清单为准勘误。复核脚本：`output/tmp/fx_closure_audit.py`（只读，可重复）、`output/tmp/fx_remaining_classify.py`（状态分类）。旧阶段交接的"191 未闭环／394 注册"与批次表头多处文件数以此为准勘误。
- 批 C～R 各页均已独立复审、修复、验收并 record；批 S 三页验收见外围批 S 验收段（本表旧行"生成（待独立复审）"为笔误，已改）；批 T 三页已于 2026-11-04 独立复审（P1×0、P2×0）验收并 record（206 引用 0 错警、check 0 漂移）。历史 claimed 不再代表占用。
- `fx-index.md` 已按主题拆为 `fx-index`（皮肤装备／角色技能／战斗反馈／杂项，8.4 KiB）与 `fx-world-index`（生物与Boss／环境与世界，8.0 KiB），父索引两行登记；后续页面按类别登记到对应叶级索引。
- 工作区改动均保留且未提交、未推送；先实读 git status、HEAD、BUILD.txt、PLAN 和本台账再续作。不要把下方历史批次检查点中「待逐名对账」或「剩 144 名」当作当前进度。

### 外围 needs-page 建页清单（2026-11-05 对账定稿）

最初 88 个 needs-page 文件形成组 1～40，后续实核增补组 41～51，共涉及 101 个候选文件。页名以实际主注册名落名；accepted 表示已有页面与对应批次验收记录，needs-page 后的批次仅为计划，不代表认领或完成。登记与验收完成后再改 accepted。

| 组 | 页（拟名） | 文件 | 注册（约） | 验收状态／批次 |
|---|---|---|---:|---|
| 1 | oceanhorror_ripples | oceanshadowcreature.lua | 2 | accepted（U1） |
| 2 | rocky_snip_fx | rocky_boss.lua＋worm_boss.lua | 4 | accepted（U1） |
| 3 | rope_bridge_fx | rope_bridge.lua＋vine_bridge.lua | 3 | accepted（U1） |
| 4 | fossilspike_base | fossil_spike.lua＋fossil_spike2.lua | 2 | accepted（U2） |
| 5 | lunarrift_terraformer_visual | rift_terraformer.lua＋floatinglantern.lua | 2 | accepted（U2） |
| 6 | shatter | shatter.lua＋splash_spiderweb.lua＋singingshell.lua＋propsign.lua＋pumpkin_bomb.lua＋tumbleweed.lua＋brokentool.lua＋bomb_lunarplant.lua | 8 | accepted（U2） |
| 7 | terrariumchest_fx | treasurechest.lua＋townportal.lua | 2 | accepted（U4） |
| 8 | cavein_dust_low | cavein_boulder.lua＋daywalker2_items.lua | 9 | accepted（U3） |
| 9 | slingshotammo_purebrilliance_debuff_fx | slingshotammo_debuffs.lua | 3 | accepted（U3） |
| 10 | spear_gungnir_lungefx | spear_gungnir.lua＋spear_wathgrithr.lua | 3 | accepted（U3） |
| 11 | spellmasterybuff | spellmasterybuff.lua | 3 | accepted（U17；旧 U4 误标已纠正） |
| 12 | winona_battery_high_shatterfx | winona_battery_high.lua＋winona_battery_sparks.lua＋winona_spotlight.lua＋winona_telebrella.lua＋purebrilliance.lua | 5 | accepted（U4） |
| 13 | forcefieldfx | forcefieldfx.lua | 1 | accepted（U4） |
| 14 | wx78_big_spark | wx78_big_spark.lua＋wx78_lightbeam.lua＋wx78_scanner.lua＋wx78_backupbody.lua | 4 | accepted（U5） |
| 15 | wagboss_beam_fx | wagboss_beam.lua＋wagboss_robot_leg.lua＋wagstaff_npc.lua | 4 | accepted（U5） |
| 16 | wortox_soulecho_buff_fx | wortox.lua＋wortox_nabbag.lua＋wortox_reviver.lua | 6 | accepted（U5） |
| 17 | wurt_shadow_merm_planar_fx | wurt.lua＋wurt_terraform_item.lua | 5 | accepted（U6） |
| 18 | waterplant_pollen_fx＋watertree_pillar 并入海树柱页 | waterplant_pollen.lua＋watertree_pillar.lua | 4 | accepted（U6） |
| 19 | wendy_gravestone_rebirth_fx | wendy_resurrectiongrave.lua | 2 | accepted（U8） |
| 20 | trap_fumarole_burn_fx | trap_fumarole.lua＋trap_vines.lua | 2 | accepted（U7） |
| 21 | boatrace_start_flag | boatrace_start.lua＋boatrace_checkpoint.lua | 4 | accepted（U7） |
| 22 | shadow_bishop_fx | shadowchesspieces.lua＋shadowrift_portal.lua＋shadowthrall_mouth.lua＋shadowthrall_parasite.lua | 4 | accepted（U7） |
| 23 | voidcloth_boomerang_fx | voidcloth_boomerang.lua＋voidcloth_scythe.lua＋voidcloth_umbrella.lua | 5 | accepted（U8） |
| 24 | vault_compass_visual | vault_compass.lua＋vault_decon_mister.lua＋vault_ground_pattern.lua＋vault_torch.lua | 4 | accepted（U8） |
| 25 | hats（12 注册，含 tophat_container 技术实体） | hats.lua | 12 | accepted（U9） |
| 26 | mushtree_tall_burntfx（动态拼名 6＋moon 1） | mushtree.lua＋mushtree_moon.lua | 7 | accepted（U9） |
| 27 | moon_altar_break | moon_altar_break.lua | 3 | accepted（U9） |
| 28 | attackfx_handpillow_petals（pillow 4＋plant_normal 3） | pillow_equipment.lua＋plant_normal.lua | 7 | accepted（U10） |
| 29 | bramblefx_armor | bramblefx.lua | 3 | accepted（U10） |
| 30 | archive_ambient_sfx | archive_props.lua＋archive_resonator.lua | 5 | accepted（U11） |
| 31 | atrium_gate_activatedfx | atrium_gate_activatedfx.lua＋atrium_gate_pulsesfx.lua | 4 | accepted（U11） |
| 32 | balloon_held_child | balloon_held_child.lua＋balloonparty.lua | 4 | accepted（U11） |
| 33 | bat_bosshat_fx | bat_boss.lua＋batbat.lua＋battreefx.lua | 5 | accepted（U13） |
| 34 | beargerfur_sack_frost_fx | beargerfur_sack.lua＋bernie_big.lua | 2 | accepted（U12） |
| 35 | bishop_targeting_fx | bishop.lua | 1 | accepted（U13） |
| 36 | burntground | burntground.lua | 2 | accepted（U10） |
| 37 | honey_trail（并入 burntground） | honey_trail.lua | 2 | accepted（U10） |
| 38 | itemmimic_revealed_shadow | itemmimic_revealed.lua | 1 | accepted（U13） |
| 39 | wintersfeastbuff_fx | wintersfeastbuff.lua＋wintersfeastoven.lua | 2 | accepted（U12） |
| 40 | witchbroom_fx | witchbroom.lua＋woby_rack.lua＋wolfgang.lua | 3 | accepted（U12） |

注：本表按页组而非页面数计数；注册数沿用已验收范围或待建规格，不作纯 FX 数／全库净新增总数。混合载体与技术实体在页内辨析；事件包缺口（spellmasterybuff、spear_gungnir_lungefx）按实际核验范围保留，不能把计划编号当作已建页。

### 外围批 U 验收检查点

**批 U1（2026-11-05）**：3 页 9 注册／9 文件——海上暗影附层 `oceanhorror_ripples`（2 注册：宿主 SG 驱动缩放涟漪＋平台跟随挂点，专用服务器 alpha 恒 0.5）、Boss 战表现 `rocky_snip_fx`（4 注册：SGrocky 帧生成的剪击弧与冲刺残影无自删靠 onexit、枯萎变体经 `commonstates` `inst.prefab.."_shadow"` 动态拼名生成——全库唯一生成方式、蠕虫地面裂纹宿主 `new_crack` 单槽管理）、桥面表现 `rope_bridge_fx`（3 注册：管理器按瓦片生成、绳桥双绳客户端本地重建 `iconffset` 源码拼写陷阱、藤桥 18 秒排定拆除与装饰双型 `inst.decor` 表）。独立复审（主会话逐锚点）：P1×0、P2×0、P3×1（rocky 页土丘 build 锚点差一跳，已修 L1654→L1654-L1661）；三页 validate_page --check-indexes 通过、strict 52/69/54 引用 0 错 0 警、record 32 组 0 漂移、索引登记 fx-creature-index（2 行）与 fx-world-index（1 行）。未运行游戏、未提交。

**批 U2（2026-11-05）**：3 页 12 注册／12 文件——化石基座 `fossilspike_base`（2 注册：两代尖刺引擎父挂基座、无自删随宿主、两代收尾不对称 base_pst vs base_impact）、裂隙视觉与河灯影层 `lunarrift_terraformer_visual`（2 注册：隐形管理器的 23 秒阶段演出层——earlyexit 门控吞后续阶段、天灯 SHADOW_ANIMS 表成对动画同步）、碎裂爆裂族 `shatter`（complex，8 注册／8 文件：shatter 等级壳＋本地动画双端代理与 freezable 11 处消费链、splash_spiderweb 无生成点诚实缺口、pumpkin 确定性 PRNG 碎块子系统、brokentool 无专服门控＋5 生成点、tumbleweedbreakfx 动画长度兜底；清单外发现 singingshell_creature_woodfx/rockfx 注册于 fx.lua 且已由既有页覆盖，页内辨析）。独立复审（主会话逐锚点核 8 工厂全文＋生成点＋PRNG/门控/兜底声明）：P1×0、P2×0、P3×2（shatter 页两处 po 键名笔误 EXLODE→EXPLODE，已修）；三页 validate_page --check-indexes 通过、strict 0 错 0 警、record 51 组 0 漂移、索引 fx-world-index 3 行。未运行游戏、未提交。台账组 6 注册数勘误：9→8（第 9 名 singingshell_creature_woodfx/rockfx 已由 messagebottle_break_fx 页承载）。

**批 U3（2026-11-05）**：3 页 15 注册／5 文件——落石尘与掉落物碎裂 `cavein_dust_low`（complex，9 注册：cavein 高低两档共用 MakeFX＋成阵/砸地触发链、daywalker2 七个掉落物碎裂动态拼名无 nil 保护、spike_loot_fx 无 persists＋OnLoad=OnAnimOver 读档即结算怪癖、scraphat 全库无注册行的诚实边界）、弹弓减益 `slingshotammo_purebrilliance_debuff_fx`（3 注册＋第 4 注册 mark 载体辨析：horrorfuel DoT 伤害由 FX 实体 GetAttacked 结算的混合载体、purebrilliance front/back 双层）、长矛突刺 `spear_gungnir_lungefx`（3 注册：gungnir 事件包缺口复核成立、奔雷矛八向突刺与装备跟随层换绑）。清单外并入：cavein_debris（fx.lua 通用表）与 warningshadow 简条。独立复审（主会话逐锚点核 MakeFX/MakeItemBreakFx/PRNG 外三工厂全文＋生成点＋po 六组官方名）：P1×0、P2×0、P3×0，零发现；三页 validate_page --check-indexes 通过、strict 0 错 0 警、record 25 组 0 漂移、索引 fx-world-index（1 行）与 fx-index（2 行）。未运行游戏、未提交。

**批 U4（2026-11-05）**：3 页 8 注册／10 文件——宝箱沙棺 `terrariumchest_fx`（2 注册：恐怖之眼门禁的宝箱双层微光＋SHIMMER 状态标记语义、传送沙柱 ObstaclePhysics 唯一带碰撞 FX 与 KillFX＋35 帧兜底）、薇诺娜设备 `winona_battery_high_shatterfx`（5 注册：碎宝石 LoseGem 链、purebrilliance_symbol_fx 槽位跟随 `_gemsymfollowers` 管理、sparks 8 处生成不走代理、spotlight_head 无显式 Remove、telebrella 两层帧区间分工）、图勒皇冠 `forcefieldfx`（1 注册：proc 链 tryproc→proc→unproc、netvar 灯光协议各端本地插值、未置 persists 的存档语义辨析）。独立复审（主会话逐锚点核三工厂全文＋宿主链＋po）：P1×0、P2×0、P3×0，零发现；三页 validate_page --check-indexes 通过、strict 0 错 0 警、record 31 组 0 漂移、索引 fx-world-index（1 行）与 fx-index（2 行）。未运行游戏、未提交。

**批 U5（2026-11-05）**：3 页 14 注册／8 文件——WX78 `wx78_big_spark`（4 注册：代理无 AnimState 闪白不可达复核成立、照明电路灯点链带客户端预测、扫描锥 AddChild、备份底盘渲染层混合辨析）、WAGboss `wagboss_beam_fx`（4 注册：轨道光束月灼混合载体＋`inst.caster` 不可达、机械腿 FX 勘误为超新星灼烧阻挡标记、瓦匠剪影（联网说话体＋本地剪影子体）、`alterguardian_contained` 混合装置）、沃拓克斯 `wortox_soulecho_buff_fx`（6 注册：灵魂回响加速层、过载警告、强抢袋三层、双尾心跟随体；panflute buff fx 死代码辨析）。**台账勘误 3 条**（批 9 结论）：`wagstaff_alterguardian_contained`→实际注册名 `alterguardian_contained`；`wagboss_robot_leg_fx`"落地喷雾"→实为超新星灼烧阻挡标记（落地只有 foot_land 音效）；组 16 注册数 5→6（nabbag 三层 body/body_fx/body_soulfx）。独立复审（主会话逐锚点核勘误三处＋生成点＋死代码＋po）：P1×0、P2×0、P3×0，零发现；三页 validate_page --check-indexes 通过、strict 0 错 0 警、record 30 组 0 漂移、索引 fx-index（2 行）与 fx-creature-index（1 行）。未运行游戏、未提交。

**批 U6（2026-11-05）**：2 新页＋1 增修，9 注册／4 文件——沃特 `wurt_shadow_merm_planar_fx`（5 注册：台账 3 名＋清单外两个 chargedfx 变体；暗影持续层／月影一次性动画、投泥弹壳改泥代理辨析、`OnExtendedBuff` 引用未定义 `OnKillBuff` 的死代码发现）、海草花粉 `waterplant_pollen_fx`（2 注册：诱饵云为混合载体——chum 标签引鱼＋1 秒周期投喂 chumpiece 上限 5）、海树柱页增修 `oceantree_ripples_short`（并入 `watertree_pillar_ripples`／`_roots` 2 注册，注册数 10→12，验证记录追加 2026-11-05 增补段）。**台账勘误 2 条**（批 9 结论）：chargedfx 非"同工厂仅 build 不同"——实为同工厂同 bank/build 仅动画名与月影泛光不同；watertree 两 FX 与柱体两件"同 bank"→实为"同名同动画、不同 bank"（`oceantree_pillar` vs `_small`）。独立复审（主会话逐锚点核两新页全文＋增修页逐行 diff＋po 五键）：P1×0、P2×0、P3×0，零发现；三页 validate_page --check-indexes 通过、strict 0 错 0 警、record 37 组 0 漂移（含增修页基线刷新）、索引 fx-index（1 行）与 fx-world-index（1 行）。未运行游戏、未提交。

**批 U7（2026-11-05）**：3 页 10 注册／8 文件——陷阱 `trap_fumarole_burn_fx`（2 注册：burn FX 为 DECOR 标签异例无自删、藤蔓底座复用影柱 bank）、龙舟竞赛 `boatrace_start_flag`（4 注册：台账 3 纯 FX＋bobber 按实归为联网标记混合载体；旗/烟花无 SetPristine 分界异例、浮标 dockjammer 禁修码头）、暗影散件 `shadow_bishop_fx`（4 注册：主教羽群、裂隙门粒子无自删、墨荒替身带真实物理、寄生面具小写 fx 标签异例）。**台账批 7 勘误 1 条**：`shadow_bishop_fx` 非"feather_loop 一帧后 animover"——实为播完一轮动画后 animover 自删（工厂无循环参数），羽群复现靠 SG 周期任务。独立复审（主会话逐锚点核三页生成点/标签异例/无 SetPristine 反例/物理段＋po）：P1×0、P2×0、P3×0，零发现；三页 validate_page --check-indexes 通过、strict 0 错 0 警、record 3 页、索引 fx-world-index（2 行）与 fx-creature-index（1 行）。未运行游戏、未提交。

**批 U8（2026-11-05）**：3 页 11 注册／7 文件——虚布装备 `voidcloth_boomerang_fx`（5 注册：回旋镖持握刀身由 FX 层渲染的新发现、proj 混合载体伤害在弹体、umbrella 内部 wave/dome 未注册复核属实）、宝库设施 `vault_compass_visual`（4 注册：罗盘三层链、地面纹样未置 persists 反例、火炬冷火三档无 heater）、温蒂墓碑 `wendy_gravestone_rebirth_fx`（2 注册：同一演出双层、仅温蒂触发、无 SetPristine 全端对称）。**台账勘误 2 条**（批 8/9）：`vault_ground_pattern_fx` 生成点"15 处"为 vaultroom_defs 单文件计数、全域实为 18 处；`voidcloth_boomerang_launch_fx`／`_impact_fx` 已有归属页（voidcloth_boomerang_launch_fx.md），批 11 建页时未在台账补记归属、此处补记。独立复审（主会话逐锚点核 OnEquip 新发现/18 处计数/SGwilson 相邻双生成/entry 条件分支＋po 五键）：P1×0、P2×0、P3×0，零发现；三页 validate_page --check-indexes 通过、strict 0 错 0 警、record 33 组 0 漂移、索引 fx-index（2 行）与 fx-world-index（1 行）。未运行游戏、未提交。

**批 U9（2026-11-05）**：3 页 22 注册／3 文件——帽子族 `hats.md`（complex，12 注册：9 个 MakeFollowFx 跟随层＋2 盏帽灯＋1 技术实体 tophat_container；页名按源文件落名，为外围族首例）、蘑菇树焦壳 `mushtree_tall_burntfx`（7 注册：treeset 动态拼名全展开）、月坛破碎 `moon_altar_break`（3 注册：9.5 秒破碎链 ClearArea 唯一生成、锤毁不生成、cosmic→crown/claw→astral 动画库名错位辨析）。**台账批 10 勘误 1 条**：`mask_halfwit_fx` 在 hats.lua 内零生成（仅注册与 deps 声明），唯一生成点在舞台剧剧本 `play_commonfn.lua:68`。独立复审（主会话逐锚点核注册块/动态拼名/映射表/舞台剧生成点＋po）：P1×0、P2×0、P3×0，零发现；三页 validate_page --check-indexes 通过、strict 0 错 0 警、record 3 页、索引 fx-index（1 行）与 fx-world-index（2 行）。未运行游戏、未提交。

**批 U10（2026-11-05）**：3 页 14 注册／5 文件——枕头碎屑与挖起 `attackfx_handpillow_petals`（complex，7 注册：四材质动态拼名＋三档挖起尘；plant_dug_small_fx 两处清单外拖尾生成；页名勘误：组 28 占位名 pillowfight_break_fx 非注册名，按实际主注册落名）、荆棘尖刺 `bramblefx_armor`（3 注册：混合载体伤害两路归属、armor_upgrade 仅 husk 荆棘茄甲接线的新发现）、焦地蜂蜜 `burntground`（4 注册：64 tick 渐隐、persists 未关闭反例、蜂蜜减速在 FX 实体逐帧推）。台账批 10 结论（64 tick×5 天、每 tick 减速）核实无误。独立复审（主会话逐锚点核注册行/动态拼名/两路伤害/husk-only 接线＋po）：P1×0、P2×0、P3×0，零发现；三页 validate_page --check-indexes 通过、strict 0 错 0 警、record 3 页、索引 fx-world-index（2 行）与 fx-index（1 行）。未运行游戏、未提交。

**批 U11（2026-11-05）**：3 页 13 注册／7 文件——档案机关 `archive_ambient_sfx`（5 注册：`arhivepoweron` 源码拼写陷阱、switch_pad 无移除路径、resonator_base 持久 timer）、远古大门 `atrium_gate_activatedfx`（4 注册：activatedfx 两态切换、makesfx 同构三兄弟、地区门控辨析；批 2"atrium_gate.lua:162 生成"复核属实）、气球 `balloon_held_child`（4 注册：代理＋客户对象两层、`ballon_held_child` 通道拼写陷阱、confetti_cloud 逐秒 AddDebuff 混合＋未置 persists 反例）。独立复审（主会话逐锚点核三处拼写陷阱/生成行/debuff 链）：P1×0、P2×0、P3×0，零发现。**修复主代理 U5/U7 引入的索引登记错位**：fx-creature-index 引言段被 reg.replace 首中导致 wagboss_beam_fx 与 shadow_bishop_fx 两行游离表外，已移回表格末尾、引言还原，两页 --check-indexes 复验通过。三页 validate_page --check-indexes 通过、strict 0 错 0 警、record 3 页、索引 fx-world-index（3 行）。未运行游戏、未提交。

**批 U12（2026-11-05）**：3 页 7 注册／6 文件——冬宴 `wintersfeastbuff_fx`（2 注册：buff 周期投放方向勘误入页、烤炉火层宿主调度）、散件组 `witchbroom_fx`（3 注册：隐形壳＋本地池化轨迹、沃比驮架双层、教练 buff 纯表现；**台账批 9 勘误 1 条**：轨迹池非"10Hz"——`DoPeriodicTask(FRAMES*5)`≈6 次/秒且有移动门控；woby_rack 幽灵名 blocked 判定复核属实）、保暖伙伴 `beargerfur_sack_frost_fx`（2 注册：霜气复用 icebox_open_fx 全库唯一使用者、伯尼火层反射伤害在宿主）。独立复审（主会话逐锚点核节拍/幽灵注册/生成行/反射归属＋po）：P1×0、P2×0、P3×0，零发现；三页 validate_page --check-indexes 通过、strict 0 错 0 警、record 3 页、索引 fx-index（2 行）与 fx-creature-index（1 行）。未运行游戏、未提交。

**批 U13（2026-11-05）**：3 页 7 注册／5 文件——蝙蝠系 `bat_bosshat_fx`（5 注册：DECOR 标签异例、栖立小蝠父挂 owner、batbat 皮肤代理＋本地双实体、battree SetClassifiedTarget 定向可见）、主教 `bishop_targeting_fx`（1 注册：dist netvar 双端光斑、专服也要光的源码注释）、拟态蠕虫 `itemmimic_revealed_shadow`（1 注册：VFXEffect 恒定 1 粒、共享段生成、net_event 开关）。台账批 1/2 结论复核属实无勘误。独立复审（主会话逐锚点核注册行/生成行/classified/net_event）：P1×0、P2×0、P3×0，零发现；三页 validate_page --check-indexes 通过、strict 0 错 0 警、record 3 页、索引 fx-creature-index（3 行）。未运行游戏、未提交。

**批 U14（2026-11-05）**：3 页 10 注册／4 文件——月亮石 `positronbeam_back`（3 注册：满月充能链、StopFX 笔误陷阱）、织影者盾 `stalker_shield1`（5 注册：PickShield 分布、基名诚实缺口）、毒菌帽 `toadstool_cap_absorbfx`（2 注册：中毒金丝雀触发链）。**勘误 2 条**：21 文件分诊节"stalker_shield 10 点伤害经 StartRepel 代理"→实核为 FX 实体直接作 `GetAttacked` 的 attacker（L65）；毒菌帽两 FX 触发源非 Boss 状态机——实为中毒金丝雀 SG 爆毒 `poisonburst`＋帽子自身状态机（建页任务规格原描述有误，已按源码改正）。清单外注记：fx.lua 注册的 shadow_shield1-6（armor_skeleton 生成，已有 shadow_shield1.md 承载）。独立复审（主会话逐锚点核三处注册/生成/GetAttacked 归属＋po）：P1×0、P2×0、P3×0，零发现；三页 validate_page --check-indexes 通过、strict 0 错 0 警、record 3 页、索引 fx-world-index（1 行）与 fx-creature-index（2 行）。未运行游戏、未提交。

**批 U17（2026-11-05）**：3 页 8 注册＋1 placer／5 文件——WX-78 `wx78_taser_projectile_fx`（1 注册：常驻电壳、三档闪白、脱父挂爆发；attacker 玩家／FX 双分支，电免疫取 1 倍；实际伤害来自 `Explode` 参数而非 combat 默认值）、枕头大战 `yotr_fightring_torch`（4 注册＋placer：擂台、套装、钟与 8 支事件驱动火炬；`gosmall` 后 `turnon` 不恢复大火）、法术精通 `spellmasterybuff`（3 注册：buff 锚点、法球载体与单颗法球；buff／orbs 主模拟初始化在缺失的 lavaarena 事件包，`spellmasteryorb` 为唯一不依赖事件包的可见注册）。独立复审（主会话逐锚点核 8 注册＋placer／5 文件＋po）：P1×0、P2×1（yotr 火炬点亮时序"后半程"表述收窄）、P3×1（WX-78 隐身壳阶段表述）已修，另校正 8 处引用边界；三页 validate_page --check-indexes 通过、strict 0 错 0 警、record 3 页 check 0 漂移；索引 fx-index（2 行）与 fx-world-index（1 行）。外部独立检视补 P1×1（WX-78 模块数覆盖 50×n 与 2+n）已修，修订后 strict／record 复验通过。未运行游戏、未提交。

**批 U16（2026-11-05）**：3 页 7 注册／6 文件——小惊吓 `hotcold_fx`（1 注册：距离→缩放、任务完成清理而放弃路径不清理、无 `FX` 标签）、盒中泰拉 `terrarium_fx`（1 注册：夜间 5 秒延迟→激活／循环／10 帧退场、读档重建、`DECOR` 无 `FX` 标签）、WAGboss 群 `wagboss_missile_target_fx`（5 注册：导弹落点环为客户端本地实体；黄莺／笼墙电束分段扫描＋1 秒冷却；黄莺／电刑机电球混合弹体，18 帧窗口）；`wagpunk_cagewall_fx` 全快照无生成点，保持诚实缺口。独立复审（主会话逐锚点核 5 注册／6 文件＋po）：P1×0、P2×3（小惊吓放弃路径清理误述、盒中泰拉白天计时表述、WAGboss"三个混合"→四个）、P3×3（`terrarium_fx` 联通→联网与生成点引用、电束／电球措辞、"附身"→电免疫）已修；三页 validate_page --check-indexes 通过、strict 0 错 0 警、record 3 页 check 0 漂移；索引 fx-creature-index（2 行）与 fx-world-index（1 行）。外部独立检视补 P1×1（WAG 电系倍率）、P2×3（SetBeam 采样缓存、hotcold 清理表述、terrarium 昼夜表述）已修，修订后 strict／record 复验通过。未运行游戏、未提交。

**批 U15（2026-11-05）**：3 页 4 注册／5 文件——潜伏梦魇 `ruinsnightmare_horn_attack`（1 注册：双角互删、宿主 combat 结算、每角独立 `targets`、落点固定、裂隙态击退）、暗影之手／恐惧之手 `shadowhand_arm`（2 注册：普通／遮蔽两套 build、固定起点 stretcher、无自删、查理手部第二生成点）、暗影憎恶死亡水洼之眼 `shrouden_eye_fx`（1 注册：本地 goop＋animsync、PUDDLE 检视链、六个死亡状态单槽清理）。独立复审（主会话逐锚点核 4 注册／5 文件＋po）：P1×0、P2×2（arm 页 `enterlight` 监听范围与 eye 页"移动中玩家"条件收窄）、P3×3（arm 生成条件与"第二处"措辞、horn 退场路径、eye 客户端初始动画）已修；三页 validate_page --check-indexes 通过、strict 0 错 0 警、record 3 页 check 0 漂移；**修复 fx-creature-index 表格两处断裂**（表头前 6 行游离、中段空行导致后 2 行无表头）并登记 3 行。外部独立检视补 P2×4（horn entitysleep、eye 状态表与 goop 级联、arm 消散表述）已修，修订后 strict／record 复验通过。未运行游戏、未提交。

**外部独立检视（2026-11-05，[fx-independent-review-20261105.md](fx-independent-review-20261105.md)）**：对 U15～U17 九页 19 注册＋1 placer 独立复核，报 P1×2（WAG 电系倍率、WX-78 模块覆盖）、P2×7（SetBeam 采样缓存、horn entitysleep、hotcold 清理表述、terrarium 昼夜表述、eye 状态表与 goop 级联、arm 消散表述）、R-01（全库 strict 口径）。全部采纳：七页修订后逐页 strict 0 错 0 警、record 7 页 0 漂移；全库复跑 1,146 页 0 failed、29,309 引用 0 error／27 存量 warning（strict 未通过）、7,415 组 0 漂移；R-01 已写入最终报告与 PLAN。该报告保留为修复前记录，不覆盖原验收段。

**第 1 轮独立检视处置（2026-11-05，[fx-review-round01-20261105.md](fx-review-round01-20261105.md)）**：对原生成批 1～2 的 8 页 74 注册复核，报 P2×7、P3×2，全部采纳。R01 月玻璃再生改为采空后监听并注明回调不校验 `fullmoon`；R02 revive drip 补 `Extend` 续期路径；R03 战歌谱号任务补 buff 归属与移除取消；R04 幽灵花补待执行任务与双链边界；R05 补 `prefabs/fx.lua`、`recipes.lua` 来源并 record；R06 升灵补 `nofadeout`；R07 治疗补 `oldager` 前提；R08 清理待登记旧注；R09 链至 `staffcastfx`。复验另补 P2×2：第 85 行分开 `battlesong_fx_task`（目标字段、任务归 buff）与 `display_fx_task`（`SingingInspiration` 组件字段、任务归歌手实体）；sources 补 `entityscript.lua`。5 页修订后 validate_page 通过、strict 264 引用 0 错 0 警、76 组来源比较 0 漂移（record 2 页 refreshed 2）。页组仍为 accepted；未运行游戏、未提交、未推送、未改 raw。

**第 2 轮独立检视处置（2026-11-05，[fx-review-round02-20261105.md](fx-review-round02-20261105.md)）**：对原生成批 3（F07～F10）6 页 25 注册复核，报 P2×10、P3×3，R01～R13 全部采纳、无驳回。R01 调谐换绑静默限定"同一实际 tag"，补等价但不同 tag（肉像 ↔ 复活墓）互换会按失去旧 tag 落 `attune_in_fx`／`attune_ghost_in_fx` 并清 `_isrezattuned`；R02 牛变身改为完成驯化与已驯化后野性化共用 `become_domesticated`；R03 补 ripple 调 `SetOceanBlendParams` 而 swim 不调；R04 补 `prefabs/player_common.lua`、`prefabs/wobster.lua` 来源；R05 改为 floater 普通冷热分支走 splash 族，沉水物品与投掷瓶另有 green 族调用；R06 浮具两分支改为条件生成点；R07 出口 tracker blocker 与受阻后偏移搜索分开，周期链到 L10／L255；R08 `slide_puff` 宿主改鱼人／鱼人守卫并补 `merm.lua`、`mermking.lua` 来源；R09 插值链到 `OnTackleCollide`、Trample 保留目标中心；R10 spawn 任务改为按动画倍率缩放的 0／20／44 基准帧；R11 状态名改 `actual_alert`；R12 wobster 改为投射后落回水中；R13 删待登记旧注。索引 `fx-table-index` 同步 2 行。6 页修订后 validate_page 通过（含索引 7/7、导航 0 问题）、strict 213 引用 0 错 0 警、89 组来源比较 0 漂移（record 2 页 refreshed 2）。页组仍为 accepted；未运行游戏、未提交、未推送、未改 raw。余 135 页待独立检视（不等于待建）。

**第 2 轮修后复验补正（2026-11-05，R14，P2）**：复验报告判 R01 部分完成——"同 tag 换绑静默"还缺 `_isrezattuned` 状态前提。已补：标志为 true 时同 tag 换绑才静默；等价不同 tag 互换清标志后，下一次同 tag 换绑会被 `OnGotNewAttunement` 当作从无到有重播 `attune_out_fx` 并设回 true，静默并非同 tag 固有属性。已改第 73 行、摘要与 `fx-table-index` 第 35 行，并把 `OnGotNewAttunement` 条件写明为 `_isrezattuned` 为 false。该页 validate_page 通过、7 页 strict 213 引用 0 错 0 警、6 页 89 组来源比较 0 漂移。页组仍 accepted；未运行游戏、未提交、未推送。

**第 3 轮独立检视处置（2026-11-05，[fx-review-round03-20261105.md](fx-review-round03-20261105.md)）**：对原生成批 4（F11～F13）4 页 24 注册复核，报 P1×2、P2×8、P3×2；采纳 10 项、驳回 2 项误报。R01 驳回：`fx.lua` 的 `boat_otterden_erode`／`_water` 在 L3401、L3415 实际调用 `SetScale(0.75,0.75,0.75)`，与草船一致，"同 fn"无误。R04 驳回：`world.lua` L555 在 `MakeWorld` 的 fn 内执行 `inst:SetPrefabName("world")`，PostInit 按该名派发，`AddPrefabPostInit("world")` 有效；不改示例，仅补该证据句。其余：R02 为侵蚀页补 13 个事件源并 record；R03 补 `LongUpdate` 补算到期直接落 fx 的路径；R05 快速拆船因 `onremove` 先清 `mast.boat`、无 `boat_death` 时改出 `collapse_small`；R06 船件页补 `prefabs/fx.lua` 来源并 record；R07 改为 death 状态不额外生成、致死碰撞可先在 hit 生成；R08 读档 nofuel 受 `POPULATING` 抑制，运行时耗尽才出气球；R09 撞墙范围改为排除发射者所在船的墙（含陆地墙）；R10 保留伤害归属但删除注册会中断后续码头伤害与 Remove，不可删注册关效果；R11 其余四名无音效；R12 链到 `junk_break_fx`／`ice_crack_grid_fx`。索引 `fx-table-world-index` 第 37 行同步。4 页修订后 validate_page 6/6（含两索引、导航 0 问题）、strict 191 引用 0 错 0 警、4 页 80 组来源比较 0 漂移（record 3 页 refreshed 3）。页组仍 accepted；未运行游戏、未提交、未推送。

**第 3 轮收口补正（2026-11-05，R13～R16）**：R13 侵蚀页第 103 行旧断言改为 `SetRepairedTime` 调用点含 `repaired_kelp`、存档恢复与 `LongUpdate` 重排；R14 炮弹页第 47 行改为"生成调用本身可移除或替换，但后续码头伤害与 `Remove` 顺序必须保留"，与第 77 行"只删注册会 nil 访问"分开；R15 world Hook 说明补第二次派发：先按实例名 `world`，再按请求名 `forest`（[mainfunctions.lua L392-L407](../../../raw/current/scripts/mainfunctions.lua#L392-L407)），并把 `mainfunctions.lua` 加入 sources；R16 船件页 sources 补 `prefabs/oceanwhirlbigportal.lua`。两页 record 刷新。4 页复跑 validate_page 6/6、strict 193 引用 0 错 0 警、4 页 82 组来源比较 0 漂移。R01／R04 仍为误报撤回；页组仍 accepted；未运行游戏、未提交、未推送。

**第 4 轮独立检视处置（2026-11-05，[fx-review-round04-20261105.md](fx-review-round04-20261105.md)）**：对原生成批 5（5 页 20 注册）复核，报 P1×2、P2×5，R01～R07 全部采纳、无驳回。R01 鱼人王世界事件只推 TheWorld，鱼人监听后向自身转推 `onmermkingcreated`，而 SGmerm 只监听 `onmermkingcreated_anywhere`，故撤回“世界事件→全体 buff 水花”的已核验结论，保留 buff 态内生成语句；R02 三页补 `prefabs/fx.lua`（浮渣／鱼人另补 `languages/chinese_s.po`）并 record 3 页；R03 浮渣父挂只写已核验的代理 1 秒回收与本地 animover，`SetParent` 未登记 `EntityScript.children`，引擎随父删除待核验；R04 `OnPicked` 返回值被 `Pickable:Pick` 丢弃，宿主由 `remove_when_picked` 移除、本体不给玩家另由 `canbepickedup=false` 保证；R05 作祟复用 `OnPicked` 但不经 `Pickable:Pick`、不移除宿主；R06 只删水柱注册会中断 `endgeyser`／海堆销毁／三叉戟的未判空调用；R07 击飞门禁是 `oceanfishable` 组件（含可钓浮渣），不按鱼类标签。索引 `fx-table-index` 第 31 行同步。5 页修订后 validate_page 7/7 页面结构通过、strict 180 引用 0 错 0 警、45 组来源比较 0 漂移（record 3 页 refreshed 3）；全局导航仅因他方未跟踪 axe／pickaxe／shovel 三页缺叶级登记而 false，与本批 5 页无关。页组仍 accepted；未运行游戏、未改 raw、未提交、未推送。累计已独立检视 32／158 页，余 126 页；下一未检轮次为原批 6 的 5 页／27 注册。

**第 4 轮复验 R08 收口（2026-11-06，[fx-review-round04-20261105.md](fx-review-round04-20261105.md) 文末）**：补 4 个页面—文件依赖——浮渣页 `components/pickable.lua`、`components/hauntable.lua`、`entityscript.lua`；鱼人页 `entityscript.lua`。两页 record 后 49 组来源比较 0 漂移；修正后第 4 轮待修项归零。

**第 5 轮独立检视处置（2026-11-06，[fx-review-round05-20261106.md](fx-review-round05-20261106.md)）**：对原生成批 6（5 页 27 注册）复核，报 P1×1、P2×8，R01～R09 全部采纳、无驳回。R01 firesplash 补 SGshadowwaxwell `idle`／`ready_pre` 在“有 timer 且无 `shadowstrike_cd`”时 `SetRange(5)` 的可见门禁，以及 `shadowprotector`／`shadowworker` 的 `protectorfn`／`workerfn`→`MakeOblivionSeeker` timer 前提（timer 不等于必然突袭）；撤回“attackrange 赋值在事件包”的旧说法，仅保留 waxwell_shadowstriker 事件包内阵容缺口。R02 lavaarena 父挂改为只保证跟随移动；`SetParent` 未登记 `AddChild` 的 children，宿主提前移除的引擎级联待核验。R03 地陷页补 `components/quaker.lua`、`SGdaywalker.lua`、`SGbearger.lua`，暴食页补 `prefabs/fx.lua`。R04 地陷页三处参数改为：caveins 玩家 4 单位内只把中心切到玩家坐标、仍按 spread 偏移；antlion 环点 x／z 各轴向外 0～10%，不是正负抖动；眼魔直出中心不缩放、环 0.6 起加 0～0.2。R05 补 autoterraformer:DoTerraform 对 `HandleDugGround` 的共享调用与 antlionhat 装备链（草皮映射缺失才出，onfinish 另出 turf_smoke_fx）。R06 glass_fx 改为 22 帧任务与 `_transform` animover 独立调度，不主张固定先后或同帧，GLASS_TIME 锚点修到 L29。R07 暴食 REPLATE 改为“可见段未直接生成”，Salt／Replate 内部与 true 参数保留事件包缺口。R08 火系页宿主改用官方名「龙蝇」、保留「蜻蜓龙」检索别名，补 DRAGONFLY／WAXWELL_SHADOWSTRIKER 的 PO 锚点。R09 从 cavein_dust_low aliases 移除 `cavein_debris`，简条改为链接 sinkhole 主知识页；定向复核后该注册名仅一个主要 aliases 归属。索引 `fx-table-index` 第 29 行同步。8 页修订后页面结构 10/10；全局导航当前被他方新未跟踪 lunarplant-tools.md 缺叶级登记阻断，与本批无关；strict 304 引用 0 错 0 警、92 组来源比较 0 漂移（record 5 页 refreshed 5）。页组仍 accepted；未运行游戏、未改 raw、未提交、未推送。累计已独立检视 37／158 页，余 121 页。

**第 5 轮 R10 收口（2026-11-06，[fx-review-round05-20261106.md](fx-review-round05-20261106.md) 文末）**：`shadowworker` 的 `workerfn` 同样调用 `MakeOblivionSeeker` 安装 timer，原“其他 MakeMinion 变体无 timer”的概括已删除；改为 `shadowprotector`／`shadowworker` 均具备 timer 前提，但 timer 只是范围门禁，不能据此断言必然突袭。

**第 6 轮独立检视处置（2026-11-06，[fx-review-round06-20261106.md](fx-review-round06-20261106.md)）**：对原生成批 7（3 页 27 注册）复核，报 P2×4、P3×2，R01～R06 全部采纳。R01 fx_book_moon 改为 book2 第 44 帧只准备 `fn50`、普通正式 READ 第 50 帧才 `PerformBufferedAction`，并收窄 READ ActionHandler 与 `book_repeatcast` 的 CASTAOE 入口；R02 高尔夫 0.5 秒改为初始随机相位后累计满 0.5 秒即每帧检查（不重置），球体停稳另显式 `CheckTeleport`，`golfpropitem` 组件页同步删除“每 0.5 秒／轮询间隔 0.5 秒”；R03 狂欢节 sources 补 `prefabs/fx.lua` 并 record；R04 fx_book_moon 区分主模拟代理回收与“各非专服端（含主机）建本地表现”；R05 狂欢节验证记录保留 2026-10-31 历史检查并补当前 240 行／24 KiB 与 `fx-table-index` 已登记状态；R06 冬季盛宴普通食物改为“投食阶段先被 RefuseFood／canfeast=false；直接进组件入口且无 finiteuses 才静默返回”。4 页修订后页面结构 4/4 通过；全局导航当前被他方 fumarole-tools／lunarplant-tools／spear 三页缺叶级登记阻断，与本批无关；strict 125 引用 0 错 0 警、29 组来源比较 0 漂移（record 1 页 refreshed 1）。页组仍 accepted；未运行游戏、未改 raw、未提交、未推送。累计已独立检视 40／158 页，余 118 页。

**第 7 轮独立检视处置（2026-11-06，[fx-review-round07-20261106.md](fx-review-round07-20261106.md)）**：合并原生成批 8、9（9 页 44 注册）复核，报 P2×1、P3×1，R01／R02 全部采纳。R01 shadow_teleport_in 将“搜索到达半径 30”改为“循环变量 `radius` 达到 30”，并说明每轮先加 2、再把该轮 offset 累加到当前位置，30 不是实际搜索半径；R02 宝盒改为开始播放 `activation` 后立即排入 `174/30` 秒后的 `teach`，不等 animover。该页页面结构 1/1、全库导航 0 问题通过；strict 19 引用 0 错 0 警、8 组来源比较 0 漂移（未 record）。页组仍 accepted；未运行游戏、未改 raw、未提交、未推送。累计已独立检视 49／158 页，余 109 页。

**第 8 轮独立检视处置（2026-11-06，[fx-review-round08-20261106.md](fx-review-round08-20261106.md)）**：合并原生成批 10、11（11 页 69 注册）复核，报 P3×1，R01 采纳。wurt_terraformer_fx_shadow 的注册表把 `wurt_water_splash_1`／`2`／`3` 写全为三个完整注册名，保留合并行；aliases 与 sources 不变。该页页面结构 1/1；全库导航仍被范围外并行装备页 `nightsword`、`ruins-bat` 缺叶级登记阻断，不能记为全库通过；strict 18 引用 0 错 0 警、5 组来源比较 0 漂移（未 record）。页组仍 accepted；未运行游戏、未改 raw、未提交、未推送。累计已独立检视 60／158 页，余 98 页。

**第 9 轮独立检视处置（2026-11-06，[fx-review-round09-20261106.md](fx-review-round09-20261106.md)）**：原批 12＋外围 C（8 页 45 注册）复核，报 P2×1、P3×1，R01／R02 全部采纳。R01 lightning 的 sources 补入正文已引用的 `prefabs/forest.lua`，按月暴预载断言保留该依赖辨析，并 record 该页（refreshed 1）；R02 minotaur_blood1 注册表写全 `minotaur_blood1`／`minotaur_blood2`／`minotaur_blood3`，保留合并行，aliases／sources 不变。两页页面结构 2/2；全库导航通过（1,143 知识页／34 索引／0 问题）；strict 62 引用 0 错 0 警、26 组来源比较 0 漂移。页组仍 accepted；未运行游戏、未改 raw、未提交、未推送。累计已独立检视 68／158 页，余 90 页。

**第 10 轮独立检视处置（2026-11-06，[fx-review-round10-20261106.md](fx-review-round10-20261106.md)）**：外围 D＋E（7 页 121 注册）复核，报 P2×7、P3×3，R01～R10 全部采纳。R01 rain 三处寿命改为 `UnitRand` 实际范围（约 1.5～7.5、0～60、0.56～0.7 秒），补 `emitters.lua` 来源并 record（refreshed 1）；R02 fire 区分通用 `Make*Burnable` 2／3／4 与角色／尸体 1／2／3；R03 分开 `eyeflame` 非联网与 `firehit` 联网代理＋非专服本地可见体；R04 houndwarning 改为 range 内基础半径、超出按 `inQuad` 增长，撤回对源码注释的否定；R05 阴影清理改为宿主落地／退出 `Remove`、`onremove` 兜底，并补 quaker／草鳄／遗迹三处证据；R06 wave_med 补 `idleTime<=0` 进 `lower`；R07 reticuleaoe 改为先生成再按 `Flash` 分支；R08 fire 的 `burnfx` 改称文件内 local 表；R09 链接标签改 `stageusher`；R10 岸浪动画写全四个实际名。7 页结构 7/7；全库导航通过（1,143 知识页／34 索引／0 问题）；strict 509 引用 0 错 0 警、195 组来源比较 0 漂移（record 1 页 refreshed 1）。页组仍 accepted；未运行游戏、未改 raw、未提交、未推送。累计已独立检视 75／158 页，余 83 页。

**第 10 轮 R05 摘要补正（2026-11-06，[fx-review-round10-20261106.md](fx-review-round10-20261106.md) 文末）**：复验指出 houndwarning_lvl1 第 24 行把 `onremove` 兜底泛化到 `meteorwarning`。已改为“`warningshadow`、`meteorwarning` 均需宿主显式清理”；`warningshadow` 的落地／退出清理与兜底保留在第 87 行，`meteorwarning` 仅在 `shadowmeteor:onexplode` 清理仍见第 110 行。该页结构 1/1、全库导航通过（1,143 知识页／34 索引／0 问题）、strict 109 引用 0 错 0 警、41 组来源 unchanged。未运行游戏、未改 raw、未提交、未推送。累计仍 75／158 页，余 83 页。

**第 11 轮独立检视处置（2026-11-06，[fx-review-round11-20261106.md](fx-review-round11-20261106.md)）**：外围 F～I（12 页 80 注册）复核，报 P2×9、P3×4，R01～R13 全部采纳。R01 五页分开 `SetParent` 跟随与显式清理，未验证分支保留引擎级联未核验；fence／ice／alter／madscience 补 `EntityScript.children` 依据与 `entityscript.lua` 来源并 record 4 页。R02 archive 删除“远端默认 idle”，保留无专用 netvar 的调用边界；R03 madscience 雾采样改 `UnitRand()*radius`，并说明方形采样与原版多边形采样不同；R04 ice_splash 区分普通融化与 `CleanupForced`；R05 改为不监听 `leveldirty` 但周期 `OnUpdateLight`、`aurafn`、`getstatus` 读取 `_level`；R06 焦灌木本地代理改 `SetMultColour(.4,.4,.4,1)`；R07 删除焦灌木壳“不可父挂”旧说法；R08 buff FX 标每路 10～30、两路合计 20～60 粒/秒；R09 minion `AOEUtil.Work` 查询半径 2.9、无 arc／hitbox 窄相；R10 nitre 改可见联网表现实体；R11 madscience 统一“四组六个注册”；R12 fence 的 `.po` 来源描述改为已列入；R13 补 `propagator` helper 行。12 页结构 12/12；全库导航通过（1,143 知识页／34 索引／0 问题）；strict 930 引用 0 错 0 警（跳过 1 个普通页链）、155 组来源比较 0 漂移（record 4 页 refreshed 4）。页组仍 accepted；未运行游戏、未改 raw、未提交、未推送。累计已独立检视 87／158 页，余 71 页。

**第 12 轮独立检视处置（2026-11-06，[fx-review-round12-20261106.md](fx-review-round12-20261106.md)）**：外围 L／M／O／Q（11 页 62 注册）复核，报 P2×13、P3×1，R01～R14 全部采纳。R01 eye_charge 改为“计数初值 5，但当前表达式不会在耗尽时停止重试”；R02 grasspart 改为 GENERIC＋MEAT 双类型、素食鱼可匹配，补 `constants.lua` 来源并 record；R03 鱼饵云消散后周期任务仍可补到上限，只有即时补位检查 `persists`；R04 gestalt 背层动画改由宿主 `SGgelblob` 显式驱动，补 SG 来源并 record；R05 三页移除 Follower→Lua follower 组件误链，保留引擎符号跟随；R06 wormlight／carnival 只陈述 Lua 写入端，引擎 Light／AnimState 复制未实测；R07 三种灯笼闪烁改为半径 1.2～1.3、颜色 +0～2/255，各端独立起始相位；R08 wormlight 骑手 bloom push 受 `_popbloom` 门禁；R09 shadow_pillar／carnival 裸 `SetParent` 删除级联未核验；R10 shadow_pillar `newpillars` 实体键表配 `ipairs`，当前去重不生效；R11 carnival 风扇示例补 `ToggleSpin` 与立即应用；R12 saddle 案例改为实参不匹配、旧登记未移除，并收窄清池分支；R13 grasspart 渔网子链联网范围；R14 raindrop 两条“误区”改“边界”。11 页结构 11/11；全库导航通过（1,143 知识页／34 索引／0 问题）；strict 894 引用 0 错 0 警（跳过 1 个普通页链）、185 组来源比较 0 漂移（record 2 页 refreshed 2）。页组仍 accepted；未运行游戏、未改 raw、未提交、未推送。累计已独立检视 98／158 页，余 60 页。

**第 12 轮 R06／R07 精度补正（2026-11-06，[fx-review-round12-20261106.md](fx-review-round12-20261106.md) 文末）**：复验指出两处仍不精确。wormlight 三种灯笼的 `flicker` 由三项正弦归一化、源码未 clamp，不能写成严格 0～1；已改为“半径约 1.2～1.3”，颜色写为基色+`2/255*flicker`，并注明未 clamp。carnival 的 `carrot_spinner` 透明度初始化在 `SetPristine` 前由双端执行，只有后续淡变由主模拟 `updatelooper` 推进；已据此改写。两页结构 2/2、全库导航通过（1,143 知识页／34 索引／0 问题）、strict 212 引用 0 错 0 警。未运行游戏、未改 raw、未提交、未推送。累计仍 98／158 页，余 60 页。

**第 13 轮独立检视处置（2026-11-07，[fx-review-round13-20261106.md](fx-review-round13-20261106.md) 文末）**：外围 P／R／S／T＋U1（14 页 81 登记名）复核，报 P2×16，R01～R16 全部采纳。abigail_retaliation 撤回“gestalt ×3 为死代码”，改为实体 `AddDebuff` 六参包装把施加者透传到 `buff_OnExtended`，带 `gestalt` 标签时 2 秒→6 秒，并补 `components/incinerator.lua`、`standardcomponents.lua`、`components/debuffable.lua`、`components/debuff.lua` 四份来源后 record；staff_lunarplant_fx 把 `colouradder` 收窄为 Lua 主模拟写入、无自定义 netvar，滚雪球改述宿主雪球事件；shadow_trap 分开实体 `noattack` 标签与 SG 状态标签、`death` 与 `onremove` 动作；shadow_despawn 补光束施法者的 `health`／`combat`／tagset／符号前提；oceanhorror_ripples 撤回未经核验的父子级联保证；rocky_snip_fx 分开失真停止与本地实体回收、dash 空 target 不再承诺等待；rope_bridge_fx 把 `killed` 限定绳桥，拆分 0.5～0.8 秒生成、约 18 秒警告、约 20⅓ 秒销毁、默认约 4.3～4.7 秒销毁，并改述藤桥管理器为共同世界工厂挂载；sleepcloud 按 AND／OR 条件重写 Limbo 腐败排除；oceantree_ripples_short 区分活树砍桩／烧毁／stump prefab 三条读档分支，`fx-world-index` 两行同步 12 注册与大桥时序。9 页＋索引结构 10/10、全库导航 1,145 知识页／34 索引／0 问题；strict 713 引用 0 错 0 警、139 组来源比较 0 漂移（record 1 页 refreshed 1）。页组仍 accepted；未运行游戏、未改 raw、未提交、未推送。累计已独立检视 112／158 页，余 46 页。

**第 13 轮修后独立复验（2026-11-07，[fx-review-round13-20261106.md](fx-review-round13-20261106.md) 文末）**：R01～R16 全部通过，第 13 轮收口；14 原页＋叶索引 15/15 格式通过，strict 1,083 引用零错警、202 来源 unchanged、9 次标题读取与 2 段改动 Lua 语法通过；全局导航仅报并行新增 queen-torch 未登记，未代改。未 record、未运行游戏、未提交推送。

**第 14 轮独立检视处置（2026-11-07，[fx-review-round14-20261107.md](fx-review-round14-20261107.md) 文末）**：外围 U2～U6（14 页 56 主要注册）复核，报 P2×22，R01～R22 全部采纳。R01 给 shatter 补 `prefablist.lua`／`widgets/werebadge.lua`，给 spear_gungnir_lungefx 补 `prefabs/lavaarena.lua`，给 terrariumchest_fx 补 `worldsettings_overrides.lua`，给 winona_battery_high_shatterfx 补 `mainfunctions.lua`，给 forcefieldfx／wurt_shadow_merm_planar_fx 补根目录 `fx.lua`，给 wx78_big_spark 补 `tuning.lua`，把 wortox_soulecho_buff_fx 的 `prefabs/fx.lua` 换成根目录 `fx.lua`；R06 另给 cavein_dust_low 补 `prefabs/hats.lua`，R19 给 waterplant_pollen_fx 补 `entityscript.lua`。R02 八页保留显式 Remove／AddChild 链，纯 `SetParent` 改为引擎层未核验；R03 分开化石基座引用 nil／失效分支；R04 修正南瓜碎块 PRNG 的 `+0.5` 取整；R05 改 brokentool 为作祟成功生成、状态名 `toolbroke`；R06 补 `MakeHat("scrap")`→`scraphat` 动态注册；R07 改恐惧弹 FX 为 25 帧攻击、完整动画长度定时回池；R08 限定体形档位到 combat 回退分支并补符号门禁；R09 删除双层壳 netvar 说法；R10 奔雷突刺光改为动画与灯光两条件都完成；R11 快照外熔炉消费方改“可能”；R12 碎宝石改为新壳默认动画比较；R13 灯点半径补 `i+2`；R14 标 lightbeam 的 `inst.sg` 门禁未证实可达；R15 扫描束死亡分支改为先 StopScanFX 再 OnScanFailed；R16 大火花明确世界位置生成、不持续跟随；R17 补提取器共享 `erode` 的 Light／Remove 分支；R18 灵魂粒子改约 3～30 粒／秒并补出生速度差异；R19 空中花粉云删除监听与 35 帧任务均主模拟；R20 明确 `persists=false` 只挡即时补位；R21 护盾重入允许 0.6 秒退场重叠；R22 分开 debuff 计时终止、暗影持续层与月影一次性 FX。14/14 页面格式通过；全库导航 1,147 知识页／34 索引，仅并行 queen-torch／redlantern 未登记；strict 940 引用 0 错 0 警、786 唯一目标、122 源文件；14 页哈希 171 组 unchanged，record 10 页 refreshed 10。页组仍 accepted；未运行游戏、未改 raw、未提交、未推送。累计已独立检视 126／158 页，余 32 页。

**第 14 轮修后独立复验（2026-11-07，[fx-review-round14-20261107.md](fx-review-round14-20261107.md) 文末）**：18 项通过，R03／R08／R17／R19 仍待修；分别为化石基座动画完成时刻无时长依据、纯辉标记把 `success` 扩到 size／动画、提取器 `doerode` 误写为超时、空中花粉云清理段残留双端自删。本批暂不收口。

**第 14 轮 R03／R08／R17／R19 补正（2026-11-07）**：化石基座删除“`base_pst` 未播完”断言，改为实际时长未核验；纯辉标记第 36、40 行统一为有符号即 clamp size／播动画、`GetSymbolPosition` 成功才 `FollowSymbol`；提取器改为 `capture_emote` 入态 1 秒时间线推 `doerode`，并补 `ERODEOUT_DATA` 的 4 秒／`erodein=false`／`remove=true`；空中花粉云第 62 行同步为主模拟 `animover` 删除、客户端无监听。四页 4/4 格式通过，strict 231 引用 0 错 0 警、200 唯一目标、31 源文件，37 组来源 unchanged；导航仍仅并行 queen-torch／redlantern 未登记。未运行游戏、未改 raw、未提交、未推送。累计仍 126／158 页，余 32 页；四项已补正，待复验。

**第 15 轮独立检视处置（2026-11-07，[fx-review-round15-20261107.md](fx-review-round15-20261107.md) 文末）**：外围 U7～U10（12 页 57 主要注册）复核，报 P2×26、P3×2，R01～R28 全部采纳。R01 mushtree 补 `prefabs/mushtree_webbed.lua` 并 record；R02 五页分开 SetParent 与 AddChild（trap_vines 底座、boatrace 浮标、shadowrift 粒子、voidcloth 装备层、vault 净化雾），显式删除保留、纯父挂回收标未核验；R03 陷阱烧层改为 RemoveComponent 才走 OnRemoveFromEntity、实体 Remove 走 OnRemoveEntity；R04 煤火改 10/秒×倍率×实际间隔；R05 藤蔓 `data.varation` 写／`data.variation` 读字段不配对；R06 龙舟旗挂点按名次、图案按选手编号并补 win/prize 暂缓；R07 烟花塔位独立播放不跟随；R08 浮标只挡目标瓦片；R09 scythe 本体有官方名、其余四个附属名无 NAMES；R10 GetSkinBuild 限定魔镰／暗影伞；R11 伞 wave/dome 淡出并扩张；R12 专服骨架保留实际组件；R13 撤回 marker 必入世界存档并补 mainfunctions 门禁；R14 地纹朝向折算与 3/4 镜像 -90°；R15 罗盘 VAULT 无方向保留旧动画；R16 删除演员退场连带删壳保证；R17 fullhelm 补 owner.isplayer＋BuildHasSymbol，外观分静态符号／全跟随层；R18 祭坛读档重新播 break，不写静止第一帧；R19 `trapdamage` 改为参与出生保护危险计数并补门槛；R20 荆棘 owner 分支补 CanTarget／非 IsAlly；R21 蜂蜜减速用初始化坐标与半径，不逐 tick 重读；R22 焦地两端 `set_local`、`set` 只在构造／加载；R23 暗影憎恶阈值 speed<1 不落／1–4→6／≥4→3，弹弓蜜补 k 公式与 duration 随机；R24 作物仅成熟／枯萎挖起才收获；R25 冲锋尘土 4–6 delay 递减到 0 后下一次回调才生成；R26 焦壳读档来自保存时仍在燃烧的树；R27 两处 PO 锚点补到 msgstr；R28 删除 1～2 秒动画时长估计；R06／R19／R23／R25 补正后二次复核再收紧（编号非名次、loading 回落、蜂王尺寸三档、4–6 非首次延迟）。12/12 页面格式通过；全局导航 1,147 知识页／34 索引／0 问题；strict 643 引用 0 错 0 警、586 唯一目标、92 源文件；12 页哈希 122 组 unchanged，record 2 页 refreshed 2。页组仍 accepted；未运行游戏、未改 raw、未提交、未推送。累计已独立检视 138／158 页，余 20 页。

**第 15 轮 R06／R19／R23／R25 补正（2026-11-07）**：R06 第 30 行改 `swapflag_..id`（id 为选手编号 1～8，仅挂点 1–3 限前三），第 31 行同步 `fx_spark_parts_0..winid`；R19 拆开 always 非 loading 直接 true、never 直接 false、默认分支才要求 cycles>1 且危险数或结构门槛任一成立；R23 第 58 行蜂王 duration 改基数 1.2／2／4 再 +[0,0.5)，第 104 行弹弓尺寸改 0.3–1.1；R25 改 `{delay=0}` 首次立即生成、生成时重设 4–6、后续 5–7 次回调。四页 4/4 格式通过，strict 230 引用 0 错 0 警、206 唯一目标、41 源文件，50 组来源 unchanged；导航 1,172 知识页／35 索引，仅并行 misc-equipment-2／ranged-weapons／staffs-more 未登记。未运行游戏、未改 raw、未提交、未推送。累计仍 138／158 页，余 20 页；R29／R30 仍待处理，本批未收口。

**第 16 轮独立检视处置（2026-11-07，[fx-review-round16-20261107.md](fx-review-round16-20261107.md) 文末）**：外围 U11～U14（12 页 37 主要注册）复核，报 P1×1、P2×27、P3×2，R01～R30 全部采纳，12 页均改并同步 `updated`。R01 伯尼示例把 `EndFire` 前移、去掉多余 `inst` 实参；R02 archive 补 `archive_chandelier.lua`／`tuning.lua`、toadstool 补 `map/terrain.lua`／`map/static_layouts/toadstool_arena.lua` 并 record；R03 七页分开 Lua `children` 删除与未核验原生 `SetParent`；R04 首次 `SetFX("idle")` 只记状态；R05 highlight 改按需装配；R06 彩带 buff 改 Extend 空回调、OnTick 维持；R07 密度方向改“剩余越少越疏”；R08 改每次新建实体；R09 收尾限定正常烹饪完成；R10 分开在播／回池轨迹；R11 示例补回池协议；R12 改每非专服 2 联网层＋6 本地槽层；R13 撤回“骑乘不投放”；R14 删除 20 秒顺序因果；R15 补读档 `canlight` 门禁；R16 惊树前置改“stage 4 或可 CHOP”；R17 `net_tinybyte` 改 0–7；R18 改各端同一 netvar 号；R19 改单次 PostUpdate；R20 示例补位置与 build；R21 区分状态 1 秒与可见约 0.8 秒；R22 改旧链仍生成命中特效；R23 粒子改无淡入、末期 alpha 0.5；R24 改无 level 1 分支；R25 示例返回句柄并给 `KillFX`／`FinishFX`；R26 基名跨端未核验；R27 补 `not ignore_absorb`；R28 任务取消不恢复 `MotorVelOverride`；R29 删“单帧”；R30 改用 msgstr“拟态蠕虫”。顺带修第 15 轮 boatrace 第 31 行 `fx_spark_parts_01..03`→`0..winid`。12/12 页面格式通过；strict 623 引用 0 错 0 警、531 唯一目标、102 源文件；record 7 页后 134 组来源 unchanged、0 changed；4 段改动示例语法通过。未运行游戏、未改 raw、未提交、未推送。累计仍 150／158 页，余 8 页；本批修后独立复验待做；复验补正 R03／R10／R26／R31 后再统一机械检查：扫帚旧“随联网层级联”清零、桶霜气恢复桶体 `OnRemoveEntity` 显式清理、回池项改为“无显式清池路径、不能依赖再次 animover”、基名随机收窄为“客户端构造未核验”，balloon 补 `components/debuff.lua` 并 record、bat 补 `netvars.lua` 行锚点；五页 5/5、strict 300 引用 0 错 0 警、259 唯一目标、62 源文件、73 组来源 unchanged。

**第 17 轮独立检视处置（2026-11-07，[fx-review-round17-20261107.md](fx-review-round17-20261107.md) 文末）**：固定队列最后 8 页／29 主要注册复核，报 P2×7，R01～R07 全部采纳。R01 五页 sources 补 `languages/chinese_s.po`，impact 补 `blowdart_lava.lua`／`blowdart_lava2.lua` 及第 7–10 行锚点；R02 两投影改“无独立自删、父挂只定位、原生随父删除未核验”，裂隙收尾对象明确为 `inst.shadow`；R03 junkball 落点补 workable `Destroy`→pickable `Pick`→combat `DoAttack` 顺序、地面物品抛飞与 `formpile` 聚堆／击退，`targets` 去重限定三路；R04 改“不走攻击者 `DoAttack`，但受击侧仍进 `GetAttacked_Internal`、未 blocked 推 `attacked`／`onhitfn`”；R05 四符号锚点改 `prefabskin.lua` 的 init_fn 与 placer／客户端门禁；R06 碎屑改“无外部参数/assets，但沿用同文件 `prefabs` 预载表”；R07 `startobstacledrop` 补 `spawnlocation`／25² 门禁、3～4 次尝试×最多 8 候选、成功才生成。7 页 7/7；strict 271 引用 0 错 0 警、233 唯一目标、66 源文件；record 6 页后 72 组来源 unchanged、0 changed；`git diff --check` 干净。未运行游戏、未改 raw、未提交、未推送；首次整页检视 158／158 完成，修后独立复验待做；复验补正 R02／R03／R04／R08：impact 区分裂隙未休眠 shadow_disappear＋portal 退场与休眠直接 Remove、天体门单列无对应退场调用，目标处理统一为“择一破坏／采摘／攻击”，onhitfn 注明仅设置后调用，sources 补 `components/combat.lua` 并 record；impact 1/1、strict 126 引用 0 错 0 警、110 唯一目标、27 源文件、27 组来源 unchanged。

54 个未提及候选已全部分诊，下表为其中新增 needs-page 页组（2026-11-05）：

| 组 | 页（拟名） | 文件 | 注册（约） | 验收状态／批次 |
|---|---|---|---:|---|
| 41 | positronbeam_back | positronbeam.lua＋moonbase.lua | 3 | accepted（U14） |
| 42 | ruinsnightmare_horn_attack | ruinsnightmare_horn_attack.lua | 1 | accepted（U15） |
| 43 | shadowhand_arm | shadowhand.lua＋charlie_hand.lua | 2 | accepted（U15） |
| 44 | stalker_shield1 | stalker_shield.lua | 5 | accepted（U14） |
| 45 | hotcold_fx | smallghost.lua | 1 | accepted（U16） |
| 46 | terrarium_fx | terrarium.lua | 1 | accepted（U16） |
| 47 | toadstool_cap_absorbfx | toadstool_cap.lua | 2 | accepted（U14） |
| 48 | shrouden_eye_fx | shrouden.lua＋SGshrouden.lua | 1 | accepted（U15） |
| 49 | wagboss_missile_target_fx | wagboss_missile.lua＋wagdrone_laserwire.lua＋wagdrone_projectile.lua＋SGwx78_drone_zap.lua | 5 | accepted（U16） |
| 50 | wx78_taser_projectile_fx | wx78_taser_projectile.lua＋components/wx78_taserbuildup.lua＋wx78_moduledefs.lua | 1 | accepted（U17） |
| 51 | yotr_fightring_torch | yotr_fightring.lua（ring/kit/bell 混合载体辨析＋torch FX） | 4＋placer | accepted（U17） |

清单外发现（不计入 380 口径，只记观察）：`torchfire_common.lua` 工厂模块 0 注册，其 12 个产物 `torchfire`（torchfire.lua:184）与 `torchfire_pronged/barber/rag/carrat/shadow/pillar/cawnival/nautical/yotrpillowfight/tesla/spooky` 全部无页、未被任何既有页承载，属候选 CSV 之外的原版 FX 空白，留待独立任务。

### 未分诊 54 候选之实核（2026-11-05，21/54 文件，只读分诊代理）

逐文件结论（excluded 13／needs-page 8 文件 17 注册／covered-existing 0／blocked 0）：

| 文件 | 注册名（行号）与结论 | 状态 |
|---|---|---|
| `prefabs/portablespicer.lua` | `portablespicer`(374)、`portablespicer_item`(376)、`portablespicer_item_placer`(375，json 漏记的 MakePlacer)。portablestructure/stewer/container/workable/inventoryitem/deployable。 | excluded（玩法/结构） |
| `prefabs/positronbeam.lua` | `positronbeam_back`(57)、`positronbeam_front`(57)、`positronpulse`(132) 纯 FX；生成点 moonbase.lua:90/97/104。 | needs-page（3 个 FX） |
| `prefabs/resurrectionstone.lua` | `resurrectionstone`(408)：lootdropper/inspectable/hauntable/cooldown；CreateLight 为匿名实体非注册。 | excluded（玩法） |
| `prefabs/rook.lua` | `rook`(292)、`rook_nightmare`(293) 玩法；`ruinsrespawner` 两包装为 worldgen/重生占位。 | excluded（玩法＋内部） |
| `prefabs/ruinsnightmare_horn_attack.lua` | `ruinsnightmare_horn_attack`(206) 穿地犄角投射 FX：无玩法组件、伤害经 owner combat；生成 SGruinsnightmare.lua:117-118。 | needs-page（1 个 FX） |
| `prefabs/shadowhand.lua` | `shadowhand`(346)、`shadowhand_shrouded`(348) 玩法（locomotor/sanityaura/playerprox，生成点 shadowhandspawner.lua:189、lightsout_trial.lua:248、charlie_boss_trial.lua:347）；`shadowhand_arm`(347)、`shadowhand_arm_shrouded`(349) 纯 FX（NOCLICK+FX、highlightchild、stretcher 拉长表现；生成 shadowhand.lua:170 动态 armprefab、charlie_hand.lua:119）。`shadowhand_fx` 注册于 fx.lua、已由 waterplant_destroy.md 承载，与本文件无关。 | excluded（本体）＋needs-page（2 个 FX） |
| `prefabs/shadowheart_infused.lua` | `shadowheart_infused`(167)：inventoryitem/socketable/tradable/sanityaura。L60 `CreateEntity("shadowheart_infused_dark_fx")` 经 mainfunctions.lua:512 核实 name 仅赋 scr.name 调试名，不是注册。 | excluded（玩法物品） |
| `prefabs/shadowthrall_hands.lua` | `shadowthrall_hands`(345)：health/combat/locomotor/planardamage；CreateFlameFx/FabricFx 匿名非注册。 | excluded（玩法） |
| `prefabs/shadowthrall_horns.lua` | `shadowthrall_horns`(331)：health/combat/locomotor。 | excluded（玩法） |
| `prefabs/shadowthrall_wings.lua` | `shadowthrall_wings`(350)：health/combat/locomotor。 | excluded（玩法） |
| `prefabs/sharkboi_icespike.lua` | `sharkboi_icespike`(491)：workable 可挖/lootdropper/障碍物理；`sharkboi_icetunnel_fx`(492) 仅 Transform+CLASSIFIED、无 AnimState 无组件，隐形冰刺生成协调器（生成 SGsharkboi.lua:1129），按内部排除。 | excluded（玩法＋内部） |
| `prefabs/shrouden.lua` | `shrouden`(1620) Boss 玩法；`shrouden_eye_fx`(1621) Boss 死后遗留之眼：AnimState+persists=false+inspectable("PUDDLE") 可检视、无玩法组件；生成 SGshrouden.lua:240。 | excluded（本体）＋needs-page（1 个 FX） |
| `prefabs/smallghost.lua` | `smallghost`(574) NPC 玩法；`hotcold_fx`(573) 冷热小游戏指示 FX（persists=false，生成 smallghost.lua:51）。 | excluded（本体）＋needs-page（1 个 FX） |
| `prefabs/stalker_shield.lua` | `stalker_shield1..4`(128 循环)＋随机变体包装 `stalker_shield`(130)：FX tag、animover 自删、击退与 10 点伤害经 StartRepel 代理；生成 SGstalker.lua:171 动态拼名；基名快照内无直接生成点（诚实缺口，仅 stalker.lua:61 依赖声明）；shadow_shield1.md 复用同名 bank 非本 prefab。 | needs-page（5 名） |
| `prefabs/support_pillar.lua` | 6 注册：scaffold×4＋placer×2；constructionsite/workable/inspectable/lootdropper；CreateHelperRadiusCircle 与 OnDebrisDirty 匿名 FX 非注册。 | excluded（玩法/结构） |
| `prefabs/terrarium.lua` | `terrarium`(646) 召唤器玩法；`terrarium_fx`(647) 召唤光柱 FX（DECOR、persists=false，生成 terrarium.lua:217）。 | excluded（本体）＋needs-page（1 个 FX） |
| `prefabs/toadstool_cap.lua` | `toadstool_cap`(377) Boss 结构；`toadstool_cap_absorbfx`(378)、`toadstool_cap_releasefx`(379) MakeFX 工厂（L340-375，FX+NOCLICK；生成同文件 L78/L56）。 | excluded（本体）＋needs-page（2 个 FX） |
| `prefabs/torch.lua` | `torch`(530)：weapon/lighter/equippable/complexprojectile/fueled；火焰 `torchfire`（deps L10、生成 L149/286）无既有页，但注册于 torchfire.lua 等 12 个工厂文件、不在候选 CSV。 | excluded（玩法；清单外观察） |
| `prefabs/torchfire_common.lua` | 0 注册（MakeTorchFire 工厂 L119，被 12 文件 require）；12 个产物见上"清单外发现"。 | excluded（0 注册工厂模块） |
| `prefabs/vault_crawler.lua` | `vault_crawler`(596) 玩法（生成点 archive_chandelier.lua:486 与该页一致）；`vault_crawler_socket`(597) 无组件但为持久交互世界物（MakePondPhysics 障碍、刻意不加 NOCLICK 挡鼠标 L553-557、pathfinder 封墙、OnSave/OnLoad、生成 vault_key_trial.lua:317），按结构排除。 | excluded（玩法＋结构） |
| `prefabs/vault_lobby_exit.lua` | `vault_lobby_exit`(421)、`vault_key_exit`(422)：teleporter/inspectable/lootdropper/blocker；CreateVaultLadderVisualFor 匿名 FX 非注册。 | excluded（玩法/结构）×2 |

### 未分诊 54 候选之实核·续（2026-11-05，16/54 文件，只读分诊代理；源码穷举 40 注册，json 漏报 13 个包装注册）

结论：纯 FX 仅 1（needs-page）；4 个 `_fx` 命名注册为带 combat 真实结算的混合载体，按批 9 `wagboss_beam_fx` 先例纳入建页（组 49）；其余 excluded。逐文件：

| 文件 | 注册名与结论 | 状态 |
|---|---|---|
| `prefabs/wagboss_missile.lua` | `wagboss_missile`(899) 混合投射物（combat TUNING 伤害＋Detonate AOE 真实结算 L522-625，生成 SGwagboss_robot.lua:1651）；`wagboss_missile_target_fx`(900) 纯 FX（CLASSIFIED+netvar 壳、本地 CreateTargetRing；生成 wagboss_missile.lua:392）。 | excluded（本体）＋needs-page（1 个 FX） |
| `prefabs/wagboss_robot.lua` | `wagboss_robot`(1739)：health/combat/locomotor/commander/lootdropper（生成 wagpunk_arena_manager.lua:1142、constructionsite ReplacePrefab）；背光/皇冠/齿轮/践踏 FX 均未注册内部实体。 | excluded（玩法） |
| `prefabs/wagdrone_flying.lua` | `wagdrone_flying`(272)：locomotor/hackable/SG（生成 gestalt_cage.lua:512/671、wagstaff_npcbrain.lua:135、arena_manager ReplacePrefab）；瞄准标记未注册。 | excluded（玩法） |
| `prefabs/wagdrone_laserwire.lua` | `wagdrone_laserwire_fx`(356) 混合（CLASSIFIED/netvar 壳＋combat 电击真实结算 L153/156；生成 wagdrone_rolling.lua:83；激光段视觉为未注册本地工厂）；`wagpunk_cagewall_fx`(357) 混合、快照内无生成点（诚实缺口；竞技场生成的是不带 fx 的 wagpunk_cagewall L412）。 | needs-page（2 个混合，按批 9 先例） |
| `prefabs/wagdrone_projectile.lua` | `wagdrone_projectile_fx`(308) 混合（FX 标签＋combat WAGDRONE_FLYING_DAMAGE；生成 SGwagdrone_flying.lua:680）；`wx78_drone_zap_projectile_fx`(309) 混合（同构，ZAPDRONE_DAMAGE；生成 SGwx78_drone_zap.lua:518）。 | needs-page（2 个混合） |
| `prefabs/walter.lua` | `walter`(1383) 玩家角色；`walter_campfire_story_proxy`(1384) 理智结算逻辑代理；`wobycourier_marker`(1385)、`_close`(1386) 地图 UI 壳（Transform+MiniMapEntity+globalmapicon）；横幅/冲刺拖尾未注册。 | excluded（玩法＋内部）×4 |
| `prefabs/warg.lua` | `warg`(1082-1087)、`claywarg`(1089-1095)、`gingerbreadwarg`(1097-1103)、`mutatedwarg`(1105-1112) 同一 MakeWarg 工厂玩法（json 漏报 4 个注册）；生成 hunter.lua:368/386/47、gingerbreadhunter.lua:203、corpses_defs 变异；火焰 FX 未注册。 | excluded（玩法）×4 |
| `prefabs/wargshrine.lua` | `wargshrine`(346)：workable/prototyper/trader/burnable/lootdropper（配方生成）；`wargshrine_placer`(347) 内部；L19-20 生成 torchfire/皮肤火属清单外注册。 | excluded（玩法＋内部） |
| `prefabs/winona_catapult.lua` | `winona_catapult`(1285)、`winona_catapult_item`(1287) 玩法、`_item_placer`(1286) 内部；与 winona_catapult_projectile.lua 结论模式一致（表现全在未注册内部工厂）。 | excluded（玩法＋内部） |
| `prefabs/winona_holoitems.lua` | `winona_machineparts_1/2`(259/260)、`winona_holotelepad`(261)、`winona_holotelebrella`(262)、`winona_recipescanner`(263)：五者均有 inventoryitem（recipescanner 另有组件真实解锁配方）；全息视觉为未注册内部工厂；生成 inspectaclesbox.lua 战利品。 | excluded（玩法）×5 |
| `prefabs/winona_teleport_pad.lua` | `winona_teleport_pad`(904)、`winona_teleport_pad_item`(906) 玩法、`_item_placer`(905) 内部；信标/红蓝线/火花未注册。 | excluded（玩法＋内部） |
| `prefabs/winter_tree.lua` | `winter_tree`(1154)、`winter_twiggytree`(1178)、`winter_deciduoustree`(1192)、`winter_palmconetree`(1232)：AddWinterTree 循环工厂（json 漏报 4 个），四者 workable/container/growable/lootdropper/burnable；装饰 FX 经 CloneAsFx 克隆非注册；生成 wintersurprisespawner.lua:210、pinecone.lua:135。 | excluded（玩法）×4 |
| `prefabs/wormwood.lua` | `wormwood`(875) 玩家角色；bloom poof/花粉未注册内部实体；`wormwood_plant_fx` 是外部文件注册、本文件仅依赖声明。 | excluded（玩法） |
| `prefabs/wortox_soul_spawn.lua` | `wortox_soul_spawn`(397) 混合（灵魂球视觉＋weapon/projectile 真实给物掉落 wortox_soul＋SoulSpearTick GetAttacked "soul" 真实伤害；生成 wortox_soul_common.lua:108、panflute.lua:61）；`wortox_soul_spawn_fx` 注册在 fx.lua:3597、已由 wortox_soul_spawn_fx.md 覆盖。 | excluded（混合玩法；另一个 _fx 注册归 fx.lua，不计本文件覆盖） |
| `prefabs/wx78_drone_scout.lua` | `wx78_drone_scout`(371)：mapdeliverable/maprevealer/globaltrackingicon（配方生成）；`wx78_drone_scout_globalicon`(353-369)、`_revealableicon` 经 MakeGlobalTrackingIcons 的地图 UI 壳；扫描光柱/地印未注册。 | excluded（玩法＋内部）×3 |
| `prefabs/wx78_drone_zap.lua` | `wx78_drone_zap`(420)（0 伤 dummy weapon 拉仇恨＋真实电击在 SG）；`wx78_drone_zap_remote`(421)：inventoryitem/equippable/finiteuses/batteryuser（配方生成）。 | excluded（玩法）×2 |

### 未分诊 54 候选之实核·终（2026-11-05，17/54 文件，只读分诊代理；事件包判定）

公共事实：`event_server_data`（entityscript.lua:77）取 `eventname.."_event_server/"..path` 加载，`raw/current/scripts` 下无任何 `*_event_server` 目录——lavaarena/quagmire 事件包不在快照。44 个注册名（含循环展开）全库反查＋wiki 两轮 grep：**零 covered-existing**。结论：**blocked 15／needs-page 2 文件**。

| 文件 | 注册名与结论 | 状态 |
|---|---|---|
| `prefabs/lavaarena_firebomb.lua` | 5 注册；4 条 `event_server_data("lavaarena",…)`（firebomb/projectile/explosion/procfx postinit L85/155/184/213）；全域零 SpawnPrefab；`lavaarena_firebomb_sparks` 工厂自足（L222-249）。 | blocked（lavaarena 事件包） |
| `prefabs/lavaarena_fossilizing.lua` | `lavaarena_fossilizing`(121)＋`lavaarena_fossilizedebris1/2/3`(92-94 拼名，json 漏 4 个 debris)；master_postinit L116 在事件包；4 个 debris 工厂自足。 | blocked（事件包） |
| `prefabs/lavaarena_lucy.lua` | `lavaarena_lucy`(224)＋`lavaarena_lucy_spin`(225 自足工厂)；投掷/回旋与 spin 消费全在 master_postinit L107。 | blocked（事件包） |
| `prefabs/lavaarena_meteor.lua` | 4 注册；meteor/splash/splashhit postinit 在事件包（fireballstaff 的 ult 落雷链）；`lavaarena_meteor_splashbase` 自足。 | blocked（事件包） |
| `prefabs/lavaarena_peghook.lua` | 6 注册（peghook/projectile/spitfx/hitfx/splashfx/dot）；波次生成与 AI 吐射、hitfx/dot postinit 在事件包；spitfx/splashfx 工厂自足；goosplash.md 引用的依赖链不构成覆盖。 | blocked（事件包） |
| `prefabs/lavaarena_portal.lua` | `lavaarena_portal`(143)＋`lavaarena_keyhole`(144 自足)＋`lavaarena_portal_activefx`(145 自足)；keyhole/activefx 生成在缺失 master_postinit。附注：portal 本体在快照 worldgen 有真实生成（lava_arena.lua:250），表述为"生成在快照、初始化在事件包"。 | blocked（事件包；中置信限定） |
| `prefabs/lavaarena_rhinobumpfx.lua` | `rhinobumpfx`(56) 工厂完全自足（批 F rhinodrill 先例原型）；生成在 rhinodrill 攻击逻辑（master_postinit L186 走事件包）。 | blocked（事件包） |
| `prefabs/lavaarena_snapper.lua` | `snapper`(98)＋`goo_spit_fx`(99，FX 本体 postinit 也在事件包 L93)。 | blocked（事件包） |
| `prefabs/quagmire_coins.lua` | `quagmire_coin1..4`(43 拼名)＋`quagmire_coin_fx`(74)；发币/计分全缺；quagmire.lua 40 余条 Recipe 均为 Ingredient 花币引用非生成。 | blocked（quagmire 事件包） |
| `prefabs/quagmire_cooking_buff.lua` | `quagmire_cooking_buff`(101) 本文件无 event_server_data、工厂完全自足（ShowFX/HideFX＋客户端 CreateFX），但 ShowFX 调用方（锅具 postinit）在缺失数据。 | blocked（事件包；"注册可辨析、生成在事件包"典型） |
| `prefabs/quagmire_grill.lua` | json regs 为空＝抽取遗漏；实际 4 注册（MakeGrill L263-268）；fn/itemfn master_postinit 全缺；本地烟/炭火 FX 自足。 | blocked（事件包；json 缺口） |
| `prefabs/quagmire_oven.lua` | `quagmire_oven`(216)＋`quagmire_oven_back`(217 自足)＋`_item`(218)；back 由 master_postinit 生成；烟囱火/蒸汽自足。 | blocked（事件包） |
| `prefabs/quagmire_plantables.lua` | json 只记 7 条、实际 **34 注册**（7 产品×planted/leaf/raw/cooked＋soil 前/后＋seeds×7）；播种/生长全缺。附注：quagmire_kitchen.lua:4895-5189 有 `quagmire_potato_planted` 真实 worldgen 生成点，但 postinit L189 在事件包，无完整行为仍 blocked。 | blocked（事件包；中置信限定；json 缺口） |
| `prefabs/quagmire_portal.lua` | `quagmire_portal`(155)＋`quagmire_portal_activefx`(156 自足)＋`_bubblefx`(157 自足 Hidden)；FX 生成在缺失 master_postinit。附注：portal 本体有快照 worldgen 生成点（quagmire_kitchen.lua:119）；`quagmire_portal_player_fx` 等 3 名是 fx.lua:1715-1731 另族、已有页，不属本文件。 | blocked（事件包；中置信限定） |
| `prefabs/quagmire_pot_hanger.lua` | `quagmire_pot_hanger`(129)＋`_item`(130)；悬挂态由 item 部署转化、容器/燃料逻辑缺；蒸汽 FX 自足。 | blocked（事件包） |
| `prefabs/wx78_taser_projectile.lua` | `wx78_taser_projectile_fx`(312)：生成点 components/wx78_taserbuildup.lua:121＋SetFXOwner；组件由 wx78_moduledefs.lua:691-692 taser 模块挂载。联网主实体 shock_fx_low 常驻＋net_bool showbase/baseradius；爆炸态脱父挂定爆点、crackle_hit＋贴地 crackle_projection 环；FX 自带 combat（TASER_BUILDUP_DAMAGE=50 电击，伤害经 owner combat:GetAttacked 归属宿主玩家，命中窗 18 帧）；po 无 NAMES 键、死亡通告 SetPrefabNameOverride("wx78")。 | needs-page（1 个混合 FX，组 50） |
| `prefabs/yotr_fightring.lua` | `yotr_fightring`(1105)、`yotr_fightring_kit`(1106)、`yotr_fightring_torch`(1107)、`yotr_fightring_bell`(1108)＋placer(1109，json 漏记)；生成点全在快照（kit deploy L730、torch L528、bell L556、recipes.lua:1120）；torch 为非持久 FX 壳（事件驱动 turnon→torchfire_yotrpillowfight Follower "wick"＋AttachLightTo、turnoff→Remove）；官方名：枕头大战坑/钟/套装/兔人火炬（po 353064-353079）。 | needs-page（全族建页，组 51） |

工厂自足可建页素材（均无快照生成点，留待独立任务，不纳入本轮 51 组）：`lavaarena_firebomb_sparks`、`lavaarena_fossilizedebris×4`、`lavaarena_meteor_splashbase`、`peghook_spitfx/splashfx`、`lavaarena_keyhole/portal_activefx`、`quagmire_portal_activefx/bubblefx`、`quagmire_cooking_buff`、`quagmire_planted_soil_front/back`、`quagmire_<p>_leaf×7`、grill/oven/pot_hanger 本地烟炭火 FX。json 抽取缺口警示：`quagmire_grill`（json 0 条、实际 4）、`quagmire_plantables`（7/34）、`yotr_fightring`（漏 placer）、`quagmire_coins`（漏拼名展开）、`lavaarena_fossilizing`（漏 4 debris）——按 json 工作需回源码补齐。

**54 个未提及候选分诊至此全部完成**（21＋16＋17 文件三节）：blocked 15 文件、excluded 26 文件（21 文件节 13＋16 文件节 13）、needs-page 新增 13 文件（8＋3＋2，见组 41-51）。合并全口径：380 候选＝清单内闭环 **187**＋needs-page **101 文件**（51 页组）＋blocked **26 文件**（旧 11＋事件组 15）＋excluded **66 文件**（旧 40＋新增 26），四数合计 380。

代理对 CSV 数据行 1～140 完成只读初筛，尚未逐名完成消费链、现页语义复核或验收，均不得计入已完成文件。初筛发现下列待覆盖线索（文件数、注册数与页面数不等价）：

- 第 1～20 行：阿比盖尔反击／减益表现、花的五个动画、深渊柱、酸雨烟、天体激光／裂隙／陷阱／裂隙爆发、护符光、档案机关声画及月亮／虚布／WAGpunk 装备光效。`alterguardian_laserempty` 等有玩法的控制器、吊灯与底座等混合对象应按工厂职责复核，不可仅凭 FX 标签或是否伤害排除。
- 第 21～40 行：远古大门声画、灯光、气球表现、蝙蝠装备与武器效果、熊皮袋霜气、伯尼火、主教瞄准／充能等；`backpack_swap_fx` 已由 `reference/prefabs/backpack.md` 承载，仍待正式对账。`balloonparty_confetti_cloud`、`bishop_charge2_fx` 兼玩法，保留待核验。
- 第 41～60 行：龙舟竞赛旗帜与指示、月亮炸弹、书光、`books.lua` 的 12 个动态书面 FX、荆棘、工具碎裂、焦地、篝火、洞穴雨和光源、角色火与查莉特效；高尔夫道具及放置预览不因含未注册子效果自动成为新增 FX 注册。混合设施和 `caverain` 的既有机制部分覆盖须复核。
- 第 61～80 行：查莉召唤物、投射物、试炼／藤条、骨刺；切斯特影子／光、诱饵、盘旋秃鹫、冷火、蟹王爪、黑暗眼睛、月蛾光、湿痕等。`charlie_boss_minion1/2` 为动态注册且兼伤害，试炼、藤条、诱饵、冷火等兼玩法，保留待核验。不能将组件页提及视为完整覆盖。
- 第 81～100 行：梦魇疯猪埋藏与碎裂、鹿与激光、疾病蝇群、爆裂、眼球充能／火焰、风车、栅栏动画与电场、火焰和熔炉投射物／命中特效；混合伤害、发光和事件数据包缺口待复核。
- 第 101～120 行：火环、灭火器光、鱼网、灯笼、力场、化石底层、寒冰吐息、椅背动态注册、附身暗影炸弹、石像鬼动态特效、黏液、启迪者、瞄准器及黏液溅射等；主体、放置器及未注册子实体需按工厂分别判断。
- 第 121～140 行：洞穴水池声音、震地圆环、雷神锤、帽子动态跟随与灯光、治疗杖、隐士房装饰、蜂蜜轨迹、猎犬警告动态四级、哈奇光、冰洼与击中特效；暖石灯由既有宿主页承载。帽子各具体工厂、混合蜂蜜轨迹及事件包缺口尚待逐名复核。
- 第 141～160 行为部分初筛：石化碎裂动态注册、熔炉事件包动画、暗影物品与小浣猫表现等；灯柱动态名、事件包 postinit 与混合对象仍未展开，整段保持 pending。
- 第 161～180 行为工厂初筛：火焰工厂、天气与月陨粒子、激光／投射物混合对象，以及动态怪物警告音等；已有页面的调用提及不代表完整工厂覆盖。
- 第 181～200 行为工厂初筛：月坛与月暴、蘑菇树燃烧动画、噩梦裂隙、黏液颗粒、钓具水纹与海上暗影水纹等；动态注册与设施／控制器的边界尚待逐名复核。以上未完成消费链、建页和独立复审，不计外围闭环。
- 第 201～240 行为工厂初筛：海树、枕头、光束、雨、暴食物件及瞄准图形等；事件包、动态注册和混合对象仍待逐名复核。
- 第 241～260 行为工厂初筛：岩石龙虾、绳桥、鞍、暗影武器／裂隙等；玩法对象与内部未注册子实体的边界仍待复核。均未完成覆盖、排除或阻塞结论，不计外围闭环。
- 第 261～300 行为工厂初筛：寄生体、暗影召唤物、鲨鱼隧道、孢子、弹弓、雪人、雷声、奔雷矛与法杖光效等；`sparks` 与 `weaponsparks` 的既有覆盖另见上表，混合玩法及事件包缺口仍待逐名核验。
- 第 301～340 行为工厂初筛：雷声、火把、宝库特效、虚布装备、WAGboss／无人机、水植与海浪等；动态工厂、碰撞伤害及内部非注册子实体仍待细查。
- 第 341～380 行为工厂初筛：海岸波浪、温蒂墓碑、薇诺娜设备、冬季盛宴、沃姆伍德／沃拓克斯／沃特／WX78 及年节设施等；动态注册、附带玩法和宿主页部分引用均须逐名复核。以上三段不计外围闭环。

## 批次检查点

| 批 | 范围 | 状态 | 记录 |
|---|---|---|---|
| 1 | F01（leafpile 六注册，优先）＋F02＋F03 | 完成 | 4 页 15 注册（leafpile_spawn_fx／halloween_firepuff_1／pumpkincarving_shatter_fx＋snowball_shatter_fx 拆页）：独立复审 P1×2（leafpile 生长路径无休眠门；月绒失败落点非使用者）＋P2×5 已修；validate_page --check-indexes 7 文件全过、strict 链接 0 错 0 警、5 段 Lua 探针通过、record 4 页 check 0 漂移、检索命中、diff --check 干净；fx-table-index 36 族 178 注册、父索引同步、leafpilemanager 补回链 |
| 2 | F04＋F05＋F06 | 完成 | 4 页 59 注册（ghostlyelixir_slowregen_fx 36／ghostflower_spirit1_fx 拆页 2／wathgrithr_spirit 14／wanda_attack_pocketwatch_old_fx 7）：勘误 F04 属温蒂／阿比盖尔内容（bernie 零关联）；独立复审 P1×2（玩家 drip build=player_vial_fx；drinkelixir 仅前 4 帧 busy）＋P2×7 全部修复。7 文件 validate --check-indexes 全过（fx-table-index 超 16KiB 已拆出 fx-table-world-index 环境与物件叶级索引：40 族=角色战斗 17 族 133＋环境物件 23 族 104）、strict 链接 0 错 0 警、3 段 Lua 探针通过、record 4 页 check 0 漂移、检索命中、diff --check 干净；父索引改两行登记 |
| 3 | F07＋F08＋F09＋F10 | 完成 | 6 页 25 注册（attune_out_fx／beefalo_transform_fx 拆页／ocean_splash_med1／splash_green／splash_sink／round_puff_fx_sm）：勘误修正两个写作者误认（attune 是复活调谐非月相；快照实有 scripts/languages/chinese_s.po）；独立复审 P1×5（三页「快照无 languages」失实陈述＋「狼女／拳套」两处宿主指认错误）＋P2×6 全部修复。9 文件 validate --check-indexes 全过、strict 0 错 0 警、Lua 探针通过（写作者自检）、record 6 页 check 0 漂移、检索命中、diff --check 干净；两个叶级索引计数同步（角色战斗 21 族 151／环境物件 25 族 111） |
| 4 | F11＋F12＋F13 | 完成 | 4 页 24 注册（boat_grass_erode 7／fx_boat_crackle 5 拆姊妹页／boat_mast_sink_fx 8／cannonball_used 4）：独立复审 P1×4（spawnnewboatleak 事件源漏 14 处补全 17；modmain 示例 SpawnPrefab→GLOBAL.；drill 落点 y 论据；冰船时间线 9→10 处合计 11 枚）＋P2×12 全部修复（含 .po sources 补齐、fx_ice_pop 同包辨析、AddSoundEmitter 幂等标注引擎行为、mast 页字节超限精简回 16275B）。8 文件 validate --check-indexes 全过、strict 0 错 0 警、4 段 Lua 探针通过、record 4 页 check 0 漂移、检索命中、diff --check 干净；boatleak 组件页补真实回链；两叶级索引计数同步（角色战斗 22 族 155／环境物件 28 族 131） |
| 5 | F14＋F15＋F16＋F17 | 完成 | 5 页 20 注册（ink_puddle_land 4／flotsam_puddle 2／merm_splash 3／bee_poof_big 4／crab_king_bubble1 7）：独立复审 P1×3（蟹王气泡中心是海堆非本体、炮击冒泡重排是死链每堆只出一枚、waterspout 击飞仅限 oceanfishable 鱼类）＋P2×9 全部修复；锚点清理代理把三页 24 条 source-structural 警告按 name 行起点约定清零。8 文件 validate --check-indexes 全过、strict 0 错 0 警、record 5 页 check 0 漂移、检索命中、diff --check 干净；cannonball_used 补 waterspout 反链；两叶级索引计数同步（角色战斗 25 族 169／环境物件 30 族 137） |
| 6 | F18＋F19＋F20 | 完成 | 5 页 27 注册（sinkhole_spawn_fx 7／glass_fx 3 归档页／quagmire_portal_player_fx 5 事件数据包缺口／lavaarena_portal_player_fx 6／firesplash_fx 6）：勘误 fire 系实为蜻蜓龙、shadowstrike 实为麦斯威尔召唤物；独立复审 P1×1（firesplash 每环一枚→逐环逐点；后续复核按蜻蜓龙配置纠正为单次最多 8 枚）＋P2×11 全部修复（farm_plow 被接管、fossilspike2 计时起点、ErodeAway 归属 simutil、glass_fx 时序、MakeMinion 名单、.po 键归属、sources 补齐、尾链）。7 文件 validate --check-indexes 全过、strict 0 错 0 警、4 段 Lua 探针通过、record 5 页 check 0 漂移、检索命中、diff --check 干净；两叶级索引计数同步（角色战斗 28 族 186／环境物件 32 族 147）。首派三代理全部 600s 超时无产出，重派后一次成功 |
| 7 | F48＋F47＋F26 | 完成（**本会话末批，批 8 起交接**） | 3 页 27 注册（fx_book_moon complex 18：路径 A/B 两链 11 本 def；winters_feast_depletefood 2：首派被停代理已写入、本批代理独立核验通过；carnival_confetti_fx 7）：独立复审 P1×1（fx_book def 表缺 book_rain 行）＋P2×3（sources 补 3 文件、第 4 处声明锚、MakeFX 锚点偏移）全部修复；winters 页由独立代理按源码核验通过。5 文件 validate --check-indexes 全过、strict 0 错 0 警、3 段 Lua 探针通过、record 3 页 check 0 漂移、检索命中、diff --check 干净；两叶级索引计数同步（角色战斗 30 族 211／环境物件 33 族 149）。累计 63 族 360／504 通用表注册，剩 144 名（F21～F25、F27～F46）＋外围 380 文件核对 |
| 8 | F21～F25（18 注册） | 完成 | 5 页 18 注册；独立复审 P1×0、P2×6 已修；8 文件页面及导航通过，strict 72 引用 0 错 0 警，5 页 record 后 29 组 0 漂移；无 Lua 示例，未运行游戏。角色战斗 31 族 215 注册，环境物件 37 族 163 注册，合计 378／504，剩 126 名。 |
| 9 | F27＋F28＋F29＋F31（26 注册） | 完成 | 4 页 26 注册；独立复审 P1×0、P2×4 已修；7 文件页面及导航通过，strict 109 引用 0 错 0 警，4 页 record 后 49 组 0 漂移；无 Lua 示例，未运行游戏。角色战斗 33 族 228 注册，环境物件 39 族 176 注册，合计 404／504，剩 100 名。 |
| 10 | F30＋F32～F35（30 注册） | 完成 | 5 页 30 注册；独立复审 P1×1、P2×6 已修；10 文件页面及导航通过，strict 229 引用 0 错 0 警（另跳过 1 个工具不支持链接），5 页 record 后 40 组 0 漂移；无 Lua 示例，未运行游戏。角色战斗 37 族 243 注册，环境物件 40 族 191 注册，合计 434／504，剩 70 名。 |
| 11 | F36～F41（39 注册） | 完成 | 6 页 39 注册；独立复审修复地块判断、海冰静默、箱子／雪人／废料门禁、沃特抵伤、沃拓克斯选名、阿比盖尔资源与调用缺口。10 文件页面与导航通过，strict 118 引用 0 错 0 警，6 页 record 后 47 组 0 漂移；无 Lua 示例，未运行游戏。按索引大小拆出角色与装备补充叶级索引；共 83 族 473／504 注册，剩 31 名。 |
| 12 | F42～F46（31 注册） | 完成 | 5 页 31 注册，独立复审 P1×0、P2×1 已修；9 文件页面及导航通过，strict 116 引用 0 错 0 警，5 页 record 后 59 组 0 漂移；无 Lua 示例，未运行游戏。通用表 88 族 504／504；外围与全量统一检查仍待完成。 |
| C | 外围：stafflight／lightning／staffcastfx（＋thunder_close／thunder_far／moonstorm_lightning 扩页） | 完成 | 6 文件 3 页 17 注册；独立复审 P1×0、P2×3、P3×9 已修；8 文件页面／导航通过、strict 166 引用 0 错 0 警（跳过 2 个不支持链接）、3 页 record 后 35 组 0 漂移；无 Lua 示例、未运行游戏。fx-index 拆出 fx-world-index，父索引两行登记。 |
| D | 外围：瞄准圈家族＋天气降水＋海面波浪＋施法光源 | 完成 | 19 文件 4 页 64 注册；独立复审 P1×1、P2×6、P3×12 已修；4 页页面／导航通过、strict 266 引用 0 错 0 警、record 后 90 组 0 漂移；reticule 与 wave 各 1 段 Lua 解析通过；未运行游戏。 |
| E | 外围：火焰烟雾＋光源实体＋威胁预警 | 完成 | 30 文件 3 页 57 注册（fire 15／cavelight 13／houndwarning 29）；独立复审 P1×1、P2×9、P3×12 已修；3 页页面／导航通过、strict 235 引用 0 错 0 警、record 后 104 组 0 漂移；未运行游戏。 |
| F | 外围：宿主附属层＋远古吊灯／石像鬼＋熔炉化石化碎裂 | 完成 | 10 文件 3 页 27 注册；独立复审 P1×1、P2×3＋P3 若干已修；3 页页面／导航通过、strict 238 引用 0 错 0 警、record 后 40 组 0 漂移；未运行游戏。 |
| G | 外围：装备跟随光效＋天体英雄激光＋藏影女王战斗 | 完成 | 13 文件 3 页 22 注册；独立复审 P1×0、P2×4＋P3 若干已修；3 页页面／导航通过、strict 276 引用 0 错 0 警、record 后 31 组 0 漂移；未运行游戏。 |
| H | 外围：冲击爆点＋巨鹿激光＋贴地贴水 | 完成 | 11 文件 3 页 15 注册；独立复审 P1×0、P2×3＋P3 若干已修；3 页页面／导航通过、strict 192 引用 0 错 0 警、record 后 38 组 0 漂移；未运行游戏。 |
| I | 外围：残骸植物＋实验台粒子＋月装置 | 完成 | 12 文件 3 页 16 注册；独立复审 P1×1、P2×9＋P3 若干已修；3 页页面／导航通过、strict 221 引用 0 错 0 警、record 后 42 组 0 漂移；未运行游戏。 |
| L | 外围：地面贴片＋宿主挂接＋灯具光源 | 完成 | 22 文件 3 页 29 注册；独立复审 P1×0、P2×5＋P3 若干已修；3 页页面／导航通过、strict 321 引用 0 错 0 警、record 后 46 组 0 漂移；未运行游戏。 |
| M | 外围：附着跟随＋小型一次性物件＋有表现载体 | 完成 | 15 文件 3 页 18 注册；独立复审 P1×1、P2×6＋P3 若干已修；3 页页面／导航通过、strict 262 引用 0 错 0 警、record 后 67 组 0 漂移；未运行游戏。 |
| N | 外围：命中落点＋传送门投影＋碎屑皮肤 | 完成 | 8 文件 2 页 9 注册；独立复审 P1×0、P2×5＋P3 若干已修；2 页页面／导航通过、strict 165 引用 0 错 0 警、record 后 32 组 0 漂移；未运行游戏。 |
| O | 外围：皮肤火星＋地面痕迹＋粒子降水 | 完成 | 9 文件 3 页 9 注册；独立复审 P1×0、P2×4＋P3 若干已修；3 页页面／导航通过、strict 174 引用 0 错 0 警、record 后 48 组 0 漂移；未运行游戏。 |
| P | 外围：阿比盖尔子特效＋海树表现＋怀表传送门 | 完成 | 8 文件 3 页 20 注册；独立复审 P1×4、P2×6＋P3 若干已修；3 页页面／导航通过、strict 239 引用 0 错 0 警、record 后 35 组 0 漂移；未运行游戏。 |
| Q | 外围：影柱法术＋梦魇鞍具 | 完成 | 3 文件 2 页 6 注册；独立复审 P1×0、P2×6＋P3 若干已修；2 页页面／导航通过、strict 125 引用 0 错 0 警、record 后 22 组 0 漂移；未运行游戏。 |
| R | 外围：睡眠云＋装备持握层 | 完成 | 7 文件 2 页 11 注册（表头旧写"4 文件"有误，按分节 7 行计）；独立复审 P1×2、P2×5＋P3 若干已修；2 页页面／导航通过、strict 191 引用 0 错 0 警、record 后 35 组 0 漂移；未运行游戏。 |
| S | 外围：影陷阱＋暗影消散＋散件贴地 | 完成 | 8 文件 3 页 10 注册；独立复审 P1×0、P2×6＋P3 若干已修；3 页页面／导航通过、strict 237 引用 0 错 0 警、record 后 69 组 0 漂移；验收详见外围批 S 验收段（本行旧"生成（待独立复审）"与验收段冲突，以验收段为准）。 |
| T | 外围：书页挂载＋黏液飞溅＋暗影光源 | 完成 | 5 文件 3 页 29 注册；独立复审 P1×0、P2×0；3 页页面／导航通过、strict 206 引用 0 错 0 警、record 后 check 0 漂移、2 段 Lua 语法通过；未运行游戏。 |
| 10+ | F21～F46 及外围核对（按台账顺序滚动） | 未开始 | |

每批收尾在本表更新状态，并把该批族行状态改 accepted（含验收命令结果摘要）；排除/阻塞写"外围候选"节或对应行。

## 交接检查点（2026-11-01，批 12 收口时；历史）

- **当时进度**：通用表 504 名已覆盖 504（既有 163＋本轮批 1～12 共 341）；逐名全量统一对账与外围候选核对仍待完成。
- **索引**：fx-table-index（角色与战斗 38 族 246）、fx-table-world-index（环境与物件 44 族 223）及 fx-table-character-index（角色与装备补充 6 族 35）；父索引三行登记。
- **基线**：source-baseline.json 由 record 生成器维护，本会话所有新页已 record 且 check 0 漂移。
- **工作区**：全部改动未提交、未推送（按任务约束）；git status 中 untracked 的新页与 M 的共享文件即本轮交付。
- **续作方式**：按 PLAN"通用表后续扩库"项与本台账逐批循环"派发→写页→独立复审→修复→验收→登记→record→台账/PLAN/日志"，外围 380 文件核对与全量统一检查见台账对应节。

### 全量统一检查进度

- 只读 `fx_ledger_scan.py`：504 名全部有别名覆盖；其中 501 名同时在正文以完整字面名命中，`slow_steam_fx2/3/4` 仅命中别名，已核对 `crater_steam_fx1.md` 的 1～5 编号范围及 fx.lua 动态循环。
- `fx_ledger_verify.py`：48 行共 341 个名字且无重复；该脚本读取 `output/tmp/fx_ledger_scan_out.txt` 旧扫描产物，其 missing／extra 比较只反映历史剩余集合，不能当当前全量验证。
- 整库 `validate_page --check-indexes`（2026-11-04 外围批 E 后检查点）：1059 页、0 失败、导航通过（33 索引／1026 知识页）。整库 `validate_source_links --strict`：23690 引用、0 错、27 警（10 起点空行、11 终点空行、6 结构行；均为存量警告，新增页 0 警），985 项工具不支持的链接跳过；严格模式因此未通过。`git diff --check` 干净。
- 独立逐名对账：当前 `fx.lua` 去注释后 453 个静态名，六类动态循环展开 51 名，与 CSV 的 504 个唯一名集合一致；48 族 341 名均 accepted 且行计数正确。三个叶级索引分别为 38 页／246 名、44 页／223 名、6 页／35 名，与父索引一致；每个注册在已登记页面别名中恰有一处，整个 Prefab 分类的注册名跨页重复别名为 0（2026-11-04 复核仍成立；20 组重复均为中文描述性别名）。96 个正文无完整字面名的变体逐组核对正文范围／公式及锚点。扫描字符串不单独证明行为正确，未重做已有页全文技术复审或游戏运行。
- 口径勘误（分诊批 5 发现）：候选清单中 15 条 `prefabs/*.lua` 路径在本 buildid 不存在（`opalstaff`、`paperlantern`、`pig_house`、`pillow_fight`、`portable_sail`、`powder_monkey`、`rabbit_house`、`raft`、`reflective_armor`、`reindeer`、`researchlab`、`rock_avocado`、`rose`、`rotator`、`rowboat`），其中 `opalstaff`→`staff.lua`、`pig_house`→`pighouse.lua`、`powder_monkey`→`powdermonkey.lua`、`rabbit_house`→`rabbithouse.lua`、`researchlab`→`scienceprototyper.lua`、`rock_avocado`→`rock_avocado_bush.lua`/`rock_avocado_fruit.lua`、`rotator`→`boat_rotator.lua`/`fence_rotator.lua` 可映射到真实注册，其余 8 条无对应实体；这些路径按 blocked（清单勘误）计入，不计为未闭环 FX 文件。
- 历史检查点（2026-11-01）：当时尚需 232 个未闭环文件、批 J～M 复审及最终报告；后来“191 个未闭环、批 S 待复审”也已过时。当前续跑以上方「阶段交接（2026-11-05 待建清单复核）」及页组状态为准。

**第 02 轮 R14 与第 15 轮 R29／R30 补记（2026-11-07）**：attune_out_fx 的摘要、生成者表与 `fx-table-index` 第 35 行统一为“新源为复活调谐类型且 `_isrezattuned` 为 false 时播 out 并置 true，不查此前实际持有”，同 tag 静默限定进入前标志为 true；第 15 轮 R29 给 trap／boatrace／shadow_bishop／moon_altar 补 `entityscript.lua` 与 `mainfunctions.lua` 锚点并 record 四页；R30 把 11 个实质修改页 `updated` 改 `2026-11-07` 并补验证记录，wendy 未改保持原日期。attune＋索引 2/2、strict 27 引用 0 错 0 警；11 页 11/11、strict 622 引用 0 错 0 警、565 唯一目标、89 源文件、88 组来源 unchanged。未运行游戏、未改 raw、未提交推送。
