# B08 保质材料、植物产物与特殊消耗外部源码检视

**当前结论为外部源码复验通过，可登记收口。** 第三轮定向复验确认 R11 通过，R01–R19 累计 19 项通过。初检为 P1 5 项、P2 13 项、P3 1 项。[第三轮外部定向复验](#第三轮外部定向复验)

2026-11-10，Steam 安装清单和 [`raw/current/BUILD.txt`](../../../raw/current/BUILD.txt) 均为 `26474873`。逐页核对 [B08 台账](prefab-inventory-ledger-20261108.md) 的 11 页、[暂行索引](../../../wiki/reference/prefabs/materials-consumables-index.md) 和直接相关的跨页断言。下列 Wiki 行号对应初检工作树。检视只写本报告，知识页、索引和台账供修订方按编号处理。

## P1

| 编号 | 位置 | 问题、源码证据和修订验收 |
|---|---|---|
| R01 | [迷你冰川](../../../wiki/reference/prefabs/rock-ice.md#典型接入) L128–144 | 示例把调小阈值写成「春夏更早融、秋冬更晚长」。春、秋确实用同一组阈值，但两者都更早跨阈值，春更早降档、秋也更早升档。夏的目标恒为 `dryup`，冬的目标恒为 `tall`。验收可取 `seasonprogress=0.35`，示例的 `th1=0.30` 已越过第一档，秋季目标是 `short`，原版 `th1≥0.40` 仍以 `empty` 为目标。保留一次回调最多升降一级和等待计时器的条件。[四季映射](../../../raw/current/scripts/prefabs/rock_ice.lua#L218-L257)、[逐级夹取](../../../raw/current/scripts/prefabs/rock_ice.lua#L102-L132) |
| R02 | [晶洞果与夜莓](../../../wiki/reference/prefabs/ancient-fruit.md#夜莓生熟两态三维腐坏与堆叠) L6、L78–83、L95、L168–170；[索引](../../../wiki/reference/prefabs/materials-consumables-index.md) L21 | 正文将 `DoFoodEffects` 写成负值食物「才吞得下」的门禁，且摘要把生熟夜莓的 `cookable`／夜视混在一起。`Eat` 以 `PrefersToEat` 进主分支，`DoFoodEffects` 只决定负生命／负理智是否计入三维，强健胃、生肉耐受及食物亲和的相应命中反而跳过负效果。原版只有生夜莓挂 `cookable` 和 `NightVision_OnEaten`，后者还要求进食者有 `playervision` 才尝试加 buff。验收时摘要、数值段、陷阱和索引均能区分「吃不吃」「是否结算负值」和「生莓才给夜视」。[负效果判断](../../../raw/current/scripts/components/eater.lua#L166-L170)、[进食与三维](../../../raw/current/scripts/components/eater.lua#L198-L235)、[食用回调](../../../raw/current/scripts/components/eater.lua#L263-L276)、[生莓接线](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L446-L462)、[熟莓组件](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L498-L513)、[夜视门禁](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L339-L348) |
| R03 | [蘑菇树孢子](../../../wiki/reference/prefabs/mushtree-spores.md#月亮孢子差异) L102、L106、L119；[索引](../../../wiki/reference/prefabs/materials-consumables-index.md) L20；[蘑菇帽入口](../../../wiki/reference/prefabs/hats-environment.md) | 「月亮孢子砍后才喷」只描述月亮蘑菇树的启动路径。原版月亮蘑菇帽也能生成同一个 `spore_moon`，装备时启动 `periodicspawner`，生成回调还置 `_alwaysinstantpops=true`，因此帽产孢子触发邻近判定时跳过 26 帧 `pre_pop`，直接进 `pop`。验收应能从孢子页和索引找到两条来源，并分清普通邻近分支和帽产分支。索引的 NET 捕捉、地面约 60 秒／常温持有 15 天和农场种子应限定彩色孢子。[树砍伐启动](../../../raw/current/scripts/prefabs/mushtree_moon.lua#L155-L166)、[帽子装备启动](../../../raw/current/scripts/prefabs/hats.lua#L1871-L1889)、[帽产回调与注册接线](../../../raw/current/scripts/prefabs/hats.lua#L1937-L1973)、[孢子邻近门禁](../../../raw/current/scripts/prefabs/mushtree_moonspore.lua#L61-L79)、[两种事件与状态](../../../raw/current/scripts/stategraphs/SGmoonspore.lua#L3-L32) |
| R04 | [果蝇王与果蝇](../../../wiki/reference/prefabs/fruit-fly.md#果蝇王由-farming_manager-的计时器放行一次只一只) L43–57、L152–153；[索引](../../../wiki/reference/prefabs/materials-consumables-index.md) L22 | 「35 天内刷不出」忽略了作物腐烂会把活动计时器每次前拨 240 秒。L55 又把「到期、有成熟作物事件、场上没王」写成充分条件，漏掉作物唤醒、`SPAWN_LORDFRUITFLY`、首轮玩家距离和 4 距离内至少 15 个 `fruitflyspawner`。玩家 15 距离门禁只对 `check_others=true` 的首轮作物检查生效，后续邻株递归带 `false`。验收用两个反例，腐烂可把首刷闸门提前打开，开门后单株成熟仍不刷。[首刷与前拨](../../../raw/current/scripts/components/farming_manager.lua#L74-L86)、[生成门禁与递归](../../../raw/current/scripts/components/farming_manager.lua#L217-L236)、[成熟与腐烂事件](../../../raw/current/scripts/prefabs/farm_plants.lua#L520-L543) |
| R05 | [双尾心](../../../wiki/reference/prefabs/wortox-reviver.md#腐坏与返还灵魂) L6、L57–69、L99–108；[索引](../../../wiki/reference/prefabs/materials-consumables-index.md) L25 | 鲜度链漏掉施法中断。`crushitemcast.onenter` 先停表。若执行动作前被打断（例如满足全局 `attacked` 的 `electrocute` 分支），走 `not crushcasting` 的 `onexit`，只清符号和 body，既不消耗也不重启鲜度。它与已执行动作后进入 `crushitemcast_fail` 的分支不同，后者在 `onenter` 重启。双尾心的入包回调只绑主人，暂停也会被 `perishable` 存档保留。验收明确区分提前打断、动作失败和成功三路，把 10 天标成正常计时的定义值。[停表](../../../raw/current/scripts/stategraphs/SGwilson.lua#L16648-L16668)、[电击中断](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1905-L1933)、[提前离开与失败复表](../../../raw/current/scripts/stategraphs/SGwilson.lua#L16692-L16726)、[入包回调](../../../raw/current/scripts/prefabs/wortox_reviver.lua#L33-L49)、[暂停存读](../../../raw/current/scripts/components/perishable.lua#L329-L349) |

## P2

| 编号 | 位置 | 问题、源码证据和修订验收 |
|---|---|---|
| R06 | [冰](../../../wiki/reference/prefabs/ice.md#名称与获得) L38、L88；[索引](../../../wiki/reference/prefabs/materials-consumables-index.md) L15 | 「获得只有一条主链」收得过窄。仅 `mutateddeerclops` 的实际掉落表就有 1 件必掉冰和另 1 件 75% 的冰。`rg -n -F '"ice"' raw/current/scripts/prefabs raw/current/scripts/components` 也命中其他候选，本项以有消费者的掉落表作反例。验收可把迷你冰川明确为本文展开的来源，并补一个非迷你冰川实证，无需列齐其他来源。[掉落表](../../../raw/current/scripts/prefabs/deerclops.lua#L343-L354)、[表的接线](../../../raw/current/scripts/prefabs/deerclops.lua#L649-L654) |
| R07 | [冰](../../../wiki/reference/prefabs/ice.md#数值与结算) L52–54、L60、L79–81；[小痒痒](../../../wiki/reference/prefabs/mosquitomusk.md#修理用蚊血囊刷新鲜度) L47–51 | 修理候选不是目标带 `fresh`／`stale`／`spoiled` 即可。候选要求右键、材料标签匹配，以及骑乘时目标归自己或通过对应的搬重物分支；满鲜仍可能出现候选，结算才拒绝。冰页的明火动作也应按目标 `IsHeld()` 表述，范围比「拿在手上」广，且 `smolder` 先于 `elseif` 的明火候选。验收举普通鲜食、95% 冰帽、满鲜小痒痒三个例子，分别区分无修理候选、可成功补鲜、候选出现但不消耗材料。[修理候选](../../../raw/current/scripts/componentactions.lua#L1679-L1700)、[修理结算](../../../raw/current/scripts/components/repairable.lua#L126-L179)、[灭火候选](../../../raw/current/scripts/componentactions.lua#L1736-L1743) |
| R08 | [荧光果](../../../wiki/reference/prefabs/lightbulb.md#让一块荧光果给灯具补两倍燃料) L22、L47、L91–108 | 倍燃料示例的 180 点验收少了剩余容量门槛。`TakeFuelItem` 先乘接收方、潮湿和熟练倍率，再交 `DoDelta` 夹到 `maxfuel`。用干燥荧光果、无额外倍率且空余至少 180 点的提灯验收，才可断言 `currentfuel` 增加 180。也请把「背包里的立即生效」明确为由该 postinit 构造（含读档重建）的实例。[单件燃料计算](../../../raw/current/scripts/components/fueled.lua#L197-L221)、[容量夹取](../../../raw/current/scripts/components/fueled.lua#L305-L323)、[构造默认燃料](../../../raw/current/scripts/prefabs/lightbulb.lua#L56-L58) |
| R09 | [南瓜灯](../../../wiki/reference/prefabs/pumpkin-lantern.md) L6、L57–61、L114–116；[索引](../../../wiki/reference/prefabs/materials-consumables-index.md) L18 | 页内「只有非白天才亮」漏掉入昼后还等 `2 + random()` 秒才开始淡出，索引的「夜间」又排除了同样满足 `not isday` 的黄昏。索引还把腐坏写成「破碎」，而 `onperish` 先播 `rotten`，死亡回调在这种路径跳过 `broken`。验收覆盖地面黄昏、切回白天的延迟与两条死亡动画，保留两路都掉萤火虫。[昼夜与落地](../../../raw/current/scripts/prefabs/pumpkin_lantern.lua#L132-L166)、[两条死亡路径](../../../raw/current/scripts/prefabs/pumpkin_lantern.lua#L113-L130) |
| R10 | [格罗姆花与格罗姆](../../../wiki/reference/prefabs/glommer-flower.md#让死花真正开始腐坏) L118–119、L155、L173–176 | 示例的 `OnLoad` 补启动位置正确，停用后的描述仍前后相反。模组启用期间已经计时的死花保存 `time`，且不写 `paused`。停用后读取这份存档，原版花的 `OnPreLoad` 仍重建 `perishable`，组件的 `OnLoad` 会自行恢复任务。验收用「原版暂停旧档→启用模组→计时后存档→停用并加载」复述结果。6 天是定义值，地面无其他修正时约 4 天到零，原版死花也没有替换物自动删除分支。[死态重建](../../../raw/current/scripts/prefabs/glommerflower.lua#L7-L15)、[预载](../../../raw/current/scripts/prefabs/glommerflower.lua#L61-L69)、[组件存读](../../../raw/current/scripts/components/perishable.lua#L329-L349)、[地面倍率与扣减](../../../raw/current/scripts/components/perishable.lua#L96-L133) |
| R11 | [晶洞果与夜莓](../../../wiki/reference/prefabs/ancient-fruit.md#晶洞果加热判定与三段阈值定义处--结算处) L47–55、L165、L175 | `heatindex` 不是遍历完所有热源才求和，每次累加后首次达 40 就升温并 `break`。有正负热源时遍历顺序能决定后面的负值是否被读到；温度变化也要等 2 秒轮询或出 limbo 合并结算。验收在表里写明提前停止，在约 64 秒的推导旁标初始温度为零、第一轮在有效热源内、后续每次判定均满足升温、单果的条件。[热量累加及提前跳出](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L43-L99)、[出 limbo](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L102-L131)、[首轮任务](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L297-L307) |
| R12 | [蘑菇树孢子](../../../wiki/reference/prefabs/mushtree-spores.md) L25、L55、L78、L106、L113、L116–118、L129、L139；[索引](../../../wiki/reference/prefabs/materials-consumables-index.md) L20 | 生成侧密度门和孢子自身的拥挤检测用了不同的集合。`periodicspawner` 看宿主附近同 prefab 的数量（普通达到 10、酸雨改 15 时拒绝），不是整个场景的稳态上限。密度挡住时 `TrySpawn` 仅返回 `false`，`DoSpawn` 会按默认随机间隔重排，月亮蘑菇树没有文中的约 10 秒密度重试。孢子侧查所有 `spore` tag，大于 10 只设 0% 并重新排一次腐坏更新，之后才 `Perish`、推 `death` 或 `pop`、清 tag。验收同时说明同色生成门、多色共用的拥挤门，以及 `SetPercent(0)` 到实际删除之间的时序。[生成门与重排](../../../raw/current/scripts/components/periodicspawner.lua#L139-L170)、[DoSpawn](../../../raw/current/scripts/components/periodicspawner.lua#L199-L203)、[彩色拥挤检测](../../../raw/current/scripts/prefabs/mushtree_spores.lua#L34-L56)、[月亮拥挤检测](../../../raw/current/scripts/prefabs/mushtree_moonspore.lua#L14-L46)、[设百分比](../../../raw/current/scripts/components/perishable.lua#L223-L233)、[排更新](../../../raw/current/scripts/components/perishable.lua#L262-L271) |
| R13 | [果蝇王与果蝇](../../../wiki/reference/prefabs/fruit-fly.md) L6、L61–72、L76、L157–158；[索引](../../../wiki/reference/prefabs/materials-consumables-index.md) L22 | 三个短句删掉了紧邻的条件。①「已有友好果蝇只掉种子」仍保留共享表的 1 个 `plantmeat`，变动的只是额外掉落。②「王死后转敌对」应在摘要、标题和索引同步带 `hascausedhavoc`，还要当前没有 leader。③ L76「已有 leader 就跳过」与 L157 相冲突。`OnInit` 只跳过 leader 已经是本果的情况，绑定了别的果会转绑，旧果的 `onremovefollower` 仍可使旧果变干壳。验收分别核对有／无友好果蝇的掉落、未捣乱的脱队小果蝇、两枚活果的改绑。[共享掉落表](../../../raw/current/scripts/prefabs/fruitfly.lua#L51-L54)、[额外掉落](../../../raw/current/scripts/prefabs/fruitfly.lua#L110-L120)、[转敌对门禁](../../../raw/current/scripts/prefabs/fruitfly.lua#L274-L289)、[活果复绑](../../../raw/current/scripts/prefabs/fruitfly.lua#L533-L544)、[旧 leader 移除回调](../../../raw/current/scripts/components/follower.lua#L293-L328) |
| R14 | [月熠](../../../wiki/reference/prefabs/moonstorm-spark.md) L38–40、L48–50、L86、L93、L98；[索引](../../../wiki/reference/prefabs/materials-consumables-index.md) L23 | 索引「地面 360 秒」与正文相反。360 秒是定义值，地面无其他修正约 240 秒，普通持有者约 60 天。生成处的 3 是每次对各玩家半径 30 重数后放行的阈值，重叠范围的玩家会看到前面已经刷出的月熠，实体移动后密度也会变。捕捉在 `GiveItem` 前补检，不能用这一次捕捉来计算已经释放的名额。验收时索引与正文用同一计时口径，并将「上限／各自叠加」改为实际生成判定。[定义](../../../raw/current/scripts/tuning.lua#L6250)、[腐坏速率](../../../raw/current/scripts/components/perishable.lua#L68-L133)、[拾取补检与入库](../../../raw/current/scripts/prefabs/moonstorm_spark.lua#L118-L129)、[逐玩家重数](../../../raw/current/scripts/components/moonstormmanager.lua#L846-L866) |
| R15 | [月熠](../../../wiki/reference/prefabs/moonstorm-spark.md#电击与充能) L22、L57、L61–62、L70；[充能组件](../../../wiki/reference/components/moonsparkchargeable.md) L40；[索引](../../../wiki/reference/prefabs/materials-consumables-index.md) L23 | 「满值需 10 只火花」把次数写成实体数。一次 `DoSpark` 提高 10 个百分点，调用不消耗火花，同一只可多轮回充，从零到满需 10 次有效调用且期间无消耗。5–15 秒是从一次伤害结算到下次前置 FX 的等待，实际伤害／充能还有 5 帧前摇，0.5 秒缩灯半径任务与下一轮排程并行。验收把「只／次」与两段时间口径分别同步到正文、索引和组件关联句。[排任务与结算](../../../raw/current/scripts/prefabs/moonstorm_spark.lua#L21-L58)、[充能接口](../../../raw/current/scripts/components/moonsparkchargeable.lua#L18-L23) |
| R16 | [小痒痒](../../../wiki/reference/prefabs/mosquitomusk.md) L6、L20、L58–61、L84、L94；[远程武器](../../../wiki/reference/prefabs/ranged-weapons.md) L44、L94；[索引](../../../wiki/reference/prefabs/materials-consumables-index.md) L24 | 「携带即让蚊子与蚊子炸弹跳过」扩大了免疫范围。标签只从 `inventory:FindItem` 命中的候选中排除常规重选和炸弹刷出的蚊子的初始 `SuggestTarget`。蚊子受击仍可 `SetTarget` 和共享攻击者，炸弹命中的独立范围伤害也没有读标签。验收用携带者主动攻击蚊子和炸弹范围命中分别说明例外，同步缩窄摘要、索引和关联页。[常规重选](../../../raw/current/scripts/prefabs/mosquito.lua#L89-L111)、[受击锁定](../../../raw/current/scripts/prefabs/mosquito.lua#L134-L141)、[炸弹选敌](../../../raw/current/scripts/prefabs/mosquitobomb.lua#L45-L70)、[范围伤害](../../../raw/current/scripts/prefabs/mosquitobomb.lua#L75-L88) |
| R17 | [双尾心表现族](../../../wiki/reference/prefabs/wortox_soulecho_buff_fx.md) L62、L76；对照[双尾心](../../../wiki/reference/prefabs/wortox-reviver.md#身体子实体宿主侧挂接) L80–81、L92 | 物品页已修对，表现族页仍写双尾心走 `crushitemcast_holding` 第 9 帧和 `crushitemcast` 第 18 帧。普通 `CASTSPELL` 选 `crushitemcast`，第 9 帧挂 body，第 18 帧只播音。表现族页还把反向监听写成双尾心本体移除时触发，实际监听来源是 body，body 移除后清道具上的引用。验收把表现族同步到物品页帧位并改清理方向，引擎父子回收继续单列待核验。[动作选态](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1303-L1310)、[9／18 帧](../../../raw/current/scripts/stategraphs/SGwilson.lua#L16674-L16689)、[状态收尾](../../../raw/current/scripts/stategraphs/SGwilson.lua#L16705-L16713)、[监听来源](../../../raw/current/scripts/prefabs/wortox_reviver.lua#L180-L198) |
| R18 | [冰](../../../wiki/reference/prefabs/ice.md#只改冰的堆叠上限连带改整叠融化的加湿量) L101、L117、L123；[索引](../../../wiki/reference/prefabs/materials-consumables-index.md) L15 | 示例要求观察土壤湿度按 `25×叠数` 增长，却没限定土壤湿度上限 100。原版 40 叠和示例 60 叠在耕地上都会轻易顶满，所以满叠融化无法用实际湿度增量验证二者差别。验收可用 1–2 块冰在低湿耕地观察，或记录 `AddSoilMoistureAtPoint` 的入参验证按叠数传递；索引明确 25/件是夹取前请求量。[入参](../../../raw/current/scripts/prefabs/inv_rocks_ice.lua#L19-L34)、[土壤更新](../../../raw/current/scripts/components/farming_manager.lua#L461-L470)、[夹取](../../../raw/current/scripts/components/farming_manager.lua#L96-L105)、[上限与冰的系数](../../../raw/current/scripts/tuning.lua#L5780-L5784) |

## P3

| 编号 | 位置 | 问题、证据和修订验收 |
|---|---|---|
| R19 | [格罗姆花与格罗姆](../../../wiki/reference/prefabs/glommer-flower.md) L7、L19、L27、L111；[索引](../../../wiki/reference/prefabs/materials-consumables-index.md) L19；[台账](prefab-inventory-ledger-20261108.md) B08 行 | 11 页的 21 个台账主注册都在同版导出命中，`wortox_reviver_body` 单独命中表现族页。对 B08 `aliases` 与同版 `output/coverage/prefab-26474873-shard-1-aligned/registrations.json` 对账，另命中 `statueglommer`、`glommerfuel`、`glommerwings` 三个注册，当前 Prefab 页的精确别名还没有重复命中，但与「2 个主注册」的口径会混用。验收把黏液与翅膀保留作正文关联入口，把机器匹配的主注册与雕像获得宿主分别说明，并避免 B20／B25 写对应物品页时重复领取注册。[格罗姆产物及依赖](../../../raw/current/scripts/prefabs/glommer.lua#L7-L27)、[雕像注册](../../../raw/current/scripts/prefabs/statueglommer.lua#L295)、[黏液注册](../../../raw/current/scripts/prefabs/glommerfuel.lua)、[翅膀注册](../../../raw/current/scripts/prefabs/glommerwings.lua) |

## 已核验范围

逐项核对本批注册、主要实现和直接消费者，对照摘要、正文、示例、陷阱与暂行索引。以下列出各页的代表核对链。一致结论按这里列出的条件理解，受修订的例外见 R01–R19。

| 页面 | 事实骨架与页面命中情况 |
|---|---|
| 冰、迷你冰川 | 冰同时走 `watersource` 单件消费、`smotherer` 动作与鲜度修理，融化按 owner 分流。冰川普通采掘按级抽 1 至 `icecount`，整级结算跨过的每级分别抽，沉没只给当前级定额。[冰工厂及水源](../../../raw/current/scripts/prefabs/inv_rocks_ice.lua#L19-L51)、[逐级采掘](../../../raw/current/scripts/prefabs/rock_ice.lua#L184-L210)、[沉没](../../../raw/current/scripts/prefabs/rock_ice.lua#L308-L320) |
| 荧光果、南瓜灯 | 荧光果落地／入包回调控制物品自带光，CAVE 燃料与 `lightbattery` 筛选分别消费。南瓜灯入包停表、落地复表，腐坏与受击死亡汇入同一萤火虫掉落。[荧光果工厂](../../../raw/current/scripts/prefabs/lightbulb.lua#L6-L12)、[荧光果组件](../../../raw/current/scripts/prefabs/lightbulb.lua#L56-L80)、[南瓜灯死亡与鲜度](../../../raw/current/scripts/prefabs/pumpkin_lantern.lua#L113-L176) |
| 格罗姆花、果蝇族 | 雕像满月门禁依次看架上花、`spawned`、场上活花，摘下后跟随链转向物品。死花与友好果蝇果干壳都只挂 6 天定义值，没有启动鲜度；果蝇王的掉落在 `GenerateLoot` 前重跑 setup。[满月及摘花](../../../raw/current/scripts/prefabs/statueglommer.lua#L84-L164)、[死花](../../../raw/current/scripts/prefabs/glommerflower.lua#L7-L41)、[干壳](../../../raw/current/scripts/prefabs/fruitfly.lua#L473-L506)、[掉落 setup](../../../raw/current/scripts/components/lootdropper.lua#L186-L192) |
| 蘑菇树孢子、晶洞果与夜莓 | 彩色孢子 NET 入包，月亮孢子 NET 推 `pop`；蘑菇农场有三种彩色映射。晶洞果逐件抽取、按宝石种类合堆，生夜莓重复给同目标的同名 buff 重置为 240 秒。[彩色 NET](../../../raw/current/scripts/prefabs/mushtree_spores.lua#L97-L104)、[月亮 NET](../../../raw/current/scripts/prefabs/mushtree_moonspore.lua#L33-L53)、[农场映射](../../../raw/current/scripts/prefabs/mushroom_farm.lua#L33-L38)、[开壳抽取](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L171-L237)、[buff 重置](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L565-L572) |
| 月熠、小痒痒、双尾心 | 月熠捕捉的实际入口是 NET；入包事件监听生效，`SetOnPutInInventoryFn` 那行是赋值。小痒痒的血囊走鲜度修理，满鲜退回且不扣材料。双尾心复活走幽灵收取，传送和自用返灵魂走施法，腐坏独立返 1 灵魂。[月熠事件接线](../../../raw/current/scripts/prefabs/moonstorm_spark.lua#L208-L240)、[血囊修理量](../../../raw/current/scripts/prefabs/mosquitosack.lua#L35-L40)、[幽灵收取](../../../raw/current/scripts/prefabs/player_common.lua#L333-L366)、[施法及腐坏](../../../raw/current/scripts/prefabs/wortox_reviver.lua#L83-L178) |

关联消费者按断言定向抽查，范围包括果蝇作物事件、冰帽、提灯、蘑菇帽与农场、月风暴管理器与护目镜、蚊子炸弹、双尾心玩家状态机与表现族。未运行游戏。专服或远端客户端表现、标签投影、物理与动画观感、随机刷新节奏和极端环境下的实际秒数仍待运行验证。未把本次源码检视扩展到关联页的其余主题。

## 修订回执与定向复验

请写作者在下方按 R 编号回填实际改动的位置、同事实在页内及索引的同步位置、自验结果或异议证据。共享索引需要联动时，写作者提供已核验的条目文字，由写作主代理合并。既有页需要回源修订时，请在回执标出，由写作主代理派给对应写作者。复验方沿用编号记录「通过／部分修复／仍待修订」及剩余项。

| 编号 | 修订方回执 | 外部复验 |
|---|---|---|
| R01 | 已改：`rock-ice` L130（春秋同阈值、夏恒 dryup／冬恒 tall，补 0.35／0.30 对照，保留一次一级与计时器条件） | 通过。春早降、秋早升、夏冬目标固定，示例用目标阶段区分实际阶段。 |
| R02 | 已改：`ancient-fruit` L6／L78／L83／L168／L169（`PrefersToEat` 与 `DoFoodEffects` 分层、生莓才可烤且需 `playervision`）；索引 L21 | 通过。正文与索引分开写进食、负值结算和生莓夜视门禁。 |
| R03 | 已改：`mushtree-spores` L104／L110／L124、`hats-environment` L127、索引 L20（月亮树与月亮蘑菇帽两条来源、帽产 `_alwaysinstantpops` 跳过 pre_pop） | **部分修复。** 帽产分支和索引的彩色孢子限定已对上。复验扩大 `spore_moon` 引用反查，补到初检漏掉的蘑菇地精 `mushgnome`，其构造启动周期喷发，攻击状态 25–36 帧逐帧直接生成，死亡也从共享表掉落 3 个必出和 2 个各 50%。`mushtree-spores` L110、关联来源及索引 L20 的「两条来源」仍不成立，请加地精及三条入口，帽产跳过 `pre_pop` 继续单列。[周期与掉落接线](../../../raw/current/scripts/prefabs/mushgnome.lua#L185-L199)、[死亡掉落表](../../../raw/current/scripts/prefabs/mushgnome.lua#L18-L29)、[攻击生成函数](../../../raw/current/scripts/stategraphs/SGmushgnome.lua#L79-L85)、[死亡状态](../../../raw/current/scripts/stategraphs/SGmushgnome.lua#L89-L102)、[攻击帧位](../../../raw/current/scripts/stategraphs/SGmushgnome.lua#L199-L240) |
| R04 | 已改：`fruit-fly` L49／L55／L152（腐烂前拨 240 秒、开门后六项必要门、15 距离只首轮 `check_others`） | **部分修复。** 35 天与腐烂前拨已改对。L55 仍把 `check_others=true`、首轮作物本身 4 距离内凑够 15 个写成生成的共同必要条件，和 L45 的递归矛盾。请拆成首轮的玩家门禁、当前候选点的共通门禁与 4 距离计数，明确首轮计数不足时，附近另一候选点可用 `false` 递归通过；索引 L22 同步把 15 个限定在候选点 4 距离内。[实际分支](../../../raw/current/scripts/components/farming_manager.lua#L217-L236) |
| R05 | 已改：`wortox-reviver` L6／L66／L69（提前打断／失败／成功三路与计时） | **部分修复。** 正文三路正确。索引 L25 的「`crushitemcast_fail` 才重启」漏掉两条成功路径，页内验证记录 L118 又把失败写进「重启计时并消耗」。失败重启且保留物品，成功才重启并消耗，请同步两处。[失败](../../../raw/current/scripts/stategraphs/SGwilson.lua#L16716-L16729)、[自用成功](../../../raw/current/scripts/stategraphs/SGwilson.lua#L16752-L16763)、[传送成功](../../../raw/current/scripts/stategraphs/SGwilson.lua#L16860-L16866) |
| R06 | 已改：`ice` L38（补 `mutateddeerclops` 1＋75% 反例，`sources` 补 `deerclops.lua`） | 通过。正文和索引均把采掘作为展开入口，并保留变异巨鹿反例。 |
| R07 | 已改：`ice` L54-L56／L62（右键＋材料标签＋骑乘/搬重、候选不查鲜度）；`mosquitomusk` L6／L49／L51／L104 | **部分修复。** 满鲜拒绝且不扣材料、`smolder` 优先和 `IsHeld()` 都已改对。冰页 L54 还漏候选必须有 `fresh`／`stale`／`spoiled` 之一，应写「查鲜度标签，候选不读精确剩余百分比」。两页的搬重物排除都请写明属于骑乘条件的 `elseif` 分支。[修理候选](../../../raw/current/scripts/componentactions.lua#L1679-L1700) |
| R08 | 已改：`lightbulb` L92／L108（仅本 postinit 实例、空余 ≥180 与 `DoDelta` 夹取） | 通过。示例范围限定为该 postinit 构造的实例，干燥、无额外倍率和空余容量条件齐全。 |
| R09 | 已改：`pumpkin-lantern` L57／L59／L115（`not isday` 含黄昏、入昼 2+random 秒淡出、两条死亡动画）；索引 L18 | **部分修复。** 正文 L57–59 与索引的腐坏动画已改对。页面摘要 L6 仍写「只有落在地面且不是白天才亮」，快速判断 L22 的「先播 `rotten`」未说该路径跳过 `broken`。索引 L18 与陷阱 L115 建议用「非白天触发淡入，入昼延迟淡出」表述，避免同句绝对断言与延迟冲突。[两条死亡路径](../../../raw/current/scripts/prefabs/pumpkin_lantern.lua#L113-L130)、[入昼延迟](../../../raw/current/scripts/prefabs/pumpkin_lantern.lua#L132-L152) |
| R10 | 已改：`glommer-flower` L119／L155（停用后存档复述、6 天定义／地面约 4 天／到 0% 不自动删除） | 通过。已启动且存档的死花停用后恢复任务，未启用示例的旧档仍停表，时长条件和无替换物写清。 |
| R11 | 已改：`ancient-fruit` L49／L53／L55／L165（heatindex 提前 `break`、64 秒四项条件） | **部分修复。** 热源表和约 64 秒的示例前提已修。陷阱 L165 仍把冷火进入求和写成直接拉低果实温度，L55 的「靠火堆越远」容易读成随距离衰减。冷火只拉低当前 `heatindex`，累加仍达到 40 时这次依然升温，只有整个判定没达到 40 才按 `−dt` 降温。[加热与降温分支](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L43-L66) |
| R12 | 已改：`mushtree-spores` L55／L57／L84／L115／L120；索引 L20（生成门与拥挤门两套集合、无 10 秒重试、`SetPercent(0)` 到删除时序） | **部分修复。** 两套门、默认间隔和下一次更新才 `Perish` 已对上。陷阱 L134 仍把月亮孢子进入 `pop` 一律写成会删 `spore` tag。捕捉与 `depleted` 会删，邻近分支只发 `pop`／`preparedpop`，状态机在动画结束才移除实体，本身没有删 tag。请限定陷阱到有删 tag 的路径。[耗尽与捕捉](../../../raw/current/scripts/prefabs/mushtree_moonspore.lua#L14-L36)、[邻近分流](../../../raw/current/scripts/prefabs/mushtree_moonspore.lua#L61-L79)、[pop 状态](../../../raw/current/scripts/stategraphs/SGmoonspore.lua#L34-L56) |
| R13 | 已改：`fruit-fly` L6／L63／L65／L76／L157（`plantmeat` 照掉、`hascausedhavoc`＋无 leader、改绑旧果变干壳） | **部分修复。** 复绑与额外掉落的正文、索引已修。示例 L148 仍写「只掉种子」，须限定为额外掉落。L72 写「只要啃过一次作物」，但 `plant_attack_minion` 在第 63 帧调用 `PerformBufferedAction()` 后不检查返回值，直接置 `hascausedhavoc`。验收按「无 leader 且已置该标记」描述，动作失败也可能置位。[共享掉落](../../../raw/current/scripts/prefabs/fruitfly.lua#L51-L54)、[额外掉落](../../../raw/current/scripts/prefabs/fruitfly.lua#L110-L120)、[第 63 帧置位](../../../raw/current/scripts/stategraphs/SGfruitfly.lua#L179-L183)、[转敌对门禁](../../../raw/current/scripts/prefabs/fruitfly.lua#L274-L289) |
| R14 | 已改：`moonstorm-spark` L38／L98；索引 L23（3 为逐玩家阈值、计时口径） | 通过。360 秒定义与 240 秒地面估算已分开，正文交代重叠区域逐玩家重数与拾取前补检。 |
| R15 | 已改：`moonstorm-spark` L22／L57／L62／L70；`components/moonsparkchargeable` L40；索引 L23（10 次非 10 只、5 帧前摇） | **部分修复。** 次数、前摇和并行计时已写清。`moonstorm-spark` L62、L70 与索引 L23 仍把「从零到满需要 10 次」写成原版护目镜的实际流程。应写每次加 10 个百分点，10 次相当于一条满值的回充量，期间无消耗才可直接累计；原版护目镜耗尽会移除，不能等它到零再回充。[百分比夹取](../../../raw/current/scripts/components/moonsparkchargeable.lua#L18-L23)、[耗尽回调](../../../raw/current/scripts/prefabs/hats.lua#L2298-L2303)、[触发移除](../../../raw/current/scripts/components/fueled.lua#L305-L319) |
| R16 | 已改：`mosquitomusk` L6／L20／L58／L59／L94；`ranged-weapons` L94；索引 L24（收窄为常规重选＋炸弹初始选敌，补受击/AOE 例外） | **部分修复。** 摘要、初始选敌及独立伤害已分开。小痒痒 L58 的「被咬后」把受击主体写反，应是蚊子受到攻击。小痒痒 L94 与远程武器 L94 的「携带者也在内」请限定为符合原有范围伤害筛选的目标，例如投掷者为玩家时，其他玩家仍被排除。[蚊子受击](../../../raw/current/scripts/prefabs/mosquito.lua#L134-L141)、[范围门禁](../../../raw/current/scripts/prefabs/mosquitobomb.lua#L75-L88)、[玩家投掷参数](../../../raw/current/scripts/prefabs/mosquitobomb.lua#L154-L161) |
| R17 | 已改：`wortox_soulecho_buff_fx` L62／L76（普通 CASTSPELL 走 `crushitemcast`、9／18 帧、反向监听来源是 body）；`wortox-reviver` L80-L81／L93 已对齐 | 通过。两页帧位、移除事件来源与引擎父子回收的未核验边界一致。 |
| R18 | 已改：`ice` L22／L103／L119／L125；索引 L15（25/件为夹取前请求量、单格上限 100） | 通过。示例可用低湿耕地小叠或记录入参，正文与索引区分请求量和最终湿度。 |
| R19 | 已改：`glommer-flower` L7／L19（aliases 去掉 `glommerfuel`／`glommerwings`／`statueglommer` 精确别名，保留描述名；正文说明主注册与宿主/产物关系）；索引 L19；台账由主代理同步 | 通过。按同版运行时导出复跑，11 页命中 21 个主注册，各有唯一 Prefab 页；`wortox_reviver_body` 只命中表现族。台账和索引已区分雕像宿主与黏液、翅膀关联。 |

### 第二轮回填（2026-11-10，针对 10 项「部分修复」）

| 编号 | 修订方第二轮回执 |
|---|---|
| R03 | `mushtree-spores` L110 改为三条入口（月亮树、月亮蘑菇帽、蘑菇地精 `mushgnome`：构造即周期 2–6 秒、攻击 25–36 帧逐帧 12 枚、死亡表 3 必出＋2×50%，其 `onspawnfn` 不置 `_alwaysinstantpops`），并新增 `mushgnome` 导航行；`hats-environment` L127 改“三条入口之一”；索引 L20 同步；帽产跳过 `pre_pop` 仍单列 |
| R04 | `fruit-fly` L55 拆成三层：①共通门禁（`SPAWN_LORDFRUITFLY`／作物未睡／无活动计时器／场上无王）；②首轮追加（`check_others=true` 的 15 距离玩家）；③每个候选点各自的 4 距离 ≥15 个 `fruitflyspawner`；并写明首轮计数不足时按 4 距离内每个 spawner 为新候选点递归（`check_others=false`）；索引 L22 同步 |
| R05 | `wortox-reviver` L69／L118：`crushitemcast_fail` 重启但保留物品、不消耗；两条成功路径（selfuse、pst）才重启并 `OnConsume`；索引 L25 同步 |
| R07 | `ice` L54 补“候选必须带 `fresh`／`stale`／`spoiled` 之一、只查标签不读精确百分比”，并按源码顺序写骑乘 `IsGrandOwner` 与搬重物的 `elseif`；`mosquitomusk` L49 同步该 `elseif` 归属 |
| R09 | `pumpkin-lantern` L6 摘要改“非白天（黄昏与夜晚）触发淡入、入昼延迟约 2 秒淡出”；L22 补腐坏路径跳过 `broken`；L115 同口径；索引 L18 同步 |
| R11 | `ancient-fruit` L165 改“冷火只拉低当前 `heatindex`，本次累加仍到 40 则照样升温，整个判定未到 40 才按 −dt 降温”；L55 删“靠火堆越远”，改为离开 4 格有效热源才回落、不按距离衰减 |
| R12 | `mushtree-spores` L134 限定：只有捕捉（`onworked`）与 `depleted` 删 `spore` tag；邻近分支只发 `pop`／`preparedpop`，状态机动画结束才 `Remove`，本身不删 tag |
| R13 | `fruit-fly` L148 改“额外只掉种子（`plantmeat` 不受影响）”；L72 改“当前无 leader 且已置 `hascausedhavoc`”，并说明第 63 帧 `PerformBufferedAction()` 后不检查返回值即置位，动作失败也可能置上 |
| R15 | `moonstorm-spark` L62／L70／L108 改“每次 +10 个百分点，10 次≈一条满值回充量，仅期间无消耗才可累计；护目镜耗尽即移除，不能等归零再回充”；索引 L23 同步 |
| R16 | `mosquitomusk` L58 改“蚊子受到攻击后”；L94 与 `ranged-weapons` L94 把“携带者也在内”限定为符合原范围伤害筛选（投掷者为玩家时其他玩家仍被排除）；索引 L24 同步 |

以上 10 项待外部复验，其余 9 项维持通过。
修订方 2026-11-10 回填：R01–R19 全部按编号处理，无异议。机械校验（第二轮后）：11 页＋索引＋`hats-environment`／`ranged-weapons`／`wortox_soulecho_buff_fx`／`components/moonsparkchargeable` 共 16 个文件 `validate_page` 0 issue；`validate_source_links --strict` 1501 处引用、0 错 0 警（1 条既有 file-only、5 条 skipped unsupported）。未运行游戏。

第一轮外部定向复验于 2026-11-10 完成。Steam 与源码均为 `26474873`，按 R01–R19 回看当时的正文、摘要、索引和相关 Lua。月亮孢子扩查命令为 `rg -n -g '*.lua' -F 'spore_moon' raw/current/scripts`，补查证据并入 R03。

### 第二轮外部定向复验

2026-11-10，Steam 安装与 `raw/current/BUILD.txt` 仍为 `26474873`。本轮只核对第二轮回填的 10 项及其直接相关表述，9 项通过、1 项部分修复。上轮已通过的 9 项沿用结论。未重跑整批机械校验，未运行游戏。

| 编号 | 外部复验结果 |
|---|---|
| R03 | 通过。孢子页和索引均补蘑菇地精的周期、攻击、死亡三路；月亮蘑菇帽的 `pre_pop` 例外独立保留。 |
| R04 | 通过。果蝇王正文拆出共通门禁、首轮玩家门禁和每个候选点 4 距离内的计数，邻株递归与索引口径一致。 |
| R05 | 通过。双尾心正文、验证记录和索引均区分提前中断停表、动作失败复表并保留、成功复表并消耗。 |
| R07 | 通过。冰和小痒痒均写明鲜度标签与精确百分比的分工，搬重物判断限定在骑乘条件的 `elseif`。 |
| R09 | 通过。南瓜灯摘要、正文、陷阱与索引区分黄昏淡入、入昼延迟淡出和腐坏跳过 `broken`。 |
| R11 | **部分修复。** 热源表与陷阱已区分累计热量和果实温度。仍需改 [晶洞果推导段 L55](../../../wiki/reference/prefabs/ancient-fruit.md#晶洞果加热判定与三段阈值定义处--结算处) 的「被 `break` 之外的负值抵消到不足 40」。累加一旦达到 40 就 `break` 并升温，排在此后的负值不会读取。建议直接写「一次判定遍历完仍未达到 40 时，果实按 `−dt` 降温，最低为 0」，与紧邻的热源表和陷阱一致。[求和、提前退出与降温](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L43-L66) |
| R12 | 通过。孢子陷阱区分了捕捉／耗尽删 tag 与邻近爆炸到动画结束才移除。 |
| R13 | 通过。果蝇示例保留共享 `plantmeat`，小果蝇转敌对按无 leader 和 `hascausedhavoc` 判断，并说明标记不取决于动作成功。 |
| R15 | 通过。月熠正文和索引把 10 次写作回充量换算，正文同时保留期间消耗及护目镜耗尽移除的限制。 |
| R16 | 通过。小痒痒与远程武器页区分蚊子受到攻击、炸弹初始选敌和独立伤害筛选，索引未扩大标签作用。 |

下一轮只需回填并核对 R11 的推导段。

### 第三轮回填（2026-11-10，仅 R11）

| 编号 | 修订方回执 |
|---|---|
| R11 | `ancient-fruit` L55 改为“一次判定遍历完仍未达到 40 时，果实按 −dt 降温，最低为 0（走出 4 格半径或热源熄灭都属这种情形），代码不按距离做衰减”，删去“被 `break` 之外的负值抵消到不足 40”；验证记录 L180 补第三轮说明。

机械校验（第三轮后）：16 个文件 `validate_page` 0 issue；`validate_source_links --strict` 1501 处引用、0 错 0 警（1 条既有 file-only、5 条 skipped unsupported）。未运行游戏。

待外部复验 R11 一处。

### 第三轮外部定向复验

2026-11-10，Steam 安装与源码快照均为 `26474873`。**R11 通过。** 晶洞果推导段已写明，仅当本轮遍历结束仍未达 40 时，果实温度才按 `−dt` 降低并以 0 为下限；累加先到 40 则升温并提前结束遍历，后续热源不参与。推导段与热源表、陷阱和验证记录一致。沿用其余 18 项的复验结果，本轮只更新报告，未重跑整批校验或运行游戏。[推导段](../../../wiki/reference/prefabs/ancient-fruit.md#晶洞果加热判定与三段阈值定义处--结算处)、[Lua 判定](../../../raw/current/scripts/prefabs/ancienttree_fruits.lua#L43-L66)
