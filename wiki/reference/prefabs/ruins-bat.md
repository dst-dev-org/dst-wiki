---
title: "铥矿棒（ruins_bat）"
created: "2026-11-06"
updated: "2026-11-08"
type: prefab
summary: "原版远古手持武器 ruins_bat：伤害 59.5、耐久 200 仅攻击磨损、装备移速 +10%；命中按 20%（幸运加成）掷点在目标旁生成暗影触手（shadowtentacle），影子等级 2；单注册无蓄力／AoE，可借鉴“攻击掷点召唤”与远古站配方接入。"
aliases: ["ruins_bat", "铥矿棒", "Thulecite Club", "图勒棍棒"]
tags: [prefab, item, equipment]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/ruins_bat.lua", "components/weapon.lua", "components/finiteuses.lua", "components/equippable.lua", "components/inspectable.lua", "components/inventoryitem.lua", "components/shadowlevel.lua", "components/combat.lua", "componentutil.lua", "simutil.lua", "prefabs/shadowtentacle.lua", "prefabs/altar_prototyper.lua", "prefabs/minotaur.lua", "stategraphs/SGminotaur.lua", "stategraphs/SGshadowwaxwell.lua", "stringutil.lua", "standardcomponents.lua", "recipes.lua", "recipes_filter.lua", "strings.lua", "tuning.lua", "constants.lua", "mods.lua", "prefabskin.lua", "prefabs/skinprefabs.lua", "prefabskins.lua", "scenarios/chest_minotaur.lua", "prefabs/treasurechest.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 铥矿棒（ruins_bat）

## 快速判断

