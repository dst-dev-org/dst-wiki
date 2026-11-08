---
title: "龙虾窝（wobster_den、moonglass_wobster_den、wobster_den_spawner_shore）"
created: "2026-11-09"
updated: "2026-11-09"
type: prefab
summary: "龙虾窝的两个实体变体与世界生成标记：两套概率掉落表、9 点挖掘与四段美术、孩子刷新（最多 2 只、半径 4、水面上生成、白天停刷）、船撞换算，以及月光窝的就地改写和 shore 标记换成实体的放置链。"
aliases: ["wobster_den", "moonglass_wobster_den", "wobster_den_spawner_shore", "龙虾窝", "月光龙虾窝", "挖龙虾窝", "龙虾窝生成"]
tags: [prefab, building, worldgen]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/wobster_den.lua", "worldsettingsutil.lua", "components/childspawner.lua", "components/floater.lua", "components/halloweenmoonmutable.lua", "components/lootdropper.lua", "components/workable.lua", "components/worldstate.lua", "map/bunches.lua", "map/bunch_spawner.lua", "map/forest_map.lua", "map/rooms/forest/terrain_ocean.lua", "prefabs/forest.lua", "prefablist.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 龙虾窝（wobster_den、moonglass_wobster_den、wobster_den_spawner_shore）

## 快速判断

