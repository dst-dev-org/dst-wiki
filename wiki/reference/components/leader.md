---
title: "leader"
created: "2026-10-12"
updated: "2026-10-28"
type: component
summary: "管理主人的随从登记、战斗目标建议与忠诚增量倍率，区分普通随从和物品关联随从。"
aliases: ["招募随从", "喂食忠诚收益", "随从数量", "主人战斗目标", "leader"]
tags: [component, creature, mechanism, callback, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/leader.lua", "components/follower.lua", "prefabs/player_common.lua", "prefabs/pigman.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# leader

## 快速判断

- 职责与适用：主人端的随从集合与协作通知；随从端归属／忠诚时间在 [[reference/components/follower]]，移动另需 Brain／[[reference/components/locomotor]]。[leader.lua:15](../../../raw/current/scripts/components/leader.lua#L15 [follower.lua:293](../../../raw/current/scripts/components/follower.lua#L293
- 优先使用：已具 follower 的实体用 `AddFollower`；增加未来喂食忠诚收益可配置 `loyaltyeffectiveness`，不直接延长现有倒计时。[leader.lua:189](../../../raw/current/scripts/components/leader.lua#L189 [follower.lua:365](../../../raw/current/scripts/components/follower.lua#L365
- 重要副作用：登记会 SetLeader、向随从发 startfollowing、调主人回调并监听随从死亡／移除；主人受击／新目标可向随从建议战斗目标。[leader.lua:189](../../../raw/current/scripts/components/leader.lua#L189 [leader.lua:50](../../../raw/current/scripts/components/leader.lua#L50 [leader.lua:152](../../../raw/current/scripts/components/leader.lua#L152
- 修改边界：服务端关系管理；不替代招募资格检查、Brain、忠诚上限或跟随距离；人物原有回调应保留。[pigman.lua:93](../../../raw/current/scripts/prefabs/pigman.lua#L93 [player_common.lua:2799](../../../raw/current/scripts/prefabs/player_common.lua#L2799

## 关键接口

| 接口／字段 | 参数或返回 | 条件、作用与证据 |
|---|---|---|
| `AddFollower(ent)` | 有 follower 的实体；无显式返回 | 普通集合已登记则不重复添加；双向登记及事件／回调，不自动 AddLoyaltyTime。[leader.lua:189](../../../raw/current/scripts/components/leader.lua#L189 |
| `RemoveFollower(ent)`／`RemoveAllFollowers()` | 单实体／无参数 | 清普通集合／监听，onremovefollower → 随从 stopfollowing → SetLeader(nil)；All 只遍历普通集合。[leader.lua:170](../../../raw/current/scripts/components/leader.lua#L170 [leader.lua:240](../../../raw/current/scripts/components/leader.lua#L240 |
| `GetNumFollowers()`／`CountFollowers(tag?)` | 数量 | 无 tag 只普通随从；有 tag 时遍历普通与 itemfollowers。两集合语义不同，不能混用统计口径。[leader.lua:65](../../../raw/current/scripts/components/leader.lua#L65 |
| `IsFollower(ent)`／`GetFollowersByTag(tag)` | boolean／实体数组 | 包含两个集合；GetFollowersByTag(nil) 返回空表。[leader.lua:46](../../../raw/current/scripts/components/leader.lua#L46 [leader.lua:88](../../../raw/current/scripts/components/leader.lua#L88 |
| `onfolloweradded`／`onremovefollower` | `(主人inst, 随从ent)` | 单槽替换；人物已有回调，不为追加效果直接覆盖。[leader.lua:178](../../../raw/current/scripts/components/leader.lua#L178 [leader.lua:197](../../../raw/current/scripts/components/leader.lua#L197 [player_common.lua:2800](../../../raw/current/scripts/prefabs/player_common.lua#L2800 |
| `loyaltyeffectiveness` | 可选数值倍率，未赋默认值 | follower:AddLoyaltyTime 先乘此值，再按随从 maxfollowtime 封顶；不是忠诚上限或时间流速。[follower.lua:370](../../../raw/current/scripts/components/follower.lua#L370 |
| `AddItemFollower(ent)`／`RemoveItemFollower(ent)` | 实体 | 仅维护临时集合／数量，不 SetLeader；由物品归属协作使用，不能当普通招募捷径。[leader.lua:218](../../../raw/current/scripts/components/leader.lua#L218 [follower.lua:279](../../../raw/current/scripts/components/follower.lua#L279 |

### 关系解除与恢复

主人 death 对普通随从逐一解除，keepdeadleader 的条目例外；组件移除会解除普通关系及自身监听，实体移除亦 RemoveAllFollowers。物品关联随从的归属／清理走 follower 的物品来源路径，不由 All 自动遍历。[leader.lua:32](../../../raw/current/scripts/components/leader.lua#L32 [leader.lua:246](../../../raw/current/scripts/components/leader.lua#L246 [leader.lua:306](../../../raw/current/scripts/components/leader.lua#L306 [follower.lua:293](../../../raw/current/scripts/components/follower.lua#L293

非玩家 leader 保存普通随从 GUID，LoadPostPass 重登记；**玩家 leader 不保存这份名单**，玩家恢复由 follower 缓存用户身份协作。loyaltyeffectiveness／itemfollowers 不在 leader 保存数据中。[leader.lua:276](../../../raw/current/scripts/components/leader.lua#L276 [[reference/components/follower#生命周期与实际跟随]]

## 典型接入

### 人物未来喂食忠诚收益乘二

目标模组 `modmain.lua`，所有人物的已有 leader 增量倍率设 2；不新增组件、回调或资源。AddPlayerPostInit 及原人物配置见 [modutil.lua:586](../../../raw/current/scripts/modutil.lua#L586)、[player_common.lua:2799](../../../raw/current/scripts/prefabs/player_common.lua#L2799)。

```lua
AddPlayerPostInit(function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local leader = inst.components.leader
    if leader ~= nil then
        leader.loyaltyeffectiveness = 2
    end
end)
```

原猪人接受符合条件的肉后 AddFollower，再 AddLoyaltyTime，后续忠诚输入先乘 2 再封顶；单次设置倍率不会立即把现有剩余翻倍，满上限时也不延长。所有读取此倍率的随从受影响，不只猪人；永不过期随从跳过 AddLoyaltyTime。停用后初始化不再设置，已增加剩余不回滚。[pigman.lua:73](../../../raw/current/scripts/prefabs/pigman.lua#L73 [follower.lua:365](../../../raw/current/scripts/components/follower.lua#L365

**此字段不限喂食**：玩家重新加入时恢复缓存剩余也调用 AddLoyaltyTime，此时可能再乘倍率。只有接受这一恢复行为时采用本例；要求严格只放大新喂食，应在指定生物的喂食输入处处理，不能靠该字段保证。[follower.lua:213](../../../raw/current/scripts/components/follower.lua#L213

这是赋值而非可叠加来源；若人物／其他模组已有倍率，应确认如何合并，不能反复乘 2。仅想影响某种随从应改它的忠诚输入路径，不借全局主人字段伪装局部效果。

## 常见陷阱与限制

- **AddFollower 成功就永远跟在主人身后**：它只建关系，不启动忠诚计时，也不提供行为树；实际移动见 follower 的协作章节。[leader.lua:189](../../../raw/current/scripts/components/leader.lua#L189 [[reference/components/follower#生命周期与实际跟随]]
- **战斗目标共享等于强制攻击**：OnAttacked／OnNewTarget 用 combat:SuggestTarget，受 canaccepttarget、小游戏／PvP 和目标判定约束；具体攻击仍由目标／Brain 决定。[leader.lua:50](../../../raw/current/scripts/components/leader.lua#L50 [leader.lua:152](../../../raw/current/scripts/components/leader.lua#L152
- **RemoveAllFollowers 清空所有口径的随从**：只处理 followers，itemfollowers 是临时物品关联集合；先确认所属路径，不手改表绕过归属协作。[leader.lua:240](../../../raw/current/scripts/components/leader.lua#L240

## 验证记录

- 增量复查：2026-10-25，Steam buildid 26148036；锚点重定位（player_common 2811/2812→2797/2798，P-83 删除段在前），leader／follower／modutil 无变化；P-80（canwearhat 标签、DIET 常量化等价）不涉招募与忠诚链，pigman 72/92 锚点未移动，结论维持。
- 源码审阅：2026-10-12，Steam buildid 25643519；覆盖双向登记／事件、数量口径、忠诚倍率消费、死亡／移除与 GUID 保存，人物／猪人案例。点名、特殊宠物系统、全部招募条件及分片恢复未覆盖。
- 结构检查：元数据、证据行、链接／章节、大小、登记通过；发布 Lua 与猪人协作组合经 Full Moon 3.0.0／Lua 5.1 语法及原字节往返检查。
- 游戏运行：未运行；实际喂食增量／上限、主人死亡、重连／读档与其他模组倍率待实测。

相关：[[reference/components/follower#关键接口]]；[[reference/components/combat#关键接口]]；[[reference/components/trader#关键接口]]。
