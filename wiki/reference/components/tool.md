---
title: "tool"
created: "2026-10-11"
updated: "2026-10-28"
type: "component"
summary: "给物品配置砍、挖、挖矿等工作动作与效率，并与次数消耗分开。"
aliases: ["工具动作", "砍树效率", "挖矿工具", "多功能工具", "tool"]
tags: ["component", "item", "equipment", "actions"]
dst_build: "steam-buildid:26230496"
sources: ["actions.lua", "components/tool.lua", "prefabs/axe.lua"]
reliability: "verified"
validation: "source-reviewed"
status: "active"
---

# tool

## 快速判断

- 职责与适用：动作能力和工作效率；不保存耐久、也不自己执行对目标的工作。[components/tool.lua:11](../../../raw/current/scripts/components/tool.lua#L11 [actions.lua:1693](../../../raw/current/scripts/actions.lua#L1693
- 优先使用：SetAction(ACTIONS.CHOP, effectiveness)；按次消耗配 [[reference/components/finiteuses]]，手持展示配 [[reference/components/equippable]]。[prefabs/axe.lua:107](../../../raw/current/scripts/prefabs/axe.lua#L107
- 重要副作用：SetAction 增加动作工具标签；只能用 TOOLACTIONS 支持的动作。[components/tool.lua:41](../../../raw/current/scripts/components/tool.lua#L41
- 修改边界：服务端配置；目标须有匹配 workaction 的 workable，效果还受人物 workmultiplier 与反弹规则影响。[actions.lua:1693](../../../raw/current/scripts/actions.lua#L1693

## 关键接口

| 接口／字段 | 约定 | 作用与证据 |
|---|---|---|
| `SetAction(action, effectiveness?)` | Action；默认效率 1 | action.id 必须在 TOOLACTIONS 中；0 仍会登记动作。[components/tool.lua:41](../../../raw/current/scripts/components/tool.lua#L41 |
| `CanDoAction(action)`／`GetEffectiveness(action)` | boolean／number | 前者看键是否存在，不看效率是否大于 0；未配置效率为 0。[components/tool.lua:37](../../../raw/current/scripts/components/tool.lua#L37 |
| `EnableToughWork(flag?)` | 省略参数也开启 | 仅设置 tough 标志；特殊工作／反弹行为仍取决于调用链。[components/tool.lua:29](../../../raw/current/scripts/components/tool.lua#L29 |

## 典型接入

### 将原版斧头基础砍树效率改为 2

以下放目标模组 `modmain.lua`；复用 axe 资源与 CHOP 的原版次数消耗。工作效率由 DoToolWork 消费，耐久独立配置。[actions.lua:1703](../../../raw/current/scripts/actions.lua#L1703 [prefabs/axe.lua:108](../../../raw/current/scripts/prefabs/axe.lua#L108 [prefabs/axe.lua:116](../../../raw/current/scripts/prefabs/axe.lua#L116

```lua
AddPrefabPostInit("axe", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    if inst.components.tool ~= nil then
        inst.components.tool:SetAction(GLOBAL.ACTIONS.CHOP, 2)
    end
end)
```

基础每次工作量设 2；不保证总耗时或最终工作量恰好翻倍。停用后新实例恢复原效率。其他模组同时改动作时要核对最终值；本例没有增加动画／动作／状态图。

### 多功能工具的模式切换

本地片段参照（未随库分发，未测游戏／外部 Hook；[版本与文件指纹](../../../docs/reports/w05/local-mod-references-20261017.md)）：DSTKisaki 的 kisaki_tool.lua:401～430 把 tool 工作动作、finiteuses 消耗、useableitem 切换和 trader 修复分开；MCW 的 mcw_lollipopstaff.lua:130～176 分别登记多动作与使用回调；Sora 的 sora2sword.lua:53～66 用一次操作切换标签并返回 false。它们是选型参照，不能推断所有辅助动作共享同一种耐久规则。

通用设计需同时维护“动作在客户端是否可见”和“服务器是否允许执行”。仅摘 HAMMER_tool／DIG_tool 标签不会删除 tool.actions 中的登记，也不等于所有直接调用失效；效率、耗尽门禁、修复后恢复、模式存档分别核验。工具 SetAction／CanDoAction 的实际规则见 [components/tool.lua:41](../../../raw/current/scripts/components/tool.lua#L41)，一次切换的 inuse 边界见 [[reference/components/useableitem]]。不复制模组的动作扩展或专属状态。

## 常见陷阱与限制

- **SetAction(action, 0) 等于移除能力**：CanDoAction 检查键存在；禁用需同时设计动作可见性与执行检查，不能只把效率归零。[components/tool.lua:47](../../../raw/current/scripts/components/tool.lua#L47
- **工具效率就是耐久消耗**：效率参与目标工作量；finiteuses 的消费按动作链另处理。[actions.lua:1703](../../../raw/current/scripts/actions.lua#L1703 [prefabs/axe.lua:116](../../../raw/current/scripts/prefabs/axe.lua#L116
- **任意跨游戏动作可用**：未知 action.id 会 assert；模组扫描中的外部动作不是当前 DST 的通用能力。[components/tool.lua:41](../../../raw/current/scripts/components/tool.lua#L41

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 中的漂移文件 actions.lua（R-01）逐 hunk 对照，锚点重定位 4 处（DoToolWork 1687→1690、1697→1700），结论修订 0 处——DoToolWork 的基础量／人物倍率分工与 TOOLACTIONS 断言未变，本轮 actions.lua 改动（CORRUPTNIGHTMARE、invalid_hold_action、nomagiccast、PICKUP）均不触及本页；未覆盖：运行验证。

- 源码依据复核：2026-10-17，Steam buildid 25643519；核对范围：动作登记／标签与零效率边界、DoToolWork 的基础量／人物倍率分工、斧头效率示例，依据见上文对应锚点；未核验模组新增动作／SG 或完整模式恢复流程。
- 历次结构／语法检查：元数据、源码路径／行号、链接／章节、标签、大小和导航登记检查通过；本页 Lua 代码经 Full Moon 3.0.0／Lua 5.1 语法及原字节往返检查。
- 游戏运行：未运行；案例的多人交互、存档重载和其他模组兼容仍需游戏验证。

相关：[[reference/components/finiteuses]]；[[reference/components/weapon]]；[[reference/components/workable#关键接口]]；[[reference/components/worker#快速判断|无工具宿主的基础量]]；[[reference/components/workmultiplier#关键流程|人物倍率与结算顺序]]；[[reference/components/efficientuser#关键接口|消费倍率]]。
