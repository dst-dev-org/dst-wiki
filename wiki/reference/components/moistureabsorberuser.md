---
title: "moistureabsorberuser"
created: "2026-10-21"
updated: "2026-10-28"
type: component
summary: "挂在持有者上汇总吸湿源，每帧选出最优速率并让 moisture 扣除，最后一个源离开时自删。"
aliases: ["吸湿者", "持有吸湿源", "吸湿汇总", "moistureabsorberuser"]
tags: [component, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/moistureabsorberuser.lua", "components/moistureabsorbersource.lua", "components/moisture.lua", "components/inventory.lua", "components/container.lua", "components/inventoryitem.lua", "util/sourcemodifierlist.lua", "prefabs/desiccant.lua", "tuning.lua"]
reliability: "verified"
validation: "source-reviewed"
status: "active"
---

# moistureabsorberuser

## 快速判断

- 职责与适用：挂在**持有者**（有 moisture 与 inventory 的实体）上的汇总器，保存来源集合并向 moisture 提供“当前最优吸湿速率”。由物品侧 [[reference/components/moistureabsorbersource]] 自动添加，不给物品挂。[构造](../../../raw/current/scripts/components/moistureabsorberuser.lua#L3-L11)
- 优先使用：接入新除湿物品时只实现 source 组件；把 desiccant 放进物品栏、装备或穿着的背包即生效，玩家侧无需代码。不要手动对玩家 `AddComponent` 或 `AddSource`。[自动挂载](../../../raw/current/scripts/components/moistureabsorbersource.lua#L20-L25)
- 重要副作用：每次结算完整扫描持有者 inventory，只对速率最高的一个源调用 `ApplyDrying`；最后一个来源撤销时自动 `RemoveComponent("moistureabsorberuser")`。[扫描](../../../raw/current/scripts/components/moistureabsorberuser.lua#L20-L44) [自删](../../../raw/current/scripts/components/moistureabsorberuser.lua#L13-L18)
- 修改边界：服务端组件，无 replica、无自有事件、不存档；没有 inventory 时返回 0；只在 moisture 的更新路径内被调用，不独立计时。[消费者](../../../raw/current/scripts/components/moisture.lua#L329-L336)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `AddSource(src, key?)` | EntityScript 来源，可选 key；无返回值 | 写入 `SourceModifierList`（boolean，base=false）；同一实体重复添加只保留一份，不累加。[AddSource](../../../raw/current/scripts/components/moistureabsorberuser.lua#L9-L11) [集合类型](../../../raw/current/scripts/util/sourcemodifierlist.lua#L33-L35) |
| `RemoveSource(src, key?)` | 与添加相同的实体／key | 移除该来源；`_sources:Get()` 为 false 时删除组件。source 组件调用时不带 key，走默认 key。[RemoveSource](../../../raw/current/scripts/components/moistureabsorberuser.lua#L13-L18) |
| `GetBestAbsorberRate(rate, dt)` | number，湿度点／秒 | 无 inventory 返回 0；遍历物品，对带 source 的物品取 `GetDryingRate(rate)`，严格大于当前最优才替换；只对最优物品调用 `ApplyDrying(rate, dt)` 并返回其速率。[选择](../../../raw/current/scripts/components/moistureabsorberuser.lua#L30-L44) |
| `_sources` | `SourceModifierList(inst, false, SourceModifierList.boolean)` | 内部集合，只用于判断“是否还有来源”；不要直接改写。[字段](../../../raw/current/scripts/components/moistureabsorberuser.lua#L3-L7) |

### 每帧结算顺序

1. `moisture:OnUpdate` 先算环境、装备、自然干燥与外部 bonus 的 `rate`；强制干燥时整段跳过。[moisture.lua:338-355](../../../raw/current/scripts/components/moisture.lua#L338-L355)
2. 仅当 `moisture > 0` 或 `rate > 0` 时调用 `GetDesiccantBonus(rate, dt)`；没有 user 组件则直接返回 0。[moisture.lua:357-360](../../../raw/current/scripts/components/moisture.lua#L357-L360)
3. user 选源并返回其速率；moisture 用 `rate - drate` 后统一 `DoDelta(rate * dt)`，该吸收项绕过自然干燥的钳制与下雨抑制。[moisture.lua:359-371](../../../raw/current/scripts/components/moisture.lua#L359-L371)
4. 源自身充能由 `ApplyDrying` 实现，与返回值相互独立，因此“角色被扣多少”和“物品吸多少”可以不一致。

## 典型接入

### 原版 desiccant 的携带链路

- 端与入口：服务端物品 prefab `desiccant`／`desiccantboosted` 在 fn 内加 source；玩家侧没有手写逻辑，拾取即生效。[宿主](../../../raw/current/scripts/prefabs/desiccant.lua#L156-L163)
- 增加：拾取、装备、放进容器都会经 inventoryitem 推 `onputininventory`；source 组件据此算出 grand owner，若其有 moisture 则添加本组件并 `AddSource`。[事件](../../../raw/current/scripts/components/inventoryitem.lua#L348-L365)
- 撤销：丢地推 `ondropped`、销毁推 `onremove`、或 source 组件被移除，都走 `OnItemSourceRemoved → RemoveSource`；最后一个来源离开时删组件。[移除路径](../../../raw/current/scripts/componentutil.lua#L1805-L1832)
- 单位与数值：desiccant 0.5、强化 0.75 湿度点／秒；雨天 `rate > 0` 时对持有者只按 0.07 折算（0.035／0.0525），而 `ApplyDrying` 仍按 0.5／0.75 给物品充能，所以雨天物品先满、角色变干很慢。[雨天效率](../../../raw/current/scripts/components/moistureabsorbersource.lua#L31-L41) [充能](../../../raw/current/scripts/prefabs/desiccant.lua#L24-L38)
- 强制干燥：`moisture:ForceDry` 生效（如 [[reference/components/moistureimmunity]] 授予）时第 1 步就返回，本组件不会被调用，物品也不充能。[门禁](../../../raw/current/scripts/components/moisture.lua#L338-L343)

### 扫描范围决定“携带”是否成立

- 计入：`inventory.itemslots`、`equipslots`、`activeitem`，以及 `GetOverflowContainer()` 返回的、穿在 BODY 且可打开的容器内容（例如背包）。[ForEachItem](../../../raw/current/scripts/components/inventory.lua#L667-L684) [overflow](../../../raw/current/scripts/components/inventory.lua#L1459-L1467)
- 不计入：未被持有的箱子、未穿在身上的背包，以及源码注释点名的 Wilson 胡须；这些位置即使 user 已挂在玩家身上，也扫不到物品。[注释](../../../raw/current/scripts/components/inventory.lua#L1456-L1458)
- 两侧判定不同：source 按 grand owner 决定挂不挂 user，user 按自己的 inventory 扫描决定找不找得到；容器嵌套时可能只满足一边（箱子不挂、未穿背包挂上却扫不到）。

## 常见陷阱与限制

- **手动 AddSource 不产生吸湿**：user 只扫描带 `moistureabsorbersource` 的物品，干燥回调也只由该物品提供；只给玩家加来源集合不会让任何物品参与。[扫描条件](../../../raw/current/scripts/components/moistureabsorberuser.lua#L20-L28)
- **重复添加不加倍**：同一实体重复 `AddSource` 只保留一份；要叠加需换成不同来源实体。
- **只选一个且同速不稳定**：严格大于才替换，同速率取遍历先遇到者；多个吸湿物品不会同时充能，吸满返回 0 后才轮到下一个。
- **组件存在 ≠ 生效**：物品在扫描范围外、或持有者被强制干燥时，组件可能存在但不结算。
- **每帧全背包扫描**：有吸湿物品时，moisture 每次实际结算（`moisture > 0` 或 `rate > 0`）都遍历，成本随背包物品数增长；最后一个源离开后自删，空扫随之停止。
- **无客户端表现**：没有 replica、事件或存档，客户端不能查询本组件，UI 需另行同步。
- **直接删组件会留下未登记的源**：仍在背包的 source 不会重新 AddSource，直到物品再次进出背包；应通过 `RemoveSource` 或让物品离开来撤销。[重新挂载条件](../../../raw/current/scripts/componentutil.lua#L1810-L1818)

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 处理范围：锚点重定位 3 处（componentutil +231、inventoryitem +4，区间文本逐段比对一致）；结论修订：无。
- 依据：tuning 键级 hunk 地图＋变更清单命中 componentutil.lua、components/inventoryitem.lua、tuning.lua。

- 源码审阅：2026-10-21，`steam-buildid:25643519`。逐行读完 `components/moistureabsorberuser.lua`（46 行）、`moistureabsorbersource.lua`（53 行）、`prefabs/desiccant.lua`（182 行）、`components/inventoryitemmoisture.lua`（265 行）；定向核对 `moisture.lua:329-372`、`inventory.lua:667-684`、`inventory.lua:1456-1467`、`container.lua:783-787`、`inventoryitem.lua:344-361`、`componentutil.lua:1533-1611`、`util/sourcemodifierlist.lua:31-122`、`tuning.lua:88`、`tuning.lua:9103-9108`。覆盖挂载、扫描、选择、雨天折算、清理与强制干燥门禁；未覆盖其他模组宿主与游戏内实测。
- 游戏运行：未运行。
- 结构检查：`validate_page.py` 0 error／0 warning；`validate_source_links.py --strict` 0 error／0 warning（24 条源码引用，24 条行锚点）；本页无完整 Lua，未做语法检查。
