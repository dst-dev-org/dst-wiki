---
title: "花瓣族三注册（花瓣 petals、深色花瓣 petals_evil 与仙人掌花 cactus_flower）"
created: "2026-11-08"
updated: "2026-11-09"
type: prefab
summary: "花丛与邪恶花的单产物采摘件，加上仙人掌夏季开花的额外产物：三件的可食三维、食性、鲜度、堆叠、燃料与燃烧差异，以及干制、坟头装饰、花瓶、雪人装饰与作祟转化等接线。"
aliases: ["petals", "petals_evil", "cactus_flower", "花瓣", "深色花瓣", "仙人掌花", "花瓣族", "仙人掌开花"]
tags: [prefab, item, food]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/petals.lua", "prefabs/petals_evil.lua", "prefabs/cactus.lua", "prefabs/flower.lua", "prefabs/flower_evil.lua", "prefabs/tumbleweed.lua", "prefabs/catcoon.lua", "prefabs/gravestone.lua", "prefabs/graveurn.lua", "prefabs/skilltree_wendy.lua", "prefabs/driedplants_defs.lua", "containers.lua", "components/edible.lua", "components/fueled.lua", "components/perishable.lua", "components/stackable.lua", "components/pickable.lua", "components/dryable.lua", "components/dryingrack.lua", "components/upgrader.lua", "components/upgradeable.lua", "components/snowmandecoratable.lua", "standardcomponents.lua", "tuning.lua", "constants.lua", "cooking.lua", "recipes.lua", "preparedfoods.lua", "actions.lua", "componentactions.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 花瓣族三注册（花瓣 petals、深色花瓣 petals_evil 与仙人掌花 cactus_flower）

## 快速判断

