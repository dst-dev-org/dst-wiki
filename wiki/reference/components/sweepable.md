---
title: "sweepable"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "可清扫目标组件；布尔字段经属性 setter 钩子同步为实体 sweepable 标签，SWEEP 动作据此放行并回调宿主结算。"
aliases: ["可清扫目标", "清扫标签门禁", "sweepable"]
tags: [component, actions, network-sync, callback]
dst_build: "steam-buildid:26230496"
sources: ["components/sweepable.lua", "class.lua", "componentactions.lua", "components/playercontroller.lua", "actions.lua", "stategraphs/SGwilson.lua", "prefabs/leafpile.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# sweepable

## 快速判断

- 职责与适用：装在可被扫帚清扫的目标上；sweepable 布尔字段每次赋值经 Class 属性 setter 钩子同步为实体 "sweepable" 标签，清扫结果全部由宿主回调承担。工具端标记见 [[reference/components/sweeper]]。[字段与挂钩](../../../raw/current/scripts/components/sweepable.lua#L1-L14)
- 优先使用：SetEnabled(bool) 管标签门禁与候选可见性，SetOnSweepedFn(fn) 管清扫结算；不经过这两方法的字段直写也能触发同步，但语义上以方法为准。[开关](../../../raw/current/scripts/components/sweepable.lua#L16-L22)
- 重要副作用：AddComponent 时构造赋值 self.sweepable=true 即触发挂钩挂标签，目标立即进入候选；SetEnabled(false) 摘标签后全部候选与按钮路径消失。[构造](../../../raw/current/scripts/components/sweepable.lua#L5-L14)
- 修改边界：组件只存在于服务端，客户端仅凭复制的标签判断；组件无存读钩子，enabled 状态不入存档，由宿主载入时按自身状态重设。[全文件](../../../raw/current/scripts/components/sweepable.lua#L1-L31)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `sweepable` | 布尔，默认 true | 属性 setter 钩子（Class 第三参表）：每次赋值触发 onsweepable → inst:AddOrRemoveTag("sweepable", value)；实例字段表先于构造初始化，故构造内赋值也触发挂钩。[setter](../../../raw/current/scripts/components/sweepable.lua#L1-L3) [挂钩](../../../raw/current/scripts/class.lua#L36-L45) [实例初始化](../../../raw/current/scripts/class.lua#L183-L193) |
| `SetEnabled(sweepable)` | `(bool) → nil` | 即对字段赋值：真值挂标签、假值摘标签；不触碰回调。[实现](../../../raw/current/scripts/components/sweepable.lua#L16-L18) |
| `SetOnSweepedFn(fn)` | `(function) → nil` | 存 onsweepfn，无默认值；未设置时 Sweep 仍成功返回 true。[实现](../../../raw/current/scripts/components/sweepable.lua#L20-L22) |
| `Sweep(doer, sweeper)` | `(doer, sweeper) → true` | 调 onsweepfn(inst, doer, sweeper) 后恒返回 true；不校验 enabled，门控完全由标签同步承担。[实现](../../../raw/current/scripts/components/sweepable.lua#L24-L29) |
| 存读 | 无 | 文件内没有 OnSave／OnLoad：enabled 与回调不持久化。[全文件](../../../raw/current/scripts/components/sweepable.lua#L1-L31) |

## 关键流程：SWEEP 动作链路

1. 候选：工具端 USEITEM／EQUIPPED 两段 handler 在右键且目标带 "sweepable" 标签时插入 ACTIONS.SWEEP；动作按钮路径要求目标带标签且进入 pickup_tags 白名单。[USEITEM handler](../../../raw/current/scripts/componentactions.lua#L1824-L1828) [EQUIPPED handler](../../../raw/current/scripts/componentactions.lua#L2494-L2498) [白名单](../../../raw/current/scripts/components/playercontroller.lua#L2249)
2. 执行：ACTIONS.SWEEP（distance 2.5、mount_valid=false、invalid_hold_action=true）的 fn 先经 ShouldItemMimicBeRevealedFor 揭示拟态工具，再在服务端调 target.components.sweepable:Sweep(doer, invobject) 并返回 true；目标没有该组件时落空失败。[定义](../../../raw/current/scripts/actions.lua#L746) [fn](../../../raw/current/scripts/actions.lua#L7126-L7133)
3. 结算：Sweep 只回调宿主 onsweepfn，不自带掉落、动画或耐久消耗；原版玩家动作进入状态图 doswipeaction 状态播放。[回调](../../../raw/current/scripts/components/sweepable.lua#L24-L29) [状态图入口](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1801)

## 典型接入

### 让自定义实体可被扫帚清扫

前提与选择：目标是自定义 Prefab 的服务端构造段，无额外资源；组件构造即挂标签，想要"先生成、后显现"的目标须在构造后立即 SetEnabled(false)，再在合适时机开启。

```lua
local function onsweeped(inst, doer, sweeper)
    -- doer 为清扫玩家，sweeper 为扫帚实体；清扫结算全部在此定义
    inst:Remove()
end

local function master_postinit(inst)
    inst:AddComponent("sweepable")
    inst.components.sweepable:SetOnSweepedFn(onsweeped)
end
```

生效与撤销：AddComponent 起目标带标签并可被两条入口选中；SetEnabled(false) 即从候选摘除且不重置回调，重新 SetEnabled(true) 恢复。回调只定义结算内容，掉落、特效与移除都须自行安排。

原版参照：leafpile 是当前唯一原版消费者——放置后掩藏并延迟 RevealPile 才 SetEnabled(true) 显示，腐坏时 SetEnabled(false)，被扫时 DropLootFor、碎裂并移除。[挂载与回调](../../../raw/current/scripts/prefabs/leafpile.lua#L516-L517) [延迟开启](../../../raw/current/scripts/prefabs/leafpile.lua#L288-L294) [掩藏关闭](../../../raw/current/scripts/prefabs/leafpile.lua#L296-L304) [清扫结算](../../../raw/current/scripts/prefabs/leafpile.lua#L411-L415)

## 常见陷阱与限制

- **Sweep 不校验 enabled**：门控只由标签同步承担，直接脚本调用或强制执行的 SWEEP 在 SetEnabled(false) 后仍会触发 onsweepfn；回调内如需绝对拦截须自行检查组件状态。[Sweep](../../../raw/current/scripts/components/sweepable.lua#L24-L29)
- **只加标签不加组件会白给动作**：客户端候选全凭复制标签，服务端 SWEEP.fn 却要求 target.components.sweepable ~= nil；仅 AddTag 的实体客户端显示动作、服务端执行落空返回失败。[fn 判空](../../../raw/current/scripts/actions.lua#L7129-L7132)
- **enabled 不持久化**：组件无存读钩子，世界重载后标签回到构造默认 true；需要保持隐藏的目标由宿主载入流程重新设回 false，leafpile 即按自身掩藏／显现逻辑重走。[构造默认](../../../raw/current/scripts/components/sweepable.lua#L5-L14) [leafpile 载入重设](../../../raw/current/scripts/prefabs/leafpile.lua#L296-L304)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496（steam-buildid:26230496）；覆盖完整组件文件与 Class setter 钩子机制、ACTIONS.SWEEP 定义与 fn、两段候选 handler、动作按钮白名单、状态图入口及原版消费者 leafpile 的开关与回调；全库检索确认 "sweepable" 标签无其他消费者。未覆盖：标签网络复制逐字段细节与动作预测。
- 结构检查：格式、来源、链接／行号、大小档位通过 validate_page 与 validate_source_links 校验；lua 示例通过 parser_probe 语法检查。
- 游戏运行：未运行；可见性切换、拟态工具清扫、骑乘拦截与重载后状态待实测。

## 相关链接

- [[reference/components/sweeper]]：清扫工具端空标记与动作接线。
- [[reference/components/leafpilemanager]]：leafpile 的世界级限密生成与衰减调度。
