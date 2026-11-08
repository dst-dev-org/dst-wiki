---
title: "干制植物（petals_dried、petals_evil_dried、foliage_dried、succulent_picked_dried、firenettles_dried、tillweed_dried、moon_tree_blossom_dried、forgetmelots_dried）"
created: "2026-11-09"
updated: "2026-11-09"
type: prefab
summary: "prefabs/driedplants.lua 用一份工厂批量注册 8 个 `名_dried` 物品：晾肉架按 DRY_FAST 一日晾成，全部 hunger 恒 0、可作燃料与猫玩具，其中 8 种是隐士蟹茶铺三层茶配方的干叶材料。"
aliases: ["petals_dried", "petals_evil_dried", "foliage_dried", "succulent_picked_dried", "firenettles_dried", "tillweed_dried", "moon_tree_blossom_dried", "forgetmelots_dried", "干花瓣", "干深色花瓣", "干蕨叶", "干多肉", "干火荨麻叶", "干犁地草", "干月树花", "干必忘我", "干叶"]
tags: [prefab, item, food]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/driedplants.lua", "prefabs/driedplants_defs.lua", "prefabs/meatrack2.lua", "prefabs/petals.lua", "prefabs/petals_evil.lua", "prefabs/foliage.lua", "prefabs/succulent_plant.lua", "prefabs/firenettles.lua", "prefabs/tillweed.lua", "prefabs/moontree_blossom.lua", "prefabs/forgetmelots.lua", "components/dryable.lua", "components/dryingrack.lua", "components/dryer.lua", "components/edible.lua", "containers.lua", "recipes.lua", "cooking.lua", "tuning.lua", "standardcomponents.lua", "languages/chinese_s.po", "screens/redux/scrapbookdata.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 干制植物（8 件 `名_dried`）

## 快速判断

