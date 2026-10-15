---
title: "hounded"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "世界级猎犬袭击组件：等待→警告→分批放狗的状态机，按玩家年龄/聚团定波次，支持季节变体、升级替换（座狼/蠕虫 boss）、断线重连的波次保存与地点豁免。"
aliases: ["猎犬袭击", "狗群波次", "hounded"]
tags: [component, creature, state-machine]
size_class: complex
dst_build: "steam-buildid:26230496"
sources: ["components/hounded.lua", "prefabs/cave.lua", "prefabs/worm_boss.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# hounded

## 快速判断

- 职责与适用：世界级组件（仅主端），地表与洞穴各一份（[forest](../../../raw/current/scripts/prefabs/forest.lua#L643)、[cave](../../../raw/current/scripts/prefabs/cave.lua#L553-L555)，洞穴装配用 `SetSpawnData(wormspawn)` 换成蠕虫波数据）；按平均玩家年龄自动升级波次强度。
- 优先使用：改难度走世界事件 `hounded_setdifficulty`（never/rare/default/often/always，[L650-L662](../../../raw/current/scripts/components/hounded.lua#L650-L662)、[L719-L725](../../../raw/current/scripts/components/hounded.lua#L719-L725)）；暂停/恢复用 `pausehounded`/`unpausehounded`（带 source 的 SourceModifierList，[L611-L621](../../../raw/current/scripts/components/hounded.lua#L611-L621)）。
- 重要副作用：给陆上玩家推 `houndwarning`（HOUNDWARNINGTYPE LVL1~4，[constants](../../../raw/current/scripts/constants.lua#L2458)）与警告语音；洞穴蠕虫波在警告声里伴随 `ms_miniquake`（[L900-L902](../../../raw/current/scripts/components/hounded.lua#L900-L902)）；释放的猎犬默认 `SuggestTarget` 目标玩家。
- 修改边界：生成数与间隔全由 `_spawndata` 表驱动，`SetSpawnData` 可整体替换（洞穴 wormspawn 见 [cave.lua](../../../raw/current/scripts/prefabs/cave.lua#L240)）；组件不管猎犬 AI 与掉落。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `_spawndata` 默认结构 | `base_prefab/winter_prefab/summer_prefab/upgrade_spawn`、`attack_levels`（warnduration+numspawns）、`attack_delays`、`warning_speech`、`warning_sound_thresholds`、`ShouldUpgrade(amount, wave_pre_upgraded)`、可选 `specialupgradecheck` | 默认 hound/icehound/firehound/warglet，等级 intro→crazy（[L44-L94](../../../raw/current/scripts/components/hounded.lua#L44-L94)）。 |
| `SetSpawnData(data)` | 同上结构 | 装配期调用（洞穴在装配行即调）；影响后续所有波次的 prefab 与节奏（[L747-L749](../../../raw/current/scripts/components/hounded.lua#L747-L749)）。 |
| `SpawnModeNever/Light/Normal/Med/Heavy` | — | `never` 关闭；`light/med/heavy` 用 constant 模式固定延迟与警告时长；`normal`（escalating）按平均玩家年龄 <8/25/50/100 天分档（[L789-L820](../../../raw/current/scripts/components/hounded.lua#L789-L820)、[L120-L140](../../../raw/current/scripts/components/hounded.lua#L120-L140)）。 |
| `SetSummerVariant(bool)` / `SetWinterVariant(bool)` | — | 开关季节变体（`hounded_setsummervariant`/`setwintervariant` 世界事件驱动，[L678-L692](../../../raw/current/scripts/components/hounded.lua#L678-L692)）；按 `TUNING.HOUND_SPECIAL_CHANCE`（夏季 ×1.5）掷出时冬/春出 icehound、夏/秋出 firehound（[L394-L424](../../../raw/current/scripts/components/hounded.lua#L394-L424)、[tuning](../../../raw/current/scripts/tuning.lua#L1481)）。 |
| `SetWaveOverrideSettings({wavetype, setting})` | 蠕虫 boss 概率 | `hounds_worm_boss_setdifficulty` 难度映射 0~9999；实际抽签在 wormspawn 的 `specialupgradecheck`（[L664-L676](../../../raw/current/scripts/components/hounded.lua#L664-L676)、[L785-L787](../../../raw/current/scripts/components/hounded.lua#L785-L787)）。 |
| `hounded_storeupgraded` 事件 | 上行实体 | 蠕虫 boss 被收进包裹（非击杀）时存其存档记录，下一波以 `UPGRADEDSPAWN_NOTKILLED_DELAY_MULT`(1/6) 提前且直接还原实体（[L20](../../../raw/current/scripts/components/hounded.lua#L20)、[L694-L697](../../../raw/current/scripts/components/hounded.lua#L694-L697)、[worm_boss 推送](../../../raw/current/scripts/prefabs/worm_boss.lua#L568)、[还原](../../../raw/current/scripts/components/hounded.lua#L426-L457)）。 |
| `GetWorldEscalationLevel()` | → attack_levels 条目 | 与 CalcEscalationLevel 同档位，注释说明供树石 loot 数据使用（[L751-L767](../../../raw/current/scripts/components/hounded.lua#L751-L767)）。 |
| `ForceReleaseSpawn(target)` / `SummonSpawn(pt, radius_override)` / `ForceNextWave()` | 调试/强制入口 | 直接生成一只攻击目标的狗／在指定点半径内生成／立即打完当前波（[L822-L840](../../../raw/current/scripts/components/hounded.lua#L822-L840)）。 |
| `GetTimeToAttack` / `GetWarning` / `GetAttacking` | 读取器 | 外部 UI/模组查询倒计时与阶段（[L735-L745](../../../raw/current/scripts/components/hounded.lua#L735-L745)）。 |
| OnSave/OnLoad | 全局倒计时＋`missingplayerspawninfo`（按 userid） | 离线玩家的在途波次保存，重连经 `_delayedplayerspawninfo` 续放；读档保底警告期至少 5 秒（[L533-L581](../../../raw/current/scripts/components/hounded.lua#L533-L581)、[L1124-L1165](../../../raw/current/scripts/components/hounded.lua#L1124-L1165)）。 |

## 关键流程

### 一次袭击的生命周期

1. 规划 `PlanNextAttack`：清地点豁免、escalating 时按平均年龄换档，取 `attack_delays` 基数＋方差为倒计时；无玩家时每秒重试（[L162-L204](../../../raw/current/scripts/components/hounded.lua#L162-L204)）。
2. 警告：倒计时进入 `warnduration` 后置 `_warning`，按剩余时间分档播 `houndwarning`（<30s 每 0.3~1.3s）并定期警告语音；蠕虫波警告间隔额外 +4s（[L1087-L1115](../../../raw/current/scripts/components/hounded.lua#L1087-L1115)）。
3. 组波 `GetWaveAmounts`：20 距离内玩家成组；每玩家按**自己**年龄取数量，组内总量按 `n^(1/(0.95+0.05n))` 削峰；按 [[reference/components/houndedtarget]] 的 `GetTargetWeight` 加权分摊；`IsHoundThief` 的玩家（偷狗贼）被重分配额外目标，上限 `max_thieved_spawn_per_thief`（洞穴装配设为 1）（[L206-L340](../../../raw/current/scripts/components/hounded.lua#L206-L340)、[L28](../../../raw/current/scripts/components/hounded.lua#L28)）。
4. 放狗 `HandleSpawnInfoRec`：逐只释放，间隔按组平均年龄 0.5~8s；数量 ≥5 时按 `ShouldUpgrade` 概率用 upgrade_spawn（warglet 等）抵扣 5 只名额一次（[L927-L991](../../../raw/current/scripts/components/hounded.lua#L927-L991)、[L82-L93](../../../raw/current/scripts/components/hounded.lua#L82-L93)）。
5. 收尾：组队列清空后 `PlanNextAttack` 排下一波（[L1082-L1086](../../../raw/current/scripts/components/hounded.lua#L1082-L1086)）。

### 地点豁免

释放与警告只面向 `_targetableplayers` 为 nil 或 `"land"` 的玩家；分 `"arena"`（瓦冈朋克竞技场）、`"vault"`、`"charliearena"`、`"water"` 四类，每次警告/释放前刷新（[L623-L648](../../../raw/current/scripts/components/hounded.lua#L623-L648)、[L459-L469](../../../raw/current/scripts/components/hounded.lua#L459-L469)）。

## 常见陷阱与限制

- **波次强度看平均年龄，数量看个人年龄**：升级 delay 用平均值、`CalcPlayerAttackSize` 用个人值；新玩家在组里会把整组波拉低到 intro 数量（[L120-L149](../../../raw/current/scripts/components/hounded.lua#L120-L149)）。
- **玩家中途退出会留"在途狗"**：退出时 `SavePlayerSpawnInfo` 把剩余数量存进 missingplayerspawninfo，重连后由 `_delayedplayerspawninfo` 单独走一套倒计时与警告（不与其他玩家合波）（[L569-L581](../../../raw/current/scripts/components/hounded.lua#L569-L581)、[L998-L1047](../../../raw/current/scripts/components/hounded.lua#L998-L1047)）。
- **暂停语义不对称**：`pausehounded` 在未进警告时冻结倒计时；已进警告或已有在途波则继续（[L1056-L1061](../../../raw/current/scripts/components/hounded.lua#L1056-L1061)）。
- **出生点失败不重试**：`GetSpawnPoint` 找不到合法陆点时该只猎犬静默丢失，不补偿数量（[L426-L457](../../../raw/current/scripts/components/hounded.lua#L426-L457)）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。覆盖组件全文、cave/forest 装配与 wormspawn 数据、worm_boss 存取事件、HOUND_SPECIAL_CHANCE/HOUNDWARNINGTYPE 常量；未覆盖 wormspawn 表逐字段与 warglet prefab 细节。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、Lua 语法探测通过（见批级记录）。

相关：[[reference/components/houndedtarget]]、[[reference/components/luckuser]]。
