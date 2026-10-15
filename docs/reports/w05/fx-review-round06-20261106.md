# FX 独立检视第 6 轮（原生成批 7）

初检日期：2026-11-06。初检 P1×0、P2×4，另有 P3×2 非阻断建议。**同日修后独立复验：R01～R06 已覆盖，未发现新的 P1/P2，第 6 轮可收口。** 下文初检保留历史，以文末复验为当前结论。仅只读检视知识页，写本报告并追加日志；不修知识页、不 record、不改索引、PLAN、台账 accepted 或 Skill／工具。未运行游戏。

## 范围与基准

- K：`D:/Code/dst-wiki-develop/dst-wiki`；S：`D:/Code/dst-wiki-develop/dst-wiki-skill`；T：`D:/Code/dst-wiki-develop/dst-toolbox`。
- 实读 `raw/current/BUILD.txt`：`26335394`；源码根 K 的 `raw/current/scripts`。K HEAD：`97afb4e9e6ca05390558769926f1f9c8598d0f70`。
- 取 [覆盖台账原批 7](fx-coverage-ledger-20261101.md#L1016) 的 F48／F47／F26，3 页、27 注册；既有 accepted 和复审自报仅作定位线索。本轮不顺带复验上轮待修项，不替其改变结论。
- 已检查 Git 状态，保留上一轮修订与他方装备扩库改动。S／T 工作区干净；不重复认领生成任务。

| 页面 | 注册数 | 物理行数／UTF-8 字节 | 结果 |
|---|---:|---:|---|
| [fx_book_moon](../../../wiki/reference/prefabs/fx_book_moon.md) | 18 | 116／15,808 | P2：R01、R04 |
| [winters_feast_depletefood](../../../wiki/reference/prefabs/winters_feast_depletefood.md) | 2 | 73／8,011 | 未发现 P1/P2；P3：R06 |
| [carnival_confetti_fx](../../../wiki/reference/prefabs/carnival_confetti_fx.md) | 7 | 88／11,775 | P2：R02、R03；P3：R05 |

物理行数按 LF 分行、文件末尾换行不另计空行；三页分别为 complex／simple／默认 standard，均在当前预算内。

## 待修 P2

### R01｜普通 READ 的结算是第 50 帧，不是第 44 帧

- 页面：[fx_book_moon L74](../../../wiki/reference/prefabs/fx_book_moon.md#L74)、[L112](../../../wiki/reference/prefabs/fx_book_moon.md#L112) 把 `book2` 的第 44 帧写成普通 READ 的 `PerformBufferedAction` 窗口。
- 证据：[SGwilson L10267-L10312](../../../raw/current/scripts/stategraphs/SGwilson.lua#L10267-L10312)：第 44 帧先定义 `fn50`。仅 `repeatcast` 为真时当场执行；普通分支把它存入 `statemem.fn50`，到第 50 帧才调用。`PerformBufferedAction` 又受 `not earlycast` 限制。
- 分派也应收窄：[READ L1046-L1051](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1046-L1051) 在 cookbook／翻阅之外选 `book`；[book_repeatcast L10032-L10035](../../../raw/current/scripts/stategraphs/SGwilson.lua#L10032-L10035) 的入口另见 [CASTAOE L1316-L1320](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1316-L1320)，不能把三个状态都表述成 READ ActionHandler 的选择结果。
- 建议：本页只需写普通正式 READ 经 `book→book2`，在第 50 帧进行动作结算；第 44 帧只是准备回调。若保留其他模式，分别注明入口与 `repeatcast`／`earlycast` 门禁，不把其他施法模式直接套到这 11 本普通书。同步验证记录和锚点，延伸到第 50 帧执行处。
- 关联说明：[reader L46-L58](../../../raw/current/scripts/components/reader.lua#L46-L58) 的 aspiring 分支是 `OnPeruse`，本页已声明范围为正式阅读；L73 的链条可顺手补“正式阅读分支”，不必展开翻阅机制。

### R02｜高尔夫的 0.5 秒是累计门槛，球体另走停稳检查

- 页面：[carnival_confetti_fx L46](../../../wiki/reference/prefabs/carnival_confetti_fx.md#L46) 写“组件 0.5 秒周期 CheckTeleport”。
- 证据：[golfpropitem L11-L13](../../../raw/current/scripts/components/golfpropitem.lua#L11-L13)、[L66-L77](../../../raw/current/scripts/components/golfpropitem.lua#L66-L77)：初值为 `math.random(15)*FRAMES`，`OnUpdate` 每次累加 dt，累计小于 0.5 时返回，达到后没有重置／减去 0.5。因此不能描述为固定每 0.5 秒检查一次。
- 球体消费路径还不同：[SetupGolfBall L123-L133](../../../raw/current/scripts/prefabs/carnivalgame_golf_tee.lua#L123-L133) 安装组件、设置边界后立即 `StopUpdating()`；[golfball L97-L104](../../../raw/current/scripts/prefabs/carnivalgame_golfball.lua#L97-L104) 在落地滚动、无球洞目标且速度平方小于 0.01、没有 frictiondelay 的停稳分支显式调 `CheckTeleport()`。FX 的实际生成仍在 [TeleportBack L53-L56](../../../raw/current/scripts/components/golfpropitem.lua#L53-L56)。
- 建议：这一行分清组件自动更新与球体显式调用，两者共用出界检查和回传 FX；不要承诺固定 0.5 秒响应，也不要把球座安装组件的锚点当作球体持续轮询的证据。
- 同一错误已存在于关联 [golfpropitem 组件页 L21](../../../wiki/reference/components/golfpropitem.md#L21) 与 [L54](../../../wiki/reference/components/golfpropitem.md#L54)。这是同一问题的同步修订面，本轮不自动修改或扩为组件页全面复审；修复时应一并清除“每 0.5 秒／轮询间隔 0.5 秒”说法。两页各自已有对应来源文件，按实质修订范围更新记录。

### R03｜狂欢节正文直接引用 prefabs/fx.lua，但 sources 未列

- 页面：[carnival_confetti_fx L10](../../../wiki/reference/prefabs/carnival_confetti_fx.md#L10) 的 `sources` 无 `prefabs/fx.lua`，但 [L23](../../../wiki/reference/prefabs/carnival_confetti_fx.md#L23) 直接引用其 [L127-L136](../../../raw/current/scripts/prefabs/fx.lua#L127-L136)，用于“专服只建代理”的已采纳结论。
- 建议：将该实际支撑文件补入 `sources`，核验工厂分支后按现有规则更新本页哈希基线；或把该处实际采用的公共协议完全交由已核验机制页维护，删除直接源码结论／引用中的不一致。前者改动更小。
- 当前哈希 check 的 24 组 unchanged 只证明已声明来源未漂移，不能发现漏列文件。`validate_source_links` 证明路径／行区间有效，也不检查与 sources 对齐。

### R04｜主模拟与专服条件混用，漏掉非专服主机本地表现

- 页面：[fx_book_moon L28](../../../wiki/reference/prefabs/fx_book_moon.md#L28) 写“主模拟侧只存在隐形网络代理”，[L71](../../../wiki/reference/prefabs/fx_book_moon.md#L71) 再写“主模拟只建代理”。
- 证据：[prefabs/fx.lua L121-L157](../../../raw/current/scripts/prefabs/fx.lua#L121-L157)：调度 `startfx` 的条件是 `not TheNet:IsDedicated()`，发生在 `TheWorld.ismastersim` 返回分界之前。非专服主机同时是主模拟，也会调度本地表现。主模拟条件管的是代理的 `persists=false` 和 1 秒回收。
- 本地实体本身见 [startfx L18-L37](../../../raw/current/scripts/prefabs/fx.lua#L18-L37)，确实不带 Network，且与代理是两个实体。
- 建议：改为“主模拟负责网络代理及其 1 秒回收；各非专服端（含非专服主机）另建本地表现；专服不建本地表现”。同步构造段与导航表，保留纯表现与网络代理的区分，不改成“主模拟不创建 FX”。

## 非阻断 P3

### R05｜狂欢节验证记录仍是旧预算和未登记状态

- [carnival_confetti_fx L86](../../../wiki/reference/prefabs/carnival_confetti_fx.md#L86) 仍记旧的 standard 200 行／16 KiB，以及“叶级索引登记待办，未执行”。当前 [fx-table-index L26](../../../wiki/reference/prefabs/fx-table-index.md#L26) 已有登记，当前 standard 上限为 240 行／24 KiB。
- 建议：历史检查若需保留，明确标成当时结果；另补本次实际验证和已登记状态，避免把旧结论当当前状态。不要只修改历史数字伪装成当时已执行新检查。

### R06｜“普通食物静默跳过”宜限定为组件入口

- [winters_feast_depletefood L24](../../../wiki/reference/prefabs/winters_feast_depletefood.md#L24)、[L41](../../../wiki/reference/prefabs/winters_feast_depletefood.md#L41)、[L64](../../../wiki/reference/prefabs/winters_feast_depletefood.md#L64) 按缺 finiteuses 解释普通食物不生成这两名，组件的 [DepleteFood L65-L81](../../../raw/current/scripts/components/wintersfeasttable.lua#L65-L81) 确实有此守卫，结论本身不阻断。
- 宿主还有更早的分界：[餐桌 L214-L235](../../../raw/current/scripts/prefabs/table_winters_feast.lua#L214-L235) 对非盛宴食物调用 RefuseFood、设 `canfeast=false`；[RefuseFood L121-L133](../../../raw/current/scripts/prefabs/table_winters_feast.lua#L121-L133) 播 bump 并安排弹出，不能把完整投食过程概括为静默。建议补“普通投食先被宿主拒食；若直接进入组件入口，无 finiteuses 也不走消耗回调”，不必在 FX 页展开投食流程。

## 已确认与检视边界

- 27 名均在 `fx.lua` 存在、写入主要页 aliases 与正文；对 Prefab 目录 aliases 精确扫描，未发现这些注册名跨页重复归属。两叶索引各有登记，页尾包含实际登记叶索引。
- 读书 18 条目字段、11 本 def、路径 A／B、骑乘回落与 fish 的逐帧 alpha 递减已核对。research_station 确实生成自己的两名，与 temperature 只是选名方式同构；未发现用 temperature 特效替代的代码。bees 路径 B 通过嵌套延迟任务生成，不承诺与动作结算同帧。玩法效果不由这些 FX 结算。
- 冬季盛宴两条目、30% 概率按桌组进食人数逐名判定、两级随机延迟、finiteuses 分支、餐桌自身终局音效均有依据。`DepleteFood(feasters)` 虽收到人数实参，但并不直接消费该参数；实际人数在餐桌 OnDepleteFood 中重新按组统计，没有按人数直接多扣 uses 的结论。
- 狂欢节七条目、炮的激活／投入与连锁转发、雕塑赛季动态拼名、小鸡 SG 的第 28 帧、射击弹体命中与回合结束自毁均已核对。`projectile_OnHit` 的“命中”是组件回调语义，落地也可经 `complexprojectile:Hit()` 触发；不据此推断必须击中活物或造成伤害。
- 生成点检索覆盖 `raw/current/scripts` 根及所有子目录：完整注册名／公共前缀，加 SpawnPrefab／GetPooledFx／SpawnChild／ReplacePrefab 与拼接候选定位。补充候选检索分别为读书 22 行、狂欢节 33 行、盛宴 6 行，逐批查看候选、不用 head 截断；依赖表、动画 build 复用与真正生成已分开。未发现需在本轮追加的新生成链，不将检索未命中扩为引擎层“不存在”。
- 对 27 名构造精确 `STRINGS.NAMES.<大写注册名>`，定向扫描 `languages/chinese_s.po` 与 `strings.lua`，显示名键均未命中；页面明确采用效果描述，未强补官方译名，也不因负向查名把 PO 加进来源。
- 未核验动画包实际内容／时长、客户端预测与联网实测；不从 SetParent 推断引擎级联。本轮未重跑既有 Lua 示例语法探针，未生成临时示例文件。

## 定向校验结果

| 检查 | 实际结果 |
|---|---|
| validate_page --check-indexes，3 页 | 所选 3/3 页面格式通过；全库 navigation 未通过，退出 1 |
| navigation 失败范围 | 本次运行见 1122 知识页／34 索引，3 个 `registration: got 0`：他方在建 `fumarole-tools`、`lunarplant-tools`、`spear`；本批三页无导航问题，不自动登记他方草稿 |
| validate_source_links --strict，3 页 | 101 个源码行引用、91 个去重目标、18 个源码文件；0 error／0 warning，skipped_unsupported=3，truncated=false，退出 0 |
| validate_source_hashes check --fresh-hours 0，3 页 | 已声明来源 24/24 unchanged；0 changed／sources_changed／invalid／unrecorded，退出 0；未 record |
| sources 与正文直链人工比对 | 仅 carnival_confetti_fx 漏 prefabs/fx.lua，即 R03；另外两页正文直链无漏列 |
| 标题原文 read | 读书“生成路径与骑乘选名”、盛宴“关联机制导航”、狂欢节“生成者导航（触发窗口与落点）”均退出 0 |
| 检索／归属 | 使用当前 Markdown 标题、摘要、aliases 与索引；27 名主要 aliases 归属唯一。未用 search 缓存、未重建缓存，不将 read 成功等同语义正确 |
| 工作区与报告自查 | 27 个开始时留存哈希的本批页面／索引／来源／基线／BUILD 输入均未变化；报告 34 个本地链接及所标行区间有效，git diff --check 通过 |

## 收尾建议

优先修 R01～R04，再按需要处理 R05～R06；R02 的关联组件旧说法同步修订。修改 sources 的 R03 按规则刷新该页基线，其余页面按实际重新核验范围记录，不批量 record 全库。修后重跑受影响页、来源引用、哈希与章节 read；全库导航的外部在建状态单列报告。

本轮新增独立检视 3 页，累计 **40／158 页**，余 **118 页**；该数字是已检视量，包含待修页，不等于全部已验收。下一轮对应原批 8（F21～F25，5 页、18 注册），不在本轮展开。

## 2026-11-06 修后独立复验与收口

复验范围为本轮 3 篇 Prefab 页，加 R02 关联的 `reference/components/golfpropitem.md` 两处修订。实读 BUILD 仍为 `26335394`；重新核对修改 diff 与相关源码，未将作者日志的“已处置”当作验收证据。上一轮 R10 不在本次复验范围，不据本轮结果代其收口。

| 原问题 | 当前证据与结论 |
|---|---|
| R01，读书结算与分派 | [fx_book_moon L74](../../../wiki/reference/prefabs/fx_book_moon.md#L74)、[L112](../../../wiki/reference/prefabs/fx_book_moon.md#L112) 已改普通正式 READ 第 50 帧结算，第 44 帧准备 fn50；READ 与 CASTAOE 的 repeatcast 入口分开，证据延伸到 SGwilson 第 50 帧调用处。已覆盖 |
| R02，高尔夫累计与球体路径 | [carnival_confetti_fx L46](../../../wiki/reference/prefabs/carnival_confetti_fx.md#L46) 已区分累计门槛、达到后不重置、球体 StopUpdating 与停稳显式调用；[组件页 L21](../../../wiki/reference/components/golfpropitem.md#L21)、[L54](../../../wiki/reference/components/golfpropitem.md#L54) 已同步清除固定 0.5 秒轮询说法。已覆盖 |
| R03，sources 漏列 | [carnival_confetti_fx L10](../../../wiki/reference/prefabs/carnival_confetti_fx.md#L10) 已补 prefabs/fx.lua；本次只读 check 确认 sources 与基线一致，29 组比较含该新增来源。已覆盖，复验代理未 record |
| R04，主模拟与专服 | [fx_book_moon L28](../../../wiki/reference/prefabs/fx_book_moon.md#L28)、[L71](../../../wiki/reference/prefabs/fx_book_moon.md#L71) 已区分主模拟的代理回收、各非专服端（含主机）的本地表现与专服无本地表现。已覆盖 |
| R05，历史验证状态 | [carnival_confetti_fx L86](../../../wiki/reference/prefabs/carnival_confetti_fx.md#L86) 明确旧预算与待登记为历史状态，并补当前预算和实际叶索引登记。已采纳 |
| R06，普通食物边界 | [winters_feast_depletefood L24](../../../wiki/reference/prefabs/winters_feast_depletefood.md#L24)、[L41](../../../wiki/reference/prefabs/winters_feast_depletefood.md#L41)、[L64](../../../wiki/reference/prefabs/winters_feast_depletefood.md#L64) 已补宿主拒食／弹出与组件无 finiteuses 守卫的分工，并给实际来源锚点。已采纳 |

本次独立执行结果：

- `validate_page --check-indexes`：4/4 页面通过，全库导航通过；1122 知识页、34 索引、0 navigation issues，退出 0。初检时的他方 3 页缺登记现已消失；本代理未修改其登记。
- `validate_source_links --strict`：4 页、125 个源码行引用、111 个去重目标、19 个源码文件，0 error／0 warning，skipped_unsupported=4、truncated=false，退出 0。
- `validate_source_hashes check --fresh-hours 0`：29/29 已声明来源 unchanged；changed／sources_changed／invalid／unrecorded 均为 0，退出 0。本次不写基线。
- 4 页正文源码直链均已列入各自 sources，无新漏列；当前尺寸为读书 116 行／16,467 B、狂欢节 88 行／12,225 B、盛宴 73 行／8,628 B、组件 67 行／6,339 B，均在原档预算内。
- 用标题原文 read 读书“关联机制导航”“注册与构造”、狂欢节“生成者导航（触发窗口与落点）”、盛宴“常见陷阱与限制”、组件“典型接入”，5 次均退出 0；未使用 search 缓存，未重建索引缓存。

非阻断维护提醒：4 个修订页的 `updated` 仍为 2026-11-01；组件页的验证记录保留 10 月整页审阅与 11 月 1 日无语义修订沿用，尚未追加本次局部修订记录。建议后续顺手同步实际修订日期，并追加局部核验范围，保留既有历史；不据此升级为 P1/P2 或重开已修技术问题。

**本轮当前 P1×0、P2×0，可收口。** 未运行游戏，未重跑未改动的 Lua 示例；仅更新本报告和日志，保留其他 Agent 的修改。重验不增加已检视页数，累计仍为 40／158 页、余 118 页；后续原批 8 另起下一轮。
