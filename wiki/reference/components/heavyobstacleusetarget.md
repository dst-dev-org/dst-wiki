---
title: "heavyobstacleusetarget"
created: "2026-10-26"
updated: "2026-10-28"
type: component
summary: "给目标提供使用当前 BODY 重物的动作回调，区分可用标签、动作过滤和宿主实际转移物品的职责。"
aliases: ["搬运重物放入目标", "健身房装载重物", "USE_HEAVY_OBSTACLE", "heavyobstacleusetarget"]
tags: [component, callback, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/heavyobstacleusetarget.lua", "componentactions.lua", "actions.lua", "stategraphs/SGwilson.lua", "stategraphs/SGwilson_client.lua", "prefabs/mighty_gym.lua", "components/mightygym.lua", "class.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# heavyobstacleusetarget

## 快速判断

- 职责与适用：挂在**接收重物的目标**，将操作者及重物转交 on_use_fn；不负责搬动物体的碰撞、重量计算、库存转移或销毁。[heavyobstacleusetarget.lua:13](../../../raw/current/scripts/components/heavyobstacleusetarget.lua#L13
- 优先使用：配置 on_use_fn 实现目标实际效果；赋 can_use_heavy 控制标签，需要按物品／操作者筛选时配宿主 use_heavy_obstacle_action_filter。[heavyobstacleusetarget.lua:1](../../../raw/current/scripts/components/heavyobstacleusetarget.lua#L1 [componentactions.lua:416](../../../raw/current/scripts/componentactions.lua#L416
- 重要副作用：默认 can_use_heavy=true 会加同名标签，但未设置回调时实际使用仍返回 false。直接 UseHeavyObstacle **不检查该开关或过滤器**。[heavyobstacleusetarget.lua:13](../../../raw/current/scripts/components/heavyobstacleusetarget.lua#L13 [heavyobstacleusetarget.lua:23](../../../raw/current/scripts/components/heavyobstacleusetarget.lua#L23
- 修改边界：候选动作通过 replica.inventory／标签判断，目标服务端组件最终执行回调；原版参照是 mighty_gym 的 OnUseHeavy，不能仅凭 heavy 标签假定所有物体都满足目标需求。[componentactions.lua:416](../../../raw/current/scripts/componentactions.lua#L416 [mighty_gym.lua:102](../../../raw/current/scripts/prefabs/mighty_gym.lua#L102

## 关键接口

| 接口／字段 | 参数或返回 | 条件、作用与证据 |
|---|---|---|
| `can_use_heavy` | 默认 true | Class 属性赋值触发回调，真值添加 can_use_heavy 标签，假值移除；不自动更改 on_use_fn。[heavyobstacleusetarget.lua:1](../../../raw/current/scripts/components/heavyobstacleusetarget.lua#L1 [class.lua:36](../../../raw/current/scripts/class.lua#L36 |
| `on_use_fn` | `fn(target, doer, heavy_obstacle)` | 注释中的 nil 不是实际赋值；由宿主配置。组件不验证参数、不移除／消耗物品；返回值为回调首个返回值，nil／false 归为 false，缺回调也返回 false。[heavyobstacleusetarget.lua:13](../../../raw/current/scripts/components/heavyobstacleusetarget.lua#L13 |
| `UseHeavyObstacle(doer, heavy_obstacle)` | 回调结果或 false | 只调用回调；can_use_heavy 及动作过滤应由调用路径另查，直接调用可绕过它们。[heavyobstacleusetarget.lua:23](../../../raw/current/scripts/components/heavyobstacleusetarget.lua#L23 |
| 宿主 `use_heavy_obstacle_action_filter` | `fn(target, doer, item)` | 可选，候选与执行动作均调用，必须满足各自执行环境；不是本组件自动调用的资格函数。[componentactions.lua:416](../../../raw/current/scripts/componentactions.lua#L416 [actions.lua:5566](../../../raw/current/scripts/actions.lua#L5566 |
| 宿主 `use_heavy_obstacle_string_key` | 动作字符串 key | strfn 从目标取该字段；仅影响文案选择，不实现资格校验或使用效果。[actions.lua:5562](../../../raw/current/scripts/actions.lua#L5562 |

## 关键流程

候选阶段要求右键、操作者 BODY 槽有物品且物品带 heavy、目标带 can_use_heavy，并通过可选 filter，才插入 USE_HEAVY_OBSTACLE。动作定义为 encumbered_valid=true、rmb=true、priority=1；它使用的是当前装备物，不是任意 invobject 或背包里的重物。[componentactions.lua:416](../../../raw/current/scripts/componentactions.lua#L416 [actions.lua:582](../../../raw/current/scripts/actions.lua#L582

服务端及客户端 SG 都映射到 dolongaction；客户端执行预览，服务端状态超时才 PerformBufferedAction。实际动作重新读取当前 BODY 物品，再查目标标签／filter，最后找目标组件调用 UseHeavyObstacle；这里没有再次检查物品 heavy 标签，因此需要严格物品契约时应在 filter／最终回调落实。[SGwilson.lua:1651](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1651 [SGwilson.lua:8348](../../../raw/current/scripts/stategraphs/SGwilson.lua#L8348 [SGwilson_client.lua:895](../../../raw/current/scripts/stategraphs/SGwilson_client.lua#L895 [SGwilson_client.lua:3185](../../../raw/current/scripts/stategraphs/SGwilson_client.lua#L3185 [actions.lua:5566](../../../raw/current/scripts/actions.lua#L5566

## 典型接入

### 原版健身房：回调转移重物，目标组件装载

`mighty_gym` 设置 LOAD_GYM 文案，在服务端安装 heavyobstacleusetarget，on_use_fn=OnUseHeavy。该回调检查物品非 nil，从操作者 inventory 移除物品，再调用 mightygym:LoadWeight，最后返回 true；**物品转移是回调行为，不是 UseHeavyObstacle 自带效果**。[mighty_gym.lua:50](../../../raw/current/scripts/prefabs/mighty_gym.lua#L50 [mighty_gym.lua:85](../../../raw/current/scripts/prefabs/mighty_gym.lua#L85 [mighty_gym.lua:118](../../../raw/current/scripts/prefabs/mighty_gym.lua#L118 [mighty_gym.lua:136](../../../raw/current/scripts/prefabs/mighty_gym.lua#L136

LoadWeight 负责放入健身房库存；满槽且未指定槽位时先丢出轮替槽旧物，再装新物，更新显示／重量。烧毁回调移除目标组件并 UnloadWeight。容量、交换、重量及清理都由宿主／mightygym 承担，不应扩写成本组件的通用行为。[mightygym.lua:159](../../../raw/current/scripts/components/mightygym.lua#L159 [mightygym.lua:210](../../../raw/current/scripts/components/mightygym.lua#L210 [mighty_gym.lua:45](../../../raw/current/scripts/prefabs/mighty_gym.lua#L45

## 常见陷阱与限制

- **关闭 can_use_heavy 后直接调用方法也被拒绝**：开关只更新标签，方法无门禁；实际效果回调需要自己的有效性约束。候选通过也不代表长动作完成时仍持有同一物品。[heavyobstacleusetarget.lua:23](../../../raw/current/scripts/components/heavyobstacleusetarget.lua#L23 [actions.lua:5566](../../../raw/current/scripts/actions.lua#L5566
- **组件自行保存开关／已用重物或移除时清标签**：没有存读、更新或 OnRemoveFromEntity 钩子；重建时默认开启，配置由宿主重施。直接移组件不执行 can_use_heavy=false 的属性回调，不能当作该赋值的替代。[heavyobstacleusetarget.lua:1](../../../raw/current/scripts/components/heavyobstacleusetarget.lua#L1

## 验证记录

- 源码审阅：2026-10-26，Steam buildid 26148036；覆盖组件全文、属性回调、候选／权威动作及两端 SG、健身房回调与装卸重物。未展开整个 mightygym 锻炼／库存存档系统或其他角色 SG。
- 结构检查：validate_page 及源码引用 strict 检查通过；无新增完整 Lua 示例。机械检查不代替游戏运行。
- 游戏运行：未运行；动作中途换物、目标开关／过滤器、重复调用、满槽交换与烧毁待实测。

相关：搬运物自身的碰撞状态见 [[reference/components/heavyobstaclephysics]]；普通装备槽行为见 [[reference/components/equippable]]。
