---
title: "smotherer"
created: "2026-10-21"
updated: "2026-10-28"
type: component
summary: "空标记组件：让持有它的物品注册 SMOTHER（以及带 frozen 标签时的 MANUALEXTINGUISH）动作；实际消耗与灭火状态在 actions 与 burnable。"
aliases: ["扑灭阴燃", "闷熄", "手动灭火", "SMOTHER", "smotherer"]
tags: [component, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/smotherer.lua", "componentactions.lua", "actions.lua", "components/burnable.lua", "components/playeractionpicker.lua", "components/playercontroller.lua", "entityscript.lua", "tuning.lua", "prefabs/fertilizer.lua", "prefabs/poop.lua", "prefabs/guano.lua", "prefabs/compost.lua", "prefabs/compostwrap.lua", "prefabs/mosquitofertalizer.lua", "prefabs/spoiledfood.lua", "prefabs/inv_rocks_ice.lua", "prefabs/snowball_item.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# smotherer

## 快速判断

- 职责与适用：**不是“让目标窒息”的行为组件，而是一个空标记**。组件本体只保存 `self.inst`，没有方法、字段、标签、事件或存档；它的全部作用是让持有它的物品被注册为 `smotherer` 组件动作提供者。[组件本体](../../../raw/current/scripts/components/smotherer.lua#L1-L6) [动作注册](../../../raw/current/scripts/entityscript.lua#L665)
- 优先使用：想让某件物品能扑灭阴燃物，就在该物品上 `AddComponent("smotherer")`；想让它还能浇灭明火，再让物品带 `frozen` 标签。[动作收集](../../../raw/current/scripts/componentactions.lua#L1736-L1743)
- 重要副作用：动作执行时消耗的是**物品自己**的 `finiteuses` 或 `stackable`，不是组件；没有这两者（也没有 `health`＋`combat`）的物品可以无限次扑灭而不被消耗。空手路径还会让玩家承受 3 点火焰伤害。[消耗](../../../raw/current/scripts/components/burnable.lua#L417-L429) [空手](../../../raw/current/scripts/components/playercontroller.lua#L2005-L2007)
- 修改边界：原版宿主都在服务端 `SetPristine` 之后添加；动作 ID 通过 `actioncomponents` 网络字段同步给客户端，所以不需要客户端再 `AddComponent`。移除组件会注销动作，但没有额外清理。[注册](../../../raw/current/scripts/componentactions.lua#L3182-L3188) [注销](../../../raw/current/scripts/entityscript.lua#L670-L684)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| 组件本体 | 仅 `self.inst` | 无任何公开方法或状态；不要调用 `components.smotherer:XXX()`。[L1-L6](../../../raw/current/scripts/components/smotherer.lua#L1-L6) |
| `AddComponent("smotherer")` | 组件名 | `AddComponent` 构造后调用 `RegisterComponentActions(name)`，把动作 ID 追加进实体的 `actioncomponents`；重复添加会被同名检查拦下。[L610-L645](../../../raw/current/scripts/entityscript.lua#L632-L667) |
| `RemoveComponent("smotherer")` | 组件名 | 移除后 `UnregisterComponentActions` 从 `actioncomponents` 删 ID，并同步网络字段；组件没有 `OnRemoveFromEntity`。[L648-L662](../../../raw/current/scripts/entityscript.lua#L670-L684) |
| `COMPONENT_ACTIONS.USEITEM.smotherer` | `(inst, doer, target, actions)` | 在**当前使用物品**上收集：目标有 `smolder` → 加 `ACTIONS.SMOTHER`；否则物品有 `frozen`、目标有 `fire` 且未被持有 → 加 `ACTIONS.MANUALEXTINGUISH`。[L1736-L1743](../../../raw/current/scripts/componentactions.lua#L1736-L1743) [物品动作](../../../raw/current/scripts/components/playeractionpicker.lua#L142-L147) |
| `ACTIONS.SMOTHER` | `Action({priority=1, mount_valid=true})` | 执行前要求目标 `burnable:IsSmoldering()`；`smotherer = act.invobject or act.doer`，再调 `SmotherSmolder`。[定义](../../../raw/current/scripts/actions.lua#L426) [执行](../../../raw/current/scripts/actions.lua#L1844-L1850) |
| `Burnable:SmotherSmolder(smotherer)` | 无返回 | 依次尝试：`finiteuses:Use()` → `stackable:Get():Remove()` → `health:DoFireDamage(SMOTHER_DAMAGE, nil, true)` 并 `PushEvent("burnt")`；最后 `StopSmoldering(-1)`。[L417-L429](../../../raw/current/scripts/components/burnable.lua#L417-L429) |
| `ACTIONS.MANUALEXTINGUISH` | `Action({priority=1})` | 要求 `act.invobject` 有 `frozen`、目标 `burnable:IsBurning()`；调 `Extinguish(true, TUNING.SMOTHERER_EXTINGUISH_HEAT_PERCENT, act.invobject)`。[定义](../../../raw/current/scripts/actions.lua#L427) [执行](../../../raw/current/scripts/actions.lua#L1852-L1857) |
| `Burnable:Extinguish(resetpropagator, heatpct, smotherer)` | 无返回 | 只消耗 `finiteuses`／`stackable`，**没有** `health` 分支；随后停止燃烧与传播。[L454-L463](../../../raw/current/scripts/components/burnable.lua#L454-L463) |
| `SMOTHER_DAMAGE`／`SMOTHERER_EXTINGUISH_HEAT_PERCENT` | `3`／`.2` | 前者用于 `health`＋`combat` 分支（空手扑灭时玩家即此类）；后者作为 `Extinguish` 的 `heatpct` 参数。[L3029](../../../raw/current/scripts/tuning.lua#L3029) [L2942](../../../raw/current/scripts/tuning.lua#L2942) |

### 谁写、谁读、何时撤

| 环节 | 位置与触发 | 结果 |
|---|---|---|
| 写 | 物品 prefab 的服务端初始化 | `AddComponent("smotherer")`；组件构造只写 `inst`，动作 ID 由 `RegisterComponentActions` 写入实体的 `actioncomponents` 并同步网络。[宿主](../../../raw/current/scripts/prefabs/poop.lua#L77-L101) [网络字段](../../../raw/current/scripts/entityscript.lua#L179-L195) |
| 读 | 客户端动作选择 | `PlayerActionPicker:GetUseItemActions` 对当前物品 `CollectActions("USEITEM", ...)`，再按 `actioncomponents` 找 `smotherer` 收集器。[收集](../../../raw/current/scripts/componentactions.lua#L3235-L3246) [调用](../../../raw/current/scripts/components/playeractionpicker.lua#L142-L147) |
| 读 | 服务端执行 | `ACTIONS.SMOTHER.fn`／`MANUALEXTINGUISH.fn` 只查目标 `burnable` 与物品标签，不再查 `components.smotherer`。[执行](../../../raw/current/scripts/actions.lua#L1844-L1857) |
| 撤 | `RemoveComponent("smotherer")` | 注销动作 ID 并同步；物品若已被消耗／移除，动作随实体消失；组件本身没有计时器或清理回调。[注销](../../../raw/current/scripts/entityscript.lua#L670-L684) |

## 典型接入

### 原版宿主：9 个文件、11 个注册名

| 宿主组 | 文件与注册名 | 关键配置 | 实际效果 |
|---|---|---|---|
| 肥料／粪便类 | `poop`、`compost`、`compostwrap`、`guano`、`mosquitofertilizer`、`spoiled_food`、`spoiled_fish`、`spoiled_fish_small` | `stackable` 与 `smotherer` 均在服务端 pristine 之后 | 扑灭一次消耗 1 个物品。[poop](../../../raw/current/scripts/prefabs/poop.lua#L90-L101) [compost](../../../raw/current/scripts/prefabs/compost.lua#L66-L77) [guano](../../../raw/current/scripts/prefabs/guano.lua#L84-L95) [compostwrap](../../../raw/current/scripts/prefabs/compostwrap.lua#L102-L124) [mosquito](../../../raw/current/scripts/prefabs/mosquitofertalizer.lua#L99-L106) [spoiled](../../../raw/current/scripts/prefabs/spoiledfood.lua#L121-L127) [注册名](../../../raw/current/scripts/prefabs/spoiledfood.lua#L263-L265) |
| 有限使用次数 | `fertilizer` | `finiteuses` + `smotherer`，无 `stackable` | 扑灭一次消耗 1 次使用次数。[L55-L66](../../../raw/current/scripts/prefabs/fertilizer.lua#L55-L66) |
| 冰冻物 | `ice`、`snowball_item` | `frozen` 标签在 pristine 前添加，`smotherer` 在服务端 | 既可扑灭 `smolder`，也可对 `fire` 执行手动灭火。[ice](../../../raw/current/scripts/prefabs/inv_rocks_ice.lua#L66-L90) [snowball](../../../raw/current/scripts/prefabs/snowball_item.lua#L183-L210) |

### 给自定义物品接入

- 前提与选择：物品必须能被“使用”（是当前激活物品）；想要消耗就让它有 `stackable` 或 `finiteuses`，再在服务端初始化里 `AddComponent("smotherer")`。想让同一物品还能灭明火，在 `SetPristine` 前加 `frozen` 标签，和原版冰／雪球一致。
- 生效与撤销：动作在物品成为当前使用物品时出现；扑灭成功消耗一次使用／一个堆叠。`RemoveComponent("smotherer")` 立即注销动作；不需要改 `burnable`，也不要在客户端单独添加。
- 单位与边界：一次动作消耗 1 次 `finiteuses` 或 1 个 `stackable`；空手路径伤害 3；手动灭火传入的 `heatpct` 是 `.2`。没有这两类消耗组件的自定义物品会“免费”扑灭，若要收费得自己在 `SmotherSmolder` 之外另做动作。
- 原版依据：以上行为都来自 `actions.lua` 与 `burnable.lua` 的固定分支，组件本身不提供可配置参数。[消耗分支](../../../raw/current/scripts/components/burnable.lua#L417-L429)

## 常见陷阱与限制

- **名不副实**：`smotherer` 不会“闷死”生物，也没有范围、持续时间或免疫；它只是动作注册标记，实际逻辑全在 `actions`＋`burnable`。[组件本体](../../../raw/current/scripts/components/smotherer.lua#L1-L6)
- **`smolder` 与 `fire` 是两条动作**：`SMOTHER` 只对 `smolder` 标签，`MANUALEXTINGUISH` 只对 `fire` 标签且要求物品 `frozen`；给物品加组件不会自动获得 `frozen`。[收集器](../../../raw/current/scripts/componentactions.lua#L1736-L1743)
- **消耗顺序固定**：同时有 `finiteuses` 和 `stackable` 时只走 `finiteuses`；都没有且没有 `health`＋`combat` 时不消耗，这是“无限扑灭”的常见来源。[L419-L426](../../../raw/current/scripts/components/burnable.lua#L419-L426)
- **空手也能扑灭**：控制器拾取／动作按钮路径对 `smolder` 目标直接返回 `SMOTHER` 并传 `nil` invobject，玩家因此走 `doer` 分支，受 3 点火焰伤害并触发 `burnt`；这条路径不依赖 `smotherer` 组件。[L2005-L2007](../../../raw/current/scripts/components/playercontroller.lua#L2005-L2007) [L2267-L2276](../../../raw/current/scripts/components/playercontroller.lua#L2270-L2279)
- **手动灭火的额外限制**：目标被人持有（`replica.inventoryitem:IsHeld()`）时不提供 `MANUALEXTINGUISH`；`SMOTHER` 分支没有这条检查。[L1739-L1741](../../../raw/current/scripts/componentactions.lua#L1739-L1741)
- **原版只在服务端添加**：动作 ID 由服务端 `RegisterComponentActions` 写入网络字段，原版宿主全部在 `if not ismastersim then return end` 之后添加；客户端单独添加会与权威动作列表脱节，不要这样做。[宿主时机](../../../raw/current/scripts/prefabs/poop.lua#L77-L101) [同步](../../../raw/current/scripts/componentactions.lua#L3182-L3188)
- **动作可见性与执行校验分离**：服务端 `SMOTHER.fn` 只看目标是否 `IsSmoldering()`，不检查 `components.smotherer`；客户端动作列表与服务端组件状态不同步时，会出现“动作可见但服务端不按同一物品语义执行”的错位，模组应保持两端一致。[L1841-L1852](../../../raw/current/scripts/actions.lua#L1844-L1855)
- **不是消耗品开关**：给物品加组件不会让它变成一次性物品；`finiteuses`／`stackable` 的原有语义不变，扑灭只是调用它们的一次消耗。[L417-L429](../../../raw/current/scripts/components/burnable.lua#L417-L429)

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 处理范围：锚点重定位 10 处（actions／componentactions／tuning 行号迁移，区间文本逐段比对一致；playercontroller 拾取放宽仅涉及 torch 白名单，不触及 SMOTHER 空手分支）；结论修订：无，仅锚点迁移。
- 依据：枢纽 diff 审阅（actions.lua／componentactions.lua hunk 地图）＋变更清单命中 componentactions.lua、actions.lua、components/playercontroller.lua、entityscript.lua、tuning.lua。

- 源码审阅：2026-10-21，Steam buildid 25643519；覆盖空组件构造、`AddComponent`／`RemoveComponent` 的动作注册与网络同步、`USEITEM` 收集器的两条分支、`SMOTHER`／`MANUALEXTINGUISH` 执行、`burnable` 的两种消耗路径、空手路径、调参值，以及 9 个文件／11 个注册名宿主。
- 未运行游戏；未覆盖：多人联机下动作列表同步时序、自定义物品与 `finiteuses`／`stackable` 的实际消耗表现、客户端预测。
- 本页无完整 Lua 示例，未执行 Lua 语法检查。
- 结构检查：`validate_page.py` 无 issue（83 行，任务上限 180 行／14 KiB）；`validate_source_links.py --strict` 0 error／0 warning（47 处源码引用）。

相关：[[reference/components/burnable]]。
