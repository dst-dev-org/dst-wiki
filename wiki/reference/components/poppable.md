---
title: "poppable"
created: "2026-10-23"
updated: "2026-10-28"
type: component
summary: "为气球类实体提供单次爆破回调入口，分清触发锁、攻击效果、装备清理与持久化的责任。"
aliases: ["气球爆破", "气球受击爆炸", "气球爆破回调", "一次性爆破", "poppable"]
tags: [component, item, callback, actions, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/poppable.lua", "prefabs/balloons_common.lua", "prefabs/balloon.lua", "prefabs/balloonvest.lua", "actions.lua", "stategraphs/SGcatcoon.lua", "components/combat.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# poppable

## 快速判断

- 职责与适用：给气球等目标提供“同一组件实例最多执行一次”的 `Pop()` 入口；动画、伤害、掉落及实体删除全由 `onpopfn`／宿主决定。[完整组件](../../../raw/current/scripts/components/poppable.lua#L2-L19)
- 优先使用：直接给 `onpopfn` 赋回调 `(inst)`，通过 `Pop()` 触发；没有 `SetOnPopFn`。改原版气球时优先保留原回调，不复制整套爆破与清理。[组件字段](../../../raw/current/scripts/components/poppable.lua#L2-L17) [原版配置](../../../raw/current/scripts/prefabs/balloons_common.lua#L134-L149)
- 重要副作用：先写 `popped = true` 再回调；重入／再次调用不执行回调，即使首次没有回调也会锁住。`Pop()` 无成功返回。[调用顺序](../../../raw/current/scripts/components/poppable.lua#L9-L17)
- 修改边界：原版在服务端 `MakeBalloonMasterInit` 挂组件；组件无标签、网络值、存档或自动清理方法，不能靠它给任意物品自动添加玩家爆破动作。[气球端边界](../../../raw/current/scripts/prefabs/balloon.lua#L69-L78) [共享构造](../../../raw/current/scripts/prefabs/balloons_common.lua#L134-L155)

## 关键接口

| 接口／字段 | 签名或状态 | 条件、作用与证据 |
|---|---|---|
| `onpopfn` | 可选回调 `(inst)` | 源码中的 `--self.onpopfn = nil` 是注释；原版通过直接赋字段配置，不传操作者、不消费返回值。[组件](../../../raw/current/scripts/components/poppable.lua#L2-L17) [原版赋值](../../../raw/current/scripts/prefabs/balloons_common.lua#L139-L140) |
| `popped` | 初始读取为 nil；首次 Pop 写 true | `--self.popped = false` 也是注释，不是实际初始化。门禁是 `not self.popped`，不是严格 `== false`。[字段与门禁](../../../raw/current/scripts/components/poppable.lua#L2-L17) |
| `Pop()` | 无参数、无返回值 | 没有健康值／有效性／权限检查；先置锁，再调用现有回调。回调内再次 Pop 不会重入。[实现](../../../raw/current/scripts/components/poppable.lua#L9-L17) |

### 原版触发与效果分工

- **受击致死入口：** 共享构造给气球 health 设最大生命 1、`nofadeout = true` 并监听 `death`；`ondeath` 才调用 Pop。不是 poppable 自己监听 attacked 或自动扣血。[死亡转发](../../../raw/current/scripts/prefabs/balloons_common.lua#L130-L149)
- **猫玩入口：** CATPLAYGROUND／CATPLAYAIR 优先调用 cattoy，只有没有 cattoy 才进入 poppable 分支。SGcatcoon 的地面拍打、空中扑抓状态在各自时间点执行 BufferedAction；不是通用玩家右键菜单。[动作分支](../../../raw/current/scripts/actions.lua#L3882-L3888) [空中分支](../../../raw/current/scripts/actions.lua#L3911-L3916) [SG 映射](../../../raw/current/scripts/stategraphs/SGcatcoon.lua#L5-L10) [地面执行](../../../raw/current/scripts/stategraphs/SGcatcoon.lua#L342-L364) [空中执行](../../../raw/current/scripts/stategraphs/SGcatcoon.lua#L373-L394)
- **气球背心其他入口：** `balloonvest` 的防溺水回调、fueled 耗尽回调都转 Pop；它们共享同一个一次性锁，而非各自再爆一次。[防溺水](../../../raw/current/scripts/prefabs/balloonvest.lua#L40-L42) [服务端配置](../../../raw/current/scripts/prefabs/balloonvest.lua#L67-L87) [耗尽转发](../../../raw/current/scripts/prefabs/balloons_common.lua#L189-L191)

## 典型接入

### 爆破前调整普通气球的基础伤害，保留动画、范围攻击与删除

完整代码放入现有模组 `modmain.lua`，无需新增资源，仅作用于 `balloon`。包装原回调以保持原爆破流程；10 是本例选定的战斗基础伤害，不保证最终扣血恰好 10。气球范围攻击继续走 combat 的目标过滤与伤害计算，详见 [[reference/components/combat]]。[伤害 setter](../../../raw/current/scripts/components/combat.lua#L222-L224) [无武器基础伤害](../../../raw/current/scripts/components/combat.lua#L889-L902) [范围攻击](../../../raw/current/scripts/components/combat.lua#L1312-L1339)

```lua
AddPrefabPostInit("balloon", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    local poppable = inst.components.poppable
    if poppable == nil or poppable.onpopfn == nil then
        return
    end
    local oldonpopfn = poppable.onpopfn
    poppable.onpopfn = function(balloon)
        if balloon.components.combat ~= nil then
            balloon.components.combat:SetDefaultDamage(10)
        end
        return oldonpopfn(balloon)
    end
end)
```

真实工厂 `Prefab("balloon", fn, ...)` 在服务端将共享回调配置为 `DoPop_Floating`。它先禁交互、取消持久化／拾取，播放动画与声音，然后安排延迟范围攻击、再删除实体。本例在原回调前改基础伤害，保留其延迟和清理，不另设任务、监听或存档状态。[注册与构造](../../../raw/current/scripts/prefabs/balloon.lua#L69-L78) [注册名](../../../raw/current/scripts/prefabs/balloon.lua#L102) [禁用交互](../../../raw/current/scripts/prefabs/balloons_common.lua#L27-L35) [浮空气球爆破](../../../raw/current/scripts/prefabs/balloons_common.lua#L38-L51) [伤害与删除](../../../raw/current/scripts/prefabs/balloons_common.lua#L19-L24)

`Pop` 已先置锁，再进入包装，因此多触发源不会重复执行这一包装；调用方不要再自行 `onpopfn(inst)`。关闭模组后新加载气球使用原构造；本例不热撤销现存实体的包装。若只需全生命周期改变基础伤害，可在服务端 PostInit 直接配置 combat，无须包装 Pop。

### 浮空气球与穿戴气球不要套同一清理假设

| 模式 | 实际处理与边界 | 证据 |
|---|---|---|
| `balloon` 浮空爆破 | 使用 `DoPop_Floating`，计时攻击与删除；保留原物理／动画，不由组件自动处理 | [工厂配置](../../../raw/current/scripts/prefabs/balloon.lua#L69-L78) [回调](../../../raw/current/scripts/prefabs/balloons_common.lua#L38-L51) |
| `balloonvest` 持有／穿戴爆破 | 使用 `DoPop`，按最终持有者、装备槽与骑乘分支选择特效／攻击／删除；不等价于浮空回调 | [注册和配置](../../../raw/current/scripts/prefabs/balloonvest.lua#L65-L90) [持有者分支](../../../raw/current/scripts/prefabs/balloons_common.lua#L54-L84) |
| 手持普通气球的显示子实体 | 装备时创建 `balloon_held_child` 并监听持有者 attacked；受击经 `ShouldProcOnAttackedOrBlocked` 门控才掉落气球。卸装移除子实体并解绑持有者监听，删除实体也处理子实体 | [装备与监听](../../../raw/current/scripts/prefabs/balloons_common.lua#L87-L113) [卸装](../../../raw/current/scripts/prefabs/balloons_common.lua#L115-L127) [普通气球挂载](../../../raw/current/scripts/prefabs/balloon.lua#L92-L97) |

持有与装备关系按 [[reference/components/inventoryitem]]、[[reference/components/equippable]] 继续查，不将共享构造的所有辅助方法当作 poppable 接口。

（26148036 核验：手持气球受击掉落与气球背心受击弹出改走 componentutil 的 `ShouldProcOnAttackedOrBlocked` 门控，"骑乘中不掉落"豁免已删除——骑乘重定向、装备自身来源等受击不再掉落／弹出，复刻掉落判定需改走同一门控；防溺水、fueled 耗尽与死亡转发入口不变。）

## 常见陷阱与限制

- **先 Pop 再安装回调：** 首次调用即写 popped；后续补回调也不会执行。先完成服务端配置，再开放触发入口。[顺序](../../../raw/current/scripts/components/poppable.lua#L9-L17)
- **替换回调只做新效果而忘原处理：** 单纯覆盖会丢掉动画、禁交互、范围攻击和删除；增量修改保留原函数。不要以为 Pop 自带 Remove。[组件](../../../raw/current/scripts/components/poppable.lua#L9-L17) [浮空原处理](../../../raw/current/scripts/prefabs/balloons_common.lua#L27-L51)
- **无条件重置 popped 反复爆破：** 组件没有 reset 接口；原气球已进入禁交互／任务清理态，重置字段不能撤销这些动作。可重复效果应自定完整状态契约。[组件](../../../raw/current/scripts/components/poppable.lua#L2-L19) [原清理](../../../raw/current/scripts/prefabs/balloons_common.lua#L27-L51)
- **将 true 当永久存档状态：** 组件没有 OnSave／OnLoad；普通气球宿主仅保存形状／颜色，爆破时宿主设 `persists = false`。自定义仍存活的可爆物需要自己设计持久化，不能只依赖 popped。[宿主存读](../../../raw/current/scripts/prefabs/balloon.lua#L30-L44) [取消持久化](../../../raw/current/scripts/prefabs/balloons_common.lua#L27-L35)
- **认为 Pop 成功等于实体已删除：** 它无返回且效果取决于回调；原 `DoPop` 有骑乘分支，不是每条分支立即 Remove。不要用返回真值或同步消失作为统一判断。[组件](../../../raw/current/scripts/components/poppable.lua#L9-L17) [持有者分支](../../../raw/current/scripts/prefabs/balloons_common.lua#L54-L84)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 的全部变更文件对照 build-26148036 更新报告逐条核验，锚点重定位 7 处、结论修订 1 处（气球门控，见正文"26148036 核验"标注）；未覆盖：运行验证。

- 源码审阅：2026-10-23，`steam-buildid:25643519`；覆盖完整组件、balloon／balloonvest 的真实工厂与服务端配置、共享死亡／燃料耗尽转发、两种爆破回调、手持显示清理、猫玩动作／SG 执行及示例使用的 combat 伤害入口；不代表全部气球或战斗／装备系统已审校。
- 游戏运行：未运行；需实测致死／猫玩／耗尽等入口、基础伤害与实际受伤、延迟攻击／删除、穿戴骑乘与保存读档。
- 结构检查：页面格式／本地链接、源码引用严格检查及完整 modmain 的 Lua 5.1 语法检查通过。
