---
title: "海带叶三态（kelp、kelp_cooked、kelp_dried）"
created: "2026-11-08"
updated: "2026-11-08"
type: prefab
summary: "veggies.lua 的 SEEDLESS 条目 kelp 只导出「生／熟／干」三件：生熟都带负理智、干制后转正的数值差，dryable 的 120 秒晾晒与产物字段，海带丛与冲岸株的采集，以及 KELP 材质修补、厨锅与盐槽等消费链。"
aliases: ["kelp", "kelp_cooked", "kelp_dried", "海带叶", "熟海带叶", "干海带叶", "海带", "Kelp Fronds", "Cooked Kelp Fronds", "Dried Kelp Fronds"]
tags: [prefab, item, food]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/veggies.lua", "prefabs/bullkelp_plant.lua", "prefabs/bullkelp_beached.lua", "prefabs/meatrack2.lua", "prefabs/oceanfishableflotsam.lua", "prefabs/otterden.lua", "prefabs/crabking_cannontower.lua", "prefabs/boat_bumpers.lua", "prefabs/boat.lua", "recipes.lua", "components/dryable.lua", "components/dryingrack.lua", "components/dryer.lua", "components/perishable.lua", "components/edible.lua", "components/stackable.lua", "components/repairable.lua", "components/driedsalticon.lua", "components/dryingracksaltcollector.lua", "components/pickable.lua", "components/eater.lua", "standardcomponents.lua", "tuning.lua", "constants.lua", "cooking.lua", "preparedfoods.lua", "containers.lua", "entityscript.lua", "prefabs/wurt.lua", "prefabs/merm.lua", "screens/redux/scrapbookdata.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 海带叶三态（kelp、kelp_cooked、kelp_dried）

## 快速判断

