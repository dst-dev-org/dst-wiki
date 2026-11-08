---
title: "辛香作物（asparagus、onion、garlic、pepper 及各自 _cooked）"
created: "2026-11-08"
updated: "2026-11-08"
type: prefab
summary: "四个农田辛香作物与烤制形态共 8 个注册：三维与新鲜度取值、onion 的 Gorge 图标替换、asparagus／pepper 的雪人装饰接线、40 上限堆叠、简中名称键，以及农田收获与种子权重给出的获得路径。"
aliases: ["asparagus", "asparagus_cooked", "onion", "onion_cooked", "garlic", "garlic_cooked", "pepper", "pepper_cooked", "芦笋", "烤芦笋", "洋葱", "烤洋葱", "大蒜", "烤大蒜", "辣椒", "烤辣椒", "辛香作物", "农田蔬菜"]
tags: [prefab, item, food]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/veggies.lua", "tuning.lua", "components/edible.lua", "components/eater.lua", "components/stackable.lua", "components/inventoryitem.lua", "components/pickable.lua", "components/snowmandecoratable.lua", "componentactions.lua", "entityscript.lua", "standardcomponents.lua", "prefabs/farm_plants.lua", "prefabs/birdcage.lua", "prefabs/farm_plant_defs.lua", "prefabs/snowman.lua", "prefabs/warly.lua", "prefabs/pigking.lua", "prefabs/quagmire_plantables.lua", "cooking.lua", "preparedfoods.lua", "recipes.lua", "languages/loc.lua", "languages/chinese_s.po", "languages/strings.pot"]
reliability: verified
validation: source-reviewed
status: active
---

# 辛香作物（asparagus、onion、garlic、pepper 及各自 _cooked）

## 快速判断

