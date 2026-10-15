---
title: "forcecompostable"
created: "2026-10-24"
updated: "2026-10-28"
type: component
summary: "把物品明确归为堆肥绿料或棕料，区分动作候选、桶的材料判定与实际消费。"
aliases: ["forcecompostable", "强制可堆肥", "绿料棕料", "非食物堆肥", "堆肥候选却无法投料"]
tags: [component, item, actions, food]
dst_build: "steam-buildid:26230496"
sources: ["components/forcecompostable.lua", "components/compostingbin.lua", "prefabs/compostingbin.lua", "componentactions.lua", "actions.lua", "prefabs/hambat.lua", "prefabs/pinecone.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# forcecompostable

## 快速判断

- 职责与适用：物品端只有 `green`／`brown` 两个分类字段，默认都 false；不自带方法、标签、定时或产物。原版肉棒可作绿料，松果可作棕料。[完整定义](../../../raw/current/scripts/components/forcecompostable.lua#L1-L7) [肉棒配置](../../../raw/current/scripts/prefabs/hambat.lua#L80-L81) [松果配置](../../../raw/current/scripts/prefabs/pinecone.lua#L116-L117)
- 优先使用：新物品需要覆盖食物的堆肥分类，或允许非食物投料时，在服务端物品工厂添加本组件并明确设一种分类；桶的处理、计时与产物继续用 [[reference/components/compostingbin]]。[桶判定](../../../raw/current/scripts/prefabs/compostingbin.lua#L191-L212)
- 重要副作用：桶先读本组件，再考虑 edible；两字段都 false 会直接拒绝，**不会回退食物判定**。两者都真则 green 优先。[优先级](../../../raw/current/scripts/prefabs/compostingbin.lua#L196-L206)
- 修改边界：原版示例只在服务端添加；动作候选只检查组件存在和目标标签，不读两个分类值，服务端 AddCompostable 才最终判料／消费。[端分界](../../../raw/current/scripts/prefabs/hambat.lua#L65-L68) [候选](../../../raw/current/scripts/componentactions.lua#L1273-L1276) [执行](../../../raw/current/scripts/actions.lua#L5540-L5543)

## 关键接口

本组件没有 setter 或投料方法，直接配置下列字段；实际行为由读取方定义。[定义](../../../raw/current/scripts/components/forcecompostable.lua#L1-L7)

| 字段／读取方 | 已核验规则与证据 |
|---|---|
| `green`／`brown` | 初值 false／false；原桶按真值选绿／棕，固定贡献 `{greens=1,browns=0}` 或相反。不是材料数量，设成 3 也不代表三份。[字段](../../../raw/current/scripts/components/forcecompostable.lua#L4-L5) [消费定义](../../../raw/current/scripts/prefabs/compostingbin.lua#L197-L200) |
| `ADDCOMPOSTABLE` 候选 | 目标有 `compostingbin_accepts_items` 即提供动作；不会预查满桶、分类字段或组件回调。[收集器](../../../raw/current/scripts/componentactions.lua#L1273-L1276) |
| 桶的 `calcmaterialvalue(inst,item)` | 原版先拒绝满桶，再优先处理本组件；只有组件缺失才按 edible 的 foodtype 判断。材料边界与容量归 [[reference/components/compostingbin#关键接口]]。[原版函数](../../../raw/current/scripts/prefabs/compostingbin.lua#L191-L212) |

## 典型接入

### 给非食物物品指定堆肥类型

在目标模组自己的物品 Prefab **服务端构造区**添加组件，将 `green=true` 或 `brown=true`，另一项保持 false；物品自身的资源、拾取与堆叠仍由原工厂负责，见 [[reference/components/inventoryitem]]、[[reference/components/stackable]]。这是装配指引，不是一份完整新物品 demo。

| 原版模式 | 配置位置与选择理由 |
|---|---|
| 肉棒作绿料 | `hambat` 在服务端添加并设 green；无需把武器改成食物才能获得该分类。[端界](../../../raw/current/scripts/prefabs/hambat.lua#L65-L68) [标记](../../../raw/current/scripts/prefabs/hambat.lua#L76-L81) |
| 松果作棕料 | 松果公共工厂先过服务端返回，再添加并设 brown；燃料／部署能力与堆肥分类分别配置。[端界](../../../raw/current/scripts/prefabs/pinecone.lua#L89-L91) [燃料](../../../raw/current/scripts/prefabs/pinecone.lua#L102-L103) [标记](../../../raw/current/scripts/prefabs/pinecone.lua#L116-L117) |

真正投料仍由目标桶执行：先判料，失败不消费；成功先调投料回调，再拆一件或删除单件，最后加入材料并刷新。消费、计时与产物细节复用 [[reference/components/compostingbin#关键接口]]，不在物品标记上再写一套产物逻辑。[组件投料顺序](../../../raw/current/scripts/components/compostingbin.lua#L132-L153)

## 常见陷阱与限制

- **只加组件却保留默认值：** 客户端可能出现投料候选，原桶却返回 nil；已有 edible 也不回退。应明确选择分类，不能用“存在组件”代表已经配置。[候选](../../../raw/current/scripts/componentactions.lua#L1273-L1276) [拒绝分支](../../../raw/current/scripts/prefabs/compostingbin.lua#L196-L206)
- **两种材料都设真以获得混合贡献：** 原版先匹配 green，只计一份绿料；混合贡献属于桶的材料计算策略，不是两个布尔字段相加。[优先级](../../../raw/current/scripts/prefabs/compostingbin.lua#L197-L200)
- **视为无限容量或食物类型检查：** 原桶仍先检查容量，但该组件分支绕过 edible 的食物类型分类／排除表；若不想覆盖原规则，不要无条件给所有食物挂此组件。[满桶检查](../../../raw/current/scripts/prefabs/compostingbin.lua#L191-L204) [普通食物排除表](../../../raw/current/scripts/prefabs/compostingbin.lua#L16-L21)
- **认为字段自动保存／同步：** 七行定义没有存档或 netvar；原版在工厂重建分类。动态切换若需持久化应由物品宿主负责，客户端动作收集也不读取这两个服务端值。[完整定义](../../../raw/current/scripts/components/forcecompostable.lua#L1-L7) [动作读取](../../../raw/current/scripts/componentactions.lua#L1273-L1276)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页漂移文件 actions.lua（R-01）、componentactions.lua（R-04）对照 build-26148036 更新报告与 actions／componentactions hunk 地图逐条核验；ADDCOMPOSTABLE 动作与 forcecompostable 动作候选均未在变更 hunk 内，结论不变，锚点重定位 1 处、结论修订 0 处；未覆盖：运行验证。

- 源码审阅：2026-10-24，Steam buildid 25643519；完整七行定义、原桶分类／容量与消费、动作收集／执行、肉棒与松果工厂。桶计时和通用堆叠行为复用同版本已核验页；未全审所有可堆肥物或自定义桶。
- 结构检查：本页格式、来源、本地链接与源码引用严格检查通过；无完整 Lua 示例，导航由本批统一登记后检查。
- 游戏运行：未运行；分类切换、候选与执行差异及多人投料未实测。
