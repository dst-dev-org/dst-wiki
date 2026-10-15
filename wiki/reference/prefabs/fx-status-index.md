---
title: "FX · 增益、受击、护盾与状态反馈"
created: "2026-11-07"
updated: "2026-11-07"
type: index
tags: [overview]
---

# FX · 增益、受击、护盾与状态反馈

伤害反馈、恢复、格挡、位面抵抗、持续附着与减益。

## 命中、电击与位面反馈

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/sparks|电击命中火花（sparks 族）]] | 三注册独立工厂：随机三选一动画＋橙色闪光、AlignToTarget 贴目标朝攻击者侧、免疫变体分流（闪白机制当前不生效） | 电击火花、命中火花、electrichitsparks、electrichitsparks_electricimmune / active |
| [[reference/prefabs/weaponsparks|武器命中火花（weaponsparks 族）]] | 四注册熔炉血统工厂：基础／弹跳单段、穿刺／突刺父挂重算与 3 帧镜像背面段；主模拟初始化在 Forge 事件数据包 | weaponsparks_piercing、weaponsparks_thrusting、weaponsparks_bounce、武器命中特效、熔炉火花 / active |
| [[reference/prefabs/hitsparks_fx|命中火花特效]] | 三种命中火花；定位、目标加色与黑剪影 | `hitsparks_reflect_fx`、`hitsparks_piercing_fx` / active |
| [[reference/prefabs/electricchargedfx|电充闪光特效]] | 带电 buff 充电标记；光衰减与目标闪白 | `electric_charged_fx`、`buff_electricattack` / active |
| [[reference/prefabs/electrocute_fx|电击表现与连锁闪电]] | 电击反应与连锁；SG 接线、门禁与退场 | — / active |
| [[reference/prefabs/shock_fx|电击附着动画（shock_fx）]] | 电击附着六注册；玩家形态四选一符号跟随、电击模块反击弧光、月暴电花自附，与 electrocute_fx 分轨 | shock_arc_fx、werebeaver_shock_fx、weremoose_shock_fx、weregoose_shock_fx、moonstorm_spark_shock_fx、电击附着、电弧特效 / active |
| [[reference/prefabs/planar_hit_fx|位面命中与抵抗表现（planar_hit_fx）]] | 位面攻防两注册；命中挂受击目标（仅 inventory 持有者）、抵抗环带落点，条件在 combat／planarentity | planar_resist_fx、位面命中表现、位面抵抗表现、位面偏转 / active |
| [[reference/prefabs/slingshotammo_hitfx_rock|弹弓弹药命中表现（slingshotammo_hitfx_rock）]] | 弹弓命中 18 注册；按弹药覆符号／特定动画／光照，火药条目仅暴击时抑制常规命中特效 | slingshotammo_hitfx_gold、slingshotammo_hitfx_marble、slingshotammo_hitfx_thulecite、slingshotammo_hitfx_honey、slingshotammo_hitfx_freeze、slingshotammo_hitfx_slow、slingshotammo_hitfx_poop、slingshotammo_hitfx_moonglass、slingshotammo_hitfx_dreadstone、slingshotammo_hitfx_gunpowder、slingshotammo_hitfx_lunarplanthusk、slingshotammo_hitfx_purebrilliance、slingshotammo_hitfx_horrorfuel、slingshotammo_hitfx_gelblob、slingshotammo_hitfx_scrapfeather、slingshotammo_hitfx_stinger、slingshotammo_hitfx_trinket_1、弹弓命中特效 / active |

