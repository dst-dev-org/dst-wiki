---
title: "quagmire_music"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "大亨（Gorge）模式挂在世界 net 实体上的 BGM 组件：按网络关卡号在本地玩家焦点端播放厨房音乐。"
aliases: ["大亨音乐", "Gorge 背景音乐", "quagmire_music"]
tags: [component, network]
dst_build: "steam-buildid:26230496"
sources: ["components/quagmire_music.lua", "prefabs/quagmire_network.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# quagmire_music

## 快速判断

- 职责与适用：`quagmire_network` 装到 `TheWorld.net`（[安装](../../../raw/current/scripts/prefabs/quagmire_network.lua#L53)），客户端侧随玩家激活把 `level` netvar 映射为两首厨房 BGM，胜利时播 `gorge_win`（[曲目](../../../raw/current/scripts/components/quagmire_music.lua#L11-L15)、[胜利](../../../raw/current/scripts/components/quagmire_music.lua#L51-L53)）。
- 优先使用：模组做模式 BGM 可参照其"服务器写 level netvar → 客户端 OnLevelDirty 换曲"结构。
- 重要副作用：激活玩家变化时Kill/重建 `bgm` 声音并重绑监听，焦点端用 `TheFocalPoint.SoundEmitter`（[激活](../../../raw/current/scripts/components/quagmire_music.lua#L66-L77)）。
- 修改边界：服务器写逻辑经 `event_server_data("quagmire", ...)` 注入（[注入点](../../../raw/current/scripts/components/quagmire_music.lua#L90-L92)），对应事件服文件不在本 scripts 快照内。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `_netvars.level` | `net_tinybyte`，名 `lavaarenamusic._netvars.level`，脏事件 `leveldirty` | 关曲目索引（1/2），构造时 `set(1)`（[初始化](../../../raw/current/scripts/components/quagmire_music.lua#L84)）；netvar 名沿用熔炉前缀系源码现状（[声明](../../../raw/current/scripts/components/quagmire_music.lua#L31-L35)）。 |
| `_netvars.won` | `net_event`，名 `lavaarenamusic._netvars.won` | 胜利一次性事件，客户端播 `dontstarve/quagmire/music/gorge_win`。 |
| `OnLevelDirty`（内部） | 监听 `leveldirty` | 与 `_levelplaying` 不同才 `KillSound("bgm")` 并播 `MUSIC[level]`，未知索引静默不播（[实现](../../../raw/current/scripts/components/quagmire_music.lua#L41-L49)）。 |

## 典型接入

### 参照实现模式音乐同步

- 前提与选择：组件随模式 worldgen 的 net prefab 存在，客户端无感知即可听；服务端写值点在事件服文件（未在快照内核验）。
- 生效与撤销：`playeractivated`/`playerdeactivated`（世界事件）驱动 per-player 声音生命周期；玩家断线自动清（[清理](../../../raw/current/scripts/components/quagmire_music.lua#L55-L64)）。
- 原版依据：与 `lavaarenamusic` 同构，可互为参照（[相关页](lavaarenamusic.md)）。
- 调整与验证：组件未公开任何方法，扩曲库需直接改组件内 `MUSIC` 表（PostInit 覆盖亦可）；netvar 名不可改（客户端监听依赖该字符串）。无原版方法级消费点，不提供接入代码。

## 常见陷阱与限制

- netvar 名是 `lavaarenamusic._netvars.*`（源码复制痕迹，[声明](../../../raw/current/scripts/components/quagmire_music.lua#L33-L34)）：监听/覆写时必须用这个字符串，不要想当然写成 `quagmire_music._netvars.*`。
- `OnLevelDirty` 在 `_soundemitter` 为 nil（无玩家激活）时不安全地直接调用；组件依赖先 `playeractivated` 再有脏事件的世界时序，单独实例化调试可能报错。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，覆盖组件全文与 quagmire_network 安装点；事件服服务器写值逻辑不在快照内未核验，未运行游戏。
- 结构检查：validate_page.py、validate_source_links.py 与 Lua 语法探针通过。

相关：[lavaarenamusic](lavaarenamusic.md)
