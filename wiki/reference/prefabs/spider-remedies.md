---
title: "蜘蛛腺与治疗黏团（spidergland／spider_healer_item）"
created: "2026-11-10"
updated: "2026-11-10"
type: prefab
summary: "蜘蛛阵营两件治疗物品：蜘蛛腺是通用 8 点治疗品与韦伯／Wendy／WX-78 模组的配方材料，也可直接对蜘蛛用药；治疗黏团是韦伯专属配方，healer 组件治疗量置 0，实际回血全在回调——目标韦伯 +8、以持有者为圆心半径 5 内所有蜘蛛各 +80。两件均无新鲜度、默认堆叠 20。"
aliases: ["spidergland", "spider_healer_item", "蜘蛛腺", "治疗黏团", "Spider Gland", "Healing Glop", "蜘蛛腺体"]
tags: [prefab, item]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/spidergland.lua", "prefabs/spider_healer_item.lua", "components/healer.lua", "components/inventoryitem.lua", "components/lootdropper.lua", "components/stackable.lua", "components/health.lua", "prefabs/spider.lua", "prefabs/spiderhole.lua", "prefabs/moonspiderden.lua", "prefabs/webber.lua", "prefabs/skilltree_walter.lua", "entityscript.lua", "componentutil.lua", "prefabs/corpses.lua", "standardcomponents.lua", "actions.lua", "componentactions.lua", "recipes.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 蜘蛛腺与治疗黏团（spidergland／spider_healer_item）

## 快速判断

