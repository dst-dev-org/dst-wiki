---
title: "rabbitkingmanager"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "世界级兔子王管理组件：累积喂胡萝卜与淘气值生成 passive／aggressive／lucky 三种兔子王，负责拴绳追踪、追逐清扫、冷却与存档重连。"
aliases: ["兔子王管理器", "rabbitkingmanager"]
tags: [component, creature]
dst_build: "steam-buildid:26230496"
sources: ["components/rabbitkingmanager.lua", "prefabs/rabbithole.lua", "prefabs/rabbitking.lua", "components/quaker.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# rabbitkingmanager

## 快速判断

- 职责与适用：世界级组件（仅主端，构造时断言 `ismastersim`），挂在洞穴与地表世界上（[cave 装配](../../../raw/current/scripts/prefabs/cave.lua#L526)、[forest 装配](../../../raw/current/scripts/prefabs/forest.lua#L679)）；驱动兔子王这一单实例生物的生成入口（喂胡萝卜、淘气值、运气事件）与在场期间的拴绳清扫。
- 优先使用：模组不直接创建兔子王时无需接入；需要以代码触发走 `CreateRabbitKingForPlayer(player, pt, forcedstate, params)`（[L256-L287](../../../raw/current/scripts/components/rabbitkingmanager.lua#L256-L287)），判可否生成先查 `ShouldStopActions`（[L247-L249](../../../raw/current/scripts/components/rabbitkingmanager.lua#L247-L249)）。
- 重要副作用：追踪期间给兔子王与拴绳玩家挂 `onremove`／`attacked`／`death` 监听并 `StartUpdatingComponent`（[L162-L175](../../../raw/current/scripts/components/rabbitkingmanager.lua#L162-L175)）；玩家击杀兔子类生物会经 `killed` 事件累积淘气值（[L441-L462](../../../raw/current/scripts/components/rabbitkingmanager.lua#L441-L462)）。
- 修改边界：生成走固定三种 `SPAWN_PREFABS`（[L22-L26](../../../raw/current/scripts/components/rabbitkingmanager.lua#L22-L26)），类型由计数来源决定，组件不管兔子王内部 AI；非 `lucky` 被移除后进入 `TUNING.RABBITKING_COOLDOWN`（1 个游戏日，[tuning](../../../raw/current/scripts/tuning.lua#L8488)、[L190-L192](../../../raw/current/scripts/components/rabbitkingmanager.lua#L190-L192)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `STATES` / `SPAWN_PREFABS` | `PASSIVE=0, AGGRESSIVE=1, LUCKY=2`；对应 `rabbitking_passive/_aggressive/_lucky` | 状态→生成 prefab 的映射，PASSIVE 是缺省回退（[L12-L26](../../../raw/current/scripts/components/rabbitkingmanager.lua#L12-L26)）。 |
| `ResetCounters()` | — | 重置并随机化阈值：`carrots_fed_max = RABBITKING_CARROTS_NEEDED + rand(VARIANCE)`（12+0..3），淘气同理 20+0..9（[L30-L36](../../../raw/current/scripts/components/rabbitkingmanager.lua#L30-L36)、[tuning](../../../raw/current/scripts/tuning.lua#L8475-L8478)）。 |
| `ShouldStopActions()` | → bool | 已有兔子王、等待玩家重连（`pendingplayerload`）或冷却中为 true；喂食／淘气／生成入口都先查它（[L247-L249](../../../raw/current/scripts/components/rabbitkingmanager.lua#L247-L249)）。 |
| `CanFeedCarrot(player)` | → bool | `not ShouldStopActions() and carrots_fed < carrots_fed_max`（[L346-L352](../../../raw/current/scripts/components/rabbitkingmanager.lua#L346-L352)）；[兔洞喂食入口](../../../raw/current/scripts/prefabs/rabbithole.lua#L203-L204)。 |
| `AddCarrotFromPlayer(player, receiver)` | — | 计数加一，达到上限即对喂食者 `CreateRabbitKingForPlayer(player)`；仍会播放短促音效（[L353-L366](../../../raw/current/scripts/components/rabbitkingmanager.lua#L353-L366)）。 |
| `AddNaughtinessFromPlayer(player, naughtiness)` | — | 淘气累积，达上限以 `"aggressive"` 强制生成；同样先查 `ShouldStopActions`（[L389-L402](../../../raw/current/scripts/components/rabbitkingmanager.lua#L389-L402)）。 |
| `CreateRabbitKingForPlayer(player, pt_override, forcedstate_string, params)` | → `bool, reason` | reason 取 `"ON_COOLDOWN"`／`"ALREADY_EXISTS"`／`"NO_VALID_SPAWNPOINT"`／`"PREFAB_FAILED_TO_CREATE"`；`params.home` 附加 homeseeker、`params.jumpfrominventory`／`params.nopresentation` 改变登场表现（[L256-L287](../../../raw/current/scripts/components/rabbitkingmanager.lua#L256-L287)、[homeseeker](../../../raw/current/scripts/components/rabbitkingmanager.lua#L236-L244)）。 |
| `GetRabbitKing()` / `GetTargetPlayer()` | → 实体或 nil | 当前追踪的兔子王与拴绳玩家；rabbitking prefab 用后者做表现（[L250-L255](../../../raw/current/scripts/components/rabbitkingmanager.lua#L250-L255)、[消费](../../../raw/current/scripts/prefabs/rabbitking.lua#L170-L172)）。 |
| `GetStateByItemCountsForPlayer(player)` | → `STATES` | 玩家带肉（`HasMeatInInventoryFor`）→ AGGRESSIVE，否则 PASSIVE；LUCKY 只能由 forcedstate 指定（[L432-L438](../../../raw/current/scripts/components/rabbitkingmanager.lua#L432-L438)）。 |
| `RemoveRabbitKing(rabbitking)` | — | 未登场（intro task 在跑）静默 `Remove`；睡眠中直接移除；否则推 `burrowaway` 走钻地表现（[L288-L304](../../../raw/current/scripts/components/rabbitkingmanager.lua#L288-L304)）。 |

## 关键流程

### 生成与追踪

1. 生成：`CreateRabbitKingForPlayer` 选点失败返回 `NO_VALID_SPAWNPOINT`（无洞好地面，[L100-L103](../../../raw/current/scripts/components/rabbitkingmanager.lua#L100-L103)、[选点半径](../../../raw/current/scripts/components/rabbitkingmanager.lua#L203-L214)）；成功即 `TrackRabbitKingForPlayer`：拴绳玩家、监听 onremove／attacked、启动每秒清扫；非 aggressive 种类设 `OnEntitySleep = Remove`（玩家走远即消失，[L162-L175](../../../raw/current/scripts/components/rabbitkingmanager.lua#L162-L175)）。
2. 淘气来源：按玩家监听 `killed`，受害者带 `rabbit` 或 `manrabbit` 标签且非影怪寄生宿主时，取 `NAUGHTY_VALUE[victim.prefab] or 1`（[L441-L451](../../../raw/current/scripts/components/rabbitkingmanager.lua#L441-L451)；表定义见 [constants.lua](../../../raw/current/scripts/constants.lua#L2271)）。
3. 移除与冷却：`UnTrackRabbitKing` 清全部监听，非 lucky 设 1 日冷却；玩家移除／死亡时把拴绳转给最近的旧拴绳玩家，无人则移除（[L48-L65](../../../raw/current/scripts/components/rabbitkingmanager.lua#L48-L65)、[L176-L194](../../../raw/current/scripts/components/rabbitkingmanager.lua#L176-L194)）。

### 追逐清扫（OnUpdate，1 秒节拍）

- aggressive 在场时每秒做两件事：周围 12 距离内玩家带肉则升级为 aggressive（`ReplacePrefab`，[L135-L155](../../../raw/current/scripts/components/rabbitkingmanager.lua#L135-L155)）；与拴绳玩家距离超过 30（平方 900）就钻地追击——玩家站在船上或非地面点则直接移除（[L109-L134](../../../raw/current/scripts/components/rabbitkingmanager.lua#L109-L134)、[TUNING](../../../raw/current/scripts/tuning.lua#L8472-L8474)）。

### 存档与重连

- OnSave 记计数、冷却、`rabbitkingid`／`playerid`／`homeid`；LoadPostPass 等对应 userid 玩家重连再重新拴绳，最长 5 分钟否则移除（[L513-L543](../../../raw/current/scripts/components/rabbitkingmanager.lua#L513-L543)、[L558-L590](../../../raw/current/scripts/components/rabbitkingmanager.lua#L558-L590)）。

## 常见陷阱与限制

- **世界单实例约束**：rabbitking prefab 自检"世界只该有一只"，多余实例生成时调 `TryForceRabbitKing_Internal`，已有王在场则新实例被移除或钻地退场（[prefab 自检](../../../raw/current/scripts/prefabs/rabbitking.lua#L8-L16)、[L305-L344](../../../raw/current/scripts/components/rabbitkingmanager.lua#L305-L344)）。用 `c_spawn` 调试同理。
- **喂食入口在兔洞而非组件**：模组要改"怎么喂"去 `rabbithole` 的 giver 逻辑；lucky 出生点绑定兔洞 `home`（[rabbithole](../../../raw/current/scripts/prefabs/rabbithole.lua#L203-L228)）。quaker 的幸运事件则以 `nopresentation` 直接落下并返回实体（[quaker 消费](../../../raw/current/scripts/components/quaker.lua#L333-L336)）。
- **掉落物重新投掷**：捡起后放出的兔子王走 `jumpfrominventory` 参数重新追踪（[rabbitking 消费](../../../raw/current/scripts/prefabs/rabbitking.lua#L533-L541)），调试命令支持 forcedstate（[debugcommands](../../../raw/current/scripts/debugcommands.lua#L383-L398)）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。覆盖组件全文、rabbithole／rabbitking／quaker 消费点、TUNING 与 NAUGHTY_VALUE、cave／forest 装配行；未覆盖 rabbitking prefab 的状态图与技能细节。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、Lua 语法探测通过（见批级记录）。

相关：[[reference/components/quaker]]。
