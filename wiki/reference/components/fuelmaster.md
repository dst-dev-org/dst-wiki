---
title: "fuelmaster"
created: "2026-10-21"
updated: "2026-10-28"
type: component
summary: "配置添燃料者提供的补充量倍率，按燃料与目标计算，并区分补充收益和持续消耗率。"
aliases: ["添燃料加成", "人物燃料效率", "薇洛添火倍率", "补充燃料倍率", "fuelmaster"]
tags: [component, energy, callback, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/fuelmaster.lua", "components/fueled.lua", "prefabs/willow.lua", "prefabs/campfire.lua", "actions.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# fuelmaster

## 快速判断

- 职责与适用：挂在**添燃料者 doer** 身上；接收端 `fueled:TakeFuelItem(item, doer)` 查询它，为本次补充量提供倍率。放在燃料物品或接收目标上不会因其位置自动生效。[fueled.lua:197](../../../raw/current/scripts/components/fueled.lua#L197
- 优先使用：全局基础倍率用 `SetBonusMult`；按燃料／接收目标区分用 `SetBonusFn`，可参照原版薇洛。[fuelmaster.lua:7](../../../raw/current/scripts/components/fuelmaster.lua#L7)、[willow.lua:62](../../../raw/current/scripts/prefabs/willow.lua#L62
- 重要副作用：改变补充值，不改变 `fueled.rate`、燃料资格、库存拆分和耗尽逻辑；最终量仍被接收端上限截断。倍率为零也不表示取消动作或返还物品。[fueled.lua:200](../../../raw/current/scripts/components/fueled.lua#L200)、[:305](../../../raw/current/scripts/components/fueled.lua#L305)
- 修改边界：服务端结算；必须把真实 doer 传给 TakeFuelItem。直接 `DoDelta`、初始化燃料或缺少 doer 的调用不经过该人物加成。原版 ADDFUEL 会传动作执行者。[actions.lua:2188](../../../raw/current/scripts/actions.lua#L2188

## 关键接口

| 接口／字段 | 参数与返回约定 | 条件、消费者与副作用 |
|---|---|---|
| `SetBonusMult(mult)` | 数值；默认 `bonusmult = 1` | 直接替换基础倍率，没有范围约束、来源键或自动叠加。[fuelmaster.lua:1](../../../raw/current/scripts/components/fuelmaster.lua#L1 |
| `SetBonusFn(fn)` | `fn(doer, fuel_obj, target)`；返回数值倍率 | 单回调替换，`SetBonusFn(nil)` 清除；应保留需要的原回调。回调不应自行删物品或再添燃料。[fuelmaster.lua:11](../../../raw/current/scripts/components/fuelmaster.lua#L11 |
| `GetBonusMult(item, target)` | 返回 `(回调结果或 1) × bonusmult` | 无回调、回调返回 nil／false 时使用 1；返回 0 会保留为 0，返回非数值可能算术报错。该方法本身不改燃料量。[fuelmaster.lua:15](../../../raw/current/scripts/components/fuelmaster.lua#L15 |

### 实参、补充值与消耗顺序

原版 `TakeFuelItem` 中，`fuel_obj = item or doer`。普通添料时它是从库存拆出的燃料；fueler 自身供能路径调用 `TakeFuelItem(nil, doer)`，此时第二个回调参数就是 doer 自己。第三参始终是拥有 fueled 的接收实体，并非火焰特效。[fueled.lua:197](../../../raw/current/scripts/components/fueled.lua#L197)、[actions.lua:2207](../../../raw/current/scripts/actions.lua#L2207

成功通过 fueled 的 accepting、fueltype／secondaryfueltype 和自定义接收门禁后，补充值按下式计算：

```text
fuel.fuelvalue × 接收端fueled.bonusmult × 接收端multfn
              × 湿燃料倍率 × doer的fuelmaster:GetBonusMult(fuel_obj, target)
```

接收端无 multfn 时取 1，燃料不湿时湿倍率取 1，没有 doer／fuelmaster 时人物倍率取 1。量的单位与目标 fueled 的燃料单位一致；持续时长还取决于 rate 等消耗条件，不能直接把倍率说成固定秒数。[fueled.lua:205](../../../raw/current/scripts/components/fueled.lua#L205)；时间消耗见 [[reference/components/fueled#关键接口]]。

补充值随后交给 `DoDelta`，通知中的 fuelvalue 是公式值，不保证等于上限夹取后的净增加量。成功路径仍取用／删除传入燃料，不能用人物倍率取消消费；通用取用回调与删除、库存拆分／失败返还见 [[reference/components/fuel#关键接口]]、[[reference/components/fueled#常见陷阱与限制]]。[fueled.lua:212](../../../raw/current/scripts/components/fueled.lua#L212

### 生命周期与恢复

组件只保存 `inst`、`bonusmult` 和 `bonusfn` 三个运行时字段；全文没有更新任务、事件、`OnSave`／`OnLoad`／移除钩子。宿主构造需重新安装配置；运行时按技能、装备或状态改变倍率的模组要自行重建和撤销，不能期待组件保存闭包或旧倍率。薇洛在普通模式 master_postinit 添加组件并设置函数，熔炉／暴食走各自分支。[fuelmaster.lua:1](../../../raw/current/scripts/components/fuelmaster.lua#L1)、[willow.lua:188](../../../raw/current/scripts/prefabs/willow.lua#L188

## 典型接入

### 将薇洛每次添燃料的补充量再乘二

完整模组 `modmain.lua` 入口，普通生存模式；复用原版人物、燃料与接收目标，无新增资源。修改真实已有 fuelmaster 的薇洛，保留其按目标计算的原回调；注册与服务端门禁同 [[reference/components/eater#典型接入]]。

```lua
AddPrefabPostInit("willow", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local mastery = inst.components.fuelmaster
    if mastery ~= nil then
        mastery:SetBonusMult(mastery.bonusmult * 2)
    end
end)
```

2 是无单位乘数，作用于此人物之后成功执行的每次补充。原版薇洛回调：目标有 `firefuellight` 或 `campfire` 标签，或 prefab 为 `nightlight` 时取 `TUNING.WILLOW_CAMPFIRE_FUEL_MULT = 1.5`，其余取 1；本例在原版基础上分别变为 3 和 2，**也会增强其他可接受燃料的目标**。若需求只针对某目标，应串联旧 bonusfn 并在回调中判断 target，不能用全局基础倍率假装目标限定。[willow.lua:62](../../../raw/current/scripts/prefabs/willow.lua#L62)、[tuning.lua:3773](../../../raw/current/scripts/tuning.lua#L3773

可用原版木头给尚能添料的 campfire 观察：木头是 fuel 来源，campfire 有 `campfire` 标签、fueled 和 accepting 配置，动作将薇洛作为 doer。先选择剩余空间充足的营火，避免上限遮住数值差异；不需要修改 campfire 的消耗率或火焰资源。[campfire.lua:144](../../../raw/current/scripts/prefabs/campfire.lua#L144)、[:176](../../../raw/current/scripts/prefabs/campfire.lua#L176)、[[reference/components/fuel#典型接入]]。

停用模组并重建人物实例后恢复原版倍率，已经加入目标的燃料不会回滚；临时开关需保存并恢复旧 bonusmult，移除整个组件会一并移除薇洛原能力。本例没有新增事件或任务，未运行游戏。

## 常见陷阱与限制

- **倍率为零就不消费物品**：TakeFuelItem 通过门禁后仍调用 Taken、删除非 nil 的 item 并返回 true；0 只是本次公式加量为零。拒绝添料应在接收方门禁处理，不能用本回调表达取消。[fueled.lua:200](../../../raw/current/scripts/components/fueled.lua#L200
- **倍率可以任意返回**：没有钳制或数值类型检查；负倍率可把补充变成扣量，经 DoDelta 还可能触发分段／耗尽行为。正常效率功能返回非负数值，异常值不要交给 Lua 的 `or 1` 兜底。[fuelmaster.lua:16](../../../raw/current/scripts/components/fuelmaster.lua#L16)、[fueled.lua:305](../../../raw/current/scripts/components/fueled.lua#L305
- **给所有角色添加组件就与原版逻辑兼容**：人物可能已有自定义 bonusfn／bonusmult；setter 会替换，组件也没有多来源协调。应先确定目标与修改所有权；保留旧函数只解决串联，不自动解决不同模组的撤销顺序。
- **燃料增加等于燃烧时间翻倍**：上限、湿度、目标倍率、雨天消耗和 rate 都可能改变观测；campfire 自身根据雨势更新 rate。[campfire.lua:33](../../../raw/current/scripts/prefabs/campfire.lua#L33)；持续消耗调整另见 [[reference/components/fueled]]。

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：sources 变更文件 actions.lua／tuning.lua 对照更新报告 R-01 与 hunk 地图逐 hunk 核验（willow.lua／campfire.lua 本批未变更）。锚点重定位 3 处（ADDFUEL 传 doer 移至 actions.lua:2185，fueler 路径 TakeFuelItem(nil, doer) 移至 :2204，WILLOW_CAMPFIRE_FUEL_MULT 移至 tuning.lua:3773，均 +3／+14 行移、内容与数值不变）；ADDFUEL 无语义变化。未覆盖：运行验证。

- 源码审阅：2026-10-21，Steam buildid 25643519；读组件全文及 fueled 的资格／倍率／补充和回调删除顺序，核对 ADDFUEL 两条 doer 路径、薇洛构造与目标判定、调参值、营火实际接收配置。燃料物品和库存拆分机制复用同版本 fuel／fueled 页。
- 结构／语法：本页元数据、大小与链接检查通过，源码锚点 strict 检查无错误／警告；完整 `modmain.lua` 经已有 Full Moon 解析器的 Lua 5.1 `--syntax-only` 检查通过。
- 游戏运行：未运行；未覆盖全部 fueler、目标接收逻辑、多人表现、存读及多模组覆盖实测，结构／语法通过不代表游戏效果通过。

相关：[[reference/components/fueled]]；[[reference/components/fuel]]。
