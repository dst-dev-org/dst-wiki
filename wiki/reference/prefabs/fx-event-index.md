---
title: "FX · 节庆、小游戏与事件模式"
created: "2026-11-07"
updated: "2026-11-07"
type: index
tags: [overview]
---

# FX · 节庆、小游戏与事件模式

万圣夜、冬季盛宴、狂欢节、舞台、熔炉和暴食。

## 季节节庆与小游戏

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/halloween_firepuff_1|万圣夜火焰绒与月绒烟（halloween_firepuff_1）]] | 7 注册：药水与 livingtree_root 被营火族取作燃料起焰烟（blueflame 分冷热、三选一、随火苗符号）＋月药水与月玻璃再生的月绒落点烟；纯表现 | halloween_firepuff_2、halloween_firepuff_3、halloween_firepuff_cold_1、halloween_firepuff_cold_2、halloween_firepuff_cold_3、halloween_moonpuff、万圣夜火焰绒、冷焰绒、月绒烟、药水燃料烟 / active |
| [[reference/prefabs/winters_feast_depletefood|冬季盛宴餐桌食物枯竭特效（winters_feast_depletefood）]] | 2 注册：开餐过程每名进食者 30% 概率延迟随机闪粒（1~5 号动画＋bloom）、食物耐久耗尽终局 burst（条目无声，终局音效由餐桌自播）；仅餐桌盛宴回调一处生成，落点不父挂 | winters_feast_food_depleted、冬季盛宴食物枯竭特效、盛宴餐桌特效、食物耗尽特效 / active |
| [[reference/prefabs/wintersfeastbuff_fx|冬宴buff与烤炉火]] | 2 注册：debuff 周期投放落点 FX（强度方向与直觉相反）＋烤炉两阶段火层（宿主调度、无 firefx 组件） | `wintersfeastbuff_fx`、`wintersfeastoven_fire` / active |
| [[reference/prefabs/carnival_confetti_fx|狂欢节庆典特效族（carnival_confetti_fx 等）]] | 7 注册：炮庆三连、按季选名的盲盒开包、射击投射物；彩纸另用于高尔夫与小鸡赛跑 | carnival_sparkle_fx、carnival_streamer_fx、carnival_unwrap_fx、carnival_unwrap_fx_s2、carnival_unwrap_fx_s3、carnivalgame_shooting_projectile_fx、狂欢节特效、狂欢节彩纸、彩带特效、开包特效 / active |
| [[reference/prefabs/confetti_fx|大赛庆祝彩纸]] | 双形状彩纸爆发；专服壳需补清理 | `rabbit_confetti_fx`、`confetti` / active |
| [[reference/prefabs/yotr_fightring_torch|枕头大战坑与兔人火炬]] | 4 注册＋placer：擂台、套装、钟与 8 支事件驱动火炬；火炬非持久 FX 壳 | `yotr_fightring_torch`、`yotr_fightring` 等 / active |
| [[reference/prefabs/marionette_appear_fx|舞台剧木偶出场与退场特效]] | 舞台木偶出退场；SetTime 与渐隐 | `marionette_disappear_fx`、`marionette_fx`、`marionette` / active |

## 事件模式、特殊传送与假掉落

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/lavaarena_portal_player_fx|熔炉登场、复活与场景余烬特效（lavaarena_portal_player_fx 等）]] | 6 注册：玩家登场、尸体复活、生物传送烟与格栅余烬；登场生成点在事件数据包 | lavaarena_player_revive_from_corpse_fx、lavaarena_creature_teleport_smoke_fx_2、lavaarena_creature_teleport_smoke_fx_3、ember_short_fx、熔炉传送门特效、尸体复活特效、出生烟、余烬特效 / active |
| [[reference/prefabs/lavaarena_creature_teleport_small_fx|熔炉生物传送出生特效]] | 熔炉六种出场表现；音量衰减与出生烟 | `lavaarena_creature_teleport_medium_fx`、`lavaarena_spawnerdecor_fx_small`、`lavaarena_spawnerdecor_fx_1`、`lavaarena_spawnerdecor_fx_2`、`lavaarena_spawnerdecor_fx_3`、`lavaarena_creature_spawn_fx` / active |
| [[reference/prefabs/lavaarena_beetletaur_fossilized_break_fx_right|熔炉独眼巨猪与犀牛兄弟化石碎裂特效]] | 熔炉 7 注册化石碎裂；侧向隐藏与 `_alt` 中断变体；生成点在缺失的 fossilizable 组件／事件包 | `beetletaur_fossilized_break_fx_right` 等 / active |
| [[reference/prefabs/quagmire_portal_player_fx|暴食远古大门玩家出场与撒盐特效族（quagmire_portal_player_fx）]] | 5 注册：大门出场、滴水、出水与撒盐盘碗；生成点在快照不含的事件数据包 | quagmire_portal_playerdrip_fx、quagmire_portal_player_splash_fx、quagmire_salting_plate_fx、quagmire_salting_bowl_fx、暴食传送门特效、暴食撒盐特效、盐盘特效、盐碗特效、远古大门特效 / active |
| [[reference/prefabs/monkeyisland_portal_fxloot|猴岛传送门假掉落]] | 传送门假掉落；加权抽取、满额兜底与落地续播 | `fxloot` / active |

返回 [[reference/prefabs/fx-index|FX 主题总索引]]、[[reference/prefabs/index|Prefab 索引]]。
