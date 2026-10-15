---
title: 原版 Prefab 索引
created: 2026-10-29
updated: 2026-11-08
type: index
tags: [overview]
---

# 原版 Prefab

已知注册名可直查 `reference/prefabs/<注册名>.md`；中文名和俗称先查本表或页面 aliases，再读命中页的相关章节及验证记录。页面描述原版工厂与一层关联导航，完整造物工程仍从 [[cookbook/index|cookbook]] 选择。

## 温度物品与容器

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/heatrock|暖石（heatrock）]] | 温差分档、携带／地面换热、灯光归属与冷热循环耐久 | Thermal Stone、暖石发光、暖石耐久、携带取暖、暖石降温 / active |
| [[reference/prefabs/icebox|冰箱（icebox）]] | 九格过滤、fridge 保鲜／暖石冷却、锤击与放置预览、皮肤 FX 生成与节流 | Ice Box、冰箱保鲜、冰箱放暖石、暖石冷却、冰箱收纳条件；冰冷坟墓、水晶冰箱、知更鸟蛋冰箱、陶瓷冰箱、维多利亚冰箱 / active |

## 生物与生成行为

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/krampus|坎普斯（krampus）]] | 淘气值生成入口、偷物砸箱、Brain／SG 分工、离场及掉落 | 小偷、Krampus、坎普斯偷东西、小偷砸箱子、坎普斯掉落 / active |

## 装备（按用途）

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/equipment-index|装备 Prefab 主题索引]] | 装备主题总入口：工具与野外实用、武器与战斗、法杖与角色道具、穿戴防具与饰品、背包收纳与重物搬运五个叶索引；含槽位、别名与需求摘要 | 装备索引、工具索引、武器索引、护甲索引、帽子索引、背包索引、重物搬运 / index |

普通背包、步行手杖、唤星者魔杖与唤月者魔杖已分别迁入 [[reference/prefabs/storage-heavy-index|背包收纳与重物搬运]]、[[reference/prefabs/tools-field-index|工具与野外实用]]、[[reference/prefabs/staffs-character-index|法杖与角色道具]]；本索引不再重复登记。矮星与极光等生成物仍见下节。

## 基础材料与常用物品（暂行）

