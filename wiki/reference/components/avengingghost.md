---
title: "avengingghost"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "玩家复仇幽灵计时组件：Wendy 技能解锁后玩家变鬼时进入限时强化复仇状态，服务端计时并经网络字段同步给 HUD 徽章。"
aliases: ["复仇幽灵", "avengingghost", "Wendy 变鬼强化", "复仇倒计时"]
tags: [component, server-client, lifecycle, callback]
dst_build: "steam-buildid:26230496"
sources: ["components/avengingghost.lua", "prefabs/player_common.lua", "prefabs/skilltree_wendy.lua", "widgets/statusdisplays.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# avengingghost

## 快速判断

- 职责与适用：玩家通用组件（所有机器安装，服务端跑逻辑）；当玩家变鬼且附近有激活了 Wendy 技能 `wendy_avenging_ghost` 的玩家时，给鬼魂 15 秒强化复仇期（伤害随时段变化、附加特效与光环）。[安装](../../../raw/current/scripts/prefabs/player_common.lua#L2525) [技能判定](../../../raw/current/scripts/components/avengingghost.lua#L72-L88)
- 优先使用：读剩余时间／上限用 `GetTime()`／`GetMaxTime()`（双端）；自然触发走 `ms_becameghost` → `ShouldAvenge` → `StartAvenging`。[触发](../../../raw/current/scripts/components/avengingghost.lua#L43-L52)
- 重要副作用：开始复仇会暂存并随时段改写 `combat.defaultdamage`、生成 `abigail_attack_fx` 子实体、开光环、播循环音效并改玩家染色；结束或复活（`ms_respawnedfromghost`）全部还原。[开启](../../../raw/current/scripts/components/avengingghost.lua#L110-L133) [还原](../../../raw/current/scripts/components/avengingghost.lua#L135-L159)
- 修改边界：全部状态变更仅主模拟；客户端只有三个只读网络字段与 HUD 徽章消费。被区域攻击命中会刷新 1.1 秒减速计时器，使倒计时按 0.5 倍速流失。[减速](../../../raw/current/scripts/components/avengingghost.lua#L161-L169)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `GetTime()`／`GetMaxTime()` | `-> number` | 双端读 `_avengetime`／`_maxtime` 网络字段；上限构造即设 15（`MAX_TIME`）。HUD 徽章每帧取值。[实现](../../../raw/current/scripts/components/avengingghost.lua#L61-L67) [徽章](../../../raw/current/scripts/widgets/statusdisplays.lua#L986-L991) |
| `GetSymbol()` | `-> number` | 读 `_symbol` 网络哈希；**本快照内无任何 `:set` 调用，恒返回初始 0**。[实现](../../../raw/current/scripts/components/avengingghost.lua#L57-L59) |
| `ShouldAvenge()` | `-> boolean；仅主模拟` | 找 `PLAYER_CAMERA_SEE_DISTANCE` 内持有技能树激活项 `wendy_avenging_ghost` 的玩家。[实现](../../../raw/current/scripts/components/avengingghost.lua#L72-L88) [技能组](../../../raw/current/scripts/prefabs/skilltree_wendy.lua#L157) |
| `StartAvenging(time)` | `time: number?；仅主模拟` | 设剩余时间（缺省 15）、备份伤害并按昼夜监听改伤、生成特效子实体、开光环并 StartUpdating；未做任何技能检查（由调用方保证）。[实现](../../../raw/current/scripts/components/avengingghost.lua#L110-L133) |
| `StopAvenging()` | `仅主模拟` | 仅当 `olddamage` 存在才执行还原分支；无论如何 StopUpdating。[实现](../../../raw/current/scripts/components/avengingghost.lua#L135-L159) |
| `OnUpdate(dt)` | 组件更新 | 倒计时：处于 `avenging_ghost_attack` 计时器窗口内按 `dt*0.5` 流失，归零自动 StopAvenging；该计时器由 `onareaattackother` 刷新为 1.1 秒。[实现](../../../raw/current/scripts/components/avengingghost.lua#L161-L175) [计时器](../../../raw/current/scripts/components/avengingghost.lua#L35-L45) |
| `OnSave`／`OnLoad` | 标准存读钩子 | 存剩余时间；载入后延迟一帧，若实体仍是 playerghost 则以存档时间恢复复仇。[实现](../../../raw/current/scripts/components/avengingghost.lua#L47-L52) |

## 典型接入

### 客户端读取复仇倒计时

- 前提与选择：组件在玩家实体上双端存在（player_common 在 pristine 前安装），客户端直接读网络字段即可，无需 replica；原版 HUD 徽章即此用法。[公共段安装](../../../raw/current/scripts/prefabs/player_common.lua#L2525)

```lua
local function GetAvengeTimeLeft(player)
    local ghost = player.components.avengingghost
    if ghost ~= nil then
        return ghost:GetTime(), ghost:GetMaxTime()
    end
    return nil, nil
end
```

- 调整与验证：数值变化时实体上会推 `clientavengetimedirty` 事件（构造注册），UI 可监听该事件而非轮询。[脏事件](../../../raw/current/scripts/components/avengingghost.lua#L1-L3) [徽章注册](../../../raw/current/scripts/widgets/statusdisplays.lua#L402-L405)

## 常见陷阱与限制

- **_symbol 从未写入：** `avengingghost._symbol` 定义后全库无 `:set`，GetSymbol 恒 0；若模组想复用该字段驱动外观，需自己写值，不能指望原版填充。[定义](../../../raw/current/scripts/components/avengingghost.lua#L15)
- **StartAvenging 不查技能：** 技能门槛只在自然触发路径 `ShouldAvenge` 内；直接调用会无条件进入复仇态，模组复用时要自己把关。[无门禁](../../../raw/current/scripts/components/avengingghost.lua#L110-L116)
- **伤害还原依赖 olddamage：** 若在复仇期间外部又改了 `combat.defaultdamage`，StopAvenging 会整体覆盖回备份值；特效子实体挂在 `inst.ghost_attack_fx` 字段上，外部误占同名字段会被 Remove。[还原](../../../raw/current/scripts/components/avengingghost.lua#L140-L157)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（189 行），核验 player_common 安装段、skilltree_wendy 技能组与 statusdisplays 徽章消费；`_symbol` 写入经全库 grep 确认缺失。
- 结构检查：2026-10-28 validate_page.py 通过（本页）；validate_source_links.py --strict 本批 16 页通过。
- 游戏运行：未运行。

相关：[[reference/components/combat]]、[[reference/components/aura]]
