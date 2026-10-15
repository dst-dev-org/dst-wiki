---
title: "小型一次性外围 FX 族（grasspartfx、脆弱玫瑰、黑暗痕迹等 8 注册）"
created: "2026-11-04"
updated: "2026-11-04"
type: prefab
summary: "七个宿主文件内手写的 8 个小型一次性注册：草蜥蜴替身草屑、鱼饵碎块、恶液吐落物跟随、无眼鹿挣脱／生角、渔网开网溅水与温娜脆弱玫瑰／黑暗痕迹的工厂、端、生成点与清理。"
aliases: ["grasspartfx", "chumpiece", "gelblob_item_fx", "deer_unshackle_fx", "deer_growantler_fx", "fishingnetvisualizerfx", "charlierose", "charlieresidue", "脆弱玫瑰", "黑暗痕迹", "草屑", "草蜥蜴变身特效", "鱼饵碎块", "恶液吐物品", "无眼鹿挣脱锁链", "无眼鹿生角", "渔网溅水", "玫瑰眼镜残影"]
tags: [prefab, fx, network-sync, server-client]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/charlierose.lua", "prefabs/charlieresidue.lua", "prefabs/chum_aoe.lua", "prefabs/grass.lua", "prefabs/gelblob.lua", "prefabs/deer.lua", "prefabs/fishingnetvisualizerfx.lua", "prefabs/fishingnetvisualizer.lua", "prefabs/flower.lua", "prefabs/waterplant_pollen.lua", "prefabs/player_common_extensions.lua", "prefabs/winona.lua", "stategraphs/SGdeer.lua", "stategraphs/SGfishingnetvisualizer.lua", "stategraphs/SGwilson.lua", "components/roseinspectableuser.lua", "components/edible.lua", "components/eater.lua", "components/container.lua", "components/inventory.lua", "brains/oceanfishbrain.lua", "brains/deergemmedbrain.lua", "components/deerherdspawner.lua", "prefabs/oceanfishdef.lua", "prefabs/fishingnet.lua", "actions.lua", "simutil.lua", "tuning.lua", "languages/chinese_s.po"]
size_class: complex
reliability: verified
validation: source-reviewed
status: active
---

# 小型一次性外围 FX 族（grasspartfx、脆弱玫瑰、黑暗痕迹等 8 注册）

## 快速判断

- 职责与适用：7 个宿主文件内手写的 8 个小型一次性注册——草被草蜥蜴顶替时的草屑（`grasspartfx`）、鱼饵云碎块（`chumpiece`）、恶液吐落物品的跟随黏液（`gelblob_item_fx`）、无眼鹿挣脱／生角（`deer_unshackle_fx`／`deer_growantler_fx`）、渔网开网溅水（`fishingnetvisualizerfx`），以及温娜玫瑰链的 `charlierose`（脆弱玫瑰）与 `charlieresidue`（黑暗痕迹）。都不是 `fx.lua` 通用表条目，公共工厂协议不适用（对照 [[reference/mechanisms/generic-fx-factory|通用 FX 工厂]]）。
- 优先使用：查「草屑／草蜥蜴变身」「鱼饵碎块／海鱼吃饵」「恶液吐物品」「鹿挣脱锁链／生角」「渔网开网水花」「脆弱玫瑰／黑暗痕迹」按下方注册表与生成点定位；宿主玩法分别见 [[reference/components/roseinspectable|roseinspectable]]、[[reference/components/fishingnetvisualizer|fishingnetvisualizer]] 等页。
- 重要副作用：`charlierose` 是燃料（`TINY_FUEL`）且温娜 `winona_charlie_2` 死亡时会消耗背包里的它保命；`charlieresidue` 是 CLASSIFIED 地图／交互目标，其残影 FX 是非联网的本地实体；`chumpiece` 挂 `edible`（`edible_meat`），是海鱼真正吃掉的鱼饵块。三项都不是纯视觉。
- 修改边界：`grasspartfx` 是唯一没有显式关闭持久化的注册；`fishingnetvisualizerfx` 是整条渔网链唯一联网实体；本页不展开宿主玩法（草／鱼饵云／恶液／鹿群／渔网捕获／玫瑰眼镜流程）。

