# 非装备物品栏 Prefab 分批执行表

工作依据为 [物品栏总台账](prefab-inventory-ledger-20261108.md) 与 [Steam build 26335394](../../../raw/current/BUILD.txt)。本表将 400 个写作候选入口与 23 个复用／待核验入口排入 **B01–B43**。141 个装备／宿主入口继续留在总台账 E 组。共享工厂按注册切片，各写作任务互斥。

B04、[B04-R 补齐 B01–B04](#b04-r-补齐-b01b04-的关联对象) 与 [Prefab 注册归属复检](#prefab-注册归属复检) 均已收口；B05 九页、27 个物品注册已于 2026-11-09 通过 Steam build 26474873 定向源码复验并收口。B06（13 页）、B07（8 页）已按用户点名落地、待外部复检；B08（11 页、21 个主注册）于 2026-11-10 通过 Steam build 26474873 [外部源码复验](prefab-inventory-review-b08-20261110.md#第三轮外部定向复验)并收口；B10（8 页、25 个主注册）于 2026-11-10 通过 Steam build 26474873 [外部源码复验](prefab-inventory-review-b10-20261110.md#第三轮外部定向复验)并收口，R01–R14 全部通过；B09（14 页，53 个物品注册＋1 个鱼群注册）于 2026-11-10 通过 Steam build 26474873 外部源码复检并收口，另记 8 个茶 buff 关联注册。B11（15 页、18 个物品注册＋1 个宿主）于 2026-11-10 通过 Steam build 26474873 [外部源码复验](prefab-inventory-review-b11-20261110.md#第三轮外部定向复验)并收口，R01–R13 全部通过；B12（6 页、10 个物品注册）于 2026-11-10 通过 Steam build 26474873 [外部源码复验](prefab-inventory-review-b12-20261110.md#第二轮外部定向复验)并收口，R01–R10 全部通过；B13（11 页、13 个物品主注册，关联复用既有 1 个 FX）于 2026-11-11 登记收口，已于 2026-11-10 通过 Steam build 26474873 [外部源码复验](prefab-inventory-review-b13-20261110.md#第二轮外部定向复验)，R01–R14 全部通过；B14（8 页、58 个物品注册＋2 个世界态主注册＋10 个放置器）于 2026-11-11 通过 Steam build 26474873 [外部源码复验](prefab-inventory-review-b14-20261111.md#第四轮外部定向复验)并收口，R01–R18 全部通过。原批次编号和物品栏统计口径保留，额外纳入的世界实体在对应批次单列。

## 每轮执行和交接

1. **认领与派发。** 按工作区 `AGENTS.md` 核对 BUILD、认领与写入范围，按下表 A／B／C 派发「目标页路径｜注册候选或展开线索｜已知源码入口｜已知复用页」。优先每个写作子代理 2–4 个主题页，复杂工厂可只派 1 页。超出当轮处理量就用 Bxx-a、Bxx-b 承接明确余项。
2. **并行写作。** 写作子代理按 `assets/prompts/new-pages.md` 与对应 Ingest／Prefab 指引，核实主注册、简中实显、已有同主题覆盖和直接相关宿主的归属。已有完整正文就复用，缺页且关系紧密就回报是否随本批承接。族页说明公共工厂和各变体差异。子代理只写已认领页面，交付时回报互链、索引草案与归属冲突，主代理协调共享入口。
3. **独立检视和修订。** 写作任务全部结束后，按工作区 `AGENTS.md` 分配未参与写作的检视者。检视者按 `assets/prompts/source-review.md` 只读核对源码，按页给出 P1／P2／P3、证据与验收条件。各组交回报告后立即按原编号把可独立处理的问题分回实际写作者，修订回执到达后交原检视者复验自己负责的页面。技术异议由检视者回源裁定。
4. **主代理机械验收。** 各组内部检视和所需复验全部通过后，定向跑页面／导航校验、严格 Lua／`.po` 源码链接和源码哈希检查。按 Skill 对新页和实际改动的来源登记基线，有完整 Lua 示例改动再做语法检查。修复有证据的问题后只重跑受影响项。主代理为已写成页面统一登记到可达的暂行叶索引并同步返回链，用 `--check-indexes` 验收。暂按粗分类登记，统一的正式索引细分留到总收尾。
5. **交外部复检。** 报告实际页和注册清单、复用／排除／待核验项、复审问题及处置、实际脚本结果，并在总台账把本轮标为「待外部复检」。用户把轮次交给本对话独立核对，修订确认后由本对话标为「已收口」。外部复检通过再派发下一轮。

两名写作任务已经足够时按两名派发，表中空栏跳过。遇到并发配额不足先等写作任务退出，再派独立检视。B42／B43 优先交分诊结果，有明确缺页再派写作，保留已覆盖或确实不适用的处置。

## B01–B09 常用物品、作物与保质物

| 轮次 | 主题 | 子代理 A | 子代理 B | 子代理 C |
|---|---|---|---|---|
| B01 | 首批材料、修补与非堆叠用具 | [I01–I03](prefab-inventory-ledger-20261108.md#首批可派发) | [I04–I06](prefab-inventory-ledger-20261108.md#首批可派发) | [I07–I09](prefab-inventory-ledger-20261108.md#首批可派发) |
| B02 | 农田作物生熟态 | 基础粮菜（[V-A](#共享工厂切片)） | 蔬果（[V-B](#共享工厂切片)） | 辛香作物（[V-C](#共享工厂切片)） |
| B03 | 种子、野采果实与海带 | 普通与作物专用种子（`seeds`、[V-S](#共享工厂切片)） | 野采果实与仙人掌食材（[V-W](#共享工厂切片)） | 海带生熟干制（[V-K](#共享工厂切片)） |
| B04 | 其他野采与种植材料 | 菌菇与叶类（`cutlichen`、`foliage`、`moon_mushroom`、`mushrooms`） | 花瓣、野草及毒性（`cactus`、`firenettles`、`forgetmelots`、`petals`、`petals_evil`、`tillweed`） | 种子、坚果及可栽种形态（`acorn`、`succulent_plant`、`rock_avocado_fruit`、`tree_rock_seed`） |
| B05 | 肉类生熟和晾晒变体 | 大肉、怪物肉、小肉（[M-A](#共享工厂切片)） | 陆地野味、海鲜边料与螨虫腺（[M-B](#共享工厂切片)） | 鱼肉、活动或受限肉品（[M-C](#共享工厂切片)） |
| B06 | 动物食材与蜂产品 | 蛋类与牛奶乳脂（`egg`、`tallbirdegg`、`goatmilk`、`butter`、`milkywhites`） | 蜂产品及虫翅（`honey`、`royal_jelly`、`butterflywings`、`moonbutterflywings`） | 禽腿、陆地食材与旧鱼族（`froglegs`、`trunk`、`hareball`、`fish`、`eel`） |
| B07 | 活体水产和条件腐坏 | 海鱼世界态与物品态（`oceanfish`） | 池鱼与龙虾（`pondfish`、`wobster`） | 中毒金丝雀、月蛾与月树花（`canary_poisoned`、`moonbutterfly`、`moontree_blossom`） |
| B08 | 保质材料、植物产物与特殊消耗 | 冰、照明与南瓜灯（`inv_rocks_ice`、`lightbulb`、`pumpkin_lantern`） | 花、果、孢子（`glommerflower`、`ancienttree_fruits`、`fruitfly`、`mushtree_spores`） | 风暴、蚊群及沃拓克斯道具（`moonstorm_spark`、`mosquitomusk`、`wortox_reviver`） |
| B09 | 节庆、角色与特殊食品 | 鸦年、猪年、兔年食物（`carnival_food`、`yotp_food`、`yotr_food`） | 角色、掉落与节庆餐桌食物（`crumbs`、`wx78_foodbrick`、`wintersfeastcookedfoods`、`hermitcrabtea`） | 耐放和腐烂产物（`antchovies`、`driedplants`、`dustmeringue`、`mandrake_inactive`、`phlegm`、`spoiledfood`、`woby_treat`） |

### 共享工厂切片

| 切片 | 源码入口 | 主注册范围 |
|---|---|---|
| V-A | `veggies.lua` | `carrot`、`corn`、`pumpkin`、`potato` 的原料与 `_cooked`，8 名 |
| V-B | `veggies.lua` | `eggplant`、`durian`、`pomegranate`、`dragonfruit`、`watermelon`、`tomato` 的原料与 `_cooked`，12 名 |
| V-C | `veggies.lua` | `asparagus`、`onion`、`garlic`、`pepper` 的原料与 `_cooked`，8 名 |
| V-S | `veggies.lua` | V-A／V-B／V-C 的 14 个 `_seeds`。B03-A 另领 `seeds.lua` 的通用种子变体 |
| V-W | `veggies.lua` | `berries`、`berries_juicy`、`cave_banana`、`fig`、`cactus_meat` 的原料与 `_cooked`，10 名 |
| V-K | `veggies.lua` | `kelp`、`kelp_cooked`、`kelp_dried`，3 名 |
| M-A | `meats.lua` | `meat`、`cookedmeat`、`meat_dried`、`monstermeat`、`cookedmonstermeat`、`monstermeat_dried`、`smallmeat`、`cookedsmallmeat`、`smallmeat_dried` |
| M-B | `meats.lua` | `drumstick`、`batwing`、`plantmeat`、`barnacle`、`batnose`、`mitegland` 及其 `_cooked`。螨虫腺效果实体只作关联入口 |
| M-C | `meats.lua` | `fishmeat`、`fishmeat_small` 及其 `_cooked`、`_dried`。另核对 `humanmeat`、`humanmeat_cooked`、`humanmeat_dried`、`quagmire_smallmeat`、`quagmire_cookedsmallmeat` 的玩法可达性，证据不足单列待核验 |

V 切片合计 55 名，巨型形态继续复用装备篇。成页前核实各分支的入栏和模式条件。同一转换链优先同页对照，食物与装备等独立查询入口可分开维护。分开的页面互链，并核对两边共用的转换和产出条件。旧鱼、池鱼和海鱼分别按实际注册名归属。

### B04-R 补齐 B01–B04 的关联对象

B04 已收口；2026-11-08 已派发九页，A 组 `grass-saplings`／`berry-bushes`，B 组 `farm-plants`／`deciduous-trees`／`tree-rocks`，C 组 `bullkelp-plant`／`cactus`／`flowers`／`oceanvine`。九页已落地，分三组独立复审并直修（P1×2、P2×15）。2026-11-09 按 Steam build 26474873 完成外部源码复检，九页及直接相关跨页断言复验通过，已收口。未运行游戏。按 B01–B04 的实际交付确认需新建、补写或复用，原轮验收继续有效。补齐轮交付和外部复检单独记入台账。

派发前按实际目标 BUILD 读取同版运行时清单，定向对账 `plantables.lua`、`veggies.lua`、海带植株与冲岸来源。按注册核对现有页正文与 B14／B29 移交项，将普通移栽、蜡封和预览分别分诊。26474873 的初次结果在 `output/coverage/prefab-26474873-b04-r/coverage-by-file.csv`，派发时以实际目标和当时页面为准。

| 任务 | 承接范围 | 交付边界 |
|---|---|---|
| A 草丛、树苗与浆果类移栽 | B01 的 `grass`、`sapling`／`sapling_moon`，B03 的 `berrybush`／`berrybush2`／`berrybush_juicy`、`bananabush`，以及各自普通 `dug_` 形态 | 从现有材料、浆果和香蕉页反查采集、挖起、部署与回种后的条件。浆果丛三种世界态和三种挖出态同一任务写齐，果实生熟数值留原页。核实普通预览接线并记录注册归属。 |
| B 农田植株与可栽种树木 | B02–B03 的 14 种 `farm_plant_<作物>`，B04 的桦栗果、巨石枝种子、石果与多肉、三种杂草 | 先复用种子、巨型作物和随机株正文。新增或补写农田植株的共用生长、压力与收获，按需对照各作物差异。核对 B04 实际交付已覆盖的世界态，补齐缺失的种下／生长／采收关系，优先核对 `deciduoustree` 与 `tree_rock1`／`tree_rock2`。 |
| C 公牛海带、仙人掌与其他采收宿主 | B03 的 `bullkelp_plant`、`bullkelp_beachedroot`、`oceanvine`，以及 B03–B04 的 `cactus`／`oasis_cactus`、世界态蘑菇和花丛 | 海带植株和冲岸形态同批处理，茎、叶和补丁页按需互链。仙人掌肉和花共用宿主页，核对采收、刺伤、再生与开花条件。世界态蘑菇、花丛和苔藓叶类先核对 B04 交付，补主要来源缺口。 |

A 组接手 `plantables.lua` 的上述普通移栽形态及相应部署预览，B14 负责剩余注册及公共部署逻辑归属，蜡封变体仍由 B14 与 B43 对账。C 组接手 `bullkelp_beachedroot`，B29 复用其验收结果。其余 B01–B04 页面逐组核对是否还有决定主要获取的宿主，登记当轮补写或有归属的后续项。例如木头的常青树主体随 B10 的木料链处理。交付逐注册列出主要页和跨页入口，核对挖出、回种、采收或替换条件，沿用本表的独立复审和定向校验流程。

### B05–B43 后续认领与跨轮归属

下表指定已知会跨轮撞写的主体。派发每个后续批次时，主代理先按当前源码和已验收页面补定该批主要来源的范围，再按实际工作量拆分。每个注册只由一个任务维护正文，关联物品各留自己的属性和用途。

| 主体首次承接 | 同批写作边界 | 后续复用 |
|---|---|---|
| B05 肉类／B07 水产 | 通用肉归纳获取方式。专属产物与决定其获取条件的宿主在同一任务中核验。B07 归属海鱼、池鱼和龙虾的实际世界态与物品态。 | B05 负责鱼肉加工和获得关系，B07 落地后回填对应水产页链接。 |
| B06 动物食材 | 随羊奶处理伏特羊 `lightninggoat`，随蝴蝶翅膀和黄油处理 `butterfly`，高脚鸟蛋与产蛋／护巢主体同组。蜂产品先定蜂群／蜂巢设施的主要产出入口。 | B12 羊角和 B20 捕获活蝶复用对应主体页。B07 的 `moonbutterfly` 是另一注册，B06 的月蛾翅膀和 B07 的活体主题相互补链。 |
| B08 花、果、孢子 | 格罗姆花与 `glommer` 同组，核对关联生成、跟随与产出。按实际玩法为孢子区分采集宿主、栽培设施和次要来源。 | B20 的黏液与 B25 的翅膀复用 `glommer` 主体。 |
| B10 木料与 B14 种植 | B10 承接 B01 木头关联的常青树，联核砍伐、木头与木炭。B14 核对松果、其余普通移栽和铺设的实际部署产物。 | B01 补来源入口，B14 的松果回链常青树，B14 普通移栽复用 B04-R A 的已验收切片。 |
| B11 牛毛及 B12 生物掉落 | 牛毛与皮弗娄牛 `beefalo` 同一任务分两页，核对剪毛、梳毛、再生及击杀掉落。B11–B12 其余专属掉落按实际产出主体分组。 | B21 剃刀、B25 牛铃与鞍、B34 装扮台各自维护道具或设施，关联到 B11 主体对应章节。 |
| B21–B34 捕获、绑定与部署 | 直接变成世界实体的包、可收放设备和被捕获的世界态按同一转换链分派。绑定对象或设施已有页优先复用。 | B29 的冲岸海带入口按 B04-R C 对账，保留其他树木与种植任务。 |
| B35–B43 料理和混合工厂 | 料理继续以基础菜与调味变体为单位，公共锅具与调味机制集中复用。混合工厂按主要注册归属核验其世界态或宿主。 | 食材和装备按已有章节引用，B42／B43 对账后再安排确实缺失的主体。 |

## B10–B20 堆叠材料、消耗品和活物

| 轮次 | 主题 | 子代理 A | 子代理 B | 子代理 C |
|---|---|---|---|---|
| B10 | 通用堆叠材料与剩余加工品 | 灰烬、炭、木料（`ash`、`charcoal`、`driftwood`、`livinglog`） | 矿材及机械中间件（`boneshard`、`gears`、`moonrocknugget`、`saltrock`、`transistor`） | 植物与虫类丝材（`silk`） |
| B11 | 动物掉落与巢材上 | 毛发、尾巴、皮与毛（`beardhair`、`beefalowool`、`coontail`、`furtuft`、`manrabbit_tail`、`pigskin`） | 羽毛和编织物（`feathers`、`goose_feather`、`malbatross_feather`、`malbatross_feathered_weave`） | 牙、刺与巢材（`honeycomb`、`houndstooth`、`stinger`、`tentaclespots`、`walrus_tusk`） |
| B12 | 动物掉落与巢材下 | 大型生物掉落（`bearger_fur`、`deerclops_eyeball`、`dragon_scales`、`minotaurhorn`） | 海陆生物外壳（`cookiecuttershell`、`slurtle_shellpieces`、`steelwool`） | 洞穴与稀有掉落（`lightninggoathorn`、`shroom_skin`、`slurperpelt`） |
| B13 | 远古、裂隙和机械材料 | 暗影与裂隙（`dreadstone`、`horrorfuel`、`nightmarefuel`、`voidcloth`） | 月亮、远古与宝库（`lunarplant_husk`、`purebrilliance`、`thulecite`、`thulecite_pieces`、`vault_orb_refined`） | 细粉和科技（`refined_dust`、`scandata`、`wagdrone_parts`、`wagpunk_bits`） |
| B14 | 农业、栽种与地皮 | 基础肥料与植物促长（`compost`、`guano`、`mosquitofertalizer`、`poop`、`treegrowthsolution`） | 播种、树木及繁殖（`leif_idol`、`livingtree_root`、`lunar_seed`、`lureplant_bulb`、`marblebean`、`palmcone_scale`、`pinecone`、`spidereggsack`） | 移栽与铺设（`plantables` 剩余注册及公共逻辑、`turfs`、`waterplant_planter`），与 B04-R A 对账 |
| B15 | 无新鲜度药剂与修补 | 治疗用品（`bandage`、`bandage_butterflywings`、`healingsalve`、`lifeinjector`、`tillweedsalve`） | 修补和状态药（`bathbomb`、`compostwrap`、`coolant`、`forge_repair_kits`、`ipecacsyrup`、`mosquitosack`、`sewing_tape`） | 蜘蛛与鱼人专用药剂（`mosquitomermsalve`、`spider_healer_item`、`spider_mutators`、`spidergland`） |
| B16 | 节庆药剂、点心及调味料 | 节庆药剂（`halloweenpotion_buffs`、`halloweenpotion_embers`、`halloweenpotion_moon`） | 糖果和冬季节庆耗材（`halloweencandy`、`wintersfeastfood`、`wintersfeastfuel`） | 调味料本体（`spices`） |
| B17 | 可堆叠弹药、投放及航海补给 | 爆破及弹药（`cannonballs`、`firecrackers`、`gunpowder`、`slingshotammo`、`slurtleslime`） | 水上设施和捕鱼配件（`blackflag`、`dock_kit`、`dock_woodposts`、`oceanfishingbobber`、`oceanfishinglure`） | 投放和触发（`rope_bridge`、`trap_fumarole`、`trap_starfish`、`townportaltalisman`） |
| B18 | 打包、生活用品与可投放建材 | 打包、休息与标识（`bedroll`、`beeswax`、`bundle`、`featherpencil`、`minisign`、`waxpaper`） | 栅栏和墙体（`fence`、`fence_electric`、`walls`） | 化石与机械建材（`fossil_piece`、`wagboss_robot_constructionsite`、`wagpunk_floor_kit`） |
| B19 | 货币、收藏与装饰 | 交易道具与饰品（`antliontrinket`、`cotl_trinket`、`hermithouse_ornament`、`horseshoe`、`pig_coin`、`pig_token`、`trinkets`） | 节庆票券、挂饰、玩偶和图样（`carnival_gametoken`、`carnival_prizeticket`、`halloween_ornaments`、`winter_ornaments`、`yotb_beefalo_dolls`、`yotb_pattern_fragments`、`yotr_token`） | 资料与任务消耗（`scrapbook_page`、`stash_map`、`wagstaff_tools`） |
| B20 | 堆叠型活物与角色消耗 | 捕获小动物（`bee`、`butterfly`、`fireflies`、`lightflier`、`mosquito`） | 角色材料与药剂（`ghostflower`、`ghostly_elixirs`、`glommerfuel`、`wereitems`、`willow_ember`） | 角色专用栏或状态门禁（`cursed_monkey_token`、`winona_holoitems`、`wortox_soul`） |

B14 实际承接名：`lureplantbulb`（文件 `lureplant_bulb.lua`）与 `palmcone_seed`（可种）；候选表中的 `palmcone_scale` 只是不可种的鳞片对照件。

P、S 或 N 代表优先归档方向。实际形态有差异时按组件接线改判，在原任务内记录变化。例如包装材料和装好物品的包、普通移栽和蜡封移栽、被捕获活物和世界形态都各查一次工厂及消费端。

## B21–B34 非堆叠物、任务物和部署包

| 轮次 | 主题 | 子代理 A | 子代理 B | 子代理 C |
|---|---|---|---|---|
| B21 | 非堆叠工具、陷阱与信号 | 陷阱与状态回收（`beemine`、`birdtrap`、`trap`、`trap_bramble`、`trap_teeth`） | 种植和物件维护（`carpentry_blades`、`farm_plow`、`fertilizer`、`pumpkincarver`、`razor`、`sewingkit`） | 角色交互和信号（`horn`、`houndwhistle`、`spider_repellent`、`spider_whistle`、`spiderden_bedazzler`） |
| B22 | 非堆叠容器、资料和摆件 | 物品容器与组件承载（`battlesong_container`、`beargerfur_sack`、`elixir_container`、`slingshotammo_container`、`slingshotparts`、`tacklecontainer`） | 学习、配方及图纸（`blueprint`、`constructionplans`、`cookbook`、`cookingrecipecard`、`mapscroll`、`scrapbook_notes`、`sketch`、`tacklesketch`、`yotb_blueprints`） | 可收放室内摆件及音频设备（`decor_flowervase`、`decor_lamp`、`decor_pictureframe`、`furniture_decor_items`、`phonograph`、`records`、`w_radio`） |
| B23 | 读物、角色道具与野外设备 | 读物、战歌和卡牌（`books`、`battlesongs`、`deck_of_cards`） | 伙伴和角色专用道具（`abigail_flower`、`balloons_empty`、`cattoy_mouse`、`wolfgang_whistle`、`wortox_souljar`） | 便携设备（`miniboatlantern`、`pocket_scale`、`portableblender`、`portablecookpot`、`portablefirepit`、`portablespicer`、`portabletent`） |
| B24 | 机械、自动化和角色装备边界 | 薇诺娜电力、照明与炮台包（`winona_battery_high`、`winona_battery_low`、`winona_catapult`、`winona_spotlight`） | 储存、传送及控制（`storage_robot`、`winona_remote`、`winona_storage_robot`、`winona_teleport_pad`） | WX-78 与瓦格斯塔夫库存物（`wagstaff_items`、`wx78_gestalttrapper`、`wx78_moduleremover`、`wx78_modules`、`wx78_scanner`） |
| B25 | 捕获、养殖与伙伴物品 | 绑定、随从与驯养（`beef_bell`、`chester_eyebone`、`glommerwings`、`hutch_fishbowl`、`kitcoon_nametag`、`saddle`） | 鸟类和抓获活物（`birds`、`birds_mutant`、`birds_mutant_rift`、`lightcrab`、`mole`、`rabbit`） | 熔岩虫、胡萝卜鼠及兔王（`carrat`、`lavae_cocoon`、`lavae_egg`、`lavae_tooth`、`rabbitking`、`rabbitkinghorn`） |
| B26 | 洞穴、远古、祭坛与影界任务物 | 档案馆与远古大门（`archive_lockbox`、`archive_resonator`、`atrium_key`、`atrium_ritual_organs`） | 遗迹、暗影与阶段道具（`nightmare_timepiece`、`reviver`、`ruinsrelic`、`shadowheart`、`shadowheart_infused`、`teleportato_parts`） | 墓碑与世界事件（`gravestone`、`graveurn`、`terrarium`、`playbill`） |
| B27 | 宝库、天体、隐士与解锁道具 | 宝库库存态（`vault_key`、`vault_pillar_guard_pieces`、`vault_teleporter`、`vaultrelic`） | 天体与月岩（`alterguardianhatshard`、`moonrockcrater`、`moonrockidol`、`moonrockseed`、`mooneye`、`security_pulse_cage`） | 隐士、扩容及季节钥匙（`chestupgrade_stacksize`、`deer_antler`、`hermit_pearl`、`hermitcrab_relocation_kit`、`hermitcrab_shell`、`hermithouse_laundry`、`singingshell`） |
| B28 | 船体、船上设备和信号 | 船体与收放状态（`boat`、`yotd_boats`、`oceanfishableflotsam`） | 航海设施及升级物（`boat_magnet_beacon`、`flotationcushion`、`mast`、`mastupgrade_lamp`、`mastupgrade_lightningrod`） | 灯与信号（`floatinglantern`、`megaflare`、`miniflare`） |
| B29 | 特殊种植、药剂与角色制品 | 树木、植物与挖出形态（`ancienttree_seed`、`ancienttrees`），复用核验 B04-R C（`bullkelp_beached`） | 短期功能物与消耗剂（`desiccant`、`nonslipgrit`、`pigtorch`） | — |
| B30 | 航海部署包 | 转向、船锚和转子（`anchor`、`boat_rotator`、`steeringwheel`） | 碰撞防护、火炮和磁铁（`boat_bumpers`、`boat_cannon`、`boat_magnet`） | 赛船包与拖网（`boatrace_common`、`ocean_trawler`） |
| B31 | 嘉年华设施、布置及观景 | 嘉年华经营设施（`carnival_cannons`、`carnival_plaza`、`carnival_prizebooth`） | 装饰包与季节变体（`carnival_decor`、`carnivaldecor_eggride`、`carnivaldecor_figure`、`carnivaldecor_lamp`） | 家具与营地点缀（`furniture_chairs`、`lantern_posts`） |
| B32 | 嘉年华小游戏部署包 | 喂食、赶鸡、记忆（`carnivalgame_feedchicks`、`carnivalgame_herding`、`carnivalgame_memory`） | 高尔夫三类包（`carnivalgame_golf_hole`、`carnivalgame_golf_tee`、`carnivalgame_golfgame`） | 冰球、射击和转盘（`carnivalgame_puckdrop`、`carnivalgame_shooting`、`carnivalgame_wheelspin`） |
| B33 | 年节、猫窝与宠物活动部署包 | 猫窝与鼠年设施（`kitcoondecor`、`kitcoonden`、`rat_gym`） | 牛年与兔年活动设施（`yotb_post`、`yotb_sewingmachine`、`yotb_stage`、`yotr_decor`） | 胡萝卜鼠赛道（`yotc_carrat_race_checkpoint`、`yotc_carrat_race_finish`、`yotc_carrat_race_start`、`yotc_carrat_scale`） |
| B34 | 功能结构部署包 | 铸造与远古工程（`lunar_forge`、`shadow_forge`、`vault_pillar_guard`） | 生物、收纳与角色设备（`beefalo_groomer`、`deerclopseyeball_sentryward`、`gelblob_storage`、`wx78_drone_delivery`） | — |

先查玩家槽位或活动栏的实际入口。任务发放、捕获、钓获、设施回收与内部装填分别核验。活动对象注明模式与启用条件。部署组核对包本体的堆叠参数、后置初始化、部署产物和销毁。部署后承担主要用途的建筑同批写作或复用，预览和附属 FX 按关联说明。

## B35–B41 锅料理与调味变体

每个表内基础键连同它的四个 `_spice_*` 注册由同一任务负责，七轮共 79 个基础键、395 个注册。按菜品机制合页，必要时同一任务拆为多页。B35 首个合适的料理页建立公共工厂、保质和调味入口，其余页面链接并保留本菜差异。统一调味机制只负责机制说明，注册名仍归各菜品页。B40 单独核对果冻豆无新鲜度、芝士蛋糕正保质期和湿腻焦糊的特殊匹配。B41 核对沃利锅的实际配方入口。

| 轮次 | 主题 | 子代理 A | 子代理 B | 子代理 C |
|---|---|---|---|---|
| B35 | 日常肉菜与常用肉类 | `meatballs`、`bonestew`、`perogies`、`baconeggs` | `honeynuggets`、`honeyham`、`kabobs`、`turkeydinner` | `frogglebunwich`、`frognewton`、`bunnystew`、`monsterlasagna` |
| B36 | 鱼鲜与藤壶 | `fishsticks`、`fishtacos`、`unagi`、`ceviche` | `californiaroll`、`seafoodgumbo`、`surfnturf`、`lobsterbisque`、`lobsterdinner` | `barnaclepita`、`barnaclesushi`、`barnaclinguine`、`barnaclestuffedfishhead` |
| B37 | 农作物料理 | `ratatouille`、`stuffedeggplant`、`mashedpotatoes`、`potatotornado` | `asparagussoup`、`vegstinger`、`salsa`、`pepperpopper` | `dragonpie`、`pumpkincookie`、`guacamole`、`veggieomlet` |
| B38 | 水果、点心与饮品 | `jammypreserves`、`fruitmedley`、`trailmix`、`waffles` | `icecream`、`watermelonicle`、`bananapop`、`frozenbananadaiquiri` | `bananajuice`、`sweettea`、`butterflymuffin`、`taffy` |
| B39 | 特殊食材、蛋食与特殊配方 | `leafloaf`、`leafymeatburger`、`leafymeatsouffle`、`meatysalad` | `koalefig_trunk`、`figatoni`、`figkabab` | `justeggs`、`talleggs`、`flowersalad`、`hotchili` |
| B40 | 效果与例外料理 | `jellybean`、`powcake`、`wetgoop` | `beefalofeed`、`beefalotreat` | `shroomcake`、`shroombait`、`mandrakesoup` |
| B41 | 沃利专属锅料理 | `nightmarepie`、`voltgoatjelly`、`glowberrymousse` | `frogfishbowl`、`dragonchilisalad`、`gazpacho`、`potatosouffle` | `monstertartare`、`freshfruitcrepes`、`bonesoup`、`moqueca` |

## B42–B43 复用、归属和混合工厂

| 轮次 | 主题 | 子代理 A | 子代理 B | 子代理 C |
|---|---|---|---|---|
| B42 | 已覆盖页与名称归属复核 | 装备页复用（`bernie_active`、`pocketwatch`、`pocketwatch_common`、`pocketwatch_dismantler`、`pocketwatch_parts`、`slingshotmodkit`） | 现页复用与跨章入口（`heatrock`、`yotr_fightring`、`messagebottle`、`moonstorm_static`） | 发光浆果独立主体归属（`wormlight`） |
| B43 | 混合工厂、间接挂载和内部载体 | 混合注册（`eyeturret`、`gestalt_cage`、`spider`、`waxed_plant_common`） | 角色、状态与共用 helper（`balloons_common`、`cozy_bunnyman`、`wagdrone_common`、`wx78_inventorycontainer`） | 剧情和内部道具（`charlierose`、`giftsurprise`、`temp_beta_msg`、`waxwelljournal`） |

两个核验包的交付是「复用／补写／新建／排除／待核验」以及按注册给出的依据。发现需要移交主要归属时，在同一批同步原页 aliases、正文和索引。例如发光浆果物品页落地前，先确认 FX 页承担的关联和需要保留的检索入口。确实不能从快照判断的条目写明限制，保留在台账。

## Prefab 注册归属复检

原始候选来自 26474873 同版对账 `output/coverage/prefab-26474873-shard-1-aligned/summary.json` 的 `coverage.alias_conflicts`，共 13 个注册。

- 主体名被关联页收录时，例如 `witchbroom`，确认主体页后把关联页改用章节链接。
- 两页都展开同一个注册时，例如 `houndstooth_proj`，先确定承接正文的页面，再同步别名、另一页的关联入口和索引。

逐注册记主页面或保留的依据，定向复验实际改动页。

2026-11-09 收口。先前交付已完成逐注册归位（13→0）。按当前工作树复验，13 个注册全部匹配且各有唯一主页面，关联入口可直达章节。逐项归属与复验见 [总台账](prefab-inventory-ledger-20261108.md#注册归属复检2026-11-09)。

## 全部轮次后的总收尾

- 以台账的实际验收页和注册名为输入，分别对账物品栏原候选及补充的关联对象，核对漏项、重复归属、待核验和有依据的排除。状态归档到项目报告目录，保留各轮依据、处置与验收链接。
- 按已落地页的规模和查询需求，将暂行条目整理为正式 Prefab 叶索引。物品保留「有新鲜度／无新鲜度可堆叠／无新鲜度不可堆叠」入口，类别多时再按用途拆叶。独立的世界态主体按植物、生物或设施等实际类型归档，混合族按主用途只登记一次，其他入口走互链。同步父入口、页尾返回链及原链接，保留根索引现有 Top 入口。
- 集中复检新增与实际受影响页，验收全库导航，复跑严格源码链接和哈希基线，确认名称、摘要、别名和索引一致。用玩家实际搜索词抽验入口，必要时定向刷新查询索引。由本对话做最后独立复检和整体收口。
