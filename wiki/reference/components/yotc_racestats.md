---
title: "yotc_racestats"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "卡拉特鼠四维赛跑属性：速度／方向／反应／耐力的加点、退化与基线快照，驱动鼠跑速、耐力上限与健忘行为。"
aliases: ["卡拉特属性", "赛跑鼠属性", "yotc_racestats"]
tags: [component]
size_class: simple
dst_build: "steam-buildid:26230496"
sources: ["components/yotc_racestats.lua", "prefabs/carrat.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# yotc_racestats

## 快速判断

- 职责与适用：挂在卡拉特鼠 `carrat` 与 `carrat_ghostracer`（[幽灵鼠装配](../../../raw/current/scripts/prefabs/carrat_ghostracer.lua#L136)），存四维 0–5 的整数值；纯数据组件，无网络变量、无 mastersim 断言、无事件监听。
- 优先使用：加点用 `ModifySpeed` 等四个 Modify；批量随机用 `AddRandomPointSpread`；回退用 `DegradePoints`（[L31-L69](../../../raw/current/scripts/components/yotc_racestats.lua#L31-L69)、[L146-L210](../../../raw/current/scripts/components/yotc_racestats.lua#L146-L210)）。
- 重要副作用：无直接副作用；数值经由 carrat 装配段换算成跑速、耐力上限与 `isforgetful` 标记（[消费](../../../raw/current/scripts/prefabs/carrat.lua#L329-L355)）。
- 修改边界：上限 `TUNING.RACE_STATS.MAX_STAT_VALUE = 5`，下限 0；有基线时 Modify 的下限抬到基线（[常量](../../../raw/current/scripts/tuning.lua#L5191-L5197)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `speed` / `direction` / `reaction` / `stamina` | 整数 0..5，初始 `INIT_STAT_VALUE=0` | 构造赋初值；基线 4 字段初始为 nil（[L1-L13](../../../raw/current/scripts/components/yotc_racestats.lua#L1-L13)）。 |
| `GetSpeedModifier()` 等 4 个 | → `stat / 5`（0..1） | 消费端按比例 Lerp 出跑速／耐力（[L15-L29](../../../raw/current/scripts/components/yotc_racestats.lua#L15-L29)、[换算](../../../raw/current/scripts/prefabs/carrat.lua#L336-L355)）。 |
| `ModifySpeed(point_mod)` 等 4 个 | point_mod 整数 | clamp 在 `baseline_* or MIN` 与 MAX 之间；0 与 nil 不动（[L31-L69](../../../raw/current/scripts/components/yotc_racestats.lua#L31-L69)）。 |
| `AddRandomPointSpread(num_points)` | num_points 整数 | 在未满四维里随机各加 1，满则剔除候选；卡拉特出生撒 3 点、牛卡拉特撒 5 点（[L94-L144](../../../raw/current/scripts/components/yotc_racestats.lua#L94-L144)）。 |
| `DegradePoints(num_points)` | → `bool` 仍可退化 | 随机降高于基线的维度；`num_points=nil` 直接重置到基线；无基线时以初始值兜底（[L146-L210](../../../raw/current/scripts/components/yotc_racestats.lua#L146-L210)）。 |
| `SaveCurrentStatsAsBaseline()` | 无参 | 快照当前四维为基线；健美操退化、Modify 下限都以此为界（[L212-L217](../../../raw/current/scripts/components/yotc_racestats.lua#L212-L217)）。 |
| `OnSave` / `OnLoad` | 8 字段全存 | 四维＋四基线原样持久化（[L219-L245](../../../raw/current/scripts/components/yotc_racestats.lua#L219-L245)）。 |

## 典型接入

### 给自定义鼠起点数或训练加点（原版同构）

- 前提与选择：主机侧加组件后调用即可；原版出生在 carrat.lua `spread_stats` 撒点并立即 `SaveCurrentStatsAsBaseline`，训练器每天上限 `TUNING.CARRAT_GYM.TRAINS_PER_DAY=4`，每次 `ModifyXxx(1)`（[撒点](../../../raw/current/scripts/prefabs/carrat.lua#L525-L533)、[训练](../../../raw/current/scripts/prefabs/carrat.lua#L536-L546)）。

```lua
AddPrefabPostInit("carrat", function(inst)
    if not TheWorld.ismastersim then
        return inst
    end
    local stats = inst.components.yotc_racestats
    if stats then
        stats:ModifySpeed(1)             -- 加 1 点速度（不超 5）
        stats:AddRandomPointSpread(2)    -- 再随机撒 2 点
        stats:SaveCurrentStatsAsBaseline()
    end
end)
```

- 调整与验证：查最强维用 `GetBestStats()`（返回维度下标数组，SGcarrat 据此选自豪态，[消费](../../../raw/current/scripts/stategraphs/SGcarrat.lua#L424)）；查总点数用 `GetNumStatPoints()`（体重秤与起点用，[消费](../../../raw/current/scripts/prefabs/yotc_carrat_scale.lua#L264)）。

## 常见陷阱与限制

- **退化依赖基线快照**：忘记 `SaveCurrentStatsAsBaseline` 时 `DegradePoints` 以 `INIT_STAT_VALUE(0)` 为底，会把出生撒的点也退掉；原版在撒点后立即存基线。
- **无同步**：组件在双端各自实例化，数值只服务主机行为与本地 ghost racer；不要在客户端读它做展示。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。覆盖组件全文、carrat 撒点／训练／装配段、SGcarrat 与体重秤消费行、RACE_STATS／CARRAT_GYM 调 tuning 段；未覆盖 yotc_racecompetitor 的比赛内消费。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、Lua 语法探测通过（见批级记录）。

相关：[[reference/components/yotc_racestart]]（赛跑起点）、[[reference/components/yotc_raceprizemanager]]（赛程与奖品）。
