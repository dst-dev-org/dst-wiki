---
title: "帽子跟随层与帽灯特效族（lunarplanthat_fx 等 12 注册）"
created: "2026-11-05"
updated: "2026-11-05"
type: prefab
summary: "hats.lua 内 11 个纯 FX 注册与 1 个技术实体的工厂、生成点、端模型与清理：九顶帽的 MakeFollowFx 多层跟随层（共享工厂＋逐帽 netvar 扩展）、矿工帽／启迪之冠两盏联网帽灯，以及高礼帽暗影储物间的 tophat_container 代理；层体各端自建、帽体装卸驱动生成与移除。"
aliases: ["hats", "lunarplanthat_fx", "voidclothhat_fx", "wagpunkhat_fx", "inspectacleshat_fx", "rabbithat_fx", "pumpkinhat_fx", "bat_bosscorpsehat_fx", "shadow_thrall_parasitehat_fx", "mask_halfwit_fx", "minerhatlight", "alterguardianhatlight", "tophat_container", "帽子跟随层", "帽灯", "矿工帽灯", "亮茄头盔跟随层", "虚空风帽跟随层", "兔帽跟随层", "南瓜帽脸"]
tags: [prefab, fx, server-client]
size_class: complex
dst_build: "steam-buildid:26335394"
sources: ["prefabs/hats.lua", "play_commonfn.lua", "play_the_veil.lua", "prefabs/wagpunkhat_classified.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 帽子跟随层与帽灯特效族（lunarplanthat_fx 等 12 注册）

## 快速判断

- 职责与适用：`prefabs/hats.lua` 共注册 97 个 Prefab——85 顶玩法帽（`MakeHat`）、**11 个纯 FX 注册**（9 个 `MakeFollowFx` 多层跟随层，默认 3 层、南瓜帽 2 层＋2 盏 `Prefab` 帽灯）与 1 个技术实体（`tophat_container`）。本页只覆盖后 12 项；帽本体的组件玩法（保暖、装甲、容器等）不属于本页。注册行：9 个跟随层 [hats.lua#L7955-L8033](../../../raw/current/scripts/prefabs/hats.lua#L7955-L8033)、2 盏灯 [L8035-L8036](../../../raw/current/scripts/prefabs/hats.lua#L8035-L8036)、技术实体 [L8038](../../../raw/current/scripts/prefabs/hats.lua#L8038-L8038)
- 优先使用：查"帽子戴上看不到动画层／跟随层"或九个 `*hat_fx` 注册用[跟随层表](#注册与构造)；查"矿工帽／启迪之冠的灯"用帽灯条目；查"高礼帽暗影储物间代理"用 tophat_container；`tophat_shadow_fx`（魔术师高礼帽影子）不在本文件注册，见 [[reference/prefabs/tophat_shadow_fx|魔术师高礼帽影子特效]]。
- 重要副作用：跟随层壳是**无 AnimState 的联网壳**，可见动画层是每台非专服机器自建的非联网实体（`SpawnFollowFxForOwner`，3 层、南瓜帽 2 层）；壳 `persists=false`，生命周期完全由帽本体的 equip／unequip 回调驱动。帽灯是 Transform＋Light 的联网壳，同样 `persists=false`、随宿主开关移除。
- 修改边界：只覆盖 12 注册的工厂、生成点、端模型与清理；`MakeHat` 骨架与帽本体玩法只在[骨架辨析](#makehat-骨架与-fx-的关系)记到能解释 FX 触发的深度。`wagpunksteam_hat_up/down`（通用表蒸汽）由 [[reference/prefabs/wagpunksteam_hat_up|WAGpunk 装备蒸汽]] 页覆盖、`alterguardian_hat_equipped`（前后层）由 [[reference/prefabs/armor_lunarplant_glow_fx|装备与帽饰跟随光效]] 页覆盖，本页不重复展开。
- 名称：12 个注册名均无 `STRINGS.NAMES.*` 键（定向检索见[验证记录](#验证记录)），标题中文为已核验效果描述，**非官方译名**；宿主帽官方名仅正文指认用。

## MakeHat 骨架与 FX 的关系

`MakeHat(name)` 是 85 顶帽的参数化工厂：`fname = "hat_"..name`（动画 build）、`symname = name.."hat"`（bank 与**注册名**）；`simple()` 骨架建 Transform／AnimState／Network＋`MakeInventoryPhysics`＋`hat` 标签，主模拟装配 inventoryitem／inspectable／tradable／equippable（HEAD 槽）；无专属 fn 的帽落到 `default()`（= 裸 `simple()`）（[骨架](../../../raw/current/scripts/prefabs/hats.lua#L188-L235)、[default](../../../raw/current/scripts/prefabs/hats.lua#L273-L275)、[注册名推导](../../../raw/current/scripts/prefabs/hats.lua#L16-L20)）。

骨架与 FX 的分界：**MakeHat 本身不生成任何 FX**。装备表现走符号 override（`_onequip`／`opentop_onequip`／`fullhelm_onequip` 把 build 的 `swap_hat` 符号盖到持有者身上，[基础三路](../../../raw/current/scripts/prefabs/hats.lua#L31-L171)），需要动画层的帽在自己的 onequip 回调里显式 `SpawnPrefab` FX 壳并调 `AttachToOwner`；需要照明的帽在开关回调里生成帽灯。`prefabs` 依赖数组只是声明（如 `lunarplant` 声明 `lunarplanthat_fx`），不证明生成。

## 注册与构造

### 共享工厂 MakeFollowFx（9 个跟随层的公共骨架）

壳构造：Transform＋Network（**无 AnimState**）→ `FX` 标签 → `colouraddersync` 组件 → `data.common_postinit`（逐帽 netvar 在此挂）→ `SetPristine` → 客户端把 `OnEntityReplicated` 设为重放跟随 → 主模拟 `AttachToOwner` 方法＋`persists=false`＋`master_postinit`（[工厂](../../../raw/current/scripts/prefabs/hats.lua#L7775-L7833)）。装配协议 `AttachToOwner`（主模拟装备回调调用）：`SetParent(owner.entity)` → 持有者有 colouradder 则 `AttachChild` → 有 `owningitem` 且带皮肤时写 `skinbuildhash` → **非专服**调 `SpawnFollowFxForOwner`（[装配](../../../raw/current/scripts/prefabs/hats.lua#L7783-L7798)；客户端复制走 [OnEntityReplicated](../../../raw/current/scripts/prefabs/hats.lua#L7776-L7781) 同路径）。

`SpawnFollowFxForOwner`（[实现](../../../raw/current/scripts/prefabs/hats.lua#L7755-L7773)）：按 `framebegin..frameend` 循环建 `createfn(i)` 的本地动画层，父挂持有者、`FollowSymbol(owner.GUID, 跟随符号, nil,nil,nil,true,nil,i-1)`——尾参按层序传帧序参数（同族用法见传送伞持换装层）；全套共享一个随机起始帧 `SetFrame(frame)` 保证相位一致；逐层 `highlightchild:SetOwner(owner)`；层引用存 `inst.fx` 列表，`OnRemoveEntity` 逐层 `Remove`，`colouraddersync` 的染色变化广播到全部层。跟随符号默认 `swap_hat`；`isfullhelm = true` 的帽（亮茄、虚布、南瓜）在持有者 build 含 `headbase_hat` 符号时改跟 `headbase_hat`。

### 九个跟随层差异

| 注册名 | 本地动画层（createfn） | netvar 扩展（common_postinit） | 差异与证据 |
|---|---|---|---|
| `lunarplanthat_fx` | bank `lunarplanthat`／build `hat_lunarplant`、循环 `idle1..3`、`glow01`／`float_top` 符号泛光＋半透明 | `skinbuildhash`（net_hash→非专服换 `SetSkin`） | 亮茄头盔（官方名"亮茄头盔"）全头盔跟随层；生成前先 Remove 旧层。[层](../../../raw/current/scripts/prefabs/hats.lua#L7163-L7188)、[皮肤同步](../../../raw/current/scripts/prefabs/hats.lua#L7205-L7210)、[注册](../../../raw/current/scripts/prefabs/hats.lua#L7970-L7977) |
| `voidclothhat_fx` | bank `voidclothhat`／build `hat_voidcloth`、循环 `idle1..3` | `buffed`（net_bool→`idle*_powerup[_pre/pst]` 换装）＋`skinbuildhash` | 虚空风帽暗影 buff 层数驱动 powerup 动画；`IsCurrentAnimation` 判重防重播。[buff 换装](../../../raw/current/scripts/prefabs/hats.lua#L7234-L7253)、[注册](../../../raw/current/scripts/prefabs/hats.lua#L7978-L7985) |
| `wagpunkhat_fx` | bank `wagpunkhat`（5 个可选 build 资产）／build `hat_wagpunk`、循环 `idle1..3` | `level`（net_tinybyte→按 2-5 级切 bank `hat_wagpunk_02..05` 并向持有者推 `wagpunk_changelevel`） | W.A.R.B.I.S. 头戴齿轮的等级外壳；等级由 targettracker 时间阶段写（蒸汽表现另页）。[等级切换](../../../raw/current/scripts/prefabs/hats.lua#L7064-L7085)、[注册](../../../raw/current/scripts/prefabs/hats.lua#L7986-L7996) |
| `inspectacleshat_fx` | bank `inspectacleshat`／build `hat_inspectacles`、初始 `off1..3`、`led_on` 泛光 | `ledstate`（0 关/1 冷却闪烁/2 开）→ activate/deactivate 动画＋LED 符号 override＋**本地闪烁任务**；壳带 SoundEmitter（非专服本地播激活音） | 检查专用镜的小游戏信号层：`SetLedEnabled` 换 `led_off→led_on` 符号并调泛光；blink 周期 0.1/0.75/1.5 秒三档。[LED 协议](../../../raw/current/scripts/prefabs/hats.lua#L7303-L7396)、[注册](../../../raw/current/scripts/prefabs/hats.lua#L7997-L8003) |
| `rabbithat_fx` | bank `rabbithat`／build `rabbit_build`＋`AddOverrideBuild("hat_rabbit")`、循环 `idle1..3`；`insane` 状态客户端换 `beard_monster` build | `iswinter`（net_bool→换 `rabbit_winter_build`） | 洞穴花环（官方名）的活兔层：胡子怪物状态与冬季皮毛都换 build。[层](../../../raw/current/scripts/prefabs/hats.lua#L7398-L7421)、[冬季](../../../raw/current/scripts/prefabs/hats.lua#L7423-L7436)、[注册](../../../raw/current/scripts/prefabs/hats.lua#L8004-L8016) |
| `pumpkinhat_fx` | bank `pumpkinhat`／build `hat_pumpkin`、`idle1..2`（**两层**）、三张脸符号描边常亮 LightOverride 0.3；EightFaced | `face`（net_ushortint：5bit 右眼/5bit 左眼/5bit 嘴）→逐层 override 脸符号＋夜晚开脸灯；非专服挂 updatelooper 逐帧呼吸灯 | 无面南瓜（官方名）的雕刻脸：`CopyFaceSymbols` 从帽本体复制脸数据；光照是符号加色／乘色＋Bloom 的正弦呼吸（[实现](../../../raw/current/scripts/prefabs/hats.lua#L7470-L7549)）。[层](../../../raw/current/scripts/prefabs/hats.lua#L7441-L7468)、[脸解码](../../../raw/current/scripts/prefabs/hats.lua#L7619-L7635)、[注册](../../../raw/current/scripts/prefabs/hats.lua#L8017-L8025) |
| `bat_bosscorpsehat_fx` | bank `bat_bosscorpsehat`／build `hat_bat_bosscorpse`、`idle1..3` | `skinbuildhash`＋0 帧任务向持有者推 `startbatcorpsehatdrain`（onremove 时推 stop）；持有者带 `locomotor` 标签时给壳加 `autojiggle`（走路 `swing_loop1..3`、停步 `settle1..3`） | 蝙蝠帽（官方名）尸体随步晃动层；drain 事件归 HUD 与头盔衰变玩法。[init](../../../raw/current/scripts/prefabs/hats.lua#L7709-L7739)、[jiggle](../../../raw/current/scripts/prefabs/hats.lua#L7686-L7707)、[注册](../../../raw/current/scripts/prefabs/hats.lua#L8026-L8033) |
| `shadow_thrall_parasitehat_fx` | bank／build `hat_shadow_thrall_parasite`、循环 `idle1..3` | 壳带 SoundEmitter＋`talker` 组件（`MakeChatter`、紫色聊天字） | 虚空面具（宿主怪官方名"虚空面具"，帽本体无 NAMES 键）寄生状态的脸说话层：帽本体 `SaySpeechLine` 指到 FX 的 talker，宿主死亡／被寄生者退场时随帽移除。[talker](../../../raw/current/scripts/prefabs/hats.lua#L7117-L7140)、[层](../../../raw/current/scripts/prefabs/hats.lua#L7142-L7161)、[注册](../../../raw/current/scripts/prefabs/hats.lua#L7962-L7968) |
| `mask_halfwit_fx` | bank `mask_halfwithat`／build `hat_mask_halfwit`、播 `blink` **一次**＋`animover` 自删（九层中唯一自删） | 无 netvar | 蠢人面具（官方名"蠢人面具"）的眨眼层：**帽本体不生成它**，唯一生成点在舞台剧剧本（见生成点表）。[层](../../../raw/current/scripts/prefabs/hats.lua#L7094-L7115)、[注册](../../../raw/current/scripts/prefabs/hats.lua#L7955-L7960) |

### 两盏帽灯与一个技术实体

| 注册名 | 构造 | 差异与证据 |
|---|---|---|
| `minerhatlight` | Transform＋Light＋Network → `FX` 标签 → 固定光参 falloff 0.4／intensity 0.7／radius 2.5／色 (180,195,150)（暖绿） → SetPristine → 主模拟 `persists=false` | 纯 Light 壳无 AnimState；色温永不变化；矿工帽 fueled 链开关（见生成点表）。[工厂](../../../raw/current/scripts/prefabs/hats.lua#L6991-L7014)、[注册](../../../raw/current/scripts/prefabs/hats.lua#L8035-L8035) |
| `alterguardianhatlight` | 同构；falloff 0.5／intensity 0.8／radius 4、**工厂不设色** | 启迪之冠激活时生成；颜色由帽的 `alterguardianhat_updatelight` 按 6 槽内孢子红绿蓝计数动态写（无孢子时回落矿工帽同色），与月种玩法结算同函数。[工厂](../../../raw/current/scripts/prefabs/hats.lua#L7016-L7038)、[调色](../../../raw/current/scripts/prefabs/hats.lua#L3405-L3429)、[注册](../../../raw/current/scripts/prefabs/hats.lua#L8036-L8036) |
| `tophat_container` | Network（**无 Transform**）→ `CLASSIFIED` 标签＋`Hide()` → `container_proxy` 组件 → SetPristine → 主模拟 `SetMaster(暗影储物维度容器)`＋`persists=false` | 高礼帽变魔术师工具后开暗影储物间的隐藏会话代理：无场景实体、只做容器 classified 转发；会话由 magiciantool 开始／停止管理，协议详情见 [[reference/components/container-proxy|container_proxy 组件]]（该页已覆盖此注册）与 [[reference/components/magiciantool|magiciantool 组件]]。[工厂](../../../raw/current/scripts/prefabs/hats.lua#L7837-L7858) |

## 关联机制导航

### 生成点与宿主调用

全 `raw/current/scripts` 反查（`grep -rn "注册名" --include=*.lua`）：11 个 FX 的生成点全部在帽本体回调或舞台剧剧本内，无其他文件引用（`tophat_container` 另见 container-proxy 页）。

| 注册名 | 生成语句（file:line） | 触发条件与宿主链 |
|---|---|---|
| `minerhatlight` | [hats.lua#L681](../../../raw/current/scripts/prefabs/hats.lua#L677-L692) | `miner_turnon`：装备且 fueled 非空（燃料加注 `miner_takefuel`、 fueled 耗尽 `miner_perish` 都重走开关）；生成后 `SetParent(owner.entity)` |
| `alterguardianhatlight` | [hats.lua#L3480](../../../raw/current/scripts/prefabs/hats.lua#L3451-L3484) | `alterguardian_activate`：持有者理智超 `SANITY_BECOME_ENLIGHTENED_THRESH`（sanitydelta 监听）；同时生成的前后层 `alterguardian_hat_equipped` 归 armor_lunarplant_glow_fx 页 |
| `lunarplanthat_fx` | [hats.lua#L3820](../../../raw/current/scripts/prefabs/hats.lua#L3814-L3828) | `lunarplant_onequip`（每次装备重建：先 Remove 旧层）；`owningitem` 指回帽后 `AttachToOwner` |
| `voidclothhat_fx` | [hats.lua#L4048](../../../raw/current/scripts/prefabs/hats.lua#L4042-L4053) | `voidcloth_onequip`；同上模式；buff 层数由影子武器 attackother 链写 `buffed` |
| `wagpunkhat_fx` | [hats.lua#L4464](../../../raw/current/scripts/prefabs/hats.lua#L4450-L4469) | `wagpunk_onequip`；生成即 `level:set(1)`，后续等级由 targettracker 时间阶段写 |
| `inspectacleshat_fx` | [hats.lua#L4947](../../../raw/current/scripts/prefabs/hats.lua#L4941-L4954) | `inspectacles_onequip`；信号等级决定初始 ledstate |
| `rabbithat_fx` | [hats.lua#L5275](../../../raw/current/scripts/prefabs/hats.lua#L5269-L5278) | `rabbit_equip`；生成后立即写 `iswinter` 季节形态 |
| `pumpkinhat_fx` | [hats.lua#L5868](../../../raw/current/scripts/prefabs/hats.lua#L5851-L5871) | `pumpkin_onequip`；生成后 `CopyFaceSymbols` 从帽本体同步雕刻脸 |
| `shadow_thrall_parasitehat_fx` | [hats.lua#L5634](../../../raw/current/scripts/prefabs/hats.lua#L5625-L5638) | `shadowthrall_parasite_onequip`：寄生面具装备即生成；宿主获得 combat retarget／talker 接管等寄生玩法 |
| `bat_bosscorpsehat_fx` | [hats.lua#L6566](../../../raw/current/scripts/prefabs/hats.lua#L6533-L6568) | `bat_bosscorpse_onequip`；`owningitem` 指回帽（供皮肤同步） |
| `mask_halfwit_fx` | **[play_commonfn.lua#L68](../../../raw/current/scripts/play_commonfn.lua#L65-L71)** | 舞台剧《The Veil》的 `do_mask_blink` 台词动作（[play_the_veil.lua#L151](../../../raw/current/scripts/play_the_veil.lua#L151-L151)）：扮 HALFWIT 角色的演员临时获得眨眼层；hats.lua 的 mask_halfwit 帽本体从不生成它（`prefabs` 数组仅声明） |
| `tophat_container` | [hats.lua#L912](../../../raw/current/scripts/prefabs/hats.lua#L910-L922) | `top_onstartusing`：魔术师高礼帽开始使用时生成，停止使用／关闭容器时移除 |

南瓜帽另有**清单外连带本地实体**：帽本体掉在地上且 `showface` 时，`pumpkin_onshowfacedirty` 用 `CreateEntity` 直构一个非联网"地面脸"层跟 `follow_face` 符号（[实现](../../../raw/current/scripts/prefabs/hats.lua#L5896-L5941)）——未注册 Prefab，不属于本页 12 注册，随帽本体字段 `groundfx` 清理。

### 端模型与同步

| 层次 | 事实 | 证据 |
|---|---|---|
| 联网壳 | 9 个跟随层＋2 盏灯都是 Transform＋Network 的联网壳；壳上无动画（除南瓜脸 netvar 与灯参），可见内容在非联网层 | [MakeFollowFx 工厂](../../../raw/current/scripts/prefabs/hats.lua#L7800-L7830)、[灯工厂](../../../raw/current/scripts/prefabs/hats.lua#L6991-L7038) |
| 本地动画层 | 每台非专服机器（客户端由 `OnEntityReplicated` 触发）经 `SpawnFollowFxForOwner` 自建 2-3 个非联网动画实体，父挂持有者＋FollowSymbol；专服零本地层 | [非专服门控](../../../raw/current/scripts/prefabs/hats.lua#L7794-L7797) |
| netvar 面向客户端 | 逐帽扩展的 net_* 变量（level／buffed／skinbuildhash／iswinter／ledstate／face）都注册在联网壳上、脏事件只在非专服监听——本地层换 build／动画；`inspectacleshat_fx`／`shadow_thrall_parasitehat_fx` 的 SoundEmitter／talker 挂在联网壳上，激活音在**各客户端本地**播放 | [wagpunk 等级](../../../raw/current/scripts/prefabs/hats.lua#L7087-L7092)、[inspectacles 音](../../../raw/current/scripts/prefabs/hats.lua#L7344-L7348)、[postinit 挂点](../../../raw/current/scripts/prefabs/hats.lua#L7810-L7814) |
| 染色同步 | 壳的 `colouraddersync` 把持有者染色逐层 `SetAddColour`；`highlightchild` 每层独立 SetOwner（高亮描边可被单独选中） | [染色](../../../raw/current/scripts/prefabs/hats.lua#L7749-L7753)、[高亮](../../../raw/current/scripts/prefabs/hats.lua#L7765-L7765) |

### 清理路径

| 注册名 | 生成侧移除 | 层体兜底 | 证据 |
|---|---|---|---|
| 8 个帽装备跟随层（lunarplant／voidcloth／wagpunk／inspectacles／rabbit／pumpkin／shadow_thrall_parasite／bat_bosscorpse） | 帽 `onunequip`（及重建路径的 onequip 前置 Remove）显式 `Remove` 壳；壳 `OnRemoveEntity` 逐层 Remove 本地动画 | 无 animover（循环动画永播）；壳随帽 Remove 即全部回收 | [FollowFx_OnRemoveEntity](../../../raw/current/scripts/prefabs/hats.lua#L7743-L7747)、[lunarplant 卸下](../../../raw/current/scripts/prefabs/hats.lua#L3830-L3836)、[parasite 卸下](../../../raw/current/scripts/prefabs/hats.lua#L5724-L5727) |
| `mask_halfwit_fx` | 演出结束无人移除壳——靠本地层 `animover` 自删（blink 播完即散）；联网壳若无人 Remove 会残留（舞台剧演员实体随后退场连带） | 唯一自带自删的跟随层 | [层自删](../../../raw/current/scripts/prefabs/hats.lua#L7112-L7112) |
| `minerhatlight` | `miner_turnoff`（卸帽／掉落／装到模型／燃料耗尽）Remove；帽 `OnRemoveEntity` 兜底 Remove | 无 | [turnoff](../../../raw/current/scripts/prefabs/hats.lua#L695-L709)、[onremove](../../../raw/current/scripts/prefabs/hats.lua#L751-L755) |
| `alterguardianhatlight` | `alterguardian_deactivate`（理智回落）Remove；帽被移除时随激活清理链 | 无 | [deactivate](../../../raw/current/scripts/prefabs/hats.lua#L3486-L3495) |
| `tophat_container` | `top_onstopusing`／`top_onclose`（关容器即停用魔术工具）Remove | 无 | [onstopusing](../../../raw/current/scripts/prefabs/hats.lua#L924-L934) |

## 典型接入

### 给装备做三层跟随层

- 前提与选择：需要"随装备符号走、随持有者染色、每端自建动画"的表现时，复刻 MakeFollowFx 模式：联网壳＋`colouraddersync`＋`AttachToOwner` 协议＋非专服 `SpawnFollowFxForOwner`。层动画名按 `idleN` 编号即可被工厂的帧序参数对齐。
- 生效与撤销：装备回调里 `SpawnPrefab` → 设 `owningitem`（皮肤同步需要）→ `AttachToOwner(owner)`；卸下回调**必须显式 Remove**——层是循环动画，不 Remove 会永挂持有者身上。
- 原版依据：[lunarplant_onequip](../../../raw/current/scripts/prefabs/hats.lua#L3814-L3828)（重建式写法）、[rabbit_equip](../../../raw/current/scripts/prefabs/hats.lua#L5269-L5278)（带状态初值写法）。

```lua
-- 服务端：在装备回调里挂三层跟随层（复刻 lunarplant 模式）
local function myhat_onequip(inst, owner)
    if inst.fx ~= nil then
        inst.fx:Remove()
    end
    inst.fx = SpawnPrefab("myhat_fx")   -- MakeFollowFx 注册的联网壳
    inst.fx.owningitem = inst           -- 供 skinbuildhash 皮肤同步
    inst.fx:AttachToOwner(owner)        -- SetParent + 非专服建本地三层
end
```

- 调整与验证：替换点为注册名与 `MakeFollowFx` 的 `createfn`／netvar 扩展；客户端侧不需要额外代码（`OnEntityReplicated` 自动重放装配）。示例只做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **误区：`mask_halfwit_fx` 由蠢人面具装备生成。** 帽本体（`mask_halfwithat`）的 onequip 是普通 mask 装备，从不引用该 FX；唯一生成点在《The Veil》舞台剧台词动作 `do_mask_blink`，给演员临时挂眨眼层。`prefabs = { "mask_halfwit_fx" }` 只是资源依赖声明。[帽装备](../../../raw/current/scripts/prefabs/hats.lua#L2731-L2751)、[剧本生成](../../../raw/current/scripts/play_the_veil.lua#L151-L151)
- **边界：跟随层壳不自带换装符号。** 壳无 AnimState，帽的可见外观仍靠 `_onequip` 的 `swap_hat` 符号 override；本地层是"叠在符号上的附加动画"，移除帽符号不会移除层——清理责任在帽回调。[符号 override](../../../raw/current/scripts/prefabs/hats.lua#L31-L47)
- **边界：本地层的相位靠共享随机帧对齐。** 2-3 层用同一个 `math.random` 帧 `SetFrame`，而 `FollowSymbol` 尾参的帧序参数按层序写死——复制该工厂时漏掉其中一处会出现三层错位。[对齐](../../../raw/current/scripts/prefabs/hats.lua#L7758-L7766)
- **边界：`inspectacleshat_fx` 的激活音是各端本地播。** SoundEmitter 挂在联网壳上，但 `ledstatedirty` 处理器只在非专服注册并带"local fx on clients"注释——同一信号每台客户端各响一次，不做网络音频同步。[注释](../../../raw/current/scripts/prefabs/hats.lua#L7344-L7347)
- **边界：`pumpkinhat_fx` 只有两层、`isfullhelm` 且跟 `headbase_hat`。** 与其余 8 层（3 层、`swap_hat`）不同；地面展示态的"脸"另由帽本体的未注册直构实体承担，不要把 `pumpkinhat_fx` 当地面显示用。[工厂帧区间](../../../raw/current/scripts/prefabs/hats.lua#L8017-L8025)、[地面脸](../../../raw/current/scripts/prefabs/hats.lua#L5896-L5941)
- **误区：`wagpunkhat_classified` 也是 hats.lua 的 FX。** 它是 W.A.R.B.I.S. 帽的 classified 说话代理，注册在独立文件 `wagpunkhat_classified.lua`（[注册](../../../raw/current/scripts/prefabs/wagpunkhat_classified.lua#L146-L146)），hats.lua 只是生成方（[L4633](../../../raw/current/scripts/prefabs/hats.lua#L4629-L4636)）；不属本页 12 注册。
- **边界：帽灯只认装备状态。** `minerhatlight` 的开关挂在 fueled 消耗链上，燃料耗尽会自动灭灯并推 `torchranout`；`alterguardianhatlight` 只在理智激活窗口存在——直接 `SpawnPrefab` 复用得不到开关协议。[miner_perish](../../../raw/current/scripts/prefabs/hats.lua#L721-L737)

## 验证记录

- 源码审阅：2026-11-05，steam-buildid:26335394（实读 `raw/current/BUILD.txt`）。完整读取 `prefabs/hats.lua` 的注册区（`MakeHat` 骨架／`MakeFollowFx` 工厂／全部 12 个 FX 工厂函数与 return 注册清单 L7860-L8038）与 12 注册各自的生成／清理回调（miner L654-L787、top L902-L1004、alterguardian L3370-L3522、lunarplant L3814-L3836、voidcloth L4042-L4064、wagpunk L4400-L4660、inspectacles L4890-L4974、rabbit L5249-L5286、shadow_thrall_parasite L5560-L5798、pumpkin L5820-L5947、bat_bosscorpse L6530-L6604）；完整读取 `play_commonfn.lua`（do_mask_blink 及其舞台剧上下文）并按需读取 `play_the_veil.lua` 调用行、`prefabs/wagpunkhat_classified.lua` 注册行。
- 穷举复核：对 12 个注册名在 `raw/current/scripts` 全域 `grep -rn "注册名" --include=*.lua` 反查——9 个 `*hat_fx` 的 SpawnPrefab 仅命中正文生成点表所列语句；`minerhatlight`／`alterguardianhatlight` 命中 hats.lua 生成＋依赖声明＋注册；`mask_halfwit_fx` 在 hats.lua 仅命中声明与注册，**生成点唯一在 play_commonfn.lua:68**；`tophat_container` 命中 hats.lua 生成／声明／注册与 components/container-proxy.md 引述。`hats.lua` return 清单实数：85 `MakeHat`＋9 `MakeFollowFx`＋3 `Prefab`＝97 注册，与台账"85 帽＋11 FX＋1 技术实体"一致。
- 名称核验：`languages/chinese_s.po` 定向检索——`MINERHATLIGHT`／`ALTERGUARDIANHATLIGHT`／`MASK_HALFWIT_FX`／`SHADOW_THRALL_PARASITEHAT_FX`／`LUNARPLANTHAT_FX`／`VOIDCLOTHHAT_FX`／`WAGPUNKHAT_FX`／`INSPECTACLESHAT_FX`／`RABBITHAT_FX`／`PUMPKINHAT_FX`／`BAT_BOSSCORPSEHAT_FX`／`TOPHAT_CONTAINER` 均无 `STRINGS.NAMES.*` 键。宿主官方名（正文指认用）：`LUNARPLANTHAT`＝"亮茄头盔"（[po](../../../raw/current/scripts/languages/chinese_s.po#L344864-L344867)）、`MINERHAT`＝"矿工帽"（[po](../../../raw/current/scripts/languages/chinese_s.po#L345439-L345442)）、`TOPHAT`＝"高礼帽"（[po](../../../raw/current/scripts/languages/chinese_s.po#L350469-L350472)）、`RABBITHAT`＝"洞穴花环"（[po](../../../raw/current/scripts/languages/chinese_s.po#L348419-L348422)）、`PUMPKINHAT`＝"无面南瓜"（[po](../../../raw/current/scripts/languages/chinese_s.po#L347324-L347327)）、`BAT_BOSSCORPSEHAT`＝"蝙蝠帽"（[po](../../../raw/current/scripts/languages/chinese_s.po#L339199-L339202)）、`INSPECTACLESHAT`＝"检查专用镜"（[po](../../../raw/current/scripts/languages/chinese_s.po#L344129-L344132)）、`MASK_HALFWITHAT`＝"蠢人面具"（[po](../../../raw/current/scripts/languages/chinese_s.po#L345114-L345117)）、`VOIDCLOTHHAT`＝"虚空风帽"（[po](../../../raw/current/scripts/languages/chinese_s.po#L351344-L351347)）、`WAGPUNKHAT`＝"W.A.R.B.I.S.头戴齿轮"（[po](../../../raw/current/scripts/languages/chinese_s.po#L351444-L351447)）、`ALTERGUARDIANHAT`＝"启迪之冠"（[po](../../../raw/current/scripts/languages/chinese_s.po#L338514-L338517)）、`SHADOWTHRALL_PARASITE`＝"虚空面具"（[po](../../../raw/current/scripts/languages/chinese_s.po#L349219-L349222)，宿主怪名；帽本体 `shadow_thrall_parasitehat` 无 NAMES 键）。
- 分档说明：`size_class: complex`——12 注册共享一个参数化工厂且逐帽 netvar 扩展各异；"联网壳＋多本地层＋共享随机相位＋ colouraddersync 广播"的端模型必须整体对照说明；生成点横跨 hats.lua 与舞台剧剧本两个体系；清理分四类（帽回调显式 Remove／animover 自删／fueled 链／容器会话），超出 standard 单页容量。
- 未覆盖：85 顶帽本体的玩法组件与获取途径；`wagpunksteam_hat_up/down`（归 wagpunksteam_hat_up 页）、`alterguardian_hat_equipped` 前后层（归 armor_lunarplant_glow_fx 页）、`tophat_shadow_fx`／`tophat_swirl_fx`／`tophat_using_shadow_fx`（归 tophat_shadow_fx 页）、`wagpunkhat_classified`、`alterguardianhat_projectile` 等连带生成物；南瓜帽地面脸直构实体的完整行为；动画包内容与观感；皮肤 build 的实际资源。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 结果见本页交付记录。
- 游戏运行：未运行；`validation` 保持 `source-reviewed`。

## 相关链接

- [[reference/prefabs/tophat_shadow_fx|魔术师高礼帽影子特效]]（高礼帽的另一套影子 FX，注册在独立文件 tophat_shadow_fx.lua）
- [[reference/prefabs/armor_lunarplant_glow_fx|装备与帽饰跟随光效]]（启迪之冠前后层 `alterguardian_hat_equipped` 与帽片符号跟随的主要页）
- [[reference/prefabs/wagpunksteam_hat_up|WAGpunk 装备蒸汽]]（W.A.R.B.I.S. 帽等级阶段的蒸汽表现，与 wagpunkhat_fx 同帽协作）
- [[reference/prefabs/wormlight|灯具与生物光源的附属光／FX 实体]]（其他光源类附属实体的族页）
- 返回 [[reference/prefabs/fx-index|特效 Prefab 索引]]（装备）
