---
title: "powerload"
created: "2026-10-23"
updated: "2026-10-28"
type: component
summary: "向薇诺娜工程电源报告负载与待机标志，供宿主结算耗能；不负责储电或供电。"
aliases: ["薇诺娜机器功率", "工程设备待机耗电", "修改负载不生效", "powerload"]
tags: [component, energy, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/powerload.lua", "prefabs/winona_battery_low.lua", "prefabs/winona_battery_high.lua", "prefabs/winona_spotlight.lua", "prefabs/winona_catapult.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# powerload

## 快速判断

- 职责与适用：仅给设备保存 load 与 isidle，默认 1／false；薇诺娜低／高电源遍历 circuitnode 邻居读取并计算 fueled.rate。它本身不生成电量或工作效果。[组件](../../../raw/current/scripts/components/powerload.lua#L1-L17) [低电源消费者](../../../raw/current/scripts/prefabs/winona_battery_low.lua#L111-L140) [高电源消费者](../../../raw/current/scripts/prefabs/winona_battery_high.lua#L350-L379)
- 优先使用：设备状态变化时 `SetLoad(number, idle?)`；拓扑与广播复用 [[reference/components/circuitnode]]，燃料扣量复用 [[reference/components/fueled#关键接口]]。机器开关 [[reference/components/machine]] 是另一层约定，不能自动驱动本组件。
- 重要副作用：SetLoad 只赋值，**不通知电源重算**；原聚光灯设置负载后调用宿主 OnCircuitChanged，把 engineeringcircuitchanged 发给直接相邻节点。[赋值](../../../raw/current/scripts/components/powerload.lua#L7-L9) [广播](../../../raw/current/scripts/prefabs/winona_spotlight.lua#L552-L558) [状态切换](../../../raw/current/scripts/prefabs/winona_spotlight.lua#L575-L593)
- 修改边界：已核验聚光灯在服务端安装；本组件无网络、定时更新、存档或清理逻辑。设备恢复工作状态由工厂／宿主负责，不能把负载视为剩余电量。[端门禁](../../../raw/current/scripts/prefabs/winona_spotlight.lua#L1100-L1107) [安装](../../../raw/current/scripts/prefabs/winona_spotlight.lua#L1144-L1148) [完整定义](../../../raw/current/scripts/components/powerload.lua#L1-L20)

## 关键接口

| 接口／字段 | 参数、读取规则与证据 |
|---|---|
| `SetLoad(_load, idle?)` | 直接保留 _load，不校验／夹取；idle 仅在 `idle == true` 时为 true，省略或传 1 都变 false。对已核验电源应提供数值，nil／字符串不能作为安全禁用值。[设置](../../../raw/current/scripts/components/powerload.lua#L7-L9) [数值消费者](../../../raw/current/scripts/prefabs/winona_battery_low.lua#L120-L138) |
| `GetLoad()`／`load` | 直接返回当前值；默认 1。不是焦耳、瓦特或持续秒数，最终通过电源公式成为燃料消耗率的输入。[默认／读取](../../../raw/current/scripts/components/powerload.lua#L1-L13) [结算](../../../raw/current/scripts/prefabs/winona_battery_low.lua#L118-L138) |
| `IsIdle()`／`isidle` | 只返回标志，不检查动画、目标、机器开关或供电任务。电源仅在自己的 _noidledrain 生效时跳过 idle 设备，不是“true 永不耗电”。[读取](../../../raw/current/scripts/components/powerload.lua#L16-L17) [门禁](../../../raw/current/scripts/prefabs/winona_battery_low.lua#L121-L138) |

### 电源怎样消费负载

低／高电源逻辑一致：仅在自身 fueled.consuming 时遍历连接设备；没有 powerload 的节点按负载 1，有组件则先检查免待机条件，再读取负载，`<=0` 跳过该设备。有效负载除以该设备相邻且正在消耗燃料的电源数后累加，避免每个供电源各算一份完整负载。[低电源](../../../raw/current/scripts/prefabs/winona_battery_low.lua#L116-L138) [高电源](../../../raw/current/scripts/prefabs/winona_battery_high.lua#L355-L377)

最终 rate 在免待机模式为 total_load，否则为 `max(total_load, WINONA_BATTERY_MIN_LOAD)`；所以所有设备负载为 0 也不保证电源零消耗。当前最小值 0.1，聚光灯开／关负载为 0.5／0.05；这些数是此快照的调参输入，不是通用物理单位。[公式](../../../raw/current/scripts/prefabs/winona_battery_low.lua#L138-L140) [灯配置](../../../raw/current/scripts/tuning.lua#L3701-L3702) [最低负载](../../../raw/current/scripts/tuning.lua#L3740)

重算入口 OnCircuitChanged 安排零秒任务合并通知，监听对象是电源实体；向设备自身推事件能否转发，取决于该宿主是否装了对应监听。不可假设任意新设备都自带此链路。[电源调度](../../../raw/current/scripts/prefabs/winona_battery_low.lua#L145-L147) [电源监听](../../../raw/current/scripts/prefabs/winona_battery_low.lua#L948-L949) [灯监听](../../../raw/current/scripts/prefabs/winona_spotlight.lua#L1147-L1148)

## 典型接入

### 新工程设备按工作状态报告负载

这是工厂协作指引，不是独立可运行的电器 demo。需要先有服务端实体、可用 [[reference/components/circuitnode]] 连接和宿主实际供电／工作逻辑；不能仅 AddComponent("powerload") 就让实体自动接电。

1. 在服务端工厂安装组件并设置初始负载／idle。聚光灯以关灯负载且 idle=true 初始化，节点连接回调另行处理电路显示与通知。[安装](../../../raw/current/scripts/prefabs/winona_spotlight.lua#L1136-L1148) [连接回调](../../../raw/current/scripts/prefabs/winona_spotlight.lua#L920-L932)
2. 在**实际工作状态**变化的位置 SetLoad，随后通过宿主已实现的通知链让相邻电源重算。聚光灯 EnableLight 在关闭时设 OFF,true，打开时设 ON（清掉 idle）；两条路径均通知电源，单纯改字段不会替你做这一步。[开关负载](../../../raw/current/scripts/prefabs/winona_spotlight.lua#L561-L593) [广播函数](../../../raw/current/scripts/prefabs/winona_spotlight.lua#L552-L558)
3. 把供电与负载分开：电源周期调用邻居 `AddBatteryPower(duration)`，聚光灯把它转到 SetPowered，维护有时限的 _powertask；这段时限、找目标和灯光不在 powerload 中。需要新行为时扩展宿主，不给本组件编造供电方法。[供电](../../../raw/current/scripts/prefabs/winona_battery_low.lua#L90-L100) [接收](../../../raw/current/scripts/prefabs/winona_spotlight.lua#L892-L893) [供电期限](../../../raw/current/scripts/prefabs/winona_spotlight.lua#L674-L699)
4. 保存／恢复与烧毁清理由宿主协调。聚光灯 OnSave 保存的是剩余 power 时限等状态，OnLoad 将其恢复到 AddBatteryPower；烧毁时先断 circuitnode，再 SetLoad(0)。本组件不会自行存读这两个字段，也不会因负载归零而断连接。[保存](../../../raw/current/scripts/prefabs/winona_spotlight.lua#L949-L966) [恢复](../../../raw/current/scripts/prefabs/winona_spotlight.lua#L970-L993) [烧毁](../../../raw/current/scripts/prefabs/winona_spotlight.lua#L870-L875)

### 不把“待机”误判为“关机”

投石机睡眠模式与唤醒但暂未有目标的分支都能 SetLoad(..., true)，同时还分别维护脑、战斗搜索、定时器和 LED；idle 是当前负载分类，由宿主决定。不能通过 IsIdle 推断实体完全停用，也不能只 SetLoad(0) 取消攻击。[睡眠](../../../raw/current/scripts/prefabs/winona_catapult.lua#L758-L777) [唤醒](../../../raw/current/scripts/prefabs/winona_catapult.lua#L778-L795)

## 常见陷阱与限制

- **设 0 后仍扣燃料**：设备自身零负载被跳过，普通电源仍有最低 rate，其他邻居也可能耗能；检查电源门禁及全部有效负载，而非用负数绕过。[消费者](../../../raw/current/scripts/prefabs/winona_battery_low.lua#L118-L138)
- **改一次原版设备 load 就永久改变功率**：后续 EnableLight／投石机状态切换会再次 SetLoad 覆盖。长期调整应明确接入真实状态更新位置及恢复边界，不给原版工厂字段一次赋值后宣称所有状态生效。[灯状态](../../../raw/current/scripts/prefabs/winona_spotlight.lua#L575-L593) [投石机](../../../raw/current/scripts/prefabs/winona_catapult.lua#L768-L790)
- **只装 powerload 就取得网络供电**：消费者要求已经连接的 circuitnode；真正供电还调用宿主 AddBatteryPower。拓扑、负载、燃料、设备工作分别核验，powerload 不代管权限或线路。[结算](../../../raw/current/scripts/prefabs/winona_battery_low.lua#L119-L138) [实际供电](../../../raw/current/scripts/prefabs/winona_battery_low.lua#L90-L100)

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 处理范围：锚点重定位 2 处（tuning +14，区间文本逐段比对一致）；结论修订：无（WINONA_BATTERY_MIN_LOAD 与聚光灯负载键值未变）。
- 依据：tuning 键级 hunk 地图＋变更清单命中 tuning.lua。

- 源码审阅：2026-10-23，Steam buildid 25643519；覆盖完整 powerload、低／高电源负载结算、低电源通知／供电，以及聚光灯安装／状态切换／供电时限／存读／烧毁和投石机 idle 分支。节点方法与燃料扣量分工复用当前版本 circuitnode／fueled Wiki；未全面审校技能解锁、所有设备、燃料效率修正或跨平台电路。
- 结构检查：本页格式、大小、来源与本地链接、源码锚点已通过；本页无新增完整 Lua 示例，不执行语法检查。
- 游戏运行：未运行；多电源分摊、免待机技能、状态重入、存读恢复与断电时序待实测。
