---
title: "meteorshower"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "挂在隐形单体 meteorspawner 实体上的流星雨状态机：三级强度循环冷却/爆发，按玩家在场调节频率与流星大小预算。"
aliases: ["meteorshower", "流星雨", "流星生成器", "meteorspawner 组件"]
tags: [component, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/meteorshower.lua", "prefabs/meteorspawner.lua", "map/rooms/forest/terrain_rocky.lua", "map/customize.lua", "tuning.lua", "components/worldmeteorshower.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# meteorshower

## 快速判断

- 职责与适用：单个 `meteorspawner` 实体（非网络实体，带 CLASSIFIED 标签）的流星雨组件；世界生成时岩石地带房间放 1～2 个，并有独立的世界生成自定义项。不是全图天气系统，也不直接与 [[reference/components/worldmeteorshower]] 同一组件。[prefab](../../../raw/current/scripts/prefabs/meteorspawner.lua#L4-L9) [房间放置](../../../raw/current/scripts/map/rooms/forest/terrain_rocky.lua#L47) [自定义项](../../../raw/current/scripts/map/customize.lua#L351)
- 优先使用：想让某生成器立刻下雨用 `StartShower(level)`（1～3，缺省随机）；停止/复位用 `StopShower()`；查询状态用 `IsShowering()`/`IsCoolingDown()`。[接口](../../../raw/current/scripts/components/meteorshower.lua#L217-L250) [状态查询](../../../raw/current/scripts/components/meteorshower.lua#L144-L150)
- 重要副作用：自然节奏只会随机升级流星等级（构造与每轮冷却后都 `RandomizeLevel`），玩家在场与否只改生成频率与大小分配，不改等级；月岩壳流星每轮至多 1 颗且要求落点合法地面。[等级随机](../../../raw/current/scripts/components/meteorshower.lua#L140) [月岩壳](../../../raw/current/scripts/components/meteorshower.lua#L171-L173)
- 修改边界：有 OnSave/OnLoad（version 2，兼容旧档）与 OnRemoveFromEntity=StopShower；移除组件即取消任务。[存读](../../../raw/current/scripts/components/meteorshower.lua#L278-L321) [移除](../../../raw/current/scripts/components/meteorshower.lua#L330)

## 关键接口

| 接口 | 条件、作用与证据 |
|---|---|
| `StartShower(level?)` | 先 StopShower；从世界组件 `worldmeteorshower:GetRockMoonShellWaveOdds()` 掷月岩壳波次（无该组件时 odds=0）；`level` 缺省随机 1～3；按等级参数随机时长与频率，均有效才开周期任务（`dt=1/rate`）并掷大/中型预算。[实现](../../../raw/current/scripts/components/meteorshower.lua#L217-L236) [月岩壳 odds](../../../raw/current/scripts/components/worldmeteorshower.lua#L14) |
| `StopShower()` | 取消周期任务并清空全部运行态（dt、预算、重试、月岩壳标记）。[实现](../../../raw/current/scripts/components/meteorshower.lua#L238-L250) |
| `StartCooldown()` | StopShower 后按当前等级随机冷却（如 1 级 6 天±4 天，见 [常量](../../../raw/current/scripts/tuning.lua#L3119-L3124)），冷却到期起以 `RETRY_INTERVAL`（SEG_TIME/3）轮询；构造时即调用，构造随机等级。[实现](../../../raw/current/scripts/components/meteorshower.lua#L265-L276) [构造](../../../raw/current/scripts/components/meteorshower.lua#L128-L142) |
| `SpawnMeteor(mod?)` | 在 60 半径内出弦缓动取半径、扇形 30 次试可通行点，生成 `shadowmeteor` 并定大小；`mod` 缩放大/中概率与预算扣费（cost=floor(1/mod+.5)）。外围（半径>50）强制小型；无合法点返回 nil。[实现](../../../raw/current/scripts/components/meteorshower.lua#L152-L198) |
| `IsShowering()`／`IsCoolingDown()` | 爆发中=`dt~=nil`；冷却中=有任务且 `dt==nil`。组件没有“停止”档的自然入口（冷却轮询最终必开雨）。[实现](../../../raw/current/scripts/components/meteorshower.lua#L144-L150) |
| `OnSave()`／`OnLoad(data)` | 保存等级、按当档最大时长归一的剩余时间、间隔、大/中预算与重试数，`version=2`；加载兼容无 version 的旧档写法。[实现](../../../raw/current/scripts/components/meteorshower.lua#L278-L321) |
| `GetDebugString()` | 输出等级、剩余时间、频率修正、库存与重试进度。[实现](../../../raw/current/scripts/components/meteorshower.lua#L323-L328) |

### 三级强度与冷却参数

`SHOWER_LEVELS` 全部取自 `TUNING.METEOR_SHOWER_LVL*`：时长 1/2/3 级为 10～15/15～30/20～50 秒，频率 2～4/3～7/4～10 颗每秒，冷却基数 6/9/12 天加 0～4/0～6/0～8 天浮动（[等级表](../../../raw/current/scripts/components/meteorshower.lua#L7-L110)、[调参](../../../raw/current/scripts/tuning.lua#L3114-L3154)）。大/中型各按 `RandomizeInteger` 掷预算，逐颗扣费；小型不计数。大小概率：`rand<=0.2*mod` 大、`rand<=0.4*mod` 中（[概率](../../../raw/current/scripts/tuning.lua#L3099-L3100)），预算不足自动降级为小。

### 冷却轮询与玩家探测

冷却到期后每 `RETRY_INTERVAL` 检查：生成器 120 半径内有玩家则立即开雨；否则消耗重试次数并顺延（等待路过玩家围观），重试上限 `NUM_RETRIES`=冷却期/轮询间隔≈16 次，耗尽后无条件开雨（[轮询](../../../raw/current/scripts/components/meteorshower.lua#L252-L263)、[重试常量](../../../raw/current/scripts/components/meteorshower.lua#L3-L5)）。爆发中每颗流星判断：90 半径内有玩家满频无修正；否则隔次以 `mod=0.5` 生成（频率减半、大中概率与扣费减半）（[离屏修正](../../../raw/current/scripts/components/meteorshower.lua#L200-L215)）。

## 典型接入

### 调试或脚本触发一次流星雨

- 前提与选择：服务端、目标 meteorspawner 实体已加载。世界生成器按房间摆放，无法从世界组件直接枚举，用控制台查找函数定位实体再调组件接口；level 传 nil 则随机等级。这是调用指引，不新增世界级事件。
- 生效与撤销：`StartShower` 内部先 StopShower，可直接覆盖当前状态；任务挂在 spawner 实体上，实体移除即取消。[覆盖语义](../../../raw/current/scripts/components/meteorshower.lua#L217-L218)

```lua
-- 服务端（如控制台 c_listallplayers 同级环境）：触发最近一个生成器的 3 级流星雨
local spawner = c_findnext("meteorspawner")
if spawner ~= nil then
    spawner.components.meteorshower:StartShower(3)
end
```

- 调整与验证：`c_findnext` 为原版控制台辅助（[实现](../../../raw/current/scripts/consolecommands.lua#L731)）；要持续改频率应修改 TUNING 等级常量并在世界生成前生效，组件构造只捕获一次等级表（[捕获](../../../raw/current/scripts/components/meteorshower.lua#L7-L110)）。

## 常见陷阱与限制

- **以为没玩家就永远不下：** 冷却轮询仅重试约 16 次后无条件开雨；离屏只降频率不停止。想“无人不生成”需自行包一层条件，组件不提供。[无条件开雨](../../../raw/current/scripts/components/meteorshower.lua#L260-L262)
- **把 worldmeteorshower 当作开关：** 世界组件只提供月岩壳波次概率，本组件在每轮 StartShower 时读取一次；两者安装位置、职责不同。[读取点](../../../raw/current/scripts/components/meteorshower.lua#L219-L220)
- **外围落点与预算的连带效果：** `peripheral` 判定在大小选择之前，外围强制小型且不扣预算；预算耗尽的中大型请求静默降级为小型，不报错。[强制小型](../../../raw/current/scripts/components/meteorshower.lua#L178-L193)
- **存档兼容依赖 version 字段：** version 2 把剩余时间按档位最大值归一存储，直接手工改档数据容易写坏节奏；旧档自动按旧公式反归一。[适配](../../../raw/current/scripts/components/meteorshower.lua#L297-L308)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取 meteorshower.lua，核验等级参数、冷却轮询、大小与月岩壳分支、离屏修正、存读兼容与 prefab/世界生成安装点。未运行游戏；shadowmeteor 实体本体与 worldmeteorshower 概率实现见其独立页面。

相关：[[reference/components/worldmeteorshower]]。
