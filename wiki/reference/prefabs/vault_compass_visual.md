---
title: "宝库设施表现族：路标指针、净化雾、地纹贴片与冷火（vault_compass_visual 等 4 注册）"
created: "2026-11-05"
updated: "2026-11-07"
type: prefab
summary: "宝库（Rifts6/7）四个设施表现 FX：路标导航仪装备期生成的贴地指向指针（按虚室方位切三态动画）、圣殿净化器启雾时父挂的 VFX 粒子雾（8→50/秒渐升）、宝库房间撒下并可存档的地纹贴片（18 处生成、非默认值才入档）与火炬复用 coldfire_fire 资产的三档冷火（无 heater、熄灭转烟自删）。"
aliases: ["vault_compass_visual", "vault_decon_mister_fx", "vault_ground_pattern_fx", "vault_torch_flame", "路标导航仪指针", "圣殿净化器雾效", "宝库地纹贴片", "宝库冷火", "谜之火焰", "净化器雾粒子"]
tags: [prefab, fx, server-client]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/vault_compass.lua", "prefabs/vault_decon_mister.lua", "prefabs/vault_ground_pattern.lua", "prefabs/vault_torch.lua", "prefabs/vaultroom_defs.lua", "prefabs/vault_key_trial.lua", "prefabs/lightsout_trial.lua", "prefabs/vault_decon_switch.lua", "components/vaulttorchgrid.lua", "components/firefx.lua", "components/wisecracker.lua", "stategraphs/SGshadowcreature.lua", "stategraphs/SGoceanshadowcreature.lua", "map/static_layouts/vault_lobby.lua", "mainfunctions.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 宝库设施表现族：路标指针、净化雾、地纹贴片与冷火（vault_compass_visual 等 4 注册）

## 快速判断

