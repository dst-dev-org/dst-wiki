---
title: "useabletargeteditem"
created: "2026-10-19"
updated: "2026-10-28"
type: component
summary: "物品对实体使用的回调与使用中标志，候选筛选、服务器授权、目标引用和结束清理由宿主分别实现。"
aliases: ["命名牌服务端校验目标", "对实体使用物品", "useabletargeteditem"]
tags: [component, item, actions, callback, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/useabletargeteditem.lua", "componentactions.lua", "actions.lua", "prefabs/kitcoon_nametag.lua", "prefabs/beef_bell.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# useabletargeteditem

## 快速判断

- 职责与适用：物品端管理 inuse_targeted 标签和开始／停止回调，用于对实体命名、牛铃绑定等；不是完整目标／权限验证器，也不在组件中保存所用目标。[构造](../../../raw/current/scripts/components/useabletargeteditem.lua#L39-L61) [开始／停止](../../../raw/current/scripts/components/useabletargeteditem.lua#L113-L137)
- 优先使用：SetOnUseFn／SetOnStopUseFn 配宿主回调，正常 USEITEMON 先 CanInteract 再 StartUsingItem；自己直接调用时仍需检查目标、使用状态和执行者。[动作](../../../raw/current/scripts/actions.lua#L3351-L3374)
- 重要副作用：开始先调 onusefn，再在成功且物品仍有效时置 inuse_targeted=true；停止先清标志再调 onstopusefn。没有回调时开始默认成功。[顺序](../../../raw/current/scripts/components/useabletargeteditem.lua#L113-L137)
- 修改边界：候选侧 ValidTarget 与目标 Prefab 标签是 OR，不是双重校验；服务端 StartUsingItem 不执行该筛选，也不自行检查范围／骑乘／目标类型。已有装备主动使用另见 [[reference/components/useableitem]]。[候选](../../../raw/current/scripts/componentactions.lua#L1899-L1912)

## 关键接口

| 接口／字段 | 条件、作用与证据 |
|---|---|
| `SetOnUseFn(fn)`／`SetOnStopUseFn(fn)` | fn(item,target,doer) 返回 success,reason；stopfn(item) 无目标／doer 参数，目标引用由宿主保留。不能从 stopfn 参数推导此前目标。[设置](../../../raw/current/scripts/components/useabletargeteditem.lua#L89-L95) [调用](../../../raw/current/scripts/components/useabletargeteditem.lua#L113-L137) |
| `CanInteract()` | 仅 `not inuse_targeted`；StartUsingItem 自身不调用它，重复直接启动可多次进入宿主回调。[方法](../../../raw/current/scripts/components/useabletargeteditem.lua#L109-L128) |
| `SetTargetPrefab(name)` | 联动 `<name>_targeter` 标签；是候选匹配的一条路径，不是服务器目标限制。[属性](../../../raw/current/scripts/components/useabletargeteditem.lua#L21-L29) [候选 OR](../../../raw/current/scripts/componentactions.lua#L1899-L1912) |
| `inst.UseableTargetedItem_ValidTarget(item,target,doer)` | 配在实体、客户端可见的位置；原命名牌在 SetPristine 前提供 kitcoon 标签筛选，服务器 onuse 回调另管真正执行。[原筛选](../../../raw/current/scripts/prefabs/kitcoon_nametag.lua#L82-L105) |
| `SetInventoryDisable(bool)` | 为使用中物品提供库存 STOPUSINGITEM 候选，不等于自动关闭 UI／撤销效果；SetCanSelfTarget 允许无显式 target 时选 doer，实际标签保留源码拼写 `useabletargateditem_canselftarget`。[设置](../../../raw/current/scripts/components/useabletargeteditem.lua#L97-L103) [库存候选](../../../raw/current/scripts/componentactions.lua#L3071-L3089) [回退目标](../../../raw/current/scripts/actions.lua#L3351-L3360) |
| `SetUseableMounted(enable)` | 联动 useabletargeteditem_mounted，控制上述候选骑乘门禁；不能由它推出所有 SG／宿主支持骑乘。[属性](../../../raw/current/scripts/components/useabletargeteditem.lua#L31-L37) [门禁](../../../raw/current/scripts/componentactions.lua#L1905-L1911) |
| `SetUsingItemDoesNotToggleUseability(value)` | 为真时开始成功也不自动置使用中；若回调内部先 StopUsingItem 再返回 true，默认仍会被外层重新置 true。按一次性／持续模式选择，不用回调顺序猜状态。[设置](../../../raw/current/scripts/components/useabletargeteditem.lua#L105-L107) [开始后置状态](../../../raw/current/scripts/components/useabletargeteditem.lua#L113-L128) |

组件没有存读，不自动保存绑定对象；原牛铃在 onuse／onstop 中维护实际绑定，存档恢复另走其宿主协议。移除组件仅清部分自身标签，不调用 StopUsingItem／宿主停止，也未清 canselftarget 对应标签，热移除不能当完整效果撤销。[移除](../../../raw/current/scripts/components/useabletargeteditem.lua#L63-L79) [牛铃配置](../../../raw/current/scripts/prefabs/beef_bell.lua#L350-L354) [绑定实现](../../../raw/current/scripts/prefabs/beef_bell.lua#L153-L204)

## 典型接入

### 为原淘气小猫命名牌补服务端目标与玩家检查

目标模组 `modmain.lua`，无新资源；保留原客户端筛选、命名界面、写入／完成与使用状态，仅包装服务器 onuse。适用于原 kitcoon_nametag，不把候选提示当授权。

```lua
AddPrefabPostInit("kitcoon_nametag", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local usable = inst.components.useabletargeteditem
    if usable == nil or usable.onusefn == nil then return end
    local previous = usable.onusefn
    usable:SetOnUseFn(function(item, target, doer)
        if target == nil or not target:IsValid()
            or not target:HasTag("kitcoon")
            or doer == nil or not doer:IsValid()
            or not doer:HasTag("player") then
            return false
        end
        return previous(item, target, doer)
    end)
end)
```

新增检查在原回调写 naming_target、开编辑器和令目标停止移动之前执行；不替代距离、忙碌、归属或完整 RPC 校验，直接 StartUsingItem 仍不自检 CanInteract。原结束路径清 naming_target、监听和目标 is_being_named，再 StopUsingItem；不能只改标志而遗留命名会话。[开始](../../../raw/current/scripts/prefabs/kitcoon_nametag.lua#L38-L60) [结束](../../../raw/current/scripts/prefabs/kitcoon_nametag.lua#L68-L80) [原组装](../../../raw/current/scripts/prefabs/kitcoon_nametag.lua#L110-L122)

包装每次构造重装，不存新状态；停用重载回原回调，不中途卸载组件或手工撤销正在进行的编辑器。原命名效果、字数限制及 writeable 完整流程不由本例重新定义。

## 常见陷阱与限制

- **ValidTarget 返回 false 就能阻止所有使用**：目标 Prefab 标签路径仍可在 OR 中成功，且服务器方法不调用候选函数；在真正 onuse 中复查必要前提。[筛选](../../../raw/current/scripts/componentactions.lua#L1899-L1912) [执行](../../../raw/current/scripts/components/useabletargeteditem.lua#L113-L128)
- **StopUsingItem 会把 target 传给停止回调**：它只传物品自身；由宿主管理目标引用、移除监听和资源撤销。[停止](../../../raw/current/scripts/components/useabletargeteditem.lua#L131-L137)
- **回调忘记 return true 也与 activatable 一样被接受**：这里仅真值使开始成功并置状态，USEITEMON 不套用 ACTIVATE 的 nil 兼容；不要混用两套返回语义。[动作](../../../raw/current/scripts/actions.lua#L3351-L3360)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 中的漂移文件 componentactions.lua（R-04）与 actions.lua（R-01）逐 hunk 对照，锚点重定位 8 处（USEITEMON 候选 1865→1893、动作 3338→3341、INVENTORY 候选 3010→3059 等），结论修订 0 处——USEITEMON／STOPUSINGITEM 候选与动作、ValidTarget OR、骑乘门禁及自身目标回退均未变，同 USEITEM 段新增 spellcaster 处理器（R-04）不触及本页断言；未覆盖：运行验证。

- 源码审阅：2026-10-19，Steam buildid 25643519；覆盖完整组件、候选 OR／骑乘／自身目标、USEITEMON／STOPUSINGITEM、原命名牌筛选／开始／结束与牛铃绑定回调。未全面审校 writeable／RPC、牛铃跨存档迁移、其他材料目标宿主及全部 SG。
- 结构检查：本批格式、来源、链接／行号、大小及登记通过；完整命名牌服务端包装例通过已有 Full Moon Lua 5.1 syntax-only。
- 游戏运行：未运行；有效／伪目标、取消／完成命名、物品销毁、直接重复使用、骑乘与多模组覆盖待实测。
