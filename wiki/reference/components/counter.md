---
title: "counter"
created: "2026-10-21"
updated: "2026-10-28"
type: "component"
summary: "在实体上保存多个具名数值计数，供其他组件或 AI 主动读取，并随实体存档持久化。"
aliases: ["计数器", "具名计数", "GetCount", "DoDelta", "IncrementToZero", "salty"]
tags: ["component", "data"]
dst_build: "steam-buildid:26230496"
sources: ["components/counter.lua", "prefabs/player_common.lua", "prefabs/wilson.lua", "prefabs/polly_rogers.lua", "brains/pollyrogerbrain.lua", "prefabs/wx78_shadowdrone_debuffer.lua", "prefabs/wx78_shadowdrone_harvester.lua", "tuning.lua"]
reliability: "verified"
validation: "source-reviewed"
status: "active"
---

# counter

## 快速判断

- 职责与适用：给实体挂一个 `名字 -> 数值` 的表，按名字分桶保存短周期计数；缺省读作 0，不做类型、上下限或用途校验。[counter.lua:1-18](../../../raw/current/scripts/components/counter.lua#L1-L18)
- 优先使用：需要“按名累计／递减”的状态，如原版 `salty_dog` 的盐值、WX-78 暗影无人机的互动标记；玩家宿主由 `player_common` 统一添加。[polly_rogers.lua:226-229](../../../raw/current/scripts/prefabs/polly_rogers.lua#L226-L229) [wx78_shadowdrone_debuffer.lua:140-142](../../../raw/current/scripts/prefabs/wx78_shadowdrone_debuffer.lua#L140-L142) [player_common.lua:2849](../../../raw/current/scripts/prefabs/player_common.lua#L2849
- 重要副作用：不推送也不监听事件，没有 replica、没有网络字段；数值变化不会自动通知 UI、AI 或其他组件，消费者只能在需要时主动读。[counter.lua:8-56](../../../raw/current/scripts/components/counter.lua#L8-L56)
- 修改边界：只在服务端有意义；数据随实体存档（`OnSave`／`OnLoad`），但临时计数要显式 `DoNotSave`。移除组件即丢弃整张表，没有迁移或清理回调。[counter.lua:58-85](../../../raw/current/scripts/components/counter.lua#L58-L85)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `GetCount(name)` | number，缺省 0 | 直接读表；名字写错不会报错，只会得到 0。[counter.lua:8-10](../../../raw/current/scripts/components/counter.lua#L8-L10) |
| `Set(name, value)` | 无返回值 | 原样写入，包括 0 与负数；不会在 0 时删键。[counter.lua:12-14](../../../raw/current/scripts/components/counter.lua#L12-L14) |
| `Clear(name)` | 无返回值 | 删除该名字的键，之后 `GetCount` 回到 0。[counter.lua:16-18](../../../raw/current/scripts/components/counter.lua#L16-L18) |
| `DoDelta(name, delta)` | 无返回值 | 先加后写；结果恰为 0 时删键，否则写回。[counter.lua:20-30](../../../raw/current/scripts/components/counter.lua#L20-L30) |
| `Increment(name, magnitude?)`／`Decrement(name, magnitude?)` | magnitude 缺省 1 | 分别转调 `DoDelta(+n)`／`DoDelta(-n)`。[counter.lua:32-40](../../../raw/current/scripts/components/counter.lua#L32-L40) |
| `IncrementToZero(name, magnitude?)`／`DecrementToZero(name, magnitude?)` | magnitude 缺省 1，按正数使用 | 仅在计数为负／正时朝 0 走，单次最多到 0，不越界。[counter.lua:42-56](../../../raw/current/scripts/components/counter.lua#L42-L56) |
| `DoNotSave(name)` | 单向，无撤销接口 | 标记该名字为临时计数，`OnSave` 时过滤；原版快照内没有调用者。[counter.lua:58-60](../../../raw/current/scripts/components/counter.lua#L58-L60) |
| `OnSave()`／`OnLoad(data)` | 存档表／无返回值 | 全空时返回 nil；否则返回 `{counters = ...}`；有 `data.counters` 时载入整表替换而非合并。[counter.lua:62-85](../../../raw/current/scripts/components/counter.lua#L62-L85) |
| `GetDebugString()` | string 或 nil | 空表返回 nil，否则按名字排序输出 `"名字 : 数值"` 行。[counter.lua:87-103](../../../raw/current/scripts/components/counter.lua#L87-L103) |

### 计数事件与提供者方向

- counter 是**被动提供者**：全文 105 行没有 `PushEvent`／`ListenForEvent`，也没有 `counter_replica.lua`；方向是消费者向 counter 拉取，不是 counter 向消费者广播。[counter.lua:1-105](../../../raw/current/scripts/components/counter.lua#L1-L105)
- 驱动计数的事件在宿主侧：`salty_dog` 监听 `timer` 的 `timerdone`，到点后 `Increment("salty", 1)` 并刷新外观；brain 在自己的决策节点里 `GetCount("salty")` 判断是否上岸抖盐。[polly_rogers.lua:168-179](../../../raw/current/scripts/prefabs/polly_rogers.lua#L168-L179) [pollyrogerbrain.lua:125-129](../../../raw/current/scripts/brains/pollyrogerbrain.lua#L125-L129)
- 需要“变化通知”时，调用方必须在写后自行 `PushEvent` 或另设网络字段；counter 不会替调用方做同步，客户端读不到服务端计数。[counter.lua:62-85](../../../raw/current/scripts/components/counter.lua#L62-L85)

## 典型接入

### salty_dog：计时事件驱动、brain 拉取（原版宿主）

- 前提与选择：`prefabs/polly_rogers.lua` 注册 `polly_rogers` 与 `salty_dog`；`salty_dog` 的 master sim 先加 `counter` 再加 `timer`，并监听 `timerdone`。[polly_rogers.lua:226-229](../../../raw/current/scripts/prefabs/polly_rogers.lua#L226-L229) [polly_rogers.lua:252-253](../../../raw/current/scripts/prefabs/polly_rogers.lua#L252-L253)
- 计数规则：每次计时到期，若小于 `TUNING.SALTY_DOG_MAX_SALT_COUNT`（本快照为 2）则 `Increment("salty", 1)`；单位是“次数”，不是时间。[polly_rogers.lua:168-176](../../../raw/current/scripts/prefabs/polly_rogers.lua#L168-L176) [tuning.lua:9128](../../../raw/current/scripts/tuning.lua#L9128
- 消费与清理：外观按计数重映射到 0～2 阶；抖盐用 `DecrementToZero("salty")`，全部抖落用 `Clear("salty")`；brain 只读，不写。[polly_rogers.lua:123-129](../../../raw/current/scripts/prefabs/polly_rogers.lua#L123-L129) [polly_rogers.lua:137-149](../../../raw/current/scripts/prefabs/polly_rogers.lua#L137-L149) [polly_rogers.lua:155-158](../../../raw/current/scripts/prefabs/polly_rogers.lua#L155-L158)
- 生效与撤销：服务端加组件即生效；`RemoveComponent("counter")` 后所有读取都回到 0，没有清理钩子，宿主应先处理依赖计数的逻辑。

### WX-78 暗影无人机：一次性标记（原版宿主）

- 注册名 `wx78_shadowdrone_debuffer`／`wx78_shadowdrone_harvester`，均在 master sim 加 `counter`，用 `Set("interactions", 1)` 标记“已被使用”，掉落逻辑用 `GetCount("interactions") == 0` 判断是否全掉。[wx78_shadowdrone_debuffer.lua:129](../../../raw/current/scripts/prefabs/wx78_shadowdrone_debuffer.lua#L129 [wx78_shadowdrone_debuffer.lua:140-142](../../../raw/current/scripts/prefabs/wx78_shadowdrone_debuffer.lua#L140-L142) [wx78_shadowdrone_debuffer.lua:276](../../../raw/current/scripts/prefabs/wx78_shadowdrone_debuffer.lua#L276 [wx78_shadowdrone_harvester.lua:183](../../../raw/current/scripts/prefabs/wx78_shadowdrone_harvester.lua#L183
- 这里用 `Set` 而不是 `Increment`，因为语义是布尔标记；计数名就是协议，读写双方必须共用同一字符串。

### 玩家宿主：统一挂载点

- `player_common.lua` 的 `MakePlayerCharacter` 在所有角色共用的 master sim 段执行 `inst:AddComponent("counter")`；各角色 prefab（如 `wilson`）把自己的 `master_postinit` 传给 `MakePlayerCharacter`，因此每个玩家都有该组件。[player_common.lua:1968](../../../raw/current/scripts/prefabs/player_common.lua#L1968 [player_common.lua:2849](../../../raw/current/scripts/prefabs/player_common.lua#L2849 [wilson.lua:170](../../../raw/current/scripts/prefabs/wilson.lua#L170
- 本快照的 `scripts/` 内没有读取玩家 `components.counter` 的原版消费者，它是扩展示意入口；若模组要显示给客户端，必须自己做 replica 或网络字段。

### 最小接线骨架

```lua
-- 目标：实体的 master_postinit（服务端）
inst:AddComponent("counter")
inst.components.counter:Increment("charge", 1)
local n = inst.components.counter:GetCount("charge") -- 缺省 0
inst.components.counter:DoNotSave("charge")          -- 仅当这是临时计数
```

- 调整与验证：骨架只演示读取方向，未包含事件与 UI 同步；单位由调用方定义，counter 不限制范围。[counter.lua:32-40](../../../raw/current/scripts/components/counter.lua#L32-L40)

## 常见陷阱与限制

- **把它当事件源**：counter 不发事件、无 replica；客户端 `inst.components.counter` 为 nil，读不到服务端计数。需要通知或客户端显示时，自己在写入后推事件并同步。[counter.lua:1-105](../../../raw/current/scripts/components/counter.lua#L1-L105)
- **`Set(name, 0)` 与 `DoDelta` 归零不同**：`Set` 保留 0 键，`DoDelta` 归零会删键；两者 `GetCount` 都是 0，但存档／调试输出不同。[counter.lua:12-30](../../../raw/current/scripts/components/counter.lua#L12-L30)
- **`OnLoad` 整表替换**：有 `data.counters` 时 `self.counters = data.counters`，不合并；载入前已有的运行时计数会被覆盖，别在 `OnLoad` 之前依赖累计值。[counter.lua:79-85](../../../raw/current/scripts/components/counter.lua#L79-L85)
- **`DoNotSave` 是单向且不参与载入**：只影响后续 `OnSave`，不会把已存在的临时计数从旧档里剔除；原版没有调用者，临时用法要自证。[counter.lua:58-77](../../../raw/current/scripts/components/counter.lua#L58-L77)
- **名字拼写即协议**：任意字符串都合法，写错只会新建一个桶并读到 0，没有注册表或校验；读写两端必须共享常量。[counter.lua:8-14](../../../raw/current/scripts/components/counter.lua#L8-L14)
- **`ToZero` 假定正 magnitude**：传负数会反向远离 0（`math.min`／`math.max` 取到正 delta）；接口没有断言。[counter.lua:42-56](../../../raw/current/scripts/components/counter.lua#L42-L56)
- **不做数值校验**：`Set` 可以写入非数字，之后 `DoDelta` 做算术会报错；计数应统一用整数或明确的小数单位。[counter.lua:12-30](../../../raw/current/scripts/components/counter.lua#L12-L30)

## 验证记录

- 版本增量复核 26148036（2026-10-25）：锚点重定位 4 处（player_common MakePlayerCharacter -12／counter 挂载 -14、wilson 注册 -1、tuning SALTY_DOG_MAX_SALT_COUNT +17，显示标签同步更新）；无结论修订，仅锚点迁移——SALTY_DOG_MAX_SALT_COUNT=2 值未变（tuning-hunk-map 5 改值键不含），counter.lua 及 salty_dog／无人机宿主全部未变（wilson 本版仅 GetPointSpecialActions 两行改写）。依据：tuning-hunk-map 键级审阅＋变更清单命中 player_common.lua、wilson.lua、tuning.lua。
- 源码审阅：2026-10-21，steam-buildid:25643519；通读 `components/counter.lua`（105 行），核对 `player_common`／`wilson` 玩家宿主、`salty_dog` 计时事件与 brain 读取、WX-78 无人机的一次性标记，并检索全部 `counter:` 调用；正文源码链接 41 处，结论只覆盖这些路径。
- 校验：`validate_page.py` 0 error／0 warning；`validate_source_links.py --strict` 0 issue；接线骨架经 `parser_probe --syntax-only` 通过。
- 游戏运行：未运行；存档载入覆盖顺序、多人环境下客户端显示未实测。

相关：[[reference/components/container]]；[[reference/components/stackable]]；[[reference/components/inventoryitem]]。
