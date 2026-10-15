---
title: "circler：围绕目标移动并缩放盘旋影子"
created: 2026-10-26
updated: "2026-10-28"
type: component
summary: "直接驱动实体围绕目标的地面圆周位移、朝向与速度相关缩放，原版用于普通秃鹫影子。"
aliases: [circler, 绕目标盘旋, 秃鹫影子, 圆周移动]
tags: [component, lifecycle, prefab, server-client]
dst_build: "steam-buildid:26230496"
sources: [components/circler.lua, prefabs/circlingbuzzard.lua, prefabs/buzzardspawner.lua]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# circler：围绕目标移动并缩放盘旋影子

## 快速判断

- 职责与适用：围绕 `circleTarget` 的当前 XZ 坐标作圆周运动，直接写宿主 Transform 的位置、朝向及整体缩放；不是寻路或战斗 AI。[circler.lua:84-109](../../../raw/current/scripts/components/circler.lua#L84-L109)
- 优先使用：参考 `buzzardspawner` 生成普通 `circlingbuzzard` 后 `SetCircleTarget(inst)` → `Start()` 的组装。月亮变体 `circlingbuzzard_lunar` 使用另一组件，不能套用本文运动参数。[buzzardspawner.lua:25-30](../../../raw/current/scripts/prefabs/buzzardspawner.lua#L25-L30)、[circlingbuzzard.lua:169-184](../../../raw/current/scripts/prefabs/circlingbuzzard.lua#L169-L184)
- 重要副作用：`Start` 立即随机重置距离、角度、方向并移动实体；更新固定 Y 为 0，缩放覆盖为统一绝对比例，没有地形落点检查。[circler.lua:30-49](../../../raw/current/scripts/components/circler.lua#L30-L49)、[circler.lua:97-108](../../../raw/current/scripts/components/circler.lua#L97-L108)
- 修改边界：原版在影子 prefab 的服务端分支添加；显隐淡入淡出和实体销毁由 prefab／生成器负责，组件停更不等于移除影子。[circlingbuzzard.lua:87-151](../../../raw/current/scripts/prefabs/circlingbuzzard.lua#L87-L151)、[circler.lua:51-57](../../../raw/current/scripts/components/circler.lua#L51-L57)

## 关键接口

| 接口／字段 | 参数、默认值 | 条件、作用与证据 |
|---|---|---|
| `SetCircleTarget(tar)` | 目标实体或 `nil` | 仅赋引用，不自动启动；更新时读取目标 Transform，无效目标会停更并清空引用。[circler.lua:55-57](../../../raw/current/scripts/components/circler.lua#L55-L57)、[circler.lua:84-89](../../../raw/current/scripts/components/circler.lua#L84-L89) |
| `Start()`／`Stop()` | 无参数 | start 要求有效目标，每次重新随机初始化并注册更新；stop 只注销更新，不清目标、不恢复位置或缩放。[circler.lua:30-53](../../../raw/current/scripts/components/circler.lua#L30-L53) |
| `minDist`／`maxDist` | 默认 8／20 世界距离 | `Start` 用 `math.random(minDist, maxDist)` 取本轮固定半径 `distance`，不是每帧变半径。[circler.lua:13-14](../../../raw/current/scripts/components/circler.lua#L13-L14)、[circler.lua:36-39](../../../raw/current/scripts/components/circler.lua#L36-L39) |
| `minSpeed`／`maxSpeed`、对应 getter | 默认 5／7；getter 乘 0.01 | 有效速度默认 0.05–0.07 圈／秒；`GetSpeed(dt)` 返回带方向的 `speed * TWOPI * dt` 角增量，不是线速度。[circler.lua:10-11](../../../raw/current/scripts/components/circler.lua#L10-L11)、[circler.lua:59-70](../../../raw/current/scripts/components/circler.lua#L59-L70) |
| `minScale`／`maxScale`、对应 getter | 默认 8／12；getter 乘 0.1 | 默认绝对缩放 0.8–1.2；速度越快比例越小，每帧写三个轴相同值。[circler.lua:16-17](../../../raw/current/scripts/components/circler.lua#L16-L17)、[circler.lua:72-78](../../../raw/current/scripts/components/circler.lua#L72-L78)、[circler.lua:94-98](../../../raw/current/scripts/components/circler.lua#L94-L98) |

## 关键流程

`Start` 取随机初始速度、半径、角度和正／反方向，以目标 XZ 加圆周偏移立即定位。每次 `OnUpdate(dt)` 用 `GetSineVal(sineMod, true, inst)` 驱动速度插值，再钳制到 getter 给出的上下限；据速度计算缩放，积分角度、重算偏移和切线朝向，最后跟随目标当前 XZ 更新位置。[circler.lua:30-49](../../../raw/current/scripts/components/circler.lua#L30-L49)、[circler.lua:84-109](../../../raw/current/scripts/components/circler.lua#L84-L109)

实体睡眠映射到 `Stop`，唤醒映射到 `Start`，因此唤醒会重新随机选轨道，而非延续原角度。目标失效只停止运动并清引用，不删除自身。组件没有存读接口或专用移除钩子；原版影子 `persists = false`，由宿主重新创建和设置目标。[circler.lua:84-89](../../../raw/current/scripts/components/circler.lua#L84-L89)、[circler.lua:111-114](../../../raw/current/scripts/components/circler.lua#L111-L114)、[circlingbuzzard.lua:128-151](../../../raw/current/scripts/prefabs/circlingbuzzard.lua#L128-L151)

## 典型接入

### 普通秃鹫生成器：影子轨道与实际出鸟位置协作

`circlingbuzzard` 的服务端构造添加组件，但不自行指定中心或启动；`buzzardspawner` 的 `SpawnBuzzardShadow` 完成这两步。生成器按 `childspawner.childreninside` 对齐影子数量；醒着且非夜晚、非冬季时才由 `TryUpdateShadows` 尝试更新影子。[circlingbuzzard.lua:140-151](../../../raw/current/scripts/prefabs/circlingbuzzard.lua#L140-L151)、[buzzardspawner.lua:25-44](../../../raw/current/scripts/prefabs/buzzardspawner.lua#L25-L44)、[buzzardspawner.lua:135-142](../../../raw/current/scripts/prefabs/buzzardspawner.lua#L135-L142)

实际子体生成时，宿主 `OnSpawn` 读取一个影子的 `distance`、`angleRad` 作为可行走落点搜索参数，再让子体进入对应下落／滑翔等状态，并撤掉该影子。`RemoveBuzzardShadow` 调用 prefab 的 `KillShadow` 并移除列表项；生成器睡眠则直接移除全部影子。由此运动停止、淡出和删除是不同入口。[buzzardspawner.lua:15-23](../../../raw/current/scripts/prefabs/buzzardspawner.lua#L15-L23)、[buzzardspawner.lua:56-77](../../../raw/current/scripts/prefabs/buzzardspawner.lua#L56-L77)、[buzzardspawner.lua:189-201](../../../raw/current/scripts/prefabs/buzzardspawner.lua#L189-L201)、[circlingbuzzard.lua:87-99](../../../raw/current/scripts/prefabs/circlingbuzzard.lua#L87-L99)

## 常见陷阱与限制

- 不要把 `minSpeed` 与 `maxSpeed` 设为相等：缩放插值分母为两者 getter 的差，没有零分母保护；改变参数还须满足 `Start` 使用 `math.random` 的上下界要求。[circler.lua:36-37](../../../raw/current/scripts/components/circler.lua#L36-L37)、[circler.lua:94-98](../../../raw/current/scripts/components/circler.lua#L94-L98)
- `SetMode(mode)` 仅赋值 `circlerMode`；本文件没有消费该字段，也没有调用 `onaccelerate`，不能据这些名称承诺模式切换或加速回调。[circler.lua:19-28](../../../raw/current/scripts/components/circler.lua#L19-L28)、[circler.lua:84-109](../../../raw/current/scripts/components/circler.lua#L84-L109)

## 验证记录

- 源码审阅：2026-10-26，Steam buildid 26148036；覆盖组件全部实现，以及普通影子服务端配置、生成器设目标／启动、轨道字段消费者和睡眠／影子删除路径。简单档只描述基础绕行与原版影子接入。
- 游戏运行：未运行；未实测 Transform 运动／缩放、唤醒重随机和目标失效；未审阅 `GetSineVal`、网络 Transform 同步及月亮变体运动实现。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 通过；无新增 Lua 示例，未运行语法探针。

- 版本沿用：2026-10-28 原始哈希等价批量沿用-70（26148036→26230496，本页 sources 所列文件原始字节全部未变），页内未决项零标记、符号反查无命中、严格链接通过；批次依据见 [[updates/build-26230496/index#wiki-沿用记录]]。
相关：[[reference/components/childspawner]]。
