---
title: "repairable"
created: "2026-10-17"
updated: "2026-10-28"
type: component
summary: "被修目标的材料匹配、生命／工作量／保鲜／次数修复顺序，以及动作标签和成功消费边界。"
aliases: ["材料修复装备", "修复目标", "石头修斧头", "建筑修复", "repairable"]
tags: [component, item, building, equipment, callback, actions]
dst_build: "steam-buildid:26230496"
sources: ["components/repairable.lua", "components/repairer.lua", "components/health.lua", "components/finiteuses.lua", "components/workable.lua", "components/perishable.lua", "entityscript.lua", "actions.lua", "componentactions.lua", "constants.lua", "prefabs/axe.lua", "prefabs/inv_rocks.lua", "prefabs/walls.lua", "prefabs/amulet.lua", "stategraphs/SGwilson.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# repairable

## 快速判断

- 职责与适用：挂在**被修目标**，配合材料端 [[reference/components/repairer]] 处理 REPAIR；可修生命、工作量、腐败比例、finiteuses，按顺序选择一个分支。[repairable.lua:106](../../../raw/current/scripts/components/repairable.lua#L106
- 优先使用：设置 `repairmaterial` 匹配原版 MATERIALS 值；选目标实际组件和材料修复量，必要时加 `testvalidrepairfn`／`checkmaterialfn`。衣物时间耐久走 [[reference/components/sewing]]，护甲自身耐久见 [[reference/components/armor]]。[repairable.lua:148](../../../raw/current/scripts/components/repairable.lua#L148
- 重要副作用：分支认为可修后先改目标，随后消费一份材料，最后调用 onrepaired；成功不保证实际净恢复，已有高优先分支失败也不会转试后续。[repairable.lua:106](../../../raw/current/scripts/components/repairable.lua#L106)、[:168](../../../raw/current/scripts/components/repairable.lua#L168)
- 修改边界：服务端操作；动作候选依赖材料／目标标签与 MATERIALS 枚举。直接 Repair 不以这些标签、NeedsRepairs 或“可修”字段作为最终门禁；自定义材料字符串相同不代表 UI 能选到。[componentactions.lua:1679](../../../raw/current/scripts/componentactions.lua#L1679)、[constants.lua:1940](../../../raw/current/scripts/constants.lua#L1940

## 关键接口

| 接口／字段 | 约定 | 条件、副作用与证据 |
|---|---|---|
| `repairmaterial` | 与材料 repairer 同值 | setter 维护 repairable_<材料> 标签；Repair 只比较两者是否相等，不验证属于 MATERIALS。不要把两端 nil 相等当正式配置。[repairable.lua:1](../../../raw/current/scripts/components/repairable.lua#L1)、[:153](../../../raw/current/scripts/components/repairable.lua#L153) |
| `SetHealthRepairable(v)`／`SetWorkRepairable(v)`／`SetFiniteUsesRepairable(v)` | 布尔值／对应字段 | 维护 healthrepairable／workrepairable／finiteusesrepairable 标签供候选使用；不是禁用 Repair 的开关。[repairable.lua:10](../../../raw/current/scripts/components/repairable.lua#L10)、[:63](../../../raw/current/scripts/components/repairable.lua#L63) |
| `testvalidrepairfn` | `(target, material) → truthy` | 最先检查；失败仅返回 false，不保留第二个原因值。[repairable.lua:148](../../../raw/current/scripts/components/repairable.lua#L148 |
| `checkmaterialfn` | `(target, material) → success, reason?` | 材料匹配后检查；失败返回 false, reason，不消费。[repairable.lua:157](../../../raw/current/scripts/components/repairable.lua#L157 |
| `Repair(doer, repair_item)` | true／false，材料回调可带 reason | 接收有效实体；缺 repairer／材料不匹配／测试或恢复分支失败时不进入消费；不调用材料自带的 Repair 方法。[repairable.lua:148](../../../raw/current/scripts/components/repairable.lua#L148 |
| `onrepaired` | `(target, doer, repair_item)`；返回忽略 | 在材料消费**之后**调用，传来的原材料可能已被删除，不假设仍有效；不靠返回 false 撤销修复。[repairable.lua:172](../../../raw/current/scripts/components/repairable.lua#L172 |
| `NeedsRepairs()` | bool | 按 health→workable→perishable→finiteuses 检查首个适用组件的低于 95% 状态，与实际修复分支／100% 门槛不同；不是 Repair 前置。[repairable.lua:90](../../../raw/current/scripts/components/repairable.lua#L90 |

### 恢复分支与顺序

表中数值都来自材料的 repairer；先命中者独占本次结算。比如同时有 health／finiteuses，材料又同时提供两类正修复量，即使生命已满也直接失败，不会修次数。[repairable.lua:106](../../../raw/current/scripts/components/repairable.lua#L106

| 优先级 | 命中与拒绝条件 | 恢复方式／边界 |
|---|---|---|
| 1：生命 | 有 health，固定量或比例任一 >0；GetPercent()>=1 拒绝 | **先后两次** DoDelta：固定 HP、比例×基础 maxhealth；不乘 efficientuser，效果仍经 health 拦截／封顶。[repairable.lua:107](../../../raw/current/scripts/components/repairable.lua#L107 |
| 2：工作量 | 有有效 workleft 且材料量 >0；不可工作或已满拒绝 | SetWorkLeft(现有+量)，按 workable 上限夹取。[repairable.lua:117](../../../raw/current/scripts/components/repairable.lua#L117)、[workable.lua:83](../../../raw/current/scripts/components/workable.lua#L83 |
| 3：保鲜 | 有剩余腐败时间且材料比例 >0；时间已满拒绝 | SetPercent(原比例+增量)，上限 1；不是无限保鲜。[repairable.lua:126](../../../raw/current/scripts/components/repairable.lua#L126)、[perishable.lua:223](../../../raw/current/scripts/components/perishable.lua#L223 |
| 4：次数 | 有 finiteuses 且材料量 >0；比例>=1 拒绝 | Repair(次数)，不超过 total。[repairable.lua:135](../../../raw/current/scripts/components/repairable.lua#L135)、[finiteuses.lua:128](../../../raw/current/scripts/components/finiteuses.lua#L128 |
| 无数值分支 | 以上全部未命中 | 仅 justrunonrepaired 为真才继续消费／回调；不会绕过已命中分支的失败。[repairable.lua:144](../../../raw/current/scripts/components/repairable.lua#L144 |

### 初始化、标签与移除

构造时只根据已有 health／workable 初始化对应标签，**不初始化 finiteuses 缺损**；新增 repairable 到已损耗次数物品时应立即按 GetPercent 同步。此后 health、workable、finiteuses 的属性回调更新其标记；修复不提供独立存档，参数由工厂重建，实际数值归各组件保存。[repairable.lua:35](../../../raw/current/scripts/components/repairable.lua#L35)、[health.lua:9](../../../raw/current/scripts/components/health.lua#L9)、[workable.lua:1](../../../raw/current/scripts/components/workable.lua#L1)、[finiteuses.lua:1](../../../raw/current/scripts/components/finiteuses.lua#L1

移除时清理材料和三类目标标签；不撤销已恢复的数值。宿主负责调用 OnRemoveFromEntity，勿手调清理后留下仍有效组件。[repairable.lua:75](../../../raw/current/scripts/components/repairable.lua#L75)、[entityscript.lua:670](../../../raw/current/scripts/entityscript.lua#L670

## 典型接入

### 普通斧头用石头修复二十次耐久

放在已启用模组 `modmain.lua`，普通生存模式，无需 GLOBAL 代理；复用原版 axe／rocks 的资源、物品、动作，不创建新 Prefab。axe 已有 finiteuses，rocks 已有 STONE 修复材料配置；本例新增斧头目标能力，并给石头补次数修复量。[axe.lua:110](../../../raw/current/scripts/prefabs/axe.lua#L110)、[inv_rocks.lua:64](../../../raw/current/scripts/prefabs/inv_rocks.lua#L64)、[注册](../../../raw/current/scripts/prefabs/axe.lua#L291)

```lua
AddPrefabPostInit("axe", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local uses = inst.components.finiteuses
    if uses == nil or inst.components.repairable ~= nil then return end
    inst:AddComponent("repairable")
    local target = inst.components.repairable
    target.repairmaterial = GLOBAL.MATERIALS.STONE
    target:SetFiniteUsesRepairable(uses:GetPercent() < 1)
end)

AddPrefabPostInit("rocks", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local material = inst.components.repairer
    if material ~= nil and material.repairmaterial == GLOBAL.MATERIALS.STONE then
        material.finiteusesrepairvalue = 20
    end
end)
```

20 为次数，不是 20%：总次数 100 的斧头在剩余 10 时修到 30。已有 repairable 的斧头跳过，不覆盖其他模组的材料；石头的新数值会覆盖同字段，并影响**所有匹配 STONE 的次数修复目标**，不是斧头专属过滤。其它数值保持原值；专属限制需按需求另配目标测试／材料，勿默加全局注册。两端 MATERIALS 标签让既有 REPAIR 候选可选，骑乘等限制仍保留。[componentactions.lua:1679](../../../raw/current/scripts/componentactions.lua#L1679

新建／载入时重新配置，实际次数由 finiteuses 恢复并更新标签；停用后斧头不再新增此修复方式，已恢复次数不撤回。耐久耗尽的原版斧头已删除，不能用本例找回；耗尽策略见 [[reference/components/finiteuses]]。保留原装备、攻击及砍树消费。[axe.lua:115](../../../raw/current/scripts/prefabs/axe.lua#L115

### 原版模式参照

墙体把 health 与 repairable 相接，并在破损墙占位等条件下用 testvalidrepairfn 校验；橙色护符用 NIGHTMARE 匹配噩梦燃料恢复 finiteuses。参照的是不同恢复对象，不必复制整个工厂。[walls.lua:109](../../../raw/current/scripts/prefabs/walls.lua#L109)、[:324](../../../raw/current/scripts/prefabs/walls.lua#L324)、[amulet.lua:555](../../../raw/current/scripts/prefabs/amulet.lua#L555

## 常见陷阱与限制

- **NeedsRepairs=false 就不能修**：它用 95% 提示阈值，Repair 各分支通常到 100% 才拒绝；95%～100% 仍可能可修。请分开显示提示和最终业务校验。[repairable.lua:90](../../../raw/current/scripts/components/repairable.lua#L90
- **healthrepairable=false 是强门禁**：标签只影响候选，直接 Repair 依据真实组件和材料数值；需要硬限制放测试回调。新材料字符串还须核验双端枚举与动作链。[repairable.lua:148](../../../raw/current/scripts/components/repairable.lua#L148)、[componentactions.lua:1688](../../../raw/current/scripts/componentactions.lua#L1688
- **修复成功一定有净恢复**：生命按基础 maxhealth 判断是否满，可用上限惩罚／无敌／redirect 等仍可能让两次 DoDelta 无有效提升但消费材料；需要更严条件在测试中实现。[repairable.lua:108](../../../raw/current/scripts/components/repairable.lua#L108)、[health.lua:613](../../../raw/current/scripts/components/health.lua#L613
- **所有“耐久”都能接入**：该实现没有 armor／fueled 分支；护甲恢复或缝补用各自实际入口，不为了同名耐久强添 health。[repairable.lua:106](../../../raw/current/scripts/components/repairable.lua#L106

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 的变更文件（componentactions.lua、constants.lua、entityscript.lua、components/perishable.lua、prefabs/amulet.lua）对照更新报告 R-01/R-04、R-06/R-08、C-26、P-03 逐条核验，REPAIR 候选、MATERIALS 枚举与四类恢复分支无语义变化（C-26 仅改 perishable AddTime 前置，与本页引用的 SetPercent 无关），锚点重定位 2 处、结论修订 0 处；未覆盖：运行验证。

- 源码审阅：2026-10-17，Steam buildid 25643519；覆盖完整 repairable、材料标签与 REPAIR 分派、四类恢复消费者、宿主清理、斧头／石头案例及墙／护符原版参照；未审校全部建筑、海上系统、锻造修复或其他模组消费者。
- 结构检查：发布时核对元数据、来源、大小、链接、源码锚点及唯一登记；完整双 PostInit 示例以已有 Full Moon／Lua 5.1 作 syntax-only 检查，不包含游戏行为或跨模组兼容验证。
- 游戏运行：未运行；客户端候选、已有损耗／读档、材料消费、满值／惩罚、同时有多个恢复组件及其他模组覆盖待实测。

相关：[[reference/components/repairer]]；[[reference/components/finiteuses]]；[[reference/components/workable]]；[[reference/components/perishable]]；[[reference/components/trader]]。
