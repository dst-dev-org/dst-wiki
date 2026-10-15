---
title: "cooker"
created: "2026-10-17"
updated: "2026-10-28"
type: component
summary: "配置火堆类直接烤制设备的门禁、生成结果回调与原料消费边界。"
aliases: ["火堆烤食物", "烤熟恢复新鲜", "烹饪设备", "cooker"]
tags: [component, food, callback, building]
dst_build: "steam-buildid:26230496"
sources: ["components/cooker.lua", "components/cookable.lua", "components/perishable.lua", "actions.lua", "prefabs/firepit.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# cooker

## 快速判断

- 职责与适用：处理单件食材的直接烤制设备，如石头营火；产物来自食材 [[reference/components/cookable]]，锅中四格配方用 [[reference/components/stewer]]。[门禁与执行](../../../raw/current/scripts/components/cooker.lua#L12-L27)
- 优先使用：正常交互沿用 COOK 动作；修改炉具结果用 `oncookfn(cooker, product, chef)`，保留已有回调。[完成回调](../../../raw/current/scripts/components/cooker.lua#L43-L48)
- 重要副作用：CookItem 会删除交给它的原料实体，即使产物未生成；拆堆和发放不在本组件内。[删除](../../../raw/current/scripts/components/cooker.lua#L47)／[动作发放](../../../raw/current/scripts/actions.lua#L2033-L2035)
- 修改边界：服务端；原版火堆共同声明 cooker 标签，服务端装组件并配置燃料。CanCook 不检查所有权／持有状态。[原版声明](../../../raw/current/scripts/prefabs/firepit.lua#L164-L165)／[组件](../../../raw/current/scripts/prefabs/firepit.lua#L211)／[门禁范围](../../../raw/current/scripts/components/cooker.lua#L12-L22)

## 关键接口

| 接口／字段 | 签名或约定 | 条件、作用与证据 |
|---|---|---|
| `CanCook(item, chef)` | 资格判断 | 要求 cookable；拒绝燃料空、正在燃烧的食材、已投出的 projectile；dangerouscooker 还检查 chef 的 expertchef 标签。[条件](../../../raw/current/scripts/components/cooker.lua#L12-L18) |
| `CookItem(item, chef)` | 返回生成实体或 nil | 先门禁、调用食材 Cook，统计／回调／声音后删除原料；自身不放回人物物品栏。[执行](../../../raw/current/scripts/components/cooker.lua#L25-L48) |
| `oncookitem` | `fn(raw_item, product)` | 原料尚未删除；product 可 nil；这里没有 cooker 或 chef 参数。[调用](../../../raw/current/scripts/components/cooker.lua#L30-L31) |
| `oncookfn` | `fn(cooker_inst, product, chef)` | 在食材 Cook 完成后，原料删除前调用；product 可 nil。[调用](../../../raw/current/scripts/components/cooker.lua#L43-L48) |

## 典型接入

### 石头营火烤出的食物恢复全新鲜度

目标模组 `modmain.lua`，复用原版 firepit 和食材，不增加资源。包裹结果回调，在 cookable 已完成默认新鲜度换算后修改产物；先执行旧回调，再检查结果仍有效。[默认换算](../../../raw/current/scripts/components/cookable.lua#L29-L35)／[SetPercent](../../../raw/current/scripts/components/perishable.lua#L223)／[PostInit](../../../raw/current/scripts/modutil.lua#L593)

```lua
AddPrefabPostInit("firepit", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local cooker = inst.components.cooker
    if cooker == nil then return end

    local old = cooker.oncookfn
    cooker.oncookfn = function(cooker_inst, product, chef)
        if old ~= nil then old(cooker_inst, product, chef) end
        if product ~= nil and product:IsValid()
            and product.components.perishable ~= nil then
            product.components.perishable:SetPercent(1)
        end
    end
end)
```

只作用 firepit 的直接烤制结果，不改其他炉具、配方或物品基础保质期。原版回调不使用返回值；本例保留其调用，不阻止后续模组再次覆盖。停用后重新创建／读档的营火恢复原配置，已烤出的食物不回退；对半鲜食材、无 perishable 产物、无效产物及存档恢复做运行验证。

## 常见陷阱与限制

- **CookItem 接收整堆：** 方法直接 `item:Remove()`，不扣单件；走正常动作的拆分／失败放回与成功发放流程，详见 [[reference/components/stackable]]。[动作转移](../../../raw/current/scripts/actions.lua#L2016-L2035)／[手持炉具路径拆堆](../../../raw/current/scripts/actions.lua#L2067-L2068)
- **CanCook 已经验证角色与距离：** 源码明确不在此检查 held／canbepickedup；dangerouscooker 分支还会直接使用 chef。自定义服务端入口须另行核验执行者、距离和物品归属。[门禁](../../../raw/current/scripts/components/cooker.lua#L12-L22)／[原版距离与所有权](../../../raw/current/scripts/actions.lua#L2061-L2068)
- **产物 nil 就原料无损：** 通过 CanCook 后仍会删除原料；先保证 cookable.product 是有效已加载 Prefab，不用返回 nil 当失败门禁。[执行尾部](../../../raw/current/scripts/components/cooker.lua#L43-L48)
- **cooker 就是 stewer：** cooker 没有材料组合计算、料理等待与锅内腐败；后者另见 [[reference/components/stewer#关键接口]]。

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：sources 变更文件 actions.lua／components/perishable.lua 对照更新报告 R-01 与 hunk 地图逐 hunk 核验。锚点重定位 5 处（COOK 两路径 4 处 +3 行移，perishable SetPercent 移至 :223）；COOK 动作与 SetPercent 无语义变化，perishable 本批改动（酸雨 Update 门控、AddTime 重写）不触及本页引用。未覆盖：运行验证。

- 源码审阅：2026-10-17，Steam buildid 25643519；核验门禁、两种回调参数／顺序、删除／动作分工、firepit 接入及结果保鲜示例。未审校全部炉具和角色专属行为。
- 结构／示例检查：本批格式、来源文件、链接／行锚点、导航登记及完整 Lua 5.1 示例语法检查通过。
- 游戏运行：未运行；堆叠取用、回调叠加、联网和新鲜度显示需验证。
