---
title: "teacher"
created: "2026-10-23"
updated: "2026-10-28"
type: component
summary: "让蓝图等一次性载体教会指定配方，保留学习门禁、成功回调和整实体消耗顺序。"
aliases: ["蓝图学习", "一次性配方教学", "读蓝图奖励", "渔具图纸", "teacher"]
tags: [component, recipe, callback, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/teacher.lua", "components/builder.lua", "prefabs/blueprint.lua", "prefabs/tacklesketch.lua", "prefabs/wx78_scanner.lua", "actions.lua", "componentactions.lua", "stategraphs/SGwilson.lua", "stategraphs/SGwilson_client.lua", "mainfunctions.lua", "entityscript.lua", "modutil.lua", "components/hunger.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# teacher

## 快速判断

- 职责与适用：挂在教学载体上，用既有配方名教会带 [[reference/components/builder]] 的目标；配方由载体配置，不从被点击实体反查。反查目标配方用 [[reference/components/recipescanner]]。[teacher.lua:6-26](../../../raw/current/scripts/components/teacher.lua#L6-L26)
- 优先使用：`SetRecipe(name)` 配置，`Teach(learner)` 走完整教学；成功附加效果包装 `onteach(inst, learner)`。组件不注册新配方。[teacher.lua:6-26](../../../raw/current/scripts/components/teacher.lua#L6-L26)
- 重要副作用：成功依次调用 `UnlockRecipe`、`onteach`、载体 `Remove()`；默认有学习精神奖励。空 `recipe` 也删除载体，但返回 false。[teacher.lua:10-26](../../../raw/current/scripts/components/teacher.lua#L10-L26)、[builder.lua:463-471](../../../raw/current/scripts/components/builder.lua#L463-L471)
- 修改边界：原版蓝图在服务端挂组件。`Teach` 不负责目标存活检查、客户端提示或分堆；直接移除整个载体，不能当成自动扣一层耐久／一份堆叠物。[blueprint.lua:127-148](../../../raw/current/scripts/prefabs/blueprint.lua#L127-L148)、[teacher.lua:10-26](../../../raw/current/scripts/components/teacher.lua#L10-L26)

## 关键接口

| 接口／字段 | 签名或约定 | 条件、作用与证据 |
|---|---|---|
| `SetRecipe(recipe)`／`recipe` | 已注册配方名；初始 nil | 仅赋值，不校验、不同步显示名、不存档。[teacher.lua:1-7](../../../raw/current/scripts/components/teacher.lua#L1-L7) |
| `Teach(target)` | `true` 或 `false, reason?` | 无 builder 返回 false；已知返回 `false, "KNOWN"`，不能学习返回 `false, "CANTLEARN"`，这三条保留载体；空 recipe 特例删除载体。[teacher.lua:10-26](../../../raw/current/scripts/components/teacher.lua#L10-L26) |
| `onteach` | 可选 `(carrier, learner)` | 解锁后、删除前执行；不是否决回调，返回值不影响成功。组件自身不发送 `learnrecipe`，原版蓝图回调才向学习者发送该事件。[teacher.lua:21-26](../../../raw/current/scripts/components/teacher.lua#L21-L26)、[blueprint.lua:40-41](../../../raw/current/scripts/prefabs/blueprint.lua#L40-L41) |
| `KnowsRecipe(recipe, true)`／`CanLearn(name)` | 委托目标 builder | 前者接受配方名或对象，并非只查永久学习列表；后者核验有效配方、角色／技能门禁。[builder.lua:841-848](../../../raw/current/scripts/components/builder.lua#L841-L848)、[:954–969](../../../raw/current/scripts/components/builder.lua#L954-L969) |

物品栏动作和“对自己使用”入口都要求学习者有 builder replica；执行 `TEACH` 时目标为 `act.target or act.doer`，再调用载体 teacher。人物两端 SG 均将 TEACH 映射至长动作，服务端到超时才执行缓冲动作；出现操作提示不等于已经成功学习。[componentactions.lua:1838-1841](../../../raw/current/scripts/componentactions.lua#L1838-L1841)、[:2990–2993](../../../raw/current/scripts/componentactions.lua#L3002-L3005)、[actions.lua:3260-3266](../../../raw/current/scripts/actions.lua#L3260-L3266)、[SGwilson.lua:1214](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1214)、[:8352–8360](../../../raw/current/scripts/stategraphs/SGwilson.lua#L8384-L8392)、[SGwilson_client.lua:522](../../../raw/current/scripts/stategraphs/SGwilson_client.lua#L522

## 典型接入

### 成功学习长矛蓝图后，额外补充 5 点饥饿

目标模组 `modmain.lua`，对当前教会长矛的原版蓝图追加成功奖励，沿用资源、配方与教学动作。具体蓝图由配方循环注册，但共同工厂把实体名设成 `blueprint`；存档也保存这个实体名，重载先生成后恢复配方。因此统一挂 `blueprint` 的 PostInit，在教学回调运行时按当前配方判断，不在初始化时筛掉尚待恢复的蓝图。[blueprint.lua:120](../../../raw/current/scripts/prefabs/blueprint.lua#L120)、[:225–242](../../../raw/current/scripts/prefabs/blueprint.lua#L225-L242)、[entityscript.lua:248-251](../../../raw/current/scripts/entityscript.lua#L248-L251)、[mainfunctions.lua:392-404](../../../raw/current/scripts/mainfunctions.lua#L392-L404)、[:464–466](../../../raw/current/scripts/mainfunctions.lua#L464-L466)、[:503](../../../raw/current/scripts/mainfunctions.lua#L503)、[blueprint.lua:11-14](../../../raw/current/scripts/prefabs/blueprint.lua#L11-L14)、[modutil.lua:592-598](../../../raw/current/scripts/modutil.lua#L592-L598)

```lua
AddPrefabPostInit("blueprint", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local teacher = inst.components.teacher
    if teacher == nil then return end
    local old_onteach = teacher.onteach
    teacher.onteach = function(carrier, learner)
        local recipe = teacher.recipe
        if old_onteach ~= nil then
            old_onteach(carrier, learner)
        end
        if recipe == "spear" and learner:IsValid()
            and learner.components.hunger ~= nil then
            learner.components.hunger:DoDelta(5)
        end
    end
end)
```

`5` 为饥饿点，仍遵循饥饿上限、重定向及无敌等门禁；不是食物，也不保证实际增加恰好 5。原 `learnrecipe` 回调保留，其他配方不额外奖励，蓝图仍由 `Teach` 删除；KNOWN／CANTLEARN 不触发奖励。启用期间重建／重载的蓝图均重新包装，停用后不再包装，已恢复的数值不会回滚。[hunger.lua:128-143](../../../raw/current/scripts/components/hunger.lua#L128-L143)、[[reference/components/hunger#关键接口]]

### 原版载体的配方配置与恢复

- **普通／指定蓝图：** 随机入口选 `recipetouse`，指定入口由配方循环生成；宿主保存 `recipetouse`／稀有状态，并在 OnLoad 重新 `SetRecipe` 和设置显示。不要仅改 `teacher.recipe` 后声称已完成命名和读档。[blueprint.lua:11-32](../../../raw/current/scripts/prefabs/blueprint.lua#L11-L32)、[:196–201](../../../raw/current/scripts/prefabs/blueprint.lua#L196-L201)、[:225–242](../../../raw/current/scripts/prefabs/blueprint.lua#L225-L242)
- **渔具图纸：** 基础 `tacklesketch` 工厂虽设 `sketchid=1`，却没有在该处设置 teacher.recipe；具体 `<item>_tacklesketch` 工厂及 OnLoad 才设置。表中的索引被存档，不可任意重排；不能凭基础名字假定能直接教学。[tacklesketch.lua:6-20](../../../raw/current/scripts/prefabs/tacklesketch.lua#L6-L20)、[:23–35](../../../raw/current/scripts/prefabs/tacklesketch.lua#L23-L35)、[:97–103](../../../raw/current/scripts/prefabs/tacklesketch.lua#L97-L103)、[:116–139](../../../raw/current/scripts/prefabs/tacklesketch.lua#L116-L139)
- **非手读路径：** WX-78 扫描完成后的收获函数按 `_module_recipe` 配置 teacher，再直接 `Teach(picker)`；这是该宿主专用流程，不能泛化成 teacher 会自动扫描生物。[wx78_scanner.lua:1021-1027](../../../raw/current/scripts/prefabs/wx78_scanner.lua#L1021-L1027)、[:1135–1145](../../../raw/current/scripts/prefabs/wx78_scanner.lua#L1135-L1145)

## 常见陷阱与限制

- **把错误配方名等同空配置：** nil 配方立即删载体；无效非 nil 配方通常在 `CanLearn` 返回 CANTLEARN 并保留。初始化和读档都要明确有效配方名。[teacher.lua:10-25](../../../raw/current/scripts/components/teacher.lua#L10-L25)、[builder.lua:954-957](../../../raw/current/scripts/components/builder.lua#L954-L957)
- **认为 true 必定登记配方：** teacher 不检查 `UnlockRecipe` 的结果；`CanLearn` 不排除 nounlock，但 `UnlockRecipe` 会跳过 nounlock。因此自定义载体仍需选择允许解锁的配方，不能只凭 Teach 返回值认定已学。[teacher.lua:18-26](../../../raw/current/scripts/components/teacher.lua#L18-L26)、[builder.lua:463-471](../../../raw/current/scripts/components/builder.lua#L463-L471)、[:954–969](../../../raw/current/scripts/components/builder.lua#L954-L969)
- **回调负责撤销解锁或保留载体：** 回调发生在解锁后，无否决返回；正常返回后必删载体。无工具消耗的学习需求应选 builder 的学习接口，不用此一次性路径。[teacher.lua:21-26](../../../raw/current/scripts/components/teacher.lua#L21-L26)、[[reference/components/builder#关键接口]]

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 处理范围：锚点重定位 9 处（componentactions／actions／SGwilson／SGwilson_client／mainfunctions 行号迁移，区间文本逐段比对一致；TEACH 收集与执行语义未变）；结论修订：无，仅锚点迁移。
- 依据：枢纽 diff 审阅（actions.lua／componentactions.lua hunk 地图）＋变更清单命中 actions.lua、componentactions.lua、stategraphs/SGwilson.lua、stategraphs/SGwilson_client.lua、entityscript.lua、mainfunctions.lua。

- 源码审阅：2026-10-23，Steam buildid 25643519；覆盖完整 teacher、builder 学习门禁／副作用、蓝图与渔具图纸注册／配置／恢复、WX-78 定向调用、TEACH 分发和人物 SG，以及示例宿主 PostInit、实体名存档／恢复链与饥饿调用。未全量审校配方、WX-78 扫描系统或其他模组消费者。
- 结构检查：本页元数据、链接、大小与源码引用定向检查通过；完整 modmain 示例经现有 Full Moon／Lua 5.1 语法检查。
- 游戏运行：未运行；联机提示、实际读蓝图／奖励、重载和其他模组回调联动待实测。
