---
title: 特效 Prefab 索引·生物与Boss
created: "2026-11-04"
updated: "2026-11-05"
type: index
tags: [overview]
---

# 特效 Prefab 索引·生物与Boss

生物／Boss 战相关的特效知识页，按宿主与机制分组；环境与世界类见 [[reference/prefabs/fx-world-index|特效 Prefab 索引·环境与世界]]，角色技能、装备与杂项见 [[reference/prefabs/fx-index|特效 Prefab 索引]]。

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/beargerfur_sack_frost_fx|熊獾桶霜气与伯尼火层]] | 2 注册：开桶霜气（复用 icebox_open_fx 唯一使用者）＋伯尼三组件火层（反射伤害在宿主 damagereflect） | `beargerfur_sack_frost_fx`、`bernie_big_fire` / active |
| [[reference/prefabs/bat_bosshat_fx|蝙蝠Boss系表现]] | 5 注册：蝙蝠帽三层栖立小蝠（DECOR 标签异例）、分裂残影、蝙蝠棒皮肤代理＋本地播放双实体、惊树 classified 定向可见 | `bat_bosshat_fx`、`bat_boss_shadow_split_fx`、`batbat_bats`、`battreefx` 等 / active |
| [[reference/prefabs/bishop_targeting_fx|发条主教预警圈]] | 1 注册：dist netvar 双端光斑协议（专服也要光）、超时淡出/打断无 pst 两观感 | `bishop_targeting_fx` / active |
| [[reference/prefabs/itemmimic_revealed_shadow|拟态蠕虫影尾]] | 1 注册：单发射器恒定 1 粒的 VFXEffect、共享段非专服各建一份、net_event 开关协议 | `itemmimic_revealed_shadow` / active |
| [[reference/prefabs/stalker_shield1|远古织影者格挡盾]] | 5 注册：PickShield 分布表、FX 实体直接作 attacker 的 10 点反伤、基名诚实缺口 | `stalker_shield1`、`stalker_shield2`、`stalker_shield3`、`stalker_shield4`、`stalker_shield` / active |
| [[reference/prefabs/toadstool_cap_absorbfx|毒菌帽吸收与释放]] | 2 注册：中毒金丝雀爆毒触发链（非 Boss 状态机勘误）、releasefx 复用金丝雀爆炸资源 | `toadstool_cap_absorbfx`、`toadstool_cap_releasefx` / active |
| [[reference/prefabs/warg_mutated_breath_fx|附身座狼喷焰与灼烬]] | 地面焰真实伤害；ConfigureDamage 前提与冰烬 | `warg_mutated_ember_fx` / active |
| [[reference/prefabs/deerclops_icespike_fx|独眼巨鹿冰刺]] | 池化地面冰刺；伤害在宿主 SG | — / active |
| [[reference/prefabs/deerclops_icelance_ping_fx|晶体独眼巨鹿冰圈特效组]] | 巨鹿四种表现；冰圈加冷与落点伤害分工 | `deerclops_impact_circle_fx`、`deerclops_aura_circle_fx`、`deerclops_spikefire_fx` / active |
| [[reference/prefabs/mossling_spin_fx|麋鹿鹅幼崽旋转电光]] | 幼崽旋转电光；声效计数与宿主落雷分工 | — / active |
| [[reference/prefabs/weremoose_smash_fx|伍迪麋鹿形态重击特效]] | 麋鹿重击双层表现；客户端预测去重 | — / active |
| [[reference/prefabs/woby_dash_shadow_fx|沃比暗影冲刺特效]] | 沃比暗影冲刺表现；残影追踪与剪影清理 | `woby_dash_silhouette_fx` / active |
| [[reference/prefabs/lunarmerm_thorns_fx|鱼人荆棘反伤与翻土标记]] | 鱼人荆棘真实伤害；扩张限次与翻土标记 | `merm_soil_marker`、`merm_fx` / active |
| [[reference/prefabs/deer_ice_circle|无眼鹿冰火法阵]] | 冰火法阵有玩法；冻结、减速、点燃与传播 | `deer_fire_circle`、`deer_ice_fx`、`deer_ice_burst`、`deer_ice_flakes`、`deer_ice_charge`、`deer_fire_burst`、`deer_fire_flakes`、`deer_fire_charge`、`deer_fx` / active |
| [[reference/prefabs/bearger_swipe_fx|熊獾挥击地面特效]] | 熊獾双变体挥击；Reverse 与宿主清理 | `mutatedbearger_swipe_fx`、`bearger_fx` / active |
| [[reference/prefabs/daywalker2_swipe_fx|拾荒疯猪挥击地面特效]] | 疯猪反手挥击；临时伤害由宿主 SG 结算 | `daywalker2_fx` / active |
| [[reference/prefabs/sharkboi_swipe_fx|大霜鲨攻击冰面特效族]] | 大霜鲨五种攻击表现；随机变体与定时收尾 | `sharkboi_iceplow_fx`、`sharkboi_iceimpact_fx`、`sharkboi_icetrail_fx`、`sharkboi_icehole_fx` / active |
| [[reference/prefabs/moose_nest_fx_idle|麋鹿鹅巢电弧特效]] | 麋鹿鹅巢电弧；网络壳与本地音画 | `moose_nest_fx_hit`、`moose_nest_fx` / active |
| [[reference/prefabs/marionette_appear_fx|舞台剧木偶出场与退场特效]] | 舞台木偶出退场；SetTime 与渐隐 | `marionette_disappear_fx`、`marionette_fx`、`marionette` / active |
| [[reference/prefabs/vault_pillar_guard_swipe_fx|远古戍卫塔挥击与砸地特效]] | 戍卫塔挥击与双层砸地；首帧对时 | `vault_pillar_guard_smash_fx`、`vault_pillar_guard_fx` / active |
| [[reference/prefabs/alterguardian_summon_fx|天体英雄召唤光柱]] | 召唤警示光柱；endloop 收尾与双层灯光 | — / active |
| [[reference/prefabs/eyeofterror_arrive_fx|恐怖之眼降临传送门]] | Boss 入场光门；帧插值灯光与动画退场 | — / active |
| [[reference/prefabs/shadowthrall_projectile_fx|墨荒弹幕投射物]] | 墨荒弹幕真实伤害；落地 AOE 与共享去重 | — / active |
| [[reference/prefabs/brilliance_projectile_fx|亮茄魔杖辉光弹]] | 亮茄弹跳投射物；命中额度、闪爆与武器伤害 | `brilliance_projectile_blast_fx` / active |
| [[reference/prefabs/gelblob_attach_fx|恶液接触附着黏液]] | 黏液附着有减速；Connector 与共享注销 | — / active |
| [[reference/prefabs/fossilized_break_fx|化石碎裂与化石化特效]] | 四种化石表现；本地尘雾代理与碎裂淡出 | `fossilizing_fx`、`fossilizing_fx_1`、`fossilizing_fx_2` / active |
| [[reference/prefabs/petrified_trunk_break_fx|树木石化特效]] | 五种树木石化代理；阶段选择与灰度继承 | `petrified_tree_fx_short`、`petrified_tree_fx_normal`、`petrified_tree_fx_tall`、`petrified_tree_fx_old`、`petrify_fx` / active |
| [[reference/prefabs/houndwarning_lvl1|威胁预警与暗影窥视表现]] | 29 注册：猎犬／蠕虫／巨鹿／熊獾／坎普斯预警音、海盗与骑士号角、陨石预警与暗影窥视 | `wormwarning_worm_boss`、`piratewarningsound`、`meteorwarning`、`shadowskittish` 等 / active |
| [[reference/prefabs/archive_chandelier|远古吊灯与石像鬼石化特效]] | 8 注册：档案馆／宝库三盏吊灯＋石像鬼五石化特效；`vault_crawler_chandelier` 兼生成点 | `gargoyle_houndatk_fx` 等 / active |
| [[reference/prefabs/lavaarena_beetletaur_fossilized_break_fx_right|熔炉独眼巨猪与犀牛兄弟化石碎裂特效]] | 熔炉 7 注册化石碎裂；侧向隐藏与 `_alt` 中断变体；生成点在缺失的 fossilizable 组件／事件包 | `beetletaur_fossilized_break_fx_right` 等 / active |
| [[reference/prefabs/alterguardian_laserscorch|天体英雄激光、灼烧与地面特效]] | 8 注册：激光焦痕／烟尾／命中加色、裂隙与超新星灼烧、陷阱地圈、砸地环与喷发 | `alterguardian_laserhit`、`alterguardian_phase4_lunarrift_slam_fx` 等 / active |
| [[reference/prefabs/charlie_boss_aoe_flame_fx|藏影女王 Boss 战表现特效族]] | 7 注册：落焰、小鬼命中和反射／弹体命中；三个混合载体写清伤害归属 | `charlie_boss_minion_hit_fx`、`charlie_boss_projectile` 等 / active |
| [[reference/prefabs/deerclops_laserscorch|独眼巨鹿激光族]] | 冬季盛宴激光五注册；三个纯 FX（焦痕／烟尾／命中闪光）＋两个混合载体 | `deerclops_laser`、`deerclops_laserempty` 等 / active |
| [[reference/prefabs/abigail_retaliation|阿比盖尔反击、怨咒与暗影增益特效]] | 5 注册：复仇反击、怨咒命中／承伤减益两变体与温蒂暗影凶杀增益 | `abigail_vex_hit`、`abigail_murder_buff` 等 / active |
| [[reference/prefabs/shadow_pillar|暗影囚牢·影柱法术体系]] | 4 注册：影柱本体、地面 base FX、隐藏 root 辅助与 CLASSIFIED 扫描法术 | `shadow_pillar_base_fx`、`shadow_pillar_spell` 等 / active |
| [[reference/prefabs/saddle_shadow_fx|梦魇鞍具跟随特效与暗影脚印]] | 2 注册：梦魇鞍具的跟随层（三段改主）与暗影脚印贴片 | `saddle_shadow_fx`、`saddle_shadow_footprint` / active |
| [[reference/prefabs/shadow_despawn|暗影消散、舞台影手与暗影憎恶光束]] | 4 注册：影人消散、舞台影臂／影手与迷雾憎恶光束 | `stageusher_attackarm`、`shrouden_optic_blast_fx` 等 / active |
| [[reference/prefabs/oceanhorror_ripples|海上暗影生物涟漪与上船挂点]] | 2 注册：宿主驱动的网络化涟漪层（SG 驱动缩放、透明组件写 alpha）与随船平台跟随挂点（离船／移除／沉船三条清理路径） | `oceanhorror_ripples`、`oceanhorror_attachpivot` / active |
| [[reference/prefabs/rocky_snip_fx|岩甲 Boss 攻击特效与枯萎变体表现]] | 4 注册：SG 生成的剪击弧与冲刺残影（伤害留宿主状态机）、枯萎 Boss 的双肩火层与 fade 层、蠕虫地面裂纹（宿主单槽引用管理） | `rocky_snip_fx`、`rocky_dash_fx`、`rocky_boss_shadow`、`worm_boss_dirt_ground_fx` / active |
| [[reference/prefabs/wagboss_beam_fx|WAGboss 轨道打击与机械装置表现]] | 4 注册：轨道光束（月灼混合载体）、机械腿超新星阻挡标记、瓦匠剪影与月亮精华提取器（混合） | `wagboss_beam_fx`、`wagboss_robot_leg_fx`、`alterguardian_contained` 等 / active |
| [[reference/prefabs/shadow_bishop_fx|暗影棋子羽群与暗影裂隙散件]] | 4 注册：主教羽群（播完一轮自删）、裂隙门粒子（无自删）、墨荒替身（带真实物理）、寄生复活面具（小写 fx 标签异例） | `shadow_bishop_fx`、`shadowrift_portal_fx`、`shadowthrall_mouth_dupe_fx`、`shadowthrall_parasite_fx` / active |
| [[reference/prefabs/ruinsnightmare_horn_attack|潜伏梦魇双角突袭]] | 1 注册：受击反击的双角投射物；宿主 combat 结算、每角独立去重、命中点转碎影并互删 | `ruinsnightmare_horn_attack` / active |
| [[reference/prefabs/shadowhand_arm|暗影之手／恐惧之手的伸缩手臂]] | 2 注册：普通／遮蔽两套 build；固定起点朝手部拉伸，无自删，宿主负责清理；查理手部第二生成点 | `shadowhand_arm`、`shadowhand_arm_shrouded` / active |
| [[reference/prefabs/shrouden_eye_fx|暗影憎恶死亡水洼之眼]] | 1 注册：死亡水洼可检视眼睛；本地 goop 层＋animsync 同步，PUDDLE 专属检视文本，随状态／Boss 清理 | `shrouden_eye_fx` / active |
| [[reference/prefabs/hotcold_fx|小惊吓冷热寻物指示]] | 1 注册：按玩家到最近玩具的距离缩放；任务完成移除、放弃路径不清理 | `hotcold_fx` / active |
| [[reference/prefabs/wagboss_missile_target_fx|WAGboss 导弹落点与无人机电束电球]] | 5 注册：导弹落点环（客户端本地）、黄莺／笼墙电束（分段扫描）、黄莺／电刑机 电球（混合弹体，伤害归 FX） | `wagboss_missile_target_fx`、`wagdrone_laserwire_fx` 等 / active |
返回 [[reference/prefabs/index|Prefab 索引]]。
