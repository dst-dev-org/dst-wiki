---
title: "teamattacker"
created: "2026-10-23"
updated: "2026-10-28"
type: component
summary: "让生物加入临时围攻队伍，消费列阵与攻击命令；区分退队、暂离阵形与实际战斗行为。"
aliases: ["teamattacker", "围攻队员", "蝙蝠轮流攻击", "团队阵形", "退出围攻队伍"]
tags: [component, creature, ai, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/teamattacker.lua", "components/teamleader.lua", "prefabs/teamleader.lua", "prefabs/bat.lua", "brains/batbrain.lua", "stategraphs/SGbat.lua", "prefabs/penguin.lua", "brains/penguinbrain.lua", "constants.lua", "mainfunctions.lua", "entityscript.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# teamattacker

## 快速判断

- 职责与适用：生物侧保存临时队伍、列阵位置和命令，由 [[reference/components/teamleader]] 招募并分批派遣；[[reference/components/leader]] 的主人随从关系、[[reference/components/herd]] 的群体中心另有职责。[初始化](../../../raw/current/scripts/components/teamattacker.lua#L13-L29) [登记](../../../raw/current/scripts/components/teamleader.lua#L139-L160)
- 优先使用：已有围攻生物用 `SearchForTeam` 寻找同类型队伍、`LeaveTeam` 解除登记；只临时让 Brain 接管移动用 `LeaveFormation`，恢复阵形用 `JoinFormation`。[找队](../../../raw/current/scripts/components/teamattacker.lua#L37-L45) [退出与阵形](../../../raw/current/scripts/components/teamattacker.lua#L68-L83)
- 重要副作用：队伍达到人数门槛时，HOLD／WARN／无命令会 DropTarget 并走向阵位，ATTACK 才向 combat 建议队伍威胁。组件不会直接播放攻击或结算伤害。[更新](../../../raw/current/scripts/components/teamattacker.lua#L90-L113) [实际攻击帧](../../../raw/current/scripts/stategraphs/SGbat.lua#L918-L922)
- 修改边界：原版在主模拟生物工厂挂载，并同时配 combat／locomotor／health／knownlocations、Brain、SG 与目标回调；仅 AddComponent 不会自动寻找队伍或组成可用战斗 AI。[服务端入口](../../../raw/current/scripts/prefabs/bat.lua#L247-L264) [配套](../../../raw/current/scripts/prefabs/bat.lua#L275-L312)

## 关键接口

| 接口／字段 | 参数或默认值 | 条件、作用与证据 |
|---|---|---|
| `team_type` | 默认 `"monster"` | 属性赋值会撤旧 `team_<类型>`、加新标签并更新 `teamleader_<类型>` 搜索条件；不会将现有成员迁移到另一队。蝙蝠配置 `"bat"`。[属性](../../../raw/current/scripts/components/teamattacker.lua#L1-L24) [宿主](../../../raw/current/scripts/prefabs/bat.lua#L310-L312) |
| `SearchForTeam()` | 返回 true 或无显式返回 | 半径 `searchradius` 默认 50；遇首个未满的同类队伍便调用 NewTeammate 并返回 true，不核对招募结果，也不按目标、最近距离或成员资格排序。[实现](../../../raw/current/scripts/components/teamattacker.lua#L37-L45) [资格](../../../raw/current/scripts/components/teamleader.lua#L108-L115) |
| `SetValidMemberFn(fn)` | `fn(成员inst, 队长inst)` | 由队长 ValidMember 调用，不在 SearchForTeam 返回前复核。蝙蝠排除恐慌、电围栏回避及燃烧；回调拒绝后仍可能得到 SearchForTeam=true。[设置](../../../raw/current/scripts/components/teamattacker.lua#L86-L87) [消费](../../../raw/current/scripts/components/teamleader.lua#L108-L115) [蝙蝠条件](../../../raw/current/scripts/prefabs/bat.lua#L39-L44) |
| `teamleader`／`inteam` | 队长**组件**引用／默认 false | 引用不是队长实体；由 NewTeammate 与 OnLostTeammate 联动，不能只写 inteam 来加入队伍。[登记](../../../raw/current/scripts/components/teamleader.lua#L152-L159) [退出](../../../raw/current/scripts/components/teamleader.lua#L177-L190) |
| `GetOrders()`／`orders` | nil 或 `ORDERS` 数字 | 当前快照 NONE=0、HOLD=1、WARN=2、ATTACK=3；NONE 是真值且不等 nil，不会进入“无命令”分支。只读命令，不自行改变战斗授权。[取值](../../../raw/current/scripts/components/teamattacker.lua#L82-L83) [常量](../../../raw/current/scripts/constants.lua#L2426-L2432) [消费](../../../raw/current/scripts/components/teamattacker.lua#L93-L111) |
| `LeaveFormation()`／`JoinFormation()` | 设置 true／清为 nil | 只控制走向 formationpos；仍会 DropTarget、消费攻击命令及处理着火例外，不等于退出队伍或停用组件。[实现](../../../raw/current/scripts/components/teamattacker.lua#L74-L79) [消费](../../../raw/current/scripts/components/teamattacker.lua#L95-L111) |
| `ShouldGoHome()`／`leashdistance` | boolean／默认 70，世界距离 | 直接读取 knownlocations.home；无 home 返回 false，超距在 OnUpdate 中退队。只判断／退队，不负责发回家行动。[实现](../../../raw/current/scripts/components/teamattacker.lua#L60-L70) [更新入口](../../../raw/current/scripts/components/teamattacker.lua#L90-L93) |

### 命令消费与休眠

队长达到 `min_team_size` 才进入列阵／派遣消费。无命令、HOLD、WARN 都先丢当前 combat 目标；有阵位且未暂离时，距离平方达到 0.15 才 GoToPoint。若有阵位、非 WARN 且 health.takingfiredamage 为真，会把自身命令改为 ATTACK；因此每轮指定攻击人数不是全部情况下的硬并发上限。[更新分支](../../../raw/current/scripts/components/teamattacker.lua#L90-L111)

实体休眠由宿主分发组件 `OnEntitySleep`：队员先退队再 StopUpdatingComponent；唤醒只恢复更新，不自动找队。死亡、移除、进入 limbo 的退队由队长注册的事件处理；这两条清理路径不同。[休眠／唤醒](../../../raw/current/scripts/components/teamattacker.lua#L49-L57) [宿主分发](../../../raw/current/scripts/mainfunctions.lua#L696-L721) [唤醒分发](../../../raw/current/scripts/mainfunctions.lua#L751-L755) [成员监听](../../../raw/current/scripts/components/teamleader.lua#L139-L159)

## 典型接入

### 复用蝙蝠围攻模式，保留目标门禁与 Brain／SG

- **前提／位置：** 在已有主模拟生物的 Prefab 工厂／目标回调中接入，具有上面的配套组件、Brain、SG；新物种还需自己的资源与工厂。原版 bat 的 Retarget／OnAttacked 在无队伍时先尝试 SearchForTeam，失败才 SpawnPrefab("teamleader")、SetUp(威胁, 自己)、BroadcastDistress(自己)。不是每帧自动建队。[建队入口](../../../raw/current/scripts/prefabs/bat.lua#L43-L69) [受击入口](../../../raw/current/scripts/prefabs/bat.lua#L81-L96)
- **目标与移动：** 人数不足时 bat 允许普通目标战斗；人数达到门槛后由命令控制目标保持。Brain 的 ChaseAndAttack 配合 SG 攻击帧执行伤害；恐慌／回避电围栏会主动 LeaveTeam。复制模式时保留这些消费者，不只移植成员登记代码。[目标门禁](../../../raw/current/scripts/prefabs/bat.lua#L67-L74) [Brain](../../../raw/current/scripts/brains/batbrain.lua#L184-L220) [攻击帧](../../../raw/current/scripts/stategraphs/SGbat.lua#L918-L922)
- **局部暂离：** 酸化蝙蝠在找食物／偷硝石行动取得 act 后 LeaveFormation；Brain 每次检查酸化分支时 JoinFormation，维持成员关系。适合“暂时停止列阵让另一行为移动”，不适合完整退队。[行动与暂离](../../../raw/current/scripts/brains/batbrain.lua#L160-L177) [恢复入口](../../../raw/current/scripts/brains/batbrain.lua#L202-L209)
- **撤销边界：** 用 LeaveTeam 走队长退出流程；若要彻底停用，须先核验 Brain／目标回调仍会不会访问组件，再做统一清理。原版组件无 OnRemoveFromEntity，直接 RemoveComponent 只停止更新，不能替代关系解除。[退出实现](../../../raw/current/scripts/components/teamattacker.lua#L68-L70) [组件移除宿主](../../../raw/current/scripts/entityscript.lua#L670-L684)

## 常见陷阱与限制

- **把 SearchForTeam=true 当已登记：** 队长 ValidMember 可拒绝，函数仍返回 true；继续前检查 `inteam`／`teamleader`，否则会误以为组队成功。搜索也不要求威胁相同，修改特定目标策略时需在调用方明确筛选。[搜索](../../../raw/current/scripts/components/teamattacker.lua#L37-L45) [资格](../../../raw/current/scripts/components/teamleader.lua#L108-L115)
- **退队后命令仍残留：** 当前 OnLostTeammate 清的是 `order`（单数），实际命令字段为 `orders`；`formationpos`、`ignoreformation` 也未清。退队后组件因 teamleader=nil 不再消费命令，但独立读取 orders 的 Brain／目标回调仍可能受旧值影响；模组额外状态须自行约定清理，不能声称原版已全部复位。[退出](../../../raw/current/scripts/components/teamleader.lua#L177-L190) [命令读取](../../../raw/current/scripts/components/teamattacker.lua#L82-L83) [bat KeepTarget](../../../raw/current/scripts/prefabs/bat.lua#L72-L74)
- **照抄企鹅字符串比较：** 本页旧版（25643519）记录 penguin 的 KeepTarget 与 ShouldRunAway 都用字符串 `"ATTACK"`／`"HOLD"` 比较数字 ORDERS。`26148036` 起 KeepTarget 已修复为 `ORDERS.ATTACK`（[penguin.lua:151-152](../../../raw/current/scripts/prefabs/penguin.lua#L151-L152)），仅 Brain 的 ShouldRunAway 仍用字符串条件，不能识别组件写入的数字命令，"有命令时不逃跑"分支不生效；其余逃跑判定不受影响。模组消费命令一律参照 bat 的 `ORDERS.ATTACK` 写法。[企鹅目标](../../../raw/current/scripts/prefabs/penguin.lua#L146-L152) [企鹅 Brain](../../../raw/current/scripts/brains/penguinbrain.lua#L323-L334) [常量](../../../raw/current/scripts/constants.lua#L2426-L2432) [bat](../../../raw/current/scripts/prefabs/bat.lua#L72-L74)
- **给任意实体只挂本组件：** OnUpdate 无条件先调用 ShouldGoHome，直接访问 knownlocations；列阵分支还访问 combat／locomotor／health。缺少这些条件会报错，不存在通用空壳降级。[回家检查](../../../raw/current/scripts/components/teamattacker.lua#L60-L65) [更新](../../../raw/current/scripts/components/teamattacker.lua#L90-L111)

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 处理范围：锚点重定位 19 处；结论修订 1 条：penguin KeepTarget 已由字符串比较修复为 ORDERS.ATTACK，仅 ShouldRunAway 仍字符串（见陷阱节）。SGbat 攻击状态重写（伤害帧仍在 attack 时间线）、batbrain 包 jumping 守卫、teamleader NewTeammate 加队伍规模门控，均不改变本页登记／列阵／命令结论。
- 依据：全局枢纽 hunk 地图（R-08/R-09）＋变更清单命中 components/teamleader.lua、prefabs/bat.lua、stategraphs/SGbat.lua、brains/batbrain.lua、prefabs/penguin.lua、prefabs/pigman.lua、mainfunctions.lua、entityscript.lua、constants.lua。

- 源码审阅：2026-10-23，`steam-buildid:25643519`；完整 teamattacker／teamleader，并核验 bat 工厂、目标回调、Brain 与攻击帧、penguin 的命令读取差异及宿主休眠／组件移除路径。覆盖上述登记、列阵、命令与清理结论；未审阅所有物种、跨世界迁移或战斗所有伤害分支。
- 游戏运行：未运行；不保证上述原版差异已在实机复现，也未提供完整新生物工程。
- 结构检查：两页选页格式／本地链接检查及源码引用 strict 检查通过；未新增完整 Lua 示例，不作示例语法或游戏运行通过声明。

相关：[[reference/components/teamleader]] — 队长人数门槛、轮次计时与退队细节；[[reference/components/combat]] — 目标与伤害；[[reference/components/knownlocations]] — home 坐标。
