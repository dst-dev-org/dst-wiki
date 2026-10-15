---
title: "施法光源（staff_castinglight、staff_castinglight_small）"
created: "2026-11-04"
updated: "2026-11-04"
type: prefab
summary: "施法、复活护符与怀表状态使用的联网光效实体：按五次幂包络扩半径、降强度，结束时关灯并短期移除。"
aliases: ["staff_castinglight", "staff_castinglight_small", "施法光源", "施法光效", "复活护符光"]
tags: [prefab, fx, server-client]
size_class: simple
dst_build: "steam-buildid:26335394"
sources: ["prefabs/staff_castinglight.lua", "stategraphs/SGwilson.lua", "stategraphs/SGklaus.lua", "prefabs/player_common.lua", "prefabs/klaus.lua", "prefablist.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 施法光源（staff_castinglight、staff_castinglight_small）

## 快速判断

- 两个注册是同一工厂的普通版与小号版，只含 Transform、Light、Network 与 `FX` 标签，没有 AnimState、声音或玩法组件；作用是在施法者脚下做一段扩张后熄灭的光晕。[工厂](../../../raw/current/scripts/prefabs/staff_castinglight.lua#L38-L68)、[注册](../../../raw/current/scripts/prefabs/staff_castinglight.lua#L78-L79)
- 颜色与时长由调用方通过 `SetUp(colour, duration, delay)` 传入；普通版与小号版共用同一更新函数，只在小号分支用更小的半径上限和不同的强度／衰减曲线。[参数分支](../../../raw/current/scripts/prefabs/staff_castinglight.lua#L1-L36)
- 标题与中文别名是已核验效果描述，非官方译名；简中 `languages/chinese_s.po` 没有这两个注册名的显示名键。

## 注册与构造

| 注册名 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `staff_castinglight` | 公共构造加 Transform、Light、Network、`FX` 标签；Light 初值半径 0.3、强度 0.8、衰减 0.9；声明 `_duration`／`_value` 两个 net_smallbyte。主模拟暴露 `SetUp` 并设 `persists = false`；客户端监听 `setupdirty`。 | 普通版半径上限 10.3、强度 0.8→0.2、衰减 0.9→0.5。[构造](../../../raw/current/scripts/prefabs/staff_castinglight.lua#L38-L68)、[曲线](../../../raw/current/scripts/prefabs/staff_castinglight.lua#L12-L14) |
| `staff_castinglight_small` | 同一 `common_fn(true)` | 小号版半径上限 4.3、强度 0.8→0.1、衰减 1.1→1.4。[注册](../../../raw/current/scripts/prefabs/staff_castinglight.lua#L70-L79)、[曲线](../../../raw/current/scripts/prefabs/staff_castinglight.lua#L8-L10) |

`SetUp` 以 `floor(duration / FRAMES + 0.5)` 换算帧数写入 `_duration`（会触发客户端 dirty），再按 `delay` 排每帧任务；`OnUpdate` 用 `set_local` 递增 `_value`，`k = (value / duration)^5` 驱动半径、强度与衰减。达到时长后关灯、取消任务，主模拟再等 2 帧移除实体。[设置](../../../raw/current/scripts/prefabs/staff_castinglight.lua#L31-L36)、[更新与结束](../../../raw/current/scripts/prefabs/staff_castinglight.lua#L1-L24)

## 关联机制导航

| 调用状态／宿主 | 用法与参数 | 证据 |
|---|---|---|
| 玩家 `castspell` | 生成普通版，放到玩家世界位置，颜色取手持物 `fxcolour`，时长 1.9、延迟 0.33。 | [施法光源](../../../raw/current/scripts/stategraphs/SGwilson.lua#L16194-L16196) |
| 玩家 `cointosscastspell` | 生成普通版，颜色取抛币物品，时长 1.2、延迟 0.33。 | [抛币光源](../../../raw/current/scripts/stategraphs/SGwilson.lua#L16387-L16391) |
| 玩家动态法术状态 | 按 `lightcolour` 生成普通版，时长 1.2、延迟 0.33；同状态的实体特效按骑乘拼 `_mount` 名。 | [法术光源](../../../raw/current/scripts/stategraphs/SGwilson.lua#L16955-L16966) |
| 玩家 `pocketwatch_cast` | 只在怀表施法特效已生成时生成小号版，颜色取 `castfxcolour`，时长 0.75、无延迟。 | [怀表光源](../../../raw/current/scripts/stategraphs/SGwilson.lua#L20489-L20494) |
| 玩家复活护符 | 生成普通版，固定红色 `{150/255, 46/255, 46/255}`，时长 1.7、延迟 1。 | [护符光源](../../../raw/current/scripts/stategraphs/SGwilson.lua#L15243-L15247) |
| Klaus `resurrect` | 生成普通版，同一红色，时长 1、延迟 20 帧。 | [Klaus 光源](../../../raw/current/scripts/stategraphs/SGklaus.lua#L369-L373) |

`prefablist.lua`、`player_common` 与 `klaus.lua` 的 `prefabs` 表只是预载声明，不构成生成点。[预载清单](../../../raw/current/scripts/prefabs/player_common.lua#L2206-L2207)、[Klaus 预载](../../../raw/current/scripts/prefabs/klaus.lua#L15-L15)

## 常见陷阱与限制

- 它不是闪光弹：`SetUp` 之后光半径按五次幂从 0.3 扩到上限、强度同时下降，时长走完才关灯，主模拟再等 2 帧移除；不要按“生成后立即 Remove”的闪光模式复用。[结束分支](../../../raw/current/scripts/prefabs/staff_castinglight.lua#L17-L23)
- 没调用 `SetUp` 就不会建立每帧任务：实体保持默认小半径光且不会自行结束（主模拟只设 `persists=false`），调用方必须给颜色与时长。[设置入口](../../../raw/current/scripts/prefabs/staff_castinglight.lua#L31-L36)
- 小号版不是按父实体自动选择，而由调用方选名；两者共用 `staff_castinglight` 的 netvar 名，但颜色和时长仍各自由 `SetUp` 覆写。

## 验证记录

- 源码审阅：2026-11-04，Steam buildid 26335394；完整读取 `staff_castinglight.lua`，核对两个注册、netvar、每帧更新与两条清理路径；全文检索 `staff_castinglight` 得到六处状态生成点与两处预载声明，区分生成与依赖。未解包动画（本实体无 AnimState），未运行游戏验证实际视觉效果与网络时序。
- 游戏运行：未运行。

返回 [[reference/prefabs/fx-character-index|角色技能、施法与读书]]。
