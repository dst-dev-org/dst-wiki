---
title: "occupier"
created: "2026-10-22"
updated: "2026-10-28"
type: component
summary: "为笼中鸟等占用者保存宿主引用并接收入驻回调，实际收纳、释放与存档交给 occupiable。"
aliases: ["笼中鸟宿主", "占用者引用", "金丝雀入笼回调", "occupier"]
tags: [component, creature, lifecycle, callback, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/occupier.lua", "components/occupiable.lua", "prefabs/birds.lua", "prefabs/birdcage.lua", "componentactions.lua", "actions.lua", "entityscript.lua", "mainfunctions.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# occupier

## 快速判断

- 职责与适用：被占用空间接收的实体一侧，仅保存 owner；没有计时、容量、寻家或住户生成逻辑。鸟类在主模拟添加，canary 由通用 makebird 工厂注册。[组件全文](../../../raw/current/scripts/components/occupier.lua#L1-L15) [添加](../../../raw/current/scripts/prefabs/birds.lua#L294-L330) [注册](../../../raw/current/scripts/prefabs/birds.lua#L436-L441)
- 优先使用：读取 `GetOwner()` 找到笼子；入笼扩展包装 `onoccupied(bird, owner)`。完整占用流程使用宿主 [[reference/components/occupiable]]，不自行 SetOwner 冒充入笼。[写入与回调](../../../raw/current/scripts/components/occupiable.lua#L59-L73)
- 重要副作用：SetOwner 自身只赋值，不修改宿主、父子关系或场景；onoccupied 也由 occupiable 显式调用，不是字段赋值自动触发。[实现](../../../raw/current/scripts/components/occupier.lua#L7-L13)
- 修改边界：鸟笼喂食开关、展示与死亡产物由鸟笼控制；occupier 不是 [[reference/components/container]] 的库存 owner，也不是 [[reference/components/homeseeker]] 的家园。[宿主回调](../../../raw/current/scripts/prefabs/birdcage.lua#L323-L370)

## 关键接口

| 接口／字段 | 参数或返回 | 条件、作用与证据 |
|---|---|---|
| `GetOwner()` | owner 或 nil | 构造未显式初始化 owner，未设置时为 nil；不检查实体有效性。[实现](../../../raw/current/scripts/components/occupier.lua#L1-L9) |
| `SetOwner(owner)` | 实体或 nil；无显式返回 | 只写 `self.owner`，不会验证独占、释放旧宿主或触发回调。[实现](../../../raw/current/scripts/components/occupier.lua#L11-L13) |
| `onoccupied` | `(occupier_inst, host_inst)` | 构造未初始化此字段；occupiable 设置 owner 后调用，先于宿主 onoccupied 和隐藏。需要保留原回调。[时序](../../../raw/current/scripts/components/occupiable.lua#L61-L73) |

### 动作、保存与离开

库存物品的 occupier 动作消费者遍历 OCCUPANTTYPE，匹配物品类型标签与目标 `<类型>_occupiable` 标签后提供 STORE，骑乘还有限制；服务端 STORE 再检查双方组件／CanOccupy，移出库存后 Occupy。仅加组件不会自动变成可拾取鸟或提供任意住户动作。[动作候选](../../../raw/current/scripts/componentactions.lua#L1516-L1533) [执行](../../../raw/current/scripts/actions.lua#L2593-L2597)

occupier 自身没有 OnSave／OnLoad／删除清理。宿主 occupiable 保存占用者完整记录，SpawnSaveRecord 先还原其持久化数据，再由宿主 Occupy 重建 owner 并再次调用 onoccupied；不要额外保存旧实体引用。[组件全文](../../../raw/current/scripts/components/occupier.lua#L1-L15) [嵌套记录](../../../raw/current/scripts/components/occupiable.lua#L120-L134) [实体恢复](../../../raw/current/scripts/mainfunctions.lua#L464-L503)

正常 Harvest 将 owner 设 nil，解除父子和占用监听、ReturnToScene，但**不调用 onoccupied(bird, nil)**。金丝雀原版用 exitlimbo 停吸气任务；perished／onremove 则由宿主监听清占用。直接移除 occupier 无对应清理，不能替代 Harvest。[释放](../../../raw/current/scripts/components/occupiable.lua#L104-L118) [金丝雀](../../../raw/current/scripts/prefabs/birds.lua#L399-L416) [事件清空位](../../../raw/current/scripts/components/occupiable.lua#L75-L98)

## 典型接入

### 金丝雀每次入笼（含读档）至少半饱，保留洞穴毒气逻辑

完整目标模组 `modmain.lua`；仅原版 canary，主模拟包装 occupier 回调，不增资源或定时任务。金丝雀原回调在 cage 宿主启动吸气，原逻辑还通过 GetOwner 向笼子发送 birdpoisoned，不能覆盖丢失。[入笼回调](../../../raw/current/scripts/prefabs/birds.lua#L167-L173) [宿主消费者](../../../raw/current/scripts/prefabs/birds.lua#L129-L138) [Hook](../../../raw/current/scripts/modutil.lua#L593-L599)

```lua
AddPrefabPostInit("canary", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local occupier = inst.components.occupier
    if occupier == nil then return end
    local previous = occupier.onoccupied
    occupier.onoccupied = function(bird, owner)
        if previous ~= nil then previous(bird, owner) end
        local perishable = bird.components.perishable
        if owner ~= nil and owner.prefab == "birdcage"
            and perishable ~= nil and perishable:GetPercent() < 0.5 then
            perishable:SetPercent(0.5)
        end
    end
end)
```

`0.5` 是剩余寿命比例，不是秒；鸟笼以 perishable 比例作为饥饿程度，不是在修改 hunger 三维属性。GetPercent／SetPercent 的约束复用 [[reference/components/perishable#关键接口]]；原版喂食会补到 1，示例只把低于一半者补至一半，继续原版饥饿进程。[鸟笼读饥饿](../../../raw/current/scripts/prefabs/birdcage.lua#L96-L98) [喂食补满](../../../raw/current/scripts/prefabs/birdcage.lua#L149-L152)

加载已有笼中金丝雀也会再执行此补给，因此频繁重载可反复补至一半；这是本例明确的生效范围，不是“只在玩家首次装鸟”接口。禁用并重载恢复原回调，已补回的寿命不自动扣回。无自定义字段需要迁移，原存档、吸气／中毒与退出场景清理仍由原版管理。

## 常见陷阱与限制

- **SetOwner 就完成入住**：它不检查目标是否已被占用，也不写目标 occupant；必须由 occupiable 完成双边关系，详见 [[reference/components/occupiable#关键接口]]。
- **把 onoccupied 当入／离双向事件**：原版金丝雀函数虽能处理非 cage 参数，Harvest 并不会这样调用；若新增长期任务，需要实际离开路径的清理，不能只写一个永远收不到的 nil 分支。[原函数](../../../raw/current/scripts/prefabs/birds.lua#L167-L173) [释放](../../../raw/current/scripts/components/occupiable.lua#L104-L118)
- **直接移组件或删除鸟笼即可放鸟**：移组件不解除双边关系；直接删除宿主会删除子体。遵循宿主 Harvest／落地路径，不能把仅 owner 引用当作完整生命周期。[移组件](../../../raw/current/scripts/entityscript.lua#L670-L684) [删除子体](../../../raw/current/scripts/entityscript.lua#L1779-L1783) [原版锤毁](../../../raw/current/scripts/prefabs/birdcage.lua#L372-L387)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：sources 中漂移文件 actions.lua、componentactions.lua、mainfunctions.lua、entityscript.lua（更新报告 R-01/R-04/R-09）——STORE 候选与执行、SpawnSaveRecord 恢复路径均不在本轮变更 hunk 内，结论无涉；锚点重定位 3 处；未覆盖：运行验证。

- 源码审阅：2026-10-22，Steam buildid 25643519；完整审阅 occupier、occupiable，核对普通鸟工厂／金丝雀注册、主模拟安装、毒气宿主消费、STORE、存档重建／回调重放、Harvest 与删除清理。perishable 比例接口复用同版本已核验页；未完整审校毒气概率与所有鸟种行为。
- 结构检查：本页格式、来源文件、源码锚点与大小通过；完整 modmain 经 Full Moon Lua 5.1 syntax-only。导航与审校表由本批次统一登记。
- 游戏运行：未运行；需验证地上／库存／入笼时比例、新建／读档补给、洞穴毒气、放鸟后的任务停止、停用重载及其他模组回调包装顺序。

相关：[[reference/components/occupiable]]；[[reference/components/perishable]]；[[reference/components/inventoryitem]]。
