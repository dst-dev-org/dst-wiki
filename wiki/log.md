# DST Wiki 维护日志

上一卷：[[log-archive/log-2026-10-24-001]]。

- 2026-10-24 新增组件批「虚拟房间与查理Boss域」9 页交付：两子代理完成 virtualroommanager／virtualroomset（complex 档，1252 行源码数据面只记组织方式）／virtualroomteleporter 三组件页＋首篇机制页 [[reference/mechanisms/vault-virtualroom-migration|vault→virtualroom 迁移]]，及 stalkermanager／stalkerinspectable／charlie_tracker／shard_charlieinfo／charliearenawatcher 五组件页（后四者 simple 档）。机制页顺带把报告待审线索 4 条按当前源码转正（shadowhandspawner 改期、vault_compass 指向、vault_lobby 布局接线；SGvault_pillar_guard 核验结论为与 virtualroom 无关、判定在 prefab/brain），待审线索 23→19。主代理验收：validate_page 9/9、严格链接 295 处 0 错 0 警、标签一致性 0 缺陷、35 锚点语义抽查全实、Lua 示例语法 7 例通过＋1 布局数据节选包装后通过；record 9 页基线。登记：world-creatures 索引传送节＋新「暗影任务线与查理Boss状态」节、reference/index 补 mechanisms 分类入口。子代理各断流 2～3 次（新派发撞并发限制改恢复路径重启；断流均为「页面落地、汇报丢失」或 0 页受损模式，机械验收接管）。队列余 6 组件（atriumritualorgantracker／autojiggle／corruption／deflectable／rockybossspawner／worldstaticlayouts）与 19 条待审线索待后续批次。

- 2026-10-26 通用 C49：新增 [[reference/components/formationleader]]／[[reference/components/formationfollower]]，依据 `steam-buildid:26148036`，覆盖环形位置、多圈分层、成员事件与 lightflier 宿主／Brain 接线，说明 active 不直接停止组件更新。
  - 两页及 world-creatures 叶级导航格式／链接通过；99 处源码引用 strict 零错警、99 处显示标签与锚点一致；1 例完整 Lua 语法和两次代表章节读取通过，record 新增 2 页。只验收本批，另一对话在建页的导航缺口不代修；未运行游戏。
  - 更新覆盖清单与 PLAN，固定 98 项进度 43／98、剩 55；保留并行认领与已有记录，临时语法文件已删除。
- 2026-10-26 新增组件第二批（腐化／岩石Boss／仪式＋偏转／抖动／布局）6 页交付，26148036 新增 14 组件全部建齐：两子代理完成 corruption（王杖腐化账本＋CORRUPTNIGHTMARE 分发链，页内记原版 OnLoad 丢 timetodecay 怪癖）／rockybossspawner（含 TryPauseRockyBossSpawns 死代码发现；rocky.lua 待审条目按接口切片转正，prefab 级留待 prefab 页）／atriumritualorgantracker（simple）与 deflectable（三侧偏转协议目标侧）／autojiggle（页内记 running 判定对 FX 宿主短路、runanims 实际不参与的源码事实）／worldstaticlayouts（simple）。主代理验收：validate_page 6/6、严格链接 208 处 0 错 0 警、标签一致性 0 缺陷、Lua 示例语法 6/6、约 25 锚点语义抽查全实；record 6 页基线。登记：combat×2／items×1（155 行逼近 160 上限，下批前评估拆分）／world-creatures×3；补互链 5 处（projectile／complexprojectile 偏转段、charlie_tracker、机制页装配链、virtualroommanager）。本批两代理均存活至完整汇报（断流仅发生在注册期并发限制，恢复路径重启后一次跑通）；新 skill 规则（lua 围栏、分类索引、绝对路径）执行无摩擦。待审线索 19→18（rocky.lua 接口切片转正）。

- 2026-10-26 通用 C50～C52：三个子代理新增 [[reference/components/hatchable]]／[[reference/components/hideout]]、[[reference/components/followermemory]]／[[reference/components/leaderrollcall]]、[[reference/components/butterflyspawner]]／[[reference/components/lureplantspawner]]；依据 `steam-buildid:26148036`，主代理复核孵化与收放、身份记忆与点名、自然调度及宿主边界。
  - 六页及两张导航格式／链接、导航登记检查通过；250 处源码引用 strict 零错警，250 处显示行号匹配锚点，三次代表章节读取完整；record 新增 6 页。本批无新增 Lua 示例，未运行游戏。
  - 覆盖清单与 PLAN 已同步，固定 98 项进度 49／98、剩 49；保留另一对话新增组件批的页面及共享记录，本轮未产生临时文件。
- 2026-10-26 待审线索转正批（无子代理，主代理直接编写）：雨滴门控 2＋尖刺发射回退 4 共 6 条待审线索转正为两张机制页——[[reference/mechanisms/rain-fx-gating|雨滴 FX 与无雨区门控]]（rain/caverain 三处 CanPointHaveRain 门控逐点核验，caverain 仅 FX 参数差异；weather/caveweather 强度驱动链与雨棚补偿滴）与 [[reference/mechanisms/spike-launcher|尖刺发射物与发射传送回退]]（四族 SpikeLaunch 逐字同改确认＋TryTeleportToLaunchPos 全局语义＋原地垫高回退与 lootdropper FlingItem 回退到源上空的对照）。validate_page 2/2、严格链接 52 处 0 错 0 警（3 处空行起止锚点收紧后）、标签一致性 0 缺陷、Lua 示例语法 2/2；record 2 页，mechanisms 索引登记 2 行。待审线索 18→12（余 Charlie Boss 9＋杂项 3）。PLAN 认领条目已翻转；与 Codex C53～C55 并行无碰撞（mechanisms 域零交集）。

- 2026-10-26 通用 C53～C55：三个子代理新增 [[reference/components/followerherder]]／[[reference/components/squadmember]]、[[reference/components/minionspawner]]／[[reference/components/objectspawner]]、[[reference/components/rampingspawner]]／[[reference/components/schoolspawner]]；依据 `steam-buildid:26148036`，主代理复核实体字段／监听、生成与恢复、配额和自然鱼群分层。
  - 六页及受影响旧页、两张导航格式／链接和导航登记检查通过；247 处源码引用 strict 零错警，247 处显示行号匹配锚点，三次代表章节读取完整；record 新增 6 页。既有 lureplantspawner 仅补眼球草机制链接，不刷新其基线；无新增 Lua 示例，未运行游戏。
  - 覆盖清单与 PLAN 已同步，固定 98 项进度 55／98、剩 43；与另一对话机制页批的共享增量已合并保留，本轮未产生临时文件。

- 2026-10-26 通用 C56～C58：三个子代理新增 [[reference/components/batspawner]]／[[reference/components/circler]]、[[reference/components/shedder]]／[[reference/components/distancetracker]]、[[reference/components/amphibiouscreature]]／[[reference/components/carefulwalker]]；依据 `steam-buildid:26148036`。明确 batspawner 未完整导出、distancetracker 无累计输出，以及产物高度、水陆／跳跃分工和谨慎行走能力门槛。
  - 六页及四张导航格式／链接、导航登记检查通过；195 处源码引用 strict 零错警，195 处显示行号匹配锚点，三次代表章节读取完整；record 新增 6 页。四页简单档、两页标准档，无新增 Lua 示例，未运行游戏。
  - 覆盖清单与 PLAN 已同步，固定 98 项进度 61／98、剩 37；G07 生物生成与协作全部 19 项已覆盖，下一批转向 G09 剩余候选。本轮未产生临时文件。

