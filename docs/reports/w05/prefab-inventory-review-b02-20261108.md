# B02 农田作物生熟态独立检视

- 日期 2026-11-08，源码 `steam-buildid:26335394`。
- 覆盖 3 页、14 种作物、28 个生熟注册，以及暂行索引和 `giant-crops` 回链。逐项核对三维与基础新鲜度、堆叠档位、简中名和注册归属。28 个简中主名称与别名、索引一致，注册名未发现跨页重复。
- 结论 **需修复，暂不收口**。P1 × 2，P2 × 8，P3 × 1。本轮核对 Markdown 与源码，沿用交付方已记录的机械验收，未重跑校验器或启动游戏。

## 待修问题

| 编号 | 级别 | 位置 | 核实结果与修订范围 |
|---|---|---|---|
| R01 | P1 | [蔬果页 L117、L177、L184](../../../wiki/reference/prefabs/farm-crops-fruit.md#西瓜生食降温)、[索引 L50](../../../wiki/reference/prefabs/index.md#基础材料与常用物品暂行) | `−40` 加到合成的温差项 `self.delta`，之后才换算为实际速率。暖环境降温常规夹在 −1 ℃/秒，但体温达到过热阈值时走 −5 ℃/秒分支，冷环境另受保温限制。因而「最多约 5 ℃」「−40 与 −20 等效」「只能改时长」均不能作为通则。保留缩短 5→2.5 秒窗口的示例，去掉固定降幅承诺，正文、陷阱和索引一并改。 [温差叠加](../../../raw/current/scripts/components/temperature.lua#L417-L423)、[速率分支](../../../raw/current/scripts/components/temperature.lua#L488-L514)、[1 与 5 的常量](../../../raw/current/scripts/tuning.lua#L2306-L2307) |
| R02 | P1 | [蔬果页 L123–L127](../../../wiki/reference/prefabs/farm-crops-fruit.md#火龙果的万圣月变异)、[粮菜页 L135](../../../wiki/reference/prefabs/farm-crops-staple.md#胡萝卜的旋转变异与修补) | 变异成功生成并摆放生物后，若原物品有持有者，组件只调用该物品栏或容器的 `GiveItem`。沙拉蝾螈没有 `inventoryitem`，玩家物品栏会直接拒绝，普通容器走失败定位分支，不能写成「新生物回到容器」。无容器时才走默认 `attacked` 分支。`DECOR` 是动作候选门禁，执行函数另外判断目标为空、燃烧、冒烟或冻结。粮菜页也应把「转交」收窄为尝试。 [生成与交付](../../../raw/current/scripts/components/halloweenmoonmutable.lua#L45-L77)、[沙拉蝾螈组件](../../../raw/current/scripts/prefabs/fruitdragon.lua#L413-L488)、[物品栏拒绝](../../../raw/current/scripts/components/inventory.lua#L1029-L1033)、[容器失败定位](../../../raw/current/scripts/components/container.lua#L558-L566)、[候选](../../../raw/current/scripts/componentactions.lua#L1340-L1344)、[执行](../../../raw/current/scripts/actions.lua#L4601-L4614) |
| R03 | P2 | [粮菜页 L46](../../../wiki/reference/prefabs/farm-crops-staple.md#获得与入栏)、[蔬果页 L62、L152](../../../wiki/reference/prefabs/farm-crops-fruit.md#获得与种子权重) | `FisherPig` 定义确实有 `unknown_fruit` 池和一个布局位置。以注册名检索整个 `raw/current/scripts`，找到该定义与森林 blockers 的三条已注释引用，未确认现行世界生成选择它。两页应写「布局候选，实际放置未核验」或删去该获取项，并把「唯一的正常世界来源植株」改成已核验的农田来源。 [布局定义](../../../raw/current/scripts/map/layouts.lua#L318-L342)、[注释引用](../../../raw/current/scripts/map/rooms/forest/blockers.lua#L171)、[注释引用](../../../raw/current/scripts/map/rooms/forest/blockers.lua#L186)、[注释引用](../../../raw/current/scripts/map/rooms/forest/blockers.lua#L204) |
| R04 | P2 | [粮菜页 L22、L126–L128、L172](../../../wiki/reference/prefabs/farm-crops-staple.md#南瓜的活动期分支) | 保留「执行文件时计算、构造时抄值、改活动变量本身不会回写已有实例」。常规 Steam 加载流程确实先重载 prefab 再应用存档覆盖，但重载会重新执行文件并重建全局 `VEGGIES`，不能说「进程内固定」。脚本也只显示 `SimReset` 将旧活动作为 `Settings` 参数交给引擎，未证明切换世界时构造必读到上个存档的全局值。删掉这条推断，保留加载时序及边界。 [加载前后端](../../../raw/current/scripts/gamelogic.lua#L257-L262)、[执行 prefab 文件](../../../raw/current/scripts/mainfunctions.lua#L171-L189)、[SimReset](../../../raw/current/scripts/mainfunctions.lua#L1548-L1567)、[先加载再初始化](../../../raw/current/scripts/gamelogic.lua#L1047-L1055)、[活动覆盖](../../../raw/current/scripts/gamelogic.lua#L364-L374) |
| R05 | P2 | [粮菜页 L44–L47](../../../wiki/reference/prefabs/farm-crops-staple.md#获得与入栏) | 胡萝卜获取表漏了普通兔人死亡随机掉落，和表内的兔人小游戏是两条路径。土豆漏了沃利默认开局给 2 个。两条都有具体实现，应补上，土豆的「同 corn」一并收窄。 [普通兔人死亡表](../../../raw/current/scripts/prefabs/bunnyman.lua#L303-L317)、[接入](../../../raw/current/scripts/prefabs/bunnyman.lua#L613)、[沃利默认物品](../../../raw/current/scripts/tuning.lua#L3466)、[模式取表](../../../raw/current/scripts/prefabs/warly.lua#L25-L26) |
| R06 | P2 | [辛香页 L93](../../../wiki/reference/prefabs/farm-crops-spice.md#获得路径)、[蔬果页 L64](../../../wiki/reference/prefabs/farm-crops-fruit.md#获得与种子权重) | 「具体种子只能从上述收获里拿」被鸟笼实证反驳，普通笼鸟接受带同名 `_seeds` 注册的生作物并产对应种子。蔬果页应将农田随机种子的当季倍权与 `seeds.lua` 的 `plantable.product` 加权区分开，后者仅按基础权重。 [鸟笼产出与收取](../../../raw/current/scripts/prefabs/birdcage.lua#L131-L174)、[农田加权](../../../raw/current/scripts/prefabs/farm_plants.lua#L134-L146)、[基础权重抽取](../../../raw/current/scripts/prefabs/seeds.lua#L43-L58)、[两个入口的接线](../../../raw/current/scripts/prefabs/seeds.lua#L160-L173) |
| R07 | P2 | [蔬果页 L105–L107](../../../wiki/reference/prefabs/farm-crops-fruit.md#榴莲的怪物食性) | 两个榴莲使 `monster >= 2`，但怪物千层饼还要求 `not tags.inedible`。加入树枝可直接挡住该配方。「再加榴莲就会触发」同样需要补门禁。优先级 10 只在配方通过后参与排序，仍受其他候选配方的优先级约束。 [千层饼判定](../../../raw/current/scripts/preparedfoods.lua#L308-L321)、[树枝分类](../../../raw/current/scripts/cooking.lua#L134-L135)、[候选排序](../../../raw/current/scripts/cooking.lua#L238-L266) |
| R08 | P2 | [粮菜页 L143、L147、L162–L166](../../../wiki/reference/prefabs/farm-crops-staple.md#统一堆叠上限并分别设定保质期)、[蔬果页 L177](../../../wiki/reference/prefabs/farm-crops-fruit.md#改熟番茄的保鲜与生西瓜的降温窗口) | 两页已修正 postinit 先于读档的顺序，但新鲜度读值实际为 `min(1, 存档剩余秒数 / 新上限)`。把熟番茄从 10 天改 6 天正好可出现读档剩余大于新上限的情形。粮菜示例还声称两件改成同一档，代码却分别传 10 天和 6 天，且熟南瓜原本就是 6 天。让示例目标和实参一致，或明确两者只统一堆叠。 [读档](../../../raw/current/scripts/components/perishable.lua#L337-L349)、[读值上限](../../../raw/current/scripts/components/perishable.lua#L215-L221)、[南瓜基础值](../../../raw/current/scripts/prefabs/veggies.lua#L56-L58) |
| R09 | P2 | [粮菜页 L32、L108、L122](../../../wiki/reference/prefabs/farm-crops-staple.md#关联机制导航) | 熟态的动画 bank／build 沿用原作物名 `name`，例如 `carrot_cooked` 用 `carrot`。公共端显式赋的 `pickupsound="vegetation_firm"` 只在生件 `fn` 里出现。按生熟分开写，避免「与注册名同名」和「生熟都有」误指这两个配置。 [生件动画与音效](../../../raw/current/scripts/prefabs/veggies.lua#L553-L557)、[熟件动画与端分界](../../../raw/current/scripts/prefabs/veggies.lua#L707-L737) |
| R10 | P2 | [粮菜页 L175](../../../wiki/reference/prefabs/farm-crops-staple.md#常见陷阱与限制)、[辛香页 L61、L145](../../../wiki/reference/prefabs/farm-crops-spice.md#数值与参数) | 常规腐坏折扣会跳过负生命／理智，但可处理腐烂食物的 eater 分支会把负值钳为 0，且普通腐烂食物也可能将非负理智改成负值。将「腐坏只打折正收益」改为普通吃者的具体规则，辛香页的「原样返回」和实扣数字限定为无额外修正的基准。 [理智](../../../raw/current/scripts/components/edible.lua#L72-L93)、[生命](../../../raw/current/scripts/components/edible.lua#L121-L142)、[吃者结算](../../../raw/current/scripts/components/eater.lua#L211-L235) |
| R11 | P3 | [粮菜页 L135](../../../wiki/reference/prefabs/farm-crops-staple.md#胡萝卜的旋转变异与修补)、[蔬果页 L22、L121–L125](../../../wiki/reference/prefabs/farm-crops-fruit.md#火龙果的万圣月变异)、[索引 L50](../../../wiki/reference/prefabs/index.md#基础材料与常用物品暂行) | 28 个本批主名称都正确。关键关联对象宜在首次说明时使用游戏里的「月亮精华液」「沙拉蝾螈」「胡萝卜鼠」，旁注注册名即可。「月之药水」「月光药水」目前分别用于两页描述同一物品，影响用实显名称定位。 [月亮精华液](../../../raw/current/scripts/languages/chinese_s.po#L343600-L343602)、[沙拉蝾螈](../../../raw/current/scripts/languages/chinese_s.po#L343040-L343042)、[胡萝卜鼠](../../../raw/current/scripts/languages/chinese_s.po#L340635-L340637) |

## 修订范围

粮菜页 32,757 B、蔬果页 32,760 B，距 standard 的 32 KiB 上限分别剩 11 B 和 8 B。修订优先替换或压缩原句，避免叠加解释。暂行索引仍覆盖三页，巨型作物回链已就位。

首次检视时，[维护日志](../../../wiki/log.md) 已达 67,039 B，超过 64 KiB。收口时按日志规则先分卷，再写收口条目。

## 修订后复检（2026-11-08）

两项 P1 的核心问题均已修正：西瓜的固定降幅承诺已从正文、示例和索引移除，火龙果变异也已区分物品栏拒绝和普通容器失败定位。R03、R05、R06、R09、R11 可结项。修订方已将日志分卷至 [2026-11-08 第一卷](../../../wiki/log-archive/log-2026-11-08-001.md)。

| 编号 | 复检位置 | 尚需修改 |
|---|---|---|
| R01 | [蔬果页 L117、L177、L184](../../../wiki/reference/prefabs/farm-crops-fruit.md#西瓜生食降温) | 速率的 1／5 ℃/秒是暖环境且净温差走降温分支时的限幅，`max(self.delta, -上限)` 允许比这更慢。示例和陷阱把它写成「常规 1、过热 5」，仍像实际速率。统一为「暖环境降温上限分别为 1／5 ℃/秒」，正文「夹到」同步改成「上限」。[温度更新](../../../raw/current/scripts/components/temperature.lua#L488-L514) |
| R02 | [粮菜页 L129](../../../wiki/reference/prefabs/farm-crops-staple.md#胡萝卜的旋转变异与修补)、[蔬果页 L123](../../../wiki/reference/prefabs/farm-crops-fruit.md#火龙果的万圣月变异) | 胡萝卜鼠确实有 `inventoryitem`，但原容器存在时只尝试 `GiveItem`，代码没有检查返回值。「失败或无容器时改推 attacked」要改成「无容器且标志开启时才推 attacked」。蔬果页仍把 `DECOR` 和燃烧／冻结写成同一层执行门禁，需区分候选和服务端执行，后者还会拒绝空目标。[交付分支](../../../raw/current/scripts/components/halloweenmoonmutable.lua#L61-L75)、[胡萝卜鼠组件](../../../raw/current/scripts/prefabs/carrat.lua#L719-L723)、[候选](../../../raw/current/scripts/componentactions.lua#L1340-L1344)、[执行](../../../raw/current/scripts/actions.lua#L4601-L4614) |
| R04 | [粮菜页 L120–L122、L166](../../../wiki/reference/prefabs/farm-crops-staple.md#南瓜的活动期分支) | 已删除进程固定和继承上一存档的推断。陷阱仍写「新建物品取的是下次加载前的全局值」，应写当前工厂表已算好的值。活动变量改动不会自动重算 `VEGGIES.pumpkin.perishtime`，重跑文件时才会用当时的活动状态重新求值。[表初始化](../../../raw/current/scripts/prefabs/veggies.lua#L39-L58)、[构造取值](../../../raw/current/scripts/prefabs/veggies.lua#L602-L605) |
| R07 | [蔬果页 L105–L107](../../../wiki/reference/prefabs/farm-crops-fruit.md#榴莲的怪物食性) | 千层饼行已补 `not tags.inedible`，果盘行仍写「再加榴莲就会触发上一条」。两个榴莲、一个石榴和一根树枝仍可通过果盘的三份水果条件，同时被千层饼拒绝。果盘行只需说明多加榴莲会同时增加 `monster`，料理结果交给上表的门禁与优先级。[果盘判定](../../../raw/current/scripts/preparedfoods.lua#L265-L279)、[千层饼判定](../../../raw/current/scripts/preparedfoods.lua#L308-L321) |
| R08 | [粮菜页 L137、L141、L156–L160](../../../wiki/reference/prefabs/farm-crops-staple.md#统一堆叠上限并分别设定保质期) | 示例仍有三处互相冲突。注释写生熟保质期统一，实参分别为 10／6 天，说明又称两者都沿用原档。代码实际会把活动期生南瓜的 20 天改为 10 天。建议说明「统一堆叠 40，分别设为 10／6 天」，或移除保质修改以保留两态的实际原档。前置读档说明还写未封顶的商，合并为一次准确说明。读取比例在 1 封顶，`OnLoad` 本身照抄剩余秒数。[南瓜取值](../../../raw/current/scripts/prefabs/veggies.lua#L56-L58)、[比例读值](../../../raw/current/scripts/components/perishable.lua#L215-L221)、[读档](../../../raw/current/scripts/components/perishable.lua#L337-L349) |
| R10 | [粮菜页 L169](../../../wiki/reference/prefabs/farm-crops-staple.md#常见陷阱与限制)、[辛香页 L22、L61](../../../wiki/reference/prefabs/farm-crops-spice.md#数值与参数) | 钳零分支已补。粮菜页标题仍说「腐坏只打折正收益」，普通吃者吃非负理智的腐烂蔬菜时也可能扣理智。把标题收窄为「通常不减轻原有负值」。辛香页「生吃一次扣」和按默认生命／理智算的实扣应标为无额外修正时的基准，钳零条件明确用「该物品满足 `CanProcessSpoiledItem`」。[理智取值](../../../raw/current/scripts/components/edible.lua#L72-L93)、[腐烂处理门禁](../../../raw/current/scripts/components/eater.lua#L156-L160)、[结算修正](../../../raw/current/scripts/components/eater.lua#L198-L235) |

结论仍为待复检。本轮只更新报告，未改知识页或台账，未重跑校验器或启动游戏。

## 二次修订复检（2026-11-08）

R02、R04、R07 已通过。变异交付和动作门禁、南瓜活动取值、榴莲配方的说明均已和源码对齐。仍有三项局部问题，台账保持「待外部复检」。

| 编号 | 位置 | 剩余问题 |
|---|---|---|
| R01 | [蔬果页 L117、L177、L184](../../../wiki/reference/prefabs/farm-crops-fruit.md#西瓜生食降温) | 已明确实际降温可以更慢，但 L117 把限幅写成负数（−1／−5），公式里又写 `max(delta, -上限)`，按字面代入会变成升温。统一用正的降温幅度上限 1／5 和 `max(delta, -上限)`。L177、L184 可压为「暖环境下，未过热／过热上限为 1／5 ℃/秒」，避免把 5 读成跨环境规则。[实际限幅](../../../raw/current/scripts/components/temperature.lua#L488-L514) |
| R08 | [粮菜页 L137、L160](../../../wiki/reference/prefabs/farm-crops-staple.md#统一堆叠上限并分别设定保质期) | 统一堆叠与分别设为 10／6 天的示例已一致。L137 仍把读档比例写成「旧剩余÷新上限」，并且原来 L160 的封顶说明已删。把 L137 改为「OnLoad 照抄存档剩余秒数，GetPercent 读取比例时在 1 封顶」，将 L160 的 GetPercent 锚点移到该句。[读档](../../../raw/current/scripts/components/perishable.lua#L337-L349)、[比例读值](../../../raw/current/scripts/components/perishable.lua#L215-L221) |
| R10 | [粮菜页 L169](../../../wiki/reference/prefabs/farm-crops-staple.md#常见陷阱与限制)、[辛香页 L61](../../../wiki/reference/prefabs/farm-crops-spice.md#数值与参数) | 粮菜页把 `-SANITY_SMALL` 标成 5。源码常量是 10，5 对应 `SANITY_TINY`。普通吃者满足腐烂分支时直接以 −10 替换原先的非负理智取值。辛香页的快速判断和陷阱已经限定新鲜态、无额外修正，数值段 L61 仍以角色初始属性直接推实际扣量，补上同一限定即可。[腐烂理智取值](../../../raw/current/scripts/components/edible.lua#L72-L93)、[常量](../../../raw/current/scripts/tuning.lua#L2033-L2035) |

本轮只更新复检报告，未修改知识页或台账，未重跑校验器或启动游戏。

## 收口（2026-11-08）

最终逐项复验 R01–R11。西瓜正文、示例和陷阱的限幅符号及适用范围一致，南瓜读档区分保存秒数和读取比例，腐烂理智已校准为 −10。收口时两处措辞直接对齐源码，粮菜页明确腐烂分支会替换原先非负的理智取值，辛香页示例限定为新鲜态且无额外修正的威尔逊。其余修订及暂行索引、巨型作物回链复验通过。

B02 三页、28 注册已收口。机械验收沿用修订方的定向结果，最后两处只调整正文措辞。未重跑校验器或启动游戏。
