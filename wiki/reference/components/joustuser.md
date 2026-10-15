---
title: "joustuser"
created: "2026-10-26"
updated: "2026-10-28"
type: component
summary: "给人物提供冲刺资格回调、开始／结束通知和三方向边缘检测，实际运动与清理由 joust 状态机承担。"
aliases: ["人物骑枪冲刺", "冲刺边缘检测", "阻止骑枪冲刺", "joustuser"]
tags: [component, callback, lifecycle, state-machine]
dst_build: "steam-buildid:26230496"
sources: ["components/joustuser.lua", "prefabs/player_common.lua", "prefabs/player_common_extensions.lua", "actions.lua", "componentactions.lua", "componentutil.lua", "bufferedaction.lua", "stategraphs/SGwilson.lua", "stategraphs/SGwilson_client.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# joustuser

## 快速判断

- 职责与适用：使用者侧的资格、Start／End 回调与前方边缘测试。来源物品的命中／相撞在 [[reference/components/joustsource]]；本组件自己不设置速度、造成伤害或维护 active 状态。[joustuser.lua:1](../../../raw/current/scripts/components/joustuser.lua#L1
- 优先使用：沿用人物原版 SG 时配置回调与 edgedistance；需要拒绝动作时 CanJoust 回调应返回失败原因，仅返回 false 不保证原版前摇停止。[joustuser.lua:7](../../../raw/current/scripts/components/joustuser.lua#L7 [SGwilson.lua:21715](../../../raw/current/scripts/stategraphs/SGwilson.lua#L21715
- 重要副作用：StartJoust／EndJoust 只是回调转发，不互相配对、不查 CanJoust；原版循环状态重入可多次 Start，真正退出才 End，回调应允许这种调用方式。[joustuser.lua:24](../../../raw/current/scripts/components/joustuser.lua#L24 [SGwilson.lua:21906](../../../raw/current/scripts/stategraphs/SGwilson.lua#L21906
- 修改边界：player_common 在主模拟安装组件；客户端 joust_pre 只预览并等待服务器状态。这里的原版动作候选还明确排除骑乘中使用者，不能按名称推成通用马上战斗。[player_common.lua:2613](../../../raw/current/scripts/prefabs/player_common.lua#L2613 [player_common.lua:2861](../../../raw/current/scripts/prefabs/player_common.lua#L2861 [componentactions.lua:2127](../../../raw/current/scripts/componentactions.lua#L2127 [SGwilson_client.lua:6307](../../../raw/current/scripts/stategraphs/SGwilson_client.lua#L6307

## 关键接口

| 接口／字段 | 参数或返回 | 条件、作用与证据 |
|---|---|---|
| `SetCanJoustFn(fn)`／`CanJoust()` | `fn(user)`；结果与失败原因 | 未配置时返回 true；有回调则原样返回其结果。动作会使用结果，但原版 SG 前摇用 reason 决定是否继续；拒绝应返回 false 及有效失败原因。[joustuser.lua:7](../../../raw/current/scripts/components/joustuser.lua#L7 [actions.lua:6683](../../../raw/current/scripts/actions.lua#L6683 |
| `SetOnStartJoustFn(fn)`／`StartJoust()` | `fn(user)` | 无条件调用已配置回调，不保存“开始了”标记，不消费回调返回值；直接调用不会自动切 SG 或运动。[joustuser.lua:20](../../../raw/current/scripts/components/joustuser.lua#L20 |
| `SetOnEndJoustFn(fn)`／`EndJoust()` | `fn(user)` | 无条件调用已配置回调，无任务／碰撞／速度的内置撤销。[joustuser.lua:30](../../../raw/current/scripts/components/joustuser.lua#L30 |
| `SetEdgeDistance(distance)`／`edgedistance` | 世界距离；默认 2 | 前方探测距离，不是枪长、角色半径或累计路程。[joustuser.lua:1](../../../raw/current/scripts/components/joustuser.lua#L1 [joustuser.lua:56](../../../raw/current/scripts/components/joustuser.lua#L56 |
| `CheckEdge()` | 布尔值 | 按当前 Transform 朝向，在 0、+30、−30 度各取距离 edgedistance 的点；每点“不在地上或被 GroundTargetBlocked”才算不安全，**三个点全部不安全**才返回 true。[joustuser.lua:40](../../../raw/current/scripts/components/joustuser.lua#L40 |

## 关键流程

### 动作资格、来源与方向

来源物品在 POINT／EQUIPPED 的 joustsource 候选中提供 JOUST：均要求右键且使用者未骑乘，POINT 另外要求所点位置在地上。JOUST 动作距离为 math.huge、invalid_hold_action=true、silent_generic_fail=true；无穷动作距离不是实际冲刺路程。[componentactions.lua:2127](../../../raw/current/scripts/componentactions.lua#L2127 [componentactions.lua:2366](../../../raw/current/scripts/componentactions.lua#L2366 [actions.lua:713](../../../raw/current/scripts/actions.lua#L713

权威动作要求 doer／invobject 存在、分别具有 joustuser／joustsource，并先处理物品拟态揭露，再返回 CanJoust。来源存在不等于动作目标必须一直有效：joust_pre 完成 PerformBufferedAction 后只按保存动作的 reason 判断继续；BufferedAction:Do 会记录动作第二返回值。因此 false 且无 reason 仍可能继续进入 joust_start，这是回调要求提供失败原因的实际消费者。[actions.lua:6683](../../../raw/current/scripts/actions.lua#L6683 [bufferedaction.lua:22](../../../raw/current/scripts/bufferedaction.lua#L22 [SGwilson.lua:21715](../../../raw/current/scripts/stategraphs/SGwilson.lua#L21715

CreatingJoustingData 从当前 BufferedAction 取得 invobject 为来源，方向优先取有效 target 的位置，否则取使用者当前朝向；该函数本身不读取 action.pos。joust_pre 把显示／命中朝向吸附到 45 度，而 joustdata.dir 保留真实移动方向，不能将二者混作同一精度。[componentutil.lua:2045](../../../raw/current/scripts/componentutil.lua#L2045 [SGwilson.lua:21702](../../../raw/current/scripts/stategraphs/SGwilson.lua#L21702

### 服务端运动、边缘停止与退出

1. **joust_start** 校验来源，创建 targets 时间戳表及 edgecount=0，缓存源速度／循环参数；停 locomotor、播放前置动画并关闭 CHARACTERS 碰撞。每次更新先做来源命中检测，再检查边缘，动画完成才进 joust。[SGwilson.lua:21749](../../../raw/current/scripts/stategraphs/SGwilson.lua#L21749
2. **joust** 添加 jousting 标签，按缓存速度×locomotor 倍率及真实方向 SetMotorVel，并调用 StartJoust。来源每次更新必须仍有效且有组件；命中检测返回相撞时转 joust_collide。CheckEdge 为假就将 edgecount 清零，为真时累加，原版在连续第 4 次不安全检测时停止，不是任意一个方向触边就停，也不是“3 秒后停”。[SGwilson.lua:21825](../../../raw/current/scripts/stategraphs/SGwilson.lua#L21825 [SGwilson.lua:21882](../../../raw/current/scripts/stategraphs/SGwilson.lua#L21882
3. **循环重入** 复用 joustdata；loop>0 时超时递减并再次进入 joust，归零后使用随机长度尾段。每次 onenter 都再调 StartJoust，而继续循环时 onexit 用 statemem.jousting 跳过 End。回调计数不能直接当作完整冲刺次数。[SGwilson.lua:21848](../../../raw/current/scripts/stategraphs/SGwilson.lua#L21848 [SGwilson.lua:21906](../../../raw/current/scripts/stategraphs/SGwilson.lua#L21906 [SGwilson.lua:21931](../../../raw/current/scripts/stategraphs/SGwilson.lua#L21931
4. **真正退出** joust 时移除 jousting、停止 Physics、恢复角色碰撞并 Teleport 当前点使物理更新，再调 EndJoust（组件仍存在时）。卸下来源会转 idle；停止／相撞各自进入对应后续状态。joust_stop 还负责恢复显示动作与四方向，退出时 Physics:Stop；joust_collide 播放碰撞动画后转 slip_fall_loop，不能把事件本身当作额外伤害。[SGwilson.lua:21918](../../../raw/current/scripts/stategraphs/SGwilson.lua#L21918 [SGwilson.lua:21950](../../../raw/current/scripts/stategraphs/SGwilson.lua#L21950 [SGwilson.lua:21986](../../../raw/current/scripts/stategraphs/SGwilson.lua#L21986 [SGwilson.lua:22035](../../../raw/current/scripts/stategraphs/SGwilson.lua#L22035

客户端只在 joust_pre 停移动、预测八方向、播前摇并 PerformPreviewBufferedAction；匹配服务器状态后退出预测，超时清动作。不是客户端也运行另一份 CheckCollision／伤害检测。[SGwilson_client.lua:790](../../../raw/current/scripts/stategraphs/SGwilson_client.lua#L790 [SGwilson_client.lua:6307](../../../raw/current/scripts/stategraphs/SGwilson_client.lua#L6307

## 典型接入

### 原版人物：Start／End 回调维护冲刺尾迹

player_common 安装组件，接 player_common_extensions 的 OnStartJoust／OnEndJoust，并设探边距离 2；该安装段没有设置 CanJoust 回调。OnStartJoust 会先取消旧的 sg.mem.jousttrailtask，再创建周期尾迹特效任务；OnEndJoust 取消并清空它，适配 SG 循环多次开始通知的方式。该案例展示回调配套，不代表 StartJoust 自带加速、扣耐久或攻击行为。[player_common.lua:2861](../../../raw/current/scripts/prefabs/player_common.lua#L2861 [player_common_extensions.lua:1170](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L1170 [player_common_extensions.lua:1198](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L1198

## 常见陷阱与限制

- **单独加组件就能冲刺／骑战**：它没有自动更新、动作驱动或运动状态；需来源、动作、人物 SG、Physics／locomotor 与动画配套。原版候选排除骑乘，不能由译名推导坐骑支持。[joustuser.lua:1](../../../raw/current/scripts/components/joustuser.lua#L1 [componentactions.lua:2366](../../../raw/current/scripts/componentactions.lua#L2366
- **删组件会自动 End、恢复碰撞或保存当前冲刺**：没有 OnRemoveFromEntity、OnSave／OnLoad；组件回调配置和探边距离由宿主重建。SG 才持有运行状态／targets／任务关联，热删组件可能使退出时找不到 End 回调，不是安全撤销方案。[joustuser.lua:1](../../../raw/current/scripts/components/joustuser.lua#L1 [SGwilson.lua:21931](../../../raw/current/scripts/stategraphs/SGwilson.lua#L21931

## 验证记录

- 源码审阅：2026-10-26，Steam buildid 26148036；覆盖组件全文、人物安装与尾迹回调、POINT／EQUIPPED 候选、权威动作和 BufferedAction 原因传递、方向数据及服务端 joust 状态族／客户端预览。未展开完整玩家保存系统、其他人物 SG、击退／摔倒状态内部或地图原生阻挡判断。
- 结构检查：validate_page 及源码引用 strict 检查通过；无新增完整 Lua 示例。机械检查不代替游戏运行。
- 游戏运行：未运行；失败原因、目标失效、方向吸附、边缘连测、循环重入与卸装／相撞中断待实测。

相关：[[reference/components/joustsource]]；[[reference/components/locomotor]]；[[reference/components/combat]]。
