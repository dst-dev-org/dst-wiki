# FX 新增页独立检视报告（2026-11-05）

## 结论与范围

本轮对本对话上次清单交接后落地的 U15～U17 **9 页**独立检视，覆盖 **19 个注册＋1 个 placer**。这些页面已有其他执行者自报的复审／验收记录，不能称为全项目从未复审；它们此前尚未经本对话独立源码检视。U1～U14 的 41 个新页及 1 个增修页已有批次记录，不纳入本轮全文重审，不能用本报告证明整轮 50 新页全部重新核验。

结论：**需修复后再确认技术验收**。知识页发现 **P1×2、P2×7**，分布在 7 页；另外，原收尾报告有 1 项 P2 验收口径问题。9 页机械检查全部通过，但这些检查没有发现伤害倍率、调用方覆盖数值与生命周期条件错误。

级别口径：P1 为影响核心数值或直接复用的明确错误；P2 为接入限制、条件遗漏、证据边界或页内矛盾。本轮不修改被检视知识页、台账验收状态或源码基线；修订建议在下文列出。

环境：K=`D:/Code/dst-wiki-develop/dst-wiki`，S=`D:/Code/dst-wiki-develop/dst-wiki-skill`，T=`D:/Code/dst-wiki-develop/dst-toolbox`；实读 `raw/current/BUILD.txt` 为 **26335394**。页面均为 `source-reviewed`；本轮未运行 DST，数值例子是源码推导，不是游戏测量。

## 逐页结果

| 批次 | 页面 | 行／UTF-8 字节 | 主要核验范围 | 结果 |
|---|---|---:|---|---|
| U15 | [ruinsnightmare_horn_attack](../../../wiki/reference/prefabs/ruinsnightmare_horn_attack.md) | 161／19,344 | 完整角工厂、SetUp、双发 SG、伤害归属与重现／休眠监听 | P2-02 |
| U15 | [shadowhand_arm](../../../wiki/reference/prefabs/shadowhand_arm.md) | 133／15,799 | 两个手臂工厂、两类生成者、stretcher、光照消散与显式移除 | P2-07 |
| U15 | [shrouden_eye_fx](../../../wiki/reference/prefabs/shrouden_eye_fx.md) | 153／17,394 | 眼睛／goop 工厂、同步、六个死亡状态、名称／检视与清理 | P2-05、P2-06 |
| U16 | [hotcold_fx](../../../wiki/reference/prefabs/hotcold_fx.md) | 88／9,504 | 工厂、距离公式、开始／放弃／完成／移除／读档链 | P2-03 |
| U16 | [terrarium_fx](../../../wiki/reference/prefabs/terrarium_fx.md) | 98／10,592 | 工厂、TurnOn／StartSummoning／TurnOff、昼夜计时与读档 | P2-04 |
| U16 | [wagboss_missile_target_fx](../../../wiki/reference/prefabs/wagboss_missile_target_fx.md) | 177／24,219 | 五个工厂、落点环、两套 SG、电束采样缓存与 combat 电系结算 | P1-01、P2-01 |
| U17 | [wx78_taser_projectile_fx](../../../wiki/reference/prefabs/wx78_taser_projectile_fx.md) | 132／13,796 | 完整 FX／蓄能组件、模块调用方、伤害／attacker 与清理 | P1-02 |
| U17 | [yotr_fightring_torch](../../../wiki/reference/prefabs/yotr_fightring_torch.md) | 133／15,930 | 四注册与 placer、八火炬、事件、钟、回收与任务取消 | 未发现 P1／P2 |
| U17 | [spellmasterybuff](../../../wiki/reference/prefabs/spellmasterybuff.md) | 81／9,334 | 三注册完整可见工厂、依赖与 event_server_data 入口、检索缺口 | 未发现 P1／P2；事件包行为仍未核验 |

## P1：核心数值

### P1-01 WAG 电束／电球把默认伤害写成最终攻击伤害

