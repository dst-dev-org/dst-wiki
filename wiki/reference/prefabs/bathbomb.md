---
title: "沐浴球（bathbomb）"
created: "2026-11-10"
updated: "2026-11-10"
type: prefab
summary: "prefabs/bathbomb.lua 单注册的温泉浴弹：月亮树花＋硝石合成的可堆叠投掷耗材，对可浴弹（bathbombable）的温泉出 BATHBOMB 动作后消耗自身并进入 bathbombed 状态——月岛温泉发光发热、满月变玻璃，隐居者温泉激活水源并让 Pearl 触发泡澡。本 build 已整体移除 perishable，不腐坏。"
aliases: ["bathbomb", "沐浴球", "浴球", "浴弹", "Bath Bomb"]
tags: [prefab, item]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/bathbomb.lua", "components/bathbomb.lua", "components/bathbombable.lua", "prefabs/hotspring.lua", "prefabs/hermithotspring.lua", "brains/hermitcrabbrain.lua", "stategraphs/SGhermitcrab.lua", "widgets/itemtile.lua", "componentactions.lua", "actions.lua", "standardcomponents.lua", "tuning.lua", "recipes.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 沐浴球（bathbomb）

## 快速判断

- 职责与适用：`prefabs/bathbomb.lua` 注册 1 个可拾取耗材 `bathbomb`（[fn L16-L67](../../../raw/current/scripts/prefabs/bathbomb.lua#L16-L67)）。唯一玩法出口是 BATHBOMB 动作：把浴弹投进带 `bathbombable` 组件的温泉（月岛 `hotspring` 与隐居者 `hermithotspring`），效果由温泉侧回调实现。
- 优先使用：给自定义物品加"可投浴弹"能力直接挂 `bathbomb` 组件（典型接入）；查温泉被浴弹后的状态变化读本页"投弹结算与温泉侧效果"。
- 重要副作用：投弹成功即销毁浴弹（[actions.lua L4818-L4825](../../../raw/current/scripts/actions.lua#L4818-L4825)）；温泉进入 `is_bathbombed` 态（[bathbombable.lua L30-L35](../../../raw/current/scripts/components/bathbombable.lua#L30-L35)），月岛温泉成为 90 单位热源（[hotspring.lua L234-L243](../../../raw/current/scripts/prefabs/hotspring.lua#L234-L243)），隐居者温泉开启水源供 Pearl 泡澡（[hermithotspring.lua L479-L491](../../../raw/current/scripts/prefabs/hermithotspring.lua#L479-L491)）。
- 修改边界：浴弹本体无 SG／Replica，纯服务端组件；温泉侧的玻璃化／补水／Pearl 好感结算归温泉与 Pearl 的机制，本页只列入口。未覆盖：温泉存档细节全链、SOAKIN 后的 Pearl 好感数值。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `bathbomb`（沐浴球） | `bathbomb()`：网络段（bank/build `bath_bomb`、`show_spoilage` 标签、浮水 small）→ `SetPristine` → master 段（小燃烧 `MED_BURNTIME`＋传播、inspectable、inventoryitem、stackable MEDITEM 20、tradable、fuel `MED_FUEL`、bathbomb 组件、hauntable） | `show_spoilage` 在 pristine 段（[L29](../../../raw/current/scripts/prefabs/bathbomb.lua#L29)）但本 build 无 perishable（见陷阱）；组件接线 [L39-L64](../../../raw/current/scripts/prefabs/bathbomb.lua#L39-L64)；堆叠与燃料取 [tuning.lua L82](../../../raw/current/scripts/tuning.lua#L82)、[L1675](../../../raw/current/scripts/tuning.lua#L1675)、[L1682](../../../raw/current/scripts/tuning.lua#L1682) |

## 本 build 无 perishable（线索回源结论）

派发线索提到的"perishable 疑似残留"回源核实为**整体移除**，两处调用点都是块注释：

1. [bathbomb.lua L12-L14](../../../raw/current/scripts/prefabs/bathbomb.lua#L12-L14)：`OnPutInInventory` 函数（内含 `StartPerishing`）整体包在 Lua 块注释中，其挂接调用 [L45](../../../raw/current/scripts/prefabs/bathbomb.lua#L45) 也被注释；
2. [bathbomb.lua L55-L60](../../../raw/current/scripts/prefabs/bathbomb.lua#L55-L60)：`AddComponent("perishable")`＋`SetPerishTime(PERISH_FAST)`＋`StartPerishing`＋`onperishreplacement` 整段包在 Lua 块注释中，L56 有 `@V2C 12/25/2025: removed perishable` 备注；
3. 佐证：`prefabs` 数组（[L7-L10](../../../raw/current/scripts/prefabs/bathbomb.lua#L7-L10)）与返回值尾部的 `--, prefabs`（[L69](../../../raw/current/scripts/prefabs/bathbomb.lua#L69)）同步被注释，本实体不依赖 `spoiled_food`。

结论：本 build 浴球**不腐坏、不变质食物、无新鲜度**；残留的 `show_spoilage` 标签当前无实际效果——图鉴条的新鲜度显示要求物品同时带 `fresh`/`stale`/`spoiled` 标签（由 perishable 维护），浴球没有（[itemtile.lua L732-L748](../../../raw/current/scripts/widgets/itemtile.lua#L732-L748)）。

## 投弹结算与温泉侧效果

- **候选动作**：物品带 `bathbomb` 标签（bathbomb 组件挂载，[components/bathbomb.lua L4-L8](../../../raw/current/scripts/components/bathbomb.lua#L4-L8)）且目标带 `bathbombable` 标签（bathbombable 组件按 `can_be_bathbombed` 同步，[components/bathbombable.lua L2-L8](../../../raw/current/scripts/components/bathbombable.lua#L2-L8)）→ BATHBOMB（[componentactions.lua L1052-L1056](../../../raw/current/scripts/componentactions.lua#L1052-L1056)）。
- **结算**：`can_be_bathbombed` 才执行 `OnBathBombed(invobject, doer)` 并把浴弹从背包移除销毁（[actions.lua L4818-L4825](../../../raw/current/scripts/actions.lua#L4818-L4825)）；组件侧置 `is_bathbombed=true`、`can_be_bathbombed=false`（[bathbombable.lua L30-L35](../../../raw/current/scripts/components/bathbombable.lua#L30-L35)）。
- **月岛 `hotspring`**：满月瞬间玻璃化（`TurnToGlassed`），否则点灯＋开启洗浴池＋发光动画＋音效（[hotspring.lua L258-L271](../../../raw/current/scripts/prefabs/hotspring.lua#L258-L271)）；bombed 态作为热源输出 `HOTSPRING_HEAT.ACTIVE`＝90（未投弹而有水为 PASSIVE 45，[hotspring.lua L234-L243](../../../raw/current/scripts/prefabs/hotspring.lua#L234-L243)、[tuning.lua L3792-L3796](../../../raw/current/scripts/tuning.lua#L3792-L3796)）；状态经 OnSave/OnLoad 持久化（[hotspring.lua L300-L308](../../../raw/current/scripts/prefabs/hotspring.lua#L300-L308)）。
- **隐居者 `hermithotspring`**（Pearl 家）：激活水源、开池、启动 `bathbombed` 计时一整天（`HERMITHOTSPRING_BATHBOMB_DURATION`＝`total_day_time`，[hermithotspring.lua L479-L491](../../../raw/current/scripts/prefabs/hermithotspring.lua#L479-L491)、[tuning.lua L9232](../../../raw/current/scripts/tuning.lua#L9232)）。
- **Pearl 反应**：她的 Brain 找 20 距离内温泉泡澡（SOAKIN），要求该泉不在 `bathbombable`（未投弹）态且 `bathbombed` 剩余 ≥30 秒（`HERMITCRAB_HOTSPRING_SOAK_TIME`，[hermitcrabbrain.lua L494-L505](../../../raw/current/scripts/brains/hermitcrabbrain.lua#L494-L505)、[tuning.lua L9240](../../../raw/current/scripts/tuning.lua#L9240)）；动作经 SGhermitcrab 转 `soakin_pre`（[SGhermitcrab.lua L334](../../../raw/current/scripts/stategraphs/SGhermitcrab.lua#L334)）。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：bathbomb | 纯标签载体（挂／摘 `bathbomb` 标签），无字段无回调 | 服务端 | [[reference/components/bathbomb]]；[components/bathbomb.lua L1-L8](../../../raw/current/scripts/components/bathbomb.lua#L1-L8) |
| 组件：stackable／fuel／burnable 等 | 20 堆叠、`MED_FUEL` 燃料、`MED_BURNTIME` 燃烧、tradable | 服务端 master 段 | [bathbomb.lua L39-L53](../../../raw/current/scripts/prefabs/bathbomb.lua#L39-L53) |
| 组件：bathbombable（目标侧） | `can_be_bathbombed` 镜像标签；`OnBathBombed` 记状态并回调宿主 | 温泉实体；`Reset`／`DisableBathBombing` 由温泉补水／玻璃化流程调用 | [[reference/components/bathbombable]]；[bathbombable.lua L39-L46](../../../raw/current/scripts/components/bathbombable.lua#L39-L46) |
| 关联 Prefab：`hotspring` | 月岛温泉：bombed 发光发热、满月玻璃化 | 温泉侧状态机归温泉页，Wiki 待建 | [hotspring.lua L145-L226](../../../raw/current/scripts/prefabs/hotspring.lua#L145-L226)（Refill／TurnToGlassed／RemoveGlass） |
| 关联 Prefab：`hermithotspring` | 隐居者温泉：激活水源＋计时态 | Wiki 待建 | [hermithotspring.lua L390](../../../raw/current/scripts/prefabs/hermithotspring.lua#L390)（EnableBathingPool）、[L479-L491](../../../raw/current/scripts/prefabs/hermithotspring.lua#L479-L491) |
| Brain：Pearl `DoSoakin` | bombed 泉的泡澡触发 | 需剩余 ≥30 秒 | [hermitcrabbrain.lua L494-L505](../../../raw/current/scripts/brains/hermitcrabbrain.lua#L494-L505) |
| 资源 | `bath_bomb.zip`＋`INV_IMAGE bathbomb` | assets 声明与 bank/build 一致 | [bathbomb.lua L1-L5](../../../raw/current/scripts/prefabs/bathbomb.lua#L1-L5) |

## 典型接入

### 让自定义物品可作浴弹

- 前提与选择：目标侧已有 `bathbombable` 宿主（或自建）时，物品挂 `bathbomb` 组件即可被原版动作识别，无需写动作代码。
- 生效与撤销：标签随组件增删自动同步（[components/bathbomb.lua L4-L9](../../../raw/current/scripts/components/bathbomb.lua#L4-L9)）；投弹消耗由原版 BATHBOMB 结算负责。

```lua
-- 自定义物品 fn() 的 master 段
inst:AddComponent("bathbomb")

-- 目标侧（如自定义"可浴弹的池子"）在 master 段：
inst:AddComponent("bathbombable")
inst.components.bathbombable:SetOnBathBombedFn(function(inst, item, doer)
    -- item 为被消耗的浴弹实体，此时已从持有者移除、即将销毁
    inst.is_bathbombed = true
end)
```

- 调整与验证：`can_be_bathbombed` 默认 true（[bathbombable.lua L10-L15](../../../raw/current/scripts/components/bathbombable.lua#L10-L15)），想阶段性开门禁用 `DisableBathBombing`／`Reset`；投弹物品的销毁在动作 fn 内完成，`OnBathBombedFn` 里不要再移除 invobject。

## 获得路径

- **合成**：月亮树花×1＋硝石×1，`TECH.CELESTIAL_ONE`（天体祭坛一级），`nounlock`（[recipes.lua L835](../../../raw/current/scripts/recipes.lua#L835)）。原料见 [[reference/prefabs/moontree-blossom]]。
- 无掉落与生成点：快照内无 `SpawnPrefab("bathbomb")` 消费者，配方是唯一入口。

## 常见陷阱与限制

- **误区：浴球会腐坏成变质食物。** 本 build 已整体移除 perishable（证据见上节），这是 2025-12-25 的改动；照旧版 wiki 描述"像花瓣一样腐坏"在本 build 不成立。
- **边界：投弹后温泉不复位由温泉侧管理。** 月岛温泉的 `is_bathbombed` 无计时（保存即恢复），复位走满月玻璃化／补水流程；隐居者温泉则按 `bathbombed` 计时结束复位——两类泉不要套同一套预期。
- **边界：show_spoilage 是残留标签。** 带 tag 但无 perishable 时图鉴新鲜度条不显示（[itemtile.lua L732-L748](../../../raw/current/scripts/widgets/itemtile.lua#L732-L748)），模组仿制时不必携带该标签。

## 验证记录

- 源码审阅：2026-11-10，steam-buildid:26474873。`prefabs/bathbomb.lua` 全文（含三处块注释与 removed perishable 备注）；`components/bathbomb.lua`／`bathbombable.lua` 全文；BATHBOMB 动作定义与 componentactions 候选；hotspring／hermithotspring 的 OnBathBombed／GetHeat／OnSave 段与 Refill／TurnToGlassed 入口；Pearl brain 的 DoSoakin 与 SOAKIN 的 SG 接线；itemtile 的 show_spoilage 消费条件；tuning 热度／时长／堆叠／燃料值；配方；`chinese_s.po` `NAMES.BATHBOMB` 实显（沐浴球）。
- 未覆盖：温泉玻璃化与补水的完整状态机、Pearl 泡澡的好感结算、浴弹皮肤数据。
- 游戏运行：未运行。

返回 [[reference/prefabs/index|原版 Prefab 索引]]。
