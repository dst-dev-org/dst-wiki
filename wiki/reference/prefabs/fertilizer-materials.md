---
title: "基础肥料四件（堆肥 compost、鸟粪 guano、粪肥 poop、嗡嗡肥料 mosquitofertilizer）"
created: "2026-11-10"
updated: "2026-11-10"
type: prefab
summary: "四个注册名不同、组装壳几乎相同的可堆叠肥料：同一套部署施肥／扑灭阴燃／燃料／作祟接线，差异集中在养分与旧农田参数、鸟粪的存档缩放与便便云落点（quagmire 下 guano／poop 摘掉燃料取用回调但燃料值不变），以及嗡嗡肥料的制作／使用双技能门槛（二阶可做、三阶部署才各再 +8）。"
aliases: ["compost", "guano", "poop", "mosquitofertilizer", "mosquitofertalizer", "堆肥", "鸟粪", "粪肥", "嗡嗡肥料", "便便", "施肥"]
tags: [prefab, item]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/compost.lua", "prefabs/guano.lua", "prefabs/poop.lua", "prefabs/mosquitofertalizer.lua", "components/fertilizer.lua", "components/fertilizerresearchable.lua", "components/smotherer.lua", "components/deployable.lua", "components/fuel.lua", "components/fueled.lua", "components/hauntable.lua", "components/savedscale.lua", "components/farming_manager.lua", "components/pickable.lua", "components/grower.lua", "components/crop.lua", "components/stackable.lua", "standardcomponents.lua", "actions.lua", "componentactions.lua", "prefabs/poopcloud.lua", "prefabs/fertilizer_nutrient_defs.lua", "prefabs/compostingbin.lua", "prefabs/flies.lua", "prefabs/gridplacer.lua", "prefabs/bat.lua", "prefabs/bat_boss.lua", "prefabs/molebat.lua", "prefabs/beefalo.lua", "prefabs/babybeefalo.lua", "prefabs/koalefant.lua", "prefabs/pigman.lua", "prefabs/monkey.lua", "prefabs/grassgator.lua", "prefabs/leafpile.lua", "prefabs/winter_tree.lua", "prefabs/ipecacsyrup.lua", "prefabs/birdcage.lua", "prefabs/ash.lua", "prefabs/wormwood.lua", "plantregistrydata.lua", "recipes.lua", "tuning.lua", "skins_defs_data.lua", "prefabs/player_common.lua", "map/boons.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 基础肥料四件（堆肥 compost、鸟粪 guano、粪肥 poop、嗡嗡肥料 mosquitofertilizer）

## 快速判断

