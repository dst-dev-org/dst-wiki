---
title: "lunaralterguardianspawner"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "世界级月裂守卫生成器：从发起者位置搜索可行走点后延时生成 alterguardian_phase1_lunarrift，并跨存档跟踪在世守卫。"
aliases: ["月裂守卫生成器", "lunaralterguardianspawner"]
tags: [component]
dst_build: "steam-buildid:26230496"
sources: ["components/lunaralterguardianspawner.lua", "components/wagpunk_arena_manager.lua", "prefabs/forest.lua", "brains/wagstaff_npcbrain.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# lunaralterguardianspawner

## 快速判断

- 职责与适用：挂在 TheWorld 的世界组件（仅主机，构造即断言 [L7](../../../raw/current/scripts/components/lunaralterguardianspawner.lua#L7)），从发起者位置找可行走生成点并延时生成月裂守卫 `alterguardian_phase1_lunarrift`；原版由 forest.lua 注册（[L735](../../../raw/current/scripts/prefabs/forest.lua#L735)）。
- 优先使用：Wagpunk 竞技场装配完成时 `TrySpawnLunarGuardian`（[调用点](../../../raw/current/scripts/components/wagpunk_arena_manager.lua#L822-L823)）；查询在世守卫用 `GetGuardian`（[wagstaff 大脑](../../../raw/current/scripts/brains/wagstaff_npcbrain.lua#L246)）。
- 重要副作用：生成后跟踪守卫并在其 `onremove` 时清空记录；同一时间最多一个在世守卫或一个待生成点。
- 修改边界：纯主机端；延迟由发起者 prefab 决定（`wagstaff_npc_wagpunk_arena` 14 秒、其余 4 秒，[L72-L81](../../../raw/current/scripts/components/lunaralterguardianspawner.lua#L72-L81)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `TrySpawnLunarGuardian(spawner)` | `entity -> void` | 无在世守卫且无待生成点才生效；以发起者位置搜点，失败则放弃；成功记录 `guardiancomingpt` 并延时生成。[实现](../../../raw/current/scripts/components/lunaralterguardianspawner.lua#L72-L81) |
| `KickOffSpawn(delay)` | `number -> void` | 延时生成守卫、传送到待生成点、进入 `spawn_lunar` 状态并清空 `guardiancomingpt`；监听守卫 `onremove` 清 `_activeguardian`。[实现](../../../raw/current/scripts/components/lunaralterguardianspawner.lua#L58-L70) |
| `GetGuardian()` | `-> entity\|nil` | 当前在世守卫。[实现](../../../raw/current/scripts/components/lunaralterguardianspawner.lua#L50-L52) |
| `HasGuardianOrIsPending()` | `-> boolean` | 守卫在世或 `guardiancomingpt` 未清；竞技场管理与 Wagstaff 状态机都用它防重复。[实现](../../../raw/current/scripts/components/lunaralterguardianspawner.lua#L54-L56) |
| `OnSave/OnLoad/LoadPostPass` | 存读钩子 | 保存守卫 GUID 与待生成点 x/z；加载后重建守卫引用（重挂 `onremove`）或续延 4 秒生成。[存](../../../raw/current/scripts/components/lunaralterguardianspawner.lua#L87-L98)、[载](../../../raw/current/scripts/components/lunaralterguardianspawner.lua#L100-L118) |

## 典型接入

### 在自定义事件后生成一个月裂守卫

- 前提与选择：主机端世界组件已由 forest.lua 注册，直接调用即可复用原版找点与延时逻辑；不需要自建 prefab。
- 生效与撤销：生成点在 10 距离内用 `FindWalkableOffset` 搜索，锚点不在地面时先 `FindNearbyLand` 纠偏（[L33-L44](../../../raw/current/scripts/components/lunaralterguardianspawner.lua#L33-L44)）；守卫移除自动清跟踪，存档重载可恢复。

```lua
-- 主机端：以某实体为锚点请求生成月裂守卫
local spawner = TheWorld.components.lunaralterguardianspawner
if spawner ~= nil and not spawner:HasGuardianOrIsPending() then
    spawner:TrySpawnLunarGuardian(some_entity)
end
```

- 调整与验证：`spawner.prefab == "wagstaff_npc_wagpunk_arena"` 分支为 14 秒，其余 4 秒；无法自定义延迟，除非改动本组件或后处理生成实体。

## 常见陷阱与限制

- **同一时间单守卫**：`_activeguardian` 非空或 `guardiancomingpt` 未清时调用直接返回（[L72-L73](../../../raw/current/scripts/components/lunaralterguardianspawner.lua#L72-L73)），并发请求不会生成第二个。
- **找点失败无重试**：锚点周围找不到可行走点时本次放弃且不排队，需调用方自行重试。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；组件全文，及 wagpunk_arena_manager、forest.lua、wagstaff_npcbrain 的调用点。
- 游戏运行：未运行。
