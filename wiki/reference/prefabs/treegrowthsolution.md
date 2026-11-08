---
title: "树果酱（treegrowthsolution）"
created: "2026-11-10"
updated: "2026-11-10"
type: prefab
summary: "一件物品四类用途、五个动作入口：催长（ADVANCE_TREE_GROWTH）、施肥（DEPLOY 只认农田格；另有独立的 FERTILIZE 走旧农田目标或自肥，两条入口共用同一个 Fertilizer:OnApplied 扣件）、补船漏（REPAIR_LEAK）、按 WOOD 材质直修掉血目标（REPAIR）。扣件是三处主扣件（GrowTarget、Fertilizer:OnApplied、Repairable:Repair）加一处兜底（BoatLeak:Repair，只在补漏没修成时自扣）。物品本身不可燃、无燃料值。"
aliases: ["treegrowthsolution", "树果酱", "树生长溶液"]
tags: [prefab, item]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/treegrowthsolution.lua", "components/treegrowthsolution.lua", "components/fertilizer.lua", "components/fertilizerresearchable.lua", "components/boatpatch.lua", "components/boatleak.lua", "components/repairer.lua", "components/repairable.lua", "components/deployable.lua", "components/growable.lua", "components/stackable.lua", "components/hauntable.lua", "standardcomponents.lua", "actions.lua", "componentactions.lua", "prefabs/fertilizer_nutrient_defs.lua", "prefabs/oceantree.lua", "prefabs/walls.lua", "fx.lua", "components/health.lua", "recipes.lua", "components/piratespawner.lua", "prefabs/boatrace_primemate.lua", "brains/boatrace_primematebrain.lua", "stategraphs/SGprimemate.lua", "prefabs/pirate_stash.lua", "skins_defs_data.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 树果酱（treegrowthsolution）

## 快速判断

