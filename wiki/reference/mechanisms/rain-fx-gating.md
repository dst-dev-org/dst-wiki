---
title: "雨滴 FX 与无雨区门控（build 26148036）"
created: "2026-10-26"
updated: "2026-10-28"
type: mechanism
summary: "地面 rain／洞穴 caverain 雨 FX 实体在粒子发射与溅落取点时逐点调用 Map:CanPointHaveRain 与雨棚判定，无雨区（金库、查理竞技场、norain 标签地形）不再绘制雨滴与水花；强度由 weather／caveweather 按降水率驱动。"
aliases: ["无雨区门控", "雨滴FX门控", "rain caverain"]
tags: [mechanism, ui, data]
dst_build: "steam-buildid:26230496"
sources: ["prefabs/rain.lua", "prefabs/caverain.lua", "components/weather.lua", "components/caveweather.lua", "components/map.lua", "components/raindome.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 雨滴 FX 与无雨区门控（build 26148036）

## 快速判断

- 职责与适用：`rain`（地面）与 `caverain`（洞穴）是客户端本地雨 FX 实体（FX 标签、非网络、`persists = false`），本 build 在其三处取点统一加 `Map:CanPointHaveRain` 门控；本页记录门控链与驱动方，`CanPointHaveRain` 的区域语义（norain 标签／金库／竞技场）见 [[reference/mechanisms/vault-virtualroom-migration|vault→virtualroom 迁移]] 的 map 查询表，不重复。[prefabs/rain.lua:87-116](../../../raw/current/scripts/prefabs/rain.lua#L87-L116)、[prefabs/caverain.lua:149](../../../raw/current/scripts/prefabs/caverain.lua#L149
- 优先使用：自定义天气 FX 想复用同一表现约定时，逐点执行同款三联判定 `not IsUnderRainDomeAtXZ(x, z) and map:IsPassableAtPoint(x, 0, z, true) and map:CanPointHaveRain(x, 0, z)`；雨棚圈内补偿滴逻辑见关键流程。[prefabs/rain.lua:225](../../../raw/current/scripts/prefabs/rain.lua#L225
- 重要副作用：纯客户端表现门控，不改湿度、渗水或事件——数据侧门控另有 `caveweather` 的音效／湿地覆盖与动态 `rainimmunity`（更新报告 C-08 已核验条目）、`inventoryitemmoisture` 的入水继承（R-08）。[components/caveweather.lua:514-517](../../../raw/current/scripts/components/caveweather.lua#L514-L517)
- 修改边界：FX 实体仅在客户端 `_hasfx` 时由世界天气组件生成并挂到本地玩家，服务端与专用服务器不存在；模组改 `TUNING` 降水参数不改变门控行为。[components/weather.lua:213](../../../raw/current/scripts/components/weather.lua#L213)、[:545](../../../raw/current/scripts/components/weather.lua#L545)

## 关键接口

| 接口／配置项 | 签名或数据约束 | 消费者、条件、作用与证据 |
|---|---|---|
| `FX.particles_per_tick`／`splashes_per_tick` | number，每 tick 粒子／溅落预算 | 外部唯一驱动面：`weather` 按降水率写 `5 * preciprate`／`2 * preciprate`（雨天分支，其余降水类型清零）；`caveweather` 同构驱动 `caverain` 与酸雨 FX。[weather.lua:1005-1009](../../../raw/current/scripts/components/weather.lua#L1005-L1009)、[caveweather.lua:514-517](../../../raw/current/scripts/components/caveweather.lua#L514-L517) |
| `Map:CanPointHaveRain(x, y, z)` | -> boolean | 无雨区判定：`norain` 拓扑标签 + 竞技场恒 false；实现语义与查询族见 [[reference/mechanisms/vault-virtualroom-migration|迁移页]] map 表。[prefabs/rain.lua:148](../../../raw/current/scripts/prefabs/rain.lua#L148 |
| `IsUnderRainDomeAtXZ(x, z)`／`GetRainDomesAtXZ(x, z)` | -> boolean／-> 雨棚实体表 | 全局函数，由 raindome 组件文件导出；雨棚（[[reference/components/raindome]]）圈内不落雨，圈边补两滴表现。[raindome.lua:29](../../../raw/current/scripts/components/raindome.lua#L29)、[:45](../../../raw/current/scripts/components/raindome.lua#L45) |
| `FX:PostInit()` | 无参 | 生成后按 1/30 秒步进空跑 `updateFunc` 并 FastForward，快速填满粒子场避免开场无雨感；`weather` 在 FX 就绪时调用。[rain.lua:258-266](../../../raw/current/scripts/prefabs/rain.lua#L258-L266)、[weather.lua:837](../../../raw/current/scripts/components/weather.lua#L837 |

## 关键流程

三处门控在 `rain` 与 `caverain` 逐字对应（本 build 同批加入，此前仅雨棚与可通行判定）：

| 取点 | 判定与结果 | 证据 |
|---|---|---|
| 粒子发射 `emit_fn` | 目标点在雨棚圈内或 `CanPointHaveRain` 为 false → 丢弃该粒子（屏幕裁剪分支之后）；caverain 同。[rain.lua:148](../../../raw/current/scripts/prefabs/rain.lua#L148)、[caverain.lua:149](../../../raw/current/scripts/prefabs/caverain.lua#L149 |
| 溅落水花 `updateFunc` | 采样点须 `IsPassableAtPoint(…, true)` 且 `CanPointHaveRain` 才从对象池取 `raindrop` 水花；不可通行或无雨区跳过。[rain.lua:207-209](../../../raw/current/scripts/prefabs/rain.lua#L207-L209)、[caverain.lua:208](../../../raw/current/scripts/prefabs/caverain.lua#L208 |
| 雨棚补偿滴 | 雨棚边缘两点采样仍需可通行＋不在圈内＋`CanPointHaveRain` 三条同时满足才生成补偿水花——雨棚建在无雨区时不再产生圈边假滴。[rain.lua:212-227](../../../raw/current/scripts/prefabs/rain.lua#L212-L227)、[caverain.lua:226](../../../raw/current/scripts/prefabs/caverain.lua#L226 |

驱动链：`weather`（地面）／`caveweather`（洞穴）按各自降水状态每帧写 `particles_per_tick`／`splashes_per_tick`；FX 的 `updateFunc` 消耗预算取点、过门控后发射；水花走 `raindrop_pool` 对象池，移除 FX 时清池。[rain.lua:53-83](../../../raw/current/scripts/prefabs/rain.lua#L53-L83)、[weather.lua:1005-1040](../../../raw/current/scripts/components/weather.lua#L1005-L1040)

## 典型接入

### 给自定义环境 FX 加同款无雨区门控

- 前提与选择：客户端本地 FX（`AddVFXEffect` 粒子或周期取点 spawn 表现）；门控只影响表现取点，不承担数据职责。
- 生效与撤销：逐点判定 `TheWorld.Map:CanPointHaveRain(x, 0, z)`（需要时叠加可通行与雨棚判定）；判定失败跳过该点即可，无清理路径。

```lua
-- modmain.lua，客户端环境 FX 的取点回调内
local map = GLOBAL.TheWorld.Map
local function TrySpawnMyFxAt(x, z)
    if map:IsPassableAtPoint(x, 0, z, true) and map:CanPointHaveRain(x, 0, z) then
        -- 原版 rain.lua:207 同款三联判定的子集；需要避雨棚时再叠加 IsUnderRainDomeAtXZ
        local fx = GLOBAL.SpawnPrefab("my_rain_fx")
        fx.Transform:SetPosition(x, 0, z)
    end
end
```

- 调整与验证：`CanPointHaveRain` 返回语义随房间集注册状态变化（见迁移页陷阱：房间集未就绪时恒 false）；本例仅接入表现，未运行。

## 常见陷阱与限制

- **门控只管表现，不管数据**：无雨区内玩家湿度、物品渗水、酸雨腐蚀由各自组件的门控决定（caveweather C-08、inventoryitemmoisture R-08、perishable C-26），雨 FX 不出事件也不改状态。[rain.lua:148](../../../raw/current/scripts/prefabs/rain.lua#L148
- **洞穴 void 与不可通行点默认无水花**：溅落门控先查 `IsPassableAtPoint`，海岸边、深渊上的采样点直接跳过，模组不能用"有水花"反推"可站立"。[rain.lua:207](../../../raw/current/scripts/prefabs/rain.lua#L207
- **非网络实体取不到服务端引用**：FX 挂在本地玩家实体下由客户端独占，服务端模组试图 `TheWorld.rain` 之类查询不成立；强度只能经 weather/caveweather 的降水状态间接影响。[weather.lua:545](../../../raw/current/scripts/components/weather.lua#L545
- **雨棚补偿滴也吃无雨门控**：本 build 起 `raindome` 圈边补偿滴三条件同时满足才生成——雨棚放在金库/竞技场内不再有圈边水花。[rain.lua:225](../../../raw/current/scripts/prefabs/rain.lua#L225

## 验证记录

- 源码审阅：2026-10-26，Steam buildid 26148036；覆盖 rain.lua／caverain.lua 全文（发射器参数差异：caverain 粒子更小更慢、寿命 1 秒、prefab 名不同，门控逻辑逐字一致）与三处门控行、weather／caveweather 的生成/挂载/强度驱动点、raindome 全局函数定义。更新报告 prefabs/rain.lua（P-91）与 prefabs/caverain.lua（P-25）两行此前为待审，本页按当前源码转正；caveweather C-08／map C-23／lootdropper 等已核验条目仅复用不重抄。未覆盖：实际粒子观感、酸雨 FX（acidrain 属独立 prefab）、月雹 lunarhail 门控。
- 游戏运行：未运行。
- 结构检查：2026-10-26 执行 validate_page.py（standard 档）与 validate_source_links.py（--strict）通过；页内 Lua 示例经 parser_probe.exe --syntax-only 语法检查通过。

相关：[[reference/mechanisms/vault-virtualroom-migration]]（CanPointHaveRain 区域语义）、[[reference/components/raindome]]（雨棚判定来源；天气驱动方 weather／caveweather 暂无独立页，见 sources 所列文件）。
