---
title: "shard_wagbossinfo"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "Wagstaff 击败状态跨分片同步：主机分片从 wagboss_tracker 读值经 net_bool 广播，其他分片读 net 值感知。"
aliases: ["跨服Wagstaff信息", "shard_wagbossinfo", "分片首领击败状态"]
tags: [component, network-sync]
dst_build: "steam-buildid:26230496"
sources: ["components/shard_wagbossinfo.lua", "components/wagboss_tracker.lua", "prefabs/shard_network.lua", "components/brightmarespawner.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# shard_wagbossinfo

## 快速判断

- 职责与适用：挂在 `TheWorld.shard` 实体（shard_network prefab）上，仅 mastersim（构造断言，[断言](../../../raw/current/scripts/components/shard_wagbossinfo.lua#L7)、[宿主](../../../raw/current/scripts/prefabs/shard_network.lua#L41)）；把"Wagstaff 是否已被击败"同步给其他分片。
- 优先使用：读侧直接调 `IsWagbossDefeated()`（[读取](../../../raw/current/scripts/components/shard_wagbossinfo.lua#L27-L29)）；写侧是主机分片的 [[reference/components/wagboss_tracker]]——状态变化推 `master_wagbossinfoupdate`（[推送](../../../raw/current/scripts/components/wagboss_tracker.lua#L9)）。
- 重要副作用：主机分片初始化时从 `wagboss_tracker:IsWagbossDefeated()` 取初值（[初始化](../../../raw/current/scripts/components/shard_wagbossinfo.lua#L43-L47)）；原版消费为跃行者刷怪系数提升（[消费](../../../raw/current/scripts/components/brightmarespawner.lua#L94-L96)）。
- 修改边界：无公开写接口（写只由事件驱动）、无存读钩子；`isdefeateddirty` 事件无监听者，组件不向客户端转事件。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `IsWagbossDefeated()` | `-> bool` | 返回 net_bool `_isdefeated` 当前值；任何 mastersim 分片可读。[实现](../../../raw/current/scripts/components/shard_wagbossinfo.lua#L27-L29) |
| `_isdefeated` | `net_bool（"shard_wagbossinfo._isdefeated"）` | 私有网络变量；主机分片写（事件 `master_wagbossinfoupdate`，数据 `{ isdefeated = bool }`），其余分片只读。[定义](../../../raw/current/scripts/components/shard_wagbossinfo.lua#L21) |

## 典型接入

### 感知 Wagstaff 击败状态（原版参照）

- 前提与选择：非主机分片的系统经 `TheWorld.shard.components.shard_wagbossinfo` 读取（[原版消费](../../../raw/current/scripts/components/brightmarespawner.lua#L94-L96)）；主机分片可直接读本地 wagboss_tracker。

```lua
local info = TheWorld.shard ~= nil and TheWorld.shard.components.shard_wagbossinfo or nil
if info ~= nil and info:IsWagbossDefeated() then
    -- 本分片已知 Wagstaff 被击败
end
```

- 调整与验证：需要"状态变化即响应"时监听 TheWorld 的 `master_wagbossinfoupdate`（仅主机分片）或轮询读取；本组件不转发 dirty 事件给 Lua。

## 常见陷阱与限制

- **客户端拿不到事件**：net_bool 会把值同步到客户端，但组件不监听 `isdefeateddirty`，客户端 Lua 侧没有现成回调路径；UI 需求要另建 classified 通道。
- **值域仅一个 bool**：wagboss 击败细节（时间、次数）不在同步范围；主机分片外只能感知"是/否"。
- **依赖 shard 实体**：读侧必须判 `TheWorld.shard` 非空（分片实体在移除时会置空，[置空](../../../raw/current/scripts/prefabs/shard_network.lua#L12-L17)）。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；覆盖组件全文、shard_network 宿主注册、wagboss_tracker 推送点、brightmarespawner 消费点、`isdefeateddirty` 全库检索。
- 结构检查：本批 validate_page、validate_source_links 通过（见批级记录）。
- 游戏运行：未运行。

相关：[[reference/components/wagboss_tracker]]、[[reference/components/shard_autosaver]]
