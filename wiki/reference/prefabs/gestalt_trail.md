---
title: "宿主附着与跟随的外围 FX 族（gestalt_trail 等）"
created: "2026-11-04"
updated: "2026-11-04"
type: prefab
size_class: complex
summary: "六个宿主 prefab／状态图侧生成的外围表现注册：虚影走路拖尾、拾荒疯猪埋地垃圾层、幻灵捕获机换装跟随层、恶液背层、暗影切斯特开箱旋涡与雪球发射器光晕；差异集中在端、父挂与 Follower、朝向和清理，均不挂玩法组件。"
aliases: ["gestalt_trail", "daywalker2_buried_fx", "gestalt_cage_swap_fx", "gelblob_back_fx", "shadow_chester_swirl_fx", "firesuppressor_glow", "外围 FX 族", "附着与跟随 FX", "虚影拖尾", "埋地垃圾层", "换装跟随特效", "恶液背层", "暗影切斯特旋涡", "雪球发射器光晕"]
tags: [prefab, fx, server-client]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/gestalt.lua", "prefabs/gestalt_cage.lua", "prefabs/daywalker2.lua", "prefabs/gelblob.lua", "prefabs/chester.lua", "prefabs/firesuppressor.lua", "prefabs/gestalt_guard.lua", "prefabs/corpse_gestalt.lua", "prefabs/lunarthrall_plant_gestalt.lua", "prefabs/junk_pile_big.lua", "stategraphs/SGbrightmare_gestalt.lua", "stategraphs/SGbrightmare_invader.lua", "stategraphs/SGlunarthrall_plant_gestalt.lua", "stategraphs/SGchester.lua", "stategraphs/SGgelblob.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 宿主附着与跟随的外围 FX 族（gestalt_trail 等）

## 快速判断

