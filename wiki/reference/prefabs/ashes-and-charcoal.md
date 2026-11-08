---
title: 灰烬与木炭（ash、charcoal）
created: "2026-11-10"
updated: "2026-11-10"
type: prefab
summary: 两种独立注册的燃烧残留物品：灰烬落地后 25＋随机 10 秒自毁，作祟立即触发同一消失序列（醒着先禁点选／拾取再等动画，休眠当场删除），并带命名链（本版唯一设置点无调用，实战通常显示通用名）；木炭是 45 秒燃料、自身可燃 15 秒并蔓延。本页对照两者共有的堆叠／可食 BURNT／molebait／雪人装饰／猪王不收接线，以及烧毁（默认生成阶段追加 40% 木炭，显式列表只走熟品／灰烬改写）、焦树、篝火三类获得链与配方消费。
aliases: ["ash", "charcoal", "灰烬", "木炭", "ashes", "Ashes", "Charcoal", "燃烧残留"]
tags: [prefab, item]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/ash.lua", "prefabs/charcoal.lua", "prefabs/driftwood_trees.lua", "prefabs/marsh_tree.lua", "components/disappears.lua", "components/named.lua", "components/named_replica.lua", "components/inspectable.lua", "components/inventory.lua", "components/tradable.lua", "components/bait.lua", "components/trap.lua", "components/snowmandecoratable.lua", "components/lootdropper.lua", "components/birdspawner.lua", "brains/molebrain.lua", "standardcomponents.lua", "prefabs/campfire.lua", "prefabs/firepit.lua", "prefabs/evergreens.lua", "prefabs/deciduoustrees.lua", "prefabs/livingtree.lua", "prefabs/cave_banana_tree.lua", "prefabs/krampus.lua", "prefabs/klaus_sack.lua", "components/klaussackloot.lua", "prefabs/catcoon.lua", "prefabs/scorched_skeleton.lua", "prefabs/pigking.lua", "prefabs/lavae_pet.lua", "componentactions.lua", "stringutil.lua", "speech_wilson.lua", "tuning.lua", "constants.lua", "recipes.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 灰烬与木炭（ash、charcoal）

## 快速判断

