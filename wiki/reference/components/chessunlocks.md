---
title: "chessunlocks"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "世界级棋子解锁登记：初始锁定各棋子草图与关联饰品，雕像修复等事件按棋子解锁，供战利品与草图掉落过滤。"
aliases: ["棋子解锁", "chessunlocks"]
tags: [component, worldgen, loot]
dst_build: "steam-buildid:26230496"
sources: ["components/chessunlocks.lua", "prefabs/trinkets.lua", "prefabs/tumbleweed.lua", "prefabs/forest.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# chessunlocks

## 快速判断

- 职责与适用：挂在 TheWorld 的主机世界组件（构造断言 [L7](../../../raw/current/scripts/components/chessunlocks.lua#L7)）；维护"棋子 → 草图/饰品"锁定表，监听 `ms_unlockchesspiece` 逐个解锁（forest 与 cave 都注册，[forest](../../../raw/current/scripts/prefabs/forest.lua#L669)）。
- 优先使用：需要"未解锁内容不进掉落池"的过滤逻辑直接查 `IsLocked`；原版消费者是随机饰品（[trinkets](../../../raw/current/scripts/prefabs/trinkets.lua#L3-L14)）与风滚草战利品（[tumbleweed](../../../raw/current/scripts/prefabs/tumbleweed.lua#L204-L210)）。
- 重要副作用：解锁事件只处理表内棋子（pawn/bishop/rook/knight/muse/formal），bishop/rook/knight 各关联两个 trinket（[L13-L21](../../../raw/current/scripts/components/chessunlocks.lua#L13-L21)）。
- 修改边界：解锁状态随世界存档持久化（OnSave/OnLoad）；事件由雕像/雕塑等推送（statue_marble.lua:65、sculptures.lua:168、statuemaxwell.lua:48）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `IsLocked(prefab)` | `string -> bool` | 查草图或饰品是否仍锁定；锁定表在构造时全量初始化。[实现](../../../raw/current/scripts/components/chessunlocks.lua#L99-L101) |
| `GetNumLockedSketches/GetNumLockedTrinkets` | `-> number` | 剩余锁定量，trinkets.lua 用来判断"还有未解锁棋子"。[实现](../../../raw/current/scripts/components/chessunlocks.lua#L103-L109) |
| `ms_unlockchesspiece` 事件 | `TheWorld:PushEvent("ms_unlockchesspiece", 棋子名)` | 解锁 `chesspiece_<名>_sketch` 与关联 trinkets；OnLoad 按存档列表重放同一函数。[实现](../../../raw/current/scripts/components/chessunlocks.lua#L72-L80) |
| `OnSave/OnLoad` | 存读钩子 | 保存"已解锁棋子名"列表，加载后重放解锁。[实现](../../../raw/current/scripts/components/chessunlocks.lua#L115-L131) |

## 典型接入

### 掉落池排除未解锁内容

- 前提与选择：主机端读 `TheWorld.components.chessunlocks`，组件可能不存在（旧世界未注册），需判空。

```lua
local chessunlocks = TheWorld.components.chessunlocks
if chessunlocks ~= nil and not chessunlocks:IsLocked("trinket_15") then
    table.insert(loot, "trinket_15")
end
```

- 调整与验证：扩展自己的锁定内容时，组件表是只读常量（CHESS_UNLOCKS 定义在文件内，无注册接口），只能走 `ms_unlockchesspiece` 事件处理表内棋子。

## 常见陷阱与限制

- **锁定集硬编码**：只有表内 6 个棋子名有效，传其他名字 `IsLocked` 恒为 false 且事件不生效（[L72-L74](../../../raw/current/scripts/components/chessunlocks.lua#L72-L74)）。
- **组件在世界实体上**：查询经 `TheWorld.components.chessunlocks`，不要在物品上找；cave 与 forest 各自注册、各自存档。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；组件全文、trinkets/tumbleweed 过滤点、forest/cave 注册与事件推送方（grep 核验）。
- 游戏运行：未运行。
