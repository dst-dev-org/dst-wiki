---
title: "玻璃刀（glasscutter）"
created: "2026-11-05"
updated: "2026-11-08"
type: prefab
summary: "月岛科技手持武器：无工具功能的高伤害玻璃刀，对暗影生物增伤并对纯影目标低磨损，覆盖耐久、伤害、配方与皮肤接线。"
aliases: ["glasscutter", "玻璃刀", "Glass Cutter", "对暗影生物增伤的武器", "玻璃刀耐久"]
tags: [prefab, item, equipment]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/glasscutter.lua", "components/equippable.lua", "components/weapon.lua", "components/finiteuses.lua", "components/damagetypebonus.lua", "componentutil.lua", "standardcomponents.lua", "tuning.lua", "recipes.lua", "constants.lua", "strings.lua", "languages/chinese_s.po", "prefabskins.lua", "screens/redux/scrapbookdata.lua", "prefabs/shadowcreature.lua", "prefabs/stalker.lua", "prefabs/moon_altar.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# 玻璃刀（glasscutter）

## 快速判断

- 职责与适用：`prefabs/glasscutter.lua` 单注册 `glasscutter`，月岛科技的纯手持武器：工厂未挂 tool 组件，不能砍伐／挖矿；HANDS 槽装备，耐久 75，伤害 68，对带 `shadowsubmissive` 标签的目标伤害 ×1.25，对纯影目标每次攻击只磨损 0.5 耐久。[工厂](../../../raw/current/scripts/prefabs/glasscutter.lua#L39-L89)、[注册](../../../raw/current/scripts/prefabs/glasscutter.lua#L91)
- 优先使用：改数值查下方数值表；复刻"按目标切磨损"参考 [[reference/prefabs/axe#典型接入]]，加新标签增伤参考 [[reference/components/damagetypebonus#典型接入]]；耐久机制查 [[reference/components/finiteuses#关键接口]]。
- 重要副作用：耐久耗尽整个实体 Remove（无掉落）；攻击无精神（sanity）副作用——`onattack` 只重设磨损档，源码无 sanity 消费。[onattack](../../../raw/current/scripts/prefabs/glasscutter.lua#L32-L37)、[耗尽回调](../../../raw/current/scripts/prefabs/glasscutter.lua#L76)
- 修改边界：公共端挂标签与 floater，主模拟挂组件；装备槽用 equippable 默认 HANDS，未逐 prefab 设置。皮肤只换持握贴图与音效事件，本页不覆盖皮肤贴图内容。[equippable 默认](../../../raw/current/scripts/components/equippable.lua#L29)

## 注册与构造

| 注册名 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `glasscutter` | `Prefab("glasscutter", fn, assets)`，无变体、无 postinit 入参 | bank／build `glasscutter`，swap `swap_glasscutter`；[注册](../../../raw/current/scripts/prefabs/glasscutter.lua#L91) |

构造顺序（[fn](../../../raw/current/scripts/prefabs/glasscutter.lua#L39-L89)）：公共端 `MakeInventoryPhysics` → anim → 标签 `sharp`、`pointy` → `MakeInventoryFloatable("med", 0.05, {1.21, 0.4, 1.21}, true, -22, {sym_build="swap_glasscutter"})` → `SetPristine`（非主模拟返回）→ 主模拟依次 weapon → damagetypebonus → finiteuses → inspectable → inventoryitem → equippable → `MakeHauntableLaunch`。装备回调维护 ARM_carry／ARM_normal 与 swap_object 覆盖；皮肤 build 非空时推送 `equipskinneditem`／`unequipskinneditem` 事件并换 `OverrideItemSkinSymbol`，否则普通 `OverrideSymbol`。[onequip](../../../raw/current/scripts/prefabs/glasscutter.lua#L7-L21)、[onunequip](../../../raw/current/scripts/prefabs/glasscutter.lua#L23-L30)

名称键经简中 .po `msgctxt` 核验：`STRINGS.NAMES.GLASSCUTTER`＝玻璃刀（Glass Cutter）。[po](../../../raw/current/scripts/languages/chinese_s.po#L343294-L343297)、[strings.lua](../../../raw/current/scripts/strings.lua#L3315)

数值与消耗（tuning 键＋实际值；`wilson_attack`=34×倍率 1，[tuning.lua#L31-L37](../../../raw/current/scripts/tuning.lua#L31-L37)）：

| 项目 | tuning 键 | 实际值 | 消费链 |
|---|---|---|---|
| 耐久上限＝初始 | `GLASSCUTTER.USES` | 75 | finiteuses，仅被攻击磨损扣减 |
| 武器伤害 | `GLASSCUTTER.DAMAGE` | `wilson_attack*2`=68 | `weapon:GetDamage` 基础伤害 |
| 暗影增伤倍率 | `GLASSCUTTER.DAMAGE_VS_SHADOW_BONUS` | 1.25 | 目标带 `shadowsubmissive` 时伤害 ×1.25 |
| 纯影目标磨损 | `GLASSCUTTER.SHADOW_WEAR` | 0.5 | 目标命中 `PURE_SHADOW_TARGET_TAGS` 时每次攻击扣 0.5 |
| 普通目标磨损 | （onattack 字面量） | 1 | 分支 else 值 |

[GLASSCUTTER 表](../../../raw/current/scripts/tuning.lua#L3848-L3854)、[GetDamage 乘算](../../../raw/current/scripts/components/weapon.lua#L94-L103)、[OnAttack 磨损结算](../../../raw/current/scripts/components/weapon.lua#L105-L121)；68／75 与图鉴数据一致（[scrapbookdata](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L605)）。源码推导等效攻击次数：普通目标 75 次、纯影目标 150 次（未运行测量）。

获取途径：配方 木头 1＋月光玻璃 5，`TECH.CELESTIAL_THREE`（={CELESTIAL=3}），站点标签 `celestial_station`。[配方](../../../raw/current/scripts/recipes.lua#L839)、[TECH](../../../raw/current/scripts/constants.lua#L1257)、[站点挂标](../../../raw/current/scripts/prefabs/moon_altar.lua#L584)。全 scripts 目录 grep `glasscutter` 未检索到配方外生成点（其余命中为 debug 命令、皮肤与图鉴数据）。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：weapon | `SetDamage(68)`＋`SetOnAttack(onattack)`；onattack 按目标重设 `attackwear`，`OnAttack` 先调回调再按磨损扣 finiteuses | 主模拟 | [[reference/components/weapon#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/glasscutter.lua#L64-L66) |
| 组件：damagetypebonus | `AddBonus("shadowsubmissive", inst, 1.25)`，由 `GetBonus` 乘入伤害 | 主模拟；标签由影怪 pristine 态／shadowsubmissive 组件添加 | [[reference/components/damagetypebonus#关键接口]]、[[reference/components/shadowsubmissive#快速判断]]；[挂载](../../../raw/current/scripts/prefabs/glasscutter.lua#L68-L69) |
| 组件：finiteuses | 上限＝当前＝75；`SetOnFinished(inst.Remove)`；无 tool 动作消耗 | 主模拟 | [[reference/components/finiteuses#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/glasscutter.lua#L73-L76) |
| 组件：equippable | `SetOnEquip`／`SetOnUnequip` 接线装备显示与皮肤事件；槽位默认 HANDS | 主模拟 | [[reference/components/equippable#关键接口]]；[接线](../../../raw/current/scripts/prefabs/glasscutter.lua#L82-L84) |
| 组件：inspectable／inventoryitem | 检视与拾取 | 主模拟 | [[reference/components/inspectable#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/glasscutter.lua#L78-L80) |
| 组件：floater | 公共端 `MakeInventoryFloatable`，浮水按 swap_data 换贴图 | 公共端（SetPristine 前） | [[reference/components/floater#关键接口]]；[调用](../../../raw/current/scripts/prefabs/glasscutter.lua#L55-L56)、[helper](../../../raw/current/scripts/standardcomponents.lua#L1772-L1791) |
| 组件：hauntable | `MakeHauntableLaunch`：被作祟时弹飞 | 主模拟 | [[reference/components/hauntable#按效果选工厂]]；[调用](../../../raw/current/scripts/prefabs/glasscutter.lua#L86)、[helper](../../../raw/current/scripts/standardcomponents.lua#L1053-L1069) |
| API／数据：`PURE_SHADOW_TARGET_TAGS` | onattack 判定纯影切磨损档：{shadow, shadowminion, shadowchesspiece, stalker, stalkerminion, shadowthrall, shadowboss} | componentutil 全局表，glasscutter 与 moonglassaxe 同用 | [定义](../../../raw/current/scripts/componentutil.lua#L175-L183)、[消费](../../../raw/current/scripts/prefabs/glasscutter.lua#L32-L37) |
| 关联 Prefab：moonglassaxe | 同族月光玻璃斧：CHOP 工具＋同磨损模式，详页不重复展开 | — | [[reference/prefabs/axe#注册与构造]] |
| 关联 Prefab：moon_altar | `celestial_station` 配方站点 | 需组装月坛 | [[reference/prefabs/axe#注册与构造]] |
| 资源：动画包 | glasscutter.zip／swap_glasscutter.zip | 装备时换手持贴图符号 | [声明](../../../raw/current/scripts/prefabs/glasscutter.lua#L1-L5) |

皮肤：已注册 `glasscutter_mystical`（穿刺玻璃刀）与 `glasscutter_terraprisma` 两种，走 prefabskin 的 `glasscutter_init_fn`／`clear_fn`；事件接线见上文装备回调。[皮肤清单](../../../raw/current/scripts/prefabskins.lua#L901-L905)

## 典型接入

### 调整玻璃刀伤害与纯影磨损

- 前提与选择：服务端 `modmain.lua`；磨损档由 onattack 每次攻击现读 `TUNING.GLASSCUTTER.SHADOW_WEAR`，改键即全局生效；伤害是构造时 `SetDamage`，改 `TUNING.GLASSCUTTER.DAMAGE` 只影响之后构造／读档重建的实例。改增伤倍率或加新标签见 [[reference/components/damagetypebonus#典型接入]]。
- 生效与撤销：伤害影响此后构造／读档重建的实例，磨损影响全部实例（攻击时读取）；对纯影的伤害与磨损是两条独立链（damagetypebonus 管伤害、attackwear 管磨损），改其一不影响另一。

```lua
-- modmain.lua：玻璃刀伤害改为 80，对纯影目标磨损降为 0.25
GLOBAL.TUNING.GLASSCUTTER.SHADOW_WEAR = 0.25 -- onattack 每次攻击现读该键
AddPrefabPostInit("glasscutter", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    inst.components.weapon:SetDamage(80)
end)
```

- 调整与验证：伤害单位生命值，磨损单位耐久点/次；皮肤持握贴图不受数值影响。postinit 只在构造（含读档）时执行，已存在实例不会重跑，需重进世界或重建。

## 常见陷阱与限制

- **误区：玻璃刀是挖矿工具**：工厂未挂 tool 组件，无 CHOP／MINE 效率与动作消耗，图鉴分类为 weapon；同族的月光玻璃斧才是 CHOP 工具。[组件清单](../../../raw/current/scripts/prefabs/glasscutter.lua#L64-L86)、[图鉴](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L605)
- **边界：磨损档与增伤是两个不同条件**：磨损看 `PURE_SHADOW_TARGET_TAGS`（7 个标签），增伤看 `shadowsubmissive`；前者范围更宽——远古织影者带 `stalker` 标签（磨损 0.5）但全文件无 shadowsubmissive（无 ×1.25），影怪两者兼有。[标签定义](../../../raw/current/scripts/componentutil.lua#L175-L183)、[stalker 标签](../../../raw/current/scripts/prefabs/stalker.lua#L1376-L1382)、[影怪双标签](../../../raw/current/scripts/prefabs/shadowcreature.lua#L280-L286)
- **边界：`attackwear` 是动态字段**：onattack 在磨损结算前执行，攻击路径下总会先赋值；但 `finiteuses:Use` 只扣 `num or 1`、不读 `attackwear`——绕过 `weapon:OnAttack` 直接调用默认扣 1，要半磨损须显式传 0.5。[结算顺序](../../../raw/current/scripts/components/weapon.lua#L105-L121)
- **边界：耐久耗尽整个实体 Remove**：`SetOnFinished(inst.Remove)`，无掉落残骸；存档按 finiteuses 当前值恢复。[耗尽](../../../raw/current/scripts/prefabs/glasscutter.lua#L76)、[OnSave](../../../raw/current/scripts/components/finiteuses.lua#L48-L52)

## 验证记录

- 源码审阅：2026-11-07（第三轮检视修订：复核 finiteuses 默认扣减、伤害 PostInit 作用域与 attackwear 消费）。早前 2026-11-05，Steam buildid 26335394。核验 prefabs/glasscutter.lua 全文（组件清单、装备回调与皮肤事件、onattack、floater 参数、MakeHauntableLaunch）、equippable 默认 HANDS、weapon GetDamage／OnAttack 结算顺序、finiteuses 上限与耗尽回调、damagetypebonus AddBonus／GetBonus、tuning GLASSCUTTER 各键、配方行与 TECH.CELESTIAL_THREE、`PURE_SHADOW_TARGET_TAGS` 定义与消费、影怪／织影者标签分布、皮肤清单、po 名称键。未核验：皮肤贴图内容与实装外观、目标标签全量分布。
- 游戏运行：未运行。68 伤害与 75／150 次等效数值为源码推导，非运行测量。
- 结构检查：validate_page.py 与 validate_source_links.py --strict 实际执行通过（结果见交付报告）。

返回 [[reference/prefabs/weapons-combat-index|武器与战斗]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-08 build 26474873：本页随版本迁移升版（机械沿用；引用区间未受本次源码行号位移影响），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-07）。
