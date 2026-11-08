---
title: "象鼻与象鼻排（trunk_summer、trunk_winter、trunk_cooked）"
created: "2026-11-09"
updated: "2026-11-09"
type: prefab
summary: "prefabs/trunk.lua 的 create_common 工厂：夏／冬象鼻只差动画与显示名，三维、鲜度、堆叠、交易值一致；烤制成象鼻排，考拉象三种掉落表与燃烧换名，以及它作烹饪锅 meat=1 食材、象鼻背心材料的位置。"
aliases: ["trunk_summer", "trunk_winter", "trunk_cooked", "象鼻", "冬象鼻", "象鼻排", "考拉象鼻"]
tags: [prefab, item, food]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/trunk.lua", "prefabs/koalefant.lua", "prefabs/spat.lua", "prefabs/warg.lua", "prefablist.lua", "cooking.lua", "preparedfoods.lua", "recipes.lua", "tuning.lua", "components/edible.lua", "components/perishable.lua", "components/stackable.lua", "components/cookable.lua", "components/tradable.lua", "components/floater.lua", "components/lootdropper.lua", "containers.lua", "componentactions.lua", "standardcomponents.lua", "prefabs/meatrack2.lua", "screens/redux/scrapbookdata.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 象鼻与象鼻排（trunk_summer、trunk_winter、trunk_cooked）

## 快速判断

