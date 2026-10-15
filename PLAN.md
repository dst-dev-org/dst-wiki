# DST Wiki 建设计划

最后更新：2026-10-28。已交付至第 110 批及通用 C76，并完成检索修整 Q01／Q02，固定98项清单全部覆盖，首轮索引重整已完成，后续按任务恢复扩库。26148036 新增 14 组件已建齐，机制页增至 3 篇，待审线索余 12 条。本检查点已登记 396 旧版＋14 新增组件、421 篇知识页／24 篇导航。本对话负责通用机制，其他 AI 的交付与认领保留；执行规模按当轮用户指令确定，定向复核和统一登记，减少重复读取与工程报告。

查询／知识增修不读本文件。跨仓职责见 [总体设计](docs/design.md)，知识仓工程规则见 [维护入口](references/project-maintenance.md)。分仓前混合计划已 [原字节归档](docs/history/plan-before-doc-split-20261013.md)，历史口径见 [档案说明](docs/history/README.md)；旧任务没有因拆文档而取消或被标记完成。

## 当前资产与推进顺序

- [x] 当前资产（知识交付至通用 C76及检索修整Q01／Q02，索引已细化）：396 篇旧版已登记组件、14 篇 26148036 新增组件、3 篇 modinfo、5 篇 cookbook、3 篇 mechanism，共 421 篇已登记知识页与 24 篇导航；第 92 组 healthbar／healthsyncer、93～97 组 10 页、98～101 组 12 页、102～106 组 13 页与 107～110 组 8 页由另一 AI 复核并登记，游戏运行未验证。前一百一十批及通用 C01～C76 已登记，26148036 新增组件两批 14 页已全部登记。C74～C76 已有验收保留；Q01 新增 oldager 并修补五页与四张索引，Q02 补齐 updatelooper 并登记更新调度主题，其他 AI 既有交付和维护记录保留。
- [x] 覆盖 CSV 的 798 个旧版候选中 396 已建、402 未建，另 19 replica／3 helper／1 empty 单列；26148036 新增 14 组件单列且已全部建页（第一批 8＋第二批 6），不重复计入旧版候选完成数。当前状态以 [覆盖清单](docs/reports/w05/component-coverage.csv) 与新增组件的 W06 记录为准，剩余候选路线见 [目录盘点](docs/reports/w05/component-roadmap.md)；名称归组不表示职责已核验。
- [x] 固定组件阶段完成：[98 项清单](docs/reports/w05/component-priority-checkpoint.md)已交叉核对为98／98、剩余0，每项均有实际存在的独立页面及覆盖登记；按职责采用详细页或简要边界说明。26148036 新增14组件不在该冻结清单内，不抵扣固定范围。
- [x] 98项检查点后的首轮索引重整已完成：根导航直达组件榜单，组件总索引按四组路由16个主题；保留正文地址与旧锚点，唯一登记、大小／链接及11次代表章节读取通过。后续按任务从剩余候选恢复建设，分类随真实检索反馈增修，不预建空层级。具体范围见 [主题细化](docs/reports/w05/component-topic-refinement.md)。
- [x] 2026-10-27 索引首版试用：根导航直达组件总索引的原版静态直接接入量 Top 30；按不同调用文件去重，保留真实缺页 updatelooper 的源码入口。首版将组件总索引的九路主题缩为需求摘要，唯一叶级登记及正文地址不变。统计口径、只读复现与验收见 [榜单试用](docs/reports/w05/component-top30-26148036.md)；首版记录保留，后续主题细化见下条。
- [x] 2026-10-27 主题细化完成：九主题细分为16主题，新增环境、维护、制作、农作、协作、移动与世界管理；164条迁移登记、408条原摘要／别名／状态逐字保留，85个旧标题锚点仍可用。15张改动索引与全库导航通过，items降至114行、world-creatures降至77行；统计、边界及11次章节读取验收见 [主题细化](docs/reports/w05/component-topic-refinement.md)。
- [x] 2026-10-27 检索修整Q01：修补 health 人物65535同步边界、childspawner 蝙蝠昼夜宿主分流、preserver 普通背包禁入与模组嵌套条件；batspawner／hideout 补定向入口，新建 oldager 覆盖旺达自然老化、伤害、回春与表盘预测边界。装备战斗索引增加耐久选型链接。六页及四张索引通过，222处源码引用严格零错警、113处数字标签匹配，六次代表章节读取完整。执行中其他流程将 current 切换26230496，本批证据固定至归档26148036；仅 oldager 新录基线、health／childspawner 刷新来源清单，不将新版本差异计作已核验。覆盖清单与索引已同步，未新增Lua示例或运行游戏。
- [x] 2026-10-28 高价值缺页Q02：新增 updatelooper（steam-buildid:26230496），覆盖普通／长／墙钟／后置回调、首次调度、延迟删除与组件移除、裂缝淡入和南瓜填充初始化接线；登记 abilities 的“更新调度”，Top30第15项替换为Wiki链接，保留原26148036统计排名。页面及两张索引通过，47处源码引用strict零错警，五次章节读取完整；新增单页基线，未运行游戏、无新增Lua示例或临时文件。原始盲测3题仅Wiki／1题缺页回退，不据此宣称通用token收益。
- [ ] 按实际非组件任务继续 dst-wiki-skill 的 S03-A 代表检视，不作为当前组件扩库前置；模板／类型指引及六份隔离草稿已交付，不据此认定全部类型可靠覆盖。源码文件不机械对应知识页。
- [x] 通过当前 Windows Codex CLI 的安装／定位、已覆盖查询、缺页回退与隔离维护样本后，用独立安装 Skill 试建 moisture 并审阅发布；未将缺页回退成本或其他 harness 全部验收设为扩库前置。
- [x] 第六批 moistureimmunity／freezable 经审阅发布，少量需求检视能定位并完整读取所需章节；结果见 [潮湿与冰冻检视](docs/reports/w05/component-moisture-freezing.md)。成本与其他类型检视随内容推进。
- [x] 第七批三页及 tool／workable／finiteuses 衔接完成，检视见 [工作量与使用消耗](docs/reports/w05/component-work-efficiency.md)；未新增独立模型评测。
- [ ] 后续 cookbook 先配方、再建筑；装备背包／服装／护符及角色等按实际组装缺口推进，不等待全部组件写完。

### 并行协作分工（2026-10-24）

按用户分工，本对话负责通用、高复用组件；其他 AI 负责其余专项或低优先候选。“常用”是开发需求优先级，不是已测使用频率，职责仍须核验源码。

- **2026-10-27 通用 C74～C76 已交付（Codex）**：三个子代理完成 `giftreceiver`／`shop`／`researchpointconverter`、`pointofinterest`／`scrapbookable`、`recipestockpile`／`treasuremarked` 七页；主代理复核实际消费者、旧接口边界和清理缺口，统一验收与共享登记。固定98项全部覆盖，暂停继续组件扩库，下一任务为索引重整。

- **2026-10-27 通用 C71～C73 已交付（Codex，页面于10月26日核验）**：三个子代理分别完成 `skinner`／`wardrobe`、`transformer`／`reloaditem`、`roseinspectable`／`roseinspectableuser` 六页；主代理复核换装会话、变换恢复缺口、装填分派及残影冷却，统一验收与共享登记，固定清单 91／98。本批登记到人物、生物关联、物品交互和技能主题，items 保持160行。

- **2026-10-26 通用 C68～C70 已交付（Codex）**：三个子代理分别完成 `stretcher`／`simplemagicgrower`、`forgerepair`／`forgerepairable`、`custombuildmanager`／`symbolswapdata` 六页；主代理复核消费者、修理优先级和清理边界后统一验收与共享登记，固定清单 85／98，G09 与 G10 全部覆盖。

- **2026-10-26 通用 C65～C67 已交付（Codex）**：三个子代理分别完成 `coach`／`grouptargeter`、`strafer`／`stuckdetection`、`distancefade`／`replayproxy` 六页；主代理复核并统一验收及共享登记，明确鼓舞不筛阵营、目标池不钳权重、侧移朝向与位移分工，以及视觉／身份数据组件边界；固定清单 79／98。

- **2026-10-26 通用 C62～C64 已交付（Codex）**：三个子代理分别完成 `commander`／`crewmember`、`joustsource`／`joustuser`、`embarker`／`amorphous` 六页；主代理核对指挥回调、船员退队、冲刺门禁和登离船收尾边界，统一验收与登记，固定清单 73／98。

- **2026-10-26 通用 C59～C61 已交付（Codex）**：三个子代理完成 `playerspeedmult`／`unevenground`、`heavyobstaclephysics`／`heavyobstacleusetarget`、`pushable`／`physicsmodifiedexternally` 六页；主代理复核、统一验收和登记，固定清单 67／98。补正 playerspeedmult 标准移除路径的 dirty 回调缺口；旧 carefulwalker 仅补通知源链接，不刷新其基线。

- **2026-10-26 通用 C56～C58 已交付（Codex）**：三个子代理完成 `batspawner`／`circler`、`shedder`／`distancetracker`、`amphibiouscreature`／`carefulwalker` 六页；主代理复核并统一验收和登记，固定清单 61／98，G07 的 19 项均已覆盖。batspawner 为未完整导出代码的边界说明，distancetracker 明确无累计输出，不作为可用生成／里程接口推荐。

- **2026-10-26 已交付（本对话，无子代理）**：待审线索「雨滴门控 2＋尖刺发射回退 4」转正为机制页 [[reference/mechanisms/rain-fx-gating]] 与 [[reference/mechanisms/spike-launcher]]，主代理直接编写并验收登记毕；待审线索 18→12，明细见 W06。与 Codex C53～C55 并行零碰撞。

- **2026-10-26 通用 C53～C55 已交付（Codex）**：复用三个子代理，分别完成 `followerherder`／`squadmember`、`minionspawner`／`objectspawner`、`rampingspawner`／`schoolspawner` 六页；主代理复核、统一验收和登记，固定清单 55／98。只给既有 lureplantspawner 补眼球草机制指针，未刷新该旧页基线；与另一对话机制页批保留各自增量。

- **2026-10-26 通用 C50～C52 已交付（Codex）**：三个子代理分别完成 `hatchable`／`hideout`、`followermemory`／`leaderrollcall`、`butterflyspawner`／`lureplantspawner` 六页，主代理复核、验收并统一登记索引／覆盖清单／基线／日志，固定清单 49／98。与另一对话新增组件批互不改页，保留其并行增量。

- **2026-10-26 通用 C49 已交付（Codex）**：`formationleader`／`formationfollower` 两页，目标为 `wiki/reference/components/<组件名>.md`；自行完成源码核验、局部验收、叶级登记与两页基线。固定清单 43／98；避开另一对话认领的 6 个新增组件及其互链目标页，保留共享文件中的他方增量。

- **2026-10-26 已交付（本对话）**：26148036 新增组件第二批 6 页——`corruption`／`rockybossspawner`／`atriumritualorgantracker` 与 `deflectable`／`autojiggle`／`worldstaticlayouts`——两子代理完成、主代理验收登记毕；至此 26148036 新增 14 组件全部建齐，明细见 W06。其余执行者恢复正常选题。

- 通用 C46～C48 已交付：三个子代理分别完成 `cookbookupdater`／`feasts`、`birdattractor`／`birdspawner`、`luckitem`／`luckuser` 六页；主代理核验个人记录与世界管理、自然排程与显式生成、幸运值和概率消费者边界并统一登记，固定98项进度41／98。其他执行者避免重复。

- 通用 C43～C45 已交付：三个子代理分别完成 `dryer`／`dryingracksaltcollector`、`digester`／`incinerator`、`forcecompostable`／`incrementalproducer` 六页；主代理复核旧式与当前路径、持有对象及宿主协作并统一登记，固定98项进度35／98。物品加工导航已局部拆分，其他执行者避免重复。

- 通用 C40～C42 已交付：三个子代理分别编写 `spooked`／`decay`、`tackler`／`tacklesketch`、`aura`／`spell` 六页；主代理续接完成语义复核、光环／效果载体验收和公共登记，固定 98 项进度 29／98。满载交互导航已局部拆分，其他执行者避免重复。

- 通用 C37～C39 已交付：三个子代理分别完成 `cyclable`／`powerload`、`autoterraformer`／`dataanalyzer`、`grabbable`／`summoningitem` 六页；废弃与标记组件采用简要说明，主代理复核并统一登记，固定 98 项进度 23／98，其他执行者避免重复。

- 通用 C34～C36 已交付：三个子代理分别完成 `frostybreather`／`nonslipgritpool`、`teamattacker`／`teamleader`、`groundpounder`／`targettracker` 六页；主代理核验源码边界、模式限定并统一登记，固定 98 项进度 17／98，其他执行者避免重复。

- 通用 C31～C33 已交付：按本轮要求启用三个子代理，分别完成 `aoeweapon_base`／`aoeweapon_leap`／`aoeweapon_lunge`、`slipperyfeet`／`slipperyfeettarget`、`socketable`／`socketholder` 七页；主代理定向复核并统一登记，固定 98 项进度 11／98，其他执行者避免重复。

- 通用 C29～C30 已交付：两个子代理分别完成 `attunable`／`attuner`、`linkeditem`／`linkeditemmanager` 四页；主代理定向复核并统一登记，固定 98 项进度 4／98，其他执行者避免重复。

- 通用 C28 已交付：本轮不启用子代理，只完成 `itemweigher`／`weighable` 两页；称重动作、重量数据／捕获者与读档边界核验后统一登记，本轮到此收尾。

- 通用 C26～C27 已交付：两个子代理分别完成 `portablestructure`／`poppable`、`sittable`／`inkable` 四页；主代理定向复核并统一登记，其他执行者避免重复。

- 通用 C24～C25 已交付：两个子代理分别完成 `wax`／`waxable`、`drawingtool`／`drawable` 四页；主代理定向复核并统一登记，其他执行者避免重复。

- 通用 C22～C23 已交付：两个子代理分别完成 `teacher`／`recipescanner`、`maprecorder`／`mapdeliverable` 四页；主代理定向复核并统一登记，其他执行者避免重复。

- 通用 C20～C21 已交付：两个子代理分别完成 `papereraser`／`erasablepaper`、`fencerotator`／`closeinspector` 四页；主代理定向复核并统一登记，其他执行者避免重复。

- 通用 C18～C19 已交付：两个子代理分别完成 `shaver`／`repellent`、`bottler`／`searchable` 四页；主代理复核并统一登记，其他执行者避免重复。

- 通用 C16～C17 已交付：两个子代理分别完成 `catcher`／`thief`、`lighter`／`firebug`；主代理定向复核与统一登记，其他执行者避免重复。

- 通用 C15 已交付：本轮不启用子代理，只编写 `gravedigger`／`gravediggable` 两页；工具先执行、目标后执行与动作前置拒绝的边界已核验，本轮不再扩展其他主题。
- 通用 C13～C14 已交付：两个子代理分别完成 `sanityauraadjuster`／`temperatureoverrider` 和 `staffsanity`／`epicscare`，主代理定向复核与统一登记。四页保持 source-reviewed；后续按用户最新要求保留两个子代理，避免重复生成。

- 通用 C10～C12 已交付：三个子代理分别编写 `occupiable`／`occupier`、`firedetector`／`fan`、`appraisable`／`murderable` 六页，主代理复核与统一登记，见 [本轮报告](docs/reports/w05/component-occupation-fire-appraisal.md)；其他执行者不重复生成。

- 通用 C07～C09 已交付：三个子代理分别处理 `inventoryitemholder`／`itemstore`、`foodmemory`／`fueler`、`terraformer`／`terraformerremoveable` 六页，主代理统一复核与登记，见 [对应报告](docs/reports/w05/component-storage-food-terrain.md)；其他执行者不重复生成。

- 通用 C04～C06 已交付：三个子代理分别编写 `preservative`／`compostingbin`、`container_proxy`／`containerinstallableitem`、`battery`／`batteryuser` 六页，主代理复核、校验并统一登记，见 [对应报告](docs/reports/w05/component-preservation-containers-batteries.md)。其他执行者不重复生成；`container_proxy` 页面为 `container-proxy.md`。

- 本对话此前已交付通用 C01～C03：`hauntable`、`talker`、`inventoryitemmoisture`、`inventoryitemtemperature`、`foodaffinity`、`fuelmaster`，见 [前轮报告](docs/reports/w05/component-common-foundations.md)；其他 AI 不重复生成这些页面。
- 固定98项各组已全部覆盖，首轮索引重整也已完成；items为114行、world-creatures为77行，按功能细分的16个主题仍保持每篇唯一登记。后续从剩余候选按任务恢复，避免仅为扩分类新建占位索引。
- 另一 AI 已接收第 92 组 `healthbar`／`healthsyncer`（修正双端描述、移除风险与收窄结论）并交付第 93～97 组 10 页：`sheltered`、`rainimmunity`、`lightningblocker`、`constructionsite`、`constructionbuilder`、`fishingrod`、`oceanfishingrod`、`oar`、`walkableplatform`、`boatphysics`；该 AI 已完成语义复核、覆盖登记与主题叶级登记，本对话保留其增量、未代其验收。其余执行者选题前检查实际页面、覆盖清单与认领，不仅看旧批次号。
- 各执行者只改所属知识页；公共索引、覆盖 CSV、PLAN、设计和日志在合并时统一处理，写前重读当前内容，保留他人增量。新批次以清单实际占用为准，不争用第 92 批编号。

### 存量 Wiki 修整（先于下一批扩库）

本次检视确认现有页面可继续使用，不推倒重写。以已安装 dst-wiki Skill 的当前维护规则执行；四阶段分别验收，不将全部格式统一或新增工具作为扩库前置。

| 阶段 | 范围与完成条件 | 状态 |
|---|---|---|
| 1：源码引用 | 已修正 9 页的 26 处引用，消除原有 17 个警告；所选页格式／链接通过，336 处源码引用 strict 检查为 0 error／0 warning；日期／日志同步，机制结论与 Lua 示例未改 | 已完成（2026-10-17） |
| 2：证据与来源边界 | 24 页验证记录改为具体覆盖与未覆盖范围；8 页模组观察注明未分发／未运行边界，[来源复核](docs/reports/w05/local-mod-references-20261017.md) 留存 12 份 Lua／3 份 modinfo 指纹与实际版本线索；24 页格式／链接及 477 处源码引用 strict 检查通过，未升级验证等级 | 已完成（2026-10-17） |
| 3：阅读成本 | 精简 inventoryitem／follower／combat／debuffable 的重复说明和引用，快速判断链接已有细节；4 页净减 2,519 个实际 UTF-8 字节，inventoryitem 从 16,182 降至 15,046 字节；保留接口／流程／案例证据及完整 Lua，190 处引用 strict 通过 | 已完成（2026-10-17） |
| 4：导航与收尾 | 删除仅有占位的 Prefab／StateGraph 两张索引，reference 直接说明未建页与源码回退；62 篇知识页＋7 篇导航及唯一登记通过，1,857 处源码引用 strict 为 0 error／0 warning；两条实际章节读取完整，无截断 | 已完成（2026-10-17） |

检视基线：Steam buildid 25643519；62 篇知识页＋9 篇导航的格式／链接／登记通过；1,890 个源码引用无 error、17 个 warning。源码语义只做定向抽查，未运行 DST，不将结构通过视为全库技术结论已验证。历史报告保留原文，新结果登记于本节与日志。

本轮修整已收尾：只定向复查所改引用、验证范围、重复内容和导航，不代表全库语义或游戏运行已验收；Lua 示例未改，未重跑语法检查，未执行独立模型评测。两条章节读取为 inventoryitem 生命周期与 follower 典型接入，均保留前提／完整示例（有代码时）／验证记录；字节变化仅是读取规模代理，不换算 token，也不据此重写其他页面。组件主题导航已于第十一批前实施，Prefab／配方及后续批次队列保留。

### 近期执行顺序

**FX 当前覆盖与续作（2026-11-01，目标 Steam buildid 26335394）**

- [x] 2026-11-04 方向调整：暂停新建知识页，先维护既有页面（返回链、索引登记、链接与元数据）；FX 扩库与外围核对暂缓，不按 96／341 等缺口建页。本轮已修 46 个变更页返回链、8 个新页补链、105 个历史页；覆盖检查默认只报 Prefab 页（0 发现），`--scope all` 时才显示 4 条索引提醒与 21 条别名 info，后两类仅记录、不处理。
- [ ] 后续恢复建页时注意（来源完整性）：字面 `Prefab("…")` 只是候选下界（2,220 名；279 处变量调用、199 处 `MakePlacer`、254／1,621 文件无字面名；249 个 spawn-only，其中 94 个 FX），不能当完整注册表；动态／拼接名先定位 `file:line` 或成族记录，不猜。
- [ ] 后续恢复建页时注意（校验口径）：FX 不要求 `STRINGS.NAMES`／po 官方名；别名为检索辅助、非必需且不要求唯一；注册覆盖缺口只作 candidate／backlog，默认 warning，不设 `--strict` 门槛；新页仍写返回登记叶索引（可保留 hub），改动页先用 `--changed` 复核。清单与缺口在 `output/coverage/`（忽略目录，未提交），工具说明见 T `references/wiki-validation.md`。

