# 非装备物品栏 Prefab 扩库台账

日期 2026-11-08。源码为 [Steam build 26335394](../../../raw/current/BUILD.txt)。状态为已盘点、首批待认领。已通过验收的 [装备台账](prefab-equipment-ledger-20261105.md#整体验收2026-11-08) 与 [装备索引](../../../wiki/reference/prefabs/equipment-index.md) 用于排重。

## 统计口径

- 纳入正常玩法可进入玩家物品栏槽位或活动栏位的物品形态。可由捕获、钓获、制作、奖励等路径直接入栏，不要求可从地面手捡。用 `components/inventory.lua` 的 `GiveItem`、`actions.lua` 的 `PICKUP` 及实际获得流程区分玩家物品、NPC 临时武器、纯世界实体和内部载体。
- 按注册名与工厂形态归类。同一文件可同时出装备、物品和部署结果；装备本体和重物搬运形态留上一轮。已有 Wiki 正文覆盖的物品复用，别名或仅作依赖提及的命中先查主要归属。独占角色栏、事件专用对象、仅内部持有或条件未核实的形态先挂待核验。
- 有新鲜度指该形态运行 `perishable`，写清初始计时和停止条件。没有新鲜度的对象再按 `stackable` 的实际接线分流。活体、材料、药剂也可能有 `perishable`，有吃法也可能无新鲜度。普通计时、耐久、保质结束后替换都按各自组件核对。
- 源码扫描递归覆盖 `raw/current/scripts/prefabs` 的 **1,621 个 Lua 文件**。去注释后，`AddComponent("inventoryitem")` 有 **557 个调用位置，分布在 522 文件**。另查 `prefabutil.lua#L76` 的 `MakeDeployableKitItem`，在 43 文件找到调用，其中 1 个与直接命中重叠，净增 **42 文件**。合计 **564 个源码候选文件**，含公共工厂和混合文件。
- 下表按**主派发入口**每个源码文件计一次。**400 个进入分组候选池**，11 个先与现有页面对账，12 个先处理门禁或混合工厂，另有 141 个装备／宿主排除候选。文件数量用于估算工作量。全量唯一注册数、最终页数仍待工厂逐组核验；下文的 480 个已展开注册也包含大批调味变体。

| 队列 | 源码文件数 | 处理 |
|---|---:|---|
| P 有新鲜度或含对应变体 | 56 | 食材、作物、料理、活物和会变化的材料 |
| S 无新鲜度、可堆叠候选 | 164 | 按用途细分，确认每个变体的堆叠接线 |
| N 无新鲜度、不可堆叠候选 | 137 | 耐用品、角色道具、任务物品、收纳或部署态 |
| K 部署包补扫 | 43 | 6 个文件的调用传入堆叠上限，37 个未传或传 nil。逐个确认包本体和后置初始化，按实际接线回归 S 或 N |
| R 已有 Wiki 命中 | 11 | 查已有正文，复用、补写或协调归属 |
| M 入口待核验 | 12 | 先处理混合注册、条件入栏、内部装配和公共 helper |
| E 装备／宿主排除候选 | 141 | 默认不派。挂载位置与注册名单不一致时，只回捞可证实的非装备形态 |

## 首批可派发

以下 **9 页、30 个注册名**，与下方大组重叠的条目以本表为认领单位。盘点时目标路径和注册名均未命中现有非归档 Prefab 页；派发前再次核对当前文件、aliases、台账认领。表中目标均相对 `wiki/reference/prefabs/`，源码文件均相对 `raw/current/scripts/prefabs/`。

| 任务 | 主题 | 目标页 | 主注册名 | 源码文件（省略 .lua） | 分组 |
|---|---|---|---|---|---|
| I01 | 常用采集材料 | `basic-gathered-materials.md` | `cutgrass`、`twigs`、`cutreeds`、`log` | `cutgrass`、`twigs`、`cutreeds`、`log` | S01 |
| I02 | 基础精炼材料 | `basic-refined-materials.md` | `boards`、`cutstone`、`rope`、`papyrus` | `boards`、`cutstone`、`rope`、`papyrus` | S01 |
| I03 | 矿物原料与幸运金块 | `basic-mineral-materials.md` | `flint`、`nitre`、`rocks`、`marble`、`goldnugget`、`lucky_goldnugget` | `flint`、`nitre`、`inv_rocks`、`inv_marble`、`goldnugget` | S01 |
| I04 | 宝石族 | `gems.md` | `purplegem`、`bluegem`、`redgem`、`orangegem`、`yellowgem`、`greengem`、`opalpreciousgem` | `gem` | S03 |
| I05 | 船体修补物 | `boat-patches.md` | `boatpatch`、`boatpatch_kelp` | `boatpatch` | P05 |
| I06 | 月亮碎片与注能月亮碎片 | `moonglass-materials.md` | `moonglass`、`moonglass_charged` | `moonglass` | P05 |
| I07 | 催长剂起子与超级催长剂 | `soil-amender.md` | `soil_amender`、`soil_amender_fermented` | `soil_amender` | P05 |
| I08 | 排箫 | `panflute.md` | `panflute` | `panflute` | N01 |
| I09 | 羽毛扇与幸运扇 | `cooling-fans.md` | `featherfan`、`perdfan` | `fans` | N01 |

- I05 两态均可堆叠。木补丁无新鲜度，海带补丁在公共工厂之后追加 `perishable`，同页列差异。
- I06 两态均可堆叠。只有 `moonglass_charged` 加 `perishable`，耗尽替换为 `moonglass`，不另建重复页。
- I07 两态均不可堆叠。`soil_amender` 的 `perishable` 用于发酵，`soil_amender_fermented` 有独立构造且无 `perishable`。和常规食材腐坏分开解释。
- 首批可并行按 **I01–I03、I04–I06、I07–I09** 派发。用 dst-wiki-skill 的 `assets/prompts/new-pages.md`，先读现有相关页与实际构造，再核对关键消费者和简中显示名。写作代理只处理认领页，主代理集中复审、登记叶索引、刷新必要基线及收口。未确认的来源、获取方式和堆叠例外留缺口，禁止按台账直接写成已验证结论。

## 已展开的大工厂

两组工厂加首批，共 **480 个不重复的已展开注册名**。按实际形态分为有新鲜度 448、无新鲜度且可堆叠 28、无新鲜度且不可堆叠 4。其余候选仍以文件为工作量口径。

### P01 锅料理及调味

`prefabs/preparedfoods.lua#L128-L182` 的公共工厂从 `preparedfoods.lua`、`preparedfoods_warly.lua`、`spicedfoods.lua` 注册。普通表 68 道、沃利表 11 道，四种调味从两表浅拷贝，展开为 **79 基础 + 316 调味 = 395 个注册**，统一加 `stackable`。其中 **390 个有新鲜度**，`jellybean` 及其四种调味无新鲜度，`powcake` 虽然期限长，仍传入正的 `perishtime`。

- 普通表 68 个键：`butterflymuffin`、`frogglebunwich`、`taffy`、`pumpkincookie`、`stuffedeggplant`、`fishsticks`、`honeynuggets`、`honeyham`、`dragonpie`、`kabobs`、`mandrakesoup`、`baconeggs`、`meatballs`、`bonestew`、`perogies`、`turkeydinner`、`ratatouille`、`jammypreserves`、`fruitmedley`、`fishtacos`、`waffles`、`monsterlasagna`、`powcake`、`unagi`、`wetgoop`、`flowersalad`、`icecream`、`watermelonicle`、`trailmix`、`hotchili`、`guacamole`、`jellybean`、`potatotornado`、`mashedpotatoes`、`asparagussoup`、`vegstinger`、`bananapop`、`frozenbananadaiquiri`、`bananajuice`、`ceviche`、`salsa`、`pepperpopper`、`californiaroll`、`seafoodgumbo`、`surfnturf`、`lobsterbisque`、`lobsterdinner`、`barnaclepita`、`barnaclesushi`、`barnaclinguine`、`barnaclestuffedfishhead`、`leafloaf`、`leafymeatburger`、`leafymeatsouffle`、`meatysalad`、`shroomcake`、`sweettea`、`koalefig_trunk`、`figatoni`、`figkabab`、`frognewton`、`bunnystew`、`justeggs`、`veggieomlet`、`talleggs`、`beefalofeed`、`beefalotreat`、`shroombait`。
- 沃利表 11 个键：`nightmarepie`、`voltgoatjelly`、`glowberrymousse`、`frogfishbowl`、`dragonchilisalad`、`gazpacho`、`potatosouffle`、`monstertartare`、`freshfruitcrepes`、`bonesoup`、`moqueca`。
- 展开后缀：`_spice_chili`、`_spice_garlic`、`_spice_salt`、`_spice_sugar`。调味统一机制可集中在一页，实际负责的注册名在分派时归给唯一主体；按料理用途切批，先写公共工厂与少量典型菜，避免一次铺开几百个变体。

### P02 普通作物和种子

`prefabs/veggies.lua#L1084-L1110` 由 `VEGGIES` 与 `SEEDLESS` 展开。**20 原料 + 20 熟制 + 14 种子 + 1 干海带 = 55 个非装备注册**，各自工厂均挂 `inventoryitem`、`stackable` 与 `perishable`。14 个可出种的作物还各产出 3 个巨型形态，**42 个**已归 [巨型作物页](../../../wiki/reference/prefabs/giant-crops.md)。

- 原料键：`cave_banana`、`carrot`、`corn`、`pumpkin`、`eggplant`、`durian`、`pomegranate`、`dragonfruit`、`berries`、`berries_juicy`、`fig`、`cactus_meat`、`watermelon`、`kelp`、`tomato`、`potato`、`asparagus`、`onion`、`garlic`、`pepper`。各键加 `_cooked` 取得熟制名。
- 种子是其余作物的 `_seeds`；不出种的键为 `berries`、`cave_banana`、`cactus_meat`、`berries_juicy`、`fig`、`kelp`。干制额外只有 `kelp_dried`。
- 后续可按「农田 14 种」「野采与海带」「种子」划分所有权。通用 `seeds`／`seeds_cooked` 来自另一个文件，包含在 P02 候选；避免把普通种子和特定作物种子算成同一注册。

## 候选队列

以下名单是源码入口，省略 `prefabs/` 和 `.lua`。各组开写前按实际返回的 `Prefab` 或包装工厂展开，先确定本页的注册集合。S、N 是默认候选分桶，跨形态的例外只在一个主题页记录。带活动门禁的对象另定优先级。

### 有新鲜度

| 编号 | 主题 | 文件数 | 源码入口 |
|---|---|---:|---|
| P01 | 锅料理（含调味） | 1 | `preparedfoods` |
| P02 | 普通作物、种子与野采植物 | 16 | `veggies`、`acorn`、`cactus`、`cutlichen`、`firenettles`、`foliage`、`forgetmelots`、`moon_mushroom`、`mushrooms`、`petals`、`petals_evil`、`seeds`、`succulent_plant`、`tillweed`、`rock_avocado_fruit`、`tree_rock_seed` |
| P03 | 肉、蛋、鱼与虫翅食材 | 15 | `butter`、`butterflywings`、`egg`、`eel`、`fish`、`froglegs`、`goatmilk`、`hareball`、`honey`、`meats`、`milkywhites`、`moonbutterflywings`、`royal_jelly`、`tallbirdegg`、`trunk` |
| P04 | 活体水产和条件腐坏活物 | 5 | `canary_poisoned`、`moonbutterfly`、`oceanfish`、`pondfish`、`wobster` |
| P05 | 会腐坏／变化的非食材或多用途物品 | 14 | `ancienttree_fruits`、`boatpatch`、`fruitfly`、`glommerflower`、`inv_rocks_ice`、`lightbulb`、`moonglass`、`moonstorm_spark`、`moontree_blossom`、`mosquitomusk`、`mushtree_spores`、`pumpkin_lantern`、`soil_amender`、`wortox_reviver` |
| P06 | 角色与节庆食物 | 5 | `carnival_food`、`crumbs`、`wx78_foodbrick`、`yotp_food`、`yotr_food` |

### 无新鲜度、可堆叠

| 编号 | 主题 | 文件数 | 源码入口 |
|---|---|---:|---|
| S01 | 基础采集、建材与加工品 | 23 | `ash`、`boards`、`boneshard`、`charcoal`、`cutgrass`、`cutreeds`、`cutstone`、`driftwood`、`flint`、`gears`、`goldnugget`、`inv_marble`、`inv_rocks`、`livinglog`、`log`、`moonrocknugget`、`nitre`、`papyrus`、`rope`、`saltrock`、`silk`、`transistor`、`twigs` |
| S02 | 动物掉落与巢材 | 25 | `beardhair`、`bearger_fur`、`beefalowool`、`coontail`、`cookiecuttershell`、`deerclops_eyeball`、`dragon_scales`、`feathers`、`furtuft`、`goose_feather`、`honeycomb`、`houndstooth`、`lightninggoathorn`、`malbatross_feather`、`malbatross_feathered_weave`、`manrabbit_tail`、`minotaurhorn`、`pigskin`、`shroom_skin`、`slurperpelt`、`slurtle_shellpieces`、`steelwool`、`stinger`、`tentaclespots`、`walrus_tusk` |
| S03 | 洞穴、裂隙、远古与机械材料 | 14 | `dreadstone`、`gem`、`horrorfuel`、`lunarplant_husk`、`nightmarefuel`、`purebrilliance`、`refined_dust`、`scandata`、`thulecite`、`thulecite_pieces`、`vault_orb_refined`、`voidcloth`、`wagdrone_parts`、`wagpunk_bits` |
| S04 | 农业、栽种、肥料与改造 | 16 | `compost`、`guano`、`leif_idol`、`livingtree_root`、`lunar_seed`、`lureplant_bulb`、`marblebean`、`mosquitofertalizer`、`palmcone_scale`、`pinecone`、`plantables`、`poop`、`spidereggsack`、`treegrowthsolution`、`turfs`、`waterplant_planter` |
| S05 | 食物例外、产物与调料 | 11 | `antchovies`、`driedplants`、`dustmeringue`、`halloweencandy`、`mandrake_inactive`、`phlegm`、`spices`、`spoiledfood`、`wintersfeastfood`、`wintersfeastfuel`、`woby_treat` |
| S06 | 治疗、修补与药剂 | 19 | `bandage`、`bandage_butterflywings`、`bathbomb`、`compostwrap`、`coolant`、`forge_repair_kits`、`healingsalve`、`halloweenpotion_buffs`、`halloweenpotion_embers`、`halloweenpotion_moon`、`ipecacsyrup`、`lifeinjector`、`mosquitomermsalve`、`mosquitosack`、`sewing_tape`、`spider_healer_item`、`spider_mutators`、`spidergland`、`tillweedsalve` |
| S07 | 投放消耗、弹药、船用补给 | 14 | `blackflag`、`cannonballs`、`dock_kit`、`dock_woodposts`、`firecrackers`、`gunpowder`、`oceanfishingbobber`、`oceanfishinglure`、`rope_bridge`、`slingshotammo`、`slurtleslime`、`trap_fumarole`、`trap_starfish`、`townportaltalisman` |
| S08 | 可投放结构、打包与生活用品 | 12 | `bedroll`、`beeswax`、`bundle`、`featherpencil`、`fence`、`fence_electric`、`fossil_piece`、`minisign`、`wagboss_robot_constructionsite`、`wagpunk_floor_kit`、`walls`、`waxpaper` |
| S09 | 货币、交易、装饰与收集 | 17 | `antliontrinket`、`carnival_gametoken`、`carnival_prizeticket`、`cotl_trinket`、`halloween_ornaments`、`hermithouse_ornament`、`horseshoe`、`pig_coin`、`pig_token`、`scrapbook_page`、`stash_map`、`trinkets`、`wagstaff_tools`、`winter_ornaments`、`yotb_beefalo_dolls`、`yotb_pattern_fragments`、`yotr_token` |
| S10 | 收服活物、角色与状态物 | 13 | `bee`、`butterfly`、`cursed_monkey_token`、`fireflies`、`ghostflower`、`ghostly_elixirs`、`glommerfuel`、`lightflier`、`mosquito`、`wereitems`、`willow_ember`、`winona_holoitems`、`wortox_soul` |

### 无新鲜度、不可堆叠

| 编号 | 主题 | 文件数 | 源码入口 |
|---|---|---:|---|
| N01 | 耐用品、陷阱与消耗型工具 | 18 | `beemine`、`birdtrap`、`carpentry_blades`、`fans`、`farm_plow`、`fertilizer`、`horn`、`houndwhistle`、`panflute`、`pumpkincarver`、`razor`、`sewingkit`、`spider_repellent`、`spider_whistle`、`spiderden_bedazzler`、`trap`、`trap_bramble`、`trap_teeth` |
| N02 | 容器、资料、图纸与标记 | 22 | `battlesong_container`、`beargerfur_sack`、`blueprint`、`constructionplans`、`cookbook`、`cookingrecipecard`、`decor_flowervase`、`decor_lamp`、`decor_pictureframe`、`elixir_container`、`furniture_decor_items`、`mapscroll`、`phonograph`、`records`、`scrapbook_notes`、`sketch`、`slingshotammo_container`、`slingshotparts`、`tacklecontainer`、`tacklesketch`、`w_radio`、`yotb_blueprints` |
| N03 | 角色道具、读物与部署设备 | 28 | `abigail_flower`、`balloons_empty`、`battlesongs`、`books`、`deck_of_cards`、`miniboatlantern`、`pocket_scale`、`portableblender`、`portablecookpot`、`portablefirepit`、`portablespicer`、`portabletent`、`storage_robot`、`wagstaff_items`、`winona_battery_high`、`winona_battery_low`、`winona_catapult`、`winona_remote`、`winona_spotlight`、`winona_storage_robot`、`winona_teleport_pad`、`wolfgang_whistle`、`wortox_souljar`、`wx78_gestalttrapper`、`wx78_moduleremover`、`wx78_modules`、`wx78_scanner`、`cattoy_mouse` |
| N04 | 伙伴、被捕获活物与养殖 | 18 | `beef_bell`、`birds`、`birds_mutant`、`birds_mutant_rift`、`carrat`、`chester_eyebone`、`glommerwings`、`hutch_fishbowl`、`kitcoon_nametag`、`lavae_cocoon`、`lavae_egg`、`lavae_tooth`、`lightcrab`、`mole`、`rabbit`、`rabbitking`、`rabbitkinghorn`、`saddle` |
| N05 | 任务、解锁、遗迹与特殊摆件 | 31 | `alterguardianhatshard`、`archive_lockbox`、`archive_resonator`、`atrium_key`、`atrium_ritual_organs`、`chestupgrade_stacksize`、`deer_antler`、`gravestone`、`graveurn`、`hermit_pearl`、`hermitcrab_relocation_kit`、`hermitcrab_shell`、`hermithouse_laundry`、`moonrockcrater`、`moonrockidol`、`moonrockseed`、`nightmare_timepiece`、`reviver`、`ruinsrelic`、`security_pulse_cage`、`shadowheart`、`shadowheart_infused`、`singingshell`、`teleportato_parts`、`terrarium`、`vault_key`、`vault_pillar_guard_pieces`、`vault_teleporter`、`vaultrelic`、`mooneye`、`playbill` |
| N06 | 船体、船上设备与信号 | 11 | `boat`、`boat_magnet_beacon`、`floatinglantern`、`flotationcushion`、`mast`、`mastupgrade_lamp`、`mastupgrade_lightningrod`、`megaflare`、`miniflare`、`oceanfishableflotsam`、`yotd_boats` |
| N07 | 特殊食物、种植与药剂 | 8 | `ancienttree_seed`、`ancienttrees`、`bullkelp_beached`、`desiccant`、`hermitcrabtea`、`nonslipgrit`、`pigtorch`、`wintersfeastcookedfoods` |
| N08 | 活动搭建设备 | 1 | `yotb_stage` |

### 部署包

| 编号 | 主题 | 文件数 | 源码入口 |
|---|---|---:|---|
| K01 | 航海部署包 | 8 | `anchor`、`boat_bumpers`、`boat_cannon`、`boat_magnet`、`boat_rotator`、`boatrace_common`、`ocean_trawler`、`steeringwheel` |
| K02 | 活动、装饰与小游戏部署包 | 28 | `carnival_cannons`、`carnival_decor`、`carnival_plaza`、`carnival_prizebooth`、`carnivaldecor_eggride`、`carnivaldecor_figure`、`carnivaldecor_lamp`、`carnivalgame_feedchicks`、`carnivalgame_golf_hole`、`carnivalgame_golf_tee`、`carnivalgame_golfgame`、`carnivalgame_herding`、`carnivalgame_memory`、`carnivalgame_puckdrop`、`carnivalgame_shooting`、`carnivalgame_wheelspin`、`furniture_chairs`、`kitcoondecor`、`kitcoonden`、`lantern_posts`、`rat_gym`、`yotb_post`、`yotb_sewingmachine`、`yotc_carrat_race_checkpoint`、`yotc_carrat_race_finish`、`yotc_carrat_race_start`、`yotc_carrat_scale`、`yotr_decor` |
| K03 | 功能结构部署包 | 7 | `beefalo_groomer`、`deerclopseyeball_sentryward`、`gelblob_storage`、`lunar_forge`、`shadow_forge`、`vault_pillar_guard`、`wx78_drone_delivery` |

传入堆叠参数的 6 个入口：`boat_bumpers`、`carnival_cannons`、`carnival_decor`、`carnivaldecor_eggride`、`carnivaldecor_lamp`、`wx78_drone_delivery`。明确只派物品包注册，不把包生成的建筑、船上实体、placer 或 FX 加入物品队列。`boatrace_common` 是公共工厂，沿 `boatrace_start`、`boatrace_checkpoint`、`boatrace_seastack` 展开；这三个消费者不重复加到 564 文件数中。`carnivalgame_feedchicks` 的投放包和可装备饲料分开查。

### 复用、待核验与排除

| 编号 | 主题 | 文件数 | 源码入口 |
|---|---|---:|---|
| R | 已有主体，复用或局部补写 | 11 | `bernie_active`、`heatrock`、`pocketwatch`、`pocketwatch_common`、`pocketwatch_dismantler`、`pocketwatch_parts`、`slingshotmodkit`、`wormlight`、`yotr_fightring`、`messagebottle`、`moonstorm_static` |
| M | 混合、装配或特殊入口，先核对再派 | 12 | `balloons_common`、`charlierose`、`cozy_bunnyman`、`giftsurprise`、`temp_beta_msg`、`wagdrone_common`、`waxed_plant_common`、`waxwelljournal`、`wx78_inventorycontainer`、`eyeturret`、`gestalt_cage`、`spider` |

- **复用正文**：暖石查 [heatrock](../../../wiki/reference/prefabs/heatrock.md)；怀表及其拆解器和零件查 [pocketwatch](../../../wiki/reference/prefabs/pocketwatch.md)；伯尼查 [character-gear-extra](../../../wiki/reference/prefabs/character-gear-extra.md)；弹弓改装套件查 [ranged-weapons](../../../wiki/reference/prefabs/ranged-weapons.md)；库存形态 `moonstorm_static_item` 已有 [staffs-more](../../../wiki/reference/prefabs/staffs-more.md)；瓶中信、空瓶和可装备投掷变体已有 [utility-and-throwables](../../../wiki/reference/prefabs/utility-and-throwables.md)。
- **协调归属**：[wormlight](../../../wiki/reference/prefabs/wormlight.md) 的 FX 页收了 `wormlight` 和 `wormlight_lesser` 两个物品别名，但正文明确未展开物品本体。补库存页时先协调移交这两个注册名的主要归属。[yotr_fightring_torch](../../../wiki/reference/prefabs/yotr_fightring_torch.md) 已列套装部署及配方，首批复用该节。其他仅命中 FX、生成物或关联导航的别名一律查正文后再决定。
- **混合入口**：`eyeturret.lua` 单独核验 `eyeturret_item`，同文件的 `equippable` 属于炮台武器；`gestalt_cage.lua` 把空笼和 `gestalt_cage_filled1/2/3` 分开，后者有独立库存接线；`spider.lua` 区分可捕获蜘蛛与临时武器。温度、捕获、任务发放或变形会改拾取门禁时，记录可入栏的阶段。
- **间接挂载**：`balloons_common` 展开到 `balloon`、`balloonspeed`、`balloonparty`、`balloonvest`、`balloons_held_child`，排除已登记装备和挂件；`waxed_plant_common` 追 `CreateDugWaxedPlant` 调用，区分可入栏的挖出形态与摆在地上的植物；`wagdrone_common` 追可友好化工厂的动态挂载；`SGcozy_bunnyman.lua#L884-L889` 的抱枕动态挂载跟原装备篇对账。活动服务器单独加载、脚本侧无法证实的条目保留缺口。
- **禁止按文件排混合族**：`veggies`、`eyeturret`、`gestalt_cage`、`spider`、`carnivalgame_feedchicks`、`moonstorm_static`、`messagebottle` 的直接命中跨多种形态，已分别放入 P、K、M 或 R。其余含 `equippable` 的 E 文件默认留在装备／宿主范围，遇到独立物品注册再以实际工厂回捞。

E 组 141 个文件：

- `amulet`、`armor_bramble`、`armor_carrotlure`、`armor_dragonfly`、`armor_dreadstone`、`armor_grass`、`armor_lunarplant`、`armor_marble`、`armor_rocky`、`armor_ruins`、`armor_sanity`、`armor_skeleton`、`armor_slurper`、`armor_snurtleshell`、`armor_voidcloth`、`armor_wagpunk`
- `armor_wood`、`armor_yoth_knight`、`axe`、`axe_pickaxe`、`backpack`、`bat_boss`、`batbat`、`beard_sack`、`beargervest`、`beeswax_spray`、`bernie_inactive`、`blowdart`、`bomb_lunarplant`、`boomerang`、`bootleg`、`brush`
- `bugnet`、`bullkelp_root`、`candybag`、`cane`、`carnival_golfclub`、`carnival_vest`、`cavein_boulder`、`chesspieces`、`chum`、`compass`、`costumes`、`cutless`、`diviningrod`、`dumbbells`、`farm_hoe`、`fence_rotator`
- `firepen`、`fishingnet`、`fishingrod`、`glass_spike`、`glasscutter`、`gnarwail_horn`、`hambat`、`hammer`、`hats`、`hawaiianshirt`、`houndstooth_blowpipe`、`icepack`、`king_cane`、`krampus_sack`、`lighter`、`lucy`
- `malbatross`、`merm_tool`、`minifan`、`mininglantern`、`monkey`、`moon_altar_pieces`、`mosquitobomb`、`nightstick`、`nightsword`、`oar`、`oceanfishingrod`、`oceantreenut`、`onemanband`、`pickaxe`、`pickaxe_lunarplant`、`piggyback`
- `pillow_equipment`、`pitchfork`、`pocketwatch_weapon`、`potatosack`、`propsign`、`pumpkin_bomb`、`queen_torch`、`rabbitkingspear`、`raincoat`、`redlantern`、`reflectivevest`、`reskin_tool`、`ruins_bat`、`saddlehorn`、`sculpture_pieces`、`seedpouch`
- `shadow_battleaxe`、`shell_cluster`、`shieldofterror`、`shovel`、`shovel_lunarplant`、`sleepbomb`、`slingshot`、`slurper`、`snowball_item`、`snowman`、`spat`、`spear`、`spear_wathgrithr`、`spicepack`、`staff`、`staff_lunarplant`
- `staff_tornado`、`sweatervest`、`sword_lunarplant`、`tentaclespike`、`thurible`、`torch`、`treasurechest`、`trident`、`trunkvest`、`umbrella`、`vault_compass`、`voidcloth_boomerang`、`voidcloth_scythe`、`voidcloth_umbrella`、`walking_stick`、`walrus`
- `wateringcan`、`waterplant`、`waterplant_seed`、`waterprojectiles`、`wathgrithr_shield`、`whip`、`winona_telebrella`、`witchbroom`、`wortox_nabbag`、`wurt_terraform_item`、`wx78_drone_zap`、`yoth_knightstick`、`yoth_lance`

## 后续对账

1. 按 [B01–B43 分批执行表](prefab-inventory-batches-20261108.md) 认领，每个注册只交给一个写作任务。阶段状态记在下表，具体结论写本轮交付和维护日志，PLAN 只记阶段起止。
2. 候选分诊结果用「写作／复用／排除／待核验」记录。首批从 B01 开始，等具体接线、获得路径与已有覆盖都核实后再计入对应组的唯一注册数。
3. 需要复跑初筛时，在知识仓用 `rg -l 'AddComponent\s*\(\s*"inventoryitem"' raw/current/scripts/prefabs` 和 `rg -l '\bMakeDeployableKitItem\s*\(' raw/current/scripts/prefabs` 找入口，去注释与重复路径。用 `equippable`、`perishable`、`stackable` 的挂载和移除点辅助分诊，最终以实际工厂和消费者为准。静态扫描覆盖不了所有动态调用，已知公共 helper 的消费关系见上文。未运行游戏。

| 轮次 | 状态 | 写入页／复用／排除／待核验 | 交付与外部复检 |
|---|---|---|---|
| B01 | 已收口 | 9 页 30 注册全落地（写作，无复用／排除）：[basic-gathered-materials](../../../wiki/reference/prefabs/basic-gathered-materials.md)（cutgrass、twigs、cutreeds、log）、[basic-refined-materials](../../../wiki/reference/prefabs/basic-refined-materials.md)（boards、cutstone、rope、papyrus）、[basic-mineral-materials](../../../wiki/reference/prefabs/basic-mineral-materials.md)（flint、nitre、rocks、marble、goldnugget、lucky_goldnugget）、[gems](../../../wiki/reference/prefabs/gems.md)（六色＋彩虹宝石）、[boat-patches](../../../wiki/reference/prefabs/boat-patches.md)（boatpatch、boatpatch_kelp）、[moonglass-materials](../../../wiki/reference/prefabs/moonglass-materials.md)（moonglass、moonglass_charged）、[soil-amender](../../../wiki/reference/prefabs/soil-amender.md)（soil_amender、soil_amender_fermented）、[panflute](../../../wiki/reference/prefabs/panflute.md)、[cooling-fans](../../../wiki/reference/prefabs/cooling-fans.md)（featherfan、perdfan）。澄清：inv_rocks.lua／inv_marble.lua 只是文件名，玩家物品栏注册名就是 rocks／marble；perdfan 有独立 NAMES 键（幸运扇）。I05／I06／I07 台账线索全部经源码证实。独立检视 P1×4／P2×5／P3×12 全部处置（boat-patches 无问题）；暂行登记 [prefabs/index.md](../../../wiki/reference/prefabs/index.md)「基础材料与常用物品（暂行）」节。外部复检（[prefab-inventory-review-b01-20261108.md](prefab-inventory-review-b01-20261108.md)）P1×1／P2×12／P3×1 经主代理逐项回源码验证全部成立并采纳：R01 催长剂示例读档顺序重写（SpawnSaveRecord 先 postinit 后 Perishable:OnLoad 覆盖剩余秒数）、R02 施肥双路径与还瓶分支、R03 替换改「尝试回填」且腐坏条归仅起子、R04 月碎片替换继承原堆叠、R05 月相仪退晶分两路径、R06 扇降温两段独立结算经隔热折算、R07 吟唱轨迹改带前提、R08 排箫入睡细化与灵魂环带 3.95–7.9、R09 postinit 生效面收窄、R10 蟹王珍珠款门禁与颜色分支收窄、R11 修船回血补 Repair 成功门禁且缓存限定参数、R12 采集 PICK 与 PICKUP 分列（索引行同步）、R13 绳子 USEITEMON／燃烧 30s＜燃料值 45s／猪王切换限定 YOTP、R14 perdshrine 改「火鸡神龛」；台账 gems 括号同步改「六色＋彩虹宝石」 | 首轮验收 validate --check-indexes 10/10、strict 457 引用 0 错 0 警、record 9 页、Lua 7 段、coverage exit 0；修后复验 validate --check-indexes 10 文件全过、strict 478 引用 0 错 0 警（新锚点起点警告校准归零）、record 5 页 sources 变化刷新、Lua 7 段复跑通过、git diff --check 无实际问题。未运行游戏。修订后复检（2026-11-08，见复检报告「修订后复检」节）：R01 及其余大部分修正确认通过，余 R05/R06/R10/R12/R13 五项局部表述——即时分支取搜索结果第一个候选该堆未满才 +1（不继续找后续堆）、扇风目标限标签筛选且能读取温度、蟹王已嵌珍珠直接珍珠款（opal≥3 仅用于未嵌珍珠分支）、采集 GiveItem 尝试入栏栏满仍可能掉地（索引同步）、导航表普通金块改「非 YOTP 活动期」——经主代理回源码验证成立并直改，受影响五页＋索引 validate/strict 复跑 0 错 0 警。外部最终复检确认五项均已对齐，B01 已收口。 |
| B02 | 已收口 | 3 页 28 注册全落地（写作，无复用／排除）：[farm-crops-staple](../../../wiki/reference/prefabs/farm-crops-staple.md)（carrot、corn、pumpkin、potato 及各自 `_cooked`）、[farm-crops-fruit](../../../wiki/reference/prefabs/farm-crops-fruit.md)（eggplant、durian、pomegranate、dragonfruit、watermelon、tomato 及 `_cooked`）、[farm-crops-spice](../../../wiki/reference/prefabs/farm-crops-spice.md)（asparagus、onion、garlic、pepper 及 `_cooked`）。共用工厂与生熟两态通则只在 staple 页的「共用工厂与生熟两态」展开，另两页只写差异并链接；`<name>_seeds` 与三件巨型变体归 B03／giant-crops，本批不展开。澄清：`PEPPER_COOKED` 简中确有「烤辣椒」（chinese_s.po L346779–L346782），派发时的「缺译」判断有误（误把 `strings.pot` 的空 `msgstr` 当证据）；堆叠例外是 pumpkin／eggplant／durian／watermelon 生件落 `stackable` 默认 20（STACK_SIZE_MEDITEM），其余生件与全部熟件 40 | 独立检视（未参与写作，只读核对后直接改页）P1×4／P2×8／P3×16，主代理逐条 diff 回源码复核后采纳：南瓜活动期分支改为「prefab 文件在 `LoadAssets` 重载时构造、早于 `PopulateWorld` 的 `ApplySpecialEvent`，构造取加载前进程值」；两页「典型接入」补 postinit 与读档 `Perishable:OnLoad` 的覆盖顺序（比例＝旧剩余÷新上限，不保原比例）；fruit 页降温示例由改 `temperaturedelta` 改为缩 `temperatureduration`（速率被夹到 ≤1℃/秒，−40 与 −20 等效）；`FEED` 候选收窄为只有 Woby（Polly 无 critter／handfed 标签）；番茄世界生成来源由 SwampPig 勘正为 FisherPig；spice 页锚点起点结构行改 L50-L53。索引「基础材料与常用物品（暂行）」新增 3 行并同步返回链，giant-crops 补三页回链（`updated` 改 2026-11-08）。验收：`validate_page --check-indexes` 5 文件通过（三页 192／203／165 行，32,757／32,760／25,681 字节）、strict 354 引用 0 错 0 警（245 唯一目标／48 源文件）、record 3 页已记录、Lua 示例 3 段 `parser_probe` 通过、`git diff --check` 无实际问题。未运行游戏。待核验：quagmire 节日数据不在快照、月亮精华液活动门禁、`fruitdragon` 战斗／睡眠 SG、雪人装饰与称重台运行时；外部复检（[prefab-inventory-review-b02-20261108.md](prefab-inventory-review-b02-20261108.md)）R01–R11：10 项成立、1 项部分成立、无驳回，已由主代理直改三页与索引（西瓜降温去固定承诺、变异交付写实、FisherPig 降为未核验、删南瓜跨世界推断、补兔人死亡掉落与沃利开局、补鸟笼种子、千层饼 `not inedible` 门禁、读档按旧剩余与新上限取小、生熟动画与拾音分开、腐坏钳零、三个官方名）；压缩后 A 186 行／32,694 字节、B 32,673 字节、C 26,129 字节；修订后复检六项与二次修订复检的 R01／R08／R10 三项均已在原轮直改，strict 363 引用 0 错 0 警、record 三页 already_recorded。日志已分卷至 log-2026-11-08-001，各次修订合记于该卷；最终复检逐项核对 R01–R11，三页、28 注册及暂行索引通过。收口时补齐腐烂理智替换值与辛香数值示例的适用条件。沿用修订方定向验收，本次未重跑校验器或游戏 |
| B03 | 已收口 | 5 页 30 个注册（29 件可入栏物品＋`seeds_placer` 预览）全落地（写作，无复用／排除）：[farm-crop-seeds](../../../wiki/reference/prefabs/farm-crop-seeds.md)（14 个 `<作物>_seeds`）、[generic-seeds](../../../wiki/reference/prefabs/generic-seeds.md)（`seeds`、`seeds_cooked`、`seeds_placer`）、[wild-berries](../../../wiki/reference/prefabs/wild-berries.md)（`berries`、`berries_juicy` 及熟态）、[tropical-gathered-fruits](../../../wiki/reference/prefabs/tropical-gathered-fruits.md)（`cave_banana`、`fig`、`cactus_meat` 及熟态）、[kelp](../../../wiki/reference/prefabs/kelp.md)（`kelp`、`kelp_cooked`、`kelp_dried`）。澄清：`SEEDLESS` 6 项并非都只导出本体与熟态（`kelp` 另出 `kelp_dried`）；`kelp` 简中为「海带叶」、`cave_banana` 为「香蕉」（「洞穴香蕉树」是植株）；`fig` 用 BERRY 拟饵档而非 FIG 档；`berries_juicy` 生件堆叠也是 40 而非组件默认 20。 | 独立检视 P1×1／P2×6／P3×6（3 条仅建议），经主代理逐条回源码复核全部成立，由检视方直改 4 页 10 处：`SEEDLESS` 导出表述、`KNOWN_` 段含 `KNOWN_TURNIP_SEEDS`、`long_life` 只作用保鲜窗口、旧农田注册名改 `slow_farmplot`／`fast_farmplot`、挖起掉落条件、四处 `sources` 缺项。主代理另补跨页修正：`farm-crops-fruit` 权重句限定「经旧农田 `pickproduct`」、`farm-crops-staple`／`farm-crops-spice` 回链种子页、`giant-crops` 的 `SEEDLESS` 名单改「香蕉」「海带叶」、`bullkelp_root` 冲岸实体名改 `bullkelp_beachedroot`。验收：`validate_page --check-indexes` 11 文件通过、导航 0 问题；strict 0 错 0 警；record 10 页；Lua 示例 5 段 `parser_probe` 通过；`git diff --check` 无实际问题；暂行索引新增 5 行。未运行游戏；待核验 quagmire 数据、物品图标来源、`monkeyisland_portal` 出现条件、`driedsalticon` 客户端显示链。外部复检（[prefab-inventory-review-b03-20261108.md](prefab-inventory-review-b03-20261108.md)）P1×2／P2×11／P3×2 经主代理逐条回源码复核全部成立并已改：果蝇王掉落两分支写反、`farmplantable.plant` 改函数只适用 `PLANTSOIL`、鸟笼限普通鸟与变异鸟分支、随机株秋春 180–240 秒、`ReplaceWithPlant` 的 `plant_def` 门禁、烤种子可装饰雪人、两页保质示例区分新生成与读档绝对秒数、挖起产物名与交付分支、香蕉丛移植额度与春季倍率、仙人掌刺伤门禁与 `badfood` 作用面、冲岸掉叶需 `Pick` 门禁、晾晒暂停／酸雨／烧毁／`LongUpdate` 分支、KELP 修补限定草筏与保险杠、注册口径改 30；P3 改「诱饵」措辞、Gorge 钩子只在该页 `fn`、补浣猫／公牛海带／水獭掠夺者窝点简中名。修订后 `validate_page --check-indexes` 11 文件通过、strict 1013 引用 0 错 0 警（116 源文件）、record 已刷新、Lua 示例 5 段通过。用户复核再补 6 处：播种链改为「先 `target:Remove()` 再 `SpawnPrefab`」、Wormwood 部署改「无需预先锄地」并说明 `CollapseSoilAtPoint` 会处理土堆、浆果构造表改「拟饵符号不同」（两种生件浮水参数相同）、`onworked_juicy` 改「还没挖完」、仙人掌开花段补刺伤门禁且牛的状态机改「自己读三维符号、不看 `badfood` 标签」、海带两页统一「生成茎＋各路径采摘条件才额外掉叶」且晾晒续计限定 `dryingrack_drytime`；再验 11 页通过、strict 1013 引用 0 错 0 警。最终复检确认 R01／R02 与 5 组遗留项已处置，补正播种回交、提前定种、牛的状态、海带掉落和晾晒暂停条件，同步暂行索引。定向验收 7 文件通过，strict 628 引用 0 错 0 警，六页 185 项源码哈希比对均一致。沿用既有 Lua 解析结果，未运行游戏。见[最终复检与收口](prefab-inventory-review-b03-20261108.md#最终复检与收口) |
| B04 | 写作中 | 其他野采与种植材料三切片：A 蘑菇族 8 注册（`red_cap`／`green_cap`／`blue_cap` 及熟态、`moon_cap` 及熟态）＋苔藓叶类 3 注册（`cutlichen`、`foliage`、`quagmire_foliage_cooked`）；B 花瓣族 3 注册（`petals`、`petals_evil`、`cactus_flower`）＋野草 3 注册（`forgetmelots`、`tillweed`、`firenettles`）；C 树种子 5 注册（`acorn`、`acorn_cooked`、`acorn_placer`、`tree_rock_seed`、`tree_rock_seed_placer`）＋石果与多肉 5 注册（`rock_avocado_fruit_ripe`、`_ripe_cooked`、`_sprout`、`succulent_plant`、`succulent_picked`）。共 6 页 27 注册 | 三名写作任务并行中，待交付后独立检视 |

后续轮次在实际认领时追加。状态依次为「待认领／写作中／待外部复检／已收口」，证据不足的登记待核验事项和原因。
