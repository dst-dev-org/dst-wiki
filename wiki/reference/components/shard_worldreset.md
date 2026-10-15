---
title: "shard_worldreset"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "跨服世界重置倒计时同步：主机分片 worldreset 倒计时经 net_byte 广播，从属分片转成 secondary_worldresetupdate 世界事件。"
aliases: ["跨服重置同步", "shard_worldreset", "分片重置倒计时"]
tags: [component, network-sync]
dst_build: "steam-buildid:26230496"
sources: ["components/shard_worldreset.lua", "components/worldreset.lua", "prefabs/shard_network.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# shard_worldreset

## 快速判断

- 职责与适用：挂在 `TheWorld.shard` 实体（shard_network prefab）上，仅 mastersim（构造即断言，[断言](../../../raw/current/scripts/components/shard_worldreset.lua#L7)、[宿主](../../../raw/current/scripts/prefabs/shard_network.lua#L36)）；把主机分片的世界重置倒计时同步到其他分片。
- 优先使用：业务侧不要直接操作本组件——主机分片的 worldreset 组件推 `master_worldresetupdate`（[推送](../../../raw/current/scripts/components/worldreset.lua#L146)），从属分片的 worldreset 监听 `secondary_worldresetupdate`（[消费](../../../raw/current/scripts/components/worldreset.lua#L238)），本组件只是中间的网络桥。
- 重要副作用：主机分片（`ismastershard`）监听 TheWorld 事件并 `set` net 变量；从属分片监听 `countdowndirty` 后向本地 TheWorld 推 `secondary_worldresetupdate`（[转发](../../../raw/current/scripts/components/shard_worldreset.lua#L31-L33)）。
- 修改边界：组件无公开方法、无存读钩子；倒计时值仅随 net 变量传输，不落盘。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `_countdown` | `net_byte（"shard_worldreset._countdown"，dirty "countdowndirty"）` | 私有网络变量；主机分片写、从属分片读。[定义](../../../raw/current/scripts/components/shard_worldreset.lua#L21) |
| `OnCountdownUpdate` | `TheWorld "master_worldresetupdate" -> set` | 仅 `ismastershard` 注册（[注册](../../../raw/current/scripts/components/shard_worldreset.lua#L39-L45)）；数据形状 `{ countdown = number }`。 |
| `OnCountdownDirty` | `"countdowndirty" -> PushEvent "secondary_worldresetupdate"` | 仅非主机分片注册；携带 `{ countdown = _countdown:value() }`。[转发](../../../raw/current/scripts/components/shard_worldreset.lua#L31-L33) |

## 典型接入

### 监听本分片感知到的重置倒计时（原版参照）

- 前提与选择：从属分片上的系统按 worldreset 组件同款方式监听世界事件即可，不触碰 shard 组件（[原版消费](../../../raw/current/scripts/components/worldreset.lua#L238)）。

```lua
-- 从属分片服务端：感知主机分片的世界重置倒计时
TheWorld:ListenForEvent("secondary_worldresetupdate", function(world, data)
    local countdown = data.countdown
end)
```

- 调整与验证：事件仅在存在跨分片重置流程时触发；主机分片本地不会收到 `secondary_worldresetupdate`（它走 `master_worldresetupdate`）。

## 常见陷阱与限制

- **仅桥接，不承载重置逻辑**：倒计时怎么来（worldreset 组件）与怎么执行重置都在 worldreset 侧；改倒计时规则要改那边，本组件照搬值。
- **mastersim 断言**：组件构造即断言 `TheWorld.ismastersim`（[断言](../../../raw/current/scripts/components/shard_worldreset.lua#L7)），加到其他实体会崩；它只属于 shard_network 实体。
- **无存读**：组件没有 OnSave/OnLoad，重置倒计时跨存档不保留，恢复语义由 worldreset 侧决定。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；覆盖组件全文、shard_network 宿主注册、worldreset 两侧事件推送/消费点。
- 结构检查：本批 validate_page、validate_source_links 通过（见批级记录）。
- 游戏运行：未运行。

相关：[[reference/components/shard_autosaver]]
