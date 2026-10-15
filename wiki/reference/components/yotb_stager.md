---
title: "yotb_stager"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "牛年时装赛单台舞台组件：受理 YOTB_STARTCONTEST 行动后按任务队列驱动安全检查、NPC 补位、目标宣布、猜评标记、排名颁奖与奖品投掷的整段比赛演出。"
aliases: ["牛年舞台", "时装比赛舞台", "yotb_stager"]
tags: [component, lifecycle, state-machine]
size_class: complex
dst_build: "steam-buildid:26230496"
sources: ["components/yotb_stager.lua", "actions.lua", "componentactions.lua", "prefabs/yotb_stage.lua", "prefabs/yotb_beefalo_dolls.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# yotb_stager

## 快速判断

- 职责与适用：挂在 `yotb_stage` 舞台实体上的比赛执行组件（[装配](../../../raw/current/scripts/prefabs/yotb_stage.lua#L111)），只服务牛年活动期间玩家对舞台发起的 `YOTB_STARTCONTEST` 行动与牛年玩偶鉴定；全局协调（单槽、逐日重开）归 [[reference/components/yotb_stagemanager]]。
- 优先使用：开赛入口是行动 `YOTB_STARTCONTEST` → `StartContest`（[行动实现](../../../raw/current/scripts/actions.lua#L5599-L5617)）；中止比赛推舞台实体事件 `yotb_contest_abort`；判定评分读取 `GetBeefScore`（[L695-L718](../../../raw/current/scripts/components/yotb_stager.lua#L695-L718)）。
- 重要副作用：自动生成 NPC 训练师与临时牛（`persists=false`，[L317-L387](../../../raw/current/scripts/components/yotb_stager.lua#L317-L387)）、控制桩位 `hitcher`/`markable`/`workable` 状态、推 `yotb_onabortcontest`／`yotb_oncontestfinshed`（世界级事件）与 `conteststarted`／`onflourishstart`（舞台级事件）。
- 修改边界：组件无 OnSave，比赛进行态不落档（检查点逻辑在 stagemanager 侧）；队列 `self.queue` 是串行演出驱动，外部插入必须走 `yotb_advance_queue` 事件语义（[构造器](../../../raw/current/scripts/components/yotb_stager.lua#L156-L177)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `EnableContest()` | `-> nil` | 仅当 stagemanager 存在且"未在进行＋已启用＋商人不在场"才加 `yotb_contestenabled`／`yotb_conteststartable`／`nomagic` 标签并推 `contestenabled`（[L389-L401](../../../raw/current/scripts/components/yotb_stager.lua#L389-L401)）；由 stagemanager 与终局回调调用。 |
| `DisableContest()` | `-> nil` | 摘除上述标签并推 `contestdisabled`（[L403-L408](../../../raw/current/scripts/components/yotb_stager.lua#L403-L408)）。 |
| `TestStartContest(starter)` | `-> string 或 nil` | 返回失败原因：`"unsafe"`（猎犬警报／袭击中，或范围内有敌对与带目标战斗实体）、`"notime"`（距夜晚不足 `TUNING.PIG_MINIGAME_REQUIRED_TIME`）、`"notenoughposts"`（`YOTB_STAGERANGE` 内 `yotb_post` 桩 ≤3）、`"nocontestants"`（无拴牛）；nil 表示可开赛（[L433-L474](../../../raw/current/scripts/components/yotb_stager.lua#L433-L474)）。 |
| `StartContest(starter)` | `-> nil` | 令舞台 `workable:SetWorkable(false)`，检查失败则排队播报并 4 秒后复原；成功则生成 `yotb_stage_voice` 代言人、清队列、播音乐并入队 phase2（[L495-L532](../../../raw/current/scripts/components/yotb_stager.lua#L495-L532)）。 |
| `GetBeefScore(beef)` | `-> {FEARSOME,FESTIVE,FORMAL}` | 按 `skinner_beefalo:GetClothing()` 五部位（body/horn/tail/head/feet）累加 `require("yotb_costumes")` 各部位三属性分（[L695-L718](../../../raw/current/scripts/components/yotb_stager.lua#L695-L718)）。 |
| `GetClosest/GetFurthest/GetRandom(values)` | `-> string` | 点评与评委取向类别选择；构造器默认 `self.choice = "far"`（[L159](../../../raw/current/scripts/components/yotb_stager.lua#L159)、[L720-L755](../../../raw/current/scripts/components/yotb_stager.lua#L720-L755)）。 |
| `DeclareWinner()` | `-> nil` | 每头牛总分＝三类别与目标差绝对值之和，升序排出季军／亚军／冠军并逐一聚光播报、撒花，随后入队 `AwardVictors`（[L863-L971](../../../raw/current/scripts/components/yotb_stager.lua#L863-L971)）。 |
| `AwardVictors()` | `-> nil` | 先令 stagemanager `SetContestEnabled(false)`；冠军桩上的 `markable.marks` 的 `mark.doer` 即中奖玩家，`self.prizes = #self.victors` 决定奖品数（[L1066-L1115](../../../raw/current/scripts/components/yotb_stager.lua#L1066-L1115)）。 |
| `Tossprize(target, pattern, other)` | `-> nil` | `redpouch_yotb` 红包装 `prizes[min(#posts,8)][min(#victors,8)]` 个 `lucky_goldnugget` 弹射给中奖者；无中奖者路径改投 `yotb_pattern_fragment_1..3` 图纸（[L990-L1032](../../../raw/current/scripts/components/yotb_stager.lua#L990-L1032)）。 |
| `appraisedoll(doll)` | `-> nil` | 玩偶鉴定演出：`doll_values = set_data.categories[doll.category] 各属性 ×5`，走 target_thresholds 台词分档后 `Endppraisedoll` 复原（[L1229-L1261](../../../raw/current/scripts/components/yotb_stager.lua#L1229-L1261)、[L1317-L1323](../../../raw/current/scripts/components/yotb_stager.lua#L1317-L1323)）；入口是 `APPRAISE` 行动（[doll 侧](../../../raw/current/scripts/prefabs/yotb_beefalo_dolls.lua#L20-L23)、[候选门禁](../../../raw/current/scripts/componentactions.lua#L1040-L1043)）。 |
| `LoadPostPass(ents, data)` | 标准读档钩子 | 只做 `cleartimers()`，防止读档残留计时器（[L1327-L1329](../../../raw/current/scripts/components/yotb_stager.lua#L1327-L1329)）。 |

## 关键流程

### 一场比赛主链（StartContest → EndContest）

1. **开赛检查**：`StartContest` 先 `TestStartContest`，失败原因经 `Start_fail` 播报后恢复 `workable`（[L476-L493](../../../raw/current/scripts/components/yotb_stager.lua#L476-L493)）。
2. **布置（phase2/3）**：重找桩位、锁 `hitcher`，玩家牛受击即中止；按桩角度排序后对 FEARSOME/FESTIVE/FORMAL 各随机 0–25 目标值（[L558-L566](../../../raw/current/scripts/components/yotb_stager.lua#L558-L566)）；空桩 4 秒后由 `MakeRandomBeef` 补猪人／鱼人训练师与随机皮肤临时牛（[L584-L604](../../../raw/current/scripts/components/yotb_stager.lua#L584-L604)）。
3. **宣布目标 → 猜评**：`BuildSuspense` 给每头牛挂 `markable_proxy` 指向桩位、放 `yotb_post_spotlight`，玩家在约 20 秒窗口内对桩位做标记（`markable`），NPC 训练师随机乱标（[L788-L848](../../../raw/current/scripts/components/yotb_stager.lua#L788-L848)）。
4. **颁奖与奖品**：`DeclareWinner` → `AwardVictors` → `Tossprizes` → `Give_consoleprizes`（未中奖者的安慰金）→ `Give_Patterns`（每桩一份图纸），最后 `EndContest`（[L1034-L1138](../../../raw/current/scripts/components/yotb_stager.lua#L1034-L1138)）。
5. **收尾**：`EndContest` 推世界事件 `yotb_oncontestfinshed`、解锁桩位、按原因换结束语；`EndContest_phase2` 移除临时实体与代言人，2 秒后在 stagemanager 仍启用时重新 `EnableContest`（[L1140-L1225](../../../raw/current/scripts/components/yotb_stager.lua#L1140-L1225)）。

### 中止路径（yotb_contest_abort）

- 玩家牛或 NPC 被攻击／死亡（[L289-L295](../../../raw/current/scripts/components/yotb_stager.lua#L289-L295)、[L534-L539](../../../raw/current/scripts/components/yotb_stager.lua#L534-L539)）都会推 `yotb_contest_abort`；`AbortContest` 推 `yotb_onabortcontest`、清任务与队列，已有 victors 时只催领奖，否则 0.5 秒后 `EndContest(data.reason)`（[L196-L251](../../../raw/current/scripts/components/yotb_stager.lua#L196-L251)）。领奖截止计时 `prizedeadline`／`warndeadline` 分支见陷阱一节。

## 典型接入

### 模组观察/驱动一场牛年比赛

- 前提与选择：仅主模拟端；舞台列表与单槽状态从 stagemanager 取，舞台级行为才动 `yotb_stager`。监听世界事件 `yotb_oncontestfinshed`（注意原版拼写缺 e）即可感知比赛结束。

```lua
AddPrefabPostInit("world", function(inst)
    if not TheWorld.ismastersim then
        return
    end
    inst:ListenForEvent("yotb_oncontestfinshed", function(world)
        local mgr = world.components.yotb_stagemanager
        local stage = mgr ~= nil and mgr:GetActiveStage() or nil
        print("YOTB contest finished, active stage now:", stage)
    end)

    -- 需要强制作弊跳过比赛时（等价原版攻击中止路径）：
    -- local stage = TheWorld.components.yotb_stagemanager:GetActiveStage()
    -- if stage ~= nil then stage:PushEvent("yotb_contest_abort", { reason = "attack" }) end
end)
```

- 调整与验证：事件名 `yotb_oncontestfinshed` 必须逐字符照抄；比赛进行中舞台可被 `nomagic` 标签拦截法杖。自定义桩位需带 `yotb_post` 标签并配 `hitcher`＋`markable` 组件才会被 `TestStartContest` 与评分统计到（[L429](../../../raw/current/scripts/components/yotb_stager.lua#L429)、[L456](../../../raw/current/scripts/components/yotb_stager.lua#L456)）。

## 常见陷阱与限制

- **奖品截止计时器是死分支**：构造器监听 `prizedeadline`／`warndeadline` 两个 `timerdone`（[L168-L175](../../../raw/current/scripts/components/yotb_stager.lua#L168-L175)），但全库检索没有任何 `StartTimer("prizedeadline"...)` 调用；"超时不领奖收回"逻辑实际不会触发。
- **临时实体清理比较恒假**：`RemoveTempTrainer`／`RemoveTempBeef` 写的是 `trainer == self.temp_trainers`（实体与列表比较，[L300](../../../raw/current/scripts/components/yotb_stager.lua#L300)、[L310](../../../raw/current/scripts/components/yotb_stager.lua#L310)），列表项从未真正移除；实际清理依赖 `EndContest_phase2` 遍历 `self.temp_*` 表。
- **事件拼写陷阱**：结束事件是 `yotb_oncontestfinshed`（缺 e，[L1142](../../../raw/current/scripts/components/yotb_stager.lua#L1142)）；玩偶收尾函数名 `Endppraisedoll` 同为原版原样拼写（[L1317](../../../raw/current/scripts/components/yotb_stager.lua#L1317)）。
- **EnableContest 有门禁**：stagemanager 未启用、比赛进行中或漫游商人在场时调用直接返回，模组别绕过 stagemanager 强开（[L391-L394](../../../raw/current/scripts/components/yotb_stager.lua#L391-L394)）。
- **弹射物理论回调缺失**：`LaunchGameItem` 里 `item:ListenForEvent("onland")` 无回调参数（[L987](../../../raw/current/scripts/components/yotb_stager.lua#L987)），依赖奖品落地行为时不要指望该监听。
- **无 OnSave**：比赛进行态不落档，读档恢复的是 stagemanager 侧的 enabled/active 概要状态；比赛中途重启世界会回到"已启用未进行"。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。通读组件全部 1330 行（三套台词表与奖品数表为静态数据块，仅结构核对）；协作面逐点核验：yotb_stage 装配行与 onstagebuilt、YOTB_STARTCONTEST 行动实现、componentactions 的 yotb_stager 候选与 APPRAISE 门禁、yotb_beefalo_dolls 的 doappraise；stagemanager 交互结论复用 [[reference/components/yotb_stagemanager]] 已核验记录。`yotb_costumes` 数据文件未打开。页面为精选覆盖，未把全部台词／数值表逐项登记。
- 结构检查：validate_page、validate_source_links --strict 通过（见批级记录）。
- 游戏运行：未运行；完整比赛流程、NPC 补位与奖品弹射未实测。

相关：[[reference/components/yotb_stagemanager]]、[[reference/components/hitcher]]、[[reference/components/markable]]
