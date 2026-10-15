---
title: "冲击环、爆点与碎块表现特效（groundpoundring_fx 等）"
created: "2026-11-04"
updated: "2026-11-04"
type: prefab
summary: "六个手写工厂的冲击／爆点／碎块表现：震地中心环（可缩放＋net_bool 跳帧）、蜻蜓龙火环、地刺爆裂、炸弹爆炸、震地碎块与眼睛炮塔命中音；全为纯表现，端与清理各不相同。"
aliases: ["groundpoundring_fx", "firering_fx", "charliearena_spike_explode_fx", "ground_chunks_breaking", "fused_shadeling_bomb_death_fx", "eye_charge_hit", "震地中心环", "冲击环", "火环", "爆点特效", "碎块特效", "命中音特效"]
tags: [prefab, fx, server-client]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/charliearena_spike.lua", "prefabs/fireringfx.lua", "prefabs/groundpoundringfx.lua", "prefabs/ground_chunks_breaking.lua", "prefabs/fused_shadeling_bomb.lua", "prefabs/eye_charge.lua", "components/groundpounder.lua", "prefabs/dragonfly.lua", "prefabs/willow_ember.lua", "prefabs/bearger.lua", "prefabs/wurt_terraform_item.lua", "components/quaker.lua", "prefabs/moon_device.lua", "prefabs/ruins_cavein_obstacle.lua", "prefabs/saddle.lua", "prefabs/shadowmeteor.lua", "prefabs/eyeturret.lua", "languages/chinese_s.po"]
size_class: complex
reliability: verified
validation: source-reviewed
status: active
---

# 冲击环、爆点与碎块表现特效（groundpoundring_fx 等）

## 快速判断

