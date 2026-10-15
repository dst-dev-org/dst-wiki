---
title: "batteryuser"
created: "2026-10-22"
updated: "2026-10-28"
type: component
summary: "从 battery 电源取电：计算倍率、可选部分充电、接收成功后才调用电源消费。"
aliases: ["接收电源充电", "晨星充电", "夜棍充电", "WX78电击充能", "batteryuser"]
tags: [component, callback, energy, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/batteryuser.lua", "components/battery.lua", "prefabs/nightstick.lua", "prefabs/wx78.lua", "actions.lua", "componentactions.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# batteryuser

## 快速判断

- 职责与适用：挂在实际接收电量的角色或物品上；`ChargeFrom(source)` 协调 [[reference/components/battery]] 的检查、接收回调与消费，不内置电量容器。[实现](../../../raw/current/scripts/components/batteryuser.lua#L1-L50)
- 优先使用：原版角色／可充物品取电优先沿用 `CHARGE_FROM`；它验证电源并优先选择库存物品作为接收者，其次才是执行者。[动作](../../../raw/current/scripts/actions.lua#L5858-L5874)
- 重要副作用：接收回调返回成功后才调用电源 `OnUsed`；回调未显式返回成功会阻断消费，即使回调已经修改了接收者。[顺序](../../../raw/current/scripts/components/batteryuser.lua#L31-L48)
- 修改边界：原夜棍在 pristine 前加标签、主模拟端挂组件；组件自己不做距离、权限、目标有效性或存读检查。薇诺娜电路持续供电见 [[reference/components/battery#原版宿主与消费边界]]。[宿主](../../../raw/current/scripts/prefabs/nightstick.lua#L140-L152) [挂载](../../../raw/current/scripts/prefabs/nightstick.lua#L180-L189)

## 关键接口

| 接口／字段 | 签名与约束 | 条件、作用与证据 |
|---|---|---|
| `SetChargeMultFn(fn)` | `fn(user,source) → mult` | 默认 nil；返回值传给源和接收回调。数值含义由双方约定，不自动夹限。[设置](../../../raw/current/scripts/components/batteryuser.lua#L17-L19) [消费链](../../../raw/current/scripts/components/batteryuser.lua#L31-L48) |
| `SetAllowPartialCharge(allow)` | 布尔，默认 false | 仅 `mult` 为真且 allow 开启时，请电源 `ResolvePartialChargeMult` 降低倍率。Lua 中 0 也会进入此分支，接收端应拒绝无效倍率。[初始化](../../../raw/current/scripts/components/batteryuser.lua#L1-L11) [设置／调用](../../../raw/current/scripts/components/batteryuser.lua#L25-L36) |
| `SetOnBatteryUsedFn(fn)` | `fn(user,source,mult) → result,reason` | 电源允许后调用；必须返回成功才能扣源。未设置时沿用源检查的结果，可能只消费而没有任何接收效果。[设置](../../../raw/current/scripts/components/batteryuser.lua#L21-L23) [调用](../../../raw/current/scripts/components/batteryuser.lua#L36-L47) |
| `ChargeFrom(source)` | 返回 `result,reason` | 直接访问 `source.components.battery`，不会检查其存在；直接调用者须在服务端先验证目标。完整动作已有此门禁。[方法](../../../raw/current/scripts/components/batteryuser.lua#L31-L48) [动作](../../../raw/current/scripts/actions.lua#L5858-L5874) |
| `OnRemoveFromEntity()` | 移除组件时 | 只删 `batteryuser` 标签；不撤销既有充电、不清宿主的燃料／模块。[移除](../../../raw/current/scripts/components/batteryuser.lua#L13-L15) |

流程为：取倍率 → 可选部分倍率修正 → 电源 `CanBeUsed` → 接收回调 → 成功才调电源 `OnUsed` → 返回结果。电源消费回调的返回值被忽略，无事务回滚。[接收流程](../../../raw/current/scripts/components/batteryuser.lua#L31-L48) [源消费](../../../raw/current/scripts/components/battery.lua#L43-L47)

### 角色电击充能与物品比例充能

- 原夜棍注册名 `nightstick`：倍率为缺失燃料比例并夹到 0～1，允许部分充电；接收回调拒绝满电或非正倍率，否则将比例加到 `fueled`，播放电击特效并返回 true。[回调](../../../raw/current/scripts/prefabs/nightstick.lua#L107-L122) [挂载](../../../raw/current/scripts/prefabs/nightstick.lua#L180-L189) [注册](../../../raw/current/scripts/prefabs/nightstick.lua#L198)
- 原 `wx78` 经 `MakePlayerCharacter` 注册，仅配置接收回调，没有倍率函数／部分充电开关。满实际电量返回 `CHARGE_FULL`；成功则回血、扣理智、给 `upgrademoduleowner` 加 1 格并立即发 `electrocute`。这不是夜棍的燃料百分比；电量／有效值与模块激活复用 [[reference/components/upgrademoduleowner]]。[挂载](../../../raw/current/scripts/prefabs/wx78.lua#L783-L784) [接收](../../../raw/current/scripts/prefabs/wx78.lua#L458-L473) [注册](../../../raw/current/scripts/prefabs/wx78.lua#L843)
- `battery` 的场景动作要求电源／执行者标签；物品对目标的 USEITEM 和 EQUIPPED 路径都要求右键以及双方标签。可见提示不预检当前电量。[场景](../../../raw/current/scripts/componentactions.lua#L179-L183) [物品](../../../raw/current/scripts/componentactions.lua#L1058-L1062) [装备](../../../raw/current/scripts/componentactions.lua#L2240-L2244)

## 典型接入

### 夜棍每次取电最多补 25 个百分点，保留原部分充电与消费

目标为模组根 `modmain.lua`，模组入口直接加载；无需新增资源。只包装原夜棍的倍率函数，沿用原接收回调、`allowpartialcharge=true` 和电源消费链；不把 `fueled` 直接写满。[原倍率／接收](../../../raw/current/scripts/prefabs/nightstick.lua#L107-L122) [原挂载](../../../raw/current/scripts/prefabs/nightstick.lua#L186-L189)

```lua
AddPrefabPostInit("nightstick", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local user = inst.components.batteryuser
    if user == nil or user.chargemultfn == nil then return end
    local oldchargemult = user.chargemultfn
    user:SetChargeMultFn(function(item, source)
        local mult = oldchargemult(item, source)
        if mult == nil then return nil end
        return math.min(mult, 0.25)
    end)
end)
```

`0.25` 是 25 个百分点的倍率上限，可按需求修改为 0～1 数值；缺电不足 25% 时仅补缺口，电源不足时仍可按原协议降低倍率。原接收回调在满电时拒绝，拒绝后不消费源；原版电源的倍率消费单位由各宿主定义，不能解释成一律扣 25 秒。

覆盖该世界加载的夜棍实例，不影响 WX-78；不新增任务或事件，无需额外清理。停用模组并重载恢复原倍率，已获得燃料由原宿主保存，不回滚已充电结果。未做游戏运行验证。

## 常见陷阱与限制

- **接收回调漏写 return true**：结果被 nil 覆盖，源不扣费，接收端却可能已经改电量。先检查满电／无效倍率，再改变状态并明确返回成功。[流程](../../../raw/current/scripts/components/batteryuser.lua#L36-L47) [原夜棍](../../../raw/current/scripts/prefabs/nightstick.lua#L112-L122)
- **把 mult=nil 当作 0**：WX-78 正常不提供倍率，源据此走整次取电分支；不得全局强制改成零或百分比。组件也不会限制任意自定义倍率。[WX 挂载](../../../raw/current/scripts/prefabs/wx78.lua#L783-L784) [倍率传递](../../../raw/current/scripts/components/batteryuser.lua#L31-L39)
- **接收者就是操作玩家**：夜棍交互时传给源的 user 是物品，不能无检查访问 `user.userid`／人物三维；动作优先取 invobject。[动作](../../../raw/current/scripts/actions.lua#L5863-L5873)
- **只加标签就有充电效果**：标签仅帮助动作发现；服务端仍需组件和接收回调。自定义 prefab 的标签应在 pristine 前声明，原夜棍提供可参照边界。[动作](../../../raw/current/scripts/actions.lua#L5858-L5874) [原标签](../../../raw/current/scripts/prefabs/nightstick.lua#L140-L152)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 中变更文件 actions.lua、componentactions.lua、prefabs/wx78.lua（batteryuser.lua／battery.lua／nightstick.lua 本版未变）对照 build-26148036 更新报告核验：CHARGE_FROM 定义与 fn、SCENE／USEITEM／EQUIPPED batteryuser 候选均无语义变化，wx78 变更（eater 饮食迁移）不触及充电链；锚点重定位 5 处、结论无修订；未覆盖：运行验证。
- 源码审阅：2026-10-22，Steam buildid 25643519；覆盖组件完整实现、源默认回调、CHARGE_FROM 选择／标签动作入口、夜棍倍率／接收／注册及 WX-78 单次电击充能。模块电量含义复用同 build 的 upgrademoduleowner 页；未核验全部无人机／技能／SG 电击表现。
- 结构检查：本页格式、大小、来源与本地源码锚点；本页完整 `modmain.lua` 示例经 Full Moon Lua 5.1 syntax-only 检查。
- 游戏运行：未运行；需实测不同余量电源、满电、25% 上限、读档及其他模组对倍率函数的组合。
