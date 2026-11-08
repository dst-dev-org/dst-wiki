---
title: "薇洛的打火机（lighter）"
created: "2026-11-06"
updated: "2026-11-08"
type: prefab
summary: "原版手持照明装备 lighter：HANDS 槽武器＋点火工具＋危险炊具，光照由 lighterfire 跟随 FX 提供，燃料在装备燃烧时计时消耗（雨中加速），烹饪另按门禁直接扣油，薇洛技能树解锁吸火灭火、扩大光圈与攻击必燃。"
aliases: ["lighter", "薇洛的打火机", "打火机", "幸运打火机", "吸火"]
tags: [prefab, item, equipment]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/lighter.lua", "prefabs/lighterfire.lua", "prefabs/lighterfire_common.lua", "prefabs/lighterfire_glass.lua", "prefabs/lighterfire_haunteddoll.lua", "prefabs/lighterfire_heart.lua", "prefabs/lighterfire_old.lua", "prefabs/lighterfire_ragged.lua", "prefabs/lighterfire_rose.lua", "prefabs/skilltree_willow.lua", "prefabs/willow.lua", "prefabs/willow_ember.lua", "prefabs/player_common.lua", "prefabskin.lua", "prefabs/skinprefabs.lua", "components/weapon.lua", "components/lighter.lua", "components/equippable.lua", "components/fueled.lua", "components/cooker.lua", "components/burnable.lua", "components/channelcastable.lua", "components/wildfires.lua", "componentactions.lua", "actions.lua", "componentutil.lua", "standardcomponents.lua", "tuning.lua", "recipes.lua", "constants.lua", "mods.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 薇洛的打火机（lighter）

## 快速判断

