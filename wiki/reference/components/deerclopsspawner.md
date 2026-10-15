---
title: "deerclopsspawner"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "森林世界按冬季次数调度独眼鹿（Deerclops）袭击：优先锁定建筑多的玩家、预警阶段提示、支持离场存档（storehassler）与天灾信标（megaflare）召唤。"
aliases: ["deerclopsspawner", "独眼鹿", "deerclops", "冬季 boss 袭击", "deerclops spawner"]
tags: [component, creature, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/deerclopsspawner.lua", "prefabs/forest.lua", "prefabs/deerclops.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# deerclopsspawner

## 快速判断

- 职责与适用：森林世界组件（断言 mastersim），管理独眼鹿袭击的计时、目标挑选、预警与生成；计时器挂 worldsettingstimer 的 `deerclops_timetoattack`。boss 本体 AI 不在此组件。[安装](../../../raw/current/scripts/prefabs/forest.lua#L655 [断言](../../../raw/current/scripts/components/deerclopsspawner.lua#L11
- 优先使用：调试强召 `SummonMonster(player)`；调整频率改 `TUNING.DEERCLOPS_ATTACKS_PER_SEASON`（重开世界生效）。[强召](../../../raw/current/scripts/components/deerclopsspawner.lua#L431-L439)
- 重要副作用：目标玩家按"20 距离内结构数 < 4 优先"挑选，生成点在玩家相机距离外；boss 记住目标玩家 40 距离内最近结构的坐标（`knownlocations:RememberLocation("targetbase")`）。[目标挑选](../../../raw/current/scripts/components/deerclopsspawner.lua#L70-L102) [基地记忆](../../../raw/current/scripts/components/deerclopsspawner.lua#L184-L189)
- 修改边界：boss 在玩家看不见时睡走（WantsToLeave）或季节结束时存档并移除（`storehassler` 世界事件把存档记录暂存在本组件，下次生成还原），而非永久死亡；组件只存 warning 与 storedhassler。[存档 boss](../../../raw/current/scripts/prefabs/deerclops.lua#L160-L177) [存读](../../../raw/current/scripts/components/deerclopsspawner.lua#L377-L401)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| 生成资格（内部 `AllowedToAttack`） | → bool | 有活跃玩家、（正常流程）`cycles > NO_BOSS_TIME(26)`、冬季或 `DEERCLOPS_ATTACKS_OFF_SEASON=true`；megaflare 路径可 skipcycles。[资格](../../../raw/current/scripts/components/deerclopsspawner.lua#L54-L60) [常量](../../../raw/current/scripts/tuning.lua#L6008-L6010) |
| 袭击间隔（内部 OnPostInit） | 秒 | `(winterlength - 1) × TOTAL_DAY_TIME / (DEERCLOPS_ATTACKS_PER_SEASON(4) + 1)`；每季 4 次袭击，最后 1 次错开季节切换。[基准](../../../raw/current/scripts/components/deerclopsspawner.lua#L295-L305) |
| 目标挑选（内部 `PickAttackTarget`） | — | 合格玩家洗牌后，循环找 20 距离内 structure 数量，凑满 `STRUCTURES_PER_SPAWN(4)` 或轮数耗尽取最后一名；找不到目标则暂停袭击。[挑选](../../../raw/current/scripts/components/deerclopsspawner.lua#L70-L102) |
| `OnUpdate(dt)` | — | 剩余 0～60 秒进入预警：`deerclopswarning_lvl1-4` 音效与 `ANNOUNCE_DEERCLOPS` 台词；目标离开/不合格触发 `TargetLost` 重置或延后 61 秒。[预警](../../../raw/current/scripts/components/deerclopsspawner.lua#L328-L367) [丢失目标](../../../raw/current/scripts/components/deerclopsspawner.lua#L136-L150) |
| `ReleaseHassler(targetPlayer)`（内部） | → boss | 世界已有 deerclops 直接复用返回；否则优先还原 `_storedhassler`（SpawnSaveRecord），新 boss 传送后记住目标基地位置。[生成](../../../raw/current/scripts/components/deerclopsspawner.lua#L165-L194) |
| `storehassler`（世界事件监听） | 内部 | deerclops 在想离开（睡眠+WantsToLeave）或非冬季入睡时推送，把 `GetSaveRecord()` 暂存；下次生成优先还原该记录。[暂存](../../../raw/current/scripts/components/deerclopsspawner.lua#L234-L240) [boss 侧](../../../raw/current/scripts/prefabs/deerclops.lua#L160-L177) |
| `hasslerkilled`（世界事件监听） | 内部 | 击杀后重开计时器且延迟 ×6（`HASSLER_KILLED_DELAY_MULT`）。[击杀延迟](../../../raw/current/scripts/components/deerclopsspawner.lua#L243-L246) [倍率](../../../raw/current/scripts/components/deerclopsspawner.lua#L19 |
| `megaflare_detonated`（世界事件监听） | 内部 | 天灾信标引爆点 35 距内有玩家时，按运气公式（`DEERCLOPS_MEGAFLARE_SPAWN_CHANCE=0.6` 基数）5～25 秒后生成，可跳过季节与 NO_BOSS_TIME 限制。[信标路径](../../../raw/current/scripts/components/deerclopsspawner.lua#L261-L277) [概率](../../../raw/current/scripts/tuning.lua#L9345 |
| `OnSeasonChange`（WatchWorldState season） | 内部 | 季节切换时重新评估资格并恢复/暂停袭击与 `watchingcycles` 每日轮询。[季节轮询](../../../raw/current/scripts/components/deerclopsspawner.lua#L116-L134) |
| `SummonMonster(player)` | — | 命令入口：已有计时器压到 10 秒，否则新开；同样走完整预警流程。[强召](../../../raw/current/scripts/components/deerclopsspawner.lua#L431-L439) [控制台消费](../../../raw/current/scripts/consolecommands.lua#L1032-L1035) |
| `SetAttacksPerWinter` 等 | deprecated 空实现 | 三个旧 setter 无效果，只认 TUNING。[空方法](../../../raw/current/scripts/components/deerclopsspawner.lua#L283-L293) |

## 典型接入

### 调整独眼鹿袭击频率或目标偏好

- 前提与选择：频率改 `DEERCLOPS_ATTACKS_PER_SEASON`／`DEERCLOPS_ATTACKS_OFF_SEASON`（重开世界，OnPostInit 缓存延迟）；目标偏好逻辑在 `PickAttackTarget`，模组可 postinit 覆写组件方法实现"优先打基地少的人"。[常量](../../../raw/current/scripts/tuning.lua#L6008-L6010)
- 生效与撤销：运行中不想被袭击可用 worldsettingstimer 对 `deerclops_timetoattack` Pause；已暂存的 storedhassler 会在下次生成还原，清场需同时清 `_storedhassler`（无公开接口，可置 `TheWorld.components.deerclopsspawner._storedhassler = nil`）。[暂存还原](../../../raw/current/scripts/components/deerclopsspawner.lua#L175-L180)
- 原版依据：boss 离场存档协议要求 `ignorebase` 标签为假（调试生成的 boss 不会回存）；组件侧与 boss 侧通过 `storehassler` 世界事件解耦。[协议](../../../raw/current/scripts/prefabs/deerclops.lua#L160-L166)

```lua
-- 服务器端：查询独眼鹿袭击倒计时（秒；无计时器返回 nil）
local function GetDeerclopsTimeToAttack()
    local timer = TheWorld.components.worldsettingstimer
    if timer == nil or TheWorld.components.deerclopsspawner == nil then
        return nil
    end
    return timer:GetTimeLeft("deerclops_timetoattack")
end
```

- 调整与验证：计时器名 `deerclops_timetoattack` 为组件私有常量；GetDebugString 显示 WAITING/WARNING/ATTACKING、当前目标与 stored boss 的 prefab。[调试串](../../../raw/current/scripts/components/deerclopsspawner.lua#L414-L429)

## 常见陷阱与限制

- **kill 延迟 ×6 不是重掷：** 击杀后按 `HASSLER_KILLED_DELAY_MULT=6` 延长下一次间隔，每季 4 次的预算不变；想"杀一只少一次"需要模组自实现。[倍率](../../../raw/current/scripts/components/deerclopsspawner.lua#L119-L120)
- **离场的 boss 还会回来：** 玩家远离或冬季结束时 boss 被存档移除，组件在下次生成时优先还原该记录（含血量等状态）；确认"击杀"要看 hasslerkilled 事件而非实体消失。[离场存档](../../../raw/current/scripts/prefabs/deerclops.lua#L160-L177)
- **无结构世界也会被袭击：** 挑选循环在轮数耗尽后取最后一名合格玩家，"结构少不被选"只是概率倾向而非豁免；`nohasslers` 区域标签才是不被选中的硬条件。[挑选兜底](../../../raw/current/scripts/components/deerclopsspawner.lua#L90-L101) [区域豁免](../../../raw/current/scripts/components/deerclopsspawner.lua#L62-L67)
- **megaflare 绕过一切常规限制：** 信标路径只查运气与附近玩家，可跳过 NO_BOSS_TIME 与非冬季；不想被信标拉仇恨需在 35 距离内不留玩家。[信标分支](../../../raw/current/scripts/components/deerclopsspawner.lua#L261-L277)
- **同世界只存在一只：** ReleaseHassler 先 `FindFirstEntityWithTag("deerclops")`，已有则直接复用不做第二次生成；召唤/自然计时叠加不会产生双 boss。[唯一性](../../../raw/current/scripts/components/deerclopsspawner.lua#L168-L171)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取 deerclopsspawner.lua，核验资格、间隔、目标挑选、预警、生成与存档还原、击杀延迟、信标路径与存读；核对 forest 安装、deerclops 的 storehassler 协议与 tuning 常量。未审校 boss 状态机 WantsToLeave 的全部触发条件。
- 结构检查：本页通过 validate_page.py 与 validate_source_links.py（严格）；Lua 示例经 parser_probe 语法检查。
- 游戏运行：未运行；袭击节奏、离场还原与信标召唤未实测。

相关：[[reference/components/beargerspawner]]、[[reference/components/moosespawner]]。

## 相关链接

- [[reference/components/beargerspawner]]：秋季破坏王调度器，事件协议与预警结构同源。
- [[reference/components/worldsettingstimer]]：袭击计时器的宿主组件。
- [[reference/components/moosespawner]]：春季驼鹿调度器。
