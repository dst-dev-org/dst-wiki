# DST Wiki 建设计划

2026-10-13：四仓本地迁移已完成；工作重心转为完善 dst-wiki-skill 的安装、查询、知识生成／维护与真实使用闭环，再用该 Skill 补齐 Wiki。组件批量生成继续暂停，原队列保留；Skill 详细工作项与验收只在 [Skill 完善计划](../dst-wiki-skill/PLAN.md) 维护。分仓依据见 [决策](docs/decisions/repository-split.md)。

仅工程建设、工作项推进与进度请求按需读本文件；查询、知识页增修／审校和索引整理不加载或同步。本文件只跟踪工作与验收；总体路线见 [项目设计](docs/design.md)，取舍见 [决策索引](docs/decisions/index.md)，执行规则见 [工程维护](references/project-maintenance.md)。下列未完成能力不能当作可用工具。

最后更新：2026-10-13。`[x]` 已完成，`[ ]` 待执行。目标：服务新增内容（物品、装备、角色、生物、植物、建筑／配方）与修改已有机制两类任务，用 cookbook 骨架、机制知识、轻量索引和按需源码回查减少重复读取；实际 token 收益尚未测量。

W01–W04 已完成；W05 进行中，已交付可重建知识索引、有限候选检索、真实章节读取及标签／导航／Top 30 检查，检视见 [检索首轮](../dst-toolbox/docs/reports/w05/retrieval.md)。首篇 [模组起步与加载验证](wiki/cookbook/mod-starter.md) 已交付并登记场景导航；已只读梳理三个角色模组，后续骨架范围见下表。已补 inventoryitem 与 24 篇关联组件页，并交付 [新增可拾取物品](wiki/cookbook/new-item.md)；当前知识共 48 篇组件页、3 篇 modinfo 与 5 篇 cookbook；[装备导航](wiki/cookbook/equipment/index.md) 已交付手部、[身体护甲](wiki/cookbook/equipment/body-armor.md) 与 [头部](wiki/cookbook/equipment/head.md)，梳理见 [手部](docs/reports/w05/hand-equipment.md)／[身体](docs/reports/w05/body-armor.md)／[头部检视](docs/reports/w05/head-equipment.md)。两项新增／修改试点已有隔离代码、源码／语法检查及读取量重放，见 [任务检视](docs/reports/w05/tasks.md)；同一会话受控重放不代替独立模型评测。Skill 基础使用闭环验收后，用安装后的 Skill 恢复 [组件 Wiki 分批建设](#后续队列组件-wiki-分批建设)，前五批三维／战斗、温度／环境、制作／交互、成长／生物协作与计时／持续效果已交付，下一批 moisture／moistureimmunity／freezable；配方／建筑及其他 cookbook 顺延，任务与暖石组合验证继续保留；游戏运行与准确 token 收益仍未验证。前阶段见 [类型与知识页验收](../dst-toolbox/docs/reports/w04/acceptance.md)，环境唯一来源见 [环境说明](../dst-toolbox/references/environment.md)。

贯穿各阶段的交付要求：按 [工程维护](references/project-maintenance.md) 随项目版本核对并同步 [总体设计](docs/design.md)，保持架构、能力状态与限制符合实际交付；不把设计维护当作一次性完成项，详细进度仍由本计划跟踪。

基础使用要求：只查询或更新常用案例／教程、场景 bug 触发点／陷阱的用户，通过 Markdown 搜索、读取与编辑即可操作，不要求 Python、Rust、LuaLS、fork 克隆或全量提取／索引。Python 源码索引是可选增强，缺失时定向搜索源码；PowerShell 源码准备不依赖 Python／Rust，也允许按目录约定人工准备。源码准备是查询与审校的共同兜底，不因工具箱拆分而取消。工程提取、批量校验与索引重建按工具另备依赖。

## W04：核验类型并生成注解与 Wiki

已完成（2026-10-11）。完整范围与限制见 [两份试点验收](../dst-toolbox/docs/reports/w04/acceptance.md)，前轮状态保留在 [finiteuses 首轮检视](../dst-toolbox/docs/reports/w04/finiteuses.md)。

- [x] finiteuses 8 个字段／19 个实际方法、equippable 17 个声明及属性字段／18 个实际方法的必要类型、参数省略／nil、返回分支与行为有逐项源码依据；分别 83／93 条正式 reviews，共 176 条。推荐原版调用域与默认值分开，不推广为任意模组写入或全库准确。
- [x] 属性回调、真实继承、事件／装备顺序、宿主清理及原版消费者核验；两个试点均不继承公共编辑器 component_base，实际钩子显式声明，未知外部动态字段／依赖模型仍保留边界。
- [x] 同一 reviews + facts 经现有契约合并导出两份 LuaCATS，拒绝 stale／冲突／错误证据，不造 any；76／80 条投影输入，零缺失 facets，再导出与维护声明原字节一致。
- [x] LuaLS 3.19.1／Lua 5.1 两份声明／正例无诊断，finiteuses 三处及 equippable 六处故意错误准确命中；入袋 owner 明写 ent|nil，实测回调问号语法未命中 nil 错误的消费者差异归档。不是全库／游戏验证。
- [x] 两份组件 Wiki 共用核验结论，包含斧头次数与手杖装备速度／精神接入、证据、条件和未运行边界；按轻量模板组织，生命周期与原版参照按需要归入正文，登记分类及根人工精选入口。
- [x] 新页／全部 Wiki 元数据、链接／章节及唯一行数／字节阈值检查通过；98 项 Python 回归通过。完整标签／叶级一致性／Top 30 自动检查继续留 W05，不为最低检查扩大范围。
- [x] fork 提交并发布 1db05561ea6ea271093ea780b60d49e7499baf40，主项目 pin 更新，setup／check 可复现固定输入；覆盖／来源／注解哈希和两份局部 LuaLS 记录随 fork 保存。主项目暂存区不改，未提交主仓。
- [x] 无可见 Lua 实现的引擎内部及游戏运行需求明确留缺口；两个页面保持 source-reviewed，不声称多人网络／预测、存档／重载或模组兼容实测。

验收：所采纳必要类型、两份 Wiki 与声明同源；未知和冲突不被隐藏；固定发布版可取回。游戏运行不作为本阶段源码／类型试点已执行项，需实际测试的场景留 W05。

## W05：真实方案与检索闭环

首轮（2026-10-11）：清理 3 个已有内容目录的 `.gitkeep`，保留现已迁入 `reference/api/engine` 的占位；工具归 `scripts/tools/wiki/`，未新增骨架空页。人工 Markdown 决定分类／摘要／Top 30，`update_index.py` 检查导航并生成机器缓存，不自动重写导航；原“生成分层导航”调整为“维护人工导航＋构建校验”，避免排序覆盖人工主题。详细结果和限制见 [检视](../dst-toolbox/docs/reports/w05/retrieval.md)。

- [x] 首轮已生成一个可堆叠石片新增任务与普通斧头 200 次修改任务的隔离模组，核对源码、Lua 5.1 语法和显式结构，归档实际读取路径／重放返回量；游戏未运行，见 [任务检视](docs/reports/w05/tasks.md)。
- [ ] 扩展到 6～10 个真实任务，分别记录生成正确性、模型自主路由和必要运行结果；本轮同一作者共享上下文，不当作独立模型成功率评测。
- [ ] 纳入 [随身暖石组合场景](docs/decisions/layered-index-and-retrieval.md#组合需求与章节检索)：按需从道具骨架进入，用原版参照补温度／灯光／精神能力，章节候选共享总预算；分清物品／热源／人物 35℃、速率与净变化、主栏／背包／手持、多份叠加，验证共用持有者与转移／移除／重载／死亡清理。当前仅源码设计审阅，未生成模组或运行验收。
- [ ] 将机制方案、案例与教程写最相关具体页的“典型接入”，提供条件、选择理由、入口、影响范围及必要完整实现；跨页机制只维护一份，cookbook 通过扩展链接复用。
- [ ] 常见坑从已核验限制或真实问题提炼，放相关页面的“常见陷阱与限制”章节；共享问题维护一份并互链。源码警告只提供候选，不能为了数量自动生成“常见”结论。
- [x] 实现 `scripts/tools/wiki/update_index.py`，校验人工分层导航并生成独立、可重建的知识索引；元数据／正文提取别名、明确标识符／条件和章节范围，保留 cookbook 类型。登记／父入口、大小和标签通过才替换缓存，不覆盖人工摘要／分类／Top 30；排除非知识材料，不依赖 AST／fork。
- [x] 两份技术页已维护根人工精选 Top 30；自动检查数量／有效链接与锚点／active 状态／去重、叶级登记和根大小，快捷入口不替代登记，不要求凑满或读全部目标。
- [x] 实现 `scripts/tools/wiki/query_wiki.py`：名称／路径、标识符、摘要／别名／标题和明确条件作轻量匹配；每轮共用默认 5 个候选／16,000 UTF-8 输出字节，摘要有上限。返回类型／需求／条件／状态／核验版本／程度／页面级证据与章节入口，歧义、截断、词汇覆盖与缓存增删改失效可见；工具内过滤 JSON，源码证据增量失效留 W06。
- [x] `validate_page.py` 补标签、叶级登记／父入口／循环、Top 30 数量／去重／active 和现有链接／锚点检查；指定草稿仍可仅校验自身。结构、源码语义与游戏验证分开；工具按用途归位。
- [x] Wiki 归为 cookbook 任务骨架与 reference 机制参考；组件／Prefab／状态机／API 迁入 reference，modinfo 三页聚合主题目录。同步当前链接、Skill、检索／校验分类及缓存，保留章节直达、源码证据和唯一叶级登记；索引增加不成为必读步骤。
- [x] 明确 [开发任务读取设计](docs/decisions/layered-index-and-retrieval.md#开发任务的读取顺序)：本地实现／约定优先；新增内容或未知组装用 cookbook → reference → 源码补缺口，局部修改／已知机制直达，mod-starter 仅用于无入口项目。查询规则只链接场景导航，具体骨架主入口由所属索引维护；工具不自动执行回退，真实模型行为仍待验证。
- [ ] 首轮已重放已有入口新增、局部斧头修改与机制页缺组装的回退；继续用空目录起步、复杂新增、局部机制修改及骨架／扩展部分缺失的真实任务验证上述路由；记录实际读取路径，检查不重读起步页、不全量展开扩展、保留可复用部分。继续验证名称直查、需求筛选与分层浏览，覆盖大分类、索引遗漏／失效、重名、无命中和截断；查不到时只对缺口补 Wiki／源码证据。
- [ ] 首轮两项编码已作三路／两路受控读取比较；继续对同一组查询、编码与案例／场景问题维护任务比较直接查源码与 Wiki 路径：命中、正确性、源码量、读取文档／章节／字符、工具调用和上下文 token；区分一次性建设成本与重复使用收益。
- [x] 首轮明确使用完整输出的 UTF-8 字节／字符及所选源码行数代理，计入搜索、封装、截断补读和回退；共同前置与建设／验收成本另列，未测准确 token，不预设节省比例。后续每轮继续沿用此口径。
- [ ] 需要游戏运行确认的场景记录实际测试环境、预期与结果；未执行保持未验证。

### 后续队列：组件 Wiki 分批建设

2026-10-12 确认原内容队列：在已有 27 篇组件页与五篇 cookbook 的基础上，分批补齐组件知识，再扩展配方／建筑等骨架。2026-10-13 调整：该队列保留，先完善 dst-wiki-skill 并验收基础使用闭环，再通过该 Skill 恢复建设。前五批已交付，当前共 48 篇组件页；检视与边界见 [三维与战斗首批](docs/reports/w05/component-core.md)／[温度与环境](docs/reports/w05/component-environment.md)／[制作与交互](docs/reports/w05/component-crafting.md)／[成长与生物协作](docs/reports/w05/component-growth-following.md)／[计时与持续效果检视](docs/reports/w05/component-timed-effects.md)。批次按开发功能组织，知识页仍按具体组件维护；不新增批次知识目录，不把全量组件清单塞入根索引。近期拟定至第十五批；750 个未建候选已按名称完整归组，范围快照见 [剩余组件路线](docs/reports/w05/component-roadmap.md)。下文只留执行队列，逐组件当前状态以 [工程覆盖清单](docs/reports/w05/component-coverage.csv) 为准；归组不是职责／依赖已核验。

#### 准备与覆盖清单

- [x] 2026-10-12 运行源码准备，恢复本机 Steam buildid 25643519 的 4,030 个 Lua 文件，原 BUILD-only 目录保留到 raw/archive/build-25643519；86 个既有基准原字节锚点哈希匹配。本批证据重新核验，未重生成全量 facts，也未将同版本或基准匹配当全部旧页已审校。
- [x] 建立 [工程覆盖清单](docs/reports/w05/component-coverage.csv)：组件目录 821 个 Lua 文件，分为 798 个组件候选、19 个 replica、3 个辅助模块、1 个空文件；第一批后已建 32 页／766 个未建候选；第二批后 35 页／763 个未建；第三批后 40 页／758 个未建；第四批后 45 页／753 个未建；第五批后 48 页／750 个未建。记录来源／哈希、批次、页面／状态与已核验范围；候选身份待逐页核验，清单不进入知识检索或查询必读链。现有武器／护甲／食物／装备页只补必要互链，不重生成。
- [x] 2026-10-12 仅核对文件名与既有覆盖分类，盘点全部 750 个未建候选：近期 41、扩展主题 470、专项 236、职责待确认 3；目录／CSV 一致，名称无重复或遗漏。完整分组与 19 replica／3 helper／1 empty 的独立安排见 [路线快照](docs/reports/w05/component-roadmap.md)，不进入知识导航，不细读实现，不改变覆盖状态。
- [ ] 用清单持续安排余下批次，保留已有页面及维护经验，不覆盖重生成；“已建页”只表示注明范围已核验，不表示所有方法、全组件类型或游戏运行都已审校。待建页不提前进入知识导航，也不建立占位 demo。

#### 批次与近期队列

| 批次 | 拟补组件 | 支撑的开发需求 |
|---|---|---|
| 1：三维与伤害（已交付） | `health`、`combat`、`hunger`、`sanity`、`eater` | 修改三维、回血、伤害与进食收益，补现有武器／护甲／食物／装备页的核心协作；源码／语法／结构与章节读取通过，游戏未运行 |
| 2：温度与环境（已交付） | `temperature`、`heater`、`sanityaura`，其余按源码盘点补充 | 暖石自身锁／读档、三个热源通道和精神光环；明确范围／单位／去重，源码／语法／结构与章节读取通过，游戏未运行 |
| 3：制作与交互（已交付） | `builder`、`workable`、`lootdropper`、`burnable`、`propagator` | 配方学习／缓冲、工作完成／掉落、燃烧计时／传播及清理；源码／语法／结构与章节检视通过，游戏未运行 |
| 4：成长与生物协作（已交付） | `pickable`、`growable`、`leader`、`follower`、`locomotor` | 采收再生／阶段成长、共享表隔离、忠诚／关系与实际移动分工；源码／语法／结构及章节检视通过，游戏未运行 |
| 5：计时与持续效果（已交付） | `timer`、`debuff`、`debuffable` | 具名计时／刷新／暂停、效果来源／撤销与同键、施加门禁／返回／存档重施；源码／语法／结构与章节检视通过，游戏未运行 |
| 6：潮湿与冰冻 | `moisture`、`moistureimmunity`、`freezable` | 补现有防雨／食物效果的目标端湿度、来源免湿与冰冻累积／解冻；具体接口与场景待逐页核验 |
| 7：工作效率与消耗倍率 | `efficientuser`、`worker`、`workmultiplier` | 补现有 tool／workable／finiteuses 的效率与消耗选型；职责／接口待核验 |
| 8：睡眠与困倦 | `grogginess`、`sleeper`、`sleepingbag`、`sleepingbaguser` | 睡眠状态与物品／使用者分工，核验唤醒及撤销；职责／接口待核验 |
| 9：治疗与修复 | `healer`、`maxhealer`、`repairable`、`repairer`、`sewing` | 补恢复物品、上限恢复、修复材料与装备修补；职责／接口待核验 |
| 10：投射物与爆炸 | `complexprojectile`、`explosive`、`projectile` | 远程武器／效果对象的基础机制；职责／接口待核验 |
| 11：定向技能与瞄准 | `aoecharging`、`aoespell`、`aoetargeting`、`cooldown`、`reticule`、`spellbookcooldowns` | 补现有 spellcaster／spellbook／rechargeable 的施放与冷却选型；职责／接口待核验 |
| 12：科技与建造接入 | `craftingstation`、`deployhelper`、`placer`、`prototyper` | 为配方／建筑骨架补必要机制；注册 API 另记缺口；职责／接口待核验 |
| 13：烹饪与干燥 | `cookable`、`cooker`、`dryable`、`dryingrack`、`stewer` | 食物加工与配方判定分工；料理注册 API 另记缺口；职责／接口待核验 |
| 14：打包与拆包 | `bundlemaker`、`bundler`、`unwrappable` | 物品集合打包、资源与存档恢复的组装；职责／接口待核验 |
| 15：传送与距离触发 | `entitytracker`、`playerprox`、`positionalwarp`、`teleportedoverride`、`teleporter` | 传送对象、位置恢复、接近触发与关联实体记录分别核验；职责／接口待核验 |
| 后续：剩余主题池 | 470 个扩展主题候选、236 个专项候选、3 个职责待确认；见 [路线快照](docs/reports/w05/component-roadmap.md) | 农作／生物及物品机制扩展 → 玩家宿主／世界／网络／海上系统；角色／活动／区域／Boss 等专项按需求拆小批 |

- [x] 第一批五页已完成：精选接口、上限／收益／伤害、来源与撤销、事件／生命周期、案例及未覆盖范围；六段发布 Lua、两项隔离组合接入、49 份知识／导航检查、七次查询与八次章节读取通过。保留 source-reviewed，未扩大到正式类型数据或游戏测试；详情见 [首批检视](docs/reports/w05/component-core.md)。
- [x] 第二批三页已完成：赋温／自然锁与读档、热源通道／携带范围、精神光环单位／衰减／去重和效果选型；三段发布 Lua、两份隔离接入、52 份知识／导航、六次检索／七次章节读取检查通过。保持 source-reviewed，完整随身灯光／精神组合及运行验收仍保留，模板不增加必填项；见 [温度与环境检视](docs/reports/w05/component-environment.md)。
- [x] 第三批五页已完成：配方注册／学习／实际制作分工、工作量与完成位置、多来源掉落、燃烧计时／清理、闪点／传播／火伤门禁；五段发布 Lua、两份隔离组合、57 份知识／导航、七次检索及七项章节读取通过。另两次合并章节读取均在默认预算内，复用元数据与交叠区间；见 [制作与交互检视](docs/reports/w05/component-crafting.md)，页面保持 source-reviewed，模板不增加必填项。
- [x] 第四批五页已完成：采收门禁／回调／再生、阶段成长／暂停／离屏／共享表、关系登记与忠诚倍率、随从清理与 Brain／移动协作；五段发布 Lua、两份隔离组合、62 份知识／导航、七次检索与八项单独章节读取通过。植物四章节合并完整，随从四章节合并出现整项省略，拆分补读；忠诚倍率的重连恢复副作用明确可见，见 [成长与生物协作检视](docs/reports/w05/component-growth-following.md)。保持 source-reviewed，不增模板必填项。
- [x] 第五批三页已完成：具名计时与刷新／暂停推进、效果回调／来源撤销、实体与组件施加差异、同键及存档重施；三段发布 Lua、两项隔离接入（共三文件）、65 份知识／导航及八项单独章节读取通过。七次检索保留宽泛词额外召回／候选截断，四章节 Buff 合并省略管理器页，拆两次补齐；见 [计时与持续效果检视](docs/reports/w05/component-timed-effects.md)。保持 source-reviewed，不增模板必填项。
- [ ] 下一批 moisture／moistureimmunity／freezable，先核验实现、必要人物／装备／效果消费者，再生成三篇页面；随后按上述方向和实际依赖／缺口调整顺序，每批通常 3～6 篇。复杂组件可独立小批或按真实主题拆页，不为凑数量降低核验深度；不按源码文件顺序机械转 Markdown。
- [ ] 持续执行后续批次，完成每批后更新覆盖清单并确定下一批；本文记录阶段进度，逐组件状态留清单。若新的快照改变组件范围，记录新增／删除／待复查，不虚报完整覆盖。

#### 后续阶段与安排原则

- [ ] 第 6～15 批共 41 个候选，默认延续表中顺序，真实任务的必要依赖或缺口可提前插入；执行前核验身份和消费者，再决定案例／拆页。复杂组件独立小批，不靠文件名锁定接口或硬依赖。
- [ ] 之后从扩展主题池继续拆 3～6 篇小批：先农作／肥料／种植、生物生成／巢穴／驯养与物品／战斗／交互扩展，再推进玩家宿主、显示／HUD、世界、分片与船体／捕鱼。主题池是范围，不是一个整批；每轮只读所选记录，不加载全部路线或源码。
- [ ] 角色专属、活动／模式、区域、Boss／群落、小游戏与其他专项保留全量登记，按真实需求或覆盖轮次推进；职责待确认三项先判定宿主和调用再归组。核验后调整优先级／身份，不以名称认定通用、废弃或可直接复用。
- [ ] 按真实用途控制内容：750 是待核验文件数，不是预定 750 篇等长教程；通用接入页保留精选接口和必要完整案例，内部／专用协作说明宿主、边界与原版入口。replica 随客户端需求核验，helper／empty 单独处理；不强造 demo、空页或重复机制。
- [ ] 组件批次需要动作／Brain／SG、EntityScript、RPC／同步、配方注册、Light／资源／UI API 时记录并补实际主题；不把补齐组件当作整个编码能力已完成。现有页按缺口增修，继续真实任务与游戏／token 验证，不等全量完成。

#### 每批交付与验收

前五批已按以下标准完成源码／页面、模板复查、登记／检索、每批两项需求内容检视及进度同步；模板不增必填项。未做独立模型评测或游戏运行，实际 token 收益未测；以下保留为后续每批持续要求。

- [ ] 页面优先回答“何时使用、关键接口、条件／副作用、典型接入、生效／撤销与证据”，核验实现、必要消费者及宿主路径；按需要补存档、同步、生命周期和相近组件区别，不全量展开内部字段／方法或复制源码。采用现有 [组件模板](../dst-wiki-skill/assets/templates/component.md) 与 [页面格式](../dst-wiki-skill/references/maintain/write-format.md)，未知和未覆盖范围可见。
- [ ] 每批复查模板：只有真实页面／编码任务暴露共性遗漏时才补模板或格式规则，保留可选章节，不强制改写所有旧页；组件 Wiki 建设不自动触发正式 reviews／LuaCATS／fork 全量审校。
- [ ] 新页唯一登记所属叶级，相关已建页按需互链；分类变大时按现有大小与索引规则拆分，不复制到父级／根全量列表。校验元数据、链接、章节、大小与登记；工具可用时验证实际检索和章节输出，不因缺 Python 阻断普通 Markdown 查询／增修。
- [ ] 每批用 1～2 个新增／修改任务检视所用接口、组装上下文与清理是否齐全，记录实际读取链、重复源码回查、输出量与未解决范围；不能取得准确 token 时使用明确代理，不以页数或关键词命中证明收益。所需运行验证分别记录，未运行保留 `source-reviewed`，不因此宣称 W05 完成。
- [ ] 同步覆盖清单、受影响索引与项目能力／进度说明，追加日志；保持源码只读、已有经验与历史记录。验收以所声明场景能取得必要依据为准，全量覆盖进度与关键任务正确性分别报告。

配方、建筑及其余装备／角色 cookbook 保留后续队列；先完成 Skill 基础使用闭环，再按当前任务安排恢复组件批次。不把全量组件全部写完设为 W05 任务检视／游戏验证或单个必要骨架交付的硬前置；新增骨架仅在具体任务确有组装缺口时插入。

### cookbook：通用任务骨架

详细范围与归属见 [cookbook 决策](docs/decisions/shared-and-mod-wiki.md#cookbook-层的范围与建设)。2026-10-11 起步、新道具、手部、身体护甲与头部五篇已交付；以下按组装差异规划，未标已交付的文件名仍为计划。导航只登记实际交付内容，模组入口梳理不能代替原版 API 核验或游戏测试。

- [x] 编写 `mod-starter.md`：两份完整入口文件、所选全局读取代理、运行端选择及加载检查；创建 `wiki/cookbook/index.md`，首条登记起步页，补根分类和人工精选入口。按最新反馈删除主 demo 的测试 print／空 world 回调，使用游戏加载器日志检查；源码、Lua 5.1 语法／原字节往返、页面／登记及实际章节读取检查通过，游戏未运行。
- [x] 新增 [modinfo 元信息与配置](wiki/reference/api/modinfo/configuration.md)，收录配置项／数值生成、分组标题、GetModConfigData、本地开发名称与语言选择；起步页只提供章节链接，扩展按需加载，不增加必读 cookbook 或完整配置框架。
- [x] 只读梳理 DSTKisaki、MCW、Sora 的入口、角色初始化及代表模块，将重复组装需求归入下列计划；保留各项目原文件，不复制其实现或美术资源，不把项目私有框架当成 DST 通用 API。
- [x] 在道具骨架之前补 [inventoryitem](wiki/reference/components/inventoryitem.md)：核验拾取入口、收纳标志／图标、owner、回调与事件顺序、移出／删除，区分 inventoryitem／container／equippable；用 MCW 宝石箱检验组合边界，技术依据回查原版。交付精选 Wiki 与登记／章节检索，未作全组件类型审校或 fork 注解更新，游戏未运行。
- [x] 补齐 24 篇物品关联组件页及 [new-item](wiki/cookbook/new-item.md)：必要文件、注册／名称、网络分界、原版外观／图标及拾取／检查骨架；148 行／10,059 字节，源码、资源可用性、语法、索引和章节读取检查通过，游戏未运行。
- [x] 将统一 `new-equipment.md` 改为装备主题短导航与按场景拆页；交付 [手部骨架](wiki/cookbook/equipment/hand.md)，只读复用三模组原始哈希匹配的 facts，按工厂／回调梳理模式，通用 demo 回查原版 cane。详情见 [手部检视](docs/reports/w05/hand-equipment.md)；游戏未运行。
- [x] 索引校验支持本目录独立页与子主题入口共存，仍拒绝父级接管子目录页和重复登记；补回归及实际检索／章节读取，保持每页唯一登记，不为起步／道具加空层级。
- [x] 交付 [身体护甲骨架](wiki/cookbook/equipment/body-armor.md)：先梳理三个模组的身体候选与四组 BODY 护甲，再核验原版木甲、吸收／受击耐久、耗尽卸装与存档链；短节说明可省穿戴图，保留功能与清理。资源、语法、登记与章节读取通过，游戏未运行；见 [身体检视](docs/reports/w05/body-armor.md)。
- [x] 交付 [头部骨架](wiki/cookbook/equipment/head.md)：梳理四组显式 HEAD 与两组 NECK or HEAD，核验草帽命名／资源、普通帽显示与卸装恢复、精神消费者；露顶／全包头盔、goggles 功能与外观区别按需链接，短述完整省图边界。资源、Lua 5.1 语法／字节往返、登记与章节读取通过，游戏未运行；见 [头部检视](docs/reports/w05/head-equipment.md)。
- [ ] Skill 闭环验收后按任务安排恢复组件批次；后续 cookbook 先 `new-recipe.md`、再 `new-building.md`，必要机制先补；背包／服装／护符沿下述顺序按需求交付。单个组装任务可按实际缺口插入，不等待全组件或六类装备全部完成。
- [ ] 每份骨架提供所声明场景的必要文件、加载位置、代码、资源及替换点；列明确的前提和验证范围。取得不到必要资源或证据时报告缺口，不以片段或未核验代码充当完整可用 demo；游戏验证记录真实环境和结果。
- [ ] 扩展入口只链接已有且相关的机制章节；缺失机制先核验补足，不造悬空链接。不复制接口清单或完整机制案例，必要共用代码明确唯一维护来源和文件取得方式，验证读取少量链接即可取得组装上下文。
- [ ] 在首批试点后按下表扩展角色、容器及组件／技能等场景；食物复用道具基础，生物／植物按实际行为与生长需求细化，不把一个 demo 宣称为所有类型的通用实现。
- [ ] 用同一新增任务比较直接查源码、仅机制页、cookbook＋所需机制章节的正确性／读取量／工具调用与 token；验证机制查询和已有内容修改能跳过 cookbook，已有项目专属骨架优先适配，不强制加载模组 Wiki Skill、双写知识或安装 Python。

#### 后续章节与推进顺序

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

上述批次为内容覆盖路线，不要求 W05 一次写完全部章节；W05 先完成首批及真实任务闭环，后续按需求推进。共用起步页只在目标项目尚无入口时读取；一个 cookbook 保存其场景的最小组装，跨页机制／案例各只有一个维护来源。

#### 装备分类与交付顺序

采用 `wiki/cookbook/equipment/index.md` 统一短入口，具体需求可绕过索引直达；不建立统一长装备教程。类别按组装场景分，装备槽位与功能能力分别选择：头盔从头部骨架扩展 armor，护甲背包从背包骨架扩展减伤，避免组合页爆炸。现只建已交付文件，拟定名称不作为链接入口。

| 顺序 | 页面／状态 | 最小范围与验收重点 |
|---|---|---|
| 1 | [hand.md](wiki/cookbook/equipment/hand.md) 已交付 | 物品工厂、HANDS、地面／持握资源、穿脱与步行倍率；武器／工具／技能另按需读；源码／语法／资源／结构检查，游戏未运行 |
| 2 | [body-armor.md](wiki/cookbook/equipment/body-armor.md) 已交付 | BODY、armor 普通吸收／受击耐久、耗尽卸装／存档；穿戴图可选；修复／位面防御／套装作为扩展；源码／语法／资源／结构检查，游戏未运行 |
| 3 | [head.md](wiki/cookbook/equipment/head.md) 已交付 | HEAD、草帽资源、精神基础贡献与头发／帽子显示恢复，省图可选；头盔防御／护目镜能力与外观模式分别扩展；源码／语法／资源／结构检查，游戏未运行 |
| 4 | `backpack.md` 待执行 | 装备容器、双端布局、开关／overflow 与卸装，原版 BODY 与可选 BACK 分开；保鲜另读机制 |
| 5 | `clothing.md` 待执行 | 身体外观和穿戴属性、持续消耗；可选 BELLY 与附加 armor 分开核验 |
| 6 | `amulet.md` 待执行 | 佩戴效果／撤销与原版槽位；可选 NECK，具体护符技能与持续耗能按需扩展 |

- [x] 只读梳理三个模组的 47 个直接 equippable 文件，复用哈希一致的已有 facts；按词法工厂得到 29 文件／30 手部候选组，逐项读工厂与回调，区分配置工厂／同文件不同 Prefab／私有 helper，不当完整 Prefab 计数或运行正确性。观察及例子保存在 [检视报告](docs/reports/w05/hand-equipment.md)。
- [x] 身体候选按已有词法组筛得 13 文件／13 组（DSTKisaki／MCW／Sora 为 3／6／4），定向审阅工厂／回调与需要的 helper；四组显式 BODY 且有 armor，包含有限耐久、不破损和角色省图差异，保留分类／槽位及后置 Hook 限制。
- [x] 头部候选按已有词法组筛得 6 文件／6 组（DSTKisaki／MCW／Sora 为 0／1／5），四组 HEAD、两组 NECK or HEAD；审阅完整／条件显示、只省符号仍改头发、容器／耗能与特效清理。候选不等于所有头部实现；原版普通／露顶／全包头盔分别核验，未实测。
- [ ] 后续逐类沿用只读梳理 → 必要原版核验 → 最小 demo → 扩展链接 → 资源／语法／登记／章节读取 → 实际游戏验收的顺序；不复制模组资源／专属组件，不把原生知识或扫描共现当证据。
- [ ] 多装备栏兼容作为可选接入：记录已核验版本和启用配置，确认注册／网络／UI、物品自动分类、外观包装、容器与相关消费者。`NECK/BACK/BELLY or BODY` 仅作槽位选择线索，存在槽位不代表全兼容；指定模组当前为 3173431824／2.4.0，HANDS 非 body 分支已源码审阅，未联机实测。
- [ ] 共同规则／陷阱放 equippable／inventory／container 等主要机制页并互链；具体外部实现按适用版本归对应机制案例，cookbook 仅保留必要兼容提示。六页不各自嵌完整 Hook／兼容层；不能有依据地通用化时保留项目专属范围。

首批手部、身体与头部骨架已交付；组件 Wiki 批次待 Skill 基础使用闭环后恢复，后续 cookbook 先配方、再建筑；背包、服装和护符随真实需求继续，不要求一次全部交付。完整必要工厂随叶级页保存，共用注册原理与机制扩展各只有一个维护来源；验证实际少量章节可取得所需上下文，而非通过增加索引要求每次多读一层。

#### 三个角色模组提供的规划依据

2026-10-11 检查到的注册和实现片段如下，仅说明章节选择依据；未执行模组、联网／更新逻辑，也未对完整功能作正确性审校。

| 模组 | 观察到的接入 | 对章节规划的影响 |
|---|---|---|
| DSTKisaki | `modmain/common/` 按序导入；`prefabs/kisaki.lua` 双阶段初始化；magic／level 等 component＋replica；actions／sg／rpc；containers、ui／keybindings；world_data／info_save；shadow_protector 的 Brain／SG | 角色、组件同步、容器、技能、HUD、随从；世界共享进度／角色恢复作为按需机制章节 |
| MCW | `mcwmain/` 模块；mcw 角色共同／服务端初始化；magic／rank 与 replica；双端 SG／RPC；背包 WidgetSetup；食物／debuff／projectile；`modworldgenmain.lua` 的布局与地皮注册 | 与上述共同场景优先复用；补食物、效果对象、世界生成；角色初始资源与世界生成资源分别验证 |
| Sora | `main/init.lua` 分模块导入；角色 netvar／初始化；packer replica／dirty、action 双端 handler；容器、UI；角色／全局存档组件和 shard RPC | 同步、容器、技能／HUD；存档迁移、分片同步为机制扩展。`sora2plant` 是农作交互逻辑，`sora2tree` 含容器，不能按命名当作植物骨架 |

补充只读实例：MCW 的 `scripts/prefabs/mcw_gembox.lua:214–232` 将自身拾取／图标／入栏与内部 container、deployable、preserver 分开组合；容器布局与筛选另在 `mcwmain/containers.lua:42`，恒温标签的消费在 `mcwmain/hook.lua:305`。后续容器骨架先抽通用组装，不直接复用宝石效果或外部温度 Hook；携带时直接启用物品 Light 的显示尚未运行验证，不能据此当作已验证方案。

- [x] 核验并补齐 modinfo 23 个原生顶层字段、配置内部字段及游戏模式声明，提供图标、优先级、依赖、兼容、manifest、独占与重启的示例／缺省／限制；按 [modinfo 主题](wiki/reference/api/modinfo/index.md) 分页维护。游戏运行和开放 settings 的所有消费者仍未验证，私有扩展不当原生字段。
- [ ] 将 `modimport`／`require` 环境、PostInit 覆盖／保留原行为归相关 API 页；起步页仅保留必要入口约定，后续骨架链接所需章节。
- [ ] 存档／旧数据迁移、死亡／复活／角色重选、地表洞穴共享进度分别补相关组件／生命周期／网络章节；只在需要多文件组装且现有骨架不足时另建 cookbook，不先建“通用数据库”。
- [ ] 图标、角色肖像／小地图、动画 bank／build／symbol、音效等资源规范归相应 API／资源主题；cookbook 留必要文件清单与加载位置，资源制作流程不混入起步页。
- [ ] Hook 兼容、配置开关与可选模组联动归机制案例及陷阱；皮肤、热更新、日志上传、项目私有工具不作为通用骨架前置。

### 待验证：按需读取机制

- [x] Wiki read 子命令实际输出所选章节行区间、元数据、父前提、适用与验证边界；LF／CRLF、嵌套／重复标题和围栏代码覆盖，输出预算超出整项省略不切代码。锚点本身仍不保证其他客户端裁剪；修改原页继续读全文。
- [ ] JSON 在工具内按 key／记录／Schema 定义过滤，模型只接收所需数据。单行 JSON 不等于短输入；当前 source-index/index.json 约 1.44 MB，禁止用读一行代替选择记录。
- [ ] 保留有限候选和输出字节预算，测量后再评估精简输出；任何简化都保留版本、覆盖、歧义、截断和 stale 等必要状态。
- [x] 首轮两项编码任务记录五条读取路径和完整返回量，包括机制页超预算补读与组装源码回退；同一会话重放及正确性审阅的限制见 [任务检视](docs/reports/w05/tasks.md)，后续仍需独立任务扩样和运行。
- [ ] 比较根分类导航与加入 Top 30 后的常用任务命中、工具调用、导航读取量及根额外成本；验证名称直查能跳过根。根据实际任务调整 [文件大小初始阈值](../dst-wiki-skill/references/maintain/write-format.md#文件大小)，保留案例必要上下文，不以压缩正确性换达标。

验收：首批骨架的必要上下文可取得、机制扩展可追溯、验证与缺口可见；两类任务能找到适用方案或明确缺口，无需全量读大分类。候选总预算、歧义／覆盖可见；有源码／仅机制页／骨架＋机制的可复核成本与正确性记录，不预设节省比例。确认章节／JSON 选择生效，修改保留全文审阅，已有内容任务不强制读骨架。

## W06：增量维护与按需扩展

- [ ] 新源码版本按文件与区间哈希重提取，标记受影响的类型、机制页和 cookbook 待复查；机制变化回查实际依赖它的骨架，不因链接存在就自动认定兼容。
- [ ] 测试方法变化、注解来源变化、行号移动和文件删除；重新生成不丢核验补充，不静默沿用失效证据。
- [ ] 核验后更新版本与锚点；维护冲突、重复主题、索引遗漏和替代入口。
- [ ] 组件行为页按 W05 的分批建设与覆盖清单持续扩展；W06 处理新版本影响、相关 replica／EntityScript 等接口与更多场景，正式完整类型审校按需推进，避免重复维护一份组件建设队列。
- [ ] 内容被替代后归档，不因未引用时长自动删除陷阱。
- [ ] 当前已采用独立 Skill＋外部知识配置；真实 harness验收、release发布、自动维护与进一步分发按需求建设。

### 当前分发：四仓独立与自包含

- [x] 知识仓保留 Wiki／raw，精简 facts／索引等缓存迁到本地 output；完整 AST 默认不落盘、不推远程。
- [x] dst-mod-dev-skill负责模组开发与本地 Wiki；dst-wiki-skill负责共享查询／知识维护，彼此不作为必装前置。
- [x] scripts 工具／Schema／fixtures／版本锁与工程参考拆到 dst-toolbox，原字节输入和正式 fork 保留。
- [x] 用户级 knowledge_root配置，工具箱可选；Skill 内部链接独立，源码準备显式根路径，校验 --format消费唯一规范。
- [x] 两 Skill 异地安装／含空格路径、显式路径优先、无源码／工具箱定位与各 7 项源码准备检查通过；134 项 Python 回归、真实 health提取／索引及 Wiki检索／章节通过。106 份原知识／数据／报告文件与日志前缀保留，详见 [分仓验收](docs/reports/repository-split.md)。
- [ ] 优先完成 dst-wiki-skill 基础使用闭环，再用安装后的 Skill 试建一小批组件页；逐项状态仅在 [Skill 完善计划](../dst-wiki-skill/PLAN.md) 跟踪，不在本仓重复验收清单。
- [ ] 开发 Skill 与工具箱后续真实使用／分发验收分别跟随各仓计划；仅目录／脚本检查不当全面兼容。明确授权后建立新远程与发布版本。

基础查询、骨架／案例／陷阱 Markdown增修不需要 Python、Rust、LuaLS、fork或工具箱；工程功能按所用工具准备依赖。新版证据失效和 token效果独立验证，不以分仓替代。

验收：改动能定位到受影响条目；未变化内容可复用；维护范围和待核验缺口可见。

## 已完成交付

### 项目入口与布局

- [x] 2026-10-12 重组 Skill 入口：AGENTS 仅 DST 触发与 Skill 链接；SKILL 统一 query／refine／ingest／lint，工程维护保留条件入口。references 查询用法重命名，机制维护／格式／索引／接口规则集中 maintain；新增 [骨架维护](../dst-wiki-skill/references/maintain-skeleton.md)，沿用证据／版本／大小要求，仅显式沉淀通用组装经验。补目标模组切换与 [跨项目安装说明](README.md#使用与安装)，移除不存在的白名单引用；知识页与历史日志原文保持，实际 token 收益待测。
- [x] 根目录补齐 [MIT 许可证](LICENSE)，在 [README](README.md#许可证) 明确原创内容与游戏／第三方内容的授权边界；Rust 提取器声明 MIT，独立 lsp-dst fork 保留 MIT-0。
- [x] CLAUDE／AGENTS／Skill 最小条件路由：仅 DST 任务触发；查询和知识维护不读 README／PLAN。Skill 使用规则／模板、仓库工程参考、docs 设计／决策／报告、Wiki 知识和 scripts 工具箱各归其位。
- [x] 共用源码准备脚本：同 build 跳过解压，更新先校验并保留旧快照；PowerShell 7 与 Windows PowerShell 5.1 的 7 项模拟安装检查通过。
- [x] 短根导航与现有分类索引；机制案例和场景问题在具体页维护一份。分层检索设计已归档，cookbook 分类与 demo、知识自动检索留 W05。
- [x] scripts 工具按 setup／source／verify／maintenance 归位，Rust 核心、机器 Schema、fixtures 与 Python 测试分开，补齐 [工具箱说明](../dst-toolbox/scripts/README.md)；保留源码、fork pin、基准和最终生成物。
- [x] 收紧接口数据加载，拆分知识维护／页面格式／索引规则，目录职责统一在总体设计；PLAN 改为未完成工作优先，基础使用无需 Python 写入执行规范。本轮仅优化文档，不执行 W04、检索工具或拆仓。

详细设计见 [总体设计](docs/design.md)、[结构决策](docs/decisions/knowledge-base-structure.md) 和 [分层检索决策](docs/decisions/layered-index-and-retrieval.md)；维护历史在 `wiki/log.md`，不重复展开旧整理清单。

### W01：统一接口数据契约

- [x] 契约 1.0.0 区分 facts／candidates／reviews／resolved，保留版本、符号身份、原字节区间／哈希、条件、未知、冲突和覆盖状态；以精简事实代替默认 AST 输入。
- [x] finiteuses／equippable 两份手工参考 bundle 与结构／证据／合并检查、12 项独立测试交付；历史样例保持原上游来源，正式核验输入现由 fork 维护。

依据：[输出契约](../dst-toolbox/references/output-format.md)、[JSON Schema](../dst-toolbox/scripts/schemas/contract.schema.json)、[finiteuses 样例](../dst-toolbox/scripts/tests/fixtures/bundles/finiteuses.json)、[equippable 样例](../dst-toolbox/scripts/tests/fixtures/bundles/equippable.json)、[校验工具](../dst-toolbox/scripts/tools/verify/validate_examples.py)。验收边界：契约和参考数据已验证，不表示 AST 输出、整库审校或实际运行。

### W02：组件模板与人工基准

- [x] 组件模板覆盖需求／接口、默认赋值与类型区别、属性副作用、真实生命周期、条件关系、代表调用及页内案例／限制。
- [x] 5 个独立 Lua 输入／51 项必要结构事实／8 项禁止误报；8 个真实源码文件／65 项结构事实／7 项人工行为记录；原预期和字节证据保留，8 项比较工具测试通过。W03 完整比较 116/116 项通过。

依据：[组件模板](../dst-wiki-skill/assets/templates/component.md)、[基准说明](../dst-toolbox/references/baselines.md)、[独立样本基准](../dst-toolbox/scripts/tests/fixtures/baselines/fixtures.json)、[真实源码基准](../dst-toolbox/scripts/tests/fixtures/baselines/pilots.json)、[检查／比较工具](../dst-toolbox/scripts/tools/verify/check_baselines.py)。验收边界：基准是所选特征的人工预期，非完整 facts／类型金标准，不进入默认知识候选。

### W03：最小提取与源码关系索引

- [x] Rust 1.99.0 x64／Full Moon 3.0.0／Lua 5.1、提取器 0.4.0；声明、参数、作用域、字段／属性／别名、return／guards／trivia、call／event／component／Prefab 和有限 resolution 三态。
- [x] health 首个 facts 契约检视覆盖字段、LF／CRLF 字节位置、WARNING 注释、解析／截断诊断；首层及第二层报告保留阶段状态。
- [x] 选文件／全量包装、manifest 与并列覆盖报告、定义／反向引用 JSON 分片和精确 key 查询；默认最多 5 项／16,000 输出字节，重名、动态、截断、stale 和缺口可见。
- [x] Steam buildid 25643519 的 4,030 个 Lua 文件：2,479 complete／1,549 partial／2 failed；`.po` 主动排除，编码／字节码缺口保留，原始源码不改。
- [x] 38 项真实提取／包装、8 项源码索引测试及完整 116/116 基准通过；原 5 项 Rust 探针／12 项契约／8 项比较检查保留。

依据：[最终验收](../dst-toolbox/docs/reports/w03/acceptance.md)、[首层检视](../dst-toolbox/docs/reports/w03/stage1.md)、[第二层检视](../dst-toolbox/docs/reports/w03/stage2.md)、[包装工具](../dst-toolbox/scripts/tools/source/parse_scripts.py)、[源码索引](../dst-toolbox/scripts/tools/source/source_index.py)、[查询用法](../dst-wiki-skill/references/query-source.md)。验收边界：最小结构提取与有限查询完成，动态／隐式值／任意 CFG／跨文件语义仍有限制；没有 LuaLS、类型／Wiki 生成或游戏运行验收。

### 类型库前期交付与试点依据

- [x] [YingSP/lsp-dst](https://github.com/YingSP/lsp-dst) 独立 fork 已发布，2 份注解的 13 条修订有正式源码 reviews；固定 commit 为 `869443111c1686f7b1bf1e34e5f9d54b5e571954`，配置在 `scripts/vendor/lsp-dst.json`，克隆在同目录 `lsp-dst/`。维护见 [fork 说明](../dst-toolbox/references/lsp-dst.md)，其余条目仍为候选。

W04 先消费 finiteuses／equippable，必要属性／宿主证据和代表调用见 W02 基准说明；该基准的 8 个文件不等于全组件已审校。后续 health、inventory、bufferedaction 等旧锚点仅是扩展线索，使用前重新核对版本与原字节。源码访问已获授权，核验不修改 `raw/`。

## 暂缓工作

- 完整 codegraph、复杂跨文件类型推断、图数据库：轻量引用不足时再评估。
- 全库高精度类型、所有方法详细 Wiki：试点闭环后按收益扩展。
- 向量检索、网页展示、多模式 Wiki、定时同步：不作为当前前置。
- 类型库正式 release、打包与自动安装：按分发需求确定，保留上游许可证与固定基线。
- 不为暂缓或未执行的分发项预建目录、服务、子模块或自动化。
