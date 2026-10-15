# FX 新页独立检视·第 12 轮（外围 L／M／O／Q）

核验日期：2026-11-06；源码：`raw/current/scripts`，Steam buildid **26335394**；知识仓 HEAD：`97afb4e9e6ca05390558769926f1f9c8598d0f70`。

**当前结论：尚未收口。修后独立复验 12 项通过，R06／R07 各余一处 P2 小修，详见文末。** 初检为 P1×0、P2×13、P3×1，下文初检内容保留历史；不自动关闭其他轮次的待复验项。只维护本报告并追加维护日志，保留其他 Agent 的装备页等改动；不改知识页、索引、来源基线、PLAN、台账、Skill 或工具，不 record、不提交推送。

## 范围与口径

11 页共 **1363 个物理行／255,072 字节**；核对逐注册工厂、必要调用、门禁、端、父挂、清理与示例。页面声明 183 组 sources、145 个不同文件；其中实际行引用涉及 137 个文件。没有把这 145 个文件全部审阅一遍，也没有把机械校验当成语义证明。

| 页面 | 本轮注册范围 | 行／字节 | 初检结论 |
|---|---|---|---|
| [crabking_claw_shadow](../../../wiki/reference/prefabs/crabking_claw_shadow.md) | 6 | 103／15,814 | 未发现 P1/P2 |
| [king_cane_fx](../../../wiki/reference/prefabs/king_cane_fx.md) | 主归属 6；哈奇光仅交叉对照 | 160／31,244 | 未发现 P1/P2 |
| [wormlight](../../../wiki/reference/prefabs/wormlight.md) | 16 个详述光／FX＋基础 wormlight_light 简述 | 123／32,450 | R06、R07、R08 |
| [gestalt_trail](../../../wiki/reference/prefabs/gestalt_trail.md) | 6 | 158／28,856 | R04、R05 |
| [grasspartfx](../../../wiki/reference/prefabs/grasspartfx.md) | 8 | 103／23,588 | R02、R03、R13 |
| [eye_charge](../../../wiki/reference/prefabs/eye_charge.md) | 4 | 129／28,357 | R01 |
| [carnival_sparkle_bush](../../../wiki/reference/prefabs/carnival_sparkle_bush.md) | 4 | 106／19,388 | R05、R06、R09、R11 |
| [damp_trail](../../../wiki/reference/prefabs/damp_trail.md) | 3 | 133／18,017 | 未发现 P1/P2 |
| [raindrop](../../../wiki/reference/prefabs/raindrop.md) | 2 | 119／15,625 | 无 P1/P2；R14 为 P3 |
| [shadow_pillar](../../../wiki/reference/prefabs/shadow_pillar.md) | 4 | 109／23,962 | R09、R10 |
| [saddle_shadow_fx](../../../wiki/reference/prefabs/saddle_shadow_fx.md) | 2 | 120／17,771 | R05、R12 |

按上述主归属合计 **62 个 FX／玩法注册**，其中 1 个基础光代理仅简述。64 个 ASCII 注册形式的 aliases 还含 `wormlight`／`wormlight_lesser` 两个明确不展开的食物本体，不能把 64 当作本轮完整建页数；哈奇光也不跨页重复计数。L 批旧汇总的拆分数字与当前主归属不完全一致，本报告采用现页范围，不修改台账。

## P2 问题

### R01｜诱鱼区的代码没有实现“最多 5 次重试”

