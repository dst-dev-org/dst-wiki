---
title: "tradable"
created: "2026-10-11"
updated: "2026-10-28"
type: "component"
summary: "配置交给原版交易实体的物品数据，区分材料端与接收端的规则。"
aliases: ["可交易物品", "猪王换金", "蚁狮贡品", "交易价值", "tradable"]
tags: ["component", "item", "mechanism"]
dst_build: "steam-buildid:26230496"
sources: ["components/tradable.lua", "components/trader.lua", "modutil.lua", "prefabs/antlion.lua", "prefabs/pigking.lua"]
reliability: "verified"
validation: "source-reviewed"
status: "active"
---

# tradable

## 快速判断

- 职责与适用：被交易物品的数据载体；接收方用 [[reference/components/trader]]，tradable 本身只有数据没有交易执行方法。[components/tradable.lua:1](../../../raw/current/scripts/components/tradable.lua#L1
- 优先使用：根据接收方实际测试设置 goldvalue、tradefor 或 rocktribute，不把它们当通用商店价格。[prefabs/pigking.lua:605](../../../raw/current/scripts/prefabs/pigking.lua#L605 [prefabs/antlion.lua:117](../../../raw/current/scripts/prefabs/antlion.lua#L117
- 重要副作用：原版猪王按 goldvalue 与 tradefor 发奖励；蚁狮按 rocktribute 延后愤怒。[prefabs/pigking.lua:74](../../../raw/current/scripts/prefabs/pigking.lua#L74 [prefabs/antlion.lua:141](../../../raw/current/scripts/prefabs/antlion.lua#L141
- 修改边界：服务端配置物品；仅加组件不保证所有接收方接受，具体判定由对方 trader 决定。[components/trader.lua:125](../../../raw/current/scripts/components/trader.lua#L125

## 关键接口

| 接口／字段 | 约定 | 作用与证据 |
|---|---|---|
| `goldvalue` | 默认 0；猪王示例用正整数 | 猪王接受测试和奖励次数使用；不是所有 NPC 的金币价。[components/tradable.lua:1](../../../raw/current/scripts/components/tradable.lua#L1 [prefabs/pigking.lua:74](../../../raw/current/scripts/prefabs/pigking.lua#L74 |
| `tradefor` | Prefab 名称数组或 nil；构造器未赋数组 | 猪王奖励逻辑逐项生成，与基础金块之外的奖励相关。[prefabs/pigking.lua:80](../../../raw/current/scripts/prefabs/pigking.lua#L80 |
| `rocktribute` | 数值或 nil；构造器未赋非 nil | 蚁狮要求正值；具体单位转换由其 TUNING 决定。[prefabs/antlion.lua:117](../../../raw/current/scripts/prefabs/antlion.lua#L117 [prefabs/antlion.lua:141](../../../raw/current/scripts/prefabs/antlion.lua#L141 |

## 典型接入

### 让燧石可作为猪王换金材料

目标模组 `modmain.lua`；沿用 flint 资源，服务端添加 tradable 并设 1 点 goldvalue。猪王正常接受／奖励路径见 [prefabs/pigking.lua:598](../../../raw/current/scripts/prefabs/pigking.lua#L598 和 [prefabs/pigking.lua:74](../../../raw/current/scripts/prefabs/pigking.lua#L74)，postinit 见 [modutil.lua:593](../../../raw/current/scripts/modutil.lua#L593)。

```lua
AddPrefabPostInit("flint", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    if inst.components.tradable == nil then inst:AddComponent("tradable") end
    inst.components.tradable.goldvalue = 1
end)
```

仅声明该材料对猪王的基础价值；是否能 GIVE 还受猪王当前状态和动作规则限制。其他交易实体不自动采用这个价格。停用后新实例恢复原数据；无新增奖励 Prefab／资源。

## 常见陷阱与限制

- **有 tradable 就能交给任何东西**：对方 trader 的测试才决定接受；部分接收方允许 nontradable，不能把组件存在当最终判定。[components/trader.lua:125](../../../raw/current/scripts/components/trader.lua#L125
- **给待修装备加 tradable 就能吃材料**：待修装备是接收方，需要 trader + 修复回调；tradable 配给材料。[components/trader.lua:129](../../../raw/current/scripts/components/trader.lua#L129

## 验证记录

- 版本沿用（2026-10-25）：「原始哈希等价批量沿用-24」批次机械沿用——所列来源原始字节均未变，结论与行号直接沿用；页内筛查与枢纽反查无命中。共享依据见 [build-26148036 的 Wiki 沿用记录](../../updates/build-26148036/index.md#wiki-沿用记录)。


- 源码依据复核：2026-10-17，Steam buildid 25643519；核对范围：材料端数据与猪王 goldvalue／tradefor、蚁狮 rocktribute 消费、燧石换金示例，依据见上文对应锚点；未穷举所有接收方、活动奖励与角色资格。
- 历次结构／语法检查：元数据、源码路径／行号、链接／章节、标签、大小和导航登记检查通过；本页 Lua 代码经 Full Moon 3.0.0／Lua 5.1 语法及原字节往返检查。
- 游戏运行：未运行；案例的多人交互、存档重载和其他模组兼容仍需游戏验证。

相关：[[reference/components/trader#给物品投喂材料]]；[[reference/components/inventoryitem]]。
