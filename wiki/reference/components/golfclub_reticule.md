---
title: "golfclub_reticule"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "高尔夫球杆的网络同步瞄准组件：目标锁定与蓄力进度走 netvar，客户端据此增删 reticule 显示与范围圈，挥杆按蓄力读数加速。"
aliases: ["golfclub_reticule", "高尔夫瞄准环", "蓄力瞄准组件"]
tags: [component, network-sync, ui]
dst_build: "steam-buildid:26230496"
sources: ["components/golfclub_reticule.lua", "prefabs/carnival_golfclub.lua", "components/golfclub.lua", "actions.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# golfclub_reticule

## 快速判断

- 职责与适用：装在 `carnivalgame_golfclub`（文件 carnival_golfclub.lua）物品上、**主客机都装**（加在 SetPristine 之前）；管“瞄准谁 + 蓄力多少”的网络状态与客户端瞄准环显示。击球判定、球飞行归 [[reference/components/golfclub]] 与 [[reference/components/golfable]]。[安装](../../../raw/current/scripts/prefabs/carnival_golfclub.lua#L55) [分注](../../../raw/current/scripts/prefabs/carnival_golfclub.lua#L57-L61)
- 优先使用：服务端设目标用 `SetTarget(target)`；蓄力开始由 `GOLF_START_CHARGING` 动作调 `StartCharging(doer,pt)`（[调用](../../../raw/current/scripts/actions.lua#L7094-L7095)）；读蓄力用 `CalculateChargingScale()`/`IsMaxCharge()`。[服务端接口](../../../raw/current/scripts/components/golfclub_reticule.lua#L362-L375)
- 重要副作用：客户端会在球杆实体上**动态增删 `reticule` 组件**（复用船炮瞄准代码），并生成/移除 `cannon_aoe_range_fx` 范围圈与 `golfclub_reticuleping` 命中提示；这些实体不在服务端存在。[动态 reticule](../../../raw/current/scripts/components/golfclub_reticule.lua#L220-L233)
- 修改边界：两个 netvar（target、chargeticks）是唯一网络面；组件无 OnSave（蓄力态不持久化），物品卸下/收起由 golfclub 的 StopAiming 清目标。[卸下清目标](../../../raw/current/scripts/prefabs/carnival_golfclub.lua#L34-L35)

## 关键接口

| 接口 | 条件、作用与证据 |
|---|---|
| `SetTarget(target)` | 服务端：变更 netvar target 并对持有者立即 OnTargetDirty；target=nil 结束瞄准。[实现](../../../raw/current/scripts/components/golfclub_reticule.lua#L366-L375) [golfclub 调用](../../../raw/current/scripts/components/golfclub.lua#L38-L40) |
| `StartCharging(owner,pt)` | 服务端或有 HUD 的客户端：切 reticule prefab 为 `golfclub_reticulecharging`、记预测落点，chargeticks 置 0 启动更新；仅 target 存在时生效。[实现](../../../raw/current/scripts/components/golfclub_reticule.lua#L257-L296) |
| `OnUpdate(dt)` | 主模拟逐 tick 自增 chargeticks（每 15 tick 强制 netvar 同步一次），到 `GOLF_MAX_CHARGE_TICKS`(45≈1.5 秒) 停止；客户端只做 set_local 预测、等同步校准。[实现](../../../raw/current/scripts/components/golfclub_reticule.lua#L315-L347) [上限](../../../raw/current/scripts/tuning.lua#L9565-L9566) |
| `CalculateChargingScale()` / `IsMaxCharge()` | 蓄力读数 0～1 与满蓄判断；挥杆时 golfclub 用它作速度倍率（[消费](../../../raw/current/scripts/components/golfclub.lua#L70-L76)），SGwilson 以 IsMaxCharge 决定挥杆动画（[消费](../../../raw/current/scripts/stategraphs/SGwilson.lua#L28624-L28629)）。[实现](../../../raw/current/scripts/components/golfclub_reticule.lua#L302-L313) |
| `GetTarget()` | 当前瞄准目标（服务端动作筛选与控件提示都用它；[动作筛选](../../../raw/current/scripts/components/playeractionpicker.lua#L191)）。[实现](../../../raw/current/scripts/components/golfclub_reticule.lua#L298-L300) |
| `CancelTarget_Client()` | 客户端本地清目标（set_local + 本地 OnTargetDirty），用于动作被打断时的即时反馈，不等服务器。[实现](../../../raw/current/scripts/components/golfclub_reticule.lua#L349-L357) [调用](../../../raw/current/scripts/stategraphs/SGwilson_client.lua#L8674-L8675) |

### 瞄准环的客户端重建

target 变脏时客户端 `OnTargetDirty`：有目标→放范围圈（0.4 缩放、跟随目标平台）+ 给球杆加 reticule 组件（`golfclub_reticule_fx` 预制件，鼠标/手柄双定位函数，位置限死在玩家朝向 ±45° 扇形内，鼠标超出 4.4 距离后带摆动）；无目标→若正处蓄力（`golf_charging` 标签且 ticks≠63）先放命中 ping，再删 reticule 组件（[实现](../../../raw/current/scripts/components/golfclub_reticule.lua#L178-L234)，[扇形钳制](../../../raw/current/scripts/components/golfclub_reticule.lua#L47-L75)、[鼠标摆动](../../../raw/current/scripts/components/golfclub_reticule.lua#L76-L122)、[弧度调参](../../../raw/current/scripts/tuning.lua#L9565)）。哨兵值 `CHARGETICKS_OFF=63` 表示“未在蓄力”（[哨兵](../../../raw/current/scripts/components/golfclub_reticule.lua#L10)）。

## 典型接入

### 读取蓄力程度的服务端逻辑

- 前提与选择：服务端、玩家手持原版球杆或复刻了本组件的自定义杆。本组件是原版击球链的内置环节，模组直接读数即可与原版表现一致；改瞄准手感需要替换客户端定位函数，工程量大且跨输入设备，不建议。[读数接口](../../../raw/current/scripts/components/golfclub_reticule.lua#L302-L313)
- 生效与撤销：读数仅在有 target/蓄力时有意义；未蓄力时 scale≈0（chargeticks=63 时按公式会 >1，被 min(1,…) 钳到 1——**未蓄力时不要当 0 用**）。[钳制](../../../raw/current/scripts/components/golfclub_reticule.lua#L302-L304)

```lua
-- 服务端：读取玩家手持球杆的当前蓄力程度
local club = doer.components.inventory:GetEquippedItem(EQUIPSLOTS.HANDS)
if club ~= nil and club.components.golfclub_reticule ~= nil then
    local scale = club.components.golfclub_reticule:CalculateChargingScale()
    -- scale ∈ [0,1]；仅在开始蓄力后有实际意义
end
```

- 调整与验证：蓄力上限改 `TUNING.GOLF_MAX_CHARGE_TICKS`（帧数制，45=1.5 秒）；验证观察满蓄时长与挥杆速度变化（[挥杆消费](../../../raw/current/scripts/components/golfclub.lua#L70-L76)）。

## 常见陷阱与限制

- **把客户端动态 reticule 当常驻组件：** `inst.components.reticule` 只在客户端瞄准期间存在于球杆上，服务端恒为 nil；服务端逻辑不应依赖它。[动态增删](../../../raw/current/scripts/components/golfclub_reticule.lua#L200-L233)
- **未蓄力读数被钳到 1：** `CalculateChargingScale` 只做 min 钳制，哨兵值 63 会算出 1；判断“没蓄力”要比较 chargeticks 与 CHARGETICKS_OFF 或用动作上下文，不能只看 scale。[哨兵语义](../../../raw/current/scripts/components/golfclub_reticule.lua#L10-L24)
- **客户端预测有天花板：** 客户端每 15 tick 被服务端同步覆盖，本地自增不超过 14 tick；对蓄力时间做客户端断言（如精确读秒）会与服务器漂移。[同步策略](../../../raw/current/scripts/components/golfclub_reticule.lua#L320-L331)
- **范围圈挂在目标平台上：** `aim_range_fx` 用 `AddPlatformFollower` 跟随目标所在船/平台；自定义 target 若无平台协议，圈不会跟随移动。[平台跟随](../../../raw/current/scripts/components/golfclub_reticule.lua#L214-L217)
- **组件名与文件名不一致：** 文件是 carnival_golfclub.lua，prefab 名 `carnivalgame_golfclub`；按文件名找 prefab 会落空。[prefab 返回](../../../raw/current/scripts/prefabs/carnival_golfclub.lua#L80)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取 golfclub_reticule.lua 与 carnival_golfclub.lua，核验 netvar 协议、主客机分工、瞄准环重建、蓄力同步与全部调用点（golfclub/actions/SGwilson/SGwilson_client/playeractionpicker/controls）。未运行游戏；击球落点与球物理归 golfclub/golfball 页面。

相关：[[reference/components/golfclub]]、[[reference/components/golfable]]、[[reference/components/golfballspinner]]。
