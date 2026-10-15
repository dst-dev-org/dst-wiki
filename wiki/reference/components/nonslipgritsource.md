---
title: "nonslipgritsource"
created: "2026-10-21"
updated: "2026-10-28"
type: component
summary: "让防滑砂物品在入包时向持有者注册防滑来源，并按跑动时间消耗燃料；强化版改为原地铺出防滑池。"
aliases: ["nonslipgritsource", "防滑砂", "防滑砂来源", "nonslipgrit", "nonslipgritboosted"]
tags: [component, item, callback, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/nonslipgritsource.lua", "components/nonslipgrituser.lua", "components/slipperyfeet.lua", "components/nonslipgritpool.lua", "prefabs/nonslipgrit.lua", "componentutil.lua", "prefabs/player_common.lua", "util/sourcemodifierlist.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# nonslipgritsource

## 快速判断

- 职责与适用：挂在防滑砂物品（`nonslipgrit`／`nonslipgritboosted`）上，物品入包后把自身登记为持有者的防滑来源；真正跳过打滑累计的是持有者的 `nonslipgrituser` 与地面 `nonslipgritpool`，本组件只负责登记与消耗。[nonslipgritsource.lua:20-23](../../../raw/current/scripts/components/nonslipgritsource.lua#L20-L23)
- 优先使用：原版两件物品已完整接线；自定义防滑物品复用本组件，用 `SetOnDeltaFn` 定义跑动时消耗什么，不要另写一套 source／user。[nonslipgrit.lua:69](../../../raw/current/scripts/prefabs/nonslipgrit.lua#L69
- 重要副作用：普通砂在持有者于可打滑地面上跑动时按 `dt` 扣燃料，并给**物品自身**挂 0.25 秒 `nonslipgrit_buff`（除 prefab 自身外无读取方）；强化砂每次回调生成 1 个半径 3 的防滑池并扣 4 燃料，不挂 buff。[nonslipgrit.lua:17-19](../../../raw/current/scripts/prefabs/nonslipgrit.lua#L17-L19) [nonslipgrit.lua:24-29](../../../raw/current/scripts/prefabs/nonslipgrit.lua#L24-L29)
- 修改边界：仅在服务端添加，客户端没有该组件；只对 `components.slipperyfeet` 存在的持有者注册，否则拾取后完全无效果；无 OnSave，读档靠物品重新入包触发注册。[nonslipgrit.lua:54-55](../../../raw/current/scripts/prefabs/nonslipgrit.lua#L54-L55)

## 关键接口

| 接口／字段 | 签名或约定 | 条件、作用与证据 |
|---|---|---|
| 构造 `NonSlipGritSource(inst)` | 无参数，组件挂在物品实体 | 构造即调用 `MakeComponentAnInventoryItemSource`，监听 `onputininventory`／`ondropped`／`onremove`；若构造时物品已有 owner 会立刻注册。[nonslipgritsource.lua:1-6](../../../raw/current/scripts/components/nonslipgritsource.lua#L1-L6) |
| `OnItemSourceNewOwner(owner)` | `owner` 为最终持有者 | 仅当 `owner.components.slipperyfeet` 存在才 `AddComponent("nonslipgrituser")`（没有则新建）并 `AddSource(self.inst)`；不补加 slipperyfeet，也不校验 owner 类型。[nonslipgritsource.lua:20-23](../../../raw/current/scripts/components/nonslipgritsource.lua#L20-L23) |
| `OnItemSourceRemoved(owner)` | 同上 | 调 `user:RemoveSource(self.inst)`；最后一个来源移除时 user 自删。[nonslipgritsource.lua:13-16](../../../raw/current/scripts/components/nonslipgritsource.lua#L13-L16) |
| `SetOnDeltaFn(fn)` | `fn(inst, dt)`，单槽覆盖 | 只由 user 的 `DoDelta` 调用；nil 时砂不消耗也不产生效果。[nonslipgritsource.lua:27-28](../../../raw/current/scripts/components/nonslipgritsource.lua#L27-L28) |
| `DoDelta(dt)` | `dt` 秒 | 直接转发给回调，不判断燃料、不负责移除物品；耗尽由 `fueled:SetDepletedFn(inst.Remove)` 处理。[nonslipgritsource.lua:31-33](../../../raw/current/scripts/components/nonslipgritsource.lua#L31-L33) [nonslipgrit.lua:63-67](../../../raw/current/scripts/prefabs/nonslipgrit.lua#L63-L67) |
| `OnRemoveFromEntity()` | 无参 | 注销三个物品事件并补偿一次 `OnItemSourceRemoved`，避免持有者残留来源。[nonslipgritsource.lua:9-10](../../../raw/current/scripts/components/nonslipgritsource.lua#L9-L10) [componentutil.lua:1842-1849](../../../raw/current/scripts/componentutil.lua#L1842-L1849) |
| `itemsource_owner`／`itemsource_container` | 内部字段 | 由 helper 维护；容器易主时用 `inventoryitem:GetGrandOwner()` 重算 owner，再回调本组件。[componentutil.lua:1810](../../../raw/current/scripts/componentutil.lua#L1810 [componentutil.lua:1815](../../../raw/current/scripts/componentutil.lua#L1815 |

### 注册、撤销与三层作用域

1. **物品层（source）**：每件砂一个组件，服务端 prefab 添加、随物品存亡；入包事件把它接到持有者。[nonslipgrit.lua:54-55](../../../raw/current/scripts/prefabs/nonslipgrit.lua#L54-L55)
2. **持有者层（user）**：每个持有者最多一个布尔来源表，由 source 在 `slipperyfeet` 存在时动态添加；不在任何 prefab 中显式注册。[nonslipgritsource.lua:20-23](../../../raw/current/scripts/components/nonslipgritsource.lua#L20-L23) [nonslipgrituser.lua:6](../../../raw/current/scripts/components/nonslipgrituser.lua#L6
3. **地面层（pool）**：只有强化砂生成；独立实体，半径 3、存活 2 段（60 秒），任何走到范围内的实体都受益。[nonslipgrit.lua:96-99](../../../raw/current/scripts/prefabs/nonslipgrit.lua#L96-L99) [tuning.lua:9116-9118](../../../raw/current/scripts/tuning.lua#L9116-L9118)
4. 注册时机：物品 `onputininventory` → `itemsource_topocket` → grand owner；容器易主、丢弃、移除分别重算或撤销。[inventoryitem.lua:348-360](../../../raw/current/scripts/components/inventoryitem.lua#L348-L360) [componentutil.lua:1777](../../../raw/current/scripts/componentutil.lua#L1777
5. 撤销：离手／被移除 → `OnItemSourceRemoved` → `user:RemoveSource`；source 自身移除时先注销监听。[componentutil.lua:1842-1849](../../../raw/current/scripts/componentutil.lua#L1842-L1849)

### 消耗链与 locomotor 的关系

- source 自己不计时、没有 OnUpdate；`DoDelta` 只由 user 从 `slipperyfeet:OnUpdate` 的 accumulate 分支按帧调用。[slipperyfeet.lua:245-248](../../../raw/current/scripts/components/slipperyfeet.lua#L245-L248)
- accumulate 只在 slipperyfeet 已有打滑来源时启动，且限 `running`／`spinning` 且无 `noslip` 标签；站着、未跑、骑乘、重物、狼人、谨慎行走都不消耗也不会触发防滑。[slipperyfeet.lua:117-118](../../../raw/current/scripts/components/slipperyfeet.lua#L117-L118) [slipperyfeet.lua:92-94](../../../raw/current/scripts/components/slipperyfeet.lua#L92-L94)
- 有 user 或站在 pool 上就跳过打滑累计；有 user 且不在 pool 上才调用 `user:DoDelta(dt)`。[slipperyfeet.lua:245-248](../../../raw/current/scripts/components/slipperyfeet.lua#L245-L248)
- `slipperyfeet` 用 `Physics:GetMotorSpeed()` 估累计速度，slip 阈值是 `WILSON_RUN_SPEED*4`；locomotor 源码没有引用本组件，防滑不修改任何 speed multiplier。[slipperyfeet.lua:171-176](../../../raw/current/scripts/components/slipperyfeet.lua#L171-L176) [slipperyfeet.lua:149-155](../../../raw/current/scripts/components/slipperyfeet.lua#L149-L155)
- 触发链：超过阈值 → `feetslipped` → SGwilson 在 running 且非 noslip 时进入 slip，`locomotor:Stop()` 并清零 slipperiness。[SGwilson.lua:2062-2064](../../../raw/current/scripts/stategraphs/SGwilson.lua#L2062-L2064) [SGwilson.lua:24292](../../../raw/current/scripts/stategraphs/SGwilson.lua#L24292 [SGwilson.lua:24296](../../../raw/current/scripts/stategraphs/SGwilson.lua#L24296

## 典型接入

### 原版宿主：`nonslipgrit` 与 `nonslipgritboosted`

- 文件与注册：`prefabs/nonslipgrit.lua`；`Prefab("nonslipgrit", fn, assets, prefabs)` 与 `Prefab("nonslipgritboosted", fn_boosted, assets, prefabs_boosted)`，两者共用 `fn_common`，在 mastersim 检查之后 `AddComponent("nonslipgritsource")` 并分别设 `OnDelta`／`OnDelta_Boosted`。[nonslipgrit.lua:54-55](../../../raw/current/scripts/prefabs/nonslipgrit.lua#L54-L55) [nonslipgrit.lua:69](../../../raw/current/scripts/prefabs/nonslipgrit.lua#L69 [nonslipgrit.lua:377-378](../../../raw/current/scripts/prefabs/nonslipgrit.lua#L377-L378)
- 单位与节奏：fueled 上限 100、`rate=0`；普通砂 6 段＝180 秒（仅在可打滑地面跑动时消耗，每秒 100/180 燃料），每次回调重挂 0.25 秒 buff；强化砂每次回调扣 4 并铺 1 池，设计 25 池、池各 60 秒。[nonslipgrit.lua:63-67](../../../raw/current/scripts/prefabs/nonslipgrit.lua#L63-L67) [nonslipgrit.lua:160](../../../raw/current/scripts/prefabs/nonslipgrit.lua#L160 [tuning.lua:9116-9118](../../../raw/current/scripts/tuning.lua#L9116-L9118)
- 位置与生效：强化砂的池生成在物品世界坐标；物品在包内以 0,0,0 挂在持有者下，因此就是持有者脚下。入包即注册，离手／耗尽即撤销。[inventoryitem.lua:355-356](../../../raw/current/scripts/components/inventoryitem.lua#L355-L356) [nonslipgrit.lua:24-29](../../../raw/current/scripts/prefabs/nonslipgrit.lua#L24-L29)
- 获取：Shell Weaver 配方 `shellweaver_nonslipgrit`／`shellweaver_nonslipgritboosted`。[recipes.lua:983-984](../../../raw/current/scripts/recipes.lua#L983-L984)

```lua
-- modmain.lua：给自定义物品补上与 nonslipgrit 相同的来源接线
AddPrefabPostInit("my_grit_item", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    local source = inst:AddComponent("nonslipgritsource")
    source:SetOnDeltaFn(function(item, dt)
        item.components.fueled:DoDelta(-dt * 100 / GLOBAL.TUNING.NONSLIPGRIT_TOTAL_USE_TIME)
    end)
end)
```

- 替换与验证：`my_grit_item` 换成目标物品 prefab，且该物品必须已有 `inventoryitem`、`fueled` 和服务端构造；100／180 沿用原版燃料点每秒公式，`dt` 单位为秒。只为"跑动时不打滑"接线时，持有者还必须已有 `slipperyfeet`，否则不会注册。

## 常见陷阱与限制

- **给没有 `slipperyfeet` 的生物用**：`OnItemSourceNewOwner` 直接跳过，砂在手里也完全没有 user；不能由"加了 source"推出"会防滑"。[nonslipgritsource.lua:20-23](../../../raw/current/scripts/components/nonslipgritsource.lua#L20-L23)
- **把 `nonslipgrit_buff` 当玩家 buff**：它挂在 source 物品自己身上（`AddDebuff` 会自动补 `debuffable`），全库除 prefab 外无读取方；防滑判定是 owner 上是否存在 `nonslipgrituser`。[nonslipgrit.lua:17-19](../../../raw/current/scripts/prefabs/nonslipgrit.lua#L17-L19) [entityscript.lua:2140-2142](../../../raw/current/scripts/entityscript.lua#L2140-L2142)
- **随身容器内的砂防滑但不消耗**：helper 把背包等可携带容器的持有者当 owner，砂放进背包仍会注册 user；而 user 的 `DoDelta` 用 `Inventory:FindItem` 只扫 itemslots／activeitem／overflow，不遍历容器内容，于是防滑却扣不到燃料（源码推导，未运行）。[componentutil.lua:1810](../../../raw/current/scripts/componentutil.lua#L1810 [inventory.lua:634-635](../../../raw/current/scripts/components/inventory.lua#L634-L635)
- **多块砂叠加**：user 只取 `FindItem` 的第一个 source 调 `DoDelta`，`_sources` 仅用于存在性；`pairs` 顺序不稳定，多块砂不会更快消耗，也无法指定哪块被用。[nonslipgrituser.lua:20-21](../../../raw/current/scripts/components/nonslipgrituser.lua#L20-L21) [nonslipgrituser.lua:24-29](../../../raw/current/scripts/components/nonslipgrituser.lua#L24-L29)
- **强化砂按帧铺池**：`OnDelta_Boosted` 每次调用扣 4 燃料并生成 1 池，调用频率跟随 `slipperyfeet` 更新而非每秒一次；25 池通常在一次短跑内铺完并耗尽物品（源码推导，未运行，实际节奏需实测）。[nonslipgrit.lua:24-29](../../../raw/current/scripts/prefabs/nonslipgrit.lua#L24-L29) [slipperyfeet.lua:245-248](../../../raw/current/scripts/components/slipperyfeet.lua#L245-L248)
- **撤掉 `slipperyfeet` 后残留 user**：`slipperyfeet:OnRemoveFromEntity` 不清理 `nonslipgrituser`；重新加回后旧 user 立即生效，可能已没有对应物品。[slipperyfeet.lua:78-90](../../../raw/current/scripts/components/slipperyfeet.lua#L78-L90)
- **把 pool 当永久防滑地面**：60 秒后 timer 移除，且只有强化砂产生；普通砂必须留在背包才有效。[nonslipgrit.lua:139-141](../../../raw/current/scripts/prefabs/nonslipgrit.lua#L139-L141)
- **端与存档**：source 只在 mastersim 添加，客户端不能查询；user 无 OnSave／OnLoad，读档依赖 `Inventory:OnLoad` → `GiveItem` → `OnPutInInventory` 重新发事件。[inventory.lua:293-301](../../../raw/current/scripts/components/inventory.lua#L293-L301) [inventory.lua:1184](../../../raw/current/scripts/components/inventory.lua#L1184 [inventoryitem.lua:348-360](../../../raw/current/scripts/components/inventoryitem.lua#L348-L360)

## 验证记录

- 版本增量复核 26148036（2026-10-25）：锚点重定位 14 处（componentutil 来源 helper 段随 +231 大改平移至 1764-1842、inventoryitem OnPutInInventory 段 +4、entityscript AddDebuff +3、tuning NONSLIPGRIT 段 +17、recipes Shell Weaver +5、SGwilson feetslipped +49／slip 状态 +166，显示标签同步；slip 状态偏移与 player-hubs 分段表不符，已按新文件 rg＋逐行内容核对修正）；无结论修订——NONSLIPGRIT_TOTAL_USE_TIME=6 段／POOL_TIME=2 段值未变，nonslipgrit 族组件与 slipperyfeet 未变。依据：组内共享 componentutil diff＋tuning／player-hubs 地图（SGwilson slip 段以实测为准）＋变更清单命中 inventoryitem.lua（GetGrandOwner 重写语义等价，C-19）等。
- 源码审阅：2026-10-21，Steam buildid 25643519；完整读 `components/nonslipgritsource.lua`、`nonslipgrituser.lua`、`nonslipgritpool.lua`、`components/slipperyfeet.lua`、`prefabs/nonslipgrit.lua`；定向核验 componentutil 的 inventory item source 注册、slipperyfeet 消费链、SGwilson slip 分支、player_common／wx78_possessedbody 宿主及 tuning／recipe 数值。
- 结构与语法：本页 validate_page.py（12 字段与大小）通过；validate_source_links.py --strict 通过；文中 modmain 片段经 `parser_probe --syntax-only` 通过，仅验证 Lua 5.1 语法。
- 游戏运行：未运行；容器内注册、强化砂铺池节奏、多块砂消耗顺序、读档重注册待实测；未审阅其他防滑物品或模组改动。本页共 55 处源码引用。

相关：[[reference/components/locomotor]]；[[reference/components/inventoryitem]]；[[reference/components/fueled]]；[[reference/components/nonslipgrituser]]。