非装备物品栏 Prefab 扩库 B01–B03 登记；正式的细分类留到该批全部轮次结束后统一整理。

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/basic-gathered-materials|常用采集材料（cutgrass、twigs、cutreeds、log）]] | 采集物 SetUp 产品经 PICK 动作尝试 GiveItem 入栏（栏满仍可能掉地）、地上物品才走 PICKUP、堆叠 40／log 默认 20、ROUGHAGE／WOOD 可食数值、猪王 tradable 按 goldvalue 收 | 采下的草、树枝、采下的芦苇、木头、采集材料 / active |
| [[reference/prefabs/basic-refined-materials|基础精炼材料（boards、cutstone、rope、papyrus）]] | SCIENCE_ONE 主配方与 Woodie／Walter 技能门禁替代配方、cutstone 不可燃 SetSinks、rope 的 canrope 目标极少、rocktribute 蚁狮进贡 | 木板、石砖、绳子、莎草纸、精炼材料 / active |
| [[reference/prefabs/basic-mineral-materials|矿物原料与幸运金块（flint、nitre、rocks、marble、goldnugget、lucky_goldnugget）]] | 注册名与文件名（inv_rocks.lua／inv_marble.lua）易混分流、巨砾与大理石树 loot 获得、goldvalue 与猪王兑换、lucky_goldnugget 仅 YOT 活动 | 燧石、硝石、石头、大理石、金块、幸运金块、矿物 / active |
| [[reference/prefabs/gems|宝石族（purplegem、bluegem、redgem、orangegem、yellowgem、greengem、opalpreciousgem）]] | buildgem 单工厂七注册零组件差异、ELEMENTAL 食性与鼹鼠 bait、GEM 修复无原版消费者、蟹王嵌彩与档案馆开关 | 紫宝石、蓝宝石、红宝石、橙宝石、黄宝石、绿宝石、彩虹宝石、宝石 / active |
| [[reference/prefabs/boat-patches|船体修补物（boatpatch、boatpatch_kelp）]] | 两态可堆叠 20、海带补丁 10 天腐坏成 spoiled_food、REPAIR_LEAK 标签门禁与 repaired_kelp 临时态回漏 | 船补丁、海带补丁、船体修补 / active |
| [[reference/prefabs/moonglass-materials|月亮碎片与注能月亮碎片（moonglass、moonglass_charged）]] | 两态共用 createglass 工厂、charged 1.5 天耗尽替换 plain、月暴闪电劈岩与月雹获得、王冠种植扣 3 召唤守卫 | 月亮碎片、注能月亮碎片、月光玻璃 / active |
| [[reference/prefabs/soil-amender|催长剂起子与超级催长剂（soil_amender、soil_amender_fermented）]] | 两态不可堆叠、perishable 3 日发酵计时（非食材腐坏）、部署施肥与 FERTILIZE 旧路径、施用返还空瓶 | 催长剂起子、正在发酵的催长剂、已发酵催长剂、超级催长剂、催长剂 / active |
| [[reference/prefabs/panflute|排箫（panflute）]] | finiteuses 10 次、半径 15 听众 sleeper／farmplanttendable／grogginess 分流、骑乘不阻断自身昏沉、沃拓克斯三技能接线 | 排箫、潘笛 / active |
| [[reference/prefabs/cooling-fans|羽毛扇与幸运扇（featherfan、perdfan）]] | 同工厂 15／9 次、半径 7 灭火＋体温向 2.5 度拉、幸运扇吟唱 2 秒龙卷风、perdshrine 幸运供奉制作站 | 羽毛扇、幸运扇、扇子 / active |
| [[reference/prefabs/farm-crops-staple|基础粮菜作物（carrot、corn、pumpkin、potato 与各自 _cooked）]] | 同一 MakeVeggie 工厂：MakeVegStats 的 16 位参数、导出集合与端分界、生熟三维／10·6·15 天保质、堆叠 40 与南瓜生件默认 20、农田压力分档与野生 carrot_planted 获得 | 胡萝卜、玉米、南瓜、土豆、烤胡萝卜、爆米花、烤南瓜、烤土豆、基础粮菜 / active |
| [[reference/prefabs/farm-crops-fruit|蔬果作物（eggplant、durian、pomegranate、dragonfruit、watermelon、tomato 与各自 _cooked）]] | 12 注册；三件生件堆叠落默认 20、榴莲锅内 fruit＋monster 与可食侧 MONSTER、西瓜生食带 −40／5 秒温差项（速率再按环境与保温换算）、火龙果被月亮精华液变异成生物、番茄无条件换 Gorge 图标 | 茄子、榴莲、石榴、火龙果、西瓜、番茄、烤茄子、超臭榴莲、切片熟石榴、熟火龙果、烤西瓜、烤番茄、蔬果作物 / active |
| [[reference/prefabs/farm-crops-spice|辛香作物（asparagus、onion、garlic、pepper 与各自 _cooked）]] | 8 注册；辣椒生 −20 生命／−15 理智、洋葱与大蒜生 −10 理智、芦笋熟制饥饿翻倍、洋葱无条件换 Gorge 图标、芦笋与辣椒可作雪人装饰 | 芦笋、洋葱、大蒜、辣椒、烤芦笋、烤洋葱、烤大蒜、烤辣椒、辛香作物 / active |
| [[reference/prefabs/farm-crop-seeds|农田作物专用种子（14 个 <作物>_seeds）]] | 14 注册＝VEGGIES 20 项 − SEEDLESS 6 项；`fn_seeds` 的公共端与主模拟分界、9.375 饥饿／0.5 生命／0 理智、堆叠 40 与 40 天保质、烤制统一成通用 seeds_cooked、陷阱诱饵＋SEED 海钓拟饵、三条播种链（farmplantable／Wormwood CUSTOM 部署／已废弃 plantable）、巨型敲碎与普通笼中鸟产种 | 椭圆形种子、簇状种子、尖种子、脆籽荚、风刮来的种子、作物种子、农田种子 / active |
| [[reference/prefabs/generic-seeds|通用种子与烤种子（seeds、seeds_cooked）]] | `common()` 的生熟分界与 4.6875／0／0＋40 天 vs 4.6875／1／0＋10 天；随机株 farm_plant_randomseed 两段生长（360–480 秒，秋春减半）＋20% 杂草＋按权重抽 14 种作物；随机株 loot 空表不返种；鸟笼／果蝇／风滚草／种子包获得；seeds_placer 用 tile_outline 预览 | 种子、烤种子、随机种子、通用种子 / active |
| [[reference/prefabs/wild-berries|浆果与多汁浆果（berries、berries_juicy 与各自 _cooked）]] | 两个 SEEDLESS 键共导出 4 个注册；堆叠 40；保质 6／3 天与 2／1 天（`berries_juicy_cooked` 是全表唯一 PERISH_ONE_DAY）；两键写 BERRY 次级食性；生件可作 BERRY 档一次性海钓拟饵（hook_berries／hook_juiceberries）；多汁丛一次抛 3 个独立实体、`cycles_left` 只在移植后扣 | 浆果、烤浆果、多汁浆果、烤多汁浆果、浆果丛、多汁浆果丛 / active |
| [[reference/prefabs/tropical-gathered-fruits|香蕉、无花果与仙人掌肉（cave_banana、fig、cactus_meat 及熟态）]] | 6 注册；香蕉两态带 monkeyqueenbribe（女王进贡与火药猴索要）；无花果走 BERRY 次级食性＋hook_fig 拟饵但用 BERRY 档（FIG 档无引用）；仙人掌肉生 −3／−5 → badfood、熟 +1／+15、生熟同 10 天；洞穴香蕉树／香蕉丛／苔藓藤／仙人掌的采摘与再生 | 香蕉、熟香蕉、无花果、熟无花果、仙人掌肉、熟仙人掌肉 / active |
| [[reference/prefabs/kelp|海带叶三态（kelp、kelp_cooked、kelp_dried）]] | 3 注册；生 9.375／−1／−10、熟 9.375／0／−5、干 +1／+10；海带可作 KELP 材质修补材料（草筏／保险杠每份 +10）；dryable 接晾肉架 120 秒产干叶（20 天保质、不继承原料新鲜度）；珍珠多格架 driedsalticon 收盐；鱼人供奉锅只收 kelp | 海带叶、熟海带叶、干海带叶、海带 / active |

