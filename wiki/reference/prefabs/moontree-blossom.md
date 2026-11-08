---
title: "月树花（moon_tree_blossom、moon_tree_blossom_worldgen）"
created: "2026-11-09"
updated: "2026-11-09"
type: prefab
summary: "prefabs/moontree_blossom.lua 的两个注册：可食、可干制、可插瓶的月树花物品（1 点生命、6 天腐坏、叠 40、DRY_FAST 干成干月树花），以及月岛地面摆放用、改名为同 prefab 且不腐坏的 worldgen 变体；连同月树三档砍伐掉落、温蒂祭坛、浴盐／月亮药水／巴甫洛娃／隐士茶等消费端。"
aliases: ["moon_tree_blossom", "moon_tree_blossom_worldgen", "月树花", "干月树花", "月树花瓣", "祭坛花朵", "月树花干制"]
tags: [prefab, item, food]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/moontree_blossom.lua", "prefabs/moontree.lua", "prefabs/driedplants.lua", "prefabs/driedplants_defs.lua", "prefabs/sisturn.lua", "prefabs/pigking.lua", "components/dryable.lua", "components/dryingrack.lua", "components/vasedecoration.lua", "components/perishable.lua", "components/edible.lua", "components/stackable.lua", "components/tradable.lua", "components/lootdropper.lua", "prefabs/hermitcrabtea_defs.lua", "componentactions.lua", "containers.lua", "standardcomponents.lua", "recipes.lua", "cooking.lua", "mainfunctions.lua", "map/rooms/forest/terrain_moonisland.lua", "map/static_layouts/moontrees_2.lua", "tuning.lua", "languages/chinese_s.po", "screens/redux/scrapbookdata.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 月树花（moon_tree_blossom、moon_tree_blossom_worldgen）

## 快速判断

