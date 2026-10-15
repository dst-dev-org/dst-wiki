---
title: "carnivalhostsummon"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "嘉年华主持人召唤标记：构造即加 carnivalhostsummon 标签并提供 SetCanSummon 开关；原版脚本无 AddComponent 消费者。"
aliases: ["嘉年华主持人召唤标记", "carnivalhostsummon"]
tags: [component]
dst_build: "steam-buildid:26230496"
sources: ["components/carnivalhostsummon.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# carnivalhostsummon

## 快速判断

- 职责与适用：构造时给宿主加 `carnivalhostsummon` 标签；`SetCanSummon(bool)` 开关同一标签。组件自身无字段、无存读、无其他逻辑。[构造](../../../raw/current/scripts/components/carnivalhostsummon.lua#L2-L5)、[SetCanSummon](../../../raw/current/scripts/components/carnivalhostsummon.lua#L8-L13)
- 优先使用：无——全脚本字面量检索不存在 `AddComponent("carnivalhostsummon")`，`carnivalhostsummon` 标签也没有其他读取者（截至本快照）；加了组件也不会触发任何原版行为。
- 重要副作用：仅标签增删；`SetCanSummon(false)` 只移除标签，不移除组件，构造标签可被再次加回。
- 修改边界：可安全 AddComponent／RemoveComponent；无更新循环、无网络变量。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SetCanSummon(cansummon)` | 参数按真值判断（`if cansummon then`），无返回值 | true 加 `carnivalhostsummon` 标签，否则移除。[方法](../../../raw/current/scripts/components/carnivalhostsummon.lua#L8-L13) |

## 常见陷阱与限制

- **原版无消费者**：本组件与同名标签在 raw/current/scripts 内只出现在组件文件自身；嘉年华主持人的生成走 carnivalevent 组件的 `SpawnCarnivalHost`（活动期 SpawnPrefab("carnival_host")，[生成入口](../../../raw/current/scripts/components/carnivalevent.lua#L55-L75)），不经本组件。
- **调用 SetCanSummon 无可观察效果**：标签没有读取者，开关后无任何原版系统响应；如需"可召唤"语义，需要模组自建消费逻辑或等待后续版本引入读取者。
- **组件文件以空行开头**：Class 定义从第 2 行开始，引用行号时注意偏移（[文件头](../../../raw/current/scripts/components/carnivalhostsummon.lua#L2)）。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；覆盖组件全文，并以字面量检索确认 `carnivalhostsummon`／`carnivalhost` 在全脚本无其他引用（仅 carnivalevent 的主持人生成逻辑与组件无关）。未覆盖：引擎侧行为（如有）、DLC／活动模组、运行验证。
- 结构检查：本批 validate_page、validate_source_links。
- 游戏运行：未运行。
