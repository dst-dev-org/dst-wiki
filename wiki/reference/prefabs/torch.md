---
title: "火炬（torch）"
created: "2026-11-06"
updated: "2026-11-08"
type: prefab
summary: "原版手持照明装备 torch：HANDS 槽近战武器＋点火工具，点燃由 torchfire 跟随 FX 照明，fueled 随燃烧计时耗尽即毁，支持幸运掷点攻击引燃与右键投掷落地持续照明。"
aliases: ["torch", "火炬", "火把", "照明武器", "投掷火把"]
tags: [prefab, item, equipment]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/torch.lua", "prefabs/torchfire.lua", "prefabs/torchfire_common.lua", "components/weapon.lua", "components/equippable.lua", "components/fueled.lua", "components/lighter.lua", "components/burnable.lua", "actions.lua", "componentactions.lua", "componentutil.lua", "prefabs/player_common.lua", "components/wisecracker.lua", "prefabs/wargshrine.lua", "components/wildfires.lua", "prefabs/charlie_boss_trial.lua", "prefabskin.lua", "standardcomponents.lua", "tuning.lua", "recipes.lua", "constants.lua", "mods.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 火炬（torch）

## 快速判断

- 职责与适用：`prefabs/torch.lua` 顶层 `fn` 直接注册的单注册可拾取手持装备（物品＋装备）：HANDS 槽近战武器＋`lighter` 点火工具；点燃由独立 FX Prefab `torchfire` 跟随宿主照明，燃料随燃烧计时、耗尽整件销毁。同文件无 quagmire 分支（全 scripts grep 无 torch 专属变体注册）。[注册](../../../raw/current/scripts/prefabs/torch.lua#L415-L530)
- 优先使用：复刻“燃料驱动照明＋点燃能力”按[关联机制导航](#关联机制导航)组装并查 [[cookbook/equipment/hand]]；改燃烧时长／照明／伤害查[数值表](#注册与构造)；给已有火把挂行为查[典型接入](#典型接入)。
- 重要副作用：装备即 `burnable:Ignite`→`fueled:StartConsuming` 计时消耗，雨中按降水率加速；耗尽整件 `Remove` 并广播 `itemranout`（自动补装＋播报）。攻击按目标可燃度掷点引燃（Willow 技能免掷必燃）；右键 TOSS 抛出点燃（Wilson `wilson_torch_7` 门控，见[关联动作](#关联动作)）、落地持续照明至耗尽 `ErodeAway`；地面燃烧态仍可被灭火器／水球等外部熄灭，移除跟随 FX、动画回 idle 并弹起本体，与燃料耗尽的消失路径不同。[外部灭火](../../../raw/current/scripts/prefabs/torch.lua#L369-L392)[onfuelchange](../../../raw/current/scripts/prefabs/torch.lua#L232-L268)、[onattack](../../../raw/current/scripts/prefabs/torch.lua#L205-L210)
- 修改边界：标签与 floater 挂公共端、组件挂主模拟（[分界](../../../raw/current/scripts/prefabs/torch.lua#L429-L455)）；照明 Light 与粒子都在 `torchfire` 实体上而非本体组件；皮肤 FX 名单在数据侧 `SKIN_FX_PREFAB`，本页不含皮肤数据内容。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `torch` | 无包装工厂：`fn()` 直接构造，无 postinit | 物品＋装备；`torchfire` 是独立 FX Prefab（声明于 `prefabs` 数组且装备／投掷时实际 `SpawnPrefab`）[注册](../../../raw/current/scripts/prefabs/torch.lua#L530)、[声明](../../../raw/current/scripts/prefabs/torch.lua#L8-L11) |

共同初始化（[torch.lua#L415-L528](../../../raw/current/scripts/prefabs/torch.lua#L415-L528)）：实体组件＋`MakeInventoryPhysics` → bank `torch`＋build `swap_torch`＋`idle` → 公共端标签 `wildfireprotected`／`lighter`／`waterproofer`／`weapon`／`projectile`／`complexprojectile`／`special_action_toss`／`keep_equip_toss`／`torch` → `MakeInventoryFloatable("med", nil, 0.68)` → `SetPristine`（非主模拟返回）→ 主模拟依次 weapon → lighter → inventoryitem → equippable → complexprojectile → waterproofer → inspectable → burnable → fueled → 技能监听与雨监听 → `MakeHauntableLaunch` → OnSave／OnLoad／OnRemoveEntity。装备槽用组件默认 HANDS。[equippable 默认](../../../raw/current/scripts/components/equippable.lua#L29)、[EQUIPSLOTS.HANDS](../../../raw/current/scripts/constants.lua#L656)

名称键经简中 .po `msgctxt` 核验：`STRINGS.NAMES.TORCH`＝火炬（Torch）；通用检视语＝“驱赶黑暗的东西。”。[.po 名称](../../../raw/current/scripts/languages/chinese_s.po#L350479-L350482)、[.po 检视语](../../../raw/current/scripts/languages/chinese_s.po#L20835-L20837)

数值（tuning 键＋实际值；`wilson_attack`＝34×倍率 1、`night_time`＝30 秒×2 段＝60 秒，[tuning.lua#L19-L37](../../../raw/current/scripts/tuning.lua#L19-L37)）：

| 项 | 键 | 实际值 |
|---|---|---|
| 武器伤害 | `TUNING.TORCH_DAMAGE`＝`wilson_attack`×.5 | 17 生命值（源码推导，[tuning.lua#L361](../../../raw/current/scripts/tuning.lua#L361)） |
| 燃料总量 | `TUNING.TORCH_FUEL`＝`night_time`×1.25 | 75 燃料单位＝rate 1 下燃烧 75 秒（[tuning.lua#L1714](../../../raw/current/scripts/tuning.lua#L1714)） |
| 雨中消耗增速 | `TUNING.TORCH_RAIN_RATE` | 1.5×降水率，rate 变为 `1+该值`，满雨 2.5（[tuning.lua#L1713](../../../raw/current/scripts/tuning.lua#L1713)、[雨衰接线](../../../raw/current/scripts/prefabs/torch.lua#L212-L218)） |
| 照明半径／衰减 | `TUNING.TORCH_RADIUS[1]`／`TUNING.TORCH_FALLOFF[1]` | 2／0.5（技能档可升至半径 3~5，[tuning.lua#L7043-L7055](../../../raw/current/scripts/tuning.lua#L7044-L7056)） |
| 光强／颜色 | torchfire_common 硬编码 | 0.75；(180,195,150)/255（[CreateLight](../../../raw/current/scripts/prefabs/torchfire_common.lua#L14-L17)） |
| 攻击点燃概率 | `TUNING.TORCH_ATTACK_IGNITE_PERCENT` | 1 ×目标 `burnable.flammability`，经幸运公式 `LighterIgniteOnAttack` 修正（[tuning.lua#L421](../../../raw/current/scripts/tuning.lua#L421)、[公式](../../../raw/current/scripts/componentutil.lua#L2150)） |
| 点燃后首扣时长 | `TUNING.TURNON_FUELED_CONSUMPTION`／`TURNON_FULL_FUELED_CONSUMPTION` | 0.5 秒量／满燃料 1/60 秒量（[tuning.lua#L1668-L1669](../../../raw/current/scripts/tuning.lua#L1668-L1669)） |
| 防水效果 | `TUNING.WATERPROOFNESS_SMALL` | 0.2（[tuning.lua#L2949](../../../raw/current/scripts/tuning.lua#L2949)） |
| 投掷参数 | complexprojectile 配置 | speed 15、gravity −35、出手偏移 (0.25, 1, 0)（[挂载](../../../raw/current/scripts/prefabs/torch.lua#L479-L487)、[偏移](../../../raw/current/scripts/prefabs/torch.lua#L325)） |
| Wilson 技能档 | `TUNING.SKILLS.WILSON_TORCH_1~3`／`_4~6` | 燃料倍率 0.84/0.68/0.5；光档索引 2/3/4（半径 3/4/5，[tuning.lua#L7057-L7066](../../../raw/current/scripts/tuning.lua#L7058-L7067)） |

获取途径与消费端：

| 途径 | 证据 |
|---|---|
| 配方：cutgrass 2＋twigs 2，`TECH.NONE`（免科技，[constants.lua#L1241-L1243](../../../raw/current/scripts/constants.lua#L1241-L1243)） | [配方](../../../raw/current/scripts/recipes.lua#L425) |
| 消费端：wargshrine 神龛火把设 `SetFuelRateMult(TORCH_SHRINE_FUEL_RATE_MULT=.2)`（5 倍慢烧）、取回时恢复 1 | [wargshrine](../../../raw/current/scripts/prefabs/wargshrine.lua#L159-L161)、[tuning](../../../raw/current/scripts/tuning.lua#L1715) |
| 消费端：charlie_boss_trial 影之手按 `torch` 标签＋燃烧状态寻光目标 | [charlie_boss_trial](../../../raw/current/scripts/prefabs/charlie_boss_trial.lua#L310-L313) |

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：weapon | `SetDamage(TORCH_DAMAGE)`＋`SetOnAttack(onattack)`；无 finiteuses、无磨损、无射程 | 主模拟；onattack 见下“攻击引燃” | [[reference/components/weapon#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/torch.lua#L457-L459)、[OnAttack 调用点](../../../raw/current/scripts/components/weapon.lua#L105-L108) |
| 组件：lighter | 挂载组件（标签在公共端镜像）；`Light(target, doer)`→目标 `burnable:Ignite` | 服务端动作 `LIGHT`：收集要求目标 `canlight` 标签（[收集](../../../raw/current/scripts/componentactions.lua#L1480-L1484)），手持 lighter 对 `canlight`/`nolight` 目标不给 ATTACK（[抑制](../../../raw/current/scripts/componentactions.lua#L1993)） | [[reference/components/lighter#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/torch.lua#L462)、[Light](../../../raw/current/scripts/components/lighter.lua#L18-L26)、[动作执行](../../../raw/current/scripts/actions.lua#L2689-L2697) |
| 组件：equippable | 全套回调：OnEquip／OnUnequip／OnPocket／OnEquipToModel | 主模拟；槽默认 HANDS；ToPocket 在入袋时触发（[ToPocket](../../../raw/current/scripts/components/equippable.lua#L109-L113)） | [[reference/components/equippable#关键接口]]；[接线](../../../raw/current/scripts/prefabs/torch.lua#L471-L475) |
| 组件：fueled | `InitializeFuelLevel(TORCH_FUEL)`、耗尽回调 `inst.Remove`、分段回调 onfuelchange；`accepting=false`→不可补燃料，仅燃烧消耗 | 主模拟；Start/Stop 由 burnable Ignite/Extinguish 联动（[启动](../../../raw/current/scripts/components/burnable.lua#L380-L382)、[停止](../../../raw/current/scripts/components/burnable.lua#L486-L488)）；雨衰经 `SetUpdateFn` 改 rate | [[reference/components/fueled#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/torch.lua#L507-L511)、[消耗](../../../raw/current/scripts/components/fueled.lua#L325-L337) |
| 组件：burnable | 状态管理载体：`canlight=false`（无 `canlight` 标签）、`fxprefab=nil`（Ignite 无自身 FX）、`SetOnExtinguishFn(OnExtinguish)` | 主模拟；外部点火／闪电路径被 canlight 挡住（[闪电压](../../../raw/current/scripts/componentutil.lua#L1080-L1082)）；穿脱时 Ignite/Extinguish | [[reference/components/burnable#关键接口]]；[配置](../../../raw/current/scripts/prefabs/torch.lua#L500-L503) |
| 组件：complexprojectile | TOSS 投掷：OnLaunch 点燃＋自旋、OnHit 落地、OnDeflect 可偏转且 `SetKeepOnDeflect(true)` | 主模拟；`ismelee weapon` 属性 true；动作见[关联动作](#关联动作) | [[reference/components/complexprojectile#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/torch.lua#L479-L487) |
| 组件：waterproofer／inspectable／inventoryitem | `SetEffectiveness(0.2)`；检视；入袋灭火回 idle＋`SetOnPickupFn(RemoveThrower)` | 主模拟 | [[reference/components/waterproofer#关键接口]]、[[reference/components/inspectable#关键接口]]、[[reference/components/inventoryitem#关键接口]]；[waterproofer](../../../raw/current/scripts/prefabs/torch.lua#L491-L492)、[inventoryitem](../../../raw/current/scripts/prefabs/torch.lua#L465-L467) |
| 组件：hauntable | `MakeHauntableLaunch`：作祟弹飞 | 主模拟 | [[reference/components/hauntable#按效果选工厂]]；[调用](../../../raw/current/scripts/prefabs/torch.lua#L521) |
| 组件：floater | 公共端 `MakeInventoryFloatable("med", nil, 0.68)`：中等浮态、缩放 0.68、无换 bank | 公共端 | [[reference/components/floater#关键接口]]；[调用](../../../raw/current/scripts/prefabs/torch.lua#L449)、[helper](../../../raw/current/scripts/standardcomponents.lua#L1772-L1790) |
| Replica／同步：`torch._lightrange` | torchfire 用 `net_tinybyte` 同步光档，客户端 `lightrangedirty`→`Light:SetRadius/Falloff`；服务端走 `SetLightRange` | 光档索引 1~4 查 TORCH_RADIUS/TORCH_FALLOFF 表 | [[reference/components/fueled#关键接口]]（雨衰驱动方）；[net](../../../raw/current/scripts/prefabs/torchfire_common.lua#L86-L92)、[dirty](../../../raw/current/scripts/prefabs/torchfire_common.lua#L42-L45) |
| API／事件：`itemranout` | 耗尽（装备态）先 `Remove` 再 `PushEvent`，data 带 `announce="ANNOUNCE_TORCH_OUT"` | 玩家侧监听：自动补装同 prefab 工具（[player_common](../../../raw/current/scripts/prefabs/player_common.lua#L612-L623)）＋wisecracker 播报（[wisecracker](../../../raw/current/scripts/components/wisecracker.lua#L68-L70)） | 源码入口如前；[耗尽块](../../../raw/current/scripts/prefabs/torch.lua#L238-L250) |
| API／数据：`torch` 标签 | 拾取燃烧物判定豁免；影之手寻光 | 服务端／客户端均可读 | [拾取例外](../../../raw/current/scripts/actions.lua#L904)、[影之手](../../../raw/current/scripts/prefabs/charlie_boss_trial.lua#L310-L313) |
| API／数据：`wildfireprotected` 标签 | 夏季野火事件排除该实体 | [排除表](../../../raw/current/scripts/components/wildfires.lua#L27) | 源码入口如前 |
| API／数据：Wilson 技能 | 监听宿主 `onactivateskill_server`／`ondeactivateskill_server` 刷新：燃料倍率进 `fueled.rate_modifiers`，光档进 torchfire `SetLightRange` | 需持有者 skilltreeupdater（[[reference/components/skilltreeupdater#关键接口]]） | [刷新](../../../raw/current/scripts/prefabs/torch.lua#L77-L128)、[监听](../../../raw/current/scripts/prefabs/torch.lua#L118-L128) |
| 关联 Prefab：`torchfire` | 火焰 FX 实体（见下方简述）；无皮肤时用默认 `{ "torchfire" }`，有皮肤时只取 `SKIN_FX_PREFAB[皮肤名]`，缺键即空表（不生成火焰与照明） | 装备/投掷点燃时 `SpawnPrefab`，卸下/入袋/耗尽 `Remove` | [[reference/components/burnable#关键接口]]（Ignite 联动）；[生成](../../../raw/current/scripts/prefabs/torch.lua#L145-L160)、[皮肤 FX 注册](../../../raw/current/scripts/prefabskin.lua#L4094-L4096) |
| 资源：anim／swap／贴图 | torch.zip（bank `torch`）／swap_torch.zip（build＋持握符号 `swap_object`）／common.fsb；torchfire 粒子贴图 fx/torchfire.tex、fx/smoke.tex | 静态声明；皮肤 build 经 `GetSkinBuild` 覆盖持握符号 | [声明](../../../raw/current/scripts/prefabs/torch.lua#L1-L6)、[torchfire 声明](../../../raw/current/scripts/prefabs/torchfire.lua#L3-L17) |

### torchfire（火焰 FX 实体，同页简述）

`torchfire` 由工厂 `MakeTorchFire("torchfire", ...)` 注册（[torchfire.lua#L184](../../../raw/current/scripts/prefabs/torchfire.lua#L184)、[工厂](../../../raw/current/scripts/prefabs/torchfire_common.lua#L54-L117)）：纯 FX 实体（`FX` 标签、`persists=false`、不进存档）＋循环音 `dontstarve/wilson/torch_LP`；子 Light 实体非联网、`persists=false`，光参数如上数值表；`AttachLightTo(target)` 把 Light 改挂宿主并按宿主是否 player 加／去 `playerlight` 标签（[AttachLightTo](../../../raw/current/scripts/prefabs/torchfire_common.lua#L22-L29)）；粒子为烟＋火两个 VFX emitter，Dedicated 服务器直接跳过（[common_postinit](../../../raw/current/scripts/prefabs/torchfire.lua#L109-L117)）；跟随符号偏移 `fx_offset=-110`（[master_postinit](../../../raw/current/scripts/prefabs/torchfire.lua#L180-L182)）。皮肤可提供自定义 FX Prefab 并经 `AssignSkinData` 传数据（[调用点](../../../raw/current/scripts/prefabs/torch.lua#L148-L156)）；皮肤换贴图走 `equipskinneditem`／`unequipskinneditem` 事件与 `OverrideItemSkinSymbol`（[onequip](../../../raw/current/scripts/prefabs/torch.lua#L133-L139)、[onunequip](../../../raw/current/scripts/prefabs/torch.lua#L167-L170)）。

### 关联动作

- `LIGHT`（点火，`Action({ priority=-4 })`）：目标有 `canlight` 且非 `fueldepleted`/`INLIMBO` 时由手持 `lighter` 提供；fn 先发 `onstartedfire` 再调 `Light`，返回 true 不代表目标真的烧起来。[注册](../../../raw/current/scripts/actions.lua#L381)、[fn](../../../raw/current/scripts/actions.lua#L2689-L2697)、机制详见 [[reference/components/lighter#关键接口]]。
- `TOSS`（投掷，右键、距离 8）：两处通用收集均排除 `special_action_toss` 物品（[L2074](../../../raw/current/scripts/componentactions.lua#L2074)、[L2285](../../../raw/current/scripts/componentactions.lua#L2285)）——火把的 TOSS 仅由 Wilson 技能 `wilson_torch_7` 经 GetPointSpecialActions 给出（[wilson.lua:29-42](../../../raw/current/scripts/prefabs/wilson.lua#L29-L42)、[注册 L64](../../../raw/current/scripts/prefabs/wilson.lua#L64)：HANDS 装备的 `torch`＋`special_action_toss` 物品右键地面；背包内火把无收集路径）；fn 的 `invobject` 兜底也只读 HANDS（[fn](../../../raw/current/scripts/actions.lua#L3953-L3999)）；`keep_equip_toss` 让 TOSS 走 `throw_keep_equip` 状态并抑制通用「先装备再动作」（[playercontroller.lua:4675](../../../raw/current/scripts/components/playercontroller.lua#L4675)、[SGwilson.lua:1429](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1429)）：动作物品不在 HANDS 时状态内先补装备、第 7 帧投出（DropItem＋Launch）后恢复原装备——物品本身仍被抛出。`complexprojectile:Launch` 触发 OnThrown 点燃自旋、落地 OnHit。 [TOSS 定义](../../../raw/current/scripts/actions.lua#L501)

### 攻击引燃（onattack）

攻击命中时服务端执行序：combat → `weapon:OnAttack` → onattack（[weapon.lua#L105-L108](../../../raw/current/scripts/components/weapon.lua#L105-L108)）。onattack（[torch.lua#L205-L210](../../../raw/current/scripts/prefabs/torch.lua#L205-L210)）：目标有效且有 burnable 时，`TryLuckRoll(attacker, 1×目标 flammability, LuckFormulas.LighterIgniteOnAttack)` 成功或攻击者已激活 `willow_controlled_burn_1`，则 `burnable:Ignite(nil, attacker)` 点燃目标。掷点公式 `LighterIgniteOnAttack=CommonChanceUnluckMultAndLuckHyperbolic(.5)`（[componentutil.lua#L2150](../../../raw/current/scripts/componentutil.lua#L2150)），幸运值经 `TryLuckRoll` 换算（[componentutil.lua#L2237-L2251](../../../raw/current/scripts/componentutil.lua#L2237-L2251)）。表达式先掷点、失败后才读 `attacker.components.skilltreeupdater`，没有该组件的自定义攻击者会在此处 nil 索引；打火机同类回调有显式判空（[lighter.lua#L231-L239](../../../raw/current/scripts/prefabs/lighter.lua#L231-L239)），两个入口不能当同一模板照搬。无 finiteuses，攻击不耗耐久；燃料只随时间消耗。

## 典型接入

### 让火把燃烧时间加倍（速率减半）

- 前提与选择：服务端 `modmain.lua`。torch 预留运行时钩子 `inst.SetFuelRateMult`（[定义](../../../raw/current/scripts/prefabs/torch.lua#L270-L277)、[挂载](../../../raw/current/scripts/prefabs/torch.lua#L518-L519)），原版 wargshrine 即此用法（[wargshrine](../../../raw/current/scripts/prefabs/wargshrine.lua#L159-L161)）；倍率在无雨与雨衰两条 rate 计算里都生效，比直接改 `fueled.rate` 更不容易被下雨逻辑覆盖。modmain 环境无 `TheWorld` 全局回退，须 `GLOBAL.` 前缀；`TUNING` 在白名单内可直用（[CreateEnvironment](../../../raw/current/scripts/mods.lua#L258-L292)）。
- 生效与撤销：postinit 只影响新实例；`_fuelratemult` 不进存档，但存档重载重建实例时会重新执行 postinit 重挂。停用模组恢复原值。

```lua
-- modmain.lua：火把燃烧时长加倍（雨衰同比例减慢）
AddPrefabPostInit("torch", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    if inst.SetFuelRateMult ~= nil then
        inst:SetFuelRateMult(0.5)
    end
end)
```

- 调整与验证：倍率乘在 fueled.rate 上，0.5＝75→150 秒（满雨 30→60 秒）；Wilson 技能燃料修正在 `rate_modifiers`，与本倍率相乘（[DoUpdate](../../../raw/current/scripts/components/fueled.lua#L325-L337)）。点亮状态存档字段 `lit`／`thrower` 不受影响（[OnSave](../../../raw/current/scripts/prefabs/torch.lua#L395-L405)）。

## 常见陷阱与限制

- **误区：火把可以像篝火一样被点燃或续燃料**：`burnable.canlight=false`→无 `canlight` 标签，LIGHT 动作收集不到它，闪电等外部点火也检查 `canlight`；`fueled.accepting=false`→无 `burnable_fueled` 标签，ADDFUEL 不可用。燃料只能靠装备燃烧消耗，不能中途续。[canlight tag](../../../raw/current/scripts/components/burnable.lua#L6-L8)、[配置](../../../raw/current/scripts/prefabs/torch.lua#L500-L511)
- **边界：点燃的火把不向周围传播燃烧**：本体无 propagator 组件（`MakeSmallPropagator` 未用），`burnable:Ignite` 的 `StartSpreading` 分支不触达；点燃环境只有攻击掷点与 LIGHT 动作两条路径。[Ignite 分支](../../../raw/current/scripts/components/burnable.lua#L384-L386)
- **边界：照明 Light 是 torchfire 的非联网子实体**：`persists=false`，Dedicated 服务器不生成粒子；客户端光参数靠 `_lightrange` net 同步，直接改本地 Light 或只改服务端数值不会生效，改半径应经 `SetLightRange` 或 TORCH_RADIUS 表。[net 接线](../../../raw/current/scripts/prefabs/torchfire_common.lua#L86-L92)
- **边界：耗尽先 `Remove` 再发 `itemranout`**：自动补装依赖此顺序（原注释 "need to remove before itemranout for auto-reequip to work"）；装备态耗尽播报 `ANNOUNCE_TORCH_OUT`，地面点燃耗尽 `ErodeAway` 淡出。[耗尽块](../../../raw/current/scripts/prefabs/torch.lua#L232-L268)
- **边界：投掷落地照明不是只等烧完**：地面燃料未空时，灭火器、水球、冰杖等外部灭火会走 `OnExtinguish`——移除跟随 FX、播放 idle、弹起本体；只有燃料耗尽才 `ErodeAway`。两者不要合并描述。[外部灭火](../../../raw/current/scripts/prefabs/torch.lua#L369-L392)、[耗尽分支](../../../raw/current/scripts/prefabs/torch.lua#L254-L265)
- **边界：投掷点燃态不进背包**：OnThrown 后 `canbepickedup=false`＋`FX` 标签防选中，落地 OnHit 恢复可拾取；拾取时 OnPutInInventory 立即灭火回 idle，存档只保存地面点燃态（`lit`/`thrower`）。[OnThrown](../../../raw/current/scripts/prefabs/torch.lua#L304-L315)、[OnPutInInventory](../../../raw/current/scripts/prefabs/torch.lua#L354-L367)

## 验证记录

- 源码审阅：2026-11-07（第二轮检视修订），Steam buildid 26335394。复核皮肤 FX 回退分支、onattack 的 skilltreeupdater 读取顺序与 OnExtinguish 外部灭火路径；早前 2026-11-06 核验 prefabs/torch.lua 全文、torchfire.lua／torchfire_common.lua 全文，weapon／equippable／fueled／lighter 关键方法与默认值（equippable HANDS 默认、fueled rate=1／accepting=false／sections=1／period=1）、burnable Ignite／SpawnFX／canlight 标签、LIGHT／TOSS 动作与 componentactions 收集分支、TryLuckRoll 与 LuckFormulas.LighterIgniteOnAttack、tuning 各键实际值与换算、配方行、名称／检视语 po 键、消费端（wildfires 排除表、charlie_boss_trial 影之手、wargshrine 速率钩子、player_common 自动补装、wisecracker 播报）与 skins 工厂侧（torch_init_fn、SKIN_FX_PREFAB 注册）。
- 游戏运行：未运行。未覆盖：皮肤 fx 数据名单与实装外观、投掷轨迹与偏转实机表现、75 秒燃尽与雨衰为源码推导而非运行测量。
- 结构检查：validate_page.py 0 error（125 行，standard）与 validate_source_links.py --strict 0 error 0 warning 通过。

返回 [[reference/prefabs/tools-field-index|工具与野外实用]]、[[reference/prefabs/index|Prefab 索引]]；同族照明对照 [[reference/prefabs/lighter|薇洛的打火机]]、[[reference/prefabs/queen-torch|女王火炬]]（永燃驱影）与 [[reference/prefabs/redlantern|红灯笼]]（一次性燃料）。
- 2026-11-08 build 26474873：本页随版本迁移升版（锚点迁移 2 处），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-12）。
