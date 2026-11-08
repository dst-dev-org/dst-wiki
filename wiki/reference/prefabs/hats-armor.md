---
title: "防护与战斗帽族（footballhat、ruinshat 等 25 注册）"
created: "2026-11-07"
updated: "2026-11-08"
type: prefab
summary: "hats.lua 中带 armor 组件或明确战斗/防护用途的帽子家族：MakeHat 头部装配骨架之上逐帽 armor/planardefense/setbonus/resistance 等战斗组件、proc/套装/落物抵挡触发、真实配方与掉落、官方名与消费端标签，供模组参照原版做防护帽与受击触发。"
aliases: ["footballhat", "橄榄球头盔", "ruinshat", "铥矿皇冠", "slurtlehat", "背壳头盔", "skeletonhat", "骨头头盔", "beehat", "养蜂帽", "hivehat", "蜂王冠", "eyemaskhat", "眼面具", "wathgrithrhat", "战斗头盔", "wathgrithr_improvedhat", "统帅头盔", "cookiecutterhat", "饼干切割机帽子", "dreadstonehat", "绝望石头盔", "lunarplanthat", "亮茄头盔", "voidclothhat", "虚空风帽", "wagpunkhat", "W.A.R.B.I.S.头戴齿轮", "scraphat", "拾荒尖帽", "mermarmorhat", "沼泽斗士头盔", "mermarmorupgradedhat", "沼泽战士头盔", "shadow_thrall_parasitehat", "mask_halfwithat", "蠢人面具", "mask_sagehat", "贤者面具", "mask_toadyhat", "马屁精面具", "woodcarvedhat", "硬木帽", "mask_princesshat", "公主面具", "yoth_knighthat", "镀金骑士头盔", "alterguardianhat", "启迪之冠", "防护帽", "战斗帽", "armor 帽", "帽子装甲"]
tags: [prefab, item, equipment]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/hats.lua", "tuning.lua", "recipes.lua", "languages/chinese_s.po", "components/armor.lua", "components/equippable.lua", "components/sanity.lua", "components/resistance.lua", "components/setbonus.lua", "components/luckuser.lua", "components/luckitem.lua", "components/inventory.lua", "components/combat.lua", "components/yoth_knightmanager.lua", "components/playerlightningtarget.lua", "components/playervision.lua", "components/shadowparasitemanager.lua", "componentutil.lua", "components/quaker.lua", "prefabs/spider.lua", "prefabs/slurtle.lua", "prefabs/beequeen.lua", "prefabs/stalker.lua", "prefabs/eyeofterror.lua", "prefabs/bat_boss.lua", "prefabs/shadowthrall_parasite.lua", "prefabs/merm.lua", "prefabs/merm_supply_structures.lua", "prefabs/player_common.lua", "prefabs/knight.lua", "prefabs/charlie_stage.lua", "prefabs/armor_yoth_knight.lua", "prefabs/skilltree_woodie.lua", "prefabs/woodie.lua", "brains/mermguardbrain.lua", "stategraphs/SGwilson.lua", "stategraphs/SGwagdrone_rolling.lua", "stategraphs/commonstates.lua", "aoeutil.lua", "widgets/itemtile.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: complex
---

# 防护与战斗帽族（footballhat、ruinshat 等 25 注册）

## 快速判断

