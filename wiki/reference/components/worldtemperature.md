---
title: "worldtemperature"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "世界环境温度源：按季节曲线＋昼夜相位＋噪声逐帧计算气温并推 temperaturetick，供 weather 等消费，支持洞穴倍率偏置。"
aliases: ["世界温度", "环境温度源", "worldtemperature"]
tags: [component, network-sync]
dst_build: "steam-buildid:26230496"
sources: ["components/worldtemperature.lua", "prefabs/world_network.lua", "prefabs/cave_network.lua", "components/weather.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# worldtemperature

## 快速判断

- 职责与适用：挂在所有世界的网络实体上（[安装](../../../raw/current/scripts/prefabs/world_network.lua#L58)），每帧算一次"世界气温"并推 `temperaturetick`；对象是 weather／caveweather 等世界系统，不是玩家体温（体温见 [[reference/components/temperature]]）。
- 优先使用：调全局气温用唯一公开方法 `SetTemperatureMod(multiplier, locus)`（[L194-L198](../../../raw/current/scripts/components/worldtemperature.lua#L194-L198)）；洞穴装配即用它套 `CAVES_TEMP_MULT 0.6`（[装配](../../../raw/current/scripts/prefabs/cave_network.lua#L14)、[常量](../../../raw/current/scripts/tuning.lua#L2297-L2298)）。
- 重要副作用：每帧推 `temperaturetick`（weather 消费写入世界状态，[监听](../../../raw/current/scripts/components/weather.lua#L784)）；夏季白昼另推 `overridecolourmodifier` 泛光值（colourcube 消费）（[L157-L160](../../../raw/current/scripts/components/worldtemperature.lua#L157-L160)）。
- 修改边界：客户端也跑同一公式本地预测，噪声时间经周期同步对齐（[注释](../../../raw/current/scripts/components/worldtemperature.lua#L227-L232)）；存读仅主机（[L248-L267](../../../raw/current/scripts/components/worldtemperature.lua#L248-L267)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SetTemperatureMod(multiplier, locus)` | 数字 ×2 | 全局气温公式 `(v - locus) × mult + locus` 的参数；改动后立即推一次 tick（[L114-L117](../../../raw/current/scripts/components/worldtemperature.lua#L114-L117)、[L194-L198](../../../raw/current/scripts/components/worldtemperature.lua#L194-L198)）。 |
| `OnUpdate(dt)` / `LongUpdate` | 组件更新 | 噪声时间推进（30s 周期网络同步）、夏季泛光、推 tick；`LongUpdate = OnUpdate` 支持跳睡（[L233-L242](../../../raw/current/scripts/components/worldtemperature.lua#L233-L242)）。 |
| `OnSave` / `OnLoad` | daylight／season／seasontemperature／phasetemperature／noisetime | 仅 mastersim 定义；读档缺项按公式补算（[L248-L267](../../../raw/current/scripts/components/worldtemperature.lua#L248-L267)）。 |
| 网络变量 `_noisetime` | net_float | Perlin 噪声时钟；`SetWithPeriodicSync` 在同周期内客户端本地推、跨周期等主机同步（[L70](../../../raw/current/scripts/components/worldtemperature.lua#L70)、[L76-L101](../../../raw/current/scripts/components/worldtemperature.lua#L76-L101)）。 |

## 计算公式（关键流程）

1. 季节分量：冬 `5 + sin(π×progress)×(-25-5)`，春线性 5→55，夏 `55 + sin(π×progress)×(95-55)`，秋线性 55→5（[L20-L23](../../../raw/current/scripts/components/worldtemperature.lua#L20-L23)、[L103-L108](../../../raw/current/scripts/components/worldtemperature.lua#L103-L108)）。
2. 相位分量：`day +5`／`night -6` 乘 `sin(timeinphase×π)` 钟形；dusk 为 0（[L25-L29](../../../raw/current/scripts/components/worldtemperature.lua#L25-L29)、[L110-L112](../../../raw/current/scripts/components/worldtemperature.lua#L110-L112)）。
3. 噪声分量：`2×8×perlin(noisetime×0.025) - 8`，即 ±8 波动（[L17-L18](../../../raw/current/scripts/components/worldtemperature.lua#L17-L18)、[L114-L117](../../../raw/current/scripts/components/worldtemperature.lua#L114-L117)）。
4. 全局修饰与输出：套 mult/locus 后每帧 `PushEvent("temperaturetick", 温度值)`（[L162-L165](../../../raw/current/scripts/components/worldtemperature.lua#L162-L165)）。
5. 夏季泛光：夏季白昼 ramp 5s 升满，5–10s 周期正弦脉动，作为 `overridecolourmodifier` 推给 colourcube（[L119-L160](../../../raw/current/scripts/components/worldtemperature.lua#L119-L160)）。

## 常见陷阱与限制

- **它不是温度权威**：`temperaturetick` 的值要经 weather／caveweather 写入 `TheWorld.state.temperature` 后才是公认环境温度；直接监听本事件绕过 weather 会与 `moistureceil`／降水等脱节（[消费](../../../raw/current/scripts/components/weather.lua#L784)）。
- **相位字段自管理**：`_daylight` 只认 `phasechanged` 的 `"day"`，读档只存布尔；修改昼夜相位数量（如加中间相位）需同步 `PHASE_TEMPERATURES` 表（[L181-L183](../../../raw/current/scripts/components/worldtemperature.lua#L181-L183)）。
- **seasontick 缺失时按秋中点起步**：构造期 `_season="autumn"`、progress 取 0.5 计算初值，季节组件晚一帧才覆盖；构造同帧读到的温度可能是秋季值（[L60](../../../raw/current/scripts/components/worldtemperature.lua#L60)、[L204-L205](../../../raw/current/scripts/components/worldtemperature.lua#L204-L205)）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。覆盖组件全文、world_network／cave_network 装配、weather 消费行与 CAVES_TEMP 常量；未覆盖 caveweather 差异分支与 colourcube 泛光渲染链。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、Lua 语法探测通过（见批级记录）。

相关：[[reference/components/temperature]]（实体体温）、[[reference/components/temperatureoverrider]]（局部覆盖）。
