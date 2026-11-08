---
title: "寄居蟹茶（hermitcrabtea_petals、hermitcrabtea_petals_evil、hermitcrabtea_foliage、hermitcrabtea_succulent_picked、hermitcrabtea_firenettles、hermitcrabtea_tillweed、hermitcrabtea_moon_tree_blossom、hermitcrabtea_forgetmelots）"
created: "2026-11-09"
updated: "2026-11-09"
type: prefab
summary: "八瓶寄居蟹茶共用 MakeTea 工厂：瓶身是 10 口份数的 finiteuses 计数，喝空后返还空瓶并按喂食者→持有者→容器→掉落逐级找落点；六瓶另挂同名 hermitcrabtea_*_buff 实体结算理智、生命、负光环与暗影惊吓。"
aliases: ["hermitcrabtea_petals", "hermitcrabtea_petals_evil", "hermitcrabtea_foliage", "hermitcrabtea_succulent_picked", "hermitcrabtea_firenettles", "hermitcrabtea_tillweed", "hermitcrabtea_moon_tree_blossom", "hermitcrabtea_forgetmelots", "花瓣茶", "深色花瓣茶", "蕨叶茶", "多肉茶", "火荨麻茶", "犁地草茶", "月树花茶", "必忘我茶", "寄居蟹茶", "喝茶退瓶"]
tags: [prefab, item, food]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/hermitcrabtea.lua", "prefabs/hermitcrabtea_defs.lua", "components/finiteuses.lua", "prefabs/hermitcrab_teashop.lua", "prefabs/warly.lua", "prefabs/wendy.lua", "mainfunctions.lua", "brains/catcoonbrain.lua", "actions.lua", "componentactions.lua", "components/builder.lua", "widgets/widgetutil.lua", "techtree.lua", "constants.lua", "components/edible.lua", "components/eater.lua", "components/debuff.lua", "components/debuffable.lua", "components/timer.lua", "components/sanity.lua", "components/temperature.lua", "util/sourcemodifierlist.lua", "scheduler.lua", "entityscript.lua", "componentutil.lua", "brains/braincommon.lua", "prefabs/gestalt.lua", "prefabs/player_common.lua", "standardcomponents.lua", "recipes.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 寄居蟹茶（hermitcrabtea_petals、hermitcrabtea_petals_evil、hermitcrabtea_foliage、hermitcrabtea_succulent_picked、hermitcrabtea_firenettles、hermitcrabtea_tillweed、hermitcrabtea_moon_tree_blossom、hermitcrabtea_forgetmelots）

## 快速判断

