---
title: "幽灵灵药环绕与滴落特效（ghostlyelixir_slowregen_fx）"
created: "2026-11-01"
updated: "2026-11-05"
type: prefab
summary: "36 个通用表注册承载温蒂幽灵灵药的 buff 表现：9 种药效 × 阿比盖尔／玩家（幽魂花冠饮用）两套资源 × 环绕主体与地面滴落两形态，由灵药 buff 实体在服务端生成；特效本身不承载任何药效。"
aliases: ["ghostlyelixir_slowregen_fx", "ghostlyelixir_fastregen_fx", "ghostlyelixir_shield_fx", "ghostlyelixir_attack_fx", "ghostlyelixir_speed_fx", "ghostlyelixir_retaliation_fx", "ghostlyelixir_revive_fx", "ghostlyelixir_shadow_fx", "ghostlyelixir_lunar_fx", "ghostlyelixir_slowregen_dripfx", "ghostlyelixir_fastregen_dripfx", "ghostlyelixir_shield_dripfx", "ghostlyelixir_attack_dripfx", "ghostlyelixir_speed_dripfx", "ghostlyelixir_retaliation_dripfx", "ghostlyelixir_revive_dripfx", "ghostlyelixir_shadow_dripfx", "ghostlyelixir_lunar_dripfx", "ghostlyelixir_player_slowregen_fx", "ghostlyelixir_player_fastregen_fx", "ghostlyelixir_player_shield_fx", "ghostlyelixir_player_attack_fx", "ghostlyelixir_player_speed_fx", "ghostlyelixir_player_retaliation_fx", "ghostlyelixir_player_revive_fx", "ghostlyelixir_player_shadow_fx", "ghostlyelixir_player_lunar_fx", "ghostlyelixir_player_slowregen_dripfx", "ghostlyelixir_player_fastregen_dripfx", "ghostlyelixir_player_shield_dripfx", "ghostlyelixir_player_attack_dripfx", "ghostlyelixir_player_speed_dripfx", "ghostlyelixir_player_retaliation_dripfx", "ghostlyelixir_player_revive_dripfx", "ghostlyelixir_player_shadow_dripfx", "ghostlyelixir_player_lunar_dripfx", "幽灵灵药特效", "阿比盖尔灵药特效", "灵药环绕特效", "灵药滴落特效", "幽魂花冠饮用特效"]
tags: [prefab, fx]
size_class: complex
dst_build: "steam-buildid:26335394"
sources: ["prefabs/fx.lua", "fx.lua", "prefabs/ghostly_elixirs.lua", "entityscript.lua", "components/debuffable.lua", "components/debuff.lua", "components/ghostlyelixir.lua", "components/locomotor.lua", "components/equippable.lua", "tuning.lua", "prefabs/abigail.lua", "prefabs/player_common.lua", "prefabs/hats.lua", "stategraphs/SGwilson.lua", "actions.lua", "componentactions.lua", "prefabs/abigail_flower.lua", "componentutil.lua", "widgets/healthbadge.lua", "components/pethealthbar.lua", "recipes.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 幽灵灵药环绕与滴落特效（ghostlyelixir_slowregen_fx）

## 快速判断

- 职责与适用：36 个通用表注册承载温蒂（Wendy）幽灵灵药的 buff 表现——9 种药效 × 阿比盖尔／玩家两套资源 × 环绕主体（`_fx`）与地面滴落（`_dripfx`）两形态；纯表现实体，药效全在灵药 buff 侧。[条目组](../../../raw/current/scripts/fx.lua#L1156-L1480)、[tunings](../../../raw/current/scripts/prefabs/ghostly_elixirs.lua#L87-L346)
- 优先使用：想让自定义目标获得"药效＋表现"走 [典型接入](#典型接入) 的原版 buff 链（组件路由见 [[reference/components/ghostlyelixir]] 与 [[reference/components/ghostlyelixirable]]）；只想借环绕表现直接父挂生成。工厂公共协议见 [[reference/mechanisms/generic-fx-factory|通用 FX 工厂]]。
- 重要副作用：主 fx 由 buff 生成时父挂宿主、随宿主移动；滴落 fx 按固定周期（15 秒，busy／inlimbo 抑制）落在宿主当前位置、不父挂；主 fx 条目自带药效对应音效（例外：`player_shadow_fx`／`player_lunar_fx` 的 sound 行被注释，无声）。[attach 生成](../../../raw/current/scripts/prefabs/ghostly_elixirs.lua#L499-L502)、[driptask](../../../raw/current/scripts/prefabs/ghostly_elixirs.lua#L493-L493)、[无声例外](../../../raw/current/scripts/fx.lua#L1376-L1390)
- 修改边界：生成方是仅主模拟存在的灵药 buff 实体，`SpawnPrefab` 全在服务端，各非专服模拟各自创建本地表现；玩家侧 shadow／lunar 4 名在本快照 scripts 内无生成点。complex 档整体保留范围：36 名同属一条 buff 驱动的生成链（主 fx 父挂＋drip 落点节流＋同一组清理路径），按同族整页保留，不按 9 药效拆页。

## 注册与构造

36 名按公式成组；`<药效>` ∈ `slowregen`／`fastregen`／`shield`／`attack`／`speed`／`retaliation`／`revive`／`shadow`／`lunar`：

```text
ghostlyelixir_<药效>_fx            阿比盖尔侧环绕主体（9 名）
ghostlyelixir_<药效>_dripfx        阿比盖尔侧地面滴落（9 名）
ghostlyelixir_player_<药效>_fx     玩家侧环绕主体（9 名）
ghostlyelixir_player_<药效>_dripfx 玩家侧地面滴落（9 名）
```

- 共同结构：全部是 [[reference/mechanisms/generic-fx-factory|通用 FX 工厂]] 数据条目，无 `animqueue`／`update_while_paused`／染色字段。环绕主体 `bank`=`build`=`abigail_vial_fx`（玩家版 `player_vial_fx`）、`fn`=`FinalOffset3`；滴落 `bank`＝`anim`（阿比盖尔 `abigail_buff_drip`、玩家 `player_elixir_buff_drip`），`build` 阿比盖尔侧为 `abigail_vial_fx`、玩家侧为 `player_vial_fx`，`fn` 内联：两条 drip 均借 `abigail_vial_fx` 的符号做 `OverrideSymbol("fx_swap", "abigail_vial_fx", "fx_<药效>_02")` ＋ `SetFinalOffset(3)`；滴落条目均无声效。[阿比盖尔主 fx](../../../raw/current/scripts/fx.lua#L1156-L1226)、[阿比盖尔 drip](../../../raw/current/scripts/fx.lua#L1228-L1316)、[玩家主 fx](../../../raw/current/scripts/fx.lua#L1320-L1390)、[玩家 drip](../../../raw/current/scripts/fx.lua#L1392-L1480)
- 命名核验：36 个注册名在简中 .po 均无 `STRINGS.NAMES` 显示名键（po 的 `GHOSTLYELIXIR_*` 键只覆盖九瓶灵药物品本体），标题中文为已核验效果描述、非官方译名；九瓶药水的官方名见下表（[po:343209-343252](../../../raw/current/scripts/languages/chinese_s.po#L343209-L343252)）。
- 玩家版 shadow／lunar（4 名）在 [tunings](../../../raw/current/scripts/prefabs/ghostly_elixirs.lua#L319-L345) 中无 `fx_player`／`dripfx_player` 引用，注册处源码注释亦标 "This is unused"（[fx.lua:1376](../../../raw/current/scripts/fx.lua#L1376-L1382)、[fx.lua:1384](../../../raw/current/scripts/fx.lua#L1384-L1390)）；玩家实际喝到 super 灵药时回退用阿比盖尔版名。

| 药效（注册名段） | 主 fx：anim／sound | drip 覆写符号 | 对应灵药物品（po 官方名） | 疗效归属（buff／物品侧，非本 FX 职责） |
|---|---|---|---|---|
| `slowregen` | `buff_regen`／`.../buff/gen` | `fx_regen_02` | 亡者补药 Revenant Restorative | 周期回血：1 秒 ×2 hp、时长 1 天（玩家 1 hp/秒、时长 20 秒）；[条目](../../../raw/current/scripts/fx.lua#L1156-L1162)、[tunings](../../../raw/current/scripts/prefabs/ghostly_elixirs.lua#L89-L123) |
| `fastregen` | `buff_heal`／`.../buff/gen` | `fx_heal_02` | 灵魂万灵药 Spectral Cure-All | 周期回血：1 秒 ×20 hp、时长 30 秒；[条目](../../../raw/current/scripts/fx.lua#L1164-L1170)、[tunings](../../../raw/current/scripts/prefabs/ghostly_elixirs.lua#L124-L159) |
| `shield` | `buff_shield`／`.../buff/shield` | `fx_shield_02` | 不屈药剂 Unyielding Draught | `shield_prefab`=`abigailforcefieldbuffed`；玩家侧 50% 减伤＋受击起 `elixir_player_forcefield`；[条目](../../../raw/current/scripts/fx.lua#L1172-L1178)、[tunings](../../../raw/current/scripts/prefabs/ghostly_elixirs.lua#L222-L252) |
| `attack` | `buff_attack`／`.../buff/attack` | `fx_attack_02` | 夜影万金油 Nightshade Nostrum | Abigail `UpdateDamage` 且攻击档按 night 计；玩家侧另挂 `ghostvision_buff`；[条目](../../../raw/current/scripts/fx.lua#L1180-L1186)、[tunings](../../../raw/current/scripts/prefabs/ghostly_elixirs.lua#L160-L191) |
| `speed` | `buff_speed`／`.../buff/speed` | `fx_speed_02` | 强健精油 Vigor Mortis | 非玩家使用外部移速倍率 ×1.75；玩家改走 `vigorbuff` 标签等逻辑；药水可被作祟给幽灵加速。[条目](../../../raw/current/scripts/fx.lua#L1188-L1194)、[tunings](../../../raw/current/scripts/prefabs/ghostly_elixirs.lua#L192-L221) |
| `retaliation` | `buff_retaliation`／`.../buff/retaliation` | `fx_retaliation_02` | 蒸馏复仇 Distilled Vengeance | `shield_prefab`=`abigailforcefieldretaliation`；玩家侧同 shield 减伤＋受击反击；[条目](../../../raw/current/scripts/fx.lua#L1196-L1202)、[tunings](../../../raw/current/scripts/prefabs/ghostly_elixirs.lua#L253-L284) |
| `revive` | `buff_revive`／`.../buff/retaliation`（源码 TODO 注明待换独有声） | `fx_revive_02` | 恐怖经历 Ghastly Experience | 绑定等级直接拉满 `ghostlybond:SetBondLevel(3)`，时长仅 2 秒（玩家 0.3 秒）；[条目](../../../raw/current/scripts/fx.lua#L1204-L1210)、[tunings](../../../raw/current/scripts/prefabs/ghostly_elixirs.lua#L285-L317) |
| `shadow` | `buff_shadow`／`meta5/abigail/abigail_shadow_buff` | `fx_shadow_02` | 诅咒之苦 Cursed Vexation | super 灵药（`super_elixir_buff` 槽）；表现消费在 Abigail 侧 `GetDebuff("super_elixir_buff")` 检查；[条目](../../../raw/current/scripts/fx.lua#L1212-L1218)、[tunings](../../../raw/current/scripts/prefabs/ghostly_elixirs.lua#L319-L327) |
| `lunar` | `buff_lunar`／`meta5/abigail/abigail_lunar_buff` | `fx_lunar_02` | 光之怒 Luminous Wrath | super 灵药；目标位面伤害加成（gestalt 分档）；[条目](../../../raw/current/scripts/fx.lua#L1220-L1226)、[tunings](../../../raw/current/scripts/prefabs/ghostly_elixirs.lua#L328-L345) |

## 生成链与生命周期

| 阶段 | 行为 | 条件与证据 |
|---|---|---|
| 上药入口 | 三条路由最终都进 `ghostlyelixir:Apply` → `DoApplyElixir`：对阿比盖尔 GIVE（[actions.lua:2300](../../../raw/current/scripts/actions.lua#L2300-L2301)）、对阿比盖尔花转投绑定幽灵（[覆写](../../../raw/current/scripts/prefabs/abigail_flower.lua#L212-L214)，见 [[reference/components/ghostlyelixirable]]）、戴幽魂花冠的玩家饮用（见下"玩家饮用链"） | 目标死亡或幽灵时 `EntityScript:AddDebuff` 直接 false，不生成任何 fx（[entityscript.lua:2140-2154](../../../raw/current/scripts/entityscript.lua#L2140-L2154)、[存活判定](../../../raw/current/scripts/componentutil.lua#L15-L20)） |
| 换药先行 | 目标身上已有不同 prefab 的同槽 buff 时，`pre_buff_fn` 先 `RemoveDebuff` 旧 buff 再上新（同槽名：普通药 `elixir_buff`、super 药 `super_elixir_buff`） | [DoApplyElixir](../../../raw/current/scripts/prefabs/ghostly_elixirs.lua#L348-L367)、[pre_buff_fn 时机](../../../raw/current/scripts/entityscript.lua#L2145-L2148) |
| 新 buff attach | buff 实体（服务端专用、`persists=false`、CLASSIFIED、`keepondespawn=true`）父挂目标；执行 ONAPPLY；按 TICK_RATE 起周期任务；起 drip 周期任务；监听宿主 `death` | [buff_fn](../../../raw/current/scripts/prefabs/ghostly_elixirs.lua#L573-L606)、[buff_OnAttached](../../../raw/current/scripts/prefabs/ghostly_elixirs.lua#L475-L503)、[槽位注册](../../../raw/current/scripts/components/debuffable.lua#L72-L93) |
| 主 fx 生成 | `SpawnPrefab(fx 或 fx_player)` 后父挂目标（`target:HasTag("player")` 选名）；本地表现经代理转为目标子实体、随宿主移动；音效随条目在本地表现创建时播放 | 仅 `not target.inlimbo` 时生成，一次 attach 只生成一次；[生成点](../../../raw/current/scripts/prefabs/ghostly_elixirs.lua#L499-L502)、[父挂语义](../../../raw/current/scripts/prefabs/fx.lua#L25-L28) |
| drip 周期 | `driptask` 每 15 秒（`GHOSTLYELIXIR_DRIP_FX_DELAY`=`seg_time/2`）调用 `buff_DripFx`，首滴延迟 3.75 秒（×0.25） | [driptask](../../../raw/current/scripts/prefabs/ghostly_elixirs.lua#L493-L493)、[常量](../../../raw/current/scripts/tuning.lua#L4855-L4855)、[seg_time](../../../raw/current/scripts/tuning.lua#L19-L19) |
| drip 节流与落点 | 仅当目标非 inlimbo 且 SG 无 `busy` 状态标签才 `SpawnPrefab(dripfx 名)`，落在目标当前位置（不父挂、不跟随） | [buff_DripFx](../../../raw/current/scripts/prefabs/ghostly_elixirs.lua#L467-L473)；玩家饮药状态 `drinkelixir` 仅前 4 帧带 `busy`（第 4 帧即移除该状态标签，[SGwilson:25222-25224](../../../raw/current/scripts/stategraphs/SGwilson.lua#L25222-L25224)），饮用动画其余部分滴落节流不受影响；buff 在第 18 帧动作才 attach（[SGwilson:25225-25227](../../../raw/current/scripts/stategraphs/SGwilson.lua#L25225-L25227)），首滴 3.75 秒后到期，与饮用动画无重叠，已在身 buff 重饮也不影响滴落节奏 |
| 续杯 | 同槽同名药再次施加走 `debuff:Extend` → `buff_OnExtended`：重置 decay 计时器、重启 tick 任务，**非玩家目标**重播一次主 fx（玩家版不重播）；driptask 不动 | [Extend 分支](../../../raw/current/scripts/components/debuffable.lua#L104-L104)、[buff_OnExtended](../../../raw/current/scripts/prefabs/ghostly_elixirs.lua#L511-L532) |
| 停止路径 | decay 计时器到点、宿主 `death` 事件、换药 `pre_buff_fn`、摘下幽魂花冠（玩家侧 unequip 仅 Stop `elixir_buff` 槽，[hats.lua:5197](../../../raw/current/scripts/prefabs/hats.lua#L5195-L5200)；super 药的 `super_elixir_buff` 槽不随摘冠停止）任一 → `debuff:Stop` → `RemoveDebuff` | [计时器](../../../raw/current/scripts/prefabs/ghostly_elixirs.lua#L505-L509)、[death 监听](../../../raw/current/scripts/prefabs/ghostly_elixirs.lua#L495-L497)、[Stop](../../../raw/current/scripts/components/debuff.lua#L28-L32)、[OnDetach](../../../raw/current/scripts/components/debuff.lua#L44-L51) |
| 清理 | `buff_OnDetached` 取消 tick 与 drip 任务、执行 ONDETACH 后移除 buff 实体；已生成的 fx 实体不经 buff 管理，按工厂协议动画结束 `animover` 自删、不入存档 | [buff_OnDetached](../../../raw/current/scripts/prefabs/ghostly_elixirs.lua#L534-L554)、[工厂回收](../../../raw/current/scripts/prefabs/fx.lua#L98-L102) |

玩家饮用链：戴幽魂花冠（ghostflowerhat）的玩家获得 `elixir_drinker` 标签（[hats.lua:5189](../../../raw/current/scripts/prefabs/hats.lua#L5186-L5189)），对帽子施加灵药时 `Apply` 把目标转给帽子持有者（[ghostlyelixir.lua:9-14](../../../raw/current/scripts/components/ghostlyelixir.lua#L9-L14)），玩家进 `drinkelixir` 状态完成饮用（[SGwilson:25196-25208](../../../raw/current/scripts/stategraphs/SGwilson.lua#L25196-L25208)）；此后 buff 挂在玩家身上，`fx_player`／`dripfx_player` 名生效。玩家侧 buff 图标经 `_buffsymbol` 网络量同步到血条角标（[player_common.lua:2573](../../../raw/current/scripts/prefabs/player_common.lua#L2573-L2574)、[healthbadge](../../../raw/current/scripts/widgets/healthbadge.lua#L154-L170)）。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 工厂协议 | 36 名均经 `prefabs/fx.lua` 加载器逐项注册为 Prefab | 服务端建代理、一帧后各非专服模拟建本地表现；`fn` 收到本地实体同步执行 | [[reference/mechanisms/generic-fx-factory]]；[加载器](../../../raw/current/scripts/prefabs/fx.lua#L165-L172) |
| 生成者：灵药 buff 实体（`ghostlyelixir_<药效>_buff`） | `AddPotion` 为每瓶药注册 buff prefab；buff 在 attach／extend 时按 tunings 四字段选名生成主 fx 与 drip | 仅主模拟；`SpawnPrefab` 均在服务端 buff 回调内 | [AddPotion](../../../raw/current/scripts/prefabs/ghostly_elixirs.lua#L608-L637)；buff 实体与槽位主题 Wiki 待建（源码入口 [buff_fn](../../../raw/current/scripts/prefabs/ghostly_elixirs.lua#L573-L606)） |
| 组件：ghostlyelixir | 药水侧 `Apply` → `doapplyelixerfn`=`DoApplyElixir`；elixir_drinker 转给持有者 | 服务端；GIVE／APPLYELIXIR 双路由按标签过滤 | [[reference/components/ghostlyelixir]]；[APPLYELIXIR 候选](../../../raw/current/scripts/componentactions.lua#L1328-L1331) |
| 组件：ghostlyelixirable | 目标解析；花侧覆写转投幽灵、玩家与阿比盖尔默认本体 | 服务端 | [[reference/components/ghostlyelixirable]]；[玩家挂载](../../../raw/current/scripts/prefabs/player_common.lua#L2527-L2527) |
| 组件：debuff／debuffable | 槽位占用走 `Extend`、`Stop` → `RemoveDebuff` → `OnDetach`；`keepondespawn` 使 buff 不随宿主 despawn 清除 | 服务端 | [[reference/components/debuffable#关键接口]]、[[reference/components/debuff#关键接口]] |
| 数据：`potion_tunings`／TUNING | 四字段 `fx`／`dripfx`／`fx_player`／`dripfx_player` 承载本族全部选名；玩家侧时长／疗效另有 `*_PLAYER` 段 | 服务端读取；shadow／lunar 无 player 字段 | [tunings](../../../raw/current/scripts/prefabs/ghostly_elixirs.lua#L87-L346)、[常量](../../../raw/current/scripts/tuning.lua#L4834-L4855)、[玩家段](../../../raw/current/scripts/tuning.lua#L8614-L8634) |
| 关联 Prefab：`abigailforcefieldbuffed`／`abigailforcefieldretaliation`／`abigail_retaliation` | 护盾与反击相关实体，由 ONAPPLY 与受击回调生成；与本族 fx 是同药效的另一路 | 仅 shield／retaliation 药效；服务端 | [abigail.lua](../../../raw/current/scripts/prefabs/abigail.lua#L180-L214)、[受击反击](../../../raw/current/scripts/prefabs/ghostly_elixirs.lua#L6-L85)；[[reference/prefabs/abigail_retaliation|阿比盖尔反击特效]] |
| 关联 Prefab：`elixir_player_forcefield` | 玩家受击时附带的表现；减伤与触发由灵药 buff 和宿主回调负责 | 条目本身为通用表 FX | [受击生成](../../../raw/current/scripts/prefabs/ghostly_elixirs.lua#L6-L18)；[[reference/prefabs/wagdrone_rolling_collide_small_fx#关联机制导航]] |
| 关联 Prefab：`ghostflower`（哀悼荣耀） | 九种灵药的配方材料；其落地灵体表现与本族无生成关系 | 材料关系在配方层 | [[reference/prefabs/ghostflower_spirit1_fx|幽灵花灵体飘光]]；[recipes](../../../raw/current/scripts/recipes.lua#L151-L159) |
| 资源 | anim：`abigail_vial_fx.zip`、`abigail_buff_drip.zip`、`player_vial_fx.zip`、`player_elixir_buff_drip.zip` | 加载器按 `build` 声明 ANIM 资产；两个 drip bank 由药水 prefab 的资产表补充声明 | [加载器资产](../../../raw/current/scripts/prefabs/fx.lua#L5-L16)、[AddPotion 资产](../../../raw/current/scripts/prefabs/ghostly_elixirs.lua#L612-L617) |
| UI：pethealthbar／healthbadge | 阿比盖尔血条图标与玩家血条 buff 角标直接显示 buff prefab 名（非本族 fx 实体） | 服务端 set、客户端显示 | [OnDebuffAdded](../../../raw/current/scripts/prefabs/abigail.lua#L332-L340)、[pethealthbar](../../../raw/current/scripts/components/pethealthbar.lua#L220-L227)、[healthbadge](../../../raw/current/scripts/widgets/healthbadge.lua#L154-L170) |

## 典型接入

### 让自定义目标接受原版灵药（药效＋表现）

- 前提与选择：以下是 Prefab 文件的主模拟片段，目标为有效的非玩家实体，已有 `locomotor`、Transform 和提供 `HasStateTag` 的状态图，且 `elixir_buff` 槽为空；药效直接使用 locomotor，周期滴落检查状态图。直接 `AddDebuff` 不要求 `ghostlyelixirable`，后者用于药水动作路由。[药效](../../../raw/current/scripts/prefabs/ghostly_elixirs.lua#L192-L200)、[滴落](../../../raw/current/scripts/prefabs/ghostly_elixirs.lua#L467-L473)
- 生效与撤销：buff attach 后自动获得主 fx 父挂与周期滴落；decay 计时到点、宿主死亡或 `RemoveDebuff` 时停止并清理任务，fx 实体动画结束自删。
- 原版依据：[buff_OnAttached 生成写法](../../../raw/current/scripts/prefabs/ghostly_elixirs.lua#L499-L502)、[存活门禁](../../../raw/current/scripts/entityscript.lua#L2140-L2154)。

```lua
-- Prefab 服务端片段：inst 为具备上述组件和状态图的非玩家实体。
assert(TheWorld.ismastersim and inst:IsValid() and not inst:HasTag("player"))
assert(inst.Transform ~= nil and inst.sg ~= nil and inst.components.locomotor ~= nil)
assert(inst:GetDebuff("elixir_buff") == nil)
inst:AddDebuff("elixir_buff", "ghostlyelixir_speed_buff")
```

只借表现不要药效时，可另在服务端仿 `buff_OnAttached` 直接父挂；这是另一段示例，不要与上段叠加使用：

```lua
local fx = SpawnPrefab("ghostlyelixir_speed_fx")
fx.entity:SetParent(inst.entity)
```

- 调整与验证：直接 `AddDebuff` 不消耗药水，也不经 `buff_skill_modifier_fn`；首次构造使用普通 `DURATION`，不自动应用技能加成或玩家时长。已占同槽走 Extend 而非换实体，换药需先移除旧槽；返回 true 仅表示通过外层门禁，需另确认 buff 已生成。玩家也传 `ghostlyelixir_speed_buff`，buff 按目标 `player` 标签选择玩家表现；仅直接生成纯表现时才手选 `ghostlyelixir_player_speed_fx`。玩家 speed 分支还要求 `talker` 与 `locomotor`；其 `vigorbuff` 影响黏液地形和部分装备减速，不等于免疫一切减速。[应用与选名](../../../raw/current/scripts/prefabs/ghostly_elixirs.lua#L479-L502)、[玩家分支](../../../raw/current/scripts/prefabs/ghostly_elixirs.lua#L208-L218)、[黏液地形](../../../raw/current/scripts/components/locomotor.lua#L604-L622)、[装备](../../../raw/current/scripts/components/equippable.lua#L125-L135)

## 常见陷阱与限制

- **误区：fx 名即药效保证**：本族 36 名都是纯表现实体，无伤害／治疗／光环组件；同族注册名可在任何实体上父挂生成而不产生药效，药效判定全在 buff 的 tunings 回调。[条目结构](../../../raw/current/scripts/fx.lua#L1156-L1226)、[疗效位置](../../../raw/current/scripts/prefabs/ghostly_elixirs.lua#L87-L346)
- **边界：续杯重播的是旧药表现**：同槽施加不同药时 pre_buff_fn 先移除旧 buff（换新 fx）；但同槽同药 Extend 路径由既有 buff 执行，重播的 `fx` 是其自身 tunings 的名，且仅非玩家目标重播。[Extend 分支](../../../raw/current/scripts/components/debuffable.lua#L104-L104)、[重播条件](../../../raw/current/scripts/prefabs/ghostly_elixirs.lua#L526-L529)
- **边界：revive 的 drip 不能按单次时长判死**：基础时长 2 秒（玩家 0.3 秒）短于首滴延迟 3.75 秒，单次施加会在首滴前 detach 并取消 driptask；但同槽 `Extend` 只重置 decay、不重设 driptask（见“续杯”行），对非玩家目标反复续期可把总存活拖过首滴，届时目标不在 limbo 且非 busy 仍会生成。[时长](../../../raw/current/scripts/tuning.lua#L4853-L4853)、[首滴](../../../raw/current/scripts/prefabs/ghostly_elixirs.lua#L493-L493)、[Extend](../../../raw/current/scripts/components/debuffable.lua#L95-L105)、[OnExtended](../../../raw/current/scripts/prefabs/ghostly_elixirs.lua#L511-L532)
- **边界：revive 主 fx 复用 retaliation 音效**：条目 sound 与 retaliation 相同，源码 TODO 注明"unique sound at some point?"，非缺资源。[条目](../../../raw/current/scripts/fx.lua#L1204-L1210)
- **边界：玩家侧 shadow／lunar 4 名无生成点**：在 scripts 根全域（根级文件＋prefabs／components／stategraphs，含字面量与动态拼接检索）仅 fx.lua 注册处与 ghostly_elixirs.lua tunings 出现本族名；tunings 对 shadow／lunar 未提供 `fx_player`／`dripfx_player`，注册处注释标 "This is unused"。玩家实际用阿比盖尔版名。[检索与回退](../../../raw/current/scripts/prefabs/ghostly_elixirs.lua#L319-L345)
- **边界：阿比盖尔 OnDeath 不清灵药 buff**：`OnDeath` 移除的是名为 `ghostlyelixir` 的槽位（scripts 全域字面量检索未见该槽的 AddDebuff 调用点）；灵药 buff 的死亡清理走 buff 自身对宿主 `death` 事件的监听。[OnDeath](../../../raw/current/scripts/prefabs/abigail.lua#L227-L230)、[death 监听](../../../raw/current/scripts/prefabs/ghostly_elixirs.lua#L495-L497)
- **误区：主 fx 全程伴随 buff**：主 fx 只在 attach（与非玩家 Extend）时生成一次，其后是否持续显示取决于动画数据的循环性（scripts 不可核验）；buff 存续期内没有按周期重播主 fx 的代码路径，周期出现的是地面滴落。[生成点](../../../raw/current/scripts/prefabs/ghostly_elixirs.lua#L499-L502)、[driptask](../../../raw/current/scripts/prefabs/ghostly_elixirs.lua#L493-L493)

## 验证记录

- 源码审阅：2026-11-01，Steam buildid 26335394。fx.lua 1140-1480 共 38 条目逐项核对 bank／build／anim／sound／fn（阿比盖尔 18＋玩家 18＋幽灵花灵体 2，后两者另见 [[reference/prefabs/ghostflower_spirit1_fx|幽灵花灵体飘光]]）；生成链读 ghostly_elixirs.lua 全文（tunings、buff_fn、attach／drip／extend／detach）并追到 entityscript／debuffable／debuff 的槽位语义；玩家饮用链读 hats、SGwilson、ghostlyelixir 组件与 componentactions；玩家侧时长常量读 tuning.lua 两段；po 定向核验九瓶药水官方名与 fx 无显示名键。"无生成点"结论的检索范围：`rg "ghostlyelixir"` 全 scripts 根命中 17 文件逐一复查，fx 名字面量与 `"ghostlyelixir_"` 动态拼接检索均无 ghostly_elixirs.lua 之外的命中。未覆盖：动画 zip 内容与循环性、`elixir_container` 容器交互细节、技能树节点全链、多客户端表现。
- 游戏运行：未运行；attach／滴落节奏与音效均为源码审阅结论。
- 结构检查：2026-11-01 validate_page.py 与 validate_source_links.py --strict 通过（见本页文件）；Lua 示例经 parser_probe --syntax-only 解析通过。

返回 [[reference/prefabs/fx-status-index|增益、受击、护盾与状态反馈]]、[[reference/prefabs/index|Prefab 索引]]。
