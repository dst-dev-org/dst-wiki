---
title: "海树波纹、根须、倒伏与落叶表现（oceantree FX 族）"
created: "2026-11-04"
updated: "2026-11-05"
type: prefab
size_class: complex
summary: "海树（疙瘩树）、树干柱与大树干的 12 个表现子注册：三档水面涟漪／水下根影、倒伏树干、柱体涟漪／根影与柱体落叶、大树干涟漪／根影；逐项记录工厂、端、生成点、父挂与清理。"
aliases: ["oceantree_ripples_short", "oceantree_ripples_normal", "oceantree_ripples_tall", "oceantree_roots_short", "oceantree_roots_normal", "oceantree_roots_tall", "oceantree_falling", "oceantree_pillar_ripples", "oceantree_pillar_roots", "oceantree_pillar_leaves", "watertree_pillar_ripples", "watertree_pillar_roots", "海树", "疙瘩树", "高出平均值的树干", "大树干", "海树波纹", "海树根须", "海树落叶", "倒伏树干"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/oceantree.lua", "prefabs/oceantree_pillar.lua", "prefabs/watertree_pillar.lua", "prefabs/waxed_plants.lua", "prefabs/waxed_plant_common.lua", "fx.lua", "entityscript.lua", "components/growable.lua", "componentactions.lua", "map/waterlog_worldgen.lua", "tuning.lua", "constants.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 海树波纹、根须、倒伏与落叶表现（oceantree FX 族）

## 快速判断

- 职责与适用：`prefabs/oceantree.lua`、`prefabs/oceantree_pillar.lua` 与 `prefabs/watertree_pillar.lua` 里 12 个 `FX` 子注册——海树三档水面涟漪／水下根影（`oceantree_ripples_*`／`_roots_*`）、倒伏树干（`oceantree_falling`）、树干柱的涟漪／根影／落叶（`oceantree_pillar_ripples`／`_roots`／`_leaves`）、大树干的涟漪／根影（`watertree_pillar_ripples`／`_roots`）。都是纯表现注册，不挂玩法组件；树的生长、砍伐、倒伏结算和柱体遮阳／引雷、大树干栖息等本体玩法不在本页。[树注册](../../../raw/current/scripts/prefabs/oceantree.lua#L926-L941)、[柱注册](../../../raw/current/scripts/prefabs/oceantree_pillar.lua#L726-L729)、[大树干注册](../../../raw/current/scripts/prefabs/watertree_pillar.lua#L580-L582)
- 优先使用：按名字直达——三档涟漪／根影的档位由宿主阶段选；倒伏表现读 `oceantree_falling`；柱体三件套读对应 `_pillar_*`；大树干涟漪／根影读 `watertree_pillar_*`。生成点、父挂与清理见“关联机制导航”；海树、柱体与大树干本体注册只用于辨析。[阶段选择](../../../raw/current/scripts/prefabs/oceantree.lua#L87-L93)、[倒伏](../../../raw/current/scripts/prefabs/oceantree.lua#L843-L895)
- 重要副作用：自身无伤害／掉落；`oceantree_falling` 播 `dontstarve/forest/treefall`、在落点调用 `ShakeAllCameras`，并在落点判定为海面时生成 `splash_green_large`（阶段 2/3）或 `splash_green`（阶段 1）后自删；`oceantree_pillar_leaves` 在非可视地面改播落水动画并补 `splash/medium` 声；涟漪调用海洋混色 0.6，根影在水下层。[落水选档](../../../raw/current/scripts/prefabs/oceantree.lua#L843-L872)、[落叶水陆](../../../raw/current/scripts/prefabs/oceantree_pillar.lua#L705-L712)、[混色常量](../../../raw/current/scripts/tuning.lua#L4407-L4412)
- 修改边界：全部在服务端设 `persists=false`，服务端专属逻辑在守卫之后；客户端是否可见取决于联网实体与引擎动画同步，未运行核验。树涟漪／根影用裸 `entity:SetParent`，不登记 `EntityScript.children`；柱体在 `OnRemoveEntity` 里显式删。[树父挂](../../../raw/current/scripts/prefabs/oceantree.lua#L89-L93)、[柱清理](../../../raw/current/scripts/prefabs/oceantree_pillar.lua#L353-L360)、[children 循环](../../../raw/current/scripts/entityscript.lua#L1779-L1784)

## 注册与构造

三档涟漪共用 `make_ripples`，三档根影共用 `make_roots`；倒伏树干、柱体三件套与大树干两 FX 各自一个工厂（大树干两工厂与柱体两件同构）。树 FX 注册不带 `assets`，资源由 `oceantree` 的资产表提供；柱体 FX 注册复用柱体的 `assets` 表，大树干 FX 注册复用大树干的 `assets` 表。[树工厂](../../../raw/current/scripts/prefabs/oceantree.lua#L780-L841)、[树资产](../../../raw/current/scripts/prefabs/oceantree.lua#L1-L14)、[柱工厂](../../../raw/current/scripts/prefabs/oceantree_pillar.lua#L632-L724)、[柱资产](../../../raw/current/scripts/prefabs/oceantree_pillar.lua#L1-L8)、[大树干工厂](../../../raw/current/scripts/prefabs/watertree_pillar.lua#L523-L578)

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `oceantree_ripples_short` | `make_ripples(name, build)` → `ripples_fn`：Transform／AnimState／Network → `FX`／`NOCLICK` → bank=build=`oceantree_short`、循环 `root_ripple`、`SetOceanBlendParams(0.6)` → `SetPristine` → 客户端早退 → 服务端 `persists=false`。 | 三档只差 build；无音效、无 assets 参数。[工厂](../../../raw/current/scripts/prefabs/oceantree.lua#L780-L809)、[注册](../../../raw/current/scripts/prefabs/oceantree.lua#L934-L936) |
| `oceantree_ripples_normal` | 同上工厂。 | build=`oceantree_normal`。[注册](../../../raw/current/scripts/prefabs/oceantree.lua#L935) |
| `oceantree_ripples_tall` | 同上工厂。 | build=`oceantree_tall`。[注册](../../../raw/current/scripts/prefabs/oceantree.lua#L936) |
| `oceantree_roots_short` | `make_roots(name, build)` → `roots_fn(data)`：Transform／AnimState／Network → `FX`／`NOCLICK` → bank=build=`oceantree_short`、非循环 `root_shadow`、`SetSortOrder(ANIM_SORT_ORDER_BELOW_GROUND.UNDERWATER)`＋`SetLayer(LAYER_WIP_BELOW_OCEAN)` → `SetPristine` → 客户端早退 → 服务端 `persists=false`。 | 三档只差 build；无音效；水下排序值 0、层值 2。[工厂](../../../raw/current/scripts/prefabs/oceantree.lua#L811-L841)、[层常量](../../../raw/current/scripts/constants.lua#L1644-L1650)、[层值](../../../raw/current/scripts/constants.lua#L68)、[注册](../../../raw/current/scripts/prefabs/oceantree.lua#L937-L939) |
| `oceantree_roots_normal` | 同上工厂。 | build=`oceantree_normal`。[注册](../../../raw/current/scripts/prefabs/oceantree.lua#L938) |
| `oceantree_roots_tall` | 同上工厂。 | build=`oceantree_tall`。[注册](../../../raw/current/scripts/prefabs/oceantree.lua#L939) |
| `oceantree_falling` | `falling_tree_fn`：Transform／AnimState／SoundEmitter／Network → `FX`／`NOCLICK` → `MakeSnowCoveredPristine` → `SetPristine` → 客户端早退 → 服务端 `persists=false`、`MakeSnowCovered`、暴露 `start_falling_fn`。 | 工厂不设 bank／build／anim；`start_falling_fn(build, bank, fallleft, stage, camera_right)` 用树当前 build 与阶段 bank 播 `fallleft`／`fallright`，0.4 秒后落地处理；无父挂、无 assets。[工厂](../../../raw/current/scripts/prefabs/oceantree.lua#L897-L924)、[起倒](../../../raw/current/scripts/prefabs/oceantree.lua#L874-L895)、[注册](../../../raw/current/scripts/prefabs/oceantree.lua#L941) |
| `oceantree_pillar_ripples` | `ripples_fn`：Transform／AnimState／Network → `FX`／`NOCLICK` → bank=`oceantree_pillar_small`、build=`oceantree_pillar_small_build1`＋`AddOverrideBuild("oceantree_pillar_small_build2")`、循环 `root_ripple`、`SetOceanBlendParams(0.6)` → `SetPristine` → 客户端早退 → 服务端 `persists=false`。 | 注册复用柱体 `assets` 表（含 build1／build2 与 canopyshadows 脚本）；比树涟漪多 override build。[工厂](../../../raw/current/scripts/prefabs/oceantree_pillar.lua#L632-L657)、[注册](../../../raw/current/scripts/prefabs/oceantree_pillar.lua#L727) |
| `oceantree_pillar_roots` | `roots_fn`：Transform／AnimState／Network → `FX`／`NOCLICK` → 同 bank=`oceantree_pillar_small`、build=`…_build1`＋override build、非循环 `root_shadow`、`UNDERWATER` 排序＋`LAYER_WIP_BELOW_OCEAN` → `SetPristine` → 客户端早退 → 服务端 `persists=false`。 | 注册复用柱体 `assets` 表；水下根影。[工厂](../../../raw/current/scripts/prefabs/oceantree_pillar.lua#L659-L685)、[注册](../../../raw/current/scripts/prefabs/oceantree_pillar.lua#L728) |
| `oceantree_pillar_leaves` | `leaves_fn`：Transform／AnimState／Network／SoundEmitter → `FX`／`NOCLICK` → 同 bank=`oceantree_pillar_small`、build=`…_build1`＋override build、非循环 `leaf_fall_ground` → `animover`→`Remove`（在 `SetPristine` 前注册，双端生效）→ 0 帧任务检查可视地面，非地面改播 `leaf_fall_water`、11 帧后补 `splash/medium` 声 → `SetPristine` → 客户端早退 → 服务端 `persists=false`。 | 落点 y=0、不父挂；地面／落水分支由两端各自按地图判定。[工厂](../../../raw/current/scripts/prefabs/oceantree_pillar.lua#L687-L724)、[注册](../../../raw/current/scripts/prefabs/oceantree_pillar.lua#L729) |
| `watertree_pillar_ripples` | `ripples_fn`：Transform／AnimState／Network → `FX`／`NOCLICK` → bank=`oceantree_pillar`、build=`oceantree_pillar_build1`＋`AddOverrideBuild("oceantree_pillar_build2")`、循环 `root_ripple`、`SetOceanBlendParams(TUNING.OCEAN_SHADER.EFFECT_TINT_AMOUNT)` → `SetPristine` → 客户端早退 → 服务端 `persists=false`。 | 与柱体涟漪同构；bank／build 用大树干一代资源（`oceantree_pillar`，非 `_small`），混色取同一调值常量（0.6）；无音效、无自删。[工厂](../../../raw/current/scripts/prefabs/watertree_pillar.lua#L523-L549)、[注册](../../../raw/current/scripts/prefabs/watertree_pillar.lua#L581)、[混色常量](../../../raw/current/scripts/tuning.lua#L4407-L4412) |
| `watertree_pillar_roots` | `roots_fn(data)`：Transform／AnimState／Network → `FX`／`NOCLICK` → 同 bank=`oceantree_pillar`、build=`…_build1`＋override build2、非循环 `root_shadow`、`UNDERWATER` 排序＋`LAYER_WIP_BELOW_OCEAN` → `SetPristine` → 客户端早退 → 服务端 `persists=false`。 | 与柱体根影同构（`data` 形参未用）；水下排序值 0、层值 2；无音效、无自删。[工厂](../../../raw/current/scripts/prefabs/watertree_pillar.lua#L551-L578)、[注册](../../../raw/current/scripts/prefabs/watertree_pillar.lua#L582)、[层常量](../../../raw/current/scripts/constants.lua#L1644-L1650) |

简中 `languages/chinese_s.po` 定向检索这 12 个注册名均无 `STRINGS.NAMES`／显示名键；标题与表内中文为已核验效果描述，非官方译名。宿主官方名可查：`OCEANTREE`＝疙瘩树、`OCEANTREE_PILLAR`＝高出平均值的树干、`WATERTREE_PILLAR`＝大树干；`海树` 是页内统称／别名。[疙瘩树](../../../raw/current/scripts/languages/chinese_s.po#L346564-L346567)、[树干柱](../../../raw/current/scripts/languages/chinese_s.po#L346574-L346577)、[大树干](../../../raw/current/scripts/languages/chinese_s.po#L351859-L351862)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 生成与档位：海树生长阶段 | `update_ripples_roots` 先删旧 `ripples`／`roots`，按 `growable.stage` 取 `tall`（3）／`normal`（2）／`short`（其余）后缀，再各 `SpawnPrefab` 并 `entity:SetParent(inst.entity)`；`SetShort`／`SetNormal`／`SetTall` 与 `tree()` 初次构造都调用。 | 服务端阶段函数；读档时 `growable:OnLoad`→`SetStage` 会再触发一次，活树可重建；烧毁路径与 stump prefab 初始分支会移除 `growable`（L134／L739），砍倒后的 `make_stump` 只 `StopGrowing`（L325-L327），读档重建行为需运行核验（源码 TODO）。 | [函数](../../../raw/current/scripts/prefabs/oceantree.lua#L73-L94)、[三阶段](../../../raw/current/scripts/prefabs/oceantree.lua#L180-L250)、[构造](../../../raw/current/scripts/prefabs/oceantree.lua#L762)、[SetStage](../../../raw/current/scripts/components/growable.lua#L207-L219)、[OnLoad](../../../raw/current/scripts/components/growable.lua#L249-L254) |
| 清理：树涟漪／根影 | 裸 `SetParent`；阶段变化时先 `Remove` 旧实例再生成；`persists=false` 使它们不存档。砍倒成桩的 `make_stump` 只 `StopGrowing`、不移除 `growable`；`Growable:OnSave` 仍保存 stage，`OnLoad`→`SetStage` 会进入阶段回调并可能再次生成涟漪，因此不能由旧 TODO 断言所有砍后树桩读档都不重建；烧毁路径移除 `growable`、直接构造 stump prefab 是另外两条分支，具体可见结果未运行核验。树实体最终 `Remove` 时引擎级联未核验。 | 服务端；直挂不进 `EntityScript.children`，`AddChild` 的 children 循环覆盖不到。 | [移除旧实例](../../../raw/current/scripts/prefabs/oceantree.lua#L77-L93)、[源码注释](../../../raw/current/scripts/prefabs/oceantree.lua#L74-L75)、[树桩](../../../raw/current/scripts/prefabs/oceantree.lua#L307-L332)、[growable 存档](../../../raw/current/scripts/components/growable.lua#L227-L254)、[children](../../../raw/current/scripts/entityscript.lua#L1779-L1784) |
| 生成与落地：倒伏树干 | `chop_down_tree` 在树位置 `SpawnPrefab("oceantree_falling")`，复制已用 bud 符号，按阶段 bank＋当前 build 调 `start_falling_fn`；落点按阶段选 `splash_green_large`（2/3）或 `splash_green`（1），落水则生成水花后 `Remove`，非水则等 `animover` 自删。 | 服务端生成；船撞砍伐分支随后直接 `inst:Remove()`，但倒伏树已生成。 | [生成](../../../raw/current/scripts/prefabs/oceantree.lua#L348-L365)、[落地](../../../raw/current/scripts/prefabs/oceantree.lua#L843-L872)、[船分支](../../../raw/current/scripts/prefabs/oceantree.lua#L367-L369) |
| 生成与清理：柱体涟漪／根影 | 柱体 `fn` 末尾生成 `_ripples` 与 `roots` 并各自 `SetParent`；`OnRemoveEntity` 先删 `roots` 再删 `_ripples`，倒伏／烧毁开始时手动调用一次，最终实体 `Remove` 时再走引擎回调。 | 服务端；柱体 FX 只随柱体存在，工厂自身无自删。 | [生成](../../../raw/current/scripts/prefabs/oceantree_pillar.lua#L613-L617)、[清理](../../../raw/current/scripts/prefabs/oceantree_pillar.lua#L353-L360)、[倒伏调用](../../../raw/current/scripts/prefabs/oceantree_pillar.lua#L308-L310)、[烧毁调用](../../../raw/current/scripts/prefabs/oceantree_pillar.lua#L420-L422) |
| 生成与清理：柱体落叶 | `Dropleafitems` 在柱体周围 6–16 距离、y=0 生成 `oceantree_pillar_leaves`；`Dropleaves` 每 0.05 秒生成 3 个、计数超过 1 后取消；由船撞（`OnCollide`）、砍倒、烧毁三条 `dropcanopystuff(..., true)` 触发。工厂 `animover` 自删，落地／落水分支见上。 | 服务端生成；落水判定在两端各跑（`IsVisualGroundAtPoint` 为假即走落水分支，未另查平台，补 `splash/medium`）。 | [生成](../../../raw/current/scripts/prefabs/oceantree_pillar.lua#L114-L137)、[周期](../../../raw/current/scripts/prefabs/oceantree_pillar.lua#L219-L232)、[船撞](../../../raw/current/scripts/prefabs/oceantree_pillar.lua#L300)、[砍倒](../../../raw/current/scripts/prefabs/oceantree_pillar.lua#L343)、[烧毁](../../../raw/current/scripts/prefabs/oceantree_pillar.lua#L456)、[水语义](../../../raw/current/scripts/componentactions.lua#L48-L54) |
| 生成与清理：大树干涟漪／根影 | 大树干 `fn`（服务端）末尾生成 `_ripples` 与 `roots` 并各自裸 `SetParent`；`OnRemoveEntity` 先删 `roots` 再删 `_ripples`。宿主没有砍倒／烧毁路径，不存在柱体那样的手动 `OnRemoveEntity` 调用，清理只发生在宿主 `Remove` 时。 | 服务端；两 FX `persists=false` 不存档，读档由宿主 `fn` 重建；宿主投放走世界生成标记（`waterlog_worldgen.lua` 以 `watertree_pillar` 为 MARKER_PREFAB）。 | [生成](../../../raw/current/scripts/prefabs/watertree_pillar.lua#L509-L513)、[清理](../../../raw/current/scripts/prefabs/watertree_pillar.lua#L353-L370)、[OnRemoveEntity 挂接](../../../raw/current/scripts/prefabs/watertree_pillar.lua#L518)、[世界生成标记](../../../raw/current/scripts/map/waterlog_worldgen.lua#L10-L11) |
| 复用：涂抹海树 | `OceanTree_OnConfigure` 从当前 build 去掉 `oceantree_` 与 `_jammed_build` 得 `short`／`normal`／`tall`，`SpawnChild("oceantree_ripples_"..suffix)` 与 `_roots_`；`Configure` 在读档、部署、涂蜡时调用。 | 服务端（`onconfigure_fn` 在主模拟守卫后才赋值）；`SpawnChild` 走 `AddChild`，会登记 `children`，宿主删除时由 `Remove` 子实体循环清理。 | [配置](../../../raw/current/scripts/prefabs/waxed_plants.lua#L544-L549)、[定义](../../../raw/current/scripts/prefabs/waxed_plants.lua#L829-L843)、[Configure](../../../raw/current/scripts/prefabs/waxed_plant_common.lua#L8-L83)、[赋值](../../../raw/current/scripts/prefabs/waxed_plant_common.lua#L326-L336)、[涂蜡调用](../../../raw/current/scripts/prefabs/waxed_plant_common.lua#L569-L581)、[SpawnChild](../../../raw/current/scripts/entityscript.lua#L896-L935) |
| 依赖声明（不证明生成） | `oceantree.lua` 的 `prefabs` 表声明 7 个 FX，`oceantree_pillar.lua` 声明 3 个；这些只是加载依赖，实际生成点见上。 | 声明不等于生成。 | [树依赖](../../../raw/current/scripts/prefabs/oceantree.lua#L16-L35)、[柱依赖](../../../raw/current/scripts/prefabs/oceantree_pillar.lua#L10-L16) |
| 对照与边界 | 倒伏落水水花归 [[reference/prefabs/splash_green]]；`oceantree_leaf_fx_fall`／`_chop` 是 `fx.lua` 通用表条目，归 [[reference/prefabs/sugarwood_leaf_fx]]，与 `oceantree_pillar_leaves` 不是同一注册；6 个树本体（`oceantree`、`oceantree_normal`、`oceantree_tall`、`oceantree_short`、`oceantree_burnt`、`oceantree_stump`，均 `SetPrefabName("oceantree")`）与 `oceantree_pillar` 本体不属于本页；`watertree_pillar.lua` 的大树干涟漪／根影两注册已收进本页，但它不生成 `oceantree_pillar_*` 三件套，且 bank 用 `oceantree_pillar`（大树干一代资源）而非柱体的 `oceantree_pillar_small`。 | 树叶飘落／砍伐叶响与柱体落叶落地动画分开；两代柱体的涟漪／根影同名同动画但不同 bank。 | [fx.lua 条目](../../../raw/current/scripts/fx.lua#L2636-L2651)、[树本体注册](../../../raw/current/scripts/prefabs/oceantree.lua#L926-L932)、[统一名](../../../raw/current/scripts/prefabs/oceantree.lua#L639)、[柱本体](../../../raw/current/scripts/prefabs/oceantree_pillar.lua#L726-L729)、[大树干 bank](../../../raw/current/scripts/prefabs/watertree_pillar.lua#L533-L537) |
| 资源 | 树 FX 用 `anim/oceantree_short.zip`、`anim/oceantree_normal.zip`、`anim/oceantree_tall.zip`（另 `tall_jammed_build`）与 `sound/forest.fsb`；树 FX 注册不带 assets，资源由 `oceantree` 的 assets 声明。柱体 FX 复用柱体 assets：`anim/oceantree_pillar_small.zip`（bank）、`anim/oceantree_pillar_small_build1.zip`、`..._build2.zip`、`scripts/prefabs/canopyshadows.lua`。大树干 FX 复用大树干 assets：`anim/oceantree_pillar.zip`（bank）、`anim/oceantree_pillar_build1.zip`、`..._build2.zip`、`scripts/prefabs/canopyshadows.lua`。声效：`treefall`、`turnoftides/.../splash/medium`；混色 0.6（`EFFECT_TINT_AMOUNT`）。 | 动画时长、循环标记与视觉未核验；不分发资源。 | [树资源](../../../raw/current/scripts/prefabs/oceantree.lua#L1-L14)、[柱资源](../../../raw/current/scripts/prefabs/oceantree_pillar.lua#L1-L8)、[注册复用](../../../raw/current/scripts/prefabs/oceantree_pillar.lua#L726-L729)、[大树干资源](../../../raw/current/scripts/prefabs/watertree_pillar.lua#L1-L10)、[混色](../../../raw/current/scripts/tuning.lua#L4407-L4412) |

## 典型接入

### 给自定义海面宿主挂同档涟漪与根影

- 前提与选择：需要随宿主长期存在的水面涟漪／水下根影时，按宿主阶段拼 `oceantree_ripples_`／`oceantree_roots_` 后缀；这是原版内部注册名，不是公开接口，升级后需复核。原版由 `update_ripples_roots` 在阶段切换时先删旧再生成。[原版函数](../../../raw/current/scripts/prefabs/oceantree.lua#L73-L94)
- 生效与撤销：两组都是 `persists=false` 的联网 FX；裸 `SetParent` 只建引擎父子变换，不登记 `EntityScript.children`，宿主删除时是否随引擎级联回收未核验，复用时应自己保存引用并在宿主移除前显式 `Remove`（或改用 `AddChild`）。[直挂与循环](../../../raw/current/scripts/entityscript.lua#L921-L935)、[移除循环](../../../raw/current/scripts/entityscript.lua#L1779-L1784)
- 原版依据：阶段后缀由 `growable.stage` 3／2／其余映射，`SetShort`／`SetNormal`／`SetTall` 与初次构造都调用。[档位](../../../raw/current/scripts/prefabs/oceantree.lua#L87-L93)、[阶段函数](../../../raw/current/scripts/prefabs/oceantree.lua#L180-L250)

```lua
-- 服务端：给自定义海面宿主按档位挂同档涟漪与根影
local function AttachOceanTreeFx(inst, suffix) -- suffix = "short" | "normal" | "tall"
    if inst.ripples ~= nil then
        inst.ripples:Remove()
    end
    if inst.roots ~= nil then
        inst.roots:Remove()
    end

    inst.ripples = SpawnPrefab("oceantree_ripples_" .. suffix)
    inst.ripples.entity:SetParent(inst.entity)

    inst.roots = SpawnPrefab("oceantree_roots_" .. suffix)
    inst.roots.entity:SetParent(inst.entity)
end

return AttachOceanTreeFx
```

- 调整与验证：后缀必须与已加载的树 build 对应；树 FX 注册不带 assets，独立复用前先确认 `oceantree` 的动画资源已加载。示例只做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **边界：直挂 `SetParent` 不等于 `AddChild`**：树涟漪／根影没有进 `EntityScript.children`，宿主 `Remove` 的子实体循环覆盖不到；原版树也没有为它们写显式移除。需要“宿主删了 FX 一定走”时改用 `AddChild`，或自行清理。[树父挂](../../../raw/current/scripts/prefabs/oceantree.lua#L89-L93)、[children 循环](../../../raw/current/scripts/entityscript.lua#L1779-L1784)、[AddChild](../../../raw/current/scripts/entityscript.lua#L921-L935)
- **边界：砍倒／烧毁后的读档不能一概而论。** 普通活树砍成桩时 `make_stump` 只 `StopGrowing`、不移除 `growable`；`Growable:OnSave` 即使无剩余时间仍保存 stage，`OnLoad`→`SetStage` 会调用阶段函数并可能重建涟漪／根影。烧毁路径移除 `growable`、直接构造 stump prefab 是另外两条分支；旧 TODO 不足以证明所有砍后树桩读档都不重建。同会话内砍成树桩时没有移除旧 FX 的调用，旧涟漪／根影会留在同一实体上；实际观感与各分支读档结果未运行核验。[源码注释](../../../raw/current/scripts/prefabs/oceantree.lua#L74-L75)、[判定](../../../raw/current/scripts/prefabs/oceantree.lua#L85-L93)、[树桩](../../../raw/current/scripts/prefabs/oceantree.lua#L307-L332)、[growable 存档](../../../raw/current/scripts/components/growable.lua#L227-L254)、[读档](../../../raw/current/scripts/prefabs/oceantree.lua#L511-L535)
- **边界：倒伏树的客户端 Lua 不设动画**：`falling_tree_fn` 在 `SetPristine` 后客户端早退，bank／build／`fallleft`／`fallright` 都由服务端 `start_falling_fn` 决定；跨端动画同步属引擎层，未运行核验，不要据此断言客户端一定可见或不可见。[工厂](../../../raw/current/scripts/prefabs/oceantree.lua#L897-L924)、[起倒](../../../raw/current/scripts/prefabs/oceantree.lua#L874-L895)
- **误区：`oceantree_pillar_leaves` 不是叶片飘落 FX**：它是柱体落叶的地面／水面落地动画；柱体每次砍伐以 0.58 概率生成的 `oceantree_leaf_fx_fall` 是 `fx.lua` 的另一注册，`small_ram_products` 里也是它。两者名字相近但归属不同。[柱体落叶](../../../raw/current/scripts/prefabs/oceantree_pillar.lua#L687-L712)、[砍伐叶飘](../../../raw/current/scripts/prefabs/oceantree_pillar.lua#L106-L111)、[通用表条目](../../../raw/current/scripts/fx.lua#L2636-L2651)、[[reference/prefabs/sugarwood_leaf_fx]]
- **边界：柱体涟漪／根影没有自删**：循环动画且只随柱体 `OnRemoveEntity` 清理；单独生成或被别的宿主拿走时，需要自己保存引用并显式删除。[生成](../../../raw/current/scripts/prefabs/oceantree_pillar.lua#L613-L617)、[清理](../../../raw/current/scripts/prefabs/oceantree_pillar.lua#L353-L360)
- **边界：大树干两 FX 无自删、且宿主没有倒伏／烧毁的手动清理调用**：大树干不可砍倒／烧毁，`OnRemoveEntity` 只由宿主 `Remove` 触发；柱体在开始倒伏／烧毁时会先手动调用一次清理，大树干没有对应路径，单独生成或换宿主时同样要自管引用。[生成](../../../raw/current/scripts/prefabs/watertree_pillar.lua#L509-L513)、[清理](../../../raw/current/scripts/prefabs/watertree_pillar.lua#L353-L370)
- **边界：大树干（`watertree_pillar`）本体不是本页对象**：它是不可砍／烧的世界实体（官方名"大树干"），玩法为草鳄鱼栖息、船撞掉落坚果与枝叶、遮阴树冠、引雷转移与水草茧再生；本页只收其涟漪／根影两个表现子注册，本体玩法待建页后互链。[宿主 fn](../../../raw/current/scripts/prefabs/watertree_pillar.lua#L406-L521)、[官方名](../../../raw/current/scripts/languages/chinese_s.po#L351859-L351862)
- **边界：树 FX 的资产不在自己的注册里**：`make_ripples`／`make_roots` 与 `oceantree_falling` 都不传 assets，bank／build 依赖 `oceantree` 的动画包；柱体三件套则复用柱体 assets。独立复用前先确认对应 build 已加载，否则只是名字存在。[树资产](../../../raw/current/scripts/prefabs/oceantree.lua#L1-L14)、[柱注册](../../../raw/current/scripts/prefabs/oceantree_pillar.lua#L726-L729)

## 验证记录

- 源码审阅：2026-11-04，steam-buildid:26335394。逐项核验 12 个注册的返回位置、工厂、端分界、生成点、父挂与清理：`make_ripples`／`make_roots`／`falling_tree_fn`／柱体 `ripples_fn`／`roots_fn`／`leaves_fn`／大树干 `ripples_fn`／`roots_fn` 全文；`update_ripples_roots` 的阶段映射与 `SetParent`；`chop_down_tree` 的倒伏生成与落水选档；柱体 `fn`／`OnRemoveEntity`／`Dropleaves`；`waxed_plants` 的 `OceanTree_OnConfigure` 与 `waxed_plant_common` 的 `Configure` 链；`growable` 的 OnSave／OnLoad／SetStage。以 `grep -rn --include='*.lua'` 对 12 个完整名在 `raw/current/scripts` 全域反查：除自身注册、依赖表与上述生成点外，仅另有 `watertree_pillar.lua` 注释中的同源资产行与 `waxed_plants` 的拼接后缀；`stategraphs/` 无命中，未发现树生长／砍伐／倒伏 SG 引用。`.po` 对 12 个注册名定向检索为 0 命中；`OCEANTREE`／`OCEANTREE_PILLAR`／`WATERTREE_PILLAR` 的官方名已核。未运行游戏。
- 分档说明：覆盖 12 个注册、3 个工厂文件、8 个工厂函数、3 类宿主与多种清理路径（父挂循环、显式删除、落地自删、animover 自删），并含读档缺口与涂抹海树的 `SpawnChild` 父挂复用，按 `complex` 档保留整体对照。
- 结构检查：`validate_page.py` 通过（0 issue；111 行、29,685 字节，低于 complex 档 400 行／32 KiB）；`validate_source_links.py --strict` 通过（117 处源码引用、0 error／0 warning、85 个唯一目标、13 个源文件）；正文 1 段 Lua 示例经 `parser_probe.exe --syntax-only` 通过（542 字节，只查语法、未执行，临时文件已删）。
- 未覆盖：动画包实际帧数与循环标记、倒伏树的客户端动画同步、直挂子实体的引擎级联回收、同会话树桩残留表现、柱体落叶在真实海面／陆地的落点观感；未运行游戏。
- 2026-11-05 增补：并入 `prefabs/watertree_pillar.lua` 的 2 个注册 `watertree_pillar_ripples`／`watertree_pillar_roots`（注册 [L581/L582](../../../raw/current/scripts/prefabs/watertree_pillar.lua#L580-L582)，生成点唯一：宿主 `fn` 末尾 [L509/L512](../../../raw/current/scripts/prefabs/watertree_pillar.lua#L509-L513)）。本次完整读取 `watertree_pillar.lua` 全文并复读 `oceantree_pillar.lua` 的工厂／注册／生成／清理对照段；以完整名全域 grep 反查两名，除注册、宿主生成外无其他命中，`stategraphs/` 无命中。复核确认两 FX 与柱体两件同构，差异在 bank／build（`oceantree_pillar` 一代资源 vs 柱体 `oceantree_pillar_small`）、混色取 `TUNING.OCEAN_SHADER.EFFECT_TINT_AMOUNT`（0.6）与宿主无倒伏／烧毁手动清理路径；台账批 9"同 bank"的表述据此修正为"同名同动画、不同 bank"。`.po` 对两名定向检索 0 命中，`WATERTREE_PILLAR`＝"大树干"已核。页面注册数 10→12，aliases 增补两名与"大树干"，summary 同步扩充，`sources` 原已列 `watertree_pillar.lua`、新增 `map/waterlog_worldgen.lua`。未覆盖新增项：动画包帧数与视觉、宿主本体玩法（另页主题）、两 FX 在读档时的引擎级联回收；未运行游戏。

返回 [[reference/prefabs/fx-water-index|水面、航海与船件]]；倒伏落水水花见 [[reference/prefabs/splash_green|海面绿花与冲滩水洼]]，海树叶飘落／砍伐叶响见 [[reference/prefabs/sugarwood_leaf_fx|特殊树叶与死亡烟]]。
