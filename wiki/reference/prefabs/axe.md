---
title: "斧头族（axe、goldenaxe、moonglassaxe）"
created: "2026-11-05"
updated: "2026-11-08"
type: prefab
summary: "同文件 MakeAxe 工厂注册的三把手持斧：基础斧、黄金斧（消耗折算）与月光玻璃斧（对纯影目标低磨损），覆盖耐久消耗方向、伤害、获取途径与 Woodie 的 Lucy 替换交互。"
aliases: ["axe", "goldenaxe", "moonglassaxe", "斧头", "斧子", "黄金斧头", "黄金斧", "月光玻璃斧", "Luxury Axe", "Moon Glass Axe", "斧头族", "砍树工具", "斧头耐久"]
tags: [prefab, item, equipment]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/axe.lua", "components/equippable.lua", "components/tool.lua", "components/weapon.lua", "components/finiteuses.lua", "components/damagetypebonus.lua", "components/possessedaxe.lua", "prefabs/woodie.lua", "componentutil.lua", "standardcomponents.lua", "tuning.lua", "constants.lua", "recipes.lua", "actions.lua", "bufferedaction.lua", "entityscript.lua", "prefabskin.lua", "prefabs/skinprefabs.lua", "prefabs/moon_altar.lua", "map/layouts.lua", "map/rooms/forest/terrain_moonisland.lua", "map/static_layouts/moontreehiddenaxe.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 斧头族（axe、goldenaxe、moonglassaxe）

## 快速判断

