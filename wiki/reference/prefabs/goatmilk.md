---
title: "带电的羊奶（goatmilk）"
created: "2026-11-09"
updated: "2026-11-10"
type: prefab
summary: "prefabs/goatmilk.lua 的单一注册：3／12.5／10 三维、只对 WX-78 生效的 chargevalue=1、fooddrink／catfood 标签、40 堆叠与 6 天腐坏，以及充能伏特羊掉落与冬盛宴克劳斯袋两条获取路径。"
aliases: ["goatmilk", "带电的羊奶", "羊奶", "电羊奶"]
tags: [prefab, item, food]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/goatmilk.lua", "prefabs/lightninggoat.lua", "prefabs/wx78_common.lua", "components/upgrademoduleowner.lua", "prefabs/catcoon.lua", "prefabs/klaus_sack.lua", "components/klaussackloot.lua", "components/lootdropper.lua", "components/edible.lua", "components/perishable.lua", "components/stackable.lua", "components/tradable.lua", "cooking.lua", "recipes.lua", "actions.lua", "speech_wx78.lua", "prefabs/cookpot.lua", "componentactions.lua", "tuning.lua", "languages/chinese_s.po", "screens/redux/scrapbookdata.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 带电的羊奶（goatmilk）

## 快速判断

- 职责与适用：可生吃的“饮品”食物；三维为饥饿 12.5／生命 3／理智 10，另带只对 WX-78 生效的 `chargevalue = 1`，食物类型沿用 `edible` 默认的 `GENERIC`（**不是** `dairy` 食性，`dairy` 只是烹饪标签）。[构造](../../../raw/current/scripts/prefabs/goatmilk.lua#L11-L58)、[食性默认](../../../raw/current/scripts/components/edible.lua#L19-L25)
- 优先使用：查获取看[获得路径](#获得路径)；查 WX-78 充电与配方看[用途](#用途)；查伏特羊的充能条件看 [[reference/prefabs/lightninggoat]]。
- 重要副作用：原版**没有挤奶动作**，奶来自杀死处于充能状态的伏特羊；正常单次充能只维持 3 个世界周期（充能与放电规则见 [[reference/prefabs/lightninggoat]]），放电后掉落表恢复成不带奶的普通表。
- 修改边界：本页只负责 `goatmilk` 的构造、数值与用途；伏特羊的充能／放电与雷击行为归宿主页，WX-78 的模块系统与 `upgrademoduleowner` 归组件页。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `goatmilk` | `prefabs/goatmilk.lua` 的 `fn`，单注册；先做动画／物理／标签／漂浮，`SetPristine` 后非主模拟直接返回 | 主模拟挂 `edible`（写 3／12.5／10 与 `chargevalue`）、`tradable`（未设 `goldvalue`）、`perishable`（`PERISH_FAST` → `spoiled_food`）、`stackable`（40）、`inspectable`、`inventoryitem`，最后 `MakeHauntableLaunch`。[全工厂](../../../raw/current/scripts/prefabs/goatmilk.lua#L11-L58) |

固定细节：bank／build 都是 `goatmilk`、动画 `idle`；`catfood` 与 `fooddrink` 两个标签在 `SetPristine` **之前**加，因此两端都有；`MakeInventoryFloatable(inst, "small", 0.15, 0.6)` 给漂浮尺寸与偏移。没有 `cookable`／`bait`，也没有其它随时间推进的状态（`perishable` 除外）。[标签与漂浮](../../../raw/current/scripts/prefabs/goatmilk.lua#L18-L29)、[组件段](../../../raw/current/scripts/prefabs/goatmilk.lua#L35-L55)

### 名称核验

简中实显为**带电的羊奶**（英文 Electric Milk），`STRINGS.NAMES.GOATMILK` 一个键命中一次，与注册名同名。[名称](../../../raw/current/scripts/languages/chinese_s.po#L343345-L343347)

## 数值与获取

| 饥饿／生命／理智 | `chargevalue` | 鲜度 | 堆叠 | 其它 |
|---|---|---|---|---|
| 12.5／3／10（`CALORIES_SMALL`／`HEALING_SMALL`／`SANITY_SMALL`） | 1（`WX78_CHARGE_SMALL`） | `PERISH_FAST`＝6 天＝2880 秒 → `spoiled_food` | 40（`STACK_SIZE_SMALLITEM`） | `catfood`、`fooddrink`；`tradable.goldvalue` 保持默认 0 |

数值来源：[三维与充电值](../../../raw/current/scripts/prefabs/goatmilk.lua#L35-L39)、[鲜度](../../../raw/current/scripts/prefabs/goatmilk.lua#L43-L46)、[堆叠](../../../raw/current/scripts/prefabs/goatmilk.lua#L48-L49)、[常量](../../../raw/current/scripts/tuning.lua#L82-L83)、[三维档位](../../../raw/current/scripts/tuning.lua#L2022-L2059)、[充电常量](../../../raw/current/scripts/tuning.lua#L9411-L9411)、[`tradable` 默认值](../../../raw/current/scripts/components/tradable.lua#L1-L7)、[`edible` 默认食性](../../../raw/current/scripts/components/edible.lua#L19-L25)。采集册同项写作 `hungervalue=12.5`、`healthvalue=3`、`sanityvalue=10`、`stacksize=40`、`perishable=2880`、`foodtype="GENERIC"`，与源码一致。[采集册](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L614-L614)

### 获得路径

| 来源 | 成立条件 | 产出与证据 |
|---|---|---|
| 充能状态的伏特羊 | 伏特羊被充能后，`lootdropper` 换成带电掉落表；此时死亡即按该表结算 | `chargedlightninggoat` 表内 `goatmilk` 的 chance 是 1.00（未被燃烧改写时必掉 1 瓶）；普通表没有奶。若它死于常规燃烧／已燃状态，S2 会把奶改写成 `ash`；受控燃烧保留原物。[充能切换](../../../raw/current/scripts/prefabs/lightninggoat.lua#L109-L131)、[两张表](../../../raw/current/scripts/prefabs/lightninggoat.lua#L21-L34)、[燃烧改写](../../../raw/current/scripts/components/lootdropper.lua#L424-L442) |
| 克劳斯袋（冬季盛宴） | 仅在冬季盛宴活动生效时，冬盛宴战利品组额外加入袋子 | 该组固定 `goatmilk`、`goatmilk` 加一份 `winter_food`（4 个），即每袋 2 瓶奶。[冬盛宴组](../../../raw/current/scripts/components/klaussackloot.lua#L83-L88)、[按活动加入](../../../raw/current/scripts/components/klaussackloot.lua#L127-L142)、[成包投出](../../../raw/current/scripts/prefabs/klaus_sack.lua#L34-L50) |

### 用途

- **生吃充电**：`eater:Eat` 走 `wx78_common` 的 `OnEat`，读到 `edible.chargevalue` 就调用 `upgrademoduleowner:DoDeltaCharge(1)`；`SetChargeLevel` 会把结果夹在 0～`max_charge` 之间（默认上限 `WX78_INITIAL_MAXCHARGELEVEL`＝6，模块会抬高上限），所以接近满充时增量会被截断。其他角色没有这个消费者，吃了只得三维。[充电消费](../../../raw/current/scripts/prefabs/wx78_common.lua#L979-L992)、[夹取](../../../raw/current/scripts/components/upgrademoduleowner.lua#L335-L346)、[默认上限](../../../raw/current/scripts/tuning.lua#L9408-L9408)（组件见 [[reference/components/upgrademoduleowner]]）
- **喝的动作**：`fooddrink` 标签让 `ACTIONS.EAT.strfn` 返回 `DRINK`，走喝的分支而不是咬。[动作字符串](../../../raw/current/scripts/actions.lua#L787-L797)
- **烹饪食材**：登记 `dairy = 1`，因为没传 `cancook`，不会派生 `goatmilk_cooked` 条目；物品本身也没有 `cookable` 组件，所以**不能烤**。[食材值](../../../raw/current/scripts/cooking.lua#L145-L145)、[派生条件](../../../raw/current/scripts/cooking.lua#L46-L72)
- **WX-78 模块配方**：`wx78module_taser` 用 5 个 `scandata`＋1 瓶羊奶，`TECH.ROBOTMODULECRAFT_ONE`、`builder_tag = "upgrademoduleowner"`。[配方](../../../raw/current/scripts/recipes.lua#L768-L768)
- **猫粮**：`catfood` 标签让浣猫把它当猫粮接收（与鸟蛋同一条判定）。[浣猫判定](../../../raw/current/scripts/prefabs/catcoon.lua#L273-L279)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：edible | 写三维与 `chargevalue`；`foodtype` 未赋值 | 仅主模拟；只有 WX-78 的 `OnEat` 读 `chargevalue` | [[reference/components/edible]]；[赋值](../../../raw/current/scripts/prefabs/goatmilk.lua#L35-L39)、[消费](../../../raw/current/scripts/prefabs/wx78_common.lua#L987-L990) |
| 组件：perishable | `SetPerishTime(PERISH_FAST)`＋`StartPerishing`＋替换 `spoiled_food` | 主模拟；6 天 | [[reference/components/perishable]]；[接线](../../../raw/current/scripts/prefabs/goatmilk.lua#L43-L46) |
| 组件：stackable | 显式 `maxsize = STACK_SIZE_SMALLITEM`＝40 | 两端由 replica 同步 | [[reference/components/stackable]]；[赋值](../../../raw/current/scripts/prefabs/goatmilk.lua#L48-L49) |
| 组件：tradable | 只挂组件，未写 `goldvalue`，保持默认 0 | 主模拟 | [[reference/components/tradable]]；[挂载](../../../raw/current/scripts/prefabs/goatmilk.lua#L41-L41) |
| 标签：`fooddrink` | 让 EAT 动作走 `DRINK` 分支 | 两端（pristine 前加） | [动作](../../../raw/current/scripts/actions.lua#L787-L797) |
| 标签：`catfood` | 浣猫当猫粮接收 | 两端 | [浣猫](../../../raw/current/scripts/prefabs/catcoon.lua#L273-L279) |
| 关联 Prefab：`lightninggoat` | 唯一常规产奶者，奶在充能状态下才进掉落表 | 主模拟；见宿主页 | [[reference/prefabs/lightninggoat]]；[掉落表](../../../raw/current/scripts/prefabs/lightninggoat.lua#L28-L34) |
| 关联 Prefab：`wx78module_taser` | 消耗 1 瓶羊奶的 WX-78 模块 | 需 `upgrademoduleowner` | [配方](../../../raw/current/scripts/recipes.lua#L768-L768) |

### 共享规则接入（S4／S5／S6，S1 的反面）

- **S4 烹饪登记**：`AddIngredientValues({"goatmilk"}, {dairy=1})`，第三参未传 → 不生成 `goatmilk_cooked` 的派生食材条目。[登记](../../../raw/current/scripts/cooking.lua#L145-L145)
- **S5 堆叠**：文件里显式写 `TUNING.STACK_SIZE_SMALLITEM`（＝40），照写为 40。[常量](../../../raw/current/scripts/tuning.lua#L83-L83)
- **S6 腐坏替换**：`onperishreplacement = "spoiled_food"`，不是 `spoiled_fish` 也不是 `rottenegg`。[替换](../../../raw/current/scripts/prefabs/goatmilk.lua#L46-L46)
- **S1 的反面**：本页物品**没有** `cookable` 组件与标签，COOK 动作不会出现；即使拿到带 `cooker` 的实体旁也无法烤。[构造](../../../raw/current/scripts/prefabs/goatmilk.lua#L41-L55)、[COOK 门禁](../../../raw/current/scripts/componentactions.lua#L1096-L1105)

## 典型接入

### 提高羊奶给 WX-78 的充电量

- 前提与选择：服务端 `modmain.lua` postinit。改 `edible.chargevalue` 而不是改 `WX78_CHARGE_SMALL`（该常量还被其它充电食物引用）。
- 生效与撤销：`chargevalue` 在吃的那一刻由 WX-78 的 `OnEat` 读取，随时改都影响下一次进食；对非 WX-78 角色无效果。卸模组后新实例回到 1。
- 原版依据：`goatmilk` 写 `chargevalue = TUNING.WX78_CHARGE_SMALL`，消费端读字段后调 `DoDeltaCharge`。[写入](../../../raw/current/scripts/prefabs/goatmilk.lua#L39-L39)、[消费](../../../raw/current/scripts/prefabs/wx78_common.lua#L987-L990)

```lua
-- modmain.lua：把一瓶羊奶的充电量从 1 提到 3
AddPrefabPostInit("goatmilk", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    local edible = inst.components.edible
    if edible ~= nil then
        edible.chargevalue = 3
    end
end)
```

- 调整与验证：先用 WX-78 消耗到至少留 3 点充电空间（充能有上限，满充时会截断增量），再吃一瓶看模块充能条增量是否为 3；换非 WX-78 角色确认只按 12.5／3／10 结算、没有充电。

## 常见陷阱与限制

- **误区：可以对伏特羊“挤奶”。** 在 `components/`、`actions.lua`、`prefabs/` 与台词／字符串表中检索 `MILK`、`milking`、`Milkable`，命中只有角色描述台词（WX-78 的 `GOATMILK = "EXCITED MILK"`）与字符串表里的同名条目，没有挤奶动作或挤奶组件；奶只由击杀充能羊时的掉落表产出。[台词](../../../raw/current/scripts/speech_wx78.lua#L2619-L2619)、[掉落表](../../../raw/current/scripts/prefabs/lightninggoat.lua#L28-L34)
- **误区：任何伏特羊都掉奶。** 普通表只有两份肉与 0.25 的羊角；奶只在充能状态的 `chargedlightninggoat` 表里，且放电后立即换回普通表。[两张表](../../../raw/current/scripts/prefabs/lightninggoat.lua#L21-L34)、[放电](../../../raw/current/scripts/prefabs/lightninggoat.lua#L84-L98)
- **边界：`chargevalue` 不是通用属性。** 只有 WX-78 的 `OnEat` 消费它；其它吃者不会因此获得任何充电。[消费端](../../../raw/current/scripts/prefabs/wx78_common.lua#L979-L992)
- **边界：它不是“奶类食物类型”。** `foodtype` 保持 `GENERIC`，按食性判定的吃者（如只吃某类食物的角色）不会把它当 `dairy`；`dairy` 只在烹饪结算里生效。[食材值](../../../raw/current/scripts/cooking.lua#L145-L145)
- **边界：克劳斯袋的奶有活动门槛。** 冬盛宴战利品组只在 `IsSpecialEventActive(WINTERS_FEAST)` 时加入；非活动期开袋拿不到这 2 瓶。[加入条件](../../../raw/current/scripts/components/klaussackloot.lua#L127-L142)
- **未核验**：本页未运行游戏；冬盛宴袋的实际开袋结果、`DRINK` 动画与 WX-78 充能条表现都只按源码推导。

## 验证记录

- 源码审阅：2026-11-09，Steam buildid 26474873。核对 `prefabs/goatmilk.lua` 全文；`lightninggoat` 的两张掉落表与充能切换；`wx78_common` 的 `OnEat` 充电消费；`klaussackloot` 的冬盛宴组与 `GetLoot`、`klaus_sack` 的成包投出；`edible`／`perishable`／`stackable`／`tradable` 的默认值与接口；`cooking` 的食材值与 `cancook` 派生、`recipes` 的 `wx78module_taser`、`actions` 的 `DRINK` 分支、`componentactions` 的 COOK 门禁；`tuning` 与 `scrapbookdata` 交叉核对。
- 名称核验：`languages/chinese_s.po` 的 `NAMES.GOATMILK`＝带电的羊奶（英文 Electric Milk）。
- 未覆盖：WX-78 模块系统与 `upgrademoduleowner` 内部、`WX78_CHARGING_FOODS` 旧表的迁移、以及冬盛宴礼物包装的拆包细节。
- 游戏运行：未运行。
- 结构检查：`validate_page.py` 通过（standard 档内）；`validate_source_links.py --strict` 0 错 0 警。

## 相关链接

- 宿主与掉落：[[reference/prefabs/lightninggoat]]
- 可食／鲜度／堆叠：[[reference/components/edible]]、[[reference/components/perishable]]、[[reference/components/stackable]]
- 充电与交易：[[reference/components/upgrademoduleowner]]、[[reference/components/tradable]]
- 同批：[[reference/prefabs/bird-egg]]、[[reference/prefabs/tallbird-egg]]

返回 [[reference/prefabs/index|Prefab 索引]]。
