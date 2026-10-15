---
title: "Boss 与特殊护甲族（armor_bramble、armordragonfly 等 4 注册）"
created: "2026-11-07"
updated: "2026-11-07"
type: prefab
summary: "四个独立文件注册的 BODY 槽特殊护甲（荆棘外壳、鳞甲、W.A.R.B.I.S.盔甲、镀金骑士胸甲）：注册名与文件名差异、反伤尖刺／火免反燃／追踪提速／套装标签四套专属机制、配方与 Boss 掉落获取链、官方名与消费端标签，供模组参照原版做 Boss 级与特殊机制护甲。"
aliases: ["armor_bramble", "荆棘外壳", "荆棘甲", "armordragonfly", "鳞甲", "armorwagpunk", "W.A.R.B.I.S.盔甲", "armor_yoth_knight", "镀金骑士胸甲", "Gilded Knight's Chest Plate", "Boss 护甲", "特殊护甲族", "身体护甲"]
tags: [prefab, item, equipment]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/armor_bramble.lua", "prefabs/armor_dragonfly.lua", "prefabs/armor_wagpunk.lua", "prefabs/armor_yoth_knight.lua", "prefabs/bramblefx.lua", "prefabs/knight.lua", "prefabs/dragonfly.lua", "prefabs/beequeen.lua", "prefabs/daywalker2.lua", "prefabs/wagstaff_machinery.lua", "prefabs/armor_lunarplant.lua", "prefabs/flower.lua", "prefabs/cactus.lua", "prefabs/hedgehound_bush.lua", "prefabs/skilltree_wormwood.lua", "prefabs/wormwood.lua", "fx.lua", "components/armor.lua", "components/setbonus.lua", "components/health.lua", "components/targettracker.lua", "components/playerlightningtarget.lua", "components/yoth_knightmanager.lua", "componentutil.lua", "standardcomponents.lua", "stategraphs/SGwagdrone_rolling.lua", "stategraphs/SGwilson.lua", "tuning.lua", "recipes.lua", "constants.lua", "strings.lua", "skins_defs_data.lua", "languages/chinese_s.po", "mods.lua", "widgets/itemtile.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# Boss 与特殊护甲族（armor_bramble、armordragonfly 等 4 注册）

## 快速判断

