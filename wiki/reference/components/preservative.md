---
title: "preservative"
created: "2026-10-22"
updated: "2026-10-28"
type: component
summary: "配置盐等消耗品一次性恢复目标新鲜度的比例及目标堆叠分摊。"
aliases: ["盐补新鲜度", "恢复食材新鲜度", "一次性防腐剂", "preservative"]
tags: [component, item, food, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/preservative.lua", "actions.lua", "componentactions.lua", "prefabs/saltrock.lua", "tuning.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# preservative

## 快速判断

- 职责与适用：挂在消耗品上，供 `APPLYPRESERVATIVE` 一次性提高目标新鲜度；持续降低容器腐败倍率用 [[reference/components/preserver]]。[组件定义](../../../raw/current/scripts/components/preservative.lua#L1-L7)、[动作实现](../../../raw/current/scripts/actions.lua#L4620-L4644)
- 优先使用：调整原版 `saltrock` 的恢复比例，沿用原有资源、堆叠与动作；其实际注册名为 `saltrock`。[宿主初始化](../../../raw/current/scripts/prefabs/saltrock.lua#L62-L81)
- 重要副作用：动作先改变目标新鲜度，再从施用者 inventory 移出并删除材料；不是事务式“扣除成功后才恢复”。[动作顺序](../../../raw/current/scripts/actions.lua#L4627-L4641)
- 修改边界：组件仅承载配置字段，服务端宿主在主模拟分支添加它；客户端动作候选还检查右键、目标标签和 replica。[服务端宿主](../../../raw/current/scripts/prefabs/saltrock.lua#L47-L70)、[动作候选](../../../raw/current/scripts/componentactions.lua#L1574-L1582)

## 关键接口

| 字段／消费者 | 约定 | 条件、作用与证据 |
|---|---|---|
| `percent_increase` | 数值，默认 1 | 新鲜度比例增量，0.25 表示增加 25 个百分点；盐默认取 `TUNING.SALTROCK_PRESERVE_PERCENT_ADD = 0.5`。[默认字段](../../../raw/current/scripts/components/preservative.lua#L1-L7)、[盐配置](../../../raw/current/scripts/prefabs/saltrock.lua#L69-L70)、[调参](../../../raw/current/scripts/tuning.lua#L4114) |
| `divide_effect_by_stack_size` | boolean，默认 true | 目标有 stackable 时，增量除以**目标**当前堆叠数量；false 或目标无堆叠则直接用增量。[计算入口](../../../raw/current/scripts/actions.lua#L4627-L4633) |
| `ACTIONS.APPLYPRESERVATIVE` | 正常物品使用动作 | 服务端要求目标无 health，带 fresh／stale／spoiled 之一及 cookable，且无 deployable；动作候选另外排除 smallcreature。不能拿任意 perishable 物品作为可施用保证。[服务端门禁](../../../raw/current/scripts/actions.lua#L4620-L4625)、[客户端门禁](../../../raw/current/scripts/componentactions.lua#L1574-L1582) |

最终比例由 [[reference/components/perishable#关键接口]] 的 `SetPercent` 夹在 0～1；拆取材料的默认单件和归属边界复用 [[reference/components/inventory#关键接口]]。本组件无独立方法、任务、保存或清理回调；配置由宿主工厂重建，而目标新鲜度保存归 perishable。[组件全文](../../../raw/current/scripts/components/preservative.lua#L1-L9)

## 典型接入

### 把每件盐的总恢复量改为 25 个百分点，保持堆叠分摊

目标模组 `modmain.lua` 加以下完整代码，无新增资源。只改服务端 `saltrock` 既有组件，不替换动作、堆叠或修盐舔块能力；注册与原能力见 [saltrock 工厂](../../../raw/current/scripts/prefabs/saltrock.lua#L62-L81)，入口见 [AddPrefabPostInit](../../../raw/current/scripts/modutil.lua#L593-L599)。

```lua
AddPrefabPostInit("saltrock", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local preservative = inst.components.preservative
    if preservative ~= nil then
        preservative.percent_increase = 0.25
    end
end)
```

原版默认分摊仍生效：10 件目标食材的一堆得到 0.025 的比例增量，单件得到 0.25，最终最多到 1；不延长最大保质期，也不改变之后的腐败速率。修改常量即替换恢复量；停用后重新创建的盐恢复原厂配置，已经恢复或消耗的物品不回滚。已有实例需重新加载工厂才应用此 postinit。

## 常见陷阱与限制

- **挂组件就能给任何食物施用**：动作要求 cookable 等标签，且直接访问目标 perishable；自制对象须满足真实组件契约，不能只伪造标签。[动作门禁与调用](../../../raw/current/scripts/actions.lua#L4620-L4633)
- **一堆目标都增加完整比例**：默认按目标数量分摊；若有意改为每件均获相同提升，显式把 `divide_effect_by_stack_size` 设 false，并接受每次只消耗材料一件的经济变化。[比例计算与扣料](../../../raw/current/scripts/actions.lua#L4627-L4638)
- **动作返回 false 表示没有副作用**：若 RemoveItem 返回 nil，目标已经被 SetPercent 修改。自定义远程调用／扣料规则需自行处理先后与失败边界，不直接复制此动作用于交易。[动作顺序](../../../raw/current/scripts/actions.lua#L4627-L4641)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页漂移文件 actions.lua（R-01）、componentactions.lua（R-04）、tuning.lua 对照 build-26148036 更新报告与 actions／componentactions hunk 地图逐条核验；componentactions.lua 客户端候选判定（APPLYPRESERVATIVE，componentactions.lua:1574-1582）由逐个 HasTag 改为 HasAnyTag("fresh","stale","spoiled")／HasAnyTag("deployable","smallcreature")，逻辑逐项等价，页内门禁结论不变；APPLYPRESERVATIVE 动作定义与 fn、saltrock 调参均未变，锚点重定位 8 处、结论修订 0 处；未覆盖：运行验证。

- 源码审阅：2026-10-22，Steam buildid 25643519；覆盖两个配置字段、动作候选／执行门禁、目标堆叠分摊、扣料顺序、saltrock 注册及示例；新鲜度夹值与 inventory 拆取复用同版本页面。未审校全部可施用目标或其他模组动作改写。
- 结构检查：定向页面校验通过；源码锚点 strict 校验无错误／警告；完整 modmain 示例通过 Full Moon Lua 5.1 语法检查，未执行游戏逻辑。
- 游戏运行：未运行；多人施用、堆叠极值、存档重载和其他模组兼容待游戏验证。
