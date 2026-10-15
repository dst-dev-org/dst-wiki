---
title: "皮肤火星与随身连缀 FX 族（carnival_sparkle_bush、carrot_spinner、fan_wheel、critterbuff_lunarmoth）"
created: "2026-11-04"
updated: "2026-11-04"
type: prefab
summary: "四个跨宿主文件的外围 FX 注册：鸦年华皮肤火星、胡萝卜旋转残影、迷你风扇叶片与小蛾子发光 buff 的工厂、端、生成点、父挂与清理。"
aliases: ["carnival_sparkle_bush", "carrot_spinner", "fan_wheel", "critterbuff_lunarmoth", "皮肤火星", "鸦年华火星", "胡萝卜旋转特效", "迷你风扇叶片", "小蛾子发光", "随身连缀 FX", "外围 FX 族"]
tags: [prefab, fx, network-sync, server-client]
size_class: complex
dst_build: "steam-buildid:26335394"
sources: ["prefabs/carnival_sparkle_bush.lua", "prefabs/skinprefabs.lua", "prefabskin.lua", "prefabs/carrot_spinner.lua", "prefabs/veggies.lua", "prefabs/fan_wheel.lua", "prefabs/minifan.lua", "prefabs/critterbuff_lunarmoth.lua", "prefabs/critters.lua", "components/perishable.lua", "components/timer.lua", "components/updatelooper.lua", "stategraphs/SGwilson.lua", "entityscript.lua", "brains/cozy_bunnymanbrain.lua", "constants.lua", "prefablist.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 皮肤火星与随身连缀 FX 族（carnival_sparkle_bush、carrot_spinner、fan_wheel、critterbuff_lunarmoth）

## 快速判断

- 职责与适用：四个跨宿主文件手写的外围注册——皮肤／装备挂载的 `carnival_sparkle_bush`（鸦年华浆果丛／灌木丛帽／遮阳篷皮肤的火星），与随身连缀 FX `carrot_spinner`（旋转胡萝卜的贴地残影）、`fan_wheel`（手持迷你风扇的叶片）、`critterbuff_lunarmoth`（小蛾子的饱食发光 buff：`perish > 50%` 亮、低于则灭）。都不走 `fx.lua` 通用表（对照 [[reference/mechanisms/generic-fx-factory|通用 FX 工厂]]）。
- 优先使用：查「鸦年华火星／皮肤闪光」看 `carnival_sparkle_bush` 的皮肤映射与三处宿主 init_fn；查「胡萝卜旋转」「迷你风扇叶片」「小蛾子发光」看对应注册行与生成点。
- 重要副作用：`carnival_sparkle_bush` 与 `fan_wheel` 的可见体都是**非联网本地实体**，工厂壳只承载 Transform／net_bool（`carnival_sparkle_bush` 的壳另由宿主 `init_fn` 加 Follower）；`critterbuff_lunarmoth` 是真 Light（半径 1、强度 .6）并改宿主蛾子的 bloom／压暗／影子；`carrot_spinner` 的淡变只在主模拟。[皮肤壳](../../../raw/current/scripts/prefabs/carnival_sparkle_bush.lua#L37-L55)、[风扇壳](../../../raw/current/scripts/prefabs/fan_wheel.lua#L121-L151)、[光壳](../../../raw/current/scripts/prefabs/critterbuff_lunarmoth.lua#L75-L109)
- 修改边界：不展开宿主玩法（皮肤系统、胡萝卜旋转小游戏、迷你风扇燃料与降温、小蛾子饥饿）；四个工厂都没有 SG／Brain，`critterbuff_lunarmoth` 无 assets、无 AnimState。

官方名：`chinese_s.po` 定向检索，四个注册名都没有 `STRINGS.NAMES` 键，标题中文是已核验效果描述，非官方译名；宿主与皮肤有官方译名（浆果丛、胡萝卜、旋转的风扇、小蛾子；鸦年华灌木丛／鸦年华灌木丛帽／鸦年华遮阳棚）。[浆果丛](../../../raw/current/scripts/languages/chinese_s.po#L339414-L339417)、[胡萝卜](../../../raw/current/scripts/languages/chinese_s.po#L340654-L340657)、[小蛾子](../../../raw/current/scripts/languages/chinese_s.po#L341894-L341897)、[鸦年华灌木丛](../../../raw/current/scripts/languages/chinese_s.po#L391819-L391822)、[鸦年华灌木丛帽](../../../raw/current/scripts/languages/chinese_s.po#L394744-L394747)、[鸦年华遮阳棚](../../../raw/current/scripts/languages/chinese_s.po#L406964-L406967)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `carnival_sparkle_bush` | `fn`：Transform＋Network → FX → SetPristine → `persists=false` → 非专服 0 帧后 `PlayAnim(inst)`。`PlayAnim(proxy)` 另建**非联网**实体：`SetCanSleep(false)`、`persists=false`、`Transform:SetFromProxy(proxy.GUID)`、bank／build 同名、scale 0.75、`idle` 显式循环、`SetFinalOffset(1)`、监听 proxy `onremove` 自删、写 `proxy.fx_ent`。 | 网络壳无 AnimState，可见部分全在本地；`PlayAnim` 的 `anim`／`scale`／`flip` 形参未用（scale 被 `local scale = 0.75` 遮蔽）。[工厂](../../../raw/current/scripts/prefabs/carnival_sparkle_bush.lua#L37-L55)、[PlayAnim](../../../raw/current/scripts/prefabs/carnival_sparkle_bush.lua#L8-L31) |
| `carrot_spinner` | `fn`：Transform＋AnimState＋Network → OnGround／LAYER_BACKGROUND／sortorder 3 → `SetMultColour(1,1,1,0)` → bank／build 同名播 `idle_smear` → FX／NOCLICK → SetPristine → 客户端早退 → 主模拟 `alpha=0`、暴露 `OnTimerDone`／`AttachTo`／`FadeOut`、`timer`（`begin_delay` 0.5、`end_delay` 3.0）、`updatelooper` 每帧 `OnUpdate`、监听 `timerdone`、`persists=false`。 | 淡入每帧 2/30 至 alpha 0.6，3 秒后每帧 1/30 淡出到 <0 自删；`FadeOut` 先 `RemoveComponent("timer")` 再置淡出。[工厂](../../../raw/current/scripts/prefabs/carrot_spinner.lua#L58-L102)、[淡变](../../../raw/current/scripts/prefabs/carrot_spinner.lua#L24-L43) |
| `fan_wheel` | `fn`：Transform＋Network → FX → 非专服 0 帧后 `CreateFanWheelFX` → `net_bool _isspinning`（dirty `isspinningdirty`）→ SetPristine → 客户端早退 → 主模拟暴露 `SetSpinning`／`StartUnequipping`、`persists=false`。本地体：Transform／AnimState／SoundEmitter／Follower、`SetFourFaced`、bank／build 同名播 `idle`、`SetFinalOffset(1)`，父挂到壳的父实体并 `FollowSymbol("swap_object", 0, -114, 0)`，每帧对齐父旋转，监听 dirty 切 `spin_pre`／`spin_loop`＋`twirl` 音效，监听壳 `onremove` 自删。 | 壳无 AnimState；转动只走 net_bool，动画与音效在本地。[工厂](../../../raw/current/scripts/prefabs/fan_wheel.lua#L121-L151)、[本地体](../../../raw/current/scripts/prefabs/fan_wheel.lua#L26-L62) |
| `critterbuff_lunarmoth` | `fn`：Transform＋Light＋Network → Light falloff .65／intensity .6／radius 1／colour (0.3,0.55,0.45)／`Enable(false)`／`EnableClientModulation(true)` → FX／NOCLICK／NOBLOCK → `persists=false` → `net_smallbyte _fade`（dirty `fadedirty`）→ SetPristine → 客户端监听 `fadedirty` 后早退 → 主模拟暴露 `EnableLight` 并 `EnableLight(true)`。 | 无 assets、无 AnimState，纯 Light；淡入 5 帧（FADE_FRAMES=5，FRAMES=1/30）后每 0.1 秒闪烁。[工厂](../../../raw/current/scripts/prefabs/critterbuff_lunarmoth.lua#L75-L109)、[淡变与闪烁](../../../raw/current/scripts/prefabs/critterbuff_lunarmoth.lua#L8-L47) |

## 生成点与清理

- **`carnival_sparkle_bush`**：四个皮肤声明 `fx_prefab` → `SKIN_FX_PREFAB[skin]`，皮肤 `prefabs` 表另作加载依赖（[映射](../../../raw/current/scripts/prefabskin.lua#L4094-L4096)、[构造](../../../raw/current/scripts/prefabskin.lua#L4015-L4017)）。三处宿主 init_fn：`berrybush_init_fn` 非 placer 时生成＋`SetParent(宿主)`＋`FollowSymbol("bush_berry_build", 0, 0, 0)`，clear_fn 换肤时 `Remove`（[浆果丛](../../../raw/current/scripts/prefabskin.lua#L790-L821)）；`siestahut_init_fn` 非 placer 时延迟 `1 + math.random() * 1.2` 秒再生成＋`FollowSymbol("siesta_canopy_shadow", -125, -75, 0)`（[遮阳篷](../../../raw/current/scripts/prefabskin.lua#L1892-L1917)）；`bushhat_init_fn` 主模拟门，`equipped` 时父挂 owner＋`FollowSymbol("swap_hat", 0, -55, 0)`，`unequipped`／`onremove` 时 `Remove`（[帽子](../../../raw/current/scripts/prefabskin.lua#L1923-L1964)）。壳被删后本地体靠 proxy 的 `onremove` 自删（[监听](../../../raw/current/scripts/prefabs/carnival_sparkle_bush.lua#L28-L30)）。
- **`carrot_spinner`**：唯一生成点 `Carrot_TimerDone`（`spin` 计时结束）——`SpawnPrefab`＋`AttachTo(胡萝卜)` 复制世界坐标与旋转、监听胡萝卜 `onremove`→`FadeOut`；`Spin` 由玩家 ACTIVATE（SPIN）或兔人小游戏 `carrot:Spin(5)` 触发，装包经 `SetOnPutInInventoryFn`→`Carrot_StopSpinning`→`FadeOut`。清理：timer 淡出到 alpha<0 自删。[生成](../../../raw/current/scripts/prefabs/veggies.lua#L363-L379)、[入包](../../../raw/current/scripts/prefabs/veggies.lua#L689-L702)、[小游戏](../../../raw/current/scripts/brains/cozy_bunnymanbrain.lua#L441-L444)
- **`fan_wheel`**：唯一生成点 minifan 的 `onequip`（主模拟）——`SpawnPrefab`＋`SetParent(owner)`＋监听壳 `onremove`；`_onlocomote` 在燃料起停时调 `SetSpinning`。清理三条：①迷你风扇 `onremove`→`Remove`；②`onunequip`→`StartUnequipping`：已掉落立即 `Remove`，`item_in` 态交给 SGwilson 的 `statemem.followfx`，其他态 0 帧 `delayedremove` 等 `newstate` 判断；③无 SG 的持有者（缝纫机人台）0 帧自删。`item_in` 的 onexit 统一 `Remove` 交接的 FX。[生成](../../../raw/current/scripts/prefabs/minifan.lua#L20-L37)、[交接](../../../raw/current/scripts/prefabs/fan_wheel.lua#L80-L119)、[SG 清理](../../../raw/current/scripts/stategraphs/SGwilson.lua#L12303-L12309)
- **`critterbuff_lunarmoth`**：唯一生成点 `lunarmoth_special_powers_fn`（监听 `perishchange`）——无 buff 且 `percent > 0.5` 时 `SpawnPrefab`＋`SetParent(蛾子)`，并给蛾子开 bloom、`SetLightOverride(0.3)`、关影子；`percent < 0.5`→`EnableLight(false)`；buff 的 `onremove` 清引用并还原蛾子外观。`OnLoadPostPass` 在加载后补推一次 `perishchange`（构造时的首次推送在 POPULATING 下跳过）。清理：淡出首帧自删；蛾子移除时是裸 `SetParent`，引擎删除级联未核验，脚本内未监听。[生成](../../../raw/current/scripts/prefabs/critters.lua#L332-L358)、[初始化](../../../raw/current/scripts/prefabs/critters.lua#L268-L274)、[补推](../../../raw/current/scripts/prefabs/critters.lua#L98-L102)、[父移除](../../../raw/current/scripts/entityscript.lua#L1779-L1784)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：timer／updatelooper（`carrot_spinner`） | `StartTimer("begin_delay", .5)`／`("end_delay", 3)`，`timerdone`→`OnTimerDone`；`updatelooper` 每帧推进 alpha；`FadeOut` 里 `RemoveComponent("timer")` | 主模拟；组件在端分界后添加 | [[reference/components/timer|timer]]、[[reference/components/updatelooper|updatelooper]]；[计时](../../../raw/current/scripts/prefabs/carrot_spinner.lua#L90-L97) |
| 组件：perishable（宿主 `critter_lunarmothling`） | `perishchange` 在百分比每跨 1% 时推送，驱动 `special_powers_fn` | 主模拟；`HUNGRY_PERISH_PERCENT = 0.5` | [[reference/components/perishable|perishable]]；[推送](../../../raw/current/scripts/components/perishable.lua#L131-L137)、[阈值](../../../raw/current/scripts/prefabs/critters.lua#L6) |
| 组件：equippable／inventoryitem（宿主 minifan／carrot） | `onequip`／`onunequip` 生成与交接；`SetOnPutInInventoryFn` 停旋转 | 主模拟 | [[reference/components/equippable|equippable]]；[onequip](../../../raw/current/scripts/prefabs/minifan.lua#L20-L58) |
| 引擎 Follower（`carnival_sparkle_bush` 壳） | `AddFollower`＋`FollowSymbol` 三个宿主符号 | 生成时一次设置；这是引擎符号跟随，不是 Lua 的 follower 组件（主人、忠诚与随从关系） | [浆果丛](../../../raw/current/scripts/prefabskin.lua#L802-L807)、[遮阳篷](../../../raw/current/scripts/prefabskin.lua#L1901-L1904)、[帽子](../../../raw/current/scripts/prefabskin.lua#L1926-L1929) |
| 同步：`net_bool _isspinning`／`net_smallbyte _fade` | 前者主模拟 `set`→客户端 `isspinningdirty`→本地切动画音效；后者起止 `set` 弄脏、两端各自按 `FRAMES` 用 `set_local` 推进 | 壳联网、视觉本地；客户端听 `fadedirty` 后本地跑同一函数 | [风扇 netvar](../../../raw/current/scripts/prefabs/fan_wheel.lua#L137)、[光 netvar](../../../raw/current/scripts/prefabs/critterbuff_lunarmoth.lua#L95)、[客户端](../../../raw/current/scripts/prefabs/critterbuff_lunarmoth.lua#L99-L102)、同类光衰减见 [[reference/prefabs/wormlight|wormlight]] |
| Light（`critterbuff_lunarmoth`） | 半径 1、强度 .6、衰减 .65、色 (0.3,0.55,0.45)；主模拟 `Enable` 开关，`EnableClientModulation(true)` 让客户端光照跟随 | 双端各自插值强度／半径／衰减 | [初始化](../../../raw/current/scripts/prefabs/critterbuff_lunarmoth.lua#L82-L87)、[每帧](../../../raw/current/scripts/prefabs/critterbuff_lunarmoth.lua#L25-L31) |
| 关联 Prefab：`rabbithouse_yule_glow_fx` | 同属「皮肤 `fx_prefab` 生成并父挂」范式，行为另页维护 | 非季节门控 | [[reference/prefabs/rabbithouse_yule_glow_fx|倒挂树屋光晕特效]] |
| 资源 | 动画包 `carnival_sparkle_bush`／`carrot_spinner`／`fan_wheel`；`fan_wheel` 自身无 SOUND 声明，`twirl` 依赖 minifan 的 `sound/common.fsb`；`critterbuff_lunarmoth` 无 assets | 包内容未解包 | [assets](../../../raw/current/scripts/prefabs/minifan.lua#L1-L6) |

## 典型接入

### 用联网壳＋net_bool 驱动本地跟手 FX

- 前提与选择：需要「服务端只切开关、客户端播动画与音效」的跟手 FX 时照 `fan_wheel`：壳 `AddNetwork` 且不挂 AnimState，视觉体在非专服端本地创建、父挂到持有者并 `FollowSymbol` 对齐手部符号。
- 生效与撤销：壳的状态变化用 net_bool `set`，本地体监听 dirty 事件切动画；壳被移除时本地体随 `onremove` 自删；不要给本地体 `AddNetwork`。原版依据：[本地体](../../../raw/current/scripts/prefabs/fan_wheel.lua#L26-L62)、[开关](../../../raw/current/scripts/prefabs/fan_wheel.lua#L64-L66)。

```lua
-- 骨架：联网壳只同步开关，视觉体在本地创建并跟随持有者（取自 fan_wheel 的工厂结构）
local function ToggleSpin(fx, spin)
    if spin then
        if not fx._toggle then
            fx._toggle = true
            fx.AnimState:PlayAnimation("spin_pre")
            fx.AnimState:PushAnimation("spin_loop", true)
            fx.SoundEmitter:PlaySound("dontstarve/common/fan_twirl_LP", "twirl")
        end
    elseif fx._toggle then
        fx._toggle = false
        fx.AnimState:PlayAnimation("spin_pst")
        fx.AnimState:PushAnimation("idle")
        fx.SoundEmitter:KillSound("twirl")
    end
end

local function CreateLocalWheelFx(proxy)
    local parent = proxy.entity:GetParent()
    if parent == nil then
        return
    end
    local fx = CreateEntity()
    fx:AddTag("FX")
    fx.persists = false
    fx.entity:AddTransform()
    fx.entity:AddAnimState()
    fx.entity:AddSoundEmitter()
    fx.entity:AddFollower()
    fx.AnimState:SetBank("fan_wheel")
    fx.AnimState:SetBuild("fan_wheel")
    fx.AnimState:PlayAnimation("idle")
    fx.entity:SetParent(parent.entity)
    fx.Follower:FollowSymbol(parent.GUID, "swap_object", 0, -114, 0)
    fx:ListenForEvent("isspinningdirty", function()
        ToggleSpin(fx, proxy._isspinning:value())
    end, proxy)
    ToggleSpin(fx, proxy._isspinning:value()) -- 立即应用当前开关
    fx:ListenForEvent("onremove", function() fx:Remove() end, proxy)
end
```

- 调整与验证：符号名与偏移按目标持有者换；`spin_pre`／`spin_loop`／`spin_pst`／`idle` 与原版 `ToggleSpin` 的 pre→loop、pst→idle 顺序一致，示例已补上立即应用当前开关；资源内容未解包。示例只做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **误区：四个都只是视觉**：`critterbuff_lunarmoth` 是 Light（半径 1、强度 .6）并改宿主蛾子的 bloom／压暗／影子；`carrot_spinner` 的旋转由玩家或兔人小游戏触发，属于玩法动作的表现链。[生成](../../../raw/current/scripts/prefabs/critters.lua#L339-L355)、[小游戏](../../../raw/current/scripts/brains/cozy_bunnymanbrain.lua#L441-L444)
- **边界：`carnival_sparkle_bush` 的可见体不联网**：工厂壳只有 Transform＋Network（Follower 由宿主 `init_fn` 另加）；本地体 `SetFromProxy` 跟随；专服不生成本地体（`TheNet:IsDedicated`）。壳的删除必须由宿主做，本地体只监听壳的 `onremove`。[本地体](../../../raw/current/scripts/prefabs/carnival_sparkle_bush.lua#L8-L31)、[专服](../../../raw/current/scripts/prefabs/carnival_sparkle_bush.lua#L49-L52)
- **边界：两个皮肤 init_fn 没有主模拟门**：`berrybush_init_fn`／`siestahut_init_fn` 只在 placer 时提前返回（`basic_init_fn` 的客户端早退也只覆盖 placer），`bushhat_init_fn` 有显式 `ismastersim` 门；引擎调用点在脚本外，双端是否各生成一份未核验，复用前先双端实测重影。[basic_init_fn](../../../raw/current/scripts/prefabskin.lua#L106-L111)、[帽子门](../../../raw/current/scripts/prefabskin.lua#L1941-L1944)
- **边界：`carrot_spinner` 的透明度初始化在双端执行**：工厂在 `SetPristine` 前设 `SetMultColour(1,1,1,0)`；此后淡变才由主模拟 `updatelooper` 推进。脚本内没有客户端补正，但引擎是否继续复制 AnimState 颜色未实测，不能仅凭缺少自定义 netvar 断言客户端停在初值。[构造](../../../raw/current/scripts/prefabs/carrot_spinner.lua#L69-L82)
- **边界：`critterbuff_lunarmoth` 的淡出只走一帧**：稳态闪烁中 `EnableLight(false)` 后第一个 `FRAMES` 帧就走 `elseif _flickertask ~= nil` 分支 `Remove`，完整 5 帧淡出只在没有闪烁任务时走完；`EnableLight` 的 enable 分支还把 `2 * FRAMES + 1`（帧长 1/30）当成帧数常量，淡出途中重开会把 `_fade` 置 0（原版只在构造时调用一次）。[淡变](../../../raw/current/scripts/prefabs/critterbuff_lunarmoth.lua#L33-L46)、[分支](../../../raw/current/scripts/prefabs/critterbuff_lunarmoth.lua#L62-L66)
- **边界：`fan_wheel` 的接口与清理都只在主模拟侧闭环**：`SetSpinning` 客户端副本没有；卸下走 `StartUnequipping`，`item_in` 态交给 SG 的 `statemem.followfx`，之后迷你风扇把 `_wheel` 置 nil 不再管它。[函数](../../../raw/current/scripts/prefabs/fan_wheel.lua#L64-L66)、[交接](../../../raw/current/scripts/prefabs/fan_wheel.lua#L80-L119)、[置 nil](../../../raw/current/scripts/prefabs/minifan.lua#L39-L43)

## 验证记录

- 源码审阅：2026-11-04，steam-buildid:26335394。逐个核验 4 个注册的返回位置、工厂、端分界、生成点、父挂与清理；全 `raw/current/scripts` 以 4 个字面名反查（二进制安全检索复核，含 2 个非 UTF-8 文件；`prefabskin` 动态挂载经 `SKIN_FX_PREFAB` 与皮肤 init_fn 核验）：`carnival_sparkle_bush` 命中自身、`skinprefabs`（4 个 `fx_prefab`）、`prefablist`；`carrot_spinner` 命中自身、`veggies`、`prefablist`；`fan_wheel` 命中自身、`minifan`、`prefablist`；`critterbuff_lunarmoth` 命中自身、`critters`、`prefablist`。`chinese_s.po` 定向检索：四个注册名均无 `STRINGS.NAMES` 键，三个鸦年华皮肤键与宿主键有官方译名。本页跨 4 个工厂文件与 4 个宿主文件，含皮肤动态挂载、两套 netvar 同步与三条清理路径，需整体对照，按 complex 档。未覆盖：动画包内容与时长、客户端实际观感、皮肤 init_fn 的双端调用、运行验证。
- 游戏运行：未运行；validation 保持 source-reviewed。
- 结构检查：`validate_page.py` 通过（0 issue；106 行、约 19 KiB，`size_class: complex` 档内）；`validate_source_links.py --strict` 通过（0 error／0 warning）；Lua 片段 `parser_probe.exe --syntax-only` 通过（只查语法、未执行，临时文件已删）。

返回 [[reference/prefabs/fx-skin-index|皮肤与装备挂件]]、[[reference/prefabs/index|Prefab 索引]]。同属皮肤 `fx_prefab` 挂载范式的另见 [[reference/prefabs/rabbithouse_yule_glow_fx|倒挂树屋光晕特效]]；本页四注册都不走 [[reference/mechanisms/generic-fx-factory|通用 FX 工厂]]。