- 职责与适用：四个独立 prefab 文件的 BODY 槽护甲：荆棘外壳（受击反伤尖刺）、鳞甲（火免＋点燃近战攻击者）、W.A.R.B.I.S.盔甲（追踪提速＋破损修复）、镀金骑士胸甲（YOTH 套装挂 `luckysource` 标签）。帽侧同族见 [[reference/prefabs/hats-armor|防护与战斗帽族]]。
- 优先使用：查耐久／吸收／专属触发 → [注册与构造](#注册与构造)与[机制辨析](#机制辨析)；查获取链与标签消费 → 对应表；复刻效果从[典型接入](#典型接入)进入。
- 重要副作用：armor 耐久按 `Inventory:ApplyDamage` 分摊到该件的吸收量扣减（不是原始来伤，见 [[reference/components/armor|armor 组件]]）；荆棘外壳反伤以穿戴者为攻击者结算；鳞甲把持有者火伤倍率置 0；W.A.R.B.I.S. 卸甲即还原移速；镀金骑士胸甲单穿不触发套装。
- 修改边界：只覆盖甲本体与直接触发链；尖刺 FX 归 [[reference/prefabs/bramblefx_armor|bramblefx_armor 页]]，帽侧运值与阶段归 [[reference/prefabs/hats-armor|帽族页]]，传统身体甲归 [[reference/prefabs/armor-classic|传统身体护甲族]]。

## 注册与构造

注册名与文件名两件不一致：`armor_dragonfly.lua` 注册 `armordragonfly`（无下划线，[L98](../../../raw/current/scripts/prefabs/armor_dragonfly.lua#L98)）、`armor_wagpunk.lua` 注册 `armorwagpunk` 与附属 `armorwagpunk_fx`（[L540-L541](../../../raw/current/scripts/prefabs/armor_wagpunk.lua#L540-L541)）；另两件一致。四件均为主模拟工厂（SetPristine 后 mastersim 分界）。耐久/吸收为推导值（配置式 × `multiplayer_armor_durability_modifier=0.7`，[tuning.lua#L33-L38](../../../raw/current/scripts/tuning.lua#L33-L38)，wilson_health=150）。

| 注册名（官方名） | armor：耐久×吸收 | 附加组件与标签 | 获取 | 证据 |
|---|---|---|---|---|
| `armor_bramble`（荆棘外壳） | 525×0.8＋弱点 beaver 17 | fuel LARGE_FUEL＋小可燃＋可蔓延；`bramble_resistant` | 沃姆伍德配方：活木头2＋针刺4（`builder_tag="plantkin"`） | [fn](../../../raw/current/scripts/prefabs/armor_bramble.lua#L84-L138)、[调值](../../../raw/current/scripts/tuning.lua#L4562-L4566)、[配方](../../../raw/current/scripts/recipes.lua#L339) |
| `armordragonfly`（鳞甲） | 945×0.7 | dapperness `DAPPERNESS_MED`≈26.7 理智/天；无标签 | 配方：鳞片1＋木甲1＋猪皮3（SCIENCE_TWO） | [fn](../../../raw/current/scripts/prefabs/armor_dragonfly.lua#L58-L96)、[调值](../../../raw/current/scripts/tuning.lua#L1843-L1845)、[配方](../../../raw/current/scripts/recipes.lua#L498) |
| `armorwagpunk`（W.A.R.B.I.S.盔甲） | 730×0.85＋planar 5 | `targettracker` 提速；`equippable.insulated`；`metal`/`hardarmor`/`show_broken_ui`；破损可修（废料重建） | 配方：废料8＋晶体管3＋启迪碎片1（TECH.LOST，蓝图学习） | [fn](../../../raw/current/scripts/prefabs/armor_wagpunk.lua#L337-L400)、[破损/修复](../../../raw/current/scripts/prefabs/armor_wagpunk.lua#L317-L335)、[配方](../../../raw/current/scripts/recipes.lua#L862) |
| `armor_yoth_knight`（镀金骑士胸甲） | 840×0.9 | `setbonus`（YOTH_KNIGHT）启用加 `luckysource`；`metal`/`hardarmor`；无配方、拆解金6＋蛛丝4 | 最后一骑 `knight_yoth` 100% 掉落 | [fn](../../../raw/current/scripts/prefabs/armor_yoth_knight.lua#L40-L86)、[调值](../../../raw/current/scripts/tuning.lua#L9282-L9283)、[掉落](../../../raw/current/scripts/prefabs/knight.lua#L262-L288) |

皮肤：`armor_bramble_ancient` 受限 build 注册在案（[skins_defs_data.lua#L63](../../../raw/current/scripts/skins_defs_data.lua#L63)），其余三件未见皮肤注册；换装协议四件同帽族（`GetSkinBuild` → `OverrideItemSkinSymbol`＋`equipskinneditem`，[armor_bramble.lua#L51-L58](../../../raw/current/scripts/prefabs/armor_bramble.lua#L51-L58)）。

## 机制辨析

### 荆棘外壳：反伤尖刺（无穿戴自伤）

- 两条触发路径（[DoThorns](../../../raw/current/scripts/prefabs/armor_bramble.lua#L15-L28)、[OnAttackOther](../../../raw/current/scripts/prefabs/armor_bramble.lua#L36-L49)）：①**通用受击反伤**——任何穿戴者的 `attacked`/`blocked`＋`ShouldProcOnAttackedOrBlocked`（排除重定向与自身来源）＋0.3 秒冷却就触发，与角色无关；②**沃姆伍德专属主动连击**——只有持有者带 `plantkin`（[wormwood.lua#L754](../../../raw/current/scripts/prefabs/wormwood.lua#L754)）时才额外监听 `onattackother`，回调内再查技能 `wormwood_armor_bramble` 与冷却，通过则 `_hitcount + 1`，累计到 3（[tuning.lua#L7848](../../../raw/current/scripts/tuning.lua#L7848)）释放尖刺并清零；**只有门禁不通过（冷却中／技能未激活）才清零，未到 3 时计数保留**。另注意 `onattackother` 在伤害结算之前发出，这 3 次是"对目标发起的攻击次数"，不是"已造成 3 次伤害"。
- `DoThorns` 生成 `bramblefx_armor` 尖刺环（≈22.7 伤害，攻击者记穿戴者名下）＋仙人掌音效；结算与免疫归 [[reference/prefabs/bramblefx_armor|bramblefx_armor 页]]。
- 边界：尖刺只打周围目标，不伤穿戴者（owner 在 FX 的 `ignore` 表、甲自身 `bramble_resistant` 被尖刺免疫排除）——"穿戴自伤"在源码中不存在。技能节点连接自 `wormwood_blooming_trapbramble`（[skilltree_wormwood.lua#L360-L380](../../../raw/current/scripts/prefabs/skilltree_wormwood.lua#L360-L380)）。

### 鳞甲：火免与反燃

- 火免：装备即给持有者 `health.externalfiredamagemultipliers` 挂 modifier `1 - ARMORDRAGONFLY_FIRE_RESIST = 0`（[L36-L38](../../../raw/current/scripts/prefabs/armor_dragonfly.lua#L36-L38)），卸甲移除；该表乘进 `Health:GetFireDamageScale`（[health.lua#L190-L192](../../../raw/current/scripts/components/health.lua#L190-L192)），直接火伤与燃烧扣血都走此倍率（[burnable.lua#L353](../../../raw/current/scripts/components/burnable.lua#L353)）——火伤归零，但不阻止进入燃烧状态本身。
- 反燃：`attacked`/`blocked` → `ShouldProcOnAttackedOrBlocked` → 攻击者 `burnable:Ignite(nil, nil, owner)`；排除已死攻击者、远程武器（`IsRangedWeapon`）、无 burnable、重定向、`thorny` 标签（[OnBlocked](../../../raw/current/scripts/prefabs/armor_dragonfly.lua#L6-L19)）。

### W.A.R.B.I.S.盔甲：追踪提速与破损修复

- 提速链：持有者 `onattackother` → 目标 2 秒待定（[OnAttack](../../../raw/current/scripts/prefabs/armor_wagpunk.lua#L128-L158)）→ `SetNewTarget` 锁定 → 移速 1.10（STAGE0）＋蒸汽 FX；追踪时间逐帧累计（[TimeCheck](../../../raw/current/scripts/prefabs/armor_wagpunk.lua#L161-L210)），过 10/16/20 秒逐档升 1.15/1.20/1.30（[tuning.lua#L8025-L8032](../../../raw/current/scripts/tuning.lua#L8025-L8032)）。目标死亡或超范围（`WAGPUNK_MAXRANGE=16`＋WX78 雷达模块每个 +6，[componentutil.lua#L2367-L2380](../../../raw/current/scripts/componentutil.lua#L2367-L2380)）→ `StopTracking` → [ResetBuff](../../../raw/current/scripts/prefabs/armor_wagpunk.lua#L62-L82) 还原移速 1、关环境音、FX 归 1 级。
- 与同装帽互认：装备时 `CloneTargetFrom(hat, WAGPUNK_SYNC_TIME=3)` 克隆帽目标；`SetNewTarget` 反哺无目标的帽（[L111-L115](../../../raw/current/scripts/prefabs/armor_wagpunk.lua#L111-L115)）；恢复时取帽/甲追踪时间较大值。
- 破损与修复：`MakeForgeRepairable`（材料 wagpunk_bits，[standardcomponents.lua#L1851-L1884](../../../raw/current/scripts/standardcomponents.lua#L1851-L1884)）把耐久尽销毁替换为 [OnBroken](../../../raw/current/scripts/prefabs/armor_wagpunk.lua#L317-L325)：卸回背包、移除 `equippable`、`broken` 标签；[OnRepaired](../../../raw/current/scripts/prefabs/armor_wagpunk.lua#L327-L335) 重建组件。
- `equippable.insulated=true`：雷击走 `inventory:IsInsulated` 免伤分支（[playerlightningtarget.lua#L3-L9](../../../raw/current/scripts/components/playerlightningtarget.lua#L3-L9)）。

### 镀金骑士胸甲：YOTH 套装（护甲侧）

- `setbonus:SetSetName(EQUIPMENTSETNAMES.YOTH_KNIGHT)`（[L78-L81](../../../raw/current/scripts/prefabs/armor_yoth_knight.lua#L78-L81)）；启用条件是 HEAD+BODY 两件同名 setbonus（[[reference/components/setbonus|setbonus 组件]]，[setbonus.lua#L52-L61](../../../raw/current/scripts/components/setbonus.lua#L52-L61)）——与 `yoth_knighthat` 同名互认。
- 护甲侧回调只加/删 `luckysource` 标签（[L32-L37](../../../raw/current/scripts/prefabs/armor_yoth_knight.lua#L32-L37)）；运值本体（0.2，事件期 1.0）由帽侧 `luckuser` 提供，见 [[reference/prefabs/hats-armor|帽族页]] yoth_knighthat 行。
- 掉落链：`yoth_knightmanager` 经帽的 `petleash` 生成 4 匹骑士（CONQUEST/WAR/FAMINE/DEATH，[yoth_knightmanager.lua#L72-L107](../../../raw/current/scripts/components/yoth_knightmanager.lua#L72-L107)）；组内最后一骑死亡才掉两件装备（各 100%，其余标记 `fled` 防重复）。

## 获取与掉落链

| 甲 | 获取 | 关键中间物 | 证据 |
|---|---|---|---|
| 荆棘外壳 | 沃姆伍德专属配方（plantkin）；**不是 Boss 掉落**——蜂王 loot 是 hivehat＋stinger | 活木头（沃姆伍德 20 血换取）、针刺（蜂王 1.00） | [beequeen loot](../../../raw/current/scripts/prefabs/beequeen.lua#L39-L40) |
| 鳞甲 | 常规配方（SCIENCE_TWO） | 鳞片：龙蝇 loot 1.00＋断尾补投 | [dragonfly loot](../../../raw/current/scripts/prefabs/dragonfly.lua#L40) |
| W.A.R.B.I.S.盔甲 | 配方（TECH.LOST），蓝图先行 | 蓝图：拾荒疯猪／被丢弃的垃圾在月裂隙开启期优先投放给未学者；废料：machinery 1.00+0.75、疯猪 4×1.00+0.5 | [疯猪 lootsetfn](../../../raw/current/scripts/prefabs/daywalker2.lua#L1019-L1057) |
| 镀金骑士胸甲 | 最后一骑 knight_yoth 100%（YOTH 事件） | knight_yoth 由 yoth_knightmanager 生成 | [knight.lua#L277-L279](../../../raw/current/scripts/prefabs/knight.lua#L277-L279) |

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| `组件：armor` | `InitCondition(耐久, 吸收)`；`AddWeakness`（bramble beaver）；`TakeDamage` 只扣分摊到该件的量、乘 `conditionlossmultipliers`、推 `armordamaged` | 四件全主模拟挂载 | [[reference/components/armor]]；[armor.lua#L126-L164](../../../raw/current/scripts/components/armor.lua#L126-L164) |
| `组件：equippable` | BODY 槽；`dapperness`（鳞甲）；`insulated`、`walkspeedmult` 动态改（W.A.R.B.I.S.） | 破损态移除组件、修复重建 | [[reference/components/equippable]] |
| `组件：planardefense` | `SetBaseDefense(5)` 固定位面减伤（[L387-L388](../../../raw/current/scripts/prefabs/armor_wagpunk.lua#L387-L388)） | 仅 armorwagpunk | [[reference/components/planardefense]] |
| `组件：targettracker` | 追踪目标、时间累计回调、KeepRange 检查、克隆与恢复回调（[L378-L382](../../../raw/current/scripts/prefabs/armor_wagpunk.lua#L378-L382)） | 仅 armorwagpunk；帽同构 | [[reference/components/targettracker]] |
| `组件：setbonus` | 同名互认启用/禁用回调 | 仅 armor_yoth_knight | [[reference/components/setbonus]] |
| `事件/数据：proc 协议` | `ShouldProcOnAttackedOrBlocked` 排除重定向与自身来源；`IsRangedWeapon` 供反燃排除 | bramble 与 dragonfly 共用 | [componentutil.lua#L103-L122](../../../raw/current/scripts/componentutil.lua#L103-L122) |
| `关联 Prefab：bramblefx_armor` | 荆棘外壳反伤尖刺环（≈22.7） | `SetFXOwner(穿戴者)`；技能扩展路径同 FX | [[reference/prefabs/bramblefx_armor]]、[bramblefx.lua#L123](../../../raw/current/scripts/prefabs/bramblefx.lua#L123) |
| `关联 Prefab：armorwagpunk_fx、wagpunksteam_armor_up/down` | 9 段贴身跟随 FX（`net_tinybyte` level 1-5 → bank `armor_wagpunk_01/05`）；锁定/升档与掉档蒸汽 | FX 装备生成随主、卸甲 Remove；帽侧蒸汽见 [[reference/prefabs/wagpunksteam_hat_up|WAGpunk 装备蒸汽]] | [fxfn](../../../raw/current/scripts/prefabs/armor_wagpunk.lua#L473-L517)、[fx.lua#L3211](../../../raw/current/scripts/fx.lua#L3211)、[L3222](../../../raw/current/scripts/fx.lua#L3222) |
| `关联 Prefab：掉落源实体` | knight_yoth（镀金骑士）、dragonfly（龙蝇）、beequeen（蜂王）、daywalker2（拾荒疯猪）、wagstaff_machinery（被丢弃的垃圾） | 各 loot 段；knight_yoth 帽侧互链 [[reference/prefabs/hats-armor|帽族页]] | 见[获取与掉落链](#获取与掉落链) |
| `API／数据：Wormwood 技能` | `wormwood_armor_bramble`（"荆棘外壳专家"）解锁 3 连击触发 | 仅 plantkin 持有者 | [[reference/components/skilltreeupdater]]、[skilltree_wormwood.lua#L373-L380](../../../raw/current/scripts/prefabs/skilltree_wormwood.lua#L373-L380) |
| `资源：动画` | armor_bramble.zip／torso_dragonfly.zip／armor_wagpunk_01-05.zip／armor_yoth_knight.zip；换装符 `swap_body` | 各文件 assets 段 | [armor_bramble.lua#L1-L4](../../../raw/current/scripts/prefabs/armor_bramble.lua#L1-L4) |

## 消费端标签

| 标签 | 写入者（本页） | 消费端与效果 | 证据 |
|---|---|---|---|
| `bramble_resistant` | armor_bramble（荆棘茄甲亦带，[armor_lunarplant.lua#L292](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L292)） | 采摘花／仙人掌／冬青丛免刺伤；bramblefx 尖刺对装备栏含该标签者完全免疫 | [flower.lua#L62](../../../raw/current/scripts/prefabs/flower.lua#L62)、[cactus.lua#L24](../../../raw/current/scripts/prefabs/cactus.lua#L24)、[bramblefx.lua#L21-L22](../../../raw/current/scripts/prefabs/bramblefx.lua#L21-L22) |
| `hardarmor` | armorwagpunk、armor_yoth_knight | Wagdrone 滚动冲撞按 `EquipHasTag` 走减免分支 | [SGwagdrone_rolling.lua#L324](../../../raw/current/scripts/stategraphs/SGwagdrone_rolling.lua#L324) |
| `luckysource` | armor_yoth_knight（套装启用时） | itemtile 幸运特效判定；实际运值走帽侧 luckuser | [itemtile.lua#L923-L925](../../../raw/current/scripts/widgets/itemtile.lua#L923-L925) |
| `metal`／`broken`／`show_broken_ui` | armorwagpunk（metal 与 knight 胸甲并写） | metal 为命中音效（源注释）；broken/show_broken_ui 供破损表现与名称覆盖 | [armor_wagpunk.lua#L350-L352](../../../raw/current/scripts/prefabs/armor_wagpunk.lua#L350-L352) |

## 典型接入

### 给自定义护甲复刻"火免＋点燃近战攻击者"（鳞甲模式）

- 前提与选择：modmain 注册自定义甲并装 armor/equippable；火免走 `health.externalfiredamagemultipliers`（SourceModifierList，以甲实例为 key），反燃监听 `attacked`/`blocked` 并复用 `componentutil` 的 helper，不手写受击判定。注意 `componentutil` 把这些函数定义成**全局**（文件不 return 模块表），mod 里必须写 `GLOBAL.ShouldProcOnAttackedOrBlocked` / `GLOBAL.IsRangedWeapon`，`require` 到的是空值。
- 生效与撤销：装备挂 modifier 与监听、卸下成对移除——漏移除会永久锁 0 火伤倍率（同 wagpunk 移速陷阱）。
- 原版依据：[armor_dragonfly.lua#L6-L56](../../../raw/current/scripts/prefabs/armor_dragonfly.lua#L6-L56)。

```lua
-- modmain.lua：鳞甲式火免＋反燃（数值与 build 名可替换）
-- mod 环境无 __index=_G（mods.lua#L258-L317）：TheWorld/SpawnPrefab 等
-- 游戏全局用 GLOBAL. 前缀；TUNING 直用。
local FIRE_RESIST = 1 -- 0=无减免，1=全免（原版 ARMORDRAGONFLY_FIRE_RESIST）

local function onequip(inst, owner)
    owner.AnimState:OverrideSymbol("swap_body", "myscalemail", "swap_body")
    if owner.components.health ~= nil then
        owner.components.health.externalfiredamagemultipliers:SetModifier(inst, 1 - FIRE_RESIST)
    end
    inst._onblocked = function(owner, data)
        if data ~= nil and data.attacker ~= nil
            and GLOBAL.ShouldProcOnAttackedOrBlocked(inst, owner, data)
            and not GLOBAL.IsRangedWeapon(data.weapon)          -- 原版排除远程武器
            and not data.attacker:HasTag("thorny")              -- 原版排除带刺目标
            and not (data.attacker.components.health ~= nil and data.attacker.components.health:IsDead())
            and data.attacker.components.burnable ~= nil then
            data.attacker.components.burnable:Ignite(nil, nil, owner)
        end
    end
    inst:ListenForEvent("attacked", inst._onblocked, owner)
    inst:ListenForEvent("blocked", inst._onblocked, owner)
end

local function onunequip(inst, owner)
    owner.AnimState:ClearOverrideSymbol("swap_body")
    inst:RemoveEventCallback("attacked", inst._onblocked, owner)
    inst:RemoveEventCallback("blocked", inst._onblocked, owner)
    inst._onblocked = nil
    if owner.components.health ~= nil then
        owner.components.health.externalfiredamagemultipliers:RemoveModifier(inst)
    end
end

AddPrefabPostInit("myscalemail", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    inst.components.equippable:SetOnEquip(onequip)
    inst.components.equippable:SetOnUnequip(onunequip)
end)
```

- 调整与验证：替换点为 build 名、`FIRE_RESIST` 与 Ignite 条件；`ShouldProcOnAttackedOrBlocked` 需要 `attacked`/`blocked` 事件的 data。要与原版逐条对齐，`thorny` 与"已死亡攻击者"两个门禁不能省——漏掉会点燃原版不点燃的两类目标（[原版反燃](../../../raw/current/scripts/prefabs/armor_dragonfly.lua#L6-L19)）。荆棘反伤模式见 [[reference/prefabs/bramblefx_armor|bramblefx_armor 页]]典型接入。示例只做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **误区：注册名可从文件名猜。** `armor_dragonfly.lua` → `armordragonfly`、`armor_wagpunk.lua` → `armorwagpunk`（无下划线）；`SpawnPrefab("armor_dragonfly")` 不会命中任何注册。
- **armor 耐久扣的是分摊量，proc／反伤都不豁免。** `Inventory:ApplyDamage` 把总吸收量按各件吸收率占比分给每件（另加弱点与特殊伤害），`Armor:TakeDamage` 再乘该件损耗倍率才扣 `condition`；反伤期间甲耐久照常按分到的量消耗（[分摊](../../../raw/current/scripts/components/inventory.lua#L408-L500)、[armor.lua#L156-L164](../../../raw/current/scripts/components/armor.lua#L156-L164)）。
- **误区：荆棘外壳是 Boss 掉落或"穿了会自伤"。** 配方 `builder_tag="plantkin"` 仅沃姆伍德可造（蜂王 loot 是 hivehat＋stinger）；尖刺只结算周围目标，无自伤路径（[beequeen.lua#L39-L40](../../../raw/current/scripts/prefabs/beequeen.lua#L39-L40)）。
- **鳞甲吸收是 0.7 不是 0.9；火免只覆盖"走火伤倍率"的结算。** `externalfiredamagemultipliers` 只在 `Health:GetFireDamageScale` 被读取的路径生效（直伤与 burnable 的燃烧扣血），不阻止被点燃状态本身，也不等于对所有"看着像火"的伤害免疫（[火伤入口](../../../raw/current/scripts/components/health.lua#L190-L229)）。
- **W.A.R.B.I.S. 卸甲／失靶即还原。** 卸甲 `walkspeedmult=1`＋StopTracking＋关环境音；目标出 16 格即 ResetBuff——只复制提速代码漏掉还原会永久锁倍率（[OnUnequip](../../../raw/current/scripts/prefabs/armor_wagpunk.lua#L253-L289)）。
- **破损≠销毁是 armorwagpunk 独有协议。** `MakeForgeRepairable` 把耐久尽回调换成"卸下＋移除 equippable＋broken 标签"，靠 OnRepaired 重建；普通甲照抄会得到穿不上的残甲。
- **YOTH 套装单件不生效、运值在帽侧。** setbonus 判 HEAD+BODY 同名；胸甲侧回调只挂 `luckysource` 标签（itemtile 特效），加运由 `yoth_knighthat` 的 luckuser 完成（[[reference/prefabs/hats-armor|帽族页]]）。
- **边界：`superheavyarmor`／减速与这四件无关。** `superheavyarmor` 只有 armor_rocky 写入；本页四件不减速，W.A.R.B.I.S. 反而提速（[armor_rocky.lua#L56](../../../raw/current/scripts/prefabs/armor_rocky.lua#L56)）。
- **modmain 全局前缀。** mod 环境（[mods.lua#L258-L317](../../../raw/current/scripts/mods.lua#L258-L317)）无 `__index=_G`：`TheWorld`／`SpawnPrefab` 一律 `GLOBAL.` 前缀，`TUNING` 直用。

## 验证记录

- 源码审阅：2026-11-07，steam-buildid:26335394。完整实读四件甲文件（141／98／541／88 行）；定向实读 tuning.lua 调值段（L33-38、L2215、L1843-1845、L4562-4566、L4653、L7498、L7848、L8021-8045、L9282-9283）、recipes.lua 五条（L338-339、L352、L498、L862、L1742）、componentutil.lua（L103-122、L2367-2380）、bramblefx.lua、knight.lua、yoth_knightmanager.lua、dragonfly／beequeen／daywalker2／wagstaff_machinery loot 段、armor.lua（L126-164）、setbonus.lua（L40-75）、health.lua（L81、L190-196）、playerlightningtarget.lua、targettracker.lua（L30-160）、fx.lua steam 注册行（L3211/L3222）、skilltree_wormwood.lua、constants.lua、wormwood.lua、standardcomponents.lua、SGwagdrone_rolling.lua、armor_lunarplant.lua、skins_defs_data.lua、mods.lua、itemtile.lua、SGwilson.lua。
- 数值核验：525/0.8、945/0.7、730/0.85、840/0.9 按配置式 ×0.7 推导并核对原式；`ARMORBRAMBLE_DMG`=34/1.5≈22.7；`BEAVER_WOOD_DAMAGE`=17；`DAPPERNESS_MED`=100/(day_time×6)（注释 26.7/天）；STAGE0-3=1.10/1.15/1.20/1.30，STAGE1/2/3=10/16/20 秒；`WAGPUNK_MAXRANGE`=16、`RADAR_WAGPUNKRANGE`=6、`WAGPUNK_SYNC_TIME`=3。
- 名称核验：`languages/chinese_s.po`（Python UTF-8 读 msgctxt）——`ARMOR_BRAMBLE`="Bramble Husk"=荆棘外壳（[L338829-L338832](../../../raw/current/scripts/languages/chinese_s.po#L338829-L338832)）、`ARMORDRAGONFLY`="Scalemail"=鳞甲（L338779-L338782）、`ARMORWAGPUNK`="W.A.R.B.I.S. Armor"=W.A.R.B.I.S.盔甲（L338819-L338822）、`ARMOR_YOTH_KNIGHT`="Gilded Knight's Chest Plate"=镀金骑士胸甲（L338864-L338867）；辅助核验 `DRAGON_SCALES`／`WAGPUNK_BITS`／`KNIGHT_YOTH`／`DAYWALKER2`／`STINGER`／`LIVINGLOG`／`ALTERGUARDIANHATSHARD` 各官方中文名。注意 `ARMORBRAMBLE` 无 `STRINGS.NAMES` 键（仅 `SCRAPBOOK.SPECIALINFO`），显示名键是带下划线的 `ARMOR_BRAMBLE`（[strings.lua#L3205](../../../raw/current/scripts/strings.lua#L3205)）。
- 消费端反查：`bramble_resistant`（flower/cactus/hedgehound_bush＋bramblefx 免疫）、`hardarmor`（SGwagdrone_rolling L324）、`luckysource`（itemtile L923-925）、`superheavyarmor`（仅 armor_rocky 写入）；`ARMORBRAMBLE_PLANAR_UPGRADE=5` 在 scripts 域无消费者（升级尖刺 5 点平面走 `ARMORBRAMBLE_DMG_PLANAR_UPGRADE` → `bramblefx_armor_upgrade`，宿主是荆棘茄甲而非本页荆棘外壳）。获取链 loot 段均实读，证据见上文各表。
- 未核验／待办：armorwagpunk ambient 音效参数曲线（仅记录未听测）；`scrapbook_damage`/`scrapbook_specialinfo` 收藏册展示；`armor_bramble_ancient` 皮肤取得途径与可见效果；`metal` 标签引擎层命中音效实现；knight_yoth 战斗 AI 深度（帽族页已有同项待办）。
- 结构检查：2026-11-07 第六轮修订后 `validate_page.py`（含 `--check-indexes`）通过（179 行／26204 字节，0 issue）；`validate_source_links.py --strict` 通过（72 条引用、33 个源文件，0 error 0 warning）；Lua 示例经 `parser_probe --syntax-only` 解析通过。
- 游戏运行：未运行；`validation` 保持 `source-reviewed`。

## 相关链接

- [[reference/prefabs/hats-armor|防护与战斗帽族]]（HEAD 槽同族：yoth_knighthat 运值、wagpunkhat 阶段）
- [[reference/prefabs/armor-classic|传统身体护甲族]]（其余 10 件独立文件 BODY 槽装备）
- [[reference/prefabs/bramblefx_armor|荆棘尖刺反伤与陷阱穿刺]]（bramblefx_armor 结算与免疫链的主要页）
- [[reference/components/armor|armor 组件]]、[[reference/components/setbonus|setbonus 组件]]、[[reference/components/targettracker|targettracker 组件]]（受击协议、套装互认、追踪提速）

返回 [[reference/prefabs/wearables-index|穿戴防具与饰品]]、[[reference/prefabs/index|Prefab 索引]]。
