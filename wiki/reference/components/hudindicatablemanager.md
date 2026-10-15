---
title: "hudindicatablemanager"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "非专用端世界上的 HUD 可指示物登记表：hudindicatable 组件在世界上注册/注销自身，管理器按 GUID 维护 items 集合。"
aliases: ["hudindicatablemanager", "HUD指示物登记表", "屏幕指示管理器"]
tags: [component, ui]
size_class: simple
dst_build: "steam-buildid:26230496"
sources: ["components/hudindicatablemanager.lua", "components/hudindicatable.lua", "prefabs/world.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# hudindicatablemanager

## 快速判断

- 职责与适用：世界级注册表。`world.lua:560` 在非专用端给 TheWorld 挂载；[[reference/components/hudindicatable]] 组件构造 0 秒后把自己注册进来，移除时注销。[挂载](../../../raw/current/scripts/prefabs/world.lua#L557-L560) [注册](../../../raw/current/scripts/components/hudindicatable.lua#L25-L28)
- 优先使用：不要直接操作本管理器——给实体挂 `hudindicatable` 组件即可自动注册/注销；决定"是否显示指示"的应答逻辑也在 hudindicatable 的 `ShouldTrack`。[宿主组件](../../../raw/current/scripts/components/hudindicatable.lua#L14-L16)
- 重要副作用：仅维护 `items[GUID]` 集合，无任何行为推送；注销时由 hudindicatable 自己另推 `unregister_hudindicatable` 世界事件供 watcher 响应（与本管理器无关）。[事件推送方](../../../raw/current/scripts/components/hudindicatable.lua#L18-L23)
- 修改边界：仅非专用端存在（构造断言 `not TheNet:IsDedicated()`）；无存读内容（OnSave/OnLoad 空壳），`GetDebugString` 恒返回 nil（`if true` 硬编码）。[断言](../../../raw/current/scripts/components/hudindicatablemanager.lua#L7)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `items` | table，键为 `item.GUID` | 注册表本体；当前脚本库内只有写入（注册/注销），没有读取者。[注册表](../../../raw/current/scripts/components/hudindicatablemanager.lua#L23) |
| `RegisterItem(item)` | item: entity | 按 GUID 幂等登记。[实现](../../../raw/current/scripts/components/hudindicatablemanager.lua#L44-L48) |
| `UnRegisterItem(item)` | item: entity 或 nil | 按 GUID 注销，item 为 nil 时静默跳过。[实现](../../../raw/current/scripts/components/hudindicatablemanager.lua#L50-L56) |
| `unregister_hudindicatable` | 世界事件（相关但非本组件推送） | hudindicatable 注销时推给 TheWorld；`hudindicatorwatcher.lua:29` 与 `playertargetindicator.lua:25` 监听以停止追踪。[监听方](../../../raw/current/scripts/components/hudindicatorwatcher.lua#L29) |

## 典型接入

### 让实体获得 HUD 追踪（原版参照）

- 前提与选择：实际接入入口是 [[reference/components/hudindicatable]]：月潜者（`moonstorm_static.lua:508-509`）、玩家（`player_common.lua:2587-2588`）、Wagstaff（`wagstaff_npc.lua:469-470`）挂组件并用 `SetShouldTrackFunction` 控制可追踪条件；管理器无需手动触碰。[示例](../../../raw/current/scripts/prefabs/moonstorm_static.lua#L508-L509)
- 生效与撤销：挂组件 0 秒后注册、组件移除（含实体删除）时注销并推事件；世界没有该管理器时（专用端）宿主组件的注册路径静默跳过。

```lua
-- 服务端/客户端按宿主组件约定；此处为 hudindicatable 的驱动路径
inst:AddComponent("hudindicatable")
inst.components.hudindicatable:SetShouldTrackFunction(function(inst, viewer)
    return true -- 按需收紧
end)
-- 之后 hudindicatable 会自动经 TheWorld.components.hudindicatablemanager 注册
```

- 调整与验证：追踪判定最终消费者是 `hudindicatorwatcher`/`playertargetindicator` 的 `ShouldTrack` 查询（`hudindicatorwatcher.lua:56`），管理器的 items 表不参与判定。

## 常见陷阱与限制

- **把 items 当运行时数据源：** 管理器当前是"只写不读"的登记表，没有任何系统遍历 `items`；需要世界级列表时自行维护或读 hudindicatable 标签体系。[只写现状](../../../raw/current/scripts/components/hudindicatablemanager.lua#L44-L56)
- **在专用端访问：** 构造断言限定非专用端；专用端 `TheWorld.components.hudindicatablemanager` 恒为 nil，注册代码须判空（hudindicatable 已内置判空）。[断言](../../../raw/current/scripts/components/hudindicatablemanager.lua#L7)
- **混淆事件来源：** `unregister_hudindicatable` 由 hudindicatable 推送而非管理器；只操作管理器不会触发任何事件。[推送点](../../../raw/current/scripts/components/hudindicatable.lua#L21)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（82 行），核对 world.lua 挂载、hudindicatable 注册/注销路径、hudindicatorwatcher 与 playertargetindicator 的事件监听及 ShouldTrack 查询；全库 grep 确认 `RegisterItem`/`UnRegisterItem` 无第三处调用、`items` 无读取者。
- 结构检查：2026-10-28 validate_page.py 通过；validate_source_links.py --strict 锚点通过（见批级记录）。
- 游戏运行：未运行。

## 相关链接

- [[reference/components/playertargetindicator]]：屏外玩家 HUD 标记组件。
