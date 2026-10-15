---
title: 特效 Prefab 索引
created: "2026-10-30"
updated: "2026-11-05"
type: index
tags: [overview]
---

# 特效 Prefab

专属及宿主文件内独立特效工厂的知识页，按主题分组；注册名与文件名不一致时以实际注册名命名。皮肤附属特效（手杖／冰箱）仍登记在 [[reference/prefabs/index|Prefab 索引]] 的对应分组；生物／Boss 与环境世界类页面见 [[reference/prefabs/fx-world-index|特效 Prefab 索引·生物与环境]]。返回 [[reference/prefabs/index|Prefab 索引]]。

## 皮肤与装备特效

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/nightsword_cotl_fx|咩咩之刃攻击烟尘特效]] | 咩咩之刃烟尘；atk 动画门禁与两路爆发 | — / active |
| [[reference/prefabs/nightsword_curve_fx|扭曲匕首攻击火花特效]] | 扭曲匕首火花；三路爆发与火星加速度 | — / active |
| [[reference/prefabs/nightsword_lightsbane_fx|魔光剑攻击像素粒子特效]] | 魔光剑像素粒子；实际数量与包络重名 | — / active |
| [[reference/prefabs/nightsword_sharp_fx|腐败刀锋攻击雪粒特效]] | 腐败刀锋雪粒；三路数量与包络关键帧 | — / active |
| [[reference/prefabs/nightsword_wizard_fx|诅咒之剑攻击暗影微粒特效]] | 诅咒之剑微粒；双路 reveal 与攻击门禁 | — / active |
| [[reference/prefabs/tophat_shadow_fx|魔术师高礼帽影子特效]] | 魔术师礼帽三种表现；本地分层与粒子池 | `tophat_swirl_fx`、`tophat_using_shadow_fx` / active |
| [[reference/prefabs/pocketwatch_weapon_fx|警钟暗影微粒特效]] | 警钟环绕微粒；宿主前提、年龄速率与停发 | — / active |
| [[reference/prefabs/lantern_crystal_fx_ground|盐灯持续光点特效]] | 盐灯双形态循环光点；KillFX 立即移除 | `lantern_crystal_fx_held` / active |
| [[reference/prefabs/lantern_flower_fx_held|纸灯笼花瓣飘落特效]] | 纸灯笼花瓣流；同族机制主页与地面退场 | `lantern_flower_fx_ground` / active |
| [[reference/prefabs/lantern_gothic_fx_held|哥特灯笼光屑飘落特效]] | 哥特灯笼光屑；花瓣族协议与参数差异 | `lantern_gothic_fx_ground` / active |
| [[reference/prefabs/lantern_insect_fx_held|蝇光灯光屑飘落特效]] | 蝇光灯光屑；glowflake 符号与 petalfall 动画 | `lantern_insect_fx_ground` / active |
| [[reference/prefabs/lantern_tesla_fx_held|特斯拉线圈提灯电光特效]] | 特斯拉提灯电光；双变体与随机空白帧 | `lantern_tesla_fx_ground`、`lantern_tesla_fx_` / active |
| [[reference/prefabs/lantern_winter_fx_held|冬季盛宴提灯落雪特效]] | 冬季盛宴提灯落雪；同族协议与 alt 共用 | `lantern_winter_fx_ground` / active |
| [[reference/prefabs/rabbithouse_yule_glow_fx|倒挂树屋光晕特效]] | 兔屋皮肤常驻光晕；循环动画与清理 | — / active |
| [[reference/prefabs/armor_lunarplant_glow_fx|装备与帽饰跟随光效]] | 七项装备／护符／帽饰／电池附属 FX；护甲光效控制器与帽片符号跟随 | `armor_voidcloth_fx`、`yellowamuletlight`、`alterguardian_hat_equipped` 等 / active |