- 职责与适用：四个农田辛香作物（芦笋、洋葱、大蒜、辣椒）各有一对注册——生形态与烤制形态（`<name>_cooked`），合计 8 个，都是 `inventoryitem` 食物，食用类型统一 `FOODTYPE.VEGGIE` 且无次要类型。四个键的取向不同：asparagus 无负面收益、熟制后饥饿翻倍；onion／garlic 生吃只压理智；pepper 生吃同时掉生命与理智。可借鉴：同一张数值表驱动生熟两态、按名字做单点分支（图标、雪人装饰），以及"高惩罚食材靠烹饪改善"的取值方式。[生／熟注册](../../../raw/current/scripts/prefabs/veggies.lua#L1096-L1097)、[遍历注册](../../../raw/current/scripts/prefabs/veggies.lua#L1102-L1110)
- 优先使用：查三维、保鲜与堆叠取[数值表](#数值与参数)；生熟两态的公共构造、堆叠与新鲜度通则、食用与烹饪接线见 [[reference/prefabs/farm-crops-staple#共用工厂与生熟两态]]；获得链的压力分档见 [[reference/components/farmplantstress]]。
- 重要副作用：onion 的背包图标被无条件换成 Gorge 的 `quagmire_onion`／`quagmire_onion_cooked`（与活动模式无关）；asparagus 与 pepper 可当雪人装饰材料；四键都挂 `tradable` 但未设 `goldvalue`，猪王不收；pepper 生吃一次扣 20 生命与 15 理智（新鲜态、无额外修正）。
- 修改边界：本页不含 `<name>_seeds`（种子另属独立注册，见 [[reference/prefabs/farm-crop-seeds|作物种子页]]与 [[reference/prefabs/generic-seeds|通用种子页]]）、`<name>_oversized` 系形态（见 [[reference/prefabs/giant-crops]]）与种植／生长过程（见 [[reference/components/crop]]、[[reference/components/farmplantable]]、[[reference/components/farming-manager]]）。

## 注册与构造

8 个注册由 `MakeVeggie(name, has_seeds)` 在同一个工厂内生成两份闭包：生形态 `fn`（[veggies.lua#L539-L705](../../../raw/current/scripts/prefabs/veggies.lua#L539-L705)）与烤制形态 `fn_cooked`（[#L707-L777](../../../raw/current/scripts/prefabs/veggies.lua#L707-L777)）。公共端装配、`SetPristine` 端分界、两态组件集合的逐项差异与堆叠／新鲜度通则由 [[reference/prefabs/farm-crops-staple#共用工厂与生熟两态]] 承接，此处只列本页四键的差异来源。

表中 floater 参数按 `MakeInventoryFloatable(inst, size, offset, scale)` 的位序解释，尺寸缺省 small（[standardcomponents.lua#L1772-L1791](../../../raw/current/scripts/standardcomponents.lua#L1772-L1791)）。

逐键差异只有三类：`VEGGIES` 表的实参（三维、保鲜、种子权重、漂浮参数）、`QUAGMIRE_PORTS` 的图标替换（仅 onion，同表还有 tomato）、`issnowmandecor`（asparagus 与 pepper）。三者都在工厂里按名字读取，本页四键都没有专属分支。

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `asparagus` | `fn`；表项取 `UNCOMMON` 种子权重与 `CALORIES_SMALL`／`HEALING_SMALL`／`PERISH_FAST` | floater 参数 `("med", nil, 0.7)`（尺寸 med、缩放 0.7）；挂 `snowmandecor`。[表项](../../../raw/current/scripts/prefabs/veggies.lua#L132-L138)、[漂浮参数](../../../raw/current/scripts/prefabs/veggies.lua#L578-L582)、[雪人装饰](../../../raw/current/scripts/prefabs/veggies.lua#L679-L680) |
| `asparagus_cooked` | `fn_cooked`；读同一表项的 `cooked_*` 半段（`CALORIES_MED`／`HEALING_SMALL`／`PERISH_SUPERFAST`） | 动画沿用本作物 bank／build 的 `cooked`；无漂浮参数、无 `snowmandecor`、无 `cookable`。[熟工厂](../../../raw/current/scripts/prefabs/veggies.lua#L707-L732)、[表项](../../../raw/current/scripts/prefabs/veggies.lua#L132-L138) |
| `onion` | `fn`；`RARE` 权重、`CALORIES_TINY`、0 生命、`-SANITY_SMALL`、`PERISH_SLOW` | floater `("large", 0.05, 0.45)`（large 尺寸但缩放 0.45）；`QUAGMIRE_PORTS` 命中，图标换成 `quagmire_onion`。[表项](../../../raw/current/scripts/prefabs/veggies.lua#L140-L142)、[浮标参数](../../../raw/current/scripts/prefabs/veggies.lua#L578-L582)、[图标替换](../../../raw/current/scripts/prefabs/veggies.lua#L641-L643) |
| `onion_cooked` | `fn_cooked`；`CALORIES_TINY`／`HEALING_TINY`／`-SANITY_TINY`／`PERISH_MED` | 熟形态同样换图标，用 `quagmire_onion_cooked`。[图标替换](../../../raw/current/scripts/prefabs/veggies.lua#L757-L759) |
| `garlic` | `fn`；与 onion 同档（`RARE`、9.375、0 生命、−10 理智、15 天） | floater `(nil, 0.05, 0.775)`（尺寸回落 small、偏移 0.05）；不在 `QUAGMIRE_PORTS`，不挂雪人装饰。[表项](../../../raw/current/scripts/prefabs/veggies.lua#L144-L146)、[名字表](../../../raw/current/scripts/prefabs/veggies.lua#L33-L37) |
| `garlic_cooked` | `fn_cooked`；与 `onion_cooked` 同档 | 图标保留 `garlic_cooked`，无额外分支。 |
| `pepper` | `fn`；`RARE`、`-HEALING_MED`（−20）、`-SANITY_MED`（−15）、`PERISH_SLOW` | floater `(nil, 0.1, 0.75)`；挂 `snowmandecor`。[表项](../../../raw/current/scripts/prefabs/veggies.lua#L148-L154)、[雪人装饰](../../../raw/current/scripts/prefabs/veggies.lua#L679-L680) |
| `pepper_cooked` | `fn_cooked`；`-HEALING_SMALL`（−3）、`-SANITY_SMALL`（−10）、`PERISH_SLOW` | 熟制后保鲜期与生形态相同（15 天），是本页唯一不缩短保鲜的组合。[表项](../../../raw/current/scripts/prefabs/veggies.lua#L148-L154) |

## 数值与参数

基准写在 [tuning.lua](../../../raw/current/scripts/tuning.lua)：一天 `total_day_time`＝480 秒（[#L19-L20](../../../raw/current/scripts/tuning.lua#L19-L20)）、`perish_warp`＝1（[#L47](../../../raw/current/scripts/tuning.lua#L47)），故 `PERISH_FAST`＝6×480、`PERISH_SUPERFAST`＝3×480、`PERISH_MED`＝10×480、`PERISH_SLOW`＝15×480（[#L2043-L2051](../../../raw/current/scripts/tuning.lua#L2043-L2051)）；`calories_per_day`＝75（[#L41](../../../raw/current/scripts/tuning.lua#L41)），`CALORIES_TINY`＝75/8、`CALORIES_SMALL`＝75/6、`CALORIES_MED`＝75/3（[#L2058-L2062](../../../raw/current/scripts/tuning.lua#L2058-L2062)），`HEALING_TINY`＝1、`HEALING_SMALL`＝3、`HEALING_MED`＝20（[#L2021-L2025](../../../raw/current/scripts/tuning.lua#L2021-L2025)），`SANITY_TINY`＝5、`SANITY_SMALL`＝10、`SANITY_MED`＝15（[#L2033-L2035](../../../raw/current/scripts/tuning.lua#L2033-L2035)）。

| 注册名 | 饥饿 | 生命 | 理智 | 新鲜度 | 堆叠上限 | 备注 |
|---|---|---:|---:|---|---:|---|
| `asparagus` | 12.5 | 3 | 0 | 6 天（2880 秒） | 40 | 唯一无负面收益的键 |
| `asparagus_cooked` | 25 | 3 | 0 | 3 天（1440 秒） | 40 | 饥饿翻倍、保鲜减半 |
| `onion` | 9.375 | 0 | −10 | 15 天（7200 秒） | 40 | Gorge 图标 |
| `onion_cooked` | 9.375 | 1 | −5 | 10 天（4800 秒） | 40 | 惩罚减半、反而回 1 生命 |
| `garlic` | 9.375 | 0 | −10 | 15 天（7200 秒） | 40 | 与 onion 同档 |
| `garlic_cooked` | 9.375 | 1 | −5 | 10 天（4800 秒） | 40 | 与 `onion_cooked` 同档 |
| `pepper` | 9.375 | −20 | −15 | 15 天（7200 秒） | 40 | 生吃代价最高 |
| `pepper_cooked` | 9.375 | −3 | −10 | 15 天（7200 秒） | 40 | 熟制仍为负，只是幅度变小 |

堆叠上限 40 来自 `STACK_SIZE_SMALLITEM`（[tuning.lua#L83](../../../raw/current/scripts/tuning.lua#L83)）：生形态在 `stackable` 之后按黑名单（pumpkin、eggplant、durian、watermelon 走组件默认 20）决定是否写入 40，本页四键都不在黑名单里；熟形态无条件写 40（[生形态分支](../../../raw/current/scripts/prefabs/veggies.lua#L613-L618)、[熟形态](../../../raw/current/scripts/prefabs/veggies.lua#L752-L753)）。还原到组件默认值时上限是 `STACK_SIZE_MEDITEM`＝20（[stackable.lua#L23-L28](../../../raw/current/scripts/components/stackable.lua#L23-L28)）。

吃下的实际结算走 `edible:GetHealth`／`GetSanity`／`GetHunger`：普通吃者路径下负值原样返回，该物品满足 `CanProcessSpoiledItem` 的吃者会先把负值钳到 0（[生命取值](../../../raw/current/scripts/components/edible.lua#L121-L142)、[理智取值](../../../raw/current/scripts/components/edible.lua#L72-L93)、[钳零](../../../raw/current/scripts/components/edible.lua#L127-L128)），再由食用者的 `eater` 按 `health_delta`／`sanity_delta` 结算，且仅当 `DoFoodEffects` 为真才应用负值（[eater.lua#L211-L235](../../../raw/current/scripts/components/eater.lua#L211-L235)、[判断](../../../raw/current/scripts/components/eater.lua#L166-L170)）。以 150 生命／200 理智的威尔逊为例（[tuning.lua#L38-L40](../../../raw/current/scripts/tuning.lua#L38-L40)），新鲜态且无额外修正时，生吃一个 pepper 扣 20 生命（约 13%）与 15 理智，烤过后降到 3 生命与 10 理智；onion 与 garlic 生吃只压 10 理智，烤过后压 5 理智并回 1 生命。

### 名称与显示

| 注册名 | 简中显示名 | 证据 |
|---|---|---|
| `asparagus` | 芦笋 | [chinese_s.po#L338890-L338892](../../../raw/current/scripts/languages/chinese_s.po#L338890-L338892) |
| `asparagus_cooked` | 烤芦笋 | [chinese_s.po#L338900-L338902](../../../raw/current/scripts/languages/chinese_s.po#L338900-L338902) |
| `onion` | 洋葱 | [chinese_s.po#L346640-L346642](../../../raw/current/scripts/languages/chinese_s.po#L346640-L346642) |
| `onion_cooked` | 烤洋葱 | [chinese_s.po#L346645-L346647](../../../raw/current/scripts/languages/chinese_s.po#L346645-L346647) |
| `garlic` | 大蒜 | [chinese_s.po#L343110-L343112](../../../raw/current/scripts/languages/chinese_s.po#L343110-L343112) |
| `garlic_cooked` | 烤大蒜 | [chinese_s.po#L343115-L343117](../../../raw/current/scripts/languages/chinese_s.po#L343115-L343117) |
| `pepper` | 辣椒 | [chinese_s.po#L346770-L346772](../../../raw/current/scripts/languages/chinese_s.po#L346770-L346772) |
| `pepper_cooked` | 烤辣椒 | [chinese_s.po#L346780-L346782](../../../raw/current/scripts/languages/chinese_s.po#L346780-L346782) |

取用链：实体构造时 `SetPrefabName` 用 `STRINGS.NAMES[大写预置名]` 赋给 `self.name`，取不到才回退字面量 `"MISSING NAME"`（[entityscript.lua#L840-L844](../../../raw/current/scripts/entityscript.lua#L840-L844)）；`GetBasicDisplayName` 依次看 `displaynamefn`、`nameoverride`、`self.name`（[#L687-L692](../../../raw/current/scripts/entityscript.lua#L687-L692)）。蔬菜两态都不设 `displaynamefn`，所以显示名就是上述 `STRINGS.NAMES` 键，八个键在简中表里全部存在；简中加载的字符串文件就是 `chinese_s.po`（`strings = "chinese_s.po"`，[loc.lua#L15-L17](../../../raw/current/scripts/languages/loc.lua#L15-L17)）。`strings.pot` 是未翻译模板，全表 `msgstr` 为空（`PEPPER_COOKED` 本身也是空串），不能据它判断简中缺译（[模板条目](../../../raw/current/scripts/languages/strings.pot#L346776-L346779)）。表内八键按 `chinese_s.po` 的 `msgstr` 给出；本页未运行游戏，界面上的实际渲染与字体度量未做运行时确认。

## 获得路径

| 键 | 直接来源 | 条件与证据 |
|---|---|---|
| `asparagus`、`onion`、`garlic`、`pepper` | 各自 `farm_plant_<name>` 成熟植株的收获产物 | `plant_def.product` 就等于作物名，种子另为 `<name>_seeds`。[转换](../../../raw/current/scripts/prefabs/farm_plant_defs.lua#L254-L267)、[注册](../../../raw/current/scripts/prefabs/farm_plants.lua#L926) |

收获链：对成熟植株执行 PICK → `pickable:Pick` → 生成战利品，玩家有可用物品栏时尝试直入栏、否则落地 → `remove_when_picked` 移除植株（[farm_plants 的 pickable 装配](../../../raw/current/scripts/prefabs/farm_plants.lua#L308-L315)、[Pick](../../../raw/current/scripts/components/pickable.lua#L551-L579)）。战利品内容由植株压力在收获时决定：

- 压力 NONE／LOW：作物×1＋种子×2；
- 压力 MODERATE：作物×1＋种子×1；
- 压力 HIGH：只掉作物；
- 巨型植株走 `<name>_oversized`（另页），非巨型腐烂植株掉 spoiled_food。

战利品表由 `SetupLoot` 生成，经 `lootdropper.lootsetupfn` 挂到植株上（[SetupLoot](../../../raw/current/scripts/prefabs/farm_plants.lua#L344-L359)、[挂接](../../../raw/current/scripts/prefabs/farm_plants.lua#L873-L874)）。NONE 压力在成熟阶段通常会把植株判成巨型，未标巨型的 NONE 只在 `no_oversized` 等情形出现，因此常规作物拿到 2 颗种子的常见档位是 LOW（[成熟判定](../../../raw/current/scripts/prefabs/farm_plants.lua#L518)）。

种子稀有度由 `VEGGIES` 表首列决定：asparagus 为 `UNCOMMON`＝1，onion／garlic／pepper 为 `RARE`＝0.5，对比 `COMMON`＝2.5（[权重常量](../../../raw/current/scripts/tuning.lua#L97-L100)、[本地别名](../../../raw/current/scripts/prefabs/veggies.lua#L25-L27)）。该权重用于"随机种子长成哪种作物"的加权：按 `seed_weight × (当季是否适种 ? SEED_WEIGHT_SEASON_MOD : 1)` 取值（[季节系数](../../../raw/current/scripts/tuning.lua#L102)、[加权选择](../../../raw/current/scripts/prefabs/farm_plants.lua#L134-L146)），适种季节见 [farm_plant_defs.lua#L90-L99](../../../raw/current/scripts/prefabs/farm_plant_defs.lua#L90-L99)。所以用通用种子赌出这四种的机会低于基础粮菜。种子还有第二条来路：把本页作物喂给普通鸟笼的鸟，会按 `<prefab>_seeds` 产出对应种子（[鸟笼产出](../../../raw/current/scripts/prefabs/birdcage.lua#L135-L137)、[收取规则](../../../raw/current/scripts/prefabs/birdcage.lua#L155-L170)）。

其他来源与边界：

- `garlic` 是 Warly 在默认模式下的开局物品之一：`GAMEMODE_STARTING_ITEMS.DEFAULT.WARLY` 为 `{"portablecookpot_item", "potato", "potato", "garlic"}`（[tuning.lua#L3450-L3471](../../../raw/current/scripts/tuning.lua#L3450-L3471)），角色侧按模式取表（[warly.lua#L10-L15](../../../raw/current/scripts/prefabs/warly.lua#L10-L15)）。
- Gorge（quagmire）模式自带的 `quagmire_garlic`／`quagmire_onion` 是事件自己注册的独立预置物（名字为 `"quagmire_"..product`），不是本页四键，不能当同一物品的另一个来源（[产物段](../../../raw/current/scripts/prefabs/quagmire_plantables.lua#L344-L349)、[产品表](../../../raw/current/scripts/prefabs/quagmire_plantables.lua#L23-L37)）。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：edible | 写入三维，食用类型 `FOODTYPE.VEGGIE`；两态都写 `secondaryfoodtype`（本页四键为 nil） | 两态；主模拟 | [[reference/components/edible#关键接口]]；[生形态](../../../raw/current/scripts/prefabs/veggies.lua#L595-L600)、[熟形态](../../../raw/current/scripts/prefabs/veggies.lua#L745-L750) |
| 组件：perishable | 保鲜期取表项，腐坏替换 spoiled_food | 两态；主模拟 | [[reference/components/perishable#关键接口]]；[生形态](../../../raw/current/scripts/prefabs/veggies.lua#L602-L605)、[熟形态](../../../raw/current/scripts/prefabs/veggies.lua#L740-L743) |
| 组件：stackable | 生形态按黑名单写 40、熟形态固定 40；组件默认 20 | 两态 | [[reference/components/stackable#关键接口]]；[生形态](../../../raw/current/scripts/prefabs/veggies.lua#L613-L618)、[熟形态](../../../raw/current/scripts/prefabs/veggies.lua#L752-L753) |
| 组件：cookable | 生形态 `product = <name>_cooked`；熟形态不挂，不能二次烹饪 | 仅生形态 | [[reference/components/cookable#关键接口]]；[接线](../../../raw/current/scripts/prefabs/veggies.lua#L664-L665) |
| 组件：weighable | 生形态挂 `TROPHYSCALE_TYPES.OVERSIZEDVEGGIES` 但只设类型、不设重量值；工厂注释说明称重台对普通蔬菜只显示同一结果，且放置时会替换其他普通蔬菜 | 仅生形态（熟形态未挂） | [[reference/components/weighable#关键接口]]；[挂接](../../../raw/current/scripts/prefabs/veggies.lua#L645-L651) |
| 组件：snowmandecor | asparagus／pepper 生形态挂标记组件；装饰外观另需 `snowmandecoratable` 的本地表条目 | 仅这两个键、仅生形态 | [[reference/components/snowmandecor]]；[分支](../../../raw/current/scripts/prefabs/veggies.lua#L679-L680)、[asparagus 条目](../../../raw/current/scripts/components/snowmandecoratable.lua#L6)、[pepper 条目](../../../raw/current/scripts/components/snowmandecoratable.lua#L35) |
| API／数据：inventoryitem.imagename | onion 两态换 Gorge 图标并同步 replica | onion（`QUAGMIRE_PORTS`） | [替换](../../../raw/current/scripts/prefabs/veggies.lua#L641-L643)、[同步](../../../raw/current/scripts/components/inventoryitem.lua#L5-L7) |
| API／数据：tradable | 两态都挂，但未设 `goldvalue`；猪王 `AcceptTest` 要求 `goldvalue > 0` | 两态 | [[reference/components/tradable#关键接口]]；[猪王判定](../../../raw/current/scripts/prefabs/pigking.lua#L598-L606) |
| API／数据：烹饪食材 | 四键都记为 `veggie=1`，熟形态同时以 `<name>_cooked` 登记并带 `precook=1`；asparagus 参与芦笋汤、pepper 参与填馅辣椒的判定 | 生熟形态都可下锅 | [食材表](../../../raw/current/scripts/cooking.lua#L100-L101)、[芦笋汤](../../../raw/current/scripts/preparedfoods.lua#L507-L509)、[填馅辣椒](../../../raw/current/scripts/preparedfoods.lua#L615-L617) |
| API／数据：配方消费 | `spice_garlic` 吃 3 个 garlic、`spice_chili` 吃 3 个 pepper；两个小动物建筑配方各吃 1 个 onion／garlic | 服务器配方表 | [蒜香料](../../../raw/current/scripts/recipes.lua#L101)、[辣椒香料](../../../raw/current/scripts/recipes.lua#L103)、[建造配方](../../../raw/current/scripts/recipes.lua#L828-L829) |
| 关联 Prefab：`farm_plant_<name>` | 收获来源植株；成熟与腐烂阶段挂 `pickable` | 世界实体 | [[reference/components/pickable#关键接口]]；[pickable 装配](../../../raw/current/scripts/prefabs/farm_plants.lua#L308-L315) |

其余工厂共通组件（`bait`、可燃体与蔓延、可作祟、`tradable` 之外的同批接线）与两态公共端顺序不在此重复，见 [[reference/prefabs/farm-crops-staple#共用工厂与生熟两态]]。

## 典型接入

### 减半生辣椒的负面结算

- 前提与选择：服务端 `modmain.lua`，只改 `pepper` 的 `edible` 数值，不新建注册、不动 `VEGGIES` 表；适合"想让生辣椒不劝退但仍保留代价"的取舍。
- 生效与撤销：`AddPrefabPostInit` 在每次构造（含读档重建）后执行；停用模组后按原表值重新构造。`healthvalue`／`sanityvalue` 是带 setter 的属性，赋值会重跑 `badfood` 标签判定（[edible.lua#L1-L7](../../../raw/current/scripts/components/edible.lua#L1-L7)、[属性表](../../../raw/current/scripts/components/edible.lua#L50-L53)），所以改成 0 也能正确摘掉 `badfood`。
- 原版依据：负值由 `eater` 按取用值结算，本改动只改物品侧数值，不影响熟形态与烹饪判定。

```lua
-- modmain.lua：生辣椒的惩罚减半（生命 -20 -> -10，理智 -15 -> -7.5）
AddPrefabPostInit("pepper", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    local edible = inst.components.edible
    if edible == nil then
        return
    end
    edible.healthvalue = edible.healthvalue / 2
    edible.sanityvalue = edible.sanityvalue / 2
end)
```

- 调整与验证：单位是绝对值（生命／理智点数），不是百分比；除以 2 依赖当前原值为 −20／−15，若与其他模组叠加需改成显式赋值。本改动只影响吃下时的结算，容器与物品栏图标、名称、保鲜期均不变。

## 常见陷阱与限制

- **边界：负收益是实值，不是标签**（以下均为新鲜态、无额外修正的基准）：onion／garlic 生吃只压 10 理智、0 生命，pepper 生吃同时压 20 生命与 15 理智；烤制只减轻不归零（pepper −3／−10，onion 与 garlic −5 理智且 +1 生命），asparagus 是四键里唯一没有负项的。[表项](../../../raw/current/scripts/prefabs/veggies.lua#L132-L154)、[结算](../../../raw/current/scripts/components/eater.lua#L211-L235)
- **边界：熟形态不是生形态的超集**：`<name>_cooked` 没有 `cookable`（不能再烤）、没有 `weighable`、没有 `snowmandecor`，两态的组件集合要分开核对。[熟工厂](../../../raw/current/scripts/prefabs/veggies.lua#L740-L776)
- **误区：onion 的 Gorge 图标不是活动模式分支**：`QUAGMIRE_PORTS` 是硬编码名字表（tomato、onion），任何模式、任何季节都换；换的是 `inventoryitem.imagename`（背包／容器／拖拽图标），世界动画仍是 `onion` 的 bank／build。[名字表](../../../raw/current/scripts/prefabs/veggies.lua#L33-L37)、[替换](../../../raw/current/scripts/prefabs/veggies.lua#L641-L643)
- **边界：光挂 `snowmandecor` 不能装饰雪人**：装饰外观与可放置性来自 `snowmandecoratable` 的本地 `ITEM_DATA` 表（按名字 hash 建表，不对外暴露），而候选还要求目标带 `heavy` 标签且非 `waxedplant`；雪人（带 `heavy`）才符合，可抛雪球不符合。[候选](../../../raw/current/scripts/componentactions.lua#L1745-L1756)、[数据表](../../../raw/current/scripts/components/snowmandecoratable.lua#L44-L49)、[雪人 heavy](../../../raw/current/scripts/prefabs/snowman.lua#L639)
- **边界：装饰件会消耗并从背包扣除**：放置时按雪人尺寸的装饰上限校验，并从背包按名字扣数；雪人被锤到破坏、或中途敲掉已有装饰时，`DropAllDecor` 再按 `ITEM_DATA` 把物品掉回地面。[校验与扣除](../../../raw/current/scripts/components/snowmandecoratable.lua#L244-L296)、[掉回](../../../raw/current/scripts/components/snowmandecoratable.lua#L722-L751)、[破坏与拆装饰分支](../../../raw/current/scripts/prefabs/snowman.lua#L531-L544)、[装饰上限](../../../raw/current/scripts/tuning.lua#L8559)
- **边界：挂 `tradable` 不等于猪王收货**：四键都未设 `goldvalue`，猪王只在 `goldvalue > 0`（或节日糖果、`pig_token`）时接受。[AcceptTest](../../../raw/current/scripts/prefabs/pigking.lua#L598-L606)
- **误区：用 `strings.pot` 判断简中缺译**：模板全表 `msgstr` 为空，据此会得出"所有条目都缺"的错误结论；本页八键在 `chinese_s.po` 均有译文，`pepper_cooked` 是「烤辣椒」。[PEPPER_COOKED 简中](../../../raw/current/scripts/languages/chinese_s.po#L346780-L346782)、[模板](../../../raw/current/scripts/languages/strings.pot#L346776-L346779)

## 验证记录

- 源码审阅：2026-11-08，Steam buildid 26335394。核验 `veggies.lua` 的 `VEGGIES` 四段表项（L132-L154）、`MakeVegStats` 形参与字段、`MakeVeggie` 的生／熟闭包、`QUAGMIRE_PORTS` 与图标替换、堆叠黑名单、`issnowmandecor` 分支、导出与注册循环；`tuning.lua` 的天数、保鲜、热量、治疗、理智、堆叠与季节权重常量；`farm_plants.lua` 的 pickable 装配、压力战利品与随机种子加权；`farm_plant_defs.lua` 的产物／种子命名与适种季节；`pickable:Pick` 的入栏与移除分支；`snowmandecoratable` 的数据表、扣除与掉回；`chinese_s.po` 八个名称键与 `strings.pot` 的空翻译对照。未覆盖：Gorge 模式内 `quagmire_*` 物品的完整体验、雪人装饰的运行时表现、`quagmire_onion` 图标资源外观（资源不在源码快照内）、四季系数在实际存档中的分布。
- 游戏运行：未运行。三维、保鲜、堆叠与权重数值均为 tuning 与表项的源码推导与整除换算（如 `CALORIES_TINY`＝75/8＝9.375），不是运行测量。
- 结构检查：`validate_page.py` 通过（165 行／26,129 字节，standard 档上限 240 行／32,768 字节；元数据、标签与链接无报错）。正文 102 处源码行锚点（另 1 处仅文件级）按 build 26335394 回读首末行，无空行或结构行起点；`sources` 与实际引用文件一一对应。

## 相关链接

- 同一工厂的另外两类：[[reference/prefabs/farm-crops-staple]]（`carrot`／`corn`／`pumpkin`／`potato` 及熟态，含「共用工厂与生熟两态」一节）、[[reference/prefabs/farm-crops-fruit]]（`eggplant`／`durian`／`pomegranate`／`dragonfruit`／`watermelon`／`tomato` 及熟态）
- 巨型形态与打蜡：[[reference/prefabs/giant-crops]]
- 种植与生长：[[reference/components/crop]]、[[reference/components/farmplantable]]、[[reference/components/farming-manager]]

返回 [[reference/prefabs/index|Prefab 索引]]。
- 2026-11-08 build 26474873：本页随版本迁移升版（锚点迁移 1 处），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-10）。
