---
title: "FX · 攻击、弹道与落点"
created: "2026-11-07"
updated: "2026-11-07"
type: index
tags: [overview]
---

# FX · 攻击、弹道与落点

瞄准与范围圈、射弹、激光、弹体落点与范围伤害。

## 瞄准、蓄能与范围提示

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/reticuleaoe|瞄准圈与 AoE 目标指示]] | 八文件 48 注册：本地准星与联网目标 FX，骑乘拼名与按动作清理 | `reticule`、`reticulearc`、`reticuleline`、`reticulelong`、`reticulemultitarget`、`golfclub_reticule_fx` 等 / active |
| [[reference/prefabs/cannon_aoe_range_fx|船炮瞄准范围圈特效]] | 本地船炮／球杆瞄准圈；调用方负责清理 | `cannon_reticule_fx` / active |
| [[reference/prefabs/bishop_targeting_fx|发条主教预警圈]] | 1 注册：dist netvar 双端光斑协议（专服也要光）、超时淡出/打断无 pst 两观感 | `bishop_targeting_fx` / active |
| [[reference/prefabs/wagboss_missile_target_fx|WAGboss 导弹落点与无人机电束电球]] | 5 注册：导弹落点环（客户端本地）、黄莺／笼墙电束（分段扫描）、黄莺／电刑机 电球（混合弹体，伤害归 FX） | `wagboss_missile_target_fx`、`wagdrone_laserwire_fx` 等 / active |
| [[reference/prefabs/slingshot_ice_aoe_fx|弹弓 AOE 命中圈（slingshot_ice_aoe_fx）]] | 元素 AOE 4 名快照内无生成点（诚实缺口）；实际表现走专属工厂 slingshot_aoe_fx 单体五色着色 | slingshot_slow_aoe_fx、slingshot_shadow_aoe_fx、slingshot_lunar_aoe_fx、slingshot_aoe_fx、弹弓AOE、命中圈特效 / active |

## 挥击、突刺与射弹

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/abigail_attack_fx|阿比盖尔攻击环特效]] | 阿比盖尔双层攻击环；宿主播放与 kill_fx 收尾 | `abigail_attack_fx_ground` / active |
| [[reference/prefabs/spear_gungnir_lungefx|长矛突刺表现]] | 3 注册：奔雷矛八向突刺光与装备跟随层（换绑与皮肤偏移）、冈尼尔突刺（事件包缺口） | `spear_gungnir_lungefx`、`spear_wathgrithr_lightning_lunge_fx`、`spear_wathgrithr_lightning_fx` / active |
| [[reference/prefabs/voidcloth_boomerang_launch_fx|虚布回旋镖与皮弗娄牛复活雷光]] | 3 注册：回旋镖发射与命中、牛复活雷 | voidcloth_boomerang_impact_fx、beefalo_reviving_lightning_fx / active |
| [[reference/prefabs/voidcloth_boomerang_fx|虚布装备持握层与弹体]] | 5 注册：回旋镖两层持握帧层（本体仅 ClearOverride 的新发现）、魔镰、暗影伞；proj 混合载体伤害在弹体 | `voidcloth_boomerang_fx`、`voidcloth_boomerang_proj`、`voidcloth_scythe_fx`、`voidcloth_umbrella_fx` / active |
| [[reference/prefabs/brilliance_projectile_fx|亮茄魔杖辉光弹]] | 亮茄弹跳投射物；命中额度、闪爆与武器伤害 | `brilliance_projectile_blast_fx` / active |
| [[reference/prefabs/shadowthrall_projectile_fx|墨荒弹幕投射物]] | 墨荒弹幕真实伤害；落地 AOE 与共享去重 | — / active |
| [[reference/prefabs/eye_charge|有表现的玩法载体（眼弹、诱鱼区、黏块、速爆弹）]] | 4 注册：炮塔眼弹、落海诱鱼区、可装瓶小黏块与速爆弹；伤害/定身/吸附归属分清 | `chum_aoe`、`gelblob_small_fx`、`fused_shadeling_quickfuse_bomb` / active |
| [[reference/prefabs/cannonball_used|船炮与迫击炮弹命中表现（cannonball_used）]] | 4 注册：船炮石弹落地无声动画与蟹王迫击炮弹三变体（同动画按命中面配声）；原版 dockmanager 恒在使落地恒选 _wood，mortarball.lua 撞墙生成点为死代码 | mortarball_used、mortarball_used_wood、mortarball_used_ice、炮弹命中特效、船炮命中表现、迫击炮命中表现 / active |

