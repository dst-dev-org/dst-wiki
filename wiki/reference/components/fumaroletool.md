---
title: "fumaroletool"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "气孔热工具组件：以物品温度驱动损坏与修复、四档温度档位与热光效，使用时消耗工具热量并轻微暖手，供热工具类武器。"
aliases: ["气孔工具", "热工具温度", "fumaroletool"]
tags: [component, item, lifecycle]
dst_build: "steam-buildid:26230496"
size_class: standard
sources: ["components/fumaroletool.lua", "standardcomponents.lua", "prefabs/axe.lua", "tuning.lua", "prefabs/trap_fumarole.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# fumaroletool

## 快速判断

- 职责与适用：给"靠温度工作"的工具（原版 fumaroleaxe／fumarolepickaxe 等五件）提供温度档位、冷却自毁与复热修复。构造断言宿主必须已有 `inventoryitemtemperature` 与 `heater` 组件（components/fumaroletool.lua:54）。
- 优先使用：走原版帮手 `MakeFumaroleTool`（standardcomponents.lua:2066-2140），它会一并配好 heater、inspectable 状态名、温度修改器与对冻结目标的伤害加成。
- 重要副作用：构造时生成 `heatrocklight` 子实体并随持有者转移（lines 47-48、67-82）；温度 ≤0 自毁、≥60 自修（lines 5-9）；`broken` 属性变化自动增删 `broken` 标签（lines 1-3、57-59）。
- 修改边界：`OnRemoveFromEntity` 解除监听并移除物品源登记（lines 61-65）；损坏时原版回调会移除 equippable／tool／weapon 组件（prefabs/axe.lua:247-252）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SetBroken(broken, isloading)` | `(bool, bool可选)` | 变化时触发 `onbroken`／`onrepaired` 回调（lines 88-103） |
| `SetOnBroken`／`SetOnRepaired`／`SetHeatOnUse`／`SetOnUpdateTemperatureRange` | 回调注册 | 原版均在 `MakeFumaroleTool` 内注入（standardcomponents.lua:2127-2131） |
| `OnUsed(doer, target)` | 使用消耗 | 以 `aoediminishingreturns` 倍率缩放后 `AddTemperature(heatonuse)` 冷却工具、给持有者 `DoDelta(-heatonuse)` 升温（lines 121-131）；`heatonuse` 为负值，如斧 -0.2（tuning.lua:9488-9492） |
| `OnUsedAsItem(action, doer, target)` | 缓冲动作入口 | `CanDoAction` 通过才调 `OnUsed`（lines 149-153）；攻击路径经 `weapononattack` 事件（lines 33-35） |
| `UpdateTempRange`／`GetTempRange` | `-> 1..4` | 按 `TUNING.FUMAROLETOOL_TEMPS = {0,20,40,60}` 分档（lines 133-147；tuning.lua:9473），档位驱动贴图／灯光（prefabs/axe.lua:194-208） |
| `OnBuilt(builder)` | 建造初始化 | 初始温度 90，按建造者湿度比例扣减最多 60（lines 155-158；tuning.lua:9468-9469） |
| `OnUpdateOwner` | 灯光跟随 | 光效挂到 `GetGrandOwner`；在 `pocketdimension_container`／`buried` 内隐藏（lines 67-82），并兼任物品源新主人回调（lines 84-85） |
| `OnSave`／`OnLoad` | 存 `broken` | 载入时以 `isloading=true` 恢复（lines 160-170） |

## 典型接入

### 热工具的温度生命周期（原版路径）

- 前提与选择：工具需 `inventoryitem`（开启温度）、`inventoryitemtemperature`、`heater`；`MakeFumaroleToolPristine` 提前加 `heatrock`、`HASHEATER`、`show_broken_ui` 标签（standardcomponents.lua:2057-2064）。
- 生效与撤销：气孔陷阱把放上去的工具加热到上限（trap_fumarole.lua:604-612 同款温度机制）；随后 `fumaroletool_mod`（-30）持续降温，降到 ≤0 触发 `onbroken`（取消装备、播放 broken 动画、`finiteuses:Use(1)`，standardcomponents.lua:2079-2108），重新加热到 ≥60 时 `onrepaired`。
- 原版依据：prefabs/axe.lua:282-286（pristine 标签 + `MakeFumaroleTool(inst, TUNING.FUMAROLEAXE_HEAT_ON_USE, ...)`）；WX-78 状态图也直接调用 `OnUsed`（stategraphs/SGwx78_common.lua:557-571）。

```lua
-- 自定义热工具 master postinit（依赖组件先行）：
inst:AddComponent("inventoryitem")
inst.components.inventoryitem:EnableTemperature(true)
inst:AddComponent("inventoryitemtemperature")
inst:AddComponent("heater")
inst:AddComponent("fumaroletool")
inst.components.fumaroletool:SetHeatOnUse(-0.2) -- 每次使用降温 0.2 并暖持有者
inst.components.fumaroletool:SetOnBroken(function(inst)
    inst:RemoveComponent("tool") -- 原版还移除 equippable/weapon 并切换 broken 动画
end)
inst.components.fumaroletool:SetOnRepaired(function(inst)
    inst.components.inspectable.nameoverride = nil
end)
```

- 调整与验证：温度单位与 inventoryitemtemperature 一致；对冻结目标的额外伤害走 `damagetypebonus` 回调（standardcomponents.lua:2038-2050、2133-2135），倍率表见 tuning.lua:9475-9476。

## 常见陷阱与限制

- **缺依赖直接崩：** 无 `inventoryitemtemperature`／`heater` 时构造断言失败（components/fumaroletool.lua:54），必须按上例顺序补齐。
- **heatonuse 符号：** 该值是"对工具的温度增量"，原版传负数表示使用降温；若误传正数会让工具越用越热（lines 121-131）。
- **复热门槛：** 修复阈值是 `TEMPS[4]`（60）而非任意正值；停在 1~59 度仍是损坏态（lines 5-9）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。通读 components/fumaroletool.lua 全文（172 行）；核验 standardcomponents.lua:2031-2140 帮手与回调、axe.lua:247-286、tuning.lua:9467-9492、trap_fumarole.lua:604-612、SGwx78_common.lua:557-571。未运行游戏。
- 结构检查：validate_page.py / validate_source_links.py 通过（见批级自验）。

相关：[[reference/components/inventoryitemtemperature]]、[[reference/components/heater]]、[[reference/components/aoediminishingreturns]]
