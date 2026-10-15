---
title: "friendlevels"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "任务式好感等级组件：完成友好任务升等级并入队奖励（按等级或默认），事件通知宿主发放礼物，含顺序敏感的存读。"
aliases: ["好感等级", "友谊任务", "friendlevels"]
tags: [component, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/friendlevels.lua", "prefabs/hermitcrab.lua", "brains/hermitcrabbrain.lua", "components/wagpunk_arena_manager.lua", "components/retrofitforestmap_anr.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# friendlevels

## 快速判断

- 职责与适用：给 NPC 维护"任务清单 + 等级 + 待发奖励"的好感状态机；数据结构与奖励发放时机都由宿主 prefab 注入。[构造](../../../raw/current/scripts/components/friendlevels.lua#L1-L12)
- 优先使用：完成用 `CompleteTask(task, doer)`／`CompleteAllTasks(doer)`，发放用 `DoRewards(target)`（取空队列），查询 `GetLevel()`／`GetMaxLevel()`。[CompleteTask](../../../raw/current/scripts/components/friendlevels.lua#L56-L75)
- 重要副作用：升级推 `friend_level_changed`、每次完成推 `friend_task_complete`（数据为是否走默认奖励）；读档若恢复出等级/队列会补推同款事件。[L74](../../../raw/current/scripts/components/friendlevels.lua#L74)、[OnLoad](../../../raw/current/scripts/components/friendlevels.lua#L99-L109)
- 修改边界：任务表由宿主定义（hermitcrab 用 1..18 的枚举索引）；组件不判断"怎么算完成任务"，触发时机全在宿主事件里。[宿主任务表](../../../raw/current/scripts/prefabs/hermitcrab.lua#L1137-L1157)

## 关键接口

| 接口／字段 | 已核验类型／单位 | 条件、作用与证据 |
|---|---|---|
| `SetFriendlyTasks(data)` | 表 `{task = {complete, onetime, ...}}` | 任务条目可带 `onetime`（不重复给默认奖励）、`completestrings` 等，组件只读写 `complete`。[L31-L33](../../../raw/current/scripts/components/friendlevels.lua#L31-L33)、[宿主定义](../../../raw/current/scripts/prefabs/hermitcrab.lua#L1137-L1157) |
| `SetLevelRewards(data)` | 数组 `[{fn(inst, target, task) → gifts}]` | 索引即等级，`GetMaxLevel()` 为数组长度（hermitcrab 10 级）。[L27-L29](../../../raw/current/scripts/components/friendlevels.lua#L27-L29)、[hermitcrab 表](../../../raw/current/scripts/prefabs/hermitcrab.lua#L987-L998) |
| `SetDefaultRewards(fn)` | `fn(inst, target, task)` | 非升级任务的奖励来源；未设置时默认任务不入队。[L23-L25](../../../raw/current/scripts/components/friendlevels.lua#L23-L25)、[L63-L67](../../../raw/current/scripts/components/friendlevels.lua#L63-L67) |
| `CompleteTask(task, doer)` | 无返回 | 任务未完成且未满级 → 升级入队并推 `friend_level_changed`；否则（或任务可重复时）按默认奖励入队；条目标记 `complete` 并推 `friend_task_complete`。[L56-L75](../../../raw/current/scripts/components/friendlevels.lua#L56-L75) |
| `DoRewards(target)` | gifts 表 | 逐条入队项调等级/默认奖励 fn 拼接；宿主塞的 `specifictaskreward`（prefab 名数组）逐个 SpawnPrefab 后清空。[L35-L54](../../../raw/current/scripts/components/friendlevels.lua#L35-L54) |
| `GetLevel()`／`GetMaxLevel()` | 数值 | brain 按等级分段选台词、控制是否接受交易。[L83-L89](../../../raw/current/scripts/components/friendlevels.lua#L83-L89)、[brain](../../../raw/current/scripts/brains/hermitcrabbrain.lua#L409) |
| `Enable(enabled)` | 无返回 | 仅赋值并调 `self:SetIsInMood(...)`——该方法在组件内未定义，原版无调用者；直接调用会报错（见陷阱）。[L18-L21](../../../raw/current/scripts/components/friendlevels.lua#L18-L21) |

## 存读流程

- OnSave：`{enabled, level, taskscomplete = [{complete}], queuedrewards}`；`taskscomplete` 按 `ipairs(friendlytasks)` 顺序导出。[OnSave](../../../raw/current/scripts/components/friendlevels.lua#L91-L97)
- OnLoad：恢复 `enabled`／`level`／队列并按状态补推事件；任务完成标记不在这一步。[OnLoad](../../../raw/current/scripts/components/friendlevels.lua#L99-L109)
- LoadPostPass：按索引把 `taskscomplete` 回填进任务条目——索引对应关系即 hermitcrab.lua:49 TODO 抱怨的顺序敏感点，改任务表顺序会错位存档。[LoadPostPass](../../../raw/current/scripts/components/friendlevels.lua#L111-L119)

## 典型接入

### 原版宿主：隐士蟹

- 挂载与注入：`AddComponent("friendlevels")` 后注入默认奖励、等级奖励表、任务表并挂 `friend_task_complete` 监听。[挂载](../../../raw/current/scripts/prefabs/hermitcrab.lua#L1983-L1988)
- 触发：修屋/送礼/种植等事件里按 `TASKS.*` 调 `CompleteTask`；珍珠（Pearl）赠礼、背景音乐分段等全部读 `level` 与任务完成态。[触发样例](../../../raw/current/scripts/prefabs/hermitcrab.lua#L1163-L1167)
- 外部批量完成：wagpunk 竞技场结算调 `CompleteAllTasks(doer)`；森林地图 retrofit 会按旧档倒扣等级并补完任务。[竞技场](../../../raw/current/scripts/components/wagpunk_arena_manager.lua#L1925)、[retrofit](../../../raw/current/scripts/components/retrofitforestmap_anr.lua#L1784-L1795)

### 模组：给 NPC 加好感任务

- 前提与选择：服务端 NPC；自己定义任务表与奖励 fn，完成时机由模组事件决定；组件不提供 UI。
- 生效与撤销：`CompleteTask` 对未知 task 会在判 `friendlytasks[task].complete` 时直接索引 nil 报错，先保证任务表已注入。

```lua
-- 服务端 NPC prefab master 段
inst:AddComponent("friendlevels")
inst.components.friendlevels:SetFriendlyTasks({ [1] = { completestrings = "ANNOUNCE_GIFT" } })
inst.components.friendlevels:SetLevelRewards({ function(inst, target, task) return { "goldnugget" } end })
inst.components.friendlevels:SetDefaultRewards(function(inst, target, task) return { "cutgrass" } end)
-- 玩家送礼等时机：
inst.components.friendlevels:CompleteTask(1, doer)
local gifts = inst.components.friendlevels:DoRewards(doer) -- { "goldnugget" } 或 { "cutgrass" }
```

- 调整与验证：示例仅语法检查未运行；奖励 fn 必须返回表（组件用 `ConcatArrays` 拼接），返回 nil 会报错。

## 常见陷阱与限制

- `Enable()` 调用了不存在的 `SetIsInMood`（mood.lua 的方法，本组件未定义也未混入），当前任何调用都会抛错；原版零调用者，等价于死接口。[L20](../../../raw/current/scripts/components/friendlevels.lua#L20)
- 存档顺序敏感：`taskscomplete` 是纯数组，宿主任务表插入/删除任务会使旧档完成标记整体错位（hermitcrab 注释明确要求按枚举序维护）。[TODO 注释](../../../raw/current/scripts/prefabs/hermitcrab.lua#L49)、[LoadPostPass](../../../raw/current/scripts/components/friendlevels.lua#L111-L119)
- 满级后任务仍会标 `complete` 并推 `friend_task_complete`，但不再升级入队等级奖励；想让满级后任务有产出需靠 `defaultrewards`。[L59-L67](../../../raw/current/scripts/components/friendlevels.lua#L59-L67)
- 奖励队列跨天保留直到 `DoRewards`；`OnLoad` 恢复队列即补推 `friend_task_complete`，宿主监听需幂等。[L103-L105](../../../raw/current/scripts/components/friendlevels.lua#L103-L105)

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。覆盖 friendlevels.lua 全文（含三个存读钩与 Enable）；核对 hermitcrab 的注入/触发/事件消费、brain 读取、wagpunk_arena_manager 与 retrofitforestmap_anr 两处外部调用；未覆盖 hermitcrab 各奖励 fn 的产出清单。
- 游戏运行：未运行；本页结论均来自源码静态核验。
- 结构检查：`validate_page.py`、`validate_source_links.py --strict` 与示例 Lua 语法检查通过（见批记录）。
