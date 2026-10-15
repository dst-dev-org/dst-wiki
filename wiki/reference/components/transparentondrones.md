---
title: "transparentondrones"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "客户端无人机重叠淡出组件：多组无人机同时出现在屏幕附近时把他人无人机透明化并允许鼠标穿透，WX-78 影子无人机专用表现。"
aliases: ["无人机透明", "无人机淡出", "transparentondrones"]
tags: [component, ui]
dst_build: "steam-buildid:26230496"
size_class: simple
sources: ["components/transparentondrones.lua", "prefabs/wx78_shadowdrone_debuffer.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# transparentondrones

## 快速判断

- 职责与适用：纯客户端表现组件（文件头注记 lines 1-2："No server logic should be driven off this component"）。同一玩家的无人机按领队 userid 分组，当屏幕上有多组无人机且本组在附近时，把"非自己那组"的无人机渐隐到 `fadealpha`（components/transparentondrones.lua:160-174）。
- 优先使用：只随宿主自动工作——原版唯一挂载点是 WX-78 影子无人机（prefabs/wx78_shadowdrone_debuffer.lua:214，且仅在非专用服务器端 `AddComponent`）。
- 重要副作用：覆写 `inst.CanMouseThrough`，透明度低于 0.5 时允许鼠标点击穿透（lines 10-13、24）；同时给 `inst.fx` 特效半强度透明（lines 27-32）。
- 修改边界：无网络字段、无存读钩子；实体休眠即停更、复位透明度并注销分组（lines 103-110）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `fadealpha` | 默认 0.1 | 渐隐目标的透明度（line 17） |
| `fadedist`／`unfadedist` | 默认 5／8 | 进入／退出淡出的判定半径，带 3 单位滞回防抖（lines 20-21、160） |
| `RegisterDrone(leaderid)`／`UnregisterDrone()` | 模块级分组表 | 按 leaderid 维护 `AllDrones`／`NearGroups` 计数（lines 4-8、41-68） |
| `SetIsNear(near)` | 邻近状态 | 只有同组全部离开才清组标记（lines 70-89） |
| `IsNearMe(range)` | `-> bool` | 本地玩家或本地玩家组内任一无人机进入 range 即近（lines 119-139） |
| `OnUpdate(dt)` | 主循环 | 首次经 `replica.follower` 取领队并推算 leaderid（玩家 userid 或 `linkeditem:GetOwnerUserID()`，lines 143-150）；本地玩家是领队则常显并停更（lines 152-157） |
| `PushAlpha(a)`／`ResetAlpha()` | 表现 | `AnimState:OverrideMultColour`，fx 减半（lines 27-39） |

## 原版参照

- WX-78 影子无人机（debuffer）：客户端分支创建影子特效后挂组件（wx78_shadowdrone_debuffer.lua:209-214）；无人机由玩家的技能道具召唤，领队就是玩家实体，故 `userid` 可直接作分组键。
- 挂载对象需满足：客户端实体、有 `replica.follower`、领队最终能解析出 userid；不满足时 `OnUpdate` 首帧直接返回等待（lines 143-148）。

## 常见陷阱与限制

- **服务端不要挂：** 组件逻辑全靠 `ThePlayer` 与本地渲染（lines 114、121），专用服务器上无意义；原版在 `if not TheNet:IsDedicated()` 分支内挂载。
- **领队解析失败不淡出：** 无人机没有 follower replica 或领队无 userid／linkeditem 时永远停在未注册态（lines 143-148），表现退化为不透明。
- **无渐入渐出以外的状态：** 组件不管理无人机显隐逻辑本身，也无法被服务端行为读取；要控制无人机可见性应另走 AnimState／网络变量。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。通读 components/transparentondrones.lua 全文（177 行）；核验 wx78_shadowdrone_debuffer.lua:209-215 挂载点与影子特效创建。未运行游戏。
- 结构检查：validate_page.py / validate_source_links.py 通过（见批级自验）。