## 角色技能特效

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/abigailsummonfx|阿比盖尔之花召唤、收回与升级特效]] | 五注册：骑乘选名、动作清理与等级／惊吓复用 | abigailsummonfx_mount、abigailunsummonfx、abigailunsummonfx_mount、abigaillevelupfx / active |
| [[reference/prefabs/wortox_eat_soul_fx|进食灵魂特效]] | 灵魂进食动画；骑乘切换与状态清理 | — / active |
| [[reference/prefabs/wortox_portal_fx|灵魂跳跃出入场特效]] | 灵魂跳跃出入场；宿主位移与特效分工 | `wortox_portal_jumpin_fx`、`wortox_portal_jumpout_fx`、`soulhop` / active |
| [[reference/prefabs/wortox_soul_heal_fx|灵魂治疗特效]] | 灵魂治疗提示；帧驱动加色与组件前提 | — / active |
| [[reference/prefabs/wortox_soul_in_fx|灵魂入体特效]] | 灵魂入体提示；目标同步与客户端染色 | — / active |
| [[reference/prefabs/wurt_casting_fx|沃特施法特效]] | 四种鱼人施法动画；骑乘拼名与上下分层 | `purebrilliance_castfx`、`purebrilliance_castfx_mount`、`horrorfuel_castfx`、`horrorfuel_castfx_mount` / active |
| [[reference/prefabs/staffcastfx|施法与怀表回溯表现]] | 八注册：骑乘选名、父挂染色与状态清理 | `staffcastfx_mount`、`cointosscastfx`、`cointosscastfx_mount`、`pocketwatch_cast_fx`、`pocketwatch_cast_fx_mount`、`pocketwatch_warpback_fx`、`pocketwatch_warpbackout_fx` / active |
| [[reference/prefabs/staff_castinglight|施法光源]] | 两注册：施法／护符／怀表状态的光晕，五次幂包络与 2 帧延迟移除 | `staff_castinglight_small` / active |
| [[reference/prefabs/willow_shadow_flame|薇洛暗焰]] | 薇洛三种火焰；索敌伤害与循环清理边界 | `willow_throw_flame`、`willow_frenzy` / active |
| [[reference/prefabs/emberlight|火球（emberlight）]] | 薇洛火球术产物：光照供热、烹饪传播与定时熄灭 | 火球、薇洛火球、火球术 / active |
| [[reference/prefabs/wormwood_plant_fx|沃姆伍德开花植物特效]] | 开花小植物；变体轮换与玩家驻留条件 | — / active |
| [[reference/prefabs/reviver_cupid_beat_fx|告密的心丘比特皮肤心跳特效]] | 告密的心皮肤；心跳拍点粒子与高亮覆盖 | `reviver_cupid_glow_fx` / active |

## 战斗反馈特效

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/electricchargedfx|电充闪光特效]] | 带电 buff 充电标记；光衰减与目标闪白 | `electric_charged_fx`、`buff_electricattack` / active |
| [[reference/prefabs/electrocute_fx|电击表现与连锁闪电]] | 电击反应与连锁；SG 接线、门禁与退场 | — / active |
| [[reference/prefabs/flamethrower_fx|薇洛暗焰喷射布焰代理]] | 薇洛喷焰代理；周期布焰与延迟武器清理 | `lunar_fire` / active |
| [[reference/prefabs/channel_absorb_fire_fx|薇洛打火机吸火引导特效]] | 打火机引导吸火表现；KillFX 与一次性标记 | `channel_absorb_fire`、`channel_absorb_smoulder`、`channel_absorb_embers` / active |
| [[reference/prefabs/firepit_firebird_puff_fx|带翅膀的火盆添柴烟焰]] | 火盆皮肤添柴烟焰；燃料分档与代理同步 | — / active |
| [[reference/prefabs/hitsparks_fx|命中火花特效]] | 三种命中火花；定位、目标加色与黑剪影 | `hitsparks_reflect_fx`、`hitsparks_piercing_fx` / active |
| [[reference/prefabs/abigail_attack_fx|阿比盖尔攻击环特效]] | 阿比盖尔双层攻击环；宿主播放与 kill_fx 收尾 | `abigail_attack_fx_ground` / active |
| [[reference/prefabs/reticuleaoe|瞄准圈与 AoE 目标指示]] | 八文件 48 注册：本地准星与联网目标 FX，骑乘拼名与按动作清理 | `reticule`、`reticulearc`、`reticuleline`、`reticulelong`、`reticulemultitarget`、`golfclub_reticule_fx` 等 / active |

