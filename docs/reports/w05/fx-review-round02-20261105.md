# FX 新增页独立检视：第 2 轮

日期：2026-11-05。源码：`raw/current/scripts`，实读 `steam-buildid:26335394`；知识仓 HEAD：`e271fae`。

本轮对应原生成批 3（F07～F10），共 **6 页、25 个通用表注册**，不重复上一轮 8 页及 U15～U17 九页。原批次自报复审仅作线索，本报告为再次独立对照当前源码的结果。

结论：**需局部修复；P1×0、P2×10、P3×3**。没有发现需要重写通用工厂协议或重新组织这批页面的问题。下述时序与门禁结论均为源码推导，未运行游戏。本轮只新增报告、追加日志；不修知识页、不 record、不改台账 accepted 状态、不提交或推送。

上述结论及下方 R01～R13 保留首次检视记录；修订后的最新状态见文末“修后复验”。

## 本轮范围与逐页结论

路径相对 `wiki/reference/prefabs/`；行数按物理行统计，大小为本轮输入字节数。

| 页面 | 注册数 | 行／字节 | 核对范围与结论 |
|---|---:|---:|---|
| `attune_out_fx.md` | 3 | 83／9,999 | 调谐注册／注销事件、等价类型换绑、玩家监听窗口、远程复活标志、墓碑四处落点复用；R01（P2） |
| `beefalo_transform_fx.md` | 1 | 78／7,008 | 驯化与野性化置位、状态入口、8 帧生成与 onexit 补枚、骑乘挂接；R02（P2）、R11（P3） |
| `ocean_splash_med1.md` | 8 | 88／12,927 | 逐变体 fn、动态前缀、海鱼／浮标／游泳／影仆等生成点、门禁与延迟；R03、R04、R10（P2），R12、R13（P3） |
| `splash_green.md` | 4 | 81／10,675 | 体型映射、冲滩、生物与玩家差异、沉水组件、瓶子投掷、海树及代表生成点；R05（P2） |
| `splash_sink.md` | 3 | 94／10,533 | SinkEntity 三档、离场／重定位、两条浮具分支、传送门避障、猪神殿三触发；R06、R07（P2） |
| `round_puff_fx_sm.md` | 6 | 81／10,528 | 标签选名、伍迪两个回调、蟹王代理符号绑定、四状态图滑行烟、钻船任务与资源复用；R08、R09（P2） |

25 个注册名均在当前 `fx.lua` 注册表中存在，均已进入本页 aliases 与正文；对非归档 Prefab 页反查未发现这 25 名跨页重复归属。当前简中 `.po` 无这 25 个 `STRINGS.NAMES.<注册名大写>` 键；`BEEFALO` 的“皮弗娄牛”名称与页面锚点一致。未把效果描述当官方译名。

## 待修问题

### R01 · P2：等价调谐类型之间换绑不能概括为静默

