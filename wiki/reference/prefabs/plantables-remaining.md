---
title: "尖刺灌木、石果灌木丛与猴尾草的移栽件（dug_marsh_bush、dug_rock_avocado_bush、dug_monkeytail）"
created: "2026-11-10"
updated: "2026-11-10"
type: prefab
summary: "plantables.lua 数据表剩余三项的挖出件：bank／build 与 placer 动画覆盖、图鉴与检查名分流，三个世界态的挖起产物与移植回调，以及枯萎、冬季与烧毁的差异。"
aliases: ["dug_marsh_bush", "dug_marsh_bush_placer", "dug_rock_avocado_bush", "dug_rock_avocado_bush_placer", "dug_monkeytail", "dug_monkeytail_placer", "尖刺灌木", "石果灌木丛", "猴尾草", "移植尖刺灌木", "移植猴尾草", "marsh_bush", "monkeytail"]
tags: [prefab, item]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/plantables.lua", "prefabs/marsh_bush.lua", "prefabs/rock_avocado_bush.lua", "prefabs/monkeytail.lua", "prefabs/waxed_plant_common.lua", "prefabs/monkeyisland_portal.lua", "prefabs/forest.lua", "map/static_layouts/dragonfly_arena.lua", "prefabutil.lua", "components/deployable.lua", "components/inventoryitem_replica.lua", "components/stackable.lua", "components/inventory.lua", "components/pickable.lua", "components/witherable.lua", "components/inspectable.lua", "standardcomponents.lua", "stringutil.lua", "entityscript.lua", "tuning.lua", "constants.lua", "strings.lua", "recipes.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 尖刺灌木、石果灌木丛与猴尾草的移栽件（dug_marsh_bush、dug_rock_avocado_bush、dug_monkeytail）

## 快速判断

