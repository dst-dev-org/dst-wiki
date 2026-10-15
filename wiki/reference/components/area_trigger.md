---
title: "area_trigger"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "区域切入触发的调参覆盖器：监听 changearea 按区域 story 查表执行 tuning_override；当前快照无任何挂载与调用者，属遗留死代码。"
aliases: ["area_trigger", "区域触发器", "区域调参覆盖"]
tags: [component, worldgen]
size_class: simple
dst_build: "steam-buildid:26230496"
sources: ["components/area_trigger.lua", "tuning_override.lua", "components/areaaware.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# area_trigger

## 快速判断

- 职责与适用（设计意图）：挂在实体上监听 `changearea`，按区域数据的 `story`／`story_depth` 字段在 `stories` 表里查覆盖项，逐条执行键对应的 tuning_override 覆盖函数（以配置值作实参）改世界调参。[CheckTrigger](../../../raw/current/scripts/components/area_trigger.lua#L19-L27)
- 优先使用：无可用的原版入口。全库 grep 无 `AddComponent("area_trigger")`、无 `RegisterTriggers` 调用、无 `components.area_trigger` 读取；`tuning_override` 表的全部键也只映射到 `dummyfn` 占位函数。[覆盖表占位](../../../raw/current/scripts/tuning_override.lua#L1-L14)
- 重要副作用：构造即在宿主上注册 changearea 监听——若被挂载会常驻，但当前没有任何代码路径会走到。[监听](../../../raw/current/scripts/components/area_trigger.lua#L5)
- 修改边界：事件数据来自 areaaware 推送的 `current_area_data`（含 id／type／center／poly／tags 字段），当前版本该结构没有 `story`／`story_depth`，即使注册了 stories 也不会命中。[事件数据](../../../raw/current/scripts/components/areaaware.lua#L50-L59)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `RegisterTriggers(stories)` | `stories: {story名 = {{键, 值}, ...}}` | 整表替换 `self.stories`；无调用者。[实现](../../../raw/current/scripts/components/area_trigger.lua#L29-L32) |
| `CheckTrigger(area)` | `area: 需含 story／story_depth 字段` | 分别查 `stories[area.story]` 与 `stories[area.story_depth]` 命中即 DoOverride。[实现](../../../raw/current/scripts/components/area_trigger.lua#L19-L27) |
| `DoOverride(overrides)` | `overrides: {{键, 值}, ...}` | require `tuning_override` 后逐条 `retune[override[1]](override[2])`；键不存在时静默跳过。[实现](../../../raw/current/scripts/components/area_trigger.lua#L8-L16) |

## 常见陷阱与限制

- **把本组件当作现成的区域机制：** 当前快照中它是无消费者的遗留实现，tuning_override 的键全部是 dummyfn——挂载并注册 stories 也不会产生任何世界变化；区域相关的现役机制应查 weather／worldgen 等入口。[覆盖表占位](../../../raw/current/scripts/tuning_override.lua#L1-L14)
- **假设 changearea 数据带 story：** areaaware 的 `current_area_data` 结构没有 story／story_depth 字段，本组件的字段预期来自旧版世界生成数据结构；新代码不能沿用该假设。[事件数据](../../../raw/current/scripts/components/areaaware.lua#L50-L59)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（34 行）与 tuning_override.lua 全表（dummyfn 占位）、areaaware 的事件数据结构；grep 全库确认无 AddComponent／RegisterTriggers／字段读取者。
- 游戏运行：未运行（无可运行路径）。
