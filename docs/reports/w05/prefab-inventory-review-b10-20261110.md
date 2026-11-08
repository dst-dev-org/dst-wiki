# B10 常青树与基础制作材料外部源码检视

**当前结论为外部源码复验通过，可登记收口。** 第三轮定向复验确认 R08、R13 通过，R01–R14 累计 14 项全部通过。[第三轮外部定向复验](#第三轮外部定向复验)

初检有 P1 4 项、P2 9 项、P3 1 项。下表按 R01–R14 编号。

检视日期为 2026-11-10。Steam 安装清单和 [`raw/current/BUILD.txt`](../../../raw/current/BUILD.txt) 均为 `26474873`。按 [B10 台账](prefab-inventory-ledger-20261108.md) 核对八页、[制作材料索引](../../../wiki/reference/prefabs/crafting-materials-index.md) 及下列直接相关页面。Wiki 行号对应初检工作树。

## P1

| 编号 | 位置 | 问题、源码证据和修订验收 |
|---|---|---|
| R01 | [浮木与活木头](../../../wiki/reference/prefabs/driftwood-livinglog.md) L115–129 | `modmain.lua` 示例直接读 `TheWorld.ismastersim`。原版创建独立模组环境，只通过 `GLOBAL` 暴露全局表，注册函数也不替换回调环境。示例在新实例执行到这行时会索引 nil。改用 `GLOBAL.TheWorld`，并将「不必重载」限定到实际已改写该实例字段的情形。PostInit 在创建时执行，注册回调不会回写现存实体。验收区分启用后新建／读档重建与注册前已存在的实例，用干燥活木头、容量足够且无其他倍率的受体核对 90 秒 → 120 秒。[模组环境](../../../raw/current/scripts/mods.lua#L258-L292)、[回调登记](../../../raw/current/scripts/modutil.lua#L593-L599)、[实体构造](../../../raw/current/scripts/mainfunctions.lua#L464-L503)、[燃料读取](../../../raw/current/scripts/components/fueled.lua#L197-L221) |
| R02 | [灰烬与木炭](../../../wiki/reference/prefabs/ashes-and-charcoal.md) L101 | 「永不消失」建议只删 `disappears`，但原版拾取、落地和作祟回调都直接取该组件。单独移除会留下三个访问 nil 的入口。验收保留延长延迟的例子，删除这个不安全的操作建议。若仍给永久灰烬的做法，须明确每个入口由谁接管，并分别验证拾取、落地和作祟。[三个回调](../../../raw/current/scripts/prefabs/ash.lua#L15-L31)、[拾放接线](../../../raw/current/scripts/prefabs/ash.lua#L65-L77)、[作祟接线](../../../raw/current/scripts/prefabs/ash.lua#L91-L100) |
| R03 | [骨头碎片](../../../wiki/reference/prefabs/boneshard.md) L76、L99–100 | 「几乎只会推向湿腻焦糊」漏掉可直接复现的配方，当前唯一举出的鱼排也容不下骨头碎片。鱼排必需树枝，树枝已用满 `inedible<=1`。四块骨头碎片可满足 `beefalofeed`，优先级 −5 高于兜底 −10。两土豆、一树枝、一骨头碎片可满足 `potatotornado` 的 `inedible<=2`，优先级 10。验收用这两个正例和鱼排反例说明「可以进锅」「可匹配配方」「可直接在火上烤」的各自门禁。[树枝及骨片登记](../../../raw/current/scripts/cooking.lua#L134-L151)、[鱼排](../../../raw/current/scripts/preparedfoods.lua#L79-L83)、[兜底](../../../raw/current/scripts/preparedfoods.lua#L352-L362)、[回旋块茎](../../../raw/current/scripts/preparedfoods.lua#L480-L490)、[蒸树枝](../../../raw/current/scripts/preparedfoods.lua#L1034-L1049) |
| R04 | [月岩](../../../wiki/reference/prefabs/moonrocknugget.md) L47、L59、L84、L92、L99；[索引](../../../wiki/reference/prefabs/crafting-materials-index.md) L20 | 「月亮石被采空、仅一次」把零工作量回调当成了普通采掘，并把当前零工作量阶段的去重写成终身限制。月亮石的工作动作设为 nil。进入零工作量时，仅在无 `lootdropper` 时挂组件并放出 1–2 枚。修到正工作量会移除该组件，再次降至零可以再走掉落分支。验收把它写作可由工作量链触发的条件掉落，索引标明「初始 4/6 时一件补满」。每件 +2，零起算须三件。[动作与初始量](../../../raw/current/scripts/prefabs/moonbase.lua#L678-L687)、[零态掉落](../../../raw/current/scripts/prefabs/moonbase.lua#L195-L206)、[正值清组件](../../../raw/current/scripts/prefabs/moonbase.lua#L252-L255)、[修理回调](../../../raw/current/scripts/prefabs/moonbase.lua#L499-L524) |

## P2

| 编号 | 位置 | 问题、源码证据和修订验收 |
|---|---|---|
| R05 | [常青树族](../../../wiki/reference/prefabs/evergreen-trees.md) L56–65 | 生长表把以 `day_time` 为单位的调参值写成「天」。这里 `day_time=300` 秒，完整昼夜为 480 秒。验收全表统一口径，普通矮／中／高／老为 450±150、1500±600、1500±600、300±150 秒，多枝树对应 450±150、900±300、900±300、1500±150 秒。季节倍率和另用完整昼夜的再生计时分开写。[昼夜单位](../../../raw/current/scripts/tuning.lua#L19-L29)、[生长表](../../../raw/current/scripts/tuning.lua#L788-L802)、[计时器](../../../raw/current/scripts/components/growable.lua#L57-L78)、[再生常量](../../../raw/current/scripts/tuning.lua#L3187-L3205) |
| R06 | [常青树族](../../../wiki/reference/prefabs/evergreen-trees.md) L63、L99–100 | 「每砍一下唤醒并设目标」漏了树休眠、缺 `growable`、老态的早退。命中后对睡着的树精安排延迟唤醒，`SuggestTarget` 是建议目标。作祟特例读 `noleif`、生长阶段、`burnt`／`stump`，也要找到树精候选，页面的「未砍／没烧」含义更宽。验收用已经砍过但还活着的树、老态树和燃烧中尚无 `burnt` 的树区分对应条件，保留未命中特例时的作祟做工路径。[砍击回调](../../../raw/current/scripts/prefabs/evergreens.lua#L391-L418)、[树精候选](../../../raw/current/scripts/prefabs/evergreens.lua#L436-L449)、[作祟回调](../../../raw/current/scripts/prefabs/evergreens.lua#L792-L813) |
| R07 | [灰烬与木炭](../../../wiki/reference/prefabs/ashes-and-charcoal.md) 摘要、L22、L55、L62–75、L120；[索引](../../../wiki/reference/prefabs/crafting-materials-index.md) L16 | 作祟立即触发消失序列，休眠时当场删除，醒着时先禁点选／拾取再等动画或兜底任务。烧毁产出也有两个不同入口。40% 加灰烬属于 `GenerateLoot`，带显式列表的 `DropLoot` 跳过生成阶段；传入的列表仍可在已焦或符合燃烧门禁时走同一熟品／灰烬改写。验收用显式列表与默认列表、醒着与休眠四个反例压实限定，摘要、来源表和索引同步。[消失序列](../../../raw/current/scripts/components/disappears.lua#L12-L48)、[40% 生成条件](../../../raw/current/scripts/components/lootdropper.lua#L248-L256)、[列表改写](../../../raw/current/scripts/components/lootdropper.lua#L415-L445) |
| R08 | [发条零件](../../../wiki/reference/prefabs/clockwork-parts.md) L83、L87、L90 | 电子元件段称无稳定必掉来源。双恐怖之眼两套实际挂载的掉落表均有两条 100% 和一条 75%。两个检查眼镜箱也应分开，第一箱 75%＋50% 有可能空掉，第二箱 100%＋75% 至少一件。验收核对每个宿主选中的表，写出保底与额外概率，统一小结。[双眼挂表](../../../raw/current/scripts/prefabs/eyeofterror.lua#L379-L389)、[两套表](../../../raw/current/scripts/prefabs/eyeofterror.lua#L547-L558)、[第二套](../../../raw/current/scripts/prefabs/eyeofterror.lua#L617-L628)、[箱表](../../../raw/current/scripts/prefabs/inspectaclesbox.lua#L12-L28)、[概率结算](../../../raw/current/scripts/components/lootdropper.lua#L213-L224) |
| R09 | [发条零件](../../../wiki/reference/prefabs/clockwork-parts.md) L60、L81、L93、L98–99 | 神龛「反向消耗」掩盖供品的存续。神龛关闭收下即删除，保留实体并移出场景，换供品时把上一件同一实体送回场景。另有清空时删除的路径。验收把交存、换回、删除各自的分支写清，摘要和用途段同步区分供品、配方扣料和交易。[交存和退回](../../../raw/current/scripts/prefabs/yoth_knightshrine.lua#L90-L129)、[清空分支](../../../raw/current/scripts/prefabs/yoth_knightshrine.lua#L156-L170)、[收取配置](../../../raw/current/scripts/prefabs/yoth_knightshrine.lua#L183-L189) |
| R10 | [月岩](../../../wiki/reference/prefabs/moonrocknugget.md) L51–62、L69、L99；[索引](../../../wiki/reference/prefabs/crafting-materials-index.md) L20 | 威尔逊的 `transmute_moonrocknugget` 消耗 2 大理石产出 1 月岩，页面把它放在消费表，并把按名提及的 19 条当作同一统计口径。验收把该配方移到来源，验证记录单列 18 条以月岩为原料的 `Recipe2`、3 个建造配方和 7 个拆解配方。索引把「弹弓弹药」写明为「弹弓弹药配方原料」。[大理石转化](../../../raw/current/scripts/recipes.lua#L1477-L1484)、[弹药配方](../../../raw/current/scripts/recipes.lua#L219-L220) |
| R11 | [月岩](../../../wiki/reference/prefabs/moonrocknugget.md) L70、L79、L95；[盐晶](../../../wiki/reference/prefabs/saltrock.md) L68 | 两页都从 ELEMENTAL 食性推出尘蛾会吃。尘蛾的交易测试与日常进食目标还要求 `dustmothfood`，月岩和盐晶都没加该标签。验收把食性允许与实际选择入口分开，饱食 1、生命 10 写成食物组件参数，实际回复还取决于进食者组件。月岩同样不能仅凭食性写成会进入鼹鼠的地面诱饵选择。[尘蛾交易与食性](../../../raw/current/scripts/prefabs/dustmoth.lua#L62-L66)、[交易挂接](../../../raw/current/scripts/prefabs/dustmoth.lua#L251-L257)、[尘蛾行为树](../../../raw/current/scripts/brains/dustmothbrain.lua#L57-L85)、[鼹鼠行为树](../../../raw/current/scripts/brains/molebrain.lua#L56-L73)、[月岩](../../../raw/current/scripts/prefabs/moonrocknugget.lua#L6-L54)、[盐晶](../../../raw/current/scripts/prefabs/saltrock.lua#L26-L81) |
| R12 | [盐晶](../../../wiki/reference/prefabs/saltrock.md) 摘要、L50–53、L88–92；[索引](../../../wiki/reference/prefabs/crafting-materials-index.md) L21；[保鲜组件](../../../wiki/reference/components/preservative.md) L29–33 | 撒盐的「+50%」是夹取前的 50 个百分点，按目标叠数除。合法满鲜目标仍能撒盐并扣件，服务端先设新鲜度再取盐，取件失败返回 false，已经写入的百分比没有回滚。舔盐块的 REPAIR 有满值拒绝，和撒盐是两条路径。验收用四叠 90% 的食物（升到 100%）、已满鲜食物（仍扣盐）和满值舔盐块（拒绝修理）对照，并同步组件接口说明。[撒盐动作候选](../../../raw/current/scripts/componentactions.lua#L1574-L1582)、[撒盐结算](../../../raw/current/scripts/actions.lua#L4620-L4644)、[百分比夹取](../../../raw/current/scripts/components/perishable.lua#L223-L232)、[满值拒绝与扣件](../../../raw/current/scripts/components/repairable.lua#L135-L179) |
| R13 | [盐晶](../../../wiki/reference/prefabs/saltrock.md) L57、L66、L90 | 「采到空后生长」漏了未采空也会重启生长计时。计时受世界设置和正倍率约束，船体相交时本轮跳过增长，未满仍重排。挖掘一次跨多档时结算跨过的档位表。诱饵段的「任意已布好的陷阱」也扩大了范围，鸟必须有 `eater` 且不忽略诱饵，候选还受陷阱状态与周边危险度约束。验收覆盖只挖一档、平台压住、符合与不符合随机落陷阱条件，保留盐晶不会作为可食诱饵吸引鸟的结论。[跨档结算](../../../raw/current/scripts/prefabs/saltstack.lua#L62-L85)、[重排与船体门禁](../../../raw/current/scripts/prefabs/saltstack.lua#L121-L154)、[世界设置](../../../raw/current/scripts/prefabs/saltstack.lua#L287-L302)、[鸟候选](../../../raw/current/scripts/components/birdspawner.lua#L534-L560) |

## P3

| 编号 | 位置 | 问题、证据和修订验收 |
|---|---|---|
| R14 | [灰烬与木炭](../../../wiki/reference/prefabs/ashes-and-charcoal.md)、[发条零件](../../../wiki/reference/prefabs/clockwork-parts.md)、[骨头碎片](../../../wiki/reference/prefabs/boneshard.md)、[蜘蛛丝](../../../wiki/reference/prefabs/silk.md)、[索引](../../../wiki/reference/prefabs/crafting-materials-index.md) | 同一 `catcoon` 在正文、索引、验证记录里有「猫狸」「猫浣熊」。实体的原版简中名称为「浣猫」。验收在这五个入口统一实体称谓，保留注册名检索，配方和装备名称按自己的字符串键核对。[简中实体名称](../../../raw/current/scripts/languages/chinese_s.po#L340690-L340692) |

## 已核验范围

| 从源码建立的事实骨架 | 页面与索引命中情况 |
|---|---|
| 常青树的注册族、阶段工作量、烧毁／树桩分流、正常与作祟砍伐的树精入口 | 注册、阶段和生长再生分开有覆盖。阶段单位与入口条件见 R05–R06。 |
| 灰烬物品的 25–35 秒任务、拾取停表与作祟；木炭燃料／可燃；源列表、烧毁改写与宿主特定战利品 | 定义值和木炭两段计时可保留。拆组件建议、消失时序及共享掉落入口见 R02、R07。 |
| 浮木与活木头的组件和注册、WOOD 食性、修理满值门、活木头燃料消费 | 两个物品的角色用途有覆盖。示例的模组环境和实例生命周期见 R01。 |
| 齿轮的 GEARS 食性、修理／跟随机制；电子元件的掉落和共享神龛消费端 | 已核对页面两层门禁、跟随上限与后续结算的区分。掉落和供品状态见 R08–R09。 |
| 骨头碎片作为锅料的标签累加与最终优先级，原料没有直接火烤入口 | 页面有登记与火烤限制，缺可成立的锅料理对照，见 R03。 |
| 月岩墙与月亮石的两种修复通道、零工作量掉落、配方方向与 ELEMENTAL 食性 | 数值 +40／+2 的分流有覆盖，月亮石、转化和食性消费入口见 R04、R10–R11。 |
| 盐晶的撒盐和舔盐块两条动作、盐柱掉落／再生、鸟诱饵和 ELEMENTAL 食性 | 页面区分两个动作和食性标签，消费端和世界门禁仍须同步，见 R11–R13。 |
| 蜘蛛丝的消耗入口、韦伯和蜘蛛巢的升级门、五次升阶与三阶上限 | 已沿升级动作核到组件和蜘蛛巢消费端，页内与索引主要值一致。浣猫称谓见 R14。 |

以同 BUILD 的本地注册导出 `output/coverage/prefab-26474873-shard-1-aligned/registrations.json` 对账，八页的 25 个主注册均命中，当前 Prefab 页的精确别名无重复归属。跨页复核含制作材料索引、直接关联的基础采集／矿物／树族入口和相关组件段落。数值、范围、绝对句按八页和索引行逐项回源，消费端沿上表所列动作或回调追查。游戏运行、引擎表现、世界生成密度、掉落来源完整枚举和旧页无关章节未核验。

## 修订回执与定向复验

按编号回填，写清改动位置、同一事实在摘要／快速判断／正文／陷阱／验证记录及索引或关联页的同步情况。保留原报告证据，复验在后续记录判定。以下回执由各页实际写作者提交、主代理整理；叶索引行已由主代理同步。

| 编号 | 写作修订回执 | 提交时状态 |
|---|---|---|
| R01 | 已改：driftwood L115–L129 示例改 `GLOBAL.TheWorld.ismastersim`；写明 PostInit 只对之后创建／读档重建的实例生效，注册前已存在实例除非直接改该实例字段否则不变；示例 45→120 秒，验收用干燥活木、容量足够、无 fuelmaster／multfn 的受体。依据 mods.lua#L258-L292、modutil.lua#L593-L599、mainfunctions.lua#L464-L503、fueled.lua#L197-L221。validate_page／source_links 通过。 | 待复验 |
| R02 | 已改：ashes「典型接入」删除「为永久保留而删 disappears」建议，改为警示拾取／落地／作祟三处回调都直接索引该组件、确需永久须分别接管并逐项验证；延长延迟示例保留。依据 ash.lua#L15-L31、#L65-L77、#L91-L100。 | 待复验 |
| R03 | 已改：boneshard 改「烹饪三层门禁」；补蒸树枝（4 骨片，priority −5）与花式回旋块茎（2 土豆＋1 树枝＋1 骨片，inedible≤2，priority 10）正例，炸鱼排作反例；陷阱条同步。依据 cooking.lua#L134／L151／L217-L227、preparedfoods.lua#L79-L83／L352-L362／L480-L490／L1034-L1049、containers.lua#L373-L375、components/cooker.lua#L12-L17。 | 待复验 |
| R04 | 已改：moonrock 改为「workable 无采掘动作＋零工作量条件掉落 1–2 枚、修回正值可再触发」；补攻击侧 WorkedBy（月犬／月猪／月兽 spawner）与 0 起算 3 件；摘要、获得路径、导航、陷阱、验证记录同步。依据 moonbase.lua#L195-L206／L253-L254／L678-L687、SGhound.lua#L765-L774、SGmoonpig.lua#L264-L276、moonbeastspawner.lua#L128-L133、tuning.lua#L2147-L2148。 | 待复验 |
| R05 | 已改：evergreen L56–L66 生长表全秒化（普通／臃肿 450±150、1500±600、1500±600、300±150；多枝树 450±150、900±300、900±300、1500±150）；新增 day_time=300 与 total_day_time=480 分开计的口径条；树苗改 225±75 秒。依据 tuning.lua#L19-L29／L788-L802／L3187-L3205、growable.lua#L57-L78。 | 待复验 |
| R06 | 已改：evergreen L63、L100–L101 写三档早退（休眠或缺 growable 直返、老树放音后返、其余继续）；唤醒为延迟 `math.random()` 秒且 `SuggestTarget` 只是建议；作祟按 noleif／阶段≤3／burnt／stump 收窄，补三类反例与回落 onhauntwork。依据 evergreens.lua#L391-L418／L436-L440／L792-L813。 | 待复验 |
| R07 | 已改：ashes 摘要、重要副作用、disappears 行、获得链与陷阱统一「作祟＝到期同一序列」（醒着先禁点选／拾取再动画＋兜底，休眠当场删除）；烧毁分默认列表（GenerateLoot 内 40% 木炭）与显式列表（跳过生成、无 40%，仍走改写门禁）。依据 components/disappears.lua#L12-L48、components/lootdropper.lua#L248-L256／L415-L445。 | 待复验 |
| R08 | 已改：clockwork 电子元件改为「有保底来源」；双恐怖之眼两套表各 2 条 100%＋1 条 75%（至少 2 个）；检查眼镜箱拆两行（箱 1 可空、箱 2 至少 1 个）。依据 eyeofterror.lua#L386-L387／L547-L558／L617-L628、inspectaclesbox.lua#L12-L28、components/lootdropper.lua#L213-L224。 | 待复验 |
| R09 | 已改：clockwork 神龛由「反向消耗」改「交存」：保留实体、移出场景开原型器；换供品送回上一件；清空／点燃／烧毁才删除；摘要与用途区分配方扣料／供品／装饰。依据 yoth_knightshrine.lua#L90-L129／L156-L170／L183-L189／L193-L205。 | 待复验 |
| R10 | 已改：moonrock 把 transmute_moonrocknugget 移到来源（2 大理石→1 月岩）；验证记录改为 18 条以月岩为原料的 Recipe2＋3 条建造＋7 条拆解。依据 recipes.lua#L1483、#L219-L220。 | 待复验 |
| R11 | 已改：moonrock L70／L81／L97、saltrock L70 拆开「食性允许」与「行为入口」；尘蛾需 dustmothfood、鼹鼠需 molebait；饱食 1／生命 10 为组件参数；月岩不再写成会进鼹鼠诱饵。依据 dustmoth.lua#L62-L66／L251-L257、dustmothbrain.lua#L57-L77、molebrain.lua#L64-L73、rocky.lua#L436、slurtle.lua#L150、constants.lua#L2049-L2063、components/eater.lua#L18-L24。 | 待复验 |
| R12 | 已改：saltrock 摘要／L51–L53／陷阱与 preservative.md 接口段——夹取前 +50 个百分点、按目标叠数除、90% 只涨 10 个百分点、满鲜仍扣件、先设新鲜度后取件失败不回滚、与舔盐块 REPAIR 满值拒绝分开。依据 componentactions.lua#L1574-L1582、actions.lua#L4620-L4644、perishable.lua#L223-L232、repairable.lua#L135-L142。 | 待复验 |
| R13 | 已改：saltrock 盐柱补「每次 OnWork 重排计时、船体压住本轮跳过、世界设置与正倍率门禁、跨档 DropLoots 结算」；鸟诱饵收窄到 eater／ignorebait／CanEat，落陷阱另列（isset、targettag、issprung、BIRD_TRAP_CHANCE、危险度）。依据 saltstack.lua#L62-L85／L121-L154／L287-L302、birdspawner.lua#L534-L559。 | 待复验 |
| R14 | 已改：ashes／clockwork／boneshard／silk 与叶索引的 catcoon 实体称谓统一「浣猫」，保留 catcoon 检索；silk 的 catcoonhat 按自身名称键改「猫帽」；索引行已由主代理同步。依据 chinese_s.po#L340690-L340692、#L340700-L340702。 | 待复验 |

## 第一轮外部定向复验

2026-11-10，Steam 安装清单和源码快照均为 BUILD `26474873`。按回执逐项核对 R01–R14 的修订、同事实的复述和对应调用链。严格源码链接检查覆盖八页、制作材料索引与 `preservative` 组件页，863 处支持的引用 0 错误、0 警告，另有 2 处工具跳过。未运行游戏，未扩大到旧页无关章节。

复验时纠正初检报告两处笔误。R01 的活木头原版燃料是 45 秒，90 秒来自初稿的翻倍示例，现示例为 45 → 120 秒。R07 的 40% 追加物是木炭。[燃料常量](../../../raw/current/scripts/tuning.lua#L1673-L1676)、[活木头取值](../../../raw/current/scripts/prefabs/livinglog.lua#L51-L53)、[木炭追加](../../../raw/current/scripts/components/lootdropper.lua#L248-L256)

| 编号 | 结论 | 定向复验证据与下一次验收 |
|---|---|---|
| R01 | 通过 | 示例已改用 `GLOBAL.TheWorld`，区分注册前已存在的实例和后续创建／读档重建的实例。改写只触及 `fuelvalue`，保留原燃料消费回调。干燥、容量足够、无附加倍率的验收场景明确。[示例](../../../wiki/reference/prefabs/driftwood-livinglog.md)、[模组环境](../../../raw/current/scripts/mods.lua#L258-L292)、[燃料结算](../../../raw/current/scripts/components/fueled.lua#L197-L221) |
| R02 | 通过 | 已删去直接移除组件的操作建议，提示拾取、落地、作祟三个入口。延长延迟的示例保留组件，并只对已有任务重排，`PrepareDisappear` 会先取消旧任务。[三个入口](../../../raw/current/scripts/prefabs/ash.lua#L15-L31)、[重排](../../../raw/current/scripts/components/disappears.lua#L51-L77) |
| R03 | 通过 | 正文、陷阱及索引已有蒸树枝、花式回旋块茎的可成立组合和炸鱼排超限反例，区分锅料登记、配方测试和火上直接烤制。[配方和陷阱](../../../wiki/reference/prefabs/boneshard.md)、[标签登记](../../../raw/current/scripts/cooking.lua#L134-L151)、[锅料过滤](../../../raw/current/scripts/containers.lua#L373-L375)、[火烤条件](../../../raw/current/scripts/components/cooker.lua#L12-L17) |
| R04 | 通过 | 正文和索引已区分无普通采掘动作、零工作量阶段内去重，以及修到正值后重新归零可再掉。确认月犬、月猪攻击和月兽生成器离屏工作是实际扣减入口，初始 4/6 用一件、从零用三件。[月亮石初始化](../../../raw/current/scripts/prefabs/moonbase.lua#L678-L687)、[修理回调](../../../raw/current/scripts/prefabs/moonbase.lua#L499-L524)、[月犬](../../../raw/current/scripts/stategraphs/SGhound.lua#L765-L774)、[月猪](../../../raw/current/scripts/stategraphs/SGmoonpig.lua#L264-L276)、[离屏工作](../../../raw/current/scripts/components/moonbeastspawner.lua#L115-L133) |
| R05 | 通过 | 生长表与树苗都改用秒，保留春季阶段倍率和独立的完整昼夜再生口径。多枝树实际配置引用普通树的 5 天，正文已区分其另行定义的 8 天常量。[阶段调参](../../../raw/current/scripts/tuning.lua#L788-L802)、[树族配置](../../../raw/current/scripts/prefabs/evergreens.lua#L56-L100)、[再生时长](../../../raw/current/scripts/tuning.lua#L3189-L3205) |
| R06 | 通过 | 砍击已写明休眠、缺组件、老态早退和延迟唤醒，仇恨只是建议。作祟段按 `noleif`、阶段及 `burnt`／`stump` 收窄，保留特例不中的做工路径。[砍击](../../../raw/current/scripts/prefabs/evergreens.lua#L391-L418)、[候选](../../../raw/current/scripts/prefabs/evergreens.lua#L436-L440)、[作祟](../../../raw/current/scripts/prefabs/evergreens.lua#L792-L814) |
| R07 | 待修（P2） | 灰烬作祟时序和 40% 木炭的归属已修。灰烬页 L63、L75 仍把显式列表写成 `DropLoot(prefabs)`，此调用会把数组当成第一个参数 `pt`，第二参仍空，照样进入 `GenerateLoot`。改用 `DropLoot(pt, prefabs)`，需要默认落点时可传 `nil` 作为位置。常青树页 L88 的通用措辞也限定到此处两次采用默认列表的调用。验收以默认列表调用和原版尸体传入位置＋列表的调用对照。[灰烬页](../../../wiki/reference/prefabs/ashes-and-charcoal.md)、[常青树页](../../../wiki/reference/prefabs/evergreen-trees.md)、[参数和改写](../../../raw/current/scripts/components/lootdropper.lua#L415-L445)、[实际调用](../../../raw/current/scripts/prefabs/corpses.lua#L487-L494) |
| R08 | 待修（P2） | 两种眼镜箱的元件保底已修。发条零件 L69、L87 仍把双恐怖之眼标成 `eyeofterror`，该注册是独立的恐怖之眼，掉落表无齿轮和电子元件。应写 `twinofterror1`、`twinofterror2`；各自常规死亡掉 3–5 齿轮、2–3 电子元件，两只都击杀合计 6–10／4–6。索引 L18 的 2 个保底加「每只」，L83 小结保留第二种检查眼镜箱也有保底。[发条零件页](../../../wiki/reference/prefabs/clockwork-parts.md)、[叶索引](../../../wiki/reference/prefabs/crafting-materials-index.md)、[注册](../../../raw/current/scripts/prefabs/eyeofterror.lua#L984-L987)、[单眼掉落](../../../raw/current/scripts/prefabs/eyeofterror.lua#L461-L474)、[双眼表一](../../../raw/current/scripts/prefabs/eyeofterror.lua#L545-L558)、[双眼表二](../../../raw/current/scripts/prefabs/eyeofterror.lua#L615-L628)、[死亡结算](../../../raw/current/scripts/stategraphs/SGeyeofterror.lua#L967-L977) |
| R09 | 待修（P3） | 交存、退回、删除的正文已基本区分。发条零件摘要仍说电子元件「只作制造材料」，同时又写神龛供品；索引 L18「供品只交存、不扣料」未限定到收取时。去掉摘要里的「只」，索引写明收下时保留，换供品会退回，点燃时仍可能丢失。正文 L100 的点燃和烧毁若保留灰烬说明，分别按 `OnIgnite`、`OnBurnt` 是否仍持有供品说明，避免理解成一件供品两次都出灰。[发条零件页](../../../wiki/reference/prefabs/clockwork-parts.md)、[叶索引](../../../wiki/reference/prefabs/crafting-materials-index.md)、[交存和退回](../../../raw/current/scripts/prefabs/yoth_knightshrine.lua#L90-L129)、[点燃与烧毁](../../../raw/current/scripts/prefabs/yoth_knightshrine.lua#L193-L214) |
| R10 | 通过 | 威尔逊转化已归入来源，索引写为弹弓弹药配方的原料。按 `Ingredient("moonrocknugget", ...)` 重新分组，18 条制作、3 条建造、7 条拆解。[月岩页](../../../wiki/reference/prefabs/moonrocknugget.md)、[转化](../../../raw/current/scripts/recipes.lua#L1477-L1484)、[弹药](../../../raw/current/scripts/recipes.lua#L219-L220) |
| R11 | 待修（P2） | 月岩的尘蛾／鼹鼠两项门禁已写清。盐晶 L70 仍说 `dustmothfood` 与 `molebait`「两者都没有」，和 L69、摘要、索引相反。盐晶有 `molebait` 和 `bait`，可被鼹鼠选作偷饵，只缺尘蛾要求的 `dustmothfood`。验收分别对照「尘蛾不会在日常取食或交易入口选盐晶」「地面盐晶满足鼹鼠的物品侧偷饵条件」，实际进食后的回复继续按组件参数及进食者区分。[盐晶页](../../../wiki/reference/prefabs/saltrock.md)、[盐晶标签](../../../raw/current/scripts/prefabs/saltrock.lua#L41-L57)、[鼹鼠筛选](../../../raw/current/scripts/brains/molebrain.lua#L56-L79)、[尘蛾日常取食](../../../raw/current/scripts/brains/dustmothbrain.lua#L57-L85) |
| R12 | 通过 | 已分别写出分摊、满鲜仍扣件、先改百分比后取件、取件失败不回滚，以及满值舔盐块拒绝修理。四件一叠时夹取前加 12.5 个百分点，90% 的目标实际只增 10 个百分点。[盐晶页](../../../wiki/reference/prefabs/saltrock.md)、[组件页](../../../wiki/reference/components/preservative.md)、[撒盐结算](../../../raw/current/scripts/actions.lua#L4620-L4644)、[满值拒绝](../../../raw/current/scripts/components/repairable.lua#L135-L142) |
| R13 | 待修（P2） | 盐柱计时和跨档结算已通过。盐晶 L68、L92 把「有 `eater` 且不 `ignorebait`」只写在食饵分支，又要求陷阱「有匹配的 `targettag`」。源码把两条分支包在同一外层门禁内，陷阱允许未设 `targettag`，或者设了且鸟带对应标签。验收用忽略诱饵的鸟与未设目标标签的可用陷阱对照，仍明确盐晶不会被当作可食诱饵。[盐晶页](../../../wiki/reference/prefabs/saltrock.md)、[生长计时](../../../raw/current/scripts/prefabs/saltstack.lua#L121-L154)、[鸟落点分支](../../../raw/current/scripts/components/birdspawner.lua#L534-L560) |
| R14 | 通过 | 四页与索引的实体称谓已统一「浣猫」，蜘蛛丝页的装备按单独名称键写「猫帽」。[实体名称](../../../raw/current/scripts/languages/chinese_s.po#L340690-L340692)、[装备名称](../../../raw/current/scripts/languages/chinese_s.po#L340700-L340702) |

## 第二轮修订回执

按第一轮复验的待修项回填；仍由各页实际写作者提交、主代理整理。

| 编号 | 写作修订回执 | 提交时状态 |
|---|---|---|
| R07 | 已改：ashes 路线 2／3 与导航 lootdropper 行改用 `DropLoot(pt, prefabs)`，写明只有第二参传列表才跳过 `GenerateLoot`、无 40% 追加，只传数组会占据 `pt` 位；路线 2 补 `DropLoot(pt)` 默认入口与 corpses 调用例。evergreen L88／L89 与陷阱行把 40% 木炭限定为两处默认列表调用（`onpineconetask` 的 `DropLoot(pt)` 与 `chop_down_burnt_tree` 的 `DropLoot()`），显式列表不追加。依据 lootdropper.lua#L415-L445／#L252-L254、corpses.lua#L489-L494。两页 validate 通过。 | 待复验 |
| R08 | 已改：clockwork 齿轮／电子元件两表「双恐怖之眼」行改为 `twinofterror1`／`twinofterror2`（独立注册、各挂同名表）；每只 3–5 齿轮／2–3 电子元件，两只合计 6–10／4–6；补注册与 `DropDeathLoot` 锚，注明单眼 `eyeofterror` 表无齿轮。叶索引 L18 已由主代理改为「每只保底 2 个」。依据 eyeofterror.lua#L984-L987／#L461-L474／#L543-L564／#L615-L634、SGeyeofterror.lua#L967-L977。 | 待复验 |
| R09 | 已改：摘要去掉电子元件「只作制造材料」，改为「主要作制造材料，也能当神龛供品交存」；导航神龛行与用途段按 `OnIgnite`／`OnBurnt` 分开出灰——`OnIgnite` 仍持供品时先掉 1 灰烬再由 `MakeEmpty` 删除，`OnBurnt` 只在届时仍持有才掉 1，同一件不二次出灰。叶索引 L18 已由主代理改为「收下时保留、换供品退回，点燃／烧毁可能丢失」。依据 yoth_knightshrine.lua#L90-L129／#L156-L170／#L193-L214。 | 待复验 |
| R11 | 已改：saltrock L70 拆开写——盐晶有 bait／molebait，会被鼹鼠选作偷饵；无 dustmothfood，尘蛾日常取食（dustmothbrain#L57-L85）与交易（dustmoth.lua#L62-L66）都不会选它；保留食物组件参数与进食者修饰。依据 saltrock.lua#L41-L57、molebrain.lua#L56-L79。validate_page／source_links 通过。 | 待复验 |
| R13 | 已改：saltrock L68／L92 按 birdspawner#L536-L560 重写——两条分支共用外层门禁（鸟有 eater 且未被 ignorebait 排除）；可食诱饵分支要求 CanEat、bait、未持有、无危险且可通行；陷阱分支只看 isset、未 issprung、targettag 为空或匹配、BIRD_TRAP_CHANCE、无危险；明确 targettag 可为空，保留「盐晶不被当作可食诱饵」。validate_page／source_links 通过。 | 待复验 |

## 第二轮外部定向复验

2026-11-10，Steam 安装清单与源码快照均为 BUILD `26474873`。本轮按回执核对五个待修编号、受影响的正文复述和制作材料索引。五份文件共 452 处严格源码引用检查通过。未运行游戏。

| 编号 | 结论 | 定向复验证据与下一次验收 |
|---|---|---|
| R07 | 通过 | 灰烬导航与来源段已写清 `DropLoot(pt, prefabs)` 的参数位置和默认列表条件。常青树正文和索引把 40% 木炭限定到两处默认列表调用。对照尸体传入第二参的原版调用，以及 `FlingItem` 对 nil 落点的回落，口径一致。[掉落入口](../../../raw/current/scripts/components/lootdropper.lua#L415-L445)、[默认落点](../../../raw/current/scripts/components/lootdropper.lua#L321-L332)、[尸体调用](../../../raw/current/scripts/prefabs/corpses.lua#L487-L494)、[常青树定时调用](../../../raw/current/scripts/prefabs/evergreens.lua#L551-L559) |
| R08 | 待修（P3） | 两张表的注册、每只掉落范围和击杀两只的合计，以及索引「每只保底 2 个」均已修。发条零件页 L83 小结仍只把双恐怖之眼列为保底来源，其余归入拆废料与概率，和 L91 的第二种检查眼镜箱「至少 1 个」冲突。验收只需把箱 2 加入小结的保底来源，或删掉小结中对其余来源的概括。[发条零件页](../../../wiki/reference/prefabs/clockwork-parts.md)、[箱 2 掉落表](../../../raw/current/scripts/prefabs/inspectaclesbox.lua#L21-L28)、[箱 2 挂表](../../../raw/current/scripts/prefabs/inspectaclesbox.lua#L458-L466) |
| R09 | 通过 | 摘要已包含电子元件的供品用途，索引区分收下时保留、换供品退回和燃烧可能丢失。正文把点燃与烧毁的出灰条件分别写明，点燃已清空时烧毁不会再按同一件供品出灰。[供品交存及退回](../../../raw/current/scripts/prefabs/yoth_knightshrine.lua#L90-L129)、[点燃与烧毁](../../../raw/current/scripts/prefabs/yoth_knightshrine.lua#L193-L214) |
| R11 | 通过 | 盐晶正文、陷阱用途和索引已统一。盐晶带 `molebait` 标签与 `bait` 组件，满足鼹鼠偷饵的物品侧门禁。它缺 `dustmothfood`，尘蛾日常取食和交易入口均不会选中它。食物组件参数与实际回复仍分别说明。[盐晶标签](../../../raw/current/scripts/prefabs/saltrock.lua#L41-L64)、[鼹鼠筛选](../../../raw/current/scripts/brains/molebrain.lua#L56-L79)、[尘蛾交易](../../../raw/current/scripts/prefabs/dustmoth.lua#L62-L66)、[尘蛾取食](../../../raw/current/scripts/brains/dustmothbrain.lua#L57-L85) |
| R13 | 待修（P2） | 共用的 `eater`／`ignorebait` 外层门与可空 `targettag` 已修。盐晶 L68、L79 的「陷阱分支只看」仍漏掉其他必要条件。即便陷阱已布置、未弹起、未设 `targettag` 且掷中概率，附近有危险时仍不会被选作落点，L92 已写出这一条件。L92 还把可食诱饵的「落点可通行」写成必需，源码允许鸟有 `floater` 时跳过通行检查。验收在 L68、L79 去掉穷尽式措辞并与 L92 统一，L92 补 `floater` 例外。保留盐晶不会成为可食诱饵的结论。[盐晶页](../../../wiki/reference/prefabs/saltrock.md)、[两条落点分支](../../../raw/current/scripts/components/birdspawner.lua#L534-L560) |

## 第三轮修订回执

按第二轮复验的待修项回填。

| 编号 | 写作修订回执 | 提交时状态 |
|---|---|---|
| R08 | 已改：clockwork L83 小结改为「有保底来源：双恐怖之眼（twinofterror1／twinofterror2，每只至少 2 个）与检查眼镜补给箱 2（inspectaclesbox2，至少 1 个）；其余靠拆发条／废料与概率」，与 L87、L91 统一。依据 inspectaclesbox.lua#L21-L28、#L458-L466。 | 待复验 |
| R13 | 已改：saltrock L68 去掉「只看」并补 IsDangerNearby 为必要条件；L79 导航行同步「陷阱状态、targettag（可空）、周边无危险」；L92 可食诱饵分支补「鸟带 floater 或落点可通行」例外；「盐晶不会被当作可食诱饵」结论保留。依据 birdspawner.lua#L534-L560。validate_page／source_links 通过。 | 待复验 |

## 第三轮外部定向复验

2026-11-10，Steam 安装清单与源码快照均为 BUILD `26474873`。按第三轮回执复核两项修订及相邻断言，并对两页和制作材料索引做严格源码链接检查，230 处引用 0 错误、0 警告。未运行游戏。

| 编号 | 结论 | 定向复验证据 |
|---|---|---|
| R08 | 通过 | 发条零件 L83 小结与 L87、L90–91 一致。双恐怖之眼两只各至少 2 个电子元件，箱 2 至少 1 个，箱 1 无保底。索引的「每只保底 2 个」仍明确对应双恐怖之眼。[发条零件](../../../wiki/reference/prefabs/clockwork-parts.md)、[制作材料索引](../../../wiki/reference/prefabs/crafting-materials-index.md)、[两类箱掉落表](../../../raw/current/scripts/prefabs/inspectaclesbox.lua#L12-L28)、[两类箱挂表](../../../raw/current/scripts/prefabs/inspectaclesbox.lua#L445-L466)、[双眼表一](../../../raw/current/scripts/prefabs/eyeofterror.lua#L545-L558)、[双眼表二](../../../raw/current/scripts/prefabs/eyeofterror.lua#L615-L628) |
| R13 | 通过 | 盐晶 L68、L79、L92 已统一陷阱状态、目标标签和危险度条件。附近有危险时，即使未设目标标签且掷中概率，也不会经该分支选作落点。可食诱饵分支已补「鸟有 `floater` 或落点可通行」，两条分支仍受共用的 `eater`／`ignorebait` 门禁约束。正文和索引保留盐晶不会成为可食诱饵的结论。[盐晶](../../../wiki/reference/prefabs/saltrock.md)、[制作材料索引](../../../wiki/reference/prefabs/crafting-materials-index.md)、[两条落点分支](../../../raw/current/scripts/components/birdspawner.lua#L534-L560) |
