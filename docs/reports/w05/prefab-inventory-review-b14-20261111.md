# Prefab 非装备物品栏 B14 外部源码检视

2026-11-11。实际安装的 Steam manifest 与 `raw/current/BUILD.txt` 均为 **26474873**。依据 `raw/current/scripts`，独立核对当前工作树的 8 页、农业叶索引 8 行、父索引入口及本批统计。结论为 **需修订后定向复验**。P1×3、P2×14、P3×1。

当前状态（第四轮外部定向复验）：**R01–R18 全部通过，已收口**。见[本轮结论](#第四轮外部定向复验)。

修订按 R01–R18 编号回填本报告。每条回执列实际修改位置、自验证据及尚需主代理同步的入口。修正后的数值和门禁同时回扫摘要、快速判断、正文、导航、陷阱、示例、验证记录和索引中命中位置。

## 源码事实与页面命中

| 核对项 | 独立证据与条件 | 页面命中与差异 |
| --- | --- | --- |
| 农田肥料的部署与消费 | CUSTOM 只认农田土壤，保留材料在物品栏，先逐格加基础三养分，再扣一份，最后调用附加回调。三养分分别夹到 0–100。[standardcomponents.lua L1797–1831](../../../raw/current/scripts/standardcomponents.lua#L1797-L1831)、[components/fertilizer.lua L30–47](../../../raw/current/scripts/components/fertilizer.lua#L30-L47)、[components/farming_manager.lua L394–398](../../../raw/current/scripts/components/farming_manager.lua#L394-L398)。 | 两页公共结算吻合。燃料例外、鸟笼和树果酱动作入口见 R05–R07。 |
| 三类部署路径 | PLANT 先陆地非硬地再检查落点，DEFAULT 先鼠标目标与可通行性再检查落点，TURF 调 `CanPlaceTurfAtPoint`。服务端另有 `IsDeployable` 前置，动作执行把 `mouseover` 传 nil。[components/map.lua L162–175](../../../raw/current/scripts/components/map.lua#L162-L175)、[components/map.lua L263–282](../../../raw/current/scripts/components/map.lua#L263-L282)、[components/map.lua L316–326](../../../raw/current/scripts/components/map.lua#L316-L326)、[components/deployable.lua L89–147](../../../raw/current/scripts/components/deployable.lua#L89-L147)。 | 三类模式分工和 TURF 不查间距已写。补门禁与结果边界见 R01、R09–R11。 |
| 四种树种的种植结果 | 三个 addcone 注册与独立 marblebean 均为 PLANT，目标是四种树苗。共享苗厂构造时起表，分别长成四类成树。[prefabs/pinecone.lua L93–137](../../../raw/current/scripts/prefabs/pinecone.lua#L93-L137)、[prefabs/marblebean.lua L13–20](../../../raw/current/scripts/prefabs/marblebean.lua#L13-L20)、[prefabs/planted_tree.lua L117–152](../../../raw/current/scripts/prefabs/planted_tree.lua#L117-L152)、[prefabs/planted_tree.lua L183–189](../../../raw/current/scripts/prefabs/planted_tree.lua#L183-L189)。 | 目标与 225±75 秒的名义样本吻合。索引数值、重起计时与来源边界见 R02、R08、R17。 |
| 活树根、雕像与珠宝 | 活树根部署生成苗，两段 900±45 秒后换成树。雕像的两个消费入口各自调用唤醒和候选预约。珠宝自身无 deployable，由戴帽者的 petleash 链消费地面充能月玻璃。[prefabs/livingtree_root_planted.lua L14–46](../../../raw/current/scripts/prefabs/livingtree_root_planted.lua#L14-L46)、[prefabs/leif_idol.lua L60–124](../../../raw/current/scripts/prefabs/leif_idol.lua#L60-L124)、[prefabs/lunar_seed.lua L33–49](../../../raw/current/scripts/prefabs/lunar_seed.lua#L33-L49)、[prefabs/hats.lua L3286–3319](../../../raw/current/scripts/prefabs/hats.lua#L3286-L3319)。 | 树苗阶段和用途分流吻合。预约上限、示例期望与帽子门禁见 R03、R15。 |
| 食人花种子与蜘蛛卵 | 生成各自的世界态，食人花退出 spawn 推 freshspawn，随后排休眠。蜘蛛巢初始化阶级、外部放员计时及平台分支各自结算。[prefabs/lureplant_bulb.lua L9–18](../../../raw/current/scripts/prefabs/lureplant_bulb.lua#L9-L18)、[stategraphs/SGlureplant.lua L291–318](../../../raw/current/scripts/stategraphs/SGlureplant.lua#L291-L318)、[prefabs/lureplant.lua L137–150](../../../raw/current/scripts/prefabs/lureplant.lua#L137-L150)、[prefabs/spiderden.lua L608–626](../../../raw/current/scripts/prefabs/spiderden.lua#L608-L626)、[worldsettingsutil.lua L34–59](../../../raw/current/scripts/worldsettingsutil.lua#L34-L59)。 | 产物、休眠和平台停长／停补吻合。放员、巢容量与女王结算见 R04。 |
| 三件移栽物和世界态 | 公共厂按 SpawnPrefab 成功与否扣件，三个世界态各有独立挖起和回种分支。[prefabs/plantables.lua L17–36](../../../raw/current/scripts/prefabs/plantables.lua#L17-L36)、[prefabs/marsh_bush.lua L24–34](../../../raw/current/scripts/prefabs/marsh_bush.lua#L24-L34)、[prefabs/rock_avocado_bush.lua L142–163](../../../raw/current/scripts/prefabs/rock_avocado_bush.lua#L142-L163)、[prefabs/monkeytail.lua L14–25](../../../raw/current/scripts/prefabs/monkeytail.lua#L14-L25)。 | 三种移栽件及三个 placer 分工、挖起产物和回种空株／枯死吻合。部署动作状态与命名见 R10、R18。 |
| 地皮注册与获得 | 工厂遍历 `GroundTiles.turf` 并拼名，叉子通过 terraformer 和通用掉落处理转回物品。[prefabs/turfs.lua L75–87](../../../raw/current/scripts/prefabs/turfs.lua#L75-L87)、[tilemanager.lua L205–209](../../../raw/current/scripts/tilemanager.lua#L205-L209)、[components/terraformer.lua L12–32](../../../raw/current/scripts/components/terraformer.lua#L12-L32)、[componentutil.lua L31–45](../../../raw/current/scripts/componentutil.lua#L31-L45)。 | 40 个注册、40 条同名配方与其科技分布吻合。另有产物别名配方、蓝图边界和自定义回调，见 R11–R12。 |
| 插穗材料与目标转换 | UPGRADE 执行检查材料与目标组件，目标回调前扣 1，普通海蚀柱和水草岩均换成海芽。海芽的两天重生等待和随后产藤壶分开。[actions.lua L4056–4068](../../../raw/current/scripts/actions.lua#L4056-L4068)、[components/upgradeable.lua L75–93](../../../raw/current/scripts/components/upgradeable.lua#L75-L93)、[prefabs/seastack.lua L61–80](../../../raw/current/scripts/prefabs/seastack.lua#L61-L80)、[prefabs/waterplant_rock.lua L36–53](../../../raw/current/scripts/prefabs/waterplant_rock.lua#L36-L53)、[prefabs/waterplant_baby.lua L161–183](../../../raw/current/scripts/prefabs/waterplant_baby.lua#L161-L183)。 | 扣件与替换主体吻合。候选与执行、月相与读档边界见 R13–R14。 |

## P1

### R01 地皮示例挂在不会触发的地图组件回调

- 位置 [turfs L140–156](../../../wiki/reference/prefabs/turfs.md#L140-L156)。
- 影响 照抄后客户端和服务端仍沿用 DIRT 判定。世界 prefab 先 require 给 `Map` 加方法的 Lua 文件，构造时走引擎 `inst.entity:AddMap()`。示例用的 `AddComponentPostInit("map", ...)` 则由 `EntityScript:AddComponent(name)` 执行。用 `rg -n 'AddComponent\(["\x27]map["\x27]\)|AddMap\(' raw/current/scripts -g '*.lua'` 检索本版 Lua，实际命中世界构造的 AddMap，没有 Lua AddComponent("map") 入口。[prefabs/world.lua L1–2](../../../raw/current/scripts/prefabs/world.lua#L1-L2)、[prefabs/world.lua L443–452](../../../raw/current/scripts/prefabs/world.lua#L443-L452)、[components/map.lua L162–175](../../../raw/current/scripts/components/map.lua#L162-L175)、[modutil.lua L566–573](../../../raw/current/scripts/modutil.lua#L566-L573)、[entityscript.lua L648–667](../../../raw/current/scripts/entityscript.lua#L648-L667)。
- 修法与验收 改用本版两端实际可达的入口包装 `Map.CanPlaceTurfAtPoint`，核对包装安装在原版方法定义之后。保留原判定与新增可种植分支，注明全局影响。对照原可铺 DIRT、普通可种植陆地、硬地和水面四种输入，并分别说明服务端执行与客端预览证据。

### R02 树种叶索引复制了别组的物品数值

- 位置 [farming-and-planting-index L17](../../../wiki/reference/prefabs/farming-and-planting-index.md#L17-L17)，对照 [tree-seeds L53–63](../../../wiki/reference/prefabs/tree-seeds.md#L53-L63)、[tree-seeds L113–116](../../../wiki/reference/prefabs/tree-seeds.md#L113-L116)。
- 影响 查索引会读到四件均堆叠 10、燃料 180 秒、可燃 30 秒。四件上限都是 40，只有三个 addcone 树种有 15 秒燃料和 7.5 秒燃尽。大理石豌豆无 fuel 与 burnable。[prefabs/pinecone.lua L95–106](../../../raw/current/scripts/prefabs/pinecone.lua#L95-L106)、[prefabs/marblebean.lua L40–59](../../../raw/current/scripts/prefabs/marblebean.lua#L40-L59)、[tuning.lua L81–83](../../../raw/current/scripts/tuning.lua#L81-L83)、[tuning.lua L1673–1683](../../../raw/current/scripts/tuning.lua#L1673-L1683)。
- 修法与验收 主代理同步叶索引的三项数值及变体边界，正文正确表格可复用。验收分别读回 addcone 三件和大理石豌豆的检索摘要。

### R03 雕像示例的验收值仍用原版 2，索引把预约上限写成固定产出

- 位置 [special-tree-planting L6](../../../wiki/reference/prefabs/special-tree-planting.md#L6-L6)、[special-tree-planting L94–102](../../../wiki/reference/prefabs/special-tree-planting.md#L94-L102)、[special-tree-planting L133–152](../../../wiki/reference/prefabs/special-tree-planting.md#L133-L152)、[farming-and-planting-index L18](../../../wiki/reference/prefabs/farming-and-planting-index.md#L18-L18)。
- 影响 示例赋值 4 后，两件一叠烧毁的预约上限是 8，燃料入口是 4，L144 却仍让读者验 4／2。原版分别是 `2 × 当时堆叠` 与最多 2，附近合格树不足时会更少。候选必须实际有 `TransformIntoLeif` 或 `StartMonster` 才减额度。桦栗树延迟怪物化，和常青树生成树精是两种结果。[prefabs/leif_idol.lua L60–83](../../../raw/current/scripts/prefabs/leif_idol.lua#L60-L83)、[prefabs/leif_idol.lua L95–124](../../../raw/current/scripts/prefabs/leif_idol.lua#L95-L124)、[tuning.lua L7825–7826](../../../raw/current/scripts/tuning.lua#L7825-L7826)。
- 修法与验收 示例明确原版 4／2 与改值后 8／4，摘要和索引把“翻倍／固定”同步成按每件 2 的上限，保留两类结果及延迟条件。用合格树充足和不足两个情形检查期望值。

## P2

### R04 蜘蛛巢的容量、定时放员和女王门禁要分别写

- 位置 [plant-spawners L6](../../../wiki/reference/prefabs/plant-spawners.md#L6-L6)、[plant-spawners L22](../../../wiki/reference/prefabs/plant-spawners.md#L22-L22)、[plant-spawners L81–84](../../../wiki/reference/prefabs/plant-spawners.md#L81-L84)、[plant-spawners L139–148](../../../wiki/reference/prefabs/plant-spawners.md#L139-L148)、[farming-and-planting-index L19](../../../wiki/reference/prefabs/farming-and-planting-index.md#L19-L19)。
- 证据 5±0.5 秒／90±9 秒是普通放员／补员计时。外部计时到点仍看开关、有库存、能否离屏生成等，睡眠时可排队。地表也受冻结等门禁，平台分支只明确停生长和再生，其他事件仍可触发释放或归巢。3 级容量默认 9，春季构造时取 `floor(9 × 1.33) = 11`，生命仍 1000。女王的 60／4 用于决定巢是否保留，达到计数阈值仍可生成女王并删除旧巢，前面另有世界开关、装饰、睡袋使用和附近玩家门禁。[worldsettingsutil.lua L34–59](../../../raw/current/scripts/worldsettingsutil.lua#L34-L59)、[worldsettingsutil.lua L87–121](../../../raw/current/scripts/worldsettingsutil.lua#L87-L121)、[components/childspawner.lua L667–684](../../../raw/current/scripts/components/childspawner.lua#L667-L684)、[components/childspawner.lua L717–727](../../../raw/current/scripts/components/childspawner.lua#L717-L727)、[prefabs/spiderden.lua L142–149](../../../raw/current/scripts/prefabs/spiderden.lua#L142-L149)、[prefabs/spiderden.lua L262–309](../../../raw/current/scripts/prefabs/spiderden.lua#L262-L309)、[prefabs/spiderden.lua L398–454](../../../raw/current/scripts/prefabs/spiderden.lua#L398-L454)、[simutil.lua L560–565](../../../raw/current/scripts/simutil.lua#L560-L565)、[tuning.lua L433](../../../raw/current/scripts/tuning.lua#L433-L433)。
- 验收 把周期写成有条件的定时尝试。船上保留“停补不停普通放员”的窄结论。示例分别给非春季和春季的新巢容量，女王阈值明确作用在保留旧巢。

### R05 鸟笼产鸟粪的门禁写错投喂类别

- 位置 [fertilizer-materials L74](../../../wiki/reference/prefabs/fertilizer-materials.md#L74-L74)、[farming-and-planting-index L15](../../../wiki/reference/prefabs/farming-and-planting-index.md#L15-L15)。
- 证据 普通鸟会接受普通／作物种子，有对应种子的作物优先出种子，肉先走蛋分支，普通变异鸟在非肉分支给腐烂食物，月变异鸟走月亮碎片独立分支。只有通过投喂后、落到普通鸟的无对应种子分支，才以 0.33 基础概率叠加喂食者运气抽鸟粪。普通 `seeds` 就是该入口的反例。[prefabs/birdcage.lua L101–145](../../../raw/current/scripts/prefabs/birdcage.lua#L101-L145)、[prefabs/birdcage.lua L155–180](../../../raw/current/scripts/prefabs/birdcage.lua#L155-L180)、[prefabs/birdcage.lua L183–196](../../../raw/current/scripts/prefabs/birdcage.lua#L183-L196)。
- 验收 把“投喂非种子食物”换成实际门禁。索引提到鸟笼时保留普通鸟／无对应种子这一层。

### R06 肥料索引漏了燃料回调例外，嗡嗡肥料缺原版加成值与技能

- 位置 [fertilizer-materials L6](../../../wiki/reference/prefabs/fertilizer-materials.md#L6-L6)、[fertilizer-materials L22](../../../wiki/reference/prefabs/fertilizer-materials.md#L22-L22)、[fertilizer-materials L52–56](../../../wiki/reference/prefabs/fertilizer-materials.md#L52-L56)、[fertilizer-materials L87–106](../../../wiki/reference/prefabs/fertilizer-materials.md#L87-L106)、[fertilizer-materials L112–116](../../../wiki/reference/prefabs/fertilizer-materials.md#L112-L116)、[farming-and-planting-index L15](../../../wiki/reference/prefabs/farming-and-planting-index.md#L15-L15)。
- 证据 鸟粪与粪肥在 quagmire 移除燃料取用回调，燃料值仍在。三个取火焰 FX 的回调在 FX 缺失或无效时退回取用者坐标，鸟粪直接取取用者坐标。嗡嗡肥料制作需 `wurt_mosquito_craft_2`，农田部署扣件后，`wurt_mosquito_craft_3` 才另给三类各 8，格满仍按同一上限夹。示例加 8 点单类养分属于自定义值。[prefabs/guano.lua L24–26](../../../raw/current/scripts/prefabs/guano.lua#L24-L26)、[prefabs/guano.lua L102–108](../../../raw/current/scripts/prefabs/guano.lua#L102-L108)、[prefabs/poop.lua L24–33](../../../raw/current/scripts/prefabs/poop.lua#L24-L33)、[prefabs/poop.lua L108–114](../../../raw/current/scripts/prefabs/poop.lua#L108-L114)、[prefabs/compost.lua L15–24](../../../raw/current/scripts/prefabs/compost.lua#L15-L24)、[prefabs/mosquitofertalizer.lua L29–41](../../../raw/current/scripts/prefabs/mosquitofertalizer.lua#L29-L41)、[prefabs/mosquitofertalizer.lua L68–74](../../../raw/current/scripts/prefabs/mosquitofertalizer.lua#L68-L74)、[recipes.lua L115](../../../raw/current/scripts/recipes.lua#L115-L115)、[tuning.lua L8290](../../../raw/current/scripts/tuning.lua#L8290-L8290)。
- 验收 公共燃料叙述保留例外与回退，加成明确制作技能、使用技能和基值／额外值。验证普通与 quagmire 各取一件，农田分别用仅二阶、已三阶角色比对源码期望。

### R07 树果酱的“全部四条入口”漏掉独立 FERTILIZE 动作

- 位置 [treegrowthsolution L6](../../../wiki/reference/prefabs/treegrowthsolution.md#L6-L6)、[treegrowthsolution L20–23](../../../wiki/reference/prefabs/treegrowthsolution.md#L20-L23)、[treegrowthsolution L38–45](../../../wiki/reference/prefabs/treegrowthsolution.md#L38-L45)、[treegrowthsolution L57](../../../wiki/reference/prefabs/treegrowthsolution.md#L57-L57)、[treegrowthsolution L93–103](../../../wiki/reference/prefabs/treegrowthsolution.md#L93-L103)、[farming-and-planting-index L16](../../../wiki/reference/prefabs/farming-and-planting-index.md#L16-L16)。
- 证据 树果酱挂标准 fertilizer，导航一行也承认旧路径。受理的旧目标或自肥可走 `ACTIONS.FERTILIZE`，只有 applied 为真才调用 `Fertilizer:OnApplied`。这与用 `ACTIONS.DEPLOY` 对农田格加三养分是两条动作入口，共享扣件实现。[prefabs/treegrowthsolution.lua L52–59](../../../raw/current/scripts/prefabs/treegrowthsolution.lua#L52-L59)、[componentactions.lua L1241–1254](../../../raw/current/scripts/componentactions.lua#L1241-L1254)、[componentactions.lua L2712–2717](../../../raw/current/scripts/componentactions.lua#L2712-L2717)、[actions.lua L1812–1841](../../../raw/current/scripts/actions.lua#L1812-L1841)、[standardcomponents.lua L1797–1810](../../../raw/current/scripts/standardcomponents.lua#L1797-L1810)。
- 验收 区分四类用途与实际动作入口，或明确四条仅为本文主讲范围。扣件图保留同一个 `OnApplied`，正文和索引给出旧动作入口。

### R08 树苗熄灭后重抽整段计时，换苗示例缺必需接口

- 位置 [tree-seeds L86–95](../../../wiki/reference/prefabs/tree-seeds.md#L86-L95)、[tree-seeds L129–145](../../../wiki/reference/prefabs/tree-seeds.md#L129-L145)，以及 L156–164 的读档／验收表述。
- 证据 原版 `stopgrowing` 调 `StopTimer`，熄灭的 `startgrowing` 没有剩余时长可续，会重抽 `GetRandomWithVariance`。换自定义苗时 `plant` 除了 StartGrowing、Transform，还无条件调用 `sapling.SoundEmitter:PlaySound`。遗留存档若携带 `growtime`，`OnLoad` 同样读取已改的 `_spawn_prefab` 并走 `plant`。[prefabs/planted_tree.lua L82–93](../../../raw/current/scripts/prefabs/planted_tree.lua#L82-L93)、[prefabs/planted_tree.lua L164–173](../../../raw/current/scripts/prefabs/planted_tree.lua#L164-L173)、[prefabs/pinecone.lua L3–9](../../../raw/current/scripts/prefabs/pinecone.lua#L3-L9)、[prefabs/pinecone.lua L40–44](../../../raw/current/scripts/prefabs/pinecone.lua#L40-L44)。
- 验收 明确取消后重抽、列完整自定义苗契约，并把“只影响新部署”限定到通常存档，注明旧 `growtime` 兼容分支。用燃烧前剩余时间较短的样本区分重抽和续表。

### R09 部署的“空位”省略禁建和客户端／服务器门禁

- 位置 [plant-spawners L6](../../../wiki/reference/prefabs/plant-spawners.md#L6-L6)、[plant-spawners L20](../../../wiki/reference/prefabs/plant-spawners.md#L20-L20)、[plant-spawners L50–58](../../../wiki/reference/prefabs/plant-spawners.md#L50-L58)、[plant-spawners L145](../../../wiki/reference/prefabs/plant-spawners.md#L145-L145)、[tree-seeds L65–72](../../../wiki/reference/prefabs/tree-seeds.md#L65-L72)、[special-tree-planting L42–46](../../../wiki/reference/prefabs/special-tree-planting.md#L42-L46)、[farming-and-planting-index L17–19](../../../wiki/reference/prefabs/farming-and-planting-index.md#L17-L19)。移栽页公共链引用 [plantables-remaining L21–23](../../../wiki/reference/prefabs/plantables-remaining.md#L21-L23)。
- 证据 DEFAULT 与 PLANT 都调用 `IsDeployPointClear`，查实体前先拒绝竖起屏障的废铁战车竞技场禁建带。两类均有 `IsDeployable` 前置，骑乘／手持浮标的普通种植件过不去。DEFAULT 的 mouseover 条件在传入目标时有效，实际 `ACTIONS.DEPLOY` 和 `Deploy` 复查传 nil。预览把真实 mouseover 传给副本。[components/map.lua L263–282](../../../raw/current/scripts/components/map.lua#L263-L282)、[components/map.lua L316–326](../../../raw/current/scripts/components/map.lua#L316-L326)、[components/deployable.lua L89–102](../../../raw/current/scripts/components/deployable.lua#L89-L102)、[components/deployable.lua L132–145](../../../raw/current/scripts/components/deployable.lua#L132-L145)、[actions.lua L1607–1613](../../../raw/current/scripts/actions.lua#L1607-L1613)、[components/playercontroller.lua L2907–2911](../../../raw/current/scripts/components/playercontroller.lua#L2907-L2911)。
- 验收 公共规则只写一处完整证据，其他页链接。把“只要求可通行＋空位”收窄为一般落点条件，区分 mouseover 所属阶段。用普通硬地、骑乘和无实体障碍的禁建带三个反例交叉验证。

### R10 移栽回调未生成时不销毁，但动作已拆件且仍成功

- 位置 [plantables-remaining L42](../../../wiki/reference/prefabs/plantables-remaining.md#L42-L42)、[L87–88](../../../wiki/reference/prefabs/plantables-remaining.md#L87-L88)。[berry-bushes L67](../../../wiki/reference/prefabs/berry-bushes.md#L67-L67) 也称堆叠不减，[grass-saplings L49](../../../wiki/reference/prefabs/grass-saplings.md#L49-L49) 已区分部署返回值和实际生成。
- 证据 `plantables.ondeploy` 仅在世界态成功生成后移除物品。玩家 DEPLOY 在进回调前已经从栏里拆出一件并清掉原槽信息，只有 `Deploy` 返回失败才 GiveItem。回调没有失败值传递，`Deploy` 仍推 deployitem，对 `deployedplant` 也推 itemplanted，并返回 true。故不能把 SpawnPrefab 空返回写成“整叠不动”或“动作回滚”。[prefabs/plantables.lua L17–36](../../../raw/current/scripts/prefabs/plantables.lua#L17-L36)、[actions.lua L1622–1637](../../../raw/current/scripts/actions.lua#L1622-L1637)、[components/inventory.lua L1390–1403](../../../raw/current/scripts/components/inventory.lua#L1390-L1403)、[components/deployable.lua L132–146](../../../raw/current/scripts/components/deployable.lua#L132-L146)。
- 验收 区分回调没有销毁单体、玩家动作已经拆件、动作成功与事件仍发。直接共享页如有相同断言交主代理同步，正常生成的一件消耗结论保留。

### R11 地皮执行成功、实际换地与挖起回调各有条件

- 位置 [turfs L22](../../../wiki/reference/prefabs/turfs.md#L22-L22)、[turfs L85–88](../../../wiki/reference/prefabs/turfs.md#L85-L88)、[turfs L111](../../../wiki/reference/prefabs/turfs.md#L111-L111)、[turfs L130–131](../../../wiki/reference/prefabs/turfs.md#L130-L131)、[turfs L160–164](../../../wiki/reference/prefabs/turfs.md#L160-L164)、[farming-and-planting-index L21](../../../wiki/reference/prefabs/farming-and-planting-index.md#L21-L21)。
- 证据 铺设回调调用 SetTile 后无条件扣件，Map 包装转调引擎后推事件，Lua 没有核实地块实际变化的检查。L87 的“未铺成但扣件不会发生”把动作返回与地块结果混成一件。挖起由 terraformer 先过 CanTerraform／CanPlow，再按 `self.turf`、下层地块、DIRT 的顺序选新地。若有 `onterraformfn`，只走自定义回调，原版 farm_plow 就实际接入。[prefabs/turfs.lua L9–17](../../../raw/current/scripts/prefabs/turfs.lua#L9-L17)、[components/map.lua L3–8](../../../raw/current/scripts/components/map.lua#L3-L8)、[components/deployable.lua L132–146](../../../raw/current/scripts/components/deployable.lua#L132-L146)、[components/terraformer.lua L12–32](../../../raw/current/scripts/components/terraformer.lua#L12-L32)、[prefabs/farm_plow.lua L228–233](../../../raw/current/scripts/prefabs/farm_plow.lua#L228-L233)。
- 验收 明确落点失败与动作成功但未校验最终改地是两类情形。叉子保留通用挖起结论，去掉“任何带组件的工具都走 HandleDugGround”的说法。

### R12 地皮配方要区分同名登记、别名产物与动态蓝图

- 位置 [turfs L105–119](../../../wiki/reference/prefabs/turfs.md#L105-L119)、[turfs L163–171](../../../wiki/reference/prefabs/turfs.md#L163-L171)、[farming-and-planting-index L21](../../../wiki/reference/prefabs/farming-and-planting-index.md#L21-L21)。
- 证据 40 是同名 `Recipe2("turf_…")` 数。沃特另有 `wurt_turf_marsh`，`TECH.NONE`、`builder_tag="merm_builder"`、产出 4 张沼泽地皮。两个天体配方带 `nounlock`。蓝图工厂会动态登记合资格的 `<recipe>_blueprint`，`turf_shellbeach_blueprint` 又是实际商店配方产物，L163 把它写成无对应注册与本页 L110 冲突。环形山配方的 nounlock 会排除通用具体蓝图注册，PO 键本身不能作为注册证据。[recipes.lua L109–110](../../../raw/current/scripts/recipes.lua#L109-L110)、[recipes.lua L704–705](../../../raw/current/scripts/recipes.lua#L704-L705)、[recipes.lua L842–843](../../../raw/current/scripts/recipes.lua#L842-L843)、[recipes.lua L934](../../../raw/current/scripts/recipes.lua#L934-L934)、[prefabs/blueprint.lua L62–73](../../../raw/current/scripts/prefabs/blueprint.lua#L62-L73)、[prefabs/blueprint.lua L237–244](../../../raw/current/scripts/prefabs/blueprint.lua#L237-L244)。
- 验收 索引写“40 条同名配方”，正文另列沃特路径和天体即时制作边界。分别判定贝壳蓝图、环形山名称键与真正地皮注册。

### R13 海芽满月用于预约，执行和读档只复核产量

- 位置 [waterplant-planter L22](../../../wiki/reference/prefabs/waterplant-planter.md#L22-L22)、[waterplant-planter L58–67](../../../wiki/reference/prefabs/waterplant-planter.md#L58-L67)、[waterplant-planter L130–138](../../../wiki/reference/prefabs/waterplant-planter.md#L130-L138)、[farming-and-planting-index L22](../../../wiki/reference/prefabs/farming-and-planting-index.md#L22-L22)。
- 证据 满月事件和长满时可以预约随机 2–4 秒任务。任务到点只查 harvestable 与产量，完成动画 16 帧后才换成海草。保存 `_grow_task` 时写 `grow_on_load`，加载后可不查当前月相直接重排。死亡态 animover 发插穗，即使此刻 burnable 正在燃烧仍会发，真正的 onburnt 燃尽链才只给熟藤壶和岩石。[prefabs/waterplant_baby.lua L40–107](../../../raw/current/scripts/prefabs/waterplant_baby.lua#L40-L107)、[prefabs/waterplant_baby.lua L141–158](../../../raw/current/scripts/prefabs/waterplant_baby.lua#L141-L158)、[prefabs/waterplant_baby.lua L251–256](../../../raw/current/scripts/prefabs/waterplant_baby.lua#L251-L256)、[stategraphs/SGwaterplant.lua L122–135](../../../raw/current/scripts/stategraphs/SGwaterplant.lua#L122-L135)、[prefabs/waterplant.lua L248–259](../../../raw/current/scripts/prefabs/waterplant.lua#L248-L259)、[components/burnable.lua L291–299](../../../raw/current/scripts/components/burnable.lua#L291-L299)。
- 验收 摘要和索引将“满月满产才长成”收窄到预约，并明确读档恢复。把“烧死不给”改为“走燃尽回调不产”，用死亡动画结算时仍在燃烧的分支验证。

### R14 插穗的候选菜单不是执行前提，目标回调返回也不会向上传递

- 位置 [waterplant-planter L38–54](../../../wiki/reference/prefabs/waterplant-planter.md#L38-L54)、[waterplant-planter L90–97](../../../wiki/reference/prefabs/waterplant-planter.md#L90-L97)。
- 证据 候选查询按三种标签插入动作。执行端实际检查对象及组件、材料类型、使用者标签和目标可升级，并不重新验证是否出过菜单。海蚀柱的局部 OnUpgraded 返回海芽，但 `Upgradeable:Upgrade` 丢弃回调返回值，自己返回 true。两个目标使用原始默认阈值 5，原版一件只累加 1，随即替换目标，不会在普通路径推进阶段。[componentactions.lua L1889–1897](../../../raw/current/scripts/componentactions.lua#L1889-L1897)、[actions.lua L4056–4068](../../../raw/current/scripts/actions.lua#L4056-L4068)、[components/upgrader.lua L21–29](../../../raw/current/scripts/components/upgrader.lua#L21-L29)、[components/upgradeable.lua L17–26](../../../raw/current/scripts/components/upgradeable.lua#L17-L26)、[components/upgradeable.lua L75–93](../../../raw/current/scripts/components/upgradeable.lua#L75-L93)、[prefabs/seastack.lua L61–80](../../../raw/current/scripts/prefabs/seastack.lua#L61-L80)。
- 验收 候选和脚本执行按独立两段记，局部回调返回与公开动作返回分别写。说明原版未到阶段阈值及自定义高升级值的后续分支。

### R15 天体珠宝满格还需要装备，月玻璃必须是充能形态

- 位置 [special-tree-planting L6](../../../wiki/reference/prefabs/special-tree-planting.md#L6-L6)、[special-tree-planting L110–112](../../../wiki/reference/prefabs/special-tree-planting.md#L110-L112)、[special-tree-planting L128](../../../wiki/reference/prefabs/special-tree-planting.md#L128-L128)、[farming-and-planting-index L18](../../../wiki/reference/prefabs/farming-and-planting-index.md#L18-L18)。
- 证据 满珠宝触发尝试开任务，启动时还要求帽子已装备。tick 要求拥有者有 petleash 且对应宠物栏未满，再在帽主 8 范围寻充能月玻璃，候选片周围 4 内凑 3 件未预约的充能月玻璃。到达处理会再取材料，投射物移除时确认未失败并重新寻料后才扣料并尝试生成。普通月玻璃 `moonglass` 不够资格。[prefabs/hats.lua L3178–3222](../../../raw/current/scripts/prefabs/hats.lua#L3178-L3222)、[prefabs/hats.lua L3223–3265](../../../raw/current/scripts/prefabs/hats.lua#L3223-L3265)、[prefabs/hats.lua L3266–3319](../../../raw/current/scripts/prefabs/hats.lua#L3266-L3319)、[prefabs/hats.lua L3345–3358](../../../raw/current/scripts/prefabs/hats.lua#L3345-L3358)、[prefabs/hats.lua L3375–3388](../../../raw/current/scripts/prefabs/hats.lua#L3375-L3388)、[tuning.lua L8797–8800](../../../raw/current/scripts/tuning.lua#L8797-L8800)。
- 验收 至少写清装备、petleash 容量、充能月玻璃与成功链。索引可以压成“戴满珠宝帽，宠物槽未满时用地面充能月玻璃”，避免检索摘要把任意月玻璃都说成材料。

### R16 本批统计把两个世界态列成物品

- 位置 [总台账 B14](prefab-inventory-ledger-20261108.md#L166-L166)、[执行表总状态](prefab-inventory-batches-20261108.md#L5-L5)、[执行表 B14 行](prefab-inventory-batches-20261108.md#L83-L83)、[PLAN L9](../../../PLAN.md#L9-L9)、[日志 B14 落地](../../../wiki/log.md#L79-L79)、[special-tree-planting L27–34](../../../wiki/reference/prefabs/special-tree-planting.md#L27-L34)。
- 证据 当前登记的 60 个主注册里，`livingtree_sapling` 与 `livingtree_halloween` 是世界态。活树根先生成苗，苗成长后在原位生成成树。8 页和 10 个放置器的口径可保留。执行表 B14 计划切片仍把 `lureplant_bulb`、`palmcone_scale` 写成候选，当前实际承接是 `lureplantbulb`、`palmcone_seed`，鳞片仅对照。[prefabs/livingtree_root.lua L41–48](../../../raw/current/scripts/prefabs/livingtree_root.lua#L41-L48)、[prefabs/livingtree_root_planted.lua L14–21](../../../raw/current/scripts/prefabs/livingtree_root_planted.lua#L14-L21)、[prefabs/livingtree_root_planted.lua L107](../../../raw/current/scripts/prefabs/livingtree_root_planted.lua#L107-L107)、[prefabs/livingtree_halloween.lua L386](../../../raw/current/scripts/prefabs/livingtree_halloween.lua#L386-L386)、[prefabs/lureplant_bulb.lua L66–67](../../../raw/current/scripts/prefabs/lureplant_bulb.lua#L66-L67)、[prefabs/pinecone.lua L135–139](../../../raw/current/scripts/prefabs/pinecone.lua#L135-L139)。
- 验收 主代理统一为 8 页、58 个物品主注册＋2 个世界态主注册＋10 个放置器。历史候选仍有需要时明确标注已更正的实际承接名。

### R17 树种来源与安抚概率省略实际门禁

- 位置 [tree-seeds L22](../../../wiki/reference/prefabs/tree-seeds.md#L22-L22)、[tree-seeds L81–82](../../../wiki/reference/prefabs/tree-seeds.md#L81-L82)、[tree-seeds L103–106](../../../wiki/reference/prefabs/tree-seeds.md#L103-L106)、[special-tree-planting L45–46](../../../wiki/reference/prefabs/special-tree-planting.md#L45-L46)。
- 证据 传送门 15 秒是重排计时，睡眠会暂停。实际掷普通权重池前要清理旧记录并且 `#_loot < 10`，超限只走 FX 备用路径。大理石灌木的 0.75／0.5 和安抚的 0.33／0.15 都是 `TryLuckRoll` 的基础输入。安抚近档距离严格小于 5，且抽中后还需 sleeper，1000 是传入睡眠接口的参数。[prefabs/monkeyisland_portal.lua L140–157](../../../raw/current/scripts/prefabs/monkeyisland_portal.lua#L140-L157)、[prefabs/monkeyisland_portal.lua L184–194](../../../raw/current/scripts/prefabs/monkeyisland_portal.lua#L184-L194)、[prefabs/monkeyisland_portal.lua L330–351](../../../raw/current/scripts/prefabs/monkeyisland_portal.lua#L330-L351)、[tuning.lua L6687–6695](../../../raw/current/scripts/tuning.lua#L6687-L6695)、[prefabs/marbleshrub.lua L39–54](../../../raw/current/scripts/prefabs/marbleshrub.lua#L39-L54)、[prefabs/pinecone.lua L19–37](../../../raw/current/scripts/prefabs/pinecone.lua#L19-L37)、[prefabs/livingtree_root.lua L51–68](../../../raw/current/scripts/prefabs/livingtree_root.lua#L51-L68)、[componentutil.lua L2237–2251](../../../raw/current/scripts/componentutil.lua#L2237-L2251)、[components/sleeper.lua L279–306](../../../raw/current/scripts/components/sleeper.lua#L279-L306)。
- 验收 区分每次计时尝试与实际抽权重，数值统一写成基础概率和睡眠参数。用满额传送门、距离正好 5、有运气修正的操作人三个情形核对口径。

## P3

### R18 尖刺灌木挖出件的 bank／build 回退到数据名

- 位置 [plantables-remaining L27–31](../../../wiki/reference/prefabs/plantables-remaining.md#L27-L31)、[plantables-remaining L46–53](../../../wiki/reference/prefabs/plantables-remaining.md#L46-L53)。
- 证据 数据项 `name="marsh_bush"` 且无覆写，bank／build 均取 `marsh_bush`，注册名才拼为 `dug_marsh_bush`。L49 表格写对，L31 说回落到注册名。[prefabs/plantables.lua L5–15](../../../raw/current/scripts/prefabs/plantables.lua#L5-L15)、[prefabs/plantables.lua L48–53](../../../raw/current/scripts/prefabs/plantables.lua#L48-L53)、[prefabs/plantables.lua L112–116](../../../raw/current/scripts/prefabs/plantables.lua#L112-L116)、[prefabs/plantables.lua L156–160](../../../raw/current/scripts/prefabs/plantables.lua#L156-L160)。
- 验收 保留表格，构造段统一为“回落到数据名 marsh_bush”。

## 修订与外部复验记录

| 编号 | 写作修订回执 | 外部复验 |
| --- | --- | --- |
| R01 | turfs L141-161：示例改为包装 `GLOBAL.Map.CanPlaceTurfAtPoint`（原 `AddComponentPostInit("map")` 永不执行），说明安装在原版方法之后、全局影响、四输入验收（DIRT／可种植陆地／硬地／水面）与两端证据。自验：示例 luaparser 通过；`AddComponentPostInit("map"` 仅出现在说明不生效句。 | 通过 |
| R02 | 主代理：叶索引 L17 改为「堆叠 40；三个 addcone 15 秒燃料／7.5 秒燃尽，大理石豌豆无 fuel／burnable」。 | 通过 |
| R03 | special L6／L136／L144／L150：改为每件 2 的上限；两件一叠烧毁原版 4／改后 8、燃料原版 2／改后 4；验收分合格树充足与不足两情形，只有真命中 TransformIntoLeif／StartMonster 才减额度。索引 L18 由主代理同步。 | 通过 |
| R04 | plant-spawners L6／L22／L86-L89／L145：周期改为有条件定时尝试（开关／库存／离屏排队／冻结／caveday）；补容量与生命（3 级 9、春季 floor(9×1.33)=11、生命 1000）与女王门禁（60／4 只决定旧巢是否保留）。索引 L19 同步。 | 通过 |
| R05 | fertilizer-materials L74：改为鸟笼 DigestFood 实际分支（普通鸟看 `Prefabs[<prefab>_seeds]`，无对应种子才以 0.33 基础概率＋喂食者运气抽鸟粪；`seeds` 是反例）。索引 L15 同步。 | 通过 |
| R06 | fertilizer-materials L6／L22／L52-56／L87-106／L112-116：补 quagmire 燃料回调例外、FX 缺失退回取用者坐标、嗡嗡肥料二阶制作／三阶部署各 +8 与基础 12／12／12；示例 +8 标为自定义值。索引 L15 同步。 | 通过 |
| R07 | treegrowthsolution L6／L20／L22／L45／L57／L93：改为四类用途／五个动作入口（含独立 `FERTILIZE`），DEPLOY 与 FERTILIZE 共用 `Fertilizer:OnApplied` 扣件。索引 L16 同步。 | 通过 |
| R08 | tree-seeds L96／L132／L133／L146：stopgrowing 只 StopTimer、熄灭后重抽整段；补完整自定义苗契约（StartGrowing＋Transform＋SoundEmitter）；旧 growtime 存档同样走改后的 `_spawn_prefab`。 | 通过 |
| R09 | tree-seeds L67／L69／L72 集中公共规则（IsDeployable 前置、竞技场禁建带、mouseover 仅预览阶段）；plant-spawners L58／L60／L62／L151 收窄为「可通行＋前置」；special L44 链接同一套规则。 | 通过 |
| R10 | plantables-remaining L42／L87：拆回调侧／动作侧／结果侧三层；sources 补 components/inventory.lua。berry-bushes L67 的同一断言由主代理同步。 | 通过 |
| R11 | turfs L22／L87／L112／L131／L164：区分落点失败与动作成功但地块未校验；挖起补 CanPlowAtPoint／CanTerraformAtPoint 与 `onterraformfn` 分支（farm_plow 不走 HandleDugGround）。 | 通过 |
| R12 | turfs L107／L108／L111／L120／L165：40 条改为同名配方；补 `wurt_turf_marsh` 与天体 nounlock；蓝图动态登记——`turf_shellbeach_blueprint` 是真实注册，`TURF_METEOR_BLUEPRINT` 只有名称键。 | 通过 |
| R13 | waterplant-planter L22／L62／L65／L67／L130／L133：满月满产只用于预约 2–4 秒任务；到点复核产量、16 帧换海草；读档 `grow_on_load` 不查月相；燃尽链不产但死亡态 animover 不看燃烧。 | 通过 |
| R14 | waterplant-planter L40／L42／L43／L45／L50／L54：候选与执行分两段；补 `Upgradeable:Upgrade` 丢弃回调返回值固定 true；默认阈值 5、原版一件 +1 随即替换不推进阶段。 | 通过 |
| R15 | special L6／L111／L112：满 5 格只是启动条件，还须帽子已装备；tick 再查 petleash 与宠物槽；材料为充能月玻璃（8 半径找片→4 半径凑 3 件→到达复找→移除确认后扣料）。索引 L18 同步。 | 通过 |
| R16 | 主代理：台账／执行表／PLAN／日志统一为「58 个物品注册＋2 个世界态主注册＋10 个放置器」；执行表补 B14 实际承接名 `lureplantbulb`／`palmcone_seed`，鳞片仅对照。 | 通过 |
| R17 | tree-seeds L79／L106：传送门 15 秒为重排计时（睡眠暂停）、`#_loot<10` 否则只走 FX；安抚近档 `dist²<5²`、抽中还需 sleeper、1000 是 GoToSleep 参数；概率统一为 TryLuckRoll 基础输入。special L45-46 同步。 | 通过 |
| R18 | plantables-remaining L27／L31：bank／build 统一为「回落到数据名 marsh_bush」，注册名只是加 `dug_` 前缀。 | 通过 |

修订后机械校验（主代理，2026-11-11）：8 主页面＋叶索引＋分类索引共 10 文件 validate_page --check-indexes 逐页 0 issue（导航另报 12 个非本批页）；validate_source_links --strict 9 文件 757 处引用 0 错 0 警；8 段 Lua 示例 parser_probe --syntax-only 全部通过；8 页基线已重录（4 页 refreshed、4 页 already_recorded）。未运行游戏。

## 验证范围

- 逐项核对 8 页的注册与端、动作和材料消费、代表性获取来源，以及农业叶索引 8 行。专项追到消费端的链包括肥料的部署和鸟笼消化、树果酱的补漏和旧施肥、树种的苗计时和安抚、雕像的预约和帽子 petleash、食人花的 freshspawn 和蜘蛛巢的外部计时、移栽的挖起和回种、地皮的两端放置和挖起、插穗的扣件和海芽预约。
- 额外抽查了木墙回血、树苗遗留存档、传送门上限、食人花冬季、蜘蛛女王阈值、地皮蓝图动态注册和海芽读档恢复。对完整资产、COTL 菜单活动门禁、引擎植物图鉴登记、蜡封件、全部掉落宿主及完整宠物生命周期只保留原页未覆盖口径。
- 独立重跑 8 页＋农业叶索引的严格源码链接检查，**714 处，0 错 0 警，0 跳过**。本轮是静态源码检视，未运行游戏。

## 第一轮外部定向复验

2026-11-11，实际 Steam manifest 与当前源码快照均为 **26474873**。按 R01–R18 定向回读改后正文、索引、直接关联页和调用方。**11 项通过，7 项部分修复，当前待修。** R01 仍有 P1，R03 的示例数值已通过，余下是复述口径。

### 仍待修订

- **R01（P1）地图方法包装仍早于定义，且还有蚁狮帽调用方。** [turfs L143–153](../../../wiki/reference/prefabs/turfs.md#L143-L153) 已换对对象，但在 `modmain.lua` 顶层立刻读取 `GLOBAL.Map.CanPlaceTurfAtPoint`。启动时 `ModManager:LoadMods()` 先执行 modmain；游戏加载世界 Prefab 时，才经 forest／cave → world → `require "components/map"` 定义该方法。冷启动时顶层包装会被后续原版定义覆盖，也不能假定捕获到的旧方法非空。安装应推迟至已证实的入口，例如在 `AddSimPostInit` 回调内取原方法并包装，该回调在 `PopulateWorld` 后由 `ModManager:SimPostInit` 调用。L145 的“只被两处判定调用”还遗漏蚁狮帽：`AutoTerraformer:DoTerraform` 同样读取此方法。放宽后可能让其提前进入扣槽内地皮、直接铺设的分支，跳过原先的挖起路径。验收检查冷启动、客端与服务端四种落点，并交代蚁狮帽的影响。[main.lua L389–410](../../../raw/current/scripts/main.lua#L389-L410)、[mods.lua L542–570](../../../raw/current/scripts/mods.lua#L542-L570)、[gamelogic.lua L297–302](../../../raw/current/scripts/gamelogic.lua#L297-L302)、[prefabs/forest.lua L1](../../../raw/current/scripts/prefabs/forest.lua#L1-L1)、[prefabs/world.lua L1–2](../../../raw/current/scripts/prefabs/world.lua#L1-L2)、[components/map.lua L162–175](../../../raw/current/scripts/components/map.lua#L162-L175)、[gamelogic.lua L929–959](../../../raw/current/scripts/gamelogic.lua#L929-L959)、[mods.lua L807–812](../../../raw/current/scripts/mods.lua#L807-L812)、[components/autoterraformer.lua L57–94](../../../raw/current/scripts/components/autoterraformer.lua#L57-L94)、[prefabs/hats.lua L2433–2437](../../../raw/current/scripts/prefabs/hats.lua#L2433-L2437)。
- **R03（余 P2）示例已给对 8／4，仍有结果与循环复述残留。** [special-tree-planting L136–144](../../../wiki/reference/prefabs/special-tree-planting.md#L136-L144) 的验收数值正确，[special-tree-planting L22](../../../wiki/reference/prefabs/special-tree-planting.md#L22-L22) 和 [farming-and-planting-index L18](../../../wiki/reference/prefabs/farming-and-planting-index.md#L18-L18) 仍把配额结果写成新树精，桦栗树消耗额度后走延迟怪物化。[special-tree-planting L144](../../../wiki/reference/prefabs/special-tree-planting.md#L144-L144) 把树不足归为 `break`，[special-tree-planting L150](../../../wiki/reference/prefabs/special-tree-planting.md#L150-L150) 也写“树不够就 break”。源码只有额度用完才 `break`，候选不足时遍历耗尽并返回剩余额度。摘要、快速判断、陷阱及索引统一写“最多预约 N 棵”，分别交代常青树与桦栗树。[prefabs/leif_idol.lua L35–38](../../../raw/current/scripts/prefabs/leif_idol.lua#L35-L38)、[prefabs/leif_idol.lua L60–83](../../../raw/current/scripts/prefabs/leif_idol.lua#L60-L83)、[prefabs/leif_idol.lua L95–124](../../../raw/current/scripts/prefabs/leif_idol.lua#L95-L124)。
- **R04（P2）船上绝对句还在，放员和再生门禁仍混在同一句。** 3 级容量 9／春季 11、生命 1000 及女王阈值的作用均已改对。[plant-spawners L86](../../../wiki/reference/prefabs/plant-spawners.md#L86-L86) 把库存、离屏、冻结和洞穴白天写成放员与补员的共用门禁。实际放员要有在巢库存并过清醒／离屏检查，开始放员时还把首个外部计时置零；再生另查 `regening`、普通／应急容量，空巢也可能补员，冻结和洞穴白天的相应回调只停放员。世界开关关掉两个外部计时，L744 的构造清零早于 L798 的 `SetStage`，后者会重填在巢库存，不能据早期赋值推断最终为空。[plant-spawners L154](../../../wiki/reference/prefabs/plant-spawners.md#L154-L154) 和 [farming-and-planting-index L19](../../../wiki/reference/prefabs/farming-and-planting-index.md#L19-L19) 仍称船上三只会逐只按周期放完。船上分支只保证停成长和再生，不额外停放员，仍受开关、冻结、睡眠等门禁，也可因事件放员或归巢。验收分开写首轮、后续定时放员、补员及平台分支。[prefabs/spiderden.lua L608–626](../../../raw/current/scripts/prefabs/spiderden.lua#L608-L626)、[prefabs/spiderden.lua L736–752](../../../raw/current/scripts/prefabs/spiderden.lua#L736-L752)、[prefabs/spiderden.lua L794–799](../../../raw/current/scripts/prefabs/spiderden.lua#L794-L799)、[prefabs/spiderden.lua L481–494](../../../raw/current/scripts/prefabs/spiderden.lua#L481-L494)、[prefabs/spiderden.lua L535–546](../../../raw/current/scripts/prefabs/spiderden.lua#L535-L546)、[worldsettingsutil.lua L34–65](../../../raw/current/scripts/worldsettingsutil.lua#L34-L65)、[components/childspawner.lua L184–193](../../../raw/current/scripts/components/childspawner.lua#L184-L193)、[components/childspawner.lua L262–274](../../../raw/current/scripts/components/childspawner.lua#L262-L274)、[components/childspawner.lua L315–321](../../../raw/current/scripts/components/childspawner.lua#L315-L321)、[components/childspawner.lua L667–684](../../../raw/current/scripts/components/childspawner.lua#L667-L684)。
- **R06（P2）索引的 quagmire 例外仍覆盖到四件。** [fertilizer-materials L56](../../../wiki/reference/prefabs/fertilizer-materials.md#L56-L56) 与 [fertilizer-materials L112](../../../wiki/reference/prefabs/fertilizer-materials.md#L112-L112) 已区分两种便便云落点和两个摘回调的注册，[fertilizer-materials L87–106](../../../wiki/reference/prefabs/fertilizer-materials.md#L87-L106) 也区分二阶制作和三阶部署各 +8。[farming-and-planting-index L15](../../../wiki/reference/prefabs/farming-and-planting-index.md#L15-L15) 仍只写“quagmire 摘除取用回调”，读起来适用于整行四件。补“鸟粪／粪肥”，FX 回退只写给堆肥／粪肥／嗡嗡肥料，鸟粪始终取取用者坐标。[prefabs/guano.lua L24–26](../../../raw/current/scripts/prefabs/guano.lua#L24-L26)、[prefabs/guano.lua L102–108](../../../raw/current/scripts/prefabs/guano.lua#L102-L108)、[prefabs/poop.lua L108–114](../../../raw/current/scripts/prefabs/poop.lua#L108-L114)、[prefabs/compost.lua L15–24](../../../raw/current/scripts/prefabs/compost.lua#L15-L24)、[prefabs/mosquitofertalizer.lua L29–41](../../../raw/current/scripts/prefabs/mosquitofertalizer.lua#L29-L41)。
- **R09（P2）DEFAULT 与 PLANT 的相对差异写成所有模式的绝对规则。** 竞技场禁建带、骑乘／浮标、两次服务端校验传 nil 均已写入。[tree-seeds L72](../../../wiki/reference/prefabs/tree-seeds.md#L72-L72) 仍写 `mouseover` “只对 DEPLOYMODE.DEFAULT 生效”，但 WATER 和 CUSTOM 等也接收该参数。将结论限定为“本批比较的 DEFAULT 与 PLANT 两条路径，DEFAULT 的鼠标目标筛选在预览传入实体时生效，PLANT 不用该参数，服务端常规部署复查传 nil”，与 [plant-spawners L54–62](../../../wiki/reference/prefabs/plant-spawners.md#L54-L62) 对齐。[components/deployable.lua L104–134](../../../raw/current/scripts/components/deployable.lua#L104-L134)、[components/inventoryitem_replica.lua L327–352](../../../raw/current/scripts/components/inventoryitem_replica.lua#L327-L352)、[components/map.lua L377–400](../../../raw/current/scripts/components/map.lua#L377-L400)、[actions.lua L1607–1613](../../../raw/current/scripts/actions.lua#L1607-L1613)。
- **R10（P2）回调失败与动作失败在导航和旧页仍可混读。** [plantables-remaining L42](../../../wiki/reference/prefabs/plantables-remaining.md#L42-L42) 和 [berry-bushes L67](../../../wiki/reference/prefabs/berry-bushes.md#L67-L67) 已写清先拆件、回调不销毁以及事件照发。[plantables-remaining L87](../../../wiki/reference/prefabs/plantables-remaining.md#L87-L87) 仍用“失败不返还”，没有限定为“回调没生成而动作仍成功”；真正的 `Deploy` 返回 false 会 `GiveItem`。[grass-saplings L47–49](../../../wiki/reference/prefabs/grass-saplings.md#L47-L49) 仍称空返回时“堆叠里的挖出件不会被消耗”，应限定到回调没有移除已取出的单体，并给出动作侧已先拆件。验收分别读回动作返回 false 与回调未生成两个例子。[prefabs/plantables.lua L17–36](../../../raw/current/scripts/prefabs/plantables.lua#L17-L36)、[actions.lua L1622–1637](../../../raw/current/scripts/actions.lua#L1622-L1637)、[components/deployable.lua L132–146](../../../raw/current/scripts/components/deployable.lua#L132-L146)。
- **R13（P2）海芽的满月监听也会在满月结束时预约。** 燃尽与死亡态分离、到点复核产量、读档不查月相均已改对。初检把满月监听写成“满月事件”，遗漏了 `WatchWorldState("isfullmoon", ...)` 会在 true→false 时也触发。通用 watcher 在值变更时传入新值，而 `on_full_moon(inst)` 不读取它，也不检查月相。若月落时海芽已满产且无 `_grow_task`，仍能预约。只有 `on_grow` 当次长满分支显式查满月。[waterplant-planter L22](../../../wiki/reference/prefabs/waterplant-planter.md#L22-L22)、[waterplant-planter L61–65](../../../wiki/reference/prefabs/waterplant-planter.md#L61-L65)、[waterplant-planter L133](../../../wiki/reference/prefabs/waterplant-planter.md#L133-L133) 和 [farming-and-planting-index L22](../../../wiki/reference/prefabs/farming-and-planting-index.md#L22-L22) 都要区分两种预约入口。验收用“满月结束时满产、无待执行任务”核对，并保留读档重排。[components/worldstate.lua L24–43](../../../raw/current/scripts/components/worldstate.lua#L24-L43)、[components/worldstate.lua L72–93](../../../raw/current/scripts/components/worldstate.lua#L72-L93)、[entityscript.lua L1290–1293](../../../raw/current/scripts/entityscript.lua#L1290-L1293)、[prefabs/waterplant_baby.lua L73–80](../../../raw/current/scripts/prefabs/waterplant_baby.lua#L73-L80)、[prefabs/waterplant_baby.lua L91–107](../../../raw/current/scripts/prefabs/waterplant_baby.lua#L91-L107)、[prefabs/waterplant_baby.lua L251–256](../../../raw/current/scripts/prefabs/waterplant_baby.lua#L251-L256)。

### 已通过

| 编号 | 当前页面与源码复验 |
| --- | --- |
| R02 | [farming-and-planting-index L17](../../../wiki/reference/prefabs/farming-and-planting-index.md#L17-L17) 已与 [tree-seeds L53–63](../../../wiki/reference/prefabs/tree-seeds.md#L53-L63) 一致，四件堆叠上限 40，三个 addcone 有 15 秒燃料和 7.5 秒燃尽，大理石豌豆无两组件。[prefabs/pinecone.lua L95–106](../../../raw/current/scripts/prefabs/pinecone.lua#L95-L106)、[prefabs/marblebean.lua L40–59](../../../raw/current/scripts/prefabs/marblebean.lua#L40-L59)。 |
| R05 | [fertilizer-materials L74](../../../wiki/reference/prefabs/fertilizer-materials.md#L74-L74) 和 [farming-and-planting-index L15](../../../wiki/reference/prefabs/farming-and-planting-index.md#L15-L15) 已把鸟粪限定到普通鸟的无对应种子分支，通用种子是反例，并保留 0.33 基础值和运气修正。[prefabs/birdcage.lua L101–145](../../../raw/current/scripts/prefabs/birdcage.lua#L101-L145)、[prefabs/birdcage.lua L155–180](../../../raw/current/scripts/prefabs/birdcage.lua#L155-L180)。 |
| R07 | [treegrowthsolution L6](../../../wiki/reference/prefabs/treegrowthsolution.md#L6-L6)、[treegrowthsolution L45–57](../../../wiki/reference/prefabs/treegrowthsolution.md#L45-L57)、[treegrowthsolution L93](../../../wiki/reference/prefabs/treegrowthsolution.md#L93-L93) 和 [farming-and-planting-index L16](../../../wiki/reference/prefabs/farming-and-planting-index.md#L16-L16) 已区分四类用途、五种动作入口。农田部署和旧施肥共享扣件，后者须 `applied`。[standardcomponents.lua L1797–1810](../../../raw/current/scripts/standardcomponents.lua#L1797-L1810)、[actions.lua L1812–1841](../../../raw/current/scripts/actions.lua#L1812-L1841)。 |
| R08 | [tree-seeds L96](../../../wiki/reference/prefabs/tree-seeds.md#L96-L96)、[tree-seeds L132–146](../../../wiki/reference/prefabs/tree-seeds.md#L132-L146) 已写明熄灭重抽计时、替换苗需要的三个接口，以及旧 `growtime` 也会读修改后的目标。[prefabs/planted_tree.lua L82–93](../../../raw/current/scripts/prefabs/planted_tree.lua#L82-L93)、[prefabs/pinecone.lua L3–9](../../../raw/current/scripts/prefabs/pinecone.lua#L3-L9)、[prefabs/pinecone.lua L40–44](../../../raw/current/scripts/prefabs/pinecone.lua#L40-L44)。 |
| R11 | [turfs L87](../../../wiki/reference/prefabs/turfs.md#L87-L87)、[turfs L112](../../../wiki/reference/prefabs/turfs.md#L112-L112)、[turfs L131](../../../wiki/reference/prefabs/turfs.md#L131-L131)、[turfs L164](../../../wiki/reference/prefabs/turfs.md#L164-L164) 与 [farming-and-planting-index L21](../../../wiki/reference/prefabs/farming-and-planting-index.md#L21-L21) 已区分 Lua 调用 SetTile 与实际改地，且保留可部署判定和扣件顺序。挖起分开叉子默认处理与农田犁自定义回调。[prefabs/turfs.lua L9–17](../../../raw/current/scripts/prefabs/turfs.lua#L9-L17)、[components/terraformer.lua L12–32](../../../raw/current/scripts/components/terraformer.lua#L12-L32)、[prefabs/farm_plow.lua L228–233](../../../raw/current/scripts/prefabs/farm_plow.lua#L228-L233)。 |
| R12 | [turfs L107–112](../../../wiki/reference/prefabs/turfs.md#L107-L112)、[turfs L165](../../../wiki/reference/prefabs/turfs.md#L165-L165) 与 [farming-and-planting-index L21](../../../wiki/reference/prefabs/farming-and-planting-index.md#L21-L21) 已区分 40 条同名配方、沃特别名、两个天体 nounlock 与蓝图的动态登记。贝壳蓝图有真实注册，环形山名称键没有对应具体蓝图。[recipes.lua L109–110](../../../raw/current/scripts/recipes.lua#L109-L110)、[recipes.lua L842–843](../../../raw/current/scripts/recipes.lua#L842-L843)、[recipes.lua L934](../../../raw/current/scripts/recipes.lua#L934-L934)、[prefabs/blueprint.lua L62–73](../../../raw/current/scripts/prefabs/blueprint.lua#L62-L73)、[prefabs/blueprint.lua L237–244](../../../raw/current/scripts/prefabs/blueprint.lua#L237-L244)。 |
| R14 | [waterplant-planter L40–54](../../../wiki/reference/prefabs/waterplant-planter.md#L40-L54) 已拆候选与执行，回调返回的海芽不会被 Upgrade 传给动作，原版材料每件 +1、低于阈值 5。[componentactions.lua L1889–1897](../../../raw/current/scripts/componentactions.lua#L1889-L1897)、[actions.lua L4056–4068](../../../raw/current/scripts/actions.lua#L4056-L4068)、[components/upgradeable.lua L17–26](../../../raw/current/scripts/components/upgradeable.lua#L17-L26)、[components/upgradeable.lua L75–93](../../../raw/current/scripts/components/upgradeable.lua#L75-L93)。 |
| R15 | [special-tree-planting L110–112](../../../wiki/reference/prefabs/special-tree-planting.md#L110-L112) 与 [farming-and-planting-index L18](../../../wiki/reference/prefabs/farming-and-planting-index.md#L18-L18) 已明确帽子需装备、对应宠物槽未满，并取地面充能月玻璃。正文保留 8／4 半径、3 件及到达／移除时复查。[prefabs/hats.lua L3178–3265](../../../raw/current/scripts/prefabs/hats.lua#L3178-L3265)、[prefabs/hats.lua L3286–3319](../../../raw/current/scripts/prefabs/hats.lua#L3286-L3319)、[prefabs/hats.lua L3345–3348](../../../raw/current/scripts/prefabs/hats.lua#L3345-L3348)。 |
| R16 | [总台账](prefab-inventory-ledger-20261108.md#L166-L166)、[执行表](prefab-inventory-batches-20261108.md#L5-L5)、[PLAN](../../../PLAN.md#L9-L9) 与[日志](../../../wiki/log.md#L79-L80)均已分记 58 个物品注册、2 个世界态主注册和 10 个放置器。[承接说明](prefab-inventory-batches-20261108.md#L91-L91)明确可种的是棕榈松果树芽及食人花种子的实际注册。 |
| R17 | [tree-seeds L83](../../../wiki/reference/prefabs/tree-seeds.md#L83-L83)、[tree-seeds L106–107](../../../wiki/reference/prefabs/tree-seeds.md#L106-L107) 和 [special-tree-planting L46](../../../wiki/reference/prefabs/special-tree-planting.md#L46-L46) 已区分传送门重排与实际抽取、近档严格小于 5、运气基础值和睡眠接口参数。[prefabs/monkeyisland_portal.lua L184–194](../../../raw/current/scripts/prefabs/monkeyisland_portal.lua#L184-L194)、[prefabs/monkeyisland_portal.lua L330–351](../../../raw/current/scripts/prefabs/monkeyisland_portal.lua#L330-L351)、[prefabs/pinecone.lua L19–37](../../../raw/current/scripts/prefabs/pinecone.lua#L19-L37)、[prefabs/marbleshrub.lua L39–54](../../../raw/current/scripts/prefabs/marbleshrub.lua#L39-L54)。 |
| R18 | [plantables-remaining L27–31](../../../wiki/reference/prefabs/plantables-remaining.md#L27-L31) 已和差异表一致，尖刺灌木挖出件的 bank／build 取数据名，注册另拼 `dug_`。[prefabs/plantables.lua L48–53](../../../raw/current/scripts/prefabs/plantables.lua#L48-L53)、[prefabs/plantables.lua L156–160](../../../raw/current/scripts/prefabs/plantables.lua#L156-L160)。 |

复跑 8 主页面、农业叶索引及改动的旧页 berry-bushes，共 10 文件严格源码链接 **856 处，0 错、0 警、0 跳过**。本轮复用首轮同版证据并追查修订涉及的调用链，未运行游戏。

## 第二轮修订回执（2026-11-11）

针对第一轮复验仍待修的 R01、R03、R04、R06、R09、R10、R13，按原编号再修订。

| 编号 | 修订位置与自验 |
|---|---|
| R01 | turfs L141-162：示例改为在 AddSimPostInit 回调内安装（modmain 顶层读取会被后续原版定义覆盖、可能捕空）；补蚁狮帽 AutoTerraformer:DoTerraform 调用方与「扣槽内地皮直接铺设、跳过挖起」的影响；验收拆冷启动、服务端、客端、四种落点＋蚁狮帽。示例 9 行 luaparser 通过。 |
| R03 | special L6／L22／L99／L136／L144／L150：改为「最多预约 N 棵」，常青树变树精、桦栗树延迟怪物化；候选不足只耗尽遍历并返回剩余额度，break 只在额度归零时发生。索引 L18 由主代理同步。 |
| R04 | plant-spawners L6／L22／L86／L157：拆放员／补员／世界开关三条；首轮计时置零；空巢也会补；冻结与 iscaveday 只停放员；船上只停成长与再生、仍受开关／冻结／睡眠／事件影响。索引 L19 同步。 |
| R06 | 索引 L15：quagmire 例外限定鸟粪／粪肥；FX 回退限定堆肥／粪肥／嗡嗡肥料，鸟粪始终取取用者坐标（页面 L56／L112 已正确，主代理只改索引）。 |
| R09 | tree-seeds L72、plant-spawners L58-60：mouseover 改为「本批 DEFAULT 与 PLANT 的相对差异」，注明 WATER／CUSTOM 也接收、仅预览阶段可得、服务端传 nil。 |
| R10 | plantables-remaining L87：改为「回调未生成时动作仍成功、不返还已拆出的一件；只有 Deploy 返回 false 才 GiveItem」。berry-bushes L67 已同步；grass-saplings 未发现该断言。 |
| R13 | waterplant-planter L22／L62／L65／L133：区分钟摆监听的两种预约入口（isfullmoon 任一变化含月落，只要满产且无待执行任务；on_grow 长满分支显式查满月）；执行与读档仍不看月相。索引 L22 同步。 |

机械校验（主代理，2026-11-11）：10 文件 validate_page --check-indexes 逐页 0 issue（导航另报 12 个非本批页）；validate_source_links --strict 9 文件 764 处引用 0 错 0 警；改动示例 parser_probe --syntax-only 全部通过；未运行游戏。

## 第二轮外部定向复验

2026-11-11，实际安装与源码快照同为 Steam build **26474873**。针对上一轮剩余七项复验，R03、R06、R09 通过，R01、R04、R10、R13 部分修复。累计 **14 项通过、4 项部分修复，待修订**。

### 仍待修订

- **R01（P1）主机延后安装已修，远程加入的客户端未被当前示例覆盖。** [turfs L143–160](../../../wiki/reference/prefabs/turfs.md#L143-L160) 已在回调内捕获原方法并写清蚁狮帽分支，但 L144 用调用点无端判断推断两端都会进 `SimPostInit`。该调用实际位于本地生成或读档进入的 `DoInitGame`；远程加入的 `JOIN_SERVER` 走 `TheNet:StartClient`，客端世界的 postinit 由 `world_network` 在 `not TheWorld.ismastersim` 时另行触发，Lua 未在这条路径调用 `ModManager:SimPostInit`。上轮提出的 `AddSimPostInit` 只解决了安装时序，遗漏了远程客户端入口。保留两端功能时，可用世界 Prefab postinit 安装一次，例如在 `AddPrefabPostInitAny` 中筛 `inst == GLOBAL.TheWorld`。Lua 在 prefab 工厂返回后调用该回调，世界对象此时已通过 `AddMap`，这段不依赖主模拟。验收分开宿主／专服与远程加入的客户端。蚁狮帽例子用槽内地皮与目标格不同且原版拒绝、包装放行的落点，避开同地皮提前返回。[gamelogic.lua L950–959](../../../raw/current/scripts/gamelogic.lua#L950-L959)、[gamelogic.lua L1035–1068](../../../raw/current/scripts/gamelogic.lua#L1035-L1068)、[gamelogic.lua L1197–1202](../../../raw/current/scripts/gamelogic.lua#L1197-L1202)、[prefabs/world_network.lua L19–33](../../../raw/current/scripts/prefabs/world_network.lua#L19-L33)、[prefabs/world_network.lua L67–72](../../../raw/current/scripts/prefabs/world_network.lua#L67-L72)、[prefabs/world.lua L420–452](../../../raw/current/scripts/prefabs/world.lua#L420-L452)、[prefabs/world.lua L565–570](../../../raw/current/scripts/prefabs/world.lua#L565-L570)、[mainfunctions.lua L387–411](../../../raw/current/scripts/mainfunctions.lua#L387-L411)、[modutil.lua L575–583](../../../raw/current/scripts/modutil.lua#L575-L583)、[components/autoterraformer.lua L66–77](../../../raw/current/scripts/components/autoterraformer.lua#L66-L77)。
- **R04（P2）蜘蛛巢正文通过，索引还把首轮与后续间隔合写。** [plant-spawners L86–93](../../../wiki/reference/prefabs/plant-spawners.md#L86-L93) 已拆分放员、补员与世界开关，并说明首轮置零，[plant-spawners L157](../../../wiki/reference/prefabs/plant-spawners.md#L157-L157) 已保留平台放员门禁。[farming-and-planting-index L19](../../../wiki/reference/prefabs/farming-and-planting-index.md#L19-L19) 仍从“巢内满 3 只”直接接“5±0.5 秒放 1 只”，括号写定时尝试也没有说明首轮置零。将索引压成“开跑时首轮置零，后续 5±0.5 秒尝试放员；90±9 秒尝试补员”，船上门禁沿用现有写法。[components/childspawner.lua L262–269](../../../raw/current/scripts/components/childspawner.lua#L262-L269)、[worldsettingsutil.lua L34–59](../../../raw/current/scripts/worldsettingsutil.lua#L34-L59)、[prefabs/spiderden.lua L608–626](../../../raw/current/scripts/prefabs/spiderden.lua#L608-L626)。
- **R10（P2）移栽页导航已修，草苗旧页仍有原句。** [plantables-remaining L87](../../../wiki/reference/prefabs/plantables-remaining.md#L87-L87) 已明确回调未生成仍算动作成功、已拆出的物品不会放回，[berry-bushes L67](../../../wiki/reference/prefabs/berry-bushes.md#L67-L67) 一致。回执称草苗页未发现该断言，但 [grass-saplings L47–49](../../../wiki/reference/prefabs/grass-saplings.md#L47-L49) 的 L47 仍写“堆叠里的挖出件不会被消耗”。改成“回调不销毁已取出的单体”，补玩家 DEPLOY 此前已拆件、仅 `Deploy` 返回 false 才还回。[prefabs/plantables.lua L17–36](../../../raw/current/scripts/prefabs/plantables.lua#L17-L36)、[actions.lua L1622–1637](../../../raw/current/scripts/actions.lua#L1622-L1637)、[components/deployable.lua L132–146](../../../raw/current/scripts/components/deployable.lua#L132-L146)。
- **R13（P2）月落入口已修，索引仍缺满产和无待执行任务。** [waterplant-planter L133](../../../wiki/reference/prefabs/waterplant-planter.md#L133-L133) 已把月落的两项前置都写清。[farming-and-planting-index L22](../../../wiki/reference/prefabs/farming-and-planting-index.md#L22-L22) 仍称“满月事件与满月结束都会触发预约”，读起来任意海芽到月落都会排任务；[waterplant-planter L22](../../../wiki/reference/prefabs/waterplant-planter.md#L22-L22)、[waterplant-planter L61–65](../../../wiki/reference/prefabs/waterplant-planter.md#L61-L65) 也有“都会排”或缺少待执行任务的限制。可统一成“满月状态变化时，已满产且无待执行任务才预约；满月时长满也可预约，同样防重；执行与读档重排不查月相”。验收用未满产、已预约和满产未预约三个输入。[components/worldstate.lua L24–43](../../../raw/current/scripts/components/worldstate.lua#L24-L43)、[prefabs/waterplant_baby.lua L73–80](../../../raw/current/scripts/prefabs/waterplant_baby.lua#L73-L80)、[prefabs/waterplant_baby.lua L91–107](../../../raw/current/scripts/prefabs/waterplant_baby.lua#L91-L107)、[prefabs/waterplant_baby.lua L141–156](../../../raw/current/scripts/prefabs/waterplant_baby.lua#L141-L156)。

### 本轮通过

| 编号 | 当前页面与源码复验 |
| --- | --- |
| R03 | [special-tree-planting L22](../../../wiki/reference/prefabs/special-tree-planting.md#L22-L22)、[special-tree-planting L99–102](../../../wiki/reference/prefabs/special-tree-planting.md#L99-L102)、[special-tree-planting L136–150](../../../wiki/reference/prefabs/special-tree-planting.md#L136-L150) 和 [farming-and-planting-index L18](../../../wiki/reference/prefabs/farming-and-planting-index.md#L18-L18) 已统一为预约额度，区分常青树与桦栗树。只有额度归零才提前跳出，候选不足自然遍历完。原版 4／2 与改值 8／4 的验收仍成立。[prefabs/leif_idol.lua L35–38](../../../raw/current/scripts/prefabs/leif_idol.lua#L35-L38)、[prefabs/leif_idol.lua L60–83](../../../raw/current/scripts/prefabs/leif_idol.lua#L60-L83)、[prefabs/leif_idol.lua L95–124](../../../raw/current/scripts/prefabs/leif_idol.lua#L95-L124)。 |
| R06 | [farming-and-planting-index L15](../../../wiki/reference/prefabs/farming-and-planting-index.md#L15-L15) 已把 quagmire 例外限于鸟粪和粪肥，FX 回退限于堆肥、粪肥、嗡嗡肥料，并明确鸟粪直接取取用者坐标，与 [fertilizer-materials L56](../../../wiki/reference/prefabs/fertilizer-materials.md#L56-L56) 一致。[prefabs/guano.lua L24–26](../../../raw/current/scripts/prefabs/guano.lua#L24-L26)、[prefabs/guano.lua L102–108](../../../raw/current/scripts/prefabs/guano.lua#L102-L108)、[prefabs/poop.lua L108–114](../../../raw/current/scripts/prefabs/poop.lua#L108-L114)、[prefabs/compost.lua L15–24](../../../raw/current/scripts/prefabs/compost.lua#L15-L24)、[prefabs/mosquitofertalizer.lua L29–41](../../../raw/current/scripts/prefabs/mosquitofertalizer.lua#L29-L41)。 |
| R09 | [tree-seeds L67–73](../../../wiki/reference/prefabs/tree-seeds.md#L67-L73) 与 [plant-spawners L54–62](../../../wiki/reference/prefabs/plant-spawners.md#L54-L62) 已把 DEFAULT 和 PLANT 限定为本批的对比，明确 WATER／CUSTOM 也使用 `mouseover`。原版候选及服务端常规部署复查传 nil，客户端 placer 传真实鼠标实体。[componentactions.lua L2084–2092](../../../raw/current/scripts/componentactions.lua#L2084-L2092)、[components/deployable.lua L104–134](../../../raw/current/scripts/components/deployable.lua#L104-L134)、[components/playercontroller.lua L2906–2911](../../../raw/current/scripts/components/playercontroller.lua#L2906-L2911)。 |

独立复跑 8 个主页面、农业叶索引与直接关联的 grass-saplings、berry-bushes 共 11 文件，严格源码链接 **977 处，0 错、0 警、0 跳过**。本轮为静态定向复验，未运行游戏。

## 第三轮修订回执（2026-11-11）

针对第二轮复验仍待修的 R01、R04、R10、R13：

| 编号 | 修订位置与自验 |
|---|---|
| R01 | turfs L143-162：安装点改为 AddPrefabPostInitAny 筛 inst == GLOBAL.TheWorld（工厂返回后 TheWorld 已赋值并 AddMap，不依赖主模拟）；删「SimPostInit 两端都会执行」推断，分开宿主／专服与远程加入客端两条路径；蚁狮帽落点补「槽内地皮与目标格当前地块不同」前置；示例加 _wiki_turf_relaxed 防世界重建重复包层。示例 14 行 parser_probe 通过。 |
| R04 | 索引 L19 改为「开跑时首轮置零，后续 5±0.5 秒尝试放员；90±9 秒尝试补员（3 级容量 9、春季 11）；船上只停成长与再生」。 |
| R10 | grass-saplings L47 改为「回调不销毁已取出单体；玩家 DEPLOY 已先拆件，只有 Deploy 返回 false 才 GiveItem」，updated 改 2026-11-11；berry-bushes L67 已同步。 |
| R13 | waterplant L22／L65／L133 与索引 L22 统一为「满月状态变化时，已满产且无待执行任务才预约；满月时长满也可预约、同样防重；执行与读档重排不查月相」。 |

机械校验（主代理，2026-11-11）：12 文件 validate_page --check-indexes 逐页 0 issue（导航另报 12 个非本批页）；validate_source_links --strict 981 处引用 0 错 0 警；改动示例 parser_probe 通过；未运行游戏。

## 第三轮外部定向复验

2026-11-11，实际安装和源码快照同为 Steam build **26474873**。复验上一轮剩余四项，R04、R10 通过，R01 的两端安装和 R13 的预约正文、索引已通过，各余一处复述。累计 **16 项通过、2 项部分修复（均余 P2）**。暂无 P1。

### 仍待同步

- **R01（P2）示例通过，前提仍把读者指向旧入口。** [turfs L144–162](../../../wiki/reference/prefabs/turfs.md#L144-L162) 已改用 `AddPrefabPostInitAny`，仅在 `inst == GLOBAL.TheWorld` 时包装一次。原版世界工厂在客端早退之前已经赋值 `TheWorld` 并 AddMap，`SpawnPrefabFromSim` 在工厂返回后执行 postinit-any，没有主模拟判断。远程客端因此可随世界 Prefab 安装。蚁狮帽验收也已区分同地皮的提前返回。唯独 [turfs L143](../../../wiki/reference/prefabs/turfs.md#L143-L143) 末句仍是“改在 `AddSimPostInit` 里安装”，与下一段、代码和远程验收直接冲突。把该句换为“改在世界 Prefab 的 postinit 中安装（用 `AddPrefabPostInitAny` 筛出 `TheWorld`）”；同段的 `SpawnPrefab` 包装可明确为 `SpawnPrefabFromSim`。[prefabs/world.lua L421–452](../../../raw/current/scripts/prefabs/world.lua#L421-L452)、[prefabs/world.lua L565–570](../../../raw/current/scripts/prefabs/world.lua#L565-L570)、[mainfunctions.lua L373–411](../../../raw/current/scripts/mainfunctions.lua#L373-L411)、[gamelogic.lua L1197–1202](../../../raw/current/scripts/gamelogic.lua#L1197-L1202)、[components/autoterraformer.lua L66–77](../../../raw/current/scripts/components/autoterraformer.lua#L66-L77)。
- **R13（P2）正文与索引通过，流程图仍漏防重。** [waterplant-planter L22](../../../wiki/reference/prefabs/waterplant-planter.md#L22-L22)、[waterplant-planter L65](../../../wiki/reference/prefabs/waterplant-planter.md#L65-L65)、[waterplant-planter L133](../../../wiki/reference/prefabs/waterplant-planter.md#L133-L133) 和 [farming-and-planting-index L22](../../../wiki/reference/prefabs/farming-and-planting-index.md#L22-L22) 都已写明满月状态变化及满月时长满两路都受 `_grow_task == nil` 限制。[waterplant-planter L61](../../../wiki/reference/prefabs/waterplant-planter.md#L61-L61) 仍从两个触发条件直接画到预约，遗漏已有任务的反例。把图中箭头改成“满月状态变化时已满产，或满月时长满，且无待执行任务 → 预约”。[prefabs/waterplant_baby.lua L73–80](../../../raw/current/scripts/prefabs/waterplant_baby.lua#L73-L80)、[prefabs/waterplant_baby.lua L102–107](../../../raw/current/scripts/prefabs/waterplant_baby.lua#L102-L107)。

### 本轮通过

| 编号 | 当前页面与源码复验 |
| --- | --- |
| R04 | [farming-and-planting-index L19](../../../wiki/reference/prefabs/farming-and-planting-index.md#L19-L19) 已写清开跑首轮置零、后续 5±0.5 秒尝试放员、90±9 秒尝试补员，船上停成长和再生且放员仍受门禁。与 [plant-spawners L86–93](../../../wiki/reference/prefabs/plant-spawners.md#L86-L93)、[plant-spawners L157](../../../wiki/reference/prefabs/plant-spawners.md#L157-L157) 一致。[components/childspawner.lua L262–269](../../../raw/current/scripts/components/childspawner.lua#L262-L269)、[worldsettingsutil.lua L34–59](../../../raw/current/scripts/worldsettingsutil.lua#L34-L59)、[prefabs/spiderden.lua L608–626](../../../raw/current/scripts/prefabs/spiderden.lua#L608-L626)。 |
| R10 | [grass-saplings L47–49](../../../wiki/reference/prefabs/grass-saplings.md#L47-L49) 已补空生成时回调不销毁已取出的单体，玩家 DEPLOY 已先拆件，只有 Deploy 返回 false 才退回。[plantables-remaining L42](../../../wiki/reference/prefabs/plantables-remaining.md#L42-L42)、[plantables-remaining L87](../../../wiki/reference/prefabs/plantables-remaining.md#L87-L87) 和 [berry-bushes L67](../../../wiki/reference/prefabs/berry-bushes.md#L67-L67) 同口径。[prefabs/plantables.lua L17–36](../../../raw/current/scripts/prefabs/plantables.lua#L17-L36)、[actions.lua L1622–1637](../../../raw/current/scripts/actions.lua#L1622-L1637)、[components/deployable.lua L132–146](../../../raw/current/scripts/components/deployable.lua#L132-L146)。 |

独立复跑 8 个主页面、农业叶索引及直接关联的草苗、浆果丛两页，11 文件严格源码链接 **981 处，0 错、0 警、0 跳过**。本轮是静态定向复验，未运行游戏。

## 第四轮修订回执（2026-11-11）

第三轮复验后剩两处 P2（均为表述统一）：

| 编号 | 修订位置与自验 |
|---|---|
| R01 | turfs L143 前提末句由「改在 AddSimPostInit 里安装」改为「改在世界 prefab 的 postinit（AddPrefabPostInitAny 筛 TheWorld）里安装，见下一条」，与 L144 及示例统一。 |
| R13 | waterplant L61 流程图为两条预约入口补「且无待执行任务」：满月状态变化时满产且无待执行任务，或满月时长满且无待执行任务，才预约。 |

机械校验（主代理，2026-11-11）：turfs 108 处、waterplant 90 处严格源码引用，0 错 0 警；validate_page 均 passed；未运行游戏。

## 第四轮外部定向复验

2026-11-11，实际安装与当前源码快照均为 Steam build **26474873**。最后两项 R01、R13 通过，累计 **R01–R18 全部通过，已登记收口**。

| 编号 | 复验结果 |
| --- | --- |
| R01 | [地皮页 L143–162](../../../wiki/reference/prefabs/turfs.md#L143-L162) 的前提和示例均指向世界 Prefab postinit，筛 `TheWorld` 后只包装一次。原版工厂在客端早退前已设世界并 AddMap，postinit-any 在工厂返回后执行。原版本地初始化的 SimPostInit 保留为路径对照。证据见 [世界构造](../../../raw/current/scripts/prefabs/world.lua#L421-L452)、[客端返回](../../../raw/current/scripts/prefabs/world.lua#L565-L570)、[工厂与 postinit-any](../../../raw/current/scripts/mainfunctions.lua#L387-L411)、[远程加入](../../../raw/current/scripts/gamelogic.lua#L1197-L1202)。 |
| R13 | [海芽流程图 L61](../../../wiki/reference/prefabs/waterplant-planter.md#L61-L61) 已为两条预约入口都补上无待执行任务，与[正文](../../../wiki/reference/prefabs/waterplant-planter.md#L65-L65)及[索引](../../../wiki/reference/prefabs/farming-and-planting-index.md#L22-L22)一致。未满产或已有任务不会再预约。监听和长满分支的守卫分别见 [waterplant_baby.lua L73–80](../../../raw/current/scripts/prefabs/waterplant_baby.lua#L73-L80) 和 [L102–107](../../../raw/current/scripts/prefabs/waterplant_baby.lua#L102-L107)。 |

独立复跑 8 个主页面、农业叶索引及直接关联的草苗、浆果丛两页。11 文件严格源码链接 **981 处，0 错、0 警、0 跳过**。8 个主页面 **224 项源码哈希**与基线一致。未运行游戏。
