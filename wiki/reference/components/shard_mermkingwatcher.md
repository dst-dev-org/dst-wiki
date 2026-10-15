---
title: "shard_mermkingwatcher"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "鱼人王存在与三件贡品（三叉戟／王冠／肩甲）的跨分片同步桥：主分片聚合各分片上报，经网络变量下发，各分片转推 *_anywhere 世界事件。"
aliases: ["鱼人王跨分片同步", "鱼人王贡品同步", "shard_mermkingwatcher"]
tags: [component, network-sync]
dst_build: "steam-buildid:26230496"
sources: ["components/shard_mermkingwatcher.lua", "prefabs/shard_network.lua", "components/mermkingmanager.lua", "shardnetworking.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# shard_mermkingwatcher

## 快速判断

- 职责与适用：挂在世界实体 `TheWorld.shard` 上（[安装](../../../raw/current/scripts/prefabs/shard_network.lua#L40)，[断言](../../../raw/current/scripts/components/shard_mermkingwatcher.lua#L6)），把"任一分片是否存在鱼人王及其三件贡品"同步给所有分片；无存档、无清理钩子，纯事件＋网络变量桥。
- 优先使用：查"全分片是否鱼人王"用 [[reference/components/mermcandidate]] 所在的 mermkingmanager 的 `HasKingAnywhere` 系列（[封装](../../../raw/current/scripts/components/mermkingmanager.lua#L288-L333)）；监听变化用各分片转推的 `onmermkingcreated_anywhere` 等世界事件（[消费例](../../../raw/current/scripts/prefabs/wurt.lua#L677-L680)）。
- 重要副作用：四个布尔网络变量（king／trident／crown／pauldron）任一变化触发同名 `*dirty` 事件；非主分片收包后转推 `*_anywhere` 世界事件驱动本地鱼人守卫、沃特等（[脏处理](../../../raw/current/scripts/components/shard_mermkingwatcher.lua#L207-L238)）。
- 修改边界：构造即断言 `TheWorld.ismastersim`，客户端不存在；宿主 shard 实体 `persists = false`，状态由主分片持续重推（[宿主](../../../raw/current/scripts/prefabs/shard_network.lua#L23-L30)）。

## 关键接口

| 接口／字段 | 已核验类型 | 条件、作用与证据 |
|---|---|---|
| `AddMermKingSource(source)` / `RemoveMermKingSource(source)` | source 为 shardid | 主分片聚合：按来源（分片 id）记录存在性；首添转 `onmermkingcreated_anywhere`，清空转 destroyed（[L17-L33](../../../raw/current/scripts/components/shard_mermkingwatcher.lua#L17-L33)）。trident／crown／pauldron 三组同构（[L53-L69](../../../raw/current/scripts/components/shard_mermkingwatcher.lua#L53-L69)）。 |
| `HasMermKing()` / `HasTrident()` / `HasCrown()` / `HasPauldron()` | → `bool` | 返回网络变量值，主次分片都可用（[L35-L37](../../../raw/current/scripts/components/shard_mermkingwatcher.lua#L35-L37)）；mermkingmanager 的 `HasXxxAnywhere` 即此封装。 |
| `ResyncNetVars()` | 无参数 | 主分片专用：对 4 个网络变量做 set_local(false)+set 重广播，供晚接入分片取值（[L189-L204](../../../raw/current/scripts/components/shard_mermkingwatcher.lua#L189-L204)）；[调用点](../../../raw/current/scripts/shardnetworking.lua#L50-L55)。 |
| 4 组 net_bool | `mermkingwatcher.hasmermking` 等 | 脏事件名 `hasmermkingdirty` 等；非主分片在 DoTaskInTime(0) 后监听脏事件（[声明](../../../raw/current/scripts/components/shard_mermkingwatcher.lua#L13-L15)、[监听](../../../raw/current/scripts/components/shard_mermkingwatcher.lua#L242-L247)）。 |
| 事件 `onmermkingcreated`／`onmermkingdestroyed`（监听） | 监听 TheWorld | 主侧：鱼人王创建／销毁时由 mermkingmanager 在 TheWorld 上推事件，本组件转 `Shard_SyncMermKingExists`（[监听](../../../raw/current/scripts/components/shard_mermkingwatcher.lua#L45-L46)、[推事件](../../../raw/current/scripts/components/mermkingmanager.lua#L152)）；贡品事件由 mermking 推（[例](../../../raw/current/scripts/prefabs/mermking.lua#L194)）。 |

## 数据流

- 非主分片：本地鱼人王变化 → mermkingmanager 推 `onmermkingcreated` → 本组件调 `Shard_SyncMermKingExists(true)`；非主分片下该函数经 SHARD_RPC 发往主分片（[shardnetworking](../../../raw/current/scripts/shardnetworking.lua#L234-L246)）。
- 主分片（mastershard）：收到 `master_shardmermkingexists`／`master_shardmermkingtrident` 等事件，按 `data.shardid` 增删 SourceModifierList 来源并写网络变量（[L154-L187](../../../raw/current/scripts/components/shard_mermkingwatcher.lua#L154-L187)）；网络变量同步到各分片后触发脏事件。
- 各分片：脏事件处理转推 `onmermkingcreated_anywhere`／`onmermkingdestroyed_anywhere`／贡品四个 `*_anywhere` 事件；trident／crown／pauldron 的脏处理先查本地 mermkingmanager，本地已有该贡品时不重复推事件（[L215-L238](../../../raw/current/scripts/components/shard_mermkingwatcher.lua#L215-L238)）。

## 常见陷阱与限制

- **监听注册有延迟**：非主分片的 4 个脏事件监听在 `DoTaskInTime(0)` 后才注册，注释说明是等网络反序列化先行；构造当帧内同步到的值不会触发转推（[L240-L247](../../../raw/current/scripts/components/shard_mermkingwatcher.lua#L240-L247)）。
- **贡品事件语义**：`onmermkingtridentadded` 等由 mermking 吃到物品／读档时在 TheWorld 上推，与王的存在性独立同步；两个分片可以一个有王无贡品、另一个有贡品记录（[推事件](../../../raw/current/scripts/prefabs/mermking.lua#L194-L199)）。
- 组件无 OnSave/OnLoad/OnRemoveFromEntity；重启后由主分片数据与 `ResyncNetVars` 恢复，不应补存读。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。覆盖组件全文、shard_network 挂载段、mermkingmanager 四个 Anywhere 封装、shardnetworking 的 Sync 与 Resync 两端及 wurt／mermwatchtower 消费行；未覆盖网络层底层同步与专用服务器表现。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、Lua 语法探测通过（见批级记录）。

相关：[[reference/components/shard_clock]]、[[reference/components/mermcandidate]]（本分片鱼人王候选登记）。
