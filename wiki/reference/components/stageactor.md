---
title: "stageactor"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "舞台表演演员端组件：原版实际使用的是 stageactor 标签与 SetStage/GetStage 协作，剧本讲述半区（TellStory/performplay）在本快照内无原版调用且依赖缺失。"
aliases: ["舞台演员", "stageactor", "表演", "演戏", "演员组件"]
tags: [component, callback, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/stageactor.lua", "components/stageactingprop.lua", "prefabs/charlie_heckler.lua", "prefabs/player_common.lua", "prefabs/sewing_mannequin.lua", "strings_stageactor.lua", "brains/knightbrain.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# stageactor

## 快速判断

- 职责与适用：标记"舞台演员"并维护当前所在舞台；构造即加 `stageactor` 标签，组件移除时摘除。原版玩家（服务端）、Charlie Heckler、缝纫人偶都安装。[构造](../../../raw/current/scripts/components/stageactor.lua#L16-L33) [安装点](../../../raw/current/scripts/prefabs/player_common.lua#L2853)
- 优先使用：让实体参与原版舞台表演走 `stageactingprop` 的选角与 `SetStage` 协作，不要自己调 `TellStory`——后者在本快照无任何原版调用。[DoPerformance](../../../raw/current/scripts/components/stageactingprop.lua#L287-L303) [PERFORM 行动](../../../raw/current/scripts/actions.lua#L1593-L1597)
- 重要副作用：`SetStage(stage)` 被舞台侧在选角／清场时调用；`stageactingprop` 的 abortplay／costumecheck 据此读取演员归属。[清场读点](../../../raw/current/scripts/components/stageactingprop.lua#L181-L183) [服装检查](../../../raw/current/scripts/components/stageactingprop.lua#L211-L213)
- 修改边界：客户端实体只有 `stageactor` 标签（player_common 在 pristine 前加标签，服务端才挂组件）；无标签的实体不会被选角。[标签](../../../raw/current/scripts/prefabs/player_common.lua#L2548) [选角条件](../../../raw/current/scripts/componentactions.lua#L858)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SetStage(stage)`／`GetStage()` | `stage: 舞台实体或 nil` | 记录／读取当前舞台；原版由 `stageactingprop:DoPerformance` 置入、`ClearPerformance` 置 nil，knightbrain 也读它。[置入](../../../raw/current/scripts/components/stageactingprop.lua#L301) [置空](../../../raw/current/scripts/components/stageactingprop.lua#L254-L255) [读取](../../../raw/current/scripts/brains/knightbrain.lua#L254) |
| `TellStory(storyprop, story)` | `storyprop: 实体、story: 表或 nil -> boolean` | 经 talker 逐句讲台词，注册 ontick 周期任务与 donetalking 收尾，结束时向 10 距离内 stage/stagelistener 推 `play_performed`。**本快照无原版调用者**。[实现](../../../raw/current/scripts/components/stageactor.lua#L117-L162) |
| `performplay()` | `-> story 表或 nil` | 从 `STRINGS.STAGEACTOR` 抽独白并记录 `previous_acts`（存档），抽满一轮后重置。依赖缺失见陷阱，实际会报错。[实现](../../../raw/current/scripts/components/stageactor.lua#L42-L71) |
| `AbortStory(reason)` | `reason: 字符串或 nil` | 结束当前故事，reason 存在时让 talker 说一句、否则 ShutUp；仅由 storyprop 的 onremove 回调触发，同样依赖 TellStory 先行。[实现](../../../raw/current/scripts/components/stageactor.lua#L85-L94) |
| `SetOnStoryBeginFn`／`SetOnStoryOverFn` | `fn: function(inst, story)` | TellStory 首尾回调；无原版设置者。[实现](../../../raw/current/scripts/components/stageactor.lua#L73-L79) |
| `OnSave`／`OnLoad` | 标准存读钩子 | 只持久化 `previous_acts`。[实现](../../../raw/current/scripts/components/stageactor.lua#L172-L185) |

## 典型接入

### 让自定义实体具备原版表演选角资格

- 前提与选择：服务端挂组件获得标签；真正的表演由目标舞台的 `stageactingprop:DoPerformance` 驱动（选角、换装检查、台词线程），演员实体只需满足其筛选。[选角](../../../raw/current/scripts/components/stageactingprop.lua#L287-L303)
- 生效与撤销：清场时舞台侧调 `SetStage(nil)`；组件移除自动摘标签。

```lua
AddPrefabPostInit("my_creature", function(inst)
    if not TheWorld.ismastersim then
        return
    end
    inst:AddComponent("stageactor")
end)
```

- 调整与验证：客户端识别只认 `stageactor` 标签（玩家在 pristine 前加标签、人偶同法）；不要假设挂组件后自动开始表演，还要有 stageactingprop 舞台与剧本数据。[标签先行](../../../raw/current/scripts/prefabs/sewing_mannequin.lua#L177-L178)

## 常见陷阱与限制

- **performplay 在本快照必然报错：** 它 `deepcopy(STRINGS.STAGEACTOR["GENERIC"])`，但 strings_stageactor.lua 顶层键全是剧本名（WILSON1 等），全库 grep 无 GENERIC 键；`deepcopy(nil)` 返回 nil 后对 nil 调 `pairs` 抛错。因此依赖它的 TellStory 自动选本路径不可用。[取串](../../../raw/current/scripts/components/stageactor.lua#L42-L53) [串表结构](../../../raw/current/scripts/strings_stageactor.lua#L1-L5) [deepcopy(nil)](../../../raw/current/scripts/util.lua#L906-L910)
- **TellStory/performplay/AbortStory 无原版消费者：** 原版表演（A Little Drama）由 `stageactingprop` 的剧本线程直接推台词，不经过本组件的讲述半区；`IsTellingStory` 的外部调用都是 storyteller 组件的。[行动入口](../../../raw/current/scripts/actions.lua#L1593-L1597)
- ** STRINGS.STAGEACTOR 与 strings_stageactor.lua 的关系：** strings.lua 把该模块整体赋给 `STRINGS.STAGEACTOR`，组件里按 `string.upper(prefab)` 找同名字段也找不到（键是 WILSON1 而非 WILSON），自定义玩家独白需自行补表。[赋值](../../../raw/current/scripts/strings.lua#L17309)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（187 行），核验 stageactingprop 协作、三个原版安装点、strings_stageactor 结构与 TellStory 全库调用情况。
- 结构检查：2026-10-28 validate_page.py 通过（本页）；validate_source_links.py --strict 本批 16 页通过。
- 游戏运行：未运行。

相关：[[reference/components/storyteller]]、[[reference/components/talker]]