## 杂项表现特效

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/book_fx|读书施法书本特效]] | 六种读书表现；骑乘选型与施法失败反馈 | `book_fx_mount`、`waxwell_book_fx`、`waxwell_book_fx_mount`、`waxwell_shadow_book_fx`、`waxwell_shadow_book_fx_mount` / active |
| [[reference/prefabs/float_fx_front|漂浮物前后水环]] | 本地前后漂浮水环；宿主移除或侵蚀淡出 | `float_fx_back`、`float_fx` / active |
| [[reference/prefabs/player_float_hop_water_fx|玩家出水水环]] | 玩家出水水环；对帧、短暂跟随与解绑 | `player_hotspring_water_fx`、`player_float_fx` / active |
| [[reference/prefabs/monkeyisland_portal_fxloot|猴岛传送门假掉落]] | 传送门假掉落；加权抽取、满额兜底与落地续播 | `fxloot` / active |
| [[reference/prefabs/moonstorm_ground_lightning_fx|月风暴地面闪电]] | 本地月风暴闪电；链生节点与距离门禁 | — / active |
| [[reference/prefabs/spooked_spider_rock_fx|惊吓蜘蛛虫群特效]] | 工作惊吓双表现；组件触发与唤醒音效 | `spooked_worms_fx`、`spooked_fx` / active |
| [[reference/prefabs/rock_break_fx|岩石碎裂音效代理]] | 岩石碎裂纯音效；网络壳寿命为 1 秒 | — / active |
| [[reference/prefabs/rose_petals_fx|玫瑰花瓣飘落]] | 玫瑰花瓣；音效载体与 ErodeAway 淡出 | — / active |
| [[reference/prefabs/acidsmoke_fx|酸雨烟雾特效]] | 酸雨侵蚀烟雾；组件调档与休眠清理 | — / active |
| [[reference/prefabs/confetti_fx|大赛庆祝彩纸]] | 双形状彩纸爆发；专服壳需补清理 | `rabbit_confetti_fx`、`confetti` / active |
| [[reference/prefabs/lavae_move_fx|岩浆虫与哈奇移动拖尾]] | 岩浆虫／哈奇拖尾；变体去重与缩放量化 | `hutch_move_fx` / active |
| [[reference/prefabs/shadow_glob_fx|暗影地面痕特效]] | 暗影地面痕；去重标签与海面涟漪 | — / active |
| [[reference/prefabs/slingshotammo_purebrilliance_debuff_fx|弹弓弹药减益表现]] | 3 注册：纯粹辉煌双层标记（front/back 对）、恐惧燃料 DoT（伤害由 FX 实体结算的混合载体）、减速纯视觉 | `slingshotammo_purebrilliance_debuff_fx`、`slingshotammo_horrorfuel_debuff_fx`、`slingshotammo_slow_debuff_fx` / active |
| [[reference/prefabs/spear_gungnir_lungefx|长矛突刺表现]] | 3 注册：奔雷矛八向突刺光与装备跟随层（换绑与皮肤偏移）、冈尼尔突刺（事件包缺口） | `spear_gungnir_lungefx`、`spear_wathgrithr_lightning_lunge_fx`、`spear_wathgrithr_lightning_fx` / active |
| [[reference/prefabs/winona_battery_high_shatterfx|薇诺娜宝石发电机设备特效族]] | 5 注册：发电机碎宝石与纯粹辉煌槽位跟随层、电线火花（8 处生成不走代理）、聚光灯灯头、传送伞两层持握层 | `winona_battery_high_shatterfx`、`winona_battery_sparks`、`purebrilliance_symbol_fx` 等 / active |
| [[reference/prefabs/forcefieldfx|图勒皇冠护盾特效]] | 1 注册：铥矿皇冠 proc 护盾层（netvar 灯光协议、kill_fx 唯一收口、未置 persists 的存档语义辨析） | `forcefieldfx` / active |
| [[reference/prefabs/wx78_big_spark|WX78 外围表现族]] | 4 注册：大火花（代理无 AnimState 闪白不可达）、照明电路灯点链、扫描锥 AddChild、备份底盘渲染层（混合） | `wx78_big_spark`、`wx78_lightbeam`、`wx78_scanner_fx`、`wx78_backupbody_inventory` / active |
| [[reference/prefabs/wortox_soulecho_buff_fx|沃拓克斯技能与装备表现族]] | 6 注册：灵魂回响加速层、灵魂过载警告、强抢袋三层（载体/动画/粒子）、双尾心跟随体 | `wortox_soulecho_buff_fx`、`wortox_overloading_fx`、`wortox_nabbag_body` 等 / active |
| [[reference/prefabs/wurt_shadow_merm_planar_fx|沃特位面增益与投泥带表现]] | 5 注册：暗影持续跟随层、月影一次性 pre/pst、投泥弹壳（改泥代理辨析）与两个 chargedfx（清单外并入） | `wurt_shadow_merm_planar_fx`、`wurt_lunar_merm_planar_fx`、`wurt_terraform_projectile` 等 / active |
| [[reference/prefabs/voidcloth_boomerang_fx|虚布装备持握层与弹体]] | 5 注册：回旋镖两层持握帧层（本体仅 ClearOverride 的新发现）、魔镰、暗影伞；proj 混合载体伤害在弹体 | `voidcloth_boomerang_fx`、`voidcloth_boomerang_proj`、`voidcloth_scythe_fx`、`voidcloth_umbrella_fx` / active |
| [[reference/prefabs/wendy_gravestone_rebirth_fx|温蒂墓碑重生双层演出]] | 2 注册：同一重生演出的阿比盖尔层与温蒂幽灵层（仅温蒂触发、无 SetPristine 全端对称） | `abigail_gravestone_rebirth_fx`、`wendy_gravestone_rebirth_fx` / active |
| [[reference/prefabs/hats|帽子跟随层与帽灯特效族]] | 12 注册：9 个 MakeFollowFx 跟随层（重建式）＋2 盏帽灯＋1 技术实体；mask_halfwit_fx 唯一生成点在舞台剧剧本 | `mask_halfwit_fx`、`minerhatlight`、`lunarplanthat_fx`、`tophat_container` 等 / active |
| [[reference/prefabs/bramblefx_armor|荆棘尖刺扩环族]] | 3 注册：armor/armor_upgrade/trap 混合载体（伤害两路归属辨析、husk 独占升级接线、扩环 3 帧结算窗） | `bramblefx_armor`、`bramblefx_armor_upgrade`、`bramblefx_trap` / active |
| [[reference/prefabs/wintersfeastbuff_fx|冬宴buff与烤炉火]] | 2 注册：debuff 周期投放落点 FX（强度方向与直觉相反）＋烤炉两阶段火层（宿主调度、无 firefx 组件） | `wintersfeastbuff_fx`、`wintersfeastoven_fire` / active |
| [[reference/prefabs/witchbroom_fx|扫帚轨迹与沃比驮架与教练buff]] | 3 注册：隐形壳＋本地池化轨迹（6/s 节拍勘误）、沃比驮架双层（大沃比专属）、教练伤害buff纯表现层 | `witchbroom_fx`、`woby_rack_swap_fx`、`wolfgang_coach_buff_fx` / active |
| [[reference/prefabs/wx78_taser_projectile_fx|WX-78 电击蓄能爆发]] | 1 注册：常驻电壳＋三档动画＋脱父挂爆发；attacker 玩家／FX 双分支，电免疫取 1 倍 | `wx78_taser_projectile_fx` / active |
| [[reference/prefabs/spellmasterybuff|法术精通 buff 与法球]] | 3 注册：buff 锚点、法球载体与单颗法球；buff／orbs 主模拟初始化在缺失的 lavaarena 事件包 | `spellmasterybuff`、`spellmasteryorbs`、`spellmasteryorb` / active |
