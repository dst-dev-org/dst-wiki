---
title: "luckitem"
created: "2026-10-24"
updated: "2026-10-28"
type: component
summary: "把携带或装备物品的幸运贡献登记到最终持有者，处理堆叠倍率、动态刷新与来源撤销。"
aliases: ["luckitem", "携带幸运物品", "马蹄铁幸运", "装备幸运值", "updateownerluck"]
tags: [component, item, equipment, callback, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/luckitem.lua", "components/luckuser.lua", "componentutil.lua", "prefabs/horseshoe.lua", "prefabs/oceanfish.lua", "prefabs/hats.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# luckitem

## 快速判断

- 职责与适用：物品端选择携带 `luck` 或装备 `equippedluck`，乘本实体堆叠数，再以物品实体为来源写到最终持有者的 [[reference/components/luckuser]]；两种值**不相加**。[实际计算](../../../raw/current/scripts/components/luckitem.lua#L39-L51)
- 优先使用：已有物品用 `SetLuck(value_or_fn)`／`SetEquippedLuck(value_or_fn)` 配置；运行中改值或回调依赖条件变化后，给物品推 `updateownerluck` 刷新。setter 本身不重算持有者。[配置](../../../raw/current/scripts/components/luckitem.lua#L79-L92) [刷新事件](../../../raw/current/scripts/components/luckitem.lua#L14-L28)
- 重要副作用：入包／易主、装备、堆叠变化会更新贡献；卸装备先撤来源，丢弃／移组件也沿来源协议撤销。不要手动改 owner 的总值来代替来源管理。[监听](../../../raw/current/scripts/components/luckitem.lua#L18-L36) [所有权回调](../../../raw/current/scripts/components/luckitem.lua#L55-L75)
- 修改边界：原版马蹄铁在服务端先加 inventoryitem／stackable 再加本组件；helper 构造直接读 inventoryitem，未配置持有者 luckuser 时不会自动补加。本组件无 netvar 或存档接口。[工厂顺序](../../../raw/current/scripts/prefabs/horseshoe.lua#L48-L62) [helper前提](../../../raw/current/scripts/componentutil.lua#L1833-L1839) [持有者检查](../../../raw/current/scripts/components/luckitem.lua#L46-L51)

## 关键接口

| 接口／字段 | 已核验约定与证据 |
|---|---|
| `SetLuck(luck)`／`GetLuck()` | 初值0；可设数值或 `fn(item, itemsource_owner)`，owner 可 nil、回调须返回数值。Get 使用 FunctionOrValue 原样求值，不含堆叠倍率。[定义](../../../raw/current/scripts/components/luckitem.lua#L18-L22) [接口](../../../raw/current/scripts/components/luckitem.lua#L79-L84) |
| `SetEquippedLuck(luck)`／`GetEquippedLuck()` | 初值0；回调参数同上，仅 IsEquipped 为真时选用。只配携带值不代表装备后继续取携带值。[接口](../../../raw/current/scripts/components/luckitem.lua#L87-L92) [选择](../../../raw/current/scripts/components/luckitem.lua#L47-L50) |
| `UpdateOwnerLuck_Internal(owner?)` | owner 缺省时取 inventoryitem:GetGrandOwner；检查其 luckuser，再把所选值×GetStackSize 登记。显式 owner 不会同时更新 helper 的 itemsource_owner，常规刷新优先走现有事件。[实现](../../../raw/current/scripts/components/luckitem.lua#L39-L51) |
| `RemoveOwnerLuck_Internal(owner?)` | 同样可找最终持有者，调用 RemoveLuckSource(item)；它撤销来源而非写0到全局总值。[实现](../../../raw/current/scripts/components/luckitem.lua#L55-L65) |
| `updateownerluck`／`stacksizechange` | 都转调 UpdateOwnerLuck_Internal；读取的是当前装备、堆叠和配置，不是事件载荷提供的新数值。[回调](../../../raw/current/scripts/components/luckitem.lua#L10-L15) [监听](../../../raw/current/scripts/components/luckitem.lua#L24-L28) |

### 所有权与动态条件

helper 监听物品入包／丢弃／移除，并在直接容器易主时通过 GetGrandOwner 重算最终持有者；owner 改变时先向旧 owner 撤贡献，再向新 owner 登记。它不是每帧遍历所有容器或重算动态回调。[容器监听](../../../raw/current/scripts/componentutil.lua#L1789-L1803) [换主](../../../raw/current/scripts/componentutil.lua#L1805-L1839)

卸装回调走 RemoveOwnerLuck_Internal，**不是**改用携带值的 Update；后续是否重新成为携带来源由实际库存转移／刷新事件决定。自定义装备流必须同时处理两层，不能只手动 PushEvent("unequipped") 就承诺恢复携带贡献。[卸装](../../../raw/current/scripts/components/luckitem.lua#L6-L7) [来源重登](../../../raw/current/scripts/components/luckitem.lua#L70-L75)

## 典型接入

### 将马蹄铁携带贡献翻倍，保留原活动与套装计算

目标模组 `modmain.lua` 完整接入；沿用原版马蹄铁资源，不假定 GLOBAL 代理。原公式按活动与持有者套装动态求值，所以包装原值而非覆盖成一个常量。[原公式](../../../raw/current/scripts/prefabs/horseshoe.lua#L23-L29) [原安装](../../../raw/current/scripts/prefabs/horseshoe.lua#L61-L62)

```lua
AddPrefabPostInit("horseshoe", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local luckitem = inst.components.luckitem
    if luckitem ~= nil then
        local original = luckitem.luck
        luckitem:SetLuck(function(item, owner)
            return GLOBAL.FunctionOrValue(original, item, owner) * 2
        end)
        inst:PushEvent("updateownerluck")
    end
end)
```

`2` 是物品贡献倍率，**不是最终概率翻倍**；堆叠倍率仍由组件另外处理。此 hook 作用于创建／加载的实例，停用模组后新实例回归原工厂，不承诺热卸载。套装条件变化仍需原版刷新：骑士套装对 inventory 的物品推 updateownerluck，本例保留该链路。[套装刷新](../../../raw/current/scripts/prefabs/hats.lua#L6458-L6480) [概率分工](../../../raw/current/scripts/componentutil.lua#L2203-L2214)

## 常见陷阱与限制

- **SetLuck 后总值不变：** setter 仅赋字段，GetLuck 也只是读取；配置后用原事件刷新，回调依赖的技能／套装变化同理。[setter](../../../raw/current/scripts/components/luckitem.lua#L79-L92) [刷新入口](../../../raw/current/scripts/components/luckitem.lua#L14-L15)
- **所有带 luckyitem 标签的物品自动加幸运：** 贡献读组件与持有者，不读该外观标签。海鱼工厂分别配置标签与服务端 luckitem；单加标签没有这份来源效果。[海鱼标签](../../../raw/current/scripts/prefabs/oceanfish.lua#L470-L472) [组件配置](../../../raw/current/scripts/prefabs/oceanfish.lua#L561-L564)
- **把幸运物品当全局随机数修改器：** 组件只登记数值；具体随机路径须主动使用幸运公式。概率与负幸运副作用见 [[reference/components/luckuser#关键接口]]，不能据此保证全部掉落或所有 math.random 都受影响。[登记](../../../raw/current/scripts/components/luckitem.lua#L47-L50)
- **移组件后仍保留贡献：** 原移除钩子撤 helper 来源并去四类额外监听；自定义替代必须保留这条清理，勿只清 luck 字段或仅删一个事件回调。[组件清理](../../../raw/current/scripts/components/luckitem.lua#L31-L36) [helper清理](../../../raw/current/scripts/componentutil.lua#L1842-L1855)

## 验证记录

- 版本增量复核 26148036（2026-10-25）：锚点重定位 6 处（componentutil 来源 helper 段／幸运公式段随 +231 大改整体平移至 1776-1842／2190-2201，逐处内容核对；hats yoth_knight 套装段 -2）；无结论修订——luckitem.lua、horseshoe／oceanfish 未变，hats 本版改动不涉 yoth_knight 内容；componentutil 大改新增块（IsLifeDrainable 等）不在本页引用面。依据：组内共享 componentutil diff＋变更清单命中 componentutil.lua、prefabs/hats.lua。
- 源码审阅：2026-10-24，Steam buildid 25643519；完整 luckitem，核对来源 helper、luckuser 登记、马蹄铁动态公式与工厂、海鱼配置及骑士套装刷新。未全面核验所有库存／装备动作顺序、所有容器转移或其他模组的来源协议。
- 结构检查：本页格式、来源、本地链接与源码锚点严格检查通过；完整 modmain 示例通过 Lua 5.1 语法检查，临时文件已清理。导航登记由本批统一检查。
- 游戏运行：未运行；套装／活动变化、装备与携带切换、堆叠及嵌套容器易主未实测。
