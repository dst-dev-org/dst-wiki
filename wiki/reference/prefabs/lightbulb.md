---
title: "荧光果（lightbulb）"
created: "2026-11-09"
updated: "2026-11-10"
type: prefab
summary: "洞穴荧光花的产物荧光果：落地才亮的小光源、CAVE 燃料 90 点、可食蔬菜（+1 生命）与 6 天腐坏（地面约 4 天）、堆叠 40，月亮精华液可把它变异成球状光虫，并可作为花瓶装饰。"
aliases: ["lightbulb", "荧光果", "荧光果物品"]
tags: [prefab, item, food]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/lightbulb.lua", "prefabs/flower_cave.lua", "prefabs/halloweenpotion_moon.lua", "prefabs/mininglantern.lua", "prefabs/hats.lua", "prefabs/hutch.lua", "prefabs/spicepack.lua", "prefabs/lantern_posts.lua", "containers.lua", "components/halloweenmoonmutable.lua", "components/halloweenpotionmoon.lua", "components/fuel.lua", "components/fueled.lua", "components/perishable.lua", "components/edible.lua", "components/inventory.lua", "prefabs/lantern_posts_defs.lua", "standardcomponents.lua", "componentactions.lua", "actions.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 荧光果（lightbulb）

## 快速判断