- 职责与适用：六个宿主侧生成的外围表现注册，全都不挂生命、工作、库存等玩法组件：`gestalt_trail` 是虚影走路时落在世界坐标的地面拖尾，`daywalker2_buried_fx` 是拾荒疯猪埋进垃圾堆时挂在垃圾上的三层遮盖，`gestalt_cage_swap_fx` 是幻灵捕获机装备后跟随持有者 `swap_object` 的换装层，`gelblob_back_fx` 是恶液本体的背层，`shadow_chester_swirl_fx` 是暗影切斯特开箱时的旋涡，`firesuppressor_glow` 是雪球发射器的状态光晕。与 [[reference/prefabs/king_cane_fx|宿主持有与跟随的附属 FX 族]] 相比，本族多了“不父挂、只落世界坐标”的拖尾，以及“父挂一个实体、Follower 却跟另一个实体”的分裂挂接。
- 优先使用：按需求直达——世界坐标残影选 `gestalt_trail`；埋地／被垃圾覆盖的局部替换层选 `daywalker2_buried_fx`；装备后随持有者符号的换装层选 `gestalt_cage_swap_fx`；宿主前后分层选 `gelblob_back_fx`；开箱／关箱的旋涡收尾选 `shadow_chester_swirl_fx`；结构状态光晕选 `firesuppressor_glow`。各自的生成点与端限制见“关联机制导航”。
- 重要副作用：本身不改变玩法状态；会动到其它表现系统的是 `gestalt_cage_swap_fx`（客户端登记 `owner.highlightchildren`，并把自身挂进 `colouradder`／`bloomer` 子层）、`daywalker2_buried_fx`（`junk_pile_big` 的高亮子层）和 `firesuppressor_glow`（`net_bool` 驱动的透明度淡入淡出）。宿主伤害、拾取、灭火与开箱流程均不在本页。[高亮登记](../../../raw/current/scripts/prefabs/gestalt_cage.lua#L124-L134)、[垃圾堆高亮](../../../raw/current/scripts/prefabs/daywalker2.lua#L800-L813)、[淡入淡出](../../../raw/current/scripts/prefabs/firesuppressor.lua#L389-L422)
- 修改边界：只覆盖六个工厂、端分界、生成点、父挂／Follower 与清理；各宿主 prefab 的玩法、动画包实际帧数与循环标记、存档恢复和多人时序未展开。六者全部在主模拟分支设 `persists=false`，客户端分支不负责删除。

## 注册与构造

六个注册分散在六个 prefab 文件，均为 `Prefab(注册名, 工厂, assets[, prefabs])` 的直接返回，没有包装工厂或多形态切换；共同点是没有玩法组件、没有独立的自删职责（`gestalt_trail` 例外）。逐条核验如下。

| 注册名／宿主文件 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `gestalt_trail`（`prefabs/gestalt.lua`） | `gestalt_trail_fn`：CreateEntity → Transform／AnimState／Network → `FX` 标签 → bank／build `brightmare_gestalt_trail`、`LAYER_BACKGROUND`、`OnGround`、排序 2、bloom、先播 `trail1` → 缩放 1.2 → 非专用端装 `transparentonsanity` → Pristine → 主模拟随机 `trail1`–`trail8`、`persists=false`、40×`FRAMES` 后 `Remove` | 无 Follower、无父挂；位置与旋转由生成者写入。随机动画只取大于 1 的编号重播，`trail1` 即默认值。[工厂](../../../raw/current/scripts/prefabs/gestalt.lua#L217-L260)、[自删与随机](../../../raw/current/scripts/prefabs/gestalt.lua#L252-L258)、[注册](../../../raw/current/scripts/prefabs/gestalt.lua#L263-L264) |
| `daywalker2_buried_fx`（`prefabs/daywalker2.lua`） | `buriedfx_fn`：第一标签 `can_offset_sort_pos` → Transform／AnimState／Follower／Network → `SetEightFaced` → bank `daywalker`、build `daywalker_buried`、`buried_hold_full` 循环、隐藏 `junk_top`／`junk_mid`／`junk_back` → `FX` 标签 → Pristine → 双侧设 `OnRemoveEntity`，客户端再设 `OnEntityReplicated` → 主模拟 `persists=false` | 工厂一次造一层，生成点生成三份并各 `Show` 一个符号；父挂垃圾实体，Follower 却跟拾荒疯猪的 `follow_junk_*` 符号。头层额外 `SetSortWorldOffset(0, 0.1, 0)` 提高鼠标悬停优先级。[工厂](../../../raw/current/scripts/prefabs/daywalker2.lua#L1302-L1336)、[生成](../../../raw/current/scripts/prefabs/daywalker2.lua#L800-L813)、[注册](../../../raw/current/scripts/prefabs/daywalker2.lua#L1341-L1342) |
| `gestalt_cage_swap_fx`（`prefabs/gestalt_cage.lua`） | `fxfn`：Transform／AnimState／Follower／Network → `FX` 标签 → `SetEightFaced`（注释说明覆盖 4 面与骑乘 6 面玩家）→ bank／build `gestalt_cage`、`swap1` → Pristine → 客户端 `OnEntityReplicated` → 主模拟导出 `AttachToOwner`、`persists=false` | `AttachToOwner` 依次 SetParent、`FollowSymbol(owner.GUID, "swap_object", nil, nil, nil, true, nil, 0, 2)`、有 `colouradder`／`bloomer` 时 `AttachChild`，非专用端再登记高亮；客户端复制回调按当前父实体补登记。[工厂](../../../raw/current/scripts/prefabs/gestalt_cage.lua#L138-L167)、[AttachToOwner](../../../raw/current/scripts/prefabs/gestalt_cage.lua#L124-L134)、[注册](../../../raw/current/scripts/prefabs/gestalt_cage.lua#L788-L789) |
| `gelblob_back_fx`（`prefabs/gelblob.lua`） | `backfn`：Transform／AnimState／Network → `FX` 标签 → bank／build `gelblob`、`idle_big` 循环 → `SetFinalOffset(-7)`、隐藏 `FRONT` → Pristine → 主模拟 `persists=false` | 无 Follower；宿主 `fn` 生成后 `SetParent` 到本体。本体自己 `SetFinalOffset(7)` 并隐藏 `BACK`，前后层靠 ±7 错开。[工厂](../../../raw/current/scripts/prefabs/gelblob.lua#L616-L639)、[宿主挂接与本体层](../../../raw/current/scripts/prefabs/gelblob.lua#L531-L561)、[注册](../../../raw/current/scripts/prefabs/gelblob.lua#L966-L967) |
| `shadow_chester_swirl_fx`（`prefabs/chester.lua`） | `swirl_fn`：Transform／AnimState／Network → `NOCLICK`＋`CLASSIFIED`（注释说明故意不加 `FX`，否则父挂后仍会被鼠标悬停）→ `SetFourFaced` → bank `chester`、build `tophat_fx`、`swirl_pre` → `SetFinalOffset(1)` → Pristine → 客户端早退 → 主模拟 Push `swirl_loop` 循环、`persists=false`、导出 `ReleaseSwirl` | 生成后只 `SetParent`，不设 Follower；`ReleaseSwirl` 解父、写回世界位置与旋转、播 `swirl_pst`，再靠 `animover` 自删。[工厂](../../../raw/current/scripts/prefabs/chester.lua#L725-L755)、[收尾](../../../raw/current/scripts/prefabs/chester.lua#L715-L722)、[注册](../../../raw/current/scripts/prefabs/chester.lua#L760-L761) |
| `firesuppressor_glow`（`prefabs/firesuppressor.lua`） | `glow_fn`：Transform／AnimState／Follower／Network → `FX`＋`NOCLICK` → bank／build `firefighter_glow`、`green` 循环 → `SetLightOverride(1)`、`SetFinalOffset(3)`、alpha 0 → `net_bool("firesuppressor_glow._ison", "onisondirty")` → `DoTaskInTime(0, oninitglow)` → Pristine → 主模拟 `persists=false` | 无父挂；宿主 `oninit` 调 `FollowSymbol(inst.GUID, "swap_glow", 0, 0, 0)`。`_ison` 由宿主按预警等级设置，`onfade` 周期任务按它淡入／淡出。[工厂](../../../raw/current/scripts/prefabs/firesuppressor.lua#L425-L456)、[宿主接线](../../../raw/current/scripts/prefabs/firesuppressor.lua#L230-L231)、[注册](../../../raw/current/scripts/prefabs/firesuppressor.lua#L488-L489) |

名称：六个注册名在简中 `.po` 均无 `STRINGS.NAMES.*` 键，标题中的中文是已核验的效果描述，不是官方译名。定位宿主时命中官方名：虚影（`GESTALT`，[L343169-L343172](../../../raw/current/scripts/languages/chinese_s.po#L343169-L343172)）、大虚影（`GESTALT_GUARD`，[L343184-L343187](../../../raw/current/scripts/languages/chinese_s.po#L343184-L343187)）、拾荒疯猪（`DAYWALKER2`，[L341984-L341987](../../../raw/current/scripts/languages/chinese_s.po#L341984-L341987)）、幻灵捕获机（`GESTALT_CAGE`，[L343179-L343182](../../../raw/current/scripts/languages/chinese_s.po#L343179-L343182)）、恶液（`GELBLOB`，[L343144-L343147](../../../raw/current/scripts/languages/chinese_s.po#L343144-L343147)）、切斯特（`CHESTER`，[L341539-L341542](../../../raw/current/scripts/languages/chinese_s.po#L341539-L341542)）、雪球发射器（`FIRESUPPRESSOR`，[L342819-L342822](../../../raw/current/scripts/languages/chinese_s.po#L342819-L342822)）。这些名称只用于定位宿主，不把宿主名当作 FX 的官方名。

## 关联机制导航

### 生成点与宿主

| 生成点 | 涉及注册 | 端、阶段与条件 | 证据 |
|---|---|---|---|
| `gestalt`（虚影）走路状态 | `gestalt_trail` | 主模拟；`SGbrightmare_gestalt` 的 `walktimeline` 在 `FrameEvent(0)` 调 `SpawnTrail`，每次进入 walk 状态生成一枚；宿主 `_notrail` 为真时跳过 | [设 SG](../../../raw/current/scripts/prefabs/gestalt.lua#L209)、[生成函数](../../../raw/current/scripts/stategraphs/SGbrightmare_gestalt.lua#L374-L377)、[walktimeline](../../../raw/current/scripts/stategraphs/SGbrightmare_gestalt.lua#L382-L390) |
| `corpse_gestalt`／`lunarthrall_plant_gestalt` 走路状态 | `gestalt_trail` | 两个 prefab 共用 `SGlunarthrall_plant_gestalt`；该图 `walktimeline` 同样在 `FrameEvent(0)` 调 `SpawnTrail`，条件与上一条相同 | [corpse_gestalt 设 SG](../../../raw/current/scripts/prefabs/corpse_gestalt.lua#L127)、[lunarthrall_plant_gestalt 设 SG](../../../raw/current/scripts/prefabs/lunarthrall_plant_gestalt.lua#L89)、[生成与 walktimeline](../../../raw/current/scripts/stategraphs/SGlunarthrall_plant_gestalt.lua#L205-L221) |
| `gestalt_guard`（大虚影） | `gestalt_trail`（被抑制） | 同样 `SetStateGraph("SGbrightmare_gestalt")`，但主模拟在 `_notrail = true` 赋值后永不生成；不能因为共用 SG 就推断它会拖尾 | [设 SG](../../../raw/current/scripts/prefabs/gestalt_guard.lua#L278)、[抑制标记](../../../raw/current/scripts/prefabs/gestalt_guard.lua#L242) |
| `SGbrightmare_invader`（无宿主） | `gestalt_trail`（死代码候选） | 该文件有一模一样的 `SpawnTrail`／`walktimeline`，但对 `raw/current/scripts` 全域反查没有任何 prefab 设置 `SGbrightmare_invader`，当前快照没有实际生成者 | [生成函数](../../../raw/current/scripts/stategraphs/SGbrightmare_invader.lua#L161-L165) |
| `daywalker2` 的 `MakeBuried` | `daywalker2_buried_fx` | 主模拟；`junk` 由 `junk_pile_big` 作为参数传入 `MakeBuried`，生成后才 `TrackEntity("junk", junk)`；未生成过时循环三种符号各造一枚；`junk_pile_big` 才把三层登记进 `highlightchildren` | [生成](../../../raw/current/scripts/prefabs/daywalker2.lua#L800-L813)、[大垃圾堆高亮表](../../../raw/current/scripts/prefabs/junk_pile_big.lua#L559) |
| `gestalt_cage` 的 `OnEquip` | `gestalt_cage_swap_fx` | 主模拟；先 `Remove` 旧 `inst.fx`，再 Spawn 新层并 `AttachToOwner(owner)`；`OnUnequip` 反方向清理 | [OnEquip](../../../raw/current/scripts/prefabs/gestalt_cage.lua#L30-L39)、[OnUnequip](../../../raw/current/scripts/prefabs/gestalt_cage.lua#L42-L48) |
| `gelblob` 本体 `fn` | `gelblob_back_fx` | 主模拟；`inst.back = SpawnPrefab(...)` 后立即 `SetParent(inst.entity)`，只生成一份 | [生成](../../../raw/current/scripts/prefabs/gelblob.lua#L560-L561) |
| `SGchester` 的 `open` 状态 | `shadow_chester_swirl_fx` | 仅暗影形态：`MorphShadowChester` 先置 `inst.sg.mem.isshadow = true`，`open` 状态才 Spawn 并 `SetParent`；切回普通／雪形态会清掉该标记 | [形态标记](../../../raw/current/scripts/prefabs/chester.lua#L452)、[生成](../../../raw/current/scripts/stategraphs/SGchester.lua#L170-L173)、[设 SG](../../../raw/current/scripts/prefabs/chester.lua#L696) |
| `firesuppressor` 本体 `fn` | `firesuppressor_glow` | 主模拟生成；`DoTaskInTime(0, oninit)` 再调 `FollowSymbol`；`SetWarningLevelLight` 设置 `_ison` 与动画；宿主 `OnRemoveEntity` 显式移除 | [生成](../../../raw/current/scripts/prefabs/firesuppressor.lua#L328)、[预警灯号](../../../raw/current/scripts/prefabs/firesuppressor.lua#L69-L77)、[移除](../../../raw/current/scripts/prefabs/firesuppressor.lua#L205-L206) |

### 挂接、跟随与清理

| 注册名 | 父挂／跟随 | 清理路径 | 证据 |
|---|---|---|---|
| `gestalt_trail` | 无父挂、无 Follower；生成者写世界位置与旋转，停留在落点 | 主模拟 `DoTaskInTime(40 * FRAMES, inst.Remove)`；`persists=false`；客户端分支没有删除任务 | [自删](../../../raw/current/scripts/prefabs/gestalt.lua#L252-L258) |
| `daywalker2_buried_fx` | `SetParent(junk.entity)`；`Follower:FollowSymbol(inst.GUID, "follow_"..v, 0, 0, 0, true)`，leader 是拾荒疯猪而不是父实体垃圾堆 | `MakeFreed` 遍历 `inst.junkfx` 逐个 `Remove` 并清空；`OnRemoveEntity` 从父实体 `highlightchildren` 反登记；`persists=false`。拾荒疯猪本体没有 `OnRemoveEntity` 清理 `junkfx`，只有 `MakeFreed` 显式移除；埋地状态直接删除宿主的路径未核验 | [生成](../../../raw/current/scripts/prefabs/daywalker2.lua#L800-L813)、[释放](../../../raw/current/scripts/prefabs/daywalker2.lua#L856-L860)、[反登记](../../../raw/current/scripts/prefabs/daywalker2.lua#L1288-L1291)、[复制登记](../../../raw/current/scripts/prefabs/daywalker2.lua#L1295-L1298) |
| `gestalt_cage_swap_fx` | `SetParent(owner.entity)`＋`FollowSymbol(owner.GUID, "swap_object", ...)`；有 `colouradder`／`bloomer` 时挂为子层；非专用端登记 `owner.highlightchildren` | `OnUnequip` 显式 `Remove` 并置 `nil`；`OnEquip` 重挂前也先删旧层；`OnRemoveEntity` 从 `inst.owner.highlightchildren` 反登记；`persists=false` | [AttachToOwner](../../../raw/current/scripts/prefabs/gestalt_cage.lua#L124-L134)、[反登记](../../../raw/current/scripts/prefabs/gestalt_cage.lua#L102-L114)、[复制回调](../../../raw/current/scripts/prefabs/gestalt_cage.lua#L117-L120)、[卸下](../../../raw/current/scripts/prefabs/gestalt_cage.lua#L42-L48) |
| `gelblob_back_fx` | 宿主 `SetParent(inst.entity)`；无 Follower；背层动画由宿主 SG 显式驱动 | 源码无自删与显式 `Remove`；`persists=false`；宿主 `OnRemoveEntity` 只清理接触目标，不处理背层，删除依赖父实体回收（未运行核验） | [宿主挂接](../../../raw/current/scripts/prefabs/gelblob.lua#L560-L561)、[宿主清理](../../../raw/current/scripts/prefabs/gelblob.lua#L355-L358)、[SG 挂载](../../../raw/current/scripts/prefabs/gelblob.lua#L592)、[背层动画](../../../raw/current/scripts/stategraphs/SGgelblob.lua#L17-L26)、[电击分支](../../../raw/current/scripts/stategraphs/SGgelblob.lua#L359-L361) |
| `shadow_chester_swirl_fx` | `SetParent(inst.entity)` 到切斯特；无 Follower | `ReleaseSwirl` 解父、写回世界位置／旋转、播 `swirl_pst`、`animover` 自删；未调用时不自删；`persists=false` | [生成](../../../raw/current/scripts/stategraphs/SGchester.lua#L170-L173)、[收尾](../../../raw/current/scripts/prefabs/chester.lua#L715-L722)、[状态退场](../../../raw/current/scripts/stategraphs/SGchester.lua#L238-L241) |
| `firesuppressor_glow` | 无父挂；`Follower:FollowSymbol(host.GUID, "swap_glow", 0, 0, 0)` | 宿主 `OnRemoveEntity` 显式 `inst._glow:Remove()`；`persists=false`；自身无自删任务 | [跟随](../../../raw/current/scripts/prefabs/firesuppressor.lua#L230-L231)、[宿主移除](../../../raw/current/scripts/prefabs/firesuppressor.lua#L205-L206) |

### 组件、资源与互链

| 类型／对象 | 本注册如何使用 | 条件与边界 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`transparentonsanity` | `gestalt_trail` 在非专用端安装，按观察者理智渐隐；与主体虚影共用曲线函数 | 客户端视图组件，无服务端逻辑 | [[reference/components/transparentonsanity|transparentonsanity]]、[安装](../../../raw/current/scripts/prefabs/gestalt.lua#L236-L243) |
| 引擎 Follower | `daywalker2_buried_fx`、`gestalt_cage_swap_fx`、`firesuppressor_glow` 用 `FollowSymbol` 定位 | 只负责引擎符号跟随，不创建父子登记；它不是 Lua 的 follower 组件（主人、忠诚与随从关系） | 见各注册行的 `FollowSymbol` 调用 |
| 同步：`net_bool` | `firesuppressor_glow` 的 `_ison` 是六个注册里唯一的显式网络变量，dirty 事件 `onisondirty` 驱动透明度；`gestalt_cage_swap_fx`、`daywalker2_buried_fx` 用复制回调补高亮，其余注册无自定义 netvar | 客户端只读；`_ison` 属内部字段，不作为公开接口推荐 | [net_bool](../../../raw/current/scripts/prefabs/firesuppressor.lua#L439-L446)、[复制回调](../../../raw/current/scripts/prefabs/gestalt_cage.lua#L117-L120) |
| 组件：`highlight`／`highlightchildren` | `gestalt_cage_swap_fx` 与 `daywalker2_buried_fx` 把自身加入宿主表；宿主高亮时统一着色 | 各宿主按自身端分支建立／清理该表；不是所有宿主都有 | [[reference/components/highlight|highlight]]、[幻灵捕获机登记](../../../raw/current/scripts/prefabs/gestalt_cage.lua#L102-L114) |
| 组件：`colouradder`／`bloomer` | `gestalt_cage_swap_fx` 的 `AttachToOwner` 把层挂到持有者对应组件，跟随附加色与泛光 | 持有者没有组件时跳过 | [[reference/components/colouradder|colouradder]]、[[reference/components/bloomer|bloomer]] |
| 宿主组件：`gestaltcage`／`equippable` | `gestalt_cage_swap_fx` 的唯一生成者是幻灵捕获机的装备回调；捕获玩法在 `gestaltcage` | 本页不展开捕获与填充牢笼生成 | [[reference/components/gestaltcage|gestaltcage]]、[[reference/components/equippable|equippable]] |
| 资源 | 六组动画资源（7 个 zip）：`brightmare_gestalt_trail.zip`、`daywalker_buried.zip`、`gestalt_cage.zip`、`gelblob.zip`、`chester.zip`＋`tophat_fx.zip`、`firefighter_glow.zip` | 动画帧数、循环标记与视觉未核验；不分发资源 | [拖尾资源](../../../raw/current/scripts/prefabs/gestalt.lua#L16)、[埋地资源](../../../raw/current/scripts/prefabs/daywalker2.lua#L13)、[换装资源](../../../raw/current/scripts/prefabs/gestalt_cage.lua#L3)、[背层资源](../../../raw/current/scripts/prefabs/gelblob.lua#L3)、[旋涡资源](../../../raw/current/scripts/prefabs/chester.lua#L23-L24)、[光晕资源](../../../raw/current/scripts/prefabs/firesuppressor.lua#L14) |

## 典型接入

### 在世界点生成一枚自删的虚影拖尾

- 前提与选择：需要“按步或按事件在地面留一枚短命拖尾”时用 `gestalt_trail`；原版在虚影走路状态进入时生成，生成者只写位置与朝向。
- 生效与撤销：工厂在主模拟 40×`FRAMES` 后 `Remove`；`persists=false`。不要把它父挂到会移动的宿主，否则地面残影会跟着走，和原版“落在世界坐标”的用法相反。
- 原版依据：[生成函数](../../../raw/current/scripts/stategraphs/SGbrightmare_gestalt.lua#L374-L377)、[自删与随机](../../../raw/current/scripts/prefabs/gestalt.lua#L252-L258)

```lua
local function spawn_gestalt_trail(inst)
    if not TheWorld.ismastersim then
        return
    end
    local fx = SpawnPrefab("gestalt_trail")
    fx.Transform:SetPosition(inst.Transform:GetWorldPosition())
    fx.Transform:SetRotation(inst.Transform:GetRotation())
end
```

- 调整与验证：只在主模拟生成；客户端分支没有删除任务。`trail1`–`trail8` 由工厂随机，外层不要覆写动画。示例只做语法检查，未在游戏内执行。

### 给可装备物品挂随持有者符号的换装层

- 前提与选择：需要“装备后跟随持有者 `swap_object`、卸下即消失”的层时复用 `gestalt_cage_swap_fx` 的 `AttachToOwner`；它一次处理父挂、符号跟随、附加色／泛光子层和客户端高亮。
- 生效与撤销：`OnEquip` 先删旧层再生成；`OnUnequip` 删除并把 `item.fx` 置 `nil`；`persists=false`，不跨存档。`AttachToOwner` 只在主模拟存在。
- 原版依据：[OnEquip](../../../raw/current/scripts/prefabs/gestalt_cage.lua#L30-L39)、[AttachToOwner](../../../raw/current/scripts/prefabs/gestalt_cage.lua#L124-L134)

```lua
local function attach_swap_fx(item, owner)
    if not TheWorld.ismastersim then
        return
    end
    if item.fx ~= nil then
        item.fx:Remove()
    end
    local fx = SpawnPrefab("gestalt_cage_swap_fx")
    fx:AttachToOwner(owner)
    item.fx = fx
end

local function detach_swap_fx(item)
    if item.fx ~= nil then
        item.fx:Remove()
        item.fx = nil
    end
end
```

- 调整与验证：`swap_object` 是幻灵捕获机的持有符号，换自定义物品时要同步改持有者符号覆盖；客户端靠 `OnEntityReplicated` 补高亮。示例只做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **边界：`gestalt_trail` 的自删只在主模拟。** `transparentonsanity` 与 40 帧自删分属两个分支：前者只在非专用端安装，后者只在主模拟执行；客户端在 Pristine 后早退，不安排 `Remove`。`SetStateGraph` 也在主模拟分支，客户端没有该 SG，生成只发生在服务端，删除由服务端复制。[端分界](../../../raw/current/scripts/prefabs/gestalt.lua#L236-L258)、[主模拟分界](../../../raw/current/scripts/prefabs/gestalt.lua#L171-L172)、[设 SG](../../../raw/current/scripts/prefabs/gestalt.lua#L209)
- **边界：共用 `SGbrightmare_gestalt` 不等于会拖尾。** `gestalt_guard` 在主模拟设 `_notrail = true`，`SpawnTrail` 直接跳过；把自定义宿主也设这个标记即可静音。另有一个 `SGbrightmare_invader` 文件带同样生成函数，但当前快照没有 prefab 设置该 SG。[抑制](../../../raw/current/scripts/prefabs/gestalt_guard.lua#L242)、[无宿主生成函数](../../../raw/current/scripts/stategraphs/SGbrightmare_invader.lua#L161-L165)
- **边界：`daywalker2_buried_fx` 的父挂与跟随是两套关系。** 父实体是 `junk`，`Follower` 的 leader 却是拾荒疯猪；高亮只在 `junk_pile_big` 上登记，普通垃圾堆不登记。`OnRemoveEntity` 会从父实体 `highlightchildren` 反登记，但依赖父实体存在该表。[生成](../../../raw/current/scripts/prefabs/daywalker2.lua#L800-L813)、[反登记](../../../raw/current/scripts/prefabs/daywalker2.lua#L1288-L1291)
- **边界：`gestalt_cage_swap_fx` 的 `AttachToOwner` 只在主模拟。** 直接生成后客户端没有高亮与父挂；必须由服务端装备回调调用。`fx_OnRemoveEntity` 直接读 `inst.owner`，只适用于已挂接的实例；专用服务器不登记高亮。[AttachToOwner](../../../raw/current/scripts/prefabs/gestalt_cage.lua#L124-L134)、[反登记](../../../raw/current/scripts/prefabs/gestalt_cage.lua#L102-L114)
- **边界：`gelblob_back_fx` 不自删、也不反登记。** 它没有 `OnRemoveEntity`，宿主 `OnRemoveEntity` 只清理接触目标；长期存活依赖 `SetParent` 的引擎级回收（未运行核验）。另需同时配好本体的 `SetFinalOffset(7)`＋Hide `BACK` 与背层的 `SetFinalOffset(-7)`＋Hide `FRONT`，否则前后层会重叠。[背层](../../../raw/current/scripts/prefabs/gelblob.lua#L625-L629)、[本体层](../../../raw/current/scripts/prefabs/gelblob.lua#L531-L535)
- **误区：`shadow_chester_swirl_fx` 是 `FX` 标签实体。** 它刻意只用 `NOCLICK`＋`CLASSIFIED`，注释说明加 `FX` 会使其父挂到切斯特后仍被鼠标悬停；自行加 `FX` 会改变交互。旋涡也没有自删任务，必须由 `SGchester` 调 `ReleaseSwirl` 才进入 `swirl_pst` 收尾。[标签](../../../raw/current/scripts/prefabs/chester.lua#L732-L740)、[收尾](../../../raw/current/scripts/prefabs/chester.lua#L715-L722)
- **边界：`firesuppressor_glow` 初始不可见。** 工厂把 alpha 设为 0，宿主 `SetWarningLevelLight` 把 `_ison` 置真后才开始淡入；`_ison` 是内部 `net_bool`，不是给外部直接读写的公开接口。它本身没有 Light 组件，宿主光照另设。[工厂](../../../raw/current/scripts/prefabs/firesuppressor.lua#L433-L446)、[淡入淡出](../../../raw/current/scripts/prefabs/firesuppressor.lua#L389-L422)

## 验证记录

- 源码审阅：2026-11-04，steam-buildid:26335394（实读 `raw/current/BUILD.txt`）。逐文件核对六个工厂的注册名、bank／build／动画、朝向、标签、端分支、`persists` 与清理；生成点追到 `SGbrightmare_gestalt`／`SGlunarthrall_plant_gestalt`／`SGbrightmare_invader` 的 `SpawnTrail`、`daywalker2:MakeBuried`／`MakeFreed`、`gestalt_cage:OnEquip`／`OnUnequip`、`gelblob:fn`、`SGchester` 的 `open`／`open_idle`、`firesuppressor:oninit`／`SetWarningLevelLight`／`OnRemoveEntity`；并核 `highlightchildren`、`colouradder`／`bloomer`、`gestaltcage` 的关系。以 `grep -rn --include='*.lua' -F '"注册名"'` 对六个完整注册名在 `raw/current/scripts` 全域反查：`gestalt_trail` 另命中三个 SG 与 `gestalt` 依赖表，`shadow_chester_swirl_fx` 另命中 `SGchester`，其余只命中自身工厂、依赖表与注册；未穷举运行时拼接名。
- 名称核验：`languages/chinese_s.po` 定向检索六个 FX 注册名均无 `STRINGS.NAMES.*` 键；宿主名键命中行见“注册与构造”。标题使用效果描述，并已注明不是官方译名。
- 分档说明：覆盖 6 个注册、6 个工厂文件、9 个宿主 prefab（`gestalt`、`corpse_gestalt`、`lunarthrall_plant_gestalt`、`gestalt_guard`、`daywalker2`、`gestalt_cage`、`gelblob`、`chester`、`firesuppressor`）与 1 个无宿主 SG，三种挂接（世界点／父挂＋Follower／仅父挂）与多种清理（定时自删／显式移除＋反登记／`ReleaseSwirl` 收尾／仅父挂回收），需要整体对照保留，故 `size_class: complex`。
- 游戏运行：未运行；未验证动画包实际帧数、循环标记、贴地层效果、淡入淡出观感与多人端时序。
- 结构检查：`validate_page.py` 通过（0 issue；158 行、28785 字节，低于 complex 档 400 行／32 KiB）；`validate_source_links.py --strict` 通过（errors 0、warnings 0；93 处源码引用、66 个唯一目标、15 个源文件）；两段 Lua 示例均经 `parser_probe.exe --syntax-only` 通过（只查语法，不执行）。
- 未覆盖：各宿主 prefab 的玩法本体、`gestalt_cage_swap_fx` 在存档恢复时的重挂路径、`gelblob_back_fx` 与 `gestalt_trail` 的引擎级回收、`SGbrightmare_invader` 是否在未包含的资源或动态路径中使用。

## 相关链接

- [[reference/prefabs/fx-world-index|特效 Prefab 索引·生物与环境]]、[[reference/prefabs/fx-index|特效 Prefab 索引]]
- [[reference/prefabs/king_cane_fx|宿主持有与跟随的附属 FX 族]]（同为多宿主附属 FX，可对照 `AttachToOwner` 与 Follower 用法）
- [[reference/prefabs/ice_puddle|宿主附属的贴地与贴水 FX 族]]、[[reference/prefabs/fence_gate_anim|宿主挂接的附属表现实体]]（宿主子实体与清理协议对照）
- [[reference/prefabs/daywalker2_swipe_fx|拾荒疯猪挥击地面特效]]、[[reference/prefabs/gelblob_attach_fx|恶液接触附着黏液]]（同宿主的其他 FX）
- [[reference/prefabs/lunar_goop_trail_fx|牧者黏液拖尾]]（地面拖尾的另一种生成／回收模式）
- [[reference/components/transparentonsanity|transparentonsanity]]、[[reference/components/highlight|highlight]]、[[reference/components/colouradder|colouradder]]、[[reference/components/bloomer|bloomer]]、[[reference/components/gestaltcage|gestaltcage]]、[[reference/components/equippable|equippable]]

返回 [[reference/prefabs/fx-shadow-index|暗影生物、裂隙生物与附着]]。
