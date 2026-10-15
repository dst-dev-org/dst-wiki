---
title: "beargerspawner"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "森林世界按秋季概率表调度破坏王（Bearger）生成：秋季首日掷骰数量、按剩余天数排计时器、预警阶段锁定目标玩家并按距离播放音效与台词。"
aliases: ["beargerspawner", "破坏王", "bearger", "秋季 boss 生成", "bearger spawner"]
tags: [component, creature, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/beargerspawner.lua", "prefabs/forest.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# beargerspawner

## 快速判断

- 职责与适用：森林世界组件（断言 mastersim），管理破坏王的数量掷骰、计时、预警与生成；配合世界组件 worldsettingstimer 的 `bearger_timetospawn` 计时器，不负责 boss 本体 AI。[安装](../../../raw/current/scripts/prefabs/forest.lua#L656 [断言](../../../raw/current/scripts/components/beargerspawner.lua#L11
- 优先使用：调试/命令强召用 `SummonMonster(player)`（10 秒倒计时）；改秋季概率改 `TUNING.BEARGER_CHANCES` 并重开世界（OnPostInit 按总概率推导基准延迟）。[强召](../../../raw/current/scripts/components/beargerspawner.lua#L421-L429)
- 重要副作用：生成点取目标玩家 `PLAYER_CAMERA_SEE_DISTANCE` 距离的可走点；预警期（最后 60 秒）对目标及其附近玩家播放 `beargerwarning_lvl1-4` 方向音效并说 `ANNOUNCE_DEERCLOPS` 台词（复用独眼鹿字符串）。[生成点](../../../raw/current/scripts/components/beargerspawner.lua#L91-L101) [预警](../../../raw/current/scripts/components/beargerspawner.lua#L274-L294)
- 修改边界：世界已有 `bearger_blocker` 标签实体（突变蜂王守卫类）时不生成；击杀后的冷却与 NO_BOSS_TIME(26 天) 保护叠加；组件状态（含活跃 boss GUID）随世界存档。[阻挡](../../../raw/current/scripts/components/beargerspawner.lua#L128-L132) [存读](../../../raw/current/scripts/components/beargerspawner.lua#L346-L398)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| 生成资格（内部 `CanSpawnBearger`） | → bool | 需同时满足：秋季、`cycles > NO_BOSS_TIME(26)`、`_numSpawned < _numToSpawn`、距上次击杀超过 NO_BOSS_TIME 天。[资格](../../../raw/current/scripts/components/beargerspawner.lua#L54-L59) [常量](../../../raw/current/scripts/tuning.lua#L3041 |
| `OnSeasonTick`（监听 seasontick） | 内部 | 秋季首日（或击杀冷却过/保护期刚过）重掷 `_numToSpawn`：对 `BEARGER_CHANCES={1}` 每项掷 `math.random() < chance` 计数；`_numSpawned` 初始化为当前活跃数。非秋季首日清零并暂停计时器。[掷骰](../../../raw/current/scripts/components/beargerspawner.lua#L159-L183) [概率表](../../../raw/current/scripts/tuning.lua#L6012-L6013) |
| 生成延迟（内部 `SpawnBearger`） | 秒 | `0.25 × remainingdaysinseason × TOTAL_DAY_TIME / _numToSpawn` ± 25% 随机；现有计时器剩余时间超过新上限时会被压短。OnPostInit 的基准值为 `0.25 × autumnlength × TOTAL_DAY_TIME / 总概率`。[排程](../../../raw/current/scripts/components/beargerspawner.lua#L107-L119) [基准](../../../raw/current/scripts/components/beargerspawner.lua#L259-L272) |
| 目标挑选（内部 `PickPlayer`） | — | 从活跃玩家中筛合格者（地表、有 areaaware 区域、不在 nohasslers 标签区），按 easing.inQuint 加权随机选一人；玩家离开后重选，选不出则暂停等下次。[挑选](../../../raw/current/scripts/components/beargerspawner.lua#L61-L89) [资格](../../../raw/current/scripts/components/beargerspawner.lua#L61-L67) |
| `OnUpdate(dt)` | — | 计时器剩余 0～60 秒进入预警：周期播预警音（剩余<30s 加密）与台词（10~14 次间隔说一次）；计时器到点由 worldsettingstimer 回调 `OnBeargerTimerDone` 生成 boss。[预警循环](../../../raw/current/scripts/components/beargerspawner.lua#L297-L336) [生成回调](../../../raw/current/scripts/components/beargerspawner.lua#L234-L245) |
| `ReleaseHassler(targetPlayer)`（内部） | → boss 或 nil | 先查 `bearger_blocker`；取生成点后 SpawnPrefab("bearger") 传送，计数并按需续排下一只。[生成](../../../raw/current/scripts/components/beargerspawner.lua#L121-L153) |
| `beargerkilled`／`beargerremoved`（世界事件） | 内部 | 击杀：停计时器、记录 `_lastBeargerKillDay`（不回退 _numSpawned，防反复循环）；仅移除：清活跃表。[击杀](../../../raw/current/scripts/components/beargerspawner.lua#L223-L232) [移除](../../../raw/current/scripts/components/beargerspawner.lua#L218-L220) |
| `SummonMonster(player)` | — | 命令入口：已有计时器则压到 10 秒并恢复，否则新开 10 秒计时器；同样走预警与生成流程。[强召](../../../raw/current/scripts/components/beargerspawner.lua#L421-L429) [控制台消费](../../../raw/current/scripts/consolecommands.lua#L1040-L1043) |
| `SetFirstBeargerChance`／`SetSecondBeargerChance` | deprecated 空实现 | 调用无效果，概率只认 `TUNING.BEARGER_CHANCES`。[空方法](../../../raw/current/scripts/components/beargerspawner.lua#L251-L257) |
| `OnSave/OnLoad/LoadPostPass` | — | 存 warning、numToSpawn、lastKillDay、numSpawned 与活跃 boss GUID；OnLoad 后默认暂停，LoadPostPass 恢复 boss 引用并按资格续排；旧字段 `timetospawn` 做迁移压缩。[存读](../../../raw/current/scripts/components/beargerspawner.lua#L346-L398) |

## 典型接入

### 调整破坏王的数量与频率

- 前提与选择：改 `TUNING.BEARGER_CHANCES` 表（每项独立掷骰，如 `{1,1}` 大概率两只）；概率总和决定 OnPostInit 的基准延迟，改表需重开世界生效（或模组手动重跑 OnPostInit 逻辑）。[概率表](../../../raw/current/scripts/tuning.lua#L6012-L6013) [基准推导](../../../raw/current/scripts/components/beargerspawner.lua#L259-L272)
- 生效与撤销：世界设置开关（SPAWN_BEARGER）在 AddTimer 时作为初始启用传入，运行期由 worldsettingstimer 管理；没有运行时启停公开接口，模组可 Pause/Resume 该计时器名 `bearger_timetospawn`。[计时器](../../../raw/current/scripts/components/beargerspawner.lua#L266-L267)
- 原版依据：控制台"召唤 boss"命令直接调 SummonMonster；预警台词复用 DEERCLOPS 字符串是当前实现的既成事实（TODO 注释在源码中注明待换）。[控制台](../../../raw/current/scripts/consolecommands.lua#L1040-L1043) [TODO](../../../raw/current/scripts/components/beargerspawner.lua#L274-L277)

```lua
-- 服务器端：查询下一只破坏王的剩余时间（秒；无计时器返回 nil）
local function GetBeargerTimeLeft()
    local timer = TheWorld.components.worldsettingstimer
    if timer == nil or TheWorld.components.beargerspawner == nil then
        return nil
    end
    return timer:GetTimeLeft("bearger_timetospawn")
end
```

- 调整与验证：计时器名是组件私有常量 `bearger_timetospawn`，跨版本可能变名；运行验证建议配合世界 debug 串（GetDebugString 显示 WAITING/WARNING/SPAWNING 与目标数）。[调试串](../../../raw/current/scripts/components/beargerspawner.lua#L405-L419)

## 常见陷阱与限制

- **改 BEARGER_CHANCES 不重开世界不生效：** 掷骰只在秋季首日等重置点执行，且 `_spawndelay` 基准在 OnPostInit 缓存；运行中改 TUNING 不影响已开世界。[掷骰时机](../../../raw/current/scripts/components/beargerspawner.lua#L159-L176)
- **突变量体（bearger_blocker）会顶掉破坏王：** ReleaseHassler 先查 `FindFirstEntityWithTag("bearger_blocker")`，存在即放弃本次生成且不重排——期间破坏王不会出现。[阻挡](../../../raw/current/scripts/components/beargerspawner.lua#L128-L132)
- **_numSpawned 击杀后不回退：** 击杀只停计时器并记冷却日；同季配额被击杀消耗，不会因 boss 死亡而再补——下一季首日才重掷。[注释确认](../../../raw/current/scripts/components/beargerspawner.lua#L226-L229)
- **无目标玩家则整场预警停摆：** 目标离开或全部不合格时暂停计时器与更新，直到玩家加入触发恢复；暂停是"暂停计时器"而非清零。[玩家离开](../../../raw/current/scripts/components/beargerspawner.lua#L199-L216)
- **旧 setter 与旧存档字段：** SetFirst/SecondBeargerChance 是空壳；旧存档 `timetospawn` 在 OnLoad 迁移后按新基准取 min。[迁移](../../../raw/current/scripts/components/beargerspawner.lua#L269-L271) [空 setter](../../../raw/current/scripts/components/beargerspawner.lua#L251-L257)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取 beargerspawner.lua，核验掷骰、排程、目标挑选、预警音效/台词、生成与阻挡、击杀冷却、存读迁移与 SummonMonster；核对 forest 安装、tuning 常量与 consolecommands 消费。boss 本体 prefab 的 blocker 语义未深入。
- 结构检查：本页通过 validate_page.py 与 validate_source_links.py（严格）；Lua 示例经 parser_probe 语法检查。
- 游戏运行：未运行；概率掷骰分布、预警节奏与阻挡交互未实测。

相关：[[reference/components/deerclopsspawner]]、[[reference/components/moosespawner]]。

## 相关链接

- [[reference/components/deerclopsspawner]]：同代际的冬季 hassler 调度器，结构相近。
- [[reference/components/worldsettingstimer]]：生成计时器的宿主组件。
- [[reference/components/moosespawner]]：春季 hassler 的对应实现。