- 职责与适用：`lightbulb` 是可拾取物品（`item`）兼可食蔬菜（`food`），同时是 CAVE 燃料与“落地才亮”的微型光源。它有三个高频用途：给提灯／矿工帽补燃料、吃掉回 1 点生命、插进花瓶当装饰。[工厂](../../../raw/current/scripts/prefabs/lightbulb.lua#L14-L83)
- 优先使用：数值与鲜度查[数值与结算](#数值与结算)；照明开关查[光照开关](#光照开关只在落地亮)；燃料与标签消费者查[燃料与 lightbattery 标签](#燃料与-lightbattery-标签)。
- 重要副作用：腐坏时按 `onperishreplacement` 换成 `spoiled_food`（不像冰那样直接消失）；被用作燃料时从叠里取 1 件删除并给接收方 90 点 CAVE 燃料。[腐坏替换](../../../raw/current/scripts/prefabs/lightbulb.lua#L71-L74)、[燃料结算](../../../raw/current/scripts/components/fueled.lua#L197-L221)
- 修改边界：荧光花本体、六注册与产量归 [[reference/prefabs/flowers]]；提灯与矿工帽的开关、容量与消耗节奏归 [[reference/prefabs/lantern]]；本页只写物品自身与三类消费者（S6）。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `lightbulb` | 单工厂、无变体：实体＋Transform／AnimState／**AddLight**／Network → `MakeInventoryPhysics` → bank／build `bulb`、播 `idle`、`SetBloomEffectHandle` → 光源参数与 `Light:Enable(true)` → 标签 `lightbattery`／`vasedecoration`／`light` → `MakeInventoryFloatable("small", 0.07, 0.73)` → `SetPristine` → 客户端返回 → 主模拟挂组件 | `Light` 是实体侧引擎接口而不是组件；组件全部只在主模拟添加，燃料与腐坏都写在主模拟段。[工厂前半](../../../raw/current/scripts/prefabs/lightbulb.lua#L14-L54)、[主模拟段](../../../raw/current/scripts/prefabs/lightbulb.lua#L56-L80) |

光源参数：falloff 0.7、intensity 0.5、radius 0.5、颜色 `(237, 237, 209)/255`，半径只有 0.5；`Light:Enable(true)` 在构造时先打开，之后由 `inventoryitem` 的两个回调切换。[光源参数](../../../raw/current/scripts/prefabs/lightbulb.lua#L31-L35)

### 名称与获得

简中名 `STRINGS.NAMES.LIGHTBULB`＝“荧光果”（S2）。注意注册名就是 `lightbulb`，不要与 `wormlight`（发光浆果）或 `lightflier`（球状光虫）混写；标题、别名与正文都写注册名 `lightbulb`。[NAMES.LIGHTBULB](../../../raw/current/scripts/languages/chinese_s.po#L344710-L344712)

获得（S4）：洞穴荧光花 `flower_cave` 的采摘产物；单／双／三株各自的产量、再生倍率与花本体机制都归 [[reference/prefabs/flowers|花与荧光花]]，本页不重复。产物接线见 [单株](../../../raw/current/scripts/prefabs/flower_cave.lua#L419)。

## 数值与结算

时间基准：`seg_time`＝30 秒、`total_day_time`＝480 秒，鲜度天数按 480 秒/天折算（S1）。[基准](../../../raw/current/scripts/tuning.lua#L19-L28)

| 项 | 定义处 | 结算处与实际值 |
|---|---|---|
| 食用 | `healthvalue=HEALING_TINY`、`hungervalue=0`、`foodtype=FOODTYPE.VEGGIE`，未设 `sanityvalue` [edible 定义](../../../raw/current/scripts/prefabs/lightbulb.lua#L66-L69) | 生命 +1、饥饿 0、理智 0；`degrades_with_spoilage` 保持默认 true，所以在陈旧／腐坏时收益按 `edible` 的倍率衰减 [HEALING_TINY](../../../raw/current/scripts/tuning.lua#L2021)、[饥饿与倍率](../../../raw/current/scripts/components/edible.lua#L96-L119) |
| 鲜度 | `SetPerishTime(PERISH_FAST)`＋`StartPerishing()`，`onperishreplacement="spoiled_food"` [perishable 定义](../../../raw/current/scripts/prefabs/lightbulb.lua#L71-L74) | 定义值 6 天＝2880 秒；无持有者（丢在地上）再乘 `PERISH_GROUND_MULT`＝1.5，地面实际约 4 天＝1920 秒；普通持有者 6 天，冰箱（`fridge`）12 天、厨师袋（`spicepack`，带 `foodpreserver` 标签）8 天（倍率 `PERISH_FRIDGE_MULT`＝0.5、`PERISH_FOOD_PRESERVER_MULT`＝0.75；冰箱分支因荧光果不带 `frozen` 才取 0.5，`foodpreserver` 分支不看 `frozen`）。腐坏时先回调（本例无回调），再生成 `spoiled_food` 并接管叠数与槽位 [PERISH_FAST](../../../raw/current/scripts/tuning.lua#L2046)、[地面倍率](../../../raw/current/scripts/tuning.lua#L2083)、[冰箱与保鲜器倍率](../../../raw/current/scripts/tuning.lua#L2079-L2080)、[冰箱收件条件](../../../raw/current/scripts/containers.lua#L1019-L1041)、[厨师袋标签](../../../raw/current/scripts/prefabs/spicepack.lua#L87-L89)、[NAMES.SPICEPACK](../../../raw/current/scripts/languages/chinese_s.po#L349775-L349777)、[替换顺序](../../../raw/current/scripts/components/perishable.lua#L277-L320) |
| 燃料 | `fuel.fuelvalue=MED_LARGE_FUEL`、`fuel.fueltype=FUELTYPE.CAVE` [fuel 定义](../../../raw/current/scripts/prefabs/lightbulb.lua#L56-L58) | 90 点 CAVE 燃料；`ADDFUEL` 先从叠里取 1 件，接收方按 `fuelvalue × bonusmult × mult × 湿惩罚 × 熟练倍率` 加燃料后把那一件删掉 [MED_LARGE_FUEL](../../../raw/current/scripts/tuning.lua#L1676)、[取一件](../../../raw/current/scripts/components/inventory.lua#L1397-L1402)、[TakeFuelItem](../../../raw/current/scripts/components/fueled.lua#L197-L221) |
| 堆叠 | `stackable.maxsize=STACK_SIZE_SMALLITEM` [堆叠定义](../../../raw/current/scripts/prefabs/lightbulb.lua#L49-L50) | 本版 40 [STACK_SIZE_SMALLITEM](../../../raw/current/scripts/tuning.lua#L83) |
| 可燃 | `MakeSmallBurnable(inst, TINY_BURNTIME)`＋`MakeSmallPropagator` [可燃与蔓延](../../../raw/current/scripts/prefabs/lightbulb.lua#L60-L62) | 燃烧时间 3 秒（`seg_time*0.1`），可把火传给邻近可燃物 [TINY_BURNTIME](../../../raw/current/scripts/tuning.lua#L1680) |

燃料量的相对大小（推导）：提灯与矿工帽的容量都是 `(night_time+dusk_time)*2.6`＝468 点，一块荧光果 90 点约占 19%。[两个 LIGHTTIME](../../../raw/current/scripts/tuning.lua#L1726-L1727)

## 光照开关（只在落地亮）

开关不在 `Light` 组件里，而在 `inventoryitem` 的两个服务端回调：`SetOnDroppedFn(OnDropped)` 在落地时 `Light:Enable(true)`，`SetOnPutInInventoryFn(OnPutInInventory)` 在进入背包／容器时 `Light:Enable(false)`。所以“荧光果会亮”只对落在地面的那一件成立，拿在手上、放进箱子或背包里都不亮。[两个回调](../../../raw/current/scripts/prefabs/lightbulb.lua#L6-L12)、[接线](../../../raw/current/scripts/prefabs/lightbulb.lua#L76-L77)

两个边界：构造时先 `Light:Enable(true)`，若物品生成后直接进入容器，则由入包回调把它关掉（生成即入容器的各条 GiveItem 路径未逐一核验）；反过来，从容器取出丢到地上时才重新点亮。客户端看到的亮度由引擎侧的 Light 状态负责，本页未核验客户端的预测时序。[构造即点亮](../../../raw/current/scripts/prefabs/lightbulb.lua#L31-L35)、[回调接线](../../../raw/current/scripts/prefabs/lightbulb.lua#L64-L77)

## 燃料与 lightbattery 标签

`fuel` 组件的属性回调在设 `fueltype` 时会连带加一个 `CAVE_fuel` 标签，接收方的 `fueled` 靠 `fueltype` 配对与 `accepting` 收料，所以荧光果既能进提灯也能进矿工帽。[标签属性](../../../raw/current/scripts/components/fuel.lua#L1-L19)、[提灯 fueled](../../../raw/current/scripts/prefabs/mininglantern.lua#L265-L278)、[矿工帽 fueled](../../../raw/current/scripts/prefabs/hats.lua#L772-L778)

`lightbattery` 是另一条线：表示“可作为灯具的电池／灯芯”的标签，被容器与灯具按标签筛选，与 `fuel` 的 CAVE 数值无关。原版消费者有蘑菇灯（`mushroom_light`）与菌伞灯（`mushroom_light2`）的容器过滤、喜庆灯笼柱（`yots_lantern_post`）与珍珠的灯柱（`hermitcrab_lightpost`）的灯柱过滤、哈奇（`hutch`）的取灯逻辑。[蘑菇灯／菌伞灯过滤](../../../raw/current/scripts/containers.lua#L670-L682)、[灯柱过滤](../../../raw/current/scripts/prefabs/lantern_posts.lua#L48)、[两种灯柱](../../../raw/current/scripts/prefabs/lantern_posts_defs.lua#L91-L120)、[哈奇取灯](../../../raw/current/scripts/prefabs/hutch.lua#L134-L136)

`vasedecoration` 是第三条线：物品带该标签且目标是 `vase` 时，才会被收集到 `DECORATEVASE` 动作，执行时调用花瓶的 `vase:Decorate` 把它插进去。[花瓶动作](../../../raw/current/scripts/componentactions.lua#L1916-L1924)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：fuel | `fuelvalue=MED_LARGE_FUEL`、`fueltype=CAVE`，属性回调写 `CAVE_fuel` 标签 | 仅主模拟；被接收方 `TakeFuelItem` 结算 | [[reference/components/fuel]]；[标签属性](../../../raw/current/scripts/components/fuel.lua#L1-L19) |
| 组件：edible | 生命 +1、饥饿 0、食性 VEGGIE | 仅主模拟；可被腐烂倍率衰减 | [[reference/components/edible]]；[接线](../../../raw/current/scripts/prefabs/lightbulb.lua#L66-L69) |
| 组件：perishable | 6 天计时，`onperishreplacement="spoiled_food"` | 仅主模拟；生成替换物 | [[reference/components/perishable]]；[接线](../../../raw/current/scripts/prefabs/lightbulb.lua#L71-L74) |
| 组件：stackable | 上限 40 | 仅主模拟 | [[reference/components/stackable]]；[接线](../../../raw/current/scripts/prefabs/lightbulb.lua#L49-L50) |
| 组件：inventoryitem | 落地／入包两个回调切 Light；无自定义 `imagename` | 回调只在主模拟设置 | [[reference/components/inventoryitem]]；[回调](../../../raw/current/scripts/prefabs/lightbulb.lua#L76-L77) |
| 组件：halloweenmoonmutable | `SetPrefabMutated("lightflier")`，组件自带 `halloweenmoonmutable` 标签 | 转换由月亮精华液的 `Use` 触发，不是满月自动发生 | [[reference/components/halloweenmoonmutable]]；[接线](../../../raw/current/scripts/prefabs/lightbulb.lua#L79-L80) |
| 组件：vasedecoration | 空标记组件＋同名标签，供 `DECORATEVASE` 收集 | 目标需带 `vase` | [[reference/components/vasedecoration]]；[挂载](../../../raw/current/scripts/prefabs/lightbulb.lua#L52-L53) |
| 组件：fueled（接收方） | 提灯与矿工帽按 CAVE 配对收料 | 见提灯页 | [[reference/prefabs/lantern]]；[提灯](../../../raw/current/scripts/prefabs/mininglantern.lua#L265-L278) |
| 组件：hauntable | `MakeHauntableLaunchAndIgnite`：作祟只把物品弹飞 | 本版“作祟点燃”分支整段被注释，即使带 `burnable` 也不会被幽灵点着 | [空分支](../../../raw/current/scripts/standardcomponents.lua#L1191-L1217) |
| 关联 Prefab：`flower_cave` | 主获得来源，产量与再生见 [[reference/prefabs/flowers]] | 主模拟采摘 | [[reference/prefabs/flowers]]；[单株](../../../raw/current/scripts/prefabs/flower_cave.lua#L419) |
| 其他来源：`lightcrab`／`lightflier`／`stalker_bulb` | 发光蟹 25% 随机掉落；球状光虫的共享掉落表固定给 1 个；两个 `stalker_bulb` 注册的采摘产物也是荧光果（再生设为 1000000 并 `Pause`） | 前两者为战斗／击杀路径，后者为采摘路径；`stalker_bulb` 无 NAMES 键 | [发光蟹](../../../raw/current/scripts/prefabs/lightcrab.lua#L106)、[光虫掉落表](../../../raw/current/scripts/prefabs/lightflier.lua#L13-L16)、[灯泡采摘](../../../raw/current/scripts/prefabs/stalker_bulb.lua#L164-L177) |
| 关联 Prefab：`lightflier` | 变异产物“球状光虫”；`Mutate` 先在原实体位置 `SpawnPrefab`，原实体若在容器里再把新实体 `GiveItem` 放回同一容器 | 需要月亮精华液 | [原地生成](../../../raw/current/scripts/components/halloweenmoonmutable.lua#L48-L50)、[放回同一容器](../../../raw/current/scripts/components/halloweenmoonmutable.lua#L62-L72) |
| 关联 Prefab：`halloweenpotion_moon` | 月亮精华液，`Use` 时找到目标的 `halloweenmoonmutable` 并 `Mutate` | 消耗一瓶、成功与否都移除 | [Use](../../../raw/current/scripts/components/halloweenpotionmoon.lua#L11-L30)、[NAMES.LIGHTFLIER](../../../raw/current/scripts/languages/chinese_s.po#L344725-L344727) |
| 关联 Prefab：`spoiled_food` | 腐坏替换物 | 主模拟生成 | [[reference/prefabs/spoiled-food]]；[替换名](../../../raw/current/scripts/prefabs/lightbulb.lua#L71-L74) |
| 资源：`anim/bulb.zip` | 物品动画包，`idle` 循环；`SetBloomEffectHandle` 叠发光 | 原版资源，不随页分发 | [资产](../../../raw/current/scripts/prefabs/lightbulb.lua#L1-L4) |

## 典型接入

### 让一块荧光果给灯具补两倍燃料

- 前提与选择：服务端 `modmain.lua` 的 postinit。改 `fuel.fuelvalue` 只影响“当燃料”这条链，不动食用、腐坏与光照；不要改 `TUNING.MED_LARGE_FUEL`，那会连带所有中大型燃料。[原值](../../../raw/current/scripts/prefabs/lightbulb.lua#L56-L58)
- 生效与撤销：接收方在 `TakeFuelItem` 时才读 `fuelvalue`，因此只影响由本 postinit 构造出来的实例（含读档重建的实例），同一实例在背包里被使用时按新值结算；停用后新生成的荧光果回到 90。数值单位是燃料点，不是秒。[读取与结算](../../../raw/current/scripts/components/fueled.lua#L205-L212)
- 原版依据：提灯与矿工帽都用 `fueltype=CAVE`＋`accepting=true` 的 `fueled`，按 `fuelvalue × bonusmult` 加料；提灯的 `bonusmult` 未改，仍是 1。[提灯配置](../../../raw/current/scripts/prefabs/mininglantern.lua#L265-L278)

```lua
-- modmain.lua：荧光果的燃料值从 90 改成 180
AddPrefabPostInit("lightbulb", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    local fuel = inst.components.fuel
    if fuel ~= nil then
        fuel.fuelvalue = 2 * GLOBAL.TUNING.MED_LARGE_FUEL
    end
end)
```

- 调整与验证：要用干燥、无额外倍率（`bonusmult`、湿惩罚、熟练倍率均为 1）的荧光果，并且提灯空余容量至少 180 点；`DoDelta` 会把结果夹到 `[0, maxfuel]`，空余不足时 `currentfuel` 的增量会小于 180（上限封顶），不能拿增量反推燃料值。反过来说，验证“按叠取 1 件”应在同一提灯上连续喂同一叠，观察每次只减 1 件。[容量夹取](../../../raw/current/scripts/components/fueled.lua#L305-L323)

## 常见陷阱与限制

- **误区：荧光果拿在手上也会亮。** 只有 `SetOnDroppedFn` 触发后才 `Light:Enable(true)`，入包回调立刻关灯；拿在手上时不提供照明。[两个回调](../../../raw/current/scripts/prefabs/lightbulb.lua#L6-L12)
- **边界：`lightbattery` 不等于 `fuel`。** 前者是容器／灯具按标签筛选的灯芯类标记，后者才是 CAVE 燃料数值；两个标签都命中不代表任何接收方都会接受它。[CAVE_fuel 标签](../../../raw/current/scripts/components/fuel.lua#L1-L8)、[按标签过滤](../../../raw/current/scripts/containers.lua#L670-L682)
- **边界：变异不是满月自动触发。** `halloweenmoonmutable` 只登记变异目标，真正调用 `Mutate` 的是月亮精华液 `Use`；没有药水时荧光果不会自己变成球状光虫。[只提供 Mutate](../../../raw/current/scripts/components/halloweenmoonmutable.lua#L28-L79)、[外部触发](../../../raw/current/scripts/components/halloweenpotionmoon.lua#L11-L30)
- **边界：腐坏会换物品。** `onperishreplacement="spoiled_food"` 会在腐坏时生成腐烂物并接管叠数与槽位，与冰“直接消失”不同。[替换与槽位](../../../raw/current/scripts/components/perishable.lua#L293-L319)
- **边界：可燃且会传火。** `MakeSmallBurnable`＋`MakeSmallPropagator` 让荧光果能被点燃并把火传给邻居，堆在火边会成片烧掉。[可燃](../../../raw/current/scripts/prefabs/lightbulb.lua#L60-L62)
- **未核验**：客户端落地亮灯的同步时序、球状光虫变异后的游戏内行为（S7）。

## 验证记录

- 源码审阅：2026-11-09，steam-buildid:26474873（`raw/current/BUILD.txt` 实读）。`lightbulb.lua` 全文核对光源参数、标签、组件与两个回调；`flower_cave.lua` 三档 `pickable:SetUp`；`fuel`／`fueled` 的标签与结算；`edible` 与 `perishable` 的取值与替换；`containers.lua`、`lantern_posts.lua`、`hutch.lua` 的 `lightbattery` 过滤；`halloweenmoonmutable` 与 `halloweenpotionmoon` 的变异链；`componentactions` 的花瓶动作；`tuning` 的 `MED_LARGE_FUEL`／`PERISH_FAST`／`STACK_SIZE_SMALLITEM`／`TINY_BURNTIME`／`HEALING_TINY`／两个 `LIGHTTIME`；两个简中名键。
- 游戏运行：未运行；光照开关、燃料点与变异均为源码审阅（S7）。未覆盖：多人下地面光源的可见范围、变异后容器落点。
- 检视修订（2026-11-09，同一 build）：①摘要与鲜度行补“地面 ×1.5、约 4 天／1920 秒”（`perishable.lua` L96-L97、L133）；②`wormlight` 正名为发光浆果（`chinese_s.po` L352440-L352442），球状光虫仍指 `lightflier`；③荧光花产量压成一句并指向 [[reference/prefabs/flowers]]；④燃料改述为“从叠里取 1 件”（`actions.lua` L2188-L2194、`inventory.lua` L1397-L1402、`fueled.lua` L220）；⑤`lightbattery` 消费者改用正式名——蘑菇灯／菌伞灯、喜庆灯笼柱／珍珠的灯柱、哈奇（`containers.lua` L670-L682、`lantern_posts.lua` L48、`lantern_posts_defs.lua` L91-L120、`hutch.lua` L134-L136）；⑥变异流程改为“原地生成、容器内再放回同一容器”（`halloweenmoonmutable.lua` L44-L78）；⑦补 hauntable 行（作祟只弹飞，点燃分支已注释）；⑧“腐烂食物”“实战”改为规范措辞，`sources` 去掉无对应断言的 stackable／inventoryitem 并补实际引用文件。
- 复验修订（2026-11-10）：①导航行删去“按株数 1／2／3”，产量与再生统一指向 flowers；②鲜度行补持有者差异——普通持有者 6 天、冰箱 12 天、食物保鲜器 8 天（`PERISH_FRIDGE_MULT` 0.5／`PERISH_FOOD_PRESERVER_MULT` 0.75；冰不带 `frozen` 才走这两档），并补冰箱收件条件锚点；③复验追问后把“食物保鲜器”改为实际载体“厨师袋（`spicepack`，`foodpreserver` 标签）”，并区分两支：冰箱分支才看 `frozen`，`foodpreserver` 分支不看（`spicepack.lua` L87-L89、`perishable.lua` L81-L88）。
- 外部检视修订（2026-11-10，W05 报告 R08）：①“背包里立即生效”改为“只影响本 postinit 构造（含读档重建）的实例”；②示例验收补剩余容量门槛——干燥、无额外倍率且提灯空余 ≥180 点，`currentfuel` 才必增 180，因为 `DoDelta` 会夹到 `[0, maxfuel]`（`fueled.lua` L305-L323）。
- 结构检查：`validate_page.py` 通过；`validate_source_links.py --strict` 0 error／0 warning。本页 Lua 示例为完整片段，未在游戏内执行。

## 相关链接

- 荧光花与产量：[[reference/prefabs/flowers]]
- 提灯／矿工帽的燃料消费：[[reference/prefabs/lantern]]
- 变异链：[[reference/components/halloweenmoonmutable]]
- 燃料接口：[[reference/components/fuel]]、[[reference/components/fueled]]
- 花瓶装饰：[[reference/components/vasedecoration]]

返回 [[reference/prefabs/index|Prefab 索引]]。
