# 非装备物品栏 Prefab 分批执行表

工作依据为 [物品栏总台账](prefab-inventory-ledger-20261108.md) 与 [Steam build 26335394](../../../raw/current/BUILD.txt)。本表将 400 个写作候选入口与 23 个复用／待核验入口排入 **B01–B43**。141 个装备／宿主入口继续留在总台账 E 组。共享工厂按注册切片，各写作任务互斥。

## 每轮执行和交接

1. **认领与定页。** 主代理确认实际 BUILD、台账状态、现有页和其他 Agent 的写入范围。按下表 A／B／C 拆成 2–3 个写作任务。每个任务先固定「页路径｜实际注册名或展开规则｜源码入口｜需复用的既有页」。已有正文覆盖的注册填复用，门禁或获得路径无法确认的填待核验。优先每个子代理 2–4 个主题页，复杂工厂可只派 1 页。实测超出当轮处理量就用 Bxx-a、Bxx-b 承接同一轮的明确余项。
2. **并行写作。** 写作子代理加载 `dst-wiki-skill`，按 `assets/prompts/new-pages.md` 与对应 Ingest／Prefab 指引写页。跨页互链和共享文件交主代理，写作代理只改自己认领的页。族页说明公共工厂和各变体差异。玩家可见名称按简中实显或实际显示路径核验，可用工具箱名称定位器辅助。
3. **独立检视和修订。** 写作任务结束后，开一名未参与本轮写作的检视子代理，按 `assets/prompts/source-review.md` 只读审阅整轮，逐页核对关键逻辑、端／状态门禁、堆叠与新鲜度、获得／移除、标题和主名称。按页列 P1/P2/P3 和源码依据。主代理回源码复核发现，确认修订范围后交原写作者或指定修订代理改页。复核受影响的摘要、索引行及正文同义表述。
4. **主代理机械验收。** 定向跑页面／导航校验、严格 Lua／`.po` 源码链接和源码哈希检查。按 Skill 对新页和实际改动的来源登记基线，有完整 Lua 示例改动再做语法检查。修复有证据的问题后只重跑受影响项。主代理为已写成页面统一登记到可达的暂行叶索引并同步返回链，用 `--check-indexes` 验收。暂按粗分类登记，统一的正式索引细分留到总收尾。
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

V 切片合计 55 名，巨型形态继续由装备篇维护。肉类切片仅锁定注册归属。成页前核实分支是否实际进入玩家物品栏、是否在相应模式可得。旧鱼、池鱼和海鱼分别按实际注册名分配，交叉引用采用已确认的主体页。

## B10–B20 堆叠材料、消耗品和活物

