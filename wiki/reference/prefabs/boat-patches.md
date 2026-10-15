---
title: "船体修补物（船补丁 boatpatch、海带补丁 boatpatch_kelp）"
created: "2026-11-08"
updated: "2026-11-08"
type: prefab
summary: "prefabs/boatpatch.lua 双注册的船漏修补物：木补丁（boatpatch）永久补漏并按 WOOD 材料直修木质船体；海带补丁（boatpatch_kelp）复用木工厂后覆盖为 KELP 修复、临时补漏（中漏 0.5 天／小漏 0.75 天到期回漏）且可食可腐。"
aliases: ["boatpatch", "boatpatch_kelp", "船补丁", "海带补丁", "船体修补", "Boat Patch", "Kelp Patch", "补船"]
tags: [prefab, item]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/boatpatch.lua", "components/boatpatch.lua", "components/boatleak.lua", "components/repairable.lua", "components/repairer.lua", "actions.lua", "componentactions.lua", "components/playeractionpicker.lua", "prefabs/boat.lua", "recipes.lua", "prefabs/otterden.lua", "prefabs/merm.lua", "prefabs/mermking.lua", "prefabs/wurt.lua", "brains/waveyjoneshandbrain.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 船体修补物（船补丁 boatpatch、海带补丁 boatpatch_kelp）

## 快速判断

