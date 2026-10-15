---
title: "shadowthrall_mimics"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "世界级影子裂隙物品拟态生成器：暗影裂隙开启后按玩家周期性把黑暗中的可拟态物品克隆为 itemmimic，并供宝箱/拟态箱判定启用。"
aliases: ["影子拟态生成", "shadowthrall_mimics", "物品拟态", "宝箱拟态"]
tags: [component]
dst_build: "steam-buildid:26230496"
sources: ["components/shadowthrall_mimics.lua", "prefabs/itemmimic_data.lua", "prefabs/treasurechest.lua", "stategraphs/SGitemmimic_revealed.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# shadowthrall_mimics

## 快速判断

- 职责与适用：挂在 TheWorld 的服务端世界组件（[断言](../../../raw/current/scripts/components/shadowthrall_mimics.lua#L9)），在暗影裂隙进入裂隙池且洞穴夜晚时，为每位玩家周期性把附近黑暗/无人视野中的可拟态物品克隆成 `itemmimic`；仅 mastersim。
- 优先使用：想判定"当前是否启用拟态"用 `IsEnabled()`；想手动克隆用 `SpawnMimicFor(item)`（[接口](../../../raw/current/scripts/components/shadowthrall_mimics.lua#L199-L212)）。
- 重要副作用：生成的拟态继承目标的持久化数据（`SetPersistData`）并被强加 `itemmimic` 组件（[克隆](../../../raw/current/scripts/components/shadowthrall_mimics.lua#L57-L60)）。
- 修改边界：目标带 `itemmimic` 即拒绝（拟态不拟态，[判定](../../../raw/current/scripts/components/shadowthrall_mimics.lua#L37-L39)）；候选池由 `itemmimic_data` 的标签决定（[定义](../../../raw/current/scripts/prefabs/itemmimic_data.lua)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `IsTargetMimicable(target)` | → `bool` | 目标须含全部 MUST_TAGS（`_equippable`,`_inventoryitem`）且不含任一 CANT_TAGS（`_container`,`_health`,`_stackable`,`heavy`,`smallcreature`,`INLIMBO`）（[实现](../../../raw/current/scripts/components/shadowthrall_mimics.lua#L199-L202)、[标签](../../../raw/current/scripts/prefabs/itemmimic_data.lua)）。 |
| `SpawnMimicFor(item)` | → `success, mimic or nil` | 先做可拟态判定再克隆；成功返回克隆体（[实现](../../../raw/current/scripts/components/shadowthrall_mimics.lua#L204-L208)）。 |
| `IsEnabled()` | → `bool` | 暗影裂隙修正列表非空（[实现](../../../raw/current/scripts/components/shadowthrall_mimics.lua#L210-L212)）。 |
| `Debug_PlayerSpawns(player)` | 无返回 | 调试用，强制为玩家启动尝试任务（[实现](../../../raw/current/scripts/components/shadowthrall_mimics.lua#L231-L233)）。 |

## 典型接入

### 洞穴夜晚自动生成拟态

- 前提与选择：原版在 `cave.lua` 挂载（[cave](../../../raw/current/scripts/prefabs/cave.lua#L572)）；`forest.lua` 也挂载但只承接从洞穴带回的拟态重新躲藏，不自动生成（[forest](../../../raw/current/scripts/prefabs/forest.lua#L720-L726)）。生成需同时满足：`ms_riftaddedtopool` 推入暗影亲和裂隙（[监听](../../../raw/current/scripts/components/shadowthrall_mimics.lua#L191)）且 `iscavenight` 为真（[开关](../../../raw/current/scripts/components/shadowthrall_mimics.lua#L109-L120)）。
- 生效与撤销：每玩家 `DoPeriodicTask(10*(1+math.random()))` 尝试一次（[调度](../../../raw/current/scripts/components/shadowthrall_mimics.lua#L103-L107)）；活跃拟态数达 `TUNING.ITEMMIMIC_CAP`（=10，[数值](../../../raw/current/scripts/tuning.lua#L8520)）即停；搜索半径 15，候选须处于黑暗中或已休眠（[条件](../../../raw/current/scripts/components/shadowthrall_mimics.lua#L76-L92)）；裂隙移出池或白天到来时取消任务。
- 原版依据：宝箱在 `resetruins`／`ms_riftaddedtopool` 时按 `TUNING.CHEST_MIMIC_CHANCE`（=0.33，[数值](../../../raw/current/scripts/tuning.lua#L8529)）变成 `chest_mimic`，仅要求组件存在且 `IsEnabled()`（[宝箱](../../../raw/current/scripts/prefabs/treasurechest.lua#L444-L447)）；拟态物现形失败时经 `SpawnMimicFor` 复制新目标（[SG](../../../raw/current/scripts/stategraphs/SGitemmimic_revealed.lua#L142-L143)）。

```lua
-- 服务端，TheWorld 上已有组件；强制对一个物品做拟态克隆
local w = TheWorld.components.shadowthrall_mimics
if w ~= nil and w:IsEnabled() and w:IsTargetMimicable(target) then
    local success, mimic = w:SpawnMimicFor(target)
end
```

- 调整与验证：模组可用 `SourceModifierList` 思路直接读 `IsEnabled()` 做门控；改候选池应改 `itemmimic_data.lua` 的标签表而非组件。

## 常见陷阱与限制

- 只有洞穴世界会自动生成：地表 `forest.lua` 挂载组件但注释明确不自动 spawn（[注释](../../../raw/current/scripts/prefabs/forest.lua#L724-L726)），仅响应现形 SG 的主动克隆。
- 克隆在服务端做，但生成点刻意挑黑暗或玩家视野外（[注释](../../../raw/current/scripts/components/shadowthrall_mimics.lua#L84-L85)）；物品小数坐标近 0 时会取整对齐，借几何摆放隐藏拟态（[对齐](../../../raw/current/scripts/components/shadowthrall_mimics.lua#L49-L55)）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，覆盖组件全文、itemmimic_data 标签、cave/forest 挂载、treasurechest/chest_mimic/SGitemmimic_revealed 消费链与 TUNING 两常量；未运行游戏。
- 结构检查：validate_page.py、validate_source_links.py 与 Lua 语法探针通过。
