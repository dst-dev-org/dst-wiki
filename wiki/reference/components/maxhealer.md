---
title: "maxhealer"
created: "2026-10-17"
updated: "2026-10-28"
type: component
summary: "用强心针类物品减少生命上限惩罚，区分惩罚比例、基础生命上限与当前回血。"
aliases: ["强心针", "生命上限惩罚", "复活惩罚恢复", "上限恢复", "maxhealer"]
tags: [component, item, actions, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/maxhealer.lua", "components/health.lua", "components/health_replica.lua", "actions.lua", "componentactions.lua", "prefabs/lifeinjector.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# maxhealer

## 快速判断

- 职责与适用：挂在**物品**上，对目标 health 的 penalty 加一个增量；主要用于恢复复活造成的可用生命上限损失，不增加基础 maxhealth。[maxhealer.lua:11](../../../raw/current/scripts/components/maxhealer.lua#L11)、[health.lua:524](../../../raw/current/scripts/components/health.lua#L524
- 优先使用：`SetHealthAmount(-0.25)` 减少 25 个百分点惩罚；普通回血查 [[reference/components/healer]]，永久基础上限调整查 [[reference/components/health#关键接口]]。[maxhealer.lua:7](../../../raw/current/scripts/components/maxhealer.lua#L7
- 重要副作用：只要目标有 health 就执行 DeltaPenalty、消费一份并返回 true；惩罚已为零或 disable_penalty 开启也不自动拒绝。[maxhealer.lua:11](../../../raw/current/scripts/components/maxhealer.lua#L11)、[health.lua:460](../../../raw/current/scripts/components/health.lua#L460
- 修改边界：服务端设置；常规使用走 HEAL，直接 Heal 不检查死亡／幽灵、canheal 或是否确有惩罚，没有自定义接受／完成回调。[actions.lua:3217](../../../raw/current/scripts/actions.lua#L3217)、[maxhealer.lua:1](../../../raw/current/scripts/components/maxhealer.lua#L1

## 关键接口

| 接口／字段 | 参数／返回 | 条件、副作用与证据 |
|---|---|---|
| `SetHealthAmount(health)`／`healamount` | 惩罚比例的**有符号增量** | 默认 TUNING.MAX_HEALING_NORMAL=-0.25；参数不是 HP、也不乘当前缺失比例或 efficientuser。[maxhealer.lua:1](../../../raw/current/scripts/components/maxhealer.lua#L1)、[tuning.lua:2021](../../../raw/current/scripts/tuning.lua#L2021 |
| `Heal(target)` | 有 health：true；没有：nil | 调用 DeltaPenalty 后，堆叠大于一时拆一件删除，否则删除自身。不返回实际恢复量，不按效果决定消费。[maxhealer.lua:11](../../../raw/current/scripts/components/maxhealer.lua#L11 |
| 目标 `health:DeltaPenalty(delta)` | penalty + delta，经 SetPenalty 夹取 | 默认限制到 0～TUNING.MAXIMUM_HEALTH_PENALTY；disable_penalty 阻止惩罚赋值，随后仍 ForceUpdateHUD(false)。[health.lua:460](../../../raw/current/scripts/components/health.lua#L460 |

### 恢复的是哪一种上限

可用上限为 `maxhealth * (1 - penalty)`：基础上限 200、惩罚 0.5 时可用上限 100；传 -0.25 后惩罚 0.25，可用上限 150。DeltaPenalty 的刷新最终走 DoDelta(0)，**没有补回新增的 50 点当前生命**；实际 health 仍可经 redirect／deltamodifier 等回调处理。[health.lua:524](../../../raw/current/scripts/components/health.lua#L524)、[:468](../../../raw/current/scripts/components/health.lua#L468)、[:150](../../../raw/current/scripts/components/health.lua#L150)、[:613](../../../raw/current/scripts/components/health.lua#L613)

组件内旧注释提到复活次数因子；本快照实际直接把 healamount 传给 DeltaPenalty，不能按注释把 -1 解释为“一次复活”。正数会**增加**惩罚；降低到零后不继续增大基础上限。[maxhealer.lua:6](../../../raw/current/scripts/components/maxhealer.lua#L6)、[health.lua:464](../../../raw/current/scripts/components/health.lua#L464

## 典型接入

### 强心针一次减少五十个百分点生命惩罚

放在已启用模组 `modmain.lua`，普通生存模式；复用 lifeinjector 原版物品、资源、堆叠与 HEAL，不增加 Prefab 或注册动作。工厂只添加 maxhealer、采用组件默认量，[lifeinjector.lua:34](../../../raw/current/scripts/prefabs/lifeinjector.lua#L34)、[注册名](../../../raw/current/scripts/prefabs/lifeinjector.lua#L41)。

```lua
AddPrefabPostInit("lifeinjector", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local maxhealer = inst.components.maxhealer
    if maxhealer ~= nil then
        maxhealer:SetHealthAmount(-0.5)
    end
end)
```

-0.5 是惩罚百分点增量；例如原惩罚 0.25 只能恢复到 0，剩余量不转成回血。每份物品照常消费，包含无惩罚目标；此 demo 不改接受条件。新建／载入实例重新设置，停用模组恢复原版药品参数，已减少的惩罚不会自动补回；health 仍保存当前 penalty。[maxhealer.lua:11](../../../raw/current/scripts/components/maxhealer.lua#L11)、[health.lua:154](../../../raw/current/scripts/components/health.lua#L154

## 常见陷阱与限制

- **把 SetHealthAmount(50) 当补回五十点上限**：会增加惩罚并被上限夹取；需要减少惩罚用负比例，需要改变基础上限走 health。单位和符号见上文。
- **已有 healer 与 maxhealer 能同时生效**：HEAL 先选 healer 的 if 分支，maxhealer 是 elseif；同一物品不自动双结算。组合效果应明确选一条入口并审阅消费和回调，不能重复调用两个会删物品的 Heal。[actions.lua:3220](../../../raw/current/scripts/actions.lua#L3220
- **只有缺少惩罚才可使用**：候选使用 `replica.health:CanHeal()`，该接口只看 cannotheal 标签，不检查 penalty 或缺血；执行函数也不检查 penalty。若要求无惩罚不耗药，需另设计服务端门禁，不能调用不存在的 SetCanHealFn。[componentactions.lua:1492](../../../raw/current/scripts/componentactions.lua#L1492)、[:2802](../../../raw/current/scripts/componentactions.lua#L2802)、[health_replica.lua:200](../../../raw/current/scripts/components/health_replica.lua#L200
- **直接调用等于复活**：组件只改惩罚，动作还排除死亡／幽灵；不建立玩家复活状态，也不恢复当前生命。[actions.lua:3219](../../../raw/current/scripts/actions.lua#L3219

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 的变更文件（actions.lua、componentactions.lua、tuning.lua）对照更新报告 R-01/R-04 与 actions/componentactions hunk 地图逐条核验，HEAL 分派、maxhealer 双候选与 MAX_HEALING_NORMAL 均无语义变化，锚点重定位 5 处、结论修订 0 处；未覆盖：运行验证。

- 源码审阅：2026-10-17，Steam buildid 25643519；覆盖组件全部实现、默认数值、强心针工厂、HEAL 分派／候选及 health 惩罚、刷新与保存路径；未审校全部复活来源或特殊模式。
- 结构检查：发布时核对格式、来源、大小、链接、源码行锚点与唯一登记；完整 modmain 示例以已有 Full Moon／Lua 5.1 作 syntax-only 检查，不证明游戏效果。
- 游戏运行：未运行；零惩罚、禁用惩罚、不同基础上限、多人使用及存档恢复待实测。

相关：[[reference/components/health]]；[[reference/components/healer]]；[[reference/components/stackable]]。
