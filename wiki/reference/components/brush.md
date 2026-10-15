---
title: "brush"
created: "2026-10-19"
updated: "2026-10-28"
type: component
summary: "刷毛工具的空标记；候选、目标结算与工具耐久分别由动作、brushable 和 finiteuses 处理。"
aliases: ["刷子耐久减半", "刷毛工具", "brush"]
tags: [component, item, actions, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/brush.lua", "componentactions.lua", "actions.lua", "prefabs/brush.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# brush

## 快速判断

- 职责与适用：工具端空标记，构造只保存 inst，没有 Brush、次数或回调接口。刷毛目标见 [[reference/components/brushable]]。[完整定义](../../../raw/current/scripts/components/brush.lua#L1-L5)
- 优先使用：原刷子的消耗调整走 finiteuses:SetConsumption(ACTIONS.BRUSH, amount)，不向 brush 填造耐久字段。[工具配置](../../../raw/current/scripts/prefabs/brush.lua#L49-L55)
- 重要副作用：物品组件动作检查目标 brushable 标签收集 BRUSH；执行时检查战斗、死亡和拟态，再调用目标 Brush。[候选](../../../raw/current/scripts/componentactions.lua#L1082-L1086) [执行](../../../raw/current/scripts/actions.lua#L4205-L4216)
- 修改边界：标记不提供新物品资源、装备动作或目标掉毛；原刷子另配武器、库存、装备和有限次数组件。[宿主](../../../raw/current/scripts/prefabs/brush.lua#L45-L63)

## 关键接口

| 所在层 | 职责与证据 |
|---|---|
| `components.brush` | 只有 inst；完整文件没有公开方法、事件或存读钩子。[定义](../../../raw/current/scripts/components/brush.lua#L1-L5) |
| 候选／执行 | 左键候选识别目标标签；ACTIONS.BRUSH 调目标 brushable 并返回 true，不能据此推断实际产毛或增加驯化。[候选](../../../raw/current/scripts/componentactions.lua#L1082-L1086) [返回](../../../raw/current/scripts/actions.lua#L4212-L4214) |
| `finiteuses` | 原刷子 BRUSH 消耗 1，武器 attackwear 为 3；刷毛与攻击消耗分开配置，通用动作消费协议复用 [[reference/components/finiteuses]]。[配置](../../../raw/current/scripts/prefabs/brush.lua#L45-L55) |
| 装备显示 | onequip／onunequip 切换手持符号和手臂显示，不由标记完成。新工具骨架见 [[cookbook/equipment/hand]]。[显示](../../../raw/current/scripts/prefabs/brush.lua#L7-L16) |

## 典型接入

### 原刷子每次刷毛基准耐久改为半次

目标模组 `modmain.lua`，无新资源。只覆盖原刷子的 BRUSH 基准消耗，最终消费仍结合原动作／执行者倍率；保留攻击磨损、目标产出和耗尽移除。

```lua
AddPrefabPostInit("brush", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    if inst.components.brush ~= nil
        and inst.components.finiteuses ~= nil then
        inst.components.finiteuses:SetConsumption(GLOBAL.ACTIONS.BRUSH, 0.5)
    end
end)
```

新配置影响后续正常动作消费，不重设当前／最大耐久；目标 Brush 的直接脚本调用不自动等同完整动作消费。消费配置在每次构造应用，停用重载回原配置；剩余耐久由 finiteuses 管。动作倍率见 [[reference/components/efficientuser]]。

## 常见陷阱与限制

- **把产毛量设在刷子上**：产出与 brushed 回调位于目标 brushable；原工具没有相关字段。[标记](../../../raw/current/scripts/components/brush.lua#L1-L5) [目标委托](../../../raw/current/scripts/actions.lua#L4212-L4214)
- **仅装标记就能复制原工具全部行为**：原宿主还明确配置 finiteuses、库存和装备；新 Prefab 仍须正确组织客户端／服务端构造。[原工具](../../../raw/current/scripts/prefabs/brush.lua#L39-L63)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；漂移文件 actions.lua／componentactions.lua 对照 R-01/R-04 hunk 地图逐条核验：BRUSH 候选（componentactions.lua#L1082-L1086）与执行链（actions.lua BRUSH.fn）逐行未变，"捡拾拾取"燃烧 torch 放宽（R-01/R-04/C-27，componentactions.lua:474、actions.lua PICKUP.fn）不触及刷毛域；锚点重定位 3 处、结论无修订；未覆盖：运行验证。

- 源码审阅：2026-10-19，Steam buildid 25643519；覆盖完整标记、物品候选、动作执行和原刷子配置；耐久／倍率复用既有 Wiki。未全面审校玩家动作状态与预测。
- 结构检查：本批格式、来源、链接／行号、大小及登记通过；完整半耗例通过已有 Full Moon Lua 5.1 syntax-only。
- 游戏运行：未运行；手持／库存候选、刷毛半耗、攻击磨损、目标拒绝、耗尽和多模组覆盖待实测。
