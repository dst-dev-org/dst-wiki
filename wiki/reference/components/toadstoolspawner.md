---
title: "toadstoolspawner"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "洞穴世界的蛤蟆菌生成协调器：经 worldsettingstimer 保证全场同时至多一只活跃蛤蟆菌，协调各生成点（toadstool_cap）登记、重生与气体状态。"
aliases: ["蛤蟆菌生成器", "toadstoolspawner", "蛤蟆菌重生", "toadstool cap 协调"]
tags: [component, creature, lifecycle, callback]
dst_build: "steam-buildid:26230496"
sources: ["components/toadstoolspawner.lua", "prefabs/cave.lua", "prefabs/toadstool_cap.lua", "prefabs/birds.lua", "prefabs/birdcage.lua", "prefabs/cave_vents.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# toadstoolspawner

## 快速判断

- 职责与适用：洞穴世界主模拟组件（构造即断言 `TheWorld.ismastersim`），自己不生成实体，只维护 `worldsettingstimer` 全局计时器 `toadstool_respawntask` 与生成点列表；蛤蟆菌本体由各生成点在 `ms_spawntoadstool` 事件下生成。[断言与安装](../../../raw/current/scripts/prefabs/cave.lua#L523) [计时器注册](../../../raw/current/scripts/components/toadstoolspawner.lua#L126)
- 优先使用：查询"当前是否有活跃蛤蟆菌（毒气期）"用 `IsEmittingGas()`——金丝雀中毒、鸟笼、洞穴裂隙都以此判定；让生成点参与协调走 `ms_registertoadstoolspawner` 世界事件。[气体查询](../../../raw/current/scripts/components/toadstoolspawner.lua#L155-L160) [登记](../../../raw/current/scripts/prefabs/toadstool_cap.lua#L334)
- 重要副作用：生成点状态变化/死亡会向世界推 `toadstoolstatechanged`／`toadstoolkilled` 世界事件，并据此停止或重启重生计时器；计时器到点随机挑一个生成点推 `ms_spawntoadstool`。[状态联动](../../../raw/current/scripts/components/toadstoolspawner.lua#L69-L87) [随机重生](../../../raw/current/scripts/components/toadstoolspawner.lua#L39-L48)
- 修改边界：计时器时长受世界设置覆盖键 `TUNING.SPAWN_TOADSTOOL` 影响，默认 `TUNING.TOADSTOOL_RESPAWN_TIME`；移除生成点后若无活跃蛤蟆菌则用 `TOADSTOOL_SPAWN_TIME` 重启。[注册](../../../raw/current/scripts/components/toadstoolspawner.lua#L104-L121)

## 关键流程

构造即 `AddTimer` 注册全局计时器并以 `TOADSTOOL_SPAWN_TIME` 起跳；`ms_registertoadstoolspawner` 收到生成点后去重登记并监听其 `toadstoolstatechanged`／`toadstoolkilled`／onremove；任一时刻存在 `HasToadstool()` 的生成点就停表，否则在无活跃计时器时起表；计时器到点（`worldsettingstimer` 回调）若无活跃蛤蟆菌则 `TriggerRandomSpawner`；`OnPostInit` 与载入后再校正一次。[构造与安装](../../../raw/current/scripts/components/toadstoolspawner.lua#L126-L137) [状态机入口](../../../raw/current/scripts/components/toadstoolspawner.lua#L69-L102)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `IsEmittingGas()` | `-> boolean` | 以"重生计时器不活跃"反推有毒气（存在活跃蛤蟆菌）；比遍历生成点快，但生成点被强删时可能短暂不一致（源码注释自述）。[实现](../../../raw/current/scripts/components/toadstoolspawner.lua#L155-L160) [金丝雀消费](../../../raw/current/scripts/prefabs/birds.lua#L401) |
| `OnPostInit()` | 生命周期钩子 | 世界后初始化校正计时器：有活跃蛤蟆菌停表，否则无表则起表。[实现](../../../raw/current/scripts/components/toadstoolspawner.lua#L143-L149) |
| `OnSave`／`OnLoad` | 标准存读钩子 | 存 `toadstool_queued_spawn`；载入兼容旧字段 `timetorespawn`（截断到上限），否则按存档停表。[实现](../../../raw/current/scripts/components/toadstoolspawner.lua#L166-L177) |
| `GetDebugString()` | 调试串 | 显示活跃蛤蟆菌与剩余冷却。[实现](../../../raw/current/scripts/components/toadstoolspawner.lua#L183-L189) |

## 典型接入

### 消费"毒气期"状态

- 前提与选择：任何洞穴世界的主模拟代码读 `TheWorld.components.toadstoolspawner:IsEmittingGas()`；原版金丝雀毒化、鸟笼放出毒气、裂隙休眠判断都是这个入口。[鸟笼](../../../raw/current/scripts/prefabs/birdcage.lua#L642) [裂隙](../../../raw/current/scripts/prefabs/cave_vents.lua#L105)
- 生效与撤销：状态翻转由生成点事件驱动，模组不应手动改计时器，应生成/移除生成点或杀蛤蟆菌触发既有事件。[事件链](../../../raw/current/scripts/components/toadstoolspawner.lua#L69-L87)

```lua
local function IsToadstoolGasActive()
    local spawner = TheWorld.components.toadstoolspawner
    return spawner ~= nil and spawner:IsEmittingGas()
end
```

- 调整与验证：金丝雀毒化状态图还要求组件存在（`== nil` 时走普通分支）；裂隙按 `riftspawner` 与本组件组合判断。[状态图守卫](../../../raw/current/scripts/stategraphs/SGcanarypoisoned.lua#L11) [裂隙组合](../../../raw/current/scripts/prefabs/cave_vents.lua#L435-L442)

## 常见陷阱与限制

- **IsEmittingGas 是反推值：** 它等价于"计时器不在跑"，生成点被强制移除（不走 killed 事件）时可能与其他判定不同步；精确判断需遍历生成点 `HasToadstool()`。[注释自述](../../../raw/current/scripts/components/toadstoolspawner.lua#L156-L158)
- **重生只挑随机一个生成点：** `TriggerRandomSpawner` 不检查该点是否可用（可用性由生成点自己的 `ms_spawntoadstool` 处理），失败不会自动换点重试。[随机](../../../raw/current/scripts/components/toadstoolspawner.lua#L39-L48)
- **世界设置键与时长是两个概念：** `TUNING.SPAWN_TOADSTOOL` 是 worldsettingstimer 的覆盖开关，`TOADSTOOL_RESPAWN_TIME`／`TOADSTOOL_SPAWN_TIME` 才是时长；混淆会导致改值不生效。[注册](../../../raw/current/scripts/components/toadstoolspawner.lua#L126)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（195 行），核验 cave.lua 安装、toadstool_cap 登记与 birds/birdcage/cave_vents/SGcanarypoisoned 消费点。
- 结构检查：2026-10-28 validate_page.py 通过（本页）；validate_source_links.py --strict 本批 16 页通过。
- 游戏运行：未运行。
