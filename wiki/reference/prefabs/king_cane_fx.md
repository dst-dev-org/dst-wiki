---
title: "宿主持有与跟随的附属 FX 族（king_cane_fx 等）"
created: "2026-11-04"
updated: "2026-11-04"
type: prefab
size_class: complex
summary: "六个宿主侧生成或持有的附属 FX：国王魔杖宝石／腐蚀层、风铃装饰克隆、牧者岩核、致命亮茄背层、月装置内嵌连接件与非自然传送门掉落跟随光；哈奇音乐地光另见光源页；端、父挂、网络参数与清理各异。"
aliases: ["king_cane_fx", "hermithouse_ornament_fx", "lunar_grazer_core_fx", "lunarthrall_plant_back", "moon_altar_link_contained", "monkeyisland_portal_lootfollowfx", "附属 FX 族", "外围 FX 族", "宿主持有 FX", "跟随特效"]
tags: [prefab, fx, server-client]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/king_cane.lua", "prefabs/hermithouse_ornament.lua", "prefabs/hermithouse.lua", "prefabs/hermithouse_laundry.lua", "prefabs/wagstaff_items.lua", "prefabs/lunar_grazer.lua", "prefabs/lunarthrall_plant.lua", "prefabs/moon_altar_link.lua", "prefabs/moon_device.lua", "prefabs/monkeyisland_portal.lua", "prefabs/hutch.lua", "components/highlightchild.lua", "components/colouraddersync.lua", "components/highlight.lua", "entityscript.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 宿主持有与跟随的附属 FX 族（king_cane_fx 等）

## 快速判断

