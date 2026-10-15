# B03 种子、野采果实与海带独立检视

2026-11-08，源码快照 Steam buildid 26335394。逐页核对 B03 五页的注册、官方简中名、机制与示例，并回查暂行索引及相关交付记录。覆盖为 29 个可入栏注册和 1 个放置预览 `seeds_placer`。五页均在暂行索引各登记一次，返回链一致。物品主名称、作物种子识别前后的两组显示名、原版食物基础数值、保质档及堆叠上限均核对通过。

结论为需修复。P1 共 2 项，P2 共 11 项，P3 共 2 项。原版执行分支与接入示例是本轮的主要修复面，分类与页面归属可以沿用。

## P1

| 编号 | 页面位置 | 源码复核与建议 |
|---|---|---|
| R01 | [专用种子 · 获得与掉种](../../../wiki/reference/prefabs/farm-crop-seeds.md#获得与掉种) L91 | 果蝇王的掉落条件写反。场上**没有** `friendlyfruitfly` 时给 1 个友好果蝇果，**已有**时才加入 4 件必掉作物种子及 4 次额外种子机会。按两分支重写表格。[果蝇王战利品](../../../raw/current/scripts/prefabs/fruitfly.lua#L110-L120) |
| R02 | [专用种子 · 典型接入](../../../wiki/reference/prefabs/farm-crop-seeds.md#统一-14-件种子的堆叠与保质) L167 | 把 `farmplantable.plant` 改成函数只适合走 `PLANTSOIL` 的路径。沃姆伍德直接部署时，专用种子的 `OnDeploy` 把该字段直接传给 `SpawnPrefab`，其字符串处理会报错。建议删去这一旁支，或注明必须同步适配部署回调。现有堆叠／保质示例本身无需重写。[Plant](../../../raw/current/scripts/components/farmplantable.lua#L7-L25)、[直接部署](../../../raw/current/scripts/prefabs/veggies.lua#L187-L193)、[SpawnPrefab](../../../raw/current/scripts/mainfunctions.lua#L429-L436) |

## P2

| 编号 | 页面位置 | 源码复核与建议 |
|---|---|---|
| R03 | [专用种子](../../../wiki/reference/prefabs/farm-crop-seeds.md#获得与掉种) L90、L175；[通用种子](../../../wiki/reference/prefabs/generic-seeds.md#获得路径) L81、L86 | 鸟笼换种与掷鸟粪须限定普通鸟。带 `bird_mutant` 的鸟对所收非肉食物掉 `spoiled_food`，带 `bird_mutant_rift` 的鸟对这些种子／作物走月亮碎片专用门禁。普通鸟的 0.33 是幸运修正前基值。「笼子只拒收」应写成「额外禁喂名单」，仍有食性筛选。种子袋和鸟笼都用无结尾锚定的 `string.match(prefab, "_seeds")`，插件兼容说明宜用「名字含 `_seeds`」。种子袋也属于收纳用途，移出获得路径。[幸运公式](../../../raw/current/scripts/prefabs/birdcage.lua#L100-L104)、[消化](../../../raw/current/scripts/prefabs/birdcage.lua#L110-L147)、[接收](../../../raw/current/scripts/prefabs/birdcage.lua#L155-L181)、[种子袋](../../../raw/current/scripts/containers.lua#L1731-L1735) |
| R04 | [通用种子](../../../wiki/reference/prefabs/generic-seeds.md#随机种植与随机株) L20、L22、L70–72、L109–130；[暂行索引](../../../wiki/reference/prefabs/index.md#基础材料与常用物品暂行) L53 | 「固定通用种子」示例只改变 `PLANTSOIL`，沃姆伍德直接部署硬编码生成随机株，旧农田另读 `plantable.product`。摘要和示例标题补上适用路径。随机株秋／春发芽 180–240 秒，冬／夏才是 360–480 秒。抽到杂草时只执行生成、定位与替换，复制组件、禁巨型和推进生长由 `plant.plant_def` 门禁限定。耕地替换限定 `PLANTSOIL`；只有 `Plant` 正常返回失败时动作才回交原种，异常路径不保证回交。[直接部署与旧字段](../../../raw/current/scripts/prefabs/seeds.lua#L131-L173)、[耕地移除](../../../raw/current/scripts/components/farmplantable.lua#L7-L30)、[失败回交](../../../raw/current/scripts/actions.lua#L4727-L4745)、[适种季](../../../raw/current/scripts/prefabs/farm_plant_defs.lua#L82-L87)、[发芽](../../../raw/current/scripts/prefabs/farm_plants.lua#L364-L368)、[替换](../../../raw/current/scripts/prefabs/farm_plants.lua#L151-L167) |
| R05 | [通用种子](../../../wiki/reference/prefabs/generic-seeds.md#三件注册) L6、L32、L100、L136 | 摘要与陷阱把烤种子写成只可吃，和同页 `snowmandecor` 相冲突。生熟均有独立雪人装饰映射。改为「熟件可食、可装饰雪人，不可播种或作海钓拟饵」，可给装饰表直接证据。[挂载](../../../raw/current/scripts/prefabs/seeds.lua#L115-L126)、[装饰映射](../../../raw/current/scripts/components/snowmandecoratable.lua#L39-L41) |
| R06 | [浆果 · 典型接入](../../../wiki/reference/prefabs/wild-berries.md#抬高多汁浆果堆叠并延长烤制态的保质期) L98、L122；[海带叶 · 典型接入](../../../wiki/reference/prefabs/kelp.md#三态调参去掉生熟负收益并延长干叶保质期) L114、L143 | 两个保质示例都应区分新生成与读档。读档先构造并设新上限，再恢复旧剩余秒数，显示比例按旧剩余秒数除以新上限、最多 1。浆果页的「按比例覆盖」应明确为绝对秒数覆盖；海带叶的「40 天生效」限定新生成满鲜实例，旧干叶不会因读档重获 40 天。顺手保留浆果堆叠 60，它确为原版可同步的 `STACK_SIZE_TINYITEM` 档位。[保质换算](../../../raw/current/scripts/components/perishable.lua#L204-L221)、[读档](../../../raw/current/scripts/components/perishable.lua#L337-L349)、[堆叠档](../../../raw/current/scripts/tuning.lua#L81-L85)、[堆叠同步](../../../raw/current/scripts/components/stackable_replica.lua#L98-L100) |
| R07 | [浆果 · 获得与入栏](../../../wiki/reference/prefabs/wild-berries.md#获得与入栏) L53、L57、L59–61 | 挖 `berrybush2` 给 `dug_berrybush2`，源码拼的是 `dug_..inst.prefab`。普通浆果仅在采摘者有库存组件时尝试交付，其他分支落地；再生公式算出后还应用春季 ×0.75。火鸡概率 10%／30% 是幸运修正前的基值，另有 `_noperd` 门禁；挖起直接掉果也应与触发 `onpickedfn` 的路径分开。多汁丛的补摘可明确为未挖完的工作回调（源码注释以海狸啃挖为例）。[挖起与补摘](../../../raw/current/scripts/prefabs/berrybush.lua#L158-L198)、[交付与再生](../../../raw/current/scripts/components/pickable.lua#L493-L513)、[再生计时](../../../raw/current/scripts/components/pickable.lua#L551-L568)、[春季倍率](../../../raw/current/scripts/simutil.lua#L563-L565)、[火鸡](../../../raw/current/scripts/prefabs/berrybush.lua#L103-L107)、[幸运公式](../../../raw/current/scripts/componentutil.lua#L2190-L2199) |
| R08 | [野采果实 · 获得路径](../../../wiki/reference/prefabs/tropical-gathered-fruits.md#获得路径) L57–63、L130 | 香蕉丛的 5～7 是移植丛的采摘额度，野生丛不扣轮次；移植时先变贫瘠，施肥可经 `OnRegen` 直接设置第 4 阶段。正常从空丛到第 4 阶段经过三个计时段，春季时长还乘 0.75。猴岛布局回调保证把香蕉丛加入候选表；只引用该函数时宜写候选保证，避免推成每次实际放置保证。海树柱补藤有撞击后的调度条件，新藤会先设为空株。[阶段](../../../raw/current/scripts/prefabs/bananabush.lua#L51-L76)、[移植与再生](../../../raw/current/scripts/prefabs/bananabush.lua#L115-L140)、[构造](../../../raw/current/scripts/prefabs/bananabush.lua#L177-L200)、[扣轮次](../../../raw/current/scripts/components/pickable.lua#L535-L539)、[计时](../../../raw/current/scripts/components/growable.lua#L57-L78)、[布局](../../../raw/current/scripts/map/layouts.lua#L72-L93)、[补藤](../../../raw/current/scripts/prefabs/oceantree_pillar.lua#L234-L262)、[撞击后调度](../../../raw/current/scripts/prefabs/oceantree_pillar.lua#L286-L301) |
| R09 | [野采果实](../../../wiki/reference/prefabs/tropical-gathered-fruits.md#仙人掌肉的生熟正负) L22、L59、L64、L85、L109、L123、L128 | 「采摘必被刺／扣 6」应改成满足门禁时提交基础伤害 6，采摘者要有 combat，且排除刺抗装备和 `shadowminion`。夏季开花的唤醒分支在 L64–69，应补到证据。调参去掉 `badfood` 只移除这一道非 PvP 喂玩家限制，`spoiled`、`unsafefood` 与食性等门禁仍适用；`FEED` 与 `FEEDPLAYER` 的候选判断分开写。[采摘](../../../raw/current/scripts/prefabs/cactus.lua#L18-L43)、[唤醒](../../../raw/current/scripts/prefabs/cactus.lua#L64-L69)、[动作候选](../../../raw/current/scripts/componentactions.lua#L1184-L1197)、[喂玩家](../../../raw/current/scripts/actions.lua#L2397-L2421) |
| R10 | [海带叶 · 获得与入栏](../../../wiki/reference/prefabs/kelp.md#获得与入栏) L61–68、L101；[海带根](../../../wiki/reference/prefabs/bullkelp_root.md#获取途径) L66–67 | 冲岸时无条件生成根，只有植株还能通过 `Pick` 门禁才掉叶。三叉戟、温妮投石机等也有采摘条件，不能写成一律两件齐掉。常规再生写成基础 3 天，实际还有春季倍率。相关根页用同一口径。[冲岸](../../../raw/current/scripts/prefabs/bullkelp_plant.lua#L42-L54)、[采摘门禁](../../../raw/current/scripts/components/pickable.lua#L535-L568)、[三叉戟](../../../raw/current/scripts/prefabs/trident.lua#L98-L110)、[投石机](../../../raw/current/scripts/prefabs/winona_catapult_projectile.lua#L138-L150) |
| R11 | [海带叶 · 干制链](../../../wiki/reference/prefabs/kelp.md#干制链) L78、L82–85、L148、L151 | 120 秒是正常计时档。架在暂停时可能只存秒数；普通雨暂停时不再给零腐坏倍率，酸雨还可额外加速腐坏。中途取出后只有仍保留 `dryingrack_drytime` 才能续计，落地或堆叠变动会清掉。烧毁有即时干制分支，`LongUpdate` 也可推进暂存秒数并把超出部分施加给成品；「干完满鲜」限定正常即时完成。可在现有段落收紧，保留详细宿主逻辑给组件页。[倍率](../../../raw/current/scripts/components/dryingrack.lua#L26-L38)、[入架及取出](../../../raw/current/scripts/components/dryingrack.lua#L226-L289)、[烧毁及 LongUpdate](../../../raw/current/scripts/components/dryingrack.lua#L377-L419) |
| R12 | [海带叶](../../../wiki/reference/prefabs/kelp.md#关联机制导航) L6、L95、L103；[暂行索引](../../../wiki/reference/prefabs/index.md#基础材料与常用物品暂行) L56 | 「给船与防撞条补 10 点」易让人误以为普通木船也能用。应限定 KELP 材质且可受理的水獭掠夺者草筏与海带保险杠，10 是每份的恢复量参数。晾架可用官方名「晾肉架／晾肉巨架」，「晾晒架」可保留作描述。[普通船](../../../raw/current/scripts/prefabs/boat.lua#L570-L573)、[草筏](../../../raw/current/scripts/prefabs/boat.lua#L1210-L1218)、[保险杠](../../../raw/current/scripts/prefabs/boat_bumpers.lua#L342-L352)、[恢复量](../../../raw/current/scripts/tuning.lua#L2119-L2123)、[简中名称](../../../raw/current/scripts/languages/chinese_s.po#L345260-L345277) |
| R13 | [台账 B03](prefab-inventory-ledger-20261108.md) L152；[维护日志](../../../wiki/log.md) L10 | 「5 页 29 注册」与清单口径不一致。实际为 14＋3＋4＋6＋3＝30 个注册，其中 `seeds_placer` 是附带预览，因此可入栏物品 29 个。修订记录与后续汇总用同一口径。[三件导出](../../../raw/current/scripts/prefabs/seeds.lua#L216-L218) |

## P3

| 编号 | 页面位置 | 源码复核与建议 |
|---|---|---|
| R14 | [专用种子](../../../wiki/reference/prefabs/farm-crop-seeds.md#常见陷阱与限制) L112、L176；[浆果](../../../wiki/reference/prefabs/wild-berries.md#注册与构造) L20、L33 | 陷阱「鱼饵」可改「诱饵」，与海钓拟饵分开。Gorge 钩子的锚点在生食物工厂 `fn`，`fn_seeds` 无这一调用，待核验项改成只说事件数据缺口或删去。浆果是两个 `VEGGIES` 键在 `SEEDLESS` 表中，导出四个注册；两种生浆果浮水参数相同，浮水差异在熟件。[种子工厂](../../../raw/current/scripts/prefabs/veggies.lua#L453-L537)、[Gorge 钩子](../../../raw/current/scripts/prefabs/veggies.lua#L683-L685)、[浆果表](../../../raw/current/scripts/prefabs/veggies.lua#L85-L101)、[SEEDLESS](../../../raw/current/scripts/prefabs/veggies.lua#L160-L168) |
| R15 | [专用种子](../../../wiki/reference/prefabs/farm-crop-seeds.md#获得与掉种) L92；[海带叶](../../../wiki/reference/prefabs/kelp.md#获得与入栏) L61、L64 | 本批物品主名正确。关联实体可补简中实显供检索，`catcoon` 为「浣猫」、`bullkelp_plant` 为「公牛海带」、`otterden` 为「水獭掠夺者窝点」。现有泛称适合放在括号或说明里。[浣猫](../../../raw/current/scripts/languages/chinese_s.po#L340690-L340692)、[公牛海带](../../../raw/current/scripts/languages/chinese_s.po#L339975-L339977)、[窝点](../../../raw/current/scripts/languages/chinese_s.po#L346700-L346702) |

## 核验边界

本轮仅做独立源码与文档检视。未重跑交付方的结构、源码链接、哈希与 Lua 示例校验，未运行游戏。节日数据包、图标显示与非自然传送门的实际生成条件仍按原交付记录保留缺口。

## 最终复检与收口

2026-11-08，Steam buildid 26335394。R01、R02 已修复。复检保留 5 页的归属和暂行登记，确认 30 个注册中 29 个可入栏，另 1 个是 `seeds_placer` 放置预览。

- 种子页对齐 `PLANTSOIL` 先移除目标土堆再生成植株、正常返回失败时尝试交回种子。沃姆伍德直接部署无需预先锄地，仍会处理附近土堆。通用随机株可提前识别定种，出苗时使用已定结果或再抽取。
- 两种生浆果浮水参数一致，多汁浆果丛的补摘发生在挖掘尚未完成时。仙人掌开花段保留刺伤门禁，喂玩家的 `badfood` 标签与牛按饥饿、生命数值判定的状态分开。
- 海带叶与海带茎两页统一为冲岸或击毁生成茎、满足各路径采摘条件才额外掉叶。晾晒正常入架起计时，暂停时只存秒数，原料仍有 `dryingrack_drytime` 才能续计。补齐烧毁和长时间推进的源码锚点。

定向验收覆盖 B03 五页、海带茎关联页及 Prefab 索引。页面与导航通过，严格源码链接检查 628 条引用、0 错 0 警，六页的 185 项源码哈希比对均一致。Lua 示例沿用修订方的 5 段解析结果，本次未改代码块。未运行游戏，原有节日数据与运行时缺口继续保留。

B03 已收口。