官方名：`chinese_s.po` 有 `STRINGS.NAMES.CHARLIEROSE`＝脆弱玫瑰、`STRINGS.NAMES.CHARLIERESIDUE`＝黑暗痕迹；其余 6 个注册名无 `STRINGS.NAMES` 键，标题与表内中文为已核验效果描述，非官方译名。[黑暗痕迹](../../../raw/current/scripts/languages/chinese_s.po#L340804-L340807)、[脆弱玫瑰](../../../raw/current/scripts/languages/chinese_s.po#L340809-L340812)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `charlierose` | `fn`：Transform＋AnimState＋SoundEmitter＋Network＋`MakeInventoryPhysics` → bank／build `flowers` 播 `rose`、FX／NOCLICK → SetPristine → 客户端早退 → 主模拟 `inspectable`＋`inventoryitem`（`canbepickedup=false`、`canonlygoinpocket`、`nobounce`）＋`fuel`（`TINY_FUEL`）→ 构造末尾直接调 `OnDropped`。 | 可拾取性被关掉、只能直接 GiveItem 进背包；不父挂。掉落播 `rose_shatter`、animover→`ErodeAway`，睡眠 2 秒直接移除。[工厂](../../../raw/current/scripts/prefabs/charlierose.lua#L54-L92)、[掉落](../../../raw/current/scripts/prefabs/charlierose.lua#L40-L52) |
| `charlieresidue` | `fn`：Transform＋Network → `CLASSIFIED` → `net_tinybyte` `_animstate`／`_mapactioncontext`、`net_entity` `_target`、`valid_map_actions={ACTIVATE}` → SetPristine → 非主模拟 `InitClientFX`＋监听 `animstatedirty`／`targetdirty`（`mapactioncontextdirty` 的监听被注释）；主模拟 `inspectable`＋`activatable`、`SetClassifiedTarget(inst)`、0 秒自删任务直到 `SetFXOwner` 认领。 | 显示端 FX 由 `CreateFX` 建为**非联网**子实体并父挂残影（`highlightchildren`）：先播 `pre`，再按网络动画态切 `idle`（循环）；`pst` 时脱父、停 `residueloop` 并自删。[工厂](../../../raw/current/scripts/prefabs/charlieresidue.lua#L180-L226)、[客户端 FX](../../../raw/current/scripts/prefabs/charlieresidue.lua#L18-L40)、[退出](../../../raw/current/scripts/prefabs/charlieresidue.lua#L107-L128) |
| `chumpiece` | `chumpiecefn`：仅 Transform＋Network（AnimState／bank／build 整段被注释）→ FX／NOCLICK → SetPristine → 主模拟 `persists=false`＋`edible`（`secondaryfoodtype=FOODTYPE.MEAT`，加 `edible_meat` 标签）。 | 无动画、无物理、不父挂；不是给玩家捡的食物，而是海鱼 EAT 的目标块。[工厂](../../../raw/current/scripts/prefabs/chum_aoe.lua#L184-L211)、[注册](../../../raw/current/scripts/prefabs/chum_aoe.lua#L213-L214) |
| `grasspartfx` | `grasspart_fn`：Transform＋AnimState＋SoundEmitter＋Network → bank `grass`／build `grass1` 播 `grass_part`、`SetFinalOffset(1)` → FX／NOCLICK → **SetPristine 前**播 `dontstarve/wilson/pickup_reeds` → 主模拟监听 `animover`→`Remove`。 | 联网实体但无父挂；拾取音在各模拟的 pristine 前播放。[工厂](../../../raw/current/scripts/prefabs/grass.lua#L292-L319)、[注册](../../../raw/current/scripts/prefabs/grass.lua#L321-L323) |
| `gelblob_item_fx` | `itemfn`：Transform＋AnimState＋Network → FX／NOCLICK → bank／build `gelblob` 播循环 `splash_loop`、`SetFinalOffset(2)` → SetPristine → 主模拟 `persists=false`＋暴露 `KillFX`。 | 服务端父挂掉落物；`KillFX` 脱父、y 归零、播 `splash_impact` 后 animover→Remove。[工厂](../../../raw/current/scripts/prefabs/gelblob.lua#L919-L945)、[KillFX](../../../raw/current/scripts/prefabs/gelblob.lua#L905-L917) |
| `deer_unshackle_fx` | `unshackle_fn`：Transform＋AnimState＋SoundEmitter＋Network、`SetSixFaced` → bank `deer_unshackle`／build `deer_build` 播 `unshackle_pst`、`SetFinalOffset(1)` → 冬季盛宴覆盖项圈／锁链符号并播铃，否则 `Hide("puff")` → 锁链声＋`lock_break` → FX／NOCLICK → 主模拟 `persists=false`、15 帧补锁链声、2 秒后 `ErodeAway`。 | 不父挂，只复制鹿的位置与旋转；冬季盛宴额外 14／16 帧补铃与铃闲音。[工厂](../../../raw/current/scripts/prefabs/deer.lua#L620-L668) |
| `deer_growantler_fx` | `growantler_fn`：Transform＋AnimState＋Network、`SetSixFaced` → 同 bank／build 播 `growantler_pst`、`SetFinalOffset(1)` → FX／NOCLICK → 主模拟 `persists=false`、2 秒后 `ErodeAway`。 | 无 SoundEmitter、无音效；不父挂。[工厂](../../../raw/current/scripts/prefabs/deer.lua#L670-L698) |
| `fishingnetvisualizerfx` | `fn`：Transform＋AnimState＋SoundEmitter＋Network → `persists=false` → `NOBLOCK`／FX（无 NOCLICK）→ bank／build `boat_net_fx` 播 `hit`、`SetSortOrder(5)` → `animover`→`Remove` 的监听在 SetPristine 与端分界**之前**注册。 | 不父挂、落点 y=0；双端都挂 animover；AddSoundEmitter 但工厂内不播任何音。[工厂](../../../raw/current/scripts/prefabs/fishingnetvisualizerfx.lua#L10-L37)、[注册](../../../raw/current/scripts/prefabs/fishingnetvisualizerfx.lua#L39) |

## 生成点与清理

- **`charlierose`**：唯一生成点是花的玫瑰检查激活回调——`SpawnPrefab("charlierose")` 后 `inventory:GiveItem`（花仍是玫瑰且检查者有背包时）。掉落即 `OnDropped`：`persists=false`、播 `rose_shatter`、下一 tick 播 `rose_fall_shatter`、`animover`→`ErodeAway`；睡眠 2 秒直接移除；`OnPutInInventory` 取消碎裂与音效任务、改播 `rose`、恢复 `persists`。[生成](../../../raw/current/scripts/prefabs/flower.lua#L125-L134)、[背包切换](../../../raw/current/scripts/prefabs/charlierose.lua#L20-L33)
- **`charlieresidue`**：唯一生成点是 `roseinspectableuser:SpawnResidue`——先衰退旧残影，再 Spawn、设目标或点位、`SetFXOwner(owner)`。`SetFXOwner` 取消 0 秒自删任务、设 CLASSIFIED 目标、监听 owner 移除，有玫瑰眼镜视觉则进入 idle，否则 `Decay`；`OnActivate` 先 Decay 再回调使用者的 `OnCharlieResidueActivated`。清理：Decay 0.5 秒 Remove、`OnEntitySleep=Remove`、owner 移除→Remove。[生成](../../../raw/current/scripts/components/roseinspectableuser.lua#L90-L109)、[认领与门禁](../../../raw/current/scripts/prefabs/charlieresidue.lua#L80-L105)、[激活](../../../raw/current/scripts/prefabs/charlieresidue.lua#L142-L149)
- **`chumpiece`**：两处生成，均要求落点 `IsOceanAtPoint`——`chum_aoe:SpawnChumPieces`（0.5 秒周期、上限 7、宿主 `persists` 时补位）与 `waterplant_pollen_fx_ocean:spawn_chum_piece`（1 秒周期、上限 5、环宽 2–4）。块自身没有 timer／animover，只能被海鱼吃掉或随宿主 `onremove` 清表；`persists=false`。[鱼饵云](../../../raw/current/scripts/prefabs/chum_aoe.lua#L88-L105)、[补位](../../../raw/current/scripts/prefabs/chum_aoe.lua#L76-L86)、[水草花粉](../../../raw/current/scripts/prefabs/waterplant_pollen.lua#L100-L118)、[清表](../../../raw/current/scripts/prefabs/waterplant_pollen.lua#L134-L140)
- **`grasspartfx`**：唯一生成点是 `grass.lua` 的 `onmorphtimer`——草变草蜥蜴成功（`canmorph`＋密度检查通过）时先生成 `grassgekko`，再在**同一坐标、同旋转**生成 `grasspartfx`，复制草的 `MultColour`，随后 `inst:Remove()`。`grass` 与 `depleted_grass` 都可在移植后进入该计时链。[变身计时](../../../raw/current/scripts/prefabs/grass.lua#L138-L161)、[可变身化](../../../raw/current/scripts/prefabs/grass.lua#L163-L171)
- **`gelblob_item_fx`**：唯一生成点是恶液的 `OnDropItem`（inventory 的 `dropitem` 事件）——物品尚未落地时生成、`SetParent(物品)`，物品 `on_landed`→`KillFX`、`onputininventory`→Remove。宿主吞人／死亡时会 `DropEverything` 触发该事件（[吞人](../../../raw/current/scripts/prefabs/gelblob.lua#L293)、[死亡](../../../raw/current/scripts/prefabs/gelblob.lua#L320)）；`KillFX` 在睡眠时直接 Remove。[生成](../../../raw/current/scripts/prefabs/gelblob.lua#L495-L505)、[掉落监听](../../../raw/current/scripts/prefabs/gelblob.lua#L582-L597)
- **`deer_unshackle_fx`／`deer_growantler_fx`**：都只在 `SGdeer` 的两个状态 animover 时生成并复制鹿的位置／旋转——`unshackle` 态先把鹿换成普通 `deer` 再生成挣脱 FX，进入路径是宝石鹿 Brain 丢失看守者后的 `PushEvent("unshackle")`；`growantler` 态由冬季鹿群迁移的 `queuegrowantler`→1–2 天 timer→`setantlered`（函数内部再 `PushEvent("growantler")`）→SG 状态生成。两者清理相同：`DoTaskInTime(2, ErodeAway)` 后再走 `ErodeAway` 默认 1 秒腐蚀淡出才 Remove。[挣脱生成](../../../raw/current/scripts/stategraphs/SGdeer.lua#L452-L472)、[生角生成](../../../raw/current/scripts/stategraphs/SGdeer.lua#L198-L233)、[挣脱触发](../../../raw/current/scripts/brains/deergemmedbrain.lua#L131-L135)、[迁移排程](../../../raw/current/scripts/components/deerherdspawner.lua#L189-L194)、[生角链](../../../raw/current/scripts/prefabs/deer.lua#L107-L131)、[ErodeAway](../../../raw/current/scripts/simutil.lua#L575-L597)
- **`fishingnetvisualizerfx`**：唯一生成点是 `SGfishingnetvisualizer` 的 `opening` 态 onenter——在网自身 (x,0,z) 生成；被注释掉的 timeline 版本不生效。宿主 `fishingnetvisualizer` 全程没有 `entity:AddNetwork()`（服务端独占），本 FX 是 visualizer＋FX 视觉化子链中唯一联网实体（渔网物品 `fishingnet` 自身另有 `AddNetwork`），因此开网溅水也是该子链客户端唯一可见部分；`hit` 播完由双端 animover 自删。[生成](../../../raw/current/scripts/stategraphs/SGfishingnetvisualizer.lua#L29-L38)、[宿主不联网](../../../raw/current/scripts/prefabs/fishingnetvisualizer.lua#L12-L35)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 玩法：inventoryitem／fuel（`charlierose`） | `canbepickedup=false` 不可从地面拾取，`canonlygoinpocket` 使容器与 overflow 拒收（[容器](../../../raw/current/scripts/components/container.lua#L270-L277)、[overflow](../../../raw/current/scripts/components/inventory.lua#L802-L805)）；燃料值 `TUNING.TINY_FUEL`＝`seg_time`（30）的 1/4，即 7.5（[值](../../../raw/current/scripts/tuning.lua#L1673)、[seg_time](../../../raw/current/scripts/tuning.lua#L19)）。死亡处理按 prefab 名在背包里找它，找到即吃掉并置 `charlie_vinesave`，随后尸体走藤蔓自救分支、背包不掉落；该标记经 winona 的 OnSave／OnPreLoad 存档保留（[存档](../../../raw/current/scripts/prefabs/winona.lua#L156-L168)）。 | 主模拟；温娜 `winona_charlie_2` 技能条件。 | [[reference/components/inventoryitem|inventoryitem]]、[[reference/components/fuel|fuel]]；[物品旗标](../../../raw/current/scripts/prefabs/charlierose.lua#L79-L87)、[消耗](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L207-L216)、[自救分支](../../../raw/current/scripts/stategraphs/SGwilson.lua#L2152-L2163)、[藤蔓动画](../../../raw/current/scripts/stategraphs/SGwilson.lua#L3736-L3741) |
| 玩法：CLASSIFIED 目标（`charlieresidue`） | 三个 netvar 同步动画态／地图动作上下文／目标实体；客户端把鼠标与手柄交互转发到残影父实体（`client_forward_target`、`highlightchildren`），`valid_map_actions` 只列 ACTIVATE。地图动作消费者按 `GetTarget`／`GetMapActionContext` 读目标；SGwilson 另有 `charlieresidue_open` 等查看状态（不展开）。 | 复制只发给 `SetClassifiedTarget` 指定的玩家；目标变化经 `targetdirty` 刷新，地图动作上下文的 dirty 监听被注释、客户端按需读取。 | [[reference/components/roseinspectableuser|roseinspectableuser]]、[[reference/components/roseinspectable|roseinspectable]]；[netvar](../../../raw/current/scripts/prefabs/charlieresidue.lua#L186-L194)、[转发](../../../raw/current/scripts/prefabs/charlieresidue.lua#L65-L72)、[地图动作](../../../raw/current/scripts/actions.lua#L3002-L3040)、[查看状态](../../../raw/current/scripts/stategraphs/SGwilson.lua#L7566) |
| 玩法：edible（`chumpiece`） | `secondaryfoodtype=FOODTYPE.MEAT` 经 `onfoodtype` 加 `edible_meat`；海鱼 `eater:GetEdibleTags` 产出该标签作为食物目标（[映射](../../../raw/current/scripts/components/eater.lua#L177-L186)），杂食／肉食鱼能吃到、素食／莓果鱼不能；`EAT` 成功即 Remove，宿主据此补位。 | 主模拟；鱼群先在远处被带 `chum` 标签的宿主吸引（[找 chum](../../../raw/current/scripts/brains/oceanfishbrain.lua#L219)），再由 `FindFoodAction` 找可吃目标。 | [[reference/components/edible|edible]]、[[reference/components/eater|eater]]；[标签](../../../raw/current/scripts/components/edible.lua#L13-L16)、[鱼食谱](../../../raw/current/scripts/prefabs/oceanfishdef.lua#L157-L160)、[找食物](../../../raw/current/scripts/brains/oceanfishbrain.lua#L92-L116) |
| 生成者：草（`grass`／`depleted_grass`，Wiki 待建） | 变身计时器产物；同位置复制旋转与颜色。 | 主模拟；世界设置计时器链。 | [变身计时](../../../raw/current/scripts/prefabs/grass.lua#L138-L161) |
| 生成者：`chum_aoe`／`waterplant_pollen_fx_ocean`（Wiki 待建） | 两套鱼饵块宿主；块删除驱动补位，宿主 `onremove` 清表；宿主云本身另有鱼群生成与腐烂玩法，不在本页。 | 主模拟；海洋点检查。 | [鱼饵云](../../../raw/current/scripts/prefabs/chum_aoe.lua#L88-L105)、[水草花粉](../../../raw/current/scripts/prefabs/waterplant_pollen.lua#L100-L118) |
| 生成者：`gelblob`（Wiki 待建） | inventory 的 `dropitem` 后跟随掉落物；落地冲击由 `KillFX` 播。同文件的接触减速 FX 见 gelblob_attach_fx 页。 | 主模拟生成；网络副本随复制与移除收尾。 | [生成](../../../raw/current/scripts/prefabs/gelblob.lua#L495-L505)、[[reference/prefabs/gelblob_attach_fx|同文件接触减速 FX]] |
| 生成者：`SGdeer`／`deergemmedbrain`／`deerherdspawner` | 挣脱与生角两条状态；前者由宝石鹿脑触发，后者由冬季迁移计时链触发。 | 主模拟。 | [状态](../../../raw/current/scripts/stategraphs/SGdeer.lua#L198-L233)、[挣脱](../../../raw/current/scripts/stategraphs/SGdeer.lua#L452-L472) |
| 生成者：`SGfishingnetvisualizer` | 开网瞬间生成；宿主不联网，本 FX 是网链唯一联网实体。 | 主模拟；开网态 onenter 一次。 | [开网态](../../../raw/current/scripts/stategraphs/SGfishingnetvisualizer.lua#L29-L38)、[[reference/components/fishingnetvisualizer|fishingnetvisualizer]] |
| 依赖声明：`charlie_snap`／`charlie_snap_solid` | 只在 `charlieresidue` 的 `prefabs` 表声明，文件内无对应 SpawnPrefab；实际使用在燃料呈现链。 | 声明不等于生成。 | [[reference/prefabs/charlie_snap|暗影拍指]]；[声明](../../../raw/current/scripts/prefabs/charlieresidue.lua#L6-L9) |
| 关联 Prefab：`rose_petals_fx`／`deer_ice_circle` | 同一条死亡自救链的藤蔓花瓣、同宿主文件的宝石鹿法阵；行为各自维护，本页只链。 | 观看对应页 | [[reference/prefabs/rose_petals_fx|玫瑰花瓣飘落]]、[[reference/prefabs/deer_ice_circle|无眼鹿冰火法阵]] |
| 资源 | 动画：`flowers`（`rose`／`rose_shatter`）、`charlieresidue`、`grass.zip`／`grass1.zip`（`grass_part`）、`gelblob`（`splash_loop`／`splash_impact`）、`deer_unshackle`＋`deer_build`、`boat_net_fx`（`hit`）；音效：`pickup_reeds`、`rose_fall_shatter`、`charlie_residue/idle_lp`、`deer/chain`／`bell`／`lock_break`。 | 声明与实际播放已核验，包内容未解包、时长未核验。 | 见各注册行 |

## 典型接入

### 在自定义植物消失处复用草屑

- 前提与选择：`grasspartfx` 是联网实体、动画在双端播放，服务端生成即可；它不读取宿主，只按位置／旋转／颜色播放，适合任何「植物被替换成生物或物件」的瞬间。
- 生效与撤销：主模拟 `animover` 自删，不需要父挂或手动清理；它未显式 `persists=false`（见陷阱），不应在长存档窗口内大量残留。
- 原版依据：[变身计时](../../../raw/current/scripts/prefabs/grass.lua#L138-L161)。

```lua
-- 服务端：植物被替换的同一坐标补草屑，颜色沿用植物
local fx = SpawnPrefab("grasspartfx")
fx.Transform:SetPosition(x, y, z)
fx.Transform:SetRotation(rot)
fx.AnimState:SetMultColour(r, g, b, a)
```

- 调整与验证：拾取音固定在工厂里、pristine 前播放；不复制草的分支与帧。示例只做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **误区：这 8 个都只是视觉**：`charlierose` 是燃料＋死亡自救道具，`charlieresidue` 是 CLASSIFIED 交互／地图目标，`chumpiece` 是海鱼真正吃掉的鱼饵；`fx` 标签与 `_fx` 后缀都不证明纯表现。
- **边界：`grasspartfx` 未设 `persists=false`**：其余 7 个都显式设置（或按背包／掉落状态切换），只有它在主模拟构造段没有关闭持久化；中途存档后的动画与自删行为未核验。
- **边界：鹿 FX 的 2 秒不是总寿命**：`DoTaskInTime(2, ErodeAway)` 之后还要走默认 1 秒 `SetErosionParams` 腐蚀淡出才 Remove，总寿命约 3 秒。[ErodeAway](../../../raw/current/scripts/simutil.lua#L575-L597)
- **边界：`chumpiece` 不会自己消失**：没有 timer／animover，只能被吃、被补位逻辑换掉，或宿主 `onremove` 时清表；宿主消散后不再补位，剩余块留在海里直到被吃或随宿主移除。
- **边界：`charlieresidue` 的客户端 FX 不是网络实体**：`CreateFX` 不 `AddNetwork`，只在需要显示的那一端本地创建并父挂到残影；复制体本身才是 CLASSIFIED 网络实体，失去玫瑰眼镜视觉或技能停用会 Decay。
- **边界：`gelblob_item_fx` 的 `KillFX` 只在主模拟存在**：客户端副本没有该函数，收尾靠网络移除；`KillFX` 睡眠时直接 Remove，跳过落地冲击。
- **边界：`fishingnetvisualizerfx` 是 NOBLOCK 而非 NOCLICK**：它不阻挡移动、也不阻止鼠标射线，但自身没有物理，不能靠物理体选取；按 NOCLICK 标签过滤会漏掉它。
- **边界：`charlierose` 落地不可回收**：`canbepickedup=false` 加上 `canonlygoinpocket`，掉地即碎裂淡出、容器也拒收；只有直接 GiveItem 才能回到背包。

## 验证记录

- 源码审阅：2026-11-04，steam-buildid:26335394。逐个核验 8 个注册的返回位置、工厂、端分界、生成点、父挂与清理；全 `raw/current/scripts`（排除 languages）以 8 个字面名反查：`charlierose` 命中自身、flower、player_common_extensions、prefablist 与 scrapbook（注释），`charlieresidue` 命中自身、actions、mapscreen、SGwilson、winona、roseinspectableuser、prefablist 与 scrapbook（注释），`chumpiece` 仅 chum_aoe／waterplant_pollen，`grasspartfx` 仅 grass.lua，`gelblob_item_fx` 仅 gelblob.lua，两个鹿 FX 仅 deer.lua／SGdeer，`fishingnetvisualizerfx` 命中自身、宿主、prefablist 与 SG。`chinese_s.po` 定向检索：`CHARLIEROSE`／`CHARLIERESIDUE` 有 NAMES 键，其余 6 名无。本页跨 7 个工厂文件，含 CLASSIFIED 客户端 FX、父挂跟随、玩法混合载体与多种清理，需整体对照，按 complex 档。
- 游戏运行：未运行；validation 保持 source-reviewed。
- 结构检查：`validate_page.py` 通过（0 issue；103 行、约 23 KiB，complex 档内）；`validate_source_links.py --strict` 通过（68 处行引用、0 error／0 warning）；Lua 片段 `parser_probe.exe --syntax-only` 通过（只查语法、未执行，临时文件已删）。
- 未覆盖：动画包时长与内容、客户端实际观感、专服与晚加入、读档中途存档表现、宿主玩法（玫瑰眼镜流程、鱼饵云鱼群生成、鹿群迁移、恶液吞人、渔网捕获）细节。

返回 [[reference/prefabs/fx-index|特效 Prefab 索引]]；生物与环境分组见 [[reference/prefabs/fx-world-index|特效 Prefab 索引·生物与环境]]。