- [x] 新旧快照新增盘点：26230496→26335394 的 `fx.lua` 仅六条落叶堆特效更换 sound，注册与动态展开未变；专属 `prefabs/*_fx*.lua` 文件仍为 87 个，无增删，仅 `willow_flame_fx.lua` 内容改变。本次没有新增 FX 注册需要追加到扩库队列。
- 当前覆盖仍为 **32 个同族页、163／504 个通用表注册名，剩余 341 个注册名待覆盖**。341 是延续旧版的未覆盖注册集合，不是本次更新新增项，也不等于 341 篇待建页；已建 163 名的源码核验版本仍以各页为准，不因本次盘点视为已完成新版迁移。旧版逐注册 CSV 与历史报告保留原快照口径，实际覆盖及任务占用须结合现页与认领核对。
- [ ] 通用表后续扩库（**进行中，覆盖台账见 [fx-coverage-ledger-20261101](docs/reports/w05/fx-coverage-ledger-20261101.md)**）：341 名已按共享机制分 48 族登记（批1=万圣夜 leafpile 六注册优先＋火绒月绒＋碎裂对）；每批"派发→源码核验→写页→独立复审→验收→台账/索引/日志"闭环后滚动下一批。选题前核对他方认领，不将未覆盖数当作未认领数；专属 75 页与外围 380 文件核对在台账另节记账。
- [x] 2026-11-01 通用表 FX 批 8 完成：F21～F25 五页 18 注册；独立复审 P1×0、P2×6 已修（瓶子水中动画、水花地形条件、宝盒 recipe 门禁、交换失败边界、月脉冲端及中断）。8 文件格式与导航通过、strict 72 引用 0 错 0 警、5 页 record 后 29 组 0 漂移；无 Lua 示例，未运行游戏。累计 68 族 378／504 注册，剩 126 名；外围核对仍进行中，未提交推送。
- [x] 2026-11-01 通用表 FX 批 9 完成：F27、F28、F29、F31 共 4 页 26 注册；独立复审 P1×0、P2×4 已修（采矿目标 Transform 与 15 处调用、雕像 respawner 休眠、音乐组件回调绑定）。7 文件格式与导航通过、strict 109 引用 0 错 0 警、4 页 record 后 49 组 0 漂移；无 Lua 示例，未运行游戏。累计 72 族 404／504，剩 100 名；外围初筛仍未闭环，未提交推送。
- [x] 2026-11-01 通用表 FX 批 10 完成：F30、F32～F35 共 5 页 30 注册；独立复审 P1×1、P2×6 已修，含玩家受击者身份、死亡烟与树叶门禁、寄生与诅咒条件、火杖落点及旧页护盾职责。10 文件格式与导航通过、strict 229 引用 0 错 0 警（跳过 1 个不支持链接）、5 页 record 后 40 组 0 漂移；无 Lua 示例，未运行游戏。累计 77 族 434／504，剩 70 名；外围初筛未闭环，未提交推送。
- [x] 2026-11-01 通用表 FX 批 11 完成：F36～F41 六页 39 注册，独立复审修复触发门禁、动画资源名及无调用缺口；10 文件格式与导航通过、strict 118 引用 0 错 0 警、6 页 record 后 47 组 0 漂移；无 Lua 示例。角色索引按主题拆出角色与装备补充，累计 83 族 473／504，剩 31 名；未运行游戏、未提交推送。
- [x] 2026-11-01 通用表 FX 批 12 完成：F42～F46 五页 31 注册，独立复审 P1×0、P2×1 已修；9 文件格式与导航通过、strict 116 引用 0 错 0 警、5 页 record 后 59 组 0 漂移；无 Lua 示例。累计 88 族 504／504，逐名统一对账及外围 380 文件核对仍待完成；未运行游戏、未提交推送。
- [x] 2026-11-05 FX 终检完成：504 名源码展开、台账及三叶索引逐名对账无遗漏／重复；整库 1,146 页页面／导航通过；严格源码链接 29,309 处 0 错、27 条存量警告（strict 未通过）、987 项不支持链接跳过；外围 380 文件逐文件闭环（187＋101＋26＋66），最终报告落盘 docs/reports/w05/fx-final-report-20261105.md。
- [x] 2026-11-02 外围批 A 完成：`abigail_flower.lua`、`abysspillar.lua`、`acidraindrop.lua` 三文件的 8 个 FX 注册建 3 页，主体不计 FX；独立复审修复 4 项边界。6 文件页面／导航通过、strict 101 引用 0 错 0 警（另跳过 1 个不支持链接）、3 页 record 后 17 组 0 漂移；未运行游戏。另核已承载的 backpack、heatrock、sparks、weaponsparks 四文件，补全背包资源字段及 WX78 直接生成点；stafflight 的 emberlight 仍待覆盖。
- [x] 2026-11-02 外围批 B 完成：`diseaseflies.lua`、`explode_small.lua`、`grottopool_sfx.lua` 三文件的 12 个 FX 注册建 3 页；独立复审修正水声旧池有效门禁。7 文件页面／导航通过、strict 116 引用 0 错 0 警（另跳过 1 个不支持链接）、3 页 record 后 24 组 0 漂移；未运行游戏。
- [x] 2026-11-04 外围批 C 完成：`stafflight.lua`（4 注册，3 既有宿主页＋`emberlight` 新页，整文件闭合）、`lightning.lua`（2 注册）、`staffcastfx.lua`（8 注册）建 3 页，并把 `thunder_close.lua`、`thunder_far.lua`、`moonstorm_lightning.lua` 3 个独立文件并入 `lightning.md` 扩页，共 6 文件 17 注册。独立复审 P1×0、P2×3（标题漏注册、预载数、非官方名）＋P3×9 全部修复。8 文件页面／导航通过，strict 166 引用 0 错 0 警（跳过 2 个不支持链接），3 页 record 后 35 组 0 漂移；无 Lua 示例、未运行游戏。`fx-index` 15 KiB 达拆分阈值，拆出 `fx-world-index` 并在父索引两行登记。外围 380 文件中 10 个既有＋6 个批 C 已闭环、19 个批 D 待复审，345 个未闭环，见覆盖台账。
- [x] 2026-11-04 外围批 D 完成：瞄准圈家族 8 文件 48 注册（`reticuleaoe` complex）、天气降水 7 文件 7 注册（`rain`）、海面波浪 3 文件 7 注册（`wave_med`）、施法光源 1 文件 2 注册（`staff_castinglight`）共 4 页 64 注册。独立复审 P1×1（target 被误写为自然淡出）＋P2×6（渲染层、生成点漏项、SetChargeScale 归属、缺口推断、sources 缺 14 文件）＋P3×12 已修；4 页页面／导航通过、strict 266 引用 0 错 0 警、record 后 90 组 0 漂移；未运行游戏。外围 345 文件仍待逐文件闭环。
- [x] 2026-11-04 外围批 E 完成：火焰烟雾 `fire`（15 注册，complex）、光源实体 `cavelight_small`（13 注册，complex）、威胁预警 `houndwarning_lvl1`（29 注册，complex）3 页 57 注册。独立复审 P1×1（远古灯柱官方名漏判）＋P2×9（生成点漏宿主、sources 缺项、areas.lights 生成链、暗影阈值/夜晚门控、洞穴挂载）＋P3×12 已修；3 页页面／导航通过、strict 235 引用 0 错 0 警、record 后 104 组 0 漂移；未运行游戏。
- [x] 2026-11-04 外围批 F 完成：宿主附属层 `fence_gate_anim`（12 注册／6 文件）、远古吊灯与石像鬼 `archive_chandelier`（8 注册／2 文件）、熔炉化石碎裂 `lavaarena_beetletaur_fossilized_break_fx_right`（7 注册／2 文件）3 页 27 注册。独立复审 P1×1（archivemanager 缺失语义反转）＋P2×3（废料堆生成点、掉落镜头归属、CreateBreakFX build 名）＋P3 若干已修；3 页页面／导航通过、strict 238 引用 0 错 0 警、record 后 40 组 0 漂移；未运行游戏。
- [x] 2026-11-04 外围批 G 完成：装备跟随光效 `armor_lunarplant_glow_fx`（7 注册／6 文件）、天体英雄 `alterguardian_laserscorch`（8 注册／4 文件）、藏影女王 `charlie_boss_aoe_flame_fx`（7 注册／3 文件）3 页 22 注册。独立复审 P1×0、P2×4（客户端子帧创建点、空激光无视觉与 11+2 道、Despawn 路径、非专服主机建 rotator）＋P3 若干已修；3 页页面／导航通过、strict 276 引用 0 错 0 警、record 后 31 组 0 漂移；未运行游戏。
- [x] 2026-11-04 外围批 H 完成：冲击与爆点 `groundpoundring_fx`（6 注册／6 文件）、独眼巨鹿激光 `deerclops_laserscorch`（5 注册／1 文件）、贴地贴水 `ice_puddle`（4 注册／4 文件）3 页 15 注册。独立复审 P1×0、P2×3（hitrings 与结算半径、willow_ember 死代码表述、停酸雨不退场）＋P3 若干已修；3 页页面／导航通过、strict 192 引用 0 错 0 警、record 后 38 组 0 漂移；未运行游戏。
- [x] 2026-11-04 外围批 I 完成：生物残骸与植物 `mushtree_tall_webbed_burntfx`（6 注册／6 文件）、实验台与粒子 `madscience_lab_goop`（6 注册／4 文件）、月装置 `moon_device_pillar`（4 注册／2 文件）3 页 16 注册。独立复审 P1×1（`leveldirty` 引擎脏事件误判）＋P2×9（工厂总述、SetParent 级联断言、壳不能父挂、mist 端、Hide 互斥范围、slipperyfeet 证据链、月装置漏生成点与父挂）＋P3 若干已修；3 页页面／导航通过、strict 221 引用 0 错 0 警、record 后 42 组 0 漂移；未运行游戏。
- [x] 2026-11-04 外围批 L 完成：地面贴片 `crabking_claw_shadow`（6 注册／5 文件）、宿主挂接 `king_cane_fx`（7 注册／7 文件）、灯具光源 `wormlight`（16 注册／10 文件）3 页 29 注册。独立复审 P1×0、P2×5（hutch 交叉计数、背层帧同步死代码、soundpath 语义、背层反登记、代理结束后仍有光写反）＋P3 若干已修；3 页页面／导航通过、strict 321 引用 0 错 0 警、record 后 46 组 0 漂移；未运行游戏。
- [x] 2026-11-04 外围批 M 完成：附着跟随 `gestalt_trail`（6 注册／5 文件）、小型一次性物件 `grasspartfx`（8 注册／7 文件）、有表现载体 `eye_charge`（4 注册／4 文件）3 页 18 注册。独立复审 P1×1（chum_aoe 读档任务重建写反）＋P2×6（熔合暗影名、groundshadowhandler 双端、SetPrefabNameOverride 锚点、SGshrouden 撒块范围、生角链顺序、渔网联网范围、junk 参数来源）＋P3 若干已修；3 页页面／导航通过、strict 262 引用 0 错 0 警、record 后 67 组 0 漂移；未运行游戏。
- [x] 2026-11-04 外围批 O 完成：皮肤火星 `carnival_sparkle_bush`（4 注册／4 文件）、地面痕迹 `damp_trail`（3 注册／3 文件）、粒子降水 `raindrop`（2 注册／2 文件）3 页 9 注册。独立复审 P1×0、P2×4（damp_trail 任务引用与壳/代理消失方向、sparkle 饥饿/饱食条件与壳 Follower 表述）＋P3 若干已修；3 页页面／导航通过、strict 174 引用 0 错 0 警、record 后 48 组 0 漂移；未运行游戏。
- [x] 2026-11-04 外围批 Q 完成：影柱法术 `shadow_pillar`（4 注册／1 文件）与梦魇鞍具 `saddle_shadow_fx`（2 注册／2 文件）2 页 6 注册。独立复审 P1×0、P2×6（`t < dt` 条件、1.25 秒停扫描、重施驱散主路径、柱体/目标读档差异、DetachChild 实参、脚印池回收条件）＋P3 若干已修；2 页页面／导航通过、strict 125 引用 0 错 0 警、record 后 22 组 0 漂移；未运行游戏。
- [x] 2026-11-04 外围批 P 完成：阿比盖尔子特效 `abigail_retaliation`（5 注册／1 文件）、海树表现 `oceantree_ripples_short`（10 注册／2 文件）、怀表传送门 `pocketwatch_warp_marker`（5 注册／5 文件）3 页 20 注册。独立复审 P1×4（暗影怨咒方向、怨咒触发路径、柱体 build 名、SetParent 当 AddChild）＋P2×6＋P3 若干已修；3 页页面／导航通过、strict 239 引用 0 错 0 警、record 后 35 组 0 漂移；未运行游戏。
- [x] 2026-11-04 外围批 R 完成：睡眠云 `sleepcloud`（5 注册／2 文件）与装备持握层 `staff_lunarplant_fx`（6 注册／5 文件）2 页 11 注册。独立复审 P1×2（漏孢子云官方名与 po sources、剑破损不隐藏 blade）＋P2×5＋P3 若干已修；2 页页面／导航通过、strict 191 引用 0 错 0 警、record 后 35 组 0 漂移；未运行游戏。
- [x] 2026-11-04 外围批 S 完成：影陷阱 `shadow_trap`（2 注册）、暗影消散 `shadow_despawn`（5 注册）、散件贴地 `scorchedground`（3 注册）3 页 10 注册。独立复审 P1×0、P2×6＋P3 若干已修；3 页页面／导航通过、strict 237 引用 0 错 0 警、record 后 69 组 0 漂移；未运行游戏。
- [x] 2026-11-04 外围批 T 完成：书页挂载 `fx_book_mount`（12 注册）、黏液飞溅 `goosplash`（9 注册）、暗影光源 `nightmarelightfx`（8 注册）3 页 29 注册（5 文件）。独立复审 P1×0、P2×0；3 页页面／导航通过、strict 206 引用 0 错 0 警、record 后 check 0 漂移、2 段 Lua 语法通过；未运行游戏。2026-11-05 接手会话第二次逐锚点复审计实前次验收，修复 P2×2（两页验证记录引用旧尺寸预算，改当前 240/24 与 480/48），台账批 T 复审计实段与建页清单见覆盖台账。
- [x] 2026-11-05 外围收尾完成：51 页组全部 accepted（50 新页＋1 增修），380 候选＝187 闭环＋101 needs-page＋26 blocked＋66 excluded；全库 validate_page --check-indexes 1,146 页 0 failed、导航 0 问题，strict links 29,309 引用 0 error／27 条存量 warning（12 个既有页，strict 未通过），source hashes 7,415 组 0 漂移；外部独立检视 P1×2／P2×7 全部采纳并复核；台账/PLAN/log 已对齐，最终报告落盘 docs/reports/w05/fx-final-report-20261105.md。未运行游戏、未提交、未推送。
- [x] 2026-11-05 外部独立检视采纳：U15～U17 九页报告 P1×2（WAG 电系倍率、WX-78 模块覆盖）、P2×7（SetBeam 采样缓存、horn entitysleep、hotcold 清理、terrarium 昼夜、eye 状态表／goop、arm 消散）、R-01（strict 口径）全部采纳；七页修订后 record 7 页 0 漂移，全库复跑 1,146 页 0 failed、29,309 引用 0 error／27 存量 warning、7,415 组 0 漂移；最终报告与台账已回写。未提交、未推送。
- [x] 2026-11-05 FX 待建清单对齐：逐组核对 51 行的实际页面、Prefab aliases 与验收批次，明确 accepted／needs-page，修正错配批次与验收段已注明的注册数；组 11 恢复待建，待建 9 组均无页。交接 v3 同步数字、草稿／导航验收说明与可复制命令，并将“已知坑”分为工作区约定和本批定位线索，移除 shell 直接写文本、固定并发上限等泛化建议。本次只维护进度文档与日志，不新增知识页、不 record、不重做技术复审或游戏验证；未提交或推送。
- [x] 2026-11-05 外围批 U1 完成：海上暗影附层 `oceanhorror_ripples`（2 注册）、Boss 战表现 `rocky_snip_fx`（4 注册，含 worm_boss 裂纹）、桥面表现 `rope_bridge_fx`（3 注册，含 vine_bridge）3 页 9 注册／9 文件。主会话独立复审 P1×0、P2×0、P3×1 已修；3 页 validate --check-indexes 通过、strict 0 错 0 警、record 后 32 组 0 漂移、索引两叶登记；未运行游戏。
- [x] 2026-11-05 外围批 U2 完成：化石基座 `fossilspike_base`（2 注册）、裂隙视觉与河灯影层 `lunarrift_terraformer_visual`（2 注册）、碎裂爆裂族 `shatter`（complex，8 注册／8 文件）3 页 12 注册／12 文件。主会话独立复审 P1×0、P2×0、P3×2（po 键名笔误）已修；3 页 validate --check-indexes 通过、strict 0 错 0 警、record 后 51 组 0 漂移、索引 fx-world-index 登记；未运行游戏。54 个未分诊候选同日全部分诊完毕（blocked 15／excluded 26／needs-page 13 文件入组 41-51）。
- [x] 2026-11-05 外围批 U3 完成：落石尘与掉落物碎裂 `cavein_dust_low`（complex，9 注册／2 文件）、弹弓减益 `slingshotammo_purebrilliance_debuff_fx`（3 注册／1 文件）、长矛突刺 `spear_gungnir_lungefx`（3 注册／2 文件）3 页 15 注册／5 文件。主会话独立复审 P1×0、P2×0、P3×0 零发现；3 页 validate --check-indexes 通过、strict 0 错 0 警、record 后 25 组 0 漂移、索引两叶登记；未运行游戏。
- [x] 2026-11-05 外围批 U4 完成：宝箱沙棺 `terrariumchest_fx`（2 注册／3 文件）、薇诺娜设备 `winona_battery_high_shatterfx`（5 注册／5 文件）、图勒皇冠 `forcefieldfx`（1 注册／2 文件）3 页 8 注册／10 文件。主会话独立复审 P1×0、P2×0、P3×0 零发现；3 页 validate --check-indexes 通过、strict 0 错 0 警、record 后 31 组 0 漂移、索引两叶登记；未运行游戏。
- [x] 2026-11-05 外围批 U5 完成：WX78 `wx78_big_spark`（4 注册／4 文件）、WAGboss `wagboss_beam_fx`（4 注册／3 文件）、沃拓克斯 `wortox_soulecho_buff_fx`（6 注册／3 文件）3 页 14 注册／10 文件。主会话独立复审 P1×0、P2×0、P3×0 零发现，台账勘误 3 条（contained 注册名、leg_fx 语义、nabbag 注册数）；3 页 validate --check-indexes 通过、strict 0 错 0 警、record 后 30 组 0 漂移、索引两叶登记；未运行游戏。
- [x] 2026-11-05 外围批 U6 完成：沃特 `wurt_shadow_merm_planar_fx`（5 注册／2 文件，清单外 chargedfx 两变体并入）、海草花粉 `waterplant_pollen_fx`（2 注册／1 文件）、海树柱页增修 `oceantree_ripples_short`（并入 watertree 两注册，10→12）净 2 新页＋1 增修 9 注册／4 文件。主会话独立复审 P1×0、P2×0、P3×0 零发现，台账勘误 2 条（chargedfx 同 build 异动画、watertree 异 bank）；validate --check-indexes 通过、strict 0 错 0 警、record 37 组 0 漂移；未运行游戏。
- [x] 2026-11-05 外围批 U7 完成：陷阱 `trap_fumarole_burn_fx`（2 注册／2 文件）、龙舟竞赛 `boatrace_start_flag`（4 注册／2 文件）、暗影散件 `shadow_bishop_fx`（4 注册／4 文件）3 页 10 注册／8 文件。主会话独立复审 P1×0、P2×0、P3×0 零发现，台账批 7 勘误 1 条（feather_loop 表述）；validate --check-indexes 通过、strict 0 错 0 警、record 3 页；未运行游戏。
- [x] 2026-11-05 外围批 U12 完成：冬宴 `wintersfeastbuff_fx`（2 注册）、散件组 `witchbroom_fx`（3 注册）、保暖伙伴 `beargerfur_sack_frost_fx`（2 注册）3 页 7 注册／6 文件。主会话独立复审 P1×0、P2×0、P3×0 零发现，台账批 9 勘误 1 条（轨迹池节拍）；validate --check-indexes 通过、strict 0 错 0 警、record 3 页、索引两叶登记；未运行游戏。
- [x] 2026-11-05 外围批 U13 完成：蝙蝠系 `bat_bosshat_fx`（5 注册／3 文件）、主教 `bishop_targeting_fx`（1 注册）、拟态蠕虫 `itemmimic_revealed_shadow`（1 注册）3 页 7 注册。主会话独立复审 P1×0、P2×0、P3×0 零发现；validate --check-indexes 通过、strict 0 错 0 警、record 3 页、索引登记；未运行游戏。
- [x] 2026-11-05 外围批 U14 完成：月亮石 `positronbeam_back`（3 注册）、织影者盾 `stalker_shield1`（5 注册）、毒菌帽 `toadstool_cap_absorbfx`（2 注册）3 页 10 注册／4 文件。主会话独立复审 P1×0、P2×0、P3×0 零发现，勘误 2 条（反伤归属表述、毒菌帽触发链）；validate --check-indexes 通过、strict 0 错 0 警、record 3 页、索引两叶登记；未运行游戏。
- [x] 2026-11-05 外围批 U15 完成：潜伏梦魇 `ruinsnightmare_horn_attack`（1 注册）、暗影之手／恐惧之手 `shadowhand_arm`（2 注册）、暗影憎恶死亡水洼之眼 `shrouden_eye_fx`（1 注册）3 页 4 注册／5 文件。主会话独立复审 P1×0、P2×2（arm 页光照监听范围、eye 页移动中玩家条件）、P3×3 已修；3 页 validate --check-indexes 通过、strict 0 错 0 警、record 3 页 check 0 漂移；修复 fx-creature-index 两处表格断裂并登记 3 行；未运行游戏。
- [x] 2026-11-05 外围批 U16 完成：小惊吓 `hotcold_fx`（1 注册）、盒中泰拉 `terrarium_fx`（1 注册）、WAGboss 群 `wagboss_missile_target_fx`（5 注册）3 页 7 注册／6 文件。主会话独立复审 P1×0、P2×3（hotcold 放弃路径清理、terrarium 白天计时、WAGboss 混合数）、P3×3 已修；`wagpunk_cagewall_fx` 无生成点按诚实缺口记录；3 页 validate --check-indexes 通过、strict 0 错 0 警、record 3 页 check 0 漂移；索引两叶登记；未运行游戏。
- [x] 2026-11-05 外围批 U17 完成：WX-78 `wx78_taser_projectile_fx`（1 注册）、枕头大战 `yotr_fightring_torch`（4 注册＋placer）、法术精通 `spellmasterybuff`（3 注册）3 页 8 注册＋1 placer／5 文件。主会话独立复审 P1×0、P2×1、P3×1 已修并校正 8 处引用边界；spellmasterybuff 事件包缺口如实保留；3 页 validate --check-indexes 通过、strict 0 错 0 警、record 3 页 check 0 漂移；索引两叶登记；未运行游戏。
- [x] 2026-11-05 外围批 U8 完成：虚布装备 `voidcloth_boomerang_fx`（5 注册／3 文件）、宝库设施 `vault_compass_visual`（4 注册／4 文件）、温蒂墓碑 `wendy_gravestone_rebirth_fx`（2 注册／1 文件）3 页 11 注册／8 文件。主会话独立复审 P1×0、P2×0、P3×0 零发现，台账勘误 2 条（ground_pattern 15→18 处、launch/impact fx 归属补记）；validate --check-indexes 通过、strict 0 错 0 警、record 33 组 0 漂移、索引两叶登记；未运行游戏。
- [x] 2026-11-05 外围批 U9 完成：帽子族 `hats.md`（complex，12 注册）、蘑菇树焦壳 `mushtree_tall_burntfx`（7 注册）、月坛破碎 `moon_altar_break`（3 注册）3 页 22 注册。主会话独立复审 P1×0、P2×0、P3×0 零发现，台账批 10 勘误 1 条（mask_halfwit_fx 生成点在舞台剧）；validate --check-indexes 通过、strict 0 错 0 警、record 3 页、索引两叶登记；未运行游戏。
- [x] 2026-11-05 外围批 U11 完成：档案机关 `archive_ambient_sfx`（5 注册）、远古大门 `atrium_gate_activatedfx`（4 注册）、气球 `balloon_held_child`（4 注册）3 页 13 注册／7 文件。主会话独立复审 P1×0、P2×0、P3×0 零发现（三处源码拼写陷阱入页）；修复 U5/U7 索引登记错位（fx-creature-index 两行移回表格）；validate --check-indexes 通过、strict 0 错 0 警、record 3 页；未运行游戏。
- [x] 2026-11-05 外围批 U10 完成：枕头碎屑与挖起 `attackfx_handpillow_petals`（complex，7 注册）、荆棘尖刺 `bramblefx_armor`（3 注册）、焦地蜂蜜 `burntground`（4 注册）3 页 14 注册／5 文件。主会话独立复审 P1×0、P2×0、P3×0 零发现（husk 独占接线新发现）；validate --check-indexes 通过、strict 0 错 0 警、record 3 页、索引两叶登记；未运行游戏。
- [x] 2026-11-01 FX 评审修正：码头示例改为世界服务端事件回调并修正 dont_toss_loot；灵药示例补足前提、区分玩家 buff 与 FX 选名；蜻蜓龙重击最多 8 枚、补 combat 前提与代理／本地清理区分；蟹王闪光、圆烟的符号绑定限制于代理；蜂烟声音及落点代理生命周期纠正。6 页及两个索引结构通过，strict 294 引用 0 错 0 警，4 段 Lua 语法通过，6 页 85 组基线 0 漂移；未运行游戏。
- [x] 2026-11-01 通用表 FX 扩库批 7 完成（60 族 333→**63 族 360**；本会话批 1～7 累计新建 33 页 197 注册，**批 8 起交接其他执行者，交接检查点见覆盖台账"交接检查点"节**）：读书施法 18 注册（fx_book_moon complex 页：Book:OnRead fx/fxmount 选名链＋books.lua def fn 内按受效对象生成两路径、11 本 def 全列、无 fxmount 四书骑乘回落 SetSixFaced、temperature/research_station 仅 deps 无 fx、fish 贴地渐隐专属 fn）＋冬季盛宴餐桌枯竭 2 注册（winters_feast_depletefood：首派被停代理已写入、本批代理独立核验通过）＋狂欢节 7 注册（carnival_confetti_fx：炮庆/开包按季拼名/射击三族＋高尔夫回传与小鸡赛跑复用）。独立复审修正 P1×1（def 表缺 book_rain）＋P2×3（sources 补 3 文件、第 4 处声明锚、MakeFX 锚点偏移；aliases 缺 fish 属误报未改）。5 文件 validate --check-indexes 全过、strict 0 错 0 警、3 段 Lua 探针通过、record 3 页 0 漂移、diff --check 干净。未运行游戏、未重建缓存、未提交推送。剩余 144 名（F21～F25、F27～F46）＋外围 380 文件核对交接续作。
- [x] 2026-11-01 通用表 FX 扩库批 6 完成（55 族 306→60 族 333）：天坑塌陷 10 注册拆两页（sinkhole_spawn_fx 7：warn/spawn 语义与洞穴 caveins 链、cavein_debris 非地震落石；glass_fx 3 归档页）＋暴食 5 注册（quagmire_portal_player_fx：生成点在事件数据包、快照不含的诚实缺口）＋熔炉流程 6 注册（lavaarena_portal_player_fx：登场树内无生成点、尸体复活链父挂）＋蜻蜓龙火系与暗影斩击 6 注册（firesplash_fx：勘误 fire 系非熔炉敌人而是蜻蜓龙三状态、shadowstrike 是麦斯威尔召唤物 lunge）。三写作子代理＋独立复审修正 P1×1（firesplash 逐环逐点非每环一枚）＋P2×11。7 文件 validate --check-indexes 全过、strict 0 错 0 警、4 段 Lua 探针通过、record 5 页 0 漂移、diff --check 干净。未运行游戏、未重建缓存、未提交推送。首派三代理全部超时无产出（重派成功，已记台账）。剩余 171 名续批（批 7＝读书施法＋冬季盛宴＋狂欢节）。
- [x] 2026-11-01 通用表 FX 扩库批 5 完成（50 族 286→55 族 306）：墨水胆汁洼地 4 注册（ink_puddle_land：溅起与洼底同 OnHit 先后两实体、IsOnOcean 海陆选名）＋冲滩浮渣 2 注册（flotsam_puddle：父挂伴生＋拾取碎裂，水中与打捞不出）＋鱼人水花 3 注册（merm_splash：六生成点跨世界王增益链）＋蜂蜜蜂群病害 4 注册（bee_poof_big：disease_puff 无生成点诚实缺口——病害系统休眠、实际特效 diseaseflies）＋蟹王伴随 7 注册（crab_king_bubble1：炮击冒泡重排死链每堆只出一枚、waterspout 击飞仅限 oceanfishable 鱼类、icefx 无活跃生成点）。三写作子代理＋独立复审修正 P1×3＋P2×9；锚点清理代理清零三页 24 条结构性警告。8 文件 validate --check-indexes 全过、strict 0 错 0 警、record 5 页 0 漂移、diff --check 干净；cannonball_used 补反链。未运行游戏、未重建缓存、未提交推送。剩余 198 名续批（批 6＝天坑＋暴食＋熔炉）。
- [x] 2026-11-01 通用表 FX 扩库批 4 完成（46 族 262→50 族 286）：船体侵蚀与垫料 12 注册拆两姊妹页（boat_grass_erode 7：沉没侵蚀成对＋垫料屑 17 事件源＋水草补丁到期动态拼名；fx_boat_crackle 5：船沉／码头／海冰破坏声画，boat_crackle 字段选名辨析）＋船件沉没与受击 8 注册（boat_mast_sink_fx：桅杆随船销毁链两段表现、防撞条 bumpertype 拼名窗口、小龙舟灯气球离场）＋船炮与迫击炮 4 注册（cannonball_used：dockmanager 恒在使落地恒选 _wood、mortarball 撞墙生成点死代码）。三写作子代理＋独立复审修正 P1×4（spawnnewboatleak 事件源 3→17 处全量；modmain 示例 GLOBAL. 前缀；drill 落点 y 论据；冰船时间线 10 处合计 11 枚）＋P2×12。8 文件 validate --check-indexes 全过、strict 0 错 0 警、4 段 Lua 探针通过、record 4 页 0 漂移、diff --check 干净；boatleak 补回链。未运行游戏、未重建缓存、未提交推送。剩余 218 名续批（批 5＝水洼＋鱼人＋蜂蜜＋蟹王）。
- [x] 2026-11-01 通用表 FX 扩库批 3 完成（40 族 237→46 族 262）：复活调谐 3 注册（attune_out_fx，命名方向与语义相反的误区、墓碑四流程复用）＋皮弗娄牛驯化变身拆页（beefalo_transform_fx）＋海面水花 8 注册（ocean_splash_med1：海鱼破水／浮标拼名／暗影傀儡步进等 20 文件消费点）＋水面杂项 11 注册按机制三页（splash_green 绿花与冲滩水洼／splash_sink 沉没协议与漩涡传送门与泥花／round_puff_fx_sm 圆烟滑行烟木屑，wood_splinter_jump 无生成点诚实缺口）＋木屑 2 注册并入。三写作子代理＋独立复审修正 P1×5（三页「快照无 languages」失实陈述纠正为 .po 实核；「狼女／拳套」改伍迪驼鹿形态／骑枪）＋P2×6（attune 清除锚点、拼接计数口径、sharkboi 消费行、浮标前缀枚举、绿花条目锚拆分）。9 文件 validate --check-indexes 全过、strict 0 错 0 警、record 6 页 0 漂移、diff --check 干净；两叶级索引计数同步。未运行游戏、未重建缓存、未提交推送。剩余 242 名续批（批 4＝船体侵蚀＋船件损坏＋船炮）。
- [x] 2026-11-01 通用表 FX 扩库批 2 完成（36 族 178→40 族 237）：幽灵灵药 36 注册（温蒂／阿比盖尔内容勘误——非伯尼，ghostlyelixir_slowregen_fx complex 页：buff 链 attach 父挂＋drip 15 秒节流＋四路清理，玩家 shadow/lunar 4 名无生成点）＋幽灵花灵体拆页（ghostflower_spirit1_fx，与灵药机制无关）＋武神之魂与战歌 14 注册（wathgrithr_spirit：击杀英灵／常驻战歌三段／歌手侧音符／瞬时引述，electric 无生成点）＋旺达怀表 7 注册（wanda_attack_pocketwatch_old_fx：年龄档×暗影武器选名、不老表治疗、第二次机会表复活倒流）。三写作子代理＋独立复审修正 P1×2（玩家 drip build；drinkelixir 仅前 4 帧 busy）＋P2×7。7 文件 validate 全过、strict 0 错 0 警、3 段 Lua 探针通过、record 4 页 0 漂移、diff --check 干净；fx-table-index 达 16KiB 上限拆出 fx-table-world-index（环境与物件 23 族 104），父索引两行登记。未运行游戏、未重建缓存、未提交推送。剩余 267 名续批（批 3＝相位认同＋海面水花＋水面杂项＋木屑）。
- [x] 2026-11-01 通用表 FX 扩库批 1 完成（覆盖台账 [fx-coverage-ledger-20261101](docs/reports/w05/fx-coverage-ledger-20261101.md)，32 族 163→36 族 178）：万圣夜落叶堆六注册（leafpile_spawn_fx 族页，放置显形／非即时生长／瓦片失效·敲碎·清扫三碎裂路径，红黄 build 拼名，休眠门仅盖放置与碎裂）＋万圣夜火焰绒·冷焰绒·月绒 7 注册（halloween_firepuff_1 族页，fuel.ontaken＋campfire 门、blueflame 分冷热、fxdata 符号跟随、月药水与月玻璃再生两月绒链）＋南瓜雕刻与雪球碎裂两注册各一页。三写作子代理＋独立复审：P1×2（生长路径无休眠门；月绒失败落点在目标非使用者）与 P2×5 全部修复。7 文件 validate --check-indexes 全过、strict 链接 0 错 0 警、5 段 Lua 探针通过、record 4 页 check 0 漂移、检索命中、diff --check 干净；fx-table-index 与父索引同步、leafpilemanager 补真实回链。未运行游戏、未重建缓存、未提交推送。剩余 326 名续批（批 2＝伯尼灵药 38＋战歌 14）。
- [x] 既有 FX 新版复查已并入 build 26335394 版本维护，不再单独认领：`willow_shadow_flame` 由更新报告 P-11 与批次-1 沿用记录覆盖（页面第 79 行引用 P-11、第 97 行沿用记录，`dst_build`／`updated` 与报告一致；第 92 行旧审阅保留 26230496 是版本维护要求，不是未复查）。其他旧页按 `maintain-version` 的旧新快照 diff／哈希流程处理：无 diff 的页不单独复查，无变化不刷新日期／版本；不再把整类 FX 另设 backlog。

