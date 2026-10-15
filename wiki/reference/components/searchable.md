---
title: "searchable"
created: "2026-10-23"
updated: "2026-10-28"
type: component
summary: "接入无自动生长周期的搜寻交互，控制可搜状态、动作速度及回调结果。"
aliases: ["搜寻交互", "翻找巢穴", "加快搜寻", "searchable"]
tags: [component, callback, actions, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/searchable.lua", "componentactions.lua", "actions.lua", "stategraphs/SGwilson.lua", "stategraphs/SGwilson_client.lua", "prefabs/otterden.lua", "entityscript.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# searchable

## 快速判断

- 职责与适用：在目标上提供简化搜寻与自定义回调，不自带产品、次数或再生周期；需要采收产物与再生时用 [[reference/components/pickable]]。[组件定位与默认值](../../../raw/current/scripts/components/searchable.lua#L14-L28)
- 优先使用：设置 `onsearchfn`、`canbesearched` 与 `caninteractwith`；`quicksearch`／`jostlesearch` 选动作形式。默认 `caninteractwith=true`，但 `canbesearched=nil`，挂载后不会自动可搜。[初始化](../../../raw/current/scripts/components/searchable.lua#L19-L36) [Search 门禁](../../../raw/current/scripts/components/searchable.lua#L44-L47)
- 重要副作用：成功时先执行回调，再向目标发 `searched`（data 直接是 searcher），最后按 `remove_when_searched` 删除目标。[Search 顺序](../../../raw/current/scripts/components/searchable.lua#L49-L62)
- 修改边界：原版 `otterden` 服务端挂载；交互复用 `ACTIONS.PICK`，客户端据标签预测、服务端据字段选状态，不能只改一端状态。[宿主](../../../raw/current/scripts/prefabs/otterden.lua#L337-L340) [挂载](../../../raw/current/scripts/prefabs/otterden.lua#L385-L386) [双端状态](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1127-L1130) [客户端标签](../../../raw/current/scripts/stategraphs/SGwilson_client.lua#L438-L441)

## 关键接口

| 接口／字段 | 参数或返回 | 条件、作用与证据 |
|---|---|---|
| `canbesearched`／`caninteractwith` | 默认 nil／true | 两者共同维护 `searchable` 标签，Search 也检查两者；直接赋字段会触发标签更新。[标签回调](../../../raw/current/scripts/components/searchable.lua#L1-L3) [字段注册](../../../raw/current/scripts/components/searchable.lua#L19-L36) |
| `onsearchfn` | `(inst, searcher) -> result, reason` | 无回调时默认 `true, nil`；有回调时原样采用其返回。回调只做副作用却不 return 时，结果为 nil，不发成功事件。[Search](../../../raw/current/scripts/components/searchable.lua#L44-L62) |
| `Search(searcher)` | 门禁失败 false；其余 result, reason | 不负责生成／交付物品，也不检查 searcher 是否存在或有物品栏；回调自己满足这些前提。[searchable.lua:44-62](../../../raw/current/scripts/components/searchable.lua#L44-L62) |
| `quicksearch`／`jostlesearch` | 默认 false | 分别维护同名标签；玩家 SG 优先 jostle，其次 quick，均无则 long。它们不是次数或冷却。[字段到标签](../../../raw/current/scripts/components/searchable.lua#L5-L11) [服务端选择](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1127-L1130) |
| `remove_when_searched` | 默认 false | 真值成功后才删目标，且在 `searched` 事件之后。[searchable.lua:54-62](../../../raw/current/scripts/components/searchable.lua#L54-L62) |
| `OnSave()`／`OnLoad(data)` | 只保存真值 caninteractwith | false 时不保存；加载只恢复真值，其他字段不保存，需宿主初始化或根据内容重算。[存读](../../../raw/current/scripts/components/searchable.lua#L66-L75) |

## 典型接入

### 加快海獭巢搜寻，保留原版物品与激怒行为

目标模组 `modmain.lua`，不新增资源。只改变玩家搜寻状态，保留巢穴库存决定的可搜资格及原回调；`otterden_dead` 是另一工厂，不在此修改内。[注册](../../../raw/current/scripts/prefabs/otterden.lua#L471-L472) [PostInit 注册接口](../../../raw/current/scripts/modutil.lua#L593-L599)

```lua
AddPrefabPostInit("otterden", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local searchable = inst.components.searchable
    if searchable ~= nil then
        searchable.jostlesearch = false
        searchable.quicksearch = true
    end
end)
```

- 原版库存收到物品时更新 `canbesearched`；搜索回调先激怒／生成海獭，再取一个物品交付，最后根据剩余库存更新外观与可搜资格。空库存返回 `false, "NOTHING_INSIDE"`，也不能假设失败无副作用。[入库更新](../../../raw/current/scripts/prefabs/otterden.lua#L213-L236) [完整搜索回调](../../../raw/current/scripts/prefabs/otterden.lua#L146-L179)
- `quicksearch` 的属性回调维护客户端所用标签，双端 SG 均选短动作；清掉 jostle 是为了避免它优先盖过 quick。此示例不覆盖 `onsearchfn`，不强行设置库存资格。[标签更新](../../../raw/current/scripts/components/searchable.lua#L5-L11) [双端选择](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1127-L1130) [客户端选择](../../../raw/current/scripts/stategraphs/SGwilson_client.lua#L438-L441)
- 移除本段后，新建／读档实例恢复工厂默认；代码不撤回已经完成的搜寻、掉落或激怒效果。物品交付机制见 [[reference/components/inventory]]、[[reference/components/stackable]]。

## 常见陷阱与限制

- **复用 PICK 的优先级：** 同时有 `pickable` 时，PICK 执行分支先走它；不要靠同时挂两组件获得两种独立菜单动作。`pickable_search_str` 仅改变动作文字。[PICK 字符串与执行](../../../raw/current/scripts/actions.lua#L1923-L1942)
- **跳过动作层直接调用：** 场景动作采集排除 `fire`／`intense`，但 Search 只查两个字段；直接调用需按需求补门禁。[动作采集](../../../raw/current/scripts/componentactions.lua#L776-L780) [Search](../../../raw/current/scripts/components/searchable.lua#L44-L62)
- **期待禁用跨存档保留：** `caninteractwith=false` 不写入存档，构造又默认 true；不能靠组件存读永久禁用。宿主需另存状态并恢复，或从库存等事实重算可搜资格。[默认值](../../../raw/current/scripts/components/searchable.lua#L22-L28) [存读](../../../raw/current/scripts/components/searchable.lua#L66-L75)
- **动态移组件后遗留标签：** 清理函数名是 `OnRemoveEntity`，实体销毁会调用；`RemoveComponent` 调用的是 `OnRemoveFromEntity`，本组件未实现该钩子。动态卸载时需自行清理三个交互标签，不能把实体删除清理等同组件卸载清理。[组件清理](../../../raw/current/scripts/components/searchable.lua#L38-L42) [移组件](../../../raw/current/scripts/entityscript.lua#L670-L683) [实体删除通知](../../../raw/current/scripts/entityscript.lua#L1748-L1751)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：actions.lua（R-01）、componentactions.lua（R-04）、stategraphs/SGwilson.lua（SB-17）、SGwilson_client.lua、entityscript.lua（R-08）对照 build-26148036 更新报告逐条核验，锚点重定位 7 处、结论修订 0 处：SCENE.searchable 采集（componentactions.lua:776-780，fire／intense 排除逐行未变，hunk 地图域 8 的 HasAnyTag 等价改写只发生在 pickable 处理器）、双端 SG 状态选择、PICK 执行与 EntityScript 清理分工均仅行号位移；未覆盖：运行验证。

- 源码审阅：2026-10-23，`steam-buildid:25643519`；覆盖完整组件、PICK 动作与双端 SG、海獭巢注册／服务端挂载／库存协作及 EntityScript 清理分工。示例只改动作形式，不改变原回调与库存规则。
- 游戏运行：未运行；多人实际动画、其他模组抢写字段和禁用状态自定义持久化未实测。
- 结构检查：页面格式／链接与源码锚点严格检查通过；完整 `modmain.lua` 示例通过现有解析器 Lua 5.1 语法检查，临时文件已清理。
