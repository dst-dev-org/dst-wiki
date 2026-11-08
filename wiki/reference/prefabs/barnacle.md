---
title: "藤壶与熟藤壶（barnacle、barnacle_cooked）"
created: "2026-11-09"
updated: "2026-11-09"
type: prefab
summary: "meats.lua 的 barnacle 与 barnacle_cooked：显式写成 MEAT 食性的海产小食，selfstacker 自动合堆与 quickeat 快速进食两条标签行为、金值 0 的交易后果，以及海草与帝王蟹等获得路径。"
aliases: ["barnacle", "barnacle_cooked", "藤壶", "熟藤壶", "藤壶肉"]
tags: [prefab, item, food]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/meats.lua", "components/edible.lua", "components/perishable.lua", "components/stackable.lua", "components/cookable.lua", "components/floater.lua", "components/bait.lua", "components/eater.lua", "components/tradable.lua", "components/selfstacker.lua", "containers.lua", "standardcomponents.lua", "actions.lua", "stategraphs/SGwilson.lua", "stategraphs/SGwaterplant.lua", "prefabs/waterplant.lua", "prefabs/otterden.lua", "prefabs/shark.lua", "prefabs/crabking.lua", "prefabs/crabking_cannontower.lua", "prefabs/pigking.lua", "prefabs/antlion.lua", "cooking.lua", "recipes.lua", "tuning.lua", "languages/chinese_s.po", "screens/redux/scrapbookdata.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 藤壶与熟藤壶（barnacle、barnacle_cooked）

## 快速判断

