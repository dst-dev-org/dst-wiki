---
title: "废料（wagpunk_bits）"
created: "2026-11-10"
updated: "2026-11-10"
type: prefab
summary: "Wagstaff 系通用零件：疯猪、垃圾堆、被丢弃的垃圾与残骸都掉；既是 18 条配方原料，也能修废料墙与棋子。"
aliases: ["wagpunk_bits", "废料", "Scrap", "Wagstaff 零件", "W.A.R.B.I.S. 材料", "Wagstaff 材料"]
tags: [prefab, item]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/wagpunk_bits.lua", "prefabs/daywalker2.lua", "prefabs/wagstaff_machinery.lua", "prefabs/wagboss_robot.lua", "prefabs/junk_pile_common.lua", "prefabs/fence.lua", "prefabs/fence_electric.lua", "prefabs/monkeyisland_portal_debris.lua", "prefabs/inspectaclesbox.lua", "prefabs/walls.lua", "prefabs/chessjunk.lua", "prefabs/wagdrone_common.lua", "prefabs/forge_repair_kits.lua", "prefabs/armor_wagpunk.lua", "components/repairer.lua", "components/repairable.lua", "brains/molebrain.lua", "components/trap.lua", "components/inspectaclesparticipant.lua", "components/stackable.lua", "componentactions.lua", "constants.lua", "recipes.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# 废料（wagpunk_bits）

## 快速判断

