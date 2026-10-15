---
title: "角色专属装备补遗（伯尼／投泥带／电刑机）"
created: "2026-11-07"
updated: "2026-11-07"
type: prefab
size_class: standard
summary: "三组角色专属装备选例（不是全部角色专属装备）：Willow 的伯尼（inactive 才可装备、bernieowner 限装、fuel↔health 与腐坏）、Wurt 的影/月投泥带（rechargeable 全背包同步放电、地形改造与月态鱼人突变）、WX-78 的电刑机遥控（技能门禁、100 用、无人机与充电）。全部依赖角色标签或技能树，不是通用装备。"
aliases: ["bernie_inactive", "bernie_active", "bernie_big", "wurt_swampitem_shadow", "wurt_swampitem_lunar", "wx78_drone_zap", "wx78_drone_zap_remote", "伯尼", "伯尼！", "投泥带", "绝望投泥带", "光芒投泥带", "电刑机", "电刑机控制器", "角色专属装备补遗", "角色装备选例"]
tags: [prefab, item, equipment]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/bernie_inactive.lua", "prefabs/bernie_active.lua", "prefabs/bernie_common.lua", "prefabs/willow.lua", "prefabs/wurt_terraform_item.lua", "prefabs/wx78_drone_zap.lua", "prefabs/skilltree_wx78.lua", "prefabs/wx78_possessedbody.lua", "brains/berniebrain.lua", "components/equippable.lua", "components/fueled.lua", "components/rechargeable.lua", "components/spellcaster.lua", "components/batteryuser.lua", "components/useableequippeditem.lua", "components/inventoryitem.lua", "recipes.lua", "tuning.lua", "constants.lua", "skins_defs_data.lua", "strings.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 角色专属装备补遗（伯尼／投泥带／电刑机）

## 快速判断

