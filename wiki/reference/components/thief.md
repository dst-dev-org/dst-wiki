---
title: thief：从背包或容器偷落物品
created: 2026-10-22
updated: "2026-10-28"
type: component
summary: 调用偷窃时按防盗与物品筛选条件让目标掉落物品，并区分回调和返回值的真实含义
aliases: [thief, 偷窃, 青蛙打落物品, onitemstolen, nosteal, 防盗]
tags: [component, callback, item, server-client]
dst_build: "steam-buildid:26230496"
sources: [components/thief.lua, components/inventory.lua, components/container.lua, prefabs/frog.lua]
reliability: verified
validation: source-reviewed
status: active
---

# thief：从背包或容器偷落物品

## 快速判断

- 职责与适用：挂在偷窃者上，`StealItem` 从受害者的 `inventory` 或 `container` 选择物品并调用掉落；**不自动交给偷窃者**，也不自行寻找目标、检测距离或发起周期行为。[实现](../../../raw/current/scripts/components/thief.lua#L15-L58)
- 优先使用：已有攻击／AI 触发点只需让目标掉落物品时调用 `StealItem`；偷窃后的附加逻辑用 `SetOnStolenFn`，但必须处理 `item=nil` 与容器拆堆差异。[回调配置](../../../raw/current/scripts/components/thief.lua#L7-L9)、[分支行为](../../../raw/current/scripts/components/thief.lua#L20-L54)
- 重要副作用：可选 `attack` 会调用战斗攻击；掉落之后先执行 `onstolen(thief, victim, item)`，再向**受害者**发 `onitemstolen`，数据为 `{item=item, thief=self.inst}`。回调发生不等于实际掉落成功。[顺序](../../../raw/current/scripts/components/thief.lua#L20-L54)
- 修改边界：组件不自检服务器、`victim=nil` 或偷窃者是否有 `combat`；原版青蛙在主机分支挂载，并在攻击命中回调中调用。新增接入应保留同样的端和对象约束。[主机宿主](../../../raw/current/scripts/prefabs/frog.lua#L150-L175)、[命中调用](../../../raw/current/scripts/prefabs/frog.lua#L97-L119)

## 关键接口

| 接口／字段 | 签名或默认值 | 条件、作用与证据 |
|---|---|---|
| `StealItem(victim, itemtosteal, attack)` | `victim` 为有效实体；后两项可省略；返回布尔 | 无效实体返回 `false`；先尝试非防盗背包，否则尝试非防盗容器；两者都不适用则返回 `false`。不接受 `nil` victim。[实现](../../../raw/current/scripts/components/thief.lua#L15-L58) |
| `itemtosteal` | 可选物品实体 | 省略时用 `FindItem` 选择首个无 `nosteal` 标签的物品；显式传入直接使用，**绕过 `nosteal` 筛选**，也不由组件验证归属。调用方应保证物品属于目标且可掉落。[筛选与背包](../../../raw/current/scripts/components/thief.lua#L11-L29)、[容器](../../../raw/current/scripts/components/thief.lua#L38-L48) |
| `attack` | 可选真值 | 背包分支无论是否找到物品都可攻击 victim；容器分支只在容器自身有 `equippable`、`inventoryitem` 且有 owner 时攻击 owner。调用者需有 `combat`；这不是偷窃的必要开关。[背包攻击](../../../raw/current/scripts/components/thief.lua#L20-L25)、[容器攻击](../../../raw/current/scripts/components/thief.lua#L38-L46) |
| `SetOnStolenFn(fn)` | `fn(thief, victim, item)`；初始未设置 | 直接替换 `onstolen`，不叠加；扩展现有宿主时先保存并调用旧回调。传 `nil` 可清除回调。[设置](../../../raw/current/scripts/components/thief.lua#L1-L9)、[消费](../../../raw/current/scripts/components/thief.lua#L27-L54) |
| `stolenitems` | 初始空表，已废弃 | 当前实现未用它登记物品；不要依赖它追踪掉落物、补存档或清理引用。[构造](../../../raw/current/scripts/components/thief.lua#L1-L5) |

### 默认选物与两种掉落分支

- **选物范围：** `inventory:FindItem` 查普通物品槽、activeitem、overflow 容器，不遍历装备槽；`container:FindItem` 查自身 slots。遍历槽位使用 `pairs`，没有随机选择或按价值排序保证。防盗条件分别读取两组件的 `thiefproof`。[背包查找](../../../raw/current/scripts/components/inventory.lua#L623-L636)、[容器查找](../../../raw/current/scripts/components/container.lua#L763-L769)、[背包防盗](../../../raw/current/scripts/components/inventory.lua#L262-L264)、[容器防盗](../../../raw/current/scripts/components/container.lua#L1647-L1649)
- **背包：** 有候选物品时用 `inventory:DropItem(item, false, direction)`，并将其返回值重新赋给 `item`，再传回调和事件；`direction` 由受害者指向偷窃者。若候选存在，最后无条件返回 `true`，即使 `DropItem` 因物品类型或锁槽检查返回 `nil`。回调须防御空物品。[偷窃分支](../../../raw/current/scripts/components/thief.lua#L20-L37)、[掉落返回与限制](../../../raw/current/scripts/components/inventory.lua#L721-L753)
- **容器：** 调用 `container:DropItem(item)` 后不接收实际掉落对象，回调和事件仍传原候选；最后仅按候选是否非空返回。`DropItem` 本身不返回 `DropItemAt` 的结果；后者可能受锁槽或只读条件影响，也可能拆出新实体。因此原候选非空不证明成功落地。[偷窃分支](../../../raw/current/scripts/components/thief.lua#L38-L54)、[DropItem](../../../raw/current/scripts/components/container.lua#L220-L226)、[DropItemAt](../../../raw/current/scripts/components/container.lua#L247-L267)、[只读与拆堆](../../../raw/current/scripts/components/container.lua#L1019-L1038)
- **拆堆对象差异：** 容器掉落调用 `RemoveItem`，未要求整堆；堆叠物取出一个时返回拆出的实体，原候选可能继续留在容器中。不要在 `onstolen` 中把容器分支传来的 `item` 直接当作地面战利品交给偷窃者。[移除入口](../../../raw/current/scripts/components/container.lua#L1009-L1017)、[拆堆](../../../raw/current/scripts/components/container.lua#L1025-L1038)

## 典型接入

### 沿用普通青蛙的命中偷窃链

这是原版接入参照，不是另造 AI 或战斗工程；适合已有生物攻击链、只需在命中后让对方掉落物品的需求：

1. 注册名 `frog` 使用 `normalfn`，再调用 `commonfn("frog")`；工厂在 `SetPristine` 后的主机分支配置 `combat.onhitotherfn=OnHitOther` 并添加 `thief`。[注册](../../../raw/current/scripts/prefabs/frog.lua#L257-L259)、[普通工厂](../../../raw/current/scripts/prefabs/frog.lua#L193-L210)、[服务端组装](../../../raw/current/scripts/prefabs/frog.lua#L150-L175)
2. 普通青蛙 `OnHitOther` 在没有 `damageredirecttarget` 时调用 `inst.components.thief:StealItem(other)`。攻击已经由原命中链完成，所以这里不再传 `attack=true`，避免额外发起攻击。[命中回调](../../../raw/current/scripts/prefabs/frog.lua#L97-L119)
3. 新宿主把同类调用放进自己已核验的服务端命中回调，保留原 `onhitotherfn` 的参数和原行为；仅加 `thief` 不会自动获得青蛙攻击或偷窃触发。若追加 `onstolen`，必须保留旧回调，并按上文区分背包掉落结果与容器原候选。[组件入口](../../../raw/current/scripts/components/thief.lua#L15-L58)

这条参照还解释了月亮青蛙的差异：同一命中回调在月亮分支、无伤害重定向时，按 TUNING 给定次数重复 `StealItem`；不能把普通青蛙的一次调用推广到所有青蛙。[分支](../../../raw/current/scripts/prefabs/frog.lua#L97-L118)

生效范围由宿主命中回调决定；撤销自定义接入时恢复原回调／移除新增调用，不应撤销整次原版攻击。组件自身没有更新循环、`OnSave`／`OnLoad` 或移除清理钩子；自定义回调需随宿主构造重新注册，追踪实体和持久化数据由接入方自行管理。[组件全文](../../../raw/current/scripts/components/thief.lua#L1-L60)

## 常见陷阱与限制

- **`nosteal` 不是强制权限：** 只有自动查找应用该标签；显式物品会绕过筛选。需要硬性保护时，在调用入口同时验证标签、物品归属和防盗条件，不能只改自动筛选。[实现](../../../raw/current/scripts/components/thief.lua#L11-L39)
- **空容器也会通知：** 容器分支即使没选到物品，仍调用 `DropItem(nil)`、回调及 `onitemstolen`，最后返回 `false`；空背包分支则没有回调／事件，但若启用 `attack` 仍可攻击。[两分支](../../../raw/current/scripts/components/thief.lua#L20-L54)、[空值掉落](../../../raw/current/scripts/components/container.lua#L247-L249)
- **返回 `true` 不代表偷者拿到了物品：** 组件不调用偷者背包接收；背包失败和容器拆堆还会让返回值、回调物品与落地结果不一致。要实现拾取型偷窃，需另行核验并接管实际掉落对象，不能依赖废弃 `stolenitems`。[返回与事件](../../../raw/current/scripts/components/thief.lua#L27-L54)、[废弃字段](../../../raw/current/scripts/components/thief.lua#L1-L5)
- **攻击回调递归风险：** 在宿主 `onhitotherfn` 中再次传 `attack=true` 会要求组件再发起 `DoAttack`；原版青蛙省略该参数。是否再次命中由战斗链决定，接入时保留这种分工。[青蛙调用](../../../raw/current/scripts/prefabs/frog.lua#L97-L119)、[攻击调用点](../../../raw/current/scripts/components/thief.lua#L20-L25)

## 验证记录

- 版本沿用（2026-10-25）：「原始哈希等价批量沿用-24」批次机械沿用——所列来源原始字节均未变，结论与行号直接沿用；页内筛查与枢纽反查无命中。共享依据见 [build-26148036 的 Wiki 沿用记录](../../updates/build-26148036/index.md#wiki-沿用记录)。


- 2026-10-22，`steam-buildid:25643519`：源码审阅覆盖 thief 全部实现、背包与容器防盗／默认选物／掉落返回、容器拆堆和只读条件、普通与月亮青蛙注册／服务端组装／命中触发。
- 本页通过定向页面格式／本地链接与严格源码行号检查；本页不提供独立 Lua 示例，原版接入参照不作为可复制完整工程。
- 未运行游戏；未验证其他偷窃者 AI、背包 overflow 的全部特殊容器、联机表现或拾取型偷窃实现。

- 版本沿用：2026-10-28 原始哈希等价批量沿用-70（26148036→26230496，本页 sources 所列文件原始字节全部未变），页内未决项零标记、符号反查无命中、严格链接通过；批次依据见 [[updates/build-26230496/index#wiki-沿用记录]]。