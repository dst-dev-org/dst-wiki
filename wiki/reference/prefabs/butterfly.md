---
title: "蝴蝶（butterfly）"
created: "2026-11-09"
updated: "2026-11-09"
type: prefab
summary: "蝴蝶实体的构造与全部交互入口：捕虫网活捉（1 次、叠 20）、放生种花、击杀掉落（蝶翅权重 5 与黄油 0.1 共抽 1 件）、随身 3 天饥饿、白天采粉与夜里回巢消失，以及烧死只出灰烬、月亮药水变月蛾的边界。"
aliases: ["butterfly", "butterfly_placer", "蝴蝶", "捉蝴蝶", "放生蝴蝶"]
tags: [prefab, creature]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/butterfly.lua", "prefabs/flower.lua", "prefabs/moonbutterfly.lua", "prefabs/bugnet.lua", "prefabs/halloweenpotion_moon.lua", "stategraphs/SGbutterfly.lua", "brains/butterflybrain.lua", "components/pollinator.lua", "components/lootdropper.lua", "components/workable.lua", "components/deployable.lua", "components/container.lua", "components/map.lua", "components/halloweenmoonmutable.lua", "components/halloweenpotionmoon.lua", "standardcomponents.lua", "actions.lua", "componentactions.lua", "constants.lua", "prefabs/hermitcrab.lua", "prefablist.lua", "tuning.lua", "languages/chinese_s.po", "screens/redux/scrapbookdata.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 蝴蝶（butterfly）

## 快速判断

