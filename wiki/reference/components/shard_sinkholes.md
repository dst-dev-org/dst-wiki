---
title: "shard_sinkholes"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "挂在世界 shard 网络实体上的跨分片组件：把「蚁后塌陷」目标玩家列表在分片间同步为 warn/attack 状态。"
aliases: ["塌陷跨分片同步", "蚁后塌陷目标同步", "shard_sinkholes"]
tags: [component, network-sync]
dst_build: "steam-buildid:26230496"
sources: ["components/shard_sinkholes.lua", "prefabs/shard_network.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# shard_sinkholes

## 快速判断

- 职责与适用：由 `shard_network` 装到 `TheWorld.shard`（[安装](../../../raw/current/scripts/prefabs/shard_network.lua#L34)），最多同步 2 名塌陷目标；构造断言仅 master sim（[断言](../../../raw/current/scripts/components/shard_sinkholes.lua#L7)）。
- 优先使用：非 master 分片的世界塌陷系统（`caveins`）监听组件转发的 `secondary_sinkholesupdate` 还原本地目标。
- 重要副作用：mastershard 把地表 `sinkholespawner` 的目标表编码进 `userhash`+`state` 网络变量（[编码](../../../raw/current/scripts/components/shard_sinkholes.lua#L40-L62)）。
- 修改边界：目标槽位固定 `MAX_TARGETS = 2`（[常量](../../../raw/current/scripts/components/shard_sinkholes.lua#L13)）；无存档，数据实时重建。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `_targets[i].userhash` | `net_hash`，脏事件 `sinkholesdirty` | 目标玩家哈希；0 表示空槽（[清槽](../../../raw/current/scripts/components/shard_sinkholes.lua#L44-L46)）。[声明](../../../raw/current/scripts/components/shard_sinkholes.lua#L27-L34) |
| `_targets[i].state` | `net_tinybyte`，脏事件 `sinkholesdirty` | 0=无、1=warn（先 `set_local(1)` 再 `set(1)`）、2=attack（[状态机](../../../raw/current/scripts/components/shard_sinkholes.lua#L53-L60)）。 |

## 关键流程

1. master 分片：`sinkholespawner` 推 `master_sinkholesupdate` `{targets={{userhash, warn, attack}}}`（[推送](../../../raw/current/scripts/components/sinkholespawner.lua#L181-L194)），组件逐槽写入 netvar（[监听](../../../raw/current/scripts/components/shard_sinkholes.lua#L84-L90)）。
2. 从属分片：`sinkholesdirty` 触发后重建 targets 表并推世界事件 `secondary_sinkholesupdate`（[转发](../../../raw/current/scripts/components/shard_sinkholes.lua#L64-L78)）。
3. 消费端 `caveins` 在非 master 分片监听该事件，驱动本地塌陷预警/落石（[消费](../../../raw/current/scripts/components/caveins.lua#L145)）。

## 典型接入

### 在非 master 分片读取塌陷目标

- 前提与选择：需要玩家级塌陷目标时不要直读 netvar，监听世界事件即可拿到解码后的表。
- 生效与撤销：事件在每次网络同步后推送，数据瞬时有效，需要自行缓存。
- 原版依据：`caveins` 的 `OnSinkholesUpdate` 直接使用 `data.targets`（[消费](../../../raw/current/scripts/components/caveins.lua#L88-L91)）。

```lua
-- 任意服务器系统内（非 master 分片）
TheWorld:ListenForEvent("secondary_sinkholesupdate", function(world, data)
    for _, t in ipairs(data.targets) do
        -- t.userhash, t.warn, t.attack
    end
end)
```

- 调整与验证：`warn`/`attack` 字段是 `or nil` 压缩的布尔，缺省即 false；目标超过 2 个时组件只保留前 2 个并清空余槽（[截断](../../../raw/current/scripts/components/shard_sinkholes.lua#L41-L46)）。

## 常见陷阱与限制

- `warn` 用 `set_local` + `set` 双写（[实现](../../../raw/current/scripts/components/shard_sinkholes.lua#L54-L55)）是刻意的本地即时生效；仿写时不要漏掉 `set`，否则跨分片不同步。
- 组件不校验 `userhash` 对应玩家是否存在，消费端需自行用 `TheNet:GetUserIDFromHash` 类接口解析（`caveins` 内已处理）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，覆盖组件全文、shard_network 安装点、sinkholespawner 推送与 caveins 消费链；未运行游戏。
- 结构检查：validate_page.py、validate_source_links.py 与 Lua 语法探针通过。

## 相关链接

- [[reference/components/caveins]]：洞穴落石执行器：中继塌陷目标与本地重演。
