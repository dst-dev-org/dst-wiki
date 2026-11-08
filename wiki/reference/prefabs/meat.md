---
title: "肉与熟肉、肉干（meat、cookedmeat、meat_dried）及肉类共用工厂"
created: "2026-11-09"
updated: "2026-11-09"
type: prefab
summary: "meats.lua 的 common() 共用工厂与肉／熟肉／肉干三个注册：标签先于 SetPristine、浮动体两端都挂而其余组件只在主模拟、三维与鲜度、猪王与晾肉架消费，以及作祟把整叠普通肉换成怪物肉的转换链。"
aliases: ["meat", "cookedmeat", "meat_dried", "肉", "熟肉", "肉干", "大肉", "大肉链", "Jerky"]
tags: [prefab, item, food]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/meats.lua", "tuning.lua", "constants.lua", "components/edible.lua", "components/perishable.lua", "components/stackable.lua", "components/dryable.lua", "components/cookable.lua", "components/tradable.lua", "components/dryingrack.lua", "components/lootdropper.lua", "components/hauntable.lua", "standardcomponents.lua", "physics.lua", "componentactions.lua", "cooking.lua", "prefabs/pigking.lua", "prefabs/pigshrine.lua", "prefabs/cookpot.lua", "prefabs/meatrack2.lua", "prefabs/lureplant.lua", "containers.lua", "prefabs/beefalo.lua", "prefabs/koalefant.lua", "prefabs/pigman.lua", "prefabs/charlie_stage.lua", "prefabs/hound.lua", "screens/redux/scrapbookdata.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 肉与熟肉、肉干（meat、cookedmeat、meat_dried）及肉类共用工厂

## 快速判断

