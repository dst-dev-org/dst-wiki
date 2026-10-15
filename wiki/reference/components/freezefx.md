---
title: "freezefx"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "多档火焰式光效组件（类名 FireFX，freezefx 变体）：按 levels 配置切换光半径/强度/动画/音效，自带闪烁与熄灭收尾；原版无挂载者。"
aliases: ["冻结特效", "火焰光效变体", "freezefx"]
tags: [component]
dst_build: "steam-buildid:26230496"
size_class: simple
sources: ["components/freezefx.lua", "components/firefx.lua", "prefabs/crabking.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# freezefx

## 快速判断

- 职责与适用：按等级表驱动一个实体的 Light/AnimState/SoundEmitter 呈现"火焰类"多档光效，支持档内百分比插值与熄灭动画；**原版没有任何 AddComponent/require 消费者**（全库 grep 核验），蟹王的 `inst.freezefx` 是同名局部函数、与组件无关。[无消费者佐证](../../../raw/current/scripts/prefabs/crabking.lua#L1954
- 优先使用：切档 `SetLevel(lev)`，档内插值 `SetPercentInLevel(percent)`，熄灭 `Extinguish()`；levels 表由使用方注入。[SetLevel](../../../raw/current/scripts/components/freezefx.lua#L63-L110)
- 重要副作用：构造即 `StartUpdatingComponent` 每帧给 `Light:SetRadius` 加正弦闪烁（±0.05）；可选昼夜音效参数（`"fire","daytime"` 1/2）。[OnUpdate](../../../raw/current/scripts/components/freezefx.lua#L18-L34)
- 修改边界：假定实体自带 Light/AnimState/SoundEmitter（部分调用有 SoundEmitter 判空，Light 无判空）；`level` 从 0 起语义为"未点燃"，`SetLevel` 拒绝 ≤0。[构造](../../../raw/current/scripts/components/freezefx.lua#L1-L12)

## 关键接口

| 接口／字段 | 已核验类型／单位 | 条件、作用与证据 |
|---|---|---|
| `levels` | 数组，条目含 `radius/intensity/falloff/colour{3}/sound/soundintensity?/pre?/anim/pst?` | 每档的光参数与表现；必须先注入再 SetLevel，否则索引 nil 报错。[字段](../../../raw/current/scripts/components/freezefx.lua#L3-L11)、[取值](../../../raw/current/scripts/components/freezefx.lua#L90-L107) |
| `SetLevel(lev)` | 无返回 | 升档且 `lev >= bigignitesoundthresh`（默认 3）播大火引燃音，否则小火；首档可播 `pre` 前摇再循环 `anim`；设 Light 四参数并换循环音。[L63-L110](../../../raw/current/scripts/components/freezefx.lua#L63-L110) |
| `SetPercentInLevel(percent)` | 0..1 | 在当前档与前一档之间插值 radius 与 intensity（第 1 档以 `levels[1]` 为低点）。[L48-L61](../../../raw/current/scripts/components/freezefx.lua#L48-L61) |
| `Extinguish()` | 布尔或 nil | 停 `"fire"` 声、可经 `extinguishsoundtest` 跳过；播 `dontstarve/common/fireOut` 与 `pst` 淡出动画——返回 true 表示"有淡出动画，宿主别立刻移除实体"。[L114-L125](../../../raw/current/scripts/components/freezefx.lua#L114-L125) |
| `usedayparamforsound`／`playignitesound`／`bigignitesoundthresh` | 布尔／布尔／数值(3) | 开关昼夜音参、引燃音与大火阈值，构造后可覆写。[L8-L11](../../../raw/current/scripts/components/freezefx.lua#L8-L11) |

## 与 firefx 的关系

- 同目录 `firefx.lua` 是活跃使用的同族组件（类名同为 FireFX）：campfirefire、coldfirefire、fire、character_fire 等 prefab `AddComponent("firefx")`；它额外生成 `firefx_light` 子实体并支持 `AttachLightTo`、`OnRemoveEntity` 清理。[对照实现](../../../raw/current/scripts/components/firefx.lua#L1-L49)
- freezefx 是无子实体的精简变体：没有独立灯光实体、没有 OnRemoveEntity，构造即起闪烁更新；从函数集合看不支持挂靠到别的实体上。原版选择的是 firefx，freezefx 无调用点。
- 重复声明：组件文件里 `local FireFX = Class(...)` 与 firefx 同名但分属各自文件作用域，同 entity 同时挂两个组件不会冲突。[类定义](../../../raw/current/scripts/components/freezefx.lua#L1-L2)

## 常见陷阱与限制

- 原版零消费者：`grep -rn "freezefx"` 仅命中组件文件本身与 crabking 的同名局部函数（`crabking.lua` 的 freezefx/freezetask 是冰冻表现逻辑，未用本组件）；以它为前提的任何"原版行为"结论都不成立，接入需自行 AddComponent 并注入 levels。[crabking 局部函数](../../../raw/current/scripts/prefabs/crabking.lua#L2049
- `SetLevel` 直接索引 `self.levels[lev]` 且在比较前无保护：levels 未注入或 lev 超界（>0 但超数组长度时被 `math.min` 截断，0/负数被拒）——注入错误表会运行时报错。[L64-L90](../../../raw/current/scripts/components/freezefx.lua#L64-L90)
- `Extinguish` 不清 `self.level`：熄灭后再 `SetLevel` 同档是 no-op（`lev ~= self.level` 才继续），复用前需重置或换档。[L64](../../../raw/current/scripts/components/freezefx.lua#L64)、[L114-L125](../../../raw/current/scripts/components/freezefx.lua#L114-L125)
- 闪烁 `OnUpdate` 无条件写 `self.inst.Light`：实体没有 Light 组件时每帧报错；构造即起更，挂上就要求宿主具备三件套。[L18-L23](../../../raw/current/scripts/components/freezefx.lua#L18-L23)

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。覆盖 freezefx.lua 全文与 firefx.lua 的构造/清理段对照；全库 grep 确认无 `AddComponent("freezefx")`、无 `require("components/freezefx")`，并核对 crabking 同名局部函数非消费者；未覆盖该组件的历史用途（快照内无证据）。
- 游戏运行：未运行；本页结论均来自源码静态核验。
- 结构检查：`validate_page.py`、`validate_source_links.py --strict` 通过（见批记录）。
