---
title: "leafpilemanager：叶堆世界管理器"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "万圣夜活动期秋季在落叶林瓦片限量生成 leafpile、其余时间逐只衰减的世界宿主组件；提供分区限密登记、生成调度与存档恢复。"
aliases: [叶堆世界管理器, leafpilemanager, leafpile 生成门控, 万圣夜叶堆生成]
tags: [component, lifecycle, callback]
dst_build: "steam-buildid:26230496"
sources: ["components/leafpilemanager.lua", "tuning.lua", "components/map.lua", "constants.lua", "prefabs/world.lua", "prefabs/forest.lua", "prefabs/leafpile.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: standard
---

# leafpilemanager（叶堆世界管理器）

## 快速判断

- 职责与适用：挂在 forest 世界实体上的服务端组件，构造即断言 `ismastersim`；万圣夜活动（`SPECIAL_EVENTS.HALLOWED_NIGHTS`）且秋季时按 DataGrid 分区限密生成 `leafpile`（叶堆），活动外或秋外逐只衰减，登记表随世界存档恢复。[构造与状态](../../../raw/current/scripts/components/leafpilemanager.lua#L1-L16) [启用门控](../../../raw/current/scripts/components/leafpilemanager.lua#L136-L142)
- 优先使用：需要"活动＋季节门控的限量地表生成"参照，或让模组生成的叶堆纳入原版限密与衰减 → `TheWorld.components.leafpilemanager` 的 `SetEnabled`／`TryToCreateLeafpileAtPoint`／`RegisterLeafpile`，调谐覆盖见[典型接入](#典型接入)。
- 重要副作用：启用期每 `LEAFPILEMANAGER_SECONDS_PER_SPAWN` 秒做一轮"全图随机点＋每玩家近距偏置"尝试生成；禁用期每 0.5s 对一只已登记叶堆调 `Decay()` 并依赖其自删；冬季冻结衰减。[调度循环](../../../raw/current/scripts/components/leafpilemanager.lua#L166-L189)
- 修改边界：全库检索仅 forest 世界 `master_postinit` 挂载，洞穴无此组件；四个节奏／密度调谐键在构造时捕获为局部量，构造后改 `TUNING` 不生效；客户端构造直接断言失败。[挂载](../../../raw/current/scripts/prefabs/forest.lua#L689) [调谐捕获](../../../raw/current/scripts/components/leafpilemanager.lua#L9-L12)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| 构造 | `(inst)`，断言非客户端 | 捕获分区边长（`SPACING×TILE_SCALE`，[TILE_SCALE=4](../../../raw/current/scripts/constants.lua#L17)）、密度与两间隔为局部量；状态 `accumulator`、`enabled`、`leafpiles`（叶堆→分区索引）、`leafpilescount`。([leafpilemanager.lua:1-16](../../../raw/current/scripts/components/leafpilemanager.lua#L1-L16) |
| `GetGridCoordsForPlacement(x, z)` | 返回分区对齐坐标 | `math.floor(x/边长)×边长`。([leafpilemanager.lua:30-32](../../../raw/current/scripts/components/leafpilemanager.lua#L30-L32) |
| `RegisterLeafpile(leafpile)` | 实体 | 按位置取分区，格数据 `{count}` 自增并登记；监听叶堆 `onremove` 自动注销。([leafpilemanager.lua:54-69](../../../raw/current/scripts/components/leafpilemanager.lua#L54-L69) |
| `UnregisterLeafpile(leafpile)` | 实体 | 未登记早退；计数自减，格清空后删除格数据并移除监听。([leafpilemanager.lua:37-53](../../../raw/current/scripts/components/leafpilemanager.lua#L37-L53) |
| `NoLeafpileTest(map, x, y, z)` | 选点过滤器 | 仅 `WORLD_TILES.DECIDUOUS` 瓦片；[MAX_PHYSICS_RADIUS=4](../../../raw/current/scripts/constants.lua#L1508) 半径内无实体；所在分区登记数未满 `DENSITY`。([leafpilemanager.lua:71-88](../../../raw/current/scripts/components/leafpilemanager.lua#L71-L88) |
| `TryToCreateLeafpileAtPoint(x, y, z, skipleafpiletest)` | 返回 boolean | 过门禁后 `SpawnPrefab("leafpile")`、定位、登记并调叶堆 `OnPlaced()`。([leafpilemanager.lua:90-100](../../../raw/current/scripts/components/leafpilemanager.lua#L90-L100) |
| `TryToCreateLeafpileAnywhere()` | 返回 boolean | `Map:FindRandomPointWithFilter(50, NoLeafpileTest)`（50 为最大尝试次数）选点，已过过滤故跳过二次测试。([leafpilemanager.lua:102-108](../../../raw/current/scripts/components/leafpilemanager.lua#L102-L108) [map.lua:874](../../../raw/current/scripts/components/map.lua#L874 |
| `TryToCreateLeafpileForEachPlayer()` | 无参 | 每个不在平台上的玩家，在 [MAX_PHYSICS_RADIUS](../../../raw/current/scripts/constants.lua#L1508) 至 [PLAYER_CAMERA_SEE_DISTANCE=40](../../../raw/current/scripts/constants.lua#L26) 环带内最多 10 次采样，有意向玩家聚密。([leafpilemanager.lua:110-125](../../../raw/current/scripts/components/leafpilemanager.lua#L110-L125) |
| `SetEnabled(enabled)` | 返回 enabled | 变更门控；开启时 `StartUpdatingComponent`，关闭时不停更（由衰减分支自停）。([leafpilemanager.lua:127-135](../../../raw/current/scripts/components/leafpilemanager.lua#L127-L135) |
| `TryToEnable()` | 返回 boolean | [活动未开启返回 false](../../../raw/current/scripts/constants.lua#L1095)；否则按 `isautumn` 启停。([leafpilemanager.lua:136-142](../../../raw/current/scripts/components/leafpilemanager.lua#L136-L142) |
| `OnIsAutumnChanged`／`OnIsWinterChanged` | `WatchWorldState` 回调 | 秋季变化触发 `TryToEnable`；冬季仅记录 `iswinter` 供衰减冻结。([leafpilemanager.lua:144-159](../../../raw/current/scripts/components/leafpilemanager.lua#L144-L159) |
| `OnPostInit()` | 世界实体装载后派发 | 用当前世界状态初始化两季节标记并尝试启用；`inst.PostInit` 遍历组件调用。([leafpilemanager.lua:161-164](../../../raw/current/scripts/components/leafpilemanager.lua#L161-L164) ([world.lua:335-346](../../../raw/current/scripts/prefabs/world.lua#L335-L346) |
| `OnUpdate(dt)` | 组件更新 | 启用：每 spawn 间隔随机点＋每玩家偏置各试一批；禁用：每 despawn 间隔对 `next(self.leafpiles)` 调 `Decay()`（冬季跳过），登记清空后 `StopUpdatingComponent` 自停。([leafpilemanager.lua:166-189](../../../raw/current/scripts/components/leafpilemanager.lua#L166-L189) |
| `OnSave()`／`LoadPostPass(newents, savedata)` | 存读钩子 | 保存登记叶堆 GUID 列表与 ents；读档后重新登记，有残留即重启更新循环。([leafpilemanager.lua:191-215](../../../raw/current/scripts/components/leafpilemanager.lua#L191-L215) |

调谐键（`tuning.lua` "Halloween 2026" 段）：

| 键 | 默认值 | 语义 |
|---|---|---|
| `LEAFPILEMANAGER_SPACING` | 10 | 分区边长（瓦），[tuning.lua:9788](../../../raw/current/scripts/tuning.lua#L9788 |
| `LEAFPILEMANAGER_DENSITY` | 1 | 每分区最多同时存在的登记叶堆数，[tuning.lua:9789](../../../raw/current/scripts/tuning.lua#L9789 |
| `LEAFPILEMANAGER_SECONDS_PER_SPAWN` | 10 | 启用期生成节奏（秒），[tuning.lua:9790](../../../raw/current/scripts/tuning.lua#L9790 |
| `LEAFPILEMANAGER_SECONDS_PER_DESPAWN` | 0.5 | 禁用期衰减节奏（秒/只），[tuning.lua:9791](../../../raw/current/scripts/tuning.lua#L9791 |

同段 `LEAFPILE_*` 键为叶堆自身生长与战利品调谐（([tuning.lua:9792-9798](../../../raw/current/scripts/tuning.lua#L9792-L9798)），由叶堆 prefab 消费，如生长计时 ([leafpile.lua:400-404](../../../raw/current/scripts/prefabs/leafpile.lua#L400-L404)。

## 典型接入

### 覆盖生成节奏与分区密度（调谐覆盖）

前提与选择：在模组 `modmain.lua` 顶层（世界创建前）覆盖 `TUNING`；组件构造时把四个键捕获为局部量，世界创建后再改无效。

```lua
-- modmain.lua（服务端生效；数值为示例）
TUNING.LEAFPILEMANAGER_SPACING = 5              -- 分区边长（瓦），越小限密粒度越细
TUNING.LEAFPILEMANAGER_DENSITY = 2              -- 每分区最多登记叶堆数
TUNING.LEAFPILEMANAGER_SECONDS_PER_SPAWN = 5    -- 启用期生成间隔（秒）
TUNING.LEAFPILEMANAGER_SECONDS_PER_DESPAWN = 1  -- 禁用期衰减间隔（秒/只）
```

生效与撤销：世界加载创建组件时读取新值；删去覆盖即恢复默认。边长单位为瓦，间隔单位为秒。原版依据：([tuning.lua:9788-9791](../../../raw/current/scripts/tuning.lua#L9788-L9791)、[构造捕获](../../../raw/current/scripts/components/leafpilemanager.lua#L9-L12)。

### 服务端生成或登记叶堆（复用门禁、限密与衰减）

前提与选择：目标点已知时用 `TryToCreateLeafpileAtPoint`（走瓦片／实体／密度门禁并登记）；已有叶堆实体（如自定义生成）用 `RegisterLeafpile` 纳管。两者都要求组件已挂载（forest 世界服务端）。

```lua
local function SpawnManagedLeafpile(x, z)
    local manager = TheWorld.components.leafpilemanager
    if manager == nil then
        return false
    end
    return manager:TryToCreateLeafpileAtPoint(x, 0, z) -- 门禁不过时返回 false，不生成
end
```

生效与撤销：成功即生成、登记并经叶堆 `OnPlaced()` 延迟显现；叶堆被清扫／工作／衰减移除时经 `onremove` 自动注销并释放分区名额。绕过活动＋秋季门控可直接 `SetEnabled(true)`；`SetEnabled(false)` 后按衰减节奏清空自停。原版依据：[TryToCreateLeafpileAtPoint](../../../raw/current/scripts/components/leafpilemanager.lua#L90-L100)、[叶堆 OnPlaced](../../../raw/current/scripts/prefabs/leafpile.lua#L296-L315)。

## 常见陷阱与限制

- **未登记叶堆不受管**：世界生成或直接 `SpawnPrefab("leafpile")` 的实体不在登记表，`UnregisterLeafpile` 对其早退；它们不占分区密度、不被管理器衰减，只受叶堆自身的瓦片检查约束。[早退注释](../../../raw/current/scripts/components/leafpilemanager.lua#L39-L41)
- **调谐值构造期捕获**：四个局部量在构造时固化；运行中改 `TUNING.LEAFPILEMANAGER_*` 不影响已存在组件，需新世界／重载生效。([leafpilemanager.lua:9-12](../../../raw/current/scripts/components/leafpilemanager.lua#L9-L12)
- **冬季冻结衰减**：衰减分支受 `not iswinter` 门禁，冬季叶堆原地保留，春季恢复后按 0.5s/只清理。([leafpilemanager.lua:171-176](../../../raw/current/scripts/components/leafpilemanager.lua#L171-L176)
- **衰减依赖叶堆自删**：管理器注释明确假设 `leafpile:Decay()` 会自行 `Remove()`；叶堆实现为注销自身、置不存档、动画结束或休眠时移除。自制"叶堆"替换品须实现同名契约。[依赖注释](../../../raw/current/scripts/components/leafpilemanager.lua#L172-L175) [叶堆 Decay](../../../raw/current/scripts/prefabs/leafpile.lua#L336-L350)
- **叶堆自毁与清扫路径**：叶堆自查所在瓦片，非 DECIDUOUS（含地形改造）即碎裂自毁；被清扫时掉落并移除；这些移除都会经 `onremove` 触发注销。[瓦片检查](../../../raw/current/scripts/prefabs/leafpile.lua#L278-L286) [onterraform](../../../raw/current/scripts/prefabs/leafpile.lua#L526-L532) [清扫结算](../../../raw/current/scripts/prefabs/leafpile.lua#L411-L415)
- **读档即续衰减**：读档后只要有残留登记就重启更新循环（即使当前被门控禁用，用于继续衰减），启用与否随后在 `OnPostInit` 按活动／季节重新判定。[LoadPostPass](../../../raw/current/scripts/components/leafpilemanager.lua#L211-L213)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496（steam-buildid:26230496）；覆盖 `components/leafpilemanager.lua` 全部 218 行，`prefabs/forest.lua` 挂载与 prefab 声明段，`prefabs/leafpile.lua` 的 `OnPlaced`／`Decay`／瓦片自毁／清扫移除协作面，`tuning.lua` LEAFPILEMANAGER_* 与 LEAFPILE_* 键，`components/map.lua` 的 `FindRandomPointWithFilter` 签名，`constants.lua` 相关常量，以及 `prefabs/world.lua` 的 `OnPostInit` 派发。未覆盖：引擎侧 DataGrid 实现与实体休眠细节、动画资源。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 通过；两段 lua 示例通过 parser_probe 语法检查。
- 游戏运行：未运行；活动期实际生成节奏、分区密度表现与冬季保留待实测。

## 相关链接

- [[reference/components/sweepable]]：leafpile 挂载的可清扫组件；被清扫移除是叶堆注销路径之一。