- 职责与适用：本页覆盖 `prefabs/meats.lua` 返回的肉／熟肉／肉干三个注册 `meat`／`cookedmeat`／`meat_dried`（俗称“大肉链”，`NAMES.MEAT` 实显是“肉”），同时是同批小肉、怪物肉两页共用的工厂 `common()` 的详述主页面——那两页只写自身差异，共同初始化一律指向[共用工厂与初始化顺序](#共用工厂与初始化顺序)。三个注册是该文件导出表里的前三项。[注册](../../../raw/current/scripts/prefabs/meats.lua#L899-L901)
- 优先使用：改所有肉类的共同行为（标签、`tradable`、默认 `perishable` 档）看共用工厂；改单件数值看[数值与获得](#数值与获得)；改晾晒看[晾肉架与干制](#晾肉架与干制)；改作祟换算[作祟转换](#作祟转换)。
- 重要副作用：作祟会把这三种物品**整叠**换成对应怪物肉形态，并把新件击飞；干制品既不可再烤也不可再上架；怪物肉三态把 `tradable.goldvalue` 置 0，猪王因此拒收。[转换](../../../raw/current/scripts/prefabs/meats.lua#L104-L132)、[收不收](../../../raw/current/scripts/prefabs/pigking.lua#L598-L606)
- 修改边界：宿主生物（皮弗娄牛、考拉象等）、`spoiled_food` 与晾肉架建筑各有归属页待建，本页只给掉落与消费入口；`quagmire` 分支依赖的 `quagmire_event_server/` 不在本快照内，不覆盖。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `meat` | `raw()` → `common("meat","meat","raw",{"catfood","rawmeat"},{product="meat_dried",time=DRY_MED},{product="cookedmeat"})`；主模拟段写三维／鲜度后接 `AddMonsterMeatChange("monstermeat")` | bank 与 build 都是 `meat`，播 `raw`；`catfood`／`rawmeat` 是附加标签；饥饿 25／生命 1／理智 −10；`PERISH_FAST`；`floater` 垂直偏移 0.05。[raw](../../../raw/current/scripts/prefabs/meats.lua#L383-L405) |
| `cookedmeat` | `cooked()` → `common("meat","meat","cooked")`：第 4–6 参全缺，故无附加标签、无 `dryable`、无 `cookable` | 熟态不能再烤、不能上架，也没有 `lureplant_bait`；`PERISH_MED`；作祟目标 `cookedmonstermeat`。[cooked](../../../raw/current/scripts/prefabs/meats.lua#L343-L364) |
| `meat_dried` | `driedmeat()` → `common("meat_rack_food","meat_rack_food","idle_dried_large", nil, {isdried = true})`：bank／build 换成 `meat_rack_food` | 第 5 参只有 `isdried`，所以既无 `dryable`／`cookable` 标签也无对应组件，但仍因第 5 参非空拿到 `lureplant_bait`；`PERISH_PRESERVED`；作祟目标 `monstermeat_dried`。[driedmeat](../../../raw/current/scripts/prefabs/meats.lua#L366-L381) |

### 名称核验

`NAMES.MEAT`＝**肉**、`NAMES.COOKEDMEAT`＝**熟肉**、`NAMES.MEAT_DRIED`＝**肉干**（英文 msgid 依次是 Meat／Cooked Meat／Jerky，简中把 Jerky 译作肉干而非音译）。三键在 `chinese_s.po` 中各命中唯一一条，均无 `NAME_DETAIL_EXTENTION` 附加。[MEAT](../../../raw/current/scripts/languages/chinese_s.po#L345250-L345252)、[COOKEDMEAT](../../../raw/current/scripts/languages/chinese_s.po#L341635-L341637)、[MEAT_DRIED](../../../raw/current/scripts/languages/chinese_s.po#L345285-L345287)

## 共用工厂与初始化顺序

`common(bank, build, anim, tags, dryable, cookable)` 是 `meats.lua` 里唯一的局部工厂，本批 9 页共 27 个物品注册全部经它构造（文件里未纳入本批的 `humanmeat` 三态与 `quagmire` 两态也走同一工厂，只是不计入这 27 个）；它不返回组件表，先在两端挂标签与浮动体，再在主模拟上按固定次序把其余组件挂好。[定义](../../../raw/current/scripts/prefabs/meats.lua#L138-L222)

| 步骤 | 条件与动作 | 证据 |
|---|---|---|
| 1 基础实体 | `CreateEntity` ＋ `Transform`／`AnimState`／`Network` ＋ `MakeInventoryPhysics`；`SetBank`／`SetBuild`／`PlayAnimation(anim)`，并把 `anim` 记进 `scrapbook_anim` | [L139-L150](../../../raw/current/scripts/prefabs/meats.lua#L139-L150) |
| 2 标签 | 恒加 `meat`；`tags` 表逐项加；`dryable.product` 非空才加 `dryable`；只要 `dryable` 非空就加 `lureplant_bait`；`cookable` 非空才加 `cookable` | [L154-L172](../../../raw/current/scripts/prefabs/meats.lua#L154-L172) |
| 3 端分界 | `MakeInventoryFloatable(inst)` 在 `SetPristine()` **之前**执行、两端都会挂（客户端副本只有它和标签，尺寸默认 `small`、不带偏移）；`SetPristine()` 之后非主模拟立刻返回 | [L174-L180](../../../raw/current/scripts/prefabs/meats.lua#L174-L180) |
| 4 公共组件 | `edible`（`ismeat=true`、`FOODTYPE.MEAT`）→ `bait` → `inspectable` → `inventoryitem` → `stackable`（默认 20）→ `tradable`（`GOLD_VALUES.MEAT`）→ `perishable`（`PERISH_FAST`＋`StartPerishing()`＋腐坏换 `spoiled_food`） | [L182-L199](../../../raw/current/scripts/prefabs/meats.lua#L182-L199) |
| 5 数据驱动组件 | `dryable.product` 非空才挂 `dryable` 并设 product／drytime／build／dried_build；`cookable` 非空才挂 `cookable` 并设 product | [L201-L212](../../../raw/current/scripts/prefabs/meats.lua#L201-L212) |
| 6 模式分支 | `TheNet:GetServerGameMode() == "quagmire"` 时调 `event_server_data("quagmire","prefabs/meats").master_postinit(inst, cookable)`；`raw()`／`cooked()` 另有各自的 `master_postinit_raw`／`master_postinit_cooked` | [L214-L216](../../../raw/current/scripts/prefabs/meats.lua#L214-L216)、[raw L400-L402](../../../raw/current/scripts/prefabs/meats.lua#L400-L402)、[cooked L359-L361](../../../raw/current/scripts/prefabs/meats.lua#L359-L361) |
| 7 作祟接线 | `MakeHauntableLaunchAndPerish(inst)`，随后监听 `spawnedfromhaunt` → `OnSpawnedFromHaunt` 以 `LAUNCH_SPEED_SMALL` 抛飞 | [L218-L219](../../../raw/current/scripts/prefabs/meats.lua#L218-L219)、[回调](../../../raw/current/scripts/prefabs/meats.lua#L134-L136) |

- **标签先于 `SetPristine`，浮动体两端都挂，其余组件只在主模拟**：两个"能力标签"（`dryable`、`cookable`）都写在 pristine 之前（源码注释说明这是为优化）；`MakeInventoryFloatable` 也在 pristine 之前调用，所以两端都有 `floater`，客户端副本只有它和标签；其余组件一律在 `SetPristine` 之后的 master sim 段挂载。因此只查标签的筛选（晾肉架的 `itemtestfn`、盐盒的 `itemtestfn`）与 `COOK` 动作的目标判定都不依赖组件是否存在。[注释](../../../raw/current/scripts/prefabs/meats.lua#L161-L172)、[架子筛选](../../../raw/current/scripts/containers.lua#L234-L244)、[盐盒筛选](../../../raw/current/scripts/containers.lua#L1049-L1055)
- **两个可选表分工不同**：第 5 参 `dryable` 同时决定标签（`dryable`、`lureplant_bait`）与组件，第 6 参 `cookable` 只决定 `cookable` 标签与 `product`。传 `nil` 就是不挂；传只含 `isdried` 的表则只拿到 `lureplant_bait`。
- **`isdried` 是死字段**：全快照检索 `isdried`，只有本文件 5 处实参表出现（另有洞窟冰的 `isdriedup`，是另一个字段）；`common()` 内没有任何读取，其它脚本也不消费它。所以干制品与"普通肉"的差别全部来自第 5 参只给了 `isdried` 这一点，而非该字段本身。
- **堆叠用组件默认值**：第 4 步没有设 `maxsize`，20 来自 `stackable` 构造里的 `TUNING.STACK_SIZE_MEDITEM`；小肉三态在各自函数里另改成 40。[默认值](../../../raw/current/scripts/components/stackable.lua#L23-L34)、[常量](../../../raw/current/scripts/tuning.lua#L81-L83)
- **作祟默认行为**：`MakeHauntableLaunchAndPerish` 的成功分支把 `hauntvalue` 设为 `HAUNT_TINY`、冷却设为 `HAUNT_COOLDOWN_SMALL`，并调 `Launch` 后把物品置为未落地。[L1311-L1337](../../../raw/current/scripts/standardcomponents.lua#L1311-L1337)

## 数值与获得

时间基准：`seg_time`＝30 秒、`total_day_time`＝480 秒（16 段），`perish_warp`＝1；鲜度常量按 `total_day_time` 折算，晾晒常量也按它折算。[基准](../../../raw/current/scripts/tuning.lua#L19-L20)、[warp](../../../raw/current/scripts/tuning.lua#L47-L47)

| 注册 | 饥饿／生命／理智 | 鲜度 | 堆叠 | 猪王 | 漂浮 |
|---|---|---|---|---|---|
| `meat` | 25（`CALORIES_MED`）／1（`HEALING_TINY`）／−10（`−SANITY_SMALL`） | `PERISH_FAST`＝6 天＝2880 秒 | 20 | 1 金块 | `small`，偏移 0.05 |
| `cookedmeat` | 25／3（`HEALING_SMALL`）／0（显式写 0） | `PERISH_MED`＝10 天＝4800 秒 | 20 | 1 金块 | `small`，偏移 0.05 |
| `meat_dried` | 25／20（`HEALING_MED`）／15（`SANITY_MED`） | `PERISH_PRESERVED`＝20 天＝9600 秒 | 20 | 1 金块 | 未设偏移 |

数值与图鉴逐项一致（图鉴的 `perishable` 字段即秒数）：`meat` 2880／`cookedmeat` 4800／`meat_dried` 9600，堆叠都是 20。[图鉴 meat](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L862-L862)、[cookedmeat](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L345-L345)、[meat_dried](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L863-L863)、[常量](../../../raw/current/scripts/tuning.lua#L2021-L2064)

- **`edible` 的默认值不参与本链**：三维都在派生函数里显式覆盖；`foodtype` 统一 `FOODTYPE.MEAT`，由属性 setter 生成 `edible_MEAT` 实体标签，`ismeat=true` 只是标志位、本快照内除赋值外没有消费者（`oceanfish.lua` 的注释亦称其未被使用）。[setter](../../../raw/current/scripts/components/edible.lua#L9-L17)、[常量](../../../raw/current/scripts/constants.lua#L2026-L2030)
- **负值会带 `badfood`**：`meat` 的理智为负，因此挂上 `badfood` 标签；`cookedmeat`／`meat_dried` 三维非负，不带该标签。[判定](../../../raw/current/scripts/components/edible.lua#L1-L7)

### 获得路径

| 注册 | 已核验来源（示例，不穷举宿主） | 证据 |
|---|---|---|
| `meat` | 击杀皮弗娄牛：共享掉落表 4 条 `meat`（权重均 1.00）；考拉象：夏／冬／火三种掉落表各 8 个 `meat` ＋对应象鼻；猪人：`AddRandomLoot("meat", 3)` | [牛](../../../raw/current/scripts/prefabs/beefalo.lua#L49-L59)、[象](../../../raw/current/scripts/prefabs/koalefant.lua#L22-L24)、[猪人](../../../raw/current/scripts/prefabs/pigman.lua#L358-L358) |
| `cookedmeat` | 主要来源是烤制：`cookable.product` 指向它，烤制端是带 `cooker` 标签的实体（营火、火坑、`dragonflyfurnace` 等；注册名与文件名不同，该注册的文件是 `prefabs/dragonfurnace.lua`）——烹饪锅只挂 `stewer`、不带 `cooker`，只能出配方不能烤。另有非烤制来源：掉落者已带 `burnt` 标签，或正在燃烧且（`fueled == nil` 或 `burnable.ignorefuel`）时，`DropLoot` 会把 `meat` 换成 `cookedmeat`（`meat_cooked` 不存在，走 `cooked` 前缀分支） | [接线](../../../raw/current/scripts/prefabs/meats.lua#L384-L384)、[COOK 门禁](../../../raw/current/scripts/componentactions.lua#L1096-L1105)、[烹饪锅标签](../../../raw/current/scripts/prefabs/cookpot.lua#L291-L309)、[烧毁换熟](../../../raw/current/scripts/components/lootdropper.lua#L415-L442) |
| `meat_dried` | 主要来源是晾肉架定时完成（`dryable.product` 指向它）。另有两条**非定时**来源：①晾肉架被烧毁时 `DryingRack:OnBurnt` 对每件带 `dryable` 的库存调 `InstantDry`，立刻 `SpawnPrefab` 产物并 `LaunchAt` 抛出，不经过 `OnDoneDrying`；②查理舞台 `charlie_stage`：`OnPlayPerformed` 仅在 `not data.next and not data.error and not data.skip_hound_spawn` 三条都成立时才从随机奖励组发奖，`weather` 组含 `meat_dried`；逐件生成的 `hedgehound_bush` 转成 `hedgehound` 时把奖励记进 `hedgeitem`，击杀即掉出 | [接线](../../../raw/current/scripts/prefabs/meats.lua#L384-L384)、[烧毁回调](../../../raw/current/scripts/standardcomponents.lua#L76-L79)、[烧毁速干](../../../raw/current/scripts/components/dryingrack.lua#L376-L391)、[奖励组](../../../raw/current/scripts/prefabs/charlie_stage.lua#L154-L162)、[演出发奖](../../../raw/current/scripts/prefabs/charlie_stage.lua#L172-L181)、[击杀掉落](../../../raw/current/scripts/prefabs/hound.lua#L860-L866) |

### 消费端

- **猪王**：收 `tradable.goldvalue > 0` 的物品（`halloweencandyvalue` 的节日糖果与 `pig_token` 另走分支），随后按该值循环生成金块；肉的 `goldvalue` 是 `TUNING.GOLD_VALUES.MEAT`＝1，所以**非鱼人**交付者一件换一块；`AcceptTest` 先判 `giver:HasTag("merm")`，鱼人交任何非 `pig_token` 物品都直接返回 `nil`。[接受条件](../../../raw/current/scripts/prefabs/pigking.lua#L598-L606)、[发金块](../../../raw/current/scripts/prefabs/pigking.lua#L74-L78)、[常量](../../../raw/current/scripts/tuning.lua#L1946-L1951)
- **猪人神龛**：`pigshrine` 的供奉判定按 prefab 名精确匹配，只收 `meat`／`meat_dried`／`cookedmeat` 三件（怪物肉等其它肉类不收），收下后替换神龛上的肉模型。[供奉判定](../../../raw/current/scripts/prefabs/pigshrine.lua#L97-L101)、[模型切换](../../../raw/current/scripts/prefabs/pigshrine.lua#L75-L82)
- **烹饪锅（作食材，不是烤制端）**：`cooking.lua` 给 `meat` 登记 `meat=1`，并自动派生 `meat_cooked`（`meat=1, precook=1`）与 `meat_dried`（`meat=1, dried=1`）；`cookedmeat` 靠别名表映射到 `meat_cooked` 才被认成食材。`dried`／`precook` 两个标签在本快照内没有任何配方 `test` 读取（源码注释亦称 dried 未被使用）。[登记](../../../raw/current/scripts/cooking.lua#L46-L72)、[别名](../../../raw/current/scripts/cooking.lua#L178-L187)、[取值](../../../raw/current/scripts/cooking.lua#L217-L231)
- **晾肉架**：见下节；干制品本身没有 `dryable` 标签，不能再放回架子。
- **`lureplant`**：`lureplant_bait` 标签在 `lureplant.lua` 里触发拾取判定，`meat` 与 `meat_dried` 都带，`cookedmeat` 不带。[消费](../../../raw/current/scripts/prefabs/lureplant.lua#L130-L135)

## 晾肉架与干制

- **架子注册**：`meatrack` 为 3 槽（`meatrack_hermit` 1 槽、`meatrack_hermit_multi` 9 槽同批注册），构造时先挂 `container` 再挂 `dryingrack`，并调 `EnableDrying()`。[注册](../../../raw/current/scripts/prefabs/meatrack2.lua#L584-L592)、[顺序](../../../raw/current/scripts/prefabs/meatrack2.lua#L325-L334)
- **只收带 `dryable` 标签的物品**：`params.meatrack.itemtestfn` 只判 `item:HasTag("dryable")`（另有两条服务端例外：刚生成的替代物、以及移动失败回槽）；`acceptsstacks = false`，一次只能放一件。[筛选](../../../raw/current/scripts/containers.lua#L234-L244)、[不接堆叠](../../../raw/current/scripts/containers.lua#L213-L226)
- **计时**：放上时读 `dryable:GetDryTime()` 起 `DoTaskInTime`；取出后把剩余秒数存进 `item.dryingrack_drytime`，再放回时取 `math.min(math.max(10, 剩余), drytime)`（至少 10 秒）。但这个缓存只活到物品**落地或堆叠数变化**为止：`OnLoseItem` 把 `ForgetItem` 注册到物品的 `stacksizechange` 与 `ondropped`，任一触发就清空 `dryingrack_drytime`，回架按完整时长重算。[L236-L274](../../../raw/current/scripts/components/dryingrack.lua#L236-L274)、[清理](../../../raw/current/scripts/components/dryingrack.lua#L226-L230)、[注册](../../../raw/current/scripts/components/dryingrack.lua#L285-L288)
- **完成**：定时完成（`OnDoneDrying`）时生成 `product` 后**只继承湿度**（`InheritMoisture`），不继承原料的鲜度百分比，产物以满鲜度出现；烧毁速干（`InstantDry`）连湿度都不继承，直接把产物抛出来。不过架子的 `LongUpdate` 在 `dt` 跨过完成时刻时，会把多出的时间用 `product:LongUpdate(v.dt)` 补到产物上，这种情况下产物不再满鲜度。架上显示用 `GetDriedBuildFile()`（内部回退 `GetBuildFile()`），再回退默认 build `meat_rack_food`。[L198-L224](../../../raw/current/scripts/components/dryingrack.lua#L198-L224)、[长更新](../../../raw/current/scripts/components/dryingrack.lua#L393-L420)、[默认 build](../../../raw/current/scripts/components/dryingrack.lua#L1-L1)
- **`meat` 的数据**：只给 `product="meat_dried"` 与 `time=TUNING.DRY_MED`＝960 秒（2 天），没给 `build`／`dried_build`，所以架上用品类默认 build。[接线](../../../raw/current/scripts/prefabs/meats.lua#L384-L384)
- **腐坏倍率按分支顺序分三档**：①正常晾晒（未暂停且 `isinacid = false`）时对带 `dryable` 的物品返回 0（架上不掉鲜度）；②**普通雨**触发暂停（`isinacid = false`）时返回 `nil`，`preserver` 退回倍率 1，架上物品按常规腐坏；③架子处于**酸雨**状态（`isinacid = true`，由 `OnIsAcidRaining` 置位并同时暂停）时返回 `1 + rate`，比常规更快——该分支排在暂停判断之前，与是否暂停无关。[倍率函数](../../../raw/current/scripts/components/dryingrack.lua#L26-L38)、[暂停判定](../../../raw/current/scripts/components/dryingrack.lua#L78-L94)

## 作祟转换

- **只登记在普通肉三态上**：`AddMonsterMeatChange(inst, "monstermeat")` 在 `raw()`、`"cookedmonstermeat"` 在 `cooked()`、`"monstermeat_dried"` 在 `driedmeat()`。全文件检索该函数只有这 3 处调用；怪物肉三态**没有**这个反应，只走普通抛飞。[三处接线](../../../raw/current/scripts/prefabs/meats.lua#L357-L357)、[L378](../../../raw/current/scripts/prefabs/meats.lua#L378-L378)、[L398](../../../raw/current/scripts/prefabs/meats.lua#L398-L398)
- **概率**：`math.random() <= TUNING.HAUNT_CHANCE_OCCASIONAL`，即 0.25；未命中就直接落到抛飞分支。[常量](../../../raw/current/scripts/tuning.lua#L3049-L3055)
- **成功链**：在原坐标生成 `small_puff` → 生成目标 prefab → 双方都有 `stackable` 且旧件是堆时 `SetStackSize` 继承件数 → 继承湿度／湿润 → 继承鲜度百分比 → 新件推 `spawnedfromhaunt`（带 `haunter` 与 `oldPrefab`）→ 旧件推 `despawnedfromhaunt`、`persists = false`、`Hide()`、下一帧 `Remove`。[L104-L132](../../../raw/current/scripts/prefabs/meats.lua#L104-L132)
- **新件会被击飞**：`spawnedfromhaunt` 的回调调 `Launch(inst, haunter, LAUNCH_SPEED_SMALL)`；`Launch` 以 3 为基准再叠 0–2 随机速度，并把物品置为未落地。[回调](../../../raw/current/scripts/prefabs/meats.lua#L134-L136)、[Launch](../../../raw/current/scripts/physics.lua#L64-L81)
- **与抛飞叠加**：这里用 `AddHauntableCustomReaction(inst, fn, false, true, false)`——自定义反应排在原有反应之前，且 `ignoreinitialresult = true`，所以无论转换成功与否，`common()` 装的 `MakeHauntableLaunchAndPerish` 都会接着执行；成功时旧件已在删除队列，抛飞没有可见效果。[签名](../../../raw/current/scripts/standardcomponents.lua#L1528-L1552)
- **死赋值**：成功分支末尾把旧件的 `hauntable.hauntvalue` 设为 `HAUNT_MEDIUM`，但返回后抛飞分支会把它改成 `HAUNT_TINY`；而 `hauntvalue` 只在 `DoHaunt` 里与 `HAUNT_INSTANT_REZ` 比较，因此该赋值没有可观察效果。[复位](../../../raw/current/scripts/standardcomponents.lua#L1313-L1336)、[消费](../../../raw/current/scripts/components/hauntable.lua#L82-L104)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：edible | `ismeat=true`、`foodtype=MEAT`，三维由派生函数覆盖；负理智触发 `badfood` | 三注册都有；仅主模拟 | [[reference/components/edible#关键接口]]；[L182-L184](../../../raw/current/scripts/prefabs/meats.lua#L182-L184) |
| 组件：bait | 供陷阱当作诱饵；监听 `onremove`／`onpickup`／`oneaten`／`onstolen` | 三注册都有；`bait:IsFree()` 也参与 `selfstacker` 判定 | [[reference/components/bait#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/meats.lua#L186-L186) |
| 组件：tradable | `goldvalue = GOLD_VALUES.MEAT`（1）；怪物肉三态另置 0 | 三注册都有；猪王读它决定收不收与给几块 | [[reference/components/tradable#关键接口]]；[L193-L194](../../../raw/current/scripts/prefabs/meats.lua#L193-L194) |
| 组件：perishable | `SetPerishTime` ＋ `StartPerishing`，腐坏换 `spoiled_food` | 三档：2880／4800／9600 秒；仅主模拟 | [[reference/components/perishable#关键接口]]；[L196-L199](../../../raw/current/scripts/prefabs/meats.lua#L196-L199) |
| 组件：dryable | 仅 `meat` 挂；product／drytime／build／dried_build 来自第 5 参 | `meat`＝`meat_dried`＋`DRY_MED` | [[reference/components/dryable#关键接口]]；[L201-L207](../../../raw/current/scripts/prefabs/meats.lua#L201-L207) |
| 组件：cookable | 仅 `meat` 挂；`product = "cookedmeat"` | `cookedmeat`／`meat_dried` 无该组件 | [[reference/components/cookable#关键接口]]；[L209-L212](../../../raw/current/scripts/prefabs/meats.lua#L209-L212) |
| 组件：floater | `MakeInventoryFloatable(inst)` 在 pristine 前挂上，**两端都有**（客户端副本只有它和标签） | `meat`／`cookedmeat` 偏移 0.05；`meat_dried` 未设 | [[reference/components/floater#关键接口]]；[L174-L174](../../../raw/current/scripts/prefabs/meats.lua#L174-L174) |
| 组件：stackable | 20 上限（组件默认值）；`IsStack()` 决定作祟是否整叠继承件数 | 三注册都有 | [[reference/components/stackable#关键接口]]；[默认](../../../raw/current/scripts/components/stackable.lua#L26-L34) |
| 组件：hauntable | `MakeHauntableLaunchAndPerish` 打底，再被 `AddMonsterMeatChange` 包一层自定义反应 | 三注册都有；自定义反应仅三处接线 | [[reference/components/hauntable#关键接口]]；[L218-L219](../../../raw/current/scripts/prefabs/meats.lua#L218-L219) |
| 事件：`spawnedfromhaunt`／`despawnedfromhaunt` | 新件收到前者后抛飞；旧件推后者携带 `newPrefab` | 仅作祟转换路径；数据含 `haunter`、`oldPrefab`／`newPrefab` | [转换段](../../../raw/current/scripts/prefabs/meats.lua#L121-L122) |
| 关联 Prefab：`meatrack` | `meat` 的干制宿主；只收 `dryable` 标签物品 | 3 槽；另有 `meatrack_hermit`（1 槽）与 `meatrack_hermit_multi`（9 槽）两个变体 | Wiki 待建（晾肉架）；[注册](../../../raw/current/scripts/prefabs/meatrack2.lua#L584-L592) |
| 关联 Prefab：`spoiled_food` | 三态腐坏后的替换物 | 仅主模拟 | Wiki 待建；[替换](../../../raw/current/scripts/prefabs/meats.lua#L199-L199) |
| 关联 Prefab：`small_puff` | 作祟转换时在原坐标生成的烟尘 FX | 仅主模拟，转换前生成 | [[reference/prefabs/small_puff]]；[FX](../../../raw/current/scripts/prefabs/meats.lua#L108-L108) |
| 资源：`anim/meat.zip`／`anim/meat_rack_food.zip` | 生／熟用 `meat` bank 与 build，干制改用 `meat_rack_food` | 原版资源，不随页分发 | [assets](../../../raw/current/scripts/prefabs/meats.lua#L1-L15)、[干制动画名](../../../raw/current/scripts/prefabs/meats.lua#L367-L367) |

## 典型接入

### 在作祟转换的产物上挂钩

- 前提与选择：这是服务端 `modmain.lua` 的 postinit。转换函数是 `meats.lua` 的局部函数，模组既拿不到也改不了它的目标；但新件在转换末尾会推 `spawnedfromhaunt`，事件数据里带 `haunter` 与 `oldPrefab`，所以监听产物 prefab 就能拿到完整上下文。[事件](../../../raw/current/scripts/prefabs/meats.lua#L121-L122)
- 生效与撤销：只对"刚被作祟换出来"的实体触发，普通生成、读档恢复都不触发；监听器由 prefab 实例持有，卸载模组后不再注册，已存在的实体随存档按原版重建。`spawnedfromhaunt` 也可能由其它作祟路径推送，所以示例按 prefab 名收窄。

```lua
-- modmain.lua：记录（或给效果）普通肉被作祟换成怪物肉的那一次
local TARGETS = { "monstermeat", "cookedmonstermeat", "monstermeat_dried" }

local function OnMadeFromHaunt(inst, data)
    local old = data.oldPrefab
    print(string.format("[haunt] %s -> %s", old and old.prefab or "?", inst.prefab))
    -- 此处可改成给 data.haunter 加 debuff、给新件加标签等
end

for _, name in ipairs(TARGETS) do
    AddPrefabPostInit(name, function(inst)
        inst:ListenForEvent("spawnedfromhaunt", OnMadeFromHaunt)
    end)
end
```

- 调整与验证：`TARGETS` 必须覆盖实际会出现的产物（三态各自独立）；在控制台或日志里看 `[haunt]` 行，配合 `c_sel()` 的 `prefab` 与 `stackable:StackSize()` 确认整叠继承。注意监听不改变原版行为，转换概率仍由 `HAUNT_CHANCE_OCCASIONAL` 决定。

## 常见陷阱与限制

- **误区：干制会保留原料的新鲜度。** `OnDoneDrying` 只 `InheritMoisture`，没有 `SetPercent`，产物以满鲜度生成；晾晒期间 `preserver` 虽然对 `dryable` 物品返回 0，但普通雨暂停后回到常规腐坏，酸雨状态则按 `1 + rate` 加速（详见[晾肉架与干制](#晾肉架与干制)）。只有正常计时（且没有跨完成时刻的长更新）时产物才满鲜度；`LongUpdate` 跨过完成时刻会按剩余时间给产物扣鲜度。原版没有继承原料鲜度的入口，需要的话只能在产物生成后自行改写。
- **误区：`{isdried = true}` 是产物表。** 它不是产物，也不被 `common()` 读取；干制品没有 `dryable`／`cookable` 组件与标签，只多一个 `lureplant_bait`。因此干制品不能再上架、不能再烤。
- **边界：作祟换算整叠且不可逆。** 件数、湿度、鲜度都被继承，一件 20 叠的 `meat_dried` 会变成 20 个 `monstermeat_dried`；`goldvalue` 同时从 1 变 0，猪王不再收，`cookedmonstermeat`／`monstermeat_dried` 也没有 `cookable` 可供转回。
- **误区：怪物肉三态也会被作祟成别的。** 该反应只登记在普通肉三态；怪物肉三态被作祟只有普通抛飞。
- **未核验：`quagmire` 分支。** 它调用 `event_server_data("quagmire","prefabs/meats")`，而快照内检索不到 `quagmire_event_server/`，本页不描述该模式下的额外初始化。

## 验证记录

- 源码审阅：2026-11-09，Steam buildid 26474873。通读 `prefabs/meats.lua`（`common()`、三个大肉工厂、`AddMonsterMeatChange`、`OnSpawnedFromHaunt`、导出表），核对 `tuning.lua` 的时间基准与 `HEALING_*`／`CALORIES_*`／`SANITY_*`／`PERISH_*`／`DRY_*`／`STACK_SIZE_*`／`GOLD_VALUES`／`HAUNT_*`／`LAUNCH_SPEED_SMALL`，`edible`／`perishable`／`stackable`／`dryable`／`cookable`／`tradable`／`dryingrack`／`hauntable` 的相关分支，`standardcomponents` 的 `MakeInventoryFloatable`／`MakeHauntableLaunchAndPerish`／`AddHauntableCustomReaction`，`physics.Launch`，`cooking.lua` 的食材登记与别名，`pigking` 的接受与发金块、`pigshrine` 的供奉判定，`componentactions` 的 `COOK` 门禁与 `cookpot` 的 `stewer` 标签，`meatrack2` 与 `containers` 的架子筛选，以及 `beefalo`／`koalefant`／`pigman` 的掉落示例。
  名称核验：`chinese_s.po` 的 `NAMES.MEAT`＝肉、`NAMES.COOKEDMEAT`＝熟肉、`NAMES.MEAT_DRIED`＝肉干；三键各唯一命中。
  未覆盖：`quagmire` 模式；宿主的战斗掉落概率；模组注入与动态选名。
- 游戏运行：未运行。天数、秒数与金块数均由 tuning 与 `pigking` 分支推导（`total_day_time`＝480 秒）。
- 结构检查：见交付回报的 `validate_page.py` 结果。

## 相关链接

- 同批叶页：[[reference/prefabs/smallmeat]]、[[reference/prefabs/monstermeat]]、[[reference/prefabs/fishmeat|鱼肉两链]]
- 可食与鲜度：[[reference/components/edible]]、[[reference/components/perishable]]、[[reference/components/stackable]]
- 加工：[[reference/components/cookable]]、[[reference/components/dryable]]、[[reference/components/tradable]]
- 作祟与堆叠：[[reference/components/hauntable]]、[[reference/components/selfstacker]]

返回 [[reference/prefabs/index|Prefab 索引]]。
