---
title: "experiencecollector"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "按一个游戏日周期为有技能树的玩家自动累积 1 点技能经验，支持读档续跑与 LongUpdate 补算。"
aliases: ["技能经验累积", "经验收集器", "experiencecollector"]
tags: [component, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/experiencecollector.lua", "prefabs/player_common.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# experiencecollector

## 快速判断

- 职责与适用：给玩家角色周期性发技能 XP；仅当 `skilltreedefs.SKILLTREE_DEFS` 有该 prefab 条目时生效，原版由 player_common 统一安装（[安装](../../../raw/current/scripts/prefabs/player_common.lua#L2859)）。
- 优先使用：为自定义角色接技能树时随原版一起安装即可，周期由 `xp_period` 控制。
- 重要副作用：周期任务挂在 `inst.xpgeneration_task` 字段（非组件私有），重装组件或别人操作该字段会互相干扰（[SetTask](../../../raw/current/scripts/components/experiencecollector.lua#L10-L14)）。
- 修改边界：不存 XP 数值本体，只存任务剩余时间；XP 累积由 skilltreeupdater 的存档负责，组件自身每次仅 `AddSkillXP(1)`（[UpdateXp](../../../raw/current/scripts/components/experiencecollector.lua#L17-L23)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `xp_period` | `number`，默认 `TUNING.TOTAL_DAY_TIME` | 发放周期（一天）；可在安装后改。[构造](../../../raw/current/scripts/components/experiencecollector.lua#L3-L7) |
| `SetTask(time)` | `time: number 或 nil（首响延迟）` | 取消旧任务并重建 `DoPeriodicTask(xp_period, UpdateXp, time)`。[实现](../../../raw/current/scripts/components/experiencecollector.lua#L9-L15) |
| `UpdateXp()` | 无参数 | prefab 无技能树定义时返回 nil 不发放；否则 `skilltreeupdater:AddSkillXP(1)`。[实现](../../../raw/current/scripts/components/experiencecollector.lua#L17-L23) |
| `LongUpdate(dt)` | 组件睡眠补时钩子 | 按 dt 折算错过的整周期逐次 UpdateXp 并重排剩余相位。[实现](../../../raw/current/scripts/components/experiencecollector.lua#L25-L50) |
| `OnSave`／`OnLoad(data)` | 存读钩子 | 只存 `time`（任务剩余时间），载入后以该相位重建任务。[实现](../../../raw/current/scripts/components/experiencecollector.lua#L52-L63) |

## 典型接入

### 自定义角色接技能树经验

- 前提与选择：在 `skilltree_defs` 注册角色树并在 player_common 逻辑内安装组件（原版对所有玩家统一安装，无树定义时静默不发）。
- 生效与撤销：组件构造即建任务；发放频率由 `xp_period` 决定，改周期后需再调 `SetTask` 重排。
- 原版依据：player_common 安装（[player_common](../../../raw/current/scripts/prefabs/player_common.lua#L2859)）；技能树定义检查（[UpdateXp](../../../raw/current/scripts/components/experiencecollector.lua#L19-L21)）。

```lua
-- 角色 prefab 的 master_postinit 内（与 player_common 一致的安装方式）
inst:AddComponent("experiencecollector")
-- 可选：改成半天一发
inst.components.experiencecollector.xp_period = TUNING.TOTAL_DAY_TIME / 2
inst.components.experiencecollector:SetTask()
```

- 调整与验证：UpdateXp 每次固定 AddSkillXP(1)，没有数量配置；想改单次数值需 postinit 包 UpdateXp 或直接操作 skilltreeupdater。

## 常见陷阱与限制

- UpdateXp 无 skilltreeupdater 组件保护：prefab 在 `SKILLTREE_DEFS` 有条目但没装 skilltreeupdater 时会直接报错（[实现](../../../raw/current/scripts/components/experiencecollector.lua#L22)），自定义角色需保证两件套齐全。
- OnLoad 依赖任务存在（OnSave 调 `GetTaskRemaining(self.inst.xpgeneration_task)`，[实现](../../../raw/current/scripts/components/experiencecollector.lua#L55)），postinit 中不要手动 Cancel 该任务后不重建。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，覆盖全部接口与 player_common 安装点、TUNING 取值；未运行游戏。
- 结构检查：validate_page.py、validate_source_links.py 与 Lua 语法探针通过。
