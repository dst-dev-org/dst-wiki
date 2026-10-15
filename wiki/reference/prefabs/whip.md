---
title: "三尾猫鞭（whip）"
created: "2026-11-06"
updated: "2026-11-07"
type: prefab
summary: "原版长柄抽击武器 whip：伤害 27.2、耐久 175 仅攻击磨损、攻击距离＝玩家基础 3＋2（再加目标半径）；命中掷“超响鞭”（普通 25%／怪物 20%／Boss 5%，幸运加成），成功下一帧让 14 范围内有战斗目标的单位强制 hit 硬直并丢目标；单注册，无 quagmire 分支。"
aliases: ["whip", "三尾猫鞭", "Tail o' Three Cats", "鞭子", "长鞭", "响鞭"]
tags: [prefab, item, equipment]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/whip.lua", "components/weapon.lua", "components/equippable.lua", "components/finiteuses.lua", "components/combat.lua", "componentutil.lua", "standardcomponents.lua", "stategraphs/SGwilson.lua", "stategraphs/SGwilson_client.lua", "stategraphs/SGhermitcrab.lua", "stategraphs/SGwx78_possessedbody.lua", "prefabs/impact.lua", "prefabs/bullkelp_root.lua", "prefabs/crabking.lua", "tuning.lua", "constants.lua", "recipes.lua", "recipes_filter.lua", "prefabs/scienceprototyper.lua", "prefabskin.lua", "prefabs/skinprefabs.lua", "screens/redux/scrapbookdata.lua", "mods.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 三尾猫鞭（whip）

## 快速判断

