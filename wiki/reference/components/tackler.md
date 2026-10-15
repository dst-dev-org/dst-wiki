---
title: "tackler"
created: "2026-10-23"
updated: "2026-10-28"
type: component
summary: "为伍迪鹿形冲撞提供前方碰撞、践踏和悬崖探测；位移与中断清理由人物状态机负责。"
aliases: ["tackler", "鹿形冲撞", "伍迪冲锋", "冲撞砍树采矿", "冲锋边缘检测"]
tags: [component, actions, state-machine, callback]
dst_build: "steam-buildid:26230496"
sources: [components/tackler.lua, prefabs/woodie.lua, actions.lua, stategraphs/SGwilson.lua, stategraphs/SGwilson_client.lua]
reliability: verified
validation: source-reviewed
status: active
---

# tackler

## 快速判断

- 职责与适用：挂在服务端人物上，供移动中的鹿形冲撞检查可工作障碍、建筑与可攻击生物；不自动启动更新或设置速度。原版由伍迪配置、SG 每帧调用。[配置](../../../raw/current/scripts/prefabs/woodie.lua#L1827-L1837)、[SG 调用](../../../raw/current/scripts/stategraphs/SGwilson.lua#L22109-L22122)
- 优先使用：只改伍迪碰撞范围／建筑伤害倍率时调整现有组件；做武器突刺沿线结算先查 [[reference/components/aoeweapon-lunge]]，不要直接搬鹿形专用 SG。[配置接口](../../../raw/current/scripts/components/tackler.lua#L28-L47)、[原版移动](../../../raw/current/scripts/stategraphs/SGwilson.lua#L22077-L22082)
- 重要副作用：工作障碍或可攻击建筑会返回阻挡；普通生物走践踏攻击，单纯践踏不返回阻挡。共享 `ignores` 集合可避免同次冲撞反复命中。[碰撞结算](../../../raw/current/scripts/components/tackler.lua#L102-L138)
- 修改边界：`StartTackle` 仅调用宿主回调；原版回调只在 `tackle_pre` 才切 `tackle_start`。添加组件不会得到动作、动画或完整冲锋。[启动](../../../raw/current/scripts/components/tackler.lua#L20-L25)、[宿主门禁](../../../raw/current/scripts/prefabs/woodie.lua#L1544-L1550)

## 关键接口

| 接口／字段 | 参数／默认 | 条件、作用与证据 |
|---|---|---|
| `SetOnStartTackleFn(fn)`／`StartTackle()` | `fn(inst)`；返回其结果，无回调为 false | 动作 `TACKLE.fn` 只转调该入口，不自行运动。[组件](../../../raw/current/scripts/components/tackler.lua#L20-L25)、[动作](../../../raw/current/scripts/actions.lua#L4595-L4598) |
| `SetDistance(distance)`／`SetRadius(radius)` | 默认 .5／.75，世界距离 | 检测圆心位于朝向前 distance，精测半径为 radius 加目标物理半径；粗查半径固定为 radius+3，不是沿整段路径扫掠。[默认](../../../raw/current/scripts/components/tackler.lua#L5-L10)、[几何](../../../raw/current/scripts/components/tackler.lua#L57-L72) |
| `AddWorkAction(action,amount)` | `ACTIONS` 对象；amount 默认1 | 将 `action.id.."_workable"` 加入候选标签并存工作量；重复设置更新数量。仍排除 DIG、NPC 等默认禁用标签及 smallcreature 工作分支。[注册](../../../raw/current/scripts/components/tackler.lua#L40-L47)、[筛选](../../../raw/current/scripts/components/tackler.lua#L74-L87) |
| `SetStructureDamageMultiplier(mult)` | 默认2 | 仅攻击阻挡建筑时临时向 combat 添加 `"tackler"` 倍率，攻击后移除；不乘工作量或普通践踏伤害。[设置](../../../raw/current/scripts/components/tackler.lua#L36-L37)、[攻击](../../../raw/current/scripts/components/tackler.lua#L109-L114) |
| `CheckCollision(ignores)` | 可选实体键集合；返回 bool | 筛选最近可工作障碍／建筑，结算它及更靠前的普通生物；选中的阻挡对象及通过复检的践踏对象先写集合，再执行回调／工作或攻击。返回值只表示是否存在阻挡目标，集合登记不证明造成伤害。[目标选择](../../../raw/current/scripts/components/tackler.lua#L66-L99)、[结算](../../../raw/current/scripts/components/tackler.lua#L102-L138) |
| `SetOnCollideFn(fn)`／`SetOnTrampleFn(fn)` | `fn(inst,target)` | 分别在障碍工作／建筑攻击前、普通生物攻击前调用；无取消返回值处理，回调不要破坏后续仍直接访问的组件。[设置](../../../raw/current/scripts/components/tackler.lua#L49-L54)、[调用](../../../raw/current/scripts/components/tackler.lua#L102-L138) |
| `SetEdgeDistance(distance)`／`CheckEdge()` | 默认5；返回 bool | 前方及左右30°三个采样点都判为不在陆地或地面受阻，才返回 true；不是每帧自动刹停。[默认](../../../raw/current/scripts/components/tackler.lua#L17)、[检测](../../../raw/current/scripts/components/tackler.lua#L141-L162) |

### 命中筛选与持续状态

默认候选包含 `_combat`，禁用 `NPC_workable`／`DIG_workable`／FX／NOCLICK／DECOR／INLIMBO；构造时若关闭 PVP，再排除 player。战斗目标需 combat、未死亡 health，施撞者 combat 可攻击、非盟友且未被 `TargetForceAttackOnly` 拦截；工作分支优先于战斗分支。[构造](../../../raw/current/scripts/components/tackler.lua#L8-L13)、[分支](../../../raw/current/scripts/components/tackler.lua#L74-L96)

组件没有保存、更新或退出恢复方法；一次冲撞的 `targets` 集合和边缘计数由 SG 保存、跨循环状态传递。`tackle_start`／`tackle` 检测到阻挡转 `tackle_collide`，持续边缘检测成立后转 `tackle_stop`；普通生物践踏不会自行终止冲锋。[启动状态](../../../raw/current/scripts/stategraphs/SGwilson.lua#L22109-L22135)、[循环](../../../raw/current/scripts/stategraphs/SGwilson.lua#L22205-L22240)

## 典型接入

### 只降低原版伍迪冲撞对可攻击建筑的额外倍率

目标模组 `modmain.lua`；沿用原版伍迪与鹿形资源、动作、伤害和 SG，仅在服务端把建筑冲撞倍率从2改为1，不改变砍树／采矿／锤击的工作量，也不改普通生物伤害。[原版配置](../../../raw/current/scripts/prefabs/woodie.lua#L1827-L1837)、[倍率使用](../../../raw/current/scripts/components/tackler.lua#L109-L134)

```lua
AddPrefabPostInit("woodie", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    if inst.components.tackler ~= nil then
        inst.components.tackler:SetStructureDamageMultiplier(1)
    end
end)
```

`1` 是 combat 的乘数，不是固定伤害，也不会让原本不能攻击的建筑变为可攻击。实例创建／重载时应用；组件不保存该配置，停用模组后新建实例恢复原版值。其他模组后续赋值可能覆盖本设置。[筛选](../../../raw/current/scripts/components/tackler.lua#L81-L92)、[组件完整实现](../../../raw/current/scripts/components/tackler.lua#L1-L165)

### 参考完整冲锋时必须一起保留的宿主协作

- **入口与预测：** 鹿形点选器提供 TACKLE；服务端和客户端 SG 均映射到 `tackle_pre`，客户端仅预演并等待服务端状态，实际启动由服务端缓冲动作驱动。[点选](../../../raw/current/scripts/prefabs/woodie.lua#L292-L308)、[服务端映射](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1519)、[客户端映射](../../../raw/current/scripts/stategraphs/SGwilson_client.lua#L789)、[预测状态](../../../raw/current/scripts/stategraphs/SGwilson_client.lua#L6340-L6365)
- **运动与退出：** `tackle_start` 开马达、临时忽略 CHARACTERS 碰撞，随后跨状态传递去重表和尾迹任务。真正退出时取消任务、停止马达、恢复碰撞与位置；循环退出还清 `busyremoteoverridelocomote`。正常交接靠 `tackling`／`stopping` 标志避免过早清理。[进入](../../../raw/current/scripts/stategraphs/SGwilson.lua#L22077-L22101)、[启动退出](../../../raw/current/scripts/stategraphs/SGwilson.lua#L22140-L22152)、[循环退出](../../../raw/current/scripts/stategraphs/SGwilson.lua#L22244-L22257)
- **结束表现：** 碰撞进入反震动画，安全停止走逐步减速，结束状态负责重新显示动作；原版碰撞回调还产生 FX／镜头震动并设置伍迪 grogginess。[回调](../../../raw/current/scripts/prefabs/woodie.lua#L1552-L1569)、[碰撞结束](../../../raw/current/scripts/stategraphs/SGwilson.lua#L22261-L22292)、[减速与退出](../../../raw/current/scripts/stategraphs/SGwilson.lua#L22296-L22339)

以上是原版协作入口，不是可直接复制到任意生物的完整 SG。

## 常见陷阱与限制

- **把 CheckCollision 当纯查询：** 调用立即执行回调、工作与攻击；另一次“确认是否碰撞”的调用可能再结算。沿用同次冲锋的 ignores，并由调用方控制频率。[结算实现](../../../raw/current/scripts/components/tackler.lua#L102-L138)
- **用回调返回 false 取消命中：** 结算不检查返回值；若要改筛选应改对应判定路径，不能把回调当否决接口。[调用位置](../../../raw/current/scripts/components/tackler.lua#L106-L114)、[践踏](../../../raw/current/scripts/components/tackler.lua#L128-L134)
- **把边缘 true 当安全自动停止：** 仅三个离散采样点同时异常才成立，SG 还连续计数后停；高速移动／自定义地图仍需实测，组件不保证不落水。[采样](../../../raw/current/scripts/components/tackler.lua#L149-L162)、[SG 处理](../../../raw/current/scripts/stategraphs/SGwilson.lua#L22114-L22120)

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 处理范围：锚点重定位 18 处（SGwilson／SGwilson_client／actions／woodie 行号迁移，区间文本逐段比对一致；woodie 本版改动集中在 SetWereFighter，不触及冲撞配置与回调）；结论修订：无，仅锚点迁移。
- 依据：枢纽 diff 审阅（actions.lua／componentactions.lua hunk 地图）＋变更清单命中 actions.lua、stategraphs/SGwilson.lua、stategraphs/SGwilson_client.lua、prefabs/woodie.lua。

- 源码审阅：2026-10-23，Steam buildid 25643519；完整 tackler、伍迪配置／启动与表现回调、TACKLE 分发、人物 SG 冲锋及退出、客户端预测。示例仅修改已有建筑伤害倍率，不涵盖自定义冲锋工程或所有 combat 内部效果。
- 结构检查：本页元数据、链接、大小与源码引用定向检查通过；完整 modmain 示例通过现有 Full Moon／Lua 5.1 语法检查。
- 游戏运行：未运行；冲撞建筑／工作对象差异、PVP、落水与联机中断清理仍需实测。
