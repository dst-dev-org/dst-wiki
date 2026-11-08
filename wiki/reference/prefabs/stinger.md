---
title: "针刺（stinger）"
created: "2026-11-10"
updated: "2026-11-10"
type: prefab
summary: "单注册物品针刺：蜜蜂与杀人蜂共用一次随机抽取的 1:5 权重、蜂王必掉 1 个、嗡嗡蜜蜂 1%（无幸运修正的基准），加上睡眠吹箭与弹弓弹药两条主消费、40 叠与 SetIgnoreMovingFast 的自动并堆、可燃烧并掉同叠数灰烬。"
aliases: ["stinger", "针刺", "蜂刺", "毒刺", "吹箭材料"]
tags: [prefab, item]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/stinger.lua", "prefabs/bee.lua", "prefabs/beeguard.lua", "prefabs/beequeen.lua", "prefabs/slingshotammo.lua", "components/stackable.lua", "components/selfstacker.lua", "components/lootdropper.lua", "standardcomponents.lua", "containers.lua", "cooking.lua", "preparedfoods.lua", "recipes.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# 针刺（stinger）

## 快速判断

- 职责与适用：针刺是**唯一一个注册**的可堆叠材料（简中实显 **针刺**，40 叠），本体只挂浮空、拾取、自动并堆、可燃烧、热量扩散与作祟这几类组件，没有食用、鲜度、燃料或交易组件。[构造](../../../raw/current/scripts/prefabs/stinger.lua#L15-L45)
- 优先使用：查蜂类怎么掉看[掉落来源](#掉落来源)；查睡眠吹箭、弹弓弹药与其余配方看[用途与消耗](#用途与消耗)；查燃烧、并堆与标签看[注册与构造](#注册与构造)。
- 重要副作用：蜜蜂与杀人蜂共用**同一次随机抽取**（权重 honey 1、stinger 5），所以不是"每只蜂必掉"；蜂王那条是 1.00 的必得项。[随机表](../../../raw/current/scripts/prefabs/bee.lua#L194-L197)、[蜂王表](../../../raw/current/scripts/prefabs/beequeen.lua#L24-L43)
- 修改边界：蜂类与蜂王本体行为不展开；吹箭与弹弓武器本体归 [[reference/prefabs/ranged-weapons]]（S5）。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `stinger` | `fn()`：实体 → Transform／AnimState／Network → `MakeInventoryPhysics` → bank／build `stinger`、播 `idle` → 浮空 `("small", 0.1, 0.7)` → `selfstacker` 标签 → `SetPristine()` → 主模拟挂 `stackable`（40）／`inspectable`／`inventoryitem`／`selfstacker` → `MakeSmallBurnable(SMALL_BURNTIME)` → `MakeSmallPropagator` → `MakeHauntableLaunch` | 唯一注册，无变体；`assets` 只有 `anim/stinger.zip`。[工厂](../../../raw/current/scripts/prefabs/stinger.lua#L6-L46) |

- **堆叠显式 40**：`stackable.maxsize = TUNING.STACK_SIZE_SMALLITEM`，不落组件默认的 20（S7）。[赋值](../../../raw/current/scripts/prefabs/stinger.lua#L30-L31)、[常量](../../../raw/current/scripts/tuning.lua#L81-L83)
- **并堆忽略速度**：`selfstacker:SetIgnoreMovingFast(true)` 让它在被抛飞／滑动时也能并堆；犬牙没有这一句，两者行为不同。[设置](../../../raw/current/scripts/prefabs/stinger.lua#L37-L38)、[判定](../../../raw/current/scripts/components/selfstacker.lua#L18-L26)
- **可燃烧且会传热**：`MakeSmallBurnable` 把燃烧时长设成 `SMALL_BURNTIME`＝30×0.25＝**7.5 秒**；`MakeSmallPropagator` 让整叠接受热量并在闪点被点燃。[燃烧](../../../raw/current/scripts/standardcomponents.lua#L164-L174)、[常量](../../../raw/current/scripts/tuning.lua#L1680-L1683)、[传热](../../../raw/current/scripts/standardcomponents.lua#L200-L203)
- **烧完变灰烬，不是消失**：烧毁回调生成 1 个 `ash`，并把灰烬叠数设成与原叠数相同（受灰烬自身上限约束）。[烧毁回调](../../../raw/current/scripts/standardcomponents.lua#L18-L44)
- **作祟只抛飞**：`MakeHauntableLaunch` 无其它分支。[helper](../../../raw/current/scripts/standardcomponents.lua#L1053-L1069)

### 名称核验

`NAMES.STINGER`＝**针刺**，是简中实显名（S1）；没有改名调用。[po 名称键](../../../raw/current/scripts/languages/chinese_s.po#L350059-L350062)

## 掉落来源

| 宿主 | 产出 | 机制与条件 | 证据 |
|---|---|---|---|
| 蜜蜂 `bee` | 5/6 概率 1 个 `stinger`，1/6 概率 1 个 `honey` | `commonfn` 里 `AddRandomLoot("honey", 1)`＋`("stinger", 5)`、`numrandomloot = 1`：先按 `chancerandomloot`（未设＝1）决定抽一次，再按权重抽 1 件，共 6 份权重 | [接线](../../../raw/current/scripts/prefabs/bee.lua#L194-L197)、[权重抽取](../../../raw/current/scripts/components/lootdropper.lua#L99-L110) |
| 杀人蜂 `killerbee` | 同上（同一张随机表） | 两个注册都走 `commonfn`，掉落段不在各自的专属函数里 | [共用工厂](../../../raw/current/scripts/prefabs/bee.lua#L139-L245)、[两个注册](../../../raw/current/scripts/prefabs/bee.lua#L318-L319) |
| 蜂王 `beequeen` | 1 条 1.00，必得 1 个 | `SetChanceLootTable('beequeen')` 的独立掷点；1.00 走必中分支，不受幸运影响 | [表](../../../raw/current/scripts/prefabs/beequeen.lua#L24-L43)、[接线](../../../raw/current/scripts/prefabs/beequeen.lua#L412) |
| 嗡嗡蜜蜂 `beeguard` | 1% 概率 1 个（无幸运修正的基准） | `AddChanceLoot("stinger", 0.01)`，单条独立判定，先过 `GetChance`（击杀者幸运） | [接线](../../../raw/current/scripts/prefabs/beeguard.lua#L513) |
| 风滚草 `tumbleweed` | 权重 1 | 生成时抽 3 次（有放回），被拾取时把已抽好的 3 件全部落地；单项约 1%／抽（推导，分母随存档解锁态变化） | [权重](../../../raw/current/scripts/prefabs/tumbleweed.lua#L179)、[生成时抽取](../../../raw/current/scripts/prefabs/tumbleweed.lua#L494)、[3 抽](../../../raw/current/scripts/prefabs/tumbleweed.lua#L213-L245)、[拾取落地](../../../raw/current/scripts/prefabs/tumbleweed.lua#L121-L125)、[存档只存 burnt](../../../raw/current/scripts/prefabs/tumbleweed.lua#L324-L326) |

- **权重抽取是"有放回"的**：`PickRandomLoot` 按 `math.random() × 总权重` 逐项扣减，抽中的条目不从表里移除；本页 `numrandomloot = 1`，所以每只蜂只结算一次。[抽取](../../../raw/current/scripts/components/lootdropper.lua#L92-L112)
- **5/6 是推导值**：`5 ÷ (1+5) = 0.8333`，未做游戏内掉率采样。

## 用途与消耗

| 去向 | 消耗 | 条件与产物 | 证据 |
|---|---|---|---|
| 睡眠吹箭 `blowdart_sleep` | 芦苇 2＋针刺 1＋黑鸟羽 1 | `TECH.SCIENCE_TWO`；命中效果归 [[reference/prefabs/ranged-weapons]] | [配方](../../../raw/current/scripts/recipes.lua#L488) |
| 弹弓碎裂弹 `slingshotammo_stinger` | 针刺 2 → 20 发 | 沃尔特技能 `walter_ammo_shattershots`；装 `walter_ammo_efficiency` 时 ×1.5＝30 发 | [配方](../../../raw/current/scripts/recipes.lua#L221)、[数量覆盖](../../../raw/current/scripts/recipes.lua#L199-L204) |
| 养蜂书 `book_bees` | 纸莎草 2＋针刺 8＋蜂蜜 4 | `TECH.BOOKCRAFT_ONE`、书匠 | [配方](../../../raw/current/scripts/recipes.lua#L277) |
| 荆棘护甲／荆棘陷阱 | 活木 2＋针刺 4／活木 1＋针刺 1 | 沃姆伍德（`plantkin`） | [护甲](../../../raw/current/scripts/recipes.lua#L339)、[陷阱](../../../raw/current/scripts/recipes.lua#L340) |
| 生命注射器 `lifeinjector` | 腐烂食物 8＋硝石 2＋针刺 1 | `TECH.SCIENCE_TWO` | [配方](../../../raw/current/scripts/recipes.lua#L421) |
| 幽灵药水·攻击 | 针刺 1＋幽灵花 3 | 温蒂药水师 | [配方](../../../raw/current/scripts/recipes.lua#L155) |
| 船补丁／码头套装 | 木头 2＋针刺 1／木板 4＋石砖 1＋针刺 2＋棕榈果鳞 1 | `TECH.NONE`／`TECH.LOST`（一次 4 个） | [补丁](../../../raw/current/scripts/recipes.lua#L613)、[码头](../../../raw/current/scripts/recipes.lua#L636) |
| 隐士·昏睡鱼饵 | 饼干切割壳 1＋针刺 1 | `TECH.LOST` | [配方](../../../raw/current/scripts/recipes.lua#L663) |
| 缝纫机套装／冬宴塔马利 | 针刺 1／针刺 2 | 皮弗娄牛祭品、冬宴烹饪 | [缝纫机](../../../raw/current/scripts/recipes.lua#L1156)、[塔马利](../../../raw/current/scripts/recipes.lua#L1446) |

- **无食用组件与无锅料登记是两件事**：普通锅按注册名 `cooking.IsCookingIngredient(item.prefab)` 收料、不查 `edible`；本快照 `cooking.lua` 没有 `stinger` 条目，所以不能下锅——这与它没有 `edible`、不能直接吃互不因果（蜜脾同样没有 `edible`，却已登记为甜味剂）。同名菜 `vegstinger`（蔬菜鸡尾酒）是独立注册的料理，配方不消耗针刺。[无 edible](../../../raw/current/scripts/prefabs/stinger.lua#L30-L35)、[蜜脾反例](../../../raw/current/scripts/cooking.lua#L97)、[按名查表](../../../raw/current/scripts/cooking.lua#L185-L187)、[锅的过滤](../../../raw/current/scripts/containers.lua#L373-L375)、[菜谱](../../../raw/current/scripts/preparedfoods.lua#L522-L524)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`stackable` | 显式 40 | 主模拟 | [[reference/components/stackable]]；[赋值](../../../raw/current/scripts/prefabs/stinger.lua#L30-L31) |
| 组件：`selfstacker` | 自动并堆，忽略移动速度 | 主模拟；实体唤醒时触发 | [[reference/components/selfstacker]]；[设置](../../../raw/current/scripts/prefabs/stinger.lua#L37-L38) |
| 组件：`burnable` | 7.5 秒烧完，掉同叠数灰烬 | 主模拟；点燃与烧毁回调都在标准库 | [[reference/components/burnable]]；[挂载](../../../raw/current/scripts/prefabs/stinger.lua#L40) |
| 组件：`propagator` | 接受邻近热量并可被闪点点燃 | 主模拟 | [[reference/components/propagator]]；[挂载](../../../raw/current/scripts/prefabs/stinger.lua#L41) |
| 组件：`hauntable` | 作祟只抛飞 | 主模拟 | [[reference/components/hauntable]]；[挂载](../../../raw/current/scripts/prefabs/stinger.lua#L43) |
| 宿主：`bee`／`killerbee` | 共用一次 1:5 权重抽取 | 服务端死亡 `DropLoot` | [掉落段](../../../raw/current/scripts/prefabs/bee.lua#L194-L197) |
| 宿主：`beequeen`／`beeguard` | 必得 1 个（1.00 必中）／1%（无幸运基准） | 服务端死亡 | [蜂王](../../../raw/current/scripts/prefabs/beequeen.lua#L39)、[嗡嗡蜜蜂](../../../raw/current/scripts/prefabs/beeguard.lua#L513) |
| 关联 Prefab：`slingshotammo_stinger` | 弹药产物，本体是独立注册 | 沃尔特技能 | [[reference/prefabs/ranged-weapons]]；[弹药名与符号](../../../raw/current/scripts/prefabs/slingshotammo.lua#L1550-L1551) |
| 关联 Prefab：`blowdart_sleep` | 配方产物 | `TECH.SCIENCE_TWO` | [[reference/prefabs/ranged-weapons]]；[配方](../../../raw/current/scripts/recipes.lua#L488) |
| 资源：`anim/stinger.zip` | bank／build `stinger` | 原版资源，不随页分发 | [资产](../../../raw/current/scripts/prefabs/stinger.lua#L1-L4) |

## 常见陷阱与限制

- **误区：每只蜜蜂必掉针刺。** 蜜蜂与杀人蜂只有一次 `numrandomloot = 1` 的加权抽取，针刺占 5/6；另外 1/6 掉蜂蜜。[随机表](../../../raw/current/scripts/prefabs/bee.lua#L194-L197)、[抽取](../../../raw/current/scripts/components/lootdropper.lua#L99-L112)
- **边界：随机表挂在共用工厂里。** 两个注册的专属函数只改血量／伤害／Brain，掉落与工作组件都在 `commonfn`；改其中一个注册的掉率要动共用段。[共用工厂](../../../raw/current/scripts/prefabs/bee.lua#L139-L245)
- **边界：烧掉一叠，得到同叠数的灰烬。** `DefaultBurntFn` 把原 `stacksize` 抄给灰烬（封顶灰烬上限），不是按 1 个结算。[烧毁回调](../../../raw/current/scripts/standardcomponents.lua#L31-L43)
- **边界：并堆不受移动限制。** `SetIgnoreMovingFast(true)` 与犬牙的默认行为相反，不能按同族"都要求静止"推断。[判定](../../../raw/current/scripts/components/selfstacker.lua#L18-L26)
- **未核验**：未运行游戏。5/6 是权重推导、1% 是无幸运基准；蜂群刷新量与实战掉落频率未核验。

## 验证记录

- 源码审阅：2026-11-10，Steam buildid 26474873。逐行读 `stinger.lua`（全 48 行）；`bee.lua` 的 `commonfn`、两个注册与掉落段；`beeguard.lua` 的 1% 接线；`beequeen.lua` 的共享表与接线；`slingshotammo.lua` 的针刺弹药条目；`stackable`／`selfstacker`／`lootdropper`／`standardcomponents` 的燃烧与并堆函数；`recipes.lua` 的十二条配方；`tuning.lua` 的堆叠与燃烧常量；`chinese_s.po` 名称键。
  未覆盖：蜂类 Brain 与刷新、弹弓与吹箭发射链（归远程武器页）；场景宝箱 `scenarios/chest_bees.lua` 里的针刺摆放（[L13](../../../raw/current/scripts/scenarios/chest_bees.lua#L13)）只作来源提及，未展开箱内数量与刷新。
- 游戏运行：未运行；概率与叠数结论为源码推导。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 结果见交付回报。

## 相关链接

- 远程武器与弹药：[[reference/prefabs/ranged-weapons]]
- 组件：[[reference/components/stackable]]、[[reference/components/selfstacker]]、[[reference/components/burnable]]
- 同批页面：[[reference/prefabs/honeycomb|蜜脾]]、[[reference/prefabs/houndstooth|犬牙]]、[[reference/prefabs/tentaclespots|触手皮]]、[[reference/prefabs/walrus-tusk|海象牙]]

返回 [[reference/prefabs/index|Prefab 索引]]。
