---
title: "双尾心（wortox_reviver）"
created: "2026-11-09"
updated: "2026-11-10"
type: prefab
summary: "沃拓克斯「生命使者」技能线的双尾心：10 灵魂配方、按 userid 绑定主人的 linkeditem、幽灵收取复活、施法传送到主人身边并按剩余鲜度返还灵魂、10 天（正常计时）腐坏返还 1 灵魂。"
aliases: ["wortox_reviver", "双尾心", "Twintailed Heart", "沃拓克斯双尾心"]
tags: [prefab, item]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/wortox_reviver.lua", "prefabs/skilltree_wortox.lua", "prefabs/wortox_soul.lua", "prefabs/player_common.lua", "prefabs/ghost.lua", "components/linkeditem.lua", "components/linkeditemmanager.lua", "components/spellcaster.lua", "components/perishable.lua", "components/inventoryitem.lua", "components/builder.lua", "stategraphs/SGwilson.lua", "actions.lua", "componentactions.lua", "entityscript.lua", "componentutil.lua", "constants.lua", "recipes.lua", "recipes_filter.lua", "fx.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 双尾心（wortox_reviver）

## 快速判断

- 职责与适用：一件物品两种用途——① 带 `reviver` 标签的复活道具，幽灵玩家收下即复活；② `spellcaster` 施法物品，把施法者送到双尾心主人身边，主人自己施法则把灵魂退回来。[复活](../../../raw/current/scripts/prefabs/player_common.lua#L333-L366)、[施法](../../../raw/current/scripts/prefabs/wortox_reviver.lua#L83-L138)
- 优先使用：主人绑定机制读 [[reference/components/linkeditem]] 与 [[reference/components/linkeditemmanager]]；施法门禁与目标条件读 [[reference/components/spellcaster]]；10 天腐坏读 [[reference/components/perishable]]。
- 重要副作用：配方要 10 个 `wortox_soul`；腐坏时返还 1 个灵魂且不按鲜度缩放；任何一条施法分支成功后本件都被消耗。
- 修改边界：生命使者三个技能节点在 `skilltree_wortox.lua`（Wiki 待建）；`wortox_soul` 物品页 Wiki 待建；`wortox_reviver_body` 的注册细节归 [[reference/prefabs/wortox_soulecho_buff_fx|沃拓克斯技能与装备表现族]]，本页只写宿主侧的挂接与清理。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `wortox_reviver` | `fn` 单工厂：网络段（Transform／AnimState／SoundEmitter／Network；`MakeInventoryPhysics`；bank 与 build 均 `wortox_reviver`、播 `idle` 循环；加 `reviver`、`show_spoilage`、`crushitemcast`、`wortox_reviver_lock_spellcaster` 四个标签；`inst.spelltype = "SQUEEZE"`；挂 `linkeditem` 并设 `displaynamefn`）→ `SetPristine` → 客户端返回 → master 段（缓存配方灵魂数、`swap_build`／`swap_symbol`、OnStartBody／OnStopBody／OnConsume／OnBuiltFn／OnLoad 等钩子、`crushitemcast_sound`；inventoryitem／inspectable／tradable／perishable／spellcaster／hauntable；最后 `SetAllowConsumption(false)` 并接上 linkeditem 的两个回调） | 无变体、无形态切换。不可堆叠（没有 `stackable`）；`SetSinks(true)` 让它掉进海里下沉。[构造 L200-L267](../../../raw/current/scripts/prefabs/wortox_reviver.lua#L200-L267)、[入包与下沉 L242-L244](../../../raw/current/scripts/prefabs/wortox_reviver.lua#L242-L244) |

关联注册：`wortox_reviver_body` 是同一文件里的独立注册，主注册与工厂归 [[reference/prefabs/wortox_soulecho_buff_fx|沃拓克斯技能与装备表现族]]；本页只写宿主侧 OnStartBody／OnStopBody 挂接，不重复工厂行。

- 名称：`STRINGS.NAMES.WORTOX_REVIVER`＝双尾心；`STRINGS.NAMES.WORTOX_REVIVER_FMT`＝“{name}的双尾心”。`displaynamefn` 在 `SetPristine` 之前就挂上、双端都有，读名字时由 `GetBasicDisplayName` 调用；拿不到主人名时回退到前者。[po L352500-L352507](../../../raw/current/scripts/languages/chinese_s.po#L352500-L352507)、[挂载 L221](../../../raw/current/scripts/prefabs/wortox_reviver.lua#L221)、[显示名 L148-L151](../../../raw/current/scripts/prefabs/wortox_reviver.lua#L148-L151)、[读取 L687-L691](../../../raw/current/scripts/entityscript.lua#L687-L691)
- 四个标签分工：`reviver` 是幽灵收取的门禁；`show_spoilage` 只是新鲜度图标；`crushitemcast` 决定施法动作走"捏碎手中物"的表现；`wortox_reviver_lock_spellcaster` 是给客户端看的锁投影。[L214-L218](../../../raw/current/scripts/prefabs/wortox_reviver.lua#L214-L218)
- `spelltype = "SQUEEZE"` 是实体字段而非组件字段，只用于 `ACTIONS.CASTSPELL.strfn` 取动作名（简中"挤压"）。[L217](../../../raw/current/scripts/prefabs/wortox_reviver.lua#L217)、[strfn](../../../raw/current/scripts/actions.lua#L3441-L3443)、[po L489-L492](../../../raw/current/scripts/languages/chinese_s.po#L489-L492)

## 获得：技能配方与灵魂成本

- 配方：`wortox_soul` 10，`TECH.NONE`，`builder_skill = "wortox_lifebringer_1"`；制作门禁走同一套技能检查（`skilltreeupdater:IsActivated`）。[recipes L355](../../../raw/current/scripts/recipes.lua#L355)、[服务端门禁](../../../raw/current/scripts/components/builder.lua#L876-L877)
- 灵魂成本缓存：首次生成双尾心时读一次 `AllRecipes.wortox_reviver.ingredients`，累加所有 `wortox_soul` 的数量，读不到或为 0 时回退 10；这个数只用于"自用施法返还灵魂"的按比例计算。[L15-L31](../../../raw/current/scripts/prefabs/wortox_reviver.lua#L15-L31)、[首次初始化 L228-L230](../../../raw/current/scripts/prefabs/wortox_reviver.lua#L228-L230)
- 灵魂物品 `wortox_soul` 自身可堆叠（`maxsize = STACK_SIZE_SMALLITEM`＝40），Wiki 待建。[L309-L311](../../../raw/current/scripts/prefabs/wortox_soul.lua#L309-L311)
- 制作菜单把双尾心同时列进沃拓克斯角色组与治疗类过滤组。[L311](../../../raw/current/scripts/recipes_filter.lua#L311)、[L1154](../../../raw/current/scripts/recipes_filter.lua#L1154)

## 主人绑定：linkeditem

- 绑定条件（`TryToAttachWortoxID`）：owner 存在且不是快照游客会话、`linkeditem` 还没有 owner userid、且 owner 已激活 `wortox_lifebringer_1` → `LinkToOwnerUserID(owner.userid)`。[L33-L45](../../../raw/current/scripts/prefabs/wortox_reviver.lua#L33-L45)
- 四个触发点：进背包（`inventoryitem:SetOnPutInInventoryFn`）、建造回调（`OnBuiltFn`，由 `EntityScript:OnBuilt` 调用）、读档后 0 帧按 `GetGrandOwner` 再试一次、以及生命使者一阶激活时遍历当前背包逐件重试。[L47-L62](../../../raw/current/scripts/prefabs/wortox_reviver.lua#L47-L62)、[OnBuilt](../../../raw/current/scripts/entityscript.lua#L1725-L1727)、[技能侧 L469-L471](../../../raw/current/scripts/prefabs/skilltree_wortox.lua#L469-L471)、[重试函数 L74-L77](../../../raw/current/scripts/prefabs/skilltree_wortox.lua#L74-L77)
- 解绑：技能关闭时由技能树遍历该玩家的双尾心调 `LinkToOwnerUserID(nil)`；主人实体被移除时物品把施法关上（`SetAllowConsumption(false)`）。[技能侧 L472-L476](../../../raw/current/scripts/prefabs/skilltree_wortox.lua#L472-L476)、[物品侧 L70-L72](../../../raw/current/scripts/prefabs/wortox_reviver.lua#L70-L72)
- 管理器接力：世界 `linkeditemmanager` 在 `ms_registerlinkeditem` 时 `SetOwnerInst(player)`，玩家技能树就绪后再回调 `OnSkillTreeInitialized`；后者正是生命使者三阶解锁施法的入口。[注册 L99-L113](../../../raw/current/scripts/components/linkeditemmanager.lua#L99-L113)、[初始化 L20-L32](../../../raw/current/scripts/components/linkeditemmanager.lua#L20-L32)、[物品回调 L74-L78](../../../raw/current/scripts/prefabs/wortox_reviver.lua#L74-L78)
- 网络与显示：`netowneruserid`／`netownername` 两个 net_string；`GetOwnerName` 每 10 秒最多查一次客户端表，离线时退回缓存的 netownername。[linkeditem L6-L8](../../../raw/current/scripts/components/linkeditem.lua#L6-L8)、[L26-L43](../../../raw/current/scripts/components/linkeditem.lua#L26-L43)

## 施法：目标、灵魂返还与传送

- 组件配置：`canuseontargets`、`canusefrominventory`、`canonlyuseonlocomotorspvp` 三个开关；后者的含义是目标必须带 `locomotor`，且 `target == doer`，或 PVP 开启，或双方不都是玩家。[L254-L259](../../../raw/current/scripts/prefabs/wortox_reviver.lua#L254-L259)、[目标判定](../../../raw/current/scripts/components/spellcaster.lua#L190-L199)
- 锁与解锁：构造末尾 `SetAllowConsumption(false)` 把 `spelltype` 设成 `SPELLTYPES.WORTOX_REVIVER_LOCK`；`CanCast` 在 `spelltype` 非 nil 时要求施法者带 `wortox_reviver_lock_spelluser` 标签，而全库没有任何原版代码授予该标签，所以未解锁时谁都放不出。生命使者三阶激活时改成 `SetAllowConsumption(true)`（`spelltype` 置 nil）。[锁 L64-L68](../../../raw/current/scripts/prefabs/wortox_reviver.lua#L64-L68)、[常量 L1986](../../../raw/current/scripts/constants.lua#L1986)、[CanCast 门禁](../../../raw/current/scripts/components/spellcaster.lua#L152-L156)、[三阶 L486-L505](../../../raw/current/scripts/prefabs/skilltree_wortox.lua#L486-L505)
- 客户端可见性：动作候选由 `componentactions` 按 `<类型>_spellcaster` 与 `<类型>_spelluser` 成对判断，另外还要 `castfrominventory`、`castontargets`／`castonlocomotors(pvp)` 这几组标签；本件只在 pristine 预置了锁标签，其余标签由服务端组件的属性 setter 事后加，全库也没有别的 prefab 把它们写进 pristine。客户端副本如何拿到这些标签未运行验证。[四条门禁](../../../raw/current/scripts/componentactions.lua#L1774-L1801)、[预置 L218](../../../raw/current/scripts/prefabs/wortox_reviver.lua#L218)、[标签由组件加](../../../raw/current/scripts/components/spellcaster.lua#L1-L47)
- 结算 `SpellFn` 的三条分支（[L83-L138](../../../raw/current/scripts/prefabs/wortox_reviver.lua#L83-L138)）：
  1. 主人实体不在（未绑定或已离线）→ 推 `wortox_reviver_failteleport` 让角色说一句，物品不消耗；
  2. 主人就是施法者自己 → 临时打开 `wortox_ignoresoulcounts` 绕过灵魂计数，按 `ceil(鲜度 × 缓存成本)` 生成等量 `wortox_soul` 丢在脚下，播音效，进 SG 状态 `wortox_teleport_reviver_selfuse`；
  3. 主人是别人 → 先过 `IsTeleportingPermittedFromPointToPoint`（WagPunk 竞技场屏障、虚拟房间集），不通过同样推失败事件；通过后在主人周围 6→1 半径找 `NoHoles`（不在洞口附近）的可行走点，记平台与偏移，跨屏时 snapcamera，进 SG 状态 `wortox_teleport_reviver`。[失败与自用 L89-L112](../../../raw/current/scripts/prefabs/wortox_reviver.lua#L89-L112)、[传送许可](../../../raw/current/scripts/componentutil.lua#L1390-L1408)、[落点 L122-L136](../../../raw/current/scripts/prefabs/wortox_reviver.lua#L122-L136)
- 结算后的消耗：传送分支在 `wortox_teleport_reviver_pst.onenter` 里先 `perishable:StartPerishing()` 再 `OnConsume`；自用分支在 `wortox_teleport_reviver_selfuse.onenter` 里按 StartPerishing → OnStopBody → OnConsume 的顺序收尾。入口传送状态 `wortox_teleport_reviver` 禁控并开无敌；`_pst` 在 15.2 帧之前继续维持无敌，到 `FrameEvent(15.2)` 关闭（onexit 再兜底关一次）。[pst L16833-L16898](../../../raw/current/scripts/stategraphs/SGwilson.lua#L16833-L16898)、[15.2 帧关无敌 L16894-L16898](../../../raw/current/scripts/stategraphs/SGwilson.lua#L16894-L16898)、[pst onexit 兜底 L16919-L16921](../../../raw/current/scripts/stategraphs/SGwilson.lua#L16919-L16921)、[自用 L16752-L16763](../../../raw/current/scripts/stategraphs/SGwilson.lua#L16752-L16763)、[入口开无敌与禁控 L16791-L16794](../../../raw/current/scripts/stategraphs/SGwilson.lua#L16791-L16794)、[入口 onexit L16815-L16826](../../../raw/current/scripts/stategraphs/SGwilson.lua#L16815-L16826)、[OnConsume](../../../raw/current/scripts/prefabs/wortox_reviver.lua#L140-L146)
- `crushitemcast` 的时机限制：骑乘或搬重物时不进入候选动作。[L1796-L1799](../../../raw/current/scripts/componentactions.lua#L1796-L1799)、[动作侧](../../../raw/current/scripts/actions.lua#L3456-L3463)

## 腐坏与返还灵魂

- 计时定义：`TUNING.SKILLS.WORTOX.REVIVE_PERISH_TIME = 10 × total_day_time × perish_warp`＝10×480＝**4800 秒**（10 天）——这是**正常计时下**的定义值，施法被打断时计时会暂停（见下一条）。[tuning L7394](../../../raw/current/scripts/tuning.lua#L7394)、[基准 L19-L20](../../../raw/current/scripts/tuning.lua#L19-L20)、[赋值 L249-L252](../../../raw/current/scripts/prefabs/wortox_reviver.lua#L249-L252)
- 结算：`perishable:Update` 乘 `localPerishMultiplyer`（默认 1）；无主时再乘 `PERISH_GROUND_MULT`＝1.5 → 放地上约 **3200 秒**，在背包里按 4800 秒走。[公式](../../../raw/current/scripts/components/perishable.lua#L127-L133)、[地面分支](../../../raw/current/scripts/components/perishable.lua#L97-L105)
- `OnPerish` 按 `inventoryitem.owner` 分三种：owner 是玩家（有 `inventory`）→ 生成 1 个 `wortox_soul`（用 `wortox_ignoresoulcounts` 绕过计数）掉在 owner 脚下，随后移除本件；owner 是容器等非玩家实体 → 灵魂落在该实体的世界坐标，本件同样移除；没有 owner（在地上）→ 灵魂落原位，本件 `persists = false`、`canbepickedup = false`，播 `fallteleport` 并在 `animover` 时移除。[L153-L178](../../../raw/current/scripts/prefabs/wortox_reviver.lua#L153-L178)
- 施法的三路与计时：`crushitemcast.onenter` 一进来就 `perishable:StopPerishing()`（[L16662-L16664](../../../raw/current/scripts/stategraphs/SGwilson.lua#L16662-L16664)）。① **动作前被打断**（例如 `attacked` 命中电击分支走 `electrocute`）时走非 crushcasting 的 onexit：只清 `swap_remote` 与 body，不消耗也不重启计时，鲜度就停在暂停状态（[onexit L16705-L16713](../../../raw/current/scripts/stategraphs/SGwilson.lua#L16705-L16713)、[电击中断 L1930-L1933](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1930-L1933)）；`perishable` 存档会写 `paused`，读档仍是暂停（[L329-L349](../../../raw/current/scripts/components/perishable.lua#L329-L349)）。② **动作已执行但失败**：进入 `crushitemcast_fail`，其 onenter 重启计时，**物品保留、不消耗**（[L16720-L16726](../../../raw/current/scripts/stategraphs/SGwilson.lua#L16720-L16726)）。③ **成功**：两条成功路径——自用分支 `wortox_teleport_reviver_selfuse` 与传送分支 `wortox_teleport_reviver_pst`——各自重启计时并 `OnConsume`（[自用 L16755-L16763](../../../raw/current/scripts/stategraphs/SGwilson.lua#L16755-L16763)、[pst L16861-L16866](../../../raw/current/scripts/stategraphs/SGwilson.lua#L16861-L16866)）。
- 与自用施法的差别：腐坏恒返还 **1 个**；自用施法按剩余鲜度返还 `ceil(percent × 10)`，可以到 10 个。[L102](../../../raw/current/scripts/prefabs/wortox_reviver.lua#L102)

## 复活用途：幽灵收取

- 动作：目标带 `playerghost` 且物品带 `reviver` → 给出 `GIVETOPLAYER`。[L1403-L1406](../../../raw/current/scripts/componentactions.lua#L1403-L1406)
- 收取门禁：幽灵只在站在可通行点时接受 `reviver` 类物品；收下后 `OnGetItem` 依次做：播音、判定生命惩罚、`usereviver` 事件、`item:Remove()`、`respawnfromghost`、给给予者理智奖励。[L333-L366](../../../raw/current/scripts/prefabs/player_common.lua#L333-L366)
- 惩罚豁免：物品带 `noreviverhealthpenalty`，或（本件且给予者已激活 `wortox_lifebringer_2`）时免掉 `REVIVE_HEALTH_PENALTY`＝0.25；施救者固定拿 `REVIVE_OTHER_SANITY_BONUS`＝80 理智。[L346-L360](../../../raw/current/scripts/prefabs/player_common.lua#L346-L360)、[tuning L2177-L2178](../../../raw/current/scripts/tuning.lua#L2177-L2178)
- 条件不满足时不会给专用台词：玩家幽灵用 `SetAcceptTest(ShouldAcceptItem)` 只返回布尔，本件也没有设 `onrefuse`；`ACTIONFAIL.GIVE.GHOSTHEART` 属于敌对 ghost 的 `AbleToAcceptTest`（它对带 `reviver` 的物品返回该 reason），别套到玩家幽灵这条链上。[接受判定 L333-L338](../../../raw/current/scripts/prefabs/player_common.lua#L333-L338)、[trader 接线 L2786-L2790](../../../raw/current/scripts/prefabs/player_common.lua#L2786-L2790)、[敌对 ghost 的 reason L10-L11](../../../raw/current/scripts/prefabs/ghost.lua#L10-L11)

## 身体子实体：宿主侧挂接

- `OnStartBody`：先移除旧 body，`SpawnPrefab("wortox_reviver_body")` 并反向登记 `wortox_reviver_body.wortox_reviver = inst`，监听它的 `onremove` 清反向字段；随后 `entity:SetParent(owner.entity)`＋`AddFollower`＋`FollowSymbol(owner.GUID, "swap_remote", 25, -42, 0)`。`OnStopBody` 只负责移除。[L180-L198](../../../raw/current/scripts/prefabs/wortox_reviver.lua#L180-L198)
- 触发时机：CASTSPELL 的 ActionHandler 对带 `crushitemcast` 的物品选的是 `crushitemcast` 状态（`crushitemcast_holding`／`_trigger` 属 `STARTVAULTORBTELEPORT` 链，与本件无关）。`crushitemcast` 第 9 帧调 `OnStartBody`，非 `crushcasting` 的 onexit 调 `OnStopBody`；`crushitemcast_fail` 的 onexit、传送状态 onexit、自用状态 onenter 也各补一次收尾。body 的网格与材质细节见表现族页。[选态 L1303-L1310](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1303-L1310)、[持握状态 L16648-L16682](../../../raw/current/scripts/stategraphs/SGwilson.lua#L16648-L16682)、[正常收尾 L16705-L16713](../../../raw/current/scripts/stategraphs/SGwilson.lua#L16705-L16713)、[失败收尾 L16739-L16745](../../../raw/current/scripts/stategraphs/SGwilson.lua#L16739-L16745)、[[reference/prefabs/wortox_soulecho_buff_fx]]

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`linkeditem` | 记录 owner userid／name，负责绑定、解绑与技能树回调 | 双端 AddComponent（netvar 在客户端也建），服务端才设 owner_inst | [[reference/components/linkeditem]]；[L220-L221](../../../raw/current/scripts/prefabs/wortox_reviver.lua#L220-L221) |
| 组件：`linkeditemmanager`（世界级） | 登记物品、玩家进出时重挂、技能树就绪后回调 | 仅主端 | [[reference/components/linkeditemmanager]]；[L99-L113](../../../raw/current/scripts/components/linkeditemmanager.lua#L99-L113) |
| 组件：`spellcaster` | 施法入口、目标条件、锁标签门禁、`OnConsume` 时机由 SG 调 | 服务端组件；客户端只读标签 | [[reference/components/spellcaster]]；[L254-L259](../../../raw/current/scripts/prefabs/wortox_reviver.lua#L254-L259) |
| 组件：`perishable` | 10 天计时＋`OnPerish` 返还灵魂 | 服务端；腐烂分支不生成替换物 | [[reference/components/perishable]]；[L249-L252](../../../raw/current/scripts/prefabs/wortox_reviver.lua#L249-L252) |
| 组件：`inventoryitem` | 进包回调绑主人、`SetSinks(true)` 海里下沉 | 服务端 | [[reference/components/inventoryitem]]；[L242-L244](../../../raw/current/scripts/prefabs/wortox_reviver.lua#L242-L244) |
| SG：`SGwilson`（角色侧） | `crushitemcast` 持握（第 9 帧 OnStartBody）与 `crushitemcast_fail` 收尾，`wortox_teleport_reviver` 入口演出＋`_pst` 落点，`_selfuse` 自用 | 角色状态机；只在施法时进入 | [持握与收尾 L16648-L16745](../../../raw/current/scripts/stategraphs/SGwilson.lua#L16648-L16745)、[传送 L16784-L16827](../../../raw/current/scripts/stategraphs/SGwilson.lua#L16784-L16827) |
| API／数据：`SPELLTYPES.WORTOX_REVIVER_LOCK` | 锁用的法术类型名，拼出 `_spellcaster`／`_spelluser` 标签 | 双端标签约定 | [constants L1986](../../../raw/current/scripts/constants.lua#L1986)、[spellcaster 属性](../../../raw/current/scripts/components/spellcaster.lua#L130-L132) |
| 关联 Prefab：`wortox_soul` | 配方成本与两种返还的产物 | Wiki 待建；可堆叠 40 | [wortox_soul L309-L311](../../../raw/current/scripts/prefabs/wortox_soul.lua#L309-L311) |
| 关联 Prefab：`wortox_reviver_body` | 施法持握期间的联网跟随体 | 注册与资源归表现族页 | [[reference/prefabs/wortox_soulecho_buff_fx]]；[L269-L292](../../../raw/current/scripts/prefabs/wortox_reviver.lua#L269-L292) |
| 关联 Prefab：`wortox_teleport_reviver_top`／`_bottom` | 传送演出在起点与落点各生成一个 | 归灵魂生成 FX 页 | [[reference/prefabs/wortox_soul_spawn_fx]]；[fx L3582-L3596](../../../raw/current/scripts/fx.lua#L3582-L3596) |
| 资源：`anim/wortox_reviver.zip`／`swap_wortox_reviver.zip`、两张背包图标 | 世界模型、手持模型与两张 `INV_IMAGE` | 资源声明在 assets | [L1-L6](../../../raw/current/scripts/prefabs/wortox_reviver.lua#L1-L6) |

## 常见陷阱与限制

- **误区：双尾心的"施法"就是复活**：复活走的是幽灵收取（`GIVETOPLAYER` → `respawnfromghost`），与 `spellcaster` 无关；施法只做两件事——主人自用返还灵魂、他人施法传送到主人身边，两条都在结算后消耗本件。[复活](../../../raw/current/scripts/prefabs/player_common.lua#L341-L355)、[消耗](../../../raw/current/scripts/stategraphs/SGwilson.lua#L16860-L16866)
- **边界：未解锁时锁标签拦的是所有人**：锁依赖 `wortox_reviver_lock_spelluser` 反向标签，而快照内没有任何原版代码授予它；解锁只能靠生命使者三阶把 `spelltype` 清空。[L64-L68](../../../raw/current/scripts/prefabs/wortox_reviver.lua#L64-L68)、[CanCast](../../../raw/current/scripts/components/spellcaster.lua#L152-L156)
- **边界：锁的客户端投影与解锁未核验**：`wortox_reviver_lock_spellcaster` 在 pristine 预置，双端创建时都有；解锁时服务端由 `SetSpellType(nil)` 摘掉服务端副本的标签，而 `spellcaster` 组件本身没有 netvar／replica，标签如何跨端同步未在本页核验。玩家侧动作候选读的正是这个标签，解锁后客户端能否看到动作需运行时验证。[L218](../../../raw/current/scripts/prefabs/wortox_reviver.lua#L218)、[解锁](../../../raw/current/scripts/prefabs/wortox_reviver.lua#L64-L68)、[门禁](../../../raw/current/scripts/componentactions.lua#L1780-L1784)
- **边界：主人不在场时施法失败且不消耗**：`GetOwnerInst()` 为 nil 直接推失败事件返回。[L89-L95](../../../raw/current/scripts/prefabs/wortox_reviver.lua#L89-L95)
- **边界：非 PVP 不能对别的玩家放**：`canonlyuseonlocomotorspvp` 的第三项要求"双方不都是玩家"，所以给陌生玩家施法在非 PVP 下不可用。[目标判定](../../../raw/current/scripts/components/spellcaster.lua#L190-L199)
- **边界：不可堆叠**：没有 `stackable` 组件，`OnConsume` 里那段整叠分支是死代码。[L140-L146](../../../raw/current/scripts/prefabs/wortox_reviver.lua#L140-L146)
- **边界：腐坏只还 1 个灵魂**：与自用施法的按鲜度返还不同，别按 10 个估算收益。[OnPerish L153-L178](../../../raw/current/scripts/prefabs/wortox_reviver.lua#L153-L178)
- **边界：骑乘与搬重物不能捏**：`crushitemcast` 物品在骑乘或重物搬运时不出现在动作候选里。[L1796-L1799](../../../raw/current/scripts/componentactions.lua#L1796-L1799)

## 验证记录

- 源码审阅：2026-11-09，steam-buildid:26474873（与 `raw/current/BUILD.txt` 一致）。覆盖 `prefabs/wortox_reviver.lua` 全文、`skilltree_wortox.lua`（生命使者三节点与四个回调辅助）、`components/linkeditem.lua` 与 `linkeditemmanager.lua` 全文、`components/spellcaster.lua` 全文、`SGwilson.lua` 的 `crushitemcast_*` 与 `wortox_teleport_reviver*` 状态、`player_common.lua` 的幽灵收取段、`actions.lua` 的 CASTSPELL、`componentactions.lua` 的四处 `_spellcaster` 门禁、`componentutil.lua` 的传送许可、`perishable.lua` 的结算与 `Perish`、`entityscript.lua` 的 `OnBuilt`、`recipes.lua`／`recipes_filter.lua`、`constants.lua` 的 SPELLTYPES、`tuning.lua` 数值、`chinese_s.po` 名称与动作串。`wortox_soul` 只核验了 stackable 与产出关系（Wiki 待建）。
- 游戏运行：未运行；结论均为源码审阅，时长为定义值＋修正公式（推导，非实测）。
- 未覆盖：施法的实战目标选择与失败分支观感、客户端锁标签同步、"生命使者"技能树的完整节点行为。
- 命名归属：`wortox_reviver_body` 的主要页是 [[reference/prefabs/wortox_soulecho_buff_fx]]（该页已登记同一注册），本页只写宿主侧挂接，不重复其工厂与资源核对。
- 修订（2026-11-09，独立检视 P1/P2/P3 逐条）：①施法持握/收尾状态由 `crushitemcast_holding` 改为 `crushitemcast`／`crushitemcast_fail`，补传送与自用收尾点；②删除 body 工厂行，改「关联注册」一句并指向表现族页；③绑定触发点补生命使者一阶的背包遍历；④删除玩家幽灵的 `GHOSTHEART` 台词断言（该键属敌对 ghost），改写为「无专用失败台词」；⑤自用收尾补 `OnStopBody` 顺序，禁控限定在入口传送状态；⑥`displaynamefn` 改为双端、`GetBasicDisplayName` 调用；⑦`OnPerish` 按 owner 三种情形（玩家／容器／无主）改写；⑧sources 补 `components/builder.lua`、`fx.lua`。均按点名条目改，未顺手动其它段落。P3 补修（同日）：`_pst` 的无敌改为「15.2 帧关闭、onexit 兜底」；GHOSTHEART 句补敌对 ghost 的 `AbleToAcceptTest` 锚点并把 `prefabs/ghost.lua` 加入 sources。
- 外部检视 R05（2026-11-10）：摘要与计时定义改「10 天（正常计时）」，并在腐坏节新增「施法三路与计时」——`crushitemcast.onenter` 停表；动作前被打断（如电击 `electrocute`）只走非 crushcasting 的 onexit，不重启、不消耗、`paused` 随存档；`crushitemcast_fail.onenter` 重启计时但物品保留，只有成功的 selfuse／pst 两条路径才重启并 `OnConsume`。

相关：[[reference/components/linkeditem]]、[[reference/components/spellcaster]]、[[reference/components/perishable]]。

返回 [[reference/prefabs/index|原版 Prefab 索引]]。
