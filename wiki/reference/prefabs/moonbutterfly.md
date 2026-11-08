---
title: "月蛾（moonbutterfly、moonbutterfly_placer）"
created: "2026-11-09"
updated: "2026-11-09"
type: prefab
summary: "prefabs/moonbutterfly.lua 的两个注册：会发光的月蛾活体（捕虫网 1 次活捉、叠 20、0.5 天常驻腐坏且持有时放慢到等效 2 天；场景内腐坏都在 3 秒后移除——非燃烧掉 1 张月蛾翅膀、燃烧只把掉落改写成 ash，背包/容器内腐坏直接移除；普通击杀按 death 结算、不走这个 3 秒定时器），以及放生用 placer；放生即消耗 1 只种下 moonbutterfly_sapling，与蝴蝶在本体行为上的逐项差异一并列出。"
aliases: ["moonbutterfly", "moonbutterfly_placer", "月蛾", "月光蝴蝶", "抓月蛾", "月蛾种树", "月亮药水变月蛾"]
tags: [prefab, creature]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/moonbutterfly.lua", "brains/moonbutterflybrain.lua", "stategraphs/SGbutterfly.lua", "prefabs/butterfly.lua", "prefabs/planted_tree.lua", "prefabs/moonbutterflywings.lua", "prefabs/moontree.lua", "prefabs/flower.lua", "components/stackable.lua", "components/perishable.lua", "components/workable.lua", "components/deployable.lua", "components/lootdropper.lua", "components/inventoryitem.lua", "components/inventoryitem_replica.lua", "standardcomponents.lua", "recipes.lua", "tuning.lua", "languages/chinese_s.po", "screens/redux/scrapbookdata.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 月蛾（moonbutterfly、moonbutterfly_placer）

## 快速判断

