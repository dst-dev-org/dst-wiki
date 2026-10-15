---
title: "海面波浪与浪花（wave_med）"
created: "2026-11-04"
updated: "2026-11-04"
type: prefab
summary: "七个注册名覆盖海面表现：wave_med 为可按 SG 升起／消散、碰撞推船并加湿周围实体的网络波浪，wave_splash 是其落点水花；wave_shimmer 四档与 wave_shore 由客户端 wavemanager 按海深与岸线本地生成。"
aliases: ["wave_med", "wave_splash", "wave_shimmer", "wave_shimmer_med", "wave_shimmer_deep", "wave_shimmer_flood", "wave_shore", "海面波浪", "浪花", "岸边浪"]
tags: [prefab, fx, server-client]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/wave.lua", "prefabs/wave_shimmer.lua", "prefabs/wave_shore.lua", "stategraphs/SGwave.lua", "components/wavemanager.lua", "components/boattrail.lua", "ocean_util.lua", "componentutil.lua", "prefabs/forest.lua", "prefabs/malbatross.lua", "stategraphs/SGmalbatross.lua", "prefabs/oceanwhirlportal.lua", "prefabs/gnarwail_horn.lua", "stategraphs/SGcrabking.lua", "stategraphs/SGgnarwail.lua", "stategraphs/SGgrassgator.lua", "prefabs/oceantree_pillar.lua", "prefabs/boatrace_start.lua", "prefabs/trophyscale_fish.lua", "stategraphs/SGsalty_dog.lua", "prefabs/cannonballs.lua", "prefabs/mortarball.lua", "prefabs/trident.lua", "prefablist.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 海面波浪与浪花（wave_med）

## 快速判断

