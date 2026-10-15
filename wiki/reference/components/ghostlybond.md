---
title: "ghostlybond"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "玩家与随行幽灵的纽带组件：管理幽灵生成/召回/行为切换与 1~3 级纽带等级成长，Wendy 与 Abigail 的核心链路。"
aliases: ["幽灵纽带", "ghostlybond", "阿比盖尔纽带", "姐妹纽带"]
tags: [component, callback]
dst_build: "steam-buildid:26230496"
sources: ["components/ghostlybond.lua", "prefabs/wendy.lua", "prefabs/abigail.lua", "actions.lua", "prefabs/ghostly_elixirs.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# ghostlybond

## 快速判断

- 职责与适用：给玩家（原版仅 Wendy，[挂载](../../../raw/current/scripts/prefabs/wendy.lua#L386-L393)）绑定一只可召回的幽灵随从：`Init(prefab, 升级总时长)` 后下帧生成幽灵，纽带等级 1~3 随时间成长（[构造](../../../raw/current/scripts/components/ghostlybond.lua#L19-L37)、[Init](../../../raw/current/scripts/components/ghostlybond.lua#L147-L153)）。
- 优先使用：召唤/召回/切换行为对应 `Summon`/`Recall`/`ChangeBehaviour`，动作层 CASTSUMMON/CASTUNSUMMON/COMMUNEWITHSUMMONED 直接调用（[动作](../../../raw/current/scripts/actions.lua#L3687-L3705)）。
- 重要副作用：幽灵死亡自动把等级重置为 1 并召回（[死亡](../../../raw/current/scripts/components/ghostlybond.lua#L14-L17)）；幽灵被移除时立即重生成（[onremove](../../../raw/current/scripts/components/ghostlybond.lua#L9-L12)）。
- 修改边界：`Summon` 需要幽灵处于未召唤态且调用方带 summoningitem（换肤用），并要求宿主有 `pethealthbar` 组件（[依赖](../../../raw/current/scripts/components/ghostlybond.lua#L167-L187)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `Init(ghost_prefab, bond_levelup_time)` | ghost_prefab: string | 记录幽灵 prefab 与升级总时长，下帧 `SpawnGhost`（[实现](../../../raw/current/scripts/components/ghostlybond.lua#L147-L153)）。 |
| `SetBondLevel(level, time, isloading)` | level: 1~maxbondlevel | 成长/回退核心；等级变化触发 `onbondlevelchangefn` 并推 `ghostlybond_level_change`（[实现](../../../raw/current/scripts/components/ghostlybond.lua#L131-L145)）。 |
| `SetBondTimeMultiplier(src, mult, key)` | mult: number or nil | 外部时间乘数（SourceModifierList），原版姐姐之花用它加速成长（[实现](../../../raw/current/scripts/components/ghostlybond.lua#L115-L117)、[wendy](../../../raw/current/scripts/prefabs/wendy.lua#L262)）。 |
| `PauseBonding()` / `ResumeBonding()` | → `nil` | 暂停/恢复等级成长计时（[实现](../../../raw/current/scripts/components/ghostlybond.lua#L119-L129)）。 |
| `Summon(item, pos)` / `Recall(was_killed)` | → `bool` | 召回需当前 summoned 且宿主非 dissipate 状态；成功触发对应 onfn（[实现](../../../raw/current/scripts/components/ghostlybond.lua#L167-L212)）。 |
| `ChangeBehaviour()` | → `bool` | 转发到宿主注册的 `changebehaviourfn`（[实现](../../../raw/current/scripts/components/ghostlybond.lua#L228-L233)）。 |
| `OnSave()` / `OnLoad(data)` | 组件存档钩子 | 保存等级、已计时、幽灵存档记录与 limbo 态（[实现](../../../raw/current/scripts/components/ghostlybond.lua#L55-L95)）。 |

## 典型接入

### 为自定义角色绑定随行幽灵

- 前提与选择：mastersim；宿主需 `pethealthbar`（Summon 换肤路径）；先挂组件、设回调，再 `Init`。
- 生效与撤销：六个回调（onbondlevelchangefn/onsummonfn/onrecallfn/onsummoncompletefn/onrecallcompletefn/changebehaviourfn）都可选；等级计时经 `OnUpdate`/`LongUpdate` 推进（[推进](../../../raw/current/scripts/components/ghostlybond.lua#L99-L113)）。
- 原版依据：Wendy 注册回调并 `Init("abigail", TUNING.ABIGAIL_BOND_LEVELUP_TIME)`（[wendy](../../../raw/current/scripts/prefabs/wendy.lua#L386-L393)）；幽灵药水直接 `SetBondLevel(3)`（[灵药](../../../raw/current/scripts/prefabs/ghostly_elixirs.lua#L291-L292)）；abigail 侧回读 `bondlevel` 决定强度（[abigail](../../../raw/current/scripts/prefabs/abigail.lua#L454-L460)）。

```lua
inst:AddComponent("ghostlybond")
inst.components.ghostlybond.onbondlevelchangefn = function(owner, ghost, level, prev, isloading) end
inst.components.ghostlybond.changebehaviourfn = function(owner, ghost) return true end
inst.components.ghostlybond:Init("abigail", TUNING.ABIGAIL_BOND_LEVELUP_TIME)
-- 召回/召唤由动作系统触发，也可直接：
inst.components.ghostlybond:Recall(false)
```

- 调整与验证：等级字段 `bondlevel` 可直接读；召唤状态由 watcher 同步为 `ghostfriend_summoned`/`ghostfriend_notsummoned` 标签（[标签](../../../raw/current/scripts/components/ghostlybond.lua#L1-L7)），UI 按标签过滤。

## 常见陷阱与限制

- `OnUpdate` 里判断的是 `self.pause`，而 `PauseBonding` 设置的也是 `pause`——但构造初始字段名是 `paused`（[构造](../../../raw/current/scripts/components/ghostlybond.lua#L26)）；`GetDebugString` 读 `paused`。以 `PauseBonding`/`ResumeBonding` 为准操作，勿手写字段。
- 召回链路分两段：`Recall` 只触发 onrecallfn（原版在此播 dissipate 动画），真正从场景移除由动画完成时调 `RecallComplete`（[对照](../../../raw/current/scripts/prefabs/wendy.lua#L227)）；直接改 `summoned` 字段会卡死状态机。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，覆盖组件全文与 wendy、abigail、actions、ghostly_elixirs、abigail_flower 消费点；未运行游戏。
- 结构检查：validate_page.py、validate_source_links.py 与 Lua 语法探针通过。
