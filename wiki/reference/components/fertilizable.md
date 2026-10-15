---
title: "fertilizable"
created: "2026-10-17"
updated: "2026-10-28"
type: component
summary: "接收人物自肥并委托宿主回调，扩展沃姆伍德效果时保留成功返回与统一肥料消耗。"
aliases: ["人物自肥", "沃姆伍德施肥效果", "施肥回调", "fertilizable"]
tags: [component, callback, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/fertilizable.lua", "prefabs/wormwood.lua", "actions.lua", "componentactions.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# fertilizable

## 快速判断

- 职责与适用：仅将 `Fertilize(item)` 转交宿主 `onfertlizedfn`；没有内建土壤养分、成长、回血或消耗。[完整实现](../../../raw/current/scripts/components/fertilizable.lua#L2-L12)
- 优先使用：扩展原版沃姆伍德自肥，包装既有回调；肥料供给／现代农田接入用 [[reference/components/fertilizer]]，贫瘠采收植物用 [[reference/components/pickable]]。[角色接入](../../../raw/current/scripts/prefabs/wormwood.lua#L845-L846) [动作分派](../../../raw/current/scripts/actions.lua#L1812-L1842)
- 重要副作用：回调返回值原样传给调用者；原动作仅在真值时消耗肥料。副作用完成却返回 nil 会形成效果与消耗不一致。[委托](../../../raw/current/scripts/components/fertilizable.lua#L8-L10) [结算](../../../raw/current/scripts/actions.lua#L1829-L1840)
- 修改边界：本页服务端角色扩展；添加组件不自动添加 `self_fertilizable` 标签或人物状态／动作动画。界面自肥入口还检查肥料标签及人物可治疗状态。[可用动作](../../../raw/current/scripts/componentactions.lua#L2711-L2718) [原角色标签](../../../raw/current/scripts/prefabs/wormwood.lua#L753-L755)

## 关键接口

| 接口／字段 | 参数或返回 | 条件、作用与证据 |
|---|---|---|
| `onfertlizedfn` | `(receiver,fertilizer_item) -> 成功真值或失败值` | **拼写确为 fertlized**，无额外 i；不传 doer，构造仅留注释，无默认回调。[定义](../../../raw/current/scripts/components/fertilizable.lua#L2-L10) |
| `Fertilize(fertilizer)` | 无回调返回 false；否则返回回调结果 | 不验证 item、不扣肥料、不保存状态；直接调用者须自行负责合法性和成功结算。[实现](../../../raw/current/scripts/components/fertilizable.lua#L8-L10) |
| `ACTIONS.FERTILIZE` 自肥分支 | 无目标或 target==doer，且前面未成功 | 调用执行者组件；此分支位于骑乘限制块之外，不能把目标施肥的骑乘门禁直接推广到自肥。[实现](../../../raw/current/scripts/actions.lua#L1815-L1840) |

沃姆伍德回调读取 [[reference/components/fertilizer#关键接口|肥料三养分]]，分派给催长、堆肥持续治疗和粪肥即时治疗；满足 health.canheal 与有效养分时返回 true。不要用一个固定回血量替换整条原版路径。[回调](../../../raw/current/scripts/prefabs/wormwood.lua#L529-L544) [三种效果](../../../raw/current/scripts/prefabs/wormwood.lua#L502-L523)

## 典型接入

### 沃姆伍德成功自肥后额外恢复 3 点精神

目标模组 `modmain.lua`，仅原版 wormwood；不加资源、不改变原角色标签／动画。原回调先完成并决定成功，本例再调用已建 [[reference/components/sanity#关键接口|sanity:DoDelta]]，保留原返回值，肥料消耗继续由动作负责。[安装位置](../../../raw/current/scripts/prefabs/wormwood.lua#L845-L852) [PostInit](../../../raw/current/scripts/modutil.lua#L593-L599)

```lua
AddPrefabPostInit("wormwood", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local receiver = inst.components.fertilizable
    if receiver == nil or receiver.onfertlizedfn == nil then return end
    local original = receiver.onfertlizedfn
    receiver.onfertlizedfn = function(ent, item)
        local success = original(ent, item)
        if success and ent.components.sanity ~= nil then
            ent.components.sanity:DoDelta(3)
        end
        return success
    end
end)
```

这是每次成功自肥的即时加值，不是每分钟速率；精神上限／redirect 等仍按 sanity 规则。没有额外 OnApplied，失败无奖励；停用后重新初始化不装此包装，已恢复的精神不回滚。不能据此宣称任意新角色只加组件便能使用原版自肥动作。

## 常见陷阱与限制

- **写成 onfertilizedfn**：组件根本不读取该字段，表现为不触发；按实际拼写 `onfertlizedfn`。[实现](../../../raw/current/scripts/components/fertilizable.lua#L9)
- **给目标植物加组件期待通用 FERTILIZE**：普通目标分支不调用该组件，而是 crop／grower／pickable／quagmire_fertilizable；该组件的已核验原版用途是执行者自肥。[动作](../../../raw/current/scripts/actions.lua#L1816-L1831)
- **回调中移除／扣掉肥料**：动作后续还访问它的 fertilizer 并 OnApplied；让统一结算处理消耗，保留回调返回。[结算](../../../raw/current/scripts/actions.lua#L1836-L1840)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 的变更文件（actions.lua、componentactions.lua、prefabs/wormwood.lua）对照更新报告 R-01/R-04、P-140 与 hunk 地图逐条核验，FERTILIZE 自肥分支、fertilizer 候选与 wormwood 标签无语义变化（P-140 新增 plantcreature 标签在原标签锚点之后，属纯新增），锚点重定位 8 处、结论修订 0 处；未覆盖：运行验证。

- 源码审阅：2026-10-17，Steam buildid 25643519；覆盖薄委托、字段拼写、动作成功／消耗、可见条件与沃姆伍德三种养分消费。精神 API 复用当前版本 sanity 已核验接口；未审校 bloomness／持续治疗 Buff 全链。
- 结构检查：格式、来源、链接／行号、大小及登记通过；完整 modmain 示例通过已有 Full Moon Lua 5.1 syntax-only，不代表运行行为。
- 游戏运行：未运行；满血／骑乘入口、鬼魂／重连、技能树、其他模组回调包装顺序和实际精神收益待实测。
