---
title: "werebeast"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "月圆变身组件：满月自动或触发计数达限后调用回调换形态并限时变回，保存剩余变身时间，供猪人类生物与自定义兽形态。"
aliases: ["月圆变身", "兽形态切换", "werebeast"]
tags: [component, state-machine, lifecycle]
dst_build: "steam-buildid:26230496"
size_class: standard
sources: ["components/werebeast.lua", "prefabs/pigman.lua", "components/kramped.lua", "prefabs/pighouse.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# werebeast

## 快速判断

- 职责与适用：给生物加"兽形态"状态机。满月切换（WatchWorldState "isfullmoon"，components/werebeast.lua:68）或 `TriggerDelta` 累计到 `triggerlimit` 时进入兽态（lines 16-33、100-107），形态外观与 stategraph 由回调负责。
- 优先使用：猪人式接入——`SetOnWereFn`／`SetOnNormalFn` 设形态回调、`SetTriggerLimit` 设吃怪肉变身次数（prefabs/pigman.lua:681-683）。
- 重要副作用：`SetWere`／`SetNormal` 向实体推送 `transformwere`／`transformnormal` 事件（lines 121、138），SGpig.lua:32 与 SGwerepig.lua:24 用事件驱动状态切换；淘气值系统跳过兽态击杀（components/kramped.lua:93-95）。
- 修改边界：可随时 `RemoveComponent`，`OnRemoveFromEntity` 取消任务并解除监听（lines 73-85）；进入 limbo 挂起变身、退出后 0.2 秒重判（lines 40-54、69-70）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `weretime` | 默认 `TUNING.SEG_TIME * 4` | 兽态持续时长，`SetWere` 缺省使用（line 61、127） |
| `SetOnWereFn(fn)`／`SetOnNormalFn(fn)` | `(fn(inst))` | 变身／变回回调，先于事件推送调用（lines 87-93、118-120、135-137） |
| `SetWere(time)` | `(time秒可选)` | 进兽态：取消转换任务、回调、推 `transformwere`、重置触发计数、安排变回任务（lines 113-128） |
| `SetNormal()` | 无参 | 变回：回调、推 `transformnormal`、取消变回任务（lines 130-145） |
| `IsInWereState()` | `-> bool` | 以 `_reverttask ~= nil` 判定（lines 147-149） |
| `SetTriggerLimit(limit)`＋`TriggerDelta(amount)` | `(number)` | 设上限后开始累计，达限自动 `SetWere()`；`ResetTriggers` 归零（lines 95-111；pigman.lua:137 吃负益怪肉 +1） |
| `OnSave`／`OnLoad` | 存 `time` | 等待满月变身时存 `weretime`，兽态存剩余变回时间；OnLoad 直接 `SetWere(data.time)`（lines 151-164） |

### 满月时序

`isfullmoon` 变化 → 非_limbo_时 1~2 秒随机延迟执行 `DoTransform`（lines 30-31、1-8）：满月则 `SetWere`，否则 `SetNormal`。兽态下满月重置会把变回时刻顺延到当日结束加 1~2 秒（lines 22-29）；`OnRevert` 在实体无 `transform` 状态标签时才变回（lines 10-14）。

## 典型接入

### 给自定义生物加满月变身（服务端 master postinit）

- 前提与选择：组件只管状态与计时；外观、动画 build 与 stategraph 切换写在回调里，与原版 pigman.lua:681-683 相同的分工。
- 生效与撤销：满月自动双向切换；不设 `SetTriggerLimit` 时只走满月路径。`RemoveComponent` 后恢复常驻形态，回调不再触发（lines 73-85）。
- 原版依据：pigman.lua:681-683（挂载与上限 4）；pighouse.lua:140-141、159-160（房主监听变身事件并重置触发计数）。

```lua
local function OnBecomeWere(inst)
    inst.AnimState:SetBuild("werepig_build")
    inst:SetStateGraph("SGwerepig")
end

local function OnBecomeNormal(inst)
    inst:SetStateGraph("SGpig")
end

-- master postinit：
inst:AddComponent("werebeast")
inst.components.werebeast:SetOnWereFn(OnBecomeWere)
inst.components.werebeast:SetOnNormalFn(OnBecomeNormal)
inst.components.werebeast:SetTriggerLimit(3) -- 可选：吃 3 次负益肉类提前变身
```

- 调整与验证：`weretime` 单位秒；触发计数由 prefab 决定递增条件（原版在 OnEat 内调用 `TriggerDelta(1)`，pigman.lua:132-137）。场景脚本可强制 `SetWere()`（scenarios/chest_terrarium_pigs.lua:5-7）。

## 常见陷阱与限制

- **回调不自动换装：** 组件不修改 AnimState／stategraph／战斗数值，忘记在 `onsetwerefn` 里换形态会只看到事件与计时变化。
- **limbo 中不动：** 进入 limbo 取消待执行的变身任务，退出后固定 0.2 秒重判当前满月态（lines 40-54），短瞬间可能出现"满月但未变身"。
- **存档只覆盖两条路径：** 等待变身期间存档只记 `weretime` 而丢失随机延迟（lines 151-153）；变回剩余时间 `floor` 取整（lines 154-157），载入后以该值重新 `SetWere`。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。通读 components/werebeast.lua 全文（171 行）；核验 pigman.lua:681-683、681 附近 OnEat 触发链、kramped.lua:93-95、pighouse.lua:140-141/159-160、SGpig.lua:32 与 SGwerepig.lua:24 事件处理、scenarios/chest_terrarium_pigs.lua:5-7。未运行游戏。
- 结构检查：validate_page.py / validate_source_links.py 通过（见批级自验）。
