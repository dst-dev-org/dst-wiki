---
title: "socketable"
created: "2026-10-23"
updated: "2026-10-28"
type: component
summary: "为可镶嵌物品声明插槽名称与品质；双端目标筛选、服务端使用入口和实际效果需配合 socketholder 与宿主。"
aliases: ["物品镶嵌", "WX78暗影插槽材料", "插槽品质", "socketable"]
tags: [component, item, actions, callback, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/socketable.lua", "components/socketholder.lua", "componentutil.lua", "constants.lua", "actions.lua", "prefabs/nightmarefuel.lua", "prefabs/shadowheart.lua", "prefabs/shadowheart_infused.lua", "prefabs/wx78_common.lua", "components/useabletargeteditem.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# socketable

## 快速判断

- 职责与适用：物品端保存 `socketname` 与 `socketquality`，由 [[reference/components/socketholder]] 选择同名空槽并消费；组件自身不提供容量、增益或 UI。[定义](../../../raw/current/scripts/components/socketable.lua#L2-L28) [接收方匹配](../../../raw/current/scripts/components/socketholder.lua#L195-L214)
- 优先使用：新物品 Prefab 工厂在双端初始化区调用 `MakeItemSocketable_Client(inst, name)`，服务端再调用 `MakeItemSocketable_Server(inst)` 并设置品质；原版梦魇燃料在现有使用逻辑外接入。[helper](../../../raw/current/scripts/componentutil.lua#L2269-L2300) [原版顺序](../../../raw/current/scripts/prefabs/nightmarefuel.lua#L57-L74)
- 重要副作用：成功镶嵌会拆出堆叠中的一个，执行接收方回调／事件，再保存并删除该物品实体；物品引用不能作为持续生效的装备实体保留。[消费流程](../../../raw/current/scripts/components/socketholder.lua#L279-L312)
- 修改边界：名称在两端用于交互判断，品质由服务端传给接收方 netvar；本组件没有 netvar 或存档钩子，不会自行同步／保存运行期修改。容器保留实体的需求另用 [[reference/components/container]]。[完整组件](../../../raw/current/scripts/components/socketable.lua#L2-L32) [品质写入](../../../raw/current/scripts/components/socketholder.lua#L297-L311)

## 关键接口

| 接口／入口 | 条件、作用与证据 |
|---|---|
| `SetSocketName(name)`／`GetSocketName()` | 初始值 `"socket_DEFAULT"`；setter 直接赋值，无类型／枚举校验。接收方按名称 hash 匹配，不按物品 Prefab；正常使用非空名称并与槽配置一致。[字段与接口](../../../raw/current/scripts/components/socketable.lua#L2-L17) [槽匹配](../../../raw/current/scripts/components/socketholder.lua#L71-L85) |
| `SetSocketQuality(quality)`／`GetSocketQuality()` | 初始 `SOCKETQUALITY.NONE`；setter 不校验范围。当前枚举 NONE=0、LOW=1、MEDIUM=2、HIGH=3、PERFECT=4；接收方据品质选择行为，名字相同不代表效果相同。[接口](../../../raw/current/scripts/components/socketable.lua#L23-L28) [枚举](../../../raw/current/scripts/constants.lua#L3035-L3046) [WX78消费](../../../raw/current/scripts/prefabs/wx78_common.lua#L715-L743) |
| `MakeItemSocketable_Client(inst, name)` | 创建／复用组件、设置名字并包装 `inst.UseableTargetedItem_ValidTarget`：先尝试目标的 `CanTryToSocket`，未成功再调用旧筛选。不是仅客户端执行，须放双端工厂区。[包装器](../../../raw/current/scripts/componentutil.lua#L2255-L2277) |
| `MakeItemSocketable_Server(inst)` | 创建／复用 [[reference/components/useabletargeteditem]]，保存原 `onusefn` 后改为镶嵌包装器；目标 `TryToSocket` 成功即返回，否则回退旧回调。它不会创建 socketable 或设置名称／品质。[服务器包装](../../../raw/current/scripts/componentutil.lua#L2279-L2300) |

## 典型接入

### 新物品既能镶嵌，又保留已有的对目标使用逻辑

目标是模组自己的可拾取 Prefab 工厂，已有网络、资源与 [[reference/components/inventoryitem]] 基础；以下是组装顺序指引，不是一份可独立加载的 Prefab demo：

1. 双端先定义实体的 `UseableTargetedItem_ValidTarget(item,target,doer)`（如有旧逻辑），再调用 `MakeItemSocketable_Client`；必须在 `SetPristine()` 和客户端返回之前提供，名称与接收方空槽一致。[梦魇燃料](../../../raw/current/scripts/prefabs/nightmarefuel.lua#L57-L67)
2. 服务端先建立原 `useabletargeteditem` 与旧 `onusefn`，再调用 `MakeItemSocketable_Server` 包装，最后设置服务端品质；否则后续 `SetOnUseFn` 会覆盖镶嵌入口。原版通过 `WX78Common.MakeItemSocketable` 调服务端 helper，额外允许自用、骑乘以及成功后不切换使用中状态；这些额外选项不是基础 helper 默认行为。[工厂调用](../../../raw/current/scripts/prefabs/nightmarefuel.lua#L69-L74) [WX78扩展](../../../raw/current/scripts/prefabs/wx78_common.lua#L589-L594)
3. 新名称与物品使用动作文字一起配置：`USEITEMON.strfn` 优先使用实体的 `GetUseItemOnVerb`，否则取 socketname 大写；原版因此可针对已装暗影插槽改用“消耗”语义。[动作取词](../../../raw/current/scripts/actions.lua#L3327-L3332) [原版特例](../../../raw/current/scripts/prefabs/nightmarefuel.lua#L25-L35)
4. 镶嵌增益和撤销写在接收方，不放在待删除物品的周期任务内；装入、读档恢复都可能让接收方以 `doer=nil` 调处理函数，详见 [[reference/components/socketholder#生命周期]]。[回调时机](../../../raw/current/scripts/components/socketholder.lua#L297-L311) [恢复](../../../raw/current/scripts/components/socketholder.lua#L395-L401)

原版品质参照：梦魇燃料为 LOW、暗影心脏为 HIGH、注能暗影心脏为 PERFECT。它们均采用同一 SHADOW 名称，由接收方选最高品质，并非本组件自行产生属性加成。[梦魇燃料](../../../raw/current/scripts/prefabs/nightmarefuel.lua#L73-L74) [暗影心脏](../../../raw/current/scripts/prefabs/shadowheart.lua#L55-L64) [注能心脏](../../../raw/current/scripts/prefabs/shadowheart_infused.lua#L111-L126) [消费点](../../../raw/current/scripts/prefabs/wx78_common.lua#L715-L743)

## 常见陷阱与限制

- **只有 AddComponent，没有可执行动作：** 组件只存两项数据；需要上述目标筛选与服务端 useabletargeteditem 包装链，普通 `USEITEMON` 只调用后者的 `StartUsingItem`。[动作](../../../raw/current/scripts/actions.lua#L3351-L3360) [helper](../../../raw/current/scripts/componentutil.lua#L2269-L2300)
- **把原筛选拒绝当作服务器权限限制：** socketable 包装器先接受可镶嵌目标，旧筛选只是失败回退；镶嵌失败也可能继续旧 onuse。权威限制放接收方 `SetShouldAllowSocketableFn_SERVER`，并检查旧回退是否仍允许别的使用效果。[筛选／回退](../../../raw/current/scripts/componentutil.lua#L2255-L2294) [权威门禁](../../../raw/current/scripts/components/socketholder.lua#L315-L328)
- **读档后品质丢失或名字变化：** 组件无 OnSave／OnLoad；名称和品质通常由物品工厂重新配置。动态品质若要保留，须由物品自己的存档协作恢复，不能假定两个字段自动序列化。[完整实现](../../../raw/current/scripts/components/socketable.lua#L2-L32) [接收方重建物品](../../../raw/current/scripts/components/socketholder.lua#L256-L275)
- **重复调用 helper 包装同一实例：** 服务端每次都把当前 onuse 存为旧回调；重复包装可能把包装器自身存入回退链。双端 helper 也不是保证幂等的注册接口，每个工厂初始化一次，原筛选／回调在包装前装好。[实现](../../../raw/current/scripts/componentutil.lua#L2269-L2300)

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 处理范围：锚点重定位 11 处（componentutil／actions／constants／shadowheart 系行号迁移，区间文本逐段比对一致；SOCKETQUALITY 枚举未变）；结论修订：无，仅锚点迁移。
- 依据：枢纽 diff 审阅（actions.lua／componentactions.lua hunk 地图）＋变更清单命中 actions.lua、componentutil.lua、constants.lua、prefabs/shadowheart.lua、prefabs/shadowheart_infused.lua。

- 源码审阅：2026-10-23，`steam-buildid:25643519`；覆盖本组件完整实现、helper 包装链、接收方匹配／消费／恢复、动作取词与原版燃料／心脏／WX78品质消费，依据见上述锚点。
- 游戏运行：未运行；未验证自定义物品的动作文本、控制器交互、资源或动态品质存档。本文无完整 Lua demo，不作示例语法通过声明。
- 结构检查：本页元数据、大小和链接、受影响导航及全库登记检查通过；源码引用 strict 检查无错误／警告，不代替行为与运行验证。

相关：[[reference/components/socketholder]]、[[reference/components/useabletargeteditem]]、[[reference/components/stackable]]。
