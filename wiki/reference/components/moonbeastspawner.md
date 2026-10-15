---
title: "moonbeastspawner"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "月台（moonbase）充能期间的月亮怪兽波次生成器：周期性把范围内 werepig/hound/gargoyle 变身成 moonhound/moonpig，屏幕外用随机破坏代替变身，停止时强制石化。"
aliases: ["moonbeastspawner", "月台怪兽生成器", "月亮怪兽波次", "moonbase 充能刷怪"]
tags: [component, ai, callback]
dst_build: "steam-buildid:26230496"
sources: ["components/moonbeastspawner.lua", "prefabs/moonbase.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# moonbeastspawner

## 快速判断

- 职责与适用：挂在 moonbase 实体上的服务端组件；`Start()` 后每 3 秒一轮：范围内 werepig/hound 变身月亮兽、gargoyle 已由 Start 一次性复活、玩家屏幕外则用拆墙/毁骨架代替表现，并按 `maxspawns`（默认 6）补足波次数量。[构造默认值](../../../raw/current/scripts/components/moonbeastspawner.lua#L1-L10) [波次生成](../../../raw/current/scripts/components/moonbeastspawner.lua#L64-L174)
- 优先使用：需要"充能期间持续刷怪、停充即石化"的 Boss 场景节奏 → `Start()`／`Stop()`／`ForcePetrify()` 三个入口；原版全部由 moonbase 的 timer/fullmoon 逻辑驱动。[moonbase.lua:704](../../../raw/current/scripts/prefabs/moonbase.lua#L704
- 重要副作用：变身不生成新位实体而是"替换"——旧实体隐藏、`persists = false` 后移除，新实体继承位置朝向并 `TrackEntity("moonbase")`；busy 状态的实体挂 `newstate` 监听等空闲再变。[MorphMoonBeast](../../../raw/current/scripts/components/moonbeastspawner.lua#L25-L41)
- 修改边界：组件无 OnSave/OnLoad，刷怪节奏完全由 moonbase 的 timer 状态重建；`Stop()` 时若 moonbase 在玩家屏幕外，直接 `ForcePetrify()` 兜底（正常石化由脑处理）。[Stop](../../../raw/current/scripts/components/moonbeastspawner.lua#L207-L219)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| 构造 | `(inst)` | `range = 30`、`period = 3`、`maxspawns = 6`，均可构造后改；`started`／`task`／`cc` 为运行状态。([moonbeastspawner.lua:1-10](../../../raw/current/scripts/components/moonbeastspawner.lua#L1-L10) |
| `Start()` | 无参 | 幂等；先复活范围内所有 `gargoyle`（`v:Reanimate(inst)`），再开 `DoPeriodicTask(period, DoSpawn)`。([moonbeastspawner.lua:192-205](../../../raw/current/scripts/components/moonbeastspawner.lua#L192-L205) |
| `Stop()` | 无参 | 取消任务并清 `cc`；宿主 `IsAsleep()` 时调 `ForcePetrify()`。([moonbeastspawner.lua:207-219](../../../raw/current/scripts/components/moonbeastspawner.lua#L207-L219) |
| `ForcePetrify()` | 无参 | 范围内所有 `moonbeast`：有 brain 则 `brain:ForcePetrify()`，屏幕外推 `moonpetrify` 事件。([moonbeastspawner.lua:178-189](../../../raw/current/scripts/components/moonbeastspawner.lua#L178-L189) |
| `DoSpawn` 内部循环 | 周期闭包 | 屏幕内：busy 实体挂 `newstate` 延迟变身；被冻/睡的原生月亮兽记入 `cc` 表并随机放 wake/thaw；屏幕外：10 距内随机杀墙/毁可工作骨架，其余月亮兽按 25% 抽样对月台本体 `workable:WorkedBy`；末尾按 `math.random(2)` 只步进 1-2 只直到总数达 `maxspawns`。([moonbeastspawner.lua:64-174](../../../raw/current/scripts/components/moonbeastspawner.lua#L64-L174) |
| 候选与生成池 | tag/名字表 | 变身候选 tag：`gargoyle`、`werepig`、`hound`（排除 `INLIMBO`）；循环内再跳过已是 `moonbeast`/`gargoyle`/`clay` 的实体；生成池 `MOONBEASTS = {"moonhound", "moonpig"}`。[候选 tag](../../../raw/current/scripts/components/moonbeastspawner.lua#L61-L63) [循环跳过](../../../raw/current/scripts/components/moonbeastspawner.lua#L78-L83) [生成池](../../../raw/current/scripts/components/moonbeastspawner.lua#L19-L23) |

## 典型接入

### 复用月台充能节奏（原版触发链参照）

- 前提与选择：组件须挂在有 AnimState/timer 的"月台式"实体上；原版入口是 moonbase 给满月＋法杖充能时启动 timer，timer 到点或修理完成再 `Start`／`Stop`，模组不宜绕开宿主直接长期 `Start`。
- 生效与撤销：`Start` 到 `Stop` 之间持续刷怪；任何一次 `Stop` 都会清任务并按需石化场上月亮兽。
- 原版依据：启动点 [moonbase.lua:443](../../../raw/current/scripts/prefabs/moonbase.lua#L443)（timer `moonchargepre` 到点）与 [270](../../../raw/current/scripts/prefabs/moonbase.lua#L270)（充能中重入），停止点 [242](../../../raw/current/scripts/prefabs/moonbase.lua#L242)、[292](../../../raw/current/scripts/prefabs/moonbase.lua#L292)，充能被打断时 [492](../../../raw/current/scripts/prefabs/moonbase.lua#L492) 强制石化。

```lua
-- 宿主 prefab 的服务端 master_postinit 中：
inst:AddComponent("moonbeastspawner")
inst.components.moonbeastspawner.range = 24        -- 可选：覆盖默认 30
inst.components.moonbeastspawner.maxspawns = 4     -- 可选：覆盖默认 6
-- 充能开始时：
inst.components.moonbeastspawner:Start()
-- 充能结束／被打断时：
inst.components.moonbeastspawner:Stop()
-- 立即石化场上月亮兽（不经过脑的定时）：
inst.components.moonbeastspawner:ForcePetrify()
```

- 调整与验证：屏幕外分支会真实摧毁墙与可工作实体（([moonbeastspawner.lua:116-127](../../../raw/current/scripts/components/moonbeastspawner.lua#L116-L127)），低表现模式等价于消耗世界耐久，测试时留意。

## 常见陷阱与限制

- 变身是不可逆替换：旧实体（如普通猎犬）被移除且 `persists = false`，掉落、驯化状态等全部丢失（([moonbeastspawner.lua:35-40](../../../raw/current/scripts/components/moonbeastspawner.lua#L35-L40)）；想保留原实体数据需在 `moontransformed` 事件接收侧自记录。
- 生成候选要求目标有 `sg` 与 `brain` 之外的直接前提：`MorphMoonBeast` 只查 health 未死；对无 SG 实体会直接报错，`DoSpawn` 循环里也直接读 `v.sg:HasStateTag`（[L81](../../../raw/current/scripts/components/moonbeastspawner.lua#L81-L82)），候选 tag 范围内混入无 SG 实体会崩。
- 生成位置用 `FindWalkableOffset` 但不查瓦片禁忌，屏幕外生成还带碰撞重试（[L136-160](../../../raw/current/scripts/components/moonbeastspawner.lua#L136-L160)）；在自定义封闭空间使用时可能把怪生成进墙内。
- 生成实体带 [[reference/components/entitytracker]] 的 `moonbase` 追踪与 [[reference/components/spawnfader]] 淡入（[L162-166](../../../raw/current/scripts/components/moonbeastspawner.lua#L162-L166)），复用生成池时依赖这两个组件存在。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，通读 components/moonbeastspawner.lua 全文，核验 moonbase.lua 全部调用点与挂载。
- 游戏运行：未运行。
- 结构检查：未执行。

相关链接：[[reference/components/entitytracker]]、[[reference/components/spawnfader]]、[[reference/components/timer]]