- 职责与适用：单一注册 `wagpunk_bits`，物品态**通用零件／修理材料**，官方简中名**废料**（英文 "Scrap"）。Wagstaff 系内容的原料，也是当前快照里能修废料墙与废料棋子的材料。[注册](../../../raw/current/scripts/prefabs/wagpunk_bits.lua#L55)、[名称](../../../raw/current/scripts/languages/chinese_s.po#L351449-L351452)
- 优先使用：查从哪来 → [获取](#获取)；查配方消耗 → [合成与其他消费端](#合成与其他消费端)；查堆叠／修理数值／诱饵标签 → [关联机制导航](#关联机制导航)。
- 重要副作用：它带 `molebait` 标签并有 `bait` 组件，**放地上会被鼹鼠偷**；`repairer` 让它能右击修补 `MATERIALS.GEARS` 类目标（+80 或 +1 工作面）。[诱饵标签](../../../raw/current/scripts/prefabs/wagpunk_bits.lua#L20-L24)、[修理接线](../../../raw/current/scripts/prefabs/wagpunk_bits.lua#L45-L48)
- 修改边界：本页管构造、掉落与配方消耗；各产物行为归各自页面，鼹鼠偷饵 AI 归 `brains/molebrain.lua`。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `wagpunk_bits` | `Prefab("wagpunk_bits", fn, assets)`：公共端建实体＋transform／anim／sound／network、`MakeInventoryPhysics`、bank／build／`idle`、`AddTag("molebait")`、浮水 `("med", nil, 0.7)` → `SetPristine` → 主模拟挂组件 | 只有物品态；主模拟挂载 `bait` → `inspectable` → `inventoryitem` → `stackable`（40）→ `repairer` → `MakeHauntableLaunch`。[工厂](../../../raw/current/scripts/prefabs/wagpunk_bits.lua#L6-L53) |

- 标签加在**公共端**（鼹鼠与陷阱检索两端都要用）；`bait` 只加服务端。[标签](../../../raw/current/scripts/prefabs/wagpunk_bits.lua#L22)、[bait](../../../raw/current/scripts/prefabs/wagpunk_bits.lua#L32)
- 文件里有一段**被注释掉的 `edible`**（`FOODTYPE.GEARS`、治疗／饥饿／理智都用 Huge 档）；注释不是初始化，本注册**没有** `edible`，不能吃也不能喂齿轮食性生物。[注释段](../../../raw/current/scripts/prefabs/wagpunk_bits.lua#L39-L43)

## 获取

| 来源 | 数量与概率 | 证据 |
|---|---|---|
| 拾荒疯猪 `daywalker2` | 掉落表 4 条 `1` ＋ 1 条 `0.5` → **每次 4–5 个**；裂隙开启时另对 4 格内玩家随机给 1 张未学的 W.A.R.B.I.S.／战备蓝图，未开启则改给 1 台自动修理机 | [掉落表](../../../raw/current/scripts/prefabs/daywalker2.lua#L50-L54)、[蓝图逻辑](../../../raw/current/scripts/prefabs/daywalker2.lua#L1025-L1057) |
| 被丢弃的垃圾 `wagstaff_machinery` | 锤掉后 `1.00` ＋ `0.75` → 期望 1.75 个；另有 40% 概率给 1 张玩家未学的 W.A.R.B.I.S.／战备蓝图（只统计 1 格内玩家；都会则从四项随机） | [掉落表](../../../raw/current/scripts/prefabs/wagstaff_machinery.lua#L24-L25)、[蓝图逻辑](../../../raw/current/scripts/prefabs/wagstaff_machinery.lua#L88-L118)、[40%](../../../raw/current/scripts/tuning.lua#L8048-L8049) |
| 战争瓦器人 `wagboss_robot` | `1.0` ＋ `0.5` → 期望 1.5 个 | [掉落表](../../../raw/current/scripts/prefabs/wagboss_robot.lua#L29-L30) |
| 垃圾堆 `junk_pile` | 挖开时在 10 项权重表里取 1 项：`wagpunk_bits` 权重 8／总计 32 → **25%** | [权重表](../../../raw/current/scripts/prefabs/junk_pile_common.lua#L15-L24)、[抽取](../../../raw/current/scripts/prefabs/junk_pile_common.lua#L64-L66) |
| 废料墙 `wall_scrap` | 拆除时 `loot = "wagpunk_bits"`、`maxloots = 1` | [墙定义](../../../raw/current/scripts/prefabs/walls.lua#L414-L424) |
| 麻刺节点 `fence_electric` | 毁坏时 `SetLoot` 直接给 1 个废料＋1 个月玻璃（整表必掉，不是概率） | [掉落表](../../../raw/current/scripts/prefabs/fence_electric.lua#L14)、[接线](../../../raw/current/scripts/prefabs/fence_electric.lua#L177) |
| 损坏的机器 `monkeyisland_portal_debris` | 锤掉时 `0.85` | [掉落表](../../../raw/current/scripts/prefabs/monkeyisland_portal_debris.lua#L14-L19) |
| 无人机残骸 `wagdrone_*` | `1.0`＋`1.0`＋`0.5` → 期望 2.5 个；残骸见 [[reference/prefabs/wagdrone-parts]] | [掉落表](../../../raw/current/scripts/prefabs/wagdrone_common.lua#L125-L134) |

- 垃圾栅栏（`fence_junk`）另用一张表：`wagpunk_bits 0.25` ＋ `twigs 1.0`，拆这道栅栏有 25% 概率多给 1 个废料；检查眼镜箱（`inspectaclesbox`／`box2`）另给 0.75＋0.5／1.0＋0.75。以上是**无幸运基础值**：每条独立判定，1.00 直接入列、<1 先走 `GetChance`（按击杀者／最后工作者修正），期望按基础值相加。[围栏表](../../../raw/current/scripts/prefabs/fence.lua#L11-L15)、[栅栏名](../../../raw/current/scripts/languages/chinese_s.po#L342725-L342727)、[箱表](../../../raw/current/scripts/prefabs/inspectaclesbox.lua#L12-L28)、[生成](../../../raw/current/scripts/components/inspectaclesparticipant.lua#L288)、[GetChance](../../../raw/current/scripts/components/lootdropper.lua#L173-L227)

## 合成与其他消费端

| 用途组 | 配方（单次消耗的废料数） | 证据 |
|---|---|---|
| W.A.R.B.I.S. 装备与修理机 | `wagpunkhat` 8、`armorwagpunk` 8、`wagpunkbits_kit` 1——三条都是 `TECH.LOST`，靠蓝图学 | [L861-L863](../../../raw/current/scripts/recipes.lua#L861-L863) |
| 废料建筑 | `wall_scrap_item` 4（一次给 4 面）、`fence_electric_item` 6（给 4 个）、`wagpunk_floor_kit` 1（给 4 块）、`gestalt_cage` 2 | [L602](../../../raw/current/scripts/recipes.lua#L602)、[L604](../../../raw/current/scripts/recipes.lua#L604)、[L1015-L1016](../../../raw/current/scripts/recipes.lua#L1015-L1016) |
| 战备与新月装置 | `wagboss_robot_constructionsite_kit` 1、`wagboss_robot_creation_parts` 2、`moonstorm_static_catcher` 1、`moon_device_construction1` 4 | [L1018-L1020](../../../raw/current/scripts/recipes.lua#L1018-L1020)、[L593](../../../raw/current/scripts/recipes.lua#L593) |
| 角色技能配方 | Walter 电击废料弹 1／废料弹弓框 2、薇诺娜薇机人 8、WX-78 电刑机控制器 8（`wx78_zapdrone_1`）／灵体传输模块 2（`wx78_allegiance_lunar`） | [L223](../../../raw/current/scripts/recipes.lua#L223)、[L238](../../../raw/current/scripts/recipes.lua#L238)、[L319](../../../raw/current/scripts/recipes.lua#L319)、[L788-L789](../../../raw/current/scripts/recipes.lua#L788-L789) |
| 其它设施 | `chestupgrade_stacksize`（弹性空间制造器）4、`w_radio`（B.U.D.D.Y.）5 | [L562](../../../raw/current/scripts/recipes.lua#L562)、[L1049](../../../raw/current/scripts/recipes.lua#L1049) |
| 拆解回收 | `scrap_monoclehat` 返 2、`scraphat` 返 3、`slingshot_frame_wagpunk` 返 2 | [L1682-L1683](../../../raw/current/scripts/recipes.lua#L1682-L1683)、[L1713](../../../raw/current/scripts/recipes.lua#L1713) |

- 另有**修理**用途：`repairer.repairmaterial = MATERIALS.GEARS`、`workrepairvalue = 1`（工作面分支）、`healthrepairvalue = 80`（生命请求值，结算按上限截断），能修 `repairable.repairmaterial` 同为 GEARS 的目标——当前快照是废料墙与 `chessjunk`。[修理值](../../../raw/current/scripts/tuning.lua#L2107-L2134)、[可修目标](../../../raw/current/scripts/prefabs/walls.lua#L324-L326)、[棋子接线](../../../raw/current/scripts/prefabs/chessjunk.lua#L211-L213)
- 修理结算在 `Repairable:Repair`：材质必须相同，满血直接返回失败；80 由 `Health:SetVal` 截到上限，成功后取 1 个并移除。[结算](../../../raw/current/scripts/components/repairable.lua#L148-L183)、[候选](../../../raw/current/scripts/componentactions.lua#L1688-L1696)
- `wagpunkbits_kit`（自动修理机）被登记为 `FORGEMATERIALS.WAGPUNKBITS` 的修理件，修的是 W.A.R.B.I.S. 盔甲与帽子；用的是**修理机**而不是原始废料。[材料键](../../../raw/current/scripts/constants.lua#L1965-L1967)、[登记](../../../raw/current/scripts/prefabs/forge_repair_kits.lua#L58)、[目标](../../../raw/current/scripts/prefabs/armor_wagpunk.lua#L397)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`stackable` | `maxsize = TUNING.STACK_SIZE_SMALLITEM` = 40 | 仅主模拟 | [[reference/components/stackable]]；[定义](../../../raw/current/scripts/tuning.lua#L83)、[接线](../../../raw/current/scripts/prefabs/wagpunk_bits.lua#L36-L37) |
| 组件：`repairer` | `repairmaterial = MATERIALS.GEARS`、`workrepairvalue = 1`、`healthrepairvalue = 80`；加 `work_gears`／`health_gears` 标签 | 仅主模拟；右击可修目标 | [[reference/components/repairer]]、[[reference/components/repairable]]；[接线](../../../raw/current/scripts/prefabs/wagpunk_bits.lua#L45-L48)、[标签](../../../raw/current/scripts/components/repairer.lua#L9-L21) |
| 组件：`bait` | 无参数；配合 `molebait` 让鼹鼠来偷，也能进陷阱当饵 | 仅主模拟 | [[reference/components/bait]]；[接线](../../../raw/current/scripts/prefabs/wagpunk_bits.lua#L32) |
| 标签：`molebait` | 鼹鼠取饵检索的 must-tag；陷阱 `BAIT_TAGS` 也收它 | 公共端 | [鼹鼠检索](../../../raw/current/scripts/brains/molebrain.lua#L64-L82)、[陷阱](../../../raw/current/scripts/components/trap.lua#L264-L267) |
| 组件：`inventoryitem`／`floater` | 可携带；浮水 `"med"`、缩放 0.7 | `floater` 在公共端挂 | [[reference/components/inventoryitem]]、[[reference/components/floater]]；[接线](../../../raw/current/scripts/prefabs/wagpunk_bits.lua#L24-L24) |
| 组件：`hauntable` | `MakeHauntableLaunch`，只弹飞不点燃 | 仅主模拟 | [[reference/components/hauntable]]；[接线](../../../raw/current/scripts/prefabs/wagpunk_bits.lua#L50) |
| API／数据：`FORGEMATERIALS.WAGPUNKBITS` | 值就是 `"wagpunk_bits"`，与 `MakeForgeRepairable`／`forgerepair` 配对 | Forge 式修理件（自动修理机） | [常量](../../../raw/current/scripts/constants.lua#L1965-L1967) |
| 关联 Prefab：`wagpunkbits_kit`／`wall_scrap_item`／`fence_electric_item` | 直接产物；废料墙与麻刺节点还会反向掉废料 | 见获取与配方表 | [产物配方](../../../raw/current/scripts/recipes.lua#L861-L863) |
| 资源：`anim/wagpunk_bits.zip` | bank＝build＝`wagpunk_bits`，动画 `idle` | 公共端 | [assets](../../../raw/current/scripts/prefabs/wagpunk_bits.lua#L1-L4) |

## 典型接入

### 让自定义结构能被废料修补

- 前提与选择：这是**生命型**示例——目标要有 `health`、服务端补 `repairable`；材料端不用改，废料的 `health_gears` 标签即候选另一半。纯 `workable` 目标走另一分支（+1 工作面），见下条。
- 生效与撤销：`repairmaterial` 设为 `MATERIALS.GEARS` 后自动加 `repairable_gears`；`healthrepairable` 由构造与生命变化维护、`workrepairable` 由 `workable` 状态维护，都不用设。撤掉即移除组件，字段回 nil 时标签同步移除。
- 原版依据：`componentactions` 只在「材料有 `health_gears`」且「目标有 `healthrepairable`」时给 `REPAIR` 候选；`Repairable:Repair` 校验同材质后发 `DoDelta(80)`（`SetVal` 截到上限）并消耗 1 个。[候选](../../../raw/current/scripts/componentactions.lua#L1688-L1696)、[结算](../../../raw/current/scripts/components/repairable.lua#L148-L183)、[标签维护](../../../raw/current/scripts/components/repairable.lua#L1-L16)

```lua
-- modmain.lua：让自定义结构吃「废料修理」（GEARS 材质）
AddPrefabPostInit("my_structure", function(inst)
    if GLOBAL.TheWorld.ismastersim
        and inst.components.repairable == nil
        and inst.components.health ~= nil then
        inst:AddComponent("repairable")
        inst.components.repairable.repairmaterial = GLOBAL.MATERIALS.GEARS
    end
end)
```

- 调整与验证：示例只覆盖生命分支：每份废料请求 +80（按上限截断，`REPAIR_SCRAP_HEALTH`）；工作面目标是另一分支 +1（`REPAIR_GEARS_WORK`），两分支不同时结算，均由材料端决定；`my_structure` 是占位注册名，未运行游戏。

## 常见陷阱与限制

- **误区：废料能吃。** 文件里那段 `edible`（`FOODTYPE.GEARS`、Huge 档回复）整段是注释，实际没有 `edible` 组件；不提供任何饥饿／治疗。[注释段](../../../raw/current/scripts/prefabs/wagpunk_bits.lua#L39-L43)
- **边界：修理不是万能的。** 只认同材质（GEARS）目标，且目标要带 `healthrepairable`／`workrepairable`；满血目标直接失败、不消耗材料。当前快照只有废料墙与 `chessjunk` 接这条线。[结算](../../../raw/current/scripts/components/repairable.lua#L106-L115)、[可修目标](../../../raw/current/scripts/prefabs/walls.lua#L324-L326)
- **边界：放地上会被鼹鼠搬走。** `molebait`＋`bait` 让它进入鼹鼠的取饵检索（`STEALMOLEBAIT`），战后把废料丢在营地是常见损失来源。[检索](../../../raw/current/scripts/brains/molebrain.lua#L64-L82)
- **未核验**：未运行游戏。掉落概率与期望值都是 `SetSharedLootTable`／权重表的逐项推导（各条独立 roll），未做运行统计；鼹鼠实际偷取频率也未实测。

## 验证记录

- 源码审阅：2026-11-10，Steam buildid 26474873。通读 `prefabs/wagpunk_bits.lua` 全文；核验 `wagstaff_machinery.lua`／`daywalker2.lua`／`wagboss_robot.lua`／`junk_pile_common.lua`／`fence.lua`／`fence_electric.lua`／`monkeyisland_portal_debris.lua`／`walls.lua`／`chessjunk.lua`／`wagdrone_common.lua` 的掉落与修理接线；`components/repairer.lua`／`repairable.lua` 的材质与结算；`componentactions.lua` 的 REPAIR 候选；`brains/molebrain.lua` 的取饵；`recipes.lua` 的 22 处引用（18 条配方、1 条建造计划、3 条拆解）；`constants.lua` 的 `FORGEMATERIALS`；`tuning.lua` 的 `STACK_SIZE_SMALLITEM`／`REPAIR_GEARS_WORK`／`REPAIR_SCRAP_HEALTH`；简中 `.po` 名称键。
  未覆盖：W.A.R.B.I.S. 装备与各产物自身的机制、竞技场布置逻辑、Forge 活动是否启用、模组注入与调试命令。
- 游戏运行：未运行。掉落概率、期望值与 80 点修理量均为源码推导，非运行测量。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 见交付回报；修订记录：2026-11-10 内审 P2/P3（L48 补检查眼镜箱掉落、改「垃圾栅栏」，L39/L40 蓝图半径 4／1 格）＋外部复检 R07／R08／R11（L57 灵体传输模块费改 2 并补两条 builder_skill；L61／L62／L85／示例：80 为生命请求值、按上限截断，+1 属工作面分支、不手设 SetHealthRepairable；L48 概率改无幸运基础值、<1 走 GetChance；第二轮 L83-L84 示例限生命型）。

## 相关链接

- 同批材料：[[reference/prefabs/wagdrone-parts]]、[[reference/prefabs/scandata]]；组件：[[reference/components/repairer]]、[[reference/components/repairable]]、[[reference/components/bait]]

返回 [[reference/prefabs/index|Prefab 索引]]。