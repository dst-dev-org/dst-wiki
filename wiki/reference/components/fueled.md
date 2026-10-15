---
title: "fueled"
created: "2026-10-11"
updated: "2026-10-28"
type: "component"
summary: "配置随启用时间消耗的燃料或使用耐久，控制启动、补充与耗尽行为。"
aliases: ["燃料消耗", "装备时间耐久", "随时间消耗", "补燃料", "fueled"]
tags: ["component", "item", "equipment", "energy", "lifecycle"]
dst_build: "steam-buildid:26230496"
sources: ["actions.lua", "components/fueled.lua", "components/inventoryitem_replica.lua", "prefabs/torch.lua", "prefabs/umbrella.lua"]
reliability: "verified"
validation: "source-reviewed"
status: "active"
---

# fueled

## 快速判断

- 职责与适用：启用期间按时间消耗，也可手动 DoDelta；不要求是可燃物，雨伞用 FUELTYPE.USAGE。[components/fueled.lua:83](../../../raw/current/scripts/components/fueled.lua#L83 [prefabs/umbrella.lua:175](../../../raw/current/scripts/prefabs/umbrella.lua#L175
- 优先使用：初始化 InitializeFuelLevel，再按生命周期 StartConsuming／StopConsuming；动作次数用 [[reference/components/finiteuses]]，自动恢复用 [[reference/components/rechargeable]]。[components/fueled.lua:269](../../../raw/current/scripts/components/fueled.lua#L269
- 重要副作用：DoDelta 更新分段和 percentusedchange；跨到耗尽分段可调用 depleted，不默认删实体。[components/fueled.lua:305](../../../raw/current/scripts/components/fueled.lua#L305
- 修改边界：服务端组件；补充还需 accepting 与匹配 fuel／fueler，USAGE 并不自动允许加木头。[components/fueled.lua:169](../../../raw/current/scripts/components/fueled.lua#L169

## 关键接口

| 接口／字段 | 约定 | 作用与证据 |
|---|---|---|
| `InitializeFuelLevel(fuel)` | 初始化当前量；默认最大／当前为 0 | 仅在新量更大时提高最大量，不是通用降低上限方法，也不自动启动。[components/fueled.lua:289](../../../raw/current/scripts/components/fueled.lua#L289 |
| `rate`／`StartConsuming()`／`StopConsuming()` | 默认 rate=1、period=1 秒 | DoUpdate 按 dt × rate × rate_modifiers 扣量；停止取消任务。[components/fueled.lua:325](../../../raw/current/scripts/components/fueled.lua#L325 [components/fueled.lua:339](../../../raw/current/scripts/components/fueled.lua#L339 |
| `DoDelta(amount, doer?)`／`SetPercent(pct)` | 绝对增减／比例 | DoDelta 夹在 0～maxfuel；耗尽回调仅在分段变化且归零时触发。[components/fueled.lua:255](../../../raw/current/scripts/components/fueled.lua#L255 [components/fueled.lua:305](../../../raw/current/scripts/components/fueled.lua#L305 |
| `SetDepletedFn(fn)`／`SetSectionCallback(fn)` | `(inst)`／`(newsection, oldsection, inst, doer?)` | 单回调；InitializeFuelLevel 的分段回调不传 doer，且不走 depleted。[components/fueled.lua:145](../../../raw/current/scripts/components/fueled.lua#L145 [components/fueled.lua:289](../../../raw/current/scripts/components/fueled.lua#L289 |
| `accepting`／`fueltype`／`secondaryfueltype` | 默认 false／BURNABLE／nil | 接受匹配的燃料类型；先检查 fuel／fueler 存在，CanAcceptFuelItem 对不合格对象可能访问 nil。[components/fueled.lua:169](../../../raw/current/scripts/components/fueled.lua#L169 |

## 典型接入

### 将原版雨伞的基础时间消耗率减半

目标模组 `modmain.lua`；复用 umbrella 的 USAGE 燃料类型、耗尽和穿脱回调，无新增资源。原版装备时启动、卸下时停止。[prefabs/umbrella.lua:22](../../../raw/current/scripts/prefabs/umbrella.lua#L22 [prefabs/umbrella.lua:35](../../../raw/current/scripts/prefabs/umbrella.lua#L35 [prefabs/umbrella.lua:175](../../../raw/current/scripts/prefabs/umbrella.lua#L175

```lua
AddPrefabPostInit("umbrella", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    if inst.components.fueled ~= nil then
        inst.components.fueled.rate = 0.5
    end
end)
```

0.5 是每秒基础燃料单位，其他倍率仍参与；不补满现有量。保留原版装备／卸下与假人停止逻辑；停用后新实例恢复 rate，已经耗掉的燃料不回滚。

### 燃料与使用耐久的选择

火把：fueled + depleted 删除，光效另由装备／燃料回调控制。雨伞：USAGE + 穿戴计时，可走独立缝补机制。按动作耗次数：finiteuses。技能用后自动恢复：rechargeable。不要因界面都显示百分比就混用消费者。[prefabs/torch.lua:505](../../../raw/current/scripts/prefabs/torch.lua#L505 [prefabs/umbrella.lua:175](../../../raw/current/scripts/prefabs/umbrella.lua#L175 [components/inventoryitem_replica.lua:236](../../../raw/current/scripts/components/inventoryitem_replica.lua#L236

### 模组中的多档耗能模式

本地片段参照（未随库分发，未测游戏／外部 Hook；[版本与文件指纹](../../../docs/reports/w05/local-mod-references-20261017.md)）：Sora 的 sora2amulet.lua:84～87 按模式更新 fueled.rate 并启动消耗，:247～254 配主／副燃料类型和补充资格。该组合用于可补充、多档持续耗能，不应因为某档会回充就全部换成 rechargeable。灯光、夜视与人物修饰仍由模组效果／归属回调管理，不是 fueled 自带。

如果业务允许负 rate，DoUpdate 的负扣量可让剩余量增长；这是一种自定义能量规则，须验证上限、耗尽、停止和离开持有者时的处理。充能满／未满回调需求则优先 [[reference/components/rechargeable]]。[components/fueled.lua:325](../../../raw/current/scripts/components/fueled.lua#L325 [components/fueled.lua:305](../../../raw/current/scripts/components/fueled.lua#L305

## 常见陷阱与限制

- **直接给整堆 TakeFuelItem 只扣一件**：组件删除整个传入实体；原版 ADDFUEL 先 RemoveItem 拆一件。自定义接口需自己拆分并处理失败返还。[components/fueled.lua:216](../../../raw/current/scripts/components/fueled.lua#L216 [actions.lua:2188](../../../raw/current/scripts/actions.lua#L2188
- **InitializeFuelLevel(0) 自动触发耗尽**：该初始化不走 DoDelta 的 depleted 分支；初始化和状态变化不要混用。[components/fueled.lua:289](../../../raw/current/scripts/components/fueled.lua#L289
- **只配置量就持续消耗**：还要 StartConsuming，撤销效果时 StopConsuming；复制原版穿脱／假人边界。[prefabs/umbrella.lua:22](../../../raw/current/scripts/prefabs/umbrella.lua#L22 [prefabs/umbrella.lua:38](../../../raw/current/scripts/prefabs/umbrella.lua#L38

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：sources 变更文件 actions.lua／prefabs/torch.lua 对照更新报告 R-01/P-117 与 hunk 地图逐 hunk 核验。锚点重定位 2 处（ADDFUEL 拆一件移至 actions.lua:2185，火把 fueled 配置块移至 torch.lua:505，均内容一致）；ADDFUEL 动作与火把 fueled 配置无语义变化，torch.lua 本批新增 torch 标签（P-117）与投掷偏转不触及本页结论。未覆盖：运行验证。

- 源码依据复核：2026-10-17，Steam buildid 25643519；核对范围：初始化与 DoDelta 耗尽差异、补充时整实体删除、雨伞穿脱计时与 rate 示例，依据见上文对应锚点；未覆盖自定义负 rate 的完整上限／停止／归属流程。
- 历次结构／语法检查：元数据、源码路径／行号、链接／章节、标签、大小和导航登记检查通过；本页 Lua 代码经 Full Moon 3.0.0／Lua 5.1 语法及原字节往返检查。
- 游戏运行：未运行；案例的多人交互、存档重载和其他模组兼容仍需游戏验证。

相关：[[reference/components/fuel]]；[[reference/components/equippable]]；[[reference/components/finiteuses]]；[[reference/components/rechargeable]]。