- 页面位置：[wagboss_missile_target_fx.md:69](../../../wiki/reference/prefabs/wagboss_missile_target_fx.md)，以及 L113～L121 的伤害表。
- 现文把四个混合注册写成“30，电免疫 15”，并明确写“非免疫直接 30”。这漏掉了 `DoAttack(..., "electric")` 的公共结算倍率。
- 证据：电束的[攻击调用](../../../raw/current/scripts/prefabs/wagdrone_laserwire.lua#L151-L157)与电球的[攻击调用](../../../raw/current/scripts/prefabs/wagdrone_projectile.lua#L122-L128)均传入 electric；[combat.lua L1195～L1212](../../../raw/current/scripts/components/combat.lua#L1195-L1212)对非电免疫目标计算 `1.5 + GetWetMultiplier()`，再传给 `CalcDamage`；倍率常量见 [tuning.lua L659～L660](../../../raw/current/scripts/tuning.lua#L659-L660)。
- 正确口径：30 是默认基础伤害。其余修正为 1 时，非免疫干燥目标的攻击输入是 45、湿度倍率为 1 时是 75；电免疫分支先把默认伤害降至 15，再跳过额外电系增幅。实际掉血还受目标防具、抗性等影响，不能直接当实测掉血。
- 建议：伤害表分别写基础值、绝缘分支与电系倍率，修正“非免疫直接 30”；补充实际支撑该结论的 `components/combat.lua` 来源与锚点，并同步快速判断／验证记录。不能只把表格的 30 替成固定 45，因为湿度仍会改变结果。

### P1-02 WX-78 页漏掉模块数量对伤害与半径的覆盖

- 页面位置：[wx78_taser_projectile_fx.md:64](../../../wiki/reference/prefabs/wx78_taser_projectile_fx.md)，另见 L91、L116。
- 现文把原版组件调用总结为满蓄能“50 伤害／2 半径”，并写“组件传的是 50×perc／2×perc”。这只取了组件构造默认值，没有追到原版模块激活后的 setter。
- 证据：[wx78_moduledefs.lua L667～L676](../../../raw/current/scripts/wx78_moduledefs.lua#L667-L676)按模块数 n 写入 `blast_damage=n*50`、`blast_radius=2+n*1`；激活链见 [L690～L695](../../../raw/current/scripts/wx78_moduledefs.lua#L690-L695)。[wx78_taserbuildup.lua L148～L150](../../../raw/current/scripts/components/wx78_taserbuildup.lua#L148-L150)再把两个当前字段乘蓄能百分比 p，传给 Explode。
- 正确口径：在该原版模块接线下，参数为 `damage=50*n*p`、`radius=(2+n)*p`；例如 1 模块满蓄能是基础参数 50／3，2 模块满蓄能是 100／4。伤害还要经过本页已描述的电系／湿度倍率。
- 建议：区分组件未被调用方覆盖的默认值、原版模块接线、手动 `Explode(50,2)` 三种场景。示例作为手动绝对参数可以保留，不能再把它解释成原版模块的固定满蓄能值；同步关联表与陷阱条目。

## P2：边界、条件与页内一致性

### P2-01 电束页遗漏 SetBeam 重配后的伤害采样缓存限制

- 页面位置：[wagboss_missile_target_fx.md:59](../../../wiki/reference/prefabs/wagboss_missile_target_fx.md)、L143～L152 的接入说明及 L159。
- 页面说明 `SetBeam` 会按新长度／角度重建分段并重置冷却，却没有提醒服务端采样点并不随这次调用重算。复用者可能把它用于持续移动／转向的伤害束。
- 证据：[RefreshSegs L179～L220](../../../raw/current/scripts/prefabs/wagdrone_laserwire.lua#L179-L220)只在 `inst.targetx == nil` 时，根据当前位置、长度和角度建立 targetx／targetz；[ClearSegs L108～L124](../../../raw/current/scripts/prefabs/wagdrone_laserwire.lua#L108-L124)只删视觉段与取消任务，不清这两个缓存；[SetBeam L245～L253](../../../raw/current/scripts/prefabs/wagdrone_laserwire.lua#L245-L253)会重建视觉，但没有使采样缓存失效。
- 影响：在同一已初始化实体上移动／转向后再次 SetBeam，视觉按新配置重建，伤害检测仍使用先前的世界坐标采样线段。这是源码路径推导，未做运行时复现。
- 建议：把原版“连接时配置一次，断开即删除”的使用前提写到接入处；需要动态重配时重建实体或另行修订采样缓存实现，不能只重复 SetBeam。现文关于活动态 SetBeam 经 ClearSegs 重建周期任务、重置冷却的结论可以保留。
- 同段顺手纠正解码公式：不是“len/15、rot/360”，而是 `len:value()/255*15` 与 `rot:value()/255*360`（L180～L181）。

### P2-02 角击页把碰撞后才安装的休眠监听写到了碰撞前

- 页面位置：[ruinsnightmare_horn_attack.md:143](../../../wiki/reference/prefabs/ruinsnightmare_horn_attack.md)。
- 现文写“到达落点前被 entitysleep 或外部 Remove”会导致没有 reappear，把休眠当作飞行阶段的删除路径。
- 证据：`entitysleep -> Remove` 只在 [TurnIntoCollisionFx L37～L53](../../../raw/current/scripts/prefabs/ruinsnightmare_horn_attack.lua#L37-L53)安装；到达碰撞点时已先在 [L58～L64](../../../raw/current/scripts/prefabs/ruinsnightmare_horn_attack.lua#L58-L64)给 owner 排 reappear。初始工厂与 SetUp 没有这一休眠删除监听。
- 建议：删除“飞行阶段 entitysleep 自删”的因果链。外部在碰撞前清除两角确实可能使重现任务未排，可单独保留；不能把休眠停更与已安装的 Remove 回调混为一谈。

### P2-03 热冷指示页的陷阱段重新引入了已排除的清理路径

- 页面位置：[hotcold_fx.md:73](../../../wiki/reference/prefabs/hotcold_fx.md)～L74；与 L54～L57 的正文冲突。
- L73 写“清理只在任务完成／放弃路径”，L74 又写“只有完成检查或实体移除路径才清指示”；正文却正确说明放弃不删、实体移除没有显式删 FX。
- 证据：[完成 L86～L100](../../../raw/current/scripts/prefabs/smallghost.lua#L86-L100)取消任务并 Remove FX；[放弃 L237～L243](../../../raw/current/scripts/prefabs/smallghost.lua#L237-L243)不做这些动作；[移除回调 L261～L268](../../../raw/current/scripts/prefabs/smallghost.lua#L261-L268)没有显式 Remove FX。
- 建议：统一为“确认的显式 FX 删除在任务完成检查；放弃回调不直接清理；宿主移除时父挂的引擎处理未核验”。不要把未核验的父挂级联当作已经确认的清理路径。

### P2-04 泰拉光柱页的快速判断／陷阱未同步正确正文

- 页面位置：[terrarium_fx.md:23](../../../wiki/reference/prefabs/terrarium_fx.md)、L83。
- L23 把清空 `_summoning_fx` 的函数写成 StartSummoning，实际是 TurnOff；L83 写白天“只起 summon_delay，等夜间到点”，而正文 L43 已写白天不立刻起计时。
- 证据：[TurnOn L181～L190](../../../raw/current/scripts/prefabs/terrarium.lua#L181-L190)只在夜晚启动／调整 summon_delay；[on_night L378～L385](../../../raw/current/scripts/prefabs/terrarium.lua#L378-L385)负责入夜后的计时；[TurnOff L271～L274](../../../raw/current/scripts/prefabs/terrarium.lua#L271-L274)才播退场、排删除并置空引用。
- 建议：L23 改正确函数名；L83 与 L43 同步，说明白天开启本身不启动召唤延迟，入夜后按条件启动。保留 `is_loading` 绕过普通夜间门禁的独立说明。

### P2-05 水洼眼状态表省略了跨死亡状态保留眼睛的门禁

- 页面位置：[shrouden_eye_fx.md:84](../../../wiki/reference/prefabs/shrouden_eye_fx.md)～L86。
- 表格对 death_loop／death_blink／death_hit 只写“onexit”清理，独立读表会理解成每次退出都删；表后正文 L89 才说明中间转换保留。
- 证据：[death_loop 的 onexit](../../../raw/current/scripts/stategraphs/SGshrouden.lua#L1061-L1076)、[death_blink 的 onexit](../../../raw/current/scripts/stategraphs/SGshrouden.lua#L1113-L1128)、[death_hit 的 onexit](../../../raw/current/scripts/stategraphs/SGshrouden.lua#L1183-L1198)均受 `not inst.sg.statemem.death` 保护，正常链内切换先置 death=true。
- 建议：表格各行补同一门禁，或在表格前定义共同清理条件并让行内明确引用；不让正文与可独立读取的状态表表达不同生命周期。

### P2-06 水洼眼页从 SetParent 直接推断 goop 随父移除

- 页面位置：[shrouden_eye_fx.md:42](../../../wiki/reference/prefabs/shrouden_eye_fx.md)。
- 现文把“SetParent 到眼睛”直接推成“因此随眼睛移动／移除”，没有给出删除级联的依据。
- 证据：[创建 L1592～L1595](../../../raw/current/scripts/prefabs/shrouden.lua#L1592-L1595)只有本地创建和 SetParent；眼睛工厂没有显式 goop Remove。现有 Lua 层 [entityscript.lua L1779～L1784](../../../raw/current/scripts/entityscript.lua#L1779-L1784)处理的是 self.children，而该段未见 AddChild 登记，不能用它补证引擎 SetParent 的删除行为。
- 建议：保留已确认的父挂关系，删除级联如无另行证据就注明未核验。此项是证据越界，**不据此反向断言 goop 必然残留**。

### P2-07 影臂页把手部消散与手臂退场对立起来

- 页面位置：[shadowhand_arm.md:78](../../../wiki/reference/prefabs/shadowhand_arm.md)，另见 L117。
- 现文说照亮起点“让手部退场，而不是让手臂退场”“不是手臂的清理”，容易被截取成手臂在此路径不会退场。
- 证据：[Dissipate L22～L48](../../../raw/current/scripts/prefabs/shadowhand.lua#L22-L48)会同时让 arm 播 arm_scare、手部播 hand_scare，再由手部 animover Remove；手部移除再经 [OnRemove L203～L209](../../../raw/current/scripts/prefabs/shadowhand.lua#L203-L209)删除 arm。
- 建议：说明“事件由手部处理，手臂没有自己的自删监听；手部同时驱动手臂退场动画，随后在自身移除时显式删手臂”。保留生成者承担清理的正确结论。

## 收尾报告的独立问题

### R-01（P2）全库 strict links 被标成通过，但存档 passed=false

- 位置：[fx-final-report-20261105.md](fx-final-report-20261105.md) §6 的“全库 strict links”勾选，以及开头的“全库校验通过”。
- 本轮读取既有 `output/tmp/full_links.json`：`strict=true`、`passed=false`、error=0、warning=27、truncated=false。工具退出契约见 T 的 `validate_source_links.py` L241、L263：strict 下 warning 非零即不通过。
- 建议：写成“本批九页 strict 通过；全库 0 error、27 条存量 warning，strict 未通过，存量问题另列处理”。如项目允许带既有 warning 收口，应明确这是人工接受的范围，不能把工具失败显示成通过。
- 该存档是原执行者的历史结果，不冒充本轮重跑全库来源链接。本轮只实际重跑九页 strict，结果为通过。

## 实际检查与边界

| 检查 | 本轮执行范围 | 结果 |
|---|---|---|
| validate_page --check-indexes | 九个绝对页面路径，格式取 S/write-format，源码为当前快照；导航由工具全局检查 | 9 页 passed，页内 issue=0；navigation passed，issue=0 |
| validate_source_links --strict | 九页全部源码链接，未截断 | 385 引用（384 行级＋1 整文件）、40 来源文件，0 error／0 warning、skipped_unsupported=0，退出 0 |
| validate_source_hashes check --fresh-hours 0 | 只传九页 | 54 组比较全部 unchanged，affected_pages=0，无 invalid／未登记，退出 0；没有 record |
| 注册归属 | 九页 19 注册＋placer，在现有非归档 Prefab aliases 中逐名反查 | 共 20 个标识，未发现跨页主要归属重复 |
| 名称 | 定向核验页面使用的 18 个宿主／身份 NAMES 键与译文 | 与正文一致；不是 FX 自身的官方名，不以宿主名替代注册名 |
| 负向检索 | 当前 scripts 全目录 Lua 中复查 spellmastery、wagpunk_cagewall 公共前缀与命中上下文 | 已见依赖／注册／加载入口，未见对应直接生成链；不把未命中提升成运行时不存在 |
| 只读边界 | 九页及其声明 sources、source-baseline 输入摘要 | 交付前复核字节哈希；知识页／来源／基线未因本轮检查被改写 |

各页正文均已读取；风险相关的伤害、父子关系、事件门禁、移除与读档路径按实际实现核对。本轮不展开所有宿主的完整 AI／玩法，也没有复跑 Lua 示例语法探针、运行客户端／专服、验证晚加入或解包动画资源。19 注册＋placer 的机械覆盖不能证明清单外 FX 全部建齐。

## 建议处理顺序

先修两项核心数值，随后修电束动态配置边界与其余生命周期／摘要矛盾；同步相关摘要、表格、陷阱、索引与验证记录。只对实际增补的来源按现有规则维护基线，不能以 record 代替重新核验。原收尾报告另纠正 strict 的验收口径；本报告保留为修复前独立检视记录，不覆盖原报告或把本轮发现改成“当时已修复”。
