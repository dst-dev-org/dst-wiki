---
title: "生物数据（scandata）"
created: "2026-11-10"
updated: "2026-11-10"
type: prefab
summary: "WX-78 生物扫描分析仪印出来的数据材料：扫描成功后按该物种的数据余额整叠生成，再拿去付 23 个电路模块与两台设备的配方；可燃、浮水不沉、40 一叠。"
aliases: ["scandata", "生物数据", "Bio Data", "扫描数据", "WX78 扫描材料", "电路模块材料"]
tags: [prefab, item]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/scandata.lua", "prefabs/wx78_scanner.lua", "stategraphs/SGwx78_scanner.lua", "components/dataanalyzer.lua", "wx78_moduledefs.lua", "prefabs/wx78.lua", "components/upgrademoduleowner.lua", "recipes.lua", "constants.lua", "tuning.lua", "standardcomponents.lua", "components/stackable.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# 生物数据（scandata）

## 快速判断

- 职责与适用：单一注册 `scandata`，物品态数据材料，官方简中名**生物数据**（英文 "Bio Data"）。它是 WX-78 电路模块与两台设备（电刑机控制器、灵体传输模块）配方的**通用材料**，本身不可食、不可修补、也不是燃料。[注册](../../../raw/current/scripts/prefabs/scandata.lua#L43)、[名称](../../../raw/current/scripts/languages/chinese_s.po#L348969-L348972)
- 优先使用：查怎么来、一次给多少 → [获取与产量](#获取与产量)；查谁消耗它 → [合成消费者](#合成消费者)；查堆叠／可燃／浮水 → [关联机制导航](#关联机制导航)；查数据余额与回复 → [[reference/components/dataanalyzer]]。
- 重要副作用：**可燃且能蔓延**，烧完按原堆叠数换成灰；作祟工厂虽叫 `MakeHauntableLaunchAndIgnite`，点燃分支已被整段注释，实际只弹飞。[燃烧](../../../raw/current/scripts/prefabs/scandata.lua#L36-L37)、[作祟实现](../../../raw/current/scripts/standardcomponents.lua#L1204-L1213)
- 修改边界：本页只写与本物品有关的扫描产物流程（何时生成、给多少、怎么烧）；余额取整与回复归 [[reference/components/dataanalyzer]]，扫描仪的移动、雷达与技能加成归 `wx78_scanner`（Wiki 待建）。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `scandata` | `Prefab("scandata", fn, assets)`：公共端建实体＋transform／anim／network、`MakeInventoryPhysics`、`SetRayTestOnBB(true)`、bank／build／`idle`、`MakeInventoryFloatable`、`pickupsound` → `SetPristine` → 仅主模拟挂组件 | 只有物品态；主模拟挂载 `inspectable` → `inventoryitem` → `stackable`（40）→ `MakeSmallBurnable`＋`MakeSmallPropagator` → `MakeHauntableLaunchAndIgnite`。[工厂](../../../raw/current/scripts/prefabs/scandata.lua#L6-L41) |

- 公共端：动画包只有 `anim/scandata.zip`，拾取音 `paper`，浮水参数 `MakeInventoryFloatable(inst, nil, 0.1)`——size 取默认 `"small"`、垂直偏移 0.1。[资源](../../../raw/current/scripts/prefabs/scandata.lua#L1-L4)、[浮水](../../../raw/current/scripts/prefabs/scandata.lua#L20-L22)、[参数含义](../../../raw/current/scripts/standardcomponents.lua#L1772-L1791)
- 没有 `edible`／`tradable`／`repairer`／`fuel`：不能吃、不能进贡、不能修东西、不能当柴烧。[组件段](../../../raw/current/scripts/prefabs/scandata.lua#L30-L38)

## 获取与产量

- 唯一来源是 WX-78 的**生物扫描分析仪**（`wx78_scanner`）：`scan_success` 状态第 21 帧（约 0.7 秒）调 `inst:SpawnData()`；生成时播 `fall`→`idle`、落点 X／Z 各随机 +0–1 单位（径向最大 √2≈1.41 单位，约 0.35 格）、22 帧后落地，并按结算量设堆叠。[状态机](../../../raw/current/scripts/stategraphs/SGwx78_scanner.lua#L199-L216)、[生成](../../../raw/current/scripts/prefabs/wx78_scanner.lua#L736-L755)
- 一次给多少由主人的 `dataanalyzer:SpendData(scan_id)` 决定：返回 `math.floor(余额)`，**只有 amount > 0 才生成**；某物种首次扫描会先按 `maxdata` 记满，所以首扫给满额，之后只剩回复出来的那部分。余额、取整、回复步长的完整机制归 [[reference/components/dataanalyzer]]。[结算](../../../raw/current/scripts/components/dataanalyzer.lua#L46-L60)
- `maxdata` 来自 `wx78_moduledefs.lua` 的 43 条 `AddCreatureScanDataDefinition`，取值 1–10：蜘蛛 2、兔子 2、龙蝇 10、蜂后 10（另有 6 条 special 定义，本页不展开）。[定义处](../../../raw/current/scripts/wx78_moduledefs.lua#L12-L18)、[蜘蛛](../../../raw/current/scripts/wx78_moduledefs.lua#L220)、[兔子](../../../raw/current/scripts/wx78_moduledefs.lua#L406)、[龙蝇](../../../raw/current/scripts/wx78_moduledefs.lua#L522)、[蜂后](../../../raw/current/scripts/wx78_moduledefs.lua#L1212)
- 选目标分两层：先按 `SCAN_CAN`／`SCAN_CANT` 两个**标签过滤表**查世界实体，再查 `wx78_moduledefs` 的 Lua 表；`SCAN_CAN` 是 Lua 列表名，给实体加同名字符串无效。[过滤表](../../../raw/current/scripts/prefabs/wx78_scanner.lua#L100-L103)、[世界查找](../../../raw/current/scripts/prefabs/wx78_scanner.lua#L480-L517)
- 余额不是硬门禁：循环先保留余额 > 0 的候选，同时收集「配方未学」的目标；这类目标非空就**整体替换候选**，所以零余额仍能扫（只教配方、不出数据）。零余额且配方已会时筛不出候选，只播 `ANNOUNCE_WX_SCANNER_FOUND_NO_DATA` 提示（同一个 15 秒计时窗口最多一次）。[候选与教学优先](../../../raw/current/scripts/prefabs/wx78_scanner.lua#L499-L542)
- 教配方在**收取**成功状态的扫描仪时触发：收取动作要求收取者是 `upgrademoduleowner`（`can_harvest`），`on_harvested` 在 `_module_recipe` 非空时 `teacher:Teach` 并换回 `wx78_scanner_item`；出数据则要 `SpendData > 0`。[记录](../../../raw/current/scripts/prefabs/wx78_scanner.lua#L574-L592)、[收取时教学](../../../raw/current/scripts/prefabs/wx78_scanner.lua#L1019-L1035)

## 合成消费者

| 用途组 | 配方与单次消耗（`scandata` 个数） | 证据 |
|---|---|---|
| 容量系模块 | `maxhealth` 2／`maxhealth2` 4／`maxhunger1` 2／`maxhunger` 3／`maxsanity1` 1／`maxsanity` 3 | [L753-L758](../../../raw/current/scripts/recipes.lua#L753-L758) |
| 蜂群模块 | `wx78module_bee` 8 | [L759](../../../raw/current/scripts/recipes.lua#L759) |
| 功能模块 | `music` 4／`movespeed` 2／`movespeed2` 6／`heat` 4／`cold` 4／`taser` 5／`nightvision` 4 | [L763-L769](../../../raw/current/scripts/recipes.lua#L763-L769) |
| 感知与堆叠 | `light` 2／`radar` 2／`light2` 6／`stacksize` 8 | [L770-L773](../../../raw/current/scripts/recipes.lua#L770-L773) |
| 其余模块 | `chess` 3／`digestion` 2／`screech` 4／`shielding` 4／`spin` 6 | [L777-L781](../../../raw/current/scripts/recipes.lua#L777-L781) |
| 两台设备 | `wx78_drone_zap_remote`（电刑机控制器）10——还要晶体管 1、`wagpunk_bits` 8，`builder_skill = "wx78_zapdrone_1"`；`wx78_gestalttrapper`（灵体传输模块）12——还要 `wagpunk_bits` 2，`builder_skill = "wx78_allegiance_lunar"`；两条都是 `TECH.NONE`，不走模块的通用门禁 | [L788-L789](../../../raw/current/scripts/recipes.lua#L788-L789) |

- 门槛：23 条模块配方用自定义科技 `TECH.ROBOTMODULECRAFT_ONE`（`{ ROBOTMODULECRAFT = 1 }`）并要求 `builder_tag = "upgrademoduleowner"`；该标签由 `upgrademoduleowner` 组件给持有者挂上。两台设备改用 `TECH.NONE`＋各自的 `builder_skill`（`wx78_zapdrone_1`／`wx78_allegiance_lunar`），不适用 `builder_tag` 门禁。[科技档](../../../raw/current/scripts/constants.lua#L1328)、[一条配方](../../../raw/current/scripts/recipes.lua#L753)、[标签](../../../raw/current/scripts/components/upgrademoduleowner.lua#L59)
- 按配方逐条数：材料需求区间 1–12，共 25 条配方消费 `scandata`（23 条模块＋2 条设备），无其它消费端。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`inventoryitem` | 可拾取／入栏；浮水不沉 | 仅主模拟，但 `floater` 在公共端挂 | [[reference/components/inventoryitem]]、[[reference/components/floater]]；[接线](../../../raw/current/scripts/prefabs/scandata.lua#L30-L34) |
| 组件：`stackable` | `maxsize = TUNING.STACK_SIZE_SMALLITEM` = 40；扫描按 amount 一次设好堆叠 | 仅主模拟 | [[reference/components/stackable]]；[定义](../../../raw/current/scripts/tuning.lua#L83)、[接线](../../../raw/current/scripts/prefabs/scandata.lua#L33-L34) |
| 组件：`burnable`／`propagator` | `MakeSmallBurnable` 默认烧 10 秒、`SetFXLevel(2)`；`MakeSmallPropagator` 给 3–5 格蔓延 | 仅主模拟；点燃后 | [[reference/components/burnable]]；[接线](../../../raw/current/scripts/prefabs/scandata.lua#L36-L37)、[10 秒](../../../raw/current/scripts/standardcomponents.lua#L164-L174)、[灰烬](../../../raw/current/scripts/standardcomponents.lua#L33-L41) |
| 组件：`hauntable` | `MakeHauntableLaunchAndIgnite`：launch 生效，ignite 被注释 | 仅主模拟；作祟时 | [[reference/components/hauntable]]；[接线](../../../raw/current/scripts/prefabs/scandata.lua#L38)、[实现](../../../raw/current/scripts/standardcomponents.lua#L1191-L1216) |
| API／数据：`dataanalyzer` | 主人身上记每物种累计数据，`SpendData` 取整给数量、`GetData` 给扫描仪筛选 | 仅 WX-78 本体；主模拟 | [[reference/components/dataanalyzer]]；[组件](../../../raw/current/scripts/components/dataanalyzer.lua#L32-L60) |
| API／数据：`wx78_moduledefs` | 43 条物种定义提供 `maxdata`／`module`／`recipename` | 两端都要有（纯 Lua 模块） | [导出表](../../../raw/current/scripts/wx78_moduledefs.lua#L1686-L1693) |
| SG：`SGwx78_scanner` | `scan_success` 第 21 帧调 `SpawnData`；`animover` 切 `turn_off` | 仅扫描仪实体 | [状态](../../../raw/current/scripts/stategraphs/SGwx78_scanner.lua#L199-L216) |
| 关联 Prefab：`wx78_scanner` | 唯一生成者；同时负责教模块配方 | 出数据要 `SpendData > 0`；教学走未学配方候选，零余额也能扫 | [生成](../../../raw/current/scripts/prefabs/wx78_scanner.lua#L736-L755)（Wiki 待建） |
| 关联 Prefab：`ash` | 可燃耗尽后的产物，按原堆叠数生成 | 烧完时 | [产物](../../../raw/current/scripts/standardcomponents.lua#L33-L41) |
| 资源：`anim/scandata.zip` | bank＝build＝`scandata`，动画 `idle`，另有扫描仪播放的 `fall` | 公共端／扫描仪 | [assets](../../../raw/current/scripts/prefabs/scandata.lua#L1-L4) |

## 典型接入

### 给自定义生物补一条扫描定义

- 前提与选择：服务端与客户端都要执行（`modmain.lua` 顶层即可）；`wx78_moduledefs` 导出 `AddCreatureScanDataDefinition`，但**注册定义还不够**——目标实体必须先具备 `SCAN_CAN` 里的某个标签（如 `animal`／`monster`／`smallcreature`），且不带 `DECOR`／`FX`／`INLIMBO`／`NOCLICK`，否则连候选都进不来。
- 生效与撤销：定义写进模块内的 `scandata_definitions` 表，**同 prefab 名重复调用会覆盖**；撤掉调用即回到「无定义」——此时扫描仪找不到该目标，`SpendData` 也只会返回 0。
- 原版依据：`module` 决定要教的配方名（`"wx78module_"..module`），`maxdata` 决定首扫给多少与回复步长。[接口](../../../raw/current/scripts/wx78_moduledefs.lua#L12-L18)、[配方名推导](../../../raw/current/scripts/prefabs/wx78_scanner.lua#L104-L109)

```lua
-- modmain.lua：让自定义生物也能被生物扫描分析仪扫出数据
local WX78ModuleDefs = GLOBAL.require("wx78_moduledefs")
-- 目标 prefab 需自带 SCAN_CAN 里的标签（如 animal）且无 DECOR/FX/INLIMBO/NOCLICK
-- 参数：prefab 名、要教的模块名（对应 wx78module_<module> 配方）、数据上限
WX78ModuleDefs.AddCreatureScanDataDefinition("my_creature", "maxhealth", 4)
```

- 调整与验证：`maxdata` 建议 1–10（对齐原版区间）；取 4 时首扫给 4 个、之后每 30 秒回 0.25 个、约 8 分钟（16 次 × 30 秒）回满。示例里 `my_creature` 是占位名；未运行游戏。

## 常见陷阱与限制

- **误区：可燃就能当燃料。** `MakeSmallBurnable` 只挂 `burnable`／`propagator`，本文件没有 `fuel`，塞进火堆只会把自己烧成灰。[组件段](../../../raw/current/scripts/prefabs/scandata.lua#L36-L38)
- **边界：一整叠烧掉只换回等量灰。** `DefaultBurntFn` 用原堆叠数生成 `ash`，40 个生物数据烧完就是 40 个灰，材料全损。[灰烬](../../../raw/current/scripts/standardcomponents.lua#L33-L41)
- **边界：`amount > 0` 才生成。** 余额不足 1 点时 `SpendData` 返回 0，扫描成功也看不到掉落；连续扫同一物种会明显变慢。[生成条件](../../../raw/current/scripts/prefabs/wx78_scanner.lua#L739-L742)、[取整](../../../raw/current/scripts/components/dataanalyzer.lua#L54-L58)
- **边界：作祟不点燃。** `MakeHauntableLaunchAndIgnite` 的 ignite 段被 `#HAUNTFIX` 注释，作祟只会把物品弹飞。[实现](../../../raw/current/scripts/standardcomponents.lua#L1204-L1213)
- **未核验**：未运行游戏。只有**出数据**要正余额（`SpendData > 0`）；配方未学时仍可扫来教学，不必等余额回到 1 点。无数据提示按 15 秒计时窗口限频，具体等待秒数未实测。

## 验证记录

- 源码审阅：2026-11-10，Steam buildid 26474873。通读 `prefabs/scandata.lua` 全文；核验 `prefabs/wx78_scanner.lua` 的候选筛选、`SpawnData`、`OnSuccessfulScan`、教配方段与 `SGwx78_scanner` 的 `scan_success`；`components/dataanalyzer.lua` 全文（`GetData`／`SpendData`／`process_data_increase`）；`prefabs/wx78.lua` 的 `StartDataRegen`；`wx78_moduledefs.lua` 的定义函数、导出表与 43 条物种定义；`recipes.lua` 的 25 条消费配方与 `constants.lua` 的 `ROBOTMODULECRAFT_ONE`；`upgrademoduleowner.lua` 的标签；`standardcomponents.lua` 的 `MakeSmallBurnable`／`MakeInventoryFloatable`／`MakeHauntableLaunchAndIgnite`／`DefaultBurntFn`；`tuning.lua` 的 `STACK_SIZE_SMALLITEM`；简中 `.po` 名称键。
  未覆盖：扫描仪的技能加成与雷达范围、6 条 special 扫描定义、各模块的实际效果、模组注入与调试命令。数据余额的取整与回复细节已在 [[reference/components/dataanalyzer]] 展开，本页只引用。
- 游戏运行：未运行。首扫满额、10 秒燃烧与 40 堆叠均为源码推导，非运行测量。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 见交付回报。
- 外部复检修订（第二轮）：2026-11-10，R05 余项：导航把「数据 > 0」限定到出数据、教学候选可零余额（L69），验证段删除「余额回到 1 点才能再扫」并补 15 秒限频（L97、L41）。
- 外部复检修订：2026-11-10，R03／R05／R07 条目：①示例前提补 `SCAN_CAN`／`SCAN_CANT` 与「SCAN_CAN 是列表名」，代码加注释（L77、代码）；②候选改写为「未学配方优先替换候选、零余额仍可扫；出数据看 `SpendData > 0`、教配方在收取时」（L40-L42）；③落点改 X／Z 各 +0–1 单位、径向 ≤√2（L37）；④两台设备补 `builder_skill` 与 `TECH.NONE`（L52、L54）。
- 复审修订：2026-11-10，P3 条目：①「每 30 秒回 0.25 个」后的满槽时间由 4 分钟改为 8 分钟（16 次 × 30 秒，与 dataanalyzer 页一致，L87）；②摘要「不浮沉」改「浮水不沉」（L6）；③落点「0–1 格」改 0–1 世界单位（不到 1/4 格，L37）。

## 相关链接

- 数据余额机制：[[reference/components/dataanalyzer]]
- 同批材料：[[reference/prefabs/wagpunk-bits]]、[[reference/prefabs/wagdrone-parts]]
- 组件：[[reference/components/stackable]]、[[reference/components/burnable]]、[[reference/components/inventoryitem]]

返回 [[reference/prefabs/index|Prefab 索引]]。
