---
title: "follower"
created: "2026-10-12"
updated: "2026-10-28"
type: component
summary: "设置随从主人和忠诚期限，区分归属、战斗目标共享、实际跟随及关系清理。"
aliases: ["随从归属", "忠诚时间", "跟随主人", "不共享战斗目标", "follower"]
tags: [component, creature, mechanism, lifecycle, network-sync]
dst_build: "steam-buildid:26230496"
sources: ["components/follower.lua", "components/leader.lua", "prefabs/pigman.lua", "brains/pigbrain.lua", "behaviours/follow.lua", "entityscript.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# follower

## 快速判断

- 职责与适用：随从端归属、忠诚倒计时与物品主人映射；主人集合见 [[reference/components/leader]]。实际跟随与证据见[生命周期与实际跟随](#生命周期与实际跟随)。
- 优先使用：`SetLeader(ent)`／`SetLeader(nil)` 建立／解除；有期限先设 maxfollowtime 再 AddLoyaltyTime；CancelLoyaltyTask 只撤计时。见[关键接口与证据](#关键接口)。
- 重要副作用：SetLeader 清旧登记／计时／监听，更新 replica，可建新关系，最后发 leaderchanged；直接写 leader 会绕过。见[SetLeader](#关键接口)。
- 修改边界：服务端关系操作；neverexpire 不等于免疫主人死亡／攻击、主动改型等全部解除入口。见[陷阱与证据](#常见陷阱与限制)。

## 关键接口

| 接口／字段 | 参数或返回 | 条件、作用与证据 |
|---|---|---|
| `SetLeader(ent?)`／`GetLeader()` | 主人实体或 nil／有效归属实体或 nil | SetLeader 清旧登记／计时／监听，更新 replica，可建新登记／牵引，发 leaderchanged。getter 返回 itemowner 或原始 leader，跟随物品时可返回持有者；IsNearLeader 仍查原始 leader。[follower.lua:9](../../../raw/current/scripts/components/follower.lua#L9 [follower.lua:59](../../../raw/current/scripts/components/follower.lua#L59 [follower.lua:293](../../../raw/current/scripts/components/follower.lua#L293 [follower.lua:412](../../../raw/current/scripts/components/follower.lua#L412 |
| `maxfollowtime`／`AddLoyaltyTime(seconds)` | 上限秒数／增量秒数 | maxfollowtime 默认 nil，按 0 封顶；先乘主人 loyaltyeffectiveness，再限制剩余到上限，向随从发 gainloyalty 并重排过期任务。[follower.lua:27](../../../raw/current/scripts/components/follower.lua#L27 [follower.lua:365](../../../raw/current/scripts/components/follower.lua#L365 |
| `CancelLoyaltyTask()`／`StopFollowing()` | 无参数 | 前者只取消 task／targettime；后者在非 neverexpire 时发 loseloyalty 并 SetLeader(nil)。强制解除直接 SetLeader(nil)。[follower.lua:391](../../../raw/current/scripts/components/follower.lua#L391 [follower.lua:399](../../../raw/current/scripts/components/follower.lua#L399 |
| `neverexpire` | 可选布尔 | AddLoyaltyTime／StopFollowing 会跳过；不取消既有计时，LongUpdate 仍可按旧 task 直接解除。[follower.lua:365](../../../raw/current/scripts/components/follower.lua#L365 [follower.lua:475](../../../raw/current/scripts/components/follower.lua#L475 |
| `canaccepttarget` | 默认 true | leader 战斗建议的门禁，不阻止自身战斗／Brain 自选目标。[follower.lua:28](../../../raw/current/scripts/components/follower.lua#L28 [leader.lua:50](../../../raw/current/scripts/components/leader.lua#L50 [leader.lua:152](../../../raw/current/scripts/components/leader.lua#L152 |
| `KeepLeaderOnAttacked()`／`LoseLeaderOnAttacked()` | 无参数 | 移除／恢复“被原始主人攻击则解约”监听；不取消生物其他受击行为。[follower.lua:1](../../../raw/current/scripts/components/follower.lua#L1 [follower.lua:461](../../../raw/current/scripts/components/follower.lua#L461 |
| `OnChangedLeader` | `(inst, new, old)` | SetLeader 的回调单槽，发生变化后调用，再做友军目标清理／leaderchanged。[follower.lua:338](../../../raw/current/scripts/components/follower.lua#L338 |

## 生命周期与实际跟随

归属与行为分工：猪人 Brain 用 GetLeader 选目标，白天 Follow 节点按距离发 locomotor:GoToPoint／方向移动；其他优先行为、战斗、睡眠与夜晚逻辑可抢占。仅 AddComponent("follower")／SetLeader 不让新生物自动走动；Brain、SG、Physics 与 [[reference/components/locomotor#移动与动作边界]] 仍须匹配。[pigbrain.lua:121](../../../raw/current/scripts/brains/pigbrain.lua#L121 [pigbrain.lua:333](../../../raw/current/scripts/brains/pigbrain.lua#L333 [follow.lua:76](../../../raw/current/scripts/behaviours/follow.lua#L76 [follow.lua:143](../../../raw/current/scripts/behaviours/follow.lua#L143

StartLeashing 的离屏靠近属于特殊补位，不是通常 Follow 移动或立即传送；距离、主人醒着、地形等均有条件。[follower.lua:75](../../../raw/current/scripts/components/follower.lua#L75 [follower.lua:150](../../../raw/current/scripts/components/follower.lua#L150

组件保存剩余秒数及玩家 userid／缓存，OnLoad 重设时间与玩家缓存；重连匹配还受新角色标记、距离／期限约束，不能承诺跨分片或所有重选都恢复。配置字段在初始化重施。[follower.lua:213](../../../raw/current/scripts/components/follower.lua#L213 [follower.lua:242](../../../raw/current/scripts/components/follower.lua#L242 [follower.lua:416](../../../raw/current/scripts/components/follower.lua#L416

实体移除由 leader 的 onremove 监听清普通登记；**主动移除 follower 组件前先 SetLeader(nil)**。OnRemoveFromEntity 清任务／监听／物品来源，但不替代普通关系解除，宿主又先把 components.follower 清空才调钩子。[leader.lua:28](../../../raw/current/scripts/components/leader.lua#L28 [leader.lua:170](../../../raw/current/scripts/components/leader.lua#L170 [follower.lua:503](../../../raw/current/scripts/components/follower.lua#L503 [entityscript.lua:670](../../../raw/current/scripts/entityscript.lua#L670

## 典型接入

### 猪人不接受主人的战斗目标建议

目标模组 `modmain.lua`；沿用 pigman 资源、肉食招募、忠诚计时与 Brain／SG，仅关闭 leader 的目标共享门禁。[pigman.lua:73](../../../raw/current/scripts/prefabs/pigman.lua#L73 [pigman.lua:308](../../../raw/current/scripts/prefabs/pigman.lua#L308 [pigman.lua:689](../../../raw/current/scripts/prefabs/pigman.lua#L689 [pigman.lua:858](../../../raw/current/scripts/prefabs/pigman.lua#L858 [modutil.lua:593](../../../raw/current/scripts/modutil.lua#L593

```lua
AddPrefabPostInit("pigman", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local follower = inst.components.follower
    if follower ~= nil then
        follower.canaccepttarget = false
    end
end)
```

作用于该入口初始化的猪人；不会清已有 combat 目标、使其和平、不反击或禁止主动攻击。也不保证夜晚始终跟随；仍使用原版行为。字段不由 follower 保存，启用时每次初始化应用，停用后恢复默认；其他模组／新模式直接改字段仍可能覆盖。[leader.lua:50](../../../raw/current/scripts/components/leader.lua#L50 [leader.lua:152](../../../raw/current/scripts/components/leader.lua#L152 [follower.lua:416](../../../raw/current/scripts/components/follower.lua#L416 [pigbrain.lua:358](../../../raw/current/scripts/brains/pigbrain.lua#L358

## 常见陷阱与限制

- **只 AddLoyaltyTime、不设上限或主人**：它不招募，nil 上限按 0，见[关键接口](#关键接口)。原猪人喂食还会更新 maxfollowtime，PostInit 单次赋值不能保证后续保留。[pigman.lua:99](../../../raw/current/scripts/prefabs/pigman.lua#L99
- **neverexpire 使所有路径都无法解约**：设置永久时应取消旧忠诚任务；强制解除仍用 SetLeader(nil)，猪人改型、主人移除／攻击等仍可解除，须分别审阅。[follower.lua:391](../../../raw/current/scripts/components/follower.lua#L391 [follower.lua:475](../../../raw/current/scripts/components/follower.lua#L475 [pigman.lua:448](../../../raw/current/scripts/prefabs/pigman.lua#L448 [pigman.lua:537](../../../raw/current/scripts/prefabs/pigman.lua#L537
- **直接写 leader 或直接移组件**：可能残留旧名单／计时，或令死亡回调访问缺失组件。用[SetLeader](#关键接口)；移组件前解除，宿主顺序与证据见[生命周期与实际跟随](#生命周期与实际跟随)。不要手改主人 followers。

## 验证记录

- 版本增量复核 26148036（2026-10-25）：锚点重定位 0 处——pigman.lua 本版改动（DIET 局部化 +2、canwearhat +1、SetDiet 合并 -2）均在引用锚点（72/98/307/447/536/688/859）之外，逐点核对内容未变；follower.lua、leader.lua、pigbrain.lua、follow.lua、modutil.lua 未变。依据：全局枢纽 diff 审阅（R-08 距离函数改动不涉 SetLeader 路径）＋变更清单命中 prefabs/pigman.lua、entityscript.lua（648 行引用未变）。
- 源码审阅：2026-10-12，Steam buildid 25643519；覆盖归属／物品映射入口、忠诚与永久边界、存档缓存、清理，猪人招募／Brain 跟随及目标共享案例。全部物品来源工具、跨分片、完整宠物／角色重选协议未覆盖。
- 结构检查：元数据、证据行、链接／章节、大小、登记通过；发布 Lua 与猪人协作组合经 Full Moon 3.0.0／Lua 5.1 语法及原字节往返检查。
- 游戏运行：未运行；跟随／夜晚行为、目标共享／反击、离屏、解除／读档／重连与模组覆盖待实测。

相关：[[reference/components/leader#关键接口]]；[[reference/components/locomotor#移动与动作边界]]；[[reference/components/trader#关键接口]]。
