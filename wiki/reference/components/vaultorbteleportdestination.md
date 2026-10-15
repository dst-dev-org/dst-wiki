---
title: "vaultorbteleportdestination"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "给传送底座挂地图目的地标记：生成隐藏全图图标与关闭点图标，供宝库球传送动作与地图选点识别；随宝石插拔挂卸。"
aliases: ["vaultorbteleportdestination", "宝库球传送目的地", "传送底座地图标记"]
tags: [component, ui, server-client]
size_class: simple
dst_build: "steam-buildid:26230496"
sources: ["components/vaultorbteleportdestination.lua", "prefabs/telebase.lua", "components/vaultorbteleporter.lua", "actions.lua", "screens/mapscreen.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# vaultorbteleportdestination

## 快速判断

- 职责与适用：服务端组件，挂在传送底座上被选中的球体目标；构造即生成隐藏的 `globalmapicon`（跟随本体）与子级 `globalmapiconnoproxy` 关闭点图标，打上 `vaultorbteleportdestinationtrackericon` 标签并按 `vaultorbdestination_icon.png` 显示。[构造](../../../raw/current/scripts/components/vaultorbteleportdestination.lua#L9-L20)
- 优先使用：需要"底座当前是否可作为宝库球目的地"时，以本组件存在与否为准——传送执行方 `vaultorbteleporter:Activate` 先查 `target.components.vaultorbteleportdestination`，缺失直接返回失败 NOTARGET。[门禁](../../../raw/current/scripts/components/vaultorbteleporter.lua#L67-L69)
- 重要副作用：图标实体随组件生命周期联动：组件被移除或本体消失时清掉隐藏图标；隐藏图标被移除也会反向把组件从本体上拆掉。[清理](../../../raw/current/scripts/components/vaultorbteleportdestination.lua#L4-L13)
- 修改边界：组件不实现传送本身（在 vaultorbteleporter），也不决定哪个球可用（telebase 的宝石校验）；地图端仅按标签聚合显示。[地图消费](../../../raw/current/scripts/screens/mapscreen.lua#L381-L383)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| 构造 | `Class(inst)` | 生成 `globalmapicon` 调 `TrackEntity(self.inst, "vaultorbteleportdestinationtracker", "vaultorbdestination_icon.png")`，子级 closeicon 用 `SetRestriction("vaultorbteleportdestinationtracker")` 只对持球者可见。[实现](../../../raw/current/scripts/components/vaultorbteleportdestination.lua#L9-L20) |
| `OnRemoveEntity()` | `-> 无` | 移除隐藏 `globalmapicon`；同时赋给 `OnRemoveFromEntity`，组件卸载与实体删除共用该清理。[清理](../../../raw/current/scripts/components/vaultorbteleportdestination.lua#L23-L28) |
| 自反清理 | 监听 `onremove`（图标实体） | 隐藏图标被移除时 `RemoveComponent("vaultorbteleportdestination")`，避免留下无图标的悬挂组件。[实现](../../../raw/current/scripts/components/vaultorbteleportdestination.lua#L4-L13) |

## 典型接入

### 随宝石插拔启停目的地标记（原版参照）

- 前提与选择：telebase 在 OnGemChange 里检查 `requiredgem == "vault_orb_refined"` 才挂组件，宝石不满足或拔除时移除；模组做同类结构应复用同一时机而不是常驻挂载。[挂载时机](../../../raw/current/scripts/prefabs/telebase.lua#L57-L61) [卸载时机](../../../raw/current/scripts/prefabs/telebase.lua#L28-L31)
- 生效与撤销：挂上即出地图图标；`RemoveComponent` 触发 OnRemoveFromEntity 清图标。地图端按 `vaultorbteleportdestinationtrackericon` 标签收集（actions.lua 的 MAP_VAULTORB_MUST 与 mapscreen 选点均查该标签）。[地图过滤](../../../raw/current/scripts/actions.lua#L7008)

```lua
-- 服务端：参照 telebase 的宝石插拔时机挂载／卸载
local function OnGemSocketed(inst)
    if not inst.components.vaultorbteleportdestination then
        inst:AddComponent("vaultorbteleportdestination")
    end
end

local function OnGemRemoved(inst)
    if inst.components.vaultorbteleportdestination then
        inst:RemoveComponent("vaultorbteleportdestination")
    end
end
```

- 调整与验证：自定义传送方需仿 vaultorbteleporter 先查目标组件再执行，否则会绕过"未挂组件即不可选"的原版约定。

## 常见陷阱与限制

- **把组件当传送入口：** 组件只负责标记与地图可见性；传送校验、激活逻辑都在 vaultorbteleporter 侧，直接 AddComponent 不会让底座可传送。[门禁](../../../raw/current/scripts/components/vaultorbteleporter.lua#L67-L69)
- **在客户端挂组件：** 图标是服务端生成的网络实体（globalmapicon 有 classified 联动）；客户端挂载不会得到可同步的图标。本组件仅在主机有意义。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（31 行），核对 telebase 挂卸时机、vaultorbteleporter 门禁、actions.lua 地图过滤标签与 mapscreen 聚合逻辑。未审阅 globalmapicon／globalmapiconnoproxy 实体内部实现。
- 结构检查：2026-10-28 validate_page.py 通过；validate_source_links.py --strict 锚点通过（见批级记录）。
- 游戏运行：未运行；地图选点未实测。
