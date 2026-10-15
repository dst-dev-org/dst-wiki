---
title: "Boss 与狂欢节装备（恐怖盾牌／派对气球／防腐喷雾／推杆）"
created: "2026-11-07"
updated: "2026-11-07"
type: prefab
size_class: standard
summary: "四件 HANDS 事件/Boss/狂欢节装备：恐怖盾牌（armor 315/0.8、进食修复、攻击自损 2）、派对气球（装备生成随行气球、被打掉落、彩带云 buff）、防腐喷雾（waxspray 只对 needswaxspray 目标、75 用）、推杆（golfclub＋reticule、17 伤、tee 生成不可制作）。"
aliases: ["shieldofterror", "balloonparty", "balloonparty_confetti_cloud", "beeswax_spray", "carnivalgame_golfclub", "恐怖盾牌", "派对气球", "防腐喷雾", "推杆", "高尔夫球杆", "Shield of Terror", "Party Balloon", "Embalming Spritz", "Putter"]
tags: [prefab, item, equipment]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/shieldofterror.lua", "prefabs/eyeofterror.lua", "prefabs/balloonparty.lua", "prefabs/balloons_common.lua", "prefabs/beeswax_spray.lua", "prefabs/carnival_golfclub.lua", "prefabs/carnivalgame_golf_tee.lua", "prefabs/wes.lua", "components/equippable.lua", "components/weapon.lua", "components/armor.lua", "components/eater.lua", "components/shadowlevel.lua", "components/wax.lua", "components/waxable.lua", "components/finiteuses.lua", "components/golfclub.lua", "componentactions.lua", "recipes.lua", "tuning.lua", "constants.lua", "strings.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# Boss 与狂欢节装备（恐怖盾牌／派对气球／防腐喷雾／推杆）

## 快速判断

