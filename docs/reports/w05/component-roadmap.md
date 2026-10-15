# 剩余组件盘点与建设路线

日期：2026-10-12；源码标识沿用工程覆盖清单的 Steam buildid `25643519`。这是第五批结束后的路线快照，仅按组件文件名、已建页面及既有覆盖分类归组，未读取本轮待建 Lua 实现、未扫描调用、未确认硬依赖或游戏行为。推进状态以 [PLAN](../../../PLAN.md#当前优先组件-wiki-分批建设) 与 [覆盖清单](component-coverage.csv) 为准；本记录不进入 Wiki 检索、日常查询或 Skill 必读链。

## 盘点结果

| 范围 | 数量 | 本轮处理 |
|---|---:|---|
| 组件目录 Lua 文件 | 821 | 目录路径与覆盖 CSV 逐项一致，无新增／缺失 |
| 组件候选 | 798 | 48 已建，750 未建；候选是否可实例化仍待逐项核验 |
| 近期第 6～15 批 | 41 | 固定下一批，后面按实现／消费者／任务缺口调整 |
| 扩展主题 | 470 | 按开发能力储备，含专用实现，不承诺都是通用公开接口 |
| 专项主题 | 236 | 角色、活动、区域、Boss 等后续按真实需求拆小批 |
| 职责待确认 | 3 | 不靠名称编造定位，保留登记等待核验 |
| replica／辅助模块／空文件 | 19／3／1 | 单列处理，不算 750 个未建组件候选 |

750 个未建候选全部出现在下文且各归属一个主组，未重复计数。分组不是源码目录重构，也不是知识页的已核验标签。多个主题有交集时只登记主组，实际编写可提前抽出所需组件。

## 推进方式

1. 先完成第六批，之后按下表推进高复用能力；复杂组件可单独小批。新增／修改任务缺少必要协作者时允许插入，保留已完成页面与经验。
2. 农作、生物、容器、战斗等扩展主题继续按 3～6 篇拆批；同批解释一个接入问题，不一次读取某组的全部实现。世界、网络、玩家操作和船体复杂模块按宿主／消费者缩小范围。
3. 角色、活动、区域与 Boss 专项按实际任务或覆盖轮次推进。核验后可调整主组；文件存在不证明机制当前启用，也不因名字像旧模式就标为废弃。
4. 750 是待核验文件数，不是已决定生成 750 篇等长教程。适合模组接入的组件写精选接口／完整必要案例；内部或专用协作先写宿主、边界、原版入口与链接，必要时归实际主题／辅助接口。身份变化在覆盖清单记录，不用强造 demo 凑页数。
5. 正式写页时核验定义、必要消费者和宿主生命周期，按真实缺口复查模板；本轮不改模板、接口数据或查询规则。每批执行既有内容／结构／检索检视，游戏验证与实际 token 测量另行保留。
6. 已建组件按任务暴露的缺口增修。需要 replica、EntityScript、动作／SG、RPC、配方注册、Light／资源／UI API 时另补实际主题，不把仅补组件宣称为完整编码覆盖。配方／建筑 cookbook 与 W05 真实任务验证不用等全量完成。

## 近期第 6～15 批

除第六批沿用既有安排，其余是执行前待核验的拟定队列；确切接口、归属与案例不由文件名推断。

| 批次 | 主题 | 候选组件 | 拟支撑需求 |
|---|---|---|---|
| 6 | 潮湿与冰冻（3） | `freezable`、`moisture`、`moistureimmunity` | 延续已排定的第六批，补防雨／温度／效果协作 |
| 7 | 工作效率与消耗倍率（3） | `efficientuser`、`worker`、`workmultiplier` | 补现有 tool／workable／finiteuses 的效率与消耗选型 |
| 8 | 睡眠与困倦（4） | `grogginess`、`sleeper`、`sleepingbag`、`sleepingbaguser` | 睡眠状态与物品／使用者分工，核验唤醒及撤销 |
| 9 | 治疗与修复（5） | `healer`、`maxhealer`、`repairable`、`repairer`、`sewing` | 补恢复物品、上限恢复、修复材料与装备修补 |
| 10 | 投射物与爆炸（3） | `complexprojectile`、`explosive`、`projectile` | 远程武器／效果对象的基础机制 |
| 11 | 定向技能与瞄准（6） | `aoecharging`、`aoespell`、`aoetargeting`、`cooldown`、`reticule`、`spellbookcooldowns` | 补现有 spellcaster／spellbook／rechargeable 的施放与冷却选型 |
| 12 | 科技与建造接入（4） | `craftingstation`、`deployhelper`、`placer`、`prototyper` | 为配方／建筑骨架补必要机制；注册 API 另记缺口 |
| 13 | 烹饪与干燥（5） | `cookable`、`cooker`、`dryable`、`dryingrack`、`stewer` | 食物加工与配方判定分工；料理注册 API 另记缺口 |
| 14 | 打包与拆包（3） | `bundlemaker`、`bundler`、`unwrappable` | 物品集合打包、资源与存档恢复的组装 |
| 15 | 传送与距离触发（5） | `entitytracker`、`playerprox`、`positionalwarp`、`teleportedoverride`、`teleporter` | 传送对象、位置恢复、接近触发与关联实体记录分别核验 |

第 6～15 批共 41 个候选；若都确认为独立组件并完成页面，组件页从 48 增至 89，未建候选从 750 减至 709。该数字只是条件性进度目标，不表示全库类型、游戏行为或 W05 已完成。

## 扩展主题：470 个候选

以下为主题池，不是 19 个整批或新的 Wiki 层级。表中数量按当前唯一归属统计；名称只提供定位线索。默认先推进农作／生物与现有物品机制的扩展，再逐步展开角色宿主、世界、网络和海上系统；真实需求可改变顺序。

| 组 | 主题 | 候选数 | 拆批方向 |
|---|---|---:|---|
| G01 | [湿度／防护／温度扩展](#g01) | 22 | 补不同宿主的潮湿、遮蔽、温度与雨水关系 |
| G02 | [伤害／防御／控制扩展](#g02) | 30 | 特殊伤害、反伤、抗性与异常控制分别选小批 |
| G03 | [采集／工具／主动使用物品](#g03) | 35 | 工作动作、地形工具及目标式／装备式使用接口 |
| G04 | [容器／物品持有／燃料／堆叠扩展](#g04) | 20 | 补 container／inventory 的代理和持有者，资源消耗与自动堆叠 |
| G05 | [饮食与加工扩展](#g05) | 12 | 食物偏好、记忆、加工产出与额外保鲜／生产 |
| G06 | [农作／肥料／植物种植](#g06) | 19 | 按土壤／种植／成长／研究拆批，旧 crop 路径需核验现状 |
| G07 | [生物生成／巢穴／群体](#g07) | 26 | 生成器、巢穴与群体结构；Brain／SG 的缺口另列 |
| G08 | [驯养／骑乘／宠物状态](#g08) | 11 | 驯养、鞍具、骑手、拴系与宠物 UI 生命周期 |
| G09 | [战斗行为与陆地运动协作](#g09) | 23 | 目标选择、状态运动及外部物理改动；不按组件名推导完整 AI |
| G10 | [主动技能／读写／通道与机关](#g10) | 25 | 通道／施放／读写接口按实际交互选批 |
| G11 | [升级／套装／皮肤／外观能力](#g11) | 20 | 升级能力与视觉换装分开，记录保留原行为的扩展入口 |
| G12 | [建造扩展／拆装／记录和物品交互](#g12) | 22 | 大体量施工流程独立小批，其他交互按任务推进 |
| G13 | [玩家操作／视野／复活与迁移](#g13) | 22 | 复杂宿主和客户端边界按单组件小批，避免强行凑篇数 |
| G14 | [显示／音效／文本／HUD 与局部地图](#g14) | 53 | 呈现与机制分开；底层 Light／动画／UI API 是并行缺口 |
| G15 | [世界时间／天气／生态与区域](#g15) | 51 | 按世界状态／天气／重生／地图区域分别拆批 |
| G16 | [船体／平台／动力／驾驶](#g16) | 42 | 大主题分船体平台、动力操舵、武器船员和比赛四条小队列 |
| G17 | [捕鱼／海上采集与海岸作业](#g17) | 11 | 岸钓、海钓、渔网／拖网与海上投掷分别拆批 |
| G18 | [网络／分片／世界会话](#g18) | 22 | 多人／跨分片保存与同步，不把服务器管理器当可直接照搬 API |
| G19 | [内部调度／诊断／场景驱动](#g19) | 4 | 优先确认公开接入边界；可按底层 API／内部记录交付 |

### G01

**湿度／防护／温度扩展**。以下 22 个名称的归组待实现核验。

- `drownable`、`fan`、`firedetector`、`flotationdevice`、`frostybreather`、`inventoryitemmoisture`、`inventoryitemtemperature`、`lightningblocker`
- `moistureabsorbersource`、`moistureabsorberuser`、`nonslipgritpool`、`nonslipgritsource`、`nonslipgrituser`、`raindome`、`raindomewatcher`、`rainimmunity`
- `sanityauraadjuster`、`sheltered`、`smotherer`、`temperatureoverrider`、`watersource`、`wateryprotection`

### G02

**伤害／防御／控制扩展**。以下 30 个名称的归组待实现核验。

- `aoediminishingreturns`、`aoeweapon_base`、`aoeweapon_leap`、`aoeweapon_lunge`、`attackdodger`、`damagereflect`、`damagetracker`、`damagetypebonus`
- `damagetyperesist`、`dpstracker`、`electricattacks`、`epicscare`、`explosiveresist`、`groundpounder`、`healthbar`、`healthsyncer`
- `healthtrigger`、`parryweapon`、`pinnable`、`planarentity`、`resistance`、`rooted`、`slipperyfeet`、`slipperyfeettarget`
- `spooked`、`staffsanity`、`stunnable`、`targettracker`、`teamattacker`、`teamleader`

### G03

**采集／工具／主动使用物品**。以下 35 个名称的归组待实现核验。

- `activatable`、`appraisable`、`autoterraformer`、`bait`、`bottler`、`brush`、`brushable`、`catcher`
- `closeinspector`、`dataanalyzer`、`farmtiller`、`fencerotator`、`fillable`、`firebug`、`grabbable`、`gravediggable`
- `gravedigger`、`instrument`、`lighter`、`mapdeliverable`、`murderable`、`papereraser`、`repellent`、`searchable`
- `shaveable`、`shaver`、`summoningitem`、`tackler`、`tacklesketch`、`terraformer`、`terraformerremoveable`、`thief`
- `toggleableitem`、`useableequippeditem`、`useabletargeteditem`

### G04

**容器／物品持有／燃料／堆叠扩展**。以下 20 个名称的归组待实现核验。

- `battery`、`batteryuser`、`container_proxy`、`container_transform`、`containerinstallableitem`、`counter`、`cyclable`、`decay`
- `fueler`、`fuelmaster`、`inventoryitemholder`、`itemstore`、`linkeditem`、`linkeditemmanager`、`occupiable`、`occupier`
- `powerload`、`preservative`、`selfstacker`、`shelf`

### G05

**饮食与加工扩展**。以下 12 个名称的归组待实现核验。

- `compostingbin`、`cookbookupdater`、`digester`、`dryer`、`dryingracksaltcollector`、`feasts`、`foodaffinity`、`foodmemory`
- `forcecompostable`、`incinerator`、`incrementalproducer`、`saltlicker`

### G06

**农作／肥料／植物种植**。以下 19 个名称的归组待实现核验。

- `crop`、`farming_manager`、`farmplantable`、`farmplantstress`、`farmplanttendable`、`farmsoildrinker`、`fertilizable`、`fertilizer`
- `fertilizerresearchable`、`grower`、`harvestable`、`nutrients_visual_manager`、`plantable`、`plantregistryupdater`、`plantregrowth`、`plantresearchable`
- `pollinator`、`treegrowthsolution`、`witherable`

### G07

**生物生成／巢穴／群体**。以下 26 个名称的归组待实现核验。

- `batspawner`、`birdattractor`、`birdspawner`、`butterflyspawner`、`childspawner`、`circler`、`followerherder`、`followermemory`
- `formationfollower`、`formationleader`、`hatchable`、`herd`、`herdmember`、`hideout`、`homeseeker`、`leaderrollcall`
- `lureplantspawner`、`minionspawner`、`objectspawner`、`periodicspawner`、`petleash`、`rampingspawner`、`schoolspawner`、`shedder`
- `spawner`、`squadmember`

### G08

**驯养／骑乘／宠物状态**。以下 11 个名称的归组待实现核验。

- `beefalometrics`、`domesticatable`、`groomer`、`hitchable`、`hitcher`、`pethealthbar`、`rideable`、`rider`
- `saddler`、`skinner_beefalo`、`unsaddler`

### G09

**战斗行为与陆地运动协作**。以下 23 个名称的归组待实现核验。

- `amorphous`、`amphibiouscreature`、`carefulwalker`、`coach`、`commander`、`crewmember`、`distancefade`、`distancetracker`
- `embarker`、`grouptargeter`、`heavyobstaclephysics`、`heavyobstacleusetarget`、`joustsource`、`joustuser`、`mood`、`physicsmodifiedexternally`
- `playerspeedmult`、`pushable`、`replayproxy`、`strafer`、`stretcher`、`stuckdetection`、`unevenground`

### G10

**主动技能／读写／通道与机关**。以下 25 个名称的归组待实现核验。

- `attunable`、`attuner`、`aura`、`blinkstaff`、`channelable`、`channelcastable`、`channelcaster`、`chargingreticule`
- `circuitnode`、`cursable`、`curseditem`、`electricconnector`、`key`、`lock`、`machine`、`markable`
- `markable_proxy`、`mine`、`recallmark`、`remoteteleporter`、`simplebook`、`simplemagicgrower`、`spell`、`trap`
- `writeable`

### G11

**升级／套装／皮肤／外观能力**。以下 20 个名称的归组待实现核验。

- `custombuildmanager`、`forgerepair`、`forgerepairable`、`luckitem`、`luckuser`、`reloaditem`、`setbonus`、`skinner`
- `socketable`、`socketholder`、`symbolswapdata`、`transformer`、`upgradeable`、`upgrademodule`、`upgrademoduleowner`、`upgrademoduleremover`
- `upgrader`、`wardrobe`、`wax`、`waxable`

### G12

**建造扩展／拆装／记录和物品交互**。以下 22 个名称的归组待实现核验。

- `constructionbuilder`、`constructionbuilderuidata`、`constructionplans`、`constructionsite`、`giftreceiver`、`inkable`、`itemweigher`、`maprecorder`
- `pointofinterest`、`poppable`、`portablestructure`、`recipescanner`、`recipestockpile`、`researchpointconverter`、`roseinspectable`、`roseinspectableuser`
- `scrapbookable`、`shop`、`sittable`、`teacher`、`treasuremarked`、`weighable`

### G13

**玩家操作／视野／复活与迁移**。以下 22 个名称的归组待实现核验。

- `age`、`deathloothandler`、`experiencecollector`、`grue`、`hallucinations`、`hauntable`、`migrationmanager`、`migrationpetowner`
- `migrationpetsoverrider`、`playeractionpicker`、`playeravatardata`、`playercontroller`、`playerhearing`、`playerinspectable`、`playerlightningtarget`、`playermetrics`
- `playerspawner`、`playervision`、`revivablecorpse`、`seamlessplayerswapper`、`spectatorcorpse`、`touchstonetracker`

### G14

**显示／音效／文本／HUD 与局部地图**。以下 53 个名称的归组待实现核验。

- `ambientlighting`、`ambientsound`、`bloomer`、`blowinwind`、`camerafade`、`canopylightrays`、`canopyshadows`、`clientpickupsoundsuppressor`
- `colouradder`、`colouraddersync`、`colourcube`、`colourtweener`、`despawnfader`、`disappears`、`drawable`、`drawingtool`
- `driedsalticon`、`dsp`、`dynamicmusic`、`emitter`、`erasablepaper`、`fader`、`firefx`、`focalpoint`
- `freezefx`、`globaltrackingicon`、`groundshadowhandler`、`highlight`、`highlightchild`、`hudindicatable`、`hudindicatablemanager`、`hudindicatorwatcher`
- `lightpostpartner`、`lighttweener`、`maprevealable`、`maprevealer`、`mapspotrevealer`、`maxlightspawner`、`npc_talker`、`playertargetindicator`
- `projectedeffects`、`savedrotation`、`savedscale`、`scaler`、`shatterfx`、`sizetweener`、`spawnfader`、`talkable`
- `talker`、`transparentondrones`、`transparentonsanity`、`uianim`、`wisecracker`

### G15

**世界时间／天气／生态与区域**。以下 51 个名称的归组待实现核验。

- `area_trigger`、`area_unlock`、`areaaware`、`caveweather`、`clock`、`deciduoustreeupdater`、`discoverable`、`diseaseable`
- `fishschool`、`flotsamgenerator`、`forestpetrification`、`forestresourcespawner`、`frograin`、`knowndynamiclocations`、`knownlocations`、`messagebottlemanager`
- `meteorshower`、`oasis`、`ocean`、`oceancolor`、`oceanicemanager`、`penguinspawner`、`petrifiable`、`quaker`
- `regrowthmanager`、`retrofitcavemap_anr`、`retrofitforestmap_anr`、`ropebridgemanager`、`sandstorms`、`sandstormwatcher`、`seasons`、`sinkholespawner`
- `squidspawner`、`stormwatcher`、`undertile`、`vanish_on_sleep`、`vinebridgemanager`、`wanderingroutefollower`、`wavemanager`、`weather`
- `wildfires`、`worlddeciduoustreeupdater`、`worldmeteorshower`、`worldoverseer`、`worldroutefollower`、`worldroutes`、`worldsettings`、`worldsettingstimer`
- `worldstate`、`worldtemperature`、`worldwind`

### G16

**船体／平台／动力／驾驶**。以下 42 个名称的归组待实现核验。

- `anchor`、`boatai`、`boatcannon`、`boatcannonuser`、`boatcrew`、`boatdrag`、`boatdrifter`、`boatleak`
- `boatmagnet`、`boatmagnetbeacon`、`boatpatch`、`boatphysics`、`boatrace_proximitybeacon`、`boatrace_proximitychecker`、`boatracecrew`、`boatring`
- `boatringdata`、`boatrotator`、`boattrail`、`boattrailmover`、`deckcontainer`、`dockmanager`、`expertsailor`、`floater`
- `hull`、`hullhealth`、`mast`、`oar`、`oceanwhirlportalphysics`、`platformhopdelay`、`playerfloater`、`steeringwheel`
- `steeringwheeluser`、`submersible`、`walkableplatform`、`walkableplatformmanager`、`walkableplatformplayer`、`walkingplank`、`walkingplankuser`、`waterphysics`
- `winch`、`winchtarget`

### G17

**捕鱼／海上采集与海岸作业**。以下 11 个名称的归组待实现核验。

- `cookiecutterdrill`、`fishable`、`fishingnet`、`fishingnetvisualizer`、`fishingrod`、`oceanfishable`、`oceanfishinghook`、`oceanfishingrod`
- `oceanfishingtackle`、`oceanthrowable`、`oceantrawler`

### G18

**网络／分片／世界会话**。以下 22 个名称的归组待实现核验。

- `autosaver`、`corpsepersistmanager`、`playervoter`、`shard_autosaver`、`shard_clock`、`shard_daywalkerspawner`、`shard_mermkingwatcher`、`shard_players`
- `shard_seasons`、`shard_sinkholes`、`shard_wagbossinfo`、`shard_worldreset`、`shard_worldvoter`、`shardstate`、`shardtransactionsteps`、`townportalregistry`
- `uniqueid`、`uniqueprefabids`、`worldcharacterselectlobby`、`worldmigrator`、`worldreset`、`worldvoter`

### G19

**内部调度／诊断／场景驱动**。以下 4 个名称的归组待实现核验。

- `debugger`、`scenariorunner`、`specialeventsetup`、`updatelooper`

## 专项主题：236 个候选

低优先级仅表示当前通用开发收益预期，不能当成机制无用或代码失效的结论。实际角色／区域／活动接入可能依赖通用组；正式核验时允许迁组。

| 组 | 主题 | 候选数 | 拆批方向 |
|---|---|---:|---|
| S01 | [生肖与节日活动](#s01) | 33 | 活动专属对象／管理系统，按具体活动继续拆小批 |
| S02 | [玩法模式与旧机制](#s02) | 7 | 区分当前常规 DST、特殊模式与历史实现，不先假定弃用 |
| S03 | [角色专属与技能树](#s03) | 47 | 按角色拆批，先核验共享能力再保留专属分支 |
| S04 | [远古／档案馆／地下遗迹](#s04) | 24 | 原版区域专属结构与状态；名称仅作归组线索 |
| S05 | [月亮／暗影／裂隙／特殊灾害](#s05) | 50 | 按事件或阵营分别拆批，避免整组管理系统一并读取 |
| S06 | [Boss 与特殊群落管理](#s06) | 35 | 按 Boss／群落调用核验；生成管理与攻击行为不混为一页 |
| S07 | [小游戏／演出／装饰对象](#s07) | 40 | 按具体玩法逐项复查，通用能力可提前转入共享组 |

### S01

**生肖与节日活动**。以下 33 个名称的归组待实现核验。

- `bathbomb`、`bathbombable`、`bathingpool`、`bedazzlement`、`bedazzler`、`carnivaldecor`、`carnivaldecorranker`、`carnivalevent`
- `carnivalgamefeedable`、`carnivalgameitem`、`carnivalgameshooter`、`carnivalhostsummon`、`gingerbreadhunter`、`halloweenmoonmutable`、`halloweenpotionmoon`、`pumpkincarvable`
- `pumpkincarver`、`pumpkinhatcarvable`、`winter_treeseed`、`wintersfeasttable`、`wintersurprisespawner`、`wintertreegiftable`、`yotb_sewer`、`yotb_skinunlocker`
- `yotb_stagemanager`、`yotb_stager`、`yotc_racecompetitor`、`yotc_raceprizemanager`、`yotc_racestart`、`yotc_racestats`、`yotd_raceprizemanager`、`yoth_hecklermanager`
- `yoth_knightmanager`

### S02

**玩法模式与旧机制**。以下 7 个名称的归组待实现核验。

- `lavaarenaeventstate`、`lavaarenamobtracker`、`lavaarenamusic`、`quagmire_hangriness`、`quagmire_music`、`quagmire_recipebook`、`quagmire_recipeprices`

### S03

**角色专属与技能树**。以下 47 个名称的归组待实现核验。

- `avengingghost`、`balloonmaker`、`battleborn`、`beard`、`beaverness`、`bloomness`、`book`、`dumbbelllifter`
- `friendlevels`、`ghostbabysitter`、`ghostgestalter`、`ghostlybond`、`ghostlyelixir`、`ghostlyelixirable`、`gym`、`itemaffinity`
- `itemmimic`、`magician`、`magiciantool`、`maxwelltalker`、`mightiness`、`mightydumbbell`、`mightygym`、`nabbag`
- `oldager`、`pocketwatch`、`pocketwatch_dismantler`、`possessedaxe`、`reader`、`sentientaxe`、`singinginspiration`、`skilltreeupdater`
- `slingshotmodder`、`slingshotmods`、`soul`、`souleater`、`spidermutator`、`strongman`、`werebeast`、`wereeater`
- `wereness`、`wobycourier`、`wobyrack`、`wx78_abilitycooldowns`、`wx78_dronescouttracker`、`wx78_shield`、`wx78_taserbuildup`

### S04

**远古／档案馆／地下遗迹**。以下 24 个名称的归组待实现核验。

- `abysspillargroup`、`archivemanager`、`caveins`、`charliecutscene`、`chessunlocks`、`fumarolelocaltemperature`、`fumaroletool`、`grottowarmanager`
- `grottowaterfallsoundcontroller`、`nightmareclock`、`ruinsshadelingspawner`、`trapfumaroleburning`、`vault_floor_helper`、`vault_teleporter`、`vaultmusiclistener`、`vaultorbteleportdestination`
- `vaultorbteleporter`、`vaultroom`、`vaultroommanager`、`vaulttorchgrid`、`wagboss_tracker`、`wagpunk_arena_manager`、`wagpunk_floor_helper`、`wagpunk_manager`

### S05

**月亮／暗影／裂隙／特殊灾害**。以下 50 个名称的归组待实现核验。

- `acidbatwavemanager`、`acidinfusible`、`acidlevel`、`brightmarespawner`、`decoratedgrave_ghostmanager`、`gelblobspawner`、`gestaltcage`、`gestaltcapturable`
- `groundcreep`、`lunaralterguardianspawner`、`lunarfissureburning`、`lunarhailbirdsoundmanager`、`lunarhailbuildup`、`lunarhailmanager`、`lunarplant_tentacle_weapon`、`lunarriftmutationsmanager`
- `lunarsupernovablocker`、`lunarsupernovaburning`、`lunarthrall_plantspawner`、`miasmamanager`、`miasmawatcher`、`moonaltarlink`、`moonaltarlinktarget`、`moonbeastspawner`
- `moonrelic`、`moonsparkchargeable`、`moonstormlightningmanager`、`moonstormmanager`、`moonstorms`、`moonstormstaticcapturable`、`moonstormstaticcatcher`、`moonstormwatcher`
- `moontrader`、`mutatedbirdmanager`、`mutatedbuzzardcircler`、`mutatedbuzzardmanager`、`nightlightmanager`、`riftspawner`、`riftthralltype`、`shadowcreaturespawner`
- `shadowdominance`、`shadowhandspawner`、`shadowlevel`、`shadowparasitemanager`、`shadowsubmissive`、`shadowthrall_mimics`、`shadowthrallmanager`、`socket_shadow_harvester`
- `socket_shadow_heart`、`socket_shadow_mimicry`

### S06

**Boss 与特殊群落管理**。以下 35 个名称的归组待实现核验。

- `beargerspawner`、`centipedebody`、`clockworktracker`、`crabkingspawner`、`daywalkerspawner`、`deerclopsspawner`、`deerherding`、`deerherdspawner`
- `desolationspawner`、`forestdaywalkerspawner`、`guardian`、`hermitcrab_relocation_manager`、`hounded`、`houndedtarget`、`hunter`、`klaussackkey`
- `klaussacklock`、`klaussackloot`、`klaussackspawner`、`kramped`、`lordfruitflytrigger`、`malbatrossspawner`、`mermcandidate`、`mermkingmanager`
- `moosespawner`、`piratespawner`、`rabbitkingmanager`、`sharkboimanager`、`sharkboimanagerhelper`、`sharklistener`、`sisturnregistry`、`skeletonsweeper`
- `toadstoolspawner`、`wanderingtraderspawner`、`winonateleportpadmanager`

### S07

**小游戏／演出／装饰对象**。以下 40 个名称的归组待实现核验。

- `cattoy`、`crittertraits`、`furnituredecor`、`furnituredecortaker`、`golfable`、`golfballspinner`、`golfclub`、`golfclub_reticule`
- `golfpropitem`、`hideandseeker`、`hideandseekgame`、`hideandseekhider`、`hideandseekhidingspot`、`inspectaclesparticipant`、`kitcoon`、`kitcoonden`
- `minigame`、`minigame_participator`、`minigame_spectator`、`nis`、`pearldecorationscore`、`playbill`、`playbill_lecturn`、`playingcard`
- `playingcardsmanager`、`singable`、`singingshellmanager`、`singingshelltrigger`、`snowballmanager`、`snowballmelting`、`snowmandecor`、`snowmandecoratable`
- `stageactingprop`、`stageactor`、`storyteller`、`storytellingprop`、`tributable`、`trophyscale`、`vase`、`vasedecoration`

## 职责待确认：3 个候选

`madsciencelab`、`prophider`、`questowner`。仅登记，不给出未核验的通用定位、依赖或示例；明确宿主与实际调用后归入相应主题。

## 独立处理的文件

以下沿用覆盖清单已有分类，本轮没有读取实现来重判身份。

**19 个 replica**：优先跟随所用组件的客户端读取／同步缺口核验，不默认给每个 replica 生成与服务端页重复的独立页面。

- `builder_replica`、`combat_replica`、`constructionsite_replica`、`container_replica`、`equippable_replica`、`fishingrod_replica`、`follower_replica`、`health_replica`
- `hunger_replica`、`inventory_replica`、`inventoryitem_replica`、`moisture_replica`、`named_replica`、`oceanfishingrod_replica`、`rider_replica`、`sanity_replica`
- `sheltered_replica`、`stackable_replica`、`writeable_replica`

**3 个辅助模块**：`map`、`quagmire_map`、`spdamageutil`。按地图或特殊伤害接口任务处理，核验后决定 API／主题页归属，不放进普通组件页统计。

**1 个空文件**：`bosstargeter`。保留清单中的空文件标记，不生成空知识页；新版本出现内容时再重新判定。

## 编写前的并行缺口

- 配方与建筑批次需要注册 API、材料／科技／过滤类别和必要资源，不靠 builder／prototyper 页面替代所有入口。
- 生物与控制批次需要 Brain、SG、动作和宿主清理；组件共现不证明行为调用完整。
- 玩家操作、技能、容器与多人同步需要相应 replica、客户端动作／RPC／预测规则，不能用服务端组件案例证明网络可用。
- 随身暖石组合仍需共用持有者、Light 与转移／删除／重载／死亡验证；只补组件数量不能替代组合检视。
- 页面增长时按现有索引大小阈值拆分类，只登记已建页；全量工程清单不进入根／Skill／查询结果，名称直查仍可跳过导航。

## 本轮核对范围

核对目录路径 821 条与 CSV 逐项相等、48 已建页与 built 条目一致、750 名称全覆盖且无重复、19／3／1 独立文件分类及新增文档链接。未读取待建实现、未准备或解压源码、未生成第六批页面、未运行提取器／LuaLS／游戏或工具测试环境。本轮仅编排计划，原覆盖状态、知识页与历史日志保持，日志只追加本轮记录。
