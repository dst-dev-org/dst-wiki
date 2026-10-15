---
title: "命中与落点伤害表现及传送门投影 FX 族（impact 等 7 注册）"
created: "2026-11-04"
updated: "2026-11-07"
type: prefab
size_class: complex
summary: "六个源文件、七个注册的外围载体：impact 网络壳＋非专服本地命中动画；houndstooth_proj 自带 weapon／planardamage／Light 的弹道与独立尾迹；junkball_fx 抛掷第一段与 junkball_fall_fx 落点（对每个目标按条件择一破坏／采摘／攻击）；lunarrift_terraformer_explosion 第 5 帧爆炸直伤；lunarrift_portal_shadow 与 multiplayer_portal_moonrock_fx 的父挂投影。"
aliases: ["impact", "houndstooth_proj", "junkball_fx", "junkball_fall_fx", "lunarrift_terraformer_explosion", "lunarrift_portal_shadow", "multiplayer_portal_moonrock_fx", "月形爆裂", "命中特效", "落点伤害", "传送门投影", "裂隙阴影", "天体传送门特效", "外围 FX 族"]
tags: [prefab, fx, network-sync, server-client]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/impact.lua", "prefabs/houndstooth_blowpipe.lua", "prefabs/junkball.lua", "prefabs/lunarrift_terraformer_explosion.lua", "prefabs/lunarrift_portal.lua", "prefabs/multiplayer_portal.lua", "components/projectile.lua", "components/weapon.lua", "components/combat.lua", "prefabs/rift_terraformer.lua", "prefabs/junk_pile_big.lua", "stategraphs/SGdaywalker2.lua", "prefabs/bernie_big.lua", "prefabs/blowdart.lua", "prefabs/blowdart_lava.lua", "prefabs/blowdart_lava2.lua", "prefabs/boomerang.lua", "prefabs/bullkelp_root.lua", "prefabs/gestalt_guard_evolved.lua", "prefabs/hutch.lua", "prefabs/whip.lua", "stategraphs/SGshadowwaxwell.lua", "stategraphs/SGwilson.lua", "stategraphs/SGwx78_possessedbody.lua", "tuning.lua", "constants.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 命中与落点伤害表现及传送门投影 FX 族（impact 等 7 注册）

## 快速判断

