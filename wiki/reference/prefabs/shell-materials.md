---
title: 海陆生物外壳（饼干切割机壳、外壳碎片）
created: "2026-11-10"
updated: "2026-11-10"
type: prefab
summary: 两份独立注册的贝壳材料——饼干切割机壳（cookiecuttershell，饼干切割机 0–2 个、瓶中信宝藏 4–6 个）与外壳碎片（slurtle_shellpieces，洞穴蛞蝓龟／蜗牛龟掉落，本版唯一 repairmaterial=SHELL 的修补材料）。本页对照两者构造（堆叠 20／40、tradable 空壳、repairer 20 点、作祟只弹飞），写清掉落表的独立掷骰与 AddIfNotChanceLoot 兜底语义，并整理制作、建造工地交付与拆解回收的消费端，以及隐士蟹线（TECH.LOST／nounlock／制作站／二级组合机）的实际门禁。
aliases: ["cookiecuttershell", "slurtle_shellpieces", "饼干切割机壳", "外壳碎片", "Cookie Cutter Shell", "Broken Shell", "贝壳碎片", "饼干壳", "切割机壳"]
tags: [prefab, item]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/cookiecuttershell.lua", "prefabs/slurtle_shellpieces.lua", "prefabs/cookiecutter.lua", "prefabs/slurtle.lua", "prefabs/slurtlehole.lua", "prefabs/singingshell.lua", "prefabs/lightcrab.lua", "prefabs/boat_bumpers.lua", "prefabs/hermitcrab.lua", "prefabs/hermithouse.lua", "prefabs/shellweaver.lua", "prefabs/hermitcrab_shell.lua", "components/lootdropper.lua", "components/repairer.lua", "components/repairable.lua", "components/tradable.lua", "components/hauntable.lua", "components/health.lua", "components/constructionsite.lua", "components/craftingstation.lua", "components/builder.lua", "prefabs/tacklesketch.lua", "components/teacher.lua", "componentactions.lua", "standardcomponents.lua", "messagebottletreasures.lua", "scenarios/chest_cavesupplies.lua", "prefabs/tree_rock_data.lua", "prefabs/tree_rocks.lua", "prefabs/worm_boss.lua", "prefabs/blueprint.lua", "recipes.lua", "tuning.lua", "constants.lua", "techtree.lua", "entityscript.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 海陆生物外壳（饼干切割机壳、外壳碎片）

## 快速判断

