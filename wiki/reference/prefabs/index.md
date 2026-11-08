---
title: 原版 Prefab 索引
created: 2026-10-29
updated: 2026-11-10
type: index
tags: [overview]
---

# 原版 Prefab

按注册名、中文名或俗称查本表及页面 aliases，按需阅读相关章节。完整造物示例见 [[cookbook/index|cookbook]]。

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
| [[reference/prefabs/equipment-index|装备 Prefab 主题索引]] | 工具、武器、法杖与角色道具、防具与饰品、背包与重物 | 装备索引、工具索引、武器索引、护甲索引、帽子索引、背包索引、重物搬运 / index |

普通背包见 [[reference/prefabs/storage-heavy-index|背包收纳]]，步行手杖见 [[reference/prefabs/tools-field-index|野外工具]]，唤星者与唤月者魔杖见 [[reference/prefabs/staffs-character-index|法杖]]。矮星与极光见召唤实体。

## 基础材料与常用物品（暂行）

非装备物品栏 Prefab 扩库 B01–B04 与补齐轮 B04-R 登记；正式的细分类留到该批全部轮次结束后统一整理。

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
| [[reference/prefabs/wild-berries|浆果与多汁浆果（berries、berries_juicy 与各自 _cooked）]] | 两个 SEEDLESS 键共导出 4 个注册；堆叠 40；保质 6／3 天与 2／1 天；两键写 BERRY 次级食性；生件可作 BERRY 档一次性海钓拟饵；多汁丛一次抛 3 个独立实体 | 浆果、烤浆果、多汁浆果、烤多汁浆果、浆果丛、多汁浆果丛 / active |
| [[reference/prefabs/tropical-gathered-fruits|香蕉、无花果与仙人掌肉（cave_banana、fig、cactus_meat 及熟态）]] | 6 注册；香蕉两态带 monkeyqueenbribe（女王进贡与火药猴索要）；无花果走 BERRY 次级食性＋hook_fig 拟饵但用 BERRY 档（FIG 档无引用）；仙人掌肉生 −3／−5 → badfood、熟 +1／+15、生熟同 10 天；洞穴香蕉树／香蕉丛／苔藓藤／仙人掌的采摘与再生 | 香蕉、熟香蕉、无花果、熟无花果、仙人掌肉、熟仙人掌肉 / active |
| [[reference/prefabs/kelp|海带叶三态（kelp、kelp_cooked、kelp_dried）]] | 3 注册；生 9.375／−1／−10、熟 9.375／0／−5、干 +1／+10；海带可作 KELP 材质修补材料（草筏／保险杠每份 +10）；dryable 接晾肉架 120 秒产干叶（20 天保质、不继承原料新鲜度）；珍珠多格架 driedsalticon 收盐；鱼人供奉锅只收 kelp | 海带叶、熟海带叶、干海带叶、海带 / active |
| [[reference/prefabs/mushrooms|蘑菇族八注册（red_cap、green_cap、blue_cap、moon_cap 与各自 _cooked）]] | 八注册（三色生熟六＋月亮两态）；红 −20 生命、绿 −50 理智、蓝 +20 生命与熟帽反转、统一 10 天保质、堆叠 40、熟帽燃料 7.5 与燃烧 3 秒；世界蘑菇相位门禁与雨计数再生（采摘与挖掘分开）、农场 6／4 产量与 4 收、月亮蘑菇的坐骑推送与进食者昏睡／困倦（3 秒只是时长参数） | 采摘的红蘑菇、熟红蘑菇、采摘的绿蘑菇、熟绿蘑菇、采摘的蓝蘑菇、熟蓝蘑菇、月亮蘑菇、熟月亮蘑菇、蘑菇、蘑菇族 / active |
| [[reference/prefabs/lichen-foliage|苔藓与蕨叶（cutlichen、foliage、quagmire_foliage_cooked）]] | 洞穴苔藓 12.5／3／−5、2 天保质并触发 badfood；蕨叶 0／1／0、6 天、可作燃料 7.5 并在晾肉架干成 foliage_dried（480 秒）；熟蕨叶是 Gorge 专用注册，组件与数值依赖不在快照的事件数据 | 苔藓、蕨叶、熟蕨叶、洞穴苔藓、蕨类 / active |
| [[reference/prefabs/petals|花瓣族（petals、petals_evil、cactus_flower）]] | 饥饿／生命／理智：花瓣 0／+1／0、深色花瓣 0／0／−5、仙人掌花 12.5／+8／+5；鲜度 6／6／3 天、堆叠 40、燃料 7.5／7.5／无；花瓣可干制、坟头装饰（Wendy tag 门禁）与花瓶／雪人装饰，作祟整叠 50% 转深色花瓣 | 花瓣、深色花瓣、仙人掌花、花瓣族、仙人掌开花 / active |
| [[reference/prefabs/weeds|杂草产物（forgetmelots、tillweed、firenettles）]] | 饥饿／生命／理智：必忘我 0／0／+1、犁地草 0／+1／0、火荨麻叶 0／−3／−5；鲜度 3／6／3 天、燃料 7.5／7.5／15；随机株 20% 抽杂草（权重 5／0.5×3，含 weed_ivy）、季节播种 .05／.15、蔓延与挖除再生；firenettle_toxin 温度 +60、60 秒并可重置 | 必忘我、犁地草、火荨麻叶、杂草 / active |
| [[reference/prefabs/acorn|桦栗果与巨石枝种子（acorn、acorn_cooked、tree_rock_seed 与两个放置器）]] | 桦栗果（不是橡果）9.375／1／0 与 20 天、烤桦栗果改 SEEDS+ROUGHAGE 与 6 天；PLANT 与 CUSTOM 两条种植门禁（桦栗果拒硬地、巨石枝种子不查 GROUND_HARD）、两种树苗生长链（225±75／1200±300 秒）、桦栗树与巨石枝树／巨大洞穴蠕虫掉落 | 桦栗果、烤桦栗果、巨石枝种子 / active |
| [[reference/prefabs/rock-avocado-fruit|石果与多肉植物（成熟石果、熟石果、发芽的石果与两个多肉注册）]] | 石果丛 4 阶段、外壳逐次采掘 65%／1%／34%；成熟石果 12.5／1／0 与 3 天、熟石果 12.5／3／0 与 2 天（熟制更不耐放）、发芽石果种下 1500 秒长成新丛；succulent_plant（世界态、不可入栏）与 succulent_picked（物品态、可堆叠 40、可干制）同名异注册 | 成熟石果、熟石果、发芽的石果、多肉植物、石果、石果灌木丛 / active |
| [[reference/prefabs/grass-saplings|草丛与树苗的移植件（dug_grass、dug_sapling、dug_sapling_moon）]] | `make_plantable` 公共厂：堆叠 10、燃料 180 秒、燃烧 30 秒、PLANT 落点要陆地且非硬地、mediumspacing 半径 1、部署即 `OnTransplant`；草与树苗的挖起与再生（`barren` 枯死与 `wither` 枯萎分开）、草蜥化两段计时链、月树苗四条来源（月岛映射、世界态转换、挖出件替换、沃姆伍德配方） | 草丛、树苗、月树苗、移植草丛、挖出的树苗 / active |
| [[reference/prefabs/berry-bushes|浆果丛与香蕉丛的移植件（四种植株与八个挖出件）]] | 再生 3／9 天＋每轮 0.5 天＋0~2 天随机、春季 ×0.75；轮次只在移植后递减、野生丛采不空；冬季暂停的是 `pickable` 而不是 `growable`，出冬也未必是春季；施肥只对枯死丛、枯萎或枯死挖起只给两把树枝不给挖出件；回种后 `MakeBarren` 需施肥；香蕉丛四段生长与读档 `ForceWither` | 浆果丛、多汁浆果丛、香蕉丛、移植浆果丛 / active |
| [[reference/prefabs/farm-plants|农田植株（15 个 farm_plant_〈作物〉注册）]] | 六阶段与三段时长按压力点数插值、七类压力四个检查点（≤1／≤6／≤11 分档）、**巨型是最终压力 NONE 的确定判定而非掷骰**、好季节减半只到三段成长期、`long_life` 只乘成熟期、收获四档掉落与腐烂分支、随机株两段且只有抽到作物才置 no_oversized | 农田植株、作物植株、胡萝卜植株、南瓜植株、种下的种子 / active |
| [[reference/prefabs/deciduous-trees|桦栗树（deciduoustree 六注册）]] | 短／中／高 450±150、1500±600 秒且春季 ×0.75；春夏秋叶色与冬季不落果；CHOP 5/10/15、怪物形态 18；怪物树按季节概率预约 3～4 棵（仍要过阶段门禁）并额外掉活木／噩梦燃料；烧毁后「等 10 秒自然掉落」与「先砍」两条路径各自重新掷机会表（每次另加 40% 木炭）、衰亡清理只在休眠时执行且总共只留一件、30 天±50% 树桩；自然扩散只在春季推进 | 桦栗树、桦栗树桩、毒桦栗树、桦栗树怪 / active |
| [[reference/prefabs/tree-rocks|巨石枝（tree_rock 系列七注册）]] | build 与阶段双随机后统一名字；砍倒时先换采掘组件、碰撞体暂缓切换（变石块后可采掘 6 次、记住进度），只给 1 根树枝＋最多 5 件按地形拓扑抽表的藤蔓；种子在正常采掘与崩落两套掉落表里都出现；8 天再生且 skipCanPlantCheck 只跳硬地；砸击 200／100；烧完同样变石块；作祟不会推进工作 | 巨石枝、巨石枝树 / active |
| [[reference/prefabs/bullkelp-plant|公牛海带与水下的叶片与冲岸根（bullkelp_plant、bullkelp_plant_leaves、bullkelp_beachedroot）]] | 3 天再生（春季 ×0.75）；被推上岸时先尝试摘叶（要过 canbepicked／caninteractwith／未卡住三道门禁，过不了就不掉叶）再原位换成茎、不生成冲岸根；水下叶片是 DECOR 子实体、persists=false；冲岸根拾取或入包时先移除原件，只有拿到接收容器才生成海带叶＋茎各一件并继承湿度 | 公牛海带、海带丛、冲岸的海带、公牛海带叶片 / active |
| [[reference/prefabs/cactus|仙人掌（cactus、oasis_cactus）]] | 同一工厂两注册、绿洲种改走名字覆盖；4 天再生；采摘刺 6 点且挡三类目标（无 combat、刺抗装备、暗影仆从）；花只在夏季由再生与唤醒置真、摘下追加给采摘者；无 workable 不可铲除移植；20 天世界再生与季节倍率 | 仙人掌、绿洲仙人掌、沙漠仙人掌 / active |
| [[reference/prefabs/flowers|花、邪恶花、荧光花与枯萎的花（flower 族十一注册）]] | 采摘即整株消失、1% 抽玫瑰变体、作祟 50% 转邪恶花；枯萎的花唯一来源是洞穴白天缺光；荧光花单／双／三株与三项枯萎版各自产光半径、产量与再生倍率，枯萎版产物是 spoiled_food；园艺再生要求「不下雨且地面仍湿」；玫瑰与种下的花也在同一工厂内返回 | 花、花丛、邪恶花、荧光花、枯萎的花、玫瑰 / active |
| [[reference/prefabs/oceanvine|苔藓藤条与补丁（oceanvine、oceanvine_patch）]] | NOBLOCK／flying 让船与建筑不被挡；无花果 2.5 天再生、摘空与结果两套动画；燃烧 2 秒内扑灭可救，之后自毁并手动掉 1 份灰；补藤阈值是 4 或 5 并用严格小于判断（已有 4 条只在抽到 5 时才补）；补丁在本构建检索不到任何生成点（四条证据） | 苔藓藤条、苔藓藤、藤条、无花果藤 / active |

