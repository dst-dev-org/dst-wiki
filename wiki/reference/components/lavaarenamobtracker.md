---
title: "lavaarenamobtracker"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "熔炉怪物登记组件：在熔炉世界实体上维护怪物集合，支持双端登记、按标签与半径查找；本快照仅 StartTracking 与 ForEachMob 被使用。"
aliases: ["熔炉怪物追踪", "怪物登记表", "lavaarenamobtracker"]
tags: [component, creature]
dst_build: "steam-buildid:26230496"
sources: ["components/lavaarenamobtracker.lua", "prefabs/lavaarena.lua", "prefabs/lavaarena_boarrior.lua", "prefabs/lavaarena_peghook.lua", "prefabs/lavaarena_battlestandard.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# lavaarenamobtracker

## 快速判断

- 职责与适用：挂在熔炉世界 prefab 上（[挂接](../../../raw/current/scripts/prefabs/lavaarena.lua#L124)），文件头注明双端使用（[注释](../../../raw/current/scripts/components/lavaarenamobtracker.lua#L1)）；作为"场上怪物"的登记表，供熔炉玩法统计与范围效果查询。
- 优先使用：登记用 `StartTracking(ent)`（怪物生成时调用，如 [boarrior](../../../raw/current/scripts/prefabs/lavaarena_boarrior.lua#L53-L54)、[peghook](../../../raw/current/scripts/prefabs/lavaarena_peghook.lua#L68-L69)）；遍历用 `ForEachMob`（战旗脉冲效果，[battlestandard.lua:38](../../../raw/current/scripts/prefabs/lavaarena_battlestandard.lua#L38）。
- 重要副作用：登记时监听实体 `onremove` 自动注销，避免悬引用（[实现](../../../raw/current/scripts/components/lavaarenamobtracker.lua#L11-L23)）。
- 修改边界：集合无持久化（OnSave 缺失），重载后由各怪物重新登记。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `StartTracking(ent)` / `StopTracking(ent)` | `ent: entity -> 无` | 以实体为键的集合登记／注销，重复登记幂等（[实现](../../../raw/current/scripts/components/lavaarenamobtracker.lua#L11-L23)）。 |
| `GetAllMobs()` | `-> table`（实体数组） | 返回登记实体副本；本快照无调用者（[实现](../../../raw/current/scripts/components/lavaarenamobtracker.lua#L29-L35)）。 |
| `FindMobs(x, y, z, r, musttags, canttags, mustoneoftags)` | `-> table` | 球形半径（先平方）过滤加三组标签过滤，结果按距离升序；`y` 参数被忽略，按 `(x, 0, z)` 平面距离计算（[实现](../../../raw/current/scripts/components/lavaarenamobtracker.lua#L37-L82)）。 |
| `ForEachMob(cb, params)` | `cb(ent, params) -> 无` | 遍历登记实体（[实现](../../../raw/current/scripts/components/lavaarenamobtracker.lua#L84-L88)）。 |
| `GetNumMobs()` | `-> number` | 返回 `self.count`；**该字段从未被更新**，恒为 0（[实现](../../../raw/current/scripts/components/lavaarenamobtracker.lua#L25-L27)、[初始化](../../../raw/current/scripts/components/lavaarenamobtracker.lua#L6)）。 |

## 典型接入

### 熔炉登记与遍历（原版参照）

- 前提与选择：世界端挂组件后，怪物在 master 初始化时向 `TheWorld.components.lavaarenamobtracker` 自登记（([boarrior.lua:53-54](../../../raw/current/scripts/prefabs/lavaarena_boarrior.lua#L53-L54)）；效果组件按需遍历（[battlestandard.lua:38](../../../raw/current/scripts/prefabs/lavaarena_battlestandard.lua#L38）。

```lua
-- 登记（怪物 master postinit）与遍历（效果组件）
TheWorld.components.lavaarenamobtracker:StartTracking(inst)

TheWorld.components.lavaarenamobtracker:ForEachMob(function(ent, params)
    if ent:IsValid() then
        -- 对每个登记怪物施加 params 效果
    end
end, { power = 1 })
```

- 调整与验证：登记不分端（组件注释声明双端使用），客户端也可登记本地视角需要的实体。

## 常见陷阱与限制

- **GetNumMobs 恒为 0**：`count` 只在构造置 0，登记／注销都不维护它；统计数量应 `#GetAllMobs()` 或自行计数（[证据](../../../raw/current/scripts/components/lavaarenamobtracker.lua#L5-L27)）。
- **FindMobs／GetAllMobs 无原版调用者**：本快照内仅定义未被调用（全库 grep），接口按实现记录、未经运行验证；`y` 参数同样未生效。
- **无存档**：登记集合不持久化，依赖怪物侧在加载后重新 StartTracking（各 lavaarena 怪物 prefab 内调用）。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；覆盖组件全文、世界挂接、两个登记调用点与战旗遍历调用点；FindMobs/GetAllMobs/GetNumMobs 为无调用者状态。
- 结构检查：本批 validate_page、validate_source_links 及示例 lua 语法探测（见批级记录）。
- 游戏运行：未运行。
