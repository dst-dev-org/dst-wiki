---
title: "sharkboimanager"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "世界级鲨鱼仔竞技场组件：在海面自动铺冰Arena，中央冰洞钓鱼触发 boss；按季节胀缩冰面，战斗结束冷却后迁址重建。"
aliases: ["鲨鱼仔竞技场", "冰钓洞", "sharkboimanager"]
tags: [component, creature, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/sharkboimanager.lua", "prefabs/forest.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# sharkboimanager

## 快速判断

- 职责与适用：世界级组件（仅主端），装配于地表世界并依赖 oceanicemanager（[装配注释](../../../raw/current/scripts/prefabs/forest.lua#L709)）；世界尺寸确定后在深海自动找点建冰面竞技场，玩家在中央 `icefishing_hole` 钓满 3 条鱼即刷 `sharkboi` boss。
- 优先使用：模组一般只读状态：`IsPointInArena(x, y, z)`、`GetArenaStateString()`、`GetFishPrefab()`（[L52-L64](../../../raw/current/scripts/components/sharkboimanager.lua#L52-L64)、[L168-L177](../../../raw/current/scripts/components/sharkboimanager.lua#L168-L177)、[L370-L386](../../../raw/current/scripts/components/sharkboimanager.lua#L370-L386)）；客户端表现数据经 `world.net` 的 [[reference/components/sharkboimanagerhelper]] netvar 同步（[推送](../../../raw/current/scripts/components/sharkboimanager.lua#L12-L32)）。
- 重要副作用：直接改海洋地形（oceanicemanager 铺冰/毁冰）、生成冰洞与冰刺装饰、瞬间击碎压岸的船、推开冰洞旁战斗单位（[L307-L317](../../../raw/current/scripts/components/sharkboimanager.lua#L307-L317)、[L431-L437](../../../raw/current/scripts/components/sharkboimanager.lua#L431-L437)）。
- 修改边界：竞技场同一时刻至多一个（`PlaceOceanArenaAtPosition` 判重）；冷却 20 天后强制清场再选新址（[L647-L666](../../../raw/current/scripts/components/sharkboimanager.lua#L647-L666)、[L341-L347](../../../raw/current/scripts/components/sharkboimanager.lua#L341-L347)、[TUNING](../../../raw/current/scripts/tuning.lua#L6997-L7000)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `STATES` | UNDEFINED→CREATINGARENA→CREATEDARENA→BOSSSPAWNED→BOSSFIGHTING→CLEANUP | 状态只升不降（构造断言）；`SetArenaState` 驱动 ArenaFinishCreating/SpawnBoss/StartShrinking/StartCleanup（[L44-L51](../../../raw/current/scripts/components/sharkboimanager.lua#L44-L51)、[L179-L211](../../../raw/current/scripts/components/sharkboimanager.lua#L179-L211)）。 |
| `arena` 表 | `{state, origin, radius, desiredradius, caughtfish, fishinghole, sharkbois, cooldowntask}` | 运行期数据；radius 初始按季节取 MAX（冬）或 MIN（[L652-L657](../../../raw/current/scripts/components/sharkboimanager.lua#L652-L657)、[L140-L142](../../../raw/current/scripts/components/sharkboimanager.lua#L140-L142)）。 |
| `MAX_ARENA_SIZE` / `MIN_ARENA_SIZE` | 6×TILE_SCALE / 2.5×TILE_SCALE | 圆形瓦片缓存 `TILEOFFSET_CACHE` 供半径区间遍历（[L40-L42](../../../raw/current/scripts/components/sharkboimanager.lua#L40-L42)、[L78-L108](../../../raw/current/scripts/components/sharkboimanager.lua#L78-L108)）。 |
| `PlaceOceanArenaAtPosition(x, y, z)` / `TryToPlaceOceanArena()` / `FindAndPlaceOceanArenaOverTime()` | → bool | 选点取最深的 `GetGoodOceanArenaPoints`；无候选点时每 5 秒重试并触发地图后台找点（[L681-L739](../../../raw/current/scripts/components/sharkboimanager.lua#L681-L739)、[入口](../../../raw/current/scripts/components/sharkboimanager.lua#L891-L894)）。 |
| `TryToMakeArenaBig()` / `TryToMakeArenaSmall()` | — | 冬季放大：玩家一屏外才执行（否则 5 秒重试），铺冰到 MAX 并碎掉压岸船只；非冬季 desiredradius 缩到 MIN（[L278-L331](../../../raw/current/scripts/components/sharkboimanager.lua#L278-L331)、[季节监听](../../../raw/current/scripts/components/sharkboimanager.lua#L896-L907)）。 |
| 冰洞钓鱼 tick | `startfishinginvirtualocean` 事件驱动 | 每玩家 1s `TryLuckRoll(0.2)` 上钩；boss 已刷或 `SPAWN_SHARKBOI` 关闭时概率降为 1/4；鱼 prefab 取自 schoolspawner 区域表（[L460-L505](../../../raw/current/scripts/components/sharkboimanager.lua#L460-L505)、[TUNING](../../../raw/current/scripts/tuning.lua#L7005-L7007)）。 |
| `OnPlayerFishCaught` | fishcaught 计数 | 每 3 条转 `BOSSSPAWNED`（`SPAWN_SHARKBOI` 才生效）（[L440-L459](../../../raw/current/scripts/components/sharkboimanager.lua#L440-L459)）。 |
| `SpawnBoss()` | — | 先强制收线上玩家的鱼，在冰洞处生成 sharkboi（进 `spawn` 态）、TrackFishingHole、挂 attacked/onremove 监听、推开附近战斗单位（[L397-L438](../../../raw/current/scripts/components/sharkboimanager.lua#L397-L438)）。 |
| `OnAttacked_SharkBoi` / `OnRemove_SharkBoi` | 组件侧监听 | 玩家（或其跟随者）首击转 `BOSSFIGHTING` 并全员激怒；全部死亡转 `CLEANUP` 并进入冷却（[L516-L541](../../../raw/current/scripts/components/sharkboimanager.lua#L516-L541)）。 |
| `ForceCleanup()` | — | 停任务、销毁全部冰瓦/冰洞/sharkboi、推 `ms_cleanedupsharkboiarena`；清理阶段先逐格毁冰并逐 tick 推 `ms_cleanupticksharkboiarena`（[L258-L267](../../../raw/current/scripts/components/sharkboimanager.lua#L258-L267)、[L349-L368](../../../raw/current/scripts/components/sharkboimanager.lua#L349-L368)）。 |
| OnSave/OnLoad/LoadPostPass | origin/caughtfish/state 字符串/radius/cooldown | 未知状态直接进 CLEANUP；兼容旧单 boss 字段并重定位中心 4 距离实体（[L757-L866](../../../raw/current/scripts/components/sharkboimanager.lua#L757-L866)）。 |

## 关键流程

### 一次竞技场周期

1. 选址：`worldmapsetsize` → 每 5s 尝试 → 冰面 CREATINGARENA → 次帧 CREATEDARENA：中央冰洞＋外圈三档 `sharkboi_ice_hazard`（tall/medium/short，约 80 个），推 `ms_spawnedsharkboiarena`（[L608-L637](../../../raw/current/scripts/components/sharkboimanager.lua#L608-L637)）。
2. 钓鱼→boss：玩家冰钓上钩计数，3 条刷 boss；首次受击进 BOSSFIGHTING 并开始每 2s 缩冰 0.1 半径（战斗压迫，[TUNING](../../../raw/current/scripts/tuning.lua#L6998-L7000)）。
3. 收场：boss 全灭 → CLEANUP 逐格毁冰 → `ForceCleanup` → 冷却 20 天 → 清 arena → 回到选址（[L333-L347](../../../raw/current/scripts/components/sharkboimanager.lua#L333-L347)）。

## 常见陷阱与限制

- **缩冰是地形操作**：`DestroyIceTileAtPoint` 走 oceanicemanager 队列，模组在 arena 附近自建冰瓦可能被误清或残留（[L639-L645](../../../raw/current/scripts/components/sharkboimanager.lua#L639-L645)）。
- **放大需无人注视**：`TryToMakeArenaBig` 在玩家一屏内只重试不施工；`PauseArenaShrinking_Hack` 提供 85 帧暂停收缩的窗口（sharkboi 表现用，[L155-L166](../../../raw/current/scripts/components/sharkboimanager.lua#L155-L166)、[L298-L304](../../../raw/current/scripts/components/sharkboimanager.lua#L298-L304)）。
- **钓鱼概率受 boss 状态影响**：boss 在场或禁刷时上钩率除 4，避免无限刷鱼（[L466-L472](../../../raw/current/scripts/components/sharkboimanager.lua#L466-L472)）。
- **装饰生成函数是模组空位**：`CreateEntityDecorationsAtPoint` 故意为空 stub，模组可覆写往 arena 撒装饰（[L575-L606](../../../raw/current/scripts/components/sharkboimanager.lua#L575-L606)）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。覆盖组件全文、forest 装配行、TUNING 竞技场段；未覆盖 sharkboi/icefishing_hole prefab 内部与 oceanicemanager 瓦片实现。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、Lua 语法探测通过（见批级记录）。

相关：[[reference/components/sharkboimanagerhelper]]。
