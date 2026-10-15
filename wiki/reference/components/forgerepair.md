---
title: "forgerepair"
created: "2026-10-26"
updated: "2026-10-28"
type: component
summary: "修理包侧按护甲／次数／燃料优先分支补满目标，成功后消费材料并回调，经 REPAIR 动作匹配锻造材料标签。"
aliases: ["锻造修理包", "亮茄修理包", "虚空修理包", "forgerepair"]
tags: [component, item, callback, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/forgerepair.lua", "components/forgerepairable.lua", "prefabs/forge_repair_kits.lua", "componentactions.lua", "actions.lua", "constants.lua", "recipes.lua", "stategraphs/SGwilson.lua", "stategraphs/SGwilson_client.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# forgerepair

## 快速判断

- 职责与适用：挂在修理材料上，一次将目标的一种选定耐久补到 100%；目标门禁在 [[reference/components/forgerepairable]]。当前原版三种修理包与月亮／暗影锻造等配方仍在使用此机制。[forgerepair.lua:28](../../../raw/current/scripts/components/forgerepair.lua#L28 [forge_repair_kits.lua:56](../../../raw/current/scripts/prefabs/forge_repair_kits.lua#L56 [recipes.lua:855](../../../raw/current/scripts/recipes.lua#L855
- 优先使用：材料 SetRepairMaterial 对应 FORGEMATERIALS，目标同时装 forgerepairable 并正确维护 repairable。实际动作名是 **ACTIONS.REPAIR**，forgerepair 是材料候选处理器名。[componentactions.lua:1279](../../../raw/current/scripts/componentactions.lua#L1279 [actions.lua:984](../../../raw/current/scripts/actions.lua#L984
- 重要副作用：目标 armor→finiteuses→fueled 按组件存在性选一个 elseif 分支，成功改目标后先消费材料，再执行材料回调；不是遍历修理所有不足项。[forgerepair.lua:28](../../../raw/current/scripts/components/forgerepair.lua#L28
- 修改边界：修理包在主模拟安装组件，动作候选通过标签供客户端判断；直接 OnRepair 不查目标材料匹配、repairable 标志或动作资格，不宜代替目标 Repair。[forge_repair_kits.lua:34](../../../raw/current/scripts/prefabs/forge_repair_kits.lua#L34 [forgerepair.lua:28](../../../raw/current/scripts/components/forgerepair.lua#L28

## 关键接口

| 接口／字段 | 参数或返回 | 条件、作用与证据 |
|---|---|---|
| `SetRepairMaterial(material)`／`repairmaterial` | 默认 nil | 属性变化删除旧材料标签并添加 `forgerepair_<material>`。候选枚举 FORGEMATERIALS，不能只给任意字符串就保证出现动作。[forgerepair.lua:1](../../../raw/current/scripts/components/forgerepair.lua#L1 [constants.lua:1962](../../../raw/current/scripts/constants.lua#L1962 |
| `SetOnRepaired(fn)` | `fn(repair_item, target, doer)` | 修复与材料消耗完成后调用；返回值不消费，回调执行时材料可能已被 Remove，不能假定仍是有效实体。[forgerepair.lua:24](../../../raw/current/scripts/components/forgerepair.lua#L24 [forgerepair.lua:47](../../../raw/current/scripts/components/forgerepair.lua#L47 |
| `OnRepair(target, doer)` | 成功 true；未修成无显式返回 | 按优先分支检查并 SetPercent(1)，成功才消费材料；自身不验证 doer、目标组件归属或材料一致性。[forgerepair.lua:28](../../../raw/current/scripts/components/forgerepair.lua#L28 |

FORGEMATERIALS 本版为 LUNARPLANT=`lunarplant`、VOIDCLOTH=`voidcloth`、WAGPUNKBITS=`wagpunk_bits`；标签由属性回调生成，不应把常量键名直接当作标签后缀。[constants.lua:1962](../../../raw/current/scripts/constants.lua#L1962

## 关键流程

### 单项补满、材料消耗与回调顺序

1. 目标有 armor 就仅查 IsDamaged，成立后 armor:SetPercent(1)。即使护甲已满、finiteuses 不足，也不会落入后续分支。[forgerepair.lua:28](../../../raw/current/scripts/components/forgerepair.lua#L28
2. 没有 armor 才考虑 finiteuses，且 GetPercent()<1 才补满；两者都没有才考虑 fueled，并同样要求比例<1。任一选定分支无损坏时不消费、不回调，也不尝试更低优先级组件。[forgerepair.lua:35](../../../raw/current/scripts/components/forgerepair.lua#L35
3. 成功后材料若有 finiteuses，则 Use(1)；否则有 stackable 就 Get 一份并 Remove；两者都没有就 Remove 材料本身。有限次数工具优先于堆叠消费，不是同时扣两者。[forgerepair.lua:47](../../../raw/current/scripts/components/forgerepair.lua#L47
4. 然后执行材料 onrepaired(repair_item,target,doer)，返回 true；目标 Repair 收到 true 后，才再执行目标 onrepaired(target,doer,repair_item)。两端签名与顺序不同，均不能当作“修理前”钩子。[forgerepair.lua:56](../../../raw/current/scripts/components/forgerepair.lua#L56 [forgerepairable.lua:58](../../../raw/current/scripts/components/forgerepairable.lua#L58

这里的成功由上述判断分支置位，不检查 SetPercent 调用后的实际净变化；特殊耐久组件状态仍须核验，不能把 true 解释为任意目标都已可靠恢复。三类组件如何刷新目标可修标签在 [[reference/components/forgerepairable#关键流程]] 维护。[forgerepair.lua:28](../../../raw/current/scripts/components/forgerepair.lua#L28

### 候选动作与执行资格

forgerepair 的 USEITEM 候选要求右键，排除漂浮目标；骑乘者只能对自己作为 GrandOwner 的目标产生候选，非骑乘者搬着重物时直接拒绝。随后遍历 FORGEMATERIALS，找到材料的 forgerepair_ 标签，只在目标有匹配 forgerepairable_ 标签时加入 REPAIR。[componentactions.lua:1279](../../../raw/current/scripts/componentactions.lua#L1279

REPAIR 执行先走目标的普通 repairable，只有它不存在才检查 forgerepairable；锻造分支使用 act.invobject 并要求其有 forgerepair，然后调用目标 Repair。目标同时装两套组件不会按材料自动挑选锻造分支。漂浮／骑乘等候选限制不由 OnRepair 再检查。[actions.lua:984](../../../raw/current/scripts/actions.lua#L984

原版人物两端 SG 根据目标 repairshortaction 标签选 doshortaction 或 dolongaction；组件不自行播放修理动画或规定固定耗时。[SGwilson.lua:1031](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1031 [SGwilson_client.lua:360](../../../raw/current/scripts/stategraphs/SGwilson_client.lua#L360

## 典型接入

### 原版三种当前修理包

forge_repair_kits 的 MakeKit 在主模拟添加 stackable、forgerepair，设置材料常量及 OnRepaired；该回调只向 doer 发 repair 事件。注册 lunarplant_kit、voidcloth_kit、wagpunkbits_kit，分别对应三种 FORGEMATERIALS。[forge_repair_kits.lua:1](../../../raw/current/scripts/prefabs/forge_repair_kits.lua#L1 [forge_repair_kits.lua:34](../../../raw/current/scripts/prefabs/forge_repair_kits.lua#L34 [forge_repair_kits.lua:56](../../../raw/current/scripts/prefabs/forge_repair_kits.lua#L56

当前配方分别配置月亮锻造站、暗影锻造站与 TECH.LOST；这些现用装备修理包需要按本机制审阅。文件名中的 forge 不能作为“仅熔炉模式／已废弃”的判断依据。此处保留原版接线参照，不额外构造缺少目标破损／恢复流程的 demo。[recipes.lua:855](../../../raw/current/scripts/recipes.lua#L855 [recipes.lua:863](../../../raw/current/scripts/recipes.lua#L863 [recipes.lua:871](../../../raw/current/scripts/recipes.lua#L871

## 常见陷阱与限制

- **目标可修标签等于本次必成功**：目标标志还会再查，OnRepair 也按实际耐久判断；多耐久组件的优先分支可能已满，操作便不消费，不能自动跳到另一类。[forgerepairable.lua:58](../../../raw/current/scripts/components/forgerepairable.lua#L58 [forgerepair.lua:28](../../../raw/current/scripts/components/forgerepair.lua#L28
- **修理包回调可以再扣一份**：组件已在回调之前消费；只有确认自定义流程需要额外消费时才另加逻辑，不能重复结算原版一份消耗。[forgerepair.lua:47](../../../raw/current/scripts/components/forgerepair.lua#L47
- **组件自己保存材料配置或移除时清标签**：没有存读或 OnRemoveFromEntity 钩子；配置由材料 Prefab 重建。热移除组件不会执行 SetRepairMaterial(nil) 的标签撤销逻辑。[forgerepair.lua:1](../../../raw/current/scripts/components/forgerepair.lua#L1

## 验证记录

- 源码审阅：2026-10-26，Steam buildid 26148036；覆盖材料组件全文、目标调用／回调顺序、三种修理包及当前配方、材料常量、USEITEM 候选、REPAIR 分派与两端 SG 映射。未展开所有可修装备、完整堆叠／有限次数消费内部或其他角色 SG。
- 结构检查：validate_page 及源码引用 strict 检查通过；无新增完整 Lua 示例。机械检查不代替游戏运行。
- 游戏运行：未运行；多耐久优先级、材料最后一件／最后一次、漂浮／骑乘候选、回调时实体有效性及读档待实测。

相关：[[reference/components/forgerepairable]]；普通材料修复见 [[reference/components/repairer]]；衣物缝补见 [[reference/components/sewing]]。