## 召唤实体与附属变体

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/stafflight|矮星（stafflight）]] | 生成实体：照明、供热、烹饪、传播火势、精神光环和定时熄灭 | Dwarf Star、矮星光源、矮星持续时间、矮星供热、矮星烹饪 / active |
| [[reference/prefabs/staffcoldlight|极光（staffcoldlight）]] | 生成实体：制冷／精神光环／持续时间；同页区分月台附属 staffcoldlightfx | 冷星、Polar Light、极光持续时间、冷星降温、staffcoldlightfx、月台冷星特效 / active |

## 特效实体（FX）

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/fx-index|FX 主题总索引]] | 13 个主题、279 篇 FX 相关 Prefab 页；覆盖皮肤、施法、战斗、生物、环境、航海、装置、采集与节庆 | fx、特效、VFX、粒子、皮肤、水花、命中火花 / index |

“小偷”按用户提供映射收录，其他语境可能指不同对象。暖石、背包、冰箱已确认无额外俗称。各页为源码审阅，未运行游戏。

“矮星／冷星／极光”可能同时指施法物品与产物；查次数、施法、转换选法杖页，查温度、光照、持续时间选生成实体页。皮肤映射见主体，特效参数与回收见 [[reference/prefabs/fx-skin-index|皮肤与装备挂件]]。

返回 [[reference/index|机制与接口参考]]。