- 职责与适用：`prefabs/hermitcrabtea.lua` 用一份 `MakeTea` 工厂注册 8 瓶茶，注册名一律是 `hermitcrabtea_` 接 `hermitcrabtea_defs` 里的 `name`；每瓶 10 口，一口扣 1 次 `finiteuses`，喝空时移除瓶身并生成 `messagebottleempty`。[工厂](../../../raw/current/scripts/prefabs/hermitcrabtea.lua#L54-L150)、[注册循环](../../../raw/current/scripts/prefabs/hermitcrabtea.lua#L243-L257)
- 优先使用：查每口的饥饿／生命／理智／温度看[每口结算](#每口结算)；查份数、喝空行为与空瓶去向看[份数与退瓶](#份数与退瓶)；查配方、柜台与材料看[获取链](#获取链)；查六种 `_buff` 的周期结算看[关联机制导航](#关联机制导航)。
- 重要副作用：喝空最后一口会在同一帧 `inst:Remove()` 并生成空瓶，落点按喂食者→持有者背包→持有者容器→物理掉落逐级判定；同一目标身上已有同名 buff 时再喝，不会新建 buff 实体，而是走 `Debuff:Extend` 重置计时；周期任务是否 `Cancel` 重建取决于茶种，见[常见陷阱](#常见陷阱与限制)。[退瓶](../../../raw/current/scripts/prefabs/hermitcrabtea.lua#L10-L35)、[扩展](../../../raw/current/scripts/components/debuffable.lua#L95-L108)
- 修改边界：本页只负责 8 瓶茶与它们的 8 个 `_buff`；茶柜台本体（`hermitcrab_teashop` 的进入、酿造与珍珠装饰分档）、干燥花瓣（`dryable` 产出）与珍珠隐士本体各归其源码入口，本页不展开。多肉茶与火荨麻茶**不挂 buff**，理由见[常见陷阱](#常见陷阱与限制)。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `hermitcrabtea_petals` | `MakeTea(TEA_DEFS[1])`：`prefabname = "hermitcrabtea_"..data.name`，其余全走共用 `fn` | 理智 +`TUNING.SANITY_SMALL`（10）、生命默认 1；`buff = hermitcrabtea_petals_buff`。[定义](../../../raw/current/scripts/prefabs/hermitcrabtea_defs.lua#L4-L8) |
| `hermitcrabtea_petals_evil` | 同上 | 理智 −`TUNING.SANITY_MED`（−15）、生命 1；buff `hermitcrabtea_petals_evil_buff`；`sanityvalue` 为负会被 `edible` 打上 `badfood`，非 PVP 下喂不了其他玩家（见陷阱）。[定义](../../../raw/current/scripts/prefabs/hermitcrabtea_defs.lua#L10-L14) |
| `hermitcrabtea_foliage` | 同上 | 理智 +`TUNING.SANITY_TINY`（5）、生命 1；buff `hermitcrabtea_foliage_buff`。[定义](../../../raw/current/scripts/prefabs/hermitcrabtea_defs.lua#L16-L20) |
| `hermitcrabtea_succulent_picked` | 同上 | 理智 5、生命 +`TUNING.HEALING_MEDSMALL`（8）、`temperaturedelta = -40` 持续 120 秒；`-- buff` 被注释，**不挂 buff**。[定义](../../../raw/current/scripts/prefabs/hermitcrabtea_defs.lua#L22-L30) |
| `hermitcrabtea_firenettles` | 同上 | 理智 5、生命默认 1、`temperaturedelta = 40` 持续 120 秒；`-- buff` 被注释，**不挂 buff**。[定义](../../../raw/current/scripts/prefabs/hermitcrabtea_defs.lua#L32-L39) |
| `hermitcrabtea_tillweed` | 同上 | 理智 5、生命 +`TUNING.HEALING_MED`（20）；buff `hermitcrabtea_tillweed_buff`。[定义](../../../raw/current/scripts/prefabs/hermitcrabtea_defs.lua#L41-L46) |
| `hermitcrabtea_moon_tree_blossom` | 同上 | 理智 10、生命 1；buff `hermitcrabtea_moon_tree_blossom_buff`。[定义](../../../raw/current/scripts/prefabs/hermitcrabtea_defs.lua#L48-L52) |
| `hermitcrabtea_forgetmelots` | 同上 | 理智 +`TUNING.SANITY_MEDLARGE`（20）、生命 1；buff `hermitcrabtea_forgetmelots_buff`。[定义](../../../raw/current/scripts/prefabs/hermitcrabtea_defs.lua#L54-L58) |

八行共同构造（每瓶都跑这一段）：bank／build 都是 `hermitcrab_tea`、初始播放 `idle4`，再 `OverrideSymbol("tea_bottle", build, "tea_bottle_"..name)` 换瓶贴图；标签 `cattoy`／`fooddrink`／`pre-preparedfood` 在 `SetPristine` **之前**加，`MakeInventoryFloatable(inst, nil, .3)` 让瓶子浮在水面；非主模拟到此返回，主模拟才挂 `inspectable` → `inventoryitem` → `finiteuses` → `edible` → `tradable` → `MakeHauntableLaunchAndPerish`。[构造](../../../raw/current/scripts/prefabs/hermitcrabtea.lua#L92-L147)

瓶身**没有** `perishable`（`--inst:AddComponent("perishable")` 被注释）、没有 `stackable`、没有 `burnable`；`MakeHauntableLaunchAndPerish` 只加 `hauntable` 并抛飞，其中 "perish" 分支在 helper 里整段被注释，所以作祟不会让茶变坏。[注释](../../../raw/current/scripts/prefabs/hermitcrabtea.lua#L141-L144)、[helper](../../../raw/current/scripts/standardcomponents.lua#L1311-L1337)

同文件另注册 8 个 `hermitcrabtea_<name>_buff`（`MakeTeaBuff`，第二个循环同样遍历 `BUFF_DEFS`），它们是独立实体而不是茶的组件，逐条见[关联机制导航](#关联机制导航)。[buff 注册循环](../../../raw/current/scripts/prefabs/hermitcrabtea.lua#L251-L255)

### 每口结算

时间基准：`seg_time = 30` 秒、`total_day_time = 480` 秒，本页所有秒数都按 30 秒／段换算。[基准](../../../raw/current/scripts/tuning.lua#L19-L20)

| 茶 | 饥饿 | 生命 | 理智 | 温度 | 附带 buff 的结算 |
|---|---:|---:|---:|---|---|
| `petals` | 0 | 1 | +10 | — | 45 秒，每 1 秒 `sanity:DoDelta(1)`，≈+45 |
| `petals_evil` | 0 | 1 | −15 | — | 60 秒，每 1 秒 `DoDelta(-2)`，≈−120 |
| `foliage` | 0 | 1 | +5 | — | 180 秒，负光环乘数×0.1（只算本来源，见下） |
| `succulent_picked` | 0 | +8 | +5 | −40／120 秒 | 无 |
| `firenettles` | 0 | 1 | +5 | +40／120 秒 | 无 |
| `tillweed` | 0 | +20 | +5 | — | 15 秒，每 0.5 秒 `health:DoDelta(1)`，≈+30 |
| `moon_tree_blossom` | 0 | 1 | +10 | — | 180 秒，见下文暗影惊吓 |
| `forgetmelots` | 0 | 1 | +20 | — | 45 秒，每 1 秒 `DoDelta(2)`，≈+90 |

- 三维写入是 `edible.sanityvalue／healthvalue／hungervalue`，缺省值在工厂里定：`sanityvalue = data.sanityvalue or 0`、`healthvalue = data.healthvalue or TUNING.HEALING_TINY`（1）、`hungervalue = data.hungervalue or 0`，`foodtype = data.foodtype or FOODTYPE.GOODIES`。[赋值](../../../raw/current/scripts/prefabs/hermitcrabtea.lua#L71-L80)、[挂载](../../../raw/current/scripts/prefabs/hermitcrabtea.lua#L129-L139)
- 结算顺序：`Eater:Eat` 先按 `GetHealth／GetHunger／GetSanity` 乘 `stack_mult` 写入三维，随后才 `edible:OnEaten(eater)`，buff 在三维之后挂上；玩家 `eatwholestack` 默认 false，因此一口只乘 1。[三维](../../../raw/current/scripts/components/eater.lua#L204-L256)、[顺序](../../../raw/current/scripts/components/eater.lua#L275-L276)
- 温度走 `temperature:SetTemperatureInBelly(delta, duration)`：**覆盖**已有 `bellytemperaturedelta／bellytime` 并取消上一个到期任务，所以连喝两口是刷新时间窗而不是累加温差。[写入](../../../raw/current/scripts/components/edible.lua#L201-L208)、[覆盖](../../../raw/current/scripts/components/temperature.lua#L94-L101)
- 茶没有 `perishable`，而 `chill` 只在 `perishable` 的计时更新里累加：更新时对 `temperaturedelta > 0` 的食物调 `edible:AddChill`，`AddChill` 再按 `temperatureduration` 折成 0–1 的比例。茶不挂 `perishable`，这段永不执行，所以 `chill` 恒为 0，每口都按完整 ±40 结算。[累加点](../../../raw/current/scripts/components/perishable.lua#L139-L147)、[折算](../../../raw/current/scripts/components/edible.lua#L238-L242)

### 份数与退瓶

- 定义：`TUNING.HERMITCRABTEA_USES = 10`；结算：`SetMaxUses` 与 `SetUses` 都取这个值，于是每瓶 10 口、`percentusedchange` 的百分比以 10 为分母。[定义](../../../raw/current/scripts/tuning.lua#L9108-L9108)、[接线](../../../raw/current/scripts/prefabs/hermitcrabtea.lua#L123-L127)
- 一口：`HandleEdibleRemove` 在 `eatwholestack == false` 时调 `finiteuses:Use(1)`，即 `SetUses(current - 1)`，同时推 `percentusedchange` 并把动画切到 `idle1`–`idle4`（阈值 25%，`math.clamp(math.ceil(percent / 0.25), 1, 4)`）。[扣一口](../../../raw/current/scripts/prefabs/hermitcrabtea.lua#L37-L43)、[动画](../../../raw/current/scripts/prefabs/hermitcrabtea.lua#L49-L52)
- 喝空：`SetUses(0)` 在 `was_positive` 为真时加 `usesdepleted` 标签并调 `onfinished`；这就是 `OnFinishTea` 的唯一触发点。[触发](../../../raw/current/scripts/components/finiteuses.lua#L64-L79)
- 整叠吃（只有 `bearger`／`moose`／`mossling` 这类把 `eater.eatwholestack` 置 true 的生物会走）：`GetWholeStackEatMultiplier` 返回当前剩余口数，三维乘这份倍率后 `SetUses(0)` 直接退瓶。[倍率](../../../raw/current/scripts/prefabs/hermitcrabtea.lua#L45-L47)、[整叠 eater](../../../raw/current/scripts/components/eater.lua#L204-L204)
- 退瓶优先级（逐条）：① 先取 `inventoryitem:GetGrandOwner() or entity:GetParent()` 作为 `owner`，坐标取 `owner` 或瓶身；② 先 `inst:Remove()` 再生成 `messagebottleempty`；③ 若 `owner` 存在且其 `sg.statemem.feeder`（喂食者，由喂食状态写入）有效、背包 `isopen`、距离平方 < `TUNING.RETURN_ITEM_TO_FEEDER_RANGE`（4）的平方 → 塞给喂食者；④ 否则 `owner` 背包 `isopen` → 塞进持有者背包；⑤ 否则 `owner.components.container` → 塞进容器；⑥ 都不满足 → `inventoryitem:DoDropPhysics(x, y, z, true)`；⑦ `owner` 根本不存在 → 只 `SetPosition`。[退瓶主体](../../../raw/current/scripts/prefabs/hermitcrabtea.lua#L10-L35)、[4 格](../../../raw/current/scripts/tuning.lua#L9250-L9250)、[喂食者写入](../../../raw/current/scripts/stategraphs/SGwilson.lua#L6422-L6431)

### 获取链

- 配方是 `Recipe2` 的三层展开：`NUM_TEASHOP_LEVELS = 3`，常见花瓣 `{8, 6, 4}`、稀有花瓣 `{6, 4, 2}`，每条都以 `Ingredient("messagebottleempty", 1)` 加对应 `*_dried` 干花瓣，`TECH.LOST`、`product = 茶名`、`actionstr = "HERMITCRABSHOP"`、`hint_msg = "NEEDSHERMITCRAB_TEASHOP"`、`nounlock` 且 `manufactured = true`。[配方](../../../raw/current/scripts/recipes.lua#L1503-L1517)
- 稀有档（用 `NUM_RARE_PETALS_FOR_TEASHOP_LEVEL`）是 `succulent_picked`、`firenettles`、`tillweed`、`forgetmelots`；其余四种走常见档。[稀有](../../../raw/current/scripts/recipes.lua#L1512-L1516)、[常见](../../../raw/current/scripts/recipes.lua#L1509-L1511)
- 学哪一档由柜台算：珍珠装饰分 `score >= 75` → 3、`>= 50` → 2、否则 1，再 `LearnItem(v, v.."_"..level)`，所以同一瓶茶在三个等级下是三条不同 `"_1／_2／_3"` 配方，产物都是同一个 prefab。[柜台](../../../raw/current/scripts/prefabs/hermitcrab_teashop.lua#L74-L87)、[茶名单](../../../raw/current/scripts/prefabs/hermitcrab_teashop.lua#L14-L28)
- 干花瓣来自晾晒：例如花瓣的 `dryable:SetProduct("petals_dried")`，本页只用其名称，干燥机制与 8 件干叶归 [[reference/prefabs/dried-plants]]。[产出](../../../raw/current/scripts/prefabs/petals.lua#L127-L127)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`finiteuses` | `SetMaxUses／SetUses` 取 10，`SetOnFinished(OnFinishTea)`，监听 `percentusedchange` 切瓶身动画 | 仅主模拟；一口 `Use(1)`、整叠吃 `SetUses(0)` | [[reference/components/finiteuses]]；[接线](../../../raw/current/scripts/prefabs/hermitcrabtea.lua#L123-L127) |
| 组件：`edible` | 写三维、温度、`foodtype`；`SetOnEatenFn(OnEaten)`、`SetHandleRemoveFn(HandleEdibleRemove)`、`SetOverrideStackMultiplierFn` | 仅主模拟；每口结算一次 | [[reference/components/edible]]；[接线](../../../raw/current/scripts/prefabs/hermitcrabtea.lua#L129-L139) |
| 组件：`inventoryitem／inspectable／tradable` | 可拾取、可检视、可交易；`scrapbook_overridedata` 指向 `tea_bottle` 贴图 | 仅主模拟；没有 `stackable` 所以一瓶一格 | [[reference/components/inventoryitem]]；[接线](../../../raw/current/scripts/prefabs/hermitcrabtea.lua#L118-L121) |
| 组件：`hauntable` | `MakeHauntableLaunchAndPerish` 加组件并把瓶子抛出 | 主模拟；perish 分支在 helper 内被注释 | [[reference/components/hauntable]]；[helper](../../../raw/current/scripts/standardcomponents.lua#L1311-L1337) |
| 标签：`cattoy／fooddrink／pre-preparedfood` | 在 `SetPristine` 前加，两端都有 | `cattoy`：进浣猫 `PLAY_TAGS`，被选为玩耍目标后按 `CATCOON_PICKUP_ITEM_CHANCE`＝0.67 走叼走分支（瓶子没有 `bait` 组件，所以直接 `GiveItem` 塞进浣猫背包）；`fooddrink`：`ACTIONS.EAT.strfn` 返回 "DRINK"，同一个吃动作只换文案；`pre-preparedfood`：沃利的进食白名单 | [标签](../../../raw/current/scripts/prefabs/hermitcrabtea.lua#L106-L108)、[浣猫目标](../../../raw/current/scripts/brains/catcoonbrain.lua#L24-L24)、[玩耍与叼走](../../../raw/current/scripts/actions.lua#L3882-L3905)、[DRINK](../../../raw/current/scripts/actions.lua#L787-L793)、[沃利](../../../raw/current/scripts/prefabs/warly.lua#L35-L38)、[偏好](../../../raw/current/scripts/components/eater.lua#L310-L318)、[0.67](../../../raw/current/scripts/tuning.lua#L2985-L2985)、[GiveItem](../../../raw/current/scripts/actions.lua#L3896-L3899) |
| buff：`hermitcrabtea_petals_buff` | 45 秒；`onattachedfn` 起 `DoPeriodicTask(1, Petals_OnTick)`，每跳 +1 理智；目标死亡／变鬼立即 `Stop()` | 主模拟；由 `petals` 的 `buff` 字段挂上 | [定义](../../../raw/current/scripts/prefabs/hermitcrabtea_defs.lua#L133-L144)、[tick](../../../raw/current/scripts/prefabs/hermitcrabtea_defs.lua#L65-L71) |
| buff：`hermitcrabtea_petals_evil_buff` | 60 秒；每 1 秒 −2 理智，判定同上 | 主模拟；`petals_evil` 挂 | [定义](../../../raw/current/scripts/prefabs/hermitcrabtea_defs.lua#L147-L158) |
| buff：`hermitcrabtea_foliage_buff` | 180 秒；`sanity.neg_aura_modifiers:SetModifier(inst, 0.1)`，`ondetachedfn` 移除 | 主模拟；`foliage` 挂；字段来源是 buff 实体本身 | [定义](../../../raw/current/scripts/prefabs/hermitcrabtea_defs.lua#L161-L181) |
| buff：`hermitcrabtea_succulent_picked_buff` | 条目存在、三个回调全空、**没有** `duration` | 注册但没有任何茶引用；即便被挂也会在 `StartTimer` 处出错（见陷阱） | [定义](../../../raw/current/scripts/prefabs/hermitcrabtea_defs.lua#L184-L197) |
| buff：`hermitcrabtea_firenettles_buff` | 同 `succulent_picked`：无 `duration`、回调空 | 同上 | [定义](../../../raw/current/scripts/prefabs/hermitcrabtea_defs.lua#L200-L213) |
| buff：`hermitcrabtea_tillweed_buff` | 15 秒；每 0.5 秒 `health:DoDelta(1, nil, "hermitcrabtea_tillweed")`，≈+30 | 主模拟；`tillweed` 挂 | [定义](../../../raw/current/scripts/prefabs/hermitcrabtea_defs.lua#L216-L227)、[tick](../../../raw/current/scripts/prefabs/hermitcrabtea_defs.lua#L85-L91) |
| buff：`hermitcrabtea_moon_tree_blossom_buff` | 180 秒；监听 `attacked`，命中带 `shadowsubmissive` 的攻击者时放 `hitsparks_fx`（颜色 0,0,1）并给它留 30 秒 `_shadow_creature_panic_task`；同时被 `IsEntityGestaltProtected` 当作"对盖瑟特免疫" | 主模拟；`moon_tree_blossom` 挂 | [定义](../../../raw/current/scripts/prefabs/hermitcrabtea_defs.lua#L230-L240)、[惊吓](../../../raw/current/scripts/prefabs/hermitcrabtea_defs.lua#L106-L128)、[盖瑟特](../../../raw/current/scripts/componentutil.lua#L1996-L2000)、[读取](../../../raw/current/scripts/prefabs/gestalt.lua#L79-L81) |
| buff：`hermitcrabtea_forgetmelots_buff` | 45 秒；每 1 秒 +2 理智，≈+90 | 主模拟；`forgetmelots` 挂 | [定义](../../../raw/current/scripts/prefabs/hermitcrabtea_defs.lua#L243-L254) |
| 组件（buff 侧）：`debuff／timer` | `SetAttachedFn／SetDetachedFn／SetExtendedFn` 三回调；`timer:StartTimer("buffover", duration)`；`timerdone` → `debuff:Stop()` | 仅主模拟；`keepondespawn = true` | [[reference/components/debuff]]、[[reference/components/timer]]；[接线](../../../raw/current/scripts/prefabs/hermitcrabtea.lua#L206-L236) |
| 消费者：`recipes.lua` | 3 档 × 8 茶 = 24 条 `Recipe2`，`product` 就是茶名 | `manufactured` 门禁：必须有 `current_prototyper`，且 `CanPrototypeRecipe(trees)` 或 `craftingstation:KnowsRecipe(配方名)` 二者之一；柜台 prototyper 的 `trees` 是空 `TechTree.Create()`（全 0），`TECH.LOST`＝`{MAGIC=10, SCIENCE=10, ANCIENT=10}` 满足不了，实际只能靠 `LearnItem` 学到当前等级的配方名 | [配方](../../../raw/current/scripts/recipes.lua#L1503-L1517)、[门禁](../../../raw/current/scripts/components/builder.lua#L670-L678)、[判定](../../../raw/current/scripts/widgets/widgetutil.lua#L16-L24)、[零表](../../../raw/current/scripts/techtree.lua#L73-L79)、[LOST](../../../raw/current/scripts/constants.lua#L1324-L1324) |
| 消费者：`hermitcrab_teashop` | `TEA_RECIPES` 列 8 个茶名并塞进 `prefabs` 依赖；按装饰分学对应等级 | 服务端、柜台有珍珠隐士时 | [列表](../../../raw/current/scripts/prefabs/hermitcrab_teashop.lua#L14-L28) |
| 消费者：`braincommon` | 影子生物的行为树读攻击者身上的 `_shadow_creature_panic_task` 决定是否 `PanicTriggerShadowCreature` | 三种 shadow creature brain | [节点](../../../raw/current/scripts/brains/braincommon.lua#L115-L122) |
| 关联 Prefab：`messagebottleempty` | 既是配方配料也是喝空后的返还物，写进 `prefabs` 依赖 | 主模拟，`OnFinishTea` 内生成 | [依赖](../../../raw/current/scripts/prefabs/hermitcrabtea.lua#L66-L69)、[生成](../../../raw/current/scripts/prefabs/hermitcrabtea.lua#L16-L16) |
| 关联 Prefab：`hitsparks_fx` | 月树花茶命中暗影生物时生成，`Setup` 传蓝紫色 | 主模拟 | [生成](../../../raw/current/scripts/prefabs/hermitcrabtea_defs.lua#L105-L109) |
| 资源：`anim/hermitcrab_tea.zip` | bank／build 均为 `hermitcrab_tea`，8 个瓶贴图靠 `tea_bottle_<name>` 符号覆盖 | 原版资源，不随页分发 | [声明](../../../raw/current/scripts/prefabs/hermitcrabtea.lua#L60-L64)、[覆盖](../../../raw/current/scripts/prefabs/hermitcrabtea.lua#L101-L104) |

### buff 实体的形态

- 新建成独立实体：`MakeTeaBuff` 返回 `Prefab(name.."_buff", debuff_fn)`，`AddDebuff` 走到 `debuffable:AddDebuff` 时 `SpawnPrefab` 出这个实体，再 `SetParent(target.entity)` 挂到宿主身上。[工厂](../../../raw/current/scripts/prefabs/hermitcrabtea.lua#L162-L164)、[生成](../../../raw/current/scripts/components/debuffable.lua#L95-L101)
- 不联网也不落盘：`debuff_fn` 只 `AddTransform`（没有 `AddNetwork`）、`entity:Hide()`、`persists = false`、加 `CLASSIFIED` 标签；非主模拟分支直接 `DoTaskInTime(0, inst.Remove)` 后返回，客户端不会留下实体。[客户端分支](../../../raw/current/scripts/prefabs/hermitcrabtea.lua#L206-L223)
- 存档靠宿主：注册时 `debuffable` 把 `ent.persists = false`，由宿主 `debuffable:OnSave` 用 `GetSaveRecord` 序列化，`OnLoad` 再 `SpawnSaveRecord` 回来。[注册](../../../raw/current/scripts/components/debuffable.lua#L72-L93)、[存读](../../../raw/current/scripts/components/debuffable.lua#L126-L150)
- 死亡即断：`OnAttached` 监听宿主 `"death"` 并 `debuff:Stop()`；玩家下线时 `debuffable:RemoveOnDespawn`（`player_common` 调用）会跳过 `keepondespawn` 的条目，所以这 8 个 buff 在掉线／重连窗口里被保留。[断线](../../../raw/current/scripts/prefabs/hermitcrabtea.lua#L180-L182)、[保活](../../../raw/current/scripts/components/debuffable.lua#L39-L49)、[调用点](../../../raw/current/scripts/prefabs/player_common.lua#L1434-L1434)

## 典型接入

### 改一份茶的份数

- 前提与选择：服务端 `modmain.lua` 的 postinit。茶瓶在 `finiteuses` 里存份数，改组件实例即可，不必动 `TUNING.HERMITCRABTEA_USES`（后者会连带八瓶茶与所有引用它的模组）。
- 生效与撤销：`SetMaxUses` 只改分母，必须紧跟 `SetUses` 才让新生成／已存在的瓶身立刻变成新份数；`SetUses` 会推 `percentusedchange`，动画随之更新。读档要看瓶子的存档状态：① 本进程新生成的瓶 → 立刻是 3/3；② 存档里是满瓶（`OnSave` 只在 `current ~= total` 时写 `uses`，满瓶不写）→ 不覆盖，仍是 3/3；③ 存档里不满的瓶 → 读档时 `SpawnSaveRecord` 先 `SpawnPrefab`（postinit 在这一步跑）再 `SetPersistData` 触发 `finiteuses:OnLoad` 覆盖成旧绝对值，可能读成 `9/3` 这类当前值大于上限的错位，**postinit 里同步改修不好**，重设要放到读档之后（例如 postinit 里 `inst:DoTaskInTime(0, …)`）。[读档顺序](../../../raw/current/scripts/mainfunctions.lua#L464-L466)、[落档](../../../raw/current/scripts/mainfunctions.lua#L503-L503)、[存档](../../../raw/current/scripts/components/finiteuses.lua#L48-L58)
- 原版依据：接线在 `MakeTea` 的 `SetMaxUses／SetUses`；`OnFinishTea` 由 `SetUses(0)` 的 `onfinished` 触发，改份数不影响退瓶。[接线](../../../raw/current/scripts/prefabs/hermitcrabtea.lua#L123-L127)、[完成回调](../../../raw/current/scripts/components/finiteuses.lua#L64-L79)

```lua
-- modmain.lua：把花瓣茶从 10 口改成 3 口（仅服务端）
AddPrefabPostInit("hermitcrabtea_petals", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    local uses = inst.components.finiteuses
    if uses ~= nil then
        uses:SetMaxUses(3)
        uses:SetUses(3)
    end
end)
```

- 调整与验证：替换 `hermitcrabtea_petals` 为其它 7 个注册名即可逐瓶改；八瓶的 `buff` 与三维不受影响。验证看喝空后是否仍退空瓶、以及 `idle` 帧是否随剩余份数变化（阈值 25%）。

## 常见陷阱与限制

- **误区：多肉茶与火荨麻茶各有一个 buff。** 这两条的 `buff` 行在 `TEA_DEFS` 里被注释掉了，而 `OnEaten` 只在 `data.buff ~= nil` 时调 `AddDebuff`，所以它们只有三维与温差，没有 buff；`BUFF_DEFS` 里同名的两个条目仍然注册，但没有任何茶引用。[注释](../../../raw/current/scripts/prefabs/hermitcrabtea_defs.lua#L22-L39)、[条件](../../../raw/current/scripts/prefabs/hermitcrabtea.lua#L82-L86)
- **边界：那两个 `_buff` 一旦真的被挂上会直接报错。** 它们没有 `duration`，`MakeTeaBuff` 里 `duration` 为 nil，`debuff_fn` 调 `timer:StartTimer("buffover", nil)`；`Timer:StartTimer` 把时间原样交给 `DoTaskInTime`，再进 `Scheduler:GetListForTimeFromNow` 时对 nil 做加法。源码推导为"构造期抛 `attempt to perform arithmetic on a nil value`"，**未运行验证**。[缺 duration](../../../raw/current/scripts/prefabs/hermitcrabtea_defs.lua#L184-L197)、[StartTimer](../../../raw/current/scripts/components/timer.lua#L36-L54)、[算术](../../../raw/current/scripts/scheduler.lua#L280-L293)
- **误区：喝第二口会让 buff 数值翻倍。** `Debuffable:AddDebuff` 只有在**同一目标**身上已挂着同名条目时才调 `Extend`，`OnExtended` 里都是 `StopTimer` → `StartTimer` 重启计时；周期任务是否 `Cancel` 后重建取决于茶种——带 `onextendedfn` 的花瓣／深色花瓣／犁地草／必忘我会重建，蕨叶只重设 modifier，月树花茶没有 `onextendedfn`，扩展时只重启计时；数值、倍率与 modifier 都不会叠。[Extend](../../../raw/current/scripts/components/debuffable.lua#L104-L106)、[OnExtended](../../../raw/current/scripts/prefabs/hermitcrabtea.lua#L189-L196)
- **边界：蕨叶茶是把负光环"打折"而不是免疫。** `neg_aura_modifiers` 的初值是 1、合成方式是乘法，这 0.1 只是本来源的乘数，会与角色的 `neg_aura_mult` 及其它来源连乘（温蒂 `neg_aura_mult`＝0.75，喝下时是 0.75×0.1＝0.075），只作用于 `aura_val < 0` 的分支；正光环与点数不变。[乘法表](../../../raw/current/scripts/util/sourcemodifierlist.lua#L47-L58)、[取用](../../../raw/current/scripts/components/sanity.lua#L484-L486)、[只降负值](../../../raw/current/scripts/components/sanity.lua#L567-L571)、[温蒂](../../../raw/current/scripts/prefabs/wendy.lua#L369-L369)、[0.75](../../../raw/current/scripts/tuning.lua#L2713-L2713)
- **边界：月树花茶的"吓退"记在攻击者身上，时限也不是 buff 时长。** panic task 用的是 `TUNING.HERMITCRAB_MOONTREEBLOSSOMTEA_PANIC_SHADOWCREATURE_TIME = seg_time`（30 秒），与 buff 的 180 秒是两套时间；触发还要求攻击者带 `shadowsubmissive` 且通过 `ShouldProcOnAttackedOrBlocked`（不是重定向伤害、不是被自己装备打的）。[常量](../../../raw/current/scripts/tuning.lua#L9154-L9155)、[设定](../../../raw/current/scripts/prefabs/hermitcrabtea_defs.lua#L113-L128)、[门禁](../../../raw/current/scripts/componentutil.lua#L114-L122)
- **边界：茶不会腐烂，也不会被烧掉。** `perishable` 被注释、没有 `burnable` 组件与标签，所以既没有"几天后坏"的计时，也不会在火里变样；瓶子只能喝、交易或被作祟抛飞。[组件表](../../../raw/current/scripts/prefabs/hermitcrabtea.lua#L141-L144)
- **边界：深色花瓣茶喂不给其他玩家。** `sanityvalue` 为负，`edible` 在赋值时就被 `oncheckbadfood` 打上 `badfood`；FEEDPLAYER 候选在非 PVP 下对带 `badfood`／`unsafefood` 的物品不出现，所以非 PVP 的普通活玩家收不到这瓶；`possessedbody` 目标另有旁路（在同一个 `edible` 候选里无条件下发 FEEDPLAYER，但仍要先过前面的食性标签匹配）。[badfood](../../../raw/current/scripts/components/edible.lua#L1-L7)、[赋值钩子](../../../raw/current/scripts/components/edible.lua#L50-L51)、[门禁](../../../raw/current/scripts/componentactions.lua#L1164-L1171)
- **未核验**：未运行游戏。空瓶返还的五个落点（喂食者、持有者背包、容器、物理掉落、无 owner 定坐标）、`idle1`–`idle4` 的实际观感、以及"客户端不建 buff 实体"都按源码推导；珍珠分档 75／50 的实际阈值表现也未实测。

## 验证记录

- 修订记录（外部第四轮）：快速判断的重叠 buff 表述改为「重置计时；周期任务是否重建按茶种区分」并指向陷阱节；其余段落未动。
- 修订记录（外部第三轮）：L139 的 `Extend` 表述补全为「是否重建周期任务按茶种区分」；其余段落未动。
- 修订记录（外部第二轮）：P2-6 读档顺序改正（postinit 早于 `finiteuses:OnLoad`，重设须放到读档后）、干叶改为链接 `reference/prefabs/dried-plants`、蕨叶茶表格补「只算本来源」；其余段落未动。
- 修订记录（外部复检）：接外部复检报告后按点名条目修订 P2-5（禁喂收窄到非 PVP 普通活玩家＋`possessedbody` 旁路）、P2-6（三态存档分写）、P2-7（0.1 只为本来源乘数，补温蒂 0.75×0.1）、P2-8（Extend 需同一目标）、P3-9（三处茶名统一为月树花茶），其余段落未动。
- 修订记录：2026-11-09 第一轮独立检视后按点名条目修订（T-1 标签消费端、T-2 badfood 喂食门禁、T-3 manufactured 口径、T-4 三处锚点起点、T-5 去掉 nochill、T-6 退瓶落点口径、T-7 Extend 例外），其余段落未动。
- 源码审阅：2026-11-09，Steam buildid 26474873。通读 `prefabs/hermitcrabtea.lua`（`MakeTea`、`OnFinishTea`、`HandleEdibleRemove`、`GetWholeStackEatMultiplier`、`OnPercentChanged`、`MakeTeaBuff` 与两个注册循环）与 `prefabs/hermitcrabtea_defs.lua`（`TEA_DEFS`、`BUFF_DEFS` 与八个 tick 函数）；`components` 的 `finiteuses`／`edible`／`eater`／`debuff`／`debuffable`／`timer`／`sanity`／`temperature`、`util/sourcemodifierlist`、`scheduler`、`entityscript` 的 `AddDebuff`；`componentutil` 的 `IsEntityGestaltProtected` 与 `ShouldProcOnAttackedOrBlocked`、`brains/braincommon` 的 panic 节点、`prefabs/gestalt` 的读取、`prefabs/player_common` 的 `RemoveOnDespawn`；`recipes.lua` 的 24 条茶配方、`prefabs/hermitcrab_teashop.lua` 的茶名单与分档、`prefabs/petals.lua` 的干花瓣产物；`tuning.lua` 的 `HERMITCRABTEA_*`／`HERMITCRAB_*TEA_*` 与 `RETURN_ITEM_TO_FEEDER_RANGE`；`languages/chinese_s.po` 的八个 `NAMES`。
- 名称核验：八个显示名都取 `STRINGS.NAMES.HERMITCRABTEA_<NAME>`（大写、下划线），与注册名一一对应：花瓣茶／深色花瓣茶／蕨叶茶／多肉茶／火荨麻茶／犁地草茶／月树花茶／必忘我茶。[花瓣茶](../../../raw/current/scripts/languages/chinese_s.po#L343804-L343807)、[火荨麻茶](../../../raw/current/scripts/languages/chinese_s.po#L343784-L343787)
- 未覆盖：茶柜台的酿造动画与 `SGhermitcrab` 的交付流程、干燥机制本身、`scrapbookdata` 的采集册条目、以及其它角色对 `fooddrink`／`pre-preparedfood` 标签的读取。
- 游戏运行：未运行。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 见交付回报。

## 相关链接

- 份数与食用：[[reference/components/finiteuses]]、[[reference/components/edible]]、[[reference/components/inventoryitem]]
- buff 载体：[[reference/components/debuff]]、[[reference/components/debuffable]]、[[reference/components/timer]]
- 受影响的理智与温度：[[reference/components/sanity]]、[[reference/components/temperature]]
- 同批冬宴食物：[[reference/prefabs/wintersfeast-foods]]

返回 [[reference/prefabs/special-food-index|节庆、角色与特殊食品索引]]。
