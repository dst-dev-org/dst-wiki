# FX 独立检视第 10 轮：外围 D＋E 扩大批

日期：2026-11-06。实读 Steam buildid **26335394**；知识仓 HEAD `97afb4e9e6ca05390558769926f1f9c8598d0f70`。按用户要求扩大单轮范围，合并外围 D、E，共 **7 页／121 个注册名／759 个物理行／138,125 字节**。虽然页数少于上一轮，注册数为上一轮的约 2.7 倍，正文体积约 2.4 倍。

**当前结论：已收口。初检 P1×0、P2×7、P3×3，修后发现的 R05 摘要残项也已修改并独立复验通过。** 下文保留初检与中间复验历史，最终结论见文末；原批次 accepted、既有复审和哈希一致均不替代本次判断。

本轮只新增本报告并追加日志；不修改知识页、索引、基线、PLAN、台账、Skill 或工具，保留并行装备页改动，不提交、不推送。没有重新验收第 5 轮 R10。

## 范围与逐页结论

| 页面 | 注册数 | 行／字节 | 本次结论 |
|---|---:|---:|---|
| [reticuleaoe](../../../wiki/reference/prefabs/reticuleaoe.md) | 48 | 178／32,658 | 三工厂、48 名变体、联网 target 驻留／KillFX、本地 ping、蓄力、多目标与高尔夫调用已核对；回显接口生成条件见 R07（P2）。 |
| [rain](../../../wiki/reference/prefabs/rain.md) | 7 | 96／16,404 | 两类天气组件驱动、非专服生成、雨棚／地图门禁、双池与延迟复用已核对；三个随机寿命范围见 R01（P2）。 |
| [wave_med](../../../wiki/reference/prefabs/wave_med.md) | 7 | 104／16,348 | 碰撞推船、加湿推物、SG、共享生成 helper、波光与岸浪本地工厂已核对；入口参数边界见 R06（P2），动画名见 R10（P3）。 |
| [staff_castinglight](../../../wiki/reference/prefabs/staff_castinglight.md) | 2 | 58／5,832 | 普通／小号曲线、主模拟 SetUp、netvar 与客户端更新、结束后 2 帧删除及六处 SG 生成参数均相符。未发现 P1/P2。 |
| [fire](../../../wiki/reference/prefabs/fire.md) | 15 | 111／18,612 | 九套 firefx 等级与六个 heater、burnable 生成清理、眼焰／烟／灯与事件包边界已核对；档位见 R02、端描述见 R03（P2），内部表和宿主标签见 R08／R09（P3）。 |
| [cavelight_small](../../../wiki/reference/prefabs/cavelight_small.md) | 13 | 91／17,359 | 相位、洞穴满月初值、宽度／半径、hideout／spawner、出生光、读书与变身光、布局及中庭门供能链均定向核对。未发现 P1/P2。 |
| [houndwarning_lvl1](../../../wiki/reference/prefabs/houndwarning_lvl1.md) | 29 | 121／30,912 | 动态预警名与客户端事件、代理／本地声源、三类暗影门禁、阴影宿主与调试开关已核对；半径推导见 R04、落地清理见 R05（P2）。 |

七页全文均已阅读；工厂范围为 49 个 Prefab 文件。`caverain`／`caveacidrain` 用完整文件差异与已读 `rain` 对照，其余工厂分段阅读。宿主、组件、SG、世界生成等依赖按页面关键结论定向读取，未声称把 178 个已声明来源文件全部整文件审校。

## P2：需要修正

### R01｜rain：把 UnitRand 当成 0～1，三个寿命范围错误

