---
title: "cookable"
created: "2026-10-17"
updated: "2026-10-28"
type: component
summary: "配置食材直接烤制的产物与烤前回调，区分锅中料理和晾晒。"
aliases: ["烤熟产物", "烤肉变肉干", "直接烤食物", "cookable"]
tags: [component, item, food, callback]
dst_build: "steam-buildid:26230496"
sources: ["components/cookable.lua", "components/cooker.lua", "actions.lua", "prefabs/meats.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# cookable

## 快速判断

- 职责与适用：挂在食材上，决定直接烤制产物；处理设备用 [[reference/components/cooker]]，锅中配方料理用 [[reference/components/stewer]]，晾晒用 [[reference/components/dryable]]。[Cook](../../../raw/current/scripts/components/cookable.lua#L18)
- 优先使用：`product` 指定已有 Prefab 或选择函数；`SetOnCookedFn` 处理烤前副作用。[产物选择](../../../raw/current/scripts/components/cookable.lua#L22)
- 重要副作用：先回调、后生成产物；符合条件时新鲜度恢复一半已损失比例。本组件不删除原料、不发放产物。[生成与新鲜度](../../../raw/current/scripts/components/cookable.lua#L18-L35)
- 修改边界：服务端修改已有组件；新增食材需在共同初始化、`SetPristine` 前声明 `cookable` 标签，原版肉类服务端再装组件。[工厂分界](../../../raw/current/scripts/prefabs/meats.lua#L169-L180)

## 关键接口

| 接口／字段 | 签名或约定 | 条件、作用与证据 |
|---|---|---|
| `product` | Prefab 名或 `fn(food, cooker, chef) → Prefab名` | 默认 nil；函数返回值仍交给 SpawnPrefab，不是返回实体。[选择与生成](../../../raw/current/scripts/components/cookable.lua#L22-L27) |
| `SetOnCookedFn(fn)` | `fn(food, cooker, chef)` | 单回调槽；在产物选择／生成前执行，返回值不作门禁；没有产物参数。[设置](../../../raw/current/scripts/components/cookable.lua#L14)／[调用](../../../raw/current/scripts/components/cookable.lua#L18-L20) |
| `Cook(cooker, chef)` | 成功生成则返回实体，否则 nil | 原料与产物都有 perishable 且原料非 smallcreature 时，产物比例设为 `1-(1-原料比例)*0.5`。[新鲜度](../../../raw/current/scripts/components/cookable.lua#L29-L35) |

## 典型接入

### 让大肉直接烤出肉干

目标模组 `modmain.lua`；复用原版 `meat` 及其已声明的 `meat_dried` 依赖，不增加资源或新物品。修改食材产物，保留原版动作／炉具／消费；不经过晾晒计时。[肉类依赖](../../../raw/current/scripts/prefabs/meats.lua#L32-L37)／[原版配置](../../../raw/current/scripts/prefabs/meats.lua#L384)／[PostInit](../../../raw/current/scripts/modutil.lua#L593)

```lua
AddPrefabPostInit("meat", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    if inst.components.cookable ~= nil then
        inst.components.cookable.product = "meat_dried"
    end
end)
```

只影响直接烤 `meat`，不改变厨锅食谱或晾晒。模组停用后新建／读档实例按原版工厂配置；已生成肉干不回退。其他模组后续改同一 product 时以实际顺序为准；测试鲜／半鲜大肉、整堆取一件和火堆燃料耗尽。

## 常见陷阱与限制

- **直接调用 Cook 当成完整烹饪动作：** 它只生成产物，原料仍在。原版 COOK 动作负责拆分／转移，cooker 删除交给它的原料，动作再发放产物；不要自行多删一件或把整堆直接交给 CookItem。[动作](../../../raw/current/scripts/actions.lua#L2013-L2035)／[另一拆堆路径](../../../raw/current/scripts/actions.lua#L2067-L2092)／[删除](../../../raw/current/scripts/components/cooker.lua#L47)
- **oncooked 等于成功通知：** 回调发生在 SpawnPrefab 前；即使没有产物也可能已发生副作用。需要访问生成结果时在 cooker 的完成回调处理。[先后顺序](../../../raw/current/scripts/components/cookable.lua#L18-L27)／[结果回调](../../../raw/current/scripts/components/cooker.lua#L43-L48)
- **所有加工都保留相同新鲜度：** 本页公式只适用 Cook 的所述条件；不适用于小生物、晾晒或所有厨锅。新鲜度消费者另见 [[reference/components/perishable]]。[排除条件](../../../raw/current/scripts/components/cookable.lua#L29-L33)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：sources 变更文件 actions.lua 对照更新报告 R-01 与 hunk 地图逐 hunk 核验。锚点重定位 2 处（COOK 动作主体移至 actions.lua:2010-2032，手持炉具拆堆路径移至 :2064-2089，均 +3 行移、内容逐行一致）；COOK 动作无语义变化，本批施法门控／禁长按／拾取放宽均不触及。未覆盖：运行验证。

- 源码审阅：2026-10-17，Steam buildid 25643519；覆盖 product／回调顺序、新鲜度条件、COOK 分工及 meat 工厂／依赖与上述示例。未覆盖全部炉具、特殊食材和厨师技能。
- 结构／示例检查：本批格式、来源文件、链接／行锚点、导航登记与完整示例 Lua 5.1 语法检查通过；详见工程检视，不代表 API 或效果运行通过。
- 游戏运行：未运行；多人烹饪、堆叠消费、读档与其他模组覆盖仍需验证。
