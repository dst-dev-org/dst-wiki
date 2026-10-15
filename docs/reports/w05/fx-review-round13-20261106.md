# FX 第 13 轮独立检视：外围 P／R／S／T＋U1

检视日期：2026-11-06。源码快照：`steam-buildid:26335394`（实读 `raw/current/BUILD.txt`）；K HEAD：`97afb4e9e6ca05390558769926f1f9c8598d0f70`。

**当前结论：本批可收口。2026-11-07 独立复验 R01～R16 全部通过，未新增本批 P1/P2；全局导航另有并行在建 queen-torch 未登记，详见文末。** 初检 P1×0、P2×16、P3×0，涉及 14 页中的 9 页及相关索引；下文保留初检历史。本次只维护报告与日志，不改知识页、索引、来源基线、PLAN、台账、Skill、工具或 raw，不 record、不提交推送。保留其他 Agent 的装备页等工作区改动；旧轮待复验项不自动关闭。

## 范围与口径

14 页共 **1717 个物理行／312,048 字节**，声明 **198 组 sources／147 个不同来源文件**。按工厂、必要消费者、门禁、端、生成点、父挂、清理与示例逐页核验；没有把所有声明来源全文重审，也未运行 DST。

| 页面 | 本轮登记范围 | 行／字节 | 独立检视结论 |
|---|---:|---:|---|
| [abigail_retaliation](../../../wiki/reference/prefabs/abigail_retaliation.md) | 5 | 108／17,458 | R01、R02 |
| [oceantree_ripples_short](../../../wiki/reference/prefabs/oceantree_ripples_short.md) | 12 | 111／29,002 | R15；相关索引 R16 |
| [pocketwatch_warp_marker](../../../wiki/reference/prefabs/pocketwatch_warp_marker.md) | 5 | 114／23,980 | 未发现 P1/P2 |
| [sleepcloud](../../../wiki/reference/prefabs/sleepcloud.md) | 5 | 121／21,453 | R14 |
| [staff_lunarplant_fx](../../../wiki/reference/prefabs/staff_lunarplant_fx.md) | 6 | 170／32,261 | R03、R04 |
| [shadow_trap](../../../wiki/reference/prefabs/shadow_trap.md) | 2 | 114／16,381 | R05、R06 |
| [shadow_despawn](../../../wiki/reference/prefabs/shadow_despawn.md) | 4 | 180／31,040 | R07 |
| [scorchedground](../../../wiki/reference/prefabs/scorchedground.md) | 4 | 115／20,976 | 未发现 P1/P2 |
| [fx_book_mount](../../../wiki/reference/prefabs/fx_book_mount.md) | 12 | 93／16,291 | 未发现 P1/P2 |
| [goosplash](../../../wiki/reference/prefabs/goosplash.md) | 9 | 118／15,413 | 未发现 P1/P2 |
| [nightmarelightfx](../../../wiki/reference/prefabs/nightmarelightfx.md) | 8 | 133／29,584 | 未发现 P1/P2 |
| [oceanhorror_ripples](../../../wiki/reference/prefabs/oceanhorror_ripples.md) | 2 | 116／18,317 | R08 |
| [rocky_snip_fx](../../../wiki/reference/prefabs/rocky_snip_fx.md) | 4 | 115／20,933 | R09、R10 |
| [rope_bridge_fx](../../../wiki/reference/prefabs/rope_bridge_fx.md) | 3 | 109／18,959 | R11～R13 |

合计 81 个登记名，包括 `rocky_boss_shadow` 这个完整 Boss 注册，但本页只审其表现链，不能称已完成 Boss 全文 Wiki。`fx_book_mount` 是主题页别名，不是真实注册名，未计入 81。海树页已增补大树干两注册，本轮按当前 12 名计，不沿用旧批次的 10 名。

## P2 问题

### R01｜幻灵形态的三倍怨咒时长不是死代码

