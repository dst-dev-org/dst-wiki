---
title: "gestaltcage"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "织影牢笼工具组件：瞄准 gestaltcapturable 目标并触发捕获，成功后移除本体并生成对应等级的填充牢笼。"
aliases: ["gestaltcage", "织影牢笼", "gestalt捕获", "牢笼工具"]
tags: [component, item]
dst_build: "steam-buildid:26230496"
size_class: simple
sources: ["components/gestaltcage.lua", "prefabs/gestalt_cage.lua", "actions.lua", "components/gestaltcapturable.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# gestaltcage

## 快速判断

- 职责与适用：装在手持牢笼物品上；管理对 gestaltcapturable 目标的瞄准状态（OnTarget／OnUntarget）与捕获（Capture）。([gestaltcage.lua:40-58](../../../raw/current/scripts/components/gestaltcage.lua#L40-L58)
- 优先使用：捕获走原版 `POUNCECAPTURE` 动作链（动作 → 组件 Capture），不要绕过动作直接调；瞄准由玩家状态图在攻击流程调 OnTarget／OnUntarget。([actions.lua:6580-6589](../../../raw/current/scripts/actions.lua#L6580-L6589)
- 重要副作用：Capture 成功即 `self.inst:Remove()` 本体，并在"最外层持有者"位置生成 `gestalt_cage_filled<level>`；目标先收到 `OnCaptured`。([gestaltcage.lua:19-37](../../../raw/current/scripts/components/gestaltcage.lua#L19-L37)
- 修改边界：未启用 gestaltcapturable、目标失效或 doer 距目标超过 1 时返回 `false, "MISSED"`，本体保留。([gestaltcage.lua:11-17](../../../raw/current/scripts/components/gestaltcage.lua#L11-L17)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `Capture(target, doer)` | 布尔，失败附 "MISSED" | 读目标 `GetLevel()`／`GetIsPlanar()`，回调 OnCaptured，移除本体后按 root 位置与朝向生成 `gestalt_cage_filled<level>` 并 StartCapture。([gestaltcage.lua:10-38](../../../raw/current/scripts/components/gestaltcage.lua#L10-L38) |
| `OnTarget(target)` / `OnUntarget(target)` | 实体或 nil | 维护 `self.target`，回调目标 OnTargeted／OnUntargeted（目标侧维护 targeters 表）。([gestaltcage.lua:40-58](../../../raw/current/scripts/components/gestaltcage.lua#L40-L58) ([gestaltcapturable.lua:68-96](../../../raw/current/scripts/components/gestaltcapturable.lua#L68-L96) |
| `OnRemoveFromEntity` | 内部钩子 | 移除组件时自动 OnUntarget 清理目标侧状态。([gestaltcage.lua:6-8](../../../raw/current/scripts/components/gestaltcage.lua#L6-L8) |
| （依赖）`gestaltcapturable` | 目标组件 | IsEnabled／GetLevel／GetIsPlanar／OnCaptured；捕获成功对 doer 推 `gestaltcaptured`。([gestaltcapturable.lua:99-106](../../../raw/current/scripts/components/gestaltcapturable.lua#L99-L106) |

## 典型接入

### 给自定义牢笼类工具接入捕获

- 前提与选择：工具 prefab 需可被手持（inventoryitem）并注册进原版动作可达范围；目标 prefab 需装 gestaltcapturable。([gestalt_cage.lua:84-93](../../../raw/current/scripts/prefabs/gestalt_cage.lua#L84-L93)

```lua
inst:AddComponent("inventoryitem")
inst:AddComponent("gestaltcage")
```

- 生效与撤销：捕获生成物 prefab 名按目标等级动态拼接，须确保对应 `gestalt_cage_filled<level>` 存在；本体移除后组件随实体销毁。([gestaltcage.lua:29-35](../../../raw/current/scripts/components/gestaltcage.lua#L29-L35)

## 常见陷阱与限制

- Capture 的位置取 `inventoryitem:GetGrandOwner()`（在最外层容器／持有者处落地），不是 doer 位置；无 inventoryitem 时才用本体位置。([gestaltcage.lua:23-26](../../../raw/current/scripts/components/gestaltcage.lua#L23-L26)
- 瞄准状态只在玩家状态图流程中维护；直接 Remove 目标后旧引用由 OnUntarget 的 IsValid 守卫兜底。([gestaltcage.lua:54-56](../../../raw/current/scripts/components/gestaltcage.lua#L54-L56)
- 生成的填充牢笼经 `SetIsPlanar`（若存在）继承目标的位面属性，需牢笼 prefab 支持该接口。([gestaltcage.lua:32-34](../../../raw/current/scripts/components/gestaltcage.lua#L32-L34)

## 验证记录

- 源码审阅：2026-10-28，`steam-buildid:26230496`：审阅组件全文、牢笼 prefab 安装、POUNCECAPTURE 动作函数与 gestaltcapturable 三个回调。
- 游戏运行：未运行；未复现完整捕获流程。
- 结构检查：validate_page.py 通过；validate_source_links.py --strict 通过（本批 16 页合并执行）。
