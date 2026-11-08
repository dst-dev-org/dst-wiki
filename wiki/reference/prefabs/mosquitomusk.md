---
title: "小痒痒（mosquitomusk）"
created: "2026-11-09"
updated: "2026-11-10"
type: prefab
summary: "沃特独有的驱蚊物品：蚊子血囊＋牛毛的一阶沼泽技能配方，携带时只影响蚊子的常规重选与蚊子炸弹刷出时的初始选敌（受击锁定与范围伤害不读这个标签），8 天腐坏成腐烂食物，可用蚊子血囊整格刷新鲜度。"
aliases: ["mosquitomusk", "小痒痒", "Lil' Itchy"]
tags: [prefab, item]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/mosquitomusk.lua", "prefabs/mosquito.lua", "prefabs/mosquitobomb.lua", "prefabs/skilltree_wurt.lua", "prefabs/mosquitosack.lua", "components/repairable.lua", "components/repairer.lua", "components/perishable.lua", "components/stackable.lua", "components/builder.lua", "components/builder_replica.lua", "recipes.lua", "recipes_filter.lua", "constants.lua", "actions.lua", "componentactions.lua", "stringutil.lua", "standardcomponents.lua", "tuning.lua", "screens/redux/scrapbookdata.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 小痒痒（mosquitomusk）

## 快速判断

