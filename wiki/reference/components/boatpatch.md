---
title: "boatpatch（船体补丁）"
created: 2026-10-21
updated: "2026-10-28"
type: component
summary: "给修补道具标记 patch_type 与 boat_patch 标签，由船 leak 的 Repair 流程消耗并切换修补外观"
aliases: ["船体补丁", "补漏", "boatpatch", "patch_type"]
tags: [component]
dst_build: "steam-buildid:26230496"
sources: ["components/boatpatch.lua", "prefabs/boatpatch.lua", "prefabs/sewing_tape.lua", "prefabs/treegrowthsolution.lua", "components/boatleak.lua", "components/repairable.lua", "prefabs/boat.lua", "actions.lua", "componentactions.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# boatpatch（船体补丁）

## 快速判断

- 职责与适用：`boatpatch` 是修补道具上的“补丁类型”标记组件：构造时加 `boat_patch` 标签，`patch_type` 决定 leak 被补好后播放哪种修补外观。它本身不执行修补、不扣物品、不加血。[构造](../../../raw/current/scripts/components/boatpatch.lua#L1-L6)
- 链上位置：船—帆—舵—阻力链之外的“损伤修复”支路：它把道具接入 `boatleak:Repair` 与 `repairable:Repair`，不参与航速计算。
- 优先使用：自定义补漏道具 = 在 `SetPristine` 前加的 `boat_patch` 标签 + 服务端 `boatpatch` 组件 + 可选 `repairer`（同材质时顺带修船体）+ `stackable`；补丁类型写 `patch_type`。[原版木补丁](../../../raw/current/scripts/prefabs/boatpatch.lua#L43-L91)
- 重要副作用：修补会消耗一个道具，并把目标 leak 从 `boat_leak` 切换为修补态；`patch_type` 对应 `repaired_<type>` 的动画/时限分支，未知类型不会真正变成修补态。[状态分支](../../../raw/current/scripts/components/boatleak.lua#L234-L258)
- 修改边界：只对带 `boat_leak` 标签的目标生效，动作距离 2.5；服务端结算消耗，客户端只负责发起动作。修补船体血量是 `repairer` + `repairable` 的职责，不是本组件。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `patch_type` | 字符串或 `nil`，默认 `nil` | 木补丁保持 `nil`，补好后走 `"repaired"`；原版另有 `"kelp"`、`"tape"`、`"treegrowth"`。类型只在 `GetPatchType` 读取。[默认值](../../../raw/current/scripts/components/boatpatch.lua#L1-L6) |
| `GetPatchType()` | 返回 `patch_type` | 唯一消费者是 `BoatLeak:Repair`，用来拼 `repair_state = "repaired_"..patch_type`；`patch_type=nil` 时用 `"repaired"`。[L12-L14](../../../raw/current/scripts/components/boatpatch.lua#L12-L14) |
| `OnRemoveFromEntity()` | 引擎回调 | 组件移除时移除 `boat_patch` 标签；不影响道具上的其他组件。[L8-L10](../../../raw/current/scripts/components/boatpatch.lua#L8-L10) |
| 构造副作用 | 加标签 | 构造即 `inst:AddTag("boat_patch")`；原版 prefab 还在 `SetPristine` 前又加了一次，保证客户端也能看到该标签。[L5-L5](../../../raw/current/scripts/components/boatpatch.lua#L5 [prefab 标签](../../../raw/current/scripts/prefabs/boatpatch.lua#L52-L53) |
| `BoatLeak:Repair(doer, patch_item)` | 返回布尔 | 目标必须有 `boat_leak`；先尝试平台 `repairable:Repair`（成功则道具已在那里消耗），否则由 `Repair` 从堆叠取一个移除；随后按 `patch_type` 播放修补态。[L62-L101](../../../raw/current/scripts/components/boatleak.lua#L62-L101) |
| `Repairable:Repair(doer, repair_item)` | 返回布尔 | 要求 `repairmaterial` 与目标一致；按 `healthrepairvalue` 加船体血、播音效，并从堆叠消耗一个。无 `repairer`/材质不匹配时返回 `false`，leak 仍会被补但船体不回血。[L148-L180](../../../raw/current/scripts/components/repairable.lua#L148-L180) |
| `REPAIR_LEAK` | 动作，距离 2.5 | 由 `boatpatch` 组件动作分支在“道具 `boat_patch` + 目标 `boat_leak`”时加入；执行时转调 `BoatLeak:Repair`。[动作定义](../../../raw/current/scripts/actions.lua#L560 [候选](../../../raw/current/scripts/componentactions.lua#L1070-L1072) |

## 典型接入

### 原版补丁宿主（boatpatch / boatpatch_kelp / 缝纫胶带 / 树果酱）

- 前提与选择：目标文件 `scripts/prefabs/boatpatch.lua`。木补丁 `fn_wood` 在服务端 `AddComponent("boatpatch")`，`patch_type` 保持 `nil`；海带补丁 `fn_kelp` 复用 `fn_wood` 后把 `patch_type` 设为 `"kelp"`。[木补丁](../../../raw/current/scripts/prefabs/boatpatch.lua#L43-L91) [海带补丁](../../../raw/current/scripts/prefabs/boatpatch.lua#L93-L132)
- 加载与端：prefab 文件由游戏加载；`fn_wood` 先建实体并在 `SetPristine` 前加 `boat_patch` 标签，客户端可见；`TheWorld.ismastersim` 为假时在第 65-67 行返回，`boatpatch` 组件只在服务端存在。[端判断](../../../raw/current/scripts/prefabs/boatpatch.lua#L63-L69)
- 注册名与资源：`boatpatch`（木补丁）与 `boatpatch_kelp`（海带补丁）；资源为 `anim/boat_repair_build.zip`、`anim/boat_repair.zip`、`anim/boat_repair_kelp.zip`。其他原版宿主：`sewing_tape`（`"tape"`）、`treegrowthsolution`（`"treegrowth"`）。[注册](../../../raw/current/scripts/prefabs/boatpatch.lua#L134-L136) [缝纫胶带](../../../raw/current/scripts/prefabs/sewing_tape.lua#L39-L40) [树果酱](../../../raw/current/scripts/prefabs/treegrowthsolution.lua#L72-L78)
- 单位与消耗：木补丁 `repairer.healthrepairvalue = REPAIR_LOGS_HEALTH * 2`（默认 12.5 × 2 = 25 船体血）；海带补丁 `REPAIR_KELP_HEALTH * 配方海带数`（默认 10 × 3 = 30）；树果酱 20。三者都只消耗堆叠中的一个；海带补丁还有可腐烂、可食用、可燃等物品属性。[数值](../../../raw/current/scripts/prefabs/boatpatch.lua#L69-L77) [tuning](../../../raw/current/scripts/tuning.lua#L2118-L2122)
- 生效：玩家拿补丁对船体 leak 执行 `REPAIR_LEAK`；`BoatLeak:Repair` 先尝试修船体血（仅当平台 `repairable` 材质匹配），再改 leak 外观。木补丁走永久 `"repaired"`，缝纫胶带永久 `"repaired_tape"`，海带 `"repaired_kelp"` 到时限后回到 `med_leak`，树果酱补完在动画结束后移除 leak 实体。[修补态](../../../raw/current/scripts/components/boatleak.lua#L234-L258)
- 撤销：木/胶带补丁不可撤销；海带补丁的修补是临时的，时限为小漏 0.75 天、中漏 0.5 天。[海带时限](../../../raw/current/scripts/components/boatleak.lua#L251-L258) [tuning](../../../raw/current/scripts/tuning.lua#L8255-L8256)

服务端最小挂接骨架（放在自定义补丁 prefab 内；不是完整 prefab，图标/动画/堆叠需另备）：

```lua
inst:AddTag("boat_patch")
inst:AddComponent("boatpatch")
inst.components.boatpatch.patch_type = "kelp"
```

- 调整与验证：自定义 `patch_type` 必须同时在 `BoatLeak:SetState` 有对应分支，否则 `Repair` 会消耗道具却不改变 leak 状态；仅靠本组件不能新增修补表现。本页源码审阅范围见“验证记录”，未做游戏内运行。

## 常见陷阱与限制

- 只加 `boatpatch` 组件、不写 `patch_type` 不会出错，但补丁效果固定为木补丁的 `"repaired"`；写一个不在 `BoatLeak:SetState` 分支里的类型（如 `"mytype"`）会得到 `repair_state="repaired_mytype"`，`SetState` 无匹配分支，leak 仍是 `boat_leak`，道具却被消耗。[未知类型路径](../../../raw/current/scripts/components/boatleak.lua#L234-L258)
- `boat_patch` 标签在组件构造时由服务端加，但原版还在 `SetPristine` 之前加一份（prefab 第 53 行）；自定义道具应照做，不要只依赖服务端组件回调加标签，否则动作候选或非主机侧可能看不到它，表现为“拿着补丁没有补漏选项”。[双份标签](../../../raw/current/scripts/prefabs/boatpatch.lua#L52-L69)
- 海带补丁的 `repairer.repairmaterial` 是 `MATERIALS.KELP`，而普通木船 `repairable.repairmaterial` 是 `MATERIALS.WOOD`；材质不匹配时 `Repairable:Repair` 返回 `false`，海带补丁只补漏、不回船体血，但仍消耗一个。[材质](../../../raw/current/scripts/prefabs/boatpatch.lua#L111-L113) [木船材质](../../../raw/current/scripts/prefabs/boat.lua#L572-L573)
- 修补会把一个堆叠单位移出：若 `repairable:Repair` 成功，它已经消耗道具，`BoatLeak:Repair` 用 `consumed` 标记避免二次移除；在动作回调里再手动扣道具会多扣。[消耗判定](../../../raw/current/scripts/components/boatleak.lua#L69-L85)
- 海带修补不是永久修复：`repaired_kelp` 会 `SetRepairedTime`，到时播放 timeout 并回到 `med_leak`；把海带补丁当木补丁用于长期封漏会在数天后复漏。[超时回退](../../../raw/current/scripts/components/boatleak.lua#L48-L60) [L251-L258](../../../raw/current/scripts/components/boatleak.lua#L251-L258)
- 树果酱补丁在 `ChangeToRepaired` 后监听 `animover`，`idle` 播完会 `Remove()` 整个 leak 实体；因此它不是“永久补丁外观”，而是把 leak 实体移除，别再假设之后还能查询该 leak。[树果酱分支](../../../raw/current/scripts/components/boatleak.lua#L240-L249)
- `boatpatch` 不负责船体血量：船体修复由 `repairable`/`repairer` 的 `REPAIR` 路径处理，且不同船体的可修材质不同（草船为 `MATERIALS.HAY`）。[草船材质](../../../raw/current/scripts/prefabs/boat.lua#L842-L847)

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 锚点重定位 2 处（tuning.lua REPAIR_*_HEALTH 数值与海带时限）；结论修订：无，仅锚点迁移。依据：枢纽 diff 审阅＋变更清单命中 actions.lua、componentactions.lua、tuning.lua，其余 sources 文件未变更。
- 源码审阅：2026-10-21，`steam-buildid:25643519`；完整阅读 `components/boatpatch.lua`，并核验 `prefabs/boatpatch.lua` 两个宿主、`prefabs/sewing_tape.lua`、`prefabs/treegrowthsolution.lua`、`components/boatleak.lua` 的 `Repair`/`SetState`、`components/repairable.lua`、`prefabs/boat.lua` 的 `repairable` 材质、`actions.lua`/`componentactions.lua` 的动作入口与 `tuning.lua` 数值。未审阅海带补丁配方/可食用平衡的完整调用链。
- 游戏运行：未运行；未验证多人同时补同一 leak、海带超时存档恢复、树果酱移除 leak 后的 hullhealth 计数。
- 结构检查：`validate_page.py` 通过（0 问题）；`validate_source_links.py --strict` 通过：本页 32 个 Lua 引用，0 error / 0 warning；本页 Lua 片段经 `parser_probe.exe --syntax-only` 通过（0 错误）。

相关页面：[[reference/components/boatphysics]]、[[reference/components/mast]]、[[reference/components/boatdrag]]。
