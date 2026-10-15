---
title: "dynamicmusic"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "世界动态音乐组件：监听激活玩家的游戏事件，在 TheFocalPoint 上分 busy/danger/pirates 三通道切主题并管理优先级、延长与时段间奏。"
aliases: ["动态音乐", "战斗音乐切换", "dynamicmusic"]
tags: [component, ui]
dst_build: "steam-buildid:26230496"
sources: ["components/dynamicmusic.lua", "prefabs/cave.lua", "prefabs/forest.lua", "prefabs/player_common.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# dynamicmusic

## 快速判断

- 职责与适用：装在洞穴与地表世界上（[cave](../../../raw/current/scripts/prefabs/cave.lua#L340)、[forest](../../../raw/current/scripts/prefabs/forest.lua#L627)），双端存在但只在有本地焦点玩家时工作：跟随 `playeractivated` 给激活玩家挂事件监听，向 `TheFocalPoint.SoundEmitter` 播放（[L864-L882](../../../raw/current/scripts/components/dynamicmusic.lua#L864-L882)、[L833-L843](../../../raw/current/scripts/components/dynamicmusic.lua#L833-L843)）。
- 优先使用：模组播自家战斗曲走玩家 `triggeredevent` 事件（`{name, level, duration}` 匹配 `TRIGGERED_DANGER_MUSIC`，未注册名回退 ruins 曲，[L62-L217](../../../raw/current/scripts/components/dynamicmusic.lua#L62-L217)、[L659-L681](../../../raw/current/scripts/components/dynamicmusic.lua#L659-L681)）；活动 BGM 走 `play<theme>music` 系列事件（[监听表](../../../raw/current/scripts/components/dynamicmusic.lua#L754-L777)）。
- 重要副作用：切换主题会 `KillSound("busy")` 打断旧曲；疯狂/启蒙/昼夜切换播 stinger 并把 `_extendtime` 推迟 15s 抑制新曲（[L736-L752](../../../raw/current/scripts/components/dynamicmusic.lua#L736-L752)、[L804-L827](../../../raw/current/scripts/components/dynamicmusic.lua#L804-L827)）。
- 修改边界：无存档、无主端逻辑；音乐名/参数全在 `TheFocalPoint` 声音总线（busy/danger/pirates），组件不管理音量设置。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `ShouldPlayDangerMusic(player, target)`（全局函数） | → bool | 目标需有 combat replica；排除猎物/鸟/影怪方/墙/蔬菜等标签；己方跟随的 shadowminion/abigail 不算威胁（[L2-L26](../../../raw/current/scripts/components/dynamicmusic.lua#L2-L26)）。 |
| `triggeredevent` 消费 | `{name, level, duration}` | level 选 `TRIGGERED_DANGER_MUSIC[name][level]`（空串＝静音层），同 level 只延长；与 busy/danger 通道互斥优先（[L659-L681](../../../raw/current/scripts/components/dynamicmusic.lua#L659-L681)）。 |
| `enabledynamicmusic` 世界事件 | bool | 幽灵化关闭（[player_common](../../../raw/current/scripts/prefabs/player_common.lua#L956)）、伍迪变身期间关闭（[woodie](../../../raw/current/scripts/prefabs/woodie.lua#L425-L452)）；关闭时清空 busy/danger（[L884-L894](../../../raw/current/scripts/components/dynamicmusic.lua#L884-L894)）。 |
| busy 主题入口 | `buildsuccess`/`performaction(working)`→StartBusy 15s；`playboatmusic`→StartOcean 30s；`isfeasting`/`playracemusic`/`playhermitmusic`/`playtrainingmusic`/`playfarmingmusic`/`playcarnivalmusic`/`stageplaymusic`/`playpillowfightmusic`/`playrideofthevalkyrie`/`playboatracemusic`/`playbalatromusic` → StartBusyTheme | 洞穴分 RUINS（areaaware `Nightmare`）/CAVE 主题；地表分 LUNARISLAND（`lunacyarea`）/FOREST 按季节曲目；非洞穴仅白天播 busy（[L270-L346](../../../raw/current/scripts/components/dynamicmusic.lua#L270-L346)、[L447-L467](../../../raw/current/scripts/components/dynamicmusic.lua#L447-L467)）。 |
| danger 入口 | `performaction(attack)` 与 `attacked` | 30 距离内有 epic（无 noepicmusic）选史诗曲，否则危险曲，按废墟/洞穴/季节取变体；10s 或 `gotnewitem` 等延长（[L627-L657](../../../raw/current/scripts/components/dynamicmusic.lua#L627-L657)、[受击过滤](../../../raw/current/scripts/components/dynamicmusic.lua#L716-L729)）。 |
| pirates 通道 | `playpiratesmusic` 事件启动 | 40 距离 `crewmember` 实体按距离 remap 音量；danger 在播时海盗 intensity 压到 0.5（[L554-L604](../../../raw/current/scripts/components/dynamicmusic.lua#L554-L604)、[L578-L597](../../../raw/current/scripts/components/dynamicmusic.lua#L578-L597)）。 |
| `hasinspirationbuff` 事件 | `{on}` | 沃拓克斯灵感 buff 调 danger 的 `wathgrithr_intensity` 参数（[L731-L734](../../../raw/current/scripts/components/dynamicmusic.lua#L731-L734)）。 |
| `wx_performedspinaction` | `{isattack}` | WX-78 旋转模块攻击/工作分流入 danger/busy（[L708-L714](../../../raw/current/scripts/components/dynamicmusic.lua#L708-L714)）。 |

## 关键流程

### 通道优先级与延长

1. 优先级固定 danger > pirates > busy：任何 danger/pirates 启动前先 `StopBusy`；busy 存在时再触发只把 `_extendtime` 推 15s（[L311-L316](../../../raw/current/scripts/components/dynamicmusic.lua#L311-L316)、[L634](../../../raw/current/scripts/components/dynamicmusic.lua#L634)）。
2. `gotnewitem` 触发 `ExtendBusy`（+10s）；到期检查 `_extendtime`，未到点重排任务（[L288-L309](../../../raw/current/scripts/components/dynamicmusic.lua#L288-L309)、[L548-L552](../../../raw/current/scripts/components/dynamicmusic.lua#L548-L552)）。
3. 时段间奏：非洞穴世界在 dawn/dusk 播 stinger，之后 15 秒抑制；洞穴世界不 WatchWorldState，只按 `_iscave` 恒可播 busy（[L804-L827](../../../raw/current/scripts/components/dynamicmusic.lua#L804-L827)、[L837-L841](../../../raw/current/scripts/components/dynamicmusic.lua#L837-L841)）。
4. 玩家切换（换人/退出）时全部监听与声音清理重建（[L864-L882](../../../raw/current/scripts/components/dynamicmusic.lua#L864-L882)）。

## 常见陷阱与限制

- **`triggeredevent` 名要先注册**：`TRIGGERED_DANGER_MUSIC` 表外的 name 一律回退 `default`（废墟史诗曲），想静音需显式给空串条目（[L213-L217](../../../raw/current/scripts/components/dynamicmusic.lua#L213-L217)）。
- **busy 在夜晚地表被静默拒绝**：`StartBusy` 第一行 `not (_iscave or _isday)` 直接 return，且 `playboatmusic` 等主题事件同样受此限制；需要夜间音乐必须走 triggeredevent/danger 通道（[L312-L313](../../../raw/current/scripts/components/dynamicmusic.lua#L312-L313)、[L355-L356](../../../raw/current/scripts/components/dynamicmusic.lua#L355-L356)）。
- **海盗音量独立于 danger**：海盗音乐以自身实体距离定音量，danger 播放时只把 intensity 参数降半，不会停掉海盗通道（[L582-L590](../../../raw/current/scripts/components/dynamicmusic.lua#L582-L590)）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。覆盖组件全文、cave/forest 装配行、player_common 与 woodie 的 enabledynamicmusic 推送；未覆盖各 boss prefab 推 triggeredevent 的具体时机与舞台剧活动内部。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、Lua 语法探测通过（见批级记录）。
