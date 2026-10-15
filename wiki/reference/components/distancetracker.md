---
title: "distancetracker"
created: "2026-10-26"
updated: "2026-10-28"
type: component
summary: "说明玩家位置采样占位组件：每次更新只计算并丢弃相邻样本的三维位移，不累计里程或输出统计。"
aliases: ["玩家距离采样", "移动距离没有累计", "distancetracker"]
tags: [component, lifecycle, data]
dst_build: "steam-buildid:26230496"
sources: ["components/distancetracker.lua", "prefabs/player_common.lua", "entityscript.lua", "update.lua", "util.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# distancetracker

## 快速判断

- 职责与适用：每次组件更新采样宿主世界位置，并计算相邻两点的直线距离；现版本只保留 previous_pos，distance 是局部变量，**不能直接当作可用里程计**。[distancetracker.lua:1](../../../raw/current/scripts/components/distancetracker.lua#L1
- 优先使用：排查已有玩家组件时识别其占位边界；没有公开读取距离、累计、重置或完成阈值回调接口，不能仅添加组件便得到行走统计。[distancetracker.lua:7](../../../raw/current/scripts/components/distancetracker.lua#L7
- 重要副作用：构造立即注册组件更新；每次 OnUpdate 都用新位置覆盖 previous_pos。首次没有旧样本，局部距离保持 0。[distancetracker.lua:1](../../../raw/current/scripts/components/distancetracker.lua#L1
- 修改边界：原版在 player_common 主模拟构造中安装；统计发送语句及单位转换均已注释，不发送 furlongs_travelled，不提供网络同步字段。[player_common.lua:2613](../../../raw/current/scripts/prefabs/player_common.lua#L2613 [player_common.lua:2780](../../../raw/current/scripts/prefabs/player_common.lua#L2780 [distancetracker.lua:15](../../../raw/current/scripts/components/distancetracker.lua#L15

## 关键接口

| 接口／字段 | 参数或返回 | 条件、作用与证据 |
|---|---|---|
| `previous_pos` | 初始 nil，随后为 Point | 保存上一次更新读取的世界坐标；不是起点、总路程或最后一段长度。[distancetracker.lua:1](../../../raw/current/scripts/components/distancetracker.lua#L1 [distancetracker.lua:7](../../../raw/current/scripts/components/distancetracker.lua#L7 |
| `OnUpdate(dt)` | 更新框架传入 dt；无显式返回 | 读取当前 Point，以 sqrt(distsq) 算单次位移并覆盖旧位置；dt 未参与计算，故没有速度或按秒归一化结果。[distancetracker.lua:7](../../../raw/current/scripts/components/distancetracker.lua#L7 |
| 局部 `distance` | 每次从 0 开始 | 有旧样本才计算；既未写入组件字段，也未累计、发事件或作为返回值输出。注释中的 SendGameStat 不执行。[distancetracker.lua:7](../../../raw/current/scripts/components/distancetracker.lua#L7 |

## 关键流程

两点形式的 distsq 实际计算 x、y、z 三个分量的平方和；因此这里得到的是**三维世界坐标距离单位**，不是仅 x/z 水平距离、地皮数或沿寻路路径走过的长度。注释虽写有米到 furlong 的换算，当前没有执行任何单位转换。[util.lua:555](../../../raw/current/scripts/util.lua#L555 [distancetracker.lua:7](../../../raw/current/scripts/components/distancetracker.lua#L7

算法也不区分行走、被推动或传送：只比较连续样本；同一对样本之间绕行的细节不会被还原。不过这些差值最终都没有保留，不能据此认定玩家统计系统已累计了传送距离。[distancetracker.lua:7](../../../raw/current/scripts/components/distancetracker.lua#L7

构造调用 StartUpdatingComponent，宿主登记到更新集合，普通更新循环再调 OnUpdate(dt)。组件没有自己的定时间隔或自动结束条件；移除组件时 EntityScript:RemoveComponent 先停止其更新。单纯停止更新不会由本组件清 previous_pos，重新创建组件则从 nil 开始。[entityscript.lua:439](../../../raw/current/scripts/entityscript.lua#L439 [update.lua:255](../../../raw/current/scripts/update.lua#L255 [entityscript.lua:670](../../../raw/current/scripts/entityscript.lua#L670 [distancetracker.lua:1](../../../raw/current/scripts/components/distancetracker.lua#L1

## 典型接入

### 原版玩家：存在组件不等于存在里程输出

MakePlayerCharacter 的构造路径在客户端提前返回后，直接 AddComponent("distancetracker")，没有给它配置消费者或累计字段。保留原版接线参照即可；本页不提供以该组件读取里程的 demo，因为原实现没有相应数据接口。[player_common.lua:1968](../../../raw/current/scripts/prefabs/player_common.lua#L1968 [player_common.lua:2613](../../../raw/current/scripts/prefabs/player_common.lua#L2613 [player_common.lua:2780](../../../raw/current/scripts/prefabs/player_common.lua#L2780

## 常见陷阱与限制

- **尝试读取 components.distancetracker.distance**：原实现只定义函数局部变量，该字段不会由组件写入。必须先明确所需统计的真实实现，不能把注释中的遥测名称当成可查询接口。[distancetracker.lua:7](../../../raw/current/scripts/components/distancetracker.lua#L7
- **假定读档或离线时间继续累计**：没有 OnSave／OnLoad／LongUpdate，previous_pos 不由该组件保存；重新构造后首个样本无差值，离线期间也没有此组件的补算逻辑。[distancetracker.lua:1](../../../raw/current/scripts/components/distancetracker.lua#L1

## 验证记录

- 源码审阅：2026-10-26，Steam buildid 26148036；覆盖组件全文、玩家主模拟安装、组件更新注册／调用／移除与 distsq 两点分支。未审阅整个玩家成就／遥测系统，不将本组件无输出推广为游戏没有其他移动统计。
- 结构检查：validate_page 及源码引用 strict 检查通过；无新增完整 Lua 示例。机械检查不代替游戏运行。
- 游戏运行：未运行；更新时序、抬升／传送采样、组件停启与重建待实测。

相关：移动速度与目的地控制见 [[reference/components/locomotor]]。