- 职责与适用：两件可拾取治疗物品（inventoryitem）。蜘蛛腺挂标准 `healer`（8 点）；治疗黏团挂 `healer` 但治疗量置 0，回血全部由 `SetOnHealFn` 回调手动执行：韦伯本人 +8、半径 5 内所有蜘蛛各 +80。[蜘蛛腺组件](../../../raw/current/scripts/prefabs/spidergland.lua#L41-L42)、[黏团接线](../../../raw/current/scripts/prefabs/spider_healer_item.lua#L71-L73)、[OnHealFn](../../../raw/current/scripts/prefabs/spider_healer_item.lua#L25-L45)
- 优先使用：查"蜘蛛治疗／给蜘蛛回血／蜘蛛腺"到本页；healer 组件的通用结算与门禁读 [[reference/components/healer]]；表现层（三枚治疗 FX 与治疗声）归 [[reference/prefabs/spider_heal_fx]]。
- 重要副作用：蜘蛛腺同时是 5 条配方的材料（见「获得」）；两件都可燃、可传火、可浮水；作祟只弹飞；**无新鲜度**（见「燃烧与作祟」）。
- 修改边界：本页只写物品与治疗结算侧；护士蜘蛛（spider_healer）的 SG 群疗在 [[reference/prefabs/spider_heal_fx]]；蜘蛛本体的行为页 Wiki 待建。治疗音由 `OnHealFn` 播放、FX 三名生成也在此回调（[L26-L37](../../../raw/current/scripts/prefabs/spider_healer_item.lua#L26-L37)），本页只记触点不展开表现细节。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `spidergland` | `fn` 单工厂：网络段（`MakeInventoryPhysics`；bank/build 均 `spider_gland`、播 `idle`；加 `cattoy` 标签；`MakeInventoryFloatable`）→ `SetPristine` → 客户端返回 → master 段（`MakeSmallBurnable(TINY_BURNTIME)`＋`MakeSmallPropagator`＋`MakeHauntableLaunchAndIgnite`；`inspectable`／`inventoryitem`／`stackable`／`tradable`／`healer`） | 无变体。整段没有 `edible`，**不可食用**。[构造全文](../../../raw/current/scripts/prefabs/spidergland.lua#L6-L45) |
| `spider_healer_item` | `fn` 单工厂：网络段（`MakeInventoryPhysics`；bank/build 均 `spider_healer_item`、播 `idle`；`MakeInventoryFloatable("small", 0.15, 0.9)`）→ `SetPristine` → 客户端返回 → master 段（`inventoryitem`／`inspectable`／`stackable`／`healer`（0＋OnHealFn）；`MakeSmallBurnable(SMALL_BURNTIME)`＋`MakeSmallPropagator`＋`MakeHauntableLaunch`） | 无变体。prefabs 依赖声明 `spider_heal_fx`、`spider_heal_target_fx`（`spider_heal_ground_fx` 由回调直接 SpawnPrefab，不在依赖表里）。[依赖 L6-L10](../../../raw/current/scripts/prefabs/spider_healer_item.lua#L6-L10)、[构造 L47-L80](../../../raw/current/scripts/prefabs/spider_healer_item.lua#L47-L80) |

- 名称键：`STRINGS.NAMES.SPIDERGLAND`＝蜘蛛腺、`STRINGS.NAMES.SPIDER_HEALER_ITEM`＝治疗黏团。[po L349840-L349842](../../../raw/current/scripts/languages/chinese_s.po#L349840-L349842)、[po L349870-L349872](../../../raw/current/scripts/languages/chinese_s.po#L349870-L349872)
- 两件都是默认 `stackable`（上限 `STACK_SIZE_MEDITEM`＝20，[stackable.lua L28](../../../raw/current/scripts/components/stackable.lua#L28)、[tuning L82](../../../raw/current/scripts/tuning.lua#L82)）；蜘蛛腺另有 `tradable`（参与交易类机制）与 `cattoy`（浣熊猫玩具检索标签，[spidergland L19](../../../raw/current/scripts/prefabs/spidergland.lua#L19)）。

## 治疗：healer 组件的接入与例外

通用结算复用 [[reference/components/healer]] 的结论：HEAL 动作候选要求持有者或目标 `replica.health:CanHeal()`（或物品带 `healerbuffs`），对自己使用与对场景目标使用各有一条候选入口（[componentactions L2737-L2744](../../../raw/current/scripts/componentactions.lua#L2737-L2744)、[L1346-L1360](../../../raw/current/scripts/componentactions.lua#L1346-L1360)）；`HEAL.fn` 排除死亡与幽灵后调 `healer:Heal`（[actions L3217-L3226](../../../raw/current/scripts/actions.lua#L3217-L3226)）；`Heal` 按顺序执行 canhealfn → 基础量×执行者 efficientuser 乘数 → `health.canheal` 才 DoDelta → onhealfn → 消耗一份（[healer.lua L21-L53](../../../raw/current/scripts/components/healer.lua#L21-L53)）。满血或未实际回血也照常回调并消耗。

- **蜘蛛腺＝标准接入**：`SetHealthAmount(TUNING.HEALING_MEDSMALL)`＝8 点（[spidergland L41-L42](../../../raw/current/scripts/prefabs/spidergland.lua#L41-L42)、[tuning L2023](../../../raw/current/scripts/tuning.lua#L2023)）。无 canhealfn／onhealfn，可对任何带 health 的目标使用（含蜘蛛与玩家）。沃尔特激活急救技能后 efficientuser 的 HEAL 倍率 1.5 使基础量变 12（[skilltree_walter L410-L413](../../../raw/current/scripts/prefabs/skilltree_walter.lua#L410-L413)、[tuning L7369](../../../raw/current/scripts/tuning.lua#L7369)、[healer.lua L36](../../../raw/current/scripts/components/healer.lua#L36)），仍受目标 health 结算封顶。
- **治疗黏团＝例外接入**：`SetHealthAmount(0)`＋`SetOnHealFn(OnHealFn)`（[L72-L73](../../../raw/current/scripts/prefabs/spider_healer_item.lua#L72-L73)）。组件自身 delta 为 0（0×乘数仍 0），实际回血全部在回调里：
  - 目标带 `spiderwhisperer` 标签 → 手动 `health:DoDelta(HEALING_MEDSMALL)`＝+8 并落 target 光（[L31-L34](../../../raw/current/scripts/prefabs/spider_healer_item.lua#L31-L34)）。该标签是韦伯基础标签（[webber.lua L37-L38](../../../raw/current/scripts/prefabs/webber.lua#L37-L38)），非技能门禁。手动分支直连 DoDelta：**不乘 efficientuser（沃尔特技能不放大黏团）、不查 `health.canheal`**。
  - 无论目标是谁，都以**物品自身位置**为圆心（在背包中时＝持有者位置：入包即 `owner:AddChild`＋本地坐标归零，[inventoryitem.lua L348-L354](../../../raw/current/scripts/components/inventoryitem.lua#L348-L354)）找半径 `SPIDER_HEALING_ITEM_RADIUS`＝5 内 `spider` 标签实体（排除 FX／NOCLICK／DECOR／INLIMBO／creaturecorpse，[L12-L13](../../../raw/current/scripts/prefabs/spider_healer_item.lua#L12-L13)），每只手动 `DoDelta(SPIDER_HEALING_ITEM_AMOUNT)`＝+80（[L38-L44](../../../raw/current/scripts/prefabs/spider_healer_item.lua#L38-L44)、[tuning L6266-L6267](../../../raw/current/scripts/tuning.lua#L6266-L6267)）。蜘蛛 `health.canheal` 默认为真（[health.lua L97](../../../raw/current/scripts/components/health.lua#L97)），此处同样无门禁。
  - 治疗对象语义：源码注释（[L30](../../../raw/current/scripts/prefabs/spider_healer_item.lua#L30)）说"手动治疗韦伯、因为只有韦伯应该被它治疗"——按代码事实，注释只描述**标签分支的选择键**（spiderwhisperer＝韦伯），不是使用限制：任何角色都可持黏团对任何健康目标使用，半径 5 的蜘蛛群疗对谁都生效。
- 对"非韦伯、非蜘蛛"的目标使用黏团：只播音效与三枚 FX、0 回血，仍消耗一份。[L25-L45](../../../raw/current/scripts/prefabs/spider_healer_item.lua#L25-L45)

## 获得

### 蜘蛛腺来源

- **蜘蛛掉落**：8 种蜘蛛（spider／spider_warrior／spider_hider／spider_spitter／spider_dropper／spider_moon／spider_healer／spider_water，[spider.lua L1087-L1094](../../../raw/current/scripts/prefabs/spider.lua#L1087-L1094)）共用一套随机掉落池：monstermeat 权重 1／silk 0.5／spidergland 0.5，`numrandomloot = 1` → 每次 25% 出 1 个蜘蛛腺（[spider.lua L645-L650](../../../raw/current/scripts/prefabs/spider.lua#L645-L650)）。死亡瞬间掷定并存入 `deathloothandler`，随尸体保存、尸骸被销毁时掉出（[entityscript L2181-L2192](../../../raw/current/scripts/entityscript.lua#L2181-L2192)、[componentutil L1579](../../../raw/current/scripts/componentutil.lua#L1579)、[corpses.lua L489](../../../raw/current/scripts/prefabs/corpses.lua#L489)）。
- **作祟替换表**：蜘蛛身上注册了作祟专属随机表 `AddRandomHauntedLoot("spidergland", 1)`——作祟状态下 `GenerateLoot` 的随机表被整体替换为仅蜘蛛腺（权重 1，即必出），同样在死亡掷 Loot 时按 `hauntable.haunted` 生效（[spider.lua L649](../../../raw/current/scripts/prefabs/spider.lua#L649)、[lootdropper L57-L65](../../../raw/current/scripts/components/lootdropper.lua#L57-L65)、[L87-L88](../../../raw/current/scripts/components/lootdropper.lua#L87-L88)；蜘蛛可被作祟：[spider.lua L722](../../../raw/current/scripts/prefabs/spider.lua#L722)）。
- **巢穴摧毁**：蜘蛛洞 `spider_hole` 与月岩蜘蛛巢 `moon_spider_hole` 的共享掉落表各含 `spidergland 0.25`（摧毁时 25% 概率 1 个）（[spiderhole.lua L31-L36](../../../raw/current/scripts/prefabs/spiderhole.lua#L31-L36)、[moonspiderden.lua L28-L34](../../../raw/current/scripts/prefabs/moonspiderden.lua#L28-L34)）。

### 配方消费（蜘蛛腺）

| 配方 | 材料（蜘蛛腺份数） | 门禁 | 证据 |
|---|---|---|---|
| `ghostlyelixir_slowregen`（缓愈灵液） | 1 | `builder_tag="elixirbrewer"` | [recipes L151](../../../raw/current/scripts/recipes.lua#L151) |
| `spidereggsack`（蜘蛛卵囊袋） | 4 | `builder_tag="spiderwhisperer"` | [recipes L324](../../../raw/current/scripts/recipes.lua#L324) |
| `healingsalve`（治疗药膏） | 1 | `TECH.SCIENCE_ONE` | [recipes L414](../../../raw/current/scripts/recipes.lua#L414) |
| `reviver`（复苏仪） | 1 | `TECH.NONE`＋生命上限材料 40 | [recipes L420](../../../raw/current/scripts/recipes.lua#L420) |
| `wx78module_maxhealth`／`wx78module_maxhealth2` | 1／2 | `TECH.ROBOTMODULECRAFT_ONE`＋`upgrademoduleowner` | [recipes L753-L754](../../../raw/current/scripts/recipes.lua#L753-L754) |

### 治疗黏团配方

- `spider_healer_item`＝蜂蜜 2＋灰 2＋蛛丝 2，`TECH.NONE`，`builder_tag="spiderwhisperer"`（[recipes L328](../../../raw/current/scripts/recipes.lua#L328)）；灰烬侧的材料链见 [[reference/prefabs/ashes-and-charcoal]]。全库无其他生成点（仅配方产物）。

## 燃烧与作祟（无新鲜度）

- **无新鲜度**：`spidergland.lua`、`spider_healer_item.lua` 全文均无 `perishable` 挂载（逐文件复核），两件不腐坏。
- 燃烧：蜘蛛腺 `MakeSmallBurnable(TINY_BURNTIME)`＝`seg_time×0.1`＝3 秒；黏团 `SMALL_BURNTIME`＝`seg_time×0.25`＝7.5 秒（[spidergland L29](../../../raw/current/scripts/prefabs/spidergland.lua#L29)、[spider_healer_item L75](../../../raw/current/scripts/prefabs/spider_healer_item.lua#L75)、[tuning L1680-L1681](../../../raw/current/scripts/tuning.lua#L1680-L1681)）；两件都有 `MakeSmallPropagator` 可传火。
- 作祟：黏团 `MakeHauntableLaunch` 只弹飞（[L78](../../../raw/current/scripts/prefabs/spider_healer_item.lua#L78)、[helper L1053-L1070](../../../raw/current/scripts/standardcomponents.lua#L1053-L1070)）；蜘蛛腺 `MakeHauntableLaunchAndIgnite` 名义上"弹飞并点燃"，但点燃分支整段被 `#HAUNTFIX` 注释，实际同样只弹飞（[spidergland L31](../../../raw/current/scripts/prefabs/spidergland.lua#L31)、[helper L1191-L1217](../../../raw/current/scripts/standardcomponents.lua#L1191-L1217)）。

## 常见陷阱与限制

- **误区：治疗黏团只有韦伯能用**：使用门禁只有通用 HEAL 条件（持有者／目标可治疗）；韦伯分支只是回调里对 `spiderwhisperer` 标签的额外 +8，任何角色的黏团使用都会触发半径 5 的蜘蛛群疗。[L25-L45](../../../raw/current/scripts/prefabs/spider_healer_item.lua#L25-L45)
- **边界：黏团的 0 治疗量不是"无治疗"**：healer 组件自身 DoDelta(0)，但回调手动 +8／+80 照常结算；反之，对非韦伯非蜘蛛目标使用就是纯表现、0 回血且照常消耗。[healer.lua L36-L44](../../../raw/current/scripts/components/healer.lua#L36-L44)
- **边界：沃尔特急救 ×1.5 只放大蜘蛛腺**：倍率乘在 healer 基础量上；黏团的回血是回调直连 DoDelta，不经过该乘数。[healer.lua L36](../../../raw/current/scripts/components/healer.lua#L36)、[OnHealFn L32/L42](../../../raw/current/scripts/prefabs/spider_healer_item.lua#L31-L34)
- **边界：蜘蛛腺不可食用**：无 `edible` 组件；"蜘蛛腺能吃"只对蜘蛛侧成立（喂食蜘蛛走蜘蛛的 eater，与组件无关）。[构造 L35-L43](../../../raw/current/scripts/prefabs/spidergland.lua#L35-L43)
- **边界：作祟不会点燃**：`MakeHauntableLaunchAndIgnite` 的点燃分支被注释，别按 helper 名推断。[helper L1191-L1217](../../../raw/current/scripts/standardcomponents.lua#L1191-L1217)
- **边界：群疗半径以持有者为圆心**：黏团在背包中时物品坐标即持有者坐标；把黏团丢在地上再对远处目标用药，半径扫描仍以物品落点为准。[inventoryitem.lua L348-L354](../../../raw/current/scripts/components/inventoryitem.lua#L348-L354)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`healer` | 蜘蛛腺标准 8 点；黏团 0＋OnHealFn 例外接入 | 服务端结算；动作候选两端生成 | [[reference/components/healer]]；[spidergland L41-L42](../../../raw/current/scripts/prefabs/spidergland.lua#L41-L42)、[黏团 L71-L73](../../../raw/current/scripts/prefabs/spider_healer_item.lua#L71-L73) |
| 组件：`stackable`／`inventoryitem` | 默认堆叠 20；黏团回调用时消耗一份（堆叠取一） | 服务端 | [[reference/components/stackable]]；[healer.lua L46-L50](../../../raw/current/scripts/components/healer.lua#L46-L50) |
| 标签：`spiderwhisperer` | 黏团手动 +8 的选择键 | 韦伯基础标签，非技能门禁 | [webber.lua L37-L38](../../../raw/current/scripts/prefabs/webber.lua#L37-L38) |
| 调参：`SPIDER_HEALING_ITEM_AMOUNT`／`_RADIUS` | 黏团群疗量 80／半径 5 | 服务端 | [tuning L6266-L6267](../../../raw/current/scripts/tuning.lua#L6266-L6267) |
| 关联 Prefab：`spider` 等 8 种蜘蛛 | 蜘蛛腺的主要掉落源；也是黏团群疗对象（`spider` 标签） | 服务端掉落；群疗含全部 spider 标签实体 | [掉落池 L645-L650](../../../raw/current/scripts/prefabs/spider.lua#L645-L650)、[过滤标签 L12-L13](../../../raw/current/scripts/prefabs/spider_healer_item.lua#L12-L13) |
| 关联 Prefab：`spider_heal_fx` 三名 | 黏团回调生成的表现（地面圈＋本体＋target 光） | 服务端生成、一帧表现 | [[reference/prefabs/spider_heal_fx]]；[L33/L36-L37/L43](../../../raw/current/scripts/prefabs/spider_healer_item.lua#L36-L37) |
| 数据：`recipes.lua` | 蜘蛛腺 5 条材料消费＋黏团配方 | 制作端 | [L151/L324/L414/L420/L753-L754/L328](../../../raw/current/scripts/recipes.lua#L414) |
| 资源：`anim/spider_gland.zip`／`anim/spider_healer_item.zip` | 各自 bank/build 同名，`idle` | assets 声明 | [spidergland L1-L4/L15-L17](../../../raw/current/scripts/prefabs/spidergland.lua#L1-L4)、[黏团 L1-L4/L55-L57](../../../raw/current/scripts/prefabs/spider_healer_item.lua#L55-L57) |

## 常见接入：模组给蜘蛛治疗物品改量

- 前提与选择：改动蜘蛛腺的治疗点数用 `AddPrefabPostInit`，复用原版工厂与动作链，无需自建 healer。
- 生效与撤销：每次实例初始化生效，停用模组后回原值；不影响黏团（它走回调）。

```lua
AddPrefabPostInit("spidergland", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    inst.components.healer:SetHealthAmount(20)
end)
```

- 调整与验证：20 为基础 HP，仍乘执行者 efficientuser 倍率并受目标 health 结算封顶；改黏团的治疗量无效（组件量为 0），需改 `OnHealFn` 侧或用 `AddComponent` 后覆盖回调。

## 验证记录

- 源码审阅：2026-11-10，steam-buildid:26474873（与 `raw/current/BUILD.txt` 一致）。覆盖两份 prefab 全文、`components/healer.lua` 全文、HEAL 动作定义与候选、`OnHealFn` 半径扫描与标签过滤、蜘蛛 8 注册掉落池与作祟替换表、两个巢穴掉落表、5＋1 条配方行、沃尔特 efficientuser 倍率、韦伯标签、`inventoryitem` 入包定位、`deathloothandler` 存取与尸骸掉落、燃烧／传火／作祟 helper、`chinese_s.po` 两名。
- 逐文件复核：`spidergland.lua`／`spider_healer_item.lua` 无 `perishable` 挂载；蜘蛛腺无 `edible`。
- 游戏运行：未运行；治疗数值与半径均为源码审阅值。
- 机械校验（页面、导航、严格链接、索引一致性）待验收时运行。

相关：[[reference/components/healer]]、[[reference/prefabs/spider_heal_fx]]、[[reference/prefabs/silk]]、[[reference/prefabs/ashes-and-charcoal]]。

返回 [[reference/prefabs/index|原版 Prefab 索引]]。