- 职责与适用：两个各自独立文件、各自 `Prefab()` 注册的物品，没有共享工厂。`cookiecuttershell`（饼干切割机壳）来自海洋生物饼干切割机与瓶中信宝藏，堆叠 20，是隐士蟹线（诱饵、迁居套装、盐晶组合机、房屋与温泉）的主料；`slurtle_shellpieces`（外壳碎片）来自洞穴蛞蝓龟／蜗牛龟与贝壳保险杠回收，堆叠 40，是本版唯一 `repairmaterial = MATERIALS.SHELL` 的修补材料。可借鉴：同一材料两套接线（空壳 `tradable` 对单侧 `repairer`）与两种掉落语义。[cookiecuttershell 注册](../../../raw/current/scripts/prefabs/cookiecuttershell.lua#L42)、[slurtle_shellpieces 注册](../../../raw/current/scripts/prefabs/slurtle_shellpieces.lua#L43)
- 优先使用：查两件物品的接线与数值差异 → [注册与构造](#注册与构造)、[数值与参数](#数值与参数)；查掉多少、从谁身上掉 → [获得链](#获得链)；查谁消耗它们 → [用途与配方](#用途与配方)；想让自定义结构也能用外壳碎片修 → [典型接入](#典型接入)。
- 重要副作用：`cookiecutter` 的表把两种概率写成两条 <1.0 的独立条目，所以一只掉 0–2 个；`slurtle`／`snurtle` 的碎片走 `AddIfNotChanceLoot`，语义是"本次机会表里所有 <1.0 的掷骰都没中签才补 1 件"，不是必掉。饼干切割机壳的 `tradable` 没有 `goldvalue`（默认 0），换不到金块、也不算蚁狮供品；外壳碎片对 `repairable_shell` 目标请求回 20 点（按生命上限截断，95/100 只回 5）、每次扣 1 个。
- 修改边界：不覆盖宿主生物的 AI、刷新与生成（Wiki 待建）；不覆盖隐士蟹好感与任务链本体；拆解只列实际返回这两种材料的条目。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `cookiecuttershell` | 单文件单注册 `Prefab("cookiecuttershell", fn, assets)`。公共端：`CreateEntity` → Transform／AnimState／SoundEmitter／Network → `MakeInventoryPhysics` → bank `cookiecutter_shell`／build `cookiecuttershell`／`PlayAnimation("idle")` → `MakeInventoryFloatable(inst)` → `SetPristine`，非主模拟在此返回。 | 主模拟 5 段接线：`tradable`（无参）、`stackable`（`maxsize = STACK_SIZE_MEDITEM`＝20）、`inspectable`、`inventoryitem`、`MakeHauntableLaunchAndSmash`。无 `fuel`／`burnable`／`repairer`。[fn](../../../raw/current/scripts/prefabs/cookiecuttershell.lua#L6-L40)、[组件段](../../../raw/current/scripts/prefabs/cookiecuttershell.lua#L28-L37) |
| `slurtle_shellpieces` | 同构单文件单注册 `Prefab("slurtle_shellpieces", fn, assets)`，无 SoundEmitter；bank／build 都是 `slurtle_shellpieces`；浮水带缩放 `MakeInventoryFloatable(inst, "med", nil, 0.75)`。 | 主模拟：`inspectable`、`inventoryitem`、`stackable`（`STACK_SIZE_SMALLITEM`＝40）、`repairer`（`MATERIALS.SHELL`＋`TUNING.REPAIR_SHELL_HEALTH`）、`MakeHauntableLaunch`。没有 `tradable`／`fuel`／`burnable`。[fn](../../../raw/current/scripts/prefabs/slurtle_shellpieces.lua#L6-L41)、[组件段](../../../raw/current/scripts/prefabs/slurtle_shellpieces.lua#L27-L38) |

`MakeInventoryFloatable` 的无参路径是"尺寸 small、无垂直偏移、无缩放"，所以两件的浮水档不同：饼干壳取 small 档，碎片用 med＋0.75 缩放。[helper](../../../raw/current/scripts/standardcomponents.lua#L1772-L1791)

### 名称与实显

两件都是"注册名直接作名称键"：`entityscript` 用 `STRINGS.NAMES[string.upper(prefab)]` 填 `inst.name`，没有 `nameoverride`／`displaynamefn` 改写。[取名路径](../../../raw/current/scripts/entityscript.lua#L687-L692)、[赋名](../../../raw/current/scripts/entityscript.lua#L843)

- 饼干切割机壳：同版简中 `NAMES.COOKIECUTTERSHELL`＝**饼干切割机壳**（英文 `Cookie Cutter Shell`）。[po](../../../raw/current/scripts/languages/chinese_s.po#L341659-L341662)
- 外壳碎片：同版简中 `NAMES.SLURTLE_SHELLPIECES`＝**外壳碎片**（英文是 `Broken Shell`，不是"蜗牛壳碎片"或"贝壳片"）。[po](../../../raw/current/scripts/languages/chinese_s.po#L349659-L349662)

## 数值与参数

| 项 | `cookiecuttershell` | `slurtle_shellpieces` | 证据 |
|---|---|---|---|
| 堆叠上限 | 20（`STACK_SIZE_MEDITEM`） | 40（`STACK_SIZE_SMALLITEM`） | [壳](../../../raw/current/scripts/prefabs/cookiecuttershell.lua#L30-L31)、[碎片](../../../raw/current/scripts/prefabs/slurtle_shellpieces.lua#L31-L32)、[tuning](../../../raw/current/scripts/tuning.lua#L82-L83) |
| 浮水 | 无参 → small 档 | med 档、缩放 0.75 | [壳](../../../raw/current/scripts/prefabs/cookiecuttershell.lua#L20)、[碎片](../../../raw/current/scripts/prefabs/slurtle_shellpieces.lua#L19) |
| 交易 | `tradable` 无参：`goldvalue` 默认 0，无 `rocktribute`／`tradefor` | 无 `tradable` | [组件默认](../../../raw/current/scripts/components/tradable.lua#L1-L7)、[猪王门禁](../../../raw/current/scripts/prefabs/pigking.lua#L598-L606)、[蚁狮门禁](../../../raw/current/scripts/prefabs/antlion.lua#L115-L119) |
| 修补 | 无 `repairer` | 材料 `shell`、`healthrepairvalue`＝20，挂 `repairer`／`health_shell` 标签 | [组件段](../../../raw/current/scripts/prefabs/slurtle_shellpieces.lua#L34-L36)、[REPAIR_SHELL_HEALTH](../../../raw/current/scripts/tuning.lua#L2123)、[标签生成](../../../raw/current/scripts/components/repairer.lua#L8-L24) |
| 作祟 | `MakeHauntableLaunchAndSmash`：本版只剩抛起（砸碎片段被注释） | `MakeHauntableLaunch`：必抛起 | [壳 helper](../../../raw/current/scripts/standardcomponents.lua#L1071-L1105)、[碎片 helper](../../../raw/current/scripts/standardcomponents.lua#L1053-L1069) |
| 燃料／可燃 | 都无 | 都无 | 两文件全文 |

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| `组件：stackable` | 饼干壳 20、碎片 40 的同堆合并；`maxsize` 由本文件显式写入，不是组件默认 | 主模拟挂载；跨堆合并沿用组件规则 | [[reference/components/stackable#关键接口]]；[壳](../../../raw/current/scripts/prefabs/cookiecuttershell.lua#L30-L31)、[碎片](../../../raw/current/scripts/prefabs/slurtle_shellpieces.lua#L31-L32) |
| `组件：tradable` | 只挂空组件（`goldvalue = 0`），本版没有任何消费者放行 | 主模拟；猪王要求 `goldvalue > 0`，蚁狮要求 `rocktribute > 0`，两处都不成立 | [[reference/components/tradable#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/cookiecuttershell.lua#L28)、[猪王](../../../raw/current/scripts/prefabs/pigking.lua#L598-L606)、[蚁狮](../../../raw/current/scripts/prefabs/antlion.lua#L115-L119) |
| `组件：repairer`（物品侧） | 外壳碎片登记 `MATERIALS.SHELL` 与 `healthrepairvalue = 20`；组件按值给物品挂 `health_shell` | 主模拟；改数量要同时看 `tuning.REPAIR_SHELL_HEALTH` 与目标侧回复分支 | [[reference/components/repairer#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/slurtle_shellpieces.lua#L34-L36) |
| `组件：repairable`（目标侧） | 目标材料必须同为 `shell`，且走 `health` 分支：满血返回 false，否则 `DoDelta(20)`（按生命上限截断）；成功后有堆叠则只扣 1 个 | 服务端；动作侧还要求目标带 `repairable_shell`＋`healthrepairable`，物品带 `health_shell` | [[reference/components/repairable#关键接口]]；[材料与回复](../../../raw/current/scripts/components/repairable.lua#L106-L166)、[扣 1 个](../../../raw/current/scripts/components/repairable.lua#L172-L176)、[动作门禁](../../../raw/current/scripts/componentactions.lua#L1688-L1698) |
| `组件：hauntable` | 饼干壳用 `LaunchAndSmash`、碎片用 `Launch`，本版两者实际都只把物品抛起并复位 `is_landed` | 主模拟；`LaunchAndSmash` 的砸碎片段整段被 `--#HAUNTFIX` 注释，起名字面不能当行为 | [[reference/components/hauntable#关键接口]]；[LaunchAndSmash](../../../raw/current/scripts/standardcomponents.lua#L1071-L1105) |
| `API／数据：LootDropper 机会表` | 本页来源中五个宿主各绑一张机会表（饼干切割机、蛞蝓龟、蜗牛龟、蛞蝓龟窝、贝壳钟）；蛞蝓龟与蜗牛龟再叠一条 `AddIfNotChanceLoot` 兜底 | 服务端 `GenerateLoot`：`chance >= 1.0` 必进、`< 1.0` 逐行 `math.random() <= GetChance(chance)`，任一中签即置 `droppingchanceloot`，兜底只在全未中签时补 | [[reference/components/lootdropper#关键接口]]；[掷骰](../../../raw/current/scripts/components/lootdropper.lua#L213-L227)、[兜底](../../../raw/current/scripts/components/lootdropper.lua#L229-L234) |
| `关联 Prefab：boat_bumper_shell` | 唯一的 `repairable_shell` 目标：可被外壳碎片回血；锤拆时按剩余血量掉 1–2 个碎片 | 服务器结构；`testvalidrepairfn` 先由 `IsAboveGroundAtPoint` 放行；只有它未放行且 `IsVisualGroundAtPoint` 为真时，才检索半径 1 内可见、无 `placer`、无父节点且 X／Z 同格（取整相等）的玩家，命中则拒绝；回血按上限截断（95/100 只回 5） | Wiki 待建；[SHELL 数据行](../../../raw/current/scripts/prefabs/boat_bumpers.lua#L355-L363)、[repairable 挂载](../../../raw/current/scripts/prefabs/boat_bumpers.lua#L264-L267)、[锤拆回收](../../../raw/current/scripts/prefabs/boat_bumpers.lua#L181-L187)、[有效修补点](../../../raw/current/scripts/prefabs/boat_bumpers.lua#L59-L79) |
| `关联 Prefab：cookiecutterhat` | 饼干壳的第一条制作消费：4 壳＋1 绳，科学二档；帽体数值与战斗接线归帽族页 | 制作台（炼金引擎） | [[reference/prefabs/hats-armor]]；[配方](../../../raw/current/scripts/recipes.lua#L505) |
| `资源：anim` | 饼干壳 `anim/cookiecuttershell.zip`（bank `cookiecutter_shell`）；碎片 `anim/slurtle_shellpieces.zip` | 客户端加载；声明不等于资源可用性已验证 | [壳 assets](../../../raw/current/scripts/prefabs/cookiecuttershell.lua#L1-L4)、[碎片 assets](../../../raw/current/scripts/prefabs/slurtle_shellpieces.lua#L1-L4) |

## 获得链

### 饼干切割机壳

| 来源 | 产出 | 触发／条件 | 证据 |
|---|---|---|---|
| 饼干切割机（`cookiecutter`） | 0–2 个 | 机会表两条独立掷骰 0.50／0.25，另有 `monstermeat` 1.00 必掉；每掷按击杀者幸运修正 | [表](../../../raw/current/scripts/prefabs/cookiecutter.lua#L21-L26)、[挂表](../../../raw/current/scripts/prefabs/cookiecutter.lua#L186-L187)、[幸运](../../../raw/current/scripts/components/lootdropper.lua#L173-L184) |
| 瓶中信宝藏：沉底宝箱（`sunkenchest`） | 4–6 个（该预设保底） | `saltminer` 预设（`preset_weight = 3`）的 `guaranteed_loot`；宝箱由漂流瓶／海盗生成逻辑开出 | [预设](../../../raw/current/scripts/messagebottletreasures.lua#L40-L62)、[生成入口](../../../raw/current/scripts/components/messagebottlemanager.lua#L210) |
| 拆解回收 | 绿杖：钓具箱 2、超级钓具箱 3、珍珠的温泉 10；锤拆：珍珠的温泉 5 | 绿杖按表值全量（钓具箱／超级钓具箱是物品，只能绿杖）；锤拆按 `ceil(表值 × 0.5)`，烧毁态再降到 ×0.25 | [温泉 10](../../../raw/current/scripts/recipes.lua#L1562)、[钓具箱](../../../raw/current/scripts/recipes.lua#L1611-L1612)、[减半公式](../../../raw/current/scripts/components/lootdropper.lua#L134-L158)、[百分比](../../../raw/current/scripts/tuning.lua#L155-L156)、[绿杖全量](../../../raw/current/scripts/prefabs/staff_common.lua#L637-L674)、[温泉锤拆](../../../raw/current/scripts/prefabs/hermithotspring.lua#L527-L539) |

### 外壳碎片

| 来源 | 产出 | 触发／条件 | 证据 |
|---|---|---|---|
| 蛞蝓龟（`slurtle`） | 0–1 个（兜底） | 机会表为蛞蝓龟黏液×2（1.0）＋背壳头盔 0.1；两条 1.0 不置 `droppingchanceloot`，所以头盔没中签就补 1 个碎片；**普通燃烧中死亡时本条被换成 ash**（受控燃烧则保留原名、不闷烧），所以只有普通火要先灭 | [表](../../../raw/current/scripts/prefabs/slurtle.lua#L19-L24)、[接线](../../../raw/current/scripts/prefabs/slurtle.lua#L211-L212)、[可燃](../../../raw/current/scripts/prefabs/slurtle.lua#L180) |
| 蜗牛龟（`snurtle`） | 0–1 个（兜底） | 表为蛞蝓龟黏液×2（1.0）＋蜗壳护甲 0.75；护甲没中签时补 1 个；**普通燃烧中死亡同样换成 ash**（受控燃烧保留、不闷烧） | [表](../../../raw/current/scripts/prefabs/slurtle.lua#L26-L31)、[接线](../../../raw/current/scripts/prefabs/slurtle.lua#L251-L252) |
| 蛞蝓龟窝（`slurtlehole`） | 普通击杀 1 个；烧尽 0 个 | 表为蛞蝓龟黏液×3（1.0）＋碎片 1.0；普通击杀在同一个 `OnKilled` 里启动破坏动画并排 0.66 秒后的 `OnDoKilled`（计时起点是死亡回调，不是动画播完），由它 `DropLoot`；烧尽走 `explosive`，发出 `death` 后立刻 `Remove`，待执行任务被取消，掉落回调不执行 | [表](../../../raw/current/scripts/prefabs/slurtlehole.lua#L18-L24)、[延迟掉落](../../../raw/current/scripts/prefabs/slurtlehole.lua#L44-L55)、[爆炸接线](../../../raw/current/scripts/prefabs/slurtlehole.lua#L149-L162)、[移除取消任务](../../../raw/current/scripts/entityscript.lua#L1730-L1746) |
| 贝壳钟（`singingshell_octave*`，低音／中音／高音三档） | 1＋30% | 锤碎（`HAMMER`、`workleft = 1`）时 `onfinishwork` 掉表：1.0＋0.3 两条独立掷骰 | [表](../../../raw/current/scripts/prefabs/singingshell.lua#L18-L22)、[锤碎回调](../../../raw/current/scripts/prefabs/singingshell.lua#L74-L79)、[名称](../../../raw/current/scripts/languages/chinese_s.po#L349389-L349392) |
| 发光蟹（`lightcrab`） | 随机 1 件，权重 0.5 | `numrandomloot = 1`，权重表为小鱼块 0.25／荧光果 0.25／碎片 0.5（权重和 1，即 50%）；**普通燃烧中死亡同样换成 ash**（受控燃烧保留、不闷烧） | [接线](../../../raw/current/scripts/prefabs/lightcrab.lua#L103-L107)、[可燃](../../../raw/current/scripts/prefabs/lightcrab.lua#L112) |
| 巨大洞穴蠕虫（`worm_boss`） | 加权单件 ×3 次 | 该表以权重取值，碎片权重 2；每处掉落点连抽 3 次（YOTS 期间另有一次幸运金块） | [权重表](../../../raw/current/scripts/prefabs/worm_boss.lua#L96-L120)、[三连抽](../../../raw/current/scripts/prefabs/worm_boss.lua#L471-L482) |
| 石化树（`tree_rocks`）的藤蔓 | 0–5 个（每条命中的藤蔓位 1 个） | 藤蔓按所在区域加权抽 5 条；隐士海岸 3、泥地 5、岩地 5（同表另有石头／荧光果等），权重是相对值不是概率 | [权重](../../../raw/current/scripts/prefabs/tree_rock_data.lua#L69-L128)、[挑表](../../../raw/current/scripts/prefabs/tree_rocks.lua#L106-L123)、[抽 5 条](../../../raw/current/scripts/prefabs/tree_rocks.lua#L126-L146)、[逐位生成](../../../raw/current/scripts/prefabs/tree_rocks.lua#L615-L631) |
| 洞穴补给箱场景（`chest_cavesupplies`） | 3–8 个 | 场景 `OnCreate` 直接往箱里放 `math.random(3, 8)` | [场景](../../../raw/current/scripts/scenarios/chest_cavesupplies.lua#L14-L15) |
| 拆解回收 | 绿杖：沉底宝箱 5、贝壳保险杠 3＋绳 1；锤拆：沉底宝箱 3、贝壳保险杠按自身血量 1–2 | 绿杖按表值全量；锤拆按 `ceil(表值 × 0.5)`（沉底宝箱锤拆前先灭火）；贝壳保险杠的锤拆走自身 `onhammered`，不读 recipe 表 | [沉底宝箱](../../../raw/current/scripts/recipes.lua#L1602)、[保险杠](../../../raw/current/scripts/recipes.lua#L1656)、[减半](../../../raw/current/scripts/components/lootdropper.lua#L134-L158)、[宝箱锤拆](../../../raw/current/scripts/prefabs/treasurechest.lua#L49-L61)、[保险杠锤拆](../../../raw/current/scripts/prefabs/boat_bumpers.lua#L181-L187) |

**可燃宿主的改写边界**：`DropLoot` 按调用时的 `burnt`／燃烧与燃料条件进入改写：先查覆盖表与 `_cooked`／`cooked` 前后缀，受控燃烧保留尚未换名的条目（只有自身带 `burnable` 的掉落物才落地闷烧，门禁在[这里](../../../raw/current/scripts/components/lootdropper.lua#L398-L407)），最后才到非豁免宿主的 `ash` 分支（[改写分支](../../../raw/current/scripts/components/lootdropper.lua#L415-L445)、[落地闷烧](../../../raw/current/scripts/components/lootdropper.lua#L382-L407)、[死亡入口](../../../raw/current/scripts/entityscript.lua#L2181-L2192)）。所以上表三个宿主在普通火下要先灭火；贝壳钟、巨大洞穴蠕虫、饼干切割机没有 `burnable`，石化树走 `SpawnVineLoot` 且带 `tree` 标签；蛞蝓龟窝的烧尽另见上表；`hive` 结构反而照常换算（[#L438](../../../raw/current/scripts/components/lootdropper.lua#L438)）。

宿主显示名按同版 `.po` 核过：蛞蝓龟（`Slurtle`）、蜗牛龟（`Snurtle`）、蛞蝓龟窝（`Slurtle Mound`）、饼干切割机（`Cookie Cutter`）、发光蟹（`Crustashine`）、巨大洞穴蠕虫（`Great Depths Worm`）。[SLURTLE](../../../raw/current/scripts/languages/chinese_s.po#L349639-L349642)、[SNURTLE](../../../raw/current/scripts/languages/chinese_s.po#L349709-L349712)、[SLURTLEHOLE](../../../raw/current/scripts/languages/chinese_s.po#L349649-L349652)、[COOKIECUTTER](../../../raw/current/scripts/languages/chinese_s.po#L341649-L341652)、[LIGHTCRAB](../../../raw/current/scripts/languages/chinese_s.po#L344714-L344717)、[WORM_BOSS](../../../raw/current/scripts/languages/chinese_s.po#L352459-L352462)

## 用途与配方

**饼干切割机壳**（`recipes.lua` 实际行）：

| 消费端 | 消耗 | 科技／门禁 | 证据 |
|---|---|---|---|
| 饼干切割机帽子 `cookiecutterhat` | 4＋绳 1 | `TECH.SCIENCE_TWO`（炼金引擎） | [L505](../../../raw/current/scripts/recipes.lua#L505) |
| 雨天／雪天／麻醉／重量级鱼饵 ×4（`oceanfishinglure_hermit_*`） | 各 1（另配蚊囊／冰／毒刺／牛毛） | `TECH.LOST`；用隐士蟹给的鱼饵草图对自己使用（`TEACH`）学会配方后才做得出，也可以花 1 个空瓶在隐士蟹商店买成品 | [L661-L664](../../../raw/current/scripts/recipes.lua#L661-L664)、[商店同款](../../../raw/current/scripts/recipes.lua#L940-L943)、[草图教学](../../../raw/current/scripts/prefabs/tacklesketch.lua#L46-L48) |
| 隐士迁居套装 `hermitcrab_relocation_kit` | 4＋月岩块 4 | `TECH.LOST`＋`nounlock`，由隐士蟹工坊在满足进度条件时教出 | [L951](../../../raw/current/scripts/recipes.lua#L951)、[教学](../../../raw/current/scripts/prefabs/hermitcrab.lua#L373-L381) |
| 盐晶组合机 `shellweaver` | 5＋月光玻璃 5＋月岩块 3 | `TECH.LOST`＋`nounlock`＋`placer`：须在隐士蟹处制作，并放在珍珠区域 | [L973](../../../raw/current/scripts/recipes.lua#L973)、[放置判定](../../../raw/current/scripts/recipes.lua#L963-L965)、[教学](../../../raw/current/scripts/prefabs/hermitcrab.lua#L326-L331) |
| 隐士蟹房屋一级 `hermithouse_construction1` | 10＋木板 10＋萤火虫 1 | 建造计划（工地交料），不是制作栏配方 | [L1524](../../../raw/current/scripts/recipes.lua#L1524)、[收料](../../../raw/current/scripts/components/constructionsite.lua#L131-L150) |
| 温泉 `hermithotspring_constr`（建造计划，成品为珍珠的温泉） | 10＋石头 20＋藤壶 10＋铲 1 | 建造计划；需好感满级＋珍珠装饰分启用，装饰分 ≥60 首次解锁（跌破 50 收回） | [L1528](../../../raw/current/scripts/recipes.lua#L1528)、[解锁分档](../../../raw/current/scripts/prefabs/hermithouse.lua#L18-L25) |

**外壳碎片**：

| 消费端 | 消耗 | 科技／门禁 | 证据 |
|---|---|---|---|
| 灵魂容器 `graveurn` | 1＋灰烬 1 | `TECH.NONE`，但要温蒂技能 `wendy_makegravemounds` | [L160](../../../raw/current/scripts/recipes.lua#L160) |
| 种子袋 `seedpouch` | 2＋干草 4＋种子 2 | `TECH.SCIENCE_TWO` | [L516](../../../raw/current/scripts/recipes.lua#L516) |
| 贝壳保险杠套装 `boat_bumper_shell_kit` | 3＋绳 1 | `TECH.SEAFARING_ONE`（海料站） | [L628](../../../raw/current/scripts/recipes.lua#L628) |
| 贝壳海滩地皮 `turf_shellbeach` | 1＋石头 1 → 4 张 | `TECH.LOST`；需先用 3 个空瓶向隐士蟹换 `turf_shellbeach_blueprint` 学会 | [L704](../../../raw/current/scripts/recipes.lua#L704)、[图纸](../../../raw/current/scripts/recipes.lua#L934) |
| 蕨类盆栽 `pottedfern` | 2 叶片＋1 | `TECH.SCIENCE_TWO`＋`placer` | [L735](../../../raw/current/scripts/recipes.lua#L735) |
| 珍珠的灯柱 `hermitcrab_lightpost` | 2＋石头 3＋藤壶 2 | `TECH.LOST`＋`nounlock`；需好感满级＋珍珠装饰分启用，装饰分 ≥10 首次解锁（跌破 5 收回） | [L974](../../../raw/current/scripts/recipes.lua#L974)、[分档](../../../raw/current/scripts/prefabs/hermithouse.lua#L18-L25) |
| 风铃 `hermithouse_ornament` | 1＋树枝 1 | `TECH.LOST`＋`nounlock`；需好感满级＋珍珠装饰分启用，且房屋容器建好后才解锁 | [L977](../../../raw/current/scripts/recipes.lua#L977)、[解锁](../../../raw/current/scripts/prefabs/hermithouse.lua#L256-L260) |
| 搬运海螺 `hermitcrab_shell` | 3＋盐晶 3＋传送护符 1 | `TECH.SHELLWEAVER_THREE`，须在二级组合机制造 | [L988](../../../raw/current/scripts/recipes.lua#L988)、[二级树](../../../raw/current/scripts/tuning.lua#L1410-L1415) |

**门禁怎么读**（三条一起看，缺一条都会误判）：

1. `TECH.LOST` 是 `{MAGIC=10, SCIENCE=10, ANCIENT=10}`，远超建筑给到的科技加成，所以"攒够科技"永远解锁不了这些行。[LOST](../../../raw/current/scripts/constants.lua#L1324)、[科技比较](../../../raw/current/scripts/components/builder.lua#L896-L915)
2. 两条绕开科技的路必须分开看。本页 `TECH.LOST`＋`nounlock` 的几条（迁居套装、盐晶组合机、灯柱、风铃）科技不可达、又不产生"已解锁"记录（`UnlockRecipe` 直接跳过、蓝图抽取也排除），只能由制作站 `GetRecipes` 临时供货；不带 `nounlock` 的 `TECH.LOST` 配方相反：蓝图／草图经 `teacher` 直接 `UnlockRecipe` 写入 `builder.recipes`；`KnowsRecipe` 先查列表，命中即返回。[跳过解锁](../../../raw/current/scripts/components/builder.lua#L463-L473)、[teaching](../../../raw/current/scripts/components/teacher.lua#L10-L28)、[蓝图排除](../../../raw/current/scripts/prefabs/blueprint.lua#L44-L48)、[先查列表](../../../raw/current/scripts/components/builder.lua#L883-L895)、[站内配方](../../../raw/current/scripts/components/builder.lua#L276-L298)
3. 隐士蟹线四条入口各不相同：诱饵靠鱼饵草图（对自己使用 `TEACH` 学会）或商店成品，地皮靠商店的图纸（同样走 `teacher` 教学）；迁居套装与盐晶组合机由隐士蟹直接 `LearnItem`；灯柱／风铃要先满足「隐士蟹好感满级且珍珠装饰分启用」，再分别看装饰分（灯柱首解 10、跌破 5 收回）与房屋容器（风铃）；盐晶组合机的二级成品要装饰分 ≥40 首次升到 `SHELLWEAVER = 3`（跌破 30 才降回一级）。[草图](../../../raw/current/scripts/components/tacklesketch.lua#L13-L18)、[教学两处](../../../raw/current/scripts/prefabs/hermitcrab.lua#L326-L331)、[好感与装饰分前置](../../../raw/current/scripts/prefabs/hermithouse.lua#L246-L267)、[灯柱分档](../../../raw/current/scripts/prefabs/hermithouse.lua#L18-L24)、[二级树](../../../raw/current/scripts/prefabs/shellweaver.lua#L127-L155)

## 典型接入

### 让自定义结构接受外壳碎片修补

- 前提与选择：服务端 `modmain.lua`；目标须已有 `health`，材料须等于 `MATERIALS.SHELL`（`"shell"`）；`REPAIR` 的客户端预判只认 `repairable_<材料>`＋`healthrepairable` 标签。回调在回血与扣料之后执行，副作用要判空。
- 生效与撤销：追加发生在 prefab 构造后；`health` 的每次赋值都会把 `healthrepairable` 同步成"未满血"，所以不必手动维护该标签。移除模组即恢复原状。

```lua
-- modmain.lua：让自定义结构能像贝壳保险杠一样被外壳碎片修
AddPrefabPostInit("my_structure", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    if inst.components.health == nil or inst.components.repairable ~= nil then
        return
    end
    local repairable = inst:AddComponent("repairable")
    repairable.repairmaterial = GLOBAL.MATERIALS.SHELL
    repairable.onrepaired = function(target)
        if target.SoundEmitter ~= nil then
            target.SoundEmitter:PlaySound("dontstarve/common/place_structure_stone")
        end
    end
end)
```

- 调整与验证：`REPAIR_SHELL_HEALTH`＝20 是请求值，最终按目标生命上限截断（95/100 只回 5），每次消耗 1 个碎片；满血时 `REPAIR` 动作不出现。示例未装站位门禁（`testvalidrepairfn`）；用「有 `health`、无 `SoundEmitter`、血量未满」的结构照抄，可完成一次修补与一次扣料。本例为源码推导，未运行游戏。

## 常见陷阱与限制

- **误区：`AddIfNotChanceLoot` 是"必掉一件"。** 它只在"本次机会表中所有 `< 1.0` 的条目都没中签"时补一件；表里的 `1.0` 条目不算中签，所以蛞蝓龟的黏液每次都进背包、却不会阻止碎片兜底。反过来说，头盔真掉了就没有碎片。[判定](../../../raw/current/scripts/components/lootdropper.lua#L213-L234)
- **边界：`droppingchanceloot` 是组件实例字段，不会在每次 `GenerateLoot` 前重置。** 同一实体第二次生成掉落时，之前中过签的记录仍在，兜底不会再补。对蛞蝓龟这类一次性死亡的目标没有实际影响，但给可反复掉落的自定义实体接线时要自己清。[初始化](../../../raw/current/scripts/components/lootdropper.lua#L9)、[置位](../../../raw/current/scripts/components/lootdropper.lua#L223)
- **误区：`tradable` 就等于能卖猪王。** 组件本身只有 `goldvalue = 0` 的默认值；猪王只放行 `goldvalue > 0`（或万圣糖果），蚁狮只放行 `rocktribute > 0`。饼干切割机壳两处都不满足，挂这个组件在本版没有可观察效果。[组件](../../../raw/current/scripts/components/tradable.lua#L1-L7)、[猪王](../../../raw/current/scripts/prefabs/pigking.lua#L598-L606)、[蚁狮](../../../raw/current/scripts/prefabs/antlion.lua#L115-L119)
- **误区：`MakeHauntableLaunchAndSmash` 会砸碎物品。** 名字里的砸碎片段在本版被 `--#HAUNTFIX` 整段注释，只剩 `Launch` 与 `is_landed` 复位；饼干壳和蛞蝓龟／蜗牛龟的作祟都只弹飞。[helper](../../../raw/current/scripts/standardcomponents.lua#L1071-L1105)
- **边界：贝壳保险杠的碎片回收绕过了 `DropLoot` 的烧毁改写。** 它没有 `structure` 标签，也从不调 `LootDropper:DropLoot`——锤拆回调里直接 `SpawnLootPrefab(data.loot)`（按剩余血量 1–2 个），所以 `DropLoot` 那条"带 `burnt` 或正在燃烧时把非结构目标掉落换成 `ash`"的分支不经过它。[锤拆](../../../raw/current/scripts/prefabs/boat_bumpers.lua#L181-L187)、[改写分支](../../../raw/current/scripts/components/lootdropper.lua#L415-L445)
- **边界：`slurtlehole` 的 1 个碎片只在普通击杀时兑现。** 普通击杀在 `OnKilled` 里排 0.66 秒后的 `OnDoKilled`，由它 `DropLoot`；烧尽时 `explosive` 发 `death` 后立即 `Remove`，`CancelAllPendingTasks` 让该回调不再跑，所以不产碎片。[L44-L55](../../../raw/current/scripts/prefabs/slurtlehole.lua#L44-L55)、[移除顺序](../../../raw/current/scripts/components/explosive.lua#L151-L157)、[取消任务](../../../raw/current/scripts/entityscript.lua#L1730-L1746)
- **待核验：沉底宝箱的瓶中信来源概率。** 本页只核到 `saltminer` 预设内容与 `preset_weight = 3`，未核算漂流瓶的掉落率与宝箱预设抽取权重。

## 验证记录

- 源码审阅：2026-11-10，Steam buildid 26474873。核验范围：两份 prefab 全文与名称键；`cookiecutter`／`slurtle`／`snurtle`／`slurtlehole`／`singingshell`／`lightcrab`／`worm_boss`／`tree_rocks`／`chest_cavesupplies`／`messagebottletreasures` 的掉落表与触发动作；`lootdropper` 的掷骰、兜底、幸运与烧毁改写；`repairer`／`repairable`／`componentactions` 的修补链路与 `health` 标签同步；`tradable`／猪王／蚁狮门禁；`recipes.lua` 的制作、建造计划与拆解行，以及 `builder`／`craftingstation`／`blueprint`／`teacher`／`tacklesketch`／`hermitcrab`／`hermithouse`／`shellweaver` 的解锁路径；`staff_common` 的绿杖拆解与 `GetRecipeLoot` 的锤拆折半、宿主的可燃接线与 `DropLoot` 烧毁改写（含受控燃烧分支）、`slurtlehole` 的延迟回调与爆炸移除、保险杠的站位门禁与生命上限截断。未覆盖：各宿主自己的刷新、生成与 AI；隐士蟹好感与任务链本体（只核到 `LearnItem` 调用点）；瓶中信掉落率；运行表现。
- 游戏运行：未运行；数量、概率与修补量均为源码推导。
- 结构检查：`validate_page.py`、`validate_source_links.py` 与 `modmain` 片段的 Lua 语法检查（luaparser）结果见交付回报。

相关：[[reference/prefabs/hats-armor]]（饼干切割机帽子）、[[reference/prefabs/moonrocknugget]]（隐士迁居套装与盐晶组合机的月岩端）、[[reference/prefabs/pouch-bags]]（种子袋）、[[reference/prefabs/ashes-and-charcoal]]（灵魂容器的灰烬端）、[[reference/prefabs/living-tools]]（刷子，钢丝绵消费）、[[reference/components/topics/world-ships]]（海上设施与船具）。

返回 [[reference/prefabs/index|原版 Prefab 索引]]
