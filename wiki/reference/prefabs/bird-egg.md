---
title: "鸟蛋与熟鸟蛋（bird_egg、bird_egg_cooked）"
created: "2026-11-09"
updated: "2026-11-09"
type: prefab
summary: "prefabs/egg.lua 的 bird_egg 与 bird_egg_cooked 共用 commonfn 工厂：猫粮／诱饵／40 堆叠、10 天腐坏成 rottenegg 与烤制两条转换，外加鸟笼喂肉产蛋、企鹅下蛋、书页配方与蚁狮祭品等来源和去向。"
aliases: ["bird_egg", "bird_egg_cooked", "鸟蛋", "熟鸟蛋", "鸟蛋食材"]
tags: [prefab, item, food]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/egg.lua", "prefabs/birdcage.lua", "prefabs/penguin.lua", "brains/penguinbrain.lua", "recipes.lua", "cooking.lua", "componentactions.lua", "components/lootdropper.lua", "components/klaussackloot.lua", "components/edible.lua", "components/perishable.lua", "components/tradable.lua", "components/bait.lua", "prefabs/antlion.lua", "prefabs/catcoon.lua", "prefabs/cookpot.lua", "tuning.lua", "languages/chinese_s.po", "screens/redux/scrapbookdata.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 鸟蛋与熟鸟蛋（bird_egg、bird_egg_cooked）

## 快速判断

