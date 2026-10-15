---
title: "skeletonsweeper"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "世界玩家骷髅上限管理组件：监听 ms_skeletonspawn 收集骷髅，超过 TUNING.MAX_PLAYER_SKELETONS 时按生成时间从旧到新清除或腐坏。"
aliases: ["玩家骷髅上限", "骷髅清理", "skeletonsweeper"]
tags: [component, lifecycle]
dst_build: "steam-buildid:26230496"
size_class: simple
sources: ["components/skeletonsweeper.lua", "prefabs/world.lua", "prefabs/skeleton.lua", "prefabs/shallow_grave.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# skeletonsweeper

## 快速判断

- 职责与适用：挂世界（prefabs/world.lua:602），仅服务器（assert，components/skeletonsweeper.lua:7）；监听 `ms_skeletonspawn`（玩家骷髅 prefabs/skeleton.lua:227、浅坟 prefabs/shallow_grave.lua:193 推送）维护骷髅列表。
- 优先使用：入口只有 `Sweep(max_to_keep, no_decay)`；原版无组件外调用者，触发点为新增骷髅、启用事件与 OnPostInit。
- 重要副作用：每个骷髅注册 onremove 监听自动出列（:33-40、:44）；清扫时反注册再处理（:96）。
- 修改边界：postinit 前不清扫（`_nosweep`，:18、:73-79）；`ms_enableskeletonsweeper` 可整体开关（:52-60、:67），全库当前无推送者，只能由 mod 触发。

## 关键接口

| 接口/字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `Sweep` | `(max_to_keep\|nil, no_decay\|nil) -> nil` | 默认上限 `TUNING.MAX_PLAYER_SKELETONS`（100，tuning.lua:3224）；按 `skeletonspawntime` 升序从最旧移除，`no_decay` 或无 `Decay` 方法直接 Remove，否则调 `todecay:Decay()`。:85-104 |
| `OnPostInit` | 世界加载后调用 | 解锁清扫并立即执行一次 `Sweep(nil, true)`（首扫不腐坏直接移除）。:73-79 |
| `_enabled` | 内部布尔 | false 时 Sweep 直接返回；`ms_enableskeletonsweeper` 事件切换。:17、:52-60、:85-88 |

## 典型接入

### 原版参照：玩家骷髅生命周期

- 前提与选择：世界组件自动工作，mod 无需接入；`Decay` 由 skeleton 预制体定义（skeleton.lua:219 `inst.Decay = Player_Decay`），浅坟骷髅走同列表。
- 生效与撤销：超出上限时每次新骷髅入列/启用都会重排序再清扫（:46-49、:55-58）。
- 不提供代码（无已核验的直调需求）。

## 常见陷阱与限制

- 排序键 `skeletonspawntime` 在骷髅构造时取 `GetTime()`（skeleton.lua:218），存档重载后按加载时刻重置——跨会话不保留原始生成先后，清扫顺序以本会话为准。
- 首扫 `no_decay=true`：世界加载超限时旧骷髅直接消失而非播放腐坏（:77、:98-99）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，通读组件全部 110 行，核验挂载（world.lua:602）、两个 ms_skeletonspawn 推送点与 `ms_enableskeletonsweeper` 全库无推送者的事实；上限取值经 tuning.lua:3224 核对。
- 游戏运行：未运行。