- 职责与适用：本页收六个来自不同宿主的附属 FX 注册：`king_cane_fx`（国王魔杖的宝石与腐蚀层）、`hermithouse_ornament_fx`（风铃／洗衣绳／Wagstaff 装饰的克隆表现）、`lunar_grazer_core_fx`（牧者岩核）、`lunarthrall_plant_back`（致命亮茄背层）、`moon_altar_link_contained`（月装置内嵌连接件）、`monkeyisland_portal_lootfollowfx`（非自然传送门掉落跟随光）；哈奇音乐地光 `hutch_music_light_fx` 本页只作对照，注册归 [[reference/prefabs/wormlight|灯具与生物光源]]。六者都不挂生命／工作／库存等玩法组件，宿主负责生成与父挂；宿主的玩法本体不在本页。与 [[reference/prefabs/fence_gate_anim|宿主挂接的附属表现实体]] 的静态子实体相比，本族还包含带网络参数、本地子实体和按符号跟随的变体。
- 优先使用：按需求直达——宝石／腐蚀外观选 `king_cane_fx`；隐士屋装饰及其三族克隆选 `hermithouse_ornament_fx`；牧者核心选 `lunar_grazer_core_fx`；亮茄背层选 `lunarthrall_plant_back`；月装置下方的连接光柱选 `moon_altar_link_contained`；传送门吐物光效选 `monkeyisland_portal_lootfollowfx`；哈奇音乐地光见 [[reference/prefabs/wormlight|灯具与生物光源]]。
- 重要副作用：`king_cane_fx` 读取宿主腐蚀值分三档（33／67／100），并通过 `colouraddersync` 把持有者的附加色带到宝石／腐蚀子层；`monkeyisland_portal_lootfollowfx` 会让被附着的 loot 临时获得 `outofreach` 标签并提高 `LightOverride`，动画长度后复原；`hutch_music_light_fx` 在音乐形态首次满足电池条件时生成 3 个地面辉光，音乐电池失效时移除。其余注册不改变玩法状态。
- 修改边界：本页只覆盖 `raw/current/scripts` 的工厂、端分支、父挂／跟随、网络参数与清理；宿主本体（国王魔杖、隐士屋、牧者、亮茄、月装置、传送门、哈奇）均为 Wiki 待建，不在此展开。所有结论为源码审阅，未运行游戏；资源包内容与跨端时序未核验。覆盖 7 个注册、7 个工厂与 7 类宿主，故按 `size_class: complex` 保留整体对照。
- 名称：七个注册名在简中 `.po` 均无 `STRINGS.NAMES` 键，标题与条目中的中文是已核验的效果描述，不是官方译名。宿主名（国王魔杖／风铃／牧者／致命亮茄／神秘能量／非自然传送门／哈奇）命中官方名称键，只用于定位宿主，不把宿主名当作 FX 的官方名。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `king_cane_fx` | `fxfn`：Transform／AnimState／Follower／Network → `FX` 标签 → bank/build `king_cane`、`swap_loop1` 循环 → `highlightchild`（`SetOnChangeOwnerFn`）＋`colouraddersync` → 3 个显式 netvar（`gemshine`／`level`／`gem`）→ 非专服建本地 `corruptfx`／`gemfx` 子实体；主模拟导出 `DoGemShine`／`SetGemType`／`SetFXLevel`，`persists=false`。[工厂](../../../raw/current/scripts/prefabs/king_cane.lua#L539-L589)、[注册](../../../raw/current/scripts/prefabs/king_cane.lua#L591-L592)、[netvar](../../../raw/current/scripts/prefabs/king_cane.lua#L557-L560) | 宿主在服务端 `fn` 生成后立即调 `SetFxOwner(inst, nil)`，浮空时父挂本体并跟随 `swap_spear`；装备时改挂持有者，符号为 `swap_object`，`moonbase` 特例为 `follow_king_cane`。腐蚀值按 33／67／100 映射 level 1／2／3；宝石用 `swap_<gem>` 覆写 `gemfx` 符号。[owner 切换](../../../raw/current/scripts/prefabs/king_cane.lua#L42-L65)、[生成](../../../raw/current/scripts/prefabs/king_cane.lua#L305-L313)、[阈值](../../../raw/current/scripts/tuning.lua#L9741-L9743) |
| `hermithouse_ornament_fx` | `fxfn`：Transform／AnimState／Follower／SoundEmitter／Network → `FX` 标签 → bank/build `hermithouse_ornament_shell`、`idle_loop` 循环 → 客户端 `OnEntityReplicated` 登记；服务端暴露 `AttachToParent`／`dowind`，`persists=false`；注册不带 assets／prefabs 表。[工厂](../../../raw/current/scripts/prefabs/hermithouse_ornament.lua#L84-L115)、[注册](../../../raw/current/scripts/prefabs/hermithouse_ornament.lua#L175-L176) | 宿主物品的 `CloneAsFx` 用 `SpawnPrefab(name, skin_build, skin_id)` 注入皮肤，按 `skin_build` 第 22 字符起的后缀拼 `soundpath`；隐士屋装饰槽、洗衣绳（socks／shorts）与 Wagstaff 个人物品（`wagstaff_item_1`）三族复用。`AddFakeLaundry`／`AddFakeOrnament` 另走直接生成、改 bank／build 或传皮肤。[克隆](../../../raw/current/scripts/prefabs/hermithouse_ornament.lua#L117-L124)、[宿主接线](../../../raw/current/scripts/prefabs/hermithouse.lua#L58-L97)、[洗衣绳](../../../raw/current/scripts/prefabs/hermithouse_laundry.lua#L12-L17)、[Wagstaff](../../../raw/current/scripts/prefabs/wagstaff_items.lua#L6-L11) |
| `lunar_grazer_core_fx` | `corefn`：Transform／AnimState／Follower／Network → `FX` 标签 → 四面、bank/build `lunar_grazer`、`rock_cycle` 循环 → `persists=false`；无组件、无自删路径。[工厂](../../../raw/current/scripts/prefabs/lunar_grazer.lua#L524-L549)、[注册](../../../raw/current/scripts/prefabs/lunar_grazer.lua#L590-L592) | 仅牧者主模拟 `fn` 生成：`SpawnPrefab` → `SetParent` → `Follower:FollowSymbol(inst.GUID, "rock_cycle_follow")`。宿主 `OnRemoveEntity` 只清 debris 与追踪表，不显式移除 core。[生成](../../../raw/current/scripts/prefabs/lunar_grazer.lua#L432-L434)、[宿主清理](../../../raw/current/scripts/prefabs/lunar_grazer.lua#L161-L168) |
| `lunarthrall_plant_back` | `backfn`：Transform／AnimState／SoundEmitter／MiniMapEntity／Network → 小写 `fx` 标签 → bank/build `lunarthrall_plant_back`、`idle_med` 循环 → 客户端复制回调登记宿主 `highlightchildren`；服务端 `persists=false`；意图反登记的 `OnRemovedEntity` 拼写错误。[工厂](../../../raw/current/scripts/prefabs/lunarthrall_plant.lua#L69-L97)、[注册](../../../raw/current/scripts/prefabs/lunarthrall_plant.lua#L867-L870) | 宿主 `spawnback` 在服务端生成背层：`SetFinalOffset(-1)`、父挂宿主、`colouradder:AttachChild`、随机水平翻转与明度；宿主另在 `customSetRandomFrame` 里尝试把同一帧数同步给背层，但调用时 `inst.back` 尚未生成，该分支实际不生效（死代码）。`MiniMapEntity` 已添加但未设图标；地图图标由宿主设置。[生成](../../../raw/current/scripts/prefabs/lunarthrall_plant.lua#L99-L123)、[宿主图标](../../../raw/current/scripts/prefabs/lunarthrall_plant.lua#L428-L434) |
| `moon_altar_link_contained` | `contained_fn`：Transform／AnimState／SoundEmitter／Network → bank `moon_altar_geyser`、build `moon_geyser`、`stage1_idle` → `FX`＋`NOCLICK`＋`NOBLOCK`、`SetLightOverride(1)` → 服务端 `persists=false`、`_stage=1`、`_set_stage_fn`。[工厂](../../../raw/current/scripts/prefabs/moon_altar_link.lua#L204-L234)、[阶段设置](../../../raw/current/scripts/prefabs/moon_altar_link.lua#L197-L202)、[注册](../../../raw/current/scripts/prefabs/moon_altar_link.lua#L236-L237) | 由 `moon_device` 的 `playlinkanimation` 懒生成并 `SetParent`；阶段取装置 `inst.level`（施工 1／2、成品 3），`_set_stage_fn` 先播 `stage<N>_idle_pre` 再循环 `stage<N>_idle`。工厂初始 stage 1。[生成](../../../raw/current/scripts/prefabs/moon_device.lua#L181-L188)、[调用](../../../raw/current/scripts/prefabs/moon_device.lua#L451) |
| `monkeyisland_portal_lootfollowfx` | `followfx_fn`：Transform／AnimState／Network → `FX`／`NOBLOCK`／`NOCLICK` → bank/build `monkey_island_portal_fx`、`idle_loop` 循环；`SetFinalOffset(1)`、`SetScale(0.65, 0.65)`、乘色 alpha 0.4、bloom、`SetLightOverride(0.6)` → 服务端 `persists=false`、`fx_len` 后播 `idle_pst`，`animover` 自删。[工厂](../../../raw/current/scripts/prefabs/monkeyisland_portal.lua#L70-L102)、[收尾](../../../raw/current/scripts/prefabs/monkeyisland_portal.lua#L65-L68)、[注册](../../../raw/current/scripts/prefabs/monkeyisland_portal.lua#L445-L446) | 传送门 `attach_light_fx` 生成后用 `AddChild` 挂到 loot（不是 Follower）；局部偏移读 `PORTAL_LOOT_FXYOFFSET`，未列名者为 0。主模拟还给被附着物临时加 `outofreach` 与同值 `LightOverride`，`fx_len` 后复原。[挂接](../../../raw/current/scripts/prefabs/monkeyisland_portal.lua#L173-L182)、[偏移表](../../../raw/current/scripts/prefabs/monkeyisland_portal.lua#L48-L58) |
| `hutch_music_light_fx`（交叉引用，本页只对照；由 [[reference/prefabs/wormlight|灯具与生物光源]] 页承载） | `hutch_music_light_fx`：Transform／Follower／Network → `FX` 标签 → 5 个 `fxdirty` 参数（`_rot`／`_rotrate`／`_alpha`／`_alphadir`／`_alpharate`）＋1 个 `animdirty`（`_anim`）→ 客户端 `OnFXDirty` 建本地 `_fx`（bank/build `hutch_lightfx_base`、`idle_loop`、贴地层）并每帧按 net 值更新；服务端 `InitFX`、`persists=false`。[工厂](../../../raw/current/scripts/prefabs/hutch.lua#L633-L666)、[netvar](../../../raw/current/scripts/prefabs/hutch.lua#L642-L648)、[注册](../../../raw/current/scripts/prefabs/hutch.lua#L668-L669) | 哈奇 `MUSIC` 形态且同时有 `lightbattery`＋`band` 时，`CheckBattery` 生成 3 个地面辉光，随机缩放与正负镜像、随机旋转／透明度，`Follower:FollowSymbol(inst.GUID, "base_point")`；音乐电池失效时逐个 `Remove`。[形态](../../../raw/current/scripts/prefabs/hutch.lua#L318-L323)、[生成与接线](../../../raw/current/scripts/prefabs/hutch.lua#L173-L189)、[电池分支](../../../raw/current/scripts/prefabs/hutch.lua#L227-L258) |

## 关联机制导航

### 生成点与宿主

| 生成点 | 本注册如何使用 | 端、阶段与条件 | 证据 |
|---|---|---|---|
| 国王魔杖本体 | 本体服务端 `fn` 生成 FX 并先按浮空状态挂回本体；`OnEquip`／`OnUnequip` 重新绑定或解绑持有者；`SetFxOwner` 也导出给月岛使用。 | 主模拟生成；客户端只收网络实体。浮空跟随 `swap_spear`，装备跟随 `swap_object`／`follow_king_cane`。 | [生成](../../../raw/current/scripts/prefabs/king_cane.lua#L305-L313)、[装备](../../../raw/current/scripts/prefabs/king_cane.lua#L78-L101)、[导出](../../../raw/current/scripts/prefabs/king_cane.lua#L364-L365) |
| 隐士屋装饰槽 | `AddDecor`／`RefreshDecor` 对容器中的装饰物品调 `CloneAsFx`，再 `AttachFxToSlot` 父挂并按槽位符号跟随；`AddFakeLaundry`／`AddFakeOrnament` 供没有真实物品的槽位使用。 | 主模拟生成；`RemoveDecor`、刷新或替换时显式 `Remove` 旧 FX。 | [槽位接线](../../../raw/current/scripts/prefabs/hermithouse.lua#L58-L97)、[移除](../../../raw/current/scripts/prefabs/hermithouse.lua#L73-L78) |
| 洗衣绳与 Wagstaff 物品 | 两族各自提供 `CloneAsFx`：生成 `hermithouse_ornament_fx` 后改 bank／build 为自身动画集；Wagstaff 仅 `wagstaff_item_1` 带 `isornament`。 | 物品被放入隐士屋装饰槽时由 `AddDecor` 调；主模拟。 | [洗衣绳](../../../raw/current/scripts/prefabs/hermithouse_laundry.lua#L12-L17)、[Wagstaff](../../../raw/current/scripts/prefabs/wagstaff_items.lua#L6-L11) |
| 牧者本体 | 主模拟 `fn` 生成 core、父挂并跟随 `rock_cycle_follow`；没有 SG／事件型二次生成。 | 主模拟；`persists=false`。 | [生成](../../../raw/current/scripts/prefabs/lunar_grazer.lua#L429-L434) |
| 致命亮茄本体 | 主模拟 `fn` 末尾 `spawnback` 生成背层；客户端复制后由 `back_onentityreplicated` 登记到父实体高亮表。 | 服务端生成；客户端登记；宿主死亡动画播完再删宿主。 | [生成](../../../raw/current/scripts/prefabs/lunarthrall_plant.lua#L521)、[客户端登记](../../../raw/current/scripts/prefabs/lunarthrall_plant.lua#L55-L60)、[宿主死亡](../../../raw/current/scripts/prefabs/lunarthrall_plant.lua#L196-L217) |
| 月装置各阶段 | `playlinkanimation` 懒生成连接件并父挂；每次施工／成品构造都按 `inst.level` 调 `_set_stage_fn`。 | 主模拟；阶段 1／2／3。 | [阶段表](../../../raw/current/scripts/prefabs/moon_device.lua#L52-L54)、[生成](../../../raw/current/scripts/prefabs/moon_device.lua#L181-L188)、[调用](../../../raw/current/scripts/prefabs/moon_device.lua#L451) |
| 非自然传送门 | 周期吐物 `try_portal_spawn` 与事件 `spawn_event_loot` 在生成／抛掷 loot 后调 `attach_light_fx`；假掉落 `monkeyisland_portal_fxloot` 同样会挂。 | 主模拟；`attach_light_fx` 在 `fx_len` 非空时加临时标签与光照。 | [周期](../../../raw/current/scripts/prefabs/monkeyisland_portal.lua#L184-L194)、[事件](../../../raw/current/scripts/prefabs/monkeyisland_portal.lua#L205-L214) |
| 哈奇音乐形态 | `CheckBattery` 检测 `MUSIC` 形态、`lightbattery` 与 `band` 三者同时满足；首次从无到有时创建 3 个地面辉光并 `InitFX`。 | 主模拟生成；音乐电池消失时移除。 | [电池分支](../../../raw/current/scripts/prefabs/hutch.lua#L227-L258)、[创建](../../../raw/current/scripts/prefabs/hutch.lua#L173-L189) |

### 父挂、跟随与高亮

| 机制 | 涉及注册 | 行为与边界 | 证据 |
|---|---|---|---|
| `entity:SetParent`（直挂） | `king_cane_fx`、`hermithouse_ornament_fx`、`lunar_grazer_core_fx`、`lunarthrall_plant_back`、`moon_altar_link_contained` | 只建引擎父子变换，不登记 `EntityScript.children`；宿主 `Remove` 的子实体循环覆盖不到直挂子实体，引擎级联回收未运行核验。 | [父挂示例](../../../raw/current/scripts/prefabs/lunar_grazer.lua#L432-L434)、[AddChild／children](../../../raw/current/scripts/entityscript.lua#L921-L934)、[Remove 子实体循环](../../../raw/current/scripts/entityscript.lua#L1779-L1783) |
| `AddChild` | `monkeyisland_portal_lootfollowfx` | 登记到 loot 的 `children`；loot `Remove` 时遍历子实体逐个移除，是本族唯一有明确宿主侧联级清理的挂接。 | [挂接](../../../raw/current/scripts/prefabs/monkeyisland_portal.lua#L173-L182)、[Remove 子实体循环](../../../raw/current/scripts/entityscript.lua#L1779-L1783) |
| `Follower:FollowSymbol` | `king_cane_fx`、`hermithouse_ornament_fx`、`lunar_grazer_core_fx`、`hutch_music_light_fx` | 跟宿主符号而非仅父挂：国王魔杖按持有者／本体符号，风铃按 `follow_ornament_<slot>`，牧者按 `rock_cycle_follow`，哈奇按 `base_point`。`lunarthrall_plant_back`、`moon_altar_link_contained`、`monkeyisland_portal_lootfollowfx` 不设 Follower。 | [国王](../../../raw/current/scripts/prefabs/king_cane.lua#L51-L54)、[风铃](../../../raw/current/scripts/prefabs/hermithouse.lua#L58-L61)、[牧者](../../../raw/current/scripts/prefabs/lunar_grazer.lua#L432-L434)、[哈奇](../../../raw/current/scripts/prefabs/hutch.lua#L173-L178) |
| 高亮登记 | `king_cane_fx` 用 `highlightchild`；`hermithouse_ornament_fx`、`lunarthrall_plant_back` 手写 `highlightchildren` | `highlightchild` 自带 `net_entity` 属主同步（`highlightchild.syncowner`）与变更回调；手写路径在客户端 `OnEntityReplicated` 补登记。风铃靠自定义 `UnlinkHighlight` 反登记；背层回调因 `OnRemovedEntity` 拼写错误实际不反登记（见陷阱），正常主路径不触发，单独删背层会留悬空表项。宿主 `highlight` 组件遍历该表统一着色。 | [highlightchild](../../../raw/current/scripts/components/highlightchild.lua#L24-L48)、[高亮消费](../../../raw/current/scripts/components/highlight.lua#L81-L93)、[风铃登记](../../../raw/current/scripts/prefabs/hermithouse_ornament.lua#L21-L36)、[背层登记](../../../raw/current/scripts/prefabs/lunarthrall_plant.lua#L55-L67) |
| 附加色 | `king_cane_fx` 用 `colouraddersync`；`lunarthrall_plant_back` 接宿主 `colouradder` | 国王魔杖的 `fx_OnColourChanged` 同时给 `corruptfx` 与 `gemfx` 设附加色；背层由宿主 `colouradder:AttachChild` 跟随宿主染色。 | [颜色回调](../../../raw/current/scripts/prefabs/king_cane.lua#L435-L438)、[背层接线](../../../raw/current/scripts/prefabs/lunarthrall_plant.lua#L121-L122) |

### 清理与资源

| 注册名 | 清理路径 | 未覆盖的边界 | 资源与入口 |
|---|---|---|---|
| `king_cane_fx` | 卸下时 `SetFxOwner(inst, nil)` 把 FX 挂回本体并清零附加色；`persists=false`；无宿主移除时的显式 `Remove`。 | 本体被删除时子 FX 是否由引擎回收未核验。 | bank/build `king_cane`；[资源](../../../raw/current/scripts/prefabs/king_cane.lua#L549-L551) |
| `hermithouse_ornament_fx` | 槽位移除、刷新或替换时显式 `Remove`；FX 移除时手写 `UnlinkHighlight` 反登记；`persists=false`。 | 隐士屋整体移除时没有逐槽清理代码，父挂级联未核验。 | bank/build `hermithouse_ornament_shell`，宿主可覆写为洗衣绳／Wagstaff build；[工厂](../../../raw/current/scripts/prefabs/hermithouse_ornament.lua#L84-L115) |
| `lunar_grazer_core_fx` | `persists=false`；源码无自删与宿主显式移除。 | 牧者 `OnRemoveEntity` 只清 debris；core 的宿主删除回收未核验。 | bank/build `lunar_grazer`；[动画](../../../raw/current/scripts/prefabs/lunar_grazer.lua#L536-L538) |
| `lunarthrall_plant_back` | `persists=false`；源码无自删与宿主显式移除。 | L84 `OnRemovedEntity` 拼写错误，反登记回调永不执行；宿主删除时子实体回收未核验。 | bank/build `lunarthrall_plant_back`；[动画](../../../raw/current/scripts/prefabs/lunarthrall_plant.lua#L78-L84) |
| `moon_altar_link_contained` | `persists=false`；无自删；装置在无效落点、破碎或 Boss 生成流程中 `Remove`。 | 装置删除时连接件的引擎级联未核验；`_set_stage_fn` 仅服务端存在。 | bank `moon_altar_geyser`、build `moon_geyser`；[工厂](../../../raw/current/scripts/prefabs/moon_altar_link.lua#L212-L231) |
| `monkeyisland_portal_lootfollowfx` | 主模拟 `fx_len` 后播 `idle_pst`，`animover` 自删；`AddChild` 使 loot 删除时一并回收。 | 客户端侧无 `fx_len`／自删任务，依赖服务端删除复制；动画包实际长度未核验。 | bank/build `monkey_island_portal_fx`；[自删](../../../raw/current/scripts/prefabs/monkeyisland_portal.lua#L96-L99) |
| `hutch_music_light_fx` | `persists=false`；地面辉光仅在音乐电池失效分支逐个 `Remove`；控制器本地 `_fx` 父挂自身。 | 哈奇死亡／移除时没有显式清理 3 个辉光，其回收未运行核验。 | bank/build `hutch_lightfx_base`；[本地子实体](../../../raw/current/scripts/prefabs/hutch.lua#L533-L552) |

## 典型接入

### 把联网 FX 挂到持有者符号

- 前提与选择：需要“随装备切换持有者、并且参与高亮”的表现时，复制 `king_cane` 的三步：`SetParent`、`Follower:FollowSymbol`、`highlightchild:SetOwner`；父挂只解决变换，高亮与附加色仍需单独接线。
- 生效与撤销：卸下时把 owner 换回本体或 `nil`，并按需 `SyncColour(0,0,0,0)`；具体复位由持有者组件负责，不能只删 FX。
- 原版依据：[`SetFxOwner`](../../../raw/current/scripts/prefabs/king_cane.lua#L42-L65)、[高亮子层](../../../raw/current/scripts/prefabs/king_cane.lua#L509-L530)

```lua
local function attach_fx(owner)
    local fx = SpawnPrefab("king_cane_fx")
    fx.entity:SetParent(owner.entity)
    fx.Follower:FollowSymbol(owner.GUID, "swap_object", nil, nil, nil, true)
    fx.components.highlightchild:SetOwner(owner)
end
```

- 调整与验证：直挂不进 `EntityScript.children`，需要宿主删除时自动带走应改用 `owner:AddChild(fx)`；`swap_object` 是国王魔杖的符号，其他宿主必须换成自己的 `FollowSymbol`。示例只做语法检查，未在游戏内执行。

### 克隆带皮肤的隐士屋装饰

- 前提与选择：装饰物品已有 `CloneAsFx` 时，优先复用其皮肤参数，而不是手写 bank／build；`hermithouse_ornament_fx` 的 `AttachToParent` 只在服务端存在，并且会负责播放落位动画与高亮登记。
- 生效与撤销：槽位刷新／移除时先 `Remove` 旧 FX 再重建；`soundpath` 要在 `AttachToParent` 之前设置，否则即时播放的 `idle_LP` 因 `soundpath` 为 nil 而不响（`place` 落位音才有默认）；落位音走默认 `stocking_place`。
- 原版依据：[`CloneAsFx`](../../../raw/current/scripts/prefabs/hermithouse_ornament.lua#L117-L124)、[槽位接线](../../../raw/current/scripts/prefabs/hermithouse.lua#L58-L62)

```lua
local function clone_decor(item, host, slot)
    local skin_build = item:GetSkinBuild()
    local fx = SpawnPrefab("hermithouse_ornament_fx", skin_build, item.skin_id)
    fx.soundpath = "hookline_2/characters/hermit/house/decor/"
        .. (skin_build and string.sub(skin_build, 22) or "shell") .. "_"
    fx:AttachToParent(host)
    fx.Follower:FollowSymbol(host.GUID, "follow_ornament_" .. slot, nil, nil, nil, true)
end
```

- 调整与验证：`skin_nosound` 的宿主物品不会设置 `soundpath`；洗衣绳与 Wagstaff 不走本函数，而是在自己 `CloneAsFx` 里改 bank／build。示例只做语法检查，未在游戏内执行。

### 给可移除的 loot 挂短命跟随光

- 前提与选择：目标会被 `Remove` 且希望光效随之消失时用 `AddChild`，不要只 `SetParent`；本注册的 `fx_len` 只在主模拟赋值，客户端侧没有自删任务。
- 生效与撤销：`AddChild` 登记到 loot 的 `children`，loot 删除时由 `EntityScript:Remove` 遍历移除；光效自身播完 `idle_pst` 也会自删。
- 原版依据：[`attach_light_fx`](../../../raw/current/scripts/prefabs/monkeyisland_portal.lua#L173-L182)、[自删](../../../raw/current/scripts/prefabs/monkeyisland_portal.lua#L65-L68)

```lua
local function attach_loot_glow(loot, yoffset)
    local fx = SpawnPrefab("monkeyisland_portal_lootfollowfx")
    fx.Transform:SetPosition(0, yoffset or 0, 0)
    loot:AddChild(fx)
end
```

- 调整与验证：局部偏移要在 `AddChild` 前写入；原版偏移表只覆盖真实 loot，假掉落与未列名 prefab 取 0。示例只做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **`OnRemovedEntity` 拼写陷阱**：`lunarthrall_plant_back` 写的是 `inst.OnRemovedEntity = back_onremoveentity`，而引擎在 `EntityScript:Remove` 里调用的是 `self.OnRemoveEntity`；在 `raw/current/scripts` 全库检索该拼写仅此一处。结果：背层被移除时不会从宿主 `highlightchildren` 反登记；当前源码没有单独移除背层的路径，所以主路径不炸，但任何“单独删背层”的扩展都会留下悬空表项。正确写法是 `OnRemoveEntity`。[拼写错误](../../../raw/current/scripts/prefabs/lunarthrall_plant.lua#L84)、[引擎回调](../../../raw/current/scripts/entityscript.lua#L1792-L1793)
- **`SetParent` 不等于 `AddChild`**：本族多数注册用裸 `entity:SetParent`，不会进入 `EntityScript.children`；宿主 `Remove` 的 `children` 循环只覆盖 `AddChild` 的子实体。需要“宿主删了 FX 一定走”时改成 `AddChild`，否则宿主删除后的引擎回收要另行核验。[children 循环](../../../raw/current/scripts/entityscript.lua#L1779-L1783)、[AddChild](../../../raw/current/scripts/entityscript.lua#L921-L934)
- **`king_cane_fx` 的 3 个显式 netvar 不是全部网络变量**：工厂自建 `gemshine`／`level`／`gem` 三个；`highlightchild` 还会加 `highlightchild.syncowner`，`colouraddersync` 还会加 `colouraddersync.colour`。客户端只在非专服分支直接建 `corruptfx`／`gemfx` 本地子层，再由这些变量驱动；专用服务器完全不建这两个子层。[网络变量](../../../raw/current/scripts/prefabs/king_cane.lua#L557-L571)、[组件网络变量](../../../raw/current/scripts/components/highlightchild.lua#L10-L15)、[颜色网络变量](../../../raw/current/scripts/components/colouraddersync.lua#L17)
- **`hermithouse_ornament_fx` 自己不带资源**：注册只传工厂，没有 assets／prefabs 表；bank/build 由宿主物品或 `AddFake*` 决定。单独 `SpawnPrefab` 前必须确保对应 anim 包已由宿主加载，否则可能缺资源；`soundpath` 只有 `CloneAsFx` 路径会写，直接生成时保持 `nil` 并走默认落位音。[注册](../../../raw/current/scripts/prefabs/hermithouse_ornament.lua#L175-L176)、[资源依赖](../../../raw/current/scripts/prefabs/hermithouse_ornament.lua#L1-L9)
- **`moon_altar_link_contained._set_stage_fn` 只在主模拟有**：客户端没有这个字段，调用会报 nil；阶段名由 `stage<N>_idle_pre`＋`stage<N>_idle` 拼出，`stage` 取装置 `level`，不要传 0 或非整数字符串。[阶段设置](../../../raw/current/scripts/prefabs/moon_altar_link.lua#L197-L202)、[端分界](../../../raw/current/scripts/prefabs/moon_altar_link.lua#L224-L231)
- **`monkeyisland_portal_lootfollowfx` 的 `fx_len` 只在主模拟赋值**：`attach_light_fx` 里“加 `outofreach`＋提高 LightOverride”的分支因此只在主模拟成立；客户端只负责复制表现，复制体没有 `fx_len` 与自删任务。[主模拟赋值](../../../raw/current/scripts/prefabs/monkeyisland_portal.lua#L96-L99)、[分支](../../../raw/current/scripts/prefabs/monkeyisland_portal.lua#L173-L182)
- **`hutch_music_light_fx` 是 5＋1 个网络变量，不是 5 个**：五个 `fxdirty` 参数之外还有 `_anim`（`animdirty`）；控制器本身不设 Light 组件，只是贴地光斑，与 [[reference/prefabs/cavelight_small|光源实体页]] 的 Light 实体不重叠。地面辉光只在音乐电池失效时清理，宿主死亡路径没有显式 `Remove`。[网络变量](../../../raw/current/scripts/prefabs/hutch.lua#L642-L648)、[清理](../../../raw/current/scripts/prefabs/hutch.lua#L250-L256)

## 验证记录

- 源码审阅：2026-11-04，steam-buildid:26335394（实读 [raw/current/BUILD.txt](../../../raw/current/BUILD.txt#L1)）。逐文件核对 `king_cane.lua`、`hermithouse_ornament.lua`、`hermithouse.lua`、`hermithouse_laundry.lua`、`wagstaff_items.lua`、`lunar_grazer.lua`、`lunarthrall_plant.lua`、`moon_altar_link.lua`、`moon_device.lua`、`monkeyisland_portal.lua`、`hutch.lua` 的注册名、工厂、端分界、生成点、父挂／Follower、净变量与清理；并核验 `highlightchild.lua`、`colouraddersync.lua`、`highlight.lua`、`entityscript.lua` 的登记与删除语义。对七个完整注册名在 `raw/current/scripts` 全域反查，命中仅为上述工厂、宿主与依赖声明，未发现额外生成点。
- 名称核验：`languages/chinese_s.po` 定向检索七个 FX 注册名均无 `STRINGS.NAMES` 键；宿主键命中：`KING_CANE`（国王魔杖）[L344209-L344212](../../../raw/current/scripts/languages/chinese_s.po#L344209-L344212)、`HERMITHOUSE_ORNAMENT`（风铃）[L343894-L343897](../../../raw/current/scripts/languages/chinese_s.po#L343894-L343897)、`LUNAR_GRAZER`（牧者）[L344924-L344927](../../../raw/current/scripts/languages/chinese_s.po#L344924-L344927)、`LUNARTHRALL_PLANT`（致命亮茄）[L344899-L344902](../../../raw/current/scripts/languages/chinese_s.po#L344899-L344902)、`MOON_ALTAR_LINK`（神秘能量）[L345784-L345787](../../../raw/current/scripts/languages/chinese_s.po#L345784-L345787)、`MONKEYISLAND_PORTAL`（非自然传送门）[L345549-L345552](../../../raw/current/scripts/languages/chinese_s.po#L345549-L345552)、`HUTCH`（哈奇）[L344044-L344047](../../../raw/current/scripts/languages/chinese_s.po#L344044-L344047)。标题未使用这些宿主名作为 FX 官方名。
- 分档说明：覆盖 7 个注册、7 个工厂、7 类宿主与三种生命周期（显式移除／animover 自删／无显式清理），父挂方式与端分界均不同，需要整体对照，故选 `complex`。
- 游戏运行：未运行；未验证动画包实际帧数、循环标记、贴地层效果与多人端时序。
- 结构检查：`validate_page.py` 通过（0 issue；160 行、31,244 字节，complex 档上限 400 行／32 KiB）；`validate_source_links.py --strict` 通过（errors 0、warnings 0；104 处源码引用、80 个唯一目标、17 个源文件）。三段 Lua 示例均用 `parser_probe.exe --syntax-only` 逐段检查通过（只查语法，不执行）。
- 未覆盖：宿主本体的玩法入口、各动画资源是否存在、宿主持有者变更存档恢复、直挂子实体在宿主删除后的引擎级回收、`hutch_music_light_fx` 三个地面辉光在宿主死亡时的去向。

## 相关链接

- [[reference/prefabs/fence_gate_anim|宿主挂接的附属表现实体]]（同为宿主附属层，静态子实体对照）
- [[reference/prefabs/moon_altar_link_fx|月坛连接光效]]（与 `moon_altar_link_contained` 不是同一对象）
- [[reference/prefabs/monkeyisland_portal_fxloot|猴岛传送门假掉落]]（本页跟随光也会挂到该假掉落）
- [[reference/prefabs/moon_device_pillar|月装置支柱、顶盖与破碎特效]]（月装置生成点与父挂对照）
- [[reference/prefabs/lunar_goop_cloud_fx|牧者黏液雾云]]、[[reference/prefabs/lunar_goop_trail_fx|牧者黏液拖尾]]（同宿主其他 FX）
- [[reference/components/highlightchild|highlightchild]]、[[reference/components/colouraddersync|colouraddersync]]、[[reference/components/highlight|highlight]]
- [[reference/components/corruption|腐蚀组件]]（国王魔杖 level 分档的数值来源）
- [[reference/prefabs/cavelight_small|光源实体]]（`hutch_music_light_fx` 不含 Light 组件，不归该页）

返回 [[reference/prefabs/fx-skin-index|皮肤与装备挂件]]。