- 位置：[eye_charge 第 60 行](../../../wiki/reference/prefabs/eye_charge.md#L60)、第 114 行。
- 依据：[chum_aoe.lua 第 49～74 行](../../../raw/current/scripts/prefabs/chum_aoe.lua#L49-L74)。计数递减后使用 `retry = inst._remaining_fish_spawn_attempts <= 0 and false or retry`；当原 retry 为 true 时，`true and false or true` 仍为 true。因此失败会继续排 0.5 秒任务，不能由初值 5 推导最多五次；达到鱼数上限的分支同样置 retry=true。
- 建议：分别写“计数初值为 5”与“当前表达式不能在耗尽时停止重试”；成功令 retry=false 或实体删除才会结束该重试链。不修改游戏源码，也不把持续重试说成一定持续生成鱼。

### R02｜鱼饵碎块也有 GENERIC 食物类型，素食鱼并未被排除

- 位置：[grasspartfx 第 57 行](../../../wiki/reference/prefabs/grasspartfx.md#L57)，“素食／莓果鱼不能”中素食鱼判断错误。
- 依据：[chumpiece 构造](../../../raw/current/scripts/prefabs/chum_aoe.lua#L184-L211)只添加 secondaryfoodtype=MEAT，没有覆盖 [edible 的主类型 GENERIC](../../../raw/current/scripts/components/edible.lua#L19-L25)。[VEGETARIAN 类型集合含 GENERIC](../../../raw/current/scripts/constants.lua#L2098-L2107)，[海鱼素食谱引用该集合](../../../raw/current/scripts/prefabs/oceanfishdef.lua#L156-L160)，[鱼脑按 edible 标签选目标](../../../raw/current/scripts/brains/oceanfishbrain.lua#L92-L109)。
- 建议：写明 GENERIC＋MEAT 双类型；杂食、素食、肉食鱼均可匹配，莓果专食不由这两类标签匹配。补 constants.lua 证据与 sources，修页方再按实际核验刷新基线。

### R03｜诱鱼区消散后只停止即时补位，周期生成任务仍在

- 位置：[grasspartfx 第 90 行](../../../wiki/reference/prefabs/grasspartfx.md#L90)“宿主消散后不再补位”，相关第 45 行。
- 依据：[DoDisperse](../../../raw/current/scripts/prefabs/chum_aoe.lua#L25-L32)置 persists=false 并延迟 2 秒删除，未取消周期任务；[OnPieceRemoved](../../../raw/current/scripts/prefabs/chum_aoe.lua#L76-L86)的即时补位才检查 persists；[SpawnChumPieces](../../../raw/current/scripts/prefabs/chum_aoe.lua#L88-L105)只查数量／海面，且 [0.5 秒周期任务](../../../raw/current/scripts/prefabs/chum_aoe.lua#L168-L176)没有保存成可取消字段。
- 建议：区分“块移除后的即时补位停止”与“宿主删除前周期任务仍可补到上限”；宿主实际 Remove 时再清表。不要泛化到另一条 waterplant_pollen 链。

### R04｜恶液背层有 SG 显式同步动画，不能写“无同步动画”

- 位置：[gestalt_trail 第 64 行](../../../wiki/reference/prefabs/gestalt_trail.md#L64)。
- 依据：[gelblob 挂上 SGgelblob](../../../raw/current/scripts/prefabs/gelblob.lua#L592)；[SG 的 _PlayAnimation／_PushAnimation](../../../raw/current/scripts/stategraphs/SGgelblob.lua#L17-L26)同时驱动本体和 inst.back，电击分支也[显式播放背层动画](../../../raw/current/scripts/stategraphs/SGgelblob.lua#L351-L361)。
- 建议：改为“无 Follower；背层动画由宿主 SG 显式驱动”，补一层直接关系导航与 SGgelblob.lua sources／证据，重新核验相关基线。无需展开整个 SG。

### R05｜三页把引擎 Follower 链到了 Lua 随从归属组件

- 位置：[gestalt_trail 第 73 行](../../../wiki/reference/prefabs/gestalt_trail.md#L73)及第 156 行；[carnival_sparkle_bush 第 51 行](../../../wiki/reference/prefabs/carnival_sparkle_bush.md#L51)；[saddle_shadow_fx 第 66 行](../../../wiki/reference/prefabs/saddle_shadow_fx.md#L66)及第 117 行。
- 依据：这些注册调用 entity:AddFollower／inst.Follower:FollowSymbol（如 [gestalt_cage](../../../raw/current/scripts/prefabs/gestalt_cage.lua#L124-L150)、[鞍具帧](../../../raw/current/scripts/prefabs/saddle.lua#L57-L75)）；链接目标 [components/follower 页](../../../wiki/reference/components/follower.md#L18)实际讲主人、忠诚与随从关系，其 [Lua 构造](../../../raw/current/scripts/components/follower.lua#L21-L30)不是引擎符号跟随对象。
- 建议：保留“Follower（引擎）”说明与真实调用锚点，移除这些误导互链；无需新建机制页，也不要推荐 AddComponent("follower") 来解决符号跟随。

### R06｜不能从缺少 Lua netvar 推导客户端参数／画面不会更新

- 位置：[wormlight 第 51 行](../../../wiki/reference/prefabs/wormlight.md#L51)“复用时要自己补同步”、第 80 行“参数变化只发生在主模拟”；[carnival_sparkle_bush 第 96 行](../../../wiki/reference/prefabs/carnival_sparkle_bush.md#L96)“客户端副本停在 alpha 0”。wormlight 第 24 行的边界说明也应一起核对。
- 依据：[lanternlight](../../../raw/current/scripts/prefabs/mininglantern.lua#L206-L230)、[queen_torchlight](../../../raw/current/scripts/prefabs/queen_torch.lua#L370-L404)、[squideyelight](../../../raw/current/scripts/prefabs/squid.lua#L299-L324)均有引擎 Network／Light；[carrot_spinner](../../../raw/current/scripts/prefabs/carrot_spinner.lua#L58-L99)有 Network／AnimState，主模拟更新用 SetMultColour。Lua 调用只在主模拟、没有自定义 netvar，均不能单独证明引擎联网组件的复制结果。
- 建议：只陈述 Lua 写入端与有无自定义 netvar；引擎同步及客户端最终显示未实测时保留缺口，删除“必需自己补同步”和“客户端停在初值”的结论，避免重复叠加同步方案。

### R07｜三种灯笼的闪烁不是以 1.2 为中心的 ±0.1，也不能保证双端同相

- 位置：[wormlight 第 36～42 行](../../../wiki/reference/prefabs/wormlight.md#L36)三条灯笼参数、第 50 行“两端结果一致”。
- 依据：[miniboatlantern](../../../raw/current/scripts/prefabs/miniboatlantern.lua#L32-L39)、[redlantern](../../../raw/current/scripts/prefabs/redlantern.lua#L29-L36)、[yots 灯](../../../raw/current/scripts/prefabs/yots_worm_lantern.lua#L244-L251)实际为 `f=.5+.25*(sin(t)+sin(t+2)+sin(t+.7777))`，半径 `1.2+.1*f`、颜色为基色 `+(2/255)*f`，不是写出的对称范围；源码注释的 0～1 也不是严格数学夹取。任务 [以各端创建时的 GetTime() 为 starttime](../../../raw/current/scripts/prefabs/miniboatlantern.lua#L218-L219)，不是共享网络起点。
- 建议：保留实际公式或准确注明推导范围；写“各端独立按相同算法闪烁，起始相位不保证一致”，不把本地确定性计算等同双端逐时刻一致。

### R08｜光莓代理向骑手 push bloom 的分支没有原生初始化入口

- 位置：[wormlight 第 23 行](../../../wiki/reference/prefabs/wormlight.md#L23)“推送／收回目标（含骑手）的 bloom”。
- 依据：[OnOwnerChange 第 221～228 行](../../../raw/current/scripts/prefabs/wormlight.lua#L221-L228)必须已有 `_popbloom ~= nil` 才会进入向新骑手 push 的分支；[工厂初始化](../../../raw/current/scripts/prefabs/wormlight.lua#L298-L325)没有赋该字段。本次全 scripts 的 .lua 反查 `_popbloom` 仅此四处命中，赋值均在这个门禁内部。[light_ontarget](../../../raw/current/scripts/prefabs/wormlight.lua#L246-L275)确实对直接目标 push，[finish](../../../raw/current/scripts/prefabs/wormlight.lua#L278-L291)也会对当前骑手尝试 pop。
- 建议：拆开目标的有效 push／pop、骑手的收尾调用和受门禁限制的 push；不能凭存在 pushbloom 调用就保证原版骑手获得 bloom，也不据此断言实际渲染结果。

### R09｜两处仍把 SetParent 当成已证实的宿主删除级联

- 位置：[shadow_pillar 第 73 行](../../../wiki/reference/prefabs/shadow_pillar.md#L73)“贴片随宿主移动／删除”；[carnival_sparkle_bush 第 42 行](../../../wiki/reference/prefabs/carnival_sparkle_bush.md#L42)“父挂实体的连带清理在引擎层级”。
- 依据：原版分别是 [裸 SetParent 挂贴片](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L120-L126)与 [裸 SetParent 挂蛾子灯](../../../raw/current/scripts/prefabs/critters.lua#L338-L343)。[AddChild 才登记 Lua children](../../../raw/current/scripts/entityscript.lua#L921-L935)；[Remove 中的子实体循环](../../../raw/current/scripts/entityscript.lua#L1779-L1783)不能证明上述裸父挂实体的引擎级回收。影柱正常路径另有 [KillFX 显式收尾](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L84-L94)。
- 建议：保留跟随变换与已见 KillFX／Remove；裸父挂的宿主删除级联写“未核验”，自建示例交代正常收尾和提前删除责任。也不能反向断言一定泄漏。

### R10｜影柱 newpillars 去重分支实际上读不到写入的实体键

- 位置：[shadow_pillar 第 95 行](../../../wiki/reference/prefabs/shadow_pillar.md#L95)“1.0 的 newpillars 去重……只在 flying 分支参与”。
- 依据：[IsNearOther](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L662-L668)使用 ipairs(newpillars)，而 [唯一加入方式](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L699-L711)是 `newpillars[ent] = pt`；[施法任务传入的是空表](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L764-L767)，没有给它建立整数序列的原生路径。因此即便 flying 分支执行到这个 helper，它也遍历不到上述实体键，不产生期望的 1.0 点位去重。
- 建议：保留原有运算符优先级分析，再补“实体键表配 ipairs 导致该去重在当前调用链不生效”。不要混淆 [0.9 半径驱散已存在柱](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L137-L143)这一独立有效路径；不修改游戏实现。

### R11｜风扇本地示例缺少初值应用和持续转动状态

- 位置：[carnival_sparkle_bush 第 82～84 行](../../../wiki/reference/prefabs/carnival_sparkle_bush.md#L82)。
- 依据：示例只监听之后的 dirty，没有读取创建时的开关；开时也只 PlayAnimation("spin_pre")，未排 spin_loop。原版 [ToggleSpin](../../../raw/current/scripts/prefabs/fan_wheel.lua#L10-L23)维护 `_toggle`、pre→loop／pst→idle，并在 [监听之后立即应用当前 _isspinning](../../../raw/current/scripts/prefabs/fan_wheel.lua#L57-L61)。
- 建议：作为可复用案例，补最小状态切换函数、立即应用当前值与相应循环／退出；或者明确降为只展示事件接线的节选，不能把现段当成已实现开关驱动的跟手风扇。沿用壳 onremove 清理，补改后语法检查，仍不宣称运行通过。

### R12｜鞍具案例把已知失败的旧颜色解绑写成成功步骤

- 位置：[saddle_shadow_fx 第 76 行](../../../wiki/reference/prefabs/saddle_shadow_fx.md#L76)“换主会先解旧宿主颜色子级”，与本页第 32／99 行的实参陷阱矛盾。
- 依据：[SetOwner](../../../raw/current/scripts/prefabs/saddle.lua#L181-L204)调用 DetachChild(inst.fx)，挂接却是 AttachChild(inst)；[ColourAdder 的 children 按传入实体作键](../../../raw/current/scripts/components/colouradder.lua#L24-L49)，传帧表不能删除壳的登记。
- 建议：案例引言同步改为“尝试解绑，但当前实参不匹配，旧登记未因此移除”；明确原版限制或给实际修正的自建实现，不要让单章读取只得到成功解绑的旧结论。第 23 行泛称“换主清空脚印池”也应收窄为实际由玩家 owner 切回非玩家 owner 的分支，第 161～177 行不是所有换主都清池。

### R13｜渔网物品也联网，快速判断仍保留“整条链唯一”旧结论

- 位置：[grasspartfx 第 24 行](../../../wiki/reference/prefabs/grasspartfx.md#L24)“整条渔网链唯一联网实体”及第 62 行。第 49 行已经正确限定 visualizer＋FX 子链，并承认物品 fishingnet 另有 Network。
- 依据：[fishingnet 物品第 32～39 行](../../../raw/current/scripts/prefabs/fishingnet.lua#L32-L39)有 AddNetwork；[visualizer 构造](../../../raw/current/scripts/prefabs/fishingnetvisualizer.lua#L12-L35)没有，FX 本身则有网络组件。
- 建议：把第 49 行已核验的子链范围同步到快速判断与导航，避免入口摘要覆盖正确正文。fishingnet.lua 已在 sources，不必为此增加新依赖。

## P3 问题

### R14｜两条正确限制被标成“误区”

- 位置：[raindrop 第 104～105 行](../../../wiki/reference/prefabs/raindrop.md#L104)：“SpawnPrefab(frostbreath) 不会自己冒气”“frostbreath 不能套用 raindrop 的池”均是本页随后证实的正确事实。
- 建议：改“边界”，或把误区标题改成真正错误的正命题，例如“生成 frostbreath 就会自动发射”。不需修改技术正文，也不因体例问题升级成 P2。

## 实际校验与边界

- **格式／导航**：11/11 passed；`--check-indexes` 全库导航为 1143 知识页／34 索引，0 issue；保留并发装备页改动。
- **strict 源码引用**：882 处行引用、698 唯一目标、137 个文件，0 error／0 warning；跳过 1 个不属于 Lua／po 的普通链接，未把它算成源码证据。
- **来源基线**：183 组 unchanged；changed／sources_changed／unrecorded／invalid 均 0。只 check，未 record。报告指出的缺失语义证据不能靠此项通过豁免。
- **名称／别名**：定向读取 39 个不同 po 引用区间；全文件核对 64 个 ASCII aliases 的 NAMES 键，命中 charlierose／charlieresidue 与明确不展开的两种光莓物品，未把宿主或皮肤名冒充 FX 官方名；这些 aliases 在现存 Prefab 页无跨页主归属冲突。
- **候选反查**：完整扫描 4095 个 Lua 文件（4093 UTF-8、2 个 latin-1 回退，无文件跳过），64 个 ASCII aliases 共 660 个名称－行配对，100 个在对应页面 sources 之外。逐类检查外部命中，主要为光莓物品用途、预载／依赖／scrapbook、地图消费者及 acidraindrop 等子串；不把此计数当生成点数。另定向追查 SGgelblob、食物类型集合、_popbloom 等必要链；未穷举任意运行时拼名。
- **章节读取**：11 页分别以标题原文“注册与构造”读取，实际取得 results／完整 ranges。wormlight 默认预算曾返回 exit 0 但 results=[]／truncated=true，随后提高 `--max-bytes 65536` 补读，全 11 页在该预算下均 results=1、truncated=false；不把最初 exit 0 当成读取成功。缓存未重建，检索以现有 Markdown 与源码反查进行。
- **Lua 语法**：当前 14 段 Lua 逐段通过现有 parser_probe `--syntax-only`，临时文件经 apply_patch 创建、核对围栏一致后删除。语法检查未执行函数，不能豁免 R11 的状态逻辑缺失。
- **输入稳定性**：初检前后 11 页＋145 个声明来源共 156 个输入的内存哈希一致；报告的 73 个本地链接目标存在且行号区间有效，14 个临时文件无残留。
- **未测**：未启动 DST；引擎父子级联、网络 Light／AnimState 最终复制、动画资源、实际视觉、专服／晚加入／存读档时序不作运行结论。没有自行修复原版表达式／表遍历等源码问题。

## 后续

修页方可集中处理 R01～R13 涉及的七篇页面，R14 可顺手处理；同步检查同页入口与案例，不为本轮问题加重 Skill。增加 SGgelblob.lua／constants.lua 等来源时，仅刷新真正重新核验页面的基线；本报告不代为 record。修后按这些条目定向复验，再决定收口。

固定 158 页队列，本轮新增独立检视 **11 页**，累计 **98/158 页，余 60 页**。这是审阅覆盖，不表示 98 页全部收口；Round05 等旧轮次未复验事项保持原状态。后续每轮 12～15 页，约还有 **4～5 轮初检**，另加实际必要的修后复验。

## 2026-11-06 修后独立复验：剩余两处

八篇有问题页面均已修改；另外三篇与初检哈希一致。逐项核对修改、相关上下文与必要源码，初检的 145 个声明来源文件均未变化。新增声明来源为 SGgelblob.lua；constants.lua 原已被同批其他页面声明，本次另加入 grasspartfx 的依赖。修页方日志的“全部采纳”是处置记录，不代替下表独立验收。

| 项目 | 当前位置与复验结论 |
|---|---|
| R01 | eye_charge 第 60、114 行。已区分初值 5、失败继续重试与成功／删除终止；不再承诺最多五次。通过。 |
| R02 | grasspartfx 第 23、34、57 行。GENERIC＋MEAT 双类型、素食鱼可匹配与莓果专食边界均正确；constants.lua 来源与证据已补。通过。 |
| R03 | grasspartfx 第 45、90 行。已区分消散后仍在的周期任务与受 persists 门禁的即时补位，未泛化另一宿主链。通过。 |
| R04 | gestalt_trail 第 64 行。已补宿主 SG 显式驱动背层动画、SG 挂载与电击证据，SGgelblob.lua 已声明来源。通过。 |
| R05 | gestalt_trail 第 73、156 行；carnival 第 51 行；saddle 第 66、117 行。误链已移除，保留引擎 Follower 与源码调用。通过。 |
| R06 | wormlight 第 24、51、80 行及 carnival 第 114 行。原“无自定义 netvar 即不复制”的推断已撤回；但 carnival 改文新把透明度初始化说成只在主模拟，尚需下述小修。未完全通过。 |
| R07 | wormlight 第 36、37、42、50 行。双端相位说明通过，范围仍误作严格 0～1／1.2～1.3，见下文。未完全通过。 |
| R08 | wormlight 第 23、61 行。已拆开直接目标 push、骑手 push 门禁和结束时 pop 尝试，不再保证骑手获得 bloom。通过。 |
| R09 | shadow_pillar 第 73 行；carnival 第 42 行。裸父挂的删除级联已注明未核验，正常 KillFX 收尾仍保留。通过。 |
| R10 | shadow_pillar 第 95 行。已补实体键表配 ipairs 导致去重无效，保留优先级分析并区分 0.9 半径驱散。通过。 |
| R11 | carnival 第 66～104、107 行。补 ToggleSpin、SoundEmitter、pre→loop／pst→idle 与立即应用当前开关，仍监听壳删除清理；完整片段语法通过。通过。 |
| R12 | saddle 第 23、76 行。案例已承认 DetachChild 实参不匹配，清池说明收窄到玩家切回非玩家的实际使用链。通过。 |
| R13 | grasspartfx 第 24、62 行。快速判断与导航均限定 visualizer＋FX 子链，并注明 fishingnet 物品另有 Network。通过。 |
| R14 | raindrop 第 104、105 行。两条正确限制已改“边界”。通过。 |

### 剩余 R06（P2）：初始化透明度在双端，淡变推进才在主模拟

[carnival_sparkle_bush 第 114 行](../../../wiki/reference/prefabs/carnival_sparkle_bush.md#L114)现写“工厂在主模拟设 SetMultColour(1,1,1,0)”。实际 [carrot_spinner.lua 第 69 行](../../../raw/current/scripts/prefabs/carrot_spinner.lua#L69)在 SetPristine 和 [客户端早退第 80～82 行](../../../raw/current/scripts/prefabs/carrot_spinner.lua#L78-L82)之前，主模拟与客户端都会执行初始化；后续的 alpha 字段、timer 与 updatelooper 才只在主模拟添加。

建议改为：“工厂两端先设透明度 0；淡变的 Lua 写入由主模拟 updatelooper 推进；引擎颜色复制未实测。”保留本次已经修正的同步边界。这是 R06 改文引入的端描述误差，继续记在原项，不增开问题编号。

### 剩余 R07（P2）：未夹取的正弦组合不能当严格 0～1

[wormlight 第 36 行](../../../wiki/reference/prefabs/wormlight.md#L36)仍明确写 flicker 0～1，第 37、42 行同样用严格 1.2～1.3／颜色 +0～2/255 范围。源码 [miniboatlantern 第 34～38 行](../../../raw/current/scripts/prefabs/miniboatlantern.lua#L34-L38)、[redlantern 第 31～35 行](../../../raw/current/scripts/prefabs/redlantern.lua#L31-L35)、[yots 灯第 247～251 行](../../../raw/current/scripts/prefabs/yots_worm_lantern.lua#L247-L251)没有 clamp；三者等价于 `f=.5+.25*(sin(t)+sin(t+2)+sin(t+.7777))`。

合并正弦后幅值为 `sqrt((1+cos(2)+cos(.7777))^2+(sin(2)+sin(.7777))^2)`，约 2.06778494，因此 f 的连续公式理论范围约 -0.01694623～1.01694623，半径约 1.19830538～1.30169462。这是数学推导，不是运行测量；不要求把这些小数全写进 Wiki。直接保留实际未夹取公式，或把半径标成“约 1.2～1.3”并去掉严格 f=0～1／颜色范围即可；相位修订无需重做。

本次实际校验：

- 原 11 页格式／导航通过，1143 知识页／34 索引／0 issue。
- strict 894 处行引用、707 唯一目标、138 个文件，0 error／0 warning；跳过 1 个普通链接。
- 185 组来源全部 unchanged，changed／sources_changed／unrecorded／invalid 均 0；修页方已刷新两篇新增依赖的基线，本次只 check、未 record。
- 八篇改页的正文源码行引用均在 sources 声明内；各选相关章节以标题原文 read，8 次均实际返回完整结果且未截断（max-bytes 65536），未重建缓存。
- 唯一 Lua 实质改动是 carnival 风扇例：提取当前完整围栏，核对一致后 parser_probe --syntax-only exit 0；临时文件经 apply_patch 创建、删除，无残留。其余 13 段未改代码沿用初检语法结果，不称重新执行。
- 复验前后 11 页＋146 个声明来源共 157 个输入哈希未变。未运行 DST，网络复制、父挂级联和视觉效果仍保留未验证范围。

本轮 **12 项通过、2 项待小修，暂不收口**；剩余仅 wormlight 三条灯笼范围与 carnival 一句初始化端描述。累计仍为 **98/158 页，余 60 页**；没有新增初检页数，也不关闭其他轮次待办。本次仅更新报告并追加日志，未改知识页、索引、基线、PLAN、台账、Skill、工具或 raw，未提交推送。
