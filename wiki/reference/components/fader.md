---
title: "fader"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "通用数值渐变器：注册起点到终点的回调式补间，逐帧驱动 setter，可挂结束回调，常用于透明度与音量。"
aliases: ["fader", "数值渐变", "透明度渐变", "补间组件"]
tags: [component]
dst_build: "steam-buildid:26230496"
size_class: simple
sources: ["components/fader.lua", "prefabs/marionette_fx.lua", "stategraphs/SGwilson.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# fader

## 快速判断

- 职责与适用：在同一实体上并行维护多个"起始值→结束值、时长 time"的线性补间，每帧把当前值交给 setter 回调；首个渐变时自动开更。([fader.lua:8-24](../../../raw/current/scripts/components/fader.lua#L8-L24) ([fader.lua:54-56](../../../raw/current/scripts/components/fader.lua#L54-L56)
- 优先使用：`Fade(startval, endval, time, setter, atend, id)` 起一个渐变；不透明度类需求用 `AnimState:SetMultColour` 作 setter（原版提线木偶 FX）。([marionette_fx.lua:12-13](../../../raw/current/scripts/prefabs/marionette_fx.lua#L12-L13)
- 重要副作用：`StopAll` 会把所有进行中的渐变立即推到终点并触发各 atend，不是静默丢弃。([fader.lua:26-30](../../../raw/current/scripts/components/fader.lua#L26-L30)
- 修改边界：无按 id 停止单个渐变的接口，只有 StopAll；渐变不存档。([fader.lua:26-30](../../../raw/current/scripts/components/fader.lua#L26-L30)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `Fade(startval, endval, time, setter, atend, id)` | 返回 id（默认为插入序号） | rate=(end-start)/time；time 为秒须大于 0。setter 签名 `setter(value, inst)`。([fader.lua:8-24](../../../raw/current/scripts/components/fader.lua#L8-L24) [fader.lua:10](../../../raw/current/scripts/components/fader.lua#L10 |
| `atend` 回调 | `atend(inst, value)` | 渐变到点后调用；注意与 setter 参数顺序相反。([fader.lua:45-47](../../../raw/current/scripts/components/fader.lua#L45-L47) |
| `StopAll()` | 无参 | 以超大 dt 跑完全部渐变（触发 atend）后清空并停更。([fader.lua:26-30](../../../raw/current/scripts/components/fader.lua#L26-L30) |
| `OnUpdate(dt)` | 逐帧推进 | t 到点写 endval，否则 val+=rate*dt；先调 setter 再判结束。([fader.lua:32-52](../../../raw/current/scripts/components/fader.lua#L32-L52) |

## 典型接入

### FX 渐显后自动移除

- 前提与选择：装在任意带更新能力的实体上；原版木偶 FX 用 15 帧渐隐并以 `inst.Remove` 作 atend，渐显则无 atend。[marionette_fx.lua:13](../../../raw/current/scripts/prefabs/marionette_fx.lua#L13 ([marionette_fx.lua:97-98](../../../raw/current/scripts/prefabs/marionette_fx.lua#L97-L98)

```lua
inst:AddComponent("fader")
inst.components.fader:Fade(0, 0.8, 15 * FRAMES, function(val, ent)
    ent.AnimState:SetMultColour(1, 1, 1, val)
end, function(ent, val)
    ent:Remove()
end)
```

- 生效与撤销：温迪复活动画用 atend 里再启新渐变做"亮起后回暗"的级联，可作串接参照。([SGwilson.lua:3647-3654](../../../raw/current/scripts/stategraphs/SGwilson.lua#L3647-L3654)

## 常见陷阱与限制

- setter 与 atend 参数顺序不同：`setter(value, inst)`、`atend(inst, value)`，写反会把实体当数值用。[fader.lua:42](../../../raw/current/scripts/components/fader.lua#L42 [fader.lua:46](../../../raw/current/scripts/components/fader.lua#L46
- time 为 0 会除零（rate 计算）；瞬时变化直接调渲染接口即可，不必走本组件。[fader.lua:10](../../../raw/current/scripts/components/fader.lua#L10
- 多个渐变并行写同一渲染属性（如都改 MultColour）时按注册顺序互相覆盖，最后注册者每帧生效。

## 验证记录

- 源码审阅：2026-10-28，`steam-buildid:26230496`：审阅组件全文、marionette_fx 两个方向渐变、SGwilson 级联渐变，及 abigail／wendy／grotto 水声等其余消费者添加模式。
- 游戏运行：未运行；未核对音量渐变（grottowaterfallsoundcontroller）实际听感。
- 结构检查：validate_page.py 通过；validate_source_links.py --strict 通过（本批 16 页合并执行）。
