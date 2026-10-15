---
title: "riftspawner"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "世界级裂隙生成器：按难度模式与 worldsettingstimer 计时生成月/暗影裂隙 portal，维护裂隙池、亲和查询与启用设置事件。"
aliases: ["riftspawner", "裂隙", "rift", "月裂隙", "暗影裂隙", "rift portal"]
tags: [component, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/riftspawner.lua", "prefabs/cave.lua", "prefabs/forest.lua", "prefabs/rift_portal_defs.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# riftspawner

## 快速判断

- 职责与适用：世界组件（cave 与 forest 都安装），按 `RIFTPORTAL_DEFS`（prefabs/rift_portal_defs）选 prefab 并在世界选点生成裂隙实体；管理裂隙池（`rifts`）与启用状态，不实现裂隙本体成长/掉落。[安装](../../../raw/current/scripts/prefabs/cave.lua#L568-L569) [森林安装](../../../raw/current/scripts/prefabs/forest.lua#L705
- 优先使用：查询裂隙用 `GetRifts()`／`GetRiftsOfAffinity(affinity)`／`IsShadowPortalActive()`；命令生成用 `SpawnRift(forced_pos)`（控制台/调试路径）；改难度走 `rifts_setdifficulty` 等世界事件。[查询](../../../raw/current/scripts/components/riftspawner.lua#L257-L323)
- 重要副作用：生成裂隙即 `AddRiftToPool` 并推世界事件 `ms_riftaddedtopool`（shadowparasitemanager 等以此为波次信号）；启用月裂隙会同时 `wagpunk_manager:Enable()` 并置 `LUNAR_RIFTS_ACTIVE`/`SHADOW_RIFTS_ACTIVE` 世界标签。[入池](../../../raw/current/scripts/components/riftspawner.lua#L80-L87) [启用](../../../raw/current/scripts/components/riftspawner.lua#L183-L197)
- 修改边界：最多 `MAXIMUM_RIFTS_COUNT(1)` 个裂隙；新点必须离既有裂隙 15 瓦以外；世界设置事件在 cave/forest 各自驱动月/影分支，命令 SpawnRift 不检查启用开关。[上限](../../../raw/current/scripts/tuning.lua#L7620-L7622) [距离](../../../raw/current/scripts/components/riftspawner.lua#L51-L61)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `spawnmode` | 1-5（默认 3） | never/rare/default/often/always；经世界事件 `rifts_setdifficulty` 设置（worldsettings_overrides 在世界生成选项应用时推送），never 停计时器，其它把剩余时间压到 `RIFTS_SPAWNDELAY(5天)`。[难度](../../../raw/current/scripts/components/riftspawner.lua#L153-L174) [事件源](../../../raw/current/scripts/worldsettings_overrides.lua#L3862-L3871) |
| `SpawnRift(forced_pos?)` | → 裂隙或 nil | 从已启用亲和的 RIFTPORTAL_DEFS 随机选 prefab，用 def 的 `GetNextRiftSpawnLocation(map, def)` 选点（forced_pos 优先但同样受 15 瓦距离限制），设置 thrall 类型后入池。prefab 不存在/选点失败返回 nil。[生成](../../../raw/current/scripts/components/riftspawner.lua#L90-L123) |
| `TryToSpawnRift(forced_pos?)` | → 裂隙或 nil | 只在 `rifts_count < MAXIMUM_RIFTS_COUNT` 时转调 SpawnRift。[上限检查](../../../raw/current/scripts/components/riftspawner.lua#L125-L131) |
| 生成计时器（OnRiftTimerDone） | worldsettingstimer `rift_spawn_timer` | 构造时以 `RIFTS_SPAWNDELAY + 1` 注册，初始启用由 `SPAWN_RIFTS ~= 0` 决定；到点生成，失败 1 天后重试，成功且未满则续排 RIFTS_SPAWNDELAY。[计时](../../../raw/current/scripts/components/riftspawner.lua#L41-L46) [重试](../../../raw/current/scripts/components/riftspawner.lua#L135-L151) |
| `EnableLunarRifts`／`EnableShadowRifts`（lunarrift_opened/shadowrift_opened 事件） | 内部 | 置 `WORLDSTATETAGS` 的 LUNAR/SHADOW_RIFTS_ACTIVE 标签、记启用布尔并 TryToStartTimer（同时 `map:StartFindingGoodArenaPoints()`）；月裂隙另启用 wagpunk_manager。[启用](../../../raw/current/scripts/components/riftspawner.lua#L183-L197) [事件源](../../../raw/current/scripts/prefabs/wagstaff_npc.lua#L616 |
| `SetEnabledSetting`／`SetEnabledSettingCave`（rifts_settingsenabled*） | 内部 | 世界设置"never"关闭对应亲和并停计时器（月侧连带 wagpunk Disable）；"always"直接启用。[设置](../../../raw/current/scripts/components/riftspawner.lua#L227-L250) |
| `GetRifts()`／`GetRiftsOfPrefab(prefab)`／`GetRiftsOfAffinity(affinity)` | 表/nil | 裂隙池按 `{实体=prefab}` 记录；Affinity 查询经 RIFTPORTAL_DEFS。`GetRiftsOfType` 是 deprecated 别名。[查询](../../../raw/current/scripts/components/riftspawner.lua#L257-L288) [别名](../../../raw/current/scripts/components/riftspawner.lua#L274 |
| `IsLunarPortalActive()`／`IsShadowPortalActive()`／`GetEnabled()` | → bool | 池中存在对应亲和裂隙即真；GetEnabled 任一启用为真。[状态](../../../raw/current/scripts/components/riftspawner.lua#L307-L323) |
| `OnLunarRiftMaxSize`／`OnShadowRiftMaxSize`（ms_*rift_maxsize 事件） | 内部 | 裂隙长满时对 30 距离外的玩家 push `player._lunarportalmax`／`_shadowportalmax`（客户端小地图/指南针显示）。[满径](../../../raw/current/scripts/components/riftspawner.lua#L203-L225) |
| `OnRiftRemoved(rift)`（onremove 监听） | 内部 | 出池并推 `ms_riftremovedfrompool`；未满且计时器不在跑则重开 RIFTS_SPAWNDELAY。[出池](../../../raw/current/scripts/components/riftspawner.lua#L64-L78) |
| `OnSave/OnLoad/LoadPostPass` | — | 存启用状态与裂隙 GUID；LoadPostPass 恢复池并按需重放 Enable*（含世界标签与 wagpunk），计时器存在则重启选点。[存读](../../../raw/current/scripts/components/riftspawner.lua#L382-L436) |

### 消费者生态

`GetRiftsOfAffinity(SHADOW)`／`IsShadowPortalActive` 被 miasmamanager（雾气创建/衰减门禁）与 shadowparasitemanager（波次前提）等消费；月侧由 lunarriftmutationsmanager 等消费。裂隙实体自身经 `riftthralltype` 组件记录本局 thrall 类型。[雾气门禁](../../../raw/current/scripts/components/miasmamanager.lua#L294-L303) [波次前提](../../../raw/current/scripts/components/shadowparasitemanager.lua#L216-L231)

## 典型接入

### 模组强制开一个裂隙或查询裂隙状态

- 前提与选择：世界需装有 riftspawner（森林/洞穴默认都有）。强制生成用 `SpawnRift(pos)` 可绕过 spawnmode 与上限的 TryToSpawnRift 门禁但仍受 15 瓦间距与选点失败约束；正常启用路径走世界设置事件或 wagstaff 剧情。[命令路径](../../../raw/current/scripts/consolecommands.lua#L2029-L2032)
- 生效与撤销：生成的裂隙实体独立成长，组件侧"撤销"是移除裂隙实体（自动出池并重启计时器）；`spawnmode=1` 只停未来生成，不清现存裂隙。
- 原版依据：难度与启用的官方入口是 worldsettings_overrides 按世界选项推事件；月裂隙剧情源是 wagstaff_npc 完成剧情推 `lunarrift_opened`，暗影侧由 charliecutscene 推 `shadowrift_opened`。[设置源](../../../raw/current/scripts/worldsettings_overrides.lua#L3862-L3871) [剧情源](../../../raw/current/scripts/components/charliecutscene.lua#L254

```lua
-- 服务器端：查询当前世界暗影裂隙是否激活（供任务/生成逻辑门禁）
local function IsShadowRiftOpen()
    local spawner = TheWorld.components.riftspawner
    if spawner == nil then
        return false
    end
    return spawner:IsShadowPortalActive()
end
```

- 调整与验证：两套世界各有一个实例；在森林查询暗影裂隙恒 false（裂隙按亲和与设定落在特定世界），跨世界需求要走分片查询。GetDebugString 显示两侧开关、池数量与计时。[调试](../../../raw/current/scripts/components/riftspawner.lua#L444-L451)

## 常见陷阱与限制

- **SpawnRift 不检查 spawnmode 与上限：** 它是底层入口，控制台/调试专用；想要尊重设置的生成用 TryToSpawnRift 或走事件。[入口层级](../../../raw/current/scripts/components/riftspawner.lua#L90-L131)
- **MAXIMUM_RIFTS_COUNT=1 是硬编码约束：** 注释明示在 portal 逻辑修正前保持 1；改 tuning 提高数量会遇到其它按"唯一裂隙"假设写的系统。[注释](../../../raw/current/scripts/tuning.lua#L7621
- **forced_pos 仍受 15 瓦间距限制：** `IsPointNearPreviousSpawn` 在 forced 分支后统一检查，靠近既有裂隙的强制点位会静默返回 nil。[距离检查](../../../raw/current/scripts/components/riftspawner.lua#L107-L109)
- **GetRiftsOfAffinity 等返回 nil 而非空表：** 无命中时返回 nil，调用方必须按 nil 判（`if rifts == nil`），直接 ipairs 会报错。[返回约定](../../../raw/current/scripts/components/riftspawner.lua#L261-L273)
- **启用状态随存档自动重放：** LoadPostPass 会再次 Enable* 并设置世界标签；模组若在 LoadPostPass 之前改 WORLDSTATETAGS 会被覆盖。[重放](../../../raw/current/scripts/components/riftspawner.lua#L411-L435)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取 riftspawner.lua，核验难度/启用事件、计时与重试、选点与 thrall 类型、池管理与出池、满径推送与存读重放；核对 cave/forest 安装、worldsettings_overrides/wagstaff_npc/charliecutscene 事件源与 miasmamanager/shadowparasitemanager 消费。RIFTPORTAL_DEFS 的选点算法未展开。
- 结构检查：本页通过 validate_page.py 与 validate_source_links.py（严格）；Lua 示例经 parser_probe 语法检查。
- 游戏运行：未运行；裂隙生成节奏与满径推送未实测。

相关：[[reference/components/riftthralltype]]、[[reference/components/lunarriftmutationsmanager]]。

## 相关链接

- [[reference/components/riftthralltype]]：生成时设置的裂隙 thrall 类型组件。
- [[reference/components/lunarriftmutationsmanager]]：月裂隙的生物变异消费方。
- [[reference/components/miasmamanager]]：以阴影裂隙活跃度为门禁的雾气系统。
