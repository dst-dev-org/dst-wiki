---
title: "pocketwatch"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "旺达怀表的施法基座：inactive 开关联动 pocketwatch_inactive 标签，CanCast／CastSpell 执行 prefab 注入的 CanCastFn／DoCastSpell。"
aliases: ["pocketwatch", "怀表施法组件", "回溯表施法基座"]
tags: [component, item, actions]
size_class: simple
dst_build: "steam-buildid:26230496"
sources: ["components/pocketwatch.lua", "prefabs/pocketwatch_common.lua", "prefabs/pocketwatch.lua", "prefabs/pocketwatch_portal.lua", "componentactions.lua", "actions.lua", "stategraphs/SGwilson.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# pocketwatch

## 快速判断

- 职责与适用：挂在怀表类物品上的施法调度组件；`inactive` 字段默认 true，经 Class 属性 setter 联动 `pocketwatch_inactive` 标签（标签是动作候选的门禁）。[构造与标签](../../../raw/current/scripts/components/pocketwatch.lua#L2-L14)
- 优先使用：施法效果与目标校验都由 prefab 注入——`DoCastSpell`（pocketwatch_common 的 common_fn 参数）与可选 `CanCastFn`；模组按同一模式装配，不改组件本体。[装配](../../../raw/current/scripts/prefabs/pocketwatch_common.lua#L81-L82)
- 重要副作用：与 rechargeable 联动——放电置 `inactive = false`（候选消失），充满恢复 true；移除组件时撤标签。[充能联动](../../../raw/current/scripts/prefabs/pocketwatch_common.lua#L12-L23)
- 修改边界：组件不检查冷却（由 rechargeable 承担）、不实现效果；候选还要求使用者有 `pocketwatchcaster` 标签与坐骑兼容标签（`pocketwatch_mountedcast`）。[候选](../../../raw/current/scripts/componentactions.lua#L1560-L1563)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `inactive` | `boolean`（默认 true） | setter 同步 `pocketwatch_inactive` 标签；OnRemoveFromEntity 撤标签。[联动](../../../raw/current/scripts/components/pocketwatch.lua#L2-L14) |
| `CanCast(doer, target, pos)` | `-> boolean` | `inactive` 且 `CanCastFn` 为空或其返回真；SGwilson 施法状态里再校验一次。[实现](../../../raw/current/scripts/components/pocketwatch.lua#L21-L23) [SG 校验](../../../raw/current/scripts/stategraphs/SGwilson.lua#L20469) |
| `CastSpell(doer, target, pos)` | `-> success, reason` | 需 `DoCastSpell` 已注入且 inactive；返回 `DoCastSpell(inst, doer, target, pos)` 的结果，否则 `false`。[实现](../../../raw/current/scripts/components/pocketwatch.lua#L25-L32) |
| `DoCastSpell`／`CanCastFn` | prefab 注入的函数字段 | portal 在 common_fn 传 DoCastSpell；复活表（pocketwatch prefab）另设 `CanCastFn = Revive_CanTarget`。[复活表注入](../../../raw/current/scripts/prefabs/pocketwatch.lua#L158) |

## 典型接入

### 自定义怀表施法（原版装配参照）

- 前提与选择：物品走 `PocketWatchCommon.common_fn(bank, build, DoCastSpell, cast_from_inventory, ...)` 统一装配——组件挂载、DoCastSpell 注入、rechargeable 联动一次完成；portal 与复活表是两个已核验范例。[portal 装配](../../../raw/current/scripts/prefabs/pocketwatch_portal.lua#L112) [portal 效果](../../../raw/current/scripts/prefabs/pocketwatch_portal.lua#L42)
- 生效与撤销：CAST_POCKETWATCH 动作执行时才调 `CastSpell`（actions.lua#L3146-L3149）；复活表还支持作祟触发施法。[动作](../../../raw/current/scripts/actions.lua#L3146-L3149) [作祟施法](../../../raw/current/scripts/prefabs/pocketwatch.lua#L139-L141)

```lua
-- 服务端：自定义怀表效果（经 PocketWatchCommon.common_fn 装配后）
inst.components.pocketwatch.DoCastSpell = function(inst, doer, target, pos)
    doer:PushEvent("mypocketwatchspell")
    return true
end
-- 充能联动：放电自动 inactive=false，充满恢复 true
```

- 调整与验证：目标点／目标实体候选分两路（point 与 inventory），标签门禁一致；拆表归 pocketwatch_dismantler 组件，与本组件无耦合。[背包候选](../../../raw/current/scripts/componentactions.lua#L2847-L2850)

## 常见陷阱与限制

- **把 inactive 当"组件开关"：** 它只表达"当前可施法"，由充能状态驱动；直接置 false 而不接 rechargeable 会让表永久不可用且不显示冷却动画。[充能联动](../../../raw/current/scripts/prefabs/pocketwatch_common.lua#L12-L23)
- **忘注入 DoCastSpell：** `CastSpell` 直接返回 false，动作表现为"可用但无效"；检查装配顺序与 common_fn 参数。[实现](../../../raw/current/scripts/components/pocketwatch.lua#L25-L32)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（34 行），核对 pocketwatch_common 装配与充能联动、复活表 CanCastFn／作祟施法、portal 装配、两路动作候选、动作执行与 SG 校验。未审阅各怀表 DoCastSpell 效果体（回溯／传送／复活逻辑）。
- 结构检查：2026-10-28 validate_page.py 通过；validate_source_links.py --strict 锚点通过（见批级记录）。
- 游戏运行：未运行。

相关：[[reference/components/pocketwatch-dismantler]]
