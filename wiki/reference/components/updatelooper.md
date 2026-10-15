---
title: "updatelooper：更新回调调度与清理"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "为实体注册普通、长更新、墙钟和后置更新回调，明确首次调用、延迟删除及组件移除边界。"
aliases: ["逐帧更新", "每帧回调", "AddOnUpdateFn", "RemoveOnUpdateFn", "首次调用时机", "延迟删除回调", "长更新回调", "墙钟更新", "post-update", "updatelooper"]
tags: [component, lifecycle, callback, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/updatelooper.lua", "entityscript.lua", "update.lua", "util.lua", "prefabs/alterguardian_lunar_fissures.lua", "components/pumpkincarvable.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# updatelooper：更新回调调度与清理

## 快速判断

- 职责与适用：把实体上的多个函数接入普通、长、墙钟或后置更新；构造只建回调表，普通／墙钟类在首个回调加入时才注册相应调度，不是具名倒计时器。[构造与普通注册](../../../raw/current/scripts/components/updatelooper.lua#L6-L29)、[墙钟注册](../../../raw/current/scripts/components/updatelooper.lua#L84-L89)
- 优先使用：向已有实体组合多个更新函数时用 AddOnUpdateFn，回调参数是 fn(inst, dt)；自定义组件的 OnUpdate(dt) 则由组件实例自行注册更新，见下方比较。[回调执行](../../../raw/current/scripts/components/updatelooper.lua#L53-L68)、[组件调度](../../../raw/current/scripts/update.lua#L255-L268)
- 重要副作用：普通回调倒序执行、不去重；RemoveOnUpdateFn 只是排队，下一次本组件 OnUpdate 开头才删除。同轮回调或移除组件后的当前循环不会因此自动中断。[普通移除](../../../raw/current/scripts/components/updatelooper.lua#L31-L38)、[清队与遍历](../../../raw/current/scripts/components/updatelooper.lua#L53-L68)、[组件移除](../../../raw/current/scripts/entityscript.lua#L670-L685)
- 修改边界：端和网络由宿主决定，组件自身没有统一主从门禁或状态同步。原版裂缝在主模拟添加，南瓜填充图形使用非网络实体；不能将其概括为通用网络定时器。[裂缝服务端接线](../../../raw/current/scripts/prefabs/alterguardian_lunar_fissures.lua#L230-L243)、[非网络图形](../../../raw/current/scripts/components/pumpkincarvable.lua#L129-L139)、[图形挂载](../../../raw/current/scripts/components/pumpkincarvable.lua#L146-L160)

## 关键接口

| 接口 | 参数、顺序与清理 |
|---|---|
| AddOnUpdateFn(fn)／RemoveOnUpdateFn(fn) | fn(inst, dt)，后加入者先执行；添加不立刻调用，删除延迟到下次普通更新开头。最后一个回调删完才 StopUpdatingComponent。[注册](../../../raw/current/scripts/components/updatelooper.lua#L24-L38)、[处理](../../../raw/current/scripts/components/updatelooper.lua#L53-L68) |
| AddLongUpdateFn(fn)／RemoveLongUpdateFn(fn) | fn(inst, dt)，倒序；删除在下次 LongUpdate 开头处理。添加只入表，不启动普通更新或另建计时器；宿主长更新会遍历具有 LongUpdate 的组件。[长更新回调](../../../raw/current/scripts/components/updatelooper.lua#L40-L82)、[宿主入口](../../../raw/current/scripts/entityscript.lua#L2051-L2061) |
| AddOnWallUpdateFn(fn)／RemoveOnWallUpdateFn(fn) | fn(inst, dt)，顺序；首个回调启动墙钟调度，删除在该组件墙钟更新开头处理。服务器暂停时先返回，待删队列也暂不处理。[注册与删除](../../../raw/current/scripts/components/updatelooper.lua#L84-L117)、[墙钟调度](../../../raw/current/scripts/update.lua#L53-L68) |
| AddPostUpdateFn(fn)／RemovePostUpdateFn(fn) | fn(inst)，不传 dt。登记到全局后置表；后置遍历期间删除标 nil，其他时间即时移除并紧凑化，不能套用普通更新的延迟删除规则。[增删](../../../raw/current/scripts/components/updatelooper.lua#L123-L155)、[遍历](../../../raw/current/scripts/components/updatelooper.lua#L157-L190)、[后置入口](../../../raw/current/scripts/update.lua#L142-L147) |

### 与直接定义 OnUpdate 的区别

自定义组件可定义 OnUpdate(dt)，但还需 inst:StartUpdatingComponent(self)；AddComponent 负责构造、postinit 和动作登记，不自动因方法名是 OnUpdate 就启动更新。调度按 cmp:OnUpdate(dt) 调用，组件可访问自身字段。updatelooper 负责在一个组件内管理函数列表，传给每个函数的是宿主 inst 和 dt。[组件添加](../../../raw/current/scripts/entityscript.lua#L632-L668)、[更新注册](../../../raw/current/scripts/entityscript.lua#L439-L473)、[组件执行](../../../raw/current/scripts/update.lua#L255-L268)、[函数执行](../../../raw/current/scripts/components/updatelooper.lua#L65-L68)

仅赋 inst.OnUpdate 不会被上述组件更新循环自动发现；该循环遍历的是已登记的组件。不要把实体字段、组件方法和 fn(inst, dt) 三种调用形式混为一谈。需要具名秒数倒计时、暂停或保存计时状态时，另查 [[reference/components/timer]]。

## 关键流程

### 注册后第一次什么时候调用

AddOnUpdateFn 只做注册和入表，不同步执行函数。实体首次建立 updatecomponents 时进入 NewUpdatingEnts；Update 先遍历现有 UpdatingEnts，再在该阶段末尾合并新实体，因此新入表者要等之后的组件遍历才执行。如果实体已经在更新集合，何时轮到新组件还取决于注册发生的阶段与遍历位置，不能统一承诺“注册当帧”或“必定下一帧”。[回调注册](../../../raw/current/scripts/components/updatelooper.lua#L24-L29)、[实体入表](../../../raw/current/scripts/entityscript.lua#L439-L448)、[遍历与合并](../../../raw/current/scripts/update.lua#L255-L275)

这里的普通更新是模拟 Update 的组件阶段，不等于每个绘制帧或固定长度的 dt；需要按 elapsed time 累积时使用传入 dt，不按调用次数推定秒数。[更新入口与 tick 门槛](../../../raw/current/scripts/update.lua#L224-L242)、[组件调用](../../../raw/current/scripts/update.lua#L255-L268)

### 删除回调与移除整个组件

RemoveOnUpdateFn 要传注册时的同一个函数引用。下一次 OnUpdate 先按待删表逐个调用 table.removearrayvalue，再倒序执行仍在列表中的函数；一次待删项只移除第一个相等项，重复注册不会由一次删除全部清掉。[删除队列](../../../raw/current/scripts/components/updatelooper.lua#L31-L38)、[队列处理](../../../raw/current/scripts/components/updatelooper.lua#L53-L68)、[数组移除](../../../raw/current/scripts/util.lua#L185-L191)

如果在某个回调里请求删除另一个回调，待删队列已在本轮开头处理过，后者本轮仍可能执行。回调 return false 也不会自动注销；只结束自己负责的效果应删除自己的函数，不应删除可能被其他代码共用的整个组件。[普通遍历](../../../raw/current/scripts/components/updatelooper.lua#L53-L68)

确需整体停止时走 inst:RemoveComponent("updatelooper")：宿主停止普通／墙钟更新、移出 components，并调用 OnRemoveFromEntity；本组件还清除 _PostUpdates 登记。但 StopUpdatingComponent 用延迟停止表处理，且已进入的 updatelooper 函数循环不再次查组件是否存在，移除并不会中断当前轮剩余回调。剩余回调应能处理其依赖状态已被撤销的情形。[宿主移除](../../../raw/current/scripts/entityscript.lua#L670-L685)、[延迟停止](../../../raw/current/scripts/entityscript.lua#L475-L502)、[调度停止检查](../../../raw/current/scripts/update.lua#L255-L281)、[组件清理](../../../raw/current/scripts/components/updatelooper.lua#L14-L22)

## 典型接入

### 裂缝淡入结束后只撤销自己的函数

原版 alterguardian_lunar_fissures 在主模拟添加组件并登记 UpdateFadeIn；回调以动画时间或 dt 推进透明度，达到结束条件调用 EndFadeIn，后者用同一函数引用 RemoveOnUpdateFn，再固定最终透明度，不删除整个 updatelooper。该模式适合一个效果完成后撤销本方更新，仍保留其他注册者。[宿主挂载](../../../raw/current/scripts/prefabs/alterguardian_lunar_fissures.lua#L230-L243)、[淡入与撤销](../../../raw/current/scripts/prefabs/alterguardian_lunar_fissures.lua#L76-L98)

这是原版接线说明，不替代完整特效资源、网络状态或其他阶段逻辑；移除仍遵循上方下一次更新清队的规则。

### 需要注册时立即初始化怎么办

pumpkincarvable 的非网络填充图形先添加 updatelooper，夜间条件下登记 Fill_OnUpdate；instant 分支另外显式调用 Fill_OnUpdate(inst, 0)，结束时从函数内排队移除自己。它把“注册后续更新”和“现在初始化表现”分开处理，没有依赖注册立即触发。[图形配置](../../../raw/current/scripts/components/pumpkincarvable.lua#L129-L160)、[条件登记与显式调用](../../../raw/current/scripts/components/pumpkincarvable.lua#L88-L120)、[效果结束撤销](../../../raw/current/scripts/components/pumpkincarvable.lua#L43-L70)

自定义初始化须遵守自己的函数参数和状态前提，不能对任意回调一律传 dt=0；本例只说明原版的初始化接线方式。

## 常见陷阱与限制

- **添加组件就自动开始所有更新**：构造只建四张表；普通、墙钟和后置各有注册入口，LongUpdate 也需实际长更新调用。[构造](../../../raw/current/scripts/components/updatelooper.lua#L6-L12)、[接口](../../../raw/current/scripts/components/updatelooper.lua#L24-L51)、[其他类别](../../../raw/current/scripts/components/updatelooper.lua#L84-L128)
- **重复添加同一函数等于更新配置**：普通注册直接 table.insert，不去重；删除用函数值匹配，重新创建外观相同的闭包也不是同一引用。[添加](../../../raw/current/scripts/components/updatelooper.lua#L24-L38)、[删除匹配](../../../raw/current/scripts/util.lua#L185-L191)
- **普通、墙钟与后置更新采用相同顺序／参数／删除时机**：三者循环与移除策略不同；后置回调不接收 dt，墙钟回调在服务器暂停时跳过。按上方接口表选择，不把整个组件称为暂停期间仍会执行的统一循环。
- **保存了组件就保存回调与剩余时间**：组件没有 OnSave／OnLoad，函数和回调表由宿主重建；长更新只是另一条调用入口，不等于持久化计时器。[组件实现](../../../raw/current/scripts/components/updatelooper.lua#L6-L194)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；读取 updatelooper 全部实现，核验 EntityScript 的组件添加／更新注册与停止／移除／长更新、Update 普通集合合并与墙钟／后置入口、数组单次移除，以及裂缝淡入和南瓜填充的必要宿主接线。
- 游戏运行：未运行；首次执行的引擎调用时点、暂停／长更新、回调内交叉增删、重复注册和移除组件的运行表现尚未实测；未审校完整引擎渲染、网络复制或所有消费者。
- 本页使用原版接线案例，无新增 Lua 示例；具体参数／顺序／清理结论由上述实现支撑，不以机械检查代替游戏效果。

相关：[[reference/components/timer]]；[[reference/components/oldager#关键流程]]。
