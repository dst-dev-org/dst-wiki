---
title: "frostybreather"
created: "2026-10-23"
updated: "2026-10-28"
type: component
summary: "为角色低温闲置动画发出白色呼气粒子，以服务端动画事件通知各客户端表现；不造成伤害、降温或冰冻。"
aliases: ["frostybreather", "低温呼气", "白色吐息", "寒气表现", "frostbreath"]
tags: [component, callback, lifecycle, event, network, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/frostybreather.lua", "components/temperatureoverrider.lua", "prefabs/frostbreath.lua", "prefabs/player_common.lua", "prefabs/player_common_extensions.lua", "prefabs/wx78_common.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# frostybreather

## 快速判断

- 职责与适用：管理宿主的本地 `frostbreath` 子实体；低温时监听呼气时机，朝背面时不发粒子。完整组件只管理监听、偏移与特效，没有伤害、体温修改或冻结调用；需要这些效果分别查 [[reference/components/combat]]、[[reference/components/temperature]]、[[reference/components/freezable]]。[创建与销毁](../../../raw/current/scripts/components/frostybreather.lua#L47-L65) [朝向与发射](../../../raw/current/scripts/components/frostybreather.lua#L93-L100)
- 优先使用：原版玩家已有此组件；改嘴部位置用 `SetOffsetFn(fn)`，临时强制呼气用 `ForceBreathOn()`／`ForceBreathOff()`，不要再给玩家重复添加组件。[玩家接线](../../../raw/current/scripts/prefabs/player_common.lua#L2490-L2491) [强制开关](../../../raw/current/scripts/components/frostybreather.lua#L105-L115) [偏移接口](../../../raw/current/scripts/components/frostybreather.lua#L119-L129)
- 重要副作用：服务端在宿主 `animover` 且 SG 带 `idle` 时推送 `frostybreather.breathevent` 并本地发射；客户端监听网络事件再发自己的粒子，特效实体本身不联网。[事件生产与分端监听](../../../raw/current/scripts/components/frostybreather.lua#L20-L35) [本地特效](../../../raw/current/scripts/prefabs/frostbreath.lua#L69-L82)
- 修改边界：组件构造创建 `net_event`，原版在双方的 `SetPristine` 前添加；自定义宿主须有 Transform、AnimState、Network，以及服务端有效 SG／动画结束事件，不能仅在服务端 PostInit 动态补齐来代替双方构造。[网络变量](../../../raw/current/scripts/components/frostybreather.lua#L1-L12) [构造位置](../../../raw/current/scripts/prefabs/player_common.lua#L2490-L2491) [pristine 与端界](../../../raw/current/scripts/prefabs/player_common.lua#L2611-L2614)

## 关键接口

| 接口／字段 | 参数／初值 | 条件、作用与证据 |
|---|---|---|
| `OnTemperatureChanged(temperature)` | 世界温度监听回调 | 实际忽略入参数值，读 `GetLocalTemperature(inst)`；未强制时，局部温度 `> TUNING.FROSTY_BREATH` 停止，否则创建。当前阈值为 -5。[判定](../../../raw/current/scripts/components/frostybreather.lua#L83-L88) [常量](../../../raw/current/scripts/tuning.lua#L1612) |
| `StartBreath()`／`StopBreath()` | 无参 | 幂等创建／移除 `frostbreath` 子实体；`enabled` 时同时装卸当前端监听。创建不立即发射。[实现](../../../raw/current/scripts/components/frostybreather.lua#L47-L65) |
| `Enable()`／`Disable()` | `enabled = true` | 开／关监听，不删除已有特效，不修改 `forced_breath`，也不阻止直接调用 `EmitOnce()`。[实现](../../../raw/current/scripts/components/frostybreather.lua#L69-L79) |
| `EmitOnce()` | 无参 | 只有 `breath` 已存在才工作；排除 `FACING_UP`／`UPRIGHT`／`UPLEFT`，重新取偏移再 `Emit()`；不检查 `enabled` 或 `idle`。[实现](../../../raw/current/scripts/components/frostybreather.lua#L93-L100) |
| `ForceBreathOn()`／`ForceBreathOff()` | `forced_breath = false` | 前者关闭温度自动控制并确保特效存在；后者解除强制并立即重做温度判定，不等同必定关闭。[实现](../../../raw/current/scripts/components/frostybreather.lua#L105-L115) |
| `SetOffset(x, y, z)`／`SetOffsetFn(fn)` | 默认 `Vector3(0, 0, 0)` | `fn(inst)` 返回有 `Get()` 的向量，非 nil 时优先于静态 offset；清掉回调才回退静态值。不是返回三个裸数。[初值](../../../raw/current/scripts/components/frostybreather.lua#L3-L7) [实现](../../../raw/current/scripts/components/frostybreather.lua#L119-L129) |

### 温度触发与表现边界

- 构造只订阅世界 `temperature` 并立即判断一次；本组件没有周期更新或位置监听。`GetLocalTemperature` 按宿主坐标读取最近覆盖区、局部火山温度或世界温度；只移动进出局部温度区，不保证立即触发本组件重判。[订阅](../../../raw/current/scripts/components/frostybreather.lua#L9-L12) [局部温度来源](../../../raw/current/scripts/components/temperatureoverrider.lua#L14-L36)
- `enabled`、`forced_breath`、`offset` 都是本端 Lua 状态，组件内只有呼气事件是 netvar；不要把服务端单独 `ForceBreathOn` 当作这些配置已复制。客户端 `breath == nil` 时收到事件也不会发射；跨端强制表现须另行确保客户端状态满足条件。[构造](../../../raw/current/scripts/components/frostybreather.lua#L1-L12) [客户端消费者](../../../raw/current/scripts/components/frostybreather.lua#L20-L35) [发射前提](../../../raw/current/scripts/components/frostybreather.lua#L93-L100)
- 专用服务器的 `frostbreath.Emit` 是空函数；非专用端初始化 VFX，并在每次 `Emit` 加入粒子。服务器无需渲染不等于可删服务端组件，它仍负责网络事件。[专用端分支](../../../raw/current/scripts/prefabs/frostbreath.lua#L78-L87) [粒子发射](../../../raw/current/scripts/prefabs/frostbreath.lua#L48-L63)

## 典型接入

### 玩家嘴部偏移与骑乘高度

原版 `player_common` 在公共构造添加组件并设置 `GetFrostyBreatherOffset`；回调读 `replica.rider:IsRiding()`，返回骑乘高度 `Vector3(.3, 3.75, 0)` 或普通高度 `Vector3(.3, 1.15, 0)`。自定义角色只改高度时，应在双方公共初始化保留同等骑乘分支；已有 `offset_fn` 会盖过单独的 `SetOffset`。[偏移选择](../../../raw/current/scripts/prefabs/player_common.lua#L185-L191) [挂接](../../../raw/current/scripts/prefabs/player_common.lua#L2490-L2491) [优先级](../../../raw/current/scripts/components/frostybreather.lua#L127-L129)

### WX 温控模块的强制低温表现

`wx78_common.AddTemperatureModuleLeaning` 在模块倾向为冷时调用 `ForceBreathOn`，热或中性时调用 `ForceBreathOff`；真实热源由同函数中的 `heater:SetThermics` 配置承担。该参照说明“白气”与温度作用分工，不能只调用呼气开关来获得制冷，也不能据此推断跨端配置已自动同步。[热／中性分支](../../../raw/current/scripts/prefabs/wx78_common.lua#L434-L461) [冷分支](../../../raw/current/scripts/prefabs/wx78_common.lua#L464-L473)

## 常见陷阱与限制

- **解除强制后仍呼气：** 当局部温度仍不高于阈值，`ForceBreathOff()` 会保留／重建特效，这是恢复自动判断；需要停止自动动画触发时才考虑 `Disable()`，它仍不是粒子实体删除。[解除强制](../../../raw/current/scripts/components/frostybreather.lua#L112-L115) [禁用监听](../../../raw/current/scripts/components/frostybreather.lua#L76-L79)
- **强制后没有立即喷气：** `ForceBreathOn` 只创建，仍等 `animover + idle`；背朝镜头被过滤，`Disable` 后也无自动事件。直接 `EmitOnce` 则绕过监听门禁，但不会替调用者自动推送网络事件。[创建](../../../raw/current/scripts/components/frostybreather.lua#L47-L53) [事件门禁](../../../raw/current/scripts/components/frostybreather.lua#L24-L27) [发射](../../../raw/current/scripts/components/frostybreather.lua#L93-L100)
- **把低温显示当角色体温：** 判定读局部环境温度，不读角色 `temperature.current`；保暖或角色体温与白气可不同，不据此判定冻伤。[温度判断](../../../raw/current/scripts/components/frostybreather.lua#L83-L88) [环境来源](../../../raw/current/scripts/components/temperatureoverrider.lua#L29-L36)
- **死亡与清理：** 原版变幽灵流程调用 `Disable`，复活调用 `Enable`；组件移除时停止温度监听并移除特效／当前有效监听。组件无存档方法，重建时重新判断；不要把强制状态当持久配置。[幽灵禁用](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L670) [复活启用](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L284) [移除清理](../../../raw/current/scripts/components/frostybreather.lua#L15-L17)

## 验证记录

- 版本增量复核 26148036（2026-10-25）：锚点重定位 4 处（tuning FROSTY_BREATH +4、player_common 组件挂接 -12／pristine 段 -14）；无结论修订，仅锚点迁移——FROSTY_BREATH=-5 值未变（tuning-hunk-map 5 改值键不含），frostybreather.lua、frostbreath.lua、temperatureoverrider.lua、wx78_common.lua 全部未变；player_common_extensions 的幽灵禁用／复活启用行（670/284）在 852 行插入点之前、未变。依据：tuning-hunk-map 键级审阅＋变更清单命中 prefabs/player_common.lua、prefabs/player_common_extensions.lua、tuning.lua。
- 源码审阅：2026-10-23，Steam buildid 25643519；完整核验 `frostybreather` 与 `frostbreath`，定向核验局部环境温度函数、玩家双方构造／偏移、幽灵与复活调用、WX 模块分工和阈值。上文引用对应实际核验范围。
- 结构检查：本页 `validate_page.py` 字段、大小与本地链接检查通过；`validate_source_links.py --strict` 源码锚点检查通过。不包含完整 Lua 示例，无语法检查项。
- 游戏运行：未运行；局部温度区进出刷新、跨端强制表现、幽灵与复活的实际可见效果待实测；未覆盖其他模组的特效替换。

相关：[[reference/components/temperatureoverrider]]；[[reference/components/temperature]]；[[reference/components/freezable]]。