- 职责与适用：两个注册同出 `prefabs/egg.lua`，共用 `commonfn`；生蛋 `bird_egg` 可烤成 `bird_egg_cooked`、10 天腐坏成 `rottenegg`，熟蛋 6 天腐坏成 `spoiled_food`。二者都带 `catfood`、可堆叠 40、可作陷阱诱饵。[构造](../../../raw/current/scripts/prefabs/egg.lua#L25-L81)、[两处腐坏替换](../../../raw/current/scripts/prefabs/egg.lua#L61-L64)、[熟态](../../../raw/current/scripts/prefabs/egg.lua#L103-L119)
- 优先使用：改三维／鲜度看[数值与获得](#数值与获得)；查来源看[获得路径](#获得路径)；改烤制产物看[关联机制导航](#关联机制导航)的 `cookable` 行。
- 重要副作用：腐坏产物两个注册不同（生蛋 `rottenegg`、熟蛋 `spoiled_food`），不能统一按 `spoiled_food` 理解；烤制会把新鲜度按 `1-(1-p)*0.5` 折给产物（见 [[reference/components/cookable]]）。
- 修改边界：本页只负责这两个注册的工厂、数值与来源；`rottenegg` 仅在腐坏替换处关联（不建页）、鸟笼与企鹅本体行为、以及克劳斯袋等其它食物来源各归其页。**克劳斯袋不含 `bird_egg`**（见[常见陷阱](#常见陷阱与限制)）。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `bird_egg` | `defaultfn` → `commonfn("idle", true)`（`cookable` 为真）；`SetPristine` 后非主模拟直接返回 | 生蛋：`catfood`＋`cookable` 标签、主模拟挂 `edible`／`cookable`／`perishable`／`stackable`／`bait`／`inspectable`／`inventoryitem`／`tradable`，最后写 9.375／0／0（饥饿／生命／理智）、`PERISH_MED`、`rocktribute = 1`、`floater` 缩放 0.55／0.5／0.55。[生蛋](../../../raw/current/scripts/prefabs/egg.lua#L83-L101) |
| `bird_egg_cooked` | `cookedfn` → `commonfn("cooked")`（**不传** `cookable`） | 熟蛋：没有 `cookable` 标签也没有同名组件，因此不能再烤；`catfood`／`stackable`／`bait`／`tradable`（`goldvalue` 仍为 1）由共用工厂保留，腐坏改指 `spoiled_food`。[熟态](../../../raw/current/scripts/prefabs/egg.lua#L103-L119) |
| `rottenegg` | `rottenfn` 独立工厂，与上两者不共用 `commonfn` | 腐坏替换目标；另有 `cattoy`／`show_spoiled`／`spoiledfood` 标签、肥料与燃料用途，本页不展开。[腐烂蛋](../../../raw/current/scripts/prefabs/egg.lua#L134-L199) |

`commonfn` 的固定动作：写 bank `birdegg`／build `bird_eggs`／动画；`catfood` 标签与（可选）`cookable` 标签都在 `SetPristine` **之前**加；玩法组件（`edible`／`perishable`／`stackable`／`cookable` 等）只在主模拟分支里挂，两端共有的是标签与 `floater`；`perishable` 先写 `PERISH_FAST`＋`StartPerishing`＋替换 `rottenegg`，生蛋随后在 `defaultfn` 覆盖为 `PERISH_MED`，熟蛋则覆盖替换为 `spoiled_food` 并保持 `PERISH_FAST`。[共用工厂](../../../raw/current/scripts/prefabs/egg.lua#L25-L81)

### 名称核验

简中实显：`bird_egg`＝**鸟蛋**（英文 Egg）、`bird_egg_cooked`＝**熟鸟蛋**（Cooked Egg）、`rottenegg`＝**腐烂鸟蛋**（Rotten Egg），三个键各命中一次，注册名与 `STRINGS.NAMES` 键同名。[鸟蛋](../../../raw/current/scripts/languages/chinese_s.po#L339465-L339467)、[熟鸟蛋](../../../raw/current/scripts/languages/chinese_s.po#L339470-L339472)、[腐烂鸟蛋](../../../raw/current/scripts/languages/chinese_s.po#L348800-L348802)

## 数值与获得

时间基准：`seg_time`＝30 秒、`total_day_time`＝480 秒、`perish_warp`＝1，因此鲜度按 480 秒／天换算。[基准](../../../raw/current/scripts/tuning.lua#L19-L20)、[`perish_warp`](../../../raw/current/scripts/tuning.lua#L47-L47)

| 注册 | 饥饿／生命／理智 | 鲜度与腐坏产物 | 堆叠 | 其它 |
|---|---|---|---|---|
| `bird_egg` | 9.375／0／0（`CALORIES_TINY`） | `PERISH_MED`＝10 天＝4800 秒 → `rottenegg` | 40（`STACK_SIZE_SMALLITEM`） | `catfood`、`cookable`、`bait`、`tradable.goldvalue`＝1、`rocktribute`＝1 |
| `bird_egg_cooked` | 12.5／0／0（`CALORIES_SMALL`） | `PERISH_FAST`＝6 天＝2880 秒 → `spoiled_food` | 40 | `catfood`、`bait`、`tradable.goldvalue`＝1；无 `cookable` |

数值来源：[生蛋](../../../raw/current/scripts/prefabs/egg.lua#L90-L95)、[熟蛋](../../../raw/current/scripts/prefabs/egg.lua#L110-L116)、[堆叠常量](../../../raw/current/scripts/tuning.lua#L83-L83)、[鲜度档位](../../../raw/current/scripts/tuning.lua#L2046-L2048)、[热量档位](../../../raw/current/scripts/tuning.lua#L2058-L2059)。采集册同项写作 `stacksize=40`、`hungervalue=9.375／12.5`、`perishable=4800／2880`，与源码一致。[采集册](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L170-L171)

### 获得路径

| 来源 | 成立条件 | 产出与证据 |
|---|---|---|
| 鸟笼 `birdcage` | 普通鸟接受任意带 `edible` 的物品：其 `<prefab>_seeds` 存在、或 prefab 是 `seeds`、或名字含 `_seeds`、或食性为 `FOODTYPE.MEAT`；再剔除拒收表（蛋类与 `woby_treat`）。**带 `bird_mutant_rift` 的鸟整条规则被覆盖：只收 `LUNAR_SHARDS`，且 `moonglass_charged` 在 brilliance 冷却中拒收。** 投入后 60 帧（约 2 秒）结算 | 食性 MEAT → `bird_egg`（带 `bird_mutant` 的鸟改出 `rottenegg`）；非肉分支里带 `bird_mutant` 的鸟 → `spoiled_food`；否则 `<prefab>_seeds` 存在 → 该种子（如胡萝卜 → `carrot_seeds`）；都不满足才按 `FEED_BIRD_POOP_CHANCE` 判鸟粪。[结算](../../../raw/current/scripts/prefabs/birdcage.lua#L106-L153)、[拒收表](../../../raw/current/scripts/prefabs/birdcage.lua#L30-L39)、[接收条件](../../../raw/current/scripts/prefabs/birdcage.lua#L155-L181)、[投喂延迟](../../../raw/current/scripts/prefabs/birdcage.lua#L190-L216) |
| 企鹅 `penguin` | 冬季筑巢期（`nesting`）在巢区尝试下蛋，约每 `TOTAL_DAY_TIME/5` 检查一次，条件不满足就放弃本次 | `inst.eggprefab`＝`bird_egg`，生成的蛋另加 `penguin_egg` 标签。[配置](../../../raw/current/scripts/prefabs/penguin.lua#L330-L330)、[下蛋流程](../../../raw/current/scripts/brains/penguinbrain.lua#L242-L274) |

去向：① 烹饪锅食材 `egg = 1`，同一行的 `cancook` 会派生 `bird_egg_cooked` 条目（`egg = 1`＋`precook = 1`）[食材值](../../../raw/current/scripts/cooking.lua#L130-L130)、[派生规则](../../../raw/current/scripts/cooking.lua#L46-L72)；② 烤制产物见下节；③ 蚁狮祭品——`rocktribute = 1` 通过蚁狮的接收测试，并按 `ANTLION_TRIBUTE_TO_RAGE_TIME` 平息怒气[接收](../../../raw/current/scripts/prefabs/antlion.lua#L115-L119)、[结算](../../../raw/current/scripts/prefabs/antlion.lua#L141-L142)；④ 浣猫把 `catfood` 当猫粮接收[浣猫](../../../raw/current/scripts/prefabs/catcoon.lua#L273-L279)；⑤ 配方 `book_birds` 消耗 2 鸟蛋＋2 纸莎草（`TECH.NONE`、`builder_tag = "bookbuilder"`），是蛋的去向而不是来源[配方](../../../raw/current/scripts/recipes.lua#L275-L275)。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：edible | 写 `FOODTYPE.MEAT` 与三维（饥饿／生命／理智）；生蛋 9.375／0／0，熟蛋 12.5／0／0 | 仅主模拟；`foodtype` 赋值同步 `edible_MEAT` 标签 | [[reference/components/edible]]；[生蛋](../../../raw/current/scripts/prefabs/egg.lua#L53-L54)、[熟蛋](../../../raw/current/scripts/prefabs/egg.lua#L110-L111) |
| 组件：perishable | 两注册各写替换值；`SetPerishTime` 同时重置剩余时间并重启计时 | 主模拟；生蛋被 `defaultfn` 二次覆盖为 `PERISH_MED` | [[reference/components/perishable]]；[赋值](../../../raw/current/scripts/components/perishable.lua#L188-L194) |
| 组件：stackable | `maxsize = TUNING.STACK_SIZE_SMALLITEM`＝40 | 两端由 replica 同步堆叠数 | [[reference/components/stackable]]；[显式 40](../../../raw/current/scripts/prefabs/egg.lua#L68-L69) |
| 组件：cookable | `product = "bird_egg_cooked"`；标签在 pristine 前、组件在主模拟 | 仅生蛋；熟蛋无标签无组件 | [[reference/components/cookable]]；[接线](../../../raw/current/scripts/prefabs/egg.lua#L40-L43)、[产物](../../../raw/current/scripts/prefabs/egg.lua#L56-L59) |
| 组件：tradable | `goldvalue = 1`；生蛋另写 `rocktribute = 1` | 主模拟 | [[reference/components/tradable]]；[赋值](../../../raw/current/scripts/prefabs/egg.lua#L77-L78)、[祭品值](../../../raw/current/scripts/prefabs/egg.lua#L95-L95) |
| 组件：bait | 可作为陷阱诱饵；被吃／被偷时通知占用它的陷阱 | 主模拟；`trap` 字段由陷阱写入 | [[reference/components/bait]]；[挂载](../../../raw/current/scripts/prefabs/egg.lua#L71-L71)、[通知](../../../raw/current/scripts/components/bait.lua#L7-L19) |
| 标签：`catfood` | 让浣猫把物品当猫粮接收 | 两端都在 pristine 前加 | [浣猫](../../../raw/current/scripts/prefabs/catcoon.lua#L273-L279) |
| 关联 Prefab：`rottenegg` | 生蛋的腐坏替换目标；`rottenegg` 自身行为归其源码入口 | 到点由 `perishable` 生成 | [替换](../../../raw/current/scripts/prefabs/egg.lua#L64-L64)、[注册](../../../raw/current/scripts/prefabs/egg.lua#L203-L203) |
| 资源：`anim/bird_eggs.zip` | bank `birdegg`、build `bird_eggs`，生／熟／腐烂共用一套 | 原版资源，不随页分发 | [assets](../../../raw/current/scripts/prefabs/egg.lua#L1-L5) |

### 烤制与燃烧掉落（共享规则 S1／S2 的接入）

- **S1 烤制端**：`cookable` 的 COOK 门禁是目标带 `cooker` 标签且未 `fueldepleted`（危险炉具另需专家厨师、骑乘时要求目标归自己所有）[门禁](../../../raw/current/scripts/componentactions.lua#L1096-L1105)；烹饪锅只有 `stewer` 标签与组件，不是 `cooker`[烹饪锅](../../../raw/current/scripts/prefabs/cookpot.lua#L288-L309)。**在带 `cooker` 的实体（营火、火坑、龙鳞火炉等）上烤鸟蛋**得到 `bird_egg_cooked`。
- **S2 燃烧掉落**：`DropLoot` 仅在掉落者已带 `burnt`，或正在燃烧且（无 `fueled` 或 `burnable.ignorefuel`）时，按 `BURNED_LOOT_OVERRIDES` → `<name>_cooked` → `cooked<name>` 改写[门禁与改写](../../../raw/current/scripts/components/lootdropper.lua#L415-L442)。**本页例外**：鸟蛋自身既无 `lootdropper` 也不带 `burnable`，全脚本未找到把 `bird_egg` 当掉落项的表，因此 S2 在本页没有可触发的原版路径；相近但不同源的一条是**燃烧的企鹅会改写它抱着的蛋**（70% `rottenegg`、30% `bird_egg_cooked`，若原本已是熟蛋则直接 `rottenegg`）[企鹅](../../../raw/current/scripts/prefabs/penguin.lua#L184-L195)。

## 典型接入

### 延长生蛋鲜度

- 前提与选择：服务端 `modmain.lua` 的 postinit。改 `perishable` 实例时间而不是改 `TUNING.PERISH_MED`（后者会连带所有用该档的食物）。
- 生效与撤销：`SetPerishTime` 会同时把剩余时间设为新值并重启计时，所以新生成实例立刻生效；读档实例的剩余时间由 `perishable:OnLoad` 用存档值覆盖，鲜度百分比会按新总时长重新换算。卸模组后新实例回到 `PERISH_MED`。
- 原版依据：接线在 `commonfn` 的 `SetPerishTime`＋`StartPerishing`，`defaultfn` 再覆盖一次[鲜度](../../../raw/current/scripts/prefabs/egg.lua#L61-L64)、[覆盖](../../../raw/current/scripts/prefabs/egg.lua#L93-L93)。

```lua
-- modmain.lua：把生鸟蛋的鲜度从 PERISH_MED 延到 PERISH_SLOW（10 天 → 15 天）
AddPrefabPostInit("bird_egg", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    local perishable = inst.components.perishable
    if perishable ~= nil then
        perishable:SetPerishTime(GLOBAL.TUNING.PERISH_SLOW)
    end
end)
```

- 调整与验证：`PERISH_SLOW`＝15 天＝7200 秒[档位](../../../raw/current/scripts/tuning.lua#L2049-L2049)；实测看新生成鸟蛋的鲜度条与读档后的剩余比例，不要只看组件字段。

## 常见陷阱与限制

- **误区：熟鸟蛋还能回炉烤。** `cookedfn` 调 `commonfn("cooked")` 时没传 `cookable`，既无 `cookable` 标签也无组件，COOK 动作不会出现；熟蛋也没有 `dryable` 组件或标签，进不了晾肉架（生蛋同样没有）。[熟态](../../../raw/current/scripts/prefabs/egg.lua#L103-L119)
- **误区：克劳斯袋能开出鸟蛋。** `klaussackloot` 的巨人战利品池里只有 `tallbirdegg`，没有 `bird_egg`；鸟蛋的可核验来源是鸟笼与企鹅；`book_birds` 消耗鸟蛋，是去向不是来源。[战利品池](../../../raw/current/scripts/components/klaussackloot.lua#L23-L34)
- **边界：鸟笼不接受蛋。** `invalid_foods` 直接列了 `bird_egg`／`bird_egg_cooked`／`rottenegg`／`woby_treat`，即使食性是 `MEAT` 也会被拒。[拒收表](../../../raw/current/scripts/prefabs/birdcage.lua#L30-L39)、[判定](../../../raw/current/scripts/prefabs/birdcage.lua#L168-L170)
- **边界：`bait` 不等于自动进陷阱。** 它只是让陷阱能把物品登记为饵；要由陷阱侧写入 `bait.trap` 后才在被吃／被偷时收到通知。[诱饵组件](../../../raw/current/scripts/components/bait.lua#L21-L36)
- **未核验**：本页未运行游戏；鸟笼出蛋率、企鹅下蛋的时间分布、以及 `book_birds` 的实际出鸟行为都只按源码推导。

## 验证记录

- 源码审阅：2026-11-09，Steam buildid 26474873。核对 `prefabs/egg.lua` 的 `commonfn`／`defaultfn`／`cookedfn`／`rottenfn` 与三个注册；`perishable` 的 `SetPerishTime`／`OnLoad`、`edible` 默认值、`tradable` 默认 `goldvalue`、`bait` 通知链；`componentactions` 的 COOK 门禁与 `lootdropper` 的燃烧换名；`birdcage` 的接收／拒收与结算、`penguin`／`penguinbrain` 的下蛋与燃烧改写、`recipes` 的 `book_birds`、`cooking` 的食材值与 `cancook` 派生、`antlion` 的祭品测试；`tuning` 的时间／鲜度／热量／堆叠常量与 `scrapbookdata` 交叉核对。
- 名称核验：`languages/chinese_s.po` 的 `NAMES.BIRD_EGG`＝鸟蛋、`NAMES.BIRD_EGG_COOKED`＝熟鸟蛋、`NAMES.ROTTENEGG`＝腐烂鸟蛋。
- 未覆盖：`rottenegg` 的肥料／燃料／可放置行为、`bird_mutant`／`bird_mutant_rift` 分支的 brilliance 产出与冷却参数、`book_birds` 生成的鸟群、以及 `preparedfoods` 里把鸟蛋仅作展示卡（`card_def`）的条目。
- 游戏运行：未运行。
- 结构检查：`validate_page.py` 通过（standard 档内）；`validate_source_links.py --strict` 0 错 0 警。

## 相关链接

- 可食与鲜度：[[reference/components/edible]]、[[reference/components/perishable]]、[[reference/components/stackable]]
- 两条转换：[[reference/components/cookable]]、[[reference/components/tradable]]、[[reference/components/bait]]
- 同批蛋类：[[reference/prefabs/tallbird-egg]]

返回 [[reference/prefabs/index|Prefab 索引]]。
