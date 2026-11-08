---
title: "虚布武器持握层与回旋镖投射物（voidcloth_boomerang_fx 等 4 注册）"
created: "2026-11-05"
updated: "2026-11-09"
type: prefab
summary: "虚布三件武器的持握跟随光层与回旋镖混合投射物：三装备共用『联网 FX 骨架＋客户端本地帧层』的持握层架构（帧窗口与挂接时机各异），阴郁回旋镖的持握外观全由 FX 帧层绘制（本体不 override swap_object）；投射物自带武器／平面伤害并随飞行尺寸缩放。含虚布套装门禁、伞面未注册 wave/dome 内部 FX 辨析与皮肤映射核查。"
aliases: ["voidcloth_boomerang_fx", "voidcloth_boomerang_proj", "voidcloth_scythe_fx", "voidcloth_umbrella_fx", "阴郁回旋镖", "暗影收割者", "暗影伞", "虚布持握光效", "虚布回旋镖投射物", "虚布套装加成"]
tags: [prefab, fx, item, equipment, server-client]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/voidcloth_boomerang.lua", "prefabs/voidcloth_scythe.lua", "prefabs/voidcloth_umbrella.lua", "prefabs/boomerang.lua", "components/projectile.lua", "fx.lua", "tuning.lua", "prefabskins.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 虚布武器持握层与回旋镖投射物（voidcloth_boomerang_fx 等 4 注册）

## 快速判断

