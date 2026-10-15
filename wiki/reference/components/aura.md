---
title: "aura"
created: "2026-10-23"
updated: "2026-10-28"
type: component
summary: "让实体按周期使用 combat 进行周围伤害，配置筛选并区分开启状态与实际命中表现。"
aliases: ["周期范围伤害", "阿比盖尔攻击光环", "幽灵范围攻击", "孢子云伤害", "aura"]
tags: [component, aura, callback, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/aura.lua", "components/combat.lua", "entityscript.lua", "scheduler.lua", "prefabs/ghost.lua", "prefabs/abigail.lua", "prefabs/sporecloud.lua", "stategraphs/SGghost.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# aura

## 快速判断

- 职责与适用：围绕宿主周期执行 `combat:DoAreaAttack`；需要宿主 combat 提供伤害。周围精神变化用 [[reference/components/sanityaura]]，两者在原版 ghost 中分别安装。[伤害入口](../../../raw/current/scripts/components/aura.lua#L44-L59) [两种光环](../../../raw/current/scripts/prefabs/ghost.lua#L92-L108)
- 优先使用：启用前设置 radius、tickperiod、auratestfn 与排除标签，再 `Enable(true)`；停止用 `Enable(false)`。本组件没有独立的 damage 字段，范围伤害计算复用 [[reference/components/combat#关键接口]]。[配置与开关](../../../raw/current/scripts/components/aura.lua#L1-L10) [伤害计算](../../../raw/current/scripts/components/combat.lua#L1325-L1334)
- 重要副作用：`applying` 从“本轮是否有计数”变化时向宿主推 startaura／stopaura；Enable(true) 本身不推 startaura，也不立即攻击。计数不保证目标实际掉血。[更新](../../../raw/current/scripts/components/aura.lua#L50-L63) [首次调度](../../../raw/current/scripts/scheduler.lua#L295-L299)
- 修改边界：原版 ghost／sporecloud 在服务端分支安装；组件自身无端检查、网络、存档或移除钩子。已有原版开关会被状态机／死亡流程再次调用。[ghost 门禁](../../../raw/current/scripts/prefabs/ghost.lua#L77-L79) [云安装](../../../raw/current/scripts/prefabs/sporecloud.lua#L349-L365) [完整组件](../../../raw/current/scripts/components/aura.lua#L1-L67)

## 关键接口

| 接口／字段 | 参数、时序与证据 |
|---|---|
| `Enable(val)` | 只有 val=false 表示关，nil 等其他值均表示开；仅 active 改变时工作。开时注册一个周期任务；关时取消任务，原 applying=true 才发 stopaura，然后清 applying。无返回值。[实现](../../../raw/current/scripts/components/aura.lua#L25-L39) |
| `radius`／`tickperiod` | 默认 3／1，分别为搜索半径和秒周期。radius 每轮读取；tickperiod 创建任务时使用，运行中只改字段不重排任务，应关后重开。首轮默认等一个周期。[默认](../../../raw/current/scripts/components/aura.lua#L3-L6) [建任务](../../../raw/current/scripts/components/aura.lua#L29-L30) [调度默认](../../../raw/current/scripts/scheduler.lua#L295-L299) |
| `pretickfn(inst)` | 每轮在检查 combat 和搜索之前执行；即使没有 combat 也会先调用。不是目标逐个回调，也没有返回门禁。[调用](../../../raw/current/scripts/components/aura.lua#L44-L59) |
| `auratestfn(inst, target)` | 可选额外目标筛选；经包装交给 DoAreaAttack，在 combat:IsValidTarget 通过后调用。组件不会把 combat 回调的第二参数直接作为此函数的 target。[包装](../../../raw/current/scripts/components/aura.lua#L7-L10) [接线](../../../raw/current/scripts/components/aura.lua#L52-L58) [筛选](../../../raw/current/scripts/components/combat.lua#L1325-L1330) |
| `auraexcludetags` | 默认排除 noauradamage／INLIMBO／notarget／noattack／flight／invisible／playerghost；替换整个表须保留仍需的门禁。底层搜索还要求 `_combat`，排除宿主自身。[默认](../../../raw/current/scripts/components/aura.lua#L9) [底层](../../../raw/current/scripts/components/combat.lua#L1311-L1330) |
| `active`／`applying` | 前者记录任务开关，后者记录最近一轮计数是否大于 0；可 active=true 且 applying=false。DoAreaAttack 每次调用目标 GetAttacked_Internal 就累加，不检查其返回值，不能用 applying 判断实际伤害量。[状态](../../../raw/current/scripts/components/aura.lua#L5-L6) [更新](../../../raw/current/scripts/components/aura.lua#L50-L63) [计数](../../../raw/current/scripts/components/combat.lua#L1331-L1339) |
| `OnTick()` | 可直接调用，但不检查 active；禁用后手动调用仍会筛选／攻击。正常使用让 Enable 管理任务，避免额外调用造成重复攻击。[完整轮次](../../../raw/current/scripts/components/aura.lua#L44-L65) |

每个合格目标先令攻击者发 `onareaattackother`，再 CalcDamage 与目标 `GetAttacked_Internal`；Aura 的 startaura／stopaura 是整轮完成后的状态变化事件，没有目标数据。连续多轮有目标只在首次从 false 转 true 时发 startaura。[战斗顺序](../../../raw/current/scripts/components/combat.lua#L1325-L1339) [状态事件](../../../raw/current/scripts/components/aura.lua#L61-L63)

（26148036 核验：DoAreaAttack 等内部结算路径现直接调 `GetAttacked_Internal(..., from_doattack=true)`，绕过对 `GetAttacked` 的包装／覆盖，目标 `attacked`／`onhitother` 事件新增 `from_doattack` 数据；计数与累加逻辑未变——[combat.lua:1333](../../../raw/current/scripts/components/combat.lua#L1333）

## 典型接入

### 给已有服务端战斗实体接入周期伤害

这是宿主工厂组装指引，不是完整 Prefab demo；模型资源、combat 参数与目标阵营规则沿用目标模组现有实现。

1. 在服务端工厂已有 combat 后配置其伤害，再安装 aura、设半径／周期／筛选。ghost 的 factory 同时配置战斗值与 AuraTest，说明光环没有单独的伤害计算通道。[配置](../../../raw/current/scripts/prefabs/ghost.lua#L100-L108) [目标规则](../../../raw/current/scripts/prefabs/ghost.lua#L18-L21)
2. 决定何时开启：ghost 出场状态退出时 Enable(true)，死亡回调 Enable(false)；不要只装组件就假设已工作，也不要把启用误当成已经命中。[SG 启动](../../../raw/current/scripts/stategraphs/SGghost.lua#L113-L114) [死亡停用](../../../raw/current/scripts/prefabs/ghost.lua#L14-L15)
3. 若要发光／音效，另外监听宿主的 startaura／stopaura；ghost 的 SG 用它们切光色、染色与循环音。新实体没有相同 SG 时，组件不会自动带上这些表现或所需资源。[实际消费者](../../../raw/current/scripts/stategraphs/SGghost.lua#L3-L19)
4. 失效时停用并清宿主附加任务。孢子云另外注册“腐败物品”任务，DisableCloud 分别关闭 aura 和取消 _spoiltask；这不是 aura 自带效果。[两个任务](../../../raw/current/scripts/prefabs/sporecloud.lua#L358-L367) [分别清理](../../../raw/current/scripts/prefabs/sporecloud.lua#L166-L171)

### 防止伤到友军与宿主控制人

阿比盖尔的 auratestfn 明确排除关联玩家、小游戏参与者、禁 PVP 时的玩家、ghost／noauradamage 和 combat:IsAlly；防御范围与目标战斗关系也在该函数中判断。模组应根据自己的队友定义核验筛选，不照搬一条 tag 代替整个规则。[完整规则](../../../raw/current/scripts/prefabs/abigail.lua#L238-L278)

虽然阿比盖尔工厂写了 `aura.ignoreallies = true`，当前 Aura 实现**不读取这个字段**；真正防友伤的是上面的 auratestfn 与战斗门禁。给新实体只赋 ignoreallies 不能实现阵营过滤。[工厂赋值](../../../raw/current/scripts/prefabs/abigail.lua#L864-L873) [组件定义](../../../raw/current/scripts/components/aura.lua#L1-L67)

## 常见陷阱与限制

- **改 tickperiod 以为任务立刻变速**：当前任务保存旧周期，Enable(true) 在已经 active 时也不重建；需要 Enable(false) 后按新值启用。切换可能发 stopaura，下轮再发 startaura，要考虑音效／FX 重启。[开关](../../../raw/current/scripts/components/aura.lua#L25-L39)
- **startaura 等于启用／实际扣血**：它代表上一轮计数从 0 变为非 0，不是开关、逐目标事件或伤害结算确认；按需求分别读取 active、消费战斗事件或核对目标实际行为。[轮次事件](../../../raw/current/scripts/components/aura.lua#L50-L63) [底层计数](../../../raw/current/scripts/components/combat.lua#L1331-L1339)
- **只 RemoveComponent 就停止伤害**：Aura 没有 OnRemoveFromEntity，任务闭包仍持有 self；RemoveComponent 仅自动停止组件更新，不替它取消自行注册的周期任务。动态卸载前 Enable(false)；实体整体 Remove 则会取消其 pendingtasks。[任务闭包](../../../raw/current/scripts/components/aura.lua#L21-L30) [移组件](../../../raw/current/scripts/entityscript.lua#L670-L683) [移实体](../../../raw/current/scripts/entityscript.lua#L1739-L1746)
- **依赖组件自动存读／停尸体伤害**：没有 OnSave／OnLoad／死亡监听；是否继续伤害、恢复开关由宿主工厂、SG 和事件回调控制。ghost 明确在 death 停用。[组件](../../../raw/current/scripts/components/aura.lua#L1-L67) [宿主停用](../../../raw/current/scripts/prefabs/ghost.lua#L14-L15)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 的全部变更文件对照 build-26148036 更新报告逐条核验，锚点重定位 7 处、结论修订 2 处（结算顺序与 from_doattack，见正文"26148036 核验"标注）；未覆盖：运行验证。

- 源码审阅：2026-10-23，Steam buildid 25643519；覆盖完整 Aura、Combat:DoAreaAttack、周期初次调度／移除协作、ghost 安装与 SG 事件／启动、阿比盖尔目标门禁、孢子云双任务与停用。未全面审校 combat 伤害修正、阿比盖尔全部形态／技能或所有 aura 宿主。
- 结构检查：2026-10-24，本页格式、来源、本地链接及源码锚点严格检查通过，本批导航与全库登记检查通过；没有新增完整 Lua 示例。
- 游戏运行：未运行；友伤、状态重入、伤害免疫时的 applying 表现、动态卸载与周期切换待实测。
