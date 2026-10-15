---
title: "海面水花特效族（ocean_splash_med1）"
created: "2026-11-01"
updated: "2026-11-01"
type: prefab
summary: "八个通用表注册承载海面水花表现：med／small 两档带声水花、ripple 无声背景涟漪、swim 游泳划水尾流；海鱼破水、海钓浮标、玩家游泳、龙虾／水蜘蛛／暗影傀儡与蜜渍路径按各自门禁落点生成。"
aliases: ["ocean_splash_med1", "ocean_splash_med2", "ocean_splash_small1", "ocean_splash_small2", "ocean_splash_ripple1", "ocean_splash_ripple2", "ocean_splash_swim1", "ocean_splash_swim2", "海面水花", "游泳水花", "水面涟漪", "海面涟漪"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["fx.lua", "stategraphs/SGwilson.lua", "stategraphs/SGwobster.lua", "stategraphs/SGspider_water.lua", "stategraphs/SGoceanfish.lua", "stategraphs/SGshadowwaxwell.lua", "stategraphs/SGshadow_trap.lua", "stategraphs/SGbeeguard.lua", "prefabs/oceanfishdef.lua", "prefabs/oceanfishingbobber.lua", "prefabs/player_common.lua", "prefabs/beequeen.lua", "prefabs/shrouden.lua", "prefabs/slingshotammo.lua", "prefabs/shadow_pillar.lua", "prefabs/waterprojectiles.lua", "prefabs/wobster.lua", "components/sharkboimanager.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 海面水花特效族（ocean_splash_med1）

## 快速判断

- 职责与适用：八个通用表注册承载海面水花与涟漪的纯表现——med／small 两档带声水花、ripple 无声背景涟漪、swim 游泳划水尾流；工厂协议见 [[reference/mechanisms/generic-fx-factory|通用 FX 工厂]]。[条目组](../../../raw/current/scripts/fx.lua#L1876-L1962)
- 优先使用：查"海面水花／水面涟漪／游泳尾流"选本页；与 [[reference/prefabs/weregoose_splash|鹅形态水花]] 同 bank／build／anim 但是独立注册，fn 行为不同（见注册表下注）；[[reference/prefabs/splash|落水水花族]] 是另一组注册，并存分工见陷阱。
- 重要副作用：无玩法作用；med／small 与 swim 带鸟形水声（swim 压至 0.6 音量），ripple 无声；ripple／swim 渲染在背景层。[条目组](../../../raw/current/scripts/fx.lua#L1876-L1962)
- 修改边界：8 名在简中 .po 均无 STRINGS.NAMES 键（定向检索无结果），标题为已核验效果描述、非官方译名；当前原版消费点全部落点式，无父挂用例。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `ocean_splash_med1`／`_med2` | 通用表条目带 fn：海面着色＋`SetFinalOffset(3)` | bank=`splash_weregoose_fx`、build=`splash_water_drop`；anim=`stationary`／`stationary2`；sound=鸟形水声。[条目](../../../raw/current/scripts/fx.lua#L1876-L1896) |
| `ocean_splash_small1`／`_small2` | 同上协议 | anim=`stationary_small`／`stationary_small2`，同声。[条目](../../../raw/current/scripts/fx.lua#L1898-L1918) |
| `ocean_splash_ripple1`／`_ripple2` | fn 改设 `LAYER_WORLD_BACKGROUND`＋海面着色，**无声**、无 FinalOffset | anim=`no_splash`／`no_splash2`。[条目](../../../raw/current/scripts/fx.lua#L1920-L1938) |
| `ocean_splash_swim1`／`_swim2` | fn 设背景层＋`SetDeltaTimeMultiplier(0.7)`＋音量 ×0.6，带声（被压低），**未设** `SetOceanBlendParams` | anim 同 ripple 组（`no_splash`／`no_splash2`），无声涟漪还是划水声靠注册名区分。[条目](../../../raw/current/scripts/fx.lua#L1940-L1962) |

与 weregoose 同名 anim 条目的关键差异：本族 fn 无"挂父即解挂"逻辑，med／small 反加 `SetFinalOffset(3)`（对比 [weregoose 解挂 fn](../../../raw/current/scripts/fx.lua#L787-L792)）。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 工厂协议 | 代理＋一帧本地表现；消费点全为 `SpawnPrefab` 后 `SetPosition` 落点式 | animover 自删；代理 1 秒回收 | [[reference/mechanisms/generic-fx-factory]] |
| 海鱼破水（SGoceanfish） | helper 从 `fish_def.breach_fx` 随机取 1 落点 | 服务端；small=`oceanfish_small_1～9`、med=`oceanfish_medium_1～9`；船下（underboat）抑制；eat 破水等多状态调用 | [helper](../../../raw/current/scripts/stategraphs/SGoceanfish.lua#L38-L41)、[调用例](../../../raw/current/scripts/stategraphs/SGoceanfish.lua#L124-L133)、[分档](../../../raw/current/scripts/prefabs/oceanfishdef.lua#L178-L179) |
| 海钓浮标（oceanfishingbobber） | `SpawnSplashFx` 用 `land_splash_fx` 前缀拼 `tostring(math.random(2))` 落点 | 服务端；抛竿入海（OnProjectileLand）与浮标被鱼食（OnEaten）触发；前缀按浮标种类：med=twig／plug／ball／oval，ripple=空钩与鸟形浮标，small=goose／malbatross | [前缀表](../../../raw/current/scripts/prefabs/oceanfishingbobber.lua#L4-L15)、[拼接](../../../raw/current/scripts/prefabs/oceanfishingbobber.lua#L18-L22)、[入海](../../../raw/current/scripts/prefabs/oceanfishingbobber.lua#L49)、[被食](../../../raw/current/scripts/prefabs/oceanfishingbobber.lua#L86-L91) |
| 冰洞钓鱼（sharkboimanager） | `SpawnPrefab("ocean_splash_med"..math.random(1, 2))` 落点 | 服务端；冰洞钓鱼交互 | [调用](../../../raw/current/scripts/components/sharkboimanager.lua#L487-L487) |
| 玩家游泳（SGwilson） | swim1／2 随机落点 | 服务端；`float` 状态 onupdate 游泳前进中每 16 帧（注释标明对齐 swim_loop 长度）、面朝方向身后 0.3 距离、y=0 | [调用](../../../raw/current/scripts/stategraphs/SGwilson.lua#L26062-L26075) |
| 龙虾 wobster（SGwobster） | small1 **固定名**落点 | 服务端；`bitehook_escape` 2／21 帧＋animqueueover（船下抑制）、`launched_out_of_water` onenter、`hop_pst` 3 帧 | [escape](../../../raw/current/scripts/stategraphs/SGwobster.lua#L100-L132)、[launched](../../../raw/current/scripts/stategraphs/SGwobster.lua#L155)、[hop_pst](../../../raw/current/scripts/stategraphs/SGwobster.lua#L172) |
| 水蜘蛛（SGspider_water） | small1 **固定名**落点 | 服务端；`eat` 状态 6 帧且 `amphibiouscreature.in_water` | [调用](../../../raw/current/scripts/stategraphs/SGspider_water.lua#L303-L307) |
| 暗影傀儡（SGshadowwaxwell，[po](../../../raw/current/scripts/languages/chinese_s.po#L349239-L349242)） | `TrySplashFX(inst, size)` 拼 `"ocean_splash_"..(size or "med")..N`，`IsOceanAtPoint` 门禁 | 服务端；spawn 任务按动画倍率缩放：0／20 基准帧 med、44 基准帧 small（未传倍率时 `mult=0.8～1.0`，取倒数后约 20～25／44～55 帧）；death 13／38 帧＋animover；run_start／run 步进 `TryStepSplash` 0.1 秒节流（med）；quickdespawn／lunge_loop med、appear／disappear small | [helper](../../../raw/current/scripts/stategraphs/SGshadowwaxwell.lua#L50-L63)、[spawn](../../../raw/current/scripts/stategraphs/SGshadowwaxwell.lua#L272-L289)、[步进](../../../raw/current/scripts/stategraphs/SGshadowwaxwell.lua#L427-L467)、[death](../../../raw/current/scripts/stategraphs/SGshadowwaxwell.lua#L555-L577)、[appear／disappear](../../../raw/current/scripts/stategraphs/SGshadowwaxwell.lua#L1127-L1197)、[lunge](../../../raw/current/scripts/stategraphs/SGshadowwaxwell.lua#L1272-L1282) |
| 暗影陷阱／暗影柱 | shadow_trap `TrySplashFX` 落 small1／2；shadow_pillar `DoSplash` 落 med1／2 | 服务端；均 `IsOceanAtPoint` 门禁；trap 在 spawn／trigger／dispell 三状态，pillar 在升起与降下（降下延 10 帧） | [trap](../../../raw/current/scripts/stategraphs/SGshadow_trap.lua#L5-L10)、[trap 调用](../../../raw/current/scripts/stategraphs/SGshadow_trap.lua#L25)、[pillar](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L65-L93) |
| 蜜渍路径（beequeen／shrouden／slingshotammo） | `TrySpawnHoney`／`TrySpawnGelBlob`：可通行点落蜜渍，不可通行（海面）改落 ripple1／2 | 服务端落点；beequeen 仅按 else 分支，shrouden／slingshotammo 另加 `TheWorld.has_ocean` 门禁 | [beequeen](../../../raw/current/scripts/prefabs/beequeen.lua#L85-L93)、[shrouden](../../../raw/current/scripts/prefabs/shrouden.lua#L162-L171)、[蜂蜜弹](../../../raw/current/scripts/prefabs/slingshotammo.lua#L204-L213)、[黏弹](../../../raw/current/scripts/prefabs/slingshotammo.lua#L665-L676) |
| 水球投射（waterprojectiles） | `OnHitWaterstreak` 命中点不可通行（海面）时落 small2 **固定名** | 服务端；先 `waterstreak_burst`、再本水花，后 `SpreadProtection` 加湿并移除投射物 | [调用](../../../raw/current/scripts/prefabs/waterprojectiles.lua#L339-L348) |
| 蜂后卫蜂（SGbeeguard） | `spawnin` 状态 3 帧落 med1／2 | 服务端；`IsOceanAtPoint` 门禁（卫蜂在海上出生时） | [调用](../../../raw/current/scripts/stategraphs/SGbeeguard.lua#L100-L105) |
| 仅依赖声明 | beeguard／shadowwaxwell／shadow_trap／beequeen／shrouden 的 prefabs 表、oceanfishingbobber（6 名全列）、wobster `ocean_prefabs`、player_common（swim1／2） | 预载保证可生成，不证明生成；实际生成点见上表 | [player_common](../../../raw/current/scripts/prefabs/player_common.lua#L2226-L2227)、[wobster](../../../raw/current/scripts/prefabs/wobster.lua#L34-L43) |

## 典型接入

### 给海面事件接同款水花

- 前提与选择：宿主为服务端 SG 状态或组件回调；海洋门禁照抄原版 helper 写法（`TheWorld.Map:IsOceanAtPoint`），位置取世界坐标。
- 生效与撤销：一次性动画，animover 自删；无父挂、不随宿主移动。
- 原版依据：[SGshadowwaxwell TrySplashFX](../../../raw/current/scripts/stategraphs/SGshadowwaxwell.lua#L50-L56)。

```lua
local function TrySplashFX(inst)
    local x, y, z = inst.Transform:GetWorldPosition()
    if TheWorld.Map:IsOceanAtPoint(x, 0, z) then
        SpawnPrefab("ocean_splash_med"..tostring(math.random(2))).Transform:SetPosition(x, 0, z)
    end
end
```

- 调整与验证：涟漪换 `ocean_splash_ripple1～2`（无声背景层），游泳尾流换 swim 组；"前缀＋`tostring(math.random(2))`"的拼接写法在原版 9 个生成点一致，可安全复用。

## 常见陷阱与限制

- **边界：父挂本族会跟随（与 weregoose 相反）**：8 名 fn 均无解挂逻辑，若模组父挂代理，本地表现挂到代理父级并随宿主移动；原版当前全部落点式，不要照搬 weregoose 页"父挂只为取位"的结论。[fn 对比](../../../raw/current/scripts/fx.lua#L1881-L1885)
- **边界：ripple 与 swim 同 anim 同背景层**：两者都用 `no_splash`／`no_splash2` 并设背景层；差异在 swim 带压低音量的水声与 0.7 倍速，且 ripple 另调 `SetOceanBlendParams(EFFECT_TINT_AMOUNT)` 而 swim 不设海洋混合参数；"无声涟漪"语义由生成方（蜜渍路径、浮标）决定，不靠动画资源区分。[ripple 条目](../../../raw/current/scripts/fx.lua#L1920-L1938)、[swim 条目](../../../raw/current/scripts/fx.lua#L1940-L1962)
- **边界：med／small 档位语义由生成方定义**：海鱼按体型档、暗影傀儡显式传 size、浮标按种类——不是全局统一的"大小"标准；wobster／水蜘蛛固定 small1、水球固定 small2，不随机双选，按"每个消费者都用双变体"写代码会与原版表现不一致。
- **误区：依赖声明即生成**：player_common 等的 prefabs 表只是预载声明；生成点以上表逐点核验为准。
- **边界：与 splash 族的并存分工**：浮标抛入虚空落 [[reference/prefabs/splash|splash 族]] 的 `splash_ocean`、落海面才用本族；wobster 投射后落回水中的 `splash`（splash 族）与 SGwobster `launched_out_of_water` 的 `ocean_splash_small1` 是不同文件、不同注册。[void 分支](../../../raw/current/scripts/prefabs/oceanfishingbobber.lua#L72-L74)、[splash 落回水中](../../../raw/current/scripts/prefabs/wobster.lua#L79-L108)、[出水花](../../../raw/current/scripts/stategraphs/SGwobster.lua#L148-L157)
- **边界：swim 16 帧窗口绑定动画长度**：注释标明 16 帧＝swim_loop 长度；换宿主动画或改帧数时窗口需同步。[注释](../../../raw/current/scripts/stategraphs/SGwilson.lua#L26062-L26065)

## 验证记录

- 源码审阅：2026-11-01，Steam buildid 26335394。8 条目配置（bank／build／anim／sound／fn 逐项）；全 scripts 根 `ocean_splash` 定向检索 62 命中（20 文件），动态拼名四类全部核验：shadowwaxwell size 拼接、oceanfishingbobber `land_splash_fx` 前缀拼接、9 处"前缀＋`tostring(math.random(2))`"（含浮标经 def 表取前缀 1 处）、sharkboimanager `math.random(1, 2)`；生成者、门禁、端与落点逐点核验如上表，deps 声明点与生成点已区分；player_common 预载、wobster `ocean_prefabs` 与投射后落回水中分支已核；spawn 任务按源码 `mult`／倒数公式复核。简中 .po 无 8 名显示名键（定向检索无结果）。未覆盖：动画资源内容与实际时长、音效文件细节、各消费者完整状态转移图、晚加入时序。
- 游戏运行：未运行；水花视觉与音效未实测。
- 结构检查：2026-11-01 validate_page 通过（standard 档内）；validate_source_links --strict 通过（0 错误 0 警告）；示例 Lua 经 parser_probe --syntax-only 解析通过。

返回 [[reference/prefabs/fx-table-index|通用表 FX 索引]]、[[reference/prefabs/index|Prefab 索引]]。
