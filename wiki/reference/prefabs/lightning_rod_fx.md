---
title: "避雷针电弧动画（lightning_rod_fx）"
created: "2026-10-30"
updated: "2026-11-01"
type: prefab
summary: "避雷针充电期间的电弧标记：雷击充电／读档恢复、每个世界日终与取电三种 dozap 触发在避雷针位置落点生成；zap 声由避雷针本体播放，表现不参与充电结算。"
aliases: ["lightning_rod_fx", "避雷针电弧", "避雷针特效", "避雷针动画", "rod 特效"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/fx.lua", "fx.lua", "prefabs/lightningrod.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 避雷针电弧动画（lightning_rod_fx）

## 快速判断

- 职责与适用：避雷针（注册名 `lightning_rod`，源文件 lightningrod.lua，官方名 "Lightning Rod"／避雷针，[NAMES](../../../raw/current/scripts/languages/chinese_s.po#L344125-L344127)）**充电期间**的电弧标记表现，单注册；充电结算、放电与电池接口都在避雷针本体。协议见 [[reference/mechanisms/generic-fx-factory|通用 FX 工厂]]。[条目](../../../raw/current/scripts/fx.lua#L64-L68)
- 优先使用：查"避雷针特效／电弧动画"选本页；避雷针的免雷击与充电玩法读 lightningrod 本体源码。
- 重要副作用：无音效——zap 声由避雷针本体在 dozap 内播放（[本体声](../../../raw/current/scripts/prefabs/lightningrod.lua#L34-L34)）。[条目](../../../raw/current/scripts/fx.lua#L64-L68)
- 修改边界：避雷针位置一次性落点（不父挂）；动画是否重复未核验。[落点](../../../raw/current/scripts/prefabs/lightningrod.lua#L35-L35)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `lightning_rod_fx` | 通用表条目 → 加载器代理＋一帧后本地实体，无 fn | bank／build=`lightning_rod_fx`，anim=`idle`；无音效、无 tint、无 transform。[条目](../../../raw/current/scripts/fx.lua#L64-L68) |

注册名在简中 .po 无 `STRINGS.NAMES` 键（避雷针本体键为 `LIGHTNING_ROD`）；标题中文"电弧动画"为已核验的效果描述，非官方译名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 工厂协议 | 通用表条目，无附加字段 | 单注册 | [[reference/mechanisms/generic-fx-factory]]；[条目](../../../raw/current/scripts/fx.lua#L64-L68) |
| 生成者：dozap | 避雷针 `dozap` 播本体 zap 声后在**避雷针位置**落点生成，并按 10～40 秒随机重排自身任务（周期自续） | 服务端；fx 落点与避雷针本体声同时发生 | [dozap](../../../raw/current/scripts/prefabs/lightningrod.lua#L29-L38) |
| 触发一：充电建立 | 雷击命中（`onlightning`→`setcharged(inst, 3)`）与读档恢复充能都调 `setcharged`，进充电态即 dozap 一次 | 服务端；充电态开泛光与 Light | [雷击](../../../raw/current/scripts/prefabs/lightningrod.lua#L76-L79)、[setcharged](../../../raw/current/scripts/prefabs/lightningrod.lua#L65-L74)、[读档](../../../raw/current/scripts/prefabs/lightningrod.lua#L89-L91) |
| 触发二：世界日终 | 充电期间监听 `cycles`，`ondaycomplete` 每日终 dozap 并递减充能（源码注释：一次充能按日衰减的模型，耗尽则放电） | 服务端；充能按日衰减 | [日终](../../../raw/current/scripts/prefabs/lightningrod.lua#L56-L63)、[模型注释](../../../raw/current/scripts/prefabs/lightningrod.lua#L100-L100) |
| 触发三：取电 | 被当电池取电（`UseAsBattery`）：传入 mult 且实际电量**大于**请求（`actual_charge > mult`）才 dozap；恰好够用或未传 mult 直接放电、不出特效 | 服务端 | [取电](../../../raw/current/scripts/prefabs/lightningrod.lua#L117-L124) |
| 依赖声明 | lightningrod prefabs 表声明本名（另有 collapse_small 拆除表现） | 仅声明；生成只在 dozap | [声明](../../../raw/current/scripts/prefabs/lightningrod.lua#L10-L14) |
| 资源 | bank／build=`lightning_rod_fx`（资产与避雷针本体 build 同名族） | 动画时长与循环性未核验 | [条目](../../../raw/current/scripts/fx.lua#L64-L68) |

## 典型接入

### 复刻充电标记

- 前提与选择：给自定义"充能建筑"配标记时，照抄"状态进入即表现一次＋随机周期自续"的 dozap 结构；只想出一次表现可单独 Spawn。
- 生效与撤销：落点一次性动画，animover 自删；周期节奏由避雷针的任务控制，fx 侧无感知。
- 原版依据：[dozap 周期](../../../raw/current/scripts/prefabs/lightningrod.lua#L29-L38)。

```lua
local fx = SpawnPrefab("lightning_rod_fx")
fx.Transform:SetPosition(rod.Transform:GetWorldPosition())
```

- 调整与验证：周期（10～40 秒）与日衰减都在避雷针本体，fx 不携带计时；改节奏需动本体任务。

## 常见陷阱与限制

- **误区：每次雷击直接生成 fx**：雷击走 `onlightning`→`setcharged`，fx 由 setcharged 内的 dozap 生成；雷击命中的受击动画（onhit 播 "hit"）是避雷针本体的另一条路径。[雷击链](../../../raw/current/scripts/prefabs/lightningrod.lua#L76-L78)
- **边界：表现与充电结算分离**：充能状态、日衰减、电池接口都在避雷针本体（V2C 注释注明按日衰减的 1 次充能语义，[注释](../../../raw/current/scripts/prefabs/lightningrod.lua#L100-L100)）；fx 无任何组件。
- **边界：zap 声不是 fx 的**：音效由避雷针本体 SoundEmitter 播放，fx 数据无音效字段；只有表现无名的"哑"实体。[本体声](../../../raw/current/scripts/prefabs/lightningrod.lua#L34-L34)
- **边界：落点不父挂**：fx 固定在 dozap 时刻的避雷针位置；避雷针是建筑不会移动，但生成后重建避雷针不会带走旧表现。[落点](../../../raw/current/scripts/prefabs/lightningrod.lua#L35-L35)
- **边界：动画是否重复未核验**：条目仅 `PlayAnimation` 一次，未显式传循环参数，实际是否重复由动画数据决定。[播放](../../../raw/current/scripts/prefabs/fx.lua#L63-L65)

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496。单注册配置、加载器协议、dozap 与三种触发（雷击充电／读档、日终、取电）、避雷针官方名核验完毕；.po 注册名键定向核验无结果。未覆盖：`lightning_rod_fx` 动画资源内容、避雷针放电／拆除全流程（本体范围）、晚加入时序。
- 游戏运行：未运行；电弧视觉与周期节奏未实测。

返回 [[reference/prefabs/fx-structure-index|建筑、装置与遗迹演出]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-2）。