- [x] 2026-10-31 通用表 FX 覆盖批 E～G 完成（本会话，7 页 51 注册，25 族 112→32 族 163）：E 形态变身（werebeaver_transform_fx 等 8 名＋鹅水羽 weregoose_splash 等 10 名，接住抛射物烟全玩家复用与父挂取位即解挂两机制）；F 开包（bundle_unwrap 等 17 名，bundle.lua 按 inventoryimage 动态拼名、quagmire 两名仅依赖声明）＋年龄转换（oldager_* 6 名，healthdelta 阈值换档与静默分支）；G 弹弓 AOE（slingshot_*_aoe_fx 4 名快照内无生成点诚实缺口＋专属工厂 slingshot_aoe_fx 五色着色实况）＋震地尘土（groundpound_fx 等 2 名，环点门禁与 fxRings）＋蜘蛛治疗（spider_heal_fx 等 4 名，护士蜘蛛群疗与治疗黏团双链）。回补 possessedaxe／groundpounder／slingshotammo_hitfx_rock 真实关系链接。11 文件 validate --check-indexes、10 页 293 引用 strict 零错警、7 段 Lua 解析、章节读取与检索词 grep 命中（缓存过期走 grep 注明）、record 7 页 0 漂移、diff --check 通过。同日评审修正 7 项：震地示例补服务端／combat 前提与显式 UseRingMode 并声明该调用含伤害破坏结算；弹弓元素注册改为"未找到原版生成调用"措辞、撤回不可用推断、依赖声明行改口；蜘蛛页按 healer 组件链（治疗量 0＋SetOnHealFn）与治疗者自身 combat.target 过滤重写并补 recipes.lua 进 sources；旺达页补非静默逐档过渡与 nomorph／鬼魂／死亡提前返回；开包页红包六名复用 redpouch／yotr 自有资源、17 变体 15 烟名、烧焦分支、暴食官名并补 lavaarena_abigail_flower 进 sources；鹅页撤回同帧互斥、带声水花 5 注册、run 两处帧；变身烟摘要鹿形态与 5 名；父索引改 32 族 163 注册。复验 8 页 198 引用 strict 零错警、43 组基线一致、7 段 Lua 通过。二轮评审修正 5 项：震地纯表现示例补 `platformPushingRings = 0`（默认 2 会推船）；旺达逐档陷阱改 old→normal 落 younger 前后两枚、触发链补 normal 档与无档位记录共用阈值直选分支；开包烧焦边界撤回"不播 packaged 声"（声音判断在 burnt 分支之外照播）并明确各注册实际 bank／build；弹弓颜色证据区间扩为 COLORS＋DISC_COLORS（L29-L41）与 SetColorType 全函数（L116-L136）。复验 8 页 strict 零错警、43 组基线一致。未运行游戏、未重建缓存、未提交推送。

- [x] 2026-10-30 通用表 FX 连续批 A～D 完成（本会话，12 页 23 注册，13 族 89→25 族 112）：A 溅落（ink_splash／bile_splash／waterballoon_splash，投射物 onhit 两时序与加湿分流）；B 气球糖霜（balloon_pop_body＋head 装备槽选名／spat_splat_fx＋4 变体 pinnable 默认列表／icing_splat_fx＋4 变体无生成点）；C 烟团暗影（small_puff DoChangePrefab helper／charlie_snap＋solid 呈现链开场／dirt_puff dig_fx 字段，并回补 sand_puff 链接）；D 烟雾采集（maxwell_smoke talker 两时序／shovel_dirt 挖掘帧／mining_fx＋ice 无生成点）。12 页＋2 索引 validate --check-indexes、strict 累计零错警、Lua probe 全过、检索与章节读取命中，record 12 页 0 漂移。诚实缺口：spat_splat_fx／icing 五名／mining 两名快照内无生成点。未运行游戏、未重建缓存、未提交推送。

- [x] 2026-10-30 通用表 FX 第三小批完成（本会话）：shadow_shield1.md（shadow_shield1～6，骨甲 PickShield 主盾冷却选名＋父挂持有者）＋sanity_raise.md（sanity_raise／lower，影柱计时／梦魇岩三态／破损祭坛／演示场景四消费者，名称不代表理智增减）＋lightning_rod_fx.md（避雷针 dozap 三触发：雷击充电／日终／取电，zap 声在本体），共 3 页 9 注册；fx-table-index 增至 89 注册。4 文件 validate --check-indexes、59 引用 strict 零错警（修 SGnightmarerock 路径层级与 8 处锚点边界）、3 段 Lua、检索与章节读取通过，record 3 页 0 漂移；sparks 验证记录残留"战斗长矛"确认已清零。未运行游戏、未重建缓存、未提交推送。

- [x] 2026-10-30 外围工厂 sparks＋weaponsparks 批完成（本会话）：sparks.md（sparks／electrichitsparks／electrichitsparks_electricimmune，独立工厂：随机帧门禁、AlignToTarget 定位＋目标闪白、灯光与动画双条件回收，集中入口 SpawnElectricHitSparks 七调用点核验）与 weaponsparks.md（weaponsparks／_piercing／_thrusting／_bounce，熔炉血统：flip 双类型、穿刺父挂重算与 3 帧镜像背面段、master_postinit 在 Forge 事件数据包——快照内无生成点诚实缺口）。4 文件格式／导航、84 引用 strict 零错警、2 段 Lua、检索与章节读取通过，record 2 页 check 全一致；electricchargedfx 两处"Wiki 待建"销项并微剪回 8KiB 简单档。未运行游戏、未重建缓存、未提交推送。外围其余候选仍未认领。

- [x] 2026-10-30 通用表 FX 第二批（G06～G10）完成（本会话）：五篇同族页 44 注册（shock_fx 6／planar_hit_fx 2／slingshotammo_hitfx_rock 18／lunarflame_puff_small 12／wx78_shield_full 6），fx-table-index 增至 80 注册，四页真实关系补链；11 文件格式／导航、237 引用 strict 零错警、5 段 Lua、检索与章节读取通过，record 5 页 0 漂移。未运行游戏、未重建缓存、未提交推送；外围筛选未认领。

- [x] 2026-10-30 通用表 FX 首批（P0＋G01～G05）完成（本会话）：机制页 generic-fx-factory＋五篇同族页 36 注册，fx-table-index 挂入 Prefab 索引，floater／boatleak 补链；11 文件格式／导航、235 引用 strict 零错警、7 段 Lua、检索与章节读取通过，record 6 页 0 漂移。未运行游戏、未重建缓存、未提交推送；G06～G10 与外围筛选未认领。

- [x] 2026-10-30 源码引用与章节读取检视修复完成：工具覆盖 .po、披露编码／跳过统计、统一物理行和 anchor 归一化；Skill 仅澄清名称证据 sources 口径，保留现有 .po 依赖，修两处组件回链及 5 页共 7 处 .po 空白终点。948 页／20,525 引用 strict／5,294 来源比较通过；245 项回归为 244 通过＋1 跨盘权限受限跳过，不称全绿。未重建查询缓存、未 record、未提交推送；见 docs/reports/w05/source-citation-review-20261030.md。

- [x] 2026-10-30 剩余 FX 覆盖盘点完成（本会话）：通用表实查 504 名（453 静态＋51 动态），30 名仅有文本提及、474 名无文本信号，无 Prefab 元数据入口；外围 380 文件只作候选，不当作缺页数。交付逐注册／外围 CSV 与 10 族 80 名候选清单，集合、来源行、现页路径与报告链接核对通过；不改知识页／raw／工具／Skill／基线／查询缓存，未提交推送。见 docs/reports/w05/fx-remaining-coverage-20261030.md。

