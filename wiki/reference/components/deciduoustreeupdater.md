---
title: "deciduoustreeupdater：桦树精怪物化行为驱动"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "挂在桦树实体上的怪物化驱动：树变桦树精后按周期索敌、砸根须、放桦果鸭，睡觉/烧毁/砍倒即解除并移除组件。"
aliases: [桦树精更新器, 桦树怪物化, deciduoustreeupdater, 桦果鸭生成]
tags: [component, state-machine, ai, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/deciduoustreeupdater.lua", "prefabs/deciduoustrees.lua", "stategraphs/SGdeciduoustree.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# deciduoustreeupdater（桦树精怪物化行为驱动）

## 快速判断

- 职责与适用：服务端组件，挂载发生在桦树怪物化时——`deciduoustrees.lua` 补挂 combat 后加组件并 `StartMonster`（[挂载](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L819-L822)）；砍倒/烧毁/挖根等恢复路径都会移除它（[移除](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L589)）。负责怪物化期间的索敌、根须攻击、两种桦果鸭（proximity/passive）投放与摇摆动画事件。
- 优先使用：让一棵树怪物化 → 走 prefab 的 `StartMonster` 入口（自动补挂组件）；点燃怪物化树时 prefab 调 `SpawnIgniteWave` 额外放一波鸭（[点燃波](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L903-L904)）。
- 重要副作用：怪物化加 `monster` 标签并推 `sway` 事件驱动摇摆动画；怪物持续时间 `DECID_MONSTER_DURATION`（半个游戏日 ±33%）到点自动恢复（[时长](../../../raw/current/scripts/components/deciduoustreeupdater.lua#L26-L28)）。
- 修改边界：实体休眠 1 秒后整体停止怪物化（[休眠](../../../raw/current/scripts/components/deciduoustreeupdater.lua#L83-L87)）；唤醒时若非睡眠恢复期则重新 `StartMonster`（[唤醒](../../../raw/current/scripts/components/deciduoustreeupdater.lua#L69-L76)）；`OnRemoveFromEntity` 即 `StopMonster`（[卸载](../../../raw/current/scripts/components/deciduoustreeupdater.lua#L67)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `StartMonster(starttime)` | 起始时间或 nil | 置怪物态、`monster_duration` 半日±33%、索敌节拍 `0.5+rand`、加 `monster` 标签，19 帧后启动更新；prefab 用它跨读档续时（[实现](../../../raw/current/scripts/components/deciduoustreeupdater.lua#L21-L37)、[存读续时](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L1048-L1049)）。 |
| `StopMonster()` | 无参 | 清标签、取消全部任务、停更新；prefab 的怪物结束与组件卸载都走它（[实现](../../../raw/current/scripts/components/deciduoustreeupdater.lua#L39-L65)）。 |
| `SpawnIgniteWave()` | 无参 | 怪物化中点燃时按 `MIN~MAX_TREE_DRAKES`(3~5) 环形补放一波桦果鸭（[实现](../../../raw/current/scripts/components/deciduoustreeupdater.lua#L316-L324)）。 |
| `OnUpdate(dt)` | 组件更新 | 时长到→安排恢复；`time_to_passive_drake` 驱动被动鸭（幸运掷判定大波 2 只，`TryLuckRoll`，[被动鸭](../../../raw/current/scripts/components/deciduoustreeupdater.lua#L213-L235)）；按 `monsterFreq` 节拍索敌→根须攻击→近距离放环形鸭（[主循环](../../../raw/current/scripts/components/deciduoustreeupdater.lua#L192-L294)）。 |
| `monster_target`／`last_monster_target` | 实体或 nil | 索敌结果；SG 摇摆状态亦读取（[SG 读取](../../../raw/current/scripts/stategraphs/SGdeciduoustree.lua#L142)）。 |

攻击实现：目标存在且距上次攻击超过 `DECID_MONSTER_ATTACK_PERIOD`(2.3s) 时在树前 1.75 单位生成 `deciduous_root` 并推 `givetarget`（目标、落点、角度）（[根须](../../../raw/current/scripts/components/deciduoustreeupdater.lua#L255-L271)）；目标进入 1/2 索敌距离内则环形放 3~5 只 `birchnutdrake`，每 6 帧一只、方位按等分角（[环形鸭](../../../raw/current/scripts/components/deciduoustreeupdater.lua#L273-L282)）。

## 典型接入

### 手动触发/解除一棵树的怪物化

- 前提与选择：服务端树实体；原版入口 `StartMonster` 在 prefab 层会同时补 combat 与组件，模组直接调组件方法须自己保证 combat 存在。
- 生效与撤销：`StartMonster` 幂等（已在怪物态直接返回）；解除依赖 `StopMonster` 或树进入休眠/砍烧路径。

```lua
-- 服务端：对一棵桦树手动怪物化（对照 deciduoustrees.lua 的入口）
inst.components.deciduoustreeupdater:StartMonster()  -- 组件不存在时先 AddComponent
-- 解除怪物化
inst.components.deciduoustreeupdater:StopMonster()
```

- 调整与验证：索敌范围 `DECID_MONSTER_TARGET_DIST`(7)（[调谐](../../../raw/current/scripts/tuning.lua#L2909)），索敌排除飞行、墙与其他桦果鸭（[过滤](../../../raw/current/scripts/components/deciduoustreeupdater.lua#L146-L147)）。

## 常见陷阱与限制

- **休眠即停战**：树 `entitysleep` 1 秒后 StopMonster；玩家跑远就能"甩掉"桦树精，回来重新触发（[休眠停止](../../../raw/current/scripts/components/deciduoustreeupdater.lua#L83-L87)）。
- **烧着时不再补鸭**：恢复与继续判定要求树未在燃烧、非 stump/burnt（[恢复条件](../../../raw/current/scripts/components/deciduoustreeupdater.lua#L199-L207)）。
- **被动鸭与环形鸭是两套计数**：`num_passive_drakes`（幸运大波）与 `numdrakes`（环形 proximity）互不复用，点燃波另有 `ignitenumdrakes`（[独立计数](../../../raw/current/scripts/components/deciduoustreeupdater.lua#L316-L323)）。
- **时长计的是真实时间**：`monster_start_time = GetTime()`，`LongUpdate` 无补偿逻辑，跳时间（睡眠）会直接耗尽怪物时长（[时长判定](../../../raw/current/scripts/components/deciduoustreeupdater.lua#L193)）。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496（steam-buildid:26230496）；覆盖 `components/deciduoustreeupdater.lua` 全部 327 行、deciduoustrees.lua 挂载/移除/续时/点燃波调用点、SGdeciduoustree 读取、tuning DECID_MONSTER 与树鸭常量。未覆盖：`deciduous_root` 根须行为、`birchnutdrake` AI、幸运公式 LuckFormulas.ChildSpawnerRareChild。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 通过；lua 示例通过 parser_probe 语法检查。
- 游戏运行：未运行。
