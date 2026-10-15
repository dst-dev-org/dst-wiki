---
title: "npc_talker"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "NPC 台词队列：Say/Chatter 入队文本与声音回调，由宿主（大脑）驱动 DoNextLine 逐行播出，支持覆盖与可被打断语义。"
aliases: ["NPC说话队列", "npc_talker"]
tags: [component, callback, ai]
dst_build: "steam-buildid:26230496"
sources: ["components/npc_talker.lua", "prefabs/wagstaff_npc.lua", "brains/hermitcrabbrain.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# npc_talker

## 快速判断

- 职责与适用：给 NPC（原版 hermitcrab、wagstaff_npc、charlie_npc、stalker）维护三条平行队列（文本/声音/回调）；`Say` 收明文或行表，`Chatter` 收 STRINGS 表路径，`DoNextLine` 播出一行。
- 优先使用：NPC 闲聊/剧情台词用 `Chatter("表路径", index)`；驱动靠大脑在空闲时轮询 `HasLines` + `DoNextLine`（[hermitcrab 大脑](../../../raw/current/scripts/brains/hermitcrabbrain.lua#L519-L520)）。
- 重要副作用：播报最终落到 `talker` 组件——`DoNextLine` 直接调 `talker:Say/Chatter`（[L112-L117](../../../raw/current/scripts/components/npc_talker.lua#L112-L117)），没有 talker 的实体不能用。
- 修改边界：队列纯内存，无存读钩子；小写别名 `haslines/resetqueue/donextline` 保留向后兼容（[L133-L135](../../../raw/current/scripts/components/npc_talker.lua#L133-L135)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `Say(lines, override, stompable, sound, cb)` | `(string\|table, bool, bool, string, fn) -> void` | override 清空重排；stompable 在已有队列时直接忽略；多行时 sound 只挂首行、cb 只挂末行。[实现](../../../raw/current/scripts/components/npc_talker.lua#L15-L51) |
| `Chatter(strtbl, index, chatpriority, override, stompable, sound, cb)` | `(string, 可选 number, ...) -> void` | 解析 `STRINGS` 点分路径；给 index 或值为单串时入队一行，否则整表按序入队；chatpriority 缺省 `default_chatpriority`。[实现](../../../raw/current/scripts/components/npc_talker.lua#L53-L93) |
| `DoNextLine` | `-> void` | 弹出一行：Line 对象（有 message 字段）走 `talker:Say({行}, speaktime)`，`{表路径, index, 优先级}` 走 `talker:Chatter`，明文走 `talker:Say`；随后播声音、执行回调。[实现](../../../raw/current/scripts/components/npc_talker.lua#L105-L130) |
| `HasLines/ResetQueue` | `-> bool / void` | 队列判空与整体清空。[实现](../../../raw/current/scripts/components/npc_talker.lua#L95-L103) |
| `default_chatpriority` | CHATPRIORITIES 值（默认 NOCHAT） | Chatter 未传优先级时使用；wagstaff_npc 设为 HIGH（[接线](../../../raw/current/scripts/prefabs/wagstaff_npc.lua#L1039-L1041)）。 |
| `speaktime` | 可选 number | 传给 talker:Say 的显示时长；wagstaff_npc 设 3.5（[L1040](../../../raw/current/scripts/prefabs/wagstaff_npc.lua#L1040)）。 |
| `stompable` | bool | 当前队尾行可被打断的标记：下一个 override/普通 Say 会清掉它。[实现](../../../raw/current/scripts/components/npc_talker.lua#L20-L23) |

## 典型接入

### 给 NPC 加序列台词

- 前提与选择：实体需先有 talker 组件（见 [[reference/components/talker]]）；台词表放进 STRINGS 后用 Chatter 入队，宿主负责驱动播出。

```lua
inst:AddComponent("talker")
local npc_talker = inst:AddComponent("npc_talker")
npc_talker.default_chatpriority = CHATPRIORITIES.HIGH
npc_talker:Chatter("MY_NPC_GREETINGS", nil) -- 不给 index 则整表按序入队
-- 驱动侧（大脑或定时任务）：
if inst.components.npc_talker:HasLines() then
    inst.components.npc_talker:DoNextLine()
end
```

- 调整与验证：wagstaff_npc 的完整接法（priority/speaktime/MakeChatter）见 [wagstaff_npc.lua](../../../raw/current/scripts/prefabs/wagstaff_npc.lua#L1035-L1041)；声音文件名作为 Say/Chatter 的 sound 参数随首行播出。

## 常见陷阱与限制

- **不驱动就不播**：组件只管入队，`DoNextLine` 必须由宿主反复调用；只在 Say 后等待不会有任何显示。
- **Chatter 路径不存在静默失败**：逐级解析 STRINGS 失败直接 return，不入队也不报错（[L66-L71](../../../raw/current/scripts/components/npc_talker.lua#L66-L71)），调试时先确认表路径。
- **stompable 不是抢占**：stompable 行在队列非空时直接被丢弃（[L25-L27](../../../raw/current/scripts/components/npc_talker.lua#L25-L27)），只有"空闲时先占位、来重要话再被覆盖"的语义。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；组件全文、wagstaff_npc 挂接参数、hermitcrabbrain 驱动方式；charlie_npc/stalker 消费点经 grep 核验。
- 游戏运行：未运行。
