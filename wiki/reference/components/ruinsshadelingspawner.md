---
title: "ruinsshadelingspawner"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "洞穴世界组件：在遗迹梦魇区域的空遗迹椅子上生成暗影小妖，被搜刮后进入一天冷却。"
aliases: ["遗迹暗影小妖生成器", "暗影小妖椅子", "ruinsshadelingspawner"]
tags: [component, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/ruinsshadelingspawner.lua", "prefabs/cave.lua", "prefabs/ruinsrelic.lua", "prefabs/smashables.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# ruinsshadelingspawner

## 快速判断

- 职责与适用：挂在世界（TheWorld）上管理至多一只 `ruins_shadeling`：候选是带 sittable 且无人坐的遗迹椅，且椅位必须在 `"Nightmare"` 视觉节点内。
- 优先使用：椅子 prefab 的唤醒钩子里调 `TrySpawnShadeling(chair)`，原版遗迹座椅（[ruinsrelic](../../../raw/current/scripts/prefabs/ruinsrelic.lua#L8-L10)）与可破坏坐垫（[smashables](../../../raw/current/scripts/prefabs/smashables.lua#L73)）都这么接。
- 重要副作用：监听 shadeling 的 `ruins_shadeling_looted`／`onremove` 和椅子的 `onremove`／`becomeunsittable` 四个事件维护状态（[注册](../../../raw/current/scripts/components/ruinsshadelingspawner.lua#L49-L52)）；搜刮后启动一天冷却。
- 修改边界：仅服务端世界组件，一次只维护一只 shadeling；椅子拆除或变不可坐会 Despawn 当前 shadeling。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `TrySpawnShadeling(chair)` | `chair: entity，需 components.sittable` | 无现存 shadeling、无冷却任务、椅未被占、位于 Nightmare 节点四条同时满足才生成；成功后 `sittable:SetOccupier(shadeling)`。[实现](../../../raw/current/scripts/components/ruinsshadelingspawner.lua#L39-L57) |
| `cooldown` | `number`，默认 `TUNING.TOTAL_DAY_TIME` | shadeling 被搜刮后的重生成冷却秒数。[构造](../../../raw/current/scripts/components/ruinsshadelingspawner.lua#L5) |
| `LongUpdate(dt)` | 组件睡眠补时钩子 | 冷却任务按 dt 折算重排，保证睡眠跳时不错过。[实现](../../../raw/current/scripts/components/ruinsshadelingspawner.lua#L59-L65) |
| `shadeling` | `entity 或 nil` | 当前存活的暗影小妖引用；由其 `onremove` 清空（[回调](../../../raw/current/scripts/components/ruinsshadelingspawner.lua#L20-L23)）。 |

## 典型接入

### 自定义遗迹座椅接入小妖生成

- 前提与选择：世界 prefab（cave 类）需先装组件；椅子要有 sittable 组件并位于含 `"Nightmare"` 名的视觉节点内，否则 TrySpawn 静默不生成。
- 生效与撤销：在椅子 OnEntityWake 调 TrySpawnShadeling；搜刮 looted 事件触发后冷却一天，期间新椅子唤醒不再生成。
- 原版依据：cave 世界安装（[cave](../../../raw/current/scripts/prefabs/cave.lua#L571)）；ruinsrelic 仅在世界组件存在时才挂唤醒钩子（[条件安装](../../../raw/current/scripts/prefabs/ruinsrelic.lua#L134-L137)）。

```lua
-- 椅子 prefab 的 master 侧（仿 ruinsrelic）
local function Chair_TrySpawnShadeling(inst)
    if TheWorld.components.ruinsshadelingspawner ~= nil then
        TheWorld.components.ruinsshadelingspawner:TrySpawnShadeling(inst)
    end
end
inst.OnEntityWake = Chair_TrySpawnShadeling
```

- 调整与验证：生成条件含 `chair.components.sittable:IsOccupied()` 检查，接入方无需自行判座；冷却可通过 postinit 改 `cooldown` 字段，仅影响其后触发的搜刮。

## 常见陷阱与限制

- 本组件三个内部回调都直接取 `TheWorld.components.ruinsshadelingspawner`（[实现](../../../raw/current/scripts/components/ruinsshadelingspawner.lua#L13)），把它装在普通实体而非世界上会导致回调取错对象；必须世界级安装。
- 椅子位置不满足 Nightmare 节点时 TrySpawn 返回 nil 且无任何提示，排查时先查 `Map:FindVisualNodeAtPoint` 条件（[实现](../../../raw/current/scripts/components/ruinsshadelingspawner.lua#L46)）。

## 相关链接

- [[reference/components/sittable]]：椅子占位判定与 SetOccupier 协作。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，覆盖全部接口与 cave／ruinsrelic／smashables 消费链；未运行游戏。
- 结构检查：validate_page.py、validate_source_links.py 与 Lua 语法探针通过。
