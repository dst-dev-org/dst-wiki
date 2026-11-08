---
title: "农业、栽种与地皮（暂行索引）"
created: "2026-11-10"
updated: "2026-11-10"
type: index
tags: [overview]
---

# 农业、栽种与地皮（暂行索引）

非装备物品栏 Prefab 扩库 B14 登记：8 页，60 个物品注册＋10 个放置器（turfs 40 个物品注册；其余 7 页 20 个物品注册＋10 个放置器）。正式的叶级细分留到该批全部轮次结束后统一整理。

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/fertilizer-materials|基础肥料四件（堆肥、鸟粪、粪肥、嗡嗡肥料）]] | 同壳四注册：部署只认农田土壤（三养分按格夹 0–100）＋扑灭阴燃＋45 秒燃料（quagmire 仅鸟粪／粪肥摘除取用回调但燃料值仍在；FX 缺失时堆肥／粪肥／嗡嗡肥料退回取用者坐标，鸟粪始终取取用者坐标）＋作祟只弹飞；差异为 300／450／300／300 秒与 0-24-0／0-0-16／0-0-8／12-12-12、鸟粪存档缩放与嗡嗡肥料制作二阶、部署后三阶另给三类各 8；来源含堆肥桶、牛／小牛／考拉象／猴子、蝙蝠族、鸟笼喂食（仅普通鸟无对应种子分支）与沃特技能配方 | 堆肥、鸟粪、粪肥、嗡嗡肥料、施肥 / active |
| [[reference/prefabs/treegrowthsolution|树果酱（treegrowthsolution）]] | 四类用途／五个动作入口（催长、农田 DEPLOY、旧 FERTILIZE、补漏、直修）；三处主扣件＋一处兜底：对树 ADVANCE_TREE_GROWTH（挡 monster／fire／stump／leif／no_force_grow）、农田部署 +8／+32／+8、船漏补丁走 repaired_treegrowth（两次 animover 自删、不设到期）、WOOD 材质 20 点直修；不可燃无燃料；配方 fig×2＋glommerfuel×1 | 树果酱、树生长溶液 / active |
| [[reference/prefabs/tree-seeds|可种植的树种（pinecone、twiggy_nut、palmcone_seed、marblebean）]] | addcone 工厂三注册＋独立 marblebean：PLANT 门禁（陆地非硬地、间距 2、禁建带与 IsDeployable 前置）、四 placer、树苗 225±75 秒长成常青树／多枝树／棕榈松果树／大理石灌木；堆叠 40；三个 addcone 15 秒燃料／7.5 秒燃尽，大理石豌豆无 fuel／burnable；棕榈松果树鳞片不可种；来源含树、浣猫、小浣猫托儿所与传送门 | 松果、多枝树种、棕榈松果树芽、大理石豌豆、种树 / active |
| [[reference/prefabs/special-tree-planting|活树根种植与树精召唤（livingtree_root、leif_idol；附 lunar_seed）]] | 只有活树根能种（PLANT→livingtree_sapling→livingtree_halloween，挖苗时机决定回根／回活木）；树精雕像靠伍迪技能，烧毁按每件 2 的上限（2×当时堆叠）、当燃料固定 2 个上限；最多预约 N 棵，常青树变树精、桦栗树延迟怪物化；候选不足只耗尽遍历并返回剩余额度；天体珠宝无 deployable，戴满珠宝帽且宠物槽未满时用地面充能月玻璃（3 件） | 完全正常的树、树精守卫雕像、天体珠宝、活树根 / active |
| [[reference/prefabs/plant-spawners|可放置的植物与巢卵（lureplantbulb、spidereggsack）]] | DEPLOYMODE.DEFAULT（可通行＋空位＋禁建带与 IsDeployable 前置）：食人花种子生成 lureplant，spawn 退出推 freshspawn→hibernate 960 秒才出芽；蜘蛛卵生成 1 级蜘蛛巢，开跑时首轮置零，后续 5±0.5 秒尝试放员；90±9 秒尝试补员（3 级容量 9、春季 11）；船上只停成长与再生，放员仍受开关／冻结／睡眠门禁 | 食人花种子、蜘蛛卵、食人花、蜘蛛巢 / active |
| [[reference/prefabs/plantables-remaining|尖刺灌木、石果灌木丛与猴尾草的移栽件（dug_marsh_bush、dug_rock_avocado_bush、dug_monkeytail）]] | plantables 剩余三项：堆叠 10／燃料 180／可燃 30、PLANT 部署（MEDIUM 1／DEFAULT 2）；挖起三分支（尖刺灌木恒给移栽件、石果丛枯萎只给 2 树枝、猴尾草枯萎改给芦苇）；回种分别 MakeEmpty 与 MakeBarren | 尖刺灌木、石果灌木丛、猴尾草、移植灌木 / active |
| [[reference/prefabs/turfs|地皮物品（turf_*，本版 40 个注册）]] | 40 条注册与地块键、TURF 模式＋gridplacer、只能铺在 DIRT（不做间距检查）、SetTile 后无条件扣件、Lua 不核实地块实际变化；堆叠 20／燃料 7.5／可燃 15／molebait；40 条同名配方＋沃特别名与天体 nounlock；叉子挖起来源 | 地皮、草皮、铺地皮、森林地皮、卵石路、棋盘地板 / active |
| [[reference/prefabs/waterplant-planter|海芽插穗（waterplant_planter）]] | upgrader（WATERPLANT）材料：升级海蚀柱／水草岩→原位海芽＋爆裂 FX；满月状态变化时，已满产且无待执行任务才预约；满月时长满也可预约（同样防重）；执行与读档重排不查月相；2 天重生后才产藤壶；无 deployable；唯一来源是击杀海草（燃尽回调不产） | 海芽插穗、海芽、在石柱上种植海草、barnacle_root / active |

返回 [[reference/prefabs/index|原版 Prefab 索引]]。