- 职责与适用：月岛的发光小生物，**既是活体也是"种子"**：捕虫网 1 次活捉、可叠 20 只；放生（PLANT 部署）会消耗 1 只、在原地种下 `moonbutterfly_sapling`。它复用蝴蝶的 `SGbutterfly`，但自带走动速度、光照与"常驻腐坏"计时。[构造](../../../raw/current/scripts/prefabs/moonbutterfly.lua#L104-L219)、[注册](../../../raw/current/scripts/prefabs/moonbutterfly.lua#L221-L222)
- 优先使用：查怎么抓、抓起来要注意什么看[捕捉与携带](#捕捉与携带)；查为什么会凭空消失／掉翅膀看[数值与计时](#数值与计时)；查放生种树看[放生与种植](#放生与种植)；查与蝴蝶的差别看[与蝴蝶的差异](#与蝴蝶的差异)。
- 重要副作用：它的 `perishable` **从构造起就在走表**，落地还吃 `PERISH_GROUND_MULT`＝1.5，所以世界里的野生月蛾大约 160 秒就会"饿死"；**场景内腐坏**都会在 3 秒后移除——非燃烧掉 1 张翅膀、燃烧只把掉落改写成 `ash`，而**背包／容器里腐坏**走 `IsInLimbo()` 分支直接 `Remove`，不掉翅膀也不播死亡。被玩家拿着时倍率 0.25，等效 2 天。[计时倍率](../../../raw/current/scripts/prefabs/moonbutterfly.lua#L57-L60)、[饿死回调](../../../raw/current/scripts/prefabs/moonbutterfly.lua#L84-L97)
- 修改边界：月蛾的"来源"只写在月亮药水与本批月树掉落两处，药水本体与蝴蝶侧接线归 [[reference/prefabs/butterfly]]（S6）；翅膀数值归 [[reference/prefabs/butterfly-wings]]；种下的月树苗本体与生长链归 [[reference/prefabs/moontree]]。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `moonbutterfly` | 无包装工厂：`fn` 直构 → 物理／动画／Light／音效／网络 → build／bank／bloom／光照 → 12 个标签 → 闪烁任务 → `MakeInventoryFloatable` → `SetPristine` → 非主模拟返回 → 主模拟挂 locomotor／stackable／inventoryitem／health／combat／knownlocations／可燃可冻／inspectable／lootdropper／workable／deployable／hauntable／eater／perishable → `SGbutterfly`＋brain | 活体＋可拾取物；`SetDeployMode(DEPLOYMODE.PLANT)`、间距 `PLACER_DEFAULT`；生命 1、移速 `MOONBUTTERFLY_SPEED`＝2.5。[主体](../../../raw/current/scripts/prefabs/moonbutterfly.lua#L157-L216)、[移速](../../../raw/current/scripts/tuning.lua#L1799-L1799) |
| `moonbutterfly_placer` | `MakePlacer("moonbutterfly_placer", "baby_moon_tree", "baby_moon_tree", "idle")` | 只是部署预览：bank／build／动画全取月树苗 `baby_moon_tree`，所以放置时看到的是树苗而不是月蛾。部署器按 `<prefab>_placer` 命名取用。[注册](../../../raw/current/scripts/prefabs/moonbutterfly.lua#L222-L222)、[取用规则](../../../raw/current/scripts/components/inventoryitem_replica.lua#L360-L369) |

标签（两端，pristine 之前加）：`butterfly`、`flying`、`ignorewalkableplatformdrowning`、`insect`、`smallcreature`、`cattoyairborne`、`wildfireprotected`、`show_spoilage`、`small_livestock`、`deployedplant`、`lunar_aligned`、`plantcreature`——共 **12 个**，其中 11 个与蝴蝶相同，月蛾独有的只有 `lunar_aligned`。[标签](../../../raw/current/scripts/prefabs/moonbutterfly.lua#L133-L144)、[蝴蝶标签](../../../raw/current/scripts/prefabs/butterfly.lua#L87-L112)

### 名称核验

简中实显是**月蛾**（英文 Moon Moth），`STRINGS.NAMES.MOONBUTTERFLY` 一个键。`moonbutterfly_placer` 只是部署预览：它没有可检视名字，也不参与检视台词按 prefab 取键那条链（`NAMES.MOONBUTTERFLY_PLACER` 在 `chinese_s.po` 里 0 命中，检索范围见验证记录）。[MOONBUTTERFLY](../../../raw/current/scripts/languages/chinese_s.po#L345610-L345612)、[placer 注册](../../../raw/current/scripts/prefabs/moonbutterfly.lua#L221-L222)

## 光照与外观

| 项目 | 值 | 证据 |
|---|---|---|
| 光源 | 半径 0.5、强度 0.5、衰减 0.8、颜色 `(0.3, 0.55, 0.45)`，构造即开，并允许客户端按时间调制 | [光照配置](../../../raw/current/scripts/prefabs/moonbutterfly.lua#L126-L131)、[常量](../../../raw/current/scripts/prefabs/moonbutterfly.lua#L24-L26) |
| 闪烁 | 每 0.1 秒一次 `OnUpdateFlicker`：正弦叠正弦映射到 0–1，再把强度写成 `0.5 + 0.05 × flicker` | [任务](../../../raw/current/scripts/prefabs/moonbutterfly.lua#L146-L147)、[实现](../../../raw/current/scripts/prefabs/moonbutterfly.lua#L28-L33) |
| 外观 | bank `butterfly`、build `butterfly_moon`，播 `idle`；`SetRayTestOnBB(true)`、`SetLightOverride(0.15)`、bloom 走 `shaders/anim.ksh` | [动画](../../../raw/current/scripts/prefabs/moonbutterfly.lua#L119-L124) |
| 物理 | `MakeTinyFlyingCharacterPhysics(inst, 1, .5)`，`SetTwoFaced` | [物理](../../../raw/current/scripts/prefabs/moonbutterfly.lua#L114-L117) |

只在持有／放出时改光：拿起来关灯、丢下开灯；饿死与死亡也会关灯并清掉 `SetLightOverride`。[拾取](../../../raw/current/scripts/prefabs/moonbutterfly.lua#L57-L60)、[丢下](../../../raw/current/scripts/prefabs/moonbutterfly.lua#L35-L40)、[死亡](../../../raw/current/scripts/prefabs/moonbutterfly.lua#L99-L102)

## 捕捉与携带

- **抓取**：`workable` 动作 `ACTIONS.NET`、工作量 1；完成时把本体直接塞进抓取者背包并播 `dontstarve/common/butterfly_trap`。道具侧的网与耐久属捕虫网，见 [[reference/prefabs/butterfly]] 的同名链路。[workable](../../../raw/current/scripts/prefabs/moonbutterfly.lua#L190-L193)、[完成回调](../../../raw/current/scripts/prefabs/moonbutterfly.lua#L62-L67)
- **不能直接捡**：`canbepickedup = false` 但 `canbepickedupalive = true`，即只能活捉；`nobounce`、`pushlandedevents = false`。所以"走过去按拾取"对它无效。[inventoryitem](../../../raw/current/scripts/prefabs/moonbutterfly.lua#L164-L170)
- **叠 20**：`stackable` 没写 `maxsize`，落组件默认 `STACK_SIZE_MEDITEM`＝20（采集册同项也记 20）。[组件默认](../../../raw/current/scripts/components/stackable.lua#L23-L28)、[采集册](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L947-L947)
- **丢下会拆堆**：`OnDropped` 先把倍率恢复为 1、记住 `home`、开灯、把 `workable` 重置为 1，然后 `while StackSize() > 1` 逐只 `Get()`＋传送到同一坐标＋调各自的 `OnDropped`——所以丢一叠 20 只等于在地上放 20 只独立月蛾，每只单独计时。[拆堆](../../../raw/current/scripts/prefabs/moonbutterfly.lua#L35-L55)

## 数值与计时

时间基准：`seg_time`＝30 秒、`total_day_time`＝480 秒、`perish_warp`＝1（S1）。[基准](../../../raw/current/scripts/tuning.lua#L19-L20)、[`perish_warp`](../../../raw/current/scripts/tuning.lua#L47-L47)

| 项目 | 值 | 推导与证据 |
|---|---|---|
| 生命 | 1 | [赋值](../../../raw/current/scripts/prefabs/moonbutterfly.lua#L172-L173) |
| 鲜度总时长 | `MOONBUTTERFLY_PERISH_TIME`＝`total_day_time / 2`＝**240 秒（0.5 天）** | [常量](../../../raw/current/scripts/tuning.lua#L1800-L1800)、[赋值](../../../raw/current/scripts/prefabs/moonbutterfly.lua#L207-L210) |
| 计时开关 | **构造即 `StartPerishing`**（不随持有状态启停），与蝴蝶的 `MakeFeedableSmallLivestock`（进容器才计时）相反 | [赋值](../../../raw/current/scripts/prefabs/moonbutterfly.lua#L207-L210)、[蝴蝶的计时](../../../raw/current/scripts/standardcomponents.lua#L972-L994) |
| 持有 | `OnPickedUp` 写 `SetLocalMultiplier(MOONBUTTERFLY_PERISH_INV_MODIFIER)`＝0.25 → 等效 240／0.25＝**960 秒（2 天）** | [拾取](../../../raw/current/scripts/prefabs/moonbutterfly.lua#L57-L60)、[常量](../../../raw/current/scripts/tuning.lua#L1801-L1801)、[局部倍率参与结算](../../../raw/current/scripts/components/perishable.lua#L127-L127) |
| 野生／落地 | 丢下恢复倍率 1，且 owner 为空时再乘 `PERISH_GROUND_MULT`＝1.5 → 240／1.5＝**160 秒** | [丢下](../../../raw/current/scripts/prefabs/moonbutterfly.lua#L35-L37)、[地面倍率](../../../raw/current/scripts/components/perishable.lua#L97-L105)、[常量](../../../raw/current/scripts/tuning.lua#L2083-L2083) |
| 喂食 | 食性 `VEGGIE`；`eater:SetOnEatFn(oneat)` 把鲜度写回 100%（不是加时间） | [食性](../../../raw/current/scripts/prefabs/moonbutterfly.lua#L203-L205)、[oneat](../../../raw/current/scripts/prefabs/moonbutterfly.lua#L78-L82) |
| 饿死 | 见下方三步 | [onperish](../../../raw/current/scripts/prefabs/moonbutterfly.lua#L84-L97) |

饿死（`onperish`）的三步细节：

1. 若处在 limbo（不在场景里，背包／容器内的腐坏就是这条）直接 `Remove`：不出掉落、不播死亡、不掉翅膀。
2. 否则先关掉 `workable`（不能再被网）与灯光、清 `SetLightOverride`，推 `death` 让 `SGbutterfly` 播死亡并 `DropLoot`。
3. 摘 `spore` 标签（月蛾从未加过该标签，这一行是从孢子类实体照抄的空操作）、`persists = false`、`DoTaskInTime(3, inst.Remove)`——3 秒后自删，不留尸体。[三步](../../../raw/current/scripts/prefabs/moonbutterfly.lua#L84-L97)

掉落表：共享表 `moonbutterfly` 只有一条 `{"moonbutterflywings", 1.0}`，`chance >= 1.0` 直接给，所以**场景内的非燃烧死亡固定 1 张翅膀**（击杀与饿死同一条）；燃烧中的死亡会被 `lootdropper` 的改写链换成 `ash`，而 limbo 里腐坏根本不结算掉落。翅膀的三维与用途见 [[reference/prefabs/butterfly-wings]]。[共享表](../../../raw/current/scripts/prefabs/moonbutterfly.lua#L17-L20)、[必给分支](../../../raw/current/scripts/components/lootdropper.lua#L213-L227)、[死亡状态](../../../raw/current/scripts/stategraphs/SGbutterfly.lua#L50-L65)

## 放生与种植

- **入口**：`deployable` 的 `OnDeploy`——生成 `moonbutterfly_sapling`、放到落点、播 `dontstarve/wilson/plant_tree`、`stackable:Get():Remove()` 消耗 1 只。[OnDeploy](../../../raw/current/scripts/prefabs/moonbutterfly.lua#L69-L76)
- **门禁**：`DEPLOYMODE.PLANT` 走 `Map:CanDeployPlantAtPoint`，不是随便地面都能种；间距 `DEPLOYSPACING.PLACER_DEFAULT`。[模式](../../../raw/current/scripts/prefabs/moonbutterfly.lua#L196-L199)、[判定](../../../raw/current/scripts/components/deployable.lua#L104-L115)
- **产物**：`moonbutterfly_sapling` 由 `planted_tree.lua` 的 `sapling_fn` 注册（build `baby_moon_tree`、`growprefab = "moon_tree_short"`、DIG 掉 `twigs`）；本页只写接入点，公共 sapling 工厂与月树苗生长归 [[reference/prefabs/moontree]]。[注册](../../../raw/current/scripts/prefabs/planted_tree.lua#L188-L188)、[工厂](../../../raw/current/scripts/prefabs/planted_tree.lua#L106-L180)

## 来源

| 来源 | 成立条件 | 与月蛾的关系 | 证据 |
|---|---|---|---|
| 月亮药水 `halloweenpotion_moon` | 目标带 `halloweenmoonmutable`；蝴蝶的该组件被 `SetPrefabMutated("moonbutterfly")` 指到月蛾，`push_attacked_on_new_inst = false`，新体不会立刻被"打"惊飞 | **蝴蝶 → 月蛾**：不是就地改写而是替换实体；机制与门禁归 [[reference/prefabs/butterfly]] | [蝴蝶侧接线](../../../raw/current/scripts/prefabs/butterfly.lua#L192-L195) |
| 月树砍伐 | 月树的**中／高**两张 loot 表带 `moonbutterfly`（正常树必出 1 只、高树 1 只＋50% 再 1 只；矮树表没有），掉的是**活体**不是物品 | 月树 → 月蛾（月树侧见 [[reference/prefabs/moontree]]） | [普通表](../../../raw/current/scripts/prefabs/moontree.lua#L50-L58)、[高表](../../../raw/current/scripts/prefabs/moontree.lua#L60-L70) |
| 月亮药水打花 | 花也带 `halloweenmoonmutable`，但变异目标是 `moonbutterfly_sapling` 而不是月蛾 | 花 → 月树苗（与月蛾无关，提示别混） | [花侧接线](../../../raw/current/scripts/prefabs/flower.lua#L190-L191) |
| 温蒂技能配方 `wendy_moonbutterfly` | `ghostflower` 4＋`moonbutterflywings` 1，`TECH.NONE`、`builder_skill = "wendy_ghostflower_butterfly"`，产物 `moonbutterfly` | 消耗翅膀换活体 | [配方](../../../raw/current/scripts/recipes.lua#L165-L165) |

反向去向（月蛾被当材料消耗）：`critter_lunarmothling_builder`（月蛾 1＋`flowersalad` 1）与 `carpentry_blade_moonglass`（月光玻璃 6＋月蛾 1）。[幼虫配方](../../../raw/current/scripts/recipes.lua#L826-L826)、[月玻璃工具](../../../raw/current/scripts/recipes.lua#L841-L841)

## 与蝴蝶的差异

| 维度 | `butterfly` | `moonbutterfly` |
|---|---|---|
| 光照 | 无 Light，有动态阴影 | 有 Light（半径 0.5）＋闪烁，**没有** `AddDynamicShadow` |
| 采粉 | 有 `pollinator` 组件与标签，满 6 朵回巢 | 没有 pollinator 组件，brain 里也没有采粉行为，只有 RunAway＋Wander |
| 标签数量与差异 | 13 个：11 个显式（含 `pollinator`、`noember`）＋`show_spoilage`＋`small_livestock`（后两个由 `MakeFeedableSmallLivestockPristine` 补） | 12 个；公共 11 个；月蛾独有 `lunar_aligned`；蝴蝶独有 `noember`、`pollinator`；`wildfireprotected` 两者都有 |
| 掉落 | `AddRandomLoot`：蝶翅权重 5／黄油 0.1，抽 1 件 | 共享表：月蛾翅膀 1.0，**场景内**非燃烧死亡必给 1 张；背包／容器内腐坏不掉 |
| 腐坏 | 走 `MakeFeedableSmallLivestock` 传 `BUTTERFLY_PERISH_TIME`（3 天），进容器才计时 | 构造即计时 0.5 天，持有 0.25 倍 |
| 死亡处理 | 死亡只掉表 | 场景内饿死额外关网、关灯、3 秒自删；limbo 里腐坏直接移除 |
| 交易 | 有 `tradable` | 没有 `tradable` |
| 部署间距 | `DEPLOYSPACING.LESS` | `DEPLOYSPACING.PLACER_DEFAULT` |
| 月亮可变 | 有 `halloweenmoonmutable`（→月蛾） | 没有该组件，月蛾不再被药水改写 |
| 刷新器 | 受 `butterflyspawner` 追踪与预算约束 | 没有任何 spawner（来源只有药水／月树／配方），但因带 `butterfly` 标签仍**占蝴蝶计数预算**：刷新器每次补刷前按 64 格内 `butterfly` 标签实体数比对 `MAX_BUTTERFLIES`（本版 4），月蛾会被数进去而挤掉普通蝴蝶的刷新额度 |

依据：蝴蝶的同名项见 [[reference/prefabs/butterfly]]；月蛾侧逐项见[注册与构造](#注册与构造)、[光照与外观](#光照与外观)、[数值与计时](#数值与计时)。[蝴蝶标签](../../../raw/current/scripts/prefabs/butterfly.lua#L87-L112)、[蝴蝶掉落](../../../raw/current/scripts/prefabs/butterfly.lua#L158-L162)、[蝴蝶计时](../../../raw/current/scripts/prefabs/butterfly.lua#L190-L190)、[月蛾 brain](../../../raw/current/scripts/brains/moonbutterflybrain.lua#L31-L42)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| SG：`SGbutterfly` | 复用蝴蝶状态机：`moving`／`idle`／`land`／`pollinate`／`takeoff`／`death`；`sg.mem.burn_on_electrocute = true` 让电击改为点燃 | 主模拟；`death` 里 `DropLoot` | [安装](../../../raw/current/scripts/prefabs/moonbutterfly.lua#L212-L213)、[死亡状态](../../../raw/current/scripts/stategraphs/SGbutterfly.lua#L50-L65) |
| Brain：`moonbutterflybrain` | PriorityNode：恐慌→电栏恐慌→对 `scarytoprey` 逃跑（沃姆伍德的 `wormwood_bugs` 技能可豁免）→以 `home` 为中心游荡 20 格 | 主模拟；初始化时把出生点记为 home | [行为树](../../../raw/current/scripts/brains/moonbutterflybrain.lua#L31-L42)、[记家](../../../raw/current/scripts/brains/moonbutterflybrain.lua#L44-L46) |
| 组件：perishable | 构造即计时；持有 0.25、落地 1.5；饿死走自定义回调而不是替换品 | 主模拟 | [[reference/components/perishable]]；[赋值](../../../raw/current/scripts/prefabs/moonbutterfly.lua#L207-L210) |
| 组件：stackable | 未写 `maxsize`，默认 20；丢下时逐只拆开 | 主模拟 | [[reference/components/stackable]]；[拆堆](../../../raw/current/scripts/prefabs/moonbutterfly.lua#L46-L54) |
| 组件：workable | `ACTIONS.NET`、工作量 1；丢下时重置为 1 | 主模拟 | [[reference/components/workable]]；[配置](../../../raw/current/scripts/prefabs/moonbutterfly.lua#L190-L193) |
| 组件：deployable | `DEPLOYMODE.PLANT`＋`OnDeploy`；产物 `moonbutterfly_sapling` | 主模拟；客户端凭 replica 显示可部署 | [[reference/components/deployable]]；[配置](../../../raw/current/scripts/prefabs/moonbutterfly.lua#L196-L199) |
| 组件：lootdropper | `SetChanceLootTable` 指共享表 `moonbutterfly`：翅膀 1.0 | 主模拟 | [[reference/components/lootdropper]]；[配置](../../../raw/current/scripts/prefabs/moonbutterfly.lua#L187-L188) |
| 组件：eater | 食性 `{FOODTYPE.VEGGIE}`；被喂食把鲜度写回 100% | 主模拟；FEED 动作对该实体的可达性未核验 | [[reference/components/eater]]；[食性](../../../raw/current/scripts/prefabs/moonbutterfly.lua#L203-L205) |
| 组件：knownlocations | `home` 用于 brain 的游荡中心；丢下时刷新 | 主模拟 | [[reference/components/knownlocations]]；[刷新](../../../raw/current/scripts/prefabs/moonbutterfly.lua#L37-L37) |
| 标签：`cattoyairborne`／`lunar_aligned`／`deployedplant` | 浣猫把空中月蛾当玩具；月亮阵营判定；`deployedplant` 让部署事件额外推 `itemplanted` | 两端 | [标签](../../../raw/current/scripts/prefabs/moonbutterfly.lua#L133-L144)、[部署事件](../../../raw/current/scripts/components/deployable.lua#L132-L147) |
| 关联 Prefab：`moonbutterfly_sapling` | 放生产物；sapling 公共工厂留 B14，[[reference/prefabs/moontree]] 只写接入点与生长链 | 只在 `OnDeploy` 生成 | [生成](../../../raw/current/scripts/prefabs/moonbutterfly.lua#L69-L76) |
| 关联 Prefab：`moonbutterflywings` | 唯一掉落物，1.0 必给（场景内非燃烧死亡） | 死亡／饿死结算；燃烧死亡改写成 `ash`，limbo 腐坏不结算 | [[reference/prefabs/butterfly-wings]]；[共享表](../../../raw/current/scripts/prefabs/moonbutterfly.lua#L17-L20)、[燃烧改写](../../../raw/current/scripts/components/lootdropper.lua#L415-L445) |
| 资源：`anim/butterfly_basic.zip`／`anim/butterfly_moon.zip`／`anim/baby_moon_tree.zip` | bank `butterfly`＋build `butterfly_moon`；placer 用 `baby_moon_tree`；另有 `INV_IMAGE moonbutterfly` | 原版资源，不随页分发 | [assets](../../../raw/current/scripts/prefabs/moonbutterfly.lua#L1-L9) |

### 共享规则接入（S1／S3／S6／S7／S8／S9／S10 适用；S2／S4／S5 不适用）

- **S1 时间基准**：0.5 天＝240 秒、2 天＝960 秒、160 秒都按 `total_day_time`＝480 秒推导（[基准](../../../raw/current/scripts/tuning.lua#L19-L20)）。
- **S3 标签分层**：两个注册分别给了构造与 placer 的职责；月蛾的 12 个标签按"逐注册列、不用统称"写在本页[注册与构造](#注册与构造)，与蝴蝶的 11 个标签差异单列。
- **S6 月亮转换**：蝴蝶被月亮药水替换成月蛾走 `halloweenmoonmutable` 的实体替换，不是就地改写；月蛾自己没有该组件，不能再变。[蝴蝶侧](../../../raw/current/scripts/prefabs/butterfly.lua#L192-L195)
- **S7 名称**：月蛾＝"月蛾"；placer 无 `NAMES` 键。另需与"月树苗"区分：`moonbutterfly_sapling` 的简中名才是"月树苗"，而挖出的 `dug_sapling_moon` 显示"树苗"。[月树苗](../../../raw/current/scripts/languages/chinese_s.po#L345620-L345622)、[挖出件](../../../raw/current/scripts/languages/chinese_s.po#L342350-L342352)
- **S8 跨页归属**：翅膀数值与用途归 [[reference/prefabs/butterfly-wings]]；药水与蝴蝶本体归 [[reference/prefabs/butterfly]]；`moonbutterfly_sapling` 的公共 sapling 工厂留 B14，本批只在 [[reference/prefabs/moontree]] 写月树苗的接入点与生长链。
- **S9 鲜度口径**：`SetLocalMultiplier` 改的是腐坏速率而不是剩余时间，所以"2 天"是等效时长；把它当成 `SetPerishTime` 会算错。[倍率](../../../raw/current/scripts/components/perishable.lua#L196-L202)
- **S10 待核验**：FEED 对月蛾的实际可达性、`tree_petal_fx_chop` 的视觉效果、月树掉落概率的实测分布只按源码推导。
- **S2／S4／S5 不适用**：不涉及钓鱼链、出水投射与重量归属；月蛾没有 `weighable`／`oceanfishable`。

## 典型接入

### 让自定义物品也能"放生种月树苗"

- 前提与选择：服务端 `modmain.lua`。要的是"消耗 1 个物品、在原地生成 `moonbutterfly_sapling`"这一行为时，复制 `OnDeploy` 的三句即可，不要自己写 `SpawnPrefab` 后再判断间距——间距由 `deployable` 的 `DEPLOYMODE.PLANT` 门禁保证。
- 生效与撤销：部署成功才消耗；`CanDeploy` 失败时 `OnDeploy` 根本不会被调用，物品保留。卸模组后已种下的树苗照常生长。
- 原版依据：[OnDeploy](../../../raw/current/scripts/prefabs/moonbutterfly.lua#L69-L76)、[部署门禁](../../../raw/current/scripts/components/deployable.lua#L104-L115)。

```lua
-- modmain.lua：给自定义"月蛾种子"加放生行为
AddPrefabPostInit("my_moon_seed", function(inst)
    -- 只影响放置预览：没有 placer 的自定义物品会借用月蛾的预览模型
    inst.overridedeployplacername = "moonbutterfly_placer"

    if not GLOBAL.TheWorld.ismastersim then
        return
    end

    local deployable = inst:AddComponent("deployable")
    deployable:SetDeployMode(GLOBAL.DEPLOYMODE.PLANT)
    deployable:SetDeploySpacing(GLOBAL.DEPLOYSPACING.PLACER_DEFAULT)
    deployable.ondeploy = function(item, pt, deployer)
        local sapling = GLOBAL.SpawnPrefab("moonbutterfly_sapling")
        if sapling == nil then
            return
        end
        sapling.Transform:SetPosition(pt:Get())
        sapling.SoundEmitter:PlaySound("dontstarve/wilson/plant_tree")
        if item.components.stackable ~= nil then
            item.components.stackable:Get():Remove()
        else
            item:Remove()
        end
    end
end)
```

- 调整与验证：`DEPLOYMODE`／`DEPLOYSPACING` 是 `constants.lua` 的全局表，modmain 里要经 `GLOBAL` 取；`overridedeployplacername` 只改放置预览（客户端按 `inst.overridedeployplacername or prefab.."_placer"` 取 placer），不影响产物与间距。实测看种下的树苗是否在 `PINECONE_GROWTIME` 后变成 `moon_tree_short`（见 [[reference/prefabs/moontree]]）。示例只做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **误区：月蛾和蝴蝶一样"关在包里才腐烂"。** 它的 perishable 构造即启动，背包只把速率降到 0.25；放地上则恢复 1 并再乘 1.5，所以野生月蛾约 160 秒就死。[赋值](../../../raw/current/scripts/prefabs/moonbutterfly.lua#L207-L210)、[地面倍率](../../../raw/current/scripts/components/perishable.lua#L97-L105)
- **误区：月蛾饿死就掉翅膀。** 它没有 `onperishreplacement`：只有**场景内、非燃烧**死亡走 `death`＋`DropLoot` 掉 1 张 `moonbutterflywings`；燃烧死亡被改写成 `ash`；**在背包／容器里**腐坏则走 `IsInLimbo()` 分支直接 `Remove`，什么也不掉。[饿死回调](../../../raw/current/scripts/prefabs/moonbutterfly.lua#L84-L97)、[共享表](../../../raw/current/scripts/prefabs/moonbutterfly.lua#L17-L20)
- **边界：丢一叠会散成一地。** `OnDropped` 把堆叠逐只拆开并传送到同一坐标，每只都重新计时；想省格子就留在背包（0.25 倍）。[拆堆](../../../raw/current/scripts/prefabs/moonbutterfly.lua#L46-L54)
- **边界：直接拾取无效。** `canbepickedup = false`＋`canbepickedupalive = true`，只能网；网完再丢下会把 `workable` 重置成 1，所以可以反复抓放。[inventoryitem](../../../raw/current/scripts/prefabs/moonbutterfly.lua#L164-L170)、[重置](../../../raw/current/scripts/prefabs/moonbutterfly.lua#L42-L44)
- **边界：场景内的饿死不是"立刻消失"。** 只要不在 limbo，`onperish` 就 `persists = false` 并排一个 3 秒的 `DoTaskInTime` 才 `Remove`——**与是否燃烧无关**；这 3 秒里它仍占位，但网已被 `SetWorkable(false)` 关掉。燃烧只把掉落改写成 `ash`，定时器照跑；limbo 里的腐坏是立即 `Remove`，根本不进这条分支。[收尾](../../../raw/current/scripts/prefabs/moonbutterfly.lua#L88-L96)
- **边界：它不会再变。** 月蛾没有 `halloweenmoonmutable`，月亮药水对它无效；"花变月树苗"是花的组件，别当成月蛾的第二种形态。[花侧](../../../raw/current/scripts/prefabs/flower.lua#L190-L191)
- **未核验**：FEED 动作对月蛾是否可达、`tree_petal_fx_chop` 的实际外观、`wendy_moonbutterfly` 产出的月蛾是否带 `home`（由 brain 初始化写入，未运行验证）。

## 验证记录

- 源码审阅：2026-11-09，Steam buildid 26474873（`raw/current/BUILD.txt`）。通读 `prefabs/moonbutterfly.lua`、`brains/moonbutterflybrain.lua`、`stategraphs/SGbutterfly.lua`；对照 `prefabs/butterfly.lua` 的标签／掉落／计时／部署／月亮可变；核对 `prefabs/planted_tree.lua` 的 `sapling_fn` 与 `moonbutterfly_sapling` 注册、`prefabs/moontree.lua` 三张 loot 表的 `moonbutterfly` 条目、`prefabs/flower.lua` 的变异目标、`components/stackable.lua` 默认值、`components/perishable.lua` 的三类倍率与替换链、`components/deployable.lua` 的 PLANT 分支、`components/inventoryitem_replica.lua` 的 placer 命名、`recipes.lua` 的三条相关配方、`tuning.lua` 的常量与 `screens/redux/scrapbookdata.lua` 的采集册条目。
- 名称核验：`languages/chinese_s.po` 的 `NAMES.MOONBUTTERFLY`＝月蛾、`NAMES.MOONBUTTERFLY_SAPLING`＝月树苗。检索范围：在该文件搜 `MOONBUTTERFLY_PLACER` 0 命中（`msgctxt` 与正文都算），所以 placer 没有可检视名字。
- 标签计数：逐条比对 `prefabs/butterfly.lua` L87-L99＋L112（`MakeFeedableSmallLivestockPristine` 补 `show_spoilage`／`small_livestock`）与 `prefabs/moonbutterfly.lua` L133-L144，得蝴蝶 13、月蛾 12、公共 11。
- 刷新器预算：`components/butterflyspawner.lua` L47-L50 用 `FindEntities(pt, 64, {"butterfly"})` 与 `TUNING.MAX_BUTTERFLIES`（`tuning.lua` L6017＝4）比较。
- 未覆盖：捕虫网本体（耐久、动作门禁）、`SGbutterfly` 里 `land`／`pollinate` 对月蛾的可达性、`butterflyspawner` 侧行为、`critter_lunarmothling` 的养成链、月亮药水本体。
- R2 独立复审修订（2026-11-09）：标签计数改为蝴蝶 13／月蛾 12／公共 11（P2-05）；补 `butterflyspawner` 的 64 格计数预算（P2-06）；掉落补"非燃烧死亡"条件；`spore` 标注为照抄的空操作；示例补 `overridedeployplacername` 预览说明（P3）。
- 外部复检修订（2026-11-09）：限定掉落口径——只有场景内饿死／击杀才掉翅膀，背包／容器内腐坏走 `IsInLimbo()` 直接移除（P1-5）；同步修正 summary、重要副作用、三步细节、差异表与导航行。
- 二次复检修订（2026-11-09）：summary、重要副作用与两处陷阱统一为"场景内、非燃烧死亡才掉翅膀并 3 秒自删；燃烧死亡改 `ash`；limbo 直接移除"（P2-4）。
- 三次定向复检修订（2026-11-09）：把 3 秒清理与燃烧解绑——`onperish` 只要不在 limbo 就排 `DoTaskInTime(3, Remove)`，与燃烧无关；燃烧只经 `lootdropper` 把掉落改写成 `ash`。summary／快速判断／陷阱同步（P2-2）。
- 游戏运行：未运行；160／240／960 秒为源码推导（`total_day_time`＝480）。
- 结构检查：`validate_page.py` 通过（standard 档内）；`validate_source_links.py --strict` 0 错 0 警。

## 相关链接

- 本体与药水入口：[[reference/prefabs/butterfly]]
- 翅膀数值：[[reference/prefabs/butterfly-wings]]
- 种下的月树苗与月树：[[reference/prefabs/moontree]]
- 组件：[[reference/components/perishable]]、[[reference/components/deployable]]、[[reference/components/workable]]

返回 [[reference/prefabs/index|Prefab 索引]]。
