---
title: "装备与物品持握跟随层 FX 族（staff_lunarplant_fx 等）"
created: "2026-11-04"
updated: "2026-11-04"
type: prefab
size_class: complex
summary: "六项装备／物品持握与跟随表现：亮茄魔杖持握光、亮茄剑双层 blade、暗影香炉持握本体与皮肤、弹弓六层部件、雪人帽子实体与滚雪球拖影；逐项记录工厂、端、生成点、父挂与清理。"
aliases: ["staff_lunarplant_fx", "sword_lunarplant_blade_fx", "thuriblebody", "slingshotparts_fx", "snowmanhat_fx", "snowball_rolling_fx", "持握跟随FX", "装备持握层", "雪人帽子FX", "滚雪球特效"]
tags: [prefab, fx, server-client]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/staff_lunarplant.lua", "prefabs/sword_lunarplant.lua", "prefabs/thurible.lua", "prefabs/slingshot.lua", "prefabs/snowman.lua", "components/snowmandecoratable.lua", "components/highlightchild.lua", "components/colouradder.lua", "components/equippable.lua", "prefabskin.lua", "actions.lua", "standardcomponents.lua", "prefabs/shadow_battleaxe.lua", "entityscript.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 装备与物品持握跟随层 FX 族（staff_lunarplant_fx 等）

## 快速判断

- 职责与适用：本页收六项“挂在装备或物品上的持握／跟随表现”，宿主玩法（武器伤害、耐久、燃料、弹弓部件数值、雪人堆叠与融化）不在本页。按生成时机与生命周期分四类：
  1. **随物品创建、装备时改挂持有者**：`staff_lunarplant_fx`（亮茄魔杖的持握光层）与 `sword_lunarplant_blade_fx`（亮茄剑的两层 blade）。
  2. **装备时生成、卸下删除**：`thuriblebody`（暗影香炉的持握本体／皮肤层）与 `slingshotparts_fx`（弹弓的六层部件；五个弹弓变体共用）。
  3. **雪人装饰时生成、且实际承载帽子**：`snowmanhat_fx`（雪人的帽子装备模型，内部 `inventory` 持有帽子实体）。
  4. **推雪球时生成、世界物体跟随**：`snowball_rolling_fx`（滚雪球拖影，无 Follower）。