- [x] 2026-10-30 通用表 FX 首批完成（本会话，P0＋G01～G05）：generic-fx-factory 机制页（fx.lua 加载器公共协议）＋五篇同族页覆盖 36 注册（温泉蒸汽 9／水花 5／烟团 9／树叶 8／生成烟雾 5），fx-table-index 挂入 Prefab 索引，floater／boatleak 补真实链接；11 文件 validate --check-indexes、235 引用 strict 零错警、标签与 Lua 示例通过，record 6 页 check 全一致，检索问法与章节读取命中。未运行游戏、未重建查询缓存、未提交推送。G06～G10 与外围筛选仍未认领。同日复审修正：六页 sources 补全（数据表 fx.lua、机制页 4 个引用文件、splash 13 个消费者）并刷新基线，机制页 description 函数约束／示例注册方式／transform 分量／父挂随宿主语义、烟团洞穴条件归位、鸦年华名称与顺序修正，复验 179 引用 strict 零错警＋record check 0 漂移。

- [x] 2026-10-30 通用表 FX 第二批 G06～G10 完成（本会话）：shock_fx／planar_hit_fx／slingshotammo_hitfx_rock／lunarflame_puff_small／wx78_shield_full 五篇同族页覆盖 44 注册（电击 6／位面 2／弹弓 18／月焰烟团 12／WX78 护盾 6），复用 generic-fx-factory 协议；fx-table-index 增至 80 注册两组，electrocute_fx／planarentity／wx78_shield／warg_mutated_breath_fx 四页真实关系补链。11 文件 validate --check-indexes、9 页 237 引用 strict 零错警、标签与 Lua 示例通过，record 5 页 check 全一致，检索问法与章节读取命中。诚实缺口入页：hitfx_gunpowder 仅暴击时抑制、weregoose 判定注释兜底、wx78 无 empty 常驻注册、月焰烟团单调用点。未运行游戏、未重建查询缓存、未提交推送。外围 380 文件筛选仍未认领。同日复审修正 7 项：火药抑制归位暴击分支、purebrilliance 光照补遗、shock 页 sources 补 electrocute_fx.lua 并刷新基线、月焰页撤回火墙表述并补 WillMutate 门禁、位面命中改组件条件表述、护盾重播补充能源前提、索引尚未建页节清理；复验 6 文件格式／导航、104 引用 strict 零错警、record＋check 0 漂移。

- [x] 2026-10-30 专属 FX 75 页产物分组完成：36 页保留单主线、33 页保留变体表、6 页增加产物小节，共 19 个可独立读取入口；保留注册总表、原 52 条关系与 Lua 示例，三页补必要前提。6 页格式／导航、277 引用 strict、19 次标题原文 read、52 组来源基线通过；未运行游戏、未重建缓存、未提交或推送。清单与边界见 docs/reports/w05/fx-product-grouping-20261030.md。

- [x] 2026-10-30 本会话 FX 75 页全文复核与修补完成：逐篇读取正文与工厂，修复接入前提、清理、真实数值、客户端推断和译名，补齐 13 页 sources 及瘴气变体别名，重写 fx-index 75 条完整需求摘要；77 文件格式／导航、1,880 引用 strict、48 段 Lua 语法、4 次标题原文 read、75 页 431 组基线全部通过。保留其他 Agent 增量，raw 只读，未重建查询缓存、未运行游戏、未提交或推送；逐页记录见 docs/reports/w05/fx-full-review-20261030.md。

- [x] 2026-10-30 Prefab FX 全量批完成（本会话）：11 批串行子代理建成 75 篇专属 FX 工厂页（census 纠偏后按实际注册名落名，动态拼名／多注册族并页收别名，别名标识符 168 个），逐批主代理四件套验收＋源码锚点内容抽查；新建 fx-index 叶级索引（六主题组）挂入 Prefab 索引，14 个组件页按真实关系回链；record 75 页全库 0 漂移，整库 validate 948 页 0 issue、strict 20,404 处引用 0 错 0 警，问法抽验命中。事件数据包生成、无运行时生成点等诚实缺口入页；未运行游戏、未解包动画资源、未提交或推送。

- [x] 2026-10-30 术语与 FX 标题收口（本会话）：beaverness 区分 player_classified 与 currentwereness 网络变量，groundcreep 明确同步实体的淡入调用条件；七篇手杖 FX 使用中文效果描述＋注册名并声明非官方译名，索引同步。10 文件格式／导航、222 处 Lua 引用 strict 零错警，七组标题／H1／索引一致、两个章节读取完整；只刷新新增来源的 beaverness 基线，9 页 41 组来源一致。Skill 窄修既有命名规则／模板，检索验收用 Markdown／grep，不重建缓存，未提交或推送。

- [x] 2026-10-30 Prefab 第四批复审修正（本会话）：四篇持续霜气页撤回“循环播放”推断，改为 pre→loop 队列描述，未显式传循环参数、实际是否重复未核验，实体驻留改由宿主清理协议支撑；五页删通用“注册加载”行、蝙蝠页补 netvars.lua 进 sources。工具箱 validate_source_links 明确编码契约（latin-1 回退＋物理 CR／LF 分行，raw 未改，链接测试 25 项通过）后，7 文件 163 处引用 strict 0 错 0 警，validate_page --check-indexes、标签一致性与全库基线复验通过。未运行游戏、未解包动画数据、未提交或推送。

- [x] 2026-10-29 Prefab 第四批完成（本会话）：建冰箱皮肤 5 篇 FX 页——冰冷坟墓开箱蝙蝠（宿主 1.3 秒节流生成联网代理、客户端据 variation 三变体另建本地动画与扇翅声，两级分别回收）与水晶／知更鸟蛋／陶瓷／维多利亚四篇同构持续霜气（开箱生成动画实体、关箱／移除经工厂安装的 Kill 以 pst 退场、皮肤符号覆盖）；官方皮肤译名经简中 .po 定向核验，标题效果词注明非官方译名。[[reference/prefabs/icebox]] 皮肤表与 [[reference/prefabs/index]] 回补真实链接并销项。7 文件格式／导航通过，173 处 Lua 引用 strict 除 easing.lua 非 UTF-8 工具读取错误 1 处（锚点已人工核对，缺编码回退移交工具箱）外零错警；record 新增 5＋主体 1，全库基线 0 漂移。未运行游戏、未解包动画资源、未重建缓存、未提交或推送。

- [x] 2026-10-29 Prefab 检视收口（本会话）：修冰箱动画 bank 误译、暖石第五档启灯与灯强渐变、7 篇 FX 的粒子参数／不适用说明及远古门禁／禁休眠；16 页 43 条陷阱区分误区／边界。query_wiki 复用章节解析归一标题原文、保留规范锚点及重名歧义，Skill 仅补两句原则；17 文件格式／导航、544 处 Lua 引用 strict 零错警、33 项检索回归及 12 项内存兼容／歧义检查通过，4 个真实章节原文 CLI 完整读取，122 组来源一致。缓存仍 62 页／现存 884 Markdown，search 为 stale，检索验收走 grep；可选 7～9 未做，sources 未改、不刷新基线，未提交或推送。

- [x] 2026-10-29 Prefab 第三批完成（本会话）：按新 Skill 建手杖 7 篇 FX 页，含远古 3 个编号／棋子 12 个形色变体，共覆盖 22 个注册；主体与索引补齐雪花、花瓣、彩纸、紫烟、闪光、影子／棋子拖尾检索词和互链。9 文件格式／导航通过，8 页 225 处 Lua 引用严格零错警；7 项效果词定位主体与产物，9 个章节分四组默认预算读取完整；record 新增 7／刷新主体 1，47 组来源一致。未运行游戏、未重建缓存、未提交或推送；冰箱 5 个 FX 继续待建。

- [x] 2026-10-29 Prefab 第二批完成（本会话）：按皮肤／强关联规则修整首批四页，新建 `cane`／`yellowstaff`／`opalstaff`／`stafflight`／`staffcoldlight` 五页；手杖七皮肤映射留本体，月台附属 staffcoldlightfx 作为冷星同页变体，法杖与产物互链并按对象区分重叠俗称。10 文件格式及导航通过，9 页 337 处 Lua 引用严格零错警，8 项章节读取完整；record 新增 5／刷新皮肤补核 3，9 页 82 组来源一致。未运行游戏、未独立评测、未重建查询缓存，保留其他执行者增量，未提交或推送。

- [x] 2026-10-29 Prefab 首批完成（本会话）：`heatrock`／`backpack`／`icebox`／`krampus` 四页，以 26230496 核验工厂、一层关联及简中名称，收录用户提供的“小偷”别名；Prefab 分类索引唯一登记并补 reference／根导航入口。7 文件格式与全库导航通过，153 处 Lua 源码引用严格零错警，四项章节读取完整；record 仅新增四页，check 37 组来源一致。游戏未运行，其他俗称待用户补充；保留其他执行者改动，未提交或推送。

| 顺序 | 工作 | 本轮交付与完成条件 |
|---|---|---|
| 1（已交付） | 收尾第六批：`moistureimmunity`、`freezable` | 实现／必要宿主／消费者、来源撤销、两段冻结与保存边界已审阅，登记／覆盖与章节读取通过；三个示例 Lua 5.1 解析通过，游戏未运行 |
| 2（已交付） | 拆分组件导航 | 归并五个主题索引，分类入口从 156 行降至 25 行，66 条原登记逐行保留，正文地址稳定；新批加入后 72 页唯一登记／父入口、主题浏览和名称直查通过，见 [本批报告](docs/reports/w05/component-targeting-cooldowns.md) |
| 3（已交付） | 第七批：工作效率与消耗倍率 | efficientuser／worker／workmultiplier 已发布，补消费、基础量、人物倍率与反弹顺序；三段示例语法／往返和两类需求章节读取通过，游戏未运行 |
| 4（已交付） | 首批原版 Prefab 参照：`heatrock`／`backpack`／`icebox`／`krampus` | 四页与分类入口已完成，按实际关系链接组件章节，四项定向章节读取完整；未运行游戏，不据此宣称通用查询收益或组合 demo 已验证 |
| 5 | 恢复场景骨架扩展 | 先补配方骨架实际需要的注册 API，再写 `new-recipe`，随后推进建筑及必要机制；不等待全部组件完成，不为准备骨架预建空页 |

前一百一十批及通用 C01～C14 已登记。C13～C14 四页与两张导航定向检查、全库登记通过，详细范围记入本次维护日志；此前批次报告保留原文。正文地址不迁移，Prefab 与配方队列继续保留，不将组件数量等同完整编码能力。

W01 契约、W02 提取基准、W03 提取／源码索引、W04 类型库试点及 W05 检索工具的完成范围移交 dst-toolbox 的 PLAN、references 和历史验收；组件格式／模板的后续改动由 dst-wiki-skill 维护。本仓不重复维护工具版本、正式 reviews 数量或工程测试清单。

<a id="当前优先组件-wiki-分批建设"></a>

## 后续队列：组件 Wiki 分批建设

