---
title: "waterproofer"
created: "2026-10-11"
updated: "2026-10-28"
type: "component"
summary: "区分物品自身不受潮与装备对人物的防雨贡献，避免把携带误作装备生效。"
aliases: ["物品防潮", "装备防雨", "防水", "防水倍率", "waterproofer"]
tags: ["component", "item", "equipment"]
dst_build: "steam-buildid:26230496"
sources: ["components/inventory.lua", "components/inventoryitem.lua", "components/moisture.lua", "components/waterproofer.lua", "prefabs/umbrella.lua"]
reliability: "verified"
validation: "source-reviewed"
status: "active"
---

# waterproofer

## 快速判断

- 职责与适用：组件阻止 inventoryitem 自身湿度更新，并给装备槽防水汇总提供 effectiveness。[components/waterproofer.lua:1](../../../raw/current/scripts/components/waterproofer.lua#L1 [components/inventory.lua:2714](../../../raw/current/scripts/components/inventory.lua#L2714
- 优先使用：`SetEffectiveness(0..1)` 配置贡献；仅物品防潮可取 0，人物完全防雨看装备汇总与其他来源。[components/waterproofer.lua:26](../../../raw/current/scripts/components/waterproofer.lua#L26 [components/moisture.lua:217](../../../raw/current/scripts/components/moisture.lua#L217
- 重要副作用：effectiveness 设为 0 也不会重新启用物品湿度；移除组件才执行 EnableMoisture(true)。[components/waterproofer.lua:14](../../../raw/current/scripts/components/waterproofer.lua#L14
- 修改边界：放物品栏不向人物提供装备防雨贡献；防水不是电气绝缘或温度保温。[components/inventory.lua:2727](../../../raw/current/scripts/components/inventory.lua#L2727

## 关键接口

| 接口／字段 | 约定 | 作用与证据 |
|---|---|---|
| `SetEffectiveness(val)`／`GetEffectiveness()` | number／number；默认 1 | setter 不夹范围；按业务使用 0～1 的比例。[components/waterproofer.lua:22](../../../raw/current/scripts/components/waterproofer.lua#L22 |
| 组件添加／移除 | 物品湿度更新开关 | 若 inventoryitem 已存在，构造器禁湿度；inventoryitem 后添加也检查 waterproofer。[components/waterproofer.lua:9](../../../raw/current/scripts/components/waterproofer.lua#L9 [components/inventoryitem.lua:87](../../../raw/current/scripts/components/inventoryitem.lua#L87 |
| 人物 `inventory:GetWaterproofness()` | 无 slot 时累加装备槽 | 某些 moisture 规则可直接返回 1；人物雨水速率还包含环境／自身修饰。[components/inventory.lua:2714](../../../raw/current/scripts/components/inventory.lua#L2714 [components/moisture.lua:217](../../../raw/current/scripts/components/moisture.lua#L217 |

## 典型接入

### 将原版雨伞防水贡献设为 100%

目标模组 `modmain.lua`；不新增资源，只改原版 umbrella 的服务端值。原版工厂与注册名见 [prefabs/umbrella.lua:168](../../../raw/current/scripts/prefabs/umbrella.lua#L168 和 [prefabs/umbrella.lua:194](../../../raw/current/scripts/prefabs/umbrella.lua#L194)。

```lua
AddPrefabPostInit("umbrella", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    if inst.components.waterproofer ~= nil then
        inst.components.waterproofer:SetEffectiveness(1)
    end
end)
```

人物须装备雨伞才消费该贡献；卸下后不再计入装备汇总，伞自身仍受组件湿度规则影响。停用后新实例恢复原值。此例不阻止全部外部湿度事件，也不修改雨伞耐久。

## 常见陷阱与限制

- **背包里有防水物品就防雨**：默认汇总读取 equipslots；携带效果须另写机制，不能只加本组件。[components/inventory.lua:2727](../../../raw/current/scripts/components/inventory.lua#L2727
- **effectiveness = 0 会让物品受潮**：湿度开关在组件构造／移除，不在 SetEffectiveness；0 只把防水贡献归零。[components/waterproofer.lua:9](../../../raw/current/scripts/components/waterproofer.lua#L9 [components/waterproofer.lua:26](../../../raw/current/scripts/components/waterproofer.lua#L26
- **所有湿度都被防雨拦截**：雨水与其他事件入口消费不同规则；本页仅核验雨水与通用防水查询，不承诺全环境免湿。[components/moisture.lua:212](../../../raw/current/scripts/components/moisture.lua#L212 [components/moisture.lua:239](../../../raw/current/scripts/components/moisture.lua#L239

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 处理范围：锚点重定位 0 处（inventoryitem#L87 引用区间未变；C-19 GetGrandOwner 回退改动在 502+ 行，不在本页引用范围，防水汇总结论不受影响）；结论修订：无。
- 依据：变更清单命中 components/inventoryitem.lua（C-19，detail-components.md:30）。

- 源码依据复核：2026-10-17，Steam buildid 25643519；核对范围：物品湿度开关与装备防雨汇总的区别、effectiveness=0 和雨伞示例，依据见上文对应锚点；未覆盖全部外部湿度事件，不承诺全环境免湿。
- 历次结构／语法检查：元数据、源码路径／行号、链接／章节、标签、大小和导航登记检查通过；本页 Lua 代码经 Full Moon 3.0.0／Lua 5.1 语法及原字节往返检查。
- 游戏运行：未运行；案例的多人交互、存档重载和其他模组兼容仍需游戏验证。

- 版本沿用：2026-10-28 原始哈希等价批量沿用-70（26148036→26230496，本页 sources 所列文件原始字节全部未变），页内未决项零标记、符号反查无命中、严格链接通过；批次依据见 [[updates/build-26230496/index#wiki-沿用记录]]。
相关：[[reference/components/equippable]]；[[reference/components/insulator]]；[[reference/components/inventoryitem]]。
