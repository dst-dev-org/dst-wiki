---
title: "蔬果作物与熟食（eggplant、durian、pomegranate、dragonfruit、watermelon、tomato 及各自 _cooked）"
created: "2026-11-08"
updated: "2026-11-08"
type: prefab
summary: "六个农田蔬果与烤制形态共 12 个注册：三维与新鲜度、生熟堆叠 20／40、榴莲的料理分类、西瓜生食降温的速率与门禁、火龙果的万圣月变异，以及番茄的 Gorge 图标与农田来源。"
aliases: ["eggplant", "durian", "pomegranate", "dragonfruit", "watermelon", "tomato", "eggplant_cooked", "durian_cooked", "pomegranate_cooked", "dragonfruit_cooked", "watermelon_cooked", "tomato_cooked", "茄子", "榴莲", "石榴", "火龙果", "西瓜", "番茄", "烤茄子", "超臭榴莲", "切片熟石榴", "熟火龙果", "烤西瓜", "烤番茄", "蔬果作物"]
tags: [prefab, item, food]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/veggies.lua", "prefabs/farm_plants.lua", "prefabs/farm_plant_defs.lua", "map/layouts.lua", "map/rooms/forest/blockers.lua", "prefabs/fruitdragon.lua", "prefabs/halloweenpotion_moon.lua", "prefabs/quagmire_plantables.lua", "prefabs/seeds.lua", "prefabs/wobysmall.lua", "prefabs/wobybig.lua", "prefabs/polly_rogers.lua", "cooking.lua", "preparedfoods.lua", "components/halloweenmoonmutable.lua", "components/halloweenpotionmoon.lua", "components/edible.lua", "components/eater.lua", "components/temperature.lua", "components/perishable.lua", "components/stackable.lua", "components/cookable.lua", "components/inventoryitem.lua", "actions.lua", "componentactions.lua", "recipes.lua", "tuning.lua", "constants.lua", "entityscript.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 蔬果作物与熟食（eggplant、durian、pomegranate、dragonfruit、watermelon、tomato 及各自 _cooked）

## 快速判断