- 职责与适用：`prefabs/wobster_den.lua` 注册三个 prefab——普通窝 `wobster_den`、月光窝 `moonglass_wobster_den`（同一 `basefn`，只换 build、掉落表、孩子名），以及世界生成标记 `wobster_den_spawner_shore`（空实体，仅用于摆放）。[注册](../../../raw/current/scripts/prefabs/wobster_den.lua#L311-L313)
- 优先使用：改掉落看[掉落与挖掘](#掉落与挖掘)；改刷新节奏与上限看[孩子刷新](#孩子刷新)；月光转换看[月光就地改写](#月光就地改写)；世界生成怎么落点看[世界生成标记](#世界生成标记)。
- 重要副作用：挖掉时先 `DropLoot` 再 `ReleaseAllChildren` 把"在窝里"的孩子放到水面（连续 3 次找不到落点会提前停），然后自删；被船撞除了扣挖掘进度，也会立刻触发一次释放。[挖完](../../../raw/current/scripts/prefabs/wobster_den.lua#L158-L173)、[船撞](../../../raw/current/scripts/prefabs/wobster_den.lua#L175-L186)
- 修改边界：孩子形态与数值归 [[reference/prefabs/wobster]]，`childspawner` 通用接口归 [[reference/components/childspawner]]；本页只写窝自身的掉落、挖掘、刷新接线与世界生成（S8）。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `wobster_den` | `fn()` → `basefn("lobster_den_build", "wobster_den", "wobster_sheller")`，返回后再挂 `halloweenmoonmutable` 并设 `SetConversionOverrideFn(moonconversionoverridefn)` | 主模拟才有组件；转换函数只对普通窝存在。[普通窝](../../../raw/current/scripts/prefabs/wobster_den.lua#L285-L292) |
| `moonglass_wobster_den` | `moonglassfn()` → `basefn("lobster_den_moonglass_build", "moonglass_wobster_den", "wobster_moonglass")` | 没有 `halloweenmoonmutable`，不会再转一次。[月光窝](../../../raw/current/scripts/prefabs/wobster_den.lua#L294-L298) |
| `wobster_den_spawner_shore` | `spawner_fn()`：`CreateEntity`＋`AddTransform`＋`AddTag("CLASSIFIED")`，无组件 | 世界生成标记，源码注释 "Non-networked entity"。[标记工厂](../../../raw/current/scripts/prefabs/wobster_den.lua#L300-L309) |

`basefn` 的构造顺序（两端段 → `SetPristine` → 主模拟）：小地图图标 → `SetPhysicsRadiusOverride(2.35)` → `MakeWaterObstaclePhysics(inst, 0.80, 2, 0.75)` → 标签 `ignorewalkableplatforms`／`event_trigger` → bank `lobster_den`／build／播 `full` → `MakeInventoryFloatable` ＋ `SetIsObstacle()` ＋ `bob_percent = 0`（浮在水面不上下浮动）→ `SetPristine`；主模拟再补掉落、可挖、可查看、刷孩子、碰撞监听与 `OnPreLoad`。[构造](../../../raw/current/scripts/prefabs/wobster_den.lua#L204-L283)

主模拟挂件：`DoTaskInTime(land_time, floater:OnLandedServer)`（`POPULATING` 时随机延迟几帧）→ `lootdropper`（`SetChanceLootTable`＋弹射参数）→ `workable`（`ACTIONS.MINE`、`SetWorkLeft(TUNING.WOBSTER_DEN_WORK)`、`savestate = true`）→ `inspectable` → `childspawner`（周期、上限、世界设置定时器、`spawnradius`、`childname`、`wateronly = true`、`SetOccupiedFn`／`SetVacateFn`）→ `StartSpawning()` → `ListenForEvent("on_collide", OnCollide)` → `DoTaskInTime(0, initialize)` → `inst.OnPreLoad = OnPreLoad`。[挂件](../../../raw/current/scripts/prefabs/wobster_den.lua#L236-L282)

- 掉落用的是**概率表**不是权重表：`GenerateLoot` 对 `chance >= 1.0` 的直接给，`< 1.0` 的各自独立掷一次，所以 `0.50` 的两条可能同时出也可能都不出。[抽取](../../../raw/current/scripts/components/lootdropper.lua#L213-L227)
- `worldsettingstimer` 由 `WorldSettings_ChildSpawner_SpawnPeriod`／`RegenPeriod` 建立，把周期变成可被世界设置暂停的外部计时器；`OnPreLoad` 里 `WorldSettings_ChildSpawner_PreLoad` 再把旧存档的剩余时间折算成计时器比例。[定时器](../../../raw/current/scripts/worldsettingsutil.lua#L87-L122)、[预载](../../../raw/current/scripts/worldsettingsutil.lua#L71-L85)、[接线](../../../raw/current/scripts/prefabs/wobster_den.lua#L188-L190)

### 名称核验

`NAMES.WOBSTER_DEN`＝**龙虾窝**（Wobster Mound），`NAMES.MOONGLASS_WOBSTER_DEN`＝**月光玻璃窝**（Moon Glass Mound）——转化后的窝叫"月光玻璃窝"，而它刷出的生物叫"月光龙虾"，两个中文名不能互换；`wobster_den_spawner_shore` 没有名字键，是世界生成标记，正常玩法看不到。[DEN](../../../raw/current/scripts/languages/chinese_s.po#L352329-L352332)、[MOONGLASS_DEN](../../../raw/current/scripts/languages/chinese_s.po#L345654-L345657)、[strings](../../../raw/current/scripts/strings.lua#L3600)（S7）

## 掉落与挖掘


| 注册 | 概率掉落表（`SetSharedLootTable` 名） | 条目 |
|---|---|---|
| `wobster_den` | `"wobster_den"` | `rocks`×3（1.00）＋`rocks`×1（0.50）＋`wobster_sheller`×2（1.00）＋`wobster_sheller`×1（0.50） |
| `moonglass_wobster_den` | `"moonglass_wobster_den"` | `moonglass`×3（1.00）＋`moonglass`×2（0.50）＋`rocks`×2（1.00）＋`wobster_moonglass`×2（1.00）＋`wobster_moonglass`×1（0.50） |

两表的源码：[普通窝](../../../raw/current/scripts/prefabs/wobster_den.lua#L39-L48)、[月光窝](../../../raw/current/scripts/prefabs/wobster_den.lua#L50-L62)。期望值：普通窝 3.5 块石头＋2.5 只龙虾，月光窝 4 块月光玻璃＋2 块石头＋2.5 只月光龙虾（按独立概率推算，未实测）。

- 挖掘用 `ACTIONS.MINE`、总量 `TUNING.WOBSTER_DEN_WORK`＝9，`savestate = true` 会存进度。[可挖](../../../raw/current/scripts/prefabs/wobster_den.lua#L249-L253)、[常量](../../../raw/current/scripts/tuning.lua#L5449)
- 两个阈值把美术切成**四段动画**（`full`／`eyes_loop`、`med`、`low`）：`FIRST_WORK_LEVEL = ceil(2*9/3) = 6`、`SECOND_WORK_LEVEL = ceil(9/3) = 3`；`workleft > 6` 时有孩子播 `eyes_loop`、没孩子播 `full`（同一档两种动画），`> 3` 播 `med`，否则播 `low`。[阈值与选段](../../../raw/current/scripts/prefabs/wobster_den.lua#L64-L84)
- 有孩子时每 5 秒尝试眨眼（`try_blink`，首次延迟随机 0–3 秒），孩子进出分别播 `eyes_pre`／`eyes_post` 过渡；这些回调来自 `childspawner` 的 `SetOccupiedFn`／`SetVacateFn`。[眨眼](../../../raw/current/scripts/prefabs/wobster_den.lua#L86-L115)、[回调注册](../../../raw/current/scripts/prefabs/wobster_den.lua#L272-L273)
- 挖到 0：`DropLoot(pt)` → `ReleaseAllChildren()` → `Remove()` → 生成 `rock_break_fx`；没挖完只更新美术。[OnWork](../../../raw/current/scripts/prefabs/wobster_den.lua#L158-L173)

## 孩子刷新

`childspawner` 参数：`SetSpawnPeriod(WOBSTER_DEN_SPAWN_PERIOD)`＝4×`seg_time`＝120 秒（±10%，即 ±12 秒）、`SetRegenPeriod(WOBSTER_DEN_REGEN_PERIOD)`＝3×`seg_time`＝90 秒（同样 ±10%，即 ±9 秒）、`SetMaxChildren(WOBSTER_DEN_MAX_CHILDREN)`＝2（默认）、`spawnradius = WOBSTER_DEN_SPAWNRADIUS`＝4、`childname` 是窝变体对应的水体龙虾、`wateronly = true` 让偏移只用 `FindSwimmableOffset` 找水面（时长按 `seg_time`＝30 秒折算，S1）。120／90 秒只是默认值：两个周期实际由 `WorldSettings_ChildSpawner_SpawnPeriod`／`RegenPeriod` 建的 `worldsettingstimer` 承载，世界设置可整体改周期或关掉——`wobsters` 档位是 never（关）、rare（spawn 8／regen 6 ×`seg_time`＝240／180 秒）、default（4／3＝120／90 秒）、often（2／1.5＝60／45 秒，改 MAX 3）、always（1／1＝30／30 秒，改 MAX 4）。[档位](../../../raw/current/scripts/worldsettings_overrides.lua#L1749-L1779)、[定时器](../../../raw/current/scripts/worldsettingsutil.lua#L87-L122)、[接线](../../../raw/current/scripts/prefabs/wobster_den.lua#L262-L263)、[参数](../../../raw/current/scripts/prefabs/wobster_den.lua#L257-L274)、[常量](../../../raw/current/scripts/tuning.lua#L5445-L5450)、[水面偏移](../../../raw/current/scripts/components/childspawner.lua#L532-L552)

- 窝的"库存"是 `childreninside`：组件构造默认 1，`SetMaxChildren(2)` 会把它改写成 `2 - 当前外放数`，所以**默认档**下新窝一开始就有 2 只在里面（`WOBSTER_DEN_MAX_CHILDREN` 的世界设置档可给 2／3／4），而且只在 `TUNING.WOBSTER_DEN_ENABLED` 为真时成立，禁用时构造里会把 `childreninside` 置 0。刷出一只就 `-1`，孩子死亡／被抓不会直接回补，要靠 `regen` 每 90 秒（默认）补 1；**库存为 0 或 `CanSpawn()` 不通过（例如窝在睡眠）时，挖毁／碰撞可能一只都放不出**。[可刷判定](../../../raw/current/scripts/components/childspawner.lua#L721-L727)、[启用门禁](../../../raw/current/scripts/prefabs/wobster_den.lua#L264-L266)、[默认库存](../../../raw/current/scripts/components/childspawner.lua#L52-L60)、[改写库存](../../../raw/current/scripts/components/childspawner.lua#L315-L321)、[再生](../../../raw/current/scripts/components/childspawner.lua#L185-L194)
- **白天停刷、黄昏起开刷（S10 已核验）**：构造时先 `StartSpawning()`，再在 `DoTaskInTime(0, initialize)` 里按 `TheWorld.state.iscaveday` 决定是否 `StopSpawning()`，随后 `WatchWorldState("startcaveday", on_day_started)`／`("stopcaveday", on_day_ended)`——即**白昼开始停刷、洞日结束（黄昏）起恢复**；两个回调都用 `DoTaskInTime(1 + math.random()*2)` 延迟 1–3 秒再切换，避免同一帧抖动。[初始化](../../../raw/current/scripts/prefabs/wobster_den.lua#L143-L156)、[切换](../../../raw/current/scripts/prefabs/wobster_den.lua#L117-L141)
- 用 `startcaveday`／`stopcaveday` 这对**洞日**事件在海洋（地表）世界同样有效：`worldstate` 的 `OnPhaseChanged` 末尾会调 `OnCavePhaseChanged`，把 `iscaveday` 等变量与地表 `isday` 同步，因此地表世界的阶段切换会一并推 `startcaveday`／`stopcaveday`。[世界状态](../../../raw/current/scripts/components/worldstate.lua#L63-L80)
- `ReleaseAllChildren` 是 `while CanSpawn() and failures < 3`：每次成功就把失败计数清零，所以正常情况会一直放到库存清空；但**连续 3 次 `SpawnChild` 失败**（例如半径内找不到可游泳落点）就提前停手，库存留在窝里。因此"挖掉／被撞"通常是同一帧把剩余孩子撒到 `spawnradius` 内的水面，但不是保证全部放出。[释放](../../../raw/current/scripts/components/childspawner.lua#L758-L793)

## 船撞

`OnCollide` 只对带 `boatphysics` 的撞击方结算伤害：`hit_velocity = floor(|boat_velocity × hit_dot_velocity| × 0.5 / boat_physics.max_velocity + 0.5)`，再以 `hit_velocity × TUNING.SEASTACK_MINE` 作为工作量提交给 `workable:WorkedBy`。伤害只在**撞击方带 `boatphysics`** 的分支里结算；`ReleaseAllChildren` 写在分支外，所以任何碰撞都会触发一次释放（不论是否挖穿、是否船）。[碰撞](../../../raw/current/scripts/prefabs/wobster_den.lua#L175-L186)、[常量](../../../raw/current/scripts/tuning.lua#L858)

- `DAMAGE_SCALE = 0.5` 与 `+0.5` 的取整意味着满速正撞时 `hit_velocity` 最大为 1，乘 `SEASTACK_MINE`＝9 正好等于总工作量 9——理论上一记满速正撞就能挖掉整个窝（推导，未实测）。[缩放](../../../raw/current/scripts/prefabs/wobster_den.lua#L175-L181)

## 月光就地改写

普通窝的 `halloweenmoonmutable:SetConversionOverrideFn(moonconversionoverridefn)` 覆盖了默认的"生成替换实体"路径：被月亮药水命中时，回调**在同一个实体上**做四件事——`inst.prefab = "moonglass_wobster_den"`、`SetBuild("lobster_den_moonglass_build")`、`lootdropper:SetChanceLootTable("moonglass_wobster_den")`、`childspawner.childname = "wobster_moonglass"`，再 `RemoveComponent("halloweenmoonmutable")`，返回 `inst, nil`（不生成新实体）。（S6）[改写](../../../raw/current/scripts/prefabs/wobster_den.lua#L193-L202)、[Mutate 分支](../../../raw/current/scripts/components/halloweenmoonmutable.lua#L28-L43)

- 已死的窝不会被改写：`Mutate` 开头检查 `health:IsDead()`；窝本身没有 `health` 组件，所以这条门禁对窝恒不成立。[门禁](../../../raw/current/scripts/components/halloweenmoonmutable.lua#L28-L31)
- 组件**两端都挂**（`fn()` 在 `basefn` 返回后无条件 `AddComponent`，客户端也拿到），但改写只由服务端的月亮药水流程触发；组件内没有 netvar 回放逻辑。[挂载点](../../../raw/current/scripts/prefabs/wobster_den.lua#L285-L292)、[Mutate](../../../raw/current/scripts/components/halloweenmoonmutable.lua#L28-L43)

## 世界生成标记

`wobster_den_spawner_shore` 没有任何组件，唯一作用是**在坐标列表里占位**：世界生成读取它的坐标，再由 `map/bunches.lua` 里的函数把它换成真正的窝。

| 环节 | 事实 | 证据 |
|---|---|---|
| 房间分布 | `OceanCoastalShore` 房间把该标记按 `distributepercent = 0.005` 撒在沿海浅水环 | [房间](../../../raw/current/scripts/map/rooms/forest/terrain_ocean.lua#L4-L13) |
| 成组参数 | `range = 12`、`min = 4`、`max = 5`、`min_spacing = 3`，`valid_tile_types` 只接受 `OCEAN_COASTAL` | [Bunches](../../../raw/current/scripts/map/bunches.lua#L40-L69) |
| 落点判定 | 参考点周围 ±4 格取 tile：`PEBBLEBEACH`／`METEOR`／`SHELLBEACH` → `moonglass_wobster_den`；非海洋 tile → `wobster_den`；其余（含无效／`ROCKY`）继续找，找不到返回 `nil` 放弃 | [判定函数](../../../raw/current/scripts/map/bunches.lua#L41-L61) |
| 替换与放置 | `placeitemoffgrids` 发现 `data.prefab` 是函数就调用它，用返回值替换后再交给 `WorldSim` 放置 | [放置](../../../raw/current/scripts/map/bunch_spawner.lua#L137-L147)、[批量入口](../../../raw/current/scripts/map/bunch_spawner.lua#L156-L184) |
| 密度修正 | `forest_map` 的 `MULTIPLY_PREFABS` 把 `wobster_den_spawner_shore` 按"陆地 spawner"口径折减（注释：它虽是海洋设置，但放置条件要求按陆地口径处理） | [倍率表](../../../raw/current/scripts/map/forest_map.lua#L149-L160) |
| 设置项映射 | 世界设置 `ocean_wobsterden` → `wobster_den_spawner_shore` | [译名表](../../../raw/current/scripts/map/forest_map.lua#L237-L241) |
| 预载与登记 | 地表 prefab 列表里直接写着 `wobster_den_spawner_shore`；`prefablist.lua` 登记的是**文件** `wobster_den`（三个 prefab 随文件一起注册），shore 这个名字本身不在表里 | [forest](../../../raw/current/scripts/prefabs/forest.lua#L360-L361)、[prefablist](../../../raw/current/scripts/prefablist.lua#L1465-L1466) |

- 因此玩家看到的每一个窝都来自这条替换链：标记只提供坐标，真正进世界的是 `wobster_den` 或 `moonglass_wobster_den`；标记工厂本身不刷任何东西。
- 读源码时要注意 `valid_tile_types = { OCEAN_COASTAL }` 与房间撒点用的 `OCEAN_COASTAL_SHORE` 不是同一个 tile：标记先撒在浅水环，随后 `checkIfValidGround` 只接受 `OCEAN_COASTAL`，最终落点偏向沿海浅水环外沿那一圈（按代码推导，未实测）。
- 标记实体本身是否留在世界里，源码没有显式移除；其 `CLASSIFIED` 与 "Non-networked" 注释说明它是世界生成期的坐标宿主，运行期行为未核验。


## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`childspawner` | 周期／上限／库存、`wateronly`＋`spawnradius`、进出窝回调 | 主模拟；`useexternaltimer` 后由 `worldsettingstimer` 计时 | [[reference/components/childspawner]]；[接线](../../../raw/current/scripts/prefabs/wobster_den.lua#L257-L274) |
| 组件：`workable` | `ACTIONS.MINE`、9 点、`savestate`；`OnWork` 处理掉落与释放 | 主模拟 | [[reference/components/workable]]；[接线](../../../raw/current/scripts/prefabs/wobster_den.lua#L249-L253) |
| 组件：`lootdropper` | 概率表＋弹射参数（`max_speed` 2、`y_speed` 14、`spawn_loot_inside_prefab`） | 主模拟；挖掉时 `DropLoot` | [[reference/components/lootdropper]]；[接线](../../../raw/current/scripts/prefabs/wobster_den.lua#L241-L247) |
| 组件：`floater` | `SetIsObstacle()` 让船撞得到、`bob_percent = 0`，落水时 `OnLandedServer` | 两端挂载、主模拟调 `OnLandedServer` | [[reference/components/floater]]；[接线](../../../raw/current/scripts/prefabs/wobster_den.lua#L227-L239) |
| 组件：`halloweenmoonmutable` | 普通窝被月亮药水命中后就地改写为月光窝 | 仅服务端、仅 `wobster_den` | [[reference/components/halloweenmoonmutable]]；[接线](../../../raw/current/scripts/prefabs/wobster_den.lua#L288-L289) |
| 数据：`worldsettingstimer` | 让世界设置能统一暂停／恢复刷新与再生计时 | 主模拟；由 `WorldSettings_ChildSpawner_*` 建立 | [定时器](../../../raw/current/scripts/worldsettingsutil.lua#L87-L104) |
| 事件：`on_collide` | 船撞扣挖掘进度并释放孩子（伤害仅船分支，释放不分撞击方） | 碰撞方带 `boatphysics` 才算伤害 | [监听](../../../raw/current/scripts/prefabs/wobster_den.lua#L276) |
| 事件：`startcaveday`／`stopcaveday` | 白昼开始停刷、洞日结束（黄昏）起开刷的开关 | 地表世界与 `isday` 同步 | [接线](../../../raw/current/scripts/prefabs/wobster_den.lua#L154-L155)、[世界状态](../../../raw/current/scripts/components/worldstate.lua#L72-L80) |
| 关联 Prefab：`wobster_sheller`／`wobster_moonglass` | 孩子；由 `childname` 决定 | 依赖表声明＋运行时 `SpawnPrefab` | [[reference/prefabs/wobster]]；[依赖](../../../raw/current/scripts/prefabs/wobster_den.lua#L17-L31) |
| 关联 Prefab：`rocks`／`moonglass`／`rock_break_fx` | 掉落物与破碎演出 | 由 `lootdropper` 与 `OnWork` 生成 | [[reference/prefabs/moonglass-materials]]；[掉落表](../../../raw/current/scripts/prefabs/wobster_den.lua#L39-L62) |
| 世界生成：`map/bunches.lua`＋`map/bunch_spawner.lua` | 把标记坐标换成真窝（函数式 `prefab`） | 仅世界生成；`IsBunchSpawner` 用于复查 | [判定函数](../../../raw/current/scripts/map/bunches.lua#L41-L61)、[替换](../../../raw/current/scripts/map/bunch_spawner.lua#L137-L147) |
| 资源：`lobster_den.zip`／`lobster_den_build.zip`／`lobster_den_moonglass_build.zip` | 两个 build 共用一套 bank；小地图图标 `wobster_den.png` | 原版资源，不随页分发 | [assets](../../../raw/current/scripts/prefabs/wobster_den.lua#L3-L15) |

## 常见陷阱与限制

- **误区：`wobster_den_spawner_shore` 是运行时刷怪器。** 它没有组件，只提供世界生成坐标；实际落进世界的是 `wobster_den`／`moonglass_wobster_den`，而且是否生成还取决于 ±4 格内有没有陆地或贝壳沙滩 tile。[标记工厂](../../../raw/current/scripts/prefabs/wobster_den.lua#L300-L309)、[判定](../../../raw/current/scripts/map/bunches.lua#L41-L61)
- **边界：挖窝会在同一帧放出孩子，但不是"保证全部"。** `OnWork` 在 `workleft <= 0` 时先 `DropLoot` 再 `ReleaseAllChildren()`，所以掉落的石头和活龙虾会同时出现；被船撞无论是否带 `boatphysics` 也会释放。`ReleaseAllChildren` 连续 3 次刷不出就停手，剩余库存随窝一起消失。[挖完](../../../raw/current/scripts/prefabs/wobster_den.lua#L158-L173)、[船撞](../../../raw/current/scripts/prefabs/wobster_den.lua#L183-L185)
- **边界：白天的"停刷"不是立即生效。** 两个昼夜回调都延迟 `1 + math.random()*2` 秒再切 `StartSpawning`／`StopSpawning`，跨昼夜瞬间看不到立刻切换。[切换](../../../raw/current/scripts/prefabs/wobster_den.lua#L117-L141)
- **边界：默认库存 2、再生慢，而且不保证放得出来。** `SetMaxChildren` 把 `childreninside` 设成"上限 - 外放数"（默认上限 2，世界设置档可改 3／4）；掏空后全靠 90 秒（默认）一次的 `regen` 补，孩子死亡不会立刻回补。**库存已是 0，或 `CanSpawn()` 因睡眠／库存等条件不通过时，挖毁或碰撞可能一只都放不出。**[上限](../../../raw/current/scripts/components/childspawner.lua#L315-L321)、[再生](../../../raw/current/scripts/components/childspawner.lua#L185-L194)
- **边界：月光窝是"改写"不是"替换"。** 同一个实体改 `prefab` 名、build、掉落表与孩子名，外部持有的实体引用仍然有效；这与陆态龙虾用 `SetPrefabMutated` 生成新实体的路径相反。[改写](../../../raw/current/scripts/prefabs/wobster_den.lua#L193-L202)、[对比](../../../raw/current/scripts/prefabs/wobster.lua#L421-L422)
- **未核验**：未运行游戏。海洋世界昼夜切换对刷新节奏的实测、标记实体是否留在世界、满速船撞一次挖穿的实战表现都未验证。

## 验证记录

- 源码审阅：2026-11-09，Steam buildid 26474873。通读 `prefabs/wobster_den.lua` 全文（两套掉落表、四段美术、`OnWork`／`OnCollide`／`OnPreLoad`、`moonconversionoverridefn`、`spawner_fn`）；`components/childspawner.lua` 的库存、`SetMaxChildren`、`DoSpawnChild` 的 `wateronly` 分支、`ReleaseAllChildren`、`OnUpdate`；`worldsettingsutil.lua` 的三个 `WorldSettings_ChildSpawner_*`；`components/worldstate.lua` 的洞日同步；`components/floater.lua` 的 `SetIsObstacle`／`OnLandedServer`；`components/halloweenmoonmutable.lua` 的 `Mutate`；`map/bunches.lua` 全文、`map/bunch_spawner.lua` 的放置与替换、`map/forest_map.lua` 的倍率与译名表、`map/rooms/forest/terrain_ocean.lua` 的沿海房间；`prefabs/forest.lua` 与 `prefablist.lua` 的登记；`tuning.lua` 的 `WOBSTER_DEN_*` 与 `SEASTACK_MINE`。
  检索范围：`wobster_den` 相关检索覆盖 prefab、组件、世界生成三处；未覆盖：模组注入、世界设置界面到实际倍率的换算细节、调试命令。
  名称核验：`NAMES.WOBSTER_DEN`＝龙虾窝、`NAMES.MOONGLASS_WOBSTER_DEN`＝月光玻璃窝；`wobster_den_spawner_shore` 没有名字键（世界生成标记，正常玩法不可见）。
- 游戏运行：未运行。刷新周期、期望掉落与船撞换算均为源码推导（`seg_time`＝30 秒、`total_day_time`＝480 秒，S1）。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 见交付回报。

## 相关链接

- 孩子与形态：[[reference/prefabs/wobster]]
- 刷新组件：[[reference/components/childspawner]]、[[reference/components/floater]]
- 掉落与转换：[[reference/components/lootdropper]]、[[reference/components/halloweenmoonmutable]]
- 月光玻璃：[[reference/prefabs/moonglass-materials]]

返回 [[reference/prefabs/index|Prefab 索引]]。
