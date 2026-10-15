---
title: "builder"
created: "2026-10-12"
updated: "2026-10-28"
type: component
summary: "管理制作方的配方学习、材料消耗与建筑预制作，区分注册配方和完成制作。"
aliases: ["学习配方", "解锁配方", "制作材料折扣", "预制作建筑", "建造回调", "builder"]
tags: [component, recipe, callback, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/builder.lua", "constants.lua", "recipes.lua", "prefabs/player_common.lua", "modutil.lua", "entityscript.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# builder

## 快速判断

- 职责与适用：挂在制作方，管理已学配方、材料与预制作；人物已有该组件，不给产物加 builder。[player_common.lua:2775](../../../raw/current/scripts/prefabs/player_common.lua#L2775
- 优先使用：学会已注册配方用 `UnlockRecipe(name, true)`；新增配方从 modmain 的 `AddRecipe2` 注册，两者职责不同。[builder.lua:463](../../../raw/current/scripts/components/builder.lua#L463 [modutil.lua:732](../../../raw/current/scripts/modutil.lua#L732
- 重要副作用：学习默认加精神；建筑 `BufferBuild` 会先扣材料；`MakeRecipe` 返回 true 只表示已排入动作。[builder.lua:467](../../../raw/current/scripts/components/builder.lua#L467 [builder.lua:1180](../../../raw/current/scripts/components/builder.lua#L1180 [builder.lua:629](../../../raw/current/scripts/components/builder.lua#L629
- 修改边界：服务端修改已有 builder，由属性／方法维护 replica；学会配方仍不绕过角色、技能、材料与放置条件。具体事件见下文。[builder.lua:3](../../../raw/current/scripts/components/builder.lua#L3 [builder.lua:869](../../../raw/current/scripts/components/builder.lua#L869

## 关键接口

| 接口／字段 | 参数或返回 | 条件、作用与证据 |
|---|---|---|
| `UnlockRecipe(recname, nosanity?)` | 已注册配方名；无显式返回 | 仅有效且非 nounlock 配方；nosanity=true 取消学习精神奖励，登记并发制作方 unlockrecipe。重复调用仍可能奖励／发事件。[builder.lua:463](../../../raw/current/scripts/components/builder.lua#L463 |
| `AddRecipe(name)`／`RemoveRecipe(name)` | 学习列表增删 | 同步 replica；AddRecipe 列表去重，但不负责有效性检查或 UnlockRecipe 的奖励／事件。[builder.lua:451](../../../raw/current/scripts/components/builder.lua#L451 |
| `KnowsRecipe(recipe_or_name, ignore_tempbonus?)` | boolean | 综合已学／工作站／科技及角色限制；true 不等于已永久学习。[builder.lua:841](../../../raw/current/scripts/components/builder.lua#L841 |
| `HasIngredients(recipe_or_name)` | boolean | 查材料、角色代价、科技材料及限额；旧 CanBuild 只是此检查的别名，不是全部制作资格。[builder.lua:918](../../../raw/current/scripts/components/builder.lua#L918 [builder.lua:950](../../../raw/current/scripts/components/builder.lua#L950 |
| `ingredientmod` | 0／0.25／0.5／0.75／1 | 赋其他值 assert；普通正数量材料折扣后向偏置方向取整且至少 1，不覆盖全部角色代价。[constants.lua:2169](../../../raw/current/scripts/constants.lua#L2169 [builder.lua:500](../../../raw/current/scripts/components/builder.lua#L500 |
| `MakeRecipeFromMenu(recipe, skin?)` | recipe 对象 | 非 placer 的菜单路径，处理学习／科技后排入动作；建筑另走预制作与地点路径。[builder.lua:1041](../../../raw/current/scripts/components/builder.lua#L1041 |
| `BufferBuild(name)`／`IsBuildBuffered(name)` | 预制作／状态 | 仅 placer 配方，检查后消耗材料并存缓冲；SetBuildBuffered 仅记账，不能替代扣料流程。[builder.lua:1126](../../../raw/current/scripts/components/builder.lua#L1126 [builder.lua:1131](../../../raw/current/scripts/components/builder.lua#L1131 |
| `MakeRecipeAtPoint(recipe, pt, rot, skin?)` | recipe、坐标、旋转 | 要有缓冲与有效放置地点，再调用 MakeRecipe；不要用 DoBuild 绕过上层放置检查。[builder.lua:1113](../../../raw/current/scripts/components/builder.lua#L1113 [builder.lua:657](../../../raw/current/scripts/components/builder.lua#L657 |

### 制作事件与产物初始化

`makerecipe` 是尝试通知，发出时材料还未核验。`builditem` 在扣料后、发放／装备产物前通知制作方；监听到它不能假定产物已在物品栏。[builder.lua:631](../../../raw/current/scripts/components/builder.lua#L631 [builder.lua:741](../../../raw/current/scripts/components/builder.lua#L741

建筑路径向制作方发 `buildstructure`，向产物发 `onbuilt {builder, pos}`。两条普通产物路径随后均调用产物 `OnBuilt(builder)`：宿主分发组件 OnBuilt，再调用实体 `OnBuiltFn`（self, builder）。可拾取产物不能依赖建筑专用 onbuilt 事件；保留原回调并选择对应对象。[builder.lua:791](../../../raw/current/scripts/components/builder.lua#L791 [builder.lua:821](../../../raw/current/scripts/components/builder.lua#L821 [entityscript.lua:1718](../../../raw/current/scripts/entityscript.lua#L1718

## 典型接入

### 让人物直接学会长矛配方而不增加精神

目标模组 `modmain.lua`；影响所有带 builder 的玩家实例，沿用原版 spear 配方、材料与资源，不创建新配方。原版已注册科技一级长矛；玩家入口是按 player 标签筛选的 Prefab PostInit。[recipes.lua:478](../../../raw/current/scripts/recipes.lua#L478 [modutil.lua:586](../../../raw/current/scripts/modutil.lua#L586

```lua
AddPlayerPostInit(function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local builder = inst.components.builder
    if builder ~= nil then
        builder:UnlockRecipe("spear", true)
    end
end)
```

替换配方名时确认有效、允许学习，并核对角色／技能门禁。此例取消学习奖励，不免材料、不直接发物品；学习记录进入人物存档，停用模组不会自动撤销已学配方。组件 OnLoad 增补已保存的学习记录，不清空此时列表。[builder.lua:80](../../../raw/current/scripts/components/builder.lua#L80 [builder.lua:98](../../../raw/current/scripts/components/builder.lua#L98

## 常见陷阱与限制

- **学习接口用来注册配方**：未注册或 nounlock 时 UnlockRecipe 无效果；先注册，再设计学习规则。[builder.lua:463](../../../raw/current/scripts/components/builder.lua#L463
- **折扣设 0 就免材料**：正数量普通材料仍至少消耗 1；用 freebuildmode 需要另评估行为范围。`GiveAllRecipes()` 实际切换此模式，再调用会关闭，并非永久学会所有配方。[builder.lua:136](../../../raw/current/scripts/components/builder.lua#L136 [builder.lua:500](../../../raw/current/scripts/components/builder.lua#L500
- **DoBuild 是通用安全制作入口**：其检查不等于菜单／地点路径全部前提，失败前也可能已清缓冲；先选菜单或建筑流程，不能据返回值假设无副作用。[builder.lua:690](../../../raw/current/scripts/components/builder.lua#L690 [builder.lua:1113](../../../raw/current/scripts/components/builder.lua#L1113
- **直接改学习数组或缓冲表**：可能遗漏 replica 同步／材料消费；用对应方法。存档保存已学配方、缓冲和临时科技，未保存 ingredientmod／freebuildmode；持续属性修改需在初始化时重新应用。[builder.lua:451](../../../raw/current/scripts/components/builder.lua#L451 [builder.lua:80](../../../raw/current/scripts/components/builder.lua#L80

## 验证记录

- 版本增量复核 26148036（2026-10-25）：锚点重定位 3 处（player_common builder 构造 -14、constants INGREDIENT_MOD_LOOKUP +4、entityscript OnBuilt +3）；无结论修订，仅锚点迁移——builder.lua 本版未变，recipes.lua spear 配方行号未变。依据：全局枢纽 diff 审阅＋变更清单命中 constants.lua（本版改动在前部与 SOCKETQUALITY 段）、recipes.lua、player_common.lua、entityscript.lua。
- 源码审阅：2026-10-12，Steam buildid 25643519；覆盖上述学习／材料／缓冲与排入动作、普通产物事件、玩家接入和存档。科技树完整计算、制造站特例、皮肤与所有配方消费者未全量审校。
- 结构检查：元数据、源码路径／行号、链接／章节、大小和登记通过；发布示例经 Full Moon 3.0.0／Lua 5.1 语法及原字节往返检查。
- 游戏运行：未运行；客户端菜单刷新、重选角色、重载与其他模组联动待实测。

相关：[[reference/components/inventory#关键接口]]；[[reference/components/workable#典型接入]]；[[reference/components/lootdropper#关键接口]]。