- 职责与适用：`prefabs/boatpatch.lua` 注册 2 个可拾取修补物：`boatpatch`（木补丁，[fn_wood L43-L91](../../../raw/current/scripts/prefabs/boatpatch.lua#L43-L91)）与 `boatpatch_kelp`（海带补丁，[fn_kelp L93-L132](../../../raw/current/scripts/prefabs/boatpatch.lua#L93-L132)）。都可堆叠 20、可燃可作燃料；核心消费是补船漏（REPAIR_LEAK）与直接修船体（REPAIR）；海带补丁额外可食、会腐坏。
- 优先使用：查补漏链、到期回漏、修船材料匹配读本页；漏点状态机正文在 [[reference/components/boatleak]]，组件接口在 [[reference/components/boatpatch]]；补漏贴图与到期碎屑 fx 在 [[reference/prefabs/boat_grass_erode|草／海带船侵蚀与垫料特效]]。
- 重要副作用：海带补丁进入 `repaired_kelp` 临时态并挂到期任务，到期漏点回 `med_leak`（[boatleak.lua L251-L260](../../../raw/current/scripts/components/boatleak.lua#L251-L260)）；海带补丁可作 VEGGIE 食物（负理智），鱼人与沃特有亲和抵消负面（[merm.lua L1108](../../../raw/current/scripts/prefabs/merm.lua#L1108)）。
- 修改边界：两注册差异全部在服务端构造段与 fn_kelp 覆盖段（`show_spoilage`、bank/build 替换在 `SetPristine` 之后）；海带修复强度与食用数值读配方缓存，模组改 `boatpatch_kelp` 配方在这几个取缓存的参数上首次生成后不生效（见陷阱）。本页不覆盖漏点产生与船体伤害机制（[[reference/components/hullhealth]]）。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `boatpatch` | `fn_wood` 直接构造：网络段（Transform／AnimState／SoundEmitter／MiniMapEntity／Network，`allow_action_on_impassable`＋`boat_patch` 标签、`MakeInventoryPhysics`、bank `boat_repair`／build `boat_repair_build` 播 `item`、`MakeInventoryFloatable("med", 0.1)`）→ `SetPristine` → 客户端返回 → master 段（boatpatch／repairer WOOD／stackable／inventoryitem／inspectable／burnable＋propagator／fuel／hauntable） | 服务端 `boatpatch` 组件 `patch_type` 保持默认 nil（[components/boatpatch.lua L3](../../../raw/current/scripts/components/boatpatch.lua#L3)），补漏进入永久 `repaired` 态 |
| `boatpatch_kelp` | `fn_kelp = fn_wood()` 复用木工厂返回值后覆盖：加 `show_spoilage`、bank/build 换 `boat_repair_kelp` → 第二次 mastersim 检查返回客户端 → master 覆盖段：`patch_type = "kelp"`、repairer 换 KELP、`SetBurnTime(SMALL_BURNTIME)`、加 bait／tradable／perishable／edible | 木段组件（boatpatch／stackable 20／fuel／hauntable 等）全部保留；`boat_patch` 标签在 fn_wood 网络段已加（客户端可见），组件构造会再补一次（[components/boatpatch.lua L5](../../../raw/current/scripts/components/boatpatch.lua#L5)） |

`boat_patch` 标签同时是 REPAIR_LEAK 动作入口的物品侧门禁；`allow_action_on_impassable` 让玩家可对不可通行（水面）目标出动作（[playeractionpicker.lua L463](../../../raw/current/scripts/components/playeractionpicker.lua#L463)、[L482](../../../raw/current/scripts/components/playeractionpicker.lua#L482)）。

## 修补动作与消费链

### REPAIR_LEAK：补漏

- **动作入口**：物品侧按 `boatpatch` 组件注册的 componentaction——持有物有 `boat_patch` 标签且目标有 `boat_leak` 标签才给 REPAIR_LEAK（[componentactions.lua L1070-L1073](../../../raw/current/scripts/componentactions.lua#L1070-L1073)）；动作定义 distance 2.5、不可按住连发（[actions.lua L560](../../../raw/current/scripts/actions.lua#L560)）。
- **结算**（[ACTIONS.REPAIR_LEAK.fn L4916-L4920](../../../raw/current/scripts/actions.lua#L4916-L4920) → [BoatLeak:Repair L62-L94](../../../raw/current/scripts/components/boatleak.lua#L62-L94)）：
  1. 补丁带 repairer 且当前平台有 repairable 且材料匹配 → `repairable:Repair` 给船体回血并消耗补丁；材料不匹配则跳过回血；船体满血（`health:GetPercent() >= 1`）时 `Repair` 直接返回 false 不回血（[repairable.lua L106-L114](../../../raw/current/scripts/components/repairable.lua#L106-L114)），此时补丁仍按第 2 条消耗；
  2. 未被修复消耗的补丁按堆叠移除一枚；
  3. `repair_state = "repaired_"..patch_type or "repaired"`（[L87-L88](../../../raw/current/scripts/components/boatleak.lua#L87-L88)）：木补丁 patch_type=nil → 永久 `repaired` 态（[L234-L235](../../../raw/current/scripts/components/boatleak.lua#L234-L235)，换 `boat_repair_build` 贴图，无到期任务）；海带补丁 → `repaired_kelp` 态（[L251-L260](../../../raw/current/scripts/components/boatleak.lua#L251-L260)），`SetRepairedTime` 中漏 `0.5` 天／小漏 `0.75` 天（[tuning.lua L8255-L8256](../../../raw/current/scripts/tuning.lua#L8255-L8256)），到期回 `med_leak`——到期 fx 链见 [[reference/prefabs/boat_grass_erode|草／海带船侵蚀与垫料特效]]。

### 直接修船体：REPAIR

- `repairer` 组件按数值挂 `health_<材料>` 标签（[repairer.lua L14-L16](../../../raw/current/scripts/components/repairer.lua#L14-L16)）；componentaction 要求目标带对应 `repairable_<材料>` 标签与 `healthrepairable`（[componentactions.lua L1679-L1697](../../../raw/current/scripts/componentactions.lua#L1679-L1697)）。
- 船侧 repairable 材料在共享 master 构造默认 WOOD（[boat.lua L572-L573](../../../raw/current/scripts/prefabs/boat.lua#L572-L573)），各船覆盖：草船 HAY（[L847](../../../raw/current/scripts/prefabs/boat.lua#L847)）、海獭窝船 KELP（[L1217](../../../raw/current/scripts/prefabs/boat.lua#L1217)）、冰面船 nil（[L1358](../../../raw/current/scripts/prefabs/boat.lua#L1358)）。因此：木补丁（`REPAIR_LOGS_HEALTH × 2` = 12.5×2 = 25 点，[boatpatch.lua L72-L74](../../../raw/current/scripts/prefabs/boatpatch.lua#L72-L74)、[tuning.lua L2118](../../../raw/current/scripts/tuning.lua#L2118)）可直修木质船；海带补丁（`REPAIR_KELP_HEALTH × 3` = 30 点，[boatpatch.lua L111-L112](../../../raw/current/scripts/prefabs/boatpatch.lua#L111-L112)、[tuning.lua L2122](../../../raw/current/scripts/tuning.lua#L2122)）可直修海獭窝船。修船回血用的是船的 `health` 组件（[boat.lua L582-L584](../../../raw/current/scripts/prefabs/boat.lua#L582-L584)），经 [repairable.lua L107-L114](../../../raw/current/scripts/components/repairable.lua#L107-L114) 结算。

### UNPATCH：拆除补丁（原版无玩家入口）

- 动作定义与结算在 [actions.lua L570](../../../raw/current/scripts/actions.lua#L570)、[L5376-L5405](../../../raw/current/scripts/actions.lua#L5376-L5405)，但快照内唯一调用者是海妖船长手 AI（[waveyjoneshandbrain.lua L107](../../../raw/current/scripts/brains/waveyjoneshandbrain.lua#L107)），componentactions 未向玩家提供该动作。
- 结算差异：临时补丁（有剩余修复时间，即海带补丁）直接回 `small_leak` 且不退还；永久补丁按贴图 build 生成 `boatpatch`（或缝纫带的 `sewing_tape`）弹出，并把船血扣减等额 `healthrepairvalue`。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：boatpatch | `patch_type` 载体（nil／"kelp"）＋`boat_patch` 标签；OnRemoveFromEntity 摘标签 | 服务端；同组件还挂在 sewing_tape、treegrowthsolution（[sewing_tape.lua L39](../../../raw/current/scripts/prefabs/sewing_tape.lua#L39)、[treegrowthsolution.lua L72-L73](../../../raw/current/scripts/prefabs/treegrowthsolution.lua#L72-L73)，对应 `repaired_tape`／`repaired_treegrowth` 态） | [[reference/components/boatpatch]]；[boatpatch.lua L69](../../../raw/current/scripts/prefabs/boatpatch.lua#L69) |
| 组件：repairer | 木：MATERIALS.WOOD＋25；海带：MATERIALS.KELP＋30，`boatrepairsound` 各配 | 服务端；数值随海带配方缓存缩放 | [[reference/components/repairer]]；[boatpatch.lua L71-L74](../../../raw/current/scripts/prefabs/boatpatch.lua#L71-L74)、[L111-L113](../../../raw/current/scripts/prefabs/boatpatch.lua#L111-L113) |
| 组件：stackable／inventoryitem／inspectable | `maxsize = TUNING.STACK_SIZE_MEDITEM`（20） | 两注册同值 | [boatpatch.lua L76-L80](../../../raw/current/scripts/prefabs/boatpatch.lua#L76-L80)、[tuning.lua L82](../../../raw/current/scripts/tuning.lua#L82)；[[reference/components/stackable]]、[[reference/components/inventoryitem]] |
| 组件：burnable／propagator／fuel | 木：`MED_BURNTIME`；海带覆盖 `SMALL_BURNTIME`；两者 `fuelvalue = MED_FUEL × 2` | 服务端；海带不改 fuel | [boatpatch.lua L82-L86](../../../raw/current/scripts/prefabs/boatpatch.lua#L82-L86)、[L115](../../../raw/current/scripts/prefabs/boatpatch.lua#L115)、[tuning.lua L1675-L1682](../../../raw/current/scripts/tuning.lua#L1675-L1682)；[[reference/components/burnable]]、[[reference/components/fuel]] |
| 组件：perishable（仅海带） | `PERISH_MED`（10 天）、耗尽替换 `spoiled_food`、`StartPerishing` | 服务端；`show_spoilage` 标签驱动新鲜度显示 | [boatpatch.lua L120-L123](../../../raw/current/scripts/prefabs/boatpatch.lua#L120-L123)、[tuning.lua L2048](../../../raw/current/scripts/tuning.lua#L2048)；[[reference/components/perishable]] |
| 组件：edible／bait／tradable（仅海带） | VEGGIE：`hungervalue = CALORIES_TINY × 3`、`healthvalue = -3`、`sanityvalue = -45`（负值按成本缓存缩放）；鱼人／沃特亲和抵消负面 | 服务端；玩家可食 | [boatpatch.lua L125-L129](../../../raw/current/scripts/prefabs/boatpatch.lua#L125-L129)、[tuning.lua L2021-L2058](../../../raw/current/scripts/tuning.lua#L2021-L2058)；[merm.lua L1108](../../../raw/current/scripts/prefabs/merm.lua#L1108)、[mermking.lua L638](../../../raw/current/scripts/prefabs/mermking.lua#L638)、[wurt.lua L651](../../../raw/current/scripts/prefabs/wurt.lua#L651)；[[reference/components/edible]] |
| API／数据：配方成本缓存 | `CACHED_KELP_RECIPE_COST` 首次生成时从 `AllRecipes.boatpatch_kelp` 汇总海带数量（非全海带或读不到回退默认 3） | 服务端、全局一次；驱动修复值与食物数值 | [boatpatch.lua L23-L41](../../../raw/current/scripts/prefabs/boatpatch.lua#L23-L41) |
| 关联 Prefab：`boat` 系 | REPAIR_LEAK 目标宿主＋repairable 材料（WOOD／HAY／KELP／nil） | 修船收益按材料匹配 | [[reference/components/hullhealth]]；[boat.lua L572-L573](../../../raw/current/scripts/prefabs/boat.lua#L572-L573) |
| 关联 Prefab：`fishingnetvisualizer`／`repaired_kelp_timeout_fx` | 两注册的 prefabs 依赖声明 | 木／海带各一；到期 fx 消费链已核验 | [boatpatch.lua L12-L20](../../../raw/current/scripts/prefabs/boatpatch.lua#L12-L20)；[[reference/prefabs/boat_grass_erode]] |
| 资源 | `boat_repair_build.zip`＋`boat_repair.zip`（木）、`boat_repair_kelp.zip`（海带） | assets 声明与 bank/build 使用已核验 | [boatpatch.lua L1-L10](../../../raw/current/scripts/prefabs/boatpatch.lua#L1-L10) |

## 获得路径

- **合成**：`boatpatch_kelp` = 海带×3（[recipes.lua L612](../../../raw/current/scripts/recipes.lua#L612)）；`boatpatch` = 木头×2＋蜂刺×1（[L613](../../../raw/current/scripts/recipes.lua#L613)）；均为 `TECH.NONE`。
- **掉落**：拆海獭窝掉 `boatpatch_kelp`（概率 0.75／0.50，[otterden.lua L34-L41](../../../raw/current/scripts/prefabs/otterden.lua#L34-L41)）。
- **活动代理**：龙年供奉 `yotd_boatpatch_proxy` 一次给 3 个 `boatpatch`（[recipes.lua L1117](../../../raw/current/scripts/recipes.lua#L1117)）。

## 常见陷阱与限制

- **误区：海带补丁是永久补漏**：`patch_type = "kelp"` 进入的 `repaired_kelp` 态挂了到期任务（中漏 0.5 天／小漏 0.75 天），到期回 `med_leak` 并落 `repaired_kelp_timeout_fx`；只有 patch_type 为 nil 的木补丁（及缝纫带、树长液）走无到期任务的永久态。
- **边界：补漏不等于修船**：REPAIR_LEAK 的船体回血要求补丁 repairer 材料与船 repairable 材料匹配，且船体未满血——海带补丁补木质船漏点时只堵漏不回血（[repairable.lua L154-L156](../../../raw/current/scripts/components/repairable.lua#L154-L156) 判材料不等直接返回 false；满血 [L106-L114](../../../raw/current/scripts/components/repairable.lua#L106-L114) 直接返回 false）；木补丁同理修不了草船（HAY）与海獭窝船（KELP）。材料不匹配或满血时补漏仍消耗一件补丁。
- **边界：海带数值绑定配方缓存**：`CACHED_KELP_RECIPE_COST` 在首个补丁生成时读一次配方（[boatpatch.lua L105-L107](../../../raw/current/scripts/prefabs/boatpatch.lua#L105-L107)），此后改配方对取缓存的参数——`healthrepairvalue`（修复量）与 `edible` 三数值——不生效；且配方一旦含非海带材料，缓存直接回退默认 3——模组自定义海带配方要在任何补丁生成前生效。
- **边界：UNPATCH 不是玩家动作**：拆除补丁目前只由海妖船长手 AI 触发；模组想给玩家提供"揭补丁"要自己接动作入口。
- **边界：客户端构造顺序**：kelp 的 `show_spoilage` 标签与 bank/build 替换发生在 `SetPristine` 之后（fn_wood 客户端段之后覆盖），客户端本地动画正常，但依赖"pristine 前标签"的客户端逻辑读不到 `show_spoilage` 的及时值。

## 验证记录

- 源码审阅：2026-11-08，steam-buildid:26335394。`prefabs/boatpatch.lua` 全文（双工厂、覆盖段、配方缓存）；`components/boatpatch.lua` 全文；消费链 boatleak（Repair／ChangeToRepaired／repaired 与 repaired_kelp 态／SetRepairedTime）、repairable（材料匹配与 health 分支）、repairer（标签挂载）、actions（REPAIR_LEAK／UNPATCH 定义与结算）、componentactions（boatpatch 与 repairer 入口）、boat（repairable 材料与 health 组件）、playeractionpicker（allow_action_on_impassable）、recipes／otterden／merm／mermking／wurt／waveyjoneshandbrain、tuning 数值核验；`chinese_s.po` 两 `NAMES.*` 键读取。未核验：MiniMapEntity 未设图标的具体表现、动画包内部内容、sewing_tape／treegrowthsolution 两兄弟补丁的完整页（仅核验其 boatpatch 组件接线）。
- 游戏运行：未运行。
- 结构检查：validate_page 通过（0 issue；89 行、15386 字节，standard 档内）；validate_source_links 通过（58 处引用、0 error／0 warning）。
- 外部复检修订（2026-11-08，R11，主代理回源码核实）：补漏回血补 `Repairable:Repair` 成功门禁——满血（GetPercent>=1）返回 false、材料不匹配跳过（repairable.lua L106-L114、L154-L156），此时补漏仍消耗一件补丁；「修船扣船体血」改「修船回血」；配方缓存不生效限定到取缓存的参数（healthrepairvalue 与 edible 三数值，boatpatch.lua L105-L129）。

返回 [[reference/prefabs/index|原版 Prefab 索引]]。