- 职责与适用：`wave_med`／`wave_splash` 是带网络组件的波浪与水花实体，前者有碰撞、施力与加湿；`wave_shimmer` 四档与 `wave_shore` 是无网络、`CLASSIFIED` 的本地波光，覆盖浅滩、涌浪、深水与岸线。[波浪](../../../raw/current/scripts/prefabs/wave.lua#L91-L196)、[波光](../../../raw/current/scripts/prefabs/wave_shimmer.lua#L10-L75)、[岸浪](../../../raw/current/scripts/prefabs/wave_shore.lua#L35-L70)
- 优先使用：需要推船、弹开漂浮物或打湿目标时用 `SpawnAttackWave(s)`（默认 `wave_med`）；只要视觉波纹时按 wavemanager 的本地 `SpawnPrefab`＋`SetPosition` 模式（落点式），不要加网络。
- 重要副作用：`wave_med` 只在 `waveactive` 时对 `boatphysics` 施力；碰撞／触岸／被弹道命中时 `DoSplash` 删自身、生成 `wave_splash`、给半径 4 内可加湿实体各加 9 点湿度并落 `splash`，激活期每 10 帧把半径 2 内物品／海带按自身速度弹开。[DoSplash](../../../raw/current/scripts/prefabs/wave.lua#L18-L33)、[碰撞](../../../raw/current/scripts/prefabs/wave.lua#L35-L45)、[推物](../../../raw/current/scripts/prefabs/wave.lua#L65-L85)
- 修改边界：`boattrail` 只在非专用端挂载，仅生成航迹水花，不参与玩法判定；`wave_shimmer`／`wave_shore` 不进存档、不走网络，原版专用服务器不生成；7名在简中 `.po` 无 `STRINGS.NAMES` 键，中文标题为效果描述、非官方译名。[尾迹](../../../raw/current/scripts/prefabs/wave.lua#L122-L128)、[生成器](../../../raw/current/scripts/components/wavemanager.lua#L78-L104)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `wave_med` | `med_fn`：八向；build `wave`／bank `wave_ripple`；`TintByOceanTile`；球形物理半径 1（`OBSTACLES` 组，`SetCollides(false)` 仍收回调）；标签 `scarytoprey`／`wave`／`FX`；非专用端挂 `boattrail`（rate 2.5、radius 1、scale 0.75）；主模拟端 `persists=false`、0.5 秒触岸检测、10 帧推物、睡眠即删、`SGwave`、循环水声。 | 有网络与玩法作用的波浪实体。[工厂](../../../raw/current/scripts/prefabs/wave.lua#L91-L156) |
| `wave_splash` | `wavesplash_fn`：build／bank `splash_water_rot`、anim `burst`、`SetScale(0.7)`、四向、`FX`；非专用端播 `splash/small`；主模拟端随机 `SetTime(math.random()/3)`，`animover` 或 动画长度＋0.1 秒自删。 | 一次性水花；被 `wave_med` 的 `prefabs` 表列为依赖。[工厂](../../../raw/current/scripts/prefabs/wave.lua#L158-L193)、[依赖](../../../raw/current/scripts/prefabs/wave.lua#L11-L14) |
| `wave_shimmer` | `commonfn`：Transform／AnimState、`CLASSIFIED`、海洋混色 0.8、`LAYER_BACKGROUND`＋`OCEAN_WAVES` 排序、`FX`／`NOCLICK`／`NOBLOCK`／`ignorewalkableplatforms`、客户端 `AddClientSleepable`、睡眠／`animover` 删除、`persists=false`；build `wave_shimmer`、bank `shimmer`、anim `idle`（单次）。 | 无 `AddNetwork` 的浅滩波光。[公共壳](../../../raw/current/scripts/prefabs/wave_shimmer.lua#L10-L38)、[浅滩](../../../raw/current/scripts/prefabs/wave_shimmer.lua#L40-L46) |
| `wave_shimmer_med` | 同公共壳；build `wave_shimmer_med`、bank `shimmer`。 | 与浅滩同 bank、不同 build。[工厂](../../../raw/current/scripts/prefabs/wave_shimmer.lua#L48-L54) |
| `wave_shimmer_deep` | 同公共壳；build `wave_shimmer_deep`、bank `shimmer_deep`。 | 深水换独立 bank。[工厂](../../../raw/current/scripts/prefabs/wave_shimmer.lua#L56-L62) |
| `wave_shimmer_flood` | 同公共壳；build／bank 均为 `wave_shimmer_flood`。 | 注册存在，但全 scripts 未检出生成点。[工厂](../../../raw/current/scripts/prefabs/wave_shimmer.lua#L64-L70) |
| `wave_shore` | `fn`：同公共壳，但 build／bank `wave_shore`、初始 `idle_small`、`ANIM_ORIENTATION.OnGround`，并挂 `SetAnim`；管理器设位／转向后调用 `SetAnim`，按左右 45°／90° 地块组合选 `idle_big`／`idle_med`（向岸侧移半格）／随机 `idle_small1-4`。 | 岸边浪朝向由 `SetAnim` 决定，工厂本身不探测地形。[工厂](../../../raw/current/scripts/prefabs/wave_shore.lua#L35-L70)、[选向](../../../raw/current/scripts/prefabs/wave_shore.lua#L9-L33) |

三个文件都在 `PREFABFILES`（prefablist.lua）中登记，7 名随文件注册。[登记](../../../raw/current/scripts/prefablist.lua#L1417-L1419) 简中 `chinese_s.po` 定向检索仅 `STRINGS.NAMES.WAVEYJONES` 含 WAVE，7 名均无名称键。

## 关联机制导航

### 生成点（全 scripts 反查）

| 注册名 | 生成者与触发条件 | 端 | 证据 |
|---|---|---|---|
| `wave_med` | `SpawnAttackWaves` 的默认 `wavePrefab`（`SpawnAttackWave` 为单发包装）；仅 `Map:IsPassableAtPoint` 为假处生成，并设位／转向／电机速度／`idle_time`。调用：Malbatross 12＋直呼 3、Crab King 8＋8、Gnarwail／Grass Gator／Ocean Tree Pillar／Boat Race 各 6、Horn 2、Whirl Portal 2–3。 | 服务端 SG／prefab 回调 | [helper](../../../raw/current/scripts/ocean_util.lua#L30-L76)、[Malbatross](../../../raw/current/scripts/prefabs/malbatross.lua#L234-L246)、[直呼](../../../raw/current/scripts/stategraphs/SGmalbatross.lua#L68-L74)、[Crab King](../../../raw/current/scripts/stategraphs/SGcrabking.lua#L499)、[Gnarwail](../../../raw/current/scripts/stategraphs/SGgnarwail.lua#L94-L97)、[Grass Gator](../../../raw/current/scripts/stategraphs/SGgrassgator.lua#L312)、[Ocean Tree Pillar](../../../raw/current/scripts/prefabs/oceantree_pillar.lua#L326)、[Boat Race](../../../raw/current/scripts/prefabs/boatrace_start.lua#L346)、[Horn](../../../raw/current/scripts/prefabs/gnarwail_horn.lua#L52-L66)、[Whirl Portal](../../../raw/current/scripts/prefabs/oceanwhirlportal.lua#L44-L51) |
| `wave_splash` | `wave_med` 的 `DoSplash`；Gnarwail Horn 每条浪成功后各落 1；Trophy Scale Fish 被锤毁且未烧焦；Salty Dog `summon`／`desummon`。 | 服务端生成；声效仅非专用端 | [DoSplash](../../../raw/current/scripts/prefabs/wave.lua#L18-L33)、[Horn](../../../raw/current/scripts/prefabs/gnarwail_horn.lua#L52-L66)、[Trophy](../../../raw/current/scripts/prefabs/trophyscale_fish.lua#L192-L194)、[Salty Dog](../../../raw/current/scripts/stategraphs/SGsalty_dog.lua#L159)、[退场](../../../raw/current/scripts/stategraphs/SGsalty_dog.lua#L200-L203) |
| `wave_shimmer` | wavemanager 的 `TrySpawnWavesOrShore`：`GetWaveBearing` 返回 `true`（近岸无陆地）时；用于 `OCEAN_COASTAL_SHORE`／`COASTAL`／`BRINEPOOL`／`WATERLOG` 四类地皮。 | 非专用端本地 | [选择](../../../raw/current/scripts/components/wavemanager.lua#L78-L90)、[地皮表](../../../raw/current/scripts/components/wavemanager.lua#L109-L118) |
| `wave_shimmer_med` | 同管理器：`OCEAN_SWELL` 且 `IsSurroundedByWater(...,4)`。 | 非专用端本地 | [生成](../../../raw/current/scripts/components/wavemanager.lua#L92-L97)、[触发项](../../../raw/current/scripts/components/wavemanager.lua#L114) |
| `wave_shimmer_deep` | 同管理器：`OCEAN_ROUGH`／`OCEAN_HAZARDOUS` 且 `IsSurroundedByWater(...,5)`。 | 非专用端本地 | [生成](../../../raw/current/scripts/components/wavemanager.lua#L99-L104)、[触发项](../../../raw/current/scripts/components/wavemanager.lua#L115-L116) |
| `wave_shimmer_flood` | 全 scripts 未检出引用；仅注册和资源声明。 | — | [注册](../../../raw/current/scripts/prefabs/wave_shimmer.lua#L64-L75) |
| `wave_shore` | 同一管理器近岸分支：`GetWaveBearing` 返回角度时生成，`SetRotation(bearing)`＋`SetAnim()`。 | 非专用端本地 | [选择](../../../raw/current/scripts/components/wavemanager.lua#L78-L90) |

检索范围：`raw/current/scripts/` 全目录反查 7 名，`SpawnAttackWave(s)` 逐调用点核对；动态拼名只有 `wavePrefab or "wave_med"`，调用点均传 `nil` 或 `"wave_med"`；`map/`、`scenarios/` 无命中，世界生成不静态布点。`prefabs` 数组（malbatross／cannonballs 等）仅预载，不证明生成。

### 直接协作

| 类型／对象 | 本 Prefab 如何使用 | 端／条件 | 证据 |
|---|---|---|---|
| 生命周期：`SGwave` | `rise` 第 10 帧置 `waveactive=true`；`instant_rise` 立即置真；`idle` 超时为 `idle_time or 5`；`lower` 第 20 帧置假、`animover` 删除。 | 主模拟端 | [状态图](../../../raw/current/scripts/stategraphs/SGwave.lua#L12-L90)、[写入](../../../raw/current/scripts/ocean_util.lua#L62-L66) |
| 生成器：`wavemanager` | forest 世界实体在非专用端挂载；围绕玩家按 30–75 半径、1–1.5 倍速率、每类地皮约 80 个/秒尝试；`canSpawn` 检查地皮与 blocker。 | 非专用端本地 | [安装](../../../raw/current/scripts/prefabs/forest.lua#L626-L636)、[构造](../../../raw/current/scripts/components/wavemanager.lua#L106-L128)、[更新](../../../raw/current/scripts/components/wavemanager.lua#L164-L196) |
| 染色：`TintByOceanTile` | 按地块 `tile_info.wavetint` 设 `SetMultColour`；`GetTileInfo` 无该地块信息时移除实体；`wave_splash` 仅经 `DoSplash` 获得染色。 | 生成后下一帧 | [实现](../../../raw/current/scripts/ocean_util.lua#L264-L280) |
| 弹道命中 | cannonballs／mortarball／trident 命中 `waveactive` 的浪时调用 `wave:DoSplash()`，提前结束浪并触发水花与加湿。 | 服务端命中回调 | [炮弹](../../../raw/current/scripts/prefabs/cannonballs.lua#L122-L123)、[迫击炮](../../../raw/current/scripts/prefabs/mortarball.lua#L140-L141)、[三叉戟](../../../raw/current/scripts/prefabs/trident.lua#L127-L128) |
| 关联 Prefab：`splash` | `DoSplash` 给每个可加湿实体再落一个基础水花；本文件依赖表只列 `wave_splash`，`splash` 由通用 FX 注册提供。 | 服务端 | [依赖](../../../raw/current/scripts/prefabs/wave.lua#L11-L14)、[[reference/prefabs/splash]] |

## 典型接入

### 在自定义海面事件中放一道可推船的浪

- 前提与选择：在服务端 SG 状态或组件回调中调用；位置取宿主世界坐标，helper 沿角度偏移 1.5 并在不可通行点检查后才生成；默认 `wave_med`，换外观才传第 4 参。
- 生效与撤销：`idle_time` 到期后 `lower`→`animover` 自删；被弹道命中、撞岸或睡眠会提前 `DoSplash`／`Remove`；`persists=false`，不存档。
- 原版依据：[SpawnAttackWaves](../../../raw/current/scripts/ocean_util.lua#L30-L72)、[Malbatross 批量调用](../../../raw/current/scripts/stategraphs/SGmalbatross.lua#L41)。

```lua
-- 服务端：沿宿主朝向偏移 1.5 格放一道默认 wave_med
local function SpawnRogueWave(inst)
    local x, _, z = inst.Transform:GetWorldPosition()
    SpawnAttackWave(Vector3(x, 0, z), inst.Transform:GetRotation(), 3, nil, 2, true)
end
```

- 调整与验证：参数为 `position, rotation, waveSpeed, wavePrefab, idleTime, instantActive`；`waveSpeed` 写入 `Physics:SetMotorVel`（默认 6），`wavePrefab` 为 `nil` 时用 `wave_med`，`idleTime` 默认 5 秒，`instantActive` 为真时直接进 `instant_rise`，否则从 `rise` 起播；helper 返回是否至少生成 1 道浪。

## 常见陷阱与限制

- **误区：`wave_med` 是纯表现**：它有物理碰撞、按 `waveactive` 给 `boatphysics` 施力、给范围内实体加湿并弹开物品／海带；只要视觉时改用 `wave_shimmer` 或 `wave_splash`。[碰撞](../../../raw/current/scripts/prefabs/wave.lua#L35-L45)、[推物](../../../raw/current/scripts/prefabs/wave.lua#L65-L85)
- **边界：`DoSplash` 是复合收尾**：碰撞回调在未激活时对非 `wave` 对象也会触发；触岸、弹道命中同样走这里，调用一次就删浪、生成 `wave_splash`、逐实体加湿并落 `splash`。[DoSplash](../../../raw/current/scripts/prefabs/wave.lua#L18-L33)、[触岸](../../../raw/current/scripts/prefabs/wave.lua#L47-L55)
- **边界：睡眠即删**：`wave_med` 设 `OnEntitySleep = inst.Remove` 且 `persists=false`，进入睡眠会提前消失，不能当持久海面物件。[清理](../../../raw/current/scripts/prefabs/wave.lua#L136-L143)
- **边界：`boattrail` 只生成 `boat_water_fx`**：rate 2.5／radius 1／scale 0.75 是尾迹表现参数，不改变碰撞、湿度或推物。[挂载](../../../raw/current/scripts/prefabs/wave.lua#L122-L128)
- **边界：`wave_shimmer`／`wave_shore` 是客户端本地实体**：无 `AddNetwork`、带 `CLASSIFIED`，专用服务器没有 wavemanager，客户端各自生成，不进存档。[公共壳](../../../raw/current/scripts/prefabs/wave_shimmer.lua#L10-L38)、[安装分支](../../../raw/current/scripts/prefabs/forest.lua#L626-L636)
- **边界：直接生成 `wave_shore` 不带朝向**：工厂只播初始 `idle_small`，`idle_big`／`idle_med` 与半格移位都在 `SetAnim` 里，只有 wavemanager 会调用。[工厂](../../../raw/current/scripts/prefabs/wave_shore.lua#L35-L70)、[选向](../../../raw/current/scripts/prefabs/wave_shore.lua#L9-L33)
- **边界：`wave_shimmer_flood` 无生成点**：只注册并声明资源，不能当"洪水波光"的现成入口。[资源](../../../raw/current/scripts/prefabs/wave_shimmer.lua#L1-L8)、[flood](../../../raw/current/scripts/prefabs/wave_shimmer.lua#L64-L75)
- **边界：引擎海洋浪是另一套系统**：forest 的 `AddWaveComponent` 是全局海面网格，与本页 FX 无关。[海洋渲染](../../../raw/current/scripts/prefabs/forest.lua#L611-L619)

## 验证记录

- 源码审阅：2026-11-04，Steam buildid 26335394。全文读取 `wave.lua` 196 行、`wave_shimmer.lua` 75 行、`wave_shore.lua` 70 行、`SGwave.lua` 90 行、`wavemanager.lua` 202 行、`boattrail.lua` 70 行；定向读取 `ocean_util.lua` 29–76、`componentutil.lua` 2399–2408、`forest.lua` 611–636。全 `raw/current/scripts` 反查 7 名与 `SpawnAttackWave(s)`：动态拼名仅 `wavePrefab or "wave_med"`，调用点均传 `nil` 或 `"wave_med"`；`map/`／`scenarios/` 无命中。简中 `chinese_s.po` 定向检索仅 `WAVEYJONES` 含 WAVE 的 `STRINGS.NAMES` 键，7 名无官方名。未覆盖：动画包内容／时长／循环、实际视觉与音效、联机端动画同步、`wave_shimmer_flood` 的潜在动态生成、各 Boss 状态机完整门禁。
- 游戏运行：未运行。
- 结构检查：validate_page 通过（104 行、standard 档）；validate_source_links --strict 通过（67 引用／22 文件、0 错 0 警）；parser_probe --syntax-only 通过。

## 相关链接

- [[reference/prefabs/boat_water_fx|船尾航迹水花（boat_water_fx）]]、[[reference/components/boattrail|boattrail]]、[[reference/components/wavemanager|wavemanager]]
- [[reference/prefabs/ocean_splash_med1|海面水花特效族（ocean_splash_med1）]]、[[reference/prefabs/splash|落水水花（splash 族）]]
- 返回 [[reference/prefabs/fx-world-index|特效 Prefab 索引·生物与环境]]。
