---
title: "FX · 采集、制作、碎裂与生成烟"
created: "2026-11-07"
updated: "2026-11-07"
type: index
tags: [overview]
---

# FX · 采集、制作、碎裂与生成烟

砍伐、铲土、采矿、作物、物件碎裂与开包。

## 采集、挖掘与作物

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/mining_fx|采石碎屑（mining_fx）]] | 两注册采矿碎屑；PlayMiningFX 按标签选名（frozen→冰、月晶水晶→组外），玩家啃咬／猎犬月兔直落点等十余处 | mining_ice_fx、采矿碎屑、冰屑特效 / active |
| [[reference/prefabs/shovel_dirt|铲土飞尘（shovel_dirt）]] | 单注册铲挖飞尘；海狸啃咬 DIG 分支与冬牧兔人动作帧／5% 概率分支落点，woodie 仅声明 | 铲土飞尘、挖土特效 / active |
| [[reference/prefabs/dirt_puff|作物挖掘烟（dirt_puff）]] | 单注册作物挖除烟；farm_plants 按 dig_fx 字段生成，与 small_puff 同 build 不同注册（无声多 FinalOffset） | 作物挖掘烟、挖除烟 / active |
| [[reference/prefabs/farm_plant_happy|作物表情与活动彩纸]] | 4 注册：作物压力、杂草照料与枕头战出界；牛年彩纸未找到生成点 | farm_plant_unhappy、yotb_confetti、pillowfight_confetti_fx / active |
| [[reference/prefabs/attackfx_handpillow_petals|枕头命中碎屑与作物挖起]] | 7 注册：四材质枕头碎屑（动态拼名、伤害 0 击退分流的混合载体）＋三档作物挖起尘（small_fx 两处清单外拖尾生成） | `attackfx_handpillow_petals`、`plant_dug_small_fx` 等 / active |
| [[reference/prefabs/icespike_fx_1|月矿采掘、旧冰刺与雕像转变]] | 9 注册：月矿采掘、四旧冰刺与两层雕像转变；旧冰刺无活跃生成点 | mining_moonglass_fx、mining_charged_moonglass_fx、mining_crystal_fx、icespike_fx_2、icespike_fx_3、icespike_fx_4、statue_transition、statue_transition_2 / active |