- 职责与适用：`prefabs/lighter.lua` 顶层 `fn` 注册的单注册手持装备（物品＋装备）：HANDS 槽近战武器＋`lighter` 点火工具＋`dangerouscooker` 炊具；光照由独立 FX Prefab `lighterfire`（及皮肤变体）跟随宿主提供，燃料仅装备燃烧时计时。[注册](../../../raw/current/scripts/prefabs/lighter.lua#L312-L399)
- 优先使用：复刻"燃料照明＋点火工具"按[关联机制导航](#关联机制导航)组装并查 [[cookbook/equipment/hand]]；改伤害／燃料／光圈／点燃概率查[数值](#注册与构造)；给已有打火机挂行为查[典型接入](#典型接入)；对照同批手持照明 [[reference/prefabs/torch]]。
烹饪另扣油：原版仅 expertchef 可开锅、每次 1%，放开门禁后非 expertchef 扣 5% 并烧人（[oncook](../../../raw/current/scripts/prefabs/lighter.lua#L286-L299)）；
- 修改边界：标签与 floater 挂公共端、组件挂主模拟（[分界](../../../raw/current/scripts/prefabs/lighter.lua#L343-L347)）；光照 Light 与粒子都在 `lighterfire` 实体而非本体；技能接线只认持有者 `skilltreeupdater` 组件而非角色本名；皮肤 FX 名单在数据侧 `SKIN_FX_PREFAB`。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `lighter` | 无包装工厂：`fn()` 直接构造，无 postinit | 物品＋装备；`lighterfire` 是独立 FX Prefab（声明于 `prefabs` 数组，装备时实际 `SpawnPrefab`）[注册](../../../raw/current/scripts/prefabs/lighter.lua#L399)、[声明](../../../raw/current/scripts/prefabs/lighter.lua#L8-L15) |

共同初始化（[lighter.lua#L312-L397](../../../raw/current/scripts/prefabs/lighter.lua#L312-L397)）：实体组件（含 SoundEmitter／MiniMapEntity）＋`MakeInventoryPhysics` → bank `lighter`＋build `lighter`＋`idle` → 小地图图标 `lighter.png` → 公共端标签 `dangerouscooker`／`wildfireprotected`／`lighter`／`cooker`／`weapon`（[pristine 优化](../../../raw/current/scripts/prefabs/lighter.lua#L329-L339)）→ `MakeInventoryFloatable("small", 0.05, 0.8)` → `SetPristine`（非主模拟返回）→ 主模拟依次 weapon → lighter → inventoryitem → cooker → equippable → inspectable → burnable → fueled → 技能与雨监听 → `MakeHauntableLaunch` → `OnRemoveEntity`。装备槽用组件默认 HANDS。[equippable 默认](../../../raw/current/scripts/components/equippable.lua#L29)、[EQUIPSLOTS.HANDS](../../../raw/current/scripts/constants.lua#L656)

名称经简中 .po `msgctxt` 核验：`STRINGS.NAMES.LIGHTER`＝薇洛的打火机；通用检视语＝"她的幸运打火机。"，薇洛检视语＝"这是我的幸运打火机！"。[名称](../../../raw/current/scripts/languages/chinese_s.po#L344719-L344722)、[检视](../../../raw/current/scripts/languages/chinese_s.po#L14709-L14712)、[薇洛检视](../../../raw/current/scripts/languages/chinese_s.po#L186190-L186192)

数值（tuning 键＋实际值；`wilson_attack`＝34×倍率 1、`seg_time`＝30 秒、`total_day_time`＝480 秒，[tuning.lua#L19-L37](../../../raw/current/scripts/tuning.lua#L19-L37)）：

| 项 | 键 | 实际值 |
|---|---|---|
| 武器伤害 | `TUNING.LIGHTER_DAMAGE`＝`wilson_attack`×.5 | 17 生命值（源码推导，[tuning.lua#L2671](../../../raw/current/scripts/tuning.lua#L2671)） |
| 燃料总量 | `TUNING.LIGHTER_FUEL`＝`total_day_time`×1.25（[tuning.lua#L1724](../../../raw/current/scripts/tuning.lua#L1724)） | 600 燃料单位＝rate 1 下烧 600 秒；点燃首扣 0.5 秒量、满燃料 1/60（`TURNON_FUELED_CONSUMPTION`，[tuning.lua#L1668-L1669](../../../raw/current/scripts/tuning.lua#L1668-L1669)） |
| 雨中消耗增速 | `TUNING.LIGHTER_RAIN_RATE` | rate＝`1+该值×降水率`，满雨 2；持有者避雨或有 rainimmunity 组件则恒 1（[tuning.lua#L1723](../../../raw/current/scripts/tuning.lua#L1723)、[onupdatefueledraining](../../../raw/current/scripts/prefabs/lighter.lua#L241-L247)） |
| 攻击点燃基础概率 | `TUNING.LIGHTER_ATTACK_IGNITE_PERCENT` | 0.5×目标 `burnable.flammability`，经幸运公式 `LighterIgniteOnAttack` 修正（[tuning.lua#L2670](../../../raw/current/scripts/tuning.lua#L2670)、[公式](../../../raw/current/scripts/componentutil.lua#L2150)） |
| 光照默认参数 | lighterfire_common 硬编码 | 强度 0.75、颜色 (200,150,50)/255、衰减 0.5、半径 1（[CreateLight](../../../raw/current/scripts/prefabs/lighterfire_common.lua#L1-L19)） |
| 光档表 | `TUNING.TORCH_RADIUS`／`TORCH_FALLOFF` | {2,3,4,5}／{0.5,0.6,0.75,0.9}；净变量档 1＝半径 2／衰减 0.5（[tuning.lua#L7043-L7055](../../../raw/current/scripts/tuning.lua#L7044-L7056)） |
| 薇洛光圈技能档 | `TUNING.SKILLS.WILLOW_BRIGHTNESS_1/2` | 2.5／4，作为 `_lightrange` 净变量值写入（[tuning.lua#L7186-L7187](../../../raw/current/scripts/tuning.lua#L7187-L7188)） |

获取途径与消费端：

| 途径 | 证据 |
|---|---|
| 出生自带：DEFAULT 模式薇洛起始物品含 `lighter`（`TUNING.GAMEMODE_STARTING_ITEMS`，willow 装配） | [tuning.lua#L3453](../../../raw/current/scripts/tuning.lua#L3453)、[willow.lua#L25-L28](../../../raw/current/scripts/prefabs/willow.lua#L25-L28) |
| 配方：rope 1＋goldnugget 1＋petals 3，`TECH.NONE`，`builder_tag="pyromaniac"`（薇洛，[willow.lua#L134](../../../raw/current/scripts/prefabs/willow.lua#L134)） | [recipes.lua#L93](../../../raw/current/scripts/recipes.lua#L93) |
| 消费端：装备燃烧带 `fire` 标签，参与薇洛近火回理智（`sanityfn`） | [willow.lua#L38-L60](../../../raw/current/scripts/prefabs/willow.lua#L38-L60)、[fire 标签](../../../raw/current/scripts/components/burnable.lua#L13) |
| 消费端：夏季野火排除表含 `wildfireprotected` | [wildfires.lua#L27](../../../raw/current/scripts/components/wildfires.lua#L27) |

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：weapon | `SetDamage(LIGHTER_DAMAGE)`＋`SetOnAttack(onattack)` 攻击掷点引燃；无 finiteuses、无磨损 | 主模拟 | [[reference/components/weapon#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/lighter.lua#L349-L351)、[SetDamage](../../../raw/current/scripts/components/weapon.lua#L44-L46) |
| 组件：lighter | 点火能力：`LIGHT` 动作→`Light(target, doer)`→目标 `burnable:Ignite`；组件自身无消耗逻辑 | 主模拟挂载，`lighter` 标签公共端镜像 | [[reference/components/lighter#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/lighter.lua#L354) |
| 组件：cooker | `oncookfn=oncook`：非 expertchef 烧人（5 火伤＋`burnt`＋扣 5% 燃料）、expertchef 扣 1% | 主模拟；`dangerouscooker` 门禁见[烹饪](#烹饪cooker) | [[reference/components/cooker#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/lighter.lua#L358-L359) |
| 组件：equippable | OnEquip／OnUnequip／OnPocket／OnEquipToModel 四回调；槽默认 HANDS | 主模拟；ToPocket 在入袋时触发 | [[reference/components/equippable#关键接口]]；[接线](../../../raw/current/scripts/prefabs/lighter.lua#L362-L366)、[ToPocket](../../../raw/current/scripts/components/equippable.lua#L109-L113) |
| 组件：fueled | `InitializeFuelLevel(LIGHTER_FUEL)`、`fueltype=FUELTYPE.LIGHTER`＋`accepting=true`（可喂 willow_ember）、耗尽 `Remove` | 主模拟；Start/StopConsuming 由 burnable Ignite/Extinguish 联动（[启动](../../../raw/current/scripts/components/burnable.lua#L380-L382)、[停止](../../../raw/current/scripts/components/burnable.lua#L486-L488)）；雨衰经 `SetUpdateFn` 改 rate | [[reference/components/fueled#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/lighter.lua#L378-L385) |
| 组件：burnable | 状态开关：`canlight=false`、`fxprefab=nil`（自身无燃 FX）；穿脱／入袋 Ignite/Extinguish；燃烧时带 `fire` 标签 | 主模拟 | [[reference/components/burnable#关键接口]]；[配置](../../../raw/current/scripts/prefabs/lighter.lua#L374-L376) |
| 组件：inspectable／inventoryitem | 检视；拾取持有；`OnRemoveEntity` 只清 `snuff_fx` | 主模拟 | [[reference/components/inspectable#关键接口]]、[[reference/components/inventoryitem#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/lighter.lua#L356-L370)、[OnRemoveEntity](../../../raw/current/scripts/prefabs/lighter.lua#L301-L306) |
| 组件：channelcastable | attuned 技能激活时动态 AddComponent，`SetStrafing(false)`＋吸火启停回调；失活／换人 `RemoveComponent` | 主模拟；仅持有者激活 `willow_attuned_lighter` 时存在，卸下自动停止（[OnUnequipped](../../../raw/current/scripts/components/channelcastable.lua#L29-L32)） | [[reference/components/channelcastable#关键接口]]；[动态挂载](../../../raw/current/scripts/prefabs/lighter.lua#L113-L146) |
| 组件：floater／hauntable | `MakeInventoryFloatable("small", 0.05, 0.8)`；`MakeHauntableLaunch` 作祟弹飞 | floater 公共端，hauntable 主模拟 | [[reference/components/floater#关键接口]]、[[reference/components/hauntable#按效果选工厂]]；[调用](../../../raw/current/scripts/prefabs/lighter.lua#L341)、[helper](../../../raw/current/scripts/standardcomponents.lua#L1772-L1790) |
| Replica／同步：`lighter._lightrange` | lighterfire 用 `net_tinybyte` 同步光档，客户端 `lightrangedirty`→`Light:SetRadius/Falloff` 查 TORCH 表；服务端走 `SetLightRange`（值变化才生效） | 光档初始 1；技能档 2.5／4 整型化规则在引擎侧 | [[reference/components/fueled#关键接口]]；[net](../../../raw/current/scripts/prefabs/lighterfire_common.lua#L77-L83)、[dirty](../../../raw/current/scripts/prefabs/lighterfire_common.lua#L36-L46) |
| API／事件：`itemranout` | 装备态耗尽先 `Remove` 再 `PushEvent`（`announce="ANNOUNCE_TORCH_OUT"`＝"火灭了！"） | 玩家侧监听：自动补装同 prefab 同槽工具 | [耗尽块](../../../raw/current/scripts/prefabs/lighter.lua#L261-L284)、[自动补装](../../../raw/current/scripts/prefabs/player_common.lua#L612-L623)、[播报键](../../../raw/current/scripts/languages/chinese_s.po#L7650-L7652) |
| API／数据：`lighter`／`cooker`／`weapon`／`dangerouscooker` 标签 | 动作收集：手持/装备对 `canlight` 目标给 LIGHT（先发 `onstartedfire`，并抑制对可燃目标的 ATTACK）；cooker/fuel 按标签收集 | 公共端＋两端读取 | [LIGHT 收集](../../../raw/current/scripts/componentactions.lua#L1480-L1484)、[ATTACK 抑制](../../../raw/current/scripts/componentactions.lua#L1993)、[EQUIPPED](../../../raw/current/scripts/componentactions.lua#L2384-L2388)、[LIGHT.fn](../../../raw/current/scripts/actions.lua#L2689-L2697) |
| API／数据：薇洛技能接线 | 监听宿主 `onactivateskill_server`／`ondeactivateskill_server` 刷新：`willow_attuned_lighter`→channelcastable；`willow_lightradius_1/2`→`applyskillbrightness`；`willow_controlled_burn_1`→onattack 必燃；consumption_1~3 在 lighter.lua 中被注释，未生效 | 需持有者 skilltreeupdater（[[reference/components/skilltreeupdater#关键接口]]）；技能定义 [skilltree_willow.lua#L53-L68](../../../raw/current/scripts/prefabs/skilltree_willow.lua#L53-L68)、[L97-L108](../../../raw/current/scripts/prefabs/skilltree_willow.lua#L97-L108)、[L154-L175](../../../raw/current/scripts/prefabs/skilltree_willow.lua#L154-L175) | [RefreshAttunedSkills](../../../raw/current/scripts/prefabs/lighter.lua#L113-L146)、[监听](../../../raw/current/scripts/prefabs/lighter.lua#L148-L158) |
| 关联 Prefab：`lighterfire` | 装备时 `SpawnPrefab`→`SetParent(owner)`＋`FollowSymbol("swap_object")`＋`AttachLightTo(owner)`；卸下／入袋／模型态 `Remove` | 主模拟生成；皮肤时改用 `SKIN_FX_PREFAB[皮肤名]` 名单，无则回退 `{ "lighterfire" }`（[onequip](../../../raw/current/scripts/prefabs/lighter.lua#L176-L188)） | 皮肤 FX 注册：[skinprefabs.lua#L7462-L7538](../../../raw/current/scripts/prefabs/skinprefabs.lua#L7462-L7538)、[SKIN_FX_PREFAB](../../../raw/current/scripts/prefabskin.lua#L4095) |
| 关联 Prefab：`willow_ember` | 唯一可喂燃料：`fuel.fueltype=FUELTYPE.LIGHTER`、`fuelvalue=HUGE_FUEL`（270 单位，湿 0.75 折）；attuned 吸火把地面 ember 收回背包 | ember 本体 `canbepickedup=false`，只能靠吸火或技能产出 | [fuel](../../../raw/current/scripts/prefabs/willow_ember.lua#L975-L977)、[喂食收集](../../../raw/current/scripts/componentactions.lua#L1303-L1317) |
| 资源：anim／swap／贴图 | lighter.zip（bank `lighter`）／swap_lighter.zip（持握符号 `swap_object`）；lighterfire 粒子贴图 fx/torchfire.tex | 静态声明；皮肤 build 经 `GetSkinBuild` 覆盖持握符号 | [声明](../../../raw/current/scripts/prefabs/lighter.lua#L1-L6)、[onequip](../../../raw/current/scripts/prefabs/lighter.lua#L160-L174) |

### lighterfire（火焰 FX 实体，同页简述）

六个变体均由工厂 `MakeLighterFire(name, ...)` 注册（[lighterfire.lua#L114](../../../raw/current/scripts/prefabs/lighterfire.lua#L114)、[工厂](../../../raw/current/scripts/prefabs/lighterfire_common.lua#L48-L108)）：`lighterfire`＋皮肤变体 `lighterfire_glass`／`_haunteddoll`／`_heart`／`_old`／`_ragged`／`_rose`（各变体仅粒子包络与 `fx_offset` 不同，见各文件 `master_postinit`）。纯 FX 实体（`FX` 标签、`persists=false`）＋循环音 lighter_LP；子 Light 实体非联网、`persists=false`（[CreateLight](../../../raw/current/scripts/prefabs/lighterfire_common.lua#L1-L19)、[AttachLightTo](../../../raw/current/scripts/prefabs/lighterfire_common.lua#L21-L23)）；单个火粒子 emitter（fx/torchfire.tex），Dedicated 服务器跳过粒子（[common_postinit](../../../raw/current/scripts/prefabs/lighterfire.lua#L67-L106)）；`OnRemoveEntity` 移除子 Light（[lighterfire_common.lua#L32-L34](../../../raw/current/scripts/prefabs/lighterfire_common.lua#L32-L34)）。

### 吸火通道施法（willow_attuned_lighter）

- 接线：持有者激活技能后装备期内动态挂 channelcastable（[RefreshAttunedSkills](../../../raw/current/scripts/prefabs/lighter.lua#L113-L146)）；玩家侧 channelcaster 常驻（[player_common.lua#L2855-L2857](../../../raw/current/scripts/prefabs/player_common.lua#L2855-L2857)），触发 `START_CHANNELCAST`／`STOP_CHANNELCAST`（[收集](../../../raw/current/scripts/componentactions.lua#L2261-L2270)、[SCENE](../../../raw/current/scripts/componentactions.lua#L2595-L2604)、[strfn/fn](../../../raw/current/scripts/actions.lua#L4313-L4327)），文案"吸火"／"停止吸火"（[po](../../../raw/current/scripts/languages/chinese_s.po#L1914-L1916)）。
- 执行：每 0.3 秒 `UpdateSnuff` 在吸火者周围 2.5 距离（`ABSORB_RANGE`）找 `smolder`/`fire`/`willow_ember` 单选标签、排除 `INLIMBO`/`snuffed`（[UpdateSnuff](../../../raw/current/scripts/prefabs/lighter.lua#L23-L63)）：ember→加 `snuffed`，10 帧后未死亡则 `GiveItem` 收进背包；燃烧物→`Extinguish`；阴燃物→`SmotherSmolder`；每次命中播 `ember_absorb` 并 `SpawnPrefab` 对应 FX `channel_absorb_embers`/`_fire`/`_smoulder`，吸火者身边另有跟随 `channel_absorb_fire_fx`（[声明](../../../raw/current/scripts/prefabs/lighter.lua#L8-L15)）。
- 清理：停止时取消任务、KillSound、KillFX（[OnStopChanneling](../../../raw/current/scripts/prefabs/lighter.lua#L80-L93)）；卸下经 channelcastable `OnUnequipped` 停止；本体移除时 `OnRemoveEntity` 清 `snuff_fx`。吸火本身**不回填燃料**，补给见[陷阱](#常见陷阱与限制)。

### 攻击引燃（onattack）

服务端执行序：combat → `weapon:OnAttack` → onattack。onattack（[lighter.lua#L231-L239](../../../raw/current/scripts/prefabs/lighter.lua#L231-L239)）：目标有效且有 burnable 时，攻击者激活 `willow_controlled_burn_1` 或 `TryLuckRoll(attacker, 0.5×目标 flammability, LighterIgniteOnAttack)` 成功则 `burnable:Ignite(nil, attacker)`（[掷点公式](../../../raw/current/scripts/componentutil.lua#L2150)、[TryLuckRoll](../../../raw/current/scripts/componentutil.lua#L2237-L2251)）。Ignite 的 source 带受控燃烧：`controlled_burn_2/3` 分别把生物燃烧时长×3／燃烧伤害×1.5（`CONTROLLED_BURN_DURATION_CREATURE_MULT`/`_DAMAGE_MULT`，[tuning.lua#L8119-L8120](../../../raw/current/scripts/tuning.lua#L8120-L8121)），非生物火标记 `stokeablefire` 可被 `STOKEFIRE` 拨旺（[burnable.lua#L353-L365](../../../raw/current/scripts/components/burnable.lua#L353-L365)）。攻击不耗耐久；燃料的计时消耗只随燃烧时间，烹饪则是另一个直接扣油入口（见下节），不要把"只随时间消耗"当成全部消耗路径。

### 烹饪（cooker）

COOK 收集两分支（[USEITEM](../../../raw/current/scripts/componentactions.lua#L1113-L1118)、[EQUIPPED](../../../raw/current/scripts/componentactions.lua#L2292-L2298)）：目标 `cookable` 且非 `fire`/`catchable`、打火机无 `fueldepleted`，且非 `dangerouscooker` 或 chef `expertchef`。服务端 `COOK.fn`→`cooker:CanCook`（同一门禁，[cooker.lua#L12-L23](../../../raw/current/scripts/components/cooker.lua#L12-L23)）→`CookItem`→oncook（[lighter.lua#L286-L299](../../../raw/current/scripts/prefabs/lighter.lua#L286-L299)）：非 expertchef 走烧人分支（`DoFireDamage(5)`＋`burnt` 事件＋扣 5% maxfuel），expertchef 扣 1%。原版 expertchef 角色为薇洛（[willow.lua#L135](../../../raw/current/scripts/prefabs/willow.lua#L135)）、沃利（[warly.lua#L20](../../../raw/current/scripts/prefabs/warly.lua#L20)）、沃尔特（[walter.lua#L1135](../../../raw/current/scripts/prefabs/walter.lua#L1135)）。

## 典型接入

### 让非 expertchef 角色也能用打火机烤食物

- 前提与选择：服务端＋客户端 `modmain.lua`。`dangerouscooker` 同时挡客户端动作收集（两条 cooker 分支）与服务端 `CanCook`，删标签比改组件覆盖面小；`AddPrefabPostInit` 在两端实例上执行，标签保持一致。modmain 环境仅直挂 `TUNING` 等白名单，`TheWorld` 等须 `GLOBAL.` 前缀（[CreateEnvironment](../../../raw/current/scripts/mods.lua#L258-L292)）。
- 生效与撤销：影响所有 lighter 实例（拾取与存档重建都重跑 postinit）；停用模组后新实例恢复原标签。
- 原版依据：cooker 挂载与 oncook 双分支（[挂载](../../../raw/current/scripts/prefabs/lighter.lua#L358-L359)、[oncook](../../../raw/current/scripts/prefabs/lighter.lua#L286-L299)）。

```lua
-- modmain.lua：打火机对全员开放烹饪
AddPrefabPostInit("lighter", function(inst)
    inst:RemoveTag("dangerouscooker")
end)
```

- 调整与验证：删标签后非 expertchef 烹饪走烧人分支：5 火伤＋`burnt`＋扣 5% 燃料；expertchef 扣 1%（满燃料 6 单位）。只想放开个别角色时，保留标签、给角色 postinit `AddTag("expertchef")`。

## 常见陷阱与限制

- **误区：放背包或地上也在耗油**：燃料消耗由 burnable 燃烧联动（`Ignite`→`StartConsuming`、`Extinguish`→`StopConsuming`），点燃只在装备时发生；入袋（`onpocket`）、卸下、上模型都立即灭火停耗。[联动](../../../raw/current/scripts/components/burnable.lua#L380-L382)、[onpocket](../../../raw/current/scripts/prefabs/lighter.lua#L227-L229)
- **误区：吸火能给打火机充能**：`UpdateSnuff` 只灭火／收 ember，无任何 `fueled:DoDelta`；补燃料只有喂 `willow_ember` 一条路（`fueltype=LIGHTER` 经 `lighter_fuel`/`lighter_fueled` 标签配对走 ADDFUEL）。[CanAcceptFuelItem](../../../raw/current/scripts/components/fueled.lua#L169-L175)、[TakeFuelItem](../../../raw/current/scripts/components/fueled.lua#L197-L230)
- **边界：默认光照半径两端不对称**：光实体硬编码半径 1（[CreateLight](../../../raw/current/scripts/prefabs/lighterfire_common.lua#L1-L19)）；"档 1＝半径 2"只在 `lightrangedirty` 处理器应用——客户端监听该事件（初始复制即触发），服务端只在 `SetLightRange` 值变化时显式调用，无技能时值不变，主机侧维持半径 1。改亮度应走 fx `SetLightRange` 或 TORCH 表，直接改本地 Light 不跨端；观感未运行验证。
- **边界：低理智起火不存在**：旧版"薇洛低理智烧东西"当前无实现：`TUNING.WILLOW_LIGHTFIRE_SANITY_THRESH` 全 scripts grep 无消费者（仅 [tuning.lua#L2672](../../../raw/current/scripts/tuning.lua#L2672) 定义）；Bernie（`bernie_inactive`）独立 Prefab，lighter.lua 无接线；薇洛火相关理智逻辑只有近火回理智。检索范围：raw/current 全部 .lua。
- **边界：点燃的打火机不传播燃烧**：本体无 propagator（未用 `MakeSmallPropagator`），`burnable:Ignite` 的蔓延分支不触达；`canlight=false` 挡住外部点火与 LIGHT 收集，本体不会被别人点着。[配置](../../../raw/current/scripts/prefabs/lighter.lua#L374-L376)
- **边界：耗尽先 `Remove` 再发 `itemranout`**：顺序是自动补装的前提；仅装备态耗尽广播，非装备态静默 `Remove`。[onfuelchange](../../../raw/current/scripts/prefabs/lighter.lua#L261-L284)
- **边界：技能接线认组件不认角色**：任何持有 `skilltreeupdater` 且激活对应技能的实体都能触发刷新；无该组件的持有者走 `RemoveComponent("channelcastable")` 并复位光档。[RefreshAttunedSkills](../../../raw/current/scripts/prefabs/lighter.lua#L113-L146)

## 验证记录

- 源码审阅：2026-11-07（第二轮修订），Steam buildid 26335394。复核 oncook 两条扣油分支与 cooker 门禁、攻击回调的 skilltreeupdater 判空。早前核验 prefabs/lighter.lua 与六个 lighterfire 变体、weapon／equippable／fueled／cooker／channelcastable、burnable 联动、LIGHT／COOK／ADDFUEL／START_CHANNELCAST／STOKEFIRE、TryLuckRoll、tuning 换算、配方与起始物品、willow 标签、willow_ember、SKIN_FX_PREFAB、wildfires、player_common、po 与 mods 白名单；同文件无 quagmire 分支。
- 游戏运行：未运行。未覆盖：默认光半径两端不对称的观感、WILLOW_BRIGHTNESS 2.5 经 net_tinybyte 的取整、吸火收 ember 实机表现、600 秒燃尽与雨衰为源码推导、皮肤实装外观。
- 结构检查：validate_page 通过

返回 [[reference/prefabs/tools-field-index|工具与野外]]、[[reference/prefabs/index|Prefab]]。
- 2026-11-08 build 26474873：本页随版本迁移升版（锚点迁移 3 处），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-11）。
