---
title: "fueler"
created: "2026-10-22"
updated: "2026-10-28"
type: component
summary: "让动作执行者自身提供补充燃料量，沿原版小乌鸦添火路径区分 fueler、燃料物品与人物加成。"
aliases: ["自身供燃料", "生物添火", "小乌鸦添燃料", "不消耗物品添燃料", "fueler"]
tags: [component, energy, callback, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/fueler.lua", "components/fuel.lua", "components/fueled.lua", "actions.lua", "componentactions.lua", "prefabs/carnival_crowkid.lua", "brains/carnival_crowkidbrain.lua", "stategraphs/SGcarnival_crowkid.lua", "tuning.lua", "entityscript.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# fueler

## 快速判断

- 职责与适用：作为燃料来源的实体组件；原版 `carnival_crowkid` 在服务端添加，设基础量为 `TUNING.SMALL_FUEL`。它经 Brain 发起无库存物品的 ADDFUEL，由自己供燃料。[carnival_crowkid.lua:103-135](../../../raw/current/scripts/prefabs/carnival_crowkid.lua#L103-L135)、[carnival_crowkidbrain.lua:100-118](../../../raw/current/scripts/brains/carnival_crowkidbrain.lua#L100-L118)
- 优先使用：已有这种动作执行者需要改基础量时改 `fuelvalue`；普通库存燃料使用 [[reference/components/fuel]]，添料者效率加成使用 [[reference/components/fuelmaster]]，接收与消耗使用 [[reference/components/fueled]]。
- 重要副作用：`Taken` 先向来源实体发 `fueltaken`，再调 ontaken；来源是否删除由接收端传入的 item 是否为 nil 决定。fueler 本身不扣量、不删自己。[fueler.lua:31-36](../../../raw/current/scripts/components/fueler.lua#L31-L36)、[fueled.lua:212-228](../../../raw/current/scripts/components/fueled.lua#L212-L228)
- 修改边界：服务端实际结算；只有添加组件不会自行寻找营火或生成玩家操作，原版小乌鸦另有 Brain、SG 与动作消费链。[carnival_crowkid.lua:147-156](../../../raw/current/scripts/prefabs/carnival_crowkid.lua#L147-L156)、[SGcarnival_crowkid.lua:5-10](../../../raw/current/scripts/stategraphs/SGcarnival_crowkid.lua#L5-L10)

## 关键接口

| 接口／字段 | 参数与返回约定 | 条件、消费者与副作用 |
|---|---|---|
| `fuelvalue` | 数值；默认 1 | 供 fueled 的补充公式读取，不是来源自身的剩余能量池。受目标倍率、湿度、fuelmaster 与目标上限影响。[fueler.lua:10-19](../../../raw/current/scripts/components/fueler.lua#L10-L19)、[fueled.lua:205-212](../../../raw/current/scripts/components/fueled.lua#L205-L212) |
| `fueltype` | 默认 `FUELTYPE.BURNABLE` | 属性回调移除旧 `<type>_fuel` 标签，再添加新标签；接收端仍须 accepting 且匹配主／副 fueltype。[fueler.lua:1-19](../../../raw/current/scripts/components/fueler.lua#L1-L19)、[fueled.lua:169-175](../../../raw/current/scripts/components/fueled.lua#L169-L175) |
| `SetOnTakenFn(fn)` | `fn(source_inst, target)` | 单回调直接替换；target 是接收燃料的实体。附加逻辑需串联原函数，不能用此回调拒绝已发生的补充。[fueler.lua:27-36](../../../raw/current/scripts/components/fueler.lua#L27-L36) |
| `Taken(target)` | 无返回约定 | 在来源实体发 `fueltaken`，数据 `{taker = target}`，然后回调；直接调用只通知，不增加目标燃料。[fueler.lua:31-36](../../../raw/current/scripts/components/fueler.lua#L31-L36) |
| `OnRemoveFromEntity()` | 组件移除钩子 | 清当前燃料类型标签；组件全文没有任务、事件监听或 OnSave／OnLoad，运行时字段不会由它保存。[fueler.lua:10-38](../../../raw/current/scripts/components/fueler.lua#L10-L38)；宿主移除分发见 [entityscript.lua:670-680](../../../raw/current/scripts/entityscript.lua#L670-L680) |

### 实际动作消费与组件分工

`ACTIONS.ADDFUEL.fn` 先处理 `inventory + invobject` 的库存路径；只有未进入该分支且 doer 有 fueler，才调用 `target.components.fueled:TakeFuelItem(nil, doer)`。ADDWETFUEL 复用同一动作函数；直接组件调用仍须给有效来源与目标。[actions.lua:2188-2213](../../../raw/current/scripts/actions.lua#L2188-L2213)

fueled 使用 `fuel_obj = item or doer`，从来源优先选 fuel、其次 fueler；检查目标 accepting／类型和自定义接收函数后计算补充量，再 `DoDelta` → 来源 `Taken` → 非 nil item 的取用回调与删除 → 通用取用回调／目标 `takefuel`。在自身供能路径，item 为 nil，接收端不删除 doer，也跳过只针对 item 的回调；自定义接收函数会收到 `(target, nil, doer)`，必须能处理 nil。[fueled.lua:197-228](../../../raw/current/scripts/components/fueled.lua#L197-L228)

fuel 与 fueler 在本快照中的字段、类型标签和 Taken 实现相同，差异落实在挂载位置与动作入口；两者同时存在时，fueled 的 `fuel or fueler` 先取 fuel。fuelmaster 属于 doer 的倍率贡献者，不提供基础燃料值；完整补充公式复用 [[reference/components/fuelmaster#实参补充值与消耗顺序]]。[fuel.lua:10-38](../../../raw/current/scripts/components/fuel.lua#L10-L38)、[fueled.lua:205-210](../../../raw/current/scripts/components/fueled.lua#L205-L210)

## 典型接入

### 将鸦年华小乌鸦单次添火的基础量翻倍

完整模组 `modmain.lua` 入口，复用原版 `carnival_crowkid` 的资源、Brain、SG 与 fueler；只改数值，保留回调及行为优先级，不生成额外生物或启动事件。其注册名与工厂绑定见 [carnival_crowkid.lua:147-156](../../../raw/current/scripts/prefabs/carnival_crowkid.lua#L147-L156)，模组 postinit／端门禁复用 [[reference/components/fuel#典型接入]]。

```lua
AddPrefabPostInit("carnival_crowkid", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local source = inst.components.fueler
    if source ~= nil then
        source.fuelvalue = source.fuelvalue * 2
    end
end)
```

2 为无单位倍率，基础量从原版 SMALL_FUEL 翻倍；这是目标燃料单位，不能直接承诺燃烧几秒。最终补充仍受目标与湿度等倍率、上限影响。目标接收和消耗配置可参照 [[reference/components/fuelmaster#典型接入]] 的营火依据。

原版行为入口：夜间寻找 12 单位范围内带 campfire、fire 标签的对象；已有观察目标须仍有效且在燃烧。ActivateDecor 在非小游戏观众且冷却允许时，优先选择观察中燃料比例不高于 .15 的 fueled 目标，并构造无 invobject 的 ADDFUEL。SG 将动作映射到 activate，并在 15 帧执行 BufferedAction；不会因为改大 fuelvalue 就每帧补充。[carnival_crowkidbrain.lua:86-146](../../../raw/current/scripts/brains/carnival_crowkidbrain.lua#L86-L146)、[tuning.lua:5986](../../../raw/current/scripts/tuning.lua#L5986)、[SGcarnival_crowkid.lua:487-501](../../../raw/current/scripts/stategraphs/SGcarnival_crowkid.lua#L487-L501)

观察效果需原版小乌鸦实际留场，避免非鸦年华时段或缺家导致飞走；原版构造据事件设置 ShouldFlyAway，Brain 还检查无家和离场条件。本例不调整这些条件。[carnival_crowkid.lua:109](../../../raw/current/scripts/prefabs/carnival_crowkid.lua#L109)、[carnival_crowkidbrain.lua:39-60](../../../raw/current/scripts/brains/carnival_crowkidbrain.lua#L39-L60)

停用模组并重建小乌鸦实例后恢复原版基础量，已加入营火的燃料不会回滚。组件不存 fuelvalue／ontaken，原版工厂每次重设；临时开关须保存并恢复旧值，不要删除组件而破坏原本添火能力。示例没有新增监听或任务。

## 常见陷阱与限制

- **给玩家加 fueler 就出现右键添火**：已核验的物品动作收集是 USEITEM 下的 `fuel` 分支；原版小乌鸦自行构造 BufferedAction。本页不承诺 fueler 自动提供玩家 UI，需要另行设计并核验动作收集与 SG。[componentactions.lua:1301-1320](../../../raw/current/scripts/componentactions.lua#L1301-L1320)
- **带 fueler 的来源永不删除**：仅 `TakeFuelItem(nil, doer)` 路径不删来源；若把实体作为非 nil item 传入，成功后仍走 item:Remove。纯通知 Taken 也不会自动扣资源或设冷却。[fueled.lua:214-220](../../../raw/current/scripts/components/fueled.lua#L214-L220)
- **把任意实体交给 CanAcceptFuelItem 探测**：目标 accepting 时直接访问 `fuel.fueltype`；应先确认来源有 fuel 或 fueler。拒绝补充应在目标接收门禁处理，不能等 Taken 回调才取消。[fueled.lua:169-175](../../../raw/current/scripts/components/fueled.lua#L169-L175)、[:197-214](../../../raw/current/scripts/components/fueled.lua#L197-L214)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：sources 变更文件 actions.lua／componentactions.lua／tuning.lua／carnival_crowkid.lua／entityscript.lua 对照更新报告 R-01/R-04/R-08 与 hunk 地图逐 hunk 核验。锚点重定位 7 处：ADDFUEL fn 移至 actions.lua:2185-2210，tuning CROWKID_STOKE_FIRE_PRECENT 移至 :5986，crowkid 三处随 eater SetDiet 行合并顺移（103-135／147-156／109）；USEITEM fuel 候选（componentactions.lua:1301-1320）与 entityscript 移除分发（:648-658）行号不变。ADDFUEL 两路径与 fueler 消费链无语义变化。未覆盖：运行验证。

- 源码审阅：2026-10-22，Steam buildid 25643519；读 fueler／fuel 全文，核对 fueled 接收与删除分支、ADDFUEL 两路径、fuel 物品动作收集、小乌鸦服务端构造／注册、Brain 观察与冷却及 SG 执行动作。补充公式、营火配置和模组入口复用同 build fuelmaster／fuel 页。
- 结构／语法：本页定向页面校验与源码链接 strict 校验通过；完整 modmain 示例通过已有 Full Moon 解析器 Lua 5.1 `--syntax-only` 检查。
- 游戏运行：未运行；鸦年华留场、夜间补充、湿来源、存读与多人表现、多模组覆盖均待实测，未核验自定义玩家 UI 接入。

相关：[[reference/components/fuel]]；[[reference/components/fueled]]；[[reference/components/fuelmaster]]。
