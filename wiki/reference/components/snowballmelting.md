---
title: "snowballmelting"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "雪球融化状态机：按环境温度与世界雪覆盖状态在 solid/melting 间切换，周期复查并触发宿主的开始/持续/停止融化回调。"
aliases: ["雪球融化", "snowballmelting"]
tags: [component, state-machine]
dst_build: "steam-buildid:26230496"
sources: ["components/snowballmelting.lua", "prefabs/snowball_item.lua", "prefabs/snowman.lua", "components/snowballmanager.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# snowballmelting

## 快速判断

- 职责与适用：挂在雪制实体（原版 [snowball_item](../../../raw/current/scripts/prefabs/snowball_item.lua#L265-L267) 与 snowman），在 `solid`/`melting` 两态间切换：世界有雪覆盖时保持 solid，否则按环境温度（`GetTemperatureAtXZ > 0`）判定融化。
- 优先使用：需要"落地才融化、收进背包冻结"的雪类物品，按原版接法配 `onputininventory`/`ondropped` 事件（[L159-L165](../../../raw/current/scripts/prefabs/snowball_item.lua#L159-L165)）。
- 重要副作用：组件自排 30±5 秒复查任务（[L1-L2](../../../raw/current/scripts/components/snowballmelting.lua#L1-L2)），并在雪覆盖状态上动态挂/撤 `issnowcovered` 世界状态监听（[L30-L47](../../../raw/current/scripts/components/snowballmelting.lua#L30-L47)）。
- 修改边界：组件不销毁实体——融化持续由宿主在 `ondomeltaction` 里自行 `Remove()`；`OnRemoveEntity` 时停止全部任务（[L20-L22](../../../raw/current/scripts/components/snowballmelting.lua#L20-L22)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `AllowMelting()` | `-> void` | 取消现有任务后立即做一次 `CheckStartMelting`，并重排 30~35 秒周期复查；snowball_item 在落地/初始化时调用。[实现](../../../raw/current/scripts/components/snowballmelting.lua#L87-L98) |
| `StopMelting()` | `-> void` | 取消任务、撤雪覆盖监听并回 solid（触发 `onstopmelting`）；入背包或 snowballmanager 批量冻结时调用。[实现](../../../raw/current/scripts/components/snowballmelting.lua#L104-L123) |
| `ShouldMelt()` | `-> bool` | 实体所在位置环境温度 > 0。[实现](../../../raw/current/scripts/components/snowballmelting.lua#L24-L28) |
| `CheckTemperature` | 内部复查流程 | 雪覆盖 → 保持 solid 并挂监听等待；无雪 → 按温度切 melting（进 `onstartmelting`）或每周期触发 `ondomeltaction`。[实现](../../../raw/current/scripts/components/snowballmelting.lua#L30-L72) |
| `SetOnStartMelting(fn)` | `fn(inst) -> void` | solid → melting 时触发；snowman 用它做视觉变化。[实现](../../../raw/current/scripts/components/snowballmelting.lua#L100-L102) |
| `SetOnDoMeltAction(fn)` | `fn(inst) -> void` | 已在 melting 态且复查仍应融化时周期触发；snowball_item 在此销毁实体。[实现](../../../raw/current/scripts/components/snowballmelting.lua#L74-L76) |
| `SetOnStopMelting(fn)` | `fn(inst) -> void` | 回 solid 时触发。[实现](../../../raw/current/scripts/components/snowballmelting.lua#L125-L127) |
| `state` | `"solid" \| "melting"` | 当前状态字段，构造默认 solid（[L9](../../../raw/current/scripts/components/snowballmelting.lua#L9)）。 |

## 典型接入

### 落地会融化的雪球类物品

- 前提与选择：组件在物品 master postinit 挂；融化表现与销毁都通过回调由宿主定义，原版组合是 `OnPutInInventory → StopMelting`、`OnDropped → AllowMelting`（[L159-L165](../../../raw/current/scripts/prefabs/snowball_item.lua#L159-L165)）。

```lua
inst:AddComponent("snowballmelting")
inst.components.snowballmelting:SetOnDoMeltAction(function(inst_)
    inst_:Remove() -- 原版在此生成碎裂特效后移除
end)
inst.components.snowballmelting:AllowMelting()

inst:ListenForEvent("onputininventory", function(inst_)
    inst_.components.snowballmelting:StopMelting()
end)
inst:ListenForEvent("ondropped", function(inst_)
    inst_.components.snowballmelting:AllowMelting()
end)
```

- 调整与验证：复查周期固定 30±5 秒不可配；世界下雪期间组件只挂监听不排任务，雪停后才进入周期复查。

## 常见陷阱与限制

- **不接背包事件就会在背包里融化**：组件不感知 INLIMBO，宿主必须像 snowball_item 一样在入包/落地时切换，否则持续触发 `ondomeltaction`。
- **ondomeltaction 是周期性触发**：melting 态下每 30~35 秒触发一次而非一次性，销毁逻辑要幂等。
- **snowballmanager 会批量接管**：世界组件按需对场上雪球调 `AllowMelting/StopMelting`（[snowballmanager](../../../raw/current/scripts/components/snowballmanager.lua#L49-L50)），宿主回调需可被外部多次启停。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；组件全文、snowball_item/snowman 挂接、snowballmanager 批量启停。
- 游戏运行：未运行。