- 职责与适用：4 注册都是宝库设施链上的**联网表现层**——①`vault_compass_visual` 是装备路标导航仪时生成的**贴地指向指针**（OnGround＋背景层＋泛光，按虚室方位切 idle_marker／success／fail 三态并 FacePoint）；②`vault_decon_mister_fx` 是圣殿净化器启雾时父挂的 **VFX 粒子雾**；③`vault_ground_pattern_fx` 是宝库房间贴地的**地纹贴片**（世界生成撒下，带 SetVariation／HideCenter／SetOrientation／ChangeSortOrder／EnableOn API 且**可存档**）；④`vault_torch_flame` 是火炬的**三档冷火**（firefx 等级表、光挂火炬本体）。
- 优先使用：查"罗盘为什么转/什么时候变绿"用 compass 链；查"净化房绿雾是谁、驱散影子怎么触发"用 mister 链；查"宝库地板花纹怎么摆放、读档为什么不乱"用 pattern；查"火炬冷火几档、怎么熄"用 torch 链。
- 重要副作用：torch_flame 与 compass_visual 都无自删（前者由火炬换烟自删，后者由 ClearMarker 显式移除）；mister_fx 无自删（StopMisting 移除）；ground_pattern_fx **不设 persists=false**——它是随世界存档持久化的贴片（与其他 FX 相反）。粒子和指针在专用服务器上是空壳或由宿主驱动。
- 修改边界：只覆盖 4 注册的工厂、生成点、端、挂接与清理；虚室（virtualroom）系统、lightsout 点灯谜题与 key 试用玩法、净化门时序玩法不展开；宿主 `vault_compass`／`vault_decon_mister`／`vault_torch` 本体只在链条里辨析。
- 名称：宿主官方名 `VAULT_COMPASS`＝"路标导航仪"（[po](../../../raw/current/scripts/languages/chinese_s.po#L351189-L351192)）、`VAULT_DECON_MISTER`＝"圣殿净化器"（[po](../../../raw/current/scripts/languages/chinese_s.po#L351214-L351217)）、`VAULT_TORCH`＝"谜之火焰"（[po](../../../raw/current/scripts/languages/chinese_s.po#L351319-L351322)）；4 个 FX 注册名在简中 po 全文 0 命中，标题中文为已核验效果描述，非官方译名。

## 注册与构造

| 注册名 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `vault_compass_visual` | `visualpointerfn`：Transform／AnimState／Follower／Network；bank=build `vault_compass`、循环播 `idle_marker`；OnGround＋LAYER_BACKGROUND＋SortOrder 3＋泛光 anim.ksh；标签 staysthroughvirtualrooms／FX／NOCLICK；主模拟 `persists=false` | **贴地指针**：自身无朝向逻辑——动画切换与 FacePoint 全由宿主罗盘的周期任务驱动。[工厂](../../../raw/current/scripts/prefabs/vault_compass.lua#L214-L243)、[注册](../../../raw/current/scripts/prefabs/vault_compass.lua#L249) |
| `vault_decon_mister_fx` | `decon_mist_fx_fn`：Transform／Network、标签 `FX`；SetPristine 后 `persists=false`；专服早退空壳；非专服 VFXEffect 单发射器（fx/animsmoke2.tex、vfx_particle_reveal_withlight 着色器、128 上限、寿命 1.2 秒、4×4 UV 帧、旋转粒子、AlphaBlended＋泛光、drag 0.15） | **粒子雾**：发射率随存活时间 8→50/秒渐升，方向角按 PI2/16 螺旋推进加抖动；颜色包络取"月岩着色"的橄榄绿（[L118 注释](../../../raw/current/scripts/prefabs/vault_decon_mister.lua#L111-L123)）。[工厂](../../../raw/current/scripts/prefabs/vault_decon_mister.lua#L162-L225)、[注册](../../../raw/current/scripts/prefabs/vault_decon_mister.lua#L229) |
| `vault_ground_pattern_fx` | `fn`：Transform／AnimState／Network；标签 FX／NOCLICK；bank=build `vault_ground_pattern`、播 idle1；OnGround＋LAYER_BACKGROUND＋SortOrder -3＋FinalOffset -1；主模拟默认 variation=1、orientation=1 并暴露 SetVariation／HideCenter／SetOrientation／ChangeSortOrder／EnableOn | **可存档地纹**：OnSave 只记非默认值（variation≠1、nocenter、orientation≠1、sortorder≠-3），OnLoad 逐项回放；**未置 persists=false**，随世界存档持久化。[工厂](../../../raw/current/scripts/prefabs/vault_ground_pattern.lua#L73-L108)、[存读 API](../../../raw/current/scripts/prefabs/vault_ground_pattern.lua#L49-L71)、[注册](../../../raw/current/scripts/prefabs/vault_ground_pattern.lua#L110) |
| `vault_torch_flame` | `flamefn`：Transform／AnimState／SoundEmitter／Network；bank=build `coldfire_fire`；泛光 anim.ksh、SetRayTestOnBB、FinalOffset 3；标签 `FX`；主模拟挂 firefx 组件（levels=三档 FLAME_LEVELS、usedayparamforsound=true） | **三档冷火**：level1/2/3_redux，半径 2/3/4、强度 0.25、冷白色 (180,240,255)/255、coldfire_LP 音量参 0.06/0.08/0.1；光实体由 firefx 自建并 AttachLightTo 到火炬。[工厂](../../../raw/current/scripts/prefabs/vault_torch.lua#L347-L374)、[等级表](../../../raw/current/scripts/prefabs/vault_torch.lua#L338-L344)、[注册](../../../raw/current/scripts/prefabs/vault_torch.lua#L377) |

## 关联机制导航

### 路标指针：罗盘装备期的三层结构与方位判定

| 环节 | 行为与条件 | 证据 |
|---|---|---|
| 生成与挂接 | `vault_compass` OnEquip：先 `SpawnPrefab("vault_compass_marker")` 父挂持有者（播 `empty` 占位动画的联网实体），再生成 `vault_compass_visual` 并 `FollowSymbol(marker.GUID, "empty", 0,0,0)`——指针挂在 marker 的空符号上随人走 | [OnEquip](../../../raw/current/scripts/prefabs/vault_compass.lua#L82-L100) |
| 方位驱动 | 罗盘起 `DoPeriodicTask(0, OnUpdateDirection)` 每 tick 判定：VAULT 虚室内当前房==key1 → `idle_marker_success`；否则找通向 key1 的方向及朝该方向的传送器 → `idle_marker`＋FacePoint；LOBBYVAULT 内找 `vaultmarker_lobby_to_vault`；室外找 archive_portal 标记（2.5 格内直接 success）。VAULT 分支 direction 为 nil 时没有 else，保留旧动画／方向；LOBBY 与室外找不到标记才切 `idle_marker_fail`；VAULT 内 direction 有值但找不到匹配传送器也切 fail | [OnUpdateDirection](../../../raw/current/scripts/prefabs/vault_compass.lua#L21-L80) |
| 端与清理 | 罗盘是普通可装备物品（另可被 lore1 房间摆在地上，[vaultroom_defs.lua:467](../../../raw/current/scripts/prefabs/vaultroom_defs.lua#L467)）；OnUnequip／OnEquipToModel 走 `ClearMarker`：取消任务并同删 marker 与指针 | [ClearMarker](../../../raw/current/scripts/prefabs/vault_compass.lua#L102-L111) |

辨析：marker 与 pointer 都有 `staysthroughvirtualrooms` 标签（虚室重载时不被清除）；marker **未置 persists=false**，但装备期间已 `SetParent` 到持有者，[世界存档入口](../../../raw/current/scripts/mainfunctions.lua#L1113-L1115) 还要求 `entity:GetParent()==nil`，因此不能据此推出装备期会把 marker 写入世界存档、读档留下无指针 marker；该跨会话表现未实测。

### 净化雾：净化器启停与雾效生灭

| 环节 | 行为与条件 | 证据 |
|---|---|---|
| 宿主布点 | decon1 房间布局在中心四象限 ±3.5 各放一台净化器，交给 `vault_decon_switch` 的 entitytracker 以 mist1–4 跟踪 | [布局](../../../raw/current/scripts/prefabs/vaultroom_defs.lua#L1137-L1148) |
| 启雾 | 拉杆后 1.5 秒（DOOR_WAIT_TIME_START）对四台调 `StartMisting`：0.5 秒周期 `OnApplyMist`（首延随机）、播 misting_activate→misting_loop 循环与 active_LP、首次生成 `vault_decon_mister_fx` 父挂本体（mistfx 单槽，已存在不重复生成）；雾持续 MIST_TIME_SECONDS=4 秒后 `StopMisting` | [StartMisting](../../../raw/current/scripts/prefabs/vault_decon_mister.lua#L31-L45)、[开关时序](../../../raw/current/scripts/prefabs/vault_decon_switch.lua#L11-L13) |
| 雾效判定 | `OnApplyMist` 半径 5.1（+0.1 为 decon1 布局重叠优化）找 `_combat`＋locomotor 实体：玩家推 `got_decon_misted`（[wisecracker.lua:462](../../../raw/current/scripts/components/wisecracker.lua#L462) 消费吐槽），影子生物推 `dispelsanityshadowcreature`（SGshadowcreature.lua:28／SGoceanshadowcreature.lua:36 消散） | [OnApplyMist](../../../raw/current/scripts/prefabs/vault_decon_mister.lua#L13-L25) |
| 停雾 | `StopMisting`：取消任务、KillSound＋播 close、动画 misting_deactivated→misting_closed、`mistfx:IsValid()` 才 Remove 并清槽 | [StopMisting](../../../raw/current/scripts/prefabs/vault_decon_mister.lua#L47-L67) |

### 地纹贴片：18 处生成与存档 API 语义

生成点全为**世界生成/房间装配期**的贴地摆放，共 18 处调用：`vaultroom_defs.lua` 15 处——puzzle1 三环 3×3 阵（变体池 3–5，[L329](../../../raw/current/scripts/prefabs/vaultroom_defs.lua#L329)）、hall1–3 安保厅中心 1 处 HideCenter（[L391](../../../raw/current/scripts/prefabs/vaultroom_defs.lua#L391)）、hall5–7 条件 1 处（按房号/种子 50%，[L401/L404](../../../raw/current/scripts/prefabs/vaultroom_defs.lua#L401-L404)）、lore1 符文与雕像各 1（[L463/L472](../../../raw/current/scripts/prefabs/vaultroom_defs.lua#L463-L472)）、lore2（[L582](../../../raw/current/scripts/prefabs/vaultroom_defs.lua#L582)）、lore3（[L661](../../../raw/current/scripts/prefabs/vaultroom_defs.lua#L661)）、teleport1（[L725](../../../raw/current/scripts/prefabs/vaultroom_defs.lua#L725)）、mask1 三处（[L816-L818](../../../raw/current/scripts/prefabs/vaultroom_defs.lua#L816-L818)）、generator1（[L906](../../../raw/current/scripts/prefabs/vaultroom_defs.lua#L906)）、fountain1/2（[L977](../../../raw/current/scripts/prefabs/vaultroom_defs.lua#L977)）、playbill1 家具旁 2 处（[L1043](../../../raw/current/scripts/prefabs/vaultroom_defs.lua#L1043)）；`vault_key_trial.lua` 2 处（柱环×4＋中央 1，均 ChangeSortOrder(-2) 且由 entitytracker 跟踪、OnLoadPostPass 重连，[L315/L331](../../../raw/current/scripts/prefabs/vault_key_trial.lua#L315-L331)）；静态布局 `map/static_layouts/vault_lobby.lua` 1 处（type 字段直填注册名，[L254](../../../raw/current/scripts/map/static_layouts/vault_lobby.lua#L254)）。

API 语义（台账批 8 结论复核属实）：`SetVariation` 播 `idle..n` 且同值跳过；`SetOrientation` 在 orientation>2 且 variation<3 且未 HideCenter 时先减 2，得到 1/2 后走正常缩放与 0/180°；其余保留的 3/4 才镜像（scale -1、旋转 -90°；原版 else 中 `orientation==1` 不可达）。`HideCenter` 幂等置 nocenter；`EnableOn` 播 `idle..n.."_on"`（谜题点亮态）。存档**只记非默认值**——variation=1、orientation=1、sortorder=-3、未 HideCenter 的贴片在存档里零字节，读档回默认外观；这与"多数 FX 不存档"相反，是本注册被单独维护存档 API 的原因（[OnSave/OnLoad](../../../raw/current/scripts/prefabs/vault_ground_pattern.lua#L49-L71)）。

### 火炬冷火：machine 三态与 firefx 复用

| 环节 | 行为与条件 | 证据 |
|---|---|---|
| 生成 | 火炬 `DoTurnOn`（TurnOn/BrokenTurnOn 动画后 0.2 秒，或 asleep/POPULATING 直进）：SpawnPrefab → `AddChild(火炬)` → Follower 跟随火炬 `fire_marker` 符号（偏移 0,40,0）→ 置 `persists=false` → `SetLevel`（损坏 1／卡死 3／正常 2）→ `AttachLightTo(火炬)` | [DoTurnOn](../../../raw/current/scripts/prefabs/vault_torch.lua#L22-L32) |
| 等级 | firefx 协议：SetLevel 播 `levelN_redux` 循环＋设光参数＋coldfire_LP（usedayparamforsound 把昼夜写进音效参数）；光实体是 firefx 自建的 `firefx_light`（另页），AttachLightTo 把它改挂火炬本体 | [firefx:SetLevel](../../../raw/current/scripts/components/firefx.lua#L88-L135)、[AttachLightTo](../../../raw/current/scripts/components/firefx.lua#L33-L49) |
| 熄灭 | `DoTurnOff`：`Extinguish(true)` 因三档都无 pst 动画返回 false → 火炬直接把 flame 换成烟：build 换 `smoke_plants`、bank smoke_out 播 smoke_single、清泛光、随机镜像缩放（损坏 0.6 倍）与 0.8–1.3 倍速、加 NOCLICK、关光，动画播完 1 帧后 Remove——**同一实体冷火转烟自删** | [DoTurnOff](../../../raw/current/scripts/prefabs/vault_torch.lua#L70-L92)、[Extinguish 返回值](../../../raw/current/scripts/components/firefx.lua#L139-L155) |
| 三态宿主 | 火炬 machine 三态：normal（turnon/turnoff＋冷却 0.5）、stuck（MakeStuckOn/Off 摘除 turnonfn/turnofffn，对 machineturnedoff/on 事件播 stuck 动画与音效）、broken（machine.enabled=false，BrokenTurnOn/Off）；OnSave/OnLoad 记 stuck/broken 两旗还原；lightsout 试用以 vaulttorchgrid 生成 3×3 火炬阵（间距 6、每台 `persists=false`），灯阵开关状态序列化在 trial 实体的组件存档里（cols/rows/bits），读档重建网格并逐台 PopTorch | [三态函数](../../../raw/current/scripts/prefabs/vault_torch.lua#L178-L247)、[灯阵生成](../../../raw/current/scripts/components/vaulttorchgrid.lua#L49-L67)、[网格存读](../../../raw/current/scripts/components/vaulttorchgrid.lua#L196-L231) |

与 [[reference/prefabs/fire|火焰、火光与烟雾特效族]] 的 coldfire 体系辨析：`vault_torch_flame` 复用 `coldfire_fire` bank/build 与 firefx 等级表协议，但与 `coldfirefire`（fire.md 登记）差异是——**只有 3 档**（coldfirefire 4 档）、**无 heater 组件**（不吸热不供热，纯光＋动画）、**无 pre/pst 动画**（Extinguish 恒 false，熄灭表现由宿主换烟实现）、光色为冷白 (180,240,255)/255 而非 00B7FF、光挂在宿主火炬上。fire.md 的 firefx 协议描述适用于本注册，等级参数各自维护。

### 清理路径

| 注册 | 清理方式 | 边界与证据 |
|---|---|---|
| `vault_compass_visual` | 无自删：罗盘 OnUnequip/OnEquipToModel 的 ClearMarker 同删 marker＋指针＋任务 | 罗盘本体被移除时是否经 unequip 事件路径或随引擎回收未核验。 |
| `vault_decon_mister_fx` | 无自删：StopMisting 校验 IsValid 后 Remove；净化器本体被砸/移除时是否随父挂回收未核验 | mistfx 单槽引用在净化器上，重复 StartMisting 不会叠雾（[L41-L44](../../../raw/current/scripts/prefabs/vault_decon_mister.lua#L41-L44)）。 |
| `vault_ground_pattern_fx` | **不主动清理**：随房间布局持久存在，随世界存档往返；key 试用房的 5 处由 entitytracker 跟踪重连 | 无移除调用（全域 grep 无 Remove 路径）；`EnableOn` 只换动画不改持久性。 |
| `vault_torch_flame` | 熄灭时转烟自删（宿主排定 Remove）；火炬本体被移除时随 AddChild 级联 | `persists=false` 不存档；读档后由 torch OnLoad 的 MakeStuck/Broken/Normal 状态路径重建（asleep 分支直进 DoTurnOn/Off）。 |

## 典型接入

### 做"随世界存档往返"的地面贴片

- 前提与选择：多数 FX 用 `persists=false`＋由宿主重建；若贴片是房间布局的一部分且读档后要原样回放（花纹变体、朝向、层级），参照 ground_pattern_fx——暴露幂等 setter、OnSave 只记非默认值、OnLoad 逐项回放。
- 生效与撤销：生成方在装配期 `SpawnPrefab` 后链式调 API（`:SetVariation(n):SetOrientation(k)` 等，工厂方法都返回 inst 支持链式）；无撤销路径——贴片生命周期即房间生命周期。

```lua
local fx = SpawnPrefab("vault_ground_pattern_fx")
fx:SetVariation(math.random(2)):SetOrientation(math.random(4))
fx:HideCenter():ChangeSortOrder(-2) -- 方法返回 inst，可链式
```

- 调整与验证：替换点是注册名、动画名（idle..n / idle..n_on）与默认值表；OnSave 的"非默认才记"约定要与新默认值同步，否则读档回错外观。示例只做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **边界：`vault_ground_pattern_fx` 不是"不存档的 FX"。** 它未置 persists=false 且带完整存读 API；给这种贴片套"FX 都不存档"的假设会做出读档消失/乱摆的错误改造（[OnSave](../../../raw/current/scripts/prefabs/vault_ground_pattern.lua#L49-L54)）。
- **边界：地纹朝向的镜像规则依赖变体与中心。** `SetOrientation` 收到 3/4 时，只有 variation<3 且未 HideCenter 才先减 2 转成 1/2 的正常缩放；variation≥3 或已 HideCenter 的贴片传 3/4 会直接以镜像缩放渲染（-90°），摆放代码里变体、中心与朝向是耦合的（[L22-L37](../../../raw/current/scripts/prefabs/vault_ground_pattern.lua#L22-L37)）。
- **误区：净化雾是"毒雾/伤害雾"。** `vault_decon_mister_fx` 纯视觉无碰撞无伤害；真正的作用是本体周期任务推事件（玩家吐槽、影子生物消散），5.1 半径判定在宿主不在雾粒子（[OnApplyMist](../../../raw/current/scripts/prefabs/vault_decon_mister.lua#L13-L25)）。
- **边界：罗盘指针的动画由宿主每 tick 重判。** 指针工厂无任何逻辑，改指向行为要改罗盘的 `OnUpdateDirection`；`ChangeToAnim` 同动画不重播，FacePoint 每 tick 重设（[L15-L19](../../../raw/current/scripts/prefabs/vault_compass.lua#L15-L19)）。
- **边界：火炬冷火的档位不对应火焰大小直觉。** stuck 态反而 3 档最大、broken 态 1 档最小、normal 2 档——档位由火炬状态决定而非燃料/时间（[L29](../../../raw/current/scripts/prefabs/vault_torch.lua#L22-L32)）；三档半径 2/3/4 只影响光，无热效应。
- **边界：lightsout 火炬阵的火炬不存档、灯阵存档。** 每台火炬 persists=false，读档由 trial 的 vaulttorchgrid 组件按 bits 重建并逐台还原开关；直接改火炬存档字段无效（[网格存读](../../../raw/current/scripts/components/vaulttorchgrid.lua#L196-L231)）。
- **边界：mistfx 在专服是空壳。** 工厂 IsDedicated 早退（[L175-L179](../../../raw/current/scripts/prefabs/vault_decon_mister.lua#L174-L179)），专服仍有实体与父挂（雾效判定/事件照常），只有粒子表现缺失。

## 验证记录

- 源码审阅：2026-11-05，steam-buildid:26335394（实读 `raw/current/BUILD.txt`）。完整读取 `prefabs/vault_compass.lua`、`prefabs/vault_decon_mister.lua`、`prefabs/vault_ground_pattern.lua`、`prefabs/vault_torch.lua`、`components/vaulttorchgrid.lua`、`components/firefx.lua`，按需读取 `prefabs/vaultroom_defs.lua`（18 处生成点所在布局段）、`prefabs/vault_key_trial.lua`（生成与跟踪段）、`prefabs/lightsout_trial.lua`（网格初始化与存读段）、`prefabs/vault_decon_switch.lua`（时序常量与启停）、`map/static_layouts/vault_lobby.lua`（type 字段段）。对 4 个注册名以完整名在 `raw/current/scripts` 全域反查：`vault_compass_visual` 命中注册（vault_compass.lua:249）、依赖（:9）、生成（:92）——**生成点唯一**；`vault_decon_mister_fx` 命中注册（vault_decon_mister.lua:229）、依赖（:6）、生成（:42）——**生成点唯一**；`vault_ground_pattern_fx` 命中注册（vault_ground_pattern.lua:110）、依赖（vault_markers.lua:30、vault_key_trial.lua:14）、生成 18 处（vaultroom_defs.lua 15＋vault_key_trial.lua 2＋vault_lobby.lua 静态布局 1）；`vault_torch_flame` 命中注册（vault_torch.lua:377）、依赖（:14）、生成（:25，DoTurnOn 内）——**生成点唯一**。`dispelsanityshadowcreature` 消费端（SGshadowcreature.lua:28／SGoceanshadowcreature.lua:36）与 `got_decon_misted` 消费端（wisecracker.lua:462）以定向 grep 核验。未运行游戏。
- 名称核验：`languages/chinese_s.po` 定向检索——三宿主 NAMES 键译文见"快速判断"；4 个 FX 注册名全文 0 命中（含 DESCRIBE），标题使用效果描述。
- 分档说明：`size_class: standard`——4 注册四条链，每链单层生成/清理，无多变体状态机；四链结构同型故并入一页。
- 未覆盖：虚室（virtualroommanager）系统全貌、lightsout 点灯谜题解法与 abysspillargroup、key 试用玩法全链、净化门时序玩法（仅记 1.5/4/0.75 秒常量）、`vault_compass_marker`（非 FX 注册，仅记链条角色）、firefx_light 工厂细节（见 fire.md）、动画包帧数与观感；装备中存档 marker 的跨会话实测；未运行游戏。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 结果见交付记录（0 error 0 warning 后入库）。
- 游戏运行：未运行；`validation` 保持 `source-reviewed`。

- 第 15 轮补正（2026-11-07）：撤回 marker 必入世界存档、补朝向折算与 VAULT 无方向保留旧动画；同步维护日期。

## 相关链接

- [[reference/prefabs/fire|火焰、火光与烟雾特效族]]（coldfire_fire 资产与 firefx 等级表协议的体系主页）
- [[reference/prefabs/archive_chandelier|远古吊灯与石像鬼石化特效]]（同批宝库照明设施，`vault_crawler_chandelier` 兼生成点）
- [[reference/prefabs/vault_pillar_guard_swipe_fx|宝库守卫柱挥击特效]]（宝库战斗侧表现）
- 返回 [[reference/prefabs/fx-structure-index|建筑、装置与遗迹演出]]（宝库设施）
