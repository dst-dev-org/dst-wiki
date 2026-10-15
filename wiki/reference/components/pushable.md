---
title: "pushable"
created: 2026-10-26
updated: "2026-10-28"
type: component
summary: "由被推实体管理单个推者、前向速度和间距补偿，配合玩家推动状态图完成开始、持续与停止推滚。"
aliases: ["pushable", "推动物体", "滚雪球", "推者与被推物间距", "START_PUSHING"]
tags: [component, lifecycle, callback, state-machine, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/pushable.lua", "actions.lua", "componentactions.lua", "stategraphs/SGwilson.lua", "prefabs/snowman.lua", "prefabs/snowball_item.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# pushable

## 快速判断

- 职责与适用：组件装在被推物 `inst` 上，保存一个推者 `doer`，持续设置被推物的本地前向 motor 速度；不是对任意方向施加冲量的通用推力组件。原版雪人、雪球在服务端接入。[pushable.lua:1-10](../../../raw/current/scripts/components/pushable.lua#L1-L10) [pushable.lua:77-95](../../../raw/current/scripts/components/pushable.lua#L77-L95) [snowman.lua:653-665](../../../raw/current/scripts/prefabs/snowman.lua#L653-L665) [snowball_item.lua:203-214](../../../raw/current/scripts/prefabs/snowball_item.lua#L203-L214)
- 优先使用：需要原版“玩家进入推动动画并持续推滚目标”的交互时，核对 START_PUSHING、推者 SG 和目标回调完整接线；只给物体加组件并不负责推者动画、朝向或输入。[actions.lua:6191-6196](../../../raw/current/scripts/actions.lua#L6191-L6196) [SGwilson.lua:24965-25018](../../../raw/current/scripts/stategraphs/SGwilson.lua#L24965-L25018)
- 重要副作用：StartPushing 先结束旧推动，再检查新推者；停止会对被推物 Physics:Stop、停组件更新并执行宿主回调。开始回调可能替换或移除原目标，原版雪球就会转成雪人。[pushable.lua:48-75](../../../raw/current/scripts/components/pushable.lua#L48-L75) [snowball_item.lua:125-147](../../../raw/current/scripts/prefabs/snowball_item.lua#L125-L147)
- 修改边界：持续推动要求推者有效且其 SG 带 `pushing_walk`；组件不检查障碍、安全地形或物体可推资格，也不自行朝向推者。原版雪人资格与朝向由宿主控制。[pushable.lua:77-93](../../../raw/current/scripts/components/pushable.lua#L77-L93) [snowman.lua:185-193](../../../raw/current/scripts/prefabs/snowman.lua#L185-L193) [snowman.lua:219-233](../../../raw/current/scripts/prefabs/snowman.lua#L219-L233)

## 关键接口

| 接口／字段 | 签名或约束 | 条件、作用与证据 |
|---|---|---|
| `StartPushing(doer)` | 有效实体且有 sg；无显式返回值 | 无条件先 StopPushing；登记新 doer、开始更新、调用 `onstartpushingfn(inst,doer)`，向被推物发 `startpushing`。若回调后仍是同一推者，立即 OnUpdate(0)。[pushable.lua:48-62](../../../raw/current/scripts/components/pushable.lua#L48-L62) |
| `StopPushing(doer?)` | 不传则停止当前推动；传入须匹配当前 doer | 先清 doer，再停被推物物理与更新，执行停止回调并向被推物发 `stoppushing`，两事件都携带 `{doer=doer}`。不匹配时无动作。[pushable.lua:64-75](../../../raw/current/scripts/components/pushable.lua#L64-L75) |
| `IsPushing()` | 返回 doer 是否非nil | 不重新验证实体、SG或距离；刚启动后也可能因立即更新而已停止。[pushable.lua:36-38](../../../raw/current/scripts/components/pushable.lua#L36-L38) [pushable.lua:58-60](../../../raw/current/scripts/components/pushable.lua#L58-L60) |
| `SetPushingSpeed(speed)`／`GetPushingSpeed()` | 默认3；前向运动速度 | 被推物用作基础速度；玩家推动 SG 也用此值控制推者前进。不是力、加速度或每帧位移。[pushable.lua:9-10](../../../raw/current/scripts/components/pushable.lua#L9-L10) [pushable.lua:24-26](../../../raw/current/scripts/components/pushable.lua#L24-L26) [pushable.lua:40-42](../../../raw/current/scripts/components/pushable.lua#L40-L42) [SGwilson.lua:25009-25014](../../../raw/current/scripts/stategraphs/SGwilson.lua#L25009-L25014) |
| `SetTargetDist(dist)` | 相对推者物理半径的目标中心距离偏移 | 可选，默认未设置；影响被推物速度补偿，不移动到指定位置、不自动改变朝向。[pushable.lua:12-14](../../../raw/current/scripts/components/pushable.lua#L12-L14) [pushable.lua:85-91](../../../raw/current/scripts/components/pushable.lua#L85-L91) |
| `SetMinDist(dist)`／`ShouldStopForwardMotion()` | 可选；返回近距判定或nil／false | 检查推者到物体是否在 `推者物理半径+mindist` 内；组件仅提供判断，玩家 SG 消费结果以停止推者前进，物体本身仍由 OnUpdate 驱动。[pushable.lua:16-18](../../../raw/current/scripts/components/pushable.lua#L16-L18) [pushable.lua:44-46](../../../raw/current/scripts/components/pushable.lua#L44-L46) [SGwilson.lua:25051-25055](../../../raw/current/scripts/stategraphs/SGwilson.lua#L25051-L25055) |
| `SetMaxDist(dist)` | 可选的分离距离偏移 | 超出 `推者物理半径+maxdist` 时取消推动；不是最大可移动距离或以出生点为中心的活动范围。[pushable.lua:20-22](../../../raw/current/scripts/components/pushable.lua#L20-L22) [pushable.lua:77-84](../../../raw/current/scripts/components/pushable.lua#L77-L84) |
| `SetOnStartPushingFn(fn)`／`SetOnStopPushingFn(fn)` | 各单槽可选 `fn(inst,doer)` | 设置回调，不立即调用；组件不替宿主回滚回调中的动画、任务或实体替换。[pushable.lua:28-34](../../../raw/current/scripts/components/pushable.lua#L28-L34) [pushable.lua:48-75](../../../raw/current/scripts/components/pushable.lua#L48-L75) |

## 关键流程

### 组件驱动物体，SG 驱动推者

更新先验证推者 sg 存在、带 `pushing_walk`、实体有效且未超 maxdist；不满足则 StopPushing。否则基础速度为 speed；设置 targetdist 时计算 `目标中心距=推者物理半径+targetdist`，加上 `clamp((目标中心距-当前中心距)*10,-0.5*speed,2*speed)` 补偿，最后 `inst.Physics:SetMotorVel(speed,0,0)`。在正基础速度下，补偿后范围为0.5～3倍基础速度；它不按dt积分，不沿“推者到物体向量”自动定向。[pushable.lua:77-93](../../../raw/current/scripts/components/pushable.lua#L77-L93)

玩家 pushing_walk 状态对推者另设 motor 速度：初始为目标基础速度的0.4倍，动画前几帧过渡到1倍；每次更新再按 ShouldStopForwardMotion 决定停止或前进。目标组件的自我速度补偿与玩家 SG 的近距停步是两套协作，不能只保留其中一边就期望间距稳定。[SGwilson.lua:25000-25018](../../../raw/current/scripts/stategraphs/SGwilson.lua#L25000-L25018) [SGwilson.lua:25051-25075](../../../raw/current/scripts/stategraphs/SGwilson.lua#L25051-L25075)

### 动作开始与结束时序

右键组件动作入口提供 START_PUSHING；动作执行只在目标有组件且未推动时调用 StartPushing，并返回true，不代表推动能持续。SGwilson 将动作映射到 pushing_walk_pre，该状态自身已有 `pushing_walk` 标签；2帧后执行动作并转正式推动状态，使 StartPushing 内立即更新时能通过标签门槛。[componentactions.lua:699-703](../../../raw/current/scripts/componentactions.lua#L699-L703) [actions.lua:6191-6196](../../../raw/current/scripts/actions.lua#L6191-L6196) [SGwilson.lua:1715](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1715 [SGwilson.lua:24965-24997](../../../raw/current/scripts/stategraphs/SGwilson.lua#L24965-L24997)

正式推动状态发现目标无效或不再由该玩家推动就退出；允许停止后，玩家不再按右键／手柄替代动作键会先停止自身运动，再延后3帧退出，让目标稍晚停止。状态退出对推者 Physics:Stop，并对有效目标调用 `StopPushing(inst)`；这样旧推者退出不会误停已经换给新推者的推动。[SGwilson.lua:25020-25049](../../../raw/current/scripts/stategraphs/SGwilson.lua#L25020-L25049) [SGwilson.lua:25077-25088](../../../raw/current/scripts/stategraphs/SGwilson.lua#L25077-L25088)

组件删除钩子直接复用 StopPushing；组件无独立睡眠／死亡／存读钩子，也不保存 doer、速度配置或推动进度。已保存的宿主数据如何恢复不由此组件完成；新建实例默认未推动。[pushable.lua:1-10](../../../raw/current/scripts/components/pushable.lua#L1-L10) [pushable.lua:64-97](../../../raw/current/scripts/components/pushable.lua#L64-L97)

## 典型接入

### 雪人与雪球：宿主负责资格、方向与目标替换

雪人的服务端工厂添加组件，基础滚动速度来自 `SNOWBALL_ROLLING_SPEED=3`；按体型设置 target／min／max 三个间距。开始回调将目标朝向设为从推者指向物体的方向，播放滚动动画、禁止拾取并启动宿主成长任务；停止回调负责结束相关任务／特效、恢复静止动画，小型雪人还会转成雪球物品。成长细节属于宿主，本页只核验推动接线。[snowman.lua:80-110](../../../raw/current/scripts/prefabs/snowman.lua#L80-L110) [snowman.lua:185-217](../../../raw/current/scripts/prefabs/snowman.lua#L185-L217) [snowman.lua:653-665](../../../raw/current/scripts/prefabs/snowman.lua#L653-L665) [tuning.lua:8571](../../../raw/current/scripts/tuning.lua#L8571

雪人被装饰、叠放、戴帽、上蜡或正在由装饰者操作时，宿主资格检查移除 pushable；放入库存也显式 StopPushing。起始雪球的开始回调则消费／移除原物品、创建 small 雪人，对新目标 StartPushing，再向推者 SG 发 `pushable_targetswap`。pre 状态在执行动作期间接收换目标，不能把开始回调写成“原实体必定仍存在”的协议。[snowman.lua:195-233](../../../raw/current/scripts/prefabs/snowman.lua#L195-L233) [snowball_item.lua:125-147](../../../raw/current/scripts/prefabs/snowball_item.lua#L125-L147) [SGwilson.lua:24975-24996](../../../raw/current/scripts/stategraphs/SGwilson.lua#L24975-L24996)

这是原版内部交互参照。自定义目标须具备相应 Physics、Transform、资源与回调，并保证推者支持所需 SG；本页不提供只添加组件的通用推动 demo。

## 常见陷阱与限制

- **直接 StartPushing 就能让普通生物持续推：** 初检只要求有效且有SG，但立即 OnUpdate 要求 pushing_walk 标签；缺对应状态时会开始后马上停止。[pushable.lua:48-62](../../../raw/current/scripts/components/pushable.lua#L48-L62) [pushable.lua:77-84](../../../raw/current/scripts/components/pushable.lua#L77-L84)
- **无效新推者不会影响旧推者：** StartPushing 在检查新参数前先无条件停止旧推动；若要保留旧操作，调用方应先完成自身资格判断。[pushable.lua:48-50](../../../raw/current/scripts/components/pushable.lua#L48-L50)
- **min／target／max 是被推物半径：** 三者最终都与推者半径组合；原版再由宿主把被推物动画／物理半径折进配置。组件未校验范围、排序或速度正负，不能当自动碰撞安全器。[pushable.lua:12-26](../../../raw/current/scripts/components/pushable.lua#L12-L26) [pushable.lua:44-46](../../../raw/current/scripts/components/pushable.lua#L44-L46) [snowman.lua:201-207](../../../raw/current/scripts/prefabs/snowman.lua#L201-L207)

## 验证记录

- 2026-10-26，`steam-buildid:26148036`：审阅组件全文、右键动作入口与动作执行、SGwilson 完整 pre／推动状态、雪人安装／间距／开始停止／资格与库存回调、雪球目标替换及速度配置，覆盖推者归属、双向运动分工、回调重入与退出边界。
- 游戏运行：未运行；未测网络输入时序、多人争用、碰撞间距手感、雪球成长或宿主存读全过程。未穷举自定义推者与物体物理兼容性。
- 结构检查：`validate_page.py` 通过；两页合并执行 `validate_source_links.py --strict`，99个行号引用、0错误／0警告。本页无新增 Lua 示例；机械检查不代替游戏测试。

相关：[[reference/components/locomotor]] · [[reference/components/physicsmodifiedexternally]]
