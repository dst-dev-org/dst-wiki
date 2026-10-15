---
title: "penguinspawner"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "冬季企鹅族群世界管理器：冬季在玩家附近近水陆地选址建立族群（含冰面与冰岩），非冬季清场，月岛区域生成变异企鹅。"
aliases: ["企鹅生成", "企鹅族群", "penguinspawner 组件"]
tags: [component, ai]
dst_build: "steam-buildid:26230496"
size_class: standard
sources: ["components/penguinspawner.lua", "prefabs/forest.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# penguinspawner

## 快速判断

- 职责与适用：forest 世界主机组件（:14 断言；prefabs/forest.lua:651）；仅在 `iswinter` 且冬季剩余 >3 天时尝试建群（:328）。
- 优先使用：被动运行无需调用；控制冰岩数量 → 世界事件 `ms_setpenguinnumboulders`（:461，注意该处理引用未定义函数，见陷阱）。
- 重要副作用：每群生成 `penguin_ice` 冰面与 3~7 块 `rock_ice`（remove_on_dryup，:281-309）；企鹅记 rookery/home 到 knownlocations（:485-486）。
- 修改边界：`seasontick` 非冬季即清空族群、移除冰面并归零计数（:431-448）；群上限 `PENGUINS_MAX_COLONIES`(5)+buffer(1)，全场企鹅上限 = 群均规模 × (5+1)（:32-34）。

## 关键接口

| 接口/字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `AddToColony` | `(colonyNum, pengu) -> nil` | 注册成员、计数、监听 death/onremove、记巢点并刷新变异状态（:471-490）。 |
| `SpawnModeNever/Light/Normal/Med/Heavy` | `() -> nil` | 全部为空的 deprecated 桩（:492-510），调用无效果。 |
| `OnSave`/`OnLoad` | 常规存取 | 只存族群 rookery 坐标、is_mutated 与 numspawned（:515-531）；载入重建 `penguin_ice`，成员表清空待回归（:391-408）。 |
| 选址（局部） | `FindSpawnLocationForPlayer`/`EstablishColony` | 玩家 6 半径找非地面点再找邻水陆地并验 Pathfinder 通达（:43-99）；新群 50→30 半径找位：距水、距建筑 20（`PENGUINS_MIN_DIST_FROM_STRUCTURES`）、距他群 60（:200-276）。 |
| 轮询（局部） | `TryToSpawnFlock` | 每 5s 一轮、在玩家间轮转（round-robin，:380-389）；全局冷却 `PENGUINS_SPAWN_INTERVAL`=30s（:356-359）、距上次落点 `MIN_SPAWN_DIST`=相机视距（:349-354）。 |

## 关键流程：一次建群

1. 门禁：冬季剩余 >3 天、群数 ≤ 上限、鸟数未满、冷却结束、玩家不在旧落点附近（:326-359）。
2. 选址：玩家周边 6 半径找水面，再在水面 2 半径内找可走陆地（:70-99）。
3. 建群：能并入 50+√60 范围内未满员（<12-群规模×0.8）且路径通达的旧群则复用（:253-266）；否则新建——月岛判定 `Map:IsInLunacyArea` + `TUNING.SPAWN_MOON_PENGULLS` 决定变异群（:280，tuning.lua:9084），生成冰面与按扇区散布的冰岩（:278-316）。
4. 生成：群规模 `PENGUINS_FLOCK_SIZE`=9 ±3，逐只延迟 1±1s 生成（:178-198）；变异群生成 `mutated_penguin`（:162-165）。
5. 变异状态：群内 `mutated_penguin` 过半即整群标记变异并换小地图图标（:119-140）。

## 典型接入

### 理解/调整企鹅生成参数

- 前提与选择：全部数值来自 TUNING（tuning.lua:6022-6028），mod 覆盖 TUNING 表即可调；组件本身无公开配置接口。
- 生效与撤销：`SpawnMode*` 系列已废弃，不要指望它们改密度；实际门禁在 `TryToSpawnFlockForPlayer`（:326-359）。
- 原版依据：prefabs/forest.lua:651。

```lua
-- 只读示例：查询企鹅压力（调试）
local function GetPenguinPressure()
    local mgr = TheWorld.components.penguinspawner
    if mgr ~= nil then
        print(mgr:GetDebugString())
    end
end
```

- 调整与验证：`GetDebugString` 输出当前鸟数/上限、群数与下次生成倒计时（:537-555）。

## 常见陷阱与限制

- `ms_setpenguinnumboulders` 处理引用未定义的 `OnSetNumBoulders`（:461，全文件无定义也无全局），推该事件会因调用 nil 报错——mod 勿用此事件调冰岩数。
- 族群成员不落存档：OnSave 只存 rookery，企鹅按自身 knownlocations 回巢，成员关系载入后靠后续 AddToColony 重建（:520、:391-408）。
- 选址强依赖 Pathfinder：`Pathfinder:IsClear` 不通即放弃（:222-228、:259-261），导航网格异常的世界可能建不出群。
- 非冬季清场是硬清空：`_colonies = {}` 且 `_totalBirds = 0`（:444-447），冬季回归后重新按新地点建群，不会复用旧冰面（已 QueueRemove）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，通读组件全部 560 行；挂载 forest.lua:651、TUNING tuning.lua:6022-6028/:9084、SEASONS 全局 constants.lua:1611-1617 逐行核对。
- 游戏运行：未运行。

## 相关链接

- [[reference/components/seasons]]
- [[reference/components/mutatedbirdmanager]]