- 职责与适用：月树的附带产出物，同一份工厂注册两次——常规物品版**构造即计时**（掉落、丢地、手工生成都一样），月岛地面摆放用的 `moon_tree_blossom_worldgen` 在构造后立刻暂停鲜度、并把 `prefab` 改名成 `moon_tree_blossom`，所以玩法判定与普通版完全一致。[工厂](../../../raw/current/scripts/prefabs/moontree_blossom.lua#L17-L79)、[worldgen 版](../../../raw/current/scripts/prefabs/moontree_blossom.lua#L81-L92)
- 优先使用：查数值与腐坏看[数值与计时](#数值与计时)；查从哪来看[来源：月树掉落与月岛摆放](#来源月树掉落与月岛摆放)；查能干什么看[消费端](#消费端)。
- 重要副作用：常规版**构造即 `StartPerishing`**，被丢在地上也会烂（6 天）；只有 worldgen 版是"摆着不烂、拾起才计时"。干制产物是独立的 `moon_tree_blossom_dried`（理智 +10），不是把本体改名。[鲜度](../../../raw/current/scripts/prefabs/moontree_blossom.lua#L62-L65)、[干制接线](../../../raw/current/scripts/prefabs/moontree_blossom.lua#L67-L71)
- 修改边界：本页写到"物品侧"为止——月树本体、砍伐与三张 loot 表归 [[reference/prefabs/moontree]]；祭坛收花与 BLOSSOM 判定的完整机制归 [[reference/prefabs/petals]] 与 [[reference/components/sisturnregistry]]（本节只写月树花这一格）；浴盐配方归 [[reference/components/bathbomb]]。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `moon_tree_blossom` | `fn` 直构 → Transform／AnimState／Network → `MakeInventoryPhysics`（可当战利品落地）→ bank／build `moon_tree_petal`、播 `idle`、`SetRayTestOnBB` → 三个标签 → `MakeInventoryFloatable` → `SetPristine` → 非主模拟返回 → 主模拟挂 inspectable／inventoryitem／tradable／vasedecoration／stackable／edible／perishable／dryable → 可燃＋可蔓延 → `MakeHauntableLaunch` | 物品版：鲜度从构造起走表 | [主体](../../../raw/current/scripts/prefabs/moontree_blossom.lua#L17-L79)、[注册](../../../raw/current/scripts/prefabs/moontree_blossom.lua#L94-L94) |
| `moon_tree_blossom_worldgen` | `ground_fn` 先调 `fn()` 复用全部构造，再 `SetPrefabName("moon_tree_blossom")`，最后在主模拟 `StopPerishing`；注册时不带 `prefabs` 依赖表 | 只为世界摆放：外观、可食、可干制与普通版一致，唯一差别是"不腐坏，拾起后才开始计时" | [ground_fn](../../../raw/current/scripts/prefabs/moontree_blossom.lua#L81-L92)、[注册](../../../raw/current/scripts/prefabs/moontree_blossom.lua#L95-L95) |

固定细节（两注册共用）：`pickupsound = "vegetation_grassy"`；标签 `cattoy`／`vasedecoration`／`dryable` 都在 `SetPristine` **之前**加（`dryable` 由组件构造时也会补一次）；可燃用小体型 `MakeSmallBurnable`，蔓延用 `MakeSmallPropagator`，作祟只抛飞不点燃。[标签与音效](../../../raw/current/scripts/prefabs/moontree_blossom.lua#L31-L38)、[可燃与作祟](../../../raw/current/scripts/prefabs/moontree_blossom.lua#L73-L76)、[dryable 标签](../../../raw/current/scripts/components/dryable.lua#L9-L13)

### 名称核验

`moon_tree_blossom`＝**月树花**（Lune Tree Blossom）、`moon_tree_blossom_dried`＝**干月树花**（Dried Lune Tree Blossom）；`moon_tree_blossom_worldgen` **没有** `NAMES` 键，靠 `SetPrefabName("moon_tree_blossom")` 取到"月树花"，所以名字层面看不出它是世界摆放版。[月树花](../../../raw/current/scripts/languages/chinese_s.po#L345875-L345877)、[干月树花](../../../raw/current/scripts/languages/chinese_s.po#L345880-L345882)、[改名](../../../raw/current/scripts/prefabs/moontree_blossom.lua#L84-L84)

## 数值与计时

时间基准：`seg_time`＝30 秒、`total_day_time`＝480 秒、`perish_warp`＝1（S1）。[基准](../../../raw/current/scripts/tuning.lua#L19-L20)、[`perish_warp`](../../../raw/current/scripts/tuning.lua#L47-L47)

| 项目 | 值 | 证据 |
|---|---|---|
| 三维（饥饿／生命／理智） | 0／1／0，`FOODTYPE.VEGGIE`；生命值来自 `HEALING_TINY`＝1，理智未写（默认 0） | [edible](../../../raw/current/scripts/prefabs/moontree_blossom.lua#L57-L60)、[常量](../../../raw/current/scripts/tuning.lua#L2021-L2021)、[采集册](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L943-L943) |
| 鲜度 | `PERISH_FAST`＝6 天＝2880 秒；腐坏替换 `spoiled_food` | [赋值](../../../raw/current/scripts/prefabs/moontree_blossom.lua#L62-L65)、[档位](../../../raw/current/scripts/tuning.lua#L2046-L2046) |
| 计时起点 | 常规版：`SetPerishTime` 后立刻 `StartPerishing`；worldgen 版：随后 `StopPerishing`，拾起（`OnPutInInventory`）才重启 | [常规版](../../../raw/current/scripts/prefabs/moontree_blossom.lua#L62-L65)、[拾起回调](../../../raw/current/scripts/prefabs/moontree_blossom.lua#L13-L15)、[worldgen 版](../../../raw/current/scripts/prefabs/moontree_blossom.lua#L90-L90) |
| 堆叠 | `STACK_SIZE_SMALLITEM`＝40 | [赋值](../../../raw/current/scripts/prefabs/moontree_blossom.lua#L54-L55)、[常量](../../../raw/current/scripts/tuning.lua#L83-L83) |
| 干制 | 产物 `moon_tree_blossom_dried`、`DRY_FAST`＝1 天＝480 秒；**架面与干后架面**都用 `meat_rack_food_petals`（干品本体仍是 `moon_tree_petal`） | [dryable](../../../raw/current/scripts/prefabs/moontree_blossom.lua#L67-L71)、[常量](../../../raw/current/scripts/tuning.lua#L2055-L2055) |
| 交易 | `tradable` 未赋值 → `goldvalue` 默认 0，进不了猪王收物门槛 | [组件默认](../../../raw/current/scripts/components/tradable.lua#L1-L7)、[猪王判定](../../../raw/current/scripts/prefabs/pigking.lua#L598-L606) |
| 燃料 | 生花**没有** `fuel` 组件（只可燃）；干花才有 `SMALL_FUEL`＝15 | [可燃](../../../raw/current/scripts/prefabs/moontree_blossom.lua#L73-L74)、[干花的燃料](../../../raw/current/scripts/prefabs/driedplants.lua#L63-L64)、[常量](../../../raw/current/scripts/tuning.lua#L1674-L1674) |

干花 `moon_tree_blossom_dried` 的数值来自干制件工厂：生命 `data.healthvalue or HEALING_TINY`＝1、饥饿 0、理智 `SANITY_SMALL`＝**+10**，食性仍 `VEGGIE`、叠 40、可作燃料、可燃且可蔓延、作祟只抛飞（`MakeHauntableLaunchAndPerish` 里的"加速腐坏"分支在原版已注释掉）。[干制定义](../../../raw/current/scripts/prefabs/driedplants_defs.lua#L38-L43)、[工厂默认值](../../../raw/current/scripts/prefabs/driedplants.lua#L51-L55)、[理智常量](../../../raw/current/scripts/tuning.lua#L2034-L2034)

S9 的两条口径要分开记：常规月树花是"构造即计时"，`OnPutInInventory` 只是再调一次 `StartPerishing`（重置任务起点，不重置剩余时间）；worldgen 版才是"拾起才开始"。`SetPerishTime` 会同时改总时长与剩余，`SetPercent` 只改剩余。[SetPerishTime](../../../raw/current/scripts/components/perishable.lua#L188-L194)、[SetPercent](../../../raw/current/scripts/components/perishable.lua#L223-L233)

### 改名带来的两个后果

`ground_fn` 在构造里就把 `inst.prefab` 改成 `moon_tree_blossom`，于是：

- **玩法判定无需分支**：祭坛 `item.prefab == "moon_tree_blossom"`、花瓶 `VASE_FLOWER_MAP`、烹饪与配方都只认这一个名字，世界摆放版自动被当成普通花。[祭坛判定](../../../raw/current/scripts/containers.lua#L818-L832)、[花瓶映射](../../../raw/current/scripts/tuning.lua#L8008-L8016)
- **postinit 会命中两次，但只在"新生成"那一次**：`SpawnPrefab` 最终走 `SpawnPrefabFromSim`——先按 `inst.prefab`（已被改成 `moon_tree_blossom`）取一次 `PrefabPostInit`，再因为 `inst.prefab ~= name` 补一次注册名（`moon_tree_blossom_worldgen`）的 postinit。所以给普通版写的 postinit 也会作用到新生成的月岛地面花。**重载不是这条路径**：存档记的是改名后的 `moon_tree_blossom`，读档只按这个名字生成，`moon_tree_blossom_worldgen` 的 postinit 不会跑在旧花上。[应用顺序](../../../raw/current/scripts/mainfunctions.lua#L387-L407)、[存档写改名后的 prefab](../../../raw/current/scripts/prefabs/moontree_blossom.lua#L84-L84)

## 来源：月树掉落与月岛摆放

| 来源 | 成立条件 | 结果 | 证据 |
|---|---|---|---|
| 小月树 `moontree_small` | 砍倒（8 次 `MOON_TREE_CHOPS_SMALL`）后一次性结算 | 2 根 `log` 必给，`moon_tree_blossom` 0.5（50% 一件） | [表](../../../raw/current/scripts/prefabs/moontree.lua#L43-L48)、[砍数](../../../raw/current/scripts/tuning.lua#L3868-L3870) |
| 普通月树 `moontree_normal` | 砍倒（12 次） | 3 根 `log`＋1 件必给花＋25% 再一件＋1 只活体 `moonbutterfly` | [表](../../../raw/current/scripts/prefabs/moontree.lua#L50-L58) |
| 高月树 `moontree_tall` | 砍倒（16 次） | 4 根 `log`＋2 件必给花＋1 只月蛾＋50% 再一只 | [表](../../../raw/current/scripts/prefabs/moontree.lua#L60-L70) |
| 月岛世界生成 | `MoonIsland_Forest` 权重 0.2、`MoonIsland_Meadows` 权重 0.5；另有 `moontrees_2` 等静态布局直接摆 `moon_tree_blossom_worldgen` | 地面上的花不腐坏，拾起才开始计时 | [森林房](../../../raw/current/scripts/map/rooms/forest/terrain_moonisland.lua#L64-L91)、[草甸房](../../../raw/current/scripts/map/rooms/forest/terrain_moonisland.lua#L141-L162)、[布局](../../../raw/current/scripts/map/static_layouts/moontrees_2.lua#L109-L117) |

掉落表用 `chance >= 1.0` 必给、其余逐条掷点的语义，所以"0.5 一件"是独立掷一次，不是"半件"。[判定](../../../raw/current/scripts/components/lootdropper.lua#L213-L227)

## 消费端

| 消费 | 条件与消耗 | 作用 | 证据 |
|---|---|---|---|
| 温蒂祭坛（sisturn） | `itemtestfn` 只放行 `petals`／`moon_tree_blossom`／`petals_evil`，且要求打开者已点 `wendy_sisturn_3` 技能（无主时放行，供读档） | 祭坛内 `> 3` 朵月树花 → 判定为 `BLOSSOM`，换 `flowers_lunar` 图案并出月花特效，联动 Abigail 加成（机制归 [[reference/prefabs/petals]]） | [收花判定](../../../raw/current/scripts/containers.lua#L818-L832)、[判定与图案](../../../raw/current/scripts/prefabs/sisturn.lua#L98-L112)、[特效与事件](../../../raw/current/scripts/prefabs/sisturn.lua#L146-L169) |
| 花瓶（vase） | 物品带 `vasedecoration` 标签、目标带 `vase` 标签时出现 `DECORATEVASE` 动作 | 按 `VASE_FLOWER_MAP.moon_tree_blossom = { 15 }` 换成第 15 号花样 | [动作](../../../raw/current/scripts/componentactions.lua#L1916-L1924)、[映射](../../../raw/current/scripts/tuning.lua#L8008-L8016) |
| 浴盐 `bathbomb` | 月树花 1＋硝石 1，`TECH.CELESTIAL_ONE`、`nounlock` | 产出浴盐；组件侧见 [[reference/components/bathbomb]] | [配方](../../../raw/current/scripts/recipes.lua#L835-L835) |
| 月亮药水 `halloween_experiment_moon` | 月蛾翅膀 1＋月树花 1＋10 理智，`TECH.MADSCIENCE_ONE`、`manufactured` | 产出能把蝴蝶变月蛾的药水（入口归 [[reference/prefabs/butterfly]]） | [配方](../../../raw/current/scripts/recipes.lua#L1420-L1420) |
| 冬宴巴甫洛娃 `wintercooking_pavlova` | 冬宴燃料 1＋月树花 2，`TECH.WINTERSFEASTCOOKING_ONE` | 冬宴节日料理 | [配方](../../../raw/current/scripts/recipes.lua#L1439-L1439) |
| 隐士茶（干花） | 空瓶 1＋`moon_tree_blossom_dried` 8／6／4（对应茶铺三档），`TECH.LOST`、`manufactured` | 产出 `hermitcrabtea_moon_tree_blossom`，理智 +10 并带对应 buff | [配方](../../../raw/current/scripts/recipes.lua#L1503-L1517)、[茶定义](../../../raw/current/scripts/prefabs/hermitcrabtea_defs.lua#L48-L52)、[筹码](../../../raw/current/scripts/recipes.lua#L1504-L1504) |
| 烹饪锅 | `cooking.lua` 只把**干花**登记为食材（`decoration = 1`、`dried = 1`），生花没有登记 | 生花不被当作有效食材 | [食材登记](../../../raw/current/scripts/cooking.lua#L167-L171) |

猪王不收：`tradable.goldvalue` 为 0，`AcceptTest` 要求大于 0（或万圣糖果值、`pig_token`）。[判定](../../../raw/current/scripts/prefabs/pigking.lua#L598-L606)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：perishable | 6 天；常规版构造即计时、worldgen 版构造后暂停；腐坏换 `spoiled_food` | 主模拟 | [[reference/components/perishable]]；[赋值](../../../raw/current/scripts/prefabs/moontree_blossom.lua#L62-L65) |
| 组件：edible | 0／1／0、`VEGGIE`；干花另有 +10 理智 | 主模拟 | [[reference/components/edible]]；[赋值](../../../raw/current/scripts/prefabs/moontree_blossom.lua#L57-L60) |
| 组件：dryable | 产物 `moon_tree_blossom_dried`、`DRY_FAST`、架面／干后架面都写 `meat_rack_food_petals`（干品本体仍 `moon_tree_petal`） | 主模拟；消耗端是晾肉架 | [[reference/components/dryable]]；[接线](../../../raw/current/scripts/prefabs/moontree_blossom.lua#L67-L71)、[架面取值](../../../raw/current/scripts/components/dryingrack.lua#L62-L76) |
| 组件：stackable | 40 | 主模拟 | [[reference/components/stackable]]；[赋值](../../../raw/current/scripts/prefabs/moontree_blossom.lua#L54-L55) |
| 组件：tradable | 未赋值，`goldvalue`＝0 | 主模拟 | [[reference/components/tradable]]；[默认值](../../../raw/current/scripts/components/tradable.lua#L1-L7) |
| 组件：vasedecoration | 空组件，只作标记；真正判定读 `vasedecoration` 标签 | 主模拟 | [[reference/components/vasedecoration]]；[组件](../../../raw/current/scripts/components/vasedecoration.lua#L1-L4)、[动作门禁](../../../raw/current/scripts/componentactions.lua#L1916-L1924) |
| 标签：`cattoy`／`dryable`／`vasedecoration` | 浣猫玩具、晾肉架可干、可插瓶；三者在 pristine 前加，客户端也认 | 两端 | [标签](../../../raw/current/scripts/prefabs/moontree_blossom.lua#L33-L36) |
| 组件：burnable／propagator | 可燃（`MakeSmallBurnable`）且可把火传给邻居（`MakeSmallPropagator`）；烧完由 `DefaultBurntFn` 在脚下生成 1 个 `ash`（继承堆叠数）再移除本体——本 prefab **没有** `lootdropper`，不是掉落表改写 | 主模拟 | [[reference/components/burnable]]；[接线](../../../raw/current/scripts/prefabs/moontree_blossom.lua#L73-L74)、[DefaultBurntFn](../../../raw/current/scripts/standardcomponents.lua#L18-L44)、[MakeSmallBurnable 接它](../../../raw/current/scripts/standardcomponents.lua#L164-L174) |
| 关联 Prefab：`moon_tree_blossom_dried` | 干制产物，独立注册（`driedplants.lua` 批量生成） | 由晾肉架完成时生成 | [定义](../../../raw/current/scripts/prefabs/driedplants_defs.lua#L38-L43)、[工厂](../../../raw/current/scripts/prefabs/driedplants.lua#L51-L70) |
| 关联 Prefab：`spoiled_food` | 腐坏替换目标 | 到点由 `perishable` 生成 | [替换](../../../raw/current/scripts/components/perishable.lua#L293-L319) |
| 资源：`anim/moon_tree_petal.zip`／`anim/meat_rack_food_petals.zip` | 生花与干品的本体 bank／build 都是 `moon_tree_petal`；`meat_rack_food_petals` 只用于晾肉架的架面与干后架面 | 原版资源，不随页分发 | [assets](../../../raw/current/scripts/prefabs/moontree_blossom.lua#L1-L11)、[干制定义](../../../raw/current/scripts/prefabs/driedplants_defs.lua#L38-L43) |

### 共享规则接入（S1／S3／S7／S8／S9／S10 适用；S2／S4／S5／S6 不适用）

- **S1 时间基准**：6 天＝2880 秒、1 天干制＝480 秒，按 `total_day_time` 推导（[基准](../../../raw/current/scripts/tuning.lua#L19-L20)）。
- **S3 标签分层**：两个注册分别列了工厂分界；本页主体是物品，标签按"可食／可干／可插瓶／可作祟"逐条给证据，没有用"植物附属品"一类统称。
- **S7 名称**：月树花／干月树花有 `NAMES` 键，`moon_tree_blossom_worldgen` 没有，靠 `SetPrefabName` 取普通版名字。
- **S8 跨页归属**：月树本体与三张 loot 表归 [[reference/prefabs/moontree]]；祭坛收花与 BLOSSOM 判定归 [[reference/prefabs/petals]] 与 [[reference/components/sisturnregistry]]；浴盐归 [[reference/components/bathbomb]]；`planted_tree.lua` 的公共 sapling 工厂归 B14。
- **S9 鲜度口径**：常规版"构造即计时"、worldgen 版"拾起才计时"，两者都不能按"掉落在地再捡才开始"统一理解（见[数值与计时](#数值与计时)）。
- **S10 待核验**：月树三档掉落概率的实测分布、`tree_petal_fx_chop` 的视觉效果未运行核验；泡茶档位与月亮药水的实际可达性只按配方推导。
- **S2／S4／S5／S6 不适用**：不涉及钓鱼链、出水投射、重量归属与月亮转换；月树花没有 `weighable`，也没有 `halloweenmoonmutable`。

## 典型接入

### 让地上的月树花也腐坏

- 前提与选择：服务端 `modmain.lua`。要"地上摆的花也会烂"就挂钩**普通名字** `moon_tree_blossom`，而不是 `moon_tree_blossom_worldgen`：地球摆放版在构造时已把 `inst.prefab` 改成 `moon_tree_blossom`（`ground_fn`），存档写的也是改名后的名字，重载时按 `moon_tree_blossom` 生成——挂 `_worldgen` 的 postinit 对旧花完全不跑。
- 生效与撤销：挂在普通名字上会**同时**作用到"新生成的地面花"和"砍树掉出来的花"，无法只挑旧 worldgen 花；启动要延后一帧（`DoTaskInTime(0, ...)`），因为读档时 `perishable:OnLoad` 会在 postinit 之后按存档 `paused` 再 `StopPerishing`。**卸载不等于回滚**：已被模组启动计时并存过档的花，存档里 `paused=nil`，卸载后仍会继续走表。要让指定目标回到暂停态，得先对它们调用 `StopPerishing()` 再存盘（`OnSave` 会写 `paused=true`），或直接把存档里的 `paused` 设成 `true`；**清掉 `paused` 只会让它继续计时**——`OnLoad` 只在 `paused` 为真时停表。新生成的花才回到原版（掉落的照常计时、地面摆放的不计时）。[OnSave 写 paused](../../../raw/current/scripts/components/perishable.lua#L329-L334)、[OnLoad 只认 paused](../../../raw/current/scripts/components/perishable.lua#L337-L348)。`SpawnPrefab` 的顺序见[改名带来的两个后果](#改名带来的两个后果)。
- 原版依据：[worldgen 版暂停](../../../raw/current/scripts/prefabs/moontree_blossom.lua#L81-L92)、[postinit 应用顺序](../../../raw/current/scripts/mainfunctions.lua#L387-L407)、[读档按 paused 恢复](../../../raw/current/scripts/components/perishable.lua#L337-L348)、[地面 1.5 倍](../../../raw/current/scripts/components/perishable.lua#L97-L105)。

```lua
-- modmain.lua：让地上的月树花也走鲜度（挂普通名字，重载时才会跑到）
AddPrefabPostInit("moon_tree_blossom", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end

    -- 读档时 OnLoad 会在 postinit 之后再按 paused 停表，所以延后一帧启动
    inst:DoTaskInTime(0, function(inst)
        local perishable = inst.components.perishable
        if perishable ~= nil then
            perishable:StartPerishing()
        end
    end)
end)
```

- 调整与验证：**满鲜度、无其它倍率**的地面花按 `PERISH_FAST`＝6 天、再乘 `PERISH_GROUND_MULT`＝1.5 计，所以约 **4 天（1920 秒）** 变 `spoiled_food`，不是 6 天；背包里的花按 1 倍（6 天）走，冰箱／雨天／冬季等其它倍率会再改这个数。验收：读档后看旧花是否开始掉鲜度、约 4 天后变腐烂物，并确认砍树掉的花也照常腐坏。示例只做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **误区：地上摆的月树花和掉落的月树花一样会烂。** 只有 `moon_tree_blossom_worldgen` 被 `StopPerishing`；砍树掉的是普通版，构造即计时。[两版差别](../../../raw/current/scripts/prefabs/moontree_blossom.lua#L81-L92)
- **误区：祭坛随便塞花就行。** `itemtestfn` 在"有主"时要求开容器的人已点 `wendy_sisturn_3`，否则只认 `petals`；月树花与 `petals_evil` 都是这条技能后的扩展。[判定](../../../raw/current/scripts/containers.lua#L818-L832)
- **边界：进祭坛的是"花"不是"干花"。** 判定只认 `moon_tree_blossom`；`moon_tree_blossom_dried` 的 `prefab` 不同，插不进去。[判定](../../../raw/current/scripts/prefabs/sisturn.lua#L98-L112)
- **边界：猪王不收，也别指望万圣节糖果。** `goldvalue`＝0 且没有 `halloweencandyvalue`，只有 `pig_token` 之类特例才过 `AcceptTest`。[判定](../../../raw/current/scripts/prefabs/pigking.lua#L598-L606)
- **边界：干制不改本体的名字。** 产物是独立注册的 `moon_tree_blossom_dried`（+10 理智、可作燃料），不是给本体换 build。[干制定义](../../../raw/current/scripts/prefabs/driedplants_defs.lua#L38-L43)
- **未核验**：月树砍伐的实际掉落分布、花在月岛之外（如移植月树）的可持续性、`DECORATEVASE` 的实际交互表现未运行验证。

## 验证记录

- 源码审阅：2026-11-09，Steam buildid 26474873（`raw/current/BUILD.txt`）。通读 `prefabs/moontree_blossom.lua` 全文、`prefabs/driedplants.lua` 与 `prefabs/driedplants_defs.lua`；核对 `prefabs/moontree.lua` 三张 loot 表、`map/rooms/forest/terrain_moonisland.lua` 三处权重与 `map/static_layouts/moontrees_2.lua` 摆放、`prefabs/sisturn.lua` 的 BLOSSOM 判定与图案、`containers.lua` 的祭坛 `itemtestfn`、`prefabs/pigking.lua` 的 `AcceptTest`、`components/perishable.lua` 的计时语义、`components/dryable.lua` 与 `components/dryingrack.lua` 的架面取值、`componentactions.lua` 的 `DECORATEVASE` 门禁、`recipes.lua` 的四条配方、`cooking.lua` 的干花食材登记、`mainfunctions.lua` 的 postinit 顺序、`tuning.lua` 的常量与 `screens/redux/scrapbookdata.lua` 的采集册条目。
- 名称核验：`languages/chinese_s.po` 的 `NAMES.MOON_TREE_BLOSSOM`＝月树花、`NAMES.MOON_TREE_BLOSSOM_DRIED`＝干月树花；`NAMES.MOON_TREE_BLOSSOM_WORLDGEN` 不存在。
- 未覆盖：晾肉架本体的湿度／酸雨结算、隐士茶 buff 的实际数值、月树本体与生长链（见 [[reference/prefabs/moontree]]）。
- R2 独立复审修订（2026-11-09）：烧毁改述为 `DefaultBurntFn` 生成 1 个 `ash`（P2-10）；典型接入补 reload 后 `OnLoad` 按 `paused` 再暂停（P2-11）；干制 build 与 BLOSSOM 归属改正（P3）。
- 外部复检修订（2026-11-09）：补"postinit 只在新生成本次命中、重载按改名后的 `moon_tree_blossom` 生成"（P1-4）；典型接入改为挂普通名字 + `DoTaskInTime(0)` 延后启动，并说明会连带掉落品、无法只挑旧 worldgen 花；验收改按地面 1.5 倍≈4 天。
- 二次复检修订（2026-11-09）：卸载说明改为"已启动并已存档的花不会因卸载恢复暂停"；"约 4 天"限定为满鲜、无其它倍率（P2-3）。
- 三次定向复检修订（2026-11-09）："回到暂停态"的指令改为"先 `StopPerishing()` 再存盘（`OnSave` 写 `paused=true`），或把存档 `paused` 设 `true`；清掉 `paused` 会重新计时"（P2-1），并补 `OnSave`／`OnLoad` 引用。
- 游戏运行：未运行；天数、秒数与三维为源码推导（`total_day_time`＝480）。
- 结构检查：`validate_page.py` 通过（standard 档内）；`validate_source_links.py --strict` 0 错 0 警。

## 相关链接

- 月树与月树苗：[[reference/prefabs/moontree]]
- 祭坛与花瓣：[[reference/prefabs/petals]]、[[reference/components/sisturnregistry]]
- 浴盐与花瓶：[[reference/components/bathbomb]]、[[reference/components/vasedecoration]]
- 干制与鲜度：[[reference/components/dryable]]、[[reference/components/perishable]]

返回 [[reference/prefabs/index|Prefab 索引]]。