- 职责与适用：`prefabs/hats.lua` 共 85 个 `MakeHat` 玩法帽注册，本页收其中 25 个「防护/战斗」帽：21 顶带 armor 组件（含 4 顶新系 planar 帽与 woodcarvedhat 抵挡落物）、3 顶宿主面具（`mask_halfwithat`／`mask_sagehat`／`mask_toadyhat`，armor＋宿主标记）、1 顶无 armor 但明确战斗用途（`alterguardianhat` 攻击时生成 `alterguardianhat_projectile` 攻击弹体）。工厂骨架（equipslot＝HEAD、换装符号三路、皮肤协议）与 11 个 FX／1 个技术实体注册归 [[reference/prefabs/hats|帽子跟随层与帽灯特效族]]，本页不重复。
- 优先使用：查某顶帽的 armor 耐久/吸收与附加组件 → [armor 帽表](#注册与构造)；查受击 proc／套装触发／落物抵挡 → 对应表行与[关联机制导航](#关联机制导航)；查"限制只有特定实体能装备"→ mermarmor 行；自己写防护帽/受击触发从[典型接入](#典型接入)进入。
- 重要副作用：armor 耐久按「`Inventory:ApplyDamage` 分摊到该件的吸收量」扣减——先乘伤害类型抗性、取各件最高有效吸收率得总吸收量、再按各自吸收率占比分给每件，另加弱点与该件吸收的特殊伤害，最后 `Armor:TakeDamage` 乘该件损耗倍率，并推 `armordamaged`（[分摊](../../../raw/current/scripts/components/inventory.lua#L408-L499)、[扣耐久](../../../raw/current/scripts/components/armor.lua#L156-L164)）；`ruinshat` proc 期间改吸收并把伤害按 5% 转理智扣减；`wagpunkhat` 阶段改持有者伤害倍率、卸帽移除；`shadow_thrall_parasitehat` 卸下即杀宿主；YOTH 两顶套装禁用/卸帽即反目骑士。逐帽见注册表行。
- 修改边界：本页只覆盖帽本体的战斗/防护组件与触发链；帽 FX 层、帽灯、储物代理归 [[reference/prefabs/hats]]；纯环境/功能帽（保暖、防水、防暑、地形改造、伪装跟随、贝雷帽、蝙蝠帽等）归 [[reference/prefabs/hats-environment|环境与功能帽族]]（`spiderhat`/`bat_bosscorpsehat` 已按主代理裁决移入，见[分派出页](#分派出页如实分派不进本页)）；事件/装扮帽归 [[reference/prefabs/hats-event|事件装扮帽族]]。数值区分配置值与推导值：armor 耐久配置式里的联机乘数 `multiplayer_armor_durability_modifier = 0.7`（[tuning.lua#L33-L38](../../../raw/current/scripts/tuning.lua#L33-L38)，wilson_health＝150），本页表格给出推导值。

## 注册与构造

`MakeHat(name)` 的注册名是 `name.."hat"`、动画 build 是 `"hat_"..name`（[注册名推导](../../../raw/current/scripts/prefabs/hats.lua#L16-L20)）；`simple()` 骨架统一 `equipslot = EQUIPSLOTS.HEAD` 并装配 inventoryitem／inspectable／tradable／equippable（[骨架](../../../raw/current/scripts/prefabs/hats.lua#L188-L235)），无专属 fn 落 `default()`（[L273-L275](../../../raw/current/scripts/prefabs/hats.lua#L273-L275)）。换装符号三路 `_onequip`／`opentop_onequip`／`fullhelm_onequip`（全头盔额外隐藏 face／beard／cheeks 符号并 `UseHeadHatExchange`）与皮肤协议（`GetSkinBuild` → `OverrideItemSkinSymbol`＋`equipskinneditem` 事件）见 [[reference/prefabs/hats|hats.md]] 骨架小节（[换装三路](../../../raw/current/scripts/prefabs/hats.lua#L31-L171)）。

全量枚举：`grep -n 'MakeHat("'` 命中 85 个注册名（return 区 [hats.lua#L7860-L7953](../../../raw/current/scripts/prefabs/hats.lua#L7860-L7953)），逐顶按组件分派如下；分派依据全部实读自各帽 fn 本体。

### armor 帽（24 注册）

耐久/吸收为推导值（配置式 ×0.7 联机耐久乘数）；"耐久×吸收"单位为点数×比例。

| 注册名（官方名） | armor：耐久×吸收 | 附加组件／关键回调 | 获取 | 证据 |
|---|---|---|---|---|
| `footballhat`（橄榄球头盔） | 315×0.8 | waterproofer 小；onequip 带 `open_top_hat` 分支（本体未加该标签，分支保留未核验用途） | 配方猪皮1＋绳1（SCIENCE_TWO） | [hats#L372-L389](../../../raw/current/scripts/prefabs/hats.lua#L372-L389)、[tuning#L1832-L1833](../../../raw/current/scripts/tuning.lua#L1832-L1833)、[recipes#L504](../../../raw/current/scripts/recipes.lua#L504) |
| `woodcarvedhat`（硬木帽） | 262.5×0.7 | `resistance` 抵挡 quakedebris/lunarhaildebris 落物（`OnResistDamageFn` 把碎片伤害转 armor 耐久，原注释注明不计 bonus/planar 伤害）；armor 弱点 beaver（受击方是变身海狸时 armor 额外磨损 17，不加本体伤害）；`wood` 标签（命中音效）；fuel LARGE_FUEL＋可燃＋可蔓延 | 配方露西斧0（仅要求持有，不消耗）＋木头6＋松果1（TECH.NONE，builder_skill woodie_human_lucy_2，sg_state carvewood 雕刻制作） | [工厂](../../../raw/current/scripts/prefabs/hats.lua#L391-L425)、[分派](../../../raw/current/scripts/prefabs/hats.lua#L6927-L6928)、[tuning#L1837-L1838](../../../raw/current/scripts/tuning.lua#L1837-L1838)、[recipes#L174](../../../raw/current/scripts/recipes.lua#L174)、[弱点消费](../../../raw/current/scripts/components/inventory.lua#L451) |
| `ruinshat`（铥矿皇冠） | 840×0.9（proc 期改 1）＋`hardarmor`/`metal` 标签 | `shadowlevel` 组件；监听持有者 `attacked` 走 proc：33% 运（`LuckFormulas.RuinsHatProc`＝基率+0.1 运加成）、4 秒 100% 吸收＋伤害×0.05 转理智、冷却 5 秒；proc 挂 `forcefieldfx` 到 owner | 配方铥矿4＋噩梦燃料4（ANCIENT_FOUR） | [proc](../../../raw/current/scripts/prefabs/hats.lua#L455-L484)、[构造](../../../raw/current/scripts/prefabs/hats.lua#L496-L555)、[tuning#L1852-L1857](../../../raw/current/scripts/tuning.lua#L1852-L1857)、[recipes#L807](../../../raw/current/scripts/recipes.lua#L807)、[运公式](../../../raw/current/scripts/componentutil.lua#L2164) |
| `slurtlehat`（背壳头盔） | 525×0.9 | waterproofer 小；onequip 检查 BODY 槽 `armorsnurtleshell` 发放组合成就 | 蜗牛（slurtle）10% 掉落 | [hats#L1231-L1260](../../../raw/current/scripts/prefabs/hats.lua#L1231-L1260)、[slurtle#L19-L25](../../../raw/current/scripts/prefabs/slurtle.lua#L19-L24)、[tuning#L1859-L1860](../../../raw/current/scripts/tuning.lua#L1859-L1860) |
| `skeletonhat`（骨头头盔） | 945×0.7 | `shadowlevel`＋`shadowdominance` 组件；装备令持有者 induced insanity＋魔法 `CRAZINESS_MED`；waterproofer 小 | stalker 概率掉落 | [hats#L2858-L2910](../../../raw/current/scripts/prefabs/hats.lua#L2858-L2910)、[stalker#L1017](../../../raw/current/scripts/prefabs/stalker.lua#L1017)、[tuning#L1871-L1872](../../../raw/current/scripts/tuning.lua#L1871-L1872) |
| `beehat`（养蜂帽） | 1050×0.8，`SetTags({"bee"})` 仅吸收 bee 标签攻击 | waterproofer 小 | 配方蛛丝8＋绳1（SCIENCE_TWO） | [hats#L282-L300](../../../raw/current/scripts/prefabs/hats.lua#L282-L300)、[tuning#L1861-L1862](../../../raw/current/scripts/tuning.lua#L1861-L1862)、[recipes#L543](../../../raw/current/scripts/recipes.lua#L543) |
| `hivehat`（蜂王冠） | 945×0.7＋`regal` 标签 | 装备置持有者 `sanity.neg_aura_absorb`＝0.5（负理智光环吸收一半，卸下清零）；waterproofer 小 | 蜂王掉落 | [hats#L2047-L2090](../../../raw/current/scripts/prefabs/hats.lua#L2047-L2090)、[sanity#L570-L582](../../../raw/current/scripts/components/sanity.lua#L570-L582)、[beequeen#L40](../../../raw/current/scripts/prefabs/beequeen.lua#L40)、[tuning#L1867-L1869](../../../raw/current/scripts/tuning.lua#L1867-L1869) |
| `eyemaskhat`（眼面具） | 315×0.8（复用 ARMOR_FOOTBALLHAT） | `eater` 组件（OMNI+HORRIBLE、生肉、强胃）；吃食按吸收倍率 4.0/1.75 折损修 armor；waterproofer 小 | 恐惧之眼掉落 | [hats#L2314-L2367](../../../raw/current/scripts/prefabs/hats.lua#L2314-L2367)、[eyeofterror#L464](../../../raw/current/scripts/prefabs/eyeofterror.lua#L464)、[tuning#L1832](../../../raw/current/scripts/tuning.lua#L1832-L1833) |
| `wathgrithrhat`（战斗头盔） | 525×0.8＋`battlehelm` 标签 | Wigfrid 技能 `helmetcondition` 级数改耐久损耗倍率（`conditionlossmultipliers` modifier "arsenal_helm"）；waterproofer 小 | 配方金2＋石2（NONE，builder_tag valkyrie） | [hats#L1423-L1506](../../../raw/current/scripts/prefabs/hats.lua#L1423-L1506)、[recipes#L179](../../../raw/current/scripts/recipes.lua#L179)、[tuning#L1849-L1850](../../../raw/current/scripts/tuning.lua#L1849-L1850) |
| `wathgrithr_improvedhat`（统帅头盔） | 682.5×0.8＋`battlehelm`/`heavyarmor` 标签 | `planardefense`（技能 `wathgrithr_arsenal_helmet_4` 加 `HELM_PLANAR_DEF` bonus）；技能 helmet_5 加 `battleborn_repairable` 标签；insulator 小＋waterproofer 小中 | 配方金2＋牛毛2＋大理石1（NONE，builder_skill helmet_3） | [hats#L1508-L1540](../../../raw/current/scripts/prefabs/hats.lua#L1508-L1540)、[recipes#L193](../../../raw/current/scripts/recipes.lua#L193)、[tuning#L8072-L8073](../../../raw/current/scripts/tuning.lua#L8073-L8074) |
| `cookiecutterhat`（饼干切割机帽子） | 525×0.7 | waterproofer 小中 | 配方饼干切割机壳4＋绳1（SCIENCE_TWO） | [hats#L1210-L1224](../../../raw/current/scripts/prefabs/hats.lua#L1210-L1224)、[recipes#L505](../../../raw/current/scripts/recipes.lua#L505)、[tuning#L1834-L1835](../../../raw/current/scripts/tuning.lua#L1834-L1835) |
| `dreadstonehat`（绝望石头盔） | 840×0.9＋planar 5＋`shadow_aligned` 抗 0.9 | `hardarmor`/`dreadstone`/`shadow_item`/`shadowlevel` 标签；armor 自愈：戴上已受损的帽或 armor 受击后启动周期 1 秒的任务，每次修复 `maxcondition × rate`，其中 `rate = 1/Lerp(1/MAXRATE, 1/MINRATE, 理智百分比)`＝每秒最大耐久的 **0.0667%–0.1111%**（理智越高越慢），套装再 ×1.5；装满或卸下即停，且只在普通疯狂模式下真的回血；`dapperfn` 给理智损耗：`CRAZINESS_MED`（= −100/`day_time`，day_time＝300 秒 → **−0.333 理智/秒**；别拿 480 秒的 `total_day_time` 换算）——**不成套**时只有本件 regen 任务存在才扣一份 `CRAZINESS_MED`；**戴同族 `armordreadstone` 成套**时只要任一件有自愈任务，两件就各扣 `CRAZINESS_MED / 2`（≈**−0.167 理智/秒**，合计仍是一份；常量本身为负，别再前置负号，否则变成回理智）；两种情况都要普通疯狂模式；setbonus DREADSTONE；waterproofer 小 | 配方绝望石4＋噩梦燃料4（TECH.LOST） | [hats#L3705-L3812](../../../raw/current/scripts/prefabs/hats.lua#L3705-L3812)、[recipes#L503](../../../raw/current/scripts/recipes.lua#L503)、[tuning#L1877-L1888](../../../raw/current/scripts/tuning.lua#L1877-L1888) |
| `lunarplanthat`（亮茄头盔） | 830×0.8＋planar 10 | fullhelm 换装＋`lunarplanthat_fx` 跟随层（重建式）；grue 黑暗免疫；`gestaltprotection`/`goggles`/`fullhelm_hat` 标签；`lunar_aligned` 抗性＋setbonus LUNARPLANT（套装启用加 lunar 抗）；Wortox 技能接 lunar resist；`MakeForgeRepairable` 破损/修复（equippable 动态移除/重建） | 配方纯粹辉煌4＋亮茄皮2（LUNARFORGING_TWO，lunar_forge） | [hats#L3814-L3926](../../../raw/current/scripts/prefabs/hats.lua#L3814-L3926)、[recipes#L849](../../../raw/current/scripts/recipes.lua#L849)、[tuning#L1894-L1896](../../../raw/current/scripts/tuning.lua#L1894-L1896) |
| `voidclothhat`（虚空风帽） | 830×0.8＋planar 10＋`shadow_aligned` 抗性 | fullhelm＋`voidclothhat_fx`；`miasmaimmune`/`shadow_item`/`shadowlevel` 标签；**单戴即生效、不需要长袍**：装备时监听持有者 `equip`/`unequip`/`attacked`/`onattackother`，只有 HANDS 槽、带 `shadow_item` 标签且有 `planardamage` 的物品成为叠层目标（`AddBonus` key `voidclothhat_rampingbuff`）；每次 `onattackother` 叠 1 层，最多 6 层＝+24 位面伤害，3 秒未继续攻击清零，非自身装备来源的 `attacked` 也清零；换手/卸手立即把旧目标归零。套装 VOIDCLOTH 回调只增减 `shadow_aligned` 抗性，不参与叠层；`MakeForgeRepairable` | 配方噩梦燃料4＋虚布2（SHADOWFORGING_TWO，shadow_forge） | [hats#L3928-L4149](../../../raw/current/scripts/prefabs/hats.lua#L3928-L4149)、[recipes#L867](../../../raw/current/scripts/recipes.lua#L867)、[tuning#L1906-L1908](../../../raw/current/scripts/tuning.lua#L1906-L1908) |
| `wagpunkhat`（W.A.R.B.I.S.头戴齿轮） | 730×0.85＋planar 5＋`hardarmor`/`metal` 标签 | `targettracker` 追踪：攻击目标 2 秒定靶、阶段 10/16/20 秒把持有者 `externaldamagemultipliers` 升到 1.10/1.15/1.20（装备即 1.05，卸帽 RemoveModifier）；`talker` 说话＋`wagpunkhat_classified` 网络代理＋net_entity wearer/target；`equippable.insulated`＝true（防雷击）；`MakeForgeRepairable` | 配方 wagpunk_bits8＋晶体管3＋启迪冠残片1（TECH.LOST） | [init](../../../raw/current/scripts/prefabs/hats.lua#L4207-L4239)、[阶段](../../../raw/current/scripts/prefabs/hats.lua#L4379-L4443)、[构造](../../../raw/current/scripts/prefabs/hats.lua#L4608-L4660)、[recipes#L861](../../../raw/current/scripts/recipes.lua#L861)、[tuning#L8030-L8042](../../../raw/current/scripts/tuning.lua#L8031-L8043) |
| `scraphat`（拾荒尖帽） | 262.5×0.85 | `junk` 标签；waterproofer 小 | 无 Recipe2；有 DeconstructRecipe（wagpunk_bits3）；唯一产出＝拾荒疯猪（`daywalker2`）的尖刺：击败或撤场时 `DropItemAsLoot` 按 `"daywalker2_"..item.."_loot_fx"` 动态拼名，`daywalker2_spike_loot_fx` 的 `animover` 或读档 `OnLoad` 时生成，耐久随机 6%–20%（FX 层见 [[reference/prefabs/cavein_dust_low|拾荒疯猪碎装表现族]]） | [hats#L4721-L4742](../../../raw/current/scripts/prefabs/hats.lua#L4721-L4742)、[recipes#L1683](../../../raw/current/scripts/recipes.lua#L1683)、[tuning#L1840-L1841](../../../raw/current/scripts/tuning.lua#L1840-L1841)、[拼名](../../../raw/current/scripts/prefabs/daywalker2.lua#L365-L395)、[击败分支](../../../raw/current/scripts/stategraphs/SGdaywalker2.lua#L2285-L2298)、[撤场分支](../../../raw/current/scripts/prefabs/daywalker2.lua#L751-L763)、[掉落与读档](../../../raw/current/scripts/prefabs/daywalker2_items.lua#L10-L17) |
| `mermarmorhat`（沼泽斗士头盔） | 210×0.3 | `restrictedtag`＝"merm_npc"（仅拦玩家装备，见关联导航）；`mermarmorhat` 标签供鱼人守卫 AI 拾取判定 | 鱼人补给结构发放（supplyprefab） | [hats#L5474-L5508](../../../raw/current/scripts/prefabs/hats.lua#L5474-L5508)、[merm_supply_structures#L473](../../../raw/current/scripts/prefabs/merm_supply_structures.lua#L473)、[tuning#L8301-L8302](../../../raw/current/scripts/tuning.lua#L8302-L8303) |
| `mermarmorupgradedhat`（沼泽战士头盔） | 315×0.5 | 同上，标签 `mermarmorupgradedhat` | 鱼人补给结构发放（升级档） | [hats#L5512-L5545](../../../raw/current/scripts/prefabs/hats.lua#L5512-L5545)、[merm_supply_structures#L485](../../../raw/current/scripts/prefabs/merm_supply_structures.lua#L485)、[tuning#L8304-L8305](../../../raw/current/scripts/tuning.lua#L8305-L8306) |
| `shadow_thrall_parasitehat`（虚空面具·宿主态） | 2100×0.2 | 装备给宿主挂 `planardamage`/`planarentity`（卸下还原）、非玩家换 hostedbrain、禁睡眠、combat retarget/keeptarget、talker 接管；`keepondeath`＋`nosteal`＋`monsterhat` 标签；卸下/移除链杀宿主 | shadowparasitemanager 把它装备到蜘蛛/兔人/石虾等宿主身上 | [装备](../../../raw/current/scripts/prefabs/hats.lua#L5625-L5694)、[卸下](../../../raw/current/scripts/prefabs/hats.lua#L5696-L5763)、[构造](../../../raw/current/scripts/prefabs/hats.lua#L5789-L5815)、[manager#L135-L140](../../../raw/current/scripts/components/shadowparasitemanager.lua#L135-L140)、[tuning#L8547-L8548](../../../raw/current/scripts/tuning.lua#L8548-L8549) |
| `mask_halfwithat`／`mask_sagehat`／`mask_toadyhat`（蠢人/贤者/马屁精面具） | 2100×0.2（与上行同组） | 装备/卸下加/删 `shadowthrall_parasite_mask` 标签（manager 选宿主的排除条件）；`mask_common` 基座带燃料+可燃 | 宿主死亡 luck 运掉落三选一（DROP_SHADOWTHRALL_MASKHAT_CHANCE） | [mask_armor](../../../raw/current/scripts/prefabs/hats.lua#L2731-L2755)、[fn 映射](../../../raw/current/scripts/prefabs/hats.lua#L6905-L6911)、[掉落](../../../raw/current/scripts/prefabs/hats.lua#L5729-L5743)、[manager#L174](../../../raw/current/scripts/components/shadowparasitemanager.lua#L174) |
| `mask_princesshat`（公主面具） | 105×0.6 | `mask_common(noburn)` 基座无燃料不可燃；`metal`/`hardarmor`/`waterproofer` 标签＋waterproofer 小；`petleash` 收编 knight_yoth（上限 4＝YOTH_HORSE_NAMES，入队 MakeFriendly、领队非玩家即反目并给近旁玩家上冷却 debuff）、`migrationpetowner` 宠物随帽迁移＋玩家 despawn 宠物随撤；setbonus YOTH_PRINCESS：**须配身体件 `costume_princess_body` 才启用**（身体件在 `costumes.lua` 注册同名 setbonus，与 YOTH_KNIGHT 的 `armor_yoth_knight` 是两套）；启用推 `ms_register_yoth_princess`＋5 秒周期收编 16 格无主镀金骑士（`gilded_knight` 标签、horseman_type 去重）＋帽与身体件各挂 `unluckysource`＋owner 运 −1，禁用逐骑 DetachPet/MakeHostile/SuggestTarget；卸帽/移除强制反目 | Charlie 剧场一次性送达（YOTH 骑士神龛激活期由 hedgehound 携带，timer 保证仅一次）；DeconstructRecipe 金4＋蛛丝2 | [构造](../../../raw/current/scripts/prefabs/hats.lua#L6399-L6442)、[套装回调](../../../raw/current/scripts/prefabs/hats.lua#L6345-L6372)、[反目链](../../../raw/current/scripts/prefabs/hats.lua#L6185-L6233)、[tuning#L9285-L9286](../../../raw/current/scripts/tuning.lua#L9286-L9287)、[recipes#L1738](../../../raw/current/scripts/recipes.lua#L1738)、[送达](../../../raw/current/scripts/prefabs/charlie_stage.lua#L453-L467) |
| `yoth_knighthat`（镀金骑士头盔） | 630×0.9 | `metal`/`hardarmor`/`waterproofer` 标签＋waterproofer 小；onequip/onunequip 走通用换装；setbonus YOTH_KNIGHT（与 body 甲 armor_yoth_knight 同名互认，head+body 同名才启用）：启用加 `luckysource` 标签＋owner 运 0.2（YOTH 事件期 1.0）并对背包每件 luckitem 推 `updateownerluck` 重算，禁用反向移除 | 最后一骑 knight_yoth 100% 掉落（余骑 fled 标记防重复掉落）；DeconstructRecipe 金4＋红色羽毛1 | [构造](../../../raw/current/scripts/prefabs/hats.lua#L6446-L6507)、[掉落](../../../raw/current/scripts/prefabs/knight.lua#L262-L288)、[套装伙伴](../../../raw/current/scripts/prefabs/armor_yoth_knight.lua#L78-L81)、[tuning#L9279-L9280](../../../raw/current/scripts/tuning.lua#L9280-L9281)、[recipes#L1741](../../../raw/current/scripts/recipes.lua#L1741) |

### 无 armor 战斗帽（1 注册）

| 注册名（官方名） | 战斗/防护机制 | 获取 | 证据 |
|---|---|---|---|
| `alterguardianhat`（启迪之冠） | 无 armor：`open_top_hat`+`gestaltprotection` 标签；持有者理智 >85%（`SANITY_BECOME_ENLIGHTENED_THRESH`）激活（前后层＋`alterguardianhatlight` 帽灯；佩戴初查用 `sanity:GetPercent`、`sanitydelta` 复查用 `GetPercentWithPenalty`，两者口径不同）；仅激活且 `onhitother` 的 `data.from_doattack` 时生成 **`alterguardianhat_projectile` 攻击弹体**（不是 petleash 宠物、不是 `gestalt_guard_evolved`）：目标需排除 `gestalt_guard_evolved` 与 `structure`/`wall`/`balloon`/`smashable`/`deck_of_cards` 标签，弹体 `follower:SetLeader(owner)` 归属，另扣 1 理智（overtime 不发声）；`container` 装孢子（`preserver` 不腐）动态调色；月种满时诱导 lunacy | DeconstructRecipe 残片5（成型途径未在本页展开） | [构造](../../../raw/current/scripts/prefabs/hats.lua#L3671-L3703)、[装备/卸下](../../../raw/current/scripts/prefabs/hats.lua#L3571-L3642)、[recipes#L1672](../../../raw/current/scripts/recipes.lua#L1672) |

### 分派出页（如实分派，不进本页）

| 注册名（官方名） | 归属与依据 |
|---|---|
| `icehat`（冰帽） | 环境页：无 armor；heater 降温＋夏季隔热＋装备潮湿度＋减速（[hats#L1659-L1712](../../../raw/current/scripts/prefabs/hats.lua#L1659-L1712)） |
| `walrushat`（贝雷帽） | 环境页：无 armor；DAPPERNESS_LARGE＋保暖（[hats#L654-L675](../../../raw/current/scripts/prefabs/hats.lua#L654-L675)） |
| `mermhat`（聪明的伪装） | 环境页：无 armor；`merm`/`mermdisguise` 伪装＋鱼人跟随（[hats#L2918-L3004](../../../raw/current/scripts/prefabs/hats.lua#L2918-L3004)）；与 spiderhat 同型伪装，两者均已裁决归环境页伪装族 |
| `antlionhat`（刮地皮头盔） | 环境页：无 armor；`autoterraformer` 地形改造＋装地皮容器（[hats#L2371-L2446](../../../raw/current/scripts/prefabs/hats.lua#L2371-L2446)） |
| `spiderhat`（蜘蛛帽） | 归环境页（伪装族）：无 armor；`spiderdisguise` 伪装＋蜘蛛跟随＋fueled 耗尽即毁（[伪装链](../../../raw/current/scripts/prefabs/hats.lua#L789-L845)、[构造](../../../raw/current/scripts/prefabs/hats.lua#L872-L900)），`spiderdisguise` 标签消费端（[spider.lua#L209](../../../raw/current/scripts/prefabs/spider.lua#L209)、[player_common.lua#L27](../../../raw/current/scripts/prefabs/player_common.lua#L27)、[SGwilson.lua#L260](../../../raw/current/scripts/stategraphs/SGwilson.lua#L260)）随页移出本表 |
| `bat_bosscorpsehat`（蝙蝠帽） | 归环境页：无 armor；perishable 腐坏成 spoiled_food＋waterproofer SMALL＋dapperness＝CRAZINESS_MED＝−0.333 理智/秒＋`bathat` 套装标记（[标签](../../../raw/current/scripts/prefabs/hats.lua#L6637-L6651)、[bat_boss#L47-L52](../../../raw/current/scripts/prefabs/bat_boss.lua#L47-L52)），近战吸血与 bathat 套装机制归环境页说明，`mufflehat`/`bathat` 消费端随页移出本表 |
| `dragonheadhat`/`dragonbodyhat`/`dragontailhat`（幸运兽头/躯/尾） | 事件页：无 armor；舞龙组队玩法（[hats#L2092-L2207](../../../raw/current/scripts/prefabs/hats.lua#L2092-L2206)） |
| 其余 `mask_*`（玩偶/铁匠/镜子/女王/国王/树木/小丑/三远古面具等 10 顶） | 事件页：无 armor；`mask_common` 装扮基座＋舞台剧/远古阅读（[hats#L2704-L2770](../../../raw/current/scripts/prefabs/hats.lua#L2704-L2770)） |
| `pumpkinhat`（无面南瓜） | 事件页：无 armor；雕刻脸玩法，FX 层已归 [[reference/prefabs/hats]] |

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| `组件：armor` | `InitCondition(耐久, 吸收)`；`SetTags` 限定可挡的攻击者标签；`AddWeakness` 给特定攻击者标签加 armor 额外磨损（woodcarvedhat beaver）；`TakeDamage` 只扣 `ApplyDamage` 分摊到该件的量（`absorbed_damage × 本件吸收率 / 各件吸收率之和`＋本件弱点）并乘 `conditionlossmultipliers`、推 `armordamaged`；`ontakedamage` 回调（dreadstone regen、ruins proc 转理智）；`Repair`（eyemask 吃食、dreadstone regen） | 24 个 armor 帽全部主模拟挂载；皮肤不影响组件 | [[reference/components/armor]]；[CanResist/GetAbsorption](../../../raw/current/scripts/components/armor.lua#L126-L141)、[弱点结算](../../../raw/current/scripts/components/inventory.lua#L451)、[TakeDamage](../../../raw/current/scripts/components/armor.lua#L156-L164) |
| `组件：resistance` | `AddResistance(tag)` 登记可挡的攻击者/武器标签；`Inventory:ApplyDamage` 装备槽循环里命中即 `ResistDamage` 并直接返回 0 伤害，`OnResistDamageFn` 自行决定转嫁（woodcarvedhat 转 armor 耐久） | 仅 woodcarvedhat；quakedebris/lunarhaildebris 碎片落地 `GetAttacked` 是主要伤害源（重物 30/普通 20） | [[reference/components/resistance]]；[ApplyDamage 检查](../../../raw/current/scripts/components/inventory.lua#L408-L420)、[碎片攻击](../../../raw/current/scripts/components/quaker.lua#L214-L233)、[登记](../../../raw/current/scripts/prefabs/hats.lua#L409-L412) |
| `组件：planardefense` | `SetBaseDefense` 固定位面减伤；`AddBonus` 供技能树动态加成 | wathgrithr_improved/dreadstone/lunarplant/voidcloth/wagpunk 五顶 | [[reference/components/planardefense]]；[equippable 加成](../../../raw/current/scripts/prefabs/hats.lua#L1442-L1447) |
| `组件：damagetyperesist` | `AddResist("shadow_aligned"/"lunar_aligned", inst, 值[, "setbonus"])` 按 key 增删 | dreadstone/lunarplant/voidcloth 三顶；套装通道 key 为 "setbonus" | [[reference/components/damagetyperesist]]；[lunarplant#L3913-L3919](../../../raw/current/scripts/prefabs/hats.lua#L3913-L3919) |
| `组件：setbonus` | `SetSetName(EQUIPMENTSETNAMES.*)`＋启用/禁用回调；启用条件是 HEAD+BODY 两件同名（`UpdateSetBonus` 由 inventory Equip/Unequip 调） | DREADSTONE/LUNARPLANT/VOIDCLOTH/YOTH_PRINCESS/YOTH_KNIGHT | [[reference/components/setbonus]]；[同名判定](../../../raw/current/scripts/components/setbonus.lua#L52-L64)、[dreadstone#L3808-L3809](../../../raw/current/scripts/prefabs/hats.lua#L3808-L3809) |
| `组件：equippable` | 全家 HEAD 槽；`restrictedtag` 装备限制（仅对 player/possessedbody 判定）；`insulated` 防雷击；`dapperness`/`dapperfn` 理智 | 全部 25 注册；mermarmor 两顶 restrictedtag 生效 | [[reference/components/equippable]]；[IsRestricted](../../../raw/current/scripts/components/equippable.lua#L147-L158)、[IsInsulated](../../../raw/current/scripts/components/equippable.lua#L64-L66)、[雷电消费](../../../raw/current/scripts/components/playerlightningtarget.lua#L5)、[装备拒绝](../../../raw/current/scripts/components/inventory.lua#L1270) |
| `组件：sanity` | `neg_aura_absorb`（hivehat 负光环吸收）；`SetInducedInsanity`（skeletonhat） | 装备期间设置、卸下还原 | [[reference/components/sanity]]；[sanity#L570-L582](../../../raw/current/scripts/components/sanity.lua#L570-L582)、[skeleton#L2858-L2870](../../../raw/current/scripts/prefabs/hats.lua#L2858-L2870) |
| `组件：luckuser` | 套装启用回调给 owner `SetLuckSource(运值, 帽)`、禁用/卸帽 `RemoveLuckSource`；同时给背包 luckitem 推 `updateownerluck` 重算 | YOTH 两顶；princess −1（unluckysource 标签）、knight 0.2（事件期 1.0，luckysource 标签） | [[reference/components/luckuser]]；[SetLuckSource](../../../raw/current/scripts/components/luckuser.lua#L38-L52)、[knight 回调](../../../raw/current/scripts/prefabs/hats.lua#L6467-L6483) |
| `组件：shadowlevel`／`shadowdominance` | 阴影等级与支配组件挂载 | ruinshat/skeletonhat/dreadstonehat/voidclothhat | [[reference/components/shadowlevel]]、[[reference/components/shadowdominance]] |
| `组件：eater` | 吃食修 armor（按食物 health/hunger ×吸收倍率） | 仅 eyemaskhat | [[reference/components/armor]] 的 Repair；[eyemask_oneatfn](../../../raw/current/scripts/prefabs/hats.lua#L2329-L2340) |
| `事件/数据："attacked"` proc 协议 | 监听持有者 `attacked`，`ShouldProcOnAttackedOrBlocked` 排除自己攻击/重定向，`TryLuckRoll`＋`LuckFormulas.RuinsHatProc` 决定 proc；三者都是 `componentutil.lua` 定义的**全局**函数／表（该文件不 return 模块表），mod 里走 `GLOBAL.` 前缀 | ruinshat；voidcloth **单戴即**生效：`onattackother` 给 HANDS 槽符合条件的武器加层，非自身装备来源的 `attacked` 清层（与套装无关，套装回调只改 `shadow_aligned` 抗性） | [判定](../../../raw/current/scripts/componentutil.lua#L114-L132)、[运表](../../../raw/current/scripts/componentutil.lua#L2129)、[TryLuckRoll](../../../raw/current/scripts/componentutil.lua#L2237) |
| `事件/数据：skilltreeupdater` | 监听 `onactivateskill_server`/`ondeactivateskill_server` 重算 armor 损耗倍率与 planar bonus | wathgrithr 两顶（装备时挂监听、卸下解绑） | [[reference/components/skilltreeupdater]]；[hats#L1430-L1485](../../../raw/current/scripts/prefabs/hats.lua#L1430-L1485) |
| `Replica／同步：wagpunkhat` | net_entity wearer/target＋`wagpunkhat_classified` 说话代理；非专服监听 dirty 事件 | 仅 wagpunkhat；FX 等级 netvar 归 hats.md | [[reference/prefabs/hats]]；[netvar](../../../raw/current/scripts/prefabs/hats.lua#L4618-L4627) |
| `关联 Prefab：forcefieldfx` | ruinshat proc 挂 owner 显示护盾 | proc 窗口期 | [proc](../../../raw/current/scripts/prefabs/hats.lua#L455-L476) |
| `关联 Prefab：knight_yoth（镀金骑士）` | mask_princesshat 的 petleash 宠物（MakeFriendly/MakeHostile 切换、horseman_type 唯一名）；yoth_knighthat 的掉落源（最后一骑 100%） | YOTH 事件期；manager 监听 `ms_register_yoth_princess` | [掉落](../../../raw/current/scripts/prefabs/knight.lua#L262-L288)、[骑士监听](../../../raw/current/scripts/prefabs/knight.lua#L604) |
| `关联 Prefab：*hat_fx 跟随层` | lunarplanthat/voidcloth/wagpunk/shadow_thrall_parasite 四顶的跟随层；装备生成、卸下 Remove | 层体端模型归 FX 族页 | [[reference/prefabs/hats]] |
| `关联 Prefab：armorsnurtleshell` | slurtlehat onequip 检查 BODY 组合成就 | 仅成就判定，无属性联动 | [slurtle_equip](../../../raw/current/scripts/prefabs/hats.lua#L1231-L1242) |
| `关联 Prefab：wagpunksteam_hat_up/down` | wagpunkhat 阶段蒸汽表现 | 随阶段生成 | [[reference/prefabs/wagpunksteam_hat_up|WAGpunk 装备蒸汽]] |

### 消费端标签（组件/AI 实读）

| 标签 | 写入者 | 消费端与效果 | 证据 |
|---|---|---|---|
| `hardarmor` | ruinshat、dreadstonehat、wagpunkhat、mask_princesshat、yoth_knighthat | Wagdrone 滚动冲撞按 `EquipHasTag("hardarmor")` 走减免分支 | [SGwagdrone_rolling.lua#L324](../../../raw/current/scripts/stategraphs/SGwagdrone_rolling.lua#L324) |
| `heavyarmor` | wathgrithr_improvedhat | AoE 击退按 `ArmorHasTag` 用重甲强度；commonstates 冻结判定 `EquipHasTag` | [aoeutil.lua#L131](../../../raw/current/scripts/aoeutil.lua#L131)、[commonstates.lua#L3119](../../../raw/current/scripts/stategraphs/commonstates.lua#L3119) |
| `goggles` | lunarplanthat | `gogglevision`（风沙护目视觉切换） | [playervision.lua#L50](../../../raw/current/scripts/components/playervision.lua#L50) |
| `gestaltprotection` | lunarplanthat、alterguardianhat | `IsEntityGestaltProtected`（启迪态占有豁免） | [componentutil.lua#L1996-L2000](../../../raw/current/scripts/componentutil.lua#L1996-L2000) |
| `wood` | woodcarvedhat | `GetArmorImpactSound` 命中音效选 wood_armour_ | [componentutil.lua#L1624-L1644](../../../raw/current/scripts/componentutil.lua#L1624-L1644) |
| `luckysource`／`unluckysource` | yoth_knighthat（luckysource）、mask_princesshat（unluckysource），套装启用时挂 | itemtile 幸运/厄运特效播放判定；实际运值走 luckuser | [itemtile.lua#L923-L925](../../../raw/current/scripts/widgets/itemtile.lua#L923-L925)、[luckuser#L38-L52](../../../raw/current/scripts/components/luckuser.lua#L38-L52) |
| `shadowthrall_parasite_mask` | 三顶宿主面具 | 寄生虫 manager 选宿主时排除已标记目标 | [shadowparasitemanager.lua#L174](../../../raw/current/scripts/components/shadowparasitemanager.lua#L174) |
| `battlehelm` | wathgrithr 两顶 | scripts 域 grep 未见消费者（仅本文件写入）；消费方可能在 scrapbook/引擎层，未核验 | [写入](../../../raw/current/scripts/prefabs/hats.lua#L1423-L1428) |

## 典型接入

### 给自定义头盔复刻铥矿皇冠式受击 proc

- 前提与选择：modmain.lua 注册自定义帽后，在帽实体上挂 armor 并监听持有者 `attacked`；proc 判定直接复用 `componentutil.lua` 的 helper，不要手写受击判定——原版排除了自身装备伤害与重定向。注意该文件把这些函数与 `LuckFormulas` 表**定义成全局**（没有 `return` 工具表），mod 里只能 `GLOBAL.ShouldProcOnAttackedOrBlocked` / `GLOBAL.TryLuckRoll` / `GLOBAL.LuckFormulas` / `GLOBAL.IsRangedWeapon`，`require` 到的是空值。
- 生效与撤销：proc 期全额吸收＋伤害转理智可选；冷却用帽实体任务句柄防重入；卸下回调必须解绑监听、取消任务，并**成对复位 `forcefield` 标签与进入 proc 前的吸收率**——原版 `ruins_onunequip → inst.ondetach() → ruinshat_unproc` 就是这套收尾，只复位一半会让帽重新戴出 100% 吸收。示例直接覆盖 `SetOnEquip`/`SetOnUnequip`，只适用于自建帽；若改原版帽的回调，必须保留原装配回调或自行补全复位。
- 原版依据：[tryproc](../../../raw/current/scripts/prefabs/hats.lua#L478-L484)、[ruinshat_proc](../../../raw/current/scripts/prefabs/hats.lua#L455-L476)、[onattach/ondetach](../../../raw/current/scripts/prefabs/hats.lua#L533-L552)。

```lua
-- modmain.lua：自定义头盔受击 proc（铥矿皇冠模式，数值可替换）
-- mod 环境白名单仅注入 TUNING 等（mods.lua#L258-L317 无 __index=_G）：
-- TheWorld／EQUIPSLOTS 等游戏全局一律 GLOBAL. 前缀；TUNING 直用。
-- componentutil 的 helper 是全局函数（该文件不 return 模块表），只能 GLOBAL. 访问。

local function endproc(inst)
    inst:RemoveTag("forcefield")
    inst.components.armor:SetAbsorption(inst._absorption)  -- 复位进入 proc 前的吸收率
    if inst._proc_task ~= nil then
        inst._proc_task:Cancel()
        inst._proc_task = nil
    end
end

local function startproc(inst)
    inst:AddTag("forcefield")
    inst.components.armor:SetAbsorption(1)                 -- proc 期全额吸收（原版 TUNING.FULL_ABSORPTION）
    if inst._proc_task ~= nil then inst._proc_task:Cancel() end
    inst._proc_task = inst:DoTaskInTime(4, function()      -- 持续 4 秒
        inst.components.armor:SetAbsorption(inst._absorption)
        inst:RemoveTag("forcefield")
        inst._proc_task = inst:DoTaskInTime(5, function()  -- 冷却 5 秒
            inst._proc_task = nil
        end)
    end)
end

AddPrefabPostInit("mycrown", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    if inst.components.armor == nil then
        inst:AddComponent("armor")
        inst.components.armor:InitCondition(TUNING.ARMOR_FOOTBALLHAT, TUNING.ARMOR_FOOTBALLHAT_ABSORPTION)
    end
    inst._absorption = inst.components.armor.absorb_percent  -- 原版 ruinshat_proc 记住进入前的吸收率
    inst.components.equippable:SetOnEquip(function(inst, owner)
        inst._absorption = inst.components.armor.absorb_percent
        inst._onattacked = function(owner, data)
            if inst._proc_task == nil
                and GLOBAL.ShouldProcOnAttackedOrBlocked(inst, owner, data)
                and GLOBAL.TryLuckRoll(owner, TUNING.ARMOR_RUINSHAT_PROC_CHANCE, GLOBAL.LuckFormulas.RuinsHatProc) then
                startproc(inst)
            end
        end
        inst:ListenForEvent("attacked", inst._onattacked, owner)
    end)
    inst.components.equippable:SetOnUnequip(function(inst, owner)
        if inst._onattacked ~= nil then
            inst:RemoveEventCallback("attacked", inst._onattacked, owner)
            inst._onattacked = nil
        end
        endproc(inst)  -- 成对复位：forcefield 标签＋吸收率（原版 ondetach → ruinshat_unproc）
    end)
end)
```

- 调整与验证：替换点为 prefab 名、`InitCondition` 数值与 proc 参数；`ShouldProcOnAttackedOrBlocked` 需要 `attacked`/`blocked` 事件 data，勿在其他事件复用；`inst._absorption` 必须在覆盖回调前取到，否则复位值不可知；示例只做语法检查，未在游戏内执行。

### 限制装备对象（NPC 帽模式）

- 前提与选择：给 NPC 专属装备（如鱼人守卫盔）时用 `equippable.restrictedtag`：`inst.components.equippable.restrictedtag = "merm_npc"`。限制只对玩家/possessedbody 生效（[IsRestricted](../../../raw/current/scripts/components/equippable.lua#L147-L158) 对其他实体恒 false），NPC 拾取走 inventory 的 GiveItem/Equip 检查（[inventory.lua#L1270](../../../raw/current/scripts/components/inventory.lua#L1270)）。
- 生效与撤销：装备时拒绝是即时判定的，无运行时副作用；NPC 侧用同类标签拾取（[mermguardbrain.lua#L44](../../../raw/current/scripts/brains/mermguardbrain.lua#L44)）。
- 原版依据：[mermarmor](../../../raw/current/scripts/prefabs/hats.lua#L5474-L5508)。

## 常见陷阱与限制

- **误区：`beehat` 是通用盔。** `armor:SetTags({"bee"})` 后 `CanResist` 只认 bee 标签的攻击者或武器，其他来源伤害完全穿透（吸收率不参与）；做通用防护帽不要抄这一行。armor 组件 tags 为 nil 才是"全挡"。[CanResist](../../../raw/current/scripts/components/armor.lua#L126-L141)、[写入](../../../raw/current/scripts/prefabs/hats.lua#L292-L294)
- **`eyemaskhat` 与 `footballhat` 共用 `TUNING.ARMOR_FOOTBALLHAT`。** 改这组调参同时改两顶帽；eyemaskhat 的修复走 eater 吸收倍率（4.0/1.75）而非修理工具。[复用](../../../raw/current/scripts/prefabs/hats.lua#L2360-L2361)、[吃食修复](../../../raw/current/scripts/prefabs/hats.lua#L2329-L2340)
- **armor 耐久扣的是分摊后的吸收量，不是原始来伤。** `Inventory:ApplyDamage` 先乘伤害类型抗性，再取各装备最高有效吸收率算出总吸收量，按各自吸收率占比分给每件，另加该件弱点（`GetBonusDamage`）与它吸收的特殊伤害，`Armor:TakeDamage` 才乘该件 `conditionlossmultipliers` 并扣 `condition`。所以「与吸收率无关」是错的：吸收率决定这件分到多少。ruinshat proc 期 100% 吸收保护的是生命值，帽耐久仍按分到该件的量消耗并推 `armordamaged`。[分摊](../../../raw/current/scripts/components/inventory.lua#L408-L500)、[TakeDamage](../../../raw/current/scripts/components/armor.lua#L156-L164)、[proc 伤害转理智](../../../raw/current/scripts/prefabs/hats.lua#L466-L470)
- **联机耐久是脚本注释单机值的 0.7 倍。** `multiplayer_armor_durability_modifier = 0.7` 写进所有 armor 配置式；吸收乘数当前为 1（0.75 被注释未启用）。跨版本换算勿用旧单机数值。[tuning.lua#L33-L38](../../../raw/current/scripts/tuning.lua#L33-L38)
- **`mermarmor` 两顶玩家装不上。** `restrictedtag = "merm_npc"` 且玩家无该标签 → `IsRestricted` 拒绝装备；给玩家用需清 restrictedtag 或自建副本。反向利用：给 NPC 专属装备可直接抄。[限制](../../../raw/current/scripts/prefabs/hats.lua#L5505)、[IsRestricted](../../../raw/current/scripts/components/equippable.lua#L147-L158)
- **YOTH 套装单件不生效。** `setbonus` 启用判定要求 HEAD+BODY 两件都带同名 setbonus（[同名判定](../../../raw/current/scripts/components/setbonus.lua#L52-L64)），只戴 yoth_knighthat 或只穿 armor_yoth_knight 都不会触发套装回调；做单件特效不要复用 setbonus。[套装协议](../../../raw/current/scripts/components/setbonus.lua#L67-L76)
- **`shadow_thrall_parasitehat` 卸下即杀宿主。** 卸下/移除链对宿主 `health:Kill()`（`set_to_remove_owner` 时连宿主实体一起 Remove）；移植或做实验先看清 `noloot`/`remove_self` 分支。[卸下链](../../../raw/current/scripts/prefabs/hats.lua#L5745-L5763)
- **`wagpunkhat` 伤害倍率以帽实体为 modifier key。** 装备即设 STAGE0（1.05），阶段升档覆盖；卸帽 `RemoveModifier(inst)` 还原——直接复制阶段代码而漏掉卸帽移除会永久锁倍率。[升档](../../../raw/current/scripts/prefabs/hats.lua#L4379-L4443)、[卸帽还原](../../../raw/current/scripts/prefabs/hats.lua#L4476-L4510)
- **全头盔（fullhelm）会隐藏头部符号。** `fullhelm_onequip` 隐藏 face/swap_face/beard/cheeks 并 `UseHeadHatExchange(true)`，影响角色胡须/脸部表现；卸帽 `fullhelm_onunequip` 成对还原。[换装三路](../../../raw/current/scripts/prefabs/hats.lua#L130-L171)
- **皮肤兼容必须走 `_base_onequip` 协议。** 直接 `OverrideSymbol` 会绕过 `GetSkinBuild`/`equipskinneditem` 事件，导致皮肤帽错图；`_skinfns` 表还暴露给皮肤系统复用。[协议](../../../raw/current/scripts/prefabs/hats.lua#L31-L47)、[skinfns](../../../raw/current/scripts/prefabs/hats.lua#L179-L186)
- **modmain 全局前缀。** mod 环境（[mods.lua#L258-L317](../../../raw/current/scripts/mods.lua#L258-L317)）无 `__index=_G`，`TheWorld`/`EQUIPSLOTS`/`SpawnPrefab` 等一律 `GLOBAL.` 前缀，`TUNING` 直用；`AddPrefabPostInit` 等注册函数由 modutil 注入、无前缀。

## 验证记录

- 源码审阅：2026-11-07，steam-buildid:26335394（实读 `raw/current/BUILD.txt`）。全文枚举 `prefabs/hats.lua` 的 `MakeHat` 注册（grep 计数 85）并逐顶实读本页 25 注册与剩余 dispatch 候选（ice/walrus/merm/antlion/dragon 三顶/masks/pumpkin）的组件段；工厂骨架、换装三路与皮肤协议复用 [[reference/prefabs/hats]] 同批核验。
- 主代理裁决执行：裁决一将 `spiderhat`（蜘蛛伪装＋蜘蛛跟随）、`bat_bosscorpsehat`（perishable 腐坏＋waterproofer SMALL＋dapperness＝CRAZINESS_MED＝−0.333/秒＋bathat 套装标记，均无 armor）移出本页归 [[reference/prefabs/hats-environment|环境页]]（[分派出页](#分派出页如实分派不进本页)留去向与互链注，spiderdisguise/mufflehat/bathat 消费端行随移）；裁决二补入 `woodcarvedhat`/`mask_princesshat`/`yoth_knighthat` 三注册，页内注册数 24→25。
- armor 分派复核：hats.lua `AddComponent("armor")` 本页 21 处逐行实读（L292/L379/L414/L519/L1217/L1251/L1496/L1525/L2080/L2360/L2751/L2903/L3787/L3901/L4124/L4647/L4735/L5500/L5538/L6413/L6492，其中 L414 woodcarved、L6413 princess、L6492 knight 为本次补入，L2751 为 shadow_thrall_parasitehat 与三顶面具共用 fn）；`resistance` 1 处（L409）、`planardefense` 5 处、`damagetyperesist` 3 处、`setbonus` 5 处（含 YOTH 两顶 L6432/L6501）实读。
- 名称核验：`languages/chinese_s.po`（Python UTF-8 逐键读取 msgctxt `STRINGS.NAMES.*`）：FOOTBALLHAT＝橄榄球头盔（[L342952](../../../raw/current/scripts/languages/chinese_s.po#L342949-L342952)）、RUINSHAT＝铥矿皇冠（L348817）、SLURTLEHAT＝背壳头盔（L349647）、SKELETONHAT＝骨头头盔（L349422）、BEEHAT＝养蜂帽（L339322）、HIVEHAT＝蜂王冠（L343932）、EYEMASKHAT＝眼面具（L342507）、WATHGRITHRHAT＝战斗头盔（L351882）、WATHGRITHR_IMPROVEDHAT＝统帅头盔（L351887）、COOKIECUTTERHAT＝饼干切割机帽子（L341657）、DREADSTONEHAT＝绝望石头盔（L342267）、LUNARPLANTHAT＝亮茄头盔（L344867）、VOIDCLOTHHAT＝虚空风帽（L351347）、WAGPUNKHAT＝W.A.R.B.I.S.头戴齿轮（L351447）、SCRAPHAT＝拾荒尖帽（L348997）、MERMARMORHAT＝沼泽斗士头盔（L345302）、MERMARMORUPGRADEDHAT＝沼泽战士头盔（L345307）、MASK_HALFWITHAT＝蠢人面具（L345117）、MASK_SAGEHAT＝贤者面具（L345142）、MASK_TOADYHAT＝马屁精面具（L345147）、WOODCARVEDHAT＝硬木帽（L352392）、MASK_PRINCESSHAT＝公主面具（L345132）、YOTH_KNIGHTHAT＝镀金骑士头盔（L353012）、ALTERGUARDIANHAT＝启迪之冠（L338517）。`shadow_thrall_parasitehat` 无 NAMES 键：帽实体 `displaynamefn` 对暗域阵营玩家返回 `SHADOWTHRALL_PARASITE_ALLEGIANCE`＝"暗域先驱"（[L349227](../../../raw/current/scripts/languages/chinese_s.po#L349227)，非 aligned 玩家的显示名回落引擎层未核）；分派帽 ICEHAT＝冰帽、WALRUSHAT＝贝雷帽、MERMHAT＝聪明的伪装、ANTLIONHAT＝刮地皮头盔一并核验（KNIGHT_YOTH＝镀金骑士 L344322、FEATHER_ROBIN＝红色羽毛 L342682 作配方/宠物名辅助核验）。
- 配方核验：`recipes.lua` 逐行——Recipe2：woodcarvedhat L174（builder_skill woodie_human_lucy_2＋sg_state carvewood，Ingredient("lucy",0) 仅要求持有露西不消耗，[carvewood 状态](../../../raw/current/scripts/stategraphs/SGwilson.lua#L8536-L8579)、[技能点](../../../raw/current/scripts/prefabs/skilltree_woodie.lua#L375-L380)）、wathgrithrhat L179、wathgrithr_improvedhat L193（builder_skill）、dreadstonehat L503（TECH.LOST）、footballhat L504、cookiecutterhat L505、beehat L543、ruinshat L807、lunarplanthat L849（lunar_forge）、wagpunkhat L861、voidclothhat L867（shadow_forge）；DeconstructRecipe：alterguardianhat L1672、hivehat L1673、skeletonhat L1676、eyemaskhat L1678、scraphat L1683、mermarmor 两顶 L1696-L1697、mask_princesshat L1738、yoth_knighthat L1741。掉落/发落实读：slurtle.lua#L19-L25（10%）、beequeen.lua#L40、stalker.lua#L1017、eyeofterror.lua#L464、[knight.lua#L262-L288](../../../raw/current/scripts/prefabs/knight.lua#L262-L288)（最后一骑 knight_yoth 100% 掉 yoth_knighthat＋armor_yoth_knight，fled 标记防重复）、[charlie_stage.lua#L453-L467](../../../raw/current/scripts/prefabs/charlie_stage.lua#L453-L467)（YOTH 神龛激活期一次性送达 mask_princesshat）、hats.lua#L5729-L5743（面具 luck 掉落）、shadowparasitemanager.lua#L135-L140（宿主装备）、merm_supply_structures.lua#L473/L485；`scraphat` 唯一产出＝[daywalker2.lua#L751-L763](../../../raw/current/scripts/prefabs/daywalker2.lua#L751-L763) 的撤场 `DropItemAsLoot("tackle")` 与 [SGdaywalker2.lua#L2285-L2298](../../../raw/current/scripts/stategraphs/SGdaywalker2.lua#L2285-L2298) 的击败分支，经 [daywalker2_items.lua#L51-L73](../../../raw/current/scripts/prefabs/daywalker2_items.lua#L51-L73) 的 `spike_loot_fx`（animover 或读档 `OnLoad`）生成、耐久 6%–20%。
- 数值核验：woodcarvedhat 262.5×0.7（[tuning.lua#L1837-L1838](../../../raw/current/scripts/tuning.lua#L1837-L1838)）、BEAVER_WOOD_DAMAGE＝wilson_attack×0.5＝17（[L4653](../../../raw/current/scripts/tuning.lua#L4653)，beaver 标签源自 [woodie.lua#L1068](../../../raw/current/scripts/prefabs/woodie.lua#L1068) 变身切换）、mask_princesshat 105×0.6 与 yoth_knighthat 630×0.9（[L9279-L9286](../../../raw/current/scripts/tuning.lua#L9280-L9287)）、YOTH 运值 −1/0.2/事件期 1（[L9298-L9300](../../../raw/current/scripts/tuning.lua#L9299-L9301)）、YOTH_HORSE_NAMES 4 名（[constants.lua#L3014-L3019](../../../raw/current/scripts/constants.lua#L3014-L3019)）；dreadstonehat 自愈 rate 区间 0.0667%–0.1111%/秒、套装 ×1.5（[tuning.lua#L1885-L1888](../../../raw/current/scripts/tuning.lua#L1885-L1888)）；voidclothhat 叠层 6 层／+24／3 秒衰减（[tuning.lua#L1912-L1914](../../../raw/current/scripts/tuning.lua#L1912-L1914)）。
- 消费端反查：quakedebris/lunarhaildebris 碎片落地 `GetAttacked`（[quaker.lua#L214-L233](../../../raw/current/scripts/components/quaker.lua#L214-L233)）→ 玩家 `combat.lua#L630` 转 `Inventory:ApplyDamage`（[inventory.lua#L408-L420](../../../raw/current/scripts/components/inventory.lua#L408-L420)，resistance 命中即返回 0 伤害）；armor 弱点经 `GetBonusDamage` 计入 armor 耐久损耗（[inventory.lua#L451](../../../raw/current/scripts/components/inventory.lua#L451)）；wood 标签命中音效（[componentutil.lua#L1624-L1644](../../../raw/current/scripts/componentutil.lua#L1624-L1644)）；setbonus 同名判定（[setbonus.lua#L52-L64](../../../raw/current/scripts/components/setbonus.lua#L52-L64)）；luckuser SetLuckSource（[luckuser.lua#L38-L52](../../../raw/current/scripts/components/luckuser.lua#L38-L52)）、luckitem 监听 updateownerluck（[luckitem.lua#L28](../../../raw/current/scripts/components/luckitem.lua#L28)）、luckysource/unluckysource 特效（[itemtile.lua#L923-L925](../../../raw/current/scripts/widgets/itemtile.lua#L923-L925)）、ms_register_yoth_princess（[yoth_knightmanager.lua#L204-L205](../../../raw/current/scripts/components/yoth_knightmanager.lua#L204-L205)、[knight.lua#L604](../../../raw/current/scripts/prefabs/knight.lua#L604)）、hardarmor（SGwagdrone_rolling.lua#L324）、heavyarmor（aoeutil.lua#L131、commonstates.lua#L3119）、goggles（playervision.lua#L50）、gestaltprotection（componentutil.lua#L1996-L2000）、shadowthrall_parasite_mask（shadowparasitemanager.lua#L174）；`battlehelm`/`regal` scripts 域未见消费者。
- 分档说明：`size_class: complex`——25 注册共享工厂但逐帽组件/触发差异大，proc（luck＋冷却＋FX）、套装（setbonus＋damagetyperesist＋武器/运值联动）、宿主标记（parasite 家族＋标签消费端）、落物抵挡（resistance＋quaker 碎片）四条协作链必须整体对照；获取途径横跨配方、Boss 掉落、AI 发放、事件送达四个体系，超出 standard 单页容量。
- 未核验／待办：`mask_princesshat` 送达链只核到 charlie_stage timer 段（hedgehound 携带与玩家拾取路径未展开）；knight_yoth 战斗/反目深度（knightbrain）与 yoth_knightmanager 全量未展开；`shadow_thrall_parasitehat` 家族（含三顶面具）属 Hallowed Nights 引入内容但为常驻机制，如事件页另收会与本页重叠；`alterguardianhat` 的容器孢子/月种/lunacy 深度机制与弹体伤害（`lunarseedbonusbasephysicaldamage`）换算未展开；`battlehelm`/`nosteal`/`regal` 消费端未找到或未核验；`footballhat` 的 `open_top_hat` onequip 分支用途未核验；非 aligned 玩家视角 `shadow_thrall_parasitehat` 显示名回落值未核验。
- 结构检查：2026-11-07 第六轮修订后 `validate_page.py`（含 `--check-indexes`）通过（228 行／53794 字节，complex 档上限 480 行／65536 字节，0 issue）；`validate_source_links.py --strict` 通过（200 条引用、41 个源文件，0 error 0 warning）；全部 Lua 示例经 `parser_probe --syntax-only` 解析通过。
- 游戏运行：未运行；`validation` 保持 `source-reviewed`。

## 相关链接

- [[reference/prefabs/hats|帽子跟随层与帽灯特效族]]（同一 hats.lua 的 FX 附属页：MakeFollowFx 跟随层、帽灯、tophat_container；本页帽本体的 FX 层入口均归它）
- [[reference/components/armor|armor 组件]]（耐久/吸收/受击协议的主要页）
- [[reference/components/equippable|equippable 组件]]（装备槽/ restrictedtag/insulated 的主要页）
- [[reference/components/setbonus|setbonus 组件]] 与 [[reference/components/damagetyperesist|damagetyperesist 组件]]（套装与新系减伤链）
- [[reference/prefabs/wagpunksteam_hat_up|WAGpunk 装备蒸汽]]（wagpunkhat 阶段表现）
- [[reference/prefabs/hats-environment|环境与功能帽族]]（`spiderhat`/`bat_bosscorpsehat` 及其他纯环境功能帽的归属页，伪装/套装机制在彼处展开）与 [[reference/prefabs/hats-event|事件装扮帽族]]（`mask_charlie_bosshat` 等 28 注册）

返回 [[reference/prefabs/wearables-index|穿戴防具与饰品]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-08 build 26474873：本页随版本迁移升版（锚点迁移 9 处），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-11）。
