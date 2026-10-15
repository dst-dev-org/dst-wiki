---
title: "睡眠云与孢子云（sleepcloud 与 sporecloud 两族五注册）"
created: "2026-11-04"
updated: "2026-11-04"
type: prefab
summary: "两族同构云实体：sleepcloud／sleepcloud_lunar 持续范围催眠，sporecloud 持续范围伤害并加速腐败；各由本地底座、7 片 overlay 与 net_tinybyte 状态组成，逐注册核验生成点、端、父挂与清理。"
aliases: ["sleepcloud", "sleepcloud_lunar", "sleepcloud_overlay", "sporecloud", "sporecloud_overlay", "睡眠云", "孢子云", "睡眠云特效", "孢子云特效"]
tags: [prefab, fx, server-client]
size_class: complex
dst_build: "steam-buildid:26335394"
sources: ["prefabs/sleepcloud.lua", "prefabs/sporecloud.lua", "prefabs/sleepbomb.lua", "prefabs/moon_mushroom.lua", "prefabs/sporebomb.lua", "prefabs/mushroomsprout.lua", "prefabs/toadstool.lua", "stategraphs/SGtoadstool.lua", "components/aura.lua", "components/combat.lua", "components/perishable.lua", "widgets/fumeover.lua", "widgets/countdownbeta.lua", "prefablist.lua", "fx.lua", "prefabs/alterguardian_phase4_lunarrift.lua", "prefabs/bomb_lunarplant.lua", "prefabs/charliearena_spike.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 睡眠云与孢子云（sleepcloud 与 sporecloud 两族五注册）

## 快速判断

- 职责与适用：本页收 5 个注册——`sleepcloud` 是睡眠炸弹命中后的睡眠云（混合：0.5s 周期 `DoAreaDrowsy`＋20s `disperse` timer）；`sporecloud` 是毒菌蟾蜍的孢子炸弹／烧毁的孢子帽产生的孢子云（混合：combat 默认伤害＋aura 半径与 tick＋60s timer）；`sleepcloud_lunar` 是月亮蘑菇触发的蓝色变体；`sleepcloud_overlay`／`sporecloud_overlay` 是纯 FX。两族不在 `fx.lua` 通用注册表，各由独立 prefab 工厂构造。[睡眠云工厂](../../../raw/current/scripts/prefabs/sleepcloud.lua#L313-L363)、[孢子云工厂](../../../raw/current/scripts/prefabs/sporecloud.lua#L313-L388)
- 尺寸档：`complex`。5 个注册、两套玩法（催眠／伤害＋腐败）、底座／overlay／状态三部分和新建／读档／消散三条生命周期分支需要整体说明；不是因为源码行数升档。
- 优先使用：查“谁让人睡着、谁造成伤害”先看下面的玩法归属；查 5 个注册的工厂、端与初始化看“注册与构造”；查生成宿主和清理看“关联机制导航”；纯表现差异看 overlay 行。
- 重要副作用：睡眠云每 0.5s 对半径 3.5 内目标结算，非 PVP 仅 `sleeper` 生物（玩家被排除），PVP 才把玩家纳入并走 `grogginess`；孢子云每 1s 对半径 3.5 内目标造成 20 伤害，并让新鲜度物品每 tick 减少 0.07（7 个百分点），两者都在 `disperse` 后播放 pst 并在 3 秒后移除。[睡眠筛选](../../../raw/current/scripts/prefabs/sleepcloud.lua#L238-L307)、[孢子配置](../../../raw/current/scripts/prefabs/sporecloud.lua#L358-L367)、[ReducePercent](../../../raw/current/scripts/components/perishable.lua#L235-L238)
- 修改边界：组件、timer、周期任务与范围结算都在 `SetPristine` 后的主模拟分支；客户端只读 netvar 并建本地表现；专服不建底座；overlay 本身无玩法组件、无自删调用，清理由云统一发起。[端分界](../../../raw/current/scripts/prefabs/sleepcloud.lua#L337-L344)、[专服判断](../../../raw/current/scripts/prefabs/sleepcloud.lua#L228-L236)

### 玩法归属

- `sleepcloud`：睡眠由云自身的 `DoAreaDrowsy` 周期任务结算，云没有 `combat`、不造成伤害；目标有 `sleeper` 时 `AddSleepiness`，玩家走 `grogginess:AddGrogginess`／`ExtendKnockout`，两者都没有时只推 `knockedout` 事件；刚被攻击 2s 内、正在燃烧／冻结／被钉／化石化的目标会跳过。`inst.owner` 仅用于跳过指定实体（月亮蘑菇的进食者）。[周期任务](../../../raw/current/scripts/prefabs/sleepcloud.lua#L344-L344)、[结算](../../../raw/current/scripts/prefabs/sleepcloud.lua#L242-L307)
- `sporecloud`：伤害由云自身的 `combat` 默认伤害经 `aura:OnTick` 的 `DoAreaAttack` 结算，攻击者是云本身，没有玩家归属；腐败由独立的 `DoAreaSpoil` 对 `perishable:ReducePercent` 结算；云不施加睡眠。[安装](../../../raw/current/scripts/prefabs/sporecloud.lua#L358-L367)、[光环实现](../../../raw/current/scripts/components/aura.lua#L44-L59)、[腐败](../../../raw/current/scripts/prefabs/sporecloud.lua#L305-L311)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `sleepcloud` | `fn`：Transform／AnimState／SoundEmitter／Network → bank/build `sleepcloud` 播 `sleepcloud_pre` → FX／NOCLICK／notarget → `spore_loop` 循环声 → `_state` net_tinybyte（statedirty）→ 主模拟：0.5s `DoAreaDrowsy`、`sleepcloud_loop`、`timer.disperse`＝`SLEEPBOMB_DURATION`（20）、7 片 overlay 排程、`SetOwner`／`OnLoad` | 睡眠云主实体；主模拟才装 timer 与周期任务；`_state` 0→1（pre 播完）→2（disperse）。[工厂](../../../raw/current/scripts/prefabs/sleepcloud.lua#L313-L363)、[常量](../../../raw/current/scripts/prefabs/sleepcloud.lua#L12-L23)、[timer](../../../raw/current/scripts/prefabs/sleepcloud.lua#L349-L352) |
| `sleepcloud_lunar` | `lunar_fn`：调用 `fn()` 后把 `_create_base_fn` 换成 `CreateBaseLunar` | 与 `sleepcloud` 唯一差异是底座调色 `(0.2/0.6, 0.25/0.6, 1.0, 0.6)`；催眠参数、时长、overlay 全同。[lunar_fn](../../../raw/current/scripts/prefabs/sleepcloud.lua#L366-L372)、[调色](../../../raw/current/scripts/prefabs/sleepcloud.lua#L95-L102) |
| `sleepcloud_overlay` | `overlayfn`：独立联网实体，Transform／AnimState／Network、`SetTwoFaced`、bank/build `sleepcloud` 播 `sleepcloud_overlay_pre`；FX／NOCLICK；主模拟监听 animover 循环、`persists=false` | 纯 FX，不能单独触发睡眠；只由 `SpawnOverlayFX` 生成并父挂到云。[工厂](../../../raw/current/scripts/prefabs/sleepcloud.lua#L376-L403)、[生成](../../../raw/current/scripts/prefabs/sleepcloud.lua#L36-L62) |
| `sporecloud` | `fn`：Transform／AnimState／Light／SoundEmitter／Network → bank/build `sporecloud` 播 `sporecloud_pre`、bloom、绿光 → FX／NOCLICK／notarget／sporecloud → `_state` net_tinybyte＋`_fade` net_smallbyte → 主模拟：combat 20、aura 3.5／1s、腐败任务、`sporecloud_loop`、`timer.disperse`＝`TOADSTOOL_SPORECLOUD_LIFETIME`（60）、7 片 overlay、`FadeInImmediately`／`FinishImmediately` | 混合载体；动画声效是壳，玩法在服务端 combat／aura／perishable。[工厂](../../../raw/current/scripts/prefabs/sporecloud.lua#L313-L388)、[安装](../../../raw/current/scripts/prefabs/sporecloud.lua#L358-L367) |
| `sporecloud_overlay` | `overlayfn`：同壳、bank/build `sporecloud`、`SetLightOverride(.2)`、播 `sporecloud_overlay_pre`；主模拟监听 animover 循环、`persists=false` | 纯 FX；不提供伤害或腐败。[工厂](../../../raw/current/scripts/prefabs/sporecloud.lua#L390-L419) |

两个文件各有一条 return：[sleepcloud／lunar／overlay](../../../raw/current/scripts/prefabs/sleepcloud.lua#L405-L407)、[sporecloud／overlay](../../../raw/current/scripts/prefabs/sporecloud.lua#L421-L422)。资源声明为 `anim/sleepcloud.zip`＋`anim/sporecloud_base.zip`、`anim/sporecloud.zip`＋`anim/sporecloud_base.zip`；底座统一用 `sporecloud_base` 的 bank/build，睡眠云底座调灰、lunar 调蓝，overlay 各用本族资源。[睡眠云资源](../../../raw/current/scripts/prefabs/sleepcloud.lua#L1-L5)、[孢子云资源](../../../raw/current/scripts/prefabs/sporecloud.lua#L1-L5)

### 同构云工厂与变体辨析

- 两文件各自复制同一套三段式：本地非联网底座 `CreateBase`（bank/build `sporecloud_base`）＋7 片 overlay（`OVERLAY_COORDS`：中心 1 片、六边形 6 片，位置乘 0.85、缩放 0.8/1）＋`_state` net_tinybyte 驱动 0 预动画／1 循环／2 消散。[睡眠云三段](../../../raw/current/scripts/prefabs/sleepcloud.lua#L64-L93)、[孢子云三段](../../../raw/current/scripts/prefabs/sporecloud.lua#L106-L133)
- 父挂与端：底座与 overlay 都 `SetParent(inst.entity)`；底座在专服跳过，客户端在 `statedirty` 且状态为 1 时本地重建；overlay 是联网实体，由主模拟统一生成，7 片在 0～0.7s 内错开，读档时一次补齐并以随机相位进入循环。[底座](../../../raw/current/scripts/prefabs/sleepcloud.lua#L228-L236)、[overlay 排程](../../../raw/current/scripts/prefabs/sleepcloud.lua#L357-L360)、[状态分支](../../../raw/current/scripts/prefabs/sleepcloud.lua#L106-L121)
- 不是共享模块：`sleepcloud.lua` 与 `sporecloud.lua` 各自维护 `OVERLAY_COORDS`／`SpawnOverlayFX`／`CreateBase`，改一族不会影响另一族；共用的只有 `sporecloud_base` 资源与这套结构。`sleepcloud_lunar` 只替换底座生成函数，不改变玩法或 overlay。[两族结构对照](../../../raw/current/scripts/prefabs/sleepcloud.lua#L25-L62)、[孢子云副本](../../../raw/current/scripts/prefabs/sporecloud.lua#L67-L104)

### 清理路径

- `DoDisperse`（timer 到期或 `FinishImmediately`）：取消 `_inittask` 与 `_drowsytask`（睡眠云）或 `DisableCloud`（孢子云，关 aura、取消腐败任务、移除 `sporecloud` 标签），摘掉 animover，`_state` 设为 2，播本族 `_pst`，杀 `spore_loop`，`persists=false`，3 秒后 `Remove`；overlay 任务全取消，overlay 播 pst（首片立即，其余 0～0.5s 错开）。读档时 timer 已过期则直接 Hide 后移除，不播 pst。[睡眠云清理](../../../raw/current/scripts/prefabs/sleepcloud.lua#L137-L171)、[孢子云清理](../../../raw/current/scripts/prefabs/sporecloud.lua#L177-L209)、[读档过期](../../../raw/current/scripts/prefabs/sleepcloud.lua#L179-L211)

## 关联机制导航

### 组件与同步

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| `组件：timer` | 两者主模拟都 `AddComponent("timer")` 并 `StartTimer("disperse", …)`；`timerdone` 判 `data.name == "disperse"` 后 `DoDisperse` | `sleepcloud` 20s；`sporecloud` 60s；读档由 `OnLoad` 检查剩余时间，过期走隐藏并移除 | [[reference/components/timer]]；[睡眠云](../../../raw/current/scripts/prefabs/sleepcloud.lua#L349-L352)、[孢子云](../../../raw/current/scripts/prefabs/sporecloud.lua#L372-L375) |
| `组件：sleeper` | `sleepcloud` 目标侧：`AddSleepiness(10, sleeptime / GetSleepTimeMultiplier())`，睡着后按 `IsAsleep()` 续时 | 非 PVP 的 `sleeper` 生物；每 0.5s 加 10 睡眠值，睡眠时长首轮 5s、睡着期间每 0.5s 续时（4.5→1.5s）；刚被攻击 2s 内跳过，醒来后约 4s 内不再施加 | [[reference/components/sleeper]]；[结算](../../../raw/current/scripts/prefabs/sleepcloud.lua#L275-L283)、[门槛](../../../raw/current/scripts/prefabs/sleepcloud.lua#L22-L23) |
| `组件：grogginess` | `sleepcloud` 对 PVP 玩家 `AddGrogginess(1, sleeptime)`，已晕则 `ExtendKnockout`；两者皆无的目标推 `knockedout` | PVP 才纳入 `player`；首轮 4s，续时最低 1s | [[reference/components/grogginess]]；[玩家分支](../../../raw/current/scripts/prefabs/sleepcloud.lua#L275-L301) |
| `组件：combat` | `sporecloud` `SetDefaultDamage(20)`；`aura:OnTick` 以云自身为攻击者调 `DoAreaAttack` | 主模拟；每 tick 1s、半径 3.5，排除 `AURA_EXCLUDE_TAGS`（含 `toadstool`） | [[reference/components/combat]]；[安装](../../../raw/current/scripts/prefabs/sporecloud.lua#L358-L367)、[DoAreaAttack](../../../raw/current/scripts/components/combat.lua#L1312-L1340) |
| `组件：aura` | `sporecloud` 设 `radius`／`tickperiod`／`auraexcludetags` 后 `Enable(true)`；`DisableCloud` 关闭 | 主模拟；消散或读档过期时关闭；周期任务在 `Enable` 时按当时 `tickperiod` 建立 | [[reference/components/aura]]；[配置](../../../raw/current/scripts/prefabs/sporecloud.lua#L361-L365)、[关闭](../../../raw/current/scripts/prefabs/sporecloud.lua#L166-L175) |
| `组件：perishable` | `sporecloud` `DoAreaSpoil` 对半径内 `fresh`／`stale`／`spoiled` 物品 `ReducePercent(.07)` | 每 1s；排除 `small_livestock`；Limbo 中不在背包或打开的容器则跳过 | [[reference/components/perishable]]；[扫描](../../../raw/current/scripts/prefabs/sporecloud.lua#L305-L311)、[单件](../../../raw/current/scripts/prefabs/sporecloud.lua#L286-L301)、[扣减](../../../raw/current/scripts/components/perishable.lua#L235-L238) |
| 同步：net_tinybyte／net_smallbyte | `_state` 三态驱动底座与主层动画；`sporecloud` 另有 `_fade` 灯光淡变 | 主模拟 `set`，客户端 `statedirty`／`fadedirty` 本地插值；专服不建底座 | [状态分支](../../../raw/current/scripts/prefabs/sleepcloud.lua#L106-L121)、[淡变](../../../raw/current/scripts/prefabs/sporecloud.lua#L43-L65) |
| 标签：`sporecloud` | 毒菌蟾蜍死亡时按标签清云；客户端 HUD 按标签叠加雾效 | 工厂在 `SetPristine` 前添加（两端都有）；主模拟 `DisableCloud` 移除；HUD 在 4 单位内按距离累计 | [清云](../../../raw/current/scripts/stategraphs/SGtoadstool.lua#L458-L462)、[HUD](../../../raw/current/scripts/widgets/fumeover.lua#L16-L27) |

### 生成宿主与消费者

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 关联 Prefab：`sleepbomb` | `OnHit` 在命中点生成 `sleepbomb_burst`＋`sleepcloud` | 服务端命中；爆裂动画归 [[reference/prefabs/glass_fx]] | [OnHit](../../../raw/current/scripts/prefabs/sleepbomb.lua#L16-L21) |
| 关联 Prefab：`moon_mushroom` | 月亮蘑菇进食且激活 `wormwood_moon_cap_eating` 时生成 `sleepcloud_lunar` 并 `SetOwner(eater)` | 服务端；owner 用来跳过进食者 | [生成](../../../raw/current/scripts/prefabs/moon_mushroom.lua#L37-L40) |
| 关联 Prefab：`sporebomb` | 毒菌蟾蜍的孢子炸弹以 debuff 附到玩家，结束／脱离时在目标位置生成 `sporecloud` 并 `FadeInImmediately` | 目标玩家；主模拟 | [DoSporeBomb](../../../raw/current/scripts/prefabs/toadstool.lua#L239-L243)、[OnDetached](../../../raw/current/scripts/prefabs/sporebomb.lua#L70-L84) |
| 关联 Prefab：`mushroomsprout` | 孢子帽被烧毁时在自身位置生成 `sporecloud` | 主模拟 | [OnBurnt](../../../raw/current/scripts/prefabs/mushroomsprout.lua#L79-L112) |
| 调用方：`SGtoadstool` | 毒菌蟾蜍死亡第 35 帧对 8 单位内的 `sporecloud` 调 `FinishImmediately` | 主模拟；停表并进入消散，不是立即 Remove | [死亡清云](../../../raw/current/scripts/stategraphs/SGtoadstool.lua#L458-L462)、[FinishImmediately](../../../raw/current/scripts/prefabs/sporecloud.lua#L217-L222) |
| 资源 | `anim/sleepcloud.zip`、`anim/sporecloud.zip`、`anim/sporecloud_base.zip` | 声明与实际 bank/build 一致；包内动画内容未解包 | [睡眠云资源](../../../raw/current/scripts/prefabs/sleepcloud.lua#L1-L5)、[孢子云资源](../../../raw/current/scripts/prefabs/sporecloud.lua#L1-L5) |
| API／数据：`TUNING` | `TOADSTOOL_SPORECLOUD_DAMAGE`／`ROT`／`RADIUS`／`TICK`／`LIFETIME`；`SLEEPBOMB_DURATION` | 20／.07／3.5／1／60；20 | [孢子云常量](../../../raw/current/scripts/tuning.lua#L2521-L2525)、[睡眠云时长](../../../raw/current/scripts/tuning.lua#L1939) |

## 典型接入

### 从自定义来源生成一朵云

- 前提与选择：目标文件放在任意服务端命中、技能或植物烧毁回调；用 `SpawnPrefab` 是因为四个原版生成点都采用同一方式并在随后设置世界坐标；睡眠云用 `sleepcloud`／`sleepcloud_lunar`，孢子云用 `sporecloud`。
- 生效与撤销：只在主模拟生成；云自管 timer、消散与移除，生成方不需要保存引用或手动清理；`SetOwner` 只影响睡眠跳过，不改变云的寿命。
- 原版依据：月亮蘑菇的生成点与 `SetOwner` 调用见 [moon_mushroom.lua](../../../raw/current/scripts/prefabs/moon_mushroom.lua#L37-L40)；睡眠炸弹的同点生成见 [sleepbomb.lua](../../../raw/current/scripts/prefabs/sleepbomb.lua#L16-L21)。

```lua
-- 在服务端命中/触发点生成月亮睡眠云；owner 可为 nil
local function SpawnLunarSleepCloud(x, y, z, owner)
    local cloud = SpawnPrefab("sleepcloud_lunar")
    cloud.Transform:SetPosition(x, y, z)
    if owner ~= nil then
        cloud:SetOwner(owner) -- 仅主模拟存在；用于跳过 owner 的睡眠结算
    end
    return cloud
end
```

- 调整与验证：必须显式设置坐标，否则云会落在原点；睡眠云的 20s 时长来自 `TUNING.SLEEPBOMB_DURATION`，0.5s tick、3.5 半径、5/1.5s 与 4/1s 睡眠时长是文件内常量；孢子云的伤害／半径／tick／寿命在 `TUNING`。本页未运行游戏，生成位置与睡眠结果未实测。

## 常见陷阱与限制

- **误区：两族共用一套云工厂函数**：实际是两份文件各自复制的同构实现，`sleepcloud.lua` 与 `sporecloud.lua` 的 `OVERLAY_COORDS`、`SpawnOverlayFX`、`CreateBase` 互不引用；改一族不会影响另一族。[对照](../../../raw/current/scripts/prefabs/sleepcloud.lua#L25-L62)
- **误区：睡眠云在非 PVP 也能放倒玩家**：非 PVP 分支要求 `sleeper` 且排除 `player`，玩家只有在 PVP 开启时才进入 `player` 分支并走 `grogginess`；睡眠炸弹在非 PVP 主要作用于生物。[筛选](../../../raw/current/scripts/prefabs/sleepcloud.lua#L238-L249)
- **误区：`sleepcloud_lunar` 是另一套机制**：它只替换 `_create_base_fn` 做蓝色底座，催眠、时长、overlay 与 `sleepcloud` 全同。[lunar_fn](../../../raw/current/scripts/prefabs/sleepcloud.lua#L366-L372)
- **误区：`FinishImmediately` 会立刻删除云**：它只停掉 `disperse` 表并调用 `DoDisperse`，云还要播 pst，3 秒后才 Remove。[实现](../../../raw/current/scripts/prefabs/sporecloud.lua#L217-L222)、[移除](../../../raw/current/scripts/prefabs/sporecloud.lua#L177-L192)
- **边界：孢子云伤害没有玩家归属**：`DoAreaAttack` 的攻击者是云实体，不是投掷者；毒菌蟾蜍本体因 `toadstool` 在排除标签内而不吃孢子云的伤害。[排除标签](../../../raw/current/scripts/prefabs/sporecloud.lua#L12-L12)、[伤害入口](../../../raw/current/scripts/components/aura.lua#L44-L59)
- **边界：运行期改 `aura.tickperiod` 不会自动改已排任务**：`Enable(true)` 时按当时值建立周期任务；要改先 `Enable(false)` 再设值再 `Enable(true)`，且 `_spoiltask` 仍按创建时的周期运行。[Enable](../../../raw/current/scripts/components/aura.lua#L25-L42)、[腐败任务](../../../raw/current/scripts/prefabs/sporecloud.lua#L367-L367)
- **边界：overlay 不自删，旋转取的是缩放值**：`KillOverlayFX` 只切 pst；`SetRotation(set[4])` 传入的也是缩放（第 5 项角度未使用），视觉效果未运行核验；不要依赖 overlay 自己消失。[清理](../../../raw/current/scripts/prefabs/sleepcloud.lua#L132-L135)、[旋转](../../../raw/current/scripts/prefabs/sleepcloud.lua#L44-L50)
- **边界：专服没有底座**：`TheNet:IsDedicated()` 为真时跳过底座；客户端在 `statedirty` 后本地重建，因此底座只存在于有渲染的一端。[专服判断](../../../raw/current/scripts/prefabs/sleepcloud.lua#L228-L236)、[客户端重建](../../../raw/current/scripts/prefabs/sleepcloud.lua#L106-L121)

## 验证记录

- 源码审阅：2026-11-04，Steam buildid 26335394。5 个注册逐项核对工厂、初始化分界、端、父挂与清理；全 `raw/current/scripts` 的 Lua 逐文件扫描（编码异常按忽略处理）共 77 处命中、13 个文件。主注册生成点仅 `sleepbomb.lua`（L20）、`moon_mushroom.lua`（L38）、`sporebomb.lua`（L79-L81）、`mushroomsprout.lua`（L94）；`sleepcloud_overlay`／`sporecloud_overlay` 只在各自文件的 `SpawnOverlayFX` 生成。`alterguardian_phase4_lunarrift`、`bomb_lunarplant` 仅 `OverrideSymbol("sleepcloud_pre", …)`，`charliearena_spike` 仅对 `sleepcloud_pre` 设符号高光，均不是生成点；`widgets/countdownbeta.lua` 的 UI 烟段已被块注释禁用（死代码），当前不构成生成/复用；`prefablist.lua` 只预载两个主注册。
- 消费者与宿主：`toadstool.lua` 的 `DoSporeBomb` 与 `SGtoadstool.lua` 的 `sporebomb` 状态、死亡清云、`widgets/fumeover.lua` 的 `sporecloud` 标签 HUD 均已核对；`sleepbomb` 的同点 `sleepbomb_burst` 归 [[reference/prefabs/glass_fx]]。
- 语言文件：`languages/chinese_s.po` 中 `STRINGS.NAMES.SPORECLOUD`＝“孢子云”（官方名），`sleepcloud` 无名称键，故“睡眠云”是效果描述、不是官方译名；“孢子云”按官方名使用。[名称](../../../raw/current/scripts/languages/chinese_s.po#L349925-L349927)
- 游戏运行：未运行；动画包内容与时长、父子实体移除的实际时机、睡眠／伤害／腐败在游戏内的表现均未实测。未覆盖：模组侧接入未验证，多语言名称未逐一检索。

返回 [[reference/prefabs/fx-world-index|特效 Prefab 索引·生物与环境]]；相邻条目：[[reference/prefabs/glass_fx|睡眠炸弹爆裂与玻璃化 FX]]。
