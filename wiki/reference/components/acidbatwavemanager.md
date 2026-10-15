---
title: "acidbatwavemanager"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "酸雨蝙蝠潮世界管理器：酸雨期间跟踪玩家背包硝石数量按几率预警并生成蝙蝠潮，数量达标时额外召回蝙蝠 boss。"
aliases: ["蝙蝠潮", "酸雨蝙蝠", "acidbatwavemanager 组件"]
tags: [component, ai]
dst_build: "steam-buildid:26230496"
size_class: standard
sources: ["components/acidbatwavemanager.lua", "prefabs/cave.lua", "prefabs/bat_boss.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# acidbatwavemanager

## 快速判断

- 职责与适用：洞穴世界的 master 端组件（cave.lua:525 在 master_postinit；:10 断言不许客户端存在）；`isacidraining` 为真期间按玩家背包内 `nitre` 数量滚动概率，预警后在其周围生成蝙蝠（`bat`）潮。
- 优先使用：暂停/恢复蝙蝠潮 → 复用 hounded 的 `pausehounded`/`unpausehounded` 事件（:410-427）；查询某蝙蝠是否本潮生成 → `IsTrackedAcidBat`（bat_boss.lua:436、bat_bossbrain.lua:16）。
- 重要副作用：跟踪中的蝙蝠离场即解除跟踪（:30-38）；蝙蝠 boss 洞穴（`ms_registerbatbosscave`）被注册后可能被抽调生成 boss（:429-432、:64-86）。
- 修改边界：目标物品、距离、数量、冷却全部来自 TUNING（:15-18）；组件只对在线玩家建元数据，玩家离场时暂存数据待回（:215、:259-266）。

## 关键接口

| 接口/字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `TrackAcidBat`／`IsTrackedAcidBat` | `(bat) -> nil`／`(bat) -> boolean` | 注册/查询跟踪表；外部消费见 bat_boss.lua:436、bat_bossbrain.lua:16。:33-50 |
| `StartWatchingPlayer`／`StopWatchingPlayer` | `(player) -> nil` | 监听玩家 itemget/itemlose/newactiveitem/stacksizechange 维护硝石计数（:252-255）；离场且仍在酸雨时把元数据存入 `savedplayermetadata[userid]`（:264-266）。 |
| `UpdateOddsForPlayer` | `(player, meta) -> nil` | 死亡/鬼魂、点位不可酸雨（`Map:CanPointHaveAcidRain`，map.lua:925）、带 `batdisguise` 标签时几率归零；否则 `easing.inQuad(数量, 0, 1, 20)`（:311-329）。 |
| `TryToSpawnWaveForPlayer` | `(player, meta, t) -> nil` | 状态机：冷却中→掷骰（`TryLuckRoll` + `LuckFormulas.AcidBatWave`，componentutil.lua:2129/:2237）→预警→到点生成；`pausesources` 只挡新预警不挡已排的潮（:330-367）。 |
| `OnSave`/`OnLoad`/`LoadPostPass` | 常规存取 | 剩余时间换相对值存 `data.userids`（按 userid），蝙蝠按 GUID 续跟踪（:435-507）。 |
| `OnPostInit` | `() -> nil` | 按当前 `isacidraining` 恢复监听（:305-307）。 |

## 关键流程：一次蝙蝠潮

1. 酸雨开始：`WatchWorldState("isacidraining")`（:304）触发 `StartWatchingPlayers`；首个玩家被监听时 `StartUpdatingComponent`（:247-249）。
2. 每 10s（`update_time_seconds`=10，:21，实际带随机抖动 :393-398）刷新几率；掷骰成功后设 `spawn_wave_time = t + time_for_warning` 并开始警告音效 `acidbatwavewarning_lvl1`（:364-366、:375-382）与台词 `ANNOUNCE_ACIDBATS`（:372-374）。
3. 到点生成：数量 = 按硝石比例在 `ACIDBATWAVE_SPAWN_COUNT_MIN`(2)~MAX(6) 间取整（:130-143），落点用 `FindWalkableOffset` 在 `ACIDBATWAVE_SPAWN_DISTANCE`(12) 半径找无洞点（:88-100）；≥`ACIDBATWAVE_BATBOSS_BATCOUNT`(5) 只时尝试抽调/生成蝙蝠 boss（:145-160）。
4. 生成后立即重算硝石计数并进入冷却 `ACIDBATWAVE_COOLDOWN_BETWEEN_WAVES`（=half day，:343-345）。
5. 蝙蝠追踪：所有潮内蝙蝠注册 onremove；boss 洞穴经 `GetNextAvailableBatBoss` 可离屏强制 `SpawnChild`（:72-79）。

## 典型接入

### 复用暂停事件挡住蝙蝠潮（与 hounded 同一套约定）

- 前提与选择：事件数据须带 `source` 与 `reason`（:412-425），挂在 TheWorld 上推；与 hounded 互相独立（组件注释 :410-411 明确两者并行）。
- 生效与撤销：`pausehounded` 置 `pausesources` 修饰器，`unpausehounded` 按 `data.source` 移除；漏发 unpause 会永久暂停新潮。
- 原版依据：components/acidbatwavemanager.lua:426-427。

```lua
TheWorld:PushEvent("pausehounded", { source = inst, reason = "myevent" })
-- 事件结束后：
TheWorld:PushEvent("unpausehounded", { source = inst, reason = "myevent" })
```

- 调整与验证：reason 可重复用于区分来源；仅对"未进入预警"的玩家生效（:332-334）。

## 常见陷阱与限制

- `StopTrackingFn` 引用不匹配：bat_boss.lua:442 调用 `acidbatwavemanager.StopTrackingFn(inst)`，组件内实际名为 `StopTrackingAcidBatFn`（:40），该分支（跟踪蝙蝠睡眠且近家）会因调用 nil 报错——mod 端勿仿照此名字。
- 计数只看 `nitre`：`target_prefab = "nitre"`（:20）写死；`inventory:GetItemByName` 一次性取至多 20 个（`max_target_prefab`，tuning.lua:7764），改目标物需同步调 TUNING。
- 潮内蝙蝠不属巢：跟踪蝙蝠的归宿逻辑由 bat_boss/bat 脚本按 `IsTrackedAcidBat` 特判（bat_boss.lua:430-446），组件本身不驱动蝙蝠行为。
- 玩家离开仅暂存：`savedplayermetadata` 只在世界仍酸雨时保留（:264-266），酸雨结束统一清空（:287-291）；重进世界经 OnLoad 恢复相对时间。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，通读组件全部 510 行；挂载 cave.lua:525、TUNING tuning.lua:7760-7766/:9660、消费方 bat_boss.lua:382/:436/:442、bat_bossbrain.lua:16、运气公式 componentutil.lua:2129/:2237 逐行核对。
- 游戏运行：未运行。
