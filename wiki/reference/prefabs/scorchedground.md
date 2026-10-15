---
title: "地面焦痕、跟随换装与附着状态的外围 FX 族（scorchedground）"
created: "2026-11-04"
updated: "2026-11-04"
type: prefab
summary: "4 文件 4 注册的散件／附属 FX：可存读的随机地面焦痕、随暗影槌换装的跟随帧层（双 netvar＋高亮／颜色同步）、姐妹骨灰罐满花时的 loop／pst 状态机，以及附着玩家的孢子炸弹（本地无网光＋3.5 秒 timer 引爆）；宿主玩法不在本页。"
aliases: ["scorchedground", "shadow_battleaxe_fx", "sisturn_moon_petal_fx", "sporebomb", "地面焦痕", "暗影槌跟随特效", "姐妹骨灰罐花瓣特效", "孢子炸弹", "散件贴地特效", "附着状态特效"]
tags: [prefab, fx, server-client]
size_class: complex
dst_build: "steam-buildid:26335394"
sources: ["prefabs/scorchedground.lua", "prefabs/shadow_battleaxe.lua", "prefabs/sisturn.lua", "prefabs/sporebomb.lua", "prefabs/toadstool.lua", "stategraphs/SGtoadstool.lua", "components/debuffable.lua", "components/debuff.lua", "components/highlightchild.lua", "components/colouraddersync.lua", "components/colouradder.lua", "components/container.lua", "components/sisturnregistry.lua", "prefabs/player_common.lua", "prefabs/player_common_extensions.lua", "prefabs/forest.lua", "prefablist.lua", "map/layouts.lua", "map/rooms/forest/DLCrooms.lua", "map/tasks/dst_tasks_forestworld.lua", "map/static_layouts/dragonfly_arena.lua", "map/graphnode.lua", "map/object_layout.lua", "widgets/fumeover.lua", "entityscript.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 地面焦痕、跟随换装与附着状态的外围 FX 族（scorchedground）

## 快速判断

- 职责与适用：本页收 4 个文件 4 个注册——`scorchedground`（世界生成贴地焦痕，10 种随机 `idle`，存读 anim／rotation）、`shadow_battleaxe_fx`（暗影槌的联网跟随层，按帧窗口在本机生成 5 个非联网帧层）、`sisturn_moon_petal_fx`（姐妹骨灰罐四朵月树花时的花瓣光效，`lunar_fx_loop`／`lunar_fx_pst` 状态机）、`sporebomb`（附着目标的孢子炸弹，本地无网光＋3.5 秒 timer 引爆）。四者都是外围表现实体，但宿主关系不同：世界生成贴片、武器跟随层、结构附着层、目标 debuff 层；除 `sporebomb` 的 debuff／timer 存活机制外，不承担伤害或状态结算。[焦痕注册](../../../raw/current/scripts/prefabs/scorchedground.lua#L72)、[跟随层注册](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L1036-L1037)、[花瓣注册](../../../raw/current/scripts/prefabs/sisturn.lua#L413-L414)、[孢子炸弹注册](../../../raw/current/scripts/prefabs/sporebomb.lua#L184)
- 优先使用：永久或可存读的地面焦痕用 `scorchedground`；跟随武器换装看 `shadow_battleaxe_fx` 的帧窗口与 netvar 协议；姐妹骨灰罐的视觉状态看 `sisturn_moon_petal_fx`；给目标挂孢子炸弹走 `AddDebuff("sporebomb", "sporebomb")`，不要直接 `SpawnPrefab` 后手动挂父级。
- 重要副作用：`sporebomb` 的 timer 到期或 debuff 被移除时，在目标位置生成 `sporecloud`，其伤害／腐败玩法归 [[reference/prefabs/sleepcloud|睡眠云与孢子云]]；`sisturn_moon_petal_fx` 生成时推世界事件 `moon_blossom_sisturn`（快照内无监听），跨系统状态实际走 `ms_updatesisturnstate`／[[reference/components/sisturnregistry|sisturnregistry]]。[引爆生成](../../../raw/current/scripts/prefabs/sporebomb.lua#L70-L84)
- 修改边界：四者的层位／排序、帧窗口、动画名、光参数与 timer 名都是硬编码，不是通用 FX 协议；端分界也不同（`scorchedground`、花瓣、孢子炸弹客户端早退，跟随层另有客户端 dirty 监听）。宿主玩法（暗影槌等级／吸血、姐妹骨灰罐技能、毒菌蟾蜍 AI）不在本页展开。
- 尺寸档：`complex`。4 个注册来自 4 个宿主／工厂，世界生成贴片、武器跟随层、结构附着状态机、debuff 附着与 timer 引爆的端分界、父挂和清理各不相同，需要在一页内横向对照；不是按源码行数升档。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `scorchedground` | `fn`：Transform／AnimState／Network → bank／build `scorched_ground` → `ANIM_ORIENTATION.OnGround`＋`LAYER_BACKGROUND`＋sort 3 → `NOCLICK`／`FX` → `SetPristine` → 客户端早退 → 主模拟 10 选 1 `PlayAnimation`＋随机 0–360° → 挂 `OnSave`／`OnLoad`（anim、rotation；scale 已注释） | 可存档的纯贴片；`persists` 默认 true，随机只发生在主模拟首次构造，读档由 OnLoad 覆盖；随机缩放 1–1.33 被注释，实际不缩放。[工厂](../../../raw/current/scripts/prefabs/scorchedground.lua#L36-L70)、[动画表](../../../raw/current/scripts/prefabs/scorchedground.lua#L6-L13)、[存读](../../../raw/current/scripts/prefabs/scorchedground.lua#L15-L34)、[注册](../../../raw/current/scripts/prefabs/scorchedground.lua#L72) |
| `shadow_battleaxe_fx` | `FollowSymbolFxFn`：Transform／AnimState／Follower／Network → `FX` → bank／build `nightmare_axe` 播 `swap_level1_f3`（帧 3 兼作漂浮）→ LightOverride／符号高亮 → `highlightchild`＋`colouraddersync` → `_level` net_tinybyte（`leveldirty`，初值 1）＋`equiptoggle` net_bool（`equiptoggledirty`）→ SetPristine → 客户端监听两个 dirty → 主模拟挂 `SetFxLevel`／`ToggleEquipped`，`persists=false` | 联网层只有帧 3；装备后客户端再生成 5 个非联网帧层（f1／f4／f6／f7／f8；f4 固定 level 1）并共享 owner 的 `swap_object` 符号。[工厂](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L988-L1031)、[帧表](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L869-L877)、[本机帧工厂](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L879-L902)、[注册](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L1036-L1037) |
| `sisturn_moon_petal_fx` | `fxfn`：Transform／AnimState／Network／Follower／SoundEmitter → `FX` → bank／build `sisturn` → `lunar_fx_pre`＋`lunar_fx_loop` → 循环声 `loop` → SetPristine → 客户端早退 → 主模拟 `animover` 状态机，`persists=false` | `done` 由宿主设置；done 后 loop 播完切 pst，pst 播完才 Remove；重新激活时若已在 pst，回放 pre／loop。无技能／玩法组件，纯视觉。[工厂](../../../raw/current/scripts/prefabs/sisturn.lua#L365-L411)、[状态机](../../../raw/current/scripts/prefabs/sisturn.lua#L389-L406)、[注册](../../../raw/current/scripts/prefabs/sisturn.lua#L413-L414) |
| `sporebomb` | `fn`：Transform／Follower／AnimState／Network → `FX`／`NOCLICK` → bank／build `sporebomb` 播 `sporebomb_pre` → `_light` 非联网光源＋`_fade` net_smallbyte（`fadedirty`）＋周期淡入 → `OnRemoveEntity`／`OnInit` → SetPristine → 客户端监听 `fadedirty`＋`OnEntityReplicated` → 主模拟 `sporebomb_loop`＋`debuff`（Attach／Detach／ChangeFollowSymbol）＋`timer`（`explode`，`TUNING.TOADSTOOL_SPOREBOMB_TIMER`＝3.5） | 附着型 debuff 实体；不靠自身网络光，光在每个端本地创建，复制后重挂到目标；timer 到点 `debuff:Stop()`。[工厂](../../../raw/current/scripts/prefabs/sporebomb.lua#L132-L182)、[光](../../../raw/current/scripts/prefabs/sporebomb.lua#L92-L110)、[timer](../../../raw/current/scripts/prefabs/sporebomb.lua#L177-L179)、[注册](../../../raw/current/scripts/prefabs/sporebomb.lua#L184) |

`languages/chinese_s.po` 对 4 个注册名均无 `STRINGS.NAMES.*` 命中；命中仅宿主名：`SHADOW_BATTLEAXE`＝暗影槌（[L349244-L349247](../../../raw/current/scripts/languages/chinese_s.po#L349244-L349247)）、`SISTURN`＝姐妹骨灰罐（[L349409-L349412](../../../raw/current/scripts/languages/chinese_s.po#L349409-L349412)）。标题与别名中的中文是已核验效果描述，不是官方译名；`scorchedground`／`sporebomb` 无名称键。

## 关联机制导航

| 注册 | 生成点／父挂 | 清理与端 |
|---|---|---|
| `scorchedground` | 世界生成：`DragonflyArena` 静态布局 30 处 `type = "scorchedground"`（[L179-L522](../../../raw/current/scripts/map/static_layouts/dragonfly_arena.lua#L179-L522)）→ 房间 `countstaticlayouts`（[房间](../../../raw/current/scripts/map/rooms/forest/DLCrooms.lua#L60-L64)、[任务](../../../raw/current/scripts/map/tasks/dst_tasks_forestworld.lua#L129)）与 `StaticLayout.Get` 接线（[L899-L904](../../../raw/current/scripts/map/layouts.lua#L899-L904)）→ `Node:ConvertGround`（[L259-L270](../../../raw/current/scripts/map/graphnode.lua#L259-L270)）→ `object_layout` 的 `add_entity.fn`（[L479-L482](../../../raw/current/scripts/map/object_layout.lua#L479-L482)）写入 `entitiesOut`（[L188-L218](../../../raw/current/scripts/map/graphnode.lua#L188-L218)）；无父挂 | 无自身清理；`persists` 默认 true，世界存档按 anim／rotation 存读。`forest.lua` 世界依赖与 `prefablist` 的 `PREFABFILES` 只保证可用，不是生成点。[依赖](../../../raw/current/scripts/prefabs/forest.lua#L297)、[总表](../../../raw/current/scripts/prefablist.lua#L1091)、[默认 persists](../../../raw/current/scripts/entityscript.lua#L222) |
| `shadow_battleaxe_fx` | 宿主物品构造时 `SpawnPrefab`（[L814-L822](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L814-L822)）；`SetFxOwner(nil)` 挂到物品实体、跟随 `swap_spear`、关闭装备态；装备时 `SetFxOwner(owner)` 挂到持有者、跟随 `swap_object`、`highlightchild:SetOwner(owner)`、`ToggleEquipped(true)`，有 colouradder 则 `AttachChild`（[L46-L61](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L46-L61)、[L155-L178](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L155-L178)） | 卸下 `SetFxOwner(nil)`：DetachChild、`ToggleEquipped(false)` 触发 `FxRemoveAll` 清 5 个本机帧层（[L63-L80](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L63-L80)、[L941-L946](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L941-L946)）；装备态下 FX 自身 `OnRemoveEntity=FxRemoveAll`；`persists=false`，物品读档重建后由 `SetLevel` 同步等级（[L205-L207](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L205-L207)）。物品实体被直接移除时 `OnRemoveEntity` 只处理 `_classified`（[L31-L42](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L31-L42)），跟随 FX 的显式回收未见，未核验 |
| `sisturn_moon_petal_fx` | 宿主 `update_abigail_status`：容器 4 格全满且 `getsisturnfeel()=="BLOSSOM"`（4 朵 `moon_tree_blossom`）→ `SpawnPrefab`＋`AddChild`（[L146-L154](../../../raw/current/scripts/prefabs/sisturn.lua#L146-L154)）；`AddChild` 设 `parent` 并挂实体（[L921-L935](../../../raw/current/scripts/entityscript.lua#L921-L935)） | 条件失效→`done=true`（[L165-L168](../../../raw/current/scripts/prefabs/sisturn.lua#L165-L168)）；主模拟 `animover`：pst 播完→`parent.lune_fx=nil`＋Remove，loop 播完且 done→pst（[L389-L406](../../../raw/current/scripts/prefabs/sisturn.lua#L389-L406)）。父实体 Remove 时 `children` 循环连带移除（[L1779-L1783](../../../raw/current/scripts/entityscript.lua#L1779-L1783)）；锤毁直接 Remove 父实体（[L52-L62](../../../raw/current/scripts/prefabs/sisturn.lua#L52-L62)）；`persists=false`，读档由容器装填的 `itemget` 重建（[容器 OnLoad](../../../raw/current/scripts/components/container.lua#L996-L1005)） |
| `sporebomb` | 毒菌蟾蜍 `DoSporeBomb` 对每个目标 `AddDebuff("sporebomb", "sporebomb")`（[L239-L243](../../../raw/current/scripts/prefabs/toadstool.lua#L239-L243)），由 SG `sporebomb` 状态第 22 帧触发（[L803-L807](../../../raw/current/scripts/stategraphs/SGtoadstool.lua#L803-L807)）；`AddDebuff` 经 `debuffable` 生成并 `AttachTo`，`RegisterDebuff` 置 `persists=false`（[L2140-L2154](../../../raw/current/scripts/entityscript.lua#L2140-L2154)、[L95-L108](../../../raw/current/scripts/components/debuffable.lua#L95-L108)、[L72-L89](../../../raw/current/scripts/components/debuffable.lua#L72-L89)）。`OnAttached` 把本体与本地光 `SetParent(target.entity)`、`FollowSymbol`、每帧对齐朝向（[L59-L68](../../../raw/current/scripts/prefabs/sporebomb.lua#L59-L68)） | timer `explode` 3.5 秒→`timerdone`→`debuff:Stop()`（[L28-L32](../../../raw/current/scripts/components/debuff.lua#L28-L32)）→目标 `RemoveDebuff`→OnDetach 在目标位置生成 `sporecloud`＋`FadeInImmediately`＋Remove（[L70-L84](../../../raw/current/scripts/prefabs/sporebomb.lua#L70-L84)、[L177-L179](../../../raw/current/scripts/prefabs/sporebomb.lua#L177-L179)、[数值](../../../raw/current/scripts/tuning.lua#L2507)）；`OnRemoveEntity` 移除本地光（[L119-L123](../../../raw/current/scripts/prefabs/sporebomb.lua#L119-L123)）。目标死亡 `CommonPlayerDeath` 关闭 debuffable、移除全部 debuff（[L648](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L648)），仍走同一 OnDetach；玩家 despawn 走 `RemoveOnDespawn`（[L1434](../../../raw/current/scripts/prefabs/player_common.lua#L1434)） |

- 跟随层同步链：`highlightchild` 用 `syncowner`（net_entity）同步高亮归属，客户端加入 `owner.highlightchildren`（[L24-L50](../../../raw/current/scripts/components/highlightchild.lua#L24-L50)）；`colouraddersync` 用 `colour`（net_uint）同步颜色：`AttachChild` 把 RGBA 写入该 netvar，本机再经 `colourchangedfn` 刷到 5 个帧层（[L24-L57](../../../raw/current/scripts/components/colouradder.lua#L24-L57)、[L1-L49](../../../raw/current/scripts/components/colouraddersync.lua#L1-L49)）。玩家在 `player_common` 常驻 `colouradder`（[L2669](../../../raw/current/scripts/prefabs/player_common.lua#L2669)），所以装备态的颜色分支实际会走。
- 事件与 HUD：`sporebomb` 的 `OnInit` 向父实体推 `startfumedebuff`，由 `FumeOver` 接收做屏幕雾效（[L125-L130](../../../raw/current/scripts/prefabs/sporebomb.lua#L125-L130)、[L106-L113](../../../raw/current/scripts/widgets/fumeover.lua#L106-L113)）；这不是玩法伤害。`sisturn_moon_petal_fx` 的生成／停止各推一次 `moon_blossom_sisturn`（[L150](../../../raw/current/scripts/prefabs/sisturn.lua#L150)、[L165](../../../raw/current/scripts/prefabs/sisturn.lua#L165)），但全 `raw/current/scripts` 反查无监听；实际跨系统状态由 `ms_updatesisturnstate`（[L181](../../../raw/current/scripts/prefabs/sisturn.lua#L181)）进入 registry。

## 典型接入

### 生成可存读的地面焦痕

- 前提与选择：主模拟生成，用 `scorchedground` 是因为它自带 `OnSave`／`OnLoad`；客户端工厂早退，不需要客户端生成。
- 生效与撤销：设置 `decal.anim` 后再 `PlayAnimation`，否则读档会回到构造时的随机 `inst.anim`；rotation 直接设 Transform；实体不自删，撤掉需自行 `Remove`。
- 原版依据：[工厂](../../../raw/current/scripts/prefabs/scorchedground.lua#L36-L70)、[存读](../../../raw/current/scripts/prefabs/scorchedground.lua#L15-L34)、[世界生成](../../../raw/current/scripts/map/static_layouts/dragonfly_arena.lua#L179-L522)。

```lua
local function SpawnScorchedDecal(x, z, anim, rotation)
    if not TheWorld.ismastersim then
        return nil
    end

    local decal = SpawnPrefab("scorchedground")
    decal.Transform:SetPosition(x, 0, z)

    if anim ~= nil then
        decal.anim = anim -- OnSave 读取此字段；只 PlayAnimation 不会存下来
        decal.AnimState:PlayAnimation(anim)
    end

    if rotation ~= nil then
        decal.Transform:SetRotation(rotation)
    end

    return decal
end
```

- 调整与验证：`anim` 取 `idle`、`idle2`…`idle10`；rotation 单位为度；原版布局实例没有 properties，随机值全由工厂完成。生成前需保证 prefab 已加载（原版 `PREFABFILES` 与森林世界依赖已含）；模组加载方式不在本页核验范围。

### 给目标挂孢子炸弹

- 前提与选择：只在主模拟、目标有效且未死亡时调用；`AddDebuff` 会在缺省时补 `debuffable`，再用 `DebuffsEnabled`／死亡检查 [过滤](../../../raw/current/scripts/entityscript.lua#L2140-L2154)。
- 生效与撤销：debuff 自行计时 3.5 秒并在结束时生成 `sporecloud`；不要手动挂父级或提前 Remove，否则跳过 `OnDetach` 的云生成。

```lua
local function ApplySporeBomb(target)
    if not TheWorld.ismastersim or target == nil or not target:IsValid() then
        return false
    end
    return target:AddDebuff("sporebomb", "sporebomb") -- 返回 true／false
end
```

- 调整与验证：时长取 `TUNING.TOADSTOOL_SPOREBOMB_TIMER`；原版由毒菌蟾蜍 `DoSporeBomb` 在 SG 第 22 帧调用（[L239-L243](../../../raw/current/scripts/prefabs/toadstool.lua#L239-L243)、[L803-L807](../../../raw/current/scripts/stategraphs/SGtoadstool.lua#L803-L807)）。本页未运行游戏，云的实际落点与持续时间未实测。

## 常见陷阱与限制

- **误区：焦痕只播动画就能存读**：`OnSave` 存的是 `inst.anim` 与 `Transform:GetRotation()`，只调用 `PlayAnimation` 不写 `inst.anim`，读档会回退到构造时的随机值；随机缩放已注释，别照注释以为会缩放。[存读](../../../raw/current/scripts/prefabs/scorchedground.lua#L15-L34)
- **边界：跟随层注册没有 assets**：`Prefab("shadow_battleaxe_fx", FollowSymbolFxFn)` 未传资源表（[L1036-L1037](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L1036-L1037)），bank／build 依赖宿主物品声明的 `nightmare_axe` 资源（[L1-L8](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L1-L8)）；单独加载 FX 而不加载武器资源会缺动画。
- **边界：同名 `inst.fx` 含义不同**：物品上的 `inst.fx` 是联网 FX 实体（[L819](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L819)），FX 上的 `inst.fx` 是本机 5 帧层数组（[L904-L908](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L904-L908)）；改代码或 Hook 时不要混用这两个字段。
- **边界：花瓣 FX 的客户端不自行收尾**：`done` 只在主模拟读写，客户端 `SetPristine` 后早退、不监听 `animover`（[L385-L387](../../../raw/current/scripts/prefabs/sisturn.lua#L385-L387)）；客户端只是按复制状态播动画，Remove 由服务器发起。
- **边界：`moon_blossom_sisturn` 不是状态源**：快照内只有两处推送、没有监听；别把它当作 Wendy／Abigail 的跨系统状态判断依据，实际判断走 `ms_updatesisturnstate` 与 registry。[推送](../../../raw/current/scripts/prefabs/sisturn.lua#L150)、[registry 监听](../../../raw/current/scripts/components/sisturnregistry.lua#L75)
- **边界：孢子炸弹的光只有淡入**：`FadeOut` 已定义但本文件无调用，`_fade` 只用 `set_local` 更新，客户端 `fadedirty` 监听在快照内没有触发者；拆除时直接 Remove，不会淡出。[FadeOut](../../../raw/current/scripts/prefabs/sporebomb.lua#L44-L49)、[淡入](../../../raw/current/scripts/prefabs/sporebomb.lua#L16-L35)、[netvar](../../../raw/current/scripts/prefabs/sporebomb.lua#L153)

## 验证记录

- 源码审阅：2026-11-04，Steam buildid 26335394。4 个文件的 4 个 FX 注册逐项核对工厂、端分界、生成点、父挂与清理；全 `raw/current/scripts` 反查注册名：`scorchedground` 仅命中 `map/static_layouts/dragonfly_arena.lua` 的 30 处布局、`prefablist.lua:1091`、`prefabs/forest.lua:297` 与注册行；`shadow_battleaxe_fx` 除依赖 L12、生成 L819、注册 L1037 外，还有 L1012／L1015 两处 netvar 前缀；`sisturn_moon_petal_fx` 仅命中 `prefabs/sisturn.lua` 的依赖 L6、生成 L153、注册 L414；`sporebomb` 除 `toadstool.lua`（依赖 L38、`DoSporeBomb` L241、过滤 L206／L223）、`SGtoadstool.lua`（状态 L104／L775／L805／L806）、`prefablist.lua:1212` 与注册行外无其他；另有 `cave_vents.lua:14` 被注释的 `sporebomb_projectile` 近似名，不是本注册。拼接名检索未发现额外生成点；`.po` 仅命中两个宿主名。未覆盖：动画包内容／帧长、贴地视觉对齐、跨端实际渲染、模组加载路径。
- 游戏运行：未运行。
- 结构检查：`validate_page.py`、`validate_source_links.py --strict` 通过；Lua 示例经 `parser_probe --syntax-only` 通过。

相关：[[reference/prefabs/crabking_claw_shadow|地面贴片与投影阴影 FX 族]]（可存读贴地／层位贴片对照）、[[reference/prefabs/sleepcloud|睡眠云与孢子云]]（`sporecloud` 的玩法与清理）、[[reference/prefabs/fx-world-index|特效 Prefab 索引·环境与世界]]。

返回 [[reference/prefabs/fx-world-index|特效 Prefab 索引·环境与世界]]。
