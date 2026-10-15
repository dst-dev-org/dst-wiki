---
title: "playerfloater"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "玩家漂浮物品组件：让手部物品（原版游泳圈）在玩家落水时被 SGwilson float 状态自动装备，回岸或加载时重置。"
aliases: ["游泳圈", "漂浮装备", "playerfloater"]
tags: [component, item, equipment, callback]
dst_build: "steam-buildid:26230496"
sources: ["components/playerfloater.lua", "prefabs/flotationcushion.lua", "stategraphs/SGwilson.lua", "prefabs/player_common.lua", "actions.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# playerfloater

## 快速判断

- 职责与适用：挂在物品上（原版仅 flotationcushion，[挂接](../../../raw/current/scripts/prefabs/flotationcushion.lua#L63-L65)），负责在玩家落水时把该物品临时变成 equippable 并自动装备、上岸后卸下还原；不实现浮力本身。
- 优先使用：识别"这是漂浮物品"用 `playerfloater` 标签（构造时加，[ctor](../../../raw/current/scripts/components/playerfloater.lua#L7)），因为 equippable 只在装备期间存在；动作层已按标签识别（([actions.lua:1198-1201](../../../raw/current/scripts/actions.lua#L1198-L1201)）。
- 重要副作用：`MakeEquippable_Internal` 动态加 equippable 且 `SetPreventUnequipping(true)` 禁止手动卸下（[实现](../../../raw/current/scripts/components/playerfloater.lua#L33-L38)）。
- 修改边界：装备回调经 equippable 转发；卸下由内部包装函数触发并顺带移除 equippable（[包装](../../../raw/current/scripts/components/playerfloater.lua#L25-L31)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SetOnEquip(fn)` / `SetOnUnequip(fn)` | `fn(item, owner)` | SetOnEquip 若此刻已有 equippable 则直接转发，否则存 `onequipfn` 供装备时使用；SetOnUnequip 仅存字段（[实现](../../../raw/current/scripts/components/playerfloater.lua#L14-L23)）。 |
| `AutoDeploy(player)` | `player: entity -> 无` | 丢下手部物品、关闭 BODY 容器（背包），然后 `MakeEquippable_Internal` 并装备自身（[实现](../../../raw/current/scripts/components/playerfloater.lua#L40-L54)）。 |
| `LetGo(player, randomdir, pos)` | `-> 无` | 解除 PreventUnequipping 后 `DropItem`（落水放手）（[实现](../../../raw/current/scripts/components/playerfloater.lua#L56-L59)）。 |
| `Reset(player)` | `-> 无` | 解除禁止卸下、卸下并放回玩家背包（上岸还原）（[实现](../../../raw/current/scripts/components/playerfloater.lua#L61-L65)）。 |
| `OnSave` / `OnLoad` / `LoadPostPass` | 存读钩子 | 装备态存 `equipped=true` 并在加载时重建 equippable；`LoadPostPass` 反向移除 equippable，且"从容器加载"时不会运行（[注释](../../../raw/current/scripts/components/playerfloater.lua#L79-L83)）。 |

## 典型接入

### 落水自动换装链路（原版参照）

- 前提与选择：宿主 prefab pristine 加 `playerfloater` 标签并设好两个回调（([flotationcushion.lua:63-65](../../../raw/current/scripts/prefabs/flotationcushion.lua#L63-L65)）；玩家侧 float 状态用"有 playerfloater 且当前无 equippable"作为可进入条件（[SGwilson.lua:739](../../../raw/current/scripts/stategraphs/SGwilson.lua#L739）。
- 生效与撤销：落水进 float 状态时 `AutoDeploy`（([SGwilson.lua:25827-25828](../../../raw/current/scripts/stategraphs/SGwilson.lua#L25827-L25828)）；读档落水失败则 `Reset` 放回背包（([player_common.lua:1342-1352](../../../raw/current/scripts/prefabs/player_common.lua#L1342-L1352)）。

```lua
-- 服务端：漂浮物品宿主（flotationcushion 等价最小接法）
inst:AddTag("playerfloater") -- pristine 阶段
-- master postinit：
inst:AddComponent("playerfloater")
inst.components.playerfloater:SetOnEquip(OnEquip)
inst.components.playerfloater:SetOnUnequip(OnUnequip)
```

- 调整与验证：该组件不处理游泳速度与溺水判定（在 drownable／SGwilson）；仅装备期间可被 `inventory:GetEquippedItem` 查到（[inventory.lua:1374](../../../raw/current/scripts/components/inventory.lua#L1374）。

## 常见陷阱与限制

- **不要直接加 equippable**：漂浮物品平时不能是 equippable（否则落水判定失效）；equippable 由 `AutoDeploy` 动态添加、卸下时被移除（[实现](../../../raw/current/scripts/components/playerfloater.lua#L33-L38)）。
- **SetOnUnequip 不转发**：卸下回调只能存到组件上，实际触发在内部 `OnUnequip` 包装里（先调 fn 再 RemoveComponent("equippable")，[实现](../../../raw/current/scripts/components/playerfloater.lua#L25-L31)）。
- **读档边界**：存档时已装备则 OnLoad 重建 equippable；未装备（在背包／地上）走 LoadPostPass 清理——两分支互斥，注释明示从容器加载不走 LoadPostPass（([playerfloater.lua:79-83](../../../raw/current/scripts/components/playerfloater.lua#L79-L83)）。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；覆盖组件全文、flotationcushion 挂接、SGwilson float 状态两处调用、player_common 读档 Reset、DROP 提示串与 inventory floaterheld。
- 结构检查：本批 validate_page、validate_source_links 及示例 lua 语法探测（见批级记录）。
- 游戏运行：未运行。
