---
title: "yoth_hecklermanager"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "马年活动（YOTH）吐槽鸟（charlie_heckler）落点调度器：全场单名额把吐槽鸟预约到激活的骑士神殿，用世界定时器控制停留与回归，舞台演出打断落鸟，playbill 状态随存档保留。"
aliases: ["yoth_hecklermanager", "吐槽鸟管理器", "heckler 调度器", "马年吐槽鸟"]
tags: [component, callback, event]
dst_build: "steam-buildid:26230496"
sources: ["components/yoth_hecklermanager.lua", "prefabs/forest.lua", "prefabs/yoth_knightshrine.lua", "prefabs/charlie_stage.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# yoth_hecklermanager

## 快速判断

- 职责与适用：挂在 forest 世界实体的服务端组件（构造断言 mastersim）；全场只维护一个 `_hecklerreservation`（当前落鸟的神殿），`timerdone` 世界定时器驱动"停留 LEAVE→飞走→回归 RETURN→再落"循环；[[reference/components/yoth_knightmanager]] 提供激活神殿清单。[构造与状态](../../../raw/current/scripts/components/yoth_hecklermanager.lua#L29-L31) [定时器分派](../../../raw/current/scripts/components/yoth_hecklermanager.lua#L96-L104)
- 优先使用：神殿侧请求落鸟／驱逐 → `TryHecklerLand(shrine)`／`TryHecklerFlyAway(shrine, line)`；查询 → `IsHecklerAvailable()`／`ShrineHasHeckler(shrine)`。[公开接口](../../../raw/current/scripts/components/yoth_hecklermanager.lua#L127-L141)
- 重要副作用：落鸟启动 LEAVE 计时（基准＋方差调谐）、飞走启动 RETURN 计时；未发过 playbill 时回归时间压到 2.5-3.5 秒（引导玩家看剧）；飞走台词按烧毁／被锤／普通三档从 `STRINGS.HECKLERS_YOTH` 随机。[计时](../../../raw/current/scripts/components/yoth_hecklermanager.lua#L37-L46) [飞走](../../../raw/current/scripts/components/yoth_hecklermanager.lua#L170-L186)
- 修改边界：舞台（charlie_stage）注册后，开演即驱逐已落鸟、散场 3 秒后尝试落回；多舞台注册时旧舞台监听被整体替换，不支持同时多剧。[舞台联动](../../../raw/current/scripts/components/yoth_hecklermanager.lua#L85-L94) [开演驱逐](../../../raw/current/scripts/components/yoth_hecklermanager.lua#L67-L76)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `IsHecklerAvailable()` | 返回 boolean | 无预约即有鸟可落。([yoth_hecklermanager.lua:127-129](../../../raw/current/scripts/components/yoth_hecklermanager.lua#L127-L129) |
| `ShrineHasHeckler(shrine)` | 返回 boolean | 该神殿是否为当前预约对象；神殿台词函数以它守卫。([yoth_hecklermanager.lua:131-133](../../../raw/current/scripts/components/yoth_hecklermanager.lua#L131-L133) |
| `CanHecklerLand()` | 返回 boolean | 舞台正在演戏（`play_in_progress`）为假；[[reference/components/yoth_knightmanager]] 的 `IsKnightShrineActive` 为真；无预约且 RETURN 定时器不存在。([yoth_hecklermanager.lua:147-157](../../../raw/current/scripts/components/yoth_hecklermanager.lua#L147-L157) |
| `TryHecklerLand(shrine)` | 返回 boolean | 过门禁后预约神殿、启动 LEAVE 定时器、对 `shrine.heckler` 推 `arrive`（即时事件）。([yoth_hecklermanager.lua:159-168](../../../raw/current/scripts/components/yoth_hecklermanager.lua#L159-L168) |
| `TryHecklerFlyAway(shrine, overrideleaveline)` | 返回 boolean | 仅对当前预约对象生效；选台词（`SHRINE_BURN`／`SHRINE_HIT`／`SHRINE_LEAVE` 或强制行）、停 LEAVE 启 RETURN、推 `leave` 并解除预约。([yoth_hecklermanager.lua:170-186](../../../raw/current/scripts/components/yoth_hecklermanager.lua#L170-L186) |
| `HasGivenPlaybill()`／`SetPlaybillGiven()` | 读写布尔 | heckler SG 送 playbill 后置位；影响 `GetHecklerReturnTime` 走正常调谐而非 2.5-3.5 秒引导值。([yoth_hecklermanager.lua:188-194](../../../raw/current/scripts/components/yoth_hecklermanager.lua#L188-L194) [SG 消费](../../../raw/current/scripts/stategraphs/SGcharlie_heckler.lua#L222-L226) |
| 定时器名 | 世界 timer 组件 | `heckler_leave`（停留）／`heckler_return`（回归）；`HecklerReturnTimerExists` 查询前者防重复落鸟。[常量](../../../raw/current/scripts/components/yoth_hecklermanager.lua#L13-L17) [查询](../../../raw/current/scripts/components/yoth_hecklermanager.lua#L143-L145) |
| `OnSave`／`OnLoad` | 存档钩子 | 仅持久化 `playbillgiven`；预约与定时器由神殿／舞台重新触发恢复。([yoth_hecklermanager.lua:200-214](../../../raw/current/scripts/components/yoth_hecklermanager.lua#L200-L214) |

## 典型接入

### 让自定义"舞台／神殿"参与吐槽鸟演出

- 前提与选择：宿主为 forest 世界；自定义神殿需有 `heckler` 字段（落下的鸟实体）、可说话的 talker，并在被破坏时主动调 `TryHecklerFlyAway`；自定义舞台需推 `ms_register_charlie_stage` 并广播 `play_begun`／`play_ended`。
- 生效与撤销：预约只存组件内存，实体移除／定时器到点自动解除；RETURN 计时存在期间其他神殿的落鸟请求都会被拒。
- 原版依据：舞台注册 [charlie_stage.lua:637](../../../raw/current/scripts/prefabs/charlie_stage.lua#L637)；神殿侧触发与驱逐 ([yoth_knightshrine.lua:19-36](../../../raw/current/scripts/prefabs/yoth_knightshrine.lua#L19-L36)。

```lua
-- 服务端，自定义神殿想让 heckler 落到自己身上：
local hm = TheWorld.components.yoth_hecklermanager
if hm ~= nil and hm:CanHecklerLand() then
    hm:TryHecklerLand(self)     -- self.heckler 需已就位
end
-- 神殿被烧/被锤时：
-- hm:TryHecklerFlyAway(self)  -- 自动选 SHRINE_BURN/SHRINE_HIT 台词
```

- 调整与验证：停留／回归时长由 `TUNING.YOTH_HECKLER_SHRINE_*` 四键控制；用 `GetDebugString()` 查 playbill 与当前预约神殿。

## 常见陷阱与限制

- 单名额全局预约：`_hecklerreservation` 无实体监听，神殿被移除而不调 `TryHecklerFlyAway` 会让名额悬挂（LEAVE 定时器到点会兜底飞走，([yoth_hecklermanager.lua:99-103](../../../raw/current/scripts/components/yoth_hecklermanager.lua#L99-L103)）。
- `shrine.heckler` 直接解引用：预约/飞走都读 `shrine.heckler` 的 talker 与 SG（([yoth_hecklermanager.lua:163-180](../../../raw/current/scripts/components/yoth_hecklermanager.lua#L163-L180)），自定义神殿漏建该字段会在定时器到点时报错。
- 台词表三键缺一不可：`TryHecklerFlyAway` 默认分支假定 `STRINGS.HECKLERS_YOTH` 下 `SHRINE_LEAVE`／`SHRINE_BURN`／`SHRINE_HIT` 存在（([yoth_hecklermanager.lua:172-177](../../../raw/current/scripts/components/yoth_hecklermanager.lua#L172-L177)），复用时需补全或传 `overrideleaveline`。
- playbill 只置位不清理：无重置接口，测试引导流程需重建世界或手动改内存值（([yoth_hecklermanager.lua:188-194](../../../raw/current/scripts/components/yoth_hecklermanager.lua#L188-L194)）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，通读组件全文；核验 forest.lua 挂载、charlie_stage 注册与事件、yoth_knightshrine 调用侧与 SGcharlie_heckler 的 playbill 消费。
- 游戏运行：未运行。
- 结构检查：未执行。

相关链接：[[reference/components/yoth_knightmanager]]、[[reference/components/timer]]、[[reference/components/talker]]、[[reference/components/prototyper]]
