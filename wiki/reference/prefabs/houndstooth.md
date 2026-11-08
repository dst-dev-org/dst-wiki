---
title: "犬牙（houndstooth）"
created: "2026-11-10"
updated: "2026-11-10"
type: prefab
summary: "单注册物品犬牙：七种猎犬与座狼、猎犬丘、犬骨、宝箱拟态的掉落概率，40 叠与 selfstacker，嚎弹炮弹舱的 blowpipeammo 弹药身份，以及吹箭、陷阱、衣物的配方消费者。"
aliases: ["houndstooth", "犬牙", "猎犬牙", "嚎弹炮弹药"]
tags: [prefab, item]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/houndstooth.lua", "prefabs/hound.lua", "prefabs/warg.lua", "prefabs/warglet.lua", "prefabs/houndmound.lua", "prefabs/houndbone.lua", "prefabs/chest_mimic.lua", "prefabs/houndstooth_blowpipe.lua", "components/stackable.lua", "components/selfstacker.lua", "components/lootdropper.lua", "components/reloaditem.lua", "components/snowmandecor.lua", "standardcomponents.lua", "containers.lua", "componentactions.lua", "actions.lua", "recipes.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# 犬牙（houndstooth）

## 快速判断

- 职责与适用：犬牙是**唯一一个注册**的可堆叠材料（简中实显 **犬牙**，40 叠），本体只做浮空、拾取、自动并堆、雪人装饰标记与"弹舱弹药"标记，没有燃烧、燃料、食用或交易组件。[构造](../../../raw/current/scripts/prefabs/houndstooth.lua#L17-L51)
- 优先使用：查哪只猎犬掉多少看[掉落来源](#掉落来源)；查它能造什么、进哪个弹舱看[弹药与配方](#弹药与配方)；查标签与共享端差异看[注册与构造](#注册与构造)。
- 重要副作用：`blowpipeammo` 的消费者只有**嚎弹炮** `houndstooth_blowpipe` 的弹舱；普通吹箭 `blowdart_pipe` 里犬牙是**配方材料**，不是弹药。[弹舱过滤](../../../raw/current/scripts/containers.lua#L1944-L1950)、[吹箭配方](../../../raw/current/scripts/recipes.lua#L485)
- 修改边界：猎犬、座狼、猎犬丘、犬骨只写到"产出犬牙"的分支；吹箭与嚎弹炮本体归 [[reference/prefabs/ranged-weapons]]（S5）。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `houndstooth` | `fn()`：实体 → Transform／AnimState／SoundEmitter／Network → `MakeInventoryPhysics` → bank `houndstooth`、build `hounds_tooth`、播 `idle` → 两个 ammo 标签 → `pickupsound="rock"` → 浮空 `("small", nil, {0.6,0.55,0.6})` → 水面 override build → `selfstacker` 标签 → `SetPristine()` → 主模拟挂 `reloaditem`／`stackable`／`inspectable`／`inventoryitem`／`selfstacker`／`snowmandecor` → `MakeHauntableLaunchAndSmash` | 唯一注册，无变体；`assets` 两包：`hounds_tooth.zip` 与 `hounds_tooth_water.zip`。[工厂](../../../raw/current/scripts/prefabs/houndstooth.lua#L7-L56) |

- **两个 ammo 标签分工不同**：`blowpipeammo` 只被嚎弹炮弹舱 `itemtestfn` 读取；`reloaditem_ammo` 只改 `CHANGE_TACKLE` 的动作字符串（显示 AMMO），装舱动作由 `reloaditem` 组件候选提供。[标签](../../../raw/current/scripts/prefabs/houndstooth.lua#L21-L22)、[候选](../../../raw/current/scripts/componentactions.lua#L2856-L2863)、[动作串](../../../raw/current/scripts/actions.lua#L1442-L1447)
- **堆叠是显式 40**：`stackable.maxsize = TUNING.STACK_SIZE_SMALLITEM`，不落组件默认的 20（S7）。[赋值](../../../raw/current/scripts/prefabs/houndstooth.lua#L40-L41)、[常量](../../../raw/current/scripts/tuning.lua#L81-L83)
- **自动并堆有速度门槛**：`selfstacker` 默认要求速度平方 <1 才并堆；犬牙没有调用 `SetIgnoreMovingFast`（针刺才有）。[组件](../../../raw/current/scripts/components/selfstacker.lua#L18-L26)、[挂载](../../../raw/current/scripts/prefabs/houndstooth.lua#L47)
- **作祟只有抛飞**：`MakeHauntableLaunchAndSmash` 的碎裂分支整段被 `#HAUNTFIX` 注释，实际只 `Launch`。[helper](../../../raw/current/scripts/standardcomponents.lua#L1071-L1105)

### 名称核验

`NAMES.HOUNDSTOOTH`＝**犬牙**，是简中实显名（S1）；本 prefab 没有改名调用。[po 名称键](../../../raw/current/scripts/languages/chinese_s.po#L344009-L344012)

## 掉落来源

主线是 `SetChanceLootTable` 共享表的**逐条独立判定**，不是"从池里抽一件"；犬骨走 `AddChanceLoot` 的单条列表，同样逐条掷。[掷点](../../../raw/current/scripts/components/lootdropper.lua#L213-L225)

| 宿主 | 犬牙条目 | 同表其它产出 | 证据 |
|---|---|---|---|
| 猎犬 `hound` | 1 条 0.125 | 怪物肉 1.000 | [表](../../../raw/current/scripts/prefabs/hound.lua#L110-L114)、[接线](../../../raw/current/scripts/prefabs/hound.lua#L563-L564) |
| 火焰猎犬 `firehound` | 1 条 1.0 | 怪物肉 1.0、红宝石 0.2 | [表](../../../raw/current/scripts/prefabs/hound.lua#L116-L121)、[接线](../../../raw/current/scripts/prefabs/hound.lua#L662) |
| 寒冰猎犬 `icehound` | 2 条各 1.0（必得 2 个） | 怪物肉 1.0、蓝宝石 0.2 | [表](../../../raw/current/scripts/prefabs/hound.lua#L123-L129)、[接线](../../../raw/current/scripts/prefabs/hound.lua#L706) |
| 月光猎犬 `moonhound` | **继承** `hound` 表：1 条 0.125 | 怪物肉 1.000 | [默认接线](../../../raw/current/scripts/prefabs/hound.lua#L563-L564)、[未覆盖](../../../raw/current/scripts/prefabs/hound.lua#L732-L739) |
| 黏土猎犬 `clayhound` | 1 条 0.1 | 红袋 0.2 | [表](../../../raw/current/scripts/prefabs/hound.lua#L131-L135)、[接线](../../../raw/current/scripts/prefabs/hound.lua#L784) |
| 恐怖猎犬 `mutatedhound` | 2 条各 1.0 | 怪物肉 1.0 | [表](../../../raw/current/scripts/prefabs/hound.lua#L137-L142)、[接线](../../../raw/current/scripts/prefabs/hound.lua#L830) |
| 蔷薇狼 `hedgehound` | **无**：`SetChanceLootTable(nil)` | 死亡回调另生成 `inst.hedgeitem` | [清表](../../../raw/current/scripts/prefabs/hound.lua#L889-L919) |
| 座狼／黏土座狼／姜饼座狼 | 3 条 1.00／0.66／0.33，期望 **1.99**（推导） | 各自肉／红袋／冬宴燃料表 | [座狼](../../../raw/current/scripts/prefabs/warg.lua#L96-L108)、[黏土](../../../raw/current/scripts/prefabs/warg.lua#L110-L122)、[姜饼](../../../raw/current/scripts/prefabs/warg.lua#L124-L141) |
| 青年座狼 `warglet` | 2 条 1.00／0.33，期望 **1.33**（推导） | 怪物肉 | [表](../../../raw/current/scripts/prefabs/warglet.lua#L40-L41) |
| 猎犬丘 `houndmound` | 3 条各 1.00（击杀必得 3 个；300 生命、不可锤） | 犬骨 2、红／蓝宝石各 0.01 | [表](../../../raw/current/scripts/prefabs/houndmound.lua#L19-L28)、[生命与战斗](../../../raw/current/scripts/prefabs/houndmound.lua#L171-L196)、[死亡掉落](../../../raw/current/scripts/prefabs/houndmound.lua#L75-L85)、[接线](../../../raw/current/scripts/prefabs/houndmound.lua#L193) |
| 犬骨 `houndbone` | 骨片 1.00（共享表）＋第 3 型追加犬牙 0.5 | 先 `SetChanceLootTable('houndbone')`，第 3 型再 `ClearChanceLoot`（只清 `chanceloot`）后加犬牙 | [共享表](../../../raw/current/scripts/prefabs/houndbone.lua#L15-L18)、[分支](../../../raw/current/scripts/prefabs/houndbone.lua#L20-L29)、[接线](../../../raw/current/scripts/prefabs/houndbone.lua#L81-L82)、[清表范围](../../../raw/current/scripts/components/lootdropper.lua#L82-L85) |
| 宝箱拟态 `chest_mimic` | 2 条各 0.50 | 恐怖燃料；`lootsetupfn` 另设 `SetLoot` 不清概率表 | [表](../../../raw/current/scripts/prefabs/chest_mimic.lua#L208-L218)、[接线](../../../raw/current/scripts/prefabs/chest_mimic.lua#L393-L395) |
| 风滚草 `tumbleweed` | 权重 1 | 生成时抽 3 次（有放回），被拾取时把已抽好的 3 件全部落地；单项约 1%／抽、3 抽≈3%（推导，分母随存档解锁态变化） | [权重](../../../raw/current/scripts/prefabs/tumbleweed.lua#L178)、[生成时抽取](../../../raw/current/scripts/prefabs/tumbleweed.lua#L494)、[3 抽](../../../raw/current/scripts/prefabs/tumbleweed.lua#L213-L245)、[拾取落地](../../../raw/current/scripts/prefabs/tumbleweed.lua#L121-L125)、[存档只存 burnt](../../../raw/current/scripts/prefabs/tumbleweed.lua#L324-L326) |

- **月光猎犬显示名是"猎犬"**：`fnmoon` 调 `SetPrefabNameOverride("hound")`，po 也没有 `MOONHOUND` 键。[改名](../../../raw/current/scripts/prefabs/hound.lua#L735)、[猎犬名键](../../../raw/current/scripts/languages/chinese_s.po#L343990-L343992)
- **`SetLoot` 不清概率表**：宝箱拟态的 `lootsetupfn` 只清固定／随机段，`chanceloottable` 照常结算。[SetLoot](../../../raw/current/scripts/components/lootdropper.lua#L30-L35)、[设置函数](../../../raw/current/scripts/prefabs/chest_mimic.lua#L297-L306)

## 弹药与配方

| 去向 | 消耗 | 条件与产物 | 证据 |
|---|---|---|---|
| 嚎弹炮弹舱 | 每次发射拆 1 个 | 弹舱只收 `blowpipeammo`；发射回调从 1 号槽取整叠再拆 1 个移除 | [过滤](../../../raw/current/scripts/containers.lua#L1944-L1950)、[拆弹](../../../raw/current/scripts/prefabs/houndstooth_blowpipe.lua#L69-L77)、[弹舱](../../../raw/current/scripts/prefabs/houndstooth_blowpipe.lua#L225) |
| 吹箭 `blowdart_pipe` | 芦苇 2＋犬牙 1＋冬鸟羽 1 | `TECH.SCIENCE_TWO` | [配方](../../../raw/current/scripts/recipes.lua#L485) |
| 尖牙陷阱 `trap_teeth` | 木头 1＋绳索 1＋犬牙 1 | `TECH.SCIENCE_TWO` | [配方](../../../raw/current/scripts/recipes.lua#L510) |
| 缝纫包 `sewing_kit` | 木头 1＋蛛丝 8＋犬牙 2 | `TECH.SCIENCE_TWO` | [配方](../../../raw/current/scripts/recipes.lua#L528) |
| 毛衣背心 `sweatervest` | 犬牙 8＋蛛丝 6 | `TECH.SCIENCE_TWO` | [配方](../../../raw/current/scripts/recipes.lua#L547) |
| 饥饿上限模块 1 | 扫描数据 2＋犬牙 1 | `TECH.ROBOTMODULECRAFT_ONE` | [配方](../../../raw/current/scripts/recipes.lua#L755) |
| 宠物幼犬领养 | 犬牙 1＋怪物千层饼 1 | `TECH.ORPHANAGE_ONE` | [配方](../../../raw/current/scripts/recipes.lua#L821) |
| 嚎弹炮（月石锻炉） | 脉冲笼满 1＋月植壳 3＋纯辉 3＋月石 3 | `TECH.LUNARFORGING_TWO` | [配方](../../../raw/current/scripts/recipes.lua#L859) |
| 骨片⇄犬牙转化 | 2 换 1，双向 | 威尔逊炼金技能 | [转骨片](../../../raw/current/scripts/recipes.lua#L1491)、[转犬牙](../../../raw/current/scripts/recipes.lua#L1492) |
| 蝙蝠首领帽解构 | 蝙蝠翼 2＋犬牙 4＋怪物肉 1 | 解构法杖拆解 | [拆解表](../../../raw/current/scripts/recipes.lua#L1688) |

- **雪人装饰与水面外观**：`snowmandecor` 是空壳标记，动作候选在 `componentactions` 的 `snowmandecor` 分支；`AddOverrideBuild("hounds_tooth_water")` 只在漂浮时换水面 build，仍是同一注册。[组件](../../../raw/current/scripts/components/snowmandecor.lua#L1-L7)、[候选](../../../raw/current/scripts/componentactions.lua#L1745-L1747)、[override](../../../raw/current/scripts/prefabs/houndstooth.lua#L27)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`stackable` | 显式 40 | 主模拟 | [[reference/components/stackable]]；[赋值](../../../raw/current/scripts/prefabs/houndstooth.lua#L40-L41) |
| 组件：`selfstacker` | 自动并堆，默认带速度门槛 | 主模拟；实体唤醒时触发 | [[reference/components/selfstacker]]；[挂载](../../../raw/current/scripts/prefabs/houndstooth.lua#L47) |
| 组件：`reloaditem` | 仅作标记，提供装舱动作候选 | 主模拟；需手持已打开且能收它的容器 | [组件](../../../raw/current/scripts/components/reloaditem.lua#L2-L4)、[候选](../../../raw/current/scripts/componentactions.lua#L2856-L2863) |
| 组件：`snowmandecor` | 雪人装饰材料标记 | 主模拟 | [组件](../../../raw/current/scripts/components/snowmandecor.lua#L1-L7) |
| 标签：`blowpipeammo` | 嚎弹炮弹舱过滤 | 公共端（pristine 前） | [标签](../../../raw/current/scripts/prefabs/houndstooth.lua#L21)、[过滤](../../../raw/current/scripts/containers.lua#L1948-L1950) |
| 标签：`reloaditem_ammo` | 动作串显示 AMMO | 公共端 | [标签](../../../raw/current/scripts/prefabs/houndstooth.lua#L22)、[动作串](../../../raw/current/scripts/actions.lua#L1446) |
| 宿主：七种猎犬 | 同一工厂；五种自有表，月光猎犬复用 `hound` 表，蔷薇狼清表 | 服务端死亡 `DropLoot` | [注册](../../../raw/current/scripts/prefabs/hound.lua#L923-L929) |
| 宿主：`warg`／`warglet`／`houndmound`／`houndbone` | 额外产出口 | 死亡（`houndbone` 为锤击） | 见[掉落来源](#掉落来源) |
| 关联 Prefab：`houndstooth_blowpipe`／`houndstooth_proj` | 弹药与弹体归属 | 月石锻炉武器 | [[reference/prefabs/ranged-weapons]]；[弹舱](../../../raw/current/scripts/prefabs/houndstooth_blowpipe.lua#L225) |
| 资源：`hounds_tooth.zip`／`hounds_tooth_water.zip` | 陆地与水面两套 build | 原版资源，不随页分发 | [资产](../../../raw/current/scripts/prefabs/houndstooth.lua#L1-L5) |

## 常见陷阱与限制

- **误区：犬牙是吹箭的弹药。** 吹箭只把犬牙当配方材料；把它当弹药的是嚎弹炮弹舱。[配方](../../../raw/current/scripts/recipes.lua#L485)、[弹舱过滤](../../../raw/current/scripts/containers.lua#L1948-L1950)
- **边界：蔷薇狼不掉犬牙。** `fnhedge` 把概率表清成 `nil`，掉落改由 `hedgeitem` 死亡回调单独生成。[清表](../../../raw/current/scripts/prefabs/hound.lua#L912)、[回调](../../../raw/current/scripts/prefabs/hound.lua#L860-L866)
- **边界：月光猎犬沿用猎犬表。** 没有自己的共享表，只改显示名；按注册名检索掉落会漏掉它。[接线](../../../raw/current/scripts/prefabs/hound.lua#L732-L739)
- **边界：1.00 是"必得这一条"，不是"只掉这一件"。** 寒冰与恐怖猎犬各两条 1.00 就是两个犬牙。[寒冰表](../../../raw/current/scripts/prefabs/hound.lua#L123-L129)
- **未核验**：未运行游戏。期望 1.99／1.33 是 Σ条目概率的推导，未做游戏内掉率采样；犬骨三型分布与宝箱拟态吞物逻辑未展开。

## 验证记录

- 源码审阅：2026-11-10，Steam buildid 26474873。读 `houndstooth.lua` 全文；`hound.lua` 的五张共享表、七个工厂接线与 `SetPrefabNameOverride`；`warg.lua`／`warglet.lua`／`houndmound.lua`／`houndbone.lua`／`chest_mimic.lua` 的表与触发点；`houndstooth_blowpipe.lua` 的弹舱注册；`stackable`／`selfstacker`／`lootdropper`／`reloaditem`／`snowmandecor` 组件；`containers.lua` 过滤；`componentactions`／`actions` 的装舱动作；`recipes.lua` 十条配方；`tuning.lua` 堆叠常量；`chinese_s.po` 名称键。
  未覆盖：猎犬 AI 与刷新批次、嚎弹炮发射链（归远程武器页）、其它场景里的犬牙摆放。
- 游戏运行：未运行；概率与期望均为源码推导。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 结果见交付回报。

## 相关链接

- 远程武器与弹舱：[[reference/prefabs/ranged-weapons]]
- 组件：[[reference/components/stackable]]、[[reference/components/selfstacker]]、[[reference/components/lootdropper]]
- 同批页面：[[reference/prefabs/honeycomb|蜜脾]]、[[reference/prefabs/stinger|针刺]]、[[reference/prefabs/tentaclespots|触手皮]]、[[reference/prefabs/walrus-tusk|海象牙]]

返回 [[reference/prefabs/index|Prefab 索引]]。
