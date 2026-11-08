---
title: "猪皮包（piggyback）"
created: "2026-11-07"
updated: "2026-11-08"
type: prefab
summary: "原版猪皮包：BODY 槽十二格背包装备、行走减速 0.9 及其消费链、自身防潮与不可燃烧的组装入口。"
aliases: ["piggyback", "猪皮包", "Piggyback", "猪皮背包"]
tags: [prefab, item, equipment]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/piggyback.lua", "containers.lua", "components/equippable.lua", "components/waterproofer.lua", "components/inventory.lua", "components/locomotor.lua", "standardcomponents.lua", "tuning.lua", "recipes.lua", "constants.lua", "mods.lua", "prefabskin.lua", "prefabs/skinprefabs.lua", "prefabs/ghostly_elixirs.lua", "prefabs/abigail_flower.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 猪皮包（piggyback）

## 快速判断

- 职责与适用：可拾取、装备在 BODY 槽的十二格（2×6）大容量背包；穿戴行走减速 0.9，无耐久消耗、无温度保温、不可燃烧。适合借鉴"大容量＋减速代价"的组装与装备速度多联乘的消费链。[工厂](../../../raw/current/scripts/prefabs/piggyback.lua#L34-L89)、[布局](../../../raw/current/scripts/containers.lua#L1082-L1103)
- 优先使用：改减速幅度或抵消减速查 [[reference/components/equippable#关键接口]] 与本页"调整猪皮包的行走减速"；改格子与接收条件查 [[reference/components/container#新容器的客户端布局]]；八格普通背包、穿脱开关与燃烧生命周期的通用组装见 [[reference/prefabs/backpack]]。
- 重要副作用：减速可被大力士"强壮"状态完全忽略，可被 vigorbuff 加回 0.25（上限 1）；waterproofer 有效值为 0，只让猪皮包自身不吃湿，不给穿戴者贡献防水；作祟只发射自身，不掉出内容物。[减速](../../../raw/current/scripts/components/locomotor.lua#L191-L199)、[作祟](../../../raw/current/scripts/standardcomponents.lua#L1507-L1525)
- 修改边界：单一注册名，无运行时形态切换；本页不覆盖 icepack、krampus_sack、candybag、seedpouch、spicepack 等同族 BODY 容器（源码入口见关联导航）。皮肤为纯换装，无跟随 FX。[皮肤](../../../raw/current/scripts/prefabskin.lua#L181-L182)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `piggyback` | `fn` → 公共端：Transform／AnimState／SoundEmitter／MiniMapEntity／Network、inventory physics、bank `piggyback`＋build `swap_piggyback`、漂浮 → SetPristine → 主模拟：inspectable、inventoryitem、equippable、waterproofer、container、作祟 helper | 单一注册名 `Prefab("piggyback", fn, assets)`；无 SG／Brain、无 prefabs 依赖表、无耐久组件。`Prefab` 注册与主模拟分界见[工厂](../../../raw/current/scripts/prefabs/piggyback.lua#L34-L89)、[注册](../../../raw/current/scripts/prefabs/piggyback.lua#L91) |

名称键 `STRINGS.NAMES.PIGGYBACK` 对应"猪皮包"／Piggyback。[简中条目](../../../raw/current/scripts/languages/chinese_s.po#L346909-L346912)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| inspectable／inventoryitem | 检视、拾取；`cangoincontainer=false` 禁止被收入其他容器 | 主模拟；不阻止放进人物主物品栏 | [[reference/components/inventoryitem#关键接口]]；[配置](../../../raw/current/scripts/prefabs/piggyback.lua#L67-L70) |
| equippable | BODY 槽；onequip／onunequip／onequiptomodel 绑定；`walkspeedmult = TUNING.PIGGYBACK_SPEED_MULT`（0.9） | 主模拟组件；减速由 `equippable` 的 `onwalkspeedmult` watcher 写进 **`inventoryitem_replica`**（`classified.walkspeedmult`），客户端从那里读，不是查 `equippable` Replica；该网络字段与 `saddler` 共用，同一 prefab 不能同时挂两者 | [[reference/components/equippable#关键接口]]；[绑定与减速](../../../raw/current/scripts/prefabs/piggyback.lua#L72-L78)、[watcher](../../../raw/current/scripts/components/equippable.lua#L7-L19)、[Replica 读取](../../../raw/current/scripts/components/inventoryitem_replica.lua#L406-L421) |
| container | `WidgetSetup("piggyback")`：2×6 十二格、bank／build `ui_piggyback_2x6`、`issidewidget=true`、type `"pack"`、`openlimit=1`、无 itemtestfn（接收一切） | 主模拟组件；布局定义客户端 Replica 共用；穿戴开包见 onequip | [[reference/components/container#新容器的客户端布局]]；[布局](../../../raw/current/scripts/containers.lua#L1082-L1103)、[挂载](../../../raw/current/scripts/prefabs/piggyback.lua#L83-L84) |
| waterproofer | `SetEffectiveness(0)`：有效值为 0；组件构造时 `EnableMoisture(false)` 让物品自身不吃湿 | 主模拟；穿着者防水总和来自装备位 waterproofer 求和，猪皮包贡献 0 | [[reference/components/waterproofer#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/piggyback.lua#L80-L81)、[构造](../../../raw/current/scripts/components/waterproofer.lua#L1-L12)、[求和](../../../raw/current/scripts/components/inventory.lua#L2714-L2734) |
| hauntable | `MakeHauntableLaunchAndDropFirstItem`：作祟以固定概率发射自身 | 主模拟；掉出第一件内容物的分支在本版被注释 | [[reference/components/hauntable#按效果选工厂]]；[helper](../../../raw/current/scripts/standardcomponents.lua#L1486-L1530) |
| floater | `MakeInventoryFloatable(inst, "small", 0.1, 0.85, nil, nil, swap_data)`（`swap_data = {bank = "piggyback", anim = "anim"}`） | 公共端 | [[reference/components/floater#关键接口]]；[调用](../../../raw/current/scripts/prefabs/piggyback.lua#L59) |
| API／数据：`TUNING.PIGGYBACK_SPEED_MULT` | 减速配置值 0.9；服务端 locomotor 的 `ServerGetSpeedMultiplier` 遍历装备位联乘 `GetWalkSpeedMult()`；客户端 `ClientGetSpeedMultiplier` 从每个装备的 **`inventoryitem:GetWalkSpeedMult()`** 取（值来自 `inventoryitem_replica` 的 `classified.walkspeedmult`，公式与 `Equippable::GetWalkSpeedMult` 保持同步），与 `equippable` Replica 无关 | 主模拟读分量；客户端预测读 `inventoryitem` Replica | [[reference/components/equippable#关键接口]]；[调值](../../../raw/current/scripts/tuning.lua#L416)、[服务端消费](../../../raw/current/scripts/components/locomotor.lua#L179-L201)、[客户端消费](../../../raw/current/scripts/components/locomotor.lua#L207-L236) |
| API／数据：`vigorbuff` 标签 | 装备减速小于 1 且持有者带 vigorbuff 时加回 0.25（上限 1）；标签由幽灵药水 ghostlyelixir_speed 施加到玩家 | 持有者状态；仅 `isequipped` 且有 owner 时生效 | [判定](../../../raw/current/scripts/components/equippable.lua#L126-L144)、[施标](../../../raw/current/scripts/prefabs/ghostly_elixirs.lua#L210-L218) |
| API／数据：`backpack` 标签 | 原始态 `AddTag("backpack")`，与普通背包等家族共用；消费者用它识别"宿主是背包类容器"（如阿比盖尔之花据此跳过向宿主挂事件） | 公共端标签；家族成员见下 | [[reference/prefabs/backpack]]；[加标](../../../raw/current/scripts/prefabs/piggyback.lua#L53)、[消费示例](../../../raw/current/scripts/prefabs/abigail_flower.lua#L136-L158) |
| API／数据：溢出容器 | BODY 槽可打开容器即玩家溢出容器（`GetOverflowContainer`） | 主模拟；canbeopened=false 时失效 | [[reference/prefabs/backpack#典型接入]]；[选择](../../../raw/current/scripts/components/inventory.lua#L1459-L1466) |
| 资源 | anim `piggyback.zip`、`swap_piggyback.zip`、`ui_piggyback_2x6.zip`；小地图图标 `piggyback.png`；foleysound `dontstarve/movement/foley/backpack` | 声明与使用分别核验；未验证压缩包内容 | [资产](../../../raw/current/scripts/prefabs/piggyback.lua#L1-L6)、[图标与音效](../../../raw/current/scripts/prefabs/piggyback.lua#L45-L51) |

配方：`Recipe2("piggyback", {pigskin×4, silk×6, rope×2}, TECH.SCIENCE_TWO)`（二级科学档，`{SCIENCE=2}`）。[配方](../../../raw/current/scripts/recipes.lua#L517)、[技术档](../../../raw/current/scripts/constants.lua#L1245-L1247)

### 同族 BODY 容器与皮肤差异

同族均加 `backpack` 标签：backpack（L251）、icepack（L56）、krampus_sack（L52）、candybag（L67）、seedpouch（L51）、spicepack（L87）。已有页互链 [[reference/prefabs/backpack]]、[[reference/prefabs/candybag|糖果袋]]、[[reference/prefabs/krampus_sack|坎普斯背包]]、[[reference/prefabs/icepack|保鲜背包]]、[[reference/prefabs/pouch-bags|小袋族]]；其余差异在各自页展开。

| 皮肤／条件 | 额外关系与生成／清理 | 入口与覆盖 |
|---|---|---|
| 共用 piggyback_init_fn／clear_fn → basic_init_fn／basic_clear_fn | 仅 `AnimState:SetSkin(build, "swap_piggyback")` 换装、换物品栏图名与漂浮动画刷新；无跟随实体、无额外组件 | [皮肤接线](../../../raw/current/scripts/prefabskin.lua#L181-L182)、[basic 实现](../../../raw/current/scripts/prefabskin.lua#L106-L142) |
| piggyback_elephant（大象背包）、piggyback_nose（小猪背包）、piggyback_shadow（血色背包） | 三件 `type="item"`、Elegant＋Woven 品质的皮肤，skin_tags 分别含 MIGHTY／YOTC／SHADOW，仅走共用换装路径 | [定义](../../../raw/current/scripts/prefabs/skinprefabs.lua#L9216-L9247)、[名称](../../../raw/current/scripts/languages/chinese_s.po#L404144-L404157) |

## 典型接入

### 调整猪皮包的行走减速

- 前提与选择：modmain 中用 AddPrefabPostInit 在实体构造后赋值；equippable 组件只在主模拟实例上存在，客户端数值由 watcher 同步到 Replica，无需客户端再赋值。
- 生效与撤销：构造时一次性生效，随实体存续；改回 1.0 即撤销减速。
- 原版依据：减速读自 equippable 的 watcher 字段并进入 locomotor 联乘，见上文导航。

```lua
-- modmain.lua：把猪皮包减速改轻（主模拟赋值，Replica 自动同步）
AddPrefabPostInit("piggyback", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    inst.components.equippable.walkspeedmult = 0.95
end)
```

- 调整与验证：mod 环境只直供 `TUNING` 与 `GLOBAL`，`TheWorld` 须加 `GLOBAL.` 前缀；速度表现还受地面倍率、坐骑与其他装备联乘影响，单独验证时应清空变量。[环境与联乘](../../../raw/current/scripts/mods.lua#L258-L290)、[联乘入口](../../../raw/current/scripts/components/locomotor.lua#L180-L205)

### 穿戴外观与开包必须成对配置

onequip 覆盖角色 `swap_body` 符号后 `container:Open(owner)`；onunequip 清符号并 `Close(owner)`；onequiptomodel 只关闭容器。皮肤路径用 `OverrideItemSkinSymbol` 并推送 equipskinneditem 事件。缺任一半回调，都会出现"背包看得见却打不开"或残留外观符号。[穿戴](../../../raw/current/scripts/prefabs/piggyback.lua#L8-L19)、[脱下与模型](../../../raw/current/scripts/prefabs/piggyback.lua#L21-L32)

与普通背包不同，猪皮包皮肤不走 swap FX 实体：直接 `SetSkin` 换装本体，装备外观靠 onequip 里的 `GetSkinBuild()` 分支覆盖。需要事件驱动变色等复杂皮肤时，参考 [[reference/prefabs/backpack#关联机制导航]] 的 labrat 路线自行实现。

## 常见陷阱与限制

- **误区：猪皮包保温或保鲜**：本文件未挂 insulator、preserver 或任何耐久组件（finiteuses／fueled 均无）；waterproofer 有效值 0 也不提供防水。[组件全集](../../../raw/current/scripts/prefabs/piggyback.lua#L67-L86)
- **误区：猪皮包会像普通背包一样被烧掉**：工厂未调用 MakeSmallBurnable／MakeSmallPropagator，本体不可点燃；需要燃烧行为须自行补挂并接好掉物与灰烬清理。[工厂尾部](../../../raw/current/scripts/prefabs/piggyback.lua#L83-L86)、[普通背包燃烧对照](../../../raw/current/scripts/prefabs/backpack.lua#L201-L222)
- **误区：作祟掉出第一件内容物**：helper 名称里的 drop 分支在本版被注释，实际只有发射路径；与 [[reference/prefabs/backpack#常见陷阱与限制]] 所述同源。[helper](../../../raw/current/scripts/standardcomponents.lua#L1507-L1525)
- 边界：大力士"强壮"状态把小于 1 的装备减速按 1 处理（服务端查 mightiness 状态、客户端查标签），猪皮包在该状态下无减速；vigorbuff 只在减速小于 1 时加 0.25，不叠加超过 1。[服务端](../../../raw/current/scripts/components/locomotor.lua#L194-L196)、[客户端](../../../raw/current/scripts/components/locomotor.lua#L225-L227)、[加回](../../../raw/current/scripts/components/equippable.lua#L133-L135)

## 验证记录

- 源码审阅：2026-11-07，Steam buildid 26335394；完整读取 piggyback.lua，核验注册名、公共端／主模拟分界、equippable／waterproofer／container 配置、十二格布局、减速配置与两端消费链、作祟 helper、配方与技术档、三个皮肤及简中名称键；同批完整读取 candybag.lua 完成归属分诊（结论见交付报告，本页不覆盖）。
- 游戏运行：未运行；未验证皮肤实装视觉、资源压缩包内容、假人展示、多人装备同步与存档恢复。
- 结构检查：validate_page.py 通过（size_class standard，实际 99 行／13083 字节）；validate_source_links.py --strict 0 error 0 warning；validate_source_hashes.py check 通过（主代理已 record）。
- 2026-11-08 build 26474873：随 [P-1](../../updates/build-26474873/detail-prefabs.md) 修订浮水断言——工厂补第 7 参 `swap_data = {bank = "piggyback", anim = "anim"}`（`swap_bank` 为 nil 时 floater 走 `SetSwapData`，[helper](../../../raw/current/scripts/standardcomponents.lua#L1772-L1791)）。其余结论沿用。

返回 [[reference/prefabs/storage-heavy-index|背包收纳与重物搬运]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-08 build 26474873：本页随版本迁移升版（锚点迁移 10 处），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-11）。
