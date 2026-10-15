---
title: "fertilizer"
created: "2026-10-17"
updated: "2026-10-28"
type: component
summary: "配置肥料的三类养分与使用消耗，区分现代农田部署、植物复壮和沃姆伍德自肥。"
aliases: ["肥料养分", "便便施肥加倍", "肥料消耗", "fertilizer"]
tags: [component, item, callback, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/fertilizer.lua", "prefabs/poop.lua", "prefabs/fertilizer.lua", "standardcomponents.lua", "components/farming_manager.lua", "components/pickable.lua", "components/crop.lua", "components/grower.lua", "actions.lua", "tuning.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# fertilizer

## 快速判断

- 职责与适用：挂在肥料物品上，提供养分／旧路径参数和成功使用后的消耗；目标如何获益由消费者决定。自肥接收端见 [[reference/components/fertilizable]]。[构造与消耗](../../../raw/current/scripts/components/fertilizer.lua#L2-L48)
- 优先使用：现代农田改 `SetNutrients(n1,n2,n3)`；新肥料沿用 pristine／服务端两段部署 helper，并配物品工厂，不能只加组件。[部署接入](../../../raw/current/scripts/standardcomponents.lua#L1797-L1831)
- 重要副作用：`OnApplied` 先扣 finiteuses，再调用 onappliedfn，最后按是否末次移除一个堆叠或实体；不要额外再扣一次。[消耗顺序](../../../raw/current/scripts/components/fertilizer.lua#L30-L48)
- 修改边界：本文修改服务端参数；`fertilizervalue` 不是现代土壤三养分倍率，组件不会自动播种、浇水或降低全部压力。[旧作物](../../../raw/current/scripts/components/crop.lua#L96) [土壤消费](../../../raw/current/scripts/standardcomponents.lua#L1797-L1802)

## 关键接口

| 接口／字段 | 参数或返回 | 条件、作用与证据 |
|---|---|---|
| `SetNutrients(n1,n2,n3)`／`SetNutrients(table)` | 三个数字／三项数组 | 默认 `{0,0,0}`；table 形式直接保存引用，不复制。顺序为催长素／堆肥／粪肥。[实现](../../../raw/current/scripts/components/fertilizer.lua#L22-L28) [索引常量](../../../raw/current/scripts/tuning.lua#L5854-L5856) |
| `fertilizervalue`／`soil_cycles`／`withered_cycles` | 默认各为 1 | 分别供旧 crop 成长、旧 grower 肥力轮次、pickable 抗枯萎周期消费；不等于通用增产倍率。[默认](../../../raw/current/scripts/components/fertilizer.lua#L4-L7) [crop](../../../raw/current/scripts/components/crop.lua#L96) [grower](../../../raw/current/scripts/components/grower.lua#L77) [pickable](../../../raw/current/scripts/components/pickable.lua#L259-L292) |
| `OnApplied(doer,target?)` | 无显式返回 | 有 finiteuses 则 `Use()` 默认扣 1，剩余 ≤0 为末次；没有则一次用尽。末次堆叠 `Get():Remove()`。[实现](../../../raw/current/scripts/components/fertilizer.lua#L30-L48) |
| `onappliedfn` | `(item,final_use,doer,target)` | 位于扣耐久之后、组件自身移除分支之前；finiteuses 的耗尽回调可能已先运行。地面部署传入 target=nil。[回调位置](../../../raw/current/scripts/components/fertilizer.lua#L32-L45) [部署调用](../../../raw/current/scripts/standardcomponents.lua#L1802) |
| `fertilize_sound` | 默认音效字符串，可 nil | 由具体消费者播放；单独调用 OnApplied 不播。[默认](../../../raw/current/scripts/components/fertilizer.lua#L7) [部署](../../../raw/current/scripts/standardcomponents.lua#L1803-L1805) |
| `SetHealingAmount`／`Heal` | 已废弃 | 前者空实现，后者固定 false；人物受肥效果交由接收方。[实现](../../../raw/current/scripts/components/fertilizer.lua#L19-L20) [Heal](../../../raw/current/scripts/components/fertilizer.lua#L50-L52) |

现代农田路径是 `deployable → fertilizer_ondeploy → farming_manager:AddTileNutrients → OnApplied`，按**地块坐标**更新三养分并各限制到 0～100；`ACTIONS.FERTILIZE` 则处理 crop／grower／pickable／quagmire 目标或执行者自肥，成功后统一 OnApplied。[部署](../../../raw/current/scripts/standardcomponents.lua#L1797-L1814) [养分更新](../../../raw/current/scripts/components/farming_manager.lua#L394-L398) [动作分派](../../../raw/current/scripts/actions.lua#L1812-L1842)

## 典型接入

### 便便三类养分加倍，保留原版部署与一份消耗

放目标模组 `modmain.lua`，无新增资源。原便便已注册部署、堆叠和肥料；table 形式可能引用共享配置，因此构造新数组。只改实际存在的数值，零养分仍为零。[原便便](../../../raw/current/scripts/prefabs/poop.lua#L89-L99) [部署保留](../../../raw/current/scripts/prefabs/poop.lua#L120) [PostInit](../../../raw/current/scripts/modutil.lua#L593-L599)

```lua
AddPrefabPostInit("poop", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local fertilizer = inst.components.fertilizer
    if fertilizer == nil then return end
    local nutrients = fertilizer.nutrients
    fertilizer:SetNutrients(
        nutrients[1] * 2,
        nutrients[2] * 2,
        nutrients[3] * 2
    )
end)
```

影响所有本例初始化的 poop（含加载重建）的养分消费者，包括 [[reference/components/fertilizable#关键接口|沃姆伍德自肥]]，不是只影响土壤；不改旧参数、堆叠或耐久。停用后重新初始化恢复原配置，已经加入土壤的养分不回滚。地块上限会使实际增量小于两倍。

原版模式：poop 单份堆叠消耗；`fertilizer` 肥料桶用 finiteuses 多次使用。更改次数查 [[reference/components/finiteuses]]，不要同时注册 FERTILIZE 动作耐久扣费与 OnApplied 扣费。[肥料桶](../../../raw/current/scripts/prefabs/fertilizer.lua#L55-L68)

## 常见陷阱与限制

- **改 nutrients 数组共享项**：table 赋值不复制，可能污染其他实例／数据消费者；按示例传三个数或先复制。新增物品还需在 pristine 阶段显式准备肥料标签及部署可见性。[标签与引用](../../../raw/current/scripts/components/fertilizer.lua#L9-L28)
- **手动调用目标 Fertilize 后再走原动作**：原动作已有成功后消耗；重复 OnApplied 会多扣／多删。直接调用组件时由调用者明确负责一次结算。[动作](../../../raw/current/scripts/actions.lua#L1836-L1840)
- **用废弃 Heal 做回血肥料**：其固定 false；先选人物接收回调或实际治疗机制，不凭方法名推断。[废弃实现](../../../raw/current/scripts/components/fertilizer.lua#L50-L52)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 的变更文件（actions.lua、tuning.lua）对照更新报告 R-01/R-04 与 hunk 地图逐条核验，FERTILIZE 分派与三养分索引常量无语义变化，锚点重定位 3 处、结论修订 0 处；未覆盖：运行验证。

- 源码审阅：2026-10-17，Steam buildid 25643519；覆盖参数／引用、消耗顺序、部署到地块养分、FERTILIZE 分派及两种原版物品。旧 crop／grower 仅核验所列字段消费，未审校完整旧农场。
- 结构检查：页面格式、来源文件、链接／行号、大小与叶级登记通过；完整示例经已有 Full Moon Lua 5.1 syntax-only 检查，未验证类型或 API 绑定。
- 游戏运行：未运行；养分叠加上限、耗尽回调、多人部署／存档及人物自肥收益仍待实测，植物研究图鉴与其他肥料变体未覆盖。
