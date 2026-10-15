---
title: "暖石（heatrock）"
created: "2026-10-29"
updated: "2026-10-29"
type: prefab
summary: "原版暖石的温差分档、携带与地面换热、耐久和随持有者移动的灯光接线入口。"
aliases: ["heatrock", "暖石", "Thermal Stone", "暖石发光", "暖石耐久", "携带取暖", "暖石降温"]
tags: [prefab, item, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["prefabs/heatrock.lua", "components/heater.lua", "components/temperature.lua", "components/inventory.lua", "standardcomponents.lua", "tuning.lua", "entityscript.lua", "languages/chinese_s.po", "prefabskin.lua", "prefabs/skinprefabs.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 暖石（heatrock）

## 快速判断

- 职责与适用：以自身温度相对当地环境的差值分五档，分别驱动换热、外观和耐久；适合查原版温度物品的组装方式。[分档与输出](../../../raw/current/scripts/prefabs/heatrock.lua#L38-L64)
- 优先使用：只改自身温度查 [[reference/components/temperature#让已有暖石自身保持35度并在读档后重新赋温]]；改携带换热查 [[reference/components/heater#让暖石只在携带时按35度双向换热]]；下文说明两者如何连接。
- 重要副作用：最高温档启灯，冷热往返满足条件时扣 fueled 百分比；灯光和耐久都不等同于 heater 输出。[外观](../../../raw/current/scripts/prefabs/heatrock.lua#L78-L92)、[耐久条件](../../../raw/current/scripts/prefabs/heatrock.lua#L115-L151)
- 修改边界：温度、热源、耐久和灯光归属管理在主模拟构造；公共端先配置外观、标签与漂浮。未提供恒温、永久发光或人物体温锁定的组合工程。[工厂](../../../raw/current/scripts/prefabs/heatrock.lua#L196-L270)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `heatrock` | `fn` → 公共动画／物理／标签／floater → SetPristine → 主模拟组件和回调 | 初始 current 取世界温度，但图标先设第 3 档；之后监听 temperaturedelta 更新。[初始化](../../../raw/current/scripts/prefabs/heatrock.lua#L237-L270)、[注册](../../../raw/current/scripts/prefabs/heatrock.lua#L301-L302) |
| `heatrocklight` | `lightfn` → 公共 Light／Network → 主模拟 persists=false | 独立网络灯实体，默认关闭，由暖石显式生成和移除；不是另一种可拾取暖石。[灯工厂](../../../raw/current/scripts/prefabs/heatrock.lua#L275-L298)、[生成](../../../raw/current/scripts/prefabs/heatrock.lua#L257-L260)、[清理](../../../raw/current/scripts/prefabs/heatrock.lua#L31-L36) |

名称键为 `STRINGS.NAMES.HEATROCK`，当前简中译为“暖石”、英文为 Thermal Stone。[本地化条目](../../../raw/current/scripts/languages/chinese_s.po#L343145-L343147)；本工厂没有名称覆盖，默认注册名映射见 [SetPrefabName](../../../raw/current/scripts/entityscript.lua#L840-L844)。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| temperature | 设冬夏 inherent insulation，忽略带 heatrock 标签的附近热源 | 主模拟；自身变化与供热分开 | [[reference/components/temperature#自然温度绝热与容器如何选择]]；[配置](../../../raw/current/scripts/prefabs/heatrock.lua#L237-L241) |
| heater | heatfn 和 carriedheatfn 共用 HeatFn；携带倍率配置为 2.1；没有配置 equippedheatfn | 主模拟；地面与携带通道独立消费 | [[reference/components/heater#携带范围与地面热源的分工]]；[配置](../../../raw/current/scripts/prefabs/heatrock.lua#L243-L247)、[倍率](../../../raw/current/scripts/tuning.lua#L2292) |
| inventoryitem／inspectable／tradable | 可拾取、切换图标、按档返回检视状态，rocktribute=12 | 主模拟；不挂 equippable | [[reference/components/inventoryitem#关键接口]]；[[reference/components/inspectable#关键接口]]；[[reference/components/tradable#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/heatrock.lua#L229-L235) |
| fueled | USAGE、初始 100，耗尽移除暖石；由温度回调扣比例 | 主模拟；没有在工厂启动持续消耗或接收燃料 | [[reference/components/fueled#燃料与使用耐久的选择]]；[配置](../../../raw/current/scripts/prefabs/heatrock.lua#L249-L252) |
| floater／hauntable | MakeInventoryFloatable 添加漂浮；MakeHauntableLaunchAndSmash 添加作祟 | floater 在端分界前；作祟在主模拟 | [[reference/components/floater#关键接口]]；[[reference/components/hauntable#按效果选工厂]]；[调用](../../../raw/current/scripts/prefabs/heatrock.lua#L218)、[调用](../../../raw/current/scripts/prefabs/heatrock.lua#L266)、[helper](../../../raw/current/scripts/standardcomponents.lua#L1772-L1790) |
| temperaturedelta／归属事件 | 前者更新分档；沿 inventoryitem.owner 链监听 onputininventory／ondropped，调整灯父实体 | 主模拟；嵌套 owner 链仅用于灯光归属 | [温度监听](../../../raw/current/scripts/prefabs/heatrock.lua#L254-L255)、[归属链](../../../raw/current/scripts/prefabs/heatrock.lua#L153-L194) |
| 动画／图标／Light | heat_rock bank/build，动画 1～5，heat_rock1～5 图标；第 5 档启 bloom 和灯光 | 皮肤图标使用 skinname 前缀；资源声明不等于逐个资源验证 | [资源声明](../../../raw/current/scripts/prefabs/heatrock.lua#L1-L9)、[外观更新](../../../raw/current/scripts/prefabs/heatrock.lua#L78-L92) |

本文件没有安装 SG 或 Brain；客户端外观使用网络实体，本文未逐项核验底层动画／灯光复制协议。

### 皮肤差异与附属灯

heatrock_fantasy／heatrock_fire／heatrock_planet 共用 heatrock_init_fn，只设置皮肤 build 和当前档位图标；clear 恢复原 build／图标。所读三个定义及该初始化路径没有额外生成皮肤专属实体，仍复用本页的 heatrocklight 工厂与灯光管理。[皮肤定义](../../../raw/current/scripts/prefabs/skinprefabs.lua#L6567-L6596)、[初始化与恢复](../../../raw/current/scripts/prefabskin.lua#L3369-L3380)

## 典型接入

### 暖石何时切换温度档、发热或制冷

GetRangeForTemperature 使用严格大于比较，令 Δ 为自身温度减 GetLocalTemperature(inst)，得到下表；HeatFn 每次求值时更新热／冷开关。[算法](../../../raw/current/scripts/prefabs/heatrock.lua#L38-L64)

| 档位 | Δ 范围 | 换热输出值／方向 | 外观 |
|---|---|---|---|
| 1 | Δ≤-30 | -10，只制冷 | FROZEN |
| 2 | -30<Δ≤-10 | 10，只制冷 | COLD |
| 3 | -10<Δ≤10 | 25，关闭两方向 | 中性 |
| 4 | 10<Δ≤30 | 40，只供热 | WARM |
| 5 | Δ>30 | 60，只供热 | HOT，启 bloom 和灯光，灯强随温差渐变 |

第 5 档启灯后，AdjustLighting 另设无量纲灯强 `clamp(0.5*baseline/brightline,0,0.5)`：baseline=Δ−第 4 个阈值，brightline=该阈值+20，本版阈值为 30、brightline=50；其他档灯强设为 0。因此“启灯”与强度渐变同时存在，不是固定强度的开关灯。[阈值](../../../raw/current/scripts/prefabs/heatrock.lua#L39)、[灯强与调用](../../../raw/current/scripts/prefabs/heatrock.lua#L94-L120)

输出是 heater 的输入值，不是人物立刻达到的温度。人物还按自身温度与方向检查，汇总环境和多个来源后处理绝热与 dt。[携带汇总](../../../raw/current/scripts/components/temperature.lua#L370-L413)、[最终变化](../../../raw/current/scripts/components/temperature.lua#L487-L514)

### 主栏、背包、地面与灯光归属

人物 temperature 分别读取装备槽、主栏、GetOverflowContainer 的槽位以及鼠标 activeitem；背包溢出容器由可打开的 BODY 槽物品提供。不会递归任意嵌套容器。暖石没有装备通道配置，不能将 carriedheat 自动当作 equippedheat。[消费范围](../../../raw/current/scripts/components/temperature.lua#L370-L413)、[BODY 溢出容器](../../../raw/current/scripts/components/inventory.lua#L1459-L1467)、[getter 分工](../../../raw/current/scripts/components/heater.lua#L64-L78)

地面模式通过附近热源扫描；携带时走上述槽位路径。暖石自身忽略其他 heatrock 标签热源，不表示持有者忽略暖石。[自身过滤](../../../raw/current/scripts/prefabs/heatrock.lua#L241)、[地面热源消费](../../../raw/current/scripts/components/temperature.lua#L434-L456)

灯光沿 owner 链挂到最外层对象；最外层带 pocketdimension_container 或 buried 标签时改挂回暖石并移出场景。这条递归关系不能推导成所有嵌套包都能供热。[灯归属](../../../raw/current/scripts/prefabs/heatrock.lua#L153-L194)

### 热循环耐久与读档

TemperatureChange 在最冷／最热档记录 lowTemp／highTemp，变档后需 data.hasrate 为真，且保留的冷热极值与回到中性档条件匹配，才减去 `1 / HEATROCK_NUMUSES` 的 fueled 百分比；本版 NUMUSES=8。不是每帧、每次温度事件或每次加热都扣一次。[扣减与极值](../../../raw/current/scripts/prefabs/heatrock.lua#L115-L151)、[次数配置](../../../raw/current/scripts/tuning.lua#L1733)

实体 OnSave／OnLoad 保存恢复冷热极值，温度和 fueled 各自走组件存档；本页只核验实体极值回调，不展开通用组件存档机制。[实体回调](../../../raw/current/scripts/prefabs/heatrock.lua#L11-L29)

## 常见陷阱与限制

- **误区：固定自身温度就固定供热、图标和亮度**：HeatFn 用当时环境重新求档，外观更新则由 temperaturedelta 驱动；temperature 的 SetTemp 会提前退出自然更新。自行锁温须分别考虑输出与外观刷新。[输出](../../../raw/current/scripts/prefabs/heatrock.lua#L54-L64)、[亮度](../../../raw/current/scripts/prefabs/heatrock.lua#L105-L112)、[更新门禁](../../../raw/current/scripts/components/temperature.lua#L311-L320)
- **误区：helper 名带 Smash 就会作祟碎裂**：本版 MakeHauntableLaunchAndSmash 的碎裂分支被注释，实际保留发射；不能按函数名承诺效果。[helper 实现](../../../raw/current/scripts/standardcomponents.lua#L1071-L1105)
- **误区：放进冰箱仍按周围热源升降温**：直接 owner 带 fridge 且不带 nocool 时，自身 temperature 进入专门冷却分支；入口见 [[reference/prefabs/icebox#暖石冷却与食物保鲜]]。[冷却条件](../../../raw/current/scripts/components/temperature.lua#L328-L340)

## 验证记录

- 源码审阅：2026-10-29，Steam buildid 26230496；完整读取 heatrock.lua，核验分档、耐久、归属链、两个工厂、必要 helper、温度携带消费者及名称条目。关联组件仅按本页需要核验，不声称审校所有容器或全部热源。
- 皮肤补核：同日读取上述三种皮肤定义及共用 init／clear，仅覆盖已声明路径；附属灯保留同页说明，不另建简单 FX 页。
- 游戏运行：未运行；多层转移、口袋容器、存档重载、锁温与环境变化后的灯光／耐久表现未实测。未核验资源压缩包实际内容或皮肤全集；无新增 Lua 示例。

返回 [[reference/prefabs/index|Prefab 索引]]。
