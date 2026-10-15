---
title: "tributable"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "贡品积累器（未接线）：累积贡品值、达标后可领奖，带衰减任务与存档；当前版本无任何挂载者，蚁狮走的是自己的同名词事件。"
aliases: ["tributable", "贡品积累", "献贡组件"]
tags: [component]
size_class: simple
dst_build: "steam-buildid:26230496"
sources: ["components/tributable.lua", "prefabs/antlion.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# tributable

## 快速判断

- 职责与适用：在实体上维护 `currenttributevalue` 贡品值：`OnAccept` 累加并启动可选衰减（`decaycurrenttributetime` 秒后清零），`HasPendingReward` 判断是否达到 `rewardattributevalue`（默认 10），`OnGivenReward` 清零计数。[接受贡品](../../../raw/current/scripts/components/tributable.lua#L40-L52)
- 优先使用：**当前快照内无任何消费者**——全库 grep 仅组件自身引用，无 `AddComponent("tributable")`、无 `OnAccept`/`OnRefuse` 调用方；不能作为可靠接入点。
- 重要副作用：`OnAccept`/`OnRefuse` 推实体事件 `onaccepttribute`/`onrefusetribute`；同名事件在蚁狮（antlion）上另有一套独立实现（`antlion.lua:154,163` 直接由其交易逻辑推送），与本组件无关。[蚁狮同名事件](../../../raw/current/scripts/prefabs/antlion.lua#L154)
- 修改边界：有存读钩子（值、剩余衰减时间、已发奖次数）；组件构造不挂任何标签，可见性需宿主自理；`tributer` 参数接收但未使用。[存档](../../../raw/current/scripts/components/tributable.lua#L58-L65)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `OnAccept(value, tributer)` | value: number | 累加贡品值、推 `onaccepttribute`；`decaycurrenttributetime > 0` 且未达标时启动 `DoTaskInTime` 衰减任务，到期清零。[实现](../../../raw/current/scripts/components/tributable.lua#L40-L52) |
| `OnRefuse()` | `-> nil` | 仅推 `onrefusetribute`，不改状态。[实现](../../../raw/current/scripts/components/tributable.lua#L54-L56) |
| `OnGivenReward()` | `-> nil` | 清零贡品值、`numrewardsgiven` 加一、取消衰减任务、回调 `ongivenrewardfn(inst)`。[实现](../../../raw/current/scripts/components/tributable.lua#L26-L38) |
| `HasPendingReward()` | `-> boolean` | `currenttributevalue >= rewardattributevalue`（默认 10）。[实现](../../../raw/current/scripts/components/tributable.lua#L17-L19) |
| `decaycurrenttributetime` | number（默认 0） | 衰减延迟秒数；0 表示贡品永久保留。[使用点](../../../raw/current/scripts/components/tributable.lua#L48-L51) |
| `onaccepttribute` / `onrefusetribute` | 实体事件 | 本组件推送的进度通知；若宿主是蚁狮类实体会与其同名事件混淆，监听方需区分来源。[推送点](../../../raw/current/scripts/components/tributable.lua#L42) |

## 典型接入

### 设计意图还原（未接线，勿当现成能力）

- 前提与选择：设计意图是"持续上贡换奖励"型 NPC 交互：交易入口每收一份贡品调 `OnAccept(value)`，外部按 `HasPendingReward()` 发奖后调 `OnGivenReward()`；入口接线（谁调 OnAccept）在脚本库中不存在，需模组自行搭建（如挂到 trader 的收货流程）。[蚁狮对照](../../../raw/current/scripts/prefabs/antlion.lua#L154-L164)
- 生效与撤销：贡品值与衰减随实体存档持久化（OnLoad 用剩余时间重建衰减任务）；移除组件不退款。

```lua
-- 设计意图复刻（当前原版未接线；自行验证）
inst:AddComponent("tributable")
inst.components.tributable.rewardattributevalue = 10
inst.components.tributable.decaycurrenttributetime = 480
inst.components.tributable.ongivenrewardfn = function(inst)
    inst.components.lootdropper:SpawnLootPrefab("trinket_gift_1")
end
-- 收贡入口（自建）：inst.components.tributable:OnAccept(value, doer)
-- 发奖后：inst.components.tributable:OnGivenReward()
```

- 调整与验证：衰减在"未达标"时才启动，达标后衰减任务保持取消态；已达标又想引入衰减需自行再调 OnAccept 类逻辑。

## 常见陷阱与限制

- **与蚁狮机制混淆：** 蚁狮收贡（`SGantlion.lua:113,122` 监听同名事件）是 prefab 内直写的独立实现，从未挂本组件；在蚁狮生态里 AddComponent 本组件不会改变其行为，还可能因事件重名造成误监听。[蚁狮独立实现](../../../raw/current/scripts/prefabs/antlion.lua#L154)
- **OnAccept 不校验 value：** 负数/0 会照常累加（负数可倒扣贡品值）；入口侧需自行约束数值范围。[无守卫](../../../raw/current/scripts/components/tributable.lua#L41-L42)
- **无 OnRemoveFromEntity 清理：** 组件被移除时衰减任务不会被取消（任务挂在实体上，实体存续则任务仍会触发，回调里再访问组件会报错）；复用时注意移除时机。[衰减回调](../../../raw/current/scripts/components/tributable.lua#L21-L24)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（84 行）；全库 grep（大小写不敏感）确认 `tributable` 无挂载者与读者；核对蚁狮同名词事件的独立实现（`antlion.lua:154,163`、`SGantlion.lua:113-122`）以划清边界。
- 结构检查：2026-10-28 validate_page.py 通过；validate_source_links.py --strict 锚点通过（见批级记录）。
- 游戏运行：未运行。
