---
title: "lunarriftmutationsmanager"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "世界级月裂突变任务管理器：登记三种突变 Boss 的击杀、驱动 Wagstaff NPC 出场与奖励完成态，供猎人波次和竞技场流程查询。"
aliases: ["月裂突变管理", "突变Boss击杀登记", "lunarriftmutationsmanager"]
tags: [component, lifecycle, event]
dst_build: "steam-buildid:26230496"
size_class: standard
sources: ["components/lunarriftmutationsmanager.lua", "prefabs/forest.lua", "prefabs/warg.lua", "components/hunter.lua", "prefabs/wagstaff_npc.lua", "components/wagpunk_arena_manager.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# lunarriftmutationsmanager

## 快速判断

- 职责与适用：挂在 TheWorld 上的任务进度组件（仅 forest 世界，prefabs/forest.lua:712）。登记 `mutatedwarg`／`mutatedbearger`／`mutateddeerclops` 的击杀（components/lunarriftmutationsmanager.lua:5-10），安排 Wagstaff NPC（`wagstaff_npc_mutations`）出场，并给出任务完成态。管理器本身不改写 Boss 数值（文件头注记 lines 1-2）。
- 优先使用：查询突变任务进度用 `IsTaskCompleted`／`ShouldGiveReward`／`GetNumDefeatedMutations`（lines 113-119、41-43）；Wagstaff NPC 持有引用后驱动对话（prefabs/wagstaff_npc.lua:1048、899-919）。
- 重要副作用：达成时向 TheWorld 推送 `ms_lunarriftmutationsmanager_taskcompleted`（lines 123、150），竞技场管理器监听后推进 SparkArk（components/wagpunk_arena_manager.lua:1853）；击杀未讨伐的突变体时会将其 `health.destroytime` 延长到 15 秒（line 87）。
- 修改边界：仅 mastersim（构造断言 line 17）；随 TheWorld 保存／加载（lines 130-152）；`TryMutate` 已标注不再使用（lines 53-65）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `_MUTATIONS_NAMES`／`_MUTATIONS` | `table`（顺序固定数组／反向映射） | 供模组读取；顺序不可改（lines 4-10、23-24） |
| `SetMutationDefeated(ent)` | `(ent)`，按 prefab 去重登记 | 由突变 Boss 死亡处理调用（prefabs/warg.lua:575-576、bearger.lua:404-405、deerclops.lua:302-303）；若 wagboss 已讨伐直接发奖，否则安排 Wagstaff（lines 75-92，其中 line 81 检查 `wagboss_tracker`） |
| `TriggerWagstaffAppearance(ent)` | `(ent)` | 在击杀点附近可走偏移（≤8 距离）生成 `wagstaff_npc_mutations` 并让 NPC 讲解（lines 94-111） |
| `HasDefeatedThisMutation(prefab)` | `(prefab) -> bool` | 猎人波次用它决定是否替换出突变猎犬（components/hunter.lua:47） |
| `ShouldGiveReward()`／`IsTaskCompleted()` | `-> bool` | `task_completed` 或三只全部讨伐；Wagstaff 对话与奖励分支使用（lines 113-119；wagstaff_npc.lua:899、912、952） |
| `OnRewardGiven()` | 置位并发事件 | `task_completed = true`，推 `ms_lunarriftmutationsmanager_taskcompleted`，全部讨伐后清表重置（lines 121-128） |
| `GetNumDefeatedMutations()`／`HasDefeatedAllMutations()` | `-> number`／`-> bool` | 进度统计；上限 `num_mutations = 3`（lines 41-47、34） |
| `RefreshDefeatedMutationsTable()` | 清空 wagstaff 与进度 | 重开一轮任务（lines 29-35） |
| `OnSave`／`OnLoad` | 存 `defeated_mutations`、`task_completed` | OnLoad 置位后同样补推事件（lines 130-152） |
| `TryMutate(ent, corpseprefab)` | 已废弃 | 源码注记 "Not used anymore"（lines 53-65）；`_CanCorpseMutate` 走 `CanLunarRiftMutateFromCorpse`（components/componentutil.lua:1525） |

## 典型接入

### 查询突变任务进度（服务端模组）

- 前提与选择：组件只存在于 forest 世界的 mastersim 端；直接读 `TheWorld.components.lunarriftmutationsmanager`，配合 `_MUTATIONS_NAMES` 可枚举目标 prefab，不依赖任何客户端同步。
- 生效与撤销：只读查询无副作用；事件 `ms_lunarriftmutationsmanager_taskcompleted` 在当局达成与存档载入完成态时各推一次（lines 123、150），监听端需可重复触发。
- 原版依据：查询路径同 wagstaff_npc.lua:1048、wagpunk_arena_manager.lua:1853。

```lua
local function PrintMutationProgress()
    local mgr = TheWorld.components.lunarriftmutationsmanager
    if mgr ~= nil then
        print("defeated:", mgr:GetNumDefeatedMutations(), "done:", mgr:IsTaskCompleted())
    end
end

AddPrefabPostInit("world", function(inst)
    if not inst.ismastersim then
        return
    end
    inst:ListenForEvent("ms_lunarriftmutationsmanager_taskcompleted", function()
        print("lunar mutations quest completed")
    end)
end)
```

- 调整与验证：`_MUTATIONS_NAMES` 仅供读取，改动数组顺序会破坏 `MUTATIONS` 反向映射与存档索引（lines 4-12）。

## 常见陷阱与限制

- `TryMutate` 保留但已废弃（lines 53-65），新代码不要依赖；尸体变异实际由各 Boss prefab 的死亡链路自行处理。
- 击杀登记只认三种固定 prefab 名（lines 67-73 的 `MUTATIONS[prefab]` 检查），改名的自定义突变体不会被计入。
- 组件只挂在 forest（地表）世界上；洞穴世界没有该组件，`TheWorld.components.lunarriftmutationsmanager` 在洞穴为 nil，消费方均先判空（prefabs/warg.lua:575）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。通读 components/lunarriftmutationsmanager.lua 全文；核验 forest.lua:712 挂载、warg/bearger/deerclops 死亡登记、hunter.lua:47 波次查询、wagstaff_npc.lua:1048 引用与对话分支、wagpunk_arena_manager.lua:1853 事件监听。未运行游戏。
- 结构检查：validate_page.py / validate_source_links.py 通过（见批级自验）。

相关：[[reference/components/wagboss_tracker]]、[[reference/components/moonstorms]]
