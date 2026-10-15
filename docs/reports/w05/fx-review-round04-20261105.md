# FX 独立检视第 4 轮（原生成批 5）

初检日期：2026-11-05。初检结论：需修复 P1×2、P2×5。**2026-11-06 再次复验：R08 来源漏项已闭环，R01～R08 均可收口，当前待修 P1／P2 为 0。** 下文初检清单与早先复验保留历史，当前结论以文末收口记录为准。本报告只读复审知识页，不修页、不 record、不改索引、PLAN 或台账 accepted 状态。游戏未运行。

## 范围与基准

- K：`D:/Code/dst-wiki-develop/dst-wiki`；S：`D:/Code/dst-wiki-develop/dst-wiki-skill`；T：`D:/Code/dst-wiki-develop/dst-toolbox`。
- 实读 `raw/current/BUILD.txt`：`26335394`；源码根为 K 的 `raw/current/scripts`。K HEAD：`97afb4e9e6ca05390558769926f1f9c8598d0f70`。开始时 K 仅 `raw/current/BUILD.txt` 有未提交改动，S／T status 干净，均保留。
- 范围来自 [覆盖台账原批 5](fx-coverage-ledger-20261101.md#L1008)：5 页、20 注册。原作者复审和 accepted 只作线索，本轮重新核验。

| 页面 | 注册数 | 行数／UTF-8 字节 | 本轮待修 |
|---|---:|---:|---|
| [ink_puddle_land](../../../wiki/reference/prefabs/ink_puddle_land.md) | 4 | 79／8,202 | R02 来源漏项；未发现其他 P1／P2 |
| [flotsam_puddle](../../../wiki/reference/prefabs/flotsam_puddle.md) | 2 | 79／8,581 | R02～R05 |
| [merm_splash](../../../wiki/reference/prefabs/merm_splash.md) | 3 | 87／10,337 | R01、R02 |
| [bee_poof_big](../../../wiki/reference/prefabs/bee_poof_big.md) | 4 | 82／11,892 | 未发现 P1／P2 |
| [crab_king_bubble1](../../../wiki/reference/prefabs/crab_king_bubble1.md) | 7 | 88／15,417 | R06、R07 |

行数按 LF 物理分行，含文件末尾空项；五页均在 standard 预算内。正文、注册配置、直接生成回调、必要宿主／事件入口和清理均逐页核验；下列问题编号按修复任务分组，不把一项来源问题的多个位置拆成多个计数。

## R01｜P1：鱼人王出现的世界事件没有按文中链路送入 buff 状态

- 页面位置：[summary L6](../../../wiki/reference/prefabs/merm_splash.md#L6)、[快速判断 L20-L23](../../../wiki/reference/prefabs/merm_splash.md#L20-L23)、[导航 L41](../../../wiki/reference/prefabs/merm_splash.md#L41)、[全体广播断言 L74](../../../wiki/reference/prefabs/merm_splash.md#L74)；验证记录 L82 及 [索引 L31](../../../wiki/reference/prefabs/fx-table-index.md#L31) 也沿用了该结论。
- 现文把 `TheWorld:PushEvent("onmermkingcreated_anywhere")` 直接连到 SGmerm 的同名 EventHandler，声称各分片鱼人全体出水花。两端监听的实体和转发事件名没有接上。
- 实际链路：watcher 在已有王状态 false→true 时向 **TheWorld** 推事件（[L17-L24](../../../raw/current/scripts/components/shard_mermkingwatcher.lua#L17-L24)）；鱼人以 TheWorld 为 source 安装回调（[守卫 L1322-L1323](../../../raw/current/scripts/prefabs/merm.lua#L1322-L1323)、[普通鱼人 L1484-L1485](../../../raw/current/scripts/prefabs/merm.lua#L1484-L1485)），回调延迟升级属性，随后向鱼人自身推的是 **`onmermkingcreated`**（[守卫 L1214-L1220](../../../raw/current/scripts/prefabs/merm.lua#L1214-L1220)、[普通鱼人 L1443-L1449](../../../raw/current/scripts/prefabs/merm.lua#L1443-L1449)）。SGmerm 却监听 **`onmermkingcreated_anywhere`**（[L160-L162](../../../raw/current/scripts/stategraphs/SGmerm.lua#L160-L162)）。
- `EntityScript:PushEvent_Internal` 先执行 source 上的监听回调，再把事件送入 **source 自己的** SG，不会自动转交每个监听实体的 SG（[L1312-L1335](../../../raw/current/scripts/entityscript.lua#L1312-L1335)）。因此“世界出现王→所有鱼人 buff 出水花”不能由已查 Lua 链路支持。属性升级与水花状态必须分开描述。
- 建议：保留 buff 状态本身可生成 `merm_splash` 的事实（[L430-L445](../../../raw/current/scripts/stategraphs/SGmerm.lua#L430-L445)），明确本快照事件转发名不一致，未找到从上述世界事件进入 buff 的完整链；同步撤回摘要、快速判断、导航、陷阱、索引和验证记录中的自动广播水花结论。若要补接入说明，区分直接向鱼人推送 SG 所监听的事件与原版现有世界回调；不修改游戏源码。
- 本轮对上述两种事件名和 `GoToState("buff")` 在 scripts 全域 Lua 完整捕获检索，另读 SG 实例构造／分派，未找到补足该缺口的转发。未运行游戏，不把此结论扩展成所有运行环境下绝不可能出水花。

## R02｜P2：三页 sources 合计漏 5 个页面—文件依赖

| 页面位置 | 已使用但未声明 | 依据／处理 |
|---|---|---|
| [ink_puddle_land L10](../../../wiki/reference/prefabs/ink_puddle_land.md#L10) | `prefabs/fx.lua` | [L70](../../../wiki/reference/prefabs/ink_puddle_land.md#L70) 直接引用其播放实现；补 sources |
| [flotsam_puddle L10](../../../wiki/reference/prefabs/flotsam_puddle.md#L10) | `prefabs/fx.lua`、`languages/chinese_s.po` | [L40](../../../wiki/reference/prefabs/flotsam_puddle.md#L40)／L70 直接引工厂；[L32](../../../wiki/reference/prefabs/flotsam_puddle.md#L32)／L74 声称已核验 .po；补 sources |
| [merm_splash L10](../../../wiki/reference/prefabs/merm_splash.md#L10) | `prefabs/fx.lua`、`languages/chinese_s.po` | [L78](../../../wiki/reference/prefabs/merm_splash.md#L78) 直接引工厂；[L33](../../../wiki/reference/prefabs/merm_splash.md#L33)／L82 声称已核验 .po；补 sources |

直接正文链接与 frontmatter 对照确认三页各缺一项工厂文件；名称记录另确认两页缺 .po。Skill 的 source-types“名称与检索”规定实际用于核验名称的 .po 计入 sources。负向查无键不伪造不存在的行锚点，在正文保留检索键／范围即可。本轮重新查过 20 个注册名，确无对应 NAMES 键；不是要求改成官方名。

修复 R01／R03～R07 若新增其他实际支撑文件，也应同步声明。由修复方对真正重新核验的页面按现有规则刷新对应基线；本轮的 40 组 unchanged 只覆盖已声明来源，不能证明上述漏项已经受版本追踪。

## R03｜P2：浮渣水洼把 SetParent 写成已核验的随父删除

- 页面位置：[导航 L45](../../../wiki/reference/prefabs/flotsam_puddle.md#L45)、[案例 L52-L53](../../../wiki/reference/prefabs/flotsam_puddle.md#L52-L53)、[陷阱 L69](../../../wiki/reference/prefabs/flotsam_puddle.md#L69)。现文断言宿主删除后水洼消失、“子层级随父清理”。
- `playlandfx` 只对 FX **代理**调用 `entity:SetParent`，无 `AddChild`、引用保存或宿主移除回调（[L108-L114](../../../raw/current/scripts/prefabs/oceanfishableflotsam.lua#L108-L114)）。工厂创建的本地动画再与代理取同一父级（[L18-L37](../../../raw/current/scripts/prefabs/fx.lua#L18-L37)）。这支持引擎父挂与随动关系。
- 可见 Lua 的 children 递归清理由 `AddChild` 登记（[L921-L935](../../../raw/current/scripts/entityscript.lua#L921-L935)），`Remove` 遍历该表（[L1779-L1783](../../../raw/current/scripts/entityscript.lua#L1779-L1783)）；上述路径未登记该表，不能将它作为随父删除依据。引擎级联没有在本轮得到证据，也没有游戏实测。
- 建议：将已核验清理写为代理服务端 1 秒任务（[L156-L157](../../../raw/current/scripts/prefabs/fx.lua#L156-L157)）与本地动画 animover（[L98-L102](../../../raw/current/scripts/prefabs/fx.lua#L98-L102)）；随宿主删除的引擎行为保留待核验。同步改“随物品存在”的接入承诺，不能因此反向断言引擎一定不清理。

## R04｜P2：浮渣 OnPicked 返回 true 的作用沿用了过时注释

- 页面位置：[导航 L42](../../../wiki/reference/prefabs/flotsam_puddle.md#L42)、[陷阱 L69](../../../wiki/reference/prefabs/flotsam_puddle.md#L69)。现文认为 `OnPicked` 返回 true 会阻止 inventoryitem 把浮渣本体给玩家。
- 虽然 [OnPicked L200-L201](../../../raw/current/scripts/prefabs/oceanfishableflotsam.lua#L200-L201) 有同样注释，当前装配将它挂为 **pickable.onpickedfn**（[L336-L340](../../../raw/current/scripts/prefabs/oceanfishableflotsam.lua#L336-L340)）。`Pickable:Pick` 调用时丢弃其返回值，再独立检查 `remove_when_picked` 移除宿主（[L551-L579](../../../raw/current/scripts/components/pickable.lua#L551-L579)）。inventoryitem 的返回协议属于 `onpickupfn`（[L457-L459](../../../raw/current/scripts/components/inventoryitem.lua#L457-L459)），不是本页装配的回调。
- 建议：保留“正常采集取得战利品，浮渣宿主被移除”的结果，归因于 `OnPicked` 自己生成战利品（[L187-L200](../../../raw/current/scripts/prefabs/oceanfishableflotsam.lua#L187-L200)）和 pickable 的 `remove_when_picked`；移除返回 true 控制给予本体的说法。陆上宿主另设 `inventoryitem.canbepickedup=false`（[L331-L334](../../../raw/current/scripts/prefabs/oceanfishableflotsam.lua#L331-L334)）。

## R05｜P2：浮渣作祟复用 OnPicked 不等于经过采集清理

- 页面位置：[导航 L42](../../../wiki/reference/prefabs/flotsam_puddle.md#L42) 把正常采集和 hauntable `OnHaunt` 复用写在同一条，后半句统一写成 `remove_when_picked` 移除物品，掩盖了分支差异。
- [OnHaunt L288-L294](../../../raw/current/scripts/prefabs/oceanfishableflotsam.lua#L288-L294) 在概率命中后直接调用 `OnPicked(inst, nil)`，没有调用 `Pickable:Pick`、没有 `inst:Remove`，也没有设置采集状态。`OnPicked` 本身只生成掉落和碎裂，不删除浮渣；`hauntable:DoHaunt` 也不替它执行 pickable 的移除步骤（[L82-L104](../../../raw/current/scripts/components/hauntable.lua#L82-L104)）。
- 建议：单独交代作祟概率分支会生成战利品与 `flotsam_break`，但所查 Lua 路径不消费浮渣宿主；正常采集才经 `Pickable:Pick` 清理。不要把宿主留存与 R03 的引擎父子清理混为一事。未实测重复作祟，不补写实测收益或利用建议。

## R06｜P1：蟹王页再次出现“删 FX 注册不影响任何结算”的危险保证

- 页面位置：[陷阱 L73](../../../wiki/reference/prefabs/crab_king_bubble1.md#L73)：“删除或替换注册不影响任何结算”。纯表现职责的判断正确，但不能推导出删除注册后的调用仍可继续。
- `endgeyser` 先未判空地 `SpawnPrefab(prefab).Transform`，再遍历 oceanfishable 对象击飞（[L1794-L1816](../../../raw/current/scripts/prefabs/crabking.lua#L1794-L1816)）。若生成失败返回 nil，Transform 访问会先中断该回调，后续击飞无法继续。同样，海堆销毁回调先生成水柱，再 `workable:Destroy`（[L1094-L1100](../../../raw/current/scripts/prefabs/crabking.lua#L1094-L1100)）；三叉戟水柱调用在耐久消费前（[L183-L197](../../../raw/current/scripts/prefabs/trident.lua#L183-L197)）。
- 缺注册的生成路径见 [SpawnPrefabFromSim L377-L384](../../../raw/current/scripts/mainfunctions.lua#L377-L384) 与 [SpawnPrefab L429-L436](../../../raw/current/scripts/mainfunctions.lua#L429-L436)。这里指出未判空调用的失败边界，没有运行游戏故障注入。
- 建议：明确“FX 自身不结算玩法；可移除／替换生成调用并保留后续逻辑，或使生成失败可安全跳过；不能只删注册而保留未判空调用”。不要改成“生成调用不可省略”。对这条总括断言列出一至两个关键中断点即可，无需在 FX 页展开所有战斗代码。

## R07｜P2：蟹王喷泉的 oceanfishable 门禁被误写成仅鱼类

- 页面位置：[导航 L46](../../../wiki/reference/prefabs/crab_king_bubble1.md#L46)、[陷阱 L73](../../../wiki/reference/prefabs/crab_king_bubble1.md#L73)，前者称“鱼类（oceanfishable）”，后者称“仅 oceanfishable 实体”却又排除所有物品。快速判断 L22 的“鱼类击飞”也宜同步收准。
- `endgeyser` 实际只判 `v.components.oceanfishable ~= nil`，并未判鱼类标签／鱼类注册（[L1811-L1833](../../../raw/current/scripts/prefabs/crabking.lua#L1811-L1833)）。`MakeProjectile` 允许自定义转换或返回自身（[L52-L53](../../../raw/current/scripts/components/oceanfishable.lua#L52-L53)）。本批相邻源码已经提供非鱼反例：`oceanfishableflotsam_water` 挂 oceanfishable，并装 `OnMakeProjectile`（[L265-L271](../../../raw/current/scripts/prefabs/oceanfishableflotsam.lua#L265-L271)）；其转换增加 complexprojectile 并返回自身（[L140-L157](../../../raw/current/scripts/prefabs/oceanfishableflotsam.lua#L140-L157)）。
- 建议：写“满足搜索条件且挂 oceanfishable 的对象（包括可钓浮渣）”，保留搜索排除标签和两种发射分支的必要边界；撤回按物种概括的绝对排除。此处无需穷举全部可钓对象，也不据组件存在保证未经核验模组对象一定能安全转换。

## 其余逐页核对结果

- **ink_puddle_land**：四条目的 bank／build／anim／GroundOrientation、OnHit 海陆选名、先 splash 再 puddle 两实体、半径 1 上墨／半径 2 combat AOE 与投射物清理均与源码相符。除 R02 未发现 P1／P2。
- **flotsam_puddle**：条目 layer 确为 `LAYER_WORLD_BACKGROUND`，不同于 GroundOrientation；水洼调用延迟 2 帧、缩放 1.15、随机朝向正确；OnLand 以 `IsPassableAtPoint` 分支，海中落点和绞盘直接生成陆上形态的路径都未调用 playlandfx。待修 R02～R05。
- **merm_splash**：三条目字段、月变／月返新体 PushEvent、王 oncreated、召唤守卫第 30 帧生成与回巢节点基本正确。buff 状态定义中的生成语句确实存在，错误是把它与世界广播接成了已核验可达链；不能为了修 R01 删除该状态实现事实。待修 R01／R02。
- **bee_poof_big**：big／small 同资源同 1.4 缩放而附加声音不同；护卫换档门禁、清焦、图书逐只延迟出蜂、蜂窝受击先生成 honey_splash 的逻辑与源码一致。病害三条目为块注释；disease_puff 字面命中仅注册和 grass 依赖，未找到本快照生成路径；病害组件实际使用 diseaseflies。未发现 P1／P2。
- **crab_king_bubble1**：三气泡／水柱／shine／icefx／ring 字段、海堆作为冒泡中心、初排任务和错误自重排、喷泉发生器及清场、Follower 只加在 shine 代理、旧冰冻发生器注册注释均确认。scheduler 对 nil fn 跳过有明确 Lua 依据（[L184-L190](../../../raw/current/scripts/scheduler.lua#L184-L190)），没有另报“必崩”；引擎视觉跟随仍按页中缺口处理。待修 R06／R07。

## 检索与机械验收

| 本次实际执行 | 结果与边界 |
|---|---|
| 5 页＋fx-table-index／fx-table-world-index，`validate_page --check-indexes` | 7／7 passed，navigation passed，issue 0；不证明正文事实正确 |
| 5 页，`validate_source_links --strict` | 165 引用（164 行引用＋1 文件引用）、138 独立目标、32 源码文件；error 0、warning 0、skipped_unsupported 0 |
| 5 页，`validate_source_hashes check --fresh-hours 0` | 40 组已声明来源 unchanged；changed／sources_changed／invalid／unrecorded 0；未 record，R02 漏项不在该通过结论内 |
| sources 与正文实际使用对照 | 三页漏 5 个页面—文件依赖，详见 R02 |
| 注册名覆盖／归属 | 本批 20 名在自身 aliases 与正文均存在；Prefab 目录精确 aliases 反查无跨页主要归属冲突；五页叶级表格登记及现有返回入口可达 |
| 官方名称 | 简中 .po 定向查 20 个注册名均无 NAMES 键；另核对 BEEQUEEN／BEEGUARD／BOOK_BEES，译文与蜂群页相符；未将效果描述冒充官方名 |
| 生成点定位 | 全 scripts 根及子目录 `*.lua`：本批名称／族名前缀字面检索 100 行，相关字符串拼接候选 11 行，均完整捕获并检查，未用 head 截断；区分注册、依赖、资源字段、实际 Spawn 与注释停用路径 |
| 当前 Markdown 章节读取 | 五次标题原文 read 均退出 0：墨水洼案例、浮渣案例、鱼人导航、蜂群陷阱、蟹王导航；没有用 search 缓存声称检索通过 |

候选检索不构成对任意动态表达式的完备证明；负向结论限定所查快照／范围。未运行 DST，未重跑未改动 Lua 示例的语法解析；未验证动画二进制资源、时长、晚加入与引擎级联。

## 交付与后续

本轮仅新增本报告、追加 `wiki/log.md`；不修知识页、不刷新来源基线、不改 accepted／PLAN／索引，不提交、不推送。48 个受保护输入复核未变，报告 60 个本地链接路径／物理行区间有效，diff --check 通过。结束时另见他方新增未跟踪 `wiki/reference/prefabs/pickaxe.md`，未改动、未纳入本轮验收；开始时已有 BUILD 差异保留。

累计已独立检视 **32／158 页，余 126 页**（“已检视”包含待修页，不等于全部验收通过）。本轮先修 R01～R07 后复验；下一个未检轮次为原生成批 6：`sinkhole_spawn_fx`／`glass_fx`／`quagmire_portal_player_fx`／`lavaarena_portal_player_fx`／`firesplash_fx`，5 页／27 注册。本轮不自动进入下一批。

## 2026-11-06 修后独立复验

结论：**原 R01～R07 的整改点已覆盖，未发现剩余 P1；新增来源漏项合并为 R08（P2×1）。本批补齐来源和对应基线后可再收口。** 当前不改 accepted 状态；外部在建页的导航问题单列，不计入本批 P1／P2。

复验仍针对原批 5 的五页／20 注册。BUILD 仍为 26335394，K HEAD 仍为 `97afb4e9e6ca05390558769926f1f9c8598d0f70`。检查了四篇改页与索引的修订全文／差异，未改的蜂群页和所用源码与初检指纹一致；保留并行生成工具页、台账、来源基线、BUILD 等他方改动。

### 原问题处置

| 编号 | 复验结果 |
|---|---|
| R01 | 已覆盖：鱼人摘要、快速判断、导航、陷阱、验证记录与索引 L31 均区分 buff 态生成语句与缺失的世界事件转发链，撤回全体自动播放水花的断言；属性升级事实保留 |
| R02 | 原漏项已补：三页均纳入 `prefabs/fx.lua`，浮渣／鱼人另纳入 `.po`，原 5 项页面—文件依赖均进入声明与基线；本次修订另引入的来源遗漏见 R08 |
| R03 | 已覆盖：浮渣案例和宿主关系不再保证引擎随父删除，改为已核验的代理 1 秒任务与本地 animover，并保留引擎级联缺口 |
| R04 | 已覆盖：正常采集由 `OnPicked` 生成战利品，返回值被 pickable 丢弃，`remove_when_picked` 清理宿主；不再把不给本体归因于返回 true |
| R05 | 已覆盖：作祟另列生成者与陷阱，明确直接调用 `OnPicked`、不经过采集清理且不移除宿主；未伪造重复作祟实测 |
| R06 | 已覆盖：蟹王陷阱 L73 区分 FX 无玩法职责与未判空生成调用的失败边界，说明应删改或判空调用，没有反向禁止省略表现调用 |
| R07 | 已覆盖：蟹王快速判断、喷泉导航、陷阱与验证记录均改为 oceanfishable 组件门禁，明确包含可钓浮渣、不限鱼类 |

### R08｜P2：新增引用未进入 sources，另有同页实现依赖漏项

修订新增的技术结论与证据本身成立，但 sources 没有跟进。**直接链接机械对照查到 3 个漏项，另人工核对出 1 个实现依赖漏项，合计 4 个页面—文件依赖：**

| 页面／位置 | 应补来源 | 核验依据 |
|---|---|---|
| [flotsam_puddle L42、L70](../../../wiki/reference/prefabs/flotsam_puddle.md#L42) | `components/pickable.lua` | 正文已直接链接 [Pickable:Pick L551-L579](../../../raw/current/scripts/components/pickable.lua#L551-L579)，证明返回值未消费与正常采集删除宿主；frontmatter L10 未声明 |
| [flotsam_puddle L43、L71](../../../wiki/reference/prefabs/flotsam_puddle.md#L43) | `components/hauntable.lua` | 正文已直接链接 [DoHaunt L82-L104](../../../raw/current/scripts/components/hauntable.lua#L82-L104)，用于区分作祟与采集清理；frontmatter L10 未声明 |
| [merm_splash L74](../../../wiki/reference/prefabs/merm_splash.md#L74) | `entityscript.lua` | 正文新增 [PushEvent_Internal L1312-L1335](../../../raw/current/scripts/entityscript.lua#L1312-L1335) 直接链接，支撑 SG 分派的关键修正；frontmatter L10 未声明 |
| [flotsam_puddle L54、L76](../../../wiki/reference/prefabs/flotsam_puddle.md#L54) | `entityscript.lua` | 虽未新增直接链接，但正文已采用 `AddChild` 登记／`Remove` 遍历 children 的实现结论，应纳入实际支撑来源；证据为 [AddChild L921-L935](../../../raw/current/scripts/entityscript.lua#L921-L935)、[Remove L1779-L1783](../../../raw/current/scripts/entityscript.lua#L1779-L1783) |

建议：浮渣页补上述 3 个 sources，鱼人页补 `entityscript.lua`；浮渣 children 结论补短锚点，或链接能直接支撑该结论的已核验主要页。由修复方按现有规则仅刷新这两页的来源基线，再跑 sources 对照／hash check。本次 45 组 unchanged 不能覆盖这 4 项未声明依赖，不要只 record 现有 sources 后称来源完整。

### 本次实际复验与边界

- `validate_page --check-indexes`：所选五页＋两索引 **7／7 页面通过、页面 issue 0**；全局 navigation=false，整体退出 1，原因仅为他方在建 `axe`／`glasscutter`／`hammer`／`pickaxe`／`shovel` 五页各缺叶级表格登记。全库导航不宣称通过；本批五页均恰有一次登记（墨水／浮渣在 fx-table-world-index，其余在 fx-table-index），未发现本批导航缺陷。未修改外部页或替其登记。
- `validate_source_links --strict`：**180 引用（179 行引用＋1 文件引用）、145 独立目标、35 源码文件，error 0、warning 0、skipped_unsupported 0**，退出 0。
- `validate_source_hashes check --fresh-hours 0`：**45 组已声明来源全部 unchanged**，changed／sources_changed／invalid／unrecorded 0，退出 0；没有 record。
- 正文直接来源对照：墨水、蜂群、蟹王无直接链接漏项；浮渣两项、鱼人一项，另补入人工核验的浮渣 entityscript 依赖，详见 R08。
- 四次标题原文章节 read 均退出 0：浮渣导航、浮渣案例、鱼人陷阱、蟹王陷阱；未用查询缓存作 search 验收。
- 初检使用的 raw 指纹均未变；本次固定 48 个输入，报告／日志写入前后另核对。机械检查、源码阅读和实际游戏运行仍分别记账；未运行游戏、未重跑未改动 Lua 示例，不扩展动画资源、引擎级联、跨分片运行表现的结论。

本次仅更新本检视报告并追加 `wiki/log.md`，未修知识页、索引、台账、PLAN、来源基线或 raw；未提交、未推送。累计已独立检视仍为 **32／158 页，余 126 页**，不因复验重复计数；下一未检轮次仍为原批 6。

## 2026-11-06 R08 修后独立收口

- R08 的四项页面—文件依赖已实际补入：浮渣页 [sources L10](../../../wiki/reference/prefabs/flotsam_puddle.md#L10) 包含 `components/pickable.lua`、`components/hauntable.lua`、`entityscript.lua`；鱼人页 [sources L10](../../../wiki/reference/prefabs/merm_splash.md#L10) 包含 `entityscript.lua`。直接引用与声明对照无漏项，浮渣 children 的实现依赖也已纳入。
- 两页来源共 20 组均在本次 8 页／92 组只读 hash check 中通过，无 changed／sources_changed／invalid／unrecorded；未执行 record。新增依赖确实进入基线，未只增加 frontmatter 后沿用旧基线。
- 与上次独立复验保存的 48 个输入比对，只有这两页、来源基线和角色战斗索引变化；其余三个本批知识页及原核验源码均未变。原 R01～R07 的语义结论继续有效，索引对鱼人 buff 事件缺口的说明仍保留。
- 本次所选 8 页＋两索引格式与导航 10／10 通过，全局 1119 知识页／34 索引、导航问题 0；其中包含本轮两页。严格链接总计 301 引用零错警。浮渣案例、鱼人陷阱两次标题原文 read 退出 0。该组合检查同时覆盖第 5 轮修订，不冒充重新逐条复审其他三页。
- 第 4 轮当前待修 P1／P2 为 0，可以收口。未运行 DST、未扩大引擎级联清理结论；只改报告与日志，不改知识页／索引／台账 accepted／PLAN／基线，不提交推送。累计检视进度以第 5 轮报告为准，复验不重复计数。
