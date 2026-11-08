---
title: "陀螺传导核心（wagdrone_parts）"
created: "2026-11-10"
updated: "2026-11-10"
type: prefab
summary: "无人机残骸里拆出来的收编工具：对敌人螨地爬用一次就把它变成可部署的友方无人机（不设玩家 leader），对已收编但磨损的用一次补满；无效目标直接失败、有效但非 rolling 的返回 CANNOT_FIX_DRONE，且没有任何配方消费它。"
aliases: ["wagdrone_parts", "陀螺传导核心", "Gyroscopic Transduction Core", "无人机零件", "螨地爬零件", "修无人机"]
tags: [prefab, item]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/wagdrone_parts.lua", "prefabs/wagdrone_common.lua", "prefabs/wagdrone_rolling.lua", "prefabs/wagdrone_flying.lua", "prefabs/wagboss_robot.lua", "stategraphs/SGwagboss_robot.lua", "components/useabletargeteditem.lua", "components/stackable.lua", "components/workable.lua", "components/inventoryitem.lua", "componentactions.lua", "actions.lua", "tuning.lua", "standardcomponents.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# 陀螺传导核心（wagdrone_parts）

## 快速判断

- 职责与适用：单一注册 `wagdrone_parts`，物品态**工具**而非建材，简中名**陀螺传导核心**（英文 "Gyroscopic Transduction Core"）。它靠 `useabletargeteditem` 对无人机用：把敌方螨地爬收编成友方，或把磨损的友方补满。[注册](../../../raw/current/scripts/prefabs/wagdrone_parts.lua#L78)、[名称](../../../raw/current/scripts/languages/chinese_s.po#L351429-L351432)
- 优先使用：查使用链与两种结果 → [使用链：收编与修复](#使用链收编与修复)；查从哪来 → [获取：无人机残骸](#获取无人机残骸)；查堆叠／浮水 → [关联机制导航](#关联机制导航)。
- 重要副作用：**每成功一次消耗 1 个**（`stackable:Get():Remove()`），最后一个用完时物品本体被删；目标满耐久时仍返回成功但**不消耗**；目标失效／不 `persists` 返回 nil 直接失败，有效但非 `wagdrone_rolling` 才返回 `CANNOT_FIX_DRONE`。[使用函数](../../../raw/current/scripts/prefabs/wagdrone_parts.lua#L13-L32)
- 修改边界：收编后的行为（2000 次、受伤折算、部署点）归 `wagdrone_common`／螨地爬本体页（Wiki 待建）；本页不展开无人机 AI 与竞技场流程。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `wagdrone_parts` | `Prefab("wagdrone_parts", fn, assets)`：`require("prefabs/wagdrone_common")` 取收编函数；公共端建实体＋transform／anim／network、`MakeInventoryPhysics`、bank／build／`idle`、拾取音 `metal`、浮水 `("med", 0.5, {0.75, 1.1, 0.75})`，并把 `UseableTargetedItem_ValidTarget` 挂到实例 → `SetPristine` → 主模拟挂组件 | 只有物品态；主模拟挂 `inspectable` → `inventoryitem` → `stackable`（20）→ `useabletargeteditem`（`SetOnUseFn(OnUsed)`）→ `MakeHauntableLaunch`。[工厂](../../../raw/current/scripts/prefabs/wagdrone_parts.lua#L39-L76) |

- 目标判定放在**公共端**：`inst.UseableTargetedItem_ValidTarget = UseableTargetedItem_ValidTarget`，所以客户端点选目标时就能过滤，不必等服务器。[赋值](../../../raw/current/scripts/prefabs/wagdrone_parts.lua#L34-L37)、[客户端读取点](../../../raw/current/scripts/componentactions.lua#L1900-L1912)
- 依赖声明里带了 `scripts/prefabs/wagdrone_common.lua`（`Asset("SCRIPT", ...)`），因为 `OnUsed` 直接调 `WagdroneCommon.ChangeToFriendly`／`OnRepaired`。[资源](../../../raw/current/scripts/prefabs/wagdrone_parts.lua#L1-L7)

## 使用链：收编与修复

`USEITEMON` 候选要求目标带 `wagdrone` 标签，且带 `HAMMER_workable`（被打成残骸、可锤）或 `_inventoryitem`（已被 `inventoryitem` 接管、可携带的友方态）之一；`_inventoryitem` 是 AI／范围工具通用的「可捡物品」标记，锤子类 `workable` 则自己加 `HAMMER_workable`（`action.id.."_workable"`）。[目标判定](../../../raw/current/scripts/prefabs/wagdrone_parts.lua#L34-L37)、[workable 标签](../../../raw/current/scripts/components/workable.lua#L7-L13)、[`_inventoryitem` 的消费方示例](../../../raw/current/scripts/brains/krampusbrain.lua#L19)

| 目标状态 | 条件 | 结果 | 消耗 |
|---|---|---|---|
| 敌方螨地爬（未收编） | `target.prefab == "wagdrone_rolling"` 且 `target.components.finiteuses == nil` | `WagdroneCommon.ChangeToFriendly(target)`：移除 `workable`（不再可锤）、加 `companion`／`inventoryitem`／`finiteuses`（2000 次）／部署点组件；受伤改为扣使用次数；**不设玩家 leader** | 1 |
| 友方螨地爬（已收编） | 有 `finiteuses` 且 `GetPercent() < 1` | `SetPercent(1)` 补满，再 `OnRepaired`（不在手上也不在水面时记部署点并重新激活） | 1 |
| 友方螨地爬（满耐久） | `GetPercent() == 1` | 返回成功，但两个分支都不进 | 0 |
| 无效目标／其它无人机 | 目标失效或不 `persists` → 直接失败（返回 nil）；有效但不是 `wagdrone_rolling` → `false, "CANNOT_FIX_DRONE"` | 两种都不改状态、不扣料 | 0 |

- 收编是**就地改字段**，不生成新实体：同一架螨地爬被加组件、加标签、换回调，并移除 `workable`（不再是残骸）。[收编实现](../../../raw/current/scripts/prefabs/wagdrone_common.lua#L220-L256)
- 用完不锁手：成功路径把自己排进 `DoStaticTaskInTime(0, ResetInUse)` 调 `StopUsingItem`；最后一架用完时物品已被删，故先判 `inst:IsValid()`。[复位](../../../raw/current/scripts/prefabs/wagdrone_parts.lua#L9-L27)
- 消耗实现：`stackable:Get()` 从堆里拆出 1 个（`stacksize == 1` 时返回自身）再 `Remove()`。[Get](../../../raw/current/scripts/components/stackable.lua#L109-L153)
- 动作入口：候选插入 `ACTIONS.USEITEMON`；`USEITEMON.fn` 只调 `StartUsingItem`→`OnUseFn`，**不自动 `StopUsingItem`**（那是独立动作 `STOPUSINGITEM`）。[候选](../../../raw/current/scripts/componentactions.lua#L1900-L1913)、[执行](../../../raw/current/scripts/actions.lua#L3351-L3363)、[停止动作](../../../raw/current/scripts/actions.lua#L3365-L3375)

## 获取：无人机残骸

- 唯一来源是无人机残骸掉落表 `wagdrone_common`，被锤 1 下（`workable` 的 `HAMMER`、`SetWorkLeft(1)`）后掉：**陀螺传导核心 1.0（必掉）**、齿轮 1.0＋0.333、晶体管 0.667、废料 1.0＋1.0＋0.5。[掉落表](../../../raw/current/scripts/prefabs/wagdrone_common.lua#L125-L134)、[敲残骸](../../../raw/current/scripts/prefabs/wagdrone_common.lua#L136-L150)
- 残骸不是打死后自动生成，而是 `ChangeToLoot` 把无人机改成可锤对象；**战争瓦器人**（`wagboss_robot`）的死亡演出会在第 15 帧放掉已归队的无人机、第 159 帧再把没被接管的也放掉，两次都调 `ReleaseDrones` → `ChangeToLoot`。[死亡状态](../../../raw/current/scripts/stategraphs/SGwagboss_robot.lua#L1980-L2034)、[放无人机](../../../raw/current/scripts/prefabs/wagboss_robot.lua#L267-L295)
- 同一函数也被存档载入用到：残骸状态的无人机 `OnSave` 记 `isloot`，`OnLoad` 时补回 `workable`，所以读档后残骸仍能敲。[保存](../../../raw/current/scripts/prefabs/wagdrone_rolling.lua#L112-L131)
- 没有配方产出、也没有配方消费它：`recipes.lua` 里没有任何 `Ingredient("wagdrone_parts")` 或同名产物，原版只把它当工具。[检索范围](#验证记录)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`useabletargeteditem` | `SetOnUseFn(OnUsed)`；客户端用 `inst.UseableTargetedItem_ValidTarget` 过滤目标 | 主模拟挂组件，判定函数在公共端 | [[reference/components/useabletargeteditem]]；[接线](../../../raw/current/scripts/prefabs/wagdrone_parts.lua#L70-L71) |
| 组件：`stackable` | `maxsize = TUNING.STACK_SIZE_MEDITEM` = 20；只有收编／补满扣 1 个，满耐久成功不扣 | 仅主模拟 | [[reference/components/stackable]]；[定义](../../../raw/current/scripts/tuning.lua#L82)、[接线](../../../raw/current/scripts/prefabs/wagdrone_parts.lua#L67-L68) |
| 组件：`inventoryitem`／`floater` | 可携带；浮水 `"med"`、偏移 0.5、缩放 `{0.75, 1.1, 0.75}` | `floater` 在公共端挂 | [[reference/components/inventoryitem]]、[[reference/components/floater]]；[接线](../../../raw/current/scripts/prefabs/wagdrone_parts.lua#L54-L54) |
| 组件：`hauntable` | `MakeHauntableLaunch`，只弹飞不点燃 | 仅主模拟 | [[reference/components/hauntable]]；[接线](../../../raw/current/scripts/prefabs/wagdrone_parts.lua#L73) |
| 关联 Prefab：`wagdrone_rolling` | 收编／修复的唯一合法目标；带 `wagdrone` 标签，`MakeFriendly` 指向收编函数 | 竞技场无人机；`ChangeToLoot` 先挂可锤组件，玩家锤击才兑现掉落 | [标签](../../../raw/current/scripts/prefabs/wagdrone_rolling.lua#L165-L171)、[`MakeFriendly`](../../../raw/current/scripts/prefabs/wagdrone_rolling.lua#L220-L220)（Wiki 待建） |
| 关联 Prefab：`wagdrone_flying` | 也带 `wagdrone` 标签，但初始没有 `inventoryitem`／`workable`，只有被 `ChangeToLoot` 转成残骸后才满足目标判定；`OnUsed` 仍只认 `wagdrone_rolling` | 同一竞技场 | [标签](../../../raw/current/scripts/prefabs/wagdrone_flying.lua#L223-L227)、[转残骸](../../../raw/current/scripts/prefabs/wagdrone_common.lua#L152-L169)（Wiki 待建） |
| 关联 Prefab：`wagboss_robot` | 死亡演出把无人机转成残骸，是零件的主要来源 | 仅首领死亡状态 | [释放](../../../raw/current/scripts/prefabs/wagboss_robot.lua#L267-L295)（Wiki 待建） |
| API／数据：`TUNING.WAGDRONE_ROLLING_USES`＝2000 | 收编时写入 `finiteuses` 上限；受伤按 `math.floor(0.5 - amount/最大生命 × 2000)` 折成次数（`amount` 为带符号伤害，扣血时为负），再用 `math.max(1, uses)` 保底、每次至少 1 次 | 收编后每受一次伤 | [值](../../../raw/current/scripts/tuning.lua#L8670)、[折算](../../../raw/current/scripts/prefabs/wagdrone_common.lua#L181-L186) |
| 资源：`anim/wagdrone_parts.zip` | bank＝build＝`wagdrone_parts`，动画 `idle` | 公共端 | [assets](../../../raw/current/scripts/prefabs/wagdrone_parts.lua#L1-L4) |

## 典型接入

### 复用「一次性道具改目标状态再解锁」的模式

- 前提与选择：自定义道具已经挂了 `useabletargeteditem` 与 `stackable`；`ValidTarget` 要挂在**公共段**（`SetPristine` 之前，客户端也用它筛目标），而 `USEITEMON` 执行时**不重跑候选回调**，所以服务端 `OnUseFn` 必须自己再验目标有效、带标签、有 `health`，验过再改状态与扣料。
- 生效与撤销：`OnUseFn` 返回 `true` 只让组件置 `inuse_targeted`，**组件不自动扣料**——消耗由 `OnUseFn` 自己 `stackable:Get():Remove()` 结算；成功后要自己安排 `StopUsingItem`（原版用 0 秒任务），不想复位的道具可用 `SetUsingItemDoesNotToggleUseability(true)` 省掉这一步。
- 原版依据：候选在客户端读 `inst.UseableTargetedItem_ValidTarget`，执行时 `USEITEMON.fn` 只调 `StartUsingItem`→`OnUseFn`，成功才置 `inuse_targeted`；原版 `OnUsed` 自己判 `target.persists`／prefab 再动手。[候选](../../../raw/current/scripts/componentactions.lua#L1900-L1913)、[执行](../../../raw/current/scripts/actions.lua#L3351-L3363)、[组件](../../../raw/current/scripts/components/useabletargeteditem.lua#L105-L137)

```lua
-- modmain.lua：公共段筛目标，服务端复验后改状态、扣料
AddPrefabPostInit("my_tool", function(inst)
    inst.UseableTargetedItem_ValidTarget = function(inst, target)
        return target:HasTag("my_repairable")
    end
    if not GLOBAL.TheWorld.ismastersim then return end
    inst.components.useabletargeteditem:SetOnUseFn(function(inst, target)
        if not (target and target:IsValid() and target:HasTag("my_repairable") and target.components.health) then return false end
        target.components.health:DoDelta(50)
        inst.components.stackable:Get():Remove()
        if inst:IsValid() then inst:DoStaticTaskInTime(0, function()
            if inst:IsValid() then inst.components.useabletargeteditem:StopUsingItem() end
        end) end
        return true
    end)
end)
```

- 调整与验证：`my_tool`／`my_repairable` 都是占位名；目标无 `health` 时返回 `false` 不报错；末件被删时先 `inst:IsValid()` 再排复位。未运行游戏。

## 常见陷阱与限制

- **误区：它是通用无人机修理件。** `OnUsed` 里硬判 `target.prefab == "wagdrone_rolling"`，给 `wagdrone_flying` 用只会拿到 `CANNOT_FIX_DRONE`；标签开放但 prefab 判定没开放。[判定](../../../raw/current/scripts/prefabs/wagdrone_parts.lua#L13-L31)
- **误区：满耐久使用会白扣一个。** 满耐久时只回 `true`、不消耗；真正扣料的是「无 `finiteuses`」和「`GetPercent() < 1`」两条分支。[分支](../../../raw/current/scripts/prefabs/wagdrone_parts.lua#L16-L23)
- **边界：收编后不能再锤回残骸。** `ChangeToFriendly` 移除 `workable`，耐久归零只 `ForgetDeployPoint`＋`deactivate`、不会变回残骸，反悔没有原版路径；残骸态锤拆时核心在该表里是 `1.00` 必掉，「不一定回本」不成立。[收编](../../../raw/current/scripts/prefabs/wagdrone_common.lua#L220-L253)、[耗尽](../../../raw/current/scripts/prefabs/wagdrone_common.lua#L206-L209)
- **边界：最后一个用掉时物品消失。** `Get()` 在 `stacksize == 1` 时返回自身，随后 `Remove()`；源码用 `inst:IsValid()` 兜住，模组在 `OnUseFn` 之外继续访问该实例会拿到已删实体。[实现](../../../raw/current/scripts/components/stackable.lua#L109-L152)、[兜底](../../../raw/current/scripts/prefabs/wagdrone_parts.lua#L24-L27)
- **未核验**：未运行游戏。「收编后 2000 次」与「受伤折算」是 `finiteuses` 上限与 `FriendlyDamageToUses` 的源码推导，未实测实际耐久表现。

## 验证记录

- 源码审阅：2026-11-10，Steam buildid 26474873。通读 `prefabs/wagdrone_parts.lua` 全文；核验 `prefabs/wagdrone_common.lua` 的掉落表、`OnWorked`、`ChangeToLoot`、`ChangeToFriendly`、`OnRepaired`、`FriendlyDamageToUses`；`SGwagboss_robot.lua` 的 `death` 状态两次 `ReleaseDrones`；`wagdrone_rolling.lua` 的标签、`OnSave`／`OnLoad` 与 `MakeFriendly`；`wagdrone_flying.lua` 的 `wagdrone` 标签；`components/useabletargeteditem.lua` 全文、`componentactions.lua` 的候选、`actions.lua` 的 `USEITEMON`；`components/stackable.lua` 的 `Get` 与 `workable` 标签；`tuning.lua` 的 `STACK_SIZE_MEDITEM` 与 `WAGDRONE_ROLLING_USES`；简中 `.po` 名称键。
  检索范围：`recipes.lua` 全文件检索 `wagdrone_parts` 无命中（既非原料也非产物）；`prefabs/` 内仅 `wagdrone_` 系列与 `scrapbook` 数据引用它。未覆盖：竞技场如何布置无人机、招募与 AI 行为、模组注入与调试命令。
- 游戏运行：未运行。掉率期望为掉落表逐项相加的推导（核心 1、废料 2.5），非运行统计。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 见交付回报；修订记录：2026-11-10 内审 P3（L67 flying 仅残骸态可选中、L69 保底 1 次、L77 只置位不定消耗）＋外部复检 R02／R06（L74-L99 示例 ValidTarget 移公共段并补服务端复验；L49 补 USEITEMON 不自动 StopUsingItem；摘要／L23／L41／L44／L46／L105 补移除 workable、不设 leader、耗尽不变残骸；第二轮 L63／L66 补齐扣料条件与残骸锤击兑现）。

## 相关链接

- 同批材料：[[reference/prefabs/wagpunk-bits]]、[[reference/prefabs/scandata]]；组件：[[reference/components/useabletargeteditem]]、[[reference/components/stackable]]、[[reference/components/inventoryitem]]

返回 [[reference/prefabs/index|Prefab 索引]]。