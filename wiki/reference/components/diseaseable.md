---
title: "diseaseable"
created: "2026-10-28"
updated: "2026-10-28"
type: component
size_class: simple
summary: "疾病传播组件：延迟骰→预警→发病→周期传染的完整流程；当前快照无任何原版挂载点，仅存防御性读取。"
aliases: ["疾病", "diseaseable", "病害传播"]
tags: [component]
dst_build: "steam-buildid:26230496"
sources: ["components/diseaseable.lua", "components/inspectable.lua", "components/burnable.lua", "components/workable.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# diseaseable

## 快速判断

- 职责与适用：实体疾病生命周期组件：挂载即打 `diseaseable` 标签并自动开始延迟骰，依次经过延迟→预警（`_warningtask`）→发病（`diseased=true`、`diseased` 标签）→周期传染（[构造](../../../raw/current/scripts/components/diseaseable.lua#L62-L77)）。
- 优先使用：想接入疾病的模组需自行 `AddComponent("diseaseable")` 并处理 `onDiseasedFn`——原版快照中没有现成参照。
- 重要副作用：发病与延迟重排会以 `diseaseable` 标签搜索邻域实体并重置其延迟任务（[扩散](../../../raw/current/scripts/components/diseaseable.lua#L33-L53)）；`diseased` 字段 watcher 同步标签（[watcher](../../../raw/current/scripts/components/diseaseable.lua#L1-L7)）。
- 修改边界：`OnRemoveFromEntity` 取消全部任务并移除两个标签（[清理](../../../raw/current/scripts/components/diseaseable.lua#L83-L102)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `IsDiseased()` / `IsBecomingDiseased()` | → `bool` | 分别对应已发病与预警中（`_warningtask ~= nil`）（[实现](../../../raw/current/scripts/components/diseaseable.lua#L104-L110)）。 |
| `SetDiseasedFn(fn)` | fn(inst) | 发病回调（[实现](../../../raw/current/scripts/components/diseaseable.lua#L112-L114)）。 |
| `Disease()` | → `nil` | 直接发病：置状态、排传染任务、排 diseaseflies 特效任务、触发回调（[实现](../../../raw/current/scripts/components/diseaseable.lua#L154-L166)）。 |
| `Spread()` / `RestartNearbySpread()` | → `nil` | 传染半径内随机一个 diseaseable 发病；后者重排邻近已发病者的传染任务（[实现](../../../raw/current/scripts/components/diseaseable.lua#L169-L193)）。 |
| `OnSave()` / `OnLoad(data)` | 组件存档钩子 | 依次记录 spreadtime/delaytime/warningtime 三态（[实现](../../../raw/current/scripts/components/diseaseable.lua#L195-L227)）。 |

## 典型接入

### 无原版 AddComponent 消费者（源码事实）

- 全库 grep（scripts 根，含 prefab/stategraph/brains）：不存在任何 `AddComponent("diseaseable")` 调用；唯一的构造路径就是组件自身。
- 实际读取者只有三处防御性判空：inspectable 的 DISEASED/DISEASING 描述键（[inspectable](../../../raw/current/scripts/components/inspectable.lua#L40-L42)）、burnable 对植物烧毁的疾病豁免（[burnable](../../../raw/current/scripts/components/burnable.lua#L292)）、workable 对已发病目标的限制（[workable](../../../raw/current/scripts/components/workable.lua#L159)）——这些代码在当前快照运行时不会命中。
- 生效与撤销：因此上述消费链属于"组件存在即生效"的预留接口；组件 TUNING（DISEASE_*）仍在，机制疑似随旧版疾病系统退役，未核验历史版本行为。

```lua
-- 模组自行接入示例（服务端实体；原版无此用法，谨慎参考）
inst:AddComponent("diseaseable")
inst.components.diseaseable:SetDiseasedFn(function(ent) ent:PushEvent("mydiseased") end)
```

- 调整与验证：延迟/预警/传染时长与概率来自 `TUNING.DISEASE_*`（[取值](../../../raw/current/scripts/components/diseaseable.lua#L15-L27)）；接入后需自测 OnSave/OnLoad 三态恢复。

## 常见陷阱与限制

- 构造即 `StartDelay`：挂载瞬间就进入疾病倒计时，不想让某实体参与机制就不要挂组件（[构造](../../../raw/current/scripts/components/diseaseable.lua#L76)）。
- 传染搜索按 `diseaseable` 标签且直接访问邻居的 `_delaytask`/`_spreadtask` 私有字段（[访问](../../../raw/current/scripts/components/diseaseable.lua#L47-L51)），邻居必须真的挂了组件，否则 `components.diseaseable` 为 nil 会报错。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，覆盖组件全文与 inspectable/burnable/workable 读取点；全库 grep 确认无 AddComponent 消费者；未运行游戏。
- 结构检查：validate_page.py、validate_source_links.py 与 Lua 语法探针通过。