- 优先使用：要复用“装备时生成＋父挂＋`FollowSymbol("swap_object")`＋高亮登记”看 `slingshotparts_fx`；要“随物品常驻、浮空与持握双向改挂”看魔杖／剑的 `SetFxOwner`；要给帽子做显示模型看 `snowmanhat_fx` 的 0 槽 `inventory`；要做贴地随动物体的短命层看 `snowball_rolling_fx` 的 `KillFx`＋`animover` 收尾。
- 重要副作用：`snowmanhat_fx` 不是纯表现，它把帽子实体装进自身 `inventory`，帽子由此离开玩家背包；`sword_lunarplant_blade_fx` 一个注册会被实例化两次并分别改播 `swap_loop1`／`swap_loop2`；`staff_lunarplant_fx`／`sword_lunarplant_blade_fx` 的 `colouradder` 没有同步组件，附加色只在主模拟可见；`snowball_rolling_fx:KillFx()` 只置标志，下一次 `animover` 才真正移除。
- 修改边界：六项都由主模拟生成联网实体（`AddNetwork`＋`SetPristine`，客户端工厂在 Pristine 后早退），没有 [[reference/prefabs/armor_lunarplant_glow_fx|护甲光效页]] 那种“客户端本地建非网络子帧”的模型；本页只覆盖工厂、端、生成点、父挂／跟随与清理，宿主物品与雪人玩法均为 Wiki 待建。`size_class: complex` 因本页覆盖 6 个工厂、4 类生成时机与两种跟随模型，需要整体对照。
- 名称：六个注册名在 `languages/chinese_s.po` 均无 `STRINGS.NAMES` 键，标题中文是已核验效果描述，不是官方译名。宿主官方名只用于定位：亮茄魔杖 [L349965-L349967](../../../raw/current/scripts/languages/chinese_s.po#L349965-L349967)、亮茄剑 [L350195-L350197](../../../raw/current/scripts/languages/chinese_s.po#L350195-L350197)、暗影香炉 [L350390-L350392](../../../raw/current/scripts/languages/chinese_s.po#L350390-L350392)、可靠的弹弓 [L349455-L349457](../../../raw/current/scripts/languages/chinese_s.po#L349455-L349457)、雪人 [L349705-L349707](../../../raw/current/scripts/languages/chinese_s.po#L349705-L349707)。

## 注册与构造

六项均为主模拟生成的联网实体：工厂先加 Transform／AnimState／（必要的）Follower／Network 与 Pristine 前组件，再 `SetPristine`，随后按 `TheWorld.ismastersim` 早退；客户端只跑 Pristine 前分支与复制回调。`staff_lunarplant_fx`、`sword_lunarplant_blade_fx` 在宿主物品的 `fn` 里生成；`thuriblebody`、`slingshotparts_fx` 在 `onequip` 生成；`snowmanhat_fx` 由雪人装饰流程生成；`snowball_rolling_fx` 由推雪球周期任务生成。

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `staff_lunarplant_fx` | `fxfn`（[L235-L269](../../../raw/current/scripts/prefabs/staff_lunarplant.lua#L235-L269)）：Transform／AnimState／Follower／Network → `FX` 标签 → bank／build `staff_lunarplant`、`swap_loop` 循环 → 两处 symbol bloom 与四处 symbol light override（[L245-L254](../../../raw/current/scripts/prefabs/staff_lunarplant.lua#L245-L254)）→ `highlightchild`（[L256](../../../raw/current/scripts/prefabs/staff_lunarplant.lua#L256)）→ Pristine → 客户端早退 → 主模拟加 `colouradder`、`persists=false`（[L260-L266](../../../raw/current/scripts/prefabs/staff_lunarplant.lua#L260-L266)）。注册 [L271-L272](../../../raw/current/scripts/prefabs/staff_lunarplant.lua#L271-L272)。 | 魔杖主模拟 `fn` 在创建物品时就生成 FX 并先挂回本体（[L201-L205](../../../raw/current/scripts/prefabs/staff_lunarplant.lua#L201-L205)）；浮空跟随 `swap_spear`，装备时由 `SetFxOwner` 改挂持有者 `swap_object`（[L12-L30](../../../raw/current/scripts/prefabs/staff_lunarplant.lua#L12-L30)）。FX 与本体共享一次随机起始帧（[L201-L204](../../../raw/current/scripts/prefabs/staff_lunarplant.lua#L201-L204)）。 |
| `sword_lunarplant_blade_fx` | 同一个 `fxfn`（[L277-L307](../../../raw/current/scripts/prefabs/sword_lunarplant.lua#L277-L307)）：Transform／AnimState／Follower／Network → `FX` → bank／build `sword_lunarplant`、`swap_loop1` 循环（[L287-L292](../../../raw/current/scripts/prefabs/sword_lunarplant.lua#L287-L292)）→ `highlightchild`（[L294](../../../raw/current/scripts/prefabs/sword_lunarplant.lua#L294)）→ Pristine → 客户端早退 → 主模拟 `colouradder`、`persists=false`（[L298-L304](../../../raw/current/scripts/prefabs/sword_lunarplant.lua#L298-L304)）。注册 [L309-L310](../../../raw/current/scripts/prefabs/sword_lunarplant.lua#L309-L310)。 | 同一注册被实例化两次：剑主模拟 `fn` 生成 `blade1`、`blade2`（[L242-L243](../../../raw/current/scripts/prefabs/sword_lunarplant.lua#L242-L243)），第二枚在生成后改播 `swap_loop2`（[L244](../../../raw/current/scripts/prefabs/sword_lunarplant.lua#L244)）；两枚都使用本体的随机帧（[L240-L246](../../../raw/current/scripts/prefabs/sword_lunarplant.lua#L240-L246)）。跟随帧段分别是 0–3 与 5–8（[L76-L77](../../../raw/current/scripts/prefabs/sword_lunarplant.lua#L76-L77)）。 |
| `thuriblebody` | `thuriblebodyfn`（[L302-L327](../../../raw/current/scripts/prefabs/thurible.lua#L302-L327)）：Transform／AnimState／Network → bank／build `thurible`、`idle_body_loop` 循环、`SetFinalOffset(1)`、`FX`（[L309-L314](../../../raw/current/scripts/prefabs/thurible.lua#L309-L314)）→ Pristine → 客户端早退 → 主模拟随机帧、`persists=false`（[L318-L324](../../../raw/current/scripts/prefabs/thurible.lua#L318-L324)）。注册 [L329-L330](../../../raw/current/scripts/prefabs/thurible.lua#L329-L330)，**不传 assets**。 | 工厂不装 `Follower`；香炉 `onequip` 生成后再 `AddFollower` 并跟随 `swap_object`（[L133-L139](../../../raw/current/scripts/prefabs/thurible.lua#L133-L139)）。皮肤名与 `skin_id` 作为 `SpawnPrefab` 参数传入（[L133](../../../raw/current/scripts/prefabs/thurible.lua#L133)）；换肤时 `thurible_init_fn`／`clear_fn` 对 `_body` 调 `AnimState:SetSkin`／`SetBuild`（[prefabskin.lua:1704-L1715](../../../raw/current/scripts/prefabskin.lua#L1704-L1715)）。 |
| `slingshotparts_fx` | `partsfxfn`（[L882-L907](../../../raw/current/scripts/prefabs/slingshot.lua#L882-L907)）：Transform／AnimState／Follower／Network → `FX` → bank／build `slingshot`、默认播 `swap_1_to_5`（[L890-L894](../../../raw/current/scripts/prefabs/slingshot.lua#L890-L894)）→ Pristine → 客户端挂 `OnEntityReplicated` 登记高亮 → 主模拟 `persists=false`（[L896-L906](../../../raw/current/scripts/prefabs/slingshot.lua#L896-L906)）。注册 [L916](../../../raw/current/scripts/prefabs/slingshot.lua#L916)，**不传 assets**。 | 五个弹弓变体共用该注册与同一 `OnEquip`（依赖表 [L33-L56](../../../raw/current/scripts/prefabs/slingshot.lua#L33-L56)、[共用回调](../../../raw/current/scripts/prefabs/slingshot.lua#L543-L545)）。`OnEquip` 一次生成 6 枚，分别播 `swap_1_to_5`／`swap_6`／`swap_7`／`swap_8`／`swap_9`／`swap_17`（[L296-L304](../../../raw/current/scripts/prefabs/slingshot.lua#L296-L304)）；皮肤与部件符号逐枚覆写（[L306-L321](../../../raw/current/scripts/prefabs/slingshot.lua#L306-L321)）。 |
| `snowmanhat_fx` | `hatfxfn`（[L738-L769](../../../raw/current/scripts/prefabs/snowman.lua#L738-L769)）：Transform／AnimState／Follower／Network → bank／build `snowball`、默认 `hat_small` → `equipmentmodel`＋`FX` 标签 → 非专服挂 `OnRemoveEntity` 反登记 → Pristine → 客户端 `OnEntityReplicated` 登记高亮 → 主模拟加 `inventory`（`maxslots=0`）。注册 [L850](../../../raw/current/scripts/prefabs/snowman.lua#L850)。 | 唯一带 `inventory` 的 FX：帽子在 `SnowmanDecoratable:EquipHat_Internal` 中被 `inventory:Equip`（[snowmandecoratable.lua:635](../../../raw/current/scripts/components/snowmandecoratable.lua#L635)），用 `GetEquippedItem(EQUIPSLOTS.HEAD)` 读回（[L585](../../../raw/current/scripts/components/snowmandecoratable.lua#L585)）。宿主按顶层堆叠尺寸改播 `hat_<size>` 并 `Pause`（[L623-L624](../../../raw/current/scripts/components/snowmandecoratable.lua#L623-L624)），再设共享随机帧（[L633-L634](../../../raw/current/scripts/components/snowmandecoratable.lua#L633-L634)）。没有 `persists=false`。 |
| `snowball_rolling_fx` | `rollingfxfn`（[L816-L845](../../../raw/current/scripts/prefabs/snowman.lua#L816-L845)）：Transform／AnimState／Network → `decor`＋`NOCLICK` → 四面 → bank／build `snowball`、`fx_roll` → `SetFinalOffset(2)`（[L823-L831](../../../raw/current/scripts/prefabs/snowman.lua#L823-L831)）→ Pristine → 客户端早退 → 主模拟监听 `animover`、`persists=false`、导出 `KillFx`（[L835-L844](../../../raw/current/scripts/prefabs/snowman.lua#L835-L844)）。注册 [L852](../../../raw/current/scripts/prefabs/snowman.lua#L852)。 | 没有 Follower：父挂雪球本体并四面朝向（[L167-L172](../../../raw/current/scripts/prefabs/snowman.lua#L167-L172)）。`animover` 在未 kill 时重播 `fx_roll`，循环靠重播（[L804-L810](../../../raw/current/scripts/prefabs/snowman.lua#L804-L810)）；`KillFx` 只置 `killed`（[L812-L814](../../../raw/current/scripts/prefabs/snowman.lua#L812-L814)），下一次 `animover` 才 `Remove`。 |

## 关联机制导航

### 生成点与宿主

| 生成点 | 本注册如何使用 | 端、阶段与条件 | 证据 |
|---|---|---|---|
| 亮茄魔杖本体 | 主模拟 `fn` 创建物品时生成 FX、设同一随机帧并先 `SetFxOwner(inst, nil)`；`onequip`／`onunequip` 再改挂或挂回本体。 | 主模拟生成；浮空跟随本体的 `swap_spear`，持握跟随持有者 `swap_object`；破损路径先由 `MakeForgeRepairable` 卸下，再 `SetIsBroken` 隐藏 FX，修复时 `Show` 并重置帧 0。 | [生成](../../../raw/current/scripts/prefabs/staff_lunarplant.lua#L201-L206)、[装备](../../../raw/current/scripts/prefabs/staff_lunarplant.lua#L45-L66)、[破损与修复](../../../raw/current/scripts/prefabs/staff_lunarplant.lua#L122-L157)、[先卸下再破损](../../../raw/current/scripts/standardcomponents.lua#L1851-L1865) |
| 亮茄剑本体 | 主模拟 `fn` 创建物品时生成两枚 blade、设同一随机帧；`blade2` 改播 `swap_loop2`；`onequip`／`onunequip` 通过 `SetFxOwner` 改挂。 | 主模拟生成；两枚分别跟随 `swap_object` 的 0–3／5–8 帧段；破损路径先由 `MakeForgeRepairable` 卸下，`OnBroken` 只切 broken 动画与破损状态（不隐藏 blade），修复时把两枚帧重置 0。 | [生成](../../../raw/current/scripts/prefabs/sword_lunarplant.lua#L240-L248)、[装备](../../../raw/current/scripts/prefabs/sword_lunarplant.lua#L109-L132)、[破损与修复](../../../raw/current/scripts/prefabs/sword_lunarplant.lua#L174-L204) |
| 暗影香炉本体 | `onequip` 先移除旧身体，再用皮肤参数生成 `thuriblebody`，父挂持有者、`AddFollower`、跟随 `swap_object`，并监听持有者 `newstate` 重新切换符号。 | 主模拟生成；卸下、物品移除与再次装备都会删除旧身体；`onequiptomodel`（装备到模型）只熄烟不删身体。 | [装备](../../../raw/current/scripts/prefabs/thurible.lua#L122-L149)、[卸下](../../../raw/current/scripts/prefabs/thurible.lua#L151-L169)、[移除](../../../raw/current/scripts/prefabs/thurible.lua#L74-L84) |
| 弹弓五个变体 | `OnEquip` 先删旧 FX，再 `CreateFollowFx` 生成 6 枚，逐枚设父挂、动画、跟随帧段、高亮（非专服）与附加色（持有者有 `colouradder` 时）。 | 主模拟生成；`OnUnequip` 删除全部并置空；`OnEquipToModel` 只关容器，不额外生成或清理 FX。 | [OnEquip](../../../raw/current/scripts/prefabs/slingshot.lua#L290-L328)、[CreateFollowFx](../../../raw/current/scripts/prefabs/slingshot.lua#L194-L206)、[OnUnequip](../../../raw/current/scripts/prefabs/slingshot.lua#L330-L350) |
| 雪人装饰动作 | `DECORATESNOWMAN` 的帽子分支调 `SnowmanDecoratable:EquipHat`；`EquipHat_Internal` 首次生成 `snowmanhat_fx`、父挂雪人、按堆叠算偏移并跟随 `follow_hat`，之后只替换 `inventory` 里的帽子。 | 主模拟；非专服登记高亮；再次换帽只 `DropItem` 旧帽；卸帽／敲碎时 `DropEverything` 后移除。 | [动作分支](../../../raw/current/scripts/actions.lua#L6118-L6120)、[EquipHat_Internal](../../../raw/current/scripts/components/snowmandecoratable.lua#L592-L640)、[UnequipHat](../../../raw/current/scripts/components/snowmandecoratable.lua#L642-L654)、[敲碎路径](../../../raw/current/scripts/prefabs/snowman.lua#L531-L541) |
| 推雪球的周期任务 | `OnStartPushing` 建 0.25 秒周期任务；`_GrowSnowballSize` 在雪覆盖且地面允许时生成一次 `snowball_rolling_fx`，父挂雪球本体。 | 主模拟生成；停止推动、雪覆盖消失、FX 收到 `onremove`／`enterlimbo` 均走 `KillFx` 或先解绑再 `KillFx`。 | [开始推](../../../raw/current/scripts/prefabs/snowman.lua#L185-L193)、[生成](../../../raw/current/scripts/prefabs/snowman.lua#L137-L183)、[停止推](../../../raw/current/scripts/prefabs/snowman.lua#L80-L110)、[解绑](../../../raw/current/scripts/prefabs/snowman.lua#L124-L135) |

### 父挂、跟随与高亮

| 机制 | 涉及注册 | 行为与边界 | 证据 |
|---|---|---|---|
| `entity:SetParent`（裸父挂） | 六项全部 | 只建引擎父子变换，不登记 `EntityScript.children`；本页没有任何 `AddChild`。宿主 `Remove` 的 children 循环覆盖不到这些子实体，引擎级联回收未运行核验。 | [魔杖本体挂回](../../../raw/current/scripts/prefabs/staff_lunarplant.lua#L25)、[剑跟随](../../../raw/current/scripts/prefabs/sword_lunarplant.lua#L74-L77)、[AddChild 对照](../../../raw/current/scripts/entityscript.lua#L921-L934)、[children 移除循环](../../../raw/current/scripts/entityscript.lua#L1779-L1784) |
| `Follower:FollowSymbol` | `staff_lunarplant_fx`、`sword_lunarplant_blade_fx`、`thuriblebody`、`slingshotparts_fx`、`snowmanhat_fx`；`snowball_rolling_fx` 无 | 魔杖／剑：持握跟随 `swap_object`，浮空跟随本体 `swap_spear`；香炉身体：`swap_object` 偏移 (68,-130,0)；弹弓：`swap_object` 偏移 0，并按 6 个帧段分层；雪人帽：`follow_hat`，偏移取 `x0`／`-y0`。 | [魔杖切换](../../../raw/current/scripts/prefabs/staff_lunarplant.lua#L12-L30)、[剑切换](../../../raw/current/scripts/prefabs/sword_lunarplant.lua#L67-L93)、[香炉](../../../raw/current/scripts/prefabs/thurible.lua#L133-L139)、[弹弓](../../../raw/current/scripts/prefabs/slingshot.lua#L194-L206)、[雪人帽](../../../raw/current/scripts/components/snowmandecoratable.lua#L608-L626) |
| 跟随帧窗口 | `sword_lunarplant_blade_fx`、`slingshotparts_fx` | 剑的两枚实例用第 8、9 实参 0,3／5,8；弹弓 6 枚用 0,5／5／6／7／8／16。同快照 `shadow_battleaxe.lua` 的同形调用把这两个参数命名为 `frame_begin`／`frame_end`，本页据此按“跟随帧窗口”描述；引擎参数定义不在快照 Lua 内。 | [剑](../../../raw/current/scripts/prefabs/sword_lunarplant.lua#L76-L77)、[弹弓](../../../raw/current/scripts/prefabs/slingshot.lua#L298-L303)、[同形命名](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L869-L877)、[调用](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L936) |
| `highlightchild`（组件） | `staff_lunarplant_fx`、`sword_lunarplant_blade_fx` | 组件在 Pristine 前挂两侧；`SetOwner` 走 `net_entity` 属主同步，客户端收 dirty 后本地登记；专服跳过高亮簿记。魔杖与剑浮空时都把属主设为本体；持握时设持有者（剑两枚同设）。 | [魔杖](../../../raw/current/scripts/prefabs/staff_lunarplant.lua#L20-L28)、[剑](../../../raw/current/scripts/prefabs/sword_lunarplant.lua#L78-L91)、[组件同步](../../../raw/current/scripts/components/highlightchild.lua#L10-L48) |
| 手写高亮登记 | `slingshotparts_fx`、`snowmanhat_fx` | 两者工厂都不挂 `highlightchild`，而是在生成时（弹弓非专服、雪人帽非专服）或客户端 `OnEntityReplicated` 里 `table.insert(parent.highlightchildren, fx)`；移除时反登记（弹弓靠 `OnRemoveEntity` 闭包，雪人帽靠 `hat_OnRemoveEntity`）。 | [弹弓登记](../../../raw/current/scripts/prefabs/slingshot.lua#L177-L192)、[弹弓复制](../../../raw/current/scripts/prefabs/slingshot.lua#L875-L880)、[雪人帽登记](../../../raw/current/scripts/prefabs/snowman.lua#L720-L736) |
| `colouradder`（仅主模拟） | `staff_lunarplant_fx`、`sword_lunarplant_blade_fx` | FX 在 Pristine 后单独加 `colouradder`，持有者有 `colouradder` 时 `AttachChild`，换主人时先 `DetachChild`。没有 `colouraddersync`，`SetAddColour` 只在主模拟生效；跨端颜色未运行核验。 | [魔杖接线](../../../raw/current/scripts/prefabs/staff_lunarplant.lua#L13-L23)、[剑接线](../../../raw/current/scripts/prefabs/sword_lunarplant.lua#L68-L83)、[AttachChild／DetachChild](../../../raw/current/scripts/components/colouradder.lua#L24-L49)、[[reference/components/colouradder|colouradder]] |

### 清理与资源

| 注册名 | 清理路径 | 未覆盖的边界 | 资源与入口 |
|---|---|---|---|
| `staff_lunarplant_fx` | 卸下只把 FX 挂回本体（`SetFxOwner(inst, nil)`）；破损时 `Hide`、修复时 `Show`；`persists=false`。 | 物品被删除时没有显式 `Remove`，子实体是否随宿主回收未运行核验。 | `anim/staff_lunarplant.zip`，由魔杖与 FX 共用 assets 表（[L1-L4](../../../raw/current/scripts/prefabs/staff_lunarplant.lua#L1-L4)、[注册](../../../raw/current/scripts/prefabs/staff_lunarplant.lua#L271-L272)） |
| `sword_lunarplant_blade_fx` | 同魔杖：卸下挂回本体；破损先卸下，修复时把两枚帧重置 0（不隐藏 blade）；`persists=false`。 | 两枚实例在物品删除时都没有显式清理，引擎级联未核验。 | `anim/sword_lunarplant.zip`，两枚实例共用（[L1-L4](../../../raw/current/scripts/prefabs/sword_lunarplant.lua#L1-L4)） |
| `thuriblebody` | 卸下时先恢复整件 `swap_object` 符号再 `Remove` 身体；物品 `OnRemove` 也移除身体；身体被外部移除时 `onremovebody` 清 `_thurible._body`；`persists=false`。 | 换肤的跨端复现、身体在持有者死亡／传送时的去向未运行核验。 | 注册不带 assets；依赖同文件 `anim/thurible.zip`（[L1-L5](../../../raw/current/scripts/prefabs/thurible.lua#L1-L5)）；`swap_thurible.zip` 用于持有者符号 |
| `slingshotparts_fx` | `OnUnequip` 逐个 `Remove` 并置 `inst.fx = nil`；`persists=false`。 | 物品被删除时没有显式清理；`OnEquipToModel` 不清理 FX（模型实例与普通装备一样由 `onequip` 生成，再由后续 `OnUnequip` 清理）。 | 注册不带 assets；依赖同文件 `anim/slingshot.zip` 与部件 build（[L3-L11](../../../raw/current/scripts/prefabs/slingshot.lua#L3-L11)） |
| `snowmanhat_fx` | 卸帽、敲碎与摧毁时 `UnequipHat`：`DropEverything` 掉出帽子后 `Remove`；高亮反登记只在非专服。 | 未设 `persists=false` 且自带 `inventory`，帽子又由 `snowmandecoratable:OnSave` 另存 `data.hat`／`data.hatrnd`；两条存档路径的交互未运行核验。 | `anim/snowball.zip`，注册显式传入 assets（[L3-L6](../../../raw/current/scripts/prefabs/snowman.lua#L3-L6)、[L850](../../../raw/current/scripts/prefabs/snowman.lua#L850)） |
| `snowball_rolling_fx` | 停止推、雪覆盖消失、自身 `onremove`／`enterlimbo` 都会 `KillFx`；解绑路径先把父挂置 nil、保留世界坐标与朝向再 `KillFx`；下一次 `animover` 才 `Remove`；`persists=false`。 | `KillFx` 后的最长可见延迟等于剩余动画长度；宿主直接删除时的引擎回收未核验。 | `anim/snowball.zip`，注册显式传入 assets（[L3-L6](../../../raw/current/scripts/prefabs/snowman.lua#L3-L6)、[L852](../../../raw/current/scripts/prefabs/snowman.lua#L852)） |

## 典型接入

### 装备时生成持握层并跟随持有者符号

- 前提与选择：宿主物品已有 `equippable`，需要在装备期间显示独立 FX。照 `slingshot`／`thurible` 的时序：在 `onunequip` 删除旧实例，在 `onequip` 生成、`SetParent`、`FollowSymbol("swap_object")`，需要高亮就挂 `highlightchild`；不要只 `SetParent` 就指望符号跟随。
- 生效与撤销：主模拟生成；卸下必须先 `Remove` 再清字段，否则再次装备会叠加实例。`highlightchild:SetOwner` 自带网络属主同步，专用服务器只跑回调不记账。
- 原版依据：[`CreateFollowFx`](../../../raw/current/scripts/prefabs/slingshot.lua#L194-L206)、[`OnUnequip`](../../../raw/current/scripts/prefabs/slingshot.lua#L330-L350)、[香炉身体生成](../../../raw/current/scripts/prefabs/thurible.lua#L122-L149)。

```lua
local function AttachHoldFx(inst, owner)
    if inst.holdfx ~= nil then
        inst.holdfx:Remove()
        inst.holdfx = nil
    end
    local fx = SpawnPrefab("staff_lunarplant_fx") -- 换成目标 FX prefab
    fx.entity:SetParent(owner.entity)
    fx.Follower:FollowSymbol(owner.GUID, "swap_object", 0, 0, 0, true)
    fx.components.highlightchild:SetOwner(owner)
    inst.holdfx = fx
end

local function DetachHoldFx(inst, owner)
    if inst.holdfx ~= nil then
        inst.holdfx.components.highlightchild:SetOwner(nil)
        inst.holdfx:Remove()
        inst.holdfx = nil
    end
end
```

- 调整与验证：`swap_object` 是持握物位置符号，换宿主必须核对符号名；示例用带 `highlightchild` 的 `staff_lunarplant_fx`，若换用不带该组件的 FX（如 `slingshotparts_fx`），要按 `SetHighlightChildren` 手写登记与反登记。示例只做语法检查，未在游戏内执行。

### 多部件持握层用帧窗口错开

- 前提与选择：一个持握动画里有多组部件、需要每帧显示不同层时，原版用多个同名 FX 实例，每个实例播自己的动画并用第 8、9 实参指定跟随帧窗口。剑是两枚 0–3／5–8，弹弓是六枚 0–5／5／6／7／8／16。
- 生效与撤销：原版两种收尾不同——剑在物品创建时生成、卸下时挂回本体而不删除；弹弓在 `onequip` 生成、`OnUnequip` 逐个删除。示例按“生成＋删除”写，若要复刻剑的常驻层，把 `DetachHoldFx` 改成挂回物品并跟随 `swap_spear`。每枚都要单独设父挂、跟随、随机帧（原版把同一随机帧分给所有实例）与符号覆写。
- 原版依据：[剑的两枚](../../../raw/current/scripts/prefabs/sword_lunarplant.lua#L240-L248)、[弹弓的六枚](../../../raw/current/scripts/prefabs/slingshot.lua#L296-L304)、[`frame_begin`／`frame_end` 同形调用](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L936)。

```lua
local LAYERS =
{
    { anim = "swap_loop1", frame_begin = 0, frame_end = 3 },
    { anim = "swap_loop2", frame_begin = 5, frame_end = 8 },
}

local function AttachLayers(owner, frame)
    local layers = {}
    for i, def in ipairs(LAYERS) do
        local fx = SpawnPrefab("sword_lunarplant_blade_fx")
        fx.AnimState:PlayAnimation(def.anim, true)
        fx.AnimState:SetFrame(frame)
        fx.entity:SetParent(owner.entity)
        fx.Follower:FollowSymbol(owner.GUID, "swap_object", nil, nil, nil, true, nil, def.frame_begin, def.frame_end)
        fx.components.highlightchild:SetOwner(owner)
        layers[i] = fx
    end
    return layers
end
```

- 调整与验证：帧窗口只对第 8、9 实参的顺序与取值负责；原版剑第 4 帧没有实例覆盖，是否有意留空未核验。换动画时需同步总帧数与窗口，否则会出现整层缺失或重叠。示例只做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **误区：这些 FX 是玩法主体**：六项都不承担伤害、耐久、燃料或堆叠数值；唯一例外是 `snowmanhat_fx` 会持有帽子实体，但它仍不是可装备物品。宿主页均为 Wiki 待建。
- **边界：生成时机差四种**：魔杖／剑在物品创建时就生成（装备只是改挂），香炉／弹弓在 `onequip` 生成后删除，雪人帽由装饰动作生成并在整个佩戴期常驻，滚雪球 FX 只在“雪覆盖＋推动”的周期任务里生成。按“装备时一定有 FX”写 PostInit 会漏掉前两项的常驻实例。
- **边界：裸 `SetParent` 不登记 children**：六项都不走 `AddChild`，宿主 `Remove` 的 children 循环不会带走它们；魔杖／剑在物品删除时也没有显式清理。需要“宿主删了 FX 一定走”时改用 `AddChild`，否则引擎级联要另行核验。
- **陷阱：`sword_lunarplant_blade_fx` 一个注册两种动画**：先 `SpawnPrefab` 得到默认 `swap_loop1`，第二枚必须在生成后改播 `swap_loop2`；只改注册的默认动画会让两枚重叠。
- **陷阱：`KillFx` 不是立即删除**：`snowball_rolling_fx` 的 `KillFx` 只置 `killed`，要等当前 `fx_roll` 播完的 `animover` 才 `Remove`；在这之前 FX 仍在场景里，调用后不要再按“已移除”处理。
- **边界：`colouradder` 无同步**：魔杖／剑的附加色只走主模拟 `colouradder`，FX 没有 `colouraddersync`，客户端的颜色显示未运行核验（组件页已说明无同步时只在主机可见）。
- **边界：两个注册不带 assets**：`thuriblebody`、`slingshotparts_fx` 的 `Prefab(...)` 只传工厂，动画包由同文件宿主 prefab 的 assets 表注册；单独依赖这两个 FX 的模组要自行补 `anim/thurible.zip`／`anim/slingshot.zip`。
- **边界：`thuriblebody` 的 `Follower` 由宿主补装**：工厂不 `AddFollower`；单独 `SpawnPrefab("thuriblebody")` 不会跟随，必须先 `entity:AddFollower()` 再 `FollowSymbol`。
- **边界：雪人帽的存档双路径**：`snowmanhat_fx` 未设 `persists=false` 且带 `inventory`，帽子数据又由 `snowmandecoratable:OnSave` 存 `data.hat`／`data.hatrnd` 并在 `OnLoad` 重新装备；重复生成或丢失的取舍未运行核验。帽子戴上后 `CheckLiftAndPushable` 移除 `pushable`（[snowman.lua:219-L233](../../../raw/current/scripts/prefabs/snowman.lua#L219-L233)）、`CanStack` 因 `HasHat` 拒绝堆叠（[snowmandecoratable.lua:457-L464](../../../raw/current/scripts/components/snowmandecoratable.lua#L457-L464)），所以装备时一次算好的偏移不会随堆叠变化。
- **边界：雪人帽动画由宿主决定**：工厂默认 `hat_small`，宿主按顶层堆叠尺寸播 `hat_<size>` 并 `Pause`；不经过 `EquipHat_Internal` 直接生成会停在默认动画。
- **边界：`OnEquipToModel` 不负责生成或清理 FX**：`equippable:Equip` 先跑 `onequip`，再在 `owner` 带 `equipmentmodel` 时跑 `onequiptomodel`（[L99-L106](../../../raw/current/scripts/components/equippable.lua#L99-L106)）；弹弓的 `OnEquipToModel` 只关容器（[L352-L356](../../../raw/current/scripts/prefabs/slingshot.lua#L352-L356)），模型上的 FX 仍由 `onequip` 生成，不要把它当“模型不生成 FX”的开关。

## 验证记录

- 源码审阅：2026-11-04，Steam buildid 26335394。逐文件核对 `staff_lunarplant.lua`、`sword_lunarplant.lua`、`thurible.lua`、`slingshot.lua`、`snowman.lua` 的六个注册名、工厂、端分界、生成点、父挂／跟随、帧窗口、高亮与清理；并核验 `components/snowmandecoratable.lua`（帽子生成、跟随偏移、卸帽与存档）、`components/highlightchild.lua`（属主网络同步）、`components/colouradder.lua`（AttachChild／DetachChild）、`components/equippable.lua`（`equipmentmodel` 时额外跑 `onequiptomodel`）、`prefabskin.lua`（香炉换肤更新 `_body`）、`actions.lua`（`DECORATESNOWMAN` 帽子分支）、`standardcomponents.lua`（`MakeForgeRepairable` 先卸下再回调破损）、`prefabs/shadow_battleaxe.lua`（`frame_begin`／`frame_end` 同形命名）、`entityscript.lua`（`persists` 默认 true、`AddChild` 与 `children` 循环）。六个完整注册名在 `raw/current/scripts` 全域反查，命中仅为各自文件内的依赖声明、生成点与注册行；另对五个宿主文件检索字符串拼接，未发现用这六个注册名或宿主名拼出的额外 FX 生成（命中项是动画名、图标名与弹药投射物名）。
- 名称核验：`languages/chinese_s.po` 定向检索六个注册名的 `STRINGS.NAMES` 键均无命中；宿主键位置见“快速判断”。标题使用效果描述，未把宿主名当 FX 官方名。
- 分档说明：覆盖 6 个注册、6 个工厂、4 类生成时机与两种跟随模型，需要整体对照，故按 `size_class: complex`。
- 游戏运行：未运行；未验证动画包实际帧数／循环标记、各 FX 的跨端外观（符号覆写、换肤、`Hide`／`Show`、暂停帧）、专服与晚加入、宿主死亡或删除后的子实体回收、雪人帽存档往返。
- 结构检查：`validate_page.py`（complex 档）与 `validate_source_links.py --strict` 均通过；两段 Lua 示例用 `parser_probe.exe --syntax-only` 检查通过（只查语法，不执行）。

## 相关链接

- [[reference/prefabs/armor_lunarplant_glow_fx|装备与帽饰跟随光效]]：同为装备附属层，但那份用“控制器＋本地非网络子帧”模型，与本页的联网实体模型对照。
- [[reference/prefabs/wagpunksteam_hat_up|WAGpunk 装备蒸汽]]：短命通用表装备表现，对照本页常驻持握层。
- [[reference/prefabs/king_cane_fx|宿主持有与跟随的附属 FX 族]]：`SetFxOwner` 双向改挂与符号跟随的另一种写法。
- [[reference/prefabs/brilliance_projectile_fx|亮茄魔杖辉光弹]]、[[reference/prefabs/hitsparks_fx|命中火花特效]]：魔杖／剑的玩法侧 FX，不属本页持握层。
- [[reference/components/highlightchild|highlightchild]]、[[reference/components/colouradder|colouradder]]：本页两项表现组件。
- 返回 [[reference/prefabs/fx-index|特效 Prefab 索引]]；世界环境类对照见 [[reference/prefabs/fx-world-index|特效 Prefab 索引·生物与环境]]。
