---
title: "waxable"
created: "2026-10-23"
updated: "2026-10-28"
type: component
summary: "给目标接入打蜡回调，区分材料匹配、成功消耗、物品替换与持久化的责任。"
aliases: ["物品打蜡", "巨型作物防腐", "雪人喷蜡", "打蜡成功才消耗", "waxable"]
tags: [component, item, callback, actions, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/waxable.lua", "components/wax.lua", "componentactions.lua", "actions.lua", "prefabs/veggies.lua", "prefabs/snowman.lua", "prefabs/hats.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# waxable

## 快速判断

- 职责与适用：挂在被打蜡的目标上，通过 `waxfn(target, doer, waxitem)` 实现防腐、转换或停止融化；仅挂组件不自带这些效果。材料侧用 [[reference/components/wax]]。[回调与消耗](../../../raw/current/scripts/components/waxable.lua#L38-L62)
- 优先使用：`SetWaxfn(fn)` 设置效果，返回 true 才消费材料；需要喷雾时 `SetNeedsSpray(true)`，普通蜡用 false。普通动作还要求材料和目标喷雾标签严格相等。[setter](../../../raw/current/scripts/components/waxable.lua#L26-L36) [发现条件](../../../raw/current/scripts/componentactions.lua#L1931-L1935)
- 重要副作用：回调先执行，成功后按 finiteuses→stackable→整件删除的优先级消费；目标可在回调内替换自己或移除 waxable，组件不会额外保护／回滚。[消费顺序](../../../raw/current/scripts/components/waxable.lua#L43-L58) [替换目标](../../../raw/current/scripts/prefabs/veggies.lua#L290-L309)
- 修改边界：服务端配置组件；客户端动作依赖标签。组件仅管理标签和转发，不自带存档、同步完成状态或失败原因；这些由宿主承担。[标签与构造](../../../raw/current/scripts/components/waxable.lua#L1-L24) [移除](../../../raw/current/scripts/components/waxable.lua#L64-L67) [宿主存读](../../../raw/current/scripts/prefabs/hats.lua#L6007-L6023)

## 关键接口

| 接口／字段 | 参数或返回 | 条件、作用与证据 |
|---|---|---|
| `SetWaxfn(fn)`／`waxfn` | 回调 `(target, doer, waxitem) → result, reason` | 有回调时加 `waxable`，设 nil 删除该标签。result 真值才消费；结果／原因原样返回。[属性回调](../../../raw/current/scripts/components/waxable.lua#L1-L7) [设置](../../../raw/current/scripts/components/waxable.lua#L26-L28) [调用](../../../raw/current/scripts/components/waxable.lua#L43-L58) |
| `SetNeedsSpray(val)` | 仅 false 关闭，其余值开启 | 无参数／nil 也设 true，联动 `needswaxspray` 标签。[标签](../../../raw/current/scripts/components/waxable.lua#L9-L15) [setter](../../../raw/current/scripts/components/waxable.lua#L30-L32) |
| `NeedsSpray()` | 默认 false | 返回字段；不是完整材料适配判断。[默认](../../../raw/current/scripts/components/waxable.lua#L17-L24) [getter](../../../raw/current/scripts/components/waxable.lua#L34-L36) |
| `Wax(doer, waxitem)` | `result, reason`；无回调／喷雾门禁失败返回 false | 先查“要求喷雾＋材料有 wax＋不是喷雾”；通过后调用回调。参数必须有效，没有通用实体／持有者校验。[完整执行](../../../raw/current/scripts/components/waxable.lua#L38-L62) |
| 成功消费 | `finiteuses:Use()`／`stackable:Get():Remove()`／`Remove()` | 有耐久时优先扣默认一次；有堆叠时只删拆出的一件；两者都无则删除整个材料。不得在回调重复消费。[分支](../../../raw/current/scripts/components/waxable.lua#L46-L55) |
| `OnRemoveFromEntity()` | 无返回 | 清掉 `waxable`、`needswaxspray`，不撤销已产生的目标效果。[清理](../../../raw/current/scripts/components/waxable.lua#L64-L67) |

## 典型接入

### 只允许无持有者的南瓜帽接受普通蜂蜡

将以下完整代码放入现有模组 `modmain.lua`，无需新资源。只修改服务端 `pumpkinhat` 的资格检查，保留原回调、防腐效果、标签清理及读档恢复；`IsHeld()` 表示直接 owner 非 nil，因此人物、容器等持有状态均拒绝，详见 [[reference/components/inventoryitem#关键接口]]。

```lua
AddPrefabPostInit("pumpkinhat", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    local waxable = inst.components.waxable
    if waxable == nil or waxable.waxfn == nil then
        return
    end
    local oldwaxfn = waxable.waxfn
    waxable:SetWaxfn(function(target, doer, waxitem)
        if target.components.inventoryitem:IsHeld() then
            return false
        end
        return oldwaxfn(target, doer, waxitem)
    end)
end)
```

原版 `MakeHat("pumpkin")` 通过 `name.."hat"` 注册为 `pumpkinhat`，服务端挂 waxable。原回调移除 perishable／waxable、设置网络蜡状态，并在 OnSave／OnLoad 保存恢复 `data.waxed`。[注册名](../../../raw/current/scripts/prefabs/hats.lua#L16-L20) [工厂选择](../../../raw/current/scripts/prefabs/hats.lua#L6958-L6963) [导出](../../../raw/current/scripts/prefabs/hats.lua#L7944) [挂载](../../../raw/current/scripts/prefabs/hats.lua#L6164-L6173) [原回调](../../../raw/current/scripts/prefabs/hats.lua#L6055-L6082) [持久化](../../../raw/current/scripts/prefabs/hats.lua#L6007-L6023)

失败只返回 false，不消费蜂蜡；将帽子放到地面再执行时才沿用原行为。代码不隐藏原交互菜单，也不添加专用失败台词。已打蜡帽读档仍走原恢复逻辑；本包装不加任务、监听或额外存档状态，无需新增清理。替换目标 Prefab 前，重新核验目标组件、回调及持有语义。[消费门槛](../../../raw/current/scripts/components/waxable.lua#L43-L58) [动作转发](../../../raw/current/scripts/actions.lua#L5547-L5554)

### 按目标效果选择原版模式

| 使用模式 | 组装与效果 | 证据 |
|---|---|---|
| 巨型作物：生成替代 Prefab | 共同构造预加 waxable 标签，服务端设回调；生成 `<原名>_waxed`，搬运中替换身体装备，否则放在原位置；转移雕刻组件后删除原实体并返回 true | [标签／端边界](../../../raw/current/scripts/prefabs/veggies.lua#L835-L867) [回调](../../../raw/current/scripts/prefabs/veggies.lua#L290-L309) [挂载](../../../raw/current/scripts/prefabs/veggies.lua#L897-L898) [两种工厂注册](../../../raw/current/scripts/prefabs/veggies.lua#L1084-L1090) |
| 南瓜帽：保留实体并保存状态 | 原实体移除腐烂和 waxable，通过网络值改变表现；载入 waxed 状态重做配置 | [清理／网络值](../../../raw/current/scripts/prefabs/hats.lua#L6055-L6082) [存读](../../../raw/current/scripts/prefabs/hats.lua#L6007-L6023) |
| 雪人：动态允许喷雾 | 堆叠或不是小型、且未打蜡才挂组件；要求喷雾。回调拒绝推行、被持有及单个小雪球，成功停止融化并移除 waxable；OnLoad 恢复蜡状态 | [动态资格](../../../raw/current/scripts/prefabs/snowman.lua#L366-L383) [回调](../../../raw/current/scripts/prefabs/snowman.lua#L342-L364) [配置](../../../raw/current/scripts/prefabs/snowman.lua#L269-L273) [保存／恢复](../../../raw/current/scripts/prefabs/snowman.lua#L570-L596) [工厂注册](../../../raw/current/scripts/prefabs/snowman.lua#L849) |

新目标需要完整 Prefab 的资源／网络／注册上下文；以上是模式入口，不是三套可直接复制的独立工程。普通与喷雾材料及玩家双端动作路径统一见 [[reference/components/wax#典型接入]]。

## 常见陷阱与限制

- **要求喷雾不等于严格校验所有材料：** 裸 `Wax` 对“没有 wax 的非 nil 物品”不会因喷雾需求自动拒绝；普通目标也不拒绝喷雾。玩家菜单的严格匹配不能替代自定义调用者的检查。[执行门禁](../../../raw/current/scripts/components/waxable.lua#L38-L41) [菜单门禁](../../../raw/current/scripts/componentactions.lua#L2563-L2567)
- **回调做完效果却没 return true：** nil／false 不消费，效果也不会回滚；返回真值前完成必需处理，失败前避免不可逆副作用。[顺序](../../../raw/current/scripts/components/waxable.lua#L43-L58)
- **先在回调删除材料：** 组件成功后仍访问 waxitem 的组件并消费；让组件负责消费，回调只实现目标效果。[消费](../../../raw/current/scripts/components/waxable.lua#L44-L55)
- **只添加组件没有菜单：** waxable 标签来自 waxfn 非空；巨型作物、南瓜帽在共同构造预加标签，服务端再配置组件。对新网络目标保留客户端标签与服务端实际资格的一致性。[标签回调](../../../raw/current/scripts/components/waxable.lua#L1-L7) [作物](../../../raw/current/scripts/prefabs/veggies.lua#L847-L849) [帽子](../../../raw/current/scripts/prefabs/hats.lua#L6106-L6107)
- **以为 waxable 自动持久化效果：** 组件无存读方法；替换 Prefab、宿主保存字段／网络值是不同实现。按上表选择目标模式，不能只加标签宣称永久防腐。[组件实现](../../../raw/current/scripts/components/waxable.lua#L17-L69) [帽子存读](../../../raw/current/scripts/prefabs/hats.lua#L6007-L6023)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 的变更文件（componentactions.lua、actions.lua、prefabs/hats.lua）对照 build-26148036 更新报告条目 R-01／R-04 逐条核验；hats.lua 的新增 hunk 均为新帽类功能，不触及南瓜帽／蜡状态代码，引用区域内容未变化；锚点重定位 12 处，无结论修订；未覆盖：运行验证。

- 源码审阅：2026-10-23，`steam-buildid:25643519`；覆盖完整 waxable／wax、动作匹配与执行、三类目标的注册／端边界、回调／消费和南瓜帽／雪人蜡状态存读；完整 modmain 保留南瓜帽原处理。持有语义复用同快照 inventoryitem 页；不代表全部帽子、作物或雪人机制已审校。
- 游戏运行：未运行；持有时失败、落地成功、材料消费及打蜡后读档需实测。
- 结构检查：页面格式／链接、源码锚点严格检查及完整 modmain 的 Lua 5.1 语法检查通过。
