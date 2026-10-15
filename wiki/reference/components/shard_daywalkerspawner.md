---
title: "shard_daywalkerspawner"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "挂在世界 shard 网络实体上的跨分片组件：同步「挖掘者」当前被囚位置（洞穴/地表垃圾堆）。"
aliases: ["挖掘者跨分片位置", "daywalker 位置同步", "shard_daywalkerspawner"]
tags: [component, network-sync]
dst_build: "steam-buildid:26230496"
sources: ["components/shard_daywalkerspawner.lua", "prefabs/shard_network.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# shard_daywalkerspawner

## 快速判断

- 职责与适用：由 `shard_network` 装到 `TheWorld.shard`（[安装](../../../raw/current/scripts/prefabs/shard_network.lua#L39)），保存并跨分片同步 daywalker（挖掘者）当前所在囚禁点；构造断言仅 master sim（[断言](../../../raw/current/scripts/components/shard_daywalkerspawner.lua#L4)）。
- 优先使用：洞穴/地表两边的生成器组件读取 `GetLocationName()` 决定本轮是否生成 daywalker。
- 重要副作用：mastershard 收到 `master_shardbossdefeated`（daywalker 击败）后在 `cavejail`/`forestjunkpile` 间翻转位置（[处理](../../../raw/current/scripts/components/shard_daywalkerspawner.lua#L51-L62)）。
- 修改边界：位置枚举仅两个且 net_tinybyte 上限 7（[枚举](../../../raw/current/scripts/components/shard_daywalkerspawner.lua#L11-L15)）；位置随世界存档走 `OnSave`/`OnLoad`（[存读](../../../raw/current/scripts/components/shard_daywalkerspawner.lua#L63-L74)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `DAYWALKERLOCATION` | 表 `{cavejail=0, forestjunkpile=1}` | 位置枚举；`UpdateLocationNames()` 反向建 `DAYWALKERLOCATION_LOOKUP`（[构建](../../../raw/current/scripts/components/shard_daywalkerspawner.lua#L16-L22)）。 |
| `location` | `net_tinybyte`，名 `daywalkernetwork.location`，脏事件 `locationdirty` | 跨分片同步的原始值；构造末尾 `set(0)` 即 cavejail（[初始化](../../../raw/current/scripts/components/shard_daywalkerspawner.lua#L89)）。 |
| `GetLocation()` / `GetLocationName()` | → `number` / `string or nil` | 读当前位置；未知枚举值名字为 nil。 |
| `SetLocation(location)` | 参数 `string or number` | 字符串经枚举表转数字（未知名回退 0）后 `set`（[实现](../../../raw/current/scripts/components/shard_daywalkerspawner.lua#L33-L39)）。 |
| `GetNewLocationName(oldlocation)` | 仅 mastershard | `cavejail`↔`forestjunkpile` 互换，其他返回 nil（[实现](../../../raw/current/scripts/components/shard_daywalkerspawner.lua#L43-L50)）。 |

## 典型接入

### 生成器按跨分片位置门控

- 前提与选择：洞穴侧 `daywalkerspawner` 与地表侧 `forestdaywalkerspawner` 各自检查位置名，只有"自己关着"的一方倒计时生成。
- 生效与撤销：组件随 shard 实体常驻；`master_shardbossdefeated` 由 `shardnetworking` 在 boss 战报内推送（[推送](../../../raw/current/scripts/shardnetworking.lua#L225)）。
- 原版依据：洞穴生成器要求 `GetLocationName() == "cavejail"`（[消费](../../../raw/current/scripts/components/daywalkerspawner.lua#L199-L201)）；地表侧要求 `"forestjunkpile"`（[消费](../../../raw/current/scripts/components/forestdaywalkerspawner.lua#L48-L50)、[另处](../../../raw/current/scripts/components/wagpunk_manager.lua#L324-L326)）。

```lua
-- 自定义生成器内读取（服务器，任一分片）
local shard_comp = TheWorld.shard ~= nil and TheWorld.shard.components.shard_daywalkerspawner or nil
if shard_comp ~= nil and shard_comp:GetLocationName() == "forestjunkpile" then
    -- daywalker 当前在地表垃圾堆下
end
```

- 调整与验证：扩展新位置需同时扩 `DAYWALKERLOCATION`、`GetNewLocationName` 与 net_tinybyte 上限内的枚举值。

## 常见陷阱与限制

- 非 mastershard 分支只有空的 `OnLocationDirty` 桩（[桩](../../../raw/current/scripts/components/shard_daywalkerspawner.lua#L79-L85)）：从属分片不做额外反应，读侧直接轮询 `GetLocationName()`。
- `GetNewLocationName` 对两个已知值之外返回 nil，随后 `SetLocation(nil)` 会把非字符串原样 `set`；当前事件流只会传入两枚举值，扩展时注意补默认值。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，覆盖组件全文、shard_network 安装点、shardnetworking 事件推送与 daywalkerspawner/forestdaywalkerspawner/wagpunk_manager 消费链；未运行游戏。
- 结构检查：validate_page.py、validate_source_links.py 与 Lua 语法探针通过。
