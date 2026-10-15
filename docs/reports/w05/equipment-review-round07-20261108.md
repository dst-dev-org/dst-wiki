# 装备 Prefab 第七轮检视（2026-11-08）

范围为「背包收纳与重物搬运」叶索引的 9 页，源码为 Steam buildid 26335394。逐页核对构造、装备和容器回调、关键消费端、获取与清理路径。9 页均登记在本叶，页尾也均能返回本叶。装备初审累计 69／69 页。未运行游戏或校验脚本。 [叶索引](../../../wiki/reference/prefabs/storage-heavy-index.md) · [BUILD](../../../raw/current/BUILD.txt)

本轮需修复，共 5 项 P1、12 项 P2、1 项 P3。正文、摘要与索引重复出现的问题合并计数。

## P1 · 事实错误

- **R01｜保鲜背包里的冰在零下仍可停止腐坏。** `icepack` L20、40–41、69 把 ×0.5 写成包内实际腐坏速度。它是容器分支的基础倍率。之后若环境温度低于 0、物品带 `frozen` 且未启用 `frozenfiremult`，倍率会被改为 0。其他物品还可能受季节、湿度和局部倍率影响。同步索引 L20，把 `nocool` 的作用限定为跳过冰箱主动降温与热食 `AddChill`。 [保鲜背包](../../../wiki/reference/prefabs/icepack.md#L40-L41) · [陷阱](../../../wiki/reference/prefabs/icepack.md#L68-L69) · [索引](../../../wiki/reference/prefabs/storage-heavy-index.md#L20) · [容器基础倍率](../../../raw/current/scripts/components/perishable.lua#L68-L91) · [环境修正与热食分支](../../../raw/current/scripts/components/perishable.lua#L107-L146) · [常量](../../../raw/current/scripts/tuning.lua#L2077-L2088)
- **R02｜胡须袋容量为 1／2／3 格。** 叶索引 L22 把胡须位数门槛 1／3／9 写成格数。`pouch-bags` L32 的容量与 L45 的门槛各自正确，可直接用这两个概念改索引，并注明 BEARD 槽的例外。 [索引](../../../wiki/reference/prefabs/storage-heavy-index.md#L22) · [页面](../../../wiki/reference/prefabs/pouch-bags.md#L29-L45) · [格子布局](../../../raw/current/scripts/containers.lua#L1353-L1429) · [选袋门禁](../../../raw/current/scripts/components/beard.lua#L253-L264) · [门槛常量](../../../raw/current/scripts/tuning.lua#L7613-L7617)
- **R03｜月坛的初始入槽与后续补件是两条链。** `moon-altar-pieces` L7、41、60–61 把 idol／seed／ward 没有 `_socket_product` 写成没有对应祭坛。`_socket_product` 只决定月亮裂隙接受哪件并生成哪个祭坛。初始件是 glass／crown／icon。`moon_altar` 建好底座后按阶段接受 seed、再接受 idol，`moon_altar_astral` 接受 ward。成功补件由 `repairable` 消耗材料并推进阶段。建议同步摘要、机制、陷阱与叶索引 L30，写清两个动作分别查哪个字段。 [现文](../../../wiki/reference/prefabs/moon-altar-pieces.md#L37-L41) · [陷阱](../../../wiki/reference/prefabs/moon-altar-pieces.md#L59-L63) · [月亮裂隙入槽](../../../raw/current/scripts/prefabs/moon_fissure.lua#L69-L82) · [后续补件及阶段](../../../raw/current/scripts/prefabs/moon_altar.lua#L230-L269) · [材料消费](../../../raw/current/scripts/components/repairable.lua#L148-L182)
- **R04｜档案馆共鸣器先找标记，贡品存在时会阻止螃蟹之王兜底。** `moon-altar-pieces` L53 的「先沉水、其次岩石、再以 crown／cosmic 兜底」会误导查找结果。实际先查未发现的星界祭坛标记。无标记时，同一遍按登记顺序遍历，首个满足沉水条件或带 `boulder` 的对象胜出。两类之间没有全表优先级。再无目标时，crown／cosmic 的存在只用于决定是否跳过螃蟹之王搜索。 [现文](../../../wiki/reference/prefabs/moon-altar-pieces.md#L51-L55) · [查找顺序](../../../raw/current/scripts/prefabs/archive_resonator.lua#L119-L161)
- **R05｜重物落海后可能被移到岸上。** `heavy-items` L23、47、81、93 和 `moon-altar-pieces` L53 将落海与进入 `underwater_salvageable` 直接等同。`Submerge` 会看可达水域和近岸地形，可能换水点或走 `move_to_land`。`MakeSunken` 生成沉水实体后调用 `GiveItem`，随后发送 `on_submerge`，代码未检查 `GiveItem` 的返回值。疙瘩树果收到该事件时才开始或恢复生长计时。保留通用链，补足分流和生长前提。 [重物沉水](../../../wiki/reference/prefabs/heavy-items.md#L45-L49) · [月坛沉水](../../../wiki/reference/prefabs/moon-altar-pieces.md#L51-L55) · [落水门禁及选点](../../../raw/current/scripts/components/submersible.lua#L73-L103) · [上岸及沉水分流](../../../raw/current/scripts/components/submersible.lua#L136-L193) · [生长起表](../../../raw/current/scripts/prefabs/oceantreenut.lua#L90-L100)

## P2 · 边界与检索

- **R06｜胡须袋换档用新槽序重放，转移失败会掉地。** `pouch-bags` L22、45 的自动转移应注明 `RemoveAllItems` 返回压紧后的物品表，随后 `ipairs` 按新槽序交给 `GiveItem(item, slot, nil, true)`。缩容时超出容量的物品会走失败掉落，旧槽号也没有保留保证。 [现文](../../../wiki/reference/prefabs/pouch-bags.md#L45) · [换袋](../../../raw/current/scripts/components/beard.lua#L266-L287) · [提取内容](../../../raw/current/scripts/components/container.lua#L578-L591) · [槽位检查与失败掉落](../../../raw/current/scripts/components/container.lua#L269-L295) · [失败清理](../../../raw/current/scripts/components/container.lua#L557-L566)
- **R07｜土豆袋的拆解材料和锤毁掉落要分开。** `pouch-bags` L34 的草 2、石头 3 来自 `DeconstructRecipe`。L47 的「锤 2 下掉落」没有说明 `onhammered` 虽调用 `DropLoot`，工厂却没有设材料表，且 `GenerateLoot` 的配方返料查 `AllRecipes`。可写成锤满 2 工作量调用掉落流程、生成碎裂效果并移除，原版常规锤毁无已配置返料。 [页面](../../../wiki/reference/prefabs/pouch-bags.md#L44-L48) · [锤毁](../../../raw/current/scripts/prefabs/potatosack.lua#L18-L29) · [工厂](../../../raw/current/scripts/prefabs/potatosack.lua#L83-L93) · [掉落取配方](../../../raw/current/scripts/components/lootdropper.lua#L236-L256) · [拆解配方](../../../raw/current/scripts/recipes.lua#L1650)
- **R08｜巨型作物的蜡光任务是一次性延迟。** `giant-crops` L31、56 应把「每 20～40 秒」改为「相应入口安排 20～40 秒后播放一次」。`StartWaxTask` 使用 `DoTaskInTime`，`PlayWaxAnimation` 只播动画，既不清 `_waxtask` 也不续排。实例创建、落地、唤醒调用 Start，进入物品栏、睡眠调用 Cancel。 [现文](../../../wiki/reference/prefabs/giant-crops.md#L54-L60) · [任务](../../../raw/current/scripts/prefabs/veggies.lua#L311-L329) · [接线](../../../raw/current/scripts/prefabs/veggies.lua#L997-L1003)
- **R09｜`vigorbuff` 与大力士强壮态是两层速度修正。** `heavy-items` L43 把前者的部分抵消归到强壮态。装备读取在 `vigorbuff` 下加 0.25、上限 1，0.15 在这一层变成 0.40。locomotor 的 `mighty`／`mightiness_mighty` 分支再把低于 1 的装备倍率视为 1。0.15 应保留为基础倍率。猪皮包页已区分两层。 [重物速度](../../../wiki/reference/prefabs/heavy-items.md#L43-L45) · [猪皮包对照](../../../wiki/reference/prefabs/piggyback.md#L88-L91) · [装备倍率](../../../raw/current/scripts/components/equippable.lua#L126-L144) · [两端 locomotor](../../../raw/current/scripts/components/locomotor.lua#L186-L230)
- **R10｜重型障碍的延迟由落地事件触发，半径任务有终点。** `heavy-items` L39 的「卸下或丢地」应收敛到 `ondropped`，单独 `Unequip` 不等于落地。回障碍态时先立即检查一次半径，之后每 0.5 秒复查，半径达到上限即停。大障碍进入 `OBSTACLES`，月坛部件使用 `MakeSmallObstacle` 分支的 `SMALLOBSTACLES`。若重物有 falling 接线，`stopfalling` 会直接走回障碍态，不等这 0.5 秒。 [现文](../../../wiki/reference/prefabs/heavy-items.md#L37-L41) · [卸装](../../../raw/current/scripts/components/inventory.lua#L1219-L1247) · [碰撞和半径任务](../../../raw/current/scripts/components/heavyobstaclephysics.lua#L60-L125) · [停止坠落](../../../raw/current/scripts/components/heavyobstaclephysics.lua#L149-L159) · [落地监听](../../../raw/current/scripts/components/heavyobstaclephysics.lua#L225-L240) · [月坛小障碍配置](../../../raw/current/scripts/prefabs/moon_altar_pieces.lua#L57-L61)
- **R11｜疙瘩树果避树位置是尽力选取，受阻重试会随睡眠暂停。** `heavy-items` L63 的「试 5 个空位避开」应写成至多检查 5 个候选，找不到无树点也仍使用最后检查的位置。L67、95 的 60 秒是重试任务周期，睡眠时会取消任务，唤醒时按标记恢复。 [撞落与重试现文](../../../wiki/reference/prefabs/heavy-items.md#L61-L69) · [选点回退](../../../raw/current/scripts/prefabs/watertree_pillar.lua#L137-L153) · [睡眠与唤醒](../../../raw/current/scripts/prefabs/oceantreenut.lua#L113-L136)
- **R12｜熊獾找的是可食用的内容物。** `icepack` L43、71 与 `krampus_sack` L42 仅以非空为条件。`backpack` 分支还调用熊獾 `eater:CanEat`，找出可食物件，再对蜂蜜类目标优先使用 `STEAL`。没有可食物件的非空背包不满足该分支。 [保鲜背包](../../../wiki/reference/prefabs/icepack.md#L68-L72) · [坎普斯背包](../../../wiki/reference/prefabs/krampus_sack.md#L42) · [筛选与动作](../../../raw/current/scripts/brains/beargerbrain.lua#L104-L166)
- **R13｜共享掉落表的非必掉数值是基础概率。** `krampus_sack` L44、55、58、75 和叶索引 L19 的坎普斯 1%／示例 10%，以及 `heavy-items` L71、91 的疙瘩树果 50%／1%，都由 `LootDropper:GetChance` 处理，可按有效击杀者或最后工作者运势修正。克劳斯袋的 10% 走 `math.random()`，保留独立说明。 [坎普斯掉落与示例](../../../wiki/reference/prefabs/krampus_sack.md#L44-L68) · [索引](../../../wiki/reference/prefabs/storage-heavy-index.md#L19) · [疙瘩树果掉落](../../../wiki/reference/prefabs/heavy-items.md#L90-L92) · [共享表运势入口](../../../raw/current/scripts/components/lootdropper.lua#L163-L183) · [掷点](../../../raw/current/scripts/components/lootdropper.lua#L213-L225) · [克劳斯袋](../../../raw/current/scripts/components/klaussackloot.lua#L107-L113)
- **R14｜分开作祟与漂浮的初始化端，注明移速由哪份 Replica 读取。** `icepack` L44、`candybag` L55、`krampus_sack` L43 的「公共端 helper」同时覆盖作祟与漂浮，但只有漂浮在 `SetPristine` 前。作祟 helper 在主模拟分支。`piggyback` L38、43 可把移速同步落到 `inventoryitem` Replica，而非让读者在 `equippable` Replica 查找客户端读取接口。 [保鲜背包表](../../../wiki/reference/prefabs/icepack.md#L43-L45) · [糖果袋表](../../../wiki/reference/prefabs/candybag.md#L54-L56) · [坎普斯背包表](../../../wiki/reference/prefabs/krampus_sack.md#L42-L44) · [猪皮包表](../../../wiki/reference/prefabs/piggyback.md#L37-L43) · [保鲜背包构造](../../../raw/current/scripts/prefabs/icepack.lua#L55-L87) · [糖果袋构造](../../../raw/current/scripts/prefabs/candybag.lua#L67-L98) · [坎普斯背包构造](../../../raw/current/scripts/prefabs/krampus_sack.lua#L52-L85) · [移速同步](../../../raw/current/scripts/components/equippable.lua#L7-L13) · [客户端读取](../../../raw/current/scripts/components/locomotor.lua#L219-L229)
- **R15｜区分给当前实例加标签与注册构造回调。** `icepack` L53 说示例对已存在的背包也同样有效。已经存在的实例要实际执行 `AddTag` 才会改变后续更新。示例 `AddPrefabPostInit` 只作用于此后完成构造的实例，包括重新载入时重建的实例，不会就地扫描已有对象。 [现文与示例](../../../wiki/reference/prefabs/icepack.md#L50-L62) · [Prefab 后置回调登记](../../../raw/current/scripts/modutil.lua#L593) · [构造调用](../../../raw/current/scripts/mainfunctions.lua#L385-L407)
- **R16｜糖果袋的活动解锁也支持额外事件。** `candybag` L42、89 的开关只写了主 `specialevent`。世界生成和加载都可通过对应事件的 `enabled` 设置调用 `ApplyExtraEvent`，同样把该科技档置 0。可顺手把 L88 的结论收回到已核验的装备回调与检索范围。 [现文](../../../wiki/reference/prefabs/candybag.md#L37-L44) · [世界生成](../../../raw/current/scripts/map/forest_map.lua#L455-L460) · [读档](../../../raw/current/scripts/gamelogic.lua#L364-L373) · [额外事件](../../../raw/current/scripts/simutil.lua#L649-L659) · [装备回调](../../../raw/current/scripts/prefabs/candybag.lua#L12-L16)
- **R17｜挖取章节使用游戏名，并补该名的来源锚点。** 标题、首页和叶索引已经能查到「吸引人的结构」。`moon-altar-pieces` L45 仍用「岩石包裹与挖取」，现有名称锚点只证部件。建议章节以「吸引人的结构」开头，给三个岩石键加 `.po` 锚点。若继续保留「岩石包裹」，按旧检索词保留即可，现有证据没有确认它是玩家俗称。 [章节](../../../wiki/reference/prefabs/moon-altar-pieces.md#L45-L49) · [索引](../../../wiki/reference/prefabs/storage-heavy-index.md#L30) · [简中名称](../../../raw/current/scripts/languages/chinese_s.po#L345790-L345802)

## P3 · 验证记录

- **R18｜验证记录中的档位数字已过时。** `heavy-items` L102、`giant-crops` L78、`moon-altar-pieces` L69 将 standard 写为 480 行／24 KiB。`icepack` L79、`krampus_sack` L82、`pouch-bags` L91 仍写 24 KiB。当前标准档为 240 行／32 KiB。同步修改时可直接保留有日期的实测记录，省去档位上限，收口时再统一校验。 [重物页记录](../../../wiki/reference/prefabs/heavy-items.md#L98-L102) · [巨型作物记录](../../../wiki/reference/prefabs/giant-crops.md#L74-L78) · [月坛记录](../../../wiki/reference/prefabs/moon-altar-pieces.md#L65-L69)

## 逐页结论

| 页面 | 结论 |
|---|---|
| [背包](../../../wiki/reference/prefabs/backpack.md) | 未发现 P1／P2，保留现有皮肤 FX 关联入口。 |
| [猪皮包](../../../wiki/reference/prefabs/piggyback.md) | R14。 |
| [坎普斯背包](../../../wiki/reference/prefabs/krampus_sack.md) | R12–R14、R18。 |
| [保鲜背包](../../../wiki/reference/prefabs/icepack.md) | R01、R12、R14–R15、R18。 |
| [糖果袋](../../../wiki/reference/prefabs/candybag.md) | R14、R16。 |
| [小袋族](../../../wiki/reference/prefabs/pouch-bags.md) | 页面 R06–R07、R18，索引容量 R02。 |
| [重型障碍持握族](../../../wiki/reference/prefabs/heavy-items.md) | R05、R09–R11、R13、R18。 |
| [巨型作物](../../../wiki/reference/prefabs/giant-crops.md) | R08、R18。 |
| [天体祭坛部件与吸引人的结构](../../../wiki/reference/prefabs/moon-altar-pieces.md) | R03–R05、R17–R18。 |

## 处置结果（2026-11-08）

18 项逐条对照 `raw/current` 源码复核：17 项成立并采纳，**R07 部分驳回**（见下），改 8 页＋收纳叶索引（10 文件）。

- **R01 采纳**：`icepack` 的 0.5 改称「容器分支**基础倍率**」，并补后续修正（`<0` 且 `frozen` 无 `frozenfiremult` → 直接 0；湿润 ×1.3、非冻结冬 ×0.75、过热 ×1.25）；`nocool` 限定为「跳过冰箱主动降温与热食 `AddChill`」，不再写成"包内始终 0.5"；摘要与索引同步。
- **R02 采纳**：索引把胡须袋容量改为 **1/2/3 格**（`ui_beard_1x1/2x1/3x1`），1/3/9 是胡须位数门槛，注明 BEARD 槽例外。
- **R03 采纳**：`_socket_product` 只决定裂隙接受哪件并生成哪个祭坛；`moon_altar` 按 stage 收 seed→idol、`moon_altar_astral` 收 ward 是 `checkmaterialfn` 的另一条链。摘要、机制与陷阱都改成"两条链"，删「idol／seed／ward 没有对应祭坛」。
- **R04 采纳**：检索顺序改为「先找未发现的 `moon_altar_astral_marker`；无标记时按登记顺序同一遍遍历，首个满足沉水或 `boulder` 的胜出（两类无全表优先级）；仍无目标再查 crown／cosmic，但它们只决定是否跳过螃蟹之王兜底」。
- **R05 采纳**：`Submerge` 可能换水点或走 `move_to_land` 上岸，落海**不保证沉水**；`MakeSunken` 的 `GiveItem` 返回值未检查，`on_submerge` 在其后才推。重物页与月坛页同步。
- **R06 采纳**：胡须袋换档写明 `RemoveAllItems` 得到**压紧**物品表、以压紧序号当目标槽 `GiveItem(item, slot, nil, true)`，缩容时超容量物品走 `drop_on_fail` 掉地、旧槽号不保证保留。
- **R07 部分驳回**：锤毁返料不是"无已配置返料"。`LootDropper` 构造即 `droprecipeloot = true`，`AllRecipes["potatosack"]` 就是那条 `DeconstructRecipe`（`Recipe._ctor` 无条件登记），`GetRecipeLoot` 按 `HAMMER_LOOT_PERCENT=0.5` 向上取整 → 锤毁实际掉 **草 1＋石头 2**；`DeconstructRecipe` 的草 2＋石头 3 是拆解法杖产出。页面按这两条写，并补「先灭火再掉落」。
- **R08 采纳**：蜡光任务改为**一次性延迟**——创建／`ondropped`／`OnEntityWake` 各排一个 20～40 秒 `DoTaskInTime`，到点只播动画、不清 `_waxtask` 也不续排，要等 `CancelWaxTask`（入包／睡眠）清空后才可能重排。
- **R09 采纳**：速度拆成两层——装备层 `vigorbuff` 把 <1 的倍率 +0.25（0.15→0.40），locomotor 层强壮态把 <1 的装备倍率当 1；0.15 保留为基础倍率。
- **R10 采纳**：回障碍态由 `ondropped` 触发（单独 `Unequip` 不算），并补碰撞组（`OBSTACLES`／`SMALLOBSTACLES`）、半径任务到 `maxradius` 即停、`stopfalling` 直接回障碍态不等 0.5 秒。
- **R11 采纳**：撞落改为"至多检查 5 个候选，找不到无树点也接受最后一次随机位置"；60 秒重试任务睡眠取消、唤醒按标记恢复。
- **R12 采纳**：熊獾只挑"非空且 `eater:CanEat` 找得到内容物"的背包，蜂蜜类优先 `STEAL`。
- **R13 采纳**：坎普斯共享表 1% 与示例 10%、疙瘩树果 twigs 50%／1% 都经 `GetChance`（按有效击杀者/最后工作者幸运修正）；克劳斯袋 10% 走 `math.random()`，单独标注。
- **R14 采纳**：漂浮（`MakeInventoryFloatable`）在 `SetPristine` 前／公共端，作祟 helper 在主模拟；猪皮包减速落到 `inventoryitem_replica`（`classified.walkspeedmult`），客户端读 `inventoryitem:GetWalkSpeedMult()`，不是 `equippable` Replica。
- **R15 采纳**：`AddPrefabPostInit` 只接管此后完成构造的实例（含重载重建），不会就地扫描世界里已有背包；要让现存实例生效须自己 `AddTag`。
- **R16 采纳**：生成与读档两处都会遍历 `SPECIAL_EVENTS` 对 `enabled` 的额外事件调 `ApplyExtraEvent`；"穿戴不收集"的结论收回到已核验的回调与检索范围；`ApplyEvent` 只有置 0 一条路径，恢复实现未核到。
- **R17 采纳**：章节改为「吸引人的结构（旧检索词"岩石包裹"）与挖取」，补三个 `MOON_ALTAR_ROCK_*` 的 .po 锚点，并说明"岩石包裹"未经证据确认为玩家俗称。
- **R18 采纳**：六页验证记录去掉过时的 24 KiB／480 行写法，统一为当前 standard 档（240 行／32 KiB），不再逐次记字节数。

校验：10 文件 `validate_page.py --check-indexes` 通过、导航 0 问题；`validate_source_links.py --strict` 407 条引用／72 源文件／0 error 0 warning；9 页 `record` 全部 already_recorded、0 changed。未运行游戏、未改 raw、未提交推送。

## 复核补正（2026-11-08，用户复核 3 组）

- **R05 生长小节**：`heavy-items` 的「果落海后走 submersible 链进入 `underwater_salvageable`，推 `on_submerge`」改为「只有在**真的沉水**、收到 `on_submerge` 之后才起表（被换水点或挪上岸时不发该事件），`OnSubmerge` 新建或恢复 `grow` 计时」，与同页沉水分流段一致。
- **R01／R12**：`icepack` 摘要与索引的「零下停腐」补上 `frozenfiremult` 门禁（环境 <0＋`frozen`＋未开 `frozenfiremult` 才改 0），"不会被降温"收窄为"跳过冰箱的主动降温"；L71 的熊獾陷阱改「有可食物件就会成为 `STEAL` 目标，蜂蜜类只是先占 `honeyed_backpack` 分支、在动作优先级上压过 `harvest`/`hammer`」，删「只有蜂蜜类才给 STEAL」。
- **R03／R17**：`moon-altar-pieces` 补件量写明「每件 `workrepairvalue = MOON_ALTAR_COMPLETE_WORK/3 = 1`，给 `workable.workleft` 加 1 点；普通祭坛满值 3＝一件 1/3，星界祭坛满值 2＝一件 1/2」；补两条链的源码锚点（`moon_fissure.lua#L69-L83` 入槽、`moon_altar.lua#L234-L270` 补件 checkmaterialfn、`repairable.lua#L117-L123` workleft 加值）；岩石译名锚点延到 `chinese_s.po#L345789-L345802`，覆盖第三条简中译文。

校验：10 文件 `validate_page.py --check-indexes` 通过、导航 0 问题；`validate_source_links.py --strict` 410 条引用／75 源文件／0 error 0 warning；9 页 `record` 全部已记录、0 changed。

补记 2（用户复核 3 组）：

- heavy-items 生长段：删「换水点时不发该事件」——Submerge 被挤时只有走 move_to_land（挪上岸）才跳过沉水；换到附近可用水点后仍会 MakeSunken 并发 on_submerge。同时写清 576～1296 秒只在**新建**计时时抽取，已有 grow 计时则 ResumeTimer 沿用剩余时间。
- icepack：快速判断的「物品和热食都不会被降温」改为「跳过冰箱的主动降温（物品不被压向冰点、热食不 AddChill）」；nocool 机制表把零下停腐条件写全（环境 <0＋frozen＋未开 frozenfiremult），与摘要一致。
- moon-altar-pieces：sources 补 prefabs/moon_fissure.lua、prefabs/moon_altar.lua 并 record，source-baseline.json 该页来源由 15 增至 17。

校验：9 页＋索引 validate_page.py --check-indexes 通过、导航 0 问题；strict 410 条引用／75 源文件／0 error 0 warning；record 后 9 页 0 changed（moon-altar-pieces 已刷新）。
