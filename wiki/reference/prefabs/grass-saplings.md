---
title: "草丛与树苗的移植件（dug_grass、dug_sapling、dug_sapling_moon）"
created: "2026-11-08"
updated: "2026-11-11"
type: prefab
summary: "plantables.lua 的 make_plantable 公共厂：草丛、树苗、月树苗挖出件的资产与客户端段、主模拟组件、PLANT 部署与 ondeploy 回种，以及三者的数据差异和草、树苗世界态的挖起与再生。"
aliases: ["dug_grass", "dug_grass_placer", "dug_sapling", "dug_sapling_placer", "dug_sapling_moon", "dug_sapling_moon_placer", "草丛", "树苗", "移植草丛", "挖出的树苗", "grass", "sapling", "sapling_moon"]
tags: [prefab, item]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/plantables.lua", "prefabs/grass.lua", "prefabs/sapling.lua", "prefabs/monkeytail.lua", "prefabs/waxed_plant_common.lua", "prefabutil.lua", "components/pickable.lua", "components/deployable.lua", "components/inventoryitem_replica.lua", "components/stackable.lua", "components/halloweenmoonmutable.lua", "components/halloweenpotionmoon.lua", "components/lunarthrall_plantspawner.lua", "components/witherable.lua", "components/map.lua", "standardcomponents.lua", "tuning.lua", "constants.lua", "strings.lua", "entityscript.lua", "stringutil.lua", "map/forest_map.lua", "map/rooms/forest/terrain_moonisland.lua", "recipes.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 草丛与树苗的移植件（dug_grass、dug_sapling、dug_sapling_moon）

## 快速判断

