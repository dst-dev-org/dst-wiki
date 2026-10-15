---
title: "joustsource"
created: "2026-10-26"
updated: "2026-10-28"
type: component
summary: "给骑枪物品配置冲刺参数并执行前方线段命中检测，区分普通攻击、异向冲刺相撞与 SG 实际移动。"
aliases: ["骑枪冲刺命中", "冲刺来源物品", "骑枪相撞", "joustsource"]
tags: [component, callback, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/joustsource.lua", "prefabs/yoth_lance.lua", "stategraphs/SGwilson.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# joustsource

## 快速判断

- 职责与适用：原版 yoth_lance 物品上的冲刺参数与主动扫描器；CheckCollision 的 inst 参数是**使用者**，self.inst 才是来源物品。使用者回调／边缘检查见 [[reference/components/joustuser]]。[joustsource.lua:50](../../../raw/current/scripts/components/joustsource.lua#L50 [yoth_lance.lua:73](../../../raw/current/scripts/prefabs/yoth_lance.lua#L73
- 优先使用：保留原版动作／人物 SG 协作，配置速度、枪长、动画循环数及命中回调；只加组件不会自行更新、移动或发起冲刺。[joustsource.lua:15](../../../raw/current/scripts/components/joustsource.lua#L15 [SGwilson.lua:21749](../../../raw/current/scripts/stategraphs/SGwilson.lua#L21749
- 重要副作用：一般命中可调使用者 combat:DoAttack 并发 knockback；异向 jousting 对象走相撞分支。CheckCollision 返回 true **仅表示发生后一类相撞**，不是一般命中／掉血成功。[joustsource.lua:98](../../../raw/current/scripts/components/joustsource.lua#L98
- 修改边界：来源组件装在主模拟物品上，角色移动、方向与退出由 SG 负责；它不是通用骑乘或物理碰撞回调系统。[yoth_lance.lua:58](../../../raw/current/scripts/prefabs/yoth_lance.lua#L58 [SGwilson.lua:21825](../../../raw/current/scripts/stategraphs/SGwilson.lua#L21825

## 关键接口

| 接口／字段 | 参数或返回 | 条件、作用与证据 |
|---|---|---|
| `SetSpeed(speed)`／`GetSpeed()` | 速度；默认 TUNING.WILSON_RUN_SPEED | 只存取参数；原版 SG 在 joust_start 缓存，进入 joust 时再乘 locomotor:GetSpeedMultiplier 并设置 MotorVel，不是组件自己设速度。[joustsource.lua:1](../../../raw/current/scripts/components/joustsource.lua#L1 [joustsource.lua:15](../../../raw/current/scripts/components/joustsource.lua#L15 [SGwilson.lua:21752](../../../raw/current/scripts/stategraphs/SGwilson.lua#L21752 [SGwilson.lua:21828](../../../raw/current/scripts/stategraphs/SGwilson.lua#L21828 |
| `SetRunAnimLoopCount(loops)`／`GetRunAnimLoopCount()` | 动画循环参数，默认未赋值 | SG 缓存并据此安排运行段；不是秒数、固定距离或更新次数。循环归零后仍可有随机长度的动画尾段。[joustsource.lua:23](../../../raw/current/scripts/components/joustsource.lua#L23 [SGwilson.lua:21848](../../../raw/current/scripts/stategraphs/SGwilson.lua#L21848 [SGwilson.lua:21906](../../../raw/current/scripts/stategraphs/SGwilson.lua#L21906 |
| `SetLanceLength(length)`／`GetLanceLength()` | 世界距离，默认未赋值 | 检测端点用 length−0.6；不能遗漏配置。该长度差还参与平方长度归一化，length=0.6 会形成退化线段；不保证任意放大后的扫描完整性。[joustsource.lua:31](../../../raw/current/scripts/components/joustsource.lua#L31 [joustsource.lua:50](../../../raw/current/scripts/components/joustsource.lua#L50 |
| `SetOnHitOtherFn(fn)` | `fn(source, user, target)` | 普通非友军命中分支，在 DoAttack 与 knockback 前调用；返回值不消费，不代表已造成伤害。相撞分支不调用它。[joustsource.lua:39](../../../raw/current/scripts/components/joustsource.lua#L39 [joustsource.lua:109](../../../raw/current/scripts/components/joustsource.lua#L109 |
| `CheckCollision(user, targets)` | 使用者实体、调用方维护的时间戳表；返回 collided | 直接扫描／执行效果；没有内部“已启动”门禁，要求已配置枪长。表键为目标实体，值为上次几何命中 GetTime 秒值。[joustsource.lua:50](../../../raw/current/scripts/components/joustsource.lua#L50 |
| `collide_tags`／`no_collide_tags` | 默认 `_combat` 候选；排除 FX/NOCLICK/DECOR/INLIMBO | 构造时若 PvP 关闭，额外排除 player；本组件不持续监听 PvP 配置变化。[joustsource.lua:1](../../../raw/current/scripts/components/joustsource.lua#L1 |

## 关键流程

### 前方线段与重命中时间戳

CheckCollision 以使用者位置和 Transform 朝向构造水平线段，从使用者中心到前方 length−0.6；内部把世界 x/z 转成二维向量 x/y。宽度判定为 0.6 加目标 GetPhysicsRadius(0)，先排除投影在起点背后的目标，再检查到线段／末端的平方距离严格小于范围平方。[joustsource.lua:50](../../../raw/current/scripts/components/joustsource.lua#L50

候选查询仍以**使用者位置**为中心，半径取半段长再加 0.6+3，并非无限长度射线或以计算出的线段中点为查询中心。还要求目标有效、未死亡、使用者 combat:CanTarget 通过（若有 combat），且不被 TargetForceAttackOnly 排除。[joustsource.lua:68](../../../raw/current/scripts/components/joustsource.lua#L68 [joustsource.lua:98](../../../raw/current/scripts/components/joustsource.lua#L98

targets 中无记录或距记录时间**超过** 0.75 秒才继续几何检查；命中后立即更新时间戳，随后才判断相撞／友军。因此记录不等于造成伤害，友军分支也可能占用一次检测间隔。表不在组件内创建或保存；原版 joust_start 初始化空表，并由后续动画循环复用。[joustsource.lua:74](../../../raw/current/scripts/components/joustsource.lua#L74 [SGwilson.lua:21760](../../../raw/current/scripts/stategraphs/SGwilson.lua#L21760

### 普通攻击与相撞分流

若目标带 jousting 且双方朝向差 DiffAngle>44 度，向目标立即发 joust_collide，并记 collided=true；这个分支不做普通攻击。扫描不会因此立即 break，最终将 collided 返回调用者，由使用者 SG 再决定自身进入 joust_collide。[joustsource.lua:46](../../../raw/current/scripts/components/joustsource.lua#L46 [joustsource.lua:109](../../../raw/current/scripts/components/joustsource.lua#L109 [SGwilson.lua:21892](../../../raw/current/scripts/stategraphs/SGwilson.lua#L21892

否则，使用者无 combat 或目标非其友军时，依次执行来源 onhitotherfn、可选 combat:DoAttack，再给目标发 knockback，载荷为 knocker=user、radius=6.5、forcelanded=true。它没有独立冲刺伤害字段；伤害、武器与耐久消费交给普通战斗链，不能将特效回调或击退事件当作实际掉血确认。[joustsource.lua:113](../../../raw/current/scripts/components/joustsource.lua#L113 [[reference/components/combat#关键接口]] [[reference/components/weapon#关键接口]]

检测前若有使用者 combat，临时将 ignorehitrange=true，结束后**固定设回 false**，不是恢复进入前的旧值。与其他会长期开启此字段的机制组合时应核对覆盖关系。[joustsource.lua:98](../../../raw/current/scripts/components/joustsource.lua#L98

## 典型接入

### 原版 yoth_lance：来源参数与人物 SG 配对

`Prefab("yoth_lance", fn, assets)` 在服务端同时装 weapon、finiteuses 和 joustsource；枪长同样用于 weapon:SetRange。组件配置速度 10、枪长 2、循环参数 2，OnHitOther 只按目标大小播放 puff 特效；攻击基础值配置在 weapon 上，并非 joustsource 的新伤害通道。[yoth_lance.lua:26](../../../raw/current/scripts/prefabs/yoth_lance.lua#L26 [yoth_lance.lua:58](../../../raw/current/scripts/prefabs/yoth_lance.lua#L58 [yoth_lance.lua:92](../../../raw/current/scripts/prefabs/yoth_lance.lua#L92 [tuning.lua:9272](../../../raw/current/scripts/tuning.lua#L9272

原版人物的 joust_start 与 joust 每次更新调用本组件，joust 才设置 MotorVel、添加 jousting 标签；正常结束、卸下来源、边缘与相撞分别由 SG 退出路径处理。完整动作门禁与 Start／End 回调在 [[reference/components/joustuser#关键流程]] 维护，不将这套内部配对抽成任意武器可直接套用的 demo。[SGwilson.lua:21771](../../../raw/current/scripts/stategraphs/SGwilson.lua#L21771 [SGwilson.lua:21828](../../../raw/current/scripts/stategraphs/SGwilson.lua#L21828 [SGwilson.lua:21918](../../../raw/current/scripts/stategraphs/SGwilson.lua#L21918

## 常见陷阱与限制

- **普通命中返回 true**：只有异向 jousting 目标触发的相撞才使 collided=true；普通攻击可以发生但返回 false。[joustsource.lua:109](../../../raw/current/scripts/components/joustsource.lua#L109
- **一次冲刺每个目标只命中一次**：时间戳允许超过 0.75 秒后再次检测；跨循环复用表，下一次新冲刺则重新建表，没有永久免疫名单。[joustsource.lua:74](../../../raw/current/scripts/components/joustsource.lua#L74 [SGwilson.lua:21760](../../../raw/current/scripts/stategraphs/SGwilson.lua#L21760
- **保存／删来源组件会自动处理冲刺退出**：组件无存读、更新或 OnRemoveFromEntity 钩子；配置由物品重建时施加。原版 SG 检查来源有效性／组件存在，并处理 unequip；热移除组件不是代替 SG 清理的通用接口。[joustsource.lua:1](../../../raw/current/scripts/components/joustsource.lua#L1 [SGwilson.lua:21882](../../../raw/current/scripts/stategraphs/SGwilson.lua#L21882

## 验证记录

- 源码审阅：2026-10-26，Steam buildid 26148036；覆盖组件全文、yoth_lance 服务端组装与 tuning、SG 参数消费／时间戳复用／碰撞调用与退出。未重新展开完整战斗伤害、击退消费者或其他人物 SG，战斗分工复用现有 combat／weapon 页。
- 结构检查：validate_page 及源码引用 strict 检查通过；无新增完整 Lua 示例。机械检查不代替游戏运行。
- 游戏运行：未运行；线段端点、重复命中、PvP／友军、双方相撞、耐久与 SG 中断待实测。

相关：[[reference/components/joustuser]]；[[reference/components/combat]]；[[reference/components/weapon]]。
