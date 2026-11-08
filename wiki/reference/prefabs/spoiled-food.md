---
title: "腐烂物与变质的鱼（spoiled_food、spoiled_fish、spoiled_fish_small）"
created: "2026-11-09"
updated: "2026-11-09"
type: prefab
summary: "prefabs/spoiledfood.lua 的三个同厂注册：腐烂物的肥料数值、海钓鱼饵与雨曝消失，变质的鱼／小鱼块的锤击拆分掉落，以及三者共用的燃料、灼烧传播、40 堆叠、负饥饿与负生命（理智 0）的可食。"
aliases: ["spoiled_food", "spoiled_fish", "spoiled_fish_small", "腐烂物", "变质的鱼", "变质的小鱼块", "腐烂食物", "海钓腐烂物鱼饵"]
tags: [prefab, item, food]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/spoiledfood.lua", "prefabs/fertilizer_nutrient_defs.lua", "prefabs/hammer.lua", "tuning.lua", "components/fertilizer.lua", "components/fertilizable.lua", "components/fertilizerresearchable.lua", "components/workable.lua", "components/stackable.lua", "components/lootdropper.lua", "components/disappears.lua", "components/rainimmunity.lua", "components/edible.lua", "components/eater.lua", "components/inspectable.lua", "components/selfstacker.lua", "components/smotherer.lua", "components/driedsalticon.lua", "components/fuel.lua", "components/fueled.lua", "components/tradable.lua", "components/tool.lua", "standardcomponents.lua", "containers.lua", "componentactions.lua", "actions.lua", "entityscript.lua", "stringutil.lua", "languages/chinese_s.po", "screens/redux/scrapbookdata.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 腐烂物与变质的鱼（spoiled_food、spoiled_fish、spoiled_fish_small）

## 快速判断

