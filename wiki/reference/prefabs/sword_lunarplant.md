---
title: "亮茄剑（sword_lunarplant）"
created: "2026-11-06"
updated: "2026-11-08"
type: prefab
summary: "原版亮茄族纯武器参照：基础＋位面双伤害与暗影目标增伤、亮茄头盔手工套装监听、破损保留实体可修理、双刃发光 FX 跟随与幸运触手协战。"
aliases: ["sword_lunarplant", "亮茄剑", "Brightshade Sword", "亮茄武器"]
tags: [prefab, item, equipment]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/sword_lunarplant.lua", "prefabs/hitsparks_fx.lua", "prefabs/charlie_boss.lua", "prefabs/staff_lunarplant.lua", "components/weapon.lua", "components/equippable.lua", "components/finiteuses.lua", "components/planardamage.lua", "components/damagetypebonus.lua", "components/lunarplant_tentacle_weapon.lua", "components/combat.lua", "components/fumaroletool.lua", "components/setbonus.lua", "standardcomponents.lua", "constants.lua", "tuning.lua", "recipes.lua", "componentutil.lua", "mods.lua", "screens/redux/scrapbookdata.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 亮茄剑（sword_lunarplant）

## 快速判断

- 职责与适用：亮茄（Brightshade）装备族的纯武器件——"可拾取＋手部装备＋武器＋耐久＋可修理"组合，无任何工具动作；每次命中基础 38＋位面 30，对 shadow_aligned 目标双乘 1.1。与同族工具（[[reference/prefabs/lunarplant-tools]]）共用"套装手工监听／破损循环／Forge 修理／触手协战"四套机制，本页只展开 sword 的差异，共同协议互链不复制。同文件另一注册 `sword_lunarplant_blade_fx` 是本体的双刃发光 FX（纯视觉，见[注册与构造](#注册与构造)）。
- 优先使用：借"纯武器＋耐久＋破损不消失"组装直接看[注册与构造](#注册与构造)；改数值走 TUNING 键（见[数值](#数值)）；给每次武器结算附加效果优先监听 `weapononattack` 事件（[典型接入](#典型接入)），比覆盖 `SetOnAttack` 安全。
- 重要副作用：耐久耗尽不删除实体——先强制卸下归还持有者，再移除 equippable／weapon、切 broken 动画、加 `broken` 标签；破损完全失能但可用 lunarplant_kit 一次修满；持握期间每次武器结算默认扣 1 耐久（乘区与 efficientuser 可改变）。[OnBroken](../../../raw/current/scripts/prefabs/sword_lunarplant.lua#L184-L192)、[修理 helper](../../../raw/current/scripts/standardcomponents.lua#L1851-L1884)
- 修改边界：组件配置全在服务端，公共端只有标签、floater 与 `isbroken` net 同步；双刃 FX 是网络实体，服务端生成与换绑、客户端可见但无独立逻辑；装备槽缺省 HANDS；onequip 有皮肤分支但本 build 未登记本注册的皮肤（见[验证记录](#验证记录)）。[端分界](../../../raw/current/scripts/prefabs/sword_lunarplant.lua#L232-L238)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `sword_lunarplant` | `Prefab("sword_lunarplant", fn, assets, prefabs)`：公共端 CreateEntity→Transform/AnimState/Network→MakeInventoryPhysics→bank/build `sword_lunarplant` 播 `idle` 循环＋`pb_energy_loop01` 泛光→tags `sharp`/`show_broken_ui`/`weapon`→floater＋`isbroken` net_bool→SetPristine→客户端仅监听 `isbrokendirty` 后返回→服务端主模拟：随机帧同步双刃 FX、finiteuses、base_damage、planardamage、damagetypebonus、inspectable、inventoryitem、SetupComponents（equippable＋weapon）、lunarplant_tentacle_weapon、MakeForgeRepairable、MakeHauntableLaunch | 对象类别：item＋equipment；equipslot 缺省 `EQUIPSLOTS.HANDS`（equippable 构造默认）；无 tool 组件、无动作消耗；全文无 quagmire 等分支。[工厂](../../../raw/current/scripts/prefabs/sword_lunarplant.lua#L206-L275)、[主模拟段](../../../raw/current/scripts/prefabs/sword_lunarplant.lua#L240-L274) |
| `sword_lunarplant_blade_fx` | 同文件 `Prefab("sword_lunarplant_blade_fx", fxfn, assets)`：Transform/AnimState/Follower/Network→`FX` 标签→bank/build 同本体→播 `swap_loop1` 循环＋同款泛光→highlightchild→SetPristine→客户端直接返回→服务端补 colouradder＋`persists=false` | 对象类别：纯视觉 fx；本体 fn 服务端生成 2 实例（`blade1`/`blade2`），blade2 改播 `swap_loop2`，两帧与本体随机帧同步；跟随与换绑全由本体 SetFxOwner 维护，自身无逻辑。[fxfn](../../../raw/current/scripts/prefabs/sword_lunarplant.lua#L277-L307)、[生成](../../../raw/current/scripts/prefabs/sword_lunarplant.lua#L240-L247) |

- 名称核验：`STRINGS.NAMES.SWORD_LUNARPLANT`＝"亮茄剑"（[简中条目](../../../raw/current/scripts/languages/chinese_s.po#L350194-L350197)）；破损态经 `inspectable.nameoverride="BROKEN_FORGEDITEM"` 改检视文案（[键](../../../raw/current/scripts/languages/chinese_s.po#L9959-L9962)）。
- 获取：月亮锻造台配方——纯粹辉煌×4＋亮茄外壳×3，`TECH.LUNARFORGING_TWO`＋`station_tag="lunar_forge"`＋nounlock（[配方](../../../raw/current/scripts/recipes.lua#L852)；材料名 [PUREBRILLIANCE](../../../raw/current/scripts/languages/chinese_s.po#L347384-L347387)、[LUNARPLANT_HUSK](../../../raw/current/scripts/languages/chinese_s.po#L344869-L344872)）；锻造站与科技树与工具族同站同档，见 [[reference/prefabs/lunarplant-tools#注册与构造]]。
- 同族：pickaxe/shovel 见工具族页；`staff_lunarplant` 的套装走 setbonus 组件路线（[对照](../../../raw/current/scripts/prefabs/staff_lunarplant.lua#L225)）。

### 数值

- `SWORD_LUNARPLANT_USES = 200`（[tuning](../../../raw/current/scripts/tuning.lua#L185)）；无动作消耗（不调 `SetConsumption`），唯一耐久路径是每次武器结算默认扣 1；`attackwear` 乘区、`efficientuser` 与 `IgnoresCombatDurabilityLoss` 会改变实际扣减。
- `SWORD_LUNARPLANT_DAMAGE = wilson_attack * 2 - 30`＝38、`SWORD_LUNARPLANT_PLANAR_DAMAGE = 30`（[tuning](../../../raw/current/scripts/tuning.lua#L388-L389)；基数 wilson_attack=34，[tuning](../../../raw/current/scripts/tuning.lua#L34)）。
- 套装三键同工具族：`WEAPONS_LUNARPLANT_SETBONUS_DAMAGE_MULT = 1.1`、`..._SETBONUS_PLANAR_DAMAGE = 5`、`WEAPONS_LUNARPLANT_VS_SHADOW_BONUS = 1.1`（[tuning](../../../raw/current/scripts/tuning.lua#L394-L396)）。套装激活时基础 38×1.1＝41.8、位面 30＋5＝35；对 shadow_aligned 目标再双乘 1.1。
- 交叉印证：`scrapbookdata` 的 `sword_lunarplant` 条目 weapondamage=38、planardamage=30、finiteuses=200、forgerepairable={"lunarplant_kit"}（[条目](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L1486)）。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：equippable | `SetOnEquip/SetOnUnequip` 挂 onequip/onunequip（持握换装＋SetFxOwner＋SetBuffOwner）；不设槽位，默认 HANDS | 主模拟 SetupComponents 挂载；破损随组件移除 | [[reference/components/equippable#关键接口]]；[默认槽位](../../../raw/current/scripts/components/equippable.lua#L29)、[配置](../../../raw/current/scripts/prefabs/sword_lunarplant.lua#L142-L144) |
| 组件：weapon | `SetDamage(inst.base_damage)`（修复重建时按 `_bonusenabled` 选择数值）；`SetOnAttack` 命中生成火花 | attackwear 缺省→每命中扣 1 耐久（efficientuser 可减免）；GetDamage 时 damagetypebonus 双乘基础与位面 | [[reference/components/weapon#关键接口]]；[扣耐久](../../../raw/current/scripts/components/weapon.lua#L111-L120)、[双乘](../../../raw/current/scripts/components/weapon.lua#L94-L103)、[combat 调用点](../../../raw/current/scripts/components/combat.lua#L1223-L1225) |
| 组件：finiteuses | 200 用；唯一消耗路径是武器命中（无 SetConsumption） | 耗尽触发 MakeForgeRepairable 接管的 onfinished | [[reference/components/finiteuses#关键接口]]；[配置](../../../raw/current/scripts/prefabs/sword_lunarplant.lua#L251-L253)、[helper 分支](../../../raw/current/scripts/standardcomponents.lua#L1869-L1876) |
| 组件：planardamage | `SetBaseDamage(30)`；套装激活 `AddBonus(+5,"setbonus")`，关闭移除 | 位面求和＝base×乘数＋加成，经 weapon 链以 SpDamage 进入结算 | [[reference/components/planardamage#关键接口]]；[配置](../../../raw/current/scripts/prefabs/sword_lunarplant.lua#L258-L259)、[求和](../../../raw/current/scripts/components/planardamage.lua#L18-L20) |
| 组件：damagetypebonus | `AddBonus("shadow_aligned", inst, 1.1)`：目标带该标签时基础与位面同乘 1.1 | 消费在 Weapon:GetDamage 的 target 标签判断 | [[reference/components/damagetypebonus#关键接口]]；[注册](../../../raw/current/scripts/prefabs/sword_lunarplant.lua#L261-L262)、[消费](../../../raw/current/scripts/components/weapon.lua#L97-L101) |
| 组件：lunarplant_tentacle_weapon | 装备期监听持有者 `onattackother`，命中本武器后按 20%＋幸运在目标旁刷 `lunarplanttentacle` 协战 | 默认门槛：虫木技能 `wormwood_allegiance_lunar_plant_gear_2`；组件自校验 attack_data.weapon | [[reference/components/lunarplant_tentacle_weapon#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/sword_lunarplant.lua#L269)、[掷点](../../../raw/current/scripts/componentutil.lua#L2237) |
| 组件：inspectable／inventoryitem／floater | 基础检视、拾取与漂浮；破损改 nameoverride，floater 尺寸/缩放按破损态切换 | 公共端配置；完好浮空换立式贴图（`SetBankSwapOnFloat(true,-17.5,SWAP_DATA)`），破损浮空不换 bank | [floater 切换](../../../raw/current/scripts/prefabs/sword_lunarplant.lua#L156-L182)、[破损检视](../../../raw/current/scripts/prefabs/sword_lunarplant.lua#L190)、[[reference/components/floater#关键接口]] |
| 同步：isbroken（net_bool） | 服务端 SetIsBroken `set()`；客户端监听 `isbrokendirty` 调 OnIsBrokenDirty 调整 floater 尺寸/偏移/缩放 | net 名 `sword_lunarplant.isbroken` 带自身前缀 | [定义](../../../raw/current/scripts/prefabs/sword_lunarplant.lua#L229)、[客户端回调](../../../raw/current/scripts/prefabs/sword_lunarplant.lua#L159-L169) |
| API／数据：MakeForgeRepairable＋FORGEMATERIALS.LUNARPLANT | 修理材料 `"lunarplant"`；`_onbroken` 先卸装归还再调 OnBroken；finiteuses 分支接管耗尽回调 | 与工具族同构；修理协议（lunarplant_kit 右键一次补满、候选标签、漂浮限制）见工具族页 | [helper](../../../raw/current/scripts/standardcomponents.lua#L1851-L1884)、[材料枚举](../../../raw/current/scripts/constants.lua#L1962-L1967)、[[reference/components/forgerepairable#关键流程]] |
| API／数据：套装判定（手工监听） | 与工具族同一结构：监听持有者 equip/unequip，判 HEAD 槽帽名 `lunarplanthat`，自带初值检查；激活＝weapon 伤害×1.1＋planardamage +5 | 不走 setbonus 组件系统；结构复刻与陷阱见工具族页典型接入 | [监听对](../../../raw/current/scripts/prefabs/sword_lunarplant.lua#L13-L65)、[[reference/prefabs/lunarplant-tools#典型接入]] |
| API／数据：weapononattack 事件 | weapon 组件在每次武器结算后于本体推送一次 `{attacker,target,projectile}`，只对主目标；范围副目标走攻击者的 `onareaattackother`，不逐目标推武器事件。原版消费者 fumaroletool（组件自听）与 charlie_boss（听持械者武器做偏转） | 附加每击效果无需覆盖 SetOnAttack；事件在 combat 命中流程内推送 | [推送](../../../raw/current/scripts/components/weapon.lua#L109)、[组件消费者](../../../raw/current/scripts/components/fumaroletool.lua#L52)、[实体消费者](../../../raw/current/scripts/prefabs/charlie_boss.lua#L260-L288) |
| 关联 Prefab：lunarplanttentacle | 触手协战产物（20%＋幸运掷点） | 9 秒自动退场、`persists=false`；机制与门槛见组件页 | [[reference/components/lunarplant_tentacle_weapon#关键接口]] |
| 关联 Prefab：lunarplant_kit | 修理包（纯粹辉煌×1＋亮茄外壳×1，同站） | 一次补满，消耗 1 个 | [[reference/prefabs/lunarplant-tools#关联机制导航]]、[配方](../../../raw/current/scripts/recipes.lua#L855) |
| 关联 Prefab：lunarplanthat | 套装帽（HEAD 槽，官方名"亮茄头盔"）；戴帽才触发本页加成 | 帽自身另有 setbonus 组件（月亮抗性）与独立破损循环 | [[reference/prefabs/lunarplant-tools#关联机制导航]] |
| 关联 Prefab：hitsparks_fx | OnAttack 命中有效目标时 `SpawnPrefab(...):Setup(attacker, target)` | 每次武器结算一次火花 FX | [回调](../../../raw/current/scripts/prefabs/sword_lunarplant.lua#L135-L139)、[Setup 签名](../../../raw/current/scripts/prefabs/hitsparks_fx.lua#L53) |
| 关联 Prefab：sword_lunarplant_blade_fx | 本体双刃发光 FX：服务端 fn 生成 2 实例；装备时引擎级 `entity:SetParent(owner.entity)`＋`Follower:FollowSymbol(owner,"swap_object")`（blade1 帧 0-3、blade2 帧 5-8）＋highlightchild/colouradder 跟随；卸下或漂浮时回落本体、改跟 `swap_spear` | 网络实体（AddNetwork），服务端生成客户端可见；`persists=false` 不入存档；本文件无显式 Kill，换绑只发生在 equip/unequip 回调 | [SetFxOwner](../../../raw/current/scripts/prefabs/sword_lunarplant.lua#L67-L93)、[fxfn](../../../raw/current/scripts/prefabs/sword_lunarplant.lua#L277-L307) |
| 资源：动画 | `anim/sword_lunarplant.zip`；bank/build 同名；本体 `idle`/`broken`，双刃 `swap_loop1`/`swap_loop2`，浮空换装符号 `swap_sword_lunarplant` | 注册声明并实际使用 | [assets](../../../raw/current/scripts/prefabs/sword_lunarplant.lua#L1-L4)、[SWAP_DATA](../../../raw/current/scripts/prefabs/sword_lunarplant.lua#L171-L172) |

## 典型接入

### 给每次武器结算附加效果（监听 weapononattack，不覆盖原回调）

- 前提与选择：服务端 `modmain.lua`。weapon 组件在每次武器结算后于本体推送一次 `weapononattack`（只对主目标，范围副目标走 `onareaattackother`）（[推送点](../../../raw/current/scripts/components/weapon.lua#L105-L109)），覆盖 `SetOnAttack` 会顶掉原版火花回调，而事件监听不碰原回调——要"叠加"而非"替换"时按此接入。mod 环境白名单只直挂 `TUNING` 等少量全局（[env 构造](../../../raw/current/scripts/mods.lua#L258-L317)），`TheWorld`/`SpawnPrefab`/`EQUIPSLOTS` 等须 `GLOBAL.` 前缀。
- 生效与撤销：AddPrefabPostInit 只影响新实例；事件由服务端 combat 命中流程触发（[调用点](../../../raw/current/scripts/components/combat.lua#L1223-L1225)），宿主卸下武器后事件仍只在本体推送，监听随实体存续。撤取消除 PostInit 即可。

```lua
-- modmain.lua：亮茄剑每次武器结算额外消耗持有者 1 点理智（服务端）
AddPrefabPostInit("sword_lunarplant", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    inst:ListenForEvent("weapononattack", function(inst, data)
        local attacker = data ~= nil and data.attacker or nil
        if attacker ~= nil and attacker:IsValid() and attacker.components.sanity ~= nil then
            attacker.components.sanity:DoDelta(-1)
        end
    end)
end)
```

- 调整与验证：事件数据 `projectile` 对本武器恒为 nil（近战、无 SetProjectile）；改基础数值不改结构时直接覆盖 `TUNING.SWORD_LUNARPLANT_DAMAGE`／`_PLANAR_DAMAGE`／`_USES`（构造时读表，只影响新实例；套装乘数在每次启用时重读 TUNING）；要动双刃 FX 表现须连同 SetFxOwner 的换绑与帧同步一起改，单独 Kill blade 实例会让泛光跟随失效。

## 常见陷阱与限制

- **边界：200 用不等于恒定 200 次命中**：无 tool 组件、无动作消耗，唯一耐久路径是 `Weapon:OnAttack` 每次武器结算扣 1；`attackwear` 乘区、`efficientuser` 与 `IgnoresCombatDurabilityLoss` 都会改变实际扣减，无修正时约 200 次。掉落/漂浮/检视均不扣。[消耗](../../../raw/current/scripts/components/weapon.lua#L111-L120)、[配置](../../../raw/current/scripts/prefabs/sword_lunarplant.lua#L251-L253)
- **边界：破损不消失但完全失能**：与铥矿棒等"耗尽即 Remove"不同，破损先被强制卸下归还（helper 统一逻辑），再移除 equippable/weapon、切 broken 动画加 `broken` 标签；修复由 SetupComponents 重建同一批组件。正常流程下破损前加成已随卸下关闭，修复重建总是基础伤害。[OnBroken/OnRepaired](../../../raw/current/scripts/prefabs/sword_lunarplant.lua#L184-L204)、[卸装归还](../../../raw/current/scripts/standardcomponents.lua#L1852-L1861)
- **边界：套装判定不走 setbonus 组件**：本注册（与 pickaxe/shovel）用手工监听，只判 HEAD 槽帽名 `lunarplanthat`，身体护甲不参与；staff/armor/hat 上的 setbonus 组件按"头＋身同 setname"配对驱动各自加成。两套机制并存，别混为一谈。[监听](../../../raw/current/scripts/prefabs/sword_lunarplant.lua#L31-L65)、[setbonus 配对对照](../../../raw/current/scripts/components/setbonus.lua#L52-L64)
- **边界：加成只在持握期间维护**：监听挂在持有者身上，卸下即 SetBuffOwner(nil) 并关加成；放背包/箱子不因戴帽保留状态，重新装备时按当前头槽重算。[解绑](../../../raw/current/scripts/prefabs/sword_lunarplant.lua#L31-L40)
- **边界：触手协战有双重门槛**：需持有者激活虫木技能 `wormwood_allegiance_lunar_plant_gear_2`，且单次命中过 20%（＋幸运加成）掷点；其他角色持同款武器无触手。见 [[reference/components/lunarplant_tentacle_weapon#常见陷阱与限制]]。[门槛](../../../raw/current/scripts/components/lunarplant_tentacle_weapon.lua#L1-L4)
- **边界：双刃 FX 换绑窗口**：SetFxOwner 只在 equip/unequip 回调执行；本文件没有 OnRemove 解绑逻辑，装备态下绕过卸装直接删除本体时 FX 归属不回落（该路径的清理行为未核验，见验证记录）。[换绑](../../../raw/current/scripts/prefabs/sword_lunarplant.lua#L67-L93)
- **边界：皮肤分支当前为死路径**：onequip/onunequip 有 GetSkinBuild 分支，但本 build prefabskin 未登记本注册、skins_defs_data 仅 scrapbook 解锁键注释，走不到 skinned 分支。[分支](../../../raw/current/scripts/prefabs/sword_lunarplant.lua#L109-L116)、[验证记录](#验证记录)

## 验证记录

- 源码审阅：2026-11-07（第三轮检视修订：复核 weapononattack 只随武器结算推主目标、范围副目标走 onareaattackother，及耐久扣减乘区）。早前 2026-11-06，Steam buildid 26335394（实读 raw/current/BUILD.txt）。完整读取 sword_lunarplant.lua 全文（311 行）与 weapon、finiteuses、planardamage、damagetypebonus、lunarplant_tentacle_weapon 组件全文；equippable 默认槽位与 equipped 事件段、combat OnAttack 调用点、MakeForgeRepairable/MakeHauntableLaunch/FORGEMATERIALS/TryLuckRoll/LuckFormulas.ShadowTentacleSpawn 定义、hitsparks_fx Setup 签名、fumaroletool 与 charlie_boss 的 weapononattack 消费段、mods.lua 环境构造段；tuning 键＋wilson_attack 基数；recipes L852 与 kit 配方；scrapbookdata L1482 交叉印证数值。名称核验 chinese_s.po 四键（SWORD_LUNARPLANT、BROKEN_FORGEDITEM、PUREBRILLIANCE、LUNARPLANT_HUSK）。皮肤检索：prefabskin.lua 全文无匹配、skins_defs_data.lua 仅 L6796 scrapbook 键注释。quagmire：全文无该分支。未覆盖：blade FX 引擎级父子级联清理与"装备态直接删本体"路径、colouradder/highlightchild 内部实现（仅核验挂载调用）、lunarplanttentacle 本体机制（链接工具族页）、皮肤清单数据侧、多人同步时序。
- 游戏运行：未运行；未验证实际持握外观、双刃跟随/帧同步表现与破损/修理的端表现。
- 结构检查：2026-11-06，validate_page.py 与 validate_source_links.py（--strict）通过。

相关：[[reference/prefabs/index|Prefab 索引]]；[[reference/prefabs/lunarplant-tools|亮茄工具族（共用机制主维护页）]]；[[reference/components/weapon|武器组件]]；[[reference/components/forgerepairable|锻造修理目标侧]]。

返回 [[reference/prefabs/weapons-combat-index|武器与战斗]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-08 build 26474873：本页随版本迁移升版（锚点迁移 1 处），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-12）。
