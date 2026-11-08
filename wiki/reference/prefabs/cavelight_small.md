---
title: "光源实体（cavelight_small、cavelightmoon、chesterlight、booklight、spawnlight_multiplayer、atrium_light）"
created: "2026-11-04"
updated: "2026-11-08"
type: prefab
summary: "六个工厂、十三个注册的光源实体：洞穴相位光、月光与鼹鼠蝠巢、切斯特变身光柱、读书光、出生光与中庭供电灯的参数、生成点和清理边界。"
aliases: ["cavelight", "cavelight_small", "cavelight_tiny", "cavelight_atrium", "cavelightmoon", "cavelightmoon_small", "cavelightmoon_tiny", "chesterlight", "booklight", "spawnlight_multiplayer", "atrium_light", "atrium_light_back", "atrium_light_light", "远古灯柱", "洞穴光源", "洞穴出口光", "中庭灯"]
tags: [prefab, fx, worldgen, server-client]
size_class: complex
dst_build: "steam-buildid:26474873"
sources: ["prefabs/cavelight.lua", "prefabs/cavelightmoon.lua", "prefabs/chesterlight.lua", "prefabs/booklight.lua", "prefabs/spawnlight_multiplayer.lua", "prefabs/atrium_light.lua", "components/hideout.lua", "components/playerspawner.lua", "components/temperature.lua", "components/worldstate.lua", "prefabs/books.lua", "prefabs/chester.lua", "stategraphs/SGchester.lua", "prefabs/atrium_gate.lua", "map/static_layouts/rooms/atrium_end/atrium_end.lua", "map/static_layouts/cave_start.lua", "map/rooms/cave/rabbits.lua", "map/rooms/cave/bluemush.lua", "map/layouts.lua", "map/object_layout.lua", "prefablist.lua", "prefabs/cave.lua", "prefabs/world.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 光源实体（cavelight_small、cavelightmoon、chesterlight、booklight、spawnlight_multiplayer、atrium_light）

## 快速判断

- 职责与适用：六个工厂、十三个注册。`cavelight` 四档、`cavelightmoon` 三档、`chesterlight`、`booklight`、`spawnlight_multiplayer`、`atrium_light` 都直接设 Light，不走 `fx.lua` 通用表。[注册](../../../raw/current/scripts/prefabs/cavelight.lua#L271-L274)、[月光](../../../raw/current/scripts/prefabs/cavelightmoon.lua#L284-L286)、[中庭](../../../raw/current/scripts/prefabs/atrium_light.lua#L155-L157)
- 优先使用：洞穴常驻光选 `cavelight`；光源兼藏身处／刷怪点看 hideout 与 `cavelightmoon` spawner；一次性光柱选 `chesterlight`／`booklight`；出生光选 `spawnlight_multiplayer`；可供能、检视、阻挡的灯具选 `atrium_light`。施法光见 [[reference/prefabs/staff_castinglight|施法光源]]，矮星见 [[reference/prefabs/stafflight|矮星]]。
- 重要副作用：`cavelight` 服务端收放蝙蝠（5±4 秒）；`cavelightmoon` 普通档产鼹鼠蝠；`spawnlight` 被温度系统当无 heater 的保温源（10～50）；`atrium_light` 带 obstacle、小地图、检视与玫瑰镜燃料目标，由中庭门开关。[hideout](../../../raw/current/scripts/prefabs/cavelight.lua#L236-L241)、[温度](../../../raw/current/scripts/components/temperature.lua#L349-L360)、[中庭](../../../raw/current/scripts/prefabs/atrium_light.lua#L59-L63)
- 修改边界：相位只同步 id，客户端按 `FRAMES` 本地插值，服务端才 Enable／Hide；`chesterlight.TurnOn`／`TurnOff`、`booklight.SetDuration`／`FadeOut` 只在主模拟存在。[相位](../../../raw/current/scripts/prefabs/cavelight.lua#L112-L126)、[切斯特端](../../../raw/current/scripts/prefabs/chesterlight.lua#L79-L86)、[读书光端](../../../raw/current/scripts/prefabs/booklight.lua#L82-L93)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `cavelight`、`cavelight_small`、`cavelight_tiny`、`cavelight_atrium` | `common_fn(widthscale)` 建 Transform／AnimState／SoundEmitter／Light／Network；前三档只差 `widthscale`（1／.5／.2），`atrium` 为 `.6` 并另听 `charliecutscene`。主模拟加 hideout（5±4 秒）与 `onspawned`。 | `widthscale` 同时缩放 Transform 与半径；`atrium` 过场强制白天、结束恢复。[工厂](../../../raw/current/scripts/prefabs/cavelight.lua#L192-L247)、[变体](../../../raw/current/scripts/prefabs/cavelight.lua#L249-L269) |
| `cavelightmoon`、`cavelightmoon_small`、`cavelightmoon_tiny` | `common_fn(widthscale, is_spawner)` 建 Transform／AnimState／Light／Network；normal `.7` 且 spawner，small `.4`、tiny `.2` 无。主模拟普通档挂 `spawner`／`playerprox`。 | 青色 `SetMultColour(.3,.5,1)`；小／微档白天切 `off`，normal 白天仍亮；满月由 `iscavefullmoon` 驱动。[工厂](../../../raw/current/scripts/prefabs/cavelightmoon.lua#L210-L270)、[注册](../../../raw/current/scripts/prefabs/cavelightmoon.lua#L272-L286) |
| `chesterlight` | 独立 `fn`；Light 半径 0、强度 `.9`、衰减 `.3`、色 (180,195,150)；`net_tinybyte _lightframe` 与 `net_bool _islighton` 共用 `lightdirty`。 | TurnOn 播 `on`→`idle_loop`；TurnOff 播 `off` 并在 `animover` 移除。[工厂](../../../raw/current/scripts/prefabs/chesterlight.lua#L53-L91)、[曲线](../../../raw/current/scripts/prefabs/chesterlight.lua#L6-L51) |
| `booklight` | 独立 `fn`；固定 Light 3／.85／.3／(180,195,150)，无 netvar；主模拟 `SetDuration` 排 `FadeOut`，`OnSave`／`OnLoad` 以 `time_remaining` 续时；`FadeOut` 设 `persists=false`。 | 淡出每帧降半径／强度、升衰减，播 `off` 后移除；睡眠中 `IsAsleep()` 直接 `Remove`；客户端不跑淡出。[工厂](../../../raw/current/scripts/prefabs/booklight.lua#L54-L98)、[淡出](../../../raw/current/scripts/prefabs/booklight.lua#L15-L52) |
| `spawnlight_multiplayer` | 独立 `fn`，只有 Transform／Light／Network，无 AnimState；`net_float _pulsetime` 带 `pulsetimedirty`；两端每 .1 秒脉动，主模拟每 30 秒同步。 | `cycles` 变化触发淡出（负 `_pulsetime`）；`persists=false`；洞穴内 10 秒后检查查理竞技场。[工厂](../../../raw/current/scripts/prefabs/spawnlight_multiplayer.lua#L79-L123)、[脉动](../../../raw/current/scripts/prefabs/spawnlight_multiplayer.lua#L1-L70) |
| `atrium_light`、`atrium_light_back`、`atrium_light_light` | `fn` 建 Transform／AnimState／Light／MiniMap／Network 并 `MakeObstaclePhysics(.45)`；主模拟生成 `_back`／`_light` 并 `SetParent`，客户端 `OnEntityReplicated` 加入 `highlightchildren`。官方名「远古灯柱」（`STRINGS.NAMES.ATRIUM_LIGHT`）。 | 本体 Light 8／.65／.9／(200,140,140)，初始关闭；子实体无 Light（`_light` 仅动画）、`persists=false`。[工厂](../../../raw/current/scripts/prefabs/atrium_light.lua#L16-L90)、[子实体](../../../raw/current/scripts/prefabs/atrium_light.lua#L99-L157)、[名称](../../../raw/current/scripts/languages/chinese_s.po#L338929-L338932) |

### 光照参数与曲线

| 注册组 | 相位或状态：半径／强度／衰减（颜色，过渡秒） | 更新与动画 | 证据 |
|---|---|---|---|
| `cavelight` | day 5／.85／.3（180,195,150，2）；dusk 5／.6／.6（91,164,255，4）；night 0／0／1（0,0,0，6）；fullmoon 5／.6／.6（131,194,255，4） | 半径再乘 `widthscale`；每 `FRAMES` 按 time 线性插值；`idle_loop` 只播一次，动画色 `OverrideMultColour(colour×.5)`。 | [参数](../../../raw/current/scripts/prefabs/cavelight.lua#L14-L60) |
| `cavelightmoon` | day 2／.2／.9（10,120,255，2）；dusk 3／.5／.6（40,164,255，4）；night 4／.7／.5（90,195,255，6）；fullmoon 6／.9／.3（120,225,255，4）；off 0／0／1（0,0,0，6） | normal 用 day，小／微档 day→off；半径再乘 `widthscale`，按 time 插值；底色青色 `SetMultColour`，相位 tint 再 `OverrideMultColour`。 | [参数](../../../raw/current/scripts/prefabs/cavelightmoon.lua#L13-L60) |
| `chesterlight` | 0→5 半径（6 帧 × 5/6）／.9／.3（180,195,150） | TurnOn 播 `on`→`idle_loop`；TurnOff 播 `off`，退完后 `animover` 移除。 | [曲线](../../../raw/current/scripts/prefabs/chesterlight.lua#L6-L28) |
| `booklight` | 3／.85／.3（180,195,150） | 每帧半径 −3/7、强度 −.85/7、衰减 +.3/7，播 `off` 后移除。 | [淡出](../../../raw/current/scripts/prefabs/booklight.lua#L20-L42) |
| `spawnlight_multiplayer` | `Lerp(4,5,s)`／`Lerp(.8,.7,s)`／`Lerp(.8,.7,s)`（223,208,69） | `s=abs(sin(PI*(timealive+offset)*.05))`；`_fadek` 平方缩放，0 后 1 秒移除。 | [脉动](../../../raw/current/scripts/prefabs/spawnlight_multiplayer.lua#L24-L41) |
| `atrium_light` | 8／.65／.9（200,140,140） | 初始关闭；供电播 `idle_active`、`light_turn_on`→`light_idle_active`，断电 `light_turn_off` 后复位。 | [初值](../../../raw/current/scripts/prefabs/atrium_light.lua#L35-L39) |

## 关联机制导航

公共底座为 Transform／Light／Network；`cavelight` 与 `booklight` 加 SoundEmitter，`atrium_light` 加 MiniMap 与 obstacle；除 `spawnlight` 外都有 AnimState。

### 同步、组件与父挂

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| `Replica：cavelight._lightphase` | `cavelight` 与 `cavelightmoon` 都用 `net_tinybyte` 只发相位 id；服务端写 id，客户端听 `lightphasedirty` 后本地插值。 | 每帧光变不占网络；相位表两端一致。 | [cavelight](../../../raw/current/scripts/prefabs/cavelight.lua#L112-L228)、[moon](../../../raw/current/scripts/prefabs/cavelightmoon.lua#L121-L246) |
| `Replica：chesterlight／spawnlight` | chesterlight 用 `_lightframe`＋`_islighton` 共一个 dirty；spawnlight 用 `net_float _pulsetime`，负值=淡出，客户端据此算相位。 | 前者 `TurnOn`／`TurnOff` 仅服务端；后者主模拟每 30 秒 `set` 真值、其余 `set_local`。 | [切斯特](../../../raw/current/scripts/prefabs/chesterlight.lua#L73-L82)、[出生光](../../../raw/current/scripts/prefabs/spawnlight_multiplayer.lua#L8-L70) |
| `组件：hideout` | `cavelight` 主模拟挂 hideout，`SetSpawnPeriod(5,4)`；释放时 `onspawned` 推 `fly_back`。 | 只有服务端；白天 StartSpawning，其他相位 Stop；存档保存居民与 `spawning`。 | [[reference/components/hideout|hideout]]；[配置](../../../raw/current/scripts/prefabs/cavelight.lua#L188-L241)、[存档](../../../raw/current/scripts/components/hideout.lua#L80-L115) |
| `组件：spawner／playerprox／worldsettingstimer` | `cavelightmoon` 普通档 `Configure("molebat", MOLEBAT_ALLY_COOLDOWN*2)`，`playerprox` 5／10；玩家靠近且无待生成时排 5～10 秒。 | 只有服务端；读档由 `WorldSettings_Spawner_PreLoad` 换算剩余计时。 | [[reference/components/spawner|spawner]]、[[reference/components/playerprox|playerprox]]；[配置](../../../raw/current/scripts/prefabs/cavelightmoon.lua#L206-L267) |
| `组件：inspectable／hauntable／roseinspectable` | `atrium_light` 主模拟挂 inspectable（ON／OFF）、`MakeHauntableWork` 与 `MakeRoseTarget_CreateFuel_IncreasedHorror`。 | 只有服务端；玫瑰镜燃料目标是玩法，不是纯 FX。 | [[reference/components/inspectable|inspectable]]、[[reference/components/hauntable|hauntable]]；[挂载](../../../raw/current/scripts/prefabs/atrium_light.lua#L59-L63) |
| `父挂：atrium_light_back／_light` | 主模拟 `SpawnPrefab` 后 `SetParent` 到本体；客户端 `OnEntityReplicated` 把子实体插入父级 `highlightchildren`。 | 子实体 `persists=false`，随本体重建。 | [父挂](../../../raw/current/scripts/prefabs/atrium_light.lua#L51-L57)、[回调](../../../raw/current/scripts/prefabs/atrium_light.lua#L92-L97) |

### 生成点与依赖声明

| 生成者／路径 | 触发条件与对象 | 关系 | 证据 |
|---|---|---|---|
| 洞穴房间、静态布局与 areas 覆写 | 房间 `distributeprefabs` 直接铺 `cavelight`／`_small`／`_tiny`；静态布局的 `type` 字段也在洞穴出口、起始区等直接写注册名；另有第三类：`map/layouts.lua` 给 EvergreenSinkhole／GrassySinkhole／PondSinkhole 配 `areas.lights={"cavelight","cavelight"}`，由 `map/object_layout.lua` 消费（sinkhole／greenmush／swamp 布局使用）。 | 实际生成。 | [兔子房](../../../raw/current/scripts/map/rooms/cave/rabbits.lua#L110-L112)、[起始区](../../../raw/current/scripts/map/static_layouts/cave_start.lua#L81-L93)、[areas 配置](../../../raw/current/scripts/map/layouts.lua#L733-L751)、[消费](../../../raw/current/scripts/map/object_layout.lua#L184-L198) |
| `cavelight_atrium` | 只在中庭终点布局出现；`atrium` 工厂另监听 `charliecutscene`，过场强制白天光、结束恢复。 | 实际生成。 | [布局](../../../raw/current/scripts/map/static_layouts/rooms/atrium_end/atrium_end.lua#L162-L168)、[监听](../../../raw/current/scripts/prefabs/cavelight.lua#L261-L268) |
| `cavelightmoon` | 蓝蘑菇林等洞穴房间低概率铺 0.003；普通档同时是鼹鼠蝠 spawner。 | 实际生成；不属于森林世界。 | [蓝蘑菇林](../../../raw/current/scripts/map/rooms/cave/bluemush.lua#L149-L151) |
| `chesterlight` | `SGchester` 的 `transition` 在切斯特位置生成并 `TurnOn`，退出状态 `TurnOff`；切斯特 `prefabs` 表声明依赖。 | 实际生成；仅变身过场存在。 | [生成](../../../raw/current/scripts/stategraphs/SGchester.lua#L290-L308)、[依赖](../../../raw/current/scripts/prefabs/chester.lua#L27-L34) |
| `booklight` | `book_light` 与 `book_light_upgraded` 在读者位置生成，时长 `TOTAL_DAY_TIME/2` 与 `TOTAL_DAY_TIME*2`。 | 实际生成；`deps` 只是依赖声明。 | [普通版](../../../raw/current/scripts/prefabs/books.lua#L757-L769) |
| `spawnlight_multiplayer` | 出生落点不是白天且 4 距离内无 `spawnlight` 标签实体时生成；温度系统另把它当保温源。 | 实际生成；由 playerspawner 触发。 | [生成](../../../raw/current/scripts/components/playerspawner.lua#L273-L274) |
| `atrium_light` | 中庭终点等布局放置；门 `OnKeyGiven` 推 `atriumpowered(true)`、`OnKeyTaken` 推 false；读档 `OnLoadPostPass` 在已供能时重走 `OnKeyGiven`。 | 实际生成；灯本体是世界设施，`back`／`light` 为子实体。 | [布局](../../../raw/current/scripts/map/static_layouts/rooms/atrium_end/atrium_end.lua#L118-L129)、[供能](../../../raw/current/scripts/prefabs/atrium_gate.lua#L188-L236)、[读档](../../../raw/current/scripts/prefabs/atrium_gate.lua#L583-L605) |
| 预载／依赖（非生成） | `prefablist.lua` 列注册；`cave.lua` 列洞穴世界依赖；`world.lua` 列森林依赖。 | 只保证可用，不生成。 | [总表](../../../raw/current/scripts/prefablist.lua#L262-L263)、[洞穴](../../../raw/current/scripts/prefabs/cave.lua#L12-L15)、[森林](../../../raw/current/scripts/prefabs/world.lua#L197-L197) |

## 常见陷阱与限制

- **误区：`cavelight` 的满月档在洞穴可用**。洞穴 `OnCavePhaseChanged` 只写 `cavephase` 与 `iscave*` 变量，不写 `isfullmoon`；`cavelight` 却用 `isfullmoon` 选 fullmoon，洞穴内不可达。[洞穴分支](../../../raw/current/scripts/components/worldstate.lua#L64-L68)、[选档](../../../raw/current/scripts/prefabs/cavelight.lua#L137-L147)
- **`cavelightmoon` 首次构造／读档初值缺口**：`OnCavePhase` 用 `iscavefullmoon`，`OnInit`（每次构造 `DoTaskInTime(0)`）初值仍用 `isfullmoon`；洞穴满月读档先取 night 参数，等下一次相位或满月变化才切 fullmoon。[初值](../../../raw/current/scripts/prefabs/cavelightmoon.lua#L140-L175)
- **`booklight` 淡出只在服务端**：`FadeOut` 在主模拟分支之后且无 netvar，客户端保持原半径直到实体移除。[分支](../../../raw/current/scripts/prefabs/booklight.lua#L80-L93)、[淡出](../../../raw/current/scripts/prefabs/booklight.lua#L20-L42)
- **`atrium_light_light` 不是光源**：本体 Light 才照亮，`_light` 没有 Light，只是动画与装饰；改半径／开关要改父实体。[子实体](../../../raw/current/scripts/prefabs/atrium_light.lua#L127-L153)
- **`spawnlight` 的 `_pulsetime` 不是存活时间**：负值表示淡出，主模拟每 30 秒才写一次真值。`chesterlight`／`booklight` 的方法只在一端存在，跨端调用前先判 `TheWorld.ismastersim`。[同步](../../../raw/current/scripts/prefabs/spawnlight_multiplayer.lua#L8-L70)、[端](../../../raw/current/scripts/prefabs/chesterlight.lua#L79-L86)

## 验证记录

- 源码审阅：2026-11-04，Steam buildid 26335394；完整读取六个工厂文件，核对十三个注册、Light 参数、netvar、父子挂与清理；全文反查生成点与依赖。
- 名称：`STRINGS.NAMES.ATRIUM_LIGHT` 为官方名「远古灯柱」／Ancient Beacon（[po](../../../raw/current/scripts/languages/chinese_s.po#L338929-L338932)）；其余 12 个注册名在简中 po 的 `STRINGS.NAMES` 均未命中，标题与别名按已核验效果描述＋注册名处理，非官方译名。
- 分档说明：六工厂、十三注册、世界生成三类入口与两套相位状态机需在一页内对照，按 `complex` 记录；实测 90 行／约 16.6 KiB，在 400 行／32 KiB 内。
- 未覆盖：未解包动画资源，未运行游戏验证光色、客户端插值与中庭灯客户端可见性；满月档不可达、读档初值、读书光客户端淡出均为源码推导，未运行复现。无 Lua 片段。

返回 [[reference/prefabs/fx-light-index|灯光、火焰与音效代理]]。
- 2026-11-08 build 26474873：本页随版本迁移升版（机械沿用；引用区间未受本次源码行号位移影响），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-06）。