- 职责与适用：三个注册同出 `fn(common_init, mastersim_init, nutrients, kind)`，只有 `common_init`／`mastersim_init`、nutrient 表与第 4 参 `kind` 按注册不同（`spoiled_food` 传 `food`、两条鱼传 `nil`）；三者都是可堆叠 40 的可食物品，同时是肥料、燃料与可灼烧传播物。[工厂](../../../raw/current/scripts/prefabs/spoiledfood.lua#L73-L159)、[三处注册](../../../raw/current/scripts/prefabs/spoiledfood.lua#L263-L265)
- 优先使用：要改"施肥效果"看[肥料与部署](#肥料与部署)；要改"锤鱼取骨片"看[锤击拆分](#锤击拆分)；要改"雨中消失"看[雨曝与免疫](#雨曝与免疫)。
- 重要副作用：**只有 `spoiled_food`** 会在落地且下雨时被 `disappears` 处理：计时 25–35 秒，到期前被取消（入包、雨停、拿到雨免疫）就**不保留剩余时间**，再暴露会重抽一整个延迟，只有清醒实体才播溶解动画；它还是 `oceanfishingtackle` 的一次性鱼饵。两种鱼没有雨曝也没有鱼饵，只有锤击拆分。[雨曝接线](../../../raw/current/scripts/prefabs/spoiledfood.lua#L196-L212)、[鱼饵](../../../raw/current/scripts/prefabs/spoiledfood.lua#L197-L198)
- 修改边界：腐坏的**来源**是各食物自己的 `perishable.onperishreplacement`，不归本页；`boneshard`、`gridplacer_farmablesoil` 只作为产物与部署预览出现，本页不展开其本体行为。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `spoiled_food` | `fn(food_init, food_mastersim_init, FERTILIZER_DEFS.spoiled_food.nutrients, "food")`：共同 pristine → 主模拟枢纽 → 专属初始化 | 唯一传 `kind = "food"`（因此加 `SoundEmitter`）；`food_init` 加 `oceanfishing_lure` 标签与 `driedsalticon`；主模拟另加 `oceanfishingtackle`／`disappears` 与雨曝监听。nutrients = `{0, 8, 0}`。[注册](../../../raw/current/scripts/prefabs/spoiledfood.lua#L263-L263)、[food_init](../../../raw/current/scripts/prefabs/spoiledfood.lua#L161-L166)、[养分表](../../../raw/current/scripts/tuning.lua#L5862-L5862) |
| `spoiled_fish` | `fn(fish_init, fish_mastersim_init, FERTILIZER_DEFS.spoiled_fish.nutrients, nil)` | 换 bank／build 为 `spoiled_fish`、加 `spoiled_fish` 标签、floater 缩放 0.6、实体缩放 1.3 倍；主模拟加 `lootdropper`（`fish_loot`、`numrandomloot = 1`）与 `workable`（HAMMER）。nutrients = `{16, 0, 0}`。[fish_init](../../../raw/current/scripts/prefabs/spoiledfood.lua#L214-L222)、[fish_mastersim_init](../../../raw/current/scripts/prefabs/spoiledfood.lua#L224-L235) |
| `spoiled_fish_small` | `fn(fish_small_init, fish_small_mastersim_init, FERTILIZER_DEFS.spoiled_fish_small.nutrients, nil)` | 与上者同构，但 bank／build 为 `spoiled_fish_small`、floater 0.35、`inspectable.nameoverride = "spoiled_fish"`、workleft 用 `SPOILED_FISH_SMALL_WORK_REQUIRED`；掉落表用 `AddRandomLoot` 逐项 1 权重（与 `SetLoot` 那条固定表**不等价**，见[锤击拆分](#锤击拆分)）。nutrients = `{8, 0, 0}`。[fish_small_init](../../../raw/current/scripts/prefabs/spoiledfood.lua#L237-L245)、[fish_small_mastersim_init](../../../raw/current/scripts/prefabs/spoiledfood.lua#L247-L261) |

共用 pristine（两端）：bank `spoiled`／build `spoiled_food`／播 `idle`，标签 `icebox_valid`／`saltbox_valid`／`show_spoiled`，`MakeInventoryFloatable`，`MakeDeployableFertilizerPristine`（加 `deployable`／`tile_deploy` 并指定 `gridplacer_farmablesoil` 预览），另有 `fertilizerresearchable`、`selfstacker`、`spoiledfood` 三个标签，随后 `SetPristine`。[共用段](../../../raw/current/scripts/prefabs/spoiledfood.lua#L85-L109)、[部署预览名](../../../raw/current/scripts/standardcomponents.lua#L1816-L1821)

共用主模拟（仅服务端）：`fertilizer`（75／2／0.5 + nutrients）→ `smotherer` → `inspectable`（`getstatus = GetStatus`）→ `inventoryitem` → `stackable`（40）→ `fertilizerresearchable` → `selfstacker` → `fuel`（`SMALL_FUEL`＝15 秒）+ `MakeSmallBurnable`（`SMALL_BURNTIME`＝7.5 秒）+ `MakeSmallPropagator` → `edible`（强制 spoiled、默认 -1／-10／0）→ `tradable`，最后 `MakeDeployableFertilizer` 与 `MakeHauntableLaunchAndIgnite`。[主模拟](../../../raw/current/scripts/prefabs/spoiledfood.lua#L115-L156)、[时长常量](../../../raw/current/scripts/tuning.lua#L1674-L1681)

### 名称核验

简中实显：`spoiled_food`＝**腐烂物**（Rot）、`spoiled_fish`＝**变质的鱼**（Spoiled Fish）、`spoiled_fish_small`＝**变质的小鱼块**（Spoiled Fish Morsel），三个注册名都有同名 `STRINGS.NAMES` 键。[腐烂物](../../../raw/current/scripts/languages/chinese_s.po#L349919-L349922)、[变质的鱼](../../../raw/current/scripts/languages/chinese_s.po#L349909-L349912)、[小鱼块](../../../raw/current/scripts/languages/chinese_s.po#L349914-L349917)

小鱼块只写了 `components.inspectable.nameoverride`，按组件注释它只替换**检查语句**的键（"this is actually the inspect string name, not the display name of the object"），显示名仍走 `inst.nameoverride`／prefab，所以物品名仍是"变质的小鱼块"，只是台词借用 `SPOILED_FISH` 的描述。[注释](../../../raw/current/scripts/components/inspectable.lua#L5-L7)、[语句取键](../../../raw/current/scripts/stringutil.lua#L374-L376)、[显示名](../../../raw/current/scripts/entityscript.lua#L687-L692)

## 肥料与部署

- 数值来源：`SPOILEDFOOD_FERTILIZE = day_time/4`＝300/4＝**75**，`SOILCYCLES = 2`，`WITHEREDCYCLES = 0.5`；`day_time = seg_time * 10`、`seg_time = 30`，故基准 `total_day_time`＝480 秒。[定义](../../../raw/current/scripts/tuning.lua#L1653-L1655)、[时间基准](../../../raw/current/scripts/tuning.lua#L19-L27)
- 两个入口各自结算：**部署**到可耕作土时 `fertilizer_ondeploy` 取 `inst.components.fertilizer.nutrients` 调 `AddTileNutrients`，再 `OnApplied` 扣掉一件；**施肥动作**则按目标分派到 `crop:Fertilize`（growthpercent += fertilizervalue × rate）、`grower:Fertilize`（cycles_left += soil_cycles）、`pickable:Fertilize`（protected_cycles += withered_cycles）。[部署结算](../../../raw/current/scripts/standardcomponents.lua#L1797-L1802)、[动作分派](../../../raw/current/scripts/actions.lua#L1812-L1838)、[crop](../../../raw/current/scripts/components/crop.lua#L96-L96)、[grower](../../../raw/current/scripts/components/grower.lua#L77-L77)、[pickable](../../../raw/current/scripts/components/pickable.lua#L264-L277)
- 两条结算各带一道额外门禁：`crop:Fertilize` 的整体逻辑包在 `not (TheWorld.state.iswinter and TheWorld.state.temperature <= 0)` 里，**冬季且气温 ≤0°C 时作物完全不长**（连音效都不播）；`pickable` 的 `protected_cycles` 累加写在 `if self.inst.components.witherable ~= nil` 之内，**没有 `witherable` 的可采物只重置轮数、不累加保护轮**。[冬季门禁](../../../raw/current/scripts/components/crop.lua#L89-L96)、[witherable 门禁](../../../raw/current/scripts/components/pickable.lua#L274-L277)
- FERTILIZE 候选要求目标带 `notreadyforharvest`（未熟作物，**且不带 `withered`**）／`fertile`／`infertile`／`barren`／`fertilizable` 之一，骑乘时不出；另有 `self_fertilizable` 自身分支（目标为空或就是自己、物品带 `fertilizer` 标签、执行者带 `self_fertilizable` 且 `health:CanHeal()`；本版只有沃姆伍德带该标签）。部署候选要求落点是可耕作土，且还要过 `Deployable:IsDeployable` 的两道门：骑乘时只允许带 `complexprojectile` 的可投掷物、手上有浮具时只允许 `boatbuilder` 的可部署物，腐烂物两者都不满足。施肥动作也是**成功才扣件**——`ACTIONS.FERTILIZE.fn` 只在 `applied` 为真时才调 `OnApplied`，再由它 **从堆叠里扣 1 个**（`stackable:Get():Remove()`）；`keep_in_inventory_on_deploy` 只保证部署动作不预先移除物品。[候选](../../../raw/current/scripts/componentactions.lua#L1241-L1254)、[自身分支](../../../raw/current/scripts/componentactions.lua#L1248-L1252)、[自身结算](../../../raw/current/scripts/actions.lua#L1829-L1834)、[部署门禁](../../../raw/current/scripts/standardcomponents.lua#L1812-L1831)、[骑乘／浮具](../../../raw/current/scripts/components/deployable.lua#L89-L102)、[成功才扣](../../../raw/current/scripts/actions.lua#L1836-L1838)、[扣件](../../../raw/current/scripts/components/fertilizer.lua#L30-L48)
- 三种养分互不相同：`spoiled_food` 补第 2 项 8，`spoiled_fish` 补第 1 项 16，`spoiled_fish_small` 补第 1 项 8；`fertilizer:SetNutrients` 直接持有该表引用。[表](../../../raw/current/scripts/tuning.lua#L5862-L5867)、[赋值](../../../raw/current/scripts/components/fertilizer.lua#L22-L28)
- `fertilizerresearchable` 的研究键取 `inst.prefab`，玩家学习后进植物登记册，三种腐烂物因此各自可被"研究"。[研究键](../../../raw/current/scripts/prefabs/spoiledfood.lua#L60-L66)、[学习](../../../raw/current/scripts/components/fertilizerresearchable.lua#L17-L22)

## 锤击拆分

- 鱼的两件都挂 `workable`：`SetWorkAction(ACTIONS.HAMMER)`、`SetWorkLeft(stacksize × SPOILED_FISH_WORK_REQUIRED)`、回调 `fish_onhit`；`SetWorkLeft` 会把 `workable` 置回 true 并触发 `HAMMER_workable` 标签，锤击候选只认这个标签。[挂件](../../../raw/current/scripts/prefabs/spoiledfood.lua#L229-L234)、[标签维护](../../../raw/current/scripts/components/workable.lua#L1-L14)、[候选校验](../../../raw/current/scripts/componentactions.lua#L3125-L3128)
- `fish_onhit` 的算式：`num_loots = floor(clamp(workdone / SPOILED_FISH_WORK_REQUIRED, 1, SPOILED_FISH_LOOT.WORK_MAX_SPAWNS))`，再与当前堆叠取 min；随后按 `num_loots` 次 `lootdropper:DropLoot()`，最后 `stackable:Get(num_loots):Remove()`。[结算](../../../raw/current/scripts/prefabs/spoiledfood.lua#L33-L52)
- 每次锤击的 `workdone` 由工具有效性给出：`tool:GetEffectiveness` 读 `SetAction` 登记的权重（不传则 1），再乘 `workmultiplier`；普通锤子不传权重、也没有工具效率或 `workmultiplier` 修正时，一次 1 点。`SPOILED_FISH_WORK_REQUIRED = 1`、`WORK_MAX_SPAWNS = 10`，因此**一锤 = 结算 1 次掉落 + 堆叠减 1 条**；每次结算掉 2 件（**仅 `spoiled_fish`**；`spoiled_fish_small` 每次只掉 1 件，见下一条），workdone ≥ 10 的效率来源才会一锤结算满 10 次。[权重](../../../raw/current/scripts/components/tool.lua#L37-L45)、[锤子登记](../../../raw/current/scripts/prefabs/hammer.lua#L95-L96)、[工作量来源](../../../raw/current/scripts/actions.lua#L1703-L1717)、[常量](../../../raw/current/scripts/tuning.lua#L4129-L4138)
- 掉落表 `fish_loot` 走的是 `SetLoot`：它把两项写进 `self.loot`，同时清空 `randomloot` 与 `numrandomloot`；prefab 随后又直接赋 `numrandomloot = 1`，但 `PickRandomLoot` 读的 `GetRandomLootTable()` 仍是 nil，随机分支什么也不加，`GenerateLoot` 末尾再把 `self.loot` 两项**全部**插入。**所以 `spoiled_fish` 每次 `DropLoot` 得到 1 份腐烂物 + 1 份骨片**，一叠 N 条大鱼锤完共 2N 件。相邻的 `spoiled_fish_small` 走另一条路：`AddRandomLoot` 两条各 1 权重再赋 `numrandomloot = 1`，`PickRandomLoot` 从两项里抽 1 件，**每次只掉 1 件**，一叠 N 条小鱼块锤完是 N 件。[小鱼掉落表](../../../raw/current/scripts/prefabs/spoiledfood.lua#L250-L253)、[固定表](../../../raw/current/scripts/components/lootdropper.lua#L30-L35)、[再赋 numrandomloot](../../../raw/current/scripts/prefabs/spoiledfood.lua#L225-L227)、[随机表为空](../../../raw/current/scripts/components/lootdropper.lua#L87-L112)、[整表插入](../../../raw/current/scripts/components/lootdropper.lua#L236-L240)
- 堆叠变化会重算工作量：`stacksizechange` 事件把 `workleft` 重设为 `stacksize × WORK_REQUIRED`，合并、拆堆后不必重新生成实体。[监听](../../../raw/current/scripts/prefabs/spoiledfood.lua#L54-L58)、[事件源](../../../raw/current/scripts/components/stackable.lua#L103-L107)
- `stackable:Get(n)` 在剩余数 > n 时**新建一个同 prefab 实体**再删掉它，只剩 n 件时直接返回自身并被 `Remove()`：所以锤最后一条鱼是在工作回调里把实体自己删掉。[分堆](../../../raw/current/scripts/components/stackable.lua#L109-L153)

## 雨曝与免疫

- 范围：整段只接在 `spoiled_food` 上——`fn` 的 `mastersim_init` 里只有 `food_mastersim_init` 加 `disappears` 与雨监听，两个鱼的 init 都不加。
- 判定：`food_IsExposedToRain` 要求"正在下雨 且 没有 `rainimmunity` 组件 且 未被持有"；成立时 `PrepareDisappear`，否则 `StopDisappear`。[判定](../../../raw/current/scripts/prefabs/spoiledfood.lua#L170-L184)
- 计时：`disappears.delay` 默认 25，`PrepareDisappear` 再加 `math.random() * 10`，即 25–35 秒后收尾。期间若被 `StopDisappear` 中断（入包、雨停、拿到雨免疫），任务取消**且不保留剩余时间**——再次暴露时 `PrepareDisappear` 会重抽一整个 25–35 秒；收尾分两支：实体处于睡眠（`IsAsleep`）时直接 `Remove`，清醒时先置 `persists = false`、加 `NOCLICK`、`canbepickedup = false`，再播 `dissolve`（本 prefab 设的 `anim`）并靠 `animover`／计时器移除。[延迟](../../../raw/current/scripts/components/disappears.lua#L68-L77)、[消失流程](../../../raw/current/scripts/components/disappears.lua#L12-L49)、[中断不保值](../../../raw/current/scripts/components/disappears.lua#L51-L61)、[重抽延迟](../../../raw/current/scripts/components/disappears.lua#L68-L77)、[音画设置](../../../raw/current/scripts/prefabs/spoiledfood.lua#L200-L202)
- 监听：`WatchWorldState("israining")` 处理下雨状态切换；到期前**入包或获得雨免疫都会取消任务**（`gainrainimmunity` 走 `StopDisappear`、`SetOnPutInInventoryFn` 在入包瞬间也取消），**不保留剩余时间**；`loserainimmunity` 与 `ondropped` 会重新判定，再次暴露时重抽完整的 25–35 秒（`delay + math.random() * 10`）。[接线](../../../raw/current/scripts/prefabs/spoiledfood.lua#L204-L211)、[入包回调](../../../raw/current/scripts/prefabs/spoiledfood.lua#L208-L208)
- `rainimmunity` 组件在构造时推 `gainrainimmunity`、移除时推 `loserainimmunity`，遮蔽来源（雨棚 `raindome`、洞穴天气 `caveweather` 这类遮雨结构／环境效果）通过给它加源来临时挂上该组件；本 prefab 只接事件，不自己加组件。[推送点](../../../raw/current/scripts/components/rainimmunity.lua#L14-L30)、[雨棚加源](../../../raw/current/scripts/components/raindome.lua#L196-L202)、[洞穴加源](../../../raw/current/scripts/components/caveweather.lua#L438-L446)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`fertilizer` | 75／2／0.5 + `SetNutrients(nutrients)`；组件构造时自带 `fertilizer` 标签 | 仅主模拟；三注册共用同一段 | [[reference/components/fertilizer]]；[赋值](../../../raw/current/scripts/prefabs/spoiledfood.lua#L115-L119) |
| 组件：`deployable` | `DEPLOYMODE.CUSTOM` + `fertilizer_ondeploy` + `keep_in_inventory_on_deploy` | 仅主模拟；pristine 已给 `deployable` 标签与预览名 | [[reference/components/deployable]]；[构造](../../../raw/current/scripts/standardcomponents.lua#L1823-L1831) |
| 组件：`fertilizerresearchable` | 研究键 = `inst.prefab`，喂给植物登记册 | 仅主模拟 | [[reference/components/fertilizerresearchable]]；[接线](../../../raw/current/scripts/prefabs/spoiledfood.lua#L130-L131) |
| 组件：`workable`／`lootdropper`／`stackable` | 锤击把堆叠换成 `spoiled_food`／`boneshard` | 仅 `spoiled_fish`／`spoiled_fish_small`，仅主模拟 | [[reference/components/workable]]、[[reference/components/lootdropper]]；[挂件](../../../raw/current/scripts/prefabs/spoiledfood.lua#L225-L235) |
| 组件：`edible` | `SetForceSpoiledFood(true)`、`healthvalue = -1`、`hungervalue = -10`、理智 0；**能处理腐食的食客**取用时这三项会被 `math.max(0, …)` 截为 0/0/0 | 仅主模拟；三注册同值；截断只在 `CanProcessSpoiledItem` 为真时发生 | [[reference/components/edible]]；[赋值](../../../raw/current/scripts/prefabs/spoiledfood.lua#L140-L143)、[理智截断](../../../raw/current/scripts/components/edible.lua#L76-L77)、[饥饿截断](../../../raw/current/scripts/components/edible.lua#L101-L102)、[生命截断](../../../raw/current/scripts/components/edible.lua#L127-L128) |
| 组件：`eater`／`inspectable` | 两套判定各管一处：`GetStatus` 挂在主模拟的 `inspectable` 上、检查时由服务端求值，它读 `eater:IsSpoiledProcessor()`（组件布尔）返回 `CAN_PROCESS`；EAT 的 `strfn` 读**标签**——执行者带 `spoiledprocessor` 且物品带 `spoiledfood`，或执行者带 `allspoiledprocessor` 且物品带 `spoiled`，才显示 PROCESS | 检查走服务端 `inspectable`（主模拟求值）；WX-78 消化模块／技能会同时置组件与标签 | [[reference/components/eater]]、[[reference/components/inspectable]]；[状态](../../../raw/current/scripts/prefabs/spoiledfood.lua#L68-L71)、[动作字符串](../../../raw/current/scripts/actions.lua#L787-L797) |
| 组件：`disappears`／`rainimmunity` | 雨中消失 25–35 秒；**到期前**获雨免疫或入包会取消任务、不保留剩余，再次暴露重抽完整延迟；消失一旦触发，`StopDisappear` 直接返回 | 仅 `spoiled_food` 的主模拟 | [[reference/components/disappears]]、[[reference/components/rainimmunity]]；[接线](../../../raw/current/scripts/prefabs/spoiledfood.lua#L200-L211) |
| 组件：`oceanfishingtackle` | 一次性鱼饵：build `oceanfishing_lure_mis`、符号 `hook_spoiledfood`、`SPOILED_FOOD` 参数 | 仅 `spoiled_food`、仅主模拟 | [[reference/components/oceanfishingtackle]]；[设置](../../../raw/current/scripts/prefabs/spoiledfood.lua#L197-L198)、[参数](../../../raw/current/scripts/tuning.lua#L296-L296) |
| 组件：`fuel`／`burnable`／`propagator` | `SMALL_FUEL`＝15 秒；燃烧 7.5 秒；传播者 flashpoint 5–10、decay 0.5 | 主模拟；燃料值在投料时被 `fueled` 乘倍率结算 | [[reference/components/fuel]]、[[reference/components/burnable]]；[挂载](../../../raw/current/scripts/prefabs/spoiledfood.lua#L135-L138)、[投料结算](../../../raw/current/scripts/components/fueled.lua#L205-L212) |
| 组件：`selfstacker`／`smotherer`／`driedsalticon` | 落地自动并堆；`smotherer` 让它能当灭火物（目标带 `smolder` 时出 SMOTHER 候选）；盐渍图标（本 prefab 关掉 `SetCollectsOnDried(false)`） | 三注册都在 pristine 加 `selfstacker`；`driedsalticon` 仅 `spoiled_food` | [[reference/components/selfstacker]]、[[reference/components/smotherer]]、[[reference/components/driedsalticon]]；[接线](../../../raw/current/scripts/prefabs/spoiledfood.lua#L164-L165) |
| 标签：`icebox_valid`／`saltbox_valid`／`spoiledfood` | 冰箱、盐盒的过滤标签；`spoiledfood` 供 EAT 走 PROCESS 分支 | 两端加标签，容器过滤读标签 | [盐盒过滤](../../../raw/current/scripts/containers.lua#L1049-L1055)、[冰箱过滤](../../../raw/current/scripts/containers.lua#L1020-L1022) |
| 关联 Prefab：`boneshard`／`gridplacer_farmablesoil` | 锤鱼产物；部署预览 | 由 `lootdropper`／`overridedeployplacername` 引入 | [依赖表](../../../raw/current/scripts/prefabs/spoiledfood.lua#L8-L27) |
| 资源：`anim/spoiled_food.zip`／`spoiled_fish.zip`／`spoiled_fish_small.zip`／`oceanfishing_lure_mis.zip` | 三套 bank／build 与鱼饵外观 | 原版资源，不随页分发 | [assets](../../../raw/current/scripts/prefabs/spoiledfood.lua#L1-L21) |

## 典型接入

### 调高变质的鱼的堆叠上限与燃料值

- 前提与选择：服务端 `modmain.lua` 的 `AddPrefabPostInit`。选实例字段而非改 `TUNING.SPOILED_FISH_*`，因为常量同时是 `fish_onhit` 的除数与 `workleft` 的乘数，改错一边会让"一锤掉几件"的比例失真。
- 生效与撤销：新生成实例立刻生效；`maxsize` 上调只影响之后的合并，已有堆叠不会自动补满。`maxsize` 只能取五个档位常量（`STACK_SIZE_LARGEITEM` 10／`MEDITEM` 20／`SMALLITEM` 40／`TINYITEM` 60／`PELLET` 120）——赋值会转给 replica 的 `SetMaxSize` 按键值查表，写 55 这类任意数会查不到档位；本例用 60，是真正的上调（40→60）。卸模组后回到 40／15，**已经超过 40 的堆不会被自动拆开**，replica 只用 `IsOverStacked()` 报告超限。[堆叠](../../../raw/current/scripts/prefabs/spoiledfood.lua#L127-L128)、[档位](../../../raw/current/scripts/tuning.lua#L81-L85)、[replica 编码](../../../raw/current/scripts/components/stackable_replica.lua#L1-L9)、[超限判定](../../../raw/current/scripts/components/stackable_replica.lua#L126-L128)、[燃料](../../../raw/current/scripts/prefabs/spoiledfood.lua#L135-L136)
- 原版依据：见[锤击拆分](#锤击拆分)与关联导航的 `fuel` 行。

```lua
-- modmain.lua：变质的鱼 40 → 60 一叠，当燃料时给 30 秒
AddPrefabPostInit("spoiled_fish", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    inst.components.stackable.maxsize = GLOBAL.TUNING.STACK_SIZE_TINYITEM
    inst.components.fuel.fuelvalue = GLOBAL.TUNING.SEG_TIME
end)
```

- 调整与验证：`TUNING.SEG_TIME`＝30、`STACK_SIZE_TINYITEM`＝60；实机看燃料槽增量（`fueled` 还会乘湿 penalty 0.75 等倍率），并对着新合成的 60 条鱼堆锤一锤——**在没有工具效率／`workmultiplier` 修正时**，按[锤击拆分](#锤击拆分)，`spoiled_fish` 应掉 1 腐烂物 + 1 骨片、堆叠减 1。

## 常见陷阱与限制

- **误区：变质的鱼只能吃、不能当肥料。** 三注册共用同一个 `fn`，肥料组件、部署、`deployable` 标签全在共用段里，鱼的 nutrients 甚至比腐烂物高（16 对 8）；区别只在 `spoiled_food` 多了鱼饵与雨曝。[共用段](../../../raw/current/scripts/prefabs/spoiledfood.lua#L115-L155)
- **边界：雨天的 `spoiled_food` 会真的消失，不是腐坏。** 走 `disappears`：计时 25–35 秒，到期前可被取消（入包、雨停、拿到雨免疫），取消不保留剩余时间、再暴露重抽整段；到期时睡眠实体直接 `Remove`、清醒实体先禁拾（`canbepickedup = false`）再播 `dissolve` 并移除。两种鱼没有 `disappears`，淋雨不会消失。[消失](../../../raw/current/scripts/components/disappears.lua#L12-L49)
- **边界：`inspectable.nameoverride` 不是显示名。** 小鱼块仍显示"变质的小鱼块"，只有检查台词按 `SPOILED_FISH` 取键。[注释](../../../raw/current/scripts/components/inspectable.lua#L5-L7)
- **边界：锤烧着的鱼，普通火给灰、受控燃烧保留原产物。** `fish_onhit` 直接调 `DropLoot`，而 `DropLoot` 在实体带 `burnt`／正在燃烧时会改写成 `ash`（`boneshard`、`spoiled_food` 都不在 `BURNED_LOOT_OVERRIDES` 里，也没有 `_cooked` 变体）；但同一分支里 `burnable:GetControlledBurn()` 为真时**跳过改写**，产物照旧。[改写](../../../raw/current/scripts/components/lootdropper.lua#L415-L442)、[受控燃烧](../../../raw/current/scripts/components/lootdropper.lua#L432-L437)、[覆盖表](../../../raw/current/scripts/tuning.lua#L4511-L4530)
- **边界：两条锤击常量不是同一个。** 掉落数用 `SPOILED_FISH_WORK_REQUIRED` 做除数，小鱼块的 `workleft` 用 `SPOILED_FISH_SMALL_WORK_REQUIRED`；两者当前都是 1，只改其中一个会让小鱼块的"锤数"与"每次掉落数"脱钩。[常量](../../../raw/current/scripts/tuning.lua#L4129-L4130)、[算式](../../../raw/current/scripts/prefabs/spoiledfood.lua#L34-L34)
- **边界：`tradable` 有组件但没有卖价。** `goldvalue` 组件默认 0，本 prefab 未赋值，"收金值"的判定不会为它出货。[默认值](../../../raw/current/scripts/components/tradable.lua#L1-L7)
- **未核验**：未运行游戏。25–35 秒消失窗口、鱼饵咬钩权重、"锤烧着的鱼在**普通火**下掉灰（受控燃烧除外）"均为源码推导（推导式：`delay 25 + math.random()*10` 落在 25–35；`floor(clamp(1/1,1,10)) = 1`）。

## 验证记录

- 源码审阅：2026-11-09，Steam buildid 26474873。通读 `prefabs/spoiledfood.lua` 全文（共用 `fn`、三组 init、锤击、雨曝、GetStatus、注册）；`prefabs/fertilizer_nutrient_defs.lua` 全文与 `tuning.lua` 的 `SPOILEDFOOD_*`／`SPOILED_*`／`SPOILED_FISH_*`／`SMALL_FUEL`／`STACK_SIZE_SMALLITEM`／养分表；`components` 下 `fertilizer`／`fertilizable`／`fertilizerresearchable`／`workable`／`stackable`／`lootdropper`／`disappears`／`rainimmunity`／`edible`／`eater`／`inspectable`／`selfstacker`／`smotherer`／`driedsalticon`／`fuel`／`fueled`／`tradable`／`tool`；`standardcomponents` 的 `MakeDeployableFertilizer*` 与 `fertilizer_ondeploy`；`componentactions` 的 FERTILIZE 候选与 workable 校验；`actions` 的 FERTILIZE／EAT／DoToolWork；`containers` 的盐盒与冰箱过滤；`chinese_s.po` 名称；`scrapbookdata` 交叉核对（`stacksize=40`、`hungervalue=-10`、`healthvalue=-1`、`fuelvalue=15`、`workable="HAMMER"`）。
- 修订记录：2026-11-09 第 1 轮独立检视回源复核后按点名条目修正——P1-1 掉落改为 `SetLoot` 固定表＋每次 `DropLoot` 两件（L55-L56、L31）、P2-2 补 `withered` 门禁与 `self_fertilizable` 自身分支（L47）、P2-3 雨免疫来源改为雨棚／洞穴（L66）、P3-4 补第 4 参 `kind` 差异（L20）；R3 复验再补 P1-1 限定（L55-L56）：“每次 2 件”仅适用于 `spoiled_fish`，`spoiled_fish_small` 走 `AddRandomLoot` 每次只掉 1 件。外部复检轮再改：P2 补雨曝中断重抽与睡眠／清醒两种收尾（L64）、三维负值在可处理腐食时截为 0（L35、L77）、`GetStatus` 与 EAT `strfn` 两套判定分开（L77）、部署补骑乘／浮具门禁与“施肥成功才扣件”（L47）、锤击补“无效率修正”前提与受控燃烧不改写（L55、L112）、堆叠示例改为 40→60 的真实上调并说明档位限制与超限堆不自动拆（L88-L105）。D 组第二轮再改：摘要改“负饥饿与负生命、理智 0”（L6）、雨曝改“取消即不保值／重抽／仅清醒播动画”（L22）、`GetStatus` 注明主模拟求值（L78）、补作物冬季≤0°C 与 `witherable` 两道施肥门禁（L47）、示例与灰烬补“无效率修正”“普通火”前提（L106、L116）。第三轮再改：监听行与陷阱统一为“取消不保值／重抽／睡眠直删、清醒先禁拾再溶解”（L66、L111），导航第三列由“客户端读 getstatus”改为“服务端 inspectable 求值”与 L78 对齐。第四轮再改：L66 去掉“停表”说法，统一为“入包或获得雨免疫即取消任务、不保值、再暴露重抽完整 25–35 秒”；其余段落未动。
- 未覆盖：`quagmire`（暴食模式）分支的 `master_postinit`、鱼饵入水后的实际咬钩结算、其它模组对 `spoiledfood` 标签的读取。
- 游戏运行：未运行；天数与秒数按 `total_day_time`＝480 秒换算。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 见交付回报。

## 相关链接

- 肥料链路：[[reference/components/fertilizer]]、[[reference/components/fertilizable]]、[[reference/components/fertilizerresearchable]]、[[reference/prefabs/soil-amender]]
- 可食与鲜度：[[reference/components/edible]]、[[reference/components/eater]]
- 拆分与燃烧：[[reference/components/workable]]、[[reference/components/lootdropper]]、[[reference/components/burnable]]、[[reference/components/fuel]]
- 雨曝：[[reference/components/disappears]]、[[reference/components/rainimmunity]]
- 同族物品：[[reference/prefabs/bird-egg]]（熟蛋腐坏成 `spoiled_food`）、[[reference/prefabs/wobster]]（死龙虾腐坏成 `spoiled_fish`）

返回 [[reference/prefabs/special-food-index|节庆、角色与特殊食品索引]]。