- 职责与适用：`prefabs/plantables.lua` 的 `make_plantable(data)` 为数据表每项产出 1 个挖出件 `dug_<name>`，注册循环再为每项配 1 个 placer 与 1 个蜡封件。本页负责 `dug_grass`、`dug_grass_placer`、`dug_sapling`、`dug_sapling_placer`、`dug_sapling_moon`、`dug_sapling_moon_placer` 共 6 个注册；世界态 `grass`／`sapling`／`sapling_moon` 只作来源与挖起端。[工厂](../../../raw/current/scripts/prefabs/plantables.lua#L5-L14)、[注册循环](../../../raw/current/scripts/prefabs/plantables.lua#L186-L191)、[文件导出](../../../raw/current/scripts/prefabs/plantables.lua#L193)
- 优先使用：改挖出件的资产／组件／燃料看[公共厂与调用链](#公共厂与调用链)；改放置间距与地块条件看[放置器与放置校验](#放置器与放置校验)；改世界态再生、挖起产物与移植后果看[世界态来源与挖起](#世界态来源与挖起)。挖出件是普通 `inventoryitem`，组件语义见 [[reference/components/pickable#关键接口]] 与 [[reference/components/deployable#关键接口]]。
- 重要副作用：放置时 `SpawnPrefab` 同名世界态、从堆叠里取 1 个 `Remove`，再调世界态的 `pickable:OnTransplant()`；草丛因此立刻进入枯死（barren）态（`cycles_left` 归零、需施肥），树苗只回到空株并重新计时。移植的草丛只起 25±5 天的 `morphdelay`，到期也不会自己变草蜥（见陷阱）。[ondeploy](../../../raw/current/scripts/prefabs/plantables.lua#L17-L36)、[移植入口](../../../raw/current/scripts/components/pickable.lua#L418-L424)
- 修改边界：`cutgrass`／`twigs` 的物品数值与采集／挖起这条主来源留在 [[reference/prefabs/basic-gathered-materials]]，风滚草、草蜥等旁支来源不在本页补列；蜡封挖出件 `dug_<name>_waxed`（含本页三件的蜡封形态）归 B14／B43 批次，本页只在[蜡封形态](#蜡封形态)给指针；`marsh_bush`、`monkeytail`、`rock_avocado_bush` 的挖出形态归 B14。

## 注册与构造

六个注册全部出自同一份数据表与同一个工厂：`for _, data in ipairs(plantables)` 每轮插入 `make_plantable(data)`、`MakePlacer("dug_"..data.name.."_placer", ...)`、`WAXED_PLANTS.CreateDugWaxedPlant(data)` 三件。[循环](../../../raw/current/scripts/prefabs/plantables.lua#L186-L191)

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `dug_grass`／`dug_grass_placer` | `make_plantable`：资产 → 客户端段 → `SetPristine` → 主模拟段；placer 另由 `MakePlacer(..., "grass", "grass1", "idle")` 注册 | build 被数据表覆盖为 `grass1`，bank 仍取注册名 `grass`；`mediumspacing` 为真。[数据项](../../../raw/current/scripts/prefabs/plantables.lua#L151-L154) |
| `dug_sapling`／`dug_sapling_placer` | 同上，bank／build 均为 `sapling` | 多挂 `halloweenmoonmutable`，突变目标 `dug_sapling_moon`。[数据项](../../../raw/current/scripts/prefabs/plantables.lua#L139-L142) |
| `dug_sapling_moon`／`dug_sapling_moon_placer` | 同上，bank／build 均为 `sapling_moon` | `inspectoverride = "dug_sapling"`，即显示名借用树苗键；无 `halloweenmoonmutable`。[数据项](../../../raw/current/scripts/prefabs/plantables.lua#L145-L148) |

注册行本身只决定 `dug_<name>`：工厂末行是 `Prefab("dug_"..data.name, fn, assets)`，所以世界态名与挖出件名的对应关系是固定拼接，不能从文件名反推。[注册](../../../raw/current/scripts/prefabs/plantables.lua#L116)

### 公共厂与调用链

`make_plantable(data)` 是后续轮次复用的公共厂：先闭包 `ondeploy`，再返回 `fn`，两者共用同一份 `data`。[工厂入口](../../../raw/current/scripts/prefabs/plantables.lua#L5-L14)

1. **资产**：`anim/<bank>.zip`（bank 缺省取 `data.name`）与 `INV_IMAGE dug_<name>`；`data.build` 存在时再追加 `anim/<build>.zip`。草丛因此是 `anim/grass.zip` ＋ `anim/grass1.zip` 两份动画。[资产段](../../../raw/current/scripts/prefabs/plantables.lua#L7-L15)
2. **客户端段（`SetPristine` 前）**：`AddTag("deployedplant")`；`SetBank(data.bank or data.name)`、`SetBuild(data.build or data.name)`、播 `dropped`，并把 `scrapbook_anim` 也设为 `dropped`；浮水外观用 `swap_data = {bank, anim="dropped"}` 传入 `MakeInventoryFloatable`，参数按 `size, y_offset, scale` 取 `data.floater` 前三项（缺省时 size 回落 `"small"`）。[标签与动画](../../../raw/current/scripts/prefabs/plantables.lua#L48-L53)、[浮水](../../../raw/current/scripts/prefabs/plantables.lua#L55-L60)、[浮水签名](../../../raw/current/scripts/standardcomponents.lua#L1772-L1791)
3. **图鉴分流**：按 `data.name` 硬编码两组。`PLANTABLE_FERTILIZE`（"This needs some fertilizing after planting."）给 `grass`／`berrybush`／`berrybush2`／`berrybush_juicy`／`monkeytail`／`bananabush`／`rock_avocado_bush`；`PLANTABLE`（"Can be planted in the ground."）给 `sapling`／`sapling_moon`／`marsh_bush`。两者文本分别在 `STRINGS.SCRAPBOOK.SPECIALINFO` 的对应键。[分流](../../../raw/current/scripts/prefabs/plantables.lua#L62-L76)、[文本](../../../raw/current/scripts/strings.lua#L18843)、[文本](../../../raw/current/scripts/strings.lua#L18960)
4. **主模拟段**：`stackable.maxsize = TUNING.STACK_SIZE_LARGEITEM`（10）；`inspectable.nameoverride = data.inspectoverride or ("dug_"..data.name)`；`inventoryitem`（不写 `imagename`，客户端按 `inst.prefab..".tex"` 取图，正好对上 `INV_IMAGE dug_<name>`）；`fuel.fuelvalue = TUNING.LARGE_FUEL`（180 秒）；`MakeMediumBurnable(inst, TUNING.LARGE_BURNTIME)`（燃尽 30 秒）；`MakeSmallPropagator`；`MakeHauntableLaunchAndIgnite`。[组件段](../../../raw/current/scripts/prefabs/plantables.lua#L84-L97)、[堆叠档](../../../raw/current/scripts/tuning.lua#L81)、[燃料与燃尽](../../../raw/current/scripts/tuning.lua#L1677)、[图像默认值](../../../raw/current/scripts/components/inventoryitem_replica.lua#L151-L157)
5. **部署**：`deployable.ondeploy = ondeploy`，`SetDeployMode(DEPLOYMODE.PLANT)`；`data.mediumspacing` 为真时再 `SetDeploySpacing(DEPLOYSPACING.MEDIUM)`。间距写入经 `deployable.spacing` 的属性监听同步到 replica 的 `deployspacing`。[部署段](../../../raw/current/scripts/prefabs/plantables.lua#L99-L105)、[模式与间距](../../../raw/current/scripts/components/deployable.lua#L64-L82)
6. **可选变异**：`data.halloweenmoonmutable_settings.prefab` 存在时挂 `halloweenmoonmutable` 并 `SetPrefabMutated`；本页只有 `sapling` 命中（`dug_sapling_moon`）。[变异段](../../../raw/current/scripts/prefabs/plantables.lua#L107-L110)
7. **ondeploy**：先 `SpawnPrefab(data.name, inst.linked_skinname, inst.skin_id)`，**只有返回非空**才 `Transform:SetPosition(pt:Get())` → `inst.components.stackable:Get():Remove()` → 世界态有 `pickable` 时 `pickable:OnTransplant()` → `deployer.SoundEmitter:PlaySound("dontstarve/common/plant")`（注释说明多数植物自身没有 SoundEmitter）→ 世界态带 `lunarplant_target` 且 `TheWorld.components.lunarthrall_plantspawner` 存在时 `setHerdsOnPlantable(tree)`；返回空则整段跳过、回调不销毁已取出的单体；但玩家 DEPLOY 已在进回调前从物品栏拆出 1 件，只有 Deploy 返回失败才 GiveItem，所以动作仍算成功并照推 deployitem／itemplanted。[ondeploy](../../../raw/current/scripts/prefabs/plantables.lua#L17-L36)、[取 1 个](../../../raw/current/scripts/components/stackable.lua#L109-L120)、[herdmember 登记](../../../raw/current/scripts/components/lunarthrall_plantspawner.lua#L529-L533)

`deployedplant` 标签在 `Deployable:Deploy` 里被读一次：有该标签时额外向世界推 `itemplanted`，所以挖出件与普通部署件的区别不只是外观。`Deploy` 只回报 `CanDeploy` 的结果、不检查 `ondeploy` 的返回值，`deployitem`／`itemplanted` 仍照推，所以“种植成功”的提示与事件都不能反推世界态真的生成。[部署执行](../../../raw/current/scripts/components/deployable.lua#L132-L147)

### 放置器与放置校验

- **placer 名字来源**：客户端用 `inventoryitem_replica:GetDeployPlacerName()` 取 `inst.prefab.."_placer"`（除非 `overridedeployplacername` 或 `usegridplacer`），所以 `dug_grass` 必然找 `dug_grass_placer`。[取名](../../../raw/current/scripts/components/inventoryitem_replica.lua#L360-L369)
- **placer 本体**：`MakePlacer` 造的是 `CLASSIFIED`／`NOCLICK`／`placer`、`persists = false` 的非联网预览实体，只按传入的 bank／build／anim 播放并挂 `placer` 组件。[工厂](../../../raw/current/scripts/prefabutil.lua#L1-L65)、[标签与不持久](../../../raw/current/scripts/prefabutil.lua#L5-L10)
- **动画参数**：循环把 `data.anim or "idle"` 传给 placer；`data.anim` 在 `fn` 里完全不被读取，挖出件本体固定播 `dropped`，所以该字段只影响 placer 预览。本页三件都没写 `anim`，传入的是字面量 `idle`。[placer 参数](../../../raw/current/scripts/prefabs/plantables.lua#L188)
- **能否落地**：先过 `Deployable:IsDeployable`——`restrictedtag` 不符直接否决；骑乘（`rider:IsRiding()`）时只允许带 `complexprojectile` 的物件，手持浮标（`inventory:IsFloaterHeld()`）时只允许带 `boatbuilder` tag 的物件，本页三件两者都不满足，所以骑乘或手持浮标时不能种植。通过后才进 `DEPLOYMODE.PLANT` 的 `TheWorld.Map:CanDeployPlantAtPoint`：地块属于陆地且不是 `GROUND_HARD`（`tilemanager` 里 `ground_tile_def.hard` 的地块），再用拥有者 `replica.inventoryitem:DeploySpacingRadius()` 做 `IsDeployPointClear`。[部署门禁](../../../raw/current/scripts/components/deployable.lua#L89-L102)、[模式分支](../../../raw/current/scripts/components/deployable.lua#L112-L113)、[地块判定](../../../raw/current/scripts/components/map.lua#L167-L175)、[落点判定](../../../raw/current/scripts/components/map.lua#L323-L326)
- **间距档**：`DEPLOYSPACING.MEDIUM` 的半径是 1（`DEPLOYSPACING_RADIUS`），默认档是 2；本页三件都命中 `mediumspacing`，所以种植最小间距比浆果丛／香蕉丛那类默认档更小。[枚举](../../../raw/current/scripts/constants.lua#L2236-L2246)、[半径表](../../../raw/current/scripts/constants.lua#L2254-L2263)、[枚举](../../../raw/current/scripts/constants.lua#L2215-L2226)
- **世界态一侧**：草丛与两种树苗在构造时都 `SetDeploySmartRadius(DEPLOYSPACING_RADIUS[DEPLOYSPACING.MEDIUM] / 2)`（0.5），该半径参与“别人放在它旁边”的间距计算。[草丛](../../../raw/current/scripts/prefabs/grass.lua#L219)、[树苗](../../../raw/current/scripts/prefabs/sapling.lua#L114)

### 三变体差异

| 维度 | `dug_grass` | `dug_sapling` | `dug_sapling_moon` |
|---|---|---|---|
| 数据表项 | `grass` | `sapling` | `sapling_moon` |
| bank／build | `grass`／`grass1` | `sapling`／`sapling` | `sapling_moon`／`sapling_moon` |
| floater | `{"med", 0.1, 0.92}` | `{"large", 0.1, 0.55}` | `{"large", 0.1, 0.55}` |
| `mediumspacing` | 是 | 是 | 是 |
| `inspectoverride` | 无，回落到 `dug_grass` | 无，回落到 `dug_sapling` | `dug_sapling` |
| `scrapbook_specialinfo` | `PLANTABLE_FERTILIZE` | `PLANTABLE` | `PLANTABLE` |
| `halloweenmoonmutable` | 无 | 有，指向 `dug_sapling_moon` | 无 |
| placer 动画参数 | `idle`（缺省） | `idle`（缺省） | `idle`（缺省） |
| 挖起来源 | `grass` | `sapling` | `sapling_moon` |

差异全部落在数据表，工厂代码不分叉：同一条 `make_plantable` 路径里没有按名字判断的组件分支，唯一按名字判断的是图鉴分流。[数据表](../../../raw/current/scripts/prefabs/plantables.lua#L119-L182)、[分流](../../../raw/current/scripts/prefabs/plantables.lua#L62-L76)

同表其它条目靠 `bank`／`build` 覆盖换外观，例如 `monkeytail` 用 `bank = "grass"` ＋ `build = "reeds_monkeytails"`，产物是 `cutreeds`；`rock_avocado_bush` 另覆盖 bank／build／anim，`marsh_bush` 只写 `mediumspacing` 与 floater，bank／build 沿用默认名、动画走缺省 `idle`。这些条目与它们的挖出形态归 B14，本页只在数据表层面引用。[monkeytail 数据项](../../../raw/current/scripts/prefabs/plantables.lua#L175-L179)、[marsh_bush 数据项](../../../raw/current/scripts/prefabs/plantables.lua#L157-L159)

### 蜡封形态

注册循环同时产出 `dug_<name>_waxed`（`CreateDugWaxedPlant`，注册名取 `parentprefab.."_waxed"`），它与本体共用 `data`、复用同一 placer（`overridedeployplacername = parentprefab.."_placer"`），但自身不可堆叠、另存 `savedata` 以保住原植株外观。[接线](../../../raw/current/scripts/prefabs/plantables.lua#L190)、[工厂](../../../raw/current/scripts/prefabs/waxed_plant_common.lua#L440-L539)、[[reference/components/waxable|蜡封组件]]。这三件的蜡封链路与数值归 B14／B43，本页不展开。

## 世界态来源与挖起

| 世界态 | 采集（`pickable`） | 挖起（`workable` DIG 1 次） | 移植回调 |
|---|---|---|---|
| `grass` | `SetUp("cutgrass", TUNING.GRASS_REGROW_TIME)`，3 天；`max_cycles = cycles_left = GRASS_CYCLES`（20） | 可采时先掉 `cutgrass`，再掉 `dug_grass`；高温枯萎（`withered`）时改掉 `cutgrass`，仍移除本体 | `MakeBarren()`（转枯死）＋ `makemorphable` ＋ 起 `morphdelay` |
| `depleted_grass` | 同一 `grass(name, stage)` 工厂传 `stage = 1`，构造即 `MakeBarren()` | 同上 | 同上 |
| `sapling` | `SetUp("twigs", TUNING.SAPLING_REGROW_TIME)`，4 天；不设 `max_cycles`，永不 `IsBarren` | 可采时先掉 `twigs`，再掉 `dug_sapling`；高温枯萎时只掉 `twigs` | `MakeEmpty()`（空株，4 天后自然再生） |
| `sapling_moon` | 同 `sapling`（`sapling_common(inst, true)`，`_is_moon = true`） | 同上，但挖出件按 `_is_moon` 换成 `dug_sapling_moon` | 同 `sapling` |

- 采集产物与轮次：草每次 1 个 `cutgrass`，`cycles_left` 初始 20，但只有移植过的草丛才每采一次扣 1，扣满 20 次归零进枯死（barren）态，野生丛不扣；树苗每次 1 个 `twigs`，无轮次上限。[草](../../../raw/current/scripts/prefabs/grass.lua#L245-L255)、[树苗](../../../raw/current/scripts/prefabs/sapling.lua#L138-L146)、[常量](../../../raw/current/scripts/tuning.lua#L1765-L1766)、[轮次](../../../raw/current/scripts/tuning.lua#L1786)、[扣减条件](../../../raw/current/scripts/components/pickable.lua#L535-L539)
- 挖起分支：两者都先补一次“仍可采则掉产物”，再决定移栽件；草用 `withered and "cutgrass" or "dug_grass"`，树苗用 `withered and "twigs" or (_is_moon and "dug_sapling_moon" or "dug_sapling")`。[草挖起](../../../raw/current/scripts/prefabs/grass.lua#L68-L86)、[树苗挖起](../../../raw/current/scripts/prefabs/sapling.lua#L31-L46)
- 挖起条件：两者都只在 `not GetGameModeProperty("disable_transplanting")` 时挂 `workable`（`ACTIONS.DIG`、`SetWorkLeft(1)`），该模式世界里没有挖起入口。[草](../../../raw/current/scripts/prefabs/grass.lua#L266-L271)、[树苗](../../../raw/current/scripts/prefabs/sapling.lua#L154-L159)
- 冬季：两者都 `MakeNoGrowInWinter`，入冬即 `pickable:Pause()`，出冬 `Resume()`（出冬不一定是春季：长度为 0 的季节会被跳过，只有出冬时恰为春季才按 `SPRING_GROWTH_MODIFIER` 0.75 折算剩余时间）。`paused` 期间采摘不会排再生计时。[helper](../../../raw/current/scripts/standardcomponents.lua#L822-L833)、[采摘排程条件](../../../raw/current/scripts/components/pickable.lua#L559-L573)
- 高温枯萎（`withered`）：`witherable` 在高温且未淋雨时调 `MakeBarren()`（转入枯死／`barren`）并记下 `restore_cycles`；恢复时先把 `restore_cycles` 写回 `cycles_left`，写回后仍非 `barren` 才 `MakeEmpty`。草丛与树苗都在主模拟挂该组件。[枯萎](../../../raw/current/scripts/components/witherable.lua#L59-L69)、[恢复](../../../raw/current/scripts/components/witherable.lua#L102-L117)
- 草的额外分支：采摘与挖起时若玩家、非冬季且通过 `GRASSGEKKO_MORPH_CHANCE`（1/100），会对附近 `grass` 推 `morphing`／`morphrelay` 短计时；到点只有被推 `morphing`、仍 `canmorph` 且密度允许（20 半径内草蜥少于 6）的株变成 `grassgekko`，其余重排 25±5 天的 `morphdelay`。移植只起 `morphdelay`，到期不等于变形。[触发](../../../raw/current/scripts/prefabs/grass.lua#L72-L77)、[传播](../../../raw/current/scripts/prefabs/grass.lua#L34-L66)、[到点](../../../raw/current/scripts/prefabs/grass.lua#L138-L161)、[注册](../../../raw/current/scripts/prefabs/grass.lua#L163-L171)、[常量](../../../raw/current/scripts/tuning.lua#L3001-L3006)

### 月树苗的来源

本节的“月树苗”指月岛树苗的挖出件（`dug_sapling_moon`，实际显示“树苗”）；月蛾种下的 `moonbutterfly_sapling` 是另一实体（显示名“月树苗”），见 [[reference/prefabs/moontree]]。

- **月岛世界生成**：资源映射把 `moon_sapling` 映射为 `sapling_moon`；月岛房间按权重散布（森林 0.3、浴场 1.0、草甸 1）。[资源映射](../../../raw/current/scripts/map/forest_map.lua#L226)、[森林房](../../../raw/current/scripts/map/rooms/forest/terrain_moonisland.lua#L82-L90)、[浴场与草甸](../../../raw/current/scripts/map/rooms/forest/terrain_moonisland.lua#L129-L160)
- **世界态就地变异**：普通 `sapling` 挂 `halloweenmoonmutable`，但用的是 `SetConversionOverrideFn(moonconversionoverridefn)`：突变不生成新实体，而是把当前株的 `_is_moon` 置真、换 bank／build、直接把 `inst.prefab` 改成 `sapling_moon`，并移除该组件。[接线](../../../raw/current/scripts/prefabs/sapling.lua#L187-L188)、[回调](../../../raw/current/scripts/prefabs/sapling.lua#L82-L92)
- **挖出件变异**：`dug_sapling` 走通用 `SetPrefabMutated` 分支，突变时 `SpawnPrefab("dug_sapling_moon")` 到原位置，保留容器与堆叠语义（成叠时取 1 个）。[接线](../../../raw/current/scripts/prefabs/plantables.lua#L139-L142)、[突变实现](../../../raw/current/scripts/components/halloweenmoonmutable.lua#L45-L79)、[使用入口](../../../raw/current/scripts/components/halloweenpotionmoon.lua#L11-L30)
- **配方直产**：Wormwood 的技能配方 `wormwood_sapling` 产物就是 `dug_sapling_moon`，不需要先有月岛植株。[配方](../../../raw/current/scripts/recipes.lua#L343)
- **显示名**：`sapling_moon` 没有 `STRINGS.NAMES.SAPLING_MOON` 键，它靠 `SetPrefabNameOverride("sapling")` 取名字；`dug_sapling_moon` 的 HUD 名走自己的 `NAMES.DUG_SAPLING_MOON`（同样是“树苗”），`inspectoverride = "dug_sapling"` 影响的是检查台词里的物品名。[覆盖](../../../raw/current/scripts/prefabs/sapling.lua#L198)、[显示名解析](../../../raw/current/scripts/entityscript.lua#L687-L692)、[台词取名](../../../raw/current/scripts/stringutil.lua#L375)、[DUG_SAPLING_MOON](../../../raw/current/scripts/languages/chinese_s.po#L342350-L342352)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：pickable | 挖出件不挂；世界态用它定义产物、再生与 `OnTransplant` 落点 | 仅世界态，主模拟 | [[reference/components/pickable#关键接口]]；[接口](../../../raw/current/scripts/components/pickable.lua#L222-L228)、[移植](../../../raw/current/scripts/components/pickable.lua#L418-L424) |
| 组件：deployable | PLANT 模式、间距、`ondeploy` 回调 | 挖出件，主模拟；`deployedplant` 标签另触发 `itemplanted` | [[reference/components/deployable#关键接口]]；[部署段](../../../raw/current/scripts/prefabs/plantables.lua#L99-L105) |
| 组件：stackable | 上限 10；放置时 `Get()` 取 1 个再 `Remove` | 挖出件；成叠时每次只消耗 1 | [[reference/components/stackable#关键接口]]；[取 1 个](../../../raw/current/scripts/components/stackable.lua#L109-L120) |
| 组件：inventoryitem／inspectable | 入栏；`inspectable.nameoverride` 决定检查台词里的物品名 | 挖出件；`inspectoverride` 优先 | [[reference/components/inspectable#关键接口]]；[赋值](../../../raw/current/scripts/prefabs/plantables.lua#L87-L89) |
| 组件：fuel／burnable／propagator／hauntable | 燃料 180 秒、燃尽 30 秒、蔓延、作祟弹飞（点火分支已注释，实际不点燃） | 挖出件，主模拟 | [[reference/components/fuel#关键接口]]、[[reference/components/burnable#关键接口]]；[组件段](../../../raw/current/scripts/prefabs/plantables.lua#L91-L97) |
| 组件：halloweenmoonmutable | `dug_sapling` → `dug_sapling_moon`；世界态用转换覆盖函数 | 挖出件走生成替换，世界态走就地改写 | [[reference/components/halloweenmoonmutable#关键接口]]；[两种用法](../../../raw/current/scripts/components/halloweenmoonmutable.lua#L24-L45) |
| 组件：witherable | 高温枯萎（`withered`）把草丛／树苗打成枯死（`barren`）态；施肥或降温恢复 | 世界态，主模拟 | [[reference/components/witherable#关键接口]]；[枯萎](../../../raw/current/scripts/components/witherable.lua#L59-L69) |
| 组件：lunarthrall_plantspawner | 落地后给带 `lunarplant_target` 的植株补 `knownlocations`／`herdmember` | 世界态；世界组件存在时 | [[reference/components/lunarthrall_plantspawner#关键接口]]；[登记](../../../raw/current/scripts/components/lunarthrall_plantspawner.lua#L529-L533) |
| API／数据：`MakePlacer` | 生成三个 placer，动画取 `data.anim or "idle"` | 客户端预览实体 | [[reference/components/placer#关键接口]]；[工厂](../../../raw/current/scripts/prefabutil.lua#L1-L65) |
| API／数据：`DEPLOYMODE`／`DEPLOYSPACING` | 部署模式 4（PLANT）、间距 MEDIUM=1 | 主模拟设值，replica 同步 | [枚举](../../../raw/current/scripts/constants.lua#L2215-L2226)、[半径](../../../raw/current/scripts/constants.lua#L2254-L2263) |
| 关联 Prefab：`grass`／`sapling`／`sapling_moon` | 挖起来源；放置时的同名世界态 | 世界实体 | [grass](../../../raw/current/scripts/prefabs/grass.lua#L207-L290)、[sapling](../../../raw/current/scripts/prefabs/sapling.lua#L106-L208) |
| 关联 Prefab：`dug_<name>_waxed` | 蜡封挖出件，保留外观与 `savedata` | 本页只给指针，B14／B43 | [工厂](../../../raw/current/scripts/prefabs/waxed_plant_common.lua#L440-L539) |
| 资源：anim 包 | `anim/grass.zip`＋`anim/grass1.zip`；`anim/sapling.zip`；`anim/sapling_moon.zip` | 客户端段加载 | [资产](../../../raw/current/scripts/prefabs/plantables.lua#L7-L15) |

## 典型接入

### 让移植后的草丛不再枯死

- 前提与选择：服务端 `modmain.lua`。改的是世界态 `grass` 的 `pickable.ontransplantfn`——`ondeploy` 在 `SpawnPrefab` 之后才调 `OnTransplant()`，postinit 先于该调用执行，替换后放置的就是新行为。比起改 `MakeBarren` 本身，只覆盖这一株的回调影响面最小。
- 生效与撤销：`AddPrefabPostInit` 在每次构造（含读档重建）后执行；停用模组即恢复原回调。`OnTransplant` 仍会把 `transplanted` 置真，因此之后每次采摘都会扣 `cycles_left`（初始 20），扣到 0 仍会枯死（`barren`）。
- 原版依据：默认回调与 `MakeBarren`／`MakeEmpty` 的差别见[世界态来源与挖起](#世界态来源与挖起)；`MakeEmpty` 会重排 `baseregentime`（3 天）。

```lua
-- modmain.lua：移植后的草丛改为“空株 + 正常再生”，不再要求施肥
AddPrefabPostInit("grass", function(inst)
    if not GLOBAL.TheWorld.ismastersim or inst.components.pickable == nil then
        return
    end
    inst.components.pickable.ontransplantfn = function(plant)
        plant.components.pickable:MakeEmpty()
    end
end)
```

- 调整与验证：单位是轮与秒；本改动同时丢掉默认回调里的 `makemorphable` 与变草蜥计时（见陷阱）。若不想动回调，只能按原版流程施肥：`Fertilize` 会把 `cycles_left` 填回 `max_cycles`（20）并让丛以空株再生；改 `TUNING.GRASS_CYCLES` 只影响之后构造的草丛。

## 常见陷阱与限制

- **误区：`dug_sapling_moon` 的中文名是“月树苗”**：`STRINGS.NAMES.SAPLING_MOON` 这个键不存在；世界态用 `SetPrefabNameOverride("sapling")`、挖出件用 `inspectoverride = "dug_sapling"`，两者实际显示都是“树苗”。[覆盖](../../../raw/current/scripts/prefabs/sapling.lua#L198)、[`DUG_SAPLING_MOON`](../../../raw/current/scripts/languages/chinese_s.po#L342350-L342352)
- **边界：放置只消耗 1 个**：`stackable:Get()` 在成叠时生成新实例并把堆减 1，所以一叠 10 个能种 10 株，不会被一次清空；皮肤经 `linked_skinname`／`skin_id` 带到世界态。[取 1 个](../../../raw/current/scripts/components/stackable.lua#L109-L120)、[传皮肤](../../../raw/current/scripts/prefabs/plantables.lua#L18)
- **边界：草与树苗移植后果不同**：草丛 `ontransplantfn` 是 `MakeBarren()`（`cycles_left = 0`，必须施肥），树苗是 `MakeEmpty()`（只是空株，4 天后自然结果）。照草丛经验去处理树苗会多施一次肥。[草](../../../raw/current/scripts/prefabs/grass.lua#L173-L177)、[树苗](../../../raw/current/scripts/prefabs/sapling.lua#L27-L29)、[施肥重置](../../../raw/current/scripts/components/pickable.lua#L274-L292)
- **误区：「移植株 25±5 天后变草蜥」**：移植回调只 `MakeBarren()`＋`makemorphable` 起 `morphdelay`（25±5 天）；`morphdelay` 到期的 `timerdone` 在 `onmorphtimer` 里没有分支，只解除「正在计时」的排除。变形还要再被触发一次：采摘／挖起时「非冬季＋picker 是玩家＋通过 `GRASSGEKKO_MORPH_CHANCE`（1/100）」才 `triggernearbymorph(inst, true)`，对 1 半径内（无目标翻倍，最多 4）`renewable`（排除 `INLIMBO`）、不含自己且三块 morph 计时都空闲的 `grass` 计数：`canmorph`（仍在播 `idle`）且 `math.random() < 0.75` 的收 `morphing` 短计时，其余收 `morphrelay`（`count` 帧）。到点后只有收 `morphing` 的那一株再查 `canmorph` 与密度：20 半径内 `grassgekko` 少于 6 才 `SpawnPrefab`＋`emerge`＋`grasspartfx`，再以非 quick 模式 `triggernearbymorph(inst, false)` 让邻近植株续接，最后 `Remove()` 自己；失败或只收到 `morphrelay` 的个体重排 `morphdelay`，并再向邻株传播一轮（`triggernearbymorph(inst, false)`）。`GRASSGEKKO_MORPH_ENABLED` 可整体关掉；替换 `ontransplantfn` 会一并丢掉 `makemorphable`。[拾取触发](../../../raw/current/scripts/prefabs/grass.lua#L72-L77)、[传播](../../../raw/current/scripts/prefabs/grass.lua#L28-L66)、[到点](../../../raw/current/scripts/prefabs/grass.lua#L138-L161)、[注册](../../../raw/current/scripts/prefabs/grass.lua#L163-L177)、[常量](../../../raw/current/scripts/tuning.lua#L3001-L3006)
- **边界：冬季采完不会立刻开始再生**：`MakeNoGrowInWinter` 让 `pickable` 处于 `paused`，`Pick` 的排程条件里 `self.paused` 为真时直接跳过，直到出冬 `Resume()` 补一次（出冬不一定是春季，只有恰逢春季才乘 0.75；本页两种植物都没有雪覆盖组件，雪覆盖的表现见 [[reference/prefabs/berry-bushes#采摘与再生]]）。[暂停](../../../raw/current/scripts/standardcomponents.lua#L822-L833)、[排程](../../../raw/current/scripts/components/pickable.lua#L559-L573)
- **未核验**：quagmire 模式下 `sapling` 的 `event_server_data` 分支、蜡封挖出件与本体之间的外观／存档细节（B14／B43）、草蜥的后续行为与掉落。

## 验证记录

- 源码审阅：2026-11-08，Steam buildid 26474873（2026-11-09 复检草蜥两段链、部署门禁与枯死／高温枯萎用词）。逐行读完 `plantables.lua`（工厂、资产、客户端段、图鉴分流、主模拟段、ondeploy、数据表 10 项、注册循环）、`grass.lua`（挖起、再生／空／枯死回调、草蜥计时、构造与 `depleted_grass`）、`sapling.lua`（挖起、移植回调、`sapling_common`、月树苗工厂与就地转换）、`monkeytail.lua`（仅用于对照 bank 覆盖）；核验 `MakePlacer`、`deployable` 的 PLANT 与间距同步、`inventoryitem_replica` 的 placer 取名、`stackable:Get`、`map.CanDeployPlantAtPoint` 与 `CanPlantAtPoint`、`pickable` 的 SetUp／Pick／MakeBarren／MakeEmpty／OnTransplant、`witherable` 的枯萎与恢复、`halloweenmoonmutable` 两条突变路径、`lunarthrall_plantspawner.setHerdsOnPlantable`、`standardcomponents` 的浮水／可燃／冬季暂停／雪覆盖 helper、tuning 常量与月岛资源映射。
- 名称核验（`inspect_prefab_names.py`，仅认 `languages/chinese_s.po` 的 msgstr）：`GRASS`＝草、`DUG_GRASS`＝草丛、`SAPLING`＝树苗、`DUG_SAPLING`＝树苗、`DUG_SAPLING_MOON`＝树苗；`SAPLING_MOON` 无此键。[GRASS](../../../raw/current/scripts/languages/chinese_s.po#L343390-L343392)、[DUG_GRASS](../../../raw/current/scripts/languages/chinese_s.po#L342320-L342322)、[SAPLING](../../../raw/current/scripts/languages/chinese_s.po#L348965-L348967)、[DUG_SAPLING](../../../raw/current/scripts/languages/chinese_s.po#L342345-L342347)、[DUG_SAPLING_MOON](../../../raw/current/scripts/languages/chinese_s.po#L342350-L342352)
- 游戏运行：未运行。天数与秒数为 tuning 推导（3 天＝1440 秒、4 天＝1920 秒、`LARGE_FUEL`＝30×6＝180 秒、`LARGE_BURNTIME`＝30 秒）；变草蜥与突变是代码路径推导。
- 结构检查：`validate_page.py`（`--wiki-root` 指向 `dst-wiki/wiki`）通过，standard 档内（行数／字节见校验输出）；`validate_source_links.py --strict` 0 错 0 警。

## 相关链接

- 材料与产物：[[reference/prefabs/basic-gathered-materials]]（cutgrass、twigs）、[[reference/prefabs/wild-berries]]（浆果丛一侧的移植语义见 [[reference/prefabs/berry-bushes]]）
- 工具入口：[[reference/prefabs/farm-hoe]]、[[reference/prefabs/soil-amender]]（施肥与耕地工具，数值不在本页）
- 组件：[[reference/components/pickable]]、[[reference/components/deployable]]、[[reference/components/waxable]]、[[reference/components/halloweenmoonmutable]]

返回 [[reference/prefabs/index|Prefab 索引]]。