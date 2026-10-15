---
title: "brightmarespawner"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "月狂（lunacy）状态下 gestalt（Brightmare 影灵）人口的世界宿主组件：按玩家理智百分比映射档位掷运生成与迁移点，Boss 竞技场内禁生成，击败 Wagstaff 员工后生成翻倍。"
aliases: ["brightmarespawner", "月狂影灵生成器", "gestalt 人口管理", "Brightmare spawner"]
tags: [component, ai, callback]
dst_build: "steam-buildid:26230496"
sources: ["components/brightmarespawner.lua", "prefabs/forest.lua", "prefabs/cave.lua", "prefabs/gestalt.lua", "tuning.lua", "componentutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# brightmarespawner

## 快速判断

- 职责与适用：挂在 forest 与 cave 世界实体的服务端组件（构造断言 mastersim）；监听每名玩家的 `sanitymodechanged`，进入月狂模式即纳入人口统计，按 `TUNING.GESTALT_POPULATION_LEVEL` 档位（理智百分比阈值与上限）周期掷运在玩家附近生成 `gestalt`。[构造与事件](../../../raw/current/scripts/components/brightmarespawner.lua#L209-L216) [档位判定](../../../raw/current/scripts/components/brightmarespawner.lua#L33-L48)
- 优先使用：gestalt prefab 是主要消费者——找归属玩家 `FindBestPlayer(gestalt)`、找迁移点 `FindRelocatePoint(gestalt)`；外部无需直接调用生成。[消费侧](../../../raw/current/scripts/prefabs/gestalt.lua#L38-L60)
- 重要副作用：地图 `IsPointInCharlieBossArena` 区域内既不计档也不选点；已击败 Wagstaff 员工（`shard_wagbossinfo:IsWagbossDefeated`）时生成概率乘 `TUNING.WAGBOSS_DEFEATED_GESTALT_SPAWN_FACTOR`(2)。[竞技场排除](../../../raw/current/scripts/components/brightmarespawner.lua#L34-L36) [翻倍](../../../raw/current/scripts/components/brightmarespawner.lua#L94-L98)
- 修改边界：人口循环以自适应间隔自续（`GESTALT_POP_CHANGE_INTERVAL` 减去等级折算再±方差）；无人处于月狂时任务取消，全靠事件重启；组件无存读钩子。[循环](../../../raw/current/scripts/components/brightmarespawner.lua#L124-L129)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| 玩家登记 | `sanitymodechanged` 监听 | 月狂模式进入 `_players`，退出移除；`ms_playerjoined`／`ms_playerleft` 维护监听；最后一个玩家退出月狂即 `Stop()`。([brightmarespawner.lua:179-203](../../../raw/current/scripts/components/brightmarespawner.lua#L179-L203) |
| `GetTuningLevelForPlayer(player)` | 私有闭包 → 档位 | 竞技场内返回 0；月狂且 `GetPercentWithPenalty() >= GESTALT_MIN_SANITY_TO_SPAWN`(0.25) 时按 `GESTALT_POPULATION_LEVEL` 表取 `MAX_SANITY` 命中档。([brightmarespawner.lua:33-48](../../../raw/current/scripts/components/brightmarespawner.lua#L33-L48) [档位表](../../../raw/current/scripts/tuning.lua#L3965-L3982) |
| `FindGestaltSpawnPtForPlayer(player, wantstomorph)` | 私有闭包 → 点 | 竞技场内 nil；`FindValidPositionByFan` 扇形采样，半径按 `GESTALT_SPAWN_DIST`(14)/`GESTALT_SPAWN_MORPH_DIST`(8) ± 方差，6 距内不能有玩家/幽灵/既有 gestalt。([brightmarespawner.lua:59-79](../../../raw/current/scripts/components/brightmarespawner.lua#L59-L79) |
| 人口循环 | 私有闭包 `UpdatePopulation` | 每玩家统计有效档位；按周围 `GESTALT_POPULATION_DIST`(30) 内 gestalt 数与档位取 `BRIGHTMARE_SPAWN_INC_CHANCES.LOW/MED/HIGH` 概率，经 `TryLuckRoll`（`LuckFormulas.BrightmareSpawn`）生成。([brightmarespawner.lua:93-129](../../../raw/current/scripts/components/brightmarespawner.lua#L93-L129) [公式](../../../raw/current/scripts/componentutil.lua#L2135 |
| `FindBestPlayer(gestalt)` | 返回 player, level | 给定 gestalt 找 `GESTALT_POPULATION_DIST` 内档位 >0 且未超 `MAX_SPAWNS+1` 的最近玩家；gestalt 归属判定用。([brightmarespawner.lua:148-169](../../../raw/current/scripts/components/brightmarespawner.lua#L148-L169) [gestalt.lua:60](../../../raw/current/scripts/prefabs/gestalt.lua#L60 |
| `FindRelocatePoint(gestalt)` | 返回点或 nil | 为迁移中的 gestalt 按其 tracking_target 重新选点（`wantstomorph` 用更近的 morph 距离）；[gestalt.lua:38](../../../raw/current/scripts/prefabs/gestalt.lua#L38 消费。([brightmarespawner.lua:171-173](../../../raw/current/scripts/components/brightmarespawner.lua#L171-L173) |
| 追踪表 | `_gestalts`／`_players` | 生成的 gestalt 登记 `_gestalts[ent]` 并监听 `onremove`；仅用于计数与调试串，不持久化。([brightmarespawner.lua:81-90](../../../raw/current/scripts/components/brightmarespawner.lua#L81-L90) [调试](../../../raw/current/scripts/components/brightmarespawner.lua#L222-L224) |

## 典型接入

### 调整月狂影灵的生成压力

- 前提与选择：覆盖 `TUNING.GESTALT_*`／`BRIGHTMARE_SPAWN_INC_CHANCES`／`GESTALT_POPULATION_LEVEL` 即可整调节奏；组件在 forest 与 cave 都挂载（[forest.lua:662](../../../raw/current/scripts/prefabs/forest.lua#L662)、[cave.lua:522](../../../raw/current/scripts/prefabs/cave.lua#L522），地表洞穴各自运行。
- 生效与撤销：调谐在下次档位/概率读取时生效（除构造捕获项外多为逐次读取）；玩家退出月狂即停止该玩家参与。
- 原版依据：档位与概率结构见 ([tuning.lua:3944-3982](../../../raw/current/scripts/tuning.lua#L3944-L3982)、([tuning.lua:9394](../../../raw/current/scripts/tuning.lua#L9394)。

```lua
-- 在 modmain/tuning override 中（示例：更激进的生成概率档）
-- TUNING.BRIGHTMARE_SPAWN_INC_CHANCES = { LOW = ..., MED = ..., HIGH = ... }
-- 服务端查询某个 gestalt 的迁移点（供自定义脑使用）：
-- local pt = TheWorld.components.brightmarespawner:FindRelocatePoint(gestalt)
-- 服务端查询归属（gestalt 脑内已是这么做的）：
-- local player, level = TheWorld.components.brightmarespawner:FindBestPlayer(gestalt)
```

- 调整与验证：`GetDebugString()` 返回当前追踪的 gestalt 数；概率实际还叠加玩家幸运值（`TryLuckRoll`）与 Wagstaff 击败翻倍。

## 常见陷阱与限制

- 理智用 `GetPercentWithPenalty`：装具/惩罚后的百分比才计入档位（([brightmarespawner.lua:38-39](../../../raw/current/scripts/components/brightmarespawner.lua#L38-L39)），纯 `sanity:GetPercent()` 调试会与实际不符。
- 生成不查瓦片与障碍，只查 6 距内的玩家/幽灵/gestalt（([brightmarespawner.lua:65-68](../../../raw/current/scripts/components/brightmarespawner.lua#L65-L68)），自定义封闭空间可能出现落地在障碍旁的 gestalt。
- 竞技场判定只认 `Map:IsPointInCharlieBossArena`：自定义"禁生成区"需要另行处理，组件没有通用区域开关（([brightmarespawner.lua:34-36](../../../raw/current/scripts/components/brightmarespawner.lua#L34-L36)）。
- 循环自续间隔受档位影响（等级越高间隔越短，下限 `GESTALT_POP_CHANGE_INTERVAL/2`，([brightmarespawner.lua:124-127](../../../raw/current/scripts/components/brightmarespawner.lua#L124-L127)）；`Stop()` 后只有 `sanitymodechanged`／玩家进出事件能重启。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，通读组件全文；核验 forest/cave 双挂载、gestalt.lua 两个消费点、TUNING 档位与概率表、componentutil 的掷运公式。
- 游戏运行：未运行。
- 结构检查：未执行。

相关链接：[[reference/components/sanity]]、[[reference/components/shard_wagbossinfo]]
