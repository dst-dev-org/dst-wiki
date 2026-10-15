---
title: formationleader
created: 2026-10-26
updated: "2026-10-28"
type: component
summary: 为同类成员计算旋转环形编队位置，管理入退队、多圈分层和解散；移动由成员宿主实现
aliases: [环形编队, 围绕玩家飞行, 编队队长, formationleader, 发光飞虫编队]
tags: [component, ai, lifecycle]
dst_build: "steam-buildid:26230496"
sources: [components/formationleader.lua, components/formationfollower.lua, prefabs/formationleader.lua, prefabs/lightflier.lua]
reliability: verified
validation: source-reviewed
status: active
---

# formationleader

## 快速判断

- **职责与适用：** 维护一个目标实体、成员集合与旋转圆周上的目标位置；原版宿主是非联网、`persists = false` 的 `formationleader` Prefab，不是队伍中的一只可见生物。[队长宿主:1–15](../../../raw/current/scripts/prefabs/formationleader.lua#L1-L15)、[位置生成:199–211](../../../raw/current/scripts/components/formationleader.lua#L199-L211)
- **优先使用：** 需要同类对象围绕一个中心排成一圈／多圈时配合 [[reference/components/formationfollower]]；普通归属／忠诚用 [[reference/components/follower]]，战斗轮攻用 [[reference/components/teamleader]]，不要仅凭“队长”名称互换。[成员接入:155–173](../../../raw/current/scripts/components/formationleader.lua#L155-L173)
- **重要副作用：** 成员入队可能改其普通 `follower` 主人；退队无条件访问 `combat:DropTarget()`，并可能停止普通跟随。`DisbandFormation()` 还会删除队长实体。[入队:168–172](../../../raw/current/scripts/components/formationleader.lua#L168-L172)、[退队:177–194](../../../raw/current/scripts/components/formationleader.lua#L177-L194)、[解散:121–136](../../../raw/current/scripts/components/formationleader.lua#L121-L136)
- **修改边界：** 本页核验服务端 lightflier 组装链；组件不会移动中心去追目标、不会移动成员，也不做地面可达性检查。中心追踪由宿主 `LeaderOnUpdate`，成员运动由 `FollowerOnUpdate` 实现。[中心追踪:46–67](../../../raw/current/scripts/prefabs/lightflier.lua#L46-L67)、[成员运动:126–134](../../../raw/current/scripts/prefabs/lightflier.lua#L126-L134)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SetUp` | `SetUp(target, first_member)`；无显式返回值 | 设置目标，从首成员读取 `formation_type`，再尝试入队；不是自动寻找成员的初始化器。[71–76](../../../raw/current/scripts/components/formationleader.lua#L71-L76) |
| `formation_type` | 字符串；默认 `"monster"` | 赋值维护 `formationleader_<类型>` 标签与搜索标签；首成员通常覆盖默认值。不要在已有成员时随意换型。[1–11](../../../raw/current/scripts/components/formationleader.lua#L1-L11)、[默认值:40–49](../../../raw/current/scripts/components/formationleader.lua#L40-L49) |
| `ValidMember` | `ValidMember(member)` → `true` 或 `nil` | 要求有效、非 limbo、类型标签匹配、非已死且未挂队长；直接取 `formationfollower.formationleader`，不是对任意实体都安全的探测。[110–117](../../../raw/current/scripts/components/formationleader.lua#L110-L117) |
| `NewFormationMember` | `NewFormationMember(member)` | 资格通过后执行入队回调、监听成员事件并写入成员引用；本方法不检查容量，调用方需先问 `IsFormationFull()`。[155–173](../../../raw/current/scripts/components/formationleader.lua#L155-L173) |
| `GetFormationSize`／`IsFormationFull` | 数量；`true` 或 `nil` | 数量排除带同类型队长标签的成员；默认上限 6。不是严格 `true/false` 接口。[61–68](../../../raw/current/scripts/components/formationleader.lua#L61-L68)、[106–107](../../../raw/current/scripts/components/formationleader.lua#L106-L107) |
| `radius`／`theta`／`thetaincrement`／`reverse` | 数值／数值／数值／布尔值 | 默认半径 5、随机初始角、角增量 1、正向；以 `dt` 推进角度，`reverse` 反向。半径是世界坐标距离，角度按弧度。[40–49](../../../raw/current/scripts/components/formationleader.lua#L40-L49)、[219–223](../../../raw/current/scripts/components/formationleader.lua#L219-L223) |
| `onupdatefn`／`ondisbandfn` | `fn(leader_entity)`，可省略 | 更新回调早于分层／位置计算；解散回调早于目标清空和实体移除。[239–248](../../../raw/current/scripts/components/formationleader.lua#L239-L248)、[121–136](../../../raw/current/scripts/components/formationleader.lua#L121-L136) |
| `OnLostFormationMember`／`DisbandFormation` | `(member)`／`()` | 前者退出单成员并清理监听，后者对全部成员执行退出并移除队长；详情见下文。[177–194](../../../raw/current/scripts/components/formationleader.lua#L177-L194)、[121–136](../../../raw/current/scripts/components/formationleader.lua#L121-L136) |

### 圆周位置与多圈分层

`GetFormationPositions()` 以队长实体坐标为圆心，用半径和角度写各成员的 `formationpos`，相邻间隔 `TWOPI / GetFormationSize()`。成员按 `pairs` 遍历，没有稳定槽位编号；这里只产出坐标，不检测障碍、水面或移动能力。[199–211](../../../raw/current/scripts/components/formationleader.lua#L199-L211)

每次更新都会搜索半径 `searchradius`（默认 50）内同类型且 `target` 相同的队长，按 `age` 从大到小排序；仅首个调用分层回调。默认分层从首队半径／转速／容量出发，逐队半径加 5、转向交替、转速乘 0.6、容量加 6。宿主若要求固定单圈参数，应考虑此回调会重复覆盖次圈参数，不能只改某个次圈字段便宣称永久生效。[搜索与排序:80–102](../../../raw/current/scripts/components/formationleader.lua#L80-L102)、[默认分层:15–33](../../../raw/current/scripts/components/formationleader.lua#L15-L33)

## 典型接入

### 复用原版发光飞虫的组装与中心追踪

本例是原版接线参照，不是向任意生物添加一个组件即可工作的 demo。`lightflier` 的服务端已经具备 `formationfollower`、`combat`、`follower`、`locomotor` 及专用 Brain，成员侧说明见配套页面。[服务端入口:452–454](../../../raw/current/scripts/prefabs/lightflier.lua#L452-L454)、[成员配置:511–518](../../../raw/current/scripts/prefabs/lightflier.lua#L511-L518)

1. `FindTarget` 先搜索现有同类编队；无可加入编队时才找持有 lightflier 且尚未绑定编队的玩家，调用本文件局部函数 `MakeFormation`。[目标条件:71–74](../../../raw/current/scripts/prefabs/lightflier.lua#L71-L74)、[搜索分派:220–237](../../../raw/current/scripts/prefabs/lightflier.lua#L220-L237)
2. `MakeFormation` 生成 `formationleader`，把中心放到首成员处并初始化 `_offset`，然后 `SetUp(target, inst)`。接着登记 `target._lightflier_formation` 和解散回调，再设上限 3、半径 4.1、转速 0.9、更新回调以及每秒一次的宿主核查。[参数常量:18–26](../../../raw/current/scripts/prefabs/lightflier.lua#L18-L26)、[组装:198–217](../../../raw/current/scripts/prefabs/lightflier.lua#L198-L217)
3. `LeaderOnUpdate` 对中心做插值追踪；距目标超过 16 时解散。周期核查另行检查玩家是否仍持有 lightflier，并平衡编队／为其他玩家分队。这些都是宿主约定，不是本组件自动提供的行为。[追踪:46–67](../../../raw/current/scripts/prefabs/lightflier.lua#L46-L67)、[周期核查:77–117](../../../raw/current/scripts/prefabs/lightflier.lua#L77-L117)
4. 宿主收到受击或传送时主动解散；`ondisbandfn` 清除玩家的 `_lightflier_formation`。不要把这个私有字段推广成所有编队的公共接口。[解散登记清理:192–194](../../../raw/current/scripts/prefabs/lightflier.lua#L192-L194)、[受击与传送:287–308](../../../raw/current/scripts/prefabs/lightflier.lua#L287-L308)

## 生命周期

- 构造时启动更新；每帧依次检查成员、执行宿主更新、累计年龄、组织多圈、控制容量，再在目标有效时推进角度并生成位置。空队或目标引用已失效时解散；`target == nil` 本身不满足“目标引用已失效”这一条件。[构造:37–51](../../../raw/current/scripts/components/formationleader.lua#L37-L51)、[更新:239–252](../../../raw/current/scripts/components/formationleader.lua#L239-L252)
- 入队先执行成员 `onenterformationfn(member, leader_entity)`，之后才写 `formationleader`／`in_formation` 并监听 `death/onremove/onenterlimbo`；回调不能假定本次成员登记已完成。[155–172](../../../raw/current/scripts/components/formationleader.lua#L155-L172)
- 退队先执行 `onleaveformationfn`，再取消监听、清成员状态、丢战斗目标及普通跟随。正常销毁应走 `DisbandFormation()`；本组件未提供独立移除组件时的等价清理钩子，不把直接删队长当作解散。[177–194](../../../raw/current/scripts/components/formationleader.lua#L177-L194)、[解散:121–136](../../../raw/current/scripts/components/formationleader.lua#L121-L136)

## 常见陷阱与限制

- **“资格通过”不等于任意实体可入队。** 类型标签不是完整契约；退出路径直接访问 `formationfollower` 和 `combat`，自定义宿主必须核对这些协作，不能只补标签。[资格:110–117](../../../raw/current/scripts/components/formationleader.lua#L110-L117)、[退队:177–194](../../../raw/current/scripts/components/formationleader.lua#L177-L194)
- **上限不负责每次入队前拒绝。** `NewFormationMember` 不读上限，超额由后续更新裁减，且 `pairs` 不保证保留顺序。需要即时限制时先检查容量，不依赖裁减保留特定对象。[入队:155–156](../../../raw/current/scripts/components/formationleader.lua#L155-L156)、[裁减:139–149](../../../raw/current/scripts/components/formationleader.lua#L139-L149)
- **宿主写出的字段未必有消费者。** lightflier 写 `min_formation_size = 1`，但此组件只用空队判断，不读这个字段；它不是已实现的最小人数配置。[宿主赋值:209–210](../../../raw/current/scripts/prefabs/lightflier.lua#L209-L210)、[空队:215–216](../../../raw/current/scripts/components/formationleader.lua#L215-L216)
- **更新检查不是万能清理。** `ValidateFormation` 发现无效实体后调用退出，但退出主体又要求 `member:IsValid()`；不能据此宣称无效引用必被移除。自定义直接删除／重入回调等路径未运行验证。[检查:227–234](../../../raw/current/scripts/components/formationleader.lua#L227-L234)、[退出门控:177–178](../../../raw/current/scripts/components/formationleader.lua#L177-L178)

## 验证记录

- 2026-10-26，`steam-buildid:26148036`：审阅所列四个来源，覆盖编队类型、容量、坐标生成、多圈分层、成员事件与 lightflier 创建／追踪／解散链；依据见对应章节。不代表其他自定义宿主均满足契约。
- 游戏未运行；未验证地面生物适用性、同年龄队长排序、外部直接移除与回调重入。本文未新增完整 Lua 示例，原版接线按实际源码说明。
