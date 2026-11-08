---
title: "传送宝珠碎片（vault_orb_refined）"
created: "2026-11-10"
updated: "2026-11-10"
type: prefab
summary: "圣殿（Vault）路线的传送消耗品：断链的圣殿路标旁会生成未精炼传送宝珠，在圣殿锻台 1 换 3 精炼为本物品；堆叠 20，使用时在地图上选一个已放置的传送宝珠目的地并消耗 1 个把人瞬移过去。"
aliases: ["vault_orb_refined", "传送宝珠碎片", "宝库宝珠", "Portation Orb Fragment", "圣殿传送", "宝库传送", "vault_orb"]
tags: [prefab, item]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/vault_orb_refined.lua", "prefabs/vault_teleporter.lua", "prefabs/vaultroom_defs.lua", "components/vaultorbteleporter.lua", "components/vaultorbteleportdestination.lua", "actions.lua", "componentactions.lua", "constants.lua", "tuning.lua", "recipes.lua", "recipes_filter.lua", "prefabs/telebase.lua", "prefabs/telebase_gemsocket.lua", "prefabs/vault_key_trial.lua", "screens/redux/scrapbookdata.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# 传送宝珠碎片（vault_orb_refined）

## 快速判断

- 职责与适用：`prefabs/vault_orb_refined.lua` 只注册 `vault_orb_refined`，官方简中"传送宝珠碎片"，类别是 item：可捡、可叠 20、可交易、可作祟；组件 `vaultorbteleporter` 让它成为"先点地图目的地，再瞬移"的两段式消耗品。[注册](../../../raw/current/scripts/prefabs/vault_orb_refined.lua#L108)、[接线](../../../raw/current/scripts/prefabs/vault_orb_refined.lua#L90-L101)
- 优先使用：来路见[获取路径](#获取路径)；用法与目的地见[传送消费者](#传送消费者)；点不到目标见[常见陷阱与限制](#常见陷阱与限制)。
- 重要副作用：成功只消耗 **1 个**（`stackable:Get():Remove()`），目标点生成 `vault_portal_fx`、玩家 `Physics:Teleport` 过去，附 `SnapCamera` 与到达音效；不整组消耗、不换实体。[DoTeleport](../../../raw/current/scripts/prefabs/vault_orb_refined.lua#L9-L38)
- 修改边界：组件接口与 `bufferedmapaction` 协作归 [[reference/components/vaultorbteleporter]]；目的地组件归 [[reference/components/vaultorbteleportdestination]]；圣殿房间生成、断链规则与钥匙流程不在本页。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `vault_orb_refined` | `fn()`：实体 → Transform／AnimState／Network → `MakeInventoryPhysics` → bank／build `vault_orb_refined` 播 `idle` → `MakeInventoryFloatable(inst, "small", 0.05, { 0.8, 0.75, 0.8 })` → `SetPristine` → 客户端返回 → 主模拟：写 `swap_build`／`swap_symbol`／`crushitemcast_sound` → `tradable`／`inspectable`／`inventoryitem` → `stackable`（`STACK_SIZE_MEDITEM`） → `vaultorbteleporter`＋三个回调 → 监听 `stopcontinuousaction` → `MakeHauntableLaunch` | 唯一形态与唯一注册；`prefabs` 表只声明 `bufferedmapaction`（组件会 `SpawnPrefab` 它）。堆叠 20。[工厂](../../../raw/current/scripts/prefabs/vault_orb_refined.lua#L65-L106)、[依赖](../../../raw/current/scripts/prefabs/vault_orb_refined.lua#L5-L7) |

- **主模拟分界**：`SetPristine` 之前只有物理、动画、浮水；`swap_build`／`swap_symbol`（手持替换外观）、`crushitemcast_sound`、四个组件与事件监听都在 `if not TheWorld.ismastersim then return inst end` 之后。[端分界](../../../raw/current/scripts/prefabs/vault_orb_refined.lua#L80-L103)
- **堆叠是显式赋值**：`inst.components.stackable.maxsize = TUNING.STACK_SIZE_MEDITEM`，常量 20，与组件默认值相同；采集册记 `stacksize=20`。[赋值](../../../raw/current/scripts/prefabs/vault_orb_refined.lua#L94-L95)、[常量](../../../raw/current/scripts/tuning.lua#L82)、[采集册](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L1657)
- **没有游戏标签**：本 prefab 不调用 `AddTag`；`MakeHauntableLaunch` 只挂 `hauntable`。[全文件](../../../raw/current/scripts/prefabs/vault_orb_refined.lua#L65-L106)、[helper](../../../raw/current/scripts/standardcomponents.lua#L1053-L1068)

### 名称核验

`STRINGS.NAMES.VAULT_ORB_REFINED` 是 `Portation Orb Fragment`／"传送宝珠碎片"，`VAULT_ORB` 是 `Portation Orb`／"传送宝珠"。**"宝库宝珠"不是显示名**，只作检索别名。[两个名称键](../../../raw/current/scripts/languages/chinese_s.po#L351244-L351252)

## 获取路径

圣殿（`VIRTUALROOMSETS.VAULT`，旧称／俗称"宝库"）里断掉的传送链会生成**未精炼**宝珠，未精炼宝珠在圣殿锻台上换成碎片：

| 步骤 | 机制 | 证据 |
|---|---|---|
| 1. 断链生成未精炼宝珠 | `CreateTeleportersForRoomSet` 遍历房间四个方向的 link，`IsLinkBroken` 为真时对该路标（`vault_teleporter`）调 `MakeBroken()` 再 `SpawnOrb()` | [房间装配](../../../raw/current/scripts/prefabs/vaultroom_defs.lua#L173-L188) |
| 2. 掉在路标旁 | `SpawnOrb`：极坐标半径 1–1.5、随机角，`SpawnPrefab("vault_orb")` 后设位置 | [SpawnOrb](../../../raw/current/scripts/prefabs/vault_teleporter.lua#L396-L403) |
| 3. 未精炼宝珠两种去向 | ① 交给断链路标的 `trader`（只认 `vault_orb`）→ `OnRepair` 记入 `repairedlinks` 并永久接通；② 拿去精炼 | [交易判定](../../../raw/current/scripts/prefabs/vault_teleporter.lua#L318-L320)、[修复](../../../raw/current/scripts/prefabs/vault_teleporter.lua#L329-L355) |
| 4. 精炼 1 换 3（限圣殿锻台） | 配方 `vault_orb_refined`＝1 个未精炼宝珠，`TECH.VAULT_REFINE_ONE`、`nounlock`、`no_deconstruction`、`numtogive = 3`；它列在 `recipes_filter` 的 `vault_refiner_pedestal` 段，锻台由钥匙试炼的 `OpenPlate` 打开 | [配方](../../../raw/current/scripts/recipes.lua#L1086)、[站点](../../../raw/current/scripts/recipes_filter.lua#L575-L577)、[开启](../../../raw/current/scripts/prefabs/vault_key_trial.lua#L31-L41) |

- **未精炼宝珠不是可堆叠物品**：`orbfn` 没有 `stackable`，并带 `irreplaceable`／`forcedtosavethroughvirtualrooms`／`donotautopick` 标签——不会自动捡起，跨虚拟房间也强制保存。[orbfn](../../../raw/current/scripts/prefabs/vault_teleporter.lua#L615-L647)
- **精炼门槛**：`TECH.VAULT_REFINE_ONE` 只含 `VAULT_REFINE = 1` 一级；配方 `nounlock = true`（不需蓝图）、`no_deconstruction = true`（拆不掉）。[科技定义](../../../raw/current/scripts/constants.lua#L1340)

## 传送消费者

| 环节 | 机制 | 证据 |
|---|---|---|
| 从背包发起 | `componentactions` 给 `vaultorbteleporter` 物品插入动作：没有待选地图动作时给 `ACTIONS.STARTVAULTORBTELEPORT` | [候选](../../../raw/current/scripts/componentactions.lua#L3093-L3101) |
| 开始选点 | `STARTVAULTORBTELEPORT.fn` → `vaultorbteleporter:StartMapAction(doer)`；组件 `SpawnPrefab("bufferedmapaction")`、`SetupMapAction(ACTIONS.VAULTORBTELEPORT_MAP, inst, doer)`，返回 true | [动作](../../../raw/current/scripts/actions.lua#L7002-L7006)、[组件](../../../raw/current/scripts/components/vaultorbteleporter.lua#L43-L58) |
| 选点期间 | `OnStartMapAction` 给施法者加标签 `vaultorbteleportdestinationtracker`；取消时 `OnCancelMapAction` 移除并推 `interruptcontinuousaction` | [回调](../../../raw/current/scripts/prefabs/vault_orb_refined.lua#L49-L59) |
| 地图校验 | `VAULTORBTELEPORT_MAP.maponly_checkvalidpos_fn`：在点击点 `VAULT_ORB_REFINED_DETECTION_RADIUS`＝16 世界单位（`TILE_SCALE`＝4，约 4 格）内找同时带 `CLASSIFIED`／`globalmapicon`／`vaultorbteleportdestinationtrackericon` 的实体，再要 `IsTeleportingPermittedFromPointToPoint` 通过 | [校验](../../../raw/current/scripts/actions.lua#L7008-L7034)、[半径](../../../raw/current/scripts/tuning.lua#L9459) |
| 执行 | `Activate(doer, mapent._target)`：目标实体必须有 `vaultorbteleportdestination` 组件，否则 `"NOTARGET"`；通过后推 `vault_teleport` 事件，`onplayerready` 回调里才真正 `DoTeleport` | [组件 Activate](../../../raw/current/scripts/components/vaultorbteleporter.lua#L67-L80)、[OnActivate](../../../raw/current/scripts/prefabs/vault_orb_refined.lua#L40-L47) |
| 目的地：三碎片 | 三座同装 `vault_orb_refined` 且都能交互 → `getstatus`＝`VALID`，`OnGemChange` 才加 `vaultorbteleportdestination`（连出 `vaultorbteleportdestinationtrackericon` 地图图标）；缺一座／空座 → `GEMS` 并移除该组件；试混装会被宝石座以 `WRONGGEM` 拒绝 | [全真](../../../raw/current/scripts/prefabs/telebase.lua#L34-L41)、[同种](../../../raw/current/scripts/prefabs/telebase_gemsocket.lua#L14-L16)、[加／移除](../../../raw/current/scripts/prefabs/telebase.lua#L43-L68)、[状态](../../../raw/current/scripts/prefabs/telebase.lua#L116-L118) |
| 目的地：三紫宝石 | 同样 `VALID`、可被法杖传送到该焦点（`FindNearestActiveTelebase` 要 `validteleporttarget`），但 `requiredgem` 不是碎片 → **不挂**地图目的地；法杖传送会打碎紫宝石，碎片不会（`ShouldDestroyGem` 对碎片返回 false） | [法杖取焦点](../../../raw/current/scripts/prefabs/telebase.lua#L84-L112)、[防碎](../../../raw/current/scripts/prefabs/telebase_gemsocket.lua#L56-L63) |

- **消耗与表现**：`DoTeleport` 先取目标位置（目标无效则退化为自己），生成 `vault_portal_fx`，`doer.Physics:Teleport(x, 0, z)`（无 Physics 改 `Transform:SetPosition`），玩家另 `SnapCamera` 与到达音效；最后才删 1 个碎片。[DoTeleport](../../../raw/current/scripts/prefabs/vault_orb_refined.lua#L9-L38)
- **中断也能收尾**：`inst:ListenForEvent("stopcontinuousaction", OnStopContinuousAction)` → `vaultorbteleporter:CancelMapAction()`；组件自己的 `OnRemoveEntity`／`OnRemoveFromEntity` 也会清掉 `bufferedmapaction`。[监听](../../../raw/current/scripts/prefabs/vault_orb_refined.lua#L61-L63)、[清理](../../../raw/current/scripts/components/vaultorbteleporter.lua#L17-L29)

## 堆叠与标签

| 属性 | 本物品 | 证据 |
|---|---|---|
| 堆叠 | 20（`STACK_SIZE_MEDITEM`） | [赋值](../../../raw/current/scripts/prefabs/vault_orb_refined.lua#L94-L95) |
| 游戏标签／交易 | 无 `AddTag`；有 `tradable`、`goldvalue` 默认 0 | [全文件](../../../raw/current/scripts/prefabs/vault_orb_refined.lua#L65-L106)、[默认值](../../../raw/current/scripts/components/tradable.lua#L1-L7) |

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`vaultorbteleporter` | 三个回调（激活／开始选点／取消）＋ `CancelMapAction` | 仅主模拟 | [[reference/components/vaultorbteleporter]]；[接线](../../../raw/current/scripts/prefabs/vault_orb_refined.lua#L97-L101) |
| 组件：`vaultorbteleportdestination` | 不在本物品上，在目标上 | 目的地实体侧 | [[reference/components/vaultorbteleportdestination]]；[校验收口](../../../raw/current/scripts/components/vaultorbteleporter.lua#L67-L70) |
| 组件：`stackable`／`inventoryitem`／`tradable`／`inspectable`／`hauntable` | 20 叠、背包、交易、检视、作祟抛飞 | 仅主模拟 | [[reference/components/stackable]]；[接线](../../../raw/current/scripts/prefabs/vault_orb_refined.lua#L90-L95)、[helper](../../../raw/current/scripts/standardcomponents.lua#L1053-L1068) |
| 关联 Prefab：`bufferedmapaction` | 选点中介，承载 `VAULTORBTELEPORT_MAP` | `StartMapAction` 时生成、取消／移除时清理 | [组件](../../../raw/current/scripts/components/vaultorbteleporter.lua#L54-L58)（Wiki 待建） |
| 关联 Prefab：`vault_orb` | 未精炼宝珠，本物品的原料；也用于接通断链传送台 | 仅圣殿路径 | [orbfn](../../../raw/current/scripts/prefabs/vault_teleporter.lua#L615-L647) |
| 关联 Prefab：`vault_portal_fx` | 传送落点特效 | 每次 `DoTeleport` | [生成](../../../raw/current/scripts/prefabs/vault_orb_refined.lua#L17) |
| 关联 Prefab：`telebase`／`telebase_gemsocket` | 目的地设施：三座同种宝石且全真才 `VALID`；只有碎片才挂地图目的地 | `requiredgem == "vault_orb_refined"`＋三座 `caninteractwith` 全真；紫宝石仅法杖 | [telebase](../../../raw/current/scripts/prefabs/telebase.lua#L43-L68)（Wiki 待建） |
| 数据：`TECH.VAULT_REFINE_ONE` | 精炼配方的科技门槛 | `VAULT_REFINE = 1` | [定义](../../../raw/current/scripts/constants.lua#L1340) |
| 资源：`anim/vault_orb_refined.zip` | bank／build 同名，播 `idle`；另有 `swap_symbol = "swap_vault_orb_refined_gem"` 供手持外观 | 原版资源，不随页分发 | [assets](../../../raw/current/scripts/prefabs/vault_orb_refined.lua#L1-L3)、[swap](../../../raw/current/scripts/prefabs/vault_orb_refined.lua#L86-L88) |

## 常见陷阱与限制

- **误区：这是"宝库宝珠"。** 官方简中是"传送宝珠碎片"，未精炼那件才叫"传送宝珠"；两者不同注册（`vault_orb` 见 `vault_teleporter.lua`），掉落、堆叠与用途都不同。[名称](../../../raw/current/scripts/languages/chinese_s.po#L351244-L351252)、[未精炼注册](../../../raw/current/scripts/prefabs/vault_teleporter.lua#L650)
- **边界：三颗同种宝石不等于三颗紫宝石就够。** 三紫宝石只让状态 `VALID`、供法杖使用；地图目的地只在全座有效且 `requiredgem == "vault_orb_refined"` 时挂上，三碎片才同时满足；缺一座／空座回到 `GEMS`，混装被 `WRONGGEM` 拒绝，取走或碎裂任一宝石即移除目的地。[校验](../../../raw/current/scripts/actions.lua#L7008-L7034)、[组件](../../../raw/current/scripts/components/vaultorbteleporter.lua#L67-L70)、[三座全真](../../../raw/current/scripts/prefabs/telebase.lua#L34-L41)、[加／移除](../../../raw/current/scripts/prefabs/telebase.lua#L43-L68)
- **边界：一次只消耗 1 个。** 无论堆叠多少，`DoTeleport` 只 `Get():Remove()` 一个；没有"整组传送"的分支。[消耗](../../../raw/current/scripts/prefabs/vault_orb_refined.lua#L31-L37)
- **边界：精炼是 1 换 3 且拆不回来。** `no_deconstruction = true`；未精炼宝珠另有"接通断链圣殿路标"一条消耗路径。[配方](../../../raw/current/scripts/recipes.lua#L1086)、[修复消耗](../../../raw/current/scripts/prefabs/vault_teleporter.lua#L329-L355)
- **边界：判定半径是 16 世界单位（约 4 格）。** `VAULT_ORB_REFINED_DETECTION_RADIUS = 16`，1 格＝`TILE_SCALE`＝4，点偏即 `"NOTARGET"`；还要过 `IsTeleportingPermittedFromPointToPoint`（内部规则未展开）。[半径](../../../raw/current/scripts/tuning.lua#L9459)、[格宽](../../../raw/current/scripts/constants.lua#L17)、[校验](../../../raw/current/scripts/actions.lua#L7021-L7033)
- **未核验**：未运行游戏。圣殿房间何时断链、`repairedlinks` 持久化范围、`IsTeleportingPermittedFromPointToPoint` 细则与 `bufferedmapaction` 生命周期都未实测。

## 验证记录

- 源码审阅：2026-11-10，Steam buildid 26474873。`vault_orb_refined.lua` 108 行与 `vaultorbteleporter`／`vaultorbteleportdestination` 全文；`vault_teleporter.lua` 的 `ItemTradeTest`／`OnRepair`／`SpawnOrb`／`orbfn`；`vaultroom_defs.lua`、`actions.lua`、`componentactions.lua`、`telebase.lua`／`telebase_gemsocket.lua`／`vault_key_trial.lua`、`recipes.lua`／`recipes_filter.lua`、`constants.lua`／`tuning.lua`、`scrapbookdata.lua`、`chinese_s.po`。
- 未覆盖：圣殿钥匙与试炼流程；`bufferedmapaction` 内部实现；`vault_teleporter` 的供电与房间逻辑。
- 游戏运行：未运行；数值只有堆叠 20、精炼 3 与半径 16，均为绝对值。
- 复审修订：2026-11-10，P1 目的地门禁、P3 半径单位与路径用词；R01 三组装填结果。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 见交付回报。

## 相关链接

- 组件：[[reference/components/vaultorbteleporter]]、[[reference/components/vaultorbteleportdestination]]
- 圣殿与远古材料：[[reference/prefabs/thulecite]]、[[reference/prefabs/vault_compass_visual]]（同区道具的视觉件）

返回 [[reference/prefabs/index|Prefab 索引]]。
