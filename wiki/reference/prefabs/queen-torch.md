---
title: "女王火炬（queen_torch）"
created: "2026-11-07"
updated: "2026-11-07"
type: prefab
summary: "原版女王火炬 queen_torch：HANDS 槽近战武器＋右键投掷抛射的永燃照明装备，光来自独立 queen_torchlight 光实体（蓝白光），16 单位内驱退暗影类生物并门控其生成；无燃料永不熄灭、不可被点燃、不防水；无配方，仅远古大门 Charlie 战败仪式落地生成，可选皮肤暗花火炬。"
aliases: ["queen_torch", "女王火炬", "Queen's Torch", "Queen Torch", "queen_torch_fx", "queen_torchlight", "暗花火炬", "投掷照明火炬", "驱影火炬"]
tags: [prefab, item, equipment]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/queen_torch.lua", "prefabs/atrium_gate.lua", "prefabs/wilson.lua", "prefabs/miasma_cloud_fx.lua", "prefabs/ruins_shadeling.lua", "prefabs/shadowskittish.lua", "prefabs/shadowhand.lua", "prefabs/nightmarerock.lua", "prefabs/waveyjones.lua", "prefabs/oceanshadowcreature.lua", "prefabs/charlie_boss_runner.lua", "prefabs/charlie_boss.lua", "prefabs/hats.lua", "prefabs/skinprefabs.lua", "prefabskins.lua", "components/hallucinations.lua", "components/equippable.lua", "components/weapon.lua", "components/burnable.lua", "components/complexprojectile.lua", "components/inventoryitem.lua", "components/inspectable.lua", "components/stalkerinspectable.lua", "components/skilltreeupdater.lua", "components/firedetector.lua", "components/playercontroller.lua", "stategraphs/SGwilson.lua", "brains/braincommon.lua", "brains/stalkerbrain.lua", "standardcomponents.lua", "constants.lua", "tuning.lua", "actions.lua", "componentactions.lua", "modutil.lua", "mods.lua", "screens/redux/scrapbookdata.lua", "strings.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 女王火炬（queen_torch）

## 快速判断

