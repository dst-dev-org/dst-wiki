---
title: "月树与月树苗（moon_tree、moon_tree_short、moon_tree_normal、moon_tree_tall）"
created: "2026-11-09"
updated: "2026-11-10"
type: prefab
size_class: complex
summary: "prefabs/moontree.lua 一个工厂注册四次、对应三个尺寸（通用入口随机）：三阶 growable（可循环回矮树）、8／12／16 次砍伐与三张 loot 表（木头＋月树花＋活体月蛾）、砍倒成树桩／被烧成木炭树、30 天腐烂计时清理地面掉落，以及 plantregrowth 产月树苗、desolationspawner 与月岛 worldgen 的再生链。"
aliases: ["moon_tree", "moon_tree_short", "moon_tree_normal", "moon_tree_tall", "月树", "砍月树", "月树掉落", "月亮树", "moonbutterfly_sapling", "月树苗", "月蛾种的月树苗", "baby_moon_tree", "月树桩"]
tags: [prefab]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/moontree.lua", "prefabs/planted_tree.lua", "prefabs/moonbutterfly.lua", "prefabs/waxed_plants.lua", "prefabs/waxed_plant_common.lua", "components/growable.lua", "components/plantregrowth.lua", "components/simplemagicgrower.lua", "components/timer.lua", "components/lootdropper.lua", "components/waxable.lua", "components/desolationspawner.lua", "standardcomponents.lua", "simutil.lua", "regrowthutil.lua", "worldsettings_overrides.lua", "map/rooms/forest/terrain_moonisland.lua", "map/static_layouts/moontrees_2.lua", "map/layouts.lua", "mainfunctions.lua", "prefabs/flower.lua", "fx.lua", "tuning.lua", "languages/chinese_s.po", "screens/redux/scrapbookdata.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 月树与月树苗（moon_tree、moon_tree_short、moon_tree_normal、moon_tree_tall）

## 快速判断

