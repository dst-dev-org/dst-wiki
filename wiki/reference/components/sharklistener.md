---
title: "sharklistener"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "世界级鲨鱼音效距离跟踪：周期测量玩家与在场鲨鱼的距离，写入玩家 _sharksoundparam 网络变量驱动本地焦点音效。"
aliases: ["鲨鱼监听", "sharklistener"]
tags: [component, network-sync]
dst_build: "steam-buildid:26230496"
sources: ["components/sharklistener.lua", "prefabs/player_common.lua", "prefabs/shark.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# sharklistener

## 快速判断

- 职责与适用：挂在 TheWorld（forest.lua 注册，[L703](../../../raw/current/scripts/prefabs/forest.lua#L703)）的主机世界组件；监听 `ms_playerjoined/ms_playerleft` 维护玩家表、`sharkspawned` 事件登记鲨鱼（推送方 [shark.lua](../../../raw/current/scripts/prefabs/shark.lua#L349)），每 3 秒重算距离参数。
- 优先使用：想让自定义鲨鱼类实体参与原版危险音效，只需在生成时推 `sharkspawned` 事件；组件无任何可调用的公开操作接口（除 GetDebugString），行为完全内部。
- 重要副作用：直接写玩家实体的 `_sharksoundparam`（net_float，[player_common](../../../raw/current/scripts/prefabs/player_common.lua#L2562)）；本地客户端读该值驱动 TheFocalPoint 的 "shark" 循环音（[L1920-L1928](../../../raw/current/scripts/prefabs/player_common.lua#L1920-L1928)）。
- 修改边界：参数映射固定——30 距离内（DIST=30，[L13](../../../raw/current/scripts/components/sharklistener.lua#L13)）按最近距离 Remap 出 0..1，无接近鲨鱼时 3 秒后置 2（表示静音）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `sharkspawned` 事件消费 | `TheWorld:PushEvent("sharkspawned", {target=实体})` | 登记任意实体进跟踪表并挂 `onremove` 自动清除。[实现](../../../raw/current/scripts/components/sharklistener.lua#L78-L86) |
| 周期复查 | DoPeriodicTask(3, processsharks) | 对每个玩家：范围内存在"非行走"鲨鱼时整体判静音（break），否则取最近距离 `Remap(max(100, distsq), 0, 900, 1, 0)` 写参数；无目标用 killtask 延时置 2。[实现](../../../raw/current/scripts/components/sharklistener.lua#L31-L69) |
| `_sharksoundparam` | 玩家身上的 net_float | 组件写主机值，classified 同步给本地玩家；语义 ≤1 播放并设距离参数、>1 停止。[消费](../../../raw/current/scripts/prefabs/player_common.lua#L1920-L1928) |

## 典型接入

### 让自定义鲨鱼接入原版音效

- 前提与选择：仅主机端推事件；实体应表现鲨鱼的游走行为（"walking" 标签会被用于判断），移除时自动退出跟踪。
- 生效与撤销：推送后 3 秒内开始影响附近玩家音效；实体 `onremove` 自动注销。

```lua
-- 主机端：自定义鲨鱼实体生成后登记
TheWorld:PushEvent("sharkspawned", {target = my_shark_ent})
```

- 调整与验证：距离/延迟常量在组件文件内（DIST=30、周期 3 秒、静音延时 3 秒），无 TUNING 入口；音效名与参数语义固定在 player_common。

## 常见陷阱与限制

- **无组件读取者**：全库没有 `components.sharklistener` 的调用点，逻辑不可外部驱动或查询，改行为只能改本组件或换事件流。
- **mastersim 断言被注释**：构造里的主机断言处于注释状态（[L7](../../../raw/current/scripts/components/sharklistener.lua#L7)），但写入 net 变量、FindEntities 都是主机语境，实际仍应在世界实体（仅主机创建）上运行。
- **一头非行走鲨鱼即静音**：范围内任一鲨鱼无 "walking" 标签就放弃该玩家的距离计算（[L43-L45](../../../raw/current/scripts/components/sharklistener.lua#L43-L45)），登岸/搁浅的鲨鱼会让音效消失。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；组件全文、player_common 的 net 变量与音效消费点、shark.lua 推送点。
- 游戏运行：未运行。
