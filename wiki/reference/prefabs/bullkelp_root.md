---
title: "公牛海带茎（bullkelp_root）"
created: "2026-11-06"
updated: "2026-11-07"
type: prefab
summary: "原版海洋手持武器 bullkelp_root（公牛海带茎）：伤害 27.2、射程 2 与 whip 同档，无 finiteuses——新鲜度即耐久，每次命中折损 2%–4%，剩余新鲜度越低越易整件折断自毁（≤0 档 1.0，幸运降低折断概率）；可种回水面重生为空牛海带植物；单注册，无 quagmire 分支。"
aliases: ["bullkelp_root", "公牛海带茎", "Bull Kelp Stalk", "牛海带根", "海带茎"]
tags: [prefab, item, equipment]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/bullkelp_root.lua", "components/weapon.lua", "components/equippable.lua", "components/perishable.lua", "components/pickable.lua", "components/stackable.lua", "components/deployable.lua", "components/inspectable.lua", "components/inventoryitem.lua", "components/inventoryitem_replica.lua", "components/inventory.lua", "componentutil.lua", "standardcomponents.lua", "stategraphs/SGwilson.lua", "stategraphs/SGwilson_client.lua", "prefabs/bullkelp_plant.lua", "prefabs/bullkelp_beached.lua", "prefabs/cannonballs.lua", "prefabs/mortarball.lua", "prefabs/trident.lua", "prefabs/winona_catapult_projectile.lua", "components/oceanicemanager.lua", "prefabs/oceanfishableflotsam.lua", "prefabs/otterden.lua", "tuning.lua", "constants.lua", "mods.lua", "screens/redux/scrapbookdata.lua", "widgets/itemtile.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 公牛海带茎（bullkelp_root）

## 快速判断