- 职责与适用：六个手写 `fn` 的表现 Prefab，覆盖震地中心环、火环、地刺爆裂、炸弹爆点、地面碎块与眼睛炮塔命中音；都不是 `fx.lua` 通用表条目，公共工厂协议见 [[reference/mechanisms/generic-fx-factory|通用 FX 工厂]]。震地落点尘土和蜻蜓龙环点火溅分别是 [[reference/prefabs/groundpound_fx|groundpound_fx]] 与 [[reference/prefabs/firesplash_fx|firesplash_fx]] 的职责，本页只链不重复。
- 优先使用：查「震地中心环／火焰环／碎块／地刺爆炸／炸弹爆炸／命中音」从本页的注册与构造、关联机制导航进入；只查震地组件接口去 [[reference/components/groundpounder|groundpounder]]。
- 重要副作用：六个注册都没有伤害、点燃或生命字段，生成它们不等于结算；唯一的网络状态变量是 `groundpoundring_fx` 的 `net_bool` 跳帧，其余差异在网络壳与本地代理或联网实体动画的寿命。
- 修改边界：改表现改对应文件的 `fn`；清理由各文件自带的 `animover` 或定时器完成，没有统一工厂。本体 `charliearena_spike`、`fused_shadeling_bomb`、`eye_charge` 是玩法或混合载体，不在本页展开。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `charliearena_spike_explode_fx` | `fxfn`：Transform＋AnimState＋SoundEmitter＋Network → bank／build `charliearena_spike`、anim `explode`、三个符号灯覆盖 → SetPristine → 主模拟分支。[fxfn](../../../raw/current/scripts/prefabs/charliearena_spike.lua#L310-L340) [注册](../../../raw/current/scripts/prefabs/charliearena_spike.lua#L346) | 联网实体内置动画，专服也建实体；FX／NOCLICK；主模拟 `OnEntityWake` 播一次 `rifts2/shrouden/pillar_explode`，`animover` 自删、无定时器。[唤醒音](../../../raw/current/scripts/prefabs/charliearena_spike.lua#L305-L308) [接线与清理](../../../raw/current/scripts/prefabs/charliearena_spike.lua#L334-L337) |
| `firering_fx` | 网络壳：Transform＋Network → 非专服 `DoTaskInTime(0, PlayRingAnim)` → SetPristine → 主模拟 `persists=false`＋3 秒定时删。[壳](../../../raw/current/scripts/prefabs/fireringfx.lua#L33-L58) [注册](../../../raw/current/scripts/prefabs/fireringfx.lua#L60) | 代理 bank／build `dragonfly_ring_fx`、anim `idle`、FinalOffset 3、OnGround、背景层、bloom；本地 `animover` 自删；无缩放／跳帧接口。[代理](../../../raw/current/scripts/prefabs/fireringfx.lua#L6-L31) [定时](../../../raw/current/scripts/prefabs/fireringfx.lua#L55) |
| `groundpoundring_fx` | 网络壳：Transform＋Network → 非专服一帧后 `PlayRingAnim`；`net_bool("groundpoundring_fx._fastforward")` 在 SetPristine 前创建 → 主模拟暴露 `FastForward`＋3 秒定时删。[壳](../../../raw/current/scripts/prefabs/groundpoundringfx.lua#L39-L69) [net_bool](../../../raw/current/scripts/prefabs/groundpoundringfx.lua#L54) | 代理 bank／build `bearger_ring_fx`、anim `idle`、FinalOffset 3、OnGround、背景层；创建时读 `net_bool`，为真则 `SetFrame(5)`（只读一次，无 dirty 回调）；本地 animover 自删。[代理与跳帧](../../../raw/current/scripts/prefabs/groundpoundringfx.lua#L6-L33) [定时](../../../raw/current/scripts/prefabs/groundpoundringfx.lua#L64) |
| `ground_chunks_breaking` | 网络壳：Transform＋Network → 非专服一帧后 `PlayChunksAnim` → SetPristine → 主模拟 1 秒定时删。[壳](../../../raw/current/scripts/prefabs/ground_chunks_breaking.lua#L30-L55) [注册](../../../raw/current/scripts/prefabs/ground_chunks_breaking.lua#L57) | 代理 bank `ground_breaking`／build `ground_chunks_breaking`、anim `idle`、FinalOffset 3，并播 `dontstarve/common/stone_drop`；本地 animover 自删。[代理](../../../raw/current/scripts/prefabs/ground_chunks_breaking.lua#L6-L28) [定时](../../../raw/current/scripts/prefabs/ground_chunks_breaking.lua#L52) |
| `fused_shadeling_bomb_death_fx` | `death_fx_fn`：Transform＋AnimState＋SoundEmitter＋Network → bank／build `fused_shadeling_bomb`、anim `death_fx` → SetPristine → 主模拟分支。[工厂](../../../raw/current/scripts/prefabs/fused_shadeling_bomb.lua#L295-L318) [注册](../../../raw/current/scripts/prefabs/fused_shadeling_bomb.lua#L451) | 联网实体内置动画；未加 FX 标签（同族其余均加）；主模拟 `OnEntityWake` 播一次 `rifts2/parasitic_shadeling/dreadmite_explode`，`animover` 自删、无定时器。[唤醒音](../../../raw/current/scripts/prefabs/fused_shadeling_bomb.lua#L290-L293) [清理](../../../raw/current/scripts/prefabs/fused_shadeling_bomb.lua#L315) |
| `eye_charge_hit` | `hit_fn`：Transform＋Network（无 AnimState／SoundEmitter）→ 非专服一帧后本地声音代理 → AddTag FX → SetPristine → 主模拟 0.5 秒定时删。[工厂](../../../raw/current/scripts/prefabs/eye_charge.lua#L74-L98) [注册](../../../raw/current/scripts/prefabs/eye_charge.lua#L101) | 无视觉；本地代理只绑位置并播 `dontstarve/creatures/eyeballturret/shotexplo` 后立即 `Remove`；主壳 0.5 秒自删。[本地音](../../../raw/current/scripts/prefabs/eye_charge.lua#L59-L72) [定时](../../../raw/current/scripts/prefabs/eye_charge.lua#L95) |

范围辨析：`charliearena_spike`（官方名「绝望石矿苗」）是可工作／战斗本体，`fused_shadeling_bomb`（官方名「绝望螨」）是带 SG／timer 的怪物本体，`eye_charge` 是 `eyeturret`（官方名「眼睛炮塔」）隐藏武器发射的玩法投射物；三者只在此说明关系，行为不写入本页。[本体注册](../../../raw/current/scripts/prefabs/charliearena_spike.lua#L345) [炸弹本体](../../../raw/current/scripts/prefabs/fused_shadeling_bomb.lua#L450) [眼弹注册](../../../raw/current/scripts/prefabs/eye_charge.lua#L100) [炮塔武器](../../../raw/current/scripts/prefabs/eyeturret.lua#L134-L148)

名称：简中 `languages/chinese_s.po` 中六个注册名均无 `STRINGS.NAMES` 键；`CHARLIEARENA_SPIKE`（绝望石矿苗）与 `FUSED_SHADELING_BOMB`（绝望螨）是本体名，不是 FX 名。标题与上表中文为已核验效果描述，非官方译名。[矿苗名](../../../raw/current/scripts/languages/chinese_s.po#L340795-L340797) [绝望螨名](../../../raw/current/scripts/languages/chinese_s.po#L343095-L343097) [炮塔名](../../../raw/current/scripts/languages/chinese_s.po#L342530-L342532)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：groundpounder | 每次 `GroundPound` 先在 pt（缺省宿主位置）`SpawnPrefab(self.groundpoundringfx)`（默认 `groundpoundring_fx`）；默认名且非 PointMode 时用 `min(numRings,max(damageRings,destructionRings))` 算 hitrings，按 `sqrt(hitradius/12)` 缩放，hitrings≤2 调 `FastForward`，客户端代理跳到第 5 帧。 | 主模拟；环在逐环结算前无条件生成，出现环不代表命中；换非默认名不做缩放／跳帧。 | [默认字段](../../../raw/current/scripts/components/groundpounder.lua#L21) [入口与缩放](../../../raw/current/scripts/components/groundpounder.lua#L307-L322) [[reference/components/groundpounder|组件页]] |
| 关联 Prefab：dragonfly | `groundpounder.groundpoundringfx = "firering_fx"`（同宿主 `groundpoundfx` 仍是 `firesplash_fx`）；`firering_fx` 在 prefabs 依赖表中。 | 主模拟 Boss 震地；换名后中心环不缩放、不跳帧；环点火焰分工见 firesplash_fx 页。 | [配置](../../../raw/current/scripts/prefabs/dragonfly.lua#L548-L550) [依赖](../../../raw/current/scripts/prefabs/dragonfly.lua#L20) [[reference/prefabs/firesplash_fx|firesplash_fx]] |
| 关联 Prefab：bearger | 普通与变异两份 prefabs 列表各声明一次 `groundpoundring_fx`，由组件默认中心环生成。 | 依赖声明不等于工厂调用；实际生成仍在 groundpounder 的 `GroundPound`。 | [普通](../../../raw/current/scripts/prefabs/bearger.lua#L24-L25) [变异](../../../raw/current/scripts/prefabs/bearger.lua#L39-L40) |
| 直接生成：wurt_terraform_item | lunar 变体在道具上设 `_extra_onhit_fn = OnHit_Lunar`，施法生成 `wurt_terraform_projectile` 时复制到弹体；弹体命中 `OnHitTerraformer` 后调该回调，按地形范围对角 `sqrt(zone_diagonal/12)` 缩放并在 (x,0,z) 生成 `groundpoundring_fx`，不调 `FastForward`。 | 主模拟；仅 lunar 变体。 | [OnHit_Lunar](../../../raw/current/scripts/prefabs/wurt_terraform_item.lua#L264-L272) [传递](../../../raw/current/scripts/prefabs/wurt_terraform_item.lua#L46-L52) [接线](../../../raw/current/scripts/prefabs/wurt_terraform_item.lua#L307) [命中](../../../raw/current/scripts/prefabs/wurt_terraform_item.lua#L317-L334) |
| 生成点：quaker（3 行） | `_BreakDebris` 在碎块位置 (x,0,z) 生成（L166），供坠洞／落地碎裂等路径复用；重型碎块撞到同类 quakedebris（L200）与轻型碎块撞到同类（L230）各直接生成。 | 主模拟；`_BreakDebris` 以 `_ismastersim` 门禁。 | [L164-L168](../../../raw/current/scripts/components/quaker.lua#L164-L168) [L198-L201](../../../raw/current/scripts/components/quaker.lua#L198-L201) [L228-L231](../../../raw/current/scripts/components/quaker.lua#L228-L231) |
| 生成点：moon_device | 装置破裂序列的 `ClearArea` 对 15 半径内 inventoryitem 走 `meteor_invitem_behaviour`；按 `TUNING.METEOR_SMASH_INVITEM_CHANCE` 砸碎物品时在物品位置生成碎块。 | 主模拟；`stage1_break` 破裂清场。 | [ClearArea](../../../raw/current/scripts/prefabs/moon_device.lua#L251-L281) [调用](../../../raw/current/scripts/prefabs/moon_device.lua#L289-L290) [生成](../../../raw/current/scripts/prefabs/moon_device.lua#L207-L222) |
| 生成点：ruins_cavein_obstacle | 坍塌碎块落到洞口（`IsPointNearHole`）时生成碎块。 | 主模拟；洞口分支。 | [生成](../../../raw/current/scripts/prefabs/ruins_cavein_obstacle.lua#L66-L71) [洞口判断](../../../raw/current/scripts/prefabs/ruins_cavein_obstacle.lua#L190-L191) |
| 生成点：saddle | 鞍具耐久耗尽 `OnUsedUp` 时在鞍具位置生成碎块。 | 主模拟；仅用尽时。 | [OnUsedUp](../../../raw/current/scripts/prefabs/saddle.lua#L6-L9) |
| 生成点：shadowmeteor | `shadowmeteor` 砸物品时按 `inst.peripheral` 或 `TUNING.METEOR_SMASH_INVITEM_CHANCE` 概率生成碎块。 | 主模拟；peripheral／概率分支。 | [生成](../../../raw/current/scripts/prefabs/shadowmeteor.lua#L106-L111) |
| 生成点：charliearena_spike | `DoExplosion` 在 AOE 结算后、销毁本体前，于本体位置生成一次爆裂 FX；AOE 由本体结算。 | 主模拟；近身且处于 lunacy 状态的玩家触发闪烁后，由 `0.6 + math.random() * 0.25` 秒任务引爆。 | [DoExplosion](../../../raw/current/scripts/prefabs/charliearena_spike.lua#L133-L161) [爆炸任务](../../../raw/current/scripts/prefabs/charliearena_spike.lua#L180) [触发](../../../raw/current/scripts/prefabs/charliearena_spike.lua#L193-L199) |
| 生成点：fused_shadeling_bomb | `do_explosion_effect` 在爆炸 AOE 前生成 `fused_shadeling_bomb_death_fx`；同一函数还生成 `fused_shadeling_bomb_scorch` 灼痕（本页不展开）。 | 主模拟；完整引爆（timer）与 quickfuse 引爆都调用该函数。 | [生成](../../../raw/current/scripts/prefabs/fused_shadeling_bomb.lua#L95-L106) [完整引爆](../../../raw/current/scripts/prefabs/fused_shadeling_bomb.lua#L123-L140) [快速引爆](../../../raw/current/scripts/prefabs/fused_shadeling_bomb.lua#L321-L325) |
| 生成点：eye_charge | 投射物 `OnHit` 在投射物位置生成 `eye_charge_hit` 后自删；未命中走 `OnMissFn(inst.Remove)`，不生成 FX。 | 主模拟；仅命中瞬间。 | [OnHit](../../../raw/current/scripts/prefabs/eye_charge.lua#L7-L10) [OnMiss](../../../raw/current/scripts/prefabs/eye_charge.lua#L52-L53) |
| 关联 Prefab：willow_ember | `spawnfirefx(pos)` 生成 1 个 `firering_fx`＋4 个 `firesplash_fx`；但全树只有定义行，没有调用，是死代码。 | 不能当活跃生成点；详细分工见 firesplash_fx 页。 | [定义](../../../raw/current/scripts/prefabs/willow_ember.lua#L260-L274) |
| 资源 | 动画：`charliearena_spike.zip`（anim `explode`）、`dragonfly_ring_fx.zip`、`bearger_ring_fx.zip`、`ground_chunks_breaking.zip`（bank `ground_breaking`）、`fused_shadeling_bomb.zip`（anim `death_fx`）；`eye_charge_hit` 无动画。 | 注册字段引用；未解包资源内容，动画时长未核验。 | [地刺](../../../raw/current/scripts/prefabs/charliearena_spike.lua#L1-L4) [火环](../../../raw/current/scripts/prefabs/fireringfx.lua#L1-L4) [震地环](../../../raw/current/scripts/prefabs/groundpoundringfx.lua#L1-L4) [碎块](../../../raw/current/scripts/prefabs/ground_chunks_breaking.lua#L1-L4) [炸弹](../../../raw/current/scripts/prefabs/fused_shadeling_bomb.lua#L3-L6) |

## 典型接入

### 复用中心震地环并跳帧

- 前提与选择：服务端执行；只生成表现，不结算伤害。`groundpoundring_fx` 不依赖玩法组件，可直接生成；需要和命中范围一致时沿用 groundpounder 的缩放算法。
- 生效与撤销：服务端主壳 3 秒后删；本地代理在非专服生成后一帧创建，读取 `net_bool` 决定是否跳到第 5 帧，动画结束自行删除。
- 原版依据：[groundpounder 入口与缩放](../../../raw/current/scripts/components/groundpounder.lua#L307-L322)；[代理跳帧](../../../raw/current/scripts/prefabs/groundpoundringfx.lua#L6-L33)。

```lua
-- 服务端；x／z 为世界坐标，hitradius 为期望外观半径，hitrings 为命中环数
local fx = SpawnPrefab("groundpoundring_fx")
fx.Transform:SetPosition(x, 0, z)
local fxscale = math.sqrt(hitradius / 12) -- 原版注释：art radius 为 12，开方补偿 Transform 缩放
fx.Transform:SetScale(fxscale, fxscale, fxscale)
if hitrings <= 2 and fx.FastForward ~= nil then
    fx:FastForward()
end
```

- 调整与验证：`FastForward` 只置 `net_bool`，不改变服务端寿命；`hitrings` 只推外观半径（`hitradius`／缩放／跳帧），真正的命中与破坏范围由 `damageRings`／`destructionRings` 逐环决定（`DestroyRing` 用 `radius ± ringWidth`），改 `hitradius` 只改外观、不改结算半径。

## 常见陷阱与限制

- **误区：看到 FX 就以为伤害已结算**：`GroundPound` 在逐环结算前就无条件生成中心环；地刺爆裂 FX 在 AOE 之后生成，炸弹爆炸 FX 在 AOE 之前生成——三者时序不同，不能用一个顺序推断所有爆炸。[中心环入口](../../../raw/current/scripts/components/groundpounder.lua#L307-L322) [地刺](../../../raw/current/scripts/prefabs/charliearena_spike.lua#L133-L161) [炸弹](../../../raw/current/scripts/prefabs/fused_shadeling_bomb.lua#L95-L106)
- **边界：`_fastforward` 是出生时快照**：客户端代理创建时读一次 `net_bool`；代理创建后再调 `FastForward()` 不会补跳，也没有 dirty 回调。[代理](../../../raw/current/scripts/prefabs/groundpoundringfx.lua#L30-L32) [setter](../../../raw/current/scripts/prefabs/groundpoundringfx.lua#L35-L37)
- **边界：环的 3 秒定时不是动画时长**：`firering_fx` 与 `groundpoundring_fx` 的主壳各自 3 秒定时删，本地代理另按 `animover` 自删；不要用 3 秒推断动画长度（资源时长未核验）。[火环](../../../raw/current/scripts/prefabs/fireringfx.lua#L55) [震地环](../../../raw/current/scripts/prefabs/groundpoundringfx.lua#L64)
- **边界：碎块 1 秒定时与 7 个生成点独立**：主壳 1 秒删、代理 animover 删；7 行 `SpawnPrefab` 中 6 行把 y 压平为 0（saddle 直接沿用鞍具世界坐标），且都依附宿主的使用耗尽／破坏／砸落路径，不是独立伤害源。[定时](../../../raw/current/scripts/prefabs/ground_chunks_breaking.lua#L52) [quaker](../../../raw/current/scripts/components/quaker.lua#L164-L168)
- **边界：`eye_charge_hit` 没有视觉**：主壳无 AnimState，0.5 秒是网络壳寿命而非音效时长；本地代理只播命中音，专服不创建该代理。[工厂](../../../raw/current/scripts/prefabs/eye_charge.lua#L74-L98)
- **边界：`fused_shadeling_bomb_death_fx` 不带 FX 标签**：同族其余都加 `FX`，按该标签筛选、排除或查找时找不到它；清理只靠 `animover`。[工厂](../../../raw/current/scripts/prefabs/fused_shadeling_bomb.lua#L295-L318)
- **边界：`willow_ember` 的 `spawnfirefx` 是死代码**：该局部函数全树无调用（`willow_ember` 本体仍是活跃 prefab）；`firering_fx` 的原版活跃使用只有 dragonfly 的 `groundpoundringfx` 配置。[定义](../../../raw/current/scripts/prefabs/willow_ember.lua#L260-L274) [配置](../../../raw/current/scripts/prefabs/dragonfly.lua#L550)
- **分工：三个震地表现分属三页**：中心环看本页，落点尘土看 [[reference/prefabs/groundpound_fx|groundpound_fx]]，蜻蜓龙环点火溅看 [[reference/prefabs/firesplash_fx|firesplash_fx]]；字段与生成时机分别维护。

## 验证记录

- 源码审阅：2026-11-04，Steam buildid 26335394（实读 `raw/current/BUILD.txt`）。六个注册逐项核验工厂、端、生成点与清理；全 `raw/current/scripts` 以六个字面名反查，`groundpoundring_fx` 另查 `groundpoundringfx` 字段消费，碎块确认 7 行 `SpawnPrefab`；`languages/chinese_s.po` 定向检索名称键。本页跨 6 个工厂文件，含联网实体动画、网络壳＋本地代理、`net_bool` 跳帧与多种清理，需整体对照，按 complex 档。未覆盖：动画资源时长与内容、客户端实际观感、专服晚加入时序、事件包／其他模式是否复用、本体 SG 与玩法流程。
- 游戏运行：未运行；validation 保持 source-reviewed。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 通过（0 errors／0 warnings）；Lua 片段 `parser_probe.exe --syntax-only` 通过。

返回 [[reference/prefabs/fx-world-index|特效 Prefab 索引·生物与环境]]。
