---
title: "冰（ice）"
created: "2026-11-09"
updated: "2026-11-10"
type: prefab
summary: "迷你冰川的采掘产物冰：GENERIC 冷食＋固定三维、frozen／molebait／watersource 三标签、smotherer 扑灭阴燃与浇灭明火、以 MATERIALS.ICE 补冰帽新鲜度、一次水源、堆叠 40；腐坏按有无持有者分成加湿与播 melt 两支。"
aliases: ["ice", "冰", "冰块"]
tags: [prefab, item, food]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/inv_rocks_ice.lua", "prefabs/rock_ice.lua", "prefabs/deerclops.lua", "prefabs/hats.lua", "prefabs/icebox.lua", "prefabs/wateringcan.lua", "prefabs/player_common.lua", "components/perishable.lua", "components/edible.lua", "components/smotherer.lua", "components/burnable.lua", "components/watersource.lua", "components/repairer.lua", "components/repairable.lua", "components/stackable.lua", "components/farming_manager.lua", "componentutil.lua", "componentactions.lua", "actions.lua", "cooking.lua", "recipes.lua", "prefabs/firesuppressor.lua", "standardcomponents.lua", "stringutil.lua", "tuning.lua", "constants.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 冰（ice）

## 快速判断

- 职责与适用：`ice` 是 `inv_rocks_ice.lua` 注册的可拾取物品（`item`）兼可食用（`food`），注册名与文件名不同。它同时承担冷食、一次性水源、灭火材料与 `MATERIALS.ICE` 修补材料四种职责，获得链是 [迷你冰川的阶段表](../../../raw/current/scripts/prefabs/rock_ice.lua#L23-L58) 的采掘产物（S4）。[注册行](../../../raw/current/scripts/prefabs/inv_rocks_ice.lua#L131)、[构造段](../../../raw/current/scripts/prefabs/inv_rocks_ice.lua#L53-L128)
- 优先使用：要数值、鲜度与实际秒数查[数值与结算](#数值与结算)；要火、燃烧与融化查[灭火与遇火](#灭火与遇火)；要谁消耗它查[关联机制导航](#关联机制导航)。
- 重要副作用：腐坏不产生 `spoiled_food`，而按“有无持有者”分成两支——有持有者就按其叠数给持有者加湿后直接删除；在地面就按每件 25 点向所在农田格请求土壤湿度（单格上限 100，夹取前）、关掉 `persists` 与可拾取、播 `melt` 后自删（S5）。[onperish](../../../raw/current/scripts/prefabs/inv_rocks_ice.lua#L19-L35)
- 修改边界：本页只写物品侧。季节阶段、采掘与逐级冰产量见 [[reference/prefabs/rock-ice]]；水洼与冰溅 FX 见 [[reference/prefabs/ice_puddle]]；提灯与矿工帽的 CAVE 燃料结算见 [[reference/prefabs/lantern]]（S6）。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `ice` | 单工厂、无变体：`fn` 建实体 → Transform／AnimState／SoundEmitter／Network＋`MakeInventoryPhysics` → bank／build 均为 `ice` → 加 `frozen`／`molebait`／`watersource` 三标签与 `MakeInventoryFloatable("med", 0.05, {0.65, 0.5, 0.65})` → `SetPristine` → 客户端到此返回 → 主模拟从 `f1`–`f3` 随机一个动画名并挂组件 | 动画名写入存档，读档按 `data.anim` 重播；组件全部只在主模拟添加。[工厂前半](../../../raw/current/scripts/prefabs/inv_rocks_ice.lua#L53-L77)、[抽动画名](../../../raw/current/scripts/prefabs/inv_rocks_ice.lua#L79-L80)、[OnSave／OnLoad](../../../raw/current/scripts/prefabs/inv_rocks_ice.lua#L8-L17) |

- 标签分层（S3）：`frozen` 与 `molebait` 在 `SetPristine` 之前由物品自己加，两端都执行；`watersource` 则是注释写明“由 watersource 组件负责”的重复标签，组件构造时还会按 `available` 再加一次。[标签](../../../raw/current/scripts/prefabs/inv_rocks_ice.lua#L66-L69)、[组件侧标签](../../../raw/current/scripts/components/watersource.lua#L1-L11)
- 三条标签各有独立消费者：`frozen` 决定能否浇灭明火、决定冰箱内是否停止腐坏与要不要吃 `firemelt`；`molebait` 是鼹鼠引诱类标签；`watersource` 是填充动作的资格。

### 名称与获得

简中名 `STRINGS.NAMES.ICE`＝“冰”（S2）。同族的 `rock_ice` 是“迷你冰川”，其融化态 `ROCK_ICE_MELTED` 是“融化的迷你冰川”；标题、别名与正文都按这三个名字分开写，不要用“冰矿／冰川”代称。[NAMES.ICE](../../../raw/current/scripts/languages/chinese_s.po#L344054-L344057)

获得（S4）：本页展开的来源是采掘 `rock_ice`——按当前阶段随机掉冰，`abandon_ship` 沉没时按当前阶段给固定件数；逐级产量与整级快照的数字归 [[reference/prefabs/rock-ice#采掘与冰产量]]。冰并非只有这一个来源：原版 `mutateddeerclops` 的掉落表就含 1 件必掉与另 1 件 75% 的冰，按注册名检索还会命中其他掉落表，不能读成“唯一来源”。[变异巨鹿掉落表](../../../raw/current/scripts/prefabs/deerclops.lua#L343-L354)、[掉表接线](../../../raw/current/scripts/prefabs/deerclops.lua#L650-L654)

## 数值与结算

时间基准：`seg_time`＝30 秒、`total_day_time`＝480 秒、`perish_warp`＝1，鲜度天数一律按 480 秒/天折算（S1）。[基准](../../../raw/current/scripts/tuning.lua#L19-L28)、[perish_warp](../../../raw/current/scripts/tuning.lua#L47)

| 项 | 定义处 | 结算处与实际值 |
|---|---|---|
| 食用 | `foodtype=FOODTYPE.GENERIC`、`healthvalue=HEALING_TINY/2`、`hungervalue=CALORIES_TINY/4`、`degrades_with_spoilage=false` [edible 定义](../../../raw/current/scripts/prefabs/inv_rocks_ice.lua#L82-L86) | `HEALING_TINY`＝1、`CALORIES_TINY`＝75/8＝9.375，故生命 0.5、饥饿 2.34375；`degrades_with_spoilage=false` 让三项不随鲜度衰减 [HEALING_TINY](../../../raw/current/scripts/tuning.lua#L2021)、[CALORIES_TINY](../../../raw/current/scripts/tuning.lua#L2058)、[饥饿取值](../../../raw/current/scripts/components/edible.lua#L96-L119) |
| 冷食 | `temperaturedelta=COLD_FOOD_BONUS_TEMP`、`temperatureduration=FOOD_TEMP_BRIEF*1.5` [温差定义](../../../raw/current/scripts/prefabs/inv_rocks_ice.lua#L87-L88) | −40 度、7.5 秒；进食时按 `SetTemperatureInBelly(delta*(1-chill)*mult, duration*mult)` 把体温往 −40 拉 [两个常量](../../../raw/current/scripts/tuning.lua#L2328-L2330)、[体温结算](../../../raw/current/scripts/components/edible.lua#L201-L208) |
| 鲜度 | `SetPerishTime(TUNING.PERISH_SUPERFAST)`＋`StartPerishing()`，且没有 `onperishreplacement` [perishable 定义](../../../raw/current/scripts/prefabs/inv_rocks_ice.lua#L95-L98) | 定义值 3 天＝1440 秒；地面上再乘 `PERISH_GROUND_MULT`＝1.5，实际约 2 天＝960 秒，持有者路径不乘这一项。倍率取法是分步而非一条乘法链：先按地点定基数（无持有者 1.5／有持有者 1），再乘湿度 1.3；温度项是三支互斥——环境低于 0 且未 `firemelt` 的 `frozen` 物品 → 置 0（不腐坏）、低于 0 且 `firemelt` → ×0.75 再 ×30、不低于 0 且 `firemelt` → ×30；最后乘 local 与 global [PERISH_SUPERFAST](../../../raw/current/scripts/tuning.lua#L2045)、[倍率链](../../../raw/current/scripts/components/perishable.lua#L97-L129)、[地面与湿倍率](../../../raw/current/scripts/tuning.lua#L2083-L2084) |
| 冰箱内 | 同一 `frozen` 标签与持有者 `fridge` 标签 [冰箱标签](../../../raw/current/scripts/prefabs/icebox.lua#L59) | `fridge` 分支对 `frozen` 且非 `nocool`／`lowcool` 的物品把倍率置 0，冰在冰箱里不会腐坏（源码推导） [fridge 分支](../../../raw/current/scripts/components/perishable.lua#L81-L86) |
| 堆叠 | `stackable.maxsize=TUNING.STACK_SIZE_SMALLITEM` [堆叠定义](../../../raw/current/scripts/prefabs/inv_rocks_ice.lua#L102-L103) | 本版 40，不是 20 [STACK_SIZE_SMALLITEM](../../../raw/current/scripts/tuning.lua#L83) |
| 水源 | `watersource.override_fill_uses=1`＋`onusefn=onuseaswatersource` [水源定义](../../../raw/current/scripts/prefabs/inv_rocks_ice.lua#L115-L117) | 填充一次时从叠里取一件删除、非叠则整件删除；洒水壶读该字段只补 1 次耐久 [消费回调](../../../raw/current/scripts/prefabs/inv_rocks_ice.lua#L45-L51)、[接收方](../../../raw/current/scripts/prefabs/wateringcan.lua#L26-L37)、[Use](../../../raw/current/scripts/components/watersource.lua#L33-L37) |
| 修补 | `repairer.repairmaterial=MATERIALS.ICE`、`perishrepairpercent=.25`，其余修理字段保持默认 0 [repairer 定义](../../../raw/current/scripts/prefabs/inv_rocks_ice.lua#L111-L113) | 对 `repairmaterial` 同为 ICE 的新鲜度型目标补 25% 鲜度；原版消费者是冰帽 [补鲜度结算](../../../raw/current/scripts/components/repairable.lua#L126-L133)、[冰帽 repairable](../../../raw/current/scripts/prefabs/hats.lua#L1707-L1709) |

修理候选不是“目标有鲜度标签就会出现”：它要求右键（`right=true`）、目标带 `repairable_ice`、冰带 `freshen_ice`，且目标必须带 `fresh`／`stale`／`spoiled` 之一——候选只查鲜度标签，不读精确剩余百分比。前置先看骑乘：骑乘时目标要归自己（`IsGrandOwner`）否则直接返回；`elseif` 的搬重物分支只在**未骑乘**时求值，正在搬重物（`IsHeavyLifting`）也直接返回。因此满鲜目标照样会出现候选。[修理候选](../../../raw/current/scripts/componentactions.lua#L1679-L1700)、[标签写入](../../../raw/current/scripts/components/repairable.lua#L1-L20)

结算在 `Repair`：材料必须同 `repairmaterial`；鲜度分支在剩余满 100%（`perishremainingtime >= perishtime`）时返回 false，此时**既不补鲜也不消耗冰**；95%–99.9% 会成功并消耗一块。`NEEDSREPAIRS_THRESHOLD`＝0.95 只服务 `NeedsRepairs` 的“需要修理”提示，全快照唯一外部调用在 `stringutil`。反例：普通鲜食没有 `repairable_ice`，从不出现候选；满鲜的冰帽同样会出现候选，但结算退回、不消耗冰。[Repair 结算](../../../raw/current/scripts/components/repairable.lua#L148-L179)、[鲜度分支](../../../raw/current/scripts/components/repairable.lua#L126-L133)、[NeedsRepairs](../../../raw/current/scripts/components/repairable.lua#L90-L104)、[唯一外部调用](../../../raw/current/scripts/stringutil.lua#L352)

## 灭火与遇火

### 灭火：smotherer＋frozen

`ice` 挂的是空标记组件 `smotherer`（本体只有 `self.inst`），起作用的是“拥有该组件的物品会成为 `USEITEM` 动作的提供者”这条注册关系；再配合物品自带的 `frozen` 标签，同一个冰会贡献两条动作：目标带 `smolder` 时给 `SMOTHER`（扑灭阴燃，先判，命中后不再进 `elseif`）；否则目标带 `fire`、冰带 `frozen`、且目标不在任何持有者里（`replica.inventoryitem:IsHeld()` 为假——比“拿在手上”更广，背包、箱子里的目标都算持有）时给 `MANUALEXTINGUISH`（浇灭明火）。[空组件](../../../raw/current/scripts/components/smotherer.lua#L1-L6)、[动作收集](../../../raw/current/scripts/componentactions.lua#L1736-L1743)、[IsHeld](../../../raw/current/scripts/components/inventoryitem.lua#L462-L468)、[挂载](../../../raw/current/scripts/prefabs/inv_rocks_ice.lua#L90)

两条动作的消耗都写在 `burnable`，但分支不同：`SMOTHER` 依次试 `finiteuses` → `stackable:Get():Remove()` → `health`＋`combat`；`MANUALEXTINGUISH` 只试前两者，没有 `health` 分支。冰只有 `stackable`，所以两条都只耗 1 块。浇灭明火时传入的 `heatpct` 是 `SMOTHERER_EXTINGUISH_HEAT_PERCENT`＝0.2。[扑灭消耗](../../../raw/current/scripts/components/burnable.lua#L417-L429)、[灭火消耗](../../../raw/current/scripts/components/burnable.lua#L454-L463)、[heatpct](../../../raw/current/scripts/tuning.lua#L2942)、[两条动作执行](../../../raw/current/scripts/actions.lua#L1844-L1857)

### 遇火：firemelt 与 frozenfiremult

`perishable.frozenfiremult` 不是“会不会着火”，而是腐坏速率开关：物品收到 `firemelt` 置真、收到 `stopfiremelt` 置假；为真时倍率再乘 `PERISH_FROZEN_FIRE_MULT`＝30。[开关回调](../../../raw/current/scripts/prefabs/inv_rocks_ice.lua#L37-L43)、[两段分支](../../../raw/current/scripts/components/perishable.lua#L111-L121)、[两个倍率](../../../raw/current/scripts/tuning.lua#L2077-L2078)

两处外部推事件＋一处本地清位：燃烧物的 `propagator` 以 `PROPAGATOR_DT`＝0.5 秒为周期 tick，对范围内带 `frozen`／`meltable` 的实体推 `firemelt`，火灭时对当时仍在范围内的实体推 `stopfiremelt`；玩家吃到火焰伤害时，`player_common` 给背包内所有 `frozen` 物品推 `firemelt`，伤害结束时推 `stopfiremelt`；物品被收进背包时直接调用 `onstopfiremelt` 清位。[tick 周期](../../../raw/current/scripts/components/propagator.lua#L3)、[推 firemelt](../../../raw/current/scripts/components/propagator.lua#L213-L216)、[推 stopfiremelt](../../../raw/current/scripts/components/propagator.lua#L239-L242)、[玩家着火](../../../raw/current/scripts/prefabs/player_common.lua#L428-L440)、[入包清除](../../../raw/current/scripts/prefabs/inv_rocks_ice.lua#L107-L109)

边界：冰没有 `burnable`、没有 `fuel`，也不是任何火源；`firemelt` 只改腐坏速率，不会让冰点火或烧起来。真正的“火”相关能力只有上节的灭火。[组件段无燃烧件](../../../raw/current/scripts/prefabs/inv_rocks_ice.lua#L90-L98)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：perishable | `SetPerishTime`＋`StartPerishing`；`SetOnPerishFn(onperish)`；`frozenfiremult` 由外部事件切换 | 仅主模拟；无替换物，腐坏即删 | [[reference/components/perishable]]；[更新与腐坏](../../../raw/current/scripts/components/perishable.lua#L60-L137) |
| 组件：edible | 写饥饿／生命／食性／温差；`degrades_with_spoilage=false` | 仅主模拟 | [[reference/components/edible]]；[吃下结算](../../../raw/current/scripts/components/edible.lua#L196-L208) |
| 组件：stackable | 上限 40；腐坏按整叠计件，水源与灭火只取 1 件 | 仅主模拟 | [[reference/components/stackable]]；[接线](../../../raw/current/scripts/prefabs/inv_rocks_ice.lua#L102-L103) |
| 组件：watersource | `onusefn` 删一件、`override_fill_uses=1` | 仅主模拟；填充方是洒水壶 | [[reference/components/watersource]]；[Use](../../../raw/current/scripts/components/watersource.lua#L33-L37) |
| 组件：smotherer | 空标记，注册 `SMOTHER`／`MANUALEXTINGUISH` 候选 | 仅主模拟添加；动作随后端判定 | [[reference/components/smotherer]]；[收集器](../../../raw/current/scripts/componentactions.lua#L1736-L1743) |
| 组件：repairer | `repairmaterial=MATERIALS.ICE`、`perishrepairpercent=.25`，据此写 `freshen_ice` 标签 | 修的是新鲜度，不是耐久 | [[reference/components/repairer]]；[标签写入](../../../raw/current/scripts/components/repairer.lua#L8-L24)、[MATERIALS](../../../raw/current/scripts/constants.lua#L1940-L1949) |
| 组件：inventoryitem | 图片名 `ice`；`SetOnPutInInventoryFn(onstopfiremelt)` | 仅主模拟设置回调 | [[reference/components/inventoryitem]]；[接线](../../../raw/current/scripts/prefabs/inv_rocks_ice.lua#L107-L109) |
| 组件：hauntable | `MakeHauntableLaunchAndSmash`：作祟只把物品弹飞 | 本版“砸碎”分支整段被注释，作祟不会损坏冰 | [空分支](../../../raw/current/scripts/standardcomponents.lua#L1071-L1105) |
| 组件：tradable／bait／snowmandecor／inspectable | 可交易、鼹鼠诱饵、可装饰雪人、可查看 | 只在主模拟添加 | [其余组件](../../../raw/current/scripts/prefabs/inv_rocks_ice.lua#L100-L121) |
| API：cooking 食材 | 注册为烹饪食材，属性给 `frozen=1` | 进锅时按 frozen 计 | [食材属性](../../../raw/current/scripts/cooking.lua#L137) |
| API：farming_manager | 地面腐坏时 `AddSoilMoistureAtPoint` | 只在有耕作覆写的格子生效 | [加土壤湿度](../../../raw/current/scripts/components/farming_manager.lua#L461-L470) |
| API：DoDeltaMoistureToEntity | 有持有者时传 `amount=2×叠数`、`itemmult=2`、`skipwaterproof=true` | 玩家走 moisture 分支、忽略 itemmult（2/件）；带 `inventoryitem` 的持有者走 itemmult 分支（4/件）；两者都没有的容器不生效 | [加湿函数](../../../raw/current/scripts/componentutil.lua#L2399-L2408) |
| 关联 Prefab：`rock_ice` | 采掘与沉没的来源；`ice_puddle`／`ice_splash` 是它的子表现 | 主模拟生成冰 | [[reference/prefabs/rock-ice]]；[依赖声明](../../../raw/current/scripts/prefabs/rock_ice.lua#L7-L12) |
| 关联 Prefab：`ice_puddle` | 冰面巨石贴地水洼与冰溅 FX，与冰物品无组件关系 | 非专用端子挂 | [[reference/prefabs/ice_puddle]] |
| 消费者（只列名）：`icehat` | 配方 2 绳＋10 冰；本体 `repairable(ICE)`＋perishable | 见帽子环境页的冰帽条目 | [[reference/prefabs/hats-environment]]；[冰帽配方](../../../raw/current/scripts/recipes.lua#L542) |
| 消费者（只列名）：`firesuppressor`（雪球发射器） | 配方消耗 15 冰；本体 `fueled` 主燃料是 BURNABLE（默认）、副燃料 CHEMICAL，两种燃料都收，但不烧冰 | 见雪球发射器页（Wiki 待建） | [配方](../../../raw/current/scripts/recipes.lua#L569)、[主／副燃料](../../../raw/current/scripts/prefabs/firesuppressor.lua#L341-L349) |
| 消费者（只列名）：`waterballoon`／`moondial`／`trophyscale_fish`／`turf_mud` | 各自配方按件消耗冰 | 配方行 | [水球](../../../raw/current/scripts/recipes.lua#L512)、[月晷](../../../raw/current/scripts/recipes.lua#L570)、[鱼类计重器](../../../raw/current/scripts/recipes.lua#L689)、[泥地皮](../../../raw/current/scripts/recipes.lua#L717) |
| 消费者（只列名）：`icebox` | 冰箱的 `fridge` 标签让 `frozen` 物品倍率归零 | 持有者路径 | [[reference/prefabs/icebox]]；[fridge 分支](../../../raw/current/scripts/components/perishable.lua#L81-L86) |
| 资源：`anim/ice.zip` | 物品动画包，三个帧名 `f1`／`f2`／`f3` | 原版资源，不随页分发 | [资产与帧名](../../../raw/current/scripts/prefabs/inv_rocks_ice.lua#L1-L6) |

## 典型接入

### 只改冰的堆叠上限（连带改整叠融化的加湿量）

- 前提与选择：这是服务端 `modmain.lua` 的 postinit。冰的上限取全局 `TUNING.STACK_SIZE_SMALLITEM`（本版 40），直接改 TUNING 会连带所有小件，所以在 `ice` 实例上覆写 `maxsize`。[原上限](../../../raw/current/scripts/prefabs/inv_rocks_ice.lua#L102-L103)
- 生效与撤销：新生成的冰立刻用新上限；因为腐坏回调用的是 `stackable:StackSize()`，上限变化会线性放大“整叠融化”的加湿量——有持有者时玩家每件 2 点（带 `inventoryitem` 的容器持有者 4 点），地面时每件 25 点（单格土壤湿度上限 100，满叠一次就顶满）。停用模组后新生成的冰回到 40，已存在的叠不变。[按叠数结算](../../../raw/current/scripts/prefabs/inv_rocks_ice.lua#L19-L35)、[地面每件水量](../../../raw/current/scripts/tuning.lua#L5784)
- 原版依据：组件全部在主模拟添加，postinit 拿到的是已挂好的实例。`maxsize` 的 setter 会把值交给 stackable replica 的 `SetMaxSize`，后者用一张只有五个合法值的码表编码成网络量，客户端再按同一张表还原；60 正好是 `STACK_SIZE_TINYITEM`，所以在允许集合内。[默认上限与赋值](../../../raw/current/scripts/components/stackable.lua#L23-L34)、[五个码位](../../../raw/current/scripts/components/stackable_replica.lua#L1-L9)

```lua
-- modmain.lua：把冰的堆叠上限从 40 改成 60
AddPrefabPostInit("ice", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    local stackable = inst.components.stackable
    if stackable ~= nil then
        stackable.maxsize = 60
    end
end)
```

- 调整与验证：`maxsize` 的单位是“件”。土壤湿度另有上限——`SetSoilMoisture` 把值夹在 `[wetness, SOIL_MAX_MOISTURE_VALUE]`（100），所以 40 叠请求 1000 点、60 叠请求 1500 点都会顶满，“最终湿度”分不出两者。要验证按叠数传递，可把 1–2 块冰丢在**低湿耕地**上看每件 25 点的差额，或记录调用 `AddSoilMoistureAtPoint` 时的入参（请求量 25×叠数，夹取前）。[夹取](../../../raw/current/scripts/components/farming_manager.lua#L96-L105)、[上限](../../../raw/current/scripts/tuning.lua#L5780-L5784)

## 常见陷阱与限制

- **误区：冰能“生火”或点火。** 冰没有 `burnable`、没有 `fuel`，也不是火源；`firemelt` 是被外部火源推来的腐坏加速标记。把“有 `firemelt` 监听”读成“与火同源”会写错模组。[监听](../../../raw/current/scripts/prefabs/inv_rocks_ice.lua#L37-L43)、[组件段](../../../raw/current/scripts/prefabs/inv_rocks_ice.lua#L90-L98)
- **边界：`frozenfiremult` 不会自己复位，且复位只在范围内。** 地面上的冰被火烤过后再挪到火场外，只要火还在烧、它又不在任何火的 `propagaterange` 内，就收不到 `stopfiremelt`——`propagator` 只在火灭或自身移除时对**当时仍在范围内**的实体推该事件；这段窗口里它仍按 ×30 腐坏，可靠的复位只有入包调用 `onstopfiremelt`。[只在这两处推](../../../raw/current/scripts/components/propagator.lua#L239-L242)、[入包清除](../../../raw/current/scripts/prefabs/inv_rocks_ice.lua#L107-L109)
- **边界：地面融化只对农田格有效。** `AddSoilMoistureAtPoint` 先查该格有没有耕作覆写数据，没有就整段跳过；把冰丢在普通草地上融化不会留下任何湿度。即使在农田里，单格土壤湿度也被夹到 `[wetness, 100]`，满叠融化会顶满，比较叠数要看请求量而不是最终值。[先查 overlay](../../../raw/current/scripts/components/farming_manager.lua#L461-L470)、[夹取](../../../raw/current/scripts/components/farming_manager.lua#L96-L105)
- **边界：加湿分支的“持有者”不只是手。** 判断条件是 `inventoryitem.owner`，所以放在背包／箱子里也走“有持有者”分支；若那个持有者既没有 moisture 也没有 inventoryitem（例如普通箱子），加湿会静默不生效。[owner 判定](../../../raw/current/scripts/prefabs/inv_rocks_ice.lua#L19-L25)、[两个分支](../../../raw/current/scripts/componentutil.lua#L2399-L2408)
- **边界：腐坏不留掉落，且按整叠删除。** 冰没设 `onperishreplacement`，两支都直接删掉实体（地面分支先播 `melt` 再在 `animover` 删）；有持有者时加湿量按 `stackable:StackSize()` 整叠计件，而水源与灭火只从叠里取 1 件。想让它化成水必须自己挂替换物。[先回调后替换](../../../raw/current/scripts/components/perishable.lua#L277-L320)
- **边界：堆叠上限只能取五个档位。** `maxsize` 的 setter 用 `STACK_SIZE_CODES` 编码，合法值只有 10／20／40／60／120；写别的数字会在 setter 里取到 `nil` 而报错，不能用它做任意上限。[setter](../../../raw/current/scripts/components/stackable.lua#L11-L21)、[码表](../../../raw/current/scripts/components/stackable_replica.lua#L1-L9)
- **边界：`repairer` 只有鲜度一项。** `workrepairvalue`／`healthrepairvalue`／`finiteusesrepairvalue` 都是默认 0，冰不能修耐久或生命，只能补新鲜度。[默认值](../../../raw/current/scripts/components/repairer.lua#L35-L54)
- **未核验**：腐坏秒数、灭火手感、冰箱内不腐坏均为源码审阅结论，未运行游戏（S7）。

## 验证记录

- 源码审阅：2026-11-09，steam-buildid:26474873（`raw/current/BUILD.txt` 实读）。核对 `inv_rocks_ice.lua` 全文；`rock_ice.lua` 的阶段表、逐级掉冰、整级快照与沉没；`perishable` 的地面／持有者／冰箱／冻／火烤倍率链；`edible` 的饥饿、生命与体温结算；`smotherer`、`burnable`、`watersource`、`repairer`／`repairable`、`stackable`、`farming_manager`、`componentutil`、`propagator`、`player_common` 的相关分支；`tuning` 的 `PERISH_SUPERFAST`／`STACK_SIZE_SMALLITEM`／`HEALING_TINY`／`CALORIES_TINY`／`COLD_FOOD_BONUS_TEMP`／`FOOD_TEMP_BRIEF`／`ICE_MELT_GROUND_MOISTURE_AMOUNT`／`SMOTHERER_EXTINGUISH_HEAT_PERCENT`；`cooking` 的 frozen 食材登记与两处配方消费者；简中名按 `STRINGS.NAMES.ICE` 定向检索。
- 游戏运行：未运行；全部结论为源码审阅（S7）。未覆盖：多人同步时序、不同世界设置下的腐坏观感、灭火动作的实际判定距离。
- 检视修订（2026-11-09，同一 build）：①修补门禁改为“`Repair` 只在 100% 时拒绝，`NEEDSREPAIRS_THRESHOLD` 只服务 `NeedsRepairs` 提示”（`repairable.lua` L90-L104、L126-L133、`stringutil.lua` L352）；②鲜度倍率改为三支互斥（`perishable.lua` L111-L121）；③`DoDeltaMoistureToEntity` 按持有者类型分 2/件与 4/件（`componentutil.lua` L2399-L2407）；④`propagator` tick 由“每帧”改为 0.5 秒（`PROPAGATOR_DT`，L3、L94），并写明 `stopfiremelt` 只推给当时仍在范围内的实体；⑤采掘掉冰细节移归 [[reference/prefabs/rock-ice#采掘与冰产量]]，本页只留来源与链接；⑥补 `SMOTHER` 与 `MANUALEXTINGUISH` 的消耗分支差异、腐坏按整叠删除、`firesuppressor`（雪球发射器）主 BURNABLE／副 CHEMICAL、hauntable 只弹飞、`moondial`／`turf_mud` 配方锚点。锚点与结论未受影响的段落未改。
- 复验修订（2026-11-10）：①导航行 stackable 改为“腐坏按整叠计件，水源与灭火只取 1 件”，与陷阱段一致；②`firesuppressor` 链接文字改为“雪球发射器页”；③“鱼标本架”按 `STRINGS.NAMES.TROPHYSCALE_FISH`（简中“鱼类计重器”）改名。
- 外部检视修订（2026-11-10，W05 报告 R06／R07／R18）：①R06 把“只有一条主链”收窄为“本页展开的来源”，补 `mutateddeerclops` 掉落表反例（`deerclops.lua` L343-L354、L650-L654）并加进 `sources`；②R07 修理候选改写为“右键＋`repairable_ice`＋`freshen_ice`＋骑乘／搬重前置”，写明满鲜仍有候选但 `Repair` 退回且不消耗（`componentactions.lua` L1679-L1700、`repairable.lua` L148-L179）；明火候选改按 `replica.inventoryitem:IsHeld()`（背包／容器也算持有）并注明 `smolder` 分支在前；复验补充：候选还要求目标带 fresh／stale／spoiled 之一（只查标签、不读精确百分比），搬重物排除属于骑乘条件的 elseif 分支。③R18 土壤湿度补“请求量／上限 100 夹取”两层（`farming_manager.lua` L96-L105、`tuning.lua` L5780-L5784），验证改为低湿小叠或看入参。
- 结构检查：`validate_page.py` 通过；`validate_source_links.py --strict` 0 error／0 warning。本页 Lua 示例为完整片段，未在游戏内执行。

## 相关链接

- 冰的来源、季节阶段与逐级产量：[[reference/prefabs/rock-ice]]
- 融化水洼与冰溅：[[reference/prefabs/ice_puddle]]
- 冰箱保鲜与冷藏：[[reference/prefabs/icebox]]
- 冰帽与降温装备：[[reference/prefabs/hats-environment]]
- 水源与洒水壶：[[reference/components/watersource]]
- 灭火标记组件：[[reference/components/smotherer]]
- 鲜度与食物：[[reference/components/perishable]]、[[reference/components/edible]]

返回 [[reference/prefabs/index|Prefab 索引]]。
