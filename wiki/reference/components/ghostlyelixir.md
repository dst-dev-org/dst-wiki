---
title: "ghostlyelixir"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "幽灵灵药的应用执行：按 ghostlyelixirable 解析实际受益人、回调 doapplyelixerfn 出效果并消耗药水；效果全在宿主注入的回调里。"
aliases: ["ghostlyelixir", "幽灵灵药", "阿比盖尔药水"]
tags: [component, item, actions]
size_class: simple
dst_build: "steam-buildid:26230496"
sources: ["components/ghostlyelixir.lua", "prefabs/ghostly_elixirs.lua", "actions.lua", "componentactions.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# ghostlyelixir

## 快速判断

- 职责与适用：挂在各幽灵灵药物品上；构造加 `ghostlyelixir` 标签供动作路由，`Apply(doer, target)` 负责目标解析与效果分发。[构造](../../../raw/current/scripts/components/ghostlyelixir.lua#L2-L6)
- 优先使用：具体药效写在 `doapplyelixerfn`（源码拼写如此，注意无 t）；目标解析委托目标的 `ghostlyelixirable:GetApplyToTarget`，宿主可用 overrideapplytotargetfn 覆写（目标侧机制见文末 ghostlyelixirable 页）。[Apply](../../../raw/current/scripts/components/ghostlyelixir.lua#L8-L29)
- 重要副作用：`Apply` 成功即消耗药水（stackable 取一件删除，否则整瓶 Remove）；目标带 `elixir_drinker` 标签时先转给其持有者。[目标改写](../../../raw/current/scripts/components/ghostlyelixir.lua#L9-L14) [消耗](../../../raw/current/scripts/components/ghostlyelixir.lua#L19-L25)
- 修改边界：组件不校验目标合法性——GetApplyToTarget 返回 nil 或未注入 doapplyelixerfn 都直接 false；GIVE／APPLYELIXIR 动作的两条路由都先按标签过滤。[路由](../../../raw/current/scripts/actions.lua#L2300-L2301)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `Apply(doer, target)` | `-> success, reason` | elixir_drinker 转持有者 → `ghostlyelixirable:GetApplyToTarget(doer, inst)` 替换目标 → 调 `doapplyelixerfn(inst, doer, target)`，成功才消耗药水。[实现](../../../raw/current/scripts/components/ghostlyelixir.lua#L8-L29) |
| `doapplyelixerfn` | `fn(inst, doer, target) -> success, reason?` | 宿主注入的药效函数；原版在 ghostly_elixirs 的 potion_tunings 里为每种灵药分别定义。[注入](../../../raw/current/scripts/prefabs/ghostly_elixirs.lua#L436-L437) |
| `ghostlyelixir` 标签 | 构造加、移除撤 | GIVE 候选按目标 `ghostlyelixirable` + 物品本标签组合改文案并路由。[标签](../../../raw/current/scripts/components/ghostlyelixir.lua#L2-L6) [路由条件](../../../raw/current/scripts/actions.lua#L2227) |

## 典型接入

### 自定义幽灵灵药（原版装配参照）

- 前提与选择：药水 prefab 挂组件并赋 doapplyelixerfn；饮用目标需有 `ghostlyelixirable` 组件（Abigail、Abigail 花与所有玩家原版挂载）。[装配](../../../raw/current/scripts/prefabs/ghostly_elixirs.lua#L436-L437)
- 生效与撤销：两条动作路由——对目标直接 GIVE（actions.lua#L2300-L2301），或对带 `elixir_drinker` 的物品 APPLYELIXIR（actions.lua#L6208 起的查找函数）。[APPLYELIXIR](../../../raw/current/scripts/actions.lua#L6208) [第二调用点](../../../raw/current/scripts/actions.lua#L6223)

```lua
-- 服务端：自定义灵药（仿 ghostly_elixirs 装配）
inst:AddComponent("ghostlyelixir")
inst.components.ghostlyelixir.doapplyelixerfn = function(inst, doer, target)
    target.components.health:DoDelta(20)
    return true
end
```

- 调整与验证：效果回调返回 false 时不消耗药水；候选入口在 componentactions（目标带 elixir_drinker 标签时给 APPLYELIXIR）。[候选](../../../raw/current/scripts/componentactions.lua#L1328-L1331)

## 常见陷阱与限制

- **直接给无 ghostlyelixirable 的目标施加：** `Apply` 第 15 行无条件访问 `target.components.ghostlyelixirable`，目标缺组件会直接报错；路由层已按标签过滤，绕过动作直接调用要先自检。[访问点](../../../raw/current/scripts/components/ghostlyelixir.lua#L15)
- **把药效写在组件里：** 组件不含任何效果逻辑；想复用现有药的数值应引用 potion_tunings 的 doapplyelixerfn 而不是重写。[注入点](../../../raw/current/scripts/prefabs/ghostly_elixirs.lua#L436-L437)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（35 行），核对 ghostly_elixirs 装配、GIVE／APPLYELIXIR 两条路由、componentactions 候选；目标解析语义与 ghostlyelixirable 页互证。未逐个审阅各灵药 doapplyelixerfn 的数值效果。
- 结构检查：2026-10-28 validate_page.py 通过；validate_source_links.py --strict 锚点通过（见批级记录）。
- 游戏运行：未运行。

相关：[[reference/components/ghostlyelixirable]]
