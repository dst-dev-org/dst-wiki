---
title: "活扫帚轨迹、沃比烘干架与训练哨增益 FX（witchbroom_fx 等 3 注册）"
created: "2026-11-05"
updated: "2026-11-09"
type: prefab
summary: "角色技能与装备外围的 3 个 FX 注册：活扫帚装备期的隐形联网载体（非专服每 5 帧从本地轨迹池投放三变体拖尾）、大沃比烘干架的联网换装层（net_hash 三槽驱动本地 FX 子体、与冲刺隐身层同步染色）与训练哨教练 buff 的联网跟随体（伤害加成在 debuff 宿主）。含幽灵注册 woby_rack 的复核结论。"
aliases: ["witchbroom_fx", "woby_rack_swap_fx", "wolfgang_coach_buff_fx", "附魔女巫扫帚", "活扫帚轨迹", "扫帚拖尾", "沃比烘干架", "烘干架换装层", "训练哨增益", "教练buff", "wolfgang 教练"]
tags: [prefab, fx, server-client]
size_class: standard
dst_build: "steam-buildid:26474873"
sources: ["prefabs/witchbroom.lua", "prefabs/livingtree_root.lua", "prefabs/madscience_lab.lua", "prefabs/woby_rack.lua", "prefabs/wobybig.lua", "prefabs/wobysmall.lua", "components/wobyrack.lua", "prefabs/wolfgang.lua", "components/coach.lua", "prefabs/wolfgang_whistle.lua", "prefabs/skilltree_wolfgang.lua", "components/mightiness.lua", "stategraphs/SGwilson.lua", "constants.lua", "recipes.lua", "actions.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 活扫帚轨迹、沃比烘干架与训练哨增益 FX（witchbroom_fx 等 3 注册）

## 快速判断

- 职责与适用：**3 个互不相干的 FX 注册**。①`witchbroom_fx`（[注册](../../../raw/current/scripts/prefabs/witchbroom.lua#L305-L306)）——活扫帚"附魔女巫扫帚"装备期的隐形联网载体，可见拖尾是它按 5 帧节拍从**本地轨迹池**取出的独立小实体；②`woby_rack_swap_fx`（[注册](../../../raw/current/scripts/prefabs/woby_rack.lua#L282-L283)）——大沃比烘干架的联网换装层（一次生成两实例分挂鞍座两侧），3 个烘干槽位走 net_hash，另有与沃比冲刺隐身层的染色同步；③`wolfgang_coach_buff_fx`（[注册](../../../raw/current/scripts/prefabs/wolfgang.lua#L789-L790)）——训练哨教练 buff 的联网跟随体，父挂受buff目标循环播动画。
- 重要副作用：载体／火层／跟随体自身均零玩法——活扫帚的加速与磨损在扫帚物品上，烘干数据流在 `wobyrack` 组件，教练的伤害 ×2 在 debuff 宿主的 `combat.externaldamagemultipliers`；3 个注册都 `persists=false`，不进存档。
- 优先使用：查"骑扫帚跑的星尘拖尾"走轨迹池链；查"沃比身上的晾肉架怎么换贴图/跟着隐身"走烘干架链；查"训练哨吹了给队友什么"走教练 buff 链。
- 修改边界：只覆盖 3 注册的工厂、生成点、端模型与清理；扫帚本体 `witchbroom` 的耐久、清扫与附魔入口归 [[reference/prefabs/tools-misc#witchbroom女巫扫帚]]，本页不展开；沃比转形与对齐皮肤、沃尔冈三形态数值不展开。**幽灵注册 `woby_rack` 复核结论见[幽灵注册](#幽灵注册-wobyrack-的复核)小节。**
- 名称：链上宿主官方名 `WITCHBROOM`＝"女巫扫帚"、`WITCHBROOM_LIVING`＝"附魔女巫扫帚"、`WOBYBIG`／`WOBYSMALL`＝"沃比"、`WOLFGANG_WHISTLE`＝"训练哨"（锚点见[验证记录](#验证记录)）；3 个覆盖注册名在简中 `chinese_s.po` 均无 `STRINGS.NAMES.*` 键，标题中文为已核验效果描述非官方译名。

## 注册与构造

| 注册名 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `witchbroom_fx` | `fxfn`：Transform／Network（**无 AnimState——载体本身不可见**）；标签 `FX` → SetPristine；非专服建 `trail_pool`＋`DoPeriodicTask(FRAMES*5, fx_DoTrail)`；主模拟 `persists=false` | FRAMES=1/30（[constants.lua:16](../../../raw/current/scripts/constants.lua#L16)），5 帧≈0.167 秒≈**每秒最多 6 次**；可见拖尾 `CreateTrailFx` 是本地非网络实体（Transform／AnimState，`SetCanSleep(false)`、`persists=false`），bank＝build `witchbroom_trail`，`RestartFx` 随机 `witchbroom_trail1..3` 并 50% 概率 X 镜像（[载体工厂](../../../raw/current/scripts/prefabs/witchbroom.lua#L281-L303)、[轨迹工厂](../../../raw/current/scripts/prefabs/witchbroom.lua#L223-L247)） |
| `woby_rack_swap_fx` | `swapfxfn`：Transform／AnimState／Follower／Network；标签 `decor`＋`NOCLICK`（**非 FX 标签**）；SetSixFaced；bank＝build `woby_rack`；生成方区分两实例：默认播 `swap_1`，第二实例强制 `swap_2`；highlightchild＋colouraddersync；3 槽 `net_hash`（build/name，dirty 事件 `slotdirty[i]`）→ 客户端监听；非专服为每槽建本地 `CreateSlotFx` 子体（Transform／AnimState／Follower＋highlightchild）父挂并 `FollowSymbol(inst.GUID, "swap_slot<i>")`；主模拟 `persists=false`，暴露 `ShowRackItem`／`HideRackItem` | 联网层＋本地显示层两层结构；槽位符号 `swap_dried`／绳结 `rope` 按 net_hash 切换（[工厂](../../../raw/current/scripts/prefabs/woby_rack.lua#L185-L249)、[槽位净变量](../../../raw/current/scripts/prefabs/woby_rack.lua#L205-L217)） |
| `wolfgang_coach_buff_fx` | `bufffn_fx`：Transform／AnimState／Network／Follower；标签 `FX`；bank＝build `coached_fx`，`idle` 循环；SetPristine 客户端早退 → 主模拟 `persists=false` | 纯表现层；Follower 组件**声明但全文件无 `FollowSymbol` 调用**——跟随靠 `entity:SetParent` 引擎父挂；无自删，由 debuff 宿主 Remove（[工厂](../../../raw/current/scripts/prefabs/wolfgang.lua#L763-L785)、[父挂](../../../raw/current/scripts/prefabs/wolfgang.lua#L702-L718)） |

## 关联机制导航

### witchbroom_fx 活扫帚轨迹链

| 环节 | 行为与条件 | 证据 |
|---|---|---|
| 载体生成 | 扫帚物品 `OnEquip` 时若已转活（`inst.living`）生成载体父挂持有者；装备状态下被"完全正常的树根"点化（`MakeLiving`）时补生成 | [OnEquip](../../../raw/current/scripts/prefabs/witchbroom.lua#L13-L34)、[MakeLiving](../../../raw/current/scripts/prefabs/witchbroom.lua#L94-L114) |
| 转活入口 | `livingtree_root`（完全正常的树根，由"树木园实验"经疯狂科学实验室制造）对非 `flyingbroom` 扫帚使用 → `MakeLiving`：换 build、改图标、walkspeedmult 1.1、监听持有者 `locomote` 移动磨损 | [点化交互](../../../raw/current/scripts/prefabs/livingtree_root.lua#L78-L85)、[SetLiving](../../../raw/current/scripts/prefabs/witchbroom.lua#L74-L92)、[制造映射](../../../raw/current/scripts/prefabs/madscience_lab.lua#L49-L52) |
| 拖尾投放 | `fx_DoTrail` 每 5 帧：读父级（持有者）`sg:HasStateTag("moving")`（无 sg 退化为 `HasTag("moving")`），移动中从池取轨迹放到载体当前位置 | [fx_DoTrail](../../../raw/current/scripts/prefabs/witchbroom.lua#L261-L279) |
| 池化回收 | `GetPooledTrailFx`：池空则 `CreateTrailFx`（记 `fx.owner = 载体`）；播完 `animover` → 有主且池在则 `RemoveFromScene` 回池，否则自删 | [GetPooledTrailFx](../../../raw/current/scripts/prefabs/witchbroom.lua#L249-L259)、[trail_OnAnimOver](../../../raw/current/scripts/prefabs/witchbroom.lua#L214-L221) |
| 清理 | 卸装 `OnUnequip`／收进携带模型 `OnEquipToModel` 显式 Remove 载体；`OnEquip` 先删旧再建；载体移除后，仍在播的轨迹会在本次 `animover` 检查 owner 失败后自删；已回池项没有显式清池路径，不能依赖再次 `animover` 保证回收，最终结果未核验 | [OnUnequip](../../../raw/current/scripts/prefabs/witchbroom.lua#L36-L55)、[OnEquipToModel](../../../raw/current/scripts/prefabs/witchbroom.lua#L57-L62) |

### woby_rack_swap_fx 烘干架两层

| 环节 | 行为与条件 | 证据 |
|---|---|---|
| 生成 | 大沃比 `EnableRack(true)`：挂 `wobyrack` 组件（dryingrack 子类，另生成 `woby_rack_container` CLASSIFIED 子体）后一次生成**两个**联网层（rackfx1 播 `swap_1`、rackfx2 强制 `swap_2`）；门禁是沃尔特技能 `walter_camp_wobyholder` | [EnableRack](../../../raw/current/scripts/prefabs/wobybig.lua#L227-L255)、[技能门禁](../../../raw/current/scripts/prefabs/wobybig.lua#L681-L684)、[wobyrack 组件](../../../raw/current/scripts/components/wobyrack.lua#L3-L8) |
| 挂接 | `SetRackFxOwner` 把两实例父挂骑手（无骑手挂沃比本体）、`FollowSymbol(owner, "swap_saddle")`（两实例末段参数不同）、highlightchild/colouradder 换绑；骑手变更时重调 | [SetRackFxOwner](../../../raw/current/scripts/prefabs/wobybig.lua#L190-L206)、[OnRiderChanged](../../../raw/current/scripts/prefabs/wobybig.lua#L464-L466) |
| 数据流 | `wobyrack` 的 Show/HideItemFn → 大沃比 `ShowRackItem`/`HideRackItem` 同时写两个联网层的 net_hash → 客户端 `slotdirty[i]` → `_OnSlotDirty` 切 `swap_dried`/`rope_empty` 符号；沃比静止时播 `bounce_change1..3`＋`idle_sway` | [ShowRackItem](../../../raw/current/scripts/prefabs/wobybig.lua#L209-L223)、[_OnSlotDirty](../../../raw/current/scripts/prefabs/woby_rack.lua#L139-L161) |
| 隐身同步 | 非专服的**网络 rack 实体**持 `OnPostUpdateFading`／`SetFadeColour`：每帧查宿主 `woby_dash_fade`／`woby_align_fade` 标签，把宿主 AnimState 的 MultColour 写到自身与 3 个本地 slot 子体；OnEntitySleep/Wake 增删 updatelooper 的 PostUpdateFn，不是各本地子体分别执行整套宿主轮询 | [woby_rack L52-L60](../../../raw/current/scripts/prefabs/woby_rack.lua#L52-L60)、[OnOwnerChanged](../../../raw/current/scripts/prefabs/woby_rack.lua#L96-L131) |
| 清理 | `EnableRack(false)` 移除组件并 `Remove` 两实例（[L258-L274](../../../raw/current/scripts/prefabs/wobybig.lua#L258-L274)）；读档按 `data.wobyrack` 重开（[OnPreLoad](../../../raw/current/scripts/prefabs/wobybig.lua#L276-L278)）；本地子体只 `SetParent` 挂网络层、未登记 Lua `children`，网络层删除时是否带走未核验 | — |
| 小沃比对照 | `wobysmall` 的 EnableRack 只做本体符号 override（`swap_rack`），**不生成 swap_fx**——烘干架换装层是大沃比专属 | [wobysmall EnableRack](../../../raw/current/scripts/prefabs/wobysmall.lua#L167-L205)、[技能门禁](../../../raw/current/scripts/prefabs/wobysmall.lua#L296-L300) |

#### 幽灵注册 `woby_rack` 的复核

`prefablist.lua:1468` 有 `"woby_rack"` 条目，但全域反查**无任何 `Prefab("woby_rack", ...)` 注册、无 `SpawnPrefab("woby_rack")`**；`prefabs/woby_rack.lua` 实际只注册 `woby_rack_swap_fx` 与 `woby_rack_container`。游戏内出现的"架子动画"由大沃比/小沃比对本体 `AddOverrideBuild("woby_rack")` 借用动画库绘制（[wobybig.lua](../../../raw/current/scripts/prefabs/wobybig.lua#L115-L126)、[wobysmall.lua](../../../raw/current/scripts/prefabs/wobysmall.lua#L174-L180)）。台账批 9 "本体仅 prefablist 有名、blocked"的判定复核属实——`woby_rack` 是无实现的幽灵条目，不是可生成的实体。

### wolfgang_coach_buff_fx 教练 buff 链

| 环节 | 行为与条件 | 证据 |
|---|---|---|
| 技能门禁 | 技能树节点 `wolfgang_normal_coach` 激活 → 挂 `wolfgang_coach` 标签；训练哨配方要求同技能；`PLAY` 动作双标签门禁（`wolfgang_coach`＋`mightiness_normal`） | [技能节点](../../../raw/current/scripts/prefabs/skilltree_wolfgang.lua#L77-L86)、[哨子配方](../../../raw/current/scripts/recipes.lua#L264)、[动作门禁](../../../raw/current/scripts/actions.lua#L2846-L2855) |
| 开关教练 | 吹哨 `OnPlayed`：需 coach＋mightiness 组件与标签；**仅沃尔冈 normal 形态**可开/关 `coaching`，wimpy/mighty 播拒绝台词；`mightiness` 任何形态切换都 `coach:Disable()` | [OnPlayed](../../../raw/current/scripts/prefabs/wolfgang_whistle.lua#L13-L30)、[形态禁用](../../../raw/current/scripts/components/mightiness.lua#L313-L315) |
| 激励结算 | coach 组件 `inspire`：首轮 7 秒后，之后 10＋0–30 秒一轮；25 格内玩家（理智<75% 各 +5 理智）及其跟随者＋教练自身跟随者进 `fightbuff`，逐个延迟 0.2＋0.1·i 秒 `cheer` 并 `AddDebuff("wolfgang_coach_buff")` | [inspire](../../../raw/current/scripts/components/coach.lua#L21-L75)、[调度](../../../raw/current/scripts/components/coach.lua#L85-L90)、[调值](../../../raw/current/scripts/tuning.lua#L7883-L7885) |
| debuff 宿主 | `wolfgang_coach_buff`（master-only、CLASSIFIED、`persists=false`、`keepondespawn=true`）：`OnAttached` 给目标 `combat.externaldamagemultipliers` ×2（`TUNING.WOLFGANG_COACH_BUFF=2`）并生成 FX 父挂；10 秒 `bufftask` 到期自停；续吹走 `OnExtendedBuff` 重置计时；`OnDetached` 移除乘数并 Remove FX | [bufffn](../../../raw/current/scripts/prefabs/wolfgang.lua#L738-L761)、[OnAttached](../../../raw/current/scripts/prefabs/wolfgang.lua#L702-L718)、[OnDetached](../../../raw/current/scripts/prefabs/wolfgang.lua#L720-L728) |

### 清理路径

| 注册 | 清理方式 | 边界 |
|---|---|---|
| `witchbroom_fx` | 卸装/收模型显式 Remove 载体；宿主被直接移除时原生父挂结果未核验；在播轨迹在 animover 检查 owner 失败后自删 | `trail_pool` 是数组；已回池项没有显式清池路径，不能依赖再次 `animover` 保证回收，最终结果未核验 |
| `woby_rack_swap_fx` | `EnableRack(false)` 显式 Remove 两实例；转形链把烘干快照迁给新沃比后同样走禁用路径 | 本地槽层只 `SetParent` 挂网络层、未登记 Lua `children`，网络层删除时是否带走未核验；每个非专服进程是 2 个联网层＋6 个本地槽层（每层 3 槽），均 `persists=false` |
| `wolfgang_coach_buff_fx` | debuff `OnDetached` 显式 Remove（10 秒到期／目标死亡 → `debuff:Stop`） | 目标被直接移除时 FX 只经 `SetParent` 父挂、原生结果未核验；`keepondespawn` 下线重连由 debuff 系统重建（重建细节归 debuff 组件，本页未展开） |

## 典型接入

### 父挂隐形载体＋本地轨迹池

- 前提与选择：需要"跟随宿主的移动拖尾"且想省网络开销时，模式是：联网壳只做挂点与节拍，可见粒子全部本地化＋池化（RemoveFromScene 回池而非反复 SpawnPrefab）。
- 生效与撤销：载体由装备链生成/移除；轨迹池与周期任务只在非专服建；池元素播完自动回池，卸载只删网络载体、不逐个清池——在播轨迹会在 animover 自删；回池项没有显式清池路径，不能依赖再次 animover 保证回收，结局未核验。
- 原版依据：[载体工厂](../../../raw/current/scripts/prefabs/witchbroom.lua#L281-L303)、[池化](../../../raw/current/scripts/prefabs/witchbroom.lua#L249-L259)。

```lua
-- 协议示意：CreateTrailFx 需自带 bank/build、RestartFx，并监听 animover→TrailOnAnimOver
local function TrailOnAnimOver(fx)
    if fx.owner and fx.owner:IsValid() and fx.owner.trail_pool then
        fx:RemoveFromScene()
        table.insert(fx.owner.trail_pool, fx)
    else
        fx:Remove()
    end
end

local function GetPooledTrailFx(inst)
    local fx = table.remove(inst.trail_pool)
    if fx ~= nil then
        fx:ReturnToScene()
        fx:RestartFx()
    else
        fx = CreateTrailFx()   -- 需补 bank/build、RestartFx、animover 监听
        fx.owner = inst
    end
    return fx
end

local function fxfn()
    local inst = CreateEntity()
    inst.entity:AddTransform()
    inst.entity:AddNetwork()
    inst:AddTag("FX")
    if not TheNet:IsDedicated() then
        inst.trail_pool = {}
        inst.trail_task = inst:DoPeriodicTask(FRAMES * 5, fx_DoTrail) -- fx_DoTrail 需按 moving 标签取池
    end
    inst.entity:SetPristine()
    if not TheWorld.ismastersim then
        return inst
    end
    inst.persists = false
    return inst
end
```

- 调整与验证：替换点是节拍（FRAMES*n）与移动判定；复用前必须补齐 `CreateTrailFx`（bank/build、`RestartFx`、`animover` 回池监听）与 `fx_DoTrail` 两个辅助。示例为协议示意，不是可直接接入的完整骨架，未在游戏内执行。

## 常见陷阱与限制

- **边界：轨迹池是纯客户端／非专服产物。** 专用服务器上载体只剩网络壳，无池、无任务、无拖尾；两端各自投放各自的池，数量不互相同步。[fxfn](../../../raw/current/scripts/prefabs/witchbroom.lua#L281-L303)
- **边界：节拍是 5 帧≈0.167 秒（约每秒 6 次），不是 10Hz。** 台账批 9 所记"10Hz 轨迹池"与源码不符：`DoPeriodicTask(FRAMES*5)`、`FRAMES = 1/30`，且移动中才投放，实际频率还受宿主 `moving` 状态门控。已按源码修正入页，建议台账勘误。[fx_DoTrail](../../../raw/current/scripts/prefabs/witchbroom.lua#L261-L279)
- **边界：拖尾判定读 `moving` 状态标签而非速度。** 只认 `sg:HasStateTag("moving")`（无 sg 时退化为实体 `moving` 标签），没有 `IsRiding` 排除：[SGwilson 的 run_start／run](../../../raw/current/scripts/stategraphs/SGwilson.lua#L11296-L11443) 都带 `moving`，骑乘坐骑仍可能投放；反过来原地播放带 moving 的动画会空投。不能把“骑乘”直接等同于不投放。[fx_DoTrail](../../../raw/current/scripts/prefabs/witchbroom.lua#L267-L272)
- **边界：`woby_rack_swap_fx` 是 `decor`＋`NOCLICK` 而非 `FX` 标签。** 参与 highlightchild 高亮与 colouraddersync 染色链，别按普通 FX 的点击排除语义处理；同文件 `woby_rack_container` 是 CLASSIFIED 数据壳（仅容器组件），无动画。[工厂](../../../raw/current/scripts/prefabs/woby_rack.lua#L185-L249)
- **边界：烘干架换装层只在大沃比。** 小沃比（`wobysmall`）的架位是本体符号 override，无 swap_fx 生成；两代沃比共享 `woby_rack` 动画库但装配路径不同，转形时烘干数据走快照迁移。[wobysmall EnableRack](../../../raw/current/scripts/prefabs/wobysmall.lua#L167-L205)
- **边界：教练 buff 的数值与表现分离。** 伤害 ×2 写在 debuff 宿主的 `externaldamagemultipliers`，FX 只有动画；关掉教练（`coach:Disable`）不回收已有 buff——已发出的 buff 走完各自 10 秒，只是不再发新的。[OnDetached](../../../raw/current/scripts/prefabs/wolfgang.lua#L720-L728)、[Disable](../../../raw/current/scripts/components/coach.lua#L11-L17)
- **边界：`wolfgang_coach_buff_fx` 的 Follower 是未接线的声明。** 全文件无 `FollowSymbol` 调用，跟随完全靠 `entity:SetParent`；想让它停在符号上需自己补 FollowSymbol（对照 [[reference/prefabs/wortox_soulecho_buff_fx|沃拓克斯灵魂增益与装备跟随特效族]] 的 overloading 模式）。[工厂](../../../raw/current/scripts/prefabs/wolfgang.lua#L763-L785)

## 验证记录

- 源码审阅：2026-11-05，steam-buildid:26335394（实读 `raw/current/BUILD.txt`）。完整读取 `prefabs/witchbroom.lua`、`prefabs/woby_rack.lua`、`components/wobyrack.lua`、`components/coach.lua`、`prefabs/wolfgang_whistle.lua`；按需读取 `prefabs/wobybig.lua`／`prefabs/wobysmall.lua` 的 EnableRack／SetRackFxOwner／OnRiderChanged／技能门禁段、`prefabs/wolfgang.lua` 的 coach buff 全函数族与注册块、`components/mightiness.lua` Disable 段、`prefabs/skilltree_wolfgang.lua` 节点、`prefabs/livingtree_root.lua` 点化段、`prefabs/madscience_lab.lua` MANUFACTURED_PRODUCTS、`actions.lua` PLAY.strfn、`recipes.lua` 哨子配方、`constants.lua` FRAMES、`tuning.lua`（WITCHBROOM_*、COACH_*）。对 3 个覆盖注册名全域反查：`witchbroom_fx` 命中 4 处（依赖 [witchbroom.lua:10](../../../raw/current/scripts/prefabs/witchbroom.lua#L10)、生成 [L23](../../../raw/current/scripts/prefabs/witchbroom.lua#L23)／[L103](../../../raw/current/scripts/prefabs/witchbroom.lua#L103)、注册 [L306](../../../raw/current/scripts/prefabs/witchbroom.lua#L306)）——生成点 2 处（装备／点化补挂）；`woby_rack_swap_fx` 命中注册（[woby_rack.lua:282](../../../raw/current/scripts/prefabs/woby_rack.lua#L282)）、净变量名串（L208-213）、依赖（[wobybig.lua:36](../../../raw/current/scripts/prefabs/wobybig.lua#L36)）与生成（[L236-L237](../../../raw/current/scripts/prefabs/wobybig.lua#L236)）——**生成点唯一**（一次两实例）；`wolfgang_coach_buff_fx` 命中依赖（[wolfgang.lua:29](../../../raw/current/scripts/prefabs/wolfgang.lua#L29)）、生成（[L714](../../../raw/current/scripts/prefabs/wolfgang.lua#L714)）、注册（[L790](../../../raw/current/scripts/prefabs/wolfgang.lua#L790)）——**生成点唯一**。3 注册均不在 prefablist.lua。清单外未发现 FX 注册；`Prefab("woby_rack"` 全域 0 命中（幽灵条目，见正文）。
- 名称核验：`languages/chinese_s.po` 定向检索——`WITCHBROOM`＝"女巫扫帚"、`WITCHBROOM_LIVING`＝"附魔女巫扫帚"（[L352319-L352328](../../../raw/current/scripts/languages/chinese_s.po#L352319-L352327)）、`WOBYBIG`／`WOBYSMALL`＝"沃比"（[L352359-L352368](../../../raw/current/scripts/languages/chinese_s.po#L352359-L352367)）、`WOLFGANG_WHISTLE`＝"训练哨"（[L352379-L352383](../../../raw/current/scripts/languages/chinese_s.po#L352379-L352382)）；`WITCHBROOM_FX`／`WOBY_RACK`／`WOLFGANG_COACH_BUFF` 无 NAMES 键。标题中文使用已核验效果描述。
- 分档说明：`size_class: standard`——3 注册三条独立链（轨迹池链／烘干架两层链／教练 buff 链）加一个幽灵注册辨析，各链用表格压缩；未选 complex，三链互不依赖、无整体生命周期。
- 未覆盖：扫帚 sweeper 打扫交互与武器磨损（`WITCHBROOM_WEAPON_ATTACKWEAR`）、woby 转形状态机与对齐皮肤 FX（`woby_shadow_fx` 另页）、`wobyrack` 继承的 dryingrack 烘干数值、教练激励的跟随者距离与 `cheer` 事件消费、`woby_rack_container` 容器 widget 细节、动画包帧内容；未运行游戏。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 结果见本页交付记录（0 error 0 warning 后入库）。
- 第 16 轮补正（2026-11-07）：拆分 `SetParent` 与 Lua `children` 的删除归因；轨迹池区分“在播自删”与“回池项无显式清池路径、不能依赖再次 animover”；撤回“卸载无需清理”；示例补 `GetPooledTrailFx` 复用协议并标为片段；烘干架改为每非专服 2 联网层＋6 本地槽层、褪色轮询归网络层；骑乘不再等同无 `moving`（补 `SGwilson.lua` 来源）。
- 游戏运行：未运行；`validation` 保持 `source-reviewed`。

## 相关链接

- [[reference/prefabs/woby_dash_shadow_fx|沃比暗影冲刺特效]]（`woby_dash_fade`／`woby_align_fade` 隐身标签的来源层）
- [[reference/prefabs/wortox_soulecho_buff_fx|沃拓克斯灵魂增益与装备跟随特效族]]（父挂粒子与 FollowSymbol 两种跟随的对照）
- [[reference/prefabs/fire|火焰、火光与烟雾特效族]]（若给拖尾加燃烧语义的正规接入）
- 返回 [[reference/prefabs/fx-character-index|角色技能、施法与读书]]（角色技能/装备）
- 2026-11-08 build 26474873：本页随版本迁移升版（锚点迁移 1 处），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-12）。
