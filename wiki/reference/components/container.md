---
title: "container"
created: "2026-10-11"
updated: "2026-10-28"
type: "component"
summary: "为可拾取容器或建筑配置格子、过滤、开关与内容转移，区分布局和服务端行为。"
aliases: ["可拾取容器", "容器格子", "背包过滤", "箱子", "容器删除", "container"]
tags: ["component", "item", "building", "lifecycle", "server-client"]
dst_build: "steam-buildid:26230496"
sources: ["componentactions.lua", "components/container.lua", "components/container_replica.lua", "components/edible.lua", "containers.lua", "entityreplica.lua", "entityscript.lua", "prefabs/backpack.lua", "prefabs/container_classified.lua"]
reliability: "verified"
validation: "source-reviewed"
status: "active"
---

# container

## 快速判断

- 职责与适用：实体内部的格子；容器本身能否被拾取由 inventoryitem 决定，携带／装备开关由相应生命周期控制。[prefabs/backpack.lua:268](../../../raw/current/scripts/prefabs/backpack.lua#L268
- 优先使用：WidgetSetup 设置布局，CanTakeItemInSlot／GiveItem 收纳，Open／Close 管理交互。[components/container.lua:86](../../../raw/current/scripts/components/container.lua#L86
- 重要副作用：GiveItem 可合并并删除输入实体；itemget／itemlose 在容器宿主上发送。开关回调形参不对称。[components/container.lua:542](../../../raw/current/scripts/components/container.lua#L542 [components/container.lua:658](../../../raw/current/scripts/components/container.lua#L658 [components/container.lua:711](../../../raw/current/scripts/components/container.lua#L711
- 修改边界：格子／过滤配置必须让两端获得；服务端独有组件不代替客户端布局。保鲜另见 [[reference/components/preserver]]。[components/container_replica.lua:232](../../../raw/current/scripts/components/container_replica.lua#L232

## 关键接口

| 接口／字段 | 约定 | 作用与证据 |
|---|---|---|
| `WidgetSetup(prefab?, data?)` | 原版 params 键或完整配置表 | 同步配置对象；WidgetSetup 后 numslots／widget／itemtestfn 等被标为只读，应重设配置而非直接赋值。[components/container.lua:86](../../../raw/current/scripts/components/container.lua#L86 [containers.lua:9](../../../raw/current/scripts/containers.lua#L9 |
| `itemtestfn` | `(container, item, slot?) -> boolean` | 与 inventoryitem 权限、槽位等共同判断；客户端不能依赖服务端 components。[components/container.lua:269](../../../raw/current/scripts/components/container.lua#L269 |
| `GiveItem(item, slot?, src_pos?, drop_on_fail?)` | boolean | 不自动脱离旧 owner；拒收默认丢地，false 也可能已接收一部分堆叠。[components/container.lua:447](../../../raw/current/scripts/components/container.lua#L447 [components/container.lua:534](../../../raw/current/scripts/components/container.lua#L534 |
| `RemoveItem(item, wholestack?, ..., keepoverstacked?)` | 移出对象或 nil | 默认堆叠取一件；RemoveItemBySlot 取整槽；找不到也可能原样返回 item。[components/container.lua:1009](../../../raw/current/scripts/components/container.lua#L1009 [components/container.lua:569](../../../raw/current/scripts/components/container.lua#L569 |
| `Open(doer)`／`Close(doer?)` | Close 省略参数关闭所有打开者 | Open 本身不检查 canbeopened；业务调用需先检查自己的开门条件。[components/container.lua:617](../../../raw/current/scripts/components/container.lua#L617 [components/container.lua:674](../../../raw/current/scripts/components/container.lua#L674 |
| `onopenfn`／`onclosefn` | `(inst, {doer=...})`／`(inst, doer)` | 第一位打开者／最后一位关闭者触发；多用户逐次事件另查 onanyopenfn／onanyclosefn。[components/container.lua:658](../../../raw/current/scripts/components/container.lua#L658 [components/container.lua:711](../../../raw/current/scripts/components/container.lua#L711 |

## 典型接入

### 让原版背包只接收具有食物类型标签的物品

目标模组 `modmain.lua`；复用原版 backpack 布局／资源及穿脱回调。在两端注册独立配置键；服务端 postinit 设置，客户端等 OnEntityReplicated 后设置，保留旧回调。基础布局见 [containers.lua:23](../../../raw/current/scripts/containers.lua#L23)，服务端入口见 [prefabs/backpack.lua:277](../../../raw/current/scripts/prefabs/backpack.lua#L277)，客户端构造／回调顺序见 [entityreplica.lua:75](../../../raw/current/scripts/entityreplica.lua#L75)。

```lua
local containers = GLOBAL.require("containers")
local config = {}
for key, value in pairs(containers.params.backpack) do
    config[key] = value
end
local previous_test = config.itemtestfn
config.itemtestfn = function(container, item, slot)
    for _, foodtype in pairs(GLOBAL.FOODTYPE) do
        if item:HasTag("edible_"..foodtype) then
            return previous_test == nil or previous_test(container, item, slot)
        end
    end
    return false
end
containers.params.dst_wiki_food_backpack = config

AddPrefabPostInit("backpack", function(inst)
    if GLOBAL.TheWorld.ismastersim then
        if inst.components.container ~= nil then
            inst.components.container:WidgetSetup("dst_wiki_food_backpack")
        end
    else
        local previous_replicated = inst.OnEntityReplicated
        inst.OnEntityReplicated = function(item)
            if previous_replicated ~= nil then previous_replicated(item) end
            if item.replica.container ~= nil then
                item.replica.container:WidgetSetup("dst_wiki_food_backpack")
            end
        end
    end
end)
```

过滤依据两端可见的 edible_类型 标签（[components/edible.lua:9](../../../raw/current/scripts/components/edible.lua#L9)，原版同类遍历见 [entityscript.lua:727](../../../raw/current/scripts/entityscript.lua#L727），并不证明任意角色都能吃，包括某些角色才吃的木／石等类型；未深拷贝 widget，也不修改它。停用后新实例恢复原过滤。加载旧存档时不合格内容可能被 GiveItem 拒收／丢落；先备份测试，示例不提供内容迁移。也不保证与其他修改同一布局的模组兼容。

### 新容器的客户端布局

新增 Prefab 时，先在双方加载的代码中注册 `containers.params.<键>`（widget.slotpos、animbank／animbuild、pos、type、过滤）；服务端添加 container 后 WidgetSetup，客户端在 replica 可用时设置同键。自定义键无法从服务端自动传递配置表。新的槽数超过原版最大值时，还须在创建 container_classified 前提高 MAXITEMSLOTS；不能事后补网络槽。此处是接入清单，完整新物品工厂／资源不在本页复制。[containers.lua:9](../../../raw/current/scripts/containers.lua#L9 [containers.lua:2495](../../../raw/current/scripts/containers.lua#L2495 [components/container_replica.lua:22](../../../raw/current/scripts/components/container_replica.lua#L22 [prefabs/container_classified.lua:1057](../../../raw/current/scripts/prefabs/container_classified.lua#L1057

### 可拾取且未装备的容器

本地片段参照（未随库分发，未测游戏／外部 Hook；[版本与文件指纹](../../../docs/reports/w05/local-mod-references-20261017.md)）：三模组中有“携带袋／箱”组合：MCW 的 mcw_gembox.lua:214～230 分别配置 inventoryitem、container、deployable、preserver；Sora 的 sora3chest.lua:66～92 把主栏限制、容器开关、无限堆叠和保鲜分开。这里只作设计参照，不复制其专有管理器／资源，也不把 onpickup 当作入栏成功。

通用实现先用 [[reference/components/inventory#携带范围的选择]] 定义主栏／背包／地面能否打开；格子能力在本页，进出事件顺序在 [[reference/components/inventoryitem#生命周期]]，保鲜在 [[reference/components/preserver]]。携带方式不自动获得原版 backpack 的装备开关逻辑；正常库存开包动作消费 replica.CanBeOpened 等条件，直接 Open 仍须自己校验。[componentactions.lua:2605](../../../raw/current/scripts/componentactions.lua#L2605 [components/container.lua:617](../../../raw/current/scripts/components/container.lua#L617

### 超堆叠与删除内容

`EnableInfiniteStackSize(true)` 管容器内物品忽略上限，关闭／取出需处理超堆叠余量；网络显示限制见 [[reference/components/stackable#常见陷阱与限制]]。删除容器实体或移组件的关闭清理不自动 DropEverything；业务若要求散落内容，须在删除前显式调用并检查特殊储存条件。存档由 OnSave／OnLoad 保存和重新 GiveItem 内容。[components/container.lua:1565](../../../raw/current/scripts/components/container.lua#L1565 [components/container.lua:1040](../../../raw/current/scripts/components/container.lua#L1040 [components/container.lua:979](../../../raw/current/scripts/components/container.lua#L979 [components/container.lua:1086](../../../raw/current/scripts/components/container.lua#L1086

## 常见陷阱与限制

- **给 container 添加 inventoryitem 就自动获得开包逻辑**：只能让宿主可拾取，开关时机仍需设计；原版 backpack 用装备流程，携带包可选择入栏范围。[prefabs/backpack.lua:268](../../../raw/current/scripts/prefabs/backpack.lua#L268
- **转移只调用目标 GiveItem**：源槽位可能残留；先走原 owner 的移出入口，保留失败／部分接收处理。不要把 false 简化为“完全没动”。[components/container.lua:534](../../../raw/current/scripts/components/container.lua#L534 [components/container.lua:1009](../../../raw/current/scripts/components/container.lua#L1009
- **把 currentuser 当可靠操作者**：源码明确提示可能 nil 或不正确；操作回调使用实际 doer，并核验权限。[components/container.lua:57](../../../raw/current/scripts/components/container.lua#L57

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 的漂移文件（componentactions.lua、containers.lua、entityscript.lua）对照 build-26148036 更新报告（R-04/R-07/R-08）逐条核验，containers.lua 的 socket_keystone／quagmire_pot 改动与 entityscript.lua 的湿度／距离改动不触及本页结论；componentactions.lua container 开包候选与 entityscript.lua:705 edible 遍历逐行未变仅移位，containers.lua MAXITEMSLOTS 汇总行随文件尾部删行上移；锚点重定位 2 处、结论修订 0 处；未覆盖：运行验证。

- 源码依据复核：2026-10-17，Steam buildid 25643519；核对范围：双方 WidgetSetup／复制后配置、GiveItem 部分接收、开关回调参数与食物过滤背包示例，依据见上文对应锚点；未覆盖自定义多槽网络扩容的完整工程与旧内容迁移。
- 历次结构／语法检查：元数据、源码路径／行号、链接／章节、标签、大小和导航登记检查通过；本页 Lua 代码经 Full Moon 3.0.0／Lua 5.1 语法及原字节往返检查。
- 游戏运行：未运行；案例的多人交互、存档重载和其他模组兼容仍需游戏验证。

相关：[[reference/components/inventoryitem]]；[[reference/components/preserver]]；[[reference/components/stackable]]。
