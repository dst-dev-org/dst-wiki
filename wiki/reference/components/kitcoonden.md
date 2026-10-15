---
title: "kitcoonden"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "小猫狸窝收容账本：记录窝内 kitcoon 实体与数量，onremove 自动减员并转发增减回调。"
aliases: ["小猫狸窝", "kitcoonden", "猫狸窝收容"]
tags: [component]
dst_build: "steam-buildid:26230496"
sources: ["components/kitcoonden.lua", "prefabs/kitcoonden.lua", "actions.lua", "componentactions.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# kitcoonden

## 快速判断

- 职责与适用：挂在窝实体上；`kitcoons` 表（键为实体）+ `num_kitcoons` 计数，每只 kitcoon 挂 onremove 监听自动减员（[组件全文](../../../raw/current/scripts/components/kitcoonden.lua#L1-L20)）。
- 优先使用：收编走 `AddKitcoon(kitcoon, doer)`——原版由 RETURN_FOLLOWER 动作在玩家靠近时调用（[动作](../../../raw/current/scripts/actions.lua#L4250-L4260)）；宿主挂 `OnAddKitcoon`/`OnRemoveKitcoon` 接收增减（[宿主](../../../raw/current/scripts/prefabs/kitcoonden.lua#L397-L399)）。
- 重要副作用：宿主 onbuilt 时自动收编附近无领队的 kitcoon（[自动收编](../../../raw/current/scripts/prefabs/kitcoonden.lua#L35-L47)）；playerprox 接近/离开给玩家加/摘 `near_kitcoonden` 标签，是动作候选前提（[标签](../../../raw/current/scripts/prefabs/kitcoonden.lua#L50-L55)）。
- 修改边界：`OnRemoveFromEntity` 等价于清空全部 kitcoon（逐个走同一减员路径，[清空](../../../raw/current/scripts/components/kitcoonden.lua#L22-L26)）；计数与表由组件独占维护。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `AddKitcoon(kitcoon, doer)` | `kitcoon: entity；doer: entity\|nil -> 无` | 幂等（已在表内不重复）；加监听、计数加一并调 `OnAddKitcoon(inst, kitcoon, doer)`。[实现](../../../raw/current/scripts/components/kitcoonden.lua#L28-L37) |
| `RemoveKitcoon(kitcoon)` | `kitcoon: entity -> 无` | 等价直接调内部减员函数；触发 `OnRemoveKitcoon(inst, kitcoon)`。[实现](../../../raw/current/scripts/components/kitcoonden.lua#L39-L41) |
| `RemoveAllKitcoons()` | `-> 无` | 遍历清空；宿主被烧毁时调用。[实现](../../../raw/current/scripts/components/kitcoonden.lua#L43-L47)、[烧毁](../../../raw/current/scripts/prefabs/kitcoonden.lua#L109) |
| `num_kitcoons` / `kitcoons` | `number` / `table` | 仅供读取（debug 串也用计数，[GetDebugString](../../../raw/current/scripts/components/kitcoonden.lua#L49-L51)）；原版按数量判定满员与空窝（[满员](../../../raw/current/scripts/prefabs/kitcoonden.lua#L77)、[空窝](../../../raw/current/scripts/prefabs/kitcoonden.lua#L95)）。 |

## 典型接入

### 收编回调与数量判定（原版参照）

- 前提与选择：组件在宿主内接线两个回调；玩家侧动作链要求玩家带 `near_kitcoonden`（靠近触发）且目标 kitcoon 是其 follower（[候选](../../../raw/current/scripts/componentactions.lua#L506-L510)）。
- 生效与撤销：kitcoon 实体任何方式被移除都自动减员；窝实体移除时组件清理回调逐只注销监听（[减员路径](../../../raw/current/scripts/components/kitcoonden.lua#L9-L18)）。

```lua
-- 宿主侧接线（kitcoonden.lua 的原版用法）
inst:AddComponent("kitcoonden")
inst.components.kitcoonden.OnAddKitcoon = function(inst, kitcoon, doer) end
inst.components.kitcoonden.OnRemoveKitcoon = function(inst, kitcoon) end
```

- 调整与验证：`doer` 仅在动作收编路径有值（自动收编/读档为 nil）；满员数量判定在宿主（NUM_BASIC_KITCOONS），组件不限制上限。

## 常见陷阱与限制

- **组件只记账不管行为**：kitcoon 入窝后的跟随、产出、奖励都在宿主与 kitcoon prefab 内；改收编条件（距离、领队检查）要动 actions/componentactions，不是本组件。
- **回调字段按约定存在**：`OnAddKitcoon`/`OnRemoveKitcoon` 构造中被注释声明（[注释](../../../raw/current/scripts/components/kitcoonden.lua#L6-L7)），以真值判断调用；不设也不报错，但收编无反馈。
- **宿主依赖玩家接近标签**：RETURN_FOLLOWER 候选要求玩家带 `near_kitcoonden`，该标签由宿主 playerprox 管理；自定义窝复用组件时要自建这套接近逻辑，组件不提供。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；覆盖组件全文、kitcoonden 宿主的自动收编/接近标签/满员空窝判定、RETURN_FOLLOWER 候选与执行。
- 结构检查：本批 validate_page、validate_source_links 及示例 lua 语法探测（见批级记录）。
- 游戏运行：未运行。

相关：[[reference/components/kitcoon]]