- 职责与适用：两个注册都由 `meats.lua` 的 `common()` 生成；生藤壶额外带 `selfstacker` 与 `quickeat` 两个行为标签，可烤成 `barnacle_cooked`，两件都不上晾肉架（`dryable` 传 `nil`）。[生藤壶](../../../raw/current/scripts/prefabs/meats.lua#L690-L712)、[熟藤壶](../../../raw/current/scripts/prefabs/meats.lua#L715-L731)
- 优先使用：查自动合堆看[selfstacker](#selfstacker-自动合堆)；查为什么吃得快看[quickeat](#quickeat-快速进食)；查金值与交易后果看[数值与获得](#数值与获得)。
- 重要副作用：`tradable.goldvalue` 被显式改成 0（`common()` 默认是 `GOLD_VALUES.MEAT`＝1），普通换金需 `goldvalue > 0`（万圣节糖果与 `pig_token` 另路，鱼人交付者先被拒），所以它走不了普通换金。[覆盖](../../../raw/current/scripts/prefabs/meats.lua#L705-L705)、[收件判定](../../../raw/current/scripts/prefabs/pigking.lua#L598-L606)
- 修改边界：只讲这两个注册；海草（`waterplant`）本体、岩石大白鲨／帝王蟹的战斗与掉落机制只记获得关系；`selfstacker` 组件的通用接口归组件页。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `barnacle` | `barnacle()` 调 `common("barnacle", "barnacle", "raw", {"barnacle", "rawmeat", "selfstacker", "quickeat"}, nil, {product="barnacle_cooked"})`：`dryable` 为 `nil`，`cookable` 指向熟态 | 标签：`meat`＋`barnacle`／`rawmeat`／`selfstacker`／`quickeat`＋`cookable`；`dryable` 与 `lureplant_bait` 都不存在。主模拟再显式重写一次 `edible.ismeat = true` 与 `edible.foodtype = FOODTYPE.MEAT`（`common()` 已写过，属重复赋值），然后写三维、金值 0、漂浮 0.05，并 `AddComponent("selfstacker")`。[构造](../../../raw/current/scripts/prefabs/meats.lua#L690-L712) |
| `barnacle_cooked` | `barnacle_cooked()` 调 `common("barnacle", "barnacle", "cooked", {"barnacle", "quickeat"})`：`dryable`／`cookable` 均缺省 | 标签只剩 `meat`＋`barnacle`／`quickeat`；没有 `selfstacker` 组件，因此不能自动合堆；`sanityvalue` 显式写 0；金值同样 0。[熟态](../../../raw/current/scripts/prefabs/meats.lua#L715-L731) |

`common()` 的共用部分：写 bank／build／anim、加 `meat` 标签、`MakeInventoryFloatable(inst)`（默认 `small`、无偏移）、`SetPristine`、非主模拟返回；主模拟统一挂 `edible`（`ismeat=true`、`FOODTYPE.MEAT`）、`bait`、`inspectable`、`inventoryitem`、`stackable`（默认 20）、`tradable`（先写金值 1）、`perishable`（腐坏→`spoiled_food`）。[工厂](../../../raw/current/scripts/prefabs/meats.lua#L138-L222)、[漂浮默认](../../../raw/current/scripts/standardcomponents.lua#L1772-L1791)、[bait 组件](../../../raw/current/scripts/components/bait.lua#L21-L29)

### 名称核验

`barnacle` 的简中实显是**藤壶**（Barnacles，源串为复数），`barnacle_cooked` 是**熟藤壶**（Cooked Barnacles）；两个键各命中一次，注册名与 `STRINGS.NAMES` 键同名。[BARNACLE](../../../raw/current/scripts/languages/chinese_s.po#L339044-L339047)、[BARNACLE_COOKED](../../../raw/current/scripts/languages/chinese_s.po#L339065-L339067)

## 数值与获得

时间基准：`seg_time`＝30 秒、`total_day_time`＝480 秒，下面天数按 480 秒/天换算。[基准](../../../raw/current/scripts/tuning.lua#L19-L20)

| 注册 | 饥饿／生命／理智 | 鲜度 | 堆叠 | 漂浮 | 转换 |
|---|---|---|---|---|---|
| `barnacle` | 12.5／0／−5 | `PERISH_FAST`＝6 天＝2880 秒 | 20 | 垂直偏移 0.05（尺寸沿用默认 `small`） | 烤制 → `barnacle_cooked`；无干制；金值 0 |
| `barnacle_cooked` | 12.5／1／0 | `PERISH_SLOW`＝15 天＝7200 秒 | 20 | 垂直偏移 0.05 | 无；金值 0 |

数值来源：[生藤壶](../../../raw/current/scripts/prefabs/meats.lua#L698-L707)、[熟藤壶](../../../raw/current/scripts/prefabs/meats.lua#L722-L728)、[常量](../../../raw/current/scripts/tuning.lua#L2021-L2023)、[热量](../../../raw/current/scripts/tuning.lua#L2058-L2060)、[理智](../../../raw/current/scripts/tuning.lua#L2033-L2034)、[鲜度](../../../raw/current/scripts/tuning.lua#L2043-L2051)、[堆叠](../../../raw/current/scripts/tuning.lua#L82-L82)。采集册同值：`hungervalue=12.5`、`healthvalue=0/1`、`sanityvalue=-5/0`、`perishable=2880/7200`。[采集册](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L102-L103)

金值的落点：`common()` 统一写 `GOLD_VALUES.MEAT`，两个工厂随后覆盖成 0。生熟藤壶金值均为 0，不能走普通换金分支——`AcceptTest` 只放行 `goldvalue > 0`、活动糖果或 `pig_token`，且鱼人（`merm`）交付者会先被拒（`pig_token` 例外）。[默认金值](../../../raw/current/scripts/prefabs/meats.lua#L193-L194)、[覆盖](../../../raw/current/scripts/prefabs/meats.lua#L705-L705)、[`GOLD_VALUES`](../../../raw/current/scripts/tuning.lua#L1946-L1952)、[猪王换金](../../../raw/current/scripts/prefabs/pigking.lua#L74-L74)、[收件判定](../../../raw/current/scripts/prefabs/pigking.lua#L598-L606)、[蚁狮](../../../raw/current/scripts/prefabs/antlion.lua#L138-L138)

### selfstacker 自动合堆

生藤壶在 pristine 标签里就写了 `selfstacker`（注释说明是为优化，组件自身也会补同一标签），主模拟再挂组件。行为要点：

- 触发时机：只在 `OnEntityWake` 时检查一次——若 `CanSelfStack()` 为真，随机 0–0.1 秒后执行一次 `DoStack()`；组件不注册周期任务，实体一直醒着就不会反复尝试。[唤醒](../../../raw/current/scripts/components/selfstacker.lua#L18-L26)、[`OnEntityWake`](../../../raw/current/scripts/components/selfstacker.lua#L64-L73)
- 可合条件：`bait` 无陷阱占用、未燃烧、`stackable` 未满、未被手持、速度平方小于 1（即基本静止）、且当前没有配对目标。
- 找目标：半径 20 内、必须带 `selfstacker` 标签、排除 `outofreach`，再按 `stackable:CanStackWith` 判定同种可堆。
- 合堆结果：把对方能放进空位的数量取出合并（取 `RoomLeft()` 与对方堆叠数较小者）。[组件](../../../raw/current/scripts/components/selfstacker.lua#L1-L75)、[pristine 标签](../../../raw/current/scripts/prefabs/meats.lua#L691-L692)

### quickeat 快速进食

`quickeat` 是给**吃者**看的标签，不是给物品加的状态：

- 吃者图选择：顺序是 `quickeat` → `sloweat`（走 `eat`）→ 肉食性（`MEAT` 且非 `fooddrink`，走 `eat`）→ 兜底 `quickeat`。所以藤壶（`foodtype = MEAT`）本会走长的 `eat`，仅因带 `quickeat` 才被提到第一位走短状态；同时带 `quickeat` 与 `sloweat` 时仍走 `quickeat`。[选择](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1251-L1255)、[喂食端同规则](../../../raw/current/scripts/actions.lua#L2436-L2440)
- 时长差异：`quickeat` 在 12 帧（0.4 秒）结算 `Eater:Eat`，`eat` 在 28 帧（约 0.93 秒）才结算；解除 `busy` 分别是 12 帧与 30 帧。[quickeat](../../../raw/current/scripts/stategraphs/SGwilson.lua#L6576-L6598)、[`eat`](../../../raw/current/scripts/stategraphs/SGwilson.lua#L6466-L6498)
- 熟态仍带 `quickeat`，所以烤过之后依然吃得快。[熟态标签](../../../raw/current/scripts/prefabs/meats.lua#L716-L716)

### 获得路径

| 注册 | 已核验来源 | 产出与证据 |
|---|---|---|
| `barnacle` | ①海草（`waterplant`）的可收获产物：`harvestable:SetUp("barnacle", MAX_BARNACLES)`，`shaveable` 也用同一产物，上限 3；②水獭掠夺者窝点（`otterden`）掉落表 0.50／0.25；③岩石大白鲨（`shark`）1.00／0.50／0.25；④帝王蟹（`crabking`）1.00×2＋0.25×2；⑤帝王蟹的炮塔（`crabking_cannontower`）0.75（补充来源，不穷举） | ①[产物](../../../raw/current/scripts/prefabs/waterplant.lua#L363-L363)、[登记](../../../raw/current/scripts/prefabs/waterplant.lua#L443-L447)、[上限](../../../raw/current/scripts/tuning.lua#L5534-L5534)、②[otterden](../../../raw/current/scripts/prefabs/otterden.lua#L32-L41)、③[shark](../../../raw/current/scripts/prefabs/shark.lua#L27-L37)、④[crabking](../../../raw/current/scripts/prefabs/crabking.lua#L80-L88)、⑤[炮塔](../../../raw/current/scripts/prefabs/crabking_cannontower.lua#L29-L38) |
| `barnacle_cooked` | ①生藤壶烤制；②海草被烧毁时，按剩余产物数量逐个生成熟藤壶；③海草在燃烧状态下走死亡状态图的 `animover` 分支时，产物同样按 `barnacle_cooked` 生成（与②是两个入口） | ①[接线](../../../raw/current/scripts/prefabs/meats.lua#L692-L692)、②[烧毁掉落](../../../raw/current/scripts/prefabs/waterplant.lua#L248-L259)、③[死亡状态](../../../raw/current/scripts/stategraphs/SGwaterplant.lua#L122-L136) |

用途补充：烹饪锅把两者记为 `meat = 0.25`、`fish = 0.25`（半份都不到，换菜时要留意）。隐士蟹建造材料**只认生藤壶**——`chum` 3 份、`hermitcrab_teashop` 6 份、`hermitcrab_lightpost` 2 份、`meatrack_hermit_multi` 4 份、温泉 10 份，配方写作 `Ingredient("barnacle", …)`，熟藤壶不能替代；熟藤壶只在烹饪锅食材值上与生品相同。[食材值](../../../raw/current/scripts/cooking.lua#L155-L155)、[chum](../../../raw/current/scripts/recipes.lua#L665-L665)、[茶店](../../../raw/current/scripts/recipes.lua#L972-L972)、[灯柱](../../../raw/current/scripts/recipes.lua#L974-L974)、[肉架](../../../raw/current/scripts/recipes.lua#L976-L976)、[温泉](../../../raw/current/scripts/recipes.lua#L1528-L1528)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：edible | 写 12.5／0／−5（熟态 12.5／1／0）与 `FOODTYPE.MEAT`；生藤壶重复写一次 `ismeat`／`foodtype` | 仅主模拟；理智为负，赋值即加 `badfood`，挡住「喂给其他玩家」；放行条件是 PVP 开启或目标带 `possessedbody`，`strongstomach` 分支只对带 `monstermeat` 的食物生效，对藤壶无效 | [[reference/components/edible#关键接口]]；[生](../../../raw/current/scripts/prefabs/meats.lua#L698-L702)、[熟](../../../raw/current/scripts/prefabs/meats.lua#L723-L725)、[`badfood` 判定](../../../raw/current/scripts/components/edible.lua#L1-L7)、[喂食门槛](../../../raw/current/scripts/actions.lua#L2406-L2410) |
| 组件：selfstacker | 仅生藤壶挂；唤醒时自动与附近同种合并 | 组件仅主模拟：构造器在服务端补 `selfstacker` 标签，`OnRemoveEntity` 撤标签并取消任务；配对搜索也只跑在服务端 | [[reference/components/selfstacker#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/meats.lua#L709-L709)、[标签与移除](../../../raw/current/scripts/components/selfstacker.lua#L3-L34)、[配对](../../../raw/current/scripts/components/selfstacker.lua#L36-L50) |
| 组件：cookable | `product=barnacle_cooked`；`Cook()` 生成产物并把鲜度折半补回；同一标签还决定能否进盐盒，故生藤壶可、熟藤壶不可 | 仅生藤壶；熟态无 `cookable` 标签也无可烤入口 | [[reference/components/cookable#关键接口]]；[接线](../../../raw/current/scripts/prefabs/meats.lua#L692-L692)、[折算](../../../raw/current/scripts/components/cookable.lua#L28-L36)、[盐盒](../../../raw/current/scripts/containers.lua#L1049-L1055) |
| 组件：tradable | 金值显式 0，覆盖 `common()` 的 1 | 仅主模拟，无 replica | [[reference/components/tradable#关键接口]]；[默认](../../../raw/current/scripts/components/tradable.lua#L1-L7)、[覆盖](../../../raw/current/scripts/prefabs/meats.lua#L705-L705) |
| 组件：perishable | 2880／7200 秒，腐坏替换 `spoiled_food` | 仅主模拟；按 0.5／0.2 百分比切标签 | [[reference/components/perishable#关键接口]]；[生](../../../raw/current/scripts/prefabs/meats.lua#L703-L703)、[熟](../../../raw/current/scripts/prefabs/meats.lua#L726-L726) |
| 组件：dryable | **未挂载**：没有组件也没有 `dryable` 标签，肉架拒收 | 两端一致，都因缺标签上不了架；主模拟的两条入槽例外不改变结论（缺组件不会开始计时） | [[reference/components/dryable#关键接口]]；[接收条件](../../../raw/current/scripts/containers.lua#L234-L244)、[例外](../../../raw/current/scripts/containers.lua#L236-L243)、[计时条件](../../../raw/current/scripts/components/dryingrack.lua#L236-L270) |
| 组件：floater | 两件都把垂直偏移设为 0.05，尺寸沿用默认 `small` | 写入在主模拟分支内，服务端实体生效 | [[reference/components/floater#关键接口]]；[生](../../../raw/current/scripts/prefabs/meats.lua#L707-L707)、[熟](../../../raw/current/scripts/prefabs/meats.lua#L728-L728) |
| 标签：`quickeat` | 让吃者走短进食状态；熟态保留 | 吃者侧图判定，物品只需带标签 | [状态选择](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1251-L1255) |
| 标签：`barnacle`／`rawmeat` | `barnacle` 在快照内无 `HasTag` 消费者；`rawmeat` 让 `eatsrawmeat` 吃者跳过负理智结算 | 熟态只剩 `barnacle`＋`quickeat` | [吃者](../../../raw/current/scripts/components/eater.lua#L166-L170)、[结算](../../../raw/current/scripts/components/eater.lua#L211-L242) |
| 关联 Prefab：`waterplant` | 主要来源；可收获／可刮取都用 `barnacle`，烧毁改掉熟藤壶 | 主模拟结算 | [登记](../../../raw/current/scripts/prefabs/waterplant.lua#L443-L447) |
| 资源：`anim/barnacle.zip` | 生／熟共用 bank／build，只切 `raw`／`cooked` 动画 | 原版资源，不随页分发 | [assets](../../../raw/current/scripts/prefabs/meats.lua#L14-L14) |

## 典型接入

### 让熟藤壶也能自动合堆

- 前提与选择：服务端 `modmain.lua` 的 postinit。`selfstacker` 的配对搜索与合并都在服务端完成，客户端不参与合并，无需该标签或组件；组件自己会在服务端补标签，这里不必再手动加。
- 生效与撤销：只影响新生成的熟藤壶；卸下模组后不再自动合并，已合并的堆叠保持不变。
- 原版依据：生藤壶就是这么做的——pristine 标签写 `selfstacker`、主模拟 `AddComponent("selfstacker")`；组件在 `OnEntityWake` 里按 0–0.1 秒延迟尝试合并。[生藤壶](../../../raw/current/scripts/prefabs/meats.lua#L691-L709)、[组件](../../../raw/current/scripts/components/selfstacker.lua#L64-L73)

```lua
-- modmain.lua：给熟藤壶补上自动合堆
AddPrefabPostInit("barnacle_cooked", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    if inst.components.selfstacker == nil then
        inst:AddComponent("selfstacker")
    end
end)
```

- 调整与验证：默认搜索半径 20、堆叠上限沿用 `stackable` 的 20。合并只在唤醒时触发，所以实测要让实体先休眠再唤醒（例如走远让它休眠、再走回唤醒），只把两堆丢近并等静止不会主动跑一次；远处带 `outofreach` 标签的目标不会被选中。

## 常见陷阱与限制

- **误区：藤壶能像鸟腿一样晒成肉干。** 两件都没有 `dryable` 组件与标签，肉架提交会被拒（主模拟的两条入槽例外见 [`itemtestfn`](../../../raw/current/scripts/containers.lua#L236-L243)，但不带组件也不会开始计时）。[构造](../../../raw/current/scripts/prefabs/meats.lua#L692-L692)、[接收条件](../../../raw/current/scripts/containers.lua#L234-L244)
- **边界：金值 0 是显式覆盖。** `common()` 先写 `GOLD_VALUES.MEAT`＝1，两个工厂随后覆盖成 0。生熟藤壶金值均为 0，不能走普通换金分支（`AcceptTest` 只放行 `goldvalue > 0`、活动糖果或 `pig_token`，并先拒鱼人交付者，`pig_token` 例外）。[默认](../../../raw/current/scripts/prefabs/meats.lua#L193-L194)、[覆盖](../../../raw/current/scripts/prefabs/meats.lua#L705-L705)、[收件判定](../../../raw/current/scripts/prefabs/pigking.lua#L598-L606)
- **边界：只有生藤壶会自动合堆。** `barnacle_cooked` 没有 `selfstacker` 组件，也没有该标签，熟藤壶只能手动堆。[熟态](../../../raw/current/scripts/prefabs/meats.lua#L715-L731)
- **边界：`quickeat` 是吃者侧行为，不是本地动画。** 物品只提供标签，实际状态切换发生在吃者状态图；且 `quickeat` 排在 `sloweat` 之前，同时带两个标签仍走短状态，只给带 `quickeat` 的物品补 `sloweat` 不会拉长进食——要拉长必须先去掉 `quickeat`。[选择](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1251-L1255)
- **未核验**：本页未运行游戏；`selfstacker` 的实际合并时机（唤醒后 0–0.1 秒）与「静止」判定（速度平方小于 1）只按源码推导，未实测。

## 验证记录

- 源码审阅：2026-11-09，Steam buildid 26474873。核对 `meats.lua` 的 `common()` 与 `barnacle`／`barnacle_cooked` 两个工厂及返回表；`edible`／`perishable`／`stackable`／`cookable`／`floater`／`tradable`／`selfstacker`／`bait`／`eater` 的相关分支；`SGwilson` 的 `eat`／`quickeat` 状态与动作处理器、`actions.lua` 的喂食分支；`containers.lua` 的肉架接收条件；`waterplant` 的产物与烧毁掉落、`otterden`／`shark`／`crabking` 掉落表、`pigking` 与 `antlion` 的金值判定；`cooking.lua` 与 `recipes.lua` 的用途；`tuning` 的鲜度／热量／理智／堆叠／水草上限常量；`scrapbookdata` 交叉核对。
  复审修订（同日）：补核 `badfood` 喂食放行的实际分支、`selfstacker` 的端边界、肉架入槽例外与计时条件、盐盒门槛，以及 `crabking_cannontower`／`SGwaterplant` 两条补充来源。
  复检修订（同日，第二轮）：建造材料改为只认生藤壶，补 `selfstacker` 仅在唤醒时触发、`quickeat` 的状态选择顺序与 `sloweat` 不生效条件。
  复检修订（同日，第四轮）：猪王句收窄为「普通换金需 `goldvalue > 0`，活动糖果与 `pig_token` 另路，鱼人交付者先被拒」。
  名称核验：`languages/chinese_s.po` 的 `NAMES.BARNACLE`＝藤壶、`NAMES.BARNACLE_COOKED`＝熟藤壶；两个注册名与 `NAMES` 键同名。[po 锚点](#名称核验)
  未覆盖：海草本体的战斗与再生、隐士蟹商店的建造流程；模组注入与动态选名。
- 游戏运行：未运行。天数与三维均为 tuning 推导（`total_day_time`＝480 秒）。
- 结构检查：`validate_page.py` 通过（standard 档内）；`validate_source_links.py --strict` 0 错 0 警。

## 相关链接

- 可食与鲜度：[[reference/components/edible]]、[[reference/components/perishable]]
- 行为组件：[[reference/components/selfstacker]]、[[reference/components/floater]]
- 交易与转换：[[reference/components/tradable]]、[[reference/components/cookable]]

返回 [[reference/prefabs/index|Prefab 索引]]。
