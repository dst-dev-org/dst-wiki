---
title: "recipescanner"
created: "2026-10-23"
updated: "2026-10-28"
type: component
summary: "从扫描目标反查配方并交给操作者学习，区分配方重定向、目标事件与扫描器自身消耗。"
aliases: ["扫描学习配方", "薇诺娜配方扫描器", "扫描目标配方重定向", "SCANNABLE_RECIPENAME", "recipescanner"]
tags: [component, recipe, callback, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/recipescanner.lua", "components/builder.lua", "prefabs/winona_holoitems.lua", "actions.lua", "componentactions.lua", "stategraphs/SGwilson.lua", "stategraphs/SGwilson_client.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# recipescanner

## 快速判断

- 职责与适用：挂在扫描工具上，从目标实体解析配方，再让 `doer.components.builder` 学习。固定配方的一次性蓝图用 [[reference/components/teacher]]；单纯解锁用 [[reference/components/builder#关键接口]]。[recipescanner.lua:17-44](../../../raw/current/scripts/components/recipescanner.lua#L17-L44)
- 优先使用：`Scan(target, doer)` 保留完整门禁与顺序；成功扩展用 `SetOnScannedFn`，指定目标配方可由目标设置 `SCANNABLE_RECIPENAME`。[recipescanner.lua:13-42](../../../raw/current/scripts/components/recipescanner.lua#L13-L42)
- 重要副作用：先解锁，向目标发 `onrecipescanned`，再回调工具；组件自己不删除任一实体。原版全息目标监听事件自删，扫描器宿主在回调中自删，不能将其误认为组件通用消耗。[recipescanner.lua:39-44](../../../raw/current/scripts/components/recipescanner.lua#L39-L44)、[winona_holoitems.lua:217-244](../../../raw/current/scripts/prefabs/winona_holoitems.lua#L217-L244)
- 修改边界：原版 `winona_recipescanner` 在服务端挂组件、pristine 前加动作标签；它沿用全息物品的所属者、掉落计时和可见性，不是可直接复制的普通永久物品。[winona_holoitems.lua:121-136](../../../raw/current/scripts/prefabs/winona_holoitems.lua#L121-L136)、[:169–199](../../../raw/current/scripts/prefabs/winona_holoitems.lua#L169-L199)、[:247–263](../../../raw/current/scripts/prefabs/winona_holoitems.lua#L247-L263)

## 关键接口

| 接口／字段 | 签名或约定 | 条件、作用与证据 |
|---|---|---|
| `Scan(target, doer)` | `true`／`false, reason?` | 无 builder 或目标有 NOCLICK 返回 false；缺配方、不可解锁／分解或 CanLearn 不通过返回 CANTLEARN；已知配方返回 KNOWN。[recipescanner.lua:17-44](../../../raw/current/scripts/components/recipescanner.lua#L17-L44) |
| `target.SCANNABLE_RECIPENAME` | 目标实体的可选配方名 | 真值时直接 `GetValidRecipe`；无效不会回退到 prefab，也不会继续处理该配方的 source_recipename。[recipescanner.lua:22-29](../../../raw/current/scripts/components/recipescanner.lua#L22-L29) |
| `recipe.source_recipename` | 配方上的可选来源名 | 仅未显式指定时，先查 `AllRecipes[target.prefab]`，再对该字段做一次 GetValidRecipe；不是递归追溯。[recipescanner.lua:25-27](../../../raw/current/scripts/components/recipescanner.lua#L25-L27) |
| `SetOnScannedFn(fn)`／`onscanned` | `(scanner, target, doer, recipe_name)` | 成功且目标事件已发出后执行；不能用返回 false 否决已发生的解锁。[recipescanner.lua:13-14](../../../raw/current/scripts/components/recipescanner.lua#L13-L14)、[:39–44](../../../raw/current/scripts/components/recipescanner.lua#L39-L44) |
| `onrecipescanned` | 发给目标；`{scanner, doer, recipe}` | recipe 是名称字符串。监听者可移除目标，所以工具回调不能无条件假定目标仍有效。[recipescanner.lua:39-42](../../../raw/current/scripts/components/recipescanner.lua#L39-L42)、[winona_holoitems.lua:217-237](../../../raw/current/scripts/prefabs/winona_holoitems.lua#L217-L237) |
| `recipescanner` 标签／`OnRemoveFromEntity()` | 构造添加、移除组件时删除 | 原版提前放到 pristine 状态；这是动作能力标记，不是配方信息的网络变量。[recipescanner.lua:1-10](../../../raw/current/scripts/components/recipescanner.lua#L1-L10)、[winona_holoitems.lua:247-254](../../../raw/current/scripts/prefabs/winona_holoitems.lua#L247-L254) |

学习检查先 `KnowsRecipe(recipe_object, true)`，后检查 nounlock／`FunctionOrValue(recipe.no_deconstruction, target)`，再 `CanLearn(recipe.name)`；builder 前者接受对象，后者接受名称并核对角色／技能。成功走默认 `UnlockRecipe(name)`，仍有学习精神奖励。[recipescanner.lua:30-39](../../../raw/current/scripts/components/recipescanner.lua#L30-L39)、[builder.lua:841-848](../../../raw/current/scripts/components/builder.lua#L841-L848)、[:954–969](../../../raw/current/scripts/components/builder.lua#L954-L969)、[:463–471](../../../raw/current/scripts/components/builder.lua#L463-L471)

## 典型接入

### 让薇诺娜扫描器只扫描明确标注配方的目标

完整目标模组 `modmain.lua`；定向包装现有 `winona_recipescanner` 的服务端 Scan，新增“目标必须显式设置配方名”的限制，其余门禁、学习、目标事件和宿主回调原样委托。沿用原版资源与全息物品获取方式，不新增通用工具。[winona_holoitems.lua:252-263](../../../raw/current/scripts/prefabs/winona_holoitems.lua#L252-L263)、[modutil.lua:592-598](../../../raw/current/scripts/modutil.lua#L592-L598)

```lua
AddPrefabPostInit("winona_recipescanner", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local scanner = inst.components.recipescanner
    if scanner == nil then return end
    local old_scan = scanner.Scan
    scanner.Scan = function(self, target, doer)
        if not target.SCANNABLE_RECIPENAME then
            return false, "CANTLEARN"
        end
        return old_scan(self, target, doer)
    end
end)
```

原版 `winona_holotelepad`／`winona_holotelebrella` 分别显式指定传送配方；它们成功扫描后自删，工具随后发学习者 `learnrecipe` 并自删。未标注目标仅被本例拒绝，不消耗工具；已知配方等仍由原逻辑决定。禁用模组后新工具恢复原有目标范围，无新增存档字段。[winona_holoitems.lua:217-244](../../../raw/current/scripts/prefabs/winona_holoitems.lua#L217-L244)

**动作提示边界：** componentactions 自己解析配方并只筛 nounlock／no_deconstruction，没有读取本例 Scan 附加限制；因此部分目标仍显示扫描提示，执行时才失败。需要同步改变交互提示时应另设计动作筛选，不能把本例称为完整客户端过滤。[componentactions.lua:1664-1675](../../../raw/current/scripts/componentactions.lua#L1664-L1675)

原版复用 `TEACH`：标签将文本选为 SCAN，执行分发到 `Scan(target, doer)`。人物两端都走长动作，服务端超时执行缓冲动作；添加组件并不立即扫描。[actions.lua:3249-3256](../../../raw/current/scripts/actions.lua#L3249-L3256)、[:3269–3270](../../../raw/current/scripts/actions.lua#L3279-L3280)、[SGwilson.lua:1214](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1214)、[:8352–8360](../../../raw/current/scripts/stategraphs/SGwilson.lua#L8384-L8392)、[SGwilson_client.lua:522](../../../raw/current/scripts/stategraphs/SGwilson_client.lua#L522

## 常见陷阱与限制

- **扫描成功会通用销毁或拆解目标：** 组件仅发事件，没有删除／返还材料。全息目标与工具的删除来自各自宿主；迁到自定义工具时自行决定消费，并防止事件后访问已删除目标。[recipescanner.lua:39-44](../../../raw/current/scripts/components/recipescanner.lua#L39-L44)、[winona_holoitems.lua:217-244](../../../raw/current/scripts/prefabs/winona_holoitems.lua#L217-L244)
- **把 `no_deconstruction` 仅当拆解工具规则：** 扫描同样会检查，函数值以目标为参数；已知配方检查先发生，可能先返回 KNOWN。不能从提示或目标存在配方推断能扫描成功。[recipescanner.lua:30-37](../../../raw/current/scripts/components/recipescanner.lua#L30-L37)
- **同物品同时添加 teacher 和 recipescanner：** TEACH 执行优先 teacher，扫描器分支在其后；仅加扫描标签不能改变这个优先级。请选择单一教学路径或单独设计动作。[actions.lua:3263-3280](../../../raw/current/scripts/actions.lua#L3263-L3280)
- **自定义指定配方只在服务端赋值：** 客户端动作收集也直接读取 SCANNABLE_RECIPENAME。原版在 common_postinit、pristine 前赋值；动态变化须自行提供匹配两端的状态，不把普通 Lua 字段当自动同步。[componentactions.lua:1664-1675](../../../raw/current/scripts/componentactions.lua#L1664-L1675)、[winona_holoitems.lua:169-173](../../../raw/current/scripts/prefabs/winona_holoitems.lua#L169-L173)、[:228–233](../../../raw/current/scripts/prefabs/winona_holoitems.lua#L228-L233)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 变更文件（actions.lua、componentactions.lua、stategraphs/SGwilson.lua、stategraphs/SGwilson_client.lua，更新报告 R-01/R-04）逐条核验，TEACH／SCAN 文本与分发、recipescanner 采集解析均无语义变化，锚点重定位 7 处、无结论修订；未覆盖：运行验证。

- 源码审阅：2026-10-23，Steam buildid 25643519；覆盖完整 recipescanner、配方解析两分支与学习门禁／事件顺序、原版全息工厂和扫描器／目标注册及删除、TEACH／人物 SG 与示例 PostInit。未全量审校可扫描配方、所有目标或全息物品生成来源。
- 结构检查：本页元数据、链接、大小与源码引用定向检查通过；完整 modmain 示例经现有 Full Moon／Lua 5.1 语法检查。
- 游戏运行：未运行；目标提示与执行差异、实际解锁／双端显示、目标与工具消费及其他模组联动待实测。
