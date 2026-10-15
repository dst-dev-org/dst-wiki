---
title: "riftthralltype"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "在裂隙传送门实体上登记爪牙类型（lunar／shadow 系），供爪牙管理器在生成时读取并分支。"
aliases: ["裂隙爪牙类型", "暗影裂隙类型", "riftthralltype"]
tags: [component, lifecycle]
dst_build: "steam-buildid:26230496"
size_class: simple
sources: ["components/riftthralltype.lua", "components/riftspawner.lua", "components/shadowthrallmanager.lua", "prefabs/lunarrift_portal.lua", "prefabs/shadowrift_portal.lua", "constants.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# riftthralltype

## 快速判断

- 职责与适用：给月／影裂隙传送门记录一个 `thrall_type` 字符串，决定该裂隙会产出哪类爪牙（thrall）；生成时由 riftspawner 随机抽取并写入，爪牙管理器读取后分支。宿主为 `lunarrift_portal` 与 `shadowrift_portal`（[宿主1](../../../raw/current/scripts/prefabs/lunarrift_portal.lua#L682)、[宿主2](../../../raw/current/scripts/prefabs/shadowrift_portal.lua#L437)）。
- 优先使用：读取用 `GetThrallType()`／`IsThrallType(t)`；写入正规入口是 `SetThrallType(t)`，但原版只在裂隙生成瞬间由 `RiftSpawner:SpawnRift` 设置一次。
- 重要副作用：`OnSave`／`OnLoad` 持久化 `thrall_type`，读档后裂隙类型保持（[存](../../../raw/current/scripts/components/riftthralltype.lua#L19-L25)、[读](../../../raw/current/scripts/components/riftthralltype.lua#L27-L31)）。
- 修改边界：组件不触发任何生成，只存字符串；类型值来自 `THRALL_TYPES` 常量表，不要自造字符串与消费方分支对不上。

## 关键接口

| 接口／字段 | 已核验类型 | 条件、作用与证据 |
|---|---|---|
| `thrall_type` | string 或 nil | 构造时不赋值（注释占位），生成裂隙时写入。[构造](../../../raw/current/scripts/components/riftthralltype.lua#L1-L5)、[写入](../../../raw/current/scripts/components/riftspawner.lua#L114-L117) |
| `SetThrallType(new_type)`／`GetThrallType()` | string | 直读直写；无校验。[L7-L13](../../../raw/current/scripts/components/riftthralltype.lua#L7-L13) |
| `IsThrallType(check_type)` | boolean | 等值比较。[L15-L17](../../../raw/current/scripts/components/riftthralltype.lua#L15-L17) |
| `GetDebugString()` | string | 返回类型或 "NONE"。[L33-L35](../../../raw/current/scripts/components/riftthralltype.lua#L33-L35) |

## 关键流程

1. `RiftSpawner:SpawnRift` 生成裂隙实体时，从 `rift_def.ThrallTypes` 随机取一项写入：月裂隙定义给 `THRALL_TYPES.LUNAR`（只有 `"plant"`），影裂隙给 `THRALL_TYPES.SHADOW`（`"trio"`／`"mouth"`）([riftspawner.lua#L111-L117](../../../raw/current/scripts/components/riftspawner.lua#L111-L117)、[lunarrift_portal.lua#L749](../../../raw/current/scripts/prefabs/lunarrift_portal.lua#L749)、[shadowrift_portal.lua#L516](../../../raw/current/scripts/prefabs/shadowrift_portal.lua#L516)、[常量表](../../../raw/current/scripts/constants.lua#L2798-L2804))。
2. 爪牙生成时 `shadowthrallmanager` 读 `data.rift.components.riftthralltype:GetThrallType()`，按 `TRIO`／`MOUTH` 分支决定影爪牙构成（[L161-L163](../../../raw/current/scripts/components/shadowthrallmanager.lua#L161-L163)）。

## 常见陷阱与限制

- `rift_def.ThrallTypes` 缺省是 nil（[rift_portal_defs.lua#L27](../../../raw/current/scripts/prefabs/rift_portal_defs.lua#L27) 附近的 fallback 定义）：此时 `riftspawner` 写入 nil，`GetThrallType()` 返回 nil；消费方只对 SHADOW 的 TRIO／MOUTH 做了分支，月爪牙路径不经过该判断。自定义裂隙若想接这套流程，需要同时提供 `ThrallTypes` 与消费侧分支。
- 类型是"裂隙级"而不是"爪牙级"：所有从该裂隙出来的爪牙共用一次随机结果。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。完整阅读 components/riftthralltype.lua（37 行），核对两个裂隙宿主、riftspawner 写入点、shadowthrallmanager 读取分支与 THRALL_TYPES 常量值。未覆盖爪牙实际生成与战斗表现。
- 游戏运行：未运行。
- 结构检查：validate_page.py 通过（0 issue；49 行、4420 字节，simple 档内）；validate_source_links.py --strict 通过（15 处源码引用，0 error／0 warning）；本页无完整 Lua 代码块。
