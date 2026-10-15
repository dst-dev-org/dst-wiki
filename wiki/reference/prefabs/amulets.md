---
title: "宝石护符族（amulet 等六注册）"
created: 2026-11-07
updated: 2026-11-07
type: prefab
summary: "原版六件 BODY 槽宝石护符的注册与组装：重生换血、受击冰冻反制、强制疯狂、半价制作、自动拾取、发光加速，以及作祟复活链与噩梦燃料补给的接入入口。"
aliases: ["amulet", "blueamulet", "purpleamulet", "orangeamulet", "greenamulet", "yellowamulet", "重生护符", "生命护符", "寒冰护符", "建造护符", "懒人护符", "梦魇护符", "魔光护符", "宝石护符", "Life Giving Amulet", "Chilled Amulet", "Construction Amulet", "The Lazy Forager", "Nightmare Amulet", "Magiluminescence"]
tags: [prefab, item, equipment]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/amulet.lua", "prefabs/nightmarefuel.lua", "prefabs/player_common_extensions.lua", "prefabs/walter.lua", "components/klaussackloot.lua", "prefabs/altar_prototyper.lua", "prefabskins.lua", "prefabskin.lua", "components/builder.lua", "components/fueled.lua", "components/fuel.lua", "components/finiteuses.lua", "components/equippable.lua", "components/heater.lua", "components/temperature.lua", "components/repairable.lua", "components/repairer.lua", "components/sanity.lua", "components/hauntable.lua", "components/locomotor.lua", "standardcomponents.lua", "componentutil.lua", "simutil.lua", "mathutil.lua", "constants.lua", "tuning.lua", "recipes.lua", "actions.lua", "componentactions.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
size_class: complex
---

# 宝石护符族（amulet 等六注册）

## 快速判断

