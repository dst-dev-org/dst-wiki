---
title: "oceanicemanager"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "海冰地块世界管理器：把海格改写为 OCEAN_ICE、维护伤害/删除数据网格，破坏时弹出浮冰、处理平台上的实体落水与掉落。"
aliases: ["海冰管理", "冰面地块", "oceanicemanager 组件"]
tags: [component, lifecycle]
dst_build: "steam-buildid:26230496"
size_class: standard
sources: ["components/oceanicemanager.lua", "prefabs/forest.lua", "components/sharkboimanager.lua", "prefabs/crabking.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# oceanicemanager

## 快速判断

- 职责与适用：仅主机的世界组件（:10 断言；forest.lua:708 挂载，:709 注释明示 sharkboimanager 依赖它）；按 tile 粒度创建/伤害/破坏 `WORLD_TILES.OCEAN_ICE` 地块。
- 优先使用：造冰 → `QueueCreateIceAtPoint`（:228-243，延迟 2.1+0.3s）；破坏 → `QueueDestroyForIceAtPoint`（:414-449，先裂后碎）；直接伤害 → `DamageIceAtPoint/AtTile`（:451-478）。
- 重要副作用：破坏按 undertile 还原底地块并弹出 `boat_ice` 浮冰（:259-316，crabking.lua:792 可换 `boat_ice_crabking`）；推世界事件 `icefloebreak`（:313）。
- 修改边界：数据网格在 `worldmapsetsize` 事件初始化（:43-54）；不存实体引用，仅存 DataGrid（`ZipAndEncodeSaveData`，:504-511）。

## 关键接口

| 接口/字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `CreateIceAtPoint`／`CreateIceAtTile` | `(x,y,z) -> true` | `SetTile(OCEAN_ICE)` 且先保存 undertile 再回写（farming_manager 的 terraforming 回调会清 undertile，:160-164 注释）；清场：鱼投掷物弹射、bullkelp 掉落产物并换 `bullkelp_root`、物品弹飞（:175-222）。 |
| `QueueCreateIceAtPoint` | `(x,y,z,data?) -> nil` | 无冰才排；`data.base_time/random_time` 可调延迟。:228-243 |
| `QueueDestroyForIceAtPoint` | `(x,y,z,data?) -> nil` | 标记删除格、`fx_ice_crackle`+`ice_crack_grid_fx` 裂纹、70+0..10 帧后破坏（:414-449）；先给台上实体推 `abandon_ship`/`onpresink`（:437-446）。 |
| `DestroyIceAtPoint` | `(x,y,z,data?) -> boolean` | 非 OCEAN_ICE 返回 false；`data.silent` 跳过浮冰与碎屑；玩家被送上浮冰、无 drownable 的实体推 `onsink` 后销毁（:245-358）。 |
| `DamageIceAtPoint`／`DamageIceAtTile` | `(…, damage) -> 新血量\|nil` | 无冰或已 0 早退；扣到 0 自动排破坏；血量上限 `OCEAN_ICE_HEALTH`=30（tuning.lua:8137）。:451-478 |
| `SpawnDamagePrefab` | `(tile_index, health) -> nil` | `oceanice_damage` 实体按损伤百分比显示（:480-498）。 |
| `FixupFloaterObjects` | `(x,z,r,is_ocean?) -> nil` | 对 `floaterobject` 推 `on_landed`/`on_no_longer_landed`。:138-150 |
| `OnSave`/`OnLoad` | 常规存取 | 存删除标记与血量两网格；载入时对标记格重启破坏任务（:504-544）。 |

## 关键流程：一次冰面破坏

1. 入口：`QueueDestroyForIceAtPoint`（sharkboimanager.lua:644、crabking.lua:792 调用）——必须已有冰（health grid 有值，:416-417）。
2. 预警：裂纹 FX 按四邻陆地分布摆位（:383-412），台上实体收 `abandon_ship`。
3. 破坏：sharkboi 竞技场 4 距离内暂缓（"HACK"，:75-88）；还原 undertile 或退 `OCEAN_SWELL`（:259-269）；按邻接陆地合计向量决定浮冰推出方向，最多一格（:288-315）。
4. 平台处置：复刻 walkableplatform 移除逻辑——浮冰半径内免处置、玩家传送到浮冰旁、其余推 `onsink` 并按 drownable/amphibiouscreature/平台站立即决定销毁（:318-357）。
5. 表现：`fx_ice_pop`、`ice` 碎块抛掷与 `degrade_fx_ice` 环形散布（:362-378）。

## 典型接入

### 让脚本破坏一片冰面（crabking 同款）

- 前提与选择：先确认点位确为冰格（`QueueDestroy` 内部判断，`DestroyIceAtPoint` 直接返回 false）；想留实体残骸用 silent+自建表现。
- 生效与撤销：破坏不可撤销——地块已还原，浮冰独立成实体；重建走 `QueueCreateIceAtPoint`。
- 原版依据：crabking.lua:790-792、:889。

```lua
if TheWorld.ismastersim then
    local mgr = TheWorld.components.oceanicemanager
    if mgr then
        mgr:QueueDestroyForIceAtPoint(x, 0, z, { icefloe_prefab = "boat_ice" })
    end
end
```

- 调整与验证：`OCEAN_ICE_TILE_HEALTH`=200 用于排队占位、`OCEAN_ICE_HEALTH`=30 为伤害链上限（tuning.lua:8137-8138）；`OCEAN_ICE_RADIUS`=1.6 决定浮冰偏移与降质 FX 半径（tuning.lua:8136）。

## 常见陷阱与限制

- 仅 forest 世界挂载：cave 世界无本组件，调用前必须判 nil（原版消费方均带判断）。
- 排队破坏依赖 health grid：对从未 `QueueCreateIceAtPoint`/`CreateIceAtTile` 的格直接 `QueueDestroyForIceAtPoint` 无效（:416-417 早退），即便地面上视觉是冰。
- 裂纹 FX 是普通实体：`ice_crack_grid_fx` 4.5 范围查重防叠加（:102-107、:393-397），破坏时统一移除（:273）。
- 网格不存实体：`_ice_damage_prefabs_grid` 里的 `oceanice_damage` 不落存档，载入按血量重建（:533-543）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，通读组件全部 546 行；挂载 forest.lua:708-709、消费方 sharkboimanager.lua:640/:644、crabking.lua:790/:792/:889、TUNING tuning.lua:8136-8139 逐行核对。
- 游戏运行：未运行。
