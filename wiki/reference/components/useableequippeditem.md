---
title: "useableequippeditem"
created: "2026-10-20"
updated: "2026-10-28"
type: component
summary: "已装备物品开始／停止协议与使用者回调；区分启动拒绝、状态标签、动作门禁及宿主清理。"
aliases: ["服务端停止WX78无人机遥控", "已装备物品持续使用", "useableequippeditem"]
tags: [component, equipment, actions, callback, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/useableequippeditem.lua", "actions.lua", "componentactions.lua", "prefabs/wx78_drone_zap.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# useableequippeditem

## 快速判断

- 职责与适用：为已装备物品提供 USEEQUIPPEDITEM／STOPUSINGEQUIPPEDITEM 的两态使用协议；不是背包被动效果，也不是 [[reference/components/useableitem]] 的 USEITEM 协议。[动作](../../../raw/current/scripts/actions.lua#L3381-L3398) [候选](../../../raw/current/scripts/componentactions.lua#L3044-L3052)
- 优先使用：SetOnUseFn(fn)／SetOnStopUseFn(fn)，由宿主安排实际效果和结束入口；两个回调均接收 item,doer。[接口](../../../raw/current/scripts/components/useableequippeditem.lua#L20-L59)
- 重要副作用：启动先置 inuse=true，再调用 onuse；明确 false 会回滚并返回 false,reason，nil 为兼容旧回调仍算成功。停止仅在使用中调用回调，且本身不返回成功布尔。[启动／停止](../../../raw/current/scripts/components/useableequippeditem.lua#L32-L59)
- 修改边界：组件方法不查装备、持有或技能；默认启动动作仅检查已装备与 doer 的 inventory 开启条件。原遥控器另由技能／装备链安装组件，不能只 AddComponent 就声称复现无人机技能。[动作门禁](../../../raw/current/scripts/actions.lua#L3381-L3391) [动态安装](../../../raw/current/scripts/prefabs/wx78_drone_zap.lua#L292-L305) [技能条件](../../../raw/current/scripts/prefabs/wx78_drone_zap.lua#L399-L406)

## 关键接口

| 接口／字段 | 条件、作用与证据 |
|---|---|
| `inuse`／`IsInUse()` | 初值 false；属性变化增删 equipped_and_inuse 标签，候选据此选开始或停止动作。状态不是独立效果实现。[属性](../../../raw/current/scripts/components/useableequippeditem.lua#L1-L14) [读取](../../../raw/current/scripts/components/useableequippeditem.lua#L28-L30) [消费者](../../../raw/current/scripts/componentactions.lua#L3044-L3052) |
| `StartUsingItem(doer)` | 已在使用时返回 false；onuse(item,doer) 返回 false,reason 时撤销 inuse，否则返回 true。状态回滚不撤销回调已经产生的副作用。[完整方法](../../../raw/current/scripts/components/useableequippeditem.lua#L32-L47) |
| `StopUsingItem(doer)` | 不在使用中直接返回；否则先清 inuse，再调用 onstopuse(item,doer)。重复调用不会再次执行回调，返回 nil 不说明没停止。[完整方法](../../../raw/current/scripts/components/useableequippeditem.lua#L49-L59) |
| `OnRemoveFromEntity()` | 只移除 equipped_and_inuse 标签，不调用 StopUsingItem／业务清理；组件也没有通用卸装监听或存读实现。[移除](../../../raw/current/scripts/components/useableequippeditem.lua#L16-L18) [完整定义](../../../raw/current/scripts/components/useableequippeditem.lua#L1-L61) |

## 典型接入

### 服务端停止 WX78 遥控器当前无人机操作

目标模组 `modmain.lua`，不新增资源；实际物品名是 wx78_drone_zap_remote，wx78_drone_zap 是被生成的无人机，不能把注册文件名当物品名。[工厂注册](../../../raw/current/scripts/prefabs/wx78_drone_zap.lua#L420-L421)

```lua
AddPrefabPostInit("wx78_drone_zap_remote", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    inst:ListenForEvent("my_mod_stop_drone_use", function(item)
        local useable = item.components.useableequippeditem
        if useable ~= nil and useable:IsInUse() then
            useable:StopUsingItem()
        end
    end)
end)
```

服务端模组逻辑向已取得的遥控器实体发送 `remote:PushEvent("my_mod_stop_drone_use")` 才触发；本例没有新增按键或客户端 RPC。事件不传 doer 是针对这个宿主：原 OnStopUse 不消费 doer，而技能失效时原版也这样停止；不能推广到所有自定义停止回调。[停止与移组件](../../../raw/current/scripts/prefabs/wx78_drone_zap.lua#L285-L305)

原 OnUse 先检查位置，再创建／关联无人机，并安装攻击消耗回调，成功路径没有显式 return；组件接受 nil 正好兼容它。OnStopUse 调用 drone:Kill() 并清引用；本例沿用该业务收尾，不直接改 inuse 或删除 drone 字段。[启动](../../../raw/current/scripts/prefabs/wx78_drone_zap.lua#L259-L283) [停止](../../../raw/current/scripts/prefabs/wx78_drone_zap.lua#L285-L290)

组件由装备时的技能刷新动态增删，PostInit 时未必存在，所以事件执行时再取；保留原装备／卸装回调，卸装会停止使用，实体移除另清理无人机。反复发送事件只对活动状态生效；停用本例后重载不再安装私有监听。[装备刷新](../../../raw/current/scripts/prefabs/wx78_drone_zap.lua#L202-L237) [卸装](../../../raw/current/scripts/prefabs/wx78_drone_zap.lua#L239-L257) [实体移除](../../../raw/current/scripts/prefabs/wx78_drone_zap.lua#L336-L341) [服务端安装与回调](../../../raw/current/scripts/prefabs/wx78_drone_zap.lua#L372-L415)

## 常见陷阱与限制

- **把 false 当一次性成功后退出**：本协议的 false 是启动失败并向动作返回 reason；nil 则成功保留使用状态。不要照搬 useableitem 的一次切换用法。[启动契约](../../../raw/current/scripts/components/useableequippeditem.lua#L32-L47)
- **RemoveComponent 自动收回效果**：组件移除只删标签；应先按实际回调前提停止，原宿主也先 StopUsingItem 再 RemoveComponent。不能因此跳过宿主实体移除清理。[组件移除](../../../raw/current/scripts/components/useableequippeditem.lua#L16-L18) [原版顺序](../../../raw/current/scripts/prefabs/wx78_drone_zap.lua#L299-L305)
- **停止动作得到 nil 就再次补启动**：STOPUSINGEQUIPPEDITEM 原样返回停止方法的 nil；应读 IsInUse 和实际效果判断，不把返回值改造成另一种状态协议。[停止](../../../raw/current/scripts/components/useableequippeditem.lua#L49-L59) [动作](../../../raw/current/scripts/actions.lua#L3395-L3398)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 中的漂移文件 actions.lua（R-01）与 componentactions.lua（R-04）逐 hunk 对照，锚点重定位 5 处（USEEQUIPPEDITEM／STOPUSINGEQUIPPEDITEM 3368→3371、EQUIPPED 候选 2983→3032），结论修订 0 处——两动作定义、启动／停止门禁与 equipped_and_inuse 候选消费均未变，同段新增的 spellcaster nomagiccast 门控与 corruption 处理器（R-04）不影响本页断言；未覆盖：运行验证。

- 源码审阅：2026-10-20，Steam buildid 25643519；覆盖完整组件、动作／候选、遥控器工厂与技能动态挂载、启动回调及卸装／移组件／实体移除清理。未完整审校 WX78 人物状态机、无人机攻击 AI、RPC 权限或新装备动作适配。
- 结构检查：本批格式、来源、链接／行号、大小与登记通过；完整服务端事件例通过已有 Full Moon Lua 5.1 syntax-only。
- 游戏运行：未运行；技能获得／失去、位置拒绝、装备／卸装、重复私有事件、无人机收回与多人动作反馈待实测。
