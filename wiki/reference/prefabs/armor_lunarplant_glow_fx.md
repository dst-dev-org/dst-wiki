---
title: "装备与帽饰跟随光效（armor_lunarplant_glow_fx 等）"
created: "2026-11-04"
updated: "2026-11-08"
type: prefab
size_class: complex
summary: "七项装备／护符／帽饰／电池附属 FX：四注册护甲光效控制器与本地子帧、魔光护符独立光源、启迪之冠前后层与帽片符号跟随；记录工厂参数、端边界、生成点、跟随符号与清理。"
aliases: ["armor_lunarplant_glow_fx", "armor_lunarplant_husk_glow_fx", "armor_voidcloth_fx", "armorwagpunk_fx", "yellowamuletlight", "alterguardian_hat_equipped", "alterguardianhatshard_symbol_fx", "亮茄盔甲光效", "荆棘茄甲光效", "虚空长袍光效", "W.A.R.B.I.S.盔甲光效", "魔光护符光效", "启迪之冠光效", "帽片符号光效", "装备跟随光效"]
tags: [prefab, fx, server-client]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/armor_lunarplant.lua", "prefabs/armor_voidcloth.lua", "prefabs/armor_wagpunk.lua", "prefabs/amulet.lua", "prefabs/alterguardian_hat_equipped.lua", "prefabs/alterguardianhatshard.lua", "prefabs/hats.lua", "prefabs/winona_battery_high.lua", "prefablist.lua", "entityscript.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 装备与帽饰跟随光效（armor_lunarplant_glow_fx 等）

## 快速判断

- 职责与适用：本页只收宿主装备／护符／帽饰／电池的附属 FX，不展开宿主本体的防御、燃料、装备槽、容器与目标追踪数值。按实现分三类：
  1. **护甲控制器＋本地子帧**：`armor_lunarplant_glow_fx`、`armor_lunarplant_husk_glow_fx`、`armor_voidcloth_fx`、`armorwagpunk_fx`。父控制器网络化但没有 AnimState；可见子帧是 `Non-networked entity`，各端本地创建后跟随宿主 `swap_body`。
  2. **独立光源**：`yellowamuletlight`，网络化 Light 实体，父挂持有者，没有 AnimState、Follower 或颜色同步。
  3. **独立可见跟随实体**：`alterguardian_hat_equipped`（前／后两层，跟随 `hair`）与 `alterguardianhatshard_symbol_fx`（跟随电池 `gemN` 符号）。每个可见实体自身即完整实体，没有再套控制器子帧。
