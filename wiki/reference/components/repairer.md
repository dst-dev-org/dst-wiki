---
title: "repairer"
created: "2026-10-17"
updated: "2026-10-28"
type: component
summary: "修复材料的类型、固定值与比例配置，区分材料标签、目标分支与一份材料的消费。"
aliases: ["修复材料", "噩梦燃料修护符", "材料修复量", "repairer"]
tags: [component, item, actions, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/repairer.lua", "components/repairable.lua", "componentactions.lua", "actions.lua", "constants.lua", "entityscript.lua", "prefabs/nightmarefuel.lua", "prefabs/amulet.lua", "prefabs/gears.lua", "prefabs/boatpatch.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# repairer

## 快速判断

- 职责与适用：挂在**修复材料**上，保存材料类型及五类恢复量并生成动作标签；实际结算由目标 [[reference/components/repairable]] 执行，组件自身没有 Repair 方法。[repairer.lua:35](../../../raw/current/scripts/components/repairer.lua#L35)、[repairable.lua:148](../../../raw/current/scripts/components/repairable.lua#L148
- 优先使用：复用原版材料时只改对应修复量，保留 repairmaterial 与其他用途；恢复次数用 finiteusesrepairvalue，固定生命与生命比例分别配置。[repairer.lua:8](../../../raw/current/scripts/components/repairer.lua#L8
- 重要副作用：参数赋值会重建 work_／health_／freshen_／finiteuses_ 材料标签，改变所有匹配目标的候选与恢复量；不是仅影响当前拿着的目标。[repairer.lua:8](../../../raw/current/scripts/components/repairer.lua#L8)、[componentactions.lua:1688](../../../raw/current/scripts/componentactions.lua#L1688
- 修改边界：服务端配置，用既有 REPAIR 链路；修复量默认都是 0，只有材料名不足以提供普通数值修复。使用次数通常仍是一份材料，不读取材料自己的 finiteuses。[repairer.lua:35](../../../raw/current/scripts/components/repairer.lua#L35)、[repairable.lua:172](../../../raw/current/scripts/components/repairable.lua#L172

## 关键接口

| 字段 | 单位／默认 | 条件、用途与证据 |
|---|---|---|
| `repairmaterial` | MATERIALS 值；默认 nil | 与目标完全一致；改值先清除旧材料标签，再依据数值补新标签。自动动作只遍历 MATERIALS，任意字符串不会自动注册。[repairer.lua:26](../../../raw/current/scripts/components/repairer.lua#L26)、[constants.lua:1940](../../../raw/current/scripts/constants.lua#L1940)、[componentactions.lua:1688](../../../raw/current/scripts/componentactions.lua#L1688 |
| `healthrepairvalue` | HP 点数；默认 0 | 与比例任一 >0 时产生 health_<材料>；目标先 DoDelta(固定量)，再结算比例。[repairer.lua:14](../../../raw/current/scripts/components/repairer.lua#L14)、[repairable.lua:107](../../../raw/current/scripts/components/repairable.lua#L107 |
| `healthrepairpercent` | 基础 maxhealth 的比例；默认 0 | 0.1 表示基础上限 10%，不是缺血量 10%；与固定值可以同时生效，但走两次 DoDelta。[repairable.lua:112](../../../raw/current/scripts/components/repairable.lua#L112 |
| `workrepairvalue` | 工作量增量；默认 0 | >0 产生 work_<材料>；目标由 workable:SetWorkLeft 恢复。[repairer.lua:11](../../../raw/current/scripts/components/repairer.lua#L11)、[repairable.lua:117](../../../raw/current/scripts/components/repairable.lua#L117 |
| `perishrepairpercent` | 新鲜度比例增量；默认 0 | >0 产生 freshen_<材料>；0.2 是增加 20 个百分点，不是乘 1.2。[repairer.lua:17](../../../raw/current/scripts/components/repairer.lua#L17)、[repairable.lua:126](../../../raw/current/scripts/components/repairable.lua#L126 |
| `finiteusesrepairvalue` | 次数；默认 0 | >0 产生 finiteuses_<材料>；由目标 finiteuses:Repair 夹取到上限。[repairer.lua:20](../../../raw/current/scripts/components/repairer.lua#L20)、[repairable.lua:135](../../../raw/current/scripts/components/repairable.lua#L135 |
| `boatrepairsound` | 可选声音事件字符串 | repairable 成功分支在目标 SoundEmitter 上播放；不是恢复量，也不自动添加 SoundEmitter。[repairable.lua:168](../../../raw/current/scripts/components/repairable.lua#L168)、[boatpatch.lua:71](../../../raw/current/scripts/prefabs/boatpatch.lua#L71 |

目标一次只选生命→工作量→保鲜→次数中的首个匹配分支，详细门禁、失败与回调顺序集中在 [[reference/components/repairable#恢复分支与顺序]]。数值不在 repairer 里夹取；不把负量当受支持的反向修复方案。

## 典型接入

### 噩梦燃料每份恢复二十五次装备耐久

放在已启用模组 `modmain.lua`，可独立追加，无新增资源或动作。噩梦燃料已是 NIGHTMARE 材料，橙色护符是匹配目标；只修改 finiteusesrepairvalue，不重建物品或改其燃料用途。[nightmarefuel.lua:78](../../../raw/current/scripts/prefabs/nightmarefuel.lua#L78)、[amulet.lua:555](../../../raw/current/scripts/prefabs/amulet.lua#L555

```lua
AddPrefabPostInit("nightmarefuel", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local material = inst.components.repairer
    if material ~= nil and material.repairmaterial == GLOBAL.MATERIALS.NIGHTMARE then
        material.finiteusesrepairvalue = 25
    end
end)
```

25 是绝对次数，非百分比；原版 REPAIR 仍检查目标匹配／缺损、骑乘与动作状态，成功消费一份。影响**所有匹配 NIGHTMARE 的次数修复目标**，不只橙色护符。每次实例创建／载入重新设定，无 repairer 的变体不强加组件；停用模组后恢复材料默认参数，不撤销已经修好的次数。[componentactions.lua:1679](../../../raw/current/scripts/componentactions.lua#L1679)、[repairable.lua:148](../../../raw/current/scripts/components/repairable.lua#L148

需要新建完整“可修目标＋材料”组合时直接读 [[reference/components/repairable#普通斧头用石头修复二十次耐久]]，不在两页维护同一份组合 demo。

### 原版材料模式

齿轮同时配置工作量和生命量，具体修哪类由目标决定；补船板提供固定生命量与声音；噩梦燃料恢复次数。这些是不同消费模式，不按引用数量选示例。[gears.lua:87](../../../raw/current/scripts/prefabs/gears.lua#L87)、[boatpatch.lua:71](../../../raw/current/scripts/prefabs/boatpatch.lua#L71)、[nightmarefuel.lua:84](../../../raw/current/scripts/prefabs/nightmarefuel.lua#L84

## 常见陷阱与限制

- **材料方法直接执行修复**：没有 repairer:Repair；普通行为是 REPAIR 选材料并调用目标 repairable:Repair。不要交换 doer 与材料参数。[actions.lua:984](../../../raw/current/scripts/actions.lua#L984
- **给多种值就全部恢复**：分支独占；生命分支已满会失败，不自动转向次数分支。材料量和目标组件组合必须一起核验，见 [[reference/components/repairable#恢复分支与顺序]]。
- **调整数值只影响一个目标**：字段属于材料实例，所有同类型匹配目标共享这个量；要限定目标需额外设计目标检查或动作链，不把全局材料修改伪装成专属功能。[componentactions.lua:1688](../../../raw/current/scripts/componentactions.lua#L1688
- **移除组件就删除所有标签**：OnRemoveFromEntity 只清四类材料能力标签，未在本组件钩子里移除通用 repairer 标签；宿主还会注销 ComponentActions。不要仅用 HasTag("repairer") 证明组件存在。[repairer.lua:44](../../../raw/current/scripts/components/repairer.lua#L44)、[:56](../../../raw/current/scripts/components/repairer.lua#L56)、[entityscript.lua:670](../../../raw/current/scripts/entityscript.lua#L670
- **参数自动随材料存档**：组件没有 OnSave／OnLoad；固定配置从工厂或 PostInit 重建。动态个体数值要由其所有者另设保存策略，不能假定 stackable 会复制任意自定义字段。[repairer.lua:35](../../../raw/current/scripts/components/repairer.lua#L35

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 的变更文件（componentactions.lua、actions.lua、constants.lua、entityscript.lua、prefabs/amulet.lua）对照更新报告 R-01/R-04、R-06/R-08、P-03 逐条核验，REPAIR 动作、材料标签候选与五类修复量字段无语义变化，锚点重定位 2 处、结论修订 0 处；未覆盖：运行验证。

- 源码审阅：2026-10-17，Steam buildid 25643519；覆盖完整材料字段／标签与移除、REPAIR 候选／分派、目标恢复与消费，以及噩梦燃料／护符／齿轮／补船板所选配置；未审校全部材料和海上动作系统。
- 结构检查：发布时核对格式、来源、大小、链接、源码锚点与唯一登记；完整 modmain 示例以已有 Full Moon／Lua 5.1 作 syntax-only 检查，不验证多人 UI 或游戏结算。
- 游戏运行：未运行；不同目标、满次数、堆叠消费、读档及其他模组同时改材料待实测。

相关：[[reference/components/repairable]]；[[reference/components/finiteuses]]；[[reference/components/fuel]]；[[reference/components/stackable]]。
