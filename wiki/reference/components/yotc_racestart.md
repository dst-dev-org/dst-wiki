---
title: "yotc_racestart"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "卡拉特赛跑起点柱的赛跑开关：START_CARRAT_RACE 动作调 StartRace，比赛结束调 EndRace，用 race_on tag 表达状态。"
aliases: ["赛跑起点组件", "卡拉特比赛开始", "yotc_racestart"]
tags: [component, callback]
dst_build: "steam-buildid:26230496"
size_class: simple
sources: ["components/yotc_racestart.lua", "prefabs/yotc_carrat_race_start.lua", "actions.lua", "componentactions.lua", "prefabs/rat_gym.lua", "prefabs/carrat.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# yotc_racestart

## 快速判断

- 职责与适用：YOTC 卡拉特赛跑起点（`yotc_carrat_race_start`）与 `rat_gym` 的比赛状态组件：`StartRace()`／`EndRace()` 切换 `race_on` tag 并回调宿主钩子；`CanInteract()` 供动作层判断"能否再开一场"。宿主带 `yotc_racestart` tag 供卡拉特搜索起点（[tag](../../../raw/current/scripts/components/yotc_racestart.lua#L8)、[搜索](../../../raw/current/scripts/prefabs/carrat.lua#L313)）。
- 优先使用：动作与界面通过 `CanInteract()` 判断、`StartRace()` 启动；宿主 prefab 负责设置 `onstartracefn`／`onendracefn` 回调（[宿主设置](../../../raw/current/scripts/prefabs/yotc_carrat_race_start.lua#L178-L180)）。
- 重要副作用：`StartRace()` 调 `onstartracefn(inst)` 后加 `race_on` tag；`EndRace()` 调 `onendracefn(inst)` 后移除 tag；`OnRemoveFromEntity` 只移除 `yotc_racestart` tag（[L13-L15](../../../raw/current/scripts/components/yotc_racestart.lua#L13-L15)）。
- 修改边界：状态只用 tag 表达、不存档：读档后 `race_on` 由宿主的 OnSave／OnLoad 字段恢复（起点柱存 burnt，赛道状态由赛跑管理器管理）。宿主被烧毁时直接 RemoveComponent（[起点柱](../../../raw/current/scripts/prefabs/yotc_carrat_race_start.lua#L112-L116)、[rat_gym](../../../raw/current/scripts/prefabs/rat_gym.lua#L233-L235)）。

## 关键接口

| 接口／字段 | 已核验类型 | 条件、作用与证据 |
|---|---|---|
| `CanInteract()` | true 或 nil | 无 `race_on` tag 返回 true，否则返回 nil（无显式 false）。[L31-L35](../../../raw/current/scripts/components/yotc_racestart.lua#L31-L35) |
| `StartRace()` | 无返回 | 先调 `onstartracefn`（已设置时）再加 tag。[L17-L22](../../../raw/current/scripts/components/yotc_racestart.lua#L17-L22) |
| `EndRace()` | 无返回 | 调 `onendracefn`（构造未初始化，必须由宿主赋值）后移除 tag。[L24-L29](../../../raw/current/scripts/components/yotc_racestart.lua#L24-L29) |
| `onstartracefn`／`onendracefn` | `fn(inst)` 或 nil | 回调字段，宿主赋值；构造只初始化了 `onstartracefn`。[L5-L6](../../../raw/current/scripts/components/yotc_racestart.lua#L5-L6)、[赋值](../../../raw/current/scripts/prefabs/yotc_carrat_race_start.lua#L179-L180) |

## 典型接入

### 原版动作链

- 右键起点柱时 `componentactions` 按 `yotc_racestart` 组件插入 `ACTIONS.START_CARRAT_RACE`，排除 burnt／fire／race_on（[componentactions.lua#L1031-L1034](../../../raw/current/scripts/componentactions.lua#L1031-L1034)）。
- 动作执行再做三重检查：组件存在、`CanInteract()`、以及赛跑管理器报名的 `num_racers > 0`，通过才 `StartRace()`；结束后 `OnRaceOver` 调 `EndRace()`（[actions.lua#L4696-L4709](../../../raw/current/scripts/actions.lua#L4696-L4709)、[OnRaceOver](../../../raw/current/scripts/prefabs/yotc_carrat_race_start.lua#L128-L129)）。

## 常见陷阱与限制

- `self.rats = {}` 在构造里声明但全库无任何读写（grep 仅命中声明行），不要依赖它收集参赛鼠。
- `EndRace()` 读 `onendracefn`，而构造没有该字段：自定义宿主漏赋值时比赛结束会静默跳过回调（tag 仍会移除）；`CanInteract()` 返回 nil 而非 false，用 `not CanInteract()` 判断时注意语义一致。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。完整阅读 components/yotc_racestart.lua（38 行），核对起点柱宿主的组件添加与回调赋值、烧毁清理、动作链（componentactions／actions）、rat_gym 与 carrat 的 tag 消费。未覆盖比赛全程状态机（yotc_raceprizemanager 侧）。
- 游戏运行：未运行。
- 结构检查：validate_page.py 通过（0 issue；51 行、4576 字节，simple 档内）；validate_source_links.py --strict 通过（14 处源码引用，0 error／0 warning）；本页无完整 Lua 代码块。
