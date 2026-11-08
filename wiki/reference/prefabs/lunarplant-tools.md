---
title: "亮茄工具族（pickaxe_lunarplant／shovel_lunarplant）"
created: "2026-11-06"
updated: "2026-11-08"
type: prefab
summary: "亮茄镐与亮茄锄铲两注册的同构组装参照：动作工具加武器位面伤害、亮茄头盔套装加成监听、破损循环与修补套件修理、幸运触手协战。"
aliases: ["pickaxe_lunarplant", "shovel_lunarplant", "亮茄粉碎者", "亮茄锄铲", "Brightshade Smasher", "Brightshade Shoevel", "亮茄工具", "月晶工具", "月亮植物工具"]
tags: [prefab, item, equipment]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/pickaxe_lunarplant.lua", "prefabs/shovel_lunarplant.lua", "prefabs/sword_lunarplant.lua", "prefabs/staff_lunarplant.lua", "prefabs/forge_repair_kits.lua", "prefabs/lunar_forge.lua", "prefabs/lunarplanttentacle.lua", "prefabs/skilltree_wormwood.lua", "prefabs/hats.lua", "prefabs/lunarrift_crystals.lua", "components/lunarplant_tentacle_weapon.lua", "components/planardamage.lua", "components/damagetypebonus.lua", "components/spdamageutil.lua", "components/combat.lua", "components/weapon.lua", "components/tool.lua", "components/equippable.lua", "components/finiteuses.lua", "components/forgerepairable.lua", "components/forgerepair.lua", "components/farmtiller.lua", "components/workable.lua", "components/setbonus.lua", "standardcomponents.lua", "constants.lua", "tuning.lua", "recipes.lua", "componentactions.lua", "actions.lua", "componentutil.lua", "widgets/itemtile.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 亮茄工具族（pickaxe_lunarplant／shovel_lunarplant）

## 快速判断

