---
title: "caveweather"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "洞穴天气组件：主端与客户端双端模拟湿度-降水-潮湿循环（普通雨/酸雨），经 netvar 周期同步并推 weathertick，主管音效、粒子与地面overlay。"
aliases: ["洞穴天气", "酸雨", "caveweather"]
tags: [component, network-sync]
dst_build: "steam-buildid:26230496"
sources: ["components/caveweather.lua", "prefabs/cave_network.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# caveweather

## 快速判断

- 职责与适用：装在洞穴网络世界（[cave_network](../../../raw/current/scripts/prefabs/cave_network.lua#L8-L12)），**主端与所有客户端都存在**：主端推进湿度循环与降水开关，客户端用同一段 `OnUpdate` 本地模拟再等服务器周期校正（[双端注释](../../../raw/current/scripts/components/caveweather.lua#L591-L596)）。
- 优先使用：读天气状态走世界状态（worldstate 消费 `weathertick`）；代码改天气用主端事件 `ms_forceprecipitation`／`ms_setprecipitationmode`／`ms_deltamoisture` 等（[注册](../../../raw/current/scripts/components/caveweather.lua#L550-L556)）。
- 重要副作用：降雪与光照恒定（`snowlevel=0, light=1`），只推 `weathertick`（[L364-L375](../../../raw/current/scripts/components/caveweather.lua#L364-L375)）；酸雨需要 `TUNING.ACIDRAIN_ENABLED` 且影裂激活（[L334-L339](../../../raw/current/scripts/components/caveweather.lua#L334-L339)）。
- 修改边界：玩家级湿身数值实际存于玩家的 moisture 组件，本组件只维护世界级 `_wetness` 展示值；洞穴温度修正由 cave_network 装配处的 worldtemperature 处理（[装配](../../../raw/current/scripts/prefabs/cave_network.lua#L14)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| netvar 组 | `weather._noisetime/_moisture/_moisturerate/_moistureceil/_moisturefloor/_precipmode/_preciptype/_peakprecipitationrate/_wetness/_wet` | dirty 事件转推世界事件 `moistureceilchanged`／`precipitationchanged`（类型名字符串）／`wetchanged`（[L172-L181](../../../raw/current/scripts/components/caveweather.lua#L172-L181)、[L521-L529](../../../raw/current/scripts/components/caveweather.lua#L521-L529)）。 |
| `PRECIP_MODES` / `PRECIP_TYPES` | dynamic/always/never；none/rain/acidrain | [L23-L37](../../../raw/current/scripts/components/caveweather.lua#L23-L37)。 |
| `SetWithPeriodicSync(netvar, val, period, ismastersim)` | 内部函数 | 双端同步核心：同周期内各端本地推进，跨周期才真正 `set`；停变时主端强制同步（[L275-L295](../../../raw/current/scripts/components/caveweather.lua#L275-L295)；周期：噪声 30s、湿度 100s、潮湿 10s，[L17](../../../raw/current/scripts/components/caveweather.lua#L17)、[L55](../../../raw/current/scripts/components/caveweather.lua#L55)、[L109](../../../raw/current/scripts/components/caveweather.lua#L109)）。 |
| `CalculatePrecipitationRate()` | → 0~1 | always 模式走 perlin 噪声；dynamic 按湿度在 floor~ceil 间正弦、封顶 `_peakprecipitationrate`（[L319-L328](../../../raw/current/scripts/components/caveweather.lua#L319-L328)）。 |
| `StartPrecipitation` / `StopPrecipitation` | 主端专用 | 开雨：湿度顶到 ceil、随机 floor 与当季峰值、影裂激活时选 acidrain；停雨：湿度落回 floor、随机下一个 ceil（[L330-L346](../../../raw/current/scripts/components/caveweather.lua#L330-L346)）。 |
| `CalculateWetnessRate(temperature, preciprate)` | → -1~0.75 | 雨中按雨量 inSine 加湿；无雨按温度（70 度最优）与当前湿度的 inExpo 组合干燥（[L355-L362](../../../raw/current/scripts/components/caveweather.lua#L355-L362)）。 |
| 主端 ms_* 事件 | `ms_forceprecipitation(enable)`、`ms_setprecipitationmode(mode)`、`ms_setmoisturescale(mult)`、`ms_deltamoisture/ceil/wetness(delta)` | c_forceprecipitation 等控制台/模组入口，把湿度直接置 ceil/floor 或平移（[L463-L486](../../../raw/current/scripts/components/caveweather.lua#L463-L486)）。 |
| `PushWeather()` | 推 `weathertick` | 消费者：worldstate（世界状态 wet/pop 等）、ambientlighting、ambientsound、wildfires（[L364-L375](../../../raw/current/scripts/components/caveweather.lua#L364-L375)、[worldstate](../../../raw/current/scripts/components/worldstate.lua)）。 |
| OnSave/OnLoad | 全量主端状态 | 含湿度的 rateval/offset/multiplier 与 ceil/floor 随机上限（[L732-L777](../../../raw/current/scripts/components/caveweather.lua#L732-L777)）。 |

## 关键流程

### 湿度-降水循环（OnUpdate 每帧）

1. 累积：无降水且 ceil>0 时湿度按 `_moisturerate` 增长（seasontick 按 `1 - sin(progress)` 在季节 MIN/MAX 间取值，[L381-L394](../../../raw/current/scripts/components/caveweather.lua#L381-L394)、[MOISTURE_RATES](../../../raw/current/scripts/components/caveweather.lua#L41-L54)），到 ceil 主端 `StartPrecipitation`。
2. 降水：`preciptype` 非 none 时湿度按 `preciprate * 10` 消散，落到 floor 即停雨（[L604-L636](../../../raw/current/scripts/components/caveweather.lua#L604-L636)）。
3. 潮湿：`_wetness` 按雨情/温度涨落，跨过 `TUNING.MOISTURE_WET_THRESHOLD`/`DRY_THRESHOLD` 切换 `_wet` 推 `wetchanged`（[L638-L649](../../../raw/current/scripts/components/caveweather.lua#L638-L649)）。
4. 表现：非专用端把 `caverain`／`caveacidrain` 粒子挂在激活玩家身上，按 `changearea` 的 `CanPointHaveRain/AcidRain` 开关音量与粒子；雨声分环境/树雨（躲避）/伞面（金属伞特判）/雨罩四种，湿度高时铺泥地 overlay（lerp = wetness*3/100）（[L404-L429](../../../raw/current/scripts/components/caveweather.lua#L404-L429)、[L651-L721](../../../raw/current/scripts/components/caveweather.lua#L651-L721)）。

### 玩家区域逻辑（主端）

玩家 changearea 时：不能酸雨的点给 `acidlevel:SetIgnoreAcidRainTicks(true)`；不能下雨的点挂 `rainimmunity` 组件并以 TheWorld 为来源，离开移除（[L430-L461](../../../raw/current/scripts/components/caveweather.lua#L430-L461)）。

## 常见陷阱与限制

- **客户端也跑完整 OnUpdate**：客户端不能自行开关降水（只 `set_local` 预测），逻辑分支写模组时勿假设只有主端更新（[双端注释](../../../raw/current/scripts/components/caveweather.lua#L591-L596)）。
- **洞穴没有雪与光照变化**：`snowlevel` 恒 0、`light` 恒 1，冬夏外观差异只来自湿度/雨声；地表天气请用 [[reference/components/weather]]（两页常量结构相近但独立维护）（[L364-L375](../../../raw/current/scripts/components/caveweather.lua#L364-L375)）。
- **酸雨双重门槛**：TUNING 开关 + 影裂激活；且玩家所处区域 `CanPointHaveAcidRain` 为否时免疫酸雨 tick（[L334-L339](../../../raw/current/scripts/components/caveweather.lua#L334-L339)、[L430-L437](../../../raw/current/scripts/components/caveweather.lua#L430-L437)）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。覆盖组件全文、cave_network 装配、worldstate 等 weathertick 消费者清单；未覆盖 caverain/caveacidrain 粒子 prefab 内部与 ambientlighting 消费细节。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、Lua 语法探测通过（见批级记录）。

相关：[[reference/components/weather]]、[[reference/components/quaker]]（同装配于 cave_network）。
