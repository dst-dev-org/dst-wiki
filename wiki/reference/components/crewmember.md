---
title: "crewmember：AI 船员归船与划船施力"
created: 2026-10-26
updated: "2026-10-28"
type: component
summary: "为 AI 船员保存所属船引用、判定是否应划船并向船体施加一次划船力，由船队管理器和 Brain／SG 协作驱动。"
aliases: [crewmember, AI船员划船, 海盗猴船员, 龙舟船员, 船员退队]
tags: [component, lifecycle, callback, ai, server-client]
dst_build: "steam-buildid:26230496"
sources: [components/crewmember.lua, components/boatcrew.lua, components/boatracecrew.lua, components/piratespawner.lua, prefabs/powdermonkey.lua, brains/powdermonkeybrain.lua, stategraphs/SGpowdermonkey.lua, prefabs/yotd_boats.lua, prefabs/boatrace_primemate.lua]
reliability: verified
validation: source-reviewed
status: active
---

# crewmember：AI 船员归船与划船施力

## 快速判断

- 职责与适用：挂在 AI 船员上，保存船引用并提供划船条件查询／单次施力；实际成员表、航向、目标与状态由船上的 `boatcrew` 或 `boatracecrew` 提供，与 commander 士兵登记独立。[crewmember.lua:33-68](../../../raw/current/scripts/components/crewmember.lua#L33-L68)、[crewmember.lua:93-164](../../../raw/current/scripts/components/crewmember.lua#L93-L164)
- 优先使用：参考海盗生成器动态加组件并调用船端 AddMember，再由猴子 Brain／SG 控制划船时机；不是给人物装桨或自动持续划船的组件。玩家物品桨走 [[reference/components/oar]]，船体力学见 [[reference/components/boatphysics#关键接口]]。[piratespawner.lua:157-175](../../../raw/current/scripts/components/piratespawner.lua#L157-L175)、[SGpowdermonkey.lua:328-366](../../../raw/current/scripts/stategraphs/SGpowdermonkey.lua#L328-L366)
- 重要副作用：`Row()` 直接调用船的 `boatphysics:ApplyRowForce` 并向船发 `rowed`（数据为船员）；没有经过 oar，也没有向执行者发 `rowing`。`Row` 本身不检查 enabled、当前平台或 `Shouldrow()`。[crewmember.lua:93-165](../../../raw/current/scripts/components/crewmember.lua#L93-L165)
- 修改边界：原版 piratespawner 断言服务端，猴子 Brain／SG 也在服务端安装；组件未定义网络字段、更新任务或自己的存读。移除组件只删 `crewmember` 标签，不能作为完整退船方案。[piratespawner.lua:6-8](../../../raw/current/scripts/components/piratespawner.lua#L6-L8)、[powdermonkey.lua:257-259](../../../raw/current/scripts/prefabs/powdermonkey.lua#L257-L259)、[powdermonkey.lua:327-328](../../../raw/current/scripts/prefabs/powdermonkey.lua#L327-L328)、[crewmember.lua:1-31](../../../raw/current/scripts/components/crewmember.lua#L1-L31)

## 关键接口

| 接口／字段 | 参数、返回／默认值 | 条件、副作用与证据 |
|---|---|---|
| `SetBoat(boat)`／`GetBoat()` | 船实体或 nil／当前引用 | set 撤旧船 onremove 监听、换引用并监听新船；船删除回调只清引用。不增删船端成员表，也不把实体送上船。[crewmember.lua:20-21](../../../raw/current/scripts/components/crewmember.lua#L20-L21)、[crewmember.lua:58-72](../../../raw/current/scripts/components/crewmember.lua#L58-L72) |
| `Shouldrow()` | 应划返回 true，其余无显式返回 | 当前平台必须等于保存的船，船需有管理器及 target 或 heading；assault 状态且靠近有效实体目标时不划。此接口不检查 enabled。[crewmember.lua:33-56](../../../raw/current/scripts/components/crewmember.lua#L33-L56) |
| `Row()` | 无参数／无显式返回 | 需要保存的船、管理器及 boatphysics；只施加一次划船力，不循环调度，不自动调用 Shouldrow。[crewmember.lua:93-165](../../../raw/current/scripts/components/crewmember.lua#L93-L165) |
| `force`／`max_velocity` | 默认1／4 | 传给 ApplyRowForce；delivery 状态将后者乘0.65。`max_velocity` 是该力学接口的阻尼参考，不是整船速度硬上限，见 [[reference/components/oar#关键接口]]。[crewmember.lua:14-16](../../../raw/current/scripts/components/crewmember.lua#L14-L16)、[crewmember.lua:162-164](../../../raw/current/scripts/components/crewmember.lua#L162-L164) |
| `max_target_dsq` | 默认 `TUNING.CREWMEMBER_TARGET_DSQ` | 与距离平方严格比较，用于 assault 停划和 Row 近目标判断；自定义值要使用平方距离口径。[crewmember.lua:15](../../../raw/current/scripts/components/crewmember.lua#L15)、[crewmember.lua:47-54](../../../raw/current/scripts/components/crewmember.lua#L47-L54)、[crewmember.lua:136-140](../../../raw/current/scripts/components/crewmember.lua#L136-L140) |
| `Enable(enabled)`、`enabled` | 初始 true | 属性回调按值增删 crewmember 标签；Enable(false) 在赋值前通过 boatcrew 移出当前成员。直接写字段只触发标签回调，不完整退队。[crewmember.lua:1-26](../../../raw/current/scripts/components/crewmember.lua#L1-L26)、[crewmember.lua:86-91](../../../raw/current/scripts/components/crewmember.lua#L86-L91) |
| `Leave()`／`OnLeftCrew()`、`leavecrewfn` | 回调 `leavecrewfn(inst)` | Leave 仅调用当前船的 boatcrew:RemoveMember；OnLeftCrew 仅调用已配置回调，自己不清引用或船端账目。[crewmember.lua:74-91](../../../raw/current/scripts/components/crewmember.lua#L74-L91) |

## 关键流程

### 单次划船的方向与门禁

`Row` 从保存船上的 boatcrew 优先取管理器，否则取 boatracecrew，再读取船体速度和目标。若目标有 boatphysics，计算目标速度与自身船速度向量之和；其平方长度不大于自身速度平方时才允许后续刹车分支。非 retreat 状态还须靠近有效实体／点目标，才选反自身速度方向施力，并非只按距离就停船。[crewmember.lua:93-146](../../../raw/current/scripts/components/crewmember.lua#L93-L146)

retreat 状态中，只有管理器全部成员当前平台都等于本船，才将方向改为 away；否则仍按 toward。正常方向由管理器 `GetHeadingNormal()` 提供，取不到两轴时退回“船员朝船中心”的向量；away 再反转，最后施力并发 rowed。该过程不替代登船、找站位或 enabled／平台校验，调用者须提供这些约束。[crewmember.lua:122-164](../../../raw/current/scripts/components/crewmember.lua#L122-L164)

### 双向成员关系、退出与存读

`boatcrew:AddMember` 维护船端集合、监听成员死亡／移除／传送，并在已有 crewmember 时调用 `SetBoat`；`RemoveMember` 先调用 OnLeftCrew 和船端回调，再撤监听、`SetBoat(nil)` 及扣成员。因此船引用和成员表应由管理器成对维护，不能只手动 SetBoat 来完成入／退队。[boatcrew.lua:5-15](../../../raw/current/scripts/components/boatcrew.lua#L5-L15)、[boatcrew.lua:208-253](../../../raw/current/scripts/components/boatcrew.lua#L208-L253)

crewmember 没有保存接口或剩余划船计时；boatcrew 保存成员 GUID 并在 LoadPostPass 再 AddMember，而动态添加的组件仍需宿主重建。海盗生成器加载船只及成员引用时，会再次执行 SetPirateBoat、SetCaptain／SetCrewMember，恢复组件与退出回调。[boatcrew.lua:292-315](../../../raw/current/scripts/components/boatcrew.lua#L292-L315)、[piratespawner.lua:822-851](../../../raw/current/scripts/components/piratespawner.lua#L822-L851)

`OnRemoveFromEntity` 只移除标签，没有调用 Leave、SetBoat(nil) 或撤旧船监听；正常撤销应先通过实际管理器 RemoveMember 清双向关系，不能把直接删组件或只改 enabled 字段视为完整清理。[crewmember.lua:29-31](../../../raw/current/scripts/components/crewmember.lua#L29-L31)、[crewmember.lua:58-91](../../../raw/current/scripts/components/crewmember.lua#L58-L91)

## 典型接入

### 海盗猴：动态船员组件由 Brain 和 SG 驱动

`piratespawner` 给船长和船员添加 crewmember 后，在船端 boatcrew:AddMember；普通船员的 leavecrewfn 会清除既有 tinkertarget。`powder_monkey` 的服务端构造安装 Brain／`SGpowdermonkey`；Brain 的 rowboat 先检查 Shouldrow 和 busy、核对平台，再寻找站位并提交 ROW 动作；SG 的 row 状态在第7帧调用 Row，动画结束再次按 Shouldrow 选择继续或退出。组件不决定动作频率与动画。[piratespawner.lua:157-175](../../../raw/current/scripts/components/piratespawner.lua#L157-L175)、[powdermonkey.lua:327-328](../../../raw/current/scripts/prefabs/powdermonkey.lua#L327-L328)、[powdermonkey.lua:356](../../../raw/current/scripts/prefabs/powdermonkey.lua#L356)、[powdermonkeybrain.lua:45-92](../../../raw/current/scripts/brains/powdermonkeybrain.lua#L45-L92)、[SGpowdermonkey.lua:328-366](../../../raw/current/scripts/stategraphs/SGpowdermonkey.lua#L328-L366)

### 龙舟 AI：另一管理器与独立调度

`yotd_boats` 的 spawn_ai_captain 生成 `boatrace_primemate`、添加 crewmember，设置平方距离阈值，再加入 boatracecrew；后者 Add／RemoveMember 同样设置／清空船引用并通知 OnLeftCrew。竞赛猴的 sleep_ai_update 先核对所在平台、优先修漏，存在目标时也可直接 Row；这说明并非所有原版调用都由 Shouldrow 开始，门禁是宿主协议的一部分。[yotd_boats.lua:560-566](../../../raw/current/scripts/prefabs/yotd_boats.lua#L560-L566)、[boatracecrew.lua:65-107](../../../raw/current/scripts/components/boatracecrew.lua#L65-L107)、[boatrace_primemate.lua:121-146](../../../raw/current/scripts/prefabs/boatrace_primemate.lua#L121-L146)

## 常见陷阱与限制

- **两类船队接口不完全对称**：Shouldrow／Row 支持 boatcrew 或 boatracecrew，但 Leave 与 Enable(false) 只取 `boat.components.boatcrew`。竞赛船只有 boatracecrew 时直接调用这两个入口可能 nil 访问；应由实际 boatracecrew:RemoveMember 退出。此为源码控制流推导，未游戏复现。[crewmember.lua:41-42](../../../raw/current/scripts/components/crewmember.lua#L41-L42)、[crewmember.lua:74-100](../../../raw/current/scripts/components/crewmember.lua#L74-L100)、[boatracecrew.lua:85-107](../../../raw/current/scripts/components/boatracecrew.lua#L85-L107)
- **只有 enabled 标签并不拦住划船**：Row、Shouldrow 均不读 enabled；直接禁用字段而保留船引用，再被宿主调用 Row，仍可能推进原船。Enable 方法的退队副作用与字段赋值需要区分。[crewmember.lua:1-56](../../../raw/current/scripts/components/crewmember.lua#L1-L56)、[crewmember.lua:86-165](../../../raw/current/scripts/components/crewmember.lua#L86-L165)
- **target 形状要符合状态**：Row 的近目标分支兼容实体与点，但 Shouldrow 在 assault 分支直接调用 target:IsValid()；不能把任意点目标套入该分支并承诺两接口都安全。[crewmember.lua:47-54](../../../raw/current/scripts/components/crewmember.lua#L47-L54)、[crewmember.lua:136-140](../../../raw/current/scripts/components/crewmember.lua#L136-L140)

## 验证记录

- 源码审阅：2026-10-26，Steam buildid 26148036；覆盖组件全文，boatcrew 成员监听／增删／GUID 保存恢复，海盗动态接线与加载、powder_monkey 服务端 Brain／SG 划船消费者，boatracecrew 成员增删及龙舟 AI 创建／休眠更新参照。ApplyRowForce 的阻尼含义复用同版本 oar／boatphysics 页。
- 游戏运行：未运行；未实测加减速、退队清理、加载顺序、竞赛船错误路径或动画时序；未审阅整个船队调度、龙舟网络行为或全部海盗 AI。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 通过；无新增 Lua 示例，未运行语法探针。

相关：[[reference/components/boatphysics]]、[[reference/components/oar]]。
