---
title: "shard_players"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "跨分片玩家计数：主分片汇总本分片与从属分片的玩家/幽灵数，经网络变量广播并聚合成 ms_playercounts 事件。"
aliases: ["分片玩家计数", "shard_players", "跨分片人数"]
tags: [component, network-sync]
dst_build: "steam-buildid:26230496"
sources: ["components/shard_players.lua", "prefabs/shard_network.lua", "components/sanity.lua", "components/worldreset.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# shard_players

## 快速判断

- 职责与适用：挂在世界 shard 网络实体（`TheWorld.shard`，[安装](../../../raw/current/scripts/prefabs/shard_network.lua#L35)），仅 master sim（[断言](../../../raw/current/scripts/components/shard_players.lua#L7)）；mastershard 负责计算，其余分片只做镜像转发。
- 优先使用：读人数用 `GetNumPlayers/GetNumGhosts/GetNumAlive`（任一端），或监听聚合事件 `ms_playercounts`。
- 重要副作用：计数变化统一推世界事件 `ms_playercounts`，带 `total/ghosts/alive` 三值（[推送](../../../raw/current/scripts/components/shard_players.lua#L97-L104)）。
- 修改边界：无存读（世界实体不持久化）；只有 mastershard 才监听生成/离开/变鬼事件（[初始化](../../../raw/current/scripts/components/shard_players.lua#L110-L124)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `GetNumPlayers()` / `GetNumGhosts()` / `GetNumAlive()` | → `number` | 读网络变量值，全端可用（[实现](../../../raw/current/scripts/components/shard_players.lua#L133-L143)）。 |
| `_numPlayers` / `_numGhosts` | `net_byte`，脏事件 `playercountsdirty` | 广播总数与幽灵数（[网络变量](../../../raw/current/scripts/components/shard_players.lua#L29-L30)）。 |
| `ms_playercounts`（事件） | `{total, ghosts, alive}` | 计数脏事件触发时由组件推给 TheWorld（[转换](../../../raw/current/scripts/components/shard_players.lua#L97-L104)）。 |
| `master_secondaryplayerschanged`（事件） | 从 TheShard 汇入 | mastershard 收到从属分片人数变化后重算（[监听](../../../raw/current/scripts/components/shard_players.lua#L112)）。 |

## 典型接入

### 按全分片人数门控逻辑

- 前提与选择：读数在任意端可用；事件监听要挂在 `TheWorld` 上。
- 生效与撤销：组件随 shard 实体常驻，无需接入清理。
- 原版依据：噩梦疯狂按幽灵/存活数取参数（[sanity](../../../raw/current/scripts/components/sanity.lua#L458-L459)）；重置投票界面监听计数（[worldreset](../../../raw/current/scripts/components/worldreset.lua#L196-L201)）。

```lua
-- 方式一：直接读（任意端）
local sp = TheWorld.shard ~= nil and TheWorld.shard.components.shard_players or nil
if sp ~= nil then
    local alive = sp:GetNumAlive()
end

-- 方式二：监听聚合事件（任意端）
TheWorld:ListenForEvent("ms_playercounts", function(world, data)
    -- data.total / data.ghosts / data.alive
end)
```

- 调整与验证：数值含从属分片玩家（[汇总](../../../raw/current/scripts/components/shard_players.lua#L51-L56)），比本分片遍历 `AllPlayers` 的口径大。

## 常见陷阱与限制

- 非 mastershard 分片不计算本地人数，只把网络变量转发成 `ms_playercounts`（[转发](../../../raw/current/scripts/components/shard_players.lua#L127)）：在这类分片上直接数 `AllPlayers` 得到的是本分片值，与本组件口径不同。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，覆盖组件全文、shard_network 安装点、sanity 与 worldreset 消费链；未运行游戏。
- 结构检查：validate_page.py、validate_source_links.py 与 Lua 语法探针通过。
