---
title: "gingerbreadhunter"
created: "2026-10-28"
updated: "2026-10-28"
type: component
size_class: simple
summary: "冬日盛宴姜饼猎捕世界组件：每 3 天在玩家远处放姜饼猪，跟随面包屑追踪多轮后结算姜饼屋或姜饼狼犬。"
aliases: ["姜饼猎捕", "gingerbreadhunter", "姜饼猪", "面包屑追踪"]
tags: [component]
dst_build: "steam-buildid:26230496"
sources: ["components/gingerbreadhunter.lua", "prefabs/forest.lua", "prefabs/gingerbreadpig.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# gingerbreadhunter

## 快速判断

- 职责与适用：地表世界组件（仅 `WINTERS_FEAST` 特殊事件激活时挂载，[挂载](../../../raw/current/scripts/prefabs/forest.lua#L684-L686)），按天数周期开启"猎捕"：在玩家 40+5 单位外、且距任意玩家 40 单位外生成 `gingerbreadpig`。
- 优先使用：组件自驱；姜饼猪死亡时由猪 prefab 反向调用 `GenerateCrumbPoints`+`SpawnCrumbTrail` 续链（[猪侧](../../../raw/current/scripts/prefabs/gingerbreadpig.lua#L64-L66)）。
- 重要副作用：生成的猪带 `killtask`（限时 1.5 天自动杀）与 `leash_target` 指向被选玩家（[生成](../../../raw/current/scripts/components/gingerbreadhunter.lua#L118-L122)）。
- 修改边界：每轮猎捕从 `availableplayers` 中移除成功者，轮空后重置候选池（[轮换](../../../raw/current/scripts/components/gingerbreadhunter.lua#L133-L143)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `StartNewHunt()` | → `nil` | 随机选玩家、最多 20 次尝试找合法出生点，生成带自杀任务的姜饼猪（[实现](../../../raw/current/scripts/components/gingerbreadhunter.lua#L98-L144)）。 |
| `GenerateCrumbPoints(origin_pt, amount)` | → `bool` | 从起点链式取 CRUMB_DISTANCE=15 间距的点存入 `crumb_pts`；失败返回 false；成功递增 `hunt_count`（[实现](../../../raw/current/scripts/components/gingerbreadhunter.lua#L146-L171)）。 |
| `SpawnCrumbTrail(killtime, player)` | killtime: 秒 | 沿面包屑点放 `crumbs`；末点按 `hunt_count > MAX_HUNT_COUNT(3)` 决定姜饼屋（2~5 座，20% 幸运骰换成姜饼狼犬）或再放一只被追猪（[实现](../../../raw/current/scripts/components/gingerbreadhunter.lua#L173-L218)）。 |
| `OnSave()` / `Load(data)` | 组件存档钩子 | 仅保存 `hunt_count` 与 `days`（[实现](../../../raw/current/scripts/components/gingerbreadhunter.lua#L220-L235)）。 |

## 典型接入

### 事件驱动与玩家轮换

- 前提与选择：需要特殊事件系统（`IsSpecialEventActive`）；`ms_playerjoined`/`ms_playerleft` 监听在 TheWorld 上，`cycles` 世界状态驱动天数（[监听](../../../raw/current/scripts/components/gingerbreadhunter.lua#L27-L29)）。
- 生效与撤销：每 3 天（`SPAWN_PERIOD`）在半天内随机时刻开猎（[周期](../../../raw/current/scripts/components/gingerbreadhunter.lua#L33-L45)）；`availableplayers` 保证玩家不被连续选中。
- 原版依据：猪被打死时以尸体位置为起点生成 5 个面包屑点，剩余存活时间作为下一只猪的 killtime（[猪侧](../../../raw/current/scripts/prefabs/gingerbreadpig.lua#L64-L66)）；末点奖励经 `TryLuckRoll` 幸运系统（[结算](../../../raw/current/scripts/components/gingerbreadhunter.lua#L186-L205)）。

```lua
-- 服务端 TheWorld 上（冬日盛宴期间）：
local h = TheWorld.components.gingerbreadhunter
if h ~= nil then
    local ok = h:GenerateCrumbPoints(somepos, 5)
    if ok then h:SpawnCrumbTrail(1.5 * TUNING.TOTAL_DAY_TIME, player) end
end
```

- 调整与验证：出生点距离与夹角偏差复用 `TUNING.TRACK_ANGLE_DEVIATION`（[角度](../../../raw/current/scripts/components/gingerbreadhunter.lua#L75-L84)）；`LoadPostPass` 为空实现，实体引用不存档（[空实现](../../../raw/current/scripts/components/gingerbreadhunter.lua#L237-L238)）。

## 常见陷阱与限制

- 出生点条件是"远离所有玩家"：`IsAnyPlayerInRange(spawn_pt, 40)` 不满足即重试 20 次，密集玩家群会导致 `Hunt failed`（[尝试](../../../raw/current/scripts/components/gingerbreadhunter.lua#L114-L130)）。
- 面包屑链每点以新点为原点继续，若中途 20 次尝试失败整轮 `GenerateCrumbPoints` 返回 false 且不清理已生成点（[失败分支](../../../raw/current/scripts/components/gingerbreadhunter.lua#L160-L164)）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，覆盖组件全文、forest 事件挂载、gingerbreadpig 反向调用；未运行游戏。
- 结构检查：validate_page.py、validate_source_links.py 与 Lua 语法探针通过。
