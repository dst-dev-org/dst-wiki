---
title: "discoverable"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "给带小地图图标的实体提供点击后从未知图标切换为已发现图标的轻量状态切换，原版未使用。"
aliases: ["discoverable", "地图发现", "小地图图标切换", "可发现状态"]
tags: [component, ui]
dst_build: "steam-buildid:26230496"
size_class: simple
sources: ["components/discoverable.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# discoverable

## 快速判断

- 职责与适用：维护 `discovered` 布尔态，在两个小地图图标间切换；宿主需自带 `MiniMapEntity`。([discoverable.lua:17-29](../../../raw/current/scripts/components/discoverable.lua#L17-L29)
- 优先使用：`SetIcons(未发现图标, 已发现图标)` 配置并立即隐藏，点击地图事件自动发现。([discoverable.lua:31-36](../../../raw/current/scripts/components/discoverable.lua#L31-L36)
- 重要副作用：`SetIcons` 结尾调 `Hide()`，会立即覆盖 `MiniMapEntity` 当前图标为未发现图标。[discoverable.lua:35](../../../raw/current/scripts/components/discoverable.lua#L35
- 修改边界：**无原版 AddComponent 消费者**——全库检索 `discoverable` 仅命中组件自身文件；`onclick` 事件在脚本层也没有 Lua 推送者（引擎地图点击行为待核验）。([discoverable.lua:8-14](../../../raw/current/scripts/components/discoverable.lua#L8-L14)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `discovered` | 布尔，默认 false | 点击发现后置 true；仅由本组件读写。[discoverable.lua:3](../../../raw/current/scripts/components/discoverable.lua#L3 [discoverable.lua:18](../../../raw/current/scripts/components/discoverable.lua#L18 |
| `SetIcons(undiscovered, discovered)` | 图标名（.tex）字符串对 | 存两个图标并立即 `Hide()`。([discoverable.lua:31-36](../../../raw/current/scripts/components/discoverable.lua#L31-L36) |
| `Discover()` / `Hide()` | 无参 | 分别把图标切为已发现／未发现并翻转状态；需 `MiniMapEntity`，否则访问 nil 报错。([discoverable.lua:17-29](../../../raw/current/scripts/components/discoverable.lua#L17-L29) |
| `OnSave()` / `OnLoad(data)` | 保存 `{discovered=true}` 或空表 | OnLoad 读 `data.discovered` 才重新发现；注意 OnLoad 无 data nil 守卫。([discoverable.lua:38-52](../../../raw/current/scripts/components/discoverable.lua#L38-L52) |

## 典型接入

### 地图实体点击后发现

- 前提与选择：实体需在 pristine 阶段添加 `MiniMapEntity` 并设置初始图标；组件监听实体自身的 `onclick` 事件，首次触发即发现。([discoverable.lua:8-14](../../../raw/current/scripts/components/discoverable.lua#L8-L14)
- 生效与撤销：发现态随实体存档；`Hide()` 可手动复位图标与状态。

```lua
inst.MiniMapEntity:SetIcon("myruin_undiscovered.tex")
inst:AddComponent("discoverable")
inst.components.discoverable:SetIcons("myruin_undiscovered.tex", "myruin.tex")
```

- 调整与验证：图标名须为有效小地图图标资源；点击行为依赖引擎推送 `onclick`，脚本层未闭环。

## 常见陷阱与限制

- 无原版消费者：组件行为链（谁点击、点击是否到达实体）未被原版内容使用过，直接照搬需自行验证引擎事件。
- `OnSave` 恒返回表（含空表），`OnLoad` 直接索引 `data.discovered`；给该组件伪造 nil 存档会报错。([discoverable.lua:38-52](../../../raw/current/scripts/components/discoverable.lua#L38-L52)
- 图标切换只改 `MiniMapEntity`，不隐藏实体本体，也没有发现回调可挂。

## 验证记录

- 源码审阅：2026-10-28，`steam-buildid:26230496`：审阅组件全文；全库检索确认无 AddComponent／读取消费者、脚本内无 `onclick` 推送者。
- 游戏运行：未运行；`onclick` 的引擎侧来源未核验。
- 结构检查：validate_page.py 通过；validate_source_links.py --strict 通过（本批 16 页合并执行）。
