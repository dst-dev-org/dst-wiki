---
title: "牛毛（beefalowool）"
created: "2026-11-10"
updated: "2026-11-10"
type: prefab
summary: "单注册材料牛毛：皮弗娄牛刷毛与剃毛的主产出、成牛与幼牛死亡掉落、伯尼腐坏与风滚草的旁支来源，40 叠、45 秒燃料，并作为伯尼、保暖装备、地毯、枕头、鱼饵、鞍与炼金转化的原料。"
aliases: ["beefalowool", "牛毛", "刷牛毛", "剃牛毛", "皮弗娄牛毛"]
tags: [prefab, item, loot, recipe]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/beefalowool.lua", "prefabs/beefalo.lua", "prefabs/babybeefalo.lua", "prefabs/bernie_inactive.lua", "prefabs/tumbleweed.lua", "components/brushable.lua", "components/beard.lua", "components/lootdropper.lua", "components/tradable.lua", "prefabs/pillow_defs.lua", "stategraphs/SGBeefalo.lua", "standardcomponents.lua", "recipes.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# 牛毛（beefalowool）

## 快速判断

- 职责与适用：`prefabs/beefalowool.lua` 单注册的通用材料“牛毛”（S1）。主来源是皮弗娄牛的刷毛、剃毛与死亡掉落，旁支有幼牛、伯尼腐坏与风滚草；消费者横跨伯尼、保暖装备、地毯、枕头、鱼饵、鞍与炼金转化。[注册](../../../raw/current/scripts/prefabs/beefalowool.lua#L6-L50)
- 优先使用：查“怎么稳定拿牛毛”看[获取](#获取)；查堆叠／燃料／交易属性看[注册与构造](#注册与构造)；查具体配方看[配方与消费者](#配方与消费者)；刷毛与剃毛的完整时序归 [[reference/prefabs/beefalo]]。
- 重要副作用：刷毛按 `brushable` 的 `prize`／`max` 结算，剃毛一次给 3 份并把牛剃秃；成牛在非保尸、非 `file_load` 路径死亡保底 3 份，幼牛按成长阶段 1／2／2 份；`DropLoot` 结算时仍在燃烧则牛毛没有熟制变体、转成灰，保尸分支不执行掉落。[刷毛](../../../raw/current/scripts/prefabs/beefalo.lua#L1145-L1149)、[成牛表](../../../raw/current/scripts/prefabs/beefalo.lua#L49-L59)、[幼牛表](../../../raw/current/scripts/prefabs/babybeefalo.lua#L18-L20)
- 修改边界：牛毛本身是物品页；牛群、驯化、骑乘、牛铃与鞍只给入口，分别归 [[reference/components/herd]]、[[reference/components/domesticatable]]、[[reference/components/rideable]] 与对应源码。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 属性与证据 |
|---|---|---|
| `beefalowool` | `fn()`：实体→Transform／AnimState／Network→`MakeInventoryPhysics`→bank／build `beefalo_wool`、播 `idle`→`pickupsound="cloth"`→`AddTag("cattoy")`→`MakeInventoryFloatable("med", nil, 0.67)`→`SetPristine`；主模拟挂 `inspectable`／`inventoryitem`／`stackable`／`tradable`／`fuel`／`MakeSmallBurnable`／`MakeSmallPropagator`／`MakeHauntableLaunchAndIgnite`。[工厂](../../../raw/current/scripts/prefabs/beefalowool.lua#L6-L48) | 单注册材料；40 叠；`fuel.fuelvalue=TUNING.MED_FUEL`＝45 秒；`MakeSmallBurnable(inst, TUNING.MED_BURNTIME)`＝15 秒；`tradable` 已挂但未赋 `goldvalue`，默认 0；无 `edible`、无 `perishable`。[堆叠](../../../raw/current/scripts/prefabs/beefalowool.lua#L34-L35)、[燃料](../../../raw/current/scripts/prefabs/beefalowool.lua#L39-L40)、[交易](../../../raw/current/scripts/prefabs/beefalowool.lua#L37-L37)、[默认金值](../../../raw/current/scripts/components/tradable.lua#L1-L7) |

- 40 叠来自 [`STACK_SIZE_SMALLITEM`](../../../raw/current/scripts/tuning.lua#L81-L84)；45 秒燃料来自 [`MED_FUEL`](../../../raw/current/scripts/tuning.lua#L1673-L1678)；15 秒可燃来自 [`MED_BURNTIME`](../../../raw/current/scripts/tuning.lua#L1680-L1683)。
- `cattoy` 是猫玩具标签；`MakeInventoryFloatable("med", nil, 0.67)` 只影响落水漂浮，不是保鲜。
- 名称核验：`STRINGS.NAMES.BEEFALOWOOL` 为 **牛毛**（S1）。[po](../../../raw/current/scripts/languages/chinese_s.po#L339284-L339287)

## 获取

| 来源 | 触发与数量 | 证据 |
|---|---|---|
| 成牛刷毛 `brushable` | `prize="beefalowool"`、`max=1`；`CalculateNumPrizes` 在 `elapsed<=0` 时返回 0，否则 `floor(elapsed/cyclesperprize)` 封顶 `max`。宿主没有写 `cyclesperprize`（默认 0），只写了不被组件读取的 `regrowthdays=1`；按 Lua 5.1 浮点语义，`elapsed>0` 时 `elapsed/0=inf`、`min(inf,1)=1`，因此同一世界日内 0、跨天恰 1 份（源码推导，未运行验证）。[配置](../../../raw/current/scripts/prefabs/beefalo.lua#L1145-L1149)、[公式](../../../raw/current/scripts/components/brushable.lua#L31-L34)、[默认值](../../../raw/current/scripts/components/brushable.lua#L5-L13) |
| 成牛剃毛 `beard` | `beard.prize="beefalowool"`、初始 `bits=3`；`Beard:Shave` 按 `oldbits−newbits` 生成奖品，剃秃一次给 3 份。需牛在睡眠，且牛铃主人限制见 beefalo 页。 | [奖品](../../../raw/current/scripts/prefabs/beefalo.lua#L1135-L1143)、[剃毛](../../../raw/current/scripts/components/beard.lua#L144-L155)、[睡眠门禁](../../../raw/current/scripts/prefabs/beefalo.lua#L445-L460) |
| 成牛非保尸／非读档死亡 | 共享概率表逐项独立结算：`beefalowool` 3 条 1.00 恒过，另有 4 条 `meat` 1.00 与 0.33 的 `horn`；因此牛毛保底 3 份。骑乘中死亡要等甩下骑手后才进 `death`／`DropDeathLoot`。 | [表](../../../raw/current/scripts/prefabs/beefalo.lua#L49-L59)、[接线](../../../raw/current/scripts/prefabs/beefalo.lua#L1169-L1170)、[结算](../../../raw/current/scripts/components/lootdropper.lua#L213-L227)、[骑乘死亡](../../../raw/current/scripts/prefabs/beefalo.lua#L688-L692) |
| 成牛燃烧死亡 | `DropLoot` 结算时仍在燃烧，才把每个战利品换成熟制同名前缀或 `ash`；牛毛没有 `beefalowool_cooked`／`cookedbeefalowool`，落 `ash`。 | [转换](../../../raw/current/scripts/components/lootdropper.lua#L415-L442) |
| 幼牛 `babybeefalo` | 成长阶段各自 `SetLoot`：baby 3 小肉＋1 牛毛、toddler 4 小肉＋2 牛毛、teen 3 肉＋2 牛毛；每次转阶段替换旧表，不是追加。`DropLoot` 结算时仍在燃烧则其中的牛毛同样转灰。 | [表](../../../raw/current/scripts/prefabs/babybeefalo.lua#L18-L20)、[阶段赋值](../../../raw/current/scripts/prefabs/babybeefalo.lua#L70-L89) |
| 伯尼 `bernie_inactive` 空燃料腐坏 | 空燃料且未持有，落地 3 总日＝1440 秒后固定生成胡须、牛毛、蜘蛛丝各 1；拿起取消。 | [掉落](../../../raw/current/scripts/prefabs/bernie_inactive.lua#L28-L43)、[时长](../../../raw/current/scripts/tuning.lua#L3755-L3755) |
| 风滚草 | 开包按权重抽 3 件；`beefalowool` 权重 1（是权重不是 100%），同表还有切草 40、树枝 40 等。 | [权重表](../../../raw/current/scripts/prefabs/tumbleweed.lua#L171-L173)、[抽 3 件](../../../raw/current/scripts/prefabs/tumbleweed.lua#L218-L244) |

- **刷毛收益不是纯产出**：`OnBrushed` 只在 `numprizes>0` 时给驯化与服从；数量为 0 的同一动作仍会推 `brushed` 事件。[回调](../../../raw/current/scripts/prefabs/beefalo.lua#L482-L491)、[事件](../../../raw/current/scripts/components/brushable.lua#L52-L56)
- **剃毛与刷毛是两条资源路径**：剃毛把 `bits` 归 0、关掉 `brushable` 并让牛进入 `shaved` 外观；刷毛不改变胡须，只在有毛时按日结算。[剃毛复位](../../../raw/current/scripts/prefabs/beefalo.lua#L431-L443)、[刷毛配置](../../../raw/current/scripts/prefabs/beefalo.lua#L1145-L1149)

## 配方与消费者

| 消费族 | 配方（本物品数量） | 证据 |
|---|---|---|
| 伯尼与小痒痒 | `bernie_inactive` 牛毛 2＋胡须 2＋蜘蛛丝 2（Willow）；`mosquitomusk`（小痒痒）＝牛毛 1＋蚊子血囊 1（Wurt 技能） | [伯尼](../../../raw/current/scripts/recipes.lua#L94-L94)、[小痒痒配方](../../../raw/current/scripts/recipes.lua#L113-L113)、[po](../../../raw/current/scripts/languages/chinese_s.po#L345939-L345947) |
| 保暖与护甲 | `beefalohat` 8＋牛角 1；`winterhat` 4＋蜘蛛丝 4；`trunkvest_winter` 2；`antlionhat` 5；`wathgrithr_improvedhat` 2；`wathgrithr_shield` 3；`spear_wathgrithr_lightning` 1 | [牛角帽](../../../raw/current/scripts/recipes.lua#L534-L535)、[冬帽](../../../raw/current/scripts/recipes.lua#L535-L535)、[背心](../../../raw/current/scripts/recipes.lua#L549-L549)、[蚁狮帽](../../../raw/current/scripts/recipes.lua#L587-L587)、[女武神三件](../../../raw/current/scripts/recipes.lua#L192-L196) |
| 鞍与皮弗娄牛美妆台 | `saddle_basic` 4＋猪皮 4＋金块 4；`beefalo_groomer` 2＋木板 4 | [鞍](../../../raw/current/scripts/recipes.lua#L682-L682)、[美妆台](../../../raw/current/scripts/recipes.lua#L688-L688)、[po](../../../raw/current/scripts/languages/chinese_s.po#L339299-L339302) |
| 地皮与设施 | `turf_carpetfloor`／`turf_carpetfloor2` 各 1＋木板 1（一次 4 张）；`table_winters_feast` 1＋木板 1；`yotb_stage_item` 2＋木板 4＋金块 2；`mermthrone_construction` 15＋海带 20＋猪皮 10 | [地毯](../../../raw/current/scripts/recipes.lua#L695-L699)、[冬宴桌](../../../raw/current/scripts/recipes.lua#L1425-L1425)、[舞台](../../../raw/current/scripts/recipes.lua#L1154-L1154)、[鱼人王座](../../../raw/current/scripts/recipes.lua#L1523-L1523) |
| 皮弗娄牛毛枕头与枕头盔甲 | `handpillow_beefalowool`（皮弗娄牛毛枕头）3＋幸运金块 3＋蜘蛛丝 2；`bodypillow_beefalowool`（皮弗娄牛枕头盔甲）3＋幸运金块 4＋蜘蛛丝 2；材质参数 `strengthmult=1.8`、`defense_amount=0.5` | [皮弗娄牛毛枕头](../../../raw/current/scripts/recipes.lua#L1137-L1137)、[皮弗娄牛枕头盔甲](../../../raw/current/scripts/recipes.lua#L1142-L1142)、[材质](../../../raw/current/scripts/prefabs/pillow_defs.lua#L23-L32)、[po 毛枕](../../../raw/current/scripts/languages/chinese_s.po#L343714-L343717)、[po 枕头盔甲](../../../raw/current/scripts/languages/chinese_s.po#L339804-L339807)；[[reference/prefabs/role-tools]] |
| 鱼饵 | `oceanfishinglure_spinner_red/green/blue` 各 1；`oceanfishinglure_hermit_heavy` 1 | [三色亮片](../../../raw/current/scripts/recipes.lua#L657-L659)、[隐居者重饵](../../../raw/current/scripts/recipes.lua#L664-L664) |
| 交换与转化 | 流浪商人 `wanderingtradershop_pigskin`／`livinglog` 各 3；`transmute_beardhair` 2→胡须 1；`transmute_beefalowool` 2 胡须→牛毛 1 | [商店](../../../raw/current/scripts/recipes.lua#L1002-L1003)、[正反转化](../../../raw/current/scripts/recipes.lua#L1488-L1489) |
| 拆解 | `mermthrone` 拆回 15；`costume_king_body` 拆回 3 | [王座](../../../raw/current/scripts/recipes.lua#L1638-L1638)、[国王服](../../../raw/current/scripts/recipes.lua#L1728-L1728) |

## 关联机制导航

| 类型／对象 | 本页如何使用 | 条件、端与证据 |
|---|---|---|
| 组件：`brushable` | 成牛刷毛的 `prize`／`max`／`cyclesperprize` 结算；产出进刷毛者背包并推 `brushed` | 仅服务端；`SetBrushable(false)` 后动作标签消失 | [配置](../../../raw/current/scripts/prefabs/beefalo.lua#L1145-L1149)、[算法](../../../raw/current/scripts/components/brushable.lua#L31-L56)；[[reference/components/brushable]] |
| 组件：`beard` | 成牛剃毛的奖品与 bit 数；剃秃后驱动 `OnResetBeard` | 睡眠＋剃刀；每次 bit 一份 | [奖品](../../../raw/current/scripts/prefabs/beefalo.lua#L1135-L1143)、[Shave](../../../raw/current/scripts/components/beard.lua#L107-L166)；[[reference/components/beard]] |
| 组件：`lootdropper` | 成牛共享概率表、幼牛阶段固定表；`DropLoot` 结算时仍燃烧才统一改名／成灰 | 非保尸结算；表项独立判定 | [牛表](../../../raw/current/scripts/prefabs/beefalo.lua#L49-L59)、[燃烧转换](../../../raw/current/scripts/components/lootdropper.lua#L415-L442)；[[reference/components/lootdropper]] |
| 组件：`stackable`／`fuel`／`tradable` | 40 叠、45 秒燃料、默认金值 0 | 物品态 | [堆叠](../../../raw/current/scripts/prefabs/beefalowool.lua#L34-L35)、[燃料](../../../raw/current/scripts/prefabs/beefalowool.lua#L39-L40)、[交易](../../../raw/current/scripts/components/tradable.lua#L1-L7) |
| 关联 Prefab：`beefalo`／`babybeefalo` | 成牛与幼牛的产出宿主；幼牛成长后生成成牛 | 见“获取”表 | [[reference/prefabs/beefalo]]、[幼牛成长](../../../raw/current/scripts/prefabs/babybeefalo.lua#L91-L100) |
| 数据：`pillow_defs` | 牛毛材质的枕头参数：`strengthmult=1.8`、`defense_amount=0.5`、`hand/body_prize_value=0.75` | 枕头注册与技能门禁归 role-tools | [材质表](../../../raw/current/scripts/prefabs/pillow_defs.lua#L23-L32)；[[reference/prefabs/role-tools]] |

## 常见陷阱与限制

- **误区：`regrowthdays=1` 就是“一天一份”。** `brushable` 只读 `cyclesperprize`，宿主没设它（默认 0）；`regrowthdays` 在本版没有任何读取点。源码推导：`elapsed<=0`→0，`elapsed>0` 时 `elapsed/0=inf`、`min(inf,max)=1`，即同一世界日 0、跨天恰 1；未运行验证。[宿主字段](../../../raw/current/scripts/prefabs/beefalo.lua#L1146-L1147)、[实际读取](../../../raw/current/scripts/components/brushable.lua#L31-L34)
- **边界：刷毛与剃毛不能互相替代。** 刷毛一次最多 1 份且不剃毛；剃毛一次 3 份、把牛剃秃并关掉 `brushable`，3 天后长回才恢复可刷。[剃毛复位](../../../raw/current/scripts/prefabs/beefalo.lua#L431-L443)、[3 天常量](../../../raw/current/scripts/tuning.lua#L898-L899)、[长回](../../../raw/current/scripts/stategraphs/SGBeefalo.lua#L820-L838)
- **边界：`DropLoot` 结算时仍燃烧，牛毛才转灰。** 牛毛没有熟制变体，`_cooked` 与 `cooked` 两条回退都不命中，最终换成 `ash`；保尸分支先熄火且不执行 `DropDeathLoot`，只有非保尸普通死亡才保底 3 份。[转换](../../../raw/current/scripts/components/lootdropper.lua#L415-L442)、[保尸分支](../../../raw/current/scripts/stategraphs/SGBeefalo.lua#L642-L655)
- **边界：幼牛的表会整表替换。** 三个成长阶段分别 `SetLoot`，后一阶段不会继承前一阶段的表；不要按“累计掉落”写。[阶段](../../../raw/current/scripts/prefabs/babybeefalo.lua#L70-L89)
- **边界：`tradable` 已挂但金值为 0。** 组件默认 `goldvalue=0`，prefab 没有赋值；不要把它写成有金值的交易品，交易端是否接受还要看对方条件。[默认值](../../../raw/current/scripts/components/tradable.lua#L1-L7)、[赋值缺失](../../../raw/current/scripts/prefabs/beefalowool.lua#L37-L37)
- **边界：伯尼腐坏是固定三件。** 每次 `dodecay` 各生成 1 个胡须、牛毛、蜘蛛丝，不是随机池；必须空燃料且未持有。[掉落](../../../raw/current/scripts/prefabs/bernie_inactive.lua#L28-L36)

## 验证记录

- 源码审阅：2026-11-10，Steam buildid 26474873。通读 `prefabs/beefalowool.lua` 全文、`prefabs/beefalo.lua` 的 `beard`／`brushable`／共享战利品表、`prefabs/babybeefalo.lua` 的三阶段表、`prefabs/bernie_inactive.lua` 的腐坏链、`prefabs/tumbleweed.lua` 的权重与抽 3 件、`components/brushable.lua`／`beard.lua`／`lootdropper.lua`／`tradable.lua`、`prefabs/pillow_defs.lua`、`standardcomponents.lua`、`recipes.lua` 的 20 余条配方、`tuning.lua` 的堆叠／燃料／时间基数、`chinese_s.po` 的名称。
- 游戏运行：未运行。未核验：幼牛燃烧死亡的逐项转换、风滚草实际开包频率。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 见交付回报。

## 相关链接

- 宿主：[[reference/prefabs/beefalo]]
- 刷毛组件：[[reference/components/brushable]]
- 胡须：[[reference/prefabs/beardhair]]
- 伯尼：[[reference/prefabs/character-gear-extra]]
- 枕头与鞍：[[reference/prefabs/role-tools]]
- 小痒痒：[[reference/prefabs/mosquitomusk]]
- 返回 [[reference/prefabs/index|Prefab 索引]]
