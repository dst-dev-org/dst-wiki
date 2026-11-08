---
title: "大型生物掉落材料：熊皮与鳞片（bearger_fur、dragon_scales）"
created: "2026-11-10"
updated: "2026-11-10"
type: prefab
summary: "熊皮与鳞片两种大型生物材料的对照页：均为单文件注册、10 叠、med 漂浮、作祟只弹飞、无 edible／tradable／burnable；来源是熊獾与龙蝇的必掉条目（龙蝇另有补鳞：倒地累计血量变化，正负都计），并汇总冰包、熊皮背心、极地熊獾桶、龙鳞火炉、鳞甲、龙鳞宝箱、龙鳞地板与坍塌宝箱重建的全部消费与科技门禁，写清 TECH.LOST 蓝图、辉煌铁匠铺、拆解回收与普通／受控烧毁改写（受控时熊皮保留原名、不冒烟）。"
aliases: ["bearger_fur", "dragon_scales", "熊皮", "鳞片", "Thick Fur", "Scales", "Boss 材料", "大型生物掉落材料"]
tags: [prefab, item]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/bearger_fur.lua", "prefabs/dragon_scales.lua", "prefabs/bearger.lua", "prefabs/dragonfly.lua", "stategraphs/SGbearger.lua", "stategraphs/SGdragonfly.lua", "stategraphs/commonstates.lua", "components/lootdropper.lua", "components/damagetracker.lua", "components/stackable.lua", "components/health.lua", "components/corpsepersistmanager.lua", "components/mutatedbuzzardmanager.lua", "components/builder.lua", "components/spellcaster.lua", "components/burnable.lua", "prefabs/furtuft.lua", "widgets/redux/craftingmenu_widget.lua", "componentutil.lua", "entityscript.lua", "prefabs/corpses.lua", "prefabs/corpses_defs.lua", "standardcomponents.lua", "physics.lua", "recipes.lua", "recipes_filter.lua", "constants.lua", "prefabs/blueprint.lua", "prefabs/klaus_sack.lua", "components/klaussackloot.lua", "prefabs/dragonfly_chest.lua", "prefabs/collapsedchest.lua", "prefabs/beargerfur_sack.lua", "prefabs/lunar_forge.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 大型生物掉落材料：熊皮与鳞片（bearger_fur、dragon_scales）

## 快速判断

