---
title: "shard_autosaver"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "跨分片自动存档快照号同步：主机分片保存时经 net_uint 广播当前快照号，从属分片转成 secondary_autosaverupdate 世界事件。"
aliases: ["跨服自动存档", "shard_autosaver", "分片快照同步"]
tags: [component, network-sync]
dst_build: "steam-buildid:26230496"
sources: ["components/shard_autosaver.lua", "components/autosaver.lua", "prefabs/shard_network.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# shard_autosaver

## 快速判断

- 职责与适用：挂在 `TheWorld.shard` 实体（shard_network prefab）上，仅 mastersim（构造断言，[断言](../../../raw/current/scripts/components/shard_autosaver.lua#L7)、[宿主](../../../raw/current/scripts/prefabs/shard_network.lua#L38)）；把主机分片的自动存档快照号同步给其他分片。
- 优先使用：业务侧不直接操作本组件——主机分片 autosaver 组件在真正执行保存时推 `master_autosaverupdate`（[推送](../../../raw/current/scripts/components/autosaver.lua#L111)），从属分片 autosaver 监听 `secondary_autosaverupdate`（[消费](../../../raw/current/scripts/components/autosaver.lua#L209)），本组件是中间网络桥。
- 重要副作用：主机分片初始化即写入 `TheNet:GetCurrentSnapshot()`（[初始化](../../../raw/current/scripts/components/shard_autosaver.lua#L41)）；从属分片收到 dirty 后向本地 TheWorld 推 `secondary_autosaverupdate`（[转发](../../../raw/current/scripts/components/shard_autosaver.lua#L31-L33)）。
- 修改边界：无公开方法、无存读钩子；快照号只随 net 变量传输，不落盘。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `_snapshot` | `net_uint（"shard_autosaver._snapshot"，dirty "snapshotdirty"）` | 私有网络变量；主机分片写、从属分片读。[定义](../../../raw/current/scripts/components/shard_autosaver.lua#L21) |
| `OnAutoSaverUpdate` | `TheWorld "master_autosaverupdate" -> set` | 仅 `ismastershard` 注册；数据形状 `{ snapshot = number }`。[注册](../../../raw/current/scripts/components/shard_autosaver.lua#L27-L29)、[注册分支](../../../raw/current/scripts/components/shard_autosaver.lua#L39-L48) |
| `OnSnapshotDirty` | `"snapshotdirty" -> PushEvent "secondary_autosaverupdate"` | 仅非主机分片注册；携带 `{ snapshot = _snapshot:value() }`。[转发](../../../raw/current/scripts/components/shard_autosaver.lua#L31-L33) |

## 典型接入

### 感知主机分片完成存档（原版参照）

- 前提与选择：从属分片上的 autosaver 组件用世界事件感知保存进度（[原版消费](../../../raw/current/scripts/components/autosaver.lua#L209)）；mod 同样只监听事件，不触碰 shard 组件。

```lua
-- 从属分片服务端：感知主机分片存档快照变化
TheWorld:ListenForEvent("secondary_autosaverupdate", function(world, data)
    local snapshot = data.snapshot
end)
```

- 调整与验证：事件值为主机分片 `TheNet:GetCurrentSnapshot()` 的快照号；主机分片本地不会收到 `secondary_autosaverupdate`。

## 常见陷阱与限制

- **仅桥接，不做保存**：何时保存、保存什么都在 autosaver 组件侧；本组件不触发任何 I/O。
- **mastersim 断言**：加到非 shard 实体会构造即崩（[断言](../../../raw/current/scripts/components/shard_autosaver.lua#L7)）。
- **单向下行**：只有主机分片 → 从属分片方向；从属分片自身的存档进度不经此组件回传。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；覆盖组件全文、shard_network 宿主注册、autosaver 两侧推送/消费点。
- 结构检查：本批 validate_page、validate_source_links 通过（见批级记录）。
- 游戏运行：未运行。

相关：[[reference/components/shard_worldreset]]、[[reference/components/shard_wagbossinfo]]
