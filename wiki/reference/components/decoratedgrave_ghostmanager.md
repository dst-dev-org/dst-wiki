---
title: "decoratedgrave_ghostmanager"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "Meta 5 装饰墓碑的守墓幽灵世界管理器：登记装饰墓碑，在有 wendy_gravestone_1 技能的玩家 15 距离内限量放 graveguard_ghost，玩家远离或墓碑消失时排队消散。"
aliases: ["decoratedgrave_ghostmanager", "装饰墓碑幽灵管理器", "守墓幽灵", "graveguard_ghost 调度"]
tags: [component, lifecycle, callback]
dst_build: "steam-buildid:26230496"
sources: ["components/decoratedgrave_ghostmanager.lua", "prefabs/forest.lua", "prefabs/cave.lua", "prefabs/gravestone.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# decoratedgrave_ghostmanager

## 快速判断

- 职责与适用：挂在 forest 与 cave 世界实体的服务端组件（构造断言 mastersim）；登记"已装饰墓碑"，仅当存在带 `ghostlyfriend` tag（温蒂技能树标识）的在线玩家时才每秒跑一轮：为技能 `wendy_gravestone_1` 玩家 15 距离内尚无幽灵的墓碑生成 `graveguard_ghost`，距该玩家 15 距内的有幽灵墓碑数上限为 `TUNING.WENDYSKILL_GRAVESTONE_GHOSTCOUNT`(4)。[构造](../../../raw/current/scripts/components/decoratedgrave_ghostmanager.lua#L7-L15) [生成判定](../../../raw/current/scripts/components/decoratedgrave_ghostmanager.lua#L159-L198)
- 优先使用：墓碑侧只需在完成装饰时调 `RegisterDecoratedGrave(grave)`；幽灵消散由 `_despawn_queued` 标记交给幽灵 prefab 自行处理，组件不直接 Remove。[登记](../../../raw/current/scripts/components/decoratedgrave_ghostmanager.lua#L131-L143) [标暗](../../../raw/current/scripts/components/decoratedgrave_ghostmanager.lua#L54-L62)
- 重要副作用：幽灵死亡或玩家远离 20 距离即排队消散，墓碑条目按死亡与否延迟 5 秒或一天重置为"无幽灵"，期间不重复生成。[重置](../../../raw/current/scripts/components/decoratedgrave_ghostmanager.lua#L181-L193) [远离消散](../../../raw/current/scripts/components/decoratedgrave_ghostmanager.lua#L203-L213)
- 修改边界：登记表不落盘（无 OnSave/OnLoad），恢复靠墓碑 prefab 载入后重新登记；无 `ghostlyfriend` 玩家在线时组件自停更新、已有幽灵全部排队消散。[玩家驱动启停](../../../raw/current/scripts/components/decoratedgrave_ghostmanager.lua#L85-L115)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `RegisterDecoratedGrave(grave)` | 墓碑实体 | 幂等登记为 `false`（"已知但无幽灵"），首条登记时启动更新；监听墓碑 `onremove` 自动注销并清最后一条时停更。([decoratedgrave_ghostmanager.lua:131-143](../../../raw/current/scripts/components/decoratedgrave_ghostmanager.lua#L131-L143) [墓碑调用](../../../raw/current/scripts/prefabs/gravestone.lua#L121 |
| `UnregisterDecoratedGrave(grave)` | 墓碑实体 | 转发到移除处理：幽灵标暗、注销监听、空表停更；未登记早退。([decoratedgrave_ghostmanager.lua:145-149](../../../raw/current/scripts/components/decoratedgrave_ghostmanager.lua#L145-L149) |
| `OnUpdate(dt)` | 组件更新（内部 1 秒节流） | 对每个 `ghostlyfriend` 玩家：技能 `wendy_gravestone_1` 已点亮才参与；附近有幽灵墓碑数 < 4 时为遍历命中的第一块无幽灵墓碑生成 `graveguard_ghost`、`LinkToHome(grave)` 并登记实体引用。([decoratedgrave_ghostmanager.lua:151-198](../../../raw/current/scripts/components/decoratedgrave_ghostmanager.lua#L151-L198) |
| 幽灵生命周期 | 内部状态 `false → 幽灵实体 → 定时重置` | 幽灵 `onremove` 后按 `health:IsDead()` 选 `WENDYSKILL_GRAVEGHOST_DEADTIME`（一天）或 5 秒重置墓碑条目；重复触发会取消旧任务避免竞态。([decoratedgrave_ghostmanager.lua:181-193](../../../raw/current/scripts/components/decoratedgrave_ghostmanager.lua#L181-L193) [调谐](../../../raw/current/scripts/tuning.lua#L8606-L8608) |
| 距离常数 | 私有常量 | `SPAWN_DISTANCE = 15.0001`（生成）、`DESPAWN_DISTANCE = 20.0001`（无玩家在旁即消散）、`UPDATE_RATE = 1` 秒。([decoratedgrave_ghostmanager.lua:13-15](../../../raw/current/scripts/components/decoratedgrave_ghostmanager.lua#L13-L15) |

## 典型接入

### 让自定义墓碑类建筑接入守墓幽灵

- 前提与选择：宿主世界为 forest 或 cave（组件仅这两处挂载，[forest.lua:729](../../../raw/current/scripts/prefabs/forest.lua#L729)、[cave.lua:575](../../../raw/current/scripts/prefabs/cave.lua#L575）；自定义实体需接受 `graveguard_ghost:LinkToHome` 的家园协议并响应 `_despawn_queued` 标记。
- 生效与撤销：登记即可参与生成与距离消散；移除实体时组件经 `onremove` 自动清理，无需手动反注册。
- 原版依据：装饰完成时机见 [gravestone.lua:121](../../../raw/current/scripts/prefabs/gravestone.lua#L121)（在 petal_decay／evilflower 计时器一并启动处）。

```lua
-- 自定义墓碑 prefab，服务端在"完成装饰"时调用一次：
if TheWorld.components.decoratedgrave_ghostmanager ~= nil then
    TheWorld.components.decoratedgrave_ghostmanager:RegisterDecoratedGrave(inst)
end
-- 幽灵侧消散约定（graveguard_ghost 已实现）：被标 _despawn_queued 后自行退场
```

- 调整与验证：数量上限与死亡冷却分别由 `WENDYSKILL_GRAVESTONE_GHOSTCOUNT`／`WENDYSKILL_GRAVEGHOST_DEADTIME` 控制；用 `GetDebugString()` 查墓碑与幽灵计数。

## 常见陷阱与限制

- 玩家资格检查在每轮更新而非入场时（([decoratedgrave_ghostmanager.lua:160-161](../../../raw/current/scripts/components/decoratedgrave_ghostmanager.lua#L160-L161) 注释说明技能晚于入场生效）：刚入场的温蒂要等下一秒轮询。
- 幽灵上限按"距该温蒂 15 距内"统计而非组件登记总数：多名温蒂各自结算自己附近的 4 只额度（([decoratedgrave_ghostmanager.lua:162-170](../../../raw/current/scripts/components/decoratedgrave_ghostmanager.lua#L162-L170)）。
- `_decorated_graves` 值混用 `false`／幽灵实体／定时任务句柄三种类型，外部遍历该内部表需按 `type(...) == "table" and ghost.prefab` 判别（([decoratedgrave_ghostmanager.lua:33-41](../../../raw/current/scripts/components/decoratedgrave_ghostmanager.lua#L33-L41)）。
- 生成距离按组件登记的墓碑全集计算，不查遮挡与可行走性，封闭空间内幽灵可能落在墙另一侧（([decoratedgrave_ghostmanager.lua:43-52](../../../raw/current/scripts/components/decoratedgrave_ghostmanager.lua#L43-L52)）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，通读组件全文；核验 forest/cave 双挂载、gravestone.lua 登记时机与两处调谐值。
- 游戏运行：未运行。
- 结构检查：未执行。
