---
title: "晶洞果与夜莓：开壳计时、生熟两态与夜视 buff（ancientfruit_gem、ancientfruit_nightvision、ancientfruit_nightvision_cooked、nightvision_buff）"
created: "2026-11-09"
updated: "2026-11-10"
type: prefab
summary: "ancienttree_fruits.lua 的四个注册：晶洞果靠 HASHEATER 火源累计温度，只有温度到 35 才 Resume 的 30 秒开壳计时结束后掉一颗加权宝石；生夜莓才挂 cookable 与食用回调，且回调要求进食者有 playervision 才挂 240 秒 nightvision_buff（强制夜视＋每秒理智惩罚，重吃只重置时长），熟莓无回调、不再给夜视。"
aliases: ["ancientfruit_gem", "ancientfruit_nightvision", "ancientfruit_nightvision_cooked", "nightvision_buff", "晶洞果", "夜莓", "熟夜莓", "Geode Fruit", "Nightberry", "宝石果（俗称）", "夜视果"]
tags: [prefab, item, food]
size_class: complex
dst_build: "steam-buildid:26474873"
sources: ["prefabs/ancienttree_fruits.lua", "prefabs/ancienttree_defs.lua", "prefabs/ancienttrees.lua", "components/edible.lua", "components/eater.lua", "componentactions.lua", "components/perishable.lua", "components/stackable.lua", "components/timer.lua", "components/debuff.lua", "components/debuffable.lua", "components/playervision.lua", "components/sanity.lua", "components/grogginess.lua", "components/heater.lua", "prefabs/campfirefire.lua", "prefabs/coldfirefire.lua", "prefabs/rocky.lua", "entityscript.lua", "util.lua", "tuning.lua", "constants.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 晶洞果与夜莓：开壳计时、生熟两态与夜视 buff（ancientfruit_gem、ancientfruit_nightvision、ancientfruit_nightvision_cooked、nightvision_buff）

## 快速判断

- 职责与适用：`prefabs/ancienttree_fruits.lua` 一个文件返回四个注册——晶洞果 `ancientfruit_gem`、生夜莓 `ancientfruit_nightvision`、熟夜莓 `ancientfruit_nightvision_cooked`、内部 buff 实体 `nightvision_buff`。前三项分两组资产表（果实各一套，两个夜莓共一套），buff 不带资产，依赖表只声明不生成。[注册表](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L660-L664)、[资产与依赖](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L3-L18)
- 优先使用：要调宝石产出与分布改 `inst.GEMS_WEIGHTS`（只改已有键，见[典型接入](#改晶洞果宝石权重只动已有键)）；要调升温／开壳节奏改 `TUNING.ANCIENTFRUIT_GEM_*`；要调夜视时长改 `TUNING.ANCIENTTREE_NIGHTVISION_FRUIT_BUFF_DURATION`；两件果实的构造在[注册与构造](#注册与构造)。[权重字段](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L273-L274)、[开壳参数](../../../raw/current/scripts/tuning.lua#L8259-L8270)
- 重要副作用：开壳计时**只有温度 ≥ 35 时才继续走**，其余时间一律 Pause；计时结束时本体 `canbepickedup=false`、`persists=false`，按叠数逐件掉宝石后自删。`nightvision_buff` 是挂在进食者身上的隐藏实体，同时改 `playervision` 与 `sanity.externalmodifiers`。[计时切换](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L68-L99)、[结算](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L213-L238)、[buff 挂载](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L525-L541)
- 修改边界：本页只写果实与 buff。远古树本体、树种、树的可采阶段归 B29（`prefabs/ancienttrees.lua`／`ancienttree_seed`），Wiki 待建，本页只给 `fruit_prefab` 来源；宝石物品本身见 [[reference/prefabs/gems]]，腐烂产物见 [[reference/prefabs/spoiled-food]]。

## 注册与构造

中文名取自 `languages/chinese_s.po` 的 `STRINGS.NAMES`：晶洞果／夜莓／熟夜莓；`nightvision_buff` 无 NAMES，是内部实体（S2）。[三名](../../../raw/current/scripts/languages/chinese_s.po#L338605-L338617)

四个注册都不设 StateGraph／Brain，全是物品与 buff 实体；工厂返回值顺序即上表顺序，依赖表里的 `nightvision_buff`／`ancientfruit_nightvision_cooked` 只是声明，不等于构造时生成（S3：各注册的显式 `AddTag` 见各行，helper 间接添加的标签未逐一列出）。[返回](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L660-L664)

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `ancientfruit_gem` | `gem_fruit_fn`：Transform／SoundEmitter／AnimState／Network → build＝bank＝`ancienttree_gem_fruit`、`idle`、`pickupsound="rock"`、tag `molebait`、`MakeInventoryPhysics` → `SetPristine` → 主模拟建 `_temperature`／`GEMS_WEIGHTS`／钩子 | 组件：bait、inspectable、lootdropper、tradable、edible（`ELEMENTAL`，hunger 3）、inventoryitem（`SetSinks(true)`）、stackable（20）、timer（`hatch_timer` 30 秒）；**无** perishable、**无** cookable；末挂 `MakeHauntableLaunch`。[工厂](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L248-L317)、[组件段](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L285-L314) |
| `ancientfruit_nightvision` | `nightvision_fruit_fn`：Transform／AnimState／SoundEmitter／Network，AnimState `idle` 循环、`SetLightOverride(0.1)`／`SetSymbolLightOverride("fruit",0.4)`、`pickupsound="vegetation_firm"`、`MakeInventoryFloatable("small",.08,.75)`、pristine 前就打 tag `cookable`（注释说明为优化放进 pristine 态）→ `SetPristine` → 主模拟 | 组件：inspectable、inventoryitem、tradable、edible（`VEGGIE`，三维 12.5／−8／−20，`SetOnEatenFn`）、perishable（`PERISH_FAST`＝2880 秒，地面再乘 1.5）、stackable（40）、cookable（`product="ancientfruit_nightvision_cooked"`）；另挂 `MakeSmallBurnable`／`MakeSmallPropagator`。[工厂](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L407-L475)、[数值段](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L446-L462) |
| `ancientfruit_nightvision_cooked` | `cooked_nightvision_fruit_fn`：同 bank／build，`PlayAnimation("cooked",false)`、`MakeInventoryFloatable("small",0.1,0.9)`；**无** SoundEmitter、**无** cookable tag → `SetPristine` → 主模拟 | 组件：inspectable、inventoryitem、tradable、edible（未设 foodtype，落回默认 `GENERIC`；18.75／−3／−5）、perishable（`PERISH_SUPERFAST`＝1440 秒，地面再乘 1.5）、stackable（40）；**无** `SetOnEatenFn`（吃熟莓不给夜视）。[工厂](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L477-L521)、[数值段](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L498-L513) |
| `nightvision_buff` | `fn_nightvisionbuff`：只加 Transform／Network，`AddTag("CLASSIFIED")`，pristine 前建 net_bool `_enabled` → `SetPristine` → 客户端只监听 `enableddirty` 后返回 | 主模拟：`Hide()`、`persists=false`、debuff 组件设三回调且 `keepondespawn=true`、构造末尾立即 `buff_OnExtended` 起算时长；存读钩子挂 `OnSave`／`OnLoad`／`OnLongUpdate`。[工厂](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L621-L657) |

夜莓的两个状态都带 `edible` 与 `perishable`，晶洞果两个都没有；生莓在 pristine 前就打了 `cookable` tag（注释说明为优化放进 pristine 态，客户端因此也带该 tag），熟莓没有。三件物品都不是 equipment，也不参与装备槽。

### 晶洞果：加热判定与三段阈值（定义处 → 结算处）

时间基准沿用 480 秒一天、`perish_warp=1`（S1）。晶洞果没有 `perishable`，也不走孢子那套 `localPerishMultiplyer`；它的“手持／地面差异”由 **timer 暂停 + limbo 合并结算** 实现，见[limbo 与存读](#晶洞果limbo-内的暂停与合并结算)。[时间基准](../../../raw/current/scripts/tuning.lua#L19-L20)、[腐坏倍率](../../../raw/current/scripts/tuning.lua#L47)

| 名称 | 定义处 | 值 | 结算处与条件 |
|---|---|---:|---|
| `GEMFRUIT_UPDATE_TIME` | [文件常量](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L20) | 2 秒 | `DoPeriodicTask(2, _OnUpdate, 0, 2)`，首次立即执行；等于“每秒 +1 温度”的步长。[挂任务](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L304) |
| `HATCH_CAMPFIRE_RADIUS` | [tuning](../../../raw/current/scripts/tuning.lua#L1248) | 4 | 以果实为心找 `HASHEATER` 且不含 `INLIMBO` 的实体；与鸟蛋共用同一常量。[搜索](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L45) |
| `ANCIENTFRUIT_GEM_MIN_HEAT` | [tuning](../../../raw/current/scripts/tuning.lua#L8259) | 40 | 逐个热源累加 `heater:GetHeat`，**累加值首次 ≥ 40 就立刻升温（`+dt`，上限 60）并 `break`**：后面的热源不再读，所以排在后面的冷火不被计入、排在前面的会先拉低累加值；全部读完仍 < 40 才降温（`−dt`，下限 0）。[累加与提前跳出](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L48-L62) |
| `ANCIENTFRUIT_GEM_TEMPERATURE_THRESHOLD` | [tuning](../../../raw/current/scripts/tuning.lua#L8262-L8268) | MIN 15／WARMED 25／HEATED 35／MAX 60 | ≥MIN 起播 `heating_1`；≥WARMED 播 `heating_2`；≥HEATED 播 `heatin_loop` 并 **Resume** 计时；否则 Pause。[阈值分支](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L68-L99) |
| `ANCIENTFRUIT_GEM_HATCH_TIME` | [tuning](../../../raw/current/scripts/tuning.lua#L8260) | 30 秒 | 构造即 `StartTimer("hatch_timer", 30)`，但只有温度 ≥35 的帧才会被 Resume。[启动](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L301-L302) |
| 营火热度 | [等级表](../../../raw/current/scripts/prefabs/campfirefire.lua#L15) | 70／85／100／115 | `GetHeatFn` 按 `firefx.level` 取，缺档回 20；营火任意等级都超过 40。[取值](../../../raw/current/scripts/prefabs/campfirefire.lua#L37-L40) |
| 冷火 | [热性](../../../raw/current/scripts/prefabs/coldfirefire.lua#L51-L53) | 负 | `SetThermics(false,true)` 且 `GetHeat` 返回负值，会拉低 `heatindex`——但只在其被遍历到、且累加尚未达 40 时才有影响。[累加顺序](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L48-L59) |

推导（条件：**单个**果实、初始温度 0、t=0 起就在有效热源覆盖内即每次判定累加都 ≥ 40、期间不离火也不进 limbo）：首次更新在 t≈0，第 n 次更新在 t≈2(n−1) 秒、温度为 2n；第 18 次（t≈34 秒）到 36 才越过 35，计时器再从那一刻走 30 秒，**最短约 64 秒**开出第一颗宝石；温度上限 60 只是封顶，不加速。温度只在 2 秒轮询（或出 limbo 的合并结算）时变化。放进背包（limbo）后计时暂停；一次判定遍历完仍未达到 40 时，果实按 −dt 降温，最低为 0（走出 4 格半径或热源熄灭都属这种情形），代码不按距离做衰减（此段为推导，未实测）。

### 晶洞果：开壳、堆叠拆壳与宝石权重

计时结束走 `GemFruit_OnTimerDone`：先 `canbepickedup=false`（掉地前不能再捡）、播 `breaking`、放音效；**叠着就等 13 帧后整堆逐件开**，单体直接 `SpawnGem`；随后 `persists=false`，等当前动画长度减 1 帧 `Remove`。[结算](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L213-L238)

`GEMS_WEIGHTS` 默认六项：`bluegem` 4、`redgem` 4、`purplegem` 3、`greengem` 1、`orangegem` 1、`yellowgem` 1，合计 14；`weighted_random_choice` 抽的是“权重 ÷ 总和”，所以单件概率为 4/14≈28.6%（蓝、红）、3/14≈21.4%（紫）、1/14≈7.1%（绿、橙、黄）。[权重表](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L26-L33)、[抽取](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L162)、[权重算法](../../../raw/current/scripts/util.lua#L725-L744)

堆叠拆壳分三层，缺一层就会写出错：

- 抽奖按**件**算：先把`GEMS`（仅六键）浅拷贝成计数表，再按 `StackSize()` 逐件 `weighted_random_choice` 累加；因此 20 件叠加一次开壳等于 20 次独立抽取，不是“整堆一件”。[逐件累加](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L179-L186)
- 生成时按类型合堆：每种宝石循环 `SpawnGem`，能用 `RoomLeft()` 补满就 `SetStackSize`，再用 `Launch2(loot, inst, 1.5, 1.25, 0.3, 0, 2)` 抛出。[合堆与抛射](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L190-L208)
- 拆壳传温度：`SetOnDeStack` 回调把母堆的 `_temperature` 复制给拆出的新个体，并立刻 `_OnUpdate(0)` 重算动画与计时状态；`Stackable:Get` 就是在这里回调的。[拆壳复制](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L240-L246)、[挂回调](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L297-L298)、[Get 调用](../../../raw/current/scripts/components/stackable.lua#L109-L120)

### 晶洞果：limbo 内的暂停与合并结算

- 进入 limbo：`PauseTimer("hatch_timer")`、记下 `_inlimbo_time`、取消 2 秒轮询任务。[进入](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L122-L131)
- 离开 limbo：用 `GetTime() - _inlimbo_time` 作为**一个** dt 调一次 `_OnUpdate`，再重建轮询任务；因为带 dt 的升温／降温都被 `math.min/max` 夹住，长时间在背包里回来只结算一次，不会逐帧补齐。[离开](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L102-L120)、[夹取](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L56-L66)
- 存档：仅在 `_temperature > 0` 时写 `data.temperature`，并先减去 limbo 已经过的时间；减到 ≤0 就写 nil。[存档](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L133-L145)
- 读档：回填温度后立即 `_OnUpdate(0)` 刷新动画与计时状态；`hatch_timer` 的剩余时间由 timer 组件自己存读。[读档](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L147-L153)、[timer 存读](../../../raw/current/scripts/components/timer.lua#L122-L142)

### 夜莓生熟两态：三维、腐坏与堆叠

生莓三维 12.5／−8／−20，熟莓 18.75／−3／−5；两者都因为负生命或负理智在赋值瞬间被打上 `badfood`。定义处是 `edible` 的赋值行，结算处是 `eater:Eat` 里乘 `foodmemory` 乘 `*absorption` 后的三行 `DoDelta`。三层门要分清：①**吃不吃**由 `PrefersToEat`（食性组）决定，`Eat` 只在它为真时进主分支；②**负值是否结算**由 `DoFoodEffects` 决定——它对“强健胃×怪物肉”“生肉耐受×生肉”“命中食物亲和”返回 false，此时该条负生命／负理智**不计入**（delta 保持 0、不做 `DoDelta`），并不是“才吞得下”；③**生莓才给夜视**：只有生莓挂 `SetOnEatenFn`，且回调里还要求进食者有 `playervision`。[生莓数值](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L446-L451)、[熟莓数值](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L502-L505)、[三维换算](../../../raw/current/scripts/tuning.lua#L2022-L2060)、[结算](../../../raw/current/scripts/components/eater.lua#L198-L256)、[吃不吃](../../../raw/current/scripts/components/eater.lua#L203)、[负值门](../../../raw/current/scripts/components/eater.lua#L211-L213)、[负效果判断](../../../raw/current/scripts/components/eater.lua#L166-L170)、[生莓回调](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L339-L348)、[badfood](../../../raw/current/scripts/components/edible.lua#L1-L7)

| 项目 | `ancientfruit_nightvision` | `ancientfruit_nightvision_cooked` | 定义／结算 |
|---|---|---|---|
| foodtype | `VEGGIE` | 未设，落回 `GENERIC` | `Edible` 构造默认值即 `GENERIC`。[默认](../../../raw/current/scripts/components/edible.lua#L19-L24) |
| 饥饿／生命／理智 | `CALORIES_SMALL` 12.5／`−HEALING_MEDSMALL` −8／`−SANITY_MEDLARGE` −20 | `CALORIES_MEDSMALL` 18.75／`−HEALING_SMALL` −3／`−SANITY_TINY` −5 | 常量见 tuning；负生命／负理智只在 `DoFoodEffects` 为真时计入（免疫命中则跳过，见上文三层门）；吃熟莓**不会**触发夜视，因为没挂 `SetOnEatenFn`。[常量](../../../raw/current/scripts/tuning.lua#L2022-L2060) |
| 腐坏（定义值／地面实际） | 定义 `PERISH_FAST`＝2880 秒（6 天）；掉在地上时 ÷`PERISH_GROUND_MULT`(1.5)→1920 秒（4 天） | 定义 `PERISH_SUPERFAST`＝1440 秒（3 天）；地面 960 秒（2 天） | `SetPerishTime` 同时写 `perishtime` 与 `perishremainingtime`，`StartPerishing` 才起 10 秒级轮询；地面且无 owner 时 `modifier` 直接取 1.5，在背包／容器里不乘这一项。[生莓腐坏](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L453-L456)、[熟莓腐坏](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L507-L510)、[SetPerishTime](../../../raw/current/scripts/components/perishable.lua#L188-L196)、[起轮询](../../../raw/current/scripts/components/perishable.lua#L262-L271)、[地面倍率](../../../raw/current/scripts/components/perishable.lua#L97)、[速率结算](../../../raw/current/scripts/components/perishable.lua#L127-L133)、[常量](../../../raw/current/scripts/tuning.lua#L2083) |
| 腐烂产物 | `spoiled_food` | `spoiled_food` | `Perish` 时按 `onperishreplacement` 生成，堆数超过目标上限会 `SetIgnoreMaxSize`。[替换](../../../raw/current/scripts/components/perishable.lua#L293-L315) |
| 堆叠 | `STACK_SIZE_SMALLITEM`＝40 | 40 | 晶洞果是 `STACK_SIZE_MEDITEM`＝20。[堆叠表](../../../raw/current/scripts/tuning.lua#L82-L83) |
| 可燃 | `MakeSmallBurnable(SMALL_BURNTIME)`＋`MakeSmallPropagator` | 同左 | `SMALL_BURNTIME`＝30×0.25＝7.5 秒。[常量](../../../raw/current/scripts/tuning.lua#L1681)、[挂载](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L469-L470) |

上表“地面实际”只对夜莓两态成立：晶洞果没有 `perishable`，不参与任何腐坏计时；夜莓拿在手里或放进容器时按定义值走，只有丢在地上（无 owner）才吃 1.5 倍速率（S5）。

晶洞果的 `edible` 是 `ELEMENTAL`＋hunger 3，但**默认玩家吃不到**：`EAT` 动作要求施动者的食性组里含该 `edible_<type>` 标签，而玩家食性默认是 `OMNI`，其 `types` 不含 `ELEMENTAL`；真正消费它的是显式声明 `ELEMENTAL` 食性的生物（如岩石龙虾 `SetDiet(EATER_FOODTYPES, EATER_FOODTYPES)`）。未设 `healthvalue`／`sanityvalue` 时取组件默认的 10／0，不要当成“只回 3 点饥饿、无生命”。[EAT 门禁](../../../raw/current/scripts/componentactions.lua#L2683-L2690)、[OMNI 组](../../../raw/current/scripts/constants.lua#L2049-L2063)、[默认食性](../../../raw/current/scripts/components/eater.lua#L18-L24)、[岩石龙虾](../../../raw/current/scripts/prefabs/rocky.lua#L436-L498)

### 夜视 buff：挂载、时长、叠加与保存

食用回调只有两步：进食者有 `playervision` 就 `AddDebuff("nightvision_buff", "nightvision_buff")`，并把 `grogginess` 抬到至少 1.5（不是加 1.5）。`AddDebuff` 会按需补挂 `debuffable`，实体已死或成鬼时直接拒绝。[回调](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L339-L348)、[AddDebuff 包装](../../../raw/current/scripts/entityscript.lua#L2140-L2156)、[grogginess](../../../raw/current/scripts/components/grogginess.lua#L202-L208)

| 环节 | 行为 | 证据 |
|---|---|---|
| 时长来源 | `ANCIENTTREE_NIGHTVISION_FRUIT_BUFF_DURATION`＝`total_day_time/2`＝240 秒；buff 一生成就起算，不等 Attach | [tuning](../../../raw/current/scripts/tuning.lua#L8254)、[起算](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L649) |
| 挂载 | 父级设为目标、位置归零；监听目标 `death` → `debuff:Stop()`；`PushForcedNightVision(inst, 1, 四时段同图, true, 粉色环境)`；理智外挂修正器 `−DAPPERNESS_MED_LARGE` | [Attached](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L525-L541)、[色立方](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L321-L335) |
| 理智结算 | `externalmodifiers` 是 additive 源列表，被加进每秒 `rate`，再 `DoDelta(rate*dt)`；`DAPPERNESS_MED_LARGE`＝100/(300×4.5)≈0.0741/秒（约 35.5/天） | [列表](../../../raw/current/scripts/components/sanity.lua#L87)、[rate](../../../raw/current/scripts/components/sanity.lua#L590)、[结算](../../../raw/current/scripts/components/sanity.lua#L609)、[常量](../../../raw/current/scripts/tuning.lua#L2216) |
| 重复食用 | `debuffable:AddDebuff` 发现同名已在 → `debuff:Extend` → `buff_OnExtended` 取消旧任务、重设**整段** 240 秒；不累加、不叠层 | [Extend 分支](../../../raw/current/scripts/components/debuffable.lua#L95-L108)、[延长实现](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L565-L572) |
| 到期与移除 | 到期 `Stop()` → `debuffable:RemoveDebuff` → `OnDetach` → Pop 夜视、移除理智修正、10 帧后 `Remove`（留给客户端跑 dirty 事件） | [Stop](../../../raw/current/scripts/components/debuff.lua#L28-L32)、[移除](../../../raw/current/scripts/components/debuffable.lua#L110-L124)、[Detached](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L543-L557) |
| 保存 | 存 `GetTaskRemaining(task)` 到 `data.remaining`，读档据此重建任务；`OnLongUpdate` 按 dt 重算剩余，≤0 立即到期 | [存读](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L574-L593)、[长更新](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L595-L609) |
| 客户端 | 主模拟改 `_enabled` 触发 `enableddirty`；客户端只在自己就是该 buff 的父级（`ThePlayer`）时 Push／Pop | [dirty](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L611-L619)、[强制夜视栈](../../../raw/current/scripts/components/playervision.lua#L229-L262) |

因此“叠加”只有一种语义：**重吃刷新满时长**。两颗夜莓连续吃不会变 480 秒，也不会叠两层理智惩罚——第二次只走 Extend 分支，不会新建实体。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| `组件：timer` | `hatch_timer` 30 秒；≥HEATED 时 Resume，其余 Pause；`timerdone` 事件带 `name` | 仅主模拟构造；暂停保留剩余时间 | [[reference/components/timer]]、[启动](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L301-L309) |
| `组件：stackable` | 上限 20；`SetOnDeStack` 复制温度；开壳时 `RoomLeft` 合堆 | 主模拟；拆壳由 `Get` 触发 | [[reference/components/stackable]]、[挂回调](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L297-L299) |
| `组件：edible` | 晶洞果 `ELEMENTAL`／3；生莓 `VEGGIE` 三维＋回调；熟莓默认 `GENERIC` | 主模拟赋值时：设 `foodtype` 打 `edible_<type>` 标签；`healthvalue`／`sanityvalue` 为负才额外打 `badfood` | [[reference/components/edible]]、[赋值](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L290-L292) |
| `组件：perishable` | 生莓 2880 秒、熟莓 1440 秒，产物 `spoiled_food` | 仅夜莓两态；晶洞果没有该组件 | [[reference/components/perishable]]、[腐坏段](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L453-L456) |
| `组件：debuff`／`debuffable` | `AddDebuff` 生成 buff 并 `AttachTo`；已有则 `Extend`；`keepondespawn` 让 despawn 不清 | 挂在进食者身上；`debuffable` 缺失时由 `EntityScript:AddDebuff` 补挂 | [[reference/components/debuff]]、[[reference/components/debuffable]]、[三回调](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L643-L647) |
| `组件：playervision` | `PushForcedNightVision(inst, 1, cc, true, ambient)`／`PopForcedNightVision` | 服务端先 push；客户端靠 net_bool 脏事件补 | [push/pop](../../../raw/current/scripts/components/playervision.lua#L229-L262)（Wiki 待建） |
| `组件：sanity` | `externalmodifiers:SetModifier(inst, −DAPPERNESS_MED_LARGE)` | 仅目标有 sanity 时；Detach 时 `RemoveModifier` | [[reference/components/sanity]]、[挂载](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L538-L540) |
| `Replica／同步：nightvision_buff._enabled` | `net_bool(..., "enableddirty")`，客户端只读 | 建在 pristine 之前，客户端可见 | [net_bool](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L629) |
| `SG／Brain：无` | 四注册都不 `SetStateGraph`／`SetBrain` | 全端 | [工厂返回](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L660-L664) |
| `API／数据：weighted_random_choice` | 按权重 ÷ 总和抽宝石 | 主模拟 | [实现](../../../raw/current/scripts/util.lua#L733-L744) |
| `关联 Prefab：ancientfruit_nightvision_cooked` | 生莓 `cookable.product` | 熟制时替换 | [产物](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L461-L462) |
| `关联 Prefab：spoiled_food` | `onperishreplacement` | 腐坏时生成 | [[reference/prefabs/spoiled-food]] |
| `关联 Prefab：六种宝石` | 开壳产物 `bluegem`／`redgem`／`purplegem`／`greengem`／`orangegem`／`yellowgem` | 每次开壳按叠数逐件抽 | [[reference/prefabs/gems]] |
| `资源：ancienttree_gem_fruit.zip`／`ancienttree_nightvision_fruit.zip`／`nightvision_fruit_cc.tex` | bank／build 与色立方贴图 | 声明即依赖，不证明资源已加载 | [资产表](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L3-L12) |

### 获得：远古树 `fruit_prefab`

两种树各在 `TREE_DEFS` 里声明自己的果实与采数：宝石树 `workaction="MINE"`、`fruit_prefab="ancientfruit_gem"`、`numtoharvest=3`；夜视树 `workaction="CHOP"`、`fruit_prefab="ancientfruit_nightvision"`、`numtoharvest=4`（S4）。[宝石树](../../../raw/current/scripts/prefabs/ancienttree_defs.lua#L206-L211)、[夜视树](../../../raw/current/scripts/prefabs/ancienttree_defs.lua#L273-L278)

采果与再生：`pickable:SetUp(fruit_prefab, fruit_regen.max, numtoharvest)` 先按 25 天建，随后 `UpdatePickableRegenTime` 用每棵树的 `_plantdata.fruit_regen` 覆盖（12–25 天）；树被采倒时若仍可采，按 `numtoharvest` 逐颗掉果。[初始化](../../../raw/current/scripts/prefabs/ancienttrees.lua#L505-L511)、[再生区间](../../../raw/current/scripts/prefabs/ancienttree_defs.lua#L337-L340)、[覆盖](../../../raw/current/scripts/prefabs/ancienttrees.lua#L375-L377)、[倒树掉果](../../../raw/current/scripts/prefabs/ancienttrees.lua#L313-L318)

夜莓额外受时间门控：只有当世界入夜或本身在洞穴时果实才可交互，白天 `caninteractwith=false`；`CanRegenFruits` 不满足时直接 `MakeEmpty`。这条门控属于树侧的 `master_postinit`，果实本体不感知昼夜。[夜门控](../../../raw/current/scripts/prefabs/ancienttree_defs.lua#L151-L182)、[挂载](../../../raw/current/scripts/prefabs/ancienttree_defs.lua#L315-L333)

## 典型接入

### 改晶洞果宝石权重（只动已有键）

- 前提与选择：目标是 `modmain.lua`，不需要新资源。选 `AddPrefabPostInit` 而不是改 `TUNING`，因为权重是实例字段 `inst.GEMS_WEIGHTS`，且只在主模拟被赋值；客户端没有这个字段，必须先判端。[赋值](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L269-L274)
- 生效与撤销：写的是模块级共享表，所有已存在与后续生成的晶洞果立即共享新权重；删除这段代码后新开局恢复原权重，已经掉在地上的宝石不回滚。
- 原版依据：`SpawnGem` 用 `weighted_random_choice(inst.GEMS_WEIGHTS)` 抽单件，`SpawnAndLaunchGems` 先按叠数逐件抽再按类型合堆。[单件](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L155-L169)、[整堆](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L171-L211)

```lua
-- modmain.lua：把晶洞果的宝石分布压向紫宝石
AddPrefabPostInit("ancientfruit_gem", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    local weights = inst.GEMS_WEIGHTS
    if weights == nil then
        return
    end
    weights.bluegem = 1
    weights.redgem = 1
    weights.purplegem = 6
end)
```

- 调整与验证：只要保留正数，权重之和不必等于 14，概率按新总和重算（本例 1+1+6+1+1+1=11，紫宝石 6/11≈54.5%），但**不要新增键**——`GEMS` 的键集在建文件时就固定为原六种，新增键会让整堆开壳那条 `spawned_prefabs[gem] + 1` 变成 `nil` 加一。同步改 `TUNING.ANCIENTFRUIT_GEM_HATCH_TIME`（在果实被构造前生效）可缩短开壳等待。[键集构造](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L35-L41)、[危险行](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L182-L186)

## 常见陷阱与限制

- **把晶洞果当玩家食物**：它的 `edible` 是 `ELEMENTAL`，玩家默认 `OMNI` 食性拿不到 EAT 动作；`hungervalue=3` 只对声明 `ELEMENTAL` 的生物有意义。误判会写出“喂玩家回 3 饥饿”的错误结论。[食性](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L290-L292)、[EAT 门禁](../../../raw/current/scripts/componentactions.lua#L2683-L2690)
- **以为“有火就开始孵化”**：计时器只在温度 ≥ `HEATED`(35) 的帧 Resume；温度 15–34 只播不同动画、计时仍停。把果实放在 `HATCH_CAMPFIRE_RADIUS`(4) 之外（或热源熄灭）时，温度按 −1/秒回落到 0——这段代码**不按距离衰减**，只看 4 格内有没有 `HASHEATER` 且其 `GetHeat` 够不够。冷火只拉低**当前** `heatindex`：若本次遍历累加仍能到 40，这一轮照样升温；只有整个判定始终没到 40，才按 `−dt` 降温（到 40 即 `break`，后面热源不再读）。[阈值分支](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L68-L99)
- **以为开壳是“整堆算一次”**：叠 20 件等于抽 20 次，并按类型合并成几堆抛出；单堆掉落后 `persists=false`，存档里不会留下“正在开壳”的果实本体。[逐件](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L182-L186)、[自删](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L236-L237)
- **在 limbo 里等温度自己降**：进入 limbo 时轮询任务被取消、计时暂停，离开时只用合并 dt 结算一次；因此背包里的果实不会“逐帧冷却”，也不会在包里孵化。[limbo](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L102-L131)
- **以为熟夜莓也给夜视**：`ancientfruit_nightvision_cooked` 没挂 `SetOnEatenFn`，夜视只来自生莓的回调，且回调还要求进食者有 `playervision`；熟莓只是另一种三维的食物（默认 `GENERIC` 食性）。
- **以为负生命／负理智一定会扣，或以为 `DoFoodEffects` 管“能不能吃”**：能不能吃是 `PrefersToEat`（食性组）这一层；`DoFoodEffects` 只影响负值是否计入，且它返回 false 的三种情形（强健胃×怪物肉、生肉耐受×生肉、食物亲和）恰恰是**跳过**负值。[吃不吃](../../../raw/current/scripts/components/eater.lua#L203)、[负值结算](../../../raw/current/scripts/components/eater.lua#L211-L213)、[负效果判断](../../../raw/current/scripts/components/eater.lua#L166-L170)[熟莓组件](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L498-L513)
- **假设重吃夜莓会叠时长**：`AddDebuff` 对同名已在只走 `Extend`，`buff_OnExtended` 把任务重置为整段 240 秒；不累加、不产生第二个 buff 实体。[Extend](../../../raw/current/scripts/components/debuffable.lua#L95-L108)、[重置](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L565-L572)
- **忽略 buff 的理智代价**：除了入口的 −20 理智，生效期间还有约 0.0741/秒的持续流失。入口那 −20 属于负值分支，`edible` 对 `sanityvalue < 0` 直接忽略腐烂衰减，不会因为放久变陈而变少。[rate](../../../raw/current/scripts/components/sanity.lua#L590)、[负值分支](../../../raw/current/scripts/components/edible.lua#L72-L84)

## 验证记录

- 源码审阅：2026-11-09，steam-buildid:26474873（与 `raw/current/BUILD.txt` 一致）。逐行读过 `prefabs/ancienttree_fruits.lua`（663 行）、`prefabs/ancienttree_defs.lua`（349 行）、`prefabs/ancienttrees.lua` 的果实相关段，以及 edible／eater／perishable／stackable／timer／debuff／debuffable／playervision／sanity／grogginess／heater、campfirefire／coldfirefire／rocky、componentactions、util、tuning、constants 与 chinese_s.po 的对应行。
- 游戏运行：未运行；以下为源码推导或未核验（S7）——最短开壳时间约 64 秒（推导，见[加热判定](#晶洞果加热判定与三段阈值定义处--结算处)）、冷火负热量的实际手感、limbo 合并结算的手感、`grogginess` 抬到 1.5 的实际表现、多人下 buff 的 `_enabled` 同步时序。
- 档位：`complex`。整体保留的范围是四个注册的构造与端分界、晶洞果“温度 → 计时 → 开壳 → 结算”整链（含 limbo 与存读）、夜莓两态数值与腐坏、buff 的挂载／延长／保存三段；这些环节互相引用（例如拆壳要读温度、开壳要读计时器状态），拆开会让任一段失去上下文，故不降为 `standard`。
- 结构检查：见交付回执（`validate_page.py` 与 `validate_source_links.py --strict`）。
- 外部复验回改（2026-11-10，R11）：陷阱改为“冷火只拉低当前 `heatindex`，累加仍到 40 这轮照样升温，整个判定没到 40 才按 `−dt` 降温”，并写明本段代码不按距离衰减；推导删去“靠火堆越远”，改为“离开有效热源（走出 4 格／熄灭）才按 −1/秒回落”。索引条目由 B 同步。
- 外部复验回改（第二轮，2026-11-10，R11）：推导末句再收紧为“一次判定遍历完仍未达到 40 时，果实按 −dt 降温，最低为 0（走出 4 格半径或热源熄灭都属这种情形）”，删去会被读成“负值参与抵消”的表述——累加达 40 即 `break` 并升温，排在其后的负值不会读取，与热源表、陷阱口径一致。
- 外部检视回改（2026-11-10，R02／R11）：R02——摘要拆开“生莓才挂 cookable 与回调／回调需 playervision／熟莓无回调”，正文把三层门（`PrefersToEat` 决定吃不吃、`DoFoodEffects` 决定负值是否计入、`SetOnEatenFn` 只在生莓且有 playervision）写明并新增一条陷阱；R11——加热表的 `MIN_HEAT` 行改为“逐个累加、首次 ≥40 立刻升温并 break”，冷火行注明只在被遍历到且未达 40 时生效，64 秒推导补“单果／初始 0／全程有效热源／仅 2 秒轮询或出 limbo 结算”四项条件，陷阱同步。索引行由主代理同步。
- 检视回改（2026-11-09）：按独立检视报告只改点名条目——标题／H1／首段主名统一为「晶洞果」，「宝石果」降为俗称别名；开壳最短时间更正为 64 秒并给推导式（第 18 次更新 t≈34 秒达 36 才越 35，再走 30 秒）；「S5 例外」改写为“不用 `localPerishMultiplyer`”；腐坏行的地面倍率补（S5）；页尾补 Prefab 索引返回行。复验轮次（同日）：验证记录的开壳时间与正文统一为 64 秒，结构检查行改为「见交付回执」，edible 行把 `edible_<type>` 与 `badfood` 的触发条件拆开。其余段落未动。

## 相关链接

- [[reference/components/timer]]、[[reference/components/stackable]]、[[reference/components/edible]]、[[reference/components/perishable]]、[[reference/components/debuff]]、[[reference/components/debuffable]]、[[reference/components/sanity]]、[[reference/components/playervision]]
- [[reference/prefabs/gems]]、[[reference/prefabs/spoiled-food]]
- 远古树本体与树种（`prefabs/ancienttrees.lua`／`ancienttree_seed`）由 B29 维护，Wiki 待建；本页只覆盖果实侧。

返回 [[reference/prefabs/index|Prefab 索引]]
