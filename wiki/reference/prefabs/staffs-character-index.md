---
title: "装备·法杖与角色道具"
created: "2026-11-07"
updated: "2026-11-07"
type: index
tags: [overview]
---

# 装备·法杖与角色道具

法杖、施法器与角色专属／活动装备的叶级索引；除特别标注外均为 HANDS 槽。按主要用途唯一登记，工具／武器等其他用法从文末“跨用途入口”进入。返回 [[reference/prefabs/equipment-index|装备主题总索引]]、[[reference/prefabs/index|Prefab 索引]]。

## 法杖与施法器

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/staffs|基础法杖族（icestaff 等 7 注册）]] | HANDS 槽；staff.lua/staff_common 共用工厂：冰杖三档冷量、火杖引燃与投喂、传送杖 telebase 链与洞穴 ms_miniquake、橙杖 blinkstaff、绿杖拆解与宝石不返还；kingcane 充能分支 | icestaff、icestaff2、icestaff3、firestaff、telestaff、orangestaff、greenstaff、冰魔杖、火魔杖、传送魔杖、懒人魔杖、拆解魔杖 / active |
| [[reference/prefabs/staffs-more|法杖补充族（staff_lunarplant 等 12 注册）]] | HANDS 槽；亮茄魔杖弹跳+套装直读、天气风向标召龙卷风、国王魔杖嵌宝石换效果（未腐化无次数/充能消耗，精神与腐化照算）、滑稽马疾驰、静电捕捉/实验/充能三态系统 | staff_lunarplant、staff_tornado、king_cane、yoth_knightstick、moonstorm_static_catcher、亮茄魔杖、天气风向标、国王魔杖、滑稽马、能量静电、静电约束仪 / active |
| [[reference/prefabs/yellowstaff|唤星者魔杖（yellowstaff）]] | HANDS 槽；施法物品：次数／精神消耗、生成 stafflight、月台转换与皮肤对应 | 唤星、矮星、唤星法杖、Star Caller's Staff、召唤矮星 / active |
| [[reference/prefabs/opalstaff|唤月者魔杖（opalstaff）]] | HANDS 槽；施法物品：50 次地点召唤、生成 staffcoldlight、来源与皮肤 build | 唤月、冷星、极光、唤月法杖、Moon Caller's Staff、召唤极光 / active |
| [[reference/prefabs/gnarwail_horn|一角鲸的角（gnarwail_horn）]] | HANDS 槽；施法器（非武器）：点海面召唤双浪、农田照料 20 范围；耐久 10、至少一道浪生成才扣 1；一角鲸掉落/船袭打断/沉箱保底；三叉戟配方原料 | gnarwail_horn、一角鲸的角、海号角、Gnarwail Horn、号角 / active |

