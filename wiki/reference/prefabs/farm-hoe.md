---
title: "园艺锄（farm_hoe、golden_farm_hoe）"
created: "2026-11-06"
updated: "2026-11-06"
type: prefab
summary: "MakeFarmHoe 工厂两注册的手持园艺锄族：右键锄地生成 farm_soil、耐久消耗与黄金锄 1/4 折算、HANDS 装备持握与皮肤事件、耗尽即移除并广播 toolbroke 的组装入口。"
aliases: ["farm_hoe", "golden_farm_hoe", "园艺锄", "黄金园艺锄", "Garden Hoe", "Splendid Garden Hoe", "金锄", "锄头", "锄地工具", "翻耕土壤", "园艺锄耐久"]
tags: [prefab, item, equipment]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/farm_hoe.lua", "components/farmtiller.lua", "components/equippable.lua", "components/weapon.lua", "components/finiteuses.lua", "components/map.lua", "components/fumaroletool.lua", "actions.lua", "componentactions.lua", "components/playeractionpicker.lua", "components/playercontroller.lua", "bufferedaction.lua", "entityscript.lua", "standardcomponents.lua", "stategraphs/SGwilson.lua", "prefabskin.lua", "prefabs/skinprefabs.lua", "recipes.lua", "tuning.lua", "modutil.lua", "mainfunctions.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 园艺锄（farm_hoe、golden_farm_hoe）

## 快速判断

