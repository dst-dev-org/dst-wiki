---
title: "weather"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "地表天气组件：主端与客户端双端模拟 湿度→降水(雨/雪/月雹)→湿身 循环与温度驱动的积雪/融雪，管闪电调度/落点分流、花粉与地面overlay，推 weathertick。"
aliases: ["天气", "下雨下雪", "weather"]
tags: [component, network-sync]
size_class: complex
dst_build: "steam-buildid:26230496"
sources: ["components/weather.lua", "prefabs/forest_network.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# weather

## 快速判断

- 职责与适用：装在地表网络世界（[forest_network](../../../raw/current/scripts/prefabs/forest_network.lua#L17)），**主端与所有客户端都存在**：主端推进湿度/雪/月雹/闪电，客户端本地模拟再按周期等服务器校正（[双端注释](../../../raw/current/scripts/components/weather.lua#L873-L878)）。
- 优先使用：读天气走 `weathertick` 世界事件（worldstate 聚合成 `wet`/`issnowing` 等世界状态）；控制天气用主端事件 `ms_forceprecipitation`／`ms_setprecipitationmode`／`ms_setsnowlevel` 等（[注册](../../../raw/current/scripts/components/weather.lua#L814-L825)）。
- 重要副作用：降雪按温度自动切换雨/雪并累积 `_snowlevel` 推 `snowcoveredchanged`；月雹需月裂激活（riftspawner）或 `ms_startlunarhail` 事件；闪电 40 距离内优先 redirect 到避雷针/挡雷物（[L630-L730](../../../raw/current/scripts/components/weather.lua#L630-L730)）。
- 修改边界：玩家湿身数值在玩家 moisture 组件，这里只维护世界级 `_wetness` 展示值；洞穴天气是独立的 [[reference/components/caveweather]]（结构相近但不共享实例，洞穴无雪/闪电/月雹）（[对照](../../../raw/current/scripts/components/caveweather.lua#L364-L375)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| netvar 组 | `weather._noisetime/_moisture/_moisturerate/_moistureceil/_moisturefloor/_precipmode/_preciptype/_peakprecipitationrate/_snowlevel/_lunarhaillevel/_snowcovered/_wetness/_wet` | dirty 事件转推 `moistureceilchanged`／`precipitationchanged`（none/rain/snow/lunarhail）／`snowcoveredchanged`／`wetchanged`（[L238-L250](../../../raw/current/scripts/components/weather.lua#L238-L250)、[L777-L780](../../../raw/current/scripts/components/weather.lua#L777-L780)）。 |
| `PRECIP_MODES` / `PRECIP_TYPES` / `LIGHTNING_MODES` | dynamic/always/never；none/rain/snow/lunarhail；rain/snow/any/always/never | [L23-L38](../../../raw/current/scripts/components/weather.lua#L23-L38)、[L158-L166](../../../raw/current/scripts/components/weather.lua#L158-L166)。 |
| `SetWithPeriodicSync(netvar, val, period, ismastersim)` | 内部函数 | 双端同步核心（噪声 30s、湿度 100s、潮湿 10s、雪面 0.1s）（[L356-L376](../../../raw/current/scripts/components/weather.lua#L356-L376)、[常量](../../../raw/current/scripts/components/weather.lua#L17-L61)）。 |
| 主端 ms_* 事件 | `ms_forceprecipitation(enable)`、`ms_setprecipitationmode(mode)`、`ms_setmoisturescale(mult)`、`ms_deltamoisture/ceil(delta)`、`ms_setsnowlevel(level)`、`ms_deltawetness(delta)`、`ms_setlightningmode(mode)`、`ms_setlightningdelay({min,max})`、`ms_sendlightningstrike(pos)`、`ms_startlunarhail` | c_forceprecipitation／魔杖闪电（altar_prototyper 也在用）等入口（[L586-L625](../../../raw/current/scripts/components/weather.lua#L586-L625)、[消费](../../../raw/current/scripts/prefabs/altar_prototyper.lua#L148)）。 |
| `StartPrecipitation(temperature)` | 主端 | 湿度顶满 ceil、随机 floor/当季峰值；温度低于当季 `START_SNOW_THRESHOLDS`（冬 +5，其余 -5）出雪否则出雨；同时初始化下一次闪电延迟（[L419-L427](../../../raw/current/scripts/components/weather.lua#L419-L427)、[阈值](../../../raw/current/scripts/components/weather.lua#L79-L93)）。 |
| 雨/雪自动切换 | 主端 OnUpdate | 雨中温度 <start 阈值转雪；雪中温度 >stop 阈值（冬 10 其余 0）转雨（[L1088-L1092](../../../raw/current/scripts/components/weather.lua#L1088-L1092)）。 |
| 月雹循环 | `_lunarhaillevel` 0~100 | 月裂激活时按 `LUNARHAIL_EVENT_COOLDOWN`(10 日) 充能，满则 `StartLunarHail`；雹期按 `LUNARHAIL_EVENT_TIME`(3 segment) 放空；rate 为 sin 曲线（[L921-L949](../../../raw/current/scripts/components/weather.lua#L921-L949)、[tuning](../../../raw/current/scripts/tuning.lua#L7896-L7897)）。 |
| `OnSendLightningStrike(pos)` | `ms_sendlightningstrike` 处理器 | 40 范围内找 `lightningblocker`（命中则改 thunder）/`lightningrod`（推 `lightningstrike`）/`lightningtarget`（玩家按 playerlightningtarget 概率，运气组件见 [[reference/components/luckuser]]）；风暴区改 `moonstorm_lightning`；最后 `StrikeLightningAtPoint` 落火/电击（[L627-L730](../../../raw/current/scripts/components/weather.lua#L627-L730)）。 |
| `PushWeather()` | 推 `weathertick` | 携带 moisture/pop/precipitationrate/snowlevel/lunarhaillevel/lunarhailrate/wetness/light（[L491-L504](../../../raw/current/scripts/components/weather.lua#L491-L504)）。 |
| `CalculateLight()` | → 0~1 | 降雪用冬季 dynrange；precipmode never 且非月雹恒 1（[L455-L479](../../../raw/current/scripts/components/weather.lua#L455-L479)）。 |
| OnSave/OnLoad | 全量主端状态 | 含闪电模式/延迟与雪阈值（[L1131-L1194](../../../raw/current/scripts/components/weather.lua#L1131-L1194)）。 |

## 关键流程

### 湿度-降水-积雪（OnUpdate 每帧）

1. 湿度循环同洞天气：无雨累积到 ceil 开雨，雨中按 `preciprate*10` 消散到 floor 停；冬季第 2 天特例给 `moisturerateoffset=50` 保证初雪盖地（[L510-L531](../../../raw/current/scripts/components/weather.lua#L510-L531)、[L887-L919](../../../raw/current/scripts/components/weather.lua#L887-L919)）。
2. 积雪独立于降水开关：雪期按 `preciprate * (1/300)` 累积，非雪期温度 >0 按 `(1/120)+(1/20)*min(temp/20,1)` 融化；`_snowlevel` ≥0.015 且温度 <0 或湿身 <5 时铺雪 overlay，否则铺泥 overlay（[L1051-L1066](../../../raw/current/scripts/components/weather.lua#L1051-L1066)、[阈值](../../../raw/current/scripts/components/weather.lua#L121)）。
3. 潮湿：雨中按雨量加湿；冬季温度 <0 时干燥率恒 -1，否则同洞天气的温度/指数组合（[L481-L489](../../../raw/current/scripts/components/weather.lua#L481-L489)）。

### 闪电调度（主端）

1. 降水期按 `_lightningmode`（默认 rain）匹配当前降水类型才计时；延迟缺省按雨量 linear(30→10)。
2. 雨量 >0.75（或 always 模式）时随机抽一名玩家在 2~10 距离外落雷（走 `ms_sendlightningstrike`），否则只放远/近雷声（[L1095-L1119](../../../raw/current/scripts/components/weather.lua#L1095-L1119)）。

### 表现层

非专用端把 `rain`/`snow`/`pollen`/`lunarhail` 粒子挂激活玩家；雨声分环境/树雨/伞面/雨罩四通道（月雹有专属音效），夏季花粉按季节进度 ramp 且沙暴中关闭（[L964-L1081](../../../raw/current/scripts/components/weather.lua#L964-L1081)）。

## 常见陷阱与限制

- **客户端也跑完整 OnUpdate**：客户端不能自行开关降水（只 `set_local` 预测），模组逻辑勿假设只有主端更新（[双端注释](../../../raw/current/scripts/components/weather.lua#L873-L878)）。
- **闪电重定向顺序固定**：blocker > rod > 普通目标；rod 被改道进 blocker 范围时也会被吸收成 thunder，月风暴闪电仍可击中避雷针（[L678-L699](../../../raw/current/scripts/components/weather.lua#L678-L699)）。
- **雪阈值随季节跳变**：`START/STOP_SNOW_THRESHOLDS` 在 seasontick 换季时整体替换，跨季温度可能瞬间改变雨雪形态（[L529-L530](../../../raw/current/scripts/components/weather.lua#L529-L530)）。
- **`ms_startlunarhail` 不检查前置**：事件直接置满月雹等级，即使月裂未激活也会下一次雹（冷却积累才需要月裂）（[L438-L441](../../../raw/current/scripts/components/weather.lua#L438-L441)）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。覆盖组件全文、forest_network 装配、altar_prototyper/debugkeys 闪电触发、LUNARHAIL 调值；未覆盖 rain/snow/pollen/lunarhail 粒子 prefab 与 worldstate 聚合细节。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、Lua 语法探测通过（见批级记录）。

相关：[[reference/components/caveweather]]、[[reference/components/moonstorms]]、[[reference/components/sandstorms]]。
