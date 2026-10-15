---
title: "stageactingprop"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "查理舞台（charlie_stage）的演出调度：按演员与戏服匹配剧本、驱动台词线程与 heckler 乌鸦、处理演出中断与幕间推进。"
aliases: ["stageactingprop", "查理舞台", "舞台演出", "charlie stage", "play script"]
tags: [component, callback]
dst_build: "steam-buildid:26230496"
sources: ["components/stageactingprop.lua", "prefabs/charlie_stage.lua", "actions.lua", "componentactions.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# stageactingprop

## 快速判断

- 职责与适用：charlie_stage 实体的组件（master 端），从 `play_generalscripts` 预载通用剧本、接收 playbill 讲台的 `AddPlay`，按到场演员（stageactor 标签）与其头/身装备匹配戏服跑演出；不管理讲台与观众生成。[安装](../../../raw/current/scripts/prefabs/charlie_stage.lua#L604-L608) [通用剧本预载](../../../raw/current/scripts/components/stageactingprop.lua#L22-L24)
- 优先使用：启动演出走动作系统（PLAY 动作→`DoPerformance(doer)`）；换幕用 `FinishAct(next_act)`（同步 playbill_lecturn）；开关用 `EnableProp`/`DisableProp(time)`。[动作入口](../../../raw/current/scripts/actions.lua#L1593-L1597)
- 重要副作用：演出期间舞台加 `play_in_progress` 与 `NOCLICK` 标签、起线程逐行 Sleep 说台词、推 `play_begun`/`play_ended`/`play_performed` 事件；结束 2 秒后才移除进度标签。演员卸装或离台会被监听中断整场演出。[启动](../../../raw/current/scripts/components/stageactingprop.lua#L287-L320) [中断监听](../../../raw/current/scripts/components/stageactingprop.lua#L303-L306)
- 修改边界：非演员（无 stageactor 标签）右键不出现动作；演出线程用 scheduler 任务（`ClearPerformance` 强杀）；组件存档只存 DisableProp 的恢复时间，演出状态不存。[清场](../../../raw/current/scripts/components/stageactingprop.lua#L278-L285) [存读](../../../raw/current/scripts/components/stageactingprop.lua#L434-L447)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `AddGeneralScript(name, script_data)` | — | 注册通用剧本（构造时从 play_generalscripts 全量导入）；重名断言失败。script_data 需 `cast`（角色/戏服列表）与 `lines`。[注册](../../../raw/current/scripts/components/stageactingprop.lua#L31-L36) |
| `AddPlay(playdata)` | — | 由 playbill 讲台调用：整体替换 `costumes` 戏服表、合并通用剧本与该剧专属剧本、设置 `current_act`。传 nil costumes 字段会置空报错风险需调用方保证。[换剧](../../../raw/current/scripts/components/stageactingprop.lua#L38-L54) |
| `CollectCast(doer)`（内部） | — | 5.5 距离找 stageactor 标签实体（排除 fire/burnt），按头+身装备在 costumes 里找戏服名做键；发起者无戏服则独占 MONOLOGUE；同戏服重复出现记 `REPEAT_COSTUMES`。[选角](../../../raw/current/scripts/components/stageactingprop.lua#L107-L130) |
| `FindScript(doer)`（内部） | — | 对每个可用剧本（playbill 剧本只认 current_act）数"角色被选角覆盖"的数量，全覆盖者按角色数取最大并随机挑一；无匹配记 `NO_SCRIPT`，已有问题则直接返回问题键。[选剧本](../../../raw/current/scripts/components/stageactingprop.lua#L132-L178) |
| `DoPerformance(doer)` | → bool | 演出总入口：防重入（已有 play_in_progress 返回 false）→ CollectCast → FindScript → 起线程 DoLines、给演员加 acting 标签/SetStage/挂卸装与换状态监听、推 `play_begun` 并调 `onperformancebegun` 回调。[启动](../../../raw/current/scripts/components/stageactingprop.lua#L287-L320) |
| `DoLines`（线程体） | — | 逐行：lucytest/treetest 可跳过；actionfn 特效回调、talker Say 台词、给演员推 `perform_do_next_line` 动画数据、Sleep 行时长；结束推 `play_performed {next, error, skip_hound_spawn}`，有 next 则 FinishAct 换幕后 EndPerformance。[台词线程](../../../raw/current/scripts/components/stageactingprop.lua#L341-L393) |
| `EndPerformance(doer?)`／`ClearPerformance(doer?)` | — | End 清标签、退场台词（观众 heckler 台词池）、关灯罩、放乌鸦、清 cast/script；Clear 额外强杀线程。中断（卸装/换状态离位）走同一出口。[结束](../../../raw/current/scripts/components/stageactingprop.lua#L240-L285) |
| `FinishAct(next_act)` | — | 找 entitytracker 里的 lecturn 调 `playbill_lecturn:ChangeAct(next_act)` 并更新 current_act；无讲台时只改自身。[换幕](../../../raw/current/scripts/components/stageactingprop.lua#L395-L401) |
| `EnableProp()`／`DisableProp(time?)` | — | 开关 `stageactingprop` 标签并调 enablefn/dissablefn；带 time 时启动组件定时恢复（Hammer 拆走舞台后延迟复位即用它）。[开关](../../../raw/current/scripts/components/stageactingprop.lua#L56-L72) [延迟复位](../../../raw/current/scripts/prefabs/charlie_stage.lua#L182-L183) |
| `SpawnBirds(arch?)` | — | 在拱门符号上生成两只 `charlie_heckler` 乌鸦（sound_set a/b）；stage 侧在可演出时调用。[乌鸦](../../../raw/current/scripts/components/stageactingprop.lua#L403-L417) [调用点](../../../raw/current/scripts/prefabs/charlie_stage.lua#L295 |
| `OnSave`／`LoadPostPass` | time | 只存/恢复 DisableProp 的剩余时间；演出中存档会丢失整场演出状态。[存读](../../../raw/current/scripts/components/stageactingprop.lua#L434-L447) |

## 典型接入

### 用模组剧本扩展舞台演出

- 前提与选择：剧本数据结构与 play_generalscripts 一致（cast 角色名=戏服键、lines 数组带 line/anim/duration/actionfn 等）；模组在 stage 实体 postinit 后 `AddGeneralScript` 追加，或构造 playbill 讲台走 `AddPlay`。演员戏服通过头/身装备 prefab 名匹配 costumes 表。[数据来源](../../../raw/current/scripts/components/stageactingprop.lua#L4-L5)
- 生效与撤销：演出由 PLAY 动作触发（doer 需 stageactor 标签，如已变身小飞虫等演员态）；中断撤销靠组件挂的 unequip/newstate 监听自动 ClearPerformance，模组无需手动管理。
- 原版依据：componentactions 里对 stageactingprop 的右键过滤（有 play_in_progress 时不给动作）与 actions 里 PLAY 的执行就是全部官方入口。[动作过滤](../../../raw/current/scripts/componentactions.lua#L857-L858)

```lua
-- 服务器端：给已存在的查理舞台追加一个独白剧本（需要 MONOLOGUE 兼容的单人cast）
AddPrefabPostInit("charlie_stage", function(inst)
    if not TheWorld.ismastersim then
        return
    end
    inst:DoTaskInTime(0, function()
        local stage = inst.components.stageactingprop
        if stage == nil then
            return
        end
        stage:AddGeneralScript("my_monologue", {
            cast = { "MONOLOGUE" },
            lines = {
                { line = "Hello stage!", duration = 2 },
            },
        })
    end)
end)
```

- 调整与验证：cast 精确等于 `{ "MONOLOGUE" }` 的剧本在 FindScript 中用发起者兜底选角（cast 表被替换成 MONOLOGUE 单项）；复杂多角色剧本需真实戏服演员到场。[独白兜底](../../../raw/current/scripts/components/stageactingprop.lua#L166-L171)

## 常见陷阱与限制

- **戏服只认头+身两槽的精确匹配：** FindCostume 要求 head 与 body 同时匹配才返回戏服；只对一半时记 `BAD_COSTUMES` 并让 FindScript 直接返回问题键，演出不开。[匹配](../../../raw/current/scripts/components/stageactingprop.lua#L82-L96)
- **演员中断窗口很宽：** unequip（卸任一装备）与 newstate（SG 换状态且未在 acting/移动目标不符）都会 ClearPerformance；这意味着演出中任何换装、被打、做动作都会掐掉整场。[中断监听](../../../raw/current/scripts/components/stageactingprop.lua#L180-L220)
- **AddPlay 不做 nil 防御：** `playdata.costumes` / `playdata.scripts` 为 nil 会直接 pairs 报错；自定义讲台侧要保证字段齐全。[换剧](../../../raw/current/scripts/components/stageactingprop.lua#L40-L51)
- **play_performed 的 error 字段是字符串键：** `NO_SCRIPT`/`BAD_COSTUMES`/`REPEAT_COSTUMES` 都通过 FindScript 返回值传递，消费方按字符串比较；不存在问题码枚举表。[问题传递](../../../raw/current/scripts/components/stageactingprop.lua#L386
- **演出状态不进存档：** 存档时若在演出中，重载后 cast/script 全部为 nil、线程消失，但 `play_in_progress` 标签也不存在（标签不存档），舞台回到可演出态。[存读](../../../raw/current/scripts/components/stageactingprop.lua#L434-L439)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取 stageactingprop.lua，核验剧本注册/换剧、选角与选剧本、演出线程、中断协议、结束流程、乌鸦与存读；核对 charlie_stage 安装与回调注入、PLAY 动作与 componentactions 过滤。play_generalscripts 与 play_commonfn 的具体台词数据未展开。
- 结构检查：本页通过 validate_page.py 与 validate_source_links.py（严格）；Lua 示例经 parser_probe 语法检查。
- 游戏运行：未运行；完整演出流程与中断表现未实测。

## 相关链接

- [[reference/components/stageactor]]：演员侧组件与"入座演戏"状态协议。
- [[reference/components/playbill_lecturn]]：讲台侧的剧目选择与换幕来源。
