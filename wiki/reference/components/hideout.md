---
title: "hideout：暂存并释放原有生物实体"
created: 2026-10-26
updated: "2026-10-28"
type: component
summary: "为生物提供临时藏身点，保留实体并按周期释放，区别于补员式巢穴。"
aliases: [hideout, 生物藏身点, 暂存生物, 洞穴光源暂存蝙蝠, cavelight]
tags: [component, lifecycle, callback, ai, server-client]
dst_build: "steam-buildid:26230496"
sources: [components/hideout.lua, prefabs/cavelight.lua, brains/batbrain.lua, actions.lua]
reliability: verified
validation: source-reviewed
status: active
---

# hideout：暂存并释放原有生物实体

## 快速判断

- 职责与适用：把已有生物实体收进 `storedcreatures`，移出场景并休眠 Brain，之后释放同一实体；不生成新 prefab，也没有容量或标签准入检查。[hideout.lua:3-24](../../../raw/archive/build-26148036/scripts/components/hideout.lua#L3-L24)、[hideout.lua:120-193](../../../raw/archive/build-26148036/scripts/components/hideout.lua#L120-L193)
- 优先使用：需要保留进入者自身状态的临时藏身点。通用 `GOHOME` 在目标没有 `spawner`、`childspawner` 时才转发给 `hideout:GoHome`。[actions.lua:2946-2961](../../../raw/archive/build-26148036/scripts/actions.lua#L2946-L2961)
- 重要副作用：收纳会把生物移到 `(0,0,0)`、休眠 Brain 并停止所有声音；释放会恢复场景、重新定位、尝试唤醒 Brain，并可设置战斗目标。[hideout.lua:120-193](../../../raw/archive/build-26148036/scripts/components/hideout.lua#L120-L193)
- 修改边界：原版洞穴光源仅服务端添加组件；释放受库存、宿主有效性、睡眠和生命状态约束。宿主移除／组件移除没有组件自带的居民清理钩子，调用者须安排居民去向。[cavelight.lua:232-246](../../../raw/archive/build-26148036/scripts/prefabs/cavelight.lua#L232-L246)、[hideout.lua:195-253](../../../raw/archive/build-26148036/scripts/components/hideout.lua#L195-L253)

## 关键接口

| 接口／字段 | 参数、返回／默认值 | 条件、作用与证据 |
|---|---|---|
| `GoHome(child)` | 已有实体；不返回成功布尔值 | 重复收纳只打印提示并返回；正常路径登记实体、更新数量，然后离场、休眠与通知。[hideout.lua:168-193](../../../raw/archive/build-26148036/scripts/components/hideout.lua#L168-L193) |
| `ReleaseChild(target, prefab, radius)` | 随机选一个居民；返回所选 `child` | `prefab` 参数未使用；`radius` 默认 0.5，再加宿主物理半径寻找可行走且不近洞的位置。成功后才从表中移除。[hideout.lua:115-166](../../../raw/archive/build-26148036/scripts/components/hideout.lua#L115-L166) |
| `CanRelease()`、`spawnoffscreen` | 布尔判定；离屏释放默认 `false` | 有居民、宿主有效且未死，并且醒着或允许离屏释放才可继续；不检查实际是否能找到释放位置。[hideout.lua:17](../../../raw/archive/build-26148036/scripts/components/hideout.lua#L17)、[hideout.lua:195-217](../../../raw/archive/build-26148036/scripts/components/hideout.lua#L195-L217) |
| `SetSpawnPeriod(period, variance)` | 秒；默认周期 20、波动 2 | 省略波动时使用周期的 10%；每次尝试释放前重设剩余时间为 `period + [-variance, variance]`，未做数值校验。[hideout.lua:13-15](../../../raw/archive/build-26148036/scripts/components/hideout.lua#L13-L15)、[hideout.lua:27-38](../../../raw/archive/build-26148036/scripts/components/hideout.lua#L27-L38) |
| `StartSpawning()`、`StopSpawning()` | 开始／停止周期任务 | start 把剩余时间设零并防止重复任务；首次释放要等待周期回调。stop 只取消任务，居民保持收纳状态。[hideout.lua:44-62](../../../raw/archive/build-26148036/scripts/components/hideout.lua#L44-L62) |
| `LongUpdate(dt)` | 长更新秒数 | 仅运行任务存在时调用一次 `OnUpdate(dt)`；即使跨过多个周期，本次也至多尝试释放一个居民。[hideout.lua:32-38](../../../raw/archive/build-26148036/scripts/components/hideout.lua#L32-L38)、[hideout.lua:225-229](../../../raw/archive/build-26148036/scripts/components/hideout.lua#L225-L229) |

### 回调的实际调用约定

四个 setter 只保存函数；应按实现的调用方式接收参数，不能把它们统一写成 `(inst, child)`。[hideout.lua:64-78](../../../raw/archive/build-26148036/scripts/components/hideout.lua#L64-L78)

| setter | 实际调用 | 时机与限制 |
|---|---|---|
| `SetSpawnedFn(fn)` | `fn(inst, child)` | 实体恢复、定位、Brain 唤醒及可选设敌后调用；此时尚未从收纳表扣除。[hideout.lua:131-164](../../../raw/archive/build-26148036/scripts/components/hideout.lua#L131-L164) |
| `SetGoHomeFn(fn)` | `fn(inst, count)` | 收纳后调用，但 `count` 在本组件中没有定义，不能据此取得居民数量；数量应读组件字段。[hideout.lua:174-188](../../../raw/archive/build-26148036/scripts/components/hideout.lua#L174-L188) |
| `SetOccupiedFn(fn)` | `fn(hideout组件, inst)` | 首个居民进入时以冒号形式调用，隐含首参是组件。[hideout.lua:190-192](../../../raw/archive/build-26148036/scripts/components/hideout.lua#L190-L192) |
| `SetVacateFn(fn)` | `fn(hideout组件, inst)` | 最后一个居民成功释放、尚未扣除表项时，以冒号形式调用。[hideout.lua:157-164](../../../raw/archive/build-26148036/scripts/components/hideout.lua#L157-L164) |

## 关键流程

### 收纳、释放与存读

`GoHome` 不把生物转换为库存计数：`storedcreatures[child] = child` 持有原实体。释放先 `CanRelease`，随机取居民，再由内部 `DoReleaseChild` 找位置；找不到位置就不恢复场景、不扣除表项。内部函数不负责扣账，外部应使用 `ReleaseChild`。[hideout.lua:120-193](../../../raw/archive/build-26148036/scripts/components/hideout.lua#L120-L193)

`OnSave` 把居民 GUID 同时写入 `data.storedcreatures` 和 references，另存任务是否存在的 `spawning`。`OnLoad` 按该标志重启；`LoadPostPass` 从 `newents[guid].entity` 取回居民并重新调用 `GoHome`，会再次触发相应收纳副作用和回调；找不到引用则跳过。不保存原剩余释放时间，也不保存周期配置及回调，这些由宿主重建。[hideout.lua:80-113](../../../raw/archive/build-26148036/scripts/components/hideout.lua#L80-L113)

停止自动释放使用 `StopSpawning`，并不会放出居民。整个组件没有 `OnRemoveFromEntity`／`OnRemoveEntity` 实现；拆除宿主或移除组件前，需由接入方明确处理任务及收纳实体，不能把移除当成自动疏散。[hideout.lua:57-62](../../../raw/archive/build-26148036/scripts/components/hideout.lua#L57-L62)、[hideout.lua:1-253](../../../raw/archive/build-26148036/scripts/components/hideout.lua#L1-L253)

## 典型接入

### 洞穴光源暂存夜间撤离的蝙蝠

本节是洞穴光源暂存并释放原实体；若问蝙蝠巢穴／地表洞口在黄昏开启出巢，见 [[reference/components/childspawner#洞穴黄昏刷新蝙蝠由谁控制|childspawner 的昼夜宿主接线]]。

蝙蝠 Brain 的 `EscapeAction` 在洞穴白天尝试回原巢；其他时段在半径内找 `batdestination`，目标有 `childspawner` 或 `hideout` 即生成 `GOHOME` 动作。动作分派随后把进入者传给 `hideout:GoHome`；这里不要求进入者预先登记在巢穴表中。[batbrain.lua:19-42](../../../raw/archive/build-26148036/scripts/brains/batbrain.lua#L19-L42)、[actions.lua:2951-2957](../../../raw/archive/build-26148036/scripts/actions.lua#L2951-L2957)

`cavelight`、`cavelight_small`、`cavelight_tiny`、`cavelight_atrium` 共用构造链，带 `batdestination` 标签；服务端添加 `hideout`，设置周期 `5 ± 4` 秒与 `onspawned`，释放时向居民发送 `fly_back`。世界 `cavephase` 变化经光照过渡时间后调用 `OnSpawnTask`，洞穴白天开启释放，其他阶段停止；初始化也按当前光照阶段决定启停。这些行为来自宿主，不是组件自动监听昼夜。[cavelight.lua:128-168](../../../raw/archive/build-26148036/scripts/prefabs/cavelight.lua#L128-L168)、[cavelight.lua:188-246](../../../raw/archive/build-26148036/scripts/prefabs/cavelight.lua#L188-L246)、[cavelight.lua:249-274](../../../raw/archive/build-26148036/scripts/prefabs/cavelight.lua#L249-L274)

## 常见陷阱与限制

- **返回居民不等于释放成功**：`ReleaseChild` 无论内部找位成功与否都返回所选实体；需要确认结果时检查它是否仍在 `storedcreatures`，不要仅判断返回值非空。[hideout.lua:148-166](../../../raw/archive/build-26148036/scripts/components/hideout.lua#L148-L166)
- **批量释放可能不终止**：`ReleaseAllChildren` 在 `CanRelease()` 为真期间循环；若位置持续不可用，库存不减少而门禁仍为真，就会持续循环。此为控制流推导，未运行复现；有地形风险时使用有次数上限的单次释放策略。[hideout.lua:120-128](../../../raw/archive/build-26148036/scripts/components/hideout.lua#L120-L128)、[hideout.lua:148-166](../../../raw/archive/build-26148036/scripts/components/hideout.lua#L148-L166)、[hideout.lua:195-223](../../../raw/archive/build-26148036/scripts/components/hideout.lua#L195-L223)
- **自定义门禁返回 false 无效**：源码字段拼作 `canrealeasefn`，调用后忽略返回值并继续返回 `true`；不能依赖它否决释放。[hideout.lua:212-216](../../../raw/archive/build-26148036/scripts/components/hideout.lua#L212-L216)
- **修改周期不会重建现有轮询任务**：`StartUpdate` 只在无任务时取 `min(period - variance, 5 + random * 5)` 作为轮询间隔；调整参数需考虑停启，且应保证 `period > variance` 以避免非正间隔。[hideout.lua:27-62](../../../raw/archive/build-26148036/scripts/components/hideout.lua#L27-L62)

## 验证记录

- 检索分流补充：2026-10-27；明确 cavelight 暂存／释放与 childspawner 昼夜出巢的不同入口，原组件、Brain、动作与光源证据沿用既有审阅。
- 源码审阅：2026-10-26，Steam buildid 26148036；覆盖组件全部实现、收放实体与回调参数、定时与长更新、GUID 存读，以及洞穴光源注册／服务端配置／阶段启停、蝙蝠逃离动作和通用 `GOHOME` 分派。
- 游戏运行：未运行；未实测地形无可用位置、重复收纳回调、实体移除后的居民处置及存读表现；未审阅 `fly_back` 的后续状态图处理或其他 hideout 宿主。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 通过，格式／大小无问题，源码链接无错误或警告；正文无新增 Lua 示例，未运行语法探针。

相关：[[reference/components/homeseeker]]、[[reference/components/childspawner]]。
