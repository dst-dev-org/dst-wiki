---
title: "蜘蛛丝（silk）"
created: "2026-11-10"
updated: "2026-11-10"
type: prefab
summary: "prefabs/silk.lua 单注册的纺织材料（简中“蜘蛛丝”）：堆叠 40、cattoy 标签、可交易但 goldvalue 为 0、默认浮水、作祟弹飞；upgrader 走 SPIDER 类型，但代码里的 canupgradefn（要求 spiderwhisperer）无人读取，实际门禁是 doer 的 spider_upgradeuser 标签——只有韦伯能花 5 丝把蜘蛛巢催升一级。"
aliases: ["silk", "蜘蛛丝", "Silk"]
tags: [prefab, item]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/silk.lua", "components/upgrader.lua", "components/upgradeable.lua", "components/tradable.lua", "components/lootdropper.lua", "components/snowmandecoratable.lua", "actions.lua", "componentactions.lua", "constants.lua", "standardcomponents.lua", "prefabs/webber.lua", "prefabs/spiderden.lua", "prefabs/spider.lua", "prefabs/spiderqueen.lua", "prefabs/spiderhole.lua", "prefabs/moonspiderden.lua", "prefabs/mushtree_webbed.lua", "prefabs/oceanvine_cocoon.lua", "prefabs/smashables.lua", "prefabs/tumbleweed.lua", "prefabs/catcoon.lua", "prefabs/tree_rock_data.lua", "prefabs/pigking.lua", "recipes.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 蜘蛛丝（silk）

## 快速判断

