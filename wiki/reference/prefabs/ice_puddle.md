---
title: "宿主附属的贴地与贴水 FX 族（ice_puddle、ice_splash、nitre_formation、oceanfishinghook_ripple）"
created: "2026-11-04"
updated: "2026-11-04"
type: prefab
size_class: complex
summary: "四个宿主侧生成的纯表现注册：冰面巨石的贴地水洼、融冰水花、酸蚀池塘硝石晶簇与浮标水下涟漪；差异集中在端、父挂与清理——只有晶簇联网并登记 pond.highlightchildren，水洼不自删，水花 animover 自删，涟漪循环到宿主移除。"
aliases: ["ice_puddle", "ice_splash", "nitre_formation", "oceanfishinghook_ripple", "硝石堆", "冰面水洼", "融冰水花", "硝石晶簇", "浮标涟漪", "贴地水洼"]
tags: [prefab, fx, server-client]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/ice_puddle.lua", "prefabs/ice_splash.lua", "prefabs/nitre_formation.lua", "prefabs/oceanfishingbobber.lua", "prefabs/rock_ice.lua", "prefabs/rock_ice_temperature.lua", "prefabs/sharkboi_ice_hazard.lua", "prefabs/pond.lua", "components/highlight.lua", "entityscript.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 宿主附属的贴地与贴水 FX 族（ice_puddle、ice_splash、nitre_formation、oceanfishinghook_ripple）

## 快速判断

