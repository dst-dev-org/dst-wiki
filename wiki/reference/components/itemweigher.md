---
title: "itemweigher"
created: "2026-10-23"
updated: "2026-10-28"
type: component
summary: "为便携秤接入称重动作与执行回调，区分类型匹配、重量播报和工具消耗。"
aliases: ["便携秤", "称重工具", "鱼称重", "itemweigher"]
tags: [component, item, actions, callback, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/itemweigher.lua", "prefabs/pocket_scale.lua", "componentactions.lua", "actions.lua", "stategraphs/SGwilson.lua", "stategraphs/SGwilson_client.lua", "constants.lua", "entityscript.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# itemweigher

## 快速判断

- 职责与适用：挂在称重工具上，维护类型标签并委派执行回调；不生成重量、不改捕获者，也不自行播报。被称物的数据用 [[reference/components/weighable]]。[完整组件](../../../raw/current/scripts/components/itemweigher.lua#L1-L34)
- 优先使用：设置 `type` 和 `SetOnDoWeighInFn(fn)`，正常交互沿用 `WEIGH_ITEM`。原版 `pocket_scale` 使用鱼类型，成功回调扣一次耐久。[原版配置](../../../raw/current/scripts/prefabs/pocket_scale.lua#L42-L49) [消费回调](../../../raw/current/scripts/prefabs/pocket_scale.lua#L6-L11)
- 重要副作用：`DoWeighIn` 不检查重量或类型，只保留回调的首个真值结果；false 被转换为 nil，失败原因不透传。[实现](../../../raw/current/scripts/components/itemweigher.lua#L26-L32)
- 修改边界：原版在 pristine 前加 `trophyscale_fish`，服务端才挂组件；类型标签供动作发现，玩家 SG 才读取重量并播报。组件移除钩子存在未限定名称赋值问题，见下文。[端边界](../../../raw/current/scripts/prefabs/pocket_scale.lua#L28-L49) [播报](../../../raw/current/scripts/stategraphs/SGwilson.lua#L9984-L9995)

## 关键接口

| 接口／字段 | 参数或返回 | 条件、作用与证据 |
|---|---|---|
| `type` | 默认 nil；原版取 `TROPHYSCALE_TYPES` 的值 | 属性变化删除旧 `trophyscale_<type>`、添加新标签。当前常量为 fish／oversizedveggies；自定义字符串不会自动加入动作枚举。[属性](../../../raw/current/scripts/components/itemweigher.lua#L1-L20) [常量](../../../raw/current/scripts/constants.lua#L2265-L2269) |
| `SetOnDoWeighInFn(fn)` | `fn(tool, target, doer)`；初始未设置 | 替换单一回调；组件没有内置耐久消费。[设置](../../../raw/current/scripts/components/itemweigher.lua#L26-L28) |
| `DoWeighIn(target, doer)` | 回调首个真值，或 nil | 不验证目标、持有者或 `weighable`；`and/or` 表达式不保留第二返回值。需要附加处理时保留原回调；需要改变消费策略时明确替换。[实现](../../../raw/current/scripts/components/itemweigher.lua#L30-L32) |

正常动作有两种物品方向：拿秤作用于自己持有的匹配物品，或拿可称物作用于自己持有的匹配秤；动作发现用同一类型标签配对与 `IsGrandOwner` 检查。`WEIGH_ITEM.fn` 先取 target 上的秤，否则取 invobject 上的秤，只复查秤的 fire／burnt，不重做全部类型和持有校验。[秤侧发现](../../../raw/current/scripts/componentactions.lua#L1450-L1457) [物品侧发现](../../../raw/current/scripts/componentactions.lua#L2008-L2015) [动作执行](../../../raw/current/scripts/actions.lua#L4679-L4694)

双端玩家都路由到 `use_pocket_scale`；服务端第 30 帧先读取目标重量，重量非 nil 且缓冲动作成功才播报。重量比例达到阈值时使用重物台词；工具回调返回 true 不等于组件自己测量或显示过重量。[路由](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1553-L1554) [客户端路由](../../../raw/current/scripts/stategraphs/SGwilson_client.lua#L823-L824) [读取与播报](../../../raw/current/scripts/stategraphs/SGwilson.lua#L9969-L9995)

## 典型接入

### 让原版便携秤称重不消耗耐久

完整目标模组 `modmain.lua`，复用 `pocket_scale` 工厂、资源和动作；有意替换原版只负责扣次数的回调，不调用它，保留成功返回供 SG 播报。[原回调](../../../raw/current/scripts/prefabs/pocket_scale.lua#L6-L11) [工厂／注册](../../../raw/current/scripts/prefabs/pocket_scale.lua#L42-L63) [PostInit](../../../raw/current/scripts/modutil.lua#L593-L599)

```lua
AddPrefabPostInit("pocket_scale", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local weigher = inst.components.itemweigher
    if weigher ~= nil then
        weigher:SetOnDoWeighInFn(function(tool, target, doer)
            return true
        end)
    end
end)
```

只取消称重回调中的消耗，不恢复已有耐久，不阻止燃烧或其他删除来源。原 `finiteuses` 与耗尽删除配置仍在；其他模组在同一回调增加的行为也会被本例覆盖，合用时需重新约定消费策略。移除代码并重载／新建工具后恢复原工厂回调，无新增存档、任务或监听。[原配置](../../../raw/current/scripts/prefabs/pocket_scale.lua#L42-L49) 耐久机制复用 [[reference/components/finiteuses]]。

## 常见陷阱与限制

- **把工具当作重量数据源：** 它不计算重量；正常玩家 SG 还要求被称物有可读取的重量及比例。新对象应完整初始化 weighable，不只加类型标签。[SG 消费](../../../raw/current/scripts/stategraphs/SGwilson.lua#L9969-L9995)
- **失败原因未传回：** 回调 `return false, reason` 会在组件表达式中变成 nil。不要为该组件设计依赖第二返回值的失败提示。[返回表达式](../../../raw/current/scripts/components/itemweigher.lua#L30-L32)
- **把奖杯秤与便携秤混用：** 建筑 `COMPARE_WEIGHABLE` 调用的是 `trophyscale:Compare`；不是本组件接口。两条动作使用相似类型标签，但执行／物品消费不同。[动作分流](../../../raw/current/scripts/actions.lua#L4666-L4694)
- **动态移除留下标签：** 当前 `OnRemoveFromEntity` 写的是未限定的 `type = nil`，不是 `self.type = nil`，不会触发组件属性清标签，并会改写所在 Lua 环境的该名称绑定。这是源码疑点；不能将它描述成正常清理，也不要直接照抄到模组。框架移组件确实调用该钩子；本页示例不移除此组件。[实际钩子](../../../raw/current/scripts/components/itemweigher.lua#L22-L24) [框架调用](../../../raw/current/scripts/entityscript.lua#L670-L683)
- **期待配置自动存档：** 组件无存读方法；原版工厂重设类型和回调。运行时改型的持久化及客户端标签一致性由宿主负责。[组件全文](../../../raw/current/scripts/components/itemweigher.lua#L1-L34)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 的变更文件（componentactions.lua、actions.lua、SGwilson.lua、SGwilson_client.lua、constants.lua、entityscript.lua）对照 build-26148036 更新报告条目 R-01／R-04 逐条核验，引用区域内容均未变化，锚点重定位 8 处，无结论修订；未覆盖：运行验证。

- 源码审阅：2026-10-23，Steam buildid 25643519；覆盖完整组件、便携秤注册与服务端配置、双向动作发现／执行、玩家 SG 读取播报、类型常量及移组件调用链；移除钩子的环境写入影响未在游戏中复现。
- 游戏运行：未运行；无限称重、多人交互、耗尽边界及其他模组回调兼容待实测。
- 结构检查：本页格式／链接、源码锚点严格检查及完整 Lua 5.1 示例语法检查通过；不代替游戏验证。
