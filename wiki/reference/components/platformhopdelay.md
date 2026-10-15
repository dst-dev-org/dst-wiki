---
title: "platformhopdelay"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "给船平台配置跳跃起跳前的延迟 ticks，locomotor 取目标与当前平台延迟的较大值，无值时回退全局 TUNING。"
aliases: ["platformhopdelay", "平台跳跃延迟", "龙舟跳跃延迟", "hop delay"]
tags: [component, tuning]
size_class: simple
dst_build: "steam-buildid:26230496"
sources: ["components/platformhopdelay.lua", "prefabs/yotd_boats.lua", "components/locomotor.lua", "tuning.lua", "prefabs/player_common_extensions.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# platformhopdelay

## 快速判断

- 职责与适用：在平台实体（原版只有 YotD 龙舟 body）上存一个 `delayticks`，供 locomotor 在发起平台跳跃前决定延迟；默认 8 ticks 与 `TUNING.PLATFORM_HOP_DELAY_TICKS` 相同。[构造](../../../raw/current/scripts/components/platformhopdelay.lua#L1-L4) [挂载](../../../raw/current/scripts/prefabs/yotd_boats.lua#L596-L598)
- 优先使用：想改变某平台的起跳等待时给该平台挂组件（或改默认值）；locomotor 用 `math.max(目标平台, 当前平台)`，任一侧有值即覆盖全局默认。[消费](../../../raw/current/scripts/components/locomotor.lua#L1283-L1286)
- 重要副作用：组件在 `SetPristine()` 之前挂载，客户端实体也会构造一份；但数值无网络同步，客户端副本只是默认值，服务端 `SetDelay` 不会反映到客户端。[挂载位置](../../../raw/current/scripts/prefabs/yotd_boats.lua#L596-L598)
- 修改边界：水中漂浮的延迟不走本组件，用 `inst.forced_platformhopdelay` 或 `TUNING.PLATFORM_FLOATING_HOP_DELAY_TICKS`；玩家个人延迟由客户端授权设置写入 `forced_platformhopdelay`。[水路径](../../../raw/current/scripts/components/locomotor.lua#L1279-L1281) [玩家设置](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L1054-L1062)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `delayticks` | `number`（默认 8） | 构造即设 8；locomotor 读取的唯一字段。[实现](../../../raw/current/scripts/components/platformhopdelay.lua#L1-L4) |
| `SetDelay(time)` | `time: 秒` -> `nil` | `math.ceil(time / FRAMES)` 换算成 ticks 写入；原版无调用者，模组自定义时自行保证单位。[实现](../../../raw/current/scripts/components/platformhopdelay.lua#L6-L8) |
| `SetDelayTicks(ticks)` | `ticks: number` -> `nil` | 直接写 ticks，绕过换算。[实现](../../../raw/current/scripts/components/platformhopdelay.lua#L10-L12) |
| `GetDelayTicks()` | `-> number` | locomotor 唯一消费点，见消费表达式。[消费](../../../raw/current/scripts/components/locomotor.lua#L1283-L1285) |

## 典型接入

### 给自定义平台配置不同于全局的跳跃延迟

- 前提与选择：平台 prefab 需在服务端构造组件；如需客户端可读（本组件消费全在服务端 locomotor），原版做法是在 `SetPristine()` 前挂载。[原版挂载](../../../raw/current/scripts/prefabs/yotd_boats.lua#L594-L598)
- 生效与撤销：locomotor 的 `hop_delay` 流程里，非水面移动时取 `max(目标平台, 当前平台)` 的 GetDelayTicks，大于 0 才用平台值，否则回退 `forced_platformhopdelay` 或 `TUNING.PLATFORM_HOP_DELAY_TICKS`（8）。移除组件即回到全局默认。[消费](../../../raw/current/scripts/components/locomotor.lua#L1276-L1288) [默认值](../../../raw/current/scripts/tuning.lua#L3651-L3652)
- 原版依据：仅 `dragonboat_body`（YotD 龙舟）挂载；组件无任何 SetDelay/SetDelayTicks 的原版调用，龙舟实际使用默认 8 ticks。

## 常见陷阱与限制

- **以为 SetDelay 会同步到客户端：** 组件是纯 Lua 状态，无 netvar；跳上无物理平台的延迟计算等分支同样只读服务端值。需要玩家侧差异时用 `forced_platformhopdelay`（客户端授权设置通道）。[玩家设置](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L1054-L1062)
- **混淆水中漂浮延迟与平台延迟：** `is_water` 分支直接取 `forced_platformhopdelay or TUNING.PLATFORM_FLOATING_HOP_DELAY_TICKS`（4），完全跳过本组件；给水面目标调组件不生效。[分支](../../../raw/current/scripts/components/locomotor.lua#L1279-L1281) [浮水默认](../../../raw/current/scripts/tuning.lua#L3652)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（18 行），核对 dragonboat 挂载顺序、locomotor hop_delay 消费表达式、两个 TUNING 默认值与 forced_platformhopdelay 写入。未逐行审阅 locomotor 延迟状态机的完整推进逻辑。
- 结构检查：2026-10-28 validate_page.py 通过（platformhopdelay.md）；validate_source_links.py --strict 本页全部源码锚点通过；本页无完整 Lua 示例。
- 游戏运行：未运行；跳跃延迟表现与玩家设置覆盖未实测。
