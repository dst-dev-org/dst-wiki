---
title: fan：扇子回调与动作接入
created: 2026-10-22
updated: "2026-10-28"
type: component
summary: 为扇子动作配置使用和吟唱回调，区分原版降温灭火、自定义干燥与耐久消费。
aliases: [fan, 羽毛扇, 降温扇, 扇子干燥, 吟唱扇]
tags: [component, item, callback, actions, server-client]
dst_build: "steam-buildid:26230496"
sources: [components/fan.lua, prefabs/fans.lua, actions.lua, componentactions.lua, stategraphs/SGwilson.lua, stategraphs/SGwilson_client.lua, bufferedaction.lua, entityscript.lua, components/finiteuses.lua, components/moisture.lua, tuning.lua]
reliability: verified
validation: source-reviewed
status: active
---

# fan：扇子回调与动作接入

## 快速判断

- 职责与适用：`fan` 是回调分发组件，本身不降温、不干燥、不扣耐久；效果来自 `onusefn/onchannelingfn`，使用条件来自 `canusefn`。[完整实现](../../../raw/current/scripts/components/fan.lua#L1-L54)
- 优先使用：改羽毛扇效果时包装原版 `onusefn`；新增持续前摇效果才考虑吟唱回调。原版注册的是 `featherfan` 与 `perdfan`，文件名 `fans` 不是物品注册名。[组装](../../../raw/current/scripts/prefabs/fans.lua#L114-L125)、[注册](../../../raw/current/scripts/prefabs/fans.lua#L169-L170)
- 重要副作用：`Fan/Channel` 满足门禁并调用回调就返回 `true`，忽略回调返回值；空效果回调也可能使普通动作成功并消耗耐久。[分发](../../../raw/current/scripts/components/fan.lua#L40-L52)、[成功后的消费](../../../raw/current/scripts/bufferedaction.lua#L22-L37)
- 修改边界：原版服务端添加组件并执行效果；客户端依靠 pristine 前的 `fan/channelingfan` 标签及预测 SG。仅在服务端临时设置吟唱回调，不能代替客户端构造标签。[原版端边界](../../../raw/current/scripts/prefabs/fans.lua#L95-L125)、[客户端吟唱判断](../../../raw/current/scripts/stategraphs/SGwilson_client.lua#L3866-L3878)

## 关键接口

| 接口／字段 | 签名或约束 | 条件、作用与证据 |
|---|---|---|
| `SetCanUseFn(fn)` | `fn(item, target)` 返回真值允许 | `nil` 为无门禁；普通使用和每次吟唱调用都检查。调用方不把玩家作为独立参数传入。[设置](../../../raw/current/scripts/components/fan.lua#L14-L20)、[执行](../../../raw/current/scripts/components/fan.lua#L40-L52) |
| `SetOnUseFn(fn)`、`Fan(target)` | `fn(item, target)`；成功返回 `true`，否则无显式返回 | `ACTIONS.FAN` 把动作目标或使用者交给 `Fan`；物品拟态检查先于调用。[实现](../../../raw/current/scripts/components/fan.lua#L47-L52)、[动作](../../../raw/current/scripts/actions.lua#L3943-L3949) |
| `SetOnChannelingFn(fn)`、`Channel(target)` | 回调同上；成功返回 `true` | 设置非空回调添加 `channelingfan`，清空则移除；`IsChanneling()` 只判断回调非空。[标签与状态](../../../raw/current/scripts/components/fan.lua#L22-L45) |
| `SetOverrideSymbol(symbol)` | 符号名字符串 | 供玩家 SG 替换 `fan01` 使用；默认使用 `swap_fan`，不会自动加载新动画资源。[设置](../../../raw/current/scripts/components/fan.lua#L32-L38)、[消费者](../../../raw/current/scripts/stategraphs/SGwilson.lua#L19453-L19463) |

### 普通效果、吟唱和耐久各自执行

原版 `featherfan` 与 `perdfan` 共用 `OnUse`：以**目标**为圆心找一定范围的火、阴燃、玩家和有物品温度标签的实体；对可燃物调用 `Extinguish(true, 0)`，对有温度的实体降温。温差夹在 `FEATHERFAN_COOLING` 和 0 之间，本快照上限每次降 50、目标最低温度参数 2.5、范围 7；已低于参数值不会被加热。该回调没有减潮湿或干燥物品的步骤。[效果](../../../raw/current/scripts/prefabs/fans.lua#L16-L33)、[参数](../../../raw/current/scripts/tuning.lua#L3036-L3039)

`perdfan` 还配置 `OnChanneling`：耐久大于 3 且找到可用偏移才额外消耗 2 并生成龙卷风。玩家 SG 在第 30、50 帧调用 `Channel`，第 70 帧才执行普通 `FAN` 动作，因此吟唱阶段与普通降温并非互斥。[吟唱效果](../../../raw/current/scripts/prefabs/fans.lua#L39-L75)、[perdfan 构造](../../../raw/current/scripts/prefabs/fans.lua#L148-L166)、[服务端时间线](../../../raw/current/scripts/stategraphs/SGwilson.lua#L19443-L19486)

普通动作耐久由宿主 `finiteuses:SetConsumption(ACTIONS.FAN, 1)` 配置。成功的 `BufferedAction` 调用物品 `OnUsedAsItem`，再分发给 `finiteuses` 扣耗，可能受使用者效率/消费修正影响；直接调用 `fan:Fan(target)` 不经过这条动作消费链。吟唱回调里的 `Use(2)` 是另一笔消耗。[宿主配置](../../../raw/current/scripts/prefabs/fans.lua#L123-L125)、[动作成功](../../../raw/current/scripts/bufferedaction.lua#L22-L37)、[组件分发](../../../raw/current/scripts/entityscript.lua#L1811-L1814)、[耐久消费](../../../raw/current/scripts/components/finiteuses.lua#L101-L114)

### 保存与移除

`fan` 没有保存/加载和周期任务；回调由 prefab 构造时重新配置。移除组件只移除 `channelingfan` 标签；原版耐久由 `finiteuses` 保存/恢复，耗尽由宿主的 `SetOnFinished(inst.Remove)` 删除物品。[组件清理](../../../raw/current/scripts/components/fan.lua#L10-L12)、[耐久保存](../../../raw/current/scripts/components/finiteuses.lua#L48-L58)、[耗尽处理](../../../raw/current/scripts/components/finiteuses.lua#L64-L87)、[宿主配置](../../../raw/current/scripts/prefabs/fans.lua#L123-L125)

## 典型接入

### 羽毛扇保留降温灭火，并额外降低目标潮湿

下面是独立模组完整 `modmain.lua`，仅包装已注册的 `featherfan`；不增加资源、不改吟唱标签。保留原版范围降温与灭火，然后只对动作目标（自用时为使用者）的 `moisture` 降低 5 点；这不是范围干燥，也不改变库存物品湿度。

```lua
AddPrefabPostInit("featherfan", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    local fan = inst.components.fan
    if fan == nil or fan.onusefn == nil then
        return
    end
    local old_onuse = fan.onusefn
    fan:SetOnUseFn(function(item, target)
        old_onuse(item, target)
        if target ~= nil and target:IsValid()
            and target.components.moisture ~= nil then
            target.components.moisture:DoDelta(-5)
        end
    end)
end)
```

`-5` 的单位是潮湿点，`DoDelta` 夹到有效范围并发送 `moisturedelta`，强制干燥状态下提前返回。门禁在包装回调之前执行；普通动作仍按原配置消耗耐久，勿在此再 `Use(1)`。删除代码并重新载入世界即撤销，已降低的潮湿不反向补回；不新增持续任务或额外清理。[潮湿实现](../../../raw/current/scripts/components/moisture.lua#L132-L147)、[回调门禁](../../../raw/current/scripts/components/fan.lua#L47-L52)

原版物品动作候选由组件动作表收集，`ACTIONS.FAN` 的服务端动作再进入回调；本例保持原入口，不新增一个只手动调用组件的动作。[目标使用](../../../raw/current/scripts/componentactions.lua#L1231-L1233)、[自用](../../../raw/current/scripts/componentactions.lua#L2707-L2708)、[动作](../../../raw/current/scripts/actions.lua#L3943-L3949)

## 常见陷阱与限制

- 在 `onusefn` 内 `return false` 无法拒绝使用，也不会阻止普通耐久消费；拒绝条件应放在 `canusefn`。成功与否由分发函数决定。[分发](../../../raw/current/scripts/components/fan.lua#L40-L52)
- 仅添加 `fan` 不会自动获得降温或干燥效果，需配置回调；仅设置 `onchannelingfn` 也不代表无限周期吟唱，原版玩家 SG 只有两个指定帧调用。[组件实现](../../../raw/current/scripts/components/fan.lua#L1-L8)、[时间线](../../../raw/current/scripts/stategraphs/SGwilson.lua#L19466-L19486)
- 直接复用 `perdfan` 吟唱回调时必须同时满足 `finiteuses/inventoryitem` 等宿主前提；其龙卷风生成失败时回调也不会据此返回组件失败，不能据返回值推断龙卷风已产生。[宿主依赖](../../../raw/current/scripts/prefabs/fans.lua#L39-L75)

## 验证记录

- 版本沿用：2026-10-25，`steam-buildid:26148036`；原结论复用、锚点迁移 8 处（actions.lua FAN、componentactions.lua 自用候选、tuning.lua 参数、entityscript.lua 分发、SGwilson.lua use_fan 三区间、SGwilson_client.lua 客户端吟唱）；依据：actions.lua／componentactions.lua 枢纽 diff 审阅＋变更清单命中上述六文件，use_fan 时间线（30／50／70 帧）、channelingfan 判定与 FAN／候选代码块逐行同旧版。未运行游戏。
- 源码审阅：2026-10-22，Steam buildid 25643519；覆盖回调门禁/返回、两个真实宿主注册、普通降温灭火、吟唱时序、客户端标签、动作耐久链、保存清理与目标减潮湿例子。
- 游戏运行：未运行；未实测玩家/物品范围降温、动画中断、多人预测或模组间回调覆盖。
- 结构与语法：本页通过页面格式和严格源码链接检查；完整 `modmain.lua` 通过 Lua 5.1 语法检查。检查不验证游戏效果。
