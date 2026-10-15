---
title: "hideandseeker"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "给捉迷藏寻找者挂的验证组件：每秒校验游戏有效性与距离，游戏失效自移除并播报，离场提醒归队。"
aliases: ["hideandseeker", "捉迷藏寻找者", "寻找者距离播报", "找猫团游戏"]
tags: [component, ai]
dst_build: "steam-buildid:26230496"
size_class: simple
sources: ["components/hideandseeker.lua", "prefabs/kitcoonden.lua", "components/hideandseekgame.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# hideandseeker

## 快速判断

- 职责与适用：给参与捉迷藏的"寻找者"实体挂周期校验：游戏存在且活跃、寻找者与游戏区域的距离状态；播报依赖宿主 talker。([hideandseeker.lua:3-31](../../../raw/current/scripts/components/hideandseeker.lua#L3-L31)
- 优先使用：`AddComponent` 后立即 `SetGame(gameinst)` 指向带 hideandseekgame 组件的游戏实体（猫团窝与游戏组件都这么做）。([kitcoonden.lua:220-222](../../../raw/current/scripts/prefabs/kitcoonden.lua#L220-L222) ([hideandseekgame.lua:84-86](../../../raw/current/scripts/components/hideandseekgame.lua#L84-L86)
- 重要副作用：游戏无效／不活跃时组件自我移除，并（有 talker 时）说 abort_game_msg；远距／归队时说 hideandseekgame 配置的播报串。([hideandseeker.lua:10-13](../../../raw/current/scripts/components/hideandseeker.lua#L10-L13) ([hideandseeker.lua:23-28](../../../raw/current/scripts/components/hideandseeker.lua#L23-L28)
- 修改边界：每 1 秒 `DoPeriodicTask` 校验；组件移除时取消任务。[hideandseeker.lua:36](../../../raw/current/scripts/components/hideandseeker.lua#L36 ([hideandseeker.lua:39-45](../../../raw/current/scripts/components/hideandseeker.lua#L39-L45)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SetGame(hideandseekgame)` | 游戏实体或 nil | 记录游戏实体；同时取 `game.components.hideandseekgame.gameaborted_announce` 作中止播报。([hideandseeker.lua:47-50](../../../raw/current/scripts/components/hideandseeker.lua#L47-L50) |
| `is_faraway` | 布尔（内部） | 超出 `hiding_range_toofar` 置 true 并播 seeker_too_far_announce；回到 `hiding_range` 内复位并播 seeker_too_far_return_announce。([hideandseeker.lua:15-28](../../../raw/current/scripts/components/hideandseeker.lua#L15-L28) |
| `GetDebugString()` | 字符串 | 输出游戏引用与 is_faraway。([hideandseeker.lua:52-55](../../../raw/current/scripts/components/hideandseeker.lua#L52-L55) |
| （依赖）`hiding_range`／`hiding_range_toofar` | 游戏组件字段，默认 10 | 由 hideandseekgame 维护；距离用 `inst:IsNear(game, 值)` 判定。([hideandseekgame.lua:9-10](../../../raw/current/scripts/components/hideandseekgame.lua#L9-L10) [hideandseeker.lua:16](../../../raw/current/scripts/components/hideandseeker.lua#L16 |

## 典型接入

### 把玩家或生物登记为寻找者

- 前提与选择：game 实体需有 hideandseekgame 组件且处于活跃状态，否则下一秒即被清除；播报可选配 talker。[hideandseekgame.lua:56](../../../raw/current/scripts/components/hideandseekgame.lua#L56 ([hideandseeker.lua:9-13](../../../raw/current/scripts/components/hideandseeker.lua#L9-L13)

```lua
if seeker.components.hideandseeker == nil then
    seeker:AddComponent("hideandseeker")
    seeker.components.hideandseeker:SetGame(gameinst)
end
```

- 生效与撤销：原版在找到藏点时给 finder 挂组件、找到最后一个或游戏结束时移除全部寻找者。([kitcoonden.lua:220-234](../../../raw/current/scripts/prefabs/kitcoonden.lua#L220-L234)

## 常见陷阱与限制

- 组件会"自杀"：Validate 里游戏失效即 `RemoveComponent("hideandseeker")`，外部不要依赖组件常驻。([hideandseeker.lua:10-13](../../../raw/current/scripts/components/hideandseeker.lua#L10-L13)
- 播报串全部来自 hideandseekgame 的字段，未配置（nil）时静默，不报错。[hideandseeker.lua:10](../../../raw/current/scripts/components/hideandseeker.lua#L10 ([hideandseekgame.lua:13-15](../../../raw/current/scripts/components/hideandseekgame.lua#L13-L15)
- 组件只校验与播报，不限制移动、不影响 AI 决策。

## 验证记录

- 源码审阅：2026-10-28，`steam-buildid:26230496`：审阅组件全文、kitcoonden 与 hideandseekgame 两条挂载路径及范围字段。
- 游戏运行：未运行；未复现找猫团完整游戏流程。
- 结构检查：validate_page.py 通过；validate_source_links.py --strict 通过（本批 16 页合并执行）。

## 相关链接

- [[reference/components/hideandseekhidingspot]]：捉迷藏藏身点组件。