- 职责与适用：两个各自独立文件、各自 `Prefab()` 注册的物品——灰烬（`ash`）与木炭（`charcoal`），没有共享工厂。`ash` 是烧毁后留下的残留（自带消失计时与"某物的灰烬"命名链），`charcoal` 是可继续当燃料的烧焦木材产物。可借鉴：可消失掉落物的 `disappears` 接线、"燃料值／自身燃烧时长"两条数值分开设、以及烧毁掉落的两个入口（默认生成阶段追加 40% 木炭／显式列表只走熟品与灰烬改写）。[ash 注册](../../../raw/current/scripts/prefabs/ash.lua#L107)、[charcoal 注册](../../../raw/current/scripts/prefabs/charcoal.lua#L62)
- 优先使用：改堆叠／可食／燃料参数查[数值与参数](#数值与参数)；让灰烬留存更久查 [disappears 接线](#典型接入)；烧毁后到底产出哪种物品见[获得链](#获得链)，配方消费见[用途与配方](#用途与配方)。
- 重要副作用：`ash` 落地（含刚生成时）即排 25–35 秒计时；到期或作祟都进入同一消失序列——醒着先 `persists=false`、加 `NOCLICK`、禁拾取，再播动画并等 `animover`（另有动画长度＋0.1 秒兜底），休眠（离屏）则当场 `Remove`。木炭既可当燃料（`fuel.fuelvalue`＝45 秒）又可被点燃并蔓延（`MakeMediumBurnable`＝15 秒＋`MakeMediumPropagator`）。[ash 计时](../../../raw/current/scripts/prefabs/ash.lua#L93-L94)、[消失序列](../../../raw/current/scripts/components/disappears.lua#L12-L48)、[charcoal 燃料与可燃](../../../raw/current/scripts/prefabs/charcoal.lua#L34-L47)
- 修改边界：两者都是 BURNT 食性，普通角色吃不到（没有 `BURNT_eater` 标签）；`bait` 只表示能放进陷阱诱饵槽，不代表能吸引鸟。本页不覆盖烧毁源头实体（树、建筑）自身的完整机制，只列它们产出本页物品的规则。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `ash` | 单文件单注册 `Prefab("ash", fn, assets)`，无包装工厂。`fn` 公共端建实体＋`MakeInventoryPhysics`＋bank `ashes`／build `ash`，加 `molebait`、`ashes`、`_named` 三标签后 `SetPristine`；主模拟先 `RemoveTag("_named")` 再挂组件。 | 主模拟组件顺序：`disappears`（sound `dust_blowaway`、anim `disappear`）→ `stackable`(40) → `inspectable`(getstatus) → `inventoryitem`（拾取回调＋`SetSinks(true)`）→ `named`（nameformat＝`ASH_REMAINS`）→ `bait` → `edible`(BURNT 20/20) → `tradable` → `stacksizechange`／`ondropped` 监听 → `hauntable` → `snowmandecor`。无 `fuel`、无 `burnable`、无 `MakeInventoryFloatable`。[组件段](../../../raw/current/scripts/prefabs/ash.lua#L65-L102) |
| `charcoal` | 单文件单注册 `Prefab("charcoal", fn, assets)`。公共端 bank／build `charcoal`＋`pickupsound="wood"`＋`molebait`＋`MakeInventoryFloatable("med", 0.05, 0.6)`，`SetPristine` 后主模拟。 | 组件：`stackable`(40)、`fuel`(`MED_FUEL`)、`edible`(BURNT 20/20)、`tradable`、`bait`、`MakeMediumBurnable(MED_BURNTIME)`＋`MakeMediumPropagator`、`MakeHauntableLaunchAndIgnite`、`inspectable`、`inventoryitem`、`snowmandecor`；后三项在组件段末尾才挂。[组件段](../../../raw/current/scripts/prefabs/charcoal.lua#L31-L57) |

### 名称与实显

简中 `.po` 键：`NAMES.ASH`＝灰烬（Ashes）、`NAMES.ASH_REMAINS`＝"%s的灰烬"（Ashes of %s）、`NAMES.CHARCOAL`＝木炭（Charcoal）。[ASH](../../../raw/current/scripts/languages/chinese_s.po#L338880-L338882)、[ASH_REMAINS](../../../raw/current/scripts/languages/chinese_s.po#L338884-L338887)、[CHARCOAL](../../../raw/current/scripts/languages/chinese_s.po#L340785-L340787)

`ash` 的可变名由 `named` 组件承担：`SetName(name)` 保存原始名，经 `string.format(nameformat, name)` 写成 `inst.name`，同时通过 `replica.named` 同步网络名；`SetName(nil)` 回落到 `STRINGS.NAMES[UPPER(prefab)]`（即"灰烬"）并把网络名置空。[DoSetName](../../../raw/current/scripts/components/named.lua#L9-L13)、[SetName](../../../raw/current/scripts/components/named.lua#L22-L31)、[客户端回落](../../../raw/current/scripts/components/named_replica.lua#L1-L4)

`inspectable.getstatus` 拼的是"检查状态键"而不是显示名：有名字时返回 `"REMAINS_"..named.name`（空格换下划线），由玩家台词表的 `DESCRIBE.ASH.<状态>` 取值；简中台词表只为 `REMAINS_GLOMMERFLOWER`、`REMAINS_EYE_BONE`、`REMAINS_THINGIE` 三条写了条目，其余落通用描述。[getstatus](../../../raw/current/scripts/prefabs/ash.lua#L9-L13)、[状态键消费](../../../raw/current/scripts/stringutil.lua#L364-L404)、[台词表](../../../raw/current/scripts/speech_wilson.lua#L1949-L1955)

设置路径（本版实际）：`scripts` 内唯一给 `ash` 调 `named:SetName` 的地方是 `Inventory:BurnNonpotatableInContainer`，而该函数在 `scripts` 全部 `.lua` 中只有定义、没有调用点；本页核验的 `ash` 生成点（`DefaultBurntFn`、篝火／火坑、焦树掉落）都不设名字，所以实战中 `getstatus` 通常返回 nil、显示通用描述。堆叠数 >1 时 `stacksizechange` 监听会 `SetName(nil)` 清名，避免整摞都叫"某物的灰烬"。[唯一设置点](../../../raw/current/scripts/components/inventory.lua#L1968-L1977)、[清名](../../../raw/current/scripts/prefabs/ash.lua#L19-L23)

## 数值与参数

tuning 基准：`seg_time`＝30；堆叠 `STACK_SIZE_SMALLITEM`＝40／`MEDITEM`＝20；`MED_FUEL`＝30×1.5＝45 秒；`MED_BURNTIME`＝30×0.5＝15 秒。[堆叠](../../../raw/current/scripts/tuning.lua#L81-L83)、[燃料与燃烧](../../../raw/current/scripts/tuning.lua#L1674-L1682)

| 注册名 | 堆叠 | 燃料值 | 自身燃烧 | 可食（foodtype） | 治疗／饥饿／理智 | 交易 | 雪人装饰 | 水面 |
|---|---|---|---|---|---|---|---|---|
| `ash` | 40（显式设） | 无 `fuel` | 无 `burnable` | BURNT | 20／20／0（理智用组件默认） | `tradable`，`goldvalue` 未设→0 | 有表项，无 `canflip` | 无 floater＋`SetSinks(true)` |
| `charcoal` | 40（显式设） | `MED_FUEL`＝45 秒 | `MED_BURNTIME`＝15 秒＋中距蔓延 | BURNT | 20／20／0 | `tradable`，`goldvalue` 未设→0 | 有表项，无 `canflip` | `("med", 0.05, 0.6)` |

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| `组件：disappears` | 仅 ash：初始化先 `PrepareDisappear()`，`ondropped` 重新计时；拾取经 `inventoryitem:SetOnPutInInventoryFn(OnPickup)` → `StopDisappear`；作祟 `OnHaunt` → `Disappear()` | 主模拟。`delay`＝25，`PrepareDisappear` 取 25＋`math.random()*10` 秒；`Disappear` 先取消任务，再按**醒着／休眠**分叉：醒着依次 `persists=false` → 加 `NOCLICK` → `canbepickedup=false`（含 `canbepickedupalive`）→ 播音 → 播 `disappear` 动画，等 `animover` 或动画长度＋0.1 秒兜底后 `Remove`；休眠（离屏）不播动画、当场 `Remove`。`isdisappear` 守卫使重复作祟／到期不重入 | [[reference/components/disappears#关键接口]]；[接线](../../../raw/current/scripts/prefabs/ash.lua#L15-L32)、[计时](../../../raw/current/scripts/components/disappears.lua#L68-L77)、[消失序列](../../../raw/current/scripts/components/disappears.lua#L12-L49) |
| `组件：named`／`replica.named` | 仅 ash：`nameformat`＝`ASH_REMAINS`；`SetName` 写 `inst.name` 与 net_string | 主模拟写，客户端靠 `named_replica` 的 `namedirty` 回落 | [[reference/components/named#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/ash.lua#L79-L80)、[Replica](../../../raw/current/scripts/components/named_replica.lua#L1-L4) |
| `组件：edible` | 两者都 BURNT 20/20、无理智值；可食不等于玩家可食 | 默认玩家食性 `FOODGROUP.OMNI` 不含 BURNT；`EAT` 动作要求物品带 `edible_<type>` 且玩家带 `<type>_eater`，本版只有 `lavae_pet` 设了 `{FOODTYPE.BURNT}` | [[reference/components/edible#关键接口]]；[ash 可食](../../../raw/current/scripts/prefabs/ash.lua#L84-L87)、[OMNI](../../../raw/current/scripts/constants.lua#L2049-L2062)、[EAT 门禁](../../../raw/current/scripts/componentactions.lua#L2683-L2687)、[lavae_pet 食性](../../../raw/current/scripts/prefabs/lavae_pet.lua#L150) |
| `组件：bait`＋`molebait` 标签 | `bait` 只让物品进陷阱诱饵槽；`molebait` 让鼹鼠来偷（鼹鼠还要求物品带 `bait` 组件） | 鸟的落点判定另需 `eater:CanEat`，BURNT 不在鸟食性里；陷阱弹起时的诱饵查找同样先看 `bait` 组件 | [[reference/components/bait#关键接口]]；[诱饵动作](../../../raw/current/scripts/componentactions.lua#L1046-L1050)、[鸟的判定](../../../raw/current/scripts/components/birdspawner.lua#L536-L546)、[鼹鼠](../../../raw/current/scripts/brains/molebrain.lua#L56-L70) |
| `组件：tradable` | 二者都挂 `tradable` 但不设 `goldvalue`（默认 0） | 猪王 `AcceptTest` 只收 `goldvalue>0`（或节日糖果／`pig_token`），交付被拒并播不悦 | [[reference/components/tradable#关键接口]]；[默认 0](../../../raw/current/scripts/components/tradable.lua#L1-L7)、[猪王](../../../raw/current/scripts/prefabs/pigking.lua#L598-L606) |
| `组件：snowmandecor`＋装饰数据表 | 二者都能当雪人装饰；渲染数据在 `ITEM_DATA[hash(注册名)]` | 表项：`ash`→bank `ashes`／build `ash`；`charcoal`→同名；两者都无 `canflip`（装饰不可翻转） | Wiki 待建；[挂载](../../../raw/current/scripts/prefabs/charcoal.lua#L57)、[表项](../../../raw/current/scripts/components/snowmandecoratable.lua#L5-L14)、[取表](../../../raw/current/scripts/components/snowmandecoratable.lua#L147-L149) |
| `组件：fuel`／`burnable`／`propagator` | 仅 charcoal：45 秒燃料；自身 15 秒可燃＋中距蔓延（flashpoint 15–25、范围 5–7） | 主模拟；烧完走 `DefaultBurntFn` | [[reference/components/fuel#关键接口]]、[[reference/components/burnable#关键接口]]；[charcoal](../../../raw/current/scripts/prefabs/charcoal.lua#L34-L47)、[helper](../../../raw/current/scripts/standardcomponents.lua#L176-L227) |
| `组件：hauntable` | ash 用自定义 `OnHaunt`（直接 `Disappear`，`usefx=false`、`HAUNT_TINY`）；charcoal 用 `MakeHauntableLaunchAndIgnite` | 后者点燃段被 `#HAUNTFIX` 整段注释，实际只弹飞，并且回调返回 false | [[reference/components/hauntable#按效果选工厂]]；[ash 作祟](../../../raw/current/scripts/prefabs/ash.lua#L29-L32)、[LaunchAndIgnite](../../../raw/current/scripts/standardcomponents.lua#L1191-L1240) |
| `API／数据：lootdropper` | 烧毁产出分两个入口：默认列表（`DropLoot()` 或只传落点 `DropLoot(pt)`，`prefabs` 为空）在 `GenerateLoot` 里按 `burnt` 标签追加 40% 一个木炭；显式列表只有作为**第二参**传入（`DropLoot(pt, prefabs)`，落点可传 nil）时才跳过生成、没有这 40%；只传一个数组会占据 `pt` 位，`prefabs` 仍为空、照旧走 `GenerateLoot`。两入口之后都走同一改写块。烧毁建筑配方掉落按 `BURNT_HAMMER_LOOT_PERCENT`＝0.25 折算 | 服务器；40% 每次 `GenerateLoot` 掷一次 | [[reference/components/lootdropper#关键接口]]；[40% 木炭](../../../raw/current/scripts/components/lootdropper.lua#L252-L254)、[默认入口](../../../raw/current/scripts/components/lootdropper.lua#L415-L416)、[改写块](../../../raw/current/scripts/components/lootdropper.lua#L418-L441)、[显式列表调用示例](../../../raw/current/scripts/prefabs/corpses.lua#L489-L494)、[burnt 折算](../../../raw/current/scripts/components/lootdropper.lua#L134-L160) |
| `API／数据：MakeInventoryFloatable 参数` | 木炭 `("med", 0.05, 0.6)`＝尺寸档、垂直偏移、缩放；灰烬完全没调该 helper | 公共端（双端） | [[reference/components/inventoryitem#关键接口]]；[helper](../../../raw/current/scripts/standardcomponents.lua#L1772-L1791)、[charcoal 调用](../../../raw/current/scripts/prefabs/charcoal.lua#L23) |
| `关联 Prefab：campfire`／`firepit` | 烧尽残留：篝火给 1 ash（曾加到最高燃料段则 1 charcoal）；火坑同条件只给 1 charcoal。锤击路径只有火坑：玩家可锤掉火坑并得 1 ash；篝火的 `workable` 动作是 nil、只带 `NPC_workable`，玩家没有锤击入口（`onhammered` 只服务 NPC／AOE 工作） | 服务器 | [campfire 烧尽](../../../raw/current/scripts/prefabs/campfire.lua#L46-L62)、[campfire workable](../../../raw/current/scripts/prefabs/campfire.lua#L169-L171)、[NPC_workable](../../../raw/current/scripts/prefabs/campfire.lua#L145)、[firepit 烧尽](../../../raw/current/scripts/prefabs/firepit.lua#L52-L68)、[firepit HAMMER](../../../raw/current/scripts/prefabs/firepit.lua#L204-L208)、[火坑锤掉](../../../raw/current/scripts/prefabs/firepit.lua#L16-L24) |
| `关联 Prefab：evergreens`／`deciduoustrees`／`livingtree`／`driftwood_trees`／`cave_banana_tree` | 焦树砍倒：常青树／桦树／活树 1 个保底 charcoal，另按 `burnt` 规则 40%；浮木树与沼泽树烧毁后 `SetLoot({"charcoal"})`，同样只给木炭并吃 40%；洞穴香蕉焦干是独立注册、只给 1 个 | 服务器 | [[reference/prefabs/evergreen-trees]]、[[reference/prefabs/deciduous-trees]]、[[reference/prefabs/driftwood-livinglog]]；[evergreens](../../../raw/current/scripts/prefabs/evergreens.lua#L152-L163)、[driftwood](../../../raw/current/scripts/prefabs/driftwood_trees.lua#L124-L138)、[cave_banana](../../../raw/current/scripts/prefabs/cave_banana_tree.lua#L250-L257) |
| `关联 Prefab：krampus`／`klaus_sack`／`catcoon` | 击杀坎普斯 2 木炭；克劳斯的袋子开出的包裹含木炭；浣猫友善 tier 2 礼物含灰烬 | 服务器 | [krampus](../../../raw/current/scripts/prefabs/krampus.lua#L18-L23)、[klaussackloot](../../../raw/current/scripts/components/klaussackloot.lua#L93-L113)、[catcoon tier 2](../../../raw/current/scripts/prefabs/catcoon.lua#L130-L137) |

## 获得链

烧毁产出先分「实体自身燃尽消失」与「掉落列表从哪来」两层，再按掉落时刻的状态（`burnt` 标签／是否在燃烧／是否受控燃烧）判定改写：

1. **烧完消失给灰烬**：绝大多数可燃烧实体用 `DefaultBurntFn`——燃尽时若不在海面，就地生成 1 个 `ash`，并把原实体的 `stackable.stacksize` 复制给灰烬（不超过其 40 上限），随后移除原实体。这条路线不产木炭。[DefaultBurntFn](../../../raw/current/scripts/standardcomponents.lua#L18-L43)
2. **默认列表（`DropLoot()` 或只传落点 `DropLoot(pt)`）**：先走 `GenerateLoot` —— 带 `burnt` 标签时按 40% 追加 1 个 `charcoal`（每次调用只掷一次），生成出的整张列表随后仍要过路线 3 的改写块；配方材料另按 25% 折算。[40% 追加](../../../raw/current/scripts/components/lootdropper.lua#L252-L254)、[默认入口](../../../raw/current/scripts/components/lootdropper.lua#L415-L416)、[25% 折算](../../../raw/current/scripts/components/lootdropper.lua#L134-L160)
3. **显式列表（`DropLoot(pt, prefabs)`）与改写块**：只有把列表作为**第二参**传入时才跳过 `GenerateLoot`、**没有** 40% 追加（第一参是落点，传 nil 让物品原地掉落即可；只传一个数组会被当成 `pt`，`prefabs` 仍为空、照旧走生成并保留 40% 追加）；但改写块仍按状态门禁生效——只有“带 `burnt` 标签”，或“正在燃烧且（`fueled` 为 nil 或 `burnable.ignorefuel`）”才进入改写，带 `fueled` 的燃烧结构（如火坑、龙鳞火炉）不改灰。逐项改写顺序：`TUNING.BURNED_LOOT_OVERRIDES`（如各类种子→`seeds_cooked`、`pondfish`→`fishmeat_small_cooked`、`tree_rock_seed`→`ash`）→ `_cooked` 变体 → `cooked` 前缀 → `burnable:GetControlledBurn()` 为真时原样保留（掉落物会被点燃／smoulder）→ 其余（非结构且非 `tree`／`boulder`，或蜂巢）改成 `ash`。[前置条件](../../../raw/current/scripts/components/lootdropper.lua#L415-L421)、[改写顺序](../../../raw/current/scripts/components/lootdropper.lua#L424-L441)、[覆盖表](../../../raw/current/scripts/tuning.lua#L4511-L4530)、[显式列表调用示例](../../../raw/current/scripts/prefabs/corpses.lua#L489-L494)、[掉落物点燃](../../../raw/current/scripts/components/lootdropper.lua#L404-L407)

| 来源 | 产出 | 触发／条件 | 证据 |
|---|---|---|---|
| 常青树焦干（含 `evergreen_sparse`、多枝树） | charcoal 1 个＋40% 追加 1 个（另有 10 秒自然掉落路径，见该页） | 烧焦后变 `burnt` 状态、砍倒时先 `SpawnLootPrefab("charcoal")` 再 `DropLoot`；40% 按每次 `DropLoot` 重掷 | [chopping](../../../raw/current/scripts/prefabs/evergreens.lua#L152-L163)、[`burnt` 标签](../../../raw/current/scripts/prefabs/evergreens.lua#L208) |
| 桦树焦干（`deciduoustrees`） | 同上 | 砍倒分支相同 | [chopping](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L627-L637)、[`burnt` 标签](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L706) |
| 活树焦干（`livingtree`） | 同上 | 烧毁后 `Extinguish` 清空 loot 并改写 workable 回调 | [chopping](../../../raw/current/scripts/prefabs/livingtree.lua#L14-L25)、[`burnt` 标签](../../../raw/current/scripts/prefabs/livingtree.lua#L49) |
| 浮木树（`driftwood_tall`／`small1`／`small2`）与沼泽树（`marsh_tree`） | charcoal 1 个＋40% 追加 1 个 | 烧毁时 `SetLoot({"charcoal"})` 并加 `burnt` 标签，砍倒 `DropLoot` | [driftwood on_burnt](../../../raw/current/scripts/prefabs/driftwood_trees.lua#L124-L138)、[marsh_tree on_burnt](../../../raw/current/scripts/prefabs/marsh_tree.lua#L83-L95) |
| 洞穴香蕉树焦干（`cave_banana_burnt`） | charcoal 1 个（无 40% 追加） | 焦干是独立注册、无 `burnt` 标签，砍倒只 `SpawnLootPrefab`、不调 `DropLoot` | [burnt_chopped](../../../raw/current/scripts/prefabs/cave_banana_tree.lua#L250-L257) |
| 篝火烧尽 | 1 ash；若曾加到最高燃料段则 1 charcoal | `onfuelchange` 在燃料归零时按 `queued_charcoal` 二选一 | [onfuelchange](../../../raw/current/scripts/prefabs/campfire.lua#L46-L62) |
| 火坑烧尽 | 仅当 `queued_charcoal` 时 1 charcoal，否则无残留 | 火坑本体不消失，锤掉才另给 1 ash | [onfuelchange](../../../raw/current/scripts/prefabs/firepit.lua#L52-L68)、[onhammered](../../../raw/current/scripts/prefabs/firepit.lua#L16-L24) |
| 坎普斯（`krampus`） | charcoal 2 个必掉 | 共享掉落表中两条 1.0，命中即各自入列 | [loot 表](../../../raw/current/scripts/prefabs/krampus.lua#L18-L23) |
| 克劳斯的袋子（`klaus_sack`） | charcoal 合计 3–8 个 | 开袋后逐行 `DropBundle`；含木炭的 3 行用 `FillItems` 把该行补齐到 3–4 件，故每行 1–3 个 | [GetLoot 循环](../../../raw/current/scripts/prefabs/klaus_sack.lua#L66-L70)、[`FillItems`](../../../raw/current/scripts/components/klaussackloot.lua#L64-L68)、[三行木炭](../../../raw/current/scripts/components/klaussackloot.lua#L93-L113) |
| 浣猫友善礼物（`catcoon`） | ash 1 个 | 友善时按 tier 从 `friendGiftPrefabs` 抽一件，tier 2 含 `ash` | [table](../../../raw/current/scripts/prefabs/catcoon.lua#L130-L137)、[抽取](../../../raw/current/scripts/prefabs/catcoon.lua#L265-L273) |
| 烧焦骷髅（`scorched_skeleton`） | ash 1–4 个（1.0／0.75／0.5／0.25 四条独立判定）＋ boneshard | 锤碎时 `DropLoot` | [loot 表](../../../raw/current/scripts/prefabs/scorched_skeleton.lua#L14-L21) |

## 用途与配方

- **燃料**：仅 `charcoal` 能当燃料喂 `fueled` 对象（每次 45 秒）；`ash` 无 `fuel` 组件，不能当燃料也不会自燃。[charcoal fuel](../../../raw/current/scripts/prefabs/charcoal.lua#L34-L35)
- **配方**（`recipes.lua` 实际登记）：`ash` 用于 healing salve×2、healingsalve_fumarole（火山灰膏）×2、sisturn×1、graveurn×1、spider_healer_item×2、soil_amender×1、shellweaver 空瓶×4、wintercooking_pumpkinpie×1，并在流浪商人商店里当货币（cutgrass 2／twigs 2／flint 1／cutreeds 4／moonglass 4）；`charcoal` 用于 cookpot×6、portablecookpot×6、meatrack×2、featherpencil×1、tillweedsalve×1、dragonflyfurnace×10、blowdart_fire×1、boomerang×1、gunpowder×1、boat_cannon_kit×4、record×1、walking_stick×1、halloween_experiment_volatile×1、wintercooking_roastturkey×1。[healingsalve](../../../raw/current/scripts/recipes.lua#L414)、[healingsalve_fumarole](../../../raw/current/scripts/recipes.lua#L1084)、[cookpot](../../../raw/current/scripts/recipes.lua#L475)、[dragonflyfurnace](../../../raw/current/scripts/recipes.lua#L435)、[商店用灰烬](../../../raw/current/scripts/recipes.lua#L997-L1000)
- **雪人装饰**：两者都可被装饰到雪人上（数据表在上文导航）；装饰消耗的是实际物品实例。

## 典型接入

### 让灰烬落地后保留更久

- 前提与选择：服务端 `modmain.lua`；只改 `disappears.delay`，不重建 prefab。不要为"永久保留"单独删 `disappears`：拾取、落地、作祟三处回调都直接索引该组件（`ash.lua#L15-L31`、`#L65-L77`、`#L91-L100`），单独移除会让三个入口访问 nil；确需永久灰烬，必须由模组分别接管拾取、落地、作祟三个入口并逐项验证。
- 生效与撤销：`AddPrefabPostInit` 对之后构造（含读档重建）的实体生效；`delay` 只在 `PrepareDisappear` 时读取，已排定的任务不会自动改时长，所以要主动重排一次。移除模组即恢复默认。

```lua
-- modmain.lua：把灰烬的消失窗口从 25-35 秒延长到 90-100 秒
AddPrefabPostInit("ash", function(inst)
    if GLOBAL.TheWorld.ismastersim and inst.components.disappears ~= nil then
        inst.components.disappears.delay = 90
        if inst.components.disappears.disappeartask ~= nil then
            inst.components.disappears:PrepareDisappear() -- 先取消再按 90 + random(0,10) 重排
        end
    end
end)
```

- 调整与验证：单位是秒，实际值＝`delay`＋0–10 秒的随机；`PrepareDisappear` 内部先 `StopDisappear`，重复调用安全；`disappears` 只在主模拟存在，客户端判空。[PrepareDisappear](../../../raw/current/scripts/components/disappears.lua#L68-L77)、[`OnRemoveFromEntity`](../../../raw/current/scripts/components/disappears.lua#L85)

## 常见陷阱与限制

- **边界：消失序列分醒着与休眠两条**——到期或作祟都走 `Disappear`：醒着时先取消任务、再 `persists=false`／`NOCLICK`／`canbepickedup=false`，之后才播动画，并在 `animover` 或动画长度＋0.1 秒兜底任务后 `Remove`；休眠（离屏）则当场 `Remove`，不播动画也不留可交互实体。两种情况下序列开始后都无法再捡；拾取只在序列前有效，掉地会重抽 25–35 秒，反复捡放可以续命，但作祟不可逆。`isdisappear` 守卫让重复触发不重入。[消失序列](../../../raw/current/scripts/components/disappears.lua#L12-L48)
- **误区：有 `named` 就会显示"某物的灰烬"**——本版唯一设置点 `BurnNonpotatableInContainer` 在 `scripts` 内没有调用者，`ash` 的生成点都不设名；而且 `getstatus` 返回的是 `REMAINS_<原名>` 状态键，台词表只定义了 3 条，其余落到通用描述。检索范围：`scripts` 全部 `.lua` 的 `"REMAINS_"` 与 `BurnNonpotatable`。[设置点](../../../raw/current/scripts/components/inventory.lua#L1968-L1977)、[台词表](../../../raw/current/scripts/speech_wilson.lua#L1949-L1955)
- **误区：挂 `bait` 就能吸引鸟**——鸟的落点判定还要 `eater:CanEat`，BURNT 不在任何鸟食性里；真正生效的是 `molebait`（鼹鼠偷）与陷阱弹起时的诱饵查找。[鸟判定](../../../raw/current/scripts/components/birdspawner.lua#L536-L546)
- **误区：`tradable` 等于猪王收货**——`goldvalue` 默认 0，两者都被拒。[tradable 默认](../../../raw/current/scripts/components/tradable.lua#L1-L7)、[猪王](../../../raw/current/scripts/prefabs/pigking.lua#L598-L606)
- **边界：木炭的 45 秒与 15 秒是两条数值**——45 是喂给火的 `fuelvalue`，15 是它自己被点燃后烧完的时长；烧完走 `DefaultBurntFn`，生成 1 个 ash 并复制堆叠数（整摞 40 个木炭烧完只留 1 个 40 层灰烬）。[fuel](../../../raw/current/scripts/prefabs/charcoal.lua#L34-L35)、[burnable](../../../raw/current/scripts/prefabs/charcoal.lua#L46-L47)、[复制堆叠](../../../raw/current/scripts/standardcomponents.lua#L34-L38)
- **边界：作祟"点燃"不发生**——`MakeHauntableLaunchAndIgnite` 的点燃段被 `#HAUNTFIX` 整段注释（保留历史意图），实际只有 Launch 弹飞，且回调返回 false；不要把木炭写成"作祟会自燃"。[LaunchAndIgnite](../../../raw/current/scripts/standardcomponents.lua#L1191-L1240)
- **边界：`ashes` 标签没有 `HasTag` 消费者**——检索 `"ashes"` 共 6 处：`AddTag` 一处（`ash.lua#L49`），其余 5 处都是 bank／装饰表字段（`ash.lua#L44`、`marsh_bush.lua#L183`、`wargshrine.lua#L63`、`snowmandecoratable.lua#L5`、`screens/redux/scrapbookdata.lua#L71`）；不要假设它参与配方或食物判定。

## 验证记录

- 源码审阅：2026-11-10，Steam buildid 26474873。核验范围：两份 prefab 全文与全部组件接线；`disappears` 的计时与消失序列；`named`／`named_replica` 的设置、同步与回落；`inspectable`→`stringutil` 的状态键消费；`edible`／OMNI 食性与 EAT 门禁；`bait`／`molebait` 与鸟的判定；`tradable` 与猪王；雪人装饰表项；`lootdropper` 的默认／显式列表两个入口、40% 木炭与熟品／灰烬改写块；篝火／火坑；常青树／桦树／活树／洞穴香蕉焦干；坎普斯、克劳斯袋、浣猫、烧焦骷髅；`recipes.lua` 消费行与简中名称键。未覆盖：其他树种（月树、猴面包树等）与烧毁建筑的完整枚举；篝火 `onhammered` 的 NPC／AOE 触发者；`disappears` 的客户端表现；`GetControlledBurn` 的上游设置点（本快照仅见读取）；`_named` 标签的引擎侧消费者。
- 游戏运行：未运行。计时范围、掉落概率、堆叠复制、修复与可食数值均为源码推导，非运行测量。
- 结构检查：`validate_page.py` 与 `validate_source_links.py` 见交付回报（同批未落地页的链接另行说明）。

相关：[[reference/prefabs/evergreen-trees]]（烧毁株两次结算）、[[reference/prefabs/deciduous-trees]]（桦树焦干）、[[reference/prefabs/driftwood-livinglog]]（浮木／活木木料链）、[[reference/prefabs/basic-gathered-materials]]（基础采集材料的对照）。

返回 [[reference/prefabs/crafting-materials-index|通用材料与加工中间件（暂行索引）]]、[[reference/prefabs/index|Prefab 索引]]。
