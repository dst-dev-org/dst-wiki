---
title: "walkableplatformplayer"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "玩家侧平台跟随组件：维护玩家与所在平台的双端绑定，驱动船上音乐检测、船用镜头与移动预测碰撞的启停。"
aliases: ["玩家平台跟随", "上船检测", "walkableplatformplayer"]
tags: [component, callback]
dst_build: "steam-buildid:26230496"
sources: ["components/walkableplatformplayer.lua", "prefabs/player_common.lua", "components/walkableplatformmanager.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# walkableplatformplayer

## 快速判断

- 职责与适用：挂在每个玩家（[安装](../../../raw/current/scripts/prefabs/player_common.lua#L2518-L2520)），与 [[reference/components/walkableplatformmanager]]／[[reference/components/walkableplatform]] 协作，处理"玩家所在平台变化"的双端登记与表现；对象是玩家而不是平台。
- 优先使用：不直接调组件；由 walkableplatformmanager 周期调 `TestForPlatform`（[调用](../../../raw/current/scripts/components/walkableplatformmanager.lua#L68-L77)）、mainfunctions 在位移后调用（[调用](../../../raw/current/scripts/mainfunctions.lua#L2122-L2146)）。
- 重要副作用：登船／离船在 master 上登记 `AddPlayerOnPlatform`／`RemovePlayerOnPlatform`，本地玩家另推 `got_on_platform`／`got_off_platform` 事件并启停音乐测试与船镜头（[L198-L242](../../../raw/current/scripts/components/walkableplatformplayer.lua#L198-L242)）。
- 修改边界：客户端预测由 `enablemovementprediction` 事件开关，决定是否 `AddPlatformFollower` 与生成玩家碰撞（[L84-L97](../../../raw/current/scripts/components/walkableplatformplayer.lua#L84-L97)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `TestForPlatform()` | 无参 | master 按 `GetCurrentPlatform`、本地玩家按 `Map:GetPlatformAtPoint` 比对变更并调 GetOn/OffPlatform；其余客户端跳过（[L244-L273](../../../raw/current/scripts/components/walkableplatformplayer.lua#L244-L273)）。 |
| `GetOnPlatform(platform)` / `GetOffPlatform()` | platform 实体 | 维护 `self.platform` 与 `Transform:SetIsOnPlatform`，按端分派登记、碰撞、镜头与音乐（[L198-L242](../../../raw/current/scripts/components/walkableplatformplayer.lua#L198-L242)）。 |
| `StartBoatMusicTest()` / `StopBoatMusicTest()` | 本地玩家限定 | 0.5s 周期测平台位移，速度平方超 0.2 且递增时推 `playboatmusic`（[L1-L14](../../../raw/current/scripts/components/walkableplatformplayer.lua#L1-L14)、[L147-L160](../../../raw/current/scripts/components/walkableplatformplayer.lua#L147-L160)）。 |
| `StartBoatCamera()` / `StopBoatCamera()` | 本地玩家＋非专用 | 经 focalpoint 挂镜头焦点源：按平台速度做前视偏移与 pan gain 渐变（[L16-L66](../../../raw/current/scripts/components/walkableplatformplayer.lua#L16-L66)、[L162-L180](../../../raw/current/scripts/components/walkableplatformplayer.lua#L162-L180)）。 |
| `StartBoatCameraZooms()` / `StopBoatCameraZooms()` | 依赖平台 net 变量 | 监听平台 `doplatformcamerazoomdirty`，0.25 步 ×20 次推 `zoomcamera`（[L100-L123](../../../raw/current/scripts/components/walkableplatformplayer.lua#L100-L123)、[L182-L196](../../../raw/current/scripts/components/walkableplatformplayer.lua#L182-L196)）。 |
| `OnRemoveEntity` / `OnRemoveFromEntity` | 清理 | 移除跟随登记、平台玩家集合与 IsOnPlatform 标记（[L275-L284](../../../raw/current/scripts/components/walkableplatformplayer.lua#L275-L284)）。 |

## 事件与开关（关键流程）

1. 开关事件：构造即监听 `enableboatcamera`／`enablemovementprediction`（由 player_common 在相应设置变化时推，[推方](../../../raw/current/scripts/prefabs/player_common.lua#L879-L947)）与 `playeractivated`（激活后一帧再启动镜头，等平台就位）（[L133-L145](../../../raw/current/scripts/components/walkableplatformplayer.lua#L133-L145)）。
2. 移动预测：开启时对当前平台 `AddPlatformFollower`＋`SpawnPlayerCollision`；关闭只撤碰撞，登记留给下次登船（[L84-L97](../../../raw/current/scripts/components/walkableplatformplayer.lua#L84-L97)）。
3. 镜头生效条件：`boat_camera_enabled`＋平台 `walkableplatform:HasPlatformCamera()`（游船才有）＋本地玩家＋非专用服务器（[L125-L131](../../../raw/current/scripts/components/walkableplatformplayer.lua#L125-L131)）。

## 常见陷阱与限制

- **客户端只跑本地玩家**：`TestForPlatform` 的客户端分支限定 `self.inst == ThePlayer`，其他远程玩家的平台绑定靠 master 登记与网络同步，别在客户端组件里读别人的 `self.platform`（[L255-L272](../../../raw/current/scripts/components/walkableplatformplayer.lua#L255-L272)）。
- **镜头/音乐都默认关**：`boat_camera_enabled` 初值 false，必须由外部推 `enableboatcamera(true)`（原版在玩家设置链路里推）；想给自定义船开镜头需平台侧 `HasPlatformCamera` 同时为真。
- **`doplatformcamerazoom` 挂在实体上**：该 net 变量属于船实体而非 walkableplatform 组件（组件注释自述"应该属于组件但目前在实体上"），自定义平台要自行加同名 net 变量（[组件注释](../../../raw/current/scripts/components/walkableplatform.lua#L20-L21)、[船侧声明](../../../raw/current/scripts/prefabs/boat.lua#L433)）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。覆盖组件全文、player_common 安装与两个事件推方、walkableplatformmanager 调用点与 mainfunctions 调用行、boat.lua 的 net 变量声明；未覆盖 focalpoint 镜头内部与 yotd_boats 变体。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、Lua 语法探测通过（见批级记录）。

相关：[[reference/components/walkableplatform]]、[[reference/components/walkableplatformmanager]]。
