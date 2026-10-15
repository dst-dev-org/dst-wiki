---
title: "growable"
created: "2026-10-12"
updated: "2026-10-28"
type: component
summary: "配置多阶段成长、阶段回调与暂停恢复，修改树成长速度时隔离共享阶段表。"
aliases: ["阶段成长", "树成长速度", "成长暂停", "成长阶段", "growable"]
tags: [component, mechanism, callback, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/growable.lua", "prefabs/evergreens.lua", "mainfunctions.lua", "entityscript.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# growable

## 快速判断

- 职责与适用：多个阶段的时长、进入阶段及自然成长；采收产物／采后再生用 [[reference/components/pickable]]，两者不是同一计时。[growable.lua:10](../../../raw/current/scripts/components/growable.lua#L10 [growable.lua:121](../../../raw/current/scripts/components/growable.lua#L121
- 优先使用：先配置 stages，再 SetStage 和 StartGrowing；单纯换阶段不会开始计时。修改原树阶段时先复制共享表。[growable.lua:42](../../../raw/current/scripts/components/growable.lua#L42 [growable.lua:207](../../../raw/current/scripts/components/growable.lua#L207 [evergreens.lua:382](../../../raw/current/scripts/prefabs/evergreens.lua#L382
- 重要副作用：DoGrowth 的目标阶段 pregrowfn → SetStage／fn → growfn → 继续计时；加载和离屏追赶不是每步完整自然回调。[growable.lua:121](../../../raw/current/scripts/components/growable.lua#L121 [growable.lua:254](../../../raw/current/scripts/components/growable.lua#L254 [growable.lua:273](../../../raw/current/scripts/components/growable.lua#L273
- 修改边界：本页植物服务端配置；不要把离屏当暂停，或把阶段 time 写成数字。新植物的资源／工厂、播种／采收还需对应接入。[evergreens.lua:866](../../../raw/current/scripts/prefabs/evergreens.lua#L866 [evergreens.lua:893](../../../raw/current/scripts/prefabs/evergreens.lua#L893

## 关键接口

| 接口／字段 | 参数或返回 | 条件、作用与证据 |
|---|---|---|
| `stages` | 阶段数组；构造默认 nil | StartGrowing 假设表已存在；空表仅打印返回，nil 会在取长度时出错。[growable.lua:13](../../../raw/current/scripts/components/growable.lua#L13 [growable.lua:45](../../../raw/current/scripts/components/growable.lua#L45 |
| 阶段 `time`／`fn`／`pregrowfn`／`growfn` | time `(inst, stage, stagedata) -> 秒数或 nil`；其余自然路径同形参 | time 缺失回退 10 秒，返回 nil 停止；fn 处理阶段状态，growfn 处理自然成长额外效果。离屏最终 growfn 只传 inst。[growable.lua:57](../../../raw/current/scripts/components/growable.lua#L57 [growable.lua:130](../../../raw/current/scripts/components/growable.lua#L130 [growable.lua:308](../../../raw/current/scripts/components/growable.lua#L308 |
| `SetStage(stage)`／`GetStage()` | 阶段整数 | 上限截到数组长度，不保护小于 1；SetStage 调 fn，不调 growfn 或重排任务。[growable.lua:103](../../../raw/current/scripts/components/growable.lua#L103 [growable.lua:207](../../../raw/current/scripts/components/growable.lua#L207 |
| `StartGrowing(time?)`／`StopGrowing()` | 指定秒数或当前阶段 time | StartGrowing 先停止旧计时，springgrowth 时仍应用季节；Stop 清剩余／任务，但不清 pausereasons。[growable.lua:42](../../../raw/current/scripts/components/growable.lua#L42 [growable.lua:81](../../../raw/current/scripts/components/growable.lua#L81 |
| `DoGrowth(skipgrownfn?)` | fully stopped 时 false，否则 true | 正在跑或已暂停都可推进；skip 仅跳 growfn，仍执行 pregrowfn／fn；growonly 会跳 SetStage。[growable.lua:121](../../../raw/current/scripts/components/growable.lua#L121 |
| `loopstages`／`loopstages_start` | 循环标志／起始阶段，未初始化为 false／1 | 越过末阶段才按循环回起点，默认起点按 1 处理；不循环自然成长在末阶段停。[growable.lua:17](../../../raw/current/scripts/components/growable.lua#L17 [growable.lua:93](../../../raw/current/scripts/components/growable.lua#L93 |
| `Pause(reason)`／`Resume(reason)` | 建议唯一字符串原因；恢复成功 true，否则无显式返回 | 保存剩余，Resume 只移除本原因，所有原因移除才恢复；直接 StartGrowing／DoGrowth 可绕过原因集合。[growable.lua:155](../../../raw/current/scripts/components/growable.lua#L155 [growable.lua:177](../../../raw/current/scripts/components/growable.lua#L177 |
| `ExtendGrowTime(extra)`／`IsGrowing()`／`IsPaused()` | 秒增量／boolean／boolean | 延长当前剩余或暂停剩余；完全停止无效。Growing 查 targettime，Paused 查 pausedremaining。[growable.lua:111](../../../raw/current/scripts/components/growable.lua#L111 [growable.lua:192](../../../raw/current/scripts/components/growable.lua#L192 |

## 生命周期

宿主先调用实体睡眠／唤醒回调，再分发组件钩子。默认 growable 的 OnEntitySleep 取消 task、记录睡眠时刻，OnEntityWake 用 LongUpdate 追赶，时间仍流逝；growoffscreen 才保留离屏任务。LongUpdate 可跨多个阶段，跳中间 growfn，最后按条件只调用最终 growfn，不能用它保证每阶段奖励一次。[mainfunctions.lua:696](../../../raw/current/scripts/mainfunctions.lua#L696 [mainfunctions.lua:725](../../../raw/current/scripts/mainfunctions.lua#L725 [growable.lua:273](../../../raw/current/scripts/components/growable.lua#L273 [growable.lua:313](../../../raw/current/scripts/components/growable.lua#L313

OnSave 保存 stage／剩余／睡眠时间，不保存 pause 原因；OnLoad 先 SetStage 再 StartGrowing(data.time)，暂停不自动恢复为暂停，需所属玩法重新施加。移组件的 OnRemoveFromEntity 等于 StopGrowing；宿主 RemoveComponent 调此钩子。[growable.lua:227](../../../raw/current/scripts/components/growable.lua#L227 [growable.lua:250](../../../raw/current/scripts/components/growable.lua#L250 [growable.lua:365](../../../raw/current/scripts/components/growable.lua#L365 [entityscript.lua:670](../../../raw/current/scripts/entityscript.lua#L670

## 典型接入

### 常绿树未来阶段时长减半并保留原版效果

目标模组 `modmain.lua`；原版普通常绿树各高度工厂将实例名设 evergreen，PostInit 按实例名及原注册名分发，因此此入口也覆盖普通高／中／矮变体，不包含 sparse／twiggy。阶段数组及每项表共享，先分别浅复制，保留 time 以外字段和回调。[evergreens.lua:62](../../../raw/current/scripts/prefabs/evergreens.lua#L62 [evergreens.lua:348](../../../raw/current/scripts/prefabs/evergreens.lua#L348 [evergreens.lua:855](../../../raw/current/scripts/prefabs/evergreens.lua#L855 [evergreens.lua:989](../../../raw/current/scripts/prefabs/evergreens.lua#L989 [mainfunctions.lua:392](../../../raw/current/scripts/mainfunctions.lua#L392

```lua
AddPrefabPostInit("evergreen", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local growable = inst.components.growable
    if growable == nil or growable.stages == nil then return end
    local stages = {}
    for i, original in GLOBAL.ipairs(growable.stages) do
        local stage = {}
        for key, value in GLOBAL.pairs(original) do
            stage[key] = value
        end
        local original_time = original.time
        if original_time ~= nil then
            stage.time = function(ent, number, data)
                local seconds = original_time(ent, number, data)
                return seconds ~= nil and seconds / 2 or nil
            end
        end
        stages[i] = stage
    end
    growable.stages = stages
end)
```

原工厂已 StartGrowing，本例不重排当前阶段，**从以后读取阶段 time 时生效**；已存剩余也不直接减半。保留随机时长、春季调整、循环、工作量／外观 fn 及 growfn；树桩／烧毁无 growable 时跳过。停用后新初始化恢复原表，已有剩余不回滚。[evergreens.lua:893](../../../raw/current/scripts/prefabs/evergreens.lua#L893 [evergreens.lua:964](../../../raw/current/scripts/prefabs/evergreens.lua#L964 [growable.lua:267](../../../raw/current/scripts/components/growable.lua#L267

新增成长对象可按原版“fn 改阶段状态、growfn 改成长效果”分工；不复制树专用回调或强制在每次读档发奖励。新增完整植物工厂尚不在本例覆盖。[evergreens.lua:246](../../../raw/current/scripts/prefabs/evergreens.lua#L246 [evergreens.lua:348](../../../raw/current/scripts/prefabs/evergreens.lua#L348

## 常见陷阱与限制

- **直接 stages[i].time=数字或改共享项**：StartGrowing 会调用 time；共享表会影响其他实例。按案例复制数组及每项，time 必须是函数。[growable.lua:62](../../../raw/current/scripts/components/growable.lua#L62 [evergreens.lua:382](../../../raw/current/scripts/prefabs/evergreens.lua#L382
- **暂停自动跨存档，或 SetStage 就立即成熟**：Pause 原因没保存；SetStage 只调阶段 fn。重施玩法暂停；推进自然路径前确认已运行或暂停的计时状态。[growable.lua:207](../../../raw/current/scripts/components/growable.lua#L207 [growable.lua:227](../../../raw/current/scripts/components/growable.lua#L227
- **magicgrowable=true 就能魔法催熟**：DoMagicGrowth 只委托 domagicgrowthfn；树另有 simplemagicgrower，不能仅靠标志补造实现。[growable.lua:117](../../../raw/current/scripts/components/growable.lua#L117 [evergreens.lua:901](../../../raw/current/scripts/prefabs/evergreens.lua#L901

## 验证记录

- 版本增量复核 26148036（2026-10-25）：锚点重定位 3 处（mainfunctions 实体睡／醒分发 ×2 与 PostInit 定名处）；无结论修订，仅锚点迁移——依据：全局枢纽 diff 审阅（mainfunctions R-09 四处改动均在错误界面／落点保护／世界标签，OnEntitySleep／OnEntityWake 分发体逐行同旧版）＋变更清单命中 mainfunctions.lua、entityscript.lua（本版湿润／距离函数改动不涉组件分发钩子）。
- 源码审阅：2026-10-12，Steam buildid 25643519；覆盖阶段／回调、计时／暂停、离屏追赶、存档和常绿树共享表／注册分发与发布案例。农作物、催熟组件完整链及所有树改型未覆盖。
- 结构检查：元数据、证据行、链接／章节、大小、登记通过；发布 Lua 与植物调参组合经 Full Moon 3.0.0／Lua 5.1 语法及原字节往返检查。
- 游戏运行：未运行；各阶段动画／工作量、离屏／春季、读档与其他模组改表待实测。

相关：[[reference/components/pickable#关键接口]]；[[reference/components/workable#工作顺序与原版完成位置]]。
