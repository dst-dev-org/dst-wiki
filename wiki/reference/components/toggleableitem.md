---
title: "toggleableitem"
created: "2026-10-20"
updated: "2026-10-28"
type: component
summary: "物品双态布尔翻转与回调；没有存读／结束清理，原方法无成功返回，需明确动作适配和实际效果。"
aliases: ["剃刀模组标记开关", "物品双态翻转", "toggleableitem"]
tags: [component, item, actions, callback, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/toggleableitem.lua", "actions.lua", "componentactions.lua", "prefabs/razor.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# toggleableitem

## 快速判断

- 职责与适用：只有 on=false、回调字段和一次 ToggleItem 翻转；适合自定义双态开关基础，不等同 [[reference/components/useableitem]] 的使用中／停止协议。[完整定义](../../../raw/current/scripts/components/toggleableitem.lua#L1-L28)
- 优先使用：SetOnToggleFn(fn) 配实际效果；ToggleItem 翻转后调用 fn(item,on)，不传执行者，也不提供直接 SetOn(bool)。[方法](../../../raw/current/scripts/components/toggleableitem.lua#L9-L25)
- 重要副作用：先改 on 再回调，忽略回调返回值，没有失败回滚；原 ToggleItem 没有返回值，而 USEITEM 分支直接返回它，不能按动作返回真值判断已翻转。[执行](../../../raw/current/scripts/components/toggleableitem.lua#L17-L26) [动作](../../../raw/current/scripts/actions.lua#L3306-L3311)
- 修改边界：CanInteract(doer) 恒为 true，没有持有／权限／冷却判断；候选只检查物品 GrandOwner，实际效果、同步和保存由模组另行设计。[检查](../../../raw/current/scripts/components/toggleableitem.lua#L13-L15) [候选](../../../raw/current/scripts/componentactions.lua#L3064-L3069)

## 关键接口

| 接口／字段 | 条件、作用与证据 |
|---|---|
| `on` | 普通 Lua 布尔，默认 false；直接赋值不会自动调用效果回调，不是 netvar 或标签属性。[默认](../../../raw/current/scripts/components/toggleableitem.lua#L1-L7) |
| `SetOnToggleFn(fn)` | 只存 onusefn；fn(item,on) 在翻转后调用。原方法不接收 doer，动作多传的参数被忽略。[设置／调用](../../../raw/current/scripts/components/toggleableitem.lua#L9-L25) |
| `CanInteract(doer)` | 当前只返回 true；若要拒绝须自定义完整条件，不把候选库存归属当所有入口的服务端授权。[实现](../../../raw/current/scripts/components/toggleableitem.lua#L13-L15) |
| `onstopusefn`／`stopuseevents` | 构造确实有字段，但本文件没有 setter、消费者或停止钩子；不要由字段名编造 StopUsingItem 接口。[全部实现](../../../raw/current/scripts/components/toggleableitem.lua#L1-L28) |

定向搜索本快照的 Prefab 未找到静态 toggleableitem 挂载实例，保留“原版使用场景未核验”边界，不断言动态构造绝无使用。本页示例是依据组件／动作实现新增模组行为，不冒充原版效果。

## 典型接入

### 为原剃刀增加供模组脚本读取的双态标记

目标模组 `modmain.lua`，无新资源；原 razor 已有库存／检查／shaver。只新增私有 my_mod_marked 标签，其他模组逻辑可读取它决定自身功能；这不是剃毛能力开关，不改变原剃毛执行。[原宿主](../../../raw/current/scripts/prefabs/razor.lua#L23-L39)

```lua
AddPrefabPostInit("razor", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    if inst.components.toggleableitem ~= nil then return end
    local toggle = inst:AddComponent("toggleableitem")
    toggle:SetOnToggleFn(function(item, on)
        item:AddOrRemoveTag("my_mod_marked", on)
    end)
    local previous = toggle.ToggleItem
    toggle.ToggleItem = function(self)
        previous(self)
        return true
    end
end)
```

原库存候选将提供 USEITEM；本例仅对自己新增的组件包装返回 true，以免原动作得到 nil。脚本直接 ToggleItem 也走同一标记逻辑；不安装全局动作补丁，不覆盖其他模组已有组件。标签和 on 没有本例的存读协议，重载按关闭状态重建；停用重载不再新增能力。若扩展发光、装备属性或持久配置，须另补对应效果与撤销／保存证据，不能只给 on 赋值。

## 常见陷阱与限制

- **回调返回 false 就取消翻转**：返回被忽略，on 已变；需要拒绝先处理资格，或显式实现完整回滚，不借布尔返回推断。[顺序](../../../raw/current/scripts/components/toggleableitem.lua#L17-L25)
- **onstopusefn 会在关闭／卸装时运行**：当前文件未消费它，关闭也调用同一个 onusefn(item,false)，没有自动卸装清理。[完整实现](../../../raw/current/scripts/components/toggleableitem.lua#L1-L28)
- **与 useableitem 同挂就提供两个独立按钮**：USEITEM 先走 toggleableitem 分支，可能遮蔽后面的 useableitem；同一物品先确定交互协议。[优先级](../../../raw/current/scripts/actions.lua#L3306-L3324)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 中的漂移文件 actions.lua（R-01）与 componentactions.lua（R-04）逐 hunk 对照，锚点重定位 3 处（USEITEM 动作分支 3293→3296、INVENTORY 候选 3003→3052），结论修订 0 处——USEITEM.toggleableitem 分支、其与 useableitem 的优先级顺序及 INVENTORY 候选均未变，同段新增 USEITEM.spellcaster 处理器（R-04）只对 spellcaster 标签物提供 CASTSPELL，不影响本页断言；未覆盖：运行验证。

- 源码审阅：2026-10-20，Steam buildid 25643519；覆盖完整组件、库存候选和 USEITEM 分支优先级／返回、剃刀宿主及模组局部适配。未核验到静态原版挂载实例，未全面审校客户端动作反馈、其他模组同步或持久存档设计。
- 结构检查：本批格式、来源、链接／行号、大小与登记通过；完整标记开关例通过已有 Full Moon Lua 5.1 syntax-only。
- 游戏运行：未运行；动作提示／返回、标记两态、重载初值、原剃毛候选与其他模组同挂待实测。