- 职责与适用：`butterfly` 是 1 点生命的飞行小生物（简中实显 **蝴蝶**），同时带 `inventoryitem`／`stackable`／`deployable`：能用捕虫网活捉装进背包，能放生种花，能被击杀取翅。它只在一个文件里构造，另带一个只做放置预览的 `butterfly_placer`。[构造](../../../raw/current/scripts/prefabs/butterfly.lua#L74-L198)、[导出](../../../raw/current/scripts/prefabs/butterfly.lua#L200-L201)
- 优先使用：抓与放看[捕获、放生与击杀](#捕获放生与击杀)；掉落概率看[掉落表](#掉落表)；为什么抓进背包会"饿死"、为什么夜里蝴蝶不见了看[随身计时与消失](#随身计时与消失)。
- 重要副作用：装进容器后开始 3 天倒计时，到点被"饿死"并**尝试**把战利品放进该容器（放不下就掉在原地）；放生消耗 1 只并生成 `planted_flower`（可摘花瓣，但不登记到再生管理器）；烧死的蝴蝶只掉灰烬。
- 修改边界：月蛾（`moonbutterfly`）本体见 [[reference/prefabs/moonbutterfly]]，本页只写"蝴蝶怎么变成月蛾"的入口；蝴蝶刷新归 [[reference/components/butterflyspawner]] 组件页；蝴蝶翅膀与黄油的物品机制各归本批对应页。

## 注册与构造

`prefablist.lua` 以文件名 `"butterfly"` 登记，文件 `return` 两个 prefab：主体与预览器。[登记](../../../raw/current/scripts/prefablist.lua#L191)、[导出](../../../raw/current/scripts/prefabs/butterfly.lua#L200-L201)

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `butterfly` | `fn()`：加 Transform／AnimState／DynamicShadow／SoundEmitter／Network，`MakeTinyFlyingCharacterPhysics(inst, 1, .5)`，`SetTwoFaced()`，build `butterfly_basic`／bank `butterfly`／播 `idle`；pristine 前一次性写标签表；`SetPristine()` 后主模拟才挂组件 | 标签：`butterfly`／`flying`／`ignorewalkableplatformdrowning`／`insect`／`smallcreature`／`cattoyairborne`／`wildfireprotected`／`deployedplant`／`noember`／`plantcreature`／`pollinator`。[构造](../../../raw/current/scripts/prefabs/butterfly.lua#L74-L118) |
| `butterfly_placer` | `MakePlacer("butterfly_placer", "flowers", "flowers", "f1")` | 只做放置预览，用的是**花**的 bank／build 与 `f1` 动画，不是蝴蝶贴图；无实体行为。[注册](../../../raw/current/scripts/prefabs/butterfly.lua#L201) |

主模拟里的组件顺序（顺序本身有含义：locomotor 必须先于 stategraph）：`locomotor`（关地面速度倍率、不触发 creep）→ `SetStateGraph("SGbutterfly")` → `stackable`＋`inventoryitem`（`canbepickedup = false`、`canbepickedupalive = true`、`nobounce`）→ `pollinator` → `health`（1）→ `combat` → `knownlocations` → 可烧／可冻 → `inspectable` → `lootdropper` → `workable`（NET，1 次）→ `tradable` → `deployable`（PLANT）→ `MakeHauntablePanicAndIgnite` → `SetBrain`。[组件段](../../../raw/current/scripts/prefabs/butterfly.lua#L120-L181)

- `stackable` 没有写 `maxsize`，落组件默认 `STACK_SIZE_MEDITEM`＝20（图鉴同样记 20），所以活蝴蝶在背包里最多叠 20 只。[挂载](../../../raw/current/scripts/prefabs/butterfly.lua#L129)、[组件默认](../../../raw/current/scripts/components/stackable.lua#L23-L28)、[图鉴](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L247)
- 燃烧用 `MakeSmallBurnableCharacter` 并把 `burnable:SetBurnTime(6 * TUNING.PLANTMOB_BURNTIME_MULT)`＝8 秒、`health.fire_damage_scale`＝1.25；冻结用 `MakeTinyFreezableCharacter`。[燃烧与冻结](../../../raw/current/scripts/prefabs/butterfly.lua#L150-L153)、[常量](../../../raw/current/scripts/tuning.lua#L9461-L9462)
- 场景侧：拿到 `TheWorld.components.butterflyspawner` 时登记跟踪，入包与删除都走它的 `StopTrackingFn`。[接线](../../../raw/current/scripts/prefabs/butterfly.lua#L183-L188)

### 名称核验

`NAMES.BUTTERFLY`＝蝴蝶；同族月蛾是 `NAMES.MOONBUTTERFLY`＝月蛾，两者名称不混用。[BUTTERFLY](../../../raw/current/scripts/languages/chinese_s.po#L340024-L340027)、[MOONBUTTERFLY](../../../raw/current/scripts/languages/chinese_s.po#L345609-L345612)

## 捕获、放生与击杀

| 交互 | 触发条件 | 结果 | 证据 |
|---|---|---|---|
| 捕虫网活捉 | 目标带 `NET_workable`（`workable` 的 `workleft > 0` 且可工作）；手上的工具有 `ACTIONS.NET` | `OnWorked` 把**蝴蝶本体**塞进工人背包并播 `butterfly_trap`；`workable` 只 1 次 | [工作次数](../../../raw/current/scripts/prefabs/butterfly.lua#L165-L168)、[回调](../../../raw/current/scripts/prefabs/butterfly.lua#L44-L52)、[动作](../../../raw/current/scripts/actions.lua#L1859-L1878)、[工具标签](../../../raw/current/scripts/components/workable.lua#L1-L25) |
| 直接拾取 | `inventoryitem.canbepickedup = false` | 不走普通捡拾，只能网 | [标志位](../../../raw/current/scripts/prefabs/butterfly.lua#L131-L134) |
| 放生种花 | `deployable` 为 `DEPLOYMODE.PLANT`，`SetDeploySpacing(DEPLOYSPACING.LESS)`＝0.75 | `OnDeploy` 生成 `planted_flower`、`PushEvent("growfrombutterfly")`、从叠里取 1 只删除并发成就 | [OnDeploy](../../../raw/current/scripts/prefabs/butterfly.lua#L54-L66)、[部署模式](../../../raw/current/scripts/prefabs/butterfly.lua#L174-L177)、[间距](../../../raw/current/scripts/constants.lua#L2254-L2263) |
| 放生落点校验 | PLANT 模式 → `CanDeployPlantAtPoint` | 先查陆地且非 `GROUND_HARD`（道路／岩地不能种花），再查落点间距（`DEPLOYSPACING.LESS`＝0.75） | [分支](../../../raw/current/scripts/components/deployable.lua#L104-L130)、[地块判定](../../../raw/current/scripts/components/map.lua#L167-L175)、[间距与落点](../../../raw/current/scripts/components/map.lua#L323-L326) |
| 击杀 | 生命 1，进入 `death` 状态即 `DropLoot` | 按[掉落表](#掉落表)抽 1 件；掉落位置是死亡点 | [死亡状态](../../../raw/current/scripts/stategraphs/SGbutterfly.lua#L50-L65) |

- 网具要求：`bugnet` 用 `tool:SetAction(ACTIONS.NET)` 注册动作，每网消耗 1 点耐久；`ACTIONS.NET.fn` 里还会再校验目标的 `workable` 是 NET 且未死亡。[网具](../../../raw/current/scripts/prefabs/bugnet.lua#L60-L70)、[门禁](../../../raw/current/scripts/actions.lua#L1859-L1878)
- 放生生成的花：`plantedflowerfn` 构造后 `SetPrefabName("flower")` 并置 `inst.planted = true`，因此行为等同普通花（可摘花瓣），但 `CheckForPlanted` 不会把它加进再生管理器。[花工厂](../../../raw/current/scripts/prefabs/flower.lua#L247-L259)、[跳过再生](../../../raw/current/scripts/prefabs/flower.lua#L101-L105)
- `growfrombutterfly` 这个事件本快照里没有监听者：只有成就与 `CHEVO_growfrombutterfly` 被记录，珍珠（`hermitcrab`）监听的是后者。花是靠工厂默认外观生成的，不依赖该事件。[成就](../../../raw/current/scripts/prefabs/butterfly.lua#L60-L61)、[珍珠监听](../../../raw/current/scripts/prefabs/hermitcrab.lua#L1175)

## 掉落表

`lootdropper` 只配了随机表：`AddRandomLoot("butter", 0.1)`、`AddRandomLoot("butterflywings", 5)`、`numrandomloot = 1`。[配置](../../../raw/current/scripts/prefabs/butterfly.lua#L158-L162)

权重不是概率：`PickRandomLoot` 在 `math.random() * 总权重` 上依次扣除，总权重＝0.1＋5＝5.1。[抽取](../../../raw/current/scripts/components/lootdropper.lua#L99-L112)

| 掉落 | 权重 | 实际概率 | 说明 |
|---|---:|---:|---|
| `butterflywings` | 5 | 5 / 5.1 ≈ 98.04% | 每次死亡必出**其中一件**，不是两件都给 |
| `butter` | 0.1 | 0.1 / 5.1 ≈ 1.96% | 写成 0.1 容易被读成 10%，实际约五十分之一 |

## 随身计时与消失

- **术语**：「饿死」是玩家俗称，机制上是活体 perish 计时（`BUTTERFLY_PERISH_TIME`＝1440 秒＝3 天）：进入容器（背包或箱子等）即开始走表；落地暂停；种花式放生只消耗 1 只，留在容器里的其余继续计时；到点触发 `murdered` 事件再按 `GenerateLoot` 结算，不是饥饿值系统。[开关](../../../raw/current/scripts/standardcomponents.lua#L972-L994)、[接线](../../../raw/current/scripts/standardcomponents.lua#L1023-L1033)、[常量](../../../raw/current/scripts/tuning.lua#L1797)

| 现象 | 机制 | 条件 | 证据 |
|---|---|---|---|
| 抓进背包后"饿死" | `MakeFeedableSmallLivestock(inst, TUNING.BUTTERFLY_PERISH_TIME, OnPickedUp, OnDropped)` 加 `perishable`：进入容器才 `StartPerishing`（背包与箱子等容器都会触发 `onputininventory`），落地即 `StopPerishing` | 时长 `BUTTERFLY_PERISH_TIME`＝3 × `total_day_time`＝1440 秒；只在容器／背包里走表 | [接线](../../../raw/current/scripts/prefabs/butterfly.lua#L190)、[常量](../../../raw/current/scripts/tuning.lua#L1797)、[开关](../../../raw/current/scripts/standardcomponents.lua#L972-L994) |
| 到点结算 | `onperish` 先 `Remove` 掉蝴蝶本体，再按叠数把 `GenerateLoot()` 的产物逐组 `container:GiveItem`；本体让出的格子也能接收产物 | 投放战利品需容器（`inventory` 或 `container`）与 `lootdropper`，缺一就只删不投放；是否落地**只看该组 `GiveItem` 的返回值**——能叠进同种或占到空位就入容器，否则掉在原地 | [onperish](../../../raw/current/scripts/standardcomponents.lua#L929-L966)、[拒收落地](../../../raw/current/scripts/components/container.lua#L560-L566) |
| 落地重置 | `OnDropped` 回 `idle`、把 `workable` 重置为 1，并把叠里多余的同件逐只放出 | 落到地面或从容器取出时 | [OnDropped](../../../raw/current/scripts/prefabs/butterfly.lua#L17-L36) |
| 夜里回巢消失 | 行为树在 `not TheWorld.state.isday` 时先取 30 格内最近的花发 `GOHOME`；`SGbutterfly` 的 `land` 在动画结束看到 `GOHOME` 就 `Remove()` | 30 格内没有花时 `GoHomeAction` 返回 nil，蝴蝶继续游荡，不会消失 | [行为树](../../../raw/current/scripts/brains/butterflybrain.lua#L52-L57)、[回巢动作](../../../raw/current/scripts/brains/butterflybrain.lua#L23-L29)、[land 分支](../../../raw/current/scripts/stategraphs/SGbutterfly.lua#L87-L111) |
| 采粉到上限也消失 | `pollinator:HasCollectedEnough()`（`#flowers > 5`）同样触发 `GOHOME` → 上述 `Remove()` | 一朵花只记一次（`CanPollinate` 排除已记录的） | [行为树](../../../raw/current/scripts/brains/butterflybrain.lua#L54-L55)、[判定](../../../raw/current/scripts/components/pollinator.lua#L24-L30) |

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`workable` | NET 动作、1 次；掉落／入包后重置 | 主模拟；`workleft > 0` 才加 `NET_workable` 标签 | [[reference/components/workable]]；[挂载](../../../raw/current/scripts/prefabs/butterfly.lua#L165-L168) |
| 组件：`deployable` | 放生种花，PLANT 模式 | 主模拟；客户端靠 `deployedplant` 标签与 `<prefab>_placer` 预览 | [[reference/components/deployable]]；[挂载](../../../raw/current/scripts/prefabs/butterfly.lua#L174-L177) |
| 组件：`inventoryitem` | 只能活捉，不能直接捡 | `canbepickedup = false`／`canbepickedupalive = true` | [[reference/components/inventoryitem]]；[标志位](../../../raw/current/scripts/prefabs/butterfly.lua#L131-L134) |
| 组件：`stackable` | 活体叠 20（默认值） | 主模拟 | [[reference/components/stackable]]；[默认](../../../raw/current/scripts/components/stackable.lua#L26-L28) |
| 组件：`perishable` | 在容器里 3 天；落地暂停 | 只有 `MakeFeedableSmallLivestock` 加的这一份 | [[reference/components/perishable]]；[开关](../../../raw/current/scripts/standardcomponents.lua#L972-L994) |
| 组件：`pollinator` | 记录采过的花；满 6 朵触发回巢 | 主模拟 | [[reference/components/pollinator]]；[判定](../../../raw/current/scripts/components/pollinator.lua#L28-L30) |
| 组件：`lootdropper` | 随机抽 1 件蝶翅或黄油 | 主模拟；死亡状态里调用 | [[reference/components/lootdropper]]；[配置](../../../raw/current/scripts/prefabs/butterfly.lua#L158-L162) |
| 组件：`eater` | 被喂食时 `oneat` 把 `perishable:SetPercent(1)` 重置为满鲜 | 主模拟（`MakeFeedableSmallLivestock` 挂载）；`FEED` 对蝴蝶的可达性未核验 | [[reference/components/eater]]；[oneat](../../../raw/current/scripts/standardcomponents.lua#L923-L927) |
| 组件：`halloweenmoonmutable` | 被月亮药水命中时替换成 `moonbutterfly` | 主模拟；`push_attacked_on_new_inst = false`，新体不会立刻被"攻击"惊飞 | [[reference/components/halloweenmoonmutable]]；[接线](../../../raw/current/scripts/prefabs/butterfly.lua#L192-L195) |
| 组件：`halloweenpotionmoon`（药水侧） | 对 `halloweenmoonmutable` 目标调用 `Mutate()` | 药水用完即消耗 1 瓶 | [[reference/components/halloweenpotionmoon]]；[Use](../../../raw/current/scripts/components/halloweenpotionmoon.lua#L11-L30) |
| SG／Brain：`SGbutterfly`／`butterflybrain` | 飞行、落地、采粉、死亡、回巢删除 | 主模拟装 SG 与 Brain | [SG](../../../raw/current/scripts/stategraphs/SGbutterfly.lua#L161)、[Brain](../../../raw/current/scripts/brains/butterflybrain.lua#L44-L64) |
| 关联 Prefab：`planted_flower` | 放生产物，内部 prefab 名为 `flower` | 只在 `OnDeploy` 生成 | [注册](../../../raw/current/scripts/prefabs/flower.lua#L263)、[工厂](../../../raw/current/scripts/prefabs/flower.lua#L247-L259) |
| 关联 Prefab：`butterflywings`／`butter` | 掉落产物 | 随机表 1 件 | [[reference/prefabs/butterfly-wings]]、[[reference/prefabs/butter]]；[配置](../../../raw/current/scripts/prefabs/butterfly.lua#L160-L161) |
| 关联 Prefab：`moonbutterfly` | 月亮药水变形目标 | 需要药水与可变目标 | [[reference/prefabs/moonbutterfly]]；[构造](../../../raw/current/scripts/prefabs/moonbutterfly.lua#L104-L218) |
| 资源：`anim/butterfly_basic.zip` | 主体动画包 | 只核验声明，未验证资源内容 | [assets](../../../raw/current/scripts/prefabs/butterfly.lua#L3-L6) |

### 共享规则接入（S1–S7）

- **S1 烤制门禁**：不适用——蝴蝶不是可食物品，没有 `cookable`；它的可食用产出（蝶翅、黄油）见对应物品页；门禁本身为：目标带 `cooker`、未 `fueldepleted`，`dangerouscooker` 需 `doer:HasTag("expertchef")`，骑乘时目标须归自己所有（[门禁](../../../raw/current/scripts/componentactions.lua#L1096-L1105)）。
- **S2 燃烧掉落**：接入——蝴蝶的死亡由 `SGbutterfly` 的 `death` 状态调 `DropLoot`；掉落者已 `burnt`，或正在燃烧且（`fueled == nil` 或 `ignorefuel`）时，蝶翅与黄油都没有熟品、蝴蝶又不是结构或树／石，最终被换成 `ash`。[改写链](../../../raw/current/scripts/components/lootdropper.lua#L415-L442)
- **S3 晾晒**：不适用——蝴蝶没有 `dryable` 组件与标签。
- **S4 烹饪登记**：不适用——蝴蝶本体不进炖锅；`butterflywings`／`butter` 的登记见各自页面。
- **S5 堆叠与标签**：`stackable` 未写 `maxsize`，落组件默认 `STACK_SIZE_MEDITEM`＝20（与物品页的 40 不是一个口径）；显式标签 11 项在 pristine 前写入，其中 `cattoyairborne` 让浣猫把它当空中玩具；蝴蝶不挂 `edible`，也没有 `perishable` 的鲜度三态标签。[标签](../../../raw/current/scripts/prefabs/butterfly.lua#L87-L99)、[默认](../../../raw/current/scripts/components/stackable.lua#L23-L28)
- **S6 腐坏替换**：不适用——随身 3 天是 `MakeSmallPerishableCreature` 的 `onperish` 回调，没有设 `onperishreplacement`，不会变成 `spoiled_food`。[回调](../../../raw/current/scripts/standardcomponents.lua#L972-L994)
- **S7 索引**：索引由主代理统一登记，本页不改 `index.md`。

## 常见陷阱与限制

- **误区：蝴蝶掉黄油是 10%**。常量 0.1 是**权重**，与蝶翅的 5 一起归一化后才约 1.96%。[抽取](../../../raw/current/scripts/components/lootdropper.lua#L99-L112)
- **边界：烧死的蝴蝶掉灰烬**。掉落者已烧毁／正在燃烧且未接燃料组件时，`DropLoot` 依次查 `BURNED_LOOT_OVERRIDES` → `<x>_cooked` → `cooked<x>`；蝶翅与黄油都没有熟品，蝴蝶也不是结构或树／巨石，于是换成 `ash`。[改写链](../../../raw/current/scripts/components/lootdropper.lua#L415-L442)、[覆盖表](../../../raw/current/scripts/tuning.lua#L4511-L4530)
- **边界：蝴蝶自己不会种花**。花只由玩家 `deploy` 生成；`pollinator` 在蝴蝶身上只做采粉记账，产花用的 `CreateFlower` 在蝴蝶链路里没有被调用。[OnDeploy](../../../raw/current/scripts/prefabs/butterfly.lua#L54-L66)、[组件](../../../raw/current/scripts/components/pollinator.lua#L32-L40)
- **边界：抓进背包不是永久保存**。进容器才开始计时，3 天后在容器里"饿死"：先删除本体（它腾出的格子也能用），再把产物逐组交给容器——能叠进同种或占到空位就入容器，该组 `GiveItem` 失败才掉在原地。[计时](../../../raw/current/scripts/standardcomponents.lua#L972-L994)、[结算](../../../raw/current/scripts/standardcomponents.lua#L929-L966)
- **边界：夜里消失要看附近有没有花**。回巢目标是 30 格内最近的花；没有花就继续飞，直到天亮。[行为树](../../../raw/current/scripts/brains/butterflybrain.lua#L15-L29)
- **未核验**：`butterflyspawner` 的刷新预算与季节条件；`FindFlower` 行为节点的内部判定。月蛾本体与"蝴蝶→月蛾"后的行为差异见 [[reference/prefabs/moonbutterfly]]。

## 验证记录

- 源码审阅：2026-11-09，Steam buildid 26474873。核对 `prefabs/butterfly.lua` 全文（标签、组件顺序、NET／deployable／loot／livestock／moon 接线）、`stategraphs/SGbutterfly.lua` 与 `brains/butterflybrain.lua` 全文、`components/pollinator.lua` 全文、`prefabs/flower.lua` 的 `planted_flower` 与再生跳过分支、`components/workable.lua` 的标签维护、`components/deployable.lua` 的 PLANT 分支与 `components/map.lua` 的 `CanPlantAtPoint`、`actions.lua` 的 `ACTIONS.NET`、`prefabs/bugnet.lua` 的工装与耐久、`standardcomponents.lua` 的小型牲畜计时与 `onperish`、`components/halloweenmoonmutable.lua` 与 `components/halloweenpotionmoon.lua` 全文、`prefabs/halloweenpotion_moon.lua` 的组件挂载、`tuning.lua` 的 `BUTTERFLY_PERISH_TIME`／`PLANTMOB_*` 与 `BURNED_LOOT_OVERRIDES`、`constants.lua` 的 `DEPLOYSPACING_RADIUS`、`chinese_s.po` 的名称键、`scrapbookdata.lua` 的蝴蝶记录。
  未覆盖：刷新器预算、`FindFlower` 节点内部、掉落物的拾取者归属、`FEED` 对蝴蝶的可达性（月蛾本体见 [[reference/prefabs/moonbutterfly]]）。
- 游戏运行：未运行。概率由权重表推导（5 / 5.1 与 0.1 / 5.1），天数由 `total_day_time`＝480 秒推导。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 通过（见交付回报）。

## 相关链接

- 刷新与捕捉：[[reference/components/butterflyspawner]]、[[reference/components/workable]]、[[reference/components/deployable]]
- 随身计时：[[reference/components/perishable]]
- 同批页面：[[reference/prefabs/butterfly-wings|蝴蝶翅膀与月蛾翅膀]]、[[reference/prefabs/butter|黄油]]、[[reference/prefabs/honey|蜂蜜与蜂王浆]]
- 月亮变形：[[reference/components/halloweenmoonmutable]]

返回 [[reference/prefabs/index|Prefab 索引]]。
