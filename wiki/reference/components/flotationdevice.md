---
title: "flotationdevice"
created: "2026-10-21"
updated: "2026-10-28"
type: component
summary: "物品侧的浮具标记与回调：只向溺水伤害筛选提供启用状态，不负责检测、下沉或传送。"
aliases: ["浮具", "防溺水装备", "浮力装备", "flotationdevice"]
tags: [component, equipment, item, callback]
dst_build: "steam-buildid:26230496"
sources: ["components/flotationdevice.lua", "components/drownable.lua", "components/inventory.lua", "components/equippable.lua", "prefabs/balloonvest.lua", "stategraphs/SGwilson.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# flotationdevice

## 快速判断

- 原版宿主与注册名：唯一原版宿主是 `balloonvest`（气球背心），在 prefab 的主模拟段添加并赋回调，装备槽为 `BODY`，注册名就是 `balloonvest`。[添加](../../../raw/current/scripts/prefabs/balloonvest.lua#L78-L79)、[槽位](../../../raw/current/scripts/prefabs/balloonvest.lua#L82)、[注册](../../../raw/current/scripts/prefabs/balloonvest.lua#L90)
- 职责与适用：装备/持有物侧的“浮具标记＋一个回调”；不检测落水、不改状态、不传送、不结算，唯一消费者是 `drownable:TakeDrowningDamage` 的库存筛选。[消费者](../../../raw/current/scripts/components/drownable.lua#L328-L359)
- 优先使用：已有 `drownable` 的实体想让某件物品抵消一次溺水伤害时，在服务端物品 prefab 上加本组件并设 `onpreventdrowningdamagefn`；原版就是背心穿戴后触发 `poppable:Pop()`。[balloonvest](../../../raw/current/scripts/prefabs/balloonvest.lua#L40-L42)
- 重要副作用：回调只在伤害结算时触发一次；不阻止 `onsink`、下沉动画、传送或掉落；湿度在浮具判断之前已经结算。[顺序](../../../raw/current/scripts/components/drownable.lua#L348-L359)
- 修改边界：无网络字段、无存档、无清理钩子；原版在 `ismastersim` 后添加；物品必须在该实体的 `inventory` 中，可装备物品还必须 `IsEquipped()`。[条件](../../../raw/current/scripts/components/drownable.lua#L328-L331)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `enabled` | `boolean`，构造时置 true | 直接读写，没有 setter；`IsEnabled` 只读它。[构造](../../../raw/current/scripts/components/flotationdevice.lua#L2-L7) |
| `IsEnabled()` | 返回 `boolean` | 消费者第一道条件；false 时该物品不参与抵消。[IsEnabled](../../../raw/current/scripts/components/flotationdevice.lua#L9-L11) |
| `onpreventdrowningdamagefn` | `function(item)` 或 nil | 由宿主赋值；未赋值时 `OnPreventDrowningDamage` 是空操作。[回调字段](../../../raw/current/scripts/components/flotationdevice.lua#L13-L17) |
| `OnPreventDrowningDamage()` | 无参数、无返回值 | 消费者传入的 `Vector3` 被忽略；只调用回调一次，是否消耗物品由回调决定。[调用点](../../../raw/current/scripts/components/drownable.lua#L356) |
| 消费者契约 | 组件存在＋`IsEnabled()`＋（无 `equippable` 或 `IsEquipped()`） | `inventory:FindItems` 搜物品栏、装备槽、激活物品和溢出容器；命中多个时只取第一个，顺序取决于 `FindItems` 的 `pairs` 遍历，不保证装备槽优先。[筛选](../../../raw/current/scripts/components/drownable.lua#L328-L331)、[FindItems](../../../raw/current/scripts/components/inventory.lua#L638-L664) |
| `equippable:IsEquipped()` | 返回 `boolean` | 只反映 `isequipped` 字段；穿戴动作负责置位。[IsEquipped](../../../raw/current/scripts/components/equippable.lua#L88-L90) |

## 关键流程

1. 玩家/生物沉没结束，`washed_ashore` 状态调用 `drownable:TakeDrowningDamage()`。[SGwilson](../../../raw/current/scripts/stategraphs/SGwilson.lua#L13673-L13681)
2. 先按浅水/普通倍率加湿度；浮具不参与这一步。
3. `inventory:FindItems(is_enabled_flotation_item)` 找第一个“存在组件、`IsEnabled()`、可装备时已装备”的物品。[筛选](../../../raw/current/scripts/components/drownable.lua#L328-L359)
4. 命中就调用该物品的 `OnPreventDrowningDamage` 并提前返回：跳过饥饿、生命惩罚、生命和理智；未命中才继续扣。[分支](../../../raw/current/scripts/components/drownable.lua#L352-L381)
5. 回调自行决定代价（原版气球背心 `poppable:Pop()`）；组件本身不移除物品、不改 `enabled`。

## 典型接入

### 给装备提供一次性防溺水（原版 balloonvest 模式）

- 前提与选择：物品 prefab 在主模拟段添加组件；物品要能被 `drownable` 找到，可装备物品必须穿在对应槽位（气球背心是 `BODY`）。[balloonvest](../../../raw/current/scripts/prefabs/balloonvest.lua#L78-L83)
- 生效与撤销：组件一直有效，直到物品离开库存或 `enabled=false`；气球背心在回调里 `poppable:Pop()` 自毁，所以只挡一次。[回调](../../../raw/current/scripts/prefabs/balloonvest.lua#L40-L42)
- 原版依据：全源码只有 `drownable` 读取本组件，除 `balloonvest` 外没有其他 prefab 添加它。[筛选](../../../raw/current/scripts/components/drownable.lua#L328-L331)

```lua
-- 服务端物品 prefab 的主模拟段（与 balloonvest 同形；poppable 需宿主已有）
inst:AddComponent("flotationdevice")
inst.components.flotationdevice.onpreventdrowningdamagefn = function(item)
    item.components.poppable:Pop()
end
```

- 调整与验证：想多次生效就不要在回调里移除物品；想临时失效就写 `enabled=false`。片段只在物品已有 `poppable` 时成立。[balloonvest](../../../raw/current/scripts/prefabs/balloonvest.lua#L40-L42)

## 常见陷阱与限制

- 把“浮具”当“不会沉”：下沉、传送、掉落、湿度都由 `drownable` 的其他步骤完成，本组件只在伤害分支里起作用。[流程](../../../raw/current/scripts/components/drownable.lua#L348-L381)
- 放在背包里不等于生效：可装备物品必须 `IsEquipped()`；`FindItems` 同时搜装备槽，所以“带在身上”不够。[条件](../../../raw/current/scripts/components/drownable.lua#L328-L331)
- 多个浮具只触发一个：`FindItems` 结果取 `[1]`，其余不会收到回调。[分支](../../../raw/current/scripts/components/drownable.lua#L354-L357)
- 没有内建消耗：回调不做事就一直有效；原版背心靠 `poppable` 自毁，其他物品需自己扣燃料/耐久或移除组件。[balloonvest](../../../raw/current/scripts/prefabs/balloonvest.lua#L40-L42)
- 没有网络同步：组件只在服务端被添加和读取，客户端不会看到字段；不要把表现逻辑放在这里。[添加](../../../raw/current/scripts/prefabs/balloonvest.lua#L67-L79)

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 处理范围：锚点重定位 7 处（balloonvest +3、SGwilson washed_ashore 段 +88，区间文本逐段比对一致）；结论修订：无。balloonvest 受击爆裂监听改存 _onownerattackedfn 并加 ShouldProcOnAttackedOrBlocked 门控，不在本页溺水回调结论范围内；击退链 nointerrupt／superheavyarmor（SB-17 域 3）不触及本页 washed_ashore→TakeDrowningDamage 主张。
- 依据：player-hubs hunk 地图（SB-17）＋变更清单命中 prefabs/balloonvest.lua、stategraphs/SGwilson.lua。

- 源码审阅：2026-10-21，`steam-buildid:25643519`。覆盖 `flotationdevice.lua` 全文、`drownable` 的筛选与结算、`inventory:FindItems`、`equippable:IsEquipped`、`balloonvest` 宿主与 `SGwilson` 的调用时机；共 6 个源码文件。未运行游戏。
- 游戏运行：未运行。
- 结构检查：`validate_page.py` 通过（0 issue，行数与字节均在上限内）；`validate_source_links.py --strict` 通过（26 处源码引用、0 error／0 warning）；`parser_probe --syntax-only` 通过（片段临时文件已删除）。

- 2026-10-28 build 26230496：SGwilson washed_ashore 态补 lyingdown 标签与时间线（SB-3），锚点 +66；沉没结算链未变。
相关：[[reference/components/drownable]]、[[reference/components/inventoryitem]]、[[reference/components/equippable]]。
