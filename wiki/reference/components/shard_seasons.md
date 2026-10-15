---
title: "shard_seasons"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "跨分片季节同步组件：主分片把 seasons 组件的季节进度写进 net 变量组，次分片监听 dirty 后向本地世界推 secondary_seasonsupdate 供其 seasons 组件消费。"
aliases: ["跨分片季节同步", "shard_seasons"]
tags: [component, network-sync]
dst_build: "steam-buildid:26230496"
size_class: simple
sources: ["components/shard_seasons.lua", "prefabs/shard_network.lua", "components/seasons.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# shard_seasons

## 快速判断

- 职责与适用：挂 shard 网络实体（prefabs/shard_network.lua:33，TheWorld.net、不休眠、不持久化）；仅主模拟（assert，components/shard_seasons.lua:7）。net 变量：4×`net_byte` 季节长度 + season/totaldays/remainingdays（net_byte）、elapseddays（net_ushortint）、endlessdays（net_bool）（:27-35）。
- 优先使用：mod 一般不直调——主分片 seasons.lua:234 推 `master_seasonsupdate`，本组件写入 net（:41-78）；次分片 `seasonsdirty` 事件触发组包（:80-94）后向 TheWorld 推 `secondary_seasonsupdate`，由次分片 seasons 组件监听（components/seasons.lua:423、:475）。
- 重要副作用：无实体状态，纯转发；值变化逐项 set，不主动打包 diff。
- 修改边界：`NUM_SEASONS = 4`，注释要求与 seasons.lua 的 SEASON_NAMES 保持同步（:13）；按 `ismastershard` 决定监听哪一侧（:100-106）。

## 关键接口

| 接口/字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `OnSeasonsUpdate`（主分片） | `(src, data) -> nil` | 逐项对比 set net 值；data 字段=lengths/season/totaldaysinseason/remainingdaysinseason/elapseddaysinseason/endlessdaysinseason。:41-78 |
| `OnSeasonsDirty`（次分片） | `() -> nil` | 读全部 net 值组装同名 data 表，`_world:PushEvent("secondary_seasonsupdate", data)`。:80-94 |
| net 变量组 | 9 个 | 脏事件统一为 `seasonsdirty`（:31-35）。 |

## 典型接入

### 原版参照：次分片季节消费

- 前提与选择：次分片的 seasons 组件拿到的季节进度完全来自本组件转发的 `secondary_seasonsupdate`；mod 想感知次分片季节应监听该世界事件，而非读 net 变量。
- 生效与撤销：组件无 OnSave——数据源头始终是主分片 seasons 组件，重载后由主分片重推。

## 常见陷阱与限制

- season/totaldays/remainingdays 用 net_byte（0-255），季节长度超 255 天会被截断（:29-33 的类型选择是硬上限）。
- 仅主分片监听 `master_seasonsupdate`、仅次分片监听 net dirty（:100-106），同组件双端行为由 `ismastershard` 分叉。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，通读组件全部 112 行，核验挂载（shard_network.lua:33）、上游事件源（seasons.lua:234）与下游消费（seasons.lua:423/475）。
- 游戏运行：未运行。
