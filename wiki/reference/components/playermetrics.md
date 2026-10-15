---
title: "playermetrics"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "玩家侧指标上报组件：监听解锁配方事件并把『角色首次原型化』作为匿名指标事件发送，供遥测统计。"
aliases: ["玩家指标", "playermetrics", "角色原型化上报"]
tags: [component, callback, data]
size_class: simple
dst_build: "steam-buildid:26230496"
sources: ["components/playermetrics.lua", "prefabs/player_common.lua", "stats.lua", "components/builder.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# playermetrics

## 快速判断

- 职责与适用：挂载后自动监听宿主玩家实体上的 `unlockrecipe` 事件，把 `data.recipe` 转成指标事件 `character.prototyped` 上报；其余玩法无感知。[监听](../../../raw/current/scripts/components/playermetrics.lua#L7-L17)
- 优先使用：不要手动挂载。原版在 [OnSetOwner](../../../raw/current/scripts/prefabs/player_common.lua#L1023-L1026) 中对 `ThePlayer` 或主机侧玩家加入，[RemovePlayerComponents](../../../raw/current/scripts/prefabs/player_common.lua#L996) 中移除。
- 重要副作用：上报走 [Stats.PushMetricsEvent](../../../raw/current/scripts/stats.lua#L298-L313)，最终 `TheSim:SendProfileStats` 编码 JSON 发送；组件不改变任何游戏状态。
- 修改边界：仅监听 `unlockrecipe`（builder 在真实解锁配方时带 `recipe` 数据推送，[builder.lua:471](../../../raw/current/scripts/components/builder.lua#L471）；无 OnSave/OnLoad，无客户端表现。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| 构造 | `Class(function(self, inst))` | `inst:ListenForEvent("unlockrecipe", OnUnlockRecipe)`；无其他字段或配置。[实现](../../../raw/current/scripts/components/playermetrics.lua#L13-L17) |
| `OnUnlockRecipe`（内部） | `data: {recipe: string}` | 仅 `data.recipe ~= nil` 时上报 `PushEvent("character.prototyped", inst, { prefab = data.recipe })`。[实现](../../../raw/current/scripts/components/playermetrics.lua#L7-L11) |
| `OnRemoveFromEntity()` | `-> nil` | 移除 `unlockrecipe` 回调，防悬挂监听。[实现](../../../raw/current/scripts/components/playermetrics.lua#L19-L21) |

## 典型接入

### 按原版方式接入自定义玩家指标（原版参照）

- 前提与选择：服务端或单机侧玩家实体；不经过组件也能直接调 `Stats.PushMetricsEvent(id, player, values)`——原版多个组件（beefalometrics、constructionsite、crittertraits 等）即绕过本组件直调。[直调示例](../../../raw/current/scripts/components/beefalometrics.lua#L7)
- 生效与撤销：组件只负责 `character.prototyped` 这一条事件；卸载后不再监听，但其他直调上报不受影响。
- 原版依据：事件名与负载是遥测约定，改动负载键名只影响统计侧，不影响游戏逻辑。

```lua
-- 服务端：与组件等价的直调上报（无需挂组件）
Stats.PushMetricsEvent("character.prototyped", player, { prefab = "custom_recipe" })
```

- 调整与验证：`values` 表的键会平铺进上报 JSON（[组装逻辑](../../../raw/current/scripts/stats.lua#L300-L307)）；键名冲突时后写覆盖，自定义键应避开 `event` 等保留键。

## 常见陷阱与限制

- **期待组件统计更多玩家行为：** 当前实现只有 `unlockrecipe` 一条监听；行为统计散落在各组件与界面的 `Stats.PushMetricsEvent` 直调里，本组件不是玩家行为的统一入口。[单一职责](../../../raw/current/scripts/components/playermetrics.lua#L1-L23)
- **客户端模组挂载：** `AddComponent` 位于 `TheWorld.ismastersim` 分支或 `inst == ThePlayer` 的 OnSetOwner 路径（([player_common.lua:1020-1026](../../../raw/current/scripts/prefabs/player_common.lua#L1020-L1026)）；纯客户端实体上挂载不会有宿主推送 `unlockrecipe`，组件空转。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（23 行），核对 player_common.lua 加卸位置、stats.lua 上报链路与 builder.lua 事件推送；grep 确认 `PushMetricsEvent` 的直调组件不经本组件。
- 结构检查：2026-10-28 validate_page.py 通过（本页）；validate_source_links.py --strict 通过；Lua 示例经 parser_probe --syntax-only 解析通过。
- 游戏运行：未运行；上报是否被服务端接受未实测。

相关：[[reference/components/beefalometrics]]、[[reference/components/builder]]
