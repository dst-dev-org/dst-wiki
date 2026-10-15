---
title: "maxlightspawner"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "围绕宿主按圆周等角生成 maxwelllight 光球并登记所有权，含 GUID 级存档恢复；当前无原版 AddComponent 消费者。"
aliases: ["麦克斯韦光球生成器", "光球生成", "maxlightspawner"]
tags: [component, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/maxlightspawner.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# maxlightspawner

## 快速判断

- 职责与适用：给宿主提供"在周围圆周上摆 N 个 `maxwelllight` 光球并跟踪所有权"的能力；全库 grep 核验无任何 `AddComponent("maxlightspawner")` 消费者，属保留/废弃候选代码（[组件](../../../raw/current/scripts/components/maxlightspawner.lua#L1-L9)）。
- 优先使用：想在模组中复用麦克斯韦光球摆位可接本组件；若只想要单个光源，直接 SpawnPrefab 更简单。
- 重要副作用：SpawnLight 立即在世界生成 `maxwelllight` 实体；所有权登记只记 GUID，不自动清理光球。
- 修改边界：保存/载入依赖原版实体存档管线（OnSave 返回 GUID 列表 + LoadPostPass 解析），不能用于动态频繁增删场景。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `maxlights`／`radius`／`angleoffset` | `number`，默认 `2`／`3`／`0` | 圆周光球数量、半径（单位）与起始角（度）。[构造](../../../raw/current/scripts/components/maxlightspawner.lua#L4-L7) |
| `SpawnLight(location)` | `location: Vector3` → `entity` | 生成 `maxwelllight` 到指定点并 TakeOwnership。[实现](../../../raw/current/scripts/components/maxlightspawner.lua#L46-L53) |
| `SpawnAllLights()` | 无参数 | `numlights < maxlights` 时从 angleoffset 起按 `TWOPI/maxlights` 步进绕圈生成；注意每轮固定生成 maxlights 个新光球，不按缺口补齐。[实现](../../../raw/current/scripts/components/maxlightspawner.lua#L55-L67) |
| `TakeOwnership(light)` | `light: entity` | 把外部光球登记进 `self.lights` 并计数。[实现](../../../raw/current/scripts/components/maxlightspawner.lua#L11-L14) |
| `OnSave`／`OnLoad`／`LoadPostPass` | 存读钩子 | OnSave 收集光球 GUID；OnLoad 只做旧 `childid` 字段迁移；实体引用在 LoadPostPass 用 newents 表回填。[实现](../../../raw/current/scripts/components/maxlightspawner.lua#L16-L44) |
| `lightname` | `string`，默认 `"maxwelllight"` | 仅在构造器赋值，组件内无任何读取者。 |

## 常见陷阱与限制

- 无原版消费者：`SpawnAllLights`、`LoadPostPass` 等接口没有任何原版调用方，行为只能以组件自身源码为证；模组使用前应自测（[全库检索结论](../../../raw/current/scripts/components/maxlightspawner.lua#L1)）。
- 存档链路依赖 LoadPostPass：仅 OnLoad 不会恢复 lights（其只处理旧字段迁移），跳过 LoadPostPass 的自定义载入流程会丢所有权（[实现](../../../raw/current/scripts/components/maxlightspawner.lua#L28-L44)）。
- 光球本体移除后组件不会感知（无 onremove 监听），`numlights` 会与场上数量失配；需要精确计数时接入方自行监听。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，覆盖组件全文与全库消费者检索（含 maxwelllight 相关引用核对）；未运行游戏。
- 结构检查：validate_page.py、validate_source_links.py 通过（本页无 Lua 示例）。
