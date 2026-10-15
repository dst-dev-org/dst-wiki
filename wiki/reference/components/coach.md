---
title: "coach：沃尔夫冈定时鼓舞与随从 buff"
created: 2026-10-26
updated: "2026-10-28"
type: component
summary: "沿沃尔夫冈技能、哨子和力量形态接线，定时恢复附近其他玩家理智并给随从施加限时伤害来源。"
aliases: [coach, 沃尔夫冈鼓舞, 教练哨子, wolfgang_coach_buff, 随从鼓舞]
tags: [component, lifecycle, callback, event, server-client]
dst_build: "steam-buildid:26230496"
sources: [components/coach.lua, prefabs/wolfgang.lua, prefabs/wolfgang_whistle.lua, prefabs/skilltree_wolfgang.lua, components/mightiness.lua, tuning.lua, entityscript.lua]
reliability: verified
validation: source-reviewed
status: active
---

# coach：沃尔夫冈定时鼓舞与随从 buff

## 快速判断

- 职责与适用：挂在沃尔夫冈上的定时鼓舞任务，扫描自己及附近玩家的普通随从，另给附近其他玩家恢复理智；不是基于进出范围自动施加／移除效果的通用光环。[coach.lua:20-80](../../../raw/current/scripts/components/coach.lua#L20-L80)
- 优先使用：理解原版“技能标签与哨子开关 → 定时鼓舞 → 随从 debuff”的完整接线；自定义限时效果的工厂与成对撤销参见 [[reference/components/debuff]]，不必把 coach 复制为通用 buff 管理器。[wolfgang_whistle.lua:12-38](../../../raw/current/scripts/prefabs/wolfgang_whistle.lua#L12-L38)、[wolfgang.lua:698-761](../../../raw/current/scripts/prefabs/wolfgang.lua#L698-L761)
- 重要副作用：Enable 添加 `coaching` 标签并排任务；Disable 只去标签、取消当前主任务，既不取消已排在随从上的延迟施加，也不驱散既有 buff。[coach.lua:8-18](../../../raw/current/scripts/components/coach.lua#L8-L18)、[coach.lua:60-65](../../../raw/current/scripts/components/coach.lua#L60-L65)、[coach.lua:89-94](../../../raw/current/scripts/components/coach.lua#L89-L94)
- 修改边界：原版非 lavaarena 的沃尔夫冈 master_postinit 添加组件；组件自身不检查技能、力量形态或 enabled 才执行，且没有存读／移除清理钩子。资格和正常退出依赖哨子与 mightiness 接线。[wolfgang.lua:643-671](../../../raw/current/scripts/prefabs/wolfgang.lua#L643-L671)、[coach.lua:1-96](../../../raw/current/scripts/components/coach.lua#L1-L96)

## 关键接口

| 接口／字段 | 默认值、参数 | 条件、副作用与证据 |
|---|---|---|
| `Enable()`／`Disable()` | enabled 初始 false | 分别设标记、增删 coaching 标签及 Start／Stop；没有 normal 形态或技能校验，不主动撤现存目标效果。[coach.lua:1-18](../../../raw/current/scripts/components/coach.lua#L1-L18) |
| `StartInspiring()`／`StopInspiring()` | 初次延迟 `TUNING.COACH_TIME_TO_INSPIRE`（7秒） | start 仅在无 inspiretask 时排主任务；stop 取消并清该字段，不修改 enabled／标签或 lastcoachtime。[coach.lua:83-94](../../../raw/current/scripts/components/coach.lua#L83-L94)、[tuning.lua:7882-7884](../../../raw/current/scripts/tuning.lua#L7882-L7884) |
| `settime`／`randtime` | 10／30秒 | 鼓舞须无上次时间，或距上次严格大于 settime；每次回调结束再排 `settime + random * randtime` 的下一次尝试，默认约10–40秒，不是每10秒稳定施加。[coach.lua:4-5](../../../raw/current/scripts/components/coach.lua#L4-L5)、[coach.lua:25-26](../../../raw/current/scripts/components/coach.lua#L25-L26)、[coach.lua:80](../../../raw/current/scripts/components/coach.lua#L80 |
| `lastcoachtime`／`noteamlasttime` | 使用中才赋值 | 前者在进入鼓舞分支时更新，即便最终无人受益；后者让无受益对象的提示隔次出现。Disable 不重置这两个字段。[coach.lua:25-28](../../../raw/current/scripts/components/coach.lua#L25-L28)、[coach.lua:68-80](../../../raw/current/scripts/components/coach.lua#L68-L80) |

## 关键流程

### 一次鼓舞的对象、延迟与事件

1. 收集教练自身 leader.followers 中距教练严格小于25的实体；再找25内的其他 AllPlayers，把这些玩家加入理智候选，同时把他们普通随从中也距教练小于25者加入战斗 buff 候选。其他玩家在范围内不代表其所有随从都入选，教练自己不加入理智候选；这里没有额外阵营、PvP、幽灵、战斗中或生物类别过滤。[coach.lua:20-51](../../../raw/current/scripts/components/coach.lua#L20-L51)
2. 对有 sanity 且当前百分比低于0.75的队友调用 `DoDelta(5)`；0.75是是否调用的阈值，不是本段设置的恢复上限。[coach.lua:53-58](../../../raw/current/scripts/components/coach.lua#L53-L58)
3. 给每个随从在其自身上排 `0.2 + 0.1 * i` 秒延迟任务，届时发 `cheer` 并调用 `AddDebuff("wolfgang_coach_buff", "wolfgang_coach_buff")`。延迟执行不重查教练是否启用或双方距离，也不检查 AddDebuff 返回值。通用 AddDebuff 仍会检查目标是否允许 debuff、是否死亡／幽灵，故排任务不等于 buff 已成功附着。[coach.lua:60-65](../../../raw/current/scripts/components/coach.lua#L60-L65)、[entityscript.lua:2140-2154](../../../raw/current/scripts/entityscript.lua#L2140-L2154)
4. 有理智恢复或排出随从任务时，立即向教练发 `coach`，不等待延迟施加成功；否则通过 talker 交替提示无队友。回调最后再排下一次主任务，不根据 enabled 决定续排。[coach.lua:53-80](../../../raw/current/scripts/components/coach.lua#L53-L80)

### 伤害 buff 的消费与撤销

`wolfgang.lua` 注册同名效果 prefab。Attached 把效果挂到目标实体，监听目标死亡以 Stop，安排10秒后 Stop；目标有效且有 combat 时，以效果实体为来源向 `combat.externaldamagemultipliers` 添加倍率2，并创建附着 FX。这是该来源的伤害乘数，不代表所有最终伤害无条件翻倍。[wolfgang.lua:698-718](../../../raw/current/scripts/prefabs/wolfgang.lua#L698-L718)、[wolfgang.lua:788-790](../../../raw/current/scripts/prefabs/wolfgang.lua#L788-L790)、[tuning.lua:7882-7884](../../../raw/current/scripts/tuning.lua#L7882-L7884)

同键再次施加按 [[reference/components/debuff#关键接口]] 进入 Extended，原版回调取消旧 bufftask 再排完整10秒；不是新增另一个伤害来源。Detached 在目标有效且仍有 combat 时移除该来源，删 FX 后删除效果实体。buff 工厂只在服务端添加 debuff，设置 Attached／Detached／Extended 回调及 `persists=false`；这些独立生命周期不会由 coach:Disable 代替执行。[wolfgang.lua:720-761](../../../raw/current/scripts/prefabs/wolfgang.lua#L720-L761)

## 典型接入

### 原版沃尔夫冈：技能、哨子和力量变化分工

沃尔夫冈 master_postinit 的非 lavaarena 分支添加 mightiness 和 coach；技能 `wolfgang_normal_coach` 的激活／撤销回调增删 `wolfgang_coach` 标签。组件默认不启动，技能标签本身也不会调用 Enable。[wolfgang.lua:643-671](../../../raw/current/scripts/prefabs/wolfgang.lua#L643-L671)、[skilltree_wolfgang.lua:77-87](../../../raw/current/scripts/prefabs/skilltree_wolfgang.lua#L77-L87)

`wolfgang_whistle` 服务端通过 instrument:SetOnPlayedFn 接 OnPlayed。使用者须同时有 coach、mightiness 和技能标签；仅 normal 形态按 coaching 标签切换启停，wimpy／mighty 只选择对应提示。通过 CanTransform 的 Mightiness:BecomeState 在变换过程中调用 coach:Disable，因此力量形态变化是正常停止入口之一，不是 coach 自行持续检查形态。[wolfgang_whistle.lua:12-38](../../../raw/current/scripts/prefabs/wolfgang_whistle.lua#L12-L38)、[wolfgang_whistle.lua:66-87](../../../raw/current/scripts/prefabs/wolfgang_whistle.lua#L66-L87)、[mightiness.lua:262-267](../../../raw/current/scripts/components/mightiness.lua#L262-L267)、[mightiness.lua:288-315](../../../raw/current/scripts/components/mightiness.lua#L288-L315)

## 常见陷阱与限制

- **停鼓舞立刻撤销所有效果**：Stop 只撤主任务；随从任务已挂在目标上，已有 buff 还有自己的10秒计时和刷新逻辑，离开范围也不由 coach 自动驱散。需立即撤销效果时应沿目标的 debuff 协议另行处理。[coach.lua:60-65](../../../raw/current/scripts/components/coach.lua#L60-L65)、[coach.lua:89-94](../../../raw/current/scripts/components/coach.lua#L89-L94)、[wolfgang.lua:698-735](../../../raw/current/scripts/prefabs/wolfgang.lua#L698-L735)
- **只改 enabled／技能标签就是完整开关**：inspire 不读 enabled；Start／Stop 又不改标签，而技能 ondeactivate 自身只删资格标签。正常开关用 Enable／Disable，不能靠裸字段或标签保证停止任务。Disable 不清 lastcoachtime，短时间重启后7秒的第一次尝试也可能仍被严格冷却条件跳过。[coach.lua:8-26](../../../raw/current/scripts/components/coach.lua#L8-L26)、[coach.lua:80-94](../../../raw/current/scripts/components/coach.lua#L80-L94)、[skilltree_wolfgang.lua:85-86](../../../raw/current/scripts/prefabs/skilltree_wolfgang.lua#L85-L86)
- **删除组件自动收尾**：本组件没有 OnRemoveFromEntity、存读或睡眠钩子；主任务回调还直接读取 inst.components.coach。不要把 RemoveComponent 当已核验撤销方案，应先 Disable 并分别考虑目标上已排任务与效果。读档重建默认 enabled=false，不恢复上次鼓舞时间／任务。[coach.lua:1-96](../../../raw/current/scripts/components/coach.lua#L1-L96)

## 验证记录

- 源码审阅：2026-10-26，Steam buildid 26148036；覆盖组件全文、沃尔夫冈安装／buff 注册与三类回调、技能标签、哨子消费、mightiness 变换禁用、三个 tuning 值和通用 AddDebuff 门禁；同键扩展协议复用同版本 debuff 页。
- 游戏运行：未运行；未实测关停与延迟任务竞态、理智边界、技能撤销、形态切换、buff 刷新／死亡撤销或移除组件；未审阅 coach／cheer 的所有 SG 表现和特殊模式。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 通过；无新增 Lua 示例，未运行语法探针。

相关：[[reference/components/debuff]]、[[reference/components/debuffable]]、[[reference/components/leader]]、[[reference/components/combat]]。