- 职责与适用：四个宿主侧生成的纯表现注册，分散在四个 prefab 文件（前三个各自独立成文件，`oceanfishinghook_ripple` 与浮标玩法同文件）。`ice_puddle` 是冰面巨石脚下的贴地水洼，`ice_splash` 是融化瞬间的水花，`nitre_formation` 是酸蚀洞穴池塘周围的硝石晶簇，`oceanfishinghook_ripple` 是浮标脚下的水下涟漪。四者都不挂玩法组件，伤害／掉落／加湿等结算在宿主；工厂与配置见“注册与构造”。
- 优先使用：按需求直达——冰面巨石水洼选 `ice_puddle`；一次性融化／碎裂水花选 `ice_splash`；酸蚀池塘晶簇及其高亮联动选 `nitre_formation`；浮标水面波纹选 `oceanfishinghook_ripple`。生成点与端限制见“关联机制导航”。
- 重要副作用：本身无玩法作用；`nitre_formation` 是唯一联网注册，客户端复制后把自身登记进父实体 `pond_cave` 的 `highlightchildren`，使晶簇随池塘一起被高亮。其余三个不联网；原版普通生成路径多在非专服，`ice_splash` 的强制清理例外见下文。[登记与反登记](../../../raw/current/scripts/prefabs/nitre_formation.lua#L6-L27)、[高亮消费](../../../raw/current/scripts/components/highlight.lua#L88-L92)
- 修改边界：创建与收尾按对象分属宿主（父挂／子挂／池塘显式移除／animover），`SetParent` 的删除级联未核验；本页只记生成点与生命周期，不写 rock_ice、pond_cave、浮标及其钓鱼玩法；朝向、层与排序按源码实际调用记录，未设置的不代填。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `ice_puddle` | 独立文件：`fn` 直接 CreateEntity → Transform＋AnimState（无 Network、无物理）→ `FX` 标签、`persists=false` → bank／build `ice_puddle`、`full`、`LAYER_BACKGROUND`、排序 3 | 非联网；只播一次、不监听 animover，工厂无自删路径；生成者负责父挂与换动画。[工厂](../../../raw/current/scripts/prefabs/ice_puddle.lua#L6-L24) |
| `ice_splash` | 独立文件：同前缀初始化 → `FX` 标签、`SetCanSleep(false)`、`persists=false` → bank／build `ice_splash`、`full` → `animover` 监听 `inst.Remove` | 非联网；播完即删；未显式设置朝向、层与排序（用默认）。[工厂](../../../raw/current/scripts/prefabs/ice_splash.lua#L6-L24) |
| `nitre_formation` | 独立文件：Transform＋AnimState＋Network → bank `pond_rock`、build `pond_nitrecrystal`、`idle1`、缩放 0.75、`FX` 标签 → `SetPristine` → 客户端装 `OnEntityReplicated` 后返回 | 唯一联网注册；服务端 fn 不挂组件；客户端按父实体复制登记，移除时反登记。[工厂](../../../raw/current/scripts/prefabs/nitre_formation.lua#L29-L52) |
| `oceanfishinghook_ripple` | 同文件 `ripple_fn`：Transform＋AnimState（无 Network）→ `CLASSIFIED`＋`FX` → bank／build `oceanfishing_hook`、`fx_ripple_small`（循环）→ `OnGround`、`LAYER_BELOW_GROUND`、`ANIM_SORT_ORDER_BELOW_GROUND.UNDERWATER` → `persists=false` | 非联网；循环动画、无 animover 自删；由浮标 `AddChild` 子挂。[工厂](../../../raw/current/scripts/prefabs/oceanfishingbobber.lua#L289-L312) |

名称：仅 `nitre_formation` 命中 `STRINGS.NAMES`——`NITRE_FORMATION` 在简中为“硝石堆”；该键由酸蚀洞穴池塘的 `displaynamefn` 使用（池塘可开采时返回此名），晶簇本体不带 inspectable。[名称键](../../../raw/current/scripts/languages/chinese_s.po#L346195-L346197)、[宿主显示名](../../../raw/current/scripts/prefabs/pond.lua#L465-L467)。其余三注册在 .po 无 `STRINGS.NAMES` 键；本页标题中文是已核验的效果描述，“硝石堆”仅是 `nitre_formation` 对应的官方名称键，不代表整族。

贴地／贴水边界：`ice_puddle` 未调用 `SetOrientation`，贴地观感来自背景层、排序 3 与父挂后的动画资源；`ice_splash` 连层与排序都未设置。二者都不按常见的 `ANIM_ORIENTATION.OnGround` 贴地配置构造，引用时不要替源码补设朝向。[ice_puddle 层与排序](../../../raw/current/scripts/prefabs/ice_puddle.lua#L17-L21)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 生成者：`rock_ice`（冰面巨石） | 在 `_puddle` 分支 `SpawnPrefab("ice_puddle")` 并 `SetParent` 到巨石，阶段变化时重播水洼动画；`_ismelt` 为真、未睡眠且目标阶段非 dryup 时，在巨石位置生成 `ice_splash` 并覆写为阶段动画 | 仅非专用端（`not TheNet:IsDedicated()`）；水花同受该分支约束；巨石开采与季节玩法不展开（Wiki 待建） | [水洼生成](../../../raw/current/scripts/prefabs/rock_ice.lua#L352-L358)、[阶段动画与水花](../../../raw/current/scripts/prefabs/rock_ice.lua#L80-L90)、[阶段切换](../../../raw/current/scripts/prefabs/rock_ice.lua#L69-L93) |
| 生成者：`rock_ice_temperature` | 与 `rock_ice` 同构：`_puddle` 父挂＋阶段动画，`_ismelt` 时生成水花；`SetPrefabNameOverride("rock_ice")`，对外显示与 scrapbook 代理同名 | 非专用端；温度玩法不展开（Wiki 待建） | [名字覆写与生成](../../../raw/current/scripts/prefabs/rock_ice_temperature.lua#L289-L295)、[阶段动画与水花](../../../raw/current/scripts/prefabs/rock_ice_temperature.lua#L73-L83) |
| 生成者：`sharkboi_ice_hazard`（大霜鲨竞技场冰块） | 只生成 `ice_splash`：阶段融化且 `_shouldlocalfx` 时在冰块位置播阶段动画；`CleanupForced` 只检查未休眠，主模拟清理时也补一次当前阶段动画后 `Remove` 自身 | `_shouldlocalfx` 只在非专用端置真，只约束普通融化分支；`CleanupForced` 不查该值。不生成 `ice_puddle`；竞技场玩法不展开（Wiki 待建） | [融化水花](../../../raw/current/scripts/prefabs/sharkboi_ice_hazard.lua#L74-L80)、[强制清理](../../../raw/current/scripts/prefabs/sharkboi_ice_hazard.lua#L278-L285)、[端开关](../../../raw/current/scripts/prefabs/sharkboi_ice_hazard.lua#L342-L343) |
| 生成者：`pond_cave`（酸蚀洞穴池塘） | `SpawnNitreFormations` 随机 3–4 个环形偏移（半径约 0.25–1.5），逐个 `SpawnPrefab("nitre_formation")`、父挂池塘、按缓存数据覆写 `idle1`–`idle3` 动画、`persists=false` 后收进 `nitreformation_ents`；非专用端把该表赋给 `highlightchildren`；酸蚀结束 `DespawnNitreFormations` 逐个 `Remove` | 服务端生成、联网复制；酸蚀越过阈值时进入，退到 0 时退出；停酸雨只在尚未越过阈值（`acidinfused` 为假）时回正常 | [生成](../../../raw/current/scripts/prefabs/pond.lua#L30-L67)、[清理](../../../raw/current/scripts/prefabs/pond.lua#L69-L80)、[触发](../../../raw/current/scripts/prefabs/pond.lua#L422-L426)、[退场](../../../raw/current/scripts/prefabs/pond.lua#L395) |
| 生成者：浮标 `*_floater` | `floater_fn` 在非专用端 `SpawnPrefab("oceanfishinghook_ripple")`、`AddChild` 到浮标并置于原点；每个浮标的依赖列表都含该注册 | 非专用端；同文件 11 项 `BOBBERS` 表生成 30 个玩法注册名（projectile／floater／可选物品），不属本页 | [涟漪生成](../../../raw/current/scripts/prefabs/oceanfishingbobber.lua#L211-L215)、[每个浮标的依赖](../../../raw/current/scripts/prefabs/oceanfishingbobber.lua#L330-L342)、[表驱动](../../../raw/current/scripts/prefabs/oceanfishingbobber.lua#L321-L353) |
| 消费者：`highlight` 组件 | 高亮池塘时把颜色同时应用到 `inst.AnimState` 与 `inst.highlightchildren` 的每个子实体；`nitre_formation` 的客户端登记让晶簇跟着变色，移除时清除自身 | 客户端表现；登记只在父 prefab 为 `pond_cave` 时发生 | [高亮应用](../../../raw/current/scripts/components/highlight.lua#L88-L92)、[移除恢复](../../../raw/current/scripts/components/highlight.lua#L135-L139)、[登记条件](../../../raw/current/scripts/prefabs/nitre_formation.lua#L15-L27) |
| 依赖声明（不证明生成） | `rock_ice`、`rock_ice_temperature` 的 prefabs 表都声明 `ice_puddle`、`ice_splash`；`sharkboi_ice_hazard` 只声明 `ice_splash`；`pond_cave` 声明 `nitre_formation`；每个浮标声明 `oceanfishinghook_ripple` | 仅加载／资源依赖；实际生成点见上 | [rock_ice](../../../raw/current/scripts/prefabs/rock_ice.lua#L7-L12)、[温度版](../../../raw/current/scripts/prefabs/rock_ice_temperature.lua#L7-L12)、[大霜鲨冰块](../../../raw/current/scripts/prefabs/sharkboi_ice_hazard.lua#L7-L11)、[池塘](../../../raw/current/scripts/prefabs/pond.lua#L23-L28) |
| 对比：`ink_puddle_land` | 同为落到世界点位的洼地表现；它走通用 `fx.lua` 表、`GroundOrientation` 与 animover 自删，`ice_puddle` 是独立工厂、父挂宿主且不自删 | 选型：一次性落点选前者，随宿主长期存在选后者 | [[reference/prefabs/ink_puddle_land|墨水与胆汁洼地]]、[ice_puddle 工厂](../../../raw/current/scripts/prefabs/ice_puddle.lua#L6-L24) |
| 资源 | `anim/ice_puddle.zip`、`anim/ice_splash.zip`、`anim/pond_nitrecrystal.zip`（bank 为 `pond_rock`）、`anim/oceanfishing_hook.zip` | 动画时长、循环标记与视觉未核验；不分发资源 | [ice_puddle 资源](../../../raw/current/scripts/prefabs/ice_puddle.lua#L1-L4)、[nitre 资源](../../../raw/current/scripts/prefabs/nitre_formation.lua#L1-L4)、[ripple 资源](../../../raw/current/scripts/prefabs/oceanfishingbobber.lua#L316-L319) |

## 典型接入

### 给宿主实体配贴地水洼（ice_puddle）

- 前提与选择：需要“随宿主长期存在、不自行消失”的贴地表现时用 `ice_puddle`；宿主可为任意带 Transform 的实体，原版即父挂到冰面巨石。
- 生效与撤销：父挂后跟随宿主变换；工厂无 animover，不会自行消失。`SetParent` 不登记 `EntityScript.children`，宿主删除时是否随父实体销毁未核验，自建宿主应显式 `Remove`。生成者可在阶段变化时重播动画。
- 原版依据：[水洼生成](../../../raw/current/scripts/prefabs/rock_ice.lua#L352-L358)、[阶段动画](../../../raw/current/scripts/prefabs/rock_ice.lua#L80-L84)

```lua
local fx = SpawnPrefab("ice_puddle")
fx.entity:SetParent(inst.entity)      -- inst 为宿主实体；仅父挂跟随，删除级联未核验
fx.Transform:SetPosition(0, 0, 0)     -- 子实体局部坐标，原版即巨石原点
fx.AnimState:PlayAnimation("full")    -- 生成者随后按阶段改写
```

- 调整与验证：在非专用端生成（原版在 `not TheNet:IsDedicated()` 分支）；动画名沿用阶段表（`full`／`med`／`low`／`melted` 等，见 `rock_ice` 的 `STAGES`）。示例只做语法检查，未在游戏内执行。

### 在融化点播放一次性水花（ice_splash）

- 前提与选择：只需要一次溅起表现、播完自动消失时用 `ice_splash`；不父挂，落在世界坐标。
- 生效与撤销：`animover` 触发 `inst.Remove` 自删；`SetCanSleep(false)` 使该实体不进入休眠。
- 原版依据：[水花生成与覆写动画](../../../raw/current/scripts/prefabs/rock_ice.lua#L86-L90)、[自删监听](../../../raw/current/scripts/prefabs/ice_splash.lua#L21-L21)

```lua
local x, y, z = inst.Transform:GetWorldPosition()
local fx = SpawnPrefab("ice_splash")
fx.Transform:SetPosition(x, y, z)
fx.AnimState:PlayAnimation("melted")  -- 调用方按阶段覆写；播完 animover 自删
```

- 调整与验证：工厂默认播 `full`，实际所见由生成点覆写；普通融化分支只在非专用端生成，`CleanupForced` 例外，见陷阱。示例只做语法检查，未在游戏内执行。

### 给浮标挂贴水涟漪（oceanfishinghook_ripple）

- 前提与选择：需要浮标脚下的循环水纹时用 `oceanfishinghook_ripple`；它是贴地朝向、水下层的非联网表现。
- 生效与撤销：`AddChild` 后成为浮标子实体，随浮标一起移除；循环动画没有 animover，不会自行结束。
- 原版依据：[涟漪生成](../../../raw/current/scripts/prefabs/oceanfishingbobber.lua#L211-L215)、[层与循环](../../../raw/current/scripts/prefabs/oceanfishingbobber.lua#L299-L309)

```lua
local ripple = SpawnPrefab("oceanfishinghook_ripple")
inst:AddChild(ripple)                 -- inst 为浮标实体；随浮标移除
ripple.Transform:SetPosition(0, 0, 0) -- 浮标原点
```

- 调整与验证：原版在非专用端生成；`OnGround`＋`LAYER_BELOW_GROUND`＋`UNDERWATER` 排序只在海面场景有意义。示例只做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **边界：`ice_puddle` 不会自删**：工厂只有标签、动画与层设置，没有 animover 或定时清理；只生成不父挂会一直留在场景里。原版只 `SetParent` 到巨石，宿主删除时的引擎级联未核验，不能把父挂当已保证清理；需要一次性水洼应改用 `ice_splash` 或 [[reference/prefabs/ink_puddle_land|ink_puddle_land]]。[工厂](../../../raw/current/scripts/prefabs/ice_puddle.lua#L6-L24)、[父挂](../../../raw/current/scripts/prefabs/rock_ice.lua#L352-L358)
- **边界：`ice_splash` 的普通融化与强制清理不同**：普通融化要求 `_ismelt` 为真（来源为 `"melt"`）、实体未睡眠且目标阶段不是 dryup，开采导致的变化不播水花，专用服务器的该分支也不生成；`sharkboi_ice_hazard:CleanupForced` 只检查未休眠，主模拟清理时也会生成同一非联网水花。工厂自身没有端门禁，不能把普通分支条件扩成所有调用点。[普通融化](../../../raw/current/scripts/prefabs/rock_ice.lua#L86-L90)、[强制清理](../../../raw/current/scripts/prefabs/sharkboi_ice_hazard.lua#L278-L285)
- **边界：工厂默认动画不是最终所见**：`ice_splash` 工厂先播 `full`，生成点随即覆写为阶段动画（`melted`／`low`／`med`／`full` 等）；`ice_puddle` 也被宿主按阶段重播，读工厂时不要只看默认值。[覆写](../../../raw/current/scripts/prefabs/rock_ice.lua#L81-L89)
- **误区：`nitre_formation` 可以独立随手生成**：prefab 本身不设 `persists=false`（池塘生成时才设），服务端 fn 也不挂组件；它是可见的联网表现实体（工厂设 bank／build／`idle1` 与缩放），单独生成会缺少池塘侧的生命周期管理、也没人清理。高亮登记只在父 prefab 为 `pond_cave` 时发生。[工厂](../../../raw/current/scripts/prefabs/nitre_formation.lua#L29-L51)、[池塘配置](../../../raw/current/scripts/prefabs/pond.lua#L53-L59)、[登记条件](../../../raw/current/scripts/prefabs/nitre_formation.lua#L17-L26)
- **边界：三个本地 FX 不联网，原版生成路径并非全部同端**：`ice_puddle`、`ice_splash`、`oceanfishinghook_ripple` 都无 Network；普通路径多在 `not TheNet:IsDedicated()` 或 `_shouldlocalfx` 分支生成，但 `sharkboi_ice_hazard:CleanupForced` 在主模拟未休眠时也会生成 `ice_splash`，没有端门禁。非联网实例不会复制到远端，不能反过来说专服一定可见。[rock_ice 端](../../../raw/current/scripts/prefabs/rock_ice.lua#L352-L358)、[强制清理](../../../raw/current/scripts/prefabs/sharkboi_ice_hazard.lua#L278-L285)、[ripple 端](../../../raw/current/scripts/prefabs/oceanfishingbobber.lua#L211-L215)
- **边界：`oceanfishinghook_ripple` 循环到宿主移除**：动画以 `true` 循环且没有 animover，生命周期等于浮标子实体；别把它当一次性溅花，也别在没有浮标时长期保留。[循环与层](../../../raw/current/scripts/prefabs/oceanfishingbobber.lua#L299-L309)

## 验证记录

- 源码审阅：2026-11-04，steam-buildid:26335394（`raw/current/BUILD.txt` 实读）。四个工厂文件全文核对注册名、bank／build／anim、标签、层／排序、端与清理；生成点逐文件追到 `rock_ice`、`rock_ice_temperature`、`sharkboi_ice_hazard`、`pond_cave` 的 `SpawnNitreFormations`／`DespawnNitreFormations`、浮标 `floater_fn`；并核 `highlight` 消费者的 `highlightchildren` 应用路径。以 `grep -rn --include='*.lua'` 对四个完整注册名在 `raw/current/scripts` 全域反查，命中仅上述文件与 `prefablist.lua` 的文件清单；未穷举运行时拼接名。`.po` 对四名的 `STRINGS.NAMES` 键定向检索：仅 `NITRE_FORMATION` 命中（简中“硝石堆”）。`reference/components/pond` 页不存在，按要求未建链。
- 分档说明：覆盖 4 个注册、4 个文件、5 个生成宿主与两种明确收尾（自删、池塘显式移除）及一种父挂跟随（删除级联未核验），需要整体对照保留，故 `size_class: complex`。
- 游戏运行：未运行；分层、朝向与融化水花的实际视觉未实测。
- 结构检查：`validate_page.py` 通过（0 issue；116 行、17712 字节，低于 complex 档 400 行／32 KiB）；`validate_source_links.py --strict` 通过（errors 0、warnings 0；51 处源码引用、41 个唯一目标、10 个源文件；初版 2 处锚点分别起于空行与止于空行，已改为 `#L289-L295`、`#L342-L343`）。正文 3 段 Lua 示例均经 `parser_probe.exe --syntax-only` 通过（只查 Lua 5.1 语法，未执行）。
- 未覆盖：各动画包的实际帧数与循环标记、晚加入与断线重连时序、rock_ice／pond_cave／浮标与钓鱼玩法本身。

返回 [[reference/prefabs/fx-world-index|天气、地形与生态痕迹]]。
