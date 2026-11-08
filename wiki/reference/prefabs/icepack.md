---
title: "保鲜背包（icepack）"
created: "2026-11-07"
updated: "2026-11-08"
type: prefab
summary: "原版八格身体槽背包的保鲜入口：fridge＋nocool 标签组合让包内食物按 0.5 基础倍率腐坏（环境 <0 且物品带 frozen 又未开 frozenfiremult 时才直接停腐；湿润/季节再加成），并能跳过冰箱的主动降温；配方熊皮＋齿轮＋电子元件（炼金引擎）。"
aliases: ["icepack", "保鲜背包", "保鲜包", "保温包", "Insulated Pack"]
tags: [prefab, item, equipment]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/icepack.lua", "containers.lua", "recipes.lua", "recipes_filter.lua", "constants.lua", "prefabs/scienceprototyper.lua", "components/perishable.lua", "components/temperature.lua", "components/inventoryitemtemperature.lua", "components/inventory.lua", "tuning.lua", "prefabs/heatrock.lua", "brains/beargerbrain.lua", "stategraphs/SGwilson.lua", "standardcomponents.lua", "prefabskin.lua", "prefabs/skinprefabs.lua", "prefabskins.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 保鲜背包（icepack）

## 快速判断

- 职责与适用：可拾取、装备 BODY 槽的八格容器背包，布局与普通背包完全同参；核心特色是 `fridge`＋`nocool` 双标签：包内食物按 **0.5 的容器基础倍率**腐坏（之后还会被环境与局部修正重算：环境 <0 且物品带 `frozen` 又没有 `frozenfiremult` 时直接改 0；湿润 ×1.3、非冻结的冬季 ×0.75、过热 ×1.25），但会跳过冰箱的主动降温（物品不被压向冰点、热食不 `AddChill`）。适合借鉴“用标签接入既有消费端”与“容器型装备”的组装。主体构造见 [工厂函数](../../../raw/current/scripts/prefabs/icepack.lua#L39-L88)。
- 优先使用：改保鲜强度 → `TUNING.PERISH_FRIDGE_MULT`（全局）或按 [[reference/components/preserver#关键接口]] 换 preserver 组件；给自定义容器复制同款组合 → 本页[典型接入](#典型接入)；穿脱开关、Replica／classified 同步等通用背包机制复用 [[reference/prefabs/backpack]]。
- 重要副作用：装备时 `container:Open(owner)`、脱下或入模型时 `Close`（[装备回调](../../../raw/current/scripts/prefabs/icepack.lua#L12-L37)）；保鲜只看容器实体、不依赖是否穿戴（[owner 判定](../../../raw/current/scripts/components/perishable.lua#L68-L71)）。
- 修改边界：无 insulator／waterproofer 组件，不降低穿戴者体温、不防湿；客户端在 `SetPristine` 后返回（[端分界](../../../raw/current/scripts/prefabs/icepack.lua#L65-L69)）；本页不覆盖冰箱（icebox）、盐盒或厨师包（spicepack）。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `icepack` | `fn`：公共端外观（bank `backpack1`、build `swap_icepack`）、`backpack`／`fridge`／`nocool` 标签、floater（`small`/偏移 0.15，`swap_data = {bank = "backpack1", anim = "anim"}`）→ `SetPristine` → 主模拟 inspectable／inventoryitem／equippable／container／作祟 helper | 物品＋装备（BODY）＋容器；`cangoincontainer=false`；`prefabs` 依赖表为空；无 SG／Brain。[工厂与组件](../../../raw/current/scripts/prefabs/icepack.lua#L39-L88)、[依赖声明](../../../raw/current/scripts/prefabs/icepack.lua#L7-L10) |

名称键 `STRINGS.NAMES.ICEPACK` 对应 Insulated Pack／“保鲜背包”。[简中条目](../../../raw/current/scripts/languages/chinese_s.po#L344090-L344092)；获取走配方而非掉落，见[关联机制导航](#关联机制导航)配方行。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：equippable | BODY 槽，onequip（外观＋`Open`）、onunequip（清外观＋`Close`）、onequiptomodel（仅 `Close`），皮肤分支与 krampus_sack／backpack 同构 | 主模拟 | [绑定](../../../raw/current/scripts/prefabs/icepack.lua#L76-L80)、[回调](../../../raw/current/scripts/prefabs/icepack.lua#L12-L37)；开关语义见 [[reference/prefabs/backpack#穿脱背包如何打开和关闭容器]] |
| 组件：container | `WidgetSetup("icepack")` 直接引用 `params.backpack`：两列四行 8 格、`ui_backpack_2x4`、`type="pack"`、`openlimit=1`、侧边栏 | 布局与普通背包共用同一参数对象 | [共享布局](../../../raw/current/scripts/containers.lua#L43)、[backpack 布局](../../../raw/current/scripts/containers.lua#L23-L41)；[[reference/components/container#关键接口]] |
| 组件：inventoryitem | 拾取持有；`cangoincontainer=false` 禁止放入其他容器 | 主模拟 | [配置](../../../raw/current/scripts/prefabs/icepack.lua#L73-L74) |
| API／数据：`fridge` → 基础倍率 0.5 | 包内物品的 perishable 找不到 preserver 时按 `fridge` 标签取 `PERISH_FRIDGE_MULT`（tuning 值 .5）作**基础倍率**；同一函数随后还会改它：环境 <0 且 `frozen` 且无 `frozenfiremult` → 直接 0（覆盖，不是相乘）、湿润 ×1.3、非冻结的冬季 ×0.75、过热 ×1.25，最后再乘 `localPerishMultiplyer` 与全局倍率 | 服务端逐 tick 读取；owner 是容器实体；温度取容器所在位置 | [fridge 分支](../../../raw/current/scripts/components/perishable.lua#L78-L91)、[环境与局部修正](../../../raw/current/scripts/components/perishable.lua#L107-L146)、[调值](../../../raw/current/scripts/tuning.lua#L2077-L2088) |
| API／数据：`nocool` → 跳过冰箱主动降温 | 温度类消费端把 icepack 排除在“冰箱主动降温”外：包内物品温度按环境变化（保温石不被动降温）；热食跳过 `AddChill` 保持热量；冻结食物**不走冰箱分支**的 0（该分支条件是 `fridge and not nocool`），包内取 0.5 基础倍率 | 三处消费端均在服务端。注意 `nocool` 只管“冰箱主动降温＋热食 AddChill”，管不到 perishable 末尾那段 `ambient_temperature < 0` 的判断——环境 <0 且物品带 `frozen` 又未开 `frozenfiremult` 时照样改 0 | [物品温度排除](../../../raw/current/scripts/components/temperature.lua#L328-L339)、[库存物温度排除](../../../raw/current/scripts/components/inventoryitemtemperature.lua#L220-L227)、[热食保温](../../../raw/current/scripts/components/perishable.lua#L139-L147) |
| API／数据：配方与制作栏 | `bearger_fur`×1＋`gears`×1＋`transistor`×1，`TECH.SCIENCE_TWO`（SCIENCE=2，需二本炼金引擎）；归入 CLOTHING／COOKING／CONTAINERS 三个筛选 | 原料名：熊皮／齿轮／电子元件 | [配方](../../../raw/current/scripts/recipes.lua#L518)、[科技等级](../../../raw/current/scripts/constants.lua#L1246)、[炼金引擎](../../../raw/current/scripts/prefabs/scienceprototyper.lua#L299-L300)、[筛选归属](../../../raw/current/scripts/recipes_filter.lua#L1093) |
| API／数据：`fridge`／`backpack` 标签 AI 消费 | 熊獾只挑「非空且 `eater:CanEat` 找得到内容物」的包：`fridge` 分支要求 workable（本包没有），实际落入 `backpack` 分支；该分支先找蜂蜜类可食物件并给 `STEAL`，没有可食物件的非空包整个不入选 | 服务端 Brain | [fridge 分支](../../../raw/current/scripts/brains/beargerbrain.lua#L106-L116)、[backpack 分支](../../../raw/current/scripts/brains/beargerbrain.lua#L124-L131) |
| 作祟／漂浮／脚步声 | 漂浮 `MakeInventoryFloatable("small", 0.15, nil, nil, nil, swap_data)`（`swap_data = {bank = "backpack1", anim = "anim"}`）在 `SetPristine` **前**、公共端执行；作祟 `MakeHauntableLaunchAndDropFirstItem` 在**主模拟**分支（本 build 仅发射，掉物分支被注释）；`foleysound` 为通用背包音 | 两者初始化端不同，别都当“公共端 helper” | [调用](../../../raw/current/scripts/prefabs/icepack.lua#L85-L85)、[作祟 helper](../../../raw/current/scripts/standardcomponents.lua#L1486-L1525)、[foley 播放](../../../raw/current/scripts/stategraphs/SGwilson.lua#L101-L105) |
| 皮肤 | `icepack_kitchen`（Charming Cooler Chest／“魅力冷藏箱”，Elegant＋Woven，tags 含 CRAFTABLE）；共用 init 仅换 build | 唯一一款皮肤；无额外组件或 FX | [皮肤清单](../../../raw/current/scripts/prefabskins.lua#L1089-L1092)、[皮肤注册](../../../raw/current/scripts/prefabs/skinprefabs.lua#L7019-L7028)、[共用 init](../../../raw/current/scripts/prefabskin.lua#L184)、[名称](../../../raw/current/scripts/languages/chinese_s.po#L398780-L398782) |
| 资源 | `anim/swap_icepack.zip`、`anim/ui_backpack_2x4.zip`；小地图图标 `icepack.png` | 声明并实际用于 bank/build、界面与图标 | [资源声明](../../../raw/current/scripts/prefabs/icepack.lua#L1-L5)、[图标](../../../raw/current/scripts/prefabs/icepack.lua#L44-L48) |

## 典型接入

### 给容器复制“保鲜不降温”组合

- 前提与选择：`modmain.lua`，`AddPrefabPostInit` 由模组环境提供（[modutil.lua#L593](../../../raw/current/scripts/modutil.lua#L593)）；`fridge`／`nocool` 是普通实体标签，无需新组件。原版在 `SetPristine` 前添加这两个标签（[标签](../../../raw/current/scripts/prefabs/icepack.lua#L56-L58)），自定义 prefab 建议同位置；post-init 场景直接 `AddTag` 即可。
- 生效与撤销：perishable／temperature 消费端逐 tick 读取 owner 实体标签，标签在即生效、`RemoveTag` 即撤销。但 `AddPrefabPostInit` 只接管**此后完成构造**的实例（含重新载入时重建的），不会就地扫描世界里已经存在的背包——要让现存实例生效，得自己拿到实体再 `AddTag`。示例以原版 backpack 为对象，也可换自定义容器注册名。
- 原版依据：标签消费链见上文[关联机制导航](#关联机制导航)；背包穿脱开关与溢出容器语义见 [[reference/prefabs/backpack]]。

```lua
-- modmain.lua：让原版普通背包获得与保鲜背包相同的“保鲜不降温”组合
AddPrefabPostInit("backpack", function(inst)
    inst:AddTag("fridge")
    inst:AddTag("nocool")
end)
```

- 调整与验证：保鲜强度是全局 `TUNING.PERISH_FRIDGE_MULT`，只调单个容器需换 preserver 组件并链接 [[reference/components/preserver#关键接口]]。示例未在游戏内运行，机制依据上文源码审阅。

## 常见陷阱与限制

- **误区：保鲜背包能保温（玩家体温）**：本包没有 insulator 组件，`fridge` 消费端只作用于包内物品的腐坏与温度，不影响穿戴者体温；夏天防中暑请用降温帽类装备。“保温”仅指热食在包内不被 chill（[跳过 AddChill](../../../raw/current/scripts/components/perishable.lua#L139-L147)）。
- **误区：等同小冰箱（一半速度）**：0.5 只是容器分支的基础倍率，包内真实速度会被后续修正重算。`nocool` 确实把冻结物挡在**冰箱分支**的 0 之外，包内取 0.5；但 perishable 末尾还有一段与环境温度有关的判断——环境 <0 时，带 `frozen` 且没有 `frozenfiremult` 的物品倍率**直接改成 0**（与 `nocool` 无关），所以冬天背到零下照样完全不腐。物品温度也不会像冰箱那样被压向冰点（[冰箱分支](../../../raw/current/scripts/components/perishable.lua#L82-L86)、[环境修正](../../../raw/current/scripts/components/perishable.lua#L107-L146)）。
- **边界：容量即普通背包**：`params.icepack = params.backpack`，8 格 `ui_backpack_2x4`；不是独立 6 格布局，也不因保鲜减少格子。[共享布局](../../../raw/current/scripts/containers.lua#L43)
- **边界：熊獾打劫目标要“有它能吃的东西”**：非空只是前提，`backpack` 分支还要 `eater:CanEat` 找得到内容物才入选；有可食物件就会成为 `STEAL` 目标，蜂蜜类的只是先占 `honeyed_backpack` 分支、在动作优先级上压过 `harvest`/`hammer`。装一包石头不算目标。[目标选择](../../../raw/current/scripts/brains/beargerbrain.lua#L124-L131)
- **边界：作祟不掉物**：与普通背包共用作祟 helper，本 build 掉物分支被注释，只发射本体；详见 [[reference/prefabs/backpack#常见陷阱与限制]]。

## 验证记录

- 源码审阅：2026-11-07，Steam buildid 26335394。全文实读 `prefabs/icepack.lua`（90 行）；核验注册名、与 backpack 共享的 8 格布局（containers.lua L43、L23-L41）、配方与科技等级（recipes.lua L518、constants.lua L1246、scienceprototyper.lua L299-L300）、`fridge`／`nocool` 全部服务端消费端（perishable、temperature、inventoryitemtemperature、tuning、beargerbrain）、皮肤注册与共用 init、端口分界。
- 名称核验：`chinese_s.po` 以 python 按 `msgctxt "STRINGS.NAMES.ICEPACK"`（L344090-L344092）与 `STRINGS.SKIN_NAMES.icepack_kitchen`（L398780-L398782）定位。
- 游戏运行：未运行；未验证实际保鲜速率表现、皮肤贴图内容、冻结食物全程数值链与装备栏扩展模组兼容。
- 结构检查：validate_page.py（含 `--check-indexes`）通过；standard 档上限 240 行 / 32768 字节。
- 2026-11-08 build 26474873：随 [P-1](../../updates/build-26474873/detail-prefabs.md) 修订浮水断言——工厂补第 7 参 `swap_data = {bank = "backpack1", anim = "anim"}`（`MakeInventoryFloatable(inst, "small", 0.15, nil, nil, nil, swap_data)`），此前“无 swap_data”不再成立；`swap_bank` 为 nil 时 floater 走 `SetSwapData`（[helper](../../../raw/current/scripts/standardcomponents.lua#L1772-L1791)）。其余结论沿用。

返回 [[reference/prefabs/storage-heavy-index|背包收纳与重物搬运]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-08 build 26474873：本页随版本迁移升版（锚点迁移 6 处），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-11）。
