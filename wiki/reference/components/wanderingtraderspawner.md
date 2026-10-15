---
title: "wanderingtraderspawner"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "流浪商人生成调度世界组件：登记玩家出生点作生成点，世界初始化时按世界设置开关生成或移除唯一的 wanderingtrader，并跨存档维护其引用。"
aliases: ["流浪商人生成器", "流浪商人调度", "wanderingtraderspawner"]
tags: [component, creature, lifecycle]
dst_build: "steam-buildid:26230496"
size_class: simple
sources: ["components/wanderingtraderspawner.lua", "prefabs/forest.lua", "prefabs/spawnpoint_multiplayer.lua", "tuning.lua", "worldsettings_overrides.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# wanderingtraderspawner

## 快速判断

- 职责与适用：挂 forest 世界（prefabs/forest.lua:737），仅服务器（assert，components/wanderingtraderspawner.lua:6）；生成点来自 `ms_registerspawnpoint` 世界事件（:95）——由 spawnpoint_multiplayer/spawnpoint_master 在世界生成时推送（prefabs/spawnpoint_multiplayer.lua:13）。
- 优先使用：外部生成入口是 `TryToSpawnWanderingTrader()`（有商人在场或无点时返回 false，:26-44）；世界 `DoTaskInTime(0)` 的 OnWorldPostInit 按 `TUNING.WANDERINGTRADER_ENABLED` 生成或移除（:98-106）。
- 重要副作用：商人 onremove 自动清引用（:12-18）；随机洗牌后优先普通出生点、无则 master 出生点。
- 修改边界：OnSave 记商人 GUID，LoadPostPass 重建引用（:53-69）；`TUNING.WANDERINGTRADER_ENABLED` 默认 true，受世界设置 wanderingtrader 控制（tuning.lua:8744、worldsettings_overrides.lua:3081-3085）。

## 关键接口

| 接口/字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `TryToSpawnWanderingTrader` | `() -> boolean` | `wanderingtrader` 不在场且至少一个生成点才执行；洗牌取点后 `Transform:SetPosition` 对齐。:26-44 |
| `SpawnWanderingTrader`/`RemoveWanderingTrader` | `() -> entity/nil` | 底层生成（直接 SpawnPrefab）与移除；注释要求外部走 TryToSpawn。:20-24、:46-50 |
| `OnRegisterSpawnPoint` | `(inst, spawnpoint) -> nil` | 按 `spawnpoint.master` 分两组登记并挂 onremove 反注册。:79-93 |
| `OnSave`/`LoadPostPass` | 常规存取 | GUID 跨存档重挂，`newents` 未命中则静默丢弃。:53-69 |

## 典型接入

### 世界就绪后手动生成一名流浪商人（服务器）

- 前提与选择：原版唯一自动生成时机是 OnWorldPostInit；mod 世界已就绪后可直接调 TryToSpawn。
- 生效与撤销：直接调用不经过 `TUNING.WANDERINGTRADER_ENABLED` 开关，需要尊重世界设置时自行判断；商人被移除后不会自动重生。

```lua
-- modmain.lua（仅服务器）
AddSimPostInit(function()
    if TheWorld.ismastersim then
        TheWorld:DoTaskInTime(1, function()
            if TUNING.WANDERINGTRADER_ENABLED then
                TheWorld.components.wanderingtraderspawner:TryToSpawnWanderingTrader()
            end
        end)
    end
end)
```

- 调整与验证：世界未生成任何出生点实体时恒返回 false。

## 常见陷阱与限制

- 生成点即玩家出生点组件，无专用商人点；`spawnpoint.master` 是"固定出生模式主点"标记，不是 master sim（spawnpoint_multiplayer.lua:8-10）。
- 组件不监听商人死亡以外的事件，商人 despawn 后引用清空但不会补货。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，通读组件全部 108 行，核验挂载（forest.lua:737）、生成点事件源（spawnpoint_multiplayer.lua:13）与开关取值（tuning.lua:8744、worldsettings_overrides.lua:3081-3085）；全库无其他 TryToSpawn 调用者。
- 游戏运行：未运行。