## 农业、栽种与地皮

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/farming-and-planting-index|农业、栽种与地皮（暂行索引）]] | 8 页：肥料四件、树果酱、树种、特殊种植、植物与巢卵、移栽件、40 地皮、海芽插穗 | 肥料、树种、地皮、海芽 / index |

## 肉与加工食材（暂行）

非装备物品栏 Prefab 扩库 B05 登记；B01–B04 的材料入口见上节，正式的叶级细分留到该批全部轮次结束后统一整理。

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/meat|肉、熟肉与肉干及肉类共用工厂（meat、cookedmeat、meat_dried）]] | common() 共用工厂详述：标签与浮动体先于 SetPristine（两端），其余组件只在主模拟；三维 25／1／−10 与 6／10／20 天；猪王按金值 1 收（鱼人交付者先被拒）、晾肉架 2 天；腐坏倍率三档（正常 0／普通雨暂停恢复常规／酸雨 1+rate），定时完成不继承鲜度、烧架速干两者都不继承，干品另有查理舞台 weather 奖励，燃烧死亡可掉熟肉；作祟把整叠普通肉换成怪物肉 | 肉、熟肉、肉干、大肉 / active |
| [[reference/prefabs/smallmeat|小肉与熟小肉、小肉干（smallmeat、cookedsmallmeat、smallmeat_dried）]] | 三态显式堆叠 40；三维 12.5／0／−10 → 12.5／1／0 → 12.5／8／10；晾晒只用 1 天；符合燃烧掉落条件时可出熟小肉，架子烧毁可速干；作祟只抛飞、不换怪物肉 | 小肉、熟小肉、小肉干 / active |
| [[reference/prefabs/monstermeat|怪物肉三态（monstermeat、cookedmonstermeat、monstermeat_dried）]] | 负值三维 −20／18.75／−15 起；熟态保质 15 天反长于熟大肉；次要食性 MONSTER、金值 0 猪王拒收；生肉 selfstacker 半径 20 自动并堆；怪物值≥2 且无 inedible 才通过千层饼配方判定；示例优先级 40 象鼻 > 30 鞑靼／海鲜牛排 > 10 千层饼 | 怪物肉、熟怪物肉、怪物肉干 / active |
| [[reference/prefabs/drumstick|鸟腿与炸鸟腿（drumstick、drumstick_cooked）]] | 干制 1 天成小肉干、烤成炸鸟腿；炸鸟腿经 `cancook` 派生登记、可进锅只算半份肉；三维 12.5／0／−10；火鸡、秃鹫、企鹅与麋鹿鹅的掉落；符合燃烧掉落条件时可出炸鸟腿 | 鸟腿、炸鸟腿 / active |
| [[reference/prefabs/batwing|洞穴蝙蝠翅膀与裸露鼻孔（batwing、batwing_cooked、batnose、batnose_cooked）]] | 四注册同页对照；翅膀带漂浮与雪人装饰，鼻孔挂架与干好是两个 build；干制均成小肉干；蝙蝠 25%、裸鼹蝠约 1/3；符合燃烧掉落条件时可出熟翅膀／炭烤鼻孔 | 洞穴蝙蝠翅膀、熟蝙蝠翅膀、裸露鼻孔、炭烤鼻孔 / active |
| [[reference/prefabs/plantmeat|叶肉与熟叶肉（plantmeat、plantmeat_cooked）]] | 植物来源肉仍是 MEAT 食性；无 dryable 组件与标签，不能上晾肉架；食人花按展示诱饵与仆从数生成，另由两种小型海鱼 `_inv` 宰杀掉落，熟叶肉还接胡萝卜鼠与小鱼的 `cooking_product`；货架展示堆 ≤5 不提供消化候选、>5 才入候选，按名扣件仍可能扣到展示堆 | 叶肉、熟叶肉 / active |
| [[reference/prefabs/barnacle|藤壶与熟藤壶（barnacle、barnacle_cooked）]] | selfstacker 唤醒时自动合堆＋quickeat 优先于 sloweat 的进食状态；金值 0；建造配方只收生藤壶；海草收获上限 3、烧毁掉熟藤壶 | 藤壶、熟藤壶 / active |
| [[reference/prefabs/mitegland|火腺与熟火腺（mitegland、mitegland_cooked）]] | 熟制不延长保质；cattoy／rawmeat；生吃给非生肉食者挂火腺 debuff：温度修正量 +60、60 秒；无直接伤害但可越过过热阈值掉血（速率可被角色覆盖），烧着死亡可出熟火腺 | 火腺、熟火腺 / active |
| [[reference/prefabs/fishmeat|鱼肉两链六注册（生鱼肉、小鱼块及熟／干态）]] | 生品 3 天即腐坏成鱼专用 spoiled_fish／spoiled_fish_small；干品 15 天；烤制继承鲜度；海洋鱼按各鱼四张表（宰杀／重鱼／腐坏／上火烤）取值，如 `oceanfish_small_8` 宰杀直接给熟小鱼块、上火烤也是熟小鱼块，一角鲸／岩石大白鲨另有掉落 | 生鱼肉、鱼排、鱼肉干、小鱼块、熟小鱼块、小鱼干 / active |