| 批次 | 拟补组件 | 支撑的开发需求 |
|---|---|---|
| 1：三维与伤害（已交付） | `health`、`combat`、`hunger`、`sanity`、`eater` | 修改三维、回血、伤害与进食收益，补现有武器／护甲／食物／装备页的核心协作；源码／语法／结构与章节读取通过，游戏未运行 |
| 2：温度与环境（已交付） | `temperature`、`heater`、`sanityaura`，其余按源码盘点补充 | 暖石自身锁／读档、三个热源通道和精神光环；明确范围／单位／去重，源码／语法／结构与章节读取通过，游戏未运行 |
| 3：制作与交互（已交付） | `builder`、`workable`、`lootdropper`、`burnable`、`propagator` | 配方学习／缓冲、工作完成／掉落、燃烧计时／传播及清理；源码／语法／结构与章节检视通过，游戏未运行 |
| 4：成长与生物协作（已交付） | `pickable`、`growable`、`leader`、`follower`、`locomotor` | 采收再生／阶段成长、共享表隔离、忠诚／关系与实际移动分工；源码／语法／结构及章节检视通过，游戏未运行 |
| 5：计时与持续效果（已交付） | `timer`、`debuff`、`debuffable` | 具名计时／刷新／暂停、效果来源／撤销与同键、施加门禁／返回／存档重施；源码／语法／结构与章节检视通过，游戏未运行 |
| 6：潮湿与冰冻（已交付） | `moisture`、`moistureimmunity`、`freezable` | 湿度速率／强制干燥与多来源免湿、食物 Buff、冻结积累／两段计时、受击／SG／保存清理；源码／结构／语法与所选章节读取通过，游戏未运行 |
| 7：工作效率与消耗倍率（已交付） | `efficientuser`、`worker`、`workmultiplier` | 消费者语义、基础量选取、工作量来源／特殊回调、动态宿主及撤销保存；源码／结构／语法与章节读取通过，游戏未运行 |
| 8：睡眠与困倦（已交付） | `grogginess`、`sleeper`、`sleepingbag`、`sleepingbaguser` | 困倦来源／人物重建、生物测试／SG、床具占用与人物 tick／门禁消费差异；源码／结构／四例语法及章节读取通过，游戏未运行 |
| 9：治疗与修复（已交付） | `healer`、`maxhealer`、`repairable`、`repairer`、`sewing` | 普通治疗／惩罚恢复、材料／目标分工、分支优先级与标签、缝补单位及消费；源码／结构／五例语法和章节读取通过，游戏未运行 |
| 10：投射物与爆炸（已交付） | `complexprojectile`、`explosive`、`projectile` | 实体追踪与落点抛物线、两种发射归属、整堆爆伤／工作量和燃尽销毁；源码／结构／三例语法及章节读取通过，游戏未运行 |
| 11：定向技能与瞄准（已交付） | `aoecharging`、`aoespell`、`aoetargeting`、`cooldown`、`reticule`、`spellbookcooldowns` | 双端瞄准／蓄力与 SG 释放、地点施法授权／返回、单实体和人物多技能冷却；源码／结构／六例语法及章节读取通过，游戏未运行 |
| 12：科技与建造接入（已交付） | `craftingstation`、`deployhelper`、`placer`、`prototyper` | 科技来源／站点条目与限额、本地预览／辅助圈、多人和读档边界；四例源码／结构／语法及章节读取通过，配方注册 API 仍待补，游戏未运行 |
| 13：烹饪与干燥（已交付） | `cookable`、`cooker`、`dryable`、`dryingrack`、`stewer` | 食材／炉具／晾晒／料理分工、堆叠消费／新鲜度、天气与保存计时；五例源码／结构／语法及章节读取通过，料理注册 API 待补，游戏未运行 |
| 14：打包与拆包（已交付） | `bundlemaker`、`bundler`、`unwrappable` | 材料／人物／成品分工、两阶段完成／取消、记录恢复与预览；三例源码／结构／语法及章节读取通过，游戏未运行 |
| 15：传送与距离触发（已交付） | `entitytracker`、`playerprox`、`positionalwarp`、`teleportedoverride`、`teleporter` | 关联实体存档、近远模式、历史采样与紫法杖／虫洞分工；五例源码／结构／语法及章节读取通过，游戏未运行 |
| 16：农作与施肥基础（已交付） | `fertilizer`、`fertilizable`、`farmtiller`、`farmplanttendable` | 肥料供给／人物自肥、现代农田部署／旧参数、锄地与照料；四例源码／结构／语法及章节读取通过，游戏未运行 |
| 17：播种、土壤饮水与植物压力（已交付） | `farmplantable`、`farmsoildrinker`、`farmplantstress` | 种子／土堆消费、负率耗水与干湿统计、查看／成长检测及阶段累计；三例源码／结构／语法及章节读取通过，游戏未运行 |
| 18：世界土壤水分与养分（已交付） | `farming_manager` | 世界宿主／初始化、两类坐标、资格／限幅、阶段消费返回及必要存读；锄地补给例源码／结构／语法与章节读取通过，未覆盖全部管理器，游戏未运行 |
| 19：补水与浇水工具（已交付） | `wateryprotection`、`fillable`、`watersource` | 实体效果／单格土壤、接收方原地补水／换物、水源标签／份量与先消费；三例源码／结构／语法及章节读取通过，游戏未运行 |
| 20：枯萎保护、授粉与树木促进（已交付） | `witherable`、`pollinator`、`treegrowthsolution` | 自然／强制枯萎与保护、访问实体／回家／造花、催长回调与材料消费；三例源码／结构／语法及章节读取通过，游戏未运行 |
| 21：植物与肥料研究（已交付） | `plantresearchable`、`fertilizerresearchable`、`plantregistryupdater` | 研究键／阶段与人物事件、双端候选／RPC、本地档案与有限去重；三例源码／结构／语法及章节读取通过，游戏未运行 |
| 22：旧农场种植路径（已交付） | `crop`、`grower`、`plantable` | 连续进度／收获、设施时长与肥力轮次、种子产物数据；区分当前部署与注释、旧注册与正常制作，三例源码／结构／语法及章节读取通过，游戏未运行 |
| 23：产物累积与周边再生（已交付） | `harvestable`、`plantregrowth` | 分次增长／整批收获、回调与存读，种类时钟／桶调度、原始密度与落点；两例源码／结构／语法及章节读取通过，游戏未运行 |
| 24：养分显示（已交付） | `nutrients_visual_manager` | 非专服世界样式／登记、网络父覆盖层与本地子对象、玩家事件／HUD 及睡醒清理；排序例源码／结构／语法与章节读取通过，游戏未运行 |
| 25：巢穴与子体（已交付） | `childspawner`、`homeseeker` | 巢内／外出计数、生成接管／归巢、外部计时／存读与工蜂 Brain／动作分工；容量／召回两例源码／结构／语法与章节读取通过，游戏未运行 |
| 26：位置记忆与生成模式（已交付） | `knownlocations`、`spawner`、`periodicspawner` | 命名坐标存读、同体进出屋／补员、周期产物／密度与重试；三例源码／结构／语法及章节读取通过，游戏未运行 |
| 27：群体协作（已交付） | `herd`、`herdmember` | 软阈值／双向登记、中心资格／坐标、群体保存／清理、自动建群／禁用与重连；两例源码／结构／语法和章节读取通过，游戏未运行 |
| 28：群体发情（已交付） | `mood` | 天数循环／季节校准、状态／回调、保存倍率及牛群／企鹅实际消费者；等待参数例源码／结构／语法及章节读取通过，游戏未运行 |
| 29：驯化与服从（已交付） | `domesticatable` | 进度／服从／正式驯化、锁存完成、公开与内部增量、衰减和嵌套鞍保存 |
| 30：坐骑（已交付） | `rideable` | 上鞍／上骑资格、鞍交接、骑手事件与被骑 tick；服从阈值例 |
| 31：骑手（已交付） | `rider` | 上下骑转换／事件、同步转发与保存；骑乘精神贡献成对增删例 |
| 32：鞍与卸鞍（已交付） | `saddler`、`unsaddler` | 参数／同步约束、移动战斗消费与标记／动作分工；速度和工具消费例 |
| 33：拴牛两端（已交付） | `hitchable`、`hitcher` | 双向引用／回调、Brain／动作、候选锁／空存读／宿主清理；释放／锁事件例 |
| 34：装扮与舔盐（已交付） | `groomer`、`saltlicker` | 用户关闭／占用桥接、盐计时暂停与延后耗用；范围／半耗例 |
| 35：刷毛两端（已交付） | `brush`、`brushable` | 工具空标记／消费与目标天数产出、门禁表达式、字段差异；半耗／积攒例 |
| 36：剃取目标（已交付） | `shaveable` | 奖品／准入与宿主降级／产量协作；一级蜘蛛巢保护例 |
| 37：毛发生长（已交付） | `beard` | 精确天数回调、分级剃毛、保温／库存与恢复；威尔逊保温例 |
| 38：牛皮肤（已交付） | `skinner_beefalo` | 五部位／事件／刷新、脱装门禁与牛铃恢复；服务端脱装事件例 |
| 39：宠物集合（已交付） | `petleash` | 普通／专用限额、预留、生成／接管／解除和嵌套保存；威尔逊容量例 |
| 40：宠物血条（已交付） | `pethealthbar` | 玩家双端字段／事件、绑定和显示上限、附加条与 HUD；显示刷新事件例 |
| 41：牛行为记录（已交付） | `beefalometrics` | 事件统计、最近参与者实体／userid 与存读；只读 userid 查询例 |
| 42：小宠物性格（已交付） | `crittertraits` | 四类积分／衰减、主导／锁定、动画完成与主人事件；正向 PLAYFUL 倍率例 |
| 43：巢穴装饰两端（已交付） | `bedazzler`、`bedazzlement` | 工具直接耗用、目标成长／Buff 周期与宿主清理；半耗／解除装饰事件例 |
| 44：实体激活（已交付） | `activatable` | 资格／inactive、回调／nil 兼容与事件、重复使用；原栅栏门执行者测试例 |
| 45：定向物品使用（已交付） | `useabletargeteditem` | 候选 OR／实际校验、开始停止顺序与宿主清理；命名牌服务端检查例 |
| 46：建筑持续交互（已交付） | `channelable` | 单人／多人、SG 与停止回调、准入和清理；原传送阵中止事件例 |
| 47：持续施法物品（已交付） | `channelcastable` | 使用者握手、技能动态挂载与卸装撤销；打火机停止事件例 |
| 48：人物持续施法（已交付） | `channelcaster` | 物品／无物品状态、移动来源与事件、SG 取消；精神贡献成对增删例 |
| 49：蓄力方向显示（已交付） | `chargingreticule` | 本地 HUD／输入、手柄旋转插值与清理；平滑配置例 |
| 50：物品双态翻转（已交付） | `toggleableitem` | 布尔回调、无保存／停止协议与动作 nil 返回；剃刀私有标记例 |
| 51：装备使用协议（已交付） | `useableequippeditem` | 使用者回调、启动拒绝／nil 兼容、动态挂载和宿主清理；遥控器停止例 |
| 52：技能书（已交付） | `book` | 正式／翻阅、成功回调与消费／精神／FX；睡前故事基础精神半耗例 |
| 53：人物读者（已交付） | `reader` | 正式／翻阅分派、倍率／后回调与双端路由；麦斯威尔倍率例 |
| 54：轻量阅读（已交付） | `simplebook` | 可见性／回调、无 reader 与动作返回；烹饪指南保留弹窗通知例 |
| 55：题字会话（已交付） | `writeable` | 写入者／长度、布局／作者／存读与清理；木牌保持距离例 |
| 56：乐器演奏（已交付） | `instrument` | 回调顺序、听众筛选、SG 与消费；排箫听众半径例 |
| 57：短距瞬移（已交付） | `blinkstaff` | 延迟落地／复查、即时消费与 SG 恢复；懒人魔杖声音配置例 |
| 58：回溯标记（已交付） | `recallmark` | 坐标／分片、复制存读和显示刷新；原怀表服务端重设标记例 |
| 59：远程传送（已交付） | `remoteteleporter` | 目的台距离选择、物品集合、耗能与 SG 恢复；原伞收集半径例 |
| 60：人物魔术工具（已交付） | `magician` | 取物暂存／归还、状态事件与读档收尾；原人物停止事件例 |
| 61：魔术工具物品（已交付） | `magiciantool` | 使用者握手、动态转换和回调／容器清理；原帽子停止事件例 |
| 62：旧式钥匙（已交付） | `key` | 类型标签、使用／取回通知、doer 缺口；探测杖插入通知例 |
| 63：旧式目标锁（已交付） | `lock` | 匹配／卡住、状态回调、钥匙暂存归还及 GUID 恢复；原底座校验事件例 |
| 64：克劳斯式钥匙（已交付） | `klaussackkey` | 真假注册、世界恢复通知、门的宿主扩展键；普通鹿角兼开暴食门例 |
| 65：克劳斯式目标锁（已交付） | `klaussacklock` | 成功／原因／消费三返回值、袋子与门分工；原袋子服务端玩家检查例 |
| 66：怀表拆解（已交付） | `pocketwatch_dismantler` | 资格与执行、完整配方过滤、拟态与接收方、实际删除顺序；保护回溯表例 |
| 67：机器开关（已交付） | `machine` | 开关／回调、候选门禁、短冷却和加载重放；原灭火器间隔例 |
| 68：工程节点（已交付） | `circuitnode` | 启用／发现、双向集合、宿主供电和加载重连；聚光灯断开事件例 |
| 69：电围栏连接（已交付） | `electricconnector` | SG 配对、共享场、双方标签／回调及 GUID 重建；寻找半径例 |
| 70：标记目标（已交付） | `markable` | 新增／取消、编号池／资格、彩带与赛程清理；每柱四标记例 |
| 71：标记代理（已交付） | `markable_proxy` | 目标转交、双方开关、绑定／清引用顺序；牛侧执行门禁例 |
| 72：地雷式触发（已交付） | `mine` | 搜索／阵营与回调、重置／停用／睡醒；原犬牙半径例 |
| 73：捕捉陷阱（已交付） | `trap` | SG 触发、有限产物重建、收获删除／饥饿／保存；普通陷阱范围例 |
| 74：诱饵物品（已交付） | `bait` | 与 Trap 组件双向关联、吃／偷／拾取、解除与保存分工；熟胡萝卜撤饵例 |
| 75：玩家诅咒（已交付） | `cursable` | 整堆施咒计数、强制收纳／挤物、解除和死亡调用；单枚净化例 |
| 76：诅咒物品（已交付） | `curseditem` | 寻人／追逐／绑定归还、拾取门禁／堆叠字段；重复拾取静默例 |
| 77：升级材料（已交付） | `upgrader` | 类型／单位值、角色标签与目标最终门禁；蜘蛛丝两点贡献例 |
| 78：升级目标（已交付） | `upgradeable` | 消费／回调顺序、阶段与清零、宿主成长／保存恢复；蜘蛛巢三点门槛例 |
| 79：电路模块（已交付） | `upgrademodule` | 槽位／类型、绑定／激活撤销、移出耐久；健康电路附加移动贡献例 |
| 80：模块宿主（已交付） | `upgrademoduleowner` | 多栏实体、真实／有效电量、前缀激活、返还／成本／嵌套保存；单格补电例 |
| 81：模块拆卸工具（已交付） | `upgrademoduleremover` | 空标记、检视／手持动作、人物 SG 与请求分工；结束自身检视例 |
| 82：装备套装（已交付） | `setbonus` | 固定头身匹配、逐件启停、库存穿卸与移除；原月亮植物甲成对增抗例 |
| 83：类别增伤（已交付） | `damagetypebonus` | 目标标签／回调、来源键、武器与人物叠加；玻璃刀额外阵营贡献例 |
| 84：类别抗性（已交付） | `damagetyperesist` | 攻击者／武器标签、剩余倍率、装备与自身消费；原月亮植物甲减伤贡献例 |
| 85：整次抵挡（已交付） | `resistance` | 装备门禁／早退与事件、骨甲冷却／燃料；白天禁抵挡例 |
| 86：爆炸抗性（已交付） | `explosiveresist` | 护甲前累积、延迟衰减与百分数存读；熊獾减慢衰减例 |
| 87：格挡与闪避（已交付） | `parryweapon`、`attackdodger` | 玩家 SG 招架重定向、条件或冷却闪避与动态宿主；盾牌角度／伍迪冷却例 |
| 88：定身与眩晕（已交付） | `pinnable`、`rooted`、`stunnable` | 玩家受困／解救、多来源移动束缚、龙蝇事件倒地；时长和恢复边界 |
| 89：电击与反伤（已交付） | `electricattacks`、`damagereflect` | 存在性消费／显式撤销、电击倍率、反伤预计算／返还；电羊与原盔甲例 |
| 90：阈值与伤害记录（已交付） | `healthtrigger`、`damagetracker`、`dpstracker` | 双向比例跨线、绝对事件量累计、净失血采样；龙蝇节点／掉鳞与威胁例 |
| 91：位面与范围副效果（已交付） | `planarentity`、`aoediminishingreturns` | 非线性普通伤害变换、FX 参照、旋转来源倍率与吸血／热量消费；模块成对贡献例 |
| 92：显示与同步（已登记） | `healthbar`、`healthsyncer` | 头顶条双端显示与 net_float；生命百分比／趋势同步与消费者；修正双端描述、移除 nil 回调与结论边界 |
| 93：天气防护（已交付） | `sheltered`、`rainimmunity`、`lightningblocker` | 庇护副本时序与骑乘门槛、免雨来源集合与只挡雨路径、避雷平方半径与 thunder |
| 94：分阶段施工（已交付） | `constructionsite`、`constructionbuilder` | 材料进度／回调／超量返还／存读不回绑；玩家容器开合与 SG 三段 |
| 95：垂钓（已交付） | `fishingrod`、`oceanfishingrod` | 池塘钓等待／脱杆／捕获；海钓 tackle／lure、收线与断线、客户端预测 |
| 96：划船与平台（已交付） | `oar`、`walkableplatform` | 划船方向与失败加湿；平台成员／IsFull／UID 存读／移除沉没 |
| 97：船体物理（已交付） | `boatphysics` | 推进／转向／阻力、锚与桅杆、碰撞沉没、存读与 hullhealth 分工 |
| 98：帆舵与补丁（已交付） | `mast`、`steeringwheel`、`boatpatch`、`boatdrag` | 帆力与收放帆、驾驶者绑定与监听残留、补丁消耗、阻力四因子与 crabking 未注册 |
| 99：船体损伤（已交付） | `hullhealth`、`boatring` | 碰撞到漏水与直伤、固定漏点与自降解；整船旋转与 bumper 命中 |
| 100：钓鱼目标链（已交付） | `fishable`、`oceanfishable`、`oceanfishinghook` | 鱼量与刷新、上钩与放生；体力挣扎与捕获距离；charm／interest 诱鱼 |
| 101：容器与建造数据（已交付） | `container_transform`、`constructionplans`、`constructionbuilderuidata` | RUMMAGE 前转换；计划物品替换站点；客户端施工容器／站点桥接 |
| 102：船体附属（已交付） | `boatrotator`、`boatringdata`、`hull` | 控件绑定与解绑残留；旋转数据与半径；船唇／跳板 |
| 103：溺水与浮具（已交付） | `drownable`、`flotationdevice` | 落水判定／冲岸／伤害；浮具标记与回调 |
| 104：渔网（已交付） | `fishingnet`、`fishingnetvisualizer` | 投网入口；表现与捕获执行、无网络实体 |
| 105：收纳扩展（已交付） | `shelf`、`counter`、`selfstacker` | 展示槽取物；被动计数；唤醒自动堆叠 |
| 106：雨棚与闷熄（已交付） | `raindome`、`raindomewatcher`、`smotherer` | 周期授予免雨；玩家雨棚事件；空动作标记 |
| 107：磁力与信标（已交付） | `boatmagnet`、`boatmagnetbeacon` | 磁力合力累加、配对与开关 |
| 108：吸湿物品（已交付） | `moistureabsorbersource`、`moistureabsorberuser` | 速率回调、持有者集合与雨天折算 |
| 109：防滑砂（已交付） | `nonslipgritsource`、`nonslipgrituser` | 来源注册与持有者防滑集合 |
| 110：漏点与残骸（已交付） | `boatleak`、`flotsamgenerator` | 漏点修复／动态漏点；世界残骸生成与密度开关 |
| C01：作祟与说话（已交付） | `hauntable`、`talker` | helper 实效与组合、动作门禁／冷却、Say／Chatter 与双端消息；原宿主接入例 |
| C02：物品湿度与温度（已交付） | `inventoryitemmoisture`、`inventoryitemtemperature` | 宿主绑定、环境目标、外部控制、睡醒／存读、原码／解码；燧石与吸湿器例 |
| C03：偏好与添料（已交付） | `foodaffinity`、`fuelmaster` | 最大匹配、负生命／精神基础值豁免、添料者倍率与消费边界；威尔逊／薇洛例 |
| C04：保鲜材料与堆肥（已交付） | `preservative`、`compostingbin` | 一次性恢复／目标分摊、先恢复后扣料、绿棕材料／周期与平方刷新、保存清理；盐比例与桶周期例 |
| C05：容器代理与安装物（已交付） | `container_proxy`、`containerinstallableitem` | 共享 master／先关闭后解绑、双端资格／目标转发／卸载顺序；魔术箱暂停恢复与猪皮弹速例 |
| C06：电源与接收者（已交付） | `battery`、`batteryuser` | 倍率／部分充电、接收成功后消费、物品与人物分派、区别持续电路；避雷针门禁与晨星补能上限例 |
| C07：实物与记录存储（已交付） | `inventoryitemholder`、`itemstore` | 单物品交入取回／存档重入、记录队列／恢复失败与迁移分工；肉类筛选与单条打捞例 |
| C08：食物记忆与供料者（已交付） | `foodmemory`、`fueler` | 先读倍率后记忆、最后一次进食后遗忘；无物品添火与供料实体删除边界；沃利／小乌鸦例 |
| C09：地皮工具与移除目标（已交付） | `terraformer`、`terraformerremoveable` | 地图门禁／下层与掉落、动作耗次、目标移除回调；草叉裸地例与高尔夫生命周期说明 |
| C10：鸟笼占用两侧（已交付） | `occupiable`、`occupier` | 门禁与直接占用、释放与嵌套存档、回调重放与动态重建；金丝雀新装门禁／半寿命例 |
| C11：火情检测与扇风（已交付） | `firedetector`、`fan` | 扫描／布防与SG雪球分工、单次／吟唱回调与耗次；只选火情与额外减潮湿例 |
| C12：鉴评与宰杀（已交付） | `appraisable`、`murderable` | 委托检查与接收方约定、动作事件／掉落与堆叠边界；牛玩偶和池鱼原版接入说明 |
| C13：光环调整与局部温度（已交付） | `sanityauraadjuster`、`temperatureoverrider` | 轮询集合／来源撤销、双端构造与最近温度来源；温蒂原版说明及眼球塔配置例 |
| C14：施法精神与恐吓（已交付） | `staffsanity`、`epicscare` | 施法者倍率／年龄重设、范围事件与接收脑；威尔逊结算包装与蜂后扩范围例 |
| C15：迁墓工具与目标（已交付） | `gravedigger`、`gravediggable` | 工具先执行／目标后拆除、记录与部署转换、提示开关与执行授权；原版骨灰坛说明及白天迁墓门禁例 |
| C16：接取与偷窃（已交付） | `catcher`、`thief` | 玩家 SG／投射物接取协议、提示与实际半径、监视清理；背包／容器掉落返回、拆堆与青蛙命中链 |
| C17：点火工具与定时生成（已交付） | `lighter`、`firebug` | LIGHT／Ignite／回调与燃料成本；计时门禁、睡眠启停及未定位构造宿主的边界 |
| C18：剃毛与驱散工具（已交付） | `shaver`、`repellent` | 空工具标记、目标／SG 前检与包裹预览；随从解除和附近清目标两阶段、永久随从例外与消费 |
| C19：装瓶与搜寻交互（已交付） | `bottler`、`searchable` | 委托回调／宿主消耗与产物、PICK 分流、事件与移除顺序、存读及组件卸载边界 |
| C20：纸张擦除两侧（已交付） | `papereraser`、`erasablepaper` | 工作台标签与委派、纸张产物与数量、输入消耗及生成／交付失败边界 |
| C21：旋转与近距离检查（已交付） | `fencerotator`、`closeinspector` | 方向委派、事件消费与显式角度；装备检查回调、LOOKAT 分派和动态技能挂载／卸装 |
| C22：配方教学与扫描（已交付） | `teacher`、`recipescanner` | 指定与反查配方、builder 门禁、解锁／事件／消费；蓝图通用工厂与读档恢复、全息宿主边界 |
| C23：地图记录与投递（已交付） | `maprecorder`、`mapdeliverable` | 玩家探索器及世界会话、地图教学和转存；地图选点／计时与宿主移动、取消和恢复 |
| C24：涂蜡材料与目标（已交付） | `wax`、`waxable` | 普通／喷雾动作匹配、目标回调与成功消费；作物替换、南瓜帽状态及雪人门禁存读 |
| C25：绘图工具与图样（已交付） | `drawingtool`、`drawable` | 附近素材选择、图像／背景优先序、变化回调与消费；小木牌重绘和宿主显示／存读 |
| C26：拆卸与爆破委派（已交付） | `portablestructure`、`poppable` | 动作准入与宿主打包／耐久交接；单次触发锁、气球效果及延迟清理 |
| C27：座椅与玩家附墨（已交付） | `sittable`、`inkable` | 占位事件、SITON／玩家 SG 与退出；附墨刷新／转移、视野门禁及独立 HUD 表现 |
| C28：称重工具与重量数据（已交付） | `itemweigher`、`weighable` | 类型匹配、称重回调／SG 播报；重量／范围比例、捕获者与形态转存；记录移除钩子疑点 |
| C29：调谐目标与玩家查询（已交付） | `attunable`、`attuner` | 目标成本与同类替换、在线／离线恢复；专属代理、双端查询与角色转移 |
| C30：物品关联与世界管理（已交付） | `linkeditem`、`linkeditemmanager` | userid 绑定、网络创建与装备限制；玩家重连／技能握手、世界登记和批处理 |
| C31：跳劈与突刺范围武器（已交付） | `aoeweapon_base`、`aoeweapon_leap`、`aoeweapon_lunge` | 共用命中分流、手持武器／临时伤害、选区与抛物、SG位移及外置初始化边界 |
| C32：滑脚与实体滑面（已交付） | `slipperyfeet`、`slipperyfeettarget` | 来源累计／衰减与SG跌倒、海冰观察／防滑、滑面几何与倍率、显式撤销 |
| C33：镶嵌物与插槽宿主（已交付） | `socketable`、`socketholder` | 双端一次建槽、名称品质与门禁、序列化删除／重建、回调与读档／释放及专用拆卸链 |
| C34：寒冷吐息与地面防滑池（已交付） | `frostybreather`、`nonslipgritpool` | 双端白气表现／环境温度与强制模式、地面几何和防滑消耗分流／延迟退场 |
| C35：组队攻击两侧（已交付） | `teamattacker`、`teamleader` | 成员命令和Brain协作、轮攻与整编覆盖、容量裁减／追击解散及字段与类型差异 |
| C36：震地与装备目标追踪（已交付） | `groundpounder`、`targettracker` | 两种震地模式／去重与余波任务、单目标累计暂停／克隆与宿主装备同步撤销 |
| C37：循环切档与用电负载（已交付） | `cyclable`、`powerload` | 动作门禁与直接调用、存读回调；负载报告与宿主重算、多电源和待机边界 |
| C38：自动铺地与扫描余额（已交付） | `autoterraformer`、`dataanalyzer` | 容器先挂／耗料补槽／同格重试；扫描ID余额、惰性查询、延迟结算与任务清理 |
| C39：旧抓取与召唤标记（已交付） | `grabbable`、`summoningitem` | 废弃接口仍存NET读取与标签拾取替代边界；空标记、动作及ghostlybond分工 |
| C40：惊吓与旧式衰减（已交付） | `spooked`、`decay` | 工作惊吓累计／概率、个人幻象与SG精神分工；旧式量值事件、线程和num源码疑点 |
| C41：冲撞与旧渔具图纸（已交付） | `tackler`、`tacklesketch` | SG驱动碰撞／践踏与退出；旧站点教学保留实现、当前teacher替代及图纸存读 |
| C42：伤害光环与效果载体（已交付） | `aura`、`spell` | 周期战斗筛选／计数事件及任务清理；目标与计时恢复、光莓效果载体存读及结束边界 |
| C43：旧晾晒与晾架集盐（已交付） | `dryer`、`dryingracksaltcollector` | 旧单物记录与当前dryingrack迁移、天气腐败时序；盐槽与图标／实物及宿主返还 |
| C44：背包消化与容器焚烧（已交付） | `digester`、`incinerator` | 主栏按名消费与周期监听；容器整堆销毁、事件归属及宿主保护和动画锁 |
| C45：堆肥标记与产物预算（已交付） | `forcecompostable`、`incrementalproducer` | 绿棕分类优先级与食物回退；待生产预算、查询副作用及Brain／SG执行 |
| C46：料理记录与节庆大餐（已交付） | `cookbookupdater`、`feasts` | 个人料理缓存／持久化及RPC；世界餐桌分组、成对进餐事件、消费和Buff加时 |
| C47：飞鸟吸引与世界生成（已交付） | `birdattractor`、`birdspawner` | 玩家三键加算与羽毛帽来源；自然排程／显式生成、地皮鸟种及废弃空接口 |
| C48：幸运物品与幸运来源（已交付） | `luckitem`、`luckuser` | 携带装备贡献／堆叠与刷新；来源叠加撤销、负幸运猎犬修正与概率分工 |
| 后续：剩余主题池 | 原扩展主题池剩 175 个候选、224 个专项候选、3 个职责待确认；见 [路线快照](docs/reports/w05/component-roadmap.md) | 农作／生物及物品机制扩展 → 玩家宿主／世界／网络／海上系统；角色／活动／区域／Boss 等专项按需求拆小批 |

### 后续阶段与安排原则

