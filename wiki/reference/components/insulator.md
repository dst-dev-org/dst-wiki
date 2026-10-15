---
title: "insulator"
created: "2026-10-11"
updated: "2026-10-28"
type: "component"
summary: "配置冬季保温或夏季隔热，说明装备生效与温度变化速率的区别。"
aliases: ["冬季保温", "夏季隔热", "保温装备", "隔热装备", "insulator"]
tags: ["component", "equipment", "mechanism"]
dst_build: "steam-buildid:26230496"
sources: ["components/insulator.lua", "components/temperature.lua", "modutil.lua", "prefabs/umbrella.lua"]
reliability: "verified"
validation: "source-reviewed"
status: "active"
---

# insulator

## 快速判断

- 职责与适用：给装备的冬／夏绝热汇总增加数值；减慢相应方向的体温变化，不提供固定热源或恒温。[components/temperature.lua:254](../../../raw/current/scripts/components/temperature.lua#L254 [components/temperature.lua:487](../../../raw/current/scripts/components/temperature.lua#L487
- 优先使用：SetWinter／SetSummer 选方向，再 SetInsulation 配值。[components/insulator.lua:7](../../../raw/current/scripts/components/insulator.lua#L7
- 重要副作用：GetInsulation 返回数值和季节两个值；同组件一次只有一个 type。[components/insulator.lua:27](../../../raw/current/scripts/components/insulator.lua#L27
- 修改边界：人物默认消费装备槽，不消费物品栏携带的 insulator；电气绝缘见 [[reference/components/equippable#关键接口]]。[components/temperature.lua:259](../../../raw/current/scripts/components/temperature.lua#L259

## 关键接口

| 接口／字段 | 约定 | 作用与证据 |
|---|---|---|
| `SetInsulation(val)` | number；默认 0 | 数值参与 SEG_TIME/(SEG_TIME+insulation) 的速率计算，不是目标摄氏温度。[components/insulator.lua:23](../../../raw/current/scripts/components/insulator.lua#L23 [components/temperature.lua:493](../../../raw/current/scripts/components/temperature.lua#L493 |
| `SetSummer()`／`SetWinter()` | 无参数 | 设 SEASONS.SUMMER／WINTER；默认 WINTER。[components/insulator.lua:1](../../../raw/current/scripts/components/insulator.lua#L1 |
| `GetInsulation()`／`GetType()` | number, season／season | GetInsulation 多返回值不要误当独立冬夏值。[components/insulator.lua:27](../../../raw/current/scripts/components/insulator.lua#L27 |

## 典型接入

### 给雨伞配置原版大型夏季隔热档位

目标模组 `modmain.lua`；复用原版 umbrella 资源、组件与穿脱逻辑。原版 common_fn 设置 Summer，pigskin 工厂设基础量。[prefabs/umbrella.lua:124](../../../raw/current/scripts/prefabs/umbrella.lua#L124 [prefabs/umbrella.lua:182](../../../raw/current/scripts/prefabs/umbrella.lua#L182 [modutil.lua:593](../../../raw/current/scripts/modutil.lua#L593

```lua
AddPrefabPostInit("umbrella", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local insulator = inst.components.insulator
    if insulator ~= nil then
        insulator:SetSummer()
        insulator:SetInsulation(GLOBAL.TUNING.INSULATION_LARGE)
    end
end)
```

装备时参加夏季汇总，卸下自动退出汇总；停用后新实例恢复原配置。此值用原版绝热尺度，不保证某种环境下固定存活分钟数或体温；本例无周期任务与新增资源。

## 常见陷阱与限制

- **设 35 就恒温 35 度**：该数值降低变化速率，不指定当前温度；恒温／暖石需要 temperature 和热源路径另行核验。[components/temperature.lua:493](../../../raw/current/scripts/components/temperature.lua#L493
- **一个组件同时保温隔热**：SetSummer 会替换 type；需设计消费者支持的双方贡献，不能连续调用 SetWinter／SetSummer 期待累加。[components/insulator.lua:7](../../../raw/current/scripts/components/insulator.lua#L7

## 验证记录

- 版本沿用（2026-10-25）：「原始哈希等价批量沿用-24」批次机械沿用——所列来源原始字节均未变，结论与行号直接沿用；页内筛查与枢纽反查无命中。共享依据见 [build-26148036 的 Wiki 沿用记录](../../updates/build-26148036/index.md#wiki-沿用记录)。


- 源码依据复核：2026-10-17，Steam buildid 25643519；核对范围：单一冬夏类型、装备槽汇总与温变速率、雨伞夏季隔热示例，依据见上文对应锚点；未核验特定环境的存活时长或恒温组合。
- 历次结构／语法检查：元数据、源码路径／行号、链接／章节、标签、大小和导航登记检查通过；本页 Lua 代码经 Full Moon 3.0.0／Lua 5.1 语法及原字节往返检查。
- 游戏运行：未运行；案例的多人交互、存档重载和其他模组兼容仍需游戏验证。

- 版本沿用：2026-10-28 原始哈希等价批量沿用-70（26148036→26230496，本页 sources 所列文件原始字节全部未变），页内未决项零标记、符号反查无命中、严格链接通过；批次依据见 [[updates/build-26230496/index#wiki-沿用记录]]。
相关：[[reference/components/waterproofer]]；[[reference/components/equippable]]；[[reference/components/temperature#自然温度绝热与容器如何选择]]；[[reference/components/heater]]。
