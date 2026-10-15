---
title: "spidermutator"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "蜘蛛变异器组件：挂在可让蜘蛛变异的物品上，执行时把蜘蛛替换成目标蜘蛛 prefab，变异器自身被消耗。"
aliases: ["蜘蛛变异器", "spidermutator", "蜘蛛变异物品"]
tags: [component, item]
dst_build: "steam-buildid:26230496"
sources: ["components/spidermutator.lua", "prefabs/spider_mutators.lua", "actions.lua", "componentactions.lua", "prefabs/spider.lua", "stategraphs/SGspider.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# spidermutator

## 快速判断

- 职责与适用：挂在变异器物品（spider_mutators 系列）上，记录变异目标 prefab；执行时把目标蜘蛛换成目标 prefab 的个体，随后消耗变异器（[组件全文](../../../raw/current/scripts/components/spidermutator.lua#L1-L43)）。
- 优先使用：物品加组件并 `SetMutationTarget("spider_xxx")`（[宿主](../../../raw/current/scripts/prefabs/spider_mutators.lua#L67-L68)）；玩家对蜘蛛使用走 MUTATE_SPIDER 动作，蜘蛛自己吃下变异器也能触发。
- 重要副作用：执行后变异器被消耗——堆叠物品取一份移除，否则整个实体移除（[消耗](../../../raw/current/scripts/components/spidermutator.lua#L36-L40)）；被收容蜘蛛变异时新个体以给予者为领队（[领队](../../../raw/current/scripts/components/spidermutator.lua#L25)）。
- 修改边界：执行只分"蜘蛛被收容"与"自由个体"两条路径；`CanMutate` 仅比较 prefab 名（[判定](../../../raw/current/scripts/components/spidermutator.lua#L9-L11)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SetMutationTarget(target)` | `target: string（目标蜘蛛 prefab 名）` | 记录 `mutation_target`；宿主在 AddComponent 后立即调用。[实现](../../../raw/current/scripts/components/spidermutator.lua#L5-L7) |
| `CanMutate(spider)` | `spider: entity -> bool` | 仅 `spider.prefab ~= self.mutation_target`；同目标 prefab 返回 false。[实现](../../../raw/current/scripts/components/spidermutator.lua#L9-L11) |
| `Mutate(spider, skip_event, giver)` | `-> 无；执行即消耗自身` | 收容路径：生成新个体、占原槽位（inventory 或 container）、移除旧体、`follower:SetLeader(giver)`（[收容路径](../../../raw/current/scripts/components/spidermutator.lua#L14-L25)）；自由路径：写 `spider.mutation_target`/`spider.mutator_giver` 并推 `mutate` 事件，`skip_event=true` 跳过（[自由路径](../../../raw/current/scripts/components/spidermutator.lua#L27-L32)）。 |

## 典型接入

### 让自定义物品变异蜘蛛

- 前提与选择：物品加 `spidermutator` 标签供原版识别（原版在 prefab 内加，[标签](../../../raw/current/scripts/prefabs/spider_mutators.lua#L47)），主机端加组件并设目标 prefab。
- 生效与撤销：执行函数要求 `CanMutate` 通过，否则返回 "SAMETYPE"（[执行](../../../raw/current/scripts/actions.lua#L5641-L5647)）；候选要求目标带 `spider` 标签且未死亡（[候选](../../../raw/current/scripts/componentactions.lua#L1802-L1805)）；消耗发生在 Mutate 末尾，无法撤销。

```lua
AddPrefabPostInit("my_spider_mutator", function(inst)
    inst:AddTag("spidermutator")
    if not GLOBAL.TheWorld.ismastersim then
        return inst
    end
    inst:AddComponent("spidermutator")
    inst.components.spidermutator:SetMutationTarget("spider_dropper")
end)
```

- 调整与验证：蜘蛛吃下变异器同样触发——oneat 监听检查组件后调 `Mutate(inst)`（[蜘蛛侧](../../../raw/current/scripts/prefabs/spider.lua#L366-L369)）；蜘蛛进食状态图路径传 `skip_event=true`（[SG](../../../raw/current/scripts/stategraphs/SGspider.lua#L7-L10)）。

## 常见陷阱与限制

- **CanMutate 只比 prefab 名**：目标名必须与蜘蛛实体 prefab 完全一致；同 prefab 判定不可变异，需要绕过时只能自行调 Mutate。
- **mutate 事件无原版监听者**：自由蜘蛛路径只推事件（[推送](../../../raw/current/scripts/components/spidermutator.lua#L31)），全库未发现监听方；原版变异效果全靠"直接替换实体"完成，事件仅留给宿主脚本扩展。
- **收容路径假定新蜘蛛有 follower 组件**：`new_spider.components.follower:SetLeader(giver)` 无 nil 检查（[调用](../../../raw/current/scripts/components/spidermutator.lua#L25)），变异目标 prefab 必须带 follower。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；覆盖组件全文、spider_mutators 宿主挂接、MUTATE_SPIDER 候选与执行、蜘蛛 oneat 与 SGspider 两条触发路径、`mutate` 事件全库检索。
- 结构检查：本批 validate_page、validate_source_links 及示例 lua 语法探测（见批级记录）。
- 游戏运行：未运行。

相关：[[reference/components/follower]]