- [x] 2026-10-28 组件缺页持续建页批完成（本代理，2026-10-28）：覆盖清单 unbuilt 402 候选全部消化——398 页建成（批A-01～A-26，串行 26 子代理批＋主代理收口），vault_* 4 件已删源码记 removed-source，23 件 replica/helper/empty 维持 not-component-page；新增 activity/display/network-shard 三个主题索引并在组件索引挂组，叶级登记约 400 行，批外互链 40+ 对；A-16 收编并行执行者先行 6 页，A-16 起派发前在覆盖清单标 in-flight 防撞。覆盖清单 794 built／unbuilt 0。全库终检四件套全绿：validate_page --check-indexes 850 页、严格链接 850 页 20,983 处 0 错 0 警、标签一致性 0 缺陷、哈希基线 4,713 组全部一致。事故与恢复见 wiki/log.md（标签批量脚本缺陷→git 对账还原 80 页→重放有意改动→93 页 CRLF 规范化，复检全绿）。
- [x] 2026-10-29 en-dash 标签遗留清零＋复合链接漏括号修复（本代理）：23 页 711 个 en-dash 区间标签全部规范化为 ASCII 并数字同步到锚点（77 处错位归零）；顺带修复 94 页 895 片段复合证据链接漏闭括号（读者侧断裂 URL）与 4,082 处 #LN-LN 双写锚点折叠；严格链接口径 17,860 处 0 错 0 警，全库四件套复检全绿。过程含两轮脚本缺陷均当轮逆变换修复归零，明细见 wiki/log.md。改动未提交，待用户 amend 指令。
- [x] 第 6～15 批 41 个候选已交付，组件从 48 增至 89；下一阶段按下列主题池继续小批，真实任务必要依赖可提前插入，复杂组件独立小批。
- [x] 第 16 批农作与施肥基础四页完成；区分三养分部署、旧作物参数、人物自肥、动作耐久与阶段照料，不将其视为完整农作骨架。
- [x] 第 17 批三页完成；核验动作单颗拆分／消费、负率饮水／干湿统计、压力查看与成长结算分工及季节豁免边界。
- [x] 第 18 批 farming_manager 局部接口页完成；区分地块／世界坐标、养分／水分资格、缺养分返回及存读边界，完整锄地补给例已检查。
- [x] 第 19 批三页完成；核验实体／土壤范围、动作与直接调用、Fill 先消费及海水返回边界、来源份量和原壶回调。
- [x] 第 20 批三页完成；核验保护／复壮与强制入口、授粉严格阈值及造花宿主、催长入口与实际效果／消费返回边界。
- [x] 第 21 批三页完成；核验研究源事件与人物登记、研究／成长阶段映射、客户端候选、本地持久化与服务端有限去重；分类按植物源和人物宿主分别唯一登记。
- [x] 第 22 批三页完成；核验旧对象工厂／脚本清单与拆解记录、Reset／消费、时长乘数／存档 rate，以及现行现代部署与注释中的旧分支。
- [x] 第 23 批两页完成；核验收获／回调顺序、蜂箱事件与计时、蘑菇农场存读，种类时钟／重复登记、原始密度／落点及常青树规范名 Hook；示例保留宿主与世界门禁。
- [x] 第 24 批一页完成；核验非专服世界宿主、样式与登记、覆盖层等级同步、本地玩家视野／HUD、主机子对象睡醒与清理；排序包装保留原版视野切换，不改土壤数据。
- [x] 第 25 批两页完成；核验巢内／外出及应急池、容量补员、生成／接管／回巢顺序、外部计时与 GUID 重连；完整蜂箱加容量／工蜂召回例保留宿主门禁，未运行游戏。
- [x] 第 26 批三页完成；核验命名坐标引用／覆盖及延迟记录、单体住户／内外保存／外部补员、周期随机增量／密度／重试／回调返回／存读；三例保留当前计时与宿主边界。
- [x] 第 27 批两页完成；核验群体登记／回调／软阈值、中心资格／坐标广播、GUID 重连、卸载与实体删除、成员标签／延迟任务；合并引用一致性保留运行缺口，案例不提供搬群保证。
- [x] 第 28 批一页完成并给 herd 补关联入口；核验 cycles 扣天、季节校准／重入、两个开关、强制标志、保存归一化／恢复和牛群／企鹅消费者，参数例只延长后续等待。
- [x] 连续完成第 29～33 批七页；每批源码核验、格式／导航、来源指纹、完整 Lua 语法与章节读取通过，分别保留驯化／骑乘／标记／锁的能力边界，未运行游戏。
- [x] 记录第 29～33 批从源码审阅到检查／临时文件清理的墙钟时间，共 12 分 49 秒；统一文档收尾另计。该样本只含本轮七页，不能直接外推剩余复杂组件；明细见本轮报告。
- [x] 连续完成第 34～38 批七页；174 处源码引用 strict、七例完整 Lua 5.1 语法／章节读取与来源指纹通过，三主题各唯一登记；beard 来自原专项池，该轮结束时剩扩展 426／专项 235／职责待确认 3。
- [x] 第 34～38 批核心测时 11 分 28 秒，统一工程文档／报告收尾另计；共享上下文和提前读取的相关消费者可复用，不按七个独立冷启动样本外推，详见本轮报告。
- [x] 第 39～43 批六页交付；144 处源码引用 strict、六例完整 Lua 5.1 语法／章节读取、全部 140 个已建记录的页面集合与来源指纹核对通过；人物血条不等于宠物生命，行为记录不等于所有权。
- [x] 第 39～43 批核心测时 11 分 32 秒；统一工程收尾另计。crittertraits／bedazzlement／bedazzler 来自专项池，该轮结束时剩余扩展 423／专项 232／职责待确认 3，明细见本轮报告。
- [x] 第 44～46 批三页交付；82 处源码引用 strict、三例完整 Lua 5.1 语法／章节读取、全部 143 个已建记录的页面集合和来源指纹核对通过，保持 source-reviewed。
- [x] 第 44～46 批核心测时 7 分 12 秒，统一工程文档／日志／检查另计；该轮按用户要求在第 46 批结束，临时示例清理后收尾。
- [x] 第 47～51 批五页交付；106 处源码引用 strict、五例完整 Lua 5.1 语法／章节读取、全部 148 个已建记录的页面集合与来源指纹核对通过，保留动作／状态／宿主和本地视觉边界。
- [x] 第 47～51 批检查点墙钟跨度 13 分 32 秒，包含交叉准备／复用与临时文件清理，不是独立冷启动测时；统一工程收尾另计，详见 [对应报告](docs/reports/w05/component-channelcast-controls.md)。
- [x] 第 52～56 批五页交付；134 处源码引用 strict、五例完整 Lua 5.1 语法／章节读取、全部 153 个已建组件页面集合与来源指纹核对通过；读书、翻阅、轻量阅读、编辑会话及演奏返回各自明确边界。
- [x] 第 52～56 批检查点墙钟跨度 8 分 51 秒，跨批准备可复用，统一工程收尾另计；book／reader 来自原专项池，该轮结束时剩扩展 412／专项 230／职责待确认 3，详见 [对应报告](docs/reports/w05/component-reading-writing-music.md)。
- [x] 第 57～61 批五页交付；145 处源码引用 strict、五例完整 Lua 5.1 语法／章节读取、全部 158 个已建组件页面集合与来源指纹核对通过；保留接受／落地、标记／传送、物品集合／SG 恢复和魔术工具双向归还边界。
- [x] 第 57～61 批检查点跨度 10 分 39 秒，跨批准备复用，统一工程收尾另计；magician／magiciantool 来自原专项池，该轮结束时剩扩展 409／专项 228／职责待确认 3，详见 [本轮报告](docs/reports/w05/component-teleport-magic-tools.md)。
- [x] 第 62～66 批五页交付；122 处源码引用 strict、五例完整 Lua 5.1 语法／章节读取、全部 163 个已建组件的页面集合与来源指纹核对通过；区分钥匙通知与暂存、真假钥匙与世界登记、失败与消费、资格与拆解执行。
- [x] 两项旧式钥匙来自原扩展池，克劳斯两项和拆表工具来自原专项池；剩扩展 407／专项 225／职责待确认 3，详见 [本轮报告](docs/reports/w05/component-keys-locks-dismantling.md)。五批共享准备／合并检查，未逐批独立测时。
- [x] 第 67～71 批五页交付；147 处源码引用 strict、五例完整 Lua 5.1 语法／章节读取、全部 168 个已建组件的页面集合与来源指纹核对通过；区分开关与门禁、节点与供电、寻找与直接连线、标记与代理开关。
- [x] 五项均来自原扩展池，剩扩展 402／专项 225／职责待确认 3；详见 [本轮报告](docs/reports/w05/component-machines-circuits-marks.md)。共享准备／合并检查，未逐批独立测时。
- [x] 第 72～76 批五页交付；152 处源码引用 strict、五例完整 Lua 5.1 语法／章节读取、全部 173 个已建组件的页面集合与来源指纹核对通过；区分触发与宿主伤害、捕获与产物重建、关联与消费、诅咒计数与物品绑定。
- [x] 五项均来自原扩展池，剩扩展 397／专项 225／职责待确认 3；详见 [本轮报告](docs/reports/w05/component-traps-bait-curses.md)。共享准备／合并检查，未逐批独立测时。
- [x] 第 77～81 批五页交付；134 处源码引用 strict、五例完整 Lua 5.1 语法／章节读取、全部 178 个已建组件的页面集合与来源指纹核对通过；区分材料消费与回调、阶段与宿主成长、模块安装与激活、按件成本与工具／SG。
- [x] 五项均来自原扩展池，剩扩展 392／专项 225／职责待确认 3；详见 [本轮报告](docs/reports/w05/component-upgrades-circuit-modules.md)。共享准备／合并检查，未逐批独立测时。
- [x] 第 82～86 批五页交付；108 处源码引用 strict、五例完整 Lua 5.1 语法／章节读取、全部 183 个已建组件的页面集合与来源指纹核对通过；区分头身套装、目标增伤／来源抗性、库存整次抵挡与护甲前爆炸累积。
- [x] 五项均来自原扩展池，剩扩展 387／专项 225／职责待确认 3；详见 [本轮报告](docs/reports/w05/component-setbonus-damage-resistance.md)。共享准备／合并检查，未逐批独立测时。
- [x] 第 87～91 组共 12 页交付；429 处源码引用 strict、11 例完整 Lua 5.1 语法与 12 页章节读取、全部 195 个已建组件的页面集合／来源指纹核对通过。planarentity 以原版组装入口说明，无需强造完整生物 demo。
- [x] 本轮显式启动 5 个子代理，按 3＋2 两波调度，各只写所属页并执行局部检查；主代理审阅并统一登记、集成检查和文档收尾。12 项均来自原扩展池，剩扩展 375／专项 225／职责待确认 3；详见 [本轮报告](docs/reports/w05/component-parry-control-damage-tracking.md)。未测总 token 或逐批独立耗时。
- [x] 第 92 批两页已由另一 AI 登记；本对话的覆盖／索引合并保留其结果，未重写该批页面或代替其报告。
- [x] 通用 C01～C03 六页交付；217 处源码引用 strict、六个完整 Lua 5.1 语法／章节读取、本次六页与四张导航格式／链接通过。合并时 203 个已建源码指纹通过；全库登记仍有他方在建缺口，详见 [本轮报告](docs/reports/w05/component-common-foundations.md)。
- [x] 通用 C04～C06 六页交付；190 处源码引用 strict 零错误／警告，六个完整 Lua 语法与章节读取通过；六页及四张导航通过，全库 260 知识页／13 索引登记完整，252 个已建来源指纹和页面集合一致。详见 [本轮报告](docs/reports/w05/component-preservation-containers-batteries.md)，未运行游戏。
- [x] 通用 C07～C09 六页交付；196 处源码引用 strict 零错误／警告，五个完整 Lua 语法与六次章节读取通过；六页及四张导航通过，全库 266 知识页／13 索引登记完整，258 个来源指纹和页面集合一致。详见 [本轮报告](docs/reports/w05/component-storage-food-terrain.md)，未运行游戏。
- [x] 通用 C10～C12 六页交付；191 处源码引用 strict 零错误／警告，四个完整 Lua 语法与六次章节读取通过；六页及四张导航通过，全库 272 知识页／13 索引登记完整，264 个来源指纹和页面集合一致。详见 [本轮报告](docs/reports/w05/component-occupation-fire-appraisal.md)，未运行游戏。
- [x] rainimmunity／lightningblocker 已由另一 AI 交付；本轮确认其与其他既有页均有登记，246 个旧页的页面集合／来源指纹匹配，未重复编写或代其完成全库语义审校。
- [x] 通用 C16～C17 四页交付；两个子代理局部校验与清理后，主代理完成定向复核及登记。四页与四张导航、全库登记通过，136 处源码引用 strict 零错误／警告；3 个完整 Lua 语法通过，两次代表章节读取完整无截断且与正文一致，274 个来源指纹／页面集合一致。未运行游戏，不另写长报告。
- [x] 通用 C18～C19 四页交付；两个子代理完成局部校验与临时文件清理，主代理定向复核及统一登记。四页与四张导航、全库登记通过，109 处源码引用 strict 零错误／警告；3 个完整 Lua 语法通过，两次代表章节读取完整且与正文一致，278 个来源指纹／页面集合一致。未运行游戏，不另写长报告。
- [x] 通用 C20～C21 四页交付；两个子代理完成局部校验及临时文件清理，主代理复核并统一登记。四页与三张导航、全库登记通过，112 处源码引用 strict 零错误／警告；2 个完整 Lua 语法通过，两次代表章节读取完整且与正文一致，282 个来源指纹／页面集合一致。未运行游戏，不另写长报告。
- [x] 通用 C22～C23 四页交付；两个子代理完成局部校验与清理，主代理复核并修正蓝图示例读档入口后统一登记。四页与三张导航、全库登记通过，155 处源码引用 strict 零错误／警告；4 个完整 Lua 语法通过，两次代表章节读取完整且与正文一致，286 个来源指纹／页面集合一致。未运行游戏，不另写长报告。
- [x] 通用 C24～C25 四页交付；两个子代理完成局部校验与清理，主代理复核并修正图像覆盖 nil 回退的说明后统一登记。四页与两张导航、全库登记通过，146 处源码引用 strict 零错误／警告；3 个完整 Lua 语法通过，两次代表章节读取完整且与正文一致，290 个来源指纹／页面集合一致。abilities 索引 144 行，仍低于上限；未运行游戏，不另写长报告。
- [x] 通用 C26～C27 四页交付；两个子代理完成局部校验及临时文件清理，主代理复核并统一登记。四页与四张导航、全库登记通过，157 处源码引用 strict 零错误／警告；3 个完整 Lua 语法通过，两次代表章节读取完整且与正文一致，294 个来源指纹／页面集合一致。abilities 索引 151 行，后续新增前核对体积并按需拆分；未运行游戏，不另写长报告。
- [x] 通用 C28 两页交付；本轮不启用子代理，只完成称重工具与重量数据。两页与两张导航、全库登记通过，67 处源码引用 strict 零错误／警告；2 个完整 Lua 语法及两次完整章节读取通过，296 个来源指纹／页面集合一致。abilities 索引 158 行，继续新增前优先评估拆分；移除钩子源码疑点留页内，未运行游戏，本轮到此收尾。
- [x] 通用 C29～C30 四页交付；两个子代理局部检查并清理临时文件，主代理定向复核与统一登记。四页与三张导航、全库登记通过，156 处源码引用 strict 零错误／警告；2 个完整 Lua 语法、两次完整代表章节读取通过，300 个来源指纹／页面集合一致。固定 98 项交叉核对完成 4 项、剩余 94 项；未运行游戏，不另写长报告。
- [x] 通用 C31～C33 七页交付；三个子代理完成局部校验与临时文件清理，主代理修正企鹅冰面非持久化表述并统一登记。七页与三张导航、全库登记通过，310 处源码引用 strict 零错误／警告；2 个完整 Lua 语法通过，三次单独代表章节读取完整且与正文一致，307 个来源指纹／页面集合一致。固定 98 项完成 11 项、剩余 87 项；未运行游戏，不另写长报告。
- [x] 通用 C34～C36 六页交付；三个子代理局部校验后由主代理复核并统一登记，补明确震地两模式的船只去重边界。六页与四张导航、全库登记通过，296 处源码引用 strict 零错误／警告，三次单独代表章节读取完整且与正文一致，313 个来源指纹／页面集合一致。固定 98 项完成 17 项、剩余 81 项；本轮仅原版组装参照，无完整新 Lua 示例或临时文件，未运行游戏，不另写长报告。
- [x] 通用 C37～C39 六页交付；三个子代理完成局部检查，主代理复核旧接口／标记边界并统一登记。六页与六张导航、全库登记通过，224 处源码引用 strict 零错误／警告；1 个完整 Lua 语法通过，三次单独代表章节读取完整且与正文一致，319 个来源指纹／页面集合一致。固定 98 项完成 23 项、剩余 75 项；临时文件已清理，abilities 导航 160 行，未运行游戏，不另写长报告。
- [x] 通用 C40～C42 六页交付；主代理续接完成复核与统一登记。六页与六张导航、全库登记通过，252 处源码引用 strict 零错误／警告；沿用未改的1个完整Lua示例语法结果，三次代表章节及两次导航章节读取完整且与正文一致，325 个来源指纹／页面集合一致。物品交互27条原登记迁移无遗漏／重复，abilities从160降至132行，新item-actions为46行。固定98项完成29项、剩余69项；无遗留临时文件，未运行游戏，不另写长报告。
- [x] 通用C43～C45六页交付；三个子代理完成局部检查，主代理复核并统一登记。六页与四张导航、全库登记通过，224处源码引用strict零错误／警告；三次代表章节及两次导航读取完整且与正文一致，331个来源指纹／页面集合一致。五条原烹饪／干燥登记迁移无遗漏／重复，items为153行、新processing为38行。固定98项完成35项、剩余63项；本轮无完整新Lua示例和临时文件，未运行游戏，不另写长报告。
- [x] 通用 C46～C48 六页交付；三个子代理完成局部检查，主代理复核并统一登记。六页与五张导航、全库登记通过，237处源码引用strict零错误／警告；1个完整Lua示例语法通过，三次代表章节读取完整且与正文一致，337个来源指纹／页面集合一致。固定98项完成41项、剩余57项；临时文件已清理，未运行游戏，不另写长报告。
- [x] 2026-10-26 通用 C49 两页交付：formationleader／formationfollower 按 Steam buildid 26148036 核验环形坐标、多圈分层、入退队与 lightflier 宿主／Brain 接线，明确 active 不直接控制更新。两页及 world-creatures 导航格式／链接通过，严格源码引用 99 处零错警，显示标签 99 处与锚点一致；1 例完整 Lua 语法及两次代表章节读取通过，record 新增 2 页。本批均唯一登记；全局导航检查仅见他方在建页未登记，不代其修复或称全库通过。固定 98 项已登记 43、剩 55；临时文件已清理，未运行游戏。
- [x] 2026-10-26 通用 C50～C52 六页交付：三个子代理分别完成蛋孵化与藏身、随从身份记忆与点名、蝴蝶与食人花自然调度；主代理复核宿主消费者及失败／启停边界后统一登记。六页和两张导航格式／链接通过，导航登记检查通过，250 处源码引用 strict 零错警、250 处显示行号与锚点一致，三次代表章节读取完整；record 新增 6 页。固定 98 项已登记 49、剩 49；本轮无新增 Lua 示例或临时文件，未运行游戏，不改他方新增组件页及其互链目标。
- [x] 2026-10-26 通用 C53～C55 六页交付：三个子代理完成蜘蛛哨与小队引用、眼球草与关联对象、波次配额与海鱼群；主代理复核字段／监听清理、任务与配额、宿主生成分层后统一登记。六页及受影响旧页、两张导航格式／链接和导航登记检查通过，247 处源码引用 strict 零错警、247 处显示行号匹配锚点，三次代表章节读取完整；record 新增 6 页。三页 simple、三页 standard，固定 98 项已登记 55、剩 43；无新增 Lua 示例或临时文件，未运行游戏。
- [x] 2026-10-26 通用 C56～C58 六页交付：三个子代理完成蝙蝠代码边界与盘旋影子、产物脱落与距离采样、水陆状态与谨慎行走；主代理复核实际可用边界和宿主分工后统一登记。六页和四张导航格式／链接、导航登记检查通过，195 处源码引用 strict 零错警、195 处显示行号匹配锚点，三次代表章节读取完整；record 新增 6 页。四页 simple、两页 standard；固定 98 项已登记 61、剩 37，G07 全部 19 项已覆盖。无新增 Lua 示例或临时文件，未运行游戏。
- [x] 2026-10-26 通用 C59～C61 六页交付：三个子代理完成玩家跑速与地面通知、重物碰撞与重物接收目标、持续推动与外部速度来源；主代理补正跨文件移除边界后统一登记。六页及旧页链接、五张导航格式／链接和导航登记检查通过，246 处源码引用 strict 零错警、246 处显示行号匹配锚点，三次代表章节读取完整；record 新增 6 页。两页 simple、四页 standard；固定 98 项已登记 67、剩 31。items 导航 156 行，后续继续新增前按需评估局部拆分；无新增 Lua 示例或临时文件，未运行游戏。
- [x] 2026-10-26 通用 C62～C64 六页交付：三个子代理完成士兵指挥与船员、骑枪来源与使用者、登离船运动与容器形态选择；主代理复核并统一登记。六页及五张导航格式／链接和导航登记检查通过，260 处源码引用 strict 零错警、260 处显示行号匹配锚点，三次代表章节读取完整；record 新增 6 页，均为 standard。固定 98 项已登记 73、剩 25；items 导航 157 行，后续新增前按预算评估局部拆分。本轮无新增 Lua 示例或临时文件，未运行游戏。
- [x] 2026-10-26 通用 C65～C67 六页交付：三个子代理完成鼓舞与加权目标池、侧移朝向与卡住采样、投影淡出与代理身份记录；主代理复核并统一登记。六页及三张导航格式／链接和导航登记检查通过，189 处源码引用 strict 零错警、189 处显示行号匹配锚点，三次代表章节读取完整；record 新增 6 页，三页 simple、三页 standard。固定 98 项已登记 79、剩 19；本轮无新增 Lua 示例或临时文件，未运行游戏。
- [x] 2026-10-26 通用 C68～C70 六页交付：三个子代理完成目标拉伸与连续魔法成长、修理材料与目标门禁、分组符号覆盖与被动显示数据；主代理复核并统一登记。六页及三张导航格式／链接和导航登记检查通过，179 处源码引用 strict 零错警、179 处显示行号匹配锚点，三次代表章节读取完整；record 新增 6 页，四页 simple、两页 standard。固定 98 项已登记 85、剩 13，G09 的22项与G10的5项全部覆盖；items 导航160行，继续新增前需局部拆分。本轮无新增 Lua 示例或临时文件，未运行游戏。
- [x] 2026-10-27 通用 C71～C73 六页交付：三个子代理完成角色皮肤与衣柜会话、实体变换与装填标记、玫瑰检查目标与使用者；页面于10月26日按26148036核验，主代理复核并统一登记。六页及五张导航格式／链接和导航登记检查通过，257 处源码引用 strict 零错警、257 处显示行号匹配锚点，三次代表章节读取完整；record 新增 6 页，四页 standard、两页 simple。固定 98 项已登记 91、剩 7，G11全部覆盖；本批未向items新增条目，保持160行。无新增 Lua 示例或临时文件，未运行游戏。
- [x] 2026-10-27 通用 C74～C76 七页交付：三个子代理完成私人礼物、生成落物与旧研究接口，图鉴教学与POI，制作配额及宝藏标记；主代理复核并统一登记。七页及四张导航格式／链接和导航登记检查通过，219处源码引用strict零错警，139处数字行号标签匹配锚点，三次代表章节读取完整；record新增7页，三页standard、四页simple。固定98项逐项核对为98个已登记且实际存在的独立页面、剩0；组件扩库在此暂停，先提醒用户并转入索引重整。无新增Lua示例或临时文件，未运行游戏。
- [ ] 索引重整已完成，后续按任务继续组件建设：剩余扩展主题175个候选拆成3～6篇小批，先农作、生物与物品／战斗／交互，再玩家宿主、HUD、世界、分片及海上机制。专项224个与职责待确认3个按真实需求核验宿主后归组。
- [ ] replica 随客户端缺口处理，helper／empty 不强造组件教程；461 是待核验文件数，不是预定等长页面数。
- [ ] 涉及动作／Brain／SG、EntityScript、RPC、Light、资源或注册 API 时补实际主题，不把组件覆盖等同全部编码能力。