- 职责与适用：月岛的树，一个工厂 `moon_tree(name, stage)` 注册四次，四个注册共享同一套动画与逻辑，只靠 `stage` 决定初始尺寸；`moon_tree` 是"随机尺寸"的通用入口，另外三个是显式尺寸。[工厂](../../../raw/current/scripts/prefabs/moontree.lua#L555-L667)、[注册](../../../raw/current/scripts/prefabs/moontree.lua#L669-L672)
- 优先使用：查长多大、怎么长看[生长](#生长三阶与循环)；查砍几刀、掉什么看[砍伐与战利品](#砍伐与战利品)；查砍完／烧完怎么收场看[树桩、烧毁与腐烂计时](#树桩烧毁与腐烂计时)；查月树苗看[月树苗](#月树苗moonbutterflysapling)。
- 重要副作用：`growable.loopstages = true`，所以**高月树长满后会长回矮树**（动画 `grow_tall_to_short`），不是停在最高阶；砍倒与烧毁都会启动一个 30±15 天的 `decay` 计时，到点先清掉 6 格内多余的 `log`／`charcoal` 再自删。[阶段推进](../../../raw/current/scripts/components/growable.lua#L93-L101)、[循环动画](../../../raw/current/scripts/prefabs/moontree.lua#L310-L315)、[清理](../../../raw/current/scripts/prefabs/moontree.lua#L437-L462)
- 修改边界：`planted_tree.lua` 的公共 sapling 工厂（`timer`／`growprefab`／`DIG`／`MakeWaxablePlant`）不属本批，本页只写月树苗的接入点与差异，公共逻辑留给 B14；蜡封形态 `moon_tree_waxed` 归 B43 `waxed_plant_common`，本页只写入口。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `moon_tree` | `moon_tree("moon_tree", 0)`：`fn` 直构 → Transform／AnimState／音效／小地图／网络 → `MakeObstaclePhysics(inst, .5)`＋部署半径 → 标签 `plant`／`tree`／`shelter` → bank／build `moon_tree` → `SetPrefabName("moon_tree")`＋标签 `moon_tree` → 采集册字段 → `MakeSnowCoveredPristine` → `SetPristine` → 非主模拟返回 | `stage = 0`：主模拟里 `growable:SetStage(math.random(1, 3))`，**每次生成随机矮／中／高** | [主体](../../../raw/current/scripts/prefabs/moontree.lua#L596-L630)、[注册](../../../raw/current/scripts/prefabs/moontree.lua#L669-L669) |
| `moon_tree_short` | 同 `fn`，`stage = 1` | 矮月树：8 次砍（`MOON_TREE_CHOPS_SMALL`）、`moontree_small` 表、小号可燃与蔓延 | [砍数](../../../raw/current/scripts/prefabs/moontree.lua#L299-L308)、[常量](../../../raw/current/scripts/tuning.lua#L3868-L3868) |
| `moon_tree_normal` | 同 `fn`，`stage = 2` | 中月树：12 次砍、`moontree_normal` 表 | [设置](../../../raw/current/scripts/prefabs/moontree.lua#L342-L351)、[常量](../../../raw/current/scripts/tuning.lua#L3869-L3869) |
| `moon_tree_tall` | 同 `fn`，`stage = 3` | 高月树：16 次砍、`moontree_tall` 表、大号可燃（`SetFXLevel(5)`、燃烧时长 ×1.5） | [设置](../../../raw/current/scripts/prefabs/moontree.lua#L385-L394)、[可燃](../../../raw/current/scripts/prefabs/moontree.lua#L360-L383) |

四个注册共有的固定项（都在 `fn` 里，不分尺寸）：随机染色 `SetMultColour(0.75–1.0)`、小地图图标 `moon_tree.png`（优先级 −1）、`scrapbook_specialinfo = "TREE"` 与 `scrapbook_proxy = "moon_tree_tall"`、`OnSave`／`OnLoad`、`OnEntitySleep`／`OnEntityWake`、`MakeHauntableWorkAndIgnite`、`MakeWaxablePlant`、`MakeSnowCovered`。[固定项](../../../raw/current/scripts/prefabs/moontree.lua#L555-L664)

### 名称核验

`NAMES.MOON_TREE`＝**月树**（英文 Lune Tree）。`moon_tree` 这个通用入口与 `moon_tree_short`／`moon_tree_normal`／`moon_tree_tall` 三个尺寸注册**都**靠 `inst:SetPrefabName("moon_tree")` 取该键；三个尺寸没有独立的 `NAMES` 键，所以检视、采集册与配方里看到的都是同一个名字，看不出尺寸。[改名](../../../raw/current/scripts/prefabs/moontree.lua#L577-L577)、[MOON_TREE](../../../raw/current/scripts/languages/chinese_s.po#L345870-L345872)

### 改名带来的两个后果

- **postinit 会命中两次**：`SpawnPrefab` 最终走 `SpawnPrefabFromSim`——先按 `inst.prefab`（已被改成 `moon_tree`）取一次 `PrefabPostInit`，再因为 `inst.prefab ~= name` 补一次注册名（如 `moon_tree_tall`）的 postinit。所以 `AddPrefabPostInit("moon_tree", ...)` 会同时作用到四个注册。[应用顺序](../../../raw/current/scripts/mainfunctions.lua#L387-L407)。**重载例外**：存档记的是改名后的 `moon_tree`，读档只按这个名字生成，`moon_tree_tall` 之类的注册名 postinit 不会跑在旧树上（尺寸由 `OnLoad` 的 `data.size` 恢复）。[存读](../../../raw/current/scripts/prefabs/moontree.lua#L464-L506)
- **按名字区分的逻辑只认 `moon_tree`**：`plantregrowth` 的 `SetSearchTag("moon_tree")` 与 `desolationspawner` 的 `{"moon_tree"}` 都按标签／类型名匹配，四个注册都有该标签。[标签](../../../raw/current/scripts/prefabs/moontree.lua#L577-L578)、[再生搜索](../../../raw/current/scripts/prefabs/moontree.lua#L637-L640)

## 生长：三阶与循环

时间基准：`seg_time`＝30 秒、`total_day_time`＝480 秒（S1），下表的"天"都按 480 秒换算。[基准](../../../raw/current/scripts/tuning.lua#L19-L20)

| 阶段 | 推进到下一阶的时长 | 进入该阶时执行 | 证据 |
|---|---|---|---|
| 1 矮（short） | `1.5 天 ± 0.5`（=1–2 天） | `set_short`：工作量 8、`moontree_small` 表、加 `shelter` 标签、摇摆 | [阶段表](../../../raw/current/scripts/prefabs/moontree.lua#L403-L411)、[常量](../../../raw/current/scripts/tuning.lua#L3864-L3864) |
| 2 中（normal） | `5 天 ± 2`（=3–7 天） | `set_normal`：工作量 12、`moontree_normal` 表 | [阶段表](../../../raw/current/scripts/prefabs/moontree.lua#L413-L419)、[常量](../../../raw/current/scripts/tuning.lua#L3865-L3865) |
| 3 高（tall） | `5 天 ± 2`（=3–7 天） | `set_tall`：工作量 16、`moontree_tall` 表 | [阶段表](../../../raw/current/scripts/prefabs/moontree.lua#L421-L427)、[常量](../../../raw/current/scripts/tuning.lua#L3866-L3866) |

关键点在最后一行之后：`loopstages = true` 让 `GetNextStage` 在最高阶之后回到第 1 阶，于是**高月树到期会长回矮树**——播放的是 `grow_tall_to_short`，`growfn` 只换燃烧参数与摇摆，尺寸由 `growable:SetStage` 里的 `set_short` 改回。[循环规则](../../../raw/current/scripts/components/growable.lua#L93-L101)、[推进逻辑](../../../raw/current/scripts/components/growable.lua#L121-L151)、[缩回动画](../../../raw/current/scripts/prefabs/moontree.lua#L310-L315)

其它生长开关：

- `springgrowth = true`：春季把时长乘 `SPRING_GROWTH_MODIFIER`＝0.75（更快）。[开关](../../../raw/current/scripts/prefabs/moontree.lua#L628-L628)、[修正函数](../../../raw/current/scripts/simutil.lua#L563-L565)、[常量](../../../raw/current/scripts/tuning.lua#L2284-L2284)
- `magicgrowable = true` ＋ `simplemagicgrower:SetLastStage(3)`：魔法催长一次可连跳到第 3 阶，用完清 `magicgrowth` 标签。[接线](../../../raw/current/scripts/prefabs/moontree.lua#L633-L634)、[组件](../../../raw/current/scripts/components/simplemagicgrower.lua#L9-L27)
- `growfromseed`：只做"种子长成矮树"的入场——`SetStage(1)`、播 `grow_seed_to_short`、补摇摆；它不动 `growable` 已有的计时任务（`SetStage` 只换阶段与外观）。[实现](../../../raw/current/scripts/prefabs/moontree.lua#L430-L435)
- 砍倒时 `make_stump` 会 `growable:StopGrowing()`，所以树桩不会再长回去。[停长](../../../raw/current/scripts/prefabs/moontree.lua#L214-L216)

## 砍伐与战利品

- **工具与动作**：`workable` 绑 `ACTIONS.CHOP`，`OnWorkCallback = on_chop_tree`（每次砍播 `chop_<size>` 再摇摆、在头顶生成 `tree_petal_fx_chop`、按体型调高度与缩放），`OnFinishCallback = on_chop_tree_down`。[工作区](../../../raw/current/scripts/prefabs/moontree.lua#L614-L618)、[每次砍](../../../raw/current/scripts/prefabs/moontree.lua#L145-L176)
- **刀数**：矮 8／中 12／高 16，由 `set_*` 写入；`moon_tree` 随机尺寸时按摇到的阶段取。[砍数](../../../raw/current/scripts/tuning.lua#L3868-L3870)
- **掉落时机**：只在**砍倒那一下**结算，不是每刀掉；`on_chop_tree_down` 按砍伐者相对镜头右向的位置决定向哪侧倒，并把 `DropLoot` 的落点偏到那一侧，随后 `make_stump`。[倒树](../../../raw/current/scripts/prefabs/moontree.lua#L231-L262)

| 表 | 条目（权重／概率） | 期望 |
|---|---|---|
| `moontree_small` | `log` 1.0 ×2、`moon_tree_blossom` 0.5 | 2 木头必给，花 50% |
| `moontree_normal` | `log` 1.0 ×3、`moon_tree_blossom` 1.0＋0.25、`moonbutterfly` 1.0 | 3 木头＋1 花必给，另 25% 第二朵花，1 只活体月蛾 |
| `moontree_tall` | `log` 1.0 ×4、`moon_tree_blossom` 1.0 ×2、`moonbutterfly` 1.0＋0.5 | 4 木头＋2 花＋1 月蛾，另 50% 第二只月蛾 |

三张表都是 `SetChanceLootTable` 的**概率表**：`chance >= 1.0` 直接给，其余逐条掷点，所以同一条目重复出现等于多掷几次。[小表](../../../raw/current/scripts/prefabs/moontree.lua#L43-L48)、[中表](../../../raw/current/scripts/prefabs/moontree.lua#L50-L58)、[高表](../../../raw/current/scripts/prefabs/moontree.lua#L60-L70)、[判定](../../../raw/current/scripts/components/lootdropper.lua#L213-L227)

**月蛾是活的掉落物**：表里的 `moonbutterfly` 是实体不是物品，掉出来就会走动、并按自己的鲜度表腐坏（见 [[reference/prefabs/moonbutterfly]]）；月树花的掉落物与数值见 [[reference/prefabs/moontree-blossom]]。

## 树桩、烧毁与腐烂计时

砍倒后进入三种收场之一，三者共用 `timer` 里的 `decay` 计时：

| 收场 | 触发 | 变化 | 证据 |
|---|---|---|---|
| 树桩（stump） | 砍倒时 `make_stump` | 摘 `burnable`／`propagator`／`workable`／`hauntable` 与 `shelter` 标签，按体型重挂可燃（小／中／大）＋中号蔓延＋`MakeHauntableIgnite`；去碰撞、加 `stump` 标签、`growable:StopGrowing`、小地图换 `moon_tree_stump.png`；重新挂 `workable` 为 `ACTIONS.DIG`、工作量 1 → `dig_up_stump` 掉木头后自删（矮 1 根、中／高 2 根） | [make_stump](../../../raw/current/scripts/prefabs/moontree.lua#L199-L229)、[挖桩](../../../raw/current/scripts/prefabs/moontree.lua#L178-L186) |
| 站立树被烧毁（burnt） | `burnable` 的 `SetOnBurntFn`＝`on_moon_tree_burnt` | **烧尽回调里立即**：播 `burnt_<size>` 循环、小地图换 `moon_tree_burnt.png`、加 `burnt` 标签。**从烧尽回调起 0.5 秒后**（不是点火后）才 `burnt_changes`：熄灭并摘 `burnable`／`propagator`／`growable`，改挂 `MakeHauntableWork`；摘 `shelter`；掉落表切 `moontree_burnt`（只有 `charcoal` 1.0）；`workable` 保留（动作仍是 `CHOP`）但改成 1 次、完成回调换成 `chop_down_burnt_moon_tree`。**这 0.5 秒内若原砍伐回调先完成**（树先被砍倒），仍按**原树表**结算并变成树桩——但因为 `burnt` 标签已经加上，通用 `GenerateLoot` 的 40% 追加仍可能多给 1 块木炭 | [立即段](../../../raw/current/scripts/prefabs/moontree.lua#L124-L133)、[延时入口](../../../raw/current/scripts/prefabs/moontree.lua#L117-L137)、[burned_changes](../../../raw/current/scripts/prefabs/moontree.lua#L95-L115)、[40% 追加](../../../raw/current/scripts/components/lootdropper.lua#L252-L254) |
| 烧毁站立树再砍一下 | `chop_down_burnt_moon_tree` | 1 次 `CHOP`：摘 `workable`、播 `chop_burnt_<size>`、去碰撞、`animover` 时 `Remove`，先 `DropLoot`——`moontree_burnt` 保底 1 块木炭，通用 `GenerateLoot` 又对带 `burnt` 标签的实体按 40% 追加 1 块，所以是 **1–2 块** | [砍烧树](../../../raw/current/scripts/prefabs/moontree.lua#L78-L93)、[表](../../../raw/current/scripts/prefabs/moontree.lua#L72-L75)、[40% 追加](../../../raw/current/scripts/components/lootdropper.lua#L252-L254) |
| 树桩被烧毁 | `make_stump_burnable` 重挂的 `MakeSmall/Medium/LargeBurnable` | 它们的 `OnBurntFn` 是 **`DefaultBurntFn`**（不是 `on_moon_tree_burnt`）：在脚下生成 1 个 `ash` 并 `Remove` 本体，**不会**留下可挖出木炭的烧焦树桩，也不进 `burnt_changes`。唯一例外是上面那条 0.5 秒竞态——先被砍成树桩、随后延时任务仍套上来时，那棵树桩才会带 `chop_down_burnt_moon_tree` 回调（动作已是 `DIG`） | [树桩可燃](../../../raw/current/scripts/prefabs/moontree.lua#L188-L197)、[DefaultBurntFn](../../../raw/current/scripts/standardcomponents.lua#L18-L44)、[小／中／大 Burnable 都接它](../../../raw/current/scripts/standardcomponents.lua#L164-L198) |

`decay` 计时（30 天 ± 50%，即 15–45 天）在"第一次变成树桩或烧毁"时启动，已存在就不重起；到点时先在自身 6 格内**留下一个** `log`／`charcoal`、其余同类全部删除，再 `inst:Remove()`——这是原版控制地面物品总数的清理逻辑。[起表](../../../raw/current/scripts/prefabs/moontree.lua#L130-L132)、[树桩起表](../../../raw/current/scripts/prefabs/moontree.lua#L226-L228)、[到点清理](../../../raw/current/scripts/prefabs/moontree.lua#L437-L462)、[常量](../../../raw/current/scripts/tuning.lua#L3856-L3860)

其它与尺寸／状态相关的行为：

- **可燃参数按尺寸**：矮用 `TREE_BURN_TIME / 2`、中 `TREE_BURN_TIME`、高 `TREE_BURN_TIME × 1.5`，FX 级别 2／3／5；蔓延半径 5／5／7、热输出 5／5／8.5、伤害半径 2／2／3。[矮](../../../raw/current/scripts/prefabs/moontree.lua#L274-L297)、[中](../../../raw/current/scripts/prefabs/moontree.lua#L317-L351)、[高](../../../raw/current/scripts/prefabs/moontree.lua#L360-L394)、[燃烧时长](../../../raw/current/scripts/tuning.lua#L1685-L1685)
- **离屏睡眠会临时摘组件**：`on_sleep` 摘 `burnable`／`propagator`／`inspectable`（若在燃烧还会摘 `growable` 并加 `burnt`），`on_wake` 再按尺寸把可燃与蔓延挂回去；树桩醒来时用中号蔓延。[睡](../../../raw/current/scripts/prefabs/moontree.lua#L508-L521)、[醒](../../../raw/current/scripts/prefabs/moontree.lua#L523-L553)
- **存读**：`OnSave` 写 `burnt`／`stump`／`size`；`OnLoad` 先按尺寸重挂可燃与掉落表，再重建树桩／烧毁态（`instant` 路径不出演出）。[存](../../../raw/current/scripts/prefabs/moontree.lua#L464-L474)、[读](../../../raw/current/scripts/prefabs/moontree.lua#L476-L506)
- **蜡封**：四个注册都调 `MakeWaxablePlant`，喷蜡后由 `WaxPlant` 换成 `moon_tree_waxed`（映射 key `moon_tree`）；蜡封形态的构造与行为归 B43 `waxed_plant_common`，本页只记入口。[入口](../../../raw/current/scripts/prefabs/moontree.lua#L649-L649)、[替换函数](../../../raw/current/scripts/prefabs/waxed_plant_common.lua#L543-L566)、[映射表](../../../raw/current/scripts/prefabs/waxed_plants.lua#L798-L812)

## 月树苗（moonbutterfly_sapling）

月树苗不在 `moontree.lua`，而是与其它 7 个 sapling 同用 `planted_tree.lua` 的 `sapling_fn` 工厂（文件末尾按同一套参数注册 8 个）；它和月树是"种下 → `growprefab`"的接续关系。**归属**：本页承接 `moonbutterfly_sapling` 的主注册与生长／再生链，`planted_tree.lua` 只提供共用 sapling 工厂（公共工厂逻辑归 B14）。[注册](../../../raw/current/scripts/prefabs/planted_tree.lua#L188-L188)

| 项目 | 值／行为 | 证据 |
|---|---|---|
| 外观与标签 | bank／build `baby_moon_tree`、播 `idle`；加 `plant` 与 `moon_tree` 标签（`fireproof` 为空，所以可烧） | [工厂](../../../raw/current/scripts/prefabs/planted_tree.lua#L127-L137)、[参数](../../../raw/current/scripts/prefabs/planted_tree.lua#L188-L188) |
| 生长 | `timer` 的 `grow` 计时：`growtimes` 未传 → 用 `TUNING.PINECONE_GROWTIME` 的 `0.75 天 ± 0.25`（=0.5–1 天）；到点 `growtree` 生成 `moon_tree_short`、调它的 `growfromseed()` 再删自己 | [起表](../../../raw/current/scripts/prefabs/planted_tree.lua#L86-L93)、[换树](../../../raw/current/scripts/prefabs/planted_tree.lua#L72-L80)、[常量](../../../raw/current/scripts/tuning.lua#L802-L802) |
| 挖掘 | `workable` 为 `ACTIONS.DIG`、工作量 1，完成时 `DropLoot`（默认 `{"twigs"}`）后自删 | [挖](../../../raw/current/scripts/prefabs/planted_tree.lua#L156-L162)、[digup](../../../raw/current/scripts/prefabs/planted_tree.lua#L101-L104) |
| 燃烧 | `MakeSmallBurnable`＋`MakeSmallPropagator`，点火时 `stopgrowing`、熄灭时 `startgrowing`；`MakeHauntableIgnite` | [接线](../../../raw/current/scripts/prefabs/planted_tree.lua#L164-L173) |
| 来源 | 月蛾放生（`OnDeploy` 生成 1 株）、月树的 `plantregrowth` 产物、月亮药水打在花上（`flower.lua` 的变异目标是它） | [月蛾放生](../../../raw/current/scripts/prefabs/moonbutterfly.lua#L69-L76)、[月树再生](../../../raw/current/scripts/prefabs/moontree.lua#L637-L640)、[花侧](../../../raw/current/scripts/prefabs/flower.lua#L190-L191) |

**两个"月树苗"别混**：简中显示名是"月树苗"的只有 `moonbutterfly_sapling`；挖出来的 `dug_sapling_moon` 显示的是"树苗"，两者不是同一实体，后者归 [[reference/prefabs/grass-saplings]]。[月树苗](../../../raw/current/scripts/languages/chinese_s.po#L345620-L345622)、[挖出件](../../../raw/current/scripts/languages/chinese_s.po#L342350-L342352)

## 世界生成与再生

| 渠道 | 行为 | 证据 |
|---|---|---|
| 月岛地形分布 | `MoonIsland_Forest` 里 `moon_tree` 权重 0.3、`moon_tree_blossom_worldgen` 0.2；`MoonIsland_Baths` 与 `MoonIsland_Meadows` 都是 `moon_tree` 1.0（草甸另有花 0.5） | [森林房](../../../raw/current/scripts/map/rooms/forest/terrain_moonisland.lua#L75-L91)、[浴场房](../../../raw/current/scripts/map/rooms/forest/terrain_moonisland.lua#L116-L139)、[草甸房](../../../raw/current/scripts/map/rooms/forest/terrain_moonisland.lua#L141-L162) |
| 静态布局 | `moontrees_2` 共 10 个 `tree_area`，每个区 90% 概率被指定为 `moon_tree` 区（10% 则该区不放树）；坐标物件里只有 1 个 `moon_tree_blossom_worldgen`，所以月岛树林来自这些树区而不是逐棵坐标 | [区域生成函数](../../../raw/current/scripts/map/layouts.lua#L1034-L1040)、[布局树区与花坐标](../../../raw/current/scripts/map/static_layouts/moontrees_2.lua#L54-L164) |
| 树旁再生（plantregrowth） | 每 `MOON_TREE_REGROWTH.OFFSPRING_TIME`＝3 天 ±20% 尝试一次，在自身半径内找候选落点生成 `moonbutterfly_sapling`；**候选落点搜索半径内带 `moon_tree` 标签的实体（含原有月树）已 ≥5 就不再生成**——不是"每块区域上限 5 株"。倍率乘在**计时推进量**上（`InternalTimes = InternalTimes + dt × TimeMultipliers[moon_tree]() × TUNING.REGROWTH_TIME_MULTIPLIER`），所以它决定"长得快慢"而不是"间隔乘数"：`TimeMultipliers[moon_tree]` ＝ `MOONTREE_REGROWTH_TIME_MULT` × 月相表（`new` 0、`quarter` 0.5、`half` 1.0、`threequarter` 1.5、`full` 2.0）——顺序是**先累加、再判 `nextregrowth <= InternalTimes`**；新月倍率 0 只是不再累加，已到期的那次检查仍会执行一次。**前置条件**：母树所在区域必须有该 prefab 的世界密度条目，否则第一次尝试就 `UnregisterUpdate` 放弃 | [速率](../../../raw/current/scripts/prefabs/moontree.lua#L637-L640)、[月相倍率](../../../raw/current/scripts/components/plantregrowth.lua#L14-L21)、[月相挂钩](../../../raw/current/scripts/components/plantregrowth.lua#L45-L47)、[推进量与到期检查](../../../raw/current/scripts/components/plantregrowth.lua#L62-L68)、[尝试点与密度门禁](../../../raw/current/scripts/components/plantregrowth.lua#L208-L242)、[GetFiveRadius](../../../raw/current/scripts/regrowthutil.lua#L9-L36) |
| 荒漠化重生（desolationspawner） | 该世界组件按类型 `moon_tree` 重生，产物同样是 `moonbutterfly_sapling`；重生间隔取 `EVERGREEN_REGROWTH.DESOLATION_RESPAWN_TIME`（50 天，与未在此文件引用的 `MOON_TREE_REGROWTH.DESOLATION_RESPAWN_TIME` 同值）；计时同样是**先按 `dt × REGROWTH_TIME_MULTIPLIER × prefabtimemult` 累加 `_internaltimes`，再比较 `regrowtime`**，其中 `prefabtimemult` ＝ `MOONTREE_REGROWTH_TIME_MULT` × 月相倍率，新月倍率 0 只暂停累加、已到期的检查仍可能执行一次，且每次 `LongUpdate` 最多补一处 | [重生登记](../../../raw/current/scripts/components/desolationspawner.lua#L192-L202)、[累加与到期](../../../raw/current/scripts/components/desolationspawner.lua#L228-L245)、[常量](../../../raw/current/scripts/tuning.lua#L3191-L3191) |

两处再生都只产**月树苗**，不直接产月树：苗再按 `PINECONE_GROWTIME` 长成 `moon_tree_short`，之后才由 `growable` 走三阶（并可能循环回矮树）。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：growable | 三阶短表＋`loopstages`／`springgrowth`／`magicgrowable`；`stage = 0` 时随机 1–3 | 主模拟；砍倒后 `StopGrowing` | [[reference/components/growable]]；[接线](../../../raw/current/scripts/prefabs/moontree.lua#L623-L631) |
| 组件：plantregrowth | 速率 3 天、产物 `moonbutterfly_sapling`、搜索标签 `moon_tree` | 主模拟；受月相倍率影响 | [[reference/components/plantregrowth]]；[接线](../../../raw/current/scripts/prefabs/moontree.lua#L637-L640) |
| 组件：simplemagicgrower | `SetLastStage(3)`，魔法催长可连跳到最高阶 | 主模拟 | [[reference/components/simplemagicgrower]]；[接线](../../../raw/current/scripts/prefabs/moontree.lua#L633-L634) |
| 组件：timer | `decay` 计时（30±15 天）＋ `timerdone` 监听做地面清理与自删 | 主模拟；树桩／烧毁共用一条 | [[reference/components/timer]]；[接线](../../../raw/current/scripts/prefabs/moontree.lua#L642-L644) |
| 组件：lootdropper | 三张概率表按尺寸切换；烧毁后切 `moontree_burnt` | 主模拟；切换发生在 `set_*` 与 `burnt_changes` | [[reference/components/lootdropper]]；[小表](../../../raw/current/scripts/prefabs/moontree.lua#L43-L48)、[切换](../../../raw/current/scripts/prefabs/moontree.lua#L108-L108) |
| 组件：waxable／waxable plant | `MakeWaxablePlant`：喷蜡换 `moon_tree_waxed` | 主模拟；蜡封形态归 B43 | [[reference/components/waxable]]；[接线](../../../raw/current/scripts/prefabs/moontree.lua#L649-L649) |
| 标签：`plant`／`tree`／`shelter`／`moon_tree` | 玩家可砍的树；`shelter` 在树桩／烧毁时摘掉；`moon_tree` 供再生按标签搜索 | 两端（pristine 前） | [标签](../../../raw/current/scripts/prefabs/moontree.lua#L571-L578) |
| API：`inst.growfromseed` | 供 sapling 工厂在长成矮树时调用 | 主模拟 | [赋值](../../../raw/current/scripts/prefabs/moontree.lua#L631-L631)、[调用方](../../../raw/current/scripts/prefabs/planted_tree.lua#L72-L80) |
| 关联 Prefab：`moon_tree_blossom` | 三张表的战利品 | 砍倒时结算 | [[reference/prefabs/moontree-blossom]]；[表](../../../raw/current/scripts/prefabs/moontree.lua#L50-L58) |
| 关联 Prefab：`moonbutterfly` | **中／高两张表**的活体战利品（矮树表没有） | 砍倒时生成 | [[reference/prefabs/moonbutterfly]]；[中表](../../../raw/current/scripts/prefabs/moontree.lua#L50-L58)、[高表](../../../raw/current/scripts/prefabs/moontree.lua#L60-L70) |
| 关联 Prefab：`moonbutterfly_sapling` | 再生与荒漠化重生的产物；由月蛾放生也可生成 | `planted_tree.lua` 的 sapling 工厂，公共逻辑归 B14 | [注册](../../../raw/current/scripts/prefabs/planted_tree.lua#L188-L188) |
| 关联 Prefab：`tree_petal_fx_chop` | 每次砍伐在树冠位置生成的碎屑 FX（bank／build `tree_petal_fx`） | 主模拟，纯表现 | [生成](../../../raw/current/scripts/prefabs/moontree.lua#L160-L175)、[注册](../../../raw/current/scripts/fx.lua#L462-L464) |
| 关联 Prefab：`moon_tree_waxed` | 蜡封形态，按 `moon_tree` 映射生成 | 喷蜡时替换 | **B43**；[映射](../../../raw/current/scripts/prefabs/waxed_plants.lua#L798-L812) |
| 资源：`anim/moon_tree.zip` | bank／build `moon_tree`，含三阶与 stump／burnt 动画；另有 `MINIMAP_IMAGE` `moon_tree`／`moon_tree_burnt`／`moon_tree_stump` | 原版资源，不随页分发 | [assets](../../../raw/current/scripts/prefabs/moontree.lua#L1-L6) |

### 共享规则接入（S1／S3／S7／S8／S10 适用；S2／S4／S5／S6／S9 不适用）

- **S1 时间基准**：生长 1.5／5／5 天、再生 3 天、腐烂 30 天、重生 50 天都按 `total_day_time`＝480 秒换算（[基准](../../../raw/current/scripts/tuning.lua#L19-L20)）。
- **S3 标签分层**：四个注册同一套标签；`moon_tree` 标签既是"树类"标记也是再生的搜索条件，和 `tree`／`shelter` 的职责不同，已分列。
- **S7 名称**：`NAMES.MOON_TREE` 存在（＝月树）；通用入口与三个尺寸注册都靠 `SetPrefabName("moon_tree")` 取该键，三个尺寸**没有独立 NAMES 键**。月树苗的显示名归 `moonbutterfly_sapling`（＝月树苗）。
- **S8 跨页归属**：`planted_tree.lua` 的公共 sapling 工厂（timer／growprefab／DIG／`MakeWaxablePlant`）留 B14；蜡封形态归 B43；月树花与月蛾各有本批页。
- **S10 待核验**：月树苗的生长时长用的是 `PINECONE_GROWTIME` 默认值（本页未核验是否有其它注入）、`growfromseed` 的动画衔接、`tree_petal_fx_chop` 的实际外观均只按源码推导。
- **S2／S4／S5／S6／S9 不适用**：不涉及钓鱼链、出水投射、重量归属、月亮转换；月树没有 `perishable`，S9 的鲜度口径不涉及本页（生长计时归 S1）。

## 典型接入

### 让自定义树复用月树的"树旁再生"

- 前提与选择：服务端 `modmain.lua`。要"砍完不补种也会自己冒苗"时优先复用 `plantregrowth`，但它有三道门，缺一道都不出苗：
  1. **时间倍率**：全局更新每 31 秒遍历 `InternalTimes` 的键并直接调 `TimeMultipliers[k]()`，键表是模块内局部表（默认只有 `evergreen`、`moon_tree` 等原版类型）；自定义 prefab 名没有条目就会报 `attempt to call a nil value`，并让这一轮分桶里排在后面的原版树／灌木再生一起被跳过。
  2. **世界密度**：`TrySpawnNearby` 先 `GetFiveRadius(x, z, inst.prefab)`，它读 `TheWorld.generated.densities[区域 id][prefab]`；没有该 prefab 的密度条目就 `UnregisterUpdate` 放弃、之后不再重试。候选落点也要按同一密度再算一次半径，同样要求该区域有密度。`SetSearchTag` 只决定最后数哪些实体，**不补密度**。
  3. **搜索标签与上限对象**：`SetSearchTag` 决定"数哪些实体来判 5 棵上限"。本页示例数的是**后代**——用 `moon_tree`（`moonbutterfly_sapling` 与长成的月树都带这个标签，母树不带），所以母树本身不计入；判定是"候选落点搜索半径内带 `moon_tree` 标签的实体（含原有月树）≥5 就不生成"，不是按区域配额。若改数母树，就得给自定义树加自己的标签、把上限对象换成母树（后代不计入），两者不能混写。
- 生效与撤销：`SetRegrowthRate` 在调用时把 prefab 名登记进分桶并排下一次时间；`SetProduct` 决定产物。密度要在世界生成期（或至少在该组件第一次尝试前）写好，示例给的是"对所有已生成区域补条目"的简化版。卸模组后组件消失，已生成的苗保留。
- 原版依据：[月树接线](../../../raw/current/scripts/prefabs/moontree.lua#L637-L640)、[注册与计时](../../../raw/current/scripts/components/plantregrowth.lua#L151-L161)、[更新循环与 nil 调用点](../../../raw/current/scripts/components/plantregrowth.lua#L62-L79)、[默认键表](../../../raw/current/scripts/components/plantregrowth.lua#L23-L59)、[模块导出 TimeMultipliers](../../../raw/current/scripts/components/plantregrowth.lua#L149-L149)、[密度门禁与尝试](../../../raw/current/scripts/components/plantregrowth.lua#L208-L242)、[GetFiveRadius](../../../raw/current/scripts/regrowthutil.lua#L9-L36)。

```lua
-- modmain.lua：给自定义树挂上月树同款"树旁再生"（产物是月树苗）
-- 1) 时间倍率：没有这一条，31 秒后的全局更新会直接调 nil
--    这里照月树的写法按月亮相位给倍率，验收的"满月更快、新月停"才有依据
local PlantRegrowth = GLOBAL.require("components/plantregrowth")
local MOON_PHASE_MULT = { new = 0, quarter = 0.5, half = 1.0, threequarter = 1.5, full = 2.0 }
PlantRegrowth.TimeMultipliers["my_moon_tree"] = function()
    return GLOBAL.TUNING.MOONTREE_REGROWTH_TIME_MULT
        * (MOON_PHASE_MULT[GLOBAL.TheWorld.state.moonphase] or 0)
end

-- 2) 世界密度：GetFiveRadius 读 TheWorld.generated.densities[区域 id][prefab]
--    正式模组应在世界生成期写入；这里对所有已生成区域补一份
local function EnsureDensity(prefab, density)
    local world = GLOBAL.TheWorld
    if world == nil or world.generated == nil or world.generated.densities == nil
        or world.topology == nil then
        return
    end
    for i in ipairs(world.topology.nodes) do
        local area_id = world.topology.ids[i]
        if area_id ~= nil then
            world.generated.densities[area_id] = world.generated.densities[area_id] or {}
            world.generated.densities[area_id][prefab] = world.generated.densities[area_id][prefab] or density
        end
    end
end

-- 3) 挂组件
AddPrefabPostInit("my_moon_tree", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end

    EnsureDensity("my_moon_tree", 1) -- 密度只影响搜索半径，取原版树同量级即可

    local regrowth = inst:AddComponent("plantregrowth")
    regrowth:SetRegrowthRate(GLOBAL.TUNING.MOON_TREE_REGROWTH.OFFSPRING_TIME)
    regrowth:SetProduct("moonbutterfly_sapling")
    -- 数后代：月树苗与月树带 moon_tree 标签，母树不计入
    regrowth:SetSearchTag("moon_tree")
end)
```

- 调整与验证：`OFFSPRING_TIME`＝3 天是**基准**，实际节奏是 `InternalTimes` **先**按 `dt × 时间倍率 × REGROWTH_TIME_MULTIPLIER` 累加、**再**判是否到期；示例的时间倍率是 `MOONTREE_REGROWTH_TIME_MULT × 月相倍率`——**世界设置倍率和全局倍率均大于 0 时**，满月（2.0）加快计时、等效间隔更短，新月（0）暂停累加，已到期的那次检查仍可能先执行一次；世界设置把月树再生设为禁用（`moon_tree_regrowth = never` → `MOONTREE_REGROWTH_TIME_MULT = 0`）时倍率为 0，满月也不会推进计时。所以验收要与倍率一致，覆盖三件事：① 出苗——砍掉/种下一棵后在地面实际看到 `moonbutterfly_sapling`；② 上限——候选落点搜索半径内带 `moon_tree` 标签的实体（月树苗＋长大的月树＋**原有月树**，不含母树）≥5 后不再冒新苗；③ 月相——在世界设置与全局倍率都大于 0 的前提下，满月档明显更快、新月档基本不动，且日志无 `attempt to call a nil value`；把世界设置改成禁用档应完全不出苗（倍率 0）。密度没写对时的表现是"从不冒苗且不再重试"，不是报错。[禁用档](../../../raw/current/scripts/worldsettings_overrides.lua#L2392-L2397)。示例只做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **误区：月树只会越长越高。** `loopstages = true`，高树到期会长回矮树（`grow_tall_to_short`）；想让它停在最高阶只能不让它继续长：砍倒、手动 `StopGrowing`，或**蜡封**（`moon_tree_waxed` 没有 `growable`，之后不再生长）。[规则](../../../raw/current/scripts/components/growable.lua#L93-L101)、[动画](../../../raw/current/scripts/prefabs/moontree.lua#L310-L315)
- **误区：砍倒的木头要自己捡，树桩会一直留着。** 树桩有 30±15 天的 `decay`，到点会删掉 6 格内多余的 `log`／`charcoal` 再自删；所以要留木头就早点搬走。[清理](../../../raw/current/scripts/prefabs/moontree.lua#L437-L462)
- **边界：烧掉的站立月树出 1–2 块木炭，且不再出花／月蛾。** `burnt_changes` 把掉落表换成只有 `charcoal` 1.0 的 `moontree_burnt`，通用 `GenerateLoot` 再对带 `burnt` 标签的实体按 40% 追加 1 块，所以砍那一下是 1–2 块。[切换](../../../raw/current/scripts/prefabs/moontree.lua#L108-L108)、[表](../../../raw/current/scripts/prefabs/moontree.lua#L72-L75)、[40% 追加](../../../raw/current/scripts/components/lootdropper.lua#L252-L254)
- **边界：树桩被烧是出灰，不是挖木炭。** `make_stump_burnable` 重挂的是 `MakeSmall/Medium/LargeBurnable`，它们的 `OnBurntFn` 是 `DefaultBurntFn`——烧尽时生成 1 个 `ash` 并移除本体，走不到 `burnt_changes`，所以没有"烧焦树桩"可挖。**例外**：在**烧尽回调后 0.5 秒内**先被砍倒的树（不是"点火后"），`burnt_changes` 的延时任务随后仍会套到那棵树桩上，此时它才带 `chop_down_burnt_moon_tree` 回调、挖一下掉 1–2 块木炭。[树桩可燃](../../../raw/current/scripts/prefabs/moontree.lua#L188-L197)、[DefaultBurntFn](../../../raw/current/scripts/standardcomponents.lua#L18-L44)、[只改回调](../../../raw/current/scripts/prefabs/moontree.lua#L110-L114)
- **边界：`moon_tree` 的尺寸是随机的。** 世界生成与调试生成都会走到 `math.random(1, 3)`；要固定尺寸就 `SpawnPrefab("moon_tree_tall")` 这类显式注册。[随机](../../../raw/current/scripts/prefabs/moontree.lua#L626-L626)
- **边界：两个"月树苗"不是一回事。** `moonbutterfly_sapling` 的显示名才是"月树苗"，`dug_sapling_moon` 显示"树苗"（移植件归 [[reference/prefabs/grass-saplings]]）。[名称](../../../raw/current/scripts/languages/chinese_s.po#L345620-L345622)
- **未核验**：砍伐与掉落概率的实测分布、月树苗的动画衔接、蜡封与月雹（`RemoveLunarHailBuildup`）的实际表现、`desolationspawner` 在月岛的触发频率。

## 验证记录

- 源码审阅：2026-11-09，Steam buildid 26474873（`raw/current/BUILD.txt`）。通读 `prefabs/moontree.lua` 全文（三张 loot 表、`make_stump`／`burnt_changes`／`on_timer_done`／`on_load`／`on_sleep`／`on_wake`、三阶 `set_*`／`grow_*`、`growfromseed`、`plantregrowth`／`waxable` 接线、四个注册）；核对 `prefabs/planted_tree.lua` 的 `sapling_fn` 与 `moonbutterfly_sapling` 注册、`components/growable.lua` 的循环与休眠补偿、`components/plantregrowth.lua` 的月相倍率与尝试点、`components/simplemagicgrower.lua`、`components/desolationspawner.lua` 的 `moon_tree` 重生登记、`map/rooms/forest/terrain_moonisland.lua` 三处权重、`map/static_layouts/moontrees_2.lua` 摆放、`prefabs/waxed_plants.lua`／`waxed_plant_common.lua` 的蜡封映射、`mainfunctions.lua` 的 postinit 顺序、`fx.lua` 的 `tree_petal_fx_chop`、`tuning.lua` 的生长／砍数／腐烂／重生常量与 `screens/redux/scrapbookdata.lua` 的采集册条目。
- 名称核验：`languages/chinese_s.po` 的 `NAMES.MOON_TREE`＝月树；`moon_tree_short`／`moon_tree_normal`／`moon_tree_tall` 三个尺寸注册**没有独立 NAMES 键**（`MOON_TREE_SHORT`／`_NORMAL`／`_TALL` 检索 0 命中），全靠 `SetPrefabName("moon_tree")` 取该键；`NAMES.MOONBUTTERFLY_SAPLING`＝月树苗、`NAMES.DUG_SAPLING_MOON`＝树苗。
- 分档说明：`size_class: complex`——4 个注册＋三阶循环生长＋砍伐／树桩／烧毁／腐烂四条收场＋两条再生链＋蜡封接入，需要整体说明生命周期，故按 complex 承载（仍按章节查询）。
- 未覆盖：Boss／其它生物与月树的交互、月雹积累的数值、月岛以外（移植到地表）的月树表现、`planted_tree.lua` 其它 sapling 的差异。
- R2 独立复审修订（2026-11-09）：典型接入补 `TimeMultipliers` 注册与未注册的 nil 后果（P1-01）；月蛾只在中／高表（P2-07）；烧焦树桩是 DIG（P2-08）；`moontrees_2` 改述为 `tree_area` 90%（P2-09）；sapling 计数与蜡封停长（P3）。
- 外部复检修订（2026-11-09）：区分"站立树烧毁（0.5 秒延时、1–2 块木炭）"与"树桩烧毁（`DefaultBurntFn` 出灰并移除）"两条链（P1-1）；再生示例补世界密度（`GetFiveRadius`／`generated.densities`）与自定义搜索标签（P1-2）；倍率改述为"计时推进量"（P2-1）；补 `moonbutterfly_sapling` 主注册归属与 `NAMES.MOON_TREE` 取键口径（P3-2）。
- 二次复检修订（2026-11-09）：再生示例改按月亮相位给倍率、`SetSearchTag("moon_tree")` 数后代（P1-1）；0.5 秒改为"从烧尽回调起算"并区分立即段／延时段（P2-1，含窗口内可能多 1 块木炭）；两条再生链改述为"先累加再判到期"（P2-2）；summary 与 S7 名称口径修正（P3-2）。
- 三次定向复检修订（2026-11-09）：两条再生公式补上 `MOONTREE_REGROWTH_TIME_MULT`（世界设置倍率）；"5 棵"改述为"候选落点搜索半径内带 `moon_tree` 标签的实体（含原有月树）≥5 就不生成"，不是区域配额（P2-3）；验收再补"世界设置倍率与全局倍率均 >0 时"的前提与禁用档（`worldsettings_overrides.lua` L2392-L2397）。
- 游戏运行：未运行；天数与刀数为源码推导（`total_day_time`＝480）。
- 结构检查：`validate_page.py` 通过（complex 档内）；`validate_source_links.py --strict` 0 错 0 警。

## 相关链接

- 同批产物：[[reference/prefabs/moontree-blossom]]、[[reference/prefabs/moonbutterfly]]
- 移植件与其它树：[[reference/prefabs/grass-saplings]]、[[reference/prefabs/tree-rocks]]
- 燃烧残留：[[reference/prefabs/ashes-and-charcoal]]（木炭与灰烬）
- 组件：[[reference/components/growable]]、[[reference/components/plantregrowth]]、[[reference/components/desolationspawner]]、[[reference/components/timer]]

返回 [[reference/prefabs/index|Prefab 索引]]。