## 落点、陷阱与范围结算

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/impact|命中与落点伤害表现及传送门投影 FX 族]] | 7 注册：命中贴片、吹箭弹道、垃圾球两段、裂隙改造者爆炸与两个传送门投影 | `houndstooth_proj`、`junkball_fall_fx`、`lunarrift_portal_shadow` 等 / active |
| [[reference/prefabs/groundpoundring_fx|冲击环、爆点与碎块表现特效]] | 6 注册：震地环、火环、地刺爆裂、炸弹爆炸、碎块与眼睛炮塔命中音 | `firering_fx`、`ground_chunks_breaking`、`eye_charge_hit` 等 / active |
| [[reference/prefabs/groundpound_fx|震地尘土（groundpound_fx）]] | 震地落点尘土 2 名：groundpounder 按环点在陆地非码头落 (x,0,z)、fxRings 控制外圈、海狸模式覆写换名 | werebeaver_groundpound_fx、震地尘土、震地特效 / active |
| [[reference/prefabs/shadow_trap|暗影陷阱与减速表现]] | 2 注册：法术陷阱本体与目标减速／恐慌贴片 | `shadow_trap_debuff_fx` / active |
| [[reference/prefabs/wx78_taser_projectile_fx|WX-78 电击蓄能爆发]] | 1 注册：常驻电壳＋三档动画＋脱父挂爆发；attacker 玩家／FX 双分支，电免疫取 1 倍 | `wx78_taser_projectile_fx` / active |
| [[reference/prefabs/glass_fx|玻璃化、灰烬消散与睡眠炸弹爆裂（glass_fx 等三族）]] | 三条互不相关单消费链：glass_fx 沙晶点燃 22 帧后起爆、animover 才换玻璃产物；erode_ash 化石尖刺到期消散（moonspider 仅声明无生成）；sleepbomb_burst 睡眠炸弹命中爆裂（睡眠在 sleepcloud） | glass_fx、erode_ash、sleepbomb_burst、玻璃化特效、灰烬消散、睡眠炸弹爆裂 / active |
| [[reference/prefabs/explode_small|小爆裂、换装烟与火药爆裂]] | 十注册：爆裂、换装烟、火药及爆竹 | explode_reskin、reskin_tool_brush_explode_fx、reskin_tool_bouquet_explode_fx、reskin_tool_wand_explode_fx、reskin_tool_toilet_explode_fx、explode_small_slurtle、explode_small_slurtlehole、explode_firecrackers、slingshotammo_gunpowder_explode / active |
| [[reference/prefabs/trap_fumarole_burn_fx|喷气孔陷阱与藤蔓底座]] | 2 注册：DECOR 标签异例（无 FX）、组件复用同实体、藤蔓底座复用影柱 bank（宿主 DespawnTrap 先挪位再 KillFX） | `trap_fumarole_burn_fx`、`trap_vines_base_fx` / active |

## 飞溅、溅斑与残留

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/ink_splash|墨水弹溅落（ink_splash）]] | 墨水弹命中溅落；洼地按海陆分流与 inkable 上墨在投射物链路 | 墨水溅落、墨水弹特效 / active |
| [[reference/prefabs/bile_splash|胆汁弹溅落（bile_splash）]] | 胆汁弹命中溅落；半径 2 AOE 伤害与洼地分流由投射物结算 | 胆汁溅落、胆汁弹特效 / active |
| [[reference/prefabs/waterballoon_splash|水球溅落（waterballoon_splash）]] | 水球投射物命中与水球堆建成两路径；加湿／建造者沾湿在调用方，条目无 fn | 水球溅落、水球水花 / active |
| [[reference/prefabs/spat_splat_fx|唾沫溅斑（spat_splat_fx）]] | 五注册溅斑；spat 弹命中落斑（y=0）＋pinnable 默认碎裂列表四段量级（入参 ratio 被内部覆盖） | spat_splash_fx_full、spat_splash_fx_med、spat_splash_fx_low、spat_splash_fx_melted、唾沫特效、碎裂唾沫 / active |
| [[reference/prefabs/icing_splat_fx|糖霜溅斑（icing_splat_fx）]] | 五注册姜饼黏液弹溅斑；双命中分支落斑、pinnable:Stick 传 icing 专属列表（full→melted 顺序） | icing_splash_fx_full、icing_splash_fx_med、icing_splash_fx_low、icing_splash_fx_melted、糖霜特效、姜饼溅斑 / active |
| [[reference/prefabs/goosplash|黏液飞溅变体族]] | 9 注册：八种定号飞溅与随机基名；脚本层无生成点（事件数据未覆盖） | `goosplash1`～`goosplash8`、`goosplash` / active |
| [[reference/prefabs/ink_puddle_land|墨水与胆汁洼地（ink_puddle、bile_puddle 族）]] | 四注册：墨水／胆汁弹命中按 IsOnOcean 分海陆两档贴地洼底动画；溅起与洼底同点各自生成，上墨与 AOE 在投射物 onhit | ink_puddle_water、bile_puddle_land、bile_puddle_water、墨水洼、胆汁洼、命中洼地 / active |

## 关联入口

命中闪光、受击附着和格挡见 [[reference/prefabs/fx-status-index|增益、受击、护盾与状态反馈]]。船体破坏、航迹和落水水花见 [[reference/prefabs/fx-water-index|水面、航海与船件]]。

返回 [[reference/prefabs/fx-index|FX 主题总索引]]、[[reference/prefabs/index|Prefab 索引]]。
