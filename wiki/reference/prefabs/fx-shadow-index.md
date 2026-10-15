---
title: "FX · 暗影生物、裂隙生物与附着"
created: "2026-11-07"
updated: "2026-11-07"
type: index
tags: [overview]
---

# FX · 暗影生物、裂隙生物与附着

影手、影柱、暗影棋子、海上暗影、拟态与恶液的局部表现。

## 暗影生物、裂隙与影手

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/shadow_pillar|暗影囚牢·影柱法术体系]] | 4 注册：影柱本体、地面 base FX、隐藏 root 辅助与 CLASSIFIED 扫描法术 | `shadow_pillar_base_fx`、`shadow_pillar_spell` 等 / active |
| [[reference/prefabs/shadow_despawn|暗影消散、舞台影手与暗影憎恶光束]] | 4 注册：影人消散、舞台影臂／影手与迷雾憎恶光束 | `stageusher_attackarm`、`shrouden_optic_blast_fx` 等 / active |
| [[reference/prefabs/shadow_bishop_fx|暗影棋子羽群与暗影裂隙散件]] | 4 注册：主教羽群（播完一轮自删）、裂隙门粒子（无自删）、墨荒替身（带真实物理）、寄生复活面具（小写 fx 标签异例） | `shadow_bishop_fx`、`shadowrift_portal_fx`、`shadowthrall_mouth_dupe_fx`、`shadowthrall_parasite_fx` / active |
| [[reference/prefabs/shadowhand_arm|暗影之手／恐惧之手的伸缩手臂]] | 2 注册：普通／遮蔽两套 build；固定起点朝手部拉伸，无自删，宿主负责清理；查理手部第二生成点 | `shadowhand_arm`、`shadowhand_arm_shrouded` / active |
| [[reference/prefabs/shrouden_eye_fx|暗影憎恶死亡水洼之眼]] | 1 注册：死亡水洼可检视眼睛；本地 goop 层＋animsync 同步，PUDDLE 专属检视文本，随状态／Boss 清理 | `shrouden_eye_fx` / active |
| [[reference/prefabs/oceanhorror_ripples|海上暗影生物涟漪与上船挂点]] | 2 注册：宿主驱动的网络化涟漪层（SG 驱动缩放、透明组件写 alpha）与随船平台跟随挂点（离船／移除／沉船三条清理路径） | `oceanhorror_ripples`、`oceanhorror_attachpivot` / active |
| [[reference/prefabs/dreadstone_spawn_fx|暗影融合体、绝望石与诅咒特效]] | 4 注册：融合体出现、绝望石变化、寄生附体和牛铃诅咒 | fused_shadeling_spawn_fx、shadowthrall_parasite_attach_poof_fx、beef_bell_shadow_cursefx、绝望石特效 / active |

## 拟态、恶液与生物附着

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/gelblob_attach_fx|恶液接触附着黏液]] | 黏液附着有减速；Connector 与共享注销 | — / active |
| [[reference/prefabs/itemmimic_revealed_shadow|拟态蠕虫影尾]] | 1 注册：单发射器恒定 1 粒的 VFXEffect、共享段非专服各建一份、net_event 开关协议 | `itemmimic_revealed_shadow` / active |
| [[reference/prefabs/spooked_spider_rock_fx|惊吓蜘蛛虫群特效]] | 工作惊吓双表现；组件触发与唤醒音效 | `spooked_worms_fx`、`spooked_fx` / active |
| [[reference/prefabs/gestalt_trail|宿主附着与跟随的外围 FX 族]] | 6 注册：虚影拖尾、埋地垃圾层、换装跟随层、恶液背层、暗影切斯特旋涡与雪球发射器光晕 | `daywalker2_buried_fx`、`gestalt_cage_swap_fx`、`shadow_chester_swirl_fx` 等 / active |

## 关联入口

独眼巨鹿、天体英雄、WAGboss 等遭遇见 [[reference/prefabs/fx-creature-index|生物、Boss 与遭遇演出]]。

返回 [[reference/prefabs/fx-index|FX 主题总索引]]、[[reference/prefabs/index|Prefab 索引]]。
