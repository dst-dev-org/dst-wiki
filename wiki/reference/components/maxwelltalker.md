---
title: "maxwelltalker"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "在实体上播放剧本化台词过场（出现／对白／消失动画、语音、镜头控制、可点击跳过）的演出组件；DST 原版无挂载点，属单机冒险模式遗留代码。"
aliases: ["maxwelltalker", "麦克斯韦台词演出", "Maxwell 演说组件", "TALKTO 过场"]
tags: [component, callback, ui]
dst_build: "steam-buildid:26230496"
sources: ["components/maxwelltalker.lua", "actions.lua", "componentactions.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# maxwelltalker

## 快速判断

- 职责与适用：按 `speeches` 数据表在宿主实体上播放一整套 Maxwell 式过场：出现动画、逐句对白（经由 [[reference/components/talker]]）、语音与消失动画；`disableplayer` 演出还会接管镜头并禁用玩家操作，点击可跳过。[构造](../../../raw/current/scripts/components/maxwelltalker.lua#L9-L27) [对白主循环](../../../raw/current/scripts/components/maxwelltalker.lua#L121-L178)
- 优先使用：想复刻"演员入场→念台词→退场"的过场 → `SetSpeech(key)` 选定剧本后 `Initialize()`／`DoTalk()`，跳过由 `OnClick` 处理。[选剧本](../../../raw/current/scripts/components/maxwelltalker.lua#L211-L213) [初始化](../../../raw/current/scripts/components/maxwelltalker.lua#L94-L115)
- 重要副作用：构造即给宿主挂 `maxwellnottalking` tag 并随 `speech` 字段增删（[tag 监听](../../../raw/current/scripts/components/maxwelltalker.lua#L1-L7)）；`disableplayer` 时隐藏 HUD、禁用 `playercontroller`、把玩家推进 `sleep` 状态并在结束时唤醒。[接管玩家](../../../raw/current/scripts/components/maxwelltalker.lua#L97-L114)
- 修改边界：DST 全库无 `AddComponent("maxwelltalker")`，唯一读取点 `ACTIONS.TALKTO`（([actions.lua:1537-1541](../../../raw/current/scripts/actions.lua#L1537-L1541)）还要求目标带 `talkable` 组件，而原版脚本同样无该组件挂载（[动作门控](../../../raw/current/scripts/componentactions.lua#L872-L876)）；整条链路在本快照处于休眠，直接启用需自行补齐组件、剧本数据与触发。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| 构造 | `(inst)` | 无 mastersim 断言、无网络变量；注册 CONTROL_PRIMARY／SECONDARY／ATTACK／INSPECT／ACTION／CONTROLLER_ACTION 六个输入处理器，任一命中即 `OnClick` 跳过。([maxwelltalker.lua:9-27](../../../raw/current/scripts/components/maxwelltalker.lua#L9-L27) |
| `SetSpeech(speech)` | 剧本键名 | 仅非空时覆盖 `self.speech`，真正的表在 `self.speeches[key]`。([maxwelltalker.lua:211-213](../../../raw/current/scripts/components/maxwelltalker.lua#L211-L213) |
| `Initialize()` | 无参 | 解析 `inst.speech = speeches[speech or "NULL_SPEECH"]`；`disableplayer` 时接管镜头、把 Maxwell 定位到玩家侧后方并隐藏。([maxwelltalker.lua:94-115](../../../raw/current/scripts/components/maxwelltalker.lua#L94-L115) |
| `DoTalk()` | 无参 | 演出主循环：`delay`→出现动画→逐段播放 `anim`／`string`（`talker:Say`＋`voice` 或默认 `dontstarve/maxwell/talk_LP`）／`sound`→`disappearanim` 退场；结束移除全部输入处理器。内含 `Sleep`，须在协程线程调用。([maxwelltalker.lua:121-209](../../../raw/current/scripts/components/maxwelltalker.lua#L121-L209) |
| `IsTalking()` | 返回 boolean | `inst.speech ~= nil`；TALKTO 动作用它防重入。([maxwelltalker.lua:117-119](../../../raw/current/scripts/components/maxwelltalker.lua#L117-L119) |
| `StopTalking()`／`OnCancel()` | 无参 | 停对白／跳过后清理：`talker:ShutUp`、唤醒玩家、恢复 HUD 与镜头、播消失动画并在 `animqueueover` 后 `Remove()`。([maxwelltalker.lua:81-92](../../../raw/current/scripts/components/maxwelltalker.lua#L81-L92) [OnCancel](../../../raw/current/scripts/components/maxwelltalker.lua#L33-L68) |
| speech 数据表 | 每段 `{anim, string, sound, wait, waitbetweenlines}` | 表级字段 `delay`、`appearanim`、`idleanim`、`dialogpreanim`／`dialoganim`／`dialogpostanim`、`disappearanim`、`voice`、`disableplayer`；字段读取散布于 DoTalk 与 OnCancel，无集中 schema。[消费点](../../../raw/current/scripts/components/maxwelltalker.lua#L125-L177) |

## 常见陷阱与限制

- 触发链路休眠：`talkable` 组件动作条目（([componentactions.lua:872-876](../../../raw/current/scripts/componentactions.lua#L872-L876)）只在宿主有 `talkable` 组件且带 `maxwellnottalking` tag 时插入 TALKTO，但本快照无任何脚本调用 `AddComponent("talkable")`；只挂 maxwelltalker 不会出现任何动作。
- `DoTalk` 内多处 `Sleep`（[如 L128](../../../raw/current/scripts/components/maxwelltalker.lua#L128-L129)），必须像原版动作一样用 `StartThread` 包裹（[actions.lua:1540](../../../raw/current/scripts/actions.lua#L1540），在普通回调里直接调用会阻塞或报错。
- 输入处理器只在演出结束路径移除（[L206-208](../../../raw/current/scripts/components/maxwelltalker.lua#L206-L208)）；`OnRemoveFromEntity` 仅清 tag（[L29-31](../../../raw/current/scripts/components/maxwelltalker.lua#L29-L31)），未播完就移除组件会遗留输入监听。
- 依赖 `GetPlayer()`、`SaveGameIndex:GetCurrentMode` 等单机时代全局（[L65](../../../raw/current/scripts/components/maxwelltalker.lua#L65-L66)、[L43](../../../raw/current/scripts/components/maxwelltalker.lua#L43-L44)），无网络同步；专用服务器与分片语境下不可用。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，通读 components/maxwelltalker.lua 全文；消费者与"无 AddComponent 挂载"结论经全库 grep 核验（含 actions.lua、componentactions.lua、prefablist.lua）。
- 游戏运行：未运行。
- 结构检查：未执行。

相关链接：[[reference/components/talker]]、[[reference/components/npc_talker]]
