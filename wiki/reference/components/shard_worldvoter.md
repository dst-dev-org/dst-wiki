---
title: "shard_worldvoter"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "跨分片投票镜像：把主分片的投票状态（倒计时、命令、投票者、禁言名单）经网络变量转播到从属分片事件。"
aliases: ["分片投票", "shard_worldvoter", "跨分片投票同步"]
tags: [component, network-sync]
dst_build: "steam-buildid:26230496"
sources: ["components/shard_worldvoter.lua", "components/worldvoter.lua", "prefabs/shard_network.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# shard_worldvoter

## 快速判断

- 职责与适用：挂在世界 shard 网络实体（[安装](../../../raw/current/scripts/prefabs/shard_network.lua#L37)），仅 master sim（[断言](../../../raw/current/scripts/components/shard_worldvoter.lua#L7)）；纯状态转播器，没有任何公开方法，接口全部是事件与网络变量。
- 优先使用：投票的发起与结算仍在 [[reference/components/playervoter]] 所在的 worldvoter 流程；本组件只负责把状态搬到另一分片。
- 重要副作用：mastershard 上收 `master_worldvoterupdate` 等事件写网络变量（[监听](../../../raw/current/scripts/components/shard_worldvoter.lua#L144-L146)），从属分片把脏事件转成 `secondary_worldvoterupdate` 等世界事件（[转发](../../../raw/current/scripts/components/shard_worldvoter.lua#L84-L99)）。
- 修改边界：事件由 worldvoter 推送与消费（[推送](../../../raw/current/scripts/components/worldvoter.lua#L206)、[消费](../../../raw/current/scripts/components/worldvoter.lua#L479-L481)），本组件不发起投票。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `master_worldvoterenabled/update/squelchedupdate`（事件） | worldvoter 推送 | mastershard 写入 `_enabled`/`_countdown`/`_commandid` 等网络变量（[写入](../../../raw/current/scripts/components/shard_worldvoter.lua#L51-L82)）。 |
| `secondary_worldvoterenabled/update/squelchedupdate`（事件） | 从属分片推送 | 脏事件还原成 data 表交给 worldvoter（[还原](../../../raw/current/scripts/components/shard_worldvoter.lua#L55-L57)、[名单](../../../raw/current/scripts/components/shard_worldvoter.lua#L122-L132)）。 |
| `_voters` | 64 组 `net_string userid` + `net_tinybyte selection` | 槽位上限 `MAX_SLOTS = 64`（[槽位](../../../raw/current/scripts/components/shard_worldvoter.lua#L13)、[建槽](../../../raw/current/scripts/components/shard_worldvoter.lua#L39-L45)）。 |
| `CANNOT_VOTE` / `VOTE_PENDING` | 0 / `MAX_VOTE_OPTIONS + 1` | 选举值哨兵，与 playervoter/worldvoter 保持一致（[哨兵](../../../raw/current/scripts/components/shard_worldvoter.lua#L16-L17)）。 |
| `_squelched` | 64 个 `net_string` | 禁言名单槽，用空闲池回收槽位（[池](../../../raw/current/scripts/components/shard_worldvoter.lua#L101-L120)）。 |

## 典型接入

### 在从属分片读取投票状态

- 前提与选择：不直接读网络变量，监听转播事件拿快照数据。
- 生效与撤销：事件由组件在脏事件时自动推送，无需接入清理。
- 原版依据：worldvoter 在从属分片上监听这三个转播事件驱动本地 UI（[消费](../../../raw/current/scripts/components/worldvoter.lua#L479-L481)），启用状态经 `master_worldvoterenabled` 下发（[下发](../../../raw/current/scripts/components/worldvoter.lua#L494)）。

```lua
-- 从属分片上监听主分片转播的投票快照
TheWorld:ListenForEvent("secondary_worldvoterupdate", function(world, data)
    -- data.countdown / data.commandid / data.targetuserid
    -- data.starteruserid / data.voters（userid -> selection）
end)
```

- 调整与验证：selection 数值语义由投票器约定，组件只透传（[透传](../../../raw/current/scripts/components/shard_worldvoter.lua#L68-L76)）。

## 常见陷阱与限制

- 64 槽硬上限：超过 64 名投票者时 `OnVoterUpdate` 直接 return，剩余投票者丢弃（[截断](../../../raw/current/scripts/components/shard_worldvoter.lua#L71-L74)）；worldvoter 侧对禁言名单也有对应上限注释（[对齐](../../../raw/current/scripts/components/worldvoter.lua#L240)）。
- mastershard 与从属分片走完全不同的分支（初始化按 `ismastershard` 二选一，[初始化](../../../raw/current/scripts/components/shard_worldvoter.lua#L138-L152)），调试时先确认所在分片角色。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，覆盖组件全文、shard_network 安装点与 worldvoter 推送/消费链；未运行游戏。
- 结构检查：validate_page.py、validate_source_links.py 与 Lua 语法探针通过。