## 格挡、护盾与攻击反制

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/forcefieldfx|铥矿皇冠护盾特效]] | 1 注册：铥矿皇冠 proc 护盾层（netvar 灯光协议、kill_fx 唯一收口、未置 persists 的存档语义辨析） | `forcefieldfx` / active |
| [[reference/prefabs/wx78_shield_full|WX78 护盾动画（wx78_shield_full）]] | 护盾六注册；常驻两态周期重播、四条转场父挂宿主，破盾 11 帧后脱挂 | wx78_shield_half、wx78_shield_full_to_half、wx78_shield_half_to_full、wx78_shield_full_to_empty、wx78_shield_half_to_empty、WX78护盾动画、破盾特效 / active |
| [[reference/prefabs/shadow_shield1|骨头盔甲格挡护盾（shadow_shield1）]] | 护盾六注册（三变体×水平镜像）；骨甲 resistance 吸收时 PickShield 按主盾冷却选名并父挂持有者 | shadow_shield2、shadow_shield3、shadow_shield4、shadow_shield5、shadow_shield6、格挡护盾、骨甲护盾 / active |
| [[reference/prefabs/stalker_shield1|远古织影者格挡盾]] | 5 注册：PickShield 分布表、FX 实体直接作 attacker 的 10 点反伤、基名诚实缺口 | `stalker_shield1`、`stalker_shield2`、`stalker_shield3`、`stalker_shield4`、`stalker_shield` / active |
| [[reference/prefabs/bramblefx_armor|荆棘尖刺扩环族]] | 3 注册：armor/armor_upgrade/trap 混合载体（伤害两路归属辨析、husk 独占升级接线、扩环 3 帧结算窗） | `bramblefx_armor`、`bramblefx_armor_upgrade`、`bramblefx_trap` / active |
| [[reference/prefabs/abigail_retaliation|阿比盖尔反击、惊扰与暗影增益特效]] | 5 注册：复仇反击、惊扰命中／承伤减益两变体与温蒂暗影凶杀增益 | `abigail_vex_hit`、`abigail_murder_buff` 等 / active |
| [[reference/prefabs/toadstool_cap_absorbfx|毒菌蟾蜍帽吸收与释放]] | 2 注册：中毒金丝雀爆毒触发链（非 Boss 状态机勘误）、releasefx 复用金丝雀爆炸资源 | `toadstool_cap_absorbfx`、`toadstool_cap_releasefx` / active |

