---
title: "FX · 建筑、装置与遗迹演出"
created: "2026-11-07"
updated: "2026-11-07"
type: index
tags: [overview]
---

# FX · 建筑、装置与遗迹演出

建筑附属、装置运转、月坛与裂隙、遗迹机关与投影。

## 建筑附属、机关与破坏

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/fence_gate_anim|宿主挂接的附属表现实体]] | 12 注册：门动画、椅背层、茶铺前层、垃圾堆侧片／蓝图、灯链与水草底座 | `wood_chair_back`、`hermitcrab_teashop_front`、`yots_lantern_light_chain` 等 / active |
| [[reference/prefabs/rope_bridge_fx|绳桥与藤桥桥面表现]] | 3 注册：管理器按瓦片生成的桥面贴片（可通行面是瓦片非 FX；绳桥双绳本地重建、藤桥装饰子体、创建即排定约 18 秒警告与约 20⅓ 秒销毁） | `rope_bridge_fx`、`vine_bridge_fx`、`vine_bridge_decor_fx` / active |
| [[reference/prefabs/collapse_big|建筑倒塌特效]] | 建筑倒塌大小两款；SetMaterial 选择破碎声 | `collapse_small`、`structure_collapse_fx` / active |
| [[reference/prefabs/winona_battery_high_shatterfx|薇诺娜宝石发电机设备特效族]] | 5 注册：发电机碎宝石与纯粹辉煌槽位跟随层、电线火花（8 处生成不走代理）、聚光灯灯头、传送伞两层持握层 | `winona_battery_high_shatterfx`、`winona_battery_sparks`、`purebrilliance_symbol_fx` 等 / active |
| [[reference/prefabs/madscience_lab_goop|实验台三层特效与粒子杂项]] | 6 注册：实验台 goop／fire／glow、森林 Mist 雾、防滑砂 buff 粒子与小猫躲藏 | `mist`、`nonslipgrit_buff_fx`、`kitcoon_hide_fx` 等 / active |
| [[reference/prefabs/terrariumchest_fx|显眼箱子微光与传送塔沙柱]] | 2 注册：恐怖之眼门禁的宝箱双层微光（fx 引用即 SHIMMER 标记）与传送沙柱（唯一带 ObstaclePhysics 的 FX，KillFX＋35 帧兜底） | `terrariumchest_fx`、`townportalsandcoffin_fx` / active |
| [[reference/prefabs/lightning_rod_fx|避雷针电弧动画（lightning_rod_fx）]] | 单注册充电电弧标记；dozap 三触发（雷击充电／日终／取电）落点生成，zap 声在避雷针本体 | 避雷针电弧、避雷针特效、避雷针动画 / active |
| [[reference/prefabs/vault_compass_visual|宝库设施表现]] | 4 注册：罗盘指针三层链、净化器雾效、地面纹样（18 处生成、未置 persists 反例）、宝库火炬冷火三档（无 heater） | `vault_compass_visual`、`vault_decon_mister_fx`、`vault_ground_pattern_fx`、`vault_torch_flame` / active |
| [[reference/prefabs/ruins_cavein_obstacle_rubble_fx|废墟落石碎屑与实验室吊舱皮肤 FX]] | 2 注册：落石柱碎屑与研究台吊舱吸附皮肤 | `ruins_cavein_obstacle_rubble_fx`、`researchlab2_pod_fx` / active |

## 月坛、裂隙与古代装置

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/abysspillar_fx|深渊石柱坍塌与重建特效]] | 坍塌与形成、实体柱恢复及组追踪 | 深渊石柱特效 / active |
| [[reference/prefabs/archive_chandelier|远古吊灯与石像鬼石化特效]] | 8 注册：档案馆／宝库三盏吊灯＋石像鬼五石化特效；`vault_crawler_chandelier` 兼生成点 | `gargoyle_houndatk_fx` 等 / active |
| [[reference/prefabs/atrium_gate_activatedfx|远古大门声画三兄弟]] | 4 注册：activatedfx 两态切换（KillFX 兜底）＋makesfx 同构三兄弟（16.5 秒脉冲、爆炸、地穴战线；地区门控辨析） | `atrium_gate_activatedfx`、`atrium_gate_pulsesfx`、`atrium_gate_explodesfx`、`grotto_war_sfx` / active |
| [[reference/prefabs/moon_device_pillar|月装置支柱、顶盖、一阶段破碎与天体裂缝裂纹层]] | 4 注册：月亮虹吸器阶段支柱／顶盖／首次破碎与按月相显隐的裂纹层 | `moon_device_top`、`moon_fissure_fx` 等 / active |
| [[reference/prefabs/moon_altar_break|月坛破碎演出]] | 3 注册：月亮装置 9.5 秒破碎链清场生成（锤毁不生成）；crown/claw 为动画库名与祭坛 prefab 名错位辨析 | `moon_altar_break`、`moon_altar_claw_break`、`moon_altar_crown_break` / active |
| [[reference/prefabs/moon_altar_link_fx|月坛连接光效]] | 1 注册：月坛步进生成与非专用服月脉冲本地光点 | 月坛连接光效、月脉冲光点 / active |
| [[reference/prefabs/lunarrift_terraformer_visual|裂隙地皮改造视觉层与河灯影层]] | 2 注册：隐形管理器的 23 秒阶段演出层（earlyexit 门控）与天灯引擎父挂影层（成对动画同步） | `lunarrift_terraformer_visual`、`floatinglantern_shadow` / active |
| [[reference/prefabs/lunarrift_crystal_spawn_fx|裂隙晶体生成烟尘]] | 晶体生成烟尘；专服无内建自删 | — / active |
| [[reference/prefabs/positronbeam_back|月亮石充能光束]] | 3 注册：back/front/pulse 三件套（满月充能链、KillFX 双语义、StopFX 笔误陷阱） | `positronbeam_back`、`positronbeam_front`、`positronpulse` / active |
| [[reference/prefabs/terrarium_fx|盒中泰拉召唤光柱]] | 1 注册：夜间召唤光柱，激活／循环／退场三段，随宿主父挂，10 帧后移除 | `terrarium_fx` / active |
| [[reference/prefabs/sanity_raise|影岩升降标记动画（sanity_raise）]] | 两注册黑石升降标记；影柱计时、梦魇岩三态、破损祭坛交互与场景脚本落点，名称不代表理智增减 | sanity_lower、理智升降动画、影柱升起特效、梦魇岩升降 / active |

## 关联入口

怀表和传送门覆盖层见 [[reference/prefabs/fx-transition-index|变身、召唤、传送与退场]]。

返回 [[reference/prefabs/fx-index|FX 主题总索引]]、[[reference/prefabs/index|Prefab 索引]]。
