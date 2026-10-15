# FX 独立检视第 5 轮（原生成批 6）

初检日期：2026-11-06，初检需修复 P1×1、P2×8。**同日修后独立复验：原 R01～R09 整改点已覆盖，包括 R08；当前仅新增 R10（P2×1），待修 P1×0。** 下文初检保留历史，以文末复验为当前结论。编号按修复任务分组，多个位置属于同一问题时不重复计数。本轮只读复审知识页，只写报告与日志；不修知识页、不 record、不改索引、PLAN 或台账 accepted。未运行游戏。

## 范围与基准

- K：`D:/Code/dst-wiki-develop/dst-wiki`；S：`D:/Code/dst-wiki-develop/dst-wiki-skill`；T：`D:/Code/dst-wiki-develop/dst-toolbox`。
- 实读 `raw/current/BUILD.txt` 为 `26335394`，源码根为 K 的 `raw/current/scripts`；K HEAD 为 `97afb4e9e6ca05390558769926f1f9c8598d0f70`。沿用当前 Skill 的源码证据、Prefab 主要归属与来源规则。
- 范围取自 [覆盖台账原批 6](fx-coverage-ledger-20261101.md#L1011)：5 页、27 注册。作者既有 accepted／复审记录仅作线索，本轮独立核对。上一轮 R08 尚待修，本轮不替其收口。
- 开始时 K 已有上一轮修订、来源基线／日志／台账改动及 5 个未跟踪工具页；全部保留。S／T 工作区干净。未认领或覆盖他方生成任务。

| 页面 | 注册数 | 物理行数／UTF-8 字节 | 本轮待修 |
|---|---:|---:|---|
| [sinkhole_spawn_fx](../../../wiki/reference/prefabs/sinkhole_spawn_fx.md) | 7 | 83／11,724 | R03、R04、R05；R09 为关联页冲突 |
| [glass_fx](../../../wiki/reference/prefabs/glass_fx.md) | 3 | 67／8,424 | R06 |
| [quagmire_portal_player_fx](../../../wiki/reference/prefabs/quagmire_portal_player_fx.md) | 5 | 79／10,472 | R03、R07 |
| [lavaarena_portal_player_fx](../../../wiki/reference/prefabs/lavaarena_portal_player_fx.md) | 6 | 80／11,624 | R02 |
| [firesplash_fx](../../../wiki/reference/prefabs/firesplash_fx.md) | 6 | 87／12,002 | R01、R08 |

物理行数不把文件末尾换行再算一个空行；五页均在 standard 预算内。核对全部 27 条注册配置与可见消费路径；伤害／复活、宿主／代理／本地实体、父挂与清理等高风险表述逐项追源码，未采用抽样。事件数据包、引擎级联删除与动画包内容不可见时保留缺口。

## R01｜P1：暗影突袭的攻击范围赋值就在 SG 内，不是事件包缺口

- 页面位置：[导航 L46-L47](../../../wiki/reference/prefabs/firesplash_fx.md#L46-L47)、[陷阱 L78](../../../wiki/reference/prefabs/firesplash_fx.md#L78)、[验证记录 L83](../../../wiki/reference/prefabs/firesplash_fx.md#L83)。现文称 `attackrange=5` 的赋值未在树内核验，并把实际阵容／赋值主要归入不可见的熔炉事件包。
- `SGshadowwaxwell` 的 `idle` 与 `ready_pre` 均在“有 timer 组件且没有 shadowstrike_cd”时直接 `combat:SetRange(5)`（[L339-L361](../../../raw/current/scripts/stategraphs/SGshadowwaxwell.lua#L339-L361)）。`doattack` 再据 `attackrange == 5` 进入 `lunge_pre`（[L250-L257](../../../raw/current/scripts/stategraphs/SGshadowwaxwell.lua#L250-L257)），命中分支生成斩击并结算伤害（[L1302-L1332](../../../raw/current/scripts/stategraphs/SGshadowwaxwell.lua#L1302-L1332)）。
- 本体宿主也有可见支撑：`shadowprotector` 的 `protectorfn` 调用 `MakeOblivionSeeker`，后者安装 timer（[安装 helper](../../../raw/current/scripts/prefabs/shadowwaxwell.lua#L122-L128)、[protectorfn](../../../raw/current/scripts/prefabs/shadowwaxwell.lua#L322-L340)）；共同初始化安装 SG，再调用专属初始化（[L547-L568](../../../raw/current/scripts/prefabs/shadowwaxwell.lua#L547-L568)），实际返回 `MakeMinion("shadowprotector", ..., protectorfn)`（[L668-L675](../../../raw/current/scripts/prefabs/shadowwaxwell.lua#L668-L675)）。不能把这条本体接线记成事件包缺口，也不能把所有挂同 SG 的变体都视为同样可达。
- 建议：补上 SG 中的范围赋值门禁与本体宿主依据，同步导航、陷阱、验证记录的旧说法。仅保留 `waxwell_shadowstriker` 自身事件包内部逻辑等真正未核验范围；不把 timer 存在直接等同任意时刻必定触发突袭。

## R02｜P2：熔炉页把父挂写成了已证明随宿主清理

- 页面位置：[典型接入 L52](../../../wiki/reference/prefabs/lavaarena_portal_player_fx.md#L52)：“父挂宿主被移除时本地实体随之离场”。
- floorgrate 只对代理调用 `entity:SetParent`（[L40-L42](../../../raw/current/scripts/prefabs/lavaarena_floorgrate.lua#L40-L42)）；通用工厂把本地实体挂到代理的父级，同样只调用 `entity:SetParent`（[L18-L37](../../../raw/current/scripts/prefabs/fx.lua#L18-L37)）。这些调用没有经过 Lua `AddChild` 的 `children` 登记（[L921-L935](../../../raw/current/scripts/entityscript.lua#L921-L935)），不能直接套用 `Remove` 遍历 children 的路径（[L1779-L1783](../../../raw/current/scripts/entityscript.lua#L1779-L1783)）。本轮没有引擎级联删除的证据。
- 建议：写已核验的随宿主移动、代理 1 秒回收、本地 animover／animqueueover 自删（[工厂 L98-L102](../../../raw/current/scripts/prefabs/fx.lua#L98-L102)、[L156-L157](../../../raw/current/scripts/prefabs/fx.lua#L156-L157)），把宿主提前移除后的引擎行为留为未核验。也不能反向断言引擎绝不随父清理。

## R03｜P2：两页 sources 漏掉四项支撑依赖

| 页面位置 | 漏项 | 正文已经使用的依据 |
|---|---|---|
| [sinkhole sources L10](../../../wiki/reference/prefabs/sinkhole_spawn_fx.md#L10)、[陷阱 L71](../../../wiki/reference/prefabs/sinkhole_spawn_fx.md#L71) | `components/quaker.lua` | 正文直接链接该文件 L743 |
| [sinkhole 导航 L40](../../../wiki/reference/prefabs/sinkhole_spawn_fx.md#L40) | `stategraphs/SGdaywalker.lua` | 声称 SGdaywalker 推送 docollapse，实际见 [L1531-L1537](../../../raw/current/scripts/stategraphs/SGdaywalker.lua#L1531-L1537) |
| 同上 | `stategraphs/SGbearger.lua` | 声称 SGbearger 推送 docollapse，实际见 [L1393-L1406](../../../raw/current/scripts/stategraphs/SGbearger.lua#L1393-L1406) |
| [quagmire sources L10](../../../wiki/reference/prefabs/quagmire_portal_player_fx.md#L10)、[导航 L43／L48](../../../wiki/reference/prefabs/quagmire_portal_player_fx.md#L43-L48) | `prefabs/fx.lua` | 正文两次直接链接资产派生实现 |

建议补齐实际支撑文件与必要锚点；若收窄结论，须同步消除对应未声明依赖。修订实际核验后只更新相应页面的来源基线。当前 54 组哈希一致只覆盖已声明来源，不能替代完整性核查。R05 新补的调用链另按实际核验范围增加来源。

## R04｜P2：地陷页三处位置／缩放参数写错

- [洞穴导航 L38](../../../wiki/reference/prefabs/sinkhole_spawn_fx.md#L38) 写“玩家 4 单位内直落玩家坐标”。源码先将中心改为玩家坐标，随后仍加 `spread * cos/sin` 偏移；初值 0.2，后续每枚 +0.5 上限 1（[caveins L34-L60](../../../raw/current/scripts/components/caveins.lua#L34-L60)）。应写“中心跟到玩家坐标，再按 spread 偏移”，不能写每枚精确命中玩家坐标。
- [陷坑导航 L39](../../../wiki/reference/prefabs/sinkhole_spawn_fx.md#L39) 与 [案例说明 L67](../../../wiki/reference/prefabs/sinkhole_spawn_fx.md#L67) 写“±10%”位置抖动。源码 x／z 的偏移分别乘 `1 + math.random() * .1`（[antlion_sinkhole L36-L48](../../../raw/current/scripts/prefabs/antlion_sinkhole.lua#L36-L48)），是各轴偏移幅度向外增加 0～10%，不是正负抖动，也不是统一一个随机半径。
- [眼魔导航 L41](../../../wiki/reference/prefabs/sinkhole_spawn_fx.md#L41) 写 `scale 0.6±0.2`；实际为 `0.6 + math.random() * .2`（[SGeyeofterror L880-L896](../../../raw/current/scripts/stategraphs/SGeyeofterror.lua#L880-L896)）。应写 0.6 起加 0～0.2，并区分中心那枚未做此缩放。
- 建议同步表格与案例文字，保留案例“简化原版”的适用范围，不必为了修文字扩大示例。

## R05｜P2：HandleDugGround 共享入口漏追自动翻地链

- 页面位置：[翻地导航 L42](../../../wiki/reference/prefabs/sinkhole_spawn_fx.md#L42)、[验证记录 L78](../../../wiki/reference/prefabs/sinkhole_spawn_fx.md#L78)。现表只追了 `terraformer`／pitchfork，验证记录却称生成点与依赖命中均已列入。
- 全 scripts 查 `HandleDugGround` 有定义和两个调用方：除 `terraformer` 外，`autoterraformer:DoTerraform` 的可翻地分支也调用它（[L56-L105](../../../raw/current/scripts/components/autoterraformer.lua#L56-L105)）。helper 在无草皮映射时才生成本族 FX（[componentutil L31-L45](../../../raw/current/scripts/componentutil.lua#L31-L45)）。
- 该组件不是孤立死入口：`antlionhat` 的服务端工厂实际安装组件（[hats L2415-L2437](../../../raw/current/scripts/prefabs/hats.lua#L2415-L2437)），装备且宿主有 locomotor 时 StartTerraforming，卸装时停止（[L2371-L2387](../../../raw/current/scripts/prefabs/hats.lua#L2371-L2387)）。其 onfinish 另出 turf_smoke_fx，与 helper 的条件生成不能混为一谈。
- 建议增加这条有条件的自动翻地调用链与相关 Wiki／来源，明确无草皮映射等前提；不要写成戴帽或每次翻地必出尘爆。现有“terraformer 组件内只有 pitchfork 走默认分支”的狭义结论与 farm_plow 被接管的说明可保留，但不能代表 helper 的全部消费者。

## R06｜P2：glass_fx 的任务时刻不能证明与换物绝不同帧

- 页面位置：[快速判断 L20](../../../wiki/reference/prefabs/glass_fx.md#L20)、[流程 L37-L39](../../../wiki/reference/prefabs/glass_fx.md#L37-L39)、[陷阱 L58](../../../wiki/reference/prefabs/glass_fx.md#L58)；[摘要 L6](../../../wiki/reference/prefabs/glass_fx.md#L6) 也预设了先后。
- Lua 能证明：`GLASS_TIME = 24 * FRAMES`（实际 [L29](../../../raw/current/scripts/prefabs/sand_spike.lua#L29)），OnIgnite 安排 22 帧后的 PlayGlassFX，并单独监听动画结束 ChangeToGlass（[L170-L188](../../../raw/current/scripts/prefabs/sand_spike.lua#L170-L188)）。换物本身由 animover 回调完成（[L162-L168](../../../raw/current/scripts/prefabs/sand_spike.lua#L162-L168)）。`GLASS_TIME` 也作为燃烧时长参数使用（[L95-L110](../../../raw/current/scripts/prefabs/sand_spike.lua#L95-L110)），没有在这里设置动画总长。
- 本快照没有核验动画包，不能从上述两个调度入口推出“固定领先”“二者不同帧”，也不能保证在一切移除／打断情况下 22 帧任务都执行。建议分别记两个触发条件，删确定相对时序，动画实际结束帧与同帧调度留待验证；必要时同步摘要与索引中的先后暗示。
- L37 的 GLASS_TIME 链接目前指向 L28，那是 DAMAGE_RADIUS_PADDING；收紧到真实 L29。这类语义错锚点不会被行号存在性校验发现。

## R07｜P2：不可见的 Salt／Replate 调用不能支持“全程不生成 FX”

- 页面位置：[quagmire 导航 L46](../../../wiki/reference/prefabs/quagmire_portal_player_fx.md#L46)：“REPLATE……全程不生成 FX”。
- 可见 actions 段读取盐渍标签、重建菜品，并调用 `quagmire_saltable:Salt(0, true)` 与 `quagmire_replatable:Replate(...)`（[L5298-L5311](../../../raw/current/scripts/actions.lua#L5298-L5311)）。页面自己已注明这些组件实现不在脚本快照；据调用参数猜 true 一定跳过特效，同样需要被调用实现作证。
- 建议改为“可见 REPLATE 段未直接生成这两名 FX；Salt／Replate 内部是否生成与参数语义未核验”。保留盐渍态传递的可见事实，避免把局部无 SpawnPrefab 扩展为整条不透明调用链的负向结论。

## R08｜P2：火系页漏官方宿主名称检索词，现有名称依据也缺锚点

- 页面位置：[title／aliases L2-L7](../../../wiki/reference/prefabs/firesplash_fx.md#L2-L7)、[名称说明 L36](../../../wiki/reference/prefabs/firesplash_fx.md#L36)、[索引 L29](../../../wiki/reference/prefabs/fx-table-index.md#L29)。标题与正文全用“蜻蜓龙”，aliases 没有官方名“龙蝇”。
- `.po` 的 `STRINGS.NAMES.DRAGONFLY` 明确译为“龙蝇”（[L342199-L342202](../../../raw/current/scripts/languages/chinese_s.po#L342199-L342202)）。六个 FX 本身无 NAMES 键，并不妨碍宿主使用官方名称；“效果描述非官方译名”不能替代宿主名核验。
- 同页已有 `.po` sources，且 L36 正确记了 `WAXWELL_SHADOWSTRIKER`＝“暗影”，但全文没有其证据锚点（实际 [L351914-L351917](../../../raw/current/scripts/languages/chinese_s.po#L351914-L351917)）。
- 建议以“龙蝇”作为正式宿主名称，保留“蜻蜓龙”检索别名；同步标题、摘要、aliases、正文和索引，并补两个已核验名称键的锚点。不是要求给无译名 FX 编造官方名。

## R09｜P2：cavein_debris 与已建的专属落石尘土页发生 aliases 归属冲突

- 本轮 [sinkhole aliases L7](../../../wiki/reference/prefabs/sinkhole_spawn_fx.md#L7) 正确收录 `cavein_debris`，但另一页 [cavein_dust_low aliases L7](../../../wiki/reference/prefabs/cavein_dust_low.md#L7) 也收录同一注册名。
- 后者自己声明主要范围是 9 个专属注册，且把 cavein_debris 标成“清单外并入”（[边界 L24](../../../wiki/reference/prefabs/cavein_dust_low.md#L24)、[简条 L106](../../../wiki/reference/prefabs/cavein_dust_low.md#L106)）。源码实际由通用表注册（[fx L1687-L1692](../../../raw/current/scripts/fx.lua#L1687-L1692)），应以本轮 sinkhole 页为主要知识页。
- 建议从 cavein_dust_low 的 aliases 移除该 ASCII 注册名，简条改为关联导航并链接 sinkhole 页，保留它与落石的辨析价值；不要反过来删除本轮页的正确归属。此项只扩展到已经确认的跨页冲突，未重新复审另一页的 9 个注册。

## 非阻断证据精度提醒

- firesplash 导航与陷阱多次引用 groundpounder L161-L163，但蜻蜓龙实际调用 UseRingMode，活跃生成代码在 [DestroyRing L284-L289](../../../raw/current/scripts/components/groundpounder.lua#L284-L289)，分支调度在 [L331-L354](../../../raw/current/scripts/components/groundpounder.lua#L331-L354)。两分支目前地面门禁等价，因此不另计事实错误；修页时换成活跃分支证据更清楚。“单次 GroundPound 最多 8 枚”按实际配置与点数算法核对成立，勿误改回逐环一枚。
- lavaarena 注册表 L31 对 `_2`、`_3` 的引用结束在 build 行，未包含实际 anim 行 L917／L923，且第一个显示标签从 L913 起、实际链接从 L914 起。建议顺带对齐显示范围并包含动画值，不另计一个问题。

## 本轮实际验证与边界

| 检查 | 结果 |
|---|---|
| validate_page，5 页＋两叶索引，带 --check-indexes | 7 个目标页面 passed=true、issues=0；整体退出 1，仅另 5 个他方在建页 axe／glasscutter／hammer／pickaxe／shovel 缺登记。本批 5 页均恰有一个正确叶级登记，返回链匹配；不报告全局导航通过 |
| validate_source_links --strict，5 页 | 132 处行号引用、111 个去重目标、46 个源码文件；error=0、warning=0、skipped=0、未截断，退出 0 |
| validate_source_hashes check --fresh-hours 0，5 页 | 54 组已声明依赖全部 unchanged；sources_changed／invalid／unrecorded=0，退出 0；未 record，未写基线 |
| 注册／名称核对 | 27 名均在 fx.lua 注册、均进入本页正文与 aliases；逐名查 PO 均无同名 NAMES 显示键；跨 Prefab aliases 发现 cavein_debris 冲突 1 项（R09），其余 26 名无冲突 |
| 标题原文 read | 地陷“复用地陷尘爆的\"中心＋环\"图样”、glass“glass_fx：沙晶玻璃化转场”、暴食“复用暴食出场／撒盐特效”、熔炉“复用\"周期父挂的环境表现\"与\"随机拼名变体\"”、火系“常见陷阱与限制”5 次退出 0 |
| 检索方式 | 直接遍历 Markdown frontmatter／正文与叶索引，未运行 search，未依赖或重建查询缓存；不把章节读取成功当作语义通过 |

生成点检索覆盖 `raw/current/scripts` 根级及所有子目录 Lua，完整捕获候选后阅读相关上下文，不用截断输出作负向依据。先查 27 个名字及共同前缀，再追 `HandleDugGround`、`spawnfirefx`、`portal_.*player`、`salting`、`shadowstrike` 与攻击范围赋值。`HandleDugGround` 有定义＋两调用，`spawnfirefx` 仅定义；动态烟名、地陷后缀拼接及 shadowstrike 二选一已追至实际 SpawnPrefab。公开脚本中未定位到事件包 FX 的直接生成，只能描述本快照检索范围，不能据此证明包内的时机或完整无特效链。

四段 Lua 示例仅核对可见前提与实际字段，本轮未重跑 Lua 解析器，未运行 DST，未验证动画时长、音效实听或引擎父子删除。机械检查通过不能抵消上述 P1／P2。

报告落盘后检查 65 个本地链接的路径与物理行区间均有效；58 个预先记录的页面／索引／基线／BUILD／源码输入哈希未变。`git -c core.safecrlf=false diff --check` 通过。检视期间另出现他方未跟踪的 `multitool_axe_pickaxe.md`，保留其生成进度；上表全局导航数量与问题是执行校验时的快照，不据此宣称当前全库已收口。

## 修复与下一轮

优先修 R01，再修父挂／时序／不透明调用的边界、来源声明和注册归属；名称与数值同步修改相应摘要、索引、案例和验证记录。变更 sources 的页经实际核验后更新相应基线；未核验页不顺手 record。修后复跑本批格式、严格源码链接、来源哈希与必要标题读取，并定向复验 cavein_debris 仅一个主要 aliases 归属。

累计已独立检视 **37／158 页，余 121 页**，这里“已检视”不等于全部验收；上一轮 R08 与本轮问题仍待修。此轮不启动下一批，不把既有 accepted 改成独立通过；下一次新轮次按原台账批 7 继续。

## 2026-11-06 修后独立复验

实读 BUILD 仍为 26335394。检查本批 5 页、R09 关联页 cavein_dust_low，以及上一轮 R08 的浮渣／鱼人两页，共 8 页；保留其他执行者新增的装备页、索引、日志与基线增量。下列结果来自当前文件与源码，不沿用修复方日志作验收。

| 原项 | 复验结果 |
|---|---|
| R01 | 原缺口已补：SG 内 SetRange(5) 门禁与 shadowprotector 的 timer 安装依据已列入，未再把范围赋值归到不可见事件包；修订中新增的泛化错误另列 R10 |
| R02 | 已覆盖：熔炉案例 L52 区分父挂移动、动画／代理回收与未核验的引擎级联，补 entityscript 来源与锚点 |
| R03 | 已覆盖：地陷三项原漏来源、暴食的 prefabs/fx.lua 均已入 sources；8 页直接引用对照零漏项 |
| R04 | 已覆盖：洞穴中心跟玩家后仍按 spread 偏移、各轴向外 0～10%、眼魔中心不缩放与环 0.6 起加 0～0.2 均已修正，案例说明同步 |
| R05 | 已覆盖：自动翻地的 helper 调用、antlionhat 装备／卸装条件、无草皮映射门禁与另出的 turf_smoke_fx 已列明；新增 autoterraformer／hats 来源及基线一致 |
| R06 | 已覆盖：glass 快速判断、流程、陷阱、验证记录均撤回确定相对时序，GLASS_TIME 锚点改 L29；摘要不再预设特效必在换物之前 |
| R07 | 已覆盖：REPLATE 行改为可见段未直接生成，Salt 内部与 true 参数保留未核验边界，验证记录同步 |
| R08 | 已覆盖：title／摘要／aliases／正文／角色战斗索引采用官方宿主名“龙蝇”，保留“蜻蜓龙”检索别名，DRAGONFLY 与 WAXWELL_SHADOWSTRIKER 两个 PO 键锚点均正确 |
| R09 | 已覆盖：cavein_dust_low 已移除 cavein_debris alias，边界与简条改为关联到 sinkhole 主要页；扫描现有 Prefab 页该注册名仅有 sinkhole 一处 aliases 归属 |

上一轮 R08 的四项来源依赖也已闭环，收口记录见 [第 4 轮报告](fx-review-round04-20261105.md)。原来的非阻断证据精度提醒未全部采纳，不因此升级为本批阻断问题。

### R10｜P2：修订把其他 MakeMinion 变体一概写成无 timer，漏了 shadowworker

- 页面位置：[firesplash 验证记录 L83](../../../wiki/reference/prefabs/firesplash_fx.md#L83) 新增“其余 MakeMinion 变体无 timer，不走 lunge”。这超出了已核验的 shadowprotector 正向结论，且其中“无 timer”有直接反例。
- `workerfn` 明确调用 `MakeOblivionSeeker`（[shadowwaxwell L357-L367](../../../raw/current/scripts/prefabs/shadowwaxwell.lua#L357-L367)），helper 安装 timer（[L122-L128](../../../raw/current/scripts/prefabs/shadowwaxwell.lua#L122-L128)）；返回表把 workerfn 传给 `MakeMinion("shadowworker", ...)`（[L668-L669](../../../raw/current/scripts/prefabs/shadowwaxwell.lua#L668-L669)）。共同工厂安装 SG 后执行专属初始化（[L547-L568](../../../raw/current/scripts/prefabs/shadowwaxwell.lua#L547-L568)）。所以 shadowworker 也是有 timer 的变体，不能被这句全称排除。
- **有 timer 不等于实际战斗必然可达。** workerfn 同时设置 `SetKeepTargetFunction(nokeeptargetfn)`，本轮没有为工作分身补证完整目标／攻击链，也不把它改判成必然能突袭。SG 的范围赋值还受状态进入与 shadowstrike_cd 条件控制（[L339-L361](../../../raw/current/scripts/stategraphs/SGshadowwaxwell.lua#L339-L361)）。
- 建议只删改 L83 这句过宽结论，保留已确认的 shadowprotector 正向接线；若提及 shadowworker，则说明它同样安装 timer，而其他变体的实际突袭可达性未逐一核验。导航／陷阱中“无 timer 的变体不满足该范围赋值门禁”可保留条件式表述，不把它解释成“除 protector 外全无 timer”。无需为此扩展建页或补写完整工作分身战斗机制。

### 本次实际检查与边界

- `validate_page --check-indexes`：8 页＋两索引 **10／10 passed**，全局导航通过，1119 知识页／34 索引、问题 0，退出 0。
- `validate_source_links --strict`：**301 行引用、230 个去重目标、69 个源码文件，0 error／0 warning／0 skipped、未截断**，退出 0。
- `validate_source_hashes check --fresh-hours 0`：**92 组已声明依赖全部 unchanged**，changed／sources_changed／invalid／unrecorded 0，退出 0；未 record。
- 8 页正文直接来源与 sources 对照无漏项；上一轮浮渣 entityscript 实现依赖也已声明。`cavein_debris` aliases 主归属只有 sinkhole 页。
- 七次标题原文 read 退出 0：火系陷阱、地陷导航、玻璃化案例、熔炉案例、暴食导航、浮渣案例、鱼人陷阱。没有运行 search 或重建缓存。
- 按改动核对门禁、宿主与清理等结论，复用未变化源码的上次已核验事实；未运行游戏、未重跑未改动 Lua 示例解析，不宣称运行时通过。只更新两轮报告与日志，知识页、索引、基线及他方改动均保留。
- 报告落盘复查：第 4／5 轮报告分别 71／72 个本地链接路径与物理行区间有效，81 个受保护输入哈希未变；`git -c core.safecrlf=false diff --check` 通过。

当前第 4 轮可以收口；第 5 轮仍待 R10 这一项 P2 修订后复验，P1 为 0。累计已独立检视仍为 **37／158 页，余 121 页**，本次复验不增加页数、不启动下一轮。
