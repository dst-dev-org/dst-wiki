---
title: "mapspotrevealer"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "可读物品（漂流瓶藏宝图等）的地图点位揭示器：经 player_classified 向客户端推图钉并在延迟后揭示区域。"
aliases: ["地图揭示", "藏宝图揭示", "mapspotrevealer"]
tags: [component, network-sync]
dst_build: "steam-buildid:26230496"
sources: ["components/mapspotrevealer.lua", "prefabs/messagebottle.lua", "prefabs/stash_map.lua", "actions.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# mapspotrevealer

## 快速判断

- 职责与适用：挂在可阅读物品上，READ 动作时揭示目标点位；doer 必须是带 player_classified 的真玩家，否则返回 NO_MAP（[判断](../../../raw/current/scripts/components/mapspotrevealer.lua#L52-L64)）。
- 优先使用：藏宝类物品给 SetGetTargetFn + SetPreRevealFn 两件套，如漂流瓶（[接线](../../../raw/current/scripts/prefabs/messagebottle.lua#L177-L179)）。
- 重要副作用：揭示前推 `on_reveal_map_spot_pre`、成功后推 `on_reveal_map_spot_pst` 实体事件（[pre](../../../raw/current/scripts/components/mapspotrevealer.lua#L50)、[pst](../../../raw/current/scripts/components/mapspotrevealer.lua#L66)）；客户端先收到图钉打开大地图，4 帧后再 RevealArea。
- 修改边界：仅服务端执行；构造时给 prefab 加 `mapspotrevealer` tag 供动作系统识别（[tag](../../../raw/current/scripts/components/mapspotrevealer.lua#L5)），移除组件时摘除。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SetGetTargetFn(fn)` | `fn(inst, doer) -> targetpos: table{x,y,z} 或 nil, reason` | 必设；无函数或返回无坐标时 RevealMap 返回 `false, "NO_TARGET"`。[注册](../../../raw/current/scripts/components/mapspotrevealer.lua#L17-L19) [判定](../../../raw/current/scripts/components/mapspotrevealer.lua#L34-L48) |
| `SetPreRevealFn(fn)` | `fn(inst, doer) -> boolean` | 返回 `false` 时本次不揭示但 RevealMap 仍返回 `true`（动作已消费），用于"读出纸条"这类分支。[注册](../../../raw/current/scripts/components/mapspotrevealer.lua#L21-L23) [分支](../../../raw/current/scripts/components/mapspotrevealer.lua#L26-L32) |
| `RevealMap(doer)` | `doer: player` → `success:boolean, reason:string 或 nil` | 主入口：写 `player_classified.revealmapspot_worldx/z` 并 push `revealmapspotevent`，随后 `MapExplorer:RevealArea`。[实现](../../../raw/current/scripts/components/mapspotrevealer.lua#L25-L69) |
| `open_map_on_reveal` | `boolean`，默认 `true` | false 时不推图钉事件、只延迟揭示区域。[字段](../../../raw/current/scripts/components/mapspotrevealer.lua#L10) [分支](../../../raw/current/scripts/components/mapspotrevealer.lua#L53-L57) |
| `postreveal` | `fn(inst) 或 nil`（宿主自设，组件不声明） | actions.lua 在 RevealMap 之后无条件调用，成功与否都会执行。[动作集成](../../../raw/current/scripts/actions.lua#L3273-L3277) |

## 典型接入

### 藏宝图物品

- 前提与选择：物品 prefab 加 `mapspotrevealer` tag（pristine）并装组件；gettargetfn 决定揭示点（可随机），prerevealfn 做条件拦截。
- 生效与撤销：READ 成功后客户端自动弹图钉开地图；一次性物品用 `on_reveal_map_spot_pst` 事件把自己变空瓶（[漂流瓶用法](../../../raw/current/scripts/prefabs/messagebottle.lua#L181)）。
- 原版依据：漂流瓶（[messagebottle](../../../raw/current/scripts/prefabs/messagebottle.lua#L177-L179)）与埋藏宝藏图（[stash_map](../../../raw/current/scripts/prefabs/stash_map.lua#L49-L51)）。

```lua
-- 物品 master_postinit 内
inst:AddComponent("mapspotrevealer")
inst.components.mapspotrevealer:SetGetTargetFn(function(inst, doer)
    local x = math.random(-1200, 1200)
    local z = math.random(-1200, 1200)
    return { x = x, y = 0, z = z }
end)
inst.components.mapspotrevealer:SetPreRevealFn(function(inst, doer)
    return doer.components.mapexplorer ~= nil
end)
```

- 调整与验证：targetpos 传 `{x, y, z}` 表即可（组件只读 x/z 揭示、三个值都传给 RevealArea）；返回 nil 时第二个值作为 reason 透传给动作系统。

## 常见陷阱与限制

- prerevealfn 返回 false 表示"不揭示但动作成功"，与"揭示失败"（返回 false + reason）不同；做一次性物品时注意两者都会让 postreveal 执行（[动作](../../../raw/current/scripts/actions.lua#L3273-L3277)）。
- dedicated 服务器控制台给假玩家（无 player_classified）使用会拿到 `false, "NO_MAP"`，不报错（[实现](../../../raw/current/scripts/components/mapspotrevealer.lua#L63)）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，覆盖全部接口与 messagebottle／stash_map／actions 消费链；未运行游戏。
- 结构检查：validate_page.py、validate_source_links.py 与 Lua 语法探针通过。

## 相关链接

- [[reference/components/maprevealer]]：周期揭示周边地图的揭示器。