## 碎裂、换装与开包

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/shatter|一次性碎裂与爆裂表现族]] | 8 注册：冻结碎裂等级壳＋本地动画双端代理、蛛网溅落（无生成点诚实缺口）、贝壳小生物、木牌碎裂、南瓜炸弹确定性 PRNG 碎块、风滚草破碎、工具损坏代理、亮茄爆裂 | `shatter`、`splash_spiderweb`、`brokentool` 等 / active |
| [[reference/prefabs/junk_break_fx|退化、破裂与改造特效]] | 11 注册：草船与冰碎片、海冰、围栏、地皮、废料、箱子、打蜡与藤条 | degrade_fx_grass、degrade_fx_ice、fx_ice_pop、fence_rotator_fx、turf_smoke_fx、pull_smoke_fx、chestupgrade_stacksize_fx、chestupgrade_stacksize_taller_fx、beeswax_spray_fx、goo_vines_break_fx / active |
| [[reference/prefabs/fossilized_break_fx|化石碎裂与化石化特效]] | 四种化石表现；本地尘雾代理与碎裂淡出 | `fossilizing_fx`、`fossilizing_fx_1`、`fossilizing_fx_2` / active |
| [[reference/prefabs/petrified_trunk_break_fx|树木石化特效]] | 五种树木石化代理；阶段选择与灰度继承 | `petrified_tree_fx_short`、`petrified_tree_fx_normal`、`petrified_tree_fx_tall`、`petrified_tree_fx_old`、`petrify_fx` / active |
| [[reference/prefabs/mushtree_tall_webbed_burntfx|生物残骸与植物／菌类次级表现]] | 6 注册：牧者残骸、藤蔓节、烧尽菌树、焦灌木侵蚀、梦魇地裂与大血滴 | `lunar_grazer_debris`、`nightmaregrowth_crack`、`minotaur_blood_big` 等 / active |
| [[reference/prefabs/mushtree_tall_burntfx|蘑菇树焦壳与绽放焦壳]] | 7 注册：动态拼名全展开（tall/medium/small×普通/绽放＋月树）；烧尽与读档两条触发、双兜底清理 | `mushtree_tall_burntfx`、`mushtree_tall_bloom_burntfx`、`mushtree_moon_burntfx` 等 / active |
| [[reference/prefabs/messagebottle_break_fx|消息瓶与歌贝特效]] | 4 注册：消息瓶破碎与歌贝按平台材质选落点；浮动特效未找到生成点 | messagebottle_bob_fx、singingshell_creature_rockfx、singingshell_creature_woodfx / active |
| [[reference/prefabs/bundle_unwrap|开包特效（bundle_unwrap）]] | 开包展开烟 17 名：bundle.lua 按 inventoryimage 拼名，UNWRAP 动作战利品落点生成；龙舟赛对 yotd 直落点复用，quagmire 两名仅依赖声明 | gift_unwrap、redpouch_unwrap、redpouch_yotp_unwrap、redpouch_yotc_unwrap、redpouch_yotb_unwrap、redpouch_yot_catcoon_unwrap、redpouch_yotr_unwrap、redpouch_yotd_unwrap、redpouch_yoth_unwrap、yotc_seedpacket_unwrap、yotc_seedpacket_rare_unwrap、carnival_seedpacket_unwrap、wetpouch_unwrap、hermit_bundle_unwrap、quagmire_seedpacket_unwrap、quagmire_crate_unwrap、礼物开包、红包特效 / active |
| [[reference/prefabs/balloon_pop_body|气球爆裂（balloon_pop_body）]] | 两注册按装备槽选头部／身体变体（未装备选名 nil 不生成）；骑乘仅解除激活 | balloon_pop_head、气球爆裂、气球爆炸特效 / active |
| [[reference/prefabs/snowball_shatter_fx|雪球碎裂动画（snowball_shatter_fx）]] | 雪球原地消失碎裂 1 名：融尽、冬宴清场、雪人锤碎三路径；投掷命中是 splash_snow_fx | 雪球碎裂、雪球消失特效 / active |
| [[reference/prefabs/pumpkincarving_shatter_fx|南瓜雕刻碎裂动画（pumpkincarving_shatter_fx）]] | 雕刻定稿碎裂 1 名：pumpkincarvable／pumpkinhatcarvable 校验通过后落点，与换装代理是不同注册 | 南瓜雕刻碎裂、雕刻碎裂特效 / active |
| [[reference/prefabs/pumpkincarving_swap_fx|南瓜雕刻换装代理]] | UGC 数据代理；南瓜切口／雪人装饰贴符号 | `snowmandecorating_swap_fx`、`ugc_swap_fx` / active |

## 通用生成烟、灰尘与落地

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/spawn_fx_small|生成烟雾（spawn_fx_small）]] | 出生离场烟雾五注册；玩家进出的 medium_static 暂停时钟与 tiny 无生成点 | spawn_fx_tiny、spawn_fx_medium、spawn_fx_medium_static、spawn_fx_small_high、生成烟雾、出生烟雾 / active |
| [[reference/prefabs/round_puff_fx_sm|通用圆烟、滑行烟与木屑（round_puff 族）]] | 六注册：圆烟 sm/lg 标签选名（crabking 代理父挂并绑定符号）、slide_puff 四状态图、钻船木屑及 jump 无生成点 | round_puff_fx_lg、round_puff_fx_hi、slide_puff、wood_splinter_jump、wood_splinter_drill、圆烟、命中烟、滑行烟、木屑特效 / active |

返回 [[reference/prefabs/fx-index|FX 主题总索引]]、[[reference/prefabs/index|Prefab 索引]]。