- 职责与适用：`prefabs/bullkelp_root.lua` 顶层 `fn` 注册的单注册可拾取手持武器（物品＋装备）：weapon 伤害＋equippable HANDS 槽，与 [[reference/prefabs/whip]] 同挂 `whip` 标签共用鞭击动画；特色是“新鲜度即耐久”：无 finiteuses，perishable 承担磨损与折断；deployable 只能种回水面。无 tool、无投射、无 SG／Brain，同文件无 quagmire 分支。[工厂](../../../raw/current/scripts/prefabs/bullkelp_root.lua#L66-L128)
- 优先使用：改伤害／折断／磨损查[数值与消耗](#数值与消耗)；复刻“新鲜度武器、命中掷点自毁”读[命中折断](#命中折断)与[典型接入](#典型接入)；从零造同类手持武器查 [[cookbook/equipment/hand]]。
- 重要副作用：每次命中先按 2%–4% 折损新鲜度，再按剩余新鲜度掷“折断”（≤0 档 1.0、<0.02 档 0.3、<0.5 档 0.1，幸运降低概率），折断在目标处生成 [[reference/prefabs/impact|impact]] 特效并下一帧移除装备的那一件；自然腐坏后整叠换 spoiled_food。[onattack](../../../raw/current/scripts/prefabs/bullkelp_root.lua#L27-L51)
- 修改边界：标签与 floater 挂公共端、组件挂主模拟（[分界](../../../raw/current/scripts/prefabs/bullkelp_root.lua#L85-L91)）；伤害／射程与 whip 同档但耐久机制不同（见[命中折断](#命中折断)）；本页不覆盖牛海带植物 bullkelp_plant（Wiki 待建，来源见[获取途径](#获取途径)）。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `bullkelp_root` | 无包装工厂：`fn()` 直接构造，无 postinit；同文件附带 `MakePlacer("bullkelp_root_placer", …)` 部署预览 | 物品＋装备；无皮肤注册。[注册](../../../raw/current/scripts/prefabs/bullkelp_root.lua#L130-L131) |

共同初始化（[bullkelp_root.lua#L66-L128](../../../raw/current/scripts/prefabs/bullkelp_root.lua#L66-L128)）：实体组件＋`MakeInventoryPhysics` → bank／build `bullkelp`＋`dropped` → 公共端 `MakeInventoryFloatable()`（默认 small，不换 bank）→ 公共端标签 `whip`、`show_spoilage`、`deployedplant` → `SetPristine`（非主模拟返回）→ 主模拟依次 stackable → inspectable → inventoryitem → weapon → equippable → fuel → perishable → deployable → `MakeMediumBurnable`＋`MakeSmallPropagator` → `MakeHauntableLaunchAndIgnite`。装备槽用组件默认 HANDS（[equippable.lua#L29](../../../raw/current/scripts/components/equippable.lua#L29)）。

名称键经简中 .po `msgctxt` 核验：`STRINGS.NAMES.BULLKELP_ROOT`＝公牛海带茎（Bull Kelp Stalk）；通用检视语 `DESCRIBE.BULLKELP_ROOT`＝“可以种在深海里。”。[.po 名称](../../../raw/current/scripts/languages/chinese_s.po#L339979-L339982)、[.po 检视语](../../../raw/current/scripts/languages/chinese_s.po#L9989-L9992)

### 数值与消耗

（tuning 键＋实际值；`wilson_attack`＝34×倍率 1，[tuning.lua#L31-L37](../../../raw/current/scripts/tuning.lua#L31-L37)）

| 项 | 键 | 实际值 |
|---|---|---|
| 武器伤害 | `TUNING.BULLKELP_ROOT_DAMAGE`＝`wilson_attack`×0.8 | 27.2 生命值（源码推导，[tuning.lua#L370](../../../raw/current/scripts/tuning.lua#L370)） |
| 武器攻击距离 | `TUNING.BULLKELP_ROOT_RANGE` | 武器加值 2（[tuning.lua#L429](../../../raw/current/scripts/tuning.lua#L429)）；实际范围＝玩家基础 3＋2＋目标物理半径，见 [[reference/prefabs/whip|whip 页]]攻击距离节 |
| 每击新鲜度折损 | `TUNING.BULLKELP_ROOT_USE`＋`random()×BULLKELP_ROOT_USE_VAR` | 0.02–0.04（即 2%–4%，[tuning.lua#L430-L431](../../../raw/current/scripts/tuning.lua#L430-L431)） |
| 折断基础概率 | `TUNING.BULLKELP_ROOT_SNAP_CHANCES` | HIGH 1.0／MED 0.3／LOW 0.1（[tuning.lua#L9356-L9360](../../../raw/current/scripts/tuning.lua#L9356-L9360)），档位见[命中折断](#命中折断) |
| 保质期 | `TUNING.PERISH_SUPERSLOW`＝40×`total_day_time`×`perish_warp` | 19200 秒（40 游戏日；[tuning.lua#L2051](../../../raw/current/scripts/tuning.lua#L2051)、[时间基准](../../../raw/current/scripts/tuning.lua#L19-L20)、[perish_warp](../../../raw/current/scripts/tuning.lua#L47)） |
| 堆叠上限 | `TUNING.STACK_SIZE_LARGEITEM` | 10（[tuning.lua#L81](../../../raw/current/scripts/tuning.lua#L81)） |
| 燃料值 | `TUNING.SMALL_FUEL`＝`seg_time`×0.5 | 15 秒（[tuning.lua#L1674](../../../raw/current/scripts/tuning.lua#L1674)） |
| 自燃时间 | `TUNING.LARGE_BURNTIME`＝`seg_time` | 30 秒（[tuning.lua#L1683](../../../raw/current/scripts/tuning.lua#L1683)） |
| 攻击磨损 | 无 finiteuses，`attackwear` 无消费路径 | 0 点耐久损耗；消耗完全走新鲜度（[weapon.lua#L113-L121](../../../raw/current/scripts/components/weapon.lua#L113-L121)） |

### 命中折断

服务端每次命中：combat 结算调用 `weapon:OnAttack`——先执行 onattack 回调、再走 finiteuses 磨损段（本武器无该组件，磨损段整段跳过；结算入口与 whip 相同，见 [[reference/prefabs/whip#命中超响鞭]]；[weapon.lua#L105-L121](../../../raw/current/scripts/components/weapon.lua#L105-L121)）。onattack（[bullkelp_root.lua#L27-L51](../../../raw/current/scripts/prefabs/bullkelp_root.lua#L27-L51)）：

1. **先折损**：`perishable:ReducePercent(0.02 + math.random()×0.02)`，每击掉 2%–4%（[ReducePercent](../../../raw/current/scripts/components/perishable.lua#L235-L240)，SetPercent 钳到 0–1）；手持与装备都不暂停保鲜（主人无 preserver／fridge 修正，[perishable.lua#L70-L95](../../../raw/current/scripts/components/perishable.lua#L70-L95)）。
2. **按剩余新鲜度取档**（[bullkelp_root.lua#L30-L34](../../../raw/current/scripts/prefabs/bullkelp_root.lua#L30-L34)）：≤0 → HIGH 1.0；<0.02 → MED 0.3；<0.5 → LOW 0.1；≥0.5 → 0（不做折断掷点）。
3. **掷点**：`TryLuckRoll(attacker, chance, BullKelpSnapChanceAdditive)`（[bullkelp_root.lua#L36](../../../raw/current/scripts/prefabs/bullkelp_root.lua#L36)、[TryLuckRoll](../../../raw/current/scripts/componentutil.lua#L2237-L2251)）。幸运公式（[bullkelp_root.lua#L22-L26](../../../raw/current/scripts/prefabs/bullkelp_root.lua#L22-L26)）：幸运 >0 → `chance×1/(1+luck)`（幸运降低折断），<0 → `chance×(1+|luck|)`（倒霉提高），=0 回退基础档（[GetEntityLuckChance](../../../raw/current/scripts/componentutil.lua#L2207-L2214)，幸运读攻击者 [[reference/components/luckuser#幸运值如何进入概率]]）。
4. **折断结算**（[bullkelp_root.lua#L40-L49](../../../raw/current/scripts/prefabs/bullkelp_root.lua#L40-L49)）：在目标处生成 `impact` 特效、按攻击者→目标方向旋转（`RADIANS`＝180/π，[constants.lua#L15](../../../raw/current/scripts/constants.lua#L15)）；在目标 `SoundEmitter` 播 `dontstarve/common/whip_small`（硬编码，无皮肤音）；`DoTaskInTime(0, inst.Remove)` 下一帧移除整件。未折断则本次命中再无其他消耗。

伤害与新鲜度无关（见[常见陷阱](#常见陷阱与限制)）。新鲜度被这一击打到 0 时先按 HIGH 档掷点，未折断才会在下一次保鲜 tick 整体换 spoiled_food（[perishable.lua#L292-L319](../../../raw/current/scripts/components/perishable.lua#L293-L319)）。

### 获取途径

| 途径 | 内容与证据 |
|---|---|
| 拔起／摧毁植物 | bullkelp_plant 被摧毁时生成 root 茎（植物移除）；叶是否额外掉落取决于该路径自己的采摘条件——炮弹／迫击炮／投石机等分支先查 `CanBePicked()`，通过才按 `pickable.product` 生成海带叶：船炮炮弹（[cannonballs.lua#L107-L116](../../../raw/current/scripts/prefabs/cannonballs.lua#L92-L118)）、迫击炮（[mortarball.lua#L125-L135](../../../raw/current/scripts/prefabs/mortarball.lua#L110-L136)）、三叉戟 AoE（[三叉戟分支](../../../raw/current/scripts/prefabs/trident.lua#L98-L123)）、温妮投石机弹 AoE（[winona_catapult_projectile.lua#L134-L155](../../../raw/current/scripts/prefabs/winona_catapult_projectile.lua#L134-L155)）、海洋结冰覆盖地块（[oceanicemanager.lua#L196-L214](../../../raw/current/scripts/components/oceanicemanager.lua#L196-L214)，入口 `CreateIceAtTile` [L151](../../../raw/current/scripts/components/oceanicemanager.lua#L151)） |
| 冲上岸 | bullkelp_plant 被推上陆地／木船后 1–2 秒 `CheckBeached`：先试 Pick（通过门禁才掉 1 kelp）、再移除植物、原地生成 root（[bullkelp_plant.lua#L42-L61](../../../raw/current/scripts/prefabs/bullkelp_plant.lua#L42-L61)） |
| 拾取冲岸海带 | 拾取 `bullkelp_beachedroot` 实体替换为 kelp＋bullkelp_root 各 1（继承湿度，[bullkelp_beached.lua#L15-L33](../../../raw/current/scripts/prefabs/bullkelp_beached.lua#L15-L33)） |
| 战利品池 | 海面漂流物 flotsam 加权表：root 权重 1 对 kelp 35（[oceanfishableflotsam.lua#L18-L21](../../../raw/current/scripts/prefabs/oceanfishableflotsam.lua#L18-L21)）；海獭窝掉落池权重 1（[otterden.lua#L16-L19](../../../raw/current/scripts/prefabs/otterden.lua#L16-L19)） |
| 无配方 | recipes／recipes_filter 无 bullkelp 条目；scrapbook 图鉴收录（[scrapbookdata.lua#L240](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L240)） |

正常采摘 bullkelp_plant 只给 kelp，不给 root（[bullkelp_plant.lua#L104-L109](../../../raw/current/scripts/prefabs/bullkelp_plant.lua#L104-L109)）。海带茎的其他来源见上表。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：weapon | `SetDamage(BULLKELP_ROOT_DAMAGE)`＋`SetRange(BULLKELP_ROOT_RANGE)`＋`SetOnAttack(onattack)`；无 projectile 不能远程 | 主模拟；`OnAttack` 先回调后扣磨损，无 finiteuses 则跳过 | [[reference/components/weapon#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/bullkelp_root.lua#L99-L102)、[回调](../../../raw/current/scripts/prefabs/bullkelp_root.lua#L27-L51) |
| 组件：perishable | 保质期 19200、`StartPerishing`、`onperishreplacement = "spoiled_food"`；承担磨损与折断档位 | 主模拟；手持／装备照常衰减，整叠共享保鲜，耗尽整叠换同数量 spoiled_food | [[reference/components/perishable#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/bullkelp_root.lua#L111-L114)、[替换](../../../raw/current/scripts/components/perishable.lua#L293-L319) |
| 组件：equippable | `SetOnEquip`／`SetOnUnequip`：覆盖 `swap_object`（取 swap_bullkelproot build 的 `swap_whip` 符号）＋`whipline`、ARM_carry／ARM_normal 显隐 | 主模拟；槽默认 HANDS；无皮肤接线 | [[reference/components/equippable#关键接口]]；[接线](../../../raw/current/scripts/prefabs/bullkelp_root.lua#L104-L106)、[onequip](../../../raw/current/scripts/prefabs/bullkelp_root.lua#L10-L15) |
| 组件：deployable | `ondeploy` 生成 bullkelp_plant → `pickable:MakeEmpty()`（种下即空）并从堆叠取 1 移除 | 主模拟；`DEPLOYMODE.WATER` 仅水面（land／boat 偏移 0.2）、`DEPLOYSPACING.MEDIUM`；`deployedplant` 标签使 Deploy 推 `itemplanted` 世界事件 | [[reference/components/deployable#关键接口]]、[[reference/components/pickable#关键接口]]；[ondeploy](../../../raw/current/scripts/prefabs/bullkelp_root.lua#L53-L64)、[WATER 判定](../../../raw/current/scripts/components/deployable.lua#L118-L123)、[itemplanted](../../../raw/current/scripts/components/deployable.lua#L137-L142)、[MakeEmpty](../../../raw/current/scripts/components/pickable.lua#L426-L443) |
| 组件：stackable | `maxsize = 10`；`Get()` 拆件继承剩余保鲜时间与湿度 | 主模拟 | [[reference/components/stackable#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/bullkelp_root.lua#L93-L94)、[Get](../../../raw/current/scripts/components/stackable.lua#L109-L153) |
| 组件：inspectable／inventoryitem | 检视与拾取，默认配置无定制 | 主模拟；HUD 图标按 prefab 名回退解析 `bullkelp_root.tex` | [[reference/components/inspectable#关键接口]]、[[reference/components/inventoryitem#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/bullkelp_root.lua#L96-L97)、[GetImage 回退](../../../raw/current/scripts/components/inventoryitem_replica.lua#L151-L157) |
| 组件：fuel | `fuelvalue = SMALL_FUEL`（15 秒燃料） | 主模拟 | [[reference/components/fuel#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/bullkelp_root.lua#L108-L109) |
| 组件：burnable／propagator | `MakeMediumBurnable(inst, LARGE_BURNTIME)`（烧 30 秒）＋`MakeSmallPropagator`（引燃 5–10、传播半径 3–5、会烧伤） | 主模拟 | [[reference/components/burnable#关键接口]]、[[reference/components/propagator#关键接口]]；[burnable 工厂](../../../raw/current/scripts/standardcomponents.lua#L176-L186)、[propagator 工厂](../../../raw/current/scripts/standardcomponents.lua#L200-L213) |
| 组件：floater | 公共端 `MakeInventoryFloatable()` 默认参数（small，不换 bank） | 公共端 | [[reference/components/floater#关键接口]]；[调用](../../../raw/current/scripts/prefabs/bullkelp_root.lua#L79)、[helper](../../../raw/current/scripts/standardcomponents.lua#L1772-L1790) |
| 组件：hauntable | `MakeHauntableLaunchAndIgnite`：作祟必弹飞（`LAUNCH_SPEED_SMALL` 3、冷却 `HAUNT_COOLDOWN_SMALL` 3） | 主模拟；工厂名的 Ignite 分支在当前构建被注释停用，实际不点火 | [[reference/components/hauntable#按效果选工厂]]；[调用](../../../raw/current/scripts/prefabs/bullkelp_root.lua#L124)、[工厂](../../../raw/current/scripts/standardcomponents.lua#L1191-L1223) |
| SG：玩家 SGwilson `attack`／`dojostleaction` | 按装备 `whip` 标签选鞭击分支：`whip_pre`＋`whip` 动画、17 帧冷却下限、播 `whip_pre` 音；客户端预测镜像 | 本体无 SG／Brain；标签判定在装备槽 | [attack 分支](../../../raw/current/scripts/stategraphs/SGwilson.lua#L10857-L10863)、[客户端](../../../raw/current/scripts/stategraphs/SGwilson_client.lua#L5085-L5091)、[dojostleaction](../../../raw/current/scripts/stategraphs/SGwilson.lua#L8622-L8627) |
| API／数据：`whip` 标签 | 挥击动画分支判定键，与 [[reference/prefabs/whip|whip]] 共用 | 装备在 HANDS 槽时读取 | [挂标签](../../../raw/current/scripts/prefabs/bullkelp_root.lua#L81)、[同族数值](../../../raw/current/scripts/tuning.lua#L370-L431) |
| API／数据：`show_spoilage` 标签 | itemtile 新鲜度角标：还须带 `fresh`／`stale`／`spoiled` 之一（由 perishable 维护）才显示 | 客户端 HUD | [itemtile.lua#L730-L746](../../../raw/current/scripts/widgets/itemtile.lua#L732-L749)、[挂标签](../../../raw/current/scripts/prefabs/bullkelp_root.lua#L82) |
| API／数据：`TryLuckRoll`＋`luckuser` | 折断掷点 | 命中结算时读攻击者幸运总值 | [[reference/components/luckuser#幸运值如何进入概率]]；[TryLuckRoll](../../../raw/current/scripts/componentutil.lua#L2237-L2251) |
| 关联 Prefab：bullkelp_plant（Wiki 待建） | 部署产物与“拔起／冲岸”来源；`bullkelp_root_placer` 预览其放置 | 部署在水面；转换条件见[获取途径](#获取途径) | [CheckBeached](../../../raw/current/scripts/prefabs/bullkelp_plant.lua#L42-L61)、[pickable](../../../raw/current/scripts/prefabs/bullkelp_plant.lua#L104-L109) |
| 关联 Prefab：spoiled_food（Wiki 待建） | 腐坏整叠替换产物 | perishable 耗尽时 | [替换块](../../../raw/current/scripts/components/perishable.lua#L293-L319) |
| 关联 Prefab：impact | 折断时生成于目标处、按挥击方向旋转；未折断不生成 | 主模拟生成；FX 自删见其页 | [[reference/prefabs/impact]]；[生成块](../../../raw/current/scripts/prefabs/bullkelp_root.lua#L40-L49) |
| 数据：scrapbook 图鉴 | 条目声明 weapondamage 27.2／weaponrange 2／perishable 19200／fuelvalue 15／stacksize 10／deps spoiled_food | 客户端 UI 数据 | [条目](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L240) |
| 资源：anim／swap 动画包 | bullkelp.zip（bank＋build `bullkelp`）／swap_bullkelproot.zip（持握符号借用 whip 的 `swap_whip`＋自有 `whipline` 绳线） | 静态声明 | [声明](../../../raw/current/scripts/prefabs/bullkelp_root.lua#L3-L8) |

本页无皮肤差异：prefabskin 无 bullkelp 条目，onequip／onattack 无皮肤接线。

## 典型接入

### 调整折断概率与新鲜度磨损

- 前提与选择：服务端 `modmain.lua`。折断三档与每击折损在**每次命中时实时读 `TUNING`**，直改立即全局生效；伤害在 `fn()` 构造时读表，只对注册后构造的实例（新生成或世界重载）生效，每实例附加调整用 `AddPrefabPostInit`。`TUNING` 在 modmain 环境白名单直挂（[mods.lua#L258-L317](../../../raw/current/scripts/mods.lua#L258-L317)），`TheWorld`／`SpawnPrefab`／`TryLuckRoll` 等非白名单全局须 `GLOBAL.` 前缀。
- 生效与撤销：TUNING 改动停用模组后恢复；伤害改法对运行中的已有实例要等世界重载重新构造；已有实例的当前新鲜度按存档恢复。
- 原版依据：读取点见 [onattack](../../../raw/current/scripts/prefabs/bullkelp_root.lua#L27-L51)。

```lua
-- modmain.lua：折断更宽容（低新鲜度档 0.3→0.15、满损档 1.0→0.5）＋每击折损 2%–4%→1%–2%
TUNING.BULLKELP_ROOT_SNAP_CHANCES.MED = 0.15
TUNING.BULLKELP_ROOT_SNAP_CHANCES.HIGH = 0.5
TUNING.BULLKELP_ROOT_USE = 0.01
TUNING.BULLKELP_ROOT_USE_VAR = 0.01

AddPrefabPostInit("bullkelp_root", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    inst.components.weapon:SetDamage(34) -- 新构造实例伤害 27.2→34
end)
```

- 调整与验证：折断为每击独立掷点，档位判断在折损**之后**用剩余新鲜度；幸运公式作用于全部档位。`LUCKFORMULA_RECIPROCAL`、`BullKelpSnapChanceAdditive` 与自毁调度都是文件 local，要改幸运方向或“折断改为掉 1 层堆叠”须整体重写 `SetOnAttack`（`GLOBAL.TryLuckRoll` 取原掷点入口，参考 [[reference/prefabs/ruins-bat#典型接入]] 的重写样式）。

## 常见陷阱与限制

- **误区：它像 whip 一样有耐久**：无 finiteuses，`attackwear` 无消费路径；“耐久”完全由 perishable 新鲜度表达（每击 2%–4%，40 游戏日自然耗尽）。[磨损段](../../../raw/current/scripts/components/weapon.lua#L113-L121)
- **边界：伤害不随新鲜度下降**：`GetDamage` 无 perishable 项，烂茎仍 27.2；新鲜度只影响折断概率与腐坏替换。[GetDamage](../../../raw/current/scripts/components/weapon.lua#L94-L103)
- **边界：折断移除的是装备的那一件，不是整叠**：装备时从堆叠拆 1（`equipstack` 默认 false，[inventory.lua#L1325-L1331](../../../raw/current/scripts/components/inventory.lua#L1325-L1331)），ondeploy 也只 `Get():Remove()` 1 个；但**自然腐坏是整叠一起**换 spoiled_food（整叠共享一份保鲜）。[ondeploy](../../../raw/current/scripts/prefabs/bullkelp_root.lua#L57)、[替换](../../../raw/current/scripts/components/perishable.lua#L293-L319)
- **边界：幸运方向与 whip 相反**：bullkelp 的幸运**降低**折断概率（`chance×1/(1+luck)`），倒霉提高（`chance×(1+|luck|)`）；whip 的幸运提高超响鞭。复刻掷点时别把两个公式套串。[公式](../../../raw/current/scripts/prefabs/bullkelp_root.lua#L22-L26)
- **边界：折断音效无皮肤机制**：在目标处硬编码播 `dontstarve/common/whip_small`，且本 prefab 未注册皮肤，whip 的 `skin_sound` 注入不适用；onequip 也不处理皮肤 build（对照 [[reference/prefabs/whip#皮肤]]）。[折断块](../../../raw/current/scripts/prefabs/bullkelp_root.lua#L44-L49)
- **边界：只能种回水面**：`DEPLOYMODE.WATER` 走 `CanDeployAtPointInWater`，陆地不可；种下的是**空**植物（不患移植病，`OnTransplant` 调用被注释），`BULLKELP_REGROW_TIME`（3 游戏日）后再生 kelp。[WATER](../../../raw/current/scripts/components/deployable.lua#L118-L123)、[注释](../../../raw/current/scripts/prefabs/bullkelp_root.lua#L58)、[MakeEmpty](../../../raw/current/scripts/components/pickable.lua#L426-L443)、[再生时间](../../../raw/current/scripts/tuning.lua#L3798)
- **边界：`MakeHauntableLaunchAndIgnite` 名不副实**：工厂的点火分支在当前构建被注释，作祟只弹飞（必中、冷却 3 秒）。[工厂](../../../raw/current/scripts/standardcomponents.lua#L1207-L1216)
- **边界：图标声明与解析名不一致**：assets 声明 `Asset("INV_IMAGE", "kullkelp_root")`（源文件原样拼写），HUD 实际按 prefab 名回退解析 `bullkelp_root.tex`；复刻图标按 prefab 名配 inventoryimages 图集即可，别照抄这一行。[声明](../../../raw/current/scripts/prefabs/bullkelp_root.lua#L7)、[回退](../../../raw/current/scripts/components/inventoryitem_replica.lua#L151-L157)

## 验证记录

- 源码审阅：2026-11-07（第三轮修订：复核武器加值口径）。早前 2026-11-06，Steam buildid 26335394（raw/current/BUILD.txt）。核验 prefabs/bullkelp_root.lua 全文（单注册、onattack 折损→取档→掷点→impact／自毁、ondeploy、onequip 接线、floater／burnable／propagator／hauntable 工厂、无 quagmire 分支、placer）；weapon OnAttack／GetDamage／SetRange；equippable 默认 HANDS；perishable ReducePercent／SetPercent 钳制、perished 整叠替换、fresh／stale／spoiled 标签、held 修正；pickable MakeEmpty；stackable Get；deployable WATER 判定与 itemplanted；inventory 装备拆 1；inventoryitem_replica GetImage 回退；`TryLuckRoll`／`GetEntityLuckChance`／`GetEntityLuck`；tuning 九键＋`seg_time`／`total_day_time`／`perish_warp`／`wilson_attack`；SGwilson `attack`／`dojostleaction` 与 SGwilson_client `attack` 的 whip 分支；生成点：全 scripts grep `SpawnPrefab("bullkelp_root")` 逐处核验（cannonballs／mortarball／trident／winona_catapult_projectile／oceanicemanager CreateIceAtTile／bullkelp_plant CheckBeached／bullkelp_beached 替换拾取）加两个掉落池（oceanfishableflotsam／otterden）；recipes／recipes_filter／prefabskin grep 零命中（无配方、无皮肤）；.po 名称／检视语；scrapbookdata 条目；mods.lua CreateEnvironment 白名单。whip 同族对照与 impact 自删复用 [[reference/prefabs/whip]]、[[reference/prefabs/impact]]（同 build 已核验页）。
- 游戏运行：未运行。未覆盖：27.2 伤害、2 射程与 17 帧连击节奏为源码推导非实测；折断三档实机概率、“打到 0 后的必断窗口”实际时长、种回深海的实机表现未验证。
- 结构检查：validate_page.py 与 strict 95 引用 0 错 0 警。

返回 [[reference/prefabs/weapons-combat-index|武器与战斗]]、[[reference/prefabs/index|Prefab 索引]]。
