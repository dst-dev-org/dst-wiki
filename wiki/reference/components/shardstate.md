---
title: "shardstate"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "世界网络实体的分片状态组件：用 net_string 同步主分片会话 ID，客户端收到后写入会话缓存并序列化玩家。"
aliases: ["shardstate", "主分片会话ID", "mastersessionid", "分片会话同步"]
tags: [component, network]
dst_build: "steam-buildid:26230496"
size_class: simple
sources: ["components/shardstate.lua", "prefabs/world_network.lua", "shardnetworking.lua", "stats.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# shardstate

## 快速判断

- 职责与适用：挂在 `TheWorld.net` 网络实体上（非 TheWorld 本体），维护主分片会话 ID 的网络同步；初始值为本地 `TheWorld.meta.session_identifier`。[world_network.lua:52](../../../raw/current/scripts/prefabs/world_network.lua#L52 [shardstate.lua:47](../../../raw/current/scripts/components/shardstate.lua#L47
- 优先使用：读会话 ID 用 `GetMasterSessionId()`（如原版统计上报）。([shardstate.lua:38-40](../../../raw/current/scripts/components/shardstate.lua#L38-L40) [stats.lua:130](../../../raw/current/scripts/stats.lua#L130
- 重要副作用：非主模拟端收到 netvar dirty 后调 `TheNet:SetClientCacheSessionIdentifier` 并 `SerializeUserSession(ThePlayer, true)`，影响客户端会话缓存。([shardstate.lua:29-32](../../../raw/current/scripts/components/shardstate.lua#L29-L32)
- 修改边界：服务器端监听世界事件 `ms_newmastersessionid` 写网络变量；该事件由 shardnetworking 的 `Shard_UpdateMasterSessionId` 推送。[shardstate.lua:51](../../../raw/current/scripts/components/shardstate.lua#L51 ([shardnetworking.lua:152-154](../../../raw/current/scripts/shardnetworking.lua#L152-L154)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `GetMasterSessionId()` | 字符串（net_string 当前值） | 返回同步的主分片会话 ID；调用前判 `TheWorld.net` 与组件存在。([shardstate.lua:38-40](../../../raw/current/scripts/components/shardstate.lua#L38-L40) [stats.lua:130](../../../raw/current/scripts/stats.lua#L130 |
| `_mastersessionid` | net_string `shardstate._mastersessionid` | 变更事件名 `mastersessioniddirty`；仅主模拟监听世界事件写入。[shardstate.lua:19](../../../raw/current/scripts/components/shardstate.lua#L19 ([shardstate.lua:25-27](../../../raw/current/scripts/components/shardstate.lua#L25-L27) |
| `OnMasterSessionIdDirty`（内部，非主模拟） | dirty 回调 | 写客户端缓存会话标识并立即序列化 ThePlayer 会话。([shardstate.lua:29-32](../../../raw/current/scripts/components/shardstate.lua#L29-L32) |

## 典型接入

### 读取主分片会话 ID

- 前提与选择：组件随 world_network 自动存在，模组无需安装；路径是 `TheWorld.net.components.shardstate`。[world_network.lua:52](../../../raw/current/scripts/prefabs/world_network.lua#L52

```lua
local sessionid = TheWorld.net ~= nil
    and TheWorld.net.components.shardstate ~= nil
    and TheWorld.net.components.shardstate:GetMasterSessionId()
    or nil
```

- 调整与验证：统计场景下主／从分片读值；纯客户端该值由网络同步维护。

## 常见陷阱与限制

- 组件在 `TheWorld.net` 上：`TheWorld.components.shardstate` 恒为 nil，写错宿主读不到。[world_network.lua:52](../../../raw/current/scripts/prefabs/world_network.lua#L52
- 无存读钩子：net 实体 `persists=false`，组件仅靠事件与 netvar 维持状态。([world_network.lua:45-46](../../../raw/current/scripts/prefabs/world_network.lua#L45-L46)
- 主模拟端只写不读：dirty 回调仅在非主模拟注册，主模拟端 SetClientCacheSessionIdentifier 分支不存在。([shardstate.lua:25-32](../../../raw/current/scripts/components/shardstate.lua#L25-L32)

## 验证记录

- 源码审阅：2026-10-28，`steam-buildid:26230496`：审阅组件全文、world_network 安装上下文、事件推送端与 stats.lua 读取点。
- 游戏运行：未运行；未测多分片实际同步。
- 结构检查：validate_page.py 通过；validate_source_links.py --strict 通过（本批 16 页合并执行）。
