---
title: "boatdrag（船用阻力源）"
created: 2026-10-21
updated: "2026-10-28"
type: component
summary: "船用阻力/限速数据源：提供 drag、max_velocity_mod、forcedampening、sailforcemodifier 四个因子，由 boatphysics:AddBoatDrag 接入结算"
aliases: ["船用阻力", "锚阻力", "限速", "boatdrag"]
tags: [component, tuning]
dst_build: "steam-buildid:26230496"
sources: ["components/boatdrag.lua", "components/boatphysics.lua", "components/anchor.lua", "prefabs/anchor.lua", "prefabs/winch.lua", "prefabs/crabking_claw.lua", "stategraphs/SGanchor.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# boatdrag（船用阻力源）

## 快速判断

- 职责与适用：`boatdrag` 是纯数据组件，本身不更新、不结算、也不自动注册；它把四个因子挂在“阻力提供者”实体上，由船的 `boatphysics` 在注册后读取。[字段](../../../raw/current/scripts/components/boatdrag.lua#L1-L18)
- 链上位置：船—帆—舵—阻力链的“阻力”端。它同时影响四条路径：额外阻力 `drag`、速度上限倍率 `max_velocity_mod`、外力衰减 `forcedampening`、帆力倍率 `sailforcemodifier`；帆/舵本身见 mast、steeringwheel。
- 优先使用：给实体 `AddComponent("boatdrag")`，写字段，再在生效时调用 `boat.components.boatphysics:AddBoatDrag(inst)`；撤销用 `RemoveBoatDrag(inst)`。原版锚与绞盘都按“绑定状态/限时”成对调用。[注册](../../../raw/current/scripts/components/boatphysics.lua#L222-L242)
- 重要副作用：注册只把 `inst.components.boatdrag` 放入 `boatphysics.boatdraginstances` 并监听提供者的 `onremove`/`death`/`onburnt`；字段值在每次 `GetMaxVelocity`/`GetBoatDrag` 等结算时实时读取，改字段立即生效。注册/撤销还会触发 `physicsmodifiedexternally:RecalculateExternalVelocity()`（若存在）。[监听](../../../raw/current/scripts/components/boatphysics.lua#L222-L231)
- 修改边界：只对服务端 `boatphysics` 有意义；提供者通常必须在船上（自动移除时用 `GetCurrentPlatform()` 找船）。组件无 `OnSave`/`OnLoad`，字段与注册关系都不存档，读档后需由 prefab/状态重新施加。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `drag` | 数值，默认 0 | 所有已注册提供者的 `drag` 相加，进入 `GetBoatDrag` 的 `outExpo` 额外阻力项，并在 `ApplyDrag` 中按 `dt` 从速度扣除；数值近似“每秒扣多少速度单位”。[求和](../../../raw/current/scripts/components/boatphysics.lua#L386-L406) |
| `max_velocity_mod` | 数值，默认 1 | 乘到 `GetMaxVelocity` 的最终上限上；多个提供者连乘。1 为中性，0.15 表示 15%，0 表示上限归零。[乘法](../../../raw/current/scripts/components/boatphysics.lua#L377-L383) |
| `forcedampening` | 数值，默认 0 | `GetForceDampening` 从 1 逐个减去该值并下限 0；结果乘到 `ApplyForce`/`ApplyRowForce` 的力上，1 表示完全压掉划桨/冲击等外力；它不作用于 `ApplySailForce`，帆力另由 `sailforcemodifier` 控制。[衰减](../../../raw/current/scripts/components/boatphysics.lua#L289-L297) |
| `sailforcemodifier` | 数值，默认 1 | 所有提供者连乘后乘到每条桅杆的 `CalcSailForce()` 上；0.8 表示帆力打八折，0 表示帆力归零。[帆力倍率](../../../raw/current/scripts/components/boatphysics.lua#L545-L549) |
| `AddBoatDrag(boatdraginst)` | 传入带 `boatdrag` 组件的实体 | 写入 `boatdraginstances`、在需要时重算外部速度、监听提供者的 `onremove`/`death`/`onburnt`；不校验组件是否存在，缺失时该键写入 `nil`，等同于未注册。[L222-L231](../../../raw/current/scripts/components/boatphysics.lua#L222-L231) |
| `RemoveBoatDrag(boatdraginst)` | 同上 | 从表里删除并移除三个监听；不重置提供者字段。[L233-L242](../../../raw/current/scripts/components/boatphysics.lua#L233-L242) |
| `on_boatdrag_removed(boatdraginst)` | 内部回调 | 提供者被移除/死亡/烧毁时，用其**当时**的 `GetCurrentPlatform()` 找船并 `RemoveBoatDrag`；平台为空就静默跳过，留下悬空条目。[L137-L142](../../../raw/current/scripts/components/boatphysics.lua#L137-L142) |
| `GetDebugString()` | 返回字符串 | 打印 drag/max_velocity_mod/forcedampening，不含 sailforcemodifier；仅调试。[L14-L16](../../../raw/current/scripts/components/boatdrag.lua#L14-L16) |

## 典型接入

### 原版锚（anchor / yotd_anchor）与绞盘（winch）

- 前提与选择：目标文件 `scripts/prefabs/anchor.lua`。服务端在 `AddComponent("anchor")` 后 `AddComponent("boatdrag")`，把 `drag` 设为 `ANCHOR_DRAG`（2）、`max_velocity_mod` 设为 0.15、`sailforcemodifier` 设为 0.8；YOTD 锚复用同一 `common_fn`。[锚配置](../../../raw/current/scripts/prefabs/anchor.lua#L110-L115) [注册名](../../../raw/current/scripts/prefabs/anchor.lua#L152-L156)
- 加载与端：prefab 的物理/动画先建，`TheWorld.ismastersim` 为假时提前返回；`boatdrag` 与 `anchor` 都只在服务端加。[端判断](../../../raw/current/scripts/prefabs/anchor.lua#L100-L112)
- 生效与撤销：锚下落状态图调用 `SetIsAnchorLowered(true)`，`Anchor:SetIsAnchorLowered` 在状态变化时执行 `AddBoatDrag(self.inst)`；起锚时 `false` 走 `RemoveBoatDrag`。只加组件不调用这两个方法，锚对船速没有任何影响。[状态图](../../../raw/current/scripts/stategraphs/SGanchor.lua#L2-L12) [锚组件](../../../raw/current/scripts/components/anchor.lua#L145-L157)
- 另一个原版模式：`winch` 在 `AddComponent("boatdrag")` 后使用同一组锚参数，但只在打捞成功时 `turn_on_boat_drag(inst, boat, TUNING.BOAT_WINCH.BOAT_DRAG_DURATION)`（默认 1 秒），到时自动 `RemoveBoatDrag`；适合“短时减速”而不是常驻锚。[绞盘配置](../../../raw/current/scripts/prefabs/winch.lua#L504-L507) [限时注册](../../../raw/current/scripts/prefabs/winch.lua#L98-L124) [触发](../../../raw/current/scripts/prefabs/winch.lua#L142 [时长](../../../raw/current/scripts/tuning.lua#L5502)
- 注册名与资源：`anchor`、`yotd_anchor`、`winch`；视觉资源为 `anim/boat_anchor.zip`、`anim/yotd_anchor.zip`、`anim/boat_winch.zip` 等，阻力组件本身无资源、无动画。[绞盘注册](../../../raw/current/scripts/prefabs/winch.lua#L546-L547)

服务端最小挂接骨架（放在阻力提供者的 prefab `fn_pst` 与状态切换处；不是完整 prefab）：

```lua
local boatdrag = inst:AddComponent("boatdrag")
boatdrag.drag = TUNING.BOAT.ANCHOR.BASIC.ANCHOR_DRAG
boatdrag.max_velocity_mod = TUNING.BOAT.ANCHOR.BASIC.MAX_VELOCITY_MOD
boatdrag.sailforcemodifier = TUNING.BOAT.ANCHOR.BASIC.SAILFORCEDRAG
```

- 调整与验证：字段是“实时读取”的，可在注册后改；但 `max_velocity_mod` 是倍率、`drag` 是每秒速度单位、`forcedampening` 是 0..1 的比例，三者单位不同，不能互换。原版锚参数见 `TUNING.BOAT.ANCHOR.BASIC`。[数值](../../../raw/current/scripts/tuning.lua#L4235-L4242) 本页源码审阅范围见“验证记录”，未做游戏内运行。

## 常见陷阱与限制

- 只 `AddComponent("boatdrag")` 不会产生任何效果：必须有 `boatphysics:AddBoatDrag(inst)`。快照内只有锚组件与绞盘调用注册；`crabking_claw` 虽在第 139-143 行设置了 `drag=10`、`forcedampening=1`、`max_velocity_mod=0`、`sailforcemodifier=0`，但全库未找到它的 `AddBoatDrag` 调用，因此这些值在当前快照不会进入船的结算。接入前应先确认这是否为未启用的半成品路径。[巨钳配置](../../../raw/current/scripts/prefabs/crabking_claw.lua#L139-L143) [注册名](../../../raw/current/scripts/prefabs/crabking_claw.lua#L280
- `AddBoatDrag` 直接写 `boatdraginst.components.boatdrag`，不检查是否为 `nil`；先加组件再注册。若传入无组件实体，写进去的值是 `nil`，表中等于没有该键，船速结算会静默忽略它，但事件监听仍会挂上。[写入表](../../../raw/current/scripts/components/boatphysics.lua#L222-L224) [读取](../../../raw/current/scripts/components/boatphysics.lua#L377-L381)
- 自动移除依赖提供者“当前仍在同一平台”：`on_boatdrag_removed` 在触发时用 `GetCurrentPlatform()` 重新找船，找不到就跳过；先把提供者搬离船/置入 Limbo 再移除，会留下仍在 `boatdraginstances` 的旧条目。需要在搬离前显式 `RemoveBoatDrag`。[自动移除](../../../raw/current/scripts/components/boatphysics.lua#L137-L142)
- `forcedampening` 不压帆力：它乘在 `ApplyForce`/`ApplyRowForce` 的外力上，1 会把划桨、冲击、漩涡推送等压掉，但 `ApplySailForce` 不经过它；帆力由 `sailforcemodifier` 单独缩放。`GetForceDampening` 有 0 下限，多个提供者叠加超过 1 后仍按 0 处理。[衰减应用](../../../raw/current/scripts/components/boatphysics.lua#L299-L342)
- `max_velocity_mod` 最终仍受 `TUNING.BOAT.MAX_ALLOWED_VELOCITY = 10` 限制，且先与桅杆汇总上限、船自身 `boatphysics.max_velocity` 相乘；把倍率设为 0 只让上限归零，不会自动把当前速度清零，减速仍靠 `drag`/`forcedampening`。[上限链](../../../raw/current/scripts/components/boatphysics.lua#L344-L384)
- 组件没有存档方法：`drag` 等值不会写入存档，`boatdraginstances` 也不会；读档后必须由 prefab 初始化或状态图重新设置并注册，否则锚可能已落下但阻力丢失。[组件全文](../../../raw/current/scripts/components/boatdrag.lua#L1-L18)

## 验证记录

- 版本增量复核 26148036（2026-10-25）：锚点重定位 2 处（tuning BOAT_WINCH 时长、BOAT.ANCHOR.BASIC 参数段均 +14）；无结论修订，仅锚点迁移——锚 drag=2／0.15／0.8、绞盘限时 1 秒值未变（tuning-hunk-map 5 改值键不含），boatdrag.lua、boatphysics.lua、anchor／winch／crabking_claw 及 SGanchor 全部未变。依据：tuning-hunk-map 键级审阅＋变更清单命中 tuning.lua。
- 源码审阅：2026-10-21，`steam-buildid:25643519`；完整阅读 `components/boatdrag.lua`，并核验 `components/boatphysics.lua` 的 `AddBoatDrag`/`RemoveBoatDrag`/`GetTotalAnchorDrag`/`GetBoatDrag`/`GetMaxVelocity`/`GetForceDampening`/`GetAnchorSailForceModifier`/`ApplyDrag`、`components/anchor.lua` 与 `prefabs/anchor.lua`、`prefabs/winch.lua`、`prefabs/crabking_claw.lua`、`stategraphs/SGanchor.lua` 与 `tuning.lua`。未审阅绞盘打捞的完整状态机与巨钳的行为设计意图。
- 游戏运行：未运行；未验证锚下落/起锚时速度曲线的实际手感、巨钳未注册是否确为版本意图、读档后锚阻力恢复。
- 结构检查：`validate_page.py` 通过（0 问题）；`validate_source_links.py --strict` 通过：本页 30 个 Lua 引用，0 error / 0 warning；本页 Lua 片段经 `parser_probe.exe --syntax-only` 通过（0 错误）。

相关页面：[[reference/components/boatphysics]]、[[reference/components/mast]]、[[reference/components/steeringwheel]]。
