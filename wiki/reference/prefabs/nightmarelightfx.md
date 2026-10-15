---
title: "暗影光源、幻灵头与盘旋秃鹫的外围 FX 族（nightmarelightfx、gestalt_head、circlingbuzzard）"
created: "2026-11-04"
updated: "2026-11-04"
type: prefab
summary: "8 个注册分三组：梦魇灯座／裂缝的父挂同步动画 FX（4 变体，联网无 netvar）、虚影系头部跟随层（2 变体，非联网客户端实体）与秃鹫盘旋影子（2 变体，netvar 驱动淡入淡出）；逐项记工厂、端、生成点、父挂与清理。"
aliases: ["nightmarelightfx", "nightmarefissurefx", "upper_nightmarefissurefx", "fissure_grottowarfx", "gestalt_head", "gestalt_guard_head", "circlingbuzzard", "circlingbuzzard_lunar", "暗影光源", "幻灵头", "盘旋秃鹫", "外围 FX 族", "梦魇灯座光源", "梦魇裂缝光源", "虚影头", "秃鹫影子"]
tags: [prefab, fx, server-client]
size_class: complex
dst_build: "steam-buildid:26335394"
sources: ["prefabs/nightmarelightfx.lua", "prefabs/nightmarelight.lua", "prefabs/nightmarefissure.lua", "prefabs/gestalt_head.lua", "prefabs/gestalt.lua", "prefabs/gestalt_guard.lua", "prefabs/gestalt_guard_evolved.lua", "prefabs/gestalt_alterguardian_projectile.lua", "prefabs/circlingbuzzard.lua", "prefabs/buzzardspawner.lua", "prefabs/buzzard.lua", "components/mutatedbuzzardmanager.lua", "components/circler.lua", "components/mutatedbuzzardcircler.lua", "prefablist.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 暗影光源、幻灵头与盘旋秃鹫的外围 FX 族（nightmarelightfx、gestalt_head、circlingbuzzard）

## 快速判断

- 职责与适用：本页收拢 3 个工厂文件里的 **8 个外围表现注册**，全部不挂玩法组件，也没有独立的持久化职责；宿主本体（梦魇灯座、裂缝、虚影系生物、秃鹫与变异秃鹫）不展开：4 个父挂动画 FX（`nightmarelightfx`、`nightmarefissurefx`、`upper_nightmarefissurefx`、`fissure_grottowarfx`）、2 个头部跟随层（`gestalt_head`、`gestalt_guard_head`）、2 个盘旋影子（`circlingbuzzard`、`circlingbuzzard_lunar`）。[注册表](../../../raw/current/scripts/prefabs/nightmarelightfx.lua#L45-L48)、[头部注册](../../../raw/current/scripts/prefabs/gestalt_head.lua#L33-L34)、[影子注册](../../../raw/current/scripts/prefabs/circlingbuzzard.lua#L183-L184)
- 优先使用：要“宿主开／关状态联动、由父体统一驱动动画”的贴体光晕，选四个 `*fx` 变体中对应宿主的那一个；要“跟随宿主头部符号、只做视觉”的装饰层，选 `gestalt_head`（普通虚影）或 `gestalt_guard_head`（守卫系）；要“绕目标盘旋的世界残影”，普通昼夜版选 `circlingbuzzard`，始终显形的变异版选 `circlingbuzzard_lunar`。光源类别的另一族对照见 [[reference/prefabs/wormlight]]。
- 重要副作用：四个 `*fx` 的标签不是 `FX` 而是 `DECOR`＋`NOCLICK`，鼠标高亮依靠父体的 `highlightchildren`；`fissure_grottowarfx` 的客户端复制回调按 `parent.prefab == "fissure"` 判定，而宿主注册名是 `fissure_grottowar`，该变体的客户端高亮不会命中。[标签表达式](../../../raw/current/scripts/prefabs/nightmarelightfx.lua#L26-L27)、[回调判定](../../../raw/current/scripts/prefabs/nightmarelightfx.lua#L7-L12)、[宿主注册](../../../raw/current/scripts/prefabs/nightmarefissure.lua#L588-L590)
- 重要副作用：普通影子只在非夜、非冬的可唤醒时段由生成器补数量，显隐由世界状态在主模拟翻转后经 `_fadeframe`／`_isfadein` 淡入淡出；变异影子由世界管理器按迁移人口生成，常显并由 `mutatedbuzzardcircler` 的降落／击落／目标失效路径触发 `KillShadow`。[数量与时段](../../../raw/current/scripts/prefabs/buzzardspawner.lua#L135-L142)、[淡入淡出](../../../raw/current/scripts/prefabs/circlingbuzzard.lua#L49-L75)、[变异生成](../../../raw/current/scripts/components/mutatedbuzzardmanager.lua#L117-L121)
- 修改边界：只覆盖 8 个注册的工厂、端分界、生成点、父挂／跟随与清理；宿主玩法（梦魇相位与 `childspawner`、虚影战斗与捕获、秃鹫迁移与月雹事件）不展开，动画包实际帧数与视觉未核验，引擎层父子回收未运行验证。
- 名称：8 个注册名在简中 `chinese_s.po` 均无 `STRINGS.NAMES.*` 键；标题里的“暗影光源”“幻灵头”“盘旋秃鹫”是已核验的效果描述，**非官方译名**。正文用到的宿主官方名：梦魇灯座（`NIGHTMARELIGHT`，[po](../../../raw/current/scripts/languages/chinese_s.po#L346159-L346162)）、虚影（`GESTALT`，[po](../../../raw/current/scripts/languages/chinese_s.po#L343169-L343172)）、大虚影（`GESTALT_GUARD`，[po](../../../raw/current/scripts/languages/chinese_s.po#L343184-L343187)）、敌意虚影（`GESTALT_GUARD_EVOLVED`，[po](../../../raw/current/scripts/languages/chinese_s.po#L343189-L343192)）、秃鹫（`BUZZARD`，[po](../../../raw/current/scripts/languages/chinese_s.po#L340044-L340047)）、水晶冠秃鹫（`MUTATEDBUZZARD_GESTALT`，[po](../../../raw/current/scripts/languages/chinese_s.po#L346069-L346072)）；“幻灵头”沿用派发措辞，指 `gestalt_head`／`gestalt_guard_head` 本体。

## 注册与构造

三个工厂形状不同：`nightmarelightfx.lua` 用 `Make(name, bank, mouseparent)` 批量返回四个 `Prefab`；`gestalt_head.lua` 用 `fn(bank)` 收一个 bank／build 名，再以两个匿名工厂各传一个 build；`circlingbuzzard.lua` 用 `commonfn(build, common_postinit)` 分叉出普通与变异两套主模拟初始化。三者都不挂玩法组件；除 `gestalt_head` 在工厂里无条件设 `persists = false` 外，其余都在主模拟分支设置。

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `nightmarelightfx` | `Make(name, bank, mouseparent)`：CreateEntity → Transform／AnimState／Network → bank＝build＝`rock_light_fx`、先播 `idle_closed` → 按 `mouseparent` 加 `DECOR`／`FX` 并统一加 `NOCLICK` → Pristine → 客户端挂 `OnEntityReplicated`；主模拟只设 `persists=false` | `rock_light_fx` 动画包的灯座光晕；动画名与宿主同名、各自用本 bank（`idle_closed`／`open_1`／`open_2`／`idle_open`／`close_1`／`close_2`）。[工厂](../../../raw/current/scripts/prefabs/nightmarelightfx.lua#L1-L43)、[注册](../../../raw/current/scripts/prefabs/nightmarelightfx.lua#L45) |
| `nightmarefissurefx` | 同一 `Make`；bank 与 build 同为 `nightmare_crack_ruins_fx` | 对应下层裂缝宿主 `fissure_lower`；发光颜色由宿主 `Light` 承担，FX 自身无 `Light`。[注册](../../../raw/current/scripts/prefabs/nightmarelightfx.lua#L46) |
| `upper_nightmarefissurefx` | 同一 `Make`；bank 与 build 同为 `nightmare_crack_upper_fx` | 对应上层裂缝宿主 `fissure`；宿主进入 `controlled` 时把 fx 染成 (1,.7,.7)，并改用 `nightmare_crack_upper` bank。[注册](../../../raw/current/scripts/prefabs/nightmarelightfx.lua#L47)、[染色](../../../raw/current/scripts/prefabs/nightmarefissure.lua#L224) |
| `fissure_grottowarfx` | 同一 `Make`；bank 与 build 同为 `fissure_grottowarfx` | 对应活动区裂缝 `fissure_grottowar`；`mouseparent` 写死 `"fissure"`，与宿主注册名不一致，见陷阱。[注册](../../../raw/current/scripts/prefabs/nightmarelightfx.lua#L48) |
| `gestalt_head` | `fn(bank)`：CreateEntity → `FX` 标签 → `persists=false` → Transform／AnimState／Follower → bank＝build＝`brightmare_gestalt_head`、`idle` 循环 → `SetFourFaced`；**无 Network** | 资源 `anim/brightmare_gestalt_head.zip`；头部的透明度由宿主写入 `OverrideMultColour`，自身不设透明逻辑。[工厂](../../../raw/current/scripts/prefabs/gestalt_head.lua#L11-L31)、[注册](../../../raw/current/scripts/prefabs/gestalt_head.lua#L33) |
| `gestalt_guard_head` | 同一 `fn`；bank＝build＝`brightmare_gestalt_head_evolved` | 资源 `anim/brightmare_gestalt_head_evolved.zip`；供大虚影、敌意虚影与守卫系 projectile 共用。[资源](../../../raw/current/scripts/prefabs/gestalt_head.lua#L6-L9)、[注册](../../../raw/current/scripts/prefabs/gestalt_head.lua#L34) |
| `circlingbuzzard` | `commonfn("buzzard_build")`：Transform／AnimState／Network → bank `buzzard`、build `buzzard_build`、`shadow` 循环、`ANIM_ORIENTATION.OnGround`、`LAYER_BACKGROUND`、排序 3、alpha 0 → `FX` 标签 → `net_byte _fadeframe`＋`net_bool _isfadein`（`fadedirty`）→ Pristine → 客户端听 `fadedirty`；主模拟导出 `KillShadow`、`persists=false` | `normalfn` 额外加 `circler` 与 3～5 秒随机扇翅；动画 `shadow_flap_loop`／`shadow` 由 `DoFlap` 随机连播。[工厂](../../../raw/current/scripts/prefabs/circlingbuzzard.lua#L101-L152)、[注册](../../../raw/current/scripts/prefabs/circlingbuzzard.lua#L183) |
| `circlingbuzzard_lunar` | `commonfn("buzzard_lunar_build", mutated_common_postinit)`；postinit 加 SoundEmitter、`lunar_aligned`、飞行物理与地面碰撞掩码 | `mutatedfn` 额外加 `mutatedbuzzardcircler`、建造成 `buzzard_lunar_build`、2～3.5 秒更急的扇翅，并在初始化时强制淡入。[postinit](../../../raw/current/scripts/prefabs/circlingbuzzard.lua#L160-L167)、[主模拟](../../../raw/current/scripts/prefabs/circlingbuzzard.lua#L169-L181)、[注册](../../../raw/current/scripts/prefabs/circlingbuzzard.lua#L184) |

## 关联机制导航

### 生成点、端与父挂

| 生成点 | 涉及注册 | 端、阶段与条件 | 父挂／跟随 | 证据 |
|---|---|---|---|---|
| `nightmarelight`（梦魇灯座） | `nightmarelightfx` | 主模拟；宿主在 Pristine 之后、加玩法组件之前生成，宿主的相位表随后用 `inst.fx.AnimState` 驱动同一套动画 | `SetParent(host.entity)`；主模拟再写 `host.highlightchildren = { fx }`，客户端由 fx 的 `OnEntityReplicated` 在 `parent.prefab == "nightmarelight"` 时补写 | [生成](../../../raw/current/scripts/prefabs/nightmarelight.lua#L265-L268)、[相位表](../../../raw/current/scripts/prefabs/nightmarelight.lua#L80-L91)、[复制回调](../../../raw/current/scripts/prefabs/nightmarelightfx.lua#L7-L12) |
| `fissure`（上层梦魇裂缝） | `upper_nightmarefissurefx` | 主模拟；`Make` 把 fx 名作为 `prefabs` 依赖并作为参数传入，宿主在 Pristine 后 `SpawnPrefab(fxname)` | `SetParent(host.entity)`；宿主不写 `highlightchildren`，全靠 fx 客户端回调按 `parent.prefab == "fissure"` 登记 | [Make](../../../raw/current/scripts/prefabs/nightmarefissure.lua#L454-L474)、[生成](../../../raw/current/scripts/prefabs/nightmarefissure.lua#L525-L526)、[注册](../../../raw/current/scripts/prefabs/nightmarefissure.lua#L588) |
| `fissure_lower`（下层梦魇裂缝） | `nightmarefissurefx` | 同上 | 同上；回调按 `parent.prefab == "fissure_lower"` 命中 | [注册](../../../raw/current/scripts/prefabs/nightmarefissure.lua#L589) |
| `fissure_grottowar`（活动区裂缝） | `fissure_grottowarfx` | 同上；`grottowar_masterinit` 额外挂 `knownlocations` 并把 `war_home` 位置传给子体 | `SetParent(host.entity)`；客户端回调不会命中（宿主 prefab 名与 `mouseparent` 不同） | [生成](../../../raw/current/scripts/prefabs/nightmarefissure.lua#L525-L526)、[masterinit](../../../raw/current/scripts/prefabs/nightmarefissure.lua#L576-L585)、[注册](../../../raw/current/scripts/prefabs/nightmarefissure.lua#L590) |
| `gestalt`（虚影） | `gestalt_head` | 非专用端（`not TheNet:IsDedicated()`），在 Pristine 之前；专用服务器不建头 | `SetParent(host.entity)`＋`Follower:FollowSymbol(host.GUID, "head_fx", 0, 0, 0)`；写 `host.highlightchildren`，并把宿主的透明度经 `transparentonsanity.onalphachangedfn` 转给头部 | [生成](../../../raw/current/scripts/prefabs/gestalt.lua#L148-L165)、[透明度回调](../../../raw/current/scripts/prefabs/gestalt.lua#L21-L25) |
| `gestalt_guard`（大虚影） | `gestalt_guard_head` | 同上 | 同一父挂模式，跟随 `head_fx_big` 符号；宿主再把头的 `persists` 设一次 `false` | [生成](../../../raw/current/scripts/prefabs/gestalt_guard.lua#L210-L228) |
| `gestalt_guard_evolved`（敌意虚影） | `gestalt_guard_head` | 同上 | 跟随 `head_fx_big`（`FollowSymbol` 末位多传 `true`）；透明组件改由 `AddTransparentOnSanity(inst, .4)` 装配 | [生成](../../../raw/current/scripts/prefabs/gestalt_guard_evolved.lua#L393-L405)、[装配](../../../raw/current/scripts/prefabs/gestalt_guard_evolved.lua#L347-L355) |
| `gestalt_alterguardian_projectile` 等 5 个 projectile 变体 | `gestalt_head`（仅 `gestalt_alterguardian_projectile`）／`gestalt_guard_head`（其余 4 个） | 非专用端；`commonfn` 内、Pristine 之前按 `headdata.name` 生成 | 同一 `FollowSymbol` 模式（`head_fx`／`head_fx_big`）；小守卫与帽守卫再把自己的缩放同步到头。[映射](../../../raw/current/scripts/prefabs/gestalt_alterguardian_projectile.lua#L291-L314)、[生成](../../../raw/current/scripts/prefabs/gestalt_alterguardian_projectile.lua#L238-L245)、[注册](../../../raw/current/scripts/prefabs/gestalt_alterguardian_projectile.lua#L473-L477)、[缩放](../../../raw/current/scripts/prefabs/gestalt_alterguardian_projectile.lua#L318-L324) |
| `buzzardspawner`（秃鹫生成器） | `circlingbuzzard` | 主模拟；生成器在 `OnAddChild` 与唤醒任务里按 `childspawner.childreninside` 对齐影子数量，且跳过夜间／冬季（月雹期间仍可出现） | 不父挂；`circler:SetCircleTarget(spawner)`＋`Start()`，影子绕生成器盘旋 | [生成](../../../raw/current/scripts/prefabs/buzzardspawner.lua#L25-L30)、[同步数量](../../../raw/current/scripts/prefabs/buzzardspawner.lua#L32-L44)、[时段](../../../raw/current/scripts/prefabs/buzzardspawner.lua#L135-L142)、[触发](../../../raw/current/scripts/prefabs/buzzardspawner.lua#L303-L308) |
| `mutatedbuzzardmanager`（变异秃鹫管理器） | `circlingbuzzard_lunar` | 主模拟；按迁移人口与每名玩家 `_num_circling_buzzards < 10` 生成，世界事件驱动 | 不父挂；`mutatedbuzzardcircler:SetCircleTarget(player)`＋`Start()`，影子绕玩家盘旋 | [生成](../../../raw/current/scripts/components/mutatedbuzzardmanager.lua#L117-L121)、[上限与遍历](../../../raw/current/scripts/components/mutatedbuzzardmanager.lua#L165-L173) |

### 组件、同步与资源

| 类型／对象 | 本组注册如何使用 | 条件与边界 | Wiki／源码入口 |
|---|---|---|---|
| 组件：`circler` | 只装在 `circlingbuzzard` 主模拟分支；生成器设目标并启动，绕生成器自身盘旋 | 影子不自行启动；直接生成不会自动获得目标 | [[reference/components/circler]]；[挂载](../../../raw/current/scripts/prefabs/circlingbuzzard.lua#L147-L149)、[目标与启动](../../../raw/current/scripts/components/circler.lua#L30-L57) |
| 组件：`mutatedbuzzardcircler` | 只装在 `circlingbuzzard_lunar` 主模拟分支；绕玩家盘旋，按尸体／信号弹／迁移节点路径投放真身并 `KillShadow` | 真秃鹫是 `inst.buzzard` 引用的另一实体，影子本身无战斗 | [[reference/components/mutatedbuzzardcircler]]；[挂载](../../../raw/current/scripts/prefabs/circlingbuzzard.lua#L176-L178) |
| 组件：`transparentonsanity` | 不是头 FX 的组件，而是四个宿主（虚影、大虚影、敌意虚影、projectile）在非专用端安装；`onalphachangedfn` 把 alpha 写给 `blobhead` | 只在客户端视图侧；专用服务器没有该组件，也没有头 | [[reference/components/transparentonsanity]]；[宿主装配](../../../raw/current/scripts/prefabs/gestalt.lua#L158-L164) |
| 引擎 `Follower` | 两个头部工厂用 `inst.entity:AddFollower()` 获得引擎跟随器；宿主调 `Follower:FollowSymbol(host.GUID, 符号名, 0, 0, 0)` | 这是引擎实体跟随器，不是 `components.follower` 组件；头本身没有该组件实例 | [AddFollower](../../../raw/current/scripts/prefabs/gestalt_head.lua#L20-L22)、[跟随](../../../raw/current/scripts/prefabs/gestalt.lua#L151) |
| `highlightchildren` | `nightmarelight` 在主模拟写表；四个 `*fx` 在客户端复制回调里按父体 prefab 名补写；三个虚影宿主在非专用端直接写表 | 两套写入时机不同；`fissure_grottowarfx` 的判定名不匹配，按条件不会登记 | [灯座](../../../raw/current/scripts/prefabs/nightmarelight.lua#L268)、[fx 回调](../../../raw/current/scripts/prefabs/nightmarelightfx.lua#L7-L12)、[虚影](../../../raw/current/scripts/prefabs/gestalt.lua#L155) |
| 同步：`net_byte`／`net_bool` | 仅 `circlingbuzzard` 家族有自定义 netvar：`_fadeframe`（淡入淡出帧）与 `_isfadein`（方向），共用 `fadedirty` | 内部字段，不作为公开接口；四个 `*fx` 联网只为同步宿主写入的动画状态，没有 netvar | [netvar](../../../raw/current/scripts/prefabs/circlingbuzzard.lua#L118-L120)、[联网注释](../../../raw/current/scripts/prefabs/nightmarelightfx.lua#L19-L23) |
| 宿主组件：`childspawner` | 三个生成器用它记账：`buzzardspawner` 的影子数量对齐 `childreninside`；`nightmarelight`／`nightmarefissure` 用它生成梦魇生物 | 影子与 fx 都由 `SpawnPrefab` 单独生成，不走 `childspawner:SpawnChild` | [[reference/components/childspawner]]；[影子对齐](../../../raw/current/scripts/prefabs/buzzardspawner.lua#L32-L44) |
| 资源 | `rock_light_fx.zip`、`nightmare_crack_ruins_fx.zip`、`nightmare_crack_upper_fx.zip`、`fissure_grottowarfx.zip`、`brightmare_gestalt_head.zip`、`brightmare_gestalt_head_evolved.zip`、`buzzard_shadow.zip`＋`buzzard_build.zip`／`buzzard_lunar_build.zip` | 只核验资源声明名与工厂传入的 bank／build 字符串；分包内的实际内容、帧数与循环标记未核验，不分发资源 | [四个 fx 资源](../../../raw/current/scripts/prefabs/nightmarelightfx.lua#L2-L5)、[头部资源](../../../raw/current/scripts/prefabs/gestalt_head.lua#L1-L9)、[影子资源](../../../raw/current/scripts/prefabs/circlingbuzzard.lua#L1-L11) |

### 清理路径

| 注册组 | 清理方式 | 边界与证据 |
|---|---|---|
| 四个 `*fx` | 没有自删任务、没有 `OnRemoveEntity`、宿主也没有 `inst.fx:Remove()` 调用；只靠 `SetParent` 与主模拟的 `persists=false` | 源码内唯一的生命期约束是父挂；父体被删除时子体是否由引擎一并回收，本页未运行验证。[工厂尾部](../../../raw/current/scripts/prefabs/nightmarelightfx.lua#L29-L39)、[宿主生成](../../../raw/current/scripts/prefabs/nightmarelight.lua#L265-L268) |
| 两个头部 | 同样没有移除钩子；`gestalt_head` 的 `persists=false` 在工厂里无条件设置，宿主另有一次设置 | 非联网实体，随宿主客户端实体存在；父体删除后的回收同属引擎层，未运行验证。[工厂](../../../raw/current/scripts/prefabs/gestalt_head.lua#L14-L18) |
| `circlingbuzzard` | `KillShadow()`：未隐藏且未睡眠时先淡出，`_killed` 置真，淡到 0 帧时 `Remove`；已经隐藏／睡眠则直接 `Remove` | `KillShadow` 只在主模拟分支赋值，客户端没有该方法。[实现](../../../raw/current/scripts/prefabs/circlingbuzzard.lua#L87-L99)、[赋值](../../../raw/current/scripts/prefabs/circlingbuzzard.lua#L128-L135) |
| `circlingbuzzard`（生成器侧） | 生成器睡眠时遍历 `buzzardshadows` 逐个 `Remove` 并清表 | 睡眠不淡出；与 `KillShadow` 是两条不同入口。[睡眠清理](../../../raw/current/scripts/prefabs/buzzardspawner.lua#L189-L201) |
| `circlingbuzzard_lunar` | 组件在尸体降落、信号弹击落、目标死亡／移除、迁移节点失效时调 `KillShadow`；管理器在清空人口时遍历 `ent.shadow:KillShadow()` | 影子与真身互相清引用；影子结束时管理器从登记表注销（同时监听 `shadowkilled` 与 `onremove`）。[击落→删除](../../../raw/current/scripts/components/mutatedbuzzardcircler.lua#L229-L255)、[清空](../../../raw/current/scripts/components/mutatedbuzzardmanager.lua#L175-L181)、[注销监听](../../../raw/current/scripts/components/mutatedbuzzardmanager.lua#L79-L96) |

## 典型接入

### 复刻“宿主生成、父挂、不自删”的同步动画 FX

- 前提与选择：需要一个“跟随宿主、由宿主统一改动画、生命周期挂在宿主上”的纯视觉层时，复用 `nightmarelightfx` 的组装：主模拟生成 → `SetParent(host.entity)` → 宿主写 `highlightchildren`。原版梦魇灯座就是这个模式，并且四盏灯共用同一个工厂。[生成](../../../raw/current/scripts/prefabs/nightmarelight.lua#L265-L268)
- 生效与撤销：只在主模拟生成；`persists=false` 不跨存档；没有自删任务，撤销靠父体生命周期或自行 `Remove`。若自定义宿主希望客户端也参与鼠标高亮，需要自己写复制回调——原版回调只认 `parent.prefab == "nightmarelight"`。[回调](../../../raw/current/scripts/prefabs/nightmarelightfx.lua#L7-L12)
- 原版依据：`Make` 已把 `DECOR`＋`NOCLICK` 打包进工厂，复用生成后不必再改标签；宿主动画名与 fx 动画名一致，宿主只调 `inst.fx.AnimState`。[标签](../../../raw/current/scripts/prefabs/nightmarelightfx.lua#L26-L27)、[相位驱动](../../../raw/current/scripts/prefabs/nightmarelight.lua#L80-L91)

```lua
local function attach_light_fx(host)
    if not TheWorld.ismastersim then
        return nil
    end
    local fx = SpawnPrefab("nightmarelightfx")
    fx.entity:SetParent(host.entity)
    host.fx = fx
    host.highlightchildren = { fx }
    return fx
end
```

- 调整与验证：替换点是 `SpawnPrefab` 的注册名与宿主的动画调用；客户端高亮需另配复制回调。示例仅做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **误区：`nightmarelightfx` 家族带 `FX` 标签。** 四个变体的 `mouseparent` 都非空，标签表达式实际取 `DECOR`，另加 `NOCLICK`；源码注释说明 `FX` 会吃鼠标悬停而 `DECOR` 不会。照抄时若改成 `FX` 会改变悬停行为。[标签](../../../raw/current/scripts/prefabs/nightmarelightfx.lua#L26-L27)、[注册](../../../raw/current/scripts/prefabs/nightmarelightfx.lua#L45-L48)
- **边界：`fissure_grottowarfx` 的客户端高亮不会命中。** `mouseparent` 参数是 `"fissure"`，但宿主注册名是 `fissure_grottowar`；复制回调的 `parent.prefab == mouseparent` 不成立，`highlightchildren` 不会被补写。若要修，需把参数改成实际宿主名或另写回调。[回调](../../../raw/current/scripts/prefabs/nightmarelightfx.lua#L7-L12)、[宿主注册](../../../raw/current/scripts/prefabs/nightmarefissure.lua#L590)
- **边界：四个光晕 FX 不自删、无显式清理。** 工厂里没有定时任务或移除钩子，宿主也没有 `inst.fx:Remove()`；`persists=false` 只保证不存档。父体删除后的回收依赖引擎父子关系，未运行核验。[工厂](../../../raw/current/scripts/prefabs/nightmarelightfx.lua#L29-L39)
- **边界：`gestalt_head` 是非联网实体，只在非专用端生成。** 工厂没有 `AddNetwork`；三个虚影宿主与 projectile 都用 `not TheNet:IsDedicated()` 包住生成，专用服务器上不存在头部；每台客户端各自建头，自定义宿主不要假设服务端存在该头实体。[非联网](../../../raw/current/scripts/prefabs/gestalt_head.lua#L14-L18)、[生成条件](../../../raw/current/scripts/prefabs/gestalt.lua#L148-L150)
- **边界：头部跟随用引擎 `Follower`，不是 `components.follower`。** `AddFollower()` 加的是引擎跟随器，宿主直接调 `Follower:FollowSymbol`；同名组件页讲的 `components.follower` 生命周期不适用。[工厂](../../../raw/current/scripts/prefabs/gestalt_head.lua#L20-L22)、[跟随](../../../raw/current/scripts/prefabs/gestalt_guard.lua#L211-L213)
- **边界：头部自身不改缩放。** 工厂不设 `Transform:SetScale`；缩放由宿主决定，守卫系 projectile 会把自身缩放同步给头（小守卫 .75、帽守卫 .4）。[头工厂](../../../raw/current/scripts/prefabs/gestalt_head.lua#L24-L28)、[同步缩放](../../../raw/current/scripts/prefabs/gestalt_alterguardian_projectile.lua#L318-L324)
- **误区：直接 `SpawnPrefab("circlingbuzzard")` 就能得到盘旋效果。** 工厂只加组件，不设目标也不 `Start`；生成器缺位时影子停在生成位置，只会按世界状态淡入淡出并随机扇翅。[工厂主模拟](../../../raw/current/scripts/prefabs/circlingbuzzard.lua#L140-L152)、[生成器组装](../../../raw/current/scripts/prefabs/buzzardspawner.lua#L25-L30)
- **边界：普通影子在冬季与夜间不出现。** 冬季分支停止监听 `isnight` 并强制淡出；生成器也只在“未睡眠、非夜、非冬”时补数量（月雹期间例外）。变异影子相反：初始化即 `_isfadein=true`，昼夜都淡入。[世界状态](../../../raw/current/scripts/prefabs/circlingbuzzard.lua#L62-L75)、[生成器时段](../../../raw/current/scripts/prefabs/buzzardspawner.lua#L135-L142)、[变异常显](../../../raw/current/scripts/prefabs/circlingbuzzard.lua#L154-L158)
- **边界：`KillShadow` 只在主模拟存在。** 赋值在 `if not TheWorld.ismastersim then ... end` 之后；客户端没有该方法，跨端调用会取到 `nil`。淡出完成后的 `Remove` 也只由主模拟执行。[赋值位置](../../../raw/current/scripts/prefabs/circlingbuzzard.lua#L128-L137)
- **误区：`_fadeframe`／`_isfadein` 是公开接口。** 它们是内部 netvar，只由世界状态、`KillShadow` 与变体初始化写入；直接改写会跳过 `OnFadeDirty` 的任务建立逻辑。[netvar](../../../raw/current/scripts/prefabs/circlingbuzzard.lua#L118-L120)、[脏回调](../../../raw/current/scripts/prefabs/circlingbuzzard.lua#L49-L54)

## 验证记录

- 源码审阅：2026-11-04，Steam buildid 26335394（实读 `raw/current/BUILD.txt`）。完整读取 `nightmarelightfx.lua`、`gestalt_head.lua`、`circlingbuzzard.lua`，并按需读取三个宿主工厂、5 个 projectile 变体与两个转向组件；对 8 个完整注册名以 `grep -rnF '"注册名"' --include=*.lua` 在 `raw/current/scripts` 全域反查。命中汇总：`nightmarelightfx` 4 处（工厂注册、`nightmarelight` 的依赖表与生成、`prefablist` 文件项），另 3 个 fissure fx 各 2 处（工厂注册＋`nightmarefissure` 的 `Make` 参数）；`gestalt_head` 命中 `gestalt`、`gestalt_guard`、`gestalt_guard_evolved`、`gestalt_alterguardian_projectile` 的依赖表与生成点，`gestalt_guard_head` 命中 `gestalt_guard`、`gestalt_guard_evolved` 与 4 个 guard 系 projectile 的生成点；`circlingbuzzard` 命中工厂、`buzzardspawner` 依赖与生成、`prefablist`，`circlingbuzzard_lunar` 命中工厂、[`buzzard.lua` 的 `mutated_prefabs` 依赖声明](../../../raw/current/scripts/prefabs/buzzard.lua#L187)、`mutatedbuzzardmanager` 生成点与 `debugcommands.lua:4538`（调试命令，非玩法生成，本页不展开）。`prefablist.lua` 的 `PREFABFILES` 是文件级清单，三个工厂文件各占一项（[L294](../../../raw/current/scripts/prefablist.lua#L294)、[L494](../../../raw/current/scripts/prefablist.lua#L494)、[L850](../../../raw/current/scripts/prefablist.lua#L850)），其余同文件注册随整份文件加载。未穷举运行时拼接名。
- 名称核验：8 个注册名在 `languages/chinese_s.po` 无 `STRINGS.NAMES.*` 键；宿主名键与锚点见“快速判断”。`fissure` 系宿主无 `NAMES` 键，只有世界自定义项 `STRINGS.UI.CUSTOMIZATIONSCREEN.FISSURE`＝“梦魇裂缝”，本页未把它当作实体名。
- 分档说明：本页跨 3 个工厂文件、8 个注册、4 类直接宿主＋5 个 projectile 变体、三种端模型（联网同步动画／非联网客户端／netvar 淡入淡出）与三条清理路径（引擎父挂、`KillShadow` 淡出、生成器睡眠直删），需要在一页内整体对照，故取 `size_class: complex`；实测 133 行／29595 字节（约 28.9 KiB），在 480 行／48 KiB 内。
- 未覆盖：动画 zip 的实际帧数、循环标记与视觉观感；四个 `*fx` 与两个头部在父体删除时的引擎级回收；`fissure_grottowar` 场景与 `knownlocations` 子体定位；专用服务器／多客户端下头的本地生成时序；`circlingbuzzard` 的物理与盘旋观感。
- 结构检查：`validate_page.py` 通过（0 issue；133 行、29595 字节，`complex` 档上限 480 行／48 KiB）；`validate_source_links.py --strict` 通过（errors 0、warnings 0；106 处行级引用、84 个唯一目标、16 个源文件）；Lua 示例经 `parser_probe.exe --syntax-only` 通过（只查语法，不执行）。

## 相关链接

- [[reference/prefabs/wormlight|灯具与生物光源的附属光／FX 实体]]（同为宿主可挂接的光源／FX 族，可对照“代理＋FX”与“父挂即生命期”两种做法）
- [[reference/prefabs/gestalt_trail|宿主附着与跟随的外围 FX 族]]（同宿主 `gestalt`／`gestalt_guard` 的拖尾生成点；同样不挂玩法组件）
- [[reference/components/circler]]、[[reference/components/mutatedbuzzardcircler]]（两种盘旋组件）
- [[reference/components/transparentonsanity]]、[[reference/components/childspawner]]（宿主侧装配与计数）
- 返回 [[reference/prefabs/fx-light-index|灯光、火焰与音效代理]]
