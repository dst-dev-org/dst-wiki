---
title: "fumarolelocaltemperature"
created: "2026-10-28"
updated: "2026-10-28"
type: component
size_class: simple
summary: "洞穴世界网络实体上的地热温度层：按 9x9 瓦片统计喷气孔密度，向温度查询提供喷气孔区域的局部高温。"
aliases: ["喷气孔局部温度", "fumarolelocaltemperature", "地热温度", "局部温度"]
tags: [component]
dst_build: "steam-buildid:26230496"
sources: ["components/fumarolelocaltemperature.lua", "prefabs/cave_network.lua", "components/temperatureoverrider.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# fumarolelocaltemperature

## 快速判断

- 职责与适用：挂在 `TheWorld.net`（世界网络实体，非 TheWorld 本体）上（[挂载](../../../raw/current/scripts/prefabs/cave_network.lua#L12)、[net 实体](../../../raw/current/scripts/prefabs/world_network.lua#L32-L34)），为 `temperatureoverrider` 的全局温度查询提供喷气孔区域高温兜底（[消费](../../../raw/current/scripts/components/temperatureoverrider.lua#L30)）。
- 优先使用：读温度直接调 `GetTemperatureAtXZ(x, z)`；不要重复挂载——原版只随 cave_network 生成。
- 重要副作用：组件维护 netvar `_noisetime`（30 秒同步周期），双端各自演算温度（[netvar](../../../raw/current/scripts/components/fumarolelocaltemperature.lua#L54)、[同步](../../../raw/current/scripts/components/fumarolelocaltemperature.lua#L61-L81)）。
- 修改边界：温度瓦片缓存 `DataGrid` 与地图同尺寸，初始化即建（[缓存](../../../raw/current/scripts/components/fumarolelocaltemperature.lua#L120-L125)）；非洞穴世界无此组件，调用方须判空（temperatureoverrider 已判空）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `GetTemperatureAtXZ(x, z)` | → `number or nil` | 以目标瓦片为中心 9x9 瓦片统计可通行面积中 `fumarolearea` 节点占比，0 返回 nil，否则 `Lerp(世界温度, 当前地热温度, 占比)`（[实现](../../../raw/current/scripts/components/fumarolelocaltemperature.lua#L140-L166)）。 |
| `GetTemperature()` | → `number` | 当前地热点温度（季节正弦曲线+perlin 噪声，[曲线](../../../raw/current/scripts/components/fumarolelocaltemperature.lua#L93-L103)）。 |
| `SetTemperatureMod(multiplier, locus)` | number, number | 设置温度乘数与偏移枢轴；快照内无原版调用方（[实现](../../../raw/current/scripts/components/fumarolelocaltemperature.lua#L131-L134)）。 |
| `OnSave()` / `OnLoad(data)` | 仅 mastersim | 季节、季节温度、噪声时间随世界存档（[实现](../../../raw/current/scripts/components/fumarolelocaltemperature.lua#L212-L225)）。 |

## 典型接入

### 让实体温度感知喷气孔区域

- 前提与选择：洞穴世界专用；与 [[reference/components/temperatureoverrider]] 的实体级覆写共存——后者优先，喷气孔层兜底（[优先级](../../../raw/current/scripts/components/temperatureoverrider.lua#L29-L31)）。
- 生效与撤销：`seasontick` 驱动季节温度、`OnUpdate` 驱动噪声；`ms_simunpaused` 时强制重同步 netvar（[事件](../../../raw/current/scripts/components/fumarolelocaltemperature.lua#L179-L184)）。
- 原版依据：温度区间 70~125 度、冬夏交越点 75/90（[常量](../../../raw/current/scripts/components/fumarolelocaltemperature.lua#L21-L27)）；查询入口 `GetTemperatureAtXZ` 与实体温度（[入口](../../../raw/current/scripts/components/temperatureoverrider.lua#L34-L36)）。

```lua
-- 客户端/服务端均可读（组件在 TheWorld.net 上）
local fumarole = TheWorld.net and TheWorld.net.components.fumarolelocaltemperature
if fumarole then
    local temp = fumarole:GetTemperatureAtXZ(x, z) -- 可能为 nil（非喷气孔区）
end
```

- 调整与验证：返回值单位与其他温度组件一致（度）；瓦片结果缓存后不随喷气孔增删刷新，地形改动需重启世界才重算（[缓存写入](../../../raw/current/scripts/components/fumarolelocaltemperature.lua#L162)）。

## 常见陷阱与限制

- `GetTemperatureAtXZ` 返回 nil 表示占比为 0，调用方必须处理 nil 再回退世界温度（[返回](../../../raw/current/scripts/components/fumarolelocaltemperature.lua#L165)）。
- 源码含未启用代码：`CalculateVentingTemperature` 引用被注释的 `_venting` netvar 且无调用点（[死代码](../../../raw/current/scripts/components/fumarolelocaltemperature.lua#L88-L91)），不要参照它写功能。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，覆盖组件全文、cave_network/world_network 挂载链、temperatureoverrider 消费点；未运行游戏。
- 结构检查：validate_page.py、validate_source_links.py 与 Lua 语法探针通过。
