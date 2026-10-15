# FX 独立检视第 11 轮：外围 F～I

检视日期：2026-11-06。源码实读 `raw/current/BUILD.txt`：Steam buildid **26335394**；知识仓 HEAD 为 `97afb4e9e6ca05390558769926f1f9c8598d0f70`，工作区有他方未提交改动，予以保留。本轮只新增检视报告、记录，并定向收口上一轮已修的 R05，不修改知识页、索引、基线、PLAN、台账、Skill 或工具，不提交推送。

**当前结论：已收口。初检 P1×0、P2×9、P3×4，R01～R13 均已修改并独立复验通过；未发现新的阻塞问题。** 下文保留初检记录，最终复验见文末。同根问题跨页合并为一个编号，数量不按出现次数累加；机械校验不替代源码事实核对。

## 范围与逐页结论

本轮接续外围 F～I，12 个新页、**80 个实际注册名**，共 **1312 物理行／253064 字节**。通读页面，核验 46 个工厂来源文件中的相关构造、变体、端、生成和清理代码，并追查必要宿主、SG 与 helper；不声称通审这 46 个文件的全部本体玩法。已有台账的 accepted 和其他 AI 的复审记录只作定位线索。

| 批次 | 页面 | 注册数 | 本轮结论 |
|---|---|---:|---|
| F | [fence_gate_anim](../../../wiki/reference/prefabs/fence_gate_anim.md) | 12 | P2：R01；P3：R12 |
| F | [archive_chandelier](../../../wiki/reference/prefabs/archive_chandelier.md) | 8 | P2：R02 |
| F | [lavaarena_beetletaur_fossilized_break_fx_right](../../../wiki/reference/prefabs/lavaarena_beetletaur_fossilized_break_fx_right.md) | 7 | 未发现 P1/P2 |
| G | [armor_lunarplant_glow_fx](../../../wiki/reference/prefabs/armor_lunarplant_glow_fx.md) | 7 | 未发现 P1/P2 |
| G | [alterguardian_laserscorch](../../../wiki/reference/prefabs/alterguardian_laserscorch.md) | 8 | P2：R01 |
| G | [charlie_boss_aoe_flame_fx](../../../wiki/reference/prefabs/charlie_boss_aoe_flame_fx.md) | 7 | P2：R09 |
| H | [groundpoundring_fx](../../../wiki/reference/prefabs/groundpoundring_fx.md) | 6 | 未发现 P1/P2 |
| H | [deerclops_laserscorch](../../../wiki/reference/prefabs/deerclops_laserscorch.md) | 5 | 未发现 P1/P2 |
| H | [ice_puddle](../../../wiki/reference/prefabs/ice_puddle.md) | 4 | P2：R01、R04；P3：R10 |
| I | [mushtree_tall_webbed_burntfx](../../../wiki/reference/prefabs/mushtree_tall_webbed_burntfx.md) | 6 | P2：R01、R06、R07；P3：R13 |
| I | [madscience_lab_goop](../../../wiki/reference/prefabs/madscience_lab_goop.md) | 6 | P2：R01、R03、R08；P3：R11 |
| I | [moon_device_pillar](../../../wiki/reference/prefabs/moon_device_pillar.md) | 4 | P2：R05 |

## P2：事实精度、证据边界与页内一致

### R01：五页仍把 SetParent 当作已核验的删除协议

位置：

