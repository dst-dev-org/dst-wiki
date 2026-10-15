---
title: "冰箱（icebox）"
created: "2026-10-29"
updated: "2026-10-29"
type: prefab
summary: "原版冰箱的九格过滤、fridge 标签消费者、暖石冷却、锤击掉物及放置预览接线入口。"
aliases: ["icebox", "冰箱", "Ice Box", "冰箱保鲜", "冰箱放暖石", "暖石冷却", "冰箱收纳条件"]
tags: [prefab, building, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["prefabs/icebox.lua", "prefabs/heatrock.lua", "containers.lua", "components/container.lua", "components/container_replica.lua", "components/perishable.lua", "components/temperature.lua", "standardcomponents.lua", "prefabutil.lua", "tuning.lua", "entityscript.lua", "languages/chinese_s.po", "prefabskin.lua", "prefabs/skinprefabs.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 冰箱（icebox）

## 快速判断

- 职责与适用：带 fridge 标签的九格建筑容器，格子与过滤由 containers.params.icebox 定义；保鲜／冷却由内部物品的组件读取标签，不是冰箱每帧遍历内容物赋值。[工厂](../../../raw/current/scripts/prefabs/icebox.lua#L59-L90)、[过滤](../../../raw/current/scripts/containers.lua#L1000-L1041)
- 优先使用：容器接线读本页；腐败倍率细节查 [[reference/components/perishable#关键接口]]，暖石换热查 [[reference/prefabs/heatrock#暖石何时切换温度档发热或制冷]]。
- 重要副作用：受锤击回调会 DropEverything 并关箱；完成拆除再掉落材料、生成金属坍塌 FX、移除冰箱。[锤击回调](../../../raw/current/scripts/prefabs/icebox.lua#L24-L38)
- 修改边界：公共端配置 fridge／structure、外观与雪覆盖；主模拟挂容器、掉落和工作组件。放置预览是独立非联网 placer，不是可储物冰箱。[工厂分界](../../../raw/current/scripts/prefabs/icebox.lua#L46-L97)、[placer 工厂](../../../raw/current/scripts/prefabutil.lua#L1-L31)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `icebox` | `fn` → 公共动画／图标／标签／雪覆盖 → SetPristine → 主模拟组件和事件 | 动画 bank icebox／build ice_box，循环闲置音；HAMMER 工作量为 2。[公共构造](../../../raw/current/scripts/prefabs/icebox.lua#L46-L74)、[组件配置](../../../raw/current/scripts/prefabs/icebox.lua#L76-L95) |
| `icebox_placer` | MakePlacer 返回独立 Prefab，用 closed 动画预览 | helper 添加 CLASSIFIED／NOCLICK／placer 标签和 placer 组件，不调用冰箱 fn。[注册](../../../raw/current/scripts/prefabs/icebox.lua#L100-L101)、[预览构造](../../../raw/current/scripts/prefabutil.lua#L1-L31)、[返回](../../../raw/current/scripts/prefabutil.lua#L61-L65) |

名称键 `STRINGS.NAMES.ICEBOX` 对应“冰箱”／Ice Box。[简中条目](../../../raw/current/scripts/languages/chinese_s.po#L343450-L343452)；本工厂无名称覆盖，默认取名见 [SetPrefabName](../../../raw/current/scripts/entityscript.lua#L840-L844)。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| inspectable／container | WidgetSetup("icebox")，自定义开关动画和声音，并跳过容器默认开关声 | 主模拟；布局和过滤必须客户端也可取得 | [[reference/components/inspectable#关键接口]]；[[reference/components/container#新容器的客户端布局]]；[挂载](../../../raw/current/scripts/prefabs/icebox.lua#L76-L82) |
| container Replica／classified | WidgetSetup 联动 Replica，Replica 主模拟创建 container_classified | 客户端读取共同 containers 定义 | [组件调用](../../../raw/current/scripts/components/container.lua#L86-L97)、[Replica 构造](../../../raw/current/scripts/components/container_replica.lua#L3-L54) |
| lootdropper／workable | HAMMER、工作量 2；受击和完成使用不同回调 | 主模拟；受击掉内容，完成掉材料和 FX | [[reference/components/lootdropper#关键接口]]；[[reference/components/workable#关键接口]]；[绑定](../../../raw/current/scripts/prefabs/icebox.lua#L85-L90) |
| lunarhailbuildup／hauntable | MakeSnowCovered 间接添加月雹堆积；AddHauntableDropItemOrWork 添加作祟 | 主模拟联网实体；作祟效果分支本版被注释 | [[reference/components/lunarhailbuildup#让建筑参与月雹堆积与清理]]；[[reference/components/hauntable#按效果选工厂]]；[调用](../../../raw/current/scripts/prefabs/icebox.lua#L92-L95)、[月雹 helper](../../../raw/current/scripts/standardcomponents.lua#L842-L855)、[挂载](../../../raw/current/scripts/standardcomponents.lua#L887-L891) |
| placer | MakePlacer 给 icebox_placer 添加放置组件 | 仅非联网预览实体，不挂在可储物的 icebox 主体上 | [[reference/components/placer#关键接口]]；[helper 挂载](../../../raw/current/scripts/prefabutil.lua#L1-L31) |
| perishable／temperature 消费者 | 内部物品以直接 owner 的 fridge 标签选择腐败倍率／冷却路径 | 两者并未被本冰箱工厂挂在冰箱上；也没有在冰箱上挂 heater／preserver | [[reference/components/perishable#关键接口]]；[[reference/components/temperature#自然温度绝热与容器如何选择]]；[腐败消费](../../../raw/current/scripts/components/perishable.lua#L68-L91)、[冷却消费](../../../raw/current/scripts/components/temperature.lua#L328-L340) |
| collapse_small／资源与事件 | onhammered 显式生成 FX 并 SetMaterial("metal")；onbuilt 播 place 与建造声；声明 ice_box 与九格 UI 动画包 | 不从依赖数组推定其他生成；未检查资源包内容 | [拆除](../../../raw/current/scripts/prefabs/icebox.lua#L24-L31)、[建造](../../../raw/current/scripts/prefabs/icebox.lua#L40-L44)、[资源声明](../../../raw/current/scripts/prefabs/icebox.lua#L3-L12) |

本文件没有安装 SG／Brain；放置由 placer 处理，容器交互由组件与宿主回调处理。

### 皮肤差异与开箱特效

皮肤 fx_prefab 表在 CreatePrefabSkin 时进入 SKIN_FX_PREFAB。icebox_init_fn 以第 1 项为 frost_fx、第 2 项为 open_fx；placer 只设皮肤外观并提前返回，不安装开箱监听。[映射保存](../../../raw/current/scripts/prefabskin.lua#L4094-L4096)、[初始化](../../../raw/current/scripts/prefabskin.lua#L3704-L3730)

| 皮肤 | 额外实体与条件 | 生成／清理与入口 |
|---|---|---|
| icebox_coffin | open_fx=icebox_coffin_bat_fx；无 frost_fx | onopen 时按 1.3 秒节流生成效果；不存入 _frostfx。[定义](../../../raw/current/scripts/prefabs/skinprefabs.lua#L6955-L6966)、[生成](../../../raw/current/scripts/prefabskin.lua#L3678-L3695) |
| icebox_crystal／kitchen／porcelain／victorian | 各自的 icebox_crystal_fx／icebox_kitchen_fx／icebox_porcelain_fx／icebox_victorian_frost_fx 作为 frost_fx | 开箱时若无实例则生成并保留 _frostfx；关箱、移除和清皮肤时调用 Kill。[定义](../../../raw/current/scripts/prefabs/skinprefabs.lua#L6968-L7017)、[关闭](../../../raw/current/scripts/prefabskin.lua#L3697-L3702)、[清皮肤](../../../raw/current/scripts/prefabskin.lua#L3732-L3740) |

上述 FX Wiki 待建；本表给出注册标识与实际生成／清理入口，暂不复制粒子实现或推定其自行销毁行为。未逐读 FX 工厂；不据名含 fx 断言绝无数据侧效果。

## 典型接入

### 哪些物品能放进冰箱

九格由三行三列 slotpos 生成。过滤先接受带 icebox_valid 或 inventoryitemtemperature 标签的物品；其余物品必须带 fresh／stale／spoiled 之一、不能是 smallcreature，并至少有一个 edible_食物类型标签。不是任意带 perishable 的实体都能放，也不能只按现实中的“食物”名称判断。[格子与过滤](../../../raw/current/scripts/containers.lua#L1000-L1041)

暖石公共构造显式加 icebox_valid，因此走首个允许分支；普通背包自身先禁止入容器，见 [[reference/prefabs/backpack#常见陷阱与限制]]。[暖石资格](../../../raw/current/scripts/prefabs/heatrock.lua#L209-L210)

### 暖石冷却与食物保鲜

食物 perishable 读取直接 owner：有 preserver 时先用其倍率，否则 fridge 分支对 frozen 且容器不带 nocool／lowcool 的物品取 PERISH_COLD_FROZEN_MULT，其他取 PERISH_FRIDGE_MULT。本版两值分别是 0 和 0.5；后续环境分支仍可能改变总效果，不能把基础倍率当固定总保质期。[消费条件](../../../raw/current/scripts/components/perishable.lua#L68-L91)、[倍率配置](../../../raw/current/scripts/tuning.lua#L2077-L2079)

暖石自身 temperature 发现直接 owner 带 fridge 且不带 nocool 后，采用冷却速率，并以当地环境决定最低温限制；不是立即设成固定零度，也不是让冰箱影响附近所有人物体温。[冷却路径](../../../raw/current/scripts/components/temperature.lua#L328-L340)、[应用 dt](../../../raw/current/scripts/components/temperature.lua#L512-L514)

### 改开关或拆除表现时保留内容清理

开／关回调只处理动画声音；onhit 掉全部内容、播 hit 后回 closed 并 Close；onhammered 掉 loot 与全部内容，再播金属坍塌、Remove。扩展时保留对应原回调，不把受击与完成合并成同一条清理。[开关](../../../raw/current/scripts/prefabs/icebox.lua#L14-L22)、[锤击](../../../raw/current/scripts/prefabs/icebox.lua#L24-L38)

## 常见陷阱与限制

- **误区：fridge 标签自动提供格子与收纳权限**：标签供物品组件消费；格子、过滤与开关另由 container 和 containers 配置，不能只给空实体贴标签就替代原工厂。[组件配置](../../../raw/current/scripts/prefabs/icebox.lua#L76-L90)
- **误区：冰箱有 hauntable 就会掉物或被幽灵锤坏**：当前 AddHauntableDropItemOrWork 的掉物／工作分支均注释，回调返回 false。[helper 实现](../../../raw/current/scripts/standardcomponents.lua#L1554-L1579)
- **误区：MakeSnowCovered 仅负责显示雪**：主模拟联网实体还会安装 lunarhailbuildup 并监听状态变化，扩展或替换 helper 时须保留该关系。[helper 链](../../../raw/current/scripts/standardcomponents.lua#L842-L855)、[安装与监听](../../../raw/current/scripts/standardcomponents.lua#L887-L891)

## 验证记录

- 源码审阅：2026-10-29，Steam buildid 26230496；完整读取 icebox.lua，核验九格布局和过滤、container／Replica、fridge 的两条必要消费者、placer、雪覆盖／月雹和作祟 helper、名称键。暖石资格另读其公共初始化。
- 皮肤补核：同日核验五种皮肤定义、fx_prefab 映射及冰箱 init／clear／open／close，区分开箱效果、持续霜气和 placer 外观；FX 工厂内部未覆盖。
- 游戏运行：未运行；未实测温度环境组合、月雹清理、多人开关、锤击时序及皮肤资源，未展开所有食物腐败修正或建筑配方。无新增 Lua 示例。

返回 [[reference/prefabs/index|Prefab 索引]]。
