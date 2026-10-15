# 专属 FX 批次的产物分组（2026-10-30）

范围为 output/fx-queue.json 的 75 个专属工厂对应页，Steam buildid 26230496；不含 fx.lua 通用表与其他宿主内的 FX。这里分类的是页面内产物的组织方式，不新增游戏对象 tags 或知识目录。

- 单产物主线：36 页，无需为了分类增加空章节。
- 保留变体表：33 页，差异可在已有注册／关系表及案例中直接判别；不要求每个注册名都分节。
- 按产物分节：6 页，玩法职责或接入／清理协议差异显著。本次将关联导航的已有条目归入可独立读取的三级小节，保留注册总表、案例、原引用及原文件地址。

## 按产物分节（已执行）

| 页面 | 章节入口 |
|---|---|
| warg_mutated_breath_fx | [[reference/prefabs/warg_mutated_breath_fx#喷焰伤害与宿主（warg_mutated_breath_fx）|喷焰伤害与宿主（warg_mutated_breath_fx）]] ／[[reference/prefabs/warg_mutated_breath_fx#灼烬与结冰打滑（warg_mutated_ember_fx）|灼烬与结冰打滑（warg_mutated_ember_fx）]] ／[[reference/prefabs/warg_mutated_breath_fx#共享对象池与参数|共享对象池与参数]] |
| willow_shadow_flame | [[reference/prefabs/willow_shadow_flame#自动索敌暗焰（willow_shadow_flame）|自动索敌暗焰（willow_shadow_flame）]] ／[[reference/prefabs/willow_shadow_flame#投掷落点火（willow_throw_flame）|投掷落点火（willow_throw_flame）]] ／[[reference/prefabs/willow_shadow_flame#狂热光环（willow_frenzy）|狂热光环（willow_frenzy）]] ／[[reference/prefabs/willow_shadow_flame#共享火光与资源|共享火光与资源]] |
| miasma_cloud | [[reference/prefabs/miasma_cloud#玩法瘴气（miasma_cloud）|玩法瘴气（miasma_cloud）]] ／[[reference/prefabs/miasma_cloud#视觉云与粒子子体|视觉云与粒子子体]] ／[[reference/prefabs/miasma_cloud#共享资源|共享资源]] |
| lunarmerm_thorns_fx | [[reference/prefabs/lunarmerm_thorns_fx#荆棘伤害（lunarmerm_thorns_fx）|荆棘伤害（lunarmerm_thorns_fx）]] ／[[reference/prefabs/lunarmerm_thorns_fx#翻土占位标记（merm_soil_marker）|翻土占位标记（merm_soil_marker）]] |
| deer_ice_circle | [[reference/prefabs/deer_ice_circle#冰圈结算（deer_ice_circle）|冰圈结算（deer_ice_circle）]] ／[[reference/prefabs/deer_ice_circle#火圈结算（deer_fire_circle）|火圈结算（deer_fire_circle）]] ／[[reference/prefabs/deer_ice_circle#充能角饰与爆裂子效|充能角饰与爆裂子效]] ／[[reference/prefabs/deer_ice_circle#共享施法与资源|共享施法与资源]] |
| tophat_shadow_fx | [[reference/prefabs/tophat_shadow_fx#常驻影子（tophat_shadow_fx）|常驻影子（tophat_shadow_fx）]] ／[[reference/prefabs/tophat_shadow_fx#取物旋涡与微粒（tophat_swirl_fx 与 tophat_using_shadow_fx）|取物旋涡与微粒（tophat_swirl_fx 与 tophat_using_shadow_fx）]] ／[[reference/prefabs/tophat_shadow_fx#共享资源|共享资源]] |

## 保留变体表

| 页面 | 保留理由 |
|---|---|
| abigail_attack_fx | 共同 kill_fx 协议，朝向／收尾帧数差异 |
| bearger_swipe_fx | 同工厂着色／自发光参数差异 |
| book_fx | 共用 SG 选型，书系／骑乘差异已列总表 |
| brilliance_projectile_fx | 弹体主线与其命中闪爆的主从关系已列清 |
| cannon_aoe_range_fx | 本地指示圈，共同由消费者清理 |
| channel_absorb_fire_fx | 引导主线及一次性反馈标记的退场差异已列清 |
| collapse_big | 大小动画及共同材质协议 |
| confetti_fx | 同一爆发协议，形状表差异 |
| deerclops_icelance_ping_fx | 预警／落点／足迹／附火已逐行标清，保留同次攻击的选型总表 |
| float_fx_front | 成对前后层，同一消费者生命周期 |
| fossilized_break_fx | 碎裂与尘雾的端和退场已分别列于总表 |
| hitsparks_fx | 同壳及 Setup 家族，定位参数差异已列清 |
| lantern_crystal_fx_ground | held／ground 仅动画名差异 |
| lantern_flower_fx_held | held／ground 差异已列清，并已有同族机制主页小节 |
| lantern_gothic_fx_held | 自身差异表＋flower 主页链接 |
| lantern_insect_fx_held | 自身差异表＋flower 主页链接 |
| lantern_tesla_fx_held | 同闭包双后缀，共用循环与移除协议 |
| lantern_winter_fx_held | 自身差异表＋flower 主页链接 |
| lavaarena_creature_teleport_small_fx | 两工厂及计数页差异已列清；外部生成点未核验 |
| lavae_move_fx | 同代理协议，缩放／着色参数差异 |
| marionette_appear_fx | 同一 SetTime 协议，出场／退场方向差异 |
| moose_nest_fx_idle | 同代理协议，仅 idle／hit 动画差异 |
| petrified_trunk_break_fx | 共同染色代理，阶段／动画／音效差异 |
| player_float_hop_water_fx | 同一对帧解绑协议，动作／面向差异 |
| pumpkincarving_swap_fx | 共用 SetData，南瓜／雪人专属差异已有表格 |
| reviver_cupid_beat_fx | beat／glow 分工与宿主接线已在表中逐项列清 |
| sharkboi_swipe_fx | 五种攻击表现的选型与各自退场已列总表 |
| spooked_spider_rock_fx | 共同唤醒声效与 animover 协议 |
| superjump_fx | 主 FX 自动生成碎屑，保留主从关系说明 |
| vault_pillar_guard_swipe_fx | 挥击／双层砸地差异已列清，现有典型接入专讲双层 |
| woby_dash_shadow_fx | 残影／剪影由同一状态管理，追踪与删除差异已列清 |
| wortox_portal_fx | 同工厂出入场动画差异 |
| wurt_casting_fx | 共同施法状态，物品／骑乘／上下层差异已列清 |

## 单产物主线

| 页面 | 处理 |
|---|---|
| acidsmoke_fx | 保留现有职责／接口／案例章节 |
| alterguardian_summon_fx | 保留现有职责／接口／案例章节 |
| boat_water_fx | 保留现有职责／接口／案例章节 |
| cave_vent_ground_fx | 保留现有职责／接口／案例章节 |
| daywalker2_swipe_fx | 保留现有职责／接口／案例章节 |
| deerclops_icespike_fx | 保留现有职责／接口／案例章节 |
| electricchargedfx | 保留现有职责／接口／案例章节 |
| electrocute_fx | 保留现有职责／接口／案例章节 |
| eyeofterror_arrive_fx | 保留现有职责／接口／案例章节 |
| firepit_firebird_puff_fx | 保留现有职责／接口／案例章节 |
| flamethrower_fx | 保留现有职责／接口／案例章节 |
| gelblob_attach_fx | 保留现有职责／接口／案例章节 |
| ghost_transform_overlay_fx | 保留现有职责／接口／案例章节 |
| ice_crack_grid_fx | 保留现有职责／接口／案例章节 |
| lunarrift_crystal_spawn_fx | 保留现有职责／接口／案例章节 |
| lunar_goop_cloud_fx | 保留现有职责／接口／案例章节 |
| lunar_goop_trail_fx | 保留现有职责／接口／案例章节 |
| monkeyisland_portal_fxloot | 保留现有职责／接口／案例章节 |
| moonstorm_ground_lightning_fx | 保留现有职责／接口／案例章节 |
| mossling_spin_fx | 保留现有职责／接口／案例章节 |
| nightsword_cotl_fx | 保留现有职责／接口／案例章节 |
| nightsword_curve_fx | 保留现有职责／接口／案例章节 |
| nightsword_lightsbane_fx | 保留现有职责／接口／案例章节 |
| nightsword_sharp_fx | 保留现有职责／接口／案例章节 |
| nightsword_wizard_fx | 保留现有职责／接口／案例章节 |
| pocketwatch_weapon_fx | 保留现有职责／接口／案例章节 |
| rabbithouse_yule_glow_fx | 保留现有职责／接口／案例章节 |
| rock_break_fx | 保留现有职责／接口／案例章节 |
| rose_petals_fx | 保留现有职责／接口／案例章节 |
| shadowthrall_projectile_fx | 保留现有职责／接口／案例章节 |
| shadow_glob_fx | 保留现有职责／接口／案例章节 |
| weremoose_smash_fx | 保留现有职责／接口／案例章节 |
| wormwood_plant_fx | 保留现有职责／接口／案例章节 |
| wortox_eat_soul_fx | 保留现有职责／接口／案例章节 |
| wortox_soul_heal_fx | 保留现有职责／接口／案例章节 |
| wortox_soul_in_fx | 保留现有职责／接口／案例章节 |

## 验证

- 六篇修改页 validate_page --check-indexes 通过；现有 29 张导航、919 篇知识页登记无缺陷，六页均在各自大小预算内。
- validate_source_links --strict：277 处 Lua 行号引用、40 个源码文件，0 error／0 warning；新增引言只摘取页内已有来源所支持的前提，不增加未审游戏结论。
- 19 个新增三级小节逐一用标题原文执行 query_wiki read，全部返回 0、无截断，均包含快速判断与验证记录；最大完整输出 9,678 字节。
- 六页原关联表共 52 条数据行逐字保留、各出现一次；sources 与 Lua 示例不变。定向来源基线 check 为 6 页／52 组 unchanged，不执行 record。
- 本清单三组共 75 页，各页唯一归组。36 页按单主产物组织（可含内部子体），33 页保留表格属于当前内容组织判断，不等于注册名永远不需独立成节。
- 未运行游戏、未解包动画资源；查询缓存仍为旧 62 页，未重建，也未用 stale search 验收。本次章节读取直接读取当前 Markdown。
- 保留其他 Agent 改动，raw 只读，未提交或推送。