- 职责与适用：`prefabs/silk.lua` 单注册 1 件 item，官方简中"蜘蛛丝"（[chinese_s.po L349385-L349387](../../../raw/current/scripts/languages/chinese_s.po#L349385-L349387)）。堆叠 40、`cattoy` 标签、默认浮水、`upgrader`（SPIDER）与 `tradable` 在 master 段；资源 `anim/silk.zip`，bank／build 同为 `silk`（[L1-L4](../../../raw/current/scripts/prefabs/silk.lua#L1-L4)、[L19-L21](../../../raw/current/scripts/prefabs/silk.lua#L19-L21)）。
- 优先使用：要用丝升级蜘蛛巢先读"升级门禁与消耗"——真正的门槛是韦伯的 `spider_upgradeuser` 标签，不是 prefab 里的 `spiderwhisperer` 判断；丝的其他用途按 recipes.lua 行号取。
- 重要副作用：升级一次只吃 1 根丝、每 5 次推进一阶，最多两阶（stage 1→2→3）；作祟只弹飞（MakeHauntableLaunch 无点燃／砸碎段）；丝是 `cattoy`，可以喂浣猫换好感。
- 修改边界：本页不覆盖蜘蛛巢本体（自然生长、子蛛生成、拆卸掉落）与浣猫礼物机制的完整流程，只在必要处写到门禁。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `silk` | `fn()` 单工厂：网络段建实体 + `MakeInventoryPhysics` + bank/build/`idle` + `pickupsound = "cloth"` → 加 `cattoy` 标签 → `MakeInventoryFloatable` → `SetPristine` → 非 master 返回 → master 段挂 `stackable`／`inspectable`／`inventoryitem`／`tradable`／`upgrader` → `MakeHauntableLaunch` → `Prefab("silk", fn, assets)` | 无变体、无皮肤；`upgrader.canupgradefn = CanUpgrade` 与 `upgradetype = UPGRADETYPES.SPIDER`（[L6-L8](../../../raw/current/scripts/prefabs/silk.lua#L6-L8)、[L35-L47](../../../raw/current/scripts/prefabs/silk.lua#L35-L47)、[L52](../../../raw/current/scripts/prefabs/silk.lua#L52)） |

master 段实际挂载：

- `stackable`：`TUNING.STACK_SIZE_SMALLITEM` = 40（[L35-L36](../../../raw/current/scripts/prefabs/silk.lua#L35-L36)、[tuning.lua L83](../../../raw/current/scripts/tuning.lua#L83)）。
- `inspectable`、`inventoryitem`（无 `SetSinks`，配合 `MakeInventoryFloatable` 浮水）（[L27](../../../raw/current/scripts/prefabs/silk.lua#L27)、[L38-L39](../../../raw/current/scripts/prefabs/silk.lua#L38-L39)）。
- `tradable`：不设 `goldvalue`，保持 0（[L41](../../../raw/current/scripts/prefabs/silk.lua#L41)、[tradable.lua L1-L7](../../../raw/current/scripts/components/tradable.lua#L1-L7)），猪王只收 `> 0`（[pigking.lua L604-L605](../../../raw/current/scripts/prefabs/pigking.lua#L604-L605)）。
- `upgrader`：`canupgradefn = CanUpgrade`（返回 `doer:HasTag("spiderwhisperer")`）、`upgradetype = UPGRADETYPES.SPIDER`（[L6-L8](../../../raw/current/scripts/prefabs/silk.lua#L6-L8)、[L43-L45](../../../raw/current/scripts/prefabs/silk.lua#L43-L45)、[constants.lua L1969-L1972](../../../raw/current/scripts/constants.lua#L1969-L1972)）。
- `MakeHauntableLaunch`：作祟弹飞（[L47](../../../raw/current/scripts/prefabs/silk.lua#L47)、[standardcomponents.lua L1053-L1069](../../../raw/current/scripts/standardcomponents.lua#L1053-L1069)）。
- 未挂：`edible`、`fuel`、`perishable`、`snowmandecor`、`bait`（全文 52 行内无这些组件；雪人装饰表也没有 silk，[snowmandecoratable.lua L5-L42](../../../raw/current/scripts/components/snowmandecoratable.lua#L5-L42)）。

## 升级门禁与消耗

- **死代码先排除**：`upgrader.canupgradefn` 这个字段在 `components/upgrader.lua` 里没有读取方，`Upgrader:CanUpgrade` 只比较 `upgradetype` 与 `doer:HasTag(upgradetype.."_upgradeuser")`（[upgrader.lua L21-L29](../../../raw/current/scripts/components/upgrader.lua#L21-L29)）。全 scripts 检索 `canupgradefn`，读取点只有 `upgradeable`（目标侧）与 `upgrademoduleowner`，都不是物品侧 `upgrader`。因此 prefab 里那句 `spiderwhisperer` 判断在本快照**不生效**。
- **实际门禁**：客户端只有在"手持物带 `spider_upgrader`、目标带 `spider_upgradeable`、doer 带 `spider_upgradeuser`"三者同时成立时才给出 UPGRADE 动作（[componentactions.lua L1889-L1898](../../../raw/current/scripts/componentactions.lua#L1889-L1898)）；服务端再复核一次（[actions.lua L4056-L4069](../../../raw/current/scripts/actions.lua#L4056-L4069)）。`spider_upgradeuser` 在全 scripts 只有韦伯身上添加（[webber.lua L42](../../../raw/current/scripts/prefabs/webber.lua#L42)），故**只有韦伯**能升级蜘蛛巢；`spiderwhisperer` 标签本身不够。
- **消耗与阶数**：`upgradeable` 默认 `numstages = 3`、`upgradesperstage = 5`、`upgrader.upgradevalue = 1`（[upgradeable.lua L17-L33](../../../raw/current/scripts/components/upgradeable.lua#L17-L33)、[upgrader.lua L10-L19](../../../raw/current/scripts/components/upgrader.lua#L10-L19)），蜘蛛巢不改这两个默认值（[spiderden.lua L785-L789](../../../raw/current/scripts/prefabs/spiderden.lua#L785-L789)）。每次 UPGRADE 从堆叠扣 1 根丝（[upgradeable.lua L75-L83](../../../raw/current/scripts/components/upgradeable.lua#L75-L82)），累计 5 次推进一阶并清零计数（[upgradeable.lua L84-L93](../../../raw/current/scripts/components/upgradeable.lua#L84-L93)）；stage 3 时 `stage < numstages` 不成立，目标不再带 `spider_upgradeable` 标签。
- **推进动作**：`onstageadvancefn` 调 `growable:DoGrowth()`，巢随之长大一档（血量、子蛛上限、地皮半径一并按 `SetStage` 重设）（[spiderden.lua L548-L551](../../../raw/current/scripts/prefabs/spiderden.lua#L548-L551)、[L142-L160](../../../raw/current/scripts/prefabs/spiderden.lua#L142-L160)）。
- **拒绝条件**：目标被 `bedazzled` 且不在脱落中时 `CanUpgrade` 返回 `false, "BEDAZZLED"`；目标处在平台（船／平台）上时也返回 false（[spiderden.lua L560-L566](../../../raw/current/scripts/prefabs/spiderden.lua#L560-L566)）。这两条是目标侧 `upgradeable.canupgradefn`，与物品侧那个死字段无关。
- **计数不随自然生长清零**：`numupgrades` 只在 `AdvanceStage` 与蜘蛛巢被剃（`OnShaved`）时归零（[upgradeable.lua L51-L58](../../../raw/current/scripts/components/upgradeable.lua#L51-L58)、[spiderden.lua L650-L652](../../../raw/current/scripts/prefabs/spiderden.lua#L650-L652)）；巢自己按计时长大时只改 `upgradeable` 的 stage，不改计数。

## 获得路径

- **蜘蛛（8 个注册共用一套掉落池）**：`AddRandomLoot` 权重 `monstermeat 1 / silk 0.5 / spidergland 0.5`、`numrandomloot = 1`（[spider.lua L645-L650](../../../raw/current/scripts/prefabs/spider.lua#L645-L650)），即每只蜘蛛 25% 掉 1 根丝（0.5 / 2.0）；`spider`／`spider_warrior`／`spider_hider`／`spider_spitter`／`spider_dropper`／`spider_moon`／`spider_healer`／`spider_water` 都由同一工厂注册（[spider.lua L1087-L1094](../../../raw/current/scripts/prefabs/spider.lua#L1087-L1094)）。
- **蜘蛛女王（`spiderqueen`）**：固定掉落表 4 丝 + 4 怪肉 + 卵囊 + 蜘蛛帽，每项必掉（[spiderqueen.lua L22-L34](../../../raw/current/scripts/prefabs/spiderqueen.lua#L22-L34)）。
- **蜘蛛巢（`spiderden` 三档）**：被摧毁时按档位掉 2／4／6 根丝（大巢另加卵囊）（[spiderden.lua L61-L66](../../../raw/current/scripts/prefabs/spiderden.lua#L61-L66)、[L327-L338](../../../raw/current/scripts/prefabs/spiderden.lua#L327-L338)）；韦伯剃巢另得 1 根（[L812-L814](../../../raw/current/scripts/prefabs/spiderden.lua#L812-L814)）。
- **洞穴蜘蛛岩（`spiderhole`）**：1 根必得 + 50%（[spiderhole.lua L27-L36](../../../raw/current/scripts/prefabs/spiderhole.lua#L27-L36)）。
- **月岩蜘蛛巢（`moonspiderden`）**：1 根必得 + 50%（[moonspiderden.lua L25-L34](../../../raw/current/scripts/prefabs/moonspiderden.lua#L25-L34)）。
- **蛛网蘑菇树（`mushtree_webbed`）**：1 根必得 + 0.3 + 0.3（[mushtree_webbed.lua L18-L24](../../../raw/current/scripts/prefabs/mushtree_webbed.lua#L18-L24)）。
- **海洋藤茧（`oceanvine_cocoon`）**：1 根必得 + 0.3 + 0.3，另必掉树枝（[oceanvine_cocoon.lua L23-L29](../../../raw/current/scripts/prefabs/oceanvine_cocoon.lua#L23-L29)）。
- **档案馆花瓶类（`smashables`）**：5% 触发一次随机掉落，随机表里 silk 权重 0.1、其余 6 项合计 0.45（约 18% 命中该次）（[smashables.lua L267-L281](../../../raw/current/scripts/prefabs/smashables.lua#L267-L281)）。
- **风滚草（`tumbleweed`）**：按权重抽 3 件，silk 权重 1、基础总权重约 100.06（约 1%/件）（[tumbleweed.lua L147-L184](../../../raw/current/scripts/prefabs/tumbleweed.lua#L147-L184)、[L213-L244](../../../raw/current/scripts/prefabs/tumbleweed.lua#L213-L244)）。
- **浣猫（`catcoon`）**：好友档礼物表第 7 档含 1 根丝，随机 1 件（[catcoon.lua L181-L188](../../../raw/current/scripts/prefabs/catcoon.lua#L182-L188)、[L265-L271](../../../raw/current/scripts/prefabs/catcoon.lua#L265-L271)）。
- **石化树（`tree_rocks` 族）**：`WEB_CREEP` 区域给加权表追加 `总权重 × 0.25` 的丝权重（[tree_rock_data.lua L266-L278](../../../raw/current/scripts/prefabs/tree_rock_data.lua#L266-L278)）；沼泽／泥沼区基础权重表也含 silk 5（[L35-L40](../../../raw/current/scripts/prefabs/tree_rock_data.lua#L35-L40)、[L221-L227](../../../raw/current/scripts/prefabs/tree_rock_data.lua#L221-L227)）。
- 掉落表内每行独立掷一次、`chance >= 1` 不掷（[lootdropper.lua L213-L224](../../../raw/current/scripts/components/lootdropper.lua#L213-L224)）；上列概率是"每杀／每采一次"的口径，不含幸运加成。

## 主要消费

- **升级蜘蛛巢**：5 根丝一阶、最多两阶（详见上节）。
- **配方（主要入口 31／55 条，量按 [recipes.lua](../../../raw/current/scripts/recipes.lua)）**：捕虫网 2（[L399](../../../raw/current/scripts/recipes.lua#L399)）、捕鸟陷阱 4（[L452](../../../raw/current/scripts/recipes.lua#L452)）、钓竿／海钓竿 2／3（[L641-L642](../../../raw/current/scripts/recipes.lua#L641-L642)）、回旋镖 1（[L489](../../../raw/current/scripts/recipes.lua#L489)）、雨伞 2（[L525](../../../raw/current/scripts/recipes.lua#L525)，见 [[reference/prefabs/umbrella]]）、缝纫包 8（[L528](../../../raw/current/scripts/recipes.lua#L528)）、高礼帽 6（[L531](../../../raw/current/scripts/recipes.lua#L531)）、冬帽 4（[L535](../../../raw/current/scripts/recipes.lua#L535)）、猫帽 4（[L536](../../../raw/current/scripts/recipes.lua#L536)）、养蜂帽 8（[L543](../../../raw/current/scripts/recipes.lua#L543)）、帐篷 6／凉棚 2（[L578-L579](../../../raw/current/scripts/recipes.lua#L578-L579)）、猪皮包 6（[L517](../../../raw/current/scripts/recipes.lua#L517)）、蜘蛛卵囊 12、巢装饰 1、蜘蛛哨 3（[L324-L326](../../../raw/current/scripts/recipes.lua#L324-L326)）、伯尼 2（[L94](../../../raw/current/scripts/recipes.lua#L94)，见 [[reference/prefabs/character-gear-extra]]）、药剂容器 4（[L162](../../../raw/current/scripts/recipes.lua#L162)）、沃尔特帽 4（[L212](../../../raw/current/scripts/recipes.lua#L212)）、蛛网书 8（[L284](../../../raw/current/scripts/recipes.lua#L284)）、蜘蛛变异器 1–3（[L329-L335](../../../raw/current/scripts/recipes.lua#L329-L335)）、桅杆 8（[L617](../../../raw/current/scripts/recipes.lua#L617)）、拖网 6（[L631](../../../raw/current/scripts/recipes.lua#L631)）、缝纫人台 2（[L590](../../../raw/current/scripts/recipes.lua#L590)）。
- **配方（其余 24 条，注册名）**：`slingshot_handle_silk` 2（[L242](../../../raw/current/scripts/recipes.lua#L242)）、`slingshotammo_container` 6（[L250](../../../raw/current/scripts/recipes.lua#L250)）、`magician_chest` 1（[L295](../../../raw/current/scripts/recipes.lua#L295)）、`sewing_tape` 1（[L303](../../../raw/current/scripts/recipes.lua#L303)）、`spider_healer_item` 2（[L328](../../../raw/current/scripts/recipes.lua#L328)）、`wortox_nabbag` 2（[L357](../../../raw/current/scripts/recipes.lua#L357)）、`malbatross_feathered_weave` 1（[L471](../../../raw/current/scripts/recipes.lua#L471)）、`sweatervest` 6（[L547](../../../raw/current/scripts/recipes.lua#L547)）、`trunkvest_summer` 8／`trunkvest_winter` 8（[L548-L549](../../../raw/current/scripts/recipes.lua#L548-L549)）、`hawaiianshirt` 3（[L551](../../../raw/current/scripts/recipes.lua#L551)）、`chum` 1（[L665](../../../raw/current/scripts/recipes.lua#L665)）、`saddle_race` 4（[L684](../../../raw/current/scripts/recipes.lua#L684)）、`wx78_scanner_item` 1（[L750](../../../raw/current/scripts/recipes.lua#L750)）、`hermitcrab_teashop` 8（[L972](../../../raw/current/scripts/recipes.lua#L972)）、`handpillow_*` 4 条各 2（[L1135-L1138](../../../raw/current/scripts/recipes.lua#L1135-L1138)）、`bodypillow_*` 4 条各 2（[L1140-L1143](../../../raw/current/scripts/recipes.lua#L1140-L1143)）、`yotb_sewingmachine_item` 2（[L1156](../../../raw/current/scripts/recipes.lua#L1156)）。
- **喂浣猫**：丝带 `cattoy` 标签，可作为礼物投喂并被接受（[catcoon.lua L273-L279](../../../raw/current/scripts/prefabs/catcoon.lua#L273-L279)），之后走好友回礼链（见获得路径）。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：upgrader | 提供 `spider_upgrader` 标签与类型比较 | 服务端＋客户端动作枚举；`canupgradefn` 无人读 | [[reference/components/upgrader]]；[upgrader.lua L21-L29](../../../raw/current/scripts/components/upgrader.lua#L21-L29) |
| 组件：upgradeable | 目标侧阶数与计数、扣件、推进 | 服务端；蜘蛛巢默认 3 阶 / 5 次 | [[reference/components/upgradeable]]；[spiderden.lua L785-L789](../../../raw/current/scripts/prefabs/spiderden.lua#L785-L789) |
| 组件：tradable | 无 `goldvalue`，猪王不收 | 服务端 | [[reference/components/tradable]]；[pigking.lua L604-L605](../../../raw/current/scripts/prefabs/pigking.lua#L604-L605) |
| 组件：stackable／inventoryitem | 40 一叠、默认浮水 | 两端；`MakeInventoryFloatable` 在 `SetPristine` 前 | [[reference/components/inventoryitem]]；[silk.lua L27-L36](../../../raw/current/scripts/prefabs/silk.lua#L27-L36) |
| API／数据：`cattoy` 标签 | 浣猫收礼判定 | 服务端；喂食加好感 | [catcoon.lua L273-L279](../../../raw/current/scripts/prefabs/catcoon.lua#L273-L279) |
| API／数据：`spider_upgradeuser` 标签 | 真正决定谁能升级巢 | 仅韦伯添加 | [webber.lua L42](../../../raw/current/scripts/prefabs/webber.lua#L42) |
| 关联 Prefab：`spiderden` | 升级目标；被剃／被拆也产丝 | 服务端；巢在平台上不可升级 | [spiderden.lua L560-L566](../../../raw/current/scripts/prefabs/spiderden.lua#L560-L566) |
| 关联 Prefab：`spider`／`spiderqueen`／`spiderhole` | 主要获得源 | 击杀／摧毁掉落 | [spider.lua L645-L650](../../../raw/current/scripts/prefabs/spider.lua#L645-L650) |
| 资源 | bank／build 均为 `silk`，包 `anim/silk.zip` | 两端；无皮肤分支 | [silk.lua L1-L4](../../../raw/current/scripts/prefabs/silk.lua#L1-L4) |

## 常见陷阱与限制

- **误区：`canupgradefn = CanUpgrade` 就是蜘蛛丝的门禁**：物品侧 `upgrader.canupgradefn` 在全 scripts 没有读取方，实际判定是 `upgradetype.."_upgradeuser"` 标签；把"有 spiderwhisperer 标签就能升级"写进别处会与实现不符（[silk.lua L6-L8](../../../raw/current/scripts/prefabs/silk.lua#L6-L8)、[upgrader.lua L21-L29](../../../raw/current/scripts/components/upgrader.lua#L21-L29)）。
- **边界：只有韦伯**：`spider_upgradeuser` 仅由 `webber.lua` 添加；韦伯以外的角色手持蜘蛛丝对巢不会出现 UPGRADE（[webber.lua L42](../../../raw/current/scripts/prefabs/webber.lua#L42)、[componentactions.lua L1889-L1898](../../../raw/current/scripts/componentactions.lua#L1889-L1898)）。
- **边界：5 根一阶、封顶 3 阶**：不是"一根升一级"；stage 3 的巢没有 `spider_upgradeable` 标签，继续给丝无效（[upgradeable.lua L60-L73](../../../raw/current/scripts/components/upgradeable.lua#L60-L73)、[L88-L90](../../../raw/current/scripts/components/upgradeable.lua#L88-L90)）。
- **边界：被装饰的巢不能升级**：`bedazzled` 且未在脱落时直接返回 `BEDAZZLED`；巢在平台上也拒绝（[spiderden.lua L560-L566](../../../raw/current/scripts/prefabs/spiderden.lua#L560-L566)）。
- **误区：丝能当燃料／食物**：本 prefab 不挂 `fuel` 与 `edible`；作为 `cattoy` 只是"浣猫接受的玩具"，不等于可食用（[silk.lua L35-L47](../../../raw/current/scripts/prefabs/silk.lua#L35-L47)、[catcoon.lua L273-L279](../../../raw/current/scripts/prefabs/catcoon.lua#L273-L279)）。

## 验证记录

- 源码审阅：2026-11-10，steam-buildid:26474873。`prefabs/silk.lua` 全文（52 行）；`upgrader`／`upgradeable` 全组件（含 `canupgradefn` 的全局检索结论与默认阶数）；UPGRADE 动作服务端复核与客户端动作枚举；`spiderden` 的 `upgradeable` 挂载、`CanUpgrade` 拒绝条件、`OnStageAdvance`、剃巢与掉落；`spider`／`spiderqueen`／`spiderhole`／`moonspiderden`／`mushtree_webbed`／`oceanvine_cocoon`／`smashables`／`tumbleweed`／`catcoon`／`tree_rock_data` 掉落表按行核验；recipes 55 条（全表 `Recipe2` 含 silk；主要入口 31 条 + 其余 24 条按注册名列出）；`chinese_s.po` 名称键。未核验：`upgradesperstage` 在模组改写下的行为、浣猫好友档位的升级条件（只核到 `PickRandomGift` 的取值）、`tree_rock` 加权抽取的完整数值链（只核到权重行）、`smashables` 花瓶 5% 之外其他分支。
- R2 修订（2026-11-10，主代理复审 K-2）：配方改为「主要入口 31／55 条」并把其余 24 条按注册名与行号补齐。
- 外部检视修订（2026-11-10，R14）：实体称谓统一为 `catcoon` 的简中实显名「浣猫」（[chinese_s.po L340690-L340692](../../../raw/current/scripts/languages/chinese_s.po#L340690-L340692)），保留注册名检索；`catcoonhat` 按自己的名称键改「猫帽」（[L340700-L340702](../../../raw/current/scripts/languages/chinese_s.po#L340700-L340702)）。
- 游戏运行：未运行。
- 结构检查：validate_page 通过（0 issue，standard 档内）；validate_source_links 通过（112 处引用、0 error／0 warning）；未加 --check-indexes。

返回 [[reference/prefabs/crafting-materials-index|通用材料与加工中间件（暂行索引）]]、[[reference/prefabs/index|原版 Prefab 索引]]。
