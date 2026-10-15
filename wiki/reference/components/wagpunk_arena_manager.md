---
title: "wagpunk_arena_manager"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "瓦冈朋克竞技场任务线组件：以 8 态状态机驱动 Spark Ark→珍珠地图→铺地→组装机器人→拉杆→首领战全流程，战斗时圈人开疯癫并经 net 变量广播屏障。"
aliases: ["瓦冈朋克竞技场", "wagpunk 竞技场任务线", "wagpunk_arena_manager"]
tags: [component, lifecycle, state-machine, network-sync]
size_class: complex
dst_build: "steam-buildid:26230496"
sources: ["components/wagpunk_arena_manager.lua", "prefabs/forest.lua", "components/wagboss_tracker.lua", "tuning.lua", "prefabs/hermitcrab.lua", "prefabs/beebox.lua", "prefabs/monkeyisland_portal.lua", "prefabs/monkeyqueen.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# wagpunk_arena_manager

## 快速判断

- 职责与适用：地表世界组件（仅主端，[装配](../../../raw/current/scripts/prefabs/forest.lua#L716)、[断言](../../../raw/current/scripts/components/wagpunk_arena_manager.lua#L172)）；继承猪王镇/瓦格斯塔夫竞技场的持久任务线，从月裂任务完成到可循环的机器人首领战。范围/屏障几何同步归 [[reference/components/wagpunk_floor_helper]]，击败记录归 [[reference/components/wagboss_tracker]]，垃圾场引导事件归 [[reference/components/wagpunk_manager]]。
- 优先使用：查进度用 `GetStateString()`（SPARKARK/PEARLMAP/PEARLMOVE/TURF/CONSTRUCT/LEVER/BOSS/BOSSCOOLDOWN，[L208-L220](../../../raw/current/scripts/components/wagpunk_arena_manager.lua#L208-L220)）；首领战是否进行中即 `state == mgr.STATES.BOSS`；给竞技场搭布景前必须理解布局锚定（见下）。
- 重要副作用：按状态生成 `wagpunk_lever`/`wagpunk_workstation`/`wagboss_robot`/`wagpunk_cagewall` 等布景、开关 `sanity:EnableLunacy(true, "wagpunk_arena")`、封锁/解锁玩家、经 `wagpunk_floor_helper.barrier_active` 广播屏障（[L1472-L1488](../../../raw/current/scripts/components/wagpunk_arena_manager.lua#L1472-L1488)）。
- 修改边界：布局强依赖 `hermitcrab_01` 静态布景与注册实体，坐标由两对参照实体推角度后八方位变换（见流程）；重复注册参照实体会置 `self.failed` 直接停摆（[L1708-L1711](../../../raw/current/scripts/components/wagpunk_arena_manager.lua#L1708-L1711)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `STATES` / `state` | 0..7 枚举 | SPARKARK→PEARLMAP→PEARLMOVE→TURF→CONSTRUCT→LEVER→BOSS→（死循环回 CONSTRUCT 或 BOSSCOOLDOWN）（[L83-L92](../../../raw/current/scripts/components/wagpunk_arena_manager.lua#L83-L92)、[L1094-L1241](../../../raw/current/scripts/components/wagpunk_arena_manager.lua#L1094-L1241)）。 |
| `TILESPOTS`/`WALLSPOTS`/`ARENA_ENTITIES` | 公开只读表 | 78 格铺地点、碰撞墙点（`WAGPUNK_ARENA_COLLISION_DATA` 平移 -14/-22）与各布景相对偏移；全部是**相对 `storedx/z_pearl` 的偏移**（[L4-L81](../../../raw/current/scripts/components/wagpunk_arena_manager.lua#L4-L81)、[L178-L182](../../../raw/current/scripts/components/wagpunk_arena_manager.lua#L178-L182)）。 |
| `TryToSpawnArenaEntities(prefab, validspotfn, postinitfn)` | `-> table 或 nil` | 按 `ARENA_ENTITIES[prefab]` 偏移生成并跟踪，已存在则跳过；点位校验失败即丢弃该实例（[L514-L538](../../../raw/current/scripts/components/wagpunk_arena_manager.lua#L514-L538)）。 |
| `CheckTurfCompletion()` | `-> bool` | 逐个 TILESPOT 校验 `GetTileAtPoint == WORLD_TILES.WAGSTAFF_FLOOR`，全达标才 `TurfCompleted`（[L796-L808](../../../raw/current/scripts/components/wagpunk_arena_manager.lua#L796-L808)）。 |
| `ShouldWagstaffAcceptItem(inst, item, giver)` | `-> bool` | NPC 交易门禁：PEARLMAP 态只收能看见猴岛瓦片的 `mapscroll`（`mapscroll_tricker` 只对 Pearl 有效）；CONSTRUCT 态按剩余插口收 `gestalt_cage_filled1/2/3`，拒绝原因写 `inst.trader_chatterreason`（[L849-L876](../../../raw/current/scripts/components/wagpunk_arena_manager.lua#L849-L876)、[L886-L982](../../../raw/current/scripts/components/wagpunk_arena_manager.lua#L886-L982)）。 |
| `GetArenaSocketingInstFor(inst, item)` | `-> ent 或 nil` | filled1/2 找最近 `wagdrone_spot_marker`；filled3 找未镶槽的 `wagboss`（[L548-L571](../../../raw/current/scripts/components/wagpunk_arena_manager.lua#L548-L571)）。 |
| `LockPlayersIn()`/`UnlockPlayers()` | `-> nil` | BOSS 态入圈：跟踪玩家 alivecount（鬼/掉线扣减）、开 lunacy、推 `ms_wagpunk_barrier_playerentered/left`，1 秒节拍 OnUpdate 修正出入圈（[L1320-L1370](../../../raw/current/scripts/components/wagpunk_arena_manager.lua#L1320-L1370)、[L1400-L1461](../../../raw/current/scripts/components/wagpunk_arena_manager.lua#L1400-L1461)）。 |
| `DebugSkipState()` 等调试组 | `-> nil` | DebugForcePearl/Turf/Construct/SkipState/PrintFlags，可从主机控制台驱动状态机跳步测试（[L1910-L2032](../../../raw/current/scripts/components/wagpunk_arena_manager.lua#L1910-L2032)）。 |

## 关键流程

### 布局锚定与八方位变换（一切布景的前提）

1. 构造后监听 `ms_register_hermitcrab_marker`/`ms_register_beebox_hermit`/`ms_register_monkeyqueen`/`ms_register_monkeyisland_portal`（事件源：hermitcrab.lua:2137、beebox.lua:363、monkeyqueen.lua:188、monkeyisland_portal.lua:440）。
2. `TryToApplyRotationTransformation` 用 hermitcrab_marker→beebox_hermit 与 monkeyqueen→monkeyportal 两对连线的 `atan2` 角度定锚（[L344-L395](../../../raw/current/scripts/components/wagpunk_arena_manager.lua#L344-L395)）；`ApplyRotationTransformation_Pearl` 按角度分 8 档对 TILESPOTS/WALLSPOTS/ARENA_ENTITIES 做轴翻转／旋转（[L222-L268](../../../raw/current/scripts/components/wagpunk_arena_manager.lua#L222-L268)）。`ApplyRotationTransformation_Monkey` 已定义但无调用（[L269](../../../raw/current/scripts/components/wagpunk_arena_manager.lua#L269)）。
3. 变换完成前置位 `appliedrotationtransformation`；`QueueCheck` 在此之前会顺延一帧重排（[L191-L206](../../../raw/current/scripts/components/wagpunk_arena_manager.lua#L191-L206)）。

### 状态机推进（CheckStateForChanges while 循环）

- **SPARKARK**：监听 `ms_lunarriftmutationsmanager_taskcompleted` → `SparkArkCompleted`（[L1853](../../../raw/current/scripts/components/wagpunk_arena_manager.lua#L1853)）→ PEARLMAP。
- **PEARLMAP**：`SpawnWagstaffSetPiece` 放拉杆/工作台/废料堆/围栏（[L479-L490](../../../raw/current/scripts/components/wagpunk_arena_manager.lua#L479-L490)）；给 Pearl 正确地图后 `PearlMapCompleted` 并让 `hermitcrab_relocation_manager:SetupMovingPearlToMonkeyIsland()`（[L986-L996](../../../raw/current/scripts/components/wagpunk_arena_manager.lua#L986-L996)）；`ms_hermitcrab_relocated` → TURF。
- **TURF**：放 `wagpunk_floor_marker` 与铺地指示、工作台学习 `wagpunk_floor_kit`（[L1119-L1123](../../../raw/current/scripts/components/wagpunk_arena_manager.lua#L1119-L1123)）；`ms_wagpunk_floor_kit_deployed` 逐格校验达标 → CONSTRUCT。
- **CONSTRUCT**：放无人机插口标记与 `wagboss_robot`（未击败过时）、工作台学习 `gestalt_cage`；`ms_wagpunk_constructrobot` 检查 `CheckConstructCompleted`——没镶满槽先放一次月境守护者（[L810-L836](../../../raw/current/scripts/components/wagpunk_arena_manager.lua#L810-L836)）；镶满 → LEVER。
- **LEVER**：立 `wagpunk_cagewall`、拉杆伸出、撤 NPC 交易；`ms_wagpunk_lever_activated` → BOSS（[L1153-L1166](../../../raw/current/scripts/components/wagpunk_arena_manager.lua#L1153-L1166)）。
- **BOSS**：墙伸出、碰撞体入场并清场、`wagboss:PushEvent("activate")`、`LockPlayersIn`；战败结算见下（[L1167-L1236](../../../raw/current/scripts/components/wagpunk_arena_manager.lua#L1167-L1236)）。
- **BOSS/COOLDOWN 结算**：boss 死亡时若 [[reference/components/wagboss_tracker]] 已记永久击败 → 学会 4 个击败配方并回 CONSTRUCT 循环刷（[L1085-L1092](../../../raw/current/scripts/components/wagpunk_arena_manager.lua#L1085-L1092)）；否则 BOSSCOOLDOWN 等 `TUNING.WAGPUNK_ARENA_WAGBOSS_ROBOT_COOLDOWN_DEFEATED_TIME`（20 天，[tuning.lua:8731](../../../raw/current/scripts/tuning.lua#L8731）后回 CONSTRUCT；boss 活着把玩家清场则回 LEVER 重打（[L1167-L1193](../../../raw/current/scripts/components/wagpunk_arena_manager.lua#L1167-L1193)）。

### Wagstaff NPC 调度

- `wagstaff_npc_wagpunk_arena` 按 `WorkstationToggled`/`LeverToggled` 每帧重排（0.1 秒任务）在对应站点现身、传送、播 chatter，按状态挂 `tiedtoworkstation`/`tiedtolever`/`oneshot` 标记（[L594-L717](../../../raw/current/scripts/components/wagpunk_arena_manager.lua#L594-L717)、[L722-L771](../../../raw/current/scripts/components/wagpunk_arena_manager.lua#L722-L771)）；一次性事件走 `ms_wagstaff_arena_oneshot`（[L1868-L1904](../../../raw/current/scripts/components/wagpunk_arena_manager.lua#L1868-L1904)）。`wagboss_tracker:IsWagbossDefeated()` 为真后不再生成（[L785-L788](../../../raw/current/scripts/components/wagpunk_arena_manager.lua#L785-L788)）。

## 典型接入

### 模组查询竞技场进度 / 自定义插口逻辑

- 前提与选择：仅主端；查询用公开字段与 `GetStateString`，改流程优先走原版事件（`ms_wagpunk_constructrobot` 等）而不是直改 state。

```lua
AddPrefabPostInit("world", function(inst)
    if not TheWorld.ismastersim then
        return
    end
    inst:ListenForEvent("ms_wagpunk_constructrobot", function(world)
        local mgr = world.components.wagpunk_arena_manager
        if mgr ~= nil then
            print("arena state:", mgr:GetStateString(),
                "drones:", mgr:GetTotalDronePlacementCount())
        end
    end)
end)
```

- 调整与验证：`GetTotalDronePlacementCount` 统计 `wagdrone_rolling`/`wagdrone_flying`（[L1832-L1840](../../../raw/current/scripts/components/wagpunk_arena_manager.lua#L1832-L1840)）。测试任务线可在主机控制台对选中世界实体调 `DebugSkipState()` 逐态跳步；铺地跳步会真写 WAGSTAFF_FLOOR 地块并挪走 winch 目标（[L1929-L1971](../../../raw/current/scripts/components/wagpunk_arena_manager.lua#L1929-L1971)），测试档专用。

## 常见陷阱与限制

- **布景锚点不跟随世界原点**：全部偏移相对 Pearl（hermitcrab 布景）存储原点；没有那两对参照实体（或重复出现）就 `failed=true`，任务线在该世界无法推进，staging 分支会公告管理员（[L344-L351](../../../raw/current/scripts/components/wagpunk_arena_manager.lua#L344-L351)）。
- **圈人判定是位置节拍而非硬屏障**：OnUpdate 每秒按 `Map:IsPointInWagPunkArena` 修正成员；复活/传送瞬间出入圈属已知行为（源码 FIXME 列出 Wanda 复活、Winona 复活/传送、肉像，[L1450-L1452](../../../raw/current/scripts/components/wagpunk_arena_manager.lua#L1450-L1452)）。范围几何见 [[reference/components/wagpunk_floor_helper]]。
- **Boss 冷却读档压缩**：存档 `bosscooldownremaining` 读档时 `math.min(1, ...)` 截到最多 1 秒——旧档长冷却重开后几乎立即结束（[L1622-L1625](../../../raw/current/scripts/components/wagpunk_arena_manager.lua#L1622-L1625)）。
- **全员离场 grace period**：圈定玩家全部断线时延迟 `WAGPUNK_ARENA_WAGBOSS_ROBOT_DESPAWN_GRACE_TIME`（5 分钟）才 `doreset`（[L1290-L1302](../../../raw/current/scripts/components/wagpunk_arena_manager.lua#L1290-L1302)、[tuning.lua:8730](../../../raw/current/scripts/tuning.lua#L8730）；期间 boss 保持。
- **未击败态的 CONSTRUCT 双产物**：首次走 `wagdrone_spot_marker`+`wagboss_robot`；永久击败后只放 `wagboss_robot_constructionsite_placerindicator`（[L1134-L1146](../../../raw/current/scripts/components/wagpunk_arena_manager.lua#L1134-L1146)），模组复读该状态机时要区分两条路径。
- **进行态可存档恢复**：与 yotb 不同，这里 OnSave/OnLoad/LoadPostPass 完整保存状态、GUID 引用与掉线 KU；LoadPostPass 还会在 CONSTRUCT 态重跑完成检查并补发击败配方（[L1498-L1699](../../../raw/current/scripts/components/wagpunk_arena_manager.lua#L1498-L1699)）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。通读组件全部 2033 行；装配行（forest.lua:716）、四个注册事件源行（hermitcrab/beebox/monkeyqueen/monkeyisland_portal）、tuning 两条常量、wagboss_tracker 交互复用其页已核验记录。页面为精选覆盖：布局数据表（TILESPOTS/WALLSPOTS/ARENA_ENTITIES）按用途归并，未逐坐标登记；`ApplyRotationTransformation_Monkey` 为未接线定义，经全文件调用点检索确认。
- 结构检查：validate_page、validate_source_links --strict 通过（见批级记录）。
- 游戏运行：未运行；任务线推进、圈人与首领战结算未实测。

相关：[[reference/components/wagboss_tracker]]、[[reference/components/wagpunk_manager]]、[[reference/components/wagpunk_floor_helper]]、[[reference/components/reticule]]
