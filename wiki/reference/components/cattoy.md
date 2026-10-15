---
title: "cattoy"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "猫玩具交互组件：定义猫狸/小猫狸逗玩物品时的回调与空中玩具豁免，配合 cattoy 标签被猫脑检索。"
aliases: ["猫玩具", "cattoy", "逗猫交互"]
tags: [component, ai, actions]
size_class: simple
dst_build: "steam-buildid:26230496"
sources: ["components/cattoy.lua", "actions.lua", "brains/catcoonbrain.lua", "brains/kitcoonbrain.lua", "prefabs/kitcoondecor.lua", "prefabs/wx78_scanner.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# cattoy

## 快速判断

- 职责与适用：给"可被猫狸（catcoon）/小猫狸（kitcoon）逗玩的实体"挂交互回调：`Play(doer, is_airborne)` 分发 `onplay_fn`；逗玩方式由动作 CATPLAYGROUND（地面）/CATPLAYAIR（空中）区分。[Play](../../../raw/current/scripts/components/cattoy.lua#L22-L27)
- 优先使用：需要"猫玩具才有专属玩法"的物品挂组件并在 pristine 态加 `cattoy`（或空中玩具加 `cattoyairborne`）标签；组件头注释明确建议两者配合。[注释](../../../raw/current/scripts/components/cattoy.lua#L1-L2)
- 重要副作用：猫脑按标签 `cattoy`/`cattoyairborne`/`catfood` 用 FindEntity 找玩具并选择动作；`ShouldBypassLastAirTime` 可豁免空中玩具的 15 秒冷却限制。[猫脑检索](../../../raw/current/scripts/brains/catcoonbrain.lua#L44-L49)
- 修改边界：无存读、无事件；标签是检索依据，组件与标签可独立存在（许多物品只加标签不挂组件，逗玩回退到猫脑的攻击/拾取分支）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SetOnPlay(fn)` | `fn(inst, doer, is_airborne) -> any` | Play 的分发目标；返回值透传（nil 视为未处理，Play 返回 false）。[实现](../../../raw/current/scripts/components/cattoy.lua#L10-L12) |
| `Play(doer, is_airborne)` | `-> onplay_fn 结果 \| false` | 无回调时返回 false；动作侧不检查返回值。[实现](../../../raw/current/scripts/components/cattoy.lua#L22-L27) |
| `SetBypassLastAirTime(boolval)` / `ShouldBypassLastAirTime()` | `boolval or nil` | 置真后猫脑跳过"距上次空中逗玩 < 15 秒"的冷却（catcoonbrain 判定）。[实现](../../../raw/current/scripts/components/cattoy.lua#L14-L20) |
| `cattoy` / `cattoyairborne`（标签） | — | 检索用；kitcoonbrain 按标签二选一决定 CATPLAYAIR/CATPLAYGROUND。[标签选择](../../../raw/current/scripts/brains/kitcoonbrain.lua#L213-L215) |

## 典型接入

### 让自定义物品成为猫玩具（原版参照）

- 前提与选择：服务端；同时加标签与组件。原版挂组件的先例：kitcoondecor 装饰品（([kitcoondecor.lua:144-145](../../../raw/current/scripts/prefabs/kitcoondecor.lua#L144-L145)）、WX-78 扫描器与收割无人机（([wx78_scanner.lua:947-949](../../../raw/current/scripts/prefabs/wx78_scanner.lua#L947-L949)，均置 BypassLastAirTime）。[挂组件](../../../raw/current/scripts/prefabs/kitcoondecor.lua#L144-L145)
- 生效与撤销：kitcoon 选定目标后会先摘标签预留 30 秒防抢（([kitcoonbrain.lua:217-219](../../../raw/current/scripts/brains/kitcoonbrain.lua#L217-L219)）；动作执行时按目标是否有 cattoy 组件分发 Play，无组件回退 poppable/攻击/拾取分支。[动作分发](../../../raw/current/scripts/actions.lua#L3884-L3887)
- 原版依据：只加标签的物品（papyrus、rope、foliage、鹅毛、饰品、鱼漂等十余种）逗玩时走猫脑默认分支，不触发自定义回调——组件与标签职责不同。

```lua
-- 服务端：自定义猫玩具（pristine 态加 cattoy 标签）
inst:AddTag("cattoy")
-- master 分支：
inst:AddComponent("cattoy")
inst.components.cattoy:SetOnPlay(function(inst, doer, is_airborne)
    doer.components.health:DoDelta(1) -- 示例副作用
    return true
end)
```

- 调整与验证：空中玩具加 `cattoyairborne` 标签；若玩具落会后仍希望立刻可玩，再调 `SetBypassLastAirTime(true)`。

## 常见陷阱与限制

- **只挂组件不加标签：** 猫脑 FindEntity 的 oneof 标签表是 `{"cattoy", "cattoyairborne", "catfood"}`，无标签的组件永远不会被选中逗玩；两者必须配合。[检索标签](../../../raw/current/scripts/brains/catcoonbrain.lua#L24)
- **onplay_fn 返回值被忽略：** 动作侧 CATPLAYGROUND/CATPLAYAIR 调用 Play 后不检查结果即返回成功（([actions.lua:3912-3915](../../../raw/current/scripts/actions.lua#L3912-L3915)）；失败逻辑要在回调内自行处理。
- **kitcoondecor 运行期移除组件：** 装饰类别扣为空时 `RemoveComponent("cattoy")`（[kitcoondecor.lua:54](../../../raw/current/scripts/prefabs/kitcoondecor.lua#L54），但标签保留——组件与标签生命周期不绑定，消费方都能容忍这种半状态。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（30 行），核对 actions.lua 两处动作分发、catcoonbrain/kitcoonbrain 检索与冷却判定、kitcoondecor 与 wx78_scanner 挂载；grep 确认标签-only 物品清单。
- 结构检查：2026-10-28 validate_page.py 通过（本页）；validate_source_links.py --strict 通过；Lua 示例经 parser_probe --syntax-only 解析通过。
- 游戏运行：未运行；逗玩冷却与回调时序未实测。

相关：[[reference/components/kitcoon]]
