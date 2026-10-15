---
title: "walkingplankuser"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "记录玩家当前乘坐的船踏板：维护 current_plank 与其 onremove 监听，Dismount 时让踏板结束 mounting；玩家侧配套组件。"
aliases: ["walkingplankuser", "踏板使用者", "登板状态"]
tags: [component, callback, lifecycle]
size_class: simple
dst_build: "steam-buildid:26230496"
sources: ["components/walkingplankuser.lua", "components/walkingplank.lua", "prefabs/player_common.lua", "stategraphs/SGwilson.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# walkingplankuser

## 快速判断

- 职责与适用：挂在玩家实体上，与踏板上的 `walkingplank` 组件配对——plank 侧记住 doer，本组件记住 plank；双向引用靠 onremove 监听自动清空。[组件全文](../../../raw/current/scripts/components/walkingplankuser.lua#L8-L19)
- 优先使用：不要手动赋值 `current_plank`，一律走 `SetCurrentPlank`／`Dismount`；原版调用点在 `walkingplank:MountPlank` 与 `AbandonShip`。[MountPlank](../../../raw/current/scripts/components/walkingplank.lua#L21-L31)
- 重要副作用：`SetCurrentPlank` 每次先取消旧的 onremove 监听再对新踏板注册；踏板被移除时自动把 `current_plank` 置 nil，避免悬挂引用。[监听管理](../../../raw/current/scripts/components/walkingplankuser.lua#L8-L19)
- 修改边界：`Dismount` 会调踏板的 `walkingplank:StopMounting()`（推 stop_mounting 事件）并清监听；对 nil plank 调用安全。[Dismount](../../../raw/current/scripts/components/walkingplankuser.lua#L21-L31)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SetCurrentPlank(plank)` | `plank: entity|nil -> 无` | 取消旧监听；plank 非 nil 时在其上监听 `onremove` 清引用，最后赋 `current_plank`。[实现](../../../raw/current/scripts/components/walkingplankuser.lua#L8-L19) |
| `Dismount()` | `-> 无` | 有 plank 时调 `current_plank.components.walkingplank:StopMounting()`，随后清引用与监听；要求踏板带 walkingplank 组件（原版宿主均满足）。[实现](../../../raw/current/scripts/components/walkingplankuser.lua#L21-L31) |
| `current_plank` | `entity|nil` | 当前乘坐的踏板实体；仅供读取判断，写入请走 SetCurrentPlank。[字段](../../../raw/current/scripts/components/walkingplankuser.lua#L4-L5) |

## 典型接入

### 玩家与踏板的配对链路（原版参照）

- 前提与选择：组件在 player_common 的 mastersim 分支为所有玩家挂载；踏板实体带 `walkingplank` 组件并维护自己的 doer 字段。[玩家挂载](../../../raw/current/scripts/prefabs/player_common.lua#L2843)
- 生效与撤销：登板流程 `MountPlank` → 传送玩家到踏板 → `SetCurrentPlank`；弃船 `AbandonShip` → `Dismount` → 踏板进 start_abandoning；SGwilson 的踏板状态退出时（非弃船路径）也会强制 `Dismount` 兜底。[SG 兜底](../../../raw/current/scripts/stategraphs/SGwilson.lua#L12921-L12922)

```lua
-- 服务端：一般由 walkingplank:MountPlank 调用，勿手动乱序操作
doer.components.walkingplankuser:SetCurrentPlank(plank_inst)
-- 离开踏板（内部触发 plank 侧 StopMounting）
doer.components.walkingplankuser:Dismount()
```

- 调整与验证：自定义踏板实体必须挂 `walkingplank` 组件，否则 Dismount 内部访问 `components.walkingplank:StopMounting` 会报错；`StopMounting` 只清 doer 并推 stop_mounting，不移动玩家。[StopMounting](../../../raw/current/scripts/components/walkingplank.lua#L34-L37)

## 常见陷阱与限制

- **绕过 SetCurrentPlank 直接写 current_plank：** 会丢失 onremove 监听，踏板删除后玩家持悬挂引用，后续 Dismount 访问已删除实体报错。[监听管理](../../../raw/current/scripts/components/walkingplankuser.lua#L14-L18)
- **以为组件负责登板移动：** 传送、动画与事件顺序都在 walkingplank 侧和状态图里；本组件只维护配对关系，不阻止玩家中途瞬移。[MountPlank](../../../raw/current/scripts/components/walkingplank.lua#L21-L31)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（33 行），核对 walkingplank 的 MountPlank／StopMounting／AbandonShip、player_common 挂载与 SGwilson 状态退出兜底。
- 结构检查：2026-10-28 validate_page.py 通过；validate_source_links.py --strict 锚点通过（见批级记录）。
- 游戏运行：未运行。

相关：[[reference/components/walkableplatform]]
