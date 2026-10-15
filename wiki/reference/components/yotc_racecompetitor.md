---
title: "yotc_racecompetitor：卡拉特赛跑选手逻辑"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "挂在卡拉特身上的赛跑选手组件：按检查点链寻路计圈、体力与遗忘特质、到点/超时判负，冲线后经奖品管理器领奖。"
aliases: [赛跑选手, 卡拉特赛跑, yotc_racecompetitor, 鼠鼠赛跑]
tags: [component, state-machine, callback]
dst_build: "steam-buildid:26230496"
sources: ["components/yotc_racecompetitor.lua", "prefabs/carrat.lua", "prefabs/carrat_ghostracer.lua", "components/yotc_raceprizemanager.lua", "brains/carratbrain.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# yotc_racecompetitor（卡拉特赛跑选手逻辑）

## 快速判断

- 职责与适用：服务端组件。卡拉特被加入赛道时挂载（[carrat.lua:405](../../../raw/current/scripts/prefabs/carrat.lua#L405)，幽灵选手 [carrat_ghostracer.lua:138](../../../raw/current/scripts/prefabs/carrat_ghostracer.lua#L138），状态机 `prerace→racing→postrace→raceover`（[状态字段](../../../raw/current/scripts/components/yotc_racecompetitor.lua#L4)）：起点线广播 `yotc_racebegun` 后按 20 单位内 `yotc_racecheckpoint` 标签链式找下一检查点，冲 `yotc_racefinishline` 线后经 [[reference/components/yotc_raceprizemanager]] 结算。
- 优先使用：模组自制赛道 → 摆检查点（须带 `yotc_racecheckpoint` 标签）＋起点线，让 carrat 走原版挂载；读取选手状态/距离 → `GetRaceDistance`、`racestate`（[距离](../../../raw/current/scripts/components/yotc_racecompetitor.lua#L66-L68)）。
- 重要副作用：移动即耗体力（默认 8±2，被 carrat 按种族属性改写 2~22），耗尽推 `yotc_racer_exhausted` 并进入恢复期（[体力](../../../raw/current/scripts/components/yotc_racecompetitor.lua#L258-L268)）；忘性特质会概率折返上一检查点（[折返](../../../raw/current/scripts/components/yotc_racecompetitor.lua#L182-L186)）。
- 修改边界：75 秒没到下一检查点整场作废（`AbortRace`，无安慰奖）；比赛中途睡觉即被移除组件（[睡觉移除](../../../raw/current/scripts/components/yotc_racecompetitor.lua#L60-L64)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SetRaceStartPoint(start_point_entity)` | 起点实体 | 记为首个检查点并监听起点 `yotc_racebegun` 延迟开跑（[实现](../../../raw/current/scripts/components/yotc_racecompetitor.lua#L168-L171)、[监听](../../../raw/current/scripts/components/yotc_racecompetitor.lua#L93-L98)）。 |
| `StartRace(delay)` | 秒或 nil | 重置距离/圈数/体力，找下一检查点并启动更新、置 `racing`；带 delay 时挂队列任务（[实现](../../../raw/current/scripts/components/yotc_racecompetitor.lua#L101-L124)）。 |
| `FinishRace()` | 无参 | 停更新置 `postrace`，触发并清空 `race_finished_fn`（[实现](../../../raw/current/scripts/components/yotc_racecompetitor.lua#L126-L141)）。 |
| `AbortRace(prize_table)` | 奖品表或 nil | FinishRace＋`OnAllRacersFinished(nil)`＋从奖品管理器除名（[实现](../../../raw/current/scripts/components/yotc_racecompetitor.lua#L143-L149)）。 |
| `SetRaceBegunFn/FinishedFn/OverFn(fn)` | `fn(inst)` | 三个阶段回调；组件卸载时以 nil 参数补调（[注册](../../../raw/current/scripts/components/yotc_racecompetitor.lua#L70-L80)、[卸载](../../../raw/current/scripts/components/yotc_racecompetitor.lua#L49-L54)）。 |
| `SetLateStarter(delay)`／`IsStartingLate()` | 秒／布尔 | 迟到起跑期（脑据此跑"慌张"分支）（[实现](../../../raw/current/scripts/components/yotc_racecompetitor.lua#L278-L284)）。 |
| `CollectPrize()` | 无参 → redpouch_yotc 或 nil | 有 `race_prize` 则生成红包装奖品并移除 `has_prize` 标签；无奖则摘 `has_no_prize`（[领奖](../../../raw/current/scripts/components/yotc_racecompetitor.lua#L298-L322)）。 |
| `OnUpdate(dt)` | 组件更新 | 超时/失链→Abort；到检查点推 `yotc_racer_at_checkpoint`，终点线则注册并 `FinishRace`＋`RacerFinishedRace` 记名次（[实现](../../../raw/current/scripts/components/yotc_racecompetitor.lua#L223-L256)）。 |
| `OnRemoveEntity/OnRemoveFromEntity` | 卸载钩子 | 从 yotc_raceprizemanager 除名并复位标签/回调（[卸载](../../../raw/current/scripts/components/yotc_racecompetitor.lua#L33-L58)）。 |

寻路与特质：`_FindNextCheckPoint` 在 20 单位内搜检查点（含起点线），按"访问次数（上限 5）＋终点线 0.5 减免＋上一检查点 +1"排优先级取最小，累计 `race_distance`（[寻路](../../../raw/current/scripts/components/yotc_racecompetitor.lua#L175-L221)）；carrat 按种族属性设置 `isforgetful`（方向改造为 0）与体力上下限（([carrat.lua:336-338](../../../raw/current/scripts/prefabs/carrat.lua#L336-L338)）。

## 典型接入

### 读选手状态做自定义跟踪（原版挂载路径之外）

- 前提与选择：服务端；挂载、回调与属性都由 carrat prefab 在入赛时完成，模组赛跑一般只读。
- 生效与撤销：比赛结束领奖时 prefab 调 `CollectPrize` 后 `RemoveComponent`（[领奖移除](../../../raw/current/scripts/prefabs/carrat.lua#L213-L222)）。

```lua
-- 服务端：监控一只赛跑中的卡拉特
local comp = carrat.components.yotc_racecompetitor
if comp and comp.racestate == "racing" then
    print("next:", comp.next_checkpoint, "dist:", comp:GetRaceDistance(),
          "exhausted:", comp:IsExhausted())
end
```

- 调整与验证：检查点参数（找距 20、太远 40、到点 2、超时 75、单点最多 5 次）见 ([tuning.lua:5209-5214](../../../raw/current/scripts/tuning.lua#L5209-L5214)；脑按 `racestate` 切换行为树（[脑分支](../../../raw/current/scripts/brains/carratbrain.lua#L183-L202)）。

## 常见陷阱与限制

- **比赛中睡觉即取消资格**：`OnEntitySleep` 在非 prerace/postrace/raceover 态直接 `RemoveComponent`，没有恢复手段（[睡觉移除](../../../raw/current/scripts/components/yotc_racecompetitor.lua#L60-L64)）。
- **检查点必须互相可达**：寻路只在 `YOTC_RACER_CHECKPOINT_FIND_DIST`(20) 半径内找点，间隙大于该值会 `AbortRace`（[失链判定](../../../raw/current/scripts/components/yotc_racecompetitor.lua#L229-L234)）。
- **忘性是累积概率**：每次寻路 `forgetfulness+1`，折返概率按其平方比上限 8 归一；折返后计数清零（[概率式](../../../raw/current/scripts/components/yotc_racecompetitor.lua#L182)）。
- **奖品只有第一名拿真的**：`OnAllRacersFinished` 由奖品管理器广播，第一名得 `race_prize`，其余只挂 `has_no_prize`（[结算](../../../raw/current/scripts/components/yotc_raceprizemanager.lua#L50-L51)）。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496（steam-buildid:26230496）；覆盖 `components/yotc_racecompetitor.lua` 全部 330 行、carrat.lua 挂载/属性/领奖段、carrat_ghostracer 挂载、yotc_raceprizemanager 结算调用、carratbrain 状态分支、tuning YOTC_RACER 块。未覆盖：检查点/起点线 prefab 实现、SGcarrat 全状态。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 通过；lua 示例通过 parser_probe 语法检查。
- 游戏运行：未运行。