### 每批交付与验收

- [ ] 核验实现、必要消费者及宿主；页面回答选型、精选接口、条件／副作用、必要完整接入、生效／撤销及证据，保留未知和未覆盖范围。
- [ ] 使用所安装 dst-wiki-skill 的组件模板／格式；只有真实共性遗漏才调整模板，不自动审校整组件类型或更新 fork。
- [ ] 新页唯一叶级登记，必要互链及日期／日志同步；大分类按实际需求拆索引，不展开根全量清单。可选工具验证检索，缺 Python 时仍可直接维护 Markdown。
- [ ] 每批用 1～2 个新增／修改需求检查完整性、读取链与回退；准确 token 取不到时记录字符／字节代理，运行验证分别记录。
- [ ] 已有覆盖、部分覆盖和完全缺页任务分开记录；新增内容与 Skill 的持续评测交替推进，不将缺页回退能力当充分覆盖，也不把特定任务收益推为全库收益。
- [ ] 同步覆盖清单、受影响导航及设计中的能力边界，保留已有经验和历史。已交付批次检视分别见 [核心](docs/reports/w05/component-core.md)、[环境](docs/reports/w05/component-environment.md)、[制作](docs/reports/w05/component-crafting.md)、[成长](docs/reports/w05/component-growth-following.md)、[计时](docs/reports/w05/component-timed-effects.md)、[潮湿与冰冻](docs/reports/w05/component-moisture-freezing.md)、[工作量与使用消耗](docs/reports/w05/component-work-efficiency.md)、[睡眠与困倦](docs/reports/w05/component-sleep-drowsiness.md)、[治疗与修复](docs/reports/w05/component-healing-repair.md)、[投射物与爆炸](docs/reports/w05/component-projectiles-explosion.md)、[定向技能与冷却](docs/reports/w05/component-targeting-cooldowns.md)、[科技与建造接入](docs/reports/w05/component-building-tech.md)、[烹饪与干燥](docs/reports/w05/component-cooking-drying.md)、[打包与拆包](docs/reports/w05/component-bundling.md)、[传送与距离触发](docs/reports/w05/component-teleport-proximity.md)、[农作与施肥基础](docs/reports/w05/component-farming-basics.md)、[播种、饮水与压力](docs/reports/w05/component-farming-growth.md)、[世界土壤管理](docs/reports/w05/component-farming-manager.md)、[补水与浇水工具](docs/reports/w05/component-watering-tools.md)、[枯萎保护、授粉与树木促进](docs/reports/w05/component-plant-care.md)、[植物与肥料研究](docs/reports/w05/component-plant-research.md)、[旧农场种植路径](docs/reports/w05/component-legacy-farming.md)、[产物累积与周边再生](docs/reports/w05/component-production-regrowth.md)、[养分显示](docs/reports/w05/component-nutrients-visuals.md)、[巢穴与子体](docs/reports/w05/component-nests-home.md)、[位置记忆与生成模式](docs/reports/w05/component-location-spawning.md)、[群体协作](docs/reports/w05/component-herds.md)、[发情机制](docs/reports/w05/component-mood.md)、[驯化、骑乘与拴牛](docs/reports/w05/component-riding-hitching.md)、[装扮、刷剃与毛发生长](docs/reports/w05/component-grooming-hair.md)、[宠物、行为记录与巢穴装饰](docs/reports/w05/component-pets-decoration.md)。

## 组件与 Prefab 导航整理

采用稳定正文地址与按需主题导航，具体索引规则及大小上限仍由 dst-wiki-skill 维护。本节只记录本仓整理工作，不增加查询默认读取项。

- [x] 后续 FX 建页与回补（2026-10-29 完成：手杖 7 页与冰箱 5 页两组已交付，主体“Wiki 待建”关系全部回补）：成页后按实际注册名反查主体的“Wiki 待建”关系，补 Wiki 链接、同步索引／必要别名并逐项销项；不依赖会话上下文。以下按组记录进度，不强制一项一页，允许按实际内容同族合并。
  - [x] 手杖 `wiki/reference/prefabs/cane.md`：本会话完成 `cane_ancient_fx`、`cane_candy_fx`、`cane_chess_fx`、`cane_harlequin_fx`、`cane_rose_fx`、`cane_sharp_fx`、`cane_victorian_fx` 七页；包括全部 22 个注册，主体已补真实 Wiki 链接及可见效果词，拐杖糖雪花检索已验收。
  - [x] 冰箱 `wiki/reference/prefabs/icebox.md`：本会话完成 `icebox_coffin_bat_fx`、`icebox_crystal_fx`、`icebox_kitchen_fx`、`icebox_porcelain_fx`、`icebox_victorian_frost_fx` 五页（四种持续霜气工厂同构，分页保留各自注册名检索）；主体已补真实 Wiki 链接并销“FX 待建”注记。同页已有说明的 heatrocklight、backpack_swap_fx、staffcoldlightfx 不自动重复建页。
  - [ ] 非阻断：四篇持续霜气页正文重复较多；后续同类皮肤 FX 扩批可评估“同族页＋变体表”组织（机制正文一份、注册入变体表与别名），不为现状追溯重排。

- [x] Prefab FX 全量建页批（2026-10-30 完成）：`prefabs/*_fx*.lua` 专属工厂中未建页的 75 文件全部成页（census 纠偏后按实际 `return Prefab` 命名，动态拼名／多注册族并入同页并收全别名），11 批串行子代理＋逐批主代理验收；新建叶级索引 [[reference/prefabs/fx-index]] 挂入 Prefab 索引，14 个组件页按真实关系回链，record 与整库校验全绿。宿主文件内定义的 _fx 注册名（backpack_swap_fx、staffcoldlightfx、heatrocklight 等）按既有决定留宿主页承载，不重复建页。详见近期执行顺序与日志。

- [x] 已将原 66 条登记归并人物／物品／战斗／世界／交互技能五个主题索引；分类入口由 156 行／12,271 字节缩为 25 行／1,252 字节，本批新增后 72 篇组件各在主题叶级登记一次。
- [x] 保留 `reference/components/<组件名>.md` 正文地址，不按装备／食物／建筑搬动共享组件。主题索引不承载机制正文，跨主题关系用链接表达；已知名称／章节继续直达，不强制先读分类索引。
- [x] world 原 159 行索引已拆为制作采集／农作和生物协作／实体关联两张叶级索引，原登记逐行保留，分类入口直达两者；无旧章节入链，正文不迁移。导航共 14 篇，后续两个子代理按小批推进，减少重复源码读取与工程报告。
- [x] 既有查询／校验支持 topics 下 type:index；未发现旧组件索引章节入链，格式／链接、登记完整性与父入口通过；未改工具、Skill 或重建缓存。
- [x] reticule 名称直查跳过导航；未知组件名的“按住蓄力并松手攻击”经分类入口→交互技能主题→aoecharging 章节定位。记录实际文件／输出字节，不宣称独立模型发现或 token 收益。
- [ ] Prefab 随首篇有效内容增长再归组；角色／生物／物品／建筑／植物只是候选浏览维度，暂不建空分类或批量移动正文。交叉对象保留一份页面，真实内容积累后再判断物理子目录的价值。

## 非组件知识：覆盖方向与内容归属（新增）

源码知识还包括 Prefab、StateGraph、Brain／行为节点、UI、世界生成、公共 API 与配置／数据模块。当前正式 reference 只有组件和 modinfo 页；dst-wiki-skill 的 S03-A 已补组件／Prefab／通用机制三份模板与类型指引，新增 mechanism 归属及兼容检查，任务级生成检视仍待办。本节只记录内容方向，不重复其模板／规则／工具兼容工作项。

- [ ] 按真实模组需求检视并补注册／Hook、实体宿主和必要原版 Prefab；再随角色／生物、HUD、世界生成任务补 SG／行为、UI 和地图机制，不等待 461 个组件候选全部完成，也不阻挡已验证的组件批次。
- [ ] 机制定义／流程／数据消费者归 reference，跨文件造物与组装 demo 归 cookbook；具体 type／分类以完成核验后的 Skill 规范为准，未定或未成页不建导航占位。
- [ ] 按开发需求聚合主题：一个源码文件可支撑多个页面，一个页面可引用多个文件；生成内容只保留决策、接入、限制与依据所需部分，避免全量翻译源码或复制接口清单。
- [ ] 首批有效非组件页检查叶级登记、互链、版本和按章节读取；内容覆盖另作清单，保留现有组件队列，不把文件盘点当职责核验、把某个样例通过当整类覆盖。

## cookbook：通用任务骨架

已交付起步、可拾取物品、手部、身体护甲与头部五篇。未交付名称仅是计划，不登记为空页。组装上下文保存在 cookbook，机制与案例保存在主要 reference 页；依据见 [经验归属决策](docs/decisions/shared-and-mod-wiki.md)。

### 后续章节与推进顺序

| 批次 | 场景／交付状态或拟定文件 | 最小 demo 与需要补足的机制 |
|---|---|---|
| 1 | 新道具 [new-item.md](wiki/cookbook/new-item.md)（已交付） | 可生成、检查与拾取；注册／名称／原版动画和图标／网络分界与扩展章节；源码审阅，游戏未运行 |
| 1 | 装备 [equipment/index.md](wiki/cookbook/equipment/index.md)；[hand.md](wiki/cookbook/equipment/hand.md)／[body-armor.md](wiki/cookbook/equipment/body-armor.md)／[head.md](wiki/cookbook/equipment/head.md) 已交付 | 短导航＋按场景拆分；类别／槽位／能力／外观分开，当前含手持加速、身体护甲与帽子；后续顺序见下节 |
| 1 | 加配方 `new-recipe.md` | 材料、科技、过滤分类和产物名；已有产物可直接接入，角色专属配方／科技作为扩展，不复制造物代码 |
| 1 | 新建筑 `new-building.md` | 可放置、建造与拆除的对象；placer、配方和必要物理／资源上下文，功能容器另见容器骨架 |
| 2 | 新角色 `new-character.md` | 注册角色＋角色工厂、共同／服务端初始化、三维数值、初始物品、名字／台词与必需资源清单；详细讨论头部／身体装备（含护符／背包／护甲）省略穿戴图与角色显示策略，区分装备功能、头发／身体恢复及多槽位兼容；死亡／复活恢复放角色扩展，技能／HUD 分开按需接入 |
| 2 | 容器 `new-container.md`；装备背包独归 `equipment/backpack.md` | 地面容器与可拾取未装备容器接双端布局、筛选、开关与转移；装备背包的穿脱／overflow 在装备骨架维护，复用 container／inventoryitem，不双写完整 demo |
| 2 | 自定义组件与同步 `new-component.md` | 最小服务端组件、注册 replica、netvar／dirty 读取；附最小宿主和加载位置，恢复／卸载清理链接组件机制，不复制大等级系统 |
| 2 | 主动技能 `active-skill.md` | 一个输入到服务端执行的闭环，明确玩家／目标／参数检查与冷却；动作方案补 Action／ComponentAction／wilson 与 wilson_client 接入，详细 RPC／预测／SG 归机制页 |
| 2 | 状态 HUD 与按键 `status-hud.md` | 一个显示同步值的 widget，局部玩家／独立服务器边界、创建／重建／销毁与输入处理；请求服务端行为链接技能骨架，不重复整套 RPC |
| 3 | 新食物 `new-food.md` | 道具基础＋edible／perishable；锅料理、配方判定及进食 Buff 为扩展，制作配方与烹饪配方分清 |
| 3 | 新生物／随从 `new-creature.md` | 一个可生成的生物；根据是否需要行为补 Brain／SG，随从补 leader／follower 与主人丢失清理，复用装备／战斗机制 |
| 3 | 新植物 `new-plant.md` | 一种可种植、成长、采收的模式；growable／pickable／deployable 等先查原版，不根据模组文件名认定植物实现 |
| 3 | 效果对象 `new-effect.md` | 按真实需求先选 FX、投射物或附着 Buff 中一种；其他模式走机制扩展，确有不同组装上下文时再拆分 |
| 按需 | 世界生成 `worldgen.md` | 独立入口、一个 set piece／layout 的生成与运行时资源接入；地皮与地表／洞穴差异按需扩展，不能照搬 modmain API |

### 装备分类与交付顺序

采用 `wiki/cookbook/equipment/index.md` 统一短入口，具体需求可绕过索引直达；不建立统一长装备教程。类别按组装场景分，装备槽位与功能能力分别选择：头盔从头部骨架扩展 armor，护甲背包从背包骨架扩展减伤，避免组合页爆炸。现只建已交付文件，拟定名称不作为链接入口。

| 顺序 | 页面／状态 | 最小范围与验收重点 |
|---|---|---|
| 1 | [hand.md](wiki/cookbook/equipment/hand.md) 已交付 | 物品工厂、HANDS、地面／持握资源、穿脱与步行倍率；武器／工具／技能另按需读；源码／语法／资源／结构检查，游戏未运行 |
| 2 | [body-armor.md](wiki/cookbook/equipment/body-armor.md) 已交付 | BODY、armor 普通吸收／受击耐久、耗尽卸装／存档；穿戴图可选；修复／位面防御／套装作为扩展；源码／语法／资源／结构检查，游戏未运行 |
| 3 | [head.md](wiki/cookbook/equipment/head.md) 已交付 | HEAD、草帽资源、精神基础贡献与头发／帽子显示恢复，省图可选；头盔防御／护目镜能力与外观模式分别扩展；源码／语法／资源／结构检查，游戏未运行 |
| 4 | `backpack.md` 待执行 | 装备容器、双端布局、开关／overflow 与卸装，原版 BODY 与可选 BACK 分开；保鲜另读机制 |
| 5 | `clothing.md` 待执行 | 身体外观和穿戴属性、持续消耗；可选 BELLY 与附加 armor 分开核验 |
| 6 | `amulet.md` 待执行 | 佩戴效果／撤销与原版槽位；可选 NECK，具体护符技能与持续耗能按需扩展 |

- [ ] 后续逐类梳理实际模组模式，再核验原版、必要资源／语法、登记与读取；不复制模组资源／专属框架。观察依据见 [手部](docs/reports/w05/hand-equipment.md)、[身体](docs/reports/w05/body-armor.md)、[头部](docs/reports/w05/head-equipment.md)。
- [ ] 多装备栏作为可选接入，核验指定版本／配置的注册、网络／UI、自动分类、外观与容器消费者；仅有 NECK／BACK／BELLY 槽位不代表兼容。共同陷阱留 equippable／inventory／container 等主要页。
- [x] 起步页补全局代理、已注入名称、modimport／require／Prefab 与元信息／世界生成环境边界，两个 Skill 给简短指针；不要求已有代理项目机械添加 GLOBAL 前缀。
- [ ] 按具体任务补 PostInit、存档迁移、死亡／复活／角色重选、跨世界进度的相关机制。图标／动画／肖像／音效规范归对应 API／资源主题，骨架只留必要资源与加载上下文。
- [ ] 每份 demo 明示必要文件、资源、注册／加载、替换点、条件和验证；扩展只链接真实已建页。无法取得必要依据时报告缺口，不以片段冒充完整工程。

<a id="w05真实方案与检索闭环"></a>

## W05：真实方案与检索闭环

