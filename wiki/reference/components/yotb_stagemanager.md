---
title: "yotb_stagemanager"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "牛年（YOTB）舞台赛全局协调器：登记所有 yotb_stage 舞台、单槽管理时装比赛进行态，按日重启报名并协调漫游商人显隐。"
aliases: ["牛年舞台管理", "yotb_stagemanager", "时装比赛协调", "YOTB 竞赛"]
tags: [component, lifecycle, callback]
dst_build: "steam-buildid:26230496"
sources: ["components/yotb_stagemanager.lua", "components/yotb_stager.lua", "prefabs/yotb_stage.lua", "prefabs/wanderingtrader.lua", "prefabs/cave.lua", "prefabs/forest.lua", "actions.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# yotb_stagemanager

## 快速判断

- 职责与适用：森林与洞穴世界都安装的世界组件；维护舞台列表与 `contest_enabled`/`contest_active` 单槽状态，逐日自动开赛，比赛期间只允许 active_stage 一台舞台。[双安装](../../../raw/current/scripts/prefabs/cave.lua#L545) [构造](../../../raw/current/scripts/components/yotb_stagemanager.lua#L1-L31)
- 优先使用：查询赛况用 `IsContestActive()`／`GetActiveStage()`；开关赛用 `EnableContest()`（原版入口 `YOTB_STARTCONTEST` 行动也先查它）。[行动门禁](../../../raw/current/scripts/actions.lua#L5599-L5606) [开赛](../../../raw/current/scripts/components/yotb_stagemanager.lua#L131-L145)
- 重要副作用：监听舞台建/毁（`yotb_onstagebuilt`、onremove）、比赛开始/检查点/结束（`conteststarted`/`contestcheckpoint`/`yotb_onabortcontest`/`yotb_oncontestfinshed`）与 `wanderingtrader_created`；世界事件 `yotb_contestenabled/started/finished` 由本组件推。[接线](../../../raw/current/scripts/components/yotb_stagemanager.lua#L10-L30)
- 修改边界：事件名 `yotb_oncontestfinshed` 是原版拼写（缺 e），模组监听必须原样使用；比赛结束事件由 yotb_stager 推，`contestfinished`（舞台侧事件）的监听被源码注释停用。[拼写](../../../raw/current/scripts/components/yotb_stager.lua#L1142) [停用监听](../../../raw/current/scripts/components/yotb_stagemanager.lua#L78)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `IsContestActive()`／`GetActiveStage()` | `-> boolean`／`-> stage 或 nil` | 单槽进行态与当前舞台；开始时其他舞台被 `yotb_stager:DisableContest()`。[查询](../../../raw/current/scripts/components/yotb_stagemanager.lua#L105-L107) [独占](../../../raw/current/scripts/components/yotb_stagemanager.lua#L147-L160) |
| `EnableContest()` | `-> nil` | 置 enabled、逐台 `yotb_stager:EnableContest()`、推 `yotb_contestenabled`；进行中调用直接返回。[实现](../../../raw/current/scripts/components/yotb_stagemanager.lua#L131-L145) [舞台侧](../../../raw/current/scripts/components/yotb_stager.lua#L389-L403) |
| `OnNewDay()` | 世界 cycles 回调 | 非进行且未启用时开赛（逐日自动重开）。[实现](../../../raw/current/scripts/components/yotb_stagemanager.lua#L67-L71) |
| `SetHostVisible(visible)`／`GetHostVisible()` | `visible: boolean` | 控制漫游商人显隐：对所有登记的 wanderingtrader 推 `wanderingtrader_hide/show`；`YOTB_STARTCONTEST` 用 GetHostVisible 拒绝商人身边的报名。[实现](../../../raw/current/scripts/components/yotb_stagemanager.lua#L113-L121) [商人注册](../../../raw/current/scripts/prefabs/wanderingtrader.lua#L573) |
| `OnContestCheckPoint(stage)` | 内部回调 | 舞台检查点置 `save_contest = true`，存档时进行态只有带检查点才会保存。[实现](../../../raw/current/scripts/components/yotb_stagemanager.lua#L101-L103) |
| `OnSave`／`LoadPostPass` | 标准存读钩子 | 存 enabled/active 与舞台 GUID 列表（ents）；载入重建舞台并恢复状态，enabled 且未进行则重新 EnableContest。[实现](../../../raw/current/scripts/components/yotb_stagemanager.lua#L175-L209) |

## 典型接入

### 模组查询/控制牛年比赛

- 前提与选择：主模拟端访问 `TheWorld.components.yotb_stagemanager`（森林与洞穴各自有实例）；改状态走 EnableContest/事件链，不要直接写 contest_enabled 字段（绕过舞台侧通知）。[舞台通知](../../../raw/current/scripts/components/yotb_stagemanager.lua#L138-L142)

```lua
local function IsYotbBusy()
    local mgr = TheWorld.components.yotb_stagemanager
    return mgr ~= nil and mgr:IsContestActive()
end
```

- 调整与验证：舞台实体由 `yotb_stage` prefab 建好后推 `yotb_onstagebuilt` 注册；自定义舞台需实现 `yotb_stager` 组件（EnableContest/DisableContest 被直接调用）。[舞台注册](../../../raw/current/scripts/prefabs/yotb_stage.lua#L126) [舞台组件依赖](../../../raw/current/scripts/components/yotb_stagemanager.lua#L81-L83)

## 常见陷阱与限制

- **双世界实例：** 森林和洞穴各有一个 stagemanager，各自登记本地舞台；跨分片查询是两份状态。[双安装](../../../raw/current/scripts/prefabs/forest.lua#L697)
- **进行态存档有检查点门槛：** `OnSave` 的 `data.contest_active = self.contest_active and self.save_contest`——比赛没到检查点就退出，重进世界不算进行中。[门槛](../../../raw/current/scripts/components/yotb_stagemanager.lua#L180)
- **enabletask 是死引用：** OnSave 里 `self.enabletask ~= nil` 的判断字段从未被赋值，恒为 nil；读作 `data.contest_enabled = self.contest_enabled` 即可。[死项](../../../raw/current/scripts/components/yotb_stagemanager.lua#L179)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（211 行），核验双世界安装点、yotb_stager/yotb_stage/wanderingtrader 协作与 YOTB_STARTCONTEST 行动门禁。
- 结构检查：2026-10-28 validate_page.py 通过（本页）；validate_source_links.py --strict 本批 16 页通过。
- 游戏运行：未运行。

相关：[[reference/components/wanderingtraderspawner]]
