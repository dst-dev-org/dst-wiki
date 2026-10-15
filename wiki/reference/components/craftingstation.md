---
title: "craftingstation"
created: "2026-10-17"
updated: "2026-10-28"
type: component
summary: "工作站保存条目与配方对应关系、制作限额，经 prototyper 与 builder 提供临时配方。"
aliases: ["craftingstation","工作站配方","雕塑台图纸","制作限额"]
tags: [component, recipe, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/craftingstation.lua","components/prototyper.lua","components/builder.lua","prefabs/sculptingtable.lua","prefabs/sketch.lua","recipes.lua","entityscript.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# craftingstation

## 快速判断

- 职责与适用：站点端维护 itemname→recipename 的成对数组及限额；不是物品容器、配方注册器或人物永久学习表。[craftingstation.lua:1](../../../raw/current/scripts/components/craftingstation.lua#L1
- 优先使用：已注册配方用 LearnItem 配对；与 [[reference/components/prototyper]] 共同使用，人物 builder 只把有效且 nounlock 的条目纳入站点菜单。[builder.lua:276](../../../raw/current/scripts/components/builder.lua#L276
- 重要副作用：按 itemname 去重，不按 recipe 去重；限额用尽 ForgetRecipe 删除一对，并让传入 doers 重算科技。[craftingstation.lua:9](../../../raw/current/scripts/components/craftingstation.lua#L9)、[craftingstation.lua:40](../../../raw/current/scripts/components/craftingstation.lua#L40
- 修改边界：服务端维护，客户端配方来自人物 builder 同步，不给本组件自行假定 replica；GetItems 是知识条目，不是持有的实体。[builder.lua:350](../../../raw/current/scripts/components/builder.lua#L350)、[sculptingtable.lua:55](../../../raw/current/scripts/prefabs/sculptingtable.lua#L55

## 关键接口

| 接口 | 参数／返回 | 条件、副作用与证据 |
|---|---|---|
| `LearnItem(itemname,recipetouse)` | 两个字符串，无显式返回 | 不验配方存在、nounlock 或物品工厂；已有 item 不更新旧映射，必要时先 ForgetItem。[craftingstation.lua:9](../../../raw/current/scripts/components/craftingstation.lua#L9 |
| `KnowsItem(name)`／`KnowsRecipe(name)` | bool | 分别查两数组；不等于当前人物有材料／资格。[craftingstation.lua:16](../../../raw/current/scripts/components/craftingstation.lua#L16 |
| `GetItems()`／`GetRecipes()` | 原数组引用 | 不要单独修改其一，打乱对应关系；不同 item 可指同一 recipe，遗忘只移首个匹配。[craftingstation.lua:24](../../../raw/current/scripts/components/craftingstation.lua#L24)、[craftingstation.lua:69](../../../raw/current/scripts/components/craftingstation.lua#L69 |
| `SetRecipeCraftingLimit(name,amount)`／`GetRecipeCraftingLimit(name)` | 数值或 nil | 仅赋值／读取，不自动 Learn／Forget／刷新；0 不是可靠禁用方式，移除用 ForgetRecipe。[craftingstation.lua:32](../../../raw/current/scripts/components/craftingstation.lua#L32 |
| `RecipeCrafted(doers,name)` | doers 为实体键集合 | truthy 限额减一，耗尽删除并通知 builder；由 prototyper:Activate 分发，不应再监听 builditem 重扣一次。[craftingstation.lua:40](../../../raw/current/scripts/components/craftingstation.lua#L40)、[prototyper.lua:80](../../../raw/current/scripts/components/prototyper.lua#L80 |
| `ForgetItem/ForgetRecipe(name)`／`ForgetAllItems()` | 无显式返回 | 清对应限额；手动增删不主动刷新所有使用者。[craftingstation.lua:57](../../../raw/current/scripts/components/craftingstation.lua#L57)、[craftingstation.lua:80](../../../raw/current/scripts/components/craftingstation.lua#L80 |
| `nosave`／`OnSave/OnLoad` | 默认未设；true 禁用保存／恢复 | 保存两数组及限额；load 用已有数据替换，长度不一致清空全部。不会逐项验证内容／配方。[craftingstation.lua:86](../../../raw/current/scripts/components/craftingstation.lua#L86 |

### 原版协作

雕塑台接收 sketch 时 LearnItem，拒绝重复 item；锤毁／分解再按 GetItems 生成相应图纸。原版 rook 对应 item 为 chesspiece_rook_sketch、配方 chesspiece_rook_builder，后者已注册 nounlock／TECH.LOST；仍需雕塑材料和角色制作路径。[sculptingtable.lua:22](../../../raw/current/scripts/prefabs/sculptingtable.lua#L22)、[sculptingtable.lua:137](../../../raw/current/scripts/prefabs/sculptingtable.lua#L137)、[sketch.lua:9](../../../raw/current/scripts/prefabs/sketch.lua#L9)、[recipes.lua:883](../../../raw/current/scripts/recipes.lua#L883

## 典型接入

### 雕塑台默认具备战车图纸且读档后仍补齐

目标模组 `modmain.lua`；直接复用已注册图纸与雕塑配方、资源、材料消费／成品逻辑。组件 OnLoad 会替换两数组，故初始化与原实体 OnLoad 之后均执行；保留原回调。宿主先加载组件，再调用实体 OnLoad。[entityscript.lua:1979](../../../raw/current/scripts/entityscript.lua#L1979)、[sculptingtable.lua:242](../../../raw/current/scripts/prefabs/sculptingtable.lua#L242

```lua
local function AddRookSketch(inst)
    local station = inst.components.craftingstation
    if station ~= nil and not inst:HasTag("burnt") then
        station:LearnItem("chesspiece_rook_sketch",
            "chesspiece_rook_builder")
    end
end

AddPrefabPostInit("sculptingtable", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local original = inst.OnLoad
    inst.OnLoad = function(entity, data, ...)
        if original ~= nil then original(entity, data, ...) end
        AddRookSketch(entity)
    end
    AddRookSketch(inst)
end)
```

无额外资源／任务。新建及载入的未烧毁台补图纸，不免费提供雕塑材料；条目按原组件保存，停用后不会自动删掉已保存的图纸。原版锤毁可掉落该图纸，这是默认授予带来的资源影响，不是纯菜单改名。[craftingstation.lua:86](../../../raw/current/scripts/components/craftingstation.lua#L86)、[sculptingtable.lua:55](../../../raw/current/scripts/prefabs/sculptingtable.lua#L55

## 常见陷阱与限制

- **LearnItem 注册新配方或解锁全部已有配方**：builder 仍需 GetValidRecipe 且 nounlock；注册 API 及配方条件另核验。[builder.lua:279](../../../raw/current/scripts/components/builder.lua#L279
- **items 是实体或任意展示名**：原版可能将它传给 SpawnLootPrefab；使用正确可生成的图纸名，否则拆台出错。[sculptingtable.lua:55](../../../raw/current/scripts/prefabs/sculptingtable.lua#L55
- **有 craftingstation 就会显示菜单**：builder 从选中的 prototyper 读取，多个站点不全合并；recipe.manufactured 还有执行时站点检查。[builder.lua:265](../../../raw/current/scripts/components/builder.lua#L265)、[builder.lua:670](../../../raw/current/scripts/components/builder.lua#L670
- **load 自动合并初始化条目**：有效保存数组会整体替换，缺省授予应在恢复后应用；nosave 不替你迁移旧数据。[craftingstation.lua:94](../../../raw/current/scripts/components/craftingstation.lua#L94

## 验证记录

- 版本增量复核 26148036（2026-10-25）：锚点重定位 1 处（entityscript SetPersistData +3）；无结论修订，仅锚点迁移——craftingstation.lua／builder.lua 未变，sculptingtable.lua（本版删 2 行加 2 行均在 fn 尾部 358-361）、sketch.lua（+5 在 SKETCHES 表尾）、recipes.lua（rook 配方行 884 未变）所引锚点行号全部不变。依据：全局枢纽 diff 审阅＋变更清单命中 recipes.lua、sculptingtable.lua、sketch.lua、entityscript.lua。
- 源码审阅：2026-10-17，Steam buildid 25643519；覆盖完整组件、prototyper 限额分发、builder 站点筛选／同步／制造站分支、雕塑台接收／掉落／加载、rook 注册映射与宿主加载顺序。未全量审校雕塑成品、商店、全部限额网络编码或其他图纸系统。
- 结构检查：格式、来源、链接／源码锚点及唯一登记；完整示例经 Lua 5.1 syntax-only 检查。
- 游戏运行：未运行；新建／读档、烧毁、拆台掉图纸、多人菜单及其他 OnLoad 包装待测。

相关：[[reference/components/prototyper]]；[[reference/components/builder]]；[[reference/components/trader]]。
