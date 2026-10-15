---
title: "channelcaster"
created: "2026-10-20"
updated: "2026-10-28"
type: component
summary: "人物持续施法的物品握手、状态同步、移动来源与开始停止事件；支持无物品模式，保留原人物撤销回调。"
aliases: ["持续施法时每分钟增加三点精神贡献", "人物持续施法状态", "channelcaster"]
tags: [component, event, callback, lifecycle, network-sync, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/channelcaster.lua", "components/channelcastable.lua", "prefabs/player_common.lua", "actions.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# channelcaster

## 快速判断

- 职责与适用：人物侧管理 item／channeling、移动来源、newstate 监听与转换事件；物品效果见 [[reference/components/channelcastable]]，建筑持续交互见 [[reference/components/channelable]]。[定义](../../../raw/current/scripts/components/channelcaster.lua#L1-L24) [原玩家配置](../../../raw/current/scripts/prefabs/player_common.lua#L2855-L2857)
- 优先使用：StartChanneling(item?)／StopChanneling() 执行完整转换，IsChanneling 查服务端状态；扩展派生效果优先听 startchannelcast／stopchannelcast，保留原回调。[开始](../../../raw/current/scripts/components/channelcaster.lua#L48-L73) [停止](../../../raw/current/scripts/components/channelcaster.lua#L75-L98)
- 重要副作用：先停止旧流程再验新物品，成功时握手、加 CHANNELCAST_SPEED_MOD、监听 newstate、置状态、回调、发事件；新物品无效可导致旧流程已取消，没有自动回滚。[开始顺序](../../../raw/current/scripts/components/channelcaster.lua#L48-L73)
- 修改边界：允许 item=nil 的无物品模式，channeling=true 不意味着持有物品；只读字段同步到既有 player_classified，不是完整独立 replica 或技能授权系统。[属性](../../../raw/current/scripts/components/channelcaster.lua#L1-L11) [无物品动作](../../../raw/current/scripts/actions.lua#L4316-L4329)

## 关键接口

| 接口／事件 | 条件、作用与证据 |
|---|---|
| `StartChanneling(item?)` | 相同物品且已 channeling 时不重启，通常返回 nil；item=nil 可启动，物品需 channelcastable 且有效。方法不检查装备槽、拥有权或技能，正常动作还有未占用／拟态检查。（26148036 核验：R-01 START／STOP_CHANNELCAST 定义加 `invalid_hold_action=true`，不可长按连发，消费点 playercontroller.lua:4623）[方法](../../../raw/current/scripts/components/channelcaster.lua#L48-L73) [动作](../../../raw/current/scripts/actions.lua#L4316-L4329) [禁长按](../../../raw/current/scripts/actions.lua#L488-L489) |
| `IsChanneling()`／`IsChannelingItem(item)` | 前者返回 channeling，后者只是非 nil 物品的引用比较，不先判断 channeling；不要据它给任意对象增加权限。[查询](../../../raw/current/scripts/components/channelcaster.lua#L34-L40) |
| 两个回调 setter | fn(player,item)，返回值不作为是否开始／结束的判定。原人物开始回调调整其他减速与侧移，停止回调恢复它们；不能覆盖成只含自定义特效的函数。[调用](../../../raw/current/scripts/components/channelcaster.lua#L64-L70) [原适配](../../../raw/current/scripts/prefabs/player_common.lua#L300-L325) |
| `startchannelcast`／`stopchannelcast` | 发在玩家自身，data.item 可为 nil；事件在对应人物回调之后，停止前已去物品引用、移动来源、监听和 channeling。适合维护条件效果的独立来源。[事件顺序](../../../raw/current/scripts/components/channelcaster.lua#L64-L97) |
| `newstate` | 新 SG 状态没有 idle／running／keepchannelcasting 任一标签时自动 StopChanneling；不是按“是否播放施法动画”判断。[监听](../../../raw/current/scripts/components/channelcaster.lua#L42-L46) |
| `StopChanneling()`／移除 | 先通知物品停止，再去移动来源与监听、清状态，调人物停止回调并发事件；正在运行才返回 true，否则 nil。移除组件／实体映射到它，无存读钩子。[撤销](../../../raw/current/scripts/components/channelcaster.lua#L75-L102) |

共用的 IsChannelCasting／IsChannelCastingItem 从 player_classified 读取；前者代表持续状态，后者代表是否有 item，不是两个互斥技能。strafing=false 的物品只使原人物开始回调提前返回，基础移动倍率仍来自组件本身。[客户端可读方法](../../../raw/current/scripts/prefabs/player_common.lua#L286-L292) [移动逻辑](../../../raw/current/scripts/components/channelcaster.lua#L58-L64)

## 典型接入

### 持续施法期间增加每分钟三点精神贡献，停止撤销

目标模组 `modmain.lua`，无新资源；作用于已有 channelcaster／sanity 的玩家，包括原流程允许的无物品模式，不产生新技能或自动开始施法。精神接口、点／秒单位和最终倍率复用 [[reference/components/sanity#给人物增加每分钟三点精神贡献]]。

```lua
AddPlayerPostInit(function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local function Refresh(player)
        local sanity = player.components.sanity
        if sanity == nil then return end
        local caster = player.components.channelcaster
        if caster ~= nil and caster:IsChanneling() then
            sanity.externalmodifiers:SetModifier(player, 3 / 60, "my_mod_channelcast")
        else
            sanity.externalmodifiers:RemoveModifier(player, "my_mod_channelcast")
        end
    end
    inst:ListenForEvent("startchannelcast", Refresh)
    inst:ListenForEvent("stopchannelcast", Refresh)
    Refresh(inst)
end)
```

一个来源／key 更新不累积，3/60 是额外 0.05 点／秒，仍参与最终精神速率，不保证净每分钟上涨三点。完整停止事件到来时撤销，物品更换会先停再开始；不轮询、不替换原人物或物品回调。状态／modifier 不自行存档；停用并重载不再注册效果，动态卸载前先移除同一 key。其他模组直接改字段不发事件，本例不修复其状态。

## 常见陷阱与限制

- **所有开始失败都保留原施法**：旧流程在新物品验证前已经停止；需要保留原状态的调用方先核验自身候选，不把方法当原子替换。[顺序](../../../raw/current/scripts/components/channelcaster.lua#L48-L56)
- **只有有物品时才能 channeling**：nil 模式是实现和动作的显式分支，监听事件时不能无条件读取 data.item.prefab。[无物品](../../../raw/current/scripts/actions.lua#L4316-L4326)
- **直接 channeling=false 就撤销完了**：物品效果、速度来源、原侧移修正和事件都可能遗留；走 StopChanneling。[清理](../../../raw/current/scripts/components/channelcaster.lua#L75-L98)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 中变更文件 actions.lua、prefabs/player_common.lua（channelcaster.lua／channelcastable.lua 本版未变）对照 build-26148036 更新报告核验，条目 R-01：START_CHANNELCAST／STOP_CHANNELCAST 加 `invalid_hold_action=true` 禁长按连发；player_common 变更（OnWormDigestionSound 移除）与 channelcast 无关，仅致本页配置锚点前移；锚点重定位 4 处、结论修订 1 处；未覆盖：运行验证。
- 源码审阅：2026-10-20，Steam buildid 25643519；覆盖完整人物／物品组件、事件／newstate、原玩家配置／移动适配与动作；精神贡献复用同 build 已核验 sanity 页。未全面审校无物品技能效果、所有 SG／预测、classified 重建或其他减速内部。
- 结构检查：本批格式、来源、链接／行号、大小与登记通过；完整精神贡献例通过已有 Full Moon Lua 5.1 syntax-only。
- 游戏运行：未运行；开始／打断／换物品、nil 模式、减速恢复、精神速率与多模组直接字段改动待实测。
