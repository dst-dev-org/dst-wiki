---
title: "hunter"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "世界级脚印狩猎组件：给每名玩家周期投放可疑土堆与兽踪，调查后按分数与季节/事件结算出考拉象、座狼、山羊等猎物。"
aliases: ["狩猎管理", "脚印狩猎", "hunter"]
tags: [component, creature]
dst_build: "steam-buildid:26230496"
sources: ["components/hunter.lua", "prefabs/dirtpile.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# hunter

## 快速判断

- 职责与适用：世界级组件（仅主端），装配于地表世界（[forest](../../../raw/current/scripts/prefabs/forest.lua#L658)）；管理"可疑的土堆→脚印→猎物"全流程，每名在线玩家同时最多一场狩猎（[L52-L54](../../../raw/current/scripts/components/hunter.lua#L52-L54)、[L468-L477](../../../raw/current/scripts/components/hunter.lua#L468-L477)）。
- 优先使用：模组不改狩猎逻辑时无需接入；改"调查土堆"行为走 `OnDirtInvestigated(pt, doer)`（[L526-L567](../../../raw/current/scripts/components/hunter.lua#L526-L567)），土堆 prefab 已在调查动作里回调它（[dirtpile](../../../raw/current/scripts/prefabs/dirtpile.lua#L17-L19)）。
- 重要副作用：生成 `dirtpile`／`animal_track` 与最终猎物实体，并向调查者推 `huntstartfork`／`huntsuccessfulfork`／`huntwrongfork`／`huntbeastnearby`／`huntlosttrail` 事件（[L294-L300](../../../raw/current/scripts/components/hunter.lua#L294-L300)、[L408-L424](../../../raw/current/scripts/components/hunter.lua#L408-L424)）。
- 修改边界：组件只决定"何时何地放踪迹、结算成什么"；猎物的具体登场行为（睡/吼/扑）由猎物 prefab 响应 `spawnedforhunt` 自行处理（[L434-L435](../../../raw/current/scripts/components/hunter.lua#L434-L435)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `OnDirtInvestigated(pt, doer)` | pt 调查点，doer 玩家 | 按 `lastdirt` 位置匹配所属狩猎；推进下一枚足迹或结算猎物，失败即重置（[L526-L567](../../../raw/current/scripts/components/hunter.lua#L526-L567)）。 |
| `IsWargShrineActive()` / `IsSnakeShrineActive()` | → bool | 对应祭坛 SourceModifierList 激活且在 YOTV／YOTS 特殊活动期间（[L36-L37](../../../raw/current/scripts/components/hunter.lua#L36-L37)、[L569-L575](../../../raw/current/scripts/components/hunter.lua#L569-L575)、[注册](../../../raw/current/scripts/components/hunter.lua#L514-L517)）。 |
| hunt 状态字段 | `{lastdirt, direction, activeplayer, score}` | score 最终会被覆写为 0~1 的结算分（追踪耗时换算，[L400-L405](../../../raw/current/scripts/components/hunter.lua#L400-L405)）。 |
| `DebugForceHunt()` | — | 立即开一场 0.1s 冷却的调试狩猎（[L596-L604](../../../raw/current/scripts/components/hunter.lua#L596-L604)）。 |
| TUNING 关键值 | `HUNT_SPAWN_DIST`（一屏）、`HUNT_COOLDOWN` 1.2 日 ±0.3、`HUNT_SCORE_*`、`HUNT_RESET_TIME` 5s、`HUNT_ALTERNATE_BEAST_CHANCE` 0.05→0.33 | [tuning.lua](../../../raw/current/scripts/tuning.lua#L963-L973)。 |

## 关键流程

### 狩猎生命周期

1. 起猎：玩家进服 `KickOffHunt`，未满额则 `StartHunt` 进入冷却；冷却结束 `OnUpdateHunt` 每 2 秒挑选一名"无进行中狩猎、不近他场土堆、在地面且 areaaware 区域无 `nohunt`/`moonhunt` 标签"的玩家投放第一堆土（[L81-L98](../../../raw/current/scripts/components/hunter.lua#L81-L98)、[L276-L291](../../../raw/current/scripts/components/hunter.lua#L276-L291)、[L334-L356](../../../raw/current/scripts/components/hunter.lua#L334-L356)）。
2. 放踪：`StartDirt` 随机 6~12 步（`MIN/MAX_TRACKS`），随机方向逐步投放 `dirtpile`＋爪痕（怪物路径附加 `animal_track` 爪印，[L166-L191](../../../raw/current/scripts/components/hunter.lua#L166-L191)、[L246-L273](../../../raw/current/scripts/components/hunter.lua#L246-L273)）。万圣节活动期指定某步触发蝙蝠伏击（3+天数/35 只，上限 6，[L206-L213](../../../raw/current/scripts/components/hunter.lua#L206-L213)）。
3. 怪物踪迹判定：月裂未击败 mutatedwarg 且月之门户激活→第 0 步即怪物踪迹（hunted warg）；否则按 `TryLuckRoll` 掷 `HuntAlternateBeast` 概率在中途转怪物踪迹（[L46-L50](../../../raw/current/scripts/components/hunter.lua#L46-L50)、[L253-L259](../../../raw/current/scripts/components/hunter.lua#L253-L259)；运气组件见 [[reference/components/luckuser]]）。
4. 结算：最后一步调查后 `GetHuntedBeast` 选猎物——双祭坛活动随机 claywarg／yots_worm；春季雨天沙暴节点优先生成 lightninggoat；怪物踪迹按 score 低于 0.25 出"假叉"道具、低于 0.4 出睡兽、否则成功遭遇；普通踪迹按季节出冬/夏考拉象（[L368-L392](../../../raw/current/scripts/components/hunter.lua#L368-L392)、[L409-L424](../../../raw/current/scripts/components/hunter.lua#L409-L424)）。

## 常见陷阱与限制

- **踪迹会过期与洗掉**：最后土堆超过 1.25 个 SEG_TIME（雨天或湿度 >15 为 0.75）无人跟进就重置，玩家已在别场狩猎则静默冷却，否则推 `huntlosttrail`（湿场标记 washedaway）（[L303-L331](../../../raw/current/scripts/components/hunter.lua#L303-L331)）；土堆被移除同样重置（[L215-L220](../../../raw/current/scripts/components/hunter.lua#L215-L220)）。
- **玩家走远即弃猎**：最后土堆附近无玩家（`MAX_DIRT_DISTANCE`）0.1 秒后重新冷却另起（[L358-L365](../../../raw/current/scripts/components/hunter.lua#L358-L365)）。
- **每玩家一场的配额语义**：`GetMaxHunts()` 等于在线玩家数，冷却中的狩猎仍占 `_activehunts` 名额，超出的狩猎在冷却开始时被直接移除（[L110-L129](../../../raw/current/scripts/components/hunter.lua#L110-L129)）；`LongUpdate` 只补偿冷却计时（[L581-L590](../../../raw/current/scripts/components/hunter.lua#L581-L590)）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。覆盖组件全文、dirtpile 回调、tuning 狩猎段、forest 装配行；未覆盖各猎物 prefab 对 `spawnedforhunt` 的具体表现分支。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、Lua 语法探测通过（见批级记录）。
