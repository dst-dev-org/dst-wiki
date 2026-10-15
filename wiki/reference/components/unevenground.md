---
title: "unevenground：不平地面范围通知源"
created: 2026-10-26
updated: "2026-10-28"
type: component
summary: "让环境实体周期通知附近非幽灵玩家，由 carefulwalker 再判定实际影响半径与减速资格。"
aliases: [unevenground, 不平地面通知, 陷坑谨慎行走, 地形范围减速通知]
tags: [component, lifecycle, event, server-client]
dst_build: "steam-buildid:26230496"
sources: [components/unevenground.lua, prefabs/antlion_sinkhole.lua]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# unevenground：不平地面范围通知源

## 快速判断

- 职责与适用：挂在陷坑等环境实体上，遍历 `AllPlayers`，向检测圈内非 `playerghost` 玩家发 `unevengrounddetected`；不采样地图、不修改 Physics，也不直接设置移动倍率。[unevenground.lua:1-26](../../../raw/current/scripts/components/unevenground.lua#L1-L26)
- 优先使用：需要原版不平地面语义的区域通知时沿该生产端接线；实际玩家追踪、半径判断、减速资格和网络表现见 [[reference/components/carefulwalker#关键流程]]。普通独立速度效果用 [[reference/components/locomotor#关键接口]]，无需伪造不平地面通知。
- 重要副作用：通知范围默认15，真实影响半径默认3，两者不同。启动任务运行时立即通知一次，之后周期刷新；停止不发“关闭”事件，消费者靠原记录到期撤销。[unevenground.lua:1-26](../../../raw/current/scripts/components/unevenground.lua#L1-L26)、[[reference/components/carefulwalker#原版不平地面通知的生产端]]
- 修改边界：原版蚁狮陷坑仅在服务端添加。组件没有自己的端断言、存读接口或 replica；醒着构造会自动排启动任务，不需要额外更新器。[antlion_sinkhole.lua:242-253](../../../raw/current/scripts/prefabs/antlion_sinkhole.lua#L242-L253)、[unevenground.lua:1-11](../../../raw/current/scripts/components/unevenground.lua#L1-L11)

## 关键接口

| 接口／字段 | 默认值、参数 | 条件、副作用与证据 |
|---|---|---|
| `radius` | 3 世界距离 | 通知 payload 的真实影响半径；通知函数每次读取当前值，但生产端不使用它筛选玩家。[unevenground.lua:4-17](../../../raw/current/scripts/components/unevenground.lua#L4-L17) |
| `detectradius` | 15 世界距离 | 启动任务时计算平方并固定传给周期任务；玩家到环境实体距离平方须严格小于它才通知。[unevenground.lua:5-25](../../../raw/current/scripts/components/unevenground.lua#L5-L25) |
| `detectperiod` | 0.6 秒 | 周期任务间隔；首次周期延迟为其 `0.3 + 0.7 * random` 倍，同时也是每次通知中的 `period` 字段。[unevenground.lua:6-25](../../../raw/current/scripts/components/unevenground.lua#L6-L25) |
| `Start()`／`Stop()` | 无参数 | start 在无任务时排零延迟启动任务；随后同字段保存周期任务。stop 能取消任一阶段，清空任务字段；不修改 `enabled`。[unevenground.lua:22-39](../../../raw/current/scripts/components/unevenground.lua#L22-L39) |
| `Enable()`／`Disable()` | `enabled` 初始为 `true` | enable 仅从禁用变启用时、且实体醒着才 start；disable 仅从启用变禁用时 stop。直接 Start 不检查 enabled／睡眠。[unevenground.lua:28-55](../../../raw/current/scripts/components/unevenground.lua#L28-L55) |

## 关键流程

### 通知协议与启停

构造时若实体醒着，`Start` 排 `DoTaskInTime(0, OnStartTask, self)`。启动任务读取检测参数、创建周期任务，再立即运行一次通知。每次通知取环境实体当前世界坐标，向范围内非幽灵玩家实体推送 `{inst=环境实体, radius=当前radius, period=当前detectperiod}`；不检查目标是否已装 carefulwalker。[unevenground.lua:1-32](../../../raw/current/scripts/components/unevenground.lua#L1-L32)

实体睡眠、组件移除均映射到 `Stop`；醒来只在 enabled 为真时重新启动。暂停通知不会立即清除玩家已有追踪，关闭延迟由消费者记录的有效期和更新决定，不能承诺 Disable 当帧恢复速度。消费者接收与撤销机制复用 [[reference/components/carefulwalker#目标时效距离与真正的减速资格]]。[unevenground.lua:34-64](../../../raw/current/scripts/components/unevenground.lua#L34-L64)

## 典型接入

### 蚁狮陷坑：宿主阶段决定区域是否生效

`antlion_sinkhole` 服务端构造添加 `unevenground`，把 `radius` 改为 `TUNING.ANTLION_SINKHOLE.UNEVENGROUND_RADIUS`；`UpdateOverrideSymbols(inst, state)` 在阶段等于 `NUM_CRACKING_STAGES` 时 Enable，其他阶段 Disable。视觉裂缝和通知开关由同一宿主函数驱动，组件本身不理解坍塌／修复阶段。[antlion_sinkhole.lua:22-34](../../../raw/current/scripts/prefabs/antlion_sinkhole.lua#L22-L34)、[antlion_sinkhole.lua:242-261](../../../raw/current/scripts/prefabs/antlion_sinkhole.lua#L242-L261)

宿主 `OnSave` 保存坍塌阶段或剩余修复数；`OnLoad` 恢复这些字段并重新调用 `UpdateOverrideSymbols`，从玩法状态推导通知开关。unevenground 自身不保存 enabled、检测参数或任务剩余时间，重建时先回到默认启用，再由宿主配置与阶段恢复。[antlion_sinkhole.lua:187-206](../../../raw/current/scripts/prefabs/antlion_sinkhole.lua#L187-L206)、[unevenground.lua:1-11](../../../raw/current/scripts/components/unevenground.lua#L1-L11)

## 常见陷阱与限制

- **运行中改检测配置未完全生效**：周期任务固定使用启动时的检测半径平方和调度间隔；`radius` 和事件的 `period` 却每次读当前字段。动态改 `detectperiod` 会使调度周期与声明有效期不一致。要整体重建参数，配置后 Stop／Start，并保持正的检测周期；不要把仅改字段当作任务重排。[unevenground.lua:13-39](../../../raw/current/scripts/components/unevenground.lua#L13-L39)
- **Stop 后 Enable 不一定重启**：Stop 不清 enabled，而 Enable 在它已为真时无操作；临时停更后直接 Start，完整禁用／恢复用 Disable／Enable。直接 Start 又可绕过禁用／睡眠条件，调用方须遵守同一启停协议。[unevenground.lua:28-61](../../../raw/current/scripts/components/unevenground.lua#L28-L61)
- **通知不保证减速**：接收通知圈不等于真实影响圈；玩家还需符合 carefulwalker 条件。扩大 radius 却不考虑 detectradius 可能让影响圈内部分玩家收不到通知，不能只调一个数就承诺整个区域覆盖。详见 [[reference/components/carefulwalker#目标时效距离与真正的减速资格]]。[unevenground.lua:13-25](../../../raw/current/scripts/components/unevenground.lua#L13-L25)

## 验证记录

- 源码审阅：2026-10-26，Steam buildid 26148036；覆盖组件全文、两阶段任务、事件字段与配置读取时机、睡眠／移除停止、蚁狮陷坑服务端构造和阶段／存读接线。carefulwalker 的实际消费与撤销复用其同版本已核验页。
- 游戏运行：未运行；未实测通知延迟、动态改周期、睡眠和加载时序；未审阅其他陷坑宿主、所有玩家模式及网络表现。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 通过；无新增 Lua 示例，未运行语法探针。

- 版本沿用：2026-10-28 原始哈希等价批量沿用-70（26148036→26230496，本页 sources 所列文件原始字节全部未变），页内未决项零标记、符号反查无命中、严格链接通过；批次依据见 [[updates/build-26230496/index#wiki-沿用记录]]。
相关：[[reference/components/carefulwalker]]、[[reference/components/locomotor]]。
