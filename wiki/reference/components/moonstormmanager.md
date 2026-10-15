---
title: "moonstormmanager"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "世界级月风暴剧情组件：锁月相开启风暴区，按节点传播风暴并周期投放瓦斯塔夫猎寻/实验与闪电火花；击败天境守护者后转入无 NPC 实验变体。"
aliases: ["月风暴管理", "瓦斯塔夫猎寻", "moonstormmanager"]
tags: [component, lifecycle]
size_class: complex
dst_build: "steam-buildid:26230496"
sources: ["components/moonstormmanager.lua", "prefabs/moon_altar_link.lua", "prefabs/wagstaff_npc.lua", "components/moonaltarlinktarget.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# moonstormmanager

## 快速判断

- 职责与适用：世界级组件（仅主端），装配于地表世界（[forest](../../../raw/current/scripts/prefabs/forest.lua#L701)）；实际风暴范围由 `world.net` 上的 [[reference/components/moonstorms]] 网络组件承载，本组件负责"何时何地开风暴、刷什么剧情"。三座月坛连成三角触发 `ms_startthemoonstorms`，击败天境守护者触发 `ms_stopthemoonstorms`（[moon_altar_link](../../../raw/current/scripts/prefabs/moon_altar_link.lua#L28)、[alterguardian_phase3dead](../../../raw/current/scripts/prefabs/alterguardian_phase3dead.lua#L93)）。
- 优先使用：模组挂接点以组件回调为主——`AddMetplayer(userid)` 记录已会面、`foundTool()`/`AdvanceWagstaff()` 由 wagstaff_npc 回调（[wagstaff_npc 消费](../../../raw/current/scripts/prefabs/wagstaff_npc.lua#L110)、[L196](../../../raw/current/scripts/prefabs/wagstaff_npc.lua#L196)）。
- 重要副作用：开/停风暴会直接改写昼夜与月相——`ms_setclocksegs {0,0,16}`、`ms_setmoonphase`、锁月相并推 `ms_setmoonphasestyle`（[L195-L213](../../../raw/current/scripts/components/moonstormmanager.lua#L195-L213)）。
- 修改边界：仅主端（构造断言）；无 `OnUpdate`，全部用周期任务与 timer（`moonstorm_experiment_complete` 90s，[TUNING](../../../raw/current/scripts/tuning.lua#L6219)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `StartMoonstorm(first_node_index, nodes)` / `StopCurrentMoonstorm()` | 节点索引可为函数参数 | 从基节点沿拓扑边传播 3 步、上限 10 节点、至少 4 节点（不足换随机基节点重试）；停止时记录 lastnodes 并调 `moonstorms:StopMoonstorm(is_relocating)`（[L318-L413](../../../raw/current/scripts/components/moonstormmanager.lua#L318-L413)、[常量](../../../raw/current/scripts/components/moonstormmanager.lua#L83-L86)）。 |
| `CalcNewMoonstormBaseNodeIndex()` | → 索引或 nil | 随机起点找距上次基节点 >50 的合法节点；排除 `lunacyarea`/`sandstorm` 节点、海洋与上次节点（[L148-L153](../../../raw/current/scripts/components/moonstormmanager.lua#L148-L153)、[L284-L312](../../../raw/current/scripts/components/moonstormmanager.lua#L284-L312)）。 |
| 周期任务三件套 | wagstaff 试探 10s / 火花 30s / 闪电 10~40s | 风暴存续期间启动；结束统一取消（[L383-L385](../../../raw/current/scripts/components/moonstormmanager.lua#L383-L385)、[L392-L403](../../../raw/current/scripts/components/moonstormmanager.lua#L392-L403)）。 |
| `DoTestForWagstaff()` | — | 有玩家在风暴内且无进行中剧情时：wagboss 已击败走 `beginNoWagstaffExperiment`，否则 `beginWagstaffHunt`（[L822-L844](../../../raw/current/scripts/components/moonstormmanager.lua#L822-L844)）。 |
| `beginWagstaffHunt(player)` / `AdvanceWagstaff()` | — | 在风暴内、屏幕外 40 距离选点放 `wagstaff_npc`（带 expire/movetime 计时）；NPC 移动到 `WAGSTAFF_NPC_HUNTS`(4) 次后终点向风暴中心收敛（[L569-L614](../../../raw/current/scripts/components/moonstormmanager.lua#L569-L614)、[选点收敛](../../../raw/current/scripts/components/moonstormmanager.lua#L109-L146)）。 |
| `beginWagstaffDefence()` / `spawnGestaltWave()` | — | NPC 到达实验点后开 90s 计时：周期投 5 件 wagstaff_tool（被拾走回流池）、每隔 7~15s 刷 3~5 只 `bird_mutant`＋1~3 只 `bird_mutant_spitter`（场内 <16 只才刷）（[L653-L786](../../../raw/current/scripts/components/moonstormmanager.lua#L653-L786)、[计时](../../../raw/current/scripts/components/moonstormmanager.lua#L720-L726)）。 |
| `beginNoWagstaffExperiment(player)` / `CapturedRoamer` | — | 无 NPC 变体：风暴内放 `moonstorm_static_roamer`，玩家捕捉（`ms_moonstormstatic_roamer_captured`）后变静态装置再走同套工具/守卫/计时逻辑（[L528-L566](../../../raw/current/scripts/components/moonstormmanager.lua#L528-L566)、[事件源](../../../raw/current/scripts/prefabs/moonstorm_static.lua#L469)）。 |
| `EndExperiment()` / `FailExperiment()` | timer 回调与装置移除 | 成功：NPC 告别语、装置 finished、重新开风暴；装置被移除/死亡即失败清场（[L439-L506](../../../raw/current/scripts/components/moonstormmanager.lua#L439-L506)）。 |
| `DoTestForSparks()` / `DoTestForLightning()` | — | 给风暴内玩家周围补 `moonstorm_spark`（30 距离少于 3 个）、随机落 `moonstorm_lightning`（[L846-L889](../../../raw/current/scripts/components/moonstormmanager.lua#L846-L889)）。 |
| `TestAltarTriangleValid(altars..., center_pt)` / `TestMoonAltarLinkPositionValid(pt)` | → bool | 月坛三角几何校验：两两距离 ≥2.5、夹角点积 ≤0.975、中心点可通行且无防占位物（moon_altar_link 建链与读档都用它，[L891-L942](../../../raw/current/scripts/components/moonstormmanager.lua#L891-L942)、[消费](../../../raw/current/scripts/prefabs/moon_altar_link.lua#L120)）。 |
| `metplayers` / `AddMetplayer(id)` / `GetCelestialChampionsKilled()` | userid → true | 已见过 NPC 的玩家不再当猎寻起点；击败计数决定月相皮肤（[L594-L651](../../../raw/current/scripts/components/moonstormmanager.lua#L594-L651)、[L314-L316](../../../raw/current/scripts/components/moonstormmanager.lua#L314-L316)）。 |
| OnSave/OnLoad | stormdays、节点、metplayers、击败数、moonstyle_altar | 读档 1 秒后按存档节点重启风暴并恢复月相样式（[L950-L1015](../../../raw/current/scripts/components/moonstormmanager.lua#L950-L1015)）。 |

## 关键流程

### 风暴的启停与搬迁

1. 开启（祭坛链建成）：锁满月 + 全夜时钟 → `StartMoonstorm` 选节点传播并喂给 moonstorms 网络组件 → 挂三个周期任务（[L195-L201](../../../raw/current/scripts/components/moonstormmanager.lua#L195-L201)）。
2. 搬迁：每过一天 `stormdays+1`，达 `MOONSTORM_MOVE_TIME`(4) 天后按 0.1→1 递增概率（上限 `MOONSTORM_MOVE_MAX`(6) 天必搬）停旧开新；NPC 若在场会播拒绝台词并消散（[L215-L252](../../../raw/current/scripts/components/moonstormmanager.lua#L215-L252)、[TUNING](../../../raw/current/scripts/tuning.lua#L6221-L6222)）。
3. 关闭（守护者被击败）：击败计数 +1、月相设新月 waxing、解锁月相、停当前风暴；下一次开启的月相样式变为 `glassed_alter_active`（[L203-L213](../../../raw/current/scripts/components/moonstormmanager.lua#L203-L213)、[L190-L193](../../../raw/current/scripts/components/moonstormmanager.lua#L190-L193)）。

### 剧情会话（hunt 与 experiment）

hunt：NPC 刷新 → 玩家靠近进 `beginWagstaffDefence`（90s 实验：工具投放/需求提示/变异鸟波次）→ `foundTool` 恢复计时 → timer 到点 `EndExperiment` 重开风暴。无 NPC 变体把"NPC 位置"换成捕捉到的 `moonstorm_static` 装置，其余共用逻辑（对照 [L653-L673](../../../raw/current/scripts/components/moonstormmanager.lua#L653-L673) 与 [L548-L566](../../../raw/current/scripts/components/moonstormmanager.lua#L548-L566)）。

## 常见陷阱与限制

- **风暴不是天气而是节点集合**：`IsPointInMoonstorm` 判定在 moonstorms 网络组件；组件改节点后表现自动跟随，不要绕过 manager 直接 `AddMoonstormNodes`（[依赖点](../../../raw/current/scripts/components/moonstormmanager.lua#L380-L381)）。
- **剧情互斥**：`DoTestForWagstaff` 要求无 wagstaff／experiment_static／roamer 任一存在；三种载体同时只允许一个（[L824](../../../raw/current/scripts/components/moonstormmanager.lua#L824)）。
- **`ms_moonstormwindowover` 是开局哨**：构造时 0 帧后即推，用于窗口期提示，读档有风暴时会被取消（[L270-L272](../../../raw/current/scripts/components/moonstormmanager.lua#L270-L272)、[L999-L1003](../../../raw/current/scripts/components/moonstormmanager.lua#L999-L1003)）。
- **选点需要屏幕外且无鸟阻挡**：`screencheckfn` 要求 30 距离内无玩家、4 距离内无 `birdblocker`；两次失败放宽到只查风暴区（[L89-L107](../../../raw/current/scripts/components/moonstormmanager.lua#L89-L107)、[L141-L143](../../../raw/current/scripts/components/moonstormmanager.lua#L141-L143)）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。覆盖组件全文、moon_altar_link 触发与校验、wagstaff_npc 回调、moonstorm_static 捕获事件、TUNING 月风暴段、forest 装配行；未覆盖 wagstaff_npc 状态机与 moonstorm_spark/lightning prefab 细节。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、Lua 语法探测通过（见批级记录）。

相关：[[reference/components/moonstorms]]、[[reference/components/moonstormwatcher]]、[[reference/components/moonstormlightningmanager]]。