- 职责与适用：六个源文件里的七个外围载体，按职责分三组。纯表现：`impact`（通用命中贴片）、`lunarrift_portal_shadow`（裂隙静态投影）、`multiplayer_portal_moonrock_fx`（天体门射线层）。弹道／落点混合：`houndstooth_proj`（自带 weapon／planardamage／Light 的弹体）、`junkball_fx`＋`junkball_fall_fx`（抛掷与落点的伤害／破坏／采摘／抛飞混合载体）、`lunarrift_terraformer_explosion`（爆炸直伤）。宿主玩法——吹箭装填、拾荒疯猪投掷、裂隙地块生成、传送门建造——不在本页展开。
- 优先使用：命中瞬间贴片用 `impact`，只定位播 `idle`、不结算伤害；弹道自带伤害与灯／尾迹用 `houndstooth_proj`，命中火花另走 `hitsparks_piercing_fx`（[[reference/prefabs/hitsparks_fx|命中火花特效（hitsparks_fx）]]）；抛掷落点先 `junkball_fx` 再让它生成 `junkball_fall_fx`，伤害只在后者；范围直伤用 `lunarrift_terraformer_explosion`；静态贴地／背景层用两个投影。
- 重要副作用：`impact` 无伤害、无 combat；`houndstooth_proj` 的物理／planar／暗影加成都在弹体自身；`junkball_fall_fx` 落点对每个目标按条件**择一**：可 work → `Destroy`，否则可摘 → `Pick`，否则 `combat:CanTarget` → `DoAttack`；另把地面物品抛飞，`formpile` 条件允许时还会聚堆／击退；共享 `targets` 只对 worked／picked／attacked 去重，不覆盖抛飞等其它副作用；`lunarrift_terraformer_explosion` 没有 combat 组件，直接 `target.components.combat:GetAttacked`；两个投影都没有独立自删，退场路径不同：裂隙 `on_rift_finished` 在未休眠时给 `inst.shadow` 播 `shadow_disappear`、portal 播 `stage_3_disappear`，随后自身 animover／10 秒兜底 Remove（休眠时 portal 直接 Remove）；天体门没有对应动画或显式删 `inst.fx`，只删 portal，投影原生结果未核验。伤害归属见[关联机制导航](#关联机制导航)。
- 修改边界：本页只覆盖这七个注册的工厂、端分界、生成点、父挂与清理；动画帧长、客户端实际演出、存档恢复与宿主行为未运行核验。页面跨纯表现、弹道伤害、combat 落点、直接结算与父挂投影五类协作，需要整体对照保留，故 `size_class: complex`。

## 注册与构造

七个注册都是 `Prefab(名称, 工厂, assets[, prefabs])` 的直接返回，没有包装工厂或同名变体；`junkball.lua` 的两个注册成对出现。七个工厂都不接收外部初始化参数，差异在 `assets` 与 `prefabs` 依赖：`impact` 只声明 `anim/impact.zip`；`houndstooth_proj` 依赖 `hitsparks_piercing_fx`；`junkball_fx` 依赖 `junkball_fall_fx`，后者依赖 `junk_pile`／`splash_green_large`；`lunarrift_terraformer_explosion` 只声明 `crystalblast_fx.zip`；两个投影的 assets 分别在宿主文件内声明。以下按文件顺序逐条核验。

| 注册名／变体 | 工厂与初始化分界 | 类别、端与差异证据 |
|---|---|---|
| `impact` | `fn`（[L27-L54](../../../raw/current/scripts/prefabs/impact.lua#L27-L54)）：Transform＋Network → `SetFourFaced`＋`FX` → 非专服延迟 1 帧 `PlayImpactAnim` → `SetPristine` → 客户端早退 → 主模拟 `persists=false`、0.5 秒 Remove。本地代理（[L6-L25](../../../raw/current/scripts/prefabs/impact.lua#L6-L25)）无 Network、`SetFromProxy(proxy.GUID)`、bank／build `impact`、`idle`、FinalOffset 3、`animover` Remove。注册（[L56](../../../raw/current/scripts/prefabs/impact.lua#L56)）。 | 纯表现：壳不带 AnimState／SoundEmitter；专服只留 0.5 秒空壳，非专服（主机与客户端）各自建本地动画。 |
| `houndstooth_proj` | `ProjectileFn`（[L235-L313](../../../raw/current/scripts/prefabs/houndstooth_blowpipe.lua#L235-L313)）：Transform／Anim／Sound／Network／Light → 物理、`blow_dart`＋`dart_houndstooth` 循环、OnGround、weapon／projectile／NOCLICK／NOBLOCK → 非专服装尾迹每帧更新 → `_target` net_entity（双端）→ Pristine → 客户端早退 → 主模拟 `persists=false`、weapon＋planardamage＋暗影 1.1×、projectile 速度 25／命中距离 1.5／`has_damage_set`。注册（[L316-L317](../../../raw/current/scripts/prefabs/houndstooth_blowpipe.lua#L316-L317)）。 | 混合：尾迹（[L100-L125](../../../raw/current/scripts/prefabs/houndstooth_blowpipe.lua#L100-L125)）非联网、每帧一条、`animover` 自删；命中表现走 `hitsparks_piercing_fx`（[L148-L155](../../../raw/current/scripts/prefabs/houndstooth_blowpipe.lua#L148-L155)）。 |
| `junkball_fx` | `fn`（[L280-L320](../../../raw/current/scripts/prefabs/junkball.lua#L280-L320)）：Transform／Anim／Sound／Network → FX／NOCLICK／junk → `scrap_launch` → `updatelooper` → 非专服影子（父挂自身）→ Pristine → 客户端早退 → 主模拟 `persists=false`、`animover`→`OnAnimOver`、暴露两个 Setup。注册（[L598](../../../raw/current/scripts/prefabs/junkball.lua#L598)）。 | 抛掷第一段：无 combat；`OnAnimOver`（[L195-L278](../../../raw/current/scripts/prefabs/junkball.lua#L195-L278)）在落点前生成 6 个 `junkball_fall_fx` 并延迟移除自身；影子（[L26-L53](../../../raw/current/scripts/prefabs/junkball.lua#L26-L53)）随机 3 种贴图、背景层。 |
| `junkball_fall_fx` | `fallfn`（[L551-L594](../../../raw/current/scripts/prefabs/junkball.lua#L551-L594)）：Transform／Anim／Network → FX／NOCLICK／junk → `scrap_fall` → `updatelooper` → 非专服影子 1.3 → Pristine → 客户端早退 → 主模拟 combat：默认伤害 30、range 1.5、`KeepTarget` false、`persists=false`、`animover` Remove、暴露 `SetupJunkFall`。注册（[L599](../../../raw/current/scripts/prefabs/junkball.lua#L599)）。 | 混合：`SetupJunkFall`（[L532-L545](../../../raw/current/scripts/prefabs/junkball.lua#L532-L545)）0.5 秒飞行，落点 `UpdateFallPos`（[L470-L530](../../../raw/current/scripts/prefabs/junkball.lua#L470-L530)）先 `DoDamage` 再按海洋／聚堆分支处理；`DoDamage` 内部按 workable `Destroy`→pickable `Pick`→combat `DoAttack` 顺序，另段抛飞地面物品；聚堆分支只在 `formpile` 且未被阻挡时删树桩、推 knockback、生成 `junk_pile`。 |
| `lunarrift_terraformer_explosion` | `fn`（[L34-L63](../../../raw/current/scripts/prefabs/lunarrift_terraformer_explosion.lua#L34-L63)）：Transform／Anim／Sound／Network → bank `crystalblast`／build `crystalblast_fx`、背景层、`blast` → FX → Pristine → 客户端早退 → 主模拟 `persists=false`、5×`FRAMES` 后 `do_explosion`、`animover` Remove。注册（[L65](../../../raw/current/scripts/prefabs/lunarrift_terraformer_explosion.lua#L65)）。 | 混合：无 combat 组件；`do_explosion`（[L13-L32](../../../raw/current/scripts/prefabs/lunarrift_terraformer_explosion.lua#L13-L32)）对范围内 `_combat` 目标直接 `combat:GetAttacked(inst, 30, nil)`；半径取半地块对角线（[L10-L11](../../../raw/current/scripts/prefabs/lunarrift_terraformer_explosion.lua#L10-L11)）。 |
| `lunarrift_portal_shadow` | `portal_shadow_fn`（[L707-L735](../../../raw/current/scripts/prefabs/lunarrift_portal.lua#L707-L735)）：Transform／Anim／Network → bank／build `lunar_rift_portal`＋small override → `OnGroundFixed`、`LAYER_BACKGROUND` → `shadow_appear`→`shadow` 循环 → rotation 90 → FX；L728 `TheWorld.ismasersim` 拼写错误使早退恒成立，L732 `persists=false` 不可达（[L727-L732](../../../raw/current/scripts/prefabs/lunarrift_portal.lua#L727-L732)）。注册（[L753](../../../raw/current/scripts/prefabs/lunarrift_portal.lua#L753)）。 | 纯表现投影：由 portal 主模拟生成并父挂（[L642-L646](../../../raw/current/scripts/prefabs/lunarrift_portal.lua#L642-L646)），随阶段缩放／重播；**未休眠**收尾 `on_rift_finished` 给 `inst.shadow` 播 `shadow_disappear`、portal 播 `stage_3_disappear`，portal 自身 animover＋10 秒兜底 Remove；休眠时 portal 直接 Remove。投影自身无显式 Remove，原生随父删除未核验。 |
| `multiplayer_portal_moonrock_fx` | `moonrockfxfn`（[L429-L454](../../../raw/current/scripts/prefabs/multiplayer_portal.lua#L429-L454)）：Transform／Anim／Network → bank `portal_moonrock_dst`／build `portal_moonrock`、`idle_loop` → 隐藏 `portal`、`FX_ray1` 取自 `portal_stone` → LightOverride .2 → FX → Pristine → 客户端早退 → 主模拟 `persists=false`。注册（[L459](../../../raw/current/scripts/prefabs/multiplayer_portal.lua#L459)）。 | 纯表现：仅主模拟非洞穴生成（[L327-L330](../../../raw/current/scripts/prefabs/multiplayer_portal.lua#L327-L330)）、[L405](../../../raw/current/scripts/prefabs/multiplayer_portal.lua#L405) 父挂到天体门；无 animover／定时清理。 |

名称核验：七个注册中只有 `lunarrift_terraformer_explosion` 命中 `STRINGS.NAMES`，简中显示名为“月形爆裂”（[L344895-L344897](../../../raw/current/scripts/languages/chinese_s.po#L344895-L344897)）；其余六个注册名没有 NAMES 键，标题和正文里的中文是已核验效果描述，不是官方译名。宿主名称仅用于定位：`houndstooth_blowpipe`“嚎弹炮”（[L344015-L344017](../../../raw/current/scripts/languages/chinese_s.po#L344015-L344017)）、`lunarrift_portal`“裂隙”（[L344890-L344892](../../../raw/current/scripts/languages/chinese_s.po#L344890-L344892)）、`multiplayer_portal_moonrock`“天体传送门”（[L345970-L345972](../../../raw/current/scripts/languages/chinese_s.po#L345970-L345972)）；不能把宿主名当作对应 FX 的官方名。

### 生成点、父挂与清理

下表集中列出“全 scripts 反查”的生成点、父挂和移除路径；未写父挂的注册就是工厂内没有 `SetParent`。动画事件名只按源码记录，实际帧长未核验。

| 注册名 | 全部生成点（当前快照全量反查） | 父挂 | 清理与移除 |
|---|---|---|---|
| `impact` | 15 处 `SpawnPrefab("impact")`。prefab 侧：[blowdart L32](../../../raw/current/scripts/prefabs/blowdart.lua#L32)、[boomerang L72](../../../raw/current/scripts/prefabs/boomerang.lua#L72)、[bernie_big L48](../../../raw/current/scripts/prefabs/bernie_big.lua#L48)、[hutch L160](../../../raw/current/scripts/prefabs/hutch.lua#L160)、[whip L69](../../../raw/current/scripts/prefabs/whip.lua#L69)，另有 bullkelp_root L40、gestalt_guard_evolved L494 同型；SG 侧：[SGwilson L5275](../../../raw/current/scripts/stategraphs/SGwilson.lua#L5275)、[SGshadowwaxwell L800](../../../raw/current/scripts/stategraphs/SGshadowwaxwell.lua#L800)、[SGwx78_possessedbody L1777](../../../raw/current/scripts/stategraphs/SGwx78_possessedbody.lua#L1777)，另有 SGshadowwaxwell L861、SGwilson L5337、SGwx78 的 L1894／L4228／L4288 同型。 | 无；消费者只对壳做 `AddFollower`／`FacePoint`／`SetPosition`／`SetRotation`，不改本地代理父子。 | 主模拟 0.5 秒 Remove（[L51](../../../raw/current/scripts/prefabs/impact.lua#L51)）；本地代理 `animover` Remove（[L24](../../../raw/current/scripts/prefabs/impact.lua#L24)）；两者寿命独立。 |
| `houndstooth_proj` | 全 scripts 无直接 `SpawnPrefab("houndstooth_proj")`；生成入口是宿主 `weapon:SetProjectile("houndstooth_proj")`（[L82](../../../raw/current/scripts/prefabs/houndstooth_blowpipe.lua#L82)），宿主依赖列表也声明同名（[L7-L11](../../../raw/current/scripts/prefabs/houndstooth_blowpipe.lua#L7-L11)），由 `Weapon:LaunchProjectile` 的 `SpawnPrefab(self.projectile)` 生成（[weapon.lua L129](../../../raw/current/scripts/components/weapon.lua#L129)）。 | 无。 | 命中／落空 `OnHitFn(inst.Remove)`／`OnMissFn(inst.Remove)`（[L305](../../../raw/current/scripts/prefabs/houndstooth_blowpipe.lua#L305)、[L308](../../../raw/current/scripts/prefabs/houndstooth_blowpipe.lua#L308)）；`persists=false`；尾迹各自 `animover`。 |
| `junkball_fx` | 仅两处：[SGdaywalker2 第 21 帧](../../../raw/current/scripts/stategraphs/SGdaywalker2.lua#L1652) 调 `SetupJunkTossAttack`；[junk_pile_big L227](../../../raw/current/scripts/prefabs/junk_pile_big.lua#L227) 循环调 `SetupJunkTossFromPile`。 | 无世界父挂；影子父挂自身。 | `OnAnimOver` 生成落点后按最后一颗延迟 Remove（[L268-L277](../../../raw/current/scripts/prefabs/junkball.lua#L268-L277)）；`persists=false`。 |
| `junkball_fall_fx` | 仅 [junkball.lua L188](../../../raw/current/scripts/prefabs/junkball.lua#L188) 由 `junkball_fx` 的 `DoJunkFall` 生成；全 scripts 无其他生成点。 | 无。 | 主模拟 `animover` Remove（[L589](../../../raw/current/scripts/prefabs/junkball.lua#L589)）；`persists=false`；落点伤害在移除前由 `UpdateFallPos` 结算（[L484](../../../raw/current/scripts/prefabs/junkball.lua#L484)）。 |
| `lunarrift_terraformer_explosion` | [rift_terraformer L92](../../../raw/current/scripts/prefabs/rift_terraformer.lua#L92)（重置地块中心）、[L275](../../../raw/current/scripts/prefabs/rift_terraformer.lua#L275)（visuals 收尾）、[L381](../../../raw/current/scripts/prefabs/rift_terraformer.lua#L381)（早退）；全 scripts 仅此三处。 | 无。 | 主模拟 `animover` Remove（[L58](../../../raw/current/scripts/prefabs/lunarrift_terraformer_explosion.lua#L58)）；`persists=false`；伤害在 5 帧时点执行。 |
| `lunarrift_portal_shadow` | 仅 [lunarrift_portal L642](../../../raw/current/scripts/prefabs/lunarrift_portal.lua#L642) 的 portal 主模拟初始化；全 scripts 无其他生成点。 | 是：`SetParent(inst.entity)`，位置 0,0,0、初始缩放 0.4（[L643-L646](../../../raw/current/scripts/prefabs/lunarrift_portal.lua#L643-L646)）。 | 无自身 `animover`／定时 Remove；未休眠时 `on_rift_finished` 给 `inst.shadow` 播 `shadow_disappear`、portal 播 `stage_3_disappear` 后 animover 或 10 秒兜底 Remove，休眠时 portal 直接 Remove；投影无显式 Remove，原生父挂结果未核验。L728 拼写错误使 `persists=false` 未执行。 |
| `multiplayer_portal_moonrock_fx` | 仅 [multiplayer_portal L328](../../../raw/current/scripts/prefabs/multiplayer_portal.lua#L328)：主模拟且非洞穴时生成；全 scripts 无其他生成点。 | 是：[L405](../../../raw/current/scripts/prefabs/multiplayer_portal.lua#L405) `SetParent(inst.entity)`，只在非洞穴 master postinit。 | 无动画退场调用、无显式 `Remove(inst.fx)`；`persists=false` 不入存档；只删 portal，投影原生父挂结果未核验。 |

## 关联机制导航

### 伤害归属

混合项只在本表展开伤害来源。纯表现注册（`impact`、`junkball_fx`、`lunarrift_portal_shadow`、`multiplayer_portal_moonrock_fx`）都没有伤害组件或伤害调用。

| 混合注册 | 伤害来源与结算入口 | 数值、范围与过滤 | 证据 |
|---|---|---|---|
| `houndstooth_proj` | 弹体自身的 `weapon`＋`planardamage`；`has_damage_set=true` 让 `Projectile:Hit` 把弹体当 weapon 传给攻击者 combat 的 `DoAttack`。 | 物理与 planar 基础值都取 `wilson_attack`（34×多人攻击系数）；对 `shadow_aligned` 目标 1.1×；命中距离 1.5、速度 25、射程 30。 | [has_damage_set](../../../raw/current/scripts/prefabs/houndstooth_blowpipe.lua#L310)、[Hit](../../../raw/current/scripts/components/projectile.lua#L236-L256)、[GetDamage](../../../raw/current/scripts/components/weapon.lua#L94-L103)、[tuning](../../../raw/current/scripts/tuning.lua#L7981-L7985) |
| `junkball_fall_fx` | 落点 `DoDamage` 优先取 `inst.attacker.components.combat`，攻击者无效时退回自身 `combat`；临时把 `defaultdamage` 设为 30、`playerdamagepercent=nil`、`ignorehitrange=true`，再 `combat:DoAttack`。 | 半径 1.5＋目标物理半径；必须命中 `_combat`／pickable／workable 的 oneof 标签。每个目标按条件**择一**：可 work → `Destroy`；否则可摘且非 intense → `Pick`；否则 `combat:CanTarget` → `DoAttack`；进入一个分支后不再执行后续分支。`targets` 记录 worked／picked／attacked，去重只覆盖这三路，地面物品抛飞与 `formpile` 聚堆分支不在其内；多颗落点共享同一 `targets` 表；目标装备带 `junk` 标签时该次外部倍率置 0。 | [DoDamage](../../../raw/current/scripts/prefabs/junkball.lua#L339-L396)、[combat 配置](../../../raw/current/scripts/prefabs/junkball.lua#L583-L586)、[tuning](../../../raw/current/scripts/tuning.lua#L8167) |
| `lunarrift_terraformer_explosion` | `do_explosion` 对每个命中实体直接 `hit_entity.components.combat:GetAttacked(inst, 30, nil)`；FX 自身没有 combat 组件。 | 半径 `sqrt(2×TILE_SCALE²)/2`≈2.83：即半条地块对角线（直径等于 1 个地块的对角线），以爆心为圆心恰好覆盖 1 个地块的四角；必须 `_combat`，且 ONEOF 只取 `animal`／`character`／`monster`／`structure`，排除 `lunar_aligned`、`FX`、`flight`、`INLIMBO`、`playerghost` 等；不要求目标有 health。 | [标签与半径](../../../raw/current/scripts/prefabs/lunarrift_terraformer_explosion.lua#L6-L11)、[结算](../../../raw/current/scripts/prefabs/lunarrift_terraformer_explosion.lua#L13-L31)、[tuning](../../../raw/current/scripts/tuning.lua#L7630)、[TILE_SCALE](../../../raw/current/scripts/constants.lua#L17) |

### 组件、网络与资源

| 类型／对象 | 本族如何使用 | 端与生效条件 | 证据 |
|---|---|---|---|
| `组件：Light` | `houndstooth_proj` 在 Pristine 前装灯：falloff .6、intensity .6、radius .4、暖白色（237／237／209），`Enable(true)`。 | 各端构造都执行；实际可见只在非专服。 | [L240-L248](../../../raw/current/scripts/prefabs/houndstooth_blowpipe.lua#L240-L248) |
| `组件：weapon／planardamage／damagetypebonus` | 弹体自身承担物理、planar 与暗影类型加成；`has_damage_set` 使宿主武器的伤害不参与本次命中。 | 仅主模拟挂载。 | [L291-L310](../../../raw/current/scripts/prefabs/houndstooth_blowpipe.lua#L291-L310) |
| `组件：projectile` | 速度 25、无 homing、`OnThrown` 记录 `_target`、`OnPreHit` 生成命中火花、命中／落空都 Remove、命中距离 1.5、发射偏移 (2.5, .75, 2.5)、射程 30。 | 仅主模拟。 | [L300-L310](../../../raw/current/scripts/prefabs/houndstooth_blowpipe.lua#L300-L310) |
| `组件：combat` | `junkball_fall_fx` 的落点结算载体：默认伤害 30、range 1.5、`KeepTarget` 恒 false；实际攻击仍优先借用攻击者 combat。 | 仅主模拟。 | [L583-L586](../../../raw/current/scripts/prefabs/junkball.lua#L583-L586)、[L339-L340](../../../raw/current/scripts/prefabs/junkball.lua#L339-L340) |
| `组件：updatelooper` | `junkball_fx`／`junkball_fall_fx` 都用它：主模拟挂 wall update 推进位置，非专服挂每帧淡出与影子缩放。 | 位置更新 [L159](../../../raw/current/scripts/prefabs/junkball.lua#L159)／[L544](../../../raw/current/scripts/prefabs/junkball.lua#L544)，淡出 [L303](../../../raw/current/scripts/prefabs/junkball.lua#L303)／[L573](../../../raw/current/scripts/prefabs/junkball.lua#L573)。 | 同左 |
| `netvar：houndstooth_proj._target` | 弹体把目标实体同步给各端；客户端尾迹据此在离目标 1.5 内停止生成。 | 主模拟 `Projectile_OnThrown` 写，客户端只读 `_target:value()`。 | [L279](../../../raw/current/scripts/prefabs/houndstooth_blowpipe.lua#L279)、[L127-L146](../../../raw/current/scripts/prefabs/houndstooth_blowpipe.lua#L127-L146) |
| `父挂：portal` | 两个投影都由 portal 主模拟生成后 `SetParent` 到 portal；投影自身不带位置逻辑，随宿主变换。 | 主模拟建立，联网实体把父挂带到客户端；都无独立自删：裂隙有 `shadow_disappear` 退场动画，天体门没有对应调用；原生随父删除均未核验。 | [lunarrift L643](../../../raw/current/scripts/prefabs/lunarrift_portal.lua#L643)、[moonrock L405](../../../raw/current/scripts/prefabs/multiplayer_portal.lua#L405) |
| `标签与资源` | `impact`／两个投影／`junkball` 都带 `FX`；`junkball` 另带 `NOCLICK`／`junk`；`houndstooth_proj` 带 `weapon`／`projectile`／`NOCLICK`／`NOBLOCK`。资源：`impact.zip`、`blow_dart.zip`＋swap、`scrapball.zip`、`crystalblast_fx.zip`、`lunar_rift_portal.zip`＋small、`portal_moonrock.zip`＋`portal_stone.zip`。尾迹的 `lavaarena_blowdart_attacks` 未列入本 prefab 的 `assets`。 | 声明即核验范围；不推断资源实际存在或帧长。 | [impact assets](../../../raw/current/scripts/prefabs/impact.lua#L1-L4)、[houndstooth assets](../../../raw/current/scripts/prefabs/houndstooth_blowpipe.lua#L1-L5)、[junkball assets](../../../raw/current/scripts/prefabs/junkball.lua#L3-L6)、[terra assets](../../../raw/current/scripts/prefabs/lunarrift_terraformer_explosion.lua#L1-L4)、[lunarrift assets](../../../raw/current/scripts/prefabs/lunarrift_portal.lua#L1-L15)、[moonrock assets](../../../raw/current/scripts/prefabs/multiplayer_portal.lua#L22-L26) |
| `关联 Prefab` | `houndstooth_proj` 命中生成 `hitsparks_piercing_fx`；`junkball_fx` 依赖并生成 `junkball_fall_fx`，落点 fx 依赖 `junk_pile` 与 `splash_green_large`。 | 主模拟；命中前／落点后。 | [L148-L167](../../../raw/current/scripts/prefabs/houndstooth_blowpipe.lua#L148-L167)、[junkball 依赖](../../../raw/current/scripts/prefabs/junkball.lua#L8-L17)、[落点分支](../../../raw/current/scripts/prefabs/junkball.lua#L486-L528) |

### `impact` 复用点

`impact` 是纯定位贴片；当前快照全量反查有 15 处 `SpawnPrefab("impact")`，超过 8 个复用点。原版只关心“在哪个位置、朝向谁”，伤害与命中判定都在调用方。

| 用途 | 代表调用点 | 触发与差异 |
|---|---|---|
| 反弹伤害反馈 | [bernie_big L45-L56](../../../raw/current/scripts/prefabs/bernie_big.lua#L45-L56)、[hutch L150-L166](../../../raw/current/scripts/prefabs/hutch.lua#L150-L166) | `OnReflectDamage` 在攻击者的 `combat.hiteffectsymbol` 上 `AddFollower`；攻击者没有 combat 时退回世界坐标；再 `FacePoint` 自身。 |
| 投掷物命中 | [blowdart L31-L39](../../../raw/current/scripts/prefabs/blowdart.lua#L31-L39)、[boomerang L65-L77](../../../raw/current/scripts/prefabs/boomerang.lua#L65-L77) | `onhit`／`OnHit` 里贴到目标的 `hiteffectsymbol`，按弹体或攻击者位置 `FacePoint`；另有 gestalt_guard_evolved L494 同型。 |
| 近战／工作反馈 | [whip L62-L75](../../../raw/current/scripts/prefabs/whip.lua#L62-L75)、[bullkelp_root L40](../../../raw/current/scripts/prefabs/bullkelp_root.lua#L40) | 鞭子每次 `onattack`、海带根按腐败度概率折断时，在目标位置生成并按两点连线设旋转。 |
| 角色 SG 收招 | [SGwilson L5275](../../../raw/current/scripts/stategraphs/SGwilson.lua#L5275)、[SGshadowwaxwell L800](../../../raw/current/scripts/stategraphs/SGshadowwaxwell.lua#L800)、[SGwx78_possessedbody L1777](../../../raw/current/scripts/stategraphs/SGwx78_possessedbody.lua#L1777)、[gestalt_guard_evolved L494](../../../raw/current/scripts/prefabs/gestalt_guard_evolved.lua#L494) | `pickaxe_recoil`／`atk_recoil` 里取目标位置（含 `recoil_effect_offset`）只 `SetPosition`；全族 15 处中本表列 10 个代表调用点，其余 5 处同型。 |

## 典型接入

### 在主模拟复刻一次命中贴片

- 前提与选择：目标有 `combat` 组件、需要命中瞬间贴片；选择 `impact` 而不是 `houndstooth_proj` 或 `hitsparks_piercing_fx`，因为这里只要一次性画面，不需要伤害或穿刺火花。位置、朝向由调用方写入，`impact` 自身不结算伤害。
- 生效与撤销：主模拟 `SpawnPrefab` 后把壳贴到目标符号、按攻击者位置设朝向；非专服在一帧后各自建本地动画，`animover` 自删；主模拟壳 0.5 秒 Remove。不要把它当作持续跟随特效——本地代理没有父挂，宿主后续移动不归本工厂管。
- 原版依据：[blowdart L31-L39](../../../raw/current/scripts/prefabs/blowdart.lua#L31-L39) 与 [boomerang L65-L77](../../../raw/current/scripts/prefabs/boomerang.lua#L65-L77) 都是“贴目标位置”的写法，但朝向不同：blowdart 面向攻击者，boomerang 面向弹体所在方向；[impact 本地代理](../../../raw/current/scripts/prefabs/impact.lua#L6-L25) 只负责取位、播 `idle` 和 `animover` 自删。

```lua
local function PlayImpactFx(attacker, target)
    if target == nil or not target:IsValid() or target.components.combat == nil then
        return
    end

    local fx = SpawnPrefab("impact")
    if fx == nil then
        return
    end

    local follower = fx.entity:AddFollower()
    follower:FollowSymbol(target.GUID, target.components.combat.hiteffectsymbol, 0, 0, 0)

    if attacker ~= nil and attacker:IsValid() then
        fx:FacePoint(attacker.Transform:GetWorldPosition())
    end
end
```

- 调整与验证：替换点是 `SpawnPrefab` 的注册名与 `FollowSymbol` 的符号；伤害数值、命中判定仍由调用方的 combat／projectile 负责。代码只复刻定位，未在游戏内运行；语法检查见“验证记录”。父挂式投影的 `SetParent` 只解决位置跟随；两个投影都没有自删逻辑，且退场路径不同：裂隙仅在未休眠时给 shadow 播 `shadow_disappear` 并走 portal 的 animover／10 秒兜底，休眠时直接 Remove；天体门没有对应动画或显式删除。模组必须自行管理生命周期，不能把父挂当清理保证（原生随父删除结果未核验）。

## 常见陷阱与限制

- **陷阱：L728 拼写错误使 `lunarrift_portal_shadow` 的主模拟分支永不执行**：`TheWorld.ismasersim` 少一个 `t`，字段恒为 `nil`，`not nil` 恒 true，`portal_shadow_fn` 在 SetPristine 后立即返回；L732 的 `persists=false` 从未设置，函数也没有任何自身清理或阶段逻辑。投影的阶段缩放、重播与消失动画全在 portal 侧完成。修页面或写模组时不要把主模拟初始化补在这个分支里，先改回 `ismastersim`。[错误分支](../../../raw/current/scripts/prefabs/lunarrift_portal.lua#L727-L732)
- **边界：`impact` 是纯表现，不是命中判定**：工厂没有 weapon／combat／damage 调用；伤害在弹体、攻击者 combat 或调用方。壳的 0.5 秒 Remove 也不等于本地动画 0.5 秒：本地代理独立按 `animover` 回收。[壳清理](../../../raw/current/scripts/prefabs/impact.lua#L50-L51)、[本地清理](../../../raw/current/scripts/prefabs/impact.lua#L24)
- **边界：`impact` 本地代理不父挂**：`PlayImpactAnim` 只 `SetFromProxy(proxy.GUID)` 取壳变换并建独立实体，没有 `SetParent`；消费者给壳加 follower 只影响壳。若需要持续跟随宿主，改用父挂式投影或自行每帧更新；通用工厂页也把 `SetFromProxy` 记为创建时复制、跟随要靠父挂。[本地代理](../../../raw/current/scripts/prefabs/impact.lua#L14-L24)、[[reference/mechanisms/generic-fx-factory#常见陷阱与限制|通用 FX 工厂的跟随即挂接方式]]
- **误区：`houndstooth_proj` 的伤害来自吹箭本体的 weapon**：宿主只提供 `SetProjectile` 与射程，弹体在 `Projectile:Hit` 里因 `has_damage_set=true` 被当作 weapon，物理、planar 与暗影加成都在弹体自身；改宿主 `weapon:SetDamage` 不影响这次弹道伤害。[宿主选弹](../../../raw/current/scripts/prefabs/houndstooth_blowpipe.lua#L80-L87)、[弹体伤害](../../../raw/current/scripts/prefabs/houndstooth_blowpipe.lua#L291-L310)、[Hit 选择](../../../raw/current/scripts/components/projectile.lua#L238-L244)
- **边界：尾迹只在非专服**：`CreateTailFx`／`UpdateTail` 仅装在 `not TheNet:IsDedicated()` 的分支；每条尾迹是无 Network、无父挂的独立实体，播完 `animover` 自删，专服完全没有尾迹。尾迹使用的 `lavaarena_blowdart_attacks` 不在本 prefab 的 `assets` 中（同资源由 [blowdart_lava L7-L10](../../../raw/current/scripts/prefabs/blowdart_lava.lua#L7-L10)／[blowdart_lava2 L7-L10](../../../raw/current/scripts/prefabs/blowdart_lava2.lua#L7-L10) 声明），是否由外部预载保证未核验。[尾迹工厂](../../../raw/current/scripts/prefabs/houndstooth_blowpipe.lua#L100-L125)、[安装条件](../../../raw/current/scripts/prefabs/houndstooth_blowpipe.lua#L272-L277)、[assets](../../../raw/current/scripts/prefabs/houndstooth_blowpipe.lua#L1-L5)
- **边界：`junkball_fx` 不结算伤害，`junkball_fall_fx` 才结算**：第一段只计算方向、距离并在 `animover` 生成 6 颗落点；每颗落点在 0.5 秒后由 `UpdateFallPos` 触发 `DoDamage`。`DoDamage` 不只打伤害：每个目标按条件择一——可 work 目标 `Destroy`，否则非 intense 可摘目标 `Pick`，否则 `combat:CanTarget` 时 `DoAttack`，命中一个分支后不再进入后续分支；另有一段把半径内地面物品停用并抛飞，`formpile` 条件满足时还会删树桩、对 attacked 目标推 knockback 并生成 `junk_pile`。多颗落点共享 `targets` 表，但去重只覆盖 worked／picked／attacked，抛飞与聚堆等副作用不在其中；若落点 fx 在 0.5 秒前被移除，则不会结算。[第一段](../../../raw/current/scripts/prefabs/junkball.lua#L195-L278)、[落点时序](../../../raw/current/scripts/prefabs/junkball.lua#L470-L487)
- **边界：`lunarrift_terraformer_explosion` 是直接 `GetAttacked`，不走攻击者 `DoAttack`**：没有 combat 组件，`do_explosion` 按 `FindEntities` 过滤后直接给目标 30 点伤害，不经过攻击者的 `Combat:DoAttack`；但受击侧仍进入 `GetAttacked_Internal`，未 blocked 时照常推 `attacked`；只有该 combat 设置了 `onhitfn` 才执行回调，未设置不调用，所以不能概括为“没有事件链”。`lunar_aligned` 在 CANT_TAGS 中，月之阵营目标完全不吃这次爆炸，`FX`／`flight`／`INLIMBO`／`playerghost` 等也一样。半径取半条地块对角线，以爆心为圆心覆盖 1 个地块的四角。[过滤与结算](../../../raw/current/scripts/prefabs/lunarrift_terraformer_explosion.lua#L6-L31)、[GetAttacked](../../../raw/current/scripts/components/combat.lua#L570-L577)、[事件推送与 onhitfn](../../../raw/current/scripts/components/combat.lua#L715-L720)
- **边界：`multiplayer_portal_moonrock_fx` 只在非洞穴主模拟生成**：`inst.fx = not TheWorld:HasTag("cave") and SpawnPrefab(...) or nil`，洞穴端没有 fx；非洞穴也只在 master postinit 里 `SetParent`。它没有 animover／定时 Remove，`persists=false` 只表示不入存档，不负责清理；手动生成后既无父挂也无显式删除入口，会留场；即使父挂，随父删除也未核验，应自行管理生命周期。[生成条件](../../../raw/current/scripts/prefabs/multiplayer_portal.lua#L327-L330)、[父挂](../../../raw/current/scripts/prefabs/multiplayer_portal.lua#L404-L405)、[清理配置](../../../raw/current/scripts/prefabs/multiplayer_portal.lua#L451)
- **名称边界**：七个注册里只有 `lunarrift_terraformer_explosion` 的“月形爆裂”是 NAMES 官方名；其余中文都是效果描述。不要把宿主名“嚎弹炮”“裂隙”“天体传送门”当成对应 FX 的官方名。

## 验证记录

- 源码审阅：2026-11-04，Steam buildid 26335394（`raw/current/BUILD.txt`）。逐项核验六个源文件的七个注册：工厂与端分界、生成点（在 `raw/current/scripts` 全量反查七个注册名与 `SpawnPrefab("impact")`）、父挂、清理路径、伤害归属；另核验 `components/projectile.lua` 的 `has_damage_set`／`Hit`、`components/weapon.lua` 的 `GetDamage`／`LaunchProjectile`、`tuning.lua` 与 `constants.lua` 的数值、`languages/chinese_s.po` 的 NAMES 键，以及 `rift_terraformer.lua`、`junk_pile_big.lua`、`SGdaywalker2.lua` 与 impact 的 15 个调用点。未覆盖：动画实际帧长、客户端视觉演出、存档／读档恢复（尤其 L728 对 persists 的影响）、宿主玩法、资源包实际可用性。
- 游戏运行：未运行。
- 结构检查：`validate_page.py` 通过；`validate_source_links.py --strict` 通过；Lua 示例 `parser_probe --syntax-only` 通过（结果见交付报告）。
- 分档说明：本页覆盖纯表现网络壳、弹道伤害、combat 落点、直接 `GetAttacked`、父挂投影五类协作与多文件生成点，需要整体保留，故用 `size_class: complex`；未因源码文件总量扩额。
- 第 17 轮补正（2026-11-07）：补 `blowdart_lava.lua`／`blowdart_lava2.lua` 来源与锚点；投影清理改为“无独立自删、原生随父删除未核验”；落点补 workable Destroy→pickable Pick→combat 顺序、地面物品抛飞与聚堆分支；直伤改为“不走 DoAttack，但受击侧仍推 attacked／执行 onhitfn”。复验再收紧：区分裂隙未休眠的 `shadow_disappear`／portal 退场与休眠直接 Remove，并说明天体门无对应退场调用；`junkball_fall_fx` 改为“每个目标按条件择一破坏／采摘／攻击”；`onhitfn` 注明仅设置后调用；补 `components/combat.lua` 来源。

相关页面：[[reference/prefabs/hitsparks_fx|命中火花特效（hitsparks_fx）]]（`houndstooth_proj` 的命中火花来源）。

返回 [[reference/prefabs/fx-combat-index|攻击、弹道与落点]]。