- 职责与适用：两个独立文件、各自 `Prefab()` 注册的材料——熊皮（`bearger_fur`）与鳞片（`dragon_scales`），没有共享工厂。两者都是可拾取、可堆叠 10 的纯材料，组件段只有 `inspectable`／`inventoryitem`／`stackable`／`MakeHauntableLaunch`；主要身份是配方材料。[熊皮注册](../../../raw/current/scripts/prefabs/bearger_fur.lua#L6-L39)、[鳞片注册](../../../raw/current/scripts/prefabs/dragon_scales.lua#L6-L39)
- 优先使用：查两注册与同批共性看[注册与构造](#注册与构造)；查宿主、龙蝇补鳞、克劳斯袋与尸体改写看[获得](#获得)；查全部配方与科技门禁看[用途与配方](#用途与配方)。
- 重要副作用：两者**都没有 `burnable`**，落地后不会自燃；所谓"烧毁改写"只在掉落者带 `burnt`，或正在燃烧且（`fueled` 为 nil 或 `burnable.ignorefuel` 为真）时由 `lootdropper` 改写列表；受控燃烧保留原名。作祟走 `MakeHauntableLaunch`，只弹飞，不砸碎、不点燃。[作祟 helper](../../../raw/current/scripts/standardcomponents.lua#L1053-L1069)
- 修改边界：本页不覆盖熊獾／龙蝇的战斗、生成与刷新（只列掉落表与死亡结算入口）；龙蝇倒地累计血量变化的 `damagetracker` 细节归 [[reference/components/damagetracker]]；尸体的月变与变异秃鹫路径只写到影响本材料结算为止。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `bearger_fur` | 单文件单注册 `Prefab("bearger_fur", fn, assets)`，无包装工厂。公共端：`CreateEntity` → Transform／AnimState／SoundEmitter／Network → `MakeInventoryPhysics` → bank／build `bearger_fur` → `idle` → `MakeInventoryFloatable("med", nil, 0.77)` → `SetPristine`；非主模拟返回。 | 主模拟只挂 4 件：`inspectable` → `inventoryitem` → `stackable`（显式 `maxsize = STACK_SIZE_LARGEITEM`＝10）→ `MakeHauntableLaunch`。无 `edible`／`cookable`／`perishable`／`tradable`／`bait`／`burnable`／`snowmandecor`。[公共端与主模拟](../../../raw/current/scripts/prefabs/bearger_fur.lua#L6-L39) |
| `dragon_scales` | 同样的单文件单注册与公共端，唯一差异是 floatable 参数 `("med", nil, 0.82)`；`SetPristine` 后主模拟。 | 主模拟组件与熊皮相同：`inspectable`／`inventoryitem`／`stackable`（10）／`MakeHauntableLaunch`；同样没有 edible／tradable／burnable／snowmandecor。[全文](../../../raw/current/scripts/prefabs/dragon_scales.lua#L6-L39) |

### 同批掉落物的共性

本批四种大型生物掉落物（含 [[reference/prefabs/boss-parts]] 的眼球与角）在堆叠、漂浮、作祟和可燃四项上走同一套最小接线；本表是本批的共享主章节，器官页只写自己的 edible／tradable／雪人差异。

| 注册名 | 堆叠 | 漂浮参数（size／offset／scale） | 作祟 | 可燃 |
|---|---:|---|---|---|
| `bearger_fur` | 10 | `med`／nil／0.77 | `MakeHauntableLaunch`：必弹飞、默认冷却 3 秒、`HAUNT_TINY` | 无 `burnable` |
| `dragon_scales` | 10 | `med`／nil／0.82 | 同上 | 无 `burnable` |
| `deerclops_eyeball` | 10 | `small`／0.2／0.95 | 同上 | 无 `burnable` |
| `minotaurhorn` | 10 | `med`／0.05／0.75 | 同上 | 无 `burnable` |

- 眼球与角两行的堆叠／漂浮参数与作祟接线分别见 [眼球](../../../raw/current/scripts/prefabs/deerclops_eyeball.lua#L19-L21)、[角](../../../raw/current/scripts/prefabs/minotaurhorn.lua#L19-L21)；两者的 edible／tradable 差异归 [[reference/prefabs/boss-parts]]。
- 堆叠 10 来自 `TUNING.STACK_SIZE_LARGEITEM`；四个文件都显式写 `maxsize`，不是组件默认的 20。[常量](../../../raw/current/scripts/tuning.lua#L81-L83)、[stackable 默认](../../../raw/current/scripts/components/stackable.lua#L23-L34)
- 漂浮在 `SetPristine` 之前调用，因此客户端副本也有 `floater`；helper 依次做 `SetSize`／`SetVerticalOffset`（有 offset 时）／`SetScale`（有 scale 时）。[helper](../../../raw/current/scripts/standardcomponents.lua#L1772-L1791)
- 作祟只走 `MakeHauntableLaunch` 的实际生效段：默认 `HAUNT_CHANCE_ALWAYS`＝1 必中，`Launch` 速度取 `LAUNCH_SPEED_SMALL`＝3，并把 `inventoryitem.is_landed` 置 false。名称相近的 `LaunchAndSmash`／`LaunchAndIgnite` 的附加段在本 build 被 `#HAUNTFIX` 注释，不能按 helper 名推断砸碎或点燃。[Launch](../../../raw/current/scripts/standardcomponents.lua#L1053-L1069)、[常量](../../../raw/current/scripts/tuning.lua#L3044-L3065)

### 名称与实显

`NAMES.BEARGER_FUR`＝**熊皮**（英文 Thick Fur）；`NAMES.DRAGON_SCALES`＝**鳞片**（英文 Scales）。两个文件都不设 `nameoverride`，显示名走实体默认的 `STRINGS.NAMES[注册名大写]`。[熊皮键](../../../raw/current/scripts/languages/chinese_s.po#L339229-L339232)、[鳞片键](../../../raw/current/scripts/languages/chinese_s.po#L342254-L342257)、[默认取名](../../../raw/current/scripts/entityscript.lua#L840-L843)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`stackable` | 承载 10 上限的合并栈；堆叠变化触发 `stacksizechange` | 主模拟；改上限要显式写 `maxsize` | [[reference/components/stackable#关键接口]]；[熊皮赋值](../../../raw/current/scripts/prefabs/bearger_fur.lua#L31-L32)、[鳞片赋值](../../../raw/current/scripts/prefabs/dragon_scales.lua#L31-L32) |
| 组件：`floater` | 水面漂浮与垂直偏移；客户端也有 | 两端（`SetPristine` 前）；替换参数见上表 | [[reference/components/floater#关键接口]]；[熊皮](../../../raw/current/scripts/prefabs/bearger_fur.lua#L20-L20)、[鳞片](../../../raw/current/scripts/prefabs/dragon_scales.lua#L20-L20) |
| 组件：`hauntable` | `MakeHauntableLaunch`：弹飞并把落地状态清掉 | 主模拟；不破坏、不点燃 | [[reference/components/hauntable#按效果选工厂]]；[helper](../../../raw/current/scripts/standardcomponents.lua#L1053-L1069) |
| 缺失组件（否定断言） | 无 `edible`／`cookable`／`perishable`／`tradable`／`bait`／`burnable`／`snowmandecor` | 由两份 39 行文件全文确认；不要按"Boss 材料"推断可食或可燃 | [熊皮全文](../../../raw/current/scripts/prefabs/bearger_fur.lua#L6-L39)、[鳞片全文](../../../raw/current/scripts/prefabs/dragon_scales.lua#L6-L39) |
| API／数据：`lootdropper` | 宿主死亡时按共享表掷骰；按 burnt／燃烧与燃料条件改写列表；尸体延后结算 | 服务端；本页材料本身不挂组件 | [[reference/components/lootdropper#关键接口]]；[掷骰](../../../raw/current/scripts/components/lootdropper.lua#L213-L227)、[改写](../../../raw/current/scripts/components/lootdropper.lua#L415-L445) |
| 关联 Prefab：`furtuft` | 90 撮换 3 张熊皮的配方产物；尸体结算在多件列表且含熊皮时按 30 的基数折算成 11–20（meat_level=2）或 1–10（=3）撮，未啃到 2 级则不折算、仍掉 1 张熊皮 | 制作／尸体结算 | [[reference/prefabs/furtuft#配方与燃料]]；[配方](../../../raw/current/scripts/recipes.lua#L470)、[覆写表](../../../raw/current/scripts/prefabs/corpses_defs.lua#L1467-L1472) |
| 关联 Prefab：`dragonflyfurnace`／`dragonflychest`／`turf_dragonfly` | 鳞片的配方消费者；三者均无专门对象页 | 配方消费；火炉的焚烧功能见 incinerator 页 | Wiki 待建；[[reference/components/incinerator]]；[配方](../../../raw/current/scripts/recipes.lua#L435)、[L561](../../../raw/current/scripts/recipes.lua#L561)、[L696](../../../raw/current/scripts/recipes.lua#L696) |
| 关联 Prefab：`beargerfur_sack` | 熊皮的辉煌铁匠铺配方产物（极地熊獾桶）；本体 B22 待建，霜气 FX 已有专页 | 辉煌铁匠铺制造 | [[reference/prefabs/beargerfur_sack_frost_fx]]；[配方](../../../raw/current/scripts/recipes.lua#L857)、[容器与 preserve](../../../raw/current/scripts/prefabs/beargerfur_sack.lua#L188-L202) |

## 获得

### 宿主掉落的通用结算

- 掉落表：宿主用 `SetSharedLootTable` 登记一张共享表，再用 `lootdropper:SetChanceLootTable("名字")` 绑定。`GenerateLoot` 逐条处理：概率 ≥1.0 的条目直接入列；<1.0 的行先过 `GetChance`（击杀者幸运会抬高实际概率），再按 `math.random() <= chance` 独立判定；条目之间不互相挤占。[登记与绑定](../../../raw/current/scripts/components/lootdropper.lua#L22-L28)、[逐条掷骰](../../../raw/current/scripts/components/lootdropper.lua#L213-L227)、[幸运修正](../../../raw/current/scripts/components/lootdropper.lua#L173-L184)、[熊獾绑定](../../../raw/current/scripts/prefabs/bearger.lua#L581-L581)、[龙蝇绑定](../../../raw/current/scripts/prefabs/dragonfly.lua#L562-L562)
- 死亡入口：各宿主 SG 的 `death` 状态调用 `DropDeathLoot`。`is_corpsing` 为真时，它显式 `GenerateLoot()` 并把结果存入 `deathloothandler`，随后 `TryEntityToCorpse` 把 `GetDeathLoot` 交给 `*corpse`；为假时只把位置交给 `lootdropper:DropLoot(pos)`，由 `DropLoot` 在掉落时生成。不要把 `GenerateLoot` 写在两支之前。[DropDeathLoot](../../../raw/current/scripts/entityscript.lua#L2181-L2192)、[is_corpsing](../../../raw/current/scripts/components/health.lua#L583-L591)、[熊獾死亡帧](../../../raw/current/scripts/stategraphs/SGbearger.lua#L1545-L1572)、[龙蝇死亡帧](../../../raw/current/scripts/stategraphs/SGdragonfly.lua#L838-L872)
- 成尸资格按顺序：`EntityHasCorpse`（SG 有 `corpse` 且无 `sg.mem.nocorpse`）→ `forcecorpse` → 燃烧中阻止其余分支 → `corpsepersistmanager`（本版只由变异秃鹫管理器注册）→ 月变判定。本页宿主没有 `forcecorpse`，所以强制成尸不是原版默认路径；普通击杀通常**直接掉落**，尸体是条件路径。[资格](../../../raw/current/scripts/componentutil.lua#L1546-L1561)、[转移](../../../raw/current/scripts/componentutil.lua#L1563-L1599)、[唯一注册点](../../../raw/current/scripts/components/mutatedbuzzardmanager.lua#L377-L409)
- 尸体结算：`DropCorpseLoot` 先 `ProcessCorpseLoot`，再把处理后的 `corpse_loot` 作为 `DropLoot` 的显式第二参传入，不会重新掷表。燃尽走 `OnBurnt`（`no_destroy_on_burn` 时跳过）；熄灭时 `Burnable:Extinguish` 先清 `burning`／`controlled_burn` 再回调，尸体在未变异且无 `_skip_extinguish_fade`／`no_destroy_on_burn` 时才走 `OnExtinguish`→`DefaultExtinguishCorpseFn`→`DropCorpseLoot`，否则不结算。[燃尽](../../../raw/current/scripts/prefabs/corpses.lua#L38-L43)、[熄灭](../../../raw/current/scripts/prefabs/corpses.lua#L45-L52)、[提交](../../../raw/current/scripts/prefabs/corpses.lua#L489-L495)、[熄灭顺序](../../../raw/current/scripts/components/burnable.lua#L465-L490)
- 烧毁改写：`DropLoot` 按实际调用时的状态进入改写——掉落者带 `burnt`，或正在燃烧且（`fueled` 为 nil 或 `burnable.ignorefuel` 为真）。顺序：`BURNED_LOOT_OVERRIDES` → `_cooked` 变体 → `cooked` 前缀 → 受控燃烧保留 → 其余非结构且非 `tree`／`boulder`（或蜂巢）变 `ash`。熊皮没有熟制前缀和覆盖项：普通燃烧时 1 个灰烬，受控燃烧时保留原名、自身无 `burnable` 不冒烟。鳞片不进入该分支（原版龙蝇与尸体不可燃，详见下节）。[改写块](../../../raw/current/scripts/components/lootdropper.lua#L418-L442)、[落地闷烧](../../../raw/current/scripts/components/lootdropper.lua#L382-L407)、[覆盖表](../../../raw/current/scripts/tuning.lua#L4511-L4530)
- 尸体折算的门禁与啃空：`ProcessCorpseLoot` 只在 `meat_level >= 2`、`#corpse_loot > 1` 且列表含 `bearger_fur` 时换毛丛；单项列表不换，改走 `math.random() > meat_perc` 的保留判定（可能整表清空）。啃空时 `OnChomped`→`SetMeat` 先把 `meat` 写成 0，`SetMeatLevel(4)` 在写入 4 级前调 `DoErode`→`DropCorpseLoot`，所以读到的是旧 `meat_level`：旧级 2／3 且多件列表仍有熊皮 → 1 撮；旧级 1 一口啃空 → 跳过换算，仍 1 张熊皮。有肉时 2 级 11–20、3 级 1–10（30 基数）。[列表分支](../../../raw/current/scripts/prefabs/corpses.lua#L465-L495)、[啃食与写级](../../../raw/current/scripts/prefabs/corpses.lua#L196-L214)、[侵蚀先于写级](../../../raw/current/scripts/prefabs/corpses.lua#L178-L184)

### 本页材料的来源

| 材料 | 来源 | 产出 | 条件／概率 | 证据 |
|---|---|---|---|---|
| `bearger_fur` | 熊獾（`bearger`） | 1 张 | 普通共享表一条 1.00；同表另有 8 个 `meat` 与雕塑草图 | [表](../../../raw/current/scripts/prefabs/bearger.lua#L76-L88)、[绑定](../../../raw/current/scripts/prefabs/bearger.lua#L581-L581) |
| `bearger_fur` | 毛丛兑换（配方） | 3 张 | 90 个 `furtuft`，`TECH.SCIENCE_TWO`；归 REFINE 筛选 | [配方](../../../raw/current/scripts/recipes.lua#L470)、[筛选](../../../raw/current/scripts/recipes_filter.lua#L997-L1017) |
| `bearger_fur` | 克劳斯袋（正确钥匙开启） | 0–1 张 | 每次开袋的常规 4 个包裹中（冬宴另加 2 个），有 1 个固定含 2 件取自 9 项 `giant_loot3`（两项互不相同），熊皮是其中一项；非保底 | [抽取](../../../raw/current/scripts/components/klaussackloot.lua#L116-L124)、[开袋](../../../raw/current/scripts/prefabs/klaus_sack.lua#L66-L68) |
| `dragon_scales` | 龙蝇（`dragonfly`） | 1 片 | 共享表一条 1.00 | [表](../../../raw/current/scripts/prefabs/dragonfly.lua#L38-L75)、[绑定](../../../raw/current/scripts/prefabs/dragonfly.lua#L562-L562) |
| `dragon_scales` | 龙蝇倒地补鳞（`DoBreakOff`） | +1 片 | 倒地启用窗口内累计血量变化（`math.abs`，正负都计）跨过 `TUNING.DRAGONFLY_BREAKOFF_DAMAGE`＝2500；每次实例至多触发一次 | [回调](../../../raw/current/scripts/prefabs/dragonfly.lua#L381-L384)、[阈值](../../../raw/current/scripts/tuning.lua#L2626-L2627)、[组件](../../../raw/current/scripts/components/damagetracker.lua#L24-L35) |
| `dragon_scales` | 克劳斯袋（正确钥匙开启） | 0–1 片 | 同一包裹还固定含 1 件取自 5 项 `giant_loot1`，鳞片是其中一项；非保底 | [抽取](../../../raw/current/scripts/components/klaussackloot.lua#L116-L124) |

- **龙蝇补鳞只跨一次**：启用时每个 `healthdelta` 都按 `math.abs(data.amount)` 累加（治疗也计）；`damagetracker` 的 `damage_done` 不随 `Stop` 清零，回调条件要求"旧值 < 阈值且新值 ≥ 阈值"，跨过后同一实例不再触发；组件不存档，重载重建的实例从 0 重新累计。启停时机与"不是每倒地一次"的完整说明见 [[reference/components/damagetracker#常见陷阱与限制]]。[计数](../../../raw/current/scripts/components/damagetracker.lua#L24-L35)、[启停](../../../raw/current/scripts/stategraphs/SGdragonfly.lua#L279-L327)
- **补鳞是直接 `SpawnPrefab`**：不经过宿主掉落表，也不受尸体／燃烧改写影响；`LaunchAt` 把它从龙蝇身边 1.5 格、3 格高度抛向最近玩家。[回调](../../../raw/current/scripts/prefabs/dragonfly.lua#L381-L384)、[LaunchAt](../../../raw/current/scripts/physics.lua#L114-L137)
- **变异体另一张表**：`mutatedbearger` 的表不含熊皮，`mutateddeerclops` 的表不含眼球；本页两种材料也不来自裂隙变异体。[变异熊獾表](../../../raw/current/scripts/prefabs/bearger.lua#L90-L99)、[变异鹿表](../../../raw/current/scripts/prefabs/deerclops.lua#L343-L354)
- **尸体换算不是获得**：`bearger_fur` 只在上述"尸体 + `meat_level >= 2` + 多件列表且含熊皮"路径按 30 的基数折算成毛丛（level 2 → 11–20、level 3 → 1–10）；未到 2 级仍是 1 张熊皮，正常击杀不会触发；一口啃空时旧级 2／3 仍得 1 撮、旧级 1 跳过换算。[结算](../../../raw/current/scripts/prefabs/corpses.lua#L465-L496)

### 烧毁对两种材料的实际结果

| 材料 | 掉落者燃烧时 | 成立条件 | 证据 |
|---|---|---|---|
| `bearger_fur`（未换算） | 普通燃烧：1 个 `ash`；受控燃烧：保留原名、不冒烟 | 熊獾死亡时仍在燃烧 → 直接结算；或尸体燃尽；未走尸体折算 | [改写块](../../../raw/current/scripts/components/lootdropper.lua#L418-L442)、[尸体燃尽](../../../raw/current/scripts/prefabs/corpses.lua#L38-L43) |
| `bearger_fur`（已换算成 `furtuft`） | 普通燃烧：逐项等量 `ash`；受控燃烧：保留 `furtuft` 并冒烟 | 尸体 `meat_level>=2` 且多件列表（如 2 级剩半肉＝15 撮），再满足燃烧改写 | [列表分支](../../../raw/current/scripts/prefabs/corpses.lua#L465-L495)、[furtuft 可燃](../../../raw/current/scripts/prefabs/furtuft.lua#L53-L54) |
| `dragon_scales` | 不会被改写 | 按本版原版接线，龙蝇构造无 `burnable`／`Make*Burnable`；龙蝇尸体无 `makeburnablefn`，鳞片本身也无 `burnable` | [龙蝇全文](../../../raw/current/scripts/prefabs/dragonfly.lua#L1-L624)、[尸体定义](../../../raw/current/scripts/prefabs/corpses_defs.lua#L1081-L1090)、[尸体可燃门禁](../../../raw/current/scripts/prefabs/corpses.lua#L750-L756) |

## 用途与配方

### 熊皮（`bearger_fur`）

| 用途／产物 | 配方 | 科技／条件 | 证据 |
|---|---|---|---|
| 熊皮 ×3（反向产出） | `furtuft`×90 | `TECH.SCIENCE_TWO`；`numtogive = 3` | [配方](../../../raw/current/scripts/recipes.lua#L470-L470) |
| 冰包（`icepack`，实显"保鲜背包"） | 熊皮×1＋`gears`×1＋`transistor`×1 | `TECH.SCIENCE_TWO` | [配方](../../../raw/current/scripts/recipes.lua#L518-L518)、[[reference/prefabs/icepack]] |
| 熊皮背心（`beargervest`） | 熊皮×1＋`sweatervest`×1＋`rope`×2 | `TECH.SCIENCE_TWO` | [配方](../../../raw/current/scripts/recipes.lua#L553-L553)、[[reference/prefabs/clothing]] |
| 极地熊獾桶（`beargerfur_sack`） | 熊皮×1＋`security_pulse_cage_full`×1＋`purebrilliance`×3＋`moonrocknugget`×5 | `TECH.LUNARFORGING_TWO`（`LUNARFORGING = 2`）＋`nounlock`；`station_tag = "lunar_forge"` 仅用于制作菜单筛选，不是门禁 | [配方](../../../raw/current/scripts/recipes.lua#L857-L857)、[[reference/prefabs/moonrocknugget#主要消费]] |

### 鳞片（`dragon_scales`）

| 用途／产物 | 配方 | 科技／条件 | 证据 |
|---|---|---|---|
| 龙鳞火炉（`dragonflyfurnace`） | 鳞片×1＋`redgem`×2＋`charcoal`×10 | `TECH.LOST`；由龙蝇必掉的 `dragonflyfurnace_blueprint` 教授 | [配方](../../../raw/current/scripts/recipes.lua#L435-L435)、[蓝图掉落](../../../raw/current/scripts/prefabs/dragonfly.lua#L38-L42) |
| 鳞甲（`armordragonfly`） | 鳞片×1＋`armorwood`×1＋`pigskin`×3 | `TECH.SCIENCE_TWO` | [配方](../../../raw/current/scripts/recipes.lua#L498-L498)、[[reference/prefabs/armor-boss]] |
| 龙鳞宝箱（`dragonflychest`） | 鳞片×1＋`boards`×4＋`goldnugget`×10 | `TECH.SCIENCE_TWO`；带 placer | [配方](../../../raw/current/scripts/recipes.lua#L561-L561) |
| 龙鳞地板（`turf_dragonfly`） | 鳞片×1＋`cutstone`×1 | `TECH.SCIENCE_TWO`；`numtogive = 4` | [配方](../../../raw/current/scripts/recipes.lua#L696-L696) |
| 坍塌龙鳞宝箱（`collapsed_dragonflychest`）重建 | 鳞片×1＋`boards`×4＋`goldnugget`×10＋`alterguardianhatshard`×1 | `CONSTRUCTION_PLANS` 的建造材料，不是制造配方；宝箱超量堆叠坍塌后原地重建 | [计划](../../../raw/current/scripts/recipes.lua#L1539-L1540)、[坍塌](../../../raw/current/scripts/prefabs/dragonfly_chest.lua#L37-L56)、[重建完成](../../../raw/current/scripts/prefabs/collapsedchest.lua#L69-L91) |

- **科技门禁的分层**：`TECH.SCIENCE_TWO` 是 `SCIENCE = 2`（二本炼金引擎）；`TECH.LUNARFORGING_TWO` 是 `LUNARFORGING = 2`。`TECH.LOST` 则是三系都写 10 的"失落配方"标记，正常科技树无法满足，靠蓝图学会。[科技表](../../../raw/current/scripts/constants.lua#L1241-L1254)、[LOST](../../../raw/current/scripts/constants.lua#L1324-L1324)
- **`TECH.LOST` 的龙鳞火炉**：具体蓝图 prefab 由 `AllRecipes` 中"先排除 `nounlock`／`builder_tag`、再要求任一科技值 > 0"的条目批量生成；龙蝇共享表里必掉 1 张 `dragonflyfurnace_blueprint`，使用后教会该配方。[生成资格](../../../raw/current/scripts/prefabs/blueprint.lua#L62-L74)、[蓝图生成](../../../raw/current/scripts/prefabs/blueprint.lua#L240-L243)、[蓝图条目](../../../raw/current/scripts/prefabs/dragonfly.lua#L38-L42)、[教学回调](../../../raw/current/scripts/prefabs/blueprint.lua#L40-L41)
- **`nounlock` 与辉煌铁匠铺**：极地熊獾桶的门禁是 `LUNARFORGING = 2`（本版由[辉煌铁匠铺](../../../raw/current/scripts/languages/chinese_s.po#L344915-L344917)的原型机提供）＋`nounlock`（不写解锁记录、不进随机蓝图池）；`station_tag = "lunar_forge"` 只决定是否在制作菜单的「制作站」筛选页列出。[科技树](../../../raw/current/scripts/tuning.lua#L1448-L1450)、[builder 按科技可达放行](../../../raw/current/scripts/components/builder.lua#L883-L916)、[解锁路径](../../../raw/current/scripts/components/builder.lua#L1006-L1020)、[蓝图排除](../../../raw/current/scripts/prefabs/blueprint.lua#L44-L48)、[菜单筛选](../../../raw/current/scripts/widgets/redux/craftingmenu_widget.lua#L199-L203)
- **拆解回收**：绿色法杖以 `AllRecipes[target.prefab]` 是否存在且非 `no_deconstruction` 放行，按配方材料与耐久百分比返还、每项至少 1。熊皮自己有一条产出配方，所以能被拆回 `ceil(90 / 3) = 30` 个毛丛；鳞片没有自身配方，不能被直接拆解，只能从鳞甲／龙鳞宝箱／龙鳞火炉／龙鳞地板等产物回收。[拆解实现](../../../raw/current/scripts/prefabs/staff_common.lua#L637-L674)、[施法门禁](../../../raw/current/scripts/components/spellcaster.lua#L190-L191)、[[reference/prefabs/staffs#拆解链greenstaff]]

## 常见陷阱与限制

- **误区：熊皮／鳞片"属于 Boss 材料所以可燃"**——两份文件都没有 `burnable`，落地不会自燃；普通燃烧的宿主／尸体会把未换算的熊皮换成 1 个灰烬；受控燃烧保留原名（自身无 `burnable`，不冒烟）；已换算成 `furtuft` 的条目按 `furtuft` 逐项改写（受控时保留并冒烟）。这不是物品自身的燃烧。[熊皮全文](../../../raw/current/scripts/prefabs/bearger_fur.lua#L6-L39)、[改写块](../../../raw/current/scripts/components/lootdropper.lua#L418-L442)
- **误区：龙蝇每倒地一次就补一片鳞**——`damagetracker` 累计血量变化跨过 2500 后同一实例不再触发，且组件不存档；参见 [[reference/components/damagetracker#常见陷阱与限制]]。补鳞确实存在，但不是"每次倒地一份"。[计数](../../../raw/current/scripts/components/damagetracker.lua#L24-L35)
- **误区：熊皮只能等熊獾掉**——90 个毛丛可换 3 张（二本），克劳斯袋也可能出；但毛丛兑换是 30 撮／张，成本不低。[配方](../../../raw/current/scripts/recipes.lua#L470)、[袋](../../../raw/current/scripts/components/klaussackloot.lua#L116-L124)
- **边界：`TECH.LOST` 不是"科技等级 10"**——它是让正常科技树无法满足的标记；龙鳞火炉必须用龙蝇必掉的蓝图学。[常量](../../../raw/current/scripts/constants.lua#L1324-L1324)、[蓝图](../../../raw/current/scripts/prefabs/dragonfly.lua#L38-L42)
- **边界：极地熊獾桶不是二本配方**——除 `LUNARFORGING = 2` 与 `nounlock` 外，`station_tag` 只影响制作菜单筛选，不是"只能用某个站"的硬门禁；本版能提供该科技的是[辉煌铁匠铺](../../../raw/current/scripts/languages/chinese_s.po#L344915-L344917)。[配置](../../../raw/current/scripts/recipes.lua#L857-L857)、[科技树](../../../raw/current/scripts/tuning.lua#L1448-L1450)、[筛选](../../../raw/current/scripts/widgets/redux/craftingmenu_widget.lua#L199-L203)
- **边界：尸体折算会降级熊皮**——先分开“数量”与“是否换名”：①旧 `meat_level < 2`：跳过折算，熊皮保持原条目；②已达 2 级且 `#corpse_loot == 1`：不换毛丛，走 `math.random() > meat_perc` 的保留判定，可能整表清空；存活条目保持原名（若该条是熊皮，仍是 1 张熊皮）；③已达 2 级且列表多于一项并含熊皮：才按 30 的基数换成毛丛（2 级 11–20、3 级 1–10）。啃空时旧级 2／3 仍 1 撮、旧级 1 跳过；普通直接掉落不经过尸体结算。[列表门禁](../../../raw/current/scripts/prefabs/corpses.lua#L465-L485)、[侵蚀先于写级](../../../raw/current/scripts/prefabs/corpses.lua#L158-L184)
- **未核验：变异秃鹫保留尸体与啃食的实际可达性**；本页只按 `CORPSE_DEFS`／`corpsepersistmanager` 的注册与判定写条件，未运行实测。[管理器](../../../raw/current/scripts/components/mutatedbuzzardmanager.lua#L377-L409)

## 验证记录

- 源码审阅：2026-11-10，Steam buildid 26474873。核验范围：两份材料 prefab 全文；`MakeHauntableLaunch`／`MakeInventoryFloatable`／`stackable` 的接线与默认值；熊獾与龙蝇共享掉落表、`SetChanceLootTable`、`GenerateLoot` 的逐条掷骰与 `GetChance` 幸运修正；死亡状态 `DropDeathLoot` 与 `is_corpsing`／`CanEntityBecomeCorpse`（含 `forcecorpse` 优先）／`TryEntityToCorpse` 的即时／尸体分支；尸体燃尽、`meat_level >= 2` 的 `ProcessCorpseLoot` 与熊皮→毛丛按 30 基数折算（1–20）；`lootdropper` 的烧毁改写块（含 `burnable.ignorefuel`、受控燃烧保留与熊皮不冒烟）；龙蝇 `DoBreakOff`、`damagetracker` 与 `LaunchAt`；`recipes.lua` 的 4 条熊皮消费／产出与 5 条鳞片消费；`TECH.LOST`、`LUNARFORGING_TWO`、`nounlock`、`station_tag`（仅菜单筛选）；蓝图生成资格（排除 `nounlock`／`builder_tag`）与教学；克劳斯袋抽取；绿色法杖拆解公式；简中名称键。
- 2026-11-10 检视修订：R01 尸体折算改为按 30 基数折算（`meat_level=2` → 11–20、`=3` → 1–10，未到 2 级仍 1 张熊皮）；R04／R05 将 `station_tag` 改为仅菜单筛选，门禁改记 `LUNARFORGING = 2`＋`nounlock`，`lunar_forge` 站名统一为辉煌铁匠铺；R06 补 `GetChance` 幸运修正；R07 补 `forcecorpse` 优先于燃烧；R08 补 `burnable.ignorefuel`；R09 补蓝图 `nounlock`／`builder_tag` 排除。
- 2026-11-10 检视修订（R2）：R06 统一为雕塑草图；R02 复核 L69 已包含 `GetChance` 幸运修正，无需再改。
- 2026-11-10 外部检视修订（R02–R05、R10）：普通／受控燃烧口径统一；死亡入口、成尸资格、燃尽／熄灭、尸体列表不重掷与啃空旧级分写；熊皮列表门禁与单项保留概率；补鳞改为累计血量变化正负都计；龙蝇证据范围改为全文件并限定原版接线。
- 未覆盖：熊獾／龙蝇的生成、战斗与刷新节奏；变异秃鹫尸体路径与尸体被啃食的实际概率；克劳斯袋的完整包数与其它奖励；拆解与烧毁的运行时表现；模组覆盖后的自定义掉落表。
- 游戏运行：未运行；掉落概率、数量、烧毁结果与拆解数量均为源码推导，不是实测。
- Lua 示例：无（本页未新增可执行 Lua 示例，故未做 Lua 语法检查）。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 见交付回报。

相关：[[reference/prefabs/boss-parts]]（同批器官）、[[reference/prefabs/furtuft]]（熊皮兑换与尸体覆写的另一侧）、[[reference/prefabs/icepack]]、[[reference/prefabs/clothing]]、[[reference/prefabs/armor-boss]]、[[reference/prefabs/moonrocknugget]]、[[reference/prefabs/ashes-and-charcoal]]（烧毁改写通用块）、[[reference/prefabs/staffs]]（拆解链）、[[reference/components/lootdropper]]、[[reference/components/damagetracker]]、[[reference/prefabs/beargerfur_sack_frost_fx]]、[[reference/components/incinerator]]。

返回 [[reference/prefabs/index|原版 Prefab 索引]]。