- 职责与适用：`prefabs/veggies.lua` 的 `VEGGIES` 表为每项生成生／熟两件可食物品。本页覆盖 `eggplant`、`durian`、`pomegranate`、`dragonfruit`、`watermelon`、`tomato` 及各自 `_cooked`，共 12 个 `inventoryitem` 食物注册。可借鉴：按名单点分支。[VEGGIES 表](../../../raw/current/scripts/prefabs/veggies.lua#L39-L155)、[注册循环](../../../raw/current/scripts/prefabs/veggies.lua#L1102-L1110)
- 优先使用：查三维、新鲜度与堆叠取[生熟两态数值](#生熟两态数值)；生／熟的公共骨架、端分界、堆叠与新鲜度通则、食用与烹饪接线见 [[reference/prefabs/farm-crops-staple#共用工厂与生熟两态]]；农田来源与种子权重见[获得与种子权重](#获得与种子权重)；巨型形态见 [[reference/prefabs/giant-crops]]。
- 重要副作用：生茄、生榴莲、生西瓜走 `stackable` 默认上限 20，其余 40；榴莲在锅里同时给 `fruit` 与 `monster`；西瓜生食带 −40／5 秒的体温项；生火龙果被月亮精华液选中会变成生物 `fruitdragon`；番茄生熟图标都换 Gorge 版。
- 修改边界：不覆盖 `<name>_seeds`（另属种子注册）、三件 `<name>_oversized*`（见 [[reference/prefabs/giant-crops]]）与种植／生长（见 [[reference/components/crop]]、[[reference/components/farmplantable]]）；熟件没有 `cookable`，不能再烤一次。

## 注册与构造

12 个注册由 `MakeVeggie(name, not SEEDLESS[name])` 生成，本页只取 `<name>` 与 `<name>_cooked`。生／熟的公共骨架、端分界、组件集合差异与堆叠／新鲜度通则由 [[reference/prefabs/farm-crops-staple#共用工厂与生熟两态]] 承接，下表只列本页六键的实际差异。[导出段](../../../raw/current/scripts/prefabs/veggies.lua#L1084-L1099)

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `eggplant` | `fn`；表项 `UNCOMMON`、`CALORIES_MED`／`HEALING_MEDSMALL`／`PERISH_MED` | 堆叠**不写** `maxsize` → 组件默认 20；主模拟挂 `snowmandecor`；无浮水参数（走默认 `MakeInventoryFloatable`）。[表项](../../../raw/current/scripts/prefabs/veggies.lua#L60-L66)、[堆叠例外](../../../raw/current/scripts/prefabs/veggies.lua#L613-L619)、[雪人装饰](../../../raw/current/scripts/prefabs/veggies.lua#L679-L681) |
| `durian` | `fn`；`RARE`、`CALORIES_MED`、`-HEALING_SMALL`／`-SANITY_TINY`、`PERISH_MED` | 唯一带 `secondary_foodtype = FOODTYPE.MONSTER` 的键，生熟都写；堆叠默认 20；无浮水参数，见[榴莲的怪物食性](#榴莲的怪物食性)。[表项](../../../raw/current/scripts/prefabs/veggies.lua#L68-L70)、[写入](../../../raw/current/scripts/prefabs/veggies.lua#L600) |
| `pomegranate` | `fn`；`RARE`、`CALORIES_TINY`／`HEALING_SMALL`、`PERISH_FAST` | 无专属组件、堆叠 40；浮水 `("small", nil, 0.8)`。[表项](../../../raw/current/scripts/prefabs/veggies.lua#L72-L74)、[浮水](../../../raw/current/scripts/prefabs/veggies.lua#L578-L583) |
| `dragonfruit` | `fn`；与 pomegranate 同档（`RARE`／9.375／3／6 天） | 浮水 `("small", 0.1, 0.8)`；挂 `halloweenmoonmutable`（`fruitdragon`＋`onmutatefn`），见[火龙果的万圣月变异](#火龙果的万圣月变异)。[表项](../../../raw/current/scripts/prefabs/veggies.lua#L76-L83)、[接线](../../../raw/current/scripts/prefabs/veggies.lua#L672-L677) |
| `watermelon` | `fn`；`UNCOMMON`、`CALORIES_SMALL`／`HEALING_SMALL`／`SANITY_TINY` | 堆叠默认 20；生食写 `temperaturedelta = -40`／`temperatureduration = 5`，见[西瓜生食降温](#西瓜生食降温)；浮水 `("med", 0.05, 0.7)`。[表项](../../../raw/current/scripts/prefabs/veggies.lua#L114-L116)、[降温](../../../raw/current/scripts/prefabs/veggies.lua#L621-L624) |
| `tomato` | `fn`；`COMMON`、`CALORIES_SMALL`／`HEALING_SMALL`、`PERISH_FAST` | 浮水 `(nil, 0.1, 0.75)`（size 省略）；`QUAGMIRE_PORTS` 命中 → 图标换 `quagmire_tomato`，见[番茄的 Gorge 图标](#番茄的-gorge-图标)；堆叠 40。[表项](../../../raw/current/scripts/prefabs/veggies.lua#L124-L126)、[图标](../../../raw/current/scripts/prefabs/veggies.lua#L641-L643) |
| 六个 `_cooked` | `fn_cooked`；读同表项的 `cooked_*` 半段 | 沿用本作物 bank／build 播 `cooked`；堆叠固定 40；无 `cookable`／`weighable`／`snowmandecor`／`halloweenmoonmutable`／温度字段；榴莲仍写 MONSTER，番茄仍换 `quagmire_tomato_cooked`。[熟工厂](../../../raw/current/scripts/prefabs/veggies.lua#L707-L777)、[堆叠](../../../raw/current/scripts/prefabs/veggies.lua#L752-L753) |

### 生熟两态数值

一天 `total_day_time`＝480 秒（`seg_time`=30×16）、`perish_warp`=1，故天数与秒数可直接换算；饥饿基准 `calories_per_day`=75。[基准](../../../raw/current/scripts/tuning.lua#L19-L20)、[perish_warp](../../../raw/current/scripts/tuning.lua#L47)、[PERISH_*](../../../raw/current/scripts/tuning.lua#L2043-L2051)、[calories_per_day](../../../raw/current/scripts/tuning.lua#L41)、[CALORIES_*](../../../raw/current/scripts/tuning.lua#L2058-L2062)；档位换算：`CALORIES_TINY`=9.375、`CALORIES_SMALL`=12.5、`CALORIES_MED`=25；`HEALING_TINY`=1、`HEALING_SMALL`=3、`HEALING_MEDSMALL`=8、`HEALING_MED`=20；`SANITY_TINY`=5。[HEALING_*](../../../raw/current/scripts/tuning.lua#L2021-L2030)、[SANITY_*](../../../raw/current/scripts/tuning.lua#L2032-L2035)

| 注册名 | 饥饿 | 生命 | 理智 | 新鲜度 | 堆叠上限 |
|---|---:|---:|---:|---|---:|
| `eggplant` | 25 | 8 | 0 | 10 天（4800 秒） | 20 |
| `eggplant_cooked` | 25 | 20 | 0 | 6 天（2880 秒） | 40 |
| `durian` | 25 | −3 | −5 | 10 天（4800 秒） | 20 |
| `durian_cooked` | 25 | 0 | −5 | 6 天（2880 秒） | 40 |
| `pomegranate` | 9.375 | 3 | 0 | 6 天（2880 秒） | 40 |
| `pomegranate_cooked` | 12.5 | 20 | 0 | 3 天（1440 秒） | 40 |
| `dragonfruit` | 9.375 | 3 | 0 | 6 天（2880 秒） | 40 |
| `dragonfruit_cooked` | 12.5 | 20 | 0 | 3 天（1440 秒） | 40 |
| `watermelon` | 12.5 | 3 | 5 | 6 天（2880 秒） | 20 |
| `watermelon_cooked` | 12.5 | 1 | 7.5 | 3 天（1440 秒） | 40 |
| `tomato` | 12.5 | 3 | 0 | 6 天（2880 秒） | 40 |
| `tomato_cooked` | 12.5 | 20 | 0 | 10 天（4800 秒） | 40 |

堆叠列的两个来源不同：40 是工厂显式写 `TUNING.STACK_SIZE_SMALLITEM`，20 是 `stackable` 构造默认值 `TUNING.STACK_SIZE_MEDITEM`（工厂只对 `pumpkin`／`eggplant`／`durian`／`watermelon` 跳过赋值）。[档位](../../../raw/current/scripts/tuning.lua#L81-L83)、[默认](../../../raw/current/scripts/components/stackable.lua#L23-L28)、[生成器例外](../../../raw/current/scripts/prefabs/veggies.lua#L613-L619)

### 获得与种子权重

六键的农田来源都是各自的植株 `farm_plant_<name>`。`map/layouts.lua` 的 `FisherPig` 布局另有 `unknown_fruit` 池（含 `eggplant`／`durian`／`pomegranate`／`dragonfruit`），但该布局只见定义、房间引用未证实，**放置未核验**。[池](../../../raw/current/scripts/map/layouts.lua#L324)、[摆放](../../../raw/current/scripts/map/layouts.lua#L340)、[引用仅注释](../../../raw/current/scripts/map/rooms/forest/blockers.lua#L171-L171)。`PLANT_DEFS` 把植株的 `product`／`product_oversized`／`seed` 填成 `<name>`／`<name>_oversized`／`<name>_seeds`，收获走 `pickable`＋`lootsetupfn` 按压力出产物。[字段](../../../raw/current/scripts/prefabs/farm_plant_defs.lua#L265-L267)、[装配](../../../raw/current/scripts/prefabs/farm_plants.lua#L303-L322)、[SetupLoot](../../../raw/current/scripts/prefabs/farm_plants.lua#L344-L361)

产出分档：压力 NONE／LOW 为产物×1＋种子×2，MODERATE 为产物×1＋种子×1，HIGH 只出产物；巨型植株只出 `<name>_oversized`，非巨型腐烂株只出 `spoiled_food`。种子权重只作用于「随机种子长成哪种作物」：农田随机种子按 `seed_weight ×(适种季 ? 2 : 1)` 加权，通用种子经旧农田 `pickproduct` 抽取时只用基础权重，都不改变本键自身种子的掉落。[权重](../../../raw/current/scripts/tuning.lua#L97-L102)、[农田抽取](../../../raw/current/scripts/prefabs/farm_plants.lua#L134-L149)、[种子抽取](../../../raw/current/scripts/prefabs/seeds.lua#L43-L58)

| 生注册 | 农田植株 | 种子权重 | 适种季节 | 备注 |
|---|---|---|---|---|
| `eggplant` | `farm_plant_eggplant` | `UNCOMMON` 1 | 秋、春 | 生堆叠 20 |
| `durian` | `farm_plant_durian` | `RARE` 0.5 | 春 | 最低档，与石榴／火龙果并列 |
| `pomegranate` | `farm_plant_pomegranate` | `RARE` 0.5 | 春、夏 | — |
| `dragonfruit` | `farm_plant_dragonfruit` | `RARE` 0.5 | 春、夏 | 植株 `fireproof`，不进 `MakeSmallBurnable` |
| `watermelon` | `farm_plant_watermelon` | `UNCOMMON` 1 | 春、夏 | 生堆叠 20 |
| `tomato` | `farm_plant_tomato` | `COMMON` 2.5 | 秋、春、夏 | 生堆叠 40 |

适种季节与 `fireproof` 分别见 [farm_plant_defs.lua#L83-L100](../../../raw/current/scripts/prefabs/farm_plant_defs.lua#L83-L100) 与 [farm_plants.lua#L896-L902](../../../raw/current/scripts/prefabs/farm_plants.lua#L896-L902)。

番茄没有可移植或野生植株形态：`MakeVeggie` 只导出种子、巨型、本体与熟态，没有 `dug_tomato`／`tomato_planted` 一类注册 [导出](../../../raw/current/scripts/prefabs/veggies.lua#L1084-L1099)。Gorge（quagmire）模式自带的是独立注册 `quagmire_tomato`（`PRODUCT_VALUES` 没有 `prefab_override`，`MakeRawProduct` 用 `"quagmire_"..product` 起名），基座 `tomato` 只是借用它的图标；且只有 `TheNet:GetServerGameMode() == "quagmire"` 时才追加调用 `event_server_data("quagmire", "prefabs/veggies").master_postinit`，该事件文件不在本快照内，内容未核验。[产品表](../../../raw/current/scripts/prefabs/quagmire_plantables.lua#L55-L61)、[命名](../../../raw/current/scripts/prefabs/quagmire_plantables.lua#L349)、[钩子](../../../raw/current/scripts/prefabs/veggies.lua#L683-L685)、[event_server_data](../../../raw/current/scripts/entityscript.lua#L77-L87)

### 名称与显示

六键都不设 `displaynamefn`（只有 `_seeds` 设），显示名取 `STRINGS.NAMES.<大写注册名>`；简中 12 键均存在。

| 生／熟注册 | 简中显示名 | 名称证据 |
|---|---|---|
| `eggplant`／`eggplant_cooked` | 茄子／烤茄子 | [EGGPLANT、EGGPLANT_COOKED](../../../raw/current/scripts/languages/chinese_s.po#L342450-L342457) |
| `durian`／`durian_cooked` | 榴莲／超臭榴莲 | [DURIAN、DURIAN_COOKED](../../../raw/current/scripts/languages/chinese_s.po#L342395-L342402) |
| `pomegranate`／`pomegranate_cooked` | 石榴／切片熟石榴 | [POMEGRANATE、POMEGRANATE_COOKED](../../../raw/current/scripts/languages/chinese_s.po#L347120-L347127) |
| `dragonfruit`／`dragonfruit_cooked` | 火龙果／熟火龙果 | [DRAGONFRUIT、DRAGONFRUIT_COOKED](../../../raw/current/scripts/languages/chinese_s.po#L342215-L342222) |
| `watermelon`／`watermelon_cooked` | 西瓜／烤西瓜 | [WATERMELON](../../../raw/current/scripts/languages/chinese_s.po#L351800-L351802)、[WATERMELON_COOKED](../../../raw/current/scripts/languages/chinese_s.po#L351815-L351817) |
| `tomato`／`tomato_cooked` | 番茄／烤番茄 | [TOMATO、TOMATO_COOKED](../../../raw/current/scripts/languages/chinese_s.po#L350445-L350452) |

## 逐键机制

### 榴莲的怪物食性

榴莲的「怪物」属性落在两张表上，不能互相推断：

- **可食侧**：`edible.foodtype = FOODTYPE.VEGGIE` 不变，`secondaryfoodtype = FOODTYPE.MONSTER` 由 `VEGGIES.durian` 第 13 位传入，生熟两态都写，属性写入器据此补上 `edible_MONSTER` 标签（主类型仍是 `edible_VEGGIE`）。[表项](../../../raw/current/scripts/prefabs/veggies.lua#L68-L70)、[生写入](../../../raw/current/scripts/prefabs/veggies.lua#L595-L600)、[熟写入](../../../raw/current/scripts/prefabs/veggies.lua#L745-L750)、[标签](../../../raw/current/scripts/components/edible.lua#L9-L17)
- **烹饪侧**：`cooking.lua` 的食材表把 `durian` 单独登记为 `fruit=1, monster=1`（不走 `veggies` 列表），`cancook=true` 让 `durian_cooked` 继承同一组标签再加 `precook=1`。[食材表](../../../raw/current/scripts/cooking.lua#L95)、[派生规则](../../../raw/current/scripts/cooking.lua#L46-L72)

具体落到消费者：锅里的判定按「食材名对应的标签总量」累加后逐菜跑 `test(cooker, names, tags)`。[累加](../../../raw/current/scripts/cooking.lua#L217-L231)、[筛选](../../../raw/current/scripts/cooking.lua#L238-L266)

| 情形 | 结果与证据 |
|---|---|
| 一份菜里放两个以上榴莲 | `tags.monster >= 2` 且 `not tags.inedible` 时 `monsterlasagna` 成立（`priority = 10` 只参与通过后的排序）；加树枝（`inedible = 1`）即可挡掉。[monsterlasagna](../../../raw/current/scripts/preparedfoods.lua#L308-L321)、[树枝](../../../raw/current/scripts/cooking.lua#L134-L135) |
| 放一个榴莲 | 只让 `monster = 1`；`kabobs` 等 `monster <= 1` 的菜仍可成立，`not tags.monster` 的菜被排除。[kabobs](../../../raw/current/scripts/preparedfoods.lua#L145)、[not monster 示例](../../../raw/current/scripts/preparedfoods.lua#L459) |
| 以水果身份用 | `fruitmedley` 只要求 `fruit >= 3` 且无 meat／veggie，一个榴莲算 1 份水果；多加榴莲会同时抬高 `monster`，但两者门禁独立——两个榴莲＋一个石榴＋树枝仍可成果盘。[fruitmedley](../../../raw/current/scripts/preparedfoods.lua#L265-L279) |

进食侧另有一条：`edible_MONSTER` 参与「喂食」候选——目标带 `<TYPE>_eater` 才出 `FEED`，且还要 `critter`／`woby`／`handfed` 且为跟随者；把 `MONSTER` 写进 `SetDiet` 的只有 Woby 两形态与 `polly_rogers`，实际只有 Woby 满足。[喂食候选](../../../raw/current/scripts/componentactions.lua#L1146-L1163)、[wobysmall](../../../raw/current/scripts/prefabs/wobysmall.lua#L762)、[polly](../../../raw/current/scripts/prefabs/polly_rogers.lua#L240)。**边界**：`strongstomach` 认 `monstermeat` 标签，榴莲没有，故不减免 −3 生命／−5 理智。[判定](../../../raw/current/scripts/components/eater.lua#L166-L170)

### 西瓜生食降温

生西瓜在 `fn` 里写两个字段：`temperaturedelta = TUNING.COLD_FOOD_BONUS_TEMP`（−40）、`temperatureduration = TUNING.FOOD_TEMP_BRIEF`（5 秒）；熟西瓜整段没有这两个赋值，因此不降温。[生接线](../../../raw/current/scripts/prefabs/veggies.lua#L621-L624)、[常量](../../../raw/current/scripts/tuning.lua#L2327-L2330)

门禁与单位：[食用结算](../../../raw/current/scripts/components/edible.lua#L202-L208) 只在 `temperaturedelta ~= 0`、`duration ~= 0`、`chill < 1`（`chill` 只对升温食物累加）且吃者带 `temperature` 组件时调用 `temperature:SetTemperatureInBelly(delta, duration)`；该函数写入 `bellytemperaturedelta`／`bellytime` 并设到期任务，值随吃者存档（`OnSave` 写剩余时长）。[入胃](../../../raw/current/scripts/components/temperature.lua#L94-L101)、[存档](../../../raw/current/scripts/components/temperature.lua#L119-L131)。由于 `edible` 只在主模拟挂载（`fn` 在 `SetPristine` 后对客户端提前返回），这套计算只发生在服务端，客户端看到的是同步后的体温。[端分界](../../../raw/current/scripts/prefabs/veggies.lua#L589-L593)、[同步](../../../raw/current/scripts/components/temperature.lua#L7-L11)

**−40 不是「一口降 40 ℃」**。`Temperature:OnUpdate` 把它加进本帧的温差项 `self.delta`，且只在体温高于 `COLD_FOOD_CHILLING_THRESHOLD`（12）时加入；随后按环境与保温换算成速率：暖环境降温幅度上限 1／秒（`WARM_DEGREES_PER_SEC`），体温 ≥ `OVERHEAT_TEMP`（70）时 5／秒（`THAW_DEGREES_PER_SEC`），冷环境上限 `SEG_TIME/(SEG_TIME+保温)`；实际速率取 `max(delta, -上限)`，可以更慢。故降幅随体温与环境变化，5 秒窗口不是固定 5 ℃，`-40` 与 `-20` 也不总等效；熟西瓜没有这一段。[叠加](../../../raw/current/scripts/components/temperature.lua#L417-L423)、[夹取](../../../raw/current/scripts/components/temperature.lua#L488-L514)、[常量](../../../raw/current/scripts/tuning.lua#L2301-L2307)、[门槛](../../../raw/current/scripts/tuning.lua#L2329)

### 火龙果的万圣月变异

条件与构造：`VEGGIES.dragonfruit.halloweenmoonmutable_settings = {prefab = "fruitdragon", onmutatefn = function(inst, new_inst) new_inst:MakeRipe(true) end}`；主模拟据此挂 `halloweenmoonmutable`，`SetPrefabMutated("fruitdragon")`＋`SetOnMutateFn(...)`。只有生火龙果有这段，熟火龙果不参与。[表项](../../../raw/current/scripts/prefabs/veggies.lua#L76-L83)、[挂载](../../../raw/current/scripts/prefabs/veggies.lua#L672-L677)、[组件 API](../../../raw/current/scripts/components/halloweenmoonmutable.lua#L16-L22)

触发：本快照里 `Mutate()` 的唯一调用点是月亮精华液（`halloweenpotion_moon`）的 `Use`；动作候选只排除 `DECOR`；服务端执行另外拒绝空目标、燃烧／冒烟、冻结，药水用完按堆叠扣 1。未见月相或节日系统直接驱动该组件（检索范围：`scripts/` 全目录 grep，仅药水组件引用）。[药水 Use](../../../raw/current/scripts/components/halloweenpotionmoon.lua#L11-L30)、[动作实现](../../../raw/current/scripts/actions.lua#L4601-L4614)、[动作候选](../../../raw/current/scripts/componentactions.lua#L1340-L1344)、[药水注册](../../../raw/current/scripts/prefabs/halloweenpotion_moon.lua#L63-L64)

产物与 `onmutatefn`：`Mutate` 在原件位置 `SpawnPrefab(prefab)`，再以 `(原件, 新件)` 调回调。产物是生物「沙拉蝾螈」（`fruitdragon`）；`MakeRipe(true)` 的 `force` 让它跳过「是否待成熟」判定，直接置 `_is_ripe = true`、换 `fruit_dragon_ripe_build`、把掉落表切成 `fruit_dragon_ripe`（100% 掉 `dragonfruit`）、伤害改成 `RIPE_DAMAGE`；未成熟时是 `fruit_dragon` 表（掉 `plantmeat`）。[变异流程](../../../raw/current/scripts/components/halloweenmoonmutable.lua#L28-L79)、[MakeRipe](../../../raw/current/scripts/prefabs/fruitdragon.lua#L183-L193)、[掉落表](../../../raw/current/scripts/prefabs/fruitdragon.lua#L16-L24)、[暴露方法](../../../raw/current/scripts/prefabs/fruitdragon.lua#L463-L464)

原件处理与存档：堆叠时只 `Get()` 取 1 个移除；原件在容器／背包里时对新生物调该容器的 `GiveItem`——`fruitdragon` 没有 `inventoryitem`，物品栏直接拒绝（留在原地），普通容器落到容器坐标；无容器时才推 `attacked{attacker = nil}`。[替换分支](../../../raw/current/scripts/components/halloweenmoonmutable.lua#L62-L75) 组件无存档逻辑，但生火龙果读档重建时会重新挂上，所以**老存档里的火龙果同样可被药水变异**；变异不会在读档时自动重放，只有被药水选中才发生。[组件无 OnSave](../../../raw/current/scripts/components/halloweenmoonmutable.lua#L1-L26)

### 番茄的 Gorge 图标

条件只是一个硬编码名字表：`QUAGMIRE_PORTS = {"tomato", "onion"}`，工厂在 `MakeVeggie` 执行时算出 `usequagmireicon`，与当前游戏模式、季节、活动无关。[名字表](../../../raw/current/scripts/prefabs/veggies.lua#L33-L37)、[判定](../../../raw/current/scripts/prefabs/veggies.lua#L413-L417)

影响面仅限背包图标：生件 `ChangeImageName("quagmire_tomato")`、熟件 `ChangeImageName("quagmire_tomato_cooked")`，写入的是 `inventoryitem.imagename` 并经 `replica.inventoryitem:SetImage` 同步成 `classified.image`；世界动画仍是 `tomato` 的 bank／build，名称、三维、新鲜度、堆叠都不变。两份 INV_IMAGE 资源由工厂按需加进 `assets`。[生件](../../../raw/current/scripts/prefabs/veggies.lua#L641-L643)、[熟件](../../../raw/current/scripts/prefabs/veggies.lua#L757-L759)、[赋值](../../../raw/current/scripts/components/inventoryitem.lua#L470-L473)、[同步](../../../raw/current/scripts/components/inventoryitem.lua#L5-L7)

`quagmire_tomato` 是 Gorge 自己的注册，不是同一物品的别名；基座番茄只共用图标。[关联注册](../../../raw/current/scripts/prefabs/quagmire_plantables.lua#L289-L349)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：edible | 写三维、主食性 `FOODTYPE.VEGGIE` 与次要食性；赋值即重跑 `badfood` 与 `edible_*` 标签 | 生熟各一套；主模拟 | [[reference/components/edible#关键接口]]；[生](../../../raw/current/scripts/prefabs/veggies.lua#L595-L600)、[熟](../../../raw/current/scripts/prefabs/veggies.lua#L745-L750) |
| 组件：perishable | `SetPerishTime`＋`StartPerishing`，腐坏替换 `spoiled_food`；上限按秒计 | 生用 `perishtime`、熟用 `cooked_perishtime` | [[reference/components/perishable#关键接口]]；[生](../../../raw/current/scripts/prefabs/veggies.lua#L602-L605)、[熟](../../../raw/current/scripts/prefabs/veggies.lua#L740-L743)、[单位](../../../raw/current/scripts/components/perishable.lua#L188-L194) |
| 组件：stackable | 生件按例外名单写 40 或走默认 20，熟件恒 40；上限同步 replica | 生熟不同档，见[数值表](#生熟两态数值) | [[reference/components/stackable#关键接口]]；[例外](../../../raw/current/scripts/prefabs/veggies.lua#L613-L619)、[默认](../../../raw/current/scripts/components/stackable.lua#L23-L28) |
| 组件：cookable | 仅生件：`product = name.."_cooked"`；熟件不能再烹饪 | 生件、主模拟 | [[reference/components/cookable#关键接口]]；[接线](../../../raw/current/scripts/prefabs/veggies.lua#L664-L665)、[Cook](../../../raw/current/scripts/components/cookable.lua#L18-L38) |
| 组件：weighable | 生件挂 `TROPHYSCALE_TYPES.OVERSIZEDVEGGIES` 但不给重量，只让物品能放称重台；公共端预置 `weighable_OVERSIZEDVEGGIES` tag | 仅生件；熟件无 | [[reference/components/weighable#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/veggies.lua#L645-L651)、[tag](../../../raw/current/scripts/prefabs/veggies.lua#L567-L570) |
| 组件：snowmandecor | 空标记组件，冬季雪人装饰行为在 `componentactions` 侧接入 | 仅 `eggplant` 生件 | [[reference/components/snowmandecor]]；[挂载](../../../raw/current/scripts/prefabs/veggies.lua#L679-L681) |
| 组件：halloweenmoonmutable | `prefab_mutated = "fruitdragon"`，被月亮精华液选中时整件换生物 | 仅 `dragonfruit` 生件；主模拟 | [[reference/components/halloweenmoonmutable#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/veggies.lua#L672-L677)、[Mutate](../../../raw/current/scripts/components/halloweenmoonmutable.lua#L28-L83) |
| API／数据：inventoryitem.imagename | 生熟图标换 Gorge 版并同步 replica | 仅 `tomato`（同表还有 onion，见 [[reference/prefabs/farm-crops-spice]]） | [替换](../../../raw/current/scripts/prefabs/veggies.lua#L641-L643)、[同步](../../../raw/current/scripts/components/inventoryitem.lua#L5-L7) |
| API／数据：烹饪食材量 | 六键在锅里的身份：`eggplant`／`tomato`＝`veggie 1`；`pomegranate`／`dragonfruit`／`watermelon`＝`fruit 1`；`durian`＝`fruit 1`＋`monster 1` | 生熟同表，见[榴莲的怪物食性](#榴莲的怪物食性) | [食材表](../../../raw/current/scripts/cooking.lua#L85-L101)、[西瓜](../../../raw/current/scripts/cooking.lua#L141) |
| 关联 Prefab：`fruitdragon` | 火龙果变异的产物：生物，成熟时掉 `dragonfruit`、未成熟掉 `plantmeat` | 仅 `dragonfruit` 被药水命中时生成 | [工厂](../../../raw/current/scripts/prefabs/fruitdragon.lua#L375-L493)、[掉落](../../../raw/current/scripts/prefabs/fruitdragon.lua#L16-L24) |
| 关联 Prefab：`<name>_oversized*` | 巨型与腐烂形态，本页六键各有三件 | 巨型植株收获或巨型腐坏时生成 | [[reference/prefabs/giant-crops]]；[导出](../../../raw/current/scripts/prefabs/veggies.lua#L1086-L1091) |
| 关联 Prefab：`farm_plant_<name>` | 常规来源植株，`grow_time` 与适种季节取自 `PLANT_DEFS` | 世界实体；种植见 [[reference/components/farmplantable]] | [定义循环](../../../raw/current/scripts/prefabs/farm_plants.lua#L930-L935)、[字段](../../../raw/current/scripts/prefabs/farm_plant_defs.lua#L21-L57) |

## 典型接入

### 改熟番茄的保鲜与生西瓜的降温窗口

- 前提与选择：服务端 `modmain.lua`；两件独立注册，须分别 postinit，不能回灌数据表。`SetNewMaxPerishTime` 只对已有剩余按比例换算（postinit 时满鲜，与 `SetPerishTime` 同效）；降温只动 `edible` 字段。
- 生效与撤销：`AddPrefabPostInit` 在每次构造（含读档重建）后执行；停用模组即恢复工厂原值。`temperaturedelta`／`temperatureduration` 是普通字段，赋值不会重排已在生效的 `bellytime`。
- 原版依据：番茄熟件 10 天反而比生件 6 天长，见[数值表](#生熟两态数值)；降温字段与门禁见[西瓜生食降温](#西瓜生食降温)。[`SetNewMaxPerishTime`](../../../raw/current/scripts/components/perishable.lua#L204-L208)

```lua
-- modmain.lua：熟番茄改回 6 天（2880 秒），生西瓜的降温窗口减半
AddPrefabPostInit("tomato_cooked", function(inst)
    if GLOBAL.TheWorld.ismastersim and inst.components.perishable ~= nil then
        inst.components.perishable:SetNewMaxPerishTime(6 * GLOBAL.TUNING.TOTAL_DAY_TIME)
    end
end)

AddPrefabPostInit("watermelon", function(inst)
    if GLOBAL.TheWorld.ismastersim and inst.components.edible ~= nil then
        inst.components.edible.temperatureduration = 0.5 * GLOBAL.TUNING.FOOD_TEMP_BRIEF
    end
end)
```

- 调整与验证：单位分别是个／秒；实际降温是 `delta` 再经环境与保温换算，暖环境下未过热／过热上限为 1／5 ℃/秒，缩短窗口只等比缩短生效时长，不承诺固定降幅。改保鲜只影响此后构造的实例；读档实例先跑 postinit，再被 `Perishable:OnLoad` 用存档剩余秒数覆盖，比例取 `min(1, 旧剩余÷新上限)`（原剩余超过新上限时按满鲜封顶）。[OnLoad](../../../raw/current/scripts/components/perishable.lua#L337-L349)、[GetPercent](../../../raw/current/scripts/components/perishable.lua#L215-L221)

## 常见陷阱与限制

- **边界：生茄、生榴莲、生西瓜的 20 不是显式配置**：工厂只对这四个名字（含 `pumpkin`）跳过 `maxsize`，值来自 `stackable` 构造默认；改默认会连带改它们。[例外](../../../raw/current/scripts/prefabs/veggies.lua#L613-L619)、[默认](../../../raw/current/scripts/components/stackable.lua#L23-L28)
- **边界：熟制不总是更耐放**：`tomato_cooked` 由 6 天升到 10 天，其余五键熟后缩短（石榴／火龙果／西瓜只剩 3 天）；按直觉配货会算错。[表项](../../../raw/current/scripts/prefabs/veggies.lua#L60-L126)
- **误区：榴莲的「怪物」不是一个标签就能查**：锅里看的是 `cooking.lua` 食材表（`fruit 1`＋`monster 1`），可食侧看 `secondaryfoodtype`（`edible_MONSTER` 供喂食候选），两者互不推导；榴莲没有 `monstermeat` 标签，所以 `strongstomach` 不减免榴莲的负值。[食材表](../../../raw/current/scripts/cooking.lua#L95)、[可食侧](../../../raw/current/scripts/prefabs/veggies.lua#L600)、[strongstomach](../../../raw/current/scripts/components/eater.lua#L166-L170)
- **误区：西瓜的 −40 不是 −40 ℃**：它只是温差项，速率再经环境与保温换算（暖环境下未过热／过热上限 1／5 ℃/秒），体温 ≤12 后不再生效；熟西瓜没有这段。[门禁](../../../raw/current/scripts/components/temperature.lua#L417-L423)、[夹取](../../../raw/current/scripts/components/temperature.lua#L488-L514)
- **边界：火龙果变异由药水驱动，不是月相**：本快照中没有世界月相／节日系统调用该组件；变异产物是生物且被 `onmutatefn` 强制成熟，堆叠只转 1 个。见[火龙果的万圣月变异](#火龙果的万圣月变异)。
- **边界：番茄图标与活动模式无关**：`QUAGMIRE_PORTS` 是名字表，任何存档都会换图标；Gorge 内的 `quagmire_tomato` 是另一注册，不能当作同一物品来源。见[番茄的 Gorge 图标](#番茄的-gorge-图标)。
- **边界：`weighable` 只是称重台标记**：普通蔬菜不设重量，放上巨型称重台只显示同一结果并替换其他普通蔬菜。[注释与挂载](../../../raw/current/scripts/prefabs/veggies.lua#L645-L651)
- **未核验**：月亮精华液是否只在万圣节活动期间可造（台站与活动门禁未核验）、`quagmire_event_server` 下番茄的实际行为（文件不在快照）。[配方](../../../raw/current/scripts/recipes.lua#L1420)

## 验证记录

- 源码审阅：2026-11-08，Steam buildid 26335394。核对 `veggies.lua` 全文（`VEGGIES` 六键实参、`fn`／`fn_cooked`、图标替换、堆叠例外、降温、变异与导出循环）、`farm_plants.lua` 的选种与战利品、`farm_plant_defs.lua` 的字段与适种季节、`cooking.lua`／`preparedfoods.lua` 的食材分类与菜品判定、`halloweenmoonmutable`／`halloweenpotionmoon`／`fruitdragon.lua` 的变异链、`actions.lua`／`componentactions.lua` 的动作与喂食候选、`quagmire_plantables.lua` 的番茄注册、`seeds.lua` 的加权、`tuning.lua` 的基准与常量、`constants.lua` 的 `FOODTYPE`、`edible`／`eater`／`temperature`／`perishable`／`stackable`／`cookable`／`inventoryitem` 的相关分支、简中 12 个名称键。
  未覆盖：`fruitdragon` 的战斗与睡眠状态机、`fruit_dragon_ripe` 掉率的运行时表现、月亮精华液的活动门禁、`quagmire_event_server` 事件数据；其它生成点、动态选名与模组注入未穷举。
- 游戏运行：未运行。三维、天数与秒数均为 tuning 推导（`CALORIES_TINY`＝75/8＝9.375、`PERISH_FAST`＝6×480 秒），降温是速率夹取推导，不是运行测量。
- 结构检查：`validate_page.py` 通过（203 行／32,673 字节，standard 档内）；`validate_source_links.py --strict` 0 错 0 警（135 处锚点）。

## 相关链接

- 同一工厂的另外两类：[[reference/prefabs/farm-crops-staple]]（共用工厂与生熟两态）、[[reference/prefabs/farm-crops-spice]]（辛香 4 键）
- 巨型形态与打蜡：[[reference/prefabs/giant-crops]]
- 种植与生长：[[reference/components/crop]]、[[reference/components/farmplantable]]、[[reference/components/farming-manager]]

返回 [[reference/prefabs/index|Prefab 索引]]。
