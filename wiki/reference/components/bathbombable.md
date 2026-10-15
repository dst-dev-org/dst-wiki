---
title: "bathbombable"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "浴弹命中目标标记：用 can_be_bathbombed 同步 bathbombable 标签、记录 is_bathbombed 状态并转发命中回调，温泉宿主配套组件。"
aliases: ["可浴弹目标", "bathbombable", "温泉浴弹标记"]
tags: [component, actions]
dst_build: "steam-buildid:26230496"
sources: ["components/bathbombable.lua", "actions.lua", "componentactions.lua", "prefabs/hotspring.lua", "prefabs/hermithotspring.lua", "brains/hermitcrabbrain.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# bathbombable

## 快速判断

- 职责与适用：挂在可被浴弹命中的实体（温泉）上；维护"能否再被炸"与"已被炸过"两个状态位，命中时转发宿主回调（[组件全文](../../../raw/current/scripts/components/bathbombable.lua#L10-L20)）。
- 优先使用：宿主用 `SetOnBathBombedFn` 挂命中回调（[设置](../../../raw/current/scripts/components/bathbombable.lua#L26-L28)）；浴弹动作侧按 `can_be_bathbombed` 决定是否接受（[执行](../../../raw/current/scripts/actions.lua#L4822-L4823)）。
- 重要副作用：`can_be_bathbombed` 经 watcher 同步 `bathbombable` 标签——动作候选与隐士蟹 AI 都按此标签识别（[watcher](../../../raw/current/scripts/components/bathbombable.lua#L2-L8)、[候选](../../../raw/current/scripts/componentactions.lua#L1052-L1055)、[AI](../../../raw/current/scripts/brains/hermitcrabbrain.lua#L493)）。
- 修改边界：`OnBathBombed` 把状态置为"已炸且不可再炸"，之后由宿主按需 `Reset`／`DisableBathBombing`；`OnRemoveFromEntity` 仅摘标签（[摘标签](../../../raw/current/scripts/components/bathbombable.lua#L22-L24)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `can_be_bathbombed` | `bool，默认 true（watcher）` | 写入即同步/摘除 `bathbombable` 标签；动作候选按标签出现。[watcher](../../../raw/current/scripts/components/bathbombable.lua#L2-L8) |
| `is_bathbombed` | `bool，默认 false` | 已被浴弹命中；温泉热量与外观按此切换（[热量](../../../raw/current/scripts/prefabs/hermithotspring.lua#L511)）。 |
| `OnBathBombed(item, doer)` | `item/doer: entity\|nil -> 无` | 置 `is_bathbombed=true`、`can_be_bathbombed=false` 后调用 `onbathbombedfn(inst, item, doer)`；原版存在无参调用（item/doer 为 nil）。[实现](../../../raw/current/scripts/components/bathbombable.lua#L30-L37) |
| `SetOnBathBombedFn(new_fn)` | `new_fn: fn(inst, item, doer)` | 挂命中回调；宿主在 AddComponent 后调用。[实现](../../../raw/current/scripts/components/bathbombable.lua#L26-L28) |
| `DisableBathBombing()` | `-> 无` | 置 `is_bathbombed=false`、`can_be_bathbombed=false`（"禁用"态）。[实现](../../../raw/current/scripts/components/bathbombable.lua#L39-L42) |
| `Reset()` | `-> 无` | 置 `is_bathbombed=false`、`can_be_bathbombed=true`，恢复可命中。[实现](../../../raw/current/scripts/components/bathbombable.lua#L44-L47) |

## 典型接入

### 给自定义温泉类实体接浴弹效果

- 前提与选择：主机端加组件并挂回调；浴弹动作侧的物品端见 [[reference/components/bathbomb]]。
- 生效与撤销：执行链为候选双标签 → 动作函数检查 `can_be_bathbombed` → `OnBathBombed(invobject, doer)` → 宿主回调内做玻璃化等效果（[执行](../../../raw/current/scripts/actions.lua#L4819-L4823)）；原版温泉在玻璃化消退时调 `Reset` 重新开放（[hermithotspring](../../../raw/current/scripts/prefabs/hermithotspring.lua#L437)）。

```lua
AddPrefabPostInit("my_hotspring", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return inst
    end
    inst:AddComponent("bathbombable")
    inst.components.bathbombable:SetOnBathBombedFn(function(inst, item, doer)
        -- 浴弹命中后的自定义效果
    end)
end)
```

- 调整与验证：回调参数 `item`/`doer` 可能为 nil（原版 `OnBathBombed()` 无参调用见 [hotspring.lua:321](../../../raw/current/scripts/prefabs/hotspring.lua#L321），回调内做空值保护。

## 常见陷阱与限制

- **候选靠标签、执行靠组件**：只给实体加 `bathbombable` 标签时浴弹动作会出现，但执行函数读 `act.target.components.bathbombable`，缺组件直接失败；两者都要有。
- **命中即双重置位**：`OnBathBombed` 同时置 `is_bathbombed=true` 与 `can_be_bathbombed=false`（[实现](../../../raw/current/scripts/components/bathbombable.lua#L31-L32)）；宿主若只想"可反复命中"需自行在回调里 `Reset`。
- **效果全在宿主回调**：玻璃化、热量、计时器都在温泉宿主脚本内，本组件不携带任何效果数据；宿主侧状态恢复见 hermithotspring 的存读（hermithotspring.lua:592-597）。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；覆盖组件全文、BATHBOMB 候选与执行、hotspring/hermithotspring 两个宿主的挂接与状态消费、隐士蟹 AI 标签规避。
- 结构检查：本批 validate_page、validate_source_links 及示例 lua 语法探测（见批级记录）。
- 游戏运行：未运行。

相关：[[reference/components/bathbomb]]