- `fence_gate_anim` [摘要第 6 行](../../../wiki/reference/prefabs/fence_gate_anim.md#L6)、[案例第 63 行](../../../wiki/reference/prefabs/fence_gate_anim.md#L63)、[限制第 121 行](../../../wiki/reference/prefabs/fence_gate_anim.md#L121)，以及第 55 行，断言“宿主移除即销毁／靠父实体销毁带走”。
- `ice_puddle` [第 58～64 行](../../../wiki/reference/prefabs/ice_puddle.md#L58-L64)、[第 101 行](../../../wiki/reference/prefabs/ice_puddle.md#L101)，把水洼 `SetParent` 写成“随宿主一起移除”。同页涟漪走 `AddChild`，两者不能混为同一种已核验清理。
- `alterguardian_laserscorch` [第 70 行](../../../wiki/reference/prefabs/alterguardian_laserscorch.md#L70)，写地面子环“随父实体，不单独清理”。
- `mushtree_tall_webbed_burntfx` [第 49 行](../../../wiki/reference/prefabs/mushtree_tall_webbed_burntfx.md#L49)，前半已注明隐藏态引擎级联未核验，后半仍写“隐藏态随父实体”。
- `madscience_lab_goop` [第 88 行](../../../wiki/reference/prefabs/madscience_lab_goop.md#L88)，写 goop/glow“生命周期随父级”，而第 81 行刚说明引擎级联未核验。

依据：Lua [`AddChild`](../../../raw/current/scripts/entityscript.lua#L921-L935) 会登记 `self.children`；[`Remove` 的子项循环](../../../raw/current/scripts/entityscript.lua#L1779-L1796) 只显式遍历该表。上述对象分别直接 [`SetParent`](../../../raw/current/scripts/prefabs/furniture_chairs.lua#L295-L299)、[水洼父挂](../../../raw/current/scripts/prefabs/rock_ice.lua#L352-L358)、[子环父挂](../../../raw/current/scripts/prefabs/alterguardian_phase4_lunarrift.lua#L1175-L1178)、[隐藏残骸父挂](../../../raw/current/scripts/prefabs/lunar_grazer.lua#L75-L85)、[实验台父挂](../../../raw/current/scripts/prefabs/madscience_lab.lua#L288-L295)。牧者的 Lua 移除回调仅在 [`debrisshown` 为真](../../../raw/current/scripts/prefabs/lunar_grazer.lua#L161-L168) 时逐枚 Remove。

建议：分别写实际父挂与显式清理路径；没有 Lua 显式清理证据的分支保留“宿主删除时的引擎级联未核验”。自建宿主的示例说明由控制方保存引用并显式清理。**本项不反向断言这些实体运行时必然泄漏，也不要求为了消除未知而改游戏源码。** 同步摘要、案例注释和陷阱，不只改其中一处。

### R02：无专用 variation netvar，不能推出远端只播默认 idle

位置：`archive_chandelier` [第 119 行](../../../wiki/reference/prefabs/archive_chandelier.md#L119)，以及第 60 行“变体只有主机设置，无 netvar”的上下文。

问题：`SetVariation` 只在主模拟暴露、Lua 字段 `variation` 没有专用 netvar，这两点成立；但“远端客户端按 fn 默认 idle 绘制”没有相应依据。工厂已有 [`AnimState` 与 `Network`](../../../raw/current/scripts/prefabs/archive_chandelier.lua#L331-L345)，[`SetVariation`](../../../raw/current/scripts/prefabs/archive_chandelier.lua#L413-L432) 直接改这个联网实体的动画，不是在主机创建一份独立本地动画。“没用 netvar 同步 Lua 字段”不能等同“AnimState 改动不复制”。本批牧者页第 107 行恰好也区分了这两件事。

建议：保留“客户端没有 `SetVariation` 方法／不持有 variation 字段”的调用边界，删除远端必然停在默认 idle 的结论；实际跨端画面列为未运行核验，不据缺少专用 netvar 判定同步失败。

### R03：区域雾示例将 UnitRand 再转换一次，采样偏向负方向

位置：`madscience_lab_goop` [第 65～74 行](../../../wiki/reference/prefabs/madscience_lab_goop.md#L65-L74)，关键是第 69 行。

源码 [`UnitRand`](../../../raw/current/scripts/emitters.lua#L101-L103) 已返回 `math.random()*2-1`。示例的 `(UnitRand()*2-1)*radius` 因此覆盖约 `[-3r,r]`，中心为 `-r`，并非以实体原点为中心的 `[-r,r]`。粒子组件直接消费 [`area_emitter()` 的相对坐标](../../../raw/current/scripts/components/emitter.lua#L33-L50)，不会替调用者纠正这一偏移。

建议：方形采样直接用 `UnitRand()*radius`，或用 `(math.random()*2-1)*radius`；说明这是方形相对坐标采样，与原版 `CreateAreaEmitter(node.poly,node.cent)` 的多边形采样不同。只需修改本例，不改变公共 helper。

### R04：ice_splash 的普通融化与强制清理不是同一端条件

位置：`ice_puddle` [第 23 行](../../../wiki/reference/prefabs/ice_puddle.md#L23)、[第 45 行](../../../wiki/reference/prefabs/ice_puddle.md#L45)、[第 102～105 行](../../../wiki/reference/prefabs/ice_puddle.md#L102-L105)；案例第 83 行的非专服建议可保留，但不能当作工厂的硬门禁。

问题：页中概括“专服不生成／只在融化路径出现”，并把大霜鲨冰块的整行都归于 `_shouldlocalfx`。实际普通阶段水花确有 [`_shouldlocalfx` 与 `ismelt` 门禁](../../../raw/current/scripts/prefabs/sharkboi_ice_hazard.lua#L74-L80)，但 [`CleanupForced`](../../../raw/current/scripts/prefabs/sharkboi_ice_hazard.lua#L278-L285) 只检查 `not inst:IsAsleep()`，没有检查这两个值，也没有 `IsDedicated` 检查；该方法在[主模拟分支暴露](../../../raw/current/scripts/prefabs/sharkboi_ice_hazard.lua#L349-L364)，由[竞技场清理事件调用](../../../raw/current/scripts/prefabs/sharkboi_ice_hazard.lua#L390-L393)。`ice_splash` 工厂自身也[没有端门禁](../../../raw/current/scripts/prefabs/ice_splash.lua#L6-L24)。

建议：区分 rock_ice／temperature／hazard 的普通融化与 hazard 强制清理；后者写“主模拟清理时，未休眠则生成非联网水花，不复制到远端”，不把“普通路径只在非专服”扩成所有调用点的结论。不要反向宣称专服一定可见。

### R05：moon_fissure 的 _level 在本文件有持续读取

位置：`moon_device_pillar` [第 80 行](../../../wiki/reference/prefabs/moon_device_pillar.md#L80)“本文件只写不读”，并核对第 51 行“构造时两端都调一次”的概括。

[`OnUpdateLight`](../../../raw/current/scripts/prefabs/moon_fissure.lua#L45-L66) 第一行就读 `_level:value()`；工厂在[客户端早退之前注册周期任务并调用一次](../../../raw/current/scripts/prefabs/moon_fissure.lua#L156-L183)。主模拟另有[睡眠取消、唤醒恢复](../../../raw/current/scripts/prefabs/moon_fissure.lua#L136-L148)；`aurafn`、`getstatus` 也读 `_level`。因此“不监听 leveldirty”成立，“只写不读”不成立。

建议：写成“本 prefab 不监听 leveldirty，而由两端周期 OnUpdateLight 读取 _level 更新灯光；主模拟还用于理智光环／检查状态”。保留层动画调用位置与未运行核验边界，不为此建议补一个原版已经不需要的读取通道。

### R06：焦灌木本地代理是 RGB 乘 0.4，不是 alpha 0.4

位置：`mushtree_tall_webbed_burntfx` [第 35 行](../../../wiki/reference/prefabs/mushtree_tall_webbed_burntfx.md#L35)。

源码是 [`SetMultColour(.4,.4,.4,1)`](../../../raw/current/scripts/prefabs/marsh_bush.lua#L183-L189)，即 RGB 压暗、alpha 为 1。页面写 `α0.4` 会把变暗误当半透明。

建议：改为“MultColour=(.4,.4,.4,1)，RGB 压暗，alpha=1”；动画与侵蚀的实际透明变化另依资源／运行核验，不能用本行参数替代。

### R07：焦灌木空壳是否可父挂，案例与陷阱再次矛盾

位置：`mushtree_tall_webbed_burntfx` [第 97 行](../../../wiki/reference/prefabs/mushtree_tall_webbed_burntfx.md#L97) 已正确写“仍有 Transform/Network，可以 SetParent”，但[第 103 行](../../../wiki/reference/prefabs/mushtree_tall_webbed_burntfx.md#L103) 仍写“不要把壳当可加动画／可父挂的普通 FX”。

[`burnt_erode_fn`](../../../raw/current/scripts/prefabs/marsh_bush.lua#L194-L223) 缺 AnimState，但有 Transform/Network；这只能支撑“不能直接改壳的动画”，不能支撑“不可父挂”。

建议：第 103 行去掉“可父挂”禁令，保留“动画在本地代理；原版不父挂，额外父挂后的表现未运行验证”。这是前次结论修订未同步清除的旧说法。

### R08：防滑砂的 10～30 是每路发射率，合计需计两路

位置：`madscience_lab_goop` [第 94 行](../../../wiki/reference/prefabs/madscience_lab_goop.md#L94)“每秒 10～30 粒”。

位移决定的 [`low_per_tick/high_per_tick`](../../../raw/current/scripts/prefabs/nonslipgrit.lua#L349-L370) 确实由 10、30 乘 tick_time 得到，但每次 [`buff_fx_emit`](../../../raw/current/scripts/prefabs/nonslipgrit.lua#L274-L301) 同时向 emitter 0 和 1 各加一粒。当前说法没有区分单路与合计。

建议：标为“每路目标发射率 10～30 粒/秒，两路合计 20～60 粒/秒；实际逐帧数量由累计余量与 while>0 离散发射决定”，不将目标速率当作任意一秒窗口严格实测数量。保持第 34 行“两路各 50 粒上限”与发射速率的区分。

### R09：minion 的工作扫描不能等同攻击的 2.4 窄相半径

位置：`charlie_boss_aoe_flame_fx` [第 23 行](../../../wiki/reference/prefabs/charlie_boss_aoe_flame_fx.md#L23)、[第 51 行](../../../wiki/reference/prefabs/charlie_boss_aoe_flame_fx.md#L51)。

[`InitMinion`](../../../raw/current/scripts/prefabs/charlie_boss_minions.lua#L82-L100) 传 `AOEUtil.Work(inst,2.4,targets,caster)` 成立。但 [`_Work`](../../../raw/current/scripts/aoeutil.lua#L154-L169) 设置 `WORK_RADIUS_PADDING=.5`，以 [`radius+padding` 查询](../../../raw/current/scripts/aoeutil.lua#L221-L250)，本调用不传 arc/hitbox，没有再按 2.4 做距离筛选；这与 [`Attack` 的 radius+目标物理半径窄相](../../../raw/current/scripts/aoeutil.lua#L109-L129) 不同。

建议：工作项写“传入 radius=2.4，helper 查询半径 2.9；当前调用无额外 arc/hitbox 窄相”，不概括为“只处理 2.4 范围内”。攻击项保留其实际判定，不把两者改成同一种半径。这里只核实 Lua 查询参数与筛选路径，不推断引擎查询的全部边界行为。

## P3：局部表述与导航

### R10：nitre_formation 单独生成仍有可见 AnimState

`ice_puddle` [第 104 行](../../../wiki/reference/prefabs/ice_puddle.md#L104) 写“联网空壳”。工厂明确[设置 bank/build/idle1 与缩放](../../../raw/current/scripts/prefabs/nitre_formation.lua#L29-L51)，没有玩法组件并不等于无视觉。

建议：改为“可见的联网表现实体；独立生成时缺少池塘侧的生命周期管理，也没有由生成者设置 persists=false”。保留宿主名高亮门禁，不再用“空壳”暗示无动画。

### R11：实验台页是四组、六个注册，不是四个注册

`madscience_lab_goop` [摘要第 6 行](../../../wiki/reference/prefabs/madscience_lab_goop.md#L6)、[第 21 行](../../../wiki/reference/prefabs/madscience_lab_goop.md#L21)、第 24、37 行混用组数与注册数。goop/fire/glow 三个注册加 mist、buff FX、hide FX，共六个；aliases 与表格本身完整。

建议：统一“四组／六个注册”，名称否定核验按实际六个精确键记。无需拆页或新增页面。

### R12：fence 页说不纳入 .po，实际 sources 已纳入

`fence_gate_anim` [第 132 行](../../../wiki/reference/prefabs/fence_gate_anim.md#L132) 声称不把 `.po` 计入 sources，而[第 10 行](../../../wiki/reference/prefabs/fence_gate_anim.md#L10) 已包含它，本次哈希检查也确有这一依赖。

建议：统一文本与依赖口径。可以沿用该页现有来源并改记录；若维护者选择移除未实际用于正向名称结论的依赖，只改本页并按既有规则刷新该页基线。不要借此批量删全库 .po 来源。

### R13：藤蔓节的 helper 还安装了 propagator

`mushtree_tall_webbed_burntfx` [第 33 行](../../../wiki/reference/prefabs/mushtree_tall_webbed_burntfx.md#L33)、[第 68 行](../../../wiki/reference/prefabs/mushtree_tall_webbed_burntfx.md#L68) 已覆盖 burnable，但关联导航漏了该 helper 实际安装的 `propagator`。

[`vinefn`](../../../raw/current/scripts/prefabs/lunarthrall_plant.lua#L614-L619) 调 `MakeMediumBurnableCharacter`，helper [第 258～268 行](../../../raw/current/scripts/standardcomponents.lua#L258-L268) 另调 `MakeSmallPropagator` 并设 `acceptsheat=false`。建议在既有 burnable 行或相邻导航补这一层直接挂载与关键配置，链接现有组件页，不展开新的传播机制正文。

## 已核对但不列为问题的重点

- 熔炉化石碎裂：两局部工厂共 7 个注册、四向/六向、左右隐藏与 `_alt` 覆盖差异相符；页名多出的 `lavaarena_` 是检索别名，正文明确它不是注册名，不计为第 8 个。`fossilizable.lua` 与熔炉事件包缺失，保留选名与运行时画面缺口，不补造生成点。
- 护甲七注册：6/9/9 本地子帧、客户端复制回调与非专服主模拟创建、WAG 2～5 全切 05 bank、皮肤 hash、帽子前后层与电池符号 FX 的清理分工均有相应代码；不因没有独立颜色 netvar 就断言联网 AnimState 不同步。
- 两套激光：空激光仍结算，三个纯 FX 与玩法载体分工明确；三阶段 11 格与循环后补 2 道、巨鹿末两道无 caster 的区别成立。没有把空激光当无伤害表现或把血滴本体当触手伤害源。
- 中心震地环：hitrings 影响外观缩放/跳帧，不改真实逐环范围；FastForward 是本地代理创建时读取一次；碎块 7 处活跃字面 Spawn 不包含 standardcomponents 注释；willow_ember 的 spawnfirefx 只见局部定义，不把整个 willow_ember 叫死代码。
- 源码扫描覆盖 scripts 根级与子目录全部 **4095 个 Lua 文件**，4093 UTF-8、2 个旧编码按 latin-1 检索，不跳过。81 个 ASCII 检索别名（含上述非注册的 lavaarena_ 别名）累计 **331 个名称-行匹配**，无输出截断；额外 53 个未在对应页 sources 的命中已区分为布局、资源、变量/自然语言、依赖、注释与资料数据，没有据其机械判定缺生成点。补搜化石名/前缀、OnSpawnFX、steamon_fxfn、spawnfirefx；相关负向结论限当前脚本快照。
- 全部 80 个主要注册在 aliases 与正文中可定位，未发现主要 ASCII 别名跨 Prefab 页冲突。精确 NAMES 键只命中 `CHARLIE_BOSS_PROJECTILE`（群聚恐怖）和 `NITRE_FORMATION`（硝石堆）；其他宿主/override 键与正文来源分别核对，不拿宿主名替代 FX 名。

## 实际校验与限制

| 检查 | 本次结果 |
|---|---|
| validate_page --check-indexes | 12/12 passed；全库导航 1143 知识页、34 索引，0 issue |
| validate_source_links --strict | 927 行引用、777 唯一目标、125 个源文件；0 error、0 warning；exit 0 |
| 不支持目标 | 1 项普通 Markdown 页链接 petrified_trunk_break_fx.md 被源码校验跳过；目标实际存在，不称“927+1 个源码证据” |
| validate_source_hashes check --fresh-hours 0 | 151 组全部 unchanged；changed/sources_changed/unrecorded/invalid 均 0，未 record |
| sources 与正文引用 | 12 页已引用的本地行锚点均在 sources 中声明；R12 是记录对既有依赖的描述不一致 |
| 标题原文 read | 每页一个构造/案例章节，共 12 次 exit 0，含标点和括号标题 |
| 输入稳定性 | 对 12 页与 125 个声明来源文件保存内存哈希，末次比较 137 文件全部未变 |

未运行 DST；本轮对既有 Lua 片段做源码语义检视，没有改片段或重新跑语法探针，不沿用旧“语法通过”来证明本轮发现的数值/前提正确。未重建查询缓存，检索核对走当前 Markdown/源码扫描与 read，没有将 stale search 当作验收。动画资源、引擎父子删除、实际联网画面与晚加入表现仍按各页记录保留未验证边界。

## 进度与后续

- 固定 158 页新建队列，本轮新增独立检视 **12 页**；累计 **87/158 页**，余 **71 页**。这个数字表示审阅覆盖，包含待修页，不表示 87 页全部收口。
- 上一轮 Round10 R05 摘要已按建议删去泛化的 onremove 兜底，独立复验通过并另记原报告；Round10 收口。Round05 的 R10 历史项不在本次复验范围，不随之宣告关闭。
- 本轮建议按 R01～R09 修正，再集中复验八篇有问题页面；P3 为小改，可一并处理。保持原有例子与已核验边界，不增建协议页或加重 Skill。
- 下一轮可按外围 L/M/O/Q 约 11 页继续；剩余队列若每轮 12～15 页，约需 5～6 轮初检，另加实际必要的修后复验。

## 2026-11-06 修后独立复验：收口

实际改动为上表八篇有问题页面；另外四篇与初检输入哈希一致。本次逐项对照 R01～R13 的修改与上下文，定向重读相应源码，不重复展开整批 80 注册的初检。源码仍为 Steam buildid 26335394；初检的 125 个来源文件均未变化。

| 项目 | 当前修订位置与复验结论 |
|---|---|
| R01 | fence 第 6、23、55、63、115、121 行；ice 第 24、58、63、101、111 行；alter 第 55、70 行；mushtree 第 23、49 行；madscience 第 81、88 行。已区分 SetParent 跟随、实际 Remove 与未验证的引擎级联，自建示例保留显式清理责任。通过。 |
| R02 | archive 第 60、119 行。已删除远端必播默认 idle 的结论，保留主模拟方法／Lua 字段与联网 AnimState 的区分。通过。 |
| R03 | madscience 第 69、77 行。采样改为 UnitRand()*radius，注明方形相对坐标与原版多边形采样的差别。通过。 |
| R04 | ice 第 23、45、83、102、105 行。普通融化与主模拟 CleanupForced 分开，不再泛化非专服门禁，也未把非联网实例说成会复制到远端。通过。 |
| R05 | moon_device 第 51、80 行。已写明周期 OnUpdateLight 及主模拟 aurafn/getstatus 读取 _level；无 leveldirty 监听不等于只写不读。通过。 |
| R06 | mushtree 第 35 行。RGB=.4/.4/.4、alpha=1，与源码四参数一致。通过。 |
| R07 | mushtree 第 97、104 行。案例与限制都区分无 AnimState 和仍可 SetParent，不再保留不可父挂旧说法。通过。 |
| R08 | madscience 第 94 行。已区分每路 10～30 与两路合计 20～60，并注明逐帧累计／离散发射。通过。 |
| R09 | charlie 第 23、50、51 行。工作查询半径 2.9 与攻击的 2.4+目标物理半径分别说明，无 arc/hitbox 时不再把 Work 当 2.4 窄相。通过。 |
| R10 | ice 第 104 行。已改为可见联网表现实体，保留缺宿主管理与 persists 配置的边界。通过。 |
| R11 | madscience 摘要及第 21、24、37 行。统一四组／六注册，验证记录中的“四个”明确指四个文件。通过。 |
| R12 | fence 第 10、132 行。来源记录与已保留的 .po 依赖一致，没有批量删其他页面来源。通过。 |
| R13 | mushtree 第 69 行。补 propagator 导航、MakeSmallPropagator 与 acceptsheat=false，未展开额外传播机制。通过。 |

本次实际校验：

- **格式／导航**：重跑原 12 页，12/12 passed；全库导航 1143 知识页／34 索引／0 issue。
- **strict 源码引用**：930 行引用／778 唯一目标／125 源文件，0 error、0 warning，exit 0；仍跳过 1 个普通 Markdown 页链接，不算源码引用。
- **来源基线**：155 组全部 unchanged，changed／sources_changed／unrecorded／invalid 均 0。fence、alter、ice、madscience 新增 entityscript.lua 的四组依赖已由修页方刷新；本次只 check，未 record。
- **章节读取**：八篇改页各选一个相关章节，用标题原文 read，8 次 exit 0；正文行引用均已声明 sources。未重建缓存、未以 search 缓存作为检索验收。
- **Lua 示例**：madscience 唯一可执行语句修改是采样表达式；将当前完整函数临时另存并核对与页面围栏一致，parser_probe --syntax-only exit 0。临时文件经 apply_patch 创建、删除，无残留；仅查语法，未运行函数或 DST。其余示例仅改注释或说明。
- **稳定性**：复验前后 12 页＋125 个来源文件共 137 个输入的内存哈希全部一致；四篇未改页沿用初检源码语义结论。

R01～R13 全部通过，**第 11 轮收口**；未发现本轮修订引入的新阻塞问题。引擎删除级联、资源画面、实际跨端与晚加入仍未运行验证，不升级 validation。累计仍为 **87/158 页，余 71 页**，本次没有新增审阅页数，也不关闭未复验的其他轮次事项。本次只更新报告并追加日志，未修改知识页、索引、基线、PLAN、台账、Skill 或工具，未提交推送。