- 职责与适用：三组都靠角色标签/技能树授权，不是通用装备——Willow 的 `bernie_inactive`（活体玩偶的装备态）、Wurt 的 `wurt_swampitem_shadow/lunar`（地形改造投泥带）、WX-78 的 `wx78_drone_zap_remote`（电刑机遥控）。适合查 `restrictedtag`、技能门禁、燃料/充能与召唤物归属。
- 优先使用：限装判定查 [[reference/components/equippable]]；燃料与充能查 [[reference/components/fueled]]、[[reference/components/rechargeable]]；召唤物/无人机主体分别看 `bernie_active`、`wx78_drone_zap` 小节。
- 重要副作用：伯尼空燃料会 dead 态、掉地 3 天腐坏返料；投泥带施法会同步放电持有者物品栏内其余同类并压平新拾取件；遥控未激活技能时右键不可用，部署只检查可飞行点并生成/复用无人机，真正扣次发生在无人机开火（每次 2 点，技能 2 降为 1 点），100 是耐久上限。
- 修改边界：`bernie_big`／`lavaarena_bernie`、`wurt_swamp_terraformer` 地形主体、`SGwx78_drone_zap` 的索敌/视觉、鱼人突变后的完整战斗链均不展开。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `bernie_inactive` | fn：公共端动画＋小地图图标 → 主模拟 inspectable(BROKEN)、inventoryitem(SetSinks)、equippable(`restrictedtag="bernieowner"`)、insulator、fueled(USAGE)、timer | 装备态伯尼：有油穿戴回理智＋保温、随攻击磨损；空燃料转 dead 图标/动画。[工厂](../../../raw/current/scripts/prefabs/bernie_inactive.lua#L235-L314)、[限装](../../../raw/current/scripts/prefabs/bernie_inactive.lua#L275-L280)、[授权](../../../raw/current/scripts/prefabs/willow.lua#L136-L136) |
| `bernie_active` | fn：MakeCharacterPhysics＋companion 标签 → 主模拟 health(1000)、locomotor、combat、inventoryitem(SetOnPickupFn)、SGbernie、berniebrain | 活体形态：被拾起即 `goinactive` 转回 inactive（health%→fuel%）；无 equippable。[工厂](../../../raw/current/scripts/prefabs/bernie_active.lua#L117-L177)、[转回](../../../raw/current/scripts/prefabs/bernie_active.lua#L19-L38)、[拾起](../../../raw/current/scripts/prefabs/bernie_active.lua#L74-L97) |
| `wurt_swampitem_shadow`／`_lunar` | `wurt_terraformer_fn(anim)`：公共端 reticule＋rechargeable/weapon 标签 → 主模拟 equippable(HANDS)、rechargeable(1 总日)、spellcaster、weapon(0)；后置换 swap／tile_type／FX | Wurt 阵营投泥带：影/月同骨架；月态额外 `OnHit_Lunar` 给跟随鱼人突变。[骨架](../../../raw/current/scripts/prefabs/wurt_terraform_item.lua#L167-L236)、[影](../../../raw/current/scripts/prefabs/wurt_terraform_item.lua#L239-L259)、[月](../../../raw/current/scripts/prefabs/wurt_terraform_item.lua#L293-L314) |
| `wurt_terraform_projectile`／`wurt_terraform_cast_debuff`／`wurt_swampitem_shadow_chargedfx`／`_lunar_chargedfx` | projectile：NOCLICK＋complexprojectile(自 20 高坠落) → 生成 `wurt_swamp_terraformer`；debuff：服务端隐藏＋timer(1 总日)；fx：联网跟随 owner 的 swap_object | 投射物 `persists=false`；debuff 是防连发冷却载体；fx 在充能完成且装备时生成；[投射物](../../../raw/current/scripts/prefabs/wurt_terraform_item.lua#L338-L386)、[debuff](../../../raw/current/scripts/prefabs/wurt_terraform_item.lua#L394-L419)、[fx](../../../raw/current/scripts/prefabs/wurt_terraform_item.lua#L450-L461) |
| `wx78_drone_zap_remote` | remotefn：公共端 batteryuser 等标签＋漂浮 → 主模拟 equippable(`restrictedtag="drone_zap_user"`)、finiteuses(100)、batteryuser | 控制器：技能 `wx78_zapdrone_1` 激活后才动态加 useableequippeditem；读档由 `SKILLTREE_EQUIPPABLE_RESTRICTED_TAGS` 放行。[工厂](../../../raw/current/scripts/prefabs/wx78_drone_zap.lua#L351-L418)、[技能门禁](../../../raw/current/scripts/prefabs/wx78_drone_zap.lua#L292-L305)、[读档表](../../../raw/current/scripts/constants.lua#L2153-L2166) |
| `wx78_drone_zap` | fn：MakeFlyingCharacterPhysics＋rangedweapon 标签＋owner(net_entity) → 主模拟 weapon(0 伤假武器)、SG `SGwx78_drone_zap`；`persists=false` | 遥控器 OnUse 生成，SetOwner(doer)；视觉/索敌归 SG，本页只覆盖生成归属与扣次。[工厂](../../../raw/current/scripts/prefabs/wx78_drone_zap.lua#L128-L198)、[OnUse](../../../raw/current/scripts/prefabs/wx78_drone_zap.lua#L259-L283) |

## 机制

### 伯尼：限装、燃料与两态转换

`bernie_inactive` 的 equippable 只允许 `restrictedtag="bernieowner"`，全库仅 `willow.lua` 给 Willow 授予该标签；非 Willow 装备会被 `IsRestricted` 拒绝，配方也限 `builder_tag="pyromaniac"`。fueled 为 USAGE 型：`BERNIE_FUEL=total_day_time×15/3=2400`、rate=1/3 每秒，装备约 15 总日；owner 攻击一次额外扣 `0.01×2400=24`（骑乘或空燃料除外）。

空燃料时 `onfuelchange` 把 dapperness/保温归 0，换 `*_dead` 图标并播 dead_loop；非空恢复 DAPPERNESS_SMALL＋INSULATION_SMALL(15s)。掉地未空燃料时每秒扫 16 格内玩家：满足“疯狂领队”或“暴脾气”条件就生成 `bernie_active`（fuel%→health%）；空燃料掉地开始 3 总日（1440s）腐坏，掉 beardhair＋beefalowool＋silk 后 Remove；入袋取消腐坏。active 被拾起走 `goinactive`，health%→fuel% 转回 inactive；`bernie_big` 是技能产物，未展开。

[燃料配置](../../../raw/current/scripts/prefabs/bernie_inactive.lua#L285-L289)、[攻击磨损](../../../raw/current/scripts/prefabs/bernie_inactive.lua#L307-L311)、[空燃料双态](../../../raw/current/scripts/prefabs/bernie_inactive.lua#L125-L160)、[复活扫描](../../../raw/current/scripts/prefabs/bernie_inactive.lua#L69-L100)、[腐坏](../../../raw/current/scripts/prefabs/bernie_inactive.lua#L28-L65)、[转回](../../../raw/current/scripts/prefabs/bernie_active.lua#L19-L38)

### 投泥带：充能、防连发与地形改造

两态投泥带都有 `rechargeable`（1 总日）与 `spellcaster`；只有充能完成才挂 SpellFn。施法后 `Discharge` 一整天，并给 owner 挂 `wurt_terraform_cast_debuff`（1 总日，CanCast 返回 `TERRAFORM_TOO_SOON`），同时扣 staffsanity `-SANITY_MEDLARGE=20`（无 staffsanity 则 sanity DoDelta）。施法还会对持有者 `Inventory:FindItems` 找到的其余投泥带一起放电（该函数覆盖主物品栏、装备槽、鼠标手持 activeitem 与 overflow 装备背包）；工厂本身不设 `restrictedtag`；施法门禁按形态各用自己的用户标签——影态 `SPELLTYPES.SHADOW_SWAMP_BOMB.."_spelluser"`、月态 `LUNAR_SWAMP_BOMB.."_spelluser"`，只限制施法、不限制装备；拾取新投泥带时，若其电量高于 debuff 剩余进度才被压平，多件轮换不能缩短冷却。

施法生成 `wurt_terraform_projectile` 从 20 高坠落，OnHit 生成 `wurt_swamp_terraformer` 执行地形改造；月态额外 `OnHit_Lunar`：只对施法者的跟随者（且带 `_combat`／`_health`／`merm`）生效，有 `DoLunarMutation` 方法才调用突变，然后统一挂 `wurt_merm_planar`。充能完成且装备时生成 chargedfx 挂 owner 的 swap_object，卸装移除；存读档延迟一帧校验 debuff。

[施法](../../../raw/current/scripts/prefabs/wurt_terraform_item.lua#L46-L77)、[防连发](../../../raw/current/scripts/prefabs/wurt_terraform_item.lua#L79-L85)、[同步放电](../../../raw/current/scripts/prefabs/wurt_terraform_item.lua#L68-L76)、[拾取压平](../../../raw/current/scripts/prefabs/wurt_terraform_item.lua#L148-L164)、[投射物](../../../raw/current/scripts/prefabs/wurt_terraform_item.lua#L317-L332)、[月态](../../../raw/current/scripts/prefabs/wurt_terraform_item.lua#L264-L291)

### 电刑机：技能门禁、召唤与充电

`wx78_drone_zap_remote` 的 `restrictedtag="drone_zap_user"`：技能 `wx78_zapdrone_1` 激活时 `skilltree_wx78` 给 WX-78 加标签（附身躯壳也有），未激活时 `_onskillrefresh` 动态移除 `useableequippeditem`，右键不可用；读档时 `SKILLTREE_EQUIPPABLE_RESTRICTED_TAGS` 按 prefab 名放行，避免技能标签未恢复时丢装备。

OnUse 生成 `wx78_drone_zap` 并 SetOwner(doer)；监听 `ms_drone_zap_fired` 扣 finiteuses，每次攻击 2 次、`wx78_zapdrone_2` 技能 1 次，上限 100。控制器带 `batteryuser`，充电量=1-剩余%，满电拒绝（CHARGE_FULL），允许部分充；停用或移除遥控会 Kill 无人机。皮肤 `gothic/jewelbox` 在 remote↔drone 间镜像，reskin 同步已生成无人机。

[限装](../../../raw/current/scripts/prefabs/wx78_drone_zap.lua#L399-L406)、[技能门禁](../../../raw/current/scripts/prefabs/wx78_drone_zap.lua#L292-L305)、[授签](../../../raw/current/scripts/prefabs/skilltree_wx78.lua#L424-L430)、[OnUse](../../../raw/current/scripts/prefabs/wx78_drone_zap.lua#L259-L283)、[扣次](../../../raw/current/scripts/prefabs/wx78_drone_zap.lua#L202-L216)、[充电](../../../raw/current/scripts/prefabs/wx78_drone_zap.lua#L307-L323)、[皮肤](../../../raw/current/scripts/prefabs/wx78_drone_zap.lua#L325-L334)

## 常见陷阱与限制

- **伯尼不是“两个形态都能装备”**：equippable 只在 `bernie_inactive`；`bernie_active` 是 creature，拾起即转回 inactive。[拾起替换](../../../raw/current/scripts/prefabs/bernie_active.lua#L74-L97)
- **伯尼不是全角色可用**：`restrictedtag="bernieowner"` 由 IsRestricted 强校验，非 Willow 直接拒装；配方 builder_tag 同限 pyromaniac。[限装](../../../raw/current/scripts/components/equippable.lua#L152-L162)
- **投泥带的充能是 `Inventory:FindItems` 范围的冷却**：施法同时 Discharge 该范围内的全部同类（主物品栏／装备槽／手持／overflow 装备背包）；拾取新件只在新件电量更高时压平到当前 debuff 进度，多件轮换不缩短冷却。[同步](../../../raw/current/scripts/prefabs/wurt_terraform_item.lua#L68-L76)
- **遥控未激活技能不能右键**：`wx78_zapdrone_1` 未激活时 useableequippeditem 被动态移除；读档放行只解决受限标签，不替代技能激活。[门禁](../../../raw/current/scripts/prefabs/wx78_drone_zap.lua#L292-L305)
- **伯尼空燃料腐坏有 3 天窗口**：掉地起 1440s 才返料；入袋会取消，别把“掉地”当成立即消失。[腐坏](../../../raw/current/scripts/prefabs/bernie_inactive.lua#L28-L65)
- **官方名**：BERNIE_INACTIVE/ACTIVE＝伯尼、WURT_SWAMPITEM_SHADOW/LUNAR＝绝望投泥带／光芒投泥带、WX78_DRONE_ZAP/REMOTE＝电刑机／电刑机控制器。[伯尼名](../../../raw/current/scripts/languages/chinese_s.po#L339384-L339392)、[投泥带名](../../../raw/current/scripts/languages/chinese_s.po#L352524-L352532)、[电刑机名](../../../raw/current/scripts/languages/chinese_s.po#L352679-L352687)

## 验证记录

- 源码审阅：2026-11-07，Steam buildid 26335394。实读 bernie_inactive/active/common、willow、wurt_terraform_item、wx78_drone_zap、skilltree_wx78、wx78_possessedbody 与 berniebrain 的相关段；核验 restrictedtag 两条授权链、fueled 2400/15 总日与攻击磨损、空燃料 dead/腐坏、复活扫描、wurt 充能/同步放电/投射物/月态、wx78 技能门禁/读档表/100 用/充电；官方名与皮肤表抽查。
- 游戏运行：未运行；`bernie_big`、`SGwx78_drone_zap` 索敌/视觉、`wurt_swamp_terraformer` 地形变换主体未做运行时验证。
- 结构检查：validate_page 实测 78 行 / 13519 字节（standard 档 240 行／32 KiB 内）通过；strict 44 引用 0 错 0 警、37 唯一目标、9 源文件；record 后 22 组来源 unchanged。

返回 [[reference/prefabs/staffs-character-index|法杖与角色道具]]、[[reference/prefabs/index|Prefab 索引]]。
