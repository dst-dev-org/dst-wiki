---
title: "vaultmusiclistener"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "洞穴玩家的宝库音乐监听器：按 changearea 区块 ID 与守卫仇恨状态驱动分档战斗音乐事件。"
aliases: ["宝库音乐监听", "vault 战斗音乐", "vaultmusiclistener"]
tags: [component]
dst_build: "steam-buildid:26230496"
sources: ["components/vaultmusiclistener.lua", "prefabs/player_common.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# vaultmusiclistener

## 快速判断

- 职责与适用：仅洞穴世界为本地活跃玩家安装（`TheWorld:HasTag("cave")` 分支，[安装](../../../raw/current/scripts/prefabs/player_common.lua#L722-L730)），监听玩家 `changearea` 与宝库守卫仇恨，把音乐层级打包成 `triggeredevent` 事件交给动态音乐系统（[推送](../../../raw/current/scripts/components/vaultmusiclistener.lua#L97)）。
- 优先使用：想触发同款分档危险音乐时参照其 `PushEvent("triggeredevent", {name, level, duration})` 协议；消费端是 `dynamicmusic`（[消费](../../../raw/current/scripts/components/dynamicmusic.lua#L761)）。
- 重要副作用：进入 ID 含 `Vault_Vault` 的区块即 `StartVaultMusic()`，离开发声区块或坠虚空自动停止（[区域判定](../../../raw/current/scripts/components/vaultmusiclistener.lua#L1-L16)）。
- 修改边界：组件本身不发声，只推事件；音乐淡出缓冲用 1 秒/3 秒的静默过渡（[状态机](../../../raw/current/scripts/components/vaultmusiclistener.lua#L61-L99)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `delay` | `number or nil` | 非空即"音乐进行中"（`IsMusicPlaying()`，[实现](../../../raw/current/scripts/components/vaultmusiclistener.lua#L41-L43)）；启动值 3.5 秒倒计时（[启动](../../../raw/current/scripts/components/vaultmusiclistener.lua#L45-L51)）。 |
| `level` | `number or nil` | 音乐档位：2=过渡静默、3=战斗层；nil 回落后台（[状态机](../../../raw/current/scripts/components/vaultmusiclistener.lua#L74-L95)）。 |
| `lastaggrotime` | `number or nil` | 最近 `vault_pillar_guard_aggro` 时刻；1.5 秒内视为仇恨中（[判定](../../../raw/current/scripts/components/vaultmusiclistener.lua#L65-L72)）。 |
| `StartVaultMusic()` / `StopVaultMusic()` | 无参数 | 启动/停止更新并复位 `level`（[实现](../../../raw/current/scripts/components/vaultmusiclistener.lua#L45-L59)）。 |

## 典型接入

### 自定义区域复用 triggeredevent 协议

- 前提与选择：玩家端任意系统可直接向玩家推 `triggeredevent`，无需本组件；原版 alterguardian/beequeen 也是直接推（[例](../../../raw/current/scripts/prefabs/beequeen.lua#L351)）。
- 生效与撤销：`name="vault", level=2/3`；`duration` 仅在回落层给 5，战斗层为 nil。
- 原版依据：`dynamicmusic` 的 `StartTriggeredDanger` 按事件名取配置并播层（[消费](../../../raw/current/scripts/components/dynamicmusic.lua#L761)）。

```lua
-- 服务器脚本内对本地玩家触发一次危险音乐（客户端判定 ThePlayer 亦可）
local player = AllPlayers[1]
player:PushEvent("triggeredevent", { name = "vault", level = 3 })
```

- 调整与验证：`duration` 单位秒；事件名需在 dynamicmusic 配置表内有对应条目，自造名字无效。

## 常见陷阱与限制

- 首次仇恨且 `level` 为空时把 `delay` 直接清 0（[加速](../../../raw/current/scripts/components/vaultmusiclistener.lua#L21-L23)），战斗音乐近乎立即进入过渡层；改启动延迟时别动这段。
- 无 `data.id`（坠入虚空）时用 `TheWorld.Map:IsPointInAnyVault` 兜底防止谜题房间误停（[兜底](../../../raw/current/scripts/components/vaultmusiclistener.lua#L11-L15)）；自建区块 ID 不含 `Vault` 前缀会立刻停音乐。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，覆盖组件全文、player_common 安装/移除点与 dynamicmusic 消费点；未运行游戏。
- 结构检查：validate_page.py、validate_source_links.py 与 Lua 语法探针通过。
