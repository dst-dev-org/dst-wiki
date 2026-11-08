# B11 动物掉落与巢材上外部源码检视

**当前结论为外部源码复验通过，可登记收口。** 第三轮定向复验确认 R01、R04 通过，R01–R13 累计 13 项全部通过。[第三轮外部定向复验](#第三轮外部定向复验)

初检有 P1 1 项、P2 11 项、P3 1 项。

2026-11-10，Steam 安装清单与 [源码快照](../../../raw/current/BUILD.txt) 均为 BUILD `26474873`。范围为 [B11 台账](prefab-inventory-ledger-20261108.md) 的 15 页、[动物掉落与巢材索引](../../../wiki/reference/prefabs/animal-drops-index.md) 和直接相关的跨页断言。Wiki 行号对应初检工作树。

## P1

| 编号 | 位置 | 问题、源码证据和修订验收 |
|---|---|---|
| R01 | [邪天翁羽毛与羽毛帆布](../../../wiki/reference/prefabs/malbatross-feather.md) L89–107 | 示例包裹 `inst.spawnfeather`，并承诺受击、撞桅都增产。受击回调经实例字段调用，撞桅直接调用文件局部函数 `spawnfeather(inst,0.4)`，绕过这个包裹。把示例和验收限定为「每次受击掉毛判定命中时生成两根」，四次概率判定仍各自照常。验收对照受击只命中一次（原版一根、示例两根）与一次满足门禁的撞桅（仍为三根保底、两次额外判定）。标题、前提、代码注释和调整说明一起改。[受击调用](../../../raw/current/scripts/prefabs/malbatross.lua#L133-L147)、[撞桅调用](../../../raw/current/scripts/prefabs/malbatross.lua#L261-L283)、[实例字段绑定](../../../raw/current/scripts/prefabs/malbatross.lua#L550-L556) |

## P2

| 编号 | 位置 | 问题、源码证据和修订验收 |
|---|---|---|
| R02 | [邪天翁羽毛与羽毛帆布](../../../wiki/reference/prefabs/malbatross-feather.md) L6、L23、L58、L83、L111–116；[索引](../../../wiki/reference/prefabs/animal-drops-index.md) L24 | 「仅当带 `mast` 标签」和「每次撞桅」漏了有效对象、未在最近碰撞表、未先命中可直接击杀的 `smashable` 分支、目标可做工且动作非 `NET`。同一目标结算后还有三秒去重。3＋2×0.3 只适用于命中该分支的碰撞。导航也不能把邪天翁写作唯一来源，正文已经列弃船宝箱。验收用一个只有 `mast` 标签、不可做工的目标作反例，并在导航区分宿主产出和宝箱产出。[碰撞门禁、结算与去重](../../../raw/current/scripts/prefabs/malbatross.lua#L248-L301)、[弃船宝箱](../../../raw/current/scripts/scenarios/chest_abandonedboat.lua#L29-L35) |
| R03 | [蜜脾](../../../wiki/reference/prefabs/honeycomb.md) L51、L55–56；[蜂蜜与蜂王浆](../../../wiki/reference/prefabs/honey.md) L66；[索引](../../../wiki/reference/prefabs/animal-drops-index.md) L25 | 「满耐久才拿到 4 个蜜脾」与向上取整相反。蜂王冠 80% 耐久时 `ceil(4×0.8)=4`，75% 才降为 3。通用拆解循环有每种正数量原料至少返一件的下限，但原版蜂王冠到 0 耐久会移除，不能用 0% 蜂王冠作正常可施法案例。蜂蜜页和索引把 4／3／2／4 标作满耐久或表值，实际按各材料独立折算。[拆解表](../../../raw/current/scripts/recipes.lua#L1673)、[取护甲比例](../../../raw/current/scripts/prefabs/staff_common.lua#L637-L649)、[生成数量](../../../raw/current/scripts/prefabs/staff_common.lua#L663-L674)、[冠的护甲](../../../raw/current/scripts/prefabs/hats.lua#L2076-L2087)、[耐久耗尽](../../../raw/current/scripts/components/armor.lua#L93-L112) |
| R04 | [蜜脾](../../../wiki/reference/prefabs/honeycomb.md) L57、L92；[蜂蜜与蜂王浆](../../../wiki/reference/prefabs/honey.md) L65；[索引](../../../wiki/reference/prefabs/animal-drops-index.md) L25 | 「满耐久才有 1%」把公式的理论上界当成可达的普通敲击概率。`WorkedBy_Internal` 先扣工作量，再把剩余值传给 `OnWorked`。从 16 开始且扣 1 的首锤，以 15/16 计算，未触发召蜂王时蜜脾概率为 0.9375%。写清用本击扣减后的剩余量，保留存活非幽灵玩家、召唤成功则提前返回，以及 0 时不给蜜脾的条件。[扣减顺序](../../../raw/current/scripts/components/workable.lua#L130-L153)、[产出与提前返回](../../../raw/current/scripts/prefabs/beequeenhive.lua#L188-L213)、[初始工作量](../../../raw/current/scripts/prefabs/beequeenhive.lua#L492-L497)、[常量](../../../raw/current/scripts/tuning.lua#L2575-L2576) |
| R05 | [四色鸟羽](../../../wiki/reference/prefabs/bird-feathers.md) L6、L66–89、L108、L145；[索引](../../../wiki/reference/prefabs/animal-drops-index.md) L22 | 按四色原料名查 `recipes.lua`，有 16 条不同的 `Recipe2` 和 1 条 `DeconstructRecipe`。现有表有 18 行，是因为羽毛帽同时使用黑羽和红羽，在按颜色展开时出现两次。拆解行是返还红羽，方向与原料消费相反。验收让摘要、导航、索引和验证记录分别写清「16 条制作、1 条拆解、表按颜色展开为 18 行」。[同配方两种羽毛](../../../raw/current/scripts/recipes.lua#L544)、[拆解表](../../../raw/current/scripts/recipes.lua#L1741) |
| R06 | [四色鸟羽](../../../wiki/reference/prefabs/bird-feathers.md) L63 | 落叶堆 DEBRIS 的「表内单抽」只适用于没增加掉落次数的情形。DEBRIS 与 TREAT 都从 1 次起，后续按堆的阶段尝试增加次数，每次回到同一表抽，未移除上次选项。11.1% 是 DEBRIS 下每一抽命中黑羽的概率。验收对照一抽与已经获得两抽的同一池，两抽时至少一根为 `1-(8/9)^2`，且同次可出两根。[碎屑池](../../../raw/current/scripts/prefabs/leafpile.lua#L97-L108)、[阶段加抽与抽取循环](../../../raw/current/scripts/prefabs/leafpile.lua#L164-L211) |
| R07 | [海象牙](../../../wiki/reference/prefabs/walrus-tusk.md) L6、L23、L45–51、L59–62、L72；[索引](../../../wiki/reference/prefabs/animal-drops-index.md) L29 | 「唯一来源／要牙只能打海象」混淆新增掉落和材料回收。成年海象的 0.50 是共享战利品基础概率。拆解法杖从目标的 `AllRecipes` 读取原料，符合拆解条件的步行手杖、刷子可返 1 牙，回溯怀表还要过 `pocketwatch_inactive`。验收把宿主掉落与拆解回收分开，保留小海象和营地无掉落的结论。[海象表](../../../raw/current/scripts/prefabs/walrus.lua#L23-L33)、[回溯怀表与门禁](../../../raw/current/scripts/recipes.lua#L360-L368)、[手杖](../../../raw/current/scripts/recipes.lua#L552)、[刷子](../../../raw/current/scripts/recipes.lua#L685)、[拆解验收与产出](../../../raw/current/scripts/prefabs/staff_common.lua#L637-L674) |
| R08 | [皮弗娄牛](../../../wiki/reference/prefabs/beefalo.md) L23、L69–76、L95–108；[牛毛](../../../wiki/reference/prefabs/beefalowool.md) L23、L42–44、L79；[索引](../../../wiki/reference/prefabs/animal-drops-index.md) L17–18 | 成牛「死亡保底 3 份／死亡时仍在烧就转灰」需要限定到实际掉落路径。充能暗影铃保尸走熄火分支，根本不调用 `DropDeathLoot`。骑乘中死亡还会等甩下骑手才进死亡状态，燃烧改写应读 `DropLoot` 结算当下。L76 的「先 `GenerateLoot` 再 `DropLoot`」也把通用函数的尸体暂存与直接掉落两个分支串成一条。验收对照正常掉落、实际掉落时仍燃烧、保尸三例，统一索引与牛毛表的条件。[被骑乘死亡](../../../raw/current/scripts/prefabs/beefalo.lua#L671-L692)、[死亡状态分流](../../../raw/current/scripts/stategraphs/SGBeefalo.lua#L628-L655)、[DropDeathLoot](../../../raw/current/scripts/entityscript.lua#L2181-L2192)、[燃烧改写](../../../raw/current/scripts/components/lootdropper.lua#L415-L445) |
| R09 | [毛丛](../../../wiki/reference/prefabs/furtuft.md) L6、L21–23、L50–53、L100；[索引](../../../wiki/reference/prefabs/animal-drops-index.md) L20 | 快速判断「来源只有掉毛链」与正文的尸体换算冲突，「尸体不再掉毛」需限定为死亡回调停掉周期任务。尸体换算另要求 `meat_level >= 2` 且原战利品表有多件，单件分支只判是否清空。25–35 秒是消失序列的触发时间，醒着的毛丛禁拾取后等动画或兜底任务，休眠直接删。验收分别写周期掉毛、含熊皮的多件尸体掉落和计时到点后的实体状态。[死亡停表](../../../raw/current/scripts/prefabs/bearger.lua#L335-L339)、[熊皮替换表](../../../raw/current/scripts/prefabs/corpses_defs.lua#L1467-L1472)、[尸体门禁和数量](../../../raw/current/scripts/prefabs/corpses.lua#L465-L493)、[消失序列与计时](../../../raw/current/scripts/components/disappears.lua#L12-L77) |
| R10 | [猫尾](../../../wiki/reference/prefabs/coontail.md) L63、L103；[索引](../../../wiki/reference/prefabs/animal-drops-index.md) L19 | 「换 48 秒忠诚」把参数写成实得时间。交付者有 `leader` 组件且并非浣猫当前的攻击目标时，才进入忠诚分支，`AddLoyaltyTime` 可能乘当前 leader 的倍率，并把剩余时间夹到 480 秒。验收用原剩余 470 秒、倍率 1 的例子说明只增加 10 秒。仍保留当前攻击目标给猫尾会清目标、物品消耗而不加忠诚的分流。[给物分支](../../../raw/current/scripts/prefabs/catcoon.lua#L281-L302)、[倍率与上限](../../../raw/current/scripts/components/follower.lua#L365-L379)、[忠诚常量](../../../raw/current/scripts/tuning.lua#L2969-L2973) |
| R11 | [蜜脾](../../../wiki/reference/prefabs/honeycomb.md) L50、L54、L93；[针刺](../../../wiki/reference/prefabs/stinger.md) L6、L48–49、L81、L92；[海象牙](../../../wiki/reference/prefabs/walrus-tusk.md) L45、L49、L80；[索引](../../../wiki/reference/prefabs/animal-drops-index.md) L25、L27、L29 | 同一掉落组件的概率口径应一致。共享表小于 1 的条目和 `AddChanceLoot` 的概率项先走 `GetChance`，再与随机值比较。蜂王蜜脾的期望 1.5、嗡嗡蜜蜂 1%、海象牙 50% 都应标成无幸运修正的基准。1.00 的保底仍走必中分支。验收对照已经标清幸运口径的猫尾页或麋鹿鹅页，不给整张表套同一个最终概率。[幸运解析](../../../raw/current/scripts/components/lootdropper.lua#L159-L184)、[两类概率项](../../../raw/current/scripts/components/lootdropper.lua#L202-L224)、[嗡嗡蜜蜂挂表](../../../raw/current/scripts/prefabs/beeguard.lua#L511-L514) |
| R12 | [针刺](../../../wiki/reference/prefabs/stinger.md) L69；对照 [蜜脾](../../../wiki/reference/prefabs/honeycomb.md) L63–68 | 针刺页用「没有 `edible`，所以没有锅料登记」解释，因果和同批蜜脾页冲突。普通锅按注册名查 `IsCookingIngredient`，不查 `edible`。本快照 `rg -n '\bstinger\b' raw/current/scripts/cooking.lua` 无命中，针刺未登记，与能否直接吃是两个独立结论。验收以无 `edible` 却已登记的蜜脾作反例，分别给针刺无食用组件和无锅料登记的证据。[蜜脾登记](../../../raw/current/scripts/cooking.lua#L97)、[注册名查表](../../../raw/current/scripts/cooking.lua#L185-L187)、[锅的过滤](../../../raw/current/scripts/containers.lua#L373-L375)、[针刺挂件](../../../raw/current/scripts/prefabs/stinger.lua#L27-L45) |

## P3

| 编号 | 位置 | 问题、证据和修订验收 |
|---|---|---|
| R13 | [毛丛](../../../wiki/reference/prefabs/furtuft.md) 摘要；[索引](../../../wiki/reference/prefabs/animal-drops-index.md) L20 | 摘要叫「毛簇」，标题、名称核验和索引均用原版简中「毛丛」。摘要用同一实显名。[简中名称](../../../raw/current/scripts/languages/chinese_s.po#L343084-L343087) |

内部复验非阻塞残留（不影响收口，批末统一润色）：`malbatross-feather` L107「让攻击只命中一次」宜改「4 次 0.05 判定里恰好命中 1 次」；同页 L58 门禁一可补「是 tree／mast 或有 health」；索引 L22 别名列可补「金丝雀羽毛、鸟羽」；`beefalo` L23／L77 与 `beefalowool` L23 的「保底」宜加「未燃烧时」并补 `componentutil#L1546-L1548` 一处锚点。

## 已核验范围

| 同版源码事实骨架 | 页面命中情况 |
|---|---|
| 胡须、兔绒分别核对兔子和兔人的战利品分流，再看兔王召唤兔人的共享权重池、伯尼空燃料计时、剃须掉奖和眼面具食性 | 普通兔人 3/3/2、兔王召唤兔人 3/16 胡须与 4/16 兔绒、伯尼三件分别有出处。 |
| 牛毛、皮弗娄牛核对刷毛上限、剃毛睡眠和牛铃门禁、SG 真正恢复毛发的时点、普通死亡与保尸 | 三份剃毛奖品与可刷标签恢复分开记。跨天刷毛的除零结果仍属推导，未运行验证。掉落状态门禁见 R08。 |
| 猫尾、毛丛、猪皮分别核对猫玩具的交易与地面拾取、熊獾掉毛与尸体回收、猪人的普通和变体表与眼面具食性入口 | 三页都给了对应宿主和消费者。忠诚的上限、尸体转换及消失时序见 R09–R10。 |
| 四色鸟羽、麋鹿鹅羽毛、邪天翁羽毛与帆布核对鸟羽注册、金丝雀加权、麋鹿鹅共享表、克劳斯袋、弃船箱和邪天翁的两个调用点 | 金丝雀 1/11、克劳斯袋 2/9、宝箱整堆同种的口径可保留。调用覆盖、落叶堆抽取、配方方向和计数见 R01–R02、R05–R06。 |
| 蜜脾与针刺核对蜂类掉落、长成巢锤击、绿杖拆解、熊獾实际取食筛选和锅按名接收 | 蜜脾能入锅但无 `edible`，蜜蜂与杀人蜂同用针刺 5／蜂蜜 1 的权重池。触发条件和概率口径见 R03–R04、R11–R12。 |
| 犬牙、触手皮、海象牙核对猎犬丘死亡、嚎弹炮弹舱、触手和大触手同表的概率及 `overtaken` 门禁、海象家族战利品和配方消费 | 主注册和上述入口均有证据。回收与基础概率口径见 R07、R11。 |

同 BUILD 的本地运行时导出 `output/coverage/prefab-26474873-shard-1-aligned/registrations.json` 命中全部 18 个物品主注册与 1 个宿主 `beefalo`。当前 Prefab 页精确别名无重复归属，`beefalo_carry` 与 `malbatross_feather_fall` 分开识别为辅助载体。跨页核对含叶索引、蜂蜜及动物食物索引本批回填、远程武器和触手尖刺的直接关联断言。按上表所列范围追到消费者或回调，其他配方、掉落来源和旧页章节未逐条穷举。严格源码链接检查涵盖 15 页及 4 个直接相关页面和索引，共 1,194 处引用，0 错误、0 警告。15 个页面的 264 项源码哈希与基线一致。未运行游戏，未实测掉率、刷新量或客户端表现。

## 修订回执与定向复验

实际写作者按 R01–R13 回填改动位置、自验、同事实的正文复述及涉及索引或关联页。外部复验续记在本报告。

| 编号 | 写作修订回执 | 交付方复验记录 |
|---|---|---|
| R01 | `malbatross-feather` L89–107：示例收窄为「每次受击掉毛判定命中时掉两根」，前提写明撞桅走文件局部 `spawnfeather(inst,0.4)`、绕过实例字段；标题／前提／代码注释／调整说明四处同步；双验收（受击命中一次 1→2 根；合格撞桅仍 3 根保底＋2×30%）。[受击](../../../raw/current/scripts/prefabs/malbatross.lua#L143-L147)、[撞桅](../../../raw/current/scripts/prefabs/malbatross.lua#L275-L283)、[绑定](../../../raw/current/scripts/prefabs/malbatross.lua#L555)；第二轮：L107 验收改「4 次 0.05 判定里恰好命中 1 次」并补「一次攻击仍掷满 4 次」；L89 标题去掉「受击」；L125 记录同步。；第三轮：L89 标题恢复「受击掉毛判定」，示例范围回到受击路径。 | 通过（2026-11-10 第三轮外部复验）。 |
| R02 | `malbatross-feather` L6／L23／L58／L83／L111／L116：撞桅补四道门禁（有效且不在 `recentlycharged` → 非 `smashable`+health 直接击杀 → `workable`／`CanBeWorked`／动作非 `NET` → 最后 `mast` 标签）与结算后 3 秒去重；3＋2×0.3 只适用于命中该分支；导航改「宿主侧产出，不是全库唯一来源」；反例「只带 `mast`、无 `workable`」一根不掉。[门禁](../../../raw/current/scripts/prefabs/malbatross.lua#L252-L263)、[结算](../../../raw/current/scripts/prefabs/malbatross.lua#L265-L290)、[清除](../../../raw/current/scripts/prefabs/malbatross.lua#L248-L249) | 通过（2026-11-10 外部复验）。 |
| R03 | `honeycomb` L51／L55、`honey` L66：删「满耐久才 4 个」，改为拆解表原料 4 脾＋3 蜜＋2 蜂王浆＋4 蜜蜂、各材料按护甲耐久独立向上取整（80% 仍 `ceil(3.2)=4`、75% 起 3）；补护甲归零会 `Remove()`、0% 冠不是可施法案例。[比例](../../../raw/current/scripts/prefabs/staff_common.lua#L644-L649)、[生成](../../../raw/current/scripts/prefabs/staff_common.lua#L667-L672)、[耗尽](../../../raw/current/scripts/components/armor.lua#L101-L112) | 通过（2026-11-10 外部复验）。 |
| R04 | `honeycomb` L57／L92／L93、`honey` L65：`workleft` 写明是「本击扣减后的剩余量（首锤 16→15）」，首锤 0.01×15/16＝0.9375%；归零时 `lootscale=0` 直接不给；召唤成功提前 return、执行者须存活非幽灵。[扣减](../../../raw/current/scripts/components/workable.lua#L130-L153)、[产出](../../../raw/current/scripts/prefabs/beequeenhive.lua#L188-L213)；第二轮：正文与 `honey` 页统一为「0.01×workleft_after/16」，首锤 0.9375% 仅作示例，并补「归零不给」。；第三轮：`honeycomb` 与 `honey` 的 0.9375% 简写全部补「工作量从 16 起、本击扣 1」条件，通用式不变。 | 通过（2026-11-10 第三轮外部复验）。 |
| R05 | `bird-feathers` L6／L22／L66／L68／L108／L145：按四色原料名是 16 条 `Recipe2`＋1 条 `DeconstructRecipe`；18 行是 `featherhat` 同吃黑羽 3／红羽 2 按颜色展开，末行拆解为返还红羽、方向相反。摘要、标题、正文、导航、验证记录五处统一。[同配方](../../../raw/current/scripts/recipes.lua#L544)、[拆解](../../../raw/current/scripts/recipes.lua#L1741) | 通过（2026-11-10 外部复验）。 |
| R06 | `bird-feathers` L63：删「表内单抽」，改为抽取次数从 1 起、按叶堆阶段加抽、TRICK 固定 1 次、每抽回同一表且不移除已选项；11.1% 是每一抽命中黑羽的概率，两抽至少一根 1−(8/9)²≈21.0%，同次可出两根。[碎屑池](../../../raw/current/scripts/prefabs/leafpile.lua#L97-L108)、[加抽](../../../raw/current/scripts/prefabs/leafpile.lua#L174-L193)、[循环](../../../raw/current/scripts/prefabs/leafpile.lua#L195-L211) | 通过（2026-11-10 外部复验）。 |
| R07 | `walrus-tusk` L6／L23／L45／L49／L50／L62／L73／L81：宿主掉落与拆解回收分开——0.50 为无幸运基准、1.00 走必中；新增「拆解读回」：`greenstaff` 对「有配方且未设 `no_deconstruction`」的目标按 `AllRecipes` 原料逐项返还，手杖／刷子各返 1 牙，回溯怀表须 `pocketwatch_inactive`。[拆解](../../../raw/current/scripts/prefabs/staff_common.lua#L637-L674)、[怀表](../../../raw/current/scripts/recipes.lua#L360-L368)、[手杖](../../../raw/current/scripts/recipes.lua#L552)；第二轮：拆解条件改为「有配方且 `no_deconstruction` 求值不为真」；怀表设为 `pocketwatch_nodecon`，`pocketwatch_inactive` 时求值为假、仍可拆；nav 同步。 | 待复验（第二轮） |
| R08 | `beefalo` L23／L70–73／L77／L96／L98／L108–110、`beefalowool` L23／L42–44／L79：保底掉落限定「非保尸、非 `file_load`」；燃烧改写统一为「`DropLoot` 结算时仍在燃烧」；补骑乘死亡待甩下才进 `death`；保尸先 `Extinguish()` 且不执行 `DropDeathLoot`；拆开 `is_corpsing` 的 `GenerateLoot→StoreLoot` 与直接 `DropLoot(pos)` 两分支。[死亡](../../../raw/current/scripts/prefabs/beefalo.lua#L688-L692)、[保尸](../../../raw/current/scripts/prefabs/beefalo.lua#L671-L673)、[SG](../../../raw/current/scripts/stategraphs/SGBeefalo.lua#L628-L655) | 通过（2026-11-10 外部复验）。 |
| R09 | `furtuft` L6／L21／L23／L47／L50／L51／L53／L100：摘要与正文改为「活体周期掉毛＋尸体结算」两条链；`OnDead` 只停周期任务；尸体换算需 `meat_level >= 2` 且原表多件、含 `bearger_fur`，单件分支只掷是否清空；到点 25–35 秒进入消失序列（醒着 `NOCLICK`＋禁拾取、等动画或兜底任务才 `Remove`，休眠直接删）。[停表](../../../raw/current/scripts/prefabs/bearger.lua#L335-L339)、[尸体](../../../raw/current/scripts/prefabs/corpses.lua#L465-L493)、[消失](../../../raw/current/scripts/components/disappears.lua#L12-L77) | 通过（2026-11-10 外部复验）。 |
| R10 | `coontail` L63／L103：48 秒改回参数值，补「实得乘当前 leader 的 `loyaltyeffectiveness`、并夹到剩余 ≤ `maxfollowtime`＝480 秒」，例：原剩余 470 秒、倍率 1 只增 10 秒；保留敌视分流。[给物](../../../raw/current/scripts/prefabs/catcoon.lua#L281-L302)、[倍率](../../../raw/current/scripts/components/follower.lua#L365-L379) | 通过（2026-11-10 外部复验）。 |
| R11 | `honeycomb` L50／L54／L93、`stinger` L6／L48／L49／L81／L92、`walrus-tusk` L45／L49／L81：共享表 <1 的条目与 `AddChanceLoot` 概率项先过 `GetChance`，蜂王蜜脾期望 1.5、嗡嗡蜜蜂 1%、海象牙 50% 均标「无幸运修正的基准」，1.00 保底走必中。[解析](../../../raw/current/scripts/components/lootdropper.lua#L159-L184)、[概率项](../../../raw/current/scripts/components/lootdropper.lua#L202-L224) | 通过（2026-11-10 外部复验）。 |
| R12 | `stinger` L69：改为「无食用组件与无锅料登记是两件事」——锅按注册名 `IsCookingIngredient(item.prefab)` 收料、不查 `edible`；本快照 `cooking.lua` 无 `stinger` 条目故不能下锅；以蜜脾（无 `edible` 却登记 `sweetener`）作反例。[登记](../../../raw/current/scripts/cooking.lua#L97)、[查表](../../../raw/current/scripts/cooking.lua#L185-L187)、[过滤](../../../raw/current/scripts/containers.lua#L373-L375) | 通过（2026-11-10 外部复验）。 |
| R13 | `furtuft` L6：摘要「毛簇」→ 原版简中「毛丛」，与标题、名称核验一致。[简中](../../../raw/current/scripts/languages/chinese_s.po#L343084-L343087) | 通过（2026-11-10 外部复验）。 |

## 第一轮外部定向复验

2026-11-10，Steam 安装清单与源码快照均为 BUILD `26474873`。逐条核对 R01–R13 的修订与跨页复述，严格源码链接检查覆盖 12 份相关文件，798 处引用 0 错误、0 警告。未运行游戏，未扩大到未受影响的章节。

| 编号 | 结论 | 复验证据与下一次验收 |
|---|---|---|
| R01 | 待修（P3） | P1 的示例已修，实例包装只加倍受击的产出，撞桅仍直接走局部函数。邪天翁页 L107 的验收仍写「让攻击只命中一次」；一击会触发四次掉毛判定，可能命中 0–4 次，无法据攻击次数预期 1→2 根。把验收条件改为「同一次受击中，四次掉毛判定仅命中一次」，并写清期望比较的是这一组判定产生的羽毛。撞桅对照可保留。[邪天翁羽毛页](../../../wiki/reference/prefabs/malbatross-feather.md)、[四次掉毛判定](../../../raw/current/scripts/prefabs/malbatross.lua#L133-L147)、[撞桅直接调用](../../../raw/current/scripts/prefabs/malbatross.lua#L265-L283) |
| R02 | 通过 | 正文已加目标有效、未在去重表、未先命中直接击杀分支、可做工且动作非 `NET` 等门禁，再查 `mast`。导航已区分邪天翁与弃船宝箱，索引保留可做工和三秒去重的条件。[碰撞分支](../../../raw/current/scripts/prefabs/malbatross.lua#L252-L290)、[宝箱条目](../../../raw/current/scripts/scenarios/chest_abandonedboat.lua#L29-L35) |
| R03 | 通过 | 蜜脾页、蜂蜜页与索引都把拆解数值写成按各原料向上取整。80% 耐久仍返 4 脾、75% 返 3，正文也区分了仍存在的冠与耐久耗尽后移除的冠。[比例](../../../raw/current/scripts/prefabs/staff_common.lua#L637-L649)、[返料](../../../raw/current/scripts/prefabs/staff_common.lua#L663-L674)、[耗尽](../../../raw/current/scripts/components/armor.lua#L93-L112) |
| R04 | 待修（P2） | 蜜脾与蜂蜜页已改用扣减后剩余量，但蜜脾 L57、L92–93、蜂蜜 L65 和索引 L25 仍把「首锤 16→15、0.9375%」写作通用值。该值只适用于从 16 开始且本击实际扣 1。工具效率、执行者倍率都能改变扣减量，例如韦斯用基础效率为 1 的锤子、无其他修正时，首锤扣 0.75，剩 15.25，未触发召唤时蜜脾概率为 0.953125%。保留同一公式，把 0.9375% 标为一次扣 1 的示例，并把验证记录中的「首锤」同步限定。[蜜脾页](../../../wiki/reference/prefabs/honeycomb.md)、[蜂蜜页](../../../wiki/reference/prefabs/honey.md)、[索引](../../../wiki/reference/prefabs/animal-drops-index.md)、[效率结算](../../../raw/current/scripts/actions.lua#L1703-L1734)、[韦斯锤击倍率](../../../raw/current/scripts/prefabs/wes.lua#L63-L65)、[倍率值](../../../raw/current/scripts/tuning.lua#L2720)、[按实际剩余量结算](../../../raw/current/scripts/prefabs/beequeenhive.lua#L188-L213) |
| R05 | 通过 | 对四色原料名重新分组为 16 条制作和 1 条拆解。正文及索引已将羽毛帽的两色展开行和拆解返还方向分开。[四色鸟羽](../../../wiki/reference/prefabs/bird-feathers.md)、[羽毛帽配方](../../../raw/current/scripts/recipes.lua#L544)、[拆解返还](../../../raw/current/scripts/recipes.lua#L1741) |
| R06 | 通过 | 落叶堆已写成按阶段增加抽取次数，11.1% 限定到 DEBRIS 的单抽，两抽至少一根 17/81，允许重复抽中。[碎屑池](../../../raw/current/scripts/prefabs/leafpile.lua#L97-L108)、[抽取次数与循环](../../../raw/current/scripts/prefabs/leafpile.lua#L164-L211) |
| R07 | 待修（P3） | 海象牙页与索引已区分宿主掉落和拆解回收，也写出回溯怀表的标签门禁。L62 仍称绿杖要求「未设 `no_deconstruction`」，而同一句的怀表配方正是设了函数。验收写成「有配方，且 `no_deconstruction` 对当前目标求值不为真」，并用已设函数、但带 `pocketwatch_inactive` 的怀表作对照。[海象牙页](../../../wiki/reference/prefabs/walrus-tusk.md)、[绿杖检查配方及结果](../../../raw/current/scripts/prefabs/staff_common.lua#L637-L649)、[怀表函数和配方](../../../raw/current/scripts/recipes.lua#L360-L368) |
| R08 | 通过 | 皮弗娄牛、牛毛与索引已区分普通掉落、保尸跳过、骑乘延迟。燃烧改写改为读结算时状态，正文将尸体暂存与直接掉落列成两个分支。[牛死亡状态](../../../raw/current/scripts/stategraphs/SGBeefalo.lua#L628-L655)、[通用死亡掉落](../../../raw/current/scripts/entityscript.lua#L2181-L2192)、[改写门禁](../../../raw/current/scripts/components/lootdropper.lua#L415-L445) |
| R09 | 通过 | 毛丛的活体与尸体来源已分开，尸体转换同时带肉量、多件及熊皮条目门禁。消失计时已写成开始序列，醒着和休眠的处理不同。[尸体结算](../../../raw/current/scripts/prefabs/corpses.lua#L465-L493)、[消失时序](../../../raw/current/scripts/components/disappears.lua#L12-L48) |
| R10 | 通过 | 猫尾页及索引把 48 秒标为参数，补齐 leader 倍率和 480 秒上限。剩余 470 秒且倍率为 1 时只增 10 秒，当前攻击目标送礼仍走清仇恨分支。[交付分流](../../../raw/current/scripts/prefabs/catcoon.lua#L281-L302)、[倍率与夹取](../../../raw/current/scripts/components/follower.lua#L365-L379) |
| R11 | 通过 | 蜂王蜜脾期望、嗡嗡蜜蜂 1% 和海象牙 50% 的正文及索引均标了无幸运基准。1.00 保底和小于 1 的幸运修正分别说明。[概率解析](../../../raw/current/scripts/components/lootdropper.lua#L173-L184)、[两类概率条目](../../../raw/current/scripts/components/lootdropper.lua#L202-L224) |
| R12 | 通过 | 针刺无 `edible` 与无锅料登记已拆开。普通锅按注册名查询，蜜脾可作反例。[针刺页](../../../wiki/reference/prefabs/stinger.md)、[蜜脾登记](../../../raw/current/scripts/cooking.lua#L97)、[锅的过滤](../../../raw/current/scripts/containers.lua#L373-L375) |
| R13 | 通过 | 毛丛摘要、正文实显和叶索引用名一致。[简中名称](../../../raw/current/scripts/languages/chinese_s.po#L343084-L343087) |

## 第二轮外部定向复验

2026-11-10，Steam 安装清单与源码快照均为 BUILD `26474873`。按第二轮回执复核 R01、R04、R07。五份相关页面和索引共 321 处严格源码引用检查通过，0 错误、0 警告。未运行游戏。

| 编号 | 结论 | 复验证据与下一次验收 |
|---|---|---|
| R01 | 待修（P3） | L107 的验收已按一次受击内四次判定恰中一次写清。L89 标题本轮删了「受击」，写成「每次掉毛判定命中时掉两根」，又涵盖了撞桅的额外判定。恢复标题中的「受击掉毛判定」，与 L91、代码注释及两组验收一致即可。[邪天翁羽毛页](../../../wiki/reference/prefabs/malbatross-feather.md)、[受击的实例调用](../../../raw/current/scripts/prefabs/malbatross.lua#L133-L147)、[撞桅的局部调用](../../../raw/current/scripts/prefabs/malbatross.lua#L265-L283) |
| R04 | 待修（P2） | 蜜脾与蜂蜜正文已给出按扣减后工作量计算的通用公式，并把 16→15 的数值作为例子。叶索引 L25 仍写「首锤 0.9375%」，蜜脾 L57 的首个括注及 L93 也未直接写出扣 1 的前提。涉及 0.9375% 的短句统一写「从 16 开始、扣 1 的首锤示例」，或在索引中只保留公式及「归零不给」。工具或角色倍率可使实际扣减量不同。[叶索引](../../../wiki/reference/prefabs/animal-drops-index.md)、[蜜脾页](../../../wiki/reference/prefabs/honeycomb.md)、[工具与角色倍率](../../../raw/current/scripts/actions.lua#L1703-L1734)、[剩余量与掉落](../../../raw/current/scripts/prefabs/beequeenhive.lua#L188-L213) |
| R07 | 通过 | 海象牙正文、导航和索引都按 `no_deconstruction` 的求值写。回溯怀表配方设置了函数，带 `pocketwatch_inactive` 时函数为假，能通过这一门禁。手杖、刷子各返一牙的口径仍一致。[海象牙页](../../../wiki/reference/prefabs/walrus-tusk.md)、[绿杖门禁及返料](../../../raw/current/scripts/prefabs/staff_common.lua#L637-L674)、[怀表配方及判定函数](../../../raw/current/scripts/recipes.lua#L360-L368) |

## 第三轮外部定向复验

2026-11-10，Steam 安装清单与源码快照均为 BUILD `26474873`。按第三轮回执复核 R01、R04，四份相关页面和索引共 272 处严格源码引用检查通过，0 错误、0 警告。未运行游戏。

| 编号 | 结论 | 定向复验证据 |
|---|---|---|
| R01 | 通过 | 邪天翁页标题恢复「受击掉毛判定」，与代码、注释和验收一致。一次受击仍进行四次判定，其中恰好一次命中时，原版产一根、示例产两根。撞桅仍调用文件局部函数，三根保底及两次额外判定沿用原版。[邪天翁羽毛页](../../../wiki/reference/prefabs/malbatross-feather.md)、[四次受击判定](../../../raw/current/scripts/prefabs/malbatross.lua#L133-L147)、[撞桅局部调用](../../../raw/current/scripts/prefabs/malbatross.lua#L265-L283) |
| R04 | 通过 | 蜜脾正文、陷阱、验证记录、蜂蜜页和叶索引均把 0.9375% 限定到从 16 开始且本击扣 1 的例子。通用公式取本击扣减后的工作量，工具或角色倍率变化时仍按实际剩余量计算。正文保留玩家条件、召蜂王成功时提前返回和剩余量为零时不掉蜜脾。[蜜脾页](../../../wiki/reference/prefabs/honeycomb.md)、[蜂蜜页](../../../wiki/reference/prefabs/honey.md)、[叶索引](../../../wiki/reference/prefabs/animal-drops-index.md)、[扣减顺序](../../../raw/current/scripts/components/workable.lua#L130-L153)、[掉落回调](../../../raw/current/scripts/prefabs/beequeenhive.lua#L188-L213) |
