---
title: "nonslipgrituser"
created: "2026-10-21"
updated: "2026-10-28"
type: component
summary: "记录持有者是否带着防滑砂，并把 slipperyfeet 的跑动更新转交给背包中的一个 nonslipgritsource 消耗。"
aliases: ["nonslipgrituser", "防滑砂持有者", "防滑来源集合", "防滑状态"]
tags: [component, callback, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/nonslipgrituser.lua", "components/nonslipgritsource.lua", "components/slipperyfeet.lua", "componentutil.lua", "prefabs/player_common.lua", "prefabs/player_common_extensions.lua", "prefabs/wx78_possessedbody.lua", "stategraphs/SGwilson.lua", "util/sourcemodifierlist.lua", "components/inventory.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# nonslipgrituser

## 快速判断

- 职责与适用：持有者侧的布尔"防滑来源集合"；只回答"身上是否有防滑砂"，并把防滑更新转交给一个 source 物品消耗，不订阅事件、不计时、不保存、不改移动速度。[nonslipgrituser.lua:1-7](../../../raw/current/scripts/components/nonslipgrituser.lua#L1-L7)
- 优先使用：让 [[reference/components/nonslipgritsource]] 自动增删；自定义防滑物品复用该 source 组件，不要手写 AddSource／RemoveSource 配对，容易漏撤销。[nonslipgritsource.lua:20-23](../../../raw/current/scripts/components/nonslipgritsource.lua#L20-L23)
- 重要副作用：组件一旦存在，`slipperyfeet` 在跑动累计阶段跳过打滑，并在未站上池时把每帧 `dt` 转给背包里**一个** source 消耗；空组件同样生效。[slipperyfeet.lua:245-248](../../../raw/current/scripts/components/slipperyfeet.lua#L245-L248)
- 修改边界：服务端；由 source 在 `owner.components.slipperyfeet` 存在时动态添加；没有自己的 prefab 注册、没有客户端副本、没有 OnSave／OnLoad。[nonslipgritsource.lua:22](../../../raw/current/scripts/components/nonslipgritsource.lua#L22

## 关键接口

| 接口／字段 | 签名或约定 | 条件、作用与证据 |
|---|---|---|
| 构造 `NonSlipGritUser(inst)` | 无参数，组件挂在持有者实体 | 建 `SourceModifierList(inst, false, SourceModifierList.boolean)`，初值 false；不监听事件。[nonslipgrituser.lua:3-7](../../../raw/current/scripts/components/nonslipgrituser.lua#L3-L7) |
| `AddSource(src, key?)` | `src` 物品实体；`key` 可选 | `SetModifier(src, true, key)`；`src` 为实体时来源列表会监听其 `onremove` 自动移除，重复添加替换同 key。[nonslipgrituser.lua:9-10](../../../raw/current/scripts/components/nonslipgrituser.lua#L9-L10) [sourcemodifierlist.lua:84-91](../../../raw/current/scripts/util/sourcemodifierlist.lua#L84-L91) |
| `RemoveSource(src, key?)` | 原实体与可选 key | `RemoveModifier` 后若 `_sources:Get()` 为 false，立即 `RemoveComponent("nonslipgrituser")`；最后一个来源撤销即自删。[nonslipgrituser.lua:13-16](../../../raw/current/scripts/components/nonslipgrituser.lua#L13-L16) |
| `DoDelta(dt)` | `dt` 秒 | 有 inventory 时用 `FindItem(IsValidGritSource)` 找第一个带 source 的物品并调其 `DoDelta(dt)`；找不到就空转，不报错。[nonslipgrituser.lua:24-29](../../../raw/current/scripts/components/nonslipgrituser.lua#L24-L29) |
| `_sources` | `SourceModifierList`（boolean） | 只用于存在性与自删判断；`slipperyfeet` 不读它的值，只看 `components.nonslipgrituser` 是否存在。[nonslipgrituser.lua:6](../../../raw/current/scripts/components/nonslipgrituser.lua#L6 |
| 判定谓词 `IsValidGritSource(item)` | 返回 `item.components.nonslipgritsource ~= nil` | 内部函数，只按组件类型筛选，不校验该物品是否属于当前注册集合。[nonslipgrituser.lua:20-21](../../../raw/current/scripts/components/nonslipgrituser.lua#L20-L21) |

### 注册与撤销

- **谁添加**：只有 `nonslipgritsource:OnItemSourceNewOwner`；它先检查持有者的 `slipperyfeet`，没有就什么都不做，有才新建或复用 user 并 `AddSource`。[nonslipgritsource.lua:20-23](../../../raw/current/scripts/components/nonslipgritsource.lua#L20-L23)
- **何时添加**：物品拾取入包时经 `onputininventory` 到 `itemsource_topocket`；放入容器后容器易主也会重算，认的是 `inventoryitem:GetGrandOwner()`。[componentutil.lua:1805-1817](../../../raw/current/scripts/componentutil.lua#L1805-L1817)
- **何时撤销**：物品丢弃或移除时 `OnItemSourceRemoved` → `RemoveSource`，来源表空则自删；source 自身移除会先注销监听并补偿一次撤销。[nonslipgritsource.lua:9-10](../../../raw/current/scripts/components/nonslipgritsource.lua#L9-L10) [componentutil.lua:1842-1849](../../../raw/current/scripts/componentutil.lua#L1842-L1849)
- **读档**：组件没有 OnSave／OnLoad；`Inventory:OnLoad` 逐项 `GiveItem`，后者调用 `OnPutInInventory` 重新发事件，从而在持有者已有 `slipperyfeet` 时重注册。[inventory.lua:293-301](../../../raw/current/scripts/components/inventory.lua#L293-L301) [inventory.lua:1184](../../../raw/current/scripts/components/inventory.lua#L1184 [inventoryitem.lua:348-360](../../../raw/current/scripts/components/inventoryitem.lua#L348-L360)
- **层级**：每个持有者最多一个 user，可同时登记多件砂，但消费只作用于其中一件；它是"随身、全图"的保护，与地面 pool 的半径 3 保护是两层。[slipperyfeet.lua:245-248](../../../raw/current/scripts/components/slipperyfeet.lua#L245-L248)

### 与 locomotor／地面效果的消费关系

- user 不引用 locomotor；保护发生在 `slipperyfeet` 的 accumulate 分支，只有打滑来源已激活、状态带 `running`／`spinning` 且无 `noslip` 时才进入。[slipperyfeet.lua:117-118](../../../raw/current/scripts/components/slipperyfeet.lua#L117-L118) [slipperyfeet.lua:92-94](../../../raw/current/scripts/components/slipperyfeet.lua#L92-L94)
- 判定顺序：先查半径 12 内带 `nonslipgritpool` 标签的实体并在半径 3 内命中，再查 `components.nonslipgrituser`；有 pool 时不扣燃料，只有 user 且不在池上才调 `DoDelta`。[slipperyfeet.lua:237-241](../../../raw/current/scripts/components/slipperyfeet.lua#L237-L241) [slipperyfeet.lua:245-248](../../../raw/current/scripts/components/slipperyfeet.lua#L245-L248)
- 已有 slipperiness 不会因 user 立刻归零：仍按 decay 分支衰减；只是不再新增累计。[slipperyfeet.lua:250-251](../../../raw/current/scripts/components/slipperyfeet.lua#L250-L251)
- 速度只通过 `Physics:GetMotorSpeed()` 进入累计速率；超过阈值发 `feetslipped`，SG 切 slip 时调 `locomotor:Stop()`。因此防滑的效果是不进入 slip、不再新增累计，而不是加速、减速或修改移动倍率。[slipperyfeet.lua:171-176](../../../raw/current/scripts/components/slipperyfeet.lua#L171-L176) [SGwilson.lua:2062-2064](../../../raw/current/scripts/stategraphs/SGwilson.lua#L2062-L2064) [SGwilson.lua:24292](../../../raw/current/scripts/stategraphs/SGwilson.lua#L24292

## 典型接入

### 原版宿主：玩家与 WX 附身躯体

- **玩家**：所有角色经 `player_common` 的服务端构造添加 `slipperyfeet`，因此都有 user 入口；`nonslipgrituser` 仍不在任何 prefab 里，砂入包时才动态出现。[player_common.lua:2827](../../../raw/current/scripts/prefabs/player_common.lua#L2827 [nonslipgritsource.lua:22](../../../raw/current/scripts/components/nonslipgritsource.lua#L22
- **复活流程**：`CommonActualRez` 再次 `AddComponent("slipperyfeet")`；user 是否重新出现仍取决于物品再次入包，不依赖该流程。[player_common_extensions.lua:298](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L298
- **WX 附身躯体**：`wx78_possessedbody` 服务端构造添加 `slipperyfeet`，所以它也能成为 user 宿主。[wx78_possessedbody.lua:661](../../../raw/current/scripts/prefabs/wx78_possessedbody.lua#L661 [wx78_possessedbody.lua:569-571](../../../raw/current/scripts/prefabs/wx78_possessedbody.lua#L569-L571)
- **注册名**：本组件没有自己的 Prefab；宿主就是这些 owner prefab（玩家 `wilson` 等经 `player_common` 构建、`wx78_possessedbody`），组件名 `nonslipgrituser` 仅在运行期 `AddComponent` 中使用。
- **生效与撤销**：拾取砂 → 注册；砂离手／被移除／燃料耗尽（物品 Remove）→ 最后一个来源撤销时组件删除。无需自定义代码，也没有存档字段。[nonslipgrit.lua:67](../../../raw/current/scripts/prefabs/nonslipgrit.lua#L67
- **扩展自定义生物**：先让它拥有 `slipperyfeet`，并在其 SG 处理 `feetslipped`／`slip`，再让物品走 source 的注册路径；只给生物加空 user 不会自动移除，反而会在它有 slipperyfeet 时永久免疫。

## 常见陷阱与限制

- **空组件也防滑**：`slipperyfeet` 只看组件是否存在，不查 `_sources:Get()`；手动 AddComponent 后不加来源会获得永久防滑，且 `DoDelta` 仍可能消耗背包里任意砂。[slipperyfeet.lua:245-248](../../../raw/current/scripts/components/slipperyfeet.lua#L245-L248) [nonslipgrituser.lua:6](../../../raw/current/scripts/components/nonslipgrituser.lua#L6
- **随身容器内的砂注册但不消耗**：helper 以 `GetGrandOwner` 把背包持有者当 owner，故背包内砂也会生成 user；`DoDelta` 的 `Inventory:FindItem` 不遍历容器内容，导致防滑但不扣燃料（源码推导，未运行）。[componentutil.lua:1810](../../../raw/current/scripts/componentutil.lua#L1810 [nonslipgrituser.lua:24-29](../../../raw/current/scripts/components/nonslipgrituser.lua#L24-L29) [inventory.lua:634-635](../../../raw/current/scripts/components/inventory.lua#L634-L635)
- **多来源只消耗一个**：`FindItem` 返回第一个，`pairs` 顺序不稳定；`_sources` 只用于存在性，不能据此做"数量越多越久"或指定消耗顺序。[nonslipgrituser.lua:24-29](../../../raw/current/scripts/components/nonslipgrituser.lua#L24-L29)
- **残留空组件**：若来源经 `SourceModifierList` 的 `onremove` 自动移除（或代码直接清空 `_sources`），组件不会自删；之后没有砂也会一直防滑。[sourcemodifierlist.lua:84-91](../../../raw/current/scripts/util/sourcemodifierlist.lua#L84-L91) [nonslipgrituser.lua:13-16](../../../raw/current/scripts/components/nonslipgrituser.lua#L13-L16)
- **移除 `slipperyfeet` 不清理 user**：`slipperyfeet:OnRemoveFromEntity` 没有处理 `nonslipgrituser`；重新加回后旧组件立即恢复生效。[slipperyfeet.lua:78-90](../../../raw/current/scripts/components/slipperyfeet.lua#L78-L90)
- **客户端没有该组件**：source 在 mastersim 之后添加，user 也只会在服务端生成；客户端 UI／预测不能查询它，需要另设复制变量。[nonslipgrit.lua:54-55](../../../raw/current/scripts/prefabs/nonslipgrit.lua#L54-L55)
- **`DoDelta` 不校验来源归属**：它只按"有 nonslipgritsource 组件"找物品，可能消耗与当前注册无关的另一块砂（同一背包内）。[nonslipgrituser.lua:20-21](../../../raw/current/scripts/components/nonslipgrituser.lua#L20-L21)
- **把 user 当计数或等级**：值是 boolean 的 OR，多件砂不增加层数、时长或强度；需要强度时改 source 的回调，而不是给 user 加 key。[sourcemodifierlist.lua:33-34](../../../raw/current/scripts/util/sourcemodifierlist.lua#L33-L34)

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 处理范围：锚点重定位 7 处（componentutil +231——其中 RemoveComponentInventoryItemSource 的 local owner 遮蔽修为等价改写、inventoryitem +4、SGwilson +49／+166、player_common −14，区间文本逐段比对一致，显示标签同步）；结论修订：无，仅锚点迁移。
- 依据：player-hubs hunk 地图＋变更清单命中 componentutil.lua、components/inventoryitem.lua、stategraphs/SGwilson.lua、prefabs/player_common.lua。

- 源码审阅：2026-10-21，Steam buildid 25643519；完整读 `components/nonslipgrituser.lua`、`nonslipgritsource.lua`、`nonslipgritpool.lua` 与 `prefabs/nonslipgrit.lua`；定向核验 componentutil 注册、SourceModifierList 语义、player_common／player_common_extensions／wx78_possessedbody 宿主、Inventory 装载与 FindItem、SGwilson slip 链。
- 结构与语法：本页 validate_page.py（12 字段与大小）通过；validate_source_links.py --strict 通过；本页无可执行 Lua 片段，未新增临时 Lua。
- 游戏运行：未运行；空组件免疫、容器内注册、读档重建、多来源消耗顺序待实测；未审阅其他防滑物品或模组改动。本页共 45 处源码引用。

相关：[[reference/components/nonslipgritsource]]；[[reference/components/locomotor]]；[[reference/components/inventoryitem]]。
