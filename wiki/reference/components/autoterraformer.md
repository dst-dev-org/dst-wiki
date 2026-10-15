---
title: "autoterraformer"
created: "2026-10-23"
updated: "2026-10-28"
type: component
summary: "让带容器的实体随位置自动铲地或铺设槽内地皮，区分下层地块、耗材补槽与装备启停。"
aliases: ["自动铺地皮", "蚁狮帽铲地", "走路换地皮", "自动整地", "autoterraformer"]
tags: [component, equipment, callback, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/autoterraformer.lua", "components/container.lua", "components/finiteuses.lua", "prefabs/hats.lua", "containers.lua", "entityscript.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# autoterraformer

## 快速判断

- 职责与适用：按**宿主实体当前位置**处理地图格，用容器第 1 槽物品的 `tile` 决定铺地目标；原版宿主是 `antlionhat`。只需一次动作铲地优先 [[reference/components/terraformer]]。[组件入口](../../../raw/current/scripts/components/autoterraformer.lua#L57-L90)、[逐帧坐标](../../../raw/current/scripts/components/autoterraformer.lua#L115-L131)、[帽子工厂](../../../raw/current/scripts/prefabs/hats.lua#L2415-L2437)
- 优先使用：先添加并配置 [[reference/components/container]]，再添加本组件；由装备或业务生命周期调用 `StartTerraforming`／`StopTerraforming`，完成效果设 `onfinishterraformingfn`。[构造断言](../../../raw/current/scripts/components/autoterraformer.lua#L1-L9)、[原版启停](../../../raw/current/scripts/prefabs/hats.lua#L2371-L2391)
- 重要副作用：成功后宿主收到无载荷 `onterraform`，可选 `finiteuses` 扣 1 次，最后才调用完成回调；铲地分支还返还旧地皮、使同格 `soil` 坍塌。[完成顺序](../../../raw/current/scripts/components/autoterraformer.lua#L12-L20)、[铲地后处理](../../../raw/current/scripts/components/autoterraformer.lua#L83-L103)、[默认扣次](../../../raw/current/scripts/components/finiteuses.lua#L85-L86)
- 修改边界：原版仅服务端添加；组件不做装备、持有者或体力资格判断，也不自建容器、动作、存档和特效。地图资格与世界事件沿用 [[reference/components/terraformer#动作地图门禁与事件分工|动作、地图门禁与事件分工]]，不能靠完成回调取消已修改地块。[宿主端边界](../../../raw/current/scripts/prefabs/hats.lua#L2421-L2437)、[先改图后完成](../../../raw/current/scripts/components/autoterraformer.lua#L72-L75)

## 关键接口

| 接口／字段 | 约定 | 条件、作用与证据 |
|---|---|---|
| `StartTerraforming()` | 无返回约定 | 清空上次格坐标和重复延迟，注册组件更新；并非立即铲地。[实现](../../../raw/current/scripts/components/autoterraformer.lua#L106-L108) |
| `StopTerraforming()` | 无返回约定 | 停止组件更新；不回滚地块、不关闭容器。[实现](../../../raw/current/scripts/components/autoterraformer.lua#L111-L112) |
| `repeat_tile_delay` | 秒；默认 0.25 | 仅一次铲地露出下层时，在同格等待后重试；不是每格操作的通用冷却。[初始化](../../../raw/current/scripts/components/autoterraformer.lua#L5)、[默认值](../../../raw/current/scripts/tuning.lua#L6853)、[更新条件](../../../raw/current/scripts/components/autoterraformer.lua#L119-L131) |
| `DoTerraform(px, py, pz, x, y)` | 前三参世界坐标，后两参地图格坐标 | 处理一次，露出下层成功返回 true；其他成功／无操作通常返回 nil，不能用返回值判断总体成功。[完整分支](../../../raw/current/scripts/components/autoterraformer.lua#L57-L103) |
| `onfinishterraformingfn` | `fn(inst, world_x, world_y, world_z)` | 地图已改、事件和耐久已处理后调用；返回值被忽略。[实现](../../../raw/current/scripts/components/autoterraformer.lua#L12-L20) |
| `container` | 构造时缓存的组件实例 | 不是每次读取 `inst.components.container`；运行中替换容器组件不会自动重绑。[缓存与取物](../../../raw/current/scripts/components/autoterraformer.lua#L2-L9)、[使用](../../../raw/current/scripts/components/autoterraformer.lua#L61) |

### 地皮资格与消费顺序

1. 第 1 槽有物品且有 `tile` 字段时取该值；与当前地块相同则直接跳过。组件本体不检查 `groundtile` 标签，原版容器的 `itemtestfn` 才要求标签和 `tile` 同时满足。[读取与跳过](../../../raw/current/scripts/components/autoterraformer.lua#L60-L68)、[容器限制](../../../raw/current/scripts/containers.lua#L1282-L1283)
2. 有目标地块且 `CanPlaceTurfAtPoint` 允许：移出一件耗材并删除，然后 `SetTile`、完成；不走旧地皮掉落或 soil 坍塌循环。[直接铺设](../../../raw/current/scripts/components/autoterraformer.lua#L72-L76)、[移除耗材](../../../raw/current/scripts/components/autoterraformer.lua#L24-L29)
3. 否则先检查 `CanTerraformAtPoint`。有 `undertile` 下层则恢复下层，**此步不消费槽内地皮**；无下层才消费地皮并换成其 `tile`，没有地皮时换成 DIRT。之后处理原地块掉落、soil 和完成回调。[铲地分支](../../../raw/current/scripts/components/autoterraformer.lua#L79-L103)
4. 移出用 `RemoveItem(item, false)`，普通堆叠一次拆一件。槽中原实体失效后，只有装备的总持有者具备 inventory 且正打开该容器，才从其库存寻找同 prefab、同 skinname 的替补；补槽迁移整堆，GiveItem 失败尝试还回库存。[补槽](../../../raw/current/scripts/components/autoterraformer.lua#L25-L50)、[容器拆堆实现](../../../raw/current/scripts/components/container.lua#L1019-L1036)

## 典型接入

### 沿用蚁狮帽的自动铺地与卸装停止

入口是 `hats.lua` 的 `fns.antlion`：服务端配置装备回调、finiteuses，再 `container:WidgetSetup("antlionhat")`，最后添加 autoterraformer。该布局仅 1 槽，带原版 UI 资源、排除合成材料查询；复制组件不自动取得这些资源与布局。[工厂](../../../raw/current/scripts/prefabs/hats.lua#L2415-L2437)、[容器布局](../../../raw/current/scripts/containers.lua#L1263-L1283)、[动画依赖](../../../raw/current/scripts/prefabs/hats.lua#L6878-L6883)

装备回调要求穿戴者有 locomotor 才启动，并向穿戴者打开容器；卸装停止更新并关闭容器。完成效果生成 `turf_smoke_fx`，耗尽回调先 `DropEverything` 再删除帽子。这些是**宿主负责的启停、展示与内容返还**，移植时保留完整边界，不能只复制 StartTerraforming 一行。[装备与卸装](../../../raw/current/scripts/prefabs/hats.lua#L2371-L2391)、[完成特效与耗尽](../../../raw/current/scripts/prefabs/hats.lua#L2395-L2402)、[耗尽注册](../../../raw/current/scripts/prefabs/hats.lua#L2428-L2437)

已有模组若只需改变连续剥层间隔，可在 `antlionhat` 的服务端 Prefab postinit 中修改 `repeat_tile_delay`；只影响此后需重复的同格操作，不会让静止于普通地块时无限铺地。该字段没有存档方法，须在每次构造时重设；停用模组不恢复已经改变的地图。原版挂载与间隔语义见上文，不把此配置片段称为完整新装备骨架。

## 常见陷阱与限制

- **每帧 Start，或把延迟当自动重试：** Start 清空格缓存；框架以组件实例为键注册更新，不新增并行计时任务，但反复启动会重置判断。正常更新只在初次、换格或“露出下层后延迟到零”时操作；资格失败／槽为空后原地补料不会主动重试，可走出再走回或由业务明确重启。[启动](../../../raw/current/scripts/components/autoterraformer.lua#L106-L108)、[条件与缓存](../../../raw/current/scripts/components/autoterraformer.lua#L123-L131)、[框架注册](../../../raw/current/scripts/entityscript.lua#L461-L468)
- **把成功返回值当布尔接口：** 直接铺设成功也返回 nil；只有露出下层返回 true 以申请下一次同格操作。观察完成用回调或宿主事件。[铺设返回](../../../raw/current/scripts/components/autoterraformer.lua#L72-L76)、[下层返回](../../../raw/current/scripts/components/autoterraformer.lua#L101-L103)
- **以为耗材移出失败会取消换地：** `HandleRemovingItem` 无成功返回，调用方随后仍 SetTile；只读容器会让 RemoveItem_Internal 返回 nil，因此不要将该组件直接用于不可移出耗材的容器。[调用顺序](../../../raw/current/scripts/components/autoterraformer.lua#L72-L75)、[只读门禁](../../../raw/current/scripts/components/container.lua#L1019-L1022)
- **忽略保存与拆组件：** 本组件无 OnSave／OnLoad／移除钩子；容器内容保存见 [[reference/components/container#超堆叠与删除内容]]，更新状态由装备流程重建。移除组件框架会停止其更新，但删除容器前应先停用依赖它的本组件，避免保留缓存旧实例。[完整定义](../../../raw/current/scripts/components/autoterraformer.lua#L1-L136)、[框架移除](../../../raw/current/scripts/entityscript.lua#L670-L683)
- **回调假定宿主仍存在：** 扣次可能触发宿主的耗尽删除，再进入完成回调；原版回调只取给定坐标生成特效。自定义回调访问组件前应核对实例状态，不能靠回调延后扣次。[组件顺序](../../../raw/current/scripts/components/autoterraformer.lua#L12-L20)、[耐久结束](../../../raw/current/scripts/components/finiteuses.lua#L64-L74)、[帽子删除](../../../raw/current/scripts/prefabs/hats.lua#L2400-L2402)

## 验证记录

- 版本增量复核 26148036（2026-10-25）：锚点重定位 8 处（hats.lua antlion 装配段 +2、动画依赖段随新增帽型平移至 6878-6883，tuning AUTOTERRAFORMER_REPEAT_DELAY +14）；无结论修订，仅锚点迁移——本版 hats.lua 改动均为其他帽型新增，antlion 段内容逐行同旧版。依据：全局枢纽 diff 审阅（entityscript 改动在湿润查询／距离函数，不涉组件更新注册 L461-468）＋变更清单命中 prefabs/hats.lua、containers.lua（本版改动在 socket_keystone 段）、tuning.lua、entityscript.lua。
- 源码审阅：2026-10-23，Steam buildid 25643519；覆盖完整组件、原版蚁狮帽服务端构造与装备／卸装／耗尽、容器槽限制与拆堆、有限耐久顺序及更新注册／移除。地图门禁细节复用同版本 terraformer 页，未审阅全部地块或其他模组适配。
- 结构检查：本页元数据、链接、大小及源码引用定向严格检查通过。未新增完整 Lua 示例，无示例语法检查项。
- 游戏运行：未运行；铺地／剥层、原地补料、耗尽删除、存档重载及多人容器交互仍待实测。

相关：[[reference/components/terraformer]]；[[reference/components/container]]；[[reference/components/finiteuses]]；[[reference/components/equippable]]。

## 相关链接

- [[reference/components/undertile]]：临时地皮底层 tile 记忆网格。