- 职责与适用：两注册均为"可拾取＋手部装备＋工具＋武器"四合一手持装备，构成亮茄（Brightshade）装备族的工具件：`pickaxe_lunarplant` 登记 HAMMER＋MINE 双动作并 EnableToughWork，`shovel_lunarplant` 登记 DIG 动作、TILL 固有动作与 farmtiller。两文件结构同构：套装加成、破损循环、Forge 修理、触手协战共用同一套回调，仅动作、消耗与数值不同。[pickaxe 主模拟段](../../../raw/current/scripts/prefabs/pickaxe_lunarplant.lua#L192-L218)、[shovel 主模拟段](../../../raw/current/scripts/prefabs/shovel_lunarplant.lua#L191-L217)
- 优先使用：借"耗尽→破损→修补套件复原"循环直接用 MakeForgeRepairable，协议见 [[reference/components/forgerepairable#关键流程]]；手工套装监听的复刻见下方典型接入；触手协战按 [[reference/components/lunarplant_tentacle_weapon#典型接入]]。
- 重要副作用：耐久耗尽不删除实体，而是移除 equippable／tool／weapon 并加 `broken` 标签（`isbroken` net_bool 同步客户端 floater 外观）；佩戴亮茄头盔持握时武器伤害×1.1、位面伤害＋5；武器攻击每次也扣 1 耐久。[OnBroken](../../../raw/current/scripts/prefabs/pickaxe_lunarplant.lua#L136-L144)、[套装加成](../../../raw/current/scripts/prefabs/pickaxe_lunarplant.lua#L11-L27)
- 修改边界：组件配置全在服务端，公共端仅标签、floater 与 net 同步；onequip 有皮肤分支，但本 build 的 skins_defs_data 与 prefabskin 均未登记这两注册的皮肤条目；同族武器 sword_lunarplant／staff_lunarplant 与帽本体 lunarplanthat 的机制不在本页展开（关联导航留源码入口）。[端分界](../../../raw/current/scripts/prefabs/pickaxe_lunarplant.lua#L184-L189)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `pickaxe_lunarplant` | `Prefab("pickaxe_lunarplant", fn, assets, prefabs)`：fn 内公共端建实体／动画／标签／floater／`isbroken` net_bool → SetPristine → 主模拟 finiteuses、planardamage、damagetypebonus、inspectable、inventoryitem、SetupComponents、lunarplant_tentacle_weapon、MakeForgeRepairable、MakeHauntableLaunch | 工具挂 HAMMER＋MINE，效率同 `PICKAXE_LUNARPLANT_EFFICIENCY`（4/3）＋EnableToughWork；消耗 HAMMER 1、MINE `HAMMER_USES/PICKAXE_USES`（75/33≈2.27）；600 用；武器伤害 32.5、位面 10。[工具](../../../raw/current/scripts/prefabs/pickaxe_lunarplant.lua#L93-L96)、[消耗](../../../raw/current/scripts/prefabs/pickaxe_lunarplant.lua#L193-L197)、[数值](../../../raw/current/scripts/prefabs/pickaxe_lunarplant.lua#L200-L206) |
| `shovel_lunarplant` | 同构（另一文件，同段式初始化，同为四合一） | 工具仅挂 DIG（效率缺省 1），另 `AddInherentAction(ACTIONS.TILL)`＋farmtiller；破损时同步移除 TILL 与 farmtiller；消耗 DIG／TILL 各 1；250 用；武器伤害 17.2、位面 10。[工具](../../../raw/current/scripts/prefabs/shovel_lunarplant.lua#L88-L100)、[消耗](../../../raw/current/scripts/prefabs/shovel_lunarplant.lua#L192-L196)、[破损移除](../../../raw/current/scripts/prefabs/shovel_lunarplant.lua#L102-L108) |

- 名称核验：`STRINGS.NAMES.PICKAXE_LUNARPLANT`＝"亮茄粉碎者"（[简中条目](../../../raw/current/scripts/languages/chinese_s.po#L346854-L346857)）；`STRINGS.NAMES.SHOVEL_LUNARPLANT`＝"亮茄锄铲"（[简中条目](../../../raw/current/scripts/languages/chinese_s.po#L349339-L349342)）。
- 获取：两注册都是月亮锻造台配方——纯粹辉煌×1＋亮茄外壳×2，`TECH.LUNARFORGING_TWO`＋`station_tag="lunar_forge"`＋nounlock（[pickaxe 配方](../../../raw/current/scripts/recipes.lua#L853)、[shovel 配方](../../../raw/current/scripts/recipes.lua#L854)）；科技由 lunar_forge 站 prototyper 树 `LUNARFORGING=2` 就近提供（[站组装](../../../raw/current/scripts/prefabs/lunar_forge.lua#L148-L152)、[树值](../../../raw/current/scripts/tuning.lua#L1448-L1450)）。全库 grep 未发现 SpawnPrefab／掉落／世界生成入口，常规获取只有该配方。
- 同族：`sword_lunarplant`（亮茄剑）复制了本页的 SetBuffOwner／破损循环结构（[同款监听](../../../raw/current/scripts/prefabs/sword_lunarplant.lua#L31-L63)），见 [[reference/prefabs/sword_lunarplant|亮茄剑]]；`staff_lunarplant` 的套装走 setbonus 组件路线（[对照](../../../raw/current/scripts/prefabs/staff_lunarplant.lua#L225)）。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：equippable | `SetOnEquip/SetOnUnequip` 挂 onequip/onunequip（持握换装＋SetBuffOwner）；不改 equipslot，默认 `EQUIPSLOTS.HANDS` | 两注册同；主模拟挂载 | [[reference/components/equippable#关键接口]]；[默认槽位](../../../raw/current/scripts/components/equippable.lua#L29) |
| 组件：tool | pickaxe：`SetAction(HAMMER,4/3)`＋`SetAction(MINE,4/3)`＋`EnableToughWork(true)`；shovel：`SetAction(DIG)`＋TILL 固有动作＋farmtiller | EnableToughWork 使持有者可对 tough workable（月裂水晶等）施工不弹刀 | [[reference/components/tool#关键接口]]、[[reference/components/farmtiller#关键接口]]；[pickaxe 工具](../../../raw/current/scripts/prefabs/pickaxe_lunarplant.lua#L93-L96)、[tough 消费](../../../raw/current/scripts/components/workable.lua#L189-L206)、[水晶弹刀示例](../../../raw/current/scripts/prefabs/lunarrift_crystals.lua#L78-L82) |
| 组件：weapon | `SetDamage(inst.base_damage)`，套装激活时改乘 1.1；attackwear 保持默认 1（每次攻击扣 1 耐久） | GetDamage 时武器侧 damagetypebonus 同乘普通与位面伤害 | [[reference/components/weapon#关键接口]]；[攻击扣耐久](../../../raw/current/scripts/components/weapon.lua#L111-L121)、[伤害与位面同乘](../../../raw/current/scripts/components/weapon.lua#L94-L103) |
| 组件：finiteuses | pickaxe 600 用（HAMMER 1／MINE 75/33）；shovel 250 用（DIG／TILL 各 1）；耗尽触发 MakeForgeRepairable 的 `_onbroken` | current/total 变化经 onfiniteuses 刷新 forgerepairable 可修标志 | [[reference/components/finiteuses#关键接口]]；[pickaxe 消耗](../../../raw/current/scripts/prefabs/pickaxe_lunarplant.lua#L193-L197)、[刷新路径](../../../raw/current/scripts/components/finiteuses.lua#L1-L8) |
| 组件：planardamage | `SetBaseDamage(10)`；套装激活 `AddBonus(+5,"setbonus")`，关闭时移除 | 位面值随 weapon 链以 SpDamage 进入战斗结算 | [[reference/components/planardamage#关键接口]]；[pickaxe 配置](../../../raw/current/scripts/prefabs/pickaxe_lunarplant.lua#L202-L203)、[加成](../../../raw/current/scripts/prefabs/pickaxe_lunarplant.lua#L11-L27)、[位面收集](../../../raw/current/scripts/components/spdamageutil.lua#L22-L29) |
| 组件：damagetypebonus | `AddBonus("shadow_aligned", inst, 1.1)`：目标带 shadow_aligned 标签时武器普通与位面伤害同乘 1.1 | 消费在 Weapon:GetDamage 的 target 标签判断 | [[reference/components/damagetypebonus#关键接口]]；[注册](../../../raw/current/scripts/prefabs/pickaxe_lunarplant.lua#L205-L206)、[消费](../../../raw/current/scripts/components/weapon.lua#L97-L101) |
| 组件：lunarplant_tentacle_weapon | 装备期监听持有者 `onattackother`，命中后按 20%＋幸运刷 `lunarplanttentacle` 协战 | 默认门槛：虫木技能 `wormwood_allegiance_lunar_plant_gear_2` | [[reference/components/lunarplant_tentacle_weapon#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/pickaxe_lunarplant.lua#L213)、[技能节点](../../../raw/current/scripts/prefabs/skilltree_wormwood.lua#L210-L218) |
| 组件：inspectable／inventoryitem／floater | 基础检视、拾取与漂浮；floater 尺寸/换装由 SetIsBroken 按破损态切换 | 破损漂浮换 broken 动画（SWAP_DATA_BROKEN） | [[reference/components/finiteuses#关键接口]]（耗尽）、[破损 floater](../../../raw/current/scripts/prefabs/pickaxe_lunarplant.lua#L111-L134) |
| 同步：isbroken（net_bool） | 服务端 SetIsBroken `set()`；客户端监听 `isbrokendirty` 调 OnIsBrokenDirty 调整 floater 尺寸/偏移/缩放 | 两注册同构，net 名各带自身 prefab 前缀 | [pickaxe net](../../../raw/current/scripts/prefabs/pickaxe_lunarplant.lua#L180-L189)、[shovel 同构](../../../raw/current/scripts/prefabs/shovel_lunarplant.lua#L179-L189) |
| API／数据：MakeForgeRepairable＋FORGEMATERIALS.LUNARPLANT | 注册修理材料 `"lunarplant"`；`_onbroken` 先卸装归还再调 OnBroken；finiteuses 分支接管耗尽回调 | 破损后仍可修理的实体保留由本 helper 支撑 | [[reference/components/forgerepairable#关键流程]]；[helper](../../../raw/current/scripts/standardcomponents.lua#L1851-L1884)、[材料枚举](../../../raw/current/scripts/constants.lua#L1962-L1967) |
| API／数据：REPAIR 修理协议 | 修理物是 `lunarplant_kit`（亮茄修补套件，forgerepair 材料同名）；右键 kit→工具触发 ACTIONS.REPAIR，一次补满并消耗 1 个 kit | 候选动作要求 kit 带 `forgerepair_lunarplant`、工具带 `forgerepairable_lunarplant`（仅 current<total 时出现）；漂浮中不可修 | [[reference/components/forgerepair#关键流程]]；[候选](../../../raw/current/scripts/componentactions.lua#L1279-L1299)、[执行](../../../raw/current/scripts/actions.lua#L1000-L1005)、[补满](../../../raw/current/scripts/components/forgerepair.lua#L35-L45) |
| API／数据：套装判定（手工监听） | 持握时监听持有者 equip/unequip：装备 `lunarplanthat`（HEAD）→启用加成；换/摘 HEAD 帽→关闭；自带初值检查当前头槽 | 只看 `hat.prefab=="lunarplanthat"`，与 setbonus 组件系统的 head+body 配对无关（见陷阱） | [监听对](../../../raw/current/scripts/prefabs/pickaxe_lunarplant.lua#L29-L63)、[setbonus 组件对照](../../../raw/current/scripts/components/setbonus.lua#L52-L64) |
| API／数据：show_broken_ui＋broken 标签 | 公共端预置 show_broken_ui；破损加 broken 标签、inspectable.nameoverride 改检视文案 | 消费在物品栏格：broken 时显示破损底并置零熟度条 | [标签](../../../raw/current/scripts/prefabs/pickaxe_lunarplant.lua#L169-L178)、[OnBroken](../../../raw/current/scripts/prefabs/pickaxe_lunarplant.lua#L136-L144)、[itemtile 消费](../../../raw/current/scripts/widgets/itemtile.lua#L945-L962) |
| API／数据：TUNING | `PICKAXE_LUNARPLANT_USES=600`／`SHOVEL_LUNARPLANT_USES=250`、`PICKAXE_LUNARPLANT_EFFICIENCY=4/3`、伤害 `wilson_attack*1.25-10`／`wilson_attack*0.8-10`（wilson_attack=34）、位面 10、套装三键 1.1／1.1／5 | 服务端读取；改键即改全族 | [次数与效率](../../../raw/current/scripts/tuning.lua#L182-L192)、[伤害](../../../raw/current/scripts/tuning.lua#L384-L396)、[基数](../../../raw/current/scripts/tuning.lua#L31-L37) |
| 关联 Prefab：lunarplanttentacle | 触手协战产物：目标旁生成、SetTarget 攻击；9 秒自动退场，`persists=false` 不存档 | 触手自带 70 伤害＋15 位面 | [生成](../../../raw/current/scripts/components/lunarplant_tentacle_weapon.lua#L59-L82)、[触手实体](../../../raw/current/scripts/prefabs/lunarplanttentacle.lua#L45-L77)；Wiki 待建 |
| 关联 Prefab：lunarplant_kit | 修理包（配方同站：纯粹辉煌×1＋亮茄外壳×1） | 堆叠道具，修理消耗 1 个 | [配方](../../../raw/current/scripts/recipes.lua#L855)、[kit 组装](../../../raw/current/scripts/prefabs/forge_repair_kits.lua#L41-L58)；名称见 [[reference/components/forgerepair#典型接入]] |
| 关联 Prefab：lunarplanthat | 套装帽（官方名"亮茄头盔"，HEAD 槽）；戴帽才触发本页加成 | 帽自身另有 setbonus 组件（月亮抗性）与独立破损循环 | [帽组装](../../../raw/current/scripts/prefabs/hats.lua#L3890-L3926)、[帽名](../../../raw/current/scripts/languages/chinese_s.po#L344864-L344867)；见 [[reference/prefabs/hats-armor|防护与战斗帽族]] |
| 资源：动画 | `anim/pickaxe_lunarplant.zip`／`anim/shovel_lunarplant.zip`，bank/build 同名，破损切 `broken` 动画 | 注册声明并实际使用 | [pickaxe assets](../../../raw/current/scripts/prefabs/pickaxe_lunarplant.lua#L1-L4)、[shovel assets](../../../raw/current/scripts/prefabs/shovel_lunarplant.lua#L1-L4) |

## 典型接入

### 复刻手工套装监听（对照 setbonus 组件系统）

- 前提与选择：放目标模组 `modmain.lua`，服务端；宿主需已带 weapon 组件。原版工具把套装判定写在持有者 equip/unequip 监听里（帽名硬编码 `lunarplanthat`），而不是给工具加 setbonus 组件——想改判定对象或条件时按此结构复刻最直接。
- 生效与撤销：监听随持有者换绑（equipped/unequipped 驱动），卸下工具即关加成；`_bonusenabled` 只存内存，读档后经重新装备重算，无持久化需求。撤取消除 PostInit 即可。

```lua
-- modmain.lua：给自定义手持装备复刻"戴指定头盔→武器增伤"监听（服务端）
AddPrefabPostInit("myweapon", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    inst.base_damage = 20

    local function SetBuffEnabled(enabled)
        if enabled then
            if not inst._bonusenabled then
                inst._bonusenabled = true
                inst.components.weapon:SetDamage(inst.base_damage * 1.5)
            end
        elseif inst._bonusenabled then
            inst._bonusenabled = nil
            inst.components.weapon:SetDamage(inst.base_damage)
        end
    end

    local function SetBuffOwner(owner)
        if inst._owner == owner then
            return
        elseif inst._owner ~= nil then
            inst:RemoveEventCallback("equip", inst._onownerequip, inst._owner)
            inst:RemoveEventCallback("unequip", inst._onownerunequip, inst._owner)
            SetBuffEnabled(false)
        end
        inst._owner = owner
        if owner ~= nil then
            inst._onownerequip = function(_, data)
                if data ~= nil and data.item ~= nil and data.item.prefab == "wathgrithrhat" then
                    SetBuffEnabled(true)
                elseif data ~= nil and data.eslot == GLOBAL.EQUIPSLOTS.HEAD then
                    SetBuffEnabled(false)
                end
            end
            inst._onownerunequip = function(_, data)
                if data ~= nil and data.eslot == GLOBAL.EQUIPSLOTS.HEAD then
                    SetBuffEnabled(false)
                end
            end
            inst:ListenForEvent("equip", inst._onownerequip, owner)
            inst:ListenForEvent("unequip", inst._onownerunequip, owner)

            local hat = owner.components.inventory:GetEquippedItem(GLOBAL.EQUIPSLOTS.HEAD)
            if hat ~= nil and hat.prefab == "wathgrithrhat" then
                SetBuffEnabled(true)
            end
        end
    end

    inst:ListenForEvent("equipped", function(_, data) SetBuffOwner(data.owner) end)
    inst:ListenForEvent("unequipped", function() SetBuffOwner(nil) end)
end)
```

- 调整与验证：原版把"换 HEAD 帽即关闭"与"帽子判名"写进同一回调，复刻时保留两分支，否则同槽换帽会残留加成；若想改原版两注册的套装数值而非结构，直接覆盖 `TUNING.WEAPONS_LUNARPLANT_SETBONUS_DAMAGE_MULT`／`..._SETBONUS_PLANAR_DAMAGE` 即可（加成在每次启用时重读 TUNING）。事件时序见 [equipped 推送](../../../raw/current/scripts/components/equippable.lua#L92-L107)、[原版监听对](../../../raw/current/scripts/prefabs/pickaxe_lunarplant.lua#L29-L63)。

## 常见陷阱与限制

- **误区：600 用＝600 次挖掘**：MINE 每次消耗 `HAMMER_USES/PICKAXE_USES`＝75/33≈2.27，600 用约可挖 264 次、锤 600 次；shovel 的 DIG/TILL 才是每次 1。攻击按 attackwear=1 扣。[消耗](../../../raw/current/scripts/prefabs/pickaxe_lunarplant.lua#L193-L197)、[攻击扣耐久](../../../raw/current/scripts/components/weapon.lua#L111-L121)
- **边界：套装加成不走 setbonus 组件**：本页两注册（与 sword）用手工监听，只判 HEAD 槽帽名，身体护甲不参与；armor_lunarplant／lunarplanthat／staff 上的 setbonus 组件按"头＋身同 setname"配对，只驱动那些装备自身的加成。两套机制并存，别混为一谈。[手工判定](../../../raw/current/scripts/prefabs/pickaxe_lunarplant.lua#L40-L47)、[setbonus 配对](../../../raw/current/scripts/components/setbonus.lua#L52-L64)
- **边界：加成只在工具被持握期间维护**：监听挂在持有者身上，卸下即 SetBuffOwner(nil) 并关加成；放背包/箱子里不会因戴帽而保留状态，重新装备时按当前头槽重算。[解绑](../../../raw/current/scripts/prefabs/pickaxe_lunarplant.lua#L29-L39)
- **边界：破损不消失但完全失能**：与普通镐铲（耗尽即 Remove）不同，本族保留实体但移除 equippable/tool/weapon（shovel 另移除 TILL 与 farmtiller），无法装备、工作、攻击；修复后由 SetupComponents 重建同一批组件。[OnBroken/OnRepaired](../../../raw/current/scripts/prefabs/pickaxe_lunarplant.lua#L136-L154)、[shovel 移除](../../../raw/current/scripts/prefabs/shovel_lunarplant.lua#L102-L108)
- **边界：可修窗口与修理消耗**：只有 `current<total` 才会出现 `forgerepairable_lunarplant` 标签（满耐久时右键 kit 无动作）；一次修理直接补满 100% 并消耗 1 个 kit，且目标漂浮在水面时不可修。[刷新](../../../raw/current/scripts/components/finiteuses.lua#L1-L8)、[补满](../../../raw/current/scripts/components/forgerepair.lua#L35-L45)、[漂浮限制](../../../raw/current/scripts/componentactions.lua#L1280-L1283)
- **边界：触手协战有双重门槛**：需持有者已激活虫木技能 `wormwood_allegiance_lunar_plant_gear_2`，且单次命中过 20%（＋幸运加成）掷点；其他角色持同款武器无触手。细节与修改入口见 [[reference/components/lunarplant_tentacle_weapon#常见陷阱与限制]]。[门槛](../../../raw/current/scripts/components/lunarplant_tentacle_weapon.lua#L1-L4)
- **边界：MINE 消耗联动**：`SetConsumption(MINE, x)` 会由 finiteuses 自动同设 REMOVELUNARBUILDUP 消耗为 x；shovel 的 DIG 无此联动。[联动](../../../raw/current/scripts/components/finiteuses.lua#L27-L34)

## 验证记录

- 源码审阅：2026-11-06，Steam buildid 26335394（实读 raw/current/BUILD.txt）。完整读取两注册 prefab 全文（pickaxe 221 行／shovel 220 行）、lunarplant_tentacle_weapon、planardamage、damagetypebonus、finiteuses、forgerepair、forgerepairable、weapon、tool、equippable、workable、setbonus、farmtiller、spdamageutil 的相关全文或定义段；MakeForgeRepairable／MakeHauntableLaunch／FORGEMATERIALS／TOOLACTIONS／EQUIPSLOTS 定义；tuning 相关键与 PROTOTYPER_TREES.LUNAR_FORGE；recipes 两注册＋kit 配方与 lunar_forge 站组装；componentactions 候选、actions REPAIR 执行、combat 伤害结算、itemtile 破损 UI；TryLuckRoll／ShadowTentacleSpawn／技能节点；hats.lua 的 lunarplanthat 段与 chinese_s.po 五个名称键。破损伤害数值另经 scrapbookdata.lua 的 weapondamage 交叉印证（32.5／17.2）。未覆盖：皮肤清单（数据侧）、战斗结算完整分支、触手 SG 行为、sword/staff/armor 族页机制细节。
- 游戏运行：未运行；未验证实际持握外观、破损/修理的端表现与多人同步时序。
- 结构检查：2026-11-06，validate_page.py 与 validate_source_links.py（--strict）通过。

相关：[[reference/prefabs/index|Prefab 索引]]；[[reference/prefabs/pickaxe|鹤嘴锄（基础镐族参照）]]；[[reference/prefabs/shovel|铁铲（基础铲族参照）]]；[[reference/components/forgerepairable|锻造修理目标侧]]。

返回 [[reference/prefabs/tools-field-index|工具与野外实用]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-08 build 26474873：本页随版本迁移升版（机械沿用；引用区间未受本次源码行号位移影响），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-07）。
