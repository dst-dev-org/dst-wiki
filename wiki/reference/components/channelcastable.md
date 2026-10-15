---
title: "channelcastable"
created: "2026-10-20"
updated: "2026-10-28"
type: component
summary: "持续施法物品的使用者／效果回调桥接，通过人物 channelcaster 成对结束，保留技能动态挂载与卸装清理。"
aliases: ["服务端停止打火机持续吸火", "持续施法物品", "channelcastable"]
tags: [component, item, callback, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/channelcastable.lua", "components/channelcaster.lua", "prefabs/lighter.lua", "prefabs/player_common.lua", "actions.lua", "componentactions.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# channelcastable

## 快速判断

- 职责与适用：物品侧保存一个 user，提供开始／停止效果回调；人物状态、减速与事件由 channelcaster 管，不与建筑 [[reference/components/channelable]] 的协议混用。[物品定义](../../../raw/current/scripts/components/channelcastable.lua#L1-L27) [人物入口](../../../raw/current/scripts/components/channelcaster.lua#L48-L73)
- 优先使用：设置两个回调，正常启动交给人物 StartChanneling(item)／动作；物品主动取消用 StopChanneling，转交人物完整撤销。[回调设置](../../../raw/current/scripts/components/channelcastable.lua#L13-L19) [取消](../../../raw/current/scripts/components/channelcastable.lua#L64-L71)
- 重要副作用：OnStartChanneling／OnStopChanneling 是内部握手，直接改物品侧不会自动完成旧人物状态转换。活跃期间 unequipped 会停止并移除本组件。[握手](../../../raw/current/scripts/components/channelcastable.lua#L29-L62)
- 修改边界：原打火机根据 willow_attuned_lighter 动态安装／移除组件并配置效果；不为一次查询强制给所有打火机 AddComponent，或覆盖原技能条件。[宿主](../../../raw/current/scripts/prefabs/lighter.lua#L113-L146)

## 关键接口

| 接口／字段 | 条件、作用与证据 |
|---|---|
| `SetOnStartChannelingFn(fn)`／`SetOnStopChannelingFn(fn)` | 均 fn(item,user)，返回值不作为握手成功判定；原打火机启动吸火任务／声音／FX，停止取消任务及 FX。[调用](../../../raw/current/scripts/components/channelcastable.lua#L35-L62) [原效果](../../../raw/current/scripts/prefabs/lighter.lua#L65-L93) |
| `IsUserChanneling(user)`／`IsAnyUserChanneling()` | 前者比较引用，传 nil 不保证返回布尔；后者只看 self.user 非 nil，不核验人物当前 SG 或双向一致性。[查询](../../../raw/current/scripts/components/channelcastable.lua#L21-L27) |
| `StopChanneling()` | 有 user 时调用 user.components.channelcaster:StopChanneling，依赖对端存在；不手工清 user 绕过玩家减速／事件撤销。[转交](../../../raw/current/scripts/components/channelcastable.lua#L64-L71) [人物停止](../../../raw/current/scripts/components/channelcaster.lua#L75-L102) |
| `SetStrafing(enable)` | 默认 true；人物原回调读取 false 时跳过侧移适配，但人物组件的基础 CHANNELCAST_SPEED_MOD 仍会设置，不能称为“完全取消减速”。[默认](../../../raw/current/scripts/components/channelcastable.lua#L1-L11) [消费者](../../../raw/current/scripts/prefabs/player_common.lua#L300-L325) [基础倍率](../../../raw/current/scripts/components/channelcaster.lua#L58-L64) |

活跃时卸装先走完整停止再 RemoveComponent；移除组件／实体也映射到 StopChanneling。没有本组件存读，原技能重配负责恢复能力；不要持久保存一个旧组件引用来跨越动态重建。[生命周期](../../../raw/current/scripts/components/channelcastable.lua#L29-L32) [移除](../../../raw/current/scripts/components/channelcastable.lua#L64-L71)

## 典型接入

### 为原打火机提供服务端取消当前吸火的事件

目标模组 `modmain.lua`，无新资源。服务端对物品发 `lighter:PushEvent("my_mod_stop_lighter_channel")`；只交付接收端，沿用原技能／装备获得能力，不提供新技能、RPC 或权限系统。

```lua
AddPrefabPostInit("lighter", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    inst:ListenForEvent("my_mod_stop_lighter_channel", function(item)
        local castable = item.components.channelcastable
        if castable ~= nil then
            castable:StopChanneling()
        end
    end)
end)
```

事件发生时取当前组件，避免捕获构造时尚不存在／后来已移除的组件；保留物品原停止回调与人物撤销链。组件不存在／没有 user 时不处理，不禁止以后重新吸火，也不改变耐久或技能。停用重载移除接收端；自定义组件若破坏对端引用，不由本例修复。[原动态配置](../../../raw/current/scripts/prefabs/lighter.lua#L113-L146) [停止链](../../../raw/current/scripts/components/channelcaster.lua#L75-L98)

## 常见陷阱与限制

- **直接调 OnStartChanneling 就完成使用者交接**：它只做物品端握手，换 user 前没有自动清旧人物组件；正常动作先检查物品未被占用，再调人物入口。（26148036 核验：R-01 START／STOP_CHANNELCAST 定义加 `invalid_hold_action=true`，不可长按连发，消费点 playercontroller.lua:4623）[动作](../../../raw/current/scripts/actions.lua#L4316-L4329) [禁长按](../../../raw/current/scripts/actions.lua#L488-L489)
- **删物品 user 就能干净取消**：会绕开原效果、移动来源、newstate 监听与人物停止事件；用公开 StopChanneling。[停止](../../../raw/current/scripts/components/channelcaster.lua#L75-L98)
- **初始没有组件说明这个物品不支持持续施法**：原打火机按技能动态安装，活跃卸装还会移除；按实际条件取当前实例。[宿主](../../../raw/current/scripts/prefabs/lighter.lua#L113-L146)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 中变更文件 actions.lua、componentactions.lua、prefabs/player_common.lua（channelcastable.lua／channelcaster.lua／lighter.lua 本版未变）对照 build-26148036 更新报告核验，条目 R-01：START_CHANNELCAST／STOP_CHANNELCAST 加 `invalid_hold_action=true` 禁长按连发；componentactions 的 EQUIPPED.channelcastable 处理器无变化，player_common 变更与 channelcast 无关；锚点重定位 1 处、结论修订 1 处；未覆盖：运行验证。
- 源码审阅：2026-10-20，Steam buildid 25643519；覆盖完整物品／人物组件、原打火机动态配置和效果回调、人物侧移适配、开始／停止动作与候选片段。未全面审校吸火目标选择、技能刷新全流程、所有 SG／预测或引擎同步时序。
- 结构检查：本批格式、来源、链接／行号、大小与登记通过；完整取消事件例通过已有 Full Moon Lua 5.1 syntax-only。
- 游戏运行：未运行；技能取得／失去、动态重建、吸火取消／卸装、FX／移动恢复和其他模组交接待实测。
