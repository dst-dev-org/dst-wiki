---
title: "蚊子血囊（mosquitosack）"
created: "2026-11-10"
updated: "2026-11-10"
type: prefab
summary: "蚊子 50% 掉落的多用途材料：默认堆叠 20、可直接当 8 点治疗品；沃特蚊子技能线四条配方（小痒痒／蚊子炸弹／嗡嗡肥料／蚊血注射液）与弹弓、水气球、救生垫等共 10 条配方的材料；能装水变水气球，也是刷新小痒痒鲜度的唯一修理材料。无新鲜度。"
aliases: ["mosquitosack", "蚊子血囊", "Mosquito Sack", "蚊子血袋"]
tags: [prefab, item]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/mosquitosack.lua", "prefabs/mosquito.lua", "stategraphs/SGmosquito.lua", "prefabs/balloons_empty.lua", "components/repairer.lua", "components/repairable.lua", "components/fillable.lua", "components/stackable.lua", "components/health.lua", "components/lootdropper.lua", "actions.lua", "componentactions.lua", "recipes.lua", "constants.lua", "prefabs/pond.lua", "prefabs/mosquitomusk.lua", "tuning.lua", "standardcomponents.lua", "entityscript.lua", "componentutil.lua", "prefabs/corpses.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 蚊子血囊（mosquitosack）

## 快速判断