- 职责与适用：可拾取物品（inventoryitem，没有 edible），作用是缩小被选中的机会——背包里只要有 1 个带 mosquitomusk 标签的东西，蚊子常规重选目标、蚊子炸弹刷出时给蚊子指派初始目标会跳过你；**受击后的锁定与炸弹命中的独立范围伤害不读这个标签**（见「标签消费」）。[标签](../../../raw/current/scripts/prefabs/mosquitomusk.lua#L18-L19)、[蚊子筛选](../../../raw/current/scripts/prefabs/mosquito.lua#L89-L111)
- 优先使用：沃特技能线与配方见下文「获得」；鲜度刷新见「修理」；标签消费见「标签消费」。
- 重要副作用：8 天腐坏成 spoiled_food；可燃、可传火、可浮水；**不可堆叠**；作祟只会被弹飞、不会点燃。
- 修改边界：蚊子本体与沃特技能树全文不在本页（Wiki 待建）；蚊子炸弹的弹道与范围伤害见 [[reference/prefabs/ranged-weapons]]。本页只写诱饵这一侧与被消费的标签。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `mosquitomusk` | `fn` 单工厂：网络段（Transform／AnimState／Network；`MakeInventoryPhysics`；bank 与 build 均 `mosquitomusk`、播 `idle`；`MakeInventoryFloatable("small", 0.08, 0.8)`；加 `mosquitomusk`、`show_spoilage` 两个标签）→ `SetPristine` → 客户端返回 → master 段（`repairable`／`inspectable`／`inventoryitem`／`perishable`／`burnable`／`propagator`／`hauntable`） | 无变体（快照内没有皮肤条目）。整段没有 `stackable`、`edible`、`fuel`、`equippable`、`light`，所以不可堆叠、不能吃、不能当燃料。[构造 L11-L54](../../../raw/current/scripts/prefabs/mosquitomusk.lua#L11-L54) |

- 名称键 `STRINGS.NAMES.MOSQUITOMUSK`＝小痒痒（英文 Lil' Itchy）。[po L345940-L345942](../../../raw/current/scripts/languages/chinese_s.po#L345940-L345942)
- 标签两级：`mosquitomusk` 是消费端检索条件（蚊子、蚊子炸弹、沃特技能都按它找），`show_spoilage` 只负责新鲜度图标。[L18-L19](../../../raw/current/scripts/prefabs/mosquitomusk.lua#L18-L19)
- 浮水参数：尺寸 `small`、垂直偏移 0.08、缩放 0.8，掉进水里漂浮可捡。[调用 L27](../../../raw/current/scripts/prefabs/mosquitomusk.lua#L27)、[helper](../../../raw/current/scripts/standardcomponents.lua#L1772-L1790)
- 出厂即 `StartPerishing`，不留"未开始计时"的空窗。[L43-L46](../../../raw/current/scripts/prefabs/mosquitomusk.lua#L43-L46)

## 获得：沃特技能配方

- 配方：`mosquitosack` 1（蚊子血囊）＋`beefalowool` 1（牛毛），`TECH.NONE`，`builder_skill = "wurt_mosquito_craft_1"`。[recipes L113](../../../raw/current/scripts/recipes.lua#L113)
- 门禁：制作端用 `skilltreeupdater:IsActivated(recipe.builder_skill)` 判断，客户端同样读 replica 版本；未激活时配方不可制作。[服务端](../../../raw/current/scripts/components/builder.lua#L876-L877)、[客户端](../../../raw/current/scripts/components/builder_replica.lua#L312-L313)
- 技能位置：`wurt_mosquito_craft_1` 是沼泽大师组（swampmaster）的根技能，**只连** `_2`；`_2`（抓蚊）再连 `_3`，并在激活时解锁蚊子炸弹（`mosquitobomb`）、嗡嗡肥料（`mosquitofertilizer`）与蚊血注射液（`mosquitomermsalve`）三条配方（简中名见 `chinese_s.po`）。[L179-L188](../../../raw/current/scripts/prefabs/skilltree_wurt.lua#L179-L188)、[`_2` 的 connects 与回调 L191-L208](../../../raw/current/scripts/prefabs/skilltree_wurt.lua#L191-L208)、[三条配方 L114-L116](../../../raw/current/scripts/recipes.lua#L114-L116)、[简中名 L345924-L345937](../../../raw/current/scripts/languages/chinese_s.po#L345924-L345937)
- 同组三阶的分工：`_2` 注册抓蚊标签（见「标签消费」），`_3` 自身没有 onactivate／ondeactivate，效果全写在消费者一侧（已核到蚊子炸弹多刷 2 只，其余消费者见各自源码）。[L191-L215](../../../raw/current/scripts/prefabs/skilltree_wurt.lua#L191-L215)、[炸弹 L38-L43](../../../raw/current/scripts/prefabs/mosquitobomb.lua#L38-L43)
- 制作菜单归到沃特分组。[recipes_filter L120](../../../raw/current/scripts/recipes_filter.lua#L120)
- 材料之一蚊子血囊来自蚊子 50% 掉落。[掉落表 L25-L28](../../../raw/current/scripts/prefabs/mosquito.lua#L25-L28)

## 修理：用蚊血囊刷新鲜度

- 本件挂 `repairable`，材料 `MATERIALS.VITAE`（`"vitae"`），并设 `noannounce = true`。[L35-L38](../../../raw/current/scripts/prefabs/mosquitomusk.lua#L35-L38)、[材料常量](../../../raw/current/scripts/constants.lua#L1959)
- 修理物是蚊子血囊：`repairer` 材料同为 VITAE，`perishrepairpercent = 1`（一份回满鲜度）。[mosquitosack L38-L40](../../../raw/current/scripts/prefabs/mosquitosack.lua#L38-L40)
- 动作候选条件（缺一不可）：右键、修理物带 `freshen_vitae` 标签、目标带 `repairable_vitae` 且已有 fresh／stale／spoiled 之一（实例化时由 perishable 打上）；骑乘时还要求目标归自己（`IsGrandOwner`），否则 return；搬重物是这条骑乘判断的 **`elseif` 分支**——只在未骑乘时才检查，命中就不给候选。候选只按标签判定、不看剩余鲜度，所以**满鲜度的小痒痒也会出现候选**，只是结算拒绝。[入口](../../../raw/current/scripts/componentactions.lua#L1679-L1700)、[骑乘与搬重物分支](../../../raw/current/scripts/componentactions.lua#L1679-L1687)、[repairer 打标签](../../../raw/current/scripts/components/repairer.lua#L8-L24)
- 动作串：`ACTIONS.REPAIR.strfn` 对 `repairable_vitae` 返回 `REFRESH`，简中动作名"刷新"。[actions L975-L982](../../../raw/current/scripts/actions.lua#L975-L982)、[po L1674-L1677](../../../raw/current/scripts/languages/chinese_s.po#L1674-L1677)
- 结算：`REPAIR.fn` 把目标与材料交给 `repairable:Repair`；材料必须相等，随后走 perishable 分支 `SetPercent(现有 + 1)` 并把血囊消耗 1 个；已经满鲜度时该分支在消耗之前就返回 false，**血囊不消耗**。三个对照：普通鲜食没有 `repairable_vitae` → 没有候选；吃到 95% 的小痒痒 → 候选出现且能补满；**满鲜度的小痒痒 → 候选照样出现，点下去被拒绝、不扣血囊**。[actions L984-L1007](../../../raw/current/scripts/actions.lua#L984-L1007)、[repairable 分支](../../../raw/current/scripts/components/repairable.lua#L126-L133)、[消耗](../../../raw/current/scripts/components/repairable.lua#L172-L176)
- `noannounce` 的消费端：检查文本里"可修理"那句被跳过，所以物品描述不会提示它能刷新。[stringutil L352-L354](../../../raw/current/scripts/stringutil.lua#L352-L354)

## 标签消费：mosquitomusk

| 消费者 | 行为 | 条件与证据 |
|---|---|---|
| 蚊子 `mosquito` | `KillerRetarget` 常规重选目标时跳过背包里 `FindItem` 命中 `mosquitomusk` 的实体 | 重选间隔 2 秒；同队（`IsAlly`）另判。**例外**：蚊子受到攻击后其 `OnAttacked` 仍会 `SetTarget(攻击者)` 并 `ShareTarget`，这条路径不查标签。[L89-L111](../../../raw/current/scripts/prefabs/mosquito.lua#L89-L111)、[L240](../../../raw/current/scripts/prefabs/mosquito.lua#L240)、[受击锁定 L134-L141](../../../raw/current/scripts/prefabs/mosquito.lua#L134-L141) |
| 蚊子炸弹 `mosquitobomb` | 炸开刷蚊子时，不给携带者 `SuggestTarget`（只影响**初始选敌**） | 生成循环里比对每个目标背包；命中时的独立范围伤害走 `DoImpactDamage`，不读标签。炸弹本体见 [[reference/prefabs/ranged-weapons]]。[选敌 L30-L70](../../../raw/current/scripts/prefabs/mosquitobomb.lua#L30-L70)、[范围伤害 L75-L88](../../../raw/current/scripts/prefabs/mosquitobomb.lua#L75-L88) |
| 沃特技能 `wurt_mosquito_craft_2` | 背包里有 ≥1 个带标签物品就加 `mosquitograbber` 标签；监听 itemget／itemlose 实时刷新，技能关闭时摘标签 | 只在本技能激活期间生效。[L45-L51](../../../raw/current/scripts/prefabs/skilltree_wurt.lua#L45-L51)、[L198-L207](../../../raw/current/scripts/prefabs/skilltree_wurt.lua#L198-L207) |
| 抓活蚊子 | 蚊子 `inventoryitem.grabbableoverridetag = "mosquitograbber"`、`canbepickedupalive = true`；`ACTIONS.PICKUP` 的结算在"活体只放行非玩家"之后，再用这个标签放行带标签的玩家 | 这是小痒痒唯一改变玩家能力的路径。[L204-L207](../../../raw/current/scripts/prefabs/mosquito.lua#L204-L207)、[拾取结算](../../../raw/current/scripts/actions.lua#L894-L902) |

## 腐坏、燃烧与浮水

- 计时定义：`PERISH_FASTISH = 8 × total_day_time × perish_warp`＝8×480＝**3840 秒**（8 天）；图鉴数据同样记 3840。[tuning L2047](../../../raw/current/scripts/tuning.lua#L2047)、[基准 L19-L20](../../../raw/current/scripts/tuning.lua#L19-L20)、[图鉴](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L976)
- 结算：`perishable:Update` 把 `localPerishMultiplyer`（默认 1）乘上地面修正 `PERISH_GROUND_MULT`＝1.5 → 放在地上约 **2560 秒≈5.33 天**；在背包里按 3840 秒走，冰箱／冬季等再有各自修正。[公式](../../../raw/current/scripts/components/perishable.lua#L127-L133)、[地面分支](../../../raw/current/scripts/components/perishable.lua#L97-L105)、[tuning L2083](../../../raw/current/scripts/tuning.lua#L2083)
- 到点：没有自定义 `perishfn`，走 `onperishreplacement = "spoiled_food"`，在原来的位置（或原背包格）换上腐烂食物；本件没有 stackable，所以恒为单件。[L43-L46](../../../raw/current/scripts/prefabs/mosquitomusk.lua#L43-L46)、[替换逻辑](../../../raw/current/scripts/components/perishable.lua#L293-L318)
- 燃烧：`MakeSmallBurnable` 用 `TINY_BURNTIME`＝`seg_time × 0.1`＝**3 秒**；燃烧期间 `persists = false`，烧完在陆地生成 1 个 `ash` 后移除（水里不留灰）。[L48](../../../raw/current/scripts/prefabs/mosquitomusk.lua#L48)、[tuning L1680](../../../raw/current/scripts/tuning.lua#L1680)、[烧毁结果](../../../raw/current/scripts/standardcomponents.lua#L12-L44)
- 传火：`MakeSmallPropagator` 给的是闪点 5–10 秒、衰减 0.5、传播半径 3–5、热输出 3–5、伤害半径 2。[L49](../../../raw/current/scripts/prefabs/mosquitomusk.lua#L49)、[propagator](../../../raw/current/scripts/standardcomponents.lua#L200-L213)
- 作祟：`MakeHauntableLaunchAndIgnite` 只执行弹飞分支，点燃分支在源码里被注释掉；函数仍返回 false，但 `Launch` 已经发生。[L51](../../../raw/current/scripts/prefabs/mosquitomusk.lua#L51)、[helper](../../../raw/current/scripts/standardcomponents.lua#L1191-L1217)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`repairable` | 材料 VITAE、`noannounce = true`；只由"刷新鲜度"这一条分支结算 | 服务端；非 workable／health 修复 | [[reference/components/repairable]]；[L35-L38](../../../raw/current/scripts/prefabs/mosquitomusk.lua#L35-L38) |
| 组件：`repairer`（在蚊子血囊上） | 提供 `freshen_vitae` 与 `perishrepairpercent = 1` | 服务端；材料须与本件一致 | [[reference/components/repairer]]；[mosquitosack L38-L40](../../../raw/current/scripts/prefabs/mosquitosack.lua#L38-L40) |
| 组件：`perishable` | 3840 秒、`spoiled_food` 替换、出厂即计时 | 服务端；地面另有 1.5 倍修正 | [[reference/components/perishable]]；[L43-L46](../../../raw/current/scripts/prefabs/mosquitomusk.lua#L43-L46) |
| 组件：`burnable`／`propagator` | 3 秒烧毁、可把火传给邻近物 | 服务端；燃烧即 `persists = false` | [L48-L49](../../../raw/current/scripts/prefabs/mosquitomusk.lua#L48-L49) |
| 组件：`floater` | 浮水，尺寸 small、偏移 0.08、缩放 0.8 | 两端（floater 在 pristine 前挂载） | [L27](../../../raw/current/scripts/prefabs/mosquitomusk.lua#L27) |
| 组件：`inventoryitem` | 可拾取、可入箱；没有 `stackable` 所以一格一件 | 服务端 | [[reference/components/inventoryitem]]；[L41](../../../raw/current/scripts/prefabs/mosquitomusk.lua#L41) |
| API／数据：`recipe.builder_skill` | 制作门禁键 `wurt_mosquito_craft_1` | 主端与客户端各查一次技能树 | [recipes L113](../../../raw/current/scripts/recipes.lua#L113)、[builder](../../../raw/current/scripts/components/builder.lua#L876-L877) |
| 关联 Prefab：`mosquito` | 消费本标签的飞行生物 | Wiki 待建 | [L89-L111](../../../raw/current/scripts/prefabs/mosquito.lua#L89-L111) |
| 关联 Prefab：`mosquitobomb` | 炸开时按同一套标签筛目标 | 本体页已有 | [[reference/prefabs/ranged-weapons]]；[L30-L70](../../../raw/current/scripts/prefabs/mosquitobomb.lua#L30-L70) |
| 关联 Prefab：`mosquitosack` | 配方材料＋刷新鲜度的消耗品 | 有 `stackable`，与本件不同 | [mosquitosack L33-L40](../../../raw/current/scripts/prefabs/mosquitosack.lua#L33-L40) |
| 资源：`anim/mosquitomusk.zip` | bank 与 build 同名，idle 动画 `idle` | 资源声明在 assets | [L1-L4](../../../raw/current/scripts/prefabs/mosquitomusk.lua#L1-L4)、[L23-L25](../../../raw/current/scripts/prefabs/mosquitomusk.lua#L23-L25) |

## 常见陷阱与限制

- **误区：可以叠放**：整个 `fn` 没有 `stackable`，一格只放 1 件；配方材料里的蚊子血囊反而有 `stackable`，两者别混。[构造 L35-L51](../../../raw/current/scripts/prefabs/mosquitomusk.lua#L35-L51)、[血囊 L35-L36](../../../raw/current/scripts/prefabs/mosquitosack.lua#L35-L36)
- **误区：地面也是 8 天**：3840 秒只是定义值，落地后还要乘 `PERISH_GROUND_MULT`＝1.5，实际约 5.33 天。[定义](../../../raw/current/scripts/tuning.lua#L2047)、[结算](../../../raw/current/scripts/components/perishable.lua#L127-L133)
- **边界：满鲜度点修理不消耗血囊**：`address_repair_amount` 在 `perishremainingtime >= perishtime` 时直接返回 false。[L126-L133](../../../raw/current/scripts/components/repairable.lua#L126-L133)
- **边界：`noannounce` 让提示消失**：物品描述里不会有"可修理"这句，玩家只能靠配方或攻略知道血囊能刷新它。[L38](../../../raw/current/scripts/prefabs/mosquitomusk.lua#L38)、[提示逻辑](../../../raw/current/scripts/stringutil.lua#L352-L354)
- **边界：只影响两类"选谁"，不构成免疫**：① 常规重选——`KillerRetarget` 由 `SetRetargetFunction(2, ...)` 调用，正在攻击你的蚊子不会立刻掉头；② 炸弹刷出时的初始 `SuggestTarget`。**不读标签的两条**：你主动打蚊子后，它 `OnAttacked` 会直接 `SetTarget` 并共享给同伴，照样咬你；炸弹命中时的 `DoImpactDamage` 按范围对 `_combat`／`_health` 实体逐个结算，不读该标签；但命中判定另有筛选——投掷者是玩家时其他玩家仍被排除（`pvpattacker` 只放行自己或非玩家），所以携带者会不会吃到伤害取决于它是否过这条筛选，而不是它有没有小痒痒。[L240](../../../raw/current/scripts/prefabs/mosquito.lua#L240)、[受击锁定](../../../raw/current/scripts/prefabs/mosquito.lua#L134-L141)、[范围伤害](../../../raw/current/scripts/prefabs/mosquitobomb.lua#L75-L88)
- **边界：不是食物也不是燃料**：没有 `edible` 与 `fuel`，虽可点燃但不是燃料物品。[构造 L35-L51](../../../raw/current/scripts/prefabs/mosquitomusk.lua#L35-L51)
- **边界：作祟只会弹飞**：点燃分支被注释，别按"作祟能点燃可燃物"推断。[helper](../../../raw/current/scripts/standardcomponents.lua#L1191-L1217)

## 验证记录

- 源码审阅：2026-11-09，steam-buildid:26474873（与 `raw/current/BUILD.txt` 一致）。覆盖 `prefabs/mosquitomusk.lua` 全文、`mosquito.lua`（掉落表、`IsMosquitoMusk`、`KillerRetarget`、`grabbableoverridetag`、retarget 间隔）、`mosquitobomb.lua` 生成段、`skilltree_wurt.lua` 蚊子三阶与 `CheckMosquitoCounts`、`mosquitosack.lua` 的 repairer 段、`repairable.lua`／`repairer.lua` 全文、`perishable.lua` 结算与替换、`actions.lua` 的 REPAIR、`componentactions.lua` 的修理入口、`stringutil.lua` 的 `noannounce` 消费、`components/builder.lua` 与 `components/builder_replica.lua` 的技能门禁、`standardcomponents.lua` 的燃烧／传播／浮水／作祟 helper、`recipes.lua` 与 `recipes_filter.lua`、`constants.lua` 的 VITAE、`tuning.lua` 数值、`scrapbookdata.lua` 的 3840、`chinese_s.po` 名称与动作串。
- 游戏运行：未运行；结论均为源码审阅，时长按定义值＋`PERISH_GROUND_MULT` 推导，非实测。
- 未覆盖：驱蚊在实战中的观感与命中边界、沃特技能树其余分支、蚊子本体的完整行为（蚊子炸弹见 [[reference/prefabs/ranged-weapons]]）。
- 修订（2026-11-09，独立检视 P3 逐条）：①技能链改为 `_1` 只连 `_2`、`_2` 连 `_3` 并解锁三条配方（补 `recipes.lua` L114-L116 证据）；②sources 补 `components/builder.lua`、`components/builder_replica.lua`。均按点名条目改，未顺手动其它段落。P3 补修（同日）：三条配方改官方简中名并补 `chinese_s.po` L345924-L345937 锚点；验证记录覆盖清单补 `components/builder.lua`、`components/builder_replica.lua`。
- 外部检视 R07/R16（2026-11-10）：修理候选补右键、`freshen_vitae`／`repairable_vitae` 材料标签，并把搬重物写成骑乘判断的 `elseif` 分支；写明满鲜度候选出现但结算拒绝且不扣血囊，附三个对照。标签范围收窄为「常规重选＋炸弹刷出时初始选敌」，补两条不读标签的例外：蚊子**受到攻击后**的 `SetTarget`／共享攻击者、炸弹 `DoImpactDamage`（沿用自身范围筛选，玩家投掷时其他玩家仍被排除），同步摘要与陷阱。

相关：[[reference/components/repairable]]、[[reference/components/repairer]]、[[reference/components/perishable]]。

返回 [[reference/prefabs/index|原版 Prefab 索引]]。