- 职责与适用：三件都是可拾取的植物部位小件。`petals` 与 `petals_evil` 各由一个独立文件里的 `Prefab(...)` 注册，`cactus_flower` 与 `cactus`／`oasis_cactus` 同在一个工厂文件末尾导出。三者骨架相同（`edible`／`perishable`／`stackable`／`inspectable`／`inventoryitem`／`tradable`＋小件燃耗），差别集中在标签、燃料、干制产物、装饰组件与作祟反应。[花瓣注册](../../../raw/current/scripts/prefabs/petals.lua#L138)、[深色花瓣注册](../../../raw/current/scripts/prefabs/petals_evil.lua#L83)、[仙人掌花导出](../../../raw/current/scripts/prefabs/cactus.lua#L177-L179)
- 优先使用：改三维／鲜度看[数值](#数值)；改可燃时长、能否当燃料看[燃烧与燃料](#燃烧与燃料)；干制、坟头装饰、花瓶与雪人装饰看[获得与去向](#获得与去向)。`cactus_meat` 的采摘门禁与生熟正负归 [[reference/prefabs/tropical-gathered-fruits#仙人掌肉的生熟正负]]，本页不复述。
- 重要副作用：`petals_evil` 的理智 −5 让它自动带 `badfood`；`petals` 被作祟时约一半概率把整叠换成 `petals_evil`，并继承堆叠数、湿度与剩余新鲜度。[badfood hook](../../../raw/current/scripts/components/edible.lua#L1-L7)、[作祟反应](../../../raw/current/scripts/prefabs/petals.lua#L14-L56)
- 修改边界：不含 `petals_dried`／`petals_evil_dried` 的独立注册与食用数值（只在[干制](#干制与干品)给接线）；不含花丛／邪恶花本体行为（见 [[reference/prefabs/flowers]]）；`cactus`／`oasis_cactus` 只写到「花从哪来」（见 [[reference/prefabs/cactus]]）。

## 注册与构造

三件都在 `SetPristine` 前完成动画与标签，组件的挂载全部在主模拟分支内，因此标签双端可见、组件只在服务端存在。

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `petals` | `fn`：bank `petals`、build `flower_petals`、`PlayAnimation("anim")`、`pickupsound = "vegetation_grassy"`；pristine 前加 `cattoy`／`vasedecoration`／`dryable` 三个 tag，浮水走 `MakeInventoryFloatable(inst)` 默认 small → `SetPristine` → 主模拟挂 `edible`／`fuel`／`inspectable`／`inventoryitem`／`perishable`／`stackable`／`tradable`／`upgrader`／`vasedecoration`／`snowmandecor`／`dryable` 与小件燃耗／传播 helper | 本族唯一带 `upgrader`（`UPGRADETYPES.GRAVESTONE`）与 `vasedecoration` 的一件；末尾 `MakeHauntableLaunchAndPerish` 后再叠 `AddHauntableCustomReaction(inst, OnHaunt, false, true, false)`，自定义反应排在原反应之前。[`upgrader`](../../../raw/current/scripts/prefabs/petals.lua#L115-L116)、[装饰与燃耗](../../../raw/current/scripts/prefabs/petals.lua#L119-L122)、[作祟接线](../../../raw/current/scripts/prefabs/petals.lua#L132-L133) |
| `petals_evil` | `fn`：bank／build 都是 `flower_petals_evil`、`PlayAnimation("anim")`；pristine 前只加 `dryable` 一个 tag，浮水默认 small → `SetPristine` → 主模拟挂 `stackable`／`tradable`／`inspectable`／`fuel`／`inventoryitem`／`edible`／`perishable`／`snowmandecor`／`dryable` | 无 `vasedecoration`、无 `upgrader`、无 `cattoy`；`dryable` 产物是 `petals_evil_dried`；`MakeHauntableLaunchAndPerish` 后监听 `spawnedfromhaunt`，被转化出来的新件按 `LAUNCH_SPEED_SMALL` 抛出。[干制接线](../../../raw/current/scripts/prefabs/petals_evil.lua#L71-L78)、[抛出](../../../raw/current/scripts/prefabs/petals_evil.lua#L13-L15) |
| `cactus_flower` | `cactusflowerfn`：bank `cactusflower`、build `cactus_flower`、`PlayAnimation("idle")`；不加任何 tag，浮水默认 small → `SetPristine` → 主模拟挂 `inspectable`／`inventoryitem`／`edible`／`perishable`／`stackable`／`tradable` 与小件燃耗／传播 helper | 无 `fuel`、无 `dryable`、无 `snowmandecor`、无 `vasedecoration`；作祟用 `MakeHauntableLaunch` 而非 LaunchAndPerish。[工厂](../../../raw/current/scripts/prefabs/cactus.lua#L129-L175) |

### 数值

天数为「游戏日」，1 天 = `TUNING.TOTAL_DAY_TIME` = 16 × `seg_time`(30) = 480 秒。[基准](../../../raw/current/scripts/tuning.lua#L19-L20)、[`TOTAL_DAY_TIME`](../../../raw/current/scripts/tuning.lua#L63)

| 注册 | 饥饿 | 生命 | 理智 | 食性 | 鲜度 | 堆叠 | 燃料值 | 燃烧时长 | 浮水 |
|---|---|---|---|---|---|---|---|---|---|
| `petals` | 0 | **+1**（`HEALING_TINY`） | 0 | `FOODTYPE.VEGGIE` | 6 天（`PERISH_FAST` = 2880 秒） | 40 | 7.5 秒（`TINY_FUEL`） | 3 秒（`TINY_BURNTIME`） | 默认 small |
| `petals_evil` | 0 | 0 | **−5**（`-SANITY_TINY`） | `FOODTYPE.VEGGIE` | 6 天 | 40 | 7.5 秒 | 3 秒 | 默认 small |
| `cactus_flower` | **12.5**（`CALORIES_SMALL`） | **+8**（`HEALING_MEDSMALL`） | **+5**（`SANITY_TINY`） | `FOODTYPE.VEGGIE` | 3 天（`PERISH_SUPERFAST` = 1440 秒） | 40 | 无 `fuel` 组件 | 10 秒（helper 默认） | 默认 small |

- 赋值来源：`petals` 只显式写 `healthvalue`／`hungervalue`／`foodtype`，理智留组件默认 0；`petals_evil` 写 0／0／`-SANITY_TINY`；`cactus_flower` 三项全写。[花瓣](../../../raw/current/scripts/prefabs/petals.lua#L86-L89)、[深色花瓣](../../../raw/current/scripts/prefabs/petals_evil.lua#L58-L62)、[仙人掌花](../../../raw/current/scripts/prefabs/cactus.lua#L153-L157)
- `badfood` 由 `edible` 的属性 setter 结算：`healthvalue < 0` 或 `sanityvalue < 0` 即打标，否则移除。所以只有 `petals_evil` 带该标签，`petals`／`cactus_flower` 不带；赋 `foodtype` 时同时加 `edible_VEGGIE`。[判定](../../../raw/current/scripts/components/edible.lua#L1-L7)、[食性 hook](../../../raw/current/scripts/components/edible.lua#L9-L17)
- 常量：`HEALING_TINY`=1、`HEALING_MEDSMALL`=8、`SANITY_TINY`=5、`CALORIES_SMALL`=12.5、`PERISH_FAST`=6 天、`PERISH_SUPERFAST`=3 天、`STACK_SIZE_SMALLITEM`=40、`TINY_FUEL`=7.5、`TINY_BURNTIME`=3。[生命](../../../raw/current/scripts/tuning.lua#L2021-L2023)、[理智](../../../raw/current/scripts/tuning.lua#L2032-L2033)、[鲜度](../../../raw/current/scripts/tuning.lua#L2045-L2046)、[热量](../../../raw/current/scripts/tuning.lua#L2059)、[堆叠](../../../raw/current/scripts/tuning.lua#L83)、[燃料与燃烧](../../../raw/current/scripts/tuning.lua#L1673-L1681)
- 堆叠 40 是显式赋值，`stackable` 组件自身默认是 `STACK_SIZE_MEDITEM`＝20；三件都显式覆盖。[赋值](../../../raw/current/scripts/prefabs/petals.lua#L108-L109)、[默认](../../../raw/current/scripts/components/stackable.lua#L23-L29)
- 腐坏结算对三件同样适用：陈旧饥饿 ×0.667、腐烂 ×0.5；负的生命／理智跳过折扣，腐烂时原本非负的理智改为 `-SANITY_SMALL`（−10）。所以 `petals` 腐烂后只剩理智 −10；`cactus_flower` 腐烂后生命归零、理智 −10，但饥饿仍按 ×0.5 结算（12.5→6.25）。[折扣](../../../raw/current/scripts/components/edible.lua#L96-L119)、[腐烂理智](../../../raw/current/scripts/components/edible.lua#L72-L86)
- 鲜度门槛与损耗：`PERISH_FRESH`＝0.5、`PERISH_STALE`＝0.2 决定 fresh／stale／spoiled 标签；无主物品在地面按 `PERISH_GROUND_MULT`＝1.5 加速。`SetPerishTime` 同时写 `perishtime` 与剩余秒数，腐坏时用 `onperishreplacement` 换 `spoiled_food`。[门槛](../../../raw/current/scripts/components/perishable.lua#L1-L27)、[写值](../../../raw/current/scripts/components/perishable.lua#L188-L194)、[替换](../../../raw/current/scripts/components/perishable.lua#L293-L302)

### 燃烧与燃料

- `petals`／`petals_evil` 显式传 `TUNING.TINY_BURNTIME`（3 秒）给 `MakeSmallBurnable`；`cactus_flower` 不传第二个参数，落 helper 的默认 10 秒，不是 `SMALL_BURNTIME`（7.5 秒）。[helper 默认](../../../raw/current/scripts/standardcomponents.lua#L164-L174)、[仙人掌花接线](../../../raw/current/scripts/prefabs/cactus.lua#L169-L170)
- `fuel` 组件与 `burnable` 是两条链：前者是「投进火堆／火坑的燃料值」，被 `fueled:TakeFuelItem` 乘倍率后写进 `fueled` 的燃烧秒数；后者是「物品在地上会被点着」。`cactus_flower` 只有后者。[取料](../../../raw/current/scripts/components/fueled.lua#L198-L214)、[燃料赋值](../../../raw/current/scripts/prefabs/petals.lua#L92-L93)、[仙人掌花无 fuel](../../../raw/current/scripts/prefabs/cactus.lua#L159-L170)

### 获得与去向

| 注册 | 路径 | 门禁与细节 | 证据 |
|---|---|---|---|
| `petals` | 花丛 `flower` 采摘 | `pickable:SetUp("petals", 10)`＋`remove_when_picked`＋`quickpick`：采摘即移除花丛，走 `SpawnPrefab("petals")` 单产物分支（不查 loot 表）；采摘者非 `plantkin` 另得 `SANITY_TINY` | [接线](../../../raw/current/scripts/prefabs/flower.lua#L170-L176)、[采摘附带理智](../../../raw/current/scripts/prefabs/flower.lua#L52-L58)、[单产物分支](../../../raw/current/scripts/components/pickable.lua#L493-L513) |
| `petals` | 风滚草／浣猫回礼／回收坟头装饰 | 风滚草表权重 1；浣猫「中立」第 6 档与「朋友」第 2 档含 `petals`（深色花瓣在中立第 5 档与朋友第 2 档）；用 `graveurn` 的 gravedigger 收走已装饰的坟时，会把装饰阶段退回 1 并按 3 次 ×50% 掉回 `petals` | [风滚草](../../../raw/current/scripts/prefabs/tumbleweed.lua#L152)、[浣猫礼物](../../../raw/current/scripts/prefabs/catcoon.lua#L78-L119)、[回收](../../../raw/current/scripts/prefabs/graveurn.lua#L120-L135) |
| `petals_evil` | 邪恶花 `flower_evil` 采摘 | `SetUp("petals_evil", 10)`＋`remove_when_picked`；采摘者理智 −5，Wendy 的 `wendy_gravestone_1` 技能改为 +5；花体自身带 `sanityaura` = `-SANITYAURA_SMALL` | [接线](../../../raw/current/scripts/prefabs/flower_evil.lua#L68-L74)、[采摘理智](../../../raw/current/scripts/prefabs/flower_evil.lua#L25-L33) |
| `petals_evil` | 风滚草权重 1／浣猫回礼；`petals` 作祟转化 | 作祟分支继承堆叠数、湿度与剩余新鲜度，并推 `spawnedfromhaunt`／`despawnedfromhaunt` | [作祟](../../../raw/current/scripts/prefabs/petals.lua#L23-L52)、[风滚草](../../../raw/current/scripts/prefabs/tumbleweed.lua#L176) |
| `cactus_flower` | `cactus`／`oasis_cactus` 采摘且植株当前带花 | 花**不是** `pickable.product`（产物是 `cactus_meat`）；`has_flower` 只在夏季的 `onregenfn` 与 `OnEntityWake` 置真，采摘时有采摘者且植株带花就由 `onpickedfn` 独立于刺伤门禁追加 1 朵（`SpawnPrefab("cactus_flower")`、先 `InheritWorldWetnessAtTarget` 再入包或落地） | [开花与采摘](../../../raw/current/scripts/prefabs/cactus.lua#L18-L56)、[唤醒校正](../../../raw/current/scripts/prefabs/cactus.lua#L64-L72)、[再生 4 天](../../../raw/current/scripts/prefabs/cactus.lua#L104-L111) |

#### 干制与干品

- 三件里只有 `petals`／`petals_evil` 有 `dryable`：产物 `petals_dried`／`petals_evil_dried`，干制时间 `DRY_FAST`＝1 天（480 秒），`SetBuildFile` 与 `SetDriedBuildFile` 都指向 `meat_rack_food_petals`；`dryable` 组件构造时自己补一次 `dryable` tag，两个 prefab 另在 pristine 前显式加同一 tag（源码注释自称「优化」）。[干制接线](../../../raw/current/scripts/prefabs/petals.lua#L126-L130)、[tag 自补](../../../raw/current/scripts/components/dryable.lua#L1-L18)、[`DRY_FAST`](../../../raw/current/scripts/tuning.lua#L2055)
- 消费端是晾肉架：入架判定 `params.meatrack.itemtestfn` 只要求 `item:HasTag("dryable")`；真正干制时 `dryingrack` 读 `GetProduct()`／`GetDryTime()`，计时到期才换上干品。[入架判定](../../../raw/current/scripts/containers.lua#L234-L244)、[干制](../../../raw/current/scripts/components/dryingrack.lua#L245-L266)
- 干品数值与原件不同：`petals_dried` 理智 +1，`petals_evil_dried` 仍 −5；两者的食材值也只登记在干品上。[干品定义](../../../raw/current/scripts/prefabs/driedplants_defs.lua#L4-L15)、[食材值](../../../raw/current/scripts/cooking.lua#L169-L171)

#### 去向与消费

- **制造材料**：`flowerhat` 花瓣 12、`grass_umbrella` 6、`turf_grass` 1、`reskin_tool` 4、`lighter` 3；深色花瓣进 `nightmarefuel`（4）、`wendy_gravestone`（4）、`halloween_experiment_sanity`（1）、`wintercooking_mulleddrink`（1）。[花冠](../../../raw/current/scripts/recipes.lua#L529)、[深色花瓣配方](../../../raw/current/scripts/recipes.lua#L467)、[`wendy_gravestone`](../../../raw/current/scripts/recipes.lua#L161)
- **坟头装饰**：只有 `petals` 挂 `upgrader`，`gravestone` 侧挂 `upgradeable`（2 阶段、每阶段 3 件）；`ACTIONS.UPGRADE` 先过 `upgrader:CanUpgrade`——类型须同为 `GRAVESTONE`，且执行者要有 `gravestone_upgradeuser` tag（Wendy `wendy_gravestone_1` 技能给）。[坟](../../../raw/current/scripts/prefabs/gravestone.lua#L270-L275)、[门禁](../../../raw/current/scripts/components/upgrader.lua#L21-L29)、[动作](../../../raw/current/scripts/actions.lua#L4056-L4069)、[技能 tag](../../../raw/current/scripts/prefabs/skilltree_wendy.lua#L208-L228)
- **花瓶**：`vasedecoration` tag＋组件（与 `forgetmelots` 同形），可装饰带 `vase` 的目标，机制见 [[reference/components/vasedecoration]]。
- **雪人装饰**：`snowmandecor` 组件＋`snowmandecoratable` 的映射（两件都有 bank／build／`snowman_decor` 动画、`canflip`）。[映射](../../../raw/current/scripts/components/snowmandecoratable.lua#L36-L37)
- **姐姐祭坛**：祭坛容器检定只收 `petals`；当打开者拥有 Wendy 的 `wendy_sisturn_3` 技能、或读档时无 owner，才同时接受 `petals_evil` 与 `moon_tree_blossom`（月树花本体见 [[reference/prefabs/moontree-blossom]]）。[检定](../../../raw/current/scripts/containers.lua#L818-L832)
- **烹饪口径**：`petals`／`petals_evil` 在 `cooking.lua` 没有 `AddIngredientValues` 条目，`IsCookingIngredient` 查表即返回假，下不了锅；`cactus_flower` 是 `veggie = 0.5` 食材，另有专属料理判定（`names.cactus_flower` 且 `veggie ≥ 2`）与 `hawaiianshirt`（5 个）配方。[登记表](../../../raw/current/scripts/cooking.lua#L139-L142)、[查表](../../../raw/current/scripts/cooking.lua#L185-L187)、[专属料理](../../../raw/current/scripts/preparedfoods.lua#L367)、[衬衫](../../../raw/current/scripts/recipes.lua#L551)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`edible` | 写三维与 `foodtype`；负数触发 `badfood` | 主模拟；赋值即结算标签 | [[reference/components/edible#关键接口]]；[赋值](../../../raw/current/scripts/prefabs/petals_evil.lua#L58-L62) |
| 组件：`perishable` | `SetPerishTime`＋`StartPerishing`＋`onperishreplacement` | 主模拟；三件都用 `spoiled_food` 收尾 | [[reference/components/perishable#关键接口]]；[接线](../../../raw/current/scripts/prefabs/petals.lua#L102-L105) |
| 组件：`stackable` | 显式 `maxsize = 40` | 主模拟；组件默认 20 | [[reference/components/stackable#关键接口]]；[赋值](../../../raw/current/scripts/prefabs/petals_evil.lua#L43-L44) |
| 组件：`dryable` | `petals`／`petals_evil` 的干制产物与时长 | 主模拟；客户端只看到 pristine tag | [[reference/components/dryable#关键接口]]；[接线](../../../raw/current/scripts/prefabs/petals_evil.lua#L71-L75) |
| 组件：`upgrader` | 只 `petals`：`upgradetype = GRAVESTONE` | 主模拟；配 `upgradeable`＋执行者 tag | [[reference/components/upgrader#关键接口]]；[接线](../../../raw/current/scripts/prefabs/petals.lua#L115-L116) |
| 组件：`vasedecoration` | `petals` 的 tag 在 pristine 前，组件在主模拟 | 双端 tag、单端组件 | [[reference/components/vasedecoration#典型接入]]；[接线](../../../raw/current/scripts/prefabs/petals.lua#L119) |
| 组件：`snowmandecor` | `petals`／`petals_evil` 都有，空组件仅作标记 | 主模拟 | [[reference/components/snowmandecor]]；[映射](../../../raw/current/scripts/components/snowmandecoratable.lua#L36-L37) |
| helper：`MakeSmallBurnable`／`MakeSmallPropagator` | 点火与火焰传播 | 双端可用；燃烧时长按参数 | [helper](../../../raw/current/scripts/standardcomponents.lua#L164-L174) |
| helper：`MakeHauntableLaunchAndPerish` | `petals`／`petals_evil` 的作祟抛飞 | 主模拟；perish 分支已被注释 | [实现](../../../raw/current/scripts/standardcomponents.lua#L1311-L1337) |
| helper：`AddHauntableCustomReaction` | `petals` 叠自定义作祟反应 | 主模拟；`secondrxn=false` 先跑自定义 | [实现](../../../raw/current/scripts/standardcomponents.lua#L1528-L1552) |
| 关联 Prefab：`flower`／`flower_evil` | 花瓣产地，采摘即移除 | 世界资源；邪恶花另有 `sanityaura` | [花丛](../../../raw/current/scripts/prefabs/flower.lua#L167-L176)、[邪恶花](../../../raw/current/scripts/prefabs/flower_evil.lua#L63-L74) |
| 关联 Prefab：`cactus`／`oasis_cactus` | 夏季开花时额外给花 | `SetPrefabNameOverride("cactus")`，同一工厂 | [工厂](../../../raw/current/scripts/prefabs/cactus.lua#L74-L127) |
| 数据：`UPGRADETYPES.GRAVESTONE` | 坟头装饰的类型键 | 与 `upgradeable.upgradetype` 成对匹配 | [常量](../../../raw/current/scripts/constants.lua#L1969-L1977) |

## 典型接入

### 让深色花瓣不再扣理智

- 前提与选择：服务端 `modmain.lua`。`petals_evil` 是独立注册，不能靠改 `petals` 的字段回灌；改 `edible.sanityvalue` 走属性 setter，会顺带重算 `badfood`。[setter](../../../raw/current/scripts/components/edible.lua#L1-L7)
- 生效与撤销：postinit 在每次构造时运行，只影响新生成的实例；停用模组即恢复工厂原值。存档里的旧实例读档不重跑该赋值以外的东西，新鲜度仍按 `Perishable:OnLoad` 照抄。
- 原版依据：数值与 `badfood` 来源见[数值](#数值)；同形态的改法见 [[reference/prefabs/tropical-gathered-fruits#让生仙人掌肉不再带-badfood]]。

```lua
-- modmain.lua：把深色花瓣的理智损失归零（0 不再触发 badfood）
AddPrefabPostInit("petals_evil", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    local edible = inst.components.edible
    if edible ~= nil then
        edible.sanityvalue = 0
    end
end)
```

- 调整与验证：单位是理智点；`sanityvalue` 是基础值，腐坏倍率与香料加成在 `GetSanity` 里另算，所以改成 0 后陈旧／腐烂仍不会再扣。未跑游戏验证。

## 常见陷阱与限制

- **边界：不传参数不等于小件时长**：`cactus_flower` 的 `MakeSmallBurnable(inst)` 落默认 10 秒，`petals`／`petals_evil` 才是 3 秒；把 `SMALL_BURNTIME` 当默认会算错。[默认值](../../../raw/current/scripts/standardcomponents.lua#L164-L174)
- **误区：`fuel` 与 `burnable` 混用**：`cactus_flower` 能被点着但没有 `fuel`，不能当燃料投入；反过来，只有 `fuel` 也不代表能被火点燃。
- **边界：`MakeHauntableLaunchAndPerish` 名不副实**：该 helper 的 perish 分支整段被注释，只保留 launch，所以 `petals` 的「变深色花瓣」并非来自它，而是 `OnHaunt` 自定义反应。[helper](../../../raw/current/scripts/standardcomponents.lua#L1324-L1332)
- **边界：`cactus_flower` 不是 `pickable.product`**：改 `pickable` 的产物、`numtoharvest` 或 `droppicked` 都不影响花；花只在 `onpickedfn` 里按 `has_flower` 生成。[采摘](../../../raw/current/scripts/prefabs/cactus.lua#L29-L42)
- **边界：干制改写收益**：干花瓣 +1 理智、干深色花瓣仍 −5，且食材值只登记在干品上——不是「干制等于保鲜」。[干品](../../../raw/current/scripts/prefabs/driedplants_defs.lua#L4-L15)
- **未核验**：`AddHauntableCustomReaction` 传入 `ignoreinitialresult = true`，自定义反应成功后仍会继续执行原来的 launch 反应；对已 `Hide()` 且排队移除的实体调用 `Launch` 的实际表现未做运行验证。[接线](../../../raw/current/scripts/prefabs/petals.lua#L132-L133)

## 验证记录

- 源码审阅：2026-11-08，Steam buildid 26335394。核对 `petals.lua`／`petals_evil.lua` 全文（标签、浮水、组件顺序、作祟反应）、`cactus.lua` 的开花与采摘分支及 `cactus_flower` 工厂、`flower.lua`／`flower_evil.lua` 的 `pickable` 接线、`tumbleweed`／`catcoon` 掉落表、`gravestone`／`graveurn`／`skilltree_wendy`／`actions`／`componentactions` 的 UPGRADE 链、`containers` 的祭坛检定、`cooking`／`recipes`／`preparedfoods` 的食材与配方、`edible`／`perishable`／`stackable`／`pickable`／`dryable`／`dryingrack`／`upgrader`／`snowmandecoratable` 相关分支、`standardcomponents` 四个 helper 与 `tuning`／`constants` 常量。
  名称核验：`NAMES.PETALS`＝花瓣、`PETALS_EVIL`＝深色花瓣、`CACTUS_FLOWER`＝仙人掌花。[PETALS](../../../raw/current/scripts/languages/chinese_s.po#L346820-L346822)、[PETALS_EVIL](../../../raw/current/scripts/languages/chinese_s.po#L346830-L346832)、[CACTUS_FLOWER](../../../raw/current/scripts/languages/chinese_s.po#L340060-L340062)
  未覆盖：花丛／邪恶花的再生与世界布点、`petals_dried`／`petals_evil_dried` 的独立注册全文、`snowmandecor` 装饰动作的执行链、动态选名与模组注入的生成点；`cactus_meat` 的数值与门禁归 [[reference/prefabs/tropical-gathered-fruits]]。
- 游戏运行：未运行。三维、天数与秒数均由 `tuning` 常量推导，非运行测量。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 通过；页内 Lua 片段经 `parser_probe.exe --syntax-only` 通过。
- 复核：2026-11-08，Steam buildid 26474873。核对本页引用源在 26335394→26474873 间的差异（`tuning.lua`、`plantables.lua` 各插入 1 行，`SGwilson.lua` 仅换声效），锚点按新树重定位、结论未变；上一条 26335394 审阅记录保留作历史。

## 相关链接

- 仙人掌肉的采摘门禁与生熟两态：[[reference/prefabs/tropical-gathered-fruits]]
- 干制／花瓶／雪人装饰的组件侧机制：[[reference/components/dryable]]、[[reference/components/vasedecoration]]、[[reference/components/snowmandecor]]
- 花瓣族的上游物品与随机株：[[reference/prefabs/generic-seeds]]

返回 [[reference/prefabs/index|Prefab 索引]]。
