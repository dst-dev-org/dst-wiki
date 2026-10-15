---
title: "bathingpool"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "泡澡池占用协议组件：管理实体（玩家/寄居蟹等）入池占位、池边定位与离开，配合 SG 状态标记 occupying_bathingpool 生效。"
aliases: ["泡澡池", "bathingpool", "温泉泡澡", "入池协议"]
tags: [component, state-machine, callback]
dst_build: "steam-buildid:26230496"
sources: ["components/bathingpool.lua", "prefabs/hermithotspring.lua", "prefabs/hotspring.lua", "actions.lua", "componentactions.lua", "components/moisture.lua", "stategraphs/SGwilson.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# bathingpool

## 快速判断

- 职责与适用：温泉类建筑（hermithotspring 半径 2、hotspring 半径 1.1）的占位协议：服务端维护 occupants 列表，入池走 `ACTIONS.SOAKIN` → `EnterPool`，实体 SG 状态用 `sg.statemem.occupying_bathingpool` 自证。[安装](../../../raw/current/scripts/prefabs/hermithotspring.lua#L394-L399) [行动](../../../raw/current/scripts/actions.lua#L6654-L6657)
- 优先使用：入池只用 `EnterPool(ent)`、离开只用 `LeavePool(ent)`；原版行动过滤器对任何带组件的实体给 SOAKIN 选项。[行动构造](../../../raw/current/scripts/componentactions.lua#L175-L177)
- 重要副作用：入池/离池用 `PushEventImmediate` 推 `ms_enterbathingpool`（带 dest 坐标）／`ms_leavebathingpool`，由 SGwilson、SGhermitcrab、SGwx78_possessedbody 的状态响应并回填 statemem；组件监听占用者的 onremove/newstate 自动清退。[事件](../../../raw/current/scripts/components/bathingpool.lua#L177-L187) [玩家状态](../../../raw/current/scripts/stategraphs/SGwilson.lua#L27115)
- 修改边界：不检查实体能否入水，位置合法性交给 SG 状态；组件移除（OnRemoveFromEntity）会清退全部占用者，但实体级 OnRemoveEntity 别名被注释停用。[清理](../../../raw/current/scripts/components/bathingpool.lua#L15-L28)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `EnterPool(ent)` | `-> ok, 原因?`（`"NOSPACE"`） | 满员/无空位返回 false；成功路径：算池边落点（避让已占用者，5°~45° 扫描）→ `PushEventImmediate("ms_enterbathingpool", {target, dest})` → 校验 `CheckOccupant` → `AddOccupant`。[实现](../../../raw/current/scripts/components/bathingpool.lua#L121-L188) |
| `LeavePool(ent)` | `-> boolean` | 非占用者返回 false；statemem 标记丢失（不同步）时强拆并 dev 断言；推 `ms_leavebathingpool` 后以 `CheckOccupant` 反向验证。[实现](../../../raw/current/scripts/components/bathingpool.lua#L190-L203) |
| `CheckOccupant(ent)` | `-> boolean` | 唯一权威判定：`ent.sg.statemem.occupying_bathingpool == self.inst`——SG 状态必须自行设置该标记，组件不写。[实现](../../../raw/current/scripts/components/bathingpool.lua#L105-L108) [湿度消费](../../../raw/current/scripts/components/moisture.lua#L266-L268) |
| `CheckAvailableSpot(x, z, r)` | `-> ok, blocker?` | 与现有占用者按物理半径判定重叠。[实现](../../../raw/current/scripts/components/bathingpool.lua#L110-L119) |
| `SetMaxOccupants(max)` | `max: number或 nil` | 设上限并立即清退超编占用者。[实现](../../../raw/current/scripts/components/bathingpool.lua#L38-L52) |
| `SetRadius(r)`／`GetRadius()` | `r: number` | 未设置时回退 `inst:GetPhysicsRadius(0)`。[实现](../../../raw/current/scripts/components/bathingpool.lua#L30-L36) |
| `SetOnStart/StopBeingOccupiedBy(fn)` | `fn(inst, ent)` | 占用开始/结束回调；原版两个温泉都接（做玻璃化/显隐等）。[实现](../../../raw/current/scripts/components/bathingpool.lua#L54-L60) [温泉接线](../../../raw/current/scripts/prefabs/hermithotspring.lua#L397-L398) |
| `ForEachOccupant(fn, ...)` | `fn(pool, ent, ...) -> 终止` | 倒序遍历，fn 返回真即停。[实现](../../../raw/current/scripts/components/bathingpool.lua#L97-L103) |

## 典型接入

### 让自定义实体入池泡澡

- 前提与选择：主模拟端；实体 SG 必须响应 `ms_enterbathingpool` 并在状态里设 `statemem.occupying_bathingpool = data.target`，否则 EnterPool 在校验步失败回滚（原版温泉 SG 即此约定）。[事件](../../../raw/current/scripts/components/bathingpool.lua#L177-L187)

```lua
local function SoakIn(pool, ent)
    if not TheWorld.ismastersim then
        return false
    end
    local bathingpool = pool.components.bathingpool
    if bathingpool == nil then
        return false
    end
    return bathingpool:EnterPool(ent)
end
```

- 调整与验证：入池失败返回 `false, "NOSPACE"`；离开校验失败（statemem 丢失）时组件会自愈强拆并返回 false，调用方按 false 处理即可，不必补推事件。[自愈](../../../raw/current/scripts/components/bathingpool.lua#L192-L199)

## 常见陷阱与限制

- **statemem 标记是双向契约：** 组件读 SG 写；模组给不带该约定的 SG 调 EnterPool 会永远失败（校验步回滚，且占用者未入列）。[契约](../../../raw/current/scripts/components/bathingpool.lua#L105-L108)
- **实体移除不走清理：** `OnRemoveEntity` 别名被注释（L28），整实体删除时占用者只由各自 onremove 监听解绑，`onstopbeingoccupiedby` 回调可能不触发；按需启停温泉的原版用法是 Add/RemoveComponent（走 OnRemoveFromEntity 清理）。[注释行](../../../raw/current/scripts/components/bathingpool.lua#L28) [按需启停](../../../raw/current/scripts/prefabs/hotspring.lua#L124-L133)
- **泡澡时湿度速率被豁免：** `Moisture:IsInBathingPool` 直接读同款 statemem 标记，泡澡实体不计雨湿——这是消费方约定而非本组件功能。[湿度](../../../raw/current/scripts/components/moisture.lua#L266-L268)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（215 行），核验两个温泉安装与回调接线、SOAKIN 行动链、SGwilson 事件处理与 moisture/componentactions 消费。
- 结构检查：2026-10-28 validate_page.py 通过（本页）；validate_source_links.py --strict 本批 16 页通过。
- 游戏运行：未运行。

相关：[[reference/components/moisture]]
