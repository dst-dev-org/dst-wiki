---
title: "怀表与传送门附属表现 FX 族（pocketwatch_warp_marker 等 5 注册）"
created: "2026-11-04"
updated: "2026-11-04"
type: prefab
summary: "两个宿主文件里的五个 FX 子注册：倒走表历史标记与溯源表标记，以及时间裂缝入口的前／地下叠层和到达落地表现；逐项核验工厂、端分界、全部生成点、父挂与清理。"
aliases: ["pocketwatch_warp_marker", "pocketwatch_recall_marker", "pocketwatch_portal_entrance_overlay", "pocketwatch_portal_entrance_underlay", "pocketwatch_portal_exit_fx", "怀表与传送门附属表现", "时间裂缝传送门表现", "怀表标记特效", "传送门落地特效"]
tags: [prefab, fx, network-sync, server-client]
size_class: complex
dst_build: "steam-buildid:26335394"
sources: ["prefabs/pocketwatch.lua", "prefabs/pocketwatch_portal.lua", "prefabs/pocketwatch_common.lua", "prefabs/wanda.lua", "components/positionalwarp.lua", "components/teleporter.lua", "stategraphs/SGwilson.lua", "stategraphs/SGwilsonghost.lua", "entityscript.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 怀表与传送门附属表现 FX 族（pocketwatch_warp_marker 等 5 注册）

## 快速判断

- 职责与适用：`pocketwatch.lua` 的两个 marker 与 `pocketwatch_portal.lua` 的三个 portal 表现，都是宿主文件内手写的 `Prefab(...)` 直接返回注册，不经过 `fx.lua` 通用表（对照 [[reference/mechanisms/generic-fx-factory|通用 FX 工厂]]）。marker 负责世界落点预览；overlay／underlay 构成入口的前层与地下层；exit_fx 是玩家经时间裂缝到达时的落地表现。
- 优先使用：要复刻「落地一瞬」→ `pocketwatch_portal_exit_fx`（自收尾：`portal_exit_pre`→`portal_exit_loop`→`pst` 后 1 秒移除）；要父挂式入口分层→ overlay／underlay（必须跟随 entrance）；要按玩家私有的世界标记→ 两个 marker，但两者接口不同，不能互换 [协议差异](#常见陷阱与限制)。
- 重要副作用：两个 marker 用 `Network:SetClassifiedTarget` 只复制给目标玩家，并在本地主机用 alpha 0 隐藏；overlay／underlay 用 `SetParent` 挂到 entrance（未 `AddChild`，引擎级联未核验）；exit_fx 自带 Light 与两个 netvar，没有父挂。五个注册都不结算伤害、不执行传送。
- 修改边界：物品与传送门玩法（施法、冷却、分片迁移、入口 10 秒计时）不在本页；本页只到工厂、端分界、生成点、父挂与清理。怀表主体见 [[reference/components/pocketwatch|pocketwatch]]、[[reference/components/recallmark#关键接口|recallmark]]，传送见 [[reference/components/teleporter#生命周期|teleporter]]。

名称核验：五个注册名在 `languages/chinese_s.po` 的 `STRINGS.NAMES` 中均无键，标题与别名是已核验效果描述。可作定位的只有宿主名：`pocketwatch_portal_entrance` 与 `pocketwatch_portal_exit` 同译“时间裂缝”、`pocketwatch_recall`“溯源表”、`pocketwatch_warp`“倒走表”（[L347064-L347092](../../../raw/current/scripts/languages/chinese_s.po#L347064-L347092)）。`pocketwatch_portal_exit_fx` 在工厂里 `SetPrefabNameOverride("pocketwatch_portal_exit")`（[L484](../../../raw/current/scripts/prefabs/pocketwatch_portal.lua#L484)），显示名解析会命中宿主键；这不是该 FX 注册自己的官方名（[解析顺序](../../../raw/current/scripts/entityscript.lua#L687-L691)）。

## 注册与构造

两个文件都直接 `return Prefab(...)`，没有包装工厂或同名变体；工厂均不接收参数。两个 marker 各自声明同一个 `anim/pocketwatch_warp_marker.zip`；portal 的五个子注册共用 `portal_assets` 的 `anim/pocketwatch_portal_fx.zip`（[L509-L514](../../../raw/current/scripts/prefabs/pocketwatch_portal.lua#L509-L514)）。

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `pocketwatch_warp_marker` | `warpmarkerfn`：Transform＋AnimState＋Network → bank／build `pocketwatch_warp_marker`、`OnGround`＋`LAYER_BACKGROUND`、播 `off`、α=0.6、初始 `Hide()` → `NOBLOCK`／`FX` → SetPristine → 客户端早退 → 主模拟 `persists=false`，暴露 `SetMarkerViewer`／`ShowMarker`／`HideMarker`。 | 纯表现；显示时 `math.random(4)` 选 `mark1..4` 图案并随机旋转，隐藏播对应 `_pst`→`off`；三个方法只在主模拟存在。[工厂](../../../raw/current/scripts/prefabs/pocketwatch.lua#L334-L366)、[显示／隐藏](../../../raw/current/scripts/prefabs/pocketwatch.lua#L317-L332)、[注册](../../../raw/current/scripts/prefabs/pocketwatch.lua#L427) |
| `pocketwatch_recall_marker` | `recallmarkerfn`：同 bank／build＋`OnGround`＋背景层、`idle_pre`→`idle_loop(true)`、α=0.6 → `NOBLOCK`／`FX` → SetPristine → 客户端早退 → 主模拟 `persists=false`，暴露 `ShowMarker(viewer)`／`RemoveMarker`。 | 不初始隐藏，没有 `HideMarker`；`RemoveMarker` 播 `idle_pst`→`off` 后 0.5 秒 Remove，`IsAsleep()` 时直接 Remove。与 warp marker 共用动画包，但动画名不同（`idle_*` 对 `mark*`）。[工厂](../../../raw/current/scripts/prefabs/pocketwatch.lua#L187-L217)、[收尾](../../../raw/current/scripts/prefabs/pocketwatch.lua#L177-L185)、[注册](../../../raw/current/scripts/prefabs/pocketwatch.lua#L429) |
| `pocketwatch_portal_entrance_overlay` | `portal_entrance_overlayfn`：Transform＋AnimState＋Network → 同 portal bank／build、`portal_entrance_pre`→`portal_entrance_loop(true)`、bloom、`SetInheritsSortKey(false)`、隐藏 `back`／`water_shadow` → `FX` → SetPristine → 客户端登记 `OnEntityReplicated`（把父 `highlightchildren` 指向自己）→ 主模拟 `persists=false`。 | 入口前层；无自身收音、计时或 `animover`，寿命靠父 entrance；关闭时由 entrance 侧调它播 `portal_entrance_pst`。[工厂](../../../raw/current/scripts/prefabs/pocketwatch_portal.lua#L312-L341)、[复制回调](../../../raw/current/scripts/prefabs/pocketwatch_portal.lua#L306-L310)、[注册](../../../raw/current/scripts/prefabs/pocketwatch_portal.lua#L511) |
| `pocketwatch_portal_entrance_underlay` | `portal_entrance_underlayfn`：同 bank／build 与同一对 `portal_entrance_pre`→`loop(true)`；`SetLayer(LAYER_BELOW_GROUND)`、`ANIM_SORT_ORDER_BELOW_GROUND.BOAT_LIP`、`SetFinalOffset(3)`、海洋混合参数；隐藏 `front`／`back` → `FX` → 主模拟 `persists=false`。 | 入口地下层；没有客户端逻辑；同样随父移除。三层按符号分工：entrance 藏 `front`／`water_shadow`，overlay 藏 `back`／`water_shadow`，underlay 藏 `front`／`back`，只有 underlay 不隐藏 `water_shadow`。[工厂](../../../raw/current/scripts/prefabs/pocketwatch_portal.lua#L343-L373)、[注册](../../../raw/current/scripts/prefabs/pocketwatch_portal.lua#L512) |
| `pocketwatch_portal_exit_fx` | `portal_exit_fx_fn`：Transform＋AnimState＋SoundEmitter＋Light＋Network → 同 bank／build、`portal_exit_pre`→`portal_exit_loop(false)`、bloom、`SetFinalOffset(1)`、缩放 .9、灯 radius 0／intensity .6／falloff 1.5 → `net_smallbyte` `_lightframe`＋`net_bool` `_islighton`（dirty `lightdirty`）、`lightupdaterate=2`、初始 `_islighton=true`、`SetPrefabNameOverride("pocketwatch_portal_exit")` → `ignorewalkableplatforms` → SetPristine → 客户端监听 `lightdirty`；主模拟监听 `animqueueover`→`portal_fx_close`、播落地音、10 帧后循环音、挂 `inspectable`、`OnLightDirty` 起步渐亮。 | 唯一带灯／netvar／inspectable 的一项；无父挂，位置由生成方写到绝对 y=4；没有 `persists=false`（见陷阱）。关闭：`portal_fx_close` 播 `portal_exit_pst`、停循环音、灯灭、1 秒 Remove。[工厂](../../../raw/current/scripts/prefabs/pocketwatch_portal.lua#L450-L505)、[关闭](../../../raw/current/scripts/prefabs/pocketwatch_portal.lua#L437-L448)、[注册](../../../raw/current/scripts/prefabs/pocketwatch_portal.lua#L514) |

依赖声明不等于生成：`pocketwatch.lua` 的怀表主注册 `prefabs` 表列了 `pocketwatch_warp_marker`（[L13-L24](../../../raw/current/scripts/prefabs/pocketwatch.lua#L13-L24)），但该文件没有任何生成调用；它也不列 `pocketwatch_recall_marker`。`pocketwatch_portal.lua` 的 `entrance_prefabs` 只列 overlay／underlay／`pocketwatch_portal_exit`（[L22-L27](../../../raw/current/scripts/prefabs/pocketwatch_portal.lua#L22-L27)），既不列 `exit_fx`（由玩家 SG 生成）也不含 recall marker。实际生成点见下节。

### 生成点、父挂与清理

下表是五个注册在 `raw/current/scripts` 的全量反查结果（含动态选名与 SG 侧）；未写父挂的注册就是工厂内没有 `SetParent`。

| 注册名 | 全部生成点（当前快照） | 父挂 | 清理与移除 |
|---|---|---|---|
| `pocketwatch_warp_marker` | 仅旺达 `master_postinit`：挂 `positionalwarp` 后延迟 0 秒 `SetMarker("pocketwatch_warp_marker")`（[wanda L400-L401](../../../raw/current/scripts/prefabs/wanda.lua#L400-L401)）；`SetMarker` 一次建 3 个缓存实例（[positionalwarp L63-L69](../../../raw/current/scripts/components/positionalwarp.lua#L63-L69)），`_MakeMarker` 逐个 `SpawnPrefab`（[L52-L61](../../../raw/current/scripts/components/positionalwarp.lua#L52-L61)）。全 scripts 无其他 SpawnPrefab。 | 无。位置由 `UpdateMarker` 写到历史落点（[L76-L97](../../../raw/current/scripts/components/positionalwarp.lua#L76-L97)）。 | 无自身 timer／`animover`；组件 `OnRemoveFromEntity`／`OnRemoveEntity` 意图移除全部标记（[L33-L50](../../../raw/current/scripts/components/positionalwarp.lua#L33-L50)）；但 `v:Remove()` 会触发 L60 的 `onremove` 补建同槽，循环结束后标记表被清空、补建实例可能留场，实际是否干净需运行复核；`HideMarker` 只隐藏不删除。 |
| `pocketwatch_recall_marker` | 仅 `recallmarker_create` 的 `SpawnPrefab`（[pocketwatch_common L121](../../../raw/current/scripts/prefabs/pocketwatch_common.lua#L121)）；`MakeRecallMarkable` 在标记写入、放入背包、主人复活与 0 秒初始化时调用它（[L143-L158](../../../raw/current/scripts/prefabs/pocketwatch_common.lua#L143-L158)），`pocketwatch_recall` 与 `pocketwatch_portal` 都通过 helper 安装（[pocketwatch L303](../../../raw/current/scripts/prefabs/pocketwatch.lua#L303)、[portal L120](../../../raw/current/scripts/prefabs/pocketwatch_portal.lua#L120)）。 | 无。位置取同分片标记坐标并随机旋转（[L119-L124](../../../raw/current/scripts/prefabs/pocketwatch_common.lua#L119-L124)）。 | `RemoveMarker`：醒着播 `idle_pst`→`off` 后 0.5 秒 Remove，睡眠直接 Remove（[pocketwatch L177-L185](../../../raw/current/scripts/prefabs/pocketwatch.lua#L177-L185)）；掉落、主人死亡、怀表移除触发（[common L97-L111](../../../raw/current/scripts/prefabs/pocketwatch_common.lua#L97-L111)、[L126](../../../raw/current/scripts/prefabs/pocketwatch_common.lua#L126)）。 |
| `pocketwatch_portal_entrance_overlay` | 仅 entrance 主模拟：生成后 `SetParent(inst.entity)` 并写 `inst.highlightchildren`（[portal L297-L299](../../../raw/current/scripts/prefabs/pocketwatch_portal.lua#L297-L299)）。全 scripts 无其他生成点。 | 是：父挂 `pocketwatch_portal_entrance` 实体。 | 无自身清理；仅 `SetParent`（未 `AddChild`，`EntityScript.children` 循环覆盖不到，引擎级联未核验）；关闭路径先给两层播 `portal_entrance_pst` 再移除入口（[portal L166-L188](../../../raw/current/scripts/prefabs/pocketwatch_portal.lua#L166-L188)）。 |
| `pocketwatch_portal_entrance_underlay` | 同一主模拟段生成并 `SetParent`（[portal L300-L301](../../../raw/current/scripts/prefabs/pocketwatch_portal.lua#L300-L301)）。全 scripts 无其他生成点。 | 是：父挂 entrance。 | 同上；自身只有 `persists=false`，没有 timer／`animover`。 |
| `pocketwatch_portal_exit_fx` | 仅三处，均由玩家到达态生成并写 `(x, 4, z)`：SGwilson `pocketwatch_portal_land`（[L20813-L20830](../../../raw/current/scripts/stategraphs/SGwilson.lua#L20813-L20830)）、SGwilson `pocketwatch_portal_fallout`（[L20878-L20891](../../../raw/current/scripts/stategraphs/SGwilson.lua#L20878-L20891)）、SGwilsonghost `pocketwatch_portal_land`（[L554-L563](../../../raw/current/scripts/stategraphs/SGwilsonghost.lua#L554-L563)）。 | 无。 | `animqueueover`→`portal_fx_close`：播 `portal_exit_pst`、`KillSound("loop")`、灯灭、1 秒 Remove（[L437-L448](../../../raw/current/scripts/prefabs/pocketwatch_portal.lua#L437-L448)、[L495](../../../raw/current/scripts/prefabs/pocketwatch_portal.lua#L495)）；没有 `persists=false`。 |

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`positionalwarp`（warp marker） | `SetMarker` 建 3 个缓存；`UpdateMarker` 按历史点移动并 `ShowMarker`／`HideMarker`；`EnableMarker` 只控显示。外部 Remove 会触发补建。 | 主模拟；旺达专属，显示与否由倒走表在持有者物品栏中的事件驱动。 | [[reference/components/positionalwarp#关键接口|positionalwarp]]；[宿主](../../../raw/current/scripts/prefabs/wanda.lua#L324-L329)、[显示链](../../../raw/current/scripts/prefabs/pocketwatch.lua#L386-L401) |
| 组件：`recallmark`（recall marker） | 怀表侧 `MakeRecallMarkable` 安装组件与 `onMarkPosition` 回调；标记在写入、背包、复活等事件后重建，只给非幽灵主人。 | 主模拟；`pocketwatch_recall` 与 `pocketwatch_portal` 都安装。 | [[reference/components/recallmark#关键接口|recallmark]]；[helper](../../../raw/current/scripts/prefabs/pocketwatch_common.lua#L143-L158) |
| 宿主：入口 `pocketwatch_portal_entrance` | 主模拟生成 overlay／underlay 并父挂；`CloseEntrance` 给两层播 `portal_entrance_pst`；`highlightchildren` 供本地主机高亮。 | 主模拟生成、双端显示；入口 10 秒 timer 关闭。 | [生成](../../../raw/current/scripts/prefabs/pocketwatch_portal.lua#L297-L301)、[10 秒计时](../../../raw/current/scripts/prefabs/pocketwatch_portal.lua#L283-L293) |
| 宿主：`SGwilson`／`SGwilsonghost` 到达态 | 出口 teleporter 的 `overrideteleportarrivestate` 指向 `pocketwatch_portal_land`，到达时在该玩家位置生成 exit_fx；非施法者走 fallout 分支，幽灵有独立 land 态。 | 主模拟；三个生成点都把 y 写成 4。 | [[reference/components/teleporter#生命周期|teleporter]]；[到达覆写](../../../raw/current/scripts/components/teleporter.lua#L311-L323)、[配置](../../../raw/current/scripts/prefabs/pocketwatch_portal.lua#L425-L429) |
| 网络：ClassifiedTarget | 两个 marker 生成时／显示时指定 viewer；`viewer ~= ThePlayer` 时本机把 alpha 覆为 0。 | 主模拟设置；复制只发给指定玩家，其他客户端不持有该实体。 | [warp 设置](../../../raw/current/scripts/prefabs/pocketwatch.lua#L309-L315)、[recall 设置](../../../raw/current/scripts/prefabs/pocketwatch.lua#L169-L175) |
| 组件：`Light`＋netvar | 入口与出口共用渐亮／渐灭：radius=`2.5×frame/14`，入口亮起 rate 1、出口 2，熄灭每帧 3；`_lightframe` 用 `set_local`，客户端靠 `lightdirty` 启动任务。 | 双端渲染；exit_fx 初始 `_islighton=true`。 | [更新](../../../raw/current/scripts/prefabs/pocketwatch_portal.lua#L128-L155)、[触发](../../../raw/current/scripts/prefabs/pocketwatch_portal.lua#L190-L203) |
| 组件：`inspectable`＋名称覆盖 | 仅 exit_fx 主模拟挂 `inspectable`；`nameoverride` 指向 `pocketwatch_portal_exit`，名称/描述按宿主解析。 | 主模拟。 | [挂载](../../../raw/current/scripts/prefabs/pocketwatch_portal.lua#L500)、[解析](../../../raw/current/scripts/entityscript.lua#L687-L691) |
| 关联 Prefab：`pocketwatch_portal_exit`（传送实体，Wiki 待建） | entrance 的 `SpawnExit` 生成非联网 teleporter 并写到达态；它不是本页的 exit_fx。 | 主模拟；同分片才配普通出口，跨分片走迁移。 | [SpawnExit](../../../raw/current/scripts/prefabs/pocketwatch_portal.lua#L205-L220)、[到达态](../../../raw/current/scripts/prefabs/pocketwatch_portal.lua#L428) |
| 对照：父挂式传送门投影 | `impact` 页的 `lunarrift_portal_shadow`、`multiplayer_portal_moonrock_fx` 也由 portal 生成并父挂；本页 overlay／underlay 同属父挂，exit_fx 则是无父挂的玩家落地表现。 | — | [[reference/prefabs/impact#生成点父挂与清理|impact 的父挂投影]] |
| 资源 | 两个 marker 共用 `anim/pocketwatch_warp_marker.zip`；portal 子注册共用 `anim/pocketwatch_portal_fx.zip`。exit_fx 播放 `wanda1/wanda/portal_exit_pre`、`wanda2/characters/wanda/watch/portal_LP`、`wanda1/wanda/portal_exit_pst`。 | 声明与播放点已核验；快照无 `anim/` 目录，包内容、符号与帧长未解包。 | [marker assets](../../../raw/current/scripts/prefabs/pocketwatch.lua#L427-L429)、[portal assets](../../../raw/current/scripts/prefabs/pocketwatch_portal.lua#L17-L20)、[音效](../../../raw/current/scripts/prefabs/pocketwatch_portal.lua#L441-L442) |

## 典型接入

### 在自定义传送落点复用时间裂缝出口表现

- 前提与选择：目标端在主模拟、只需一次落地表现；选 `pocketwatch_portal_exit_fx` 而不是把 `pocketwatch_portal_exit` teleporter 搬来，因为前者自带灯光、音效与自删，后者只负责传送配对。
- 生效与撤销：主模拟 `SpawnPrefab` 后把位置设为 `(x, 4, z)`；y=4 是原版的绝对高度，不是「玩家 y+4」。生成后不父挂，动画队列结束自动 `portal_exit_pst`＋1 秒 Remove；不要手动长期保留。
- 原版依据：[SGwilson 的 land／fallout](../../../raw/current/scripts/stategraphs/SGwilson.lua#L20813-L20891) 与 [SGwilsonghost 的 land](../../../raw/current/scripts/stategraphs/SGwilsonghost.lua#L554-L563) 都是「到达 → 在玩家位置生成」；[teleporter 的到达覆写](../../../raw/current/scripts/components/teleporter.lua#L311-L323) 是这条链的入口。

```lua
-- 服务端：自定义传送到达点补一次时间裂缝出口表现
local function PlayPocketwatchExitFx(player)
    if not TheWorld.ismastersim or player == nil or not player:IsValid() then
        return
    end

    local x, _, z = player.Transform:GetWorldPosition()
    local fx = SpawnPrefab("pocketwatch_portal_exit_fx")
    if fx ~= nil then
        fx.Transform:SetPosition(x, 4, z)
    end
end
```

- 调整与验证：替换点是注册名与坐标；`exit_fx` 没有公开的 Setup／染色接口，动画、音效、灯光参数都在工厂内。示例只做语法检查，未在游戏内执行；中间变量 `_` 只是占位，不读取 y。

## 常见陷阱与限制

- **误区：两个 marker 可以互换给 `positionalwarp:SetMarker`**：warp marker 有 `SetMarkerViewer`／`ShowMarker()`／`HideMarker`，recall marker 只有 `ShowMarker(viewer)`／`RemoveMarker`；`UpdateMarker` 会调无参 `ShowMarker`／`HideMarker`，换用 recall marker 会因缺少 `HideMarker` 报错。[协议](../../../raw/current/scripts/components/positionalwarp.lua#L57-L58)、[调用](../../../raw/current/scripts/components/positionalwarp.lua#L81-L93)、[两组接口](../../../raw/current/scripts/prefabs/pocketwatch.lua#L317-L332)
- **边界：`SetClassifiedTarget` 不是删除**：两个 marker 仍以联网实体存在，只是复制目标受限；本机主机再用 `OverrideMultColour(...,0)` 做视觉隐藏，客户端实际访问与删除语义不同。[warp](../../../raw/current/scripts/prefabs/pocketwatch.lua#L309-L315)、[recall](../../../raw/current/scripts/prefabs/pocketwatch.lua#L169-L175)
- **边界：overlay／underlay 没有自己的回收**：它们没有 timer／`animover`，生命周期挂在 entrance 上；但两层只 `SetParent`（未 `AddChild`），`EntityScript:Remove` 的 children 循环覆盖不到，引擎级联未核验；关闭路径会先播 `portal_entrance_pst`，非关闭路径移除父实体时是否补播退场未运行核验。[递归删除](../../../raw/current/scripts/entityscript.lua#L1779-L1784)、[关闭路径](../../../raw/current/scripts/prefabs/pocketwatch_portal.lua#L166-L188)
- **边界：exit_fx 不是 portal 生成的**：`pocketwatch_portal_entrance` 只生成 `pocketwatch_portal_exit` teleporter；`exit_fx` 由玩家 SG 的 `pocketwatch_portal_land`／`pocketwatch_portal_fallout` 生成，触点是 teleporter 的 `overrideteleportarrivestate`。[SpawnExit](../../../raw/current/scripts/prefabs/pocketwatch_portal.lua#L205-L220)、[到达态](../../../raw/current/scripts/prefabs/pocketwatch_portal.lua#L425-L429)
- **边界：exit_fx 是五者中唯一没有 `persists=false` 的**：它靠 `animqueueover` 后 1 秒 Remove，`Remove` 才会把 `persists` 置 false；播放期间存档或休眠后的恢复未核验。[默认持久化](../../../raw/current/scripts/entityscript.lua#L222)、[移除置 false](../../../raw/current/scripts/entityscript.lua#L1793-L1797)、[工厂未设置](../../../raw/current/scripts/prefabs/pocketwatch_portal.lua#L450-L505)
- **边界：exit_fx 的灯在客户端靠 `lightdirty` 启动**：工厂初始 `_islighton=true`，但渐亮任务只在 dirty 回调里建立；专用服务器客户端能否在首次复制时收到该 dirty，未运行核验。[netvar](../../../raw/current/scripts/prefabs/pocketwatch_portal.lua#L477-L482)、[客户端监听](../../../raw/current/scripts/prefabs/pocketwatch_portal.lua#L486-L492)
- **误区：用 `prefabs` 依赖表反推生成点**：warp marker 被怀表依赖表声明却只由旺达的 positionalwarp 生成；recall marker 与 exit_fx 都不在对应文件的依赖表里，仍分别由怀表 helper 与玩家 SG 生成。[依赖声明](../../../raw/current/scripts/prefabs/pocketwatch.lua#L13-L24)、[portal 依赖](../../../raw/current/scripts/prefabs/pocketwatch_portal.lua#L22-L27)
- **名称边界**：五个注册名都没有 `STRINGS.NAMES` 键；`exit_fx` 的显示名经 `SetPrefabNameOverride` 借宿主 `pocketwatch_portal_exit` 的“时间裂缝”，不能把它当成本注册的官方名。[po](../../../raw/current/scripts/languages/chinese_s.po#L347064-L347092)

## 验证记录

- 源码审阅：2026-11-04，Steam buildid 26335394（`raw/current/BUILD.txt`）。逐项核验两个宿主文件的五个注册：返回位置、工厂与端分界、全部生成点（在 `raw/current/scripts` 全量反查五个字面名及 `SetMarker`、`SpawnPrefab` 调用）、父挂、清理与依赖声明；另核验 `wanda.lua` 的 marker 宿主、`positionalwarp.lua` 的标记协议与移除、`pocketwatch_common.lua` 的标记创建／移除、`teleporter.lua` 的到达覆写、`SGwilson`／`SGwilsonghost` 的三个生成点、`entityscript.lua` 的默认持久化／子实体删除／名称解析，以及 `languages/chinese_s.po` 的宿主名键。未覆盖：动画包内容与帧长、客户端实际观感与音频、专服灯光首帧、存档／读档恢复、物品与传送门玩法。
- 游戏运行：未运行；validation 保持 `source-reviewed`。
- 结构检查：`validate_page.py` 通过；`validate_source_links.py --strict` 通过；Lua 示例 `parser_probe --syntax-only` 通过（结果见交付报告）。
- 分档说明：本页覆盖两个 marker 的两种不同可见性协议、portal 三层父挂组合、玩家 SG 的落地链与跨文件生成／清理，需要整体对照保留，故用 `size_class: complex`；未按源码文件总量扩额。

相关页面：[[reference/prefabs/impact|命中与落点伤害表现及传送门投影 FX 族]]（父挂式传送门投影对照）。

返回 [[reference/prefabs/fx-world-index|特效 Prefab 索引·环境与世界]]。
