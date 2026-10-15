---
title: "moistureabsorbersource"
created: "2026-10-21"
updated: "2026-10-28"
type: component
summary: "在持有物品上声明可被持有者发现的吸湿源，由回调提供干燥速率并自行承担储水。"
aliases: ["吸湿源", "干燥物品", "除湿物品", "desiccant", "moistureabsorbersource"]
tags: [component, lifecycle, item]
dst_build: "steam-buildid:26230496"
sources: ["components/moistureabsorbersource.lua", "components/moistureabsorberuser.lua", "components/moisture.lua", "componentutil.lua", "prefabs/desiccant.lua", "components/inventoryitem.lua", "components/inventoryitemmoisture.lua", "tuning.lua", "prefablist.lua", "recipes.lua"]
reliability: "verified"
validation: "source-reviewed"
status: "active"
---

# moistureabsorbersource

## 快速判断

- 职责与适用：挂在**物品**上的吸湿源声明。组件只保存两个回调，并把物品登记为库存来源；每秒吸多少、吸到哪里、何时吸满都由宿主回调决定，组件本身没有速率、储水或上限。[构造](../../../raw/current/scripts/components/moistureabsorbersource.lua#L1-L7) [GetDryingRate](../../../raw/current/scripts/components/moistureabsorbersource.lua#L31-L41)
- 优先使用：自定义可携带／可装备的除湿物品时，确保物品先有 `inventoryitem`，再加本组件并设置 `SetGetDryingRateFn`、`SetApplyDryingFn`。原版只有 desiccant 的两个注册名使用它。[desiccant 接入](../../../raw/current/scripts/prefabs/desiccant.lua#L156-L163)
- 重要副作用：物品进入背包或装备格时，按**最终持有者**（grand owner，含穿在身上的容器）自动添加 `moistureabsorberuser`；离开、丢地或销毁时按同一持有者撤销。持有者没有 moisture 时不添加，也不产生吸湿。[OnItemSourceNewOwner](../../../raw/current/scripts/components/moistureabsorbersource.lua#L20-L25) [grand owner](../../../raw/current/scripts/componentutil.lua#L1810-L1818)
- 修改边界：服务端组件、无客户端副本；构造期直接读取 `inventoryitem.owner`；雨天效率与储水归属都在宿主回调内，组件不钳制。[绑定实现](../../../raw/current/scripts/componentutil.lua#L1833-L1839)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SetGetDryingRateFn(fn)` | `fn(inst, rate) -> number` | 登记查询回调；`rate` 是持有者当前湿度速率（正=变湿），不是物品状态。未登记时返回 0。[设置](../../../raw/current/scripts/components/moistureabsorbersource.lua#L27-L29) |
| `GetDryingRate(rate)` | number，湿度点／秒 | 无回调返回 0；`rate > 0` 时把回调结果乘 `MOISTUREABSORBER_RAINED_ON_EFFICIENCY_MULT`（0.07），`rate <= 0` 不乘。[实现](../../../raw/current/scripts/components/moistureabsorbersource.lua#L31-L41) |
| `SetApplyDryingFn(fn)` | `fn(inst, rate, dt)` | 只在该源被持有者选中时调用；`rate` 仍是持有者速率，不是乘过 0.07 的干燥量；`dt` 来自 moisture 更新。未登记则什么也不做。[ApplyDrying](../../../raw/current/scripts/components/moistureabsorbersource.lua#L43-L51) |
| `OnItemSourceNewOwner(owner)`／`OnItemSourceRemoved(owner)` | 由 componentutil 回调 | 前者仅在 owner 有 moisture 时确保 user 组件并 `AddSource(self.inst)`；后者只调用 user 的 `RemoveSource`。不要手动调用。[user 增删](../../../raw/current/scripts/components/moistureabsorberuser.lua#L9-L18) |
| `OnRemoveFromEntity()` | 移除本组件时 | 解绑库存事件并触发 `OnItemSourceRemoved`，让持有者撤掉该来源。[清理](../../../raw/current/scripts/components/moistureabsorbersource.lua#L9-L11) |

### 成对结构与接线

物品侧只负责“声明 + 提供回调”，持有者侧的 [[reference/components/moistureabsorberuser]] 负责集合、扫描与选择，`moisture` 在每帧更新时调用持有者侧；顺序见该页“关键接口”。绑定由 `MakeComponentAnInventoryItemSource` 完成：监听物品自身的 `onputininventory`、`ondropped`、`onremove`；物品在容器内时还监听该容器的同名事件，用 `inventoryitem:GetGrandOwner()` 重算 owner。[事件绑定](../../../raw/current/scripts/componentutil.lua#L1777-L1840) [容器监听](../../../raw/current/scripts/componentutil.lua#L1789-L1796)

## 典型接入

### 原版宿主：desiccant 与 desiccantboosted

- 文件与注册：`prefabs/desiccant.lua` 返回 `desiccant`、`desiccantboosted` 两个 Prefab；`prefablist.lua` 按文件登记 `desiccant`，两个注册名同文件产生。制造站科技 `TECH.SHELLWEAVER_ONE`／`THREE` 的配方分别产出二者。[注册](../../../raw/current/scripts/prefabs/desiccant.lua#L181-L182) [prefablist](../../../raw/current/scripts/prefablist.lua#L365 [recipes](../../../raw/current/scripts/recipes.lua#L985-L986)
- 单位与数值：普通 0.5、强化 0.75，单位是**湿度点／秒**；两者都用自身 `inventoryitemmoisture:DoDelta(常量 * dt)` 累积，吸到 `TUNING.MAX_WETNESS`（100）后 `GetRate` 返回 0，持有者的选择会跳过它。[速率与储水](../../../raw/current/scripts/prefabs/desiccant.lua#L16-L38) [满值门禁](../../../raw/current/scripts/prefabs/desiccant.lua#L7-L14) [MAX_WETNESS](../../../raw/current/scripts/tuning.lua#L88
- 端与生效：组件只在 `TheWorld.ismastersim` 分支添加，且位于 `inventoryitem`／`inventoryitemmoisture` 之后。拾取、装备、进出容器由 inventoryitem 的 `onputininventory` 推送 user 增删；丢地、销毁或 `RemoveComponent` 时撤销。[宿主装配](../../../raw/current/scripts/prefabs/desiccant.lua#L139-L163) [OnPutInInventory](../../../raw/current/scripts/components/inventoryitem.lua#L348-L365)
- 撤销与边界：desiccant 把自身湿度设为外部控制，并按温度与持有者是否已干燥切换是否恢复自然漂移（见 [[reference/components/inventoryitemmoisture]]）；充能只发生在被持有者选中时，温度分支不属于本组件。

### 把一个已有物品接成吸湿源

前提：物品已有 `inventoryitem` 与可用的储水组件（示例沿用 `inventoryitemmoisture`），且持有者已有 moisture。仅服务端接入；替换 `myitem` 与速率即可。

```lua
-- modmain.lua：把已有物品接成吸湿源；myitem 换成你的 prefab 名。
local ABSORB_RATE = 0.5 -- 湿度点/秒，与 desiccant 的 0.5 同单位

local function GetRate(inst, owner_rate)
    local stored = inst.components.inventoryitemmoisture
    if stored == nil or stored.moisture >= GLOBAL.TUNING.MAX_WETNESS then
        return 0 -- 无储水组件或已吸满：退出竞争
    end
    return ABSORB_RATE
end

local function ApplyDrying(inst, owner_rate, dt)
    local stored = inst.components.inventoryitemmoisture
    if stored ~= nil then
        stored:DoDelta(ABSORB_RATE * dt) -- 只有被选中的源会被调用
    end
end

AddPrefabPostInit("myitem", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    if inst.components.inventoryitem == nil or inst.components.moistureabsorbersource ~= nil then
        return
    end
    local source = inst:AddComponent("moistureabsorbersource")
    source:SetGetDryingRateFn(GetRate)
    source:SetApplyDryingFn(ApplyDrying)
end)
```

- 为什么放在 postinit：此时 prefab 已构造完，正常物品的 `inventoryitem` 已存在；下方仍保留 nil 检查，避免接到没有该组件的实体上。
- 生效与撤销：拾取后由持有者扫描并扣速率，`GetRate` 返回 0 即退出；移除本组件或让物品离开持有者即撤销。不要手动对持有者 `AddSource`。
- 充能边界：`ApplyDrying` 不受雨天 0.07 折算影响，示例会全速给物品充能；若希望雨天少充，需要在该回调内自行按 `owner_rate` 判断。

## 常见陷阱与限制

- **构造顺序**：先加 source 再加 inventoryitem，会在读取 `inventoryitem.owner` 时索引 nil；原版始终先加 inventoryitem。[owner 读取](../../../raw/current/scripts/componentutil.lua#L1833-L1839)
- **持有者必须已有 moisture**：给没有 moisture 的生物或容器持有时不会添加 user；之后补 moisture 也不会补挂，需让物品重新进出背包触发。
- **只有最终持有者生效**：物品在箱子等未持有容器内时 grand owner 可能是容器或 nil，两边都不成立；在未穿在身上的背包里则可能挂上 user 却扫不到物品（见 user 页扫描范围）。
- **组件不钳制数值**：`GetDryingRate` 原样返回回调值（仅雨天乘 0.07），`ApplyDrying` 也不检查剩余可吸量；上限与“吸满返回 0”必须由宿主实现，否则会无限充能。
- **速率参数易误解**：`ApplyDrying(rate, dt)` 的 `rate` 是持有者总速率、可能为负，不能当成本物品实际吸湿量。
- **多个源只选一个**：持有者只对速率最高的源调用 `ApplyDrying`；同速取遍历先遇到者，顺序不稳定。
- **强制干燥期间不结算**：`moisture:ForceDry`（含 [[reference/components/moistureimmunity]]）生效时更新提前返回，源不会被调用，也不会充能。[moisture OnUpdate](../../../raw/current/scripts/components/moisture.lua#L338-L343)

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 处理范围：锚点重定位 7 处（componentutil +231、prefablist +16、recipes +5、inventoryitem +4，区间文本逐段比对一致）；结论修订：无（MAX_WETNESS 未变）。
- 依据：tuning 键级 hunk 地图＋变更清单命中 componentutil.lua、prefablist.lua、recipes.lua、components/inventoryitem.lua、tuning.lua。

- 源码审阅：2026-10-21，`steam-buildid:25643519`。逐行读完 `components/moistureabsorbersource.lua`（53 行）、`moistureabsorberuser.lua`（46 行）、`prefabs/desiccant.lua`（182 行）、`components/inventoryitemmoisture.lua`（265 行）；定向核对 `componentutil.lua:1533-1611`、`inventoryitem.lua:344-361`、`inventoryitem.lua:494-502`、`moisture.lua:329-372`、`tuning.lua:88`、`tuning.lua:9103-9108`、`prefablist.lua:349`、`recipes.lua:981-982`。覆盖构造、绑定、回调、原版宿主与清理；未覆盖其他模组宿主与游戏内数值。
- 游戏运行：未运行。
- 结构检查：`validate_page.py` 0 error／0 warning；`validate_source_links.py --strict` 0 error／0 warning（23 条源码引用，23 条行锚点）；`parser_probe --syntax-only` 通过。
