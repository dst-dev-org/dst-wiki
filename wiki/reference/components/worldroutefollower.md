---
title: "worldroutefollower"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "让实体沿世界 worldroutes 注册的环形路径点巡游：到点推进索引、卡住自动重定位或传送、休眠改走虚拟步行计时。"
aliases: ["worldroutefollower", "世界路线跟随", "环形路线巡游", "FollowRoute"]
tags: [component, ai, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/worldroutefollower.lua", "components/worldroutes.lua", "prefabs/wanderingtrader.lua", "prefabs/world.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# worldroutefollower

## 快速判断

- 职责与适用：实体级组件，让拥有 locomotor 的实体沿 `TheWorld.components.worldroutes` 里的点列循环行走（`FollowRoute(routename)`），到点推进、卡住传送兜底；不负责寻路细节与路线生成。[安装示例](../../../raw/current/scripts/prefabs/wanderingtrader.lua#L554-L559) [路线源](../../../raw/current/scripts/components/worldroutefollower.lua#L240-L241)
- 优先使用：换路线／换起点用 `FollowRoute(name, indexoverride)`；暂停用 `SetPaused(true, reason)`（SourceModifierList，多来源互不覆盖）；到达回调 `SetOnArrivedFn`。[跟随](../../../raw/current/scripts/components/worldroutefollower.lua#L235-L272) [暂停](../../../raw/current/scripts/components/worldroutefollower.lua#L224-L233)
- 重要副作用：卡住会 `Physics:Teleport`（先 `FindValidPointNear` 找合法点，preteleportfn 返回 true 时由调用者执行传送）；连续 3 次找不到传送点会强制跳过当前点；实体休眠后改为定时任务"虚拟步行"再传送。[传送](../../../raw/current/scripts/components/worldroutefollower.lua#L145-L168) [跳点](../../../raw/current/scripts/components/worldroutefollower.lua#L318-L328) [休眠](../../../raw/current/scripts/components/worldroutefollower.lua#L336-L343)
- 修改边界：需要 `inst.components.locomotor`（SetRouteIndex 读速度）；有 stuckdetection 组件时用它判卡，否则用内部时间估算；`OnRemoveFromEntity` 只取消传送任务，路线状态留在实体上供存档。[时间预算](../../../raw/current/scripts/components/worldroutefollower.lua#L207-L222) [移除](../../../raw/current/scripts/components/worldroutefollower.lua#L27-L32)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `FollowRoute(routename, routeindexoverride?)` | → bool | 从 `worldroutes:GetRoute` 取点列；同名同索引直接返回 true。取不到路线时停止更新并清空状态返回 false；无 override 时选距实体最近的点起步。[跟随](../../../raw/current/scripts/components/worldroutefollower.lua#L235-L272) |
| `GetRoute()`／`GetRouteDestination()` | → 路线表 / Vector3 | 当前点列与目标点；`routeindex` 越界后回到 1 形成环。[读取](../../../raw/current/scripts/components/worldroutefollower.lua#L58-L68) [推进](../../../raw/current/scripts/components/worldroutefollower.lua#L195-L201) |
| `SetRouteIndex(routeindex)` | 内部/可调 | 计算本段时间预算 `routetimetoarrivemax = 段长×2/速度`（TIME_FORGIVENESS_FACTOR=2）并启动更新；速度取 locomotor 当前 run/walk。[预算](../../../raw/current/scripts/components/worldroutefollower.lua#L207-L222) [因子](../../../raw/current/scripts/components/worldroutefollower.lua#L3 |
| `SetPaused(paused, reason)` | SourceModifierList | 以来源+理由登记暂停；解除时若还有 stuckdetection 会 Reset，避免旧卡记录立即触发传送。[暂停](../../../raw/current/scripts/components/worldroutefollower.lua#L224-L233) |
| `SetCloseEnoughDist(dist)`／`SetVirtualWalkingSpeedMult(mult)` | number | 到点判定半径（默认 4，另加实体物理半径）；虚拟步行速度倍率（默认 1，0 视作不加速），原版商人用 4。[可调项](../../../raw/current/scripts/components/worldroutefollower.lua#L10-L11) [商人调参](../../../raw/current/scripts/prefabs/wanderingtrader.lua#L556-L557) |
| `SetOnArrivedFn/SetIsValidPointForRouteFn/SetPreTeleportFn/SetPostTeleportFn(fn)` | 回调 | 到点推进回调；自定义点位合法性（先于碰撞检查）；传送前后钩子，pre 返回 true 表示调用者自行执行 `TeleportToDestination`。[回调注册](../../../raw/current/scripts/components/worldroutefollower.lua#L42-L56) [pre 约定](../../../raw/current/scripts/components/worldroutefollower.lua#L158-L165) |
| `TryToTeleportToDestination()` | → bool | 找合法点（NoEnts 碰撞 + 自定义校验，环状 4→32 半径搜索），失败 false；成功设置 teleportdest 并走 pre→TeleportToDestination→post 流程。[尝试](../../../raw/current/scripts/components/worldroutefollower.lua#L145-L168) |
| `TryToStartVirtualWalk()` | → bool | 休眠兜底：按剩余时间预算（除以虚拟倍率）`DoTaskInTime` 后传送；已有任务则不重复。[虚拟步行](../../../raw/current/scripts/components/worldroutefollower.lua#L274-L285) |
| `OnSave()`／`OnLoad(data)` | routename + routeindex | 沿途状态不存，重载后按存档路线与索引 `FollowRoute` 恢复（路线必须仍存在于 worldroutes）。[存读](../../../raw/current/scripts/components/worldroutefollower.lua#L357-L374) |

### 卡住判定与到点推进

`OnUpdate` 只在未暂停时推进：目标点距离小于（物理半径+closeenoughdist）则推进索引并触发 onarrivedfn；在移动状态累计 `routetimeelapsed`，超出预算或 stuckdetection 判卡才尝试传送。原地不动（无 moving 标签）会 Reset stuckdetection，不会误判。[推进](../../../raw/current/scripts/components/worldroutefollower.lua#L287-L334) [stuckdetection](../../../raw/current/scripts/components/stuckdetection.lua#L1-L9)

## 典型接入

### 让自定义商人／NPC 沿固定环线巡游

- 前提与选择：宿主需有 locomotor（走路表现）与世界端注册过的路线（`worldroutes:SetRoute(name, points)`，点为 Vector3(x,0,z)；原版路线由 worldgen 写入或代码注册，存档随 worldroutes 持久化）。商人额外注册 stuckdetection、teleportedoverride 与四个回调来处理停摊与刷新。[worldroutes](../../../raw/current/scripts/components/worldroutes.lua#L12-L18) [商人组合](../../../raw/current/scripts/prefabs/wanderingtrader.lua#L548-L559)
- 生效与撤销：`FollowRoute` 成功即开始移动；停止巡游用 `SetPaused(true,"reason")`（商人停摊/到点等待都用它），或清空 self.route 后手动 StopUpdatingComponent。路线数据不在本组件存档，改 worldroutes 后旧存档索引可能越界（FollowRoute 会按 override 或最近点重设）。[商人暂停](../../../raw/current/scripts/prefabs/wanderingtrader.lua#L95-L117)
- 原版依据：商人本体在贸易/到点等待时对组件 `SetPaused`，脑层用 `ShouldIterate()` 与 `GetRouteDestination()` 配合 wander 行为，暂停与巡游在两层协同。[本体暂停](../../../raw/current/scripts/prefabs/wanderingtrader.lua#L95-L117) [脑消费](../../../raw/current/scripts/brains/wanderingtraderbrain.lua#L21-L25)

```lua
-- 服务器端：给自定义 NPC 挂世界环线巡游（worldroutes 已由 world 端提供）
local function StartPatrol(inst)
    if inst.components.worldroutefollower == nil then
        inst:AddComponent("worldroutefollower")
    end
    local follower = inst.components.worldroutefollower
    follower:SetOnArrivedFn(function(ent) print("reached point") end)
    follower:FollowRoute("wanderingtrader") -- 原版路线名；自定义路线先用 worldroutes:SetRoute 注册
end
```

- 调整与验证：路线名必须与 worldroutes 登记一致，否则 FollowRoute 返回 false 且组件停更新；`SetRouteIndex` 内部立即 StartUpdatingComponent，不需手动启动。[失败分支](../../../raw/current/scripts/components/worldroutefollower.lua#L242-L252)

## 常见陷阱与限制

- **以为没装 stuckdetection 就不会传送：** 内部有 `routetimetoarrivemax` 估算兜底，超时会直接尝试传送；该预算是"走路时间×2"，速度为 0 或极慢的实体会把预算拉得极长。[兜底](../../../raw/current/scripts/components/worldroutefollower.lua#L316-L317)
- **休眠期间不会真的走：** OnEntitySleep 停更新并启动虚拟步行任务，醒来时若任务已把实体传送，位置直接跳变（postteleportfn 之后才回到正常循环）；读位置做逻辑要考虑该跳变。[休眠切换](../../../raw/current/scripts/components/worldroutefollower.lua#L336-L355)
- **pause 来源要成对清理：** `SetPaused(false, reason)` 只移除同 reason 的来源；商人用 "trading" 与 "arrivedwait" 两个独立理由，漏解一个会停在暂停态。[来源](../../../raw/current/scripts/prefabs/wanderingtrader.lua#L95-L117)
- **preteleportfn 返回 true 的契约：** 组件不再自动传送，由回调持有者决定何时调 `TeleportToDestination`；忘记调用会卡在 teleportdest 待定态（debug 串显示 "via externally managed"）。[契约](../../../raw/current/scripts/components/worldroutefollower.lua#L157-L167)
- **到点回调只在推进成功时触发：** `IterateRoute(false)` 在未达点时返回 false 且不触发 onarrivedfn；强行 force=true 的推进（跳点路径）也会触发回调，可用于检测跳点。[推进语义](../../../raw/current/scripts/components/worldroutefollower.lua#L183-L205)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取 worldroutefollower.lua 与 worldroutes.lua，核验跟随／推进／暂停／传送／虚拟步行全流程、回调契约、存读与 OnRemoveFromEntity；核对 wanderingtrader 安装与暂停消费、world 端 worldroutes 安装。未逐行审校 brains/wanderingtraderbrain 的全部状态。
- 结构检查：本页通过 validate_page.py 与 validate_source_links.py（严格）；Lua 示例经 parser_probe 语法检查。
- 游戏运行：未运行；自定义路线注册与休眠跳变表现未实测。

## 相关链接

- [[reference/components/worldroutes]]：路线点列的注册与存档。
- [[reference/components/stuckdetection]]：卡住检测来源之一。
- [[reference/components/locomotor]]：组件读取的移动速度接口。
- [[reference/components/wanderingtraderspawner]]：原版巡游商人的生成管理。
