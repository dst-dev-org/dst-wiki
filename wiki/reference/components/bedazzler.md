---
title: "bedazzler"
created: "2026-10-19"
updated: "2026-10-28"
type: component
summary: "蜘蛛巢装饰工具的目标状态检查与直接耐久消费；角色候选、装饰效果由动作和目标组件协作。"
aliases: ["蜘蛛巢装饰工具耗用减半", "装饰工具消耗", "bedazzler"]
tags: [component, item, actions, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/bedazzler.lua", "prefabs/spiderden_bedazzler.lua", "actions.lua", "componentactions.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# bedazzler

## 快速判断

- 职责与适用：装在蜘蛛巢装饰工具上，检查目标燃烧／烧毁／冻结／已装饰状态，再委托目标 [[reference/components/bedazzlement]]；不是通用换肤或新动画注册接口。[检查与执行](../../../raw/current/scripts/components/bedazzler.lua#L9-L29)
- 优先使用：SetUseAmount(amount) 调本工具直接耗用量；正常交互经 ACTIONS.BEDAZZLE 先 CanBedazzle 再 Bedazzle。[赋值](../../../raw/current/scripts/components/bedazzler.lua#L5-L7) [动作](../../../raw/current/scripts/actions.lua#L5660-L5667)
- 重要副作用：Bedazzle 不自检、先目标 Start 再 finiteuses:Use(use_amount or 1)，自身无成功返回；动作执行层另返回 true。[执行](../../../raw/current/scripts/components/bedazzler.lua#L23-L29)
- 修改边界：CanBedazzle 不检查玩家、距离、spiderden／bedazzleable 或组件存在；候选侧才要求 spiderwhisperer 和相应标签。直接调用不能据测试 true 就认定目标完整合法。[候选](../../../raw/current/scripts/componentactions.lua#L1064-L1068) [测试范围](../../../raw/current/scripts/components/bedazzler.lua#L9-L21)

## 关键接口

| 接口／层 | 条件、作用与证据 |
|---|---|
| `SetUseAmount(use_amount)` | 直接存字段，构造无默认 use_amount；执行处 nil／false 回退 1。以原工具真实配置为基准，不把缺字段当 0。[定义](../../../raw/current/scripts/components/bedazzler.lua#L1-L7) [消费](../../../raw/current/scripts/components/bedazzler.lua#L23-L28) |
| `CanBedazzle(target)` | 返回 true 或 false,reason；按 BURNING→BURNT→FROZEN→ALREADY_BEDAZZLED 顺序。target 必须有效，测试不保证有 bedazzlement。[方法](../../../raw/current/scripts/components/bedazzler.lua#L9-L21) |
| `Bedazzle(target)` | 直接访问 target.components.bedazzlement:Start；有 finiteuses 才直接 Use。目标已装饰时 Start 可以不处理，工具仍继续扣耗用量。[执行](../../../raw/current/scripts/components/bedazzler.lua#L23-L29) |
| 原工具 | spiderden_bedazzler 服务端配库存、检查、finiteuses 耗尽移除和 bedazzler；use_amount 来自 TUNING.BEDAZZLER_USE_AMOUNT，当前为 20。[组装](../../../raw/current/scripts/prefabs/spiderden_bedazzler.lua#L29-L45) [参数](../../../raw/current/scripts/tuning.lua#L6281-L6284) |

## 典型接入

### 原装饰工具每次直接耗用量减半

目标模组 `modmain.lua`，无新资源；以原工具已配置 use_amount 为基准，只改这条直接消费路径，不重设当前／最大耐久或角色候选。

```lua
AddPrefabPostInit("spiderden_bedazzler", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local tool = inst.components.bedazzler
    if tool ~= nil and GLOBAL.type(tool.use_amount) == "number"
        and tool.use_amount > 0 then
        tool:SetUseAmount(tool.use_amount * 0.5)
    end
end)
```

单位为 finiteuses 的使用量，当前原配置从 20 改为 10，影响下一次 Bedazzle 内直接 Use。通用动作消费见 [[reference/components/finiteuses]]；不要仅设 SetConsumption(ACTIONS.BEDAZZLE,...) 来“覆盖”这次 Use，其他模组若再配置动作消费需另核对是否叠加。配置本身不保存，每次构造重设；停用重载回原配置，不恢复已消耗耐久。[消费依据](../../../raw/current/scripts/components/bedazzler.lua#L23-L28)

## 常见陷阱与限制

- **Bedazzle 内部会复查目标并返回成功**：检查只在 CanBedazzle／动作层；直接脚本调用须遵守原候选和执行前提，方法没有显式返回。[动作分工](../../../raw/current/scripts/actions.lua#L5660-L5667)
- **CanBedazzle 可用作任意目标白名单**：它只拒绝几种状态，缺目标组件仍可能返回 true，后续 Start 报错；目标合法性单独检查。[测试](../../../raw/current/scripts/components/bedazzler.lua#L9-L24)
- **重复对已装饰目标调用不会耗材**：Bedazzle 不读取 Start 的结果，仍继续 Use；使用正常检查链防止无效消费。[执行](../../../raw/current/scripts/components/bedazzler.lua#L23-L28)

## 验证记录

- 版本沿用：2026-10-25，`steam-buildid:26148036`；原结论复用、锚点迁移 2 处（actions.lua BEDAZZLE、tuning.lua BEDAZZLER_USE_AMOUNT；componentactions.lua 候选区间行号未变）；依据：actions.lua／componentactions.lua 枢纽 diff 审阅＋变更清单命中 actions.lua、componentactions.lua、tuning.lua，本版均未触及 BEDAZZLE 检查／执行路径，BEDAZZLER_USE_AMOUNT 仍为 20。未运行游戏。
- 源码审阅：2026-10-19，Steam buildid 25643519；覆盖完整组件、原工具配置与当前耗用值、候选标签和动作检查／执行顺序。finiteuses 通用消费复用已有页；未全面审校角色动作预测、其他模组额外消费和所有目标变体。
- 结构检查：本批格式、来源、链接／行号、大小及登记通过；完整半耗例通过已有 Full Moon Lua 5.1 syntax-only。
- 游戏运行：未运行；正常／燃烧／冻结／重复装饰拒绝、剩余耐久耗尽及额外动作消费叠加待实测。