- 职责与适用：`prefabs/ruins_bat.lua` 顶层 `fn` 注册的单注册可拾取手持武器（物品＋装备）：weapon 伤害＋finiteuses 耐久＋equippable HANDS 槽＋装备移速 +10%，特色是命中掷点生成暗影触手与 shadowlevel 2；无 tool、无 SG／Brain、无 floater（`SetSinks(true)` 落水下沉），同文件无 quagmire 分支。[工厂](../../../raw/current/scripts/prefabs/ruins_bat.lua#L60-L115)、[注册](../../../raw/current/scripts/prefabs/ruins_bat.lua#L117)
- 优先使用：改伤害／耐久／移速查[数值与消耗](#数值与消耗)；复刻“命中掷点召唤”读[命中暗影触手](#命中暗影触手)与[典型接入](#典型接入)；从零造同类手持武器查 [[cookbook/equipment/hand]]。
- 重要副作用：每次攻击命中先执行 onattack 掷点（基础 20%，幸运 >0 时加成），成功才在目标旁生成暗影触手（`shadowtentacle`；显式传目标、收招后提前 Remove，9 秒只是兜底、不存档）；耐久耗尽 `SetOnFinished(inst.Remove)` 整个实体移除。[onattack](../../../raw/current/scripts/prefabs/ruins_bat.lua#L37-L58)、[触手自删](../../../raw/current/scripts/prefabs/shadowtentacle.lua#L67-L68)
- 修改边界：标签挂公共端、组件挂主模拟（[分界](../../../raw/current/scripts/prefabs/ruins_bat.lua#L74-L86)）；皮肤只换贴图（见[皮肤](#皮肤)）；本 build 的铥矿棒**没有**蓄力／范围群攻——`aoeweapon_base`／`aoeweapon_leap`／`aoeweapon_lunge`／`aoecharging` 均未挂载本 prefab，玩家 SGwilson 也没有 ru​ins_bat 专属状态，触手收到显式目标后只做单目标攻击，不自行索敌（见[陷阱](#常见陷阱与限制)）。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `ruins_bat` | 无包装工厂：`fn()` 直接构造，无 postinit | 物品＋装备；4 款皮肤均为 CreatePrefabSkin 注册的换装（`ruins_bat_init_fn`），非独立机制注册。[注册](../../../raw/current/scripts/prefabs/ruins_bat.lua#L117)、[皮肤 init](../../../raw/current/scripts/prefabskin.lua#L187-L188) |

`ruins_bat` 共同初始化（[ruins_bat.lua#L60-L115](../../../raw/current/scripts/prefabs/ruins_bat.lua#L60-L115)）：实体组件＋`MakeInventoryPhysics` → bank `ruins_bat`＋build `swap_ruins_bat`＋`idle` → 公共端标签 `sharp`、`weapon`、`shadowlevel`（pristine 镜像注释）→ `SetPristine`（非主模拟返回）→ 主模拟依次 weapon → finiteuses → inspectable → inventoryitem（`SetSinks(true)`）→ equippable（`walkspeedmult`）→ shadowlevel（`SetDefaultLevel(2)`）→ `MakeHauntableLaunch`。装备槽不逐 prefab 设置，用组件默认 HANDS。[equippable 默认](../../../raw/current/scripts/components/equippable.lua#L29)、[EQUIPSLOTS.HANDS](../../../raw/current/scripts/constants.lua#L656)

名称键经简中 .po `msgctxt` 核验：`STRINGS.NAMES.RUINS_BAT`＝铥矿棒（Thulecite Club）；通用检视语 `DESCRIBE.RUINS_BAT`＝“份量相当重。”。[.po 名称](../../../raw/current/scripts/languages/chinese_s.po#L348854-L348857)、[.po 检视语](../../../raw/current/scripts/languages/chinese_s.po#L18989-L18992)

### 数值与消耗

（tuning 键＋实际值；`wilson_attack`＝34×倍率 1，[tuning.lua#L31-L37](../../../raw/current/scripts/tuning.lua#L31-L37)）

| 项 | 键 | 实际值 |
|---|---|---|
| 武器伤害 | `TUNING.RUINS_BAT_DAMAGE`＝`wilson_attack`×1.75 | 59.5 生命值（源码推导，[tuning.lua#L375](../../../raw/current/scripts/tuning.lua#L375)） |
| 耐久上限 | `TUNING.RUINS_BAT_USES` | 200（[tuning.lua#L186](../../../raw/current/scripts/tuning.lua#L186)） |
| 装备移速倍率 | `TUNING.RUINS_BAT_SPEED_MULT` | 1.1（[tuning.lua#L419](../../../raw/current/scripts/tuning.lua#L419)） |
| 影子等级 | `TUNING.RUINS_BAT_SHADOW_LEVEL` | 2（[tuning.lua#L2871](../../../raw/current/scripts/tuning.lua#L2871)） |
| 触手掷点基础概率 | `TUNING.SHADOW_TENTACLE_RUINS_BAT_CHANCE` | 0.2（[tuning.lua#L9330](../../../raw/current/scripts/tuning.lua#L9331)） |
| 每次攻击磨损 | weapon `attackwear`（从未赋值） | 1 耐久点/次，且为唯一消耗路径（无 tool、无 `SetConsumption`；[OnAttack 结算](../../../raw/current/scripts/components/weapon.lua#L111-L120)） |
| 触手属性 | `TUNING.TENTACLE_HEALTH`／`TENTACLE_DAMAGE`／`TENTACLE_ATTACK_PERIOD` | 500 血／34 伤害／2 秒攻击间隔（[tuning.lua#L1180-L1184](../../../raw/current/scripts/tuning.lua#L1180-L1184)） |

### 命中暗影触手

每次攻击命中的服务端执行序：combat 结算调用 `weapon:OnAttack`（[combat.lua#L1223-L1225](../../../raw/current/scripts/components/combat.lua#L1223-L1225)）→ 先执行 onattack 回调（[weapon.lua#L105-L121](../../../raw/current/scripts/components/weapon.lua#L105-L121)）→ 再按 1 耐久点扣磨损。onattack（[ruins_bat.lua#L37-L58](../../../raw/current/scripts/prefabs/ruins_bat.lua#L37-L58)）流程：

1. **掷点**：`TryLuckRoll(owner, 0.2, LuckFormulas.ShadowTentacleSpawn)`（[定义](../../../raw/current/scripts/componentutil.lua#L2237-L2251)）。公式 `ShadowTentacleSpawn` 为 `CommonChanceLuckAdditive(0.2)`（[componentutil.lua#L2169](../../../raw/current/scripts/componentutil.lua#L2169)、[公式](../../../raw/current/scripts/componentutil.lua#L2078-L2082)）：攻击者幸运 >0 时概率＝0.2＋幸运×0.2，幸运 ≤0 时公式返回假值、回退基础 0.2；幸运读攻击者 `luckuser` 组件总值（[GetEntityLuck](../../../raw/current/scripts/componentutil.lua#L2203-L2205)），无组件按 0。
2. **落点**：目标有效取目标位置（否则取攻击者位置并把触手目标置空），`FindWalkableOffset` 以随机起始角、半径 2、最多 3 次尝试找可行走点，自定义检查 `NoHoles`（排除洞点），不允许水面（`allow_water=false`）。[调用](../../../raw/current/scripts/prefabs/ruins_bat.lua#L44-L46)、[签名](../../../raw/current/scripts/simutil.lua#L330)、[NoHoles](../../../raw/current/scripts/prefabs/ruins_bat.lua#L33-L35)
3. **生成**：找到落点才播两段触手音效并 `SpawnPrefab("shadowtentacle")`，`tentacle.owner = owner` 后 `combat:SetTarget(target)`。找不到落点本击无触手也无音效。[生成块](../../../raw/current/scripts/prefabs/ruins_bat.lua#L46-L57)

触手是独立生物实体：500 血、34 伤、2 秒攻击间隔、带 `shadow`／`notarget`／`shadow_aligned` 标签与负理智光环（`-SANITYAURA_MED`），SG 在 attack 动画第 7、17 帧各尝试一次，收招（`attack_post` 动画结束）即 Remove，另有 9 秒兜底；`persists = false` 不进存档；`owner` 只是归属字段，触手消失与武器无联动。[构造](../../../raw/current/scripts/prefabs/shadowtentacle.lua#L40-L68)

### 获取途径

| 途径 | 内容与证据 |
|---|---|
| 配方 | 活木 3＋铥矿 4＋噩梦燃料 4，`TECH.ANCIENT_FOUR`（＝`{ANCIENT = 4}`，[constants.lua#L1254](../../../raw/current/scripts/constants.lua#L1254)），`nounlock=true` 不可解锁、每次现场合成；`ancient_altar`（完整远古伪科学站）提供 `ANCIENTALTAR_HIGH`＝ANCIENT 4（[tuning.lua#L1284-L1286](../../../raw/current/scripts/tuning.lua#L1284-L1286)、[接线](../../../raw/current/scripts/prefabs/altar_prototyper.lua#L348)），`ancient_altar_broken` 只有 ANCIENT 2（[L456](../../../raw/current/scripts/prefabs/altar_prototyper.lua#L456)）做不了。[配方](../../../raw/current/scripts/recipes.lua#L809) |
| 守护者宝箱 | minotaur 死亡状态生成 `minotaurchestspawner`（[SGminotaur.lua#L341-L355](../../../raw/current/scripts/stategraphs/SGminotaur.lua#L341-L355)）→ 3 秒后 `dospawnchest` 生成 `minotaurchest` 并按 `PickSome` 抽 `max_loots-2..max_loots` 项填入（[minotaur.lua#L613-L640](../../../raw/current/scripts/prefabs/minotaur.lua#L613-L640)、[3 秒任务](../../../raw/current/scripts/prefabs/minotaur.lua#L686)）；`chest_loot` 第一项在 armorruins／ruinshat／ruins_bat 中随机 1 个（[L64-L75](../../../raw/current/scripts/prefabs/minotaur.lua#L64-L75)）。 |
| scenario 宝箱 | `chest_minotaur`：武器位 25% 给 ruins_bat／orangestaff／yellowstaff 之一，耐久 70%–100%；仅对应 scenario 启用时生效。[scenario](../../../raw/current/scripts/scenarios/chest_minotaur.lua#L23-L27) |
| 数据依赖（非生成点） | `treasurechest.lua` 的 `minotaur_scrapbook_adddeps` 收录 ruins_bat，属图鉴依赖数据。[列表](../../../raw/current/scripts/prefabs/treasurechest.lua#L518-L536) |

配方过滤归 crafting 菜单 CRAFTING_STATION（“All Crafting Stations”）过滤器的 ancients 分组。[过滤器](../../../raw/current/scripts/recipes_filter.lua#L398-L414)、[显示名](../../../raw/current/scripts/strings.lua#L10824)

### 皮肤

装备接线与常见手持物一致：`GetSkinBuild()` 非空时装备／卸装推 `equipskinneditem`／`unequipskinneditem`，持握贴图用 `OverrideItemSkinSymbol`（[onequip](../../../raw/current/scripts/prefabs/ruins_bat.lua#L12-L22)）。皮肤只换贴图，无额外组件或命中 FX；皮肤清单见 [prefabskins.lua#L1642-L1647](../../../raw/current/scripts/prefabskins.lua#L1642-L1647)，名称经 .po `SKIN_NAMES` 核验（[.po](../../../raw/current/scripts/languages/chinese_s.po#L406804-L406821)）：

| 皮肤注册名（官方简中名） | 稀有度 | 注册证据 |
|---|---|---|
| `ruins_bat_an`（新大棒） | Loyal | [CreatePrefabSkin](../../../raw/current/scripts/prefabs/skinprefabs.lua#L10734-L10742) |
| `ruins_bat_arcane`（低语锤矛） | Elegant／Woven | [CreatePrefabSkin](../../../raw/current/scripts/prefabs/skinprefabs.lua#L10744-L10753) |
| `ruins_bat_chess`（棋士球棒） | Elegant／Woven | [CreatePrefabSkin](../../../raw/current/scripts/prefabs/skinprefabs.lua#L10755-L10764) |
| `ruins_bat_heavy`（守门人之锤） | Elegant／Woven | [CreatePrefabSkin](../../../raw/current/scripts/prefabs/skinprefabs.lua#L10766-L10775) |

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：weapon | `SetDamage(RUINS_BAT_DAMAGE)`＋`SetOnAttack(onattack)` | 主模拟；`OnAttack` 先回调后扣耐久，`attackwear` 缺省按 1 | [[reference/components/weapon#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/ruins_bat.lua#L88-L90) |
| 组件：finiteuses | 上限＝当前＝200；`SetOnFinished(inst.Remove)`；未设 `SetConsumption` | 主模拟；耗尽先加 `usesdepleted` 标签再调回调 | [[reference/components/finiteuses#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/ruins_bat.lua#L94-L97)、[SetUses](../../../raw/current/scripts/components/finiteuses.lua#L64-L79) |
| 组件：equippable | `SetOnEquip`／`SetOnUnequip` 覆盖 `swap_object`、ARM_carry 显隐；`walkspeedmult = 1.1` | 主模拟；槽默认 HANDS；移速倍率经 replica `inventoryitem:SetWalkSpeedMult` 同步 | [[reference/components/equippable#关键接口]]；[接线](../../../raw/current/scripts/prefabs/ruins_bat.lua#L104-L107)、[replica 同步](../../../raw/current/scripts/components/equippable.lua#L7-L13) |
| 组件：inspectable／inventoryitem | 检视与拾取；`SetSinks(true)` 落水下沉 | 主模拟；已落地的物品改 sinks 会立即尝试下沉 | [[reference/components/inspectable#关键接口]]、[[reference/components/inventoryitem#关键接口]]；[SetSinks](../../../raw/current/scripts/components/inventoryitem.lua#L310-L317) |
| 组件：shadowlevel | `SetDefaultLevel(TUNING.RUINS_BAT_SHADOW_LEVEL)`＝2 | 主模拟；pristine 加同名标签镜像 | [[reference/components/shadowlevel#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/ruins_bat.lua#L109-L110) |
| API／数据：`TryLuckRoll`＋`LuckFormulas` | onattack 掷点决定是否生成触手 | 命中结算时读攻击者 `luckuser` 总值；公式 `CommonChanceLuckAdditive(0.2)` | [[reference/components/luckuser#幸运值如何进入概率]]；[TryLuckRoll](../../../raw/current/scripts/componentutil.lua#L2237-L2251)、[公式](../../../raw/current/scripts/componentutil.lua#L2078-L2082) |
| API／数据：`sharp` 标签 | `Combat:GetImpactSound` 按武器 `sharp` 选锐器命中音 | 命中穿甲结算时读取 | [[reference/components/combat#关键接口]]；[消费](../../../raw/current/scripts/components/combat.lua#L745) |
| 关联 Prefab：暗影触手（shadowtentacle，Wiki 待建） | 掷点成功后生成，`owner`＝攻击者、`SetTarget`＝受击目标 | 主模拟生成；收招可提前移除、9 秒兜底，`persists = false`；与武器无生命周期联动 | [构造](../../../raw/current/scripts/prefabs/shadowtentacle.lua#L40-L68)、[生成块](../../../raw/current/scripts/prefabs/ruins_bat.lua#L50-L55) |
| 资源：anim／swap 动画包 | ruins_bat.zip（bank）／swap_ruins_bat.zip（持握与掉落贴图） | 静态声明 | [声明](../../../raw/current/scripts/prefabs/ruins_bat.lua#L1-L5) |

`shadowlevel` 的快照消费端与 batbat 一致：麦斯威尔影像随从（protector）按领导者装备的 shadowlevel 总和加伤害（[SGshadowwaxwell.lua#L145-L160](../../../raw/current/scripts/stategraphs/SGshadowwaxwell.lua#L145-L160)），`shadowmagic` 角色检视时播报影子等级（[stringutil.lua#L343](../../../raw/current/scripts/stringutil.lua#L343)）；普通角色装备无玩法效果。本 prefab 未挂载任何 AoE／蓄力组件、无 SG／Brain、无 floater。

## 典型接入

### 调整命中触手概率

- 前提与选择：服务端 `modmain.lua`。`Weapon:SetOnAttack`（[weapon.lua#L53-L55](../../../raw/current/scripts/components/weapon.lua#L53-L55)）整体替换回调，改概率须重写整个生成块；`TryLuckRoll`／`LuckFormulas`／`FindWalkableOffset`／`SpawnPrefab`／`TWOPI` 是游戏全局，modmain 环境无全局回退，须经 `GLOBAL.` 读取（[GLOBAL 字段](../../../raw/current/scripts/mods.lua#L289)、[env 构造](../../../raw/current/scripts/mods.lua#L258-L317)）。
- 生效与撤销：postinit 只影响新实例；回调不存档，已有实例下次加载即用新回调。停用模组即恢复原回调。
- 原版依据：重写版保持原回调“掷点→落点→生成”结构与皮肤无关性，仅改概率参数（原版 20%）。[原回调](../../../raw/current/scripts/prefabs/ruins_bat.lua#L37-L58)

```lua
-- modmain.lua：铥矿棒命中触手概率从 20% 提到 50%，保留幸运加成与生成逻辑
local TryLuckRoll = GLOBAL.TryLuckRoll
local LuckFormulas = GLOBAL.LuckFormulas
local FindWalkableOffset = GLOBAL.FindWalkableOffset
local SpawnPrefab = GLOBAL.SpawnPrefab
local TWOPI = GLOBAL.TWOPI

AddPrefabPostInit("ruins_bat", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    inst.components.weapon:SetOnAttack(function(inst, owner, target)
        if not TryLuckRoll(owner, 0.5, LuckFormulas.ShadowTentacleSpawn) then
            return
        end
        local pt
        if target ~= nil and target:IsValid() then
            pt = target:GetPosition()
        else
            pt = owner:GetPosition()
            target = nil
        end
        local offset = FindWalkableOffset(pt, math.random() * TWOPI, 2, 3, false, true,
            function(p) return not GLOBAL.TheWorld.Map:IsPointNearHole(p) end,
            false, true, true)
        if offset == nil then
            return
        end
        inst.SoundEmitter:PlaySound("dontstarve/common/shadowTentacleAttack_1")
        inst.SoundEmitter:PlaySound("dontstarve/common/shadowTentacleAttack_2")
        local tentacle = SpawnPrefab("shadowtentacle")
        if tentacle ~= nil then
            tentacle.owner = owner
            tentacle.Transform:SetPosition(pt.x + offset.x, 0, pt.z + offset.z)
            tentacle.components.combat:SetTarget(target)
        end
    end)
end)
```

- 调整与验证：概率为每次命中独立掷点，无内置冷却或累计；只调数值不重写回调时，改 `TUNING.SHADOW_TENTACLE_RUINS_BAT_CHANCE` 会改基础值；公式的加法项固定 `luck×0.2`，幸运 >0 时概率＝调后的基础值＋幸运×0.2，并非只影响无幸运分支，改伤害／耐久在构造时读表、只对新实例生效。

## 常见陷阱与限制

- **误区：铥矿棒有蓄力／AoE 地面重击**：本 build 无此机制——`aoeweapon_base`／`aoeweapon_leap`／`aoeweapon_lunge`（使用者仅 hammer_mjolnir、spear_gungnir、spear_lance、spear_wathgrithr 等）与 `aoecharging`（slingshot／playercontroller 链）都未挂载 ruins_bat，SGwilson 亦无其专属状态；每击只结算单目标；触手收到显式目标后单目标攻击，不自行索敌。
- **边界：触手生成不是必现**：每击独立掷点，基础 20%；幸运 ≤0 回退基础值、幸运 >0 时概率＝0.2＋幸运×0.2；目标旁找不到可行走落点（洞点排除、`allow_water=false`）时本击无触手也无音效。[掷点](../../../raw/current/scripts/componentutil.lua#L2237-L2251)、[落点](../../../raw/current/scripts/prefabs/ruins_bat.lua#L44-L46)
- **边界：触手收招可提前移除，9 秒只是兜底**：`attack_post` 动画结束即 `Remove`，另有 `DoTaskInTime(9, inst.Remove)`＋`persists = false`；重进世界后消失，武器耐久不因触手消失返还。[自删](../../../raw/current/scripts/prefabs/shadowtentacle.lua#L67-L68)
- **边界：耐久耗尽整个实体 Remove**：`SetOnFinished(inst.Remove)`，无掉落；`OnSave` 只在当前数≠上限时存 `uses`。[回调](../../../raw/current/scripts/prefabs/ruins_bat.lua#L97)、[存档](../../../raw/current/scripts/components/finiteuses.lua#L48-L52)
- **边界：无 floater，落水即沉**：与 spear／batbat 的 `MakeInventoryFloatable` 不同，本 prefab 只 `SetSinks(true)`；物品掉在水面会直接下沉（[SetSinks](../../../raw/current/scripts/components/inventoryitem.lua#L310-L317)、[下沉判定](../../../raw/current/scripts/components/inventoryitem.lua#L546)）。
- **边界：影子等级无普适加成**：`shadowlevel` 只被麦斯威尔影像随从增伤与 `shadowmagic` 角色检视播报消费，普通角色装备铥矿棒没有额外暗影加成。[消费](../../../raw/current/scripts/stategraphs/SGshadowwaxwell.lua#L154-L160)
- **边界：配方不可解锁**：`nounlock=true`，必须站在完整远古伪科学站（`ancient_altar`，ANCIENT 4）前现场合成；碎裂站 ANCIENT 2 不满足。[科技档](../../../raw/current/scripts/tuning.lua#L1280-L1286)

## 验证记录

- 源码审阅：2026-11-07（第三轮检视修订：复核掷点基础值与幸运加法、触手显式目标与提前退场）。早前 2026-11-06，Steam buildid 26335394（raw/current/BUILD.txt）。核验 prefabs/ruins_bat.lua 全文（单注册、onattack 掷点与触手生成、装备皮肤接线、无 floater／无 quagmire）；weapon／finiteuses／equippable／shadowlevel／inventoryitem（SetSinks）／inspectable 关键方法与挂载顺序；`TryLuckRoll`／`LuckFormulas.ShadowTentacleSpawn`／`CommonChanceLuckAdditive`／`GetEntityLuck`（componentutil.lua）；`FindWalkableOffset` 签名（simutil.lua）；shadowlevel 消费端（SGshadowwaxwell `CheckLeaderShadowLevel`、stringutil 播报）；`sharp` 消费端（combat `GetImpactSound`）；shadowtentacle 构造、自删与不存档；tuning 七键＋`wilson_attack`＋触手三键；配方、`ANCIENT_FOUR` 与 altar_prototyper 两档科技、recipes_filter CRAFTING_STATION 过滤器；minotaur 宝箱链（SGminotaur 死亡状态→spawner→`chest_loot`）与 chest_minotaur scenario；4 皮肤注册与 .po 名称；`MakeHauntableLaunch` helper（standardcomponents.lua#L1053）；全 scripts grep "ruins_bat" 复核其余命中（debugcommands、misc_items 皮肤礼包、scrapbook 数据、skin_* 等）均非新机制。
- 游戏运行：未运行。未覆盖：59.5 伤害为源码推导（`wilson_attack`＝34），非运行测量；20% 掷点、触手落点、收招提前退场与 9 秒兜底为源码推导；宝箱与 scenario 掉率、皮肤实装外观未实测。
- 结构检查：validate_page.py（0 error，164 行／21320 字节，standard 档）与 validate_source_links.py --strict（80 条源码引用、26 个源文件，0 error 0 warning）实际执行通过。

返回 [[reference/prefabs/weapons-combat-index|武器与战斗]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-08 build 26474873：本页随版本迁移升版（锚点迁移 1 处），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-11）。
