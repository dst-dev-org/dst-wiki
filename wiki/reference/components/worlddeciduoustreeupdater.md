---
title: "worlddeciduoustreeupdater"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "弃用空转组件：forest 世界仍挂载但 update 是空函数，无任何调用者；仅为兼容引用旧组件的模组保留。"
aliases: ["worlddeciduoustreeupdater", "弃用桦树更新器", "世界落叶树更新"]
tags: [component]
size_class: simple
dst_build: "steam-buildid:26230496"
sources: ["components/worlddeciduoustreeupdater.lua", "prefabs/forest.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# worlddeciduoustreeupdater

## 快速判断

- 职责与适用：无实际职责。文件头注明 "This file is deprecated but left here in case it is referenced by mods"，实现只剩 `self.update = function() end` 空函数。[组件全文](../../../raw/current/scripts/components/worlddeciduoustreeupdater.lua#L2-L7)
- 优先使用：无可用入口。forest 世界仍在 master_postinit 挂载它（`inst:AddComponent("worlddeciduoustreeupdater")`），但全库没有任何代码调用其 `update` 字段或注册更新，挂载后为纯空转。[挂载](../../../raw/current/scripts/prefabs/forest.lua#L648)
- 重要副作用：无。构造只保存 `self.inst` 与空 `update` 函数。
- 修改边界：组件归 TheWorld（forest）；不参与存读、无事件、无网络变量；桦树生长的实际实现不在本组件（见 treegrowthsolution、plantregrowth 等现役组件）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `update` | `function() -> 无`（空函数） | 构造时赋空实现，全库无调用者；曾有的落叶树批量更新逻辑已删。[定义](../../../raw/current/scripts/components/worlddeciduoustreeupdater.lua#L4-L7) |

## 常见陷阱与限制

- **以为挂载即有功能：** forest.lua 仍执行 AddComponent，但组件既不 StartUpdatingComponent 也不被外部调用；指望它更新桦树状态的模组会静默失效。[空实现](../../../raw/current/scripts/components/worlddeciduoustreeupdater.lua#L6)
- **从旧快照或旧模组找行为参考：** 该组件历史上承载的落叶树更新已迁移到别的系统；不要按本文件名推断现役机制，应搜索实际管理 deciduoustree 的组件与状态图。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（9 行）与 forest.lua 挂载行，grep 全库确认除挂载行外无任何 `worlddeciduoustreeupdater` 引用（无 update 调用者）。
- 游戏运行：未运行。
