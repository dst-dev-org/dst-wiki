---
title: "flotsamgenerator"
created: "2026-10-21"
updated: "2026-10-28"
type: component
summary: "服务端 forest 世界的海面漂浮物生成器：按玩家船速周期刷残骸、3–4 天沉没，并提供手动刷点与沉没计时接口。"
aliases: ["漂浮物生成", "海上残骸", "海面漂浮物", "flotsamgenerator"]
tags: [component, lifecycle, worldgen]
dst_build: "steam-buildid:26230496"
sources: ["components/flotsamgenerator.lua", "prefabs/forest.lua", "prefabs/world.lua", "map/locations.lua", "components/map.lua", "components/periodicspawner.lua", "prefabs/oceanfishableflotsam.lua", "simutil.lua", "tuning.lua", "entityscript.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# flotsamgenerator

## 快速判断

- 职责与适用：forest 世界级的海面漂浮物生成器。给每个站在平台上的玩家按 30–180 秒随机周期，在其周围约 40 格海面刷一个残骸（有航速时偏向前方），并统一管理“什么时候沉”。作用对象是 `TheWorld`，不是某个残骸 prefab；与船上的 [[reference/components/boatleak]] 分属两个层级。[构造](../../../raw/current/scripts/components/flotsamgenerator.lua#L5-L63)
- 优先使用：手动刷一个 `SpawnFlotsam(spawnpoint, prefab, notrealflotsam)`；给已有实体补“会沉”的计时 `setinsttoflotsam(inst, time, notag)`；整体开关 `ToggleUpdate()`；改密度／周期改 `TUNING.FLOTSAM_SPAWN_MAX`、`TUNING.FLOTSAM_SPAWN_DELAY`。[SpawnFlotsam](../../../raw/current/scripts/components/flotsamgenerator.lua#L322-L342)
- 重要副作用：给每个在线玩家挂一条循环任务和一条 messagebottle 保底任务；刷出的实体被加 `flotsam` tag 和 `timer`；`StartTracking` 会把实体 `persists` 改成 false。[ToggleUpdate](../../../raw/current/scripts/components/flotsamgenerator.lua#L159-L183)
- 修改边界：只在服务端（组件内 `assert(TheWorld.ismastersim)`）；只在有海洋的世界有效（`TheWorld.has_ocean`）；不生成海面地形或沉船、也不给漂浮物做物理推进。

## 关键接口

| 接口／字段 | 已核验类型／单位 | 条件、作用与证据 |
|---|---|---|
| `SpawnFlotsam(spawnpoint, prefab, notrealflotsam)` | `Vector3` 落点；prefab 名可省 | 省略 prefab 时按加权表抽；`notrealflotsam` 为真则不加 `flotsam` tag（不挡后续刷点）。返回实体或 nil（抽不到 prefab 时）。[L322-L342](../../../raw/current/scripts/components/flotsamgenerator.lua#L322-L342) |
| `setinsttoflotsam(inst, time, notag)` | `time` 秒；nil → 3 天 + 0..1 天 | 补 `timer` 组件、加 `flotsam` tag（`notag` 为真跳过）、起 `flotsamgenerator_sink` 计时，并挂 `timerdone`／`onpickup`／`onremove` 监听。[L303-L320](../../../raw/current/scripts/components/flotsamgenerator.lua#L303-L320) |
| `ToggleUpdate()` | 无参 | 等价 `ToggleUpdate(true)`：取消并重排所有在场玩家的刷点与保底任务；`_maxflotsam > 0` 才排。[L299-L301](../../../raw/current/scripts/components/flotsamgenerator.lua#L299-L301) |
| `SetSpawnTimes(delay)` | `{min, max}` 秒 | 从 birdspawner 抄来的废弃接口，会 `print` 警告并改本组件周期；无返回值。[L293-L297](../../../raw/current/scripts/components/flotsamgenerator.lua#L293-L297) |
| `StartTracking(target)`／`StopTracking(target)` | 实体 | 暂存并清空 `target.persists`，并在 `entitysleep` 时移除目标；原版没有任何调用者（见陷阱）。[L344-L367](../../../raw/current/scripts/components/flotsamgenerator.lua#L344-L367) |
| `ScheduleGuaranteedSpawn(player, preset, override_time)` | 秒或 nil | 保底任务内部用；要求该玩家已由 `StartGuaranteedSpawn` 建过表，否则索引 nil。[L369-L371](../../../raw/current/scripts/components/flotsamgenerator.lua#L369-L371) |
| `OnSave`／`OnLoad`／`LoadPostPass` | 存档表 | 存 `maxflotsam`、`minspawndelay`、`maxspawndelay` 和每个在场漂浮物的 GUID、剩余时间、是否带 tag；读档用 `setinsttoflotsam` 复原。[L377-L421](../../../raw/current/scripts/components/flotsamgenerator.lua#L377-L421) |

### 刷点条件与方向

1. 每个玩家一条 `DoTaskInTime(随机周期)`：首轮（`initialspawn`）取 `0..max-min`，之后取 `min..max`（默认 30–180 秒）；周期还乘一个恒为 1 的 `_timescale`。[ScheduleSpawn](../../../raw/current/scripts/components/flotsamgenerator.lua#L125-L133)
2. 玩家必须站在平台上：`player:GetCurrentPlatform()` 为 nil 就直接跳过本次刷点，任务照常续排。[L107-L116](../../../raw/current/scripts/components/flotsamgenerator.lua#L107-L116)
3. 世界要有海：`TheWorld.has_ocean` 为假则 `GetSpawnPoint` 返回 nil；候选点用 `Map:IsSurroundedByWater(…, 2)` 判（函数内部把半径 +1，再按方框采样海洋格）。[L70-L77](../../../raw/current/scripts/components/flotsamgenerator.lua#L70-L77)、[map](../../../raw/current/scripts/components/map.lua#L645-L675)
4. 间距：35 格内不能有 player 或带 `flotsam` tag 的实体，5 格内不能有 `INLIMBO`／`fx`。[L68-L77](../../../raw/current/scripts/components/flotsamgenerator.lua#L68-L77)
5. 方向：平台有 `boatphysics` 时按速度方向加偏置——速度 ≤0.1 单位/秒时偏置可达 ±π（近似全向），≥1.5 时收窄到约 ±0.21 弧度（贴着正前方）；再以 `FindValidPositionByFan(theta, 40, 12, …)` 扇形找点，找不到这轮不刷。[L79-L105](../../../raw/current/scripts/components/flotsamgenerator.lua#L79-L105)、[fan](../../../raw/current/scripts/simutil.lua#L294-L325)
6. 抽取：加权表为 `driftwood_log`、`boatfragment03/04/05`、`cutgrass`、`twigs` 各 1（碎片 0.3）、`oceanfishableflotsam_water` 1；刷出后有一半概率转 180°。[L19-L28](../../../raw/current/scripts/components/flotsamgenerator.lua#L19-L28)、[L185-L188](../../../raw/current/scripts/components/flotsamgenerator.lua#L185-L188)

### 沉没与清理

- 默认寿终 3–4 天（`LIFESPAN`）；到期生成 `splash_sink` 后 `Remove`。实体若自带 `overrideflotsamsinkfn`（如 `oceanfishableflotsam_water`）则整体交给它。[L205-L221](../../../raw/current/scripts/components/flotsamgenerator.lua#L205-L221)、[覆写](../../../raw/current/scripts/prefabs/oceanfishableflotsam.lua#L277-L278)
- 被捡起或移除：停掉计时、摘 `flotsam` tag、从在场表删除，不再占间距。[L217-L221](../../../raw/current/scripts/components/flotsamgenerator.lua#L217-L221)
- 存读：`OnSave` 顺带把在场实体 GUID 交给引擎；`LoadPostPass` 用 `setinsttoflotsam(实体, 剩余时间, notag)` 复原，所以读档后剩余沉没时间不重置。[L377-L421](../../../raw/current/scripts/components/flotsamgenerator.lua#L377-L421)

## 典型接入

### 原版宿主：forest 世界

- 唯一宿主：`prefabs/forest.lua` 的 master_postinit 里 `AddComponent("flotsamgenerator")`，世界注册名 `forest`；洞穴世界没有这个组件。[宿主](../../../raw/current/scripts/prefabs/forest.lua#L681-L682)、[注册](../../../raw/current/scripts/prefabs/forest.lua#L745-L747)
- 海是 forest 世界的一部分：location 覆盖里 `has_ocean = true`，所以默认海图判定通过。[location](../../../raw/current/scripts/map/locations.lua#L3-L17)
- 启动链路：构造时登记 `ms_playerjoined`／`ms_playerleft`；世界 `PostInit` 遍历组件调 `OnPostInit` → `ToggleUpdate(true)` 才开始给在场玩家排任务，`OnLoad` 也会重排一次。[初始化](../../../raw/current/scripts/components/flotsamgenerator.lua#L269-L284)、[world PostInit](../../../raw/current/scripts/prefabs/world.lua#L336-L348)、[OnLoad](../../../raw/current/scripts/components/flotsamgenerator.lua#L401-L407)
- 消费者：`periodicspawner` 在海上、落点又不在平台／视觉地面上时，改调 `TheWorld.components.flotsamgenerator:SpawnFlotsam(spawnpos, prefab, true)`，于是这类世界刷点产物也带沉没计时、且不互相挡刷点。[periodicspawner](../../../raw/current/scripts/components/periodicspawner.lua#L179-L183)

### 模组：改密度与周期（含读档覆盖）

- 前提与选择：`_maxflotsam`／`_minspawndelay`／`_maxspawndelay` 都是文件内 local，构造时从 TUNING 取值，没有公开 setter；唯一写入口是 `OnLoad`（读存档里的三项）。所以新世界改 TUNING，已有存档要包 `OnLoad`。
- 端与资源：仅服务端；`AddComponentPostInit("flotsamgenerator", fn)`，fn 收到 `(组件实例, 宿主实体)`。[postinit 调用点](../../../raw/current/scripts/entityscript.lua#L659-L663)
- 生效与撤销：新世界看 TUNING；存档世界看包过的 `OnLoad`，且写回存档的数值会被下一次读档继续沿用，删 postinit 后旧档仍是改过的值。
- 单位：周期与 `time` 都是秒；`maxflotsam` 只是开关量（见陷阱）。

```lua
-- modmain.lua：新世界 8 个、60–120 秒一轮；读档时覆盖存档里的旧值
GLOBAL.TUNING.FLOTSAM_SPAWN_MAX = 8
GLOBAL.TUNING.FLOTSAM_SPAWN_DELAY = { min = 60, max = 120 }

AddComponentPostInit("flotsamgenerator", function(self)
    local onload = self.OnLoad
    self.OnLoad = function(inner, data)
        data = data or {}
        data.maxflotsam = GLOBAL.TUNING.FLOTSAM_SPAWN_MAX
        data.minspawndelay = GLOBAL.TUNING.FLOTSAM_SPAWN_DELAY.min
        data.maxspawndelay = GLOBAL.TUNING.FLOTSAM_SPAWN_DELAY.max
        return onload(inner, data)
    end
end)
```

- 调整与验证：示例只做 Lua 5.1 语法检查，未在游戏内执行；调参点与存档字段见上表行锚点。

## 常见陷阱与限制

- `FLOTSAM_SPAWN_MAX` 不是硬上限：`_maxflotsam` 只在 `ToggleUpdate` 里当 0/非 0 的开关（`> 0`），刷点路径不数在场数量，所以实际数量由周期、玩家数与 3–4 天寿命共同决定；`GetDebugString` 的 `flotsam:n/max` 才会暴露超出。[L159-L183](../../../raw/current/scripts/components/flotsamgenerator.lua#L159-L183)、[L427-L433](../../../raw/current/scripts/components/flotsamgenerator.lua#L427-L433)
- 改 TUNING 对既有存档无效：`OnLoad` 用存档里的 `maxflotsam`／`minspawndelay`／`maxspawndelay` 覆盖构造值，老档看不到变化。[L401-L407](../../../raw/current/scripts/components/flotsamgenerator.lua#L401-L407)
- `_maxflotsam = 0` 拦不住 messagebottle：关停分支只清当前在场玩家的保底任务，而 `OnPlayerJoined` 对新玩家无条件 `StartGuaranteedSpawn`，保底 bottle 仍会刷。[L239-L251](../../../raw/current/scripts/components/flotsamgenerator.lua#L239-L251)
- 玩家离开会漏保底任务：`OnPlayerLeft` 命中列表后立刻 `return`，`CancelGuaranteedSpawn` 写在没命中的分支里；任务挂在玩家实体上会随实体销毁，但 `_guaranteed_spawn_tasks[player]` 表项不再回收。[L253-L263](../../../raw/current/scripts/components/flotsamgenerator.lua#L253-L263)
- 只在船上刷：`SpawnFlotsamForPlayer` 要求 `GetCurrentPlatform()`，岸上或纯游泳的玩家不会触发；若平台没有 `boatphysics`（非船平台）方向完全随机，不会被航速收窄。[L107-L123](../../../raw/current/scripts/components/flotsamgenerator.lua#L107-L123)
- `StartTracking`／`StopTracking` 是死接口：原版只有定义没有调用者，所以“漂浮物睡着就移除”的分支（`OnTargetSleep` → `AutoRemoveTarget`）在纯净版不会触发；自己调用会把实体 `persists` 改成 false，别对需要存档的东西用。[L344-L367](../../../raw/current/scripts/components/flotsamgenerator.lua#L344-L367)、[自动移除](../../../raw/current/scripts/components/flotsamgenerator.lua#L190-L203)
- `OnSave` 假设每个在场实体都有 `timer`：直接 `k.components.timer:GetTimeLeft`，用 `StartTracking` 塞进来的非漂浮物会在这里报错。[L389-L396](../../../raw/current/scripts/components/flotsamgenerator.lua#L389-L396)
- 刷点开销随玩家数与船速上升：一次刷点最多试 12 个扇形候选，每个候选 1 次海洋判定 + 最多 2 次 `FindEntities`，都在玩家附近执行；别把 `ToggleUpdate()` 当高频调用用。[L70-L105](../../../raw/current/scripts/components/flotsamgenerator.lua#L70-L105)

## 验证记录

- 版本增量复核 26148036（2026-10-25）：锚点重定位 2 处（forest.lua 宿主装配／注册段 -3，本版删除 prefabs 表 3 行）；无结论修订，仅锚点迁移——flotsamgenerator.lua、map.lua（本版改动在 925 行之后，IsSurroundedByWater 645-675 未变）、simutil.lua（改动在 371 行之后，fan 段未变）、world.lua（336-348 在 398 行插入之前，未变）、locations／periodicspawner／oceanfishableflotsam 未变。依据：全局枢纽 diff 审阅＋变更清单命中 prefabs/world.lua、components/map.lua、simutil.lua、tuning.lua、entityscript.lua、prefabs/forest.lua（本页 entityscript 引用 637-641 未变）。
- 源码审阅：2026-10-21，steam-buildid:25643519。覆盖构造与常量表、`GetSpawnPoint` 判定与船速偏置、`ScheduleSpawn`／`ToggleUpdate`、messagebottle 保底链、`setinsttoflotsam`／`SpawnFlotsam`、沉没回调、`OnSave`／`OnLoad`／`LoadPostPass`；核对宿主为 forest 世界（非洞穴）、`periodicspawner` 消费点与 `oceanfishableflotsam_water` 的沉没覆写；未覆盖多人并发与表现层。
- 游戏运行：未运行；本页结论均来自源码静态核验，不含实测。
- 结构检查：`validate_page.py` 通过（0 issue；104 行、13279 字节，低于 180 行／14 KiB）；`validate_source_links.py --strict` 通过（41 处引用、0 error／0 warning）；正文 Lua 示例另存 `output/group110-flotsam.lua`，`parser_probe --syntax-only` 通过（只查 Lua 5.1 语法，未执行）。

- 2026-10-28 build 26230496：forest.lua 挂载 leafpilemanager 使注册行下移；oceanfishableflotsam 活动分支重排锚点随移。flotsamgenerator 组件逻辑未变。
相关：[[reference/components/boatleak]]、[[reference/components/boatphysics]]。