- 页面：[rain:32](../../../wiki/reference/prefabs/rain.md#L32) 写雪寿命 4.5～7.5 秒；第 33 行写花粉 30～60 秒；第 34 行写月雹为 `0.7 × (.9～1)`。
- 依据：[emitters.lua:101–103](../../../raw/current/scripts/emitters.lua#L101-L103) 的 `UnitRand()` 为 `math.random()*2-1`，取值范围约为 −1～1。结合 [snow.lua:59–60](../../../raw/current/scripts/prefabs/snow.lua#L59-L60)、[snow.lua:97–100](../../../raw/current/scripts/prefabs/snow.lua#L97-L100)、[pollen.lua:113](../../../raw/current/scripts/prefabs/pollen.lua#L113)、[lunarhail.lua:158–163](../../../raw/current/scripts/prefabs/lunarhail.lua#L158-L163)，实际传入粒子接口的范围分别约为 **1.5～7.5 秒、0～60 秒、0.56～0.7 秒**。
- 建议：以完整表达式及其范围修正三行，区分常量 `MIN_LIFETIME` 的名字与实际下界；补 `emitters.lua` 的正文依据、sources，并按规则刷新该页基线。上述是源码参数推导，粒子接口对极小寿命的实际显示未运行验证。

### R02｜fire：通用燃烧 helper 的档位与正文自身冲突

- 页面：[fire:67](../../../wiki/reference/prefabs/fire.md#L67) 将 `Make*Burnable` 写成 1／2／3 档；第 91 行却正确写了 Medium／Large 为 3／4 档。
- 依据：[standardcomponents.lua:164–198](../../../raw/current/scripts/standardcomponents.lua#L164-L198)：通用 Small／Medium／Large 分别为 **2／3／4**；[Character 分支](../../../raw/current/scripts/standardcomponents.lua#L245-L281) 才是 **1／2／3**。
- 建议：第 67 行明确区分通用 2／3／4 与角色／尸体 1／2／3，保留第 91 行的正确结论。此处关系到按档查询热度和动画，不只是编号体例。

### R03｜fire：把 firehit 网络代理的生成端与本地动画的渲染端混为一谈

- 页面：[fire:98](../../../wiki/reference/prefabs/fire.md#L98) 将 `eyeflame` 与 `firehit` 合并，进而写“服务端或专服生成看不到”。
- 依据：[firehits.lua:31–54](../../../raw/current/scripts/prefabs/firehits.lua#L31-L54)：`firehit` 本体有 Network；每个非专服端排任务创建本地动画，专服仅跳过自身可见体，并未据此阻止客户端表现。[eyeflame 工厂](../../../raw/current/scripts/prefabs/eyeflame.lua#L112-L131) 才是完全非联网的独立实体。
- 建议：分别说明“eyeflame 直接生成只在本端存在”与“firehit 联网代理＋非专服本地可见体”。继续保留 `event_server_data` 缺失的限制；不能反过来承诺在缺失事件包时直接生成 firehit 可正常运行。

### R04｜houndwarning：零基础半径不是所有客户端始终零偏移；对源码注释的数学纠错也不成立

- 页面：[houndwarning_lvl1:38](../../../wiki/reference/prefabs/houndwarning_lvl1.md#L38)、第 67／110 行把坎普斯、酸蝠描述为固定焦点、没有方向偏移；第 108 行又称源码“2 倍 range＋buffer 时到 30”的注释与参数不符。
- 依据：[monsterwarningsounds.lua:95–103](../../../raw/current/scripts/prefabs/monsterwarningsounds.lua#L95-L103) 与 [easing.lua:44–47](../../../raw/current/scripts/easing.lua#L44-L47)：仅当焦点与代理距离 `d <= range` 时使用基础半径；超出后为 `r = base + (30-base)*((d-range)/(range+10))^2`。
- 反例：坎普斯 `base=0, range=30, d=50` 时半径为 **7.5**；巨鹿 `base=25, range=40, d=90` 时恰为 **30**，正好满足注释的 `2*range+10`。
- 建议：将零偏移限定为 range 内，修正第 38／67／110 行；撤回第 108 行对该注释的否定，保留“未 clamp，超出该距离可继续增长”的正确边界。这些是坐标计算，未验证实际音量／听感。

### R05｜houndwarning：落点阴影不能只等宿主删除才清理

- 页面：[houndwarning_lvl1:87](../../../wiki/reference/prefabs/houndwarning_lvl1.md#L87) 写“宿主 onremove 是唯一清理点”，并称三个原版宿主均按同一模式接；示例只接了宿主删除。
- 依据：[SGgrassgator.lua:293–297](../../../raw/current/scripts/stategraphs/SGgrassgator.lua#L293-L297) 在离开 `fall` 时主动删阴影，草鳄实体继续存在。`quaker` 也有落地过程中的显式删除，不能把工厂无自删推导成宿主只有 onremove 清理。
- 建议：改为“宿主负责落地／状态退出时 Remove，onremove 作兜底”。示例可保留最小片段，但须补清落地或结束时应移除返回的 shadow；否则给不会随落地销毁的宿主复用时，会残留阴影。

### R06｜wave_med：instantActive 的状态选择漏了 idleTime <= 0 分支

- 页面：[wave_med:81](../../../wiki/reference/prefabs/wave_med.md#L81) 将 `instantActive=true` 一律描述为直接进入 `instant_rise`。
- 依据：[ocean_util.lua:62–66](../../../raw/current/scripts/ocean_util.lua#L62-L66)：真分支实际为 `idleTime > 0 and "instant_rise" or "lower"`；[SGwave.lua:64–78](../../../raw/current/scripts/stategraphs/SGwave.lua#L64-L78) 的 lower 从消散动画开始。
- 建议：补充“idleTime>0 进入 instant_rise，否则进入 lower”；原示例传 2，示例本身不需要修改。非 instantActive 仍沿默认 rise 路径，不要扩大修订范围。

### R07｜reticuleaoe：EchoReticuleAt 的 Flash 判断控制后处理，不控制生成

- 页面：[reticuleaoe:122](../../../wiki/reference/prefabs/reticuleaoe.md#L122) 写 `EchoReticuleAt` “仅在 reticule.Flash == nil 时……生成同 prefab 1 秒”。
- 依据：[playercontroller.lua:1711–1720](../../../raw/current/scripts/components/playercontroller.lua#L1711-L1720)：先调用 `SpawnPrefab`，成功后才分支；有 Flash 时调用 Flash，无 Flash 才排 1 秒 Remove。
- 建议：改为“先生成回显，有 Flash 则调用，否则 1 秒后删除”。保留当前快照无调用点的范围说明，不据此虚构原版使用场景。

## P3：局部精度修整

### R08｜fire：burnfx 是 local 映射表，不是全局 helper

[fire:68](../../../wiki/reference/prefabs/fire.md#L68) 的“全局 helper burnfx”与 [standardcomponents.lua:158–164](../../../raw/current/scripts/standardcomponents.lua#L158-L164) 不符：`burnfx` 是文件内 local 表；全局函数是 `MakeSmallBurnable` 等。建议改对象名为“标准燃烧 helper／内部 burnfx 映射”，避免查询者尝试访问全局 burnfx；不需要新建 API 页。

### R09｜fire：stagehand 显示标签链接到了 stageusher

[fire:31](../../../wiki/reference/prefabs/fire.md#L31) 的 `[stagehand]` 实际指向 [stageusher.lua:223](../../../raw/current/scripts/prefabs/stageusher.lua#L220-L223)，是另一个宿主。`stagehand` 的同类调用在 [stagehand.lua:101–104](../../../raw/current/scripts/prefabs/stagehand.lua#L101-L104)。最小改法是把现有链接标签改为 `stageusher`；两者都会用 campfirefire，不必为了此标签错误额外扩 sources。

### R10｜wave_med：岸浪随机动画首项不是 idle_small1

[wave_med:35](../../../wiki/reference/prefabs/wave_med.md#L35) 写随机 `idle_small1-4`；[wave_shore.lua:6](../../../raw/current/scripts/prefabs/wave_shore.lua#L6) 的实际名单为 `idle_small`、`idle_small2`、`idle_small3`、`idle_small4`。建议写全或写“idle_small 及 2～4 后缀变体”，避免复制不存在的动画名。

## 已核验且不作为问题的边界

- 本族共 121 个实际注册名，均同时有 aliases 和正文完整名称，跨 Prefab 页未发现这些注册名的 aliases 重复。reticuleaoe 另有 `golfclub_reticule` 检索词，它是相关组件／工厂文件名，不混算为第 49 个注册。
- 7 页正文带行号的源码路径均已在本页 sources 声明；没有沿用上一轮的“漏声明来源”结论。此检查不保证所有隐含依赖都已声明。
- `.po` 对 121 名逐键查到 `FIRE=火`、`ATRIUM_LIGHT=远古灯柱` 和 `RAIN=Short Circuit／短路`。前两者已正确引用；rain 页已解释第三个键不作为降水 FX 的官方名，没有误套译名。
- `cavelight` 在洞穴用 `isfullmoon` 而非 `iscavefullmoon` 的边界成立；`worldstate` 洞穴初始化又明确 `isnight=true`，因此不能据此反推 hallucinations 的两个 nightonly 工厂在洞穴不可达。
- target FX 淡入后驻留、服务端 KillFX 反向淡出成立；多目标 sub 是对实际目标 `ent:AddChild`，由 main 的 `_targets` 列表清理，不等同 SetParent 的引擎生命周期推测。
- `firehit`、Lava Arena reticule 的事件数据包未核验范围继续保留；`wave_shimmer_flood` 的注册存在与当前 scripts 中未发现生成调用仍须分开，不上升为引擎／模组均不可能生成。

## 本次独立校验与覆盖边界

| 检查 | 本次实际结果 |
|---|---|
| 实际快照／Git | BUILD 26335394；HEAD 如首段；保留已有改动和装备批认领。 |
| 页面格式与导航 | `validate_page.py --check-indexes`：7／7 passed；全库 1143 知识页／34 索引，navigation 0 问题。 |
| 严格源码引用 | `validate_source_links.py --strict`：501 个行引用／428 个唯一目标／158 个文件，0 error、0 warning、0 skipped，exit 0。 |
| 来源基线 | 所选 7 页 `check --fresh-hours 0`：194 组 unchanged；changed／sources_changed／unrecorded／invalid 均 0，未 record。 |
| 来源声明与主归属 | 正文行引用缺声明 0；121 个实际注册名均在正文和 aliases，跨页主注册 aliases 冲突 0。 |
| 标题原文章节读取 | 7 页各选 1 节，包含括号、空格及长中文标题，`query_wiki read` 全部 exit 0。未重建搜索缓存，不以 stale search 作验收。 |
| 叶级返回链 | 实读 7 条登记和页尾返回：reticule／staff→fx-index；rain／wave／fire／cavelight→fx-world-index；houndwarning→fx-creature-index，一致。 |
| 源码反查 | 全 4095 个 Lua 文件作 121 名全词候选扫描；deerclops／bearger 的第 4 档无完整字面名，由循环和拼接实际核对，未当作缺失注册。负向项另检查直接调用、公共入口及依赖声明。 |
| 输入稳定性 | 七页与首次机械盘点的字节哈希一致；复核后段对 7 页＋178 个声明来源作 185 文件哈希复查，未变化。 |
| 未检 | 未运行 DST、未解包资源、未验证听感／网络时序；未独立重跑四段既有 Lua 示例的语法工具。机械检查不证明本报告列出的语义结论。 |

## 后续与计数

修复 R01～R07 后再对改页及相关章节复验；R08～R10 可同批局部修整。只有实际新增／改变 sources 的页需要按现有规则处理基线，不因报告产生就 record 整批。

累计已独立检视 **75／158 页**，固定队列余 **83 页**；这是“已检视”计数，包含待修项，不表示 75 页全部独立收口。下一轮可继续扩大为外围 **F～H 共 9 篇**：`fence_gate_anim`、`archive_chandelier`、`lavaarena_beetletaur_fossilized_break_fx_right`、`armor_lunarplant_glow_fx`、`alterguardian_laserscorch`、`charlie_boss_aoe_flame_fx`、`groundpoundring_fx`、`deerclops_laserscorch`、`ice_puddle`；涉及混合玩法载体，仍逐页核验，不能只做结构抽查。

## 2026-11-06 修后独立复验：R05 仍有摘要泛化

本次实读 BUILD 仍为 26335394、HEAD 未变。完整核对五篇改页相对 HEAD 的新增／替换文本，并回读对应源码；`staff_castinglight`／`cavelight_small` 无工作区差异、字节哈希与初检相同，沿用初检结论。没有把他方“全部采纳”的处置日志当独立验收。

| 初检项 | 修后核验 | 当前状态 |
|---|---|---|
| R01 | rain:32～34 改为约 1.5～7.5／0～60／0.56～0.7 秒，第 39 行引用 UnitRand 定义；sources 已加 emitters.lua，所选页基线检查无漂移。 | 通过 |
| R02 | fire:67 区分通用 2／3／4 与角色／尸体 1／2／3，和 standardcomponents 的实际函数及原第 91 行相符。 | 通过 |
| R03 | fire:98 分开 eyeflame 非联网与 firehit 联网代理＋非专服本地动画，并保留事件包缺失限制。 | 通过 |
| R04 | houndwarning:38／67／109／111 限定 range 内基础半径，承认超范围增长，撤回对 `2*range+10` 注释的错误否定。 | 通过 |
| R05 | 第 87 行补三类宿主的落地／退出及删除兜底证据，第 96／104 行明确示例还需落地清理；但第 24 行把同样的 onremove 兜底写给了 meteorwarning。 | 部分修复，仍有 P2 |
| R06 | wave_med:81 补 idleTime>0／<=0 分别进入 instant_rise／lower，非 instantActive 仍从 rise 开始。 | 通过 |
| R07 | reticuleaoe:122 改为先生成，再调用 Flash 或排 1 秒删除，保留当前 scripts 无调用点的限制。 | 通过 |
| R08 | fire:68 明确 burnfx 是 local 表、Make*Burnable 才是全局函数。 | 通过 |
| R09 | fire:31 显示标签已改 stageusher，与目标文件、行号一致。 | 通过 |
| R10 | wave_med:35 写全 idle_small、idle_small2、idle_small3、idle_small4。 | 通过 |

### 唯一待修：不要把 warningshadow 的宿主兜底写成 meteorwarning 的既有保证

- 当前位置：[houndwarning_lvl1:24](../../../wiki/reference/prefabs/houndwarning_lvl1.md#L24)：“`warningshadow`、`meteorwarning` 都依赖宿主在落地或退出时 Remove，onremove 兜底”。
- 实查 [shadowmeteor.lua:46–52](../../../raw/current/scripts/prefabs/shadowmeteor.lua#L46-L52) 明确在 `onexplode` 删除 warnshadow；对该文件全文反查 `warnshadow`／`onremove`／`OnRemoveEntity`，未发现为它注册 Lua onremove 兜底。[shadowmeteor.lua:298–299](../../../raw/current/scripts/prefabs/shadowmeteor.lua#L298-L299) 的 SetParent 不能充当已核验的 Lua 清理回调证据。这里不对引擎父子级联作结论。
- 同页 [第 110 行](../../../wiki/reference/prefabs/houndwarning_lvl1.md#L110) 仍正确限定 meteorwarning 原版在 `shadowmeteor:onexplode` 清理，与第 24 行发生冲突。
- 最小修法：第 24 行只概括“两者无自身定时删除，需宿主显式清理”；如保留细分，再分别写“warningshadow 的原版宿主在落地／退出时清理并接 onremove 兜底；meteorwarning 的 shadowmeteor 路径在 onexplode 清理”。无需改源码、扩大示例或更新 sources／基线。

### 本次实际校验

- 五篇改页格式全部 passed；全库导航仍为 1143 知识页／34 索引／0 问题。
- strict **426 行引用／361 唯一目标／136 文件，0 error、0 warning、0 skipped**，exit 0。
- 所选五页来源 **164 组 unchanged**；changed／sources_changed／unrecorded／invalid 均 0，未 record。
- 五篇正文行引用均已声明 sources；五次标题原文章节 read 均 exit 0。没有重建查询缓存。
- 只新增 Lua 注释，没有改示例可执行语句；本次未重跑语法工具，未运行 DST。

本次仅更新报告并追加日志，不改知识页或他方交付。累计仍为 **75／158 页，余 83 页**；R05 修完后仅需定向复验该摘要与前后文，不必重开整批源码审阅。

## 2026-11-06 R05 摘要定向复验：收口

启动第 11 轮时发现他方已修摘要，按上次留下的最小范围独立复验：[houndwarning_lvl1 第 24 行](../../../wiki/reference/prefabs/houndwarning_lvl1.md#L24) 现只概括两类阴影均需宿主显式清理，不再把 onremove 兜底泛化到 meteorwarning；与第 87、110 行的分工及 `shadowmeteor.lua:49-51` 一致。前次已验收的其余九项不重复通审。

本次该页格式 1/1 passed、strict 109 行引用零错警、41 组来源 unchanged，三个命令均 exit 0；未 record、未运行游戏。R05 残项通过，**第 10 轮全部收口**。本次随第 11 轮报告统一追加日志，不改知识页；不涉及第 5 轮 R10 的历史复验。
