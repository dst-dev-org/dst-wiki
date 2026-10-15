---
title: "prophider"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "把实体移出场景并在原地生成替身道具的藏身组件：按周期尝试现身，找到目标时回调，休眠/唤醒下保持计时。"
aliases: ["藏身道具", "实体隐藏替身", "prophider"]
tags: [component, lifecycle, callback]
dst_build: "steam-buildid:26230496"
size_class: simple
sources: ["components/prophider.lua", "prefabs/spat.lua", "prefabs/warg.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# prophider

## 快速判断

- 职责与适用：生物"钻进道具躲藏"场景；`HideWithProp` 把实体移出场景并生成替身 prop，按 `hideupdate_duration±variance`（默认 6±1 秒）周期尝试现身（components/prophider.lua:4-5、:78-112）。原版消费者为 spat 与 warg。
- 优先使用：有"找回者"目标的玩法→`SetWillUnhideFn` 返回目标实体即现身并回调 `onunhidefn(inst, target)`；纯计时躲藏→只设 `SetOnVisibleFn`。
- 重要副作用：prop 强制 `persists = false`（:105）；现身时对 prop 推 `"propreveal"` 事件（:129）。
- 修改边界：服务器组件；实体休眠即取消计时（:135），唤醒时若仍处 hiding 则恢复计时（:137-141）。

## 关键接口

| 接口/字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `HideWithProp` | `(duration: number\|nil, counter: number\|nil) -> nil` | 已 hiding 时忽略；counter 默认 10；移出场景、清旧 prop、经 `propcreationfn` 造新 prop、回调 `onhidefn`。:78-112 |
| `ShowFromProp` | `() -> nil` | 回到场景、清 counter、回调 `onvisiblefn`、对 prop 推 `propreveal` 后置空。:114-132 |
| `SetWillUnhideFn` | `(fn: (inst) -> ent\|nil)` | 周期触发：返回非 nil 目标→立即现身并 `onunhidefn(inst, target)`；返回 nil→counter-1 重排，counter≤1 时仅现身、不调 `onunhidefn`。:48-76 |
| `SetPropCreationFn` | `(fn: (inst) -> ent\|nil)` | 每次藏身调用；返回 nil 则无替身道具。:101-107 |
| `SetOnHideFn` / `SetOnVisibleFn` / `SetOnUnhideFn` | `(fn)` | 藏身成功／回到场景／找到目标现身三类回调。:17-35 |
| `GenerateHideTime` | `() -> number` | `duration + variance*(rand*2-1)`。:37-39 |

## 典型接入

### 可躲藏生物（服务器 master postinit）

- 前提与选择：参照 spat（prefabs/spat.lua:283-288）与 warg（prefabs/warg.lua:1022-1026）的接入；五个回调按需选配，prop 建议选低价值不阻挡 prefab。
- 生效与撤销：藏身期间本实体不在场景（不可见不可点）；现身或组件移除即恢复。

```lua
AddPrefabPostInit("my_hider", function(inst)
    if not TheWorld.ismastersim then
        return
    end
    inst:AddComponent("prophider")
    inst.components.prophider:SetPropCreationFn(function(inst_)
        local prop = SpawnPrefab("twigs")
        prop.Transform:SetPosition(inst_.Transform:GetWorldPosition())
        return prop
    end)
    inst.components.prophider:SetOnVisibleFn(function(inst_)
        -- 现身：恢复动画/可交互状态
    end)
end)
```

- 调整与验证：`HideWithProp()` 不传 duration 时用默认 6±1 秒节奏；立即现身调 `ShowFromProp()`。

## 常见陷阱与限制

- counter 用尽自然现身时**不会**触发 `onunhidefn`（源码注释明确保留该行为，components/prophider.lua:66-70）；"现身=找到目标"的 mod 需求应改用 `onvisiblefn`。
- 重复 `HideWithProp` 在已 hiding 时直接返回（:79-81），不会重置 prop 或 counter。
- 存档兼容：`OnLoad` 同时接受旧字段 `hidetime`（:150-155）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，通读组件全部 161 行；核验 spat 与 warg 两处挂载及 `HideWithProp` 触发点（spat.lua:193-194、warg.lua:388-389）。
- 游戏运行：未运行。