- 职责与适用：`prefabs/veggies.lua` 的 `MakeVeggie` 为 `VEGGIES.kelp` 生成 3 个可拾取食物：生 `kelp`、烤 `kelp_cooked`、干 `kelp_dried`。`kelp` 在 `SEEDLESS` 中，不注册种子与巨型变体（见 [[reference/prefabs/giant-crops#快速判断]]）；20 项 `VEGGIES` 里也只有它带 `dryable`，干制态因此唯一。[条目](../../../raw/current/scripts/prefabs/veggies.lua#L118-L121)、[SEEDLESS](../../../raw/current/scripts/prefabs/veggies.lua#L160-L168)、[导出](../../../raw/current/scripts/prefabs/veggies.lua#L1084-L1099)
- 优先使用：改三维／保质看[三态数值](#三态数值)；晾晒与 `dryable` 字段看[干制链](#干制链)；世界来源看[获得与入栏](#获得与入栏)。共用工厂、`MakeVegStats` 位序与生熟两态通则见 [[reference/prefabs/farm-crops-staple#共用工厂与生熟两态]]，本页只写海带差异。
- 重要副作用：生叶 −1 生命／−10 理智，熟叶只去掉生命惩罚（−5 理智），干叶 +1 生命／+10 理智；生熟都因负值带上 `badfood`，干制才转正。[生熟数值](../../../raw/current/scripts/prefabs/veggies.lua#L118-L121)、[badfood](../../../raw/current/scripts/components/edible.lua#L1-L7)
- 修改边界：本页覆盖三件物品的构造、数值、获得与消费；海带丛 `bullkelp_plant` 本体（Wiki 待建）、海带根见 [[reference/prefabs/bullkelp_root]]、海带补丁见 [[reference/prefabs/boat-patches]]，只作关联入口。

## 注册与构造

`MakeVeggie("kelp", false)`（`SEEDLESS.kelp = true`）只导出 3 件，顺序为 `kelp_dried`（`dryable ~= nil` 时插入）→ `kelp` → `kelp_cooked`；三件都是物品＋食物，无装备、建筑或生物职责。[调用](../../../raw/current/scripts/prefabs/veggies.lua#L1102-L1108)、[导出段](../../../raw/current/scripts/prefabs/veggies.lua#L1086-L1097)

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `kelp` | `fn`：bank／build＝`kelp`、`PlayAnimation("idle")`、`pickupsound="vegetation_firm"`、公共端 tag `cookable`＋`dryable`；浮水 `{"med",nil,0.7}` → `SetPristine`（非主模拟返回）→ 主模拟逐项挂组件 | edible／perishable／stackable 40／dryable／repairer（KELP、+10 生命）／inspectable／inventoryitem／burnable＋propagator／bait／tradable／cookable→`kelp_cooked`／hauntable；无 weighable。[工厂](../../../raw/current/scripts/prefabs/veggies.lua#L539-L704)、[公共端](../../../raw/current/scripts/prefabs/veggies.lua#L560-L565) |
| `kelp_cooked` | `fn_cooked`：同 bank／build、`PlayAnimation("cooked")`、`scrapbook_anim="cooked"`；浮水 `{"med",nil,0.65}` → `SetPristine` → 主模拟 | 只有 perishable／edible／stackable 40／inspectable／inventoryitem／burnable＋propagator／bait／tradable／hauntable；无 cookable、dryable、repairer。[工厂](../../../raw/current/scripts/prefabs/veggies.lua#L707-L777) |
| `kelp_dried` | `fn_dried`：bank／build＝`dryable.build`（`meat_rack_food_tot`）、`PlayAnimation("dried_kelp")`、`scrapbook_anim="dried_kelp"`；额外挂 `driedsalticon` 并 `SetCollectsOnDried(true)`；浮水默认 small → `SetPristine` → 主模拟 | 与熟态同组件集（含 bait／tradable），另加 `driedsalticon`；无 cookable、dryable、repairer。[工厂](../../../raw/current/scripts/prefabs/veggies.lua#L779-L833) |

端分界与同族一致：`SetPristine` 之前是双端可读的公共端，其后的 `if not TheWorld.ismastersim then return inst end` 短路客户端，组件与数值只在主模拟出现；细节见 [[reference/prefabs/farm-crops-staple#导出集合与端分界]]。[生端](../../../raw/current/scripts/prefabs/veggies.lua#L589-L593)、[熟端](../../../raw/current/scripts/prefabs/veggies.lua#L734-L738)、[干端](../../../raw/current/scripts/prefabs/veggies.lua#L800-L804)

资源按态拼：生／熟共用 `anim/kelp.zip`，熟态另加 `INV_IMAGE kelp_cooked`；干态只加载 `anim/meat_rack_food_tot.zip`（架上显示与本体同 build），动画名 `dried_kelp`。[生熟资源](../../../raw/current/scripts/prefabs/veggies.lua#L392-L411)、[干态资源](../../../raw/current/scripts/prefabs/veggies.lua#L433-L437)

### 三态数值

时间基准 `total_day_time`＝`seg_time`×16＝480 秒，`perish_warp`=1，饥饿基准 `calories_per_day`=75。[基准](../../../raw/current/scripts/tuning.lua#L19-L20)、[热量](../../../raw/current/scripts/tuning.lua#L41-L41)、[perish_warp](../../../raw/current/scripts/tuning.lua#L47-L47)

| 注册 | 饥饿／生命／理智 | 新鲜度 | 堆叠 | 浮水（size／offset／scale） |
|---|---|---|---|---|
| `kelp` | 9.375／**−1**／**−10**（`CALORIES_TINY`／`−HEALING_TINY`／`−SANITY_SMALL`） | 10 天＝4800 秒（`PERISH_MED`） | 40 | `med`／nil／0.7 |
| `kelp_cooked` | 9.375／0／**−5**（`CALORIES_TINY`／0／`−SANITY_TINY`） | 10 天＝4800 秒 | 40 | `med`／nil／0.65 |
| `kelp_dried` | 9.375／**+1**／**+10**（`CALORIES_TINY`／`HEALING_TINY`／`SANITY_SMALL`） | 20 天＝9600 秒（`PERISH_PRESERVED`） | 40 | `small`（默认） |

常量：`TUNING.CALORIES_TINY`＝`calories_per_day/8`＝9.375、`HEALING_TINY`=1、`SANITY_TINY`=5、`SANITY_SMALL`=10、`PERISH_MED`＝10×`total_day_time`、`PERISH_PRESERVED`＝20×`total_day_time`。[热量档](../../../raw/current/scripts/tuning.lua#L2058-L2058)、[治疗](../../../raw/current/scripts/tuning.lua#L2021-L2021)、[理智](../../../raw/current/scripts/tuning.lua#L2033-L2034)、[保质档](../../../raw/current/scripts/tuning.lua#L2048-L2050)

干叶的三维不走 `cooked_*` 位，而是 `dryable` 子表的 `hunger`／`health`／`sanity`；`fn_dried` 用 `dryable.health or 0` 等逐项写入，保质读 `dryable.perish`。[配置](../../../raw/current/scripts/prefabs/veggies.lua#L118-L121)、[取值](../../../raw/current/scripts/prefabs/veggies.lua#L806-L815)

堆叠三态都在构造里显式赋值 `TUNING.STACK_SIZE_SMALLITEM`＝40，`stackable` 组件默认的 `STACK_SIZE_MEDITEM`（20）不生效。[默认](../../../raw/current/scripts/components/stackable.lua#L23-L34)、[档位](../../../raw/current/scripts/tuning.lua#L83-L83)、[生](../../../raw/current/scripts/prefabs/veggies.lua#L613-L619)、[熟](../../../raw/current/scripts/prefabs/veggies.lua#L752-L753)、[干](../../../raw/current/scripts/prefabs/veggies.lua#L817-L818)

图鉴数据与工厂同值（9.375／−1、0、+1；−10、−5、+10；4800、4800、9600），可作交叉核对。[图鉴](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L737-L739)

### 获得与入栏

只有生 `kelp` 有世界来源；`kelp_cooked` 与 `kelp_dried` 分别由 `cookable` 与晾晒架按产物名动态生成（全 scripts 检索 `SpawnPrefab("kelp_cooked")`／`("kelp_dried")` 均零命中）。

- **海带丛（主来源）**：海面 `bullkelp_plant` 的 `pickable:SetUp("kelp", TUNING.BULLKELP_REGROW_TIME)`，每次 1 件、不毁株；基础再生 3×480＝1440 秒，春季再乘 `SPRING_GROWTH_MODIFIER`＝0.75。植株官方名是「公牛海带」（`bullkelp_plant`）。[采集](../../../raw/current/scripts/prefabs/bullkelp_plant.lua#L104-L109)、[再生常量](../../../raw/current/scripts/tuning.lua#L3798-L3798)、[SetUp](../../../raw/current/scripts/components/pickable.lua#L222-L228)
- **冲岸**：植物漂上陆地／船时先试 `pickable:Pick(TheWorld)`——只有通过采摘门禁（`canbepicked`）才掉 1 kelp——随后无条件在原位生成 `bullkelp_root`。[CheckBeached](../../../raw/current/scripts/prefabs/bullkelp_plant.lua#L42-L54)
- **拾取冲岸株**：`bullkelp_beachedroot` 被拾取／入包时替换为 kelp＋`bullkelp_root` 各 1，并继承湿度。[替换](../../../raw/current/scripts/prefabs/bullkelp_beached.lua#L15-L35)、[注册名](../../../raw/current/scripts/prefabs/bullkelp_beached.lua#L88-L88)
- **掉落池**：海洋漂流物 `oceanfishableflotsam` 权重 35；水獭掠夺者窝点 `otterden` 池 1.00／1.00／0.75；帝王蟹炮塔 `ck_cannontower` 池 0.75；海带防撞条被锤下时按剩余生命掉 1–2 个 kelp（`loot="kelp"`、`maxloots=2`）。[漂流物](../../../raw/current/scripts/prefabs/oceanfishableflotsam.lua#L18-L21)、[海獭窝](../../../raw/current/scripts/prefabs/otterden.lua#L32-L41)、[炮塔](../../../raw/current/scripts/prefabs/crabking_cannontower.lua#L29-L38)、[回收规则](../../../raw/current/scripts/prefabs/boat_bumpers.lua#L181-L187)、[kelp 条目](../../../raw/current/scripts/prefabs/boat_bumpers.lua#L342-L353)

入栏走 `pickable:SpawnProductLoot` 的单产物分支：继承世界湿度后交给 `inventory:GiveItem`，无库存时直接落在植株处；`Pick` 随后置 `canbepicked=false` 并起再生任务，海带丛未设 `remove_when_picked`，所以是原地再生而非换株。[单产物](../../../raw/current/scripts/components/pickable.lua#L493-L513)、[再生](../../../raw/current/scripts/components/pickable.lua#L559-L573)

被炮弹、三叉戟、投石机等击毁时同样损失植株并生成 root 茎；叶只在该路径自己的采摘条件成立时额外掉落（三叉戟、投石机等分支先查 `pickable:CanBePicked()` 再 `SpawnPrefab` 叶）；各伤害源入口与证据归 [[reference/prefabs/bullkelp_root#获取途径]]，本页未逐个复核。

## 干制链

`dryable` 是生 `kelp` 与晾晒架之间的唯一协议：`fn` 把 `product` 设成 `kelp_dried`、`buildfile` 设成 `meat_rack_food_tot`、`drytime` 设成 `dryable.time`，产物数值由 `fn_dried` 读同一张子表。[接线](../../../raw/current/scripts/prefabs/veggies.lua#L632-L637)、[取值](../../../raw/current/scripts/prefabs/veggies.lua#L806-L815)

| `dryable` 字段 | `kelp` 的值 | 消费位置 |
|---|---|---|
| `build` | `"meat_rack_food_tot"` | 架上显示 build（`GetBuildFile`）与 `kelp_dried` 的 bank／build |
| `product` | `"kelp_dried"`（由 `SetProduct(name.."_dried")` 拼出） | 完成时 `SpawnPrefab(product)` |
| `time` | `TUNING.DRY_SUPERFAST`＝0.25×480＝120 秒 | `OnGetItem` 起 `DoTaskInTime` |
| `hunger`／`health`／`sanity`／`perish` | `CALORIES_TINY`／`HEALING_TINY`／`SANITY_SMALL`／`PERISH_PRESERVED` | `fn_dried` 建 edible／perishable |

1. **入架**：晾晒架容器 `params.meatrack.itemtestfn` 只接受带 `dryable` 标签的件，且 `acceptsstacks=false`；该标签在 `kelp` 的公共端就加上。[过滤](../../../raw/current/scripts/containers.lua#L213-L244)、[标签](../../../raw/current/scripts/prefabs/veggies.lua#L562-L565)
2. **计时**：`OnGetItem` 取 `GetProduct()`＋`GetDryTime()`，正常建 120 秒的一次性任务。架子已暂停时只存 `info.drytime`。晾晒中原料腐坏倍率为 0。无雨免疫时降雨会暂停计时，普通雨恢复常规腐坏，酸雨另加腐坏速率。恢复晾晒后继续倒计时。[取值](../../../raw/current/scripts/components/dryingrack.lua#L236-L274)、[零倍率](../../../raw/current/scripts/components/dryingrack.lua#L26-L38)、[暂停](../../../raw/current/scripts/components/dryingrack.lua#L78-L84)、[恢复](../../../raw/current/scripts/components/dryingrack.lua#L347-L374)
3. **完成**：`OnDoneDrying` 生成 `kelp_dried`、只继承湿度、移除原料并在同槽交付产物；**不复制原料剩余新鲜度**，正常即时完成的产物满鲜；`LongUpdate` 推进时超出的秒数会施加给成品，烧毁则走 `InstantDry` 立即干成。[完成](../../../raw/current/scripts/components/dryingrack.lua#L198-L224)、[烧毁](../../../raw/current/scripts/components/dryingrack.lua#L376-L390)、[离线推进](../../../raw/current/scripts/components/dryingrack.lua#L393-L417)
4. **中途取出**：把原料拿离架子会把剩余干制秒数记到 `dryingrack_drytime`，再入架按 `min(max(10, 剩余), 干制时间)` 续计；落地或堆叠变化会清掉该字段，只剩重新计满。[清除](../../../raw/current/scripts/components/dryingrack.lua#L226-L229)、[入架](../../../raw/current/scripts/components/dryingrack.lua#L236-L261)、[取出](../../../raw/current/scripts/components/dryingrack.lua#L280-L288)
5. **盐槽副产**：`veggies.lua` 的干制工厂只对 `kelp` 追加 `driedsalticon` 并开 `collects`；在珍珠多格架 `meatrack_hermit_multi` 上干成后该槽登记一份待收盐，图标与取盐时机见 [[reference/components/driedsalticon]]、[[reference/components/dryingracksaltcollector]]。[挂载](../../../raw/current/scripts/prefabs/veggies.lua#L793-L796)、[宿主](../../../raw/current/scripts/prefabs/meatrack2.lua#L395-L411)、[登记](../../../raw/current/scripts/components/dryingracksaltcollector.lua#L12-L22)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：edible | 三态各写 `healthvalue`／`hungervalue`／`sanityvalue`，食性 `FOODTYPE.VEGGIE`；赋值回调在任一为负时打 `badfood` | 主模拟；生熟带 tag，干叶不带 | [[reference/components/edible#关键接口]]；[生](../../../raw/current/scripts/prefabs/veggies.lua#L595-L600)、[熟](../../../raw/current/scripts/prefabs/veggies.lua#L745-L750)、[干](../../../raw/current/scripts/prefabs/veggies.lua#L811-L815)、[食性](../../../raw/current/scripts/constants.lua#L2026-L2031) |
| 组件：perishable | `SetPerishTime`＋`StartPerishing`，腐坏替换 `spoiled_food` | 生／熟 4800 秒、干 9600 秒；主模拟 | [[reference/components/perishable#关键接口]]；[生](../../../raw/current/scripts/prefabs/veggies.lua#L602-L605)、[熟](../../../raw/current/scripts/prefabs/veggies.lua#L740-L743)、[干](../../../raw/current/scripts/prefabs/veggies.lua#L806-L809) |
| 组件：dryable | 仅生件：`SetProduct`／`SetBuildFile`／`SetDryTime`，并供容器按 `dryable` 标签筛选 | 主模拟；客户端靠公共端标签 | [[reference/components/dryable#关键接口]]；[接线](../../../raw/current/scripts/prefabs/veggies.lua#L632-L637)、[字段](../../../raw/current/scripts/components/dryable.lua#L16-L37) |
| 组件：stackable | 三态都显式 40，组件默认 20 不生效 | 主模拟；上限同步 replica | [[reference/components/stackable#关键接口]]；[生](../../../raw/current/scripts/prefabs/veggies.lua#L613-L619)、[熟](../../../raw/current/scripts/prefabs/veggies.lua#L752-L753)、[干](../../../raw/current/scripts/prefabs/veggies.lua#L817-L818) |
| 组件：repairer | 仅生件：`repairmaterial=MATERIALS.KELP`、`healthrepairvalue=REPAIR_KELP_HEALTH`＝10；每份材料 10 点，只对 KELP 材质且可受理的目标生效（草筏 `boat_otterden`、保险杠 `boat_bumper_kelp`），普通木船不吃 | 主模拟；目标须 `repairable_KELP` 且 `healthrepairable`（如 `boat_otterden`、`boat_bumper_kelp`） | [[reference/components/repairer#关键接口]]、[[reference/components/repairable#恢复分支与顺序]]；[接线](../../../raw/current/scripts/prefabs/veggies.lua#L626-L630)、[结算](../../../raw/current/scripts/components/repairable.lua#L106-L146)、[材料](../../../raw/current/scripts/constants.lua#L1940-L1960)、[船](../../../raw/current/scripts/prefabs/boat.lua#L1217-L1217)、[防撞条](../../../raw/current/scripts/prefabs/boat_bumpers.lua#L264-L267) |
| 组件：cookable | 仅生件：`product="kelp_cooked"`；熟／干不挂，不能互转 | 主模拟；客户端靠 `cookable` 标签识别 | [[reference/components/cookable#关键接口]]；[接线](../../../raw/current/scripts/prefabs/veggies.lua#L664-L665) |
| 组件：driedsalticon | 仅干件：`collects=true`，晾架侧据此登记盐槽并显示盐图标 | 双端 `net_bool`；服务端开关 | [[reference/components/driedsalticon#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/veggies.lua#L793-L796)、[消费](../../../raw/current/scripts/prefabs/meatrack2.lua#L395-L411)、[接口](../../../raw/current/scripts/components/driedsalticon.lua#L50-L55) |
| 组件：burnable／propagator／hauntable／bait／tradable | 三态同套标准工厂：`MakeSmallBurnable`＋`MakeSmallPropagator`、`MakeHauntableLaunchAndPerish`、`bait`、`tradable` | 主模拟；可燃可蔓延、作祟弹飞 | [[reference/components/burnable#关键接口]]、[[reference/components/hauntable#关键接口]]；[生](../../../raw/current/scripts/prefabs/veggies.lua#L653-L660)、[干](../../../raw/current/scripts/prefabs/veggies.lua#L823-L830) |
| API／数据：厨锅食材 | `AddIngredientValues({"kelp"},{veggie=.5},true,true)` 同时登记 `kelp_cooked`（`precook=1`）与 `kelp_dried`（`dried=1`）；`unagi`／`californiaroll` 的 test 都把三名当同类原料 | 食谱判定时读 | [登记](../../../raw/current/scripts/cooking.lua#L46-L72)、[海带行](../../../raw/current/scripts/cooking.lua#L125-L125)、[用法一](../../../raw/current/scripts/preparedfoods.lua#L339-L341)、[用法二](../../../raw/current/scripts/preparedfoods.lua#L631-L644) |
| API／数据：鱼人供奉锅 | 容器过滤只收 `item.prefab == "kelp"`（熟／干都进不去） | 服务端容器 | [过滤](../../../raw/current/scripts/containers.lua#L865-L867) |
| 关联 Prefab：`bullkelp_plant`（Wiki 待建） | `pickable:SetUp("kelp", …)` 的产品与再生宿主；被毁／冲岸时生成茎，满足各路径的采摘条件才额外掉叶 | 海面；基础再生 3 天 | [采集](../../../raw/current/scripts/prefabs/bullkelp_plant.lua#L104-L109)、[冲岸](../../../raw/current/scripts/prefabs/bullkelp_plant.lua#L42-L54)、[注册](../../../raw/current/scripts/prefabs/bullkelp_plant.lua#L150-L151) |
| 关联 Prefab：`bullkelp_beachedroot` | 拾取／入包时替换为 kelp＋`bullkelp_root` 各 1，继承湿度 | 冲岸 | [替换](../../../raw/current/scripts/prefabs/bullkelp_beached.lua#L15-L35)、[注册名](../../../raw/current/scripts/prefabs/bullkelp_beached.lua#L88-L88) |
| 关联 Prefab：`meatrack`／`meatrack_hermit_multi` | 干制宿主（官方名「晾肉架」／「晾肉巨架」）：container＋`dryingrack`，三格／九格；后者带盐槽 | 结构；细节见 [[reference/components/dryingrack]] | [装配](../../../raw/current/scripts/prefabs/meatrack2.lua#L322-L334)、[注册](../../../raw/current/scripts/prefabs/meatrack2.lua#L584-L592) |
| 关联 Prefab：`boatpatch_kelp` | 配方原料 kelp×3；自身是 KELP 材质的临时补漏 | 玩家合成 | [[reference/prefabs/boat-patches]]；[配方](../../../raw/current/scripts/recipes.lua#L612-L612) |
| 资源：anim | `anim/kelp.zip`（`idle`／`cooked`）、`anim/meat_rack_food_tot.zip`（`dried_kelp`） | 静态声明 | [生熟资源](../../../raw/current/scripts/prefabs/veggies.lua#L392-L411)、[干态资源](../../../raw/current/scripts/prefabs/veggies.lua#L433-L437) |

浮水由 `MakeInventoryFloatable` 消费，三态参数见[三态数值](#三态数值)；接口见 [[reference/components/floater#关键接口]]，[helper](../../../raw/current/scripts/standardcomponents.lua#L1772-L1791)。

## 典型接入

### 三态调参：去掉生熟负收益并延长干叶保质期

- 前提与选择：服务端 `modmain.lua`；三件是独立注册，必须分别 postinit。`edible.healthvalue`／`sanityvalue` 是属性 setter，赋值即重算 `badfood`；`SetNewMaxPerishTime` 保留当前百分比，构造后调用时物品仍满鲜。[setter](../../../raw/current/scripts/components/edible.lua#L50-L53)、[上限](../../../raw/current/scripts/components/perishable.lua#L204-L208)
- 生效与撤销：postinit 在每次构造时运行；停用模组恢复工厂值。已生成实例与已恢复的生命／理智不倒回；读档时 `Perishable:OnLoad` 照抄存档剩余秒数。[读档](../../../raw/current/scripts/components/perishable.lua#L337-L349)
- 原版依据：三态原值见[三态数值](#三态数值)；干叶的 20 天来自 `dryable.perish`。

```lua
-- modmain.lua：生熟海带叶不再扣生命／理智，干海带叶保质期 20 → 40 天
local function retune(prefab, health, sanity)
    AddPrefabPostInit(prefab, function(inst)
        if not GLOBAL.TheWorld.ismastersim then
            return
        end
        local edible = inst.components.edible
        if edible ~= nil then
            edible.healthvalue = health
            edible.sanityvalue = sanity
        end
    end)
end

retune("kelp", 0, 0)
retune("kelp_cooked", 0, 0)

AddPrefabPostInit("kelp_dried", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    inst.components.perishable:SetNewMaxPerishTime(GLOBAL.TUNING.PERISH_SUPERSLOW)
end)
```

- 调整与验证：`health`／`sanity` 是点数，不是比例；`PERISH_SUPERSLOW`＝40×480＝19200 秒。示例不新增 prefab、不改 `TUNING` 表；干叶原为 20 天；`SetNewMaxPerishTime` 只对新建的满鲜实例等价于直接改成 40 天，读档实例仍被存档剩余秒数覆盖，旧干叶不会因读档重获 40 天。未运行游戏。

## 常见陷阱与限制

- **误区：熟海带叶已经"去毒"**：`kelp_cooked` 只把生命从 −1 改成 0，理智仍 −5，照样带 `badfood`；只有 `kelp_dried` 是正的。[数值](../../../raw/current/scripts/prefabs/veggies.lua#L118-L121)、[回调](../../../raw/current/scripts/components/edible.lua#L1-L7)
- **边界：干制会"刷新"新鲜度**：正常即时完成只 `InheritMoisture`，不复制原料剩余百分比，快坏的 kelp 干完是满鲜干叶；与 `cookable` 按损失结转的规则不同，见 [[reference/components/cookable#关键接口]]。[完成](../../../raw/current/scripts/components/dryingrack.lua#L198-L224)
- **误区：干海带叶能再上架**：`fn_dried` 不挂 `dryable`，成品没有标签，容器过滤直接拒绝；熟叶同理。[过滤](../../../raw/current/scripts/containers.lua#L234-L244)
- **误区：晾晒架就是 `components/dryer.lua`**：当前 `meatrack` 家族装的是 container＋`dryingrack`；`dryer` 是旧单件路径，本快照 scripts 全量检索 `AddComponent("dryer")` 零命中，`meatrack2` 只在读档时接管旧 `data.dryer` 字段，燃烧时还会移除该旧组件。[旧档](../../../raw/current/scripts/prefabs/meatrack2.lua#L176-L205)、[移除](../../../raw/current/scripts/standardcomponents.lua#L85-L87)、[旧组件](../../../raw/current/scripts/components/dryer.lua#L176-L218)、[[reference/components/dryer#关键接口]]
- **边界：雨天不计时**：无雨免疫时 `dryingrack` 会暂停，120 秒只是可推进的游戏秒数；暂停期间不再给零腐败倍率，酸雨还会加速腐坏；同一件离架再入架按剩余秒数续计（仅当原料仍保有 `dryingrack_drytime`）。[暂停](../../../raw/current/scripts/components/dryingrack.lua#L78-L84)、[续计](../../../raw/current/scripts/components/dryingrack.lua#L236-L261)
- **边界：沃特／鱼人吃生熟海带不扣负收益**：`kelp`／`kelp_cooked` 的 prefab 偏好让 `Eater:DoFoodEffects` 返回 false，负生命／理智整段跳过；沃特另有 1.33 倍饥饿，`badfood` 标签仍留在物品上。偏好机制见 [[reference/components/foodaffinity#原版数值为-1-的偏好有什么用途]]，[数值](../../../raw/current/scripts/prefabs/wurt.lua#L647-L651)、[鱼人](../../../raw/current/scripts/prefabs/merm.lua#L1104-L1110)、[结算](../../../raw/current/scripts/components/eater.lua#L166-L170)
- **未核验**：quagmire 模式对所有蔬菜调 `event_server_data("quagmire", "prefabs/veggies").master_postinit(inst)`，该节日数据不在本快照 scripts 根内，海带是否有额外行为未核验。[钩子](../../../raw/current/scripts/prefabs/veggies.lua#L683-L685)

## 验证记录

- 源码审阅：2026-11-08，Steam buildid 26335394（`raw/current/BUILD.txt` 已核）。核对 `veggies.lua` 的 `VEGGIES.kelp`／`SEEDLESS`／导出段／`fn`／`fn_cooked`／`fn_dried`，`bullkelp_plant` 的 pickable 与冲岸、`bullkelp_beached` 的替换，`meatrack2` 的容器装配与旧档分支，`dryingrack`／`dryable` 的入架、暂停、完成与端分界，`perishable`／`edible`／`stackable`／`repairable`／`pickable`／`eater` 的相关分支，`tuning` 的相关常量与时间基准，`containers` 的架子过滤与供奉锅，`cooking`／`preparedfoods` 的食材登记，`scrapbookdata` 的三行，`wurt`／`merm` 的偏好，以及简中三个名称键。
- 名称核验：`STRINGS.NAMES.KELP`＝海带叶、`KELP_COOKED`＝熟海带叶、`KELP_DRIED`＝干海带叶；显示名按 prefab 名大写查 `STRINGS.NAMES`。[KELP](../../../raw/current/scripts/languages/chinese_s.po#L344184-L344187)、[KELP_COOKED](../../../raw/current/scripts/languages/chinese_s.po#L344194-L344197)、[KELP_DRIED](../../../raw/current/scripts/languages/chinese_s.po#L344199-L344202)、[查名](../../../raw/current/scripts/entityscript.lua#L840-L844)
- 未覆盖：海带丛被炮弹／三叉戟／投石机击毁的逐个伤害源只引用 [[reference/prefabs/bullkelp_root#获取途径]]，未逐文件复核；其它干制物的对照未展开；quagmire 节日数据不在快照内。
- 游戏运行：未运行。三态数值、120 秒干制与雨天暂停均为源码推导，非实测。
- 结构检查：`validate_page.py` 通过（standard 档内）；`validate_source_links.py --strict` 0 错 0 警；页内 Lua 示例经 `parser_probe.exe --syntax-only` 通过。