- 职责与适用：可拾取材料物品（inventoryitem），四个独立用途——`healer` 8 点治疗、`repairer`（vitae）刷新小痒痒鲜度、`fillable` 装水变水气球、10 条配方的通用材料。[组件段 L33-L46](../../../raw/current/scripts/prefabs/mosquitosack.lua#L33-L46)
- 优先使用：查"蚊子血囊／血袋／mosquitosack"到本页；刷新鲜度的修理结算细节（候选条件、满鲜度拒绝）归 [[reference/prefabs/mosquitomusk]] 的「修理」节，本页只写血囊这一侧。
- 重要副作用：可燃（3 秒）、可传火、可浮水；作祟只弹飞；无新鲜度；默认堆叠 20。
- 修改边界：蚊子本体（宿主）与沃特技能树页 Wiki 待建；水气球与小痒痒各有归属页，本页不展开它们的内部机制。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `mosquitosack` | `fn` 单工厂：网络段（`MakeInventoryPhysics`；bank/build 均 `bladder`、播 `idle`；`MakeInventoryFloatable("small", 0.05)`）→ `SetPristine` → 客户端返回 → master 段（`MakeSmallBurnable(TINY_BURNTIME)`＋`MakeSmallPropagator`＋`MakeHauntableLaunchAndIgnite`；`inspectable`／`inventoryitem`／`stackable`／`repairer`／`fillable`／`healer`） | 无变体（快照内没有皮肤条目）。网络段无任何游戏标签；`fillable`／`repairer`／`freshen_vitae` 标签由组件构造器在 master 段添加（实体标签自动同步到客户端，候选判定可用）。[构造全文](../../../raw/current/scripts/prefabs/mosquitosack.lua#L6-L49) |

- 名称键 `STRINGS.NAMES.MOSQUITOSACK`＝蚊子血囊（英文 Mosquito Sack）。[po L345945-L345947](../../../raw/current/scripts/languages/chinese_s.po#L345945-L345947)
- 组件明细：`stackable` 默认上限 `STACK_SIZE_MEDITEM`＝20（[stackable.lua L28](../../../raw/current/scripts/components/stackable.lua#L28)、[tuning L82](../../../raw/current/scripts/tuning.lua#L82)）；`repairer` 材料 `MATERIALS.VITAE`（`"vitae"`）＋`perishrepairpercent = 1`（[L38-L40](../../../raw/current/scripts/prefabs/mosquitosack.lua#L38-L40)、[constants L1959](../../../raw/current/scripts/constants.lua#L1959)）；`fillable.filledprefab = "waterballoon"`（[L42-L43](../../../raw/current/scripts/prefabs/mosquitosack.lua#L42-L43)）；`healer` 8 点 `HEALING_MEDSMALL`（[L45-L46](../../../raw/current/scripts/prefabs/mosquitosack.lua#L45-L46)）。

## 获得

| 来源 | 条件与概率 | 证据 |
|---|---|---|
| 击杀蚊子 | 共享掉落表 `{"mosquitosack", .5}`：死亡瞬间掷 50% 出 1 个；若生成蚊子尸骸，血囊挂在尸骸上、尸骸被销毁时掉出（未生成尸骸则立即掉落） | [掉落表 L25-L28](../../../raw/current/scripts/prefabs/mosquito.lua#L25-L28)、[SetChanceLootTable L211-L212](../../../raw/current/scripts/prefabs/mosquito.lua#L211-L212)、[死亡掉落 SGmosquito L93](../../../raw/current/scripts/stategraphs/SGmosquito.lua#L88-L110)、[存取链 entityscript L2181-L2192](../../../raw/current/scripts/entityscript.lua#L2181-L2192)、[componentutil L1579](../../../raw/current/scripts/componentutil.lua#L1579)、[尸骸掉落 corpses.lua L489](../../../raw/current/scripts/prefabs/corpses.lua#L489) |
| 谋杀库存蚊子 | 谋杀按堆叠数逐个 `GenerateLoot`，每只 50%；蚊子 `canmurder` 默认为真 | [MURDER 结算](../../../raw/current/scripts/actions.lua#L3166-L3200)、[health.lua L96](../../../raw/current/scripts/components/health.lua#L96) |
| 气球堆消失 | `balloons_empty`（气球遗留物）在地面放置满 `BALLOON_PILE_DECAY_TIME`＝3 天后自行消失，生成 2 个血囊；放进背包会暂停计时 | [dodecay L14-L21](../../../raw/current/scripts/prefabs/balloons_empty.lua#L14-L21)、[tuning L5928](../../../raw/current/scripts/tuning.lua#L5928)、[暂停计时 L111-L114](../../../raw/current/scripts/prefabs/balloons_empty.lua#L111-L114) |

- 作祟异常路径：活蚊子被幽灵作祟时按概率直接进入 `splat` 爆浆状态（[mosquito.lua L250-L257](../../../raw/current/scripts/prefabs/mosquito.lua#L250-L257)），该状态只造成范围伤害后 `Remove`，**不走死亡掉落**（[SGmosquito splat L58-L77](../../../raw/current/scripts/stategraphs/SGmosquito.lua#L58-L77)）——作祟爆浆拿不到血囊。
- 燃烧死亡：蚊子烧死时不生成尸骸，走立即 `DropLoot`；源在燃烧中时掉落按燃烧换算处理——血囊无熟制变体，受控燃烧下带火掉落、普通烧死换算为灰烬（[DropLoot 换算 L415-L442](../../../raw/current/scripts/components/lootdropper.lua#L415-L442)）。

## 用途

### 治疗：标准 healer 接入

`healer` 挂 `HEALING_MEDSMALL`＝8 点（[L45-L46](../../../raw/current/scripts/prefabs/mosquitosack.lua#L45-L46)），无 canhealfn／onhealfn，是标准治疗品接入；通用结算（HEAL 动作候选、efficientuser 倍率、满血仍消耗）见 [[reference/components/healer]]。

### 装水：fillable → 水气球

- 持血囊对带 `watersource` 标签的目标（水塘、绿洲湖、温泉、冰岩等）出现 FILL 动作（[componentactions L1257-L1260](../../../raw/current/scripts/componentactions.lua#L1257-L1260)；水塘打标签示例 [pond.lua L235](../../../raw/current/scripts/prefabs/pond.lua#L235)）；对场景目标右键的候选入口同条件（[L2326-L2328](../../../raw/current/scripts/componentactions.lua#L2326-L2328)）。
- 结算 `fillable:Fill(source)`：在原位置生成 `filledprefab`＝`waterballoon` 并消耗 1 个血囊（堆叠拆一份；在容器中时容器内换格）（[fillable.lua L28-L62](../../../raw/current/scripts/components/fillable.lua#L28-L62)、[FILL 结算](../../../raw/current/scripts/actions.lua#L2126-L2163)）。
- 血囊的 `acceptsoceanwater` 为默认 false、也没有 `fillable_showoceanaction` 标签，**不能对海面装水**（[fillable 默认 L9-L14](../../../raw/current/scripts/components/fillable.lua#L9-L14)、[海洋装水候选](../../../raw/current/scripts/componentactions.lua#L2110-L2112)）。这是不耗冰的单个水气球来源；批量配方见下表 `waterballoon` 行。

### 修理：刷新小痒痒鲜度

- 血囊是全库唯一 `repairable` 材料 `"vitae"` 的修理物——挂该材料的可修对象只有小痒痒（[mosquitomusk L35-L38](../../../raw/current/scripts/prefabs/mosquitomusk.lua#L35-L38)，全库检索 `MATERIALS.VITAE` 仅此一处消费者）。
- 血囊 `repairer` 四通道里只有 `perishrepairpercent = 1` 非零（其余为 0），因此只加 `freshen_vitae` 标签、只能走鲜度分支：给小痒痒 `SetPercent(现有+1)`（即回满）并消耗 1 个血囊（[repairer.lua L1-L24](../../../raw/current/scripts/components/repairer.lua#L1-L24)、[鲜度分支](../../../raw/current/scripts/components/repairable.lua#L126-L133)、[材料校验与消耗](../../../raw/current/scripts/components/repairable.lua#L148-L183)）。动作候选、满鲜度拒绝、`noannounce` 等结算细节见 [[reference/prefabs/mosquitomusk]]「修理」节。

### 配方消费

| 配方 | 材料 | 门禁 | 证据 |
|---|---|---|---|
| `mosquitomusk`（小痒痒） | 血囊 1＋牛毛 1 | `builder_skill="wurt_mosquito_craft_1"` | [recipes L113](../../../raw/current/scripts/recipes.lua#L113) |
| `mosquitobomb`（蚊子炸弹） | 血囊 1＋蚊子 4 | `builder_skill="wurt_mosquito_craft_2"` | [recipes L114](../../../raw/current/scripts/recipes.lua#L114) |
| `mosquitofertilizer`（嗡嗡肥料） | 血囊 1＋硝石 2＋粪便 1 | `builder_skill="wurt_mosquito_craft_2"` | [recipes L115](../../../raw/current/scripts/recipes.lua#L115) |
| `mosquitomermsalve`（蚊血注射液） | 血囊 2 | `builder_skill="wurt_mosquito_craft_2"` | [recipes L116](../../../raw/current/scripts/recipes.lua#L116) |
| `slingshot`（弹弓） | 树枝 1＋血囊 2 | `builder_tag="pebblemaker"`（沃尔特） | [recipes L211](../../../raw/current/scripts/recipes.lua#L211) |
| `waterballoon`（水气球） | 血囊 2＋冰 1，出 4 个 | `TECH.SCIENCE_ONE` | [recipes L512](../../../raw/current/scripts/recipes.lua#L512) |
| `flotationcushion`（救生垫） | 血囊 1＋绳索 1 | `TECH.SEAFARING_ONE` | [recipes L634](../../../raw/current/scripts/recipes.lua#L634) |
| `oceanfishinglure_hermit_rain` | 舟壳 1＋血囊 1 | `TECH.LOST`（隐居者处学得） | [recipes L661](../../../raw/current/scripts/recipes.lua#L661) |
| `beeswax_spray` | 血囊 2＋蜂蜡 3＋噩梦燃料 5 | `TECH.SHADOWFORGING_TWO`＋影刀锻造台 | [recipes L872](../../../raw/current/scripts/recipes.lua#L872) |
| `wintercooking_berrysauce` | 血囊 2＋冬宴燃料 1 | `TECH.WINTERSFEASTCOOKING_ONE`（冬宴炉） | [recipes L1430](../../../raw/current/scripts/recipes.lua#L1430) |

## 燃烧与作祟（无新鲜度）

- **无新鲜度**：`mosquitosack.lua` 全文无 `perishable` 挂载（逐文件复核），与它修理的小痒痒（8 天腐坏）形成对照。
- 燃烧：`MakeSmallBurnable(TINY_BURNTIME)`＝3 秒，烧毁生成灰烬；`MakeSmallPropagator` 传火（[L27-L28](../../../raw/current/scripts/prefabs/mosquitosack.lua#L27-L28)、[helper L12-L44](../../../raw/current/scripts/standardcomponents.lua#L12-L44)）。
- 作祟：`MakeHauntableLaunchAndIgnite` 的点燃分支整段被 `#HAUNTFIX` 注释，实际只弹飞（[L29](../../../raw/current/scripts/prefabs/mosquitosack.lua#L29)、[helper L1191-L1217](../../../raw/current/scripts/standardcomponents.lua#L1191-L1217)）。

## 常见陷阱与限制

- **边界：作祟蚊子不掉血囊**：作祟直接 `splat` 的路径绕过 `DropDeathLoot`；要血囊请正常击杀（含打爆贪血蚊子——爆浆前的死亡状态已调用掉落）。[mosquito.lua L250-L257](../../../raw/current/scripts/prefabs/mosquito.lua#L250-L257)、[SGmosquito L58-L110](../../../raw/current/scripts/stategraphs/SGmosquito.lua#L58-L110)
- **边界：血囊掉落可能延迟随尸骸出现**：死亡瞬间掷定 50%，生成蚊子尸骸时挂在尸骸上、尸骸销毁时才落地；不要把"打死没掉"当成概率失败。[entityscript L2181-L2192](../../../raw/current/scripts/entityscript.lua#L2181-L2192)、[corpses.lua L489](../../../raw/current/scripts/prefabs/corpses.lua#L489)
- **边界：修理物只能刷小痒痒**：`"vitae"` 材料没有其他 `repairable` 消费者；四通道仅鲜度非零，不能修血量／耐久／工作量。[repairable.lua L106-L146](../../../raw/current/scripts/components/repairable.lua#L106-L146)
- **误区：能对海装水**：`acceptsoceanwater` 默认 false 且无 `fillable_showoceanaction`，海面无 FILL 候选。[componentactions L2110-L2112](../../../raw/current/scripts/componentactions.lua#L2110-L2112)
- **边界：不是食物也不是燃料物品**：无 `edible`／`fuel` 组件；虽可点燃但自身不能喂火堆。[构造 L33-L46](../../../raw/current/scripts/prefabs/mosquitosack.lua#L33-L46)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`healer` | 标准 8 点治疗 | 服务端 | [[reference/components/healer]]；[L45-L46](../../../raw/current/scripts/prefabs/mosquitosack.lua#L45-L46) |
| 组件：`repairer` | 材料 vitae、`perishrepairpercent = 1`，加 `freshen_vitae` 标签 | 服务端 | [[reference/components/repairer]]；[L38-L40](../../../raw/current/scripts/prefabs/mosquitosack.lua#L38-L40) |
| 组件：`fillable` | `filledprefab = "waterballoon"`；不带海洋装水 | 服务端结算；候选两端 | [[reference/components/fillable]]；[L42-L43](../../../raw/current/scripts/prefabs/mosquitosack.lua#L42-L43) |
| 组件：`stackable` | 默认上限 20 | 服务端 | [[reference/components/stackable]] |
| 关联 Prefab：`mosquito` | 血囊的主要来源（掉落／谋杀）；血囊不驱蚊——驱蚊靠小痒痒标签 | 蚊子本体页 Wiki 待建 | [掉落表 L25-L28](../../../raw/current/scripts/prefabs/mosquito.lua#L25-L28) |
| 关联 Prefab：`mosquitomusk` | 修理目标（vitae 消费者） | 本体页已有 | [[reference/prefabs/mosquitomusk]] |
| 关联 Prefab：`waterballoon` | 装水产物（1:1）与配方产物（2+冰→4） | 投掷结算见 [[reference/prefabs/ranged-weapons]]，本体页未单列 | [fillable.lua L41-L43](../../../raw/current/scripts/components/fillable.lua#L41-L43) |
| 关联 Prefab：`balloons_empty` | 放置 3 天消失换 2 血囊 | 服务端计时，仅地面 | [L15-L21](../../../raw/current/scripts/prefabs/balloons_empty.lua#L15-L21) |
| 资源：`anim/bladder.zip` | bank/build 均 `bladder`，`idle` | assets 声明 | [L1-L4](../../../raw/current/scripts/prefabs/mosquitosack.lua#L1-L4)、[L15-L17](../../../raw/current/scripts/prefabs/mosquitosack.lua#L15-L17) |

## 验证记录

- 源码审阅：2026-11-10，steam-buildid:26474873（与 `raw/current/BUILD.txt` 一致）。覆盖 `mosquitosack.lua` 全文、蚊子掉落表与死亡／作祟／splat 路径、`balloons_empty.lua` 全文、`healer`／`repairer`／`repairable`／`fillable` 相关段落（鲜度分支、消耗、装水）、MURDER／FILL／REPAIR 动作与候选入口、10 条配方行、`STACK_SIZE_MEDITEM`／`VITAE`／燃烧与作祟 helper、`deathloothandler` 存取与尸骸掉落链、`chinese_s.po` 名称。
- 逐文件复核：`mosquitosack.lua` 无 `perishable` 挂载；`MATERIALS.VITAE` 的 `repairable` 消费者全库仅 `mosquitomusk.lua` 一处。
- 游戏运行：未运行；概率与装水行为均为源码审阅。
- 机械校验（页面、导航、严格链接、索引一致性）待验收时运行。

相关：[[reference/prefabs/mosquitomusk]]、[[reference/components/healer]]、[[reference/components/repairer]]、[[reference/components/fillable]]。

返回 [[reference/prefabs/index|原版 Prefab 索引]]。
