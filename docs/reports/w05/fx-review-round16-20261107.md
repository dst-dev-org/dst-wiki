# FX 第 16 轮独立检视：外围 U11～U14

核验日期：2026-11-07；实际源码快照：steam-buildid:26335394；知识仓 HEAD：97afb4e9e6ca05390558769926f1f9c8598d0f70。

**当前结论：R01～R30 及新增 R31 均通过修后内容复验，第 16 轮内容可收口；统一机械检查留待最后执行，尚未由本次复验确认。详见文末“剩余项再次内容复验”。** 初检 P1×1、P2×27、P3×2 及历次复验记录保留为历史。本次按用户要求只看文件内容，不运行校验或测试脚本；只维护本报告和当前日志，不改知识页、索引、基线、PLAN、台账、Skill、工具或 raw，不 record、不提交推送，未运行游戏。

## 范围与逐页结论

12 页／37 个主要覆盖注册名，1,463 个物理行／216,971 字节；声明 123 组 sources／96 个不同文件。核验必要调用链，不表示将 96 份来源的其他玩法全文重审。

| 页面（wiki/reference/prefabs） | 注册 | 行／字节 | 独立检视结论 |
|---|---:|---:|---|
| [archive_ambient_sfx](../../../wiki/reference/prefabs/archive_ambient_sfx.md) | 5 | 115／19,985 | P2：R02、R03 |
| [atrium_gate_activatedfx](../../../wiki/reference/prefabs/atrium_gate_activatedfx.md) | 4 | 122／17,500 | P2：R03～R05 |
| [balloon_held_child](../../../wiki/reference/prefabs/balloon_held_child.md) | 4 | 112／18,684 | P2：R06 |
| [wintersfeastbuff_fx](../../../wiki/reference/prefabs/wintersfeastbuff_fx.md) | 2 | 108／17,107 | P2：R07～R09；P3：R29 |
| [witchbroom_fx](../../../wiki/reference/prefabs/witchbroom_fx.md) | 3 | 134／21,175 | P2：R03、R10～R13 |
| [beargerfur_sack_frost_fx](../../../wiki/reference/prefabs/beargerfur_sack_frost_fx.md) | 2 | 124／18,511 | P1：R01；P2：R03、R14、R15 |
| [bat_bosshat_fx](../../../wiki/reference/prefabs/bat_bosshat_fx.md) | 5 | 143／24,182 | P2：R03、R16～R20 |
| [bishop_targeting_fx](../../../wiki/reference/prefabs/bishop_targeting_fx.md) | 1 | 111／14,218 | P2：R03、R21、R22 |
| [itemmimic_revealed_shadow](../../../wiki/reference/prefabs/itemmimic_revealed_shadow.md) | 1 | 131／14,420 | P2：R03、R23；P3：R30 |
| [positronbeam_back](../../../wiki/reference/prefabs/positronbeam_back.md) | 3 | 125／17,280 | P2：R24、R25 |
| [stalker_shield1](../../../wiki/reference/prefabs/stalker_shield1.md) | 5 | 114／16,673 | P2：R26～R28 |
| [toadstool_cap_absorbfx](../../../wiki/reference/prefabs/toadstool_cap_absorbfx.md) | 2 | 124／17,236 | P2：R02；其余本轮未发现 P1/P2 |

## P1：示例收尾失效

### R01｜伯尼示例绑定了全局 EndFire，而不是后面的局部函数