- 2026-10-26 通用 C59～C61：三个子代理新增 [[reference/components/playerspeedmult]]／[[reference/components/unevenground]]、[[reference/components/heavyobstaclephysics]]／[[reference/components/heavyobstacleusetarget]]、[[reference/components/pushable]]／[[reference/components/physicsmodifiedexternally]]；依据 `steam-buildid:26148036`。主代理复核来源、预测、物理与动作分工，补入 playerspeedmult 标准移除路径的 dirty 回调风险。
  - 六页及受影响旧页、五张导航格式／链接和导航登记检查通过；246 处源码引用 strict 零错警，246 处显示行号匹配锚点，三次代表章节读取完整；record 新增 6 页。旧 carefulwalker 仅补 unevenground 链接，不刷新其基线；无新增 Lua 示例，未运行游戏。
  - 覆盖清单与 PLAN 已同步，固定 98 项进度 67／98、剩 31；items 导航 156 行，后续新增前需按预算评估局部拆分。本轮未产生临时文件。

- 2026-10-26 通用 C62～C64：三个子代理新增 [[reference/components/commander]]／[[reference/components/crewmember]]、[[reference/components/joustsource]]／[[reference/components/joustuser]]、[[reference/components/embarker]]／[[reference/components/amorphous]]；依据 `steam-buildid:26148036`。主代理复核士兵回调实参、两类船队退出不对称、冲刺门禁、登船完成／取消与形态回调边界。
  - 六页及五张导航格式／链接、导航登记检查通过；260 处源码引用 strict 零错警，260 处显示行号匹配锚点，三次代表章节读取完整；record 新增 6 页。均为标准档，无新增 Lua 示例，未运行游戏。
  - 覆盖清单与 PLAN 已同步，固定 98 项进度 73／98、剩 25；items 导航 157 行。本轮未产生临时文件。

- 2026-10-26 通用 C65～C67：三个子代理新增 [[reference/components/coach]]／[[reference/components/grouptargeter]]、[[reference/components/strafer]]／[[reference/components/stuckdetection]]、[[reference/components/distancefade]]／[[reference/components/replayproxy]]；依据 `steam-buildid:26148036`。主代理复核鼓舞筛选与停止、权重反馈、预测朝向、采样频率和表现／身份数据边界。
  - 六页及三张导航格式／链接、导航登记检查通过；189 处源码引用 strict 零错警，189 处显示行号匹配锚点，三次代表章节读取完整；record 新增 6 页。三页简单档、三页标准档，无新增 Lua 示例，未运行游戏。
  - 覆盖清单与 PLAN 已同步，固定 98 项进度 79／98、剩 19；grouptargeter 覆盖指纹改记实际已审的当前快照。本轮未产生临时文件。

- 2026-10-26 通用 C68～C70：三个子代理新增 [[reference/components/stretcher]]／[[reference/components/simplemagicgrower]]、[[reference/components/forgerepair]]／[[reference/components/forgerepairable]]、[[reference/components/custombuildmanager]]／[[reference/components/symbolswapdata]]；依据 `steam-buildid:26148036`。主代理复核拉伸与成长、现用修理包分支及破损恢复、符号覆盖与实际数据消费者。
  - 六页及三张导航格式／链接、导航登记检查通过；179 处源码引用 strict 零错警，179 处显示行号匹配锚点，三次代表章节读取完整；record 新增 6 页。四页简单档、两页标准档，无新增 Lua 示例，未运行游戏。
  - 覆盖清单与 PLAN 已同步，固定 98 项进度 85／98、剩 13；G09与G10全部覆盖，items 导航达到160行，继续新增前需局部拆分。本轮未产生临时文件。

- 2026-10-27 通用 C71～C73 收尾：三个子代理新增 [[reference/components/skinner]]／[[reference/components/wardrobe]]、[[reference/components/transformer]]／[[reference/components/reloaditem]]、[[reference/components/roseinspectable]]／[[reference/components/roseinspectableuser]]；页面于10月26日按 `steam-buildid:26148036` 核验。主代理复核换装会话、实体恢复缺口、装填分派与残影冷却，完成统一登记。
  - 六页及五张导航格式／链接、导航登记检查通过；257 处源码引用 strict 零错警，257 处显示行号匹配锚点，三次代表章节读取完整；record 新增 6 页。四页标准档、两页简单档，无新增 Lua 示例，未运行游戏。
  - 覆盖清单与 PLAN 已同步，固定 98 项进度 91／98、剩 7；G11全部覆盖，items 保持160行。skinner覆盖指纹改记实际已审的当前快照；本轮未产生临时文件。

- 2026-10-27 通用 C74～C76：三个子代理新增 [[reference/components/giftreceiver]]／[[reference/components/shop]]／[[reference/components/researchpointconverter]]、[[reference/components/pointofinterest]]／[[reference/components/scrapbookable]]、[[reference/components/recipestockpile]]／[[reference/components/treasuremarked]]；依据 `steam-buildid:26148036`。主代理复核实际消费者、旧接口和清理边界后统一登记。
  - 七页及四张导航格式／链接、导航登记检查通过；219处源码引用strict零错警，139处数字行号标签匹配锚点，三次代表章节读取完整；record新增7页。三页标准档、四页简单档，无新增Lua示例，未运行游戏。
  - 覆盖清单与PLAN已同步，固定98项逐项核对为98／98、剩0、98个独立页面；按约定暂停组件扩库，先提醒用户并重整索引。pointofinterest指纹改记实际已审快照；本轮未产生临时文件。