- 职责与适用：`prefabs/plantables.lua` 的 `plantables` 数据表共 10 项，每项产出 1 个挖出件、1 个 placer 与 1 个蜡封件；本页负责其中剩余 3 项——`dug_marsh_bush`（尖刺灌木）、`dug_rock_avocado_bush`（石果灌木丛）、`dug_monkeytail`（猴尾草），连同 `dug_marsh_bush_placer`、`dug_rock_avocado_bush_placer`、`dug_monkeytail_placer` 共 6 个注册。世界态 `marsh_bush`／`rock_avocado_bush`／`monkeytail` 只作挖起来源与回种落点。[数据表](../../../raw/current/scripts/prefabs/plantables.lua#L119-L182)、[注册循环](../../../raw/current/scripts/prefabs/plantables.lua#L186-L191)、[文件导出](../../../raw/current/scripts/prefabs/plantables.lua#L193)
- 优先使用：改外观、浮水、间距与图鉴文案看[三件的数据差异](#三件的数据差异)；改挖起产物与回种后果看[世界态来源与挖起](#世界态来源与挖起)；公共厂（资产、浮水、堆叠、燃料、PLANT 部署门禁）与 placer 取名见 [[reference/prefabs/grass-saplings#公共厂与调用链]]，本页不重复。
- 重要副作用：挖起不保证给移栽件。尖刺灌木无论状态都给 `dug_marsh_bush`，另在可采时补 `twigs`；石果灌木丛在高温枯萎或枯死时只给 2 根 `twigs`；猴尾草在枯萎时把移栽件就地换成 `cutreeds`。回种后 `ondeploy` 调 `pickable:OnTransplant()`：尖刺灌木变空株（`MakeEmpty`，4 天后再生），石果灌木丛与猴尾草变枯死（`MakeBarren`，需施肥）。[尖刺灌木挖起](../../../raw/current/scripts/prefabs/marsh_bush.lua#L28-L34)、[石果丛挖起](../../../raw/current/scripts/prefabs/rock_avocado_bush.lua#L142-L163)、[猴尾草挖起](../../../raw/current/scripts/prefabs/monkeytail.lua#L14-L25)
- 修改边界：蜡封件 `dug_<name>_waxed` 只给入口与命名，细节归 B14／B43；石果灌木丛的阶段与掉落留在 [[reference/prefabs/rock-avocado-fruit]]，浆果丛与香蕉丛的挖起对照留在 [[reference/prefabs/berry-bushes]]；三件都没有 `halloweenmoonmutable_settings`，三个世界态也都没有 `lunarplant_target` 标签，所以公共厂里的月岛植物登记分支对它们不成立。

## 注册与构造

三件都由同一个 `make_plantable(data)` 生成：函数末行固定拼接 `Prefab("dug_"..data.name, fn, assets)`，注册名不能从文件名反推；bank／build 分别取 `data.bank or data.name`／`data.build or data.name`，即**回落到数据名**，注册名只是数据名前加 `dug_`；placer 同样按这三项生成。[注册](../../../raw/current/scripts/prefabs/plantables.lua#L116)、[取法与 placer](../../../raw/current/scripts/prefabs/plantables.lua#L6-L15)、[placer](../../../raw/current/scripts/prefabs/plantables.lua#L188)

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `dug_marsh_bush`／`dug_marsh_bush_placer` | 数据表项 `marsh_bush` → 公共厂：资产 → 客户端段 → `SetPristine` → 主模拟段；placer 动画参数走缺省 `idle` | bank 与 build 都回落到数据名 `marsh_bush`，只加载 `anim/marsh_bush.zip` 一份动画；`mediumspacing` 为真；图鉴提示 `PLANTABLE`。[数据项](../../../raw/current/scripts/prefabs/plantables.lua#L157-L159) |
| `dug_rock_avocado_bush`／`dug_rock_avocado_bush_placer` | 同上，bank／build／placer 动画全部由数据表覆盖 | `bank = "rock_avocado"`、`build = "rock_avocado_build"`、`anim = "dead1"`（只作用于 placer 预览）；因此额外加载 `anim/rock_avocado_build.zip`；`inspectoverride = "rock_avocado_bush"`；floater 第二项为 `nil`；无 `mediumspacing`。[数据项](../../../raw/current/scripts/prefabs/plantables.lua#L162-L167) |
| `dug_monkeytail`／`dug_monkeytail_placer` | 同上，bank／build 覆盖，placer 动画缺省 `idle` | `bank = "grass"`、`build = "reeds_monkeytails"`，所以加载 `anim/grass.zip`＋`anim/reeds_monkeytails.zip`；`mediumspacing` 为真；图鉴提示 `PLANTABLE_FERTILIZE`。[数据项](../../../raw/current/scripts/prefabs/plantables.lua#L175-L179) |

### 公共厂的剩余分支

1. **资产**：`anim/<bank>.zip` 与 `INV_IMAGE dug_<name>`；`data.build` 存在时再追加 `anim/<build>.zip`。所以只有尖刺灌木是单份动画，另外两件各两份。[资产段](../../../raw/current/scripts/prefabs/plantables.lua#L7-L15)
2. **客户端段**：加 `deployedplant` 标签，`bank`／`build` 按同一覆盖回退，播 `dropped` 并把 `scrapbook_anim` 也设成 `dropped`；浮水外观用 `swap_data = {bank, anim = "dropped"}` 传入 `MakeInventoryFloatable`，前三项取 `data.floater`。[客户端段](../../../raw/current/scripts/prefabs/plantables.lua#L48-L60)
3. **图鉴分流**：按 `data.name` 硬编码两组。尖刺灌木落在 `PLANTABLE`（"Can be planted in the ground."），石果灌木丛与猴尾草落在 `PLANTABLE_FERTILIZE`（"This needs some fertilizing after planting."）。[分流](../../../raw/current/scripts/prefabs/plantables.lua#L62-L76)、[文本组](../../../raw/current/scripts/strings.lua#L18960)、[文本组](../../../raw/current/scripts/strings.lua#L18843)
4. **主模拟段**：`stackable.maxsize = TUNING.STACK_SIZE_LARGEITEM`（10）；`inspectable.nameoverride = data.inspectoverride or ("dug_"..data.name)`；`inventoryitem` 不写 `imagename`，客户端按注册名取 `INV_IMAGE dug_<name>`；`fuel.fuelvalue = TUNING.LARGE_FUEL`（180 秒）；`MakeMediumBurnable(inst, TUNING.LARGE_BURNTIME)`（燃尽 30 秒）；`MakeSmallPropagator`；`MakeHauntableLaunchAndIgnite`。[组件段](../../../raw/current/scripts/prefabs/plantables.lua#L84-L97)、[堆叠档](../../../raw/current/scripts/tuning.lua#L81-L82)、[燃料与燃尽](../../../raw/current/scripts/tuning.lua#L1673-L1683)
5. **部署**：`deployable.ondeploy = ondeploy`、`SetDeployMode(DEPLOYMODE.PLANT)`；`data.mediumspacing` 为真时再 `SetDeploySpacing(DEPLOYSPACING.MEDIUM)`，所以石果灌木丛是 DEFAULT（半径 2），另两件是 MEDIUM（半径 1）。[部署段](../../../raw/current/scripts/prefabs/plantables.lua#L99-L105)、[间距枚举](../../../raw/current/scripts/constants.lua#L2236-L2263)
6. **没有的部分与消耗三层**：三件都不挂 `halloweenmoonmutable`（数据表无该字段），也不带 `deployedfarmplant` 之类的额外标签。消耗要分三层记——①**回调侧**：`ondeploy` 只在 `SpawnPrefab` 返回非空时对 `stackable:Get()` 得到的 1 个实体 `Remove`，返回空则回调不动手；②**动作侧**：`ACTIONS.DEPLOY.fn` 在调 `Deploy` 之前已用 `container:RemoveItem` 从栏里取出 1 件并清掉 `prevcontainer`／`prevslot`，只有 `Deploy` 返回失败才 `GiveItem` 放回；③**结果侧**：`ondeploy` 没有失败返回值，`Deploy` 照样推 `deployitem`（带 `deployedplant` 时再推 `itemplanted`）并返回 `true`。所以“回调没销毁单体”不等于“物品还在栏里”，也不等于“动作失败”；正常生成时净消耗 1 件（成叠时栏里余 N−1）。该门禁与浆果丛同一段代码。[回种](../../../raw/current/scripts/prefabs/plantables.lua#L17-L36)、[动作](../../../raw/current/scripts/actions.lua#L1608-L1638)、[执行](../../../raw/current/scripts/components/deployable.lua#L132-L147)、[取出 1 件](../../../raw/current/scripts/components/inventory.lua#L1390-L1403)

### 三件的数据差异

| 维度 | `dug_marsh_bush` | `dug_rock_avocado_bush` | `dug_monkeytail` |
|---|---|---|---|
| 数据表项 | `marsh_bush` | `rock_avocado_bush` | `monkeytail` |
| bank／build | 回落 `marsh_bush`／`marsh_bush` | `rock_avocado`／`rock_avocado_build` | `grass`／`reeds_monkeytails` |
| placer 动画 | `idle`（缺省） | `dead1` | `idle`（缺省） |
| floater | `{"med", 0.1, 0.9}` | `{"med", nil, 0.95}`，垂直偏移取默认 | `{"med", 0.1, 0.92}` |
| `mediumspacing` | 是（MEDIUM） | 否（DEFAULT） | 是（MEDIUM） |
| `inspectoverride` | 无，回落 `dug_marsh_bush` | `rock_avocado_bush` | 无，回落 `dug_monkeytail` |
| `scrapbook_specialinfo` | `PLANTABLE` | `PLANTABLE_FERTILIZE` | `PLANTABLE_FERTILIZE` |
| 挖起来源 | `marsh_bush` | `rock_avocado_bush` | `monkeytail` |
| 移植回调 | `MakeEmpty` | `MakeBarren` | `MakeBarren` |

差异全部落在数据表，工厂代码不按名字分叉；唯一按名字判断的仍是图鉴分流。[数据表](../../../raw/current/scripts/prefabs/plantables.lua#L119-L182)

## 与已验收页的分工

- [[reference/prefabs/grass-saplings]] 已核验 `make_plantable` 的完整公共厂、placer 取名链路、PLANT 落点门禁、燃料与堆叠数值、蜡封指针；本页只补充三项的数据差异与三个世界态的挖起端，不重复推导门禁。
- [[reference/prefabs/berry-bushes]] 的“高温枯萎或枯死只给 twigs”只适用于浆果丛／香蕉丛；尖刺灌木相反（见下节），石果灌木丛与浆果丛同侧但多一条 `stage_3` 的三果分支。
- 世界态本身归 [[reference/prefabs/rock-avocado-fruit]]（石果丛四阶段与采掘）与本页（尖刺灌木、猴尾草及三者的挖起／回种对比）。

## 世界态来源与挖起

| 世界态 | pickable | 挖起（`workable` DIG 1 次） | 移植回调与后续 |
|---|---|---|---|
| `marsh_bush` | `SetUp("twigs", 4 天)`，无 `max_cycles`，从不枯死 | 可采时先补 1 个 `twigs`，**无条件**再掉 `dug_marsh_bush` | `MakeEmpty`：空株并排 4 天再生，不需要施肥 |
| `rock_avocado_bush` | 四阶段 `growable` + `numtoharvest = 3`；`max_cycles = cycles_left = 3` | 枯萎或枯死只掉 2 根 `twigs`；否则 `stage_3` 时先喷 3 个石果，再掉 `dug_rock_avocado_bush` | `MakeBarren`：`growable` 停在第 1 段，需施肥 |
| `monkeytail` | `SetUp("cutreeds", 3 天)`；`max_cycles = 4 + random(3)`（5–7） | 可采时先补 `cutreeds`，再掉 `withered and "cutreeds" or "dug_monkeytail"` | `MakeBarren`：需施肥，施肥后 `MakeEmpty` 重排 3 天再生 |

- **尖刺灌木永不枯死**：构造只设 `pickable.SetUp("twigs", TUNING.MARSHBUSH_REGROW_TIME)`，从不写 `max_cycles`／`cycles_left`；`IsBarren()` 判 `cycles_left == 0`，`nil` 不等于 0，因此既不会进 `barren`，也没有 `witherable` 造成的 `withered`，`CanBeFertilized` 恒为假。[接线](../../../raw/current/scripts/prefabs/marsh_bush.lua#L102-L109)、[判定](../../../raw/current/scripts/components/pickable.lua#L378-L380)、[可施肥](../../../raw/current/scripts/components/pickable.lua#L251-L253)
- **尖刺灌木的挖起不受移植模式门禁**：另外两件把 `workable` 包在 `not GetGameModeProperty("disable_transplanting")` 里，尖刺灌木不加判断，该模式世界里仍能挖起它。[无门禁](../../../raw/current/scripts/prefabs/marsh_bush.lua#L111-L115)、[石果丛门禁](../../../raw/current/scripts/prefabs/rock_avocado_bush.lua#L280-L285)、[猴尾草门禁](../../../raw/current/scripts/prefabs/monkeytail.lua#L134-L139)
- **猴尾草枯萎即失去移栽件**：挖起回调的第二段是三元表达式，`withered` 为真时掉的是 `cutreeds` 而不是 `dug_monkeytail`；此时 `pickable.product` 仍是 `cutreeds`，所以“可采才补产物”的分支与替换无关。[挖起](../../../raw/current/scripts/prefabs/monkeytail.lua#L14-L25)
- **石果灌木丛的挖起与回种**：挖起分支与 [[reference/prefabs/berry-bushes#采摘与再生]] 的浆果丛同侧（枯萎／枯死不给移栽件），差异是 `stage_3` 额外喷 3 个石果；回种后 `ontransplantfn` 只调 `MakeBarren()`，而它注册了 `makebarrenfn`，于是 `growable` 退回第 1 段并停下。[挖起](../../../raw/current/scripts/prefabs/rock_avocado_bush.lua#L142-L163)、[移植](../../../raw/current/scripts/prefabs/rock_avocado_bush.lua#L202-L217)
- **施肥重置**：`Fertilize` 把 `cycles_left` 写回 `max_cycles`；无 `witherable` 的尖刺灌木直接走 `MakeEmpty`，有 `witherable` 的两件在未枯萎时也走 `MakeEmpty`，枯萎时改走 `ForceRejuvenate`。[施肥](../../../raw/current/scripts/components/pickable.lua#L259-L292)
- **其它来源**：`dug_monkeytail` 另有两处——Wormwood 技能配方 `wormwood_reeds`（`builder_skill = "wormwood_reedscrafting"`）与猴子岛传送门的加权／事件掉落；`dug_marsh_bush` 与 `dug_rock_avocado_bush` 在本版脚本里只有挖起一处生成点。[配方](../../../raw/current/scripts/recipes.lua#L347)、[传送门权重](../../../raw/current/scripts/prefabs/monkeyisland_portal.lua#L33-L53)
- **冬季与烧毁**：只有猴尾草挂 `MakeNoGrowInWinter`。尖刺灌木用 `MakeLargeBurnable`（默认 30 秒）且不设自定义燃尽回调，烧完走 `DefaultBurntFn`——陆上生成 `ash` 并删除本体，不留残桩。`burnt_marsh_bush`／`burnt_marsh_bush_erode` 是另两个注册：前者只在龙蝇竞技场静态布局里以 `type = "burnt_marsh_bush"` 预置（`forest.lua` 里只是依赖声明），后者是它的消散表现件，两者都与烧毁路径无关。石果灌木丛用自定义 `on_bush_burnt`：`stage_3` 先喷 3 个石果、`StopGrowing`，最后再调 `DefaultBurntFn`（陆上另掉 `ash` 并删除）。[猴尾草](../../../raw/current/scripts/prefabs/monkeytail.lua#L142-L148)、[尖刺灌木可燃](../../../raw/current/scripts/prefabs/marsh_bush.lua#L119-L121)、[默认燃尽](../../../raw/current/scripts/standardcomponents.lua#L18-L44)、[竞技场预置](../../../raw/current/scripts/map/static_layouts/dragonfly_arena.lua#L623-L624)、[依赖声明](../../../raw/current/scripts/prefabs/forest.lua#L262)、[石果丛烧毁](../../../raw/current/scripts/prefabs/rock_avocado_bush.lua#L124-L136)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：pickable | 挖出件不挂；世界态用它定义产物、再生与 `OnTransplant` 落点 | 世界态，主模拟 | [[reference/components/pickable#关键接口]]；[SetUp](../../../raw/current/scripts/prefabs/marsh_bush.lua#L102-L105) |
| 组件：deployable | PLANT 模式；只有石果灌木丛用 DEFAULT 间距 | 挖出件；回调未生成时动作仍成功、不返还已拆出的一件，只有 `Deploy` 返回 false 才 `GiveItem` | [[reference/components/deployable#关键接口]]；[部署段](../../../raw/current/scripts/prefabs/plantables.lua#L99-L105) |
| 组件：stackable | 上限 10；放置时取 1 个 | 挖出件，成叠时每次只消耗 1 | [[reference/components/stackable#关键接口]]；[取 1 个](../../../raw/current/scripts/prefabs/plantables.lua#L21) |
| 组件：fuel／burnable／propagator／hauntable | 燃料 180 秒、燃尽 30 秒、蔓延、作祟只弹飞 | 挖出件，主模拟 | [[reference/components/fuel#关键接口]]、[[reference/components/burnable#关键接口]]；[组件段](../../../raw/current/scripts/prefabs/plantables.lua#L91-L97) |
| 组件：inspectable | `nameoverride` 决定检查台词取键，不改 HUD 名 | 挖出件；石果灌木丛指向世界态名 | [[reference/components/inspectable#关键接口]]；[赋值](../../../raw/current/scripts/prefabs/plantables.lua#L87-L88)、[取名](../../../raw/current/scripts/stringutil.lua#L375) |
| 组件：witherable | 高温枯萎 → `MakeBarren`；恢复先写回轮次 | 石果灌木丛与猴尾草；尖刺灌木不挂 | [[reference/components/witherable#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/rock_avocado_bush.lua#L299-L302) |
| 组件：workable | DIG 1 次挖起；尖刺灌木不受 `disable_transplanting` 门禁 | 世界态 | [[reference/components/workable#关键接口]]；[无门禁](../../../raw/current/scripts/prefabs/marsh_bush.lua#L111-L115) |
| 组件：growable | 石果灌木丛四阶段；`SetStage(1)` 归零重长 | 仅石果灌木丛 | [[reference/components/growable#关键接口]]；[阶段表](../../../raw/current/scripts/prefabs/rock_avocado_bush.lua#L89-L115) |
| API／数据：`MakePlacer` | 生成三个 placer，动画取 `data.anim or "idle"` | 客户端预览实体 | [[reference/components/placer#关键接口]]；[placer](../../../raw/current/scripts/prefabs/plantables.lua#L188) |
| API／数据：`DEPLOYMODE`／`DEPLOYSPACING` | PLANT 模式；MEDIUM 半径 1、DEFAULT 半径 2 | 主模拟设值，replica 同步 | [枚举](../../../raw/current/scripts/constants.lua#L2214-L2226)、[半径](../../../raw/current/scripts/constants.lua#L2254-L2263) |
| 关联 Prefab：`burnt_marsh_bush`／`burnt_marsh_bush_erode` | 龙蝇竞技场静态布局预置的焦黑残桩（非烧毁产物）与它的消散表现件 | 世界态；残桩被激活后消散，侵蚀 FX 0.5 秒自删 | [残桩](../../../raw/current/scripts/prefabs/marsh_bush.lua#L132-L167)、[消隐](../../../raw/current/scripts/prefabs/marsh_bush.lua#L194-L223)、[预置](../../../raw/current/scripts/map/static_layouts/dragonfly_arena.lua#L623-L624) |
| 关联 Prefab：`rock_avocado_fruit` | 石果灌木丛 `stage_3` 的产物；挖起时也喷 3 个 | 物品 | [[reference/prefabs/rock-avocado-fruit]]；[挖起](../../../raw/current/scripts/prefabs/rock_avocado_bush.lua#L150-L159) |
| 关联 Prefab：`cutreeds` | 猴尾草产物，也是枯萎时的挖起替身 | 物品 | [[reference/prefabs/basic-gathered-materials]]；[挖起](../../../raw/current/scripts/prefabs/monkeytail.lua#L22) |
| 关联 Prefab：`dug_<name>_waxed` | 蜡封挖出件，复用同一 placer 与 `savedata` | 本页只给指针，细节归 B14／B43 | [工厂](../../../raw/current/scripts/prefabs/waxed_plant_common.lua#L440-L470) |
| 资源：anim 包 | `anim/marsh_bush.zip`；`anim/rock_avocado.zip`＋`anim/rock_avocado_build.zip`；`anim/grass.zip`＋`anim/reeds_monkeytails.zip` | 客户端段加载 | [资产](../../../raw/current/scripts/prefabs/plantables.lua#L7-L15) |

## 典型接入

### 让枯萎的猴尾草挖起后仍返还移栽件

- 前提与选择：服务端 `modmain.lua`。原版 `dig_up` 是工厂内的 `local` 函数，不能直接替换；但它只通过 `workable:SetOnFinishCallback` 挂到实体上，PostInit 里重设该回调即可，影响面限于猴尾草这一种植物。
- 生效与撤销：`AddPrefabPostInit` 在每次构造（含读档重建）后执行；停用模组即恢复原回调。`workable` 本身仍受 `disable_transplanting` 门禁，该模式下没有挖起入口，替换回调不会创造新入口。
- 原版依据：默认回调的两段与枯萎替换见[世界态来源与挖起](#世界态来源与挖起)；`pickable.product` 在枯萎时仍是 `cutreeds`，所以“可采才补产物”那段保持原样即可。[原回调](../../../raw/current/scripts/prefabs/monkeytail.lua#L14-L25)

```lua
-- modmain.lua：枯萎的猴尾草挖起后仍掉移栽件，不再被 cutreeds 顶替
AddPrefabPostInit("monkeytail", function(inst)
    if not GLOBAL.TheWorld.ismastersim or inst.components.workable == nil then
        return
    end
    inst.components.workable:SetOnFinishCallback(function(plant, worker)
        local pickable = plant.components.pickable
        local lootdropper = plant.components.lootdropper
        if pickable ~= nil and lootdropper ~= nil then
            if pickable:CanBePicked() then
                lootdropper:SpawnLootPrefab(pickable.product)
            end
            lootdropper:SpawnLootPrefab("dug_monkeytail")
        end
        plant:Remove()
    end)
end)
```

- 调整与验证：工作量单位是 `SetWorkLeft(1)` 的 1 次 DIG；替换点在 `workable` 的完成回调，不在 `pickable`。若还想让回种后不枯死，可把世界态 `pickable.ontransplantfn` 换成 `MakeEmpty`（同 [[reference/prefabs/grass-saplings#典型接入]] 的做法）；两者叠加时轮次仍按 `transplanted` 递减，扣到 0 仍会枯死。

## 常见陷阱与限制

- **误区：“这三种灌木挖起都必给移栽件”**：猴尾草枯萎时换成 `cutreeds`，石果灌木丛枯萎／枯死时只给 2 根 `twigs`；只有尖刺灌木无条件给。[分支](../../../raw/current/scripts/prefabs/monkeytail.lua#L22)、[分支](../../../raw/current/scripts/prefabs/rock_avocado_bush.lua#L145-L148)、[分支](../../../raw/current/scripts/prefabs/marsh_bush.lua#L28-L34)
- **误区：“尖刺灌木也会枯死、也要施肥”**：它没有 `max_cycles`／`cycles_left`，也没有 `witherable`，`IsBarren()` 与 `CanBeFertilized()` 都不会为真；移栽后只是空株，4 天后自然再长。[构造](../../../raw/current/scripts/prefabs/marsh_bush.lua#L102-L109)、[判定](../../../raw/current/scripts/components/pickable.lua#L378-L380)
- **误区：“作祟会点燃这些物品”**：三件挖出件走 `MakeHauntableLaunchAndIgnite`，但函数里的点火分支在 `#HAUNTFIX` 注释下停用，实际只 `Launch` 并返回 `false`；尖刺灌木本体用的 `MakeHauntableIgnite` 回调同样整段被注释、恒返回 false，但它在装配时仍把 `hauntable.cooldown` 设为 `TUNING.HAUNT_COOLDOWN_MEDIUM`（5 秒），所以作祟会进入冷却却不点燃。[标准组件](../../../raw/current/scripts/standardcomponents.lua#L1191-L1217)、[空回调](../../../raw/current/scripts/standardcomponents.lua#L1173-L1189)、[冷却常量](../../../raw/current/scripts/tuning.lua#L3044-L3045)
- **边界：`inspectoverride` 不是显示名**：`dug_rock_avocado_bush` 的 `inspectable.nameoverride = "rock_avocado_bush"` 只让检查台词取世界态的键；HUD 名仍按注册名取 `STRINGS.NAMES.DUG_ROCK_AVOCADO_BUSH`（石果灌木丛），所以两者都显示“石果灌木丛”。[取名](../../../raw/current/scripts/stringutil.lua#L375)、[显示名](../../../raw/current/scripts/entityscript.lua#L687-L691)
- **边界：`dug_marsh_bush` 与 `marsh_bush` 同名**：HUD 与检查台词都是“尖刺灌木”，只能靠 `deployedplant`、`deployable` 与堆叠区分物品形态。[名称](#验证记录)、[标签](../../../raw/current/scripts/prefabs/plantables.lua#L48-L53)
- **未核验**：蜡封件的存档与外观（B14／B43）、`burnt_marsh_bush` 被激活后的交互细节、猴子岛传送门的完整刷新周期与权重取值。

## 验证记录

- 源码审阅：2026-11-10，Steam buildid 26474873。逐行读完 `plantables.lua`（公共厂全部分支、数据表 10 项、注册循环）、`marsh_bush.lua`（挖起、采摘伤害、再生／空株回调、烧毁与两个附属注册）、`rock_avocado_bush.lua`（阶段表、挖起与烧毁、`MakeEmpty`／`MakeBarren`、读档补正）、`monkeytail.lua`（挖起、轮次、冬季与三类动画回调）；核验 `pickable` 的 `SetUp`／`IsBarren`／`MakeBarren`／`MakeEmpty`／`OnTransplant`／`Fertilize`、`deployable` 的 PLANT 与间距同步、`stackable:Get`、`MakeInventoryFloatable`／`MakeMediumBurnable`／`MakeSmallPropagator`／`MakeHauntableLaunchAndIgnite`、tuning 常量换算，以及两项旁支来源（Wormwood 配方、猴子岛传送门）。
- 名称核验（`inspect_prefab_names.py`，只认 `languages/chinese_s.po` 的 msgstr）：`MARSH_BUSH`／`DUG_MARSH_BUSH` 都是“尖刺灌木”，`ROCK_AVOCADO_BUSH`／`DUG_ROCK_AVOCADO_BUSH` 都是“石果灌木丛”，`MONKEYTAIL`／`DUG_MONKEYTAIL` 都是“猴尾草”。[MARSH_BUSH](../../../raw/current/scripts/languages/chinese_s.po#L345050-L345052)、[DUG_MARSH_BUSH](../../../raw/current/scripts/languages/chinese_s.po#L342330-L342332)、[ROCK_AVOCADO_BUSH](../../../raw/current/scripts/languages/chinese_s.po#L348705-L348707)、[DUG_ROCK_AVOCADO_BUSH](../../../raw/current/scripts/languages/chinese_s.po#L342340-L342342)、[MONKEYTAIL](../../../raw/current/scripts/languages/chinese_s.po#L345570-L345572)、[DUG_MONKEYTAIL](../../../raw/current/scripts/languages/chinese_s.po#L342335-L342337)
- 游戏运行：未运行。天数与秒数为 tuning 推导（4 天＝1920 秒、3 天＝1440 秒；换算基准见 `seg_time = 30` 与 `total_day_time = 480`；`LARGE_FUEL`＝180 秒、`LARGE_BURNTIME`＝30 秒）；挖起分支与移植后果为代码路径推导。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 结果见本批报告。

## 相关链接

- 公共厂与普通移栽切片：[[reference/prefabs/grass-saplings]]
- 对照：[[reference/prefabs/berry-bushes]]、[[reference/prefabs/rock-avocado-fruit]]
- 组件：[[reference/components/pickable]]、[[reference/components/deployable]]、[[reference/components/witherable]]、[[reference/components/growable]]、[[reference/components/workable]]、[[reference/components/waxable]]

返回 [[reference/prefabs/index|Prefab 索引]]。
