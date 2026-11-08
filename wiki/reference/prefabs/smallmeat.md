---
title: "小肉与熟小肉、小肉干（smallmeat、cookedsmallmeat、smallmeat_dried）"
created: "2026-11-09"
updated: "2026-11-09"
type: prefab
summary: "小肉三态的构造与差异：堆叠上限 40、晾晒只用 1 天、三维与鲜度分档，以及它与共用 common() 工厂、大肉链在作祟与消费上的区别。"
aliases: ["smallmeat", "cookedsmallmeat", "smallmeat_dried", "小肉", "熟小肉", "小肉干", "小肉链"]
tags: [prefab, item, food]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/meats.lua", "tuning.lua", "constants.lua", "components/edible.lua", "components/perishable.lua", "components/stackable.lua", "components/dryable.lua", "components/cookable.lua", "components/tradable.lua", "components/dryingrack.lua", "components/lootdropper.lua", "components/hauntable.lua", "standardcomponents.lua", "cooking.lua", "componentactions.lua", "prefabs/pigking.lua", "prefabs/meatrack2.lua", "prefabs/lureplant.lua", "containers.lua", "prefabs/rabbit.lua", "prefabs/birds.lua", "prefabs/mole.lua", "prefabs/froglegs.lua", "screens/redux/scrapbookdata.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 小肉与熟小肉、小肉干（smallmeat、cookedsmallmeat、smallmeat_dried）

## 快速判断

