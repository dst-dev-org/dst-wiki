---
title: "坎普斯背包（krampus_sack）"
created: "2026-11-07"
updated: "2026-11-07"
type: prefab
summary: "原版十四格大容量身体槽背包：坎普斯 1% 与克劳斯袋 10% 掉落、无配方；穿脱开关容器与普通背包同源，本体不吸湿但不提供防水。"
aliases: ["krampus_sack", "坎普斯背包", "小偷背包", "Krampus Sack", "十四格背包"]
tags: [prefab, item, equipment]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/krampus_sack.lua", "prefabs/krampus.lua", "containers.lua", "components/lootdropper.lua", "components/klaussackloot.lua", "prefabs/klaus_sack.lua", "prefabs/world.lua", "components/waterproofer.lua", "components/inventory.lua", "components/acidlevel.lua", "brains/beargerbrain.lua", "stategraphs/SGwilson.lua", "standardcomponents.lua", "prefabskin.lua", "prefabs/skinprefabs.lua", "prefabskins.lua", "languages/chinese_s.po", "constants.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 坎普斯背包（krampus_sack）

## 快速判断

- 职责与适用：可拾取、装备 BODY 槽的十四格容器背包，原版容量最大的身体槽背包；无配方、仅靠掉落获取。适合借鉴“大容量独立布局”与“稀有掉落装备”的组装。主体构造见 [工厂函数](../../../raw/current/scripts/prefabs/krampus_sack.lua#L34-L86)。
- 优先使用：改掉率 → 本页[典型接入](#典型接入)的共享掉落表整表替换或克劳斯袋概率；改槽数／布局 → `containers.lua` 的 `params.krampus_sack`；穿脱开关、Replica／classified 同步、人物溢出容器等通用背包机制直接复用 [[reference/prefabs/backpack]]，本页不复制。
- 重要副作用：装备时 `container:Open(owner)`，脱下或入模型展示时 `Close`（[装备回调](../../../raw/current/scripts/prefabs/krampus_sack.lua#L7-L32)）；挂 `waterproofer` 组件但 `effectiveness=0`，仅令背包自体不吸湿（[组件与标签](../../../raw/current/scripts/prefabs/krampus_sack.lua#L52-L78)）。
- 修改边界：单一注册名，四款皮肤均为纯换 build；客户端在 `SetPristine` 后即返回、不建组件（[端分界](../../../raw/current/scripts/prefabs/krampus_sack.lua#L60-L64)）。本页不覆盖 icepack、piggyback 或装备栏扩展模组；未逐项审校 container／equippable 组件内部。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `krampus_sack` | `fn`：公共端外观（bank `backpack1`、build `swap_krampus_sack`）、地图图标、`backpack`／`waterproofer` 标签、floater → `SetPristine` → 主模拟 inspectable／inventoryitem／equippable／waterproofer／container／作祟 helper | 物品＋装备（BODY，[constants.lua#L658](../../../raw/current/scripts/constants.lua#L658)）＋容器三合一；`cangoincontainer=false` 不能再入其他容器；无 SG／Brain。[工厂与组件](../../../raw/current/scripts/prefabs/krampus_sack.lua#L34-L86) |

名称键 `STRINGS.NAMES.KRAMPUS_SACK` 对应 Krampus Sack／“坎普斯背包”。[简中条目](../../../raw/current/scripts/languages/chinese_s.po#L344445-L344447)；俗称“小偷背包”来自坎普斯（Krampus）即玩家社群所称“小偷”。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：equippable | BODY 槽，绑定 onequip（换装外观＋`Open`）、onunequip（清外观＋`Close`）、onequiptomodel（仅 `Close`） | 主模拟；皮肤走 `OverrideItemSkinSymbol`，无皮肤走 `OverrideSymbol` | [绑定](../../../raw/current/scripts/prefabs/krampus_sack.lua#L71-L75)、[回调](../../../raw/current/scripts/prefabs/krampus_sack.lua#L7-L32)；共用开关语义见 [[reference/prefabs/backpack#穿脱背包如何打开和关闭容器]] |
| 组件：container | `WidgetSetup("krampus_sack")`：两列七行 14 格，`ui_krampusbag_2x8` 布局，`type="pack"`、`openlimit=1`、侧边栏 | 布局定义客户端也须可取（containers 为共享 require） | [布局](../../../raw/current/scripts/containers.lua#L1057-L1079)；[[reference/components/container#关键接口]] |
| 组件：inventoryitem | 拾取与库存持有；`cangoincontainer=false` 禁止放入其他容器 | 主模拟组件；不阻止放入人物物品栏本身 | [配置](../../../raw/current/scripts/prefabs/krampus_sack.lua#L68-L69)；[[reference/components/inventoryitem#关键接口]] |
| 组件：waterproofer | `SetEffectiveness(0)`；组件挂载即 `EnableMoisture(false)` 使背包自体不吸湿 | 主模拟；0 值对穿戴者无任何防水贡献 | [挂载](../../../raw/current/scripts/prefabs/krampus_sack.lua#L77-L78)、[组件副作用](../../../raw/current/scripts/components/waterproofer.lua#L9-L11) |
| API／数据：`waterproofer` 消费端 | `Inventory:GetWaterproofness` 累加装备栏防水值（本包加 0）；酸雨伤害只吸收 `effectiveness > 0` 的装备 | 消费在服务端库存／酸雨逻辑 | [防水累计](../../../raw/current/scripts/components/inventory.lua#L2719-L2730)、[酸雨排除](../../../raw/current/scripts/components/acidlevel.lua#L160-L176) |
| API／数据：`backpack` 标签消费者 | `DropEquipped(keepBackpack)` 可保留背包不掉；熊獾（Bearger）只打劫**非空且 `eater:CanEat` 找得到内容物**的背包（蜂蜜类优先 `STEAL`），装一包石头不算；阿比盖尔花与战歌按标签跳过部分事件监听 | 服务端／AI 消费；标签在 pristine 添加 | [掉落保留](../../../raw/current/scripts/components/inventory.lua#L1920-L1926)、[熊獾目标](../../../raw/current/scripts/brains/beargerbrain.lua#L124-L131) |
| 作祟／漂浮／脚步声 | 漂浮 `MakeInventoryFloatable("med", 0.1, 0.65, ...)` 在 `SetPristine` **前**、公共端执行；作祟 `MakeHauntableLaunchAndDropFirstItem` 在**主模拟**分支（本 build 仅保留发射，掉物分支被注释）；`foleysound` 走装备物移动音 | 两个 helper 的初始化端不同；foley 由玩家 SG 播放 | [作祟 helper](../../../raw/current/scripts/standardcomponents.lua#L1486-L1525)、[漂浮 helper](../../../raw/current/scripts/standardcomponents.lua#L1772-L1790)、[foley 播放](../../../raw/current/scripts/stategraphs/SGwilson.lua#L101-L105)、[调用](../../../raw/current/scripts/prefabs/krampus_sack.lua#L50-L83) |
| 关联 Prefab：`krampus` | 共享掉落表第四条目 `{'krampus_sack', .01}`，与其余条目独立掷点；0.01 是**基础概率**，实际判定走 `LootDropper:GetChance`（按有效击杀者/最后工作者的幸运修正）；`SetChanceLootTable('krampus')` 挂到坎普斯 | 服务端掉落时按名查 `LootTables` | [掉落表](../../../raw/current/scripts/prefabs/krampus.lua#L18-L24)、[挂接](../../../raw/current/scripts/prefabs/krampus.lua#L111-L112)、[[reference/prefabs/krampus]] |
| 关联 Prefab：`klaus_sack` | 克劳斯袋第三堆战利品 10% 概率附带 `krampus_sack`（另有金块与木炭）；这条 10% 是 `math.random() < .1`，**不经过 `GetChance`、不吃幸运** | 冬之盛宴不影响该堆；`klaussackloot` 组件挂在世界实体 | [滚点](../../../raw/current/scripts/components/klaussackloot.lua#L107-L113)、[组件挂载](../../../raw/current/scripts/prefabs/world.lua#L576)、[消费](../../../raw/current/scripts/prefabs/klaus_sack.lua#L66) |
| 皮肤 | `krampus_sack_basket`（锁链篮子）、`_garden`（觅食袋）、`_shadow`（血色大袋）、`_voidbag`（虚空袋）；共用 init 仅换 build | `CreatePrefabSkin` 条目；无额外组件或 FX | [皮肤清单](../../../raw/current/scripts/prefabskins.lua#L1114-L1120)、[皮肤注册](../../../raw/current/scripts/prefabs/skinprefabs.lua#L7206-L7249)、[共用 init](../../../raw/current/scripts/prefabskin.lua#L178)、[篮子名称](../../../raw/current/scripts/languages/chinese_s.po#L398870-L398872) |
| 资源 | `anim/backpack.zip`、`anim/swap_krampus_sack.zip`；小地图图标 `krampus_sack.png` | 声明并实际用于 bank/build 与图标 | [资源声明](../../../raw/current/scripts/prefabs/krampus_sack.lua#L1-L5)、[图标](../../../raw/current/scripts/prefabs/krampus_sack.lua#L44-L48) |

## 典型接入

### 调整坎普斯背包掉率

- 前提与选择：`modmain.lua`，模组环境默认无 `__index=_G`，`SetSharedLootTable` 是游戏全局函数而非模组环境导入项，须经 `GLOBAL` 调用；`TUNING` 等常用项由环境直接提供。整表按名覆盖，须重复原表全部条目。
- 生效与撤销：掉落生成时按 `chanceloottable` 名字现查 `LootTables`，替换后对已存在与新生成的坎普斯同样生效；该表仅服务端消费。撤销须恢复原表或重载。
- 原版依据：掉落表与掷点见上文[关联机制导航](#关联机制导航)；各条目独立掷点，1% 与必掉条目互不影响；共享表里的非必掉项都会先过 `GetChance`，所以 1% 是基础值，幸运高的击杀者实际更高。[掷点实现](../../../raw/current/scripts/components/lootdropper.lua#L213-L223)、[全局注册](../../../raw/current/scripts/components/lootdropper.lua#L22-L24)

```lua
-- modmain.lua：把坎普斯背包在坎普斯掉落表中的概率提到 10%
GLOBAL.SetSharedLootTable("krampus",
{
    { "monstermeat",  1.0 },
    { "charcoal",     1.0 },
    { "charcoal",     1.0 },
    { "krampus_sack", 0.1 },
})
```

- 调整与验证：共享表概率为每次掉落判定的独立掷点，且写入的数值是基础值（仍会经 `GetChance` 修正）；要动克劳斯袋那条 10%（`math.random()`，不吃幸运）需 post-init `klaussackloot` 组件（[滚点](../../../raw/current/scripts/components/klaussackloot.lua#L107-L113)），本示例未覆盖。示例未在游戏内运行，机制依据上文源码审阅。

## 常见陷阱与限制

- **误区：挂着 waterproofer 就防水**：`effectiveness=0`，`GetWaterproofness` 加 0、酸雨伤害吸收要求 `> 0`，对穿戴者均无效；标签只是“组件存在”的 pristine 优化（[注释](../../../raw/current/scripts/prefabs/krampus_sack.lua#L54-L55)）。要防水请改装备数值或换其他装备。
- **误区：作祟会偷出包内物品**：`MakeHauntableLaunchAndDropFirstItem` 的掉物分支在本 build 被注释，实际只发射背包本体。[helper](../../../raw/current/scripts/standardcomponents.lua#L1486-L1525)
- **边界：无减速**：本版 equippable 回调只处理外观与容器开关，无任何移速修改；背包装备不改变移速（[回调](../../../raw/current/scripts/prefabs/krampus_sack.lua#L7-L32)）。
- **边界：获取仅限掉落**：`recipes.lua` 无 `krampus_sack` 配方条目；当前检索范围（`prefabs/`、`recipes.lua`、`klaussackloot.lua`）内获取入口只有坎普斯共享表 1%（基础值，经 `GetChance`）与克劳斯袋 10%（`math.random()`）两处。

## 验证记录

- 源码审阅：2026-11-07，Steam buildid 26335394。全文实读 `prefabs/krampus_sack.lua`（89 行）；核验注册名、BODY 槽、14 格布局（containers.lua L1057-L1079）、两条获取链（krampus.lua L18-L24/L111-L112、klaussackloot.lua L107-L113 及挂载/消费）、waterproofer(0) 全部消费端、作祟／漂浮／foley helper、皮肤注册与共用 init、端口分界。
- 名称核验：`chinese_s.po` 以 python 按 `msgctxt "STRINGS.NAMES.KRAMPUS_SACK"` 定位（L344445-L344447）；四款皮肤名键同法核验（L398870-L398890 区段）。
- 游戏运行：未运行；未验证实际掉率表现、皮肤贴图内容、假人展示与装备栏扩展模组兼容。
- 结构检查：`validate_page.py`（含 `--check-indexes`）通过；standard 档上限 240 行 / 32768 字节。

返回 [[reference/prefabs/storage-heavy-index|背包收纳与重物搬运]]、[[reference/prefabs/index|Prefab 索引]]。
