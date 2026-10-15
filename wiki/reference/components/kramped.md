---
title: "kramped"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "淘气值（naughtiness）与坎普斯惩罚的世界宿主组件：按 NAUGHTY_VALUE 记录每名玩家击杀的淘气度，衰减到阈值即在其 30 距离外生成坎普斯，阈值前播放分级警告音。"
aliases: ["kramped", "淘气值管理器", "坎普斯生成器", "Krampus 惩罚"]
tags: [component, callback, event]
dst_build: "steam-buildid:26230496"
sources: ["components/kramped.lua", "constants.lua", "tuning.lua", "prefabs/forest.lua", "prefabs/cave.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# kramped

## 快速判断

- 职责与适用：挂在 forest 与 cave 世界实体的服务端组件（构造断言 mastersim）；为每名玩家维护 `{actions, threshold, timetodecay}`，监听其 `killed` 事件按 `NAUGHTY_VALUE[prefab]` 累加淘气度，达标生成坎普斯，未达标播延迟警告音。[构造](../../../raw/current/scripts/components/kramped.lua#L7-L13) [击杀入账](../../../raw/current/scripts/components/kramped.lua#L87-L101)
- 优先使用：查/改数值走 `TUNING.KRAMPUS_*` 与 `NAUGHTY_VALUE` 表；强制触发走世界事件 `ms_forcenaughtiness`。[调谐](../../../raw/current/scripts/tuning.lua#L1551-L1556) [淘气值表](../../../raw/current/scripts/constants.lua#L2271-L2294)
- 重要副作用：达到阈值即清零并按世界天数生成 1-2+只坎普斯（`spawnedforplayer` 标记归属）；未达标时 1-3 秒后按差距播 `krampuswarning_lvl1/2/3`。[生成](../../../raw/current/scripts/components/kramped.lua#L66-L84) [警告](../../../raw/current/scripts/components/kramped.lua#L176-L192)
- 修改边界：`TUNING.KRAMPUS_THRESHOLD == -1` 时不监听击杀事件（世界设置关死）；组件 `OnUpdate` 常开做每分钟衰减 1 点；洞穴与地表各自独立计数，互不同步。[关死开关](../../../raw/current/scripts/components/kramped.lua#L127-L129) [挂载](../../../raw/current/scripts/prefabs/forest.lua#L649)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| 玩家登记 | `ms_playerjoined`／`ms_playerleft` | 每人一份 `{player, actions=0, threshold=nil, timetodecay=60s}`；threshold 首次入账时才掷 `KRAMPUS_THRESHOLD(30) + random(VARIANCE 20)`。([kramped.lua:116-130](../../../raw/current/scripts/components/kramped.lua#L116-L130) |
| `killed` 监听 | 实体事件 | `NAUGHTY_VALUE[data.victim.prefab]` 命中才计；猪人变身态（werebeast）、影怪寄生宿主不计；`FunctionOrValue` 支持表值为函数，`stackmult` 乘算批量击杀。([kramped.lua:87-101](../../../raw/current/scripts/components/kramped.lua#L87-L101) [值表](../../../raw/current/scripts/constants.lua#L2271-L2294) |
| 阈值结算 | 私有闭包 `OnNaughtyAction` | `actions >= threshold` 时重置并生成 `1`（50 天前）／`1+random(RAMP 2)`（50-100 天）／`2+random(RAMP)`（100 天后）只坎普斯；`KRAMPUS_INCREASE_RAMP < 1`（即 never 档）直接返回。([kramped.lua:58-85](../../../raw/current/scripts/components/kramped.lua#L58-L85) [难度档](../../../raw/current/scripts/worldsettings_overrides.lua#L3361-L3400) |
| `MakeAKrampusForPlayer` | 私有闭包 | 距玩家 `SPAWN_DIST = 30` 找可行走点（洞内回退 `FindNearbyLand`），生成 `krampus`、面向玩家并挂 `spawnedforplayer`（玩家移除时清空）。([kramped.lua:45-56](../../../raw/current/scripts/components/kramped.lua#L45-L56) |
| `DoWarningSound(player)` | 公开方法 | 差值 <20 时按 <5／<15 分级延迟 1-3 秒播 `krampuswarning_lvl3/2/1`；接近阈值警告更急。([kramped.lua:190-192](../../../raw/current/scripts/components/kramped.lua#L190-L192) [_DoWarningSound](../../../raw/current/scripts/components/kramped.lua#L176-L188) |
| `OnUpdate(dt)` | 组件更新 | 有欠账的玩家每 `KRAMPUS_NAUGHTINESS_DECAY_PERIOD`（60s）减 1 点淘气度；构造即 `StartUpdatingComponent` 常驻。([kramped.lua:194-205](../../../raw/current/scripts/components/kramped.lua#L194-L205) |
| `ms_forcenaughtiness`（监听） | `data = {player, numspawns}` | 重置该玩家淘气计数后直接生成 `numspawns` 只坎普斯并设为目标（克劳斯偷走私鹿时使用）。([kramped.lua:140-156](../../../raw/current/scripts/components/kramped.lua#L140-L156) [klaus.lua:226](../../../raw/current/scripts/prefabs/klaus.lua#L226 |

## 典型接入

### 让自定义生物计入淘气度 / 强制触发坎普斯

- 前提与选择：往 `constants.lua` 的 `NAUGHTY_VALUE` 加键需要改原版常量，更稳的是在自定义生物上让击杀事件携带原版 prefab 名；`FunctionOrValue` 允许表值为 `(player, data) -> number` 函数实现动态淘气度。
- 生效与撤销：组件在玩家入场时挂 `killed` 监听，退出时摘除；`ms_forcenaughtiness` 立即生效且会先清零该玩家已有计数。
- 原版依据：击杀入账与豁免分支见上表；`ms_forcenaughtiness` 唯一脚本消费者为 klaus.lua。

```lua
-- 服务端任意脚本：强制给某玩家生成 1 只敌对坎普斯
TheWorld:PushEvent("ms_forcenaughtiness", { player = ThePlayer, numspawns = 1 })
-- 组件存在性检查（cave/forest 世界默认都有，其他世界类型没有）：
-- TheWorld.components.kramped ~= nil
```

- 调整与验证：`numspawns` 缺省视为 0（什么都不生成）；生成的坎普斯自动仇恨目标玩家，但未被锁定时可正常游走。

## 常见陷阱与限制

- 地表与洞穴各自一份组件、各自计数：把玩家从洞穴传回地表不会合并淘气度，两次"接近阈值"警告可能先后出现（[forest.lua:649](../../../raw/current/scripts/prefabs/forest.lua#L649)、[cave.lua:531](../../../raw/current/scripts/prefabs/cave.lua#L531）。
- `NAUGHTY_VALUE` 未收录的 prefab 完全不计（([kramped.lua:89-90](../../../raw/current/scripts/components/kramped.lua#L89-L90)）；新生物想计入必须改常量表或在击杀侧用同 prefab 名。
- `KRAMPUS_THRESHOLD == -1` 时监听根本不挂（([kramped.lua:127-129](../../../raw/current/scripts/components/kramped.lua#L127-L129)），此时 `ms_forcenaughtiness` 里 `threshold` 被置 nil、后续入账会重新掷阈值——"关死"的世界设置仍可被强制事件绕过。
- 衰减是周期扣 1 而非清零：`OnUpdate` 每 60 秒减 1 点（([kramped.lua:194-205](../../../raw/current/scripts/components/kramped.lua#L194-L205)），长线低频击杀仍可能缓慢攒到阈值。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，通读组件全文；核验 forest/cave 双挂载、NAUGHTY_VALUE 表、KRAMPUS_* 调谐、worldsettings 难度档与 klaus.lua 强制事件源。
- 游戏运行：未运行。
- 结构检查：未执行。

相关链接：[[reference/components/combat]]、[[reference/components/health]]