- [x] 新物品与斧头修改两项隔离任务已作源码／语法／结构及受控读取重放，见 [任务检视](docs/reports/w05/tasks.md)。同作者共享上下文不等于独立模型评测。
- [ ] 扩展至 6～10 个真实任务，检查知识覆盖、生成正确性和需要的游戏行为；安装／自主路由的流程验收归 Skill PLAN，不在此重复打勾。
- [ ] 保留 [随身暖石组合](docs/decisions/layered-index-and-retrieval.md#组合需求与章节检索)：区分物品／热源／人物 35℃、速率／净变化、主栏／背包／手持与叠加，验证持有者转移／清理／重载／死亡；当前仅设计审阅。
- [ ] 根据真实缺口增修案例和陷阱；一次警告／bug 不自动成为常见方案，不预建所有效果组合。
- [ ] 记录知识内容在真实任务中的源码回查量、章节读取与正确性。Top 30 选择／内容大小是否有效以实际任务判断；测量方法及流程由 Skill 维护，工程工具无收益证据则不增强。
- [ ] 游戏测试记录 build、环境、场景、预期／结果；未运行保持 source-reviewed，不以结构或词汇命中替代。

## W06：知识版本复查
- [x] 2026-10-28 26230496 新增组件建页 4 页完成（本代理）：sweeper／sweepable／leafpilemanager／itemmimicconfig 报告 C-1～C-4 转正——两子代理按新页派发模板建页自验，主代理复核纠正报告 C-1～C-6／R-1～R-3 转抄行号（新文件实为 5/31/218/27 行、handler 在 USEITEM/EQUIPPED 段、ShouldItemMimicBeRevealedFor :2331-2363 等），报告已勘误并注明以页面为准；validate_page --check-indexes 4/4、严格链接 122 处 0 错 0 警、标签 0 缺陷、Lua 示例 4 例通过、record 4、登记 item-actions 3 行＋world-management 新节、sweepable/leafpilemanager 互链。
- [x] 2026-10-27 版本更新 26148036→26230496 报告阶段完成（本代理）：快照已切换（旧 current 归档 raw/archive/build-26148036），哈希全检 869 处真变化／affected 350 页／sources_changed 2／删除 0；updates/build-26230496 报告五明细＋index 落地，文件覆盖 154/154、语义审校完成（待核验 1 条 P-16），updates/index 已登记。
- [x] 2026-10-27 版本更新知识页迁移完成（本代理）：350 页漂移队列全部处理——2026-10-28「原始哈希等价批量沿用-70」（三项检查全过、validate_page 70/70）＋「常规迁移-批A」组件域 38 页（机械 219＋review 46 人工、语义修订 5 页、validate_page 38/38、record 38）＋「常规迁移-批B」枢纽域 312 页（机械 1,144＋review 50 人工、标签同步 437、validate_page 312/312、严格链接 10,751 处 0 错 0 警、record 312 含 sources_changed 2 页重建）。收尾全库 check 2,996 组对比全部一致、affected 0、sources_changed 0，漂移队列清零。沿袭记录落 updates/build-26230496/index.md 的 Wiki 沿用记录节；全库改动未提交，待用户 amend 指令。

- [x] 2026-10-25 版本维护按规模拆分：单页入口与批量协议分文件，2 页及以上或整类／全库才追加批量候选／沿用／收尾；工具调用细节仍归校验入口与工具箱。单页不以全量报告、模板或全库阶段收尾为前置；本轮仅规则／工程说明整理，不执行知识页迁移。

- [x] 2026-10-25 校验规则按职责拆分：Skill 的 verify-wiki 精简为选择／常用命令／结果判断，工程专项独立为 verify-engineering；工具箱集中维护具体校验和哈希基线说明，README 不重复工具手册。原维护入口／锚点保留，普通查询不加载校验；三仓设计与计划同步，不改知识页或工具实现。

- [x] 2026-10-25 按用户约定取消查询时的新旧巡检：直接使用现有知识和快照，不在每次会话比较游戏／Wiki 版本或来源哈希；版本更新后由用户主动维护。Skill 入口、查询／检索示例、维护校验及三仓设计／计划已同步，哈希工具能力保留但不自动挂接查询。

- [x] 2026-10-25 迁移预览检视修复与记录澄清：工具增加引用区间外双侧有效上下文限制，拦截 A 删除／B 重现误映射；格式受限页不输出文本补丁，相对知识根路径已在本机 Windows、显式设置 DST_TEST_CODEX_BINARY 的进程中实际应用验证；默认命令跳过此项。共同沿用规则明确机器符号反查和页内短依据指针，三页补链至更新报告的 Wiki 沿用记录；试点统计统一为 90／28／15 的引用级分流，不推导页面免核验。工具回归与当前预览结果见工具箱 PLAN；仅补三页说明，未刷新未变来源基线或额外升版。

- [x] 2026-10-25 增量复用试点与跨仓固化：cyclable／harvestable／upgradeable／blinkstaff 四页完成，133 处原引用按引用级分为来源未变 90／相同代码块 28／需定向处理 15；其中 blinkstaff 的 14 处相同代码块所在页仍做了语义修订，32 个链接迁移；blinkstaff 修订同时纠正 updates C-05 的开始事件／落点复查边界，供后续页复用。格式 4/4、严格引用 136 处零错警，record 4，队列 157→153。Skill 固化报告优先、仅锚点迁移及增量收尾；工具箱提供只读迁移计划／补丁预览，工程测试由其 PLAN 维护。不以本轮比例推导实际 token 收益。

- [x] 2026-10-25 增补仅换行／编码差异的轻量沿用分支：沿用既有 line_ending_only 分类，不逐页重审语义，复用批次枢纽反查和机械引用检查；此类原始哈希已变，批末集中 record，区别于原始等价页不 record。当前快照全检 line_ending_only=0，无此类实际迁移任务；规则归 dst-wiki-skill。
- [x] 2026-10-25 原始哈希等价批量沿用完成：24 页所列来源均原始字节相同，本批锚点零错误／警告（全库 11,394 处引用的 32 错／768 警全部落在尚未升版页面，机械扫描不证明成因均为版本更新，随各页复查分批处理，存档 output/source-links-full-26148036.json）；枢纽反查 23 页无命中（同名近似以实际 diff 排除），linkeditem 命中 C-19（GetGrandOwner 回退中间持有者×弹弓换绑：至少装有一个部件且在无人持有容器内触发换绑时，从维持原绑定变为 userid=nil 解绑）转常规修订并把 inventoryitem.lua／slingshotmods.lua 纳入 sources＋单页 record；其余 23 页批量升版 dst_build／updated、统一记日志，不逐页 record、不重写未变基线。validate_page 24/24、严格链接 669 处 0 错 0 警。本检查点 280 个旧版本页全部落在漂移队列内；后续轮次仍按 maintain-version 候选合并（版本落后∪review-needed∪未登记∪关联页）筛候选，不恢复逐页人工检查；规则归 dst-wiki-skill 的版本维护分支。
- [x] 2026-10-24 首次真实跨版本验收（25643519→26148036）：setup_raw.ps1 归档旧快照并解压新 build，哈希基线全检真变化 1,040 对次（页面×文件对比，去重前；git 独立对账为 427 改／59 增／5 删个路径）／源码删除 0／仅行尾 0，受影响 318/345 页；git diff --no-index 独立对账与工具判定一致。建立 `wiki/updates/build-26148036/` 版本更新总结试点（分类总结＋318 页处理队列＋check 报告＋变更文件清单）；修复 beard／brushable 两页 SGBeefalo.lua 引用大小写（内容核验未做，留在队列）。工具侧 case_mismatch／受影响页队列／sources_changed 强化见工具箱 PLAN。
- [x] 2026-10-25 战斗/proc 批 32 页核验完成：结论修订 8 页（combat GetAttacked 拆分/planarentity planar 门槛/teamleader 死亡监听与驻守门槛/projectile Hit 早退/explosive stacksize 等）、锚点重定位 365+ 处，dst_build 全部升 26148036 并 record，队列 318→286。
- [x] 2026-10-25 饮食批 5 页＋vault/传送批 1 页（sanityaura）：eater 结论按 C-12 修订（SetCanEat* 弃用补提示、SetDiet 缓存陷阱补救改 ResetEdibleTagsCache、SetCanEatRawMeat 未弃用辨析），edible/foodaffinity/foodmemory/beefalometrics 仅锚点重定位与记录，sanityaura 经 vault→virtualroom 迁移核验不受影响（vault_sanity_adjuster 未变仍注册）；锚点重定位 40+ 处，dst_build 全部升 26148036 并 record（refreshed 6），队列 286→280。
- [x] 2026-10-25 源码未变分支代表页验证：burnable/cooldown/fuel 关键消费者交叉核验通过（explosive OnBurnt 新可选参不涉无参调用、rechargeable 变化不触及 cooldown、eater 变化与 fuel 无交集），fuel 顺手修正既有 log.lua 锚点偏移 2 处；三页 dst_build 升 26148036（不在队列不 record），其余旧 build 页按上方批量沿用分支执行。
- [x] 2026-10-25 动作交互片 8/9＋9/9（船只水上钓鱼 14 页＋零散使用升级 24 页）完成，动作交互域 9/9 收口：38 页全部仅锚点迁移、无结论修订、无新增 sources（s8 重定位 51 处，s9 逐页锚点全部落位）；增量复用试点 4 页（blinkstaff／cyclable／harvestable／upgradeable）抽查通过并入本批 record，upgradeable 顺带修正试点遗留的"已迁移未落锚"记录与结构行起点锚点；oceanfishingrod 超 16KB 预算，删与其他节重复表述精简至 16313 字节。validate_page 38/38、严格链接 1382 处 0 错 0 警，集中 record（刷新 34＋已录 4），队列 153→119 且本两片全部出队。
- [x] 2026-10-25 动作交互片 6/9＋7/9（动物骑乘 20 页＋施法使用 19 页）完成：语义命中较多——hauntable 命中 C-17（haunted 事件移入 onhaunt 成功分支，失败不再发事件，3 处结论改写）、rider 命中 C-34（骑乘伤害转交新增装备攻击者豁免 rider.lua:133-139）、spellcaster 5 处修订（新增"未装备直接对目标施法"USEITEM 接入节＋nomagiccast 门控陷阱＋CASTSPELL 禁长按＋onspellcast 事件＋staff 迁移说明）、channelable/channelcastable/channelcaster 补 START/STOP 类动作禁长按、aoespell 澄清 nomagiccast 门控不含 CASTAOE、lighter 补 USEITEM.weapon 分支接入与旧灭火回退存疑标注；shaveable/spooked/hauntable 6 处锚点错位或越界经严格链接检查拦截后人工修正（含 1 处超出文件行数的 ERROR）；validate_page 39/39、严格链接 1085 处 0 错 0 警（修正后），集中 record refreshed 39，队列 196→157。
- [x] 2026-10-25 动作交互片 4/9＋5/9（建造种植 22 页＋食物燃料 18 页）完成：constructionbuilder 2 处实质修订（stopconstruction 事件改 PushEventImmediate 同步派发、SGwilson constructing 态 onexit 加 stoppedconstruction guard）；dryer 证据跨文件迁移（staff.lua destroystructure → staff_common.lua:696-697）纳入 sources；healer 顺手修正旧页既有 HEALING_MED 锚点错位（旧 2013 实为 HEALING_SMALL）；其余 37 页纯锚点搬运。validate_page 40/40、严格链接 1174 处 0 错 0 警（wateryprotection 2 处尾行修正后），集中 record refreshed 40，队列 236→196。
- [x] 2026-10-25 动作交互片 2/9＋3/9（锁与钥匙 10 页＋书写阅读记录 14 页）完成：仅 pinnable 一处实质修订（SGwilson 受困三态退出新增 isstillpinned 复查并兜底 Unstick），其余 23 页纯锚点搬运；closeinspector 的 hats.lua 三处证据锚点初核错位（落 mask_charlie_boss 区块）经人工复核纠正并收紧 6 处空行起止锚点；validate_page 24/24、严格链接 738 处 0 错 0 警（纠正后），集中 record refreshed 24，队列 260→236。
- [x] 2026-10-25 动作交互片 1/9（容器与物品装载 20 页）完成：3 页实质修订（inventoryitem 命中 C-19 GetGrandOwner 回退与拾取燃烧 torch 放宽、grabbable 补燃烧例外、inventoryitemholder 补 gelblob_storage 持久化），其余 17 页纯锚点搬运；锚点重定位 120+ 处，集中 record refreshed 20，队列 280→260。先行产出 actions＋componentactions hunk 地图（output/actions-ca-hunk-map.md）供其余 8 片复用。
- [x] 2026-10-25 启动全局批（entityscript＋mods＋mainfunctions 枢纽）60 页完成：先行枢纽 diff 审阅产出 hunk 地图（output/global-hubs-hunk-map.md，13 hunk／11 语义，全部对应已核验条目 R-08／R-09／R-11，含行号偏移分段表）；语义命中 6 页——freezable（icestaff onattack_blue 随 P-112 迁 prefabs/staff_common.lua，纳入 sources）、inventoryitemmoisture（R-08：InheritWorldWetnessAtXZ/AtTarget 追加 CanPointHaveRain 门控，无雨区入水不再立即继承世界湿度）、lootdropper（FlingItem 新增 nofling 开关＋落点传送失败回退，默认行为不变）、moistureimmunity（无效实体早退 0/false＋回退分支边界）、teamattacker（penguin KeepTarget 死分支修复为 ORDERS.ATTACK，仅 ShouldRunAway 仍字符串比较）、mod-starter/modinfo/fields（R43 门控激活核验不影响元信息结论）；moisture 组件速率路径确认不受 R-08 影响。收口新增标签一致性扫描（严格链接只验锚点不验显示文本）抓出 41 处代理漏改的滞留行号标签＋2 处漏迁锚点（healthbar children 登记、mod-starter 读取代理行）＋walkableplatform 超预算精简，全部人工修正并记页内；两次子代理流中断（一次 0 页受损重派、一次页面全部落地后断于汇报，按"页内声称不作数"以机械验收确认完成）。validate_page 60/60、严格链接 2290 处 0 错 0 警，集中 record refreshed 60（manifest ok），队列 119→59。
- [x] 2026-10-25 tuning 数值批 22 页完成：先行 tuning.lua 枢纽审阅产出键级变更地图（output/tuning-hunk-map.md，10 hunk 全为键值增改：改值仅 5 键——WORM_BOSS_DAMAGE/SPINES 14→20 但配新键 PLAYERDAMAGEPERCENT=0.5 对玩家实际伤害 14→10、ROCKY_WALK_SPEED 2→2.7、ROCKY_GROW_RATE 40→4.8 天（快约 8.33 倍）、FUMAROLEHOE_HEAT_ON_USE −0.2→−0.8；新增 177 键全在 Rifts 8 尾块；0 删键；死键警示 FUMAROELWEAPON_HEAT_ON_USE 为官方拼写错误且全 scripts 零消费）。22 页中仅 firedetector 一处语义修订（NOTAGS 候选排除表新增 gestaltflame，firedetector.lua:1，发光火焰不再被灭火器扫描选中），其余 21 页纯行号迁移且无一引用 5 个改值键或死键；派发文本新增两条执行纪律（标签行号同步更新、复核记录不超页字节预算），两代理自验＋主代理三重机械复核（validate_page 22/22、严格链接 769 处 0 错 0 警、标签一致性 0 缺陷）；一次子代理流中断仍为"页面全部落地、汇报丢失"模式，按机械验收直接收口。集中 record refreshed 22，队列 59→37（余玩家侧 10／零散 27）。
- [x] 2026-10-25 玩家侧批（SGwilson＋player_common 等 8 文件枢纽）10 页完成：先行枢纽审阅产出 output/player-hubs-hunk-map.md（62 hunk／58 语义——SGwilson 41 断点全量偏移表；两处报告未覆盖的新发现：animqueueover 死代码转正即 crushcasting 清理修复 :20131-20144、construction 停止修复 :19944-19956；击退链 nointerrupt 门控＋superheavyarmor knockbackblocked、pinnable SetGooBuild＋SG 侧 Unstick、quickbufferedaction 第 14 帧副攻击键连发快进、wormdigestionsound netvar 迁 player_classified）。10 页中仅 aoediminishingreturns 一处语义修订——暗影战斧 DoLifeSteal 资格门由 not HasOneOfTags 改为 componentutil.IsLifeDrainable（lifedrainable 豁免／nolifedrainable 拒绝，battleaxe:298＋componentutil:185-188，componentutil.lua 纳入 sources），倍率与理智不缩放结论不变；其余 9 页纯锚点迁移（teleporter 经逐访问条件确认 vault_teleport 泛化与 JUMPIN 标签迁移不触及虫洞配对结论；fishingnet 钓网段两版一致；aoecharging 确认连发快进不涉蓄力链）。代理自纠两次偏移段误用并避开 spiderden 已删标签的过时示例。validate_page 10/10、严格链接 412 处 0 错 0 警、标签一致性 0 缺陷，集中 record refreshed 10，队列 37→27（余零散 27）。
- [x] 2026-10-25 零散批 27 页完成，280 页漂移队列清零（318→0）：按共享变更文件分 3 组×9 页（containers/perishable/componentutil 簇、player_common_extensions/network/spell 簇、playercontroller+map/pigman/staff 簇），共享文件 diff 组内复用；语义相关修订——teleportedoverride 与 staffsanity 的 staff.lua 证据整体迁 prefabs/staff_common.lua（六段/三段逐字一致，王杖新分支不改变 telestaff/orangestaff 行为，staff_common.lua 纳入两页 sources）、rechargeable 新增 LongUpdate=OnUpdate 别名按第九轮条目逐访问条件核验（本页无 LongUpdate 结论）、constructionsite 记 constructionbuilder stopconstruction 改 PushEventImmediate 的组件侧（SB-17，结论不变）、perishable 源码两处变化（酸雨 CanPointHaveAcidRain 门控、AddTime 移除 updatetask 门控）均不在页面记载面（已注明供后续引用）；地图勘误一处——player-hubs 偏移表 slip 状态段实测 +166 而非 +50（nonslipgritsource 发现并修正，穿过该段的 slipperyfeet/tackler/sleepingbaguser 均已经过严格链接机械验证不受影响）；game-modes 两处区间标签配单行锚点扩为区间锚点、periodicspawner 注释分界锚点端点收紧。执行层：流中断 4 次（第 1 组三次重派、第 2 组一次），其中第 2 组首断实际已落盘 6 页、代理按幂等规则自查后仅补 3 页——同时暴露主代理一次状态检查 bug（cd 到仓根致 grep 路径全空、误报 0 改动），后续状态检查一律用绝对路径。validate_page 27/27、严格链接 862 处 0 错 0 警（修正后）、标签一致性 0 缺陷（含修正 2 处），集中 record refreshed 27，check 队列 27→0、全库 2,526 组对比全部 unchanged。
- [x] 2026-10-25 版本收尾三项完成，版本变更核验任务关闭：①全库滞留标签扫描——4,292 处行号引用中 63 处缺陷（workmultiplier 14、teacher 9、sleepingbag 7、worker 6 等 14 页，全部为早期批次"锚点已迁、显示标签留旧行号"），机械修复后复扫归零；②24 页批量沿用批补录——build-26148036/index.md 的 Wiki 沿用记录新增「原始哈希等价批量沿用-24」行（23 页清单＋分支依据＋检查范围/结果/未覆盖项＋旧新快照），23 页验证记录各补一行批次标识＋记录链接（linkeditem 为常规修订已有完整条目），validate_page 23/23；③全库严格链接终检——11,437 处引用 0 错 0 警（361 页／852 源文件），检查点存档的 32 错/768 警存量全部随页面处理消化，无遗留归属问题。报告 23 条待审线索经盘点为 Rifts 8／Charlie Boss 新内容行为条目（均不在既有页面记载面），按用户决策并入后续「编写 LLM wiki」任务队列。
- [x] 2026-10-25 决策：报告 23 条待审线索（Charlie Boss 竞技场／女王火把 9、尖刺发射传送回退 4、雨滴 CanPointHaveRain 门控 2、杂项 8）随版本任务关闭保留待审标记，并入后续「编写 LLM wiki」任务队列——与 14 个新增组件＋vault→virtualroom 建页同域同批转正，暂不单独补审；read-update 规则已限制其只能作候选结论。版本变更核验任务至此关闭，全部改动 amend 固化。
- [x] 2026-10-25 决策：14 个新增组件＋vault→virtualroom 迁移机制建页移出版本变更收口范围，并入后续"编写 LLM wiki"任务队列（不在旧版冻结的 98 项清单内，41/98 口径不变）；280 页收口不再等待建页。
- [x] 2026-10-24 新增组件第一批（虚拟房间与查理Boss域）9 页交付并登记：两子代理完成 virtualroommanager／virtualroomset（complex 档，数据面只记组织方式）／virtualroomteleporter 三组件页＋首篇机制页 mechanisms/vault-virtualroom-migration.md（reference/index 随之补 mechanisms 分类入口），及 stalkermanager／stalkerinspectable／charlie_tracker／shard_charlieinfo／charliearenawatcher 五组件页（simple×4、standard×1）。待审线索随域转正 4 条：shadowhandspawner 改期逻辑、vault_compass 指向、vault_lobby 布局接线按当前源码转正，SGvault_pillar_guard 判明重构与 virtualroom 无关（判定在 prefab/brain 经 IsPointInVaultRoom），23 条余 19。验收：validate_page 9/9、严格链接 295 处 0 错 0 警、标签一致性 0 缺陷、35 锚点语义抽查全实、Lua 示例语法 7 例通过＋1 布局数据节选包装后通过；record 9 页基线，登记 world-creatures 传送节＋新「暗影任务线与查理Boss状态」节；log.md 超 64 KiB 首次分卷（log-archive/log-2026-10-24-001）。子代理断流 5 次（新派发撞模型并发限制×3 改注册＋恢复路径重启、供应商流中断×2），断流均为「页面落地、汇报丢失」或 0 页受损模式，按落盘状态机械验收，未重写已落盘页面。队列余 6 组件：atriumritualorgantracker／autojiggle／corruption／deflectable／rockybossspawner／worldstaticlayouts。
- [x] 2026-10-26 新增组件第二批（腐化／岩石Boss／仪式＋偏转／抖动／布局）6 页交付并登记，26148036 新增 14 组件全部建齐：两子代理完成 corruption（王杖腐化账本＋CORRUPTNIGHTMARE 五分支分发链，页内记原版 OnLoad 守卫读新实例致 timetodecay 恒丢的怪癖）／rockybossspawner（记 TryPauseRockyBossSpawns 死代码；待审线索 prefabs/rocky.lua +306/−43 按组件接口切片转正——IsMaxSize 契约、herdmember 群籍、ROCKY_GROW_RATE/WALK_SPEED 键消费者锚点，prefab 级留待 prefab 页，待审 19→18）／atriumritualorgantracker（simple，ms_defeated_shrouden 清表与 hounded_forceplannewattack 保底波）与 deflectable（三侧偏转协议目标侧，damagetyperesist "deflectable" 键乘 0 语义）／autojiggle（记 running 判定前两项作用于宿主实体、原版 FX 宿主短路致 runanims 不参与的源码事实）／worldstaticlayouts（simple，必放布局断言语义与 placedlayouts 随档）。验收：validate_page 6/6、严格链接 208 处 0 错 0 警、标签一致性 0 缺陷、Lua 示例语法 6/6、约 25 锚点语义抽查全实；record 6 页基线；登记 combat×2／items×1（155 行逼近 160 上限，下批前评估拆分）／world-creatures×3；按代理建议补互链 5 处（projectile／complexprojectile 偏转段、charlie_tracker、机制页装配链、virtualroommanager 消费处）。本批两代理注册期撞并发限制后经恢复路径一次跑通、均存活至完整汇报，断流恢复零损耗；新 skill 规则（lua 围栏语法完整、分类索引明确化、校验绝对路径）执行无摩擦。全库终检因 Codex C50～C52 并行在建未执行，由该批收口时统一跑。
- [x] 2026-10-26 待审线索转正批（无子代理，主代理直接编写）交付：雨滴门控 2＋尖刺发射回退 4 共 6 条转正为机制页 [[reference/mechanisms/rain-fx-gating]]（rain/caverain 三处 CanPointHaveRain 门控逐点核验——粒子发射／溅落水花／雨棚补偿滴三链，caverain 仅 FX 参数与注册名差异；weather 按 `5×preciprate`／`2×preciprate` 驱动、caveweather 同构，纯客户端表现门控不改数据侧）与 [[reference/mechanisms/spike-launcher]]（四族 SpikeLaunch 本 build 逐字同改经旧新 diff 确认；TryTeleportToLaunchPos 全局语义——海洋世界恒成功、洞穴查陆地瓦；失败回退为自身位置垫高 max(y1,startheight)，与 lootdropper FlingItem 回退到掉落源上空构成同族两型对照；SetLanded(false,true) 空中标记配套）。验收：validate_page 2/2、严格链接 52 处 0 错 0 警（3 处空行起止锚点收紧）、标签一致性 0 缺陷、Lua 语法 2/2；record 2 页，mechanisms 索引 2 行。待审线索 18→12（余 Charlie Boss 9＋杂项 3，均 prefab/SG 级留待对应页）。主代理亲写批与子代理批流程等价：认领→源码→写页→四项机械自验→登记→翻转，零断流零重试。
- [ ] 新源码版本定向复查受影响结论、案例和骨架；源码／证据变化、行号移动、删除与版本差异不能静默沿用，未核验内容明确状态。
- [ ] 核验后更新页面版本、锚点、索引／互链；复查机制消费者，不因有链接就认定骨架兼容。
- [ ] 保留有用经验；被替代内容归档并给替代入口，不按未引用时长自动删除。

自动影响分析与增量语义迁移仍是工具箱暂缓项，当前可人工复查，不设为知识维护前置。全库类型精审、图数据库、向量检索、网页与定时同步不在本仓近期任务中。