- 职责与适用：三个注册都由 `prefabs/trunk.lua` 的 `create_common(anim, cookable)` 生成。夏象鼻与冬象鼻只差动画（`idle_summer`／`idle_winter`，含采集册动画）与显示名，三维、鲜度、堆叠、交易值完全相同；两态都能烤成 `trunk_cooked`，熟态自己没有 `cookable`。[共用工厂](../../../raw/current/scripts/prefabs/trunk.lua#L12-L63)、[三个注册](../../../raw/current/scripts/prefabs/trunk.lua#L119-L121)
- 优先使用：改数值／鲜度看[数值与获得](#数值与获得)；改掉落看[掉落表](#掉落表)；改烹饪定位看[消费端](#消费端)；要复刻「燃烧后掉落熟品」的写法看[典型接入](#典型接入)。
- 重要副作用：烤制把新鲜度折成 `1-(1-p)*0.5` 给产物（[Cook](../../../raw/current/scripts/components/cookable.lua#L18-L37)）；燃烧或已烧毁的考拉象改用火焰掉落表，掉落里的 `trunk` 只是占位名，`DropLoot` 会按 `trunk_cooked` 换名（[掉落表](../../../raw/current/scripts/prefabs/koalefant.lua#L22-L25)、[换名](../../../raw/current/scripts/components/lootdropper.lua#L424-L431)）。
- 修改边界：只覆盖这三个注册。考拉象本体（AI／狩猎）、象鼻背心与 `koalefig_trunk` 料理的成品数值均 Wiki 待建（源码入口 `prefabs/koalefant.lua`、`recipes.lua`、`preparedfoods.lua`），本页只给入口；`quagmire` 等模式未查。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `trunk_summer` | `create_summer()` 调 `create_common("idle_summer", true)`：共用段做动画、可烤标签、漂浮与 `SetPristine`，非主模拟在 `SetPristine` 之后返回 | 夏象鼻：`cookable` 标签在两端，`cookable`／`perishable`／`edible` 等组件只在主模拟；主模拟末尾写 `PERISH_FAST` 并 `StartPerishing`。[夏态](../../../raw/current/scripts/prefabs/trunk.lua#L65-L81) |
| `trunk_winter` | `create_winter()` 调 `create_common("idle_winter", true)` | 与夏象鼻**数值**逐字段相同，只有动画名与显示名不同（连 `scrapbook_anim` 也跟着动画名走）；两态烤出的都是 `trunk_cooked`。[冬态](../../../raw/current/scripts/prefabs/trunk.lua#L83-L99)、[采集册动画](../../../raw/current/scripts/prefabs/trunk.lua#L24)(../../../raw/current/scripts/prefabs/trunk.lua#L83-L99) |
| `trunk_cooked` | `create_cooked()` 调 `create_common("cooked")`，`cookable` 缺省 | 熟态没有 `cookable` 标签与组件，不能再烤；三维与鲜度按熟品档重写。[熟态](../../../raw/current/scripts/prefabs/trunk.lua#L101-L117) |

`create_common` 的固定顺序：`AddTransform`／`AddAnimState`／`AddNetwork` → `MakeInventoryPhysics` → bank `trunk`／build `koalephant_trunk` → 写 `inst.scrapbook_anim = anim`（采集册按该动画取帧）→ 条件 `cookable` 标签 → `MakeInventoryFloatable(inst)`（默认 `small`、零偏移、缩放 1）→ `SetPristine` → 主模拟挂 `stackable`（20）、`inspectable`、`inventoryitem`、`tradable`、`edible`（`ismeat = true`、`FOODTYPE.MEAT`）、条件 `cookable`、`perishable`（只设腐坏替换）、`MakeHauntableLaunchAndPerish`。[工厂主体](../../../raw/current/scripts/prefabs/trunk.lua#L12-L63)

`perishable` 的三项分开写：共用段只设 `onperishreplacement = "spoiled_food"`，鲜度与 `StartPerishing` 由各变体补；漂浮参数三件都没改，所以都是默认外观。[腐坏替换](../../../raw/current/scripts/prefabs/trunk.lua#L57-L58)、[夏态数值](../../../raw/current/scripts/prefabs/trunk.lua#L72-L78)、[漂浮默认](../../../raw/current/scripts/standardcomponents.lua#L1772-L1791)

### 名称核验

`trunk_summer` 的简中实显是**象鼻**（Koalefant Trunk），`trunk_winter` 是**冬象鼻**（Winter Koalefant Trunk），`trunk_cooked` 是**象鼻排**（Koalefant Trunk Steak）；三个键各命中一次，注册名与 `STRINGS.NAMES` 键同名。[TRUNK_SUMMER](../../../raw/current/scripts/languages/chinese_s.po#L350850-L350852)、[TRUNK_WINTER](../../../raw/current/scripts/languages/chinese_s.po#L350855-L350857)、[TRUNK_COOKED](../../../raw/current/scripts/languages/chinese_s.po#L350845-L350847)

## 数值与获得

时间基准：`seg_time`＝30 秒、`total_day_time`＝480 秒、`perish_warp`＝1，鲜度按 480 秒／天换算。[基准](../../../raw/current/scripts/tuning.lua#L19-L20)、[`perish_warp`](../../../raw/current/scripts/tuning.lua#L47)

| 注册 | 饥饿／生命／理智 | 鲜度 | 堆叠 | 交易值 | 转换 |
|---|---|---|---|---|---|
| `trunk_summer` | 37.5／30／0 | `PERISH_FAST`＝6 天＝2880 秒 | 20 | `GOLD_VALUES.MEAT`＝1 | 烤制 → `trunk_cooked`；无干制 |
| `trunk_winter` | 37.5／30／0 | 6 天＝2880 秒 | 20 | 1 | 同上 |
| `trunk_cooked` | 75／40／0 | `PERISH_SLOW`＝15 天＝7200 秒 | 20 | 1 | 无（无 `cookable`／`dryable`） |

数值来源：[夏态](../../../raw/current/scripts/prefabs/trunk.lua#L72-L78)、[冬态](../../../raw/current/scripts/prefabs/trunk.lua#L90-L96)、[熟态](../../../raw/current/scripts/prefabs/trunk.lua#L108-L114)、[常量](../../../raw/current/scripts/tuning.lua#L2021-L2030)、[热量](../../../raw/current/scripts/tuning.lua#L2058-L2065)、[鲜度](../../../raw/current/scripts/tuning.lua#L2043-L2051)、[交易值](../../../raw/current/scripts/tuning.lua#L1946-L1952)、[堆叠](../../../raw/current/scripts/tuning.lua#L82-L83)。采集册同项写作 `stacksize=20`、`hungervalue=37.5/75`、`healthvalue=30/40`、`sanityvalue=0`、`perishable=2880/7200`、`build="koalephant_trunk"`，与源码一致。[采集册](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L1590-L1592)

### 掉落表

| 掉落表 | 内容 | 触发与证据 |
|---|---|---|
| `loot_summer` | 8 份 `meat` 加 1 个 `trunk_summer` | `koalefant_summer` 主模拟 `SetLoot`。[表](../../../raw/current/scripts/prefabs/koalefant.lua#L22-L23)、[接线](../../../raw/current/scripts/prefabs/koalefant.lua#L154-L163) |
| `loot_winter` | 8 份 `meat` 加 1 个 `trunk_winter` | `koalefant_winter` 主模拟 `SetLoot`。[接线](../../../raw/current/scripts/prefabs/koalefant.lua#L166-L173) |
| `loot_fire` | 8 份 `meat` 加 1 个占位名 `trunk`；注释写明它应被换成 `trunk_cooked` | `SetLootSetupFn(lootsetfn)` 在掉落生成本身时检查：掉落者正在燃烧或已带 `burnt` 就整表换成 `loot_fire`。[表与注释](../../../raw/current/scripts/prefabs/koalefant.lua#L24-L25)、[lootsetfn](../../../raw/current/scripts/prefabs/koalefant.lua#L67-L71)、[接线](../../../raw/current/scripts/prefabs/koalefant.lua#L119-L120) |

`trunk` 不是注册名：`prefablist.lua` 是**文件名**清单（`PREFABFILES`），`trunk` 指 `prefabs/trunk.lua`；`trunk.lua` 只返回三个注册。[清单头](../../../raw/current/scripts/prefablist.lua#L1-L2)、[清单条目](../../../raw/current/scripts/prefablist.lua#L1316)、[返回表](../../../raw/current/scripts/prefabs/trunk.lua#L119-L121)

`DropLoot` 的改写次序是 `BURNED_LOOT_OVERRIDES` → `<名>_cooked` → `cooked<名>`，所以占位名 `trunk` 命中第二条的 `trunk_cooked`；这条路径只要求掉落者「已 `burnt`，或正在燃烧且（无 `fueled` 或 `burnable.ignorefuel` 为真）」，不要读成无条件出熟品。[门槛](../../../raw/current/scripts/components/lootdropper.lua#L415-L421)、[换名](../../../raw/current/scripts/components/lootdropper.lua#L424-L431)

狩猎支线：钢羊（`spat`）与座狼（`warg`）从脚印调查生成时，会先跑 `SimulateKoalefantDrops`——撒 2 块 `meat`，另有 0.5 概率按季节撒一个 `trunk_summer`／`trunk_winter`，模拟「本该出现的考拉象已被吃掉」；所以象鼻不只来自亲手击杀考拉象。[函数](../../../raw/current/scripts/prefabs/koalefant.lua#L27-L39)、[钢羊调用](../../../raw/current/scripts/prefabs/spat.lua#L180-L188)、[座狼调用](../../../raw/current/scripts/prefabs/warg.lua#L375-L383)、[钢羊名](../../../raw/current/scripts/languages/chinese_s.po#L349739-L349742)、[座狼名](../../../raw/current/scripts/languages/chinese_s.po#L351749-L351752)

### 消费端

- 烹饪锅把三件都记为 `meat = 1`（没有 `cancook`、也没有 `candry`），三件各自以自己的名字进锅，不会派生 `_cooked`／`_dried` 食材条目。[食材值](../../../raw/current/scripts/cooking.lua#L163)、[槽位判定](../../../raw/current/scripts/containers.lua#L373-L375)
- 用到象鼻的锅料理是 `koalefig_trunk`（无花果酿象鼻）：判定只要求 `trunk_summer`／`trunk_winter`／`trunk_cooked` 之一加 `fig`／`fig_cooked`，优先级 40。配方与成品数值归料理页，本页只记原料关系。[配方](../../../raw/current/scripts/preparedfoods.lua#L904-L913)
- 制作配方两件：象鼻背心用 `trunk_summer` 1 加丝绸 8，冬象鼻背心用 `trunk_winter` 1 加丝绸 8 与牛毛 2。[配方](../../../raw/current/scripts/recipes.lua#L548-L549)
- 三件都**不能**上晾肉架：没有 `dryable` 标签／组件，肉架的槽位测试只认该标签（主模拟例外只有「刚生成 `GetTimeAlive() == 0`」与「从本架原位挪回」，只影响能否入槽）；`meatrack2` 的架面依赖表列了蛙腿、鱼、鳗鱼，也没有象鼻。[槽位测试](../../../raw/current/scripts/containers.lua#L234-L244)、[依赖表](../../../raw/current/scripts/prefabs/meatrack2.lua#L34-L54)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：edible | 写 `ismeat = true`、`FOODTYPE.MEAT` 与三维；三件都没写 `meat` 标签 | 仅主模拟；食性由 `foodtype` 与 `edible_MEAT` 标签决定 | [[reference/components/edible#关键接口]]；[赋值](../../../raw/current/scripts/prefabs/trunk.lua#L46-L50)、[默认三维](../../../raw/current/scripts/components/edible.lua#L19-L44) |
| 组件：perishable | 共用段设腐坏替换 `spoiled_food`，各变体补鲜度并 `StartPerishing` | 生品 2880 秒、熟品 7200 秒 | [[reference/components/perishable#关键接口]]；[替换](../../../raw/current/scripts/prefabs/trunk.lua#L57-L58) |
| 组件：cookable | `product = trunk_cooked`；`Cook()` 生成产物并把鲜度折半补回 | 仅夏／冬两态；两端只有标签，组件在主模拟 | [[reference/components/cookable#关键接口]]；[接线](../../../raw/current/scripts/prefabs/trunk.lua#L52-L55) |
| 组件：stackable | 显式写 `maxsize = STACK_SIZE_MEDITEM`＝20（比蛙腿、鱼类的 40 少一半） | 两端由 replica 同步 | [[reference/components/stackable#关键接口]]；[赋值](../../../raw/current/scripts/prefabs/trunk.lua#L39-L40) |
| 组件：tradable | 三件都写 `goldvalue = GOLD_VALUES.MEAT`＝1，可交猪王 | 主模拟 | [[reference/components/tradable#关键接口]]；[赋值](../../../raw/current/scripts/prefabs/trunk.lua#L72) |
| 组件：hauntable | `MakeHauntableLaunchAndPerish` 的作祟只抛飞 | 加速腐烂分支被注释，实际只 `Launch` | [[reference/components/hauntable#关键接口]]；[作祟](../../../raw/current/scripts/standardcomponents.lua#L1311-L1337) |
| 标签：`cookable` | 只有夏／冬两态有；熟态没有，所以熟象鼻既不提供烤制动作也不被烤制目标接受 | 两端；背包里与地上物品走同一判定 | [烤制门槛](../../../raw/current/scripts/componentactions.lua#L1096-L1105) |
| 关联 Prefab：`trunk_cooked`／`spoiled_food` | 前者是烤制产物与火焰掉落表的换名结果，后者是腐坏替换 | 生成点分别是 `cookable.product` 与 `perishable.onperishreplacement` | [烤制产物](../../../raw/current/scripts/prefabs/trunk.lua#L54)、[腐坏产物](../../../raw/current/scripts/prefabs/trunk.lua#L58) |
| 掉落宿主：`koalefant_summer`／`koalefant_winter` | 两种考拉象各 8 份肉加 1 个当季象鼻；燃烧时换火焰表 | 主模拟 `SetLoot`，燃烧判定在掉落生成时 | [lootsetfn](../../../raw/current/scripts/prefabs/koalefant.lua#L67-L71) |
| 资源：`anim/koalephant_trunk.zip` | bank `trunk`、build `koalephant_trunk`，三态只切动画 | 原版资源，不随页分发 | [assets](../../../raw/current/scripts/prefabs/trunk.lua#L1-L4) |

## 典型接入

### 让象鼻烤成自定义熟品并拉长生品鲜度

- 前提与选择：服务端 `modmain.lua` 的 postinit。改 `cookable.product` 只影响「之后在带 `cooker` 的实体上烤」这一条路径，不影响火焰掉落表（那条走 `trunk_cooked` 换名）。烤制门禁除了目标带 `cooker` 且未 `fueldepleted`，还要求：目标带 `dangerouscooker` 时 `doer:HasTag("expertchef")`；骑乘时只有烹饪器属于自己才可烤（本句走食材对烹饪器的 `cookable` 分支，骑乘门禁看目标；另一条 `cooker` 分支才检查食材归属，[cookable 分支](../../../raw/current/scripts/componentactions.lua#L1096-L1105)、[cooker 分支](../../../raw/current/scripts/componentactions.lua#L1113-L1126)）。
- 生效与撤销：两态各注册一次，已烤好的物品不变；卸模组后新烤的仍回 `trunk_cooked`。产物必须是已注册的 prefab，否则 `SpawnPrefab` 返回 `nil`、物品被消耗却不产出。
- 原版依据：`create_common` 写 `product`，`Cook()` 只在该字段非 `nil` 时生成产物，并检查产物有没有 `perishable` 再折算鲜度。[接线](../../../raw/current/scripts/prefabs/trunk.lua#L52-L55)、[Cook](../../../raw/current/scripts/components/cookable.lua#L18-L37)

```lua
-- modmain.lua：夏／冬象鼻烤成自定义熟品，并把生品鲜度改成 PERISH_MED
local function retune_trunk(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    local cookable = inst.components.cookable
    if cookable ~= nil then
        cookable.product = "my_cooked_trunk" -- 必须是已注册的 prefab
    end
    local perishable = inst.components.perishable
    if perishable ~= nil then
        perishable:SetPerishTime(GLOBAL.TUNING.PERISH_MED)
    end
end
AddPrefabPostInit("trunk_summer", retune_trunk)
AddPrefabPostInit("trunk_winter", retune_trunk)
```

- 调整与验证：`PERISH_FAST`＝2880 秒、`PERISH_MED`＝4800 秒；实测烤一份看产物名，再等一天看鲜度是否按 10 天折算。

## 常见陷阱与限制

- **误区：`trunk` 能生成。** 它只出现在 `PREFABFILES`（文件名清单）与火焰掉落表里；`SpawnPrefab("trunk")` 得到 `nil`，掉落依赖 `DropLoot` 的 `_cooked` 后缀换名。[清单](../../../raw/current/scripts/prefablist.lua#L1316)、[换名](../../../raw/current/scripts/components/lootdropper.lua#L424-L431)
- **边界：掉落有三种结果，不是两种。** ①未燃烧也未带 `burnt`：用当季表，掉生象鼻。②`lootsetfn` 只看 `burnable:IsBurning()` 或 `burnt` 就整表换成 `loot_fire`（[lootsetfn](../../../raw/current/scripts/prefabs/koalefant.lua#L67-L71)），而 `DropLoot` 的换名门禁另看 `fueled`：只有「已 `burnt`，或燃烧且（无 `fueled` 或 `burnable.ignorefuel` 为真）」时占位名 `trunk` 才换成 `trunk_cooked`（[门槛](../../../raw/current/scripts/components/lootdropper.lua#L415-L421)、[换名](../../../raw/current/scripts/components/lootdropper.lua#L424-L431)）。③若燃烧但带 `fueled` 且 `ignorefuel` 为假：表已换成 `loot_fire`、`trunk` 又不会被换名，而 `trunk` 不是注册名，`SpawnLootPrefab` 拿到 `nil` 后什么都不掉（[生成](../../../raw/current/scripts/components/lootdropper.lua#L382-L385)）。本页两种考拉象都没有 `fueled`，第三种只在模组给它加燃料组件时出现。
- **误区：冬象鼻数值更高。** 两态三维、鲜度、堆叠、交易值逐项相同，差异只有动画与显示名。[夏态](../../../raw/current/scripts/prefabs/trunk.lua#L74-L78)、[冬态](../../../raw/current/scripts/prefabs/trunk.lua#L92-L96)
- **边界：象鼻不能晾晒、熟象鼻不能再烤。** 三件都没有 `dryable`，熟态没有 `cookable`；这两条都由标签在动作层挡住，不看物品名。[槽位测试](../../../raw/current/scripts/containers.lua#L234-L244)、[烤制门槛](../../../raw/current/scripts/componentactions.lua#L1096-L1105)
- **未核验**：本页未运行游戏；狩猎支线的 0.5 概率只按源码读出，未实测。

## 验证记录

- 源码审阅：2026-11-09，Steam buildid 26474873。通读 `prefabs/trunk.lua` 的 `create_common` 与三个变体工厂；核对 `prefabs/koalefant.lua` 的三张掉落表、`lootsetfn`、`SimulateKoalefantDrops` 与两个注册；`components/lootdropper.lua` 的 `GenerateLoot`／`DropLoot` 顺序与换名规则；`edible`／`perishable`／`stackable`／`cookable`／`tradable`／`floater` 分支；`cooking.lua` 与 `preparedfoods.lua` 的食材／配方判定；`recipes.lua` 的两件象鼻背心；`tuning` 的鲜度／热量／交易／堆叠常量；`scrapbookdata` 交叉核对。
  名称核验：`languages/chinese_s.po` 的 `NAMES.TRUNK_SUMMER`＝象鼻、`NAMES.TRUNK_WINTER`＝冬象鼻、`NAMES.TRUNK_COOKED`＝象鼻排。
  未覆盖：考拉象的 AI、血量与刷新；狩猎链的完整节点；模组注入。
- 游戏运行：未运行。天数、秒数与三维均为 tuning 推导（`total_day_time`＝480 秒）。
- 结构检查：`validate_page.py` 通过（standard 档内）；`validate_source_links.py --strict` 0 错 0 警。

## 相关链接

- 可食与鲜度：[[reference/components/edible]]、[[reference/components/perishable]]
- 烤制与堆叠：[[reference/components/cookable]]、[[reference/components/stackable]]
- 掉落与换名：[[reference/components/lootdropper]]
- 相邻页：[[reference/prefabs/meat]]、[[reference/prefabs/smallmeat]]、[[reference/prefabs/fishmeat]]

返回 [[reference/prefabs/index|Prefab 索引]]。
