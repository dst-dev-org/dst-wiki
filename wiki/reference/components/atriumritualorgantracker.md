---
title: "atriumritualorgantracker"
created: "2026-10-26"
updated: "2026-10-28"
type: "component"
summary: "洞穴世界宿主组件：登记中庭仪式已掉落器官 prefab 表；激活且蠕虫器官仍缺时保底 worm_boss 波，shrouden 被击败后整表重置。"
aliases: ["中庭仪式器官追踪", "仪式器官表", "atriumritualorgantracker"]
tags: ["component", "lifecycle"]
dst_build: "steam-buildid:26230496"
sources: ["components/atriumritualorgantracker.lua", "prefabs/cave.lua", "prefabs/atrium_gate.lua", "prefabs/atrium_ritual_organs.lua", "components/charliecutscene.lua", "prefabs/charlie_boss_trial.lua", "components/hounded.lua", "prefabs/rocky_boss.lua", "tuning.lua"]
reliability: "verified"
validation: "source-reviewed"
status: "active"
size_class: "simple"
---

# atriumritualorgantracker

## 快速判断

- 职责与适用：洞穴世界主端组件（cave.lua 添加，构造即断言主端）：只维护 `enabled`（门仪式是否激活）与 `organs`（本轮已掉落器官 prefab 表）；激活且蠕虫器官仍缺时催保底 worm_boss 波。[components/atriumritualorgantracker.lua:19](../../../raw/current/scripts/components/atriumritualorgantracker.lua#L19 [prefabs/cave.lua:579](../../../raw/current/scripts/prefabs/cave.lua#L579
- 优先使用：查询 `NeedsRitualOrgan(prefab)`（＝enabled 且表中无该 prefab）；掉落登记 `SetRitualOrgan(prefab)`；不要手工改 `organs`／`enabled` 字段。
- 重要副作用：`Enable(true)` 切换瞬间若仍缺蠕虫器官会推 `hounded_forceplannewattack`（×0.75 且取最小间隔）；shrouden 被击败清表并可能再催一波。[components/atriumritualorgantracker.lua:29](../../../raw/current/scripts/components/atriumritualorgantracker.lua#L29 [tuning.lua:9670](../../../raw/current/scripts/tuning.lua#L9670
- 修改边界：仅洞穴分片主端；事件监听在 TheWorld，门与器官 prefab 各管表现（见分工）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `Enable(enabled)` | boolean | 仅状态变化时生效；置真且仍缺蠕虫器官时催保底波；原版无置假调用方。[components/atriumritualorgantracker.lua:29](../../../raw/current/scripts/components/atriumritualorgantracker.lua#L29 |
| `RefreshOrgansTable()` | — | `self.organs = {}`；构造与 shrouden 被击败时调用。[components/atriumritualorgantracker.lua:38](../../../raw/current/scripts/components/atriumritualorgantracker.lua#L38 |
| `NeedsRitualOrgan(prefab)` | prefab → boolean | `enabled and not organs[prefab]`；cave 蠕虫波 specialupgradecheck 以它决定 `wave_pre_upgraded`。[components/atriumritualorgantracker.lua:42](../../../raw/current/scripts/components/atriumritualorgantracker.lua#L42 [prefabs/cave.lua:273](../../../raw/current/scripts/prefabs/cave.lua#L273 |
| `SetRitualOrgan(prefab)` | prefab → 无返回 | `organs[prefab] = true`；原版唯一调用方是器官的 on_loot_dropped。[components/atriumritualorgantracker.lua:46](../../../raw/current/scripts/components/atriumritualorgantracker.lua#L46 [prefabs/atrium_ritual_organs.lua:52](../../../raw/current/scripts/prefabs/atrium_ritual_organs.lua#L52 |
| `OnSave()`／`OnLoad(data)` | {enabled?, organs?} | enabled 为真或表非空才落档；OnLoad 原样恢复。[components/atriumritualorgantracker.lua:50](../../../raw/current/scripts/components/atriumritualorgantracker.lua#L50 |
| `GetDebugString()` | string | `Organs: [ … ]` 列出已登记 prefab。[components/atriumritualorgantracker.lua:71](../../../raw/current/scripts/components/atriumritualorgantracker.lua#L71 |

事件接线（构造时监听 TheWorld）：[components/atriumritualorgantracker.lua:25](../../../raw/current/scripts/components/atriumritualorgantracker.lua#L25

- `ms_atriumgate_ritualstatechanged`：atrium_gate 的 `SetRitualState` 每次状态变化推送（RITUAL_STATES：DISABLED 0~SUMMONED 4）；回调读 `gate:GetRitualState() >= gate.RITUAL_STATES.ENABLED` 即 `Enable(true)`。[prefabs/atrium_gate.lua:913](../../../raw/current/scripts/prefabs/atrium_gate.lua#L913 [prefabs/atrium_gate.lua:35](../../../raw/current/scripts/prefabs/atrium_gate.lua#L35
- 激活来源之一是钥匙嵌入仪式：charliecutscene `StartKeySocket` 延时调 gate `EnableRitual(true)`（报告 C-09 已核验条目，已对照源码）；读档续接走 gate OnLoadPostPass。[components/charliecutscene.lua:448](../../../raw/current/scripts/components/charliecutscene.lua#L448 [prefabs/atrium_gate.lua:590](../../../raw/current/scripts/prefabs/atrium_gate.lua#L590
- `ms_defeated_shrouden`：charlie_boss_trial 的 Boss 死亡回调推送（同点还推 `resetvault` 与 `Shard_SyncCharlieDefeated`，见 [[reference/components/charlie_tracker]]）；回调清器官表、enabled 仍真时再催保底波。[prefabs/charlie_boss_trial.lua:1079](../../../raw/current/scripts/prefabs/charlie_boss_trial.lua#L1079
- 催波实现：推 `hounded_forceplannewattack`（multtimetoattack＝`TUNING.WORM_BOSS_RITUAL_WAVE_TIME_MULT`，choosemintime=true），消费者 hounded 组件。[components/atriumritualorgantracker.lua:1](../../../raw/current/scripts/components/atriumritualorgantracker.lua#L1 [components/hounded.lua:725](../../../raw/current/scripts/components/hounded.lua#L725

### 分工

- 本组件＝账本与保底波触发；器官 prefab（atrium_ritual_organ_rocky/bat/worm，出自各 Boss 暗影战利品，如 rocky_boss_shadow 稳掉岩石器官）＝物品本体：掉落时登记；若掉的正是仍缺的蠕虫器官，再催一次最小间隔的波。[prefabs/atrium_ritual_organs.lua:46](../../../raw/current/scripts/prefabs/atrium_ritual_organs.lua#L46 [prefabs/rocky_boss.lua:49](../../../raw/current/scripts/prefabs/rocky_boss.lua#L49
- 门与地面标记（atrium_ritual_marking）＝捡放/消耗仪式物品与状态机；charliecutscene＝钥匙嵌入过场；器官被仪式消耗（`Consume`）不经过本组件。

## 典型接入

### 把自定义仪式物品登记进器官表

- 前提与选择：`modmain.lua` 服务端监听掉落；本表只影响 `NeedsRitualOrgan` 查询方。
- 生效与撤销：登记当轮有效，shrouden 被击败即整表清空。

```lua
AddPrefabPostInit("my_ritual_organ", function(inst)
    if GLOBAL.TheWorld.ismastersim then
        inst:ListenForEvent("on_loot_dropped", function(inst)
            local tracker = GLOBAL.TheWorld.components.atriumritualorgantracker
            if tracker ~= nil then
                tracker:SetRitualOrgan(inst.prefab)
            end
        end)
    end
end)
```

## 常见陷阱与限制

- **disabled 时 NeedsRitualOrgan 恒 false**：它编码「已激活且仍缺」，不能当「是否已掉落」探测用——登记本身不看 enabled。[components/atriumritualorgantracker.lua:42](../../../raw/current/scripts/components/atriumritualorgantracker.lua#L42
- **登记以掉落为准，不追踪后续**：器官被拾起、入仪式、消耗都不回滚；表项本轮（直至 shrouden 被击败）一直为真。[prefabs/atrium_ritual_organs.lua:52](../../../raw/current/scripts/prefabs/atrium_ritual_organs.lua#L52
- **客户端无组件**：构造即断言主端。[components/atriumritualorgantracker.lua:20](../../../raw/current/scripts/components/atriumritualorgantracker.lua#L20

## 验证记录

- 源码审阅：2026-10-26，Steam buildid 26148036；覆盖组件 82 行全部逻辑、cave 挂载与蠕虫波 specialupgradecheck、atrium_gate 推送点与 RITUAL_STATES、charliecutscene 激活链（复用报告 C-09 已核验条目并对照源码）、charlie_boss_trial 事件源、atrium_ritual_organs 登记回调与 hounded 消费。未覆盖：atrium_gate 状态机全流程、器官/标记动画细节、游戏运行。
- 游戏运行：未运行。
- 结构检查：2026-10-26 执行 validate_page.py（simple 档）与 validate_source_links.py（--strict）通过；页内 Lua 示例经 parser_probe.exe --syntax-only 语法检查通过。

相关：[[reference/components/rockybossspawner]]、[[reference/components/charlie_tracker]]、[[reference/components/shard_charlieinfo]]。