- 职责与适用：单个注册 `treegrowthsolution`，一件物品有四类用途、五个动作入口——`treegrowthsolution` 组件负责对树催长、`deployable`＋`fertilizer` 负责施肥（农田 DEPLOY，以及旧农田目标／自肥的独立 FERTILIZE）、`boatpatch`＋`repairer` 既负责补船漏也负责按 WOOD 材质直修掉血目标。[工厂](../../../raw/current/scripts/prefabs/treegrowthsolution.lua#L21-L84)
- 优先使用：催长的执行语义与“没有成长入口也扣材料”读 [[reference/components/treegrowthsolution]]；补漏／到期回漏／材料匹配读 [[reference/prefabs/boat-patches]]；树对象与阶段表读 [[reference/prefabs/evergreen-trees]] 与 [[reference/components/growable]]。
- 重要副作用：五条入口只有三处主扣件——`GrowTarget` 自己扣、`Fertilizer:OnApplied` 扣（DEPLOY 与 FERTILIZE 共用）、`Repairable:Repair` 扣（补漏成功与直修都走这里）；补漏时若 `repairable:Repair` 没成功（材料不符／目标满血），才由 `BoatLeak:Repair` 兜底自扣一件。补漏成功后漏点实体在两次动画结束后自行 `Remove`，这条分支不设到期任务。作祟只弹飞。
- 修改边界：本页不展开 `growable` 阶段表、`oceantree` 的特殊进度与船体损伤机制；物品**不可燃、无燃料值**（工厂体没有 `burnable`／`fuel`），与 [[reference/prefabs/fertilizer-materials|四件基础肥料]]不同。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `treegrowthsolution`（树果酱） | `fn` 直接返回 `Prefab("treegrowthsolution", fn, assets, prefabs)`：Transform／AnimState／SoundEmitter／Network → `MakeInventoryPhysics` → bank+build `treegrowthsolution` 播 `item` → 加 `allow_action_on_impassable` 与 `boat_patch` 标签 → `MakeInventoryFloatable("med", 0.1)` → `MakeDeployableFertilizerPristine` → 加 `fertilizerresearchable` 标签与 `GetFertilizerKey` → `scrapbook_anim = "item"` → `SetPristine` → 客户端早退 → 主模拟装 fertilizerresearchable／fertilizer／treegrowthsolution／stackable／inspectable／inventoryitem／boatpatch／repairer，再 `MakeDeployableFertilizer` 与 `MakeHauntableLaunchAndSmash` | 单工厂单注册，没有包装工厂与皮肤分支；`inventoryitem:SetSinks(false)` 让它落水不下沉；`stackable.maxsize = TUNING.STACK_SIZE_LARGEITEM`（10）。[工厂](../../../raw/current/scripts/prefabs/treegrowthsolution.lua#L21-L84)、[`SetSinks`](../../../raw/current/scripts/prefabs/treegrowthsolution.lua#L69-L70)、[堆叠](../../../raw/current/scripts/prefabs/treegrowthsolution.lua#L64-L65)、[上限值](../../../raw/current/scripts/tuning.lua#L81) |

名称：注册名 `treegrowthsolution` 的 `STRINGS.NAMES.TREEGROWTHSOLUTION` 在简中实显为 **树果酱**（英文 msgid 是 "Tree Jam"）；[`chinese_s.po`](../../../raw/current/scripts/languages/chinese_s.po#L350570-L350572)。标题按实显名，"树生长溶液"只作检索别名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：treegrowthsolution | 字段只有 `fx_prefab = "treegrowthsolution_use_fx"`；`GrowTarget(target)` 先拒 `no_force_grow`／`stump`／`fire`／`burnt`，再生成 FX、调 `target:override_treegrowthsolution_fn(item)` 或 `growable:DoGrowth()`，最后扣一份 | 服务端动作 `ACTIONS.ADVANCE_TREE_GROWTH`；直接调用不含候选门禁，也不看回调与 `DoGrowth` 的返回值 | [[reference/components/treegrowthsolution]]；[组件](../../../raw/current/scripts/components/treegrowthsolution.lua#L6-L32)、[动作](../../../raw/current/scripts/actions.lua#L5679-L5685) |
| 组件：fertilizer | `fertilizervalue = TREEGROWTH_FERTILIZE`（300 秒）、`soil_cycles = 8`、`withered_cycles = 1`、`nutrients = {8, 32, 8}`（催长素／堆肥／粪肥） | 服务端；数值供部署与旧 `FERTILIZE` 路径 | [[reference/components/fertilizer]]；[写入](../../../raw/current/scripts/prefabs/treegrowthsolution.lua#L55-L59)、[数值](../../../raw/current/scripts/tuning.lua#L1657-L1659)、[养分](../../../raw/current/scripts/tuning.lua#L5876) |
| 组件：deployable（＋两端 pristine helper） | 与四件肥料同一套：`_custom_candeploy_fn` 只认农田土壤、`overridedeployplacername = "gridplacer_farmablesoil"`、`keep_in_inventory_on_deploy = true`；落地加 `+8／+32／+8` 后走 `OnApplied` | 服务端；部署等价于“施肥一次”，不会种出任何东西 | [pristine](../../../raw/current/scripts/standardcomponents.lua#L1816-L1831)、[结算](../../../raw/current/scripts/standardcomponents.lua#L1797-L1810)、[动作名](../../../raw/current/scripts/actions.lua#L1657) |
| 组件：fertilizerresearchable | `SetResearchFn(fertilizerresearchfn)`，键取 `inst.prefab` | 服务端；该键另在 `skins_defs_data.lua` 的 `AddPlantRegistryKey` 调用列表内（API 实现不在 `scripts/`，登记作用未核验） | [[reference/components/fertilizerresearchable]]；[接线](../../../raw/current/scripts/prefabs/treegrowthsolution.lua#L52-L53)、[登记](../../../raw/current/scripts/skins_defs_data.lua#L5290) |
| 组件：boatpatch | 只为提供 `patch_type = "treegrowth"`（组件构造还会补 `boat_patch` 标签） | 服务端；决定补漏后进入 `repaired_treegrowth` 外观而非永久 `repaired` | [[reference/components/boatpatch]]；[设置](../../../raw/current/scripts/prefabs/treegrowthsolution.lua#L72-L73)、[状态分流](../../../raw/current/scripts/components/boatleak.lua#L240-L249) |
| 组件：repairer | `repairmaterial = MATERIALS.WOOD`、`healthrepairvalue = REPAIR_TREEGROWTH_HEALTH`（20）、`boatrepairsound = "waterlogged1/common/use_figjam"`；构造按数值挂 `health_WOOD` 标签 | 服务端；补漏回血与 `REPAIR` 直修共用这套数值 | [[reference/components/repairer]]；[配置](../../../raw/current/scripts/prefabs/treegrowthsolution.lua#L75-L78)、[数值](../../../raw/current/scripts/tuning.lua#L2121)、[标签](../../../raw/current/scripts/components/repairer.lua#L8-L24) |
| 组件：stackable／inventoryitem | 堆叠 10；inventoryitem 不接 dropped／putin 回调 | 两者都只在主模拟添加，`SetSinks(false)` 也在主模拟设 | [堆叠](../../../raw/current/scripts/prefabs/treegrowthsolution.lua#L64-L70)、[[reference/components/stackable]] |
| 组件：hauntable | `MakeHauntableLaunchAndSmash`：`Launch` 一定执行，砸碎段被 `#HAUNTFIX` 注释，函数返回 true | 服务端；与四件肥料的 `LaunchAndIgnite`（返回 false）不同，这里返回 true，`DoHaunt` 走成功分支 push `"haunted"`，并让弹飞前已置的 `hauntvalue`＝1 生效（doer 非空时用完即清空） | [helper](../../../raw/current/scripts/standardcomponents.lua#L1071-L1105)、[成功分支](../../../raw/current/scripts/components/hauntable.lua#L89-L104) |
| 动作：`ADVANCE_TREE_GROWTH`／`DEPLOY`／`FERTILIZE`／`REPAIR_LEAK`／`REPAIR` | 分别由 `treegrowthsolution`、`deployable`、`fertilizer`、`boatpatch`、`repairer` 五个组件动作提供者注册；DEPLOY（农田格）与 FERTILIZE（旧目标／自肥）是两条入口，都落到 `Fertilizer:OnApplied` | 客户端按组件动作与标签出菜单，服务端再复核；DEPLOY 只认农田土壤；FERTILIZE 要求目标带 `notreadyforharvest`／`fertile`｜`infertile`／`barren`／`fertilizable`，或对带 `self_fertilizable` 的玩家自肥；补漏要求目标带 `boat_leak`；直修要右击、目标是 `repairable_wood`＋`healthrepairable`（后者由 `health` 在掉血时置上，满血目标不出该动作；原版如受损的木墙） | [树候选](../../../raw/current/scripts/componentactions.lua#L1870-L1881)、[施肥候选](../../../raw/current/scripts/componentactions.lua#L1241-L1254)、[施肥结算](../../../raw/current/scripts/actions.lua#L1812-L1841)、[补漏候选](../../../raw/current/scripts/componentactions.lua#L1070-L1074)、[直修候选](../../../raw/current/scripts/componentactions.lua#L1679-L1700)、[health 置标签](../../../raw/current/scripts/components/health.lua#L18-L24)、[木墙可修](../../../raw/current/scripts/prefabs/walls.lua#L324-L327) |
| 关联 Prefab：`treegrowthsolution_use_fx` | `GrowTarget` 在目标坐标生成的一次性表现，bank／build 同 `treegrowthsolution`、anim `use`、音效 `waterlogged1/common/use_figjam` | 仅服务端生成点；FX 自身表现未解包核验 | [fx 定义](../../../raw/current/scripts/fx.lua#L2629-L2634)、[生成](../../../raw/current/scripts/components/treegrowthsolution.lua#L15-L17) |
| 关联 Prefab：`boat_leak`／船 | 补漏目标；`BoatLeak:Repair` 先试 `repairable:Repair`（材料不符或船满血则只堵漏不回血），再按 `patch_type` 分流状态 | 服务端；`repaired_treegrowth` 分支把外观换成 `treegrowthsolution` 的 `pre_idle→idle`，随后移除漏点实体 | [[reference/prefabs/boat-patches]]、[[reference/components/boatleak]]；[Repair](../../../raw/current/scripts/components/boatleak.lua#L62-L94)、[状态](../../../raw/current/scripts/components/boatleak.lua#L240-L249) |

### 获得与消耗去向

| 环节 | 内容 | 条件与证据 |
|---|---|---|
| 配方 | `fig × 2 + glommerfuel × 1`，`TECH.SCIENCE_TWO` | 二级科学机器；[配方](../../../raw/current/scripts/recipes.lua#L446) |
| 藏宝箱战利品 | 海盗藏宝箱生成时 30% 概率放 2–4 个 | 由 `piratespawner:generateloot` 组表后塞进箱子；[loot](../../../raw/current/scripts/components/piratespawner.lua#L572-L576)、[入口](../../../raw/current/scripts/components/piratespawner.lua#L621-L630) |
| 海盗船长随身物 | 生成海盗船时给 `prime_mate` 背包放 2 个并加 `personal_possession` 标签 | 只作船长掉落／随身物，不是玩家建造物；[生成](../../../raw/current/scripts/components/piratespawner.lua#L295-L304) |
| AI 的“补丁代币” | 行为树第二个 `DoAction` 是 `FixBoat`：先 `inventory:FindItem(is_boatpatch)`（任何带 `boatpatch` 组件的补丁都算），背包没有才依次造 3 件塞进去，再 `BufferedAction(…, ACTIONS.REPAIR_LEAK, boatpatch)` 走正常动作结算（扣件按上节的 repairable／boatleak 分支） | `patch_boat_cooldown`＝3 秒在 `SGprimemate` 的 `action` 状态上电，冷却中 `FixBoat` 直接返回 nil；只挑 `has_leaks` 且 `IsFinishedSpawning()` 的漏点。另有睡眠分支直接 `SpawnPrefab("treegrowthsolution")` 调 `BoatLeak:Repair`，不走 BufferedAction、也不看冷却 | [FixBoat](../../../raw/current/scripts/brains/boatrace_primematebrain.lua#L63-L105)、[行为树](../../../raw/current/scripts/brains/boatrace_primematebrain.lua#L244-L247)、[冷却](../../../raw/current/scripts/stategraphs/SGprimemate.lua#L101-L103)、[睡眠分支](../../../raw/current/scripts/prefabs/boatrace_primemate.lua#L121-L151) |
| 玩家侧的五条消费 | 催长一段／农田部署一次／对旧农田目标或自身施肥一次／补一个船漏／直修掉血的 WOOD 可修目标 | 每条入口扣 1 件：催长一处，DEPLOY 与 FERTILIZE 共用 `Fertilizer:OnApplied`，补漏与直修共用 `Repairable:Repair` 的扣件，补漏没修成才由 `BoatLeak:Repair` 兜底；[催长消费](../../../raw/current/scripts/components/treegrowthsolution.lua#L25-L29)、[部署消费](../../../raw/current/scripts/components/fertilizer.lua#L41-L47)、[Repairable 扣件](../../../raw/current/scripts/components/repairable.lua#L172-L176)、[BoatLeak 兜底](../../../raw/current/scripts/components/boatleak.lua#L79-L85) |

## 典型接入

### 给自定义树接上“催长一段”

- 前提与选择：`GrowTarget` 优先调用目标的 `override_treegrowthsolution_fn`（冒号调用，参数为物品），其次才走 `growable:DoGrowth()`。自定义树若想自己决定推进多少阶段，接这个回调，不要改原组件。
- 生效与撤销：写在目标模组 `modmain.lua`，只在主模拟赋值；对象读档重建后需重新接（原版 `oceantree` 就是在工厂里直接赋的）。撤销即删该字段，已改变的阶段不回滚。
- 原版依据：[分派](../../../raw/current/scripts/components/treegrowthsolution.lua#L19-L23)、[oceantree 的接法](../../../raw/current/scripts/prefabs/oceantree.lua#L661-L665)

```lua
-- modmain.lua：自定义树被树果酱时最多推进两个阶段，仍由原逻辑扣一份
local function advance_two_stages(target, item)
    local growable = target.components.growable
    if growable == nil then
        return
    end
    for _ = 1, 2 do
        if not growable:DoGrowth() then
            break
        end
    end
end

AddPrefabPostInit("my_custom_tree", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    inst.override_treegrowthsolution_fn = advance_two_stages
end)
```

- 调整与验证：循环上限固定为 2，别写成 `while DoGrowth() do end`——`loopstages` 的 `growable` 会一直返回 true。回调返回值被忽略，扣材料发生在回调之后，所以不要在回调里再扣物品。示例只做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **边界：五条入口，施肥与补漏各有共用扣件。** 催长走 `GrowTarget` 的 `stackable` 直扣；农田 DEPLOY 与旧 FERTILIZE 都走 `Fertilizer:OnApplied`（FERTILIZE 要先 `applied` 为真）；补漏（REPAIR_LEAK）与直修（REPAIR）都走 `Repairable:Repair` 末尾的 `stackable:Get():Remove()`，只有补漏时 `repairable:Repair` 没成功才落到 `BoatLeak:Repair` 自扣。给其中一处加钩子不会影响其它入口；连续用两个入口会各扣一件。[催长](../../../raw/current/scripts/components/treegrowthsolution.lua#L25-L29)、[部署](../../../raw/current/scripts/prefabs/treegrowthsolution.lua#L80)、[Repairable 扣件](../../../raw/current/scripts/components/repairable.lua#L172-L176)、[BoatLeak 兜底](../../../raw/current/scripts/components/boatleak.lua#L79-L85)
- **边界：补漏后漏点会自己消失。** `repaired_treegrowth` 换成 `treegrowthsolution` 的动画后，`animover` 先切 `idle`、再 `Remove` 掉漏点实体，且不调 `SetRepairedTime`；所以既不会像海带补丁那样到期回漏，也不像木补丁那样长期留一块补丁。[状态](../../../raw/current/scripts/components/boatleak.lua#L240-L249)、[对照](../../../raw/current/scripts/components/boatleak.lua#L251-L258)
- **边界：补漏回血仍要材料匹配且船未满血。** `repairable:Repair` 在材料不等时直接 false、`address_repair_amount` 在 `GetPercent() >= 1` 时返回 false；两种情况补丁照样被消耗，只是不回血。[Repair](../../../raw/current/scripts/components/repairable.lua#L148-L162)、[回血](../../../raw/current/scripts/components/repairable.lua#L106-L115)
- **边界：不可燃、不能当燃料。** 工厂体没有 `burnable`／`fuel`，与 [[reference/prefabs/fertilizer-materials|四件基础肥料]]（都可燃、45 秒燃料）相反。[工厂全文](../../../raw/current/scripts/prefabs/treegrowthsolution.lua#L21-L84)
- **边界：作物候选只看标签。** `ADVANCE_TREE_GROWTH` 要求目标带 `tree` 且不带 `monster`／`fire`／`burnt`／`stump`／`leif`／`no_force_grow`；直接调 `GrowTarget` 只挡 `no_force_grow`／`stump`／`fire`／`burnt`，不验 `tree`／`monster`／`leif`。[候选](../../../raw/current/scripts/componentactions.lua#L1870-L1881)、[门禁](../../../raw/current/scripts/components/treegrowthsolution.lua#L6-L13)
- **误区：作祟会砸碎它。** `MakeHauntableLaunchAndSmash` 的砸碎段整段被 `#HAUNTFIX` 注释，实际只 `Launch`；不过它返回 true，`hauntvalue` 与 `"haunted"` 会照常生效，和四件肥料的 `LaunchAndIgnite` 不同。[helper](../../../raw/current/scripts/standardcomponents.lua#L1071-L1105)
- **边界：部署不种树。** 农田部署只写三养分，`deployable` 的 `ondeploy` 是肥料回调；把树果酱丢在土上不会生成任何植株。[结算](../../../raw/current/scripts/standardcomponents.lua#L1797-L1810)

## 验证记录

- 源码审阅（2026-11-10，steam-buildid:26474873）：通读 `prefabs/treegrowthsolution.lua` 与 `components/treegrowthsolution.lua`；核验 `fertilizer`／`fertilizerresearchable`／`deployable`／`boatpatch`／`repairer`／`stackable`／`hauntable` 的接线与数值；`actions.lua` 的 `ADVANCE_TREE_GROWTH`／`REPAIR_LEAK`／`REPAIR`／`DEPLOY` 结算；`componentactions.lua` 四处候选门禁；`boatleak.lua` 的 `Repair` 与 `repaired_treegrowth` 分支（含 `animover` 自删）；`repairable.lua` 的材料与满血门禁；`growable:DoGrowth` 的停止返回；`fx.lua` 的 `treegrowthsolution_use_fx` 定义；`recipes.lua`／`piratespawner.lua`／`boatrace_primemate.lua`／`oceantree.lua` 的获得与复用点。
- 名称核验：`inspect_prefab_names.py` 按 `STRINGS.NAMES.TREEGROWTHSOLUTION` 取到简中"树果酱"（英文 msgid "Tree Jam"）；标题用实显名，"树生长溶液"仅作别名。
- 游戏运行：未运行；补漏后漏点自删的观感、AI 大副持空补丁补漏的多人时序、`override_treegrowthsolution_fn` 在自定义树上的实际成长均未实测。
- 未覆盖：`oceantree` 的进度与 `enriched_cooldown` 定时消费者、`growable` 全阶段表、船体损伤与漏点产生机制（分别在对应页面），以及 `pirate_stash` 的 `scrapbook_adddeps`（只是图鉴依赖，不是掉落）。
- 内部检视 R1 修订（2026-11-10）：改为「入口清单＋三处扣件」，补漏成功时扣件归 `Repairable:Repair`、没修成才由 `BoatLeak:Repair` 兜底；补 AI 的 brain `FixBoat` 路径（先 FindItem、没有才造 3 件、走 BufferedAction）与睡眠分支两条路径，及 `patch_boat_cooldown`＝3 秒；直修补「`healthrepairable` 由 health 掉血置上」与受损木墙例；研究键登记降级；sources 补 brain／SGprimemate／health／walls／skins_defs_data。
- 外部检视 B14 修订（2026-11-11，R07）：改为「四类用途、五个动作入口」，把旧农田目标／自肥的 FERTILIZE 与农田 DEPLOY 并列为两条入口、共用 `Fertilizer:OnApplied`；summary／职责／副作用／动作行／消费表／陷阱同步。
- 结构检查：见本次交付报告的机械校验记录。

## 相关链接

- [[reference/components/treegrowthsolution|treegrowthsolution]]（组件执行语义与“无入口也消费”）
- [[reference/components/boatleak|boatleak]]、[[reference/components/boatpatch|boatpatch]]、[[reference/components/repairer|repairer]]
- [[reference/prefabs/boat-patches|船体修补物（boatpatch、boatpatch_kelp）]]（对照三种补漏结果）
- [[reference/prefabs/fertilizer-materials|基础肥料四件]]（同一套部署 helper、可作燃料的四件肥料）
- [[reference/prefabs/evergreen-trees|常青树]]、[[reference/components/growable|growable]]（催长目标与阶段表）

返回 [[reference/prefabs/index|原版 Prefab 索引]]。
