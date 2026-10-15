---
title: "minigame_spectator"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "把生物标记为某小游戏的观众：登记正在观看的 minigame 实体，小游戏结束或移除时组件自拆。"
aliases: ["小游戏观众", "观战标记", "minigame_spectator"]
tags: [component, lifecycle]
dst_build: "steam-buildid:26230496"
size_class: simple
sources: ["components/minigame_spectator.lua", "components/minigame.lua", "brains/pigbrain.lua", "brains/carnival_crowkidbrain.lua", "brains/carnival_hostbrain.lua", "brains/cozy_bunnymanbrain.lua", "brains/carnivalgame_herding_chick_brain.lua", "stategraphs/SGcarnival_crowkid.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# minigame_spectator

## 快速判断

- 职责与适用：观众的登记组件：持有 `minigame` 实体引用，监听其 `onremove`／`ms_minigamedeactivated`，任一发生就 `RemoveComponent("minigame_spectator")` 自拆（[L8](../../../raw/current/scripts/components/minigame_spectator.lua#L8)、[L11-L16](../../../raw/current/scripts/components/minigame_spectator.lua#L11-L16)）。没有对应 prefab 静态添加：由 `Minigame:AddSpectator` 在运行时动态加（[minigame.lua#L74-L79](../../../raw/current/scripts/components/minigame.lua#L74-L79)）。
- 优先使用：调用方侧用 `Minigame:AddSpectator(inst)`；AI／表现侧用 `GetMinigame()` 取目标小游戏。
- 重要副作用：`SetWatchingMinigame` 顺带把"正在攻击玩家"的战斗目标丢弃（[L24-L27](../../../raw/current/scripts/components/minigame_spectator.lua#L24-L27)）；观众是否随 minigame 的活跃脉冲被动态征集，由 minigame 组件扫描（[minigame.lua#L89-L99](../../../raw/current/scripts/components/minigame.lua#L89-L99)）。
- 修改边界：一对一起作用——`minigame ~= nil` 就拒绝换目标（[L19](../../../raw/current/scripts/components/minigame_spectator.lua#L19)）；不存档（组件是运行期标记，读档后由 minigame 脉冲重新征集）。

## 关键接口

| 接口／字段 | 已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SetWatchingMinigame(minigame)` | 无返回 | 只在无目标时生效：登记并挂两个事件监听（挂在 minigame 实体上）。[L18-L29](../../../raw/current/scripts/components/minigame_spectator.lua#L18-L29) |
| `GetMinigame()` | 实体或 nil | 返回当前观看的小游戏。[L31-L33](../../../raw/current/scripts/components/minigame_spectator.lua#L31-L33) |
| `_good_ending` | boolean 或 nil（外部写入字段） | 组件定义之外的字段：乌鸦小子 AI 结算时写入，SG 读取后决定欢呼／喝倒彩状态（[写](../../../raw/current/scripts/brains/carnival_crowkidbrain.lua#L207)、[读](../../../raw/current/scripts/stategraphs/SGcarnival_crowkid.lua#L13)）。 |

## 典型接入

### 原版征集与消费链

- 征集：`Minigame:DoActivePulse` 周期扫描范围内带 `character` tag、没有领队的实体并 `AddSpectator`（[minigame.lua#L89-L99](../../../raw/current/scripts/components/minigame.lua#L89-L99)）；`SPECTATOR_CANT_TAGS` 排除 monster 与 player。
- 消费：猪人／狂欢节乌鸦小子／狂欢节主持人／兔人的 brain 都用 `GetMinigame()` 取小游戏的 `watchdist_min`／`watchdist_target`／`watchdist_max` 决定站位（[pigbrain.lua#L214-L224](../../../raw/current/scripts/brains/pigbrain.lua#L214-L224)、[carnival_hostbrain.lua#L43-L94](../../../raw/current/scripts/brains/carnival_hostbrain.lua#L43-L94)）；赶鸡小游戏把该 tag 列进逃跑参数（[carnivalgame_herding_chick_brain.lua#L16](../../../raw/current/scripts/brains/carnivalgame_herding_chick_brain.lua#L16)）；SG 结算读 `_good_ending` 并监听 `minigame_spectator_start_outro`（[SGcarnival_crowkid.lua#L13](../../../raw/current/scripts/stategraphs/SGcarnival_crowkid.lua#L13)、[L50](../../../raw/current/scripts/stategraphs/SGcarnival_crowkid.lua#L50)）。

## 常见陷阱与限制

- 事件监听挂在 minigame 实体上：自拆前 `OnRemoveFromEntity` 会摘除监听，直接换 minigame 前必须先 RemoveComponent，否则旧监听残留。
- `_good_ending` 与 `minigame_spectator_start_outro` 是"组件为载体"的脑／SG 约定，不属于组件逻辑：新小游戏要复用观众体系就得自己写这层结算，组件本身只负责"正在看＋自动清理"。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。完整阅读 components/minigame_spectator.lua（39 行）与 minigame.lua 的征集段，抽查四个 brain 与 SGcarnival_crowkid 的消费点。未覆盖具体小游戏的玩法结算。
- 游戏运行：未运行。
- 结构检查：validate_page.py 通过（0 issue；50 行、4886 字节，simple 档内）；validate_source_links.py --strict 通过（16 处源码引用，0 error／0 warning）；本页无完整 Lua 代码块。