- 位置：[beargerfur_sack_frost_fx:80](../../../wiki/reference/prefabs/beargerfur_sack_frost_fx.md#L80)～96，尤其第 89、92 行。
- OnLight 定义时，后置的 local function EndFire 尚未进入词法作用域；第 89 行取的是全局 EndFire。在通常没有该全局的环境下传入 nil，预期的 20 秒撤组件／退火回调不会执行；不能用“语法检查通过”证明收尾可用。
- [DoTaskInTime](../../../raw/current/scripts/entityscript.lua#L1547-L1554) 将 fn 交给调度器，[调度执行](../../../raw/current/scripts/scheduler.lua#L178-L190) 仅在 fn 存在时调用。原版先定义 [endthornsfire](../../../raw/current/scripts/prefabs/bernie_big.lua#L61-L76)，再由 [OnLighterLight](../../../raw/current/scripts/prefabs/bernie_big.lua#L262-L265) 的闭包调用。
- 建议：把 EndFire 定义前移或显式前向声明；同步清掉多传的 inst（实体任务会自动传自身），保留复燃取消旧任务的行为。这里只修 Wiki 示例，不改原版源码。

## P2：事实、条件与证据边界

### R02｜两页正文使用了四组未声明来源

| 页面位置 | 正文已有引用、sources 缺失 |
|---|---|
| [archive_ambient_sfx:11](../../../wiki/reference/prefabs/archive_ambient_sfx.md#L11) | prefabs/archive_chandelier.lua、tuning.lua |
| [toadstool_cap_absorbfx:11](../../../wiki/reference/prefabs/toadstool_cap_absorbfx.md#L11) | map/terrain.lua、map/static_layouts/toadstool_arena.lua（正文第 70 行） |

声明来源的 123 组哈希 unchanged 不包含这些漏项。建议实际核验后补声明并按既有规则处理本页基线；或同步删除相关证据和断言，不为消除提示批量删来源。其他 10 页未发现这种“正文 Markdown 源码链接未进 sources”的漏项。

### R03｜七页仍有父挂与删除通道的错误归因

共同依据：[AddChild](../../../raw/current/scripts/entityscript.lua#L921-L935) 登记 Lua children，[Remove](../../../raw/current/scripts/entityscript.lua#L1779-L1796) 遍历并删除该表。单独 entity:SetParent 的原生删除行为不能据此证明。

| 页面位置 | 问题与应保留的边界 |
|---|---|
| [archive_ambient_sfx:70](../../../wiki/reference/prefabs/archive_ambient_sfx.md#L70) | 警戒光球环绕音实际由 [SpawnChild](../../../raw/current/scripts/prefabs/archive_props.lua#L624) 生成；应归因于 Lua 子项删除，而非“引擎父挂级联”。其 parent 为空自删守卫可另列。 |
| [atrium_gate_activatedfx:72](../../../wiki/reference/prefabs/atrium_gate_activatedfx.md#L72) | “父级大门移除时随引擎级联”虽附未运行说明，主句仍当确定清理路径；[ShowFx](../../../raw/current/scripts/prefabs/atrium_gate.lua#L160-L168) 仅 SetParent。保留 KillFX 的动画／4 秒兜底，直接删除宿主的原生结果标未核验。 |
| [witchbroom_fx:55](../../../wiki/reference/prefabs/witchbroom_fx.md#L55)、75～77 | 沃比本地槽层、扫帚网络壳和教练 FX 不能一概从 SetParent 推导“随宿主级联”。保留各自正常卸装／禁用／OnDetached 的显式删除；本地槽层 [SetParent](../../../raw/current/scripts/prefabs/woby_rack.lua#L219-L229) 没有同时 AddChild。轨迹池另见 R10。 |
| [beargerfur_sack_frost_fx:55](../../../wiki/reference/prefabs/beargerfur_sack_frost_fx.md#L55)、69 | 伯尼死亡移除本体成立，但火层 [只做 SetParent](../../../raw/current/scripts/prefabs/bernie_big.lua#L239-L241)，不能由此保证必被带走且不播 pst。桶霜气的显式 OnRemove 清理可保留，不混为同一证据。 |
| [bat_bosshat_fx:85](../../../wiki/reference/prefabs/bat_bosshat_fx.md#L85) | 帽 FX [父挂的是佩戴玩家](../../../raw/current/scripts/prefabs/bat_boss.lua#L815-L825)，不是 bat_boss 本体；“本体移除走引擎级联”连父对象都不对。原版 [OnUnequip](../../../raw/current/scripts/prefabs/bat_boss.lua#L264-L266) 显式删帽层，再由 FX 回调删三只本地蝠。 |
| [bishop_targeting_fx:68](../../../wiki/reference/prefabs/bishop_targeting_fx.md#L68) | 同一行同时说“引擎级联”和“FX 非宿主子体”。[主教 OnRemoveEntity](../../../raw/current/scripts/prefabs/bishop.lua#L183-L188) 清 targetingfx 是显式引用通道，删掉级联归因。 |
| [itemmimic_revealed_shadow:52](../../../wiki/reference/prefabs/itemmimic_revealed_shadow.md#L52) | 宿主确实 [AddChild](../../../raw/current/scripts/prefabs/itemmimic_revealed.lua#L121-L124)，清理成立，但应写 Lua children 删除，避免把正确结论归给未核验引擎机制。 |

本项不反向断言这些原版实体必定泄漏，不要求为补知识缺口修改游戏源码。

### R04｜大门首次 SetFX(idle) 并不会“补标签”

- 位置：[atrium_gate_activatedfx:37](../../../wiki/reference/prefabs/atrium_gate_activatedfx.md#L37)。
- [SetFX](../../../raw/current/scripts/prefabs/atrium_gate_activatedfx.lua#L6-L29) 的动画和标签切换都在“旧态与新态不同”的分支内。首次 idle 不进该分支，只写 inst.anim；FX 标签来自工厂第 76 行。
- 建议：写“沿用工厂 idle 动画队列和 FX 标签，仅记录状态”；同步相关案例解释。

### R05｜大门没有预装 highlight，不等于登记表只是预留接口

- 位置：[atrium_gate_activatedfx:62](../../../wiki/reference/prefabs/atrium_gate_activatedfx.md#L62)、验证记录第 111 行。
- 页面据工厂未安装 highlight，推断 highlightchildren 没有实际消费者。玩家控制器会在 [选中有效实体后按需 AddComponent(highlight)](../../../raw/current/scripts/components/playercontroller.lua#L2850-L2869)，随后 [ApplyColour](../../../raw/current/scripts/components/highlight.lua#L81-L92) 遍历 highlightchildren；不能只检查工厂静态装配。
- 建议：撤回“预留接口”结论，写成按需安装后的高亮传播通道；不夸大为所有状态下一定可见，新增采用的来源须同步声明与基线。

### R06｜彩带 buff 没有 Extend 续时回调

- 位置：[balloon_held_child:58](../../../wiki/reference/prefabs/balloon_held_child.md#L58)。
- 同名 AddDebuff 确实进 Extend，但 [Debuff:Extend](../../../raw/current/scripts/components/debuff.lua#L53-L57) 仅调用已设置的 onextendedfn。[balloonparty_buff 工厂](../../../raw/current/scripts/prefabs/balloonparty.lua#L225-L252) 没设置它，也没有倒计时可刷新。
- 持续生效来自 [buff_OnTick](../../../raw/current/scripts/prefabs/balloonparty.lua#L193-L222) 自行检查附近云，失败才 Stop。建议保留 AddDebuff 的复用事实，删除“Extend 刷新所以持续续期”的因果；第一次附着还会立即执行一次 OnTick。

### R07｜冬宴密度方向写反

- 位置：[wintersfeastbuff_fx:88](../../../wiki/reference/prefabs/wintersfeastbuff_fx.md#L88)。
- [CalcIntensity／OnFxTick](../../../raw/current/scripts/prefabs/wintersfeastbuff.lua#L25-L35)：剩余时长越多，强度越高，间隔越短；临结束趋近 1.8 秒，满强度是 0.25 秒。当前“剩余越少越密”与同句数值及第 53 行都相反。
- 建议：改为“剩余越少越疏”，同步陷阱结论。

### R08｜冬宴原版每次新投放，不是逐 tick 重定位旧粒子

- 位置：[wintersfeastbuff_fx:71](../../../wiki/reference/prefabs/wintersfeastbuff_fx.md#L71)。
- [OnFxTick](../../../raw/current/scripts/prefabs/wintersfeastbuff.lua#L29-L35) 每次 SpawnPrefab 一个新实体，只设置新粒子的位置。页面第 86 行已正确说明旧粒子留原地。
- 建议：撤回“每次 tick 重新 SetPosition，原版即此做法”；要介绍重定位旧实体须明确那是自定义方案。

### R09｜烤炉火层的“唯一消失路径”说得过满

- 位置：[wintersfeastbuff_fx:63](../../../wiki/reference/prefabs/wintersfeastbuff_fx.md#L63)。
- [MakeFireFx](../../../raw/current/scripts/prefabs/wintersfeastoven.lua#L65-L77) 也会先调用 RemoveFireFx，而非只有 OnScienceWasMade 的定时回调；[StartMakingScience](../../../raw/current/scripts/prefabs/wintersfeastoven.lua#L122-L131) 会进入该替换入口。
- [锤毁宿主](../../../raw/current/scripts/prefabs/wintersfeastoven.lua#L252-L262) 又会取消挂在宿主上的任务，不能据正常完成回调覆盖所有删除情况。建议收窄为“正常烹饪完成的收尾”，分开重建替换及宿主直接移除的未知，不据 SetParent 保证其结果。

### R10｜扫帚“已在池内”与“仍在播放”的轨迹不能共用清理结论

- 位置：[witchbroom_fx:45](../../../wiki/reference/prefabs/witchbroom_fx.md#L45)、75、84。
- [trail_OnAnimOver](../../../raw/current/scripts/prefabs/witchbroom.lua#L214-L221) 仅在动画结束时检查 owner：在播轨迹可在该次结束后发现 owner 无效并 Remove；已经 RemoveFromScene 回池的轨迹不能据此保证再有一次 animover。
- [fxfn](../../../raw/current/scripts/prefabs/witchbroom.lua#L281-L303) 没写遍历池的 OnRemoveEntity。当前“载体没了池即失效”“卸载无需逐个清理”把两种状态混为一谈；trail_pool 也是数组，不是单槽。
- 建议：记录可证的正常回池和在播无主自删，池中项随载体移除的最终结果保留未核验。若给模组复用骨架，可显式设计清池，但不声称原版已这样做或必然泄漏。

### R11｜扫帚示例省掉了复用协议的必要步骤

- 位置：[witchbroom_fx:87](../../../wiki/reference/prefabs/witchbroom_fx.md#L87)～108。
- 原版 [GetPooledTrailFx](../../../raw/current/scripts/prefabs/witchbroom.lua#L249-L259) 复用分支会 ReturnToScene＋RestartFx，新建分支会设置 fx.owner。示例只 table.remove 后改位置；按原版回池协议，复用实体仍隐藏且动画未重启，新建又没有 owner 不能按所述归池。
- CreateTrailFx 是原文件 local，片段本身也没有定义／导入它。建议补齐辅助函数与协议，或明确标“实现参照”并列出依赖，撤回“语法完整骨架”所暗示的可直接接入；补主模拟 persists=false。

### R12｜沃比层数与褪色任务的承载实体写错

- 位置：[witchbroom_fx:54](../../../wiki/reference/prefabs/witchbroom_fx.md#L54)、76。
- 每个网络 rack FX 都 [建 3 个本地槽层](../../../raw/current/scripts/prefabs/woby_rack.lua#L219-L229)，宿主 [建 2 个网络实例](../../../raw/current/scripts/prefabs/wobybig.lua#L234-L238)，故每个非专服进程是 2 个网络层＋6 个本地槽层，不是“三实例”。
- [OnPostUpdateFading／SetFadeColour](../../../raw/current/scripts/prefabs/woby_rack.lua#L45-L60) 由网络 rack 实体持有并更新自身及 slots，不是各本地子体分别执行整套宿主轮询。
- 建议统一图层数量和回调承载者，原有三个 slot 的 netvar 表可保留。

### R13｜骑乘不能直接等同于没有 moving 标签

- 位置：[witchbroom_fx:114](../../../wiki/reference/prefabs/witchbroom_fx.md#L114)。
- [fx_DoTrail](../../../raw/current/scripts/prefabs/witchbroom.lua#L262-L278) 只看 owner 的 moving，没有 IsRiding 排除。[SGwilson 的 run_start](../../../raw/current/scripts/stategraphs/SGwilson.lua#L11296-L11331) 和 [run](../../../raw/current/scripts/stategraphs/SGwilson.lua#L11426-L11443) 包含骑乘分支且有 moving 标签。
- 建议保留“真实位移不是判据”，移除“骑乘坐骑不会投放”的泛化；不把“原地播放动画”直接等同 SG moving 状态。

### R14｜交换同一回调内的退火／撤组件顺序，不会凭空多出 20 秒

- 位置：[beargerfur_sack_frost_fx:76](../../../wiki/reference/prefabs/beargerfur_sack_frost_fx.md#L76)。
- 原版 [endthornsfire](../../../raw/current/scripts/prefabs/bernie_big.lua#L61-L76) 在同一个到期回调里调用退场并撤玩法组件；[EndBernieFire](../../../raw/current/scripts/prefabs/bernie_big.lua#L726-L729) 只切 pst，不等待 20 秒。
- 建议说明“20 秒玩法窗口结束后可能仍有 pst 视觉尾巴”，删除仅因同步语句交换就导致“20 秒窗口外残留”的因果。

### R15｜伯尼读档后的 canlight 还受重新认主与技能门控

- 位置：[beargerfur_sack_frost_fx:107](../../../wiki/reference/prefabs/beargerfur_sack_frost_fx.md#L107)。
- 燃烧任务／引用未序列化可按实际范围记；但 [onLeaderChanged](../../../raw/current/scripts/prefabs/bernie_big.lua#L344-L347) 会按 leader 技能重新加 canlight 并装监听，不能把构造瞬间没有标签写成读档后的最终状态。
- 建议补恢复链门禁，撤回“回构造初值后重新点燃即可”的无条件结论；读档完整时序未核验则明确保留未知。

### R16｜惊树链不是仅 stage 4 老树

- 位置：[bat_bosshat_fx:72](../../../wiki/reference/prefabs/bat_bosshat_fx.md#L72)、125。
- [Spooked:Spook](../../../raw/current/scripts/components/spooked.lua#L79-L91) 的通过条件是 stage==4 **或**仍可 CHOP 的 workable；之后还要动画映射和 ShouldSpook 通过。当前把或关系写成 stage4 必要条件。
- 建议恢复实际门禁，保留万圣活动与惊吓阈值，不把 FX_MAP 的多阶段映射写成永远不可达。

### R17｜net_tinybyte 的值域是 0～7

- 位置：[bat_bosshat_fx:116](../../../wiki/reference/prefabs/bat_bosshat_fx.md#L116)。
- [netvars.lua](../../../raw/current/scripts/netvars.lua#L6-L15) 明载 tinybyte 为 3 位 0～7；0～255 对应 net_byte，[选型 helper](../../../raw/current/scripts/netvars.lua#L103-L110) 也一致。
- 建议改正位宽和选型说明，新增采用的来源同步 sources／基线；原版 1～4 的四变体没有越界。

### R18｜统一传一个 variation，不会让各端看到不同变体

- 位置：[bat_bosshat_fx:94](../../../wiki/reference/prefabs/bat_bosshat_fx.md#L94)。
- [主模拟设 variation](../../../raw/current/scripts/prefabs/batbat.lua#L150-L163)，各非专服读取 [相同 netvar 选动画](../../../raw/current/scripts/prefabs/batbat.lua#L122-L146)。本地各自建实体不等于各自随机选号。
- 建议改成“同一变体号、各端本地渲染”；确要不同选号须另设本地逻辑，不能归为原版行为。

### R19｜蝙蝠帽客户端只在一次 PostUpdate 尝试补建，不持续轮询

- 位置：[bat_bosshat_fx:31](../../../wiki/reference/prefabs/bat_bosshat_fx.md#L31)、48。
- [fx_PostUpdate](../../../raw/current/scripts/prefabs/bat_boss.lua#L803-L812) 首行就 RemovePostUpdateFn，之后才取 parent；parent 为空也不重试。工厂注释“等待 build 同步”不代表循环等待直到成功。
- 建议写“延后一轮 PostUpdate 单次补建”，删除持续轮询的保证；不据此断言原版实际丢层或网络同步必然失败。

### R20｜蝙蝠本地播放示例没有复制代理位置，也没配置 build

- 位置：[bat_bosshat_fx:102](../../../wiki/reference/prefabs/bat_bosshat_fx.md#L102)～110。
- 原版 [PlayBatFX](../../../raw/current/scripts/prefabs/batbat.lua#L124-L146) 明确 SetPosition(proxy 世界位置)、SetBank、SetBuild 后播放；示例只 SetBank、PlayAnimation，新建实体会留默认落点，动画资源依赖亦未交代。
- 建议补位置与实际 build／资产前提；若仅示意 netvar 解码，应标局部片段而非可复刻的完整命中特效。

### R21｜主教的 1 秒是蓄力状态时长，不是预警圈可见时长

- 位置：[bishop_targeting_fx:21](../../../wiki/reference/prefabs/bishop_targeting_fx.md#L21)。
- [attack onenter](../../../raw/current/scripts/stategraphs/SGbishop.lua#L290-L317) 设 1 秒超时，圈到 [第 6 帧](../../../raw/current/scripts/stategraphs/SGbishop.lua#L364-L370) 才标记生成，再由 onupdate 消费；超时已开始 KillFx。
- 建议区分状态计时与约 0.8 秒的生成后预警窗口，不承诺玩家从看到圈起有完整 1 秒；实际帧调度／网络延迟未运行核验。

### R22｜主教旧弹体的“除注册外无引用”与同文件源码相反

- 位置：[bishop_targeting_fx:99](../../../wiki/reference/prefabs/bishop_targeting_fx.md#L99)。
- deprecated 标记成立，但 [旧 OnHit](../../../raw/current/scripts/prefabs/bishop_charge.lua#L19-L21) 仍 SpawnPrefab(bishop_charge_hit)，并由 [projectile 的 onhit 回调](../../../raw/current/scripts/prefabs/bishop_charge.lua#L60-L65) 注册；bishop_charge 另有 prefablist 文件条目。
- 全域反查保留了这些命中。建议分清“旧弹体本身未找到现役生成入口”“旧链内部仍生成命中特效”“文件清单非生成点”，不得把 deprecated 推成代码完全无引用，也不把旧链当已证实在现役触发。

### R23｜拟态虫粒子包络没有出生淡入段

- 位置：[itemmimic_revealed_shadow:21](../../../wiki/reference/prefabs/itemmimic_revealed_shadow.md#L21)、37。
- [颜色包络](../../../raw/current/scripts/prefabs/itemmimic_revealed.lua#L204-L220) alpha 从 1 开始，经 0.9 到 0.5，尺寸从 1.5 缩到 0；没有从透明到不透明的淡入段。
- 建议改为“出生即满 alpha，逐渐变淡并缩小”；别把剩余 alpha 0.5 单独写成完全透明。

### R24｜SetLevel(1) 也不能触达 lunar_1_loop

- 位置：[positronbeam_back:107](../../../wiki/reference/prefabs/positronbeam_back.md#L107)。
- [SetLevel](../../../raw/current/scripts/prefabs/positronbeam.lua#L60-L75) 对 nil 或 <2 直接 Hide，不拼动画名。当前“level1 档存在、只能靠模组触达 lunar_1_loop”混淆接口与资源假设。
- 建议写“没有 level1 播放分支”；即使动画资源存在，额外直接播动画／改方法也是自定义行为，不能说 SetLevel 能触达它。

### R25｜月亮石示例丢掉所有句柄，没有可用收尾入口

- 位置：[positronbeam_back:88](../../../wiki/reference/prefabs/positronbeam_back.md#L88)～98。
- 三实体只有局部变量，函数不返回、不挂宿主；注释只展示 pulse 的 FinishFX／Remove，front／back 两个循环层从未 KillFX。它们 [无固有寿命上限](../../../raw/current/scripts/prefabs/positronbeam.lua#L24-L54)，也没父子登记。
- 建议返回整组句柄或挂宿主，并给出 beam:KillFX＋pulse:FinishFX／Remove 的成功／中断收尾；若只示意生成，明确片段依赖外部管理并保留句柄出口，不能让读者照抄后无法清理。

### R26｜SetPristine 前随机不足以证明骨盾外观不同步

- 位置：[stalker_shield1:97](../../../wiki/reference/prefabs/stalker_shield1.md#L97)。
- [基名工厂](../../../raw/current/scripts/prefabs/stalker_shield.lua#L91-L103) 随机后还 SetPrefabName 为定号变体；[EntityScript:SetPrefabName](../../../raw/current/scripts/entityscript.lua#L840-L843) 会写原生实体名，[SpawnPrefabFromSim](../../../raw/current/scripts/mainfunctions.lua#L388-L399) 又使用工厂最终 prefab。页面没有核验网络构造名和 AnimState 复制，就断言“逐机执行、无网络同步”。
- 建议仅保留源码确定的随机选型／改名以及现役入口只用定号名；最终跨端效果留未知，不据无显式 netvar 推出不同步。不要反向保证已运行确认同步。

### R27｜骨盾免伤还有 not ignore_absorb 门禁

- 位置：[stalker_shield1:59](../../../wiki/reference/prefabs/stalker_shield1.md#L59)。
- [nodmgshielded](../../../raw/current/scripts/prefabs/stalker.lua#L726-L728) 返回 hasshield and amount<=0 and not ignore_absorb；[Health:DoDelta](../../../raw/current/scripts/components/health.lua#L613-L624) 将 ignore_absorb 传入 redirect。
- 建议把“对伤害全拦／有盾免伤”限定普通未忽略吸收的扣血路径；attacked 仍可出盾的事件链可保留，不必扩写远古织影者全部战斗机制。

### R28｜取消击退任务不等于恢复受击者的 MotorVelOverride

- 位置：[stalker_shield1:68](../../../wiki/reference/prefabs/stalker_shield1.md#L68)、72、79。
- [UpdateRepel](../../../raw/current/scripts/prefabs/stalker_shield.lua#L9-L41) 在受击者物理体上写 override；清它依赖更新达到停止条件或 [TimeoutRepel](../../../raw/current/scripts/prefabs/stalker_shield.lua#L43-L52)。两任务都挂 FX，而 [FX Remove](../../../raw/current/scripts/entityscript.lua#L1741-L1746) 会取消任务；工厂没有另接 OnRemoveEntity 恢复受击者。
- 建议把“最多 0.33 秒／无需手动清理”限定为正常回调跑完；提前删 FX 时不能只说任务随实体取消便视作完整收尾。保留动画帧长与被打断行为未运行核验，不报告原版必现残留，也不擅改游戏。

## P3：表述

### R29｜源码选了 10 个动画名，不证明它们都是单帧动画

位置：[wintersfeastbuff_fx:33](../../../wiki/reference/prefabs/wintersfeastbuff_fx.md#L33)。[effectfn](../../../raw/current/scripts/prefabs/wintersfeastbuff.lua#L137-L166) 只能证明随机选 1～10 的动画变体；本页又声明未核验动画包内容。建议删“单帧”，无需为修措辞扩展资源检视。

### R30｜拟态虫简中官方名应取 msgstr

位置：[itemmimic_revealed_shadow:22](../../../wiki/reference/prefabs/itemmimic_revealed_shadow.md#L22)。[该 .po 项](../../../raw/current/scripts/languages/chinese_s.po#L344145-L344147) 的 msgid 是 Mimicreep、msgstr 是“拟态蠕虫”。标题与验证记录已用中文，建议本体辨析统一写简中官方名；若保留英文则注明英文原名，避免混淆翻译字段。

## 已执行的机械核验及限制

| 项目 | 实际结果 |
|---|---|
| validate_page --check-indexes，本批 12 页 | pages.count=12，failed=[]；12 页格式通过 |
| 全局导航 | 初检 35 索引／1,172 知识页，exit 1：并行 misc-equipment-2、ranged-weapons、staffs-more 各缺叶登记；他方完成登记后，本轮末重跑同一 --check-indexes，navigation.passed=true、issues=[]、整体 exit 0。保留两个时点，未代改并行页 |
| validate_source_links --strict，本批 | 613 条引用／613 行锚点；523 唯一目标、99 源文件；0 error／0 warning，exit 0 |
| validate_source_hashes check --fresh-hours 0，本批 | 123 组 unchanged；0 changed／invalid／unrecorded；只 check，不 record |
| query_wiki read | 逐页以标题原文“注册与构造”读取，max-bytes=65536；12/12 exit 0，每次 results=1、complete_selected_ranges=true、truncated=false；不使用检索缓存 |
| Lua 示例 | 12 段由 apply_patch 临时建文件，用本机既有 parser_probe --syntax-only 全部 exit 0；已用 apply_patch 删除临时文件。未跑游戏／Lua 语义运行；R01 正是语法解析发现不了的问题 |
| 注册、名称与来源扫描 | 全 raw/current/scripts 的 4,095 个 .lua，37 名逐个字面反查，追查 stalker_shield 动态拼名、SKIN_FX_PREFAB 与原版池 helper；完整候选结果未截断，区分注册／生成／依赖／注释；37 名无跨页 aliases 归属冲突 |
| .po 核对 | 37 个注册名逐键查 NAMES，仅 archive_switch_base 有命中；核对正文 35 个不同 .po 引用区间及宿主／皮肤名称。英文名表述问题见 R30 |
| 输入稳定性 | 本批 12 页＋96 个声明源文件共 108 输入，前后 SHA-256 全一致；未覆盖并行设备页的变动 |
| 运行与资源 | 未运行游戏，未拆检动画包帧内容；未核验原生 SetParent 删除、网络最终渲染与实际听感；不将这些未知写成通过或必现错误 |

源码扫描 UTF-8／UTF-8 BOM 读取，easing.lua 和 nis/xbnyn.lua 按原有编码回退 latin-1；未使用 ignore 丢字符。字面结果没有把 stalker_shield4 的零完整名命中当未注册：其生成与注册均由循环／动态拼名确认。bishop_charge_hit 仍有旧链生成，见 R22。

## 前轮边界与后续

本轮开始顺手核对第 15 轮四项补正：R19、R23、R25 已对上源码；R06 的 [boatrace_start_flag 第 31 行](../../../wiki/reference/prefabs/boatrace_start_flag.md#L31) 仍保留 fx_spark_parts_01..03，尚未与选手 winid 公式完全同步。R29 证据组、R30 修订日期也仍待处理。只记录实况，不回填中断收尾或自动关闭前轮。

本轮属于首次整页独立检视，累计 **150／158 页**，余 **8 页，预计 1 轮**：abigailsummonfx、abysspillar_fx、acidraindrop、diseaseflies、explode_small、grottopool_sfx、impact、ruins_cavein_obstacle_rubble_fx。这个数字表示首轮检视覆盖，不表示所有已检视页均修后收口；历史各轮待复验项继续保留。

修本轮时同步摘要、快速判断、示例、陷阱及验证记录，核验新采用来源并按规则处理本页基线和 updated；不要为改页面扩写 Skill／工具或顺带调整并行装备库。复验须重跑本批定向检查，不能以当前机械全绿代替上述语义修复。

## 修复处置（2026-11-07）

R01～R30 全部采纳，12 页均改并同步 `updated: 2026-11-07`：

- R01 伯尼示例把 `EndFire` 定义前移、去掉实体任务多余的 `inst` 实参；R14 删除“交换两行会多出 20 秒”的因果；R15 补 `onLeaderChanged` 重新挂 `canlight` 的门禁。R03 在 archive／atrium／witchbroom／beargerfur／bat／bishop／itemmimic 七页分开 Lua `children` 删除与未核验的原生 `SetParent`；atrium 另撤回“预留接口”（R05）、首次 `SetFX("idle")` 只记状态（R04）。
- R02 archive 补 `archive_chandelier.lua`／`tuning.lua`，toadstool 补 `map/terrain.lua`／`map/static_layouts/toadstool_arena.lua` 并 record。
- R06 彩带 buff 改“Extend 空回调、OnTick 维持”；R07 密度改“剩余越少越疏”；R08 改每次新建实体；R09 收尾限定正常烹饪完成。
- R10 分开在播／回池轨迹；R11 示例补 `GetPooledTrailFx` 协议并标片段；R12 改每非专服 2 联网层＋6 本地槽层、褪色轮询归网络层；R13 撤回“骑乘不投放”。
- R16 惊树前置改“stage 4 或可 CHOP 的 workable”；R17 `net_tinybyte` 改 0–7 并补 `netvars.lua`；R18 改各端同一 netvar 号；R19 改单次 PostUpdate；R20 示例补位置与 build。
- R21 区分 1 秒状态计时与约 0.8 秒可见窗口；R22 改“旧弹体无现役入口，但旧链仍生成 hit”；R23 粒子改无淡入、末期 alpha 0.5；R24 改“没有 level 1 播放分支”；R25 示例返回整组句柄并给 `KillFX`／`FinishFX`；R26 基名跨端效果保留未核验；R27 补 `not ignore_absorb`；R28 明确任务取消不恢复受击者 `MotorVelOverride`；R29 删“单帧”；R30 改用 msgstr“拟态蠕虫”。
- 顺带补第 15 轮遗留：[boatrace_start_flag 第 31 行](../../../wiki/reference/prefabs/boatrace_start_flag.md#L31) 的 `fx_spark_parts_01..03` 改为 `fx_spark_parts_0..winid`。

验证：本批 12 页 `validate_page` 12/12 通过；strict 623 处行引用／531 唯一目标／102 源文件，0 error／0 warning；record 7 页后本批 134 组来源 unchanged、0 changed；4 段改动示例 `parser_probe --syntax-only` 均 exit 0。未运行游戏、未改 raw、未提交推送。累计仍 **150／158 页**，余 8 页；本批修后独立复验待做，不自动收口。

## 修后独立内容复验（2026-11-07）

范围：查看本批 12 页的修改、相邻正文与必要 Lua 源码，对原 R01～R30 逐项复核，检查修订是否同步，以及新表述是否超过证据。本次未执行验证器、测试、query_wiki、哈希检查或 record；上文初检及修订方自报的机械结果不作为本次修后检查结果。机械检查按用户要求留到最后统一执行。

**原 30 项中 27 项内容通过；R03／R10／R26 尚有局部残留，新增 R31 一组证据对齐问题。当前未发现未解决 P1，仍有 4 组 P2 待补。**

| 原编号 | 本次内容复验 |
|---|---|
| R01 | 通过：EndFire 已前移，OnLight 能引用该局部函数；DoTaskInTime 去掉重复 inst。原 P1 已修复，未据语法检查推导运行结果。 |
| R02 | 通过：archive 的 archive_chandelier／tuning 和 toadstool 的两项地图来源已在各自 sources 声明；本次未核验哈希基线。 |
| R03 | 部分通过：archive／atrium／bat／bishop／itemmimic 的对应归因已修；witchbroom 有旧说法残留，beargerfur_sack 对确定的显式清理又误加未知，见下文。 |
| R04～R09 | 通过：首次 idle 不换标签、highlight 按需装配、Extend 空回调、冬宴密度方向、新建粒子与烤炉正常收尾已对齐；R06 的新增来源声明漏项另列 R31。 |
| R10 | 部分通过：在播项与回池项已分开，但“池表不可达”及“不会再收到 animover”仍写过头，见下文。 |
| R11～R25 | 通过：池复用 ReturnToScene／RestartFx／owner 已补且明确是需补辅助函数的协议示意；2+6 层、moving 门禁、20 秒时序、canlight 恢复门禁、惊树或条件、tinybyte 值域、同一变体号、单次 PostUpdate、本地示例位置/build、预警窗口、旧链引用、包络、level1 及句柄收尾均已修。R17 的正文证据补链另见 R31。 |
| R26 | 部分通过：已撤回最终外观不同步的结论，但仍把“各端各随机一次”当已核验事实，见下文。 |
| R27～R30 | 通过：ignore_absorb 门禁、取消任务不等于恢复物理状态、删除单帧断言及 msgstr 中文名均已补正。 |

### R03 残留｜两处清理说明仍不一致

- [witchbroom_fx:55](../../../wiki/reference/prefabs/witchbroom_fx.md#L55) 仍写“本地子体随联网层级联”，与已修的第 76 行“原生删除结果未核验”冲突。[本地槽层构造](../../../raw/current/scripts/prefabs/woby_rack.lua#L219-L229) 仅 SetParent；把第 55 行同步到第 76 行的边界即可。
- [beargerfur_sack_frost_fx:68](../../../wiki/reference/prefabs/beargerfur_sack_frost_fx.md#L68) 新写“桶体直接移除时霜气只经 SetParent、原生结果未核验”，但同一行右列与第 42 行又明确宿主 OnRemoveEntity 清理。源码 [第 204 行](../../../raw/current/scripts/prefabs/beargerfur_sack.lua#L201-L204) 确实安装该回调，[回调](../../../raw/current/scripts/prefabs/beargerfur_sack.lua#L129-L132) 调 ToggleFrostFX(false,true)，后者 [直接 Remove 霜气](../../../raw/current/scripts/prefabs/beargerfur_sack.lua#L38-L51)。建议写“桶体移除由 OnRemoveEntity 显式删霜气”；只有伯尼火层直接随父删除的原生结果仍未知，不要把该未知套到桶霜气。

### R10 残留｜不能由载体 Remove 推出池表不可达

- 位置：[witchbroom_fx:45](../../../wiki/reference/prefabs/witchbroom_fx.md#L45)、[第 75 行](../../../wiki/reference/prefabs/witchbroom_fx.md#L75)及第 147 行补正记录。
- 原版新建轨迹会 [设置 fx.owner=载体](../../../raw/current/scripts/prefabs/witchbroom.lua#L249-L259)，回池时载体又 [保存轨迹引用](../../../raw/current/scripts/prefabs/witchbroom.lua#L214-L221)。载体实体失效不等于 Lua 表引用都被清空，不能称“载体没了池表不可达”。
- 回池后没有显式安排下一次动画播放，也不能只凭这段 Lua 断言引擎“永远不会再发 animover”。建议统一为“已回池项没有显式清池路径，不能依赖再次 animover 保证回收；最终结果未核验”，保留在播项结束检查 owner 的确定路径，不引申为必现泄漏。

### R26 残留｜仍未核验客户端是否调用随机基名工厂

- 位置：[stalker_shield1:97](../../../wiki/reference/prefabs/stalker_shield1.md#L97)。
- “网络构造名未核验”之后仍写“只记录各端工厂各执行一次随机”。[MakeShield](../../../raw/current/scripts/prefabs/stalker_shield.lua#L80-L103) 的随机表达式仅在 num 为 nil 时选随机数；基名工厂随后改为定号 prefab，而 [SpawnPrefabFromSim](../../../raw/current/scripts/mainfunctions.lua#L388-L399) 使用工厂最终 prefab。尚未核验客户端收到基名还是定号构造，不能保证每端都走随机分支。
- 建议收窄为“基名工厂被调用时随机选型并改为定号名；客户端实际构造路径与最终外观未核验”。无需新增网络测试或扩展主题。

### R31｜新增证据的声明与正文锚点未完全对齐（P2）

| 页面位置 | 内容核对结果与建议 |
|---|---|
| [balloon_held_child:58](../../../wiki/reference/prefabs/balloon_held_child.md#L58)、第 11 行 | 新增 components/debuff.lua 第 53～57 行链接，确能支持 Extend 无回调不做事；sources 仍只有 components/debuffable.lua，没有 components/debuff.lua。补入实际采用来源，最后统一按规则处理本页基线。 |
| [bat_bosshat_fx:118](../../../wiki/reference/prefabs/bat_bosshat_fx.md#L118)、第 11 行 | netvars.lua 已进入 sources，但正文还没有指向它的行锚点。0～7／0～255 数值已正确，给该句补 [netvars.lua 第 6～9 行](../../../raw/current/scripts/netvars.lua#L6-L9) 或对应选型 helper 的证据即可。 |

此外，第 15 轮 [boatrace_start_flag:31](../../../wiki/reference/prefabs/boatrace_start_flag.md#L31) 已改为 fx_spark_parts_0..winid，原烟花 01..03 残留的内容补正通过；第 15 轮其他待办本次没有扩大复验，不据此整轮收口。

本次只追加复验结论并更新报告首页、维护日志，未改被检视知识页或他方修订记录。累计仍为首次整页检视 150／158，余 8 页；待上述局部补正后再看内容，全部准备好后统一做一次机械检查。

### R03／R10／R26／R31 局部补正（2026-11-07）

- R03：witchbroom 第 55 行同步为“本地子体只 `SetParent`、未登记 Lua `children`，网络层删除时是否带走未核验”；beargerfur 桶霜气恢复“桶体 `OnRemoveEntity` 显式 `ToggleFrostFX(false, true)` 直接 Remove”，只把伯尼火层的原生父挂删除留未知。
- R10：删除“池表不可达”“不会再收到 animover”；第 45、75、84 行统一为“已回池项没有显式清池路径，不能依赖再次 animover 保证回收，最终结果未核验”，保留在播项 animover 检查 owner 的确定路径。
- R26：收窄为“`MakeShield` 的随机分支只在 `num == nil` 时执行；基名工厂被调用时随机并改为定号名，客户端实际构造路径与最终外观未核验”，不再写各端各随机一次。
- R31：balloon 的 sources 补 `components/debuff.lua` 并 record；bat 第 118 行补 `netvars.lua` 第 6–15 行锚点。

验证：五页 `validate_page` 5/5；strict 300 处行引用／259 唯一目标／62 源文件，0 error／0 warning；五页 73 组来源 unchanged、0 changed；`git diff --check` 干净。未运行游戏、未改 raw、未提交推送；累计仍 150／158 页，余 8 页。

## 剩余项再次内容复验（2026-11-07）

实际源码快照仍为 steam-buildid:26335394。本次复核剩余四组的五个页面、相关验证记录及必要 Lua 实现；没有扩大为新一轮全库审查，也没有执行校验器、测试、query_wiki、哈希检查或 record。上节机械结果为修订方记录，不是本次独立复验重跑的结果。

| 待补项 | 本次内容结论 |
|---|---|
| R03 | 通过：[witchbroom_fx:55](../../../wiki/reference/prefabs/witchbroom_fx.md#L55) 已与第 76 行统一为 SetParent 原生结果未核验；[beargerfur_sack_frost_fx:68](../../../wiki/reference/prefabs/beargerfur_sack_frost_fx.md#L68) 已恢复 OnRemoveEntity→ToggleFrostFX(false,true) 显式删除，与第 42 行及源码安装回调、调用路径一致；伯尼火层另保留未知。 |
| R10 | 通过：[witchbroom_fx:45](../../../wiki/reference/prefabs/witchbroom_fx.md#L45)、75、84、147 行均改成“回池项无显式清池路径，不能依赖再次 animover 保证回收，最终未核验”，不再断言池表不可达或绝不再有事件；在播项的 owner 有效性检查与回收路径保留。 |
| R26 | 通过：[stalker_shield1:97](../../../wiki/reference/prefabs/stalker_shield1.md#L97) 已把随机限定为 num==nil 的基名工厂被调用时；客户端实际构造名、是否重跑随机与最终外观均留未知，不再假定各端必各随机一次。 |
| R31 | 通过：[balloon_held_child:10](../../../wiki/reference/prefabs/balloon_held_child.md#L10) 已声明 components/debuff.lua，正文第 58 行已有对应 Extend 锚点；[bat_bosshat_fx:118](../../../wiki/reference/prefabs/bat_bosshat_fx.md#L118) 已补 netvars.lua 第 6～15 行，支持 tinybyte 0～7 与 byte 0～255。 |

结合上次已通过的 27 项，原 R01～R30 与新增 R31 的内容问题均已处理，本次未发现新的阻断内容项。**本批内容可收口；统一机械检查仍待最后一次执行，不等同全库或游戏运行验收。**

仅更新本报告首页和追加本段、维护日志，未改被检视页面、他方交付记录或基线；不提交推送。累计首次整页检视仍为 150／158，余 8 页约一轮；其他历史轮次的待办保持独立，不自动关闭。

