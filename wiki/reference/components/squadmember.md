---
title: "squadmember：同名小队成员的事件互相登记"
created: 2026-10-26
updated: "2026-10-28"
type: component
summary: "为精英猪人式同名小队维护其他成员实体集合，供宿主查询并协调选敌。"
aliases: [squadmember, 同名小队, 其他队员集合, 精英猪人分配目标]
tags: [component, lifecycle, event, ai, server-client]
dst_build: "steam-buildid:26230496"
sources: [components/squadmember.lua, prefabs/pigelite.lua]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# squadmember：同名小队成员的事件互相登记

## 快速判断

- 职责与适用：每个实体持有队名和 `others[其他实体] = true` 集合，通过同名世界事件互相发现；没有队长、距离、容量、阵型或攻击调度。原版用于精英猪人查询队友。[squadmember.lua:1-24](../../../raw/current/scripts/components/squadmember.lua#L1-L24)、[pigelite.lua:34-47](../../../raw/current/scripts/prefabs/pigelite.lua#L34-L47)
- 优先使用：参考原版在服务端构造后一次加入固定队名，再只读其他成员集合；动态换队和显式退出存在下述清理缺口，不宜直接作为通用安全重置接口。[pigelite.lua:197-223](../../../raw/current/scripts/prefabs/pigelite.lua#L197-L223)、[squadmember.lua:38-59](../../../raw/current/scripts/components/squadmember.lua#L38-L59)
- 重要副作用：入队先通知已有成员，再监听后续入队；已有成员的回调会同时登记双方。其他成员退出或移除时会清理其引用。[squadmember.lua:1-24](../../../raw/current/scripts/components/squadmember.lua#L1-L24)、[squadmember.lua:38-45](../../../raw/current/scripts/components/squadmember.lua#L38-L45)
- 修改边界：宿主负责实际战斗决策；组件没有存读或网络字段。移除组件映射到 `LeaveSquad`，但该函数没有撤销世界入队事件监听。[squadmember.lua:38-73](../../../raw/current/scripts/components/squadmember.lua#L38-L73)

## 关键接口

| 接口 | 参数、返回 | 条件、作用与证据 |
|---|---|---|
| `JoinSquad(squadname)` | 字符串队名，省略变为 `""`；无显式返回 | 名称不同才先 `LeaveSquad`、赋新名、向 `TheWorld` 发 `ms_joinsquad_<队名>`（数据为自身实体），再监听同一事件；同名调用不重建集合。[squadmember.lua:38-46](../../../raw/current/scripts/components/squadmember.lua#L38-L46) |
| `GetOtherMembers()` | 原始实体键集合 | 直接返回内部 `others`，不是数组或副本；正常组装下不含自己。调用方遍历读取，不应自行增删绕过监听维护。[squadmember.lua:1-24](../../../raw/current/scripts/components/squadmember.lua#L1-L24)、[squadmember.lua:34-36](../../../raw/current/scripts/components/squadmember.lua#L34-L36) |
| `GetSquadName()` | 初始 `""`、入队字符串，退出后为 `nil` | 直接返回字段；退出后的空值应由调用方处理。[squadmember.lua:13-16](../../../raw/current/scripts/components/squadmember.lua#L13-L16)、[squadmember.lua:30-32](../../../raw/current/scripts/components/squadmember.lua#L30-L32)、[squadmember.lua:55-57](../../../raw/current/scripts/components/squadmember.lua#L55-L57) |
| `IsInSquad()` | 对当前名称执行 `:len() > 0` | 初始返回假，非空入队返回真；退出后的 `nil` 会使它及调用它的 `GetDebugString()` 报错。[squadmember.lua:26-28](../../../raw/current/scripts/components/squadmember.lua#L26-L28)、[squadmember.lua:55-68](../../../raw/current/scripts/components/squadmember.lua#L55-L68) |
| `LeaveSquad()` | 无参数／无显式返回 | 遍历清理当前 `others` 与其监听，把自身队名置 `nil`，在自身实体上发 `ms_leavesquad_<旧名>`；没有移除 `TheWorld` 上的 join 监听。[squadmember.lua:7-11](../../../raw/current/scripts/components/squadmember.lua#L7-L11)、[squadmember.lua:48-59](../../../raw/current/scripts/components/squadmember.lua#L48-L59) |

## 关键流程

### 成员互相登记与引用清理

第一个成员入队时先发世界事件再监听；第二个成员入队时，第一个收到事件，在自身集合登记新成员，再通过 `other.components.squadmember` 反向登记自己。无需中央成员表，前提是事件数据实体具有该组件。[squadmember.lua:13-24](../../../raw/current/scripts/components/squadmember.lua#L13-L24)、[squadmember.lua:38-46](../../../raw/current/scripts/components/squadmember.lua#L38-L46)

`TrackOther` 在自己实体上监听对方实体的专属退队事件和 `onremove`；触发后 `StopTrackingOther` 清除表键及这两种监听。`LeaveSquad` 清自己的引用并通知其他成员清除自己；组件移除时也调用它。这里没有自动分组存档，宿主应在重建时重新加入所需队名；正常发现依赖已有同名成员仍在监听。[squadmember.lua:1-11](../../../raw/current/scripts/components/squadmember.lua#L1-L11)、[squadmember.lua:48-73](../../../raw/current/scripts/components/squadmember.lua#L48-L73)

## 典型接入

### 精英猪人：成员集合辅助分配战斗目标

`MakePigElite(variation)` 注册 `pigelite1` 至 `pigelite4`；服务端添加 `squadmember` 后统一 `JoinSquad("pigkingelite4")`。四个变体使用同一个固定队名，不按变体拆队，也没有按猪王实例生成独立队名。[pigelite.lua:152-153](../../../raw/current/scripts/prefabs/pigelite.lua#L152-L153)、[pigelite.lua:197-223](../../../raw/current/scripts/prefabs/pigelite.lua#L197-L223)、[pigelite.lua:268-275](../../../raw/current/scripts/prefabs/pigelite.lua#L268-L275)

宿主 `IsSquadAlreadyTargeting` 遍历其他成员，检查其是否持有手持装备且正在攻击该目标、距离是否更近，并排除击退、着火受伤、恐慌、睡眠及冻结的队友。`RetargetFn` 和 `KeepTargetFn` 使用结果避免继续争抢已有合适队友负责的目标；两者注册在 `combat` 上，方法也赋给宿主实体。小队组件仅提供引用集合，不自行分配攻击目标。[pigelite.lua:30-96](../../../raw/current/scripts/prefabs/pigelite.lua#L30-L96)、[pigelite.lua:211-223](../../../raw/current/scripts/prefabs/pigelite.lua#L211-L223)、[pigelite.lua:247](../../../raw/current/scripts/prefabs/pigelite.lua#L247

## 常见陷阱与限制

- **退队不等于完整清理**：`LeaveSquad` 没有撤销 `ms_joinsquad_<旧名>` 的世界监听。显式退出但保留实体时，旧队后续有人入队，回调仍可能执行；此时 `TrackOther` 会拼接已为 `nil` 的队名而报错。换队后旧队入队事件也可能把不同队成员串入集合。上述为源码控制流推导，未运行复现。[squadmember.lua:1-4](../../../raw/current/scripts/components/squadmember.lua#L1-L4)、[squadmember.lua:17-20](../../../raw/current/scripts/components/squadmember.lua#L17-L20)、[squadmember.lua:38-59](../../../raw/current/scripts/components/squadmember.lua#L38-L59)
- **空名不是统一退出协议**：刚构造时 `JoinSquad()` 与初始空名相同而无操作；从非空队名调用它则会加入事件后缀为空的组，但 `IsInSquad()` 返回假。需要动态组队时应先设计明确的监听撤销与状态约束，不把空名当作安全离队替代。[squadmember.lua:13-16](../../../raw/current/scripts/components/squadmember.lua#L13-L16)、[squadmember.lua:26-46](../../../raw/current/scripts/components/squadmember.lua#L26-L46)

## 验证记录

- 源码审阅：2026-10-26，Steam buildid 26148036；覆盖组件全部实现、双向引用与监听对象、移除映射及存读缺省；精英猪人的固定队名注册、成员查询与战斗回调。简单档只覆盖同名引用集合与一个宿主消费者。
- 游戏运行：未运行；未实测换组、退队后事件／调试报错、组件移除后旧监听与实际战斗分配；未审阅宿主其他 AI／状态图。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 通过；无新增 Lua 示例，未运行语法探针。

- 版本沿用：2026-10-28 批量沿用-70，见 [[updates/build-26230496/index#wiki-沿用记录]]。
相关：[[reference/components/combat]]。