## 蛋奶、蜂产品与野味水产（暂行）

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/animal-food-index|蛋奶、蜂产品与野味水产（暂行索引）]] | B06–B07 登记的 17 页：蛋类、奶与乳脂、蜂产品、蝶翅、蛙腿、象鼻、兔毛球、旧鱼族，以及海鱼、淡水鱼、龙虾与龙虾窝 | 蛋奶、蜂蜜、蛙腿、象鼻、旧鱼、海鱼、龙虾 / index |

## 月岛与特殊生物（暂行）

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/moon-island-index|月岛与特殊生物（暂行索引）]] | B07 登记的 4 页：月树、月树花、月蛾与中毒金丝雀 | 月树、月树花、月蛾、中毒金丝雀 / index |

## 材料、照明与特殊生物（暂行）

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/materials-consumables-index|材料、照明与特殊生物（暂行索引）]] | B08 登记的 11 页：冰与迷你冰川、荧光果与南瓜灯、格罗姆花、蘑菇树孢子、晶洞果与夜莓、果蝇族、月熠、小痒痒、双尾心 | 冰、荧光果、南瓜灯、格罗姆、孢子、晶洞果、果蝇王、月熠、小痒痒、双尾心 / index |
| [[reference/prefabs/crafting-materials-index|通用材料与加工中间件（暂行索引）]] | B10 的 8 页：常青树、灰烬与木炭、木料、发条零件、骨片、月岩、盐晶、蛛丝 | 常青树、灰烬、木炭、活木、齿轮、月岩、盐晶、蜘蛛丝 / index |
| [[reference/prefabs/animal-drops-index|动物掉落与巢材（暂行索引）]] | B11 的 15 页：毛发、皮革、羽毛、牙刺 | 动物掉落、毛发、羽毛、蜜脾 / index |

## 节庆、角色与特殊食品（暂行）

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/special-food-index|节庆、角色与特殊食品]] | B09 的 14 页，涵盖节庆供品、冬宴菜、角色食品、茶与干叶、围网鱼群和腐烂物 | 节庆食品、冬宴菜、营养砖、寄居蟹茶、干叶、腐烂物 / index |

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