| 轮次 | 主题 | 子代理 A | 子代理 B | 子代理 C |
|---|---|---|---|---|
| B10 | 通用堆叠材料与剩余加工品 | 灰烬、炭、木料（`ash`、`charcoal`、`driftwood`、`livinglog`） | 矿材及机械中间件（`boneshard`、`gears`、`moonrocknugget`、`saltrock`、`transistor`） | 植物与虫类丝材（`silk`） |
| B11 | 动物掉落与巢材上 | 毛发、尾巴、皮与毛（`beardhair`、`beefalowool`、`coontail`、`furtuft`、`manrabbit_tail`、`pigskin`） | 羽毛和编织物（`feathers`、`goose_feather`、`malbatross_feather`、`malbatross_feathered_weave`） | 牙、刺与巢材（`honeycomb`、`houndstooth`、`stinger`、`tentaclespots`、`walrus_tusk`） |
| B12 | 动物掉落与巢材下 | 大型生物掉落（`bearger_fur`、`deerclops_eyeball`、`dragon_scales`、`minotaurhorn`） | 海陆生物外壳（`cookiecuttershell`、`slurtle_shellpieces`、`steelwool`） | 洞穴与稀有掉落（`lightninggoathorn`、`shroom_skin`、`slurperpelt`） |
| B13 | 远古、裂隙和机械材料 | 暗影与裂隙（`dreadstone`、`horrorfuel`、`nightmarefuel`、`voidcloth`） | 月亮、远古与宝库（`lunarplant_husk`、`purebrilliance`、`thulecite`、`thulecite_pieces`、`vault_orb_refined`） | 细粉和科技（`refined_dust`、`scandata`、`wagdrone_parts`、`wagpunk_bits`） |
| B14 | 农业、栽种与地皮 | 基础肥料与植物促长（`compost`、`guano`、`mosquitofertalizer`、`poop`、`treegrowthsolution`） | 播种、树木及繁殖（`leif_idol`、`livingtree_root`、`lunar_seed`、`lureplant_bulb`、`marblebean`、`palmcone_scale`、`pinecone`、`spidereggsack`） | 移栽与铺设（`plantables`、`turfs`、`waterplant_planter`） |
| B15 | 无新鲜度药剂与修补 | 治疗用品（`bandage`、`bandage_butterflywings`、`healingsalve`、`lifeinjector`、`tillweedsalve`） | 修补和状态药（`bathbomb`、`compostwrap`、`coolant`、`forge_repair_kits`、`ipecacsyrup`、`mosquitosack`、`sewing_tape`） | 蜘蛛与鱼人专用药剂（`mosquitomermsalve`、`spider_healer_item`、`spider_mutators`、`spidergland`） |
| B16 | 节庆药剂、点心及调味料 | 节庆药剂（`halloweenpotion_buffs`、`halloweenpotion_embers`、`halloweenpotion_moon`） | 糖果和冬季节庆耗材（`halloweencandy`、`wintersfeastfood`、`wintersfeastfuel`） | 调味料本体（`spices`） |
| B17 | 可堆叠弹药、投放及航海补给 | 爆破及弹药（`cannonballs`、`firecrackers`、`gunpowder`、`slingshotammo`、`slurtleslime`） | 水上设施和捕鱼配件（`blackflag`、`dock_kit`、`dock_woodposts`、`oceanfishingbobber`、`oceanfishinglure`） | 投放和触发（`rope_bridge`、`trap_fumarole`、`trap_starfish`、`townportaltalisman`） |
| B18 | 打包、生活用品与可投放建材 | 打包、休息与标识（`bedroll`、`beeswax`、`bundle`、`featherpencil`、`minisign`、`waxpaper`） | 栅栏和墙体（`fence`、`fence_electric`、`walls`） | 化石与机械建材（`fossil_piece`、`wagboss_robot_constructionsite`、`wagpunk_floor_kit`） |
| B19 | 货币、收藏与装饰 | 交易道具与饰品（`antliontrinket`、`cotl_trinket`、`hermithouse_ornament`、`horseshoe`、`pig_coin`、`pig_token`、`trinkets`） | 节庆票券、挂饰、玩偶和图样（`carnival_gametoken`、`carnival_prizeticket`、`halloween_ornaments`、`winter_ornaments`、`yotb_beefalo_dolls`、`yotb_pattern_fragments`、`yotr_token`） | 资料与任务消耗（`scrapbook_page`、`stash_map`、`wagstaff_tools`） |
| B20 | 堆叠型活物与角色消耗 | 捕获小动物（`bee`、`butterfly`、`fireflies`、`lightflier`、`mosquito`） | 角色材料与药剂（`ghostflower`、`ghostly_elixirs`、`glommerfuel`、`wereitems`、`willow_ember`） | 角色专用栏或状态门禁（`cursed_monkey_token`、`winona_holoitems`、`wortox_soul`） |

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
| B29 | 特殊种植、药剂与角色制品 | 树木、植物与挖出形态（`ancienttree_seed`、`ancienttrees`、`bullkelp_beached`） | 短期功能物与消耗剂（`desiccant`、`nonslipgrit`、`pigtorch`） | — |
| B30 | 航海部署包 | 转向、船锚和转子（`anchor`、`boat_rotator`、`steeringwheel`） | 碰撞防护、火炮和磁铁（`boat_bumpers`、`boat_cannon`、`boat_magnet`） | 赛船包与拖网（`boatrace_common`、`ocean_trawler`） |
| B31 | 嘉年华设施、布置及观景 | 嘉年华经营设施（`carnival_cannons`、`carnival_plaza`、`carnival_prizebooth`） | 装饰包与季节变体（`carnival_decor`、`carnivaldecor_eggride`、`carnivaldecor_figure`、`carnivaldecor_lamp`） | 家具与营地点缀（`furniture_chairs`、`lantern_posts`） |
| B32 | 嘉年华小游戏部署包 | 喂食、赶鸡、记忆（`carnivalgame_feedchicks`、`carnivalgame_herding`、`carnivalgame_memory`） | 高尔夫三类包（`carnivalgame_golf_hole`、`carnivalgame_golf_tee`、`carnivalgame_golfgame`） | 冰球、射击和转盘（`carnivalgame_puckdrop`、`carnivalgame_shooting`、`carnivalgame_wheelspin`） |
| B33 | 年节、猫窝与宠物活动部署包 | 猫窝与鼠年设施（`kitcoondecor`、`kitcoonden`、`rat_gym`） | 牛年与兔年活动设施（`yotb_post`、`yotb_sewingmachine`、`yotb_stage`、`yotr_decor`） | 胡萝卜鼠赛道（`yotc_carrat_race_checkpoint`、`yotc_carrat_race_finish`、`yotc_carrat_race_start`、`yotc_carrat_scale`） |
| B34 | 功能结构部署包 | 铸造与远古工程（`lunar_forge`、`shadow_forge`、`vault_pillar_guard`） | 生物、收纳与角色设备（`beefalo_groomer`、`deerclopseyeball_sentryward`、`gelblob_storage`、`wx78_drone_delivery`） | — |

先查玩家槽位或活动栏的实际入口。任务发放、捕获、钓获、设施回收与内部装填分别核验。活动对象注明模式与启用条件。部署组按包本体核对堆叠参数、后置初始化、部署产物和销毁，建筑、预览或 FX 通过关联链接说明。

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

## 全部轮次后的总收尾

- 以台账的实际验收页和注册名为输入，对账漏项、重复归属、待核验和确有理由的排除。状态归档到项目报告目录，保留各轮依据、处置与验收链接。
- 按已落地页的规模和查询需求，将暂行条目整理为正式 Prefab 叶索引。优先保留「有新鲜度／无新鲜度可堆叠／无新鲜度不可堆叠」入口，类别多时再按用途拆叶。混合族按主用途只登记一次，其他入口走互链。同步父入口、页尾返回链及原链接，保留根索引现有 Top 入口。
- 集中复检新增与实际受影响页，验收全库导航，复跑严格源码链接和哈希基线，确认名称、摘要、别名和索引一致。用玩家实际搜索词抽验入口，必要时定向刷新查询索引。由本对话做最后独立复检和整体收口。
