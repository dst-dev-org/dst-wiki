---
title: "moonstormstaticcapturable"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "月风暴静电可捕捉目标组件：维护启用开关与瞄准者集合，把被瞄准／被抓住状态广播给宿主回调、事件与状态图。"
aliases: ["静电捕捉目标", "可捕捉目标", "moonstormstaticcapturable"]
tags: [component, creature, callback]
dst_build: "steam-buildid:26230496"
sources: ["components/moonstormstaticcapturable.lua", "components/moonstormstaticcatcher.lua", "prefabs/moonstorm_static.lua", "stategraphs/SGmoonstormstatic.lua", "components/playercontroller.lua", "componentactions.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# moonstormstaticcapturable

## 快速判断

- 职责与适用：挂在月风暴漫游静电上（[挂接](../../../raw/current/scripts/prefabs/moonstorm_static.lua#L529-L530)），与捕捉器组件（见 [[reference/components/moonstormstaticcatcher]]）对接：登记瞄准者、报告启用状态、执行被抓结算（[全文](../../../raw/current/scripts/components/moonstormstaticcapturable.lua#L1-L19)）。
- 优先使用：捕捉器侧调 `OnTargeted/OnUntargeted/OnCaught`（[调用](../../../raw/current/scripts/components/moonstormstaticcatcher.lua#L25-L47)）；宿主侧只设三个回调与 `SetEnabled`。
- 重要副作用：`enabled` 变化经属性观察器同步增删 `moonstormstaticcapturable` 标签（[观察器](../../../raw/current/scripts/components/moonstormstaticcapturable.lua#L1-L4)），标签在 prefab pristine 已预置（[预置](../../../raw/current/scripts/prefabs/moonstorm_static.lua#L505)）。
- 修改边界：瞄准者删除经 `onremove` 监听自动注销（[兜底](../../../raw/current/scripts/components/moonstormstaticcapturable.lua#L14)）；组件移除时清标签。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SetEnabled(enabled)` / `IsEnabled()` | `enabled: bool`，默认 `true` | 关闭后捕捉器 `Catch` 判定 MISSED；写属性自动同步标签（[实现](../../../raw/current/scripts/components/moonstormstaticcapturable.lua#L25-L31)）。 |
| `SetOnTargetedFn/OnUntargetedFn/OnCaughtFn(fn)` | 见下三行 | 首个瞄准者出现／最后一个离开／被抓时调用（[注册](../../../raw/current/scripts/components/moonstormstaticcapturable.lua#L33-L43)）。 |
| `OnTargeted(obj)` / `OnUntargeted(obj)` | `obj: 捕捉器实体 -> 无` | 维护 `targeters` 表；只在"从无到有／从有到无"的边界触发回调与事件 `moonstormstaticcapturable_targeted/untargeted`（[实现](../../../raw/current/scripts/components/moonstormstaticcapturable.lua#L50-L76)）。 |
| `OnCaught(obj, doer)` | `-> 无` | doer 推 `moonstormstatic_caught` 事件（事件 data 为**目标实体本身**），再调 `oncaughtfn(inst, obj, doer)`（[实现](../../../raw/current/scripts/components/moonstormstaticcapturable.lua#L79-L87)）。 |
| `IsTargeted()` | `-> bool` | `targeters` 非空即被瞄准（[实现](../../../raw/current/scripts/components/moonstormstaticcapturable.lua#L45-L47)）。 |

## 典型接入

### 原版链路：选中→瞄准→抓取

- 前提与选择：玩家控制器把带标签的实体作为强制目标（半径 8 内，`distsq <= 64`，[playercontroller.lua:2185](../../../raw/current/scripts/components/playercontroller.lua#L2185）；动作候选同样按标签过滤（[componentactions.lua:2400](../../../raw/current/scripts/componentactions.lua#L2400）。
- 生效与撤销：SGmoonstormstatic 监听 targeted/untargeted 事件切换被打断状态（([SGmoonstormstatic.lua:17-23](../../../raw/current/scripts/stategraphs/SGmoonstormstatic.lua#L17-L23)）；宿主用 `SetOnCaughtFn` 结算漫游静电的抓取（[宿主](../../../raw/current/scripts/prefabs/moonstorm_static.lua#L529-L530)）。

```lua
-- 服务端：宿主侧最小接法（moonstorm_static 等价）
inst:AddTag("moonstormstaticcapturable") -- pristine 阶段预置
local capturable = inst:AddComponent("moonstormstaticcapturable")
capturable:SetOnCaughtFn(function(inst, obj, doer)
    -- obj 为捕捉器实体，doer 为玩家
end)
```

- 调整与验证：目标实体需保留标签供远程检索；禁用用 `SetEnabled(false)` 而非移除组件（移除会丢回调）。

## 常见陷阱与限制

- **事件 data 不是表**：`moonstormstatic_caught` 的第二个参数直接是目标实体（`PushEvent(name, self.inst)`），监听方按实体读而不是 `data.target`（[实现](../../../raw/current/scripts/components/moonstormstaticcapturable.lua#L81)）。
- **targeted 事件只在边界触发**：第二个捕捉器瞄准同一目标不再推事件；需要"当前有几个瞄准者"应读 `IsTargeted` 与内部表而非数事件（[边界](../../../raw/current/scripts/components/moonstormstaticcapturable.lua#L51-L60)）。
- **标签双写**：pristine 标签（视觉/检索）与属性观察器动态标签并存，禁用后客户端检索立即失效；若宿主 pristine 忘写标签，客户端在首帧前检索不到。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；覆盖组件全文、捕捉器三处调用、宿主挂接、SG 事件、玩家控制器与动作候选过滤。
- 结构检查：本批 validate_page、validate_source_links 及示例 lua 语法探测（见批级记录）。
- 游戏运行：未运行。