## 角色专属与活动装备

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/dumbbells|哑铃族（dumbbell 等 7 注册）]] | HANDS 槽；沃尔夫冈体格器材：举重涨肌肉（LIFT_DUMBBELL→dumbbelllifter）、mighty 态可投掷 AOE、近战磨损默认消耗×2（受倍率/效率修正）；七档（石/金/大理石/宝石/热/火/冰），strongman 标签或技能门 | dumbbell、dumbbell_golden、dumbbell_marble、dumbbell_gem、dumbbell_heat、dumbbell_redgem、dumbbell_bluegem、哑铃、黄金哑铃、大理石哑铃、宝石哑铃、热铃、火铃、冰铃、举重 / active |
| [[reference/prefabs/pocketwatch|旺达怀表族（pocketwatch_heal 等 11 注册）]] | 警钟 HANDS；复活表点选幽灵，其余四表物品栏施法；时间机制工厂族：不老/第二次机会/倒走/溯源/裂缝五表＋警钟＋时间碎片＋钟表匠工具＋裂缝出入口；clockmaker 门禁、充能冷却、pocketwatch_inactive 标签语义反转；FX 注册归 warp_marker/weapon_fx 页 | pocketwatch_heal、pocketwatch_revive、pocketwatch_warp、pocketwatch_recall、pocketwatch_weapon、pocketwatch_parts、pocketwatch_dismantler、pocketwatch_portal、不老表、第二次机会表、倒走表、溯源表、裂缝表、警钟、旺达怀表 / active |
| [[reference/prefabs/wortox-nabbag|强抢袋（wortox_nabbag）]] | HANDS 槽；沃拓克斯专属手持装备（非容器）：weapon+tool(NET)+finiteuses 200、nabbag 锥形扫拾（150°/半径 3）、伤害按主物品栏占用格三档 13.6/23.8/34、灵魂加成至 2×；锥形扫拾按尝试次数扣耐久、扣满单次上限仍尝试；双层技能/标签门禁 | wortox_nabbag、强抢袋、Knabsack / active |
| [[reference/prefabs/role-tools|角色与活动专用工具族（26 注册）]] | HANDS/BODY 槽；枕头 4+4 双槽、costume 八套 BODY 服装（公主服 armor+setbonus）、火笔读 book_fire 充能、鱼人工具 restrictedtag 玩家不可穿、鞍角实为卸鞍、鱼食 20 秒诱鱼圈、迅捷气球 MAGIC 120 秒三段降速 | bodypillow、handpillow、costume、firepen、merm_tool、reskin_tool、saddlehorn、chum、bootleg、balloonspeed、balloon、火焰笔、鞍角、鱼食、迅捷气球、枕头盔甲、剧场服装、铁匠服、公主服 / active |
| [[reference/prefabs/character-gear-extra|角色专属装备补遗（伯尼、投泥带、电刑机）]] | HANDS 槽；三组角色专属选例（非全部）：伯尼 fuel↔health/Willow 限装、投泥带 FindItems 范围冷却（主栏/装备/手持/装备背包）与地形改造、WX78 遥控技能门禁与无人机 | bernie_inactive、bernie_active、wurt_swampitem_shadow、wurt_swampitem_lunar、wx78_drone_zap、wx78_drone_zap_remote、伯尼、伯尼！、投泥带、电刑机 / active |
| [[reference/prefabs/event-and-carnival-gear|Boss/事件/狂欢节装备（恐怖盾牌、派对气球、防腐喷雾、推杆）]] | HANDS 槽；恐怖盾牌自损/进食修复、派对气球随行与彩带 buff、防腐喷雾 waxspray 目标过滤、推杆 tee 生成不可制作 | shieldofterror、balloonparty、beeswax_spray、carnivalgame_golfclub、恐怖盾牌、派对气球、防腐喷雾、推杆 / active |

## 跨用途入口

- 基础法杖、补充法杖、国王魔杖、亮茄魔杖、冰火杖等的武器／工具侧见各页；武器选型总入口在 [[reference/prefabs/weapons-combat-index|武器与战斗]]。
- 唤星者魔杖与唤月者魔杖的生成实体仍归 [[reference/prefabs/index|Prefab 索引]] 的“召唤实体与附属变体”：[[reference/prefabs/stafflight|矮星（stafflight）]]、[[reference/prefabs/staffcoldlight|极光（staffcoldlight）]]。
- 一角鲸的角的召唤双浪与农田照料见本页；武器侧（三叉戟原料、海战打断）见 [[reference/prefabs/weapons-combat-index|武器与战斗]]。
- 旺达怀表的警钟武器、裂缝传送门与时间机制见本页；传送／移动工具入口见 [[reference/prefabs/tools-field-index|工具与野外实用]]。
- 哑铃的投掷 AOE 与近战磨损见本页；武器选型见 [[reference/prefabs/weapons-combat-index|武器与战斗]]。
- 强抢袋的锥形扫拾与武器侧见本页；野外拾取工具入口见 [[reference/prefabs/tools-field-index|工具与野外实用]]。
- 角色与活动专用工具族中的 BODY 服装／枕头与通用衣物、护甲的分工见 [[reference/prefabs/wearables-index|穿戴防具与饰品]]；火笔、鱼食等手持件仍以本页登记为准。
- 伯尼、投泥带、电刑机的角色门禁见本页；工具／武器侧分别见 [[reference/prefabs/tools-field-index|工具与野外实用]]、[[reference/prefabs/weapons-combat-index|武器与战斗]]。
- Boss／事件／狂欢节装备中恐怖盾牌的战斗侧见 [[reference/prefabs/weapons-combat-index|武器与战斗]]；派对气球、防腐喷雾、推杆的活动用法见本页。

返回 [[reference/prefabs/equipment-index|装备主题总索引]]、[[reference/prefabs/index|Prefab 索引]]。
