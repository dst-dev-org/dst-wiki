---
title: "boatcannonuser"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "玩家端船炮瞄准状态组件：服务端经网络字段跟踪所瞄准的船炮，客户端本地渲染准星与射程特效并驱动状态图收尾。"
aliases: ["船炮瞄准", "boatcannonuser", "船炮操作者", "cannon 网络同步"]
tags: [component, server-client, network, callback]
dst_build: "steam-buildid:26230496"
sources: ["components/boatcannonuser.lua", "prefabs/player_common.lua", "prefabs/player_classified.lua", "actions.lua", "components/boatcannon.lua", "components/playercontroller.lua", "stategraphs/SGwilson.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# boatcannonuser

## 快速判断

- 职责与适用：所有玩家的通用组件（所有机器都会安装，组件内按 `ismastersim` 分支），跟踪"该玩家正在瞄准哪门船炮"；服务端权威，客户端经 `player_classified.cannon` 网络实体同步。[安装](../../../raw/current/scripts/prefabs/player_common.lua#L2521) [网络字段](../../../raw/current/scripts/prefabs/player_classified.lua#L1637)
- 优先使用：服务端读／改瞄准走 `GetCannon()`／`SetCannon(cannon)`；原版入口是 `ACTIONS.BOAT_CANNON_START_AIMING` 与 `BOAT_CANNON_SHOOT`。[开炮](../../../raw/current/scripts/components/boatcannonuser.lua#L145-L177) [行动](../../../raw/current/scripts/actions.lua#L5074-L5080)
- 重要副作用：`SetCannon` 联动 `boatcannon` 组件的 StartAiming／StopAiming、重设炮身朝向、监听炮的 onremove；客户端切炮时清理本地准星与 `cannon_aoe_range_fx`。停止瞄准时若状态图带 `is_using_cannon` 标签则强转 `aim_cannon_pst`。[炮联动](../../../raw/current/scripts/components/boatcannonuser.lua#L154-L176) [收尾](../../../raw/current/scripts/components/boatcannonuser.lua#L179-L184)
- 修改边界：本地瞄准特效只对 `ThePlayer` 生效（其他玩家不渲染）；依赖 `player_classified` 与玩家状态图，不适合非玩家实体。[本地分支](../../../raw/current/scripts/components/boatcannonuser.lua#L86-L90)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `GetCannon()` | `-> cannon 实体或 nil` | 双端可用，读 `classified.cannon:value()`；`playeractionpicker`、`playercontroller` 也用它判断是否正在瞄准。[实现](../../../raw/current/scripts/components/boatcannonuser.lua#L54-L56) [消费者](../../../raw/current/scripts/components/playeractionpicker.lua#L166-L167) |
| `SetCannon(cannon)` | `cannon: 实体或 nil；仅主模拟` | 设定/清除瞄准目标：写网络字段、与旧炮解绑（StopAiming、移除 onremove 监听）、对新炮 StartAiming 并按玩家位置算初始朝向，最后触发客户端 `OnCannonChanged`。[实现](../../../raw/current/scripts/components/boatcannonuser.lua#L145-L177) [初始朝向](../../../raw/current/scripts/components/boatcannonuser.lua#L49-L52) |
| `GetReticule()` | `-> reticule 组件或 nil` | 仅本地玩家瞄准状态下非空（读 `self.aimingcannon`）；playercontroller 用它取投掷/射击落点准星。[实现](../../../raw/current/scripts/components/boatcannonuser.lua#L62-L64) [消费者](../../../raw/current/scripts/components/playercontroller.lua#L4827-L4828) |
| `GetAimPos()` | `-> 坐标或 nil` | 同样依赖本地 `self.aimingcannon`；本快照内无原版调用（grep 全库仅组件自身），服务端恒 nil。[实现](../../../raw/current/scripts/components/boatcannonuser.lua#L58-L60) |
| `OnCannonChanged(cannon)` | `cannon: 实体或 nil` | 网络变化入口：仅 `ThePlayer` 处理；延迟一帧创建准星＋射程特效（挂到炮所在平台），切炮时清理旧特效。[实现](../../../raw/current/scripts/components/boatcannonuser.lua#L86-L115) [延迟建特效](../../../raw/current/scripts/components/boatcannonuser.lua#L66-L84) |
| `AttachClassified`／`DetachClassified` | 客户端生命周期 | 构造时绑定 `inst.player_classified`；classified 移除时解绑并清本地瞄准状态。服务端用 `SetClassified` 注入。[实现](../../../raw/current/scripts/components/boatcannonuser.lua#L34-L44) |

## 典型接入

### 服务端强制停止玩家的船炮瞄准

- 前提与选择：任何主模拟服务器代码可调用；原版射击完成、炮被移除、`boatcannon` 组件失去操作者时都走 `SetCannon(nil)`，因此这是唯一与原版一致的撤销路径。[炮侧清理](../../../raw/current/scripts/components/boatcannon.lua#L66-L67)
- 生效与撤销：清网络字段、解绑炮回调、`StopAiming`；若玩家状态图仍在瞄准态则转 `aim_cannon_pst` 收尾。[收尾](../../../raw/current/scripts/components/boatcannonuser.lua#L179-L184) [状态定义](../../../raw/current/scripts/stategraphs/SGwilson.lua#L13250)

```lua
local function StopAimingCannon(player)
    if not TheWorld.ismastersim then
        return
    end
    local user = player.components.boatcannonuser
    if user ~= nil and user:GetCannon() ~= nil then
        user:SetCannon(nil)
    end
end
```

- 调整与验证：不要直接删 `boatcannon` 或传送玩家绕过 `SetCannon(nil)`——原版依赖它解除 `boatcannon.operator` 并复位状态图；射击动作的完整时序见 `BOAT_CANNON_SHOOT.fn`。[射击](../../../raw/current/scripts/actions.lua#L5085-L5095)

## 常见陷阱与限制

- **构造里的 `self.aiming_cannon` 是死字段：** 真实状态字段是 `self.aimingcannon`（无下划线），且只在本机玩家客户端被 `OnCannonChanged` 赋值；读错字段或读服务端都得到 nil。[构造](../../../raw/current/scripts/components/boatcannonuser.lua#L11-L14) [赋值点](../../../raw/current/scripts/components/boatcannonuser.lua#L114)
- **客户端事件链在 player_classified：** classified 监听 `cannondirty` 后向玩家推 `aimingcannonchanged`，组件再调 `OnCannonChanged`；绕过 classified 直接改值不会触发本地特效。[中继](../../../raw/current/scripts/prefabs/player_classified.lua#L704) [监听](../../../raw/current/scripts/components/boatcannonuser.lua#L27)
- **OnCannonChanged 的 ThePlayer 门禁：** 多人客户端只有本地玩家实例渲染瞄准特效；对其他玩家的 `cannon` 字段变化只更新数据。[门禁](../../../raw/current/scripts/components/boatcannonuser.lua#L87-L90)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（186 行），核验 player_common 安装段、player_classified 网络字段、actions/boatcannon/playercontroller/playeractionpicker/SGwilson 消费点。
- 结构检查：2026-10-28 validate_page.py 通过（本页）；validate_source_links.py --strict 本批 16 页通过。
- 游戏运行：未运行。

相关：[[reference/components/boatcannon]]
