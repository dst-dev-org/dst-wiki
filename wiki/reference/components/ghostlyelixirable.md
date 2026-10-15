---
title: "ghostlyelixirable"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "标记实体可被幽灵灵药强化，GIVE/APPLYELIXIR 动作按标签路由，宿主可用 overrideapplytotargetfn 覆写灵药实际作用目标。"
aliases: ["ghostlyelixirable", "幽灵灵药目标", "阿比盖尔灵药", "灵药施加目标"]
tags: [component, item, creature]
size_class: simple
dst_build: "steam-buildid:26230496"
sources: ["components/ghostlyelixirable.lua", "components/ghostlyelixir.lua", "prefabs/abigail.lua", "prefabs/abigail_flower.lua", "prefabs/player_common.lua", "actions.lua", "prefabs/ghostly_elixirs.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# ghostlyelixirable

## 快速判断

- 职责与适用：构造即加 `ghostlyelixirable` 标签的动作路由标记；Abigail、Abigail 花与所有玩家挂载。[构造](../../../raw/current/scripts/components/ghostlyelixirable.lua#L2-L6) [挂载点](../../../raw/current/scripts/prefabs/abigail.lua#L888)
- 优先使用：给灵药选择实际受益人时调 `GetApplyToTarget(doer, elixir)`；宿主设了 `overrideapplytotargetfn` 就用其返回值，否则返回本体。[实现](../../../raw/current/scripts/components/ghostlyelixirable.lua#L8-L14)
- 重要副作用：灵药组件 `Apply` 的第 15 行以本方法结果替换 target，随后才走 `doapplyelixerfn`；覆写返回 nil 时整个施加失败但不报错。[消费](../../../raw/current/scripts/components/ghostlyelixir.lua#L15-L22)
- 修改边界：组件移除时同步撤标签；无存读与网络变量，"花选中谁"的规则完全在宿主回调里。[移除](../../../raw/current/scripts/components/ghostlyelixirable.lua#L16-L18)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `overrideapplytotargetfn` | `fn(inst, doer, elixir) -> entity|nil` | 可选字段；abigail_flower 用它把灵药转给 `ghostlybond.ghost`。返回值直接作为施加目标，nil 会令 Apply 失败。[宿主覆写](../../../raw/current/scripts/prefabs/abigail_flower.lua#L212-L214) |
| `GetApplyToTarget(doer, elixir)` | `-> entity` | 无覆写时返回 `self.inst`；elixir 参数原样透传给覆写回调。[实现](../../../raw/current/scripts/components/ghostlyelixirable.lua#L8-L14) |

## 典型接入

### 花灵药转投绑定幽灵（原版覆写参照）

- 前提与选择：abigail_flower 在挂组件后直接写 `overrideapplytotargetfn = GetElixirTarget`；回调里 `doer.components.ghostlybond ~= nil` 才返回其 ghost，否则 nil。[宿主实现](../../../raw/current/scripts/prefabs/abigail_flower.lua#L212-L214) [挂载](../../../raw/current/scripts/prefabs/abigail_flower.lua#L360-L361)
- 生效与撤销：GIVE 动作先按 `ghostlyelixirable`+`ghostlyelixir` 双标签改文案并路由到 `ghostlyelixir:Apply`；Apply 还处理 `elixir_drinker` 标签（转给持有者）后再取目标。[GIVE 分支](../../../raw/current/scripts/actions.lua#L2300-L2301) [Apply](../../../raw/current/scripts/components/ghostlyelixir.lua#L8-L15)
- 原版依据：药水侧在 ghostly_elixirs.lua 挂 ghostlyelixir 组件并设 `doapplyelixerfn`；玩家侧（avenging ghost）与 Abigail 侧同为直接挂载。[药水](../../../raw/current/scripts/prefabs/ghostly_elixirs.lua#L436-L437) [玩家](../../../raw/current/scripts/prefabs/player_common.lua#L2527)

```lua
-- 服务端：自定义灵药目标（按 abigail_flower 模式覆写）
local function GetElixirTarget(inst, doer, elixir)
    return (doer ~= nil and doer.components.ghostlybond ~= nil)
        and doer.components.ghostlybond.ghost
        or nil
end

inst:AddComponent("ghostlyelixirable")
inst.components.ghostlyelixirable.overrideapplytotargetfn = GetElixirTarget
```

- 调整与验证：覆写回调的返回值必须是有 `ghostlyelixirable` 组件语义的目标实体（ghostlyelixir.Apply 不再二次校验）；nil 即本次施加失败。

## 常见陷阱与限制

- **以为标签足以完成施加：** APPLYELIXIR 用 `find_elixirable_fn` 在背包/目标里找组件；GIVE 侧还要求物品有 ghostlyelixir 组件或标签。只加本组件没有药水侧组件不会发生任何效果。[查找](../../../raw/current/scripts/actions.lua#L6208-L6210) [GIVE 条件](../../../raw/current/scripts/actions.lua#L2300-L2301)
- **覆写回调返回无组件实体：** ghostlyelixir:Apply 拿到覆写结果后直接调 `doapplyelixerfn(self.inst, doer, target)`，不再检查目标是否可用药；目标不合适时效果错位或空转。[消费](../../../raw/current/scripts/components/ghostlyelixir.lua#L15-L22)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（21 行），核对三个挂载点、GIVE/APPLYELIXIR 路由、ghostlyelixir:Apply 的目标替换与药水挂载。未审阅各灵药 doapplyelixerfn 的具体数值效果。
- 结构检查：2026-10-28 validate_page.py 通过（ghostlyelixirable.md）；validate_source_links.py --strict 本页全部源码锚点通过；Lua 示例经 parser_probe --syntax-only 解析通过。
- 游戏运行：未运行；灵药施加与覆写回调未实测。

## 相关链接

- [[reference/components/ghostlyelixir]]：幽灵灵药 Apply 与受益人解析。
