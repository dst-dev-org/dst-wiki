---
title: "mermcandidate"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "记录鱼人吃下的卡路里，达到阈值时允许 mermkingmanager 把它变身为鱼人王。"
aliases: ["鱼人王候选", "鱼人卡路里", "mermcandidate"]
tags: [component, lifecycle]
dst_build: "steam-buildid:26230496"
size_class: simple
sources: ["components/mermcandidate.lua", "prefabs/merm.lua", "components/mermkingmanager.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# mermcandidate

## 快速判断

- 职责与适用：鱼人（`merm`）的服务端组件：累计喂食的饥饿值（卡路里），`ShouldTransform()` 报告是否达到变身阈值。是否真的变身由 `mermkingmanager` 结合王座位置等条件决定。
- 优先使用：喂食路径由原版接好——鱼人 `OnEat` 时若是候选就 `AddCalories(data.food)`（[merm.lua#L1162-L1168](../../../raw/current/scripts/prefabs/merm.lua#L1162-L1168)）；管理器在清理候选名单时 `ResetCalories()`（[mermkingmanager.lua#L142-L147](../../../raw/current/scripts/components/mermkingmanager.lua#L142-L147)）。
- 重要副作用：`calories` 与 `transformation_calories` 都入存档（[存](../../../raw/current/scripts/components/mermcandidate.lua#L22-L27)、[读](../../../raw/current/scripts/components/mermcandidate.lua#L29-L37)），读档后进度不丢。
- 修改边界：组件只算数，不触发变身、不移除自身；阈值默认 50，可改。

## 关键接口

| 接口／字段 | 已核验类型 | 条件、作用与证据 |
|---|---|---|
| `calories` | number，默认 0 | 已累计卡路里；`AddCalories` 加上 `edible:GetHunger(self.inst)`（按进食者折算饥饿值）。[L3-L4](../../../raw/current/scripts/components/mermcandidate.lua#L3-L4)、[L7-L12](../../../raw/current/scripts/components/mermcandidate.lua#L7-L12) |
| `transformation_calories` | number，默认 50 | 变身阈值。[L4](../../../raw/current/scripts/components/mermcandidate.lua#L4) |
| `AddCalories(food)` | 无返回 | 有 `food.components.edible` 才累加；不检查是否已是候选（上游 OnEat 负责门控）。[L7-L12](../../../raw/current/scripts/components/mermcandidate.lua#L7-L12) |
| `ShouldTransform()` | boolean | `calories >= transformation_calories`。[L18-L20](../../../raw/current/scripts/components/mermcandidate.lua#L18-L20) |

## 典型接入

### 原版候选变身链

- 组件在 merm prefab 的 master 分支添加（[merm.lua#L1099](../../../raw/current/scripts/prefabs/merm.lua#L1099)）；`mermkingmanager:IsCandidate` 决定吃食是否计入（[merm.lua#L1162-L1168](../../../raw/current/scripts/prefabs/merm.lua#L1162-L1168)）。
- 管理器侧变身条件：候选站在王座上且 `ShouldTransform()` 为真、且当前没有别的候选正在变身（[mermkingmanager.lua#L199-L210](../../../raw/current/scripts/components/mermkingmanager.lua#L199-L210)）；换王／清名单时对所有候选 `ResetCalories()`（[mermkingmanager.lua#L142-L147](../../../raw/current/scripts/components/mermkingmanager.lua#L142-L147)）。

## 常见陷阱与限制

- `AddCalories` 用 `edible:GetHunger(self.inst)` 而不是固定值：鱼人吃不新鲜的、有负面效果的食物同样按饥饿值计入；想排除某类食物要在上游 OnEat 门控。
- 卡路里只在"是候选"时累计（OnEat 有 IsCandidate 门），但组件自身不重置进度：被移出候选名单后残留的 `calories` 由管理器 Reset 清零，绕过管理器直接操作时要自己清理。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。完整阅读 components/mermcandidate.lua（39 行），核对 merm prefab 的添加与 OnEat 链、mermkingmanager 的清理与变身判定。未覆盖变身动画与王后／王共存逻辑。
- 游戏运行：未运行。
- 结构检查：validate_page.py 通过（0 issue；51 行、4063 字节，simple 档内）；validate_source_links.py --strict 通过（13 处源码引用，0 error／0 warning）；本页无完整 Lua 代码块。
