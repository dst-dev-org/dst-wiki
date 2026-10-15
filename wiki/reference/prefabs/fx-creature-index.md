---
title: "FX · 生物、Boss 与遭遇演出"
created: "2026-11-04"
updated: "2026-11-07"
type: index
tags: [overview]
---

# FX · 生物、Boss 与遭遇演出

Boss 技能、登场、攻击、预警，及普通生物动作与变体。

## Boss 登场、场地与大型攻击

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/alterguardian_summon_fx|天体英雄召唤光柱]] | 召唤警示光柱；endloop 收尾与双层灯光 | — / active |
| [[reference/prefabs/alterguardian_spintrail_fx|天体英雄与月脉冲特效]] | 7 注册：天体英雄破刺、旋转拖尾、坠落、陷阱消散及月脉冲 | alterguardian_spike_breakfx、alterguardian_phase1fallfx、alterguardian_phase3trappst、moon_geyser_explode、moonpulse_fx、moonpulse2_fx / active |
| [[reference/prefabs/alterguardian_laserscorch|天体英雄激光、灼烧与地面特效]] | 8 注册：激光焦痕／烟尾／命中加色、裂隙与超新星灼烧、陷阱地圈、砸地环与喷发 | `alterguardian_laserhit`、`alterguardian_phase4_lunarrift_slam_fx` 等 / active |
| [[reference/prefabs/eyeofterror_arrive_fx|恐怖之眼降临传送门]] | Boss 入场光门；帧插值灯光与动画退场 | — / active |
| [[reference/prefabs/charlie_boss_aoe_flame_fx|藏影女王 Boss 战表现特效族]] | 7 注册：落焰、小鬼命中和反射／弹体命中；三个混合载体写清伤害归属 | `charlie_boss_minion_hit_fx`、`charlie_boss_projectile`、群聚恐怖 / active |
| [[reference/prefabs/wagboss_beam_fx|WAGboss 轨道打击与机械装置表现]] | 4 注册：轨道光束（月灼混合载体）、机械腿超新星阻挡标记、瓦匠剪影与月亮精华提取器（混合） | `wagboss_beam_fx`、`wagboss_robot_leg_fx`、`alterguardian_contained` 等 / active |
| [[reference/prefabs/daywalker2_swipe_fx|拾荒疯猪挥击地面特效]] | 疯猪反手挥击；临时伤害由宿主 SG 结算 | `daywalker2_fx` / active |
| [[reference/prefabs/rocky_snip_fx|岩甲 Boss 攻击特效与枯萎变体表现]] | 4 注册：SG 生成的剪击弧与冲刺残影（伤害留宿主状态机）、枯萎 Boss 的双肩火层与 fade 层、蠕虫地面裂纹（宿主单槽引用管理） | `rocky_snip_fx`、`rocky_dash_fx`、`rocky_boss_shadow`、`worm_boss_dirt_ground_fx` / active |
| [[reference/prefabs/vault_pillar_guard_swipe_fx|远古戍卫塔挥击与砸地特效]] | 戍卫塔挥击与双层砸地；首帧对时 | `vault_pillar_guard_smash_fx`、`vault_pillar_guard_fx` / active |
| [[reference/prefabs/sharkboi_swipe_fx|大霜鲨攻击冰面特效族]] | 大霜鲨五种攻击表现；随机变体与定时收尾 | `sharkboi_iceplow_fx`、`sharkboi_iceimpact_fx`、`sharkboi_icetrail_fx`、`sharkboi_icehole_fx` / active |
| [[reference/prefabs/firesplash_fx|龙蝇火系与暗影斩击特效（firesplash_fx 等）]] | 6 注册：龙蝇三状态火焰（onexit 移除吐息代理）、重击火环单次最多 8 枚；暗影突袭斩击二选一 | tauntfire_fx、attackfire_fx、vomitfire_fx、shadowstrike_slash_fx、shadowstrike_slash2_fx、龙蝇火焰特效、蜻蜓龙火焰特效、重击火环、暗影斩击、暗影突袭特效 / active |
| [[reference/prefabs/crab_king_bubble1|蟹王伴随表现特效族（crab_king_bubble1）]] | 7 注册：气泡、水柱、宝石槽符号绑定代理、施法圆环与无活跃生成点的冰晶；伤害归生成者 | crab_king_bubble2、crab_king_bubble3、crab_king_waterspout、crab_king_shine、crab_king_icefx、crabking_ring_fx、蟹王气泡、蟹王水柱、水面喷泉特效、镶嵌宝石闪光、施法圆环 / active |
| [[reference/prefabs/crabking_claw_shadow|地面贴片与投影阴影 FX 族]] | 6 注册：巨钳水下阴影／挥击弧、冰圈真实冻结、焦痕淡出、柱基座／裂口 | `crabking_claw_swipe_fx`、`deerclopseyeball_sentryward_fx`、`fused_shadeling_bomb_scorch` 等 / active |
| [[reference/prefabs/fossilspike_base|化石骨刺地面基座层]] | 2 注册：Stalker 两代尖刺的引擎父挂基座（本体混合结构辨析、两代收尾不对称） | `fossilspike_base`、`fossilspike2_base` / active |

