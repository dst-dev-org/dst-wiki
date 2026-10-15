---
title: "locomotor"
created: "2026-10-12"
updated: "2026-10-28"
type: component
summary: "配置来源化移动倍率、目的地与动作衔接，区分停步、取消动作和玩家客户端预测。"
aliases: ["生物移动加速", "移动倍率", "走到目标", "跟随移动", "取消移动动作", "locomotor"]
tags: [component, creature, mechanism, lifecycle, server-client, network-sync]
dst_build: "steam-buildid:26230496"
sources: ["components/locomotor.lua", "prefabs/pigman.lua", "prefabs/player_common.lua", "stategraphs/SGpig.lua", "stategraphs/commonstates.lua", "behaviours/follow.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# locomotor

## 快速判断

- 职责与适用：移动速度、目的地／寻路与动作衔接；选择行为由 Brain／控制器负责，动画／状态切换由 SG 配合，归属用 [[reference/components/follower]]。[locomotor.lua:982](../../../raw/current/scripts/components/locomotor.lua#L982 [commonstates.lua:388](../../../raw/current/scripts/stategraphs/commonstates.lua#L388
- 优先使用：独立效果用 SetExternalSpeedMultiplier／配对移除；装备才生效的属性优先 [[reference/components/equippable#关键接口]]，不重复维护穿脱倍率。[locomotor.lua:490](../../../raw/current/scripts/components/locomotor.lua#L490 [locomotor.lua:178](../../../raw/current/scripts/components/locomotor.lua#L178
- 重要副作用：来源倍率相乘，同 source／key 替换，source 移除自动撤销；GoTo 可能替换旧 BufferedAction，目的地到达先通知再提交动作。[locomotor.lua:480](../../../raw/current/scripts/components/locomotor.lua#L480 [locomotor.lua:1078](../../../raw/current/scripts/components/locomotor.lua#L1078 [locomotor.lua:1494](../../../raw/current/scripts/components/locomotor.lua#L1494
- 修改边界：本例服务端权威配置；locomotor 在玩家客户端也可存在并用于预测，不能套“所有 components 都只在服务端”。玩家模式／组件重建另核验。[locomotor.lua:266](../../../raw/current/scripts/components/locomotor.lua#L266 [player_common.lua:891](../../../raw/current/scripts/prefabs/player_common.lua#L891

## 关键接口

| 接口／字段 | 参数或返回 | 条件、作用与证据 |
|---|---|---|
| `walkspeed`／`runspeed`／`GetWalkSpeed()`／`GetRunSpeed()` | 基础数值／结算数值 | 结算含外部、装备或骑乘、地皮与 throttle；直接改基速会被目标模式切换覆盖。[locomotor.lua:91](../../../raw/current/scripts/components/locomotor.lua#L91 [locomotor.lua:178](../../../raw/current/scripts/components/locomotor.lua#L178 [locomotor.lua:589](../../../raw/current/scripts/components/locomotor.lua#L589 [pigman.lua:321](../../../raw/current/scripts/prefabs/pigman.lua#L321 |
| `SetExternalSpeedMultiplier(source, key, mult)` | source 为实体；唯一 key；数值倍率 | 同来源键替换，各来源／键相乘；nil key 忽略，nil／1 倍率移除；source 要支持事件监听。[locomotor.lua:480](../../../raw/current/scripts/components/locomotor.lua#L480 [locomotor.lua:490](../../../raw/current/scripts/components/locomotor.lua#L490 |
| `RemoveExternalSpeedMultiplier(source, key?)` | 原实体与键 | 有 key 只移除该项，省略移整个来源；GetExternalSpeedMultiplier 无项返回 1。[locomotor.lua:515](../../../raw/current/scripts/components/locomotor.lua#L515 [locomotor.lua:534](../../../raw/current/scripts/components/locomotor.lua#L534 |
| `GoToEntity(target, action?, run?)`／`GoToPoint(pt, action?, run?, overridedest?)` | 实体／位置、可选 BufferedAction、跑步标志 | 设目的地与移动意图，通常 FindPath，directdrive 分支直接驱动；不保证立即到达或路径可达。[locomotor.lua:982](../../../raw/current/scripts/components/locomotor.lua#L982 [locomotor.lua:1040](../../../raw/current/scripts/components/locomotor.lua#L1040 |
| `SetReachDestinationCallback(fn)` | `(inst)` 单槽 | 到达时与 onreachdestination 通知发生于提交 BufferedAction 之前；不代表动作成功。[locomotor.lua:778](../../../raw/current/scripts/components/locomotor.lua#L778 [locomotor.lua:1498](../../../raw/current/scripts/components/locomotor.lua#L1498 |
| `Stop()`／`Clear()` | 无参常用途径 | Stop 停移动／路径但不清 BufferedAction；Clear 清目的地／回调／意图并清动作，但不自行 ResetPath。需取消整项意图时按当前行为用 Stop 再 Clear。[locomotor.lua:755](../../../raw/current/scripts/components/locomotor.lua#L755 [locomotor.lua:1091](../../../raw/current/scripts/components/locomotor.lua#L1091 |

### 移动与动作边界

原猪人先构造 locomotor，再设 SGpig／Brain；SG 的 locomote handler 根据移动／跑步意图与当前状态切换 walk／run。Follow 行为更新目标位置后发 GoToPoint，已有高优先级行为可覆盖意图；只调 GoToPoint 不补全新生物的 Physics、SG 或行为树。[pigman.lua:655](../../../raw/current/scripts/prefabs/pigman.lua#L655 [pigman.lua:308](../../../raw/current/scripts/prefabs/pigman.lua#L308 [SGpig.lua:20](../../../raw/current/scripts/stategraphs/SGpig.lua#L20 [commonstates.lua:388](../../../raw/current/scripts/stategraphs/commonstates.lua#L388 [follow.lua:143](../../../raw/current/scripts/behaviours/follow.lua#L143

到达目标时发 `onreachdestination {target, pos}` 并调用 atdestfn，再把 BufferedAction 交宿主执行／预览，最后 Stop／Clear。要确认采收、攻击等业务成功，必须查实际动作结果，不能把到达通知当完成。[locomotor.lua:1494](../../../raw/current/scripts/components/locomotor.lua#L1494

## 典型接入

### 猪人移动乘一点二并保留原版行为

目标模组 `modmain.lua`；沿用 pigman 资源、Physics、Brain 和 SG。不覆盖 SetNormalPig 设置的 walk／run 基速，改独立来源倍率。[pigman.lua:308](../../../raw/current/scripts/prefabs/pigman.lua#L308 [pigman.lua:655](../../../raw/current/scripts/prefabs/pigman.lua#L655 [pigman.lua:858](../../../raw/current/scripts/prefabs/pigman.lua#L858 [modutil.lua:593](../../../raw/current/scripts/modutil.lua#L593

```lua
AddPrefabPostInit("pigman", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local locomotor = inst.components.locomotor
    if locomotor ~= nil then
        locomotor:SetExternalSpeedMultiplier(inst, "my_mod_pig_speed", 1.2)
    end
end)
```

source 使用当前猪人，key 改为目标模组唯一命名。1.2 是倍率，基础模式变化后仍乘此项；实际速度还受地皮／其他倍率，不改变跟随距离、攻击频率或行为优先级。只想招募后加速，应在归属变化中配对添加／移除，不能直接套成全猪人效果。[locomotor.lua:178](../../../raw/current/scripts/components/locomotor.lua#L178 [locomotor.lua:490](../../../raw/current/scripts/components/locomotor.lua#L490 [[reference/components/follower#关键接口]]

本例不自建存档，初始化重施；停用后新初始化不再添加。当前撤销用 `RemoveExternalSpeedMultiplier(inst, "my_mod_pig_speed")`；source 删除自动撤销，组件移除亦解除来源监听。重新创建组件时须由所属玩法重施，不能假设旧来源表自动保留。[locomotor.lua:501](../../../raw/current/scripts/components/locomotor.lua#L501 [locomotor.lua:515](../../../raw/current/scripts/components/locomotor.lua#L515 [locomotor.lua:387](../../../raw/current/scripts/components/locomotor.lua#L387

## 常见陷阱与限制

- **使用字符串 source 或重复造 key**：此机制监听 source 的 onremove，传实体；同键更新不会叠加，不同键会相乘，临时效果用稳定来源／键并配对移除。[locomotor.lua:490](../../../raw/current/scripts/components/locomotor.lua#L490
- **Stop 等于取消待执行动作**：Stop 保留 bufferedaction，Clear 才 SetBufferedAction(nil) 并使旧动作 Fail；先判断当前行为拥有哪个动作，不随意取消别的系统。[locomotor.lua:755](../../../raw/current/scripts/components/locomotor.lua#L755 [locomotor.lua:1078](../../../raw/current/scripts/components/locomotor.lua#L1078 [locomotor.lua:1099](../../../raw/current/scripts/components/locomotor.lua#L1099
- **服务端加速直接推广为玩家预测已正确**：玩家通过 classified／客户端分支获取速度，predict multiplier 是另一入口；本页猪人案例不证明骑乘、幽灵或玩家预测／重建正确。[locomotor.lua:67](../../../raw/current/scripts/components/locomotor.lua#L67 [locomotor.lua:207](../../../raw/current/scripts/components/locomotor.lua#L207 [locomotor.lua:548](../../../raw/current/scripts/components/locomotor.lua#L548

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 处理范围：锚点重定位 1 处（pigman#L652→L654，显示标签同步）；结论修订：无。locomotor.lua 三处 1:1 改动（PreviewAction:845／PushAction:946／OnUpdate:1610+插入:1614）均不在本页速度倍率／GoTo／Stop 引用区间，21 处引用内容两版一致。
- 依据：变更清单命中 components/locomotor.lua、prefabs/pigman.lua、prefabs/player_common.lua、stategraphs/commonstates.lua。

- 源码审阅：2026-10-12，Steam buildid 25643519；精选基速消费、来源倍率／撤销、目的地、停步／清动作与到达顺序，猪人 Brain／SG 协作及发布案例。未全量审校本组件、路径算法、跳船、玩家预测／骑乘／幽灵重建与网络时序。
- 结构检查：元数据、证据行、链接／章节、大小、登记通过；发布 Lua 与猪人协作组合经 Full Moon 3.0.0／Lua 5.1 语法及原字节往返检查。
- 游戏运行：未运行；猪人模式速度、跟随／寻路、到达动作、组件重建和多人／其他模组倍率待实测。

相关：[[reference/components/follower#生命周期与实际跟随]]；[[reference/components/equippable#典型接入|手杖装备加速]]；[[reference/components/debuff#三十秒加速效果并成对撤销|限时来源加速]]。

## 相关链接

- [[reference/components/platformhopdelay]]：船平台起跳延迟 ticks，取双侧最大值。
