---
title: "通用材料与加工中间件（暂行索引）"
created: "2026-11-10"
updated: "2026-11-10"
type: index
tags: [overview]
---

# 通用材料与加工中间件（暂行索引）

非装备物品栏 Prefab 扩库 B10 登记 8 页：常青树族、灰烬与木炭、浮木桩与活木头、齿轮与电子元件、骨头碎片、月岩、盐晶、蜘蛛丝。B13 追加 11 页远古、裂隙与机械材料：噩梦燃料与纯粹恐惧、绝望石、暗影碎布、亮茄外壳、纯粹辉煌、铥矿与铥矿碎片、传送宝珠碎片、尘土块、生物数据、陀螺传导核心、废料。正式的叶级细分留到该批全部轮次结束后统一整理。

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/evergreen-trees|常青树族（evergreen 15 注册）]] | 15 个注册（普通 4＋臃肿 4＋多枝树 5＋预置烧毁与树桩）；四阶段循环生长与春季 ×0.75、CHOP 5/10/15（老树 1）、砍倒换桩并 DIG 返 log、烧毁后 10 秒自然掉落与先砍两条路径各自重掷 40% 木炭、树精抽签、森林石化换石块、再生 5／8 天与蜡封雪覆盖 | evergreen、常青树、臃肿常青树、多枝树、烧毁的常青树、常青树桩 / active |
| [[reference/prefabs/ashes-and-charcoal|灰烬与木炭（ash、charcoal）]] | 灰烬落地 25–35 秒自毁、拾取停表，作祟与到期同一消失序列（醒着先禁点选／拾取，休眠当场删除）；木炭 45 秒燃料＋15 秒可燃；两者 BURNT 20/20、molebait、雪人装饰、猪王不收；烧毁默认列表 40% 木炭、显式列表只走熟品／灰烬改写，焦树／篝火／火坑／克劳斯等来源 | 灰烬、木炭、燃烧残留、ashes、charcoal / active |
| [[reference/prefabs/driftwood-livinglog|浮木桩与活木头（driftwood_log、livinglog）]] | 两件木质堆叠材料：WOOD 食性 0/0、燃料 45、小号可燃 15 秒、只修 health 的 WOOD 修补件（12.5／37.5）；浮木桩防潮（effectiveness 0）、活木头进食／燃烧／焚烧惨叫；来源与烧毁分支 | 浮木桩、活木头、浮木、活木、driftwood_log、livinglog / active |
| [[reference/prefabs/clockwork-parts|齿轮与电子元件（gears、transistor）]] | 发条生物与垃圾堆掉落；齿轮 WX-78 专用可食 60/75/50、可修 GEARS 目标、对发条友好化；电子元件 10 一叠、无食性，研究站与薇诺娜设备的配方中间件，双恐怖之眼每只保底 2 个；骑士神龛收下时供品保留、换供品退回，点燃／烧毁可能丢失 | 齿轮、电子元件、gears、transistor、发条零件 / active |
| [[reference/prefabs/boneshard|骨头碎片（boneshard）]] | 骷髅／骨堆／浅坟／风滚草等来源；骨制配方（弹弓骨框、怀表复活、雨具、鞍角等）；可进锅，蒸树枝（4 骨片）与花式回旋块茎（2 土豆＋1 树枝＋1 骨片）等容忍 inedible 的配方可成立，炸鱼排会超限 | 骨头碎片、骨片、boneshard / active |
| [[reference/prefabs/moonrocknugget|月岩（moonrocknugget）]] | 月岛矿物：修月岩墙 +40 血；月亮石 workable 无采掘动作，零工作量条件掉落 1–2 枚（修回正值可再触发），初始 4/6 一件补满、0 起算 3 件；弹弓弹药配方原料；2 大理石→1 月岩（威尔逊转化，来源）；入水沉没 | 月岩、moonrocknugget、月岩墙修复、月亮石修复 / active |
| [[reference/prefabs/saltrock|盐晶（saltrock）]] | 撒盐＝夹取前 +50 个百分点、按目标叠数除、夹到 100%，满鲜目标仍扣件；修美味舔盐块 80 次、鼹鼠诱饵；鸟陷阱诱饵无效；盐罐保鲜走 preserver | 盐晶、saltrock、海盐、盐舔块修复 / active |
| [[reference/prefabs/silk|蜘蛛丝（silk）]] | 韦伯花 5 根催升蜘蛛巢一阶（封顶 3 阶）、捕虫网／缝纫包等通用材料、可喂浣猫；各种蜘蛛与巢掉落 | 蜘蛛丝、silk、升级蜘蛛巢 / active |

