# FX 第 17 轮独立内容检视：固定队列最后 8 页

核验日期：2026-11-07；源码快照：steam-buildid:26335394；知识仓 HEAD：97afb4e9e6ca05390558769926f1f9c8598d0f70。

**当前结论：固定队列 158 页、17 个编号轮次的 FX 文档检视已收口。最终格式、导航、队列内来源链接及哈希检查通过。全库 strict 来源链接检查仍有 27 条已知组件页警告，详见文末。** 初检 P1×0、P2×7 及各次复验记录保留为历史。此前修后复验只看文件内容，统一机械校验结果见最后一节。

## 范围与逐页结论

| 页面（wiki/reference/prefabs） | 主要注册数 | 内容结论 |
|---|---:|---|
| [abigailsummonfx](../../../wiki/reference/prefabs/abigailsummonfx.md) | 5 | 本轮未发现 P1/P2 |
| [abysspillar_fx](../../../wiki/reference/prefabs/abysspillar_fx.md) | 1 | R01：名称来源声明；其余本轮未发现 P1/P2 |
| [acidraindrop](../../../wiki/reference/prefabs/acidraindrop.md) | 2 | R01：名称来源声明；其余本轮未发现 P1/P2 |
| [diseaseflies](../../../wiki/reference/prefabs/diseaseflies.md) | 1 | R01：名称来源声明；其余本轮未发现 P1/P2 |
| [explode_small](../../../wiki/reference/prefabs/explode_small.md) | 10 | R01：名称来源声明；其余本轮未发现 P1/P2 |
| [grottopool_sfx](../../../wiki/reference/prefabs/grottopool_sfx.md) | 1 | R01：名称来源声明；其余本轮未发现 P1/P2 |
| [impact](../../../wiki/reference/prefabs/impact.md) | 7 | R01～R04 |
| [ruins_cavein_obstacle_rubble_fx](../../../wiki/reference/prefabs/ruins_cavein_obstacle_rubble_fx.md) | 2 | R05～R07 |

29 名包括 projectile、伤害／落点混合载体和重建石柱的 FX，不等同 29 个纯装饰实体。已读每页全文并追查与本页结论有关的工厂、必要调用、状态、门禁、清理、数值与名称；没有把全部关联宿主的其他玩法全文重审。

## P2：内容与证据问题

### R01｜五页自报核验 .po 却未声明，impact 另有两份资源依据未列来源

下列页面正文自报按简中 .po 检索名称键，但 sources 未包含 languages/chinese_s.po：