- 职责与适用：4 个注册包括三件虚布武器的**持握跟随光层**（`voidcloth_boomerang_fx`／`voidcloth_scythe_fx`／`voidcloth_umbrella_fx`）与**联网混合投射物** `voidcloth_boomerang_proj`。三种持握层共用联网 FX 骨架和客户端本地帧层，帧窗与挂接时机各异。投射物自带武器与平面伤害，伤害随飞行尺寸缩放，由投射物实体参与命中结算。
- 优先使用：查"手里的武器为什么发暗影光、怎么跟着挥动帧走"用三个 `_fx`；查"回旋镖丢出去伤害谁结算、怎么变大变小、怎么飞回来"用 proj；查"魔镰为什么说话、收割范围"见 [[reference/prefabs/voidcloth_scythe#说话系统与网络代理voidclothscytheclassified|暗影收割者本体]]。
- 重要副作用：阴郁回旋镖**持握外观不在玩家 swap_object 上 override**——其 `OnEquip` 只做 ClearOverride，可见刀身全部由 boomerang_fx 帧层绘制（对照普通回旋镖走 OverrideSymbol，[boomerang.lua:18](../../../raw/current/scripts/prefabs/boomerang.lua#L18)）；三个 `_fx` 都无自删；proj 由返回收集／越过目标／休眠三条路径移除。
- 修改边界：只覆盖 4 注册的工厂、生成点、挂接与清理及 proj 的伤害归属；虚布套装数值、魔镰采集与 AoE 玩法细节、暗影伞雨穹玩法不展开；`voidcloth_boomerang_launch_fx`／`voidcloth_boomerang_impact_fx` 已有独立页（见相关链接），不重复覆盖。
- 名称：三件装备官方名 `VOIDCLOTH_BOOMERANG`＝"阴郁回旋镖"（[po](../../../raw/current/scripts/languages/chinese_s.po#L351349-L351352)）、`VOIDCLOTH_SCYTHE`＝"暗影收割者"（[po](../../../raw/current/scripts/languages/chinese_s.po#L351359-L351362)）、`VOIDCLOTH_UMBRELLA`＝"暗影伞"（[po](../../../raw/current/scripts/languages/chinese_s.po#L351364-L351367)）；其余四个附属 FX／弹体注册名无独立 `STRINGS.NAMES.*` 键（`voidcloth_scythe` 本体有官方名），标题中文为已核验效果描述，非官方译名。

## 注册与构造

| 注册名 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `voidcloth_boomerang_fx` | `FollowSymbolFxFn`：Transform／AnimState／Follower／Network、标签 `FX`；bank=build `boomerang_voidcloth`、循环播 `swap_loop_f3`（帧 3 专供漂浮）、LightOverride 0.1；highlightchild＋colouraddersync；`equiptoggle` net_bool（事件 equiptoggledirty）；主模拟暴露 `ToggleEquipped`、`persists=false` | **联网骨架**：客户端收到 dirty 后按 FX_DEFS 补生本地帧层（f1：帧窗 0–2）；宿主武器工厂生成一次并随机同步帧。[工厂](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L638-L670)、[注册](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L676) |
| `voidcloth_boomerang_proj` | `ProjectileFn`：Transform／AnimState／SoundEmitter／Network、MakeProjectilePhysics；同 bank/build、循环播 `projectile`、初始缩放 0.4、LightOverride 0.1＋lightning 符号 0.5、blade 符号 alpha 0.8、泛光 anim.ksh；标签 weapon／projectile／NOCLICK／NOBLOCK／shadow_item；主模拟 `persists=false`、scalingdata（0.4→1）、weapon／planardamage／damagetypebonus(lunar 1.25)／updatelooper／projectile(speed 15, range 20, has_damage_set) | **混合投射物**：伤害组件在弹体上；飞行时每帧按尺寸重设伤害，返回阶段加速并朝投掷者。[工厂](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L477-L550)、[注册](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L677) |
| `voidcloth_scythe_fx` | `FollowSymbolFxFn`：同 boomerang_fx 结构，bank=build `scythe_voidcloth`、循环播 `swap_loop_3`（帧 3 供漂浮）；本地 FX_DEFS 四段：`swap_loop_1`（0–2）、`swap_loop_6`（5 起）、`swap_loop_7`（6 起）、`swap_loop_8`（7 起） | 与 boomerang_fx 同骨架；本地帧层比回旋镖多三段（刀身更长的逐帧光效），本地帧层无 LightOverride。[工厂](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L567-L597)、[FX_DEFS](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L483-L490)、[注册](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L601) |
| `voidcloth_umbrella_fx` | `FollowSymbolFxFn`：Transform／AnimState／Follower／Network、标签 `FX`；bank=build `umbrella_voidcloth`、循环播 `swap_loop1`、lightning 符号 LightOverride 1；highlightchild＋colouraddersync；主模拟暴露 `AttachToOwner`、`persists=false`；客户端 `OnEntityReplicated` 补生本地帧层（帧窗 5–8） | **本体直挂型**：联网层自己就是帧 0–2 层（`AttachToOwner` 内 FollowSymbol 帧窗 0–2），无 equiptoggle netvar，生命周期完全由伞的 OnEquip/OnUnequip 管理。[工厂](../../../raw/current/scripts/prefabs/voidcloth_umbrella.lua#L493-L523)、[注册](../../../raw/current/scripts/prefabs/voidcloth_umbrella.lua#L527) |

## 关联机制导航

### 持握层：联网骨架＋本地帧层的分工

| 环节 | 行为 | 证据 |
|---|---|---|
| 生成 | boomerang／scythe 在本体工厂 `SpawnPrefab` 一次：随机帧写入本体与 fx 的 AnimState（网络层浮空时需要同帧）；umbrella 在每次 `OnEquip` 先 `Remove` 旧 fx 再新生成 | [boomerang L289-297](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L289-L297)、[scythe L440-447](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L440-L447)、[umbrella OnEquip](../../../raw/current/scripts/prefabs/voidcloth_umbrella.lua#L32-L36) |
| 装备挂接 | boomerang/scythe：`SetFxOwner(owner)` 把 fx 父挂持有者、`FollowSymbol(swap_object, frame_begin=2)`、highlightchild 认主、colouradder AttachChild、`ToggleEquipped(true)`；umbrella：`AttachToOwner` 父挂＋FollowSymbol 帧窗 0–2 | [SetFxOwner](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L93-L116)、[FxAttachToOwner](../../../raw/current/scripts/prefabs/voidcloth_umbrella.lua#L479-L491) |
| 本地帧层 | boomerang/scythe 靠 `equiptoggle` net_bool 的 equiptoggledirty 事件驱动客户端 `FxOnEquipToggle`：清旧后按 FX_DEFS 生成非联网 `CreateFxFollowFrame` 实体（各帧窗跟随 swap_object、与宿主同帧、highlightchild），colouraddersync 转发染色，`OnRemoveEntity=FxRemoveAll`；umbrella 客户端在 fx 复制完成时 `FxOnEntityReplicated` 生成单个帧 5–8 本地层 | [boomerang FxOnEquipToggle](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L596-L626)、[scythe 同构](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L525-L555)、[umbrella FxOnEntityReplicated](../../../raw/current/scripts/prefabs/voidcloth_umbrella.lua#L467-L477) |
| 卸下／漂浮 | boomerang/scythe `SetFxOwner(nil)`：fx 改挂武器本体、跟随自身 `swap_spear` 符号（供漂浮渲染）、`ToggleEquipped(false)`（dirty 让客户端清本地帧层）；umbrella `OnUnequip` 直接 `inst._fx:Remove()` | [SetFxOwner else 分支](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L110-L115)、[umbrella OnUnequip](../../../raw/current/scripts/prefabs/voidcloth_umbrella.lua#L53-L56) |

### 回旋镖投射物：抛掷、返回与伤害归属

| 环节 | 行为 | 证据 |
|---|---|---|
| 抛掷 | 武器 `weapon:SetProjectile("voidcloth_boomerang_proj")`（充能时才设回，放电由 OnDischarged 清除）；`Projectile_OnThrown` 播 throw_lp 循环声、登记进本体 `_projectiles`、调 `weapon:OnAttack`、按套装加 planar 加成，并在攻击者身前朝目标方向生成 launch_fx | [OnCharged](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L207-L210)、[OnThrown](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L442-L468) |
| 伤害归属 | 投射物 `has_damage_set=true`：命中时 projectile 组件以**弹体自身**为 weapon 调 `attacker.components.combat:DoAttack(target, 弹体)`——武器／平面伤害都读弹体组件（初值 min 5，随尺寸 Lerp 到 max），对月盟加成 1.25 也在弹体 damagetypebonus 上；命中表现只播弹体动画，adrenaline／击杀归属走攻击者 | [projectile 结算](../../../raw/current/scripts/components/projectile.lua#L237-L264)、[弹体伤害组件](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L526-L533) |
| 尺寸-伤害耦合 | 去程 0.4→1 线性放大（totaltime=攻击距离/初速=10/15 秒）；`OnUpdateFn` 每帧 `Remap(scale, 0.4, 1, 0, 1)` 后 `SetDamage`／`SetBaseDamage`（min 5 → max wilson_attack×0.8≈27.2），套装时伤害乘 1.1 | [OnUpdateFn](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L418-L439)、[调值](../../../raw/current/scripts/tuning.lua#L8441-L8451) |
| 返回与收集 | OnHit／OnMiss 都调 `ReturnToThrower`：重置 scalingdata（当前尺寸→0.4）、解除边界碰撞、MotorVel 7.5 起步、随时间加速（≥1 倍乘 currenttime/3）；接近投掷者 1 格内、投掷者失效/入 limbo、或越过目标（点积为负）即 `Remove`；移除回调从 `_projectiles` 注销并重算充能（满员 Discharge 999999，否则回满） | [ReturnToThrower](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L347-L359)、[OnHit/OnMiss](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L361-L380)、[计数与充能](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L214-L220) |
| 兜底 | `OnEntitySleep` 停止投射并自删；同时在场弹体上限 3（套装 4） | [OnEntitySleep](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L470-L473)、[上限调值](../../../raw/current/scripts/tuning.lua#L8448-L8451) |

### 虚布套装门禁（voidclothhat）

`SetBuffOwner`（boomerang 为 `inst._onownerequip` 回调组，scythe 直调同构函数）：持有者装备/卸下 `voidclothhat`（官方名"虚空风帽"，[po](../../../raw/current/scripts/languages/chinese_s.po#L351344-L351347)）时置 `_bonusenabled`，按 prefab 名判定不看皮肤。boomerang 套装把同时在场弹体上限 3→4（[SetBuffEnabled](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L38-L47)）；proj 生成时复制 `_bonusenabled` 加 5 点平面加成、伤害乘 1.1（[L446-L450](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L446-L450)、[调值](../../../raw/current/scripts/tuning.lua#L7819-L7820)）；scythe 套装把武器伤害换成 38×1.1 并加同一平面加成（[SetBuffEnabled](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L54-L70)）。

### 暗影伞 wave／dome 未注册内部 FX 辨析（台账批 9 复核属实）

伞开启雨穹时 `SERVER_TriggerFX` 推 `triggerfx` net_event 并在非专服本地直接演（[L229-L234](../../../raw/current/scripts/prefabs/voidcloth_umbrella.lua#L229-L234)）；客户端延迟监听该事件后 `CLIENT_TriggerFX` 在伞位置生成两个**未注册**的本地 FX（[L221-L227](../../../raw/current/scripts/prefabs/voidcloth_umbrella.lua#L221-L227)）：`CreateWaveFX`（`barrier_rim` 贴地背景层，scalemult 0.75）与 `CreateDomeFX`（`barrier_dome`、FinalOffset 7、播 barrier_activate 声）。二者共用 `WaveFxOnUpdate`：0.5 秒内按平方包络淡出，同时从约 `0.3×scalemult` 向外扩张到 `2×scalemult` 后 `Remove`，`persists=false`、非联网、无注册名（[WaveFxOnUpdate](../../../raw/current/scripts/prefabs/voidcloth_umbrella.lua#L154-L167)）。全库 grep 无 `voidcloth_umbrella_wave_fx`／`_dome_fx` 注册——它们是工厂函数直接 CreateEntity 的内部表现，模组无法 SpawnPrefab 引用。

### 皮肤映射

`prefabskins.lua` 全文检索：虚布主题只有 `armor_voidcloth` 与 `voidclothhat` 有皮肤（ribbon/spooky 两款，[L2041-L2044](../../../raw/current/scripts/prefabskins.lua#L2041-L2044)）；**回旋镖／魔镰／暗影伞三装备本体无任何皮肤注册**（skins_defs_data.lua 中相关条目仅为图鉴 scrapbook 键）。魔镰与暗影伞的 OnEquip 有 `GetSkinBuild` 分支；回旋镖的 OnEquip 没有该分支，只清 `swap_object`，外观全部由帧层承担。三装备当前 build 都无可用皮肤、恒走默认分支；voidclothhat 皮肤不影响套装判定（按 prefab 名）。

### 清理路径

| 注册 | 清理方式 | 边界与证据 |
|---|---|---|
| `voidcloth_boomerang_fx` | 无自删：装备期 `SetParent` 跟随持有者，卸下改挂武器本体继续存在；本体损坏时 `fx:Hide()`、修复 `Show()`；fx 自身 `OnRemoveEntity` 清本地帧层 | 没有显式 Remove；装备期随持有者或武器移除时的引擎回收未核验。本地帧层由 FxRemoveAll 统一清（[L583-L588](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L583-L588)）。 |
| `voidcloth_boomerang_proj` | 三条主动移除：回收接近（<1 格）、投掷者失效、越过目标；休眠兜底 Stop+Remove；移除回调同步注销计数并重算充能 | `persists=false` 不存档；存档时在场弹体直接消失（武器充能状态另存）。 |
| `voidcloth_scythe`／`voidcloth_scythe_fx` | 本体为持久物品（库存实体）；fx 同 boomerang_fx 模式，损坏 Hide、修复 Show | scythe 的 classified 子实体由 OnRemoveEntity 双端拆挂（[L25-L36](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L25-L36)）。 |
| `voidcloth_umbrella_fx` | 无自删：每次 OnEquip 先删旧建新，OnUnequip 显式 Remove | `OnEquipToModel` 只停燃料不删 fx——与 OnUnequip 不对称（[L63-L69](../../../raw/current/scripts/prefabs/voidcloth_umbrella.lua#L63-L69)）；实际进该路径前是否必经 OnUnequip 未逐路径核验（见存疑）。 |
| wave／dome 内部 FX | 0.5 秒动画包络后 `Remove`（updatelooper 驱动） | 非注册实体，无网络，不需要清理登记。 |

## 典型接入

### 复刻"随持握动画帧点亮"的装备光层

- 前提与选择：需要光效贴着持握武器逐帧移动时，用 boomerang/scythe 模式——一个联网 FX 骨架负责同步装备状态（net_bool）与泛光/染色，客户端按帧窗补生非联网跟随层；每层 `FollowSymbol` 传帧窗参数（第 8/9 参 frame_begin/frame_end，引擎侧方法，参数序按本文件调用与 FX_DEFS 对照）。
- 生效与撤销：骨架由武器工厂生成一次；装备/卸下只切 `equiptoggle`，本地层由 dirty 事件增删——不要在本地层里写自删，生命周期归骨架。

```lua
local fx = SpawnPrefab("voidcloth_boomerang_fx") -- 骨架（联网）
fx.entity:SetParent(owner.entity)
fx.Follower:FollowSymbol(owner.GUID, "swap_object", nil, nil, nil, true, nil, 2)
fx:ToggleEquipped(true) -- 服务端置 net_bool，客户端 dirty 后自建本地帧层
```

- 调整与验证：替换点是注册名、bank/build 与 FX_DEFS 帧窗表；umbrella 型（本体直挂＋OnEntityReplicated）适合层数少、不需要按装备态增删的场景。示例只做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **误区：阴郁回旋镖拿在手里靠 swap_object 换装。** 其 OnEquip 只 `ClearOverrideSymbol("swap_object")`、从不设置；持握刀身由 boomerang_fx 的 `swap_loop_f1`（帧 0–2，本地）与 `swap_loop_f3`（帧 2 起，联网）两层按玩家动画帧渲染（[OnEquip](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L16-L24)）。按"武器换装"思路改它的外观会落空。
- **边界：本地帧层只存在于客户端与非专服主模拟。** `FxOnEquipToggle` 由 equiptoggledirty 事件驱动且仅非专服执行（[L632-L634](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L628-L636)）；专服不建本地帧层，但骨架仍有 AnimState／Follower／highlightchild／colouraddersync 等组件，不是 Transform/Network 空壳。
- **边界：proj 的伤害随尺寸缩放是每帧重设，不是一次性快照。** 命中瞬间的伤害由当时尺寸决定（返回阶段会一路缩回 0.4→伤害回落到 min）；想固定伤害需改 OnUpdateFn 而非初始组件值（[L431-L439](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L431-L439)）。
- **边界：弹体上限是"同时在场的 proj 数"。** 计数在武器本体的 `_projectiles` 表（移除回调注销），满员后 rechargeable 被灌 999999 放电（[L214-L220](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L214-L220)）——OnPreLoad 特意**清理** rechargeable 存档（`data.rechargeable = nil`，源码注释 "Clean up rechargeable save data"，[L224-L229](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L224-L229)），读档恒为满充、放电状态不跨存档。
- **边界：三个 `_fx` 挂接语义不同，不能互换复用。** boomerang/scythe fx 是"骨架＋net_bool 分发"，umbrella fx 是"本体直挂＋复制回调"；umbrella fx 没有 `ToggleEquipped`，boomerang fx 没有 `AttachToOwner`。
- **边界：伞的 wave/dome 不是可引用注册。** 二者是 `CreateEntity` 直建的内部表现，且只在伞于场景内开穹时演出（收进物品/休眠走 turnon 的 asleep 分支不生成，[L258-L267](../../../raw/current/scripts/prefabs/voidcloth_umbrella.lua#L258-L267)）。
- **误区：暗影收割者的说话是 talker 直接出字。** 文本经 `voidcloth_scythe_classified`（独立注册，非 FX）按持有者 `player_shadow_aligned` 标签定向发送，talker 只负责本地渲染；伍迪砍木彩蛋按 30% 概率换 lucy 台词（[SayRandomLine](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L143-L157)）。

## 验证记录

- 源码审阅：2026-11-05，steam-buildid:26335394（实读 `raw/current/BUILD.txt`）。完整读取 `prefabs/voidcloth_boomerang.lua`、`prefabs/voidcloth_scythe.lua`、`prefabs/voidcloth_umbrella.lua` 全文，按需读取 `components/projectile.lua`（OnHit 结算段）、`components/follower.lua`（确认 FollowSymbol 为引擎侧方法，脚本内无定义）、`prefabs/boomerang.lua`（普通回旋镖对照）、`tuning.lua` 三组调值、`prefabskins.lua`／`skin_assets.lua`／`skins_defs_data.lua` 皮肤检索。对 5 个注册名以完整名在 `raw/current/scripts` 全域反查：`voidcloth_boomerang_fx` 命中注册（voidcloth_boomerang.lua:676）、依赖（:8）、生成（:294）——**生成点唯一**；`voidcloth_boomerang_proj` 命中注册（:677）、依赖（:9）、SetProjectile（:209/:312）——由 weapon 组件抛出，无字面 SpawnPrefab；`voidcloth_scythe` 命中注册（voidcloth_scythe.lua:600）、依赖（prefablist.lua:1358）、配方（recipes.lua:869）；`voidcloth_scythe_fx` 命中注册（:601）、依赖（:8）、生成（:444）——**生成点唯一**；`voidcloth_umbrella_fx` 命中注册（voidcloth_umbrella.lua:527）、依赖（:8）、生成（:35，OnEquip 内，每次装备重建）。清单外发现：`voidcloth_boomerang_launch_fx`／`voidcloth_boomerang_impact_fx` 注册于 [fx.lua L3657/L3670](../../../raw/current/scripts/fx.lua#L3657-L3677)、由弹体 OnThrown/OnHit 生成，已有独立页不重复覆盖；`voidcloth_scythe_classified` 为非 FX 注册（voidcloth_scythe_classified.lua:142）；伞的 wave/dome 为未注册内部工厂（全库无注册名）。未运行游戏。
- 名称核验：`languages/chinese_s.po` 定向检索——三装备 NAMES 键译文见"快速判断"；`voidcloth_scythe` 本体有官方名"暗影收割者"，其余四个附属 FX／弹体注册名无独立 `STRINGS.NAMES.*` 键；`VOIDCLOTHHAT`＝"虚空风帽"（锚点见套装节）。
- 分档说明：`size_class: standard`——4 个注册分为持握层与混合投射物，三种持握层共用骨架。
- 未覆盖：`voidcloth_boomerang_launch_fx`／`voidcloth_boomerang_impact_fx` 工厂细节（fx.lua 通用表代理，见其独立页）、`voidcloth_scythe_classified` 完整机制、魔镰采集/暗影 AoE 的技能树门禁全链（仅记 `wortox_allegiance_shadow` 判定与 `VOIDCLOTHSCYTHE_AOE_RANGE`=2.5）、伞的 raindome／fueled 数值玩法、动画包帧数与观感、`FollowSymbol` 帧窗参数的引擎实现；未运行游戏。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 结果见交付记录（0 error 0 warning 后入库）。
- 游戏运行：未运行；`validation` 保持 `source-reviewed`。

- 第 15 轮补正（2026-11-07）：分开 `SetParent` 与 `AddChild`、补官方名／`GetSkinBuild`／wave-dome 淡出／专服组件边界；同步维护日期。

## 相关链接

- [[reference/prefabs/voidcloth_boomerang_launch_fx|虚布回旋镖发射与命中特效]]（弹体 OnThrown/OnHit 生成的两个 fx.lua 表代理）
- [[reference/prefabs/hitsparks_fx|命中火花特效]]（魔镰 OnAttack 恒定染黑复用）
- [[reference/prefabs/armor_lunarplant_glow_fx|装备与帽饰跟随光效]]（`armor_voidcloth_fx` 护甲光效，同主题虚布装备）
- 返回 [[reference/prefabs/fx-combat-index|攻击、弹道与落点]]（装备/虚布武器）
- 2026-11-08 build 26474873：本页随版本迁移升版（锚点迁移 3 处），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-12）。
