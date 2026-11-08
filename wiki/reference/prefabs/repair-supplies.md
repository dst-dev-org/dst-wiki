---
title: "修补用品（可靠的胶布 sewing_tape、锻造修理套件 lunarplant_kit／voidcloth_kit／wagpunkbits_kit）"
created: "2026-11-10"
updated: "2026-11-10"
type: prefab
summary: "prefabs/sewing_tape.lua 与 prefabs/forge_repair_kits.lua 的四件修补物：可靠的胶布（sewing_tape）以 boatpatch＋sewing 双组件分别补船漏与回补 needssewing 物品的用量耐久；三个锻造修理套件（亮茄／虚空／自动修理机）共用 forgerepair 材料通道，对受损的裂隙装备按 armor／finiteuses／fueled 优先级一次回满并消耗 1 层。"
aliases: ["sewing_tape", "lunarplant_kit", "voidcloth_kit", "wagpunkbits_kit", "可靠的胶布", "亮茄修补套件", "虚空修补套件", "自动修理机", "修理包", "修补套件", "缝纫胶带"]
tags: [prefab, item]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/sewing_tape.lua", "prefabs/forge_repair_kits.lua", "components/sewing.lua", "components/forgerepair.lua", "components/forgerepairable.lua", "components/repairable.lua", "components/repairer.lua", "components/fueled.lua", "standardcomponents.lua", "constants.lua", "componentactions.lua", "actions.lua", "recipes.lua", "tuning.lua", "prefabs/sewingkit.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 修补用品（可靠的胶布 sewing_tape、锻造修理套件 lunarplant_kit／voidcloth_kit／wagpunkbits_kit）

## 快速判断

