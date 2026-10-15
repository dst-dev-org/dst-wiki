---
title: "forgerepairable"
created: "2026-10-26"
updated: "2026-10-28"
type: component
summary: "目标侧维护锻造材料与可修标签，经修理包补满耐久，并以标准破损／修复回调恢复装备功能。"
aliases: ["锻造装备可修状态", "破损装备修复", "修理包目标", "forgerepairable"]
tags: [component, equipment, lifecycle, callback]
dst_build: "steam-buildid:26230496"
sources: ["components/forgerepairable.lua", "components/forgerepair.lua", "components/armor.lua", "components/finiteuses.lua", "components/fueled.lua", "standardcomponents.lua", "prefabs/armor_lunarplant.lua", "prefabs/pickaxe_lunarplant.lua", "prefabs/voidcloth_umbrella.lua", "actions.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# forgerepairable

## 快速判断

- 职责与适用：挂在被修目标，管理 repairmaterial／repairable 与候选标签，确认门禁后委托 [[reference/components/forgerepair]] 修理并消费材料。原版月亮护甲、亮茄镐与虚空伞都有现用接线。[forgerepairable.lua:58](../../../raw/current/scripts/components/forgerepairable.lua#L58 [armor_lunarplant.lua:228](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L228 [pickaxe_lunarplant.lua:215](../../../raw/current/scripts/prefabs/pickaxe_lunarplant.lua#L215 [voidcloth_umbrella.lua:419](../../../raw/current/scripts/prefabs/voidcloth_umbrella.lua#L419
- 优先使用：原版式可破损装备用 MakeForgeRepairable 配套保留实体、卸装、耗尽及恢复回调；不能只加目标标签就保证破损后仍有物品可修。[standardcomponents.lua:1851](../../../raw/current/scripts/standardcomponents.lua#L1851
- 重要副作用：Repair 依次查材料相等、repairable 标志与材料实际修理结果；标志和标签不独立等于真实损坏状态，需 armor／finiteuses／fueled 的刷新接线。[forgerepairable.lua:58](../../../raw/current/scripts/components/forgerepairable.lua#L58
- 修改边界：服务端目标组件，候选动作读取其标签。REPAIR 优先普通 repairable，锻造目标分支在后；两者并存时不能假定锻造修理包会进入这里。[actions.lua:984](../../../raw/current/scripts/actions.lua#L984

## 关键接口

| 接口／字段 | 参数或返回 | 条件、作用与证据 |
|---|---|---|
| `SetRepairMaterial(material)` | 材料值 | 仅在 repairable 真值时撤旧／加新 `forgerepairable_<material>` 标签；不据材料推断损坏或补充 repairable 状态。[forgerepairable.lua:1](../../../raw/current/scripts/components/forgerepairable.lua#L1 [forgerepairable.lua:46](../../../raw/current/scripts/components/forgerepairable.lua#L46 |
| `SetRepairable(repairable)` | 真／假值 | 有材料时同步可修标签；没有材料只保存标志。它不读当前耐久、不补耐久，也不处理装备功能。[forgerepairable.lua:12](../../../raw/current/scripts/components/forgerepairable.lua#L12 [forgerepairable.lua:50](../../../raw/current/scripts/components/forgerepairable.lua#L50 |
| `SetOnRepaired(fn)` | `fn(target, doer, repair_item)` | 材料 OnRepair 成功、消费与材料回调都完成后才调；返回值不消费，repair_item 可能已被移除。[forgerepairable.lua:54](../../../raw/current/scripts/components/forgerepairable.lua#L54 [forgerepair.lua:47](../../../raw/current/scripts/components/forgerepair.lua#L47 |
| `Repair(doer, repair_item)` | 布尔结果 | 要求材料有 forgerepair、两边 repairmaterial 相等、repairable 真且材料 OnRepair 成功；任一不符返回 false。自身不重算损坏、查枚举、检查漂浮／骑乘或设置固定修复量。[forgerepairable.lua:58](../../../raw/current/scripts/components/forgerepairable.lua#L58 |

## 关键流程

### 初始化与耐久状态刷新

构造只在已有 armor 时用 armor:IsDamaged 初始化 repairable，否则有 fueled 时按 GetPercent()<1 初始化；**没有 finiteuses 初始化分支**。注释中的 repairmaterial、repairable、onrepaired 不算实际赋默认值。finiteuses-only 目标应在安装本组件后通过正常次数 setter 刷新，或按已知当前状态显式 SetRepairable。[forgerepairable.lua:22](../../../raw/current/scripts/components/forgerepairable.lua#L22

| 耐久来源 | 实际刷新路径与条件 |
|---|---|
| armor | 构造监听 percentusedchange；当 armor 仍存在且载荷有 percent，若有 forgerepairable 就设 percent<1。SetPercent→SetCondition 正常发该事件；indestructible 时 SetCondition 直接返回。[armor.lua:3](../../../raw/current/scripts/components/armor.lua#L3 [armor.lua:89](../../../raw/current/scripts/components/armor.lua#L89 |
| finiteuses | current／total 属性变化调用 onfiniteuses；先刷新普通 repairable，只有它不存在才更新 forgerepairable。SetUses／SetMaxUses／SetPercent 都经相应字段赋值；安装前已经发生的损耗不会由目标构造补读。[finiteuses.lua:1](../../../raw/current/scripts/components/finiteuses.lua#L1 [finiteuses.lua:60](../../../raw/current/scripts/components/finiteuses.lua#L60 [finiteuses.lua:120](../../../raw/current/scripts/components/finiteuses.lua#L120 |
| fueled | currentfuel 变化调用 oncurrentfuel，再调 onmaxfuel；maxfuel 也有属性回调。如果任一燃料类型是 USAGE 且 no_sewing 不为真，分支只维护 needssewing；否则才刷新 forgerepairable，不能假定所有 fueled 都自动走锻造门禁。[fueled.lua:62](../../../raw/current/scripts/components/fueled.lua#L62 [fueled.lua:112-113](../../../raw/current/scripts/components/fueled.lua#L112-L113) |

Repair 自己不把 repairable 设为 false；一般由材料 SetPercent(1) 触发上述路径清掉标签。多耐久组件并存、普通 repairable 抢先或特殊 setter 拒绝修改时，标志可能与所选修复分支不同步。材料 OnRepair 又按 armor→finiteuses→fueled 的存在优先级选择，较高分支已满时不试较低分支。[forgerepairable.lua:58](../../../raw/current/scripts/components/forgerepairable.lua#L58 [forgerepair.lua:28](../../../raw/current/scripts/components/forgerepair.lua#L28

### 标准组装的破损保留与恢复

MakeForgeRepairable(inst,material,onbroken,onrepaired) 依 armor→finiteuses→fueled 优先选择耗尽处理：armor 还设 KeepOnFinished(true)，再给选定组件装 `_onbroken`。该回调先卸下已装备目标并尝试放回主人库存，随后调用传入 onbroken；最后 helper 添加目标组件、材料及修复回调。[standardcomponents.lua:1851](../../../raw/current/scripts/standardcomponents.lua#L1851

helper 不统一添加 broken 标签、禁用工具／武器或修复动画，这些由宿主 onbroken／onrepaired 做。它会设置现有耗尽回调，DEBUG_MODE 下对已有回调有断言；接入时必须核对原目标回调，不能当作无副作用的追加监听，也不应对已安装目标反复调用 helper。[standardcomponents.lua:1867](../../../raw/current/scripts/standardcomponents.lua#L1867

本组件没有 OnSave／OnLoad，材料／回调由 Prefab 重新设置；耐久本身由原组件保存恢复。armor:OnLoad 调 SetCondition、finiteuses:OnLoad 调 SetUses、fueled:OnLoad 调 InitializeFuelLevel，分别经过其事件／字段刷新路径，仍受上表门禁影响。宿主装备功能和外观恢复须另核对，不能只看可修标签。[armor.lua:116](../../../raw/current/scripts/components/armor.lua#L116 [finiteuses.lua:54](../../../raw/current/scripts/components/finiteuses.lua#L54 [fueled.lua:139](../../../raw/current/scripts/components/fueled.lua#L139 [fueled.lua:289](../../../raw/current/scripts/components/fueled.lua#L289

OnRemoveFromEntity 在 repairable 且有材料时移除对应目标标签；不恢复装备功能、不处理已存在耐久／耗尽回调，也没有生成／更新任务需要由本组件停止。[forgerepairable.lua:40](../../../raw/current/scripts/components/forgerepairable.lua#L40

## 典型接入

### 三类现用装备的原版参照

- **护甲：armor_lunarplant** 经 fn→commonfn 在主模拟先装 armor，再 MakeForgeRepairable。OnBroken 移除 equippable、标记破损；OnRepaired 重建 equippable、恢复动画／浮水显示并清破损标记。保留零耐久实体由 helper 的 armor KeepOnFinished 支撑。[armor_lunarplant.lua:245](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L245 [armor_lunarplant.lua:199](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L199 [armor_lunarplant.lua:134](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L134 [armor_lunarplant.lua:436](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L436
- **次数：pickaxe_lunarplant** 在服务端先设 finiteuses 满值／动作消耗，再装 helper；新满值实例最初没有可修标签即可。首次损耗由 current setter 激活门禁。破损回调移除 equippable／tool／weapon，修复回调通过 SetupComponents 重建，有限次数本身仍留在目标上。[pickaxe_lunarplant.lua:186](../../../raw/current/scripts/prefabs/pickaxe_lunarplant.lua#L186 [pickaxe_lunarplant.lua:88](../../../raw/current/scripts/prefabs/pickaxe_lunarplant.lua#L88 [pickaxe_lunarplant.lua:136](../../../raw/current/scripts/prefabs/pickaxe_lunarplant.lua#L136 [pickaxe_lunarplant.lua:221](../../../raw/current/scripts/prefabs/pickaxe_lunarplant.lua#L221
- **燃料：voidcloth_umbrella** 的 fueled 使用 MAGIC 类型，安装 helper 后又显式用 OnPerish 覆盖 depleted，宿主自行处理破损；OnRepaired 重建装备功能。它展示“复用目标门禁、保留专用耗尽逻辑”的模式，不能照抄成所有 USAGE 衣物也会自动刷新。[voidcloth_umbrella.lua:404](../../../raw/current/scripts/prefabs/voidcloth_umbrella.lua#L404 [voidcloth_umbrella.lua:419](../../../raw/current/scripts/prefabs/voidcloth_umbrella.lua#L419 [voidcloth_umbrella.lua:144](../../../raw/current/scripts/prefabs/voidcloth_umbrella.lua#L144 [voidcloth_umbrella.lua:526](../../../raw/current/scripts/prefabs/voidcloth_umbrella.lua#L526

## 常见陷阱与限制

- **材料名相同就能修**：repairable 真值和实际选定分支仍是独立门禁；SetRepairMaterial 不检测耐久，直接 SetRepairable(true) 也不能保证材料 OnRepair 会成功。[forgerepairable.lua:46](../../../raw/current/scripts/components/forgerepairable.lua#L46 [forgerepairable.lua:58](../../../raw/current/scripts/components/forgerepairable.lua#L58
- **先损耗 finiteuses、后加组件就自动出现动作**：构造没读次数比例，需安装后刷新或显式初始化；普通 repairable 并存时还会优先截获次数刷新和 REPAIR 动作。[forgerepairable.lua:22](../../../raw/current/scripts/components/forgerepairable.lua#L22 [finiteuses.lua:1](../../../raw/current/scripts/components/finiteuses.lua#L1 [actions.lua:984](../../../raw/current/scripts/actions.lua#L984
- **修复百分比也自动恢复所有功能**：SetPercent 只管所选耐久；破损时移除的装备／工具等组件要靠目标 onrepaired 恢复，回调参数中的材料还可能已经被消费移除。[forgerepair.lua:47](../../../raw/current/scripts/components/forgerepair.lua#L47 [pickaxe_lunarplant.lua:146](../../../raw/current/scripts/prefabs/pickaxe_lunarplant.lua#L146

## 验证记录

- 源码审阅：2026-10-26，Steam buildid 26148036；覆盖目标组件全文、材料结算、REPAIR 优先分派、MakeForgeRepairable、armor／finiteuses／fueled 初始化与运行刷新／读取路径，三类装备所选破损／恢复接线。未展开全部套装效果、装备完整存档外观恢复或所有修理目标。
- 结构检查：validate_page 及源码引用 strict 检查通过；无新增完整 Lua 示例。机械检查不代替游戏运行。
- 游戏运行：未运行；有限次数后安装、USAGE/no_sewing、两套 repairable 并存、多耐久状态、破损修复／读档回调待实测。

相关：[[reference/components/forgerepair]]；[[reference/components/armor]]；[[reference/components/finiteuses]]；[[reference/components/fueled]]；普通目标修复见 [[reference/components/repairable]]。