- 职责与适用：`prefabs/driedplants.lua` 用一个工厂 `MakeDriedPetal(data)` 批量生成 8 个注册，注册名一律是 `data.name.."_dried"`，差异只有 `healthvalue`／`sanityvalue`／`oneaten` 与三处 bank／build 例外。[工厂](../../../raw/current/scripts/prefabs/driedplants.lua#L9-L76)、[循环注册](../../../raw/current/scripts/prefabs/driedplants.lua#L78-L86)
- 优先使用：查"哪片叶子晾成什么"看[生产：晾肉架](#生产晾肉架)；查"这 8 件能干什么"看[消费：隐士蟹茶铺](#消费隐士蟹茶铺)；查数值差异看[注册与构造](#注册与构造)的表。
- 重要副作用：8 件**都**是 `fuel`（`SMALL_FUEL`＝15 秒）、可点燃、传播者 `flashpoint` 被覆写为 `2.5 + math.random() * 2.5`（2.5–5），且都带 `cattoy` 标签；`firenettles_dried` 是唯一带吃后减益的——非 `plantkin` 吃下会挂 `firenettle_toxin`。[主模拟段](../../../raw/current/scripts/prefabs/driedplants.lua#L63-L70)、[减益](../../../raw/current/scripts/prefabs/driedplants_defs.lua#L24-L31)
- 修改边界：晾肉架本体（`meatrack` 的容器、锤击、盐架分支）归 `prefabs/meatrack2.lua`；柜台茶配方归 `recipes.lua`、茶的效果归 [[reference/prefabs/hermitcrab-tea]] 与 `prefabs/hermitcrabtea_defs.lua`，普通锅菜（`sweettea`／`beefalotreat`）才归 `preparedfoods.lua`；本页只说"谁在什么条件下变成干叶"与"干叶被谁消耗"。

## 注册与构造

| 注册名 | 生命 | 理智 | 饥饿 | bank／build 与特殊项 |
|---|---:|---:|---:|---|
| `petals_dried` | 1（`HEALING_TINY` 默认） | 1（`SANITY_SUPERTINY`） | 0 | bank `petals`、build `flower_petals`（只改 build） |
| `petals_evil_dried` | 1 | -5（`-SANITY_TINY`） | 0 | bank／build 都是 `flower_petals_evil` |
| `foliage_dried` | 1 | 0 | 0 | 无覆写 |
| `succulent_picked_dried` | 8（`HEALING_MEDSMALL`） | 0 | 0 | 无覆写 |
| `firenettles_dried` | -3（`-HEALING_SMALL`） | -5 | 0 | `oneaten`：非 `plantkin` 加 `firenettle_toxin` |
| `tillweed_dried` | 8 | 0 | 0 | 无覆写 |
| `moon_tree_blossom_dried` | 1 | 10（`SANITY_SMALL`） | 0 | bank／build 都是 `moon_tree_petal` |
| `forgetmelots_dried` | 1 | 5（`SANITY_TINY`） | 0 | 无覆写 |

差异表见 `prefabs/driedplants_defs.lua`，里面只写五个可选项——`healthvalue`、`sanityvalue`、`oneaten`、`bank`、`build`，其余全由工厂给默认值。[差异定义](../../../raw/current/scripts/prefabs/driedplants_defs.lua#L1-L48)、[数值档位](../../../raw/current/scripts/tuning.lua#L2021-L2034)

共用 pristine（两端）：bank／build 取自 `data.bank or data.name`／`data.build or data.name`，播 `dried` 动画，`inst.pickupsound = "vegetation_grassy"`，加 `cattoy` 标签，`MakeInventoryFloatable(inst)`（默认参数），`SetPristine`。[共用前段](../../../raw/current/scripts/prefabs/driedplants.lua#L12-L40)

共用主模拟（仅服务端）：`inspectable` → `inventoryitem` → `tradable` → `edible`（`healthvalue = data.healthvalue or HEALING_TINY`、**`hungervalue = 0` 恒为零**、`sanityvalue = data.sanityvalue or 0`、`foodtype = FOODTYPE.VEGGIE`、可选 `SetOnEatenFn(data.oneaten)`）→ `stackable`（40）→ `fuel`（`SMALL_FUEL`）→ `MakeSmallBurnable(inst, SMALL_BURNTIME)` + `MakeSmallPropagator`（随后覆写 `flashpoint`）→ `MakeHauntableLaunchAndPerish`。[共用后段](../../../raw/current/scripts/prefabs/driedplants.lua#L46-L72)

### 名称核验

八件都有同名 `STRINGS.NAMES` 键：`petals_dried`＝**干花瓣**、`petals_evil_dried`＝**干深色花瓣**、`foliage_dried`＝**干蕨叶**、`succulent_picked_dried`＝**干多肉**、`firenettles_dried`＝**干火荨麻叶**、`tillweed_dried`＝**干犁地草**、`moon_tree_blossom_dried`＝**干月树花**、`forgetmelots_dried`＝**干必忘我**。[干花瓣](../../../raw/current/scripts/languages/chinese_s.po#L346824-L346827)、[深色花瓣](../../../raw/current/scripts/languages/chinese_s.po#L346834-L346837)、[蕨叶](../../../raw/current/scripts/languages/chinese_s.po#L342944-L342947)、[多肉](../../../raw/current/scripts/languages/chinese_s.po#L350109-L350112)、[火荨麻](../../../raw/current/scripts/languages/chinese_s.po#L342799-L342802)、[犁地草](../../../raw/current/scripts/languages/chinese_s.po#L350414-L350417)、[月树花](../../../raw/current/scripts/languages/chinese_s.po#L345879-L345882)、[必忘我](../../../raw/current/scripts/languages/chinese_s.po#L342959-L342962)

## 生产：晾肉架

| 步骤 | 判定与结算 | 证据 |
|---|---|---|
| 1. 放上架子 | 容器过滤要求物品带 `dryable` 标签；架子是 `MakeMeatRack("meatrack", …, 3, …)`，3 个格子、`acceptsstacks = false` | [过滤](../../../raw/current/scripts/containers.lua#L234-L244)、[注册](../../../raw/current/scripts/prefabs/meatrack2.lua#L325-L334)、[三格](../../../raw/current/scripts/prefabs/meatrack2.lua#L584-L584) |
| 2. 计时 | `dryingrack:OnGetItem` 读 `dryable:GetProduct()` 与 `GetDryTime()`，起一个 `DoTaskInTime(drytime, OnDoneDrying)`；8 件的来源全部写 `DRY_FAST`＝`total_day_time`＝480 秒，这是**有效晾晒时间**（暂停不计）；若入架时架子正处于暂停（下雨且无雨免疫），只把时长存进 `info.drytime` 不起任务，恢复后才开始跑 | [开始](../../../raw/current/scripts/components/dryingrack.lua#L236-L274)、[时长档](../../../raw/current/scripts/tuning.lua#L2055-L2056) |
| 3. 下雨 | `OnIsRaining` 的判定是「正在下雨或正在下酸雨，且架子（或其骑手）没有 `rainimmunity`」，成立才 `PauseDrying`，否则 `ResumeDrying`；酸雨另按 `ACIDRAIN_PERISHABLE_ROT_PERCENT` 加速架上物品腐坏 | [暂停判定](../../../raw/current/scripts/components/dryingrack.lua#L78-L94)、[免疫来源](../../../raw/current/scripts/components/dryingrack.lua#L318-L324)、[酸雨速率](../../../raw/current/scripts/components/dryingrack.lua#L26-L38) |
| 4. 完成 | `OnDoneDrying` 生成 `dryable:GetProduct()`、把湿度继承过去、`item:Remove()`，再 `container:GiveItem(product, slot)` 放回同一格 | [完成](../../../raw/current/scripts/components/dryingrack.lua#L198-L224) |

八个来源物品各自把产物写进 `dryable`：`petals.lua`→`petals_dried`、`petals_evil.lua`→`petals_evil_dried`、`foliage.lua`→`foliage_dried`、`succulent_plant.lua`→`succulent_picked_dried`、`firenettles.lua`→`firenettles_dried`、`tillweed.lua`→`tillweed_dried`、`moontree_blossom.lua`→`moon_tree_blossom_dried`、`forgetmelots.lua`→`forgetmelots_dried`，且都 `SetDryTime(TUNING.DRY_FAST)`、`SetBuildFile("meat_rack_food_petals")`。[花瓣](../../../raw/current/scripts/prefabs/petals.lua#L126-L130)、[深色花瓣](../../../raw/current/scripts/prefabs/petals_evil.lua#L71-L75)、[蕨叶](../../../raw/current/scripts/prefabs/foliage.lua#L70-L74)、[多肉](../../../raw/current/scripts/prefabs/succulent_plant.lua#L134-L138)、[火荨麻](../../../raw/current/scripts/prefabs/firenettles.lua#L65-L69)、[犁地草](../../../raw/current/scripts/prefabs/tillweed.lua#L59-L63)、[月树花](../../../raw/current/scripts/prefabs/moontree_blossom.lua#L67-L71)、[必忘我](../../../raw/current/scripts/prefabs/forgetmelots.lua#L56-L60)

- 旧的 `components/dryer.lua` 是上一代晾肉架组件，现役架子用 `container` + `dryingrack`。读旧存档的迁移在 `meatrack2.lua` 的 `OnLoad`：遇到 `data.dryer` 时按 `ingredient` 或 `product` 重新 `SpawnPrefab`、写回 `dryingrack_drytime`／`ApplyDryingInfoSnapshot` 再放进第 2 格；`standardcomponents` 那段只处理“架子被烧毁时把 `dryer` 停掉”，不是迁移。[读档迁移](../../../raw/current/scripts/prefabs/meatrack2.lua#L176-L205)、[烧毁处理](../../../raw/current/scripts/standardcomponents.lua#L85-L88)
- 架子被烧毁时走 `dryingrack:OnBurnt` 的 `InstantDry`：**只对仍在槽里、且带 `dryable` 与产物的原料**生效（等于瞬间晾完并抛出成品）；已经晾好的干叶没有 `dryable`，会被跳过，随后由容器 `DropEverything` 掉在地上。[烧毁](../../../raw/current/scripts/components/dryingrack.lua#L376-L391)、[容器掉落](../../../raw/current/scripts/standardcomponents.lua#L80-L84)

## 消费：隐士蟹茶铺

8 件干叶是隐士蟹（hermitcrab）茶铺 3 级升级的唯一可变材料，配方按"常见花瓣／稀有叶片"两档取数：

| 茶（每级 i = 1..3） | 干叶 | 每级数量 | 同配方固定材料 |
|---|---|---:|---|
| `hermitcrabtea_petals_i` | `petals_dried` | 8／6／4 | `messagebottleempty` ×1 |
| `hermitcrabtea_petals_evil_i` | `petals_evil_dried` | 8／6／4 | 同上 |
| `hermitcrabtea_foliage_i` | `foliage_dried` | 8／6／4 | 同上 |
| `hermitcrabtea_moon_tree_blossom_i` | `moon_tree_blossom_dried` | 8／6／4 | 同上 |
| `hermitcrabtea_succulent_picked_i` | `succulent_picked_dried` | 6／4／2 | 同上 |
| `hermitcrabtea_firenettles_i` | `firenettles_dried` | 6／4／2 | 同上 |
| `hermitcrabtea_tillweed_i` | `tillweed_dried` | 6／4／2 | 同上 |
| `hermitcrabtea_forgetmelots_i` | `forgetmelots_dried` | 6／4／2 | 同上 |

两档数量由 `NUM_COMMON_PETALS_FOR_TEASHOP_LEVEL = {8, 6, 4}` 与 `NUM_RARE_PETALS_FOR_TEASHOP_LEVEL = {6, 4, 2}` 给出（注释说明"杂草与多肉更贵，所以茶更便宜"），配方统一 `TECH.LOST`、`nounlock = true`、`actionstr = "HERMITCRABSHOP"`、`hint_msg = "NEEDSHERMITCRAB_TEASHOP"`。[两档数量](../../../raw/current/scripts/recipes.lua#L1502-L1508)、[八条配方](../../../raw/current/scripts/recipes.lua#L1509-L1516)

- 另一个消费端是烹饪锅：8 件在 `cooking.lua` 里登记 `{decoration = 1, dried = 1}`（`petals_evil_dried` 额外 `magic = .5`），不给 veggie／meat 这类营养标签；但**不代表它们与菜谱无关**——`forgetmelots_dried` 是两道锅菜的名称判定材料：`sweettea`（`names.forgetmelots or names.forgetmelots_dried` + sweetener + frozen）与 `beefalotreat`（`tags.inedible` + `tags.seed` + 同名判定）。[食材登记](../../../raw/current/scripts/cooking.lua#L167-L171)、[甜茶](../../../raw/current/scripts/preparedfoods.lua#L879-L881)、[牛饲料](../../../raw/current/scripts/preparedfoods.lua#L1052-L1055)
- 干叶自身**没有** `perishable`、也**没有** `dryable`：不会再腐坏，也不会被二次晾晒。[主模拟段](../../../raw/current/scripts/prefabs/driedplants.lua#L46-L72)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`edible` | `VEGGIE`、hunger 恒 0、health／sanity 按表；可选 `oneaten` | 仅主模拟 | [[reference/components/edible]]；[赋值](../../../raw/current/scripts/prefabs/driedplants.lua#L51-L58) |
| 组件：`fuel`／`burnable`／`propagator` | `SMALL_FUEL`、`SMALL_BURNTIME`、`flashpoint` 覆写为 2.5–5 | 仅主模拟；火焰强度取 `burnable:SetFXLevel(2)` | [[reference/components/fuel]]、[[reference/components/burnable]]；[挂载](../../../raw/current/scripts/prefabs/driedplants.lua#L63-L68) |
| 组件：`stackable`／`inventoryitem`／`tradable` | 40 一叠、可拾取、`goldvalue` 默认 0 | 仅主模拟 | [[reference/components/stackable]]、[[reference/components/tradable]]；[挂件](../../../raw/current/scripts/prefabs/driedplants.lua#L46-L61) |
| 组件：`hauntable` | `MakeHauntableLaunchAndPerish`：作祟只会把物品抛飞 | 仅主模拟；腐坏分支在源码里已被注释掉 | [[reference/components/hauntable]]；[接线](../../../raw/current/scripts/prefabs/driedplants.lua#L70-L70)、[实现](../../../raw/current/scripts/standardcomponents.lua#L1311-L1337) |
| 标签：`cattoy` | 浣猫把带该标签的物品当玩具 | pristine 就加，客户端可见 | [[reference/components/cattoy]]；[加标签](../../../raw/current/scripts/prefabs/driedplants.lua#L36-L36) |
| 标签：`dryable` | 晾肉架容器过滤与 `OnGetItem` 的入场券 | 由**来源**物品提供，不在这 8 件身上 | [[reference/components/dryable]]；[过滤](../../../raw/current/scripts/containers.lua#L234-L235) |
| 生产者：`meatrack`（`prefabs/meatrack2.lua`） | `container` + `dryingrack`，3 格、不可叠放 | 服务端计时，客户端显示晾晒进度 | [[reference/components/dryingrack]]、[[reference/components/dryer]]；[挂载](../../../raw/current/scripts/prefabs/meatrack2.lua#L325-L334) |
| 消费者：`recipes.lua` 的 8×3 条茶配方 | 每条消耗 1 个空瓶 + 8/6/4 或 6/4/2 片干叶 | 隐士蟹茶铺专用，`TECH.LOST` | [配方](../../../raw/current/scripts/recipes.lua#L1509-L1516) |
| 消费者：`cooking.lua` 食材表 | 登记 `decoration`／`dried`（`petals_evil_dried` 另有 `magic = .5`）；`forgetmelots_dried` 参与 `sweettea`／`beefalotreat` 的名称判定 | 进锅可作填充，也能触发指定菜谱 | [登记](../../../raw/current/scripts/cooking.lua#L167-L171)、[甜茶](../../../raw/current/scripts/preparedfoods.lua#L879-L881)、[牛饲料](../../../raw/current/scripts/preparedfoods.lua#L1052-L1055) |
| 资源：`anim/<build>.zip` | 8 件各自的 build，晾晒时额外要 `meat_rack_food_petals` | 原版资源，不随页分发 | [assets 生成](../../../raw/current/scripts/prefabs/driedplants.lua#L15-L19) |

## 典型接入

### 把某片干叶改成"能吃但不掉理智"

- 前提与选择：服务端 `modmain.lua` 的 `AddPrefabPostInit`。选实例字段而不是改 `driedplants_defs.lua`：那份表在 `require` 时就被工厂读完，运行期再改不会重建已生成的 prefab。
- 生效与撤销：只影响新生成实例；已存在的堆叠不会重算，卸模组后回到原值。
- 原版依据：见[注册与构造](#注册与构造)的共用主模拟段。

```lua
-- modmain.lua：干深色花瓣不再扣理智（原为 -5）
AddPrefabPostInit("petals_evil_dried", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    if inst.components.edible ~= nil then
        inst.components.edible.sanityvalue = 0
    end
end)
```

- 调整与验证：`sanityvalue` 走 `edible:GetSanity(doer)`；验证时看吃下后的理智条，别只看组件字段。

## 常见陷阱与限制

- **误区：文件名 `driedplants` 就是注册名。** 注册名由 `data.name.."_dried"` 拼出，8 个都对得上 `defs` 里的 `name`；文件里没有任何一个 `Prefab("driedplants"…)`。[拼名](../../../raw/current/scripts/prefabs/driedplants.lua#L10-L10)
- **边界：晾晒不是"放在地上等"。** 只有进入 `meatrack` 容器（带 `dryable` 标签）才开始计时，计时在 `dryingrack:OnGetItem` 里；拿出来会记下剩余时间（`dryingrack_drytime`），重新放回时按 `math.min(math.max(10, 剩余), drytime)` 续上；但这份缓存会被 `ForgetItem` 清掉——物品被丢到地上或发生 `stacksizechange`（合并／拆堆）时，`dryingrack_drytime` 归 nil，再放回就是重新从 480 秒起算。[续晾](../../../raw/current/scripts/components/dryingrack.lua#L236-L262)、[缓存清除](../../../raw/current/scripts/components/dryingrack.lua#L226-L230)
- **边界：下雨会暂停，酸雨还会加速腐坏。** `IsExposedToRain` 与酸雨分支都挂在架子上，不是干叶自己的属性。[暂停](../../../raw/current/scripts/components/dryingrack.lua#L78-L94)
- **边界：作祟不会让干叶更快坏。** `MakeHauntableLaunchAndPerish` 的 `perish` 分支整段被注释掉，实际只剩"抛飞"，与函数名不符。[实现](../../../raw/current/scripts/standardcomponents.lua#L1311-L1337)
- **边界：`forgetmelots` 源头可能放不上架子。** 其余 7 个来源都在 pristine 加 `dryable` 标签，`forgetmelots.lua` 只在主模拟 `AddComponent("dryable")`；而容器过滤读的是标签，客户端的候选判定会先落空（推导，未实测）。[无标签](../../../raw/current/scripts/prefabs/forgetmelots.lua#L28-L34)、[过滤](../../../raw/current/scripts/containers.lua#L234-L244)
- **边界：唯一有吃后效果的是 `firenettles_dried`。** 别的 7 件只有三维，非 `plantkin` 吃到干火荨麻叶会挂 `firenettle_toxin`。[减益](../../../raw/current/scripts/prefabs/driedplants_defs.lua#L27-L31)
- **未核验**：未运行游戏。晾晒 480 秒、续晾公式、`forgetmelots` 的客户端过滤均为源码推导；`driedsalticon`／盐架分支（`meatrack_hermit_multi`）未展开。

## 验证记录

- 源码审阅：2026-11-09，Steam buildid 26474873。通读 `prefabs/driedplants.lua` 与 `prefabs/driedplants_defs.lua` 全文；`prefabs/meatrack2.lua` 的容器与 `dryingrack` 挂载段、`components/dryingrack.lua` 全文（`OnGetItem`／`OnDoneDrying`／`PauseDrying`／`OnBurnt`）、`components/dryable.lua`、`components/dryer.lua`（旧组件）；8 个来源 prefab 的 `dryable` 段；`containers.lua` 的 `params.meatrack`；`recipes.lua` 的茶铺配方；`cooking.lua` 的干叶食材登记；`tuning.lua` 的 `DRY_*`／`HEALING_*`／`SANITY_*`／`SMALL_FUEL`／`SMALL_BURNTIME`；`chinese_s.po` 八个名称。
- 修订记录：2026-11-09 第 1 轮独立检视回源复核后按点名条目修正——P2-1 拆开读档迁移（meatrack2 `OnLoad`）与烧毁处理（L59）、P2-2 暂停判定按 `OnIsRaining` 源码改写（L54）、P3-3 可选项改为五个（L38）。外部复检轮再改：P1 干必忘我参与 `sweettea`／`beefalotreat` 名称判定，改写锅料段落与导航行（L79、L94）、P2 480 秒限定为有效晾晒时间并补暂停只存时长、烧毁只瞬间干制在槽原料、续晾缓存遇落地／堆叠变化即清（L53、L60、L122）。第三轮再改：未覆盖里“茶的实际效果”改指 [[reference/prefabs/hermitcrab-tea]] 与 `hermitcrabtea_defs.lua`，普通锅菜仍归 `preparedfoods.lua`（L133）。第四轮再改：L23 的修改边界拆开归属——柜台茶配方归 `recipes.lua`、茶效果归茶页与 `hermitcrabtea_defs.lua`、普通锅菜才归 `preparedfoods.lua`；其余段落未动。
- 未覆盖：茶的实际效果与 buff（归 [[reference/prefabs/hermitcrab-tea]] 与 `prefabs/hermitcrabtea_defs.lua`）、`meatrack_hermit_multi` 的盐架玩法、以及 `scrapbookdata` 对这 8 件的收录；普通锅菜（`sweettea`／`beefalotreat`）的完整数值仍以 `preparedfoods.lua` 为准，本页只写名称判定。
- 游戏运行：未运行；一天按 `total_day_time`＝480 秒换算。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 见交付回报。

## 相关链接

- 干制生产：[[reference/components/dryingrack]]、[[reference/components/dryable]]、[[reference/components/dryer]]
- 物品组件：[[reference/components/edible]]、[[reference/components/fuel]]、[[reference/components/burnable]]、[[reference/components/hauntable]]
- 原料来源：[[reference/prefabs/petals]]、[[reference/prefabs/weeds]]、[[reference/prefabs/moontree-blossom]]
- 同批物品：[[reference/prefabs/spoiled-food]]（晾晒失败的腐坏去向）

返回 [[reference/prefabs/special-food-index|节庆、角色与特殊食品索引]]。