| 页面 | 声明位置／名称结论位置 |
|---|---|
| [abysspillar_fx](../../../wiki/reference/prefabs/abysspillar_fx.md#L10) | sources 第 10 行；正文第 22 行 |
| [acidraindrop](../../../wiki/reference/prefabs/acidraindrop.md#L10) | sources 第 10 行；正文第 21 行 |
| [diseaseflies](../../../wiki/reference/prefabs/diseaseflies.md#L10) | sources 第 10 行；正文第 22 行 |
| [explode_small](../../../wiki/reference/prefabs/explode_small.md#L10) | sources 第 10 行；正文第 21 行 |
| [grottopool_sfx](../../../wiki/reference/prefabs/grottopool_sfx.md#L11) | sources 第 11 行；正文第 22 行 |

本次定向检索对应注册名，确未发现这些 FX 的 NAMES 键。按现行名称来源条款，实际采用的 .po 应进入 sources；缺键结论保留检索键与范围，**不要为不存在的名称键伪造行锚点**。修页时补真实采用的来源，最终统一处理相应基线，不把 .po 变动解释成 Lua 机制变动。

另见 [impact:129](../../../wiki/reference/prefabs/impact.md#L129)：文中说同一尾迹资源由 blowdart_lava／blowdart_lava2 声明，但 sources 未列这两份文件，也无相应行锚点。本次核对 [blowdart_lava.lua 第 7～10 行](../../../raw/current/scripts/prefabs/blowdart_lava.lua#L7-L10)和 [blowdart_lava2.lua 第 7～10 行](../../../raw/current/scripts/prefabs/blowdart_lava2.lua#L7-L10)确有该动画包声明。可补采用来源与证据；若该旁支对页面无必要，也可删除这句旁支断言，保留“本弹体未自行声明、外部预载是否保证未核验”。不需要为此展开 Forge 弹体主题。

### R02｜两个投影的 SetParent 不能作为已经证实的回收机制

- 位置：[impact:23](../../../wiki/reference/prefabs/impact.md#L23)、第 53～54、121、132 行。
- 现文把两投影的寿命写为“靠父挂 portal”“随父 portal 移除”，并建议“必须像原版 SetParent，否则会留场”，混淆了位置关系与删除证据。
- 已核实的挂接：[lunarrift_portal](../../../raw/current/scripts/prefabs/lunarrift_portal.lua#L641-L646) 与 [multiplayer_portal](../../../raw/current/scripts/prefabs/multiplayer_portal.lua#L399-L410) 都仅 SetParent，未登记 Lua children。投影工厂 [portal_shadow_fn](../../../raw/current/scripts/prefabs/lunarrift_portal.lua#L707-L735) 与 [moonrockfxfn](../../../raw/current/scripts/prefabs/multiplayer_portal.lua#L429-L454) 没有独立定时／动画事件自删。
- 正常裂隙收尾 [第 435～445 行](../../../raw/current/scripts/prefabs/lunarrift_portal.lua#L435-L445) 是让 shadow 播消失动画，随后删除 portal；不能由此证明原生删除 shadow 的行为。第 53 行“只给 portal 播 shadow_disappear”的对象也应改为 shadow，实际接收动画调用的是 inst.shadow。
- 建议：分别记录确定的父挂、动画和宿主删除路径；原生随父删除保留未核验。模组复用应明确管理生命周期，不能将 SetParent 单独当作清理保证。同步摘要式快速判断、表格和案例结尾；不反向断言原版必现泄漏，不改只读源码。

### R03｜垃圾球落点的玩法副作用不止 combat 伤害

- 位置：[impact:21](../../../wiki/reference/prefabs/impact.md#L21)～23、第 35、65、130 行。
- 文中把落点主体概括为 30 点 combat 结算，表中虽然提到 workable／pickable 标签，却没交代对应操作。源码 [DoDamage](../../../raw/current/scripts/prefabs/junkball.lua#L348-L389) 的顺序是：符合条件的 workable 直接 Destroy；否则可采摘且非 intense 的 pickable 执行 Pick；再否则才走 combat:CanTarget／DoAttack。CHOP／DIG／HAMMER／MINE 及 NPC_workable 的门禁也不是纯战斗过滤。
- 此外 [第 402～425 行](../../../raw/current/scripts/prefabs/junkball.lua#L402-L425) 会处理地面物品停用与抛飞；[落点聚堆分支](../../../raw/current/scripts/prefabs/junkball.lua#L488-L527) 在条件允许时可敲除树桩、推 knockback 并生成 junk_pile。这些都属于该落点载体直接执行的行为。
- 建议：快速判断明确列“伤害＋破坏／采摘＋地面物品抛飞，formpile 条件下另有聚堆／击退”。正文用一段或表行补操作顺序与必要门禁，避免按现文把它用于只想做范围伤害的效果。
- targets 共享表对已记录的 worked／picked／attacked 对象去重；不要把“同一目标整轮一次”扩大为包括另一路物品抛飞在内的所有副作用都去重。无需复制完整 DoDamage 实现或扩展宿主战斗全貌。

### R04｜直接 GetAttacked 仍会进入受击侧事件链

- 位置：[impact:131](../../../wiki/reference/prefabs/impact.md#L131)，尤其“没有 attack 事件链”。
- [lunarrift_terraformer_explosion](../../../raw/current/scripts/prefabs/lunarrift_terraformer_explosion.lua#L22-L30) 确实不调用自身 Combat:DoAttack，而是直接调用受击者 Combat:GetAttacked；但 [GetAttacked](../../../raw/current/scripts/components/combat.lua#L570-L577) 会进入 GetAttacked_Internal，[未 blocked 的分支](../../../raw/current/scripts/components/combat.lua#L715-L720)仍推 attacked 并执行 onhitfn。
- 建议：明确“不经过攻击者 Combat:DoAttack，受击侧仍走 GetAttacked 的结算与条件性事件回调”。不要用“没有事件链”概括，从而误导读者排除受击监听。具体某个目标的反伤等响应不在本页展开，也不在未核验时保证触发。

### R05｜四个 newfx 符号覆写的锚点指到了客户端早退分支

- 位置：[ruins_cavein_obstacle_rubble_fx:23](../../../wiki/reference/prefabs/ruins_cavein_obstacle_rubble_fx.md#L23)。
- 该句“4 个 newfx1～newfx4 符号覆盖只在主模拟执行”后引用 scienceprototyper.lua 第 284～290 行；[该区间](../../../raw/current/scripts/prefabs/scienceprototyper.lua#L284-L290) 仅 SetPristine、客户端安装复制回调并返回，不能直接支撑四次 OverrideItemSkinSymbol。
- 真正的四符号操作在 [researchlab2_init_fn](../../../raw/current/scripts/prefabskin.lua#L3637-L3645)，该函数前面有 [placer／客户端门禁](../../../raw/current/scripts/prefabskin.lua#L3624-L3633)。页面第 44 行已引用正确函数。
- 建议：第 23 行链接改到真实覆写与门禁，复用已声明 prefabskin.lua，无需新增机制页。

### R06｜落石碎屑注册不是“零依赖”

- 位置：[ruins_cavein_obstacle_rubble_fx:54](../../../wiki/reference/prefabs/ruins_cavein_obstacle_rubble_fx.md#L54)。
- [实际注册](../../../raw/current/scripts/prefabs/ruins_cavein_obstacle.lua#L360-L361) 是 Prefab(name, rubblefn, nil, prefabs)，第四参明确传入预载依赖表；[该表](../../../raw/current/scripts/prefabs/ruins_cavein_obstacle.lua#L8-L14) 包含 rocks、flint、rock_break_fx 和 rubble 自身。
- 建议：写“无需外部初始化参数，注册自身无 assets，但沿用同文件 prefabs 预载表；FX 工厂不主动生成这些依赖实体”。不要把“不主动 Spawn 其他对象”写成“注册零依赖”，也不把依赖声明当生成点。

### R07｜落石宿主的 3～4 是尝试次数，还有出生点门禁

- 位置：[ruins_cavein_obstacle_rubble_fx:40](../../../wiki/reference/prefabs/ruins_cavein_obstacle_rubble_fx.md#L40)。
- 当前“Boss 冲撞后在出生点附近随机挑 3～4 个可走点”没有说明 [startobstacledrop](../../../raw/current/scripts/prefabs/minotaur.lua#L344-L378) 的前提：spawnlocation 存在，且 Boss 距出生点平方距离 <25²。
- 外层随机进行 3～4 次尝试，每次最多采样 8 个候选点；除了可通行还检查附近 quake_on_charge 实体。只有找到有效点才排延迟生成，因此不能保证最终落下 3～4 个障碍。
- 建议：该行补关键距离门禁与“尝试、成功才生成”，不要扩写远古守护者整套状态机；碎屑仍需之后采完宿主才生成的结论可保留。

## 已核对而未报问题的主要内容

- 阿比盖尔五注册的 bank／build／朝向和 animover 自删；召唤 51 帧生成、62 帧执行动作、69 帧失败清理，以及收回 25／26／32 帧；等级与惊吓共用 levelupfx，声音在 SG 而非 FX 工厂。
- 深渊石柱 COLLAPSE／FORMING_DELAY／FORMING／FORMING_ABOVE 分工，StartForming 的延迟、上下层子 FX、实体柱生成与组追踪移交；读档按保存的组 GUID 是否能解析决定生成，翻转和 idle_b 重新随机。
- 酸雨烟 GetPooledFx 的 ReturnToScene／RestartFx／pool，池失效时在播项最终按 animover 移除；蘑菇树本地三烟层、Follower、显隐门禁及树 onremove 显式清烟。Show／Hide 不等于删除，已排重播任务未取消的说明与源码一致。
- 病害蝇群代理两秒、本地队列独立，SetLoops 创建时读取；DoFX 的邻近计数、概率和重排公式。全树 diseaseable 字面检索保留候选，未发现原版 AddComponent 挂载调用，不把组件内部生成代码当现役已触发流程。
- 十种爆裂的 data、bloom、LightOverride、声音和本地缩放；换皮通过 SKIN_FX_PREFAB 动态选名，弹弓火药仅 crit 分支生成；未把依赖声明计成生成点。
- 水声工厂不自行播声；非专服洞穴控制器建三发射器、每 5 帧分配，0.2 秒降／升音量与有效池回调；控制器全文未见逐个移除发射器的回调，保留世界卸载结果未知。
- impact 字面生成点 15 处与页面枚举一致；houndstooth_proj 通过武器 LaunchProjectile 动态生成、has_damage_set 分支使用弹体武器，物理与 planar 基值、暗影倍率和尾迹门禁对上源码。七族的伤害仍须按 R03／R04 补全边界。
- 落石碎屑自身 animover、颜色一次复制、声音由生成者调用；炼金打印舱皮肤的 fx_prefab、主模拟父挂与镜像动画、客户端高亮登记、clear_fn 显式删除，均对上对应实现。

## 检视方法与未覆盖

- 全文阅读 8 个 Markdown 页面；用 Get-Content／rg 查看源码内容及根级、子目录中的 Lua 字面候选，并按对象追查动态入口。候选结果未人为截断；未将注册、依赖表、资源名、文件清单或注释当成生成。
- 定向查 29 个注册的 .po NAMES 键及必要宿主／皮肤名：本批注册中仅 lunarrift_terraformer_explosion 命中，简中为“月形爆裂”；阿比盖尔、花、块状废墟、炼金引擎、炼金打印舱名称与所引条目相符。
- 没有运行 validate_page、validate_source_links、validate_source_hashes、query_wiki、parser_probe、unittest 或 git diff --check；未执行 record，也没有重建缓存或索引。本报告不声称本批格式／导航／全量锚点／哈希／Lua 语法检查已通过。
- 页面中自记的旧机械通过数作为历史，不冒充本轮结果；source-reviewed 不升级。
- 未核验原生 SetParent 删除、实际网络音画、动画包内容与帧长、实际休眠／加载时序和外部模组行为。未凭这些缺口断言原版必现泄漏／不同步，也不要求修改只读游戏源码。
- 对定向引用做了人工内容比对，未进行全库索引可达性、注册 aliases 唯一性、来源声明完备性与行锚点边界的机械全量检查，这些留最后统一执行。

## 队列与后续

本轮完成后，固定 158 页队列的**首次整页独立内容检视为 158／158，尚未首检为 0 页**。这不等同所有问题已修复，也不表示整个 FX 库或运行时验收完成。

本轮 R01～R07 仍需修复后内容复验；其他轮次的历史待办以各自最新复验记录为准，本轮不自动关闭。先修内容、汇总各轮未收口项，准备好后再按用户要求统一做一次机械检查。只维护本报告与日志，不修改他方知识内容或计划。

## 修复处置（2026-11-07）

R01～R07 全部采纳，7 页已改并同步 `updated: 2026-11-07`（abigailsummonfx 无发现，未改）：

- R01 五页 sources 补 `languages/chinese_s.po`（abysspillar_fx／acidraindrop／diseaseflies／explode_small／grottopool_sfx）；impact 补 `blowdart_lava.lua`／`blowdart_lava2.lua` 及第 7–10 行锚点。
- R02 两个投影改为“无独立自删、父挂只保证位置、原生随父删除未核验”；裂隙收尾对象明确为 `inst.shadow` 播 `shadow_disappear`，随后删除 portal；示例结尾不再把 `SetParent` 当清理保证。
- R03 junkball 落点补 `DoDamage` 顺序（workable `Destroy`→pickable `Pick`→combat `DoAttack`）、地面物品抛飞、`formpile` 聚堆／击退；`targets` 去重限定 worked／picked／attacked 三路。
- R04 改“不走攻击者 `DoAttack`，但受击侧仍进入 `GetAttacked_Internal`，未 blocked 时推 `attacked` 并执行 `onhitfn`”。
- R05 四符号覆写锚点改到 `prefabskin.lua` 的 `researchlab2_init_fn` 与 placer／客户端门禁。
- R06 碎屑改为“工厂不接收外部参数、自身无 assets，但注册沿用同文件 `prefabs` 预载表（rocks／flint／rock_break_fx／碎屑自身）”。
- R07 `startobstacledrop` 补 `spawnlocation` 存在与 25² 距离门禁、3～4 次尝试且每次最多 8 个候选点、找到有效点才排延迟生成。

验证：7 页 `validate_page` 7/7；strict 271 处行引用／233 唯一目标／66 源文件，0 error／0 warning；record 6 页后 72 组来源 unchanged、0 changed；`git diff --check` 干净。未运行游戏、未改 raw、未提交推送。158／158 首次整页检视已完成，修后独立复验待做，本轮不自动收口。

## 修后独立内容复验（2026-11-07）

实际快照仍为 steam-buildid:26335394。查看 7 个修改页面的 diff、相关正文和必要源码；没有执行任何校验或测试脚本，上节机械结果属于修订方记录。本次只就 R01～R07 及修订引入的相邻问题复验。

| 原编号 | 本次内容结论 |
|---|---|
| R01 | 通过：五页已声明 languages/chinese_s.po；impact 第 11 行和第 129 行已补两份 blowdart_lava 资源来源及其行锚点。本次不验哈希。 |
| R02 | 局部待补：两个投影的父挂删除都已改为未知，案例也要求调用方管理生命周期；快速判断仍把裂隙的退场动画与父体删除写成两种投影共有。 |
| R03 | 局部待补：破坏、采摘、抛飞、聚堆／击退和去重范围已补；三条目标处理分支还须写明择一，摘要应反映副作用。 |
| R04 | 局部待补：已区分攻击者 DoAttack 和受击方 GetAttacked；还需明确 onhitfn 只有设置后才会调用。新增证据漏声明另列 R08。 |
| R05 | 通过：落石页第 23 行换为 prefabskin.lua 的主模拟／placer 门禁和实际四符号覆写区间。 |
| R06 | 通过：落石页第 54 行不再称“零依赖”，已区分无 assets 与注册的 prefabs 预载表。 |
| R07 | 通过：落石页第 40 行已补出生点及 <25² 门禁、3～4 次尝试与每次最多 8 个候选、有效点才排延迟生成。 |

### R02 残留｜快速判断仍把裂隙退场套给天体门

[impact:23](../../../wiki/reference/prefabs/impact.md#L23) 的“正常收尾由 portal 播动画并删除父体”紧接两个投影，容易读成共同协议。实际是 [裂隙的 on_rift_finished](../../../raw/current/scripts/prefabs/lunarrift_portal.lua#L435-L445)：宿主休眠直接 Remove；非休眠时给宿主和现存 shadow 播各自退场动画，随后删除宿主。天体门 [主模拟初始化](../../../raw/current/scripts/prefabs/multiplayer_portal.lua#L399-L410) 只显示该 FX 的父挂，没有这套裂隙退场流程。建议快速判断只合并“无独立自删、随父删除未核验”，单独说明裂隙动画分支；同步第 37、53 行的非休眠前提。

### R03 残留｜垃圾球按目标择一处理

[impact:23](../../../wiki/reference/prefabs/impact.md#L23)、[第 65 行](../../../wiki/reference/prefabs/impact.md#L65)、第 130 行仍用“先……再”或箭头表示三路。源码 [DoDamage 第 371～389 行](../../../raw/current/scripts/prefabs/junkball.lua#L371-L389) 是同一目标的 if／elseif：符合破坏条件则 Destroy，否则满足采摘条件才 Pick，否则 CanTarget 才攻击。建议用一句“按条件择一，另有地面物品抛飞与条件性聚堆／击退”，并同步第 7 行摘要，避免检索时仍只看到“落点 combat 结算”。已修的 targets 去重范围保留。

### R04 残留｜onhitfn 需要已经设置

[impact:131](../../../wiki/reference/prefabs/impact.md#L131) 的“未 blocked 时照常推 attacked 并执行 onhitfn”少了一层条件。[GetAttacked_Internal 第 715～720 行](../../../raw/current/scripts/components/combat.lua#L715-L720) 在未 blocked 时推 attacked；只有 self.onhitfn 非 nil 才调用。建议正文与第 141 行补正记录同步为“未 blocked 时推 attacked；若已设置 onhitfn，再调用它”。不必展开具体受击方的监听。

### R08｜impact 新引入 combat.lua，sources 未同步（P2）

[impact:131](../../../wiki/reference/prefabs/impact.md#L131) 新增两个 components/combat.lua 行锚点，但 [sources 第 11 行](../../../wiki/reference/prefabs/impact.md#L11) 没有该文件。补声明，最终统一处理对应基线；已有两个行锚点能支持 R04 的条件，不需要另找来源。

当前仍有 **4 组局部 P2，集中在 impact**。本次没有改知识页或他方修订记录，只更新本报告与日志。固定队列首次内容检视仍为 158／158；第 17 轮待上述局部补正后再收口，统一脚本检查留最后。

### R02／R03／R04／R08 复验后补正（2026-11-07）

- R02：impact 快速判断改为“两投影仅共有‘无独立自删、随父删除未核验’”，裂隙单列未休眠分支（`inst.shadow` 播 `shadow_disappear`、portal 播 `stage_3_disappear` 后 animover／10 秒兜底）与休眠直接 `Remove`；天体门单列为无对应动画／显式 `Remove(inst.fx)`。第 37、53、121 行同步非休眠前提。
- R03：摘要改为“落点对每个目标按条件择一破坏／采摘／攻击”；第 23、65、130 行统一用“择一”表述，明确进入一个分支后不再走后续分支，保留 `targets` 三路去重与抛飞／聚堆边界。
- R04：正文与补正记录改为“未 blocked 时推 `attacked`；仅当该 combat 设置了 `onhitfn` 才调用，未设置不调用”。
- R08：impact sources 补 `components/combat.lua` 并 record 本页。

验证：impact `validate_page` 1/1；strict 126 处行引用／110 唯一目标／27 源文件，0 error／0 warning；27 组来源 unchanged、0 changed；`git diff --check` 干净。未运行游戏、未改 raw、未提交推送；158／158 首检完成，第 17 轮待复验收口。

## 固定队列收口前核对（2026-11-07）

复核以当前 Markdown 和 Steam buildid `26335394` 的 Lua 为准。158 页全部完成首次独立内容检视。17 个编号轮次中，15 轮的实质内容问题可收口，第 02、15 轮各余一组 P2。此前 U15～U17 的 9 页已含在 158 页内，按原独立检视报告及收尾报告的整改记录计入；本次定向回看其中的两项 P1 和 strict 口径。 [九页独立检视](fx-independent-review-20261105.md) · [原收尾报告](fx-final-report-20261105.md)

| 轮次 | 当前内容状态 |
|---|---|
| 01 | R10、R11 通过。目标字段持有句柄，周期任务由战歌 buff 创建；歌手续排任务由歌手实体创建，句柄留在组件。`entityscript.lua` 已声明并附任务取消锚点。 [战歌页](../../../wiki/reference/prefabs/wathgrithr_spirit.md#L85) · [目标循环](../../../raw/current/scripts/prefabs/battlesongs.lua#L198-L214) · [歌手续排](../../../raw/current/scripts/components/singinginspiration.lua#L230-L238) |
| 03、04、06～11、13、14、16 | 沿用各轮已记录的修后内容结论。 |
| 05 | R10 通过。`shadowprotector` 和 `shadowworker` 均由 `MakeOblivionSeeker` 安装 timer，现文将它限定为突袭范围门禁的前提。 [斩击页](../../../wiki/reference/prefabs/firesplash_fx.md#L46) · [范围门禁](../../../raw/current/scripts/stategraphs/SGshadowwaxwell.lua#L339-L362) |
| 12 | R06、R07 的事实通过。`carrot_spinner` 双端初始化透明度，主模拟推进淡变。三种灯笼写近似半径和未夹取公式。两条灯笼行的现有链接落在常量与工厂，后续可补计算回调的就近锚点。 [透明度说明](../../../wiki/reference/prefabs/carnival_sparkle_bush.md#L114) · [灯笼表](../../../wiki/reference/prefabs/wormlight.md#L36) · [气球灯计算](../../../raw/current/scripts/prefabs/miniboatlantern.lua#L34-L38) · [红灯笼计算](../../../raw/current/scripts/prefabs/redlantern.lua#L31-L35) |
| 17 | R02～R04、R08 通过。已区分裂隙休眠与活跃态退场及天体门父挂，写明垃圾球按目标择一，`onhitfn` 仅设置后调用，`components/combat.lua` 已声明。天体门第 23、54 行的“只删 portal”宜删掉，以免被读成脚本有明确的宿主删除入口；正文已保留父挂结果未核验。 [影响页](../../../wiki/reference/prefabs/impact.md#L23) · [天体门父挂](../../../raw/current/scripts/prefabs/multiplayer_portal.lua#L399-L410) · [受击回调](../../../raw/current/scripts/components/combat.lua#L715-L720) |

### 收口前待办

1. **第 02 轮 R14（P2）。** 陷阱条目已正确限定同 tag 静默的条件。摘要、生成者表和叶索引仍把播放 out 写成“从无到有”。统一写成新源为复活调谐类型且 `_isrezattuned` 未置位时播放 out 并置 true。玩家可能已有调谐但标志为 false。 [摘要](../../../wiki/reference/prefabs/attune_out_fx.md#L6) · [生成者表](../../../wiki/reference/prefabs/attune_out_fx.md#L40) · [索引](../../../wiki/reference/prefabs/fx-table-index.md#L35) · [实际门禁](../../../raw/current/scripts/prefabs/player_common.lua#L443-L451)
2. **第 15 轮 R29（P2）。** R06、R19、R23、R25 的事实补正已通过。证据组仍有四类缺口，分布在四页。灼烧页补 `entityscript.lua` 的两个组件移除入口及藤蔓存读锚点。赛船旗与暗影主教页补 `AddChild` 登记和 Lua `children` 删除循环的证据。月亮祭坛页补 `SpawnSaveRecord` 的重建证据。同步实际采用的 sources，按规则刷新受影响页基线。 [R29 明细](fx-review-round15-20261107.md#r29--p2-新增证据组新结论未完整迁入正文证据与-sources)
3. **第 15 轮 R30（P3）。** 11 篇实质修订页仍写 `updated: 2026-11-05`。补实际修订日期和简短核验记录，作为同批维护收尾。 [R30 明细](fx-review-round15-20261107.md#r30--p3-维护日期11-篇实质改页仍写-2026-11-05)

本次仅核对文件内容，统一机械校验留待上述整改完成后执行一次。原收尾报告的 27 条全库 strict 警告为历史记录。最终校验需按当时实际结果区分本批问题、存量警告与并行页面的导航问题。

## 最终复验与收口（2026-11-07）

- **第 02 轮 R14 通过。** 摘要、生成者表、陷阱、验证记录及叶索引均以新源类型与 `_isrezattuned` 的实际值描述 `attune_out_fx`。跨 tag 换绑后仍持有调谐、标志却清为 false 的反例保留。 [调谐页](../../../wiki/reference/prefabs/attune_out_fx.md#L40) · [叶索引](../../../wiki/reference/prefabs/fx-table-index.md#L35) · [源码门禁](../../../raw/current/scripts/prefabs/player_common.lua#L443-L451)
- **第 15 轮 R29、R30 通过。** 灼烧页补两个删除入口及藤蔓存读的源码证据。赛船旗和暗影主教页补 `AddChild` 和 `children` 回收的证据。月亮祭坛页补 `SpawnSaveRecord` 证据。四页 `sources` 与正文一致。11 篇实质修订页均更新日期并补了验证记录，未改的温蒂页保留原日期。 [灼烧页](../../../wiki/reference/prefabs/trap_fumarole_burn_fx.md#L44) · [赛船旗页](../../../wiki/reference/prefabs/boatrace_start_flag.md#L56) · [暗影主教页](../../../wiki/reference/prefabs/shadow_bishop_fx.md#L62) · [月亮祭坛页](../../../wiki/reference/prefabs/moon_altar_break.md#L59) · [第 15 轮清单](fx-review-round15-20261107.md)

从 17 份轮次报告和 U15～U17 的九页独立检视报告提取范围，合计 158 篇不同知识页。Steam buildid 为 `26335394`。三项收尾检查各执行一次。

| 检查 | 最终结果 |
|---|---|
| `validate_page --check-indexes` | 158 篇知识页及 6 份 FX 索引共 164/164 通过。全库导航检查覆盖 1,172 个知识页登记、35 份索引，0 问题，退出 0。 |
| `validate_source_links --strict` | 全库扫描 1,207 页、35,217 处支持的引用，0 错误、27 警告，结果未截断。158 篇知识页及 6 份 FX 索引均 0 错误、0 警告。全库 strict 退出 1，27 条警告的类别和 12 篇组件页分布与[原收尾报告](fx-final-report-20261105.md#32-来源链接)一致。工具另计 993 处不支持的链接形式。 |
| `validate_source_hashes check --fresh-hours 0` | 固定队列 158 页，1,972 组比较全部 unchanged，0 未登记、0 无效页、0 受影响页，退出 0。 |

固定队列按以上范围收口。此前记下的灯笼就近锚点与天体门措辞作为非阻塞编辑项保留。本次只更新本报告与日志，未运行游戏、未刷新基线、未提交或推送。

