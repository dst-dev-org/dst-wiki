---
title: "moonstormstaticcatcher"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "月风暴静电捕捉端：装备物品用它对 moonstormstaticcapturable 目标执行 DIVEGRAB 抓取并维护瞄准通知。"
aliases: ["静电捕捉器", "moonstormstaticcatcher", "月风暴捕捉"]
tags: [component, item]
dst_build: "steam-buildid:26230496"
sources: ["components/moonstormstaticcatcher.lua", "components/moonstormstaticcapturable.lua", "actions.lua", "componentactions.lua", "stategraphs/SGwilson.lua", "prefabs/moonstorm_static.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# moonstormstaticcatcher

## 快速判断

- 职责与适用：挂在捕捉用物品上（原版为月风暴静电物品，带 inventoryitem/equippable，[宿主](../../../raw/current/scripts/prefabs/moonstorm_static.lua#L394-L395)）；对目标执行"接住"判定并把瞄准状态通知给目标侧组件（[组件全文](../../../raw/current/scripts/components/moonstormstaticcatcher.lua#L1-L52)）。
- 优先使用：玩家动作 DIVEGRAB 驱动 `Catch`——动作的 `invobject` 即捕捉器（[执行](../../../raw/current/scripts/actions.lua#L6594-L6602)）；候选要求目标带 `moonstormstaticcapturable` 标签（[候选](../../../raw/current/scripts/componentactions.lua#L2399-L2402)）。
- 重要副作用：瞄准/取消瞄准经目标侧 `moonstormstaticcapturable:OnTargeted/OnUntargeted` 记账，目标会推 `moonstormstaticcapturable_targeted/untargeted` 事件（[目标侧](../../../raw/current/scripts/components/moonstormstaticcapturable.lua#L50-L76)）。
- 修改边界：`OnRemoveFromEntity` 自动取消瞄准；距离判定使用捕捉器持有者位置，目标超出即 "MISSED"。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `Catch(target, doer)` | `-> bool, reason` | 校验目标有效、距离 ≤ `1 + doer 半径 + 0.2`（0.2 与月风暴物品约定一致）、目标有启用的 `moonstormstaticcapturable`；通过则调目标 `OnCaught` 与 `oncaughtfn(inst, doer)`，否则 `false, "MISSED"`。[实现](../../../raw/current/scripts/components/moonstormstaticcatcher.lua#L15-L30) |
| `OnTarget(target)` | `target: entity -> 无` | 换目标时先对旧目标 OnUntarget；仅对有效且带 capturable 的目标登记。[实现](../../../raw/current/scripts/components/moonstormstaticcatcher.lua#L32-L40) |
| `OnUntarget(target)` | `target: entity\|nil -> 无` | nil 表示取消当前目标；对目标侧做对称注销。[实现](../../../raw/current/scripts/components/moonstormstaticcatcher.lua#L42-L50) |
| `SetOnCaughtFn(fn)` | `fn(inst, doer)` | 接住成功后的宿主回调；月风暴物品用它结算风暴计数（[挂接](../../../raw/current/scripts/prefabs/moonstorm_static.lua#L395)）。 |

## 典型接入

### 抓取判定链路（原版参照）

- 前提与选择：目标侧需实现 `moonstormstaticcapturable` 组件并提供 `IsEnabled`/`OnCaught`/`OnTargeted`/`OnUntargeted`（[目标侧约定](../../../raw/current/scripts/components/moonstormstaticcapturable.lua#L50-L78)）。
- 生效与撤销：SGwilson 瞄准阶段即调 `OnTarget` 让目标进入被瞄准态（[SG](../../../raw/current/scripts/stategraphs/SGwilson.lua#L26883-L26885)），动作执行成功后由 Catch 收尾；失败路径（MISSED）不自动清瞄准。

```lua
-- 服务端：动作链路等价调用
local ok, reason = catcher_item.components.moonstormstaticcatcher:Catch(target, doer)
if not ok then
    -- reason 为 "MISSED"
end
```

- 调整与验证：距离常数 `1 + doer:GetPhysicsRadius(0) + 0.2` 中 0.2 为硬编码并注释与目标侧保持一致（[实现](../../../raw/current/scripts/components/moonstormstaticcatcher.lua#L16-L19)），自定义目标需预留同等半径。

## 常见陷阱与限制

- **目标是 capturable 而非捕捉器**：组件名容易读反—— Catch 的参数 target 是被捕捉物，组件宿主（捕捉器）是 `invobject`；目标缺组件时一律 "MISSED"。
- **瞄准与捕捉独立**：OnTarget/OnUntarget 只维护目标侧的"被瞄准"状态，不保证后续 Catch 成功；目标删除由 capturable 的 onremove 监听兜底注销（[目标侧](../../../raw/current/scripts/components/moonstormstaticcapturable.lua#L55-L57)）。
- **oncaughtfn 不接收目标**：回调只给 `(inst, doer)`（[实现](../../../raw/current/scripts/components/moonstormstaticcatcher.lua#L26-L28)），需要目标信息时在回调外层闭包捕获。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；覆盖组件全文、moonstormstaticcapturable 三个对接方法、DIVEGRAB 候选与执行、SGwilson 瞄准调用、moonstorm_static 宿主挂接。
- 结构检查：本批 validate_page、validate_source_links 及示例 lua 语法探测（见批级记录）。
- 游戏运行：未运行。

## 相关链接

- [[reference/components/moonstormstaticcapturable]]：静电可捕捉目标端：瞄准集合与被抓结算。
