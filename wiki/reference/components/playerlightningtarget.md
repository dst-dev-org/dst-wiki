---
title: "playerlightningtarget"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "让玩家成为天雷目标：登记基础命中率与受击回调，天气系统按命中率掷骰选中后调 DoStrike。"
aliases: ["玩家雷击目标", "闪电命中率", "playerlightningtarget"]
tags: [component, callback]
dst_build: "steam-buildid:26230496"
size_class: simple
sources: ["components/playerlightningtarget.lua", "components/weather.lua", "prefabs/player_common.lua", "prefabs/wes.lua", "prefabs/wx78.lua", "tuning.lua", "prefabs/skinprefabs.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# playerlightningtarget

## 快速判断

- 职责与适用：玩家侧的雷击目标组件：`hitchance` 参与天气系统选雷击目标时的幸运掷骰，被选中后 `DoStrike()` 执行回调。默认回调造成 `LightningStrikeAttack`，装备绝缘物品可免疫（[默认回调](../../../raw/current/scripts/components/playerlightningtarget.lua#L3-L11)）。所有玩家在 player_common 的 master 分支添加（[player_common.lua#L2784](../../../raw/current/scripts/prefabs/player_common.lua#L2784)）。
- 优先使用：改命中率用 `SetHitChance` 或往 `hitchancemodifiers`（SourceModifierList）加修正；换受击表现用 `SetOnStrikeFn`。
- 重要副作用：`DoStrike()` 总会推 `playerlightningtargeted` 事件（无论回调内容），皮肤表现监听它（[推送](../../../raw/current/scripts/components/playerlightningtarget.lua#L37)、[皮肤监听](../../../raw/current/scripts/prefabs/skinprefabs.lua#L1506-L1512)）。
- 修改边界：只对"无避雷针／阻挡物兜底后的玩家候选"生效（选中链见 [weather.lua#L657](../../../raw/current/scripts/components/weather.lua#L657)）；绝缘免疫在默认回调里实现，换掉回调就没了。

## 关键接口

| 接口／字段 | 已核验类型 | 条件、作用与证据 |
|---|---|---|
| `hitchance` | number 概率，默认 `TUNING.PLAYER_LIGHTNING_TARGET_CHANCE` = 0.3 | 基础命中率；wes 0.6、wx78 1.0（[tuning.lua#L655-L657](../../../raw/current/scripts/tuning.lua#L655-L657)、[wes](../../../raw/current/scripts/prefabs/wes.lua#L61)、[wx78](../../../raw/current/scripts/prefabs/wx78.lua#L815)）。 |
| `GetHitChance()` | number | `hitchance * hitchancemodifiers:Get()`，修正器叠加在此。[L25-L27](../../../raw/current/scripts/components/playerlightningtarget.lua#L25-L27) |
| `SetOnStrikeFn(fn)` | `fn(inst)` 或 nil | 替换受击回调；wx78 用它接 `OnLightningStrike`（[wx78.lua#L815-L816](../../../raw/current/scripts/prefabs/wx78.lua#L815-L816)）。[L29-L31](../../../raw/current/scripts/components/playerlightningtarget.lua#L29-L31) |
| `DoStrike()` | 无返回 | 执行回调并推 `playerlightningtargeted`；天气系统选中后调用（[weather.lua#L718-L719](../../../raw/current/scripts/components/weather.lua#L718-L719)）。[L33-L38](../../../raw/current/scripts/components/playerlightningtarget.lua#L33-L38) |

## 关键流程

1. 雷击落点选择：天气组件遍历候选，普通实体与玩家都用 `TryLuckRoll(v, GetHitChance(), LuckFormulas.LightningStrike)` 掷骰，失败则该候选落选（[weather.lua#L657](../../../raw/current/scripts/components/weather.lua#L657)）。
2. 没被避雷针／lightningblocker 转移时，对选中的玩家候选调 `DoStrike()`（[weather.lua#L716-L720](../../../raw/current/scripts/components/weather.lua#L716-L720)）。
3. 默认回调：存活且非无敌才结算；未绝缘走 `LightningStrikeAttack(inst)`，绝缘则推 `lightningdamageavoided`（[L3-L11](../../../raw/current/scripts/components/playerlightningtarget.lua#L3-L11)）。

## 常见陷阱与限制

- `SetHitChance` 直接覆盖基础值，会丢掉原版 tuning：角色改命中率优先用 `hitchancemodifiers` 叠乘，保留 0.3 基数。
- 概率语义是"每次落雷掷骰"而非"全局受击比例"：命中率 1.0 的 wx78 也不是每雷必中，仍受选中顺序／遮挡影响。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。完整阅读 components/playerlightningtarget.lua（40 行），核对天气选中链两处调用、wes／wx78 的定制、tuning 数值与皮肤事件监听。未覆盖 LightningStrikeAttack 的内部伤害结算。
- 游戏运行：未运行。
- 结构检查：validate_page.py 通过（0 issue；50 行、4560 字节，simple 档内）；validate_source_links.py --strict 通过（16 处源码引用，0 error／0 warning）；本页无完整 Lua 代码块。
