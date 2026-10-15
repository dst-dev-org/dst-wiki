---
title: "seasons"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "四季组件：按周期/无尽/恒定三模式推进季节天数，随季节进度平滑改写时钟昼夜段配比，并同步副 shard。"
aliases: ["季节", "四季循环", "seasons 组件"]
tags: [component, network-sync, lifecycle]
dst_build: "steam-buildid:26230496"
size_class: standard
sources: ["components/seasons.lua", "prefabs/world_network.lua", "worldsettings_overrides.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# seasons

## 快速判断

- 职责与适用：挂在 world_network 实体（prefabs/world_network.lua:59）；主 sim 以"天"为单位推进 autumn/winter/spring/summer，非主 shard 通过 `secondary_seasonsupdate` 接收（:473-476）。
- 优先使用：改季节 → 世界事件 `ms_setseason`（:349-365，debugcommands.lua:2716 用例）；改时长 → `ms_setseasonlength {season, length, random?}`（:388-416，worldsettings_overrides.lua:3712-3720 用例）；调昼夜段 → `ms_setseasonclocksegs`／`ms_setseasonsegmodifier`（:367-386、:418-421）。
- 重要副作用：每个周期结束（clock 的 `ms_cyclecomplete`）自动推进一天（:466-467）；`seasontick` 事件携带 `{season, progress, elapseddaysinseason, remainingdaysinseason}`（:241-253）。
- 修改边界：昼夜段总和平恒 16（:34），时长为 0 的季节被循环跳过（GetNextSeason，:97-111）；时钟实际段数由本组件经 `ms_setclocksegs` 下发（:163）。

## 关键接口

| 接口/字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `ms_setseasonlength` 处理 | `(data{season,length,random}) -> nil` | `random=true` 只在非随机态生效一次（`_israndom` 防抖，:392-395）；改当前季时长按进度比例保留剩余天数（:401-414）。 |
| `OnAdvanceSeason`/`OnRetreatSeason` | `() -> nil` | cycle 模式倒计推进/回退并跨季；endless 模式经 `ENDLESS_PRE_DAYS`=10 预热、`ENDLESS_RAMP_DAYS`=10 缓坡到峰值后停在 `ENDLESS_DAYS`=10000（:43-45、:267-347）。 |
| `UpdateSeasonMode`（局部） | `(modified_season?) -> nil` | 唯一活跃季=当前季→`always`，唯一活跃季→`endless`，否则 `cycle`；cycle 起始季可为 0 长度"pre"段（:166-219）。 |
| `PushSeasonClockSegs`（局部） | `() -> nil` | 进度 <50% 向上一季过渡、≥50% 向下一季；跨季时 day/night 按 `1-0.5*sin(PI*p)` 混合、dusk 补足 16（:140-164）；仅 mastershard 下发（:141-143）。 |
| `GetModifiedSegs`（局部） | `(segs, mod) -> table` | 按 `ms_setseasonsegmodifier` 的 day/dusk/night 乘数缩放并补齐到 16（:113-138）。 |
| net 变量组 | `_season` net_tinybyte、`_totaldaysinseason` net_byte、`_elapseddaysinseason` net_ushortint、`_remainingdaysinseason` net_byte、`_endlessdaysinseason` net_bool、`_lengths[4]` net_byte | dirty 事件 seasondirty/lengthsdirty 驱动 `seasontick`/`seasonlengthschanged`（:67-75、:449-450、:255-265）。 |
| `OnSave`/`OnLoad` | 仅主 sim（:484、:509） | 存 mode/premode/israndom/每季段配置与天数；载入段配置总和不 16 则回退默认表（:519-523）。 |

## 关键流程：默认段配比与季节推进

- 默认昼夜段：autumn 8/6/2、winter 5/5/6、spring 5/8/3、summer 11/1/4（:35-41）；时长初值取 `TUNING.<SEASON>_LENGTH`（:445）。
- 计数口径：cycle 中 summer/winter `total=length`，spring/autumn 人为 `total=length*2` 从中段开始（避免开局过冷/过热，:205-214 注释）。
- 事件链：clock 周期完成 → `ms_cyclecomplete` → OnAdvanceSeason → PushSeasonClockSegs → `ms_setclocksegs` 给 clock → clock 推 `clocksegschanged`（两页互链）。

## 典型接入

### 调整季节配置（世界生成/运行时）

- 前提与选择：主 sim 才能推事件；`ms_setseasonlength` 是 worldsettings_overrides 的标准入口（:3712-3720），带 `random=true` 表示"随机难度只定一次"。
- 生效与撤销：改时长立即按比例生效；`ms_setseasonsegmodifier` 需要 day/dusk/night 三键（GetModifiedSegs 全键读取，:114-119）。
- 原版依据：worldsettings_overrides.lua:3712-3720、debugkeys.lua:478-480。

```lua
if TheWorld.ismastersim then
    TheWorld:PushEvent("ms_setseasonlength", { season = "winter", length = 15 })
    TheWorld:PushEvent("ms_setseason", "winter") -- 立即切换
end
```

- 调试与验证：`GetDebugString` 输出当前季、进度百分比与模式（:545-547）；`seasontick` 的 `progress` 即 1-剩余/总（:244）。

## 常见陷阱与限制

- `_lengths` 是 net_byte：单季上限 255 天；全部置 0 时循环停在当前季（GetNextSeason 的 `season == _season:value()` 兜底，:105-108）。
- endless/always 模式的 `remainingdaysinseason` 语义不同：endless 显示端替换为 `ENDLESS_DAYS`（:246、:546），勿直接拿剩余数当"几天换季"。
- 段配比是连续插值：跨季过渡期 `ms_setclocksegs` 每天都变，监听 `clocksegschanged` 做的逻辑会频繁触发。
- 副 shard 只有 net 值：`_mode`/`_segs`/`_segmod` 均为主 sim 局部量（:60-63），副 shard 逻辑须走 `seasontick` 数据。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，通读组件全部 553 行；挂载 world_network.lua:59，事件源 debugkeys.lua:478-480/:642-645/:779/:792、debugcommands.lua:2716、worldsettings_overrides.lua:3712-3720 逐行核对。
- 游戏运行：未运行。

## 相关链接

- [[reference/components/clock]]
- [[reference/components/shard_seasons]]
