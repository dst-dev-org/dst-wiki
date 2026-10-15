---
title: "wereeater"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "Woodie 专属进食变身计数组件：吃 monstermeat 累计计数（≥2）或直接吃 wereitem 即触发强制变身回调，计数按半日时长逐个遗忘并推 wereeaterchanged 事件。"
aliases: ["伍迪进食变身", "怪物食物变身计数", "wereeater"]
tags: [component, lifecycle]
dst_build: "steam-buildid:26230496"
size_class: simple
sources: ["components/wereeater.lua", "prefabs/woodie.lua", "components/sentientaxe.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# wereeater

## 快速判断

- 职责与适用：仅 Woodie 挂载并注册变身回调（prefabs/woodie.lua:1824-1825 `SetForceTransformFn(OnForceTransform)`）；监听自身 `oneat`（components/wereeater.lua:12）。
- 优先使用：调整"吃几块怪物肉变身"→改 :64 的硬编码阈值 2 或 `duration`（默认 `TUNING.TOTAL_DAY_TIME/2`，:7）；变身执行完全交给 forcetransformfn。
- 重要副作用：PushEvent `"wereeaterchanged" {old, new, istransforming}`（:46-50、:72-76）；露西斧 sentientaxe 监听该事件联动（components/sentientaxe.lua:30、:78）。
- 修改边界：已变身（`wereplayer` tag）时直接返回（:54-57）；OnSave 仅在计数>0 且任务存活时保存（:84-92）。

## 关键接口

| 接口/字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `EatMosterFood` | `(data) -> nil` | food 带 `wereitem` tag→立即变身（mode 取 `food.were_mode`）；带 `monstermeat` tag→计数+1、≥2 变身并重置遗忘任务。:53-82 |
| `ForceTransformToWere` | `(mode\|nil) -> nil` | 仅转发 forcetransformfn(inst, mode)；未注册则无操作。:31-35 |
| `ResetFoodMemory` | `() -> nil` | 清零计数、取消任务；Woodie 各变形态复位时调用（woodie.lua:1224、:1280、:1338、:1396、:1511）。:23-29 |
| `OnSave`/`OnLoad` | 常规存取 | 保存计数与任务剩余时间；载入按剩余时长重建任务。:84-102 |

## 典型接入

### 原版参照：Woodie 变身链路

- 前提与选择：组件只管"吃什么、吃几块"，是否真变身由 woodie.lua 的 OnForceTransform 决定；两个系统经 forcetransformfn 与 wereeaterchanged 事件解耦。
- 生效与撤销：计数遗忘是逐个过期——`OnMonsterCooldown` 每 duration 递减 1（:37-51）；连续进食会 Cancel 旧任务重新计时，不因停吃立即清零。
- 无 mod 接入案例，不提供代码。

## 常见陷阱与限制

- 变身阈值 2 与"wereitem 立即变身"均硬编码（:59-66），非 tuning 项。
- OnLoad 缺 `data ~= nil` 守卫（:94-95 直接 `data.monster_count`），依赖引擎"无存档数据不调 OnLoad"的行为，移植注意。
- 计数只在 monstermeat tag 分支维护，吃其它怪物食品（如榴莲）不累计。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，通读组件全部 108 行，核验挂载与回调注册（woodie.lua:1824-1825）、事件消费方（sentientaxe.lua:30/78）及 5 处 ResetFoodMemory 调用。
- 游戏运行：未运行。