- 职责与适用：`prefabs/amulet.lua` 单文件注册六件可装备护符与一个灯光 FX（[注册](../../../raw/current/scripts/prefabs/amulet.lua#L624-L630)）。“宝石护符族”为效果描述性总称，不是官方译名。适合查“可装备饰品＋专属 equip 回调＋finiteuses/fueled 耐久”的原版组装；六变体共用一个 `commonfn` 工厂、机制逐件不同，按复杂档整页对照。
- 优先使用：耐久选型（fueled 还是 finiteuses）查 [[reference/components/fueled#燃料与使用耐久的选择]]；装备槽与 dapperness 查 [[reference/components/equippable#关键接口]]；单件机制直查下表对应注册行。
- 重要副作用：装备是改 `owner` 的状态而非自身（建造护符改 `builder.ingredientmod`、梦魇护符改 `sanity`）；blue／purple／yellow 装备期间 fueled 每秒耗 1 点（[消耗率](../../../raw/current/scripts/components/fueled.lua#L89)、[更新](../../../raw/current/scripts/components/fueled.lua#L325-L332)）；orange 每 0.33 秒自动拾取并消耗次数。
- 修改边界：全部机制配置在 `TheWorld.ismastersim` 分支之后（主模拟）；客户端只挂 `playfuelsound` 补音监听（[公共段](../../../raw/current/scripts/prefabs/amulet.lua#L387-L394)）。本页不含 `nightmareamulet`（该注册名在本快照 scripts 全目录零出现，属单机版内容）。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `amulet`（重生护符） | `commonfn("redamulet", "resurrector", true)` → 主模拟挂 finiteuses(20)＋hauntable(INSTANT_REZ)＋装备回调换血循环 | 唯一入水下沉（`SetSinks(true)`）、无漂浮；唯一 `SetHauntValue` 直挂作祟复活；[工厂](../../../raw/current/scripts/prefabs/amulet.lua#L424-L450) |
| `blueamulet`（寒冰护符） | `commonfn("blueamulet", "HASHEATER")` → heater(制冷 -20)＋fueled(MAGIC, 360s)＋受击冰冻回调＋作祟范围冰冻 | 作祟走 `MakeHauntableLaunch`＋`AddHauntableCustomReaction`；[工厂](../../../raw/current/scripts/prefabs/amulet.lua#L468-L494)、[自定义反应](../../../raw/current/scripts/prefabs/amulet.lua#L452-L466) |
| `purpleamulet`（梦魇护符） | `commonfn("purpleamulet")` → fueled(MAGIC, 240s)＋dapperness 覆写为负＋`SetInducedInsanity` | 覆写公共 dapperness 为 `-DAPPERNESS_MED`；[工厂](../../../raw/current/scripts/prefabs/amulet.lua#L496-L518) |
| `orangeamulet`（懒人护符） | `commonfn("orangeamulet", "repairshortaction", nil, true)` → finiteuses(225)＋repairable(噩梦燃料)＋周期自动拾取 | 可修复、可补音（`can_refuel=true`）；[工厂](../../../raw/current/scripts/prefabs/amulet.lua#L541-L568)、[拾取](../../../raw/current/scripts/prefabs/amulet.lua#L188-L220) |
| `greenamulet`（建造护符） | `commonfn("greenamulet")` → finiteuses(5)＋装备时设 `builder.ingredientmod=0.5`＋`consumeingredients` 扣次 | 无持续消耗组件，按次扣；[工厂](../../../raw/current/scripts/prefabs/amulet.lua#L520-L539) |
| `yellowamulet`（魔光护符） | `commonfn("yellowamulet", nil, nil, true)` → fueled(NIGHTMARE, 480s, 可补)＋`walkspeedmult=1.2`＋子灯光＋bloom | 唯一接受燃料补给与移速加成；[工厂](../../../raw/current/scripts/prefabs/amulet.lua#L570-L597) |
| `yellowamuletlight`（灯笼 FX，注册实体不在本页展开） | 由 `yellowlightfn` 生成：Transform＋Light＋Network＋`FX` 标签，半径 2／falloff .7／强度 .65／暖黄色，无 equippable、无 AnimState | 关系入口：yellow 装备时 `SpawnPrefab` 后 `_light.entity:SetParent(owner.entity)`，卸下／入模型／掉落由 `turnoff_yellow` 移除；`persists=false` 不存档。工厂与端边界归 [[reference/prefabs/armor_lunarplant_glow_fx|装备与帽饰跟随光效]]（该页已登记该注册与别名），本页只保留生成与清理接线 |

公共段（[commonfn](../../../raw/current/scripts/prefabs/amulet.lua#L363-L422)）：bank/build `amulets`、`MakeInventoryPhysics`、预置 `shadowlevel` 标签（pristine 优化，真组件在主模拟）、`foleysound` 为 jewlery、未下沉者 `MakeInventoryFloatable("med", nil, 0.6)`；主模拟侧 inspectable、equippable、inventoryitem、shadowlevel(1)。装备槽全部为 `EQUIPSLOTS.BODY`（即 `"body"`，[定义](../../../raw/current/scripts/constants.lua#L654-L660)，[赋值](../../../raw/current/scripts/prefabs/amulet.lua#L409)），与护甲／背包同槽互斥；dapperness 统一 `+DAPPERNESS_SMALL` 且 `is_magic_dapperness=true`（[赋值](../../../raw/current/scripts/prefabs/amulet.lua#L410-L411)）。

名称键 `STRINGS.NAMES.AMULET`／`BLUEAMULET`／`PURPLEAMULET`／`ORANGEAMULET`／`GREENAMULET`／`YELLOWAMULET`，当前简中：重生护符（Life Giving Amulet）、寒冰护符（Chilled Amulet）、梦魇护符（Nightmare Amulet）、懒人护符（The Lazy Forager）、建造护符（Construction Amulet）、魔光护符（Magiluminescence）。[.po 条目](../../../raw/current/scripts/languages/chinese_s.po#L338585-L338587)起六处分散，行号见验证记录。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| `组件：equippable` | 公共 BODY 槽＋dapperness；各件 `SetOnEquip/OnUnequip/OnEquipToModel` 三回调；yellow 加 `walkspeedmult=1.2` | 主模拟配置；模型展示时回调取消任务／停耗 | [[reference/components/equippable#关键接口]]；[三回调](../../../raw/current/scripts/prefabs/amulet.lua#L437-L439)、[移速](../../../raw/current/scripts/prefabs/amulet.lua#L580)；消费端 locomotor 装备倍率 [读取](../../../raw/current/scripts/components/locomotor.lua#L193) |
| `组件：finiteuses` | red 20 次（每次换血扣 1）、green 5 次（每次打折制作扣 1）、orange 225 次（每次拾取扣 1）；`SetOnFinished(inst.Remove)` 耗尽即移除 | 主模拟；orange 另可修复回充 | [[reference/components/finiteuses#原版不同消费模式]]；[red](../../../raw/current/scripts/prefabs/amulet.lua#L441-L444)、[green](../../../raw/current/scripts/prefabs/amulet.lua#L531-L534)、[orange](../../../raw/current/scripts/prefabs/amulet.lua#L555-L558)；修复回充 [Repair](../../../raw/current/scripts/components/finiteuses.lua#L128-L132) |
| `组件：fueled` | blue(MAGIC)/purple(MAGIC)/yellow(NIGHTMARE)；装备回调内 `StartConsuming`／`StopConsuming`；耗尽 `SetDepletedFn(inst.Remove)` | MAGIC 注释“无对应燃料”且未设 `accepting`，不可补；yellow `accepting=true` 可喂噩梦燃料 | [[reference/components/fueled#关键接口]]；[blue](../../../raw/current/scripts/prefabs/amulet.lua#L484-L488)、[purple](../../../raw/current/scripts/prefabs/amulet.lua#L503-L507)、[yellow](../../../raw/current/scripts/prefabs/amulet.lua#L583-L589)；接受条件 [CanAcceptFuelItem](../../../raw/current/scripts/components/fueled.lua#L169-L175) |
| `组件：heater` | blue：`SetThermics(false, true)` 制冷、`equippedheat = BLUEGEM_COOLER = -20` | 仅装备槽位被 temperature 消费（`GetEquippedHeat`）；`HASHEATER` 标签公共段预置 | [[reference/components/heater#携带范围与地面热源的分工]]；[配置](../../../raw/current/scripts/prefabs/amulet.lua#L480-L482)、[装备槽消费](../../../raw/current/scripts/components/temperature.lua#L370-L381)、[常量](../../../raw/current/scripts/tuning.lua#L203) |
| `组件：builder`（改 owner） | green 装备设 `owner.builder.ingredientmod=0.5`，监听 `consumeingredients` 扣次；卸下／入模型复位为 1 | 制作时 `GetIngredients` 按单件 `RoundBiasedUp(amount*0.5)` 取量；折扣 equip 是 itemmimic 时 TurnEvil | [[reference/components/builder#制作事件与产物初始化]]；[接线](../../../raw/current/scripts/prefabs/amulet.lua#L154-L164)、[取量](../../../raw/current/scripts/components/builder.lua#L500-L517)、[mimic 检查](../../../raw/current/scripts/components/builder.lua#L534-L543) |
| `组件：repairable` | orange：`repairmaterial = MATERIALS.NIGHTMARE`、`onrepaired` 补音 | 得标签 `repairable_nightmare`；噩梦燃料（repairer `finiteusesrepairvalue=50`）可修 | [[reference/components/repairable#恢复分支与顺序]]；[配置](../../../raw/current/scripts/prefabs/amulet.lua#L560-L563)、[标签](../../../raw/current/scripts/components/repairable.lua#L1-L8)、[燃料侧](../../../raw/current/scripts/prefabs/nightmarefuel.lua#L82-L86)、[修量](../../../raw/current/scripts/tuning.lua#L6855) |
| `组件：sanity`（改 owner） | purple 装备 `SetInducedInsanity(inst, true)`、卸下复位 | 强制 `IsInsane()` 为真（不视实际理智值），`GetPercent` 显示 0；事件 `inducedinsanity` | [[reference/components/sanity#关键接口]]；[接线](../../../raw/current/scripts/prefabs/amulet.lua#L116-L118)、[判定](../../../raw/current/scripts/components/sanity.lua#L146-L147)、[注册源表](../../../raw/current/scripts/components/sanity.lua#L331-L351) |
| `组件：shadowlevel` | 全族默认等级 `AMULET_SHADOW_LEVEL = 1` | 主模拟挂组件；pristine 标签供麦斯威尔影子强度等消费端预筛 | [[reference/components/shadowlevel#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/amulet.lua#L418-L419)、[常量](../../../raw/current/scripts/tuning.lua#L2857) |
| `组件：bloomer`（改 owner） | yellow 装备 `PushBloom("shaders/anim.ksh")`，无 bloomer 组件则直接 `SetBloomEffectHandle` | 卸下／入模型 Pop 回退 bloom；掉落回调只 `turnoff_yellow` 关灯，不动 bloom | [[reference/components/bloomer#关键接口]]；[接线](../../../raw/current/scripts/prefabs/amulet.lua#L271-L275) |
| `组件：hauntable` | red 直挂 `SetHauntValue(HAUNT_INSTANT_REZ)`（`no_wipe_value=true`，默认 onhaunt 必成功）；blue 发射＋范围冰冻；其余四件 `MakeHauntableLaunch` | 作祟主体是地上物品；HAUNT 动作仅限玩家幽灵且目标非 `IsHeld` | [[reference/components/hauntable#按效果选工厂]]；[red](../../../raw/current/scripts/prefabs/amulet.lua#L446-L447)、[blue 组合](../../../raw/current/scripts/prefabs/amulet.lua#L490-L491)、[helper](../../../raw/current/scripts/standardcomponents.lua#L1053-L1069)；动作门槛 [HAUNT](../../../raw/current/scripts/actions.lua#L3153-L3164) |
| `API／事件：consumeingredients` | green 的扣次监听；事件携带 `{discounted}`，仅当非 `false` 时扣 | `RemoveIngredients` 末尾 push；`discounted=false` 表示本次制作未实际打折 | [[reference/components/builder#制作事件与产物初始化]]；[push](../../../raw/current/scripts/components/builder.lua#L590)、[扣次判断](../../../raw/current/scripts/prefabs/amulet.lua#L157-L164) |
| `API／数据：ShouldProcOnAttackedOrBlocked` | blue 受击回调的门禁：排除被重定向与“攻击者=owner”的场合 | `attacked` 事件数据；格挡类共用 helper | [实现](../../../raw/current/scripts/componentutil.lua#L114-L122)、[调用](../../../raw/current/scripts/prefabs/amulet.lua#L66-L71) |
| `API／数据：FindPickupableItem` | orange 每 0.33 秒以半径 4 最近优先搜索可拾取物 | 过滤装备/武器/工具/护甲/缝纫/燃烧中/容器/礼包机/陷阱饵/当前交互目标 | [实现](../../../raw/current/scripts/simutil.lua#L419-L500)、[调用](../../../raw/current/scripts/prefabs/amulet.lua#L189)、[参数](../../../raw/current/scripts/tuning.lua#L214-L216) |
| `关联 Prefab：nightmarefuel` | yellow 的补给燃料（`fueltype=NIGHTMARE` 得标签 `NIGHTMARE_fuel`）；orange 的修复材料 | FUEL 动作要求双方标签成对（`_fueled`/`_fuel`） | [燃料组件](../../../raw/current/scripts/prefabs/nightmarefuel.lua#L82)、[fuel 标签](../../../raw/current/scripts/components/fuel.lua#L1-L8)、[动作标签](../../../raw/current/scripts/componentactions.lua#L1311-L1317) |
| `关联 Prefab：sand_puff` | orange 拾取时的烟团特效；写入 Prefab deps | 每次拾取生成一个 | [[reference/prefabs/sand_puff]]；[生成](../../../raw/current/scripts/prefabs/amulet.lua#L208)、[声明](../../../raw/current/scripts/prefabs/amulet.lua#L627) |
| `关联 Prefab：resurrectionstone／statue` | 与 red 同挂 `resurrector` 标签的复活来源 | 复活统一走 `respawnfromghost` 事件；按 `source.prefab` 分支 | [复活流程](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L358-L370) |
| `关联 Prefab：ancient_altar（altar_prototyper）` | 锤击远古祭坛／对破损祭坛施工触发随机奖励表，六护符权重各 1 | `DoRandomThing` 权重表（生物 100+、宝石 5、护符 1）；伴随落雷 | [权重表](../../../raw/current/scripts/prefabs/altar_prototyper.lua#L62-L91)、[锤击](../../../raw/current/scripts/prefabs/altar_prototyper.lua#L329-L337) |
| `资源：anim/amulets.zip＋torso_amulets.zip` | 地面动画与 `swap_body` 穿戴覆盖（build `torso_amulets`） | 皮肤走 `OverrideItemSkinSymbol`；资源包内容未逐帧核验 | [assets](../../../raw/current/scripts/prefabs/amulet.lua#L1-L5)、[覆盖示例](../../../raw/current/scripts/prefabs/amulet.lua#L26) |

未核验关系：`resurrector` 标签在本快照 Lua 侧仅见 AddTag（护符／门／复活石等），未检索到消费端；`yellowamuletlight` 的网络复制协议未展开。

### 皮肤

皮肤清单（[prefabskins.lua](../../../raw/current/scripts/prefabskins.lua#L50-L56) 等六组）：amulet 5、blueamulet 2、greenamulet 2、orangeamulet 3、purpleamulet 2、yellowamulet 5（`amulet_yellow_an` 等）；`yellowamuletlight` 无皮肤。装备回调按 `GetSkinBuild` 分支换 `OverrideItemSkinSymbol` 并推 `equipskinneditem` 事件（[示例](../../../raw/current/scripts/prefabs/amulet.lua#L20-L30)）；yellow 皮肤附加装备音 `skin_equip_sound`（[注入](../../../raw/current/scripts/prefabskin.lua#L31)、[消费](../../../raw/current/scripts/prefabs/amulet.lua#L277-L279)）。皮肤贴图实际外观未核验。

## 典型接入

### 借用重生护符的作祟复活链

- 前提与选择：模组物品（主模拟构造）想成为“幽灵作祟即复活”的来源；复用 hauntable 的 INSTANT_REZ 约定，不必自写复活逻辑。
- 生效与撤销：`SetHauntValue` 同时置 `no_wipe_value=true` → 作祟值不会在成功后清空，可反复作祟；每次成功作祟后进作祟冷却（默认 `HAUNT_COOLDOWN_MEDIUM=5` 秒）。但**原版 red 护符复活后会被消耗**：`OnRespawnFromGhost` 先对 `source.prefab == "amulet"` 走 `inventory:Equip(source)`，随即进 `amulet_rebirth` 态把它从 BODY 槽取下并 `item:Remove()`——所以"复活不限次"只对不走该分支的自定义物品成立。
- 原版依据：red 护符即此组装（[工厂](../../../raw/current/scripts/prefabs/amulet.lua#L424-L450)）；hauntable 判定 [DoHaunt](../../../raw/current/scripts/components/hauntable.lua#L82-L114)；复活由玩家侧 `OnRespawnFromGhost` 统一处理（[入口](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L569)、[分支](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L358-L370)）。

```lua
-- 自定义物品主模拟分支内（TheWorld.ismastersim 之后）
inst:AddComponent("hauntable")
inst.components.hauntable:SetHauntValue(TUNING.HAUNT_INSTANT_REZ)
-- source.prefab == "amulet" 的"临时装备＋进 amulet_rebirth 销毁"分支不适用于其他 prefab：
-- 自定义物品只触发基础复活，既不会被消耗、也不会自动穿回
```

- 调整与验证：复用 `HAUNT_INSTANT_REZ` **不会**自动得到原版的物品销毁分支，所以要限次复活必须显式在复活成功后扣次数或销毁本件（或自写 `onhaunt` 返回控制）；删 `SetHauntValue` 只是关掉复活。目标必须在地面上（被持有/容器内时 HAUNT 动作不可用）。

### 借用建造护符的半价制作接线

- 前提与选择：模组耐久饰品想提供“制作材料减半”；机制全部在 builder 组件的消费端，挂装备回调即可，无需改配方。
- 生效与撤销：装备设 `ingredientmod`，`onunequip` 与 `onunequiptomodel` 两条收尾都要复位为 1 并 `RemoveEventCallback`；mod 取值受 `INGREDIENT_MOD`（0/0.25/0.5/0.75/1.0）断言限制（[断言](../../../raw/current/scripts/components/builder.lua#L3-L5)）。前提两条：佩戴者必须有 `builder` 组件（原版先判 `owner.components.builder ~= nil` 才写），且本件必须已初始化 `finiteuses`（否则扣次调用空引用）。
- 原版依据：[green 接线](../../../raw/current/scripts/prefabs/amulet.lua#L146-L185)；单件取量 `max(1, RoundBiasedUp(amount*mod))`（[取量](../../../raw/current/scripts/components/builder.lua#L507)、[取整](../../../raw/current/scripts/mathutil.lua#L24-L27)）。

```lua
-- 装备回调（onequip）内：佩戴者必须有 builder，绝大多数生物没有
if owner.components.builder ~= nil then
    owner.components.builder.ingredientmod = TUNING.GREENAMULET_INGREDIENTMOD -- 0.5
end
inst.onitembuild = function(owner, data)
    if not (data ~= nil and data.discounted == false) then
        inst.components.finiteuses:Use(1) -- 仅在显式 discounted == false（确定没打折）时跳过扣次
    end
end
inst:ListenForEvent("consumeingredients", inst.onitembuild, owner)
-- 对应 onunequip / onunequiptomodel：ingredientmod 复位为 1 并 RemoveEventCallback
-- 本件还须自己 AddComponent("finiteuses")，并 SetMaxUses + SetUses 初始化，否则上面 Use(1) 是空引用
```

- 调整与验证：单件材料 amount=1 时减半不减量，`discounted` 为 `false`，原版语义下不扣次；免费制作（`freebuildmode`）不进入 `RemoveIngredients`，同样不扣。

## 常见陷阱与限制

- **误区：护符挂“颈部槽”**：DST 的 `EQUIPSLOTS` 只有 hands/head/body/beard，六件护符全部是 BODY 槽（[定义](../../../raw/current/scripts/constants.lua#L654-L660)），与护甲、背包互斥——戴护符时无法同时穿身体护甲。
- **误区：重生护符佩戴时死亡会自动复活**：原版已注释掉 `keepondeath`（[注释](../../../raw/current/scripts/prefabs/amulet.lua#L433-L435)），护符落地；须幽灵对**地面上的**护符作祟（HAUNT 动作要求目标非 `IsHeld`）触发复活。**复活会消耗掉这只护符**：复活流程把它临时装进 BODY 槽再进 `amulet_rebirth` 态取下销毁（[分支](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L358-L370)、[状态内销毁](../../../raw/current/scripts/stategraphs/SGwilson.lua#L15218-L15239)）——不是"复活后自动穿回"。
- **边界：MAGIC 燃料不可补**：`FUELTYPE.MAGIC` 的注释即“没有对应燃料物品”（[定义](../../../raw/current/scripts/constants.lua#L1995-L2007)）；blue/purple 未设 `accepting`，`CanAcceptFuelItem` 直接拒绝。只有 yellow 接受 NIGHTMARE 类燃料。
- **边界：`discounted` 只在"确有一项算出来比原配方少"时为真。** `GetIngredients` 逐项 `amt = max(1, RoundBiasedUp(amount × ingredientmod))`，只有 `amt < 原 amount` 才 `discounted = true`（[取量比较](../../../raw/current/scripts/components/builder.lua#L500-L516)）——所以 amount=1 的配方减半后仍是 1、不标记折扣。另一侧是**向后兼容**：`consumeingredients` 只在 `data.discounted == false` 时跳过扣次，字段缺失或为 nil 照样扣（[扣次判断](../../../raw/current/scripts/prefabs/amulet.lua#L157-L164)）；老代码/别的调用方不传这个字段时扣次不变。（[`amount>1` 自行判定"是否打折"是不够的：要按计算后的取量比。）
- **边界：orange 拾取范围与黑名单**：半径 4 逐件拾取（叠堆只取一件）；装备/武器/工具/护甲类、燃烧中、容器、礼包机、陷阱饵、当前交互目标都不拾（[过滤](../../../raw/current/scripts/simutil.lua#L430-L468)）；陷阱则改为收猎（`trap:Harvest`）。
- **边界：blue 反制有代价与门禁**：仅 owner 的 `attacked` 事件（不含 `blocked`）、攻击者需有 `freezable`；每次命中给攻击者 0.67 寒冷并自扣 fueled 3%（[回调](../../../raw/current/scripts/prefabs/amulet.lua#L65-L73)）。
- **边界：yellow 的清理分两路，别混成一条。** `onunequip_yellow` / `onequiptomodel_yellow` 做三件事：Pop bloom（无 bloomer 则 `ClearBloomEffectHandle`）＋`fueled:StopConsuming`＋`turnoff_yellow`（移除灯）；而 `inventoryitem:SetOnDroppedFn(turnoff_yellow)` **只关灯**，不碰 bloom、也不停 fueled（没装备本就不消耗）。灯 `persists=false` 不存档，重进世界后要重新装备才生成。
- **边界：耐久数值单位**：fueled 数值是秒（`rate=1`/秒，[更新](../../../raw/current/scripts/components/fueled.lua#L325-L332)）；dapperness 是每秒理智，`DAPPERNESS_SMALL = 100/(day_time*10)` 约每天 +16（[注释](../../../raw/current/scripts/tuning.lua#L2214-L2215)）；Walter 只接受 `is_magic_dapperness` 的装备理智（[消费端](../../../raw/current/scripts/prefabs/walter.lua#L754-L760)），护符属于此类。

## 验证记录

- 源码审阅：2026-11-07，Steam buildid 26335394。完整读取 `prefabs/amulet.lua` 全 631 行（含三回调逐件、公共段、七注册）；定向核验 tuning（L19-20、198-220、1668、2214-2215、2857、3044-3063、6855）、constants（EQUIPSLOTS/FUELTYPE/MATERIALS/INGREDIENT_MOD）、recipes（L181、L674-676、L799-801）、builder（取量/事件/mimic）、fueled（rate/accepting/TakeFuelItem/耗尽）、fuel、finiteuses、repairable/repairer、heater/temperature、sanity、equippable/locomotor、hauntable/standardcomponents、actions/componentactions 动作标签、simutil/componentutil/mathutil、nightmarefuel、player_common_extensions 复活链、walter、klaussackloot、altar_prototyper、prefabskins/prefabskin。名称经 python 读 `chinese_s.po` 六个 `STRINGS.NAMES.*` 键（msgctxt 行 338585/339530/347390/346675/343430/352755）。未核验：皮肤贴图与动画资源实际内容、replica 层同步细节、其他角色 Describe 文案、除 red 外各件在实际掉落/世界生成中的完整出现面（已列 altar/klaus/骷髅布局三处直证）。
- 游戏运行：未运行；换血节奏、作祟复活手感、皮肤音效等未实测。
- 结构检查：2026-11-07 第六轮修订后 `validate_page.py`（含 `--check-indexes`）通过（129 行／24547 字节，complex 档上限 480 行／65536 字节）；`validate_source_links.py --strict` 通过（89 条引用、26 个源文件，0 error 0 warning）；两段 Lua 示例经 `parser_probe --syntax-only` 解析通过。

返回 [[reference/prefabs/wearables-index|穿戴防具与饰品]]、[[reference/prefabs/index|Prefab 索引]]。
