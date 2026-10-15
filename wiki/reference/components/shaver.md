---
title: shaver：剃须工具动作标记
created: 2026-10-23
updated: "2026-10-28"
type: component
summary: 为物品接入原版剃须动作；剃取条件和产物由目标的 beard 或 shaveable 决定。
aliases: [shaver, 剃刀, 剃须工具, 给物品增加剃毛功能]
tags: [component, item, actions, server-client]
dst_build: "steam-buildid:26230496"
sources: [components/shaver.lua, componentactions.lua, actions.lua, stategraphs/SGwilson.lua, stategraphs/SGwilson_client.lua, prefabs/razor.lua, prefabs/flint.lua, entityscript.lua, modutil.lua]
reliability: verified
validation: source-reviewed
status: active
---

# shaver：剃须工具动作标记

## 快速判断

- 职责与适用：挂在工具上，表示它可参与原版剃须动作；组件只保存 `inst`，没有 `Shave`、产物配置或耐久逻辑。[组件全实现](../../../raw/current/scripts/components/shaver.lua#L1-L7)
- 优先使用：给工具添加 `shaver`；目标条件／产物分别看 [[reference/components/beard]]、[[reference/components/shaveable]]，执行动作优先选 `beard`，其次 `shaveable`。[SHAVE 分派](../../../raw/current/scripts/actions.lua#L2833-L2844)
- 重要副作用：除剃须外，工具动作收集还能对满足条件的包裹提供 `PEEKBUNDLE`；并非只增加一个菜单项。[目标动作收集](../../../raw/current/scripts/componentactions.lua#L1716-L1727)
- 修改边界：原版 razor 在服务端挂载；实体添加组件时注册组件动作，不要求另造 `shaver` 标签或 `Shave()` 接口。目标可见条件和真正执行检查分开。[razor 工厂](../../../raw/current/scripts/prefabs/razor.lua#L23-L39) [动作注册](../../../raw/current/scripts/entityscript.lua#L632-L667)

## 关键接口

| 入口 | 条件与执行边界 |
|---|---|
| `inst:AddComponent("shaver")` | 工具侧唯一必要接入；组件本身无额外公开方法、保存／加载或移除回调。[实现](../../../raw/current/scripts/components/shaver.lua#L1-L7) |
| 对目标收集 `SHAVE` | 目标有 `bearded` 且使用者未骑乘；目标是 `spiderden` 时还需使用者有 `spiderwhisperer`。[门禁](../../../raw/current/scripts/componentactions.lua#L1716-L1727) |
| 物品栏自用 `SHAVE` | 使用者有 `bearded`；物品为鼠标活动物品且正在骑乘时不显示，以免误认为给坐骑剃毛。[自用门禁](../../../raw/current/scripts/componentactions.lua#L2883-L2893) |
| `ACTIONS.SHAVE.fn(act)` | 目标取 `act.target or act.doer`，要求工具仍有 `shaver`，返回目标组件的 `Shave(doer, item)` 结果；目标没有两类组件则不执行。[动作](../../../raw/current/scripts/actions.lua#L2833-L2844) |

玩家服务端 `shave` 状态先调目标 `beard:ShouldTryToShave` 或 `shaveable:CanShave`，不通过会发 `actionfailed` 并清动作；通过后在一秒超时处执行缓冲动作。客户端预测使用 `dolongaction`。因此仅添加 `bearded` 标签不能保证剃取成功。[预检查](../../../raw/current/scripts/stategraphs/SGwilson.lua#L9119-L9154) [执行时点](../../../raw/current/scripts/stategraphs/SGwilson.lua#L9164-L9168) [客户端映射](../../../raw/current/scripts/stategraphs/SGwilson_client.lua#L477)

## 典型接入

### 让燧石兼作可重复使用的剃刀

目标模组 `modmain.lua`，沿用原版 `flint` 的网络、物品栏、堆叠和美术资源，无新增资源。修改覆盖全部燧石，不是创建新物品；适合作为给现有可拾取物品追加工具动作的局部例子。[flint 工厂](../../../raw/current/scripts/prefabs/flint.lua#L36-L66) [PostInit 入口](../../../raw/current/scripts/modutil.lua#L593-L595)

```lua
AddPrefabPostInit("flint", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    if inst.components.shaver == nil then
        inst:AddComponent("shaver")
    end
end)
```

剃取条件、产物及自剃效果仍由目标原版组件决定；本例没有添加消耗或改写目标。原版 `razor` 同样仅挂载 `shaver`，不在工具里实现剃须。停用后新初始化实体恢复原版工厂；目标已经发生的剃取结果不会回滚。[原版工具注册](../../../raw/current/scripts/prefabs/razor.lua#L29-L39) [动作委托](../../../raw/current/scripts/actions.lua#L2833-L2844)

## 常见陷阱与限制

- **调用 `item.components.shaver:Shave(...)`**：不存在此方法；执行路径在动作和目标组件，见[关键接口](#关键接口)。工具端也没有 `SetOnShaveFn`。
- **菜单出现就认定可剃**：动作收集看标签，服务端 SG 和目标组件还会复查。不要通过给目标补一个标签来绕过其状态／角色限制。[服务端检查](../../../raw/current/scripts/stategraphs/SGwilson.lua#L9129-L9146)
- **把组件当耗材系统**：组件不扣堆叠、不消耗耐久，也不规定掉落数量。需要耗材时另核验目标回调与消费时机，不能凭工具标记推断。[组件](../../../raw/current/scripts/components/shaver.lua#L1-L7)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：sources 中漂移文件 actions.lua、componentactions.lua、SGwilson.lua、SGwilson_client.lua、entityscript.lua（更新报告 R-01/R-04/SB-17/R-08）——SHAVE 动作、目标／自用候选门禁、shave 状态与客户端映射均未变；锚点重定位 8 处；未覆盖：运行验证。

- 源码审阅：2026-10-23，Steam buildid 25643519；覆盖完整组件、razor／flint 注册工厂与端边界、组件动作注册、目标／自用门禁、SHAVE 与玩家服务端／客户端 SG；目标剃取细节复用同版本 beard／shaveable 页。
- 结构检查：本页格式、链接、源码锚点与示例 Lua 5.1 语法检查通过。
- 游戏运行：未运行；燧石自剃／目标剃取、骑乘、堆叠、包裹菜单及多人预测效果待实测；不声称适用于所有自定义角色 SG。