## 治疗、增益与减益

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/wortox_soul_heal_fx|灵魂治疗特效]] | 灵魂治疗提示；帧驱动加色与组件前提 | — / active |
| [[reference/prefabs/ghostlyelixir_slowregen_fx|幽灵灵药环绕与滴落特效（ghostlyelixir_slowregen_fx）]] | 36 注册：9 药效×阿比盖尔／玩家×环绕／滴落；buff 服务端生成，attach 父挂一次、drip 15 秒落点节流，四路停止清理；疗效在 buff 侧，玩家 shadow/lunar 无生成点 | ghostlyelixir_fastregen_fx、ghostlyelixir_shield_fx、ghostlyelixir_attack_fx、ghostlyelixir_speed_fx、ghostlyelixir_retaliation_fx、ghostlyelixir_revive_fx、ghostlyelixir_shadow_fx、ghostlyelixir_lunar_fx、ghostlyelixir_slowregen_dripfx、ghostlyelixir_fastregen_dripfx、ghostlyelixir_shield_dripfx、ghostlyelixir_attack_dripfx、ghostlyelixir_speed_dripfx、ghostlyelixir_retaliation_dripfx、ghostlyelixir_revive_dripfx、ghostlyelixir_shadow_dripfx、ghostlyelixir_lunar_dripfx、ghostlyelixir_player_slowregen_fx、ghostlyelixir_player_fastregen_fx、ghostlyelixir_player_shield_fx、ghostlyelixir_player_attack_fx、ghostlyelixir_player_speed_fx、ghostlyelixir_player_retaliation_fx、ghostlyelixir_player_revive_fx、ghostlyelixir_player_shadow_fx、ghostlyelixir_player_lunar_fx、ghostlyelixir_player_slowregen_dripfx、ghostlyelixir_player_fastregen_dripfx、ghostlyelixir_player_shield_dripfx、ghostlyelixir_player_attack_dripfx、ghostlyelixir_player_speed_dripfx、ghostlyelixir_player_retaliation_dripfx、ghostlyelixir_player_revive_dripfx、ghostlyelixir_player_shadow_dripfx、ghostlyelixir_player_lunar_dripfx、幽灵灵药特效、阿比盖尔灵药特效、灵药环绕特效、灵药滴落特效 / active |
| [[reference/prefabs/ghostflower_spirit1_fx|幽灵花灵体飘光（ghostflower_spirit1_fx）]] | 2 注册：幽灵花落地 3-9 秒周期飘光、墓碑花瓣安放固定 1 号；与灵药机制无关 | ghostflower_spirit2_fx、幽灵花灵体、灵体飘光 / active |
| [[reference/prefabs/wathgrithr_spirit|薇格弗德之魂与战歌特效（wathgrithr_spirit）]] | 14 注册：击杀英灵、战歌附着／退场及音符；electric 无生成点 | battlesong_attach、battlesong_loop、battlesong_detach、battlesong_durability_fx、battlesong_healthgain_fx、battlesong_sanitygain_fx、battlesong_sanityaura_fx、battlesong_fireresistance_fx、battlesong_shadowaligned_fx、battlesong_lunaraligned_fx、battlesong_instant_electric_fx、battlesong_instant_taunt_fx、battlesong_instant_panic_fx、薇格弗德之魂、武神之魂、战歌音符、战歌buff特效、瞬时战歌特效 / active |
| [[reference/prefabs/spider_heal_fx|蜘蛛治疗与突变（spider_heal_fx）]] | 蜘蛛阵营 4 名：护士蜘蛛 30 帧群疗（地面圈＋本体＋目标光）、治疗黏团使用链复用三名、蜘蛛突变烟落点；治疗／突变声在调用方 | spider_mutate_fx、spider_heal_target_fx、spider_heal_ground_fx、蜘蛛治疗、护士蜘蛛、治疗黏团、蜘蛛突变 / active |
| [[reference/prefabs/abigail_attack_shadow_fx|温蒂、阿比盖尔与辉煌闪现标记特效]] | 7 注册：理智光环、暗影与启迪攻击、辉煌闪现标记 | wendy_sanityaura_buff_on_fx、wendy_sanityaura_buff_off_fx、abigail_rising_twinkles_fx、abigail_gestalt_hit_fx、abigail_shadow_buff_fx、purebrilliance_mark_hit_fx、辉煌闪现标记 / active |
| [[reference/prefabs/slingshotammo_purebrilliance_debuff_fx|弹弓弹药减益表现]] | 3 注册：纯粹辉煌双层标记（front/back 对）、恐惧燃料 DoT（伤害由 FX 实体结算的混合载体）、减速纯视觉 | `slingshotammo_purebrilliance_debuff_fx`、`slingshotammo_horrorfuel_debuff_fx`、`slingshotammo_slow_debuff_fx` / active |
| [[reference/prefabs/spellmasterybuff|法术精通 buff 与法球]] | 3 注册：buff 锚点、法球载体与单颗法球；buff／orbs 主模拟初始化在缺失的 lavaarena 事件包 | `spellmasterybuff`、`spellmasteryorbs`、`spellmasteryorb` / active |
| [[reference/prefabs/sleepcloud|睡眠云与孢子云]] | 5 注册：范围催眠云（含月版）与范围伤害/腐败云，各带 7 片 overlay | `sleepcloud_lunar`、`sporecloud_overlay` 等 / active |
| [[reference/prefabs/wagdrone_rolling_collide_small_fx|滚动无人机碰撞与灵药护盾特效]] | 3 注册：碰撞火花及玩家灵药受击护盾表现 | wagdrone_rolling_collide_med_fx、elixir_player_forcefield / active |
| [[reference/prefabs/diseaseflies|病害蝇群表现]] | 病害组件生成的蝇群代理、本地动画与循环次数 | 病害蝇群 / active |

## 关联入口

施法、弹道、瞄准和实际落点伤害见 [[reference/prefabs/fx-combat-index|攻击、弹道与落点]]。

返回 [[reference/prefabs/fx-index|FX 主题总索引]]、[[reference/prefabs/index|Prefab 索引]]。
