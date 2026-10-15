---
title: "hudindicatorwatcher"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "本地玩家的屏外目标 HUD 标记器：扫描世界可追踪实体，增删屏幕边缘指示图标。"
aliases: ["HUD 标记监视器", "屏外图标", "hudindicatorwatcher"]
tags: [component, ui]
dst_build: "steam-buildid:26230496"
sources: ["components/hudindicatorwatcher.lua", "components/hudindicatable.lua", "prefabs/player_common.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# hudindicatorwatcher

## 快速判断

- 职责与适用：装在 `ThePlayer` 活跃组件组（[安装](../../../raw/current/scripts/prefabs/player_common.lua#L722-L730)），逐帧对照世界 `hudindicatablemanager` 登记表与玩家 HUD，给应追踪的屏外目标加 `AddTargetIndicator`（[扫描](../../../raw/current/scripts/components/hudindicatorwatcher.lua#L64-L105)）。
- 优先使用：想让自定义实体显示屏外图标：给实体加 `hudindicatable` 组件并设 `shouldtrackfn` 即可，本组件自动发现。
- 重要副作用：依赖 `inst.HUD`（PlayerHud 的 [接口](../../../raw/current/scripts/screens/playerhud.lua#L1729)），HUD 未激活时 OnUpdate 直接返回（[守卫](../../../raw/current/scripts/components/hudindicatorwatcher.lua#L65-L67)）。
- 修改边界：监听世界的 `playerexited`/`unregister_hudindicatable` 事件即时移除标记（[监听](../../../raw/current/scripts/components/hudindicatorwatcher.lua#L28-L29)）；组件移除时清空全部标记（[清理](../../../raw/current/scripts/components/hudindicatorwatcher.lua#L34-L53)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `offScreenItems` | 实体数组 | 当前已挂标记的目标，组件内部维护。 |
| `ShouldShowIndicator(target)` | → `boolean` | 转发 `target.components.hudindicatable:ShouldTrack(viewer)`（[实现](../../../raw/current/scripts/components/hudindicatorwatcher.lua#L55-L58)）。 |
| `ShouldRemoveIndicator(target)` | → `boolean` | 同上取反（[实现](../../../raw/current/scripts/components/hudindicatorwatcher.lua#L60-L62)）。 |
| `OnUpdate()` | 每帧 | 先清失效标记，再从 `TheWorld.components.hudindicatablemanager.items` 补新标记（[实现](../../../raw/current/scripts/components/hudindicatorwatcher.lua#L64-L105)）。 |

## 关键流程

1. 目标实体安装 `hudindicatable`，0 帧后向世界的 `hudindicatablemanager` 注册（[注册](../../../raw/current/scripts/components/hudindicatable.lua#L3-L5)、[管理器](../../../raw/current/scripts/components/hudindicatablemanager.lua#L44-L47)；管理器仅非 dedicated 世界安装，[安装](../../../raw/current/scripts/prefabs/world.lua#L558-L561)）。
2. 观察者玩家每帧扫描登记表，`ShouldTrack` 通过即 `HUD:AddTargetIndicator`。
3. 目标移除/取消注册时推 `unregister_hudindicatable`，观察者即时摘标（[摘标](../../../raw/current/scripts/components/hudindicatable.lua#L19-L23)、[响应](../../../raw/current/scripts/components/hudindicatorwatcher.lua#L3-L15)）。

## 典型接入

### 让自定义实体显示屏外标记

- 前提与选择：实体常见 master_postinit（服务器）+ 客户端都要有 `hudindicatable`；`shouldtrackfn` 自定义可见条件。
- 生效与撤销：标记随 ShouldTrack 实时增删；实体移除走事件兜底。
- 原版依据：月风暴静态装置、Wagstaff 与玩家自身都可成为标记目标（[月风暴](../../../raw/current/scripts/prefabs/moonstorm_static.lua#L508)、[玩家](../../../raw/current/scripts/prefabs/player_common.lua#L2587-L2589)）。

```lua
-- 目标实体 common_postinit（双端）
inst:AddComponent("hudindicatable")
inst.components.hudindicatable:SetShouldTrackFunction(function(inst, viewer)
    return viewer ~= nil and viewer:HasTag("player") -- 自定义追踪条件
end)
```

- 调整与验证：图标绘制在 PlayerHud 内，样式不可经本组件定制；`onScreenItemsLastTick` 屏内判定已被注释停用（[注释](../../../raw/current/scripts/components/hudindicatorwatcher.lua#L57)、[块](../../../raw/current/scripts/components/hudindicatorwatcher.lua#L97-L102)）。

## 常见陷阱与限制

- `ShouldTrack` 每帧对每个登记项调用：`shouldtrackfn` 内做重查询（FindEntities 等）会放大开销，保持轻量。
- 世界缺 `hudindicatablemanager`（dedicated）时整段扫描跳过，服务器侧组件实例空转不报错（[守卫](../../../raw/current/scripts/components/hudindicatorwatcher.lua#L86)）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，覆盖组件全文、hudindicatable/hudindicatablemanager 协作链与 player_common/moonstorm_static/wagstaff_npc 安装点；未运行游戏。
- 结构检查：validate_page.py、validate_source_links.py 与 Lua 语法探针通过。

相关：[playertargetindicator](playertargetindicator.md)
