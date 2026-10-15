---
title: "socket_shadow_mimicry"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "WX-78 影子插槽拟真开关：按插槽质量挂上组件后，让容器里的拟物道具（itemmimic）暂停或恢复伪装。"
aliases: ["影子插槽拟真", "WX78拟物控制", "socket_shadow_mimicry"]
tags: [component, lifecycle, callback]
dst_build: "steam-buildid:26230496"
size_class: simple
sources: ["components/socket_shadow_mimicry.lua", "prefabs/wx78_common.lua", "components/itemmimic.lua", "componentutil.lua", "prefabs/wx78_backupbody.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# socket_shadow_mimicry

## 快速判断

- 职责与适用：WX-78 影子套件的拟真协调组件：存在即表示"该机体现在启用拟真"。它自身没有持久状态，作用有两个——扫描宿主容器，给每个拟物道具推 `itemmimic_refreshpausestates` 事件；在 `wx78_backupbody` 上生成 `wx78_mimicspawner` 子实体。
- 优先使用：不要手工 AddComponent；正规入口是 `RefreshShadowSocketBuffs`——按 `socketholder:GetHighestQualitySocketed(SOCKETNAMES.SHADOW)` 查配置，`config.mimic` 为真时加组件、否则移除（[wx78_common.lua#L715-L726](../../../raw/current/scripts/prefabs/wx78_common.lua#L715-L726)）。
- 重要副作用：构造末尾以 `forceadd=true` 全量刷新一次（[L27](../../../raw/current/scripts/components/socket_shadow_mimicry.lua#L27)）；`OnRemoveFromEntity` 移除 mimic spawner 并再刷一遍（forceadd 为空，解除暂停源）（[L30-L36](../../../raw/current/scripts/components/socket_shadow_mimicry.lua#L30-L36)）。
- 修改边界：组件无 OnSave／OnLoad，能否重现完全取决于宿主 prefab 每次构建时的插槽质量计算；`wx78_backupbody` 的 mimic spawner 子实体由构造即时生成（[L19-L25](../../../raw/current/scripts/components/socket_shadow_mimicry.lua#L19-L25)）。

## 关键接口

| 接口／字段 | 已核验类型 | 条件、作用与证据 |
|---|---|---|
| `RefreshMimicPauseStates(forceadd)` | 构造内定义的闭包字段，非方法 | 对 `inventory`／`container` 及 `wx78_backupbody_inventory` 的每个物品推 `itemmimic_refreshpausestates`，数据 `{owner, forceadd}`。[L8-L17](../../../raw/current/scripts/components/socket_shadow_mimicry.lua#L8-L17)、[推送](../../../raw/current/scripts/components/socket_shadow_mimicry.lua#L1-L3) |
| `spawner` | 实体或 nil | 仅 `wx78_backupbody` 宿主：构造生成 `wx78_mimicspawner` 并 `SetParent`，spawner 移除时置 nil。[L19-L25](../../../raw/current/scripts/components/socket_shadow_mimicry.lua#L19-L25)、[prefab 注释](../../../raw/current/scripts/prefabs/wx78_backupbody.lua#L22) |

## 常见陷阱与限制

- 与 itemmimic 是名字耦合：itemmimic 用字符串 `"socket_shadow_mimicry"` 作为暂停源 key 调 `AddPauseSource`／`RemovePauseSource`，源码自注 "HACK coupling"（[itemmimic.lua#L115-L117](../../../raw/current/scripts/components/itemmimic.lua#L115-L117)）。改组件名或绕过组件直接改 pause source 会破坏刷新闭环。
- 拟真判定以"宿主是否带本组件"为准：`componentutil.lua` 里 `user.components.socket_shadow_mimicry == nil` 参与 `userprocsmimics` 判定，`wx78_backupbody_inventory` 会先回溯到父实体（backupbody）再判定（[componentutil.lua#L2352](../../../raw/current/scripts/componentutil.lua#L2352)、[回溯](../../../raw/current/scripts/componentutil.lua#L2345-L2350)）。
- 事件是推给容器内每个物品的：接收方是 itemmimic 组件；容器扫描不含背包装备以外的递归容器。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。完整阅读 components/socket_shadow_mimicry.lua（38 行），核对 wx78_common 的插槽质量分支、itemmimic 暂停源耦合、componentutil 的拟真判定与 wx78_backupbody 注释。未覆盖拟物道具实际表现。
- 游戏运行：未运行。
- 结构检查：validate_page.py 通过（0 issue；43 行、4191 字节，simple 档内）；validate_source_links.py --strict 通过（11 处源码引用，0 error／0 warning）；本页无完整 Lua 代码块。