- 职责与适用：`prefabs/axe.lua` 用同一 `MakeAxe` 工厂注册的可拾取手持斧。基础斧 `axe`、黄金斧 `goldenaxe`（耐久消耗按 1/GOLDENTOOLFACTOR 折算）、月光玻璃斧 `moonglassaxe`（对纯影目标低磨损＋对暗影生物增伤）；都接 CHOP 工具与 HANDS 槽武器，耐久耗尽整个实体 Remove。[工厂](../../../raw/current/scripts/prefabs/axe.lua#L38-L133)、[注册](../../../raw/current/scripts/prefabs/axe.lua#L291-L294)
- 优先使用：改耐久／消耗折算查下方数值表与 [有限耐久组件](../components/finiteuses.md)；从零造同类手持工具查 [[cookbook/equipment/hand]]；复刻"对特定目标低磨损"读[典型接入](#借鉴按目标切换的攻击磨损)。
- 重要副作用：装备回调维护 swap_object、手臂显示与皮肤事件；三注册都带 `possessable_axe` 标签，Woodie 监听自身 `working` 事件（对 CHOP 目标）时会替换为 Lucy 并 Remove 原斧；耐久耗尽 `SetOnFinished(inst.Remove)` 直接移除、无掉落。[耗尽回调](../../../raw/current/scripts/prefabs/axe.lua#L115)、[Lucy 消费](../../../raw/current/scripts/prefabs/woodie.lua#L681-L702)
- 修改边界：公共端挂标签与 floater，主模拟挂组件；`TheNet:GetServerGameMode() == "quagmire"` 时工厂不挂 finiteuses／weapon（也不打 `weapon` 标签）。本页不覆盖同文件注册的 `fumaroleaxe` 温度机制（见[注册与构造](#注册与构造)说明）与皮肤贴图内容。[quagmire 分支](../../../raw/current/scripts/prefabs/axe.lua#L110-L121)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `axe` | `MakeAxe("axe", nil, nil, nil, assets)`，data 为 nil 走全默认（bank／build `axe`、swap `swap_axe`、maxfiniteuses=`AXE_USES`） | 无 postinit；皮肤经 prefabskin 的 `axe_init_fn`。 [注册](../../../raw/current/scripts/prefabs/axe.lua#L291)、[工厂默认值](../../../raw/current/scripts/prefabs/axe.lua#L39-L47) |
| `goldenaxe` | `MakeAxe(..., gold_master_postinit, GOLD_DATA, golden_assets)`，GOLD_DATA 换 bank／build／swap 为 goldenaxe 组 | master postinit 仅改消耗折算与攻击磨损，并监听 `equipped` 播黄金装备音效。[GOLD_DATA](../../../raw/current/scripts/prefabs/axe.lua#L137-L160) |
| `moonglassaxe` | `MakeAxe(..., glass_master_postinit, GLASS_DATA, moonglass_assets)`，GLASS_DATA 换 glassaxe 组 | master postinit 改砍树效率、CHOP 消耗、伤害与攻击磨损，并加 `damagetypebonus`。[GLASS_DATA](../../../raw/current/scripts/prefabs/axe.lua#L171-L192) |

共同初始化（`fn`，[axe.lua#L63-L130](../../../raw/current/scripts/prefabs/axe.lua#L63-L130)）：实体组件与 `MakeInventoryPhysics` → bank／build／动画 → 公共端标签 `sharp`、`possessable_axe`、`tool`（非 quagmire 另打 `weapon`）→ `MakeInventoryFloatable("small", 0.05, {1.2, 0.75, 1.2}, true, -11, swap_data)` → `common_postinit` → `SetPristine`（非主模拟返回）→ 主模拟依次 `inspectable`、`inventoryitem`、`equippable`、`tool`（CHOP 效率 1）→ 非 quagmire 时 `finiteuses`（上限＝当前＝data.maxfiniteuses 或 `AXE_USES`，耗尽回调 `inst.Remove`，CHOP 消耗 1）与 `weapon`（伤害 `AXE_DAMAGE`）→ `MakeHauntableLaunch` → `master_postinit`。data 还支持 `nofiniteuses`（关耐久）与 `onequipfn`／`onunequipfn`（换装备回调），本文件四注册均未用。装备槽不逐 prefab 设置，用组件默认 HANDS。[equippable 默认](../../../raw/current/scripts/components/equippable.lua#L29)、[EQUIPSLOTS.HANDS](../../../raw/current/scripts/constants.lua#L656)

名称键经简中 .po `msgctxt` 核验：`STRINGS.NAMES.AXE`=斧头（Axe）、`NAMES.GOLDENAXE`=黄金斧头（Luxury Axe）、`NAMES.MOONGLASSAXE`=月光玻璃斧（Moon Glass Axe）。[AXE](../../../raw/current/scripts/languages/chinese_s.po#L338964-L338967)、[GOLDENAXE](../../../raw/current/scripts/languages/chinese_s.po#L343354-L343357)、[MOONGLASSAXE](../../../raw/current/scripts/languages/chinese_s.po#L345634-L345637)

数值与消耗（tuning 键＋实际值；`wilson_attack`=34×倍率 1，[tuning.lua#L32-L37](../../../raw/current/scripts/tuning.lua#L32-L37)）：

| 注册名 | 装备槽 | 耐久上限 | 每 CHOP 消耗（耐久点） | 砍树效率（工作量/动作） | 武器伤害（生命值） | 每次攻击磨损（耐久点） |
|---|---|---|---|---|---|---|
| `axe` | HANDS | `TUNING.AXE_USES`=100 | 1 | 1 | `AXE_DAMAGE`=`wilson_attack`×0.8=27.2 | 1（weapon 默认 `attackwear` 缺省值） |
| `goldenaxe` | HANDS | 100 | 1/`GOLDENTOOLFACTOR`（=4×1）=0.25 | 1 | 27.2 | 0.25（`weapon.attackwear`） |
| `moonglassaxe` | HANDS | 100 | `MOONGLASSAXE.CONSUMPTION`=1.25 | `MOONGLASSAXE.EFFECTIVENESS`=2.5 | `MOONGLASSAXE.DAMAGE`=`wilson_attack`=34；对 `shadowsubmissive` 目标 ×`DAMAGE_VS_SHADOW_BONUS`=1.25 | 每次攻击按目标重设：对纯影目标 `SHADOW_WEAR`=0.5，否则 `ATTACKWEAR`=2 |

折算方向：黄金斧改的是**消耗端**（每动作扣 1/4 点），上限仍 100，不是上限 400。源码推导的等效次数（上限÷消耗，未运行测量）：砍伐 axe 100／goldenaxe 400／moonglassaxe 80；攻击 axe 100／goldenaxe 400／moonglassaxe 50（对纯影目标 200）。[GOLDENTOOLFACTOR](../../../raw/current/scripts/tuning.lua#L95)、[AXE_USES](../../../raw/current/scripts/tuning.lua#L157)、[AXE_DAMAGE](../../../raw/current/scripts/tuning.lua#L358)、[MOONGLASSAXE 表](../../../raw/current/scripts/tuning.lua#L3838-L3845)

获取途径（均已在快照内核验）：

| 注册名 | 配方与科技 | 其他生成点 |
|---|---|---|
| `axe` | 小树枝 1＋燧石 1，`TECH.NONE` | — |
| `goldenaxe` | 小树枝 4＋金块 2，`TECH.SCIENCE_TWO` | — |
| `moonglassaxe` | 小树枝 2＋月光玻璃 3，`TECH.CELESTIAL_THREE`（={CELESTIAL=3}），站点标签 `celestial_station` | 月岛房间 `MoonIsland_Forest` 的 `MoonTreeHiddenAxe` 静态布局放 1 个 |

[axe 配方](../../../raw/current/scripts/recipes.lua#L393-L394)、[moonglassaxe 配方](../../../raw/current/scripts/recipes.lua#L838)、[TECH.CELESTIAL_THREE](../../../raw/current/scripts/constants.lua#L1257)、[celestial_station 挂标](../../../raw/current/scripts/prefabs/moon_altar.lua#L584)、[月岛房间](../../../raw/current/scripts/map/rooms/forest/terrain_moonisland.lua#L73)、[布局注册](../../../raw/current/scripts/map/layouts.lua#L1042)、[布局内容](../../../raw/current/scripts/map/static_layouts/moontreehiddenaxe.lua#L107-L113)

同文件第四注册 `fumaroleaxe`（气孔斧，走 FUMAROLE_DATA＋`MakeFumaroleTool` 温度机制）本页不覆盖，机制见 [[reference/prefabs/fumarole-tools]]（[注册行](../../../raw/current/scripts/prefabs/axe.lua#L294)）。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：equippable | 挂载并设 `_OnEquip`／`OnUnequip`：维护 ARM_carry／ARM_normal、swap_object 覆盖与皮肤事件 | 三注册同一接线；装备槽默认 HANDS | [[reference/components/equippable#关键接口]]；[回调接线](../../../raw/current/scripts/prefabs/axe.lua#L103-L105)、[_OnEquip](../../../raw/current/scripts/prefabs/axe.lua#L49-L59)、[OnUnequip](../../../raw/current/scripts/prefabs/axe.lua#L29-L36) |
| 组件：tool | `SetAction(CHOP, 1)` 挂砍树工具与 `CHOP_tool` 标签 | 三注册；moonglassaxe 在 master postinit 覆写效率 2.5 | [[reference/components/tool#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/axe.lua#L107-L108)、[覆写](../../../raw/current/scripts/prefabs/axe.lua#L180) |
| 组件：finiteuses | 上限＝当前＝maxfiniteuses；`SetOnFinished(inst.Remove)`；CHOP 消耗 1／0.25／1.25 | 非 quagmire 才挂载；每次成功的 CHOP 动作扣一次（`BufferedAction:Do` → `OnUsedAsItem` → `finiteuses:OnUsedAsItem`） | [[reference/components/finiteuses#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/axe.lua#L111-L117)、[消耗链](../../../raw/current/scripts/bufferedaction.lua#L22-L37)、[OnUsedAsItem](../../../raw/current/scripts/components/finiteuses.lua#L101-L122) |
| 组件：weapon | 伤害 `AXE_DAMAGE`；`OnAttack` 先调 onattack 回调再按 `attackwear` 扣 finiteuses | 非 quagmire 才挂载；moonglassaxe 改伤害并 `SetOnAttack(onattack_moonglass)` | [[reference/components/weapon#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/axe.lua#L119-L120)、[OnAttack 磨损](../../../raw/current/scripts/components/weapon.lua#L105-L121) |
| 组件：damagetypebonus | `AddBonus("shadowsubmissive", inst, 1.25)`：目标带该标签时伤害 ×1.25 | 仅 moonglassaxe；标签在影怪（暗影生物／梦魇／海洋影怪）身上 | [[reference/components/damagetypebonus#关键接口]]；[[reference/components/shadowsubmissive#快速判断]]；[挂载](../../../raw/current/scripts/prefabs/axe.lua#L189-L190)、[GetBonus 乘算](../../../raw/current/scripts/components/damagetypebonus.lua#L42-L59) |
| 组件：inspectable／inventoryitem | 检视与拾取 | 三注册，主模拟 | [[reference/components/inspectable#关键接口]]、[[reference/components/inventoryitem#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/axe.lua#L100-L101) |
| 组件：floater | 公共端 `MakeInventoryFloatable("small", 0.05, floater_scale, true, -11, swap_data)`，浮水换 swap 贴图 | 三注册；swap_data 随 data 换 build／符号 | [[reference/components/floater#关键接口]]；[调用](../../../raw/current/scripts/prefabs/axe.lua#L88)、[helper](../../../raw/current/scripts/standardcomponents.lua#L1772-L1790) |
| 组件：hauntable | `MakeHauntableLaunch`：被作祟时弹飞 | 三注册，主模拟 | [[reference/components/hauntable#按效果选工厂]]；[调用](../../../raw/current/scripts/prefabs/axe.lua#L123)、[helper](../../../raw/current/scripts/standardcomponents.lua#L1053-L1069) |
| API／数据：`PURE_SHADOW_TARGET_TAGS` | `onattack_moonglass` 判定目标是否纯影以切磨损档 | 定义于 componentutil.lua 的全局表 {shadow, shadowminion, shadowchesspiece, stalker, stalkerminion, shadowthrall, shadowboss}；glasscutter 同用，见 [[reference/prefabs/glasscutter]] | [定义](../../../raw/current/scripts/componentutil.lua#L175-L183)、[消费](../../../raw/current/scripts/prefabs/axe.lua#L164-L169) |
| API／数据：CHOP 消耗链 | `ACTIONS.CHOP.fn` → `DoToolWork`（取 tool 效率）→ 成功后 `OnUsedAsItem` 分发到各组件 | 服务端动作结算；效率与耐久消耗彼此独立 | [CHOP](../../../raw/current/scripts/actions.lua#L1748-L1752)、[DoToolWork](../../../raw/current/scripts/actions.lua#L1693-L1710)、[分发](../../../raw/current/scripts/entityscript.lua#L1811-L1816) |
| 关联 Prefab：lucy（possessedaxe） | Woodie 监听自身 `working`（CHOP 目标）消费 `possessable_axe`：目标动作匹配且身上无 lucy 时，SpawnPrefab("lucy")、记录 `revert_prefab`／`revert_uses`，Remove 当前斧并装备 lucy；lucy 还原时按注册名重生并恢复耐久（math.max(1, uses)） | 三注册都被覆盖；服务端 woodcutter 专属交互；lucy 侧行为见 [[reference/prefabs/lucy]] | [[reference/components/possessedaxe#关键接口]]；[消费](../../../raw/current/scripts/prefabs/woodie.lua#L681-L702)、[Revert](../../../raw/current/scripts/components/possessedaxe.lua#L167-L173) |
| 资源：anim／swap 动画包 | axe.zip／swap_axe.zip、goldenaxe.zip／swap_goldenaxe.zip、glassaxe.zip／swap_glassaxe.zip | 按 data.bank／build／swap_build 对应 | [资源声明](../../../raw/current/scripts/prefabs/axe.lua#L1-L17) |

皮肤接线（工厂内，[axe.lua#L49-L59](../../../raw/current/scripts/prefabs/axe.lua#L49-L59)、[axe.lua#L29-L36](../../../raw/current/scripts/prefabs/axe.lua#L29-L36)）：`GetSkinBuild()` 非空时装备／卸装推送 `equipskinneditem`／`unequipskinneditem` 事件，并用 `OverrideItemSkinSymbol("swap_object", skin_build, sym_name, GUID, swap_build)` 换持握贴图，否则普通 `OverrideSymbol`。已注册皮肤：`axe` 6 种（feathered／heart／invisible／northern／rose／victorian）、`goldenaxe` 2 种（invisible／northern），`moonglassaxe` 未检索到皮肤注册（prefabskin.lua／skinprefabs.lua 按 prefab 名 grep 无结果）；invisible 皮肤会关 floater 换 bank。[axe 皮肤](../../../raw/current/scripts/prefabs/skinprefabs.lua#L1124-L1187)、[goldenaxe 皮肤](../../../raw/current/scripts/prefabs/skinprefabs.lua#L6080-L6105)、[init／clear](../../../raw/current/scripts/prefabskin.lua#L877-L922)

## 典型接入

### 修改黄金斧的消耗折算

- 前提与选择：服务端 `modmain.lua`；沿用原工厂与组件，只调消耗端数值。改上限请走 [[reference/components/finiteuses#修改已有斧头总次数保留初始比例与原版耗尽行为]]，不要把折算重复乘进上限。
- 生效与撤销：postinit 在工厂之后执行，影响新实例；存档内已有实例按保存的绝对次数恢复。停用模组后新实例恢复 1/GOLDENTOOLFACTOR。
- 原版依据：gold_master_postinit 的 `SetConsumption` 与 `attackwear` 是两条独立折算（砍伐与攻击），只改其一不会影响另一条。[gold postinit](../../../raw/current/scripts/prefabs/axe.lua#L151-L160)

```lua
-- modmain.lua：黄金斧每次砍伐消耗改为 1/(GOLDENTOOLFACTOR*2) 点（等效 800 次砍伐）
AddPrefabPostInit("goldenaxe", function(inst)
    if GLOBAL.TheWorld.ismastersim and inst.components.finiteuses ~= nil then
        inst.components.finiteuses:SetConsumption(GLOBAL.ACTIONS.CHOP, 1 / (TUNING.GOLDENTOOLFACTOR * 2))
    end
end)
```

- 调整与验证：数值单位为耐久点/次；modmain 环境只直挂 `TUNING` 等白名单（mods.lua CreateEnvironment），`TheWorld`／`ACTIONS` 等运行时全局须经 `GLOBAL` 取；勿在回调内再扣 finiteuses（OnAttack 会按 attackwear 统一扣）。

### 借鉴按目标切换的攻击磨损

- 前提与选择：服务端 `modmain.lua`；复刻 `onattack_moonglass` 的"每次攻击按目标重设 `attackwear`"模式，给普通斧加纯影目标低磨损。`PURE_SHADOW_TARGET_TAGS` 是 componentutil.lua 定义在 `_G` 的全局，modmain 环境须经 `GLOBAL` 取（mod env 无 `__index=_G` 回退）。
- 生效与撤销：`Weapon:OnAttack` 每次攻击先执行该回调再结算磨损，目标合法性由回调内判断；quagmire 模式无 weapon 组件，须判空。
- 原版依据：磨损分支依赖 `PURE_SHADOW_TARGET_TAGS` 与 `attackwear` 结算顺序（onattack 先于 `Use`）。[onattack_moonglass](../../../raw/current/scripts/prefabs/axe.lua#L164-L169)、[结算顺序](../../../raw/current/scripts/components/weapon.lua#L105-L121)

```lua
-- modmain.lua：普通斧对纯影目标磨损 0.5，其余 1（与组件默认一致）
AddPrefabPostInit("axe", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local weapon = inst.components.weapon
    if weapon ~= nil then
        weapon:SetOnAttack(function(inst, attacker, target)
            inst.components.weapon.attackwear =
                target ~= nil and target:IsValid() and target:HasAnyTag(GLOBAL.PURE_SHADOW_TARGET_TAGS)
                and 0.5 or 1
        end)
    end
end)
```

- 调整与验证：数值单位为耐久点/次；勿在回调内再扣 finiteuses（OnAttack 会按 attackwear 统一扣）。

## 常见陷阱与限制

- **误区：黄金斧耐久上限是 400**：上限仍是 `AXE_USES`=100；折算在消耗端，每 CHOP 扣 1/GOLDENTOOLFACTOR=0.25 点、攻击磨损同为 0.25，等效 400 次。自定义上限时不要另乘 4。[gold postinit](../../../raw/current/scripts/prefabs/axe.lua#L151-L160)
- **边界：quagmire 游戏模式下没有 finiteuses／weapon**：工厂按 `GetServerGameMode() ~= "quagmire"` 才挂载这两个组件，公共端也不打 `weapon` 标签；读取或修改前须判空，否则该模式下报错。[分支](../../../raw/current/scripts/prefabs/axe.lua#L83-L86)、[axe.lua#L110-L121](../../../raw/current/scripts/prefabs/axe.lua#L110-L121)
- **边界：耗尽即整个实体 Remove**：`SetOnFinished(inst.Remove)`，无掉落无残骸；`finiteuses:OnSave` 只在当前数不等于上限时存 `uses`。[回调](../../../raw/current/scripts/prefabs/axe.lua#L115)、[存档](../../../raw/current/scripts/components/finiteuses.lua#L48-L51)
- **边界：`possessable_axe` 会被 Lucy 替换**：Woodie 收到自身 `working`（CHOP 目标）时若装备三斧之一且身上没有 lucy，原斧被 Remove 并换成 lucy；还原时按 `revert_prefab` 重生、耐久取 `math.max(1, revert_uses)`。自定义斧想要该交互就加同标签，不想要可在 postinit `RemoveTag`。[woodie onworked](../../../raw/current/scripts/prefabs/woodie.lua#L681-L705)
- **边界：moonglassaxe 的磨损不是常量**：`attackwear` 在每次攻击的 onattack 回调里按目标重设（纯影 0.5／其余 2）；读取当前磨损要查组件字段，不能假设固定值；首次攻击前该字段为 nil，weapon 组件按 1 结算。[onattack_moonglass](../../../raw/current/scripts/prefabs/axe.lua#L164-L169)

## 验证记录

- 源码审阅：2026-11-05，Steam buildid 26335394。核验 prefabs/axe.lua 全文（MakeAxe 工厂、三注册与 fumaroleaxe 注册行、gold／glass master postinit、皮肤事件接线）、equippable 默认 HANDS 槽、tool／weapon／finiteuses／damagetypebonus 关键方法、tuning 各键值、quagmire 分支、floater 与 MakeHauntableLaunch helper、三注册配方与 moonglassaxe 的月岛静态布局生成点、`PURE_SHADOW_TARGET_TAGS` 定义与消费、`possessable_axe` 消费者（woodie 自身 working→possessedaxe Revert）。名称键以 chinese_s.po `msgctxt` 定向核验（任务提示中的 `NAMES.MOOMGLASSAXE` 键不存在，实际键为 `NAMES.MOONGLASSAXE`）。
- 游戏运行：未运行。未覆盖：quagmire 实机行为、皮肤实装外观、Lucy 替换与还原的运行时序、月岛 setpiece 实际落点；27.2／42.5 等伤害数值为源码推导（wilson_attack=34×倍率 1），非运行测量。
- 结构检查：validate_page.py 与 strict 均通过。

返回 [[reference/prefabs/tools-field-index|工具与野外实用]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-08 build 26474873：本页随版本迁移升版（机械沿用；引用区间未受本次源码行号位移影响），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-06）。
