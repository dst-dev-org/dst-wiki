---
title: "核化液（coolant）"
created: "2026-11-10"
updated: "2026-11-10"
type: prefab
summary: "prefabs/coolant.lua 单注册的变异巨兽战利品：旺普boss击败后变异熊獾／独眼巨鹿／座狼掉落的可堆叠耗材，用于合成安保脉冲笼；挂 waterproofer 组件但有效度为 0——唯一实际效果是物品自身不受潮，「coolant」标签在快照内没有任何玩法消费者。"
aliases: ["coolant", "核化液", "冷却剂", "Coolant"]
tags: [prefab, item]
size_class: simple
dst_build: "steam-buildid:26474873"
sources: ["prefabs/coolant.lua", "components/waterproofer.lua", "components/inventoryitem.lua", "components/acidlevel.lua", "components/inventory.lua", "prefabs/bearger.lua", "prefabs/deerclops.lua", "prefabs/warg.lua", "screens/redux/scrapbookdata.lua", "prefablist.lua", "recipes.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 核化液（coolant）

## 快速判断

- 职责与适用：`prefabs/coolant.lua` 注册 1 个可堆叠物品 `coolant`（[fn L5-L45](../../../raw/current/scripts/prefabs/coolant.lua#L5-L45)）。玩家侧定位是**合成耗材**（安保脉冲笼用料）与变异巨兽掉落物，不是防水装备：waterproofer 组件有效度固定 0，玩家无法穿戴（无 equippable）。
- 优先使用：查掉落条件与消耗口读本页"获得路径与消耗"；想给物品做"不受潮"效果直接挂 waterproofer 组件（典型接入）。
- 重要副作用：物品永不积累潮湿——判定有两处：waterproofer 构造对已有 inventoryitem 调 `EnableMoisture(false)`（[waterproofer.lua L4-L10](../../../raw/current/scripts/components/waterproofer.lua#L4-L10)），inventoryitem 构造发现组件存在则不启用湿度（[inventoryitem.lua L87](../../../raw/current/scripts/components/inventoryitem.lua#L87)）；coolant 的挂载顺序（waterproofer 先、inventoryitem 后）走的是后一条。
- 修改边界：单注册、无 SG／Brain／Replica，纯服务端组件物品；「coolant」标签的消费者缺口见陷阱节。未覆盖：变异巨兽的生成与战斗（归巨兽页）、wagboss_tracker 组件内部。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `coolant`（核化液） | `fn()`：网络段（bank/build `coolant`、idle 循环动画、气泡高亮 `SetSymbolLightOverride`、`waterproofer`＋`coolant` 标签、浮水）→ `SetPristine` → master 段（随机起始帧、stackable SMALLITEM 40、inspectable、waterproofer `SetEffectiveness(0)`、hauntable、inventoryitem） | 防潮由双门禁兜底：waterproofer 先挂时其构造因 inventoryitem 未挂而跳过，随后 inventoryitem 构造检测到组件存在不开湿度（[coolant.lua L37-L42](../../../raw/current/scripts/prefabs/coolant.lua#L37-L42)、[inventoryitem.lua L87](../../../raw/current/scripts/components/inventoryitem.lua#L87)）；随机帧只在 master 段设置（[L30](../../../raw/current/scripts/prefabs/coolant.lua#L30)），客户端看到的帧由服务端动画同步 |

## waterproofer(0) 的实际消费者

派发要求"追到调用点"，快照内 waterproofer 组件的全部消费者只有四类，逐一核对：

| 消费者 | 调用点 | 对 coolant 的实际效果 |
|---|---|---|
| inventoryitemmoisture 开关 | waterproofer 构造函数：若有 inventoryitem 则 `EnableMoisture(false)`（[waterproofer.lua L4-L10](../../../raw/current/scripts/components/waterproofer.lua#L4-L10)）；卸载组件时恢复（[L14-L18](../../../raw/current/scripts/components/waterproofer.lua#L14-L18)）；`EnableMoisture(false)` 移除 `inventoryitemmoisture` 组件（[inventoryitem.lua L116-L125](../../../raw/current/scripts/components/inventoryitem.lua#L116-L125)，另一处同类判定 [L87](../../../raw/current/scripts/components/inventoryitem.lua#L87)） | **coolant 作为物品永不积累潮湿**（泡水／雨天拿在手里都不会变湿）——这是 waterproofer(0) 唯一实际生效的分支 |
| 酸雨腐蚀（acidlevel） | 遍历**装备槽**、只把 `effectiveness > 0` 的装备计入吸伤分摊（[acidlevel.lua L161-L169](../../../raw/current/scripts/components/acidlevel.lua#L161-L169)） | 无效：coolant 无 equippable，进不了装备槽；即使能穿，0 也被 `> 0` 过滤 |
| 装备防水合计（inventory） | 遍历**装备槽**累加 `GetEffectiveness()`（[inventory.lua L2722-L2731](../../../raw/current/scripts/components/inventory.lua#L2722-L2731)） | 同上，无效 |
| 调试命令 | debugcommands 展示用 | 不构成玩法 |

结论：核化液挂 waterproofer 是"物品防潮"实现方式，**不是**防水功能——它不给玩家提供任何雨/酸雨保护。`waterproofer` pristine 标签本身在快照内也查不到 `HasTag("waterproofer")` 消费者，作用是给组件 action/优化留的候选标记（组件构造时还会重复加一次，[waterproofer.lua L6-L7](../../../raw/current/scripts/components/waterproofer.lua#L6-L7)）。

## 获得路径与消耗

- **掉落**：击败变异巨兽（mutatedbearder／mutateddeerclops／mutatedwarg）固定掉 1 个，门禁为 `wagboss_tracker:IsWagbossDefeated()`（旺普 boss 已被击败才掉，[bearger.lua L753-L756](../../../raw/current/scripts/prefabs/bearger.lua#L753-L756)、[deerclops.lua L650-L653](../../../raw/current/scripts/prefabs/deerclops.lua#L650-L653)、[warg.lua L777-L780](../../../raw/current/scripts/prefabs/warg.lua#L777-L780)；接线 [bearger.lua L828](../../../raw/current/scripts/prefabs/bearger.lua#L828)）。
- **消耗**：`wagpunk_workstation_security_pulse_cage` 配方用料（铥矿×2＋月暴静电体×1＋核化液×1，[recipes.lua L1021](../../../raw/current/scripts/recipes.lua#L1021)）。
- **不可合成**：coolant 没有以它为产物的配方；图鉴登记为 item 类、堆叠上限 40（[scrapbookdata.lua L353](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L353)，与 `STACK_SIZE_SMALLITEM` 一致）。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：waterproofer | `SetEffectiveness(0)`＋防潮开关 | 服务端；换材料无效度语义见组件页 | [[reference/components/waterproofer]]；[coolant.lua L37-L38](../../../raw/current/scripts/prefabs/coolant.lua#L37-L38) |
| 组件：stackable／inventoryitem／inspectable | 40 堆叠 | 服务端 | [coolant.lua L32-L42](../../../raw/current/scripts/prefabs/coolant.lua#L32-L42)、[tuning.lua L83](../../../raw/current/scripts/tuning.lua#L83) |
| API／数据：wagboss_tracker | 掉落门禁（boss 击败状态） | 世界级组件状态 | [bearger.lua L754](../../../raw/current/scripts/prefabs/bearger.lua#L754) |
| 关联 Prefab：`mutatedbearger`／`mutateddeerclops`／`mutatedwarg` | 掉落来源（`LootSetupFn_mutated` → `SetLoot`） | 巨兽侧变异态；掉落表另有 `SetChanceLootTable` | [bearger.lua L753-L756](../../../raw/current/scripts/prefabs/bearger.lua#L753-L756)；三巨兽页 Wiki 待建 |
| 资源 | `anim/coolant.zip` | assets 声明与 bank/build 一致 | [coolant.lua L1-L3](../../../raw/current/scripts/prefabs/coolant.lua#L1-L3) |

## 典型接入

### 给自定义物品做"永不潮湿"

- 前提与选择：物品（如纸类、火药类）不希望泡水变湿时，挂 waterproofer 即可；有效度按需给 0（纯防潮）或正值（穿戴时还提供防水）。
- 生效与撤销：构造时生效；移除组件自动恢复湿度积累（[waterproofer.lua L14-L18](../../../raw/current/scripts/components/waterproofer.lua#L14-L18)）。

```lua
-- 自定义物品 fn() 的 master 段（先后顺序随意，双门禁都会关闭湿度）
inst:AddComponent("waterproofer")
inst.components.waterproofer:SetEffectiveness(0)   -- 不提供穿戴防水，仅物品自身不受潮
inst:AddComponent("inventoryitem")
```

- 调整与验证：组件构造内部也会补 `waterproofer` 标签（[waterproofer.lua L6-L7](../../../raw/current/scripts/components/waterproofer.lua#L6-L7)）；若物品可能被玩家穿戴并把有效度调为正值，acidlevel 会把它计入酸雨吸伤分摊（[acidlevel.lua L161-L169](../../../raw/current/scripts/components/acidlevel.lua#L161-L169)）。

## 常见陷阱与限制

- **边界：「coolant」标签当前无玩法消费者。** 对 `raw/current/scripts` 全部 `.lua` 检索 `coolant`（含大小写变体）仅命中：注册清单 [prefablist.lua L315](../../../raw/current/scripts/prefablist.lua#L315)、配方按注册名配料（[recipes.lua L1021](../../../raw/current/scripts/recipes.lua#L1021)）、图鉴数据（scrapbook_prefabs.lua／scrapbookdata.lua）与三处巨兽掉落表。没有任何 `HasTag("coolant")` 分支——该标签当前只作标记，模组可自行消费但不能指望原版行为。
- **误区：核化液能当雨衣材料给玩家防水。** 组件有效度为 0 且无 equippable，防水合计与酸雨分摊两条消费链都到不了它（见上表）。
- **边界：掉落绑定旺普 boss 进度。** 未击败 wagboss 前变异巨兽不掉 coolant（`SetLoot(nil)`），只有各自的 `SetChanceLootTable` 杂项。

## 验证记录

- 源码审阅：2026-11-10，steam-buildid:26474873。`prefabs/coolant.lua` 全文；`waterproofer`／`inventoryitem`（EnableMoisture 段）全文；`acidlevel`（酸雨分摊段）与 `inventory`（防水合计段）的消费点核对；`grep coolant` 全 scripts 范围反查（标签消费者缺口结论即来自该次封闭检索）；三巨兽掉落接线段；图鉴/注册清单交叉核对；tuning 堆叠常量；配方；`chinese_s.po` `NAMES.COOLANT` 实显（核化液）。
- 未覆盖：`inventoryitemmoisture` 组件内部、巨兽变异条件与战斗、wagboss_tracker 的状态维护。
- 游戏运行：未运行。

返回 [[reference/prefabs/index|原版 Prefab 索引]]。
