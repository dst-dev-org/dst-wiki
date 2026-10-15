---
title: "socket_shadow_harvester：WX-78 影子收割无人机调度"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "为 WX-78 影子插座收割形态调度 wx78_shadowdrone_harvester 宠物自动拾取周围可采物品，玩家手动采集也会触发补发调度。"
aliases: [影子收割插座, 影子收割无人机调度, socket_shadow_harvester]
tags: [component, callback, ai]
dst_build: "steam-buildid:26230496"
sources: ["components/socket_shadow_harvester.lua", "prefabs/wx78_common.lua", "brains/wx78_shadowdrone_harvesterbrain.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# socket_shadow_harvester（WX-78 影子收割无人机调度）

## 快速判断

- 职责与适用：挂在玩家（或无人机宿主）实体上的服务端调度组件。每次 Tick 经宿主 [petleash](../../../raw/current/scripts/components/socket_shadow_harvester.lua#L274-L282) 找出 `wx78_shadowdrone_harvester` 宠物，为空闲无人机在 `harvestradius` 内搜可采物品并登记监听；无人机脑再经 `GetItemForHarvester` 取目标执行拾取。挂载与半径由 WX-78 影子插座技能档位接线（([wx78_common.lua:642-643](../../../raw/current/scripts/prefabs/wx78_common.lua#L642-L643)）。
- 优先使用：要让影子收割无人机自动采集 → 由 `SetShadowHarvesterLimits_Default/_Boosted` 挂载并设半径，不手工挂；无人机自己不挂本组件，只读宿主的组件（[brain 读取](../../../raw/current/scripts/brains/wx78_shadowdrone_harvesterbrain.lua#L16-L21)）。
- 重要副作用：宿主是玩家时改为监听 `onpickupitem`／`picksomething`／`picksomethingfromaoe` 触发 Tick（玩家须主动采集才补发）；宿主睡着时进入"虚拟收割"模拟，直接瞬移无人机并用 `BufferedAction` 假执行采集与归还（[模拟](../../../raw/current/scripts/components/socket_shadow_harvester.lua#L166-L240)）。
- 修改边界：非玩家宿主按 `TICK_PERIOD`（1.5s）周期 Tick；组件卸载时对在忙无人机清链并 `DropEverything` 扔掉所持物品（[卸载](../../../raw/current/scripts/components/socket_shadow_harvester.lua#L81-L95)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SetHarvestRadius(r)` | 写 `self.harvestradius`（构造默认 6） | ([socket_shadow_harvester.lua:98-100](../../../raw/current/scripts/components/socket_shadow_harvester.lua#L98-L100) |
| `GetHarvestRadius()` | 返回基础半径＋技能加成 | 加 `wx78_extradronerange` 技能＋`SHADOWDRONE_HARVESTER_FINDITEM_RADIUS_SKILLBOOST`；玩家本体或 `wx78_bodycircuits`＋`wx78_circuitry_betabuffs_1` 再按雷达模块数加成；宿主经 `linkeditem:GetOwnerInst` 归属玩家。[加成计算](../../../raw/current/scripts/components/socket_shadow_harvester.lua#L102-L117) |
| `GetItemForHarvester(harvester)` | 返回该无人机在忙物品或 nil | 无人机脑据此取目标。[brain 调用](../../../raw/current/scripts/brains/wx78_shadowdrone_harvesterbrain.lua#L16-L21) |
| `TryToFindItem()` | 返回拾取候选实体或 nil | 用 `FindPickupableItem`，过滤保留植物类目标（非 pickable 或带 `HARVESTABLE_PLANT_TARGET_TAGS`）。[过滤](../../../raw/current/scripts/components/socket_shadow_harvester.lua#L125-L132) |
| `DoTick()` | 主循环入口 | 无 `petleash` 或无该类宠物即返回；`while DoTick_Internal` 逐个派发空闲无人机。[主循环](../../../raw/current/scripts/components/socket_shadow_harvester.lua#L274-L288) |
| 触发接线 | 玩家三事件／非玩家周期任务 | 构造时按 `self.inst.isplayer` 分支；无人机宠物自身也走周期 Tick 分支。[构造](../../../raw/current/scripts/components/socket_shadow_harvester.lua#L59-L67) |

派发与监听：`DoTick_Internal` 选空闲无人机→`TryToFindItem`→登记 `busyharvesters`／`items` 双向表，并在宿主上监听无人机 `onremove`、`braincommon_pickup_failed/success`、`entitysleep` 与物品 `onputininventory`／`onremove`（[派发](../../../raw/current/scripts/components/socket_shadow_harvester.lua#L242-L272)）。任一事件触发即清链并取消无人机在途动作。虚拟收割模拟按实测时序垫延迟：出发 2.7s、采集 1.5s、归还 2.5s（[常量](../../../raw/current/scripts/components/socket_shadow_harvester.lua#L1-L5)）。

## 典型接入

### 随技能档位挂载与撤销

- 前提与选择：服务端玩家实体；原版在 WX-78 影子插座技能升级回调里挂载，并把半径与 [[reference/components/petleash]] 的该类宠物上限一起设。
- 生效与撤销：挂载后即刻参与 Tick；降档或断开技能时 `RemoveComponent` 即清全部监听与在忙状态。

```lua
-- 服务端：WX-78 玩家实体上，对照 prefabs/wx78_common.lua 的 Default 档
local harvester = inst.components.socket_shadow_harvester
    or inst:AddComponent("socket_shadow_harvester")
harvester:SetHarvestRadius(TUNING.SKILLS.WX78.SHADOWDRONE_HARVESTER_FINDITEM_RADIUS)

-- 撤销（对照 wx78_common.lua:632）
inst:RemoveComponent("socket_shadow_harvester")
```

- 调整与验证：半径调谐值见 ([tuning.lua:7574-7582](../../../raw/current/scripts/tuning.lua#L7574-L7582)（周期 1.5s、默认 6、强化 10、上限 2/4 只）；自制形态可传自定义半径。

## 常见陷阱与限制

- **无人机必须以 `wx78_shadowdrone_harvester` 记入 petleash**：`DoTick` 只按 prefab 名取宠物，改名或用其他宠物组件都派发不到（[按名查询](../../../raw/current/scripts/components/socket_shadow_harvester.lua#L275-L282)）。
- **玩家宿主不自动 Tick**：挂在玩家身上时没有周期任务，完全依赖三 个采集事件；纯站立不会补发（[分支](../../../raw/current/scripts/components/socket_shadow_harvester.lua#L59-L63)）。
- **模拟只在宿主睡着时运行**：`DoTick_Internal_Simulations` 对 `IsAsleep` 的在忙无人机虚拟推进；宿主醒着时无人机由真实 AI 行动。模拟失败路径直接 `DropEverything`（[归还失败](../../../raw/current/scripts/components/socket_shadow_harvester.lua#L208-L210)）。
- **卸载时扔货**：`OnRemoveFromEntity` 对所有在忙无人机清链并倒出物品，降档技能可能让无人机当场掉落所采物（[卸载](../../../raw/current/scripts/components/socket_shadow_harvester.lua#L81-L95)）。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496（steam-buildid:26230496）；覆盖 `components/socket_shadow_harvester.lua` 全部 291 行、`prefabs/wx78_common.lua` 挂载三处与降档移除、`brains/wx78_shadowdrone_harvesterbrain.lua` 目标读取、`tuning.lua` 相关常量。未覆盖：无人机 prefab/脑的行为细节、`HARVESTABLE_PLANT_TARGET_TAGS` 全表。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 通过；lua 示例通过 parser_probe 语法检查。
- 游戏运行：未运行。