- 职责与适用：`prefabs/queen_torch.lua` 一文件注册三个 Prefab：主体 `queen_torch`（可拾取物品＋HANDS 槽装备＋武器）、`queen_torch_fx`（持握火焰跟随 FX）、`queen_torchlight`（联网 Light＋声音实体）。定位是“武器＋投掷＋永燃照明＋驱影”：近战伤害与火炬相同，右键投掷落地持续照明，光域内暗影类 AI 被门控。同文件无 quagmire 分支（全文无该环境代码）。[注册返回](../../../raw/current/scripts/prefabs/queen_torch.lua#L406-L408)
- 优先使用：要“不会烧尽、能驱暗影”的照明/武器对照 [[reference/prefabs/torch|火炬（torch）]]（燃料驱动、无驱影）与 [[reference/prefabs/lantern|提灯（lantern）]]（可充能开关）；复刻持握火焰 FX 与光实体分离的结构查[关联机制导航](#关联机制导航)；接入投掷查 [[reference/components/complexprojectile#关键接口]]。
- 重要副作用：生成即点燃（[fn 末尾](../../../raw/current/scripts/prefabs/queen_torch.lua#L308)）；装备/落地时光实体常驻且永燃（无 fueled，见[陷阱](#常见陷阱与限制)）；光域 16 单位内驱退/门控暗影类生物（[消费端](#关联机制导航)）；地面点燃态被瘴气系统认作火焰源使瘴气消退（[gestaltflame 行](#关联机制导航)）。
- 修改边界：标签与 floater 挂公共端、组件挂主模拟（[分界](../../../raw/current/scripts/prefabs/queen_torch.lua#L263-L267)）；光参数在 `queen_torchlight` 实体上而非本体组件；本页不含 Charlie 仪式流程与暗影 AI 内部机制（源码入口见表）。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `queen_torch` | `fn()` 直接构造：实体组件＋公共端标签→`SetPristine`→主模拟组件与存档钩子 | 物品＋装备；生成即经 `IgniteTossed` 点燃并挂光实体；无 postinit、无 SG/Brain、无配方 |
| `queen_torch_fx` | `fxfn()`：Transform/AnimState/Follower/Network→`SetPristine`→主模拟注入 `AttachToOwner` 方法、`persists=false` | 纯 FX（`FX` 标签）；装备时带皮肤参数 `SpawnPrefab`，跟随宿主 `swap_object` 符号 |
| `queen_torchlight` | `torchlightfn()`：Transform/Light/SoundEmitter/Network→双端注册进 `TORCHES`→`SetPristine`→主模拟 `persists=false`、休眠声音钩子 | 光实体（非物品）；装备/点燃时生成并 `SetParent` 到持有人或本体；两侧都注册，供 `FindClosestQueenTorch` 查询 |

主体共同初始化（[queen_torch.lua#L230-L311](../../../raw/current/scripts/prefabs/queen_torch.lua#L230-L311)）：`MakeInventoryPhysics` → bank/build `queen_torch`＋`idle` → 公共端标签 `waterproofer`／`weapon`／`projectile`／`complexprojectile`／`special_action_toss`／`keep_equip_toss`／`torch`／`gestaltflame` → `MakeInventoryFloatable("med", nil, 0.68)` → 主模拟依次 inspectable → weapon → inventoryitem → equippable → complexprojectile → burnable → stalkerinspectable → `MakeHauntableLaunch` → OnSave/OnLoad。装备槽用组件默认 HANDS（未覆盖，[equippable.lua#L29](../../../raw/current/scripts/components/equippable.lua#L29)、[EQUIPSLOTS.HANDS](../../../raw/current/scripts/constants.lua#L656)）。

名称键经简中 .po 核验：`STRINGS.NAMES.QUEEN_TORCH`＝女王火炬（[.po 名称](../../../raw/current/scripts/languages/chinese_s.po#L348409-L348412)、英文名 [strings.lua:4791](../../../raw/current/scripts/strings.lua#L4791)）；通用检视语＝“真是奇怪的火焰啊。”（[.po 检视语](../../../raw/current/scripts/languages/chinese_s.po#L18439-L18442)）；图鉴特殊说明＝“驱退暗影的永恒火焰。”（[.po 图鉴](../../../raw/current/scripts/languages/chinese_s.po#L362559-L362562)）。

数值（tuning 键＋实际值；`wilson_attack`＝34×倍率 1，[tuning.lua#L31-L37](../../../raw/current/scripts/tuning.lua#L31-L37)）：

| 项 | 键／配置 | 实际值 |
|---|---|---|
| 武器伤害 | `TUNING.TORCH_DAMAGE`＝`wilson_attack`×.5 | 17 生命值（源码推导，[tuning.lua#L361](../../../raw/current/scripts/tuning.lua#L361)、图鉴同值 [scrapbookdata.lua:1195](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L1195)） |
| 光初始参数 | torchlightfn 硬编码 | 强度 0.6／半径 6／衰减 0.8／颜色 (128,162,255)/255 蓝白（[L380-L383](../../../raw/current/scripts/prefabs/queen_torch.lua#L380-L383)） |
| 装备态光半径 | `applyskillbrightness` 硬编码 `5×修正` | 基础 5；持有者激活 `wilson_torch_4/5/6` 时修正 8/6、10/6、12/6 → 约 6.67/8.33/10（[L45-L56](../../../raw/current/scripts/prefabs/queen_torch.lua#L45-L56)） |
| 驱影判定半径 | `TUNING.QUEEN_TORCH_RADIUS`／`_SQ` | 16 单位（SQ=256；`_SQ` 为实际消费键，`_RADIUS` 无消费端，[tuning.lua#L9776-L9777](../../../raw/current/scripts/tuning.lua#L9776-L9777)） |
| 影类逃跑距离 | braincommon 常量 | 距 10 触发逃跑、距 15 停止（[braincommon.lua#L667-L671](../../../raw/current/scripts/brains/braincommon.lua#L667-L671)） |
| 投掷参数 | complexprojectile 配置 | speed 15、gravity −35、出手偏移 (0.25, 1, 0)、默认高抛物线（[L284-L292](../../../raw/current/scripts/prefabs/queen_torch.lua#L284-L292)） |
| 被偏转重掷 | OnDeflect 配置 | 暂用 speed 5／偏移 (0, 0.5, 0)／低抛掷向反方向 3 单位，随后恢复 speed 15／高抛（[L168-L182](../../../raw/current/scripts/prefabs/queen_torch.lua#L168-L182)） |
| 小地图 | MiniMapEntity | icon `queen_torch.png`、priority 6（[L239-L240](../../../raw/current/scripts/prefabs/queen_torch.lua#L239-L240)） |

获取途径：**无配方、无任何 lootdropper 掉落**（全 scripts grep `queen_torch` 无 lootdropper 命中，beequeen.lua 亦无）；唯一生成点是远古大门 Charlie 战败后的仪式布景 `SpawnShroudenLootSetPiece`，与 `king_cane`、`mask_charlie_bosshat` 一同落地（[atrium_gate.lua#L698-L716](../../../raw/current/scripts/prefabs/atrium_gate.lua#L698-L716)，生成调用 [L715](../../../raw/current/scripts/prefabs/atrium_gate.lua#L715)；依赖声明 [L31](../../../raw/current/scripts/prefabs/atrium_gate.lua#L31)）。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：weapon | `SetDamage(TUNING.TORCH_DAMAGE)`；无 SetOnAttack、无 finiteuses，攻击不引燃目标、不耗耐久 | 主模拟 | [[reference/components/weapon#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/queen_torch.lua#L271-L272)、[SetDamage](../../../raw/current/scripts/components/weapon.lua#L44-L46) |
| 组件：equippable | OnEquip 点燃＋覆写持握符号＋生成 fx/光＋技能刷新；OnUnequip 移除光/fx＋灭火；OnEquipToModel 空实现；槽默认 HANDS | 主模拟；装备态光实体 `SetParent(owner.entity)` | [[reference/components/equippable#关键接口]]；[接线](../../../raw/current/scripts/prefabs/queen_torch.lua#L279-L282)、[onequip](../../../raw/current/scripts/prefabs/queen_torch.lua#L79-L105)、[onunequip](../../../raw/current/scripts/prefabs/queen_torch.lua#L107-L129) |
| 组件：inventoryitem | OnPutInInventory 灭光灭火并记录持有者、OnPickup 清 thrower、OnDropped 重新点燃；运行时 `canbepickedup` 在投掷中关、落地开 | 主模拟；入袋态必然熄灭不发光 | [[reference/components/inventoryitem#关键接口]]；[接线](../../../raw/current/scripts/prefabs/queen_torch.lua#L274-L277)、[回调](../../../raw/current/scripts/components/inventoryitem.lua#L293-L306) |
| 组件：inspectable／stalkerinspectable | 常规检视；stalkerinspectable 让古代门 NPC（STALKER_NPC）将其视作可交谈评论对象（按名字缓存已评） | 主模拟 | [[reference/components/inspectable#关键接口]]、[[reference/components/stalkerinspectable#关键接口]]；[stalkerbrain 消费](../../../raw/current/scripts/brains/stalkerbrain.lua#L371-L374) |
| 组件：complexprojectile | TOSS 投掷载体：`ismeleeweapon=true`（装备态左键近战、右键投掷）；OnLaunch 自旋＋点燃＋暂不可拾取，OnHit 落地恢复，OnDeflect 被弹开且 `SetKeepOnDeflect(true)` 不销毁 | 主模拟；投掷动作不走 componentactions 通用收集——两处均排除 `special_action_toss`（[L2074](../../../raw/current/scripts/componentactions.lua#L2074)、[L2285](../../../raw/current/scripts/componentactions.lua#L2285)），仅 Wilson `wilson_torch_7` 门控给出（见上表 wilson 点动作行）；`keep_equip_toss` 让 TOSS 走 `throw_keep_equip` 投掷状态并抑制通用自动装备（[playercontroller.lua:4675](../../../raw/current/scripts/components/playercontroller.lua#L4675)、[SGwilson.lua:1429](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1429)：动作物品不在 HANDS 时先补装备、投出后恢复原装备，物品本身仍被抛出）；Charlie 本体战斗会把域内抛射物 `Deflect` 弹回 | [[reference/components/complexprojectile#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/queen_torch.lua#L284-L292)、[TOSS fn](../../../raw/current/scripts/actions.lua#L3953-L3999)、[点动作收集](../../../raw/current/scripts/componentactions.lua#L2280-L2292)、[Boss 弹回](../../../raw/current/scripts/prefabs/charlie_boss.lua#L310-L322)、[Deflect](../../../raw/current/scripts/components/complexprojectile.lua#L213-L220) |
| 组件：burnable | 状态开关载体：`canlight=false`（外部点不燃）、`fxprefab=nil`（Ignite 不产火 FX）；Ignite/Extinguish 只在穿脱/入袋/落地时被本文件调用 | 主模拟；burning 状态镜像出 `fire` 标签（[onburning](../../../raw/current/scripts/components/burnable.lua#L12-L15)），无 fueled/propagator/burntime → 永不自动烧尽 | [[reference/components/burnable#关键接口]]；[配置](../../../raw/current/scripts/prefabs/queen_torch.lua#L294-L297)、[Ignite](../../../raw/current/scripts/components/burnable.lua#L352-L390) |
| 组件：hauntable／floater | `MakeHauntableLaunch` 作祟弹飞；`MakeInventoryFloatable("med", nil, 0.68)` 中浮态 | 主模拟／公共端 | [[reference/components/hauntable#按效果选工厂]]、[[reference/components/floater#关键接口]]；[调用](../../../raw/current/scripts/prefabs/queen_torch.lua#L261)、[L301](../../../raw/current/scripts/prefabs/queen_torch.lua#L301)、[floater helper](../../../raw/current/scripts/standardcomponents.lua#L1772)、[hauntable helper](../../../raw/current/scripts/standardcomponents.lua#L1053) |
| API／事件：技能刷新 | 装备态监听持有者 `onactivateskill_server`／`ondeactivateskill_server`，经 `skilltreeupdater:IsActivated("wilson_torch_4/5/6")` 改光半径；投掷/入袋时一次性读技能存 thrower 修正 | 需持有者 skilltreeupdater（Wilson 技能树）；仅装备态持续监听，卸下解除 | [[reference/components/skilltreeupdater#关键接口]]；[监听](../../../raw/current/scripts/prefabs/queen_torch.lua#L67-L77)、[修正](../../../raw/current/scripts/prefabs/queen_torch.lua#L51-L65)、[IsActivated](../../../raw/current/scripts/components/skilltreeupdater.lua#L27-L33) |
| API／数据：`torch`＋`special_action_toss` 标签 | Wilson 激活 `wilson_torch_7` 后右键本体即得 TOSS 点特殊动作（原版 torch 同款组合） | 客户端动作收集 | [GetPointSpecialActions](../../../raw/current/scripts/prefabs/wilson.lua#L29-L42)、[标签](../../../raw/current/scripts/prefabs/queen_torch.lua#L255-L258) |
| API／数据：`gestaltflame` 标签 | 地面点燃态（burning→`fire` 标签）被瘴气系统认作火焰源使瘴气消退；firedetector 的 NOTAGS 排除该标签实体 | 地面点燃态；瘴气按 FIRE_RADIUS 搜索 `fire`/`heatstar` 后逐个测试 | [TestFireFn](../../../raw/current/scripts/prefabs/miasma_cloud_fx.lua#L414-L418)、[消费](../../../raw/current/scripts/prefabs/miasma_cloud_fx.lua#L462-L469)、[firedetector](../../../raw/current/scripts/components/firedetector.lua#L1) |
| API／数据：`FindClosestQueenTorch`（驱影） | 文件级全局函数在 `TORCHES` 光实体注册表上按 `QUEEN_TORCH_RADIUS_SQ` 找最近光（[L17-L41](../../../raw/current/scripts/prefabs/queen_torch.lua#L17-L41)）；逃跑入口 `BrainCommon.RunAwayFromQueenTorch` 距 10 逃、距 15 停（[braincommon.lua#L663-L672](../../../raw/current/scripts/brains/braincommon.lua#L663-L672)） | 双端可查（注册在 `SetPristine` 前）；7 个 Brain 挂逃跑节点（grep 核验行号：shadowcreaturebrain 88、nightmarecreaturebrain 46、nightmaremonkeybrain 27、oceanshadowcreaturebrain 96、shadow_leechbrain 35、hostedbrain 123、charlie_boss_runnerbrain 44）；modmain 须经 `GLOBAL.` 访问 | 本页源码入口如前；Brain 内部机制 Wiki 待建 |
| API／数据：生成／行为门控（同函数消费端） | 幻觉不在光域生成（[hallucinations.lua#L145-L158](../../../raw/current/scripts/components/hallucinations.lua#L145-L158)）；shadeling 光域内直接 `Despawn`（[ruins_shadeling.lua#L72-L75](../../../raw/current/scripts/prefabs/ruins_shadeling.lua#L72-L75)）；Grim Harrier 不索敌/不保持目标（[charlie_boss_runner.lua#L26-L36](../../../raw/current/scripts/prefabs/charlie_boss_runner.lua#L26-L36)）；影 parasites 不保持目标（[hats.lua#L5565-L5567](../../../raw/current/scripts/prefabs/hats.lua#L5565-L5567)）；海影不传送登船（[oceanshadowcreature.lua#L44-L45](../../../raw/current/scripts/prefabs/oceanshadowcreature.lua#L44-L45)）；影之手消散（[shadowhand.lua#L152-L156](../../../raw/current/scripts/prefabs/shadowhand.lua#L152-L156)）；怕光影兽消失（[shadowskittish.lua#L15-L19](../../../raw/current/scripts/prefabs/shadowskittish.lua#L15-L19)）；噩梦岩柱休眠（[nightmarerock.lua#L156-L159](../../../raw/current/scripts/prefabs/nightmarerock.lua#L156-L159)）；Wavey Jones 恐惧逃离（[waveyjones.lua#L66-L74](../../../raw/current/scripts/prefabs/waveyjones.lua#L66-L74)） | 各自 Update/retarget/KeepTarget 时机，详见锚点 | 各源码入口如前；暗影生物 Wiki 待建 |
| 关联 Prefab：`queen_torch_fx` | 装备时生成（[onequip](../../../raw/current/scripts/prefabs/queen_torch.lua#L92-L96)），`AttachToOwner`：SetParent＋`Follower:FollowSymbol(swap_object)`＋highlightchild/colouradder（[L315-L322](../../../raw/current/scripts/prefabs/queen_torch.lua#L315-L322)） | 装备态；卸下即 Remove；`persists=false` 不进存档 | [fxfn](../../../raw/current/scripts/prefabs/queen_torch.lua#L324-L351)；[[reference/components/highlightchild#关键接口]]、[[reference/components/colouradder#关键接口]] |
| 关联 Prefab：`queen_torchlight` | 装备/点燃时生成并 `SetParent`（[L98-L101](../../../raw/current/scripts/prefabs/queen_torch.lua#L98-L101)、[IgniteTossed](../../../raw/current/scripts/prefabs/queen_torch.lua#L135-L146)）；`OnRemoveEntity` 从 TORCHES 注销 | 光域判定与驱动声音都在该实体；休眠时停 `queentorch_LP` 循环音（[L353-L361](../../../raw/current/scripts/prefabs/queen_torch.lua#L353-L361)）；`persists=false`，存档只记 `lit`/`thrower` 由 OnLoad 重建（[L212-L228](../../../raw/current/scripts/prefabs/queen_torch.lua#L212-L228)） | [torchlightfn](../../../raw/current/scripts/prefabs/queen_torch.lua#L370-L404) |
| 资源：anim／声音／皮肤 | `anim/queen_torch.zip`（bank/build，含 `swap_queen_torch` 持握符号、`spin_loop` 投掷动画）、`sound/common.fsb`、小地图 `queen_torch.png`；皮肤 `queen_torch_lotus`（暗花火炬 Dark Bloom Torch）经 `OverrideItemSkinSymbol` 与带皮肤 fx 生效 | 皮肤注册与名单：[skinprefabs.lua#L9830-L9835](../../../raw/current/scripts/prefabs/skinprefabs.lua#L9830-L9835)、[prefabskins.lua#L1518-L1521](../../../raw/current/scripts/prefabskins.lua#L1518-L1521)；中文名 [po SKIN_NAMES](../../../raw/current/scripts/languages/chinese_s.po#L406394-L406397)、描述 [po SKIN_DESCRIPTIONS](../../../raw/current/scripts/languages/chinese_s.po#L385669-L385672) | 资源本体不分发；皮肤换装写法见 [onequip](../../../raw/current/scripts/prefabs/queen_torch.lua#L82-L88) |

## 典型接入

### 给女王火炬添加合成配方

- 前提与选择：服务端/客户端通用 `modmain.lua`。原版该物无配方且唯一来源在终局仪式之后，模组常需提前可获取；`AddRecipe2` 是配方注册入口，未传 filters 时自动入 MODS 分类（[modutil.lua#L732-L747](../../../raw/current/scripts/modutil.lua#L732-L747)）。modmain 环境只直挂 `TUNING` 等白名单（[CreateEnvironment L258-L317](../../../raw/current/scripts/mods.lua#L258-L317)，无 `__index=_G`），`Ingredient`/`TECH`/`TheWorld` 等必须 `GLOBAL.` 前缀。
- 生效与撤销：注册进全局 recipes 后立即出现在制造栏；停用模组即消失，无残留。

```lua
-- modmain.lua：女王火炬可合成（材料与科技为示例，按需调整）
local Ingredient = GLOBAL.Ingredient
local TECH = GLOBAL.TECH

AddRecipe2("queen_torch", {
    Ingredient("twigs", 2),
    Ingredient("nightmarefuel", 2),
}, TECH.NONE)
```

- 调整与验证：配方产物默认与注册名同名；改科技档查 `TECH` 常量，改分类传 `filters` 参数（[AddRecipe2](../../../raw/current/scripts/modutil.lua#L732-L747)）。物品本体行为不受影响——永燃/驱影/投掷逻辑都在 Prefab 内，无需 postinit。

## 常见陷阱与限制

- **误区：女王火炬由蜂后（Bee Queen）掉落**：名字易联想蜂后，但 beequeen.lua 的 lootdropper 及全 scripts 掉落链均无 `queen_torch`；唯一获取是远古大门 Charlie 战败仪式布景（[atrium_gate.lua#L715](../../../raw/current/scripts/prefabs/atrium_gate.lua#L715)）。"Queen" 指远程古之门王家遗物，与 `king_cane`、`mask_charlie_bosshat` 同组。
- **边界：永燃与不可点燃**：无 fueled 组件、burnable 无 burntime → 燃烧永不自动结束，`burnable.canlight=false` 挡住 LIGHT 动作与闪电等外部点火；唯一灭火路径是卸下/入袋（[onunequip](../../../raw/current/scripts/prefabs/queen_torch.lua#L107-L129)、[OnPutInInventory](../../../raw/current/scripts/prefabs/queen_torch.lua#L193-L206)），雨与冰不消耗它。
- **边界：`waterproofer` 标签没有对应组件**：公共端照 torch 惯例加了标签（[L249](../../../raw/current/scripts/prefabs/queen_torch.lua#L249)）但从未 `AddComponent("waterproofer")`，全 scripts 也没有直接读该标签的消费端——装备它不提供任何防水。
- **边界：光半径有新建 6／装备 5×修正／投掷重建三条路径**：`queen_torchlight` 新建时 `SetRadius(6)`；装备后 `RefreshAttunedSkills` 立即覆写为 `5×修正`（无技能修正即 5）。装备态投掷时 `TOSS` 先 `DropItem`→强制卸装，`onunequip` 移除原光源并灭火，随后 `OnDropped` 的 `IgniteTossed` 新建光（初始 6），`Launch`→`OnThrown` 再按 thrower 覆写 `5×修正`——原装备光源不会保留。`OnThrown` 只按投掷者是否有 `skilltreeupdater` 记录 thrower；有该组件就写记录，未点光圈技能也以倍率 1 覆写成 5。改半径要同时覆盖新建、装备刷新与投掷。[装备刷新](../../../raw/current/scripts/prefabs/queen_torch.lua#L58-L65)、[卸装清理](../../../raw/current/scripts/prefabs/queen_torch.lua#L107-L129)、[投掷](../../../raw/current/scripts/prefabs/queen_torch.lua#L135-L155)、[入袋重建](../../../raw/current/scripts/prefabs/queen_torch.lua#L193-L210)、[初始半径](../../../raw/current/scripts/prefabs/queen_torch.lua#L370-L385)
- **边界：投掷中不可拾取、会被 Charlie 弹回**：OnLaunch 即 `canbepickedup=false`，落地 OnHit 恢复；在 Charlie 战斗域内投掷会被 `Deflect` 弹向反方向 3 单位（`SetKeepOnDeflect(true)` 保证不销毁，[OnDeflect](../../../raw/current/scripts/prefabs/queen_torch.lua#L168-L182)）。
- **边界：存档只记地面点燃态**：仅“燃烧且不在背包”时写 `lit`/`thrower`（[OnSave](../../../raw/current/scripts/prefabs/queen_torch.lua#L212-L220)）；背包态与装备态重载后按组件默认重建（装备态重新点燃由 onequip 执行）。

## 验证记录

- 源码审阅：2026-11-07（第二轮检视修订），Steam buildid 26335394。复核 `IgniteTossed` 的光实体复用与 `thrower` 记录条件、装备态 `RefreshAttunedSkills` 的半径覆写、入袋后重建路径。核验 prefabs/queen_torch.lua 全文（三 Prefab 工厂、标签、组件顺序、皮肤写法、OnSave/OnLoad、TORCHES 注册与全局查询）；equippable（HANDS 默认）、weapon（SetDamage/GetDamage）、burnable（Ignite/Extinguish/fire 标签镜像/fueled 联动）、complexprojectile（Launch/Deflect/ismelee 注释）、inventoryitem 回调、stalkerinspectable、skilltreeupdater IsActivated；驱影消费端逐个读上下文（hallucinations/ruins_shadeling/charlie_boss_runner/hats/oceanshadowcreature/shadowhand/shadowskittish/nightmarerock/waveyjones），逃跑 Brain 名单经全库 grep `RunAwayFromQueenTorch`（7 处，未逐文件读上下文）；掉落链经全库 grep `queen_torch`（仅 atrium_gate 生成）与 beequeen.lua loot 检查；tuning 键实际值与换算（wilson_attack=34×1）；TOSS 动作与 componentactions 点动作收集；名称/检视语/图鉴/皮肤各 po 键；Charlie 弹回、瘴气 TestFireFn、stalkerbrain 消费、皮肤注册与名单。未覆盖：7 个逃跑 Brain 的调用上下文细节、charlie_boss 反射域参数全貌（仅读消费点）、暗影生物 AI 内部机制。
- 游戏运行：未运行。未覆盖：实机光半径与驱影范围、投掷轨迹与偏转表现、皮肤外观、技能档半径实际观感（均为源码推导值）。
- 结构检查：2026-11-07 validate_page.py 0 issue（107 行 / 23362 字节，standard 档内）与 validate_source_links.py --strict 0 error 0 warning（89 处行引用、37 个源文件）。

返回 [[reference/prefabs/tools-field-index|工具与野外实用]]、[[reference/prefabs/index|Prefab 索引]]；同批照明对照 [[reference/prefabs/torch|火炬（torch）]]、[[reference/prefabs/lantern|提灯（lantern）]]。