- 职责与适用：本页收录原版两份修补物文件共 4 个可拾取注册。`sewing_tape`（可靠的胶布，[sewing_tape.lua L11-L57](../../../raw/current/scripts/prefabs/sewing_tape.lua#L11-L57)）走 boatpatch＋sewing 双通道；`lunarplant_kit`／`voidcloth_kit`／`wagpunkbits_kit`（[forge_repair_kits.lua L56-L58](../../../raw/current/scripts/prefabs/forge_repair_kits.lua#L56-L58)）由同一工厂 `MakeKit` 生成，走 forgerepair 材料通道。装备侧（裂隙护甲、工具、帽子等）的修理协议见 [[reference/prefabs/armor-rift]]、[[reference/prefabs/lunarplant-tools]]。
- 优先使用：查"修什么、耗多少"读本页"修理通道总表"；改动手工修理件直接挂 `sewing` 组件（典型接入）；改动裂隙装备修理按 [[reference/components/forgerepairable]] 的协议。船漏修补链（sewing_tape 的 boatpatch 侧）见 [[reference/prefabs/boat-patches]]，本页不重复其结论。
- 重要副作用：sewing 每次成功缝补都会给修理者发 `sewing_kit` 成就（[sewing.lua L22](../../../raw/current/scripts/components/sewing.lua#L22)）；forgerepair 修复成功后向修理者推 `repair` 事件（[forge_repair_kits.lua L1-L3](../../../raw/current/scripts/prefabs/forge_repair_kits.lua#L1-L3)）。
- 修改边界：两个文件均为服务端构造分界（`SetPristine` 后返回客户端）；胶布的 `tape`／`boat_patch` 标签在 pristine 段（[sewing_tape.lua L24-L27](../../../raw/current/scripts/prefabs/sewing_tape.lua#L24-L27)），套件的 `forgerepair_<材料>` 标签由组件 setter 镜像（[forgerepair.lua L1-L8](../../../raw/current/scripts/components/forgerepair.lua#L1-L8)）。本页不覆盖装备侧破损／重建动画（归各装备页）。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `sewing_tape`（可靠的胶布） | 独立 `fn()`：网络段（bank `tape`／build `sewing_tape`、`tape`＋`boat_patch` 标签、浮水 small、scrapbook `SEWINGTAPE`）→ `SetPristine` → master 段（boatpatch `patch_type="tape"`、stackable SMALLITEM 40、sewing、inspectable、inventoryitem、hauntable） | 双通道全在 master 段接线：[L39-L47](../../../raw/current/scripts/prefabs/sewing_tape.lua#L39-L47)；`boat_patch` 标签为 REPAIR_LEAK 物品侧门禁（见 [[reference/prefabs/boat-patches#修补动作与消费链|船体修补物]]） |
| `lunarplant_kit`（亮茄修补套件） | `MakeKit(name, material)` 共用工厂：bank/build＝注册名动画、浮水 `small` 0.2 `{1.4,1,1}` → `SetPristine` → master 段（stackable LARGEITEM 10、forgerepair＋材料、inspectable、inventoryitem、hauntable） | 材料参数 `FORGEMATERIALS.LUNARPLANT`＝`"lunarplant"`（[L56-L58](../../../raw/current/scripts/prefabs/forge_repair_kits.lua#L56-L58)、[constants.lua L1962-L1967](../../../raw/current/scripts/constants.lua#L1962-L1967)）；scrapbook_specialinfo＝`"LUNARPLANTKIT"`（[L26-L30](../../../raw/current/scripts/prefabs/forge_repair_kits.lua#L26-L30)） |
| `voidcloth_kit`（虚空修补套件） | 同上工厂 | 材料 `FORGEMATERIALS.VOIDCLOTH`＝`"voidcloth"`；scrapbook_specialinfo＝`"VOIDCLOTHKIT"`（[L26-L30](../../../raw/current/scripts/prefabs/forge_repair_kits.lua#L26-L30)） |
| `wagpunkbits_kit`（自动修理机） | 同上工厂 | 材料 `FORGEMATERIALS.WAGPUNKBITS`＝`"wagpunk_bits"`——注意材料值与注册名拼写不同（`bits` 前有下划线）；无 scrapbook_specialinfo 分支（[L26-L30](../../../raw/current/scripts/prefabs/forge_repair_kits.lua#L26-L30)） |

文件名与注册名不一一对应：`forge_repair_kits.lua` 一个文件返回 3 个注册（[L53-L58](../../../raw/current/scripts/prefabs/forge_repair_kits.lua#L53-L58)），页内一律以 `Prefab(name, fn, assets)` 的注册名为准。

## 修理通道总表（共享规则）

原版"修补物 → 受损目标"共四条通道；本批 4 个注册占两条，其余两条列出对照，跨页断言以组件页与 B01 页为权威来源：

| 通道 | 修理物侧组件 | 目标侧组件 | 结算与消耗 | 代表物品 |
|---|---|---|---|---|
| repairer／repairable | `repairer`：`repairmaterial`＋各数值字段，按数值挂 `work_`／`health_`／`freshen_`／`finiteuses_<材料>` 标签（[repairer.lua L8-L24](../../../raw/current/scripts/components/repairer.lua#L8-L24)） | `repairable`：材料标签 `repairable_<材料>`（[repairable.lua L1-L8](../../../raw/current/scripts/components/repairable.lua#L1-L8)） | `Repair:Repair` 先比材料（[L154-L156](../../../raw/current/scripts/components/repairable.lua#L154-L156)），再按 health→work→perish→finiteuses 顺序只恢复第一类（[L106-L146](../../../raw/current/scripts/components/repairable.lua#L106-L146)），满值直接返回 false；四类数值与百分比字段并存（health 有 value＋percent 两档） | 船补丁（B01 已核，见 [[reference/prefabs/boat-patches#修补动作与消费链|船体修补物]]）、噩梦燃料 |
| sewing／fueled | `sewing`：`repair_value`（单位＝fueled 的燃料秒），可选 `onsewn` 回调（[sewing.lua L1-L4](../../../raw/current/scripts/components/sewing.lua#L1-L4)） | `fueled`：段类型 `FUELTYPE.USAGE` 且 `currentfuel < maxfuel` 且非 `no_sewing` 时挂 `needssewing` 标签（[fueled.lua L7-L13](../../../raw/current/scripts/components/fueled.lua#L7-L13)），满时摘标（[L38-L46](../../../raw/current/scripts/components/fueled.lua#L38-L46)） | `DoSewing` 只查 `needssewing` 标签即执行 `fueled:DoDelta`（钳制在 `[0, maxfuel]`，[fueled.lua L305-L308](../../../raw/current/scripts/components/fueled.lua#L305-L308)），随后按堆叠扣 1（[sewing.lua L6-L27](../../../raw/current/scripts/components/sewing.lua#L6-L27)） | `sewing_tape`（`TUNING.SEWING_TAPE_REPAIR_VALUE`＝5 天，[tuning.lua L2365-L2368](../../../raw/current/scripts/tuning.lua#L2365-L2368)）、缝纫包（finiteuses 5 次，[sewingkit.lua L33](../../../raw/current/scripts/prefabs/sewingkit.lua#L33)） |
| forgerepair／forgerepairable | `forgerepair`：`SetRepairMaterial` 镜像挂 `forgerepair_<材料>` 标签（[forgerepair.lua L1-L8](../../../raw/current/scripts/components/forgerepair.lua#L1-L8)） | `forgerepairable`：`SetRepairable` 门禁＋材料标签（[forgerepairable.lua L1-L20](../../../raw/current/scripts/components/forgerepairable.lua#L1-L20)） | `OnRepair` 按 armor→finiteuses→fueled 优先级把第一类受损组件 `SetPercent(1)` 一次回满（[forgerepair.lua L28-L45](../../../raw/current/scripts/components/forgerepair.lua#L28-L45)）；成功后优先耗自身 finiteuses、否则堆叠扣 1、否则移除（[L47-L54](../../../raw/current/scripts/components/forgerepair.lua#L47-L54)）；无比例、无部分修复 | 本页三个 `*_kit` |
| boatpatch／boatleak | `boatpatch`：`patch_type` 区分补丁态 | `boatleak`：漏点状态机 | 补漏＋材料匹配时船体回血；`sewing_tape` 的 `patch_type="tape"` 走无到期任务的永久 `repaired_tape` 态 | [[reference/prefabs/boat-patches]]（B01 已收口，本页不重写） |

材料标签一律用**材料枚举值全小写**：`MATERIALS`／`FORGEMATERIALS` 的键是枚举名、值才是标签后缀（`health_wood`、`forgerepair_wagpunk_bits`），写 `_WOOD` 形式与实际标签不符（[constants.lua L1940-L1967](../../../raw/current/scripts/constants.lua#L1940-L1967)）。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：boatpatch＋sewing（仅胶布） | 双通道并行：补船漏（REPAIR_LEAK）与回补 USAGE 耐久（SEW） | 服务端 master 段；`patch_type="tape"` | [[reference/prefabs/boat-patches]]、[[reference/components/boatpatch]]、[[reference/components/sewing]]；[sewing_tape.lua L39-L47](../../../raw/current/scripts/prefabs/sewing_tape.lua#L39-L47) |
| 组件：forgerepair（三个套件） | `SetRepairMaterial`＋`SetOnRepaired`（推 `repair` 事件） | 服务端；材料值 `lunarplant`／`voidcloth`／`wagpunk_bits` | [[reference/components/forgerepair]]；[forge_repair_kits.lua L41-L43](../../../raw/current/scripts/prefabs/forge_repair_kits.lua#L41-L43) |
| 动作：SEW | 候选＝目标带 `needssewing`（[componentactions.lua L1708-L1716](../../../raw/current/scripts/componentactions.lua#L1708-L1716)）；`fn` 调 `DoSewing`（[actions.lua L1015-L1020](../../../raw/current/scripts/actions.lua#L1015-L1020)）；`tape` 标签选 "PATCH" 台词（[L1009-L1012](../../../raw/current/scripts/actions.lua#L1009-L1012)） | 服务端结算；候选由标签驱动 | [[reference/components/sewing]] |
| 动作：REPAIR（forgerepairable 分支） | 候选＝右键＋双方材料标签匹配＋目标不漂浮＋未骑乘搬重（[componentactions.lua L1279-L1299](../../../raw/current/scripts/componentactions.lua#L1279-L1299)）；`fn` 分派 forgerepairable:Repair（[actions.lua L1000-L1005](../../../raw/current/scripts/actions.lua#L1000-L1005)） | 右键专用；repairable 分支（`repairer` 物品）另行走 REPAIR 前段 | [[reference/components/forgerepairable#关键流程]] |
| 修理目标（装备侧） | lunarplant 六件：armor_lunarplant、lunarplanthat（[hats.lua L3923](../../../raw/current/scripts/prefabs/hats.lua#L3923)）、pickaxe／shovel／staff／sword_lunarplant；voidcloth 七件：armor_voidcloth（[armor_voidcloth.lua L141](../../../raw/current/scripts/prefabs/armor_voidcloth.lua#L141)）、voidclothhat（[hats.lua L4144](../../../raw/current/scripts/prefabs/hats.lua#L4144)）、saddle_shadow（[saddle.lua L402](../../../raw/current/scripts/prefabs/saddle.lua#L402)）、shadow_battleaxe、voidcloth_umbrella、voidcloth_boomerang、voidcloth_scythe；wagpunk_bits 两件：`armorwagpunk`（[armor_wagpunk.lua L397](../../../raw/current/scripts/prefabs/armor_wagpunk.lua#L397)）、`wagpunkhat`（[hats.lua L4657](../../../raw/current/scripts/prefabs/hats.lua#L4657)） | 均经 `MakeForgeRepairable` 接线（[standardcomponents.lua L1851-L1884](../../../raw/current/scripts/standardcomponents.lua#L1851-L1884)）；hats.lua 的 `MakeHat("lunarplant"/"voidcloth"/"wagpunk")` 产物即 `*hat`（[prefabname＝name.."hat"](../../../raw/current/scripts/prefabs/hats.lua#L16-L20)） | 装备侧细节见 [[reference/prefabs/armor-rift]]、[[reference/prefabs/lunarplant-tools]]、[[reference/prefabs/voidcloth]] |
| 关联 Prefab：`sewingkit`（缝纫包） | 同走 sewing 通道的对照工具：finiteuses 5 次而非堆叠 | `SEWINGKIT_USES`＝5（[tuning.lua L2365](../../../raw/current/scripts/tuning.lua#L2365)） | [sewingkit.lua L33](../../../raw/current/scripts/prefabs/sewingkit.lua#L33)、[L40](../../../raw/current/scripts/prefabs/sewingkit.lua#L40) |
| 资源 | `sewing_tape.zip`＋`boat_repair_tape_build.zip`（胶布）；三套件各用 `anim/<注册名>.zip`（工厂内拼名） | assets 声明与 bank/build 使用一致 | [sewing_tape.lua L1-L5](../../../raw/current/scripts/prefabs/sewing_tape.lua#L1-L5)、[forge_repair_kits.lua L6-L9](../../../raw/current/scripts/prefabs/forge_repair_kits.lua#L6-L9) |

## 典型接入

### 把自定义物品接入 forgerepair 修理通道

- 前提与选择：模组要造"对某类装备回满耐久"的一次性修理件时，直接挂 `forgerepair` 并给目标 `MakeForgeRepairable`；无需写修复数值，回满语义由组件固定。
- 生效与撤销：标签由组件 setter 镜像维护（换材料自动换标签）；目标 `repairable=false`（满耐久）时候选动作消失。

```lua
-- modmain.lua：注册一个修 "gem" 材料装备的修理件（网络段＋服务端段）
local function fn()
    local inst = CreateEntity()
    inst.entity:AddTransform()
    inst.entity:AddAnimState()
    inst.entity:AddNetwork()
    MakeInventoryPhysics(inst)
    inst.entity:SetPristine()
    if not TheWorld.ismastersim then
        return inst
    end
    inst:AddComponent("stackable")
    inst.components.stackable.maxsize = TUNING.STACK_SIZE_LARGEITEM
    inst:AddComponent("forgerepair")
    -- 材料值即标签后缀：目标必须 MakeForgeRepairable(inst, "gem") 才能匹配
    inst.components.forgerepair:SetRepairMaterial("gem")
    inst.components.forgerepair:SetOnRepaired(function(inst, target, doer)
        doer:PushEvent("repair")
    end)
    inst:AddComponent("inspectable")
    inst:AddComponent("inventoryitem")
    return inst
end

AddPrefab("my_repairkit")
```

- 调整与验证：材料字符串任意，但目标侧与修理物侧必须同值；armor／finiteuses／fueled 三分支按组件存在性取优先级，目标同时有多组件时只回满第一个（[forgerepair.lua L30-L45](../../../raw/current/scripts/components/forgerepair.lua#L30-L45)）。手工修理件（sewing 通道）把 `forgerepair` 两行换成 `sewing`＋`repair_value`（单位为 fueled 秒），目标物需 `fueled` USAGE 段。

## 获得路径

- **合成**：`sewing_tape`＝蜘蛛丝×1＋割下的草×3（`TECH.NONE`，`builder_tag="handyperson"`，[recipes.lua L303](../../../raw/current/scripts/recipes.lua#L303)）；`lunarplant_kit`＝纯粹辉煌×1＋亮茄外壳×1（月捷锻造二级，[L855](../../../raw/current/scripts/recipes.lua#L855)）；`voidcloth_kit`＝纯粹恐惧×1＋虚空布×1（暗影锻造二级，[L871](../../../raw/current/scripts/recipes.lua#L871)）；`wagpunkbits_kit`＝旺普零件×1＋晶体管×1（`TECH.LOST`，[L863](../../../raw/current/scripts/recipes.lua#L863)）。
- 装备侧修理量与目标清单的掉落／建造来源归各装备页，本页不重复。

## 常见陷阱与限制

- **误区：修理套件能按比例修。** forgerepair 通道没有数值字段，只有"有损→一次回满"；想要按点数修要走 `repairer` 通道（船补丁类）。
- **边界：材料标签拼写。** `wagpunkbits_kit` 的标签是 `forgerepair_wagpunk_bits`（材料值），不是 `forgerepair_wagpunkbits`；候选动作按 `FORGEMATERIALS` 值逐一比对（[componentactions.lua L1290-L1294](../../../raw/current/scripts/componentactions.lua#L1290-L1294)）。
- **边界：sewing 满耐久无动作。** `needssewing` 标签在 fueled 满时被摘除（[fueled.lua L38-L46](../../../raw/current/scripts/components/fueled.lua#L38-L46)），胶布对满耐久衣物不出现 SEW 动作；但 `DoSewing` 本身不检查剩余量，模组若手动保留标签可以超额灌满（DoDelta 钳制到 maxfuel，不会溢出）。
- **边界：套件无 scrapbook 特殊面板≠无功能。** 只有 lunarplant_kit／voidcloth_kit 注册了 `scrapbook_specialinfo`；`wagpunkbits_kit` 走通用物品图鉴展示（[forge_repair_kits.lua L26-L30](../../../raw/current/scripts/prefabs/forge_repair_kits.lua#L26-L30)）。

## 验证记录

- 源码审阅：2026-11-10，steam-buildid:26474873。`forge_repair_kits.lua` 全文（58 行，三注册＋工厂）、`sewing_tape.lua` 全文；`forgerepair`／`forgerepairable`／`sewing` 组件全文；`repairable` 四通道段与材料门禁、`repairer` 标签镜像；`fueled` 的 `needssewing` 生命周期与 DoDelta 钳制；`constants.lua` MATERIALS／FORGEMATERIALS；REPAIR／SEW 动作定义与 componentactions 两处候选；`MakeForgeRepairable` 及 15 处调用点（目标清单按材料分组核对）；sewingkit 对照；tuning 堆叠与修数值；`chinese_s.po` 四个 `NAMES.*` 键实显。船漏侧仅复用 B01 结论（[[reference/prefabs/boat-patches]]），未重新展开 boatleak。
- 未覆盖：套件动画包内部、`scrapbook_specialinfo` 的图鉴面板行为、装备侧各页细节。
- 游戏运行：未运行。

返回 [[reference/prefabs/index|原版 Prefab 索引]]。