- 职责与适用：`prefabs/farm_hoe.lua` 用同一 `MakeFarmHoe` 工厂注册的两把手持园艺锄：基础锄 `farm_hoe`、黄金锄 `golden_farm_hoe`（锄地消耗与攻击磨损按 1/`GOLDENTOOLFACTOR` 折算）。都是可拾取的 HANDS 装备兼弱武器，手持右键农田地皮锄地生成 `farm_soil`，耐久耗尽整个实体移除并广播 `toolbroke`。适合借鉴"一工厂＋data＋postinit"的工具组装、消耗折算与 TILL 动作接线。[工厂](../../../raw/current/scripts/prefabs/farm_hoe.lua#L44-L134)、[注册](../../../raw/current/scripts/prefabs/farm_hoe.lua#L256-L258)
- 优先使用：改耐久／单次消耗按[数值表](#注册与构造)与 [[reference/components/farmtiller#关键接口]]、[[reference/components/finiteuses#关键接口]] 处理；从零造同类手持工具查 [[cookbook/equipment/hand]]；查两注册差异与官方名读[注册与构造](#注册与构造)；改黄金锄折算读[典型接入](#同步覆盖黄金锄的锄地消耗与攻击磨损)。
- 重要副作用：锄地成功后经 `BufferedAction:Do` 按 `SetConsumption` 扣耐久；`farmtiller:Till` 会塌陷附近土堆（collapsesoil／breaksoil）、生成 `farm_soil` 并向 doer 推 `tilling`；攻击按 `attackwear` 磨损（普通 1、金锄 1/4）；装备／卸装回调维护持握外观并推送皮肤事件，金锄另在装备时播金音。[扣次链](../../../raw/current/scripts/bufferedaction.lua#L22-L37)、[Till](../../../raw/current/scripts/components/farmtiller.lua#L5-L16)
- 修改边界：公共端只有实体基础、标签与 floater，组件装配在主模拟；本族工厂**无 quagmire 分支**，恒挂 finiteuses／weapon／farmtiller。本页不覆盖同文件第三注册 `fumarole_farm_hoe` 的温度机制、皮肤贴图端（prefabskin／skinprefabs）与 doer 侧动画全流程。[fn 全文](../../../raw/current/scripts/prefabs/farm_hoe.lua#L69-L131)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `farm_hoe` | `MakeFarmHoe("farm_hoe", nil, nil, nil, assets, prefabs)`：data 为 nil 走全默认——bank／build／swap_build `quagmire_hoe`、anim `idle`、sym_name `swap_quagmire_hoe`、`maxfiniteuses=TUNING.FARM_HOE_USES` | 本体；无 common／master postinit，行为全部来自共同初始化。[调用](../../../raw/current/scripts/prefabs/farm_hoe.lua#L256)、[默认值](../../../raw/current/scripts/prefabs/farm_hoe.lua#L45-L53) |
| `golden_farm_hoe` | `MakeFarmHoe("golden_farm_hoe", nil, gold_master_postinit, GOLD_DATA, golden_assets, prefabs)`：共同初始化后追加 master postinit | GOLD_DATA 换 bank／build `goldenhoe`、swap_build／sym_name `swap_goldenhoe`；postinit 把 TILL 消耗与 `weapon.attackwear` 折算为 1/`TUNING.GOLDENTOOLFACTOR`，并监听 `equipped` 播金音。[GOLD_DATA](../../../raw/current/scripts/prefabs/farm_hoe.lua#L138-L144)、[postinit](../../../raw/current/scripts/prefabs/farm_hoe.lua#L152-L157) |

共同初始化（`fn`，[farm_hoe.lua#L69-L131](../../../raw/current/scripts/prefabs/farm_hoe.lua#L69-L131)）：实体基础＋`MakeInventoryPhysics` → bank／build／动画 → 公共端标签 `sharp`、`weapon`（weapon 组件 pristine 优化，[weapon.lua#L36](../../../raw/current/scripts/components/weapon.lua#L36)）→ `MakeInventoryFloatable("med", 0.05, {0.8, 0.4, 0.8}, true, -7, swap_data)` → `scrapbook_subcat = "tool"` → `common_postinit`（两注册均 nil）→ `SetPristine` 端分界（客户端到此返回）→ 主模拟依次 `inspectable`、`inventoryitem`、`equippable`（默认 HANDS，接装备／卸装回调）、`weapon`（伤害 `FARM_HOE_DAMAGE`）、`AddInherentAction(ACTIONS.TILL)`、`finiteuses`（上限＝当前＝`maxfiniteuses`，TILL 消耗 1，耗尽回调推 `toolbroke` 后 `Remove`）、`farmtiller`、`MakeHauntableLaunch` → `master_postinit`。data 还支持 `nofiniteuses`（关耐久）与 `onequipfn`／`onunequipfn`（换装备回调），本文件三注册均未用。

名称核验（简中 .po `msgctxt` 定向核验）：`STRINGS.NAMES.FARM_HOE`=园艺锄（Garden Hoe）、`NAMES.GOLDEN_FARM_HOE`=黄金园艺锄（Splendid Garden Hoe）、动作 `ACTIONS.TILL`=耕地（Till）；配方描述 `RECIPE_DESC.FARM_HOE`=翻耕，为撒播农作物做准备。／`RECIPE_DESC.GOLDEN_FARM_HOE`=优雅地耕地。；检视语 `DESCRIBE.FARM_HOE`=我得把地弄的让种子更舒服一些。[NAMES.FARM_HOE](../../../raw/current/scripts/languages/chinese_s.po#L342539-L342542)、[NAMES.GOLDEN_FARM_HOE](../../../raw/current/scripts/languages/chinese_s.po#L343374-L343377)、[ACTIONS.TILL](../../../raw/current/scripts/languages/chinese_s.po#L2159-L2162)、[RECIPE_DESC.FARM_HOE](../../../raw/current/scripts/languages/chinese_s.po#L356934-L356937)、[RECIPE_DESC.GOLDEN_FARM_HOE](../../../raw/current/scripts/languages/chinese_s.po#L357189-L357192)、[DESCRIBE.FARM_HOE](../../../raw/current/scripts/languages/chinese_s.po#L12384-L12387)

数值与消耗（tuning 键＋实际值）：

| 注册名 | 装备槽 | 耐久上限（点） | 每 TILL 消耗（耐久点） | 武器伤害（生命值） | 每次攻击磨损（耐久点） |
|---|---|---|---|---|---|
| `farm_hoe` | HANDS（equippable 默认） | `TUNING.FARM_HOE_USES`=25 | 1 | `FARM_HOE_DAMAGE`=`wilson_attack`×0.5=17 | 1（weapon 组件 `attackwear` 缺省） |
| `golden_farm_hoe` | HANDS | 25 | 1/`GOLDENTOOLFACTOR`（=4×1）=0.25 | 17 | 0.25（`weapon.attackwear`） |

折算方向：黄金锄改的是**消耗端**（每锄扣 1/4 点），上限仍 25，不是上限 100。源码推导的等效次数（上限÷消耗，未运行测量）：锄地 farm_hoe 25／golden_farm_hoe 100，攻击同。伤害 17 与消耗 0.25 是 `wilson_attack`=34×倍率 1 的源码推导值，非运行测量。[GOLDENTOOLFACTOR](../../../raw/current/scripts/tuning.lua#L95)、[FARM_HOE_USES](../../../raw/current/scripts/tuning.lua#L161)、[FARM_HOE_DAMAGE](../../../raw/current/scripts/tuning.lua#L365)、[基数](../../../raw/current/scripts/tuning.lua#L31-L37)、[equippable 默认 HANDS](../../../raw/current/scripts/components/equippable.lua#L29)

获取途径（配方复核于本快照）：

| 注册名 | 配方与科技 |
|---|---|
| `farm_hoe` | 树枝 2＋燧石 2，`TECH.SCIENCE_ONE`（[recipes.lua#L439](../../../raw/current/scripts/recipes.lua#L439)） |
| `golden_farm_hoe` | 树枝 4＋金块 2，`TECH.SCIENCE_TWO`（[recipes.lua#L440](../../../raw/current/scripts/recipes.lua#L440)） |

同文件第三注册 `fumarole_farm_hoe`（烧灼园艺锄：共用工厂＋FUMAROLE_DATA＋`MakeFumaroleTool` 温度机制，配方树枝 4＋螨腺 2＋硝石 3、`TECH.SCIENCE_TWO`）本页不覆盖，机制见 [[reference/prefabs/fumarole-tools]]。[注册行](../../../raw/current/scripts/prefabs/farm_hoe.lua#L257-L258)、[配方](../../../raw/current/scripts/recipes.lua#L1081)、[NAMES.FUMAROLE_FARM_HOE](../../../raw/current/scripts/languages/chinese_s.po#L343079-L343082)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：farmtiller | 主模拟挂载；`Till(pt, doer)` 检查 `Map:CanTillSoilAtPoint` → `CollapseSoilAtPoint`（附近土堆收 collapsesoil／breaksoil）→ `SpawnPrefab("farm_soil")` → doer 推 `tilling` | 两注册同；组件自身不扣耐久 | [[reference/components/farmtiller#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/farm_hoe.lua#L122)、[Till](../../../raw/current/scripts/components/farmtiller.lua#L5-L16)、[土地资格](../../../raw/current/scripts/components/map.lua#L209-L212) |
| 组件：equippable | `SetOnEquip`／`SetOnUnequip` 接默认持握与皮肤回调；槽位为组件默认 `EQUIPSLOTS.HANDS` | 两注册同；`data.onequipfn` 可覆盖但未被用 | [[reference/components/equippable#关键接口]]；[接线](../../../raw/current/scripts/prefabs/farm_hoe.lua#L105-L107)、[默认槽](../../../raw/current/scripts/components/equippable.lua#L29) |
| 组件：weapon | `SetDamage(TUNING.FARM_HOE_DAMAGE)`；攻击磨损走 `OnAttack` 的 `attackwear`（普通缺省 1、金锄 1/4），先执行回调再扣 finiteuses | 两注册同；无 onattack 回调 | [[reference/components/weapon#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/farm_hoe.lua#L109-L110)、[磨损结算](../../../raw/current/scripts/components/weapon.lua#L105-L121) |
| 组件：finiteuses | 上限＝当前＝maxfiniteuses；`SetConsumption(ACTIONS.TILL, 1)`；耗尽 `OnFiniteUsesFinished`：owner 存在时推 `toolbroke` 再 `inst:Remove()` | 两注册同；金锄 postinit 覆盖 TILL 消耗 0.25；`OnUsedAsItem` 会再乘 efficientuser 倍率 | [[reference/components/finiteuses#关键接口]]；[挂载与配置](../../../raw/current/scripts/prefabs/farm_hoe.lua#L114-L120)、[耗尽回调](../../../raw/current/scripts/prefabs/farm_hoe.lua#L34-L40)、[折算](../../../raw/current/scripts/prefabs/farm_hoe.lua#L153)、[倍率](../../../raw/current/scripts/components/finiteuses.lua#L101-L114) |
| 组件：inspectable／inventoryitem | 主模拟挂载，检视与拾取 | 两注册同 | [[reference/components/inspectable#关键接口]]、[[reference/components/inventoryitem#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/farm_hoe.lua#L102-L103) |
| helper：漂浮与作祟 | 公共端 `MakeInventoryFloatable("med", 0.05, {0.8,0.4,0.8}, true, -7, swap_data)`；主模拟 `MakeHauntableLaunch` | swap_data 用各注册 bank／swap_build／sym_name，漂浮换装与持握一致 | [[reference/components/floater#关键接口]]、[[reference/components/hauntable#按效果选工厂]]；[漂浮](../../../raw/current/scripts/prefabs/farm_hoe.lua#L88)、[作祟](../../../raw/current/scripts/prefabs/farm_hoe.lua#L124)、[helper](../../../raw/current/scripts/standardcomponents.lua#L1772) |
| 动作：`ACTIONS.TILL` | 候选由 POINT 组 farmtiller 收集器提供：右键且 `CanTillSoilAtPoint(pos)` 才插入，经 `PlayerActionPicker:GetPointActions` 进入候选（playercontroller 持物取动作时调用）；执行时 `ACTIONS.TILL.fn` 以 `act.invobject` 优先调 `farmtiller:Till`（无 farmtiller 才试 quagmire_tiller）；玩家 SG 把 TILL 映射到 `till_start`→`till`，till_start 按"手持是否 DIG 工具"选 `till2_pre`／`till_pre`（本锄无 tool 组件，走 `till_pre`） | 两注册同；锄地资格（农田地皮＋净空）与 `farm_soil` 细节归 farmtiller 页 | [[reference/components/farmtiller#关键接口]]；[POINT 收集器](../../../raw/current/scripts/componentactions.lua#L2104-L2108)、[收集入口](../../../raw/current/scripts/components/playeractionpicker.lua#L207-L209)、[调用方](../../../raw/current/scripts/components/playercontroller.lua#L5407-L5408)、[TILL 定义](../../../raw/current/scripts/actions.lua#L535)、[TILL fn](../../../raw/current/scripts/actions.lua#L4712-L4724)、[SG 处理器](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1535)、[till_start](../../../raw/current/scripts/stategraphs/SGwilson.lua#L20331-L20355) |
| API／登记：`AddInherentAction(ACTIONS.TILL)` | 把 TILL 写入 `inst.inherentactions` 并经 actionreplica `net_bytearray` 同步客户端还原 | 登记与同步已核验；锄地候选不经它提供（见上条）；Lua 读取点 `EntityScript:CanDoAction` 在本族中仅被 fumaroletool 的 `OnUsedAsItem` 资格门消费（气孔变体专属） | [登记](../../../raw/current/scripts/entityscript.lua#L419-L433)、[同步与还原](../../../raw/current/scripts/entityscript.lua#L101-L137)、[net 字段](../../../raw/current/scripts/entityscript.lua#L182)、[CanDoAction](../../../raw/current/scripts/entityscript.lua#L1819-L1823)、[fumaroletool 消费](../../../raw/current/scripts/components/fumaroletool.lua#L149-L152) |
| 关联 Prefab：farm_soil | `farmtiller:Till` 成功时在落点生成；`prefabs` 数组仅声明依赖，实际生成在 farmtiller 组件 | 两注册同；土堆行为归 farmtiller 页 | [[reference/components/farmtiller#关键接口]]；[声明](../../../raw/current/scripts/prefabs/farm_hoe.lua#L20-L23)、[生成](../../../raw/current/scripts/components/farmtiller.lua#L8) |
| 资源：动画 | farm_hoe：`quagmire_hoe.zip`（bank／build／swap 同包）；golden_farm_hoe：`goldenhoe.zip`＋`swap_goldenhoe.zip` | 声明即使用：SetBank／SetBuild／持握 swap／floater swap_data | [assets](../../../raw/current/scripts/prefabs/farm_hoe.lua#L1-L10)、[使用](../../../raw/current/scripts/prefabs/farm_hoe.lua#L79-L81) |

### 皮肤接线与金音

默认装备回调（`data.onequipfn` 可覆盖，两注册都用默认）按 `inst:GetSkinBuild()` 分支：有皮肤时向 owner 推送 `equipskinneditem` 并 `OverrideItemSkinSymbol("swap_object", skin_build, sym_name, inst.GUID, swap_build)`；无皮肤时 `OverrideSymbol("swap_object", swap_build, sym_name)`；两分支都 Show `ARM_carry`／Hide `ARM_normal`。默认卸装回调恢复手臂并对皮肤推送 `unequipskinneditem`。[装备](../../../raw/current/scripts/prefabs/farm_hoe.lua#L55-L65)、[卸装](../../../raw/current/scripts/prefabs/farm_hoe.lua#L25-L32)。金锄监听 equippable 装备时在物品上推送的 `equipped` 事件（data.owner 为装备者），在 owner 播 `dontstarve/wilson/equip_item_gold`。[金音](../../../raw/current/scripts/prefabs/farm_hoe.lua#L146-L157)、[事件源](../../../raw/current/scripts/components/equippable.lua#L102)。已注册皮肤：`farm_hoe` 2 种（farm_hoe_invisible／farm_hoe_rustic）、`golden_farm_hoe` 2 种（golden_farmhoe_garden／golden_farmhoe_invisible，注意皮肤名前缀是 `golden_farmhoe`）；invisible 皮肤经 prefabskin init／clear 关 floater 换 bank（`do_bank_swap=false`）。皮肤定义与事件消费端不在本页展开。[farm_hoe 皮肤](../../../raw/current/scripts/prefabs/skinprefabs.lua#L4780-L4800)、[golden 皮肤](../../../raw/current/scripts/prefabs/skinprefabs.lua#L6058-L6078)、[init／clear](../../../raw/current/scripts/prefabskin.lua#L889-L909)

## 典型接入

### 同步覆盖黄金锄的锄地消耗与攻击磨损

- 前提与选择：modmain 用 `AddPrefabPostInit`（[登记](../../../raw/current/scripts/modutil.lua#L593-L599)），回调在实体构造完成时执行（[调用点](../../../raw/current/scripts/mainfunctions.lua#L394-L411)），晚于工厂与 gold_master_postinit。黄金锄的折算是**两条独立配置**（TILL 消耗与 `attackwear`），只改其一不影响另一条。
- 生效与撤销：只影响此后生成的实体；已生成物品按存档的剩余点数恢复。停用模组后新实例恢复 1/`GOLDENTOOLFACTOR`。
- 原版依据：gold_master_postinit 的 `SetConsumption` 与 `attackwear` 就是两条折算的官方用法。[折算](../../../raw/current/scripts/prefabs/farm_hoe.lua#L152-L157)

```lua
-- modmain.lua（模组环境）：黄金园艺锄整体覆盖为每锄／每击 1/8 点（等效 200 次）
AddPrefabPostInit("golden_farm_hoe", function(inst)
    if GLOBAL.TheWorld.ismastersim then
        if inst.components.finiteuses ~= nil then
            inst.components.finiteuses:SetConsumption(GLOBAL.ACTIONS.TILL,
                1 / (GLOBAL.TUNING.GOLDENTOOLFACTOR * 2))
        end
        if inst.components.weapon ~= nil then
            inst.components.weapon.attackwear = 1 / (GLOBAL.TUNING.GOLDENTOOLFACTOR * 2)
        end
    end
end)
```

- 调整与验证：对 `golden_farm_hoe` 直接 `SetConsumption` 是整体覆盖折算值而非叠加；消耗允许小数（`Use` 直接相减），等效次数＝耐久点÷单次消耗；改耐久总量见 [[reference/components/finiteuses#典型接入]]，只改锄地消耗的示例见 [[reference/components/farmtiller#典型接入]]。

## 常见陷阱与限制

- **误区：黄金园艺锄能锄 100 次是上限更大**：上限仍是 `FARM_HOE_USES`=25 点，折算在消耗端（每锄 1/`GOLDENTOOLFACTOR`=0.25 点）；自定义上限时不要把折算重复乘进去。[折算](../../../raw/current/scripts/prefabs/farm_hoe.lua#L152-L157)
- **边界：本族工厂无 quagmire 分支**：与斧／铲工厂不同，`MakeFarmHoe` 不检查 `GetServerGameMode()`，任何模式都恒挂 finiteuses／weapon／farmtiller，gold postinit 直接索引组件不会因模式差异报错；`ACTIONS.TILL.fn` 的 `quagmire_tiller` 分支对锄不生效（farm_hoe 无该组件）。[fn 全文](../../../raw/current/scripts/prefabs/farm_hoe.lua#L69-L131)、[TILL fn](../../../raw/current/scripts/actions.lua#L4712-L4724)
- **边界：耗尽即整个实体 Remove**：`OnFiniteUsesFinished` 先对持有者推 `toolbroke`（有 owner 时）再 `inst:Remove()`，无掉落无残骸；`finiteuses:OnSave` 只在当前数不等于上限时存 `uses`。[回调](../../../raw/current/scripts/prefabs/farm_hoe.lua#L34-L40)、[存档](../../../raw/current/scripts/components/finiteuses.lua#L48-L52)
- **边界：farmtiller 组件不扣耐久**：扣次发生在动作成功后 `BufferedAction:Do` → `OnUsedAsItem` 链，直接调 `farmtiller:Till` 只锄地＋发事件、不耗锄；详见 [[reference/components/farmtiller#常见陷阱与限制]]。[扣次链](../../../raw/current/scripts/bufferedaction.lua#L22-L37)
- **边界：`AddInherentAction` 不是锄地候选的来源**：右键候选由 farmtiller 的 POINT 组件动作提供，移除 inherent action 不影响正常锄地；反之气孔变体的 `OnUsedAsItem` 资格门依赖它，勿把两者混为一谈。[POINT 收集器](../../../raw/current/scripts/componentactions.lua#L2104-L2108)、[CanDoAction](../../../raw/current/scripts/entityscript.lua#L1819-L1823)

## 验证记录

- 源码审阅：2026-11-06，steam-buildid 26335394（`raw/current/BUILD.txt`）。完整读取 `prefabs/farm_hoe.lua`（MakeFarmHoe 工厂、两注册与 fumarole 注册行、GOLD_DATA、gold postinit、默认装备／卸装与皮肤分支、耗尽回调）；核验 farmtiller 组件全文与 map 的土地资格／塌陷事件、equippable 默认 HANDS 与 `equipped` 事件、weapon 磨损结算、finiteuses 消耗／倍率／存档、TILL 定义／fn／POINT 收集器／GetPointActions／SG 处理器与 till_start 状态、bufferedaction 扣次链、entityscript 登记／同步／CanDoAction、standardcomponents 三个 helper、皮肤注册与 prefabskin 接线、两注册配方、tuning 各键、`.po` 名称／动作／配方／检视键。
- 游戏运行：未运行。未覆盖：皮肤实装外观、锄地／作祟／漂浮实际表现、SG till 动画全流程、引擎侧 inherentactions 读取、`fumarole_farm_hoe` 温度机制；17／0.25／100 次等为源码推导值。
- 结构检查：validate_page.py 与 validate_source_links.py 实际运行通过（详见交付说明）。

返回 [[reference/prefabs/tools-field-index|工具与野外实用]]、[[reference/prefabs/index|Prefab 索引]]。