- 职责与适用：`prefabs/whip.lua` 顶层 `fn` 注册的单注册可拾取手持武器（物品＋装备）：weapon 伤害＋finiteuses 耐久＋equippable HANDS 槽，特色是长攻击距离与命中“超响鞭”群体硬直；无 tool、无投射、无 SG／Brain，同文件无 quagmire 分支。[工厂](../../../raw/current/scripts/prefabs/whip.lua#L92-L141)
- 优先使用：改伤害／耐久／射程查[数值与消耗](#数值与消耗)；复刻“命中掷点群体硬直”读[命中超响鞭](#命中超响鞭)与[典型接入](#典型接入)；从零造同类手持武器查 [[cookbook/equipment/hand]]。
- 重要副作用：每次命中必在目标处生成 `impact` 特效并按目标类别掷超响鞭，成功则下一帧对 14 范围内“当前有战斗目标”的可打断单位 `DropTarget`＋强制 `hit` 硬直（不造成伤害）；耐久耗尽 `SetOnFinished(inst.Remove)` 整个实体移除。[onattack](../../../raw/current/scripts/prefabs/whip.lua#L62-L90)、[supercrack](../../../raw/current/scripts/prefabs/whip.lua#L33-L56)
- 修改边界：标签与 floater 挂公共端、组件挂主模拟（[分界](../../../raw/current/scripts/prefabs/whip.lua#L105-L116)）；皮肤只换贴图与命中音效（见[皮肤](#皮肤)）；玩家攻击状态机只记 whip 分支锚点不复制（见[关联机制导航](#关联机制导航)）；`bullkelp_root` 同挂 `whip` 标签但机制不同，本文只记同族关系。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `whip` | 无包装工厂：`fn()` 直接构造，无 postinit | 物品＋装备；皮肤经 `whip_init_fn`。[注册](../../../raw/current/scripts/prefabs/whip.lua#L141)、[皮肤 init](../../../raw/current/scripts/prefabskin.lua#L845-L852) |

共同初始化（[whip.lua#L92-L138](../../../raw/current/scripts/prefabs/whip.lua#L92-L138)）：实体组件＋`MakeInventoryPhysics` → bank／build `whip`＋`idle` → 公共端标签 `whip`、`weapon`（[pristine 镜像注释](../../../raw/current/scripts/prefabs/whip.lua#L107-L108)）→ `MakeInventoryFloatable("med", nil, 0.9)`（无竖向偏移、缩放 0.9、不换 bank）→ `SetPristine`（非主模拟返回）→ 主模拟依次 weapon → finiteuses → inspectable → inventoryitem → equippable → `MakeHauntableLaunch`。装备槽不逐 prefab 设置，用组件默认 HANDS。[equippable 默认](../../../raw/current/scripts/components/equippable.lua#L29)

名称键经简中 .po `msgctxt` 核验：`STRINGS.NAMES.WHIP`＝三尾猫鞭（Tail o' Three Cats）；通用检视语 `DESCRIBE.WHIP`＝“没什么比大响声更能带来宁静的了。”。[.po 名称](../../../raw/current/scripts/languages/chinese_s.po#L352009-L352012)、[.po 检视语](../../../raw/current/scripts/languages/chinese_s.po#L22679-L22682)

### 数值与消耗

（tuning 键＋实际值；`wilson_attack`＝34×倍率 1，[tuning.lua#L31-L37](../../../raw/current/scripts/tuning.lua#L31-L37)）

| 项 | 键 | 实际值 |
|---|---|---|
| 武器伤害 | `TUNING.WHIP_DAMAGE`＝`wilson_attack`×0.8 | 27.2 生命值（源码推导，[tuning.lua#L369](../../../raw/current/scripts/tuning.lua#L369)） |
| 耐久上限 | `TUNING.WHIP_USES` | 175（[tuning.lua#L165](../../../raw/current/scripts/tuning.lua#L165)） |
| 武器攻击距离 | `TUNING.WHIP_RANGE` | 2（[tuning.lua#L423](../../../raw/current/scripts/tuning.lua#L423)） |
| 超响鞭范围 | `TUNING.WHIP_SUPERCRACK_RANGE` | 14（[tuning.lua#L424](../../../raw/current/scripts/tuning.lua#L424)） |
| 超响鞭概率 | `TUNING.WHIP_SUPERCRACK_EPIC_CHANCE`／`..._MONSTER_CHANCE`／`..._CREATURE_CHANCE` | 0.05／0.2／0.25（[tuning.lua#L425-L427](../../../raw/current/scripts/tuning.lua#L425-L427)） |
| 每次攻击磨损 | weapon `attackwear`（从未赋值） | 1 耐久点/次，且为唯一消耗路径（无 tool、无 `SetConsumption`；[OnAttack 结算](../../../raw/current/scripts/components/weapon.lua#L105-L121)） |

### 攻击距离（长射程）

- `SetRange(TUNING.WHIP_RANGE)` 只传一个参数 → `attackrange`＝`hitrange`＝2（[weapon.lua#L48-L51](../../../raw/current/scripts/components/weapon.lua#L48-L51)）；combat 基础两值均为 3（[combat.lua#L29-L30](../../../raw/current/scripts/components/combat.lua#L29-L30)）。
- 出手范围用 `CalcAttackRangeSq`、实际命中另用 `CalcHitRangeSq`，本例两者都按(目标物理半径＋5)²（SetRange(2) 同时写 attackrange／hitrange，[combat.lua#L1014-L1025](../../../raw/current/scripts/components/combat.lua#L1014-L1025)、[命中](../../../raw/current/scripts/components/combat.lua#L1067-L1092)）；即 whip 比不调 `SetRange` 的普通近战（如 spear，仅基础 3）多约 2 的攻击距离——是加长的**近战**，不是远程：`CanRangedAttack` 要求 `projectile`（[weapon.lua#L84-L86](../../../raw/current/scripts/components/weapon.lua#L84-L86)），whip 未设。
- 玩家 AI 走位附带校验：`attackrangesq > 4` 且武器无 `projectile`／`rangedweapon` 标签时，攻击路径中点须为有效地面（防隔虚空抽击）（[combat.lua#L813-L824](../../../raw/current/scripts/components/combat.lua#L813-L824)）。
- 攻击节奏更慢：SGwilson `attack` 状态 whip 分支播 `whip_pre`＋`whip` 动画、第 10 帧才执行动作、状态超时 17 帧（普通武器 13 帧、第 8 帧）（[SGwilson.lua#L10857-L10862](../../../raw/current/scripts/stategraphs/SGwilson.lua#L10857-L10862)、[时间轴](../../../raw/current/scripts/stategraphs/SGwilson.lua#L11078-L11084)）；玩家 `min_attack_period`＝0.4 秒（`wilson_attack_period`，[tuning.lua#L43](../../../raw/current/scripts/tuning.lua#L43)）短于 17 帧（≈0.57 秒），连击节奏由动画超时决定（源码推导）。

### 命中超响鞭

每次命中命中的服务端执行序：combat 结算调用 `weapon:OnAttack`（[combat.lua#L1224](../../../raw/current/scripts/components/combat.lua#L1224)）→ 先执行 onattack 回调（[weapon.lua#L105-L121](../../../raw/current/scripts/components/weapon.lua#L105-L121)）→ 再按 1 耐久点扣磨损。onattack（[whip.lua#L62-L90](../../../raw/current/scripts/prefabs/whip.lua#L62-L90)）：

1. **impact 特效**：`SpawnPrefab("impact")` 放在目标位置、按攻击者→目标方向旋转；掷点成功放大 3 倍并在目标 `SoundEmitter` 播 `whip_large`（皮肤换音），失败播 `whip_small`。两者是“额外”音效——正常命中音仍由 combat 在目标端结算（[源码注释](../../../raw/current/scripts/prefabs/whip.lua#L77-L78)）。`impact` 是纯 FX：网络实体＋本地 `impact` 动画，专用服务器不生成本地动画，0.5 秒自删、不存档（[impact.lua#L27-L54](../../../raw/current/scripts/prefabs/impact.lua#L27-L54)）。
2. **掷点**：`TryLuckRoll(attacker, chance, whipsupersnapchanceadditive)`。`chance` 按目标标签取档：`epic` 0.05／`monster` 0.2／其余 0.25；公式在幸运 >0 时返回 `chance＋幸运×0.5`，否则返回假值回退基础档。幸运读攻击者 `luckuser` 总值（[TryLuckRoll](../../../raw/current/scripts/componentutil.lua#L2237-L2251)、[公式](../../../raw/current/scripts/prefabs/whip.lua#L58-L60)、[GetEntityLuckChance](../../../raw/current/scripts/componentutil.lua#L2211-L2214)）。
3. **supercrack（AoE 硬直）**：`DoTaskInTime(0)` 下一帧执行（[调度](../../../raw/current/scripts/prefabs/whip.lua#L85)）。以 **whip 实体自身位置**为圆心 `FindEntities` 半径 14，必须带 `_combat`、排除 `player`／`epic`／`shadow`／`shadowminion`／`shadowchesspiece`／`INLIMBO`／`whip_crack_imune`／`possessedbody` 与持有者（[标签表](../../../raw/current/scripts/prefabs/whip.lua#L31-L32)）；只处理 `combat:HasTarget()`（当前有索敌目标）的实体：`DropTarget`，且在有 `hit` 状态、未死亡、无 `transform`／`nointerrupt`／`frozen` 状态标签时先唤醒 sleeper 再 `GoToState("hit")`（[supercrack](../../../raw/current/scripts/prefabs/whip.lua#L33-L56)）。**不结算任何伤害**。`whip_crack_imune` 标签在快照内由帝王蟹（crabking）挂载（[crabking.lua#L1611](../../../raw/current/scripts/prefabs/crabking.lua#L1611)）。

### 获取途径

| 途径 | 内容与证据 |
|---|---|
| 配方 | 猫尾（coontail）3＋触手皮（tentaclespots）1，`TECH.SCIENCE_TWO`（＝`{SCIENCE = 2}`，炼金引擎 `researchlab2` 的 `ALCHEMYMACHINE` 科技树档）。[配方](../../../raw/current/scripts/recipes.lua#L479)、[TECH](../../../raw/current/scripts/constants.lua#L1246)、[科技树](../../../raw/current/scripts/tuning.lua#L1267-L1270)、[researchlab2](../../../raw/current/scripts/prefabs/scienceprototyper.lua#L300) |
| 配方过滤 | crafting 菜单 WEAPONS（武器）分组。[过滤列表](../../../raw/current/scripts/recipes_filter.lua#L1019-L1055)、[显示名 .po](../../../raw/current/scripts/languages/chinese_s.po#L427194-L427197) |
| 生成点 | 全 scripts `SpawnPrefab("whip")` 零命中，map 布局／scenario 宝箱／treasurechest 掉落均无条目；scrapbook 图鉴收录（数据注明 weapondamage 27.2、weaponrange 2、finiteuses 175、deps coontail＋tentaclespots）。[图鉴数据](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L1757) |

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：weapon | `SetDamage(WHIP_DAMAGE)`＋`SetRange(WHIP_RANGE)`＋`SetOnAttack(onattack)` | 主模拟；`OnAttack` 先回调后扣耐久，`attackwear` 缺省按 1 | [[reference/components/weapon#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/whip.lua#L118-L121)、[回调](../../../raw/current/scripts/prefabs/whip.lua#L62-L90) |
| 组件：finiteuses | 上限＝当前＝175；`SetOnFinished(inst.Remove)`；未设 `SetConsumption` | 主模拟；耗尽先加 `usesdepleted` 标签再调回调 | [[reference/components/finiteuses#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/whip.lua#L123-L126)、[SetUses](../../../raw/current/scripts/components/finiteuses.lua#L64-L79) |
| 组件：equippable | `SetOnEquip`／`SetOnUnequip`：覆盖 `swap_object`＋`whipline` 符号、ARM_carry／ARM_normal 显隐 | 主模拟；槽默认 HANDS；皮肤推 `equipskinneditem`＋`OverrideItemSkinSymbol` | [[reference/components/equippable#关键接口]]；[接线](../../../raw/current/scripts/prefabs/whip.lua#L132-L134)、[onequip](../../../raw/current/scripts/prefabs/whip.lua#L7-L19) |
| 组件：inspectable／inventoryitem | 检视与拾取，默认配置无定制 | 主模拟 | [[reference/components/inspectable#关键接口]]、[[reference/components/inventoryitem#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/whip.lua#L128-L130) |
| 组件：floater | 公共端 `MakeInventoryFloatable("med", nil, 0.9)` | 公共端；无 bank 换装（落水不换贴图） | [[reference/components/floater#关键接口]]；[调用](../../../raw/current/scripts/prefabs/whip.lua#L110)、[helper](../../../raw/current/scripts/standardcomponents.lua#L1772-L1791) |
| 组件：hauntable | `MakeHauntableLaunch`：被作祟时弹飞 | 主模拟 | [[reference/components/hauntable#按效果选工厂]]；[调用](../../../raw/current/scripts/prefabs/whip.lua#L136)、[helper](../../../raw/current/scripts/standardcomponents.lua#L1053-L1069) |
| SG：玩家 SGwilson `attack`／`dojostleaction` | 按 HANDS 槽装备的 `whip` 标签选鞭击动画分支（`whip_pre`＋`whip`、17 帧超时、第 10 帧动作）；客户端预测镜像，hermitcrab 与 WX-78 被附身 SG 有同款分支 | 本体无 SG／Brain；标签判定在装备槽 | [attack 分支](../../../raw/current/scripts/stategraphs/SGwilson.lua#L10857-L10862)、[客户端 attack](../../../raw/current/scripts/stategraphs/SGwilson_client.lua#L5085-L5090)、[dojostleaction](../../../raw/current/scripts/stategraphs/SGwilson.lua#L8622-L8627)、[SGwx78_possessedbody 分支](../../../raw/current/scripts/stategraphs/SGwx78_possessedbody.lua#L1027-L1032) |
| API／数据：`whip` 标签 | 挥击动画分支的判定键；`bullkelp_root` 也挂同标签共用该动画 | 装备在 HANDS 槽时读取 | [bullkelp 挂标签](../../../raw/current/scripts/prefabs/bullkelp_root.lua#L81)、[hermitcrab 分支](../../../raw/current/scripts/stategraphs/SGhermitcrab.lua#L2413) |
| API／数据：`TryLuckRoll`＋`luckuser` | 超响鞭掷点 | 命中结算时读攻击者幸运总值 | [[reference/components/luckuser#幸运值如何进入概率]]；[TryLuckRoll](../../../raw/current/scripts/componentutil.lua#L2237-L2251) |
| 关联 Prefab：impact | 每次命中生成于目标处、按挥击方向旋转；超响鞭成功放大 3 倍 | 主模拟生成；本地动画 animover 自删、0.5 秒服务端自删、不存档 | [[reference/prefabs/impact]]；[构造](../../../raw/current/scripts/prefabs/impact.lua#L27-L54)、[生成块](../../../raw/current/scripts/prefabs/whip.lua#L69-L88) |
| 关联 Prefab：bullkelp_root | 同挂 `whip` 标签的同族武器（同伤害／射程档），onattack 走“耐久折损＋掷点 Snap 自毁”路线，无超响鞭 | 独立注册；本文只记同族关系，详见 [[reference/prefabs/bullkelp_root]] | [onattack](../../../raw/current/scripts/prefabs/bullkelp_root.lua#L27-L49)、[数值](../../../raw/current/scripts/tuning.lua#L370-L431) |
| 数据：scrapbook 图鉴 | `whip` 条目展示伤害／射程／耐久并声明配方依赖 | 客户端 UI 数据 | [条目](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L1757) |
| 资源：anim／swap 动画包 | whip.zip（bank＋build `whip`）／swap_whip.zip（持握符号 `swap_object`＋`whipline` 绳线） | 静态声明 | [声明](../../../raw/current/scripts/prefabs/whip.lua#L1-L5) |

### 皮肤

装备接线与常见手持物一致：`GetSkinBuild()` 非空时装备／卸装推 `equipskinneditem`／`unequipskinneditem`，持握贴图用 `OverrideItemSkinSymbol` 同时覆盖 `swap_object` 与 `whipline`（[onequip](../../../raw/current/scripts/prefabs/whip.lua#L8-L19)）。已注册 1 种皮肤；其命中音经 `skin_sound` 表注入：`whip_init_fn` 主模拟侧调 `AddSkinSounds` 设置 `inst.skin_sound_large`／`inst.skin_sound_small`，onattack 优先取皮肤音、无皮肤回退 `dontstarve/common/whip_large`／`whip_small`（[AddSkinSounds](../../../raw/current/scripts/prefabskin.lua#L20-L45)、[SKIN_SOUND_FX 写入](../../../raw/current/scripts/prefabskin.lua#L4090-L4092)、[取音](../../../raw/current/scripts/prefabs/whip.lua#L83-L87)）。

| 皮肤（官方简中名） | 音效与注册 | 证据 |
|---|---|---|
| 荆鞭（whip_snapthorn，Elegant／Woven） | `skin_sound` large／small＝`terraria1/skins/snapthorn` | [注册](../../../raw/current/scripts/prefabs/skinprefabs.lua#L16961-L16971)、[名称 .po](../../../raw/current/scripts/languages/chinese_s.po#L409524-L409527) |

## 典型接入

### 调整超响鞭概率、范围与武器数值

- 前提与选择：服务端 `modmain.lua`。伤害与耐久在构造时读表（只对新实例生效，用 `AddPrefabPostInit` 补已有实例）；超响鞭三档概率与范围在每次命中／触发时读 `TUNING`，直接改键立即全局生效。`TUNING` 在 modmain 环境白名单直挂（[mods.lua#L258-L317](../../../raw/current/scripts/mods.lua#L258-L317)），`TheWorld` 等非白名单全局须 `GLOBAL.` 前缀。
- 生效与撤销：TUNING 改动停用模组后恢复；postinit 影响新实例，已有实例上限即时生效、当前耐久按存档恢复。
- 原版依据：读取点见 [onattack](../../../raw/current/scripts/prefabs/whip.lua#L62-L90) 与 [supercrack](../../../raw/current/scripts/prefabs/whip.lua#L33-L56)。

```lua
-- modmain.lua：超响鞭对 Boss 概率 5%→15%、范围 14→10；新造 whip 伤害 34、耐久 250
TUNING.WHIP_SUPERCRACK_EPIC_CHANCE = 0.15
TUNING.WHIP_SUPERCRACK_RANGE = 10

AddPrefabPostInit("whip", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    inst.components.weapon:SetDamage(34)
    inst.components.finiteuses:SetMaxUses(250)
    inst.components.finiteuses:SetUses(250)
end)
```

- 调整与验证：概率为每次命中独立掷点、无冷却累计；范围单位为世界距离（`FindEntities` 半径）。`SetOnAttack`（[weapon.lua#L53-L55](../../../raw/current/scripts/components/weapon.lua#L53-L55)）整体替换回调——要改排除名单（`CRACK_CANT_TAGS` 为文件 local）或掷点公式，须连同 `supercrack` 一起重写。

## 常见陷阱与限制

- **误区：超响鞭是 AoE 伤害**：`supercrack` 只 `DropTarget`＋强制 `hit` 硬直，无任何伤害结算；伤害只有本体的单目标 27.2。[实现](../../../raw/current/scripts/prefabs/whip.lua#L33-L56)
- **边界：只打断“当前有战斗目标”的实体**：`combat:HasTarget()` 为假（未索敌、游荡中）的实体即使半径内也不进硬直；刚被 `DropTarget` 的实体下一次循环不再满足条件。[条件](../../../raw/current/scripts/prefabs/whip.lua#L38)
- **边界：硬直排除名单较宽**：玩家、`epic`（多数 Boss）、`shadow` 系（影怪／影子随从／影子棋子）、`possessedbody`、`whip_crack_imune`（蟹国王）都不被打断；处于 `transform`／`nointerrupt`／`frozen` 状态或无 `hit` 状态的实体切不进硬直，睡眠单位会先被唤醒再被打断。[标签表](../../../raw/current/scripts/prefabs/whip.lua#L31-L32)、[状态条件](../../../raw/current/scripts/prefabs/whip.lua#L40-L52)
- **边界：长射程不是远程**：无 `projectile`，不能 `CanRangedAttack`；AI 持 whip 攻击时若攻击路径中点是虚空，`LocomotorCanAttack` 拒绝“到位即打”，须走到有效地面。[校验](../../../raw/current/scripts/components/combat.lua#L813-L824)
- **边界：连击比普通武器慢**：SG `attack` 状态超时 17 帧（≈0.57 秒）且第 10 帧才执行动作，普通武器 13 帧／第 8 帧；玩家 `min_attack_period` 0.4 秒不构成瓶颈（源码推导，未实测）。[时间轴](../../../raw/current/scripts/stategraphs/SGwilson.lua#L11078-L11084)
- **边界：耐久耗尽即整个实体 Remove**：`SetOnFinished(inst.Remove)`，无掉落；`OnSave` 只在当前数≠上限时存 `uses`。[回调](../../../raw/current/scripts/prefabs/whip.lua#L126)、[存档](../../../raw/current/scripts/components/finiteuses.lua#L48-L52)
- **边界：命中音效播在目标身上且是“额外”的**：正常命中音仍由 combat 结算；目标无 `SoundEmitter` 时静默跳过，皮肤音只在装备对应皮肤时注入。[注释](../../../raw/current/scripts/prefabs/whip.lua#L77-L78)、[注入](../../../raw/current/scripts/prefabskin.lua#L20-L45)

## 验证记录

- 源码审阅：2026-11-07（第三轮检视修订：复核攻击范围与命中范围两个函数、帝王蟹名称）。早前 2026-11-06，Steam buildid 26335394（raw/current/BUILD.txt）。核验 prefabs/whip.lua 全文（单注册、onattack 三步、supercrack 范围与排除、装备皮肤接线、floater 参数、无 quagmire 分支）；weapon／finiteuses 全文，equippable／combat 关键段（GetAttackRange／CalcAttackRangeSq／GetHitRange／CanAttack／LocomotorCanAttack 虚空校验）；SGwilson `attack`／`dojostleaction` whip 分支与时间轴、SGwilson_client attack 预测分支、SGhermitcrab 同分支；`TryLuckRoll`／`GetEntityLuckChance`／`luckuser`；impact 构造与自删；crabking `whip_crack_imune`；bullkelp_root 同标签同族差异；tuning 八键＋`wilson_attack`／`wilson_attack_period`；配方、`TECH.SCIENCE_TWO` 与 researchlab2 科技树、WEAPONS 过滤器、.po 名称／检视语／皮肤名；皮肤注册与 `SKIN_SOUND_FX` 注入；modmain 环境白名单（mods.lua CreateEnvironment）；全 scripts grep "whip" 复核（SpawnPrefab 零命中；其余命中为 SG 动画分支、音效／手柄振动、皮肤礼包与图鉴数据，均非新生成点）。
- 游戏运行：未运行。未覆盖：27.2 伤害、(半径＋5) 射程与 17 帧连击节奏为源码推导，非运行测量；超响鞭实机触发范围观感、皮肤音效、帝王蟹免疫实际表现未验证。
- 结构检查：2026-11-06 validate_page.py 0 error（138 行／21604 字节，standard 档）与 validate_source_links.py --strict（89 条源码引用、24 个源文件，0 error 0 warning）实际执行通过。

返回 [[reference/prefabs/weapons-combat-index|武器与战斗]]、[[reference/prefabs/index|Prefab 索引]]。
