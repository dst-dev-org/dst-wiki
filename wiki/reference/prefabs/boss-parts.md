---
title: "大型生物掉落器官：独眼巨鹿眼球与守护者之角（deerclops_eyeball、minotaurhorn）"
created: "2026-11-10"
updated: "2026-11-10"
type: prefab
summary: "独眼巨鹿眼球与守护者之角的对照页：两者都是 10 叠、med／small 漂浮、作祟只弹飞的可食物品，edible 显式写 60／75／−15 与 MEAT（不是默认 10／10／0）；眼球另挂 tradable（goldvalue 保持默认 0，猪王与蚁狮都拒收）、雪人装饰表项与 deerclops_eyeball 标签，角没有这些。本页汇总独眼巨鹿／远古守护者掉落、克劳斯袋、眼球伞与眼睛炮塔配方及拆解，并写清哨戒设施的收存与归还。"
aliases: ["deerclops_eyeball", "minotaurhorn", "独眼巨鹿眼球", "守护者之角", "Deerclops Eyeball", "Guardian's Horn", "眼球帽", "眼球伞", "眼球炮台", "眼睛炮塔", "Houndius Shootius"]
tags: [prefab, item, food]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/deerclops_eyeball.lua", "prefabs/minotaurhorn.lua", "prefabs/deerclops.lua", "prefabs/minotaur.lua", "stategraphs/SGdeerclops.lua", "stategraphs/SGminotaur.lua", "stategraphs/commonstates.lua", "components/edible.lua", "components/tradable.lua", "components/snowmandecoratable.lua", "components/snowmandecor.lua", "components/stackable.lua", "components/inventoryitemholder.lua", "components/cooker.lua", "components/lootdropper.lua", "components/burnable.lua", "standardcomponents.lua", "componentutil.lua", "entityscript.lua", "prefabs/corpses.lua", "prefabs/corpses_defs.lua", "prefabs/eyeturret.lua", "prefabs/deerclopseyeball_sentryward.lua", "prefabs/pigking.lua", "prefabs/antlion.lua", "components/trader.lua", "prefabs/staff_common.lua", "components/spellcaster.lua", "componentactions.lua", "recipes.lua", "constants.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 大型生物掉落器官：独眼巨鹿眼球与守护者之角（deerclops_eyeball、minotaurhorn）

## 快速判断