- 职责与适用：四个可堆叠肥料物品，工厂体几乎同一份——两端先建成可漂浮物品并加 `deployable`／`tile_deploy`／`fertilizerresearchable` 标签，主模拟再装 fertilizer、smotherer、fuel、burnable、deployable、hauntable。四工厂：[compost](../../../raw/current/scripts/prefabs/compost.lua#L34-L92)、[guano](../../../raw/current/scripts/prefabs/guano.lua#L49-L124)、[poop](../../../raw/current/scripts/prefabs/poop.lua#L56-L126)、[mosquitofertilizer](../../../raw/current/scripts/prefabs/mosquitofertalizer.lua#L78-L138)。
- 优先使用：查养分与成功使用后的消耗读 [[reference/components/fertilizer]]；扑灭阴燃读 [[reference/components/smotherer]]；便便云表现（网络壳与本地代理）读 [[reference/prefabs/damp_trail]]；用同一套部署 helper 的催长物品读 [[reference/prefabs/treegrowthsolution]]。
- 重要副作用：地面部署只认农田土壤格；作为燃料被取用时在**用火处**生成 `poopcloud`（火焰 FX 缺失或无效时退回取用者坐标；quagmire 下 guano／poop 不再生成，但燃料值不变）；作祟只弹飞、不点燃（`#HAUNTFIX` 注释掉附加段）。
- 修改边界：本页覆盖四个注册的工厂、组装与直接消费者；养分落到地块后的消耗与压力归 [[reference/components/farming-manager]] 与作物／植株页；生成者只核验到下表的触发条件，未穷举全部持有 `"poop"`／`"guano"` 字面量的文件。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `compost`（堆肥） | `fn` 直接返回 `Prefab("compost", …)`：Transform／AnimState／SoundEmitter／Network → `MakeInventoryPhysics` → bank+build `compost` 播 `idle` → `MakeInventoryFloatable("med", -0.08, 0.9)` → `MakeDeployableFertilizerPristine` → 加 `fertilizerresearchable` 标签与 `GetFertilizerKey` → `SetPristine` → 客户端早退 → 主模拟 push `idle`，再装 inspectable／inventoryitem／stackable／fertilizerresearchable／fertilizer／smotherer／fuel／burnable+propagator／deployable／hauntable | 唯一同时满足三项的一件：没有 `flies` 子实体、没有 `savedscale`、`FuelTaken` 也没有 quagmire 短路（`savedscale` 只有 guano 有、quagmire 短路只有 guano／poop 有、`flies` 只有 compost 从不产生，三项合起来才唯一）；堆叠上限未显式设 `maxsize`，走 stackable 默认 20。[工厂](../../../raw/current/scripts/prefabs/compost.lua#L34-L90)、[取用回调](../../../raw/current/scripts/prefabs/compost.lua#L15-L24) |
| `guano`（鸟粪） | 同一壳：bank+build `guano`，`dump` 后 push `idle`，`MakeInventoryFloatable("med", 0.1, 0.73)`；主模拟另装 `flies` 子实体、`inventoryitem` 的 dropped／putin 回调与 `savedscale` | 便便云不抬 `y`（[taker 世界坐标](../../../raw/current/scripts/prefabs/guano.lua#L24-L26)）；quagmire 模式把 `fuel.ontaken` 置 nil；`burnable` 的 `SetOnIgniteFn(OnBurn)` 烧起时连苍蝇一起删。[工厂](../../../raw/current/scripts/prefabs/guano.lua#L49-L122) |
| `poop`（粪肥） | 同 guano 的骨架，bank+build `poop`；主模拟同样装 `flies` 与 inventoryitem 回调 | 少了 `savedscale`，其余与 guano 同构；便便云抬 `y+1`。[工厂](../../../raw/current/scripts/prefabs/poop.lua#L56-L124)、[取用回调](../../../raw/current/scripts/prefabs/poop.lua#L24-L33) |
| `mosquitofertilizer`（嗡嗡肥料） | 文件名拼作 `mosquitofertalizer.lua`，注册名、bank／build 与 `FERTILIZER_DEFS` 键都是 `mosquitofertilizer`；`MakeInventoryFloatable("small", 0.2, 0.8)`；主模拟先 smotherer 再 stackable，末尾写 `inst.ondeployed_fertilzier_extra_fn` | 唯一有部署后追加养分回调的一件：基础三养分 12／12／12，配方只要沃特 `wurt_mosquito_craft_2`，部署时仅当 deployer 点亮 `wurt_mosquito_craft_3` 才各再加 `WURT_BONUS_FERT`＝8（同样受 0–100 夹）；四项旧参数直接复用 `POOP_*`，没有自己的 `MOSQUITOFERTILIZER_FERTILIZE`。[工厂](../../../raw/current/scripts/prefabs/mosquitofertalizer.lua#L78-L136)、[部署加成](../../../raw/current/scripts/prefabs/mosquitofertalizer.lua#L68-L74) |

名称：四条 `STRINGS.NAMES.*` 键在简中实显为 **堆肥**（`COMPOST`）、**鸟粪**（`GUANO`）、**粪肥**（`POOP`，英文 msgid 是 "Manure"）、**嗡嗡肥料**（`MOSQUITOFERTILIZER`，英文 msgid 是 "Fertilizzzer"）。[compost](../../../raw/current/scripts/languages/chinese_s.po#L341610-L341612)、[guano](../../../raw/current/scripts/languages/chinese_s.po#L343485-L343487)、[poop](../../../raw/current/scripts/languages/chinese_s.po#L347175-L347177)、[mosquitofertilizer](../../../raw/current/scripts/languages/chinese_s.po#L345930-L345932)。"便便"不是本 build 的实显名，只作检索俗称留在 aliases。

### 数值（TUNING 原值）

| 注册名 | `fertilizervalue`（秒） | `soil_cycles` | `withered_cycles` | `nutrients`（催长素／堆肥／粪肥） |
|---|---:|---:|---:|---|
| `compost` | 300 | 10 | 1 | 0／24／0 |
| `guano` | 450 | 12 | 1 | 0／0／16 |
| `poop` | 300 | 10 | 1 | 0／0／8 |
| `mosquitofertilizer` | 300（`POOP_*`） | 10 | 1 | 12／12／12 |

[时间与轮次](../../../raw/current/scripts/tuning.lua#L1618-L1624)、[堆肥](../../../raw/current/scripts/tuning.lua#L5850-L5852)、[养分表](../../../raw/current/scripts/tuning.lua#L5858-L5876)、[`day_time`＝300 秒](../../../raw/current/scripts/tuning.lua#L19-L27)。`fertilizervalue` 只供旧作物路径按 `crop.rate` 折算成长百分比，单次夹到 1；`soil_cycles` 与 `withered_cycles` 分别供旧 `grower` 加轮次、`pickable` 加抗枯轮次，都不是“现代农田养分倍率”。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：fertilizer | 提供 `nutrients`、三个旧参数、`fertilize_sound` 与 `OnApplied`（四件都没有 `finiteuses`，`final_use` 恒为 true → 每次消耗 1 件：`stacksize>1` 时 `Get()` 拆出 1 件再删、净减 1，`==1` 时整件消失）；构造时加 `fertilizer` 标签 | 服务端；四件都已 `AddComponent` | [[reference/components/fertilizer]]；[组件](../../../raw/current/scripts/components/fertilizer.lua#L2-L48)、[拆堆](../../../raw/current/scripts/components/stackable.lua#L109-L153)、[数值写入](../../../raw/current/scripts/prefabs/poop.lua#L95-L99) |
| 组件：deployable（＋两端 pristine helper） | `MakeDeployableFertilizerPristine` 在两端设 `_custom_candeploy_fn`、`overridedeployplacername = "gridplacer_farmablesoil"`，并加 `deployable`／`tile_deploy` 标签；主模拟再装组件：模式 `CUSTOM`、`SetUseGridPlacer(false)`、`keep_in_inventory_on_deploy = true` | 部署判定 `TheWorld.Map:IsFarmableSoilAtPoint`；落地后先加三养分再走 `OnApplied`，因此一件只消耗一份、且不从物品栏先摘出 | [pristine](../../../raw/current/scripts/standardcomponents.lua#L1816-L1821)、[组件与结算](../../../raw/current/scripts/standardcomponents.lua#L1797-L1831)、[DEPLOY 结算](../../../raw/current/scripts/actions.lua#L1607-L1641)、[[reference/components/deployable]] |
| API／数据：DEPLOY 动作名 | 物品带 `fertilizer` 标签时 `ACTIONS.DEPLOY.strfn` 返回 `"FERTILIZE_GROUND"` | 客户端显示；标签由 fertilizer 组件构造时加 | [strfn](../../../raw/current/scripts/actions.lua#L1657) |
| 组件：fertilizerresearchable | 两端标签 ＋ 主模拟组件，`SetResearchFn` 指到 `GetFertilizerKey`（返回 `inst.prefab`） | 研究候选要求戴 `plantinspector`，已知与否由 `ThePlantRegistry:KnowsFertilizer(key)` 判；这些键另在 `skins_defs_data.lua` 的 `TheInventory:AddPlantRegistryKey` 调用列表内（实现不在 `scripts/`，登记作用未核验） | [[reference/components/fertilizerresearchable]]；[组件](../../../raw/current/scripts/components/fertilizerresearchable.lua#L1-L22)、[键登记](../../../raw/current/scripts/skins_defs_data.lua#L5275-L5291)、[已知判定](../../../raw/current/scripts/plantregistrydata.lua#L64) |
| 组件：smotherer | 空标记组件，让物品成为 `SMOTHER`（目标带 `smolder`）与 `MANUALEXTINGUISH`（物品须 `frozen`）的动作提供者 | 服务端添加；执行时消耗 1 份堆叠 | [[reference/components/smotherer]]；[组件](../../../raw/current/scripts/components/smotherer.lua#L1-L6)、[动作提供](../../../raw/current/scripts/componentactions.lua#L1736-L1743) |
| 组件：fuel ＋ `FuelTaken` | `fuelvalue = MED_FUEL`（45 秒）；`SetOnTakenFn` 在该物品被 fueled 实体当燃料取用时生成 `poopcloud`。compost／poop／mosquito 取 `burnable.fxchildren[1]`，**该 FX 为空或 `IsValid()` 为假时退回取用者坐标**再抬 `y+1`；guano 直接取用者坐标、不抬 `y` | 服务端；`Fueled:TakeFuelItem` 先结算燃料值、调 `Taken`，随后才 `item:Remove()`。`quagmire` 下 guano／poop 把该回调置 nil，**但 `fuelvalue` 不变、仍能当燃料**，只是不冒 cloud；compost／mosquito 不摘 | [[reference/components/fuel]]；[fuel.Taken](../../../raw/current/scripts/components/fuel.lua#L31-L36)、[取用处](../../../raw/current/scripts/components/fueled.lua#L197-L228)、[compost](../../../raw/current/scripts/prefabs/compost.lua#L15-L24)、[poop](../../../raw/current/scripts/prefabs/poop.lua#L24-L33)、[蚊](../../../raw/current/scripts/prefabs/mosquitofertalizer.lua#L29-L41)、[quagmire 摘回调](../../../raw/current/scripts/prefabs/poop.lua#L112-L114) |
| 组件：burnable ＋ propagator | `MakeSmallBurnable(inst, MED_BURNTIME)`（15 秒、FX 级别 2）把 `DefaultBurnFn` 挂 OnIgniteFn、`DefaultBurntFn` 挂 OnBurntFn，另加 `MakeSmallPropagator` | 服务端。**点燃**：guano／poop／mosquito 把 OnIgniteFn 换成自身 `OnBurn`（先 `DefaultBurnFn` 再删 `flies`），`DefaultBurnFn` 只把非 tree／structure 的 `persists` 置 false。**烧尽**：`DefaultBurntFn` 移除 growable、清 `inventoryitemdata`，陆地上生成 1 件 `ash`（stack＝min(40, 原 stacksize)；本批上限 20，故整叠转成 1 件同数量灰）后 `Remove` 自身，海面上不产灰直接删 | [[reference/components/burnable]]；[MakeSmallBurnable](../../../raw/current/scripts/standardcomponents.lua#L164-L174)、[DefaultBurnFn](../../../raw/current/scripts/standardcomponents.lua#L12-L16)、[DefaultBurntFn](../../../raw/current/scripts/standardcomponents.lua#L18-L44)、[ash 上限 40](../../../raw/current/scripts/prefabs/ash.lua#L69-L70)、[burntime](../../../raw/current/scripts/tuning.lua#L1682) |
| 组件：hauntable | `MakeHauntableLaunchAndIgnite`：`Launch` 一定执行（默认 `HAUNT_CHANCE_ALWAYS`），点燃段与末尾 `return true` 都被 `#HAUNTFIX` 注释，函数实际返回 false | 服务端；发射型 helper（`MakeHauntableLaunchAndIgnite`／`MakeHauntableLaunchAndSmash`）都在弹飞前先置 `hauntvalue = HAUNT_TINY`（1），本页四件走前者。返回 false 时 `Hauntable:DoHaunt` 仍进 `else` 分支设 `haunted` 与冷却，只是不 push `"haunted"`、也不清空已置的 1（清空只发生在成功且 doer 非空分支） | [[reference/components/hauntable]]；[helper](../../../raw/current/scripts/standardcomponents.lua#L1191-L1217)、[失败分支](../../../raw/current/scripts/components/hauntable.lua#L82-L115) |
| 组件：stackable／inventoryitem／floater | 四件都只用默认 `STACK_SIZE_MEDITEM`（20）；guano／poop／mosquito 走 `flies` 子实体与“入包删苍蝇”回调；guano 另用 `savedscale` | 服务端添加，floater 在工厂两端设尺寸与偏移 | [[reference/components/stackable]]；[默认上限](../../../raw/current/scripts/components/stackable.lua#L23-L34)、[缩放存档](../../../raw/current/scripts/components/savedscale.lua#L5-L19) |
| API／数据：`FERTILIZER_DEFS` | 四件都从 `TUNING.*_NUTRIENTS` 取三元组交给 `SetNutrients`；同文件的 `SORTED_FERTILIZERS` 决定图鉴排序 | 服务端读取；键名必须与注册名一致 | [定义](../../../raw/current/scripts/prefabs/fertilizer_nutrient_defs.lua#L1-L20)、[排序](../../../raw/current/scripts/prefabs/fertilizer_nutrient_defs.lua#L54-L77) |
| 关联 Prefab：`flies` | guano／poop 在工厂里就建一枚（[guano:100](../../../raw/current/scripts/prefabs/guano.lua#L100)、[poop:106](../../../raw/current/scripts/prefabs/poop.lua#L106)），mosquitofertilizer 只在落回地面时建（[mosquitofertalizer:45-49](../../../raw/current/scripts/prefabs/mosquitofertalizer.lua#L45-L49)）；入包／烧起时删 | 主模拟；读档时 guano／poop 由工厂重建，嗡嗡肥料不会（它的 `flies` 只挂在 `inventoryitem` 的 OnDropped 上，读档不走该路径）。纯 FX（`NOCLICK`／`DECOR`＋`playerprox`） | [flies](../../../raw/current/scripts/prefabs/flies.lua#L38-L72) |
| 关联 Prefab：`poopcloud` | 燃料取用时的表现，网络壳 1 秒自删、本地代理 `animover` 自删 | 详见 damp_trail 页，不在此复制 | [[reference/prefabs/damp_trail]]；[工厂](../../../raw/current/scripts/prefabs/poopcloud.lua#L27-L52) |
| 关联 Prefab：`compostingbin`／`gridplacer_farmablesoil` | 前者是 compost 的唯一原版来源；后者是部署预览，只在农田土壤格显示图层 | 见下节与陷阱 | [产物](../../../raw/current/scripts/prefabs/compostingbin.lua#L264-L272)、[预览判定](../../../raw/current/scripts/prefabs/gridplacer.lua#L153-L160) |

### 获得与生成

| 生成／获得点 | 产物 | 触发条件 | 证据 |
|---|---|---|---|
| `compostingbin` 周期产物 | `compost` | 每完成一个周期 `numtoharvest + 1`，桶上最多积 4 份，超出时直接 `FlingItem` 到地上；锤碎先把已积攒的抛出来 | [onfinishcycle](../../../raw/current/scripts/prefabs/compostingbin.lua#L264-L272)、[上限](../../../raw/current/scripts/prefabs/compostingbin.lua#L47)、[产物注册](../../../raw/current/scripts/prefabs/compostingbin.lua#L398) |
| 放牧／群居生物的 `periodicspawner` | `poop` | 皮弗娄牛 40–60 秒，死亡／不可套绳／正被骑乘时不生成，落点在身后 1 单位、`y+0.8`；小牛 80–110 秒；考拉象 40–60 秒；猴子 200–400 秒、最小间隔 15 | [牛](../../../raw/current/scripts/prefabs/beefalo.lua#L1187-L1194)、[牛的门禁](../../../raw/current/scripts/prefabs/beefalo.lua#L971-L981)、[落点](../../../raw/current/scripts/prefabs/beefalo.lua#L1038-L1052)、[小牛](../../../raw/current/scripts/prefabs/babybeefalo.lua#L165-L170)、[考拉象](../../../raw/current/scripts/prefabs/koalefant.lua#L124-L129)、[猴子](../../../raw/current/scripts/prefabs/monkey.lua#L466-L471) |
| 猪人进食 | `poop` | 吃到 `FOODTYPE.VEGGIE` 时在自己位置直接生成（无概率、无冷却） | [OnEat](../../../raw/current/scripts/prefabs/pigman.lua#L136-L147) |
| 蝙蝠／巨型蝙蝠的 `periodicspawner` | `guano` | 120–240 秒、密度 30 内 2、最小间隔 8 | [bat](../../../raw/current/scripts/prefabs/bat.lua#L292-L297)、[bat_boss](../../../raw/current/scripts/prefabs/bat_boss.lua#L581-L586) |
| 生物掉落表 | `guano` | 蝙蝠 `bat` 表 15%、酸液变体 30%；裸鼹蝠 `molebat` 表 15% | [bat 表](../../../raw/current/scripts/prefabs/bat.lua#L22-L34)、[molebat 表](../../../raw/current/scripts/prefabs/molebat.lua#L18-L21) |
| 鸟笼消化后的兜底分支 | `guano` | 投喂先过接受条件：种子／带 `<prefab>_seeds` 的食物／肉，`invalid_foods` 排除；月变异鸟只收月亮碎片（`moonglass_charged` 在辉煌冷却时被拒）。消化按分支取产物：月变异鸟只在 `moonglass_charged` 且 `bird.do_drop_brilliance` 为真时给纯粹辉煌；肉走蛋分支（普通 `bird_egg`／变异鸟 `rottenegg`）；普通变异鸟的非肉分支给 `spoiled_food`；普通鸟看 `Prefabs[<prefab>_seeds]` 是否存在——有（多数作物）就出该种子，没有才以 `FEED_BIRD_POOP_CHANCE`＝0.33 为基值、经 `BirdPoopChanceAdditive` 按喂食者运气调整后抽鸟粪。通用 `seeds` 正是落到这一支的种子（`seeds_seeds` 不存在）；抽中后鸟粪缩放 0.33，并由 `savedscale` 随存档保留 | [接受条件](../../../raw/current/scripts/prefabs/birdcage.lua#L155-L181)、[消化分支](../../../raw/current/scripts/prefabs/birdcage.lua#L106-L147)、[概率](../../../raw/current/scripts/tuning.lua#L9351)、[运气调整](../../../raw/current/scripts/prefabs/birdcage.lua#L100-L104) |
| 世界布点与其它战利品 | `guano`／`poop` | `map/boons.lua` 的奖励表、`winter_tree` 的 `loot`／`burntloot` 返回值里直接写名字；是否在当前世界流程启用、砍／烧链路是否走到未逐条核验 | [boons](../../../raw/current/scripts/map/boons.lua#L108)、[winter_tree](../../../raw/current/scripts/prefabs/winter_tree.lua#L6-L12) |
| 配方与 debuff | `poop`／`mosquitofertilizer` | `transmute_poop` 把 6 个 `spoiled_food` 转化（威尔逊炼金 10 技能）；`mosquitofertilizer`＝蚊囊×1＋硝石×2＋粪肥×1（沃特 `wurt_mosquito_craft_2`）；催吐药 debuff 每 tick 在目标位置生成一枚并扣血 | [炼金](../../../raw/current/scripts/recipes.lua#L1493)、[嗡嗡肥料](../../../raw/current/scripts/recipes.lua#L115)、[催吐](../../../raw/current/scripts/prefabs/ipecacsyrup.lua#L83-L110) |
| 落叶堆不给糖 | `poop` | `TRICK` 分支 5% | [掉落表](../../../raw/current/scripts/prefabs/leafpile.lua#L91-L96) |

### 落到地块之后

部署是三养分**按地块**写：`fertilizer_ondeploy` 用 `GetTileCoordsAtPoint` 取格，交 `farming_manager:AddTileNutrients`，后者对三类分别 `math.clamp(_n + x, 0, 100)`，格满时实际增量小于名义值。[部署结算](../../../raw/current/scripts/standardcomponents.lua#L1797-L1810)、[加养分](../../../raw/current/scripts/components/farming_manager.lua#L394-L398)。旧路径（`ACTIONS.FERTILIZE`）走另一条分支：目标按 `notreadyforharvest`／`fertile`｜`infertile`／`barren`／`fertilizable` 分流，或对沃姆伍德等带 `self_fertilizable` 的目标自肥（`wormwood.lua` 加标签与 `fertilizable` 组件）；只有 `applied` 为真才调 `OnApplied` 扣一份。[动作候选](../../../raw/current/scripts/componentactions.lua#L1241-L1255)、[结算](../../../raw/current/scripts/actions.lua#L1812-L1842)、[沃姆伍德接线](../../../raw/current/scripts/prefabs/wormwood.lua#L755)、[fertilizable](../../../raw/current/scripts/prefabs/wormwood.lua#L845-L846)。旧作物在“冬季且温度 ≤ 0”时 `Crop:Fertilize` 会先 `StopSmoldering()`（若身上有 burnable）再落空返回 nil，因此不加速也不消耗。[crop](../../../raw/current/scripts/components/crop.lua#L84-L110)。

## 典型接入

### 给任意肥料注册加“部署时追加养分”

- 前提与选择：`fertilizer_ondeploy` 在扣掉一份之后，只要 `inst.ondeployed_fertilzier_extra_fn` 非空就调用它（没有其它条件），这是原版留给单个注册的扩展点。原版唯一用户是嗡嗡肥料：基础三养分 12／12／12，`wurt_mosquito_craft_3` 已点亮时再各加 `WURT_BONUS_FERT`＝8；而制作该物品本身只要 `wurt_mosquito_craft_2`——**制作门槛与加成门槛是两个技能**。给别的肥料复制这个钩子即可，不必替换 `deployable.ondeploy`。
- 生效与撤销：写在目标模组 `modmain.lua`，只在主模拟赋值；撤销即删该字段（重开世界或重新 postinit）。已经写进地块的养分不回滚。
- 原版依据：[部署 helper](../../../raw/current/scripts/standardcomponents.lua#L1797-L1810)、[嗡嗡肥料的用法](../../../raw/current/scripts/prefabs/mosquitofertalizer.lua#L68-L74)

```lua
-- modmain.lua：便便部署后在目标格额外给 8 点催长素（8 为示例自定义值；原版三阶沃特是三类各 +8）
local function extra_poop_nutrients(inst, pt, deployer)
    local tile_x, tile_z = GLOBAL.TheWorld.Map:GetTileCoordsAtPoint(pt:Get())
    GLOBAL.TheWorld.components.farming_manager:AddTileNutrients(tile_x, tile_z, 8, 0, 0)
end

AddPrefabPostInit("poop", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    inst.ondeployed_fertilzier_extra_fn = extra_poop_nutrients
end)
```

- 调整与验证：三个数值依序是催长素／堆肥／粪肥，单位是养分点，单格上限 100（额外值同样被夹）。示例给的单类 +8 是**自定义值**，不是原版数值；原版是「基础 12／12／12 ＋ 三阶技能各 +8」。回调在 `fertilizer:OnApplied` 之后执行，不要在这里再扣物品或再调 `OnApplied`。验收可用只点二阶与已点三阶的角色分别在农田部署一次，比对是否出现额外三养分；示例只做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **误区：作祟会点燃或砸碎。** 四件调的都是 `MakeHauntableLaunchAndIgnite`，点燃段与末尾 `return true` 同被 `#HAUNTFIX` 注释，实际只有 `Launch`；又因为返回 false，`DoHaunt` 走的是失败分支——弹飞了却没 push `"haunted"`，弹飞前已置的 `hauntvalue`＝1 也不被清空（清空只发生在成功且 doer 非空）。[helper](../../../raw/current/scripts/standardcomponents.lua#L1191-L1217)、[分支](../../../raw/current/scripts/components/hauntable.lua#L82-L115)
- **边界：部署只在农田土壤。** `fertilizer_candeploy` 只问 `IsFarmableSoilAtPoint`，普通草地／地板／船上都不成立；预览实体也只在这样的格上显示图层。[判定](../../../raw/current/scripts/standardcomponents.lua#L1812-L1814)、[预览](../../../raw/current/scripts/prefabs/gridplacer.lua#L153-L160)
- **边界：便便云落点与燃料例外。** compost／poop／mosquito 取用者（火堆等 fueled 实体）的 `burnable.fxchildren[1]`，该 FX 为空或 `IsValid()` 为假时退回取用者坐标，再抬 `y+1`；guano 直接取用者坐标、不抬 `y`。`quagmire` 下 guano 与 poop 把取用回调置 nil，`fuelvalue` 仍是 `MED_FUEL`、照样能烧，只是不冒 cloud；compost 与 mosquito 不摘。[compost](../../../raw/current/scripts/prefabs/compost.lua#L15-L24)、[guano](../../../raw/current/scripts/prefabs/guano.lua#L24-L26)、[poop](../../../raw/current/scripts/prefabs/poop.lua#L24-L33)、[quagmire](../../../raw/current/scripts/prefabs/poop.lua#L112-L114)
- **边界：堆叠 20 不是工厂写的。** 四件都只调 `AddComponent("stackable")`，上限来自组件默认 `TUNING.STACK_SIZE_MEDITEM`；改上限要在 postinit 设 `maxsize`。[默认值](../../../raw/current/scripts/components/stackable.lua#L23-L34)
- **边界：客户端只有标签，没有组件。** `deployable`／`tile_deploy`／`fertilizerresearchable` 与 `GetFertilizerKey` 都在 `SetPristine` 之前加，客户端能读到这些；但 `components.fertilizer` 等只在主模拟存在，客户端按组件名取用会拿到 nil。
- **边界：研究键另有一份登记表。** 组件只提供键与事件；是否已知由 `ThePlantRegistry:KnowsFertilizer(key)` 决定（`plantregistrydata.lua`），而这四个键另外出现在 `skins_defs_data.lua` 的 `TheInventory:AddPlantRegistryKey` 调用列表里——该 API 的实现不在 `scripts/`，登记到底影响什么未核验，不要当成“不登记就一定认不出”。[判定](../../../raw/current/scripts/plantregistrydata.lua#L64)、[登记表](../../../raw/current/scripts/skins_defs_data.lua#L5275-L5291)
- **边界：嗡嗡肥料的加成不检查组件是否存在。** 回调直接写 `deployer.components.skilltreeupdater:IsActivated(...)`，只判了 `deployer` 非空；玩家都有该组件（[player_common](../../../raw/current/scripts/prefabs/player_common.lua#L2518)），非玩家实体触发的部署会在此报错。[回调](../../../raw/current/scripts/prefabs/mosquitofertalizer.lua#L68-L74)
- **误区：草鳄会长粪。** 本 build 里 `grassgator` 的 `periodicspawner` 整段被注释，注释里的 `SetPrefab("poop")` 不产生任何生成。[草鳄](../../../raw/current/scripts/prefabs/grassgator.lua#L199-L206)

## 验证记录

- 源码审阅（2026-11-10，steam-buildid:26474873）：逐行读四个工厂全文；`components/fertilizer.lua`／`smotherer.lua`／`fertilizerresearchable.lua`／`hauntable.lua`／`fuel.lua`／`fueled.lua`／`savedscale.lua`；两段部署 helper 与 `ACTIONS.DEPLOY`／`FERTILIZE`／`SMOTHER` 结算；`componentactions` 的 fertilizer／smotherer／deployable／fertilizerresearchable 候选；`farming_manager:AddTileNutrients`；旧 crop／grower／pickable 三处消费；`fuel.ontaken`→`poopcloud` 链与 `poopcloud`／`flies` 工厂。生成点按 `"poop"`／`"guano"` 字面量在 `scripts/` 全域反查后逐条读触发条件（含被注释的 `grassgator`）。
- 名称核验：`inspect_prefab_names.py` 按 `STRINGS.NAMES.COMPOST`／`GUANO`／`POOP`／`MOSQUITOFERTILIZER` 取到简中 堆肥／鸟粪／粪肥／嗡嗡肥料；标题与 aliases 用实显名，"便便"仅作俗称。
- 游戏运行：未运行；便便云在真实火堆上的可见位置、`flies` 的多人同步、作祟返回 false 的实际观感均未实测。
- 未覆盖：`spat`、`powdermonkey`、`pigtorch`、`monkeybarrel`、`worm_boss_util`、`scenarios/chest_cavesupplies`、`tree_rocks` 藤蔓战利品等同样出现同名引用，本页只确认引用存在、未核验触发条件；旧农田 `crop`／`grower`／`pickable` 的完整生命周期未展开。
- 内部检视 R1 修订（2026-11-10）：burnable 行拆成「点燃：DefaultBurnFn 只把非 tree／structure 的 persists 置 false」与「烧尽：DefaultBurntFn 产 1 件 ash（stack＝min(40,原栈)）并自删、海面无灰」；OnApplied 改为「四件无 finiteuses → 每次消耗 1 件」，补 Get() 拆堆语义；hauntvalue 改为「弹飞前已置 1、失败分支不清空」；研究键登记降级为「调用列表在、实现与作用未核验」；自肥目标点名沃姆伍德并补 StopSmoldering；扩展点改为「字段非空即调用」；sources 补 birdcage／ash／wormwood／plantregistrydata。
- 外部检视 B14 修订（2026-11-11，R05／R06）：鸟笼来源行换成实际门禁（月变异鸟／肉／普通变异鸟／普通鸟无对应 `_seeds` 才抽鸟粪，通用 `seeds` 属该支）；燃料行补「FX 缺失或无效退回取用者坐标」与「quagmire 摘回调但 `fuelvalue` 不变」；summary／副作用／陷阱同步；嗡嗡肥料补「基础 12／12／12、配方二阶、部署加成三阶 +8」并把示例的 +8 标注为自定义值。
- 结构检查：见本次交付报告的机械校验记录（validate_page.py 不加 --check-indexes、validate_source_links.py --strict、完整示例另做 Lua 语法检查）。

## 相关链接

- [[reference/prefabs/treegrowthsolution|树果酱（treegrowthsolution）]]（同一套部署 helper，另有催长与补船漏用途）
- [[reference/prefabs/damp_trail|地面痕迹与瞬时提示的外围 FX 族]]（`poopcloud` 的表现与清理）
- [[reference/prefabs/soil-amender|催长剂起子与超级催长剂]]、[[reference/prefabs/boat-patches|船体修补物]]（同为“肥料壳＋多消费者”的物品）
- [[reference/components/fertilizer|fertilizer]]、[[reference/components/smotherer|smotherer]]、[[reference/components/farming-manager|farming_manager]]

返回 [[reference/prefabs/index|原版 Prefab 索引]]。
