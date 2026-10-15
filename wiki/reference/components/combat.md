---
title: "combat"
created: "2026-10-12"
updated: "2026-10-28"
type: component
summary: "配置攻击伤害、目标与受击入口，连接武器、护甲、特殊伤害和生命变化。"
aliases: ["角色攻击倍率", "攻击增伤", "受击事件", "受击减伤", "造成普通伤害", "战斗目标", "combat"]
tags: [component, callback, lifecycle, mechanism, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/combat.lua", "components/health.lua", "components/inventory.lua", "components/weapon.lua", "prefabs/player_common.lua", "prefabs/wolfgang.lua", "util/sourcemodifierlist.lua", "stategraphs/SGpig.lua", "modutil.lua", "mainfunctions.lua", "entityscript.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# combat

## 快速判断

- 职责与适用：协调目标、距离／冷却、伤害与受击；生命数值见 [[reference/components/health]]，物品基础伤害见 [[reference/components/weapon]]。接口与证据见[关键接口](#关键接口)。
- 优先使用：无武器基础值 SetDefaultDamage；人物普通增伤 externaldamagemultipliers；普通受击用目标 GetAttacked。见[关键接口](#关键接口)。
- 重要副作用：受击可磨损护甲、重定向／格挡、致死并发事件；外部调用 GetAttacked 已不返回结果（转调 GetAttacked_Internal），依赖返回值改调 _Internal。DoAttack 另有弹体／武器扣次。见[伤害与事件顺序](#伤害与事件顺序)。
- 修改边界：服务端；TryAttack 请求 doattack，状态图 StartAttack／命中帧 DoAttack；加组件不提供完整 AI／动画。见[动作边界](#常见陷阱与限制)。

## 关键接口

| 接口／字段 | 签名或约定 | 条件、作用与证据 |
|---|---|---|
| `SetDefaultDamage(damage)` | 普通伤害数值 | 无武器基础值；持 weapon 时改走其 GetDamage，骑乘还可能取坐骑。[combat.lua:222](../../../raw/current/scripts/components/combat.lua#L222)、[:891](../../../raw/current/scripts/components/combat.lua#L891) |
| `externaldamagemultipliers`／`externaldamagetakenmultipliers` | 乘法来源列表 | 前者影响普通攻击计算；后者在护甲之后处理普通受击量，不统一乘后续特殊伤害。[combat.lua:65](../../../raw/current/scripts/components/combat.lua#L65)、[:651](../../../raw/current/scripts/components/combat.lua#L651)、[:948](../../../raw/current/scripts/components/combat.lua#L948) |
| `CalcDamage(target, weapon?, multiplier?)` | 返回普通数值、特殊伤害表或 nil；alwaysblock 只返回 0 | 只计算攻击侧，尚未进目标护甲／生命；不能当最终掉血。[combat.lua:871](../../../raw/current/scripts/components/combat.lua#L871 |
| `GetAttacked(attacker, damage, weapon?, stimuli?, spdamage?)` | 转调 `GetAttacked_Internal`，**不返回结果** | 目标侧受击入口；需要返回值（not blocked，死亡目标提前 true）改调 `GetAttacked_Internal(attacker, damage, weapon, stimuli, spdamage, from_doattack)`。本页普通场景要求有效攻击者及数值 damage，不等同目标选择、攻击动作或攻击者倍率计算。[combat.lua:570](../../../raw/current/scripts/components/combat.lua#L570)、[:574](../../../raw/current/scripts/components/combat.lua#L574) |
| `DoAttack(targ?, weapon?, projectile?, stimuli?, instancemult?, instrangeoverride?, instpos?)` | 无成功返回约定 | 自动选目标／手部武器并检查命中，再分近战／弹体；直接调用不执行 CanAttack 的完整冷却／busy 检查。[combat.lua:1120](../../../raw/current/scripts/components/combat.lua#L1120 |
| `SetRange(attack, hit?)`／`SetAttackPeriod(period)` | 距离／秒；hit 默认 attack | 改基础范围／最低间隔；武器范围和碰撞半径另参与计算，攻击动画帧率不因此改变。[combat.lua:120](../../../raw/current/scripts/components/combat.lua#L120)、[:148](../../../raw/current/scripts/components/combat.lua#L148)、[:1014](../../../raw/current/scripts/components/combat.lua#L1014) |
| `SetTarget(target)`／`SuggestTarget(target)`／`DropTarget()` | target 实体或 nil | SetTarget 有有效性／仇恨／隐藏门禁并管理监听；SuggestTarget 仅当前无目标时建议，不保证最终赋入。[combat.lua:473](../../../raw/current/scripts/components/combat.lua#L473)、[:234](../../../raw/current/scripts/components/combat.lua#L234)、[:368](../../../raw/current/scripts/components/combat.lua#L368) |
| `CanAttack(target)`／`TryAttack(target?)` | 前者成功真／失败假或 nil，部分失败含第二参；后者 boolean | 先判目标／冷却／busy／距离；TryAttack 依赖状态图，已经 attack 时也可返回 true，不是掉血确认。[combat.lua:780](../../../raw/current/scripts/components/combat.lua#L780)、[:829](../../../raw/current/scripts/components/combat.lua#L829) |
| `SetRetargetFunction(period, fn)`／`SetKeepTargetFunction(fn)` | 重选 `(inst)->target, forcechange?`；保持 `(inst, target)` | 管理周期重选／持续保持；需结合已有 AI 与目标规则，不是完整生物骨架。[combat.lua:242](../../../raw/current/scripts/components/combat.lua#L242 |
| `SetOnHit(fn)` | `(inst, attacker, damage, spdamage)` | 目标受击侧单回调；onhitotherfn 为攻击者侧另字段，勿混用。[combat.lua:226](../../../raw/current/scripts/components/combat.lua#L226)、[:718](../../../raw/current/scripts/components/combat.lua#L718) |

## 典型接入

### 给人物增加 20% 普通攻击伤害

目标模组 `modmain.lua`，普通生存模式；已有 combat，无新增资源。使用独立来源 key 保留角色原 damagemultiplier 和其他来源；原版大力士教练 buff 成对设置／撤销该列表。[modutil.lua:586](../../../raw/current/scripts/modutil.lua#L586)、[wolfgang.lua:702](../../../raw/current/scripts/prefabs/wolfgang.lua#L702

```lua
AddPlayerPostInit(function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    if inst.components.combat ~= nil then
        inst.components.combat.externaldamagemultipliers:SetModifier(
            inst, 1.2, "dstwiki_damage_passive")
    end
end)
```

此普通来源倍率乘 1.2；伤害加数、骑乘来源和特殊伤害另走分支，不保证总掉血提高 20%。常驻至该实例删除，临时效果失效用同列表 RemoveModifier(inst, "dstwiki_damage_passive")；来源删除也会自动清理，运行期配置不由 combat 存档。[combat.lua:904](../../../raw/current/scripts/components/combat.lua#L904)、[:940](../../../raw/current/scripts/components/combat.lua#L940)、[sourcemodifierlist.lua:64](../../../raw/current/scripts/util/sourcemodifierlist.lua#L64

### 已有技能给指定目标造成普通伤害

调用处已在目标模组的服务端技能／动作回调，attacker／target 是有效实体；已有技能负责阵营、距离、频率与目标资格。目标有 combat／health 时 `target.components.combat:GetAttacked(attacker, 30)` 送入普通受击链；30 是传入受击量，不会再自动调用攻击者 CalcDamage。GetAttacked 现不返回结果，需要受击结果改调 GetAttacked_Internal。[combat.lua:570](../../../raw/current/scripts/components/combat.lua#L570

若技能需人物／武器攻击倍率，先用攻击者 CalcDamage 计算，保留两个返回值再交目标 GetAttacked；需要完整武器动作／弹体／反伤时复用原攻击路径，不能仅这一步代替 DoAttack。组件入口不会替技能验证 PVP 等目标政策；本页不提供尚未核验的自建技能骨架。[combat.lua:1213](../../../raw/current/scripts/components/combat.lua#L1213

## 伤害与事件顺序

1. 近战 DoAttack 取得 CalcDamage 普通／特殊伤害；计算反射后，目标 GetAttacked（实际执行体 GetAttacked_Internal）先过重定向／反冲／闪避等分支，再由 inventory:ApplyDamage 处理装备。[combat.lua:1213](../../../raw/current/scripts/components/combat.lua#L1213)、[:570](../../../raw/current/scripts/components/combat.lua#L570)、[inventory.lua:408](../../../raw/current/scripts/components/inventory.lua#L408
2. 普通受击量经类型／外部倍率等，再处理特殊防御／特殊伤害，交 health:DoDelta。health 自身吸收／封顶与死亡仍可影响实际结果。[combat.lua:648](../../../raw/current/scripts/components/combat.lua#L648)、[health.lua:613](../../../raw/current/scripts/components/health.lua#L613
3. 未 blocked 时目标发 attacked → 目标 onhitfn → 攻击者 onhitother／onhitotherfn；blocked 时目标发 blocked。DoAttack 后续 weapon:OnAttack 另触发武器回调／耐久，不等于已经造成正掉血。[combat.lua:715](../../../raw/current/scripts/components/combat.lua#L715)、[:1224](../../../raw/current/scripts/components/combat.lua#L1224)；物品链见 [[reference/components/weapon]]。

（26148036 核验：GetAttacked 拆为包装＋`GetAttacked_Internal`，伤害重定向、DoAttack／DoAreaAttack、反伤结算五处内部路径直调 _Internal 并带 `from_doattack=true`，绕过对 `GetAttacked` 的包装／覆盖；`attacked`／`onhitother`／`blocked`／`killed` 事件新增 `from_doattack` 数据（`blocked` 另附 weapon／stimuli）；攻击者被 `IsItemEquipped` 穿在受害者身上时无视闪避、护甲与鞍防御——[combat.lua:570](../../../raw/current/scripts/components/combat.lua#L570)、[:1218](../../../raw/current/scripts/components/combat.lua#L1218)）

事件 `damage` 是受击处理量，`damageresolved` 来自 health 返回处理量；后者也不能保证等于生命前后差。人物监听自己的 attacked，攻击者监听自己的 onhitother；healthdelta 还覆盖饥饿等非战斗变化。[health.lua:640](../../../raw/current/scripts/components/health.lua#L640

## 生命周期

- SetRetargetFunction 重建周期任务；实体休眠取消、唤醒恢复，宿主显式调用组件睡眠／唤醒钩子。[combat.lua:275](../../../raw/current/scripts/components/combat.lua#L275)、[mainfunctions.lua:696](../../../raw/current/scripts/mainfunctions.lua#L696)、[:752](../../../raw/current/scripts/mainfunctions.lua#L752)
- OnRemoveFromEntity 清目标监听、blank／retarget 任务与 knockback 监听；宿主 RemoveComponent 执行该钩子。combat 无 OnSave／OnLoad，目标、倍率／临时效果须按实际状态重建。[combat.lua:1359](../../../raw/current/scripts/components/combat.lua#L1359)、[entityscript.lua:670](../../../raw/current/scripts/entityscript.lua#L670)、[:1928](../../../raw/current/scripts/entityscript.lua#L1928)

## 常见陷阱与限制

- **改 SetDefaultDamage 就增加所有武器伤害**：weapon 分支不用该基础值；物品改 weapon，人物普通倍增用独立 externaldamagemultipliers。[combat.lua:891](../../../raw/current/scripts/components/combat.lua#L891
- **直接 health:DoDelta(-30) 测护甲效果**：跳过 inventory 护甲／受击事件链；普通受击用 GetAttacked，穿甲／环境效果才明确选择直接生命入口。[combat.lua:624](../../../raw/current/scripts/components/combat.lua#L624
- **DoAttack 自动保证攻击间隔与动画**：CanAttack／StartAttack／状态图负责不同步骤；不要每 tick 强制 DoAttack 当 AI。[combat.lua:763](../../../raw/current/scripts/components/combat.lua#L763)、[:780](../../../raw/current/scripts/components/combat.lua#L780)、[SGpig.lua:175](../../../raw/current/scripts/stategraphs/SGpig.lua#L175
- **GetAttacked 返回 true 就击杀／掉血**：外部 GetAttacked 现转调 _Internal 且不返回结果，"死目标提前 true、未 blocked 返回 not blocked"的返回值在 GetAttacked_Internal；未 blocked 仍可能归零／被 health 限制。确认击杀查生命或 killed 事件，实际掉血查前后值。[combat.lua:570](../../../raw/current/scripts/components/combat.lua#L570)、[:575](../../../raw/current/scripts/components/combat.lua#L575)、[:737](../../../raw/current/scripts/components/combat.lua#L737)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 的全部变更文件对照 build-26148036 更新报告逐条核验，锚点重定位 24 处、结论修订 4 处（GetAttacked 拆分与事件数据，见正文"26148036 核验"标注）；未覆盖：运行验证。

- 源码审阅：2026-10-12，Steam buildid 25643519；覆盖普通伤害／受击核心链、来源、事件、目标／重选与状态图分工及宿主清理；未审校全部远程／AOE／反射／PVP／AI／动态类型分支。
- 结构检查：页面／索引、链接与源码行锚点检查通过；Lua 经 Full Moon 3.0.0／Lua 5.1 语法及原字节往返检查。
- 游戏运行：未运行；增伤／骑乘／受击、护甲磨损、格挡／死亡事件、目标和多人兼容待实测。

相关：[[reference/components/health]]；[[reference/components/armor]]；[[reference/components/planardamage]]；[[reference/components/planardefense]]。