- 页面：[attune_out_fx.md:73](../../../wiki/reference/prefabs/attune_out_fx.md#L73)；第 40、46 行与 [索引第 35 行](../../../wiki/reference/prefabs/fx-table-index.md#L35) 也涉及这项概括。
- 当前说法：“同类换绑中间态不出特效”“旧源解除时 HasAttunement 仍查到新源”。第 46 行同时说明肉像与温蒂复活墓互为等价类型，容易将静默结论用于这两类互换。
- 源码：[attunable.lua:20–30](../../../raw/current/scripts/components/attunable.lua#L20-L30) 会按 `EQUIVALENT_ATTUNABLE_TAGS` 解除另一类型旧源；[LinkToPlayer:105–113](../../../raw/current/scripts/components/attunable.lua#L105-L113) 先注册新源，再推 `attuned` 解除旧源。但 [HasAttunement:26–32](../../../raw/current/scripts/components/attuner.lua#L26-L32) 只检查传入的单一 tag，不查等价表。[OnAttunementLost:454–469](../../../raw/current/scripts/prefabs/player_common.lua#L454-L469) 对旧 tag 检查，故肉像→复活墓或反向换绑时，旧 tag 已无来源，仍可进入 in／ghost_in 分支，并把 `_isrezattuned` 置 false（另受 `remoterezsource` 抑制）。
- 建议：静默限定为“同一实际 attunable tag 内换绑”；补一句等价但不同 tag 互换的现有分支行为。不要把 `HasAttunement` 写成查到了等价新源，同步导航与索引。这是代码分支推导，不声称游戏复现。

### R02 · P2：快速判断把变身表现限定为驯化完成，漏掉失去驯化

- 页面：[beefalo_transform_fx.md:21](../../../wiki/reference/prefabs/beefalo_transform_fx.md#L21)，摘要第 6 行与 [索引第 36 行](../../../wiki/reference/prefabs/fx-table-index.md#L36) 同样偏向只写驯化完成；正文第 40 行却已写出 `goneferal`。
- 源码：[domesticatable.lua:79–84](../../../raw/current/scripts/components/domesticatable.lua#L79-L84) 在清除 `domesticated` 前先推 `goneferal`；[OnFeral:542–547](../../../raw/current/scripts/prefabs/beefalo.lua#L542-L547) 此时仍能判已驯化并置 `domesticationPending`。随后仍进入 [become_domesticated:722–761](../../../raw/current/scripts/stategraphs/SGBeefalo.lua#L722-L761) 播表现，11 帧的 [UpdateDomestication](../../../raw/current/scripts/prefabs/beefalo.lua#L557-L563) 可以执行 `DoFeral`。
- 建议：职责概括为“驯化外观转换，含完成驯化与已驯化后野性化”，保留两处生成及骑乘补枚边界，清除“只服务驯化完成”的限定。

### R03 · P2：ripple 与 swim 还存在海洋混合参数差异

- 页面：[ocean_splash_med1.md:76](../../../wiki/reference/prefabs/ocean_splash_med1.md#L76) 声称差异“只在 swim 带压低音量的水声与 0.7 倍速”；第 32 行表格也未列此区别。
- 源码：[ripple1／2:1920–1938](../../../raw/current/scripts/fx.lua#L1920-L1938) 都调用 `SetOceanBlendParams(TUNING.OCEAN_SHADER.EFFECT_TINT_AMOUNT)`；[swim1／2:1940–1962](../../../raw/current/scripts/fx.lua#L1940-L1962) 没有该调用，只设置背景层、动画倍率与音量。
- 建议：补明 swim 条目未设置这项海洋混合参数，删除“差异只在……”的封闭列举；不要据此进一步断言未实测的视觉颜色。

### R04 · P2：海面水花页两份正文证据未进入 sources

- 页面：[ocean_splash_med1.md:10](../../../wiki/reference/prefabs/ocean_splash_med1.md#L10)，正文第 52、79 行。
- 已引用并据此下结论的 [prefabs/player_common.lua:2226–2227](../../../raw/current/scripts/prefabs/player_common.lua#L2226-L2227)、[prefabs/wobster.lua:34–43](../../../raw/current/scripts/prefabs/wobster.lua#L34-L43) 与 [90–106](../../../raw/current/scripts/prefabs/wobster.lua#L90-L106) 不在当前 sources；链接检查通过并不补上哈希依赖。
- 建议：将 `prefabs/player_common.lua`、`prefabs/wobster.lua` 补入 sources，核验相关结论后按现有流程刷新该页基线。当前 85 组 unchanged 仅覆盖已声明的来源。

### R05 · P2：绿水花页错误排除了物品落水调用

- 页面：[splash_green.md:70](../../../wiki/reference/prefabs/splash_green.md#L70)：“误区：splash_green 用于物品落水”“本组未见物品落水路径调用”。与本页第 44、45 行矛盾。
- 源码：[submersible.lua:73–78](../../../raw/current/scripts/components/submersible.lua#L73-L78) 接物品 `on_landed`，位于海面时进入 `Submerge`，并可在 [MakeSunken:181–193](../../../raw/current/scripts/components/submersible.lua#L181-L193) 生成 `splash_green`。[投掷瓶 OnHit:285–292](../../../raw/current/scripts/prefabs/messagebottle.lua#L285-L292) 与 [胶团瓶 OnHit:385–392](../../../raw/current/scripts/prefabs/messagebottle.lua#L385-L392) 在无平台的非地面位置生成 `splash_green_small`。
- 建议：限定为“floater 的普通漂浮冷热分支使用 splash／hot_splash；沉水物品和投掷瓶等另有 green 族调用”。不能从 floater 单一路径推出整族不用于物品落水。

### R06 · P2：float_let_go 两分支不是无条件各出一枚

- 页面：[splash_sink.md:44](../../../wiki/reference/prefabs/splash_sink.md#L44)：“普通分支与 wx78 分支各落一枚”。
- 源码：[普通分支:26617–26636](../../../raw/current/scripts/stategraphs/SGwilson.lua#L26617-L26636) 在 `FindPlayerFloater` 找到替代浮具时先转 `float_pre` 并 return，不出这枚水花；无替代浮具还需原 `floater` 存在且仍有效。[WX 分支:26662–26677](../../../raw/current/scripts/stategraphs/SGwilson.lua#L26662-L26677) 需手持物带 `playerfloater`、`LetGo` 后仍有效才生成，随后才寻找替代浮具。
- 建议：改成“两分支各有一个条件生成点”，分别保留上述门禁，避免把状态到达当成必出花。

### R07 · P2：传送门出口初始被挡并不一定放弃传送

- 页面：[splash_sink.md:52](../../../wiki/reference/prefabs/splash_sink.md#L52)：“出口被挡则放弃传送不出花”。
- 源码：[oceanwhirlportal.lua:104–133](../../../raw/current/scripts/prefabs/oceanwhirlportal.lua#L104-L133) 在出口初始点被陆地／实体挡住时，搜索附近可用 offset；找到后改写 `ex, ez` 并继续，在 [138–142](../../../raw/current/scripts/prefabs/oceanwhirlportal.lua#L138-L142) 出双花及传送。只有 offset 搜索失败才 return；另有 [78–79 行](../../../raw/current/scripts/prefabs/oceanwhirlportal.lua#L78-L79) 的出口 tracker blocker 门禁，会直接 return。
- 建议：区分“出口有 tracker blocker 直接跳过”和“出口位置受阻先找替代点，找不到才放弃”。双花的出口坐标应写实际选定位置。该行“周期”链接还应从只含距离常量的 L12 改为 L10 与任务注册 L255。

### R08 · P2：slide_puff 的宿主是鱼人，非鱼人王

- 页面：[round_puff_fx_sm.md:46](../../../wiki/reference/prefabs/round_puff_fx_sm.md#L46)：“鱼人王 dodge_attack”。
- 源码：[SGmerm.lua:118–121](../../../raw/current/scripts/stategraphs/SGmerm.lua#L118-L121) 接 `attackdodged` 进该状态，[第 798–804 行](../../../raw/current/scripts/stategraphs/SGmerm.lua#L798-L804) 生成滑行烟并读 `MERMKING_CROWNBUFF_DODGE_SPEED`。常量名不表示宿主是鱼人王：[merm.lua:1303](../../../raw/current/scripts/prefabs/merm.lua#L1303)、[1467](../../../raw/current/scripts/prefabs/merm.lua#L1467) 安装 SGmerm，鱼人王则安装 [SGmermking](../../../raw/current/scripts/prefabs/mermking.lua#L624-L626)。
- 建议：改为“鱼人／鱼人守卫的 SGmerm dodge_attack”；若不展开上游赐予闪避条件，就保留该范围，不把常量前缀当实体归属。

### R09 · P2：圆烟位置建议把两个伍迪回调的依据混在一起

- 页面：[round_puff_fx_sm.md:66](../../../wiki/reference/prefabs/round_puff_fx_sm.md#L66) 建议 `r / (r + 1)` 插值，却链接到仅有目标中心落点的 `OnTackleTrample:1568–1570`。
- 源码：实际插值位于 [OnTackleCollide:1552–1557](../../../raw/current/scripts/prefabs/woodie.lua#L1552-L1557)，生成的是 hi 档；sm／lg 的 [OnTackleTrample:1568–1570](../../../raw/current/scripts/prefabs/woodie.lua#L1568-L1570) 直接取目标世界坐标，与本页示例一致。
- 建议：将插值参照链接改到 Collide，并说明是借用 hi 档调用方的位置算法；保留“sm／lg 原版 Trample 与骑枪使用目标中心”。不要让“原版依据”把两种落点写成同一回调。

### R10 · P2：影仆 spawn 水花任务时间遗漏随机倍率

- 页面：[ocean_splash_med1.md:47](../../../wiki/reference/prefabs/ocean_splash_med1.md#L47) 写“spawn 任务 0／20 帧 med、44 帧 small”。
- 源码：[SGshadowwaxwell.lua:272–289](../../../raw/current/scripts/stategraphs/SGshadowwaxwell.lua#L272-L289) 先取 `mult = mult or (0.8 + math.random() * 0.2)` 作为动画倍率，再取倒数调度任务。20、44 都要乘倒数，不是固定墙钟帧数。没有传入 mult 时，后两次名义等待约为 20～25 帧、44～55 帧；实际调度仍受引擎 tick 影响。
- 建议：写成“按动画倍率缩放的 0／20／44 基准帧任务”，或保留源码公式；不必引入新的机制页或展开整张状态图。

### R11 · P3：牛页把 actual_alert 写成 alert 的 ontimeout

- 页面：[beefalo_transform_fx.md:40](../../../wiki/reference/prefabs/beefalo_transform_fx.md#L40)。
- 源码：[SGBeefalo.lua:573–601](../../../raw/current/scripts/stategraphs/SGBeefalo.lua#L573-L601) 对应状态名是 `actual_alert`；`alert` 自身是另一个转发状态。
- 建议：把状态名改为 `actual_alert`，让按标识符查找能直达实际入口。

### R12 · P3：龙虾 splash 的出入水方向写反

- 页面：[ocean_splash_med1.md:79](../../../wiki/reference/prefabs/ocean_splash_med1.md#L79)：“wobster 出水瞬间的 splash”。
- 源码：[wobster.lua:90–106](../../../raw/current/scripts/prefabs/wobster.lua#L90-L106) 是 `on_projectile_landed` 的落回水中分支：移除 complexprojectile、设水下层、转 idle／jump_pst，再生成 `splash`。真正 `launched_out_of_water` 由 [SGwobster.lua:148–157](../../../raw/current/scripts/stategraphs/SGwobster.lua#L148-L157) 生成 `ocean_splash_small1`。
- 建议：改为“投射后落回水中的 splash”，保留两个注册分工的原意。

### R13 · P3：海面水花页仍残留待登记旧注

- 页面：[ocean_splash_med1.md:86](../../../wiki/reference/prefabs/ocean_splash_med1.md#L86)：“叶级索引登记待主代理处理”。
- 当前实际已有 [fx-table-index.md:37](../../../wiki/reference/prefabs/fx-table-index.md#L37) 唯一登记，本轮导航检查也通过。
- 建议：删去该待办或注明是当时写作者自检状态，避免被当成当前缺口。不要改写其历史工具执行结果。

## 已核对但未列为缺陷的边界

- `SpawnChild` 经 `EntityScript:SpawnChild` 调 `AddChild`；蟹王的 Follower 则加在代理上，不能据此断言本地动画持续跟随符号。本批页面对后者的保留说明是合适的。
- `wood_splinter_jump`：本轮全 scripts 根级及子目录检索 `wood_splinter`，完整读取 12 行候选；注册与依赖之外仅 drill 实际生成，SGwilson 的拼接是 OverrideSymbol。另定位了四类生成调用同一行含 `..` 的 108 行候选，无该前缀动态生成命中。保留“当前检索范围内未找到”的缺口，不将其升级成任意运行时绝无生成。
- `beefalo_transform` 前缀完整候选 3 行，对应注册及 SGBeefalo 两处生成；`slide_puff` 完整候选 20 行，四个状态图确有生成，其余需区分资源／符号覆盖。`attune_` 完整候选 39 行，墓碑落点复用与玩家两处生成均存在。候选没有用 head 截断。
- 海面水花 `ocean_splash` 字面检索完整 62 行，另追浮标配置前缀、鱼类 breach_fx 数组、影仆 size 拼接与 sharkboimanager 动态名；不会把依赖表当生成点。绿水花页明确未展开的其余生物 SG 调用，仍只按它声明的检索级范围记录，不冒充完整触发链复审。
- 通用工厂的一帧延迟、代理 1 秒回收、本地 animover 自删、父挂与落点分工已复核；未假设引擎 SetParent 自动完成 Lua children 登记，也未核验资源动画实际时长与视觉。

## 本轮实际校验

以下均为本轮对 6 页实际运行，不引用旧批自报结果代替执行。

| 检查 | 结果与限制 |
|---|---|
| `validate_page.py --check-indexes` | 6／6 passed；navigation passed，issues=0；尺寸均在对应档位内 |
| `validate_source_links.py --strict` | 199 处行引用，68 个目标文件，errors=0、warnings=0、skipped_unsupported=0，passed=true |
| `validate_source_hashes.py check --fresh-hours 0` | 6 页／85 组比较全部 unchanged，changed／sources_changed／invalid／unrecorded 均 0；不覆盖 R04 漏声明的两个文件 |
| 注册名／名称 | 25 名均存在且有 aliases／正文；非归档 Prefab 页无这 25 名重复主要归属；25 个简中 NAMES 键均无命中，BEEFALO 键命中 |
| 检索方式 | 当前 Markdown 与源码定向检索；没有重建查询缓存，未把缓存 search 当验收 |
| Lua／游戏 | 本轮未修改示例，未重跑 Lua parser；未运行 DST，不核准视觉、声音或网络实机效果 |

写入报告前后还核对本轮 6 页、基线、BUILD 及其已声明源码共 76 个输入的哈希，确认本轮未改变输入；报告链接和 Git diff 空白检查见交付时复核。机械通过不能否定上述门禁、消费者身份、来源完整性与证据语义问题。

## 后续轮次

本轮 6 页已完成独立检视，尚需按 R01～R13 修复后复验，未由本报告改变 accepted 状态。158 篇新增页中，已独立检视的范围累计为 U15～U17 九页＋第 1 轮八页＋本轮六页，共 23 页，**仍有 135 页待独立检视**，不等于 135 页待建。

下一轮按原生成批 4：`boat_grass_erode`、`fx_boat_crackle`、`boat_mast_sink_fx`、`cannonball_used`，共 4 页、24 注册；继续逐轮进行，不自动扩到全库。

## 修后复验（2026-11-05）

对修改后的 6 页与 `fx-table-index` 重新核对。BUILD 仍为 26335394，HEAD 仍为 e271fae；原已登记源码与首次检视时一致。知识页、索引及基线为用户一方的修订，本次仅追加复验记录，不修知识页、不 record。

结论：**12 项整改闭环，R01 部分完成，仍有 P2×1 的状态前提待补（R14）**。等价不同 tag 换绑的行为已经写对，但“同一 tag 换绑静默”和“out 仅从无到有”仍不能脱离 `_isrezattuned` 的实际值概括。其余本次整改未发现新的 P1/P2；不扩展到下一轮页面。

| 原问题 | 改后位置与复验结果 |
|---|---|
| R01 | `attune_out_fx:6/40/46/73/80`、`fx-table-index:35` 已区分单 tag 检查与等价解除，补出跨类型换绑播 in 并清标志；仍需 R14 的后续状态前提 |
| R02／R11 | `beefalo_transform_fx:6/21/40/68/75` 与索引第 36 行已覆盖野性化，状态名改为 `actual_alert`；符合事件在清除驯化前推送的顺序 |
| R03 | `ocean_splash_med1:32/76` 补出 ripple 设置 OceanBlend、swim 不设置的差异，未把它直接推成实测视觉结论 |
| R04 | `ocean_splash_med1:10` 已加入 player_common／wobster，正文证据保留；基线检查覆盖新增依赖 |
| R05 | `splash_green:21/70/78` 已限定 floater 普通冷热路径，并补沉水物品与投掷瓶反例 |
| R06 | `splash_sink:44` 已区分普通分支的替代浮具提前返回、原浮具有效性，以及 WX 分支的 playerfloater／LetGo 后有效性 |
| R07 | `splash_sink:52` 已区分 tracker blocker 与位置避障搜索，出口采用实际选定点；周期、任务注册锚点改为 L10／L255 |
| R08 | `round_puff_fx_sm:46/78` 已改鱼人／鱼人守卫，sources 补 merm／mermking，安装状态图的证据对应实际宿主 |
| R09 | `round_puff_fx_sm:66/78` 已区分 Collide 的 hi 插值与 Trample／骑枪 sm、lg 目标中心，链接对应正确函数 |
| R10 | `ocean_splash_med1:47/84` 已说明基准帧乘动画倍率倒数，默认约 20～25／44～55 帧的估算与源码一致 |
| R12 | `ocean_splash_med1:79/84` 已改为投射后落回水中，并同时链接落水与出水分支 |
| R13 | `ocean_splash_med1:86` 已清除待登记旧注，实际索引登记仍存在 |

### R14 · P2：跨类型换绑后再同 tag 换绑仍可能播放 out

- 改后页面：[attune_out_fx.md:73](../../../wiki/reference/prefabs/attune_out_fx.md#L73) 仍说持有同一 tag 再换绑时“gotnewattunement 被 `_isrezattuned` 拦下”；第 6、40 行与 [索引第 35 行](../../../wiki/reference/prefabs/fx-table-index.md#L35) 还把 out 概括为“从无到有”。这依赖标志与实际调谐保持一致，而本次补出的跨类型路径正会破坏这个前提。
- 源码：[OnGotNewAttunement:443–451](../../../raw/current/scripts/prefabs/player_common.lua#L443-L451) 只看 `not inst._isrezattuned` 和新源类型，并不查询玩家此前是否实际持有调谐。[OnAttunementLost:454–469](../../../raw/current/scripts/prefabs/player_common.lua#L454-L469) 在跨 tag 换绑时可将标志清 false，尽管新源仍在。全 scripts 检索 `_isrezattuned` 只有该文件中的 5 行命中：两个判断、两处赋值及 [加入时初值](../../../raw/current/scripts/prefabs/player_common.lua#L865-L866)，未发现换绑后另一路即时校正。
- 具体分支反例（源码推导，非游戏复现）：已绑定肉像且标志 true → 换绑复活墓 A，旧肉像解除后播 in、标志 false → 再换绑复活墓 B，此时同为 `gravestoneresurrector`，但 got 回调会播 out 并置 true；解除 A 时因 B 仍在，不播 in。因此“同 tag 一定静默”和“out 一定代表此前没有实际调谐”都不成立。
- 建议：生成者行直接写实际门禁：“新源为复活调谐类型且 `_isrezattuned` 为 false 时播 out 并置 true”。同 tag 静默限定为进入换绑前标志为 true；用一句话保留跨类型换绑清标志后，下一次同 tag 换绑也可能播 out 的边界。同步摘要、索引与验证记录即可，不必复制完整调谐状态机。

### 本次实际复验

- `validate_page.py --check-indexes`：6 篇知识页＋1 篇受影响索引全部 passed，navigation passed，issues=0。
- `validate_source_links.py --strict`：6 页、213 处行引用、70 个目标文件，errors=0、warnings=0、skipped_unsupported=0。
- `validate_source_hashes.py check --fresh-hours 0`：6 页、89 组比较全部 unchanged，changed／sources_changed／invalid／unrecorded 均 0。四份新增来源（海面水花两份、圆烟两份）已进入依赖检查；另核对六页正文 Lua／PO 引用，未发现漏列 sources 的目标。
- 本次输入保护共 80 个文件（6 页、受影响索引、基线、BUILD 与已声明源码）；复验期间哈希未变。自首次检视以来，原 76 个输入中仅六篇页面与基线由用户一方改动，原已登记源码未变。
- 未重跑 Lua parser、未运行 DST；本次未修改代码示例，不把工具通过等同游戏验证。待独立检视页数仍为 135，本次复验不重复计数。