- 优先使用：改护甲光效时从对应护甲文件的装备段进入（亮茄／荆棘茄共用 `onequip` 的动态拼名，虚空长袍与 W.A.R.B.I.S. 盔甲各自 Spawn／Attach）；要复用控制器结构看 `MakeGlow` 或两套 `fxfn`；要独立照明看 `yellowlightfn`；要帽子前后层看 `hats.lua` 的激活／停用三函数；要电池槽位跟随看 `SetGem`／`UnsetGem`。
- 重要副作用：四种护甲控制器在持有者有 `colouradder` 时把自身挂为其子级，可见子帧各自 `highlightchild:SetOwner(owner)`；专用服务器跳过子帧；皮肤／等级只通过 netvar 同步。帽片 FX 的运行时颜色与帽层 Skin／火焰覆写在主模拟写入，客户端复现范围未运行核验；护符灯参数在 `SetPristine` 前设置。
- 修改边界：生成与 `AttachToOwner`／`OnActivated`／`SetupFxFromHatShard` 都在主模拟分支暴露；客户端在护甲控制器的 `OnEntityReplicated`（及非专服主模拟的 `AttachToOwner`）里创建本地子帧，netvar 回调只更新已存在的 `inst.fx` 的 skin／bank，帽层与帽片只接收网络实体复制。`size_class: complex` 因本页覆盖 6 套工厂、7 个宿主／变体、三种端模型与多条清理路径，需要整体说明。
- 名称：七个注册名在 `languages/chinese_s.po` 的 `STRINGS.NAMES` 中均无键，标题中文是已核验效果描述而非官方译名。宿主官方名（`STRINGS.NAMES`，用于本文指代）：[亮茄盔甲](../../../raw/current/scripts/languages/chinese_s.po#L338840-L338842)、[荆棘茄甲](../../../raw/current/scripts/languages/chinese_s.po#L338845-L338847)、[虚空长袍](../../../raw/current/scripts/languages/chinese_s.po#L338860-L338862)、[W.A.R.B.I.S.盔甲](../../../raw/current/scripts/languages/chinese_s.po#L338820-L338822)、[魔光护符](../../../raw/current/scripts/languages/chinese_s.po#L352755-L352757)、[启迪之冠](../../../raw/current/scripts/languages/chinese_s.po#L338515-L338517)、[薇诺娜的宝石发电机](../../../raw/current/scripts/languages/chinese_s.po#L352050-L352052)。

## 注册与构造

共同端边界：七个注册都由主模拟的装备／激活／加宝石路径生成；客户端不会调用 `AttachToOwner`、`OnActivated` 或 `SetupFxFromHatShard`。护甲控制器的子帧注释标明 `Non-networked entity`，专用服务器跳过（[armor_lunarplant.lua:381-384](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L381-L384)）。

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `armor_lunarplant_glow_fx` | `MakeGlow`（[L402-L433](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L402-L433)）：Transform＋Network → `FX` 标签 → `colouraddersync` → `build="armor_lunarplant"` → `net_hash(..., ".skinbuildhash")` → `SetPristine`；非主模拟挂 `OnEntityReplicated`，主模拟只暴露 `AttachToOwner`、`persists=false`（[L421-L430](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L421-L430)）。 | 父控制器无 AnimState；`glow_SpawnFxForOwner` 建 6 个非网络子帧（[L347-L360](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L347-L360)），父到持有者、`FollowSymbol(owner.GUID, "swap_body", ..., i-1)`，播 `idle1..6`，`glowcentre` bloom＋0.5 light override（[L309-L332](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L309-L332)）；皮肤经 `skinbuildhash` 同步（[L387-L399](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L387-L399)）。注册在 [L437](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L437)。 |
| `armor_lunarplant_husk_glow_fx` | 同一 `MakeGlow`，`build="armor_lunarplant_husk"`、资源用 `huskassets`；注册在 [L440](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L440)。 | 子帧 bank 仍固定 `armor_lunarplant`（[L321](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L321)），动画同为 `idle1..6`，只换 build。装备侧用 `inst.prefab.."_glow_fx"` 动态选名，因此荆棘茄甲自然得到 husk 变体（[L85-L87](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L85-L87)）。 |
| `armor_voidcloth_fx` | 独立 `fxfn`（[L250-L276](../../../raw/current/scripts/prefabs/armor_voidcloth.lua#L250-L276)）：同样是 Transform＋Network、`FX`、`colouraddersync`、`SetPristine` 的主模拟分界；net 变量名硬编码 `armor_voidcloth_fx.skinbuildhash`（[L260](../../../raw/current/scripts/prefabs/armor_voidcloth.lua#L260)）。注册在 [L280](../../../raw/current/scripts/prefabs/armor_voidcloth.lua#L280)。 | 9 个子帧，bank／build `armor_voidcloth`，先播 `settle1..9` 再推 `idle1..9`（[L149-L168](../../../raw/current/scripts/prefabs/armor_voidcloth.lua#L149-L168)）；持有者有 `locomotor` 时给控制器追加 `autojiggle`，one-shot 回调重播 settle→idle（[L203-L207](../../../raw/current/scripts/prefabs/armor_voidcloth.lua#L203-L207)、[L177-L183](../../../raw/current/scripts/prefabs/armor_voidcloth.lua#L177-L183)）。`local frame` 已声明但从未 `SetFrame`（[L194](../../../raw/current/scripts/prefabs/armor_voidcloth.lua#L194)）。 |
| `armorwagpunk_fx` | 独立 `fxfn`（[L490-L516](../../../raw/current/scripts/prefabs/armor_wagpunk.lua#L490-L516)）：Transform＋Network、`FX`、`colouraddersync`；`net_tinybyte` 的 `level` 变量与客户端 dirty 监听（[L500-L503](../../../raw/current/scripts/prefabs/armor_wagpunk.lua#L500-L503)）；主模拟 `AttachToOwner`、`persists=false`。注册在 [L541](../../../raw/current/scripts/prefabs/armor_wagpunk.lua#L541)。 | 9 个子帧 `on1..on9`，bank／build `armor_wagpunk_01`（[L405-L423](../../../raw/current/scripts/prefabs/armor_wagpunk.lua#L405-L423)）；等级 dirty 只切子帧 bank：1 级→`armor_wagpunk_01`，2 至 5 级全部→`armor_wagpunk_05`（[L473-L486](../../../raw/current/scripts/prefabs/armor_wagpunk.lua#L473-L486)）。没有 `skinbuildhash`，不随皮肤换 build。同文件 `steamon_fxfn`（[L520-L537](../../../raw/current/scripts/prefabs/armor_wagpunk.lua#L520-L537)）未被注册、也无任何引用，是死代码。 |
| `yellowamuletlight` | `yellowlightfn`（[L599-L621](../../../raw/current/scripts/prefabs/amulet.lua#L600-L622)）：Transform＋Light＋Network → `FX` 标签 → 半径 2、falloff .7、强度 .65、颜色 (223,208,69)/255（[L608-L611](../../../raw/current/scripts/prefabs/amulet.lua#L609-L612)）→ `SetPristine`；主模拟 `persists=false`。注册在 [L630](../../../raw/current/scripts/prefabs/amulet.lua#L631)。 | 无 AnimState、Follower、colouraddersync、netvar；灯光参数在 `SetPristine` 前写入，随实体初始状态同步。它是独立光源，不是符号跟随动画。 |
| `alterguardian_hat_equipped` | `fn`（[L59-L93](../../../raw/current/scripts/prefabs/alterguardian_hat_equipped.lua#L59-L93)）：AnimState bank／build `hat_alterguardian_equipped`，`idle` 循环，`SetFinalOffset(1)`，bloom `shaders/anim.ksh`，`flame_swap` 初始 alpha .2／light override .5；`Transform:SetNoFaced`；`FX`＋`DECOR`；主模拟暴露 `SetSkin`／`SetFlameLevel`／`OnActivated`／`OnDeactivated`（[L88-L91](../../../raw/current/scripts/prefabs/alterguardian_hat_equipped.lua#L88-L91)）；无 netvar。注册在 [L96](../../../raw/current/scripts/prefabs/alterguardian_hat_equipped.lua#L96)。 | 每次激活由 `hats.lua` 生成两份：前层 `is_front=true`、后层 `false`。`OnActivated` 设父、`FollowSymbol(owner.GUID, "hair", 0, 0, 0)`、隐藏相反层、`SetFinalOffset(±1)`，播 `activate_pre`→`activate_loop`（[L8-L18](../../../raw/current/scripts/prefabs/alterguardian_hat_equipped.lua#L8-L18)）；停用时 `OnDeactivated` 播 `activate_pst` 并挂 `animover` 自删（[L20-L23](../../../raw/current/scripts/prefabs/alterguardian_hat_equipped.lua#L20-L23)）。`SetSkin` 覆写 `p4_piece`／`fx_glow`，`SetFlameLevel` 按等级覆写 `flame_swap`／`flame_outline_swap` 并监听 `snowlevel`（[L25-L57](../../../raw/current/scripts/prefabs/alterguardian_hat_equipped.lua#L25-L57)），帽子侧在 `alterguardianhat_updatelight` 调用（[L3437-L3448](../../../raw/current/scripts/prefabs/hats.lua#L3437-L3448)）。 |
| `alterguardianhatshard_symbol_fx` | `symbolfxfn`（[L164-L193](../../../raw/current/scripts/prefabs/alterguardianhatshard.lua#L164-L193)）：Transform＋AnimState＋Follower＋Network；bank／build `alterguardianhatshard`，`float` 循环，bloom，`SetLightOverride(0.5)`；`FX`＋`NOCLICK`；主模拟随机起始帧并暴露 `SetupFxFromHatShard`（[L187-L189](../../../raw/current/scripts/prefabs/alterguardianhatshard.lua#L187-L189)）；`persists=false`。注册在 [L197](../../../raw/current/scripts/prefabs/alterguardianhatshard.lua#L197)。 | 不挂在角色上，而是由 `winona_battery_high` 的宝石槽生成：实体父挂电池、`FollowSymbol(inst.GUID, symbol, 0, 0, 0, true)`，槽位符号由 `GetGemSymbol` 得 `gem3`／`gem2`／`gem1`（[winona_battery_high.lua:170-L172](../../../raw/current/scripts/prefabs/winona_battery_high.lua#L170-L172)）。`SetupFxFromHatShard` 读取帽片容器第 1 格孢子，把红／绿／蓝转成 `SetMultColour`（[L151-L161](../../../raw/current/scripts/prefabs/alterguardianhatshard.lua#L151-L161)、[L138-L149](../../../raw/current/scripts/prefabs/alterguardianhatshard.lua#L138-L149)）。 |

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| `组件：colouraddersync` | 四个护甲控制器都挂该组件；spawn 子帧时设置颜色回调，回调把 RGBA 写到每个子帧的 `AnimState:SetAddColour`。 | 亮茄／荆棘茄、虚空长袍、W.A.R.B.I.S. 盔甲；持有者有 `colouradder` 时控制器挂为其子级并由其驱动。 | [亮茄回调](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L341-L345)、[虚空回调](../../../raw/current/scripts/prefabs/armor_voidcloth.lua#L185-L189)、[WAGpunk 回调](../../../raw/current/scripts/prefabs/armor_wagpunk.lua#L432-L436)、[[reference/components/colouraddersync]] |
| `组件：highlightchild` | 可见子帧创建后 `SetOwner(owner)`，登记进持有者高亮子表。 | 仅四个护甲控制器的子帧；控制器自身、护符灯、帽层与帽片均不挂。 | [亮茄](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L328)、[虚空](../../../raw/current/scripts/prefabs/armor_voidcloth.lua#L164)、[WAGpunk](../../../raw/current/scripts/prefabs/armor_wagpunk.lua#L419)、[[reference/components/highlightchild]] |
| `组件：autojiggle` | 虚空长袍独有：持有者有 `locomotor` 时给控制器加组件，只配 one-shot 回调，让 9 个子帧重播 settle→idle。 | 仅 `armor_voidcloth_fx`；客户端 `OnEntityReplicated` 与非专服主模拟各挂一份，专服不跑。 | [挂载](../../../raw/current/scripts/prefabs/armor_voidcloth.lua#L203-L207)、[[reference/components/autojiggle]] |
| `Follower`（引擎） | 可见实体用 `FollowSymbol` 贴合宿主符号：护甲子帧→`swap_body`，启迪之冠→`hair`，帽片→电池 `gemN`。 | 护甲子帧末参 `i-1`；帽层带 0 偏移；帽片末参 `true`。护符灯无 Follower。 | [护甲](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L354)、[帽层](../../../raw/current/scripts/prefabs/alterguardian_hat_equipped.lua#L11)、[帽片](../../../raw/current/scripts/prefabs/winona_battery_high.lua#L278) |
| `netvar：skinbuildhash` | 亮茄／荆棘茄与虚空长袍同步皮肤 build；装备侧先把 `owningitem` 交给 FX，`AttachToOwner` 读 `GetSkinBuild` 后 set，dirty 时子帧 `SetSkin` 或回退 `SetBuild`。 | 两个控制器各用 `net_hash`；值为 0 时回退普通 build；仅非专服监听。 | [亮茄 set](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L375-L380)、[亮茄 dirty](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L387-L399)、[虚空 dirty](../../../raw/current/scripts/prefabs/armor_voidcloth.lua#L235-L247) |
| `netvar：level` | W.A.R.B.I.S. 盔甲把目标追踪阶段写进 `net_tinybyte`；客户端 dirty 时切换 9 个子帧的 bank。 | 装备时置 1；新目标／阶段变化写 2–5；重置回 1。只影响表现，阶段数值与蒸汽在护甲侧。 | [装备置 1](../../../raw/current/scripts/prefabs/armor_wagpunk.lua#L245-L250)、[阶段写值](../../../raw/current/scripts/prefabs/armor_wagpunk.lua#L168-L201)、[dirty 切 bank](../../../raw/current/scripts/prefabs/armor_wagpunk.lua#L473-L486)、[[reference/prefabs/wagpunksteam_hat_up]] |
| `宿主：亮茄盔甲／荆棘茄甲` | 装备时移除旧 FX，`SpawnPrefab(inst.prefab.."_glow_fx")`，设 `owningitem` 后 `AttachToOwner`；同时把持有者 `swap_body` 的 light override 设为 .1。卸下移除 FX 并恢复 0。 | 两个宿主共用同一 `onequip`／`onunequip`；荆棘茄甲走 husk 注册名。 | [生成](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L82-L88)、[清理](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L108-L112) |
| `宿主：虚空长袍` | 装备时移除旧 FX，Spawn＋`owningitem`＋`AttachToOwner`；卸下 `Remove` 并清字段。 | 皮肤 hash 与 `autojiggle` 都在此链上。 | [生成](../../../raw/current/scripts/prefabs/armor_voidcloth.lua#L27-L32)、[清理](../../../raw/current/scripts/prefabs/armor_voidcloth.lua#L47-L49) |
| `宿主：W.A.R.B.I.S. 盔甲` | 装备时移除旧 FX，Spawn＋`AttachToOwner`＋`level:set(1)`；卸下 `Remove` 并清字段，同时取消未执行的蒸汽任务。 | 蒸汽 `wagpunksteam_armor_up`／`down` 由护甲自己的 helper 生成，不是本页 FX。 | [生成](../../../raw/current/scripts/prefabs/armor_wagpunk.lua#L241-L250)、[清理](../../../raw/current/scripts/prefabs/armor_wagpunk.lua#L280-L288)、[蒸汽 helper](../../../raw/current/scripts/prefabs/armor_wagpunk.lua#L39-L59) |
| `宿主：魔光护符` | 装备时若灯无效则 Spawn，并把实体父挂持有者；卸下、转入手持模型、护符移除、燃料耗尽删除都走 `turnoff_yellow`。 | 灯不跟随符号，只随父实体；重新装备会新建。 | [生成＋父挂](../../../raw/current/scripts/prefabs/amulet.lua#L266-L269)、[关灯](../../../raw/current/scripts/prefabs/amulet.lua#L282-L288)、[卸下](../../../raw/current/scripts/prefabs/amulet.lua#L291-L308)、[移除兜底](../../../raw/current/scripts/prefabs/amulet.lua#L594-L595) |
| `宿主：启迪之冠` | 理智高于阈值时激活（[阈值](../../../raw/current/scripts/prefabs/hats.lua#L3582-L3584)）：先隐藏 swap_hat，再生成前／后两份并 `OnActivated`，有皮肤则 `SetSkin`；另生成独立的 `alterguardianhatlight`。停用时前后层都走 `OnDeactivated`，前层 8 帧后重新走 `opentop_onequip`，后层由 `animover` 自删且不再重建。 | 装备卸下与帽子移除直接 `Remove` 前后层；帽层动画 `animover` 自删是停用路径的收尾。 | [激活与生成](../../../raw/current/scripts/prefabs/hats.lua#L3451-L3484)、[停用](../../../raw/current/scripts/prefabs/hats.lua#L3486-L3512)、[卸下清理](../../../raw/current/scripts/prefabs/hats.lua#L3625-L3632)、[移除清理](../../../raw/current/scripts/prefabs/hats.lua#L3644-L3652) |
| `宿主：薇诺娜的宝石发电机` | `SetGem` 为帽片槽 Spawn 符号 FX、父挂电池、跟随 `gemN`；`UnsetGem` 与再次 `SetGem` 先移除旧 FX。便携物品形态在 `Item_OnLoad` 恢复宝石时也会走 `SetGem`。 | 同一文件的结构与物品两形态共用 `SetGem`／`UnsetGem`；帽片 FX 依赖声明在结构 prefab 的 `prefabs` 表。 | [SetGem](../../../raw/current/scripts/prefabs/winona_battery_high.lua#L263-L292)、[UnsetGem](../../../raw/current/scripts/prefabs/winona_battery_high.lua#L294-L317)、[物品恢复](../../../raw/current/scripts/prefabs/winona_battery_high.lua#L1503-L1509)、[依赖](../../../raw/current/scripts/prefabs/winona_battery_high.lua#L20-L27) |
| `关联 Prefab：hitsparks_reflect_fx` | 亮茄盔甲在反弹伤害时生成命中火花；本页的 glow FX 不生成它。 | 仅亮茄／荆棘茄的 `prefabs` 依赖，属于同宿主表现。 | [依赖](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L14-L23)、[生成点](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L163-L168)、[[reference/prefabs/hitsparks_fx]] |
| `关联 Prefab：wagpunksteam_armor_up／down` | W.A.R.B.I.S. 盔甲的阶段蒸汽与音效，由护甲侧 `SpawnSteamFX` 延迟 `AddChild`；不是 `armorwagpunk_fx` 的子帧。 | 追踪到新目标或阶段变化时生成；蒸汽条目见 [[reference/prefabs/wagpunksteam_hat_up]]。 | [helper](../../../raw/current/scripts/prefabs/armor_wagpunk.lua#L39-L59)、[[reference/prefabs/wagpunksteam_hat_up]] |
| `关联 Prefab：alterguardianhatlight` | 启迪之冠激活时的独立光源，和本页 `yellowamuletlight` 是不同注册；帽层 FX 不负责照明。 | 由帽子激活／停用生成与移除，不属于本页七注册。 | [生成](../../../raw/current/scripts/prefabs/hats.lua#L3479-L3483)、[移除](../../../raw/current/scripts/prefabs/hats.lua#L3492-L3495) |
| `资源：anim/armor_lunarplant.zip` 等 | 亮茄两变体共用 `armor_lunarplant.zip`，荆棘茄甲另加 `armor_lunarplant_husk.zip`；虚空长袍用 `armor_voidcloth.zip`；W.A.R.B.I.S. 盔甲 fx 声明 01–05 五套 zip。 | 资源声明不等于实际动画存在；帽层用 `hat_alterguardian_equipped.zip`，帽片用 `alterguardianhatshard.zip`。 | [亮茄资源](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L1-L10)、[虚空资源](../../../raw/current/scripts/prefabs/armor_voidcloth.lua#L1-L4)、[WAGpunk 资源](../../../raw/current/scripts/prefabs/armor_wagpunk.lua#L1-L17)、[帽层资源](../../../raw/current/scripts/prefabs/alterguardian_hat_equipped.lua#L1-L4)、[帽片资源](../../../raw/current/scripts/prefabs/alterguardianhatshard.lua#L15-L18) |

## 典型接入

### 复用护甲控制器＋子帧

- 前提与选择：宿主物品在主模拟的 `OnEquip` 中 `SpawnPrefab` 控制器，再调用 `AttachToOwner(owner)`；需要皮肤同步的两项还要先把 `owningitem` 指回装备。卸下时 `inst.fx:Remove()` 并清字段。原版护甲链基本是这一顺序（[生成](../../../raw/current/scripts/prefabs/armor_voidcloth.lua#L27-L32)、[清理](../../../raw/current/scripts/prefabs/armor_voidcloth.lua#L47-L49)）；W.A.R.B.I.S. 盔甲不设 `owningitem`，因为它没有皮肤 hash。
- 生效与撤销：控制器必须联网，可见子帧由各端 `SpawnFxForOwner` 本地创建；`AttachToOwner` 只存在于主模拟分支，不要在客户端调用。皮肤要生效，宿主需在 Attach 前设置 `owningitem` 且其 AnimState 有 `GetSkinBuild`（[读取](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L375-L380)）。
- 原版依据：控制器移除时靠 `OnRemoveEntity` 遍历 `inst.fx` 删子帧，`inst:Remove()` 会触发该字段（[子帧清理](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L335-L339)、[Remove 语义](../../../raw/current/scripts/entityscript.lua#L1792-L1793)）。

### 独立实体与控制器之间选择

- 需要“每个子帧叠一层”的跟随外观时用控制器＋子帧；只需要一盏灯时用 `yellowamuletlight` 式独立实体（无 AnimState／Follower）；需要前后层各自完整动画时用启迪之冠式两份独立实体；需要贴合某个建筑符号时用帽片式单实体符号跟随。
- 不要为了统一而把独立灯塞进控制器：控制器的子帧是 `Non-networked entity`，灯则需要把 Light 参数放进 `SetPristine` 前的初始状态才能跨端（[灯初始化](../../../raw/current/scripts/prefabs/amulet.lua#L600-L622)）。

### 状态同步的接入位

- 皮肤：主模拟 `net_hash` 写值，客户端 dirty 回调把 `SetSkin`／`SetBuild` 应用到每个子帧。
- 等级：主模拟写 `net_tinybyte`，客户端 dirty 只改子帧 bank；不要把“等级”当成玩法伤害值，它在本页只驱动外观。
- 帽层：`SetSkin`／`SetFlameLevel`／`OnActivated` 都是主模拟方法，调用点在 `hats.lua`；客户端复现范围未运行核验。

## 常见陷阱与限制

- **误区：这些 FX 是装备实体的一部分**：七项都是独立 prefab，装备／激活／加宝石时才生成；卸下或换槽即删除。护甲本体的防御、燃料、装备槽与目标追踪不在本页。
- **边界：护甲控制器本身不可见**：`armor_lunarplant_glow_fx`、`armor_lunarplant_husk_glow_fx`、`armor_voidcloth_fx`、`armorwagpunk_fx` 的父实体只有 Transform／Network，没有 AnimState；真正可见的是 6／9／9 个本地子帧。专用服务器没有这些子帧（[专服分支](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L381-L384)）。
- **陷阱：W.A.R.B.I.S. 盔甲 2–4 级也切到 05 bank**：dirty 分支把 2、3、4、5 全映射为 `armor_wagpunk_05`，只有 1 级用 01；同时只 `SetBank`、不 `SetBuild`，02–04 的 zip 虽在资源表中但这条分支不引用（[分支](../../../raw/current/scripts/prefabs/armor_wagpunk.lua#L473-L486)）。同族帽子的分支是 2→02、3→03、4→04、5→05（[帽子分支](../../../raw/current/scripts/prefabs/hats.lua#L7064-L7072)），不能把帽子的等级观感直接套到护甲。
- **边界：虚空长袍的 `local frame` 是未使用变量**：它声明了随机帧变量但没有 `SetFrame`；亮茄则把一次随机结果共享给 6 个子帧，WAGpunk 完全不设帧（[虚空](../../../raw/current/scripts/prefabs/armor_voidcloth.lua#L191-L201)、[亮茄](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L347-L357)）。三者的起始帧行为不同，改动画对时不要假设一致。
- **边界：`yellowamuletlight` 不是符号跟随**：灯只 `SetParent(owner.entity)`，没有 Follower、AnimState 或颜色同步；它的存在与删除完全由魔光护符侧管理。玩家实体被移除时灯是否随父实体清理，本页未单独核验。
- **边界：启迪之冠前后层的收尾不对称**：停用时两层都走 `OnDeactivated` 播 `activate_pst`，但前层 8 帧后重新走 `opentop_onequip`，后层只清字段、由 `animover` 自删且不再重建。装备卸下与帽子移除则直接 `Remove` 两层，不走动画（[OnDeactivated](../../../raw/current/scripts/prefabs/alterguardian_hat_equipped.lua#L20-L23)、[前后层停用](../../../raw/current/scripts/prefabs/hats.lua#L3497-L3511)、[直接清理](../../../raw/current/scripts/prefabs/hats.lua#L3625-L3632)）。
- **边界：帽片颜色没走颜色同步组件**：`SetupFxFromHatShard` 在主模拟用 `AnimState:SetMultColour` 写红／绿／蓝，FX 没有 `colouraddersync`；客户端是否复现该颜色未运行核验。空的帽片不会重置颜色（`SetShardColour(nil)` 没有 `else` 分支），但每次 `SetGem` 都新建实例，通常不会在同一实例上残留（[颜色](../../../raw/current/scripts/prefabs/alterguardianhatshard.lua#L138-L161)）。
- **死代码：`steamon_fxfn` 不产生任何实体**：它在 `armor_wagpunk.lua` 中定义，但既没有 `Prefab("...", steamon_fxfn, ...)` 注册，也没有 `SpawnPrefab` 引用；不是 `armorwagpunk_fx` 的别名，也没有进入 `prefabs` 表（[定义](../../../raw/current/scripts/prefabs/armor_wagpunk.lua#L520-L538)）。同一文件的 `armorwagpunk_fx` 才是实际注册。
- **边界：本族没有 `SetFX` 式状态切换**：控制器在重装时走“旧 `inst.fx:Remove()` → 新 Spawn”；电池换宝石走 `SetGem` 内先移除旧符号 FX 再生成；帽层走 `OnDeactivated`／直接 `Remove`。不要把其他系统的 `SetFX` 形态套到这里。

## 验证记录

- 源码审阅：2026-11-04，Steam buildid 26335394。逐个核验七个注册名的返回位置、工厂参数（AnimState／Light／Follower／colouraddersync／autojiggle／netvar）、主模拟与客户端分界、生成点、父挂与跟随符号、清理路径；并核验宿主侧 `hats.lua` 与 `winona_battery_high.lua` 的调用链。全 `raw/current/scripts` 反查七个注册名与 `steamon`：命中仅上述注册、依赖、生成点与 `prefablist.lua` 的 `alterguardian_hat_equipped` 条目；`steamon` 只命中定义行。`languages/chinese_s.po` 定向检索 `STRINGS.NAMES` 键，七个注册名均无命中；宿主官方名按实际 NAMES 键记录。
- 未覆盖：客户端实际颜色／皮肤／等级观感、专用服务器与晚加入、玩家死亡或主机迁移时的父挂清理、动画资源实际存在性与帧长、非简体中文名称、运行时性能。`SetSkin`／`SetFlameLevel`／`SetMultColour` 的跨端表现只按调用位置记录，未运行验证。
- 游戏运行：未运行。
- 结构检查：`validate_page.py`（complex 档）与 `validate_source_links.py --strict` 均通过；无 Lua 示例，故未执行 `parser_probe.exe`。实际行数、字节与引用数见本批交付报告。

## 相关链接

- [[reference/prefabs/wagpunksteam_hat_up|WAGpunk 装备蒸汽（wagpunksteam_hat_up 等）]]：同组蒸汽与阶段变化的宿主侧入口。
- [[reference/prefabs/hitsparks_fx|命中火花特效（hitsparks_fx）]]：亮茄盔甲反弹火花与本页 glow FX 的协作边界。
- [[reference/components/autojiggle|autojiggle]]、[[reference/components/colouraddersync|colouraddersync]]、[[reference/components/highlightchild|highlightchild]]：本页三个表现组件。

返回 [[reference/prefabs/fx-skin-index|皮肤与装备挂件]]。
- 2026-11-08 build 26474873：本页随版本迁移升版（锚点迁移 5 处），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-10）。
