---
title: 常用采集材料（cutgrass、twigs、cutreeds、log）
created: "2026-11-08"
updated: "2026-11-08"
type: prefab
summary: 四种日常采集获得的基础材料（采下的草、树枝、采下的芦苇、木头）：各自注册与构造、采集来源、堆叠／燃料／可食／修补的实际接线与参数，及配方与燃料方向的消费入口。
aliases: ["cutgrass", "twigs", "cutreeds", "log", "采下的草", "树枝", "采下的芦苇", "木头", "原木", "采集材料", "基础材料"]
tags: [prefab, item]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/cutgrass.lua", "prefabs/twigs.lua", "prefabs/cutreeds.lua", "prefabs/log.lua", "prefabs/grass.lua", "prefabs/sapling.lua", "prefabs/reeds.lua", "prefabs/evergreens.lua", "prefabs/antlion.lua", "recipes.lua", "tuning.lua", "constants.lua", "components/stackable.lua", "components/pickable.lua", "components/inventory.lua", "actions.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 常用采集材料（cutgrass、twigs、cutreeds、log）

## 快速判断

- 职责与适用：四个独立文件分别注册的 `inventoryitem` 材料（无共享工厂，各自 `fn` 走同一套标准 helper 骨架）：割草用的 `cutgrass`、修木用的 `twigs`、芦苇产物 `cutreeds`、伐木产物 `log`。可借鉴：材料类物品的标准组件接线（入栏＋堆叠＋燃料＋可燃＋可食＋修补）与采集产出口径。[cutgrass 注册](../../../raw/current/scripts/prefabs/cutgrass.lua#L63)、[twigs 注册](../../../raw/current/scripts/prefabs/twigs.lua#L82)、[cutreeds 注册](../../../raw/current/scripts/prefabs/cutreeds.lua#L52)、[log 注册](../../../raw/current/scripts/prefabs/log.lua#L60)
- 优先使用：调堆叠／燃料／可食／修补参数查[数值表](#注册与构造)与对应组件页；改采集产出与再生查 [pickable 接线](#典型接入)；做精炼材料（绳／木板）的配方消费见 [[reference/prefabs/basic-refined-materials]]。
- 重要副作用：全部主模拟挂 `fuel`＋`MakeSmallBurnable`＋`MakeSmallPropagator`＋可食 `edible`；`twigs` 另挂 `oceanfishingtackle`（树枝浮子）并留 quagmire 模式 postinit 钩子。[twigs 浮子](../../../raw/current/scripts/prefabs/twigs.lua#L70-L71)、[quagmire 钩子](../../../raw/current/scripts/prefabs/twigs.lua#L75-L77)
- 修改边界：公共端只建实体＋标签＋浮水，组件都在主模拟（`SetPristine` 分支后）挂载；本页不覆盖移植形态 `dug_grass`／`dug_sapling`、活木 `livinglog` 与芦苇代码 `papyrus`（精炼页）。

## 注册与构造

四个注册各自独立成文件，无共享工厂；骨架一致：`CreateEntity` → Transform／AnimState／Network → `MakeInventoryPhysics` → bank／build＝注册名、`PlayAnimation("idle")` → `pickupsound` → 差异标签 → `MakeInventoryFloatable` → `SetPristine`（非主模拟返回）→ 主模拟逐项挂组件。差异只在组件取舍与参数，见下表与[数值表](#数值与参数)。

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `cutgrass` | 单文件单注册 `Prefab("cutgrass", fn, assets)`；fn 主模拟依次挂 inventoryitem、stackable、edible、inspectable、tradable、fuel、burnable/propagator、hauntable、repairer、snowmandecor | 公共端标签 `cattoy`／`renewable`／`cutgrass`；浮水 `("med", 0.05, 0.68)`。 [注册](../../../raw/current/scripts/prefabs/cutgrass.lua#L63)、[组件段](../../../raw/current/scripts/prefabs/cutgrass.lua#L33-L58) |
| `twigs` | 同骨架；多挂 tradable、oceanfishingtackle（`BOBBER_TWIG` 抛投＋依赖 `oceanfishingbobber_twig_projectile`）与 quagmire postinit 钩子 | 标签多 `oceanfishing_bobber`／`twigs`；edible 二类 `FOODTYPE.WOOD`；repairer 带 `boatrepairsound`；quagmire 模式注入 `event_server_data("quagmire", "prefabs/twigs").master_postinit`。 [浮子接线](../../../raw/current/scripts/prefabs/twigs.lua#L70-L71)、[quagmire](../../../raw/current/scripts/prefabs/twigs.lua#L75-L77) |
| `cutreeds` | 同骨架；只有 inventoryitem、stackable、inspectable、edible、fuel、burnable/propagator、hauntable，**无** tradable／repairer／snowmandecor／cattoy | 公共端不加任何标签（对比 cutgrass/twigs 的 cattoy 组）；`inventoryitem` 在组件段末尾才挂（L47），挂载顺序不影响行为。 [组件段](../../../raw/current/scripts/prefabs/cutreeds.lua#L29-L47) |
| `log` | 同骨架；挂 tradable、edible、fuel、burnable/propagator、hauntable、inspectable、inventoryitem、stackable、repairer；stackable 不设 maxsize | 标签仅 `log`（L19）；edible 为 `FOODTYPE.WOOD` 且治疗／饥饿均为 0；浮水 `("med", 0.1, 0.75)`。 [组件段](../../../raw/current/scripts/prefabs/log.lua#L31-L55) |

### 获取途径与入栏路径

| 注册名 | 采集／掉落来源（已核验） |
|---|---|
| `cutgrass` | 草丛 `grass` 的 `pickable:SetUp("cutgrass", TUNING.GRASS_REGROW_TIME)`，拾取产 `pickable.product`；移植挖起时枯萎掉 `cutgrass`、否则掉 `dug_grass`。 [SetUp](../../../raw/current/scripts/prefabs/grass.lua#L248)、[挖起分支](../../../raw/current/scripts/prefabs/grass.lua#L76-L83) |
| `twigs` | 树枝丛 `sapling` 的 `pickable:SetUp("twigs", TUNING.SAPLING_REGROW_TIME)`；挖起掉 `twigs`（枯萎）或 `dug_sapling(_moon)`。 [SetUp](../../../raw/current/scripts/prefabs/sapling.lua#L141)、[挖起分支](../../../raw/current/scripts/prefabs/sapling.lua#L39-L43) |
| `cutreeds` | 芦苇 `reeds` 的 `pickable:SetUp("cutreeds", TUNING.REEDS_REGROW_TIME)`。 [SetUp](../../../raw/current/scripts/prefabs/reeds.lua#L61) |
| `log` | 伐木掉落：常青树各尺寸 loot 表 `normal_loot`／`short_loot`／`tall_loot` 含 `log`（枯树变种同理）；本页只核验 evergreens 一族作为入口，其余树种与死亡掉落未逐一枚举。 [loot 表](../../../raw/current/scripts/prefabs/evergreens.lua#L66-L96) |

玩家获得分两条入栏入口，不能混写：**采集（PICK）**——对草丛／树苗丛／芦苇执行 `ACTIONS.PICK.fn` → `pickable:Pick` → `SpawnProductLoot`，玩家身上有 inventory 时采集直接调用 `inventory:GiveItem(loot, nil, pt)` 尝试入栏，物品栏无法接收（如已满）时物品仍可能掉地；**地面拾取（PICKUP）**——地上的物品（含伐木 loot 表掉落）走 `ACTIONS.PICKUP.fn`（校验 `inventoryitem.canbepickedup` 等）→ `Inventory:GiveItem(act.target)` 入栏并按 stackable 并堆。[PICK 动作](../../../raw/current/scripts/actions.lua#L1933-L1945)、[SpawnProductLoot 尝试入栏分支](../../../raw/current/scripts/components/pickable.lua#L493-L515)、[GiveItem 满仓掉地分支](../../../raw/current/scripts/components/inventory.lua#L1167-L1215)、[PICKUP 拾取](../../../raw/current/scripts/actions.lua#L960-L963)、[GiveItem](../../../raw/current/scripts/components/inventory.lua#L1029)

### 数值与参数

tuning 基准：`seg_time`=30（[tuning.lua#L19](../../../raw/current/scripts/tuning.lua#L19)）、`calories_per_day`=75（[L41](../../../raw/current/scripts/tuning.lua#L41)）、堆叠 `STACK_SIZE_SMALLITEM`=40／`STACK_SIZE_MEDITEM`=20（[L81-L83](../../../raw/current/scripts/tuning.lua#L81-L83)）。

| 注册名 | 堆叠上限 | 燃料值（秒） | 燃烧时长（秒） | 可食（foodtype） | 治疗／饥饿 | 修补材料／回复值 |
|---|---|---|---|---|---|---|
| `cutgrass` | 40 | `SMALL_FUEL`=30×.5=15 | `SMALL_BURNTIME`=7.5 | ROUGHAGE | `HEALING_TINY`=1／`CALORIES_TINY`/2=4.6875 | HAY／13 |
| `twigs` | 40 | 15 | 7.5 | ROUGHAGE＋secondary WOOD | `HEALING_TINY`/2=0.5／`CALORIES_TINY`=9.375 | WOOD／13 |
| `cutreeds` | 40 | 15 | 7.5 | ROUGHAGE | 1／4.6875 | 无 repairer |
| `log` | 未设→组件默认 20 | `MED_FUEL`=45 | `MED_BURNTIME`=15 | WOOD | 0／0 | WOOD／`REPAIR_LOGS_HEALTH`=50/4=12.5 |

燃料值与燃烧时长是两条独立数值：前者是喂给 `fueled` 对象的秒数（`fuel.fuelvalue`），后者是自身 `MakeSmallBurnable` 燃尽时长。[fuel 段](../../../raw/current/scripts/prefabs/cutgrass.lua#L46-L49)、[SMALL_FUEL 等](../../../raw/current/scripts/tuning.lua#L1674-L1683)、修补值 [L2104-L2120](../../../raw/current/scripts/tuning.lua#L2104-L2120)、堆叠默认 [stackable.lua#L28](../../../raw/current/scripts/components/stackable.lua#L28)、MATERIALS 表 [constants.lua#L1940-L1950](../../../raw/current/scripts/constants.lua#L1940-L1950)、FOODTYPE [L2026-L2041](../../../raw/current/scripts/constants.lua#L2026-L2041)

名称键经简中 .po `msgctxt` 核验：`NAMES.CUTGRASS`=采下的草（Cut Grass）、`NAMES.TWIGS`=树枝（Twigs）、`NAMES.CUTREEDS`=采下的芦苇（Cut Reeds）、`NAMES.LOG`=木头（Log，无"原木"官方名）。[CUTGRASS](../../../raw/current/scripts/languages/chinese_s.po#L341945-L341947)、[TWIGS](../../../raw/current/scripts/languages/chinese_s.po#L351125-L351127)、[CUTREEDS](../../../raw/current/scripts/languages/chinese_s.po#L341960-L341962)、[LOG](../../../raw/current/scripts/languages/chinese_s.po#L344785-L344787)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：inventoryitem | 拾取入栏（采集走 PICK 尝试 GiveItem 入栏、地面物品走 PICKUP，两入口见[获取途径](#获取途径与入栏路径)）；`pickupsound` 双端设置，时序因注册而异：cutgrass／cutreeds／log 在 `SetPristine` 前，twigs 在 `SetPristine` 后、mastersim 分支前（[twigs.lua#L32-L34](../../../raw/current/scripts/prefabs/twigs.lua#L32-L34)） | 四注册；主模拟挂载组件 | [[reference/components/inventoryitem#关键接口]]；[PICK](../../../raw/current/scripts/actions.lua#L1933-L1945)、[PICKUP](../../../raw/current/scripts/actions.lua#L960-L963) |
| 组件：stackable | maxsize 40（三注册显式设）／20（log 用组件默认）；replica 同步上限 | 四注册；主模拟 | [[reference/components/stackable#关键接口]]；[cutgrass](../../../raw/current/scripts/prefabs/cutgrass.lua#L35-L36)、[默认](../../../raw/current/scripts/components/stackable.lua#L28) |
| 组件：edible | ROUGHAGE（草／枝／苇，枝加 secondary WOOD）与 WOOD（log）可食入口 | 主模拟；数值见[数值表](#数值与参数) | [[reference/components/edible#关键接口]]；[cutgrass](../../../raw/current/scripts/prefabs/cutgrass.lua#L38-L41)、[FOODTYPE](../../../raw/current/scripts/constants.lua#L2026-L2041) |
| 组件：fuel | 作为燃料加入 `fueled` 对象（firepit 等），fuelvalue 见数值表 | 四注册；主模拟 | [[reference/components/fuel#关键接口]]；[cutgrass](../../../raw/current/scripts/prefabs/cutgrass.lua#L46-L47) |
| 组件：burnable／propagator | `MakeSmallBurnable`＋`MakeSmallPropagator`：可被点燃、蔓延与燃尽 | 四注册；主模拟 | [[reference/components/burnable#关键接口]]；[helper 调用](../../../raw/current/scripts/prefabs/log.lua#L41-L42) |
| 组件：hauntable | `MakeHauntableLaunchAndIgnite`：作祟弹飞并点燃 | 四注册；主模拟 | [[reference/components/hauntable#按效果选工厂]]；[cutgrass](../../../raw/current/scripts/prefabs/cutgrass.lua#L52) |
| 组件：repairer | 修补 hay／wood 类可修物（墙、船等），材料与回复值见数值表 | cutgrass／twigs／log（cutreeds 无） | [[reference/components/repairer#关键接口]]；[twigs boatrepairsound](../../../raw/current/scripts/prefabs/twigs.lua#L65-L68) |
| 组件：oceanfishingtackle | `twigs` 当海钓浮子：`SetCastingData(TUNING.OCEANFISHING_TACKLE.BOBBER_TWIG, "oceanfishingbobber_twig_projectile")` | 仅 twigs；公共端带 `oceanfishing_bobber` 标签 | Wiki 待建；[接线](../../../raw/current/scripts/prefabs/twigs.lua#L70-L71)、[BOBBER_TWIG](../../../raw/current/scripts/tuning.lua#L270)、[依赖声明](../../../raw/current/scripts/prefabs/twigs.lua#L7-L10) |
| 组件：snowmandecor | 冬季雪人装饰部件 | cutgrass／twigs（cutreeds／log 无） | Wiki 待建；[cutgrass](../../../raw/current/scripts/prefabs/cutgrass.lua#L58)、[twigs](../../../raw/current/scripts/prefabs/twigs.lua#L73) |
| 组件：tradable | 挂 tradable（cutgrass／twigs／log；goldvalue 均未设→猪王不收，见陷阱） | 主模拟 | [[reference/components/tradable#关键接口]]；[猪王 AcceptTest](../../../raw/current/scripts/prefabs/pigking.lua#L604-L605) |
| API／数据：pickable 接线 | 植物侧 `SetUp(产品, 再生时间)` 定义采集产物与再生 | 服务器；grass／sapling／reeds | [[reference/components/pickable#关键接口]]；[grass](../../../raw/current/scripts/prefabs/grass.lua#L248)、[sapling](../../../raw/current/scripts/prefabs/sapling.lua#L141)、[reeds](../../../raw/current/scripts/prefabs/reeds.lua#L61) |
| API／数据：lootdropper loot 表 | 伐木按树尺寸 SpawnLootPrefab `log`；配方侧为最大消费者群（绳=草3、木板=木4） | 服务器 | [[reference/components/lootdropper#关键接口]]；[evergreens](../../../raw/current/scripts/prefabs/evergreens.lua#L66-L96)、[rope 配方](../../../raw/current/scripts/recipes.lua#L457)、[boards 配方](../../../raw/current/scripts/recipes.lua#L459) |
| 关联 Prefab：grass／sapling／reeds | 采集来源植物本体（含枯萎／月岛变种分支） | 世界实体，Wiki 待建 | [grass.lua](../../../raw/current/scripts/prefabs/grass.lua#L76-L83)、[sapling.lua](../../../raw/current/scripts/prefabs/sapling.lua#L39-L43)、[reeds.lua](../../../raw/current/scripts/prefabs/reeds.lua#L61) |
| 关联 Prefab：oceanfishingbobber_twig_projectile | twigs 浮子抛投弹道实体，声明于 twigs 的 prefabs 依赖 | 仅 twigs 抛投时生成 | [声明](../../../raw/current/scripts/prefabs/twigs.lua#L7-L10) |
| 资源：anim 包 | cutgrass.zip／twigs.zip（含 sound/common.fsb）／cutreeds.zip／log.zip，bank／build 同名 | 公共端声明 | [cutgrass assets](../../../raw/current/scripts/prefabs/cutgrass.lua#L1-L4)、[twigs assets](../../../raw/current/scripts/prefabs/twigs.lua#L1-L5) |

## 典型接入

### 调整采集材料的堆叠／燃料／可食参数

- 前提与选择：服务端 `modmain.lua`；沿用原注册与组件，postinit 只改数值，不重建实体。改采集产出或再生应动植物侧 `pickable:SetUp`，不改本页 prefab。
- 生效与撤销：`AddPrefabPostInit` 在该 prefab 每次构造（含读档重建）后执行；停用模组即恢复默认接线。
- 原版依据：stackable maxsize 经 `_.maxsize` 与 replica 同步；主模拟才有 `components`，客户端读档端须判空。[maxsize](../../../raw/current/scripts/components/stackable.lua#L11-L20)

```lua
-- modmain.lua：木头堆叠上限对齐小件材料（20 → 40）
AddPrefabPostInit("log", function(inst)
    if GLOBAL.TheWorld.ismastersim and inst.components.stackable ~= nil then
        inst.components.stackable.maxsize = TUNING.STACK_SIZE_SMALLITEM
    end
end)
```

- 调整与验证：单位为个／燃料为秒（`fuelvalue`）；改 maxsize 影响新构造实例，已存在堆叠在下次并堆时按新上限处理。`TUNING` 在 modmain 白名单直挂，`TheWorld` 须经 `GLOBAL` 取。

## 常见陷阱与限制

- **边界：四个注册的组件取舍不一致，不能按 cutgrass 类推**：`cutreeds` 无 tradable／repairer／snowmandecor 且不加标签；`log` 无 cattoy/renewable、堆叠走默认 20。复制参数前逐注册核对源文件。[cutreeds 组件段](../../../raw/current/scripts/prefabs/cutreeds.lua#L29-L47)、[log 组件段](../../../raw/current/scripts/prefabs/log.lua#L31-L55)
- **边界：挂 `tradable` 不等于猪王收货**：猪王 `AcceptTest` 只收 `goldvalue > 0`（或节日糖果／pig_token），这三种材料都未设 goldvalue，交付会被拒绝。`tradable` 在此承担其他交易系统：`rocktribute` 的消费者是蚁狮（AcceptTest 要求 `rocktribute > 0`，进贡结算按值折算怒气平息时长），本页四注册均未设。[猪王 AcceptTest](../../../raw/current/scripts/prefabs/pigking.lua#L604-L605)、[蚁狮 AcceptTest](../../../raw/current/scripts/prefabs/antlion.lua#L116-L119)、[蚁狮进贡结算](../../../raw/current/scripts/prefabs/antlion.lua#L141)
- **边界：燃料值与燃烧时长是两条独立数值**：`fuel.fuelvalue`（喂别的对象）与 `MakeSmallBurnable` 时长（自身燃尽）不同源，改其一不动另一条；数值见[数值表](#数值与参数)。[cutgrass L46-L49](../../../raw/current/scripts/prefabs/cutgrass.lua#L46-L49)
- **边界：quagmire 模式 twigs 有额外 postinit**：`event_server_data("quagmire", "prefabs/twigs").master_postinit` 注入活动模式行为，postinit 改参时勿覆盖该钩子后的状态。[twigs L75-L77](../../../raw/current/scripts/prefabs/twigs.lua#L75-L77)
- **边界：`log` 显示名是"木头"**：简中 PO 无"原木"官方名，文本与别名称需对齐 `NAMES.LOG`。[LOG](../../../raw/current/scripts/languages/chinese_s.po#L344785-L344787)

## 验证记录

- 源码审阅：2026-11-08，Steam buildid 26335394。核验四份 prefab 文件全文（注册名、公共端与主模拟组件接线、全部 tuning 键实际取值）、采集来源（grass／sapling／reeds 的 pickable SetUp 与挖起分支、evergreens loot 表）、拾取入栏链（PICKUP.fn→GiveItem）、stackable 默认上限、猪王 AcceptTest 对 tradable 的实际语义、recipes.lua 的 rope／boards 配方行、简中名称键。未覆盖：cutgrass/twigs 的 cattoy 标签完整消费者、snowmandecor 与 quagmire 活动实现细节、其余树种伐木掉落枚举。
- 游戏运行：未运行。治疗／饥饿／燃料等数值为 tuning 源码推导（如 `CALORIES_TINY/2`=4.6875），非运行测量。
- 结构检查：validate_page.py 与 validate_source_links.py 通过。
- 修订：2026-11-08，主代理独立检视回源码复核后修订两处：①"龟王 tribute 用 `rocktribute`"更正为蚁狮（`prefabs/antlion.lua#L116-L119` AcceptTest 要求 `rocktribute>0`、`#L141` 结算按值 ×`ANTLION_TRIBUTE_TO_RAGE_TIME` 折算怒气平息），并补记 antlion.lua 核验入 sources；②`pickupsound`"切网络前"表述按实际时序改写——twigs 在 `SetPristine` 后、mastersim 分支前（`prefabs/twigs.lua#L32-L34`），其余三注册在 SetPristine 前，删除错误的"切网络前"括注。
- 外部复检修订：2026-11-08，按外部复检报告（docs/reports/w05/prefab-inventory-review-b01-20261108.md R12，主代理回源码验证成立）拆分入栏入口："玩家拾取统一走 PICKUP"改为两条——采集走 `ACTIONS.PICK.fn`（actions.lua#L1933-L1945）→ `pickable:Pick` → `SpawnProductLoot` 玩家直入栏分支 `inventory:GiveItem(loot, nil, pt)`（pickable.lua#L492-L515），地面物品（含伐木掉落）才走 `ACTIONS.PICKUP.fn` → `Inventory:GiveItem(act.target)`（actions.lua#L960-L963，替换旧锚 L894-L912），nav 表同步；补记 components/pickable.lua 入 sources。
- 复检二次修订：2026-11-08（修订后复检 R12 余项）："产物不落地、直接入栏"收窄为"采集直接调用 GiveItem 尝试入栏，物品栏无法接收（如已满）时仍可能掉地"（GiveItem 满仓掉地分支 inventory.lua#L1167-L1215），nav 表与索引行同步改"尝试入栏"。

返回 [[reference/prefabs/index|Prefab 索引]]。