| [[reference/prefabs/nightmare-fuel|噩梦燃料与纯粹恐惧（nightmarefuel、horrorfuel）]] | 两个暗影燃料注册：燃料 180／360 秒（湿 0.75 参与乘算）、修懒人护符 50／100 次；纯粹恐惧接沃特暗影施法与 WX-78 满充能 20／60 秒；掉落含裂影之手与蜈蚣头；配方消耗 49（42 制作＋7 拆解）／21（20＋1） | 噩梦燃料、纯粹恐惧、Nightmare Fuel、Pure Horror、暗影燃料 / active |
| [[reference/prefabs/dreadstone|绝望石（dreadstone）]] | 裂隙矿苗是裂隙的显示态而非独立 prefab（6 点 MINE、挖完固定 3 块；480 秒是管理器冷却，回填取当时仍受控的裂隙）；修理请求 80／块按上限截断、墙物品 320；墙锤拆 max(1, floor(2×当前生命比例))；建造计划脚手架 40 块与 `charlie_hand` 5 块、配方拆解回收 2／5；非燃料 | 绝望石、恐惧石（旧译）、dreadstone、绝望石墙 / active |
| [[reference/prefabs/voidcloth|暗影碎布（voidcloth）]] | 裂影仆从手／角／口（3×1.00＋0.33）与翅膀（2×1.00＋0.67）、暗影憎恶（4×1.00＋0.75＋0.50）掉落（均为无幸运基础概率）；10 条消耗侧配方里九条 `nounlock` 跳过 `UnlockRecipe`、靠当次站点科技制作，弹弓握把只有 `force_hint`、可正常学会；装备修理走 `FORGEMATERIALS.VOIDCLOTH` 的虚空修补套件通道 | 暗影碎布、虚布（简称）、Dark Tatters、voidcloth / active |
| [[reference/prefabs/lunarplant-husk|亮茄外壳（lunarplant_husk）]] | 致命亮茄死亡掉落 2 外壳＋2 植物肉（基础名单；普通燃烧改写时外壳转灰、植物肉换熟制，受控火保留外壳原名且不闷烧）；9 条月之锻造台配方＋弹药 1→20（沃尔特技能 ×1.5＝30）；堆叠 20 显式赋值；无 AddTag | 亮茄外壳、亮茄材料、lunarplant_husk、月之锻造 / active |
| [[reference/prefabs/purebrilliance|纯粹辉煌（purebrilliance）]] | 沃特月之施法（`wurt_lunar_spelluser`，消耗 1）＋弹药 1→20＋薇诺娜宝石发电机（需 `winona_lunar_2`）；3 注能碎片→1 纯粹辉煌、反向 1→2（往返净耗 1）；本页弹药 1 条可学会，锻造配方按 `nounlock` 靠当次站点科技制作（另一条弹药在外壳页）；获取含裂隙水晶、变异三巨头、天体仇灵／后裔 | 纯粹辉煌、purebrilliance、纯粹辉煌弹药、宝石发电机 / active |
| [[reference/prefabs/thulecite|铥矿与铥矿碎片（thulecite、thulecite_pieces）]] | 两注册共 35 条配方（23 整块／12 碎片；弹药与弹弓框可正常学会，`nounlock` 的远古配方跳过 `UnlockRecipe`、靠当次站点科技制作）；修复值 1.5/100 与 1.5÷6；6 碎片合 1 铥矿；破损祭坛 6 块、敲碎后 1 块；两者同为 ELEMENTAL 可食物；来源含眼睛炮塔、尘蛾巢、梦魇发条、月石雕像、废墟雕像 | 铥矿、铥矿碎片、图勒石（俗称）、thulecite、thulecite_pieces / active |
| [[reference/prefabs/vault-orb|传送宝珠碎片（vault_orb_refined）]] | 圣殿路线消耗品：未精炼宝珠精炼 1→3；传送焦点三座宝石座装同种宝石才 VALID；只有三座皆为宝珠碎片（`requiredgem`）时才挂地图目的地，三颗紫宝石只给法杖用；空座时焦点状态 `GEMS`，异种插入返回 `WRONGGEM`（被拒后状态按当前已有座子计算）；消耗 1 个、落点生成传送门 FX | 传送宝珠碎片、宝库宝珠（旧称）、vault_orb_refined、圣殿传送 / active |
| [[reference/prefabs/refined-dust|尘土块（refined_dust）]] | 档案馆限量材料：`TECH.LOST` 配方须先拿「蒸馏的知识」蓝图；成品用于铥矿捕虫网（蓝图来自尘蛾）、琥珀美食锅与蚁狮进贡（`rocktribute` 1 计增量，需当前可收贡且受计时器上限约束），可作陷阱饵；40 叠 | 尘土块、精炼粉尘（旧称）、refined_dust、铥矿捕虫网材料 / active |
| [[reference/prefabs/scandata|生物数据（scandata）]] | WX-78 扫描产物：首扫按物种上限整叠给、之后按余额回复（满槽约 8 分钟＝16×30 秒）；零余额但有未学配方仍可扫描，只教配方不出数据；付 23 个电路模块与两台设备配方；可燃掉灰、作祟不点燃 | 生物数据、扫描数据（旧称）、scandata、Bio Data、WX-78 模块材料 / active |
| [[reference/prefabs/wagdrone-parts|陀螺传导核心（wagdrone_parts）]] | 无人机残骸工具：一次耗 1 个把残骸收编为可部署友方（不设 leader）或修满磨损件，只认 `wagdrone_rolling`；收编后 2000 次使用、受伤按 `math.floor(0.5−amount/maxhealth×2000)` 至少扣 1、耐久归零只清部署点不变回残骸；无任何配方产消 | 陀螺传导核心、无人机零件（旧称）、wagdrone_parts、收编无人机 / active |
| [[reference/prefabs/wagpunk-bits|废料（wagpunk_bits）]] | Wagstaff 系通用零件：拾荒疯猪、垃圾堆、残骸、检查眼镜箱等掉落；18 条配方＋1 建造计划＋3 拆解（灵体传输模块 12 生物数据＋2 废料、电刑机控制器 10 生物数据＋8 废料＋1 晶体管）；只修 `MATERIALS.GEARS` 目标（废料墙、废料棋子）；40 叠、molebait | 废料、Scrap、wagpunk_bits、W.A.R.B.I.S. 材料 / active |

返回 [[reference/prefabs/index|原版 Prefab 索引]]。