- 职责与适用：四件 HANDS 装备分别来自双子魔眼、Wes 派对机、暗影锻造与狂欢节高尔夫 tee。适合查“装备态生成随行体/覆盖符号”“护甲自损与进食修复”“waxspray 目标过滤”“活动设施生成不可制作物”四类接线。
- 优先使用：护甲/武器/进食分别查 [[reference/components/armor]]、[[reference/components/weapon]]、[[reference/components/eater]]；气球随行体见 [[reference/prefabs/balloon_held_child]]；蜡制链见 [[reference/components/wax]]、[[reference/components/waxable]]。
- 重要副作用：恐怖盾牌每次以该盾攻击触发 `onattackother` 都自损 2 护甲（事件在伤害结算前）、靠进食修复；派对气球在 owner 被有效攻击时掉落（排除重定向/无攻击者/自身）；防腐喷雾只对 `needswaxspray` 的 `waxable` 目标给 WAX 动作；推杆由 tee 生成、`persists=false`＋irreplaceable，无配方无掉落。
- 修改边界：双子魔眼的 Boss AI、派对 buff 的完整理智曲线、暗影锻造配方链、高尔夫小游戏与推杆物理不展开。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `shieldofterror` | fn：公共端 handfed/fedbyall/toolpunch/weapon/shadowlevel 标签＋漂浮 → 主模拟 eater(OMNI+HORRIBLE)、weapon、armor、equippable(HANDS)、shadowlevel(2) | HANDS 盾牌；眼魔双子 EXTRA_LOOT；进食修复护甲，攻击自损 2；护甲 315、吸收 0.8。[工厂](../../../raw/current/scripts/prefabs/shieldofterror.lua#L53-L124)、[掉落](../../../raw/current/scripts/prefabs/eyeofterror.lua#L688-L688) |
| `balloonparty` | fn：漂浮气球物理＋balloon 标签 → 主模拟 BALLOONS.MakeBalloonMasterInit＋equippable（复用 balloons_common 的 OnEquip_Hand／OnUnequip_Hand） | Wes 派对气球机：装备生成 `balloon_held_child` 随行体，pop 落彩带云；被攻击即 DropItem。[工厂](../../../raw/current/scripts/prefabs/balloonparty.lua#L62-L119)、[装备](../../../raw/current/scripts/prefabs/balloons_common.lua#L93-L115) |
| `balloonparty_confetti_cloud`／`_confetti_balloon`／`_buff` | confetti_fn：联网动画 FX＋每秒 ApplyBuff；confettiballoon_fn：非联网 FX 飞升；buff_fn：服务端隐藏 debuff（非主模拟自删） | cloud 每秒给 2.6 格内玩家挂 buff；小气球四颗升起；buff 按“派对规模”加理智。[cloud](../../../raw/current/scripts/prefabs/balloonparty.lua#L136-L188)、[buff](../../../raw/current/scripts/prefabs/balloonparty.lua#L193-L253) |
| `beeswax_spray` | fn：公共端动画＋漂浮 → 主模拟 inventoryitem、equippable(HANDS)、wax(`SetIsSpray`)、finiteuses(75) | 喷蜡罐：挂 `waxspray` 标签，只对 `needswaxspray` 目标给 WAX 动作；耗尽 Remove。[工厂](../../../raw/current/scripts/prefabs/beeswax_spray.lua#L102-L143)、[wax 标签](../../../raw/current/scripts/components/wax.lua#L19-L21)、[动作匹配](../../../raw/current/scripts/componentactions.lua#L1932-L1936) |
| `carnivalgame_golfclub` | fn：公共端 weapon 标签＋漂浮＋golfclub_reticule → 主模拟 equippable(HANDS)、weapon(CANE_DAMAGE)、golfclub | 注册名与文件名不同（`carnival_golfclub.lua`）；由高尔夫 tee 生成，`persists=false`＋irreplaceable，不可制作。[工厂](../../../raw/current/scripts/prefabs/carnival_golfclub.lua#L37-L80)、[生成](../../../raw/current/scripts/prefabs/carnivalgame_golf_tee.lua#L103-L108) |

## 机制

### 恐怖盾牌：护甲自损与进食修复

`shieldofterror` 同时带 weapon 与 armor：weapon 伤害 `wilson_attack×1.5=51`；armor 为 `150×3×0.7=315` 耐久、吸收 0.8。owner 的 `onattackother` 监听里，若 `data.weapon==inst` 就 `TakeDamage(2)`；进食是唯一常规修复路径——`eater` 吃 OMNI＋HORRIBLE（可生肉），`Repair(|health|×4.0＋|hunger|×1.75)`。自损监听的是 `onattackother`（在实际伤害结算前触发），因此扣 2 与“这一击是否真的扣到目标”无关；装备时 onequip 覆盖 owner 的 `lantern_overlay` 符号并隐藏 swap_object，`shadowlevel` 默认 2；获取仅双子魔眼 EXTRA_LOOT，无配方，拆解返齿轮 2＋噩梦燃料 3。

[配置](../../../raw/current/scripts/prefabs/shieldofterror.lua#L95-L108)、[自损](../../../raw/current/scripts/prefabs/shieldofterror.lua#L89-L93)、[进食修复](../../../raw/current/scripts/prefabs/shieldofterror.lua#L39-L50)、[常量](../../../raw/current/scripts/tuning.lua#L6445-L6448)、[拆解](../../../raw/current/scripts/recipes.lua#L1679-L1679)

### 派对气球：随行体、掉落与彩带 buff

`OnEquip_Hand` 生成 `balloon_held_child` 随行体；owner 的 `attacked` 事件经 `ShouldProcOnAttackedOrBlocked`（非重定向、有攻击者、非自身）判定成立才 DropItem，所以派对气球不是武器。pop 后 5 帧生成 `balloonparty_confetti_cloud`，cloud 每秒给 2.6 格内 AllPlayers 挂 `balloonparty_buff`；buff 每 2 秒根据“参会者”计数（有 buff 玩家＋cloud 实体）从 `CONFFETI_PARTY_SANITY_DELTA={0,1,2,3,3.5,4}` 取档 DoDelta，无 cloud 或死亡即 Stop。配方为 `balloons_empty×0`＋SANITY 5、`builder_tag="balloonomancer"`（Wes）、`sg_state="makeballoon"`。

[OnEquip_Hand](../../../raw/current/scripts/prefabs/balloons_common.lua#L93-L113)、[OnPop](../../../raw/current/scripts/prefabs/balloonparty.lua#L53-L60)、[buff](../../../raw/current/scripts/prefabs/balloonparty.lua#L193-L213)、[配方](../../../raw/current/scripts/recipes.lua#L375-L375)

### 防腐喷雾：waxspray 目标过滤

`wax:SetIsSpray` 让组件挂 `waxspray` 标签；`componentactions` 只在目标 `waxable` 且状态匹配 `needswaxspray` 时给 WAX 动作，所以喷蜡对象是**种在实地的植物**（浆果丛／草／树苗系、农田作物、古树等 `_waxed` 变体），不是任意物品。finiteuses 为 `BEESWAX_SPRAY_USES=75`，耗尽 Remove；配方噩梦燃料 5＋蜂蜡 3＋蚊子血囊 2，`TECH.SHADOWFORGING_TWO`＋`station_tag="shadow_forge"`、nounlock。

[标签](../../../raw/current/scripts/components/wax.lua#L19-L21)、[动作过滤](../../../raw/current/scripts/componentactions.lua#L1932-L1936)、[finiteuses](../../../raw/current/scripts/prefabs/beeswax_spray.lua#L133-L136)、[配方](../../../raw/current/scripts/recipes.lua#L872-L872)

### 推杆：golfclub、reticule 与 tee 生成

主模拟 `golfclub` 组件负责挥杆，公共端 `golfclub_reticule` 负责瞄准环，卸装 `StopAiming`。weapon 伤害 `CANE_DAMAGE=wilson_attack×0.5=17`；没有配方，由 `carnivalgame_golf_tee` 设施生成（`persists=false`＋irreplaceable＋tee_owner）；小游戏结束/清理时设施链 `ClearGolfClub` 直接 Remove，读档不保留，也不能拆解回材料。

[golfclub](../../../raw/current/scripts/prefabs/carnival_golfclub.lua#L55-L78)、[weapon](../../../raw/current/scripts/prefabs/carnival_golfclub.lua#L70-L71)、[常量](../../../raw/current/scripts/tuning.lua#L373-L373)、[tee 生成](../../../raw/current/scripts/prefabs/carnivalgame_golf_tee.lua#L103-L108)

## 常见陷阱与限制

- **恐怖盾牌的自损不是 armordamage 通用规则**：磨损写在 owner 的 `onattackother` 监听里，只有 `data.weapon==inst` 才扣 2；别把“盾牌每次攻击自损”当成所有 shield 的默认行为。[自损](../../../raw/current/scripts/prefabs/shieldofterror.lua#L89-L93)
- **派对气球被有效攻击才掉落**：`OnEquip_Hand` 监听 owner 的 `attacked`，经 `ShouldProcOnAttackedOrBlocked` 排除重定向、无攻击者与自身来源后才 DropItem；它不是武器，别按“被打掉耐久”写。[掉落](../../../raw/current/scripts/prefabs/balloons_common.lua#L101-L113)
- **防腐喷雾不能喷任意物品**：目标必须是 `waxable` 且 `needswaxspray` 状态匹配；普通蜂蜡走 `waxable` 的 WAX，两者不是同一条动作。[过滤](../../../raw/current/scripts/componentactions.lua#L1932-L1936)
- **推杆无常规获取**：由高尔夫 tee 生成且 `persists=false`＋irreplaceable，设施清理链 `ClearGolfClub` 负责 Remove，无配方无掉落；把它写进“可制作装备”会错。[生成](../../../raw/current/scripts/prefabs/carnivalgame_golf_tee.lua#L103-L108)
- **官方名**：SHIELDOFTERROR＝恐怖盾牌、BALLOONPARTY＝派对气球、BEESWAX_SPRAY＝防腐喷雾、CARNIVALGAME_GOLFCLUB＝推杆。[盾名](../../../raw/current/scripts/languages/chinese_s.po#L349329-L349332)、[气球名](../../../raw/current/scripts/languages/chinese_s.po#L338999-L339002)、[喷雾名](../../../raw/current/scripts/languages/chinese_s.po#L339364-L339367)、[推杆名](../../../raw/current/scripts/languages/chinese_s.po#L340299-L340302)

## 验证记录

- 源码审阅：2026-11-07，Steam buildid 26335394。实读 shieldofterror、balloonparty、balloons_common、beeswax_spray、carnival_golfclub、carnivalgame_golf_tee 与 eyeofterror 的掉落段；核验护甲 315/0.8、weapon 51、自损 2、进食 4.0/1.75、shadowlevel 2、气球随行/掉落/彩带 buff、waxspray 过滤与 75 用、推杆 17 伤与 tee 生成；官方名按 `chinese_s.po` 四键抽查。
- 游戏运行：未运行；气球 buff 的完整理智曲线、推杆物理、双子魔眼战斗表现未做运行时验证。
- 结构检查：validate_page 实测 76 行 / 11245 字节（standard 档 240 行／32 KiB 内）通过；strict 36 引用 0 错 0 警、30 唯一目标、12 源文件；record 后 23 组来源 unchanged。

返回 [[reference/prefabs/staffs-character-index|法杖与角色道具]]、[[reference/prefabs/index|Prefab 索引]]。
