---
title: "butterflyspawner"
created: 2026-10-26
updated: "2026-10-28"
type: component
summary: "按白天非冬季与玩家附近花朵、局部密度调度自然蝴蝶，并管理野外蝴蝶的休眠移除与库存追踪切换。"
aliases: ["butterflyspawner", "蝴蝶自然生成", "花旁不刷蝴蝶", "蝴蝶休眠消失"]
tags: [component, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/butterflyspawner.lua", "prefabs/forest.lua", "prefabs/world.lua", "prefabs/butterfly.lua", "tuning.lua", "worldsettings_overrides.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# butterflyspawner

## 快速判断

- 职责与适用：森林世界主模拟的自然蝴蝶调度器，按活跃玩家安排任务；不是花朵上的产物组件。构造断言服务端，森林 `master_postinit` 安装。[butterflyspawner.lua:5-22](../../../raw/current/scripts/components/butterflyspawner.lua#L5-L22) [forest.lua:639-642](../../../raw/current/scripts/prefabs/forest.lua#L639-L642)
- 优先使用：排查“花旁不刷蝴蝶”时，依次检查白天／非冬季、局部数量、候选花距离；管理可捕捉蝴蝶时沿原版 `StartTracking`／`StopTracking` 配对协议。[调度门槛 butterflyspawner.lua:80-99](../../../raw/current/scripts/components/butterflyspawner.lua#L80-L99) [本体接线 butterfly.lua:183-190](../../../raw/current/scripts/prefabs/butterfly.lua#L183-L190)
- 重要副作用：追踪会设置 `persists=false`，必要时补 `homeseeker`；被追踪实体进入休眠后延后一任务确认并删除。停止追踪恢复原持久化和家园组件有无，而非删除实体。[butterflyspawner.lua:101-113](../../../raw/current/scripts/components/butterflyspawner.lua#L101-L113) [butterflyspawner.lua:180-212](../../../raw/current/scripts/components/butterflyspawner.lua#L180-L212)
- 修改边界：组件没有公开“立即刷一只”或有效的模式 setter；自然数量来自构造时缓存的 TUNING，不能把后来直接生成的蝴蝶等同于通过自然门槛。[butterflyspawner.lua:21-22](../../../raw/current/scripts/components/butterflyspawner.lua#L21-L22) [butterflyspawner.lua:164-195](../../../raw/current/scripts/components/butterflyspawner.lua#L164-L195)

## 关键接口

| 接口／字段 | 签名或约束 | 条件、作用与证据 |
|---|---|---|
| `OnPostInit()` | 无参数；世界生命周期入口 | 调用内部 `ToggleUpdate(true)`，满足生成条件时建立或重排玩家任务；世界 `PostInit` 逐组件调用此钩子。[butterflyspawner.lua:156-158](../../../raw/current/scripts/components/butterflyspawner.lua#L156-L158) [world.lua:335-346](../../../raw/current/scripts/prefabs/world.lua#L335-L346) |
| `StartTracking(target)`／`StartTrackingFn(target)` | 后者为单参数回调形式，不带 self | 幂等登记；保存原 `persists` 和是否已有 `homeseeker`，开始监听目标 `entitysleep`。目标须具备实体及组件接口。[butterflyspawner.lua:180-196](../../../raw/current/scripts/components/butterflyspawner.lua#L180-L196) |
| `StopTracking(target)`／`StopTrackingFn(target)` | 后者可直接作事件／库存回调 | 只处理已登记目标，恢复保存的状态、移除本组件补建的 `homeseeker` 并撤监听；不回滚原有家园组件的内部数据。[butterflyspawner.lua:198-212](../../../raw/current/scripts/components/butterflyspawner.lua#L198-L212) |
| `SpawnModeNever/Light/Med/Heavy()` | 全部为 deprecated 空实现 | 不停止任务、不修改数量；原版世界设置改 TUNING，而非调用这些兼容方法。[butterflyspawner.lua:164-178](../../../raw/current/scripts/components/butterflyspawner.lua#L164-L178) [worldsettings_overrides.lua:1311-1333](../../../raw/current/scripts/worldsettings_overrides.lua#L1311-L1333) |
| `GetDebugString()` | 返回更新状态与追踪数／上限的字符串 | 分子来自 `_butterflies` 表；自然生成门槛用玩家附近标签查询，二者不是同一计数。[butterflyspawner.lua:218-224](../../../raw/current/scripts/components/butterflyspawner.lua#L218-L224) [butterflyspawner.lua:47-50](../../../raw/current/scripts/components/butterflyspawner.lua#L47-L50) |

## 关键流程

### 白天、季节与玩家任务

构造从 `AllPlayers` 初始化活跃表，监听世界 `ms_playerjoined`／`ms_playerleft`，以及 `isday`、`iswinter` 状态。只有白天、非冬季且缓存上限大于 0 时更新；不满足时取消所有活跃玩家的生成任务。加入玩家只在正在更新时排任务，离开取消其任务并移出表。[butterflyspawner.lua:80-99](../../../raw/current/scripts/components/butterflyspawner.lua#L80-L99) [butterflyspawner.lua:115-150](../../../raw/current/scripts/components/butterflyspawner.lua#L115-L150)

开始更新／初次加入的延迟为 `0.3 + random()*10` 秒，后续为 `10 + random()*10` 秒；每次尝试后清任务记录并续排，不保证每次生成。它不是按天结算或离线补算的计时器。[butterflyspawner.lua:47-78](../../../raw/current/scripts/components/butterflyspawner.lua#L47-L78)

### 局部数量与落花点

每次以玩家位置为中心查询半径 64 内的 `butterfly` 标签实体，只有数量小于缓存上限才找花；默认 `MAX_BUTTERFLIES=4`，不是全世界最多 4 只，也不是仅统计本组件生成的蝴蝶。世界蝴蝶设置把上限改为 never=0、rare=2、often=7、always=10；default 保留 tuning 默认值。[butterflyspawner.lua:47-50](../../../raw/current/scripts/components/butterflyspawner.lua#L47-L50) [tuning.lua:6017](../../../raw/current/scripts/tuning.lua#L6017 [worldsettings_overrides.lua:1311-1333](../../../raw/current/scripts/worldsettings_overrides.lua#L1311-L1333)

候选来自玩家半径 25 内的 `flower` 标签实体，剔除与玩家距离平方不大于 36 的花，即距离必须大于 6；剩余花中随机取一朵。成功后生成 `butterfly`，有 `pollinator` 时先 `Pollinate`，再设家园为该花，最后传送到花的位置。这里不另查花朵种类或全世界花总数；家园和授粉细节见 [[reference/components/homeseeker]]、[[reference/components/pollinator]]。[butterflyspawner.lua:28-64](../../../raw/current/scripts/components/butterflyspawner.lua#L28-L64)

## 典型接入

### 沿原版蝴蝶的库存与野外切换协议

原版 `butterfly` 服务端工厂取 `TheWorld.components.butterflyspawner`；存在时把单参数 `StopTrackingFn` 接到放入库存和 `onremove`，再开始追踪。捕网收取和小型牲畜拾取回调也停止追踪，掉落回调重新追踪；所以“存在世界组件”不代表库存蝴蝶也始终被追踪。[butterfly.lua:17-52](../../../raw/current/scripts/prefabs/butterfly.lua#L17-L52) [butterfly.lua:183-190](../../../raw/current/scripts/prefabs/butterfly.lua#L183-L190)

这是世界内部组件的原版接入参照。模组若复用此协议，应在服务端、世界组件存在且目标实现所需接口时接线，按相同捕获／入库存／移除边界成对切换；不要仅给花加该组件，也不要把 `StartTracking` 当成生成实体的接口。[构造端约束 butterflyspawner.lua:5-22](../../../raw/current/scripts/components/butterflyspawner.lua#L5-L22) [追踪实现 butterflyspawner.lua:180-212](../../../raw/current/scripts/components/butterflyspawner.lua#L180-L212)

## 常见陷阱与限制

- **花越靠近玩家越能刷：** 6 距离以内的候选被剔除；需要在玩家 6～25 距离范围内有符合标签的花，并满足局部数量和时间条件。[butterflyspawner.lua:31-64](../../../raw/current/scripts/components/butterflyspawner.lua#L31-L64)
- **改全局 TUNING 立即生效：** 上限在构造时复制到局部变量，现有实例无公开 setter。需要改变原版配置时，应处理构造前的配置入口，并核对世界设置覆盖；本页不提供运行时改私有局部值的方案。[butterflyspawner.lua:17-22](../../../raw/current/scripts/components/butterflyspawner.lua#L17-L22) [worldsettings_overrides.lua:1311-1333](../../../raw/current/scripts/worldsettings_overrides.lua#L1311-L1333)
- **自然关闭等于禁止任何蝴蝶实体出现：** 时间与密度门槛位于自然调度路径，蝴蝶本体工厂的追踪接线不检查这些门槛；直接生成或掉落的追踪行为应单独判断。[butterflyspawner.lua:47-99](../../../raw/current/scripts/components/butterflyspawner.lua#L47-L99) [butterfly.lua:17-21](../../../raw/current/scripts/prefabs/butterfly.lua#L17-L21) [butterfly.lua:183-188](../../../raw/current/scripts/prefabs/butterfly.lua#L183-L188)
- **追踪表会存档并续接原倒计时：** 本组件没有 `OnSave`／`OnLoad`；玩家任务和追踪表是会话状态，已追踪蝴蝶还被设为不持久化。组件也没有 `OnRemoveFromEntity` 清理钩子，自然任务挂在玩家上，不能用运行时直接移除组件代替完整停用协议。[任务归属 butterflyspawner.lua:66-78](../../../raw/current/scripts/components/butterflyspawner.lua#L66-L78) [初始化 butterflyspawner.lua:141-158](../../../raw/current/scripts/components/butterflyspawner.lua#L141-L158) [追踪 butterflyspawner.lua:180-212](../../../raw/current/scripts/components/butterflyspawner.lua#L180-L212)

## 验证记录

- 2026-10-26，`steam-buildid:26148036`：审阅组件全文、森林服务端安装、世界 `OnPostInit` 调用、蝴蝶追踪消费者及所用 tuning／世界设置分支，覆盖自然门槛、排程、局部数量、落花点、追踪恢复及无组件存读钩子的边界。
- 游戏运行：未运行；未验证多人交叠范围的实际生成分布、重载期间任务时序或模组替换花／蝴蝶的兼容性。授粉和家园组件内部行为复用对应页，不在本页扩大核验范围。
- 结构检查：`validate_page.py` 通过；两页合并执行 `validate_source_links.py --strict`，83 个行号引用、0 错误／0 警告。本页无新增 Lua 示例；机械检查不代替游戏测试。

相关：[[reference/components/birdspawner]] · [[reference/components/lureplantspawner]]
