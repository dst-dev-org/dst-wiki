---
title: 通用表 FX 索引
created: "2026-10-30"
updated: "2026-11-01"
type: index
tags: [overview]
---

# 通用表 FX

`fx.lua` 通用注册表特效的同族知识页，按机制族分页、变体入表。本索引收角色与战斗表现；新增角色技能与装备补充见 [[reference/prefabs/fx-table-character-index|角色与装备补充]]，环境与物件见 [[reference/prefabs/fx-table-world-index|环境与物件]]。公共协议见 [[reference/mechanisms/generic-fx-factory|通用 FX 工厂]]；专属工厂见 [[reference/prefabs/fx-index|特效 Prefab 索引]]。返回 [[reference/prefabs/index|Prefab 索引]]。

## 角色与战斗（38 族 246 注册）

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/fallingswish_clouds|坠落云与线条]] | 3 注册：虚空坠落、深渊落下与沉没选名 | fallingswish_clouds_fast、fallingswish_lines / active |
| [[reference/prefabs/spell_fire_throw|火杖、月化宠物与薇洛火特效]] | 4 注册：火杖失败、宠物结束、暗影火爆裂与未见生成的施火条目 | fire_fail_fx、wormwood_lunar_transformation_finish、willow_shadow_fire_explode / active |
| [[reference/prefabs/dreadstone_spawn_fx|暗影融合体、恐惧石与诅咒特效]] | 4 注册：融合体出现、恐惧石变化、寄生附体和牛铃诅咒 | fused_shadeling_spawn_fx、shadowthrall_parasite_attach_poof_fx、beef_bell_shadow_cursefx / active |
| [[reference/prefabs/wagpunksteam_hat_up|WAGpunk 装备蒸汽]] | 4 注册：帽与护甲追踪阶段变化的上下蒸汽 | wagpunksteam_hat_down、wagpunksteam_armor_up、wagpunksteam_armor_down / active |
| [[reference/prefabs/wagdrone_rolling_collide_small_fx|滚动无人机碰撞与灵药护盾特效]] | 3 注册：碰撞火花及玩家灵药受击护盾表现 | wagdrone_rolling_collide_med_fx、elixir_player_forcefield / active |
| [[reference/prefabs/alterguardian_spintrail_fx|天体英雄与月脉冲特效]] | 7 注册：天体英雄破刺、旋转拖尾、坠落、陷阱消散及月脉冲 | alterguardian_spike_breakfx、alterguardian_phase1fallfx、alterguardian_phase3trappst、moon_geyser_explode、moonpulse_fx、moonpulse2_fx / active |
| [[reference/prefabs/dr_warm_loop_1|探测杖距离环与 WX 音乐特效]] | 6 注册：探测杖距离选环、音乐电路音符；旧蒸汽无生成点 | dr_warm_loop_2、dr_warmer_loop、dr_hot_loop、wx78_heat_steam、wx78_musicbox_fx / active |
| [[reference/prefabs/shadow_teleport_in|暗影传送、孢子咳出与宝盒玩家特效]] | 4 注册：暗影生物海陆交换、蘑菇地精咳孢子与宝盒授予特效 | shadow_teleport_out、spore_moon_coughout、archive_lockbox_player_fx / active |
| [[reference/prefabs/fx_book_moon|读书施法特效族（fx_book_moon 等）]] | 18 注册：读者落点与书本按受效对象两条生成链，骑乘回落与 fish 特殊工厂 | fx_book_moon_mount、fx_book_research_station、fx_book_research_station_mount、fx_book_temperature、fx_book_temperature_mount、fx_book_bees、fx_book_fire、fx_book_fire_mount、fx_book_light、fx_book_light_upgraded、fx_book_birds、fx_book_birds_mount、fx_book_rain、fx_book_rain_mount、fx_book_sleep、fx_book_sleep_mount、fx_book_fish、读书特效、施法特效 / active |
| [[reference/prefabs/carnival_confetti_fx|狂欢节庆典特效族（carnival_confetti_fx 等）]] | 7 注册：炮庆三连、按季选名的盲盒开包、射击投射物；彩纸另用于高尔夫与小鸡赛跑 | carnival_sparkle_fx、carnival_streamer_fx、carnival_unwrap_fx、carnival_unwrap_fx_s2、carnival_unwrap_fx_s3、carnivalgame_shooting_projectile_fx、狂欢节特效、狂欢节彩纸、彩带特效、开包特效 / active |
| [[reference/prefabs/quagmire_portal_player_fx|暴食远古大门玩家出场与撒盐特效族（quagmire_portal_player_fx）]] | 5 注册：大门出场、滴水、出水与撒盐盘碗；生成点在快照不含的事件数据包 | quagmire_portal_playerdrip_fx、quagmire_portal_player_splash_fx、quagmire_salting_plate_fx、quagmire_salting_bowl_fx、暴食传送门特效、暴食撒盐特效、盐盘特效、盐碗特效、远古大门特效 / active |
| [[reference/prefabs/lavaarena_portal_player_fx|熔炉登场、复活与场景余烬特效（lavaarena_portal_player_fx 等）]] | 6 注册：玩家登场、尸体复活、生物传送烟与格栅余烬；登场生成点在事件数据包 | lavaarena_player_revive_from_corpse_fx、lavaarena_creature_teleport_smoke_fx_2、lavaarena_creature_teleport_smoke_fx_3、ember_short_fx、熔炉传送门特效、尸体复活特效、出生烟、余烬特效 / active |
| [[reference/prefabs/firesplash_fx|蜻蜓龙火系与暗影斩击特效（firesplash_fx 等）]] | 6 注册：蜻蜓龙三状态火焰（onexit 移除吐息代理）、重击火环单次最多 8 枚；暗影突袭斩击二选一 | tauntfire_fx、attackfire_fx、vomitfire_fx、shadowstrike_slash_fx、shadowstrike_slash2_fx、蜻蜓龙火焰特效、重击火环、暗影斩击、暗影突袭特效 / active |
| [[reference/prefabs/crab_king_bubble1|蟹王伴随表现特效族（crab_king_bubble1）]] | 7 注册：气泡、水柱、宝石槽符号绑定代理、施法圆环与无活跃生成点的冰晶；伤害归生成者 | crab_king_bubble2、crab_king_bubble3、crab_king_waterspout、crab_king_shine、crab_king_icefx、crabking_ring_fx、蟹王气泡、蟹王水柱、水面喷泉特效、镶嵌宝石闪光、施法圆环 / active |
| [[reference/prefabs/merm_splash|鱼人水花（merm_splash 族）]] | 三注册六生成点：王诞生跨世界增益、月变异变、王诞与守卫召唤／回巢出退场，全部自身落点；音效由生成方播放，条目无 sound | merm_king_splash、merm_spawn_fx、鱼人水花、鱼人王水花、生成水花 / active |
| [[reference/prefabs/bee_poof_big|蜂蜜飞溅、蜂群烟与病害蝇群烟（bee_poof 族）]] | 4 注册：蜂窝受击、护卫变阵与召蜂烟；disease_puff 未见生成 | honey_splash、disease_puff、蜂群烟、蜂蜜飞溅、蜂王护卫变身烟、病害蝇群烟 / active |
| [[reference/prefabs/cannonball_used|船炮与迫击炮弹命中表现（cannonball_used）]] | 4 注册：船炮石弹落地无声动画与蟹王迫击炮弹三变体（同动画按命中面配声）；原版 dockmanager 恒在使落地恒选 _wood，mortarball.lua 撞墙生成点为死代码 | mortarball_used、mortarball_used_wood、mortarball_used_ice、炮弹命中特效、船炮命中表现、迫击炮命中表现 / active |
| [[reference/prefabs/round_puff_fx_sm|通用圆烟、滑行烟与木屑（round_puff 族）]] | 六注册：圆烟 sm/lg 标签选名（crabking 代理父挂并绑定符号）、slide_puff 四状态图、钻船木屑及 jump 无生成点 | round_puff_fx_lg、round_puff_fx_hi、slide_puff、wood_splinter_jump、wood_splinter_drill、圆烟、命中烟、滑行烟、木屑特效 / active |
| [[reference/prefabs/attune_out_fx|复活调谐得失特效（attune_out_fx）]] | 3 注册：调谐从无到有父挂 out，当前 tag 失去来源时按幽灵态选 in／ghost_in 父挂；同一 tag 换绑仅在标志 `_isrezattuned` 为 true 时静默，等价但不同 tag 互换会清标志、使下一次同 tag 换绑重播 out；加载重连与远程复活静默不出；墓碑挖除／恶花／装饰完成与温蒂墓碑合成四处落点复用 out | attune_in_fx、attune_ghost_in_fx、调谐特效、调谐得失特效、调谐建立特效、调谐解除特效、复活绑定特效 / active |
| [[reference/prefabs/beefalo_transform_fx|皮弗娄牛驯化变身特效（beefalo_transform_fx）]] | 完成驯化与已驯化后野性化共用变身状态；骑乘退出补枚 | 皮弗娄牛变身特效、驯化变身特效、牛变身特效 / active |
| [[reference/prefabs/ocean_splash_med1|海面水花特效族（ocean_splash_med1）]] | 8 注册：海鱼破水、浮标、暗影傀儡等水花与游泳尾流 | ocean_splash_med2、ocean_splash_small1、ocean_splash_small2、ocean_splash_ripple1、ocean_splash_ripple2、ocean_splash_swim1、ocean_splash_swim2、海面水花、游泳水花、水面涟漪、海面涟漪 / active |
| [[reference/prefabs/ghostlyelixir_slowregen_fx|幽灵灵药环绕与滴落特效（ghostlyelixir_slowregen_fx）]] | 36 注册：9 药效×阿比盖尔／玩家×环绕／滴落；buff 服务端生成，attach 父挂一次、drip 15 秒落点节流，四路停止清理；疗效在 buff 侧，玩家 shadow/lunar 无生成点 | ghostlyelixir_fastregen_fx、ghostlyelixir_shield_fx、ghostlyelixir_attack_fx、ghostlyelixir_speed_fx、ghostlyelixir_retaliation_fx、ghostlyelixir_revive_fx、ghostlyelixir_shadow_fx、ghostlyelixir_lunar_fx、ghostlyelixir_slowregen_dripfx、ghostlyelixir_fastregen_dripfx、ghostlyelixir_shield_dripfx、ghostlyelixir_attack_dripfx、ghostlyelixir_speed_dripfx、ghostlyelixir_retaliation_dripfx、ghostlyelixir_revive_dripfx、ghostlyelixir_shadow_dripfx、ghostlyelixir_lunar_dripfx、ghostlyelixir_player_slowregen_fx、ghostlyelixir_player_fastregen_fx、ghostlyelixir_player_shield_fx、ghostlyelixir_player_attack_fx、ghostlyelixir_player_speed_fx、ghostlyelixir_player_retaliation_fx、ghostlyelixir_player_revive_fx、ghostlyelixir_player_shadow_fx、ghostlyelixir_player_lunar_fx、ghostlyelixir_player_slowregen_dripfx、ghostlyelixir_player_fastregen_dripfx、ghostlyelixir_player_shield_dripfx、ghostlyelixir_player_attack_dripfx、ghostlyelixir_player_speed_dripfx、ghostlyelixir_player_retaliation_dripfx、ghostlyelixir_player_revive_dripfx、ghostlyelixir_player_shadow_dripfx、ghostlyelixir_player_lunar_dripfx、幽灵灵药特效、阿比盖尔灵药特效、灵药环绕特效、灵药滴落特效 / active |
| [[reference/prefabs/ghostflower_spirit1_fx|幽灵花灵体飘光（ghostflower_spirit1_fx）]] | 2 注册：幽灵花落地 3-9 秒周期飘光、墓碑花瓣安放固定 1 号；与灵药机制无关 | ghostflower_spirit2_fx、幽灵花灵体、灵体飘光 / active |
| [[reference/prefabs/wathgrithr_spirit|武神之魂与战歌特效（wathgrithr_spirit）]] | 14 注册：击杀英灵、战歌附着／退场及音符；electric 无生成点 | battlesong_attach、battlesong_loop、battlesong_detach、battlesong_durability_fx、battlesong_healthgain_fx、battlesong_sanitygain_fx、battlesong_sanityaura_fx、battlesong_fireresistance_fx、battlesong_shadowaligned_fx、battlesong_lunaraligned_fx、battlesong_instant_electric_fx、battlesong_instant_taunt_fx、battlesong_instant_panic_fx、武神之魂、战歌音符、战歌buff特效、瞬时战歌特效 / active |
| [[reference/prefabs/wanda_attack_pocketwatch_old_fx|旺达怀表攻击与治疗特效（wanda_attack_pocketwatch_old_fx）]] | 7 注册：年龄与武器选攻击特效、骑乘治疗及复活倒流 | wanda_attack_pocketwatch_normal_fx、wanda_attack_shadowweapon_old_fx、wanda_attack_shadowweapon_normal_fx、pocketwatch_heal_fx、pocketwatch_heal_fx_mount、pocketwatch_ground_fx、旺达攻击特效、警钟攻击特效、怀表治疗特效、复活倒流地面特效 / active |
| [[reference/prefabs/werebeaver_transform_fx|变身切换烟（werebeaver_transform_fx）]] | 形态切换烟 8 名：伍迪三形态变身、露西斧收编还原、切斯特／哈奇切换；lucy_transform_fx 被全玩家接住抛射物复用 | weremoose_transform_fx、weremoose_transform2_fx、weremoose_revert_fx、weregoose_transform_fx、lucy_transform_fx、lucy_ground_transform_fx、chester_transform_fx、变身烟、接住烟、变形特效 / active |
| [[reference/prefabs/weregoose_splash|鹅形态水花与羽毛（weregoose_splash）]] | 鹅形态水上移动 10 名：三档水花与羽毛水陆分流、非扑翼状态周期涟漪、起飞羽毛落点；海猴划船复用本名水花 | weregoose_splash_med1、weregoose_splash_med2、weregoose_splash_less1、weregoose_splash_less2、weregoose_ripple1、weregoose_ripple2、weregoose_feathers1、weregoose_feathers2、weregoose_feathers3、水上羽毛、涟漪特效 / active |
| [[reference/prefabs/oldager_become_younger_front_fx|年龄转换特效（oldager_become_younger_front_fx）]] | 旺达年龄档切换 6 名（三 anim×站立／骑乘）：healthdelta 重选档、年轻段前后两枚、骑乘后缀选资源、静默变身无表现；父挂旺达持续跟随 | oldager_become_younger_back_fx、oldager_become_older_fx、oldager_become_younger_front_fx_mount、oldager_become_younger_back_fx_mount、oldager_become_older_fx_mount、变老特效、变年轻特效 / active |
| [[reference/prefabs/slingshot_ice_aoe_fx|弹弓 AOE 命中圈（slingshot_ice_aoe_fx）]] | 元素 AOE 4 名快照内无生成点（诚实缺口）；实际表现走专属工厂 slingshot_aoe_fx 单体五色着色 | slingshot_slow_aoe_fx、slingshot_shadow_aoe_fx、slingshot_lunar_aoe_fx、slingshot_aoe_fx、弹弓AOE、命中圈特效 / active |
| [[reference/prefabs/slingshotammo_hitfx_rock|弹弓弹药命中表现（slingshotammo_hitfx_rock）]] | 弹弓命中 18 注册；按弹药覆符号／特定动画／光照，火药条目仅暴击时抑制常规命中特效 | slingshotammo_hitfx_gold、slingshotammo_hitfx_marble、slingshotammo_hitfx_thulecite、slingshotammo_hitfx_honey、slingshotammo_hitfx_freeze、slingshotammo_hitfx_slow、slingshotammo_hitfx_poop、slingshotammo_hitfx_moonglass、slingshotammo_hitfx_dreadstone、slingshotammo_hitfx_gunpowder、slingshotammo_hitfx_lunarplanthusk、slingshotammo_hitfx_purebrilliance、slingshotammo_hitfx_horrorfuel、slingshotammo_hitfx_gelblob、slingshotammo_hitfx_scrapfeather、slingshotammo_hitfx_stinger、slingshotammo_hitfx_trinket_1、弹弓命中特效 / active |
| [[reference/prefabs/shock_fx|电击附着动画（shock_fx）]] | 电击附着六注册；玩家形态四选一符号跟随、电击模块反击弧光、月暴电花自附，与 electrocute_fx 分轨 | shock_arc_fx、werebeaver_shock_fx、weremoose_shock_fx、weregoose_shock_fx、moonstorm_spark_shock_fx、电击附着、电弧特效 / active |
| [[reference/prefabs/planar_hit_fx|位面命中与抵抗表现（planar_hit_fx）]] | 位面攻防两注册；命中挂受击目标（仅 inventory 持有者）、抵抗环带落点，条件在 combat／planarentity | planar_resist_fx、位面命中表现、位面抵抗表现、位面偏转 / active |
| [[reference/prefabs/wx78_shield_full|WX78 护盾动画（wx78_shield_full）]] | 护盾六注册；常驻两态周期重播、四条转场父挂宿主，破盾 11 帧后脱挂 | wx78_shield_half、wx78_shield_full_to_half、wx78_shield_half_to_full、wx78_shield_full_to_empty、wx78_shield_half_to_empty、WX78护盾动画、破盾特效 / active |
| [[reference/prefabs/shadow_shield1|骨头盔甲格挡护盾（shadow_shield1）]] | 护盾六注册（三变体×水平镜像）；骨甲 resistance 吸收时 PickShield 按主盾冷却选名并父挂持有者 | shadow_shield2、shadow_shield3、shadow_shield4、shadow_shield5、shadow_shield6、格挡护盾、骨甲护盾 / active |
| [[reference/prefabs/spider_heal_fx|蜘蛛治疗与突变（spider_heal_fx）]] | 蜘蛛阵营 4 名：护士蜘蛛 30 帧群疗（地面圈＋本体＋目标光）、治疗黏团使用链复用三名、蜘蛛突变烟落点；治疗／突变声在调用方 | spider_mutate_fx、spider_heal_target_fx、spider_heal_ground_fx、蜘蛛治疗、护士蜘蛛、治疗黏团、蜘蛛突变 / active |
| [[reference/prefabs/maxwell_smoke|麦斯威尔烟（maxwell_smoke）]] | 麦斯威尔 talker 三分支：台词收尾两模式立即、普通出现 Sleep 后、取消延迟任务；消散声在 talker 分支 | 消散烟、maxwell 特效 / active |
| [[reference/prefabs/small_puff|变身与出生烟（small_puff）]] | 单注册通用烟团；变身 helper DoChangePrefab 标准开场烟，另有月变／移除／出生等约 24 文件直接落点 | 变身烟、出生烟、小烟团 / active |
| [[reference/prefabs/charlie_snap|暗影拍指（charlie_snap）]] | 两注册呈现链开场拍指；音效由数据 fn 延迟 21 帧自挂播放，升级或洞穴走实心变体 | charlie_snap_solid、暗影拍指、查莉拍指 / active |

## 尚未建页

通用表 504 个注册名均已有族页；本索引只登记实际成页内容。
