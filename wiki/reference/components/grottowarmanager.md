---
title: "grottowarmanager"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "洞穴“地底战争”管理器：档案室被破后布防线、推野性梦魇相位，并在玩家进入致月区时按双方数量平衡刷 gestalt 卫兵与梦魇喙妖。"
aliases: ["grottowarmanager", "地底战争", "grotto war", "月魇战场"]
tags: [component, creature, ai, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/grottowarmanager.lua", "prefabs/archive_props.lua", "components/nightmareclock.lua", "prefabs/cave.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# grottowarmanager

## 快速判断

- 职责与适用：洞穴世界主模拟组件（[安装](../../../raw/current/scripts/prefabs/cave.lua#L524)），管理 archives 破关后的 grotto 战场：`ms_archivesbreached` 时一次性布防（fissure_grottowar 裂缝 + nightmaregrowth_spawner 沿入口边线）并把梦魇相位推到 wild；此后按“在 lunacyarea 的玩家”周围平衡刷 `gestalt_guard`（灵体卫兵）与 `nightmarebeak`。[断言](../../../raw/current/scripts/components/grottowarmanager.lua#L7) [开战](../../../raw/current/scripts/components/grottowarmanager.lua#L279-L292) [触发源](../../../raw/current/scripts/prefabs/archive_props.lua#L696)
- 优先使用：查询战争状态用 `IsWarStarted()`；复刻布防用公开方法 `SpawnFrontLines()`（源码注释“only public to allow modders access”）；刷怪节奏改 `TUNING.GROTTOWAR_*`。[查询与布防](../../../raw/current/scripts/components/grottowarmanager.lua#L133-L270) [调参](../../../raw/current/scripts/tuning.lua#L4036-L4040)
- 重要副作用：布防沿拓扑边直接 SpawnPrefab 并清场（LaunchAndClearArea）、记 `war_home`、打 `grotto_war_wall` 标签；刷出的 mare `persists=false`、睡着即删。[布防细节](../../../raw/current/scripts/components/grottowarmanager.lua#L208-L231) [mare 生命周期](../../../raw/current/scripts/components/grottowarmanager.lua#L44-L59)
- 修改边界：战争状态经 OnSave/OnLoad（键 `_enabled2`，源码注明 beta bug 后重命名重置）；玩家跟踪与周期任务不持久化；`nightmarephasechanged` 处理是空 TODO。[存读](../../../raw/current/scripts/components/grottowarmanager.lua#L360-L371) [TODO](../../../raw/current/scripts/components/grottowarmanager.lua#L330-L332)

## 关键接口

| 接口 | 条件、作用与证据 |
|---|---|
| `IsWarStarted()` | 战争是否已开始（存档恢复）。[实现](../../../raw/current/scripts/components/grottowarmanager.lua#L352-L354) [消费](../../../raw/current/scripts/prefabs/archive_props.lua#L693) |
| `SpawnFrontLines()` | 优先走改造世界路径，否则找带 `GrottoWarEntrance` 标签的拓扑节点，向 story_depth 不同的相邻边布防：间距 7 交错放 fissure_grottowar（奇数步）与 nightmaregrowth_spawner（偶数步或 66% 概率），边中点放 grotto_war_sfx。找不到节点只打警告。[实现](../../../raw/current/scripts/components/grottowarmanager.lua#L174-L270) |
| `RetrofittedSpawnFrontLines()` | 旧世界改造：消费 `ms_register_retrofitted_grotterwar_spawnpoint/homepoint` 注册的点，替换为带 war_home 的新生成物。[实现](../../../raw/current/scripts/components/grottowarmanager.lua#L133-L172) [注册](../../../raw/current/scripts/components/grottowarmanager.lua#L349-L350) |
| `ms_archivesbreached`（世界事件） | archive_props 破封时推送；首次开战执行 SpawnFrontLines + `ms_setnightmarephase:"wild"`（[梦魇钟消费](../../../raw/current/scripts/components/nightmareclock.lua#L212)）+ 5 秒后全员震屏。[处理](../../../raw/current/scripts/components/grottowarmanager.lua#L279-L292) |
| `changearea`（玩家事件） | 玩家进入/离开 `lunacyarea` 标签区域切换跟踪表；有人且已开战才启动 1.57 秒周期刷怪。[处理](../../../raw/current/scripts/components/grottowarmanager.lua#L294-L304) [周期](../../../raw/current/scripts/components/grottowarmanager.lua#L117-L121) |

### 战场人口平衡（UpdatePopulation）

对每个处于致月区的玩家：35 半径（`GROTTOWAR_POPULATION_DIST`）内数 `brightmare_guard` 与 `shadow` 标签实体；梦魇概率 =`(1-现有/5)*0.15`（`GROTTOWAR_MAX_NIGHTMARES`）；卫兵概率按“现有 < 梦魇×1.5 → 100%”优先补兵，否则 <2 只（`NUM_AMBIENT_BRIGHTMARES`）时 80%（[平衡式](../../../raw/current/scripts/components/grottowarmanager.lua#L92-L115)、[调参](../../../raw/current/scripts/tuning.lua#L4036-L4040)）。刷点三级回落（28/23/15 半径起试），要求 3 半径内无 brightmare/player/shadow 且不近地洞；nightmarebeak 有 25% 概率直接以玩家为攻击目标（[选点](../../../raw/current/scripts/components/grottowarmanager.lua#L67-L90)、[概率](../../../raw/current/scripts/tuning.lua#L4036)）。

## 典型接入

### 查询战争状态或复刻布防

- 前提与选择：服务端、洞穴世界。判断战场内容是否已激活用 IsWarStarted（原版 archive_props 据此切换表现）；自定义地图想复用布防算法可在世界装组件后调 SpawnFrontLines，但需自备 `GrottoWarEntrance` 拓扑标签，否则只打警告不布防。[节点查找](../../../raw/current/scripts/components/grottowarmanager.lua#L182-L194)
- 生效与撤销：战争只有开始没有结束接口；重置需清 `_enabled2` 存档数据（该键重命名本身就是重置手段）。[存读](../../../raw/current/scripts/components/grottowarmanager.lua#L360-L371)

```lua
-- 服务端：查询地底战争状态（对照原版 archive_props 的用法）
if TheWorld.components.grottowarmanager ~= nil
    and TheWorld.components.grottowarmanager:IsWarStarted() then
    -- 战场已激活，放自己的战场内容
end
```

- 调整与验证：`ms_archivesbreached` 可由脚本直接推送以提前开战（会重复布防吗？——`StartTheWar` 里布防只在 `not _enabled` 分支，二次推送只置位与刷怪，[守卫](../../../raw/current/scripts/components/grottowarmanager.lua#L280-L292)）。

## 常见陷阱与限制

- ** mare 不是持久实体：** 生成的 gestalt_guard/nightmarebeak 一律 `persists=false` 且 entitysleep 即删；想让它们留场需要换用其他生成途径。[生命周期](../../../raw/current/scripts/components/grottowarmanager.lua#L49-L53)
- **刷怪只在致月区玩家附近：** 玩家在 lunacyarea 外（哪怕在战场中）不参与平衡，`_players` 表按区域动态进出；战场表现不要依赖“全员可刷”。[区域门禁](../../../raw/current/scripts/components/grottowarmanager.lua#L294-L304)
- **布防依赖拓扑标签：** `SpawnFrontLines` 找不到 `GrottoWarEntrance` 节点时静默失败（仅 print）；改造世界优先消费注册点，两条路径互斥（retrofit 成功即返回）。[失败路径](../../../raw/current/scripts/components/grottowarmanager.lua#L191-L194) [互斥](../../../raw/current/scripts/components/grottowarmanager.lua#L174-L177)
- **梦魇相位联动只有一次性推送：** 开战把相位推到 wild，此后 `nightmarephasechanged` 处理为空 TODO，“相位加深战争”未实现。[TODO](../../../raw/current/scripts/components/grottowarmanager.lua#L330-L332)
- **卫兵以“第一个梦魇”为目标：** SpawnBrightmareForPlayer 的 target 取当轮找到的 nightmares[1]，无梦魇时卫兵无目标自由行动；这是设计而非 bug。[目标传递](../../../raw/current/scripts/components/grottowarmanager.lua#L110-L113)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取 grottowarmanager.lua，核验开战链路（archives→布防→相位→刷怪）、人口平衡式、选点回落、改造路径与存读；核对 archive_props 触发、nightmareclock 消费与 TUNING。未运行游戏；gestalt_guard/nightmarebeak 个体行为与裂缝生成物细节归各自页面。

相关：[[reference/components/archivemanager]]、[[reference/components/nightmareclock]]。