- 职责与适用：两个独立文件、各自 `Prefab()` 注册的可食器官——独眼巨鹿眼球（`deerclops_eyeball`）与守护者之角（`minotaurhorn`）。两者都靠大型生物掉落、都能直接吃（MEAT 类 60／75／−15），主要用途分别是眼球伞／眼睛炮塔与眼睛炮塔。[眼球注册](../../../raw/current/scripts/prefabs/deerclops_eyeball.lua#L6-L49)、[角注册](../../../raw/current/scripts/prefabs/minotaurhorn.lua#L6-L43)
- 优先使用：查两者构造与共性看[注册与构造](#注册与构造)；查可食值、食性门禁与"能不能进锅"看[可食与食用门禁](#可食与食用门禁)；查交易、雪人与哨戒设施看[关联机制导航](#关联机制导航)；查配方与拆解看[用途与配方](#用途与配方)。
- 重要副作用：两者都显式设置 `edible` 三维，**不是组件默认的 10／10／0**；理智为负会带 `badfood` 标签。眼球挂 `tradable` 但 `goldvalue` 保持默认 0，猪王与蚁狮都会拒收且不吞物品；眼球还能作为哨戒设施的"钥匙"被收存，取出或锤拆时归还。[edible 默认](../../../raw/current/scripts/components/edible.lua#L19-L33)、[交易门禁](../../../raw/current/scripts/prefabs/pigking.lua#L598-L606)
- 修改边界：堆叠／漂浮／作祟的共性以 [[reference/prefabs/boss-materials#注册与构造]] 为主章节；本页只写器官特有的 edible／tradable／雪人／标签差异。宿主战斗、生成与尸体系统的通用规则见 [[reference/prefabs/boss-materials#获得]]，本页只列本对象的结果。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `deerclops_eyeball` | 单文件单注册 `Prefab("deerclops_eyeball", fn, assets)`，无包装工厂。公共端：`CreateEntity` → Transform／AnimState／Network → `MakeInventoryPhysics` → bank／build `deerclops_eyeball` → `idle` → `MakeInventoryFloatable("small", 0.2, 0.95)` → `AddTag("deerclops_eyeball")` → `SetPristine`；非主模拟返回。 | 主模拟：`inspectable` → `tradable` → `inventoryitem` → `edible`（MEAT，60／75／−15）→ `stackable`（10）→ `snowmandecor` → `MakeHauntableLaunch`。[全文](../../../raw/current/scripts/prefabs/deerclops_eyeball.lua#L6-L49) |
| `minotaurhorn` | 同样的单文件单注册；两端都不建 `SoundEmitter`（公共端只有 Transform／AnimState／Network 与 `MakeInventoryPhysics`），无标签；floatable 为 `("med", 0.05, 0.75)`。 | 主模拟：`inspectable` → `inventoryitem` → `stackable`（10）→ `edible`（MEAT，60／75／−15）→ `MakeHauntableLaunch`。**没有** `tradable`／`snowmandecor`／自定义 tag。[全文](../../../raw/current/scripts/prefabs/minotaurhorn.lua#L6-L43) |

**同批共性**：这两种器官的堆叠 10（`TUNING.STACK_SIZE_LARGEITEM`）、漂浮参数、`MakeHauntableLaunch` 只弹飞、以及"物品本身无 `burnable`"的结论，以 [[reference/prefabs/boss-materials#注册与构造]] 的四行共性表为准；本页不重复。[常量](../../../raw/current/scripts/tuning.lua#L81-L83)、[作祟 helper](../../../raw/current/scripts/standardcomponents.lua#L1053-L1069)

### 名称与实显

`NAMES.DEERCLOPS_EYEBALL`＝**独眼巨鹿眼球**（Deerclops Eyeball）；`NAMES.MINOTAURHORN`＝**守护者之角**（Guardian's Horn）。两个文件都不设 `nameoverride`，显示名走实体默认的 `STRINGS.NAMES[注册名大写]`。[眼球键](../../../raw/current/scripts/languages/chinese_s.po#L342089-L342092)、[角键](../../../raw/current/scripts/languages/chinese_s.po#L345484-L345487)、[默认取名](../../../raw/current/scripts/entityscript.lua#L840-L843)

## 可食与食用门禁

| 项 | 眼球 | 角 | 证据 |
|---|---|---|---|
| `edible` 默认（仅挂组件） | 10／10／0，`GENERIC` | 同左 | [默认值](../../../raw/current/scripts/components/edible.lua#L19-L33) |
| 本页显式值 | 生命 `HEALING_HUGE`＝60；饥饿 `CALORIES_HUGE`＝75；理智 `−SANITY_MED`＝−15；`FOODTYPE.MEAT` | 完全相同 | [眼球 edible](../../../raw/current/scripts/prefabs/deerclops_eyeball.lua#L33-L37)、[角 edible](../../../raw/current/scripts/prefabs/minotaurhorn.lua#L32-L36)、[常量](../../../raw/current/scripts/tuning.lua#L2028-L2063) |
| 食物标签 | `edible_MEAT`（`foodtype` setter 生成） | 同左 | [setter](../../../raw/current/scripts/components/edible.lua#L9-L17) |
| 负面标签 | `badfood`（理智为负） | 同左 | [判定](../../../raw/current/scripts/components/edible.lua#L1-L7) |
| 能否烤 | 否：无 `cookable`，篝火／火坑的 `Cooker:CanCook` 要求该组件 | 同左 | [门禁](../../../raw/current/scripts/components/cooker.lua#L12-L17) |
| 能否进锅 | 否：`cooking.lua` 未登记这两个 prefab；锅的过滤器只查 `IsCookingIngredient` | 同左 | [过滤器](../../../raw/current/scripts/containers.lua#L373-L375)、[登记入口示例](../../../raw/current/scripts/cooking.lua#L151-L151) |
| 鲜度／干燥／诱饵 | 都没有 `perishable`／`dryable`／`bait` | 同左 | [眼球全文](../../../raw/current/scripts/prefabs/deerclops_eyeball.lua#L6-L49)、[角全文](../../../raw/current/scripts/prefabs/minotaurhorn.lua#L6-L43) |

- **默认值只在"不赋值"时生效**：`edible` 构造默认 10／10／0、`GENERIC`；这两个文件把三维和 `foodtype` 全部重写，所以实际收益是 60／75／−15、MEAT，不能反推"源码没写就是 0"。[默认](../../../raw/current/scripts/components/edible.lua#L19-L33)、[显式](../../../raw/current/scripts/prefabs/deerclops_eyeball.lua#L33-L37)
- **EAT 动作还要看吃的人**：动作收集遍历 `FOODGROUP`／`FOODTYPE`，要求物品带 `edible_MEAT` 且角色带对应的 `MEAT_eater`（或所属食性组覆盖）。所以"有 edible 组件"不等于所有角色都能吃。[动作门禁](../../../raw/current/scripts/componentactions.lua#L2672-L2689)
- **眼睛和角都不是锅料**：全 scripts 检索这两个注册名，`cooking.lua` 没有 `AddIngredientValues` 命中；也没有 `cookable`，两条烹饪入口都关闭。检索范围：`scripts/**/*.lua` 的注册名全文；不排除模组另行登记。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`edible` | 三维与 foodtype 显式写入；眼球和角都是 MEAT | 主模拟；EAT 在服务端结算 | [[reference/components/edible#关键接口]]；[眼球](../../../raw/current/scripts/prefabs/deerclops_eyeball.lua#L33-L37)、[角](../../../raw/current/scripts/prefabs/minotaurhorn.lua#L32-L36) |
| 组件：`tradable` | 只有眼球挂；没有赋 `goldvalue`／`tradefor`／`rocktribute` | 主模拟；猪王读 `goldvalue`、蚁狮读 `rocktribute` | [[reference/components/tradable#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/deerclops_eyeball.lua#L30-L30)、[默认 0](../../../raw/current/scripts/components/tradable.lua#L1-L7) |
| API／数据：猪王门禁 | `AcceptTest` 要求 `goldvalue > 0`（另有万圣糖果值与 `pig_token` 例外）；眼球是 0，因此拒收并走 `onrefuse`，不扣物品 | 服务端；鱼人交付者另有拒绝分支 | [接受测试](../../../raw/current/scripts/prefabs/pigking.lua#L598-L606)、[拒绝流程](../../../raw/current/scripts/components/trader.lua#L123-L163) |
| API／数据：蚁狮门禁 | `AcceptTest` 要求 `rocktribute > 0`（另有「无待发奖励且未索战」前置）；眼球没有该字段，同样拒收 | 服务端 | [接受测试](../../../raw/current/scripts/prefabs/antlion.lua#L115-L119) |
| 组件：`snowmandecor` | 只有眼球挂空标记组件；渲染数据在 `snowmandecoratable.ITEM_DATA[hash("deerclops_eyeball")]`，带 `canflip = true` | 物品侧组件在主模拟；表项按注册名 hash，客户端也要有 | [[reference/components/snowmandecoratable#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/deerclops_eyeball.lua#L42-L42)、[表项](../../../raw/current/scripts/components/snowmandecoratable.lua#L16-L16) |
| 标签：`deerclops_eyeball` | 眼球在 `SetPristine` 前加自定义标签；当前唯一消费者是哨戒设施的 `SetAllowedTags` | 双端可见 | [加标签](../../../raw/current/scripts/prefabs/deerclops_eyeball.lua#L21-L21)、[设施白名单](../../../raw/current/scripts/prefabs/deerclopseyeball_sentryward.lua#L485-L488) |
| 组件：`inventoryitemholder` | 哨戒设施用白名单收存眼球；给入激活、取出归还，锤拆时 `TakeItem()` 把眼球丢回场景 | 服务端 | [[reference/components/inventoryitemholder#关键接口]]；[给入](../../../raw/current/scripts/prefabs/deerclopseyeball_sentryward.lua#L153-L188)、[取出](../../../raw/current/scripts/prefabs/deerclopseyeball_sentryward.lua#L196-L227)、[锤拆](../../../raw/current/scripts/prefabs/deerclopseyeball_sentryward.lua#L116-L128) |
| 关联 Prefab：`deerclopseyeball_sentryward_kit`／`deerclopseyeball_sentryward` | 设施本体 B34 待建；建造套件配方不含眼球，眼球是另交存的钥匙 | 建造后再交存 | Wiki 待建；[套件配方](../../../raw/current/scripts/recipes.lua#L858-L858)、[设施](../../../raw/current/scripts/prefabs/deerclopseyeball_sentryward.lua#L14-L24) |
| 关联 Prefab：`eyeturret`／`eyeturret_item` | 眼睛炮塔的已放置／可放置两形态；本体 Wiki 待建，弹体与战斗见 eye_charge 页 | 放置后由 `eyeturret` 执行 | Wiki 待建；[[reference/prefabs/eye_charge]]；[放置](../../../raw/current/scripts/prefabs/eyeturret.lua#L152-L164)、[注册](../../../raw/current/scripts/prefabs/eyeturret.lua#L351-L353) |
| 关联 Prefab：`klaus_sack` | 眼球的副来源（`giant_loot1` 五选一）；角不在袋里 | 正确钥匙开袋 | [抽取](../../../raw/current/scripts/components/klaussackloot.lua#L116-L124) |

## 获得

| 材料 | 来源 | 产出 | 条件／概率 | 证据 |
|---|---|---|---|---|
| `deerclops_eyeball` | 独眼巨鹿（`deerclops`） | 1 个 | 普通共享表一条 1.00；同表另有 8 个 `meat` 与雕塑草图 | [表](../../../raw/current/scripts/prefabs/deerclops.lua#L329-L341)、[绑定](../../../raw/current/scripts/prefabs/deerclops.lua#L511-L511) |
| `deerclops_eyeball` | 克劳斯袋（正确钥匙开启） | 0–1 个 | 每次开袋的常规 4 个包裹中（冬宴另加 2 个），有 1 个固定含 1 件取自 5 项 `giant_loot1`，眼球是其中一项；非保底 | [抽取](../../../raw/current/scripts/components/klaussackloot.lua#L116-L124)、[开袋](../../../raw/current/scripts/prefabs/klaus_sack.lua#L66-L68) |
| `deerclops_eyeball` | 拆解眼球伞（`eyebrellahat`）回收 | 至少 1 个 | 眼球伞有普通配方，绿色法杖按 `AllRecipes` 拆解；数量按 fueled 百分比折算但每项至少 1 | [拆解公式](../../../raw/current/scripts/prefabs/staff_common.lua#L637-L674)、[[reference/prefabs/staffs#拆解链greenstaff]] |
| `minotaurhorn` | 远古守护者（`minotaur`） | 1 个 | 普通共享表一条 1.00；同表另有 8 个 `meat`、蓝图与雕塑草图 | [表](../../../raw/current/scripts/prefabs/minotaur.lua#L49-L62)、[绑定](../../../raw/current/scripts/prefabs/minotaur.lua#L570-L570) |
| `minotaurhorn` | 拆解眼睛炮塔（`eyeturret`／`eyeturret_item`）回收 | 1 个 | 已放置形态有显式 `DeconstructRecipe("eyeturret")`；未放置的物品按普通配方拆解 | [拆解配方](../../../raw/current/scripts/recipes.lua#L1681-L1681)、[拆解公式](../../../raw/current/scripts/prefabs/staff_common.lua#L637-L674) |

- **没有其它生成点**：角只从远古守护者掉落；眼球的生成点只有独眼巨鹿、克劳斯袋与拆解回收，哨戒设施只是归还已交存的眼球。变异独眼巨鹿的表不含眼球。[独眼巨鹿表](../../../raw/current/scripts/prefabs/deerclops.lua#L329-L341)、[克劳斯袋](../../../raw/current/scripts/components/klaussackloot.lua#L1-L8)、[变异表](../../../raw/current/scripts/prefabs/deerclops.lua#L343-L354)
- **死亡入口**：`DropDeathLoot` 在 `is_corpsing` 为真时显式 `GenerateLoot()` 并转存给 `*corpse`；为假时只把位置交给 `lootdropper:DropLoot(pos)`，由后者生成。本页两宿主默认走直接掉落。[DropDeathLoot](../../../raw/current/scripts/entityscript.lua#L2181-L2192)、[转移](../../../raw/current/scripts/componentutil.lua#L1563-L1599)
- **成尸资格**：`EntityHasCorpse`（SG 有 `corpse` 且无 `sg.mem.nocorpse`）→ `forcecorpse` → 燃烧中阻止其余分支 → 尸体持久化（本版只由变异秃鹫管理器注册）／月变。本页两宿主没有 `forcecorpse`，所以普通击杀通常直接掉落，尸体是条件路径；不要把“未燃烧”放在 `forcecorpse` 之前。[状态图前置](../../../raw/current/scripts/componentutil.lua#L1472-L1475)、[资格顺序](../../../raw/current/scripts/componentutil.lua#L1546-L1561)、[唯一注册点](../../../raw/current/scripts/components/mutatedbuzzardmanager.lua#L377-L409)
- **死亡与烧毁**：这两个宿主都可燃；若击杀时宿主仍在燃烧，死亡结算会走 `DropLoot` 的烧毁改写。改写条件为"带 `burnt`，或正在燃烧且（`fueled` 为 nil 或 `burnable.ignorefuel` 为真）"；顺序为 `BURNED_LOOT_OVERRIDES` → `_cooked` → `cooked` → 受控燃烧保留 → 其余非结构且非 `tree`／`boulder`（或蜂巢）换 `ash`。眼球／角没有覆盖项与熟制变体，所以普通燃烧各变成 1 个 `ash`，受控燃烧保留原名正常落地（它们自身没有 `burnable`，不会冒烟）。若走尸体路径，尸体燃尽时同样改写。通用规则与"何时才有尸体"见 [[reference/prefabs/boss-materials#获得]]。[改写块](../../../raw/current/scripts/components/lootdropper.lua#L418-L442)、[冒烟条件](../../../raw/current/scripts/components/lootdropper.lua#L404-L407)、[鹿可燃](../../../raw/current/scripts/prefabs/deerclops.lua#L430-L430)、[守护者可燃](../../../raw/current/scripts/prefabs/minotaur.lua#L586-L586)
- **尸体结算**：`DropCorpseLoot` 把处理后的 `corpse_loot` 作为 `DropLoot` 的显式列表传入，不会重掷；燃尽走 `OnBurnt`（`no_destroy_on_burn` 时跳过）；熄灭先由 `Burnable:Extinguish` 清 `burning`／`controlled_burn` 再回调，`OnExtinguish` 在未变异且无 `_skip_extinguish_fade`／`no_destroy_on_burn` 时才走 `DefaultExtinguishCorpseFn`→`DropCorpseLoot`。[燃尽](../../../raw/current/scripts/prefabs/corpses.lua#L38-L43)、[熄灭](../../../raw/current/scripts/prefabs/corpses.lua#L45-L52)、[提交](../../../raw/current/scripts/prefabs/corpses.lua#L489-L495)、[Extinguish](../../../raw/current/scripts/components/burnable.lua#L465-L490)

## 用途与配方

### 独眼巨鹿眼球

| 用途／产物 | 配方 | 科技／条件 | 证据 |
|---|---|---|---|
| 眼球伞（`eyebrellahat`，俗称眼球帽） | 眼球×1＋`twigs`×15＋`boneshard`×4 | `TECH.SCIENCE_TWO` | [配方](../../../raw/current/scripts/recipes.lua#L554-L554)、[[reference/prefabs/hats-environment]] |
| 眼睛炮塔（`eyeturret_item`，Houndius Shootius） | 眼球×1＋`minotaurhorn`×1＋`thulecite`×5 | `TECH.ANCIENT_FOUR`＋`nounlock` | [配方](../../../raw/current/scripts/recipes.lua#L810-L810) |
| 已放置炮塔拆解（`eyeturret`） | 眼球×1＋角×1＋`thulecite`×5 | `DeconstructRecipe`；绿色法杖 | [拆解表](../../../raw/current/scripts/recipes.lua#L1681-L1681) |
| 哨戒设施（`deerclopseyeball_sentryward`） | 不消耗：作为白名单物品交存，激活设施；取出／锤拆归还 | 需先建造设施；套件配方不含眼球 | [白名单](../../../raw/current/scripts/prefabs/deerclopseyeball_sentryward.lua#L485-L488)、[激活](../../../raw/current/scripts/prefabs/deerclopseyeball_sentryward.lua#L153-L188) |

### 守护者之角

| 用途／产物 | 配方 | 科技／条件 | 证据 |
|---|---|---|---|
| 眼睛炮塔（`eyeturret_item`） | 角×1＋眼球×1＋`thulecite`×5 | `TECH.ANCIENT_FOUR`＋`nounlock` | [配方](../../../raw/current/scripts/recipes.lua#L810-L810) |
| 已放置炮塔拆解（`eyeturret`） | 角×1＋眼球×1＋`thulecite`×5 | `DeconstructRecipe`；绿色法杖 | [拆解表](../../../raw/current/scripts/recipes.lua#L1681-L1681) |

- **科技门禁**：`TECH.SCIENCE_TWO`＝`SCIENCE = 2`；`TECH.ANCIENT_FOUR`＝`ANCIENT = 4`。眼睛炮塔另带 `nounlock = true`，不通过原型学习，只在提供该配方的制作站出现；眼球伞没有该标记，按二本正常解锁。[科技表](../../../raw/current/scripts/constants.lua#L1241-L1254)、[builder 站内解锁](../../../raw/current/scripts/components/builder.lua#L276-L298)
- **放置链**：`eyeturret_item` 的 `ondeploy` 生成 `eyeturret`，把物品形态替换成已放置形态；已放置形态另有自己的掉落表（铥矿与铥矿碎片），与拆解配方不是同一条。[放置](../../../raw/current/scripts/prefabs/eyeturret.lua#L152-L164)、[本体掉落](../../../raw/current/scripts/prefabs/eyeturret.lua#L16-L24)
- **拆解的两条入口**：`eyebrellahat` 没有专门 `DeconstructRecipe`，但绿色法杖的 `destroystructure` 直接取 `AllRecipes[target.prefab]`（普通配方即可），满耐久时按"眼球 1＋树枝 15＋骨片 4"返还，实际各材料乘 fueled 百分比后向上取整，眼球因数量 1 仍至少 1；`eyeturret` 则用显式 `DeconstructRecipe` 返回同样三件。`eyeturret_item` 是普通配方物品，也按普通配方拆。[施法门禁](../../../raw/current/scripts/components/spellcaster.lua#L190-L191)、[返还公式](../../../raw/current/scripts/prefabs/staff_common.lua#L637-L674)
- **交易与雪人**：眼球有 `tradable` 但没有价值字段，猪王与蚁狮都拒收；只有眼球能在雪人上装饰（`canflip = true`），角没有组件也没有表项。[猪王](../../../raw/current/scripts/prefabs/pigking.lua#L598-L606)、[蚁狮](../../../raw/current/scripts/prefabs/antlion.lua#L115-L119)、[表项](../../../raw/current/scripts/components/snowmandecoratable.lua#L16-L16)

## 常见陷阱与限制

- **误区：没写三维就是 0**——`edible` 构造默认是 10／10／0；这两个文件把生命、饥饿、理智全部显式写成 60／75／−15。只有"只挂组件、不赋值"的物品才拿默认值。[默认](../../../raw/current/scripts/components/edible.lua#L19-L33)、[显式](../../../raw/current/scripts/prefabs/deerclops_eyeball.lua#L33-L37)
- **误区：挂 `tradable` 就能卖给猪王**——`goldvalue` 默认 0，猪王 `AcceptTest` 要求 >0（另有万圣糖果值与 `pig_token` 例外）；蚁狮要求 `rocktribute > 0`（另有「无待发奖励且未索战」前置），眼球两者都没有。拒收走 `WantsToAccept` 的失败分支，物品不会被吞。[猪王](../../../raw/current/scripts/prefabs/pigking.lua#L598-L606)、[蚁狮](../../../raw/current/scripts/prefabs/antlion.lua#L115-L119)、[trader](../../../raw/current/scripts/components/trader.lua#L123-L163)
- **误区：有 `edible` 就能进锅或上火烤**——进锅要 `cooking.IsCookingIngredient` 登记，这两件没有；火烤要 `cookable`，也没有。它们只能直接吃。[过滤器](../../../raw/current/scripts/containers.lua#L373-L375)、[烤制门禁](../../../raw/current/scripts/components/cooker.lua#L12-L17)
- **边界：理智为负会带 `badfood`**——两件都是 −15 理智，`oncheckbadfood` 会加 `badfood` 标签；这不影响食用资格，但会影响依赖该标签的消费端。[判定](../../../raw/current/scripts/components/edible.lua#L1-L7)
- **边界：哨戒设施认标签不认 prefab 名**——`inventoryitemholder:SetAllowedTags({"deerclops_eyeball"})` 只检查物品是否带该标签；任何自定义物品只要带同标签也能被收存。这不是"只收原版眼球"的硬编码。[白名单](../../../raw/current/scripts/prefabs/deerclopseyeball_sentryward.lua#L485-L488)、[判定](../../../raw/current/scripts/components/inventoryitemholder.lua#L89-L104)
- **边界：哨戒设施不消耗眼球**——给入走 `GiveItem`（眼球被移出场景、挂到设施上），取出走 `TakeItem`（无接受者时原地掉落）；锤拆时也调用 `TakeItem()` 归还。[给入](../../../raw/current/scripts/components/inventoryitemholder.lua#L112-L163)、[取出](../../../raw/current/scripts/components/inventoryitemholder.lua#L165-L209)、[锤拆](../../../raw/current/scripts/prefabs/deerclopseyeball_sentryward.lua#L116-L128)
- **未核验：运行时食用、交易拒绝与拆解返还**；本页数值与门禁均为源码推导，未运行游戏。

## 验证记录

- 源码审阅：2026-11-10，Steam buildid 26474873。核验范围：两份器官 prefab 全文；`edible` 默认值／显式三维／foodtype setter／`badfood`；EAT 动作门禁与 `Cooker:CanCook`；`cooking.lua` 的登记入口与锅的 `itemtestfn`（两注册均无登记）；`tradable` 默认值与猪王 `AcceptTest`／蚁狮 `AcceptTest`／trader 的接受—拒绝流程；雪人装饰组件与 `ITEM_DATA` 表项；`deerclops_eyeball` 标签与哨戒设施白名单、给入／取出／锤拆；独眼巨鹿与远古守护者的共享表、`SetChanceLootTable`、`deerclops_eyeball`／`minotaurhorn` 的掉落行；克劳斯袋 `giant_loot1`；`recipes.lua` 的眼球伞、眼睛炮塔与拆解行；`TECH.SCIENCE_TWO`／`TECH.ANCIENT_FOUR`／`nounlock`；绿色法杖拆解公式；简中名称键。
- 2026-11-10 检视修订（R2）：R01 烧毁改写补受控燃烧保留、`ignorefuel` 与"自身无 burnable 不冒烟"；R05 改为两端都不建 `SoundEmitter`；R06 统一"雕塑草图"；R07 取出改记 `TakeItem`；R09 补猪王万圣糖果／`pig_token` 与蚁狮「无待发奖励且未索战」前置。
- 2026-11-10 外部检视修订（R03）：死亡入口、成尸资格顺序、燃尽／熄灭与尸体列表不重掷分写；熄灭先清燃烧／受控标记再回调，变异或跳过／保护标记下不走普通掉落。
- 未覆盖：两个宿主的战斗与刷新；`deerclopseyeball_sentryward` 本体完整机制（B34 待建）；眼球伞／炮塔的运行时拆解表现；各角色食性名单；模组自建物品带同标签后的行为。
- 游戏运行：未运行；三维、交易拒绝、拆解数量与烧毁结果均为源码推导。
- Lua 示例：无（本页未新增可执行 Lua 示例，故未做 Lua 语法检查）。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 见交付回报。

相关：[[reference/prefabs/boss-materials]]（同批材料与宿主掉落通用规则）、[[reference/prefabs/hats-environment]]（眼球伞本体）、[[reference/prefabs/staffs]]（拆解链）、[[reference/components/edible]]、[[reference/components/tradable]]、[[reference/components/snowmandecoratable]]、[[reference/components/inventoryitemholder]]、[[reference/prefabs/eye_charge]]。

返回 [[reference/prefabs/index|原版 Prefab 索引]]。