- 位置：[abigail_retaliation 第 31 行](../../../wiki/reference/prefabs/abigail_retaliation.md#L31)、[第 87 行](../../../wiki/reference/prefabs/abigail_retaliation.md#L87)。现文把组件的四参签名直接套到实体包装方法，判定 `buffer` 恒 nil。
- 依据：[ApplyDebuff 第 394 行](../../../raw/current/scripts/prefabs/abigail.lua#L380-L403) 调的是实体 `target:AddDebuff(..., nil, nil, nil, inst)`；[EntityScript:AddDebuff](../../../raw/current/scripts/entityscript.lua#L2140-L2153) 接收六参，转交 `components.debuffable:AddDebuff(name,prefab,data,buffer)`；[RegisterDebuff／AddDebuff](../../../raw/current/scripts/components/debuffable.lua#L72-L105) 及 [Debuff:AttachTo／Extend](../../../raw/current/scripts/components/debuff.lua#L35-L55) 都继续透传。最终 [buff_OnExtended](../../../raw/current/scripts/prefabs/abigail.lua#L1051-L1060) 能收到原阿比盖尔，且 [SetToGestalt](../../../raw/current/scripts/prefabs/abigail.lua#L660-L663) 确实加 `gestalt` 标签。
- 建议：撤回“多参被忽略／死代码”，写明普通基础时长 2 秒、来自带 `gestalt` 标签施加者时乘 3；同步注册表、陷阱与验证说明。补实际使用的 `components/debuff.lua` 来源后按规则刷新本页基线，不改源码。

### R02｜阿比盖尔页有三份已引用源码未进入 sources

- 位置：[abigail_retaliation 第 11 行](../../../wiki/reference/prefabs/abigail_retaliation.md#L11)、[第 49 行](../../../wiki/reference/prefabs/abigail_retaliation.md#L49)、[第 87 行](../../../wiki/reference/prefabs/abigail_retaliation.md#L87)。
- 正文引用了 `components/incinerator.lua`、`standardcomponents.lua`、`components/debuffable.lua`，frontmatter 三者均未声明。严格链接会验证它们的路径与行区间，但当前哈希检查的 198 组不包含这些未声明依赖，不能据 unchanged 认定来源列用完整。
- 建议：保留的结论实际核验后补来源；不保留的证据和断言一起删除。与 R01 合并处理本页来源及基线。

### R03｜没有 colouraddersync 不能证明联网 FX 的颜色只在主机可见

- 位置：[staff_lunarplant_fx 第 27 行](../../../wiki/reference/prefabs/staff_lunarplant_fx.md#L27)、[第 66 行](../../../wiki/reference/prefabs/staff_lunarplant_fx.md#L66)、[第 148 行](../../../wiki/reference/prefabs/staff_lunarplant_fx.md#L148)。
- 依据：[魔杖 FX 工厂](../../../raw/current/scripts/prefabs/staff_lunarplant.lua#L235-L266) 与 [剑刃 FX 工厂](../../../raw/current/scripts/prefabs/sword_lunarplant.lua#L277-L304) 都联网，仅主模拟加 `colouradder`；[ColourAdder:OnSetColour](../../../raw/current/scripts/components/colouradder.lua#L66-L80) 无 sync 组件时调用 `AnimState:SetAddColour`。这些 Lua 能证明写入端和调用路径，不能独自证明引擎对该 AnimState 属性的复制行为。
- 建议：改成“Lua 附加色由主模拟写入，没有本页自定义颜色 netvar；跨端复制与实际外观未运行核验”。不要继续以组件页的同款断言作为引擎结论的证据；本轮不扩修组件页。

### R04｜滚雪球 FX 监听的是宿主事件，不是 FX 自身事件

- 位置：[staff_lunarplant_fx 第 55 行](../../../wiki/reference/prefabs/staff_lunarplant_fx.md#L55)、[第 77 行](../../../wiki/reference/prefabs/staff_lunarplant_fx.md#L77)。现文写“FX 收到／自身 onremove、enterlimbo”。
- 依据：[snowman.lua 第 166～172 行](../../../raw/current/scripts/prefabs/snowman.lua#L166-L172) 的调用是 `fx:ListenForEvent(..., DetachRollingFx, inst)`，第三参 `inst` 为雪球宿主；[DetachRollingFx](../../../raw/current/scripts/prefabs/snowman.lua#L124-L135) 从宿主取 `_rollingfx`，解父、保世界坐标和朝向，再 `KillFx`。
- 建议：明确“FX 监听宿主雪球的 onremove／enterlimbo”；停止推动、无雪等另外两条宿主路径单独保留。不要让读者以为外部删除 FX 本身会触发这条解绑流程。

### R05｜noattack 实体标签与 SG 状态标签被混为一谈

- 位置：[shadow_trap 第 102 行](../../../wiki/reference/prefabs/shadow_trap.md#L102)。现文说带 `noattack` 标签的目标仍可命中，“只跳过 0 伤害 attacked”。
- 依据：[探测／命中排除表](../../../raw/current/scripts/prefabs/shadow_trap.lua#L115-L123) 使用实体标签；[TryTrapTarget 第 165～166 行](../../../raw/current/scripts/prefabs/shadow_trap.lua#L162-L179) 则查 `v.sg:HasStateTag("noattack")`。两者不是同一个条件。
- 建议：分别写：实体 `noattack` 只被探测表排除，爆炸命中表不排除；命中后是否跳过零伤害事件，另由 SG 当前 `noattack` 状态判定。只带实体标签不足以跳过该事件。

### R06｜目标 death 只令贴片 KillFX，onremove 才先解父

- 位置：[shadow_trap 第 43 行](../../../wiki/reference/prefabs/shadow_trap.md#L43) 把 death／onremove 合并成“先解父保坐标”。
- 依据：[OnSetTarget 第 372～380 行](../../../raw/current/scripts/prefabs/shadow_trap.lua#L372-L380)：death 回调仅调用 `KillFX()`；onremove 回调才取世界坐标、`SetParent(nil)`、写回坐标并调用 `KillFX()`。两条均可收尾，但退出父挂的时点不同。
- 建议：分开两事件动作，不改变 `KillFX` 醒着播 pst、睡眠直接删的说明。

### R07｜自定义光束施法者示例缺少 health 与 combat 前提

- 位置：[shadow_despawn 第 125～130 行](../../../wiki/reference/prefabs/shadow_despawn.md#L125-L130)、[第 149 行](../../../wiki/reference/prefabs/shadow_despawn.md#L149)。目前只强调 `GetAOEAttackTagSet()` 与跟随符号。
- 依据：[InitBlast 第 356～363 行](../../../raw/current/scripts/prefabs/shrouden_optic_blast.lua#L353-L367) 立即解引用 `caster.components.health:IsDead()`；[UpdateBeamAOE](../../../raw/current/scripts/prefabs/shrouden_optic_blast.lua#L200-L212) 继续使用 health 与 tagset；[AOEUtil.Attack](../../../raw/current/scripts/aoeutil.lua#L77-L78) 立即访问攻击者 combat，并在 [第 118～129 行](../../../raw/current/scripts/aoeutil.lua#L118-L129) 用其 CanTarget／DoAttack。只补方法和动画符号的普通实体仍会报错。
- 建议：父节前提和示例注释同步要求有效、存活且有 `health`、`combat`、兼容 tagset 接口与动画符号的施法者，或在示例入口检查必要条件。无需扩写完整 Boss 工厂。

### R08｜海上涟漪案例仍保证未经核验的父子级联回收

- 位置：[oceanhorror_ripples 第 60 行](../../../wiki/reference/prefabs/oceanhorror_ripples.md#L60)、[第 70 行](../../../wiki/reference/prefabs/oceanhorror_ripples.md#L70)。表格写“涟漪随引擎父子回收”，案例更写“不用生成方做任何事，宿主死亡即随之消失”；同页验证范围却说明引擎级回收未验证。
- 依据：[生成点](../../../raw/current/scripts/prefabs/oceanshadowcreature.lua#L378-L384) 只有裸 `SetParent`；[涟漪工厂](../../../raw/current/scripts/prefabs/oceanshadowcreature.lua#L452-L478) 没有独立移除路径；[宿主 OnRemove](../../../raw/current/scripts/prefabs/oceanshadowcreature.lua#L279-L284) 删除的是宿主的父挂点，不是 `_ripples`。这些不足以保证自定义宿主的回收。
- 建议：统一为“原版依赖父挂与非持久化，引擎级联未核验”；复用案例需要明确可自管引用、撤销时显式 Remove，不能一边保留缺口一边保证自动消失。无需据此反向断言原版必然泄漏。

### R09｜剪击 PostUpdate 收尾没有删除本地线条实体

- 位置：[rocky_snip_fx 第 64 行](../../../wiki/reference/prefabs/rocky_snip_fx.md#L64)，把本地子体说成“0.5s 后随 PostUpdate 收尾”“本地子体自管，两条路径独立”。
- 依据：[snip_PostUpdate](../../../raw/current/scripts/prefabs/rocky_boss.lua#L941-L991) 创建 `_fx` 后父挂联网壳；结束段只 `ClearLineDistortion`、清钩子、注销 PostUpdate，没有 `_fx:Remove()`。[snip_OnRemoveEntity](../../../raw/current/scripts/prefabs/rocky_boss.lua#L937-L939) 也只清屏幕失真。相比之下，[dash_OnRemoveEntity](../../../raw/current/scripts/prefabs/rocky_boss.lua#L1057-L1065) 才显式删除 `_fx`。
- 建议：把“失真／更新回调停止”与“本地实体回收”分开；snip 本地线条父挂壳，回收的引擎部分留作未核验，不能套用 dash 的显式清理结论。

### R10｜dash 的 target 为空时不会持续等待同步

- 位置：[rocky_snip_fx 第 54～55 行](../../../wiki/reference/prefabs/rocky_snip_fx.md#L54-L55)。现文写 target 为空时“PostUpdate 只等动画结束”，画面外仅“不推失真”。
- 依据：[dash_PostUpdate 第 1068～1125 行](../../../raw/current/scripts/prefabs/rocky_boss.lua#L1068-L1125)：`t < 0.4 and target`、屏幕尺寸、线段屏幕相交与 PushLineDistortion 是嵌套条件；未成功走到提前 return 时，都会落到末尾删除 `_fx`／清失真／注销 PostUpdate。本函数没有等待 target 的重试分支，也未监听 target dirty 来重启。
- 建议：说明这些条件不满足时当前本地更新会停止；晚同步或画面外开始时实际是否出现该情况未运行核验。不要承诺持续等待、随后必定恢复。

### R11｜藤桥没有 killed 防重入，不能与绳桥概括成同一协议

- 位置：[rope_bridge_fx 第 93 行](../../../wiki/reference/prefabs/rope_bridge_fx.md#L93) 写“两座桥都用 killed／OnEntitySleep=Remove 保证只走一次”。
- 依据：[绳桥 KillFX](../../../raw/current/scripts/prefabs/rope_bridge.lua#L151-L168) 有 `if not inst.killed` 并赋值；[藤桥 KillFX](../../../raw/current/scripts/prefabs/vine_bridge.lua#L40-L59) 没有该字段或防重入判断，[ShakeIt](../../../raw/current/scripts/prefabs/vine_bridge.lua#L66-L73) 同样无检查。
- 建议：共性只保留睡眠直接删／醒着播退场和睡眠移除回调；`killed` 防重入限定绳桥。重复调用藤桥 KillFX 会重播退场，不可写成被标志拦截。

### R12｜桥面页把基础延时、警告时点与真正拆除时点混写

- 位置：[rope_bridge_fx 第 6 行](../../../wiki/reference/prefabs/rope_bridge_fx.md#L6)、[第 42 行](../../../wiki/reference/prefabs/rope_bridge_fx.md#L42)、[第 53 行](../../../wiki/reference/prefabs/rope_bridge_fx.md#L53)、[第 62 行](../../../wiki/reference/prefabs/rope_bridge_fx.md#L62)、[第 77～78 行](../../../wiki/reference/prefabs/rope_bridge_fx.md#L77-L78)、[第 92 行](../../../wiki/reference/prefabs/rope_bridge_fx.md#L92)；相关 [fx-world-index 第 64 行](../../../wiki/reference/prefabs/fx-world-index.md#L64)。
- 依据一：[QueueCreateRopeBridgeAtPoint](../../../raw/current/scripts/components/ropebridgemanager.lua#L169-L183) 实际延时为 `base_time + random_time*math.random()`，默认 random_time=0.3。示例只传 base_time=0.5，因此是 0.5～0.8 秒，而非固定 0.5 秒；要固定需补 random_time=0。
- 依据二：[玫瑰点配置](../../../raw/current/scripts/componentutil.lua#L663-L678) 对第 i 格设置生成 `.25*i`、警告 `18+.25*i`、真正销毁 `18+.25*i+70*FRAMES`；[QueueDestroyForVineBridgeAtPoint](../../../raw/current/scripts/components/vinebridgemanager.lua#L164-L198) 按 destroytime 排销毁，fxtime 只排警告。因此 18 秒不是还原瓦片／删除桥面的时点；相对该格生成，真正销毁约为 20⅓ 秒。
- 依据三：无自定义 data 时，绳桥和藤桥默认销毁延时均为 `2+(70+math.random(0,10))*FRAMES`，不是固定 2 秒，见 [绳桥](../../../raw/current/scripts/components/ropebridgemanager.lua#L225-L255)、[藤桥](../../../raw/current/scripts/components/vinebridgemanager.lua#L164-L198)。
- 建议：分别标注基础延时＋随机量、警告时点和瓦片销毁时点，同步摘要、表格、示例注释、陷阱与索引，不改源码。无需给未运行的视觉时长下保证。

### R13｜vinebridgemanager 在共同世界工厂挂载，并非只在地表

- 位置：[rope_bridge_fx 第 52 行](../../../wiki/reference/prefabs/rope_bridge_fx.md#L52) 写“藤桥挂地表世界”“只在 world.lua 装配”，由文件名推导了世界范围。
- 依据：[MakeWorld](../../../raw/current/scripts/prefabs/world.lua#L405-L418) 是共同工厂，[主模拟分支](../../../raw/current/scripts/prefabs/world.lua#L568-L586) 无地表条件地添加 `vinebridgemanager`；[cave.lua 返回](../../../raw/current/scripts/prefabs/cave.lua#L581-L586) 也调用 MakeWorld。管理器的 [洞穴瓦片回退](../../../raw/current/scripts/components/vinebridgemanager.lua#L139-L145) 同时表明代码显式考虑 cave。
- 建议：区分“组件共同挂载”与“当前已核验原版藤桥触发场景”；绳桥的洞穴专属装配可保留，不能把藤桥组件的可用范围限定成地表。

### R14｜孢子云的腐败排除条件并非所有未打开的容器

- 位置：[sleepcloud 第 65 行](../../../wiki/reference/prefabs/sleepcloud.md#L65) 把 Limbo 物品概括为“不在背包或打开的容器则跳过”。
- 依据：[TryPerish](../../../raw/current/scripts/prefabs/sporecloud.lua#L286-L300) 只在 Limbo 且 owner=nil，或 owner 同时有 container、未打开、带 `structure`／`portablestorage` 标签时返回；其他带 owner 的 Limbo 物品仍可到 ReducePercent。没有“所有关闭容器都保护”这一通用条件。
- 建议：按源码的 AND／OR 条件描述，保留 ReducePercent(.07) 为当前新鲜度减 7 个百分点；不要从原版常见背包／箱子举例推出任意容器的通则。

### R15｜海树旧 TODO 不足以断言所有砍后树桩读档都不重建 FX

- 位置：[oceantree_ripples_short 第 52 行](../../../wiki/reference/prefabs/oceantree_ripples_short.md#L52)、[第 95 行](../../../wiki/reference/prefabs/oceantree_ripples_short.md#L95) 把 TODO 的“不重建”推到所有砍倒状态；同页第 51 行已承认砍倒只 StopGrowing、行为待核验，口径冲突。
- 依据：[make_stump](../../../raw/current/scripts/prefabs/oceantree.lua#L307-L332) 只 StopGrowing，不移除 growable；[Growable:OnSave](../../../raw/current/scripts/components/growable.lua#L227-L246) 即使无剩余时间仍保存 stage，[OnLoad→SetStage](../../../raw/current/scripts/components/growable.lua#L249-L254) 会调用阶段函数，[SetStage](../../../raw/current/scripts/components/growable.lua#L207-L219) 进入阶段回调后可再次生成涟漪。实体 [SetPersistData](../../../raw/current/scripts/entityscript.lua#L1992-L2012) 先调组件 OnLoad、再调 Prefab OnLoad。烧毁移除 growable、直接构造 stump prefab 又是另外两条分支，不能用 [旧注释](../../../raw/current/scripts/prefabs/oceantree.lua#L73-L85) 抹平。
- 建议：区分普通活树砍成桩后存读、烧毁和直接 stump prefab；有 growable 存档数据时存在阶段回调重建链，具体可见结果保留未运行边界。撤回“源码 TODO 明确证明全部不会重建”的说法，不要求扩大到整个树玩法。

### R16｜海树页扩到 12 注册，叶索引仍写 10

- 位置：[fx-world-index 第 42 行](../../../wiki/reference/prefabs/fx-world-index.md#L42)；对照 [海树页摘要／aliases](../../../wiki/reference/prefabs/oceantree_ripples_short.md#L7-L8) 和 [增补记录](../../../wiki/reference/prefabs/oceantree_ripples_short.md#L109)。
- 现页已负责 `watertree_pillar_ripples`／`watertree_pillar_roots`，索引仍只列原来的三档海树／倒伏／柱体三件套，不含大树干两层。
- 建议：原叶级行改为 12，并简记大树干涟漪／根影。保持唯一登记，不另建重复页；结构导航通过不代表索引文字计数正确。

## 已核验且不列为问题的边界

- `pocketwatch_warp_marker`：两种 marker 接口确实不同；positionalwarp 删除时 onremove 可补建同槽的风险已诚实保留；exit_fx 由玩家 SG 的三个到达路径生成，而非 portal 自己生成；同文件依赖表和实际生成已分开。
- `scorchedground`：anim 字段存档、暗影槌联网层与五个本地帧层、姐妹骨灰罐 AddChild 清理、孢子炸弹 AddDebuff→Stop→OnDetach 生成孢子云的链条成立。没有把 sporebomb 直接伤害错误归给其壳。
- `fx_book_mount`：6 效果键×步行／骑乘=12，动态字段与唯一 SG 入口成立；先生成再结算、重复读书跳帧、animover 与 onexit 双清理已核验。主题名不冒充注册名。
- `goosplash`：9 注册、两个本地动画层、flip 的不对称映射、壳与本地层各自清理成立。全 scripts 仍只命中自身、prefablist 与 peghook 依赖，事件包未包含的范围已说明，不判死代码。
- `nightmarelightfx`：四个光晕为 DECOR 而非 FX 标签，grottowar 的 mouseparent 不匹配、头层非联网非专服生成、秃鹫影子的两套调度与 KillShadow 分支成立；没有因无 Lua Remove 就断言必然泄漏。
- 其他混合项：舞台影手正常伤害归 owner combat；光束武器 5 位面与原版施法者 35 位面合并为结算前 40；暗影陷阱的目标变噩梦与减速互斥；岩石大师剪击／冲刺的伤害留宿主 SG。上述主要归属未发现 P1/P2，仍受各页未运行范围限制。

## 本轮实际验证

| 检查 | 范围与结果 |
|---|---|
| 页面／导航 | 14／14 通过；全局导航 1143 知识页、34 索引、0 问题。相关索引文字仍有 R12／R16，机械检查不校验数字语义 |
| 严格源码链接 | 1070 处引用（1068 行引用＋2 文件引用）、844 唯一目标、138 源文件，0 error／0 warning，未截断、exit 0 |
| 哈希 check | 14 页、198 来源对全部 unchanged；changed／sources_changed／unrecorded／invalid 均 0。未 record；不包含 R02 的未声明依赖 |
| 注册候选反查 | 4095 个 Lua 文件完整扫描，81 名匹配到 448 个候选名／行对；UTF-8-sig 解码，easing.lua 与 nis/xbnyn.lua 两文件以 Latin-1 回退，无跳过。最长名优先避免前缀重叠计数，候选数不是生成点数；动态 books／海树／腐化／head-data 链另读实现与消费者 |
| 名称／归属 | 81 名未见非归档 Prefab aliases 跨页重复归属；简中精确 NAMES 键命中 sporecloud／rocky_boss_shadow，与页面名相符；其他为效果描述，宿主名证据按页面定向核对 |
| Lua 语法 | 本批 16 段完整 Lua 分别提取，经已有 parser_probe --syntax-only 全部 exit 0；临时 .review-r13-*.lua 由 apply_patch 创建／删除，无残留。只证语法，不替代 R07 等接入前提检查 |
| 章节读取 | 14 页各取一章，传标题原文（含中文标点），max-bytes=65536；均 results=1、complete_selected_ranges=true、truncated=false、exit 0；直接读取当前 Markdown，未重建或依赖搜索缓存 |
| 输入／报告一致性 | 14 页＋147 份声明来源共 161 个文件前后 SHA-256 一致；报告 91 个本地链接目标及行区间有效；16 个语法临时文件全部移除 |
| 运行与范围 | 未运行游戏，未验证引擎属性复制、父子级联、资源包内容／时长、多人晚加入与观感；没有把来源文件数量当作全文语义审阅数量 |

章节试读依次为：生成点与触发；给自定义海面宿主挂同档涟漪与根影；生成点、父挂与清理；玩法归属；生成点与宿主；生成点与清理；同文件的 11 个 shadow 单位注册（只辨析）；生成可存读的地面焦痕；注册与构造；共同工厂与端分界；生成点、端与父挂；生成点、端与父挂；生成点与宿主调用；生成点与宿主链。

## 处理与后续

修复范围为上述 9 页与相关 `fx-world-index` 两行。先纠正 R01 的包装方法调用链、R07 的必需组件、R09／R10 的本地更新与清理，再处理边界和同步文字；R02 的 sources 及 R01 新证据合并刷新本页基线，其余不因文字修改机械 record。摘要、案例、陷阱与索引涉及旧说法时同步，不改历史日志。

固定新增页队列累计已独立检视 **112／158 页，余 46 页**；已检视包含待修页，不等于全部验收。本轮需修复后复验；前轮待复验记录保持各自范围，不自动关闭，也不在本轮另加未读页面充数。后续按剩余 U2 起各批继续，每轮约 12～15 页，最终再统一检查仍未关闭的报告项。

## 修复复核（2026-11-07）

R01～R16 全部采纳并修复。abigail_retaliation 撤回“gestalt ×3 为死代码”，改为实体包装方法六参透传、带 `gestalt` 标签的施加者令 2 秒变 6 秒；补 `components/incinerator.lua`、`standardcomponents.lua`、`components/debuffable.lua`、`components/debuff.lua` 四份来源并 record 本页。staff_lunarplant_fx 把 `colouradder` 结论收窄到 Lua 写入端，滚雪球改述为宿主雪球事件。shadow_trap 分开实体 `noattack` 标签与 SG 状态标签、`death` 与 `onremove` 动作。shadow_despawn 给光束施法者补 `health`／`combat`／tagset／动画符号前提。oceanhorror_ripples 撤回未经核验的父子级联保证。rocky_snip_fx 分开失真／更新停止与本地实体回收，dash 的 target 为空不再承诺持续等待。rope_bridge_fx 把 `killed` 防重入限定绳桥，拆分 0.5～0.8 秒生成、约 18 秒警告、约 20⅓ 秒销毁与默认约 4.3～4.7 秒销毁，并把藤桥管理器改述为共同世界工厂挂载。sleepcloud 按源码 AND／OR 条件重写 Limbo 腐败排除。oceantree_ripples_short 区分活树砍桩／烧毁／stump prefab 三条读档分支，沿用 `growable` 存档链保留未运行边界；`fx-world-index` 两行同步 12 注册与大桥时序。

验证：9 页＋`fx-world-index` 共 10/10 页面通过，全库导航 1,145 知识页／34 索引／0 问题；strict 713 处引用（711 行引用＋2 文件引用）、567 个唯一目标、95 个源文件，0 error／0 warning；9 页＋索引哈希 check 139 组 unchanged，abigail_retaliation record 1 页（新增 4 个来源）。未运行游戏、未改 raw、未提交、未推送。累计已独立检视仍 112／158 页，余 46 页；R01～R16 均已处置，仍待独立复验。

## 2026-11-07 修后独立复验：本批收口

本节为本对话实际复验，上节为修复方自报。实读 BUILD 仍为 26335394、HEAD 未变；复读修订行及其必要源码，检查原 14 页与相关叶索引。**R01～R16 全部通过，未发现本次修订引入的新增 P1/P2。** 不把源码审阅升级为游戏运行验证。

| 项目 | 当前页面位置 | 独立复验结果 |
|---|---|---|
| R01 | [abigail_retaliation:31](../../../wiki/reference/prefabs/abigail_retaliation.md#L31)、87 | 通过：六参实体包装→四参组件→AttachTo／Extend→buff_OnExtended 的 buffer 透传完整；gestalt 标签可达，基础 2 秒乘 3 |
| R02 | [abigail_retaliation:11](../../../wiki/reference/prefabs/abigail_retaliation.md#L11) | 通过：补齐原缺三来源及新读 debuff.lua；正文引用均已声明，基线 check 含新增四来源 |
| R03 | [staff_lunarplant_fx:27](../../../wiki/reference/prefabs/staff_lunarplant_fx.md#L27)、66、148 | 通过：只陈述主模拟 Lua 写色、无自定义颜色 netvar；引擎复制与客户端外观留作未核验 |
| R04 | [staff_lunarplant_fx:55](../../../wiki/reference/prefabs/staff_lunarplant_fx.md#L55)、77 | 通过：明确监听的是雪球宿主的 onremove／enterlimbo，解绑取宿主 _rollingfx |
| R05 | [shadow_trap:102](../../../wiki/reference/prefabs/shadow_trap.md#L102) | 通过：实体排除标签与 SG 状态检查分开；无 SG 时不跳过零伤害 attacked |
| R06 | [shadow_trap:43](../../../wiki/reference/prefabs/shadow_trap.md#L43) | 通过：death 仅 KillFX，onremove 才解父并保世界坐标 |
| R07 | [shadow_despawn:125](../../../wiki/reference/prefabs/shadow_despawn.md#L125)、130、149 | 通过：父节、示例注释和限制均补有效存活、health／combat／tagset／符号前提，符合 InitBlast 与 AOEUtil 的实际消费 |
| R08 | [oceanhorror_ripples:60](../../../wiki/reference/prefabs/oceanhorror_ripples.md#L60)、70 | 通过：撤回自动级联保证，说明引擎部分未核验，复用方自管引用并显式撤销 |
| R09 | [rocky_snip_fx:64](../../../wiki/reference/prefabs/rocky_snip_fx.md#L64) | 通过：snip 的失真停止／PostUpdate 注销不再等同删除本地线条，区别于 dash 显式 Remove |
| R10 | [rocky_snip_fx:55](../../../wiki/reference/prefabs/rocky_snip_fx.md#L55) | 通过：target 缺失或未成功推失真会落到末尾停更，没有自动等待／dirty 重启保证 |
| R11 | [rope_bridge_fx:93](../../../wiki/reference/prefabs/rope_bridge_fx.md#L93) | 通过：killed 防重入只归绳桥；藤桥可重播退场或被 ShakeIt 覆盖 |
| R12 | [rope_bridge_fx:42](../../../wiki/reference/prefabs/rope_bridge_fx.md#L42)、53、62、77、83、92；索引 64 | 通过：示例基础延时含默认 0.3 随机量；玫瑰点警告／销毁相差 70 帧；无 data 的默认拆除为 2＋70～80 帧；摘要与索引同步 |
| R13 | [rope_bridge_fx:52](../../../wiki/reference/prefabs/rope_bridge_fx.md#L52) | 通过：共同 MakeWorld 主模拟装配 vinebridgemanager，洞穴也调用该工厂；与已核验地表触发场景分开 |
| R14 | [sleepcloud:65](../../../wiki/reference/prefabs/sleepcloud.md#L65) | 通过：Limbo 分支 owner=nil 或 container＋关闭＋指定标签的 AND／OR 条件与实现一致 |
| R15 | [oceantree_ripples_short:52](../../../wiki/reference/prefabs/oceantree_ripples_short.md#L52)、95 | 通过：砍桩 StopGrowing 与移除 growable 分开；保留 stage 存读／阶段回调重建链，不再把旧 TODO 当全部分支证明 |
| R16 | [fx-world-index:42](../../../wiki/reference/prefabs/fx-world-index.md#L42) | 通过：12 注册含大树干两层，与正文／aliases 一致，未新增重复登记 |

### 本次实跑校验与限制

- 格式：原 14 页＋`fx-world-index` 共 **15／15 页面本身通过**，没有本批页面 issue。全局导航第一次检查与稍后复跑均为 1146 知识页／34 索引，唯一问题是并行新增 `reference/prefabs/queen-torch.md` 尚无叶级表格登记（got 0）；因此两次 `--check-indexes` 整体 exit 1，**不能称全局导航通过**。这是本批外在建装备页，保留其修改、不代为登记，不据此重开本批已修的 16 项。
- 严格源码引用：原 14 页 **1083 处引用**（1081 行引用＋2 文件引用）、854 唯一目标、138 源文件，0 error／0 warning，未截断、exit 0。
- 来源：原 14 页 **202 组比较全部 unchanged**，changed／sources_changed／unrecorded／invalid 均 0；14 页正文全部本地源码引用均已进入各自 sources。本次只 check、未 record。
- 查询：9 个修订知识页各用标题原文读取一章，包括中文标点、英文引号与加号；`max-bytes=65536` 下均返回 1 项、complete_selected_ranges=true、truncated=false、exit 0。直接读取当前 Markdown，未使用搜索缓存作验收。
- 示例：对比 HEAD，16 段 Lua 中仅光束与桥面两段的注释改变，代码逻辑未改；这两段分别提取并重新跑 parser_probe --syntax-only，exit 0。其余 14 段沿用本轮初检的语法结果，不冒称重跑；临时文件已通过 apply_patch 删除。
- 输入：14 原页＋叶索引＋148 份不同声明来源，共 163 个文件前后 SHA-256 一致。没有把其余来源全部全文重审；其他 Agent 的新装备页不在本批独立语义验收范围内。
- 日志：旧卷 68,331 字节超限，整卷归档为 `wiki/log-archive/log-2026-11-07-001.md`。apply_patch 建立归档副本后，用本机既有 unix2dos 仅恢复原 CRLF；归档与原日志逐字节一致，SHA-256 均为 `869f0fa36ed28a3d0202018bc5010313e6141f4fc52ad052c00fb6505bc17ba3`，核对原日志未被并行追加后才换新卷。报告、日志换卷之外无持久文本修改；未运行游戏、未提交推送。

本批已收口，累计仍 **112／158 页，余 46 页**；其他轮次尚未独立复验的历史项保留，后续可继续剩余 U 批。全局导航的并行在建页登记由其维护方收尾。
