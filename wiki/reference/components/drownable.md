---
title: "drownable"
created: "2026-10-21"
updated: "2026-10-28"
type: component
summary: "判定实体是否位于海面或虚空并推送落水事件；下沉、伤害与清理由状态图和平台管理器分工完成。"
aliases: ["落水判定", "溺水与下沉", "落水传送", "drownable"]
tags: [component, lifecycle, state-machine, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/drownable.lua", "components/locomotor.lua", "components/walkableplatform.lua", "components/walkableplatformplayer.lua", "components/boatphysics.lua", "componentutil.lua", "constants.lua", "stategraphs/SGwilson.lua", "stategraphs/commonstates.lua", "stategraphs/SGBeefalo.lua", "stategraphs/SGboat.lua", "prefabs/player_common.lua", "prefabs/beefalo.lua", "prefabs/woodie.lua", "prefabs/carrat.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# drownable

## 快速判断

- 原版宿主与注册名：玩家角色由 `prefabs/player_common.lua` 的主模拟段统一添加，注册名是各角色 prefab（如 `wilson`）；生物侧以 `beefalo` 为例。[player_common](../../../raw/current/scripts/prefabs/player_common.lua#L2840)、[beefalo](../../../raw/current/scripts/prefabs/beefalo.lua#L1265)、[注册](../../../raw/current/scripts/prefabs/beefalo.lua#L1349-L1350)
- 职责与适用：只负责“位置 → 落水原因 → 推事件”以及落水后的传送、掉落和伤害结算；不播动画、不切状态、不销毁实体。[构造](../../../raw/current/scripts/components/drownable.lua#L1-L12)、[CheckDrownable](../../../raw/current/scripts/components/drownable.lua#L99-L109)
- 优先使用：先确认宿主已有本组件和对应状态图；生物按 `CommonStates.AddSinkAndWashAshoreStates` 注册状态，并在 SG 的 `events` 里自写 `onsink` handler。[commonstates](../../../raw/current/scripts/stategraphs/commonstates.lua#L2133-L2143)、[SGBeefalo](../../../raw/current/scripts/stategraphs/SGBeefalo.lua#L35-L45)
- 重要副作用：`CheckDrownable` 直接 `PushEvent`；`OnFallInOcean` 会丢弃激活物品和手部装备；`WashAshore` 把实体传送到岸边；`TakeDrowningDamage` 改湿度、饥饿、生命惩罚与生命/理智。[OnFallInOcean](../../../raw/current/scripts/components/drownable.lua#L259-L289)、[TakeDrowningDamage](../../../raw/current/scripts/components/drownable.lua#L339-L393)
- 修改边界：组件方法没有 `ismastersim` 守卫；原版检测入口在 `locomotor` 更新中，世界侧的 `onsink` 发送者只在主模拟端，沉没状态带 `nopredict`、不做客户端预测。[locomotor](../../../raw/current/scripts/components/locomotor.lua#L1416-L1422)、[walkableplatform](../../../raw/current/scripts/components/walkableplatform.lua#L24-L51)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `enabled` | `boolean?`，构造时为 nil | 0 秒任务后若仍为 nil 才置 true，让角色加载完成前不判落水；weregoose 在读档时写 false 表示显式关闭。[构造](../../../raw/current/scripts/components/drownable.lua#L4-L8) |
| `IsInDrownableMapBounds(x,y,z)` | 返回 `boolean` | 默认 `TheWorld.Map:IsInMapBounds`；注释说明这是留给模组按实体覆盖的判定点。[bounds](../../../raw/current/scripts/components/drownable.lua#L22-L27) |
| `IsSafeFromFalling()` | 返回 `boolean` | 依次看 `GetCurrentPlatform()`、地图边界、`IsVisualGroundAtPoint`；任一为真即安全。[safe](../../../raw/current/scripts/components/drownable.lua#L29-L44) |
| `IsOverWater()`／`IsOverVoid()` | 返回 `boolean` | 先过 `IsSafeFromFalling`，再分别查 `IsOceanTileAtPoint`／`IsInvalidTileAtPoint`。[water](../../../raw/current/scripts/components/drownable.lua#L46-L62) |
| `ShouldDrown()`／`ShouldFallInVoid()` | 返回 `boolean` | `enabled`、非无敌、位置三者同时满足才为真；有无 health 组件会影响无敌判断。[check](../../../raw/current/scripts/components/drownable.lua#L64-L89) |
| `GetFallingReason()` | `FALLINGREASON.OCEAN`／`VOID`／nil | 先判海再判虚空；常量见 [constants](../../../raw/current/scripts/constants.lua#L2199-L2203)。[reason](../../../raw/current/scripts/components/drownable.lua#L91-L97) |
| `CheckDrownable()` | 返回 `boolean` | 有原因就推 `onsink`／`onfallinvoid` 并返回 true；不检查是否已有 SG 处理者。[push](../../../raw/current/scripts/components/drownable.lua#L99-L109) |
| `SetCustomTuningsFn(fn)`／`SetOnTakeDrowningDamageFn(fn)` | `fn(inst)` 返回 tuning 表／`fn(inst, tunings)` | 前者覆盖本次伤害数值，后者在全部结算后调用；woodie 两者都用。[hooks](../../../raw/current/scripts/components/drownable.lua#L14-L20)、[woodie](../../../raw/current/scripts/prefabs/woodie.lua#L1850-L1853) |
| `OnFallInOcean(shore_x?,shore_y?,shore_z?)` | 三个 `number?`，无返回值 | 记录 `src_*`，按显式岸边点→传送点栈→`FindRandomPointOnShoreFromOcean` 选 `dest_*`；唤醒 sleeper 并丢激活/手部物品。[ocean](../../../raw/current/scripts/components/drownable.lua#L259-L289) |
| `TakeDrowningDamage()` | 无参数、无返回值 | 先按浅水倍率加湿度，再查浮具；未命中才扣饥饿/生命惩罚/生命/理智，最后调用自定义回调。[damage](../../../raw/current/scripts/components/drownable.lua#L339-L393) |
| `PushTeleportPt(src, pt)`／`PopTeleportPt(src)`／`GetTeleportPtFor(src)` | `src` 为实体或任意 key，`pt` 为 `Vector3` | 覆盖落水目标点；实体来源会自动监听 `onremove` 清理。[stack](../../../raw/current/scripts/components/drownable.lua#L197-L247) |

## 关键流程

### 谁检测

- `locomotor` 在移动处理里调用 `CheckDrownable`；没有平台且不在跳跃时也会调用，因此检测挂在移动更新上，不是独立轮询。[locomotor](../../../raw/current/scripts/components/locomotor.lua#L1738-L1744)
- `IsSafeFromFalling` 先认平台（任何 `GetCurrentPlatform()` 都算安全），再看地图边界，最后才做较贵的视觉地面检查。[safe](../../../raw/current/scripts/components/drownable.lua#L29-L44)
- 除 `locomotor` 外，世界侧也会主动推事件：平台消失时 `walkableplatform` 向成员发 `onsink`，地图临时格管理器对附近实体发同样事件；带 `drownable` 的实体会额外得到共享岸边点。[walkableplatform](../../../raw/current/scripts/components/walkableplatform.lua#L24-L51)、[componentutil](../../../raw/current/scripts/componentutil.lua#L190-L216)

### 谁改状态

- 玩家 SG 的 `onsink` handler 用 `ShouldDrown()` 复核：有 `data.boat` 进 `sink`，否则进 `sink_fast`；已死亡、已 `drowning`/`floating` 时忽略。[SGwilson](../../../raw/current/scripts/stategraphs/SGwilson.lua#L2247-L2263)
- 玩家 `sink` 的 `onenter` 调 `OnFallInOcean`（有 `shore_pt` 就传入）；动画结束的 handler 先 `StartTeleporting`（隐藏、无敌、关控制器）再 `WashAshore`。生物走 `DoWashAshore`，隐藏并置无敌后同样调 `WashAshore`。[玩家 sink](../../../raw/current/scripts/stategraphs/SGwilson.lua#L13266-L13351)、[StartTeleporting](../../../raw/current/scripts/stategraphs/SGwilson.lua#L388-L397)、[DoWashAshore](../../../raw/current/scripts/stategraphs/commonstates.lua#L2120-L2131)
- 生物用 `CommonStates.AddSinkAndWashAshoreStates` 生成 `sink`/`washed_ashore`，但 `onsink` handler 要宿主自写；`CommonHandlers.OnSink` 只是现成封装。[commonstates](../../../raw/current/scripts/stategraphs/commonstates.lua#L2110-L2131)、[SGBeefalo](../../../raw/current/scripts/stategraphs/SGBeefalo.lua#L35-L45)
- 虚空走另一条链：`onfallinvoid` 复核 `ShouldFallInVoid` 后进 `abyss_fall`，由 `OnFallInVoid` 选点。[commonstates](../../../raw/current/scripts/stategraphs/commonstates.lua#L2282-L2298)、[SGwilson](../../../raw/current/scripts/stategraphs/SGwilson.lua#L13707-L13770)

### 谁清理

- 收尾分三处：玩家 `sink` 的 `onexit` 调 `DoneTeleporting`（`Show`、取消无敌、开控制器）并恢复 HUD；生物 `sink` 的 `onexit` 恢复碰撞 mask、`Show`、无敌与阴影；`abyss_fall` 的 `onexit` 自行恢复物理、`Show`、无敌与控制器。[玩家 onexit](../../../raw/current/scripts/stategraphs/SGwilson.lua#L13354-L13369)、[生物 onexit](../../../raw/current/scripts/stategraphs/commonstates.lua#L2196-L2213)、[abyss onexit](../../../raw/current/scripts/stategraphs/SGwilson.lua#L13755-L13770)
- 物品不由 sink 统一清：`OnFallInOcean` 只丢激活物品和手部装备，`DropInventory` 由玩家 SG 的特定路径调用。[drop inventory](../../../raw/current/scripts/components/drownable.lua#L395-L420)、[SGwilson](../../../raw/current/scripts/stategraphs/SGwilson.lua#L13319)
- 传送点栈自清：实体来源在 `onremove` 时回调 `PopTeleportPt`；`GetTeleportPtOverride` 取空后把表与回调置 nil。[stack cleanup](../../../raw/current/scripts/components/drownable.lua#L226-L256)

### 与 walkableplatform／boatphysics 的分工

- `walkableplatform:OnRemoveEntity` 是船沉时给乘客发 `onsink` 的地方：有 `drownable` 的乘客拿共享 `shore_pt`，`DestroyObjectsOnPlatform` 跳过 `drownable`，乘客靠 SG 自己脱身。[platform](../../../raw/current/scripts/components/walkableplatform.lua#L24-L51)、[skip](../../../raw/current/scripts/components/walkableplatform.lua#L88-L96)
- `boatphysics` 不检测落水、不发也不监听 `onsink`；它只给平台提供 `SetHalting` 急停，沉没由 `SGboat` 的 `ready_to_snap→snapping→popping` 后 `inst:Remove()` 触发平台清理。[boatphysics](../../../raw/current/scripts/components/boatphysics.lua#L654-L664)、[SGboat](../../../raw/current/scripts/stategraphs/SGboat.lua#L121-L132)
- 分工小结：检测并推事件的是 `locomotor` 和世界管理器，改状态的是状态图，清场的是状态图 `onexit` 与平台；`drownable` 只清自己的传送点栈。

## 典型接入

### 生物落水（原版 beefalo 模式）

- 前提与选择：prefab 在主模拟段 `AddComponent("drownable")`；SG 注册 `CommonStates.AddSinkAndWashAshoreStates(states)`，并在 `events` 里写 `onsink` handler。[beefalo](../../../raw/current/scripts/prefabs/beefalo.lua#L1265)、[SGBeefalo](../../../raw/current/scripts/stategraphs/SGBeefalo.lua#L1003)
- 生效与撤销：组件存在即参与判定；`enabled=false` 可临时关闭（woodie 切到 `WEREMODES.GOOSE` 时改写），`carrat` 则直接 `RemoveComponent("drownable")` 后再加回。[woodie](../../../raw/current/scripts/prefabs/woodie.lua#L740-L755)、[carrat](../../../raw/current/scripts/prefabs/carrat.lua#L226)
- 数值与单位：默认玩家 `HEALTH/HUNGER/SANITY=25`、`WETNESS=100`，浅水惩罚 ×1/3，掉落分组 7（浅水）/5（普通）。[tuning](../../../raw/current/scripts/tuning.lua#L4311-L4326)、[drop tuning](../../../raw/current/scripts/tuning.lua#L8646-L8649)

```lua
-- 放进生物 SG 的 events 表；状态仍需 AddSinkAndWashAshoreStates 注册
EventHandler("onsink", function(inst, data)
    if inst.components.drownable ~= nil
        and not inst.sg:HasStateTag("drowning")
        and inst.components.drownable:ShouldDrown() then
        inst.sg:GoToState("sink", data)
    end
end)
```

- 调整与验证：片段只补事件入口；没有 `sink` 状态时 `GoToState` 会失败。`SGBeefalo` 还多判了死亡并给了移除分支。[SGBeefalo](../../../raw/current/scripts/stategraphs/SGBeefalo.lua#L35-L45)

## 常见陷阱与限制

- 把“在水上”当“必落水”：任何平台、地图外、视觉地面都返回安全；三者都不满足才判海/虚空。[safe](../../../raw/current/scripts/components/drownable.lua#L29-L44)
- 把 `onsink` 当“已经会沉”：handler 必须自行复核并进状态；只加组件不加 SG 时事件发出但没有任何表现。[commonstates](../../../raw/current/scripts/stategraphs/commonstates.lua#L2110-L2118)
- `enabled` 的 0 秒初始化：构造当帧是 nil；要关闭检测应在该任务前写 false，否则会被置 true。原版用 `enabled == false` 区分“显式关闭”和“尚未初始化”。[构造](../../../raw/current/scripts/components/drownable.lua#L4-L8)、[player_common](../../../raw/current/scripts/prefabs/player_common.lua#L1329-L1333)
- 无敌与无 health：`health:IsInvincible()` 为真时不判落水；没有 health 组件则不受此限制。[gate](../../../raw/current/scripts/components/drownable.lua#L64-L73)
- 浮具只跳过部分结算：`TakeDrowningDamage` 先加湿度再查浮具，所以湿度照算，饥饿/生命/理智才被跳过。[damage](../../../raw/current/scripts/components/drownable.lua#L348-L381)
- `DropInventory` 不在通用 sink 路径：只加 `drownable` 的生物不会自动掉背包，需要自己在状态里调用；`stronggrip`／`irreplaceable`／`keepondrown` 有豁免。[drop](../../../raw/current/scripts/components/drownable.lua#L395-L420)、[ShouldDropItems](../../../raw/current/scripts/components/drownable.lua#L189-L195)
- 平台指针清理不在本组件：`walkableplatformplayer` 在后续 `TestForPlatform`/`GetOffPlatform` 里才清 `platform`，而 `ShouldDrown` 依赖该指针；平台移除瞬间的引擎解绑顺序无法只靠 Lua 脚本确认。[platformplayer](../../../raw/current/scripts/components/walkableplatformplayer.lua#L236-L241)

## 验证记录

- 版本增量复核 26148036（2026-10-25）：锚点重定位 16 处（SGwilson onsink/sink/abyss 段 +49~+120、commonstates 落水状态族 +6、locomotor 移动检查 +1、player_common -14、componentutil 临时格段 +40、constants/tuning/woodie 各偏移）；无结论修订，仅锚点迁移——击退链 nointerrupt 门控与 superheavyarmor 路径（player-hubs 域 3）不在 sink／abyss 状态族内，drownable.lua、walkableplatform.lua、boatphysics.lua、SGboat.lua、SGBeefalo 引用行均未变（SGBeefalo 本版仅 667 行单行替换）。依据：player-hubs-hunk-map 41 断点偏移表＋变更清单命中 SGwilson、commonstates、locomotor、player_common、componentutil、constants、tuning、woodie、carrat（226 行内容未变）、SGBeefalo。
- 源码审阅：2026-10-21，`steam-buildid:25643519`。覆盖 `drownable.lua` 全文，以及 `locomotor`、`SGwilson`、`commonstates`、`SGBeefalo`、`walkableplatform`、`boatphysics`、`componentutil`、`player_common`、`beefalo`、`woodie`、`carrat`、`tuning`、`constants` 的相关段落；共 16 个源码文件。未覆盖音效/动画资源。
- 游戏运行：未运行；平台移除瞬间 `ShouldDrown` 复核的时序未做运行验证。
- 结构检查：`validate_page.py` 通过（0 issue，行数与字节均在上限内）；`validate_source_links.py --strict` 通过（62 处源码引用、0 error／0 warning）；`parser_probe --syntax-only` 通过（片段临时文件已删除）。

相关：[[reference/components/flotationdevice]]、[[reference/components/walkableplatform]]、[[reference/components/boatphysics]]。