- 2026-10-27 索引首版试用：根导航直达 [[reference/components/index#原版接入量-top-30|原版组件接入量 Top 30]]；依据 `steam-buildid:26148036` 的4,084个Lua文件，按直接AddComponent调用的不同文件数排序，保留updatelooper缺页源码入口，不将公共helper间接接入或查询频率混入统计。九路主题摘要精简，叶级登记和正文不动。
  - 两张索引格式／链接与全库导航检查通过（419知识页／17索引，唯一登记）；新增源码链接strict零错警。报告代码复现30项排名一致，29个Wiki目标active；榜单、主题导航及已知名称三次章节读取完整。PLAN／设计已同步，复现记录为K相对路径 `docs/reports/w05/component-top30-26148036.md`。
  - 仅导航试用，完整主题重整尚未完成；不刷新知识基线，不改游戏知识或Skill／工具实现，未运行游戏，未生成临时文件。

- 2026-10-27 组件主题细化：[[reference/components/index#按需求浏览|组件导航]]由九主题细分为16主题，新增环境、维护、制作、农作、驯养协作、移动与世界管理；迁移164条登记，物品交互按五组细分，物品食物／垂钓分别归加工／航海。408条原登记行及85个旧标题锚点保留，正文与Top30统计不动。
  - 15张新增／修改索引格式、大小、链接与全库导航通过：419知识页／24索引，每篇唯一登记；11次新分组与旧入口读取完整。items从160降至114行，world-creatures从137降至77行；修正了新增链接的标点／大小写锚点，收尾零问题。
  - PLAN／设计同步，工程验收为K相对路径 `docs/reports/w05/component-topic-refinement.md`。本轮未改游戏知识、来源或基线，未运行游戏；不重建缓存，未生成临时文件。

- 2026-10-27 版本更新源码准备：setup_raw.ps1 按本机 Steam 定位并解压新 build 26230496（4091 个 .lua），raw/current 由 26148036 切换，旧快照归档至 raw/archive/build-26148036。知识页未动，版本增量维护随后开始。

- 2026-10-27 检索修整Q01：[[reference/components/health]]补人物65535同步边界，[[reference/components/childspawner#洞穴黄昏刷新蝙蝠由谁控制]]补昼夜宿主接线，batspawner／hideout修正分流；新增[[reference/components/oldager]]解释旺达自然老化、受伤和回春，并注明客户端表盘预测边界；[[reference/components/preserver]]补普通背包禁入与模组嵌套前提，战斗索引增加装备耐久选型。
  - 六页及四张索引格式／大小／链接与导航通过（420知识页／24索引）；222处源码引用strict零错警，113处数字标签匹配；六次定向章节读取完整。覆盖清单与PLAN已同步，组件409篇、旧候选395已建／403未建；homeseeker已有覆盖沿用。
  - 执行中current被其他流程切换为26230496，本批证据固定到`raw/archive/build-26148036/scripts/`，不宣称新版本已核验。专用工具仅为oldager新增基线、health／childspawner刷新来源清单，其余条目保留；归档快照范围check为40／40 unchanged。未运行游戏，无新增Lua示例或临时文件。

- 2026-10-27 人物索引细化：[[reference/components/topics/character#生命与寿命|生命与寿命]]集中登记health／oldager／healthsyncer；同一原章节下另分饥饿与精神、进食偏好与记忆，原“人物三维与进食”锚点保留，各页仍唯一登记。
  - 页面格式／大小／链接与全库导航检查通过（420知识页／24索引）；新旧两个章节均可完整读取。仅改人物索引，Skill查询方式与索引策略本轮只作检视，未改规则、知识正文或基线；无临时文件。

- 2026-10-28 Skill查询路由优化：修改知识Skill包内`references/query/query.md`，区分已确认名称、未知需求、机制选型与组装；元数据和主题索引均可用于定位，已有足够候选即直查，技术依据仍取自知识页，模型猜测名称不当作确认。
  - Skill工程设计／PLAN同步；包结构校验通过，27份文档152处本地链接／锚点有效，已知名称、装备耐久、蝙蝠与旺达定位受控检查通过。根SKILL.md内容哈希保持原样，未扩默认读取或改知识正文／基线，无临时文件；本轮未重做独立代理行为评测，不宣称token收益。

- 2026-10-27 版本更新报告 [[updates/build-26230496/index]] 生成：setup_raw 切换快照后按 write-update 生成流水线完成——154 文件（147 改／7 增／0 删）通读 diff 后归类五明细（components 13／root 23／prefabs 36／sg 58／skins-loc 24，95 条编号条目＋SB-10 附表），index 含主题分诊＋适配检查清单＋分类导航；机器附件 changed-files.txt／check-report.json 入报告目录。重点结论：通用击退 SG 框架（49 SG 接入＋explosive SetKnockbackData）、拟态揭示配置化（ShouldItemMimicBeRevealedFor 三参化＋itemmimicconfig，castonrecipes 特例移除）、AddIsTileInvalidForPathing_VirtualRoomSet 只认内置 VRS 名（自定义注册静默丢弃）、WORLD_SPECIAL_EVENT 默认 HALLOWED_NIGHTS、万圣新内容（SWEEP／leafpile／witchbroom／pumpkin_bomb）、Charlie/Shrouden 反斩范围扩大、南瓜腐烂翻倍。校验：报告页与上一版报告同为 updates/ 页豁免知识页 schema，严格链接 6 页 0 错 0 警。待审 1 条（P-16 AOEUtil.Work 意图）。知识页迁移未开始（漂移队列 350 页）。

- 2026-10-28 版本迁移首批「原始哈希等价批量沿用-70」完成：26148036→26230496，70 页（sources 所列文件全部未变、不在漂移队列）升 dst_build／updated 2026-10-28，各补「版本沿用」行链接 [[updates/build-26230496/index#wiki-沿用记录]]（squadmember／simplemagicgrower 为 simple 8KiB 档贴线，页内行用紧凑变体）。三项批量检查：review-needed 标记 0；38 个本轮变化符号反查仅 spike-launcher.md 命中 "knockback"（sharkboi 冰刺事件推送侧本轮未变，排除影响）；严格链接 2,019 处 0 错 0 警。validate_page 70/70 通过（2 页缩行后过线）；oldager 无基线已 record；未变页不 record 基线。余 280 页＋sources_changed 2 页（childspawner／health）走常规迁移。全库对新快照 strict 预检 14,190 处 0 错 561 警（变化文件行移导致的锚点空行落点，随各页迁移消化）。

- 2026-10-28 版本迁移批A（组件域 38 页）完成：explosive／spooked／amphibiouscreature／hounded／locomotor／map／playercontroller／brushable 变化的全部受影响页。机械迁移 219 处（补丁 29 文件＋CRLF 4 页建议手工），review 46 项逐条人工处理；语义修订 5 页（explosive C-7、explosiveresist PVP 抗性积累门禁语义、spooked C-6、amphibiouscreature C-9、vault-migration 锚点）；标签同步 111 处。validate_page 38/38、严格链接 1,424 处 0 错；record 38 页，dst_build 全部升 26230496。沿袭记录落 [[updates/build-26230496/index#wiki-沿用记录]]。

- 2026-10-28 高价值缺页Q02：新增 [[reference/components/updatelooper]]，依据steam-buildid:26230496，覆盖四类更新回调、首次执行、延迟删除与移除组件、裂缝淡入及南瓜显式初始化接线；在[[reference/components/topics/abilities#更新调度|更新调度]]唯一登记，Top30第15项换为Wiki链接，原26148036静态接入量统计不重算。
  - 新页及两张索引格式／大小／链接与导航通过（421知识页／24索引）；47处源码引用strict零错警，五次定向章节读取完整。生成器新增单页基线、check为6／6 unchanged；覆盖清单和PLAN同步，组件410篇、旧候选396已建／402未建。未运行游戏，无新增Lua示例或临时文件。

- 2026-10-28 版本迁移批B（枢纽域 312 页）完成，26230496 版本迁移收口：漂移队列 350 页全部处理完毕（沿用-70＋批A 38＋批B 312）。批B 机械迁移 1,144 处（581 hunk 补丁 259 文件＋CRLF 38 处建议手工），review 50 项逐条人工（actions +10 偏移 25 锚、SGchester/SGwilson/constants 等逐个定位 6 页、源码核对保留 5 页），标签同步 437 处；validate_page 312/312、严格链接 10,751 处 0 错 0 警；record 312（childspawner／health sources_changed 随基线重建消除）。收尾全库 check 2,996 组对比全部一致、affected 0、sources_changed 0。沿袭记录落 [[updates/build-26230496/index#wiki-沿用记录]]。

- 2026-10-28 版本收尾终检：全库哈希基线 2,996 组对比全部一致、严格链接 445 页 14,241 处 0 错 0 警、validate_page+索引 444 页全过、dst_build 无 26148036 残留（421 页全部 26230496）、报告 8 文件确认在 HEAD。标签一致性全库复扫发现 35 处残留（批B review 修正晚于标签同步所致＋历史单行/区间格式噪声），7 页再同步后归零，随本条 amend 固化。

- 2026-10-28 26230496 新增组件建页 4 页（报告 C-1～C-4 转正）：[[reference/components/sweeper]]（扫帚空标记与四条动作入口）／[[reference/components/sweepable]]（标签门禁＋Sweep 回调，Sweep 不校验 enabled 陷阱）／[[reference/components/leafpilemanager]]（活动＋秋季门控、DataGrid 分区限密、衰减调度与持久化）／[[reference/components/itemmimicconfig]]（强制揭示动作表＋ShouldItemMimicBeRevealedFor 判定链五步）。两子代理按新页派发模板建页并自验，主代理复核发现报告 C-1～C-6／R-1～R-3 转抄行号错误（新文件实际 5／31／218／27 行、itemmimic 调用 ：51 非 ：56、SWEEP 定义 ：745-746 与 fn :7126-7133、componentactions 两 handler 实在 USEITEM/EQUIPPED 段、ShouldItemMimicBeRevealedFor 实为 :2331-2363、旧函数止 ：2349、spooked IsSpookImmune 止 :50），页面按实核写入，报告已勘误并在 index 新增节注明 C-1～C-4 已建页、行号以页面为准。
  - 验收：validate_page --check-indexes 4/4（425 知识页导航无未登记）、严格链接 4 页＋2 索引 122 处 0 错 0 警、标签一致性 0 缺陷、Lua 示例 4 例 parser_probe 通过、record 4 且 check affected 0；登记 item-actions"使用切档与重物交互"3 行＋world-management 新"活动期地表生成管理"节，sweepable／leafpilemanager 互链补齐；leafpile／witchbroom／pumpkin_bomb prefab 页仍待认领。

- 2026-10-28 组件缺页持续建页批启动（覆盖清单 unbuilt 402 候选；vault_floor_helper／vault_teleporter／vaultroom／vaultroommanager 已随 26148036 vault→virtualroom 迁移删除源码，记 removed-source 不建页，实建目标 398）：批A-01 十二页 vasedecoration／moonrelic／carnivalgameitem／carnivalgamefeedable／carnivalhostsummon／pumpkincarver／snowmandecor／talkable／waterphysics／soul／winter_treeseed／bathbomb 全 simple 档——装饰/供品/雕刻标记族、嘉年华巢喂开关与召唤标记、船面弹性、灵魂、冬之树种、浴弹；talkable 与 carnivalhostsummon 原版无挂载者按源码事实记录。
  - 验收：标签 0 缺陷、validate_page --check-indexes 18 文件通过（新建 [[reference/components/topics/activity|activity 主题]]并在组件索引加"专项与活动"组）、严格链接 0 错 0 警、Lua 示例 10 例 parser_probe 过、record 12；登记 item-actions 7＋items 1＋world 1＋world-ships 1＋activity 2；覆盖清单同步 built 408／unbuilt 386／removed-source 4。

- 2026-10-28 持续建页批A-02 十四页：kitcoon／storytellingprop／yotb_skinunlocker／wobyrack／balloonmaker／platformhopdelay／strongman／slingshotmodder／ghostlyelixirable／houndedtarget／shatterfx／migrationpetsoverrider／oceanthrowable／savedscale（宠物标记与灵药目标、讲故事道具、YOTB 皮肤蓝图、沃比晾晒架、弃用气球制造器、平台起跳延迟、健身会话、弹弓改装、猎犬目标权重、碎冰特效、迁移宠物落点、掷海标记、缩放存读）。
  - 验收：标签 0 缺陷、validate_page --check-indexes 24 文件通过、严格链接 0 错 0 警、Lua 5 例通过、record 14、覆盖清单 built 422／unbuilt 372；登记 10 主题文件 14 行（登记脚本行内锚点缺陷污染 8 文件已当场修复并改整行锚定；houndedtarget 暂落 world-creatures 飞鸟节，留清理批归位）；luckuser／locomotor 反向互链留批外清理批。累计 26/398。

- 2026-10-28 持续建页批A-03 十六页：socket_shadow_heart／playermetrics／oasis／worldsettings／boatai／moonsparkchargeable／shadowlevel／playbill／singable／ocean／savedrotation／area_unlock／cattoy／playerinspectable／wintertreegiftable／sharkboimanagerhelper（WX78 插座视觉件、遥测上报、绿洲范围、世界设置表、原型期船 AI／洋流／区域解锁三件无消费者如实记录、月暴充电、影子等级、戏单、战斗歌谣、朝向存读、猫玩具、头像弹窗数据、冬季树收礼、鲨鱼竞技场同步）。
  - 验收：标签复检随批、validate_page --check-indexes 26 文件通过、严格链接 0 错 0 警、Lua 13 例通过、record 16、覆盖清单 built 438／unbuilt 356；登记 10 主题文件 16 行。写作自纠 oasis 注销路径与 shadowlevel 消费者归属。累计 42/398。

- 2026-10-28 持续建页批A-04 十六页：mutatedbirdmanager／worlddeciduoustreeupdater／vaultorbteleportdestination／wagboss_tracker／boattrailmover／halloweenpotionmoon／souleater／oceanfishingtackle／scaler／walkingplankuser／area_trigger／pocketwatch／vanish_on_sleep／ghostlyelixir／hudindicatable／uniqueid（两件弃用空壳组件如实记录、宝库球传送标记、wagboss 击败状态、船尾水花、月之药水、食魂者、海钓渔具数据、缩放入口、踏板配对、区域触发死代码、怀表施法基座、休眠消失、幽灵灵药、HUD 指示注册、prefab 分桶唯一编号）。
  - 验收：validate_page --check-indexes 25 文件通过、严格链接 0 错 0 警、Lua 13 例通过、record 16、覆盖清单 built 454／unbuilt 340；登记 9 主题文件＋新建 [[reference/components/topics/display|display 显示与表现主题]]（HUD 指示域，G14 后续落此）并挂组件索引；souleater→soul、ghostlyelixir→ghostlyelixirable 批内互链落地。uniqueid 按实体关联域登记 world-management（代理建议 processing 已酌改）。累计 58/398。

- 2026-10-28 持续建页批A-05 十六页：winchtarget／clientpickupsoundsuppressor／crabkingspawner／migrationpetowner／riftthralltype／uniqueprefabids／socket_shadow_mimicry／yotc_racestart／expertsailor／mermcandidate／minigame_spectator／boatrace_proximitybeacon／deathloothandler／playerlightningtarget／moontrader／dumbbelllifter（绞车打捞目标、拾取音效抑制、蟹王读档兜底、迁移宠物持有者、裂隙爪牙类型、prefab 计数器、WX78 拟真开关、卡拉特赛跑起点、资深水手、鱼人王候选、小游戏观众、船赛信标、死亡战利品记录、玩家雷击目标、月亮献祭、举哑铃会话）。
  - 验收：validate_page --check-indexes 25 文件通过、严格链接 226 处 0 错 0 警、Lua 3 例通过、record 16、覆盖清单 built 470／unbuilt 324；登记 9 主题文件 16 行。事实修正：minigame_spectator 经 Minigame:AddSpectator 动态添加（派发单"无消费者"有误）、crabkingspawner 存档字段无本快照写入者按旧档兼容记录。累计 74/398。

- 2026-10-28 持续建页批A-06 十六页：spidermutator／carnivaldecor／groundcreep／bathbombable／ghostgestalter／moonaltarlink／furnituredecor／walkingplank／colouraddersync／shard_worldreset／winonateleportpadmanager／moonstormstaticcatcher／itemaffinity／kitcoonden／shard_wagbossinfo／shard_autosaver（蜘蛛变异器、嘉年华装饰、蠕变垫片、浴弹目标端、月晷变形、月坛连线、家具装饰、船踏板、附加色同步、温妮传送垫注册表、静电捕捉、物品亲和、小猫狸窝）。
  - 验收：validate_page --check-indexes 26 文件通过、严格链接 225 处 0 错 0 警、Lua 16 例通过、record 16、覆盖清单 built 486／unbuilt 308；新建 [[reference/components/topics/network-shard|network-shard 网络与分片主题]]收纳 shard_* 三页（G18 后续落此）；源码事实如实记录 mutate 事件无监听者、DISMOUNT_PLANK 调不存在的 DismountPlank 等四处死代码。累计 90/398。

- 2026-10-28 持续建页批A-07 十六页：worldroutes／discoverable／groundshadowhandler／cookiecutterdrill／worldwind／hideandseeker／highlightchild／fishschool／shadowsubmissive／fader／gestaltcage／carnivalgameshooter／shardstate／carnivaldecorranker／despawnfader／shadowdominance（世界路线缓存、地图发现、贴地影子、钻船计时、世界风向、捉迷藏寻找者、子实体高亮、鱼群等级、暗影臣服、数值渐变、织影牢笼、射击小游戏、主分片会话 ID、装饰评分、淡出移除、暗影压制；shadowsubmissive↔shadowdominance 批内互链）。
  - 验收：validate_page --check-indexes 25 文件通过、严格链接 258 处 0 错 0 警、Lua 16 例通过、record 16、覆盖清单 built 502／unbuilt 292；登记 9 主题文件 16 行；源码事实如实记录 discoverable 无消费者、shadowsubmissive OnRemoveFromEntity 引用未定义全局、CARNIVALGAME_SHOOTING_POWER 键不存在等三处。累计 106/398。

- 2026-10-28 持续建页批A-08 十六页：sizetweener／scenariorunner／canopylightrays／experiencecollector／ruinsshadelingspawner／maxlightspawner／boattrail／worldmeteorshower／mapspotrevealer／moonaltarlinktarget／playingcard／globaltrackingicon／driedsalticon／lunarhailbirdsoundmanager／singingshelltrigger／undertile（缩放渐变、场景脚本执行器、树冠光柱、技能经验累积、遗迹暗影小妖、麦克斯韦光球、船尾水花、月岩壳概率、地图揭示、月坛连线端点、扑克牌、全局追踪图标、盐粒图标、月雹鸟音效、海螺触发器、地皮底层）。
  - 验收：validate_page --check-indexes 25 文件通过、严格链接 200 处 0 错 0 警、Lua 16 例通过、record 16、覆盖清单 built 518／unbuilt 276；登记 9 主题文件 16 行；批外互链建议 10 条入 output/interlink-backlog.txt 留清理批。累计 122/398。

- 2026-10-28 持续建页批A-09 十六页：emitter／golfpropitem／minigame_participator／archivemanager／lordfruitflytrigger／madsciencelab／ghostbabysitter／hudindicatablemanager／moonstormlightningmanager／vaultorbteleporter／boatrace_proximitychecker／debugger／wintersfeasttable／lunarplant_tentacle_weapon／nis／tributable（粒子发射、高尔夫道具、小游戏参与者、档案馆供电、果蝇王触发器、疯狂科学实验室进度机、幽灵照看名单、HUD 登记表、月风暴落雷、宝库球传送器、船赛检查点、调试画线、盛宴餐桌、月晶触手武器、过场演出、贡品积累器）。
  - 验收：validate_page --check-indexes 25 文件通过、严格链接 0 错 0 警、Lua 16 例通过、record 16、覆盖清单 built 534／unbuilt 260；登记 9 主题文件 16 行；madsciencelab（职责待确认件）按源码实际行为"分阶段制作进度机"成页；五处无消费者结论经全库 grep 复核。随批执行互链清理批：13 对批外反向链落地（2 对已存在跳过）、严格链接复检 0 错 0 警。累计 138/398。

- 2026-10-28 持续建页批A-10 十六页：halloweenmoonmutable／playerfloater／disappears／lavaarenamusic／wobycourier／oceancolor／boatdrifter／golfclub／moonstormstaticcapturable／playertargetindicator／wx78_abilitycooldowns／lavaarenamobtracker／touchstonetracker／walkableplatformmanager／revivablecorpse／wx78_dronescouttracker（月突变、游泳圈、自动消失、熔炉 BGM、沃比快递、海面着色、船漂移、高尔夫球杆、静电捕捉目标、屏外玩家标记、WX78 技能冷却、熔炉怪物登记、重生石记录、平台管理器、可复活尸体、侦察无人机追踪）。
  - 验收：validate_page --check-indexes 通过（首跑抓出 disappears 漏登记，补登后复检 True）、严格链接 277 处＋互链 5 页 0 错 0 警、Lua 15 例通过、record 16、覆盖清单 built 550／unbuilt 244；登记 10 主题文件 16 行＋批外反向链 5 条；lavaarenamusic 服务端初始化在快照外如实记缺口。累计 154/398。

- 2026-10-28 持续建页批A-11 十六页：lavaarenaeventstate／transparentonsanity／shard_daywalkerspawner／shard_sinkholes／singingshellmanager／golfable／quagmire_music／vaultmusiclistener／projectedeffects／beaverness／playerhearing／golfballspinner／hudindicatorwatcher／sandstormwatcher／canopyshadows／colourtweener（熔炉比赛状态、理智渐隐、挖掘者跨分片、塌陷跨分片、海螺管理器、高尔夫目标、大亨 BGM、宝库音乐、投影特效、海狸值旧件、闷音滤波、高尔夫旋盘、HUD 监视器、沙暴监听、树冠遮阴、颜色补间）。
  - 验收：validate_page --check-indexes 21 文件通过、严格链接 228 处 0 错 0 警、Lua 13 例通过、record 16、覆盖清单 built 566／unbuilt 228；登记 5 主题文件 16 行（display 单批 8 页）；批内 6 对互链落地；beaverness 零消费者与 radialstrength 死配置如实记录。累计 170/398。

- 2026-10-28 持续建页批A-12 十六页：lunarsupernovablocker／moonstormwatcher／moosespawner／wanderingtraderspawner／wereeater／gestaltcapturable／mightydumbbell／skeletonsweeper／questowner／wagpunk_floor_helper／wereness／playbill_lecturn／shard_seasons／lunarfissureburning／spawnfader／age（超新星遮挡、月暴减速、驼鹿生成、流浪商人生成、吃肉变身、织影捕获目标、哑铃负载、骷髅上限、任务宿主、竞技场几何同步、变身值、剧本架、跨分片季节、月裂灼烧、生成淡入、实体年龄）。
  - 验收：validate_page --check-indexes 25 文件通过、严格链接 0 错 0 警、Lua 2 例通过、record 16、覆盖清单 built 582／unbuilt 212；登记 9 主题文件 16 行；questowner（职责待确认件）按源码实为 BEGIN/ABANDON_QUEST 宿主成页；age 线索纠偏（crabking 本体未挂，主力挂载为玩家）如实记录。累计 186/398。

- 2026-10-28 持续建页批A-13 十六页：miasmawatcher／shard_clock／maprevealer／storyteller／knowndynamiclocations／sisturnregistry／spectatorcorpse／boatcannon／friendlevels／hideandseekhidingspot／nightlightmanager／playervoter／steeringwheeluser／yotd_raceprizemanager／freezefx／areaaware（瘴气检测、分片时钟、地图揭示器、讲故事、动态位置记忆、姐弟祭坛注册表、尸体观战、船炮、好感等级、捉迷藏藏身点、夜灯管理器、玩家投票、操舵、龙舟奖品、死组件 freezefx、区域感知）。
  - 验收：validate_page --check-indexes 23 文件通过、严格链接 374 处＋互链 5 页 0 错 0 警、Lua 12 例通过、record 16、覆盖清单 built 598／unbuilt 196；登记 9 主题文件 16 行＋批外反向链 5 条；hideandseekhidingspot 运行时挂载、areaaware 五处挂载、storyteller 死字段等如实记录。累计 202/398。

- 2026-10-28 持续建页批A-14 十六页：lunaralterguardianspawner／colouradder／deckcontainer／hideandseekhider／battleborn／snowballmelting／seamlessplayerswapper／nabbag／chessunlocks／npc_talker／sharklistener／bloomness／lighttweener／petrifiable／stormwatcher／camerafade（月裂守卫生成、附加色堆叠、牌组容器、捉迷藏藏匿者、战吼积攒、雪球融化、无缝换角色、抓取袋、棋子解锁、NPC 台词队列、鲨鱼监听、绽放度、灯光补间、可石化、风暴观测、镜头淡化）。
  - 验收：validate_page --check-indexes 25 文件通过、严格链接 225 处 0 错 0 警、Lua 16 例通过、record 16、覆盖清单 built 614／unbuilt 180；登记 9 主题文件 16 行；sharklistener 无读取者、hideandseekhider.IsHidden 未定义引用如实记录；标签超分类法两处代理自纠。累计 218/398。

- 2026-10-28 持续建页批A-15 十六页：highlight／corpsepersistmanager／lightpostpartner／abysspillargroup／furnituredecortaker／gym／shard_players／minigame／trapfumaroleburning／blowinwind／lunarsupernovaburning／carnivalevent／sandstorms／forestdaywalkerspawner／shard_worldvoter／bloomer（悬停高亮、尸体留驻、灯杆挂点、深渊柱阵、家具承物、卡墩健身房、跨分片人数、小游戏主机、地热灼烧、随风飘行、超新星灼烧、嘉年华活动、沙尘暴管理、地表挖掘者、跨分片投票、泛光堆叠）。
  - 验收：validate_page --check-indexes 24 文件通过、严格链接 247 处 0 错 0 警、Lua 16 例通过、record 16、覆盖清单 built 630／unbuilt 164；登记 8 主题文件 16 行；highlight 实为 playercontroller 动态挂载（派发单"无消费者"纠偏）。累计 234/398。

- 2026-10-28 持续建页批A-16 十六页（含撞车收编）：klaussackloot／prophider／trophyscale／moonstorms／focalpoint／boatracecrew 六页为并行执行者同日同 build 先行产出——派发前未在覆盖清单标记在途，本批子代理按不覆盖原则确认命中并复验通过，主代理补齐登记＋record＋覆盖清单后收编；lunarriftmutationsmanager／wanderingroutefollower／werebeast／fumaroletool／firefx／guardian／clockworktracker／transparentondrones／forestresourcespawner／hideandseekgame 十页本批新建（月裂突变管理、未完成路线巡游死组件、兽形态切换、气孔热工具、火焰特效、召唤守护者、发条随从计数、无人机淡出、资源再生器、捉迷藏裁判）。
  - 验收：validate_page --check-indexes 27 文件通过、严格链接 0 错 0 警、Lua 8 例通过、record 16、覆盖清单 built 646／unbuilt 148；登记 11 主题文件 16 行；wanderingroutefollower 未定义变量 bug 与占位实现如实记录。自本批起派发前在覆盖清单标 in-flight 防撞。累计 250/398。

- 2026-10-28 持续建页批A-17 十六页：playingcardsmanager／boatcannonuser／stageactor／avengingghost／squidspawner／pumpkinhatcarvable／toadstoolspawner／quagmire_recipeprices／townportalregistry／wavemanager／caveins／quagmire_recipebook／submersible／yotb_stagemanager／sentientaxe／bathingpool（扑克 ID 池、船炮瞄准、舞台演员、变鬼复仇、乌贼生成、南瓜帽雕刻、蛤蟆菌协调、暴食估价与菜谱广播、传送门登记、水面波纹、洞穴落石、物品沉水、牛年舞台、露西人格、泡澡池协议）。
  - 验收：validate_page --check-indexes 24 文件通过、严格链接 324 处＋互链 3 页 0 错 0 警、Lua 12 例通过、record 16、覆盖清单 built 662／unbuilt 132（in-flight 机制首用正常，in-flight→built 转换修复后通过）；登记 8 主题文件 16 行；stageactor performplay 必报错、Debug_ResetAvailableIDs 未定义全局、_symbol 无写入、finshed 原版拼写四处死代码/怪癖如实记录。累计 266/398。

- 2026-10-28 持续建页批A-18 十六页：maxwelltalker／wildfires／yoth_knightmanager／moonbeastspawner／grottowaterfallsoundcontroller／slingshotmods／autosaver／kramped／grue／winch／decoratedgrave_ghostmanager／snowballmanager／yoth_hecklermanager／brightmarespawner／possessedaxe／vaulttorchgrid（休眠台词演出、夏季野火、马年骑士、月亮兽波次、瀑布声控、弹弓改装容器、自动存档、淘气值、黑暗惩罚、绞盘、守墓幽灵、冬宴雪球、吐槽鸟、月狂人口、Lucy 复原、火炬网格）。
  - 验收：validate_page --check-indexes 24 文件通过、严格链接 361 处 0 错 0 警、Lua 15 例通过、record 16、覆盖清单 built 678／unbuilt 116；登记 8 主题文件 16 行；maxwelltalker 休眠遗留、tags 越分类两处代理自纠如实记录。累计 282/398。

- 2026-10-28 持续建页批A-19 十六页：shadowthrall_mimics／uianim／playeravatardata／worldsettingstimer／wx78_shield／shardtransactionsteps／acidinfusible／floater／diseaseable／ghostlybond／yotb_sewer／fumarolelocaltemperature／gingerbreadhunter／wx78_taserbuildup／lunarhailbuildup／hallucinations（影子拟态、UI 补间、头像数据、设置缩放计时、WX78 护盾与电击、跨分片事务、酸液灌注、漂浮表现、疾病传播（无消费者如实记录）、幽灵纽带、缝纫机、地热温度层、姜饼猎捕、月雹堆积、低理智幻觉）。
  - 验收：validate_page --check-indexes 26 文件通过、严格链接 0 错 0 警、Lua 16 例通过、record 16、覆盖清单 built 694／unbuilt 100；登记 10 主题文件 16 行；diseaseable 无 AddComponent 消费者（仅 3 处防御性读取）如实记录。累计 298/398。

- 2026-10-28 持续建页批A-20 十六页：shard_mermkingwatcher／frograin／worldmigrator／vase／quagmire_hangriness／oceanwhirlportalphysics／yotc_racestats／klaussackspawner／maprevealable／anchor／daywalkerspawner／worldtemperature／walkableplatformplayer／messagebottlemanager／yotc_raceprizemanager／skilltreeupdater（鱼人王跨分片、蛙雨、迁移门户、花瓶接收端、Gnaw 饥饿值、漩涡涡流、卡拉特属性与裁判、克劳斯袋生成、小地图标记、船锚、洞穴挖掘者、世界温度、玩家平台跟随、漂流瓶寻宝、技能树端组件）。
  - 验收：validate_page --check-indexes 26 文件通过、严格链接 349 处＋互链 5 页 0 错 0 警、Lua 4 例通过、record 16、覆盖清单 built 710／unbuilt 84；登记 10 主题文件 16 行＋批外反向链 5 条；六处死代码/拼写遗留如实记录。累计 314/398。

- 2026-10-28 持续建页批A-21 十六页：socket_shadow_harvester／malbatrossspawner／itemmimic／worldreset／sinkholespawner／acidlevel／desolationspawner／forestpetrification／boatcrew／gelblobspawner／vinebridgemanager／wintersurprisespawner／deciduoustreeupdater／playerspawner／nightmareclock／yotc_racecompetitor（影子收割、海鸥 Boss、物品拟态、世界重置、地陷袭击、酸雨侵蚀、荒芜再播种、森林石化、海盗船员、软泥怪、藤桥、惊喜树、桦树精、玩家出生、梦魇时钟、赛跑选手）。
  - 验收：validate_page --check-indexes 26 文件通过、严格链接 358 处 0 错 0 警、Lua 16 例通过、record 16、覆盖清单 built 726／unbuilt 52＋in-flight 16；登记 10 主题文件 16 行；boatcrew 运行时挂载纠偏、desolationspawner 双挂载、itemmimic 页补齐 itemmimicconfig 配套互链。累计 330/398。

- 2026-10-28 持续建页批A-22 十六页（全 standard 档）：shadowhandspawner／meteorshower／regrowthmanager／deerherding／dsp／centipedebody／deerherdspawner／playervision／mutatedbuzzardcircler／specialeventsetup／ambientlighting／worldstate／golfclub_reticule／shadowcreaturespawner／ropebridgemanager／grottowarmanager（暗影之手、流星雨、资源再生、鹿群放牧、音频滤镜、蜈蚣体节、鹿群生成、玩家视觉、影鸟盘旋、节日装配、环境光、世界状态、高尔夫瞄准环、制影怪、绳桥、地底战争）。
  - 验收：validate_page --check-indexes 22 文件通过、严格链接 498 处 0 错 0 警、Lua 16 例通过、record 16、覆盖清单 built 742／unbuilt 36＋in-flight 16；登记 6 主题文件 16 行；forest.lua 双挂载纠偏、max_torso 含双头等细节如实记录。累计 346/398。

- 2026-10-28 持续建页批A-23 十六页：miasmamanager／worldcharacterselectlobby／worldroutefollower／mightiness／mermkingmanager／oceantrawler／lunarhailmanager／singinginspiration／beargerspawner／stageactingprop／deerclopsspawner／worldoverseer／dockmanager／shadowparasitemanager／mightygym／riftspawner（洞穴雾气、角色选择大厅、路线巡游、沃尔夫冈体格、鱼人王、海洋拖网、月雹弃用件、战歌灵感、破坏王、查理舞台、独眼鹿、遥测统计、码头、寄生体、健身器、裂隙）。
  - 验收：validate_page --check-indexes 24 文件通过、严格链接 511 处 0 错 0 警、Lua 16 例通过、record 16、覆盖清单 built 758／unbuilt 20＋in-flight 16；登记 8 主题文件 16 行；lunarhailmanager 已弃用无安装点如实记录；worldoverseer 按遥测件归 world-management。累计 362/398。

- 2026-10-28 持续建页批A-24 十六页：hermitcrab_relocation_manager／pumpkincarvable／ambientsound／acidbatwavemanager／inspectaclesparticipant／lunarthrall_plantspawner／clock／colourcube／oceanicemanager／charliecutscene／seasons／penguinspawner／worldvoter／wisecracker／playeractionpicker／mutatedbuzzardmanager（珍珠搬迁、巨型蔬菜雕刻、环境音、蝙蝠潮、观测镜小游戏、月裂隙植物潮、世界时钟、后处理调色、海冰管理、中庭过场、四季循环、企鹅生成、投票踢人、自动播报、动作拾取、秃鹫迁徙）。
  - 验收：validate_page --check-indexes 26 文件通过、严格链接 0 错 0 警、Lua 16 例通过、record 16、覆盖清单 built 774／unbuilt 4＋in-flight 16；登记 10 主题文件 16 行；bat_boss 引用不存在方法、penguinspawner 未定义引用、inspectaclesparticipant 双 LongUpdate 等四处死代码如实记录。累计 378/398。

- 2026-10-28 持续建页批A-25 十六页（大组件批，3 页 complex 档）：rabbitkingmanager／migrationmanager／hunter／wagpunk_manager／shadowthrallmanager／pearldecorationscore／caveweather／snowmandecoratable／piratespawner／quaker／dynamicmusic／sharkboimanager／moonstormmanager／retrofitcavemap_anr／hounded／weather（兔子王、生物迁徙、脚印狩猎、瓦冈朋克、梦魇裂缝、珍珠装饰评分、洞穴天气、堆雪人、海盗刷怪、地震、动态音乐、鲨鱼仔竞技场、月风暴、洞穴地图迁移、猎犬袭击、地表天气）。
  - 验收：validate_page --check-indexes 24 文件通过（首跑抓出 pearldecorationscore 锚点放错主题文件，补登后复检 True）、严格链接 0 错 0 警、无 lua 示例批、record 16、覆盖清单 built 790／unbuilt 4＋in-flight 4；登记 9 主题文件 16 行；批内 9 对互链落地。累计 394/398。

- 2026-10-28 终批A-26 四页交付，398/398 全部建齐：yotb_stager（complex，牛年舞台比赛）／retrofitforestmap_anr（complex，地表老档迁移，记 catcoonden 死旗标）／wagpunk_arena_manager（complex，竞技场 8 态任务线）／playercontroller（complex，玩家输入枢纽精选覆盖主链路，未逐行审校范围页内列行号清单）。validate_page 4/4、严格链接 146 处 0 错 0 警、parser_probe 4 例通过；登记 activity 1＋world-management 2＋character 1；record 4。覆盖清单 794 built／unbuilt 0（23 not-component-page＋4 removed-source 不建页）。
  - 终检全库四件套全绿：validate_page --check-indexes 850 页全过；严格链接 850 页 20,983 处 0 错 0 警；标签一致性 850 页 0 缺陷；哈希基线 4,713 组对比全部一致、affected 0、unrecorded 0、sources_changed 0、删除 0。
  - 事故与恢复记录：全库标签一致性扫描后，一次自写批量"标签同步"脚本有缺陷——把含区间标签的链接重写为错误格式并破坏旧页 en-dash 区间标签（约 100+ 已提交页），后续修复脚本又只修了 ASCII 区间格式。发现后按 git 对账分类恢复：70 页纯污染直接还原 HEAD，10 页混合（含今日有意互链）还原后重放互链，boatphysics/walkableplatform 还原后重放互链＋瘦身（回 16,384 字节预算内）；git restore 按 autocrlf 写出 CRLF 使 3 个贴上限页（oceanfishingrod／simplemagicgrower／squadmember）虚胖超预算，另将 93 个还原页规范化回 LF（与 HEAD 逐字节一致，git status 保持 clean）。恢复后全库四件套复检全绿（见上）。教训：批量改标签必须先备份＋小样本验证；en-dash 旧式标签（amorphous 等页 label 区间≠锚点区间的历史遗留）不在本次同步范围，其是否归一留待后续 lint 任务。

- 2026-10-29 en-dash 标签遗留清零＋复合链接漏括号修复（本代理，无子代理）：①全库扫描发现 23 页 711 个 en-dash 区间标签（其中 77 处数字与锚点错位，如 amorphous 页 hutch.lua:391–395 配 #L395-L399），统一定案"数字同步到锚点＋连字符规范化为 ASCII"后归零；②顺带发现并修复同类历史缺陷——复合证据链接漏闭括号（如 [inventory.lua:408](…#L408、[armor.lua:39](… 缺 ')'，读者侧渲染为断裂 URL），94 页 895 片段全部补 ')' 归零；③随锚点等值折叠 #LN-LN→#LN（4,082 处，上一批脚本把单行锚点写成双写）全库引用计数口径由 20,983 变为 17,860。
  - 过程透明记录：①首版 en-dash 重写脚本有两个缺陷（可选分隔符类漏 ASCII '-' 致区间翻倍；group 拼接致 '](( ' 双括号），当轮即用逆变换全部修复（翻倍折叠 4,007、双括号 4,705、en-dash 孤尾 190），修复后 census/validate/strict/标签四层验证归零，未遗留损伤；②恢复期间 git restore 的 autocrlf CRLF 已随 93 页规范化处理。终检四件套全绿：validate_page --check-indexes 850 页全过、严格链接 17,860 处 0 错 0 警、标签一致性（含 dash-agnostic 复核）0 缺陷、基线 4,713 组全部一致。遗留预检：prose 中的裸 '#L 行号'（非链接）与全角锚点不在此修范围。

- 2026-10-29 组件主题索引按需求重新分组：整理 13 个主题页中集中追加的条目，保留既有标题锚点及每条登记原文，更新组件总索引的需求摘要；不修改知识页或源码结论。以 steam-buildid:26230496 现有页面为依据。
  - 19 个组件主题索引与全库导航校验通过，823 篇知识页登记检查无问题；季节生物、天气灾害、音频演出三个新章节的查询读取完整且未截断。仅验证索引结构和读取，未重新审校组件源码或运行游戏。

- 2026-10-29 Prefab 首批四页：[[reference/prefabs/heatrock]]、[[reference/prefabs/backpack]]、[[reference/prefabs/icebox]]、[[reference/prefabs/krampus]]，基于 Steam buildid 26230496 核验工厂、一层关联、必要消费者及 chinese_s.po 名称键；“小偷”按用户提供收录，其他额外俗称待补充。新建 [[reference/prefabs/index]] 并接入 reference／根导航。
  - 验收：7 文件页面／导航通过；4 页 153 处 Lua 引用严格 0 错 0 警，4 个官方译名条目另行逐键核对；暖石分档、背包穿脱、冰箱保鲜冷却、坎普斯偷物四项章节读取完整、两组共用预算均未截断；仅 record 新四页，37 组来源 check 全部一致，无未登记或 sources_changed。
  - 限制：未运行游戏、未验证资源包内容，无新增 Lua 示例；Brain／SG 公共状态、皮肤全集和其他模组兼容仅保留页内声明范围。未改 raw、未重建查询缓存，保留其他执行者的未提交增量；不将本批机械验收扩展为全库语义结论。

- 2026-10-29 用户确认暖石、背包、冰箱无额外俗称，移除页面／索引的待补提示。复核四页工厂显式、helper 间接及条件 FX 组件的 Wiki 关联，补齐 icebox_placer 的 placer 组件入口；不递归审校组件内部依赖。3 个改页格式／链接通过，71 处 Lua 引用严格零错警；2 篇知识页 20 组来源一致，sources 未变，不刷新基线。

- 2026-10-29 Skill／Prefab 皮肤与强关联收口（Steam buildid 26230496）：Skill 类型指引集中新增组织规则、模板补一处提示；首批四页补暖石三皮肤共用路径、背包 labrat 钩子、冰箱五皮肤生成／清理映射，并拆清坎普斯掉落与尸体依赖。皮肤信息留主体，简单 FX 不默认建独立页；Skill 基础校验、模板 YAML 与 7 处本地引用通过。
  - 新建 [[reference/prefabs/cane]]、[[reference/prefabs/yellowstaff]]、[[reference/prefabs/opalstaff]]、[[reference/prefabs/stafflight]]、[[reference/prefabs/staffcoldlight]]；手杖七种皮肤对应表、召唤物双向入口、月台 staffcoldlightfx 同页条件变体落地。“手杖／加速手杖／唤星／矮星／唤月／冷星／极光”按用户给定关系收录，索引区分施法物品与生成实体。
  - 验收：10 文件格式／全库导航通过（832 知识页／28 索引）；9 页 337 处 Lua 引用严格 0 错 0 警；实际引用均在所列 sources 内，新增对象的工厂／helper 组件均有 Wiki 入口。8 个章节分四组按默认预算读取均完整；重叠别名保留双候选。record 新增 5／刷新来源扩展页 3，9 页 check 82 组全部一致。
  - 未运行游戏，无新增 Lua 示例；未覆盖 FX 工厂全集、全部重皮肤时序、月台完整充能防守流程或资源包内容。未启动子代理、未重建查询缓存，保留其他执行者改动，未提交或推送；源码只读。

- 2026-10-29 持久化 FX 补链：Skill 现有 Prefab 规则追加新页按注册名反查主体待建关系、核验后补链；K/PLAN.md 登记手杖 7 项与冰箱 5 项缺口及主体路径，包含拐杖糖雪花检索描述待补。12 项均能反查对应主体，Skill 基础校验和两仓 diff 检查通过；未创建 FX 页、未修改知识结论或来源基线。

- 2026-10-29 Prefab 第三批（Steam buildid 26230496）：新增 [[reference/prefabs/cane_candy_fx]]、[[reference/prefabs/cane_rose_fx]]、[[reference/prefabs/cane_harlequin_fx]]、[[reference/prefabs/cane_sharp_fx]]、[[reference/prefabs/cane_victorian_fx]]、[[reference/prefabs/cane_ancient_fx]]、[[reference/prefabs/cane_chess_fx]] 七页，含编号／形色变体共 22 个注册；定向核验工厂、皮肤映射／名称、主体调用与必要粒子 helper／清理，未递归展开引擎内部。
  - [[reference/prefabs/cane]] 与 [[reference/prefabs/index]] 回补真实链接及雪花／花瓣／彩纸／紫烟／闪光／影子／棋子检索词，PLAN 将手杖组销项，冰箱 5 项保留待建。9 文件格式／全库导航通过（839 知识页／28 索引），8 页 225 处 Lua 引用 strict 零错警；来源声明与 7 组双向链接核对通过，7 个效果词命中主体及对应产物，9 个章节分四组按默认预算完整读取。
  - 基线生成器仅 record 本批新增 7 页及来源扩展的主体 1 页，check 47 组全部一致。未运行游戏、未解包验证资源、无新增 Lua 示例，未作独立模型评测；未改 raw、未重建查询缓存，保留其他执行者改动，未提交或推送。

- 2026-10-29 Prefab 类别标识：Skill 标签表新增 fx，要求按对象自身职责标物品／建筑／生物／特效并在快速判断说明，区别 Wiki 与游戏实体标签；模板同步类别提示。核对现有 16 篇 Prefab，给 7 篇手杖 FX 补 fx，冰箱保留 building、移除 item；物品／生物标签及两种召唤实体的正文区分沿用。8 改页格式／链接、Skill 基础与两仓 diff 检查通过；仅改分类，未改 sources／基线、未重建缓存，未运行游戏、未提交或推送。

- 2026-10-29 Prefab 检视收口（Steam buildid 26230496）：[[reference/prefabs/icebox]] 修“动画 bank”，[[reference/prefabs/heatrock]] 保留第五档启 bloom／灯光并补灯强公式；7 篇手杖 FX 补五种粒子上限、紫烟／闪光排序、仅紫烟显式 KillOnEntityDeath，以及两种动画拖尾不适用说明；[[reference/prefabs/cane_ancient_fx]] 补随机值门禁与本地动画禁休眠，未据此推定晚加入／离屏表现。16 篇 Prefab 的 43 条陷阱明确标误区／边界，雪花页改为 SetMaxLifetime 只约束粒子；FX 案例公共引言补主体／清理前提。
  - 工具箱 query_wiki.py 的 read_chapter 直接复用 sections 的标题归一，原文／规范锚点返回规范章节，未限定重名标题报歧义；33 项既有回归、12 项不写文件的内存检查和 4 个真实章节原文读取通过。Skill 在现有写作条款仅补两句可复用原则；未另写一套 slug 算法。
  - 验收：17 文件格式／全库导航通过，16 页 544 处 Lua 引用 strict 零错警，122 组来源基线一致；sources 未改，不刷新基线。缓存仍 62 篇知识页／当前 884 个 Wiki Markdown，search 返回 stale，检索验收走 grep；未重建缓存、未改 raw、未运行游戏、无新增 Lua 示例，未提交或推送。可选 7～9 项未做，保留其他执行者增量。

- 2026-10-30 术语与 FX 标题收口（沿用 Steam buildid 26230496）：[[reference/components/beaverness]] 将 currentwereness 明确为 player_classified 上的 net_byte 并补定义／读取证据；[[reference/components/groundcreep]] 将含混直译替换为玩家同步实体及 OnPlayerFadeDirty 的实际淡入条件，均只作局部补核。七篇手杖 FX 标题／H1 改为中文效果描述＋注册名，[[reference/prefabs/index]] 同步；正文注明非官方译名，文件地址不变。
  - Skill 合并术语与描述名规则，模板只同步名称提示。10 文件格式／全库导航、9 页 222 处 Lua 引用 strict 零错警、七组标题／H1／索引与非官方译名说明一致；beaverness／雪花两个章节按默认预算完整读取，中文标题定位验收走当前 Markdown／grep，未重建陈旧缓存。
  - beaverness 既有两份源码原始哈希先与旧基线核对一致，再由生成器仅刷新该页新增 player_classified.lua 的来源记录；9 页 41 组 check 全部一致。未改 raw、未运行游戏、无新增／改动 Lua 示例、未提交或推送，保留其他执行者的原文与增量；定向修正不代表全库语义重审。
