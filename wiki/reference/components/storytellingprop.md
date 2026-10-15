---
title: "storytellingprop"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "把营火等物件标记为可讲故事目标，供拥有 storyteller 的角色发起 TELLSTORY 动作；组件自身无方法。"
aliases: ["storytellingprop", "讲故事道具", "营地讲故事目标", "TELLSTORY"]
tags: [component, building]
size_class: simple
dst_build: "steam-buildid:26230496"
sources: ["components/storytellingprop.lua", "prefabs/campfire.lua", "componentactions.lua", "actions.lua", "components/storyteller.lua", "prefabs/walter.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# storytellingprop

## 快速判断

- 职责与适用：维护实体上的 `storytellingprop` 标签，作为 TELLSTORY 动作的目标侧开关；营火、火坑、YOTR 装饰等挂载。[构造](../../../raw/current/scripts/components/storytellingprop.lua#L1-L6)
- 优先使用：目标是"让角色能对该物件讲故事"时，挂本组件（或直接在 pristine 状态加同名校签）即可进入动作判定；讲故事能力由 doer 侧 `storyteller` 组件提供。[目标判定](../../../raw/current/scripts/componentactions.lua#L864-L870)
- 重要副作用：构造时加标签、`OnRemoveFromEntity` 时移除标签；标签推荐在 prefab pristine 阶段显式添加，保证客户端可见（源码注释要求）。[移除](../../../raw/current/scripts/components/storytellingprop.lua#L8-L10)
- 修改边界：组件无字段、无存读、无网络变量；动作真正执行走 doer 的 `storyteller:TellStory(target)`，本组件不参与故事内容。[执行](../../../raw/current/scripts/actions.lua#L1586-L1590)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| 构造 | `Class(function(self, inst))` | 保存 inst 并 `AddTag("storytellingprop")`；源码注释建议 prefab 在 pristine 状态显式加同名标签。[实现](../../../raw/current/scripts/components/storytellingprop.lua#L1-L6) |
| `OnRemoveFromEntity()` | `-> nil` | 移除组件时同步移除标签，保持标签与组件生命周期一致。[实现](../../../raw/current/scripts/components/storytellingprop.lua#L8-L10) |

## 典型接入

### 让自定义物件成为讲故事目标

- 前提与选择：服务端挂载组件；客户端动作判定只读标签，因此 prefab 需在 `SetPristine` 前有 `storytellingprop` 标签（原版组件构造即加，但网络构造发生在 pristine 前，推荐同源码注释直接给 prefab 加标签）。[注释](../../../raw/current/scripts/components/storytellingprop.lua#L4-L5)
- 生效与撤销：doer 有 `storyteller` 标签时右键目标出现 TELLSTORY；可携带营火（`portable_campfire`）时按左右键约定区分。故事执行与打断由 storyteller 组件负责。[判定](../../../raw/current/scripts/componentactions.lua#L864-L870) [storyteller 来源](../../../raw/current/scripts/components/storyteller.lua#L24)
- 原版依据：campfire 在 fueled 配置后挂载本组件；firepit、coldfirepit、coldfire、portablefirepit、cotl_tabernacle、yotr_decor 同构。[campfire](../../../raw/current/scripts/prefabs/campfire.lua#L188) [firepit](../../../raw/current/scripts/prefabs/firepit.lua#L224)

## 常见陷阱与限制

- **以为组件携带讲故事内容或进度：** TELLSTORY 的定义与执行全部在 actions.lua 与 storyteller 组件侧；`components.storytellingprop` 在全源码无方法级消费者，目标侧只有标签被读取。[动作定义](../../../raw/current/scripts/actions.lua#L498) [执行](../../../raw/current/scripts/actions.lua#L1586-L1590)
- **只加组件不加 pristine 标签：** 动作判定用 `inst:HasTag("storytellingprop")`，组件构造虽会加标签，但标签需在网络构造阶段可见才能让客户端列出动作；沿用源码注释的 pristine 写法最稳。[判定](../../../raw/current/scripts/componentactions.lua#L864-L870)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（12 行），核对 7 个原版挂载点、componentactions 目标判定、TELLSTORY 定义与执行、storyteller 组件与 Walter 挂载。未审阅 TellStory 的具体故事流程与任务内容。
- 结构检查：2026-10-28 validate_page.py 通过（storytellingprop.md）；validate_source_links.py --strict 本页全部源码锚点通过；本页无完整 Lua 示例。
- 游戏运行：未运行；TELLSTORY 动作链与自定义目标未实测。
