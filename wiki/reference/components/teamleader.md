---
title: "teamleader"
created: "2026-10-23"
updated: "2026-10-28"
type: component
summary: "为临时围攻队伍招募成员、排列阵位并按轮次发布预警／攻击命令，管理目标、追击时间与解散。"
aliases: ["teamleader", "围攻队长", "轮流攻击人数", "队伍攻击间隔", "围攻队伍解散"]
tags: [component, creature, ai, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/teamleader.lua", "components/teamattacker.lua", "prefabs/teamleader.lua", "prefabs/bat.lua", "prefabs/penguin.lua", "constants.lua", "components/combat.lua", "entityscript.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# teamleader

## 快速判断

- 职责与适用：临时战斗协调节点，招募 [[reference/components/teamattacker]] 成员，围绕同一 threat 排列阵位并分批发布命令；不是随从主人 [[reference/components/leader]] 或持久群体 [[reference/components/herd]]。[初始化](../../../raw/current/scripts/components/teamleader.lua#L15-L40) [发布命令](../../../raw/current/scripts/components/teamleader.lua#L331-L358)
- 优先使用：原版生物侧 SpawnPrefab("teamleader") 后 SetUp(目标, 首成员)、BroadcastDistress 招募；用 SetAttackGrpSize 配每轮人数、attackinterval 配 HOLD 轮次间隔，实际攻击由成员 Brain／combat 执行。[bat 建队](../../../raw/current/scripts/prefabs/bat.lua#L46-L49) [人数选择](../../../raw/current/scripts/components/teamleader.lua#L305-L313) [成员消费](../../../raw/current/scripts/components/teamattacker.lua#L90-L111)
- 重要副作用：登记成员会安装死亡／受击／攻击／移除／limbo 监听并写双向关联；解散会逐一退队、DropTarget，随后直接删除队长实体。[登记](../../../raw/current/scripts/components/teamleader.lua#L139-L159) [退出](../../../raw/current/scripts/components/teamleader.lua#L177-L190) [解散](../../../raw/current/scripts/components/teamleader.lua#L118-L124)
- 修改边界：原版 teamleader Prefab 是仅有 Transform 的非联网节点，由主模拟生物建出；`CanAttack()` 只判人数门槛，不是目标合法性、冷却或伤害资格检查。[节点工厂](../../../raw/current/scripts/prefabs/teamleader.lua#L1-L13) [主模拟入口](../../../raw/current/scripts/prefabs/bat.lua#L247-L249) [门槛](../../../raw/current/scripts/components/teamleader.lua#L194-L195)

## 关键接口

| 接口／字段 | 参数或默认值 | 条件、作用与证据 |
|---|---|---|
| `SetUp(target, first_member)` | 威胁实体／有 teamattacker 的首成员 | 先 SetNewThreat，再继承成员 team_type 并 NewTeammate；不广播招募、不保证成功，也不清空已有队伍，适合新节点。[实现](../../../raw/current/scripts/components/teamleader.lua#L53-L59) |
| `team_type`／`searchradius` | 默认 `"monster"`／50 | 类型属性更新队长标签及队长／成员搜索标签；招募、成员主动找队和多队整编据此匹配，不是战斗阵营系统。改类型不自动迁移已有成员。[属性](../../../raw/current/scripts/components/teamleader.lua#L1-L18) [招募](../../../raw/current/scripts/components/teamleader.lua#L163-L175) |
| `NewTeammate(member)`／`ValidMember(member)` | 无显式返回／boolean | 要求有效、不在 limbo、有 combat、未死、未组队、同类型标签且成员回调允许；直接访问 teamattacker，调用方需保证组件存在。NewTeammate 不检查满员。[资格与登记](../../../raw/current/scripts/components/teamleader.lua#L108-L116) [登记](../../../raw/current/scripts/components/teamleader.lua#L139-L160) |
| `BroadcastDistress(member?)` | 默认从队长位置招募 | 以所给实体为中心搜索同类型成员，逐一 ValidMember／NewTeammate；不按 max_team_size 截止，下一次 TeamSizeControl 才裁减。[广播](../../../raw/current/scripts/components/teamleader.lua#L163-L175) [裁减](../../../raw/current/scripts/components/teamleader.lua#L127-L136) |
| `GetTeamSize()`／`IsTeamFull()`／`CanAttack()` | 数量／boolean／boolean | 计数排除带 `teamleader_<本类型>` 的记录；满员为 count>=max，攻击门槛为 count>=min，默认 min=3、max=6。空集合判断另用 IsTeamEmpty。[计数](../../../raw/current/scripts/components/teamleader.lua#L43-L50) [满员](../../../raw/current/scripts/components/teamleader.lua#L104-L105) [攻击门槛](../../../raw/current/scripts/components/teamleader.lua#L194-L195) [空集合](../../../raw/current/scripts/components/teamleader.lua#L276-L277) |
| `SetAttackGrpSize(val)`／`NumberToAttack()` | 数字或**无参数**函数 | 函数直接 `fn()`，不传 self；否则用数字，再否则随机选 1（75%）或 2（25%）。用于 GiveOrders，调用方提供非负整数；实际派遣数封顶为集合条目数。[选择](../../../raw/current/scripts/components/teamleader.lua#L305-L313) [封顶](../../../raw/current/scripts/components/teamleader.lua#L228-L248) |
| `timebetweenattacks`／`attackinterval` | 默认 3／3 秒 | 前者是当前倒计时，后者是每次开始新轮时重装的值；只有有威胁、人数达标且 AllInState(HOLD) 时才递减。改 interval 不会立刻修改当前倒计时。[初值](../../../raw/current/scripts/components/teamleader.lua#L28-L36) [消费](../../../raw/current/scripts/components/teamleader.lua#L338-L352) |
| `SetNewThreat(threat)` | 实体或 nil | 转移 threat 的 onremove 与 death 监听；目标实体被移除或死亡都解散，不重置 chasetime、已有命令或阵位。[实现](../../../raw/current/scripts/components/teamleader.lua#L280-L297) |
| `chasetime`／`maxchasetime` | 默认 0／30 秒 | 每次更新累加，严格大于上限就解散；成员 onattackother 在队伍规模达标时将其归零。不是攻击次数、绝对寿命或仅成功扣血才续期。[追击](../../../raw/current/scripts/components/teamleader.lua#L316-L319) [成员回调](../../../raw/current/scripts/components/teamleader.lua#L139-L150) [事件位置](../../../raw/current/scripts/components/combat.lua#L1147-L1166) |
| `OnLostTeammate(member)`／`DisbandTeam()` | 成员／无参数 | 退出对仍有效成员撤监听、清引用／inteam 并 DropTarget，再删集合记录；解散遍历退出并 Remove 队长。字段残留边界见队员页。[退出](../../../raw/current/scripts/components/teamleader.lua#L177-L190) [解散](../../../raw/current/scripts/components/teamleader.lua#L118-L124) |

（26148036 核验：SetNewThreat 新增监听 threat 的 "death"——威胁死亡与被移除都触发解散，旧版仅 onremove；SetUp 末尾新增 CenterLeader 调用，队伍整编排序对 lifetime 相等者按 GUID 平局稳定化；其余接口行为未变。）

### 一轮派遣如何推进

OnUpdate 先累加追击时长并按需解散，随后移动队长至成员平均位置、增加 lifetime、整编附近同目标队伍、裁减超员；有 threat 且人数达标才旋转角度、写每个成员 formationpos。WARN 成员半径比当前 radius 小 1。[更新顺序](../../../raw/current/scripts/components/teamleader.lua#L331-L358) [队伍中心](../../../raw/current/scripts/components/teamleader.lua#L198-L208) [阵位](../../../raw/current/scripts/components/teamleader.lua#L212-L224)

AllInState(HOLD) 将 nil 当作就绪；默认 chk_state=true 时，冻结或燃烧者也不阻塞。倒计时结束先 GiveOrders(WARN,N)：清所有 orders，随机选 N 人 WARN，其余 HOLD；0.5 秒后把仍为 WARN 者改 ATTACK。成员 onattackother 在队伍规模 ≥ max(2, min_team_size) 时才 DropTarget、设 HOLD、重置全队 chasetime。事件在伤害结算前发出，不能写成“命中造成伤害后归队”。[就绪判定](../../../raw/current/scripts/components/teamleader.lua#L260-L273) [命令](../../../raw/current/scripts/components/teamleader.lua#L228-L256) [延迟派遣](../../../raw/current/scripts/components/teamleader.lua#L345-L352) [归队回调](../../../raw/current/scripts/components/teamleader.lua#L139-L150) [事件](../../../raw/current/scripts/components/combat.lua#L1147-L1166)

（26148036 核验：onattackother 的"原地驻守"处理新增队伍规模门槛——小于 max(2, min_team_size) 的小队攻击他人后不再强制 DropTarget／HOLD／清 chasetime，成员可继续追击；旧版无条件执行。）

## 典型接入

### 选用蝙蝠建队入口，理解企鹅的局部配置

| 原版模式 | 接入位置与作用 | 复用边界与依据 |
|---|---|---|
| bat：目标出现／受击后建临时队 | 主模拟目标回调先尝试成员 SearchForTeam；没有可用队伍再创建非联网 teamleader、SetUp、BroadcastDistress | 原版默认每轮 1～2 人、人数门槛 3；成员自身仍需目标门禁、Brain／SG，见 [[reference/components/teamattacker#典型接入]]。[入口](../../../raw/current/scripts/prefabs/bat.lua#L43-L74) [受击](../../../raw/current/scripts/prefabs/bat.lua#L81-L96) |
| penguin：短追击、多成员一轮 | MakeTeam 在新节点上设威胁、每队随机一次 6～8 人、首轮倒计时 0、interval=2、maxchasetime=10、min=0，再招募 | 函数传入的是已算出的数字，不是每轮重随机。radius=10／max=8 的局部赋值随后会被 OrganizeTeams 改写，不能当稳定配置示例。[企鹅配置](../../../raw/current/scripts/prefabs/penguin.lua#L83-L98) [整编覆盖](../../../raw/current/scripts/components/teamleader.lua#L69-L100) |

若模组调整“每轮人数／间隔”，应在自己创建队长的同一入口设置 SetAttackGrpSize、attackinterval，是否立即起新轮另设 timebetweenattacks；这不是完整新生物 demo。已有队伍直接 GiveOrders 会清掉全队命令，绕过当前轮次节奏，不作为普通配置入口。[设置与消费](../../../raw/current/scripts/components/teamleader.lua#L305-L313) [派令副作用](../../../raw/current/scripts/components/teamleader.lua#L228-L248) [计时](../../../raw/current/scripts/components/teamleader.lua#L345-L352)

## 常见陷阱与限制

- **将每轮人数当硬攻击并发上限：** GiveOrders 限制本次抽样，不控制所有战斗来源；队员着火可自行切 ATTACK，冻结／燃烧成员在默认就绪判定中不阻塞，人数不足时 bat 走普通目标战斗。硬并发限制需要额外核验全部消费者。[着火例外](../../../raw/current/scripts/components/teamattacker.lua#L93-L111) [就绪例外](../../../raw/current/scripts/components/teamleader.lua#L260-L273) [人数不足](../../../raw/current/scripts/prefabs/bat.lua#L67-L74)
- **只改 radius／max_team_size，下一帧又变：** OrganizeTeams 每帧查同类型、同 threat 的附近节点，由 lifetime 最大者分配 radius=5、max=6 起步，后续每队分别递增 5／6，反转旋转方向并缩小角速度。只一队也参与；需修改整编策略才能保证长期自定义值。[整编](../../../raw/current/scripts/components/teamleader.lua#L63-L100) [调用](../../../raw/current/scripts/components/teamleader.lua#L331-L336)
- **把招募上限当立即硬上限：** NewTeammate／BroadcastDistress 不判 IsTeamFull；随后 TeamSizeControl 才删超额条目，按 pairs 顺序，不承诺保留最近或最早者。[招募](../../../raw/current/scripts/components/teamleader.lua#L139-L171) [裁减](../../../raw/current/scripts/components/teamleader.lua#L127-L136)
- **直接删除／更换队长组件：** 两组件没有组件移除清理钩子；原版完整退出依靠 OnLostTeammate／DisbandTeam，先退旧队再入新队，不能直接覆写引用。队长还将 deathfn／attackedfn／attackedotherfn 写到成员实体，模组勿占用同名字段；残留 orders 见 [[reference/components/teamattacker#常见陷阱与限制]]。[登记](../../../raw/current/scripts/components/teamleader.lua#L139-L159) [退出](../../../raw/current/scripts/components/teamleader.lua#L177-L190) [组件移除](../../../raw/current/scripts/entityscript.lua#L670-L684)
- **新威胁等于新追击轮：** SetNewThreat 只换目标和移除监听，已有 chasetime／成员命令不重置；初次配置可参照新建节点，运行中换目标需自行明确旧命令、旧阵位和计时的处理。组件没有存档恢复接口，本页不提供跨存档队伍续接方案。[威胁设置](../../../raw/current/scripts/components/teamleader.lua#L280-L297) [更新](../../../raw/current/scripts/components/teamleader.lua#L331-L358)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 的全部变更文件对照 build-26148036 更新报告逐条核验，锚点重定位 34 处、结论修订 3 处（死亡解散/驻守门槛/锚点重定位，见正文"26148036 核验"标注）；未覆盖：运行验证。

- 源码审阅：2026-10-23，`steam-buildid:25643519`；完整两组件及非联网 teamleader 工厂，核验 bat／penguin 建队与配置、bat 目标门禁、combat.onattackother 发出位置和组件移除宿主。覆盖招募、人数门槛、轮次、整编覆盖、追击及解散；未审阅所有物种组合与完整战斗效果。
- 游戏运行：未运行；未验证多队接战、冻结／着火、目标更换或存读档的实机表现，不提供可独立运行的新生物 demo。
- 结构检查：两页选页格式／本地链接检查及源码引用 strict 检查通过；未新增完整 Lua 示例，不作示例语法或游戏运行通过声明。

相关：[[reference/components/teamattacker]] — 成员命令与原版 AI 接入；[[reference/components/combat]] — 真正的目标和攻击结算。
