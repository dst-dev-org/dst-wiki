---
title: "klaussackspawner"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "Klaus 袋冬季生成器：在鹿群繁殖点择优埋 klaus_sack，受 worldsettingstimer 冬季重生成计时与冬日盛宴事件分支控制。"
aliases: ["克劳斯袋子生成", "Klaus袋生成器", "klaussackspawner"]
tags: [component, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/klaussackspawner.lua", "prefabs/forest.lua", "prefabs/world.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# klaussackspawner

## 快速判断

- 职责与适用：挂在地表世界实体（[安装](../../../raw/current/scripts/prefabs/forest.lua#L654)，[断言](../../../raw/current/scripts/components/klaussackspawner.lua#L7)），冬季在 `deerspawningground` 登记的繁殖点里择优生成 `klaus_sack`；对象是 Klaus 入口与冬日盛宴刷袋。
- 优先使用：查当前袋子用 `GetKlausSack()`（[L116-L120](../../../raw/current/scripts/components/klaussackspawner.lua#L116-L120)）；调节奏改 `TUNING.KLAUSSACK_*`（[常量](../../../raw/current/scripts/tuning.lua#L735-L739)）与总开关 `TUNING.SPAWN_KLAUS`（[L740](../../../raw/current/scripts/tuning.lua#L740)）。
- 重要副作用：选中的生成瓦片上 2 半径内的 structure 会被销毁或移除腾位（[L74-L86](../../../raw/current/scripts/components/klaussackspawner.lua#L74-L86)）。
- 修改边界：计时走 [[reference/components/worldsettingstimer]] 的 `"klaussack_spawntimer"`；`OnPostInit` 由 world 通用分发调用（[分发](../../../raw/current/scripts/prefabs/world.lua#L342-L346)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `GetKlausSack()` | → 实体或 nil | 返回当前有效袋子；wintersurprisespawner 据此找袋子（[消费](../../../raw/current/scripts/components/wintersurprisespawner.lua#L39-L50)）。 |
| `OnPostInit()` | 无参 | 世界 PostInit 时按"盛宴／普通"二分支注册计时与 iswinter 监听；普通分支受 `TUNING.SPAWN_KLAUS` 门控（[L202-L228](../../../raw/current/scripts/components/klaussackspawner.lua#L202-L228)）。 |
| `OnSave` / `OnLoad` | `spawnsthiswinter`＋旧字段 `timetorespawn` | 读档恢复冬季已刷次数；旧档无次数时 PostInit 记 1 次（[L234-L255](../../../raw/current/scripts/components/klaussackspawner.lua#L234-L255)）。 |

## 生成规则（关键流程）

1. 候选点：监听 `ms_registerdeerspawningground`（[推事件](../../../raw/current/scripts/prefabs/deerspawningground.lua#L12)）收集繁殖点，实体移除自动退册（[L135-L144](../../../raw/current/scripts/components/klaussackspawner.lua#L135-L144)）。
2. 择点：打乱后依次要求——3×3 瓦片可通行、35 半径无玩家；优先选 5 半径内无 structure 的点，否则取 structure 最少者，再退化到随机点（[L41-L87](../../../raw/current/scripts/components/klaussackspawner.lua#L41-L87)）。
3. 落袋：取瓦片中心生成 `klaus_sack`，销毁 2 半径内 structure（可工作则 `workable:Destroy`，否则移除）（[L74-L86](../../../raw/current/scripts/components/klaussackspawner.lua#L74-L86)）。
4. 计时链：袋子移除时若盛宴活动开 → 20 天（`KLAUSSACK_EVENT_RESPAWN_TIME`）后重刷；否则冬季内 `KLAUSSACK_MAX_SPAWNS(1)` 次数未满 → `RESPAWN_DELAY 10天 + SPAWN_DELAY 1天±2天`（[L146-L157](../../../raw/current/scripts/components/klaussackspawner.lua#L146-L157)）。
5. 换季：入冬即检查条件起计时；出冬停计时清零次数；盛宴活动只监听出冬清零（[L172-L228](../../../raw/current/scripts/components/klaussackspawner.lua#L172-L228)）。
6. 钥匙恢复：监听 `ms_restoreklaussackkey`，袋子在时调其 `OnDropKey`（[L166-L170](../../../raw/current/scripts/components/klaussackspawner.lua#L166-L170)；[推事件](../../../raw/current/scripts/components/klaussackkey.lua#L20)）。

## 常见陷阱与限制

- **OnLoad 的旧字段陷阱**：`data.timetorespawn` 用真值判断，为 0 或 nil 时不恢复计时；注释明确"可能是 false 所以不做 nil 检查"（[L241-L245](../../../raw/current/scripts/components/klaussackspawner.lua#L241-L245)）。
- **盛宴分支不受 SPAWN_KLAUS 门控**：活动期直接在 PostInit 无袋即刷（"day 1 spawn"）；关 `TUNING.SPAWN_KLAUS` 只关普通链（[L218-L227](../../../raw/current/scripts/components/klaussackspawner.lua#L218-L227)）。
- **袋子登记先到先得**：`ms_registerklaussack` 只在无有效袋子时接受新袋；世界补生成的袋子需经此登记才被追踪（[L159-L164](../../../raw/current/scripts/components/klaussackspawner.lua#L159-L164)）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。覆盖组件全文、forest 装配行、world 通用 OnPostInit 分发、deerspawningground／klaus_sack／klaussackkey 事件两端、KLAUSSACK_* 调 tuning 段与 wintersurprisespawner 消费；未覆盖 klaus_sack 内部开袋战斗流程。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、Lua 语法探测通过（见批级记录）。

相关：[[reference/components/klaussackloot]]（袋子掉落）、[[reference/components/klaussacklock]]（袋锁链）。
