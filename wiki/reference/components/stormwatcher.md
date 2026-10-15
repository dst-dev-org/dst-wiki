---
title: "stormwatcher"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "玩家风暴观测：聚合沙尘暴/月风暴的世界状态，量化为 7 档 stormlevel 经 player_classified 同步客户端并联动减速与专用观测组件。"
aliases: ["风暴观测", "stormwatcher", "沙尘暴等级"]
tags: [component, network-sync, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/stormwatcher.lua", "prefabs/player_common.lua", "simutil.lua", "components/sandstormwatcher.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# stormwatcher

## 快速判断

- 职责与适用：挂在玩家（所有玩家，[player_common](../../../raw/current/scripts/prefabs/player_common.lua#L2739)）的主机组件：监听世界 `ms_stormchanged` 维护各风暴开关，活跃时周期查询 sandstorms/moonstorms 世界组件，把等级量化后写入 `player_classified.stormlevel/stormtype` 同步客户端。
- 优先使用：判断玩家是否处于全屏沙尘用 `GetStormLevel() >= TUNING.SANDSTORM_FULL_LEVEL`（原版 [simutil](../../../raw/current/scripts/simutil.lua#L537)）；按类型查询传 `STORM_TYPES` 参数。
- 重要副作用：风暴出现时启动组件更新并监听 `changearea`（[L88-L98](../../../raw/current/scripts/components/stormwatcher.lua#L88-L98)）；风暴结束移除 locomotor 上 "sandstorm"/"moonstorm" 外部速度倍率（[L119-L124](../../../raw/current/scripts/components/stormwatcher.lua#L119-L124)）。
- 修改边界：与配套观测组件 [[reference/components/sandstormwatcher]]、[[reference/components/moonstormwatcher]] 协作——它们的等级查询实际转调本组件（[sandstormwatcher](../../../raw/current/scripts/components/sandstormwatcher.lua#L101-L103)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `GetStormLevel(stormtype)` | `可选 STORM_TYPES -> number(0..1)` | 传 nil 或匹配当前风暴时返回等级，否则 0；7 档量化（`floor(level*7+.5)/7`）。[实现](../../../raw/current/scripts/components/stormwatcher.lua#L48-L50) |
| `GetCurrentStorm(inst)` | `-> STORM_TYPES` | 实时查询 `TheWorld.components.sandstorms:IsInSandstorm` 与 `TheWorld.net.components.moonstorms:IsInMoonstorm`，两者同时成立触发断言。[实现](../../../raw/current/scripts/components/stormwatcher.lua#L52-L66) |
| `UpdateStorms(data)` | `ms_stormchanged 数据 -> void` | 记 `currentstorms[类型]=开关`；有活跃风暴时启动更新、挂 changearea、初始化随机延迟。[实现](../../../raw/current/scripts/components/stormwatcher.lua#L81-L106) |
| `UpdateStormLevel` | `-> void` | 按当前风暴取世界等级量化，联动 sandstormwatcher/moonstormwatcher 的表现更新；无风暴时清减速倍率并归零。[实现](../../../raw/current/scripts/components/stormwatcher.lua#L108-L129) |
| `stormlevel/currentstorm` | 字段（带同步 watcher） | 赋值经构造的 `__index` watcher 写入 `player_classified`（stormlevel ×7 取整、stormtype 原值，[同步](../../../raw/current/scripts/components/stormwatcher.lua#L1-L11)）。 |
| `OnUpdate(dt)` | 组件更新 | 按 delay（过渡区间 0.5 秒、其余 1 秒）节流调用 UpdateStormLevel。[实现](../../../raw/current/scripts/components/stormwatcher.lua#L131-L138) |

## 典型接入

### 按风暴等级调整玩家表现

- 前提与选择：主机端读组件；依赖世界已有 sandstorms 组件（forest 世界标配）与 `TheWorld.net.components.moonstorms`。
- 生效与撤销：等级随玩家位置变化周期刷新（进入/离开风暴区经 changearea 即时刷新，[L42-L46](../../../raw/current/scripts/components/stormwatcher.lua#L42-L46)）；风暴结束自动清零。

```lua
-- 主机端：读取玩家当前沙尘暴等级（0..1，7 档量化）
local level = inst.components.stormwatcher:GetStormLevel(STORM_TYPES.SANDSTORM)
if level >= TUNING.SANDSTORM_FULL_LEVEL then
    -- 全屏沙尘判定（原版 simutil.lua 同款阈值）
end
```

- 调整与验证：量化档数与节流间隔是组件内常量；自定义风暴类型无法直接挂入（currentstorms 表只处理 sandstorms/moonstorms 两类事件）。

## 常见陷阱与限制

- **按类型查询需匹配当前风暴**：同一时刻只认一个风暴（`GetCurrentStorm` 有双风暴断言，[L61](../../../raw/current/scripts/components/stormwatcher.lua#L61)），查非当前类型恒得 0。
- **等级是玩家位置函数**：`GetStormLevel` 返回的是"玩家受到的风暴强度"，不是世界开关状态；要查世界级开关用 `ms_stormchanged` 事件或 sandstorms 组件。
- **减速倍率的键固定**：清理时按 "sandstorm"/"moonstorm" 键移除 locomotor 倍率（[L121-L123](../../../raw/current/scripts/components/stormwatcher.lua#L121-L123)），宿主复用这些键会被组件覆盖。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；组件全文、player_common 挂载、simutil 与 sandstormwatcher/moonstormwatcher 消费点。
- 游戏运行：未运行。