## 生物攻击、变身与预警

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/deerclops_icespike_fx|独眼巨鹿冰刺]] | 池化地面冰刺；伤害在宿主 SG | — / active |
| [[reference/prefabs/deerclops_icelance_ping_fx|晶体独眼巨鹿冰圈特效组]] | 巨鹿四种表现；冰圈加冷与落点伤害分工 | `deerclops_impact_circle_fx`、`deerclops_aura_circle_fx`、`deerclops_spikefire_fx` / active |
| [[reference/prefabs/deerclops_laserscorch|独眼巨鹿激光族]] | 冬季盛宴激光五注册；三个纯 FX（焦痕／烟尾／命中闪光）＋两个混合载体 | `deerclops_laser`、`deerclops_laserempty` 等 / active |
| [[reference/prefabs/deer_ice_circle|无眼鹿冰火法阵]] | 冰火法阵有玩法；冻结、减速、点燃与传播 | `deer_fire_circle`、`deer_ice_fx`、`deer_ice_burst`、`deer_ice_flakes`、`deer_ice_charge`、`deer_fire_burst`、`deer_fire_flakes`、`deer_fire_charge`、`deer_fx` / active |
| [[reference/prefabs/bearger_swipe_fx|熊獾挥击地面特效]] | 熊獾双变体挥击；Reverse 与宿主清理 | `mutatedbearger_swipe_fx`、`bearger_fx` / active |
| [[reference/prefabs/mossling_spin_fx|麋鹿鹅幼崽旋转电光]] | 幼崽旋转电光；声效计数与宿主落雷分工 | — / active |
| [[reference/prefabs/moose_nest_fx_idle|麋鹿鹅巢电弧特效]] | 麋鹿鹅巢电弧；网络壳与本地音画 | `moose_nest_fx_hit`、`moose_nest_fx` / active |
| [[reference/prefabs/warg_mutated_breath_fx|附身座狼喷焰与灼烬]] | 地面焰真实伤害；ConfigureDamage 前提与冰烬 | `warg_mutated_ember_fx` / active |
| [[reference/prefabs/weremoose_smash_fx|伍迪麋鹿形态重击特效]] | 麋鹿重击双层表现；客户端预测去重 | — / active |
| [[reference/prefabs/lunarmerm_thorns_fx|鱼人荆棘反伤与翻土标记]] | 鱼人荆棘真实伤害；扩张限次与翻土标记 | `merm_soil_marker`、`merm_fx` / active |
| [[reference/prefabs/minotaur_blood1|力量变身与远古守护者血滴]] | 4 注册：玩家 powerup 与守护者三档血滴 | wolfgang_mighty_fx、minotaur_blood2、minotaur_blood3 / active |
| [[reference/prefabs/ruinsnightmare_horn_attack|潜伏梦魇双角突袭]] | 1 注册：受击反击的双角投射物；宿主 combat 结算、每角独立去重、命中点转碎影并互删 | `ruinsnightmare_horn_attack` / active |
| [[reference/prefabs/houndwarning_lvl1|威胁预警与暗影窥视表现]] | 29 注册：猎犬／蠕虫／巨鹿／熊獾／坎普斯预警音、海盗与骑士号角、陨石预警与暗影窥视 | `wormwarning_worm_boss`、`piratewarningsound`、`meteorwarning`、`shadowskittish` 等 / active |
| [[reference/prefabs/bee_poof_big|蜂蜜飞溅、蜂群烟与病害蝇群烟（bee_poof 族）]] | 4 注册：蜂窝受击、护卫变阵与召蜂烟；disease_puff 未见生成 | honey_splash、disease_puff、蜂群烟、蜂蜜飞溅、蜂王护卫变身烟、病害蝇群烟 / active |

## 移动、随从与宿主附属

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/woby_dash_shadow_fx|沃比暗影冲刺特效]] | 沃比暗影冲刺表现；残影追踪与剪影清理 | `woby_dash_silhouette_fx` / active |
| [[reference/prefabs/lavae_move_fx|岩浆虫与哈奇移动拖尾]] | 岩浆虫／哈奇拖尾；变体去重与缩放量化 | `hutch_move_fx` / active |
| [[reference/prefabs/beargerfur_sack_frost_fx|熊獾桶霜气与伯尼火层]] | 2 注册：开桶霜气（复用 icebox_open_fx 唯一使用者）＋伯尼三组件火层（反射伤害在宿主 damagereflect） | `beargerfur_sack_frost_fx`、`bernie_big_fire` / active |
| [[reference/prefabs/hotcold_fx|小惊吓冷热寻物指示]] | 1 注册：按玩家到最近玩具的距离缩放；任务完成移除、放弃路径不清理 | `hotcold_fx` / active |

## 关联入口

暗影生物与影手见 [[reference/prefabs/fx-shadow-index|暗影生物、裂隙生物与附着]]。石化与碎裂素材见 [[reference/prefabs/fx-material-index|采集、制作、碎裂与生成烟]]。

返回 [[reference/prefabs/fx-index|FX 主题总索引]]、[[reference/prefabs/index|Prefab 索引]]。