- 职责与适用：本页覆盖 `prefabs/meats.lua` 的小肉三注册 `smallmeat`／`cookedsmallmeat`／`smallmeat_dried`。三者与同文件的大肉、怪物肉共用局部工厂 `common()`，工厂逐步说明见 [[reference/prefabs/meat#共用工厂与初始化顺序]]，本页只写差异与自身消费。[注册](../../../raw/current/scripts/prefabs/meats.lua#L905-L907)
- 优先使用：要改堆叠、晾晒时长或三维看[与共用工厂的差异](#与共用工厂的差异)与[数值与获得](#数值与获得)；要改被作祟时的表现看[作祟只有抛飞](#作祟只有抛飞)；要改烹饪标签看[消费端](#消费端)。
- 重要副作用：三态堆叠都是 40（`STACK_SIZE_SMALLITEM`），不是大肉的 20；生小肉晾晒只要 `DRY_FAST`＝480 秒；小肉三态没有作祟转换，被作祟只会被抛飞。
- 修改边界：宿主生物与晾肉架建筑各有归属页待建；`common()` 的标签／组件次序、`quagmire` 分支不在本页重复，统一看大肉页。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `smallmeat` | `smallmeat()` → `common("meat_small","meat_small","raw",{"catfood","rawmeat"},{product="smallmeat_dried",time=DRY_FAST},{product="cookedsmallmeat"})` | bank 与 build 都是 `meat_small`，播 `raw`；主模拟段显式写三维、`PERISH_FAST`、`maxsize = STACK_SIZE_SMALLITEM`，并 `floater:SetScale(0.9)`。[构造](../../../raw/current/scripts/prefabs/meats.lua#L407-L425) |
| `cookedsmallmeat` | `cookedsmallmeat()` → `common("meat_small","meat_small","cooked")`：后三参全缺，故无附加标签、无 `dryable`、无 `cookable` | 播 `cooked`；`PERISH_MED`；堆叠同样改成 40；`floater:SetScale(0.9)`；不能再烤、不能上架。[构造](../../../raw/current/scripts/prefabs/meats.lua#L427-L445) |
| `smallmeat_dried` | `driedsmallmeat()` → `common("meat_rack_food","meat_rack_food","idle_dried_small", nil, {isdried = true})`：bank／build 都换成 `meat_rack_food` | 第 5 参只含 `isdried`，所以无 `dryable`／`cookable` 组件与标签，但仍拿到 `lureplant_bait`；`PERISH_PRESERVED`；堆叠 40；未调 `floater`。[构造](../../../raw/current/scripts/prefabs/meats.lua#L447-L463) |

### 名称核验

`NAMES.SMALLMEAT`＝**小肉**、`NAMES.COOKEDSMALLMEAT`＝**熟小肉**、`NAMES.SMALLMEAT_DRIED`＝**小肉干**（英文 msgid 依次 Morsel／Cooked Morsel／Small Jerky）。三键各唯一命中，均无 `NAME_DETAIL_EXTENTION` 附加。[SMALLMEAT](../../../raw/current/scripts/languages/chinese_s.po#L349680-L349682)、[COOKEDSMALLMEAT](../../../raw/current/scripts/languages/chinese_s.po#L341645-L341647)、[SMALLMEAT_DRIED](../../../raw/current/scripts/languages/chinese_s.po#L349685-L349687)

## 与共用工厂的差异

- **堆叠 40**：`stackable` 在 `common()` 里只按默认值 20 挂上，三个小肉工厂各自在 master sim 段写 `maxsize = TUNING.STACK_SIZE_SMALLITEM`（40），所以三态都是 40。[默认值](../../../raw/current/scripts/components/stackable.lua#L26-L34)、[常量](../../../raw/current/scripts/tuning.lua#L81-L83)、[生小肉](../../../raw/current/scripts/prefabs/meats.lua#L420-L420)
- **晾晒只要 1 天**：生小肉的 `dryable.time` 用 `TUNING.DRY_FAST`＝480 秒，大肉用 `DRY_MED`＝960 秒；产物是 `smallmeat_dried`。[接线](../../../raw/current/scripts/prefabs/meats.lua#L408-L408)、[常量](../../../raw/current/scripts/tuning.lua#L2053-L2056)
- **浮动体只改缩放**：生／熟小肉调 `floater:SetScale(0.9)`，没有设垂直偏移（大肉是 `SetVerticalOffset(0.05)`）；干制什么都不调。[L422-L422](../../../raw/current/scripts/prefabs/meats.lua#L422-L422)、[L442](../../../raw/current/scripts/prefabs/meats.lua#L442-L442)
- **附加标签按参数分组**：`catfood`／`rawmeat` 只出现在生的第 4 参；`lureplant_bait` 只取决于第 5 参是否非空——生小肉与干小肉都带，只有只传 4 参的熟小肉没有。[工厂](../../../raw/current/scripts/prefabs/meats.lua#L161-L167)、[干小肉调用](../../../raw/current/scripts/prefabs/meats.lua#L448-L448)
- **没有作祟转换**：`AddMonsterMeatChange` 在 `meats.lua` 只有三处调用（大肉三态），小肉三态一个都没有。[三个工厂](../../../raw/current/scripts/prefabs/meats.lua#L407-L463)

## 数值与获得

时间基准：`total_day_time`＝480 秒、`perish_warp`＝1；鲜度与晾晒都按它折算。[基准](../../../raw/current/scripts/tuning.lua#L19-L20)、[warp](../../../raw/current/scripts/tuning.lua#L47-L47)

| 注册 | 饥饿／生命／理智 | 鲜度 | 堆叠 | 猪王 | 食性 |
|---|---|---|---|---|---|
| `smallmeat` | 12.5（`CALORIES_SMALL`）／0（显式写 0）／−10 | `PERISH_FAST`＝6 天＝2880 秒 | 40 | 1 金块 | `FOODTYPE.MEAT` |
| `cookedsmallmeat` | 12.5／1（`HEALING_TINY`）／0（显式写 0） | `PERISH_MED`＝10 天＝4800 秒 | 40 | 1 金块 | 同上 |
| `smallmeat_dried` | 12.5／8（`HEALING_MEDSMALL`）／10（`SANITY_SMALL`） | `PERISH_PRESERVED`＝20 天＝9600 秒 | 40 | 1 金块 | 同上 |

图鉴数值与源码逐项一致（`stacksize` 40、`perishable` 2880／4800／9600）：[smallmeat](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L1399-L1399)、[cookedsmallmeat](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L347-L347)、[smallmeat_dried](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L1400-L1400)

- **生小肉的生命是显式 0，不是默认 10**：`edible` 构造默认 `healthvalue = 10`、`hungervalue = 10`，三个小肉工厂都把三维重新赋值，所以"小肉吃了不加血"是数据结果而非缺省。[默认三维](../../../raw/current/scripts/components/edible.lua#L19-L44)、[赋值](../../../raw/current/scripts/prefabs/meats.lua#L414-L416)
- **只有生小肉带 `badfood`**：理智为负触发标签；熟态理智 0、干制品理智为正，都不带。[判定](../../../raw/current/scripts/components/edible.lua#L1-L7)

### 获得路径

| 注册 | 已核验来源（示例，不穷举宿主） | 证据 |
|---|---|---|
| `smallmeat` | 击杀兔子 `rabbit`：`rabbitloot = { "smallmeat" }`（冬季只换 `rabbit_winter_build`，仍是同一 prefab）；它的 `beardling` 形态改走 `beardhair`／`monstermeat`／`nightmarefuel` 随机池 | [兔子](../../../raw/current/scripts/prefabs/rabbit.lua#L45-L45)、[设定](../../../raw/current/scripts/prefabs/rabbit.lua#L90-L104) |
| `cookedsmallmeat` | 由 `cookable.product` 指向，且不止小肉：烤兔子（`rabbit`，默认给熟小肉；厨师理智崩溃或兔子被强制噩梦时改出熟怪物肉）、烤鸟（`birds.lua`）、烤鼹鼠（`mole.lua`）的产物都是它；烤制端是带 `cooker` 标签的实体（营火、火坑等），烹饪锅只挂 `stewer`、不能烤。另有非烤制来源：掉落者已带 `burnt` 标签或正在燃烧（`fueled == nil` 或 `burnable.ignorefuel`）时，`DropLoot` 会把 `smallmeat` 换成 `cookedsmallmeat`（`smallmeat_cooked` 不存在，走 `cooked` 前缀分支）。检索范围内无世界掉落表 | [接线](../../../raw/current/scripts/prefabs/meats.lua#L408-L408)、[兔子](../../../raw/current/scripts/prefabs/rabbit.lua#L275-L277)、[鸟](../../../raw/current/scripts/prefabs/birds.lua#L351-L352)、[鼹鼠](../../../raw/current/scripts/prefabs/mole.lua#L216-L217)、[COOK 门禁](../../../raw/current/scripts/componentactions.lua#L1096-L1105)、[烧毁换熟](../../../raw/current/scripts/components/lootdropper.lua#L415-L442) |
| `smallmeat_dried` | 由 `dryable.product` 指向，且不止小肉：鸡腿 `drumstick`、蝙蝠翅膀 `batwing`、蝙蝠鼻 `batnose`、蛙腿（`froglegs.lua`）干制后都变成它；架子被烧毁时 `InstantDry` 也会立即产出（不继承湿度与鲜度）。检索范围内无世界掉落表 | [小肉](../../../raw/current/scripts/prefabs/meats.lua#L408-L408)、[鸡腿](../../../raw/current/scripts/prefabs/meats.lua#L466-L466)、[蝙蝠翅膀](../../../raw/current/scripts/prefabs/meats.lua#L601-L601)、[蝙蝠鼻](../../../raw/current/scripts/prefabs/meats.lua#L734-L740)、[蛙腿](../../../raw/current/scripts/prefabs/froglegs.lua#L56-L60)、[烧毁速干](../../../raw/current/scripts/components/dryingrack.lua#L376-L391) |

### 消费端

- **猪王**：三态 `tradable.goldvalue` 都是 `GOLD_VALUES.MEAT`＝1，**非鱼人**交付者各换 1 块金块（工厂没有像怪物肉那样置 0；`AcceptTest` 对 `giver:HasTag("merm")` 且非 `pig_token` 的交付直接返回 `nil`）。[赋值](../../../raw/current/scripts/prefabs/meats.lua#L193-L194)、[接受](../../../raw/current/scripts/prefabs/pigking.lua#L598-L606)
- **烹饪锅（作食材，不是烤制端）**：`cooking.lua` 给 `smallmeat` 登记 `meat=0.5`，并自动派生 `smallmeat_cooked`（`meat=0.5, precook=1`）与 `smallmeat_dried`（`meat=0.5, dried=1`）；`cookedsmallmeat` 靠别名表映射成 `smallmeat_cooked` 才被认作食材。煮熟的小肉在锅里只算半份肉，不能顶替大肉。[登记](../../../raw/current/scripts/cooking.lua#L46-L72)、[别名](../../../raw/current/scripts/cooking.lua#L178-L187)、[取值](../../../raw/current/scripts/cooking.lua#L217-L231)
- **晾肉架**：架子 `itemtestfn` 只认 `dryable` 标签且 `acceptsstacks = false`，所以 40 叠要一件一件放；定时完成时只继承湿度、不继承鲜度，架子烧毁走 `InstantDry` 时两者都不继承。[筛选](../../../raw/current/scripts/containers.lua#L234-L244)、[完成](../../../raw/current/scripts/components/dryingrack.lua#L198-L224)
- **`lureplant`**：生小肉与干小肉都带 `lureplant_bait`（干制品的第 5 参 `{isdried = true}` 非空），熟小肉不带。[消费](../../../raw/current/scripts/prefabs/lureplant.lua#L130-L135)

## 作祟只有抛飞

- 三态都由 `common()` 调 `MakeHauntableLaunchAndPerish`，成功分支只做抛飞、把 `hauntvalue` 设为 `HAUNT_TINY`、把冷却设为 `HAUNT_COOLDOWN_SMALL`（并把物品置为未落地）；没有替换 prefab 的分支。[L1311-L1337](../../../raw/current/scripts/standardcomponents.lua#L1311-L1337)
- 对比大肉三态：它们额外被 `AddMonsterMeatChange` 包了一层自定义反应，才会在 25% 概率下变成怪物肉形态；小肉三态没有这层，所以作祟后仍是小肉。[对照](../../../raw/current/scripts/prefabs/meats.lua#L104-L132)
- 抛飞本身不含状态损失：件数、湿度、鲜度都随实体一起被抛出，落地后仍是原物品。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：edible | 三维显式覆盖；`foodtype = FOODTYPE.MEAT`；负理智触发 `badfood` | 三注册都有；仅主模拟 | [[reference/components/edible#关键接口]]；[L414-L416](../../../raw/current/scripts/prefabs/meats.lua#L414-L416) |
| 组件：stackable | 三态都改成 40；`IsFull`／`RoomLeft` 决定拾取合并 | 客户端 maxsize 经 replica 同步 | [[reference/components/stackable#关键接口]]；[L420-L420](../../../raw/current/scripts/prefabs/meats.lua#L420-L420)、[L440](../../../raw/current/scripts/prefabs/meats.lua#L440-L440)、[L460](../../../raw/current/scripts/prefabs/meats.lua#L460-L460) |
| 组件：perishable | `SetPerishTime`＋`StartPerishing`，腐坏换 `spoiled_food` | 2880／4800／9600 秒；仅主模拟 | [[reference/components/perishable#关键接口]]；[L196-L199](../../../raw/current/scripts/prefabs/meats.lua#L196-L199) |
| 组件：dryable | 仅 `smallmeat` 挂，product=`smallmeat_dried`、time=`DRY_FAST` | `smallmeat` 独有 | [[reference/components/dryable#关键接口]]；[L201-L207](../../../raw/current/scripts/prefabs/meats.lua#L201-L207) |
| 组件：cookable | 仅 `smallmeat` 挂，product=`cookedsmallmeat` | `cookedsmallmeat`／`smallmeat_dried` 无 | [[reference/components/cookable#关键接口]]；[L209-L212](../../../raw/current/scripts/prefabs/meats.lua#L209-L212) |
| 组件：tradable | `goldvalue = GOLD_VALUES.MEAT`＝1 | 三注册都有 | [[reference/components/tradable#关键接口]]；[L193-L194](../../../raw/current/scripts/prefabs/meats.lua#L193-L194) |
| 组件：hauntable | `MakeHauntableLaunchAndPerish` 打底；无自定义反应 | 三注册都有 | [[reference/components/hauntable#关键接口]]；[L218-L219](../../../raw/current/scripts/prefabs/meats.lua#L218-L219) |
| 关联 Prefab：`meatrack` | `smallmeat` 的干制宿主，只收 `dryable` 标签 | 3 槽；另有 `meatrack_hermit`（1 槽）与 `meatrack_hermit_multi`（9 槽） | Wiki 待建（晾肉架）；[注册](../../../raw/current/scripts/prefabs/meatrack2.lua#L584-L592) |
| 关联 Prefab：`spoiled_food` | 三态腐坏后的替换物 | 仅主模拟 | Wiki 待建；[替换](../../../raw/current/scripts/prefabs/meats.lua#L199-L199) |
| 资源：`anim/meat_small.zip` | 生／熟小肉共用 bank 与 build；干制改用 `meat_rack_food` | 原版资源，不随页分发 | [assets](../../../raw/current/scripts/prefabs/meats.lua#L5-L5) |

## 典型接入

### 把小肉三态的堆叠改回 20 并缩短晾晒

- 前提与选择：服务端 `modmain.lua` 的 postinit。堆叠不是常数而是每个实例的 `stackable.maxsize`，所以要在三态各自 postinit 里改；晾晒时间要在物品**放上架子之前**改，因为架子在 `OnGetItem` 时才读一次 `GetDryTime()`。[读取时机](../../../raw/current/scripts/components/dryingrack.lua#L247-L263)
- 生效与撤销：postinit 在每次构造（含读档重建）后执行，新造与读档的小肉都生效；`maxsize` 的属性 setter 会把新值同步给 replica，客户端格子的合并上限随之变化。停用模组后新建物品回到 40，已存在的实例保持旧值直到被重新构造。

```lua
-- modmain.lua：小肉三态堆叠改为 20；生小肉晾晒时间减半
local NAMES = { "smallmeat", "cookedsmallmeat", "smallmeat_dried" }

for _, name in ipairs(NAMES) do
    AddPrefabPostInit(name, function(inst)
        if inst.components.stackable ~= nil then
            inst.components.stackable.maxsize = GLOBAL.TUNING.STACK_SIZE_MEDITEM
        end
        if name == "smallmeat" and inst.components.dryable ~= nil then
            inst.components.dryable:SetDryTime(GLOBAL.TUNING.DRY_FAST * 0.5)
        end
    end)
end
```

- 调整与验证：`STACK_SIZE_MEDITEM`＝20、`DRY_FAST`＝480 秒，减半后是 240 秒；验证时看地面合并上限与架子上从放上到变成 `smallmeat_dried` 的耗时，而不是看 tuning 常量本身。已经挂在架子上的物品不受影响，因为计时任务在放上时就已按当时的值排好。

## 常见陷阱与限制

- **误区：小肉也会被作祟变成怪物肉。** 转换只登记在大肉三态；小肉三态被作祟只有抛飞。
- **边界：40 叠不能整叠上架。** `meatrack` 的容器 `acceptsstacks = false`，40 叠要逐件放；同理，作祟抛飞时件数随实体整体保留。
- **边界：熟小肉与干小肉都不可再加工。** 两者既没有 `cookable` 也没有 `dryable`，进锅只作为食材（`meat=0.5`）结算，不能回炉或回架。
- **误区：干制小肉比生小肉"更顶饱"。** 三态饥饿值都是 12.5，干制只提升生命（8）与理智（+10）并延长鲜度。
- **未核验：`quagmire` 模式的 `quagmire_smallmeat`。** 它走同一工厂但带独立 `event_server_data` 分支，且 `quagmire_event_server/` 不在本快照内，本页不描述。

## 验证记录

- 源码审阅：2026-11-09，Steam buildid 26474873。通读 `prefabs/meats.lua` 的三个小肉工厂与 `common()`，核对 `tuning.lua` 的 `STACK_SIZE_SMALLITEM`／`STACK_SIZE_MEDITEM`／`DRY_FAST`／`DRY_MED`／`PERISH_*`／`HEALING_*`／`CALORIES_*`／`SANITY_*`／`GOLD_VALUES`，`stackable`／`perishable`／`dryable`／`cookable`／`tradable`／`dryingrack`／`hauntable` 的相关分支，`cooking.lua` 的食材登记与别名，`containers.lua` 的架子筛选与 `componentactions` 的 `COOK` 门禁，以及 `rabbit`／`birds`／`mole`／`froglegs` 的产物接线。
  名称核验：`chinese_s.po` 的 `NAMES.SMALLMEAT`＝小肉、`NAMES.COOKEDSMALLMEAT`＝熟小肉、`NAMES.SMALLMEAT_DRIED`＝小肉干；三键各唯一命中。
  未覆盖：`quagmire` 模式；干制／烤制原料的完整清单（只核了兔子、鸟、鼹鼠、蛙腿、鸡腿、蝙蝠翅膀与蝙蝠鼻）；模组注入。
- 游戏运行：未运行。秒数、堆叠与金块数为 tuning 与工厂推导（`total_day_time`＝480 秒）。
- 结构检查：见交付回报的 `validate_page.py` 结果。

## 相关链接

- 共用工厂与同批叶页：[[reference/prefabs/meat]]、[[reference/prefabs/monstermeat]]、[[reference/prefabs/fishmeat|鱼肉两链]]
- 可食与鲜度：[[reference/components/edible]]、[[reference/components/perishable]]、[[reference/components/stackable]]
- 加工：[[reference/components/cookable]]、[[reference/components/dryable]]、[[reference/components/tradable]]
- 同样干成小肉干的原料：[[reference/prefabs/drumstick]]、[[reference/prefabs/batwing]]

返回 [[reference/prefabs/index|Prefab 索引]]。